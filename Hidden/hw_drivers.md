# Odoo Module: hw_drivers

Category: Hidden

This file contains the source code of the Odoo module.

## File: connection_manager.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import datetime, timedelta
import logging
import requests
from threading import Thread
import time
import urllib3

from odoo.addons.hw_drivers.main import iot_devices, manager
from odoo.addons.hw_drivers.tools import helpers

_logger = logging.getLogger(__name__)


class ConnectionManager(Thread):
    def __init__(self):
        super(ConnectionManager, self).__init__()
        self.pairing_code = False
        self.pairing_uuid = False

    def run(self):
        if not helpers.get_odoo_server_url() and not helpers.access_point():
            end_time = datetime.now() + timedelta(minutes=5)
            while datetime.now() < end_time:
                self._connect_box()
                time.sleep(10)
            self.pairing_code = False
            self.pairing_uuid = False
            self._refresh_displays()

    def _connect_box(self):
        data = {
            'jsonrpc': 2.0,
            'params': {
                'pairing_code': self.pairing_code,
                'pairing_uuid': self.pairing_uuid,
            }
        }

        try:
            urllib3.disable_warnings()
            req = requests.post('https://iot-proxy.odoo.com/odoo-enterprise/iot/connect-box', json=data, verify=False)
            result = req.json().get('result', {})
            if all(key in result for key in ['pairing_code', 'pairing_uuid']):
                self.pairing_code = result['pairing_code']
                self.pairing_uuid = result['pairing_uuid']
                self._refresh_displays()
            elif all(key in result for key in ['url', 'token', 'db_uuid', 'enterprise_code']):
                self._connect_to_server(result['url'], result['token'], result['db_uuid'], result['enterprise_code'])
        except Exception:
            _logger.exception('Could not reach iot-proxy.odoo.com')

    def _connect_to_server(self, url, token, db_uuid, enterprise_code):
        # Save DB URL and token
        helpers.save_conf_server(url, token, db_uuid, enterprise_code)
        # Notify the DB, so that the kanban view already shows the IoT Box
        manager.send_alldevices()
        # Restart to checkout the git branch, get a certificate, load the IoT handlers...
        helpers.odoo_restart(2)

    def _refresh_displays(self):
        """Refresh all displays to hide the pairing code"""
        for d in iot_devices:
            if iot_devices[d].device_type == 'display':
                iot_devices[d].action({
                    'action': 'display_refresh'
                })


connection_manager = ConnectionManager()
connection_manager.daemon = True
connection_manager.start()

```

## File: driver.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from threading import Thread, Event

from odoo.addons.hw_drivers.main import drivers, iot_devices
from odoo.tools.lru import LRU


class DriverMetaClass(type):
    priority = -1

    def __new__(cls, clsname, bases, attrs):
        newclass = super(DriverMetaClass, cls).__new__(cls, clsname, bases, attrs)
        newclass.priority += 1
        if clsname != 'Driver':
            drivers.append(newclass)
        return newclass


class Driver(Thread, metaclass=DriverMetaClass):
    """
    Hook to register the driver into the drivers list
    """
    connection_type = ''

    def __init__(self, identifier, device):
        super(Driver, self).__init__()
        self.dev = device
        self.device_identifier = identifier
        self.device_name = ''
        self.device_connection = ''
        self.device_type = ''
        self.device_manufacturer = ''
        self.data = {'value': ''}
        self._actions = {}
        self._stopped = Event()

        # Least Recently Used (LRU) Cache that will store the idempotent keys already seen.
        self._iot_idempotent_ids_cache = LRU(500)

    @classmethod
    def supported(cls, device):
        """
        On specific driver override this method to check if device is supported or not
        return True or False
        """
        return False

    def action(self, data):
        """Helper function that calls a specific action method on the device.

        :param data: the `_actions` key mapped to the action method we want to call
        :type data: string
        """
        self._actions[data.get('action', '')](data)

    def disconnect(self):
        self._stopped.set()
        del iot_devices[self.device_identifier]

    def _check_idempotency(self, iot_idempotent_id, session_id):
        """
        Some IoT requests for the same action might be received several times.
        To avoid duplicating the resulting actions, we check if the action was "recently" executed.
        If this is the case, we will simply ignore the action

        :return: the `session_id` of the same `iot_idempotent_id` if any. False otherwise,
        which means that it is the first time that the IoT box received the request with this ID
        """
        cache = self._iot_idempotent_ids_cache
        if iot_idempotent_id in cache:
            return cache[iot_idempotent_id]
        cache[iot_idempotent_id] = session_id
        return False

```

## File: event_manager.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json
from threading import Event
import time

from odoo.http import request

class EventManager(object):
    def __init__(self):
        self.events = []
        self.sessions = {}

    def _delete_expired_sessions(self, max_time=70):
        '''
        Clears sessions that are no longer called.

        :param max_time: time a session can stay unused before being deleted
        '''
        now = time.time()
        expired_sessions = [
            session
            for session in self.sessions
            if now - self.sessions[session]['time_request'] > max_time
        ]
        for session in expired_sessions:
            del self.sessions[session]

    def add_request(self, listener):
        self.session = {
            'session_id': listener['session_id'],
            'devices': listener['devices'],
            'event': Event(),
            'result': {},
            'time_request': time.time(),
        }
        self._delete_expired_sessions()
        self.sessions[listener['session_id']] = self.session
        return self.sessions[listener['session_id']]

    def device_changed(self, device):
        event = {
            **device.data,
            'device_identifier': device.device_identifier,
            'time': time.time(),
            'request_data': json.loads(request.params['data']) if request and 'data' in request.params else None,
        }
        self.events.append(event)
        for session in self.sessions:
            if device.device_identifier in self.sessions[session]['devices'] and not self.sessions[session]['event'].is_set():
                self.sessions[session]['result'] = event
                self.sessions[session]['event'].set()


event_manager = EventManager()

```

## File: exception_logger.py

```python

from io import StringIO

import logging
import sys

_logger = logging.getLogger(__name__)


class ExceptionLogger:
    """
    Redirect any unhandled python exception to the logger to keep track of them in the log file.
    """
    def __init__(self):
        self._buffer = StringIO()

    def write(self, message):
        self._buffer.write(message)
        if message.endswith('\n'):
            self._flush_buffer()

    def _flush_buffer(self):
        self._buffer.seek(0)
        _logger.error(self._buffer.getvalue().rstrip('\n'))
        self._buffer = StringIO()  # Reset the buffer

    def flush(self):
        if self._buffer.tell() > 0:
            self._flush_buffer()

    def close(self):
        self.flush()

sys.stderr = ExceptionLogger()

```

## File: http.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import odoo


def db_list(force=False, host=None):
    return []

odoo.http.db_list = db_list

```

## File: interface.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
from threading import Thread
import time

from odoo.addons.hw_drivers.main import drivers, interfaces, iot_devices

_logger = logging.getLogger(__name__)


class InterfaceMetaClass(type):
    def __new__(cls, clsname, bases, attrs):
        new_interface = super(InterfaceMetaClass, cls).__new__(cls, clsname, bases, attrs)
        if clsname != 'Interface':
            interfaces[clsname] = new_interface
        return new_interface


class Interface(Thread, metaclass=InterfaceMetaClass):
    _loop_delay = 3  # Delay (in seconds) between calls to get_devices or 0 if it should be called only once
    _detected_devices = {}
    connection_type = ''

    def __init__(self):
        super(Interface, self).__init__()
        self.drivers = sorted([d for d in drivers if d.connection_type == self.connection_type], key=lambda d: d.priority, reverse=True)

    def run(self):
        while self.connection_type and self.drivers:
            self.update_iot_devices(self.get_devices())
            if not self._loop_delay:
                break
            time.sleep(self._loop_delay)

    def update_iot_devices(self, devices={}):
        added = devices.keys() - self._detected_devices
        removed = self._detected_devices - devices.keys()
        # keys() returns a dict_keys, and the values of that stay in sync with the
        # original dictionary if it changes. This means that get_devices needs to return
        # a newly created dictionary every time. If it doesn't do that and reuses the
        # same dictionary, this logic won't detect any changes that are made. Could be
        # avoided by converting the dict_keys into a regular dict. The current logic
        # also can't detect if a device is replaced by a different one with the same
        # key. Also, _detected_devices starts out as a class variable but gets turned
        # into an instance variable here. It would be better if it was an instance
        # variable from the start to avoid confusion.
        self._detected_devices = devices.keys()

        for identifier in removed:
            if identifier in iot_devices:
                iot_devices[identifier].disconnect()
                _logger.info('Device %s is now disconnected', identifier)

        for identifier in added:
            for driver in self.drivers:
                if driver.supported(devices[identifier]):
                    _logger.info('Device %s is now connected', identifier)
                    d = driver(identifier, devices[identifier])
                    d.daemon = True
                    iot_devices[identifier] = d
                    # Start the thread after creating the iot_devices entry so the
                    # thread can assume the iot_devices entry will exist while it's
                    # running, at least until the `disconnect` above gets triggered
                    # when `removed` is not empty.
                    d.start()
                    break

    def get_devices(self):
        raise NotImplementedError()

```

## File: main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from traceback import format_exc
import json
import platform
import logging
from threading import Thread
import time
import urllib3

from odoo.addons.hw_drivers.tools import helpers
from odoo.addons.hw_drivers.websocket_client import WebsocketClient

_logger = logging.getLogger(__name__)

try:
    import schedule
except ImportError:
    schedule = None
    _logger.warning('Could not import library schedule')

try:
    from dbus.mainloop.glib import DBusGMainLoop
except ImportError:
    DBusGMainLoop = None
    _logger.error('Could not import library dbus')

drivers = []
interfaces = {}
iot_devices = {}


class Manager(Thread):
    def send_alldevices(self, iot_client=None):
        """
        This method send IoT Box and devices informations to Odoo database
        """
        server = helpers.get_odoo_server_url()
        if server:
            subject = helpers.get_conf('subject')
            if subject:
                domain = helpers.get_ip().replace('.', '-') + subject.strip('*')
            else:
                domain = helpers.get_ip()
            iot_box = {
                'name': helpers.get_hostname(),
                'identifier': helpers.get_mac_address(),
                'ip': domain,
                'token': helpers.get_token(),
                'version': helpers.get_version(detailed_version=True),
            }
            devices_list = {}
            for device in iot_devices:
                identifier = iot_devices[device].device_identifier
                devices_list[identifier] = {
                    'name': iot_devices[device].device_name,
                    'type': iot_devices[device].device_type,
                    'manufacturer': iot_devices[device].device_manufacturer,
                    'connection': iot_devices[device].device_connection,
                }
            devices_list_to_send = {
                key: value for key, value in devices_list.items() if key != 'distant_display'
            }
            data = {
                'params': {
                    'iot_box': iot_box,
                    'devices': devices_list_to_send,
                }  # Don't send distant_display to the db
            }
            # disable certifiacte verification
            urllib3.disable_warnings()
            http = urllib3.PoolManager(cert_reqs='CERT_NONE')
            try:
                resp = http.request(
                    'POST',
                    server + "/iot/setup",
                    body=json.dumps(data).encode('utf8'),
                    headers={
                        'Content-type': 'application/json',
                        'Accept': 'text/plain',
                    },
                )
                if iot_client:
                    iot_client.iot_channel = json.loads(resp.data).get('result', '')
            except json.decoder.JSONDecodeError:
                _logger.exception('Could not load JSON data: Received data is not in valid JSON format\ncontent:\n%s', resp.data)
            except Exception:
                _logger.exception('Could not reach configured server to send all IoT devices')
        else:
            _logger.info('Ignoring sending the devices to the database: no associated database')

    def run(self):
        """
        Thread that will load interfaces and drivers and contact the odoo server with the updates
        """
        helpers.migrate_old_config_files_to_new_config_file()
        server_url = helpers.get_odoo_server_url()

        helpers.start_nginx_server()
        _logger.info("IoT Box Image version: %s", helpers.get_version(detailed_version=True))
        if platform.system() == 'Linux' and server_url:
            helpers.check_git_branch()
            helpers.generate_password()
        is_certificate_ok, certificate_details = helpers.get_certificate_status()
        if not is_certificate_ok and certificate_details != 'ERR_IOT_HTTPS_CHECK_NO_SERVER':
            _logger.warning("An error happened when trying to get the HTTPS certificate: %s",
                            certificate_details)

        iot_client = server_url and WebsocketClient(server_url)
        # We first add the IoT Box to the connected DB because IoT handlers cannot be downloaded if
        # the identifier of the Box is not found in the DB. So add the Box to the DB.
        self.send_alldevices(iot_client)
        helpers.download_iot_handlers()
        helpers.load_iot_handlers()

        # Start the interfaces
        for interface in interfaces.values():
            try:
                i = interface()
                i.daemon = True
                i.start()
            except Exception:
                _logger.exception("Interface %s could not be started", str(interface))

        # Set scheduled actions
        schedule and schedule.every().day.at("00:00").do(helpers.get_certificate_status)

        #Setup the websocket connection
        if server_url:
            iot_client.start()
        # Check every 3 secondes if the list of connected devices has changed and send the updated
        # list to the connected DB.
        self.previous_iot_devices = []
        while 1:
            try:
                if iot_devices != self.previous_iot_devices:
                    self.previous_iot_devices = iot_devices.copy()
                    self.send_alldevices(iot_client)
                time.sleep(3)
                schedule and schedule.run_pending()
            except Exception:
                # No matter what goes wrong, the Manager loop needs to keep running
                _logger.exception("Manager loop unexpected error")

# Must be started from main thread
if DBusGMainLoop:
    DBusGMainLoop(set_as_default=True)

manager = Manager()
manager.daemon = True
manager.start()

```

## File: server_logger.py

```python

import logging
import queue
import requests
import threading
import time
import urllib3.exceptions

from odoo.addons.hw_drivers.tools import helpers
from odoo.netsvc import DBFormatter
from odoo.tools import config

_logger = logging.getLogger(__name__)

IOT_LOG_TO_SERVER_CONFIG_NAME = 'iot_log_to_server'  # config name in odoo.conf


class AsyncHTTPHandler(logging.Handler):
    """
    Custom logging handler which send IoT logs using asynchronous requests.
    To avoid spamming the server, we send logs by batch each X seconds
    """
    _MAX_QUEUE_SIZE = 1000
    """Maximum queue size. If a log record is received but the queue if full it will be discarded"""
    _MAX_BATCH_SIZE = 50
    """Maximum number of sent logs batched at once. Used to avoid too heavy request. Log records still in the queue will
    be handle in future flushes"""
    _FLUSH_INTERVAL = 0.5
    """How much seconds it will sleep before checking for new logs to send"""
    _REQUEST_TIMEOUT = 0.5
    """Amount of seconds to wait per log to send before timeout"""
    _DELAY_BEFORE_NO_SERVER_LOG = 5 * 60  # 5 minutes
    """Minimum delay in seconds before we log a server disconnection.
    Used in order to avoid the IoT log file to have a log recorded each _FLUSH_INTERVAL (as this value is very small)"""

    def __init__(self, odoo_server_url, active):
        """
        :param odoo_server_url: Odoo Server URL
        """
        super().__init__()
        self._odoo_server_url = odoo_server_url
        self._log_queue = queue.Queue(self._MAX_QUEUE_SIZE)
        self._flush_thread = None
        self._active = None
        self._next_disconnection_time = None
        self.toggle_active(active)

    def toggle_active(self, is_active):
        """
        Switch it on or off the handler (depending on the IoT setting) without the need to close/reset it
        """
        self._active = is_active
        if self._active and self._odoo_server_url:
            # Start the thread to periodically flush logs
            self._flush_thread = threading.Thread(target=self._periodic_flush, name="ThreadServerLogSender", daemon=True)
            self._flush_thread.start()
        else:
            self._flush_thread and self._flush_thread.join()  # let a last flush

    def _periodic_flush(self):
        odoo_session = requests.Session()
        while self._odoo_server_url and self._active:  # allow to exit the loop on thread.join
            time.sleep(self._FLUSH_INTERVAL)
            self._flush_logs(odoo_session)

    def _flush_logs(self, odoo_session):
        def convert_to_byte(s):
            return bytes(s, encoding="utf-8") + b'<log/>\n'

        def convert_server_line(log_level, line_formatted):
            return convert_to_byte(f"{log_level},{line_formatted}")

        def empty_queue():
            yield convert_to_byte(f"mac {helpers.get_mac_address()}")
            for _ in range(self._MAX_BATCH_SIZE):
                # Use a limit to avoid having too heavy requests & infinite loop of the queue receiving new entries
                try:
                    log_record = self._log_queue.get_nowait()
                    yield convert_server_line(log_record.levelno, self.format(log_record))
                except queue.Empty:
                    break

            # Report to the server if the queue is close from saturation
            if queue_size >= .8 * self._MAX_QUEUE_SIZE:
                log_message = "The IoT {} queue is saturating: {}/{} ({:.2f}%)".format(
                    self.__class__.__name__, queue_size, self._MAX_QUEUE_SIZE,
                    100 * queue_size / self._MAX_QUEUE_SIZE)
                _logger.warning(log_message)  # As we don't log our own logs, this will be part of the IoT logs
                # In order to report this to the server (on the current batch) we will append it manually
                yield convert_server_line(logging.WARNING, log_message)

        queue_size = self._log_queue.qsize()  # This is an approximate value

        if not self._odoo_server_url or queue_size == 0:
            return
        try:
            odoo_session.post(
                self._odoo_server_url + '/iot/log',
                data=empty_queue(),
                timeout=self._REQUEST_TIMEOUT
            ).raise_for_status()
            self._next_disconnection_time = None
        except (requests.exceptions.ConnectTimeout, requests.exceptions.ConnectionError, urllib3.exceptions.NewConnectionError) as request_errors:
            now = time.time()
            if not self._next_disconnection_time or now >= self._next_disconnection_time:
                _logger.info("Connection with the server to send the logs failed. It is likely down: %s", request_errors)
                self._next_disconnection_time = now + self._DELAY_BEFORE_NO_SERVER_LOG
        except Exception as _:
            _logger.exception('Unexpected error happened while sending logs to server')

    def emit(self, record):
        # This is important that this method is as fast as possible.
        # The log calls will be waiting for this function to finish
        if not self._active:
            return
        try:
            self._log_queue.put_nowait(record)
        except queue.Full:
            pass

    def close(self):
        self.toggle_active(False)
        super().close()


def close_server_log_sender_handler():
    _server_log_sender_handler.close()


def get_odoo_config_log_to_server_option():
    return config.get(IOT_LOG_TO_SERVER_CONFIG_NAME, True)  # Enabled by default


def check_and_update_odoo_config_log_to_server_option(new_state):
    """
    :return: wherever the config file need to be updated or not
    """
    if get_odoo_config_log_to_server_option() != new_state:
        config[IOT_LOG_TO_SERVER_CONFIG_NAME] = new_state
        _server_log_sender_handler.toggle_active(new_state)
        return True
    return False


def _server_log_sender_handler_filter(log_record):
    def _filter_my_logs():
        """Filter out our own logs (to avoid infinite loop)"""
        return log_record.name == __name__

    def _filter_frequent_irrelevant_calls():
        """Filter out this frequent irrelevant HTTP calls, to avoid spamming the server with useless logs"""
        return log_record.name == 'werkzeug' and log_record.args and len(log_record.args) > 0 and log_record.args[0].startswith('GET /hw_proxy/hello ')

    return not (_filter_my_logs() or _filter_frequent_irrelevant_calls())


# The server URL is set once at initlialisation as the IoT will always restart if the URL is changed
# The only other possible case is when the server URL value is "Cleared",
# in this case we force close the log handler (as it does not make sense anymore)
_server_log_sender_handler = AsyncHTTPHandler(helpers.get_odoo_server_url(), get_odoo_config_log_to_server_option())
_server_log_sender_handler.setFormatter(DBFormatter('%(asctime)s %(pid)s %(levelname)s %(dbname)s %(name)s: %(message)s %(perf_info)s'))
_server_log_sender_handler.addFilter(_server_log_sender_handler_filter)
# Set it in the 'root' logger, on which every logger (including odoo) is a child
logging.getLogger().addHandler(_server_log_sender_handler)

```

## File: websocket_client.py

```python
import json
import logging
import pprint
import time
import urllib.parse
import urllib3
import websocket

from threading import Thread

from odoo.addons.hw_drivers import main
from odoo.addons.hw_drivers.tools import helpers

_logger = logging.getLogger(__name__)
websocket.enableTrace(True, level=logging.getLevelName(_logger.getEffectiveLevel()))


def send_to_controller(print_id, device_identifier):
    """
    Send back to odoo's server the completion of the operation
    """
    server = helpers.get_odoo_server_url()
    try:
        urllib3.disable_warnings()
        http = urllib3.PoolManager(cert_reqs='CERT_NONE')
        http.request(
            'POST',
            server + "/iot/printer/status",
            body=json.dumps(
                {'params': {
                    'print_id': print_id,
                    'device_identifier': device_identifier,
                    'iot_mac': helpers.get_mac_address(),
                    }}).encode('utf8'),
            headers={
                'Content-type': 'application/json',
                'Accept': 'text/plain',
            },
        )
    except Exception:
        _logger.exception('Could not reach configured server: %s', server)


def on_message(ws, messages):
    """
        Synchronously handle messages received by the websocket.
    """
    messages = json.loads(messages)
    _logger.debug("websocket received a message: %s", pprint.pformat(messages))
    for document in messages:
        message_type = document['message']['type']
        if message_type in ['print', 'iot_action']:
            payload = document['message']['payload']
            iot_mac = helpers.get_mac_address()
            if iot_mac in payload['iotDevice']['iotIdentifiers']:
                for device in payload['iotDevice']['identifiers']:
                    device_identifier = device['identifier']
                    if device_identifier in main.iot_devices:
                        start_operation_time = time.perf_counter()
                        _logger.debug("device '%s' action started with: %s", device_identifier, pprint.pformat(payload))
                        main.iot_devices[device_identifier]._action_default(payload)
                        _logger.info("device '%s' action finished - %.*f", device_identifier, 3, time.perf_counter() - start_operation_time)
                        send_to_controller(payload['print_id'], device_identifier)
            else:
                # likely intended as IoT share the same channel
                _logger.debug("message ignored due to different iot mac: %s", iot_mac)
        elif message_type not in ['print_confirmation', 'bundle_changed']:  # intended to be ignored
            _logger.warning("message type not supported: %s", message_type)


def on_error(ws, error):
    _logger.error("websocket received an error: %s", error)


def on_close(ws, close_status_code, close_msg):
    _logger.debug("websocket closed with status: %s", close_status_code)


class WebsocketClient(Thread):
    iot_channel = ""

    def on_open(self, ws):
        """
            When the client is setup, this function send a message to subscribe to the iot websocket channel
        """
        ws.send(
            json.dumps({'event_name': 'subscribe', 'data': {'channels': [self.iot_channel], 'last': 0}})
        )

    def __init__(self, url):
        url_parsed = urllib.parse.urlsplit(url)
        scheme = url_parsed.scheme.replace("http", "ws", 1)
        self.url = urllib.parse.urlunsplit((scheme, url_parsed.netloc, 'websocket', '', ''))
        Thread.__init__(self)

    def run(self):
        self.ws = websocket.WebSocketApp(self.url,
            header={"User-Agent": "OdooIoTBox/1.0"},
            on_open=self.on_open, on_message=on_message,
            on_error=on_error, on_close=on_close)

        # The IoT synchronised servers can stop in 2 ways that we need to handle:
        #  A. Gracefully:
        #   In this case a disconnection signal is sent to the IoT-box
        #   The websocket is properly closed, but it needs to be established a new connection when
        #   the server will be back.
        #
        # B. Forced/killed:
        #   In this case there is no disconnection signal received
        #
        #   This will also happen with the graceful quit as `reconnect` will trigger if the server
        #   is offline while attempting the new connection
        while True:
            try:
                run_res = self.ws.run_forever(reconnect=10)
                _logger.debug("websocket run_forever return with %s", run_res)
            except Exception:
                _logger.exception("An unexpected exception happened when running the websocket")
            _logger.debug('websocket will try to restart in 10 seconds')
            time.sleep(10)

```

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import server_logger
from . import connection_manager
from . import controllers
from . import driver
from . import event_manager
from . import exception_logger
from . import http
from . import interface
from . import main
from . import websocket_client

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Hardware Proxy',
    'category': 'Hidden',
    'sequence': 6,
    'summary': 'Connect the Web Client to Hardware Peripherals',
    'website': 'https://www.odoo.com/app/iot',
    'description': """
Hardware Poxy
=============

This module allows you to remotely use peripherals connected to this server.

This modules only contains the enabling framework. The actual devices drivers
are found in other modules that must be installed separately.

""",
    'installable': False,
    'license': 'LGPL-3',
}

```

## File: controllers\driver.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from base64 import b64decode
from datetime import datetime
import json
import logging
import os
import subprocess
from socket import gethostname
import time
from werkzeug.exceptions import InternalServerError
from zlib import adler32

from odoo import http, tools

from odoo.addons.hw_drivers.event_manager import event_manager
from odoo.addons.hw_drivers.main import iot_devices, manager
from odoo.addons.hw_drivers.tools import helpers

_logger = logging.getLogger(__name__)


class DriverController(http.Controller):
    @http.route('/hw_drivers/action', type='json', auth='none', cors='*', csrf=False, save_session=False)
    def action(self, session_id, device_identifier, data):
        """
        This route is called when we want to make a action with device (take picture, printing,...)
        We specify in data from which session_id that action is called
        And call the action of specific device
        """
        iot_device = iot_devices.get(device_identifier)
        if iot_device:
            iot_device.data['owner'] = session_id
            data = json.loads(data)

            # Skip the request if it was already executed (duplicated action calls)
            iot_idempotent_id = data.get("iot_idempotent_id")
            if iot_idempotent_id:
                idempotent_session = iot_device._check_idempotency(iot_idempotent_id, session_id)
                if idempotent_session:
                    _logger.info("Ignored request from %s as iot_idempotent_id %s already received from session %s",
                                 session_id, iot_idempotent_id, idempotent_session)
                    return False
            _logger.debug("Calling action %s for device %s", data.get('action', ''), device_identifier)
            iot_device.action(data)
            return True
        return False

    @http.route('/hw_drivers/check_certificate', type='http', auth='none', cors='*', csrf=False, save_session=False)
    def check_certificate(self):
        """
        This route is called when we want to check if certificate is up-to-date
        Used in iot-box cron.daily, deprecated since image 24_10 but needed for compatibility with the image 24_01
        """
        helpers.get_certificate_status()

    @http.route('/hw_drivers/event', type='json', auth='none', cors='*', csrf=False, save_session=False)
    def event(self, listener):
        """
        listener is a dict in witch there are a sessions_id and a dict of device_identifier to listen
        """
        req = event_manager.add_request(listener)
        # Search for previous events and remove events older than 5 seconds
        oldest_time = time.time() - 5
        for event in list(event_manager.events):
            if event['time'] < oldest_time:
                del event_manager.events[0]
                continue
            if event['device_identifier'] in listener['devices'] and event['time'] > listener['last_event']:
                event['session_id'] = req['session_id']
                _logger.debug("Event %s found for device %s ", event, event['device_identifier'])
                return event

        # Wait for new event
        if req['event'].wait(50):
            req['event'].clear()
            req['result']['session_id'] = req['session_id']
            return req['result']

    @http.route('/hw_drivers/download_logs', type='http', auth='none', cors='*', csrf=False, save_session=False)
    def download_logs(self):
        """
        Downloads the log file
        """
        log_path = tools.config['logfile'] or "/var/log/odoo/odoo-server.log"
        try:
            stat = os.stat(log_path)
        except FileNotFoundError:
            raise InternalServerError("Log file has not been found. Check your Log file configuration.")
        check = adler32(log_path.encode())
        log_file_name = f"iot-odoo-{gethostname()}-{datetime.now().strftime('%Y-%m-%d_%H-%M-%S')}.log"
        # intentionally don't use Stream.from_path as the path used is not in the addons path
        # for instance, for the iot-box it will be in /var/log/odoo
        return http.Stream(
                type='path',
                path=log_path,
                download_name=log_file_name,
                etag=f'{int(stat.st_mtime)}-{stat.st_size}-{check}',
                last_modified=stat.st_mtime,
                size=stat.st_size,
                mimetype='text/plain',
            ).get_response(
            mimetype='text/plain', as_attachment=True
        )

```

## File: controllers\proxy.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http

proxy_drivers = {}

class ProxyController(http.Controller):
    @http.route('/hw_proxy/hello', type='http', auth='none', cors='*')
    def hello(self):
        return "ping"

    @http.route('/hw_proxy/handshake', type='json', auth='none', cors='*')
    def handshake(self):
        return True

    @http.route('/hw_proxy/status_json', type='json', auth='none', cors='*')
    def status_json(self):
        statuses = {}
        for driver in proxy_drivers:
            statuses[driver] = proxy_drivers[driver].get_status()
        return statuses

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import driver
from . import proxy

```

## File: iot_handlers\drivers\DisplayDriver_L.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import jinja2
import json
import logging
import netifaces as ni
import os
import subprocess
import socket
import threading
import time
import urllib3

from odoo import http
from odoo.addons.hw_drivers.connection_manager import connection_manager
from odoo.addons.hw_drivers.driver import Driver
from odoo.addons.hw_drivers.event_manager import event_manager
from odoo.addons.hw_drivers.main import iot_devices
from odoo.addons.hw_drivers.tools import helpers
from odoo.tools.misc import file_open

path = os.path.realpath(os.path.join(os.path.dirname(__file__), '../../views'))
loader = jinja2.FileSystemLoader(path)

jinja_env = jinja2.Environment(loader=loader, autoescape=True)
jinja_env.filters["json"] = json.dumps

pos_display_template = jinja_env.get_template('pos_display.html')

_logger = logging.getLogger(__name__)


class DisplayDriver(Driver):
    connection_type = 'display'

    def __init__(self, identifier, device):
        super(DisplayDriver, self).__init__(identifier, device)
        self.device_type = 'display'
        self.device_connection = 'hdmi'
        self.device_name = device['name']
        self.event_data = threading.Event()
        self.owner = False
        self.rendered_html = ''
        if self.device_identifier != 'distant_display':
            # helpers.get_version returns a string formatted as: <L|W><version> (L: Linux, W: Windows)
            self.browser = 'chromium-browser' if float(helpers.get_version()[1:]) >= 24.10 else 'firefox'
            self.browser_process_name = 'chromium' if self.browser == 'chromium-browser' else self.browser
            self._x_screen = device.get('x_screen', '0')
            self.load_url()

        self._actions.update({
            'update_url': self._action_update_url,
            'display_refresh': self._action_display_refresh,
            'take_control': self._action_take_control,
            'customer_facing_display': self._action_customer_facing_display,
            'get_owner': self._action_get_owner,
        })

    @classmethod
    def supported(cls, device):
        return True  # All devices with connection_type == 'display' are supported

    @classmethod
    def get_default_display(cls):
        displays = list(filter(lambda d: iot_devices[d].device_type == 'display', iot_devices))
        return len(displays) and iot_devices[displays[0]]

    def run(self):
        while self.device_identifier != 'distant_display' and not self._stopped.is_set():
            time.sleep(60)
            if self.url != 'http://localhost:8069/point_of_sale/display/' + self.device_identifier:
                # Refresh the page every minute
                self.call_xdotools('F5')

    def update_url(self, url=None):
        os.environ['DISPLAY'] = ":0." + self._x_screen
        os.environ['XAUTHORITY'] = '/run/lightdm/pi/xauthority'
        browser_env = os.environ.copy()
        for key in ['HOME', 'XDG_RUNTIME_DIR', 'XDG_CACHE_HOME']:
            browser_env[key] = '/tmp/' + self._x_screen
        self.url = url or 'http://localhost:8069/point_of_sale/display/' + self.device_identifier

        # Kill browser instance (can't `instance.pkill()` as we can't keep the instance after Odoo service restarts)
        # We need to terminate it because Odoo will create a new instance each time it is restarted.
        subprocess.run(['pkill', self.browser.split('-')[0]], check=False)
        browser_args = [
            '--start-fullscreen',
            '--log-level=3',        # avoid useless log messages
            '--bwsi',               # use chromium without signing in
            '--disable-extensions'  # disable extensions as they fill up /tmp
        ]
        subprocess.Popen([self.browser, self.url, *browser_args], env=browser_env)

        # To remove when everyone is on version >= 24.10: chromium has '--start-fullscreen' option
        if self.browser == 'firefox':
            self.call_xdotools('F11')

    def load_url(self):
        url = None
        if helpers.get_odoo_server_url():
            # disable certifiacte verification
            urllib3.disable_warnings()
            http = urllib3.PoolManager(cert_reqs='CERT_NONE')
            try:
                response = http.request('GET', "%s/iot/box/%s/display_url" % (helpers.get_odoo_server_url(), helpers.get_mac_address()))
                if response.status == 200:
                    data = json.loads(response.data.decode('utf8'))
                    url = data[self.device_identifier]
            except json.decoder.JSONDecodeError:
                url = response.data.decode('utf8')
            except Exception:
                pass
        return self.update_url(url)

    def call_xdotools(self, keystroke):
        os.environ['DISPLAY'] = ":0." + self._x_screen
        os.environ['XAUTHORITY'] = "/run/lightdm/pi/xauthority"
        try:
            subprocess.run([
                'xdotool',
                'search',
                '--sync',
                '--onlyvisible',
                '--screen',
                self._x_screen,
                '--class',
                self.browser_process_name,
                'key',
                keystroke,
            ], check=False)
            return "xdotool succeeded in stroking " + keystroke
        except:
            return "xdotool threw an error, maybe it is not installed on the IoTBox"

    def update_customer_facing_display(self, origin, html=None):
        if origin == self.owner:
            self.rendered_html = html
            self.event_data.set()

    def get_serialized_order(self):
        # IMPLEMENTATION OF LONGPOLLING
        # Times out 2 seconds before the JS request does
        if self.event_data.wait(28):
            self.event_data.clear()
            return {'rendered_html': self.rendered_html}
        return {'rendered_html': False}

    def take_control(self, new_owner, html=None):
        # ALLOW A CASHIER TO TAKE CONTROL OVER THE POSBOX, IN CASE OF MULTIPLE CASHIER PER DISPLAY
        self.owner = new_owner
        self.rendered_html = html
        self.data = {
            'value': '',
            'owner': self.owner,
        }
        event_manager.device_changed(self)
        self.event_data.set()

    def _action_update_url(self, data):
        if self.device_identifier != 'distant_display':
            self.update_url(data.get('url'))

    def _action_display_refresh(self, data):
        if self.device_identifier != 'distant_display':
            self.call_xdotools('F5')

    def _action_take_control(self, data):
        self.take_control(self.data.get('owner'), data.get('html'))

    def _action_customer_facing_display(self, data):
        self.update_customer_facing_display(self.data.get('owner'), data.get('html'))

    def _action_get_owner(self, data):
        self.data = {
            'value': '',
            'owner': self.owner,
        }
        event_manager.device_changed(self)


class DisplayController(http.Controller):
    @http.route('/hw_proxy/display_refresh', type='json', auth='none', cors='*')
    def display_refresh(self):
        display = DisplayDriver.get_default_display()
        if display and display.device_identifier != 'distant_display':
            return display.call_xdotools('F5')

    @http.route('/hw_proxy/customer_facing_display', type='json', auth='none', cors='*')
    def customer_facing_display(self, html=None):
        display = DisplayDriver.get_default_display()
        if display:
            display.update_customer_facing_display(http.request.httprequest.remote_addr, html)
            return {'status': 'updated'}
        return {'status': 'failed'}

    @http.route('/hw_proxy/take_control', type='json', auth='none', cors='*')
    def take_control(self, html=None):
        display = DisplayDriver.get_default_display()
        if display:
            display.take_control(http.request.httprequest.remote_addr, html)
            return {
                'status': 'success',
                'message': 'You now have access to the display',
            }

    @http.route('/hw_proxy/test_ownership', type='json', auth='none', cors='*')
    def test_ownership(self):
        display = DisplayDriver.get_default_display()
        if display and display.owner == http.request.httprequest.remote_addr:
            return {'status': 'OWNER'}
        return {'status': 'NOWNER'}

    @http.route(['/point_of_sale/get_serialized_order', '/point_of_sale/get_serialized_order/<string:display_identifier>'], type='json', auth='none')
    def get_serialized_order(self, display_identifier=None):
        if display_identifier:
            display = iot_devices.get(display_identifier)
        else:
            display = DisplayDriver.get_default_display()

        if display:
            return display.get_serialized_order()
        return {
            'rendered_html': False,
            'error': "No display found",
        }

    @http.route(['/point_of_sale/display', '/point_of_sale/display/<string:display_identifier>'], type='http', auth='none')
    def display(self, display_identifier=None):
        cust_js = None
        interfaces = ni.interfaces()

        with file_open("hw_drivers/static/src/js/worker.js") as js:
            cust_js = js.read()

        display_ifaces = []
        for iface_id in interfaces:
            if 'wlan' in iface_id or 'eth' in iface_id:
                iface_obj = ni.ifaddresses(iface_id)
                ifconfigs = iface_obj.get(ni.AF_INET, [])
                essid = helpers.get_ssid()
                for conf in ifconfigs:
                    if conf.get('addr'):
                        display_ifaces.append({
                            'iface_id': iface_id,
                            'essid': essid,
                            'addr': conf.get('addr'),
                            'icon': 'sitemap' if 'eth' in iface_id else 'wifi',
                        })

        default_display = DisplayDriver.get_default_display()
        if not display_identifier and default_display != 0:
            display_identifier = default_display.device_identifier

        return pos_display_template.render({
            'title': "Odoo -- Point of Sale",
            'breadcrumb': 'POS Client display',
            'cust_js': cust_js,
            'display_ifaces': display_ifaces,
            'display_identifier': display_identifier,
            'hostname': socket.gethostname(),
            'pairing_code': connection_manager.pairing_code,
        })

    @http.route('/point_of_sale/iot_devices', type='json', auth='none', methods=['POST'])
    def get_iot_devices(self):
        iot_device = [{
            'name': iot_devices[device].device_name,
            'type': iot_devices[device].device_type,
        } for device in iot_devices]

        return json.dumps({'iot_device_status': iot_device})

```

## File: iot_handlers\drivers\KeyboardUSBDriver_L.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import ctypes
import evdev
import json
import logging
from lxml import etree
import os
from pathlib import Path
from queue import Queue, Empty
import re
import subprocess
from threading import Lock
import time
import urllib3
from usb import util

from odoo import http
from odoo.addons.hw_drivers.controllers.proxy import proxy_drivers
from odoo.addons.hw_drivers.driver import Driver
from odoo.addons.hw_drivers.event_manager import event_manager
from odoo.addons.hw_drivers.main import iot_devices
from odoo.addons.hw_drivers.tools import helpers

_logger = logging.getLogger(__name__)
xlib = ctypes.cdll.LoadLibrary('libX11.so.6')


class KeyboardUSBDriver(Driver):
    # The list of devices can be found in /proc/bus/input/devices
    # or "ls -l /dev/input/by-path"
    # Note: the user running "evdev" commands must be inside the "input" group
    # Each device's input will correspond to a file in /dev/input/event*
    # The exact file can be found by looking at the "Handlers" line in /proc/bus/input/devices
    # Example: "H: Handlers=sysrq kbd leds event0" -> The file used is /dev/input/event0
    # If you read the file "/dev/input/event0" you will get the input from the device in real time
    # One usb device can have multiple associated event files (like a foot pedal which has 3 event files)

    connection_type = 'usb'
    keyboard_layout_groups = []
    available_layouts = []
    input_devices = []

    def __init__(self, identifier, device):
        if not hasattr(KeyboardUSBDriver, 'display'):
            os.environ['XAUTHORITY'] = "/run/lightdm/pi/xauthority"
            KeyboardUSBDriver.display = xlib.XOpenDisplay(bytes(":0.0", "utf-8"))

        super(KeyboardUSBDriver, self).__init__(identifier, device)
        self.device_connection = 'direct'
        self.device_name = self._set_name()

        self._actions.update({
            'update_layout': self._update_layout,
            'update_is_scanner': self._save_is_scanner,
            '': self._action_default,
        })

        # from https://github.com/xkbcommon/libxkbcommon/blob/master/test/evdev-scancodes.h
        self._scancode_to_modifier = {
            42: 'left_shift',
            54: 'right_shift',
            58: 'caps_lock',
            69: 'num_lock',
            100: 'alt_gr', # right alt
        }
        self._tracked_modifiers = {modifier: False for modifier in self._scancode_to_modifier.values()}

        if not KeyboardUSBDriver.available_layouts:
            KeyboardUSBDriver.load_layouts_list()
        KeyboardUSBDriver.send_layouts_list()

        for evdev_device in [evdev.InputDevice(path) for path in evdev.list_devices()]:
            if (device.idVendor == evdev_device.info.vendor) and (device.idProduct == evdev_device.info.product):
                self.input_devices.append(evdev_device)

        self._set_device_type('scanner') if self._is_scanner() else self._set_device_type()

    @classmethod
    def supported(cls, device):
        for cfg in device:
            for itf in cfg:
                if itf.bInterfaceClass == 3 and itf.bInterfaceProtocol != 2:
                    device.interface_protocol = itf.bInterfaceProtocol
                    return True
        return False

    @classmethod
    def get_status(self):
        """Allows `hw_proxy.Proxy` to retrieve the status of the scanners"""
        status = 'connected' if any(iot_devices[d].device_type == "scanner" for d in iot_devices) else 'disconnected'
        return {'status': status, 'messages': ''}

    @classmethod
    def send_layouts_list(cls):
        server = helpers.get_odoo_server_url()
        if server:
            urllib3.disable_warnings()
            pm = urllib3.PoolManager(cert_reqs='CERT_NONE')
            server = server + '/iot/keyboard_layouts'
            try:
                pm.request('POST', server, fields={'available_layouts': json.dumps(cls.available_layouts)})
            except Exception:
                _logger.exception('Could not reach configured server to send available layouts')

    @classmethod
    def load_layouts_list(cls):
        tree = etree.parse("/usr/share/X11/xkb/rules/base.xml", etree.XMLParser(ns_clean=True, recover=True))
        layouts = tree.xpath("//layout")
        for layout in layouts:
            layout_name = layout.xpath("./configItem/name")[0].text
            layout_description = layout.xpath("./configItem/description")[0].text
            KeyboardUSBDriver.available_layouts.append({
                'name': layout_description,
                'layout': layout_name,
            })
            for variant in layout.xpath("./variantList/variant"):
                variant_name = variant.xpath("./configItem/name")[0].text
                variant_description = variant.xpath("./configItem/description")[0].text
                KeyboardUSBDriver.available_layouts.append({
                    'name': variant_description,
                    'layout': layout_name,
                    'variant': variant_name,
                })

    def _set_name(self):
        try:
            manufacturer = util.get_string(self.dev, self.dev.iManufacturer)
            product = util.get_string(self.dev, self.dev.iProduct)
            if manufacturer and product:
                return re.sub(r"[^\w \-+/*&]", '', "%s - %s" % (manufacturer, product))
        except ValueError as e:
            _logger.warning(e)
        return 'Unknown input device'

    def run(self):
        try:
            for device in self.input_devices:
                for event in device.read_loop():
                    if self._stopped.is_set():
                        break
                    if event.type == evdev.ecodes.EV_KEY:
                        data = evdev.categorize(event)

                        modifier_name = self._scancode_to_modifier.get(data.scancode)
                        if modifier_name:
                            if modifier_name in ('caps_lock', 'num_lock'):
                                if data.keystate == 1:
                                    self._tracked_modifiers[modifier_name] = not self._tracked_modifiers[modifier_name]
                            else:
                                self._tracked_modifiers[modifier_name] = bool(data.keystate)  # 1 for keydown, 0 for keyup
                        elif data.keystate == 1:
                            self.key_input(data.scancode)

        except Exception as err:
            _logger.warning(err)

    def _change_keyboard_layout(self, new_layout):
        """Change the layout of the current device to what is specified in
        new_layout.

        Args:
            new_layout (dict): A dict containing two keys:
                - layout (str): The layout code
                - variant (str): An optional key to represent the variant of the
                                 selected layout
        """
        if hasattr(self, 'keyboard_layout'):
            KeyboardUSBDriver.keyboard_layout_groups.remove(self.keyboard_layout)

        if new_layout:
            self.keyboard_layout = new_layout.get('layout') or 'us'
            if new_layout.get('variant'):
                self.keyboard_layout += "(%s)" % new_layout['variant']
        else:
            self.keyboard_layout = 'us'

        KeyboardUSBDriver.keyboard_layout_groups.append(self.keyboard_layout)
        subprocess.call(["setxkbmap", "-display", ":0.0", ",".join(KeyboardUSBDriver.keyboard_layout_groups)])

        # Close then re-open display to refresh the mapping
        xlib.XCloseDisplay(KeyboardUSBDriver.display)
        KeyboardUSBDriver.display = xlib.XOpenDisplay(bytes(":0.0", "utf-8"))

    def save_layout(self, layout):
        """Save the layout to a file on the box to read it when restarting it.
        We need that in order to keep the selected layout after a reboot.

        Args:
            new_layout (dict): A dict containing two keys:
                - layout (str): The layout code
                - variant (str): An optional key to represent the variant of the
                                 selected layout
        """
        file_path = helpers.path_file('odoo-keyboard-layouts.conf')
        if file_path.exists():
            data = json.loads(file_path.read_text())
        else:
            data = {}
        data[self.device_identifier] = layout
        helpers.write_file('odoo-keyboard-layouts.conf', json.dumps(data))

    def load_layout(self):
        """Read the layout from the saved filed and set it as current layout.
        If no file or no layout is found we use 'us' by default.
        """
        file_path = helpers.path_file('odoo-keyboard-layouts.conf')
        if file_path.exists():
            data = json.loads(file_path.read_text())
            layout = data.get(self.device_identifier, {'layout': 'us'})
        else:
            layout = {'layout': 'us'}
        self._change_keyboard_layout(layout)

    def _action_default(self, data):
        self.data['value'] = ''
        event_manager.device_changed(self)

    def _is_scanner(self):
        """Read the device type from the saved filed and set it as current type.
        If no file or no device type is found we try to detect it automatically.
        """
        device_name = self.device_name.lower()
        scanner_name = ['barcode', 'scanner', 'reader']
        is_scanner = any(x in device_name for x in scanner_name) or self.dev.interface_protocol == '0'

        file_path = helpers.path_file('odoo-keyboard-is-scanner.conf')
        if file_path.exists():
            data = json.loads(file_path.read_text())
            is_scanner = data.get(self.device_identifier, {}).get('is_scanner', is_scanner)
        return is_scanner

    def _keyboard_input(self, scancode):
        """Deal with a keyboard input. Send the character corresponding to the
        pressed key represented by its scancode to the connected Odoo instance.

        Args:
            scancode (int): The scancode of the pressed key.
        """
        self.data['value'] = self._scancode_to_char(scancode)
        if self.data['value']:
            event_manager.device_changed(self)

    def _barcode_scanner_input(self, scancode):
        """Deal with a barcode scanner input. Add the new character scanned to
        the current barcode or complete the barcode if "Return" is pressed.
        When a barcode is completed, two tasks are performed:
            - Send a device_changed update to the event manager to notify the
            listeners that the value has changed (used in Enterprise).
            - Add the barcode to the list barcodes that are being queried in
            Community.

        Args:
            scancode (int): The scancode of the pressed key.
        """
        if scancode == 28:  # Return
            self.data['value'] = self._current_barcode
            event_manager.device_changed(self)
            self._barcodes.put((time.time(), self._current_barcode))
            self._current_barcode = ''
        else:
            self._current_barcode += self._scancode_to_char(scancode)

    def _save_is_scanner(self, data):
        """Save the type of device.
        We need that in order to keep the selected type of device after a reboot.
        """
        is_scanner = {'is_scanner': data.get('is_scanner')}
        file_path = helpers.path_file('odoo-keyboard-is-scanner.conf')
        if file_path.exists():
            data = json.loads(file_path.read_text())
        else:
            data = {}
        data[self.device_identifier] = is_scanner
        helpers.write_file('odoo-keyboard-is-scanner.conf', json.dumps(data))
        self._set_device_type('scanner') if is_scanner.get('is_scanner') else self._set_device_type()

    def _update_layout(self, data):
        layout = {
            'layout': data.get('layout'),
            'variant': data.get('variant'),
        }
        self._change_keyboard_layout(layout)
        self.save_layout(layout)

    def _set_device_type(self, device_type='keyboard'):
        """Modify the device type between 'keyboard' and 'scanner'

        Args:
            type (string): Type wanted to switch
        """
        if device_type == 'scanner':
            self.device_type = 'scanner'
            self.key_input = self._barcode_scanner_input
            self._barcodes = Queue()
            self._current_barcode = ''
            for device in self.input_devices:
                device.grab()
            self.read_barcode_lock = Lock()
        else:
            self.device_type = 'keyboard'
            self.key_input = self._keyboard_input
        self.load_layout()

    def _scancode_to_char(self, scancode):
        """Translate a received scancode to a character depending on the
        selected keyboard layout and the current state of the keyboard's
        modifiers.

        Args:
            scancode (int): The scancode of the pressed key, to be translated to
                a character

        Returns:
            str: The translated scancode.
        """
        # Scancode -> Keysym : Depends on the keyboard layout
        group = KeyboardUSBDriver.keyboard_layout_groups.index(self.keyboard_layout)
        modifiers = self._get_active_modifiers(scancode)
        keysym = ctypes.c_int(xlib.XkbKeycodeToKeysym(KeyboardUSBDriver.display, scancode + 8, group, modifiers))

        # Translate Keysym to a character
        key_pressed = ctypes.create_string_buffer(5)
        xlib.XkbTranslateKeySym(KeyboardUSBDriver.display, ctypes.byref(keysym), 0, ctypes.byref(key_pressed), 5, ctypes.byref(ctypes.c_int()))
        if key_pressed.value:
            return key_pressed.value.decode('utf-8')
        return ''

    def _get_active_modifiers(self, scancode):
        """Get the state of currently active modifiers.

        Args:
            scancode (int): The scancode of the key being translated

        Returns:
            int: The current state of the modifiers:
                0 -- Lowercase
                1 -- Highercase or (NumLock + key pressed on keypad)
                2 -- AltGr
                3 -- Highercase + AltGr
        """
        modifiers = 0
        uppercase = (self._tracked_modifiers['right_shift'] or self._tracked_modifiers['left_shift']) ^ self._tracked_modifiers['caps_lock']
        if uppercase or (scancode in [71, 72, 73, 75, 76, 77, 79, 80, 81, 82, 83] and self._tracked_modifiers['num_lock']):
            modifiers += 1

        if self._tracked_modifiers['alt_gr']:
            modifiers += 2

        return modifiers

    def read_next_barcode(self):
        """Get the value of the last barcode that was scanned but not sent yet
        and not older than 5 seconds. This function is used in Community, when
        we don't have access to the IoTLongpolling.

        Returns:
            str: The next barcode to be read or an empty string.
        """

        # Previous query still running, stop it by sending a fake barcode
        if self.read_barcode_lock.locked():
            self._barcodes.put((time.time(), ""))

        with self.read_barcode_lock:
            try:
                timestamp, barcode = self._barcodes.get(True, 55)
                if timestamp > time.time() - 5:
                    return barcode
            except Empty:
                return ''

proxy_drivers['scanner'] = KeyboardUSBDriver


class KeyboardUSBController(http.Controller):
    @http.route('/hw_proxy/scanner', type='json', auth='none', cors='*')
    def get_barcode(self):
        scanners = [iot_devices[d] for d in iot_devices if iot_devices[d].device_type == "scanner"]
        if scanners:
            return scanners[0].read_next_barcode()
        time.sleep(5)
        return None

```

## File: iot_handlers\drivers\L10nEGDrivers.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import base64
import logging
import platform
import json

from passlib.context import CryptContext

from odoo import http
from odoo.tools.config import config

_logger = logging.getLogger(__name__)

try:
    import PyKCS11
except ImportError:
    PyKCS11 = None
    _logger.error('Could not import library PyKCS11')

crypt_context = CryptContext(schemes=['pbkdf2_sha512'])


class EtaUsbController(http.Controller):

    def _is_access_token_valid(self, access_token):
        stored_hash = config.get('proxy_access_token')
        if not stored_hash:
            # empty password/hash => authentication forbidden
            return False
        return crypt_context.verify(access_token, stored_hash)

    @http.route('/hw_l10n_eg_eta/certificate', type='http', auth='none', cors='*', csrf=False, save_session=False, methods=['POST'])
    def eta_certificate(self, pin, access_token):
        """
        Gets the certificate from the token and returns it to the main odoo instance so that we can prepare the
        cades-bes object on the main odoo instance rather than this middleware
        @param pin: pin of the token
        @param access_token: token shared with the main odoo instance
        """
        if not PyKCS11:
            return self._get_error_template('no_pykcs11')
        if not self._is_access_token_valid(access_token):
            return self._get_error_template('unauthorized')
        session, error = self._get_session(pin)
        if error:
            return error
        try:
            cert = session.findObjects([(PyKCS11.CKA_CLASS, PyKCS11.CKO_CERTIFICATE)])[0]
            cert_bytes = bytes(session.getAttributeValue(cert, [PyKCS11.CKA_VALUE])[0])
            payload = {
                'certificate': base64.b64encode(cert_bytes).decode()
            }
            return json.dumps(payload)
        except Exception as ex:
            _logger.exception('Error while getting ETA certificate')
            return self._get_error_template(str(ex))
        finally:
            session.logout()
            session.closeSession()

    @http.route('/hw_l10n_eg_eta/sign', type='http', auth='none', cors='*', csrf=False, save_session=False, methods=['POST'])
    def eta_sign(self, pin, access_token, invoices):
        """
        Check if the access_token is valid and sign the invoices accessing the usb key with the pin.
        @param pin: pin of the token
        @param access_token: token shared with the main odoo instance
        @param invoices: dictionary of invoices. Keys are invoices ids, value are the base64 encoded binaries to sign
        """
        if not PyKCS11:
            return self._get_error_template('no_pykcs11')
        if not self._is_access_token_valid(access_token):
            return self._get_error_template('unauthorized')
        session, error = self._get_session(pin)
        if error:
            return error
        try:
            cert = session.findObjects([(PyKCS11.CKA_CLASS, PyKCS11.CKO_CERTIFICATE)])[0]
            cert_id = session.getAttributeValue(cert, [PyKCS11.CKA_ID])[0]
            priv_key = session.findObjects([(PyKCS11.CKA_CLASS, PyKCS11.CKO_PRIVATE_KEY), (PyKCS11.CKA_ID, cert_id)])[0]

            invoice_dict = dict()
            invoices = json.loads(invoices)
            for invoice, eta_inv in invoices.items():
                to_sign = base64.b64decode(eta_inv)
                signed_data = session.sign(priv_key, to_sign, PyKCS11.Mechanism(PyKCS11.CKM_SHA256_RSA_PKCS))
                invoice_dict[invoice] = base64.b64encode(bytes(signed_data)).decode()

            payload = {
                'invoices': json.dumps(invoice_dict),
            }
            return json.dumps(payload)
        except Exception as ex:
            _logger.exception('Error while signing invoices')
            return self._get_error_template(str(ex))
        finally:
            session.logout()
            session.closeSession()

    def _get_session(self, pin):
        session = False

        lib, error = self.get_crypto_lib()
        if error:
            return session, error

        try:
            pkcs11 = PyKCS11.PyKCS11Lib()
            pkcs11.load(pkcs11dll_filename=lib)
        except PyKCS11.PyKCS11Error:
            return session, self._get_error_template('missing_dll')

        slots = pkcs11.getSlotList(tokenPresent=True)
        if not slots:
            return session, self._get_error_template('no_drive')
        if len(slots) > 1:
            return session, self._get_error_template('multiple_drive')

        try:
            session = pkcs11.openSession(slots[0], PyKCS11.CKF_SERIAL_SESSION | PyKCS11.CKF_RW_SESSION)
            session.login(pin)
        except Exception as ex:
            error = self._get_error_template(str(ex))
        return session, error

    def get_crypto_lib(self):
        error = lib = False
        system = platform.system()
        if system == 'Linux':
            lib = '/usr/lib/x86_64-linux-gnu/opensc-pkcs11.so'
        elif system == 'Windows':
            lib = 'C:/Windows/System32/eps2003csp11.dll'
        elif system == 'Darwin':
            lib = '/Library/OpenSC/lib/onepin-opensc-pkcs11.so'
        else:
            error = self._get_error_template('unsupported_system')
        return lib, error

    def _get_error_template(self, error_str):
        return json.dumps({
            'error': error_str,
        })

```

## File: iot_handlers\drivers\L10nKeEDISerialDriver.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import serial
import time
import struct
import json
from functools import reduce

from odoo import http
from odoo.addons.hw_drivers.iot_handlers.drivers.SerialBaseDriver import SerialDriver, SerialProtocol, serial_connection
from odoo.addons.hw_drivers.main import iot_devices

_logger = logging.getLogger(__name__)

TremolG03Protocol = SerialProtocol(
    name='Tremol G03',
    baudrate=115200,
    bytesize=serial.EIGHTBITS,
    stopbits=serial.STOPBITS_ONE,
    parity=serial.PARITY_NONE,
    timeout=4,
    writeTimeout=0.2,
    measureRegexp=None,
    statusRegexp=None,
    commandTerminator=b'',
    commandDelay=0.2,
    measureDelay=3,
    newMeasureDelay=0.2,
    measureCommand=b'',
    emptyAnswerValid=False,
)

STX = 0x02
ETX = 0x0A
ACK = 0x06
NACK = 0x15

# Dictionary defining the output size of expected from various commands
COMMAND_OUTPUT_SIZE = {
    0x30: 7,
    0x31: 7,
    0x38: 157,
    0x39: 155,
    0x60: 40,
    0x68: 23,
}

FD_ERRORS = {
    0x30: 'OK',
    0x32: 'Registers overflow',
    0x33: 'Clock failure or incorrect date & time',
    0x34: 'Opened fiscal receipt',
    0x39: 'Incorrect password',
    0x3b: '24 hours block - missing Z report',
    0x3d: 'Interrupt power supply in fiscal receipt (one time until status is read)',
    0x3e: 'Overflow EJ',
    0x3f: 'Insufficient conditions',
}

COMMAND_ERRORS = {
    0x30: 'OK',
    0x31: 'Invalid command',
    0x32: 'Illegal command',
    0x33: 'Z daily report is not zero',
    0x34: 'Syntax error',
    0x35: 'Input registers orverflow',
    0x36: 'Zero input registers',
    0x37: 'Unavailable transaction for correction',
    0x38: 'Insufficient amount on hand',
}


class TremolG03Driver(SerialDriver):
    """Driver for the Kenyan Tremol G03 fiscal device."""

    _protocol = TremolG03Protocol

    def __init__(self, identifier, device):
        super().__init__(identifier, device)
        self.device_type = 'fiscal_data_module'
        self.message_number = 0

    @classmethod
    def get_default_device(cls):
        fiscal_devices = list(filter(lambda d: iot_devices[d].device_type == 'fiscal_data_module', iot_devices))
        return len(fiscal_devices) and iot_devices[fiscal_devices[0]]

    @classmethod
    def supported(cls, device):
        """Checks whether the device, which port info is passed as argument, is supported by the driver.

        :param device: path to the device
        :type device: str
        :return: whether the device is supported by the driver
        :rtype: bool
        """
        protocol = cls._protocol
        try:
            protocol = cls._protocol
            with serial_connection(device['identifier'], protocol) as connection:
                connection.write(b'\x09')
                time.sleep(protocol.commandDelay)
                response = connection.read(1)
                if response == b'\x40':
                    return True

        except serial.serialutil.SerialTimeoutException:
            pass
        except Exception:
            _logger.exception('Error while probing %s with protocol %s', device, protocol.name)

    # ----------------
    # HELPERS
    # ----------------

    @staticmethod
    def generate_checksum(message):
        """ Generate the checksum bytes for the bytes provided.

        :param message: bytes representing the part of the message from which the checksum is calculated
        :returns:       two checksum bytes calculated from the message

         This checksum is calculated as:
        1) XOR of all bytes of the bytes
        2) Conversion of the one XOR byte into the two bytes of the checksum by
           adding 30h to each half-byte of the XOR

        eg. to_check = \x12\x23\x34\x45\x56
            XOR of all bytes in to_check = \x16
            checksum generated as \x16 -> \x31 \x36
        """
        xor = reduce(lambda a, b: a ^ b, message)
        return bytes([(xor >> 4) + 0x30, (xor & 0xf) + 0x30])

    # ----------------
    # COMMUNICATION
    # ----------------

    def send(self, msgs):
        """ Send and receive messages to/from the fiscal device over serial connection

        Generate the wrapped message from the msgs and send them to the device.
        The wrapping contains the <STX> (starting byte) <LEN> (length byte)
        and <NBL> (message number byte) at the start and two <CS> (checksum
        bytes), and the <ETX> line-feed byte at the end.
        :param msgs: A list of byte strings representing the <CMD> and <DATA>
                     components of the serial message.
        :return:     A list of the responses (if any) from the device. If the
                     response is an ack, it wont be part of this list.
        """

        with self._device_lock:
            replies = []
            for msg in msgs:
                self.message_number += 1
                core_message = struct.pack('BB%ds' % (len(msg)), len(msg) + 34, self.message_number + 32, msg)
                request = struct.pack('B%ds2sB' % (len(core_message)), STX, core_message, self.generate_checksum(core_message), ETX)
                time.sleep(self._protocol.commandDelay)
                self._connection.write(request)
                _logger.debug('Debug send request: %s', request)
                # If we know the expected output size, we can set the read
                # buffer to match the size of the output.
                output_size = COMMAND_OUTPUT_SIZE.get(msg[0])
                if output_size:
                    try:
                        response = self._connection.read(output_size)
                    except serial.serialutil.SerialTimeoutException:
                        _logger.exception('Timeout error while reading response to command %s', msg)
                        self.data['status'] = "Device timeout error"
                else:
                    time.sleep(self._protocol.measureDelay)
                    response = self._connection.read_all()
                _logger.debug('Debug send response: %s', response)
                if not response:
                    self.data['status'] = "No response"
                    _logger.error("Sent request: %s,\n Received no response", request)
                    self.abort_post()
                    break
                if response[0] == ACK:
                    # In the case where either byte is not 0x30, there has been an error
                    if response[2] != 0x30 or response[3] != 0x30:
                        self.data['status'] = response[2:4].decode('cp1251')
                        _logger.error(
                            "Sent request: %s,\n Received fiscal device error: %s \n Received command error: %s",
                            request, FD_ERRORS.get(response[2], 'Unknown fiscal device error'),
                            COMMAND_ERRORS.get(response[3], 'Unknown command error'),
                        )
                        self.abort_post()
                        break
                    replies.append('')
                elif response[0] == NACK:
                    self.data['status'] = "Received NACK"
                    _logger.error("Sent request: %s,\n Received NACK \x15", request)
                    self.abort_post()
                    break
                elif response[0] == 0x02:
                    self.data['status'] = "ok"
                    size = response[1] - 35
                    reply = response[4:4 + size]
                    replies.append(reply.decode('cp1251'))
        return {'replies': replies, 'status': self.data['status']}

    def abort_post(self):
        """ Cancel the posting of the invoice

        In the event of an error, it is better to try to cancel the posting of
        the invoice, since the state of the invoice on the device will remain
        open otherwise, blocking further invoices being sent.
        """
        self.message_number += 1
        abort = struct.pack('BBB', 35, self.message_number + 32, 0x39)
        request = struct.pack('B3s2sB', STX, abort, self.generate_checksum(abort), ETX)
        self._connection.write(request)
        _logger.debug('Debug abort_post request: %s', request)
        response = self._connection.read(COMMAND_OUTPUT_SIZE[0x39])
        _logger.debug('Debug abort_post response: %s', response)
        if response and response[0] == 0x02:
            self.data['status'] += "\n The invoice was successfully cancelled"
            _logger.info("Invoice successfully cancelled")
        else:
            self.data['status'] += "\n The invoice could not be cancelled."
            _logger.error("Failed to cancel invoice, received response: %s", response)


class TremolG03Controller(http.Controller):

    @http.route('/hw_proxy/l10n_ke_cu_send', type='http', auth='none', cors='*', csrf=False, save_session=False, methods=['POST'])
    def l10n_ke_cu_send(self, messages, company_vat):
        """ Posts the messages sent to this endpoint to the fiscal device connected to the server

        :param messages:     The messages (consisting of <CMD> and <DATA>) to
                             send to the fiscal device.
        :returns:            Dictionary containing a list of the responses from
                             fiscal device and status of the fiscal device.
        """
        device = TremolG03Driver.get_default_device()
        if device:
            # First run the command to get the fiscal device numbers
            device_numbers = device.send([b'\x60'])
            # If the vat doesn't match, abort
            if device_numbers['status'] != 'ok':
                return device_numbers
            serial_number, device_vat, _dummy = device_numbers['replies'][0].split(';')
            if device_vat != company_vat:
                return json.dumps({'status': 'The company vat number does not match that of the device'})
            messages = json.loads(messages)
            device.message_number = 0
            resp = json.dumps({**device.send([msg.encode('cp1251') for msg in messages]), 'serial_number': serial_number})
            return resp
        else:
            return json.dumps({'status': 'The fiscal device is not connected to the proxy server'})

```

## File: iot_handlers\drivers\PrinterDriver_L.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from base64 import b64decode
from cups import IPPError, IPP_PRINTER_IDLE, IPP_PRINTER_PROCESSING, IPP_PRINTER_STOPPED
import dbus
import io
import logging
import netifaces as ni
from PIL import Image, ImageOps
import re
import subprocess

from odoo import http
from odoo.addons.hw_drivers.connection_manager import connection_manager
from odoo.addons.hw_drivers.controllers.proxy import proxy_drivers
from odoo.addons.hw_drivers.driver import Driver
from odoo.addons.hw_drivers.event_manager import event_manager
from odoo.addons.hw_drivers.iot_handlers.interfaces.PrinterInterface_L import PPDs, conn, cups_lock
from odoo.addons.hw_drivers.main import iot_devices
from odoo.addons.hw_drivers.tools import helpers

_logger = logging.getLogger(__name__)

RECEIPT_PRINTER_COMMANDS = {
    'star': {
        'center': b'\x1b\x1d\x61\x01', # ESC GS a n
        'cut': b'\x1b\x64\x02',  # ESC d n
        'title': b'\x1b\x69\x01\x01%s\x1b\x69\x00\x00',  # ESC i n1 n2
        'drawers': [b'\x07', b'\x1a']  # BEL & SUB
    },
    'escpos': {
        'center': b'\x1b\x61\x01',  # ESC a n
        'cut': b'\x1d\x56\x41\n',  # GS V m
        'title': b'\x1b\x21\x30%s\x1b\x21\x00',  # ESC ! n
        'drawers': [b'\x1b\x3d\x01', b'\x1b\x70\x00\x19\x19', b'\x1b\x70\x01\x19\x19']  # ESC = n then ESC p m t1 t2
    }
}

def cups_notification_handler(message, uri, device_identifier, state, reason, accepting_jobs):
    if device_identifier in iot_devices:
        reason = reason if reason != 'none' else None
        state_value = {
            IPP_PRINTER_IDLE: 'connected',
            IPP_PRINTER_PROCESSING: 'processing',
            IPP_PRINTER_STOPPED: 'stopped'
        }
        iot_devices[device_identifier].update_status(state_value[state], message, reason)


# Create a Cups subscription if it doesn't exist yet
try:
    conn.getSubscriptions('/printers/')
except IPPError:
    conn.createSubscription(
        uri='/printers/',
        recipient_uri='dbus://',
        events=['printer-state-changed']
    )

# Listen for notifications from Cups
bus = dbus.SystemBus()
bus.add_signal_receiver(cups_notification_handler, signal_name="PrinterStateChanged", dbus_interface="org.cups.cupsd.Notifier")


class PrinterDriver(Driver):
    connection_type = 'printer'

    def __init__(self, identifier, device):
        super(PrinterDriver, self).__init__(identifier, device)
        self.device_type = 'printer'
        self.device_connection = device['device-class'].lower()
        self.device_name = device['device-make-and-model']
        self.state = {
            'status': 'connecting',
            'message': 'Connecting to printer',
            'reason': None,
        }
        self.send_status()

        self._actions.update({
            'cashbox': self.open_cashbox,
            'print_receipt': self.print_receipt,
            '': self._action_default,
        })

        self.receipt_protocol = 'star' if 'STR_T' in device['device-id'] else 'escpos'
        if 'direct' in self.device_connection and any(cmd in device['device-id'] for cmd in ['CMD:STAR;', 'CMD:ESC/POS;']):
            self.print_status()

    @classmethod
    def supported(cls, device):
        if device.get('supported', False):
            return True
        protocol = ['dnssd', 'lpd', 'socket']
        if (
                any(x in device['url'] for x in protocol)
                and device['device-make-and-model'] != 'Unknown'
                or 'direct' in device['device-class']
        ):
            model = cls.get_device_model(device)
            ppd_file = ''
            for ppd in PPDs:
                if model and model in PPDs[ppd]['ppd-product']:
                    ppd_file = ppd
                    break
            with cups_lock:
                if ppd_file:
                    conn.addPrinter(name=device['identifier'], ppdname=ppd_file, device=device['url'])
                else:
                    conn.addPrinter(name=device['identifier'], device=device['url'])

                conn.setPrinterInfo(device['identifier'], device['device-make-and-model'])
                conn.enablePrinter(device['identifier'])
                conn.acceptJobs(device['identifier'])
                conn.setPrinterUsersAllowed(device['identifier'], ['all'])
                conn.addPrinterOptionDefault(device['identifier'], "usb-no-reattach", "true")
                conn.addPrinterOptionDefault(device['identifier'], "usb-unidir", "true")
            return True
        return False

    @classmethod
    def get_device_model(cls, device):
        device_model = ""
        if device.get('device-id'):
            for device_id in [device_lo for device_lo in device['device-id'].split(';')]:
                if any(x in device_id for x in ['MDL', 'MODEL']):
                    device_model = device_id.split(':')[1]
                    break
        elif device.get('device-make-and-model'):
            device_model = device['device-make-and-model']
        return re.sub(r"[\(].*?[\)]", "", device_model).strip()

    @classmethod
    def get_status(cls):
        status = 'connected' if any(
            iot_devices[d].device_type == "printer"
            and iot_devices[d].device_connection == 'direct'
            for d in iot_devices
        ) else 'disconnected'
        return {'status': status, 'messages': ''}

    def disconnect(self):
        self.update_status('disconnected', 'Printer was disconnected')
        super(PrinterDriver, self).disconnect()

    def update_status(self, status, message, reason=None):
        """Updates the state of the current printer.

        Args:
            status (str): The new value of the status
            message (str): A comprehensive message describing the status
            reason (str): The reason fo the current status
        """
        if self.state['status'] != status or self.state['reason'] != reason:
            self.state = {
                'status': status,
                'message': message,
                'reason': reason,
            }
            self.send_status()

    def send_status(self):
        """ Sends the current status of the printer to the connected Odoo instance.
        """
        self.data = {
            'value': '',
            'state': self.state,
        }
        event_manager.device_changed(self)

    def print_raw(self, data, landscape=False, duplex=True):
        """
        Print raw data to the printer
        :param data: The data to print
        :param landscape: Print in landscape mode (Default: False)
        :param duplex: Print in duplex mode (recto-verso) (Default: True)
        """
        options = []
        if landscape:
            options.extend(['-o', 'orientation-requested=4'])
        if not duplex:
            options.extend(['-o', 'sides=one-sided'])
        cmd = ["lp", "-d", self.device_identifier, *options]

        _logger.debug("Printing using command: %s", cmd)
        process = subprocess.Popen(cmd, stdin=subprocess.PIPE)
        process.communicate(data)
        if process.returncode != 0:
            # The stderr isn't meaningful, so we don't log it ('No such file or directory')
            _logger.error('Printing failed: printer with the identifier "%s" could not be found',
                          self.device_identifier)

    def print_receipt(self, data):
        _logger.debug("print_receipt called for printer %s", self.device_name)

        receipt = b64decode(data['receipt'])
        im = Image.open(io.BytesIO(receipt))

        # Convert to greyscale then to black and white
        im = im.convert("L")
        im = ImageOps.invert(im)
        im = im.convert("1")

        print_command = getattr(self, 'format_%s' % self.receipt_protocol)(im)
        self.print_raw(print_command)

    def format_star(self, im):
        width = int((im.width + 7) / 8)

        raster_init = b'\x1b\x2a\x72\x41'
        raster_page_length = b'\x1b\x2a\x72\x50\x30\x00'
        raster_send = b'\x62'
        raster_close = b'\x1b\x2a\x72\x42'

        raster_data = b''
        dots = im.tobytes()
        while len(dots):
            raster_data += raster_send + width.to_bytes(2, 'little') + dots[:width]
            dots = dots[width:]

        return raster_init + raster_page_length + raster_data + raster_close

    def format_escpos_bit_image_raster(self, im):
        """ prints with the `GS v 0`-command """
        width = int((im.width + 7) / 8)

        raster_send = b'\x1d\x76\x30\x00'
        max_slice_height = 255

        raster_data = b''
        dots = im.tobytes()
        while len(dots):
            im_slice = dots[:width*max_slice_height]
            slice_height = int(len(im_slice) / width)
            raster_data += raster_send + width.to_bytes(2, 'little') + slice_height.to_bytes(2, 'little') + im_slice
            dots = dots[width*max_slice_height:]

        return raster_data

    def extract_columns_from_picture(self, im, line_height):
        # Code inspired from python esc pos library:
        # https://github.com/python-escpos/python-escpos/blob/4a0f5855ef118a2009b843a3a106874701d8eddf/src/escpos/image.py#L73-L89
        width_pixels, height_pixels = im.size
        for left in range(0, width_pixels, line_height):
            box = (left, 0, left + line_height, height_pixels)
            im_chunk = im.transform(
                (line_height, height_pixels),
                Image.EXTENT,
                box
            )
            yield im_chunk.tobytes()

    def format_escpos_bit_image_column(self, im, high_density_vertical=True,
                                       high_density_horizontal=True,
                                       size_scale=100):
        """ prints with the `ESC *`-command
        reference: https://reference.epson-biz.com/modules/ref_escpos/index.php?content_id=88

        :param im: PIL image to print
        :param high_density_vertical: print in high density in vertical direction
        :param high_density_horizontal: print in high density in horizontal direction
        :param size_scale: picture scale in percentage,
        e.g: 50 -> half the size (horizontally and vertically)
        """
        size_scale_ratio = size_scale / 100
        size_scale_width = int(im.width * size_scale_ratio)
        size_scale_height = int(im.height * size_scale_ratio)
        im = im.resize((size_scale_width, size_scale_height))
        # escpos ESC * command print column per column
        # (instead of usual row by row).
        # So we transpose the picture to ease the calculations
        im = im.transpose(Image.ROTATE_270).transpose(Image.FLIP_LEFT_RIGHT)

        # Most of the code here is inspired from python escpos library
        # https://github.com/python-escpos/python-escpos/blob/4a0f5855ef118a2009b843a3a106874701d8eddf/src/escpos/escpos.py#L237C9-L251
        ESC = b'\x1b'
        density_byte = (1 if high_density_horizontal else 0) + \
                       (32 if high_density_vertical else 0)
        nL = im.height & 0xFF
        nH = (im.height >> 8) & 0xFF
        HEADER = ESC + b'*' + bytes([density_byte, nL, nH])

        raster_data = ESC + b'3\x10'  # Adjust line-feed size
        line_height = 24 if high_density_vertical else 8
        for column in self.extract_columns_from_picture(im, line_height):
            raster_data += HEADER + column + b'\n'
        raster_data += ESC + b'2'  # Reset line-feed size
        return raster_data

    def format_escpos(self, im):
        # Epson support different command to print pictures.
        # We use by default "GS v 0", but it  is incompatible with certain
        # printer models (like TM-U2x0)
        # As we are pretty limited in the information that we have, we will
        # use the printer name to parse some configuration value
        # Printer name examples:
        # EpsonTMM30
        #  -> Print using raster mode
        # TM-U220__IMC_LDV_LDH_SCALE70__
        #  -> Print using column bit image mode (without vertical and
        #  horizontal density and a scale of 70%)

        # Default image printing mode
        image_mode = 'raster'

        options_str = self.device_name.split('__')
        option_str = ""
        if len(options_str) > 2:
            option_str = options_str[1].upper()
            if option_str.startswith('IMC'):
                image_mode = 'column'

        if image_mode == 'column':
            # Default printing mode parameters
            high_density_vertical = True
            high_density_horizontal = True
            scale = 100

            # Parse the printer name to get the needed parameters
            # The separator need to not be filtered by `get_identifier`
            options = option_str.split('_')
            for option in options:
                if option == 'LDV':
                    high_density_vertical = False
                elif option == 'LDH':
                    high_density_horizontal = False
                elif option.startswith('SCALE'):
                    scale_value_str = re.search(r'\d+$', option)
                    if scale_value_str is not None:
                        scale = int(scale_value_str.group())
                    else:
                        raise ValueError(
                            "Missing printer SCALE parameter integer "
                            "value in option: " + option)

            res = self.format_escpos_bit_image_column(im,
                                                      high_density_vertical,
                                                      high_density_horizontal,
                                                      scale)
        else:
            res = self.format_escpos_bit_image_raster(im)
        return res + RECEIPT_PRINTER_COMMANDS['escpos']['cut']

    def print_status(self):
        """Prints the status ticket of the IoTBox on the current printer."""
        wlan = ''
        ip = ''
        mac = ''
        homepage = ''
        pairing_code = ''

        ssid = helpers.get_ssid()
        wlan = '\nWireless network:\n%s\n\n' % ssid

        interfaces = ni.interfaces()
        ips = []
        for iface_id in interfaces:
            iface_obj = ni.ifaddresses(iface_id)
            ifconfigs = iface_obj.get(ni.AF_INET, [])
            for conf in ifconfigs:
                if conf.get('addr') and conf.get('addr'):
                    ips.append(conf.get('addr'))
        if len(ips) == 0:
            ip = '\nERROR: Could not connect to LAN\n\nPlease check that the IoTBox is correc-\ntly connected with a network cable,\n that the LAN is setup with DHCP, and\nthat network addresses are available'
        elif len(ips) == 1:
            ip = '\nIP Address:\n%s\n' % ips[0]
        else:
            ip = '\nIP Addresses:\n%s\n' % '\n'.join(ips)

        if len(ips) >= 1:
            ips_filtered = [i for i in ips if i != '127.0.0.1']
            main_ips = ips_filtered and ips_filtered[0] or '127.0.0.1'
            mac = '\nMAC Address:\n%s\n' % helpers.get_mac_address()
            homepage = '\nHomepage:\nhttp://%s:8069\n\n' % main_ips

        code = connection_manager.pairing_code
        if code:
            pairing_code = '\nPairing Code:\n%s\n' % code

        commands = RECEIPT_PRINTER_COMMANDS[self.receipt_protocol]
        title = commands['title'] % b'IoTBox Status'
        self.print_raw(commands['center'] + title + b'\n' + wlan.encode() + mac.encode() + ip.encode() + homepage.encode() + pairing_code.encode() + commands['cut'])

    def open_cashbox(self, data):
        """Sends a signal to the current printer to open the connected cashbox."""
        _logger.debug("open_cashbox called for printer %s", self.device_name)

        commands = RECEIPT_PRINTER_COMMANDS[self.receipt_protocol]
        for drawer in commands['drawers']:
            self.print_raw(drawer)

    def _action_default(self, data):
        _logger.debug("_action_default called for printer %s", self.device_name)
        self.print_raw(b64decode(data['document']))


class PrinterController(http.Controller):

    @http.route('/hw_proxy/default_printer_action', type='json', auth='none', cors='*')
    def default_printer_action(self, data):
        printer = next((d for d in iot_devices if iot_devices[d].device_type == 'printer' and iot_devices[d].device_connection == 'direct'), None)
        if printer:
            iot_devices[printer].action(data)
            return True
        return False


proxy_drivers['printer'] = PrinterDriver

```

## File: iot_handlers\drivers\PrinterDriver_W.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from PIL import Image, ImageOps
import logging
from base64 import b64decode
import io
import win32print
import ghostscript

from odoo.addons.hw_drivers.controllers.proxy import proxy_drivers
from odoo.addons.hw_drivers.driver import Driver
from odoo.addons.hw_drivers.event_manager import event_manager
from odoo.addons.hw_drivers.main import iot_devices
from odoo.addons.hw_drivers.tools import helpers
from odoo.tools.mimetypes import guess_mimetype

_logger = logging.getLogger(__name__)

RECEIPT_PRINTER_COMMANDS = {
    'star': {
        'center': b'\x1b\x1d\x61\x01', # ESC GS a n
        'cut': b'\x1b\x64\x02',  # ESC d n
        'title': b'\x1b\x69\x01\x01%s\x1b\x69\x00\x00',  # ESC i n1 n2
        'drawers': [b'\x07', b'\x1a']  # BEL & SUB
    },
    'escpos': {
        'center': b'\x1b\x61\x01',  # ESC a n
        'cut': b'\x1d\x56\x41\n',  # GS V m
        'title': b'\x1b\x21\x30%s\x1b\x21\x00',  # ESC ! n
        'drawers': [b'\x1b\x3d\x01', b'\x1b\x70\x00\x19\x19', b'\x1b\x70\x01\x19\x19']  # ESC = n then ESC p m t1 t2
    }
}

class PrinterDriver(Driver):
    connection_type = 'printer'

    def __init__(self, identifier, device):
        super().__init__(identifier, device)
        self.device_type = 'printer'
        self.device_connection = self._compute_device_connection(device)
        self.device_name = device.get('identifier')
        self.printer_handle = device.get('printer_handle')
        self.state = {
            'status': 'connecting',
            'message': 'Connecting to printer',
            'reason': None,
        }
        self.send_status()

        self._actions.update({
            'cashbox': self.open_cashbox,
            'print_receipt': self.print_receipt,
            '': self._action_default,
        })

        self.receipt_protocol = 'escpos'

    @classmethod
    def supported(cls, device):
        # discard virtual printers (like "Microsoft Print to PDF") as they will trigger dialog boxes prompt
        return device['port'] != 'PORTPROMPT:'

    @classmethod
    def get_status(cls):
        status = 'connected' if any(iot_devices[d].device_type == "printer" and iot_devices[d].device_connection == 'direct' for d in iot_devices) else 'disconnected'
        return {'status': status, 'messages': ''}

    @staticmethod
    def _compute_device_connection(device):
        return 'direct' if device['port'].startswith(('USB', 'COM', 'LPT')) else 'network'

    def disconnect(self):
        self.update_status('disconnected', 'Printer was disconnected')
        super(PrinterDriver, self).disconnect()

    def update_status(self, status, message, reason=None):
        """Updates the state of the current printer.

        Args:
            status (str): The new value of the status
            message (str): A comprehensive message describing the status
            reason (str): The reason fo the current status
        """
        if self.state['status'] != status or self.state['reason'] != reason:
            self.state = {
                'status': status,
                'message': message,
                'reason': reason,
            }
            self.send_status()

    def send_status(self):
        """ Sends the current status of the printer to the connected Odoo instance.
        """
        self.data = {
            'value': '',
            'state': self.state,
        }
        event_manager.device_changed(self)

    def print_raw(self, data):
        win32print.StartDocPrinter(self.printer_handle, 1, ('', None, "RAW"))
        win32print.StartPagePrinter(self.printer_handle)
        win32print.WritePrinter(self.printer_handle, data)
        win32print.EndPagePrinter(self.printer_handle)
        win32print.EndDocPrinter(self.printer_handle)

    def print_report(self, data):
        helpers.write_file('document.pdf', data, 'wb')
        file_name = helpers.path_file('document.pdf')
        printer = self.device_name

        args = [
            "-dPrinted", "-dBATCH", "-dNOPAUSE", "-dNOPROMPT", "-dNORANGEPAGESIZE",
            "-q",
            "-sDEVICE#mswinpr2",
            f'-sOutputFile#%printer%{printer}',
            f'{file_name}'
        ]

        _logger.debug("Printing report with ghostscript using %s", args)
        stderr_buf = io.BytesIO()
        stdout_buf = io.BytesIO()
        stdout_log_level = logging.DEBUG
        try:
            ghostscript.Ghostscript(*args, stdout=stdout_buf, stderr=stderr_buf)
        except Exception:
            _logger.exception("Error while printing report, ghostscript args: %s, error buffer: %s", args, stderr_buf.getvalue())
            stdout_log_level = logging.ERROR # some stdout value might contains relevant error information
            raise
        finally:
            _logger.log(stdout_log_level, "Ghostscript stdout: %s", stdout_buf.getvalue())

    def print_receipt(self, data):
        _logger.debug("print_receipt called for printer %s", self.device_name)

        receipt = b64decode(data['receipt'])
        im = Image.open(io.BytesIO(receipt))

        # Convert to greyscale then to black and white
        im = im.convert("L")
        im = ImageOps.invert(im)
        im = im.convert("1")

        print_command = getattr(self, 'format_%s' % self.receipt_protocol)(im)
        self.print_raw(print_command)

    def format_escpos(self, im):
        width = int((im.width + 7) / 8)

        raster_send = b'\x1d\x76\x30\x00'
        max_slice_height = 255

        raster_data = b''
        dots = im.tobytes()
        while dots:
            im_slice = dots[:width*max_slice_height]
            slice_height = int(len(im_slice) / width)
            raster_data += raster_send + width.to_bytes(2, 'little') + slice_height.to_bytes(2, 'little') + im_slice
            dots = dots[width*max_slice_height:]

        return raster_data + RECEIPT_PRINTER_COMMANDS['escpos']['cut']

    def open_cashbox(self, data):
        """Sends a signal to the current printer to open the connected cashbox."""
        _logger.debug("open_cashbox called for printer %s", self.device_name)
        
        commands = RECEIPT_PRINTER_COMMANDS[self.receipt_protocol]
        for drawer in commands['drawers']:
            self.print_raw(drawer)

    def _action_default(self, data):
        _logger.debug("_action_default called for printer %s", self.device_name)

        document = b64decode(data['document'])
        mimetype = guess_mimetype(document)
        if mimetype == 'application/pdf':
            self.print_report(document)
        else:
            self.print_raw(document)
        _logger.debug("_action_default finished with mimetype %s for printer %s", mimetype, self.device_name)


proxy_drivers['printer'] = PrinterDriver

```

## File: iot_handlers\drivers\SerialBaseDriver.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import namedtuple
from contextlib import contextmanager
import logging
import serial
from threading import Lock
import time
import traceback

from odoo.addons.hw_drivers.event_manager import event_manager
from odoo.addons.hw_drivers.driver import Driver

_logger = logging.getLogger(__name__)

SerialProtocol = namedtuple(
    'SerialProtocol',
    "name baudrate bytesize stopbits parity timeout writeTimeout measureRegexp statusRegexp "
    "commandTerminator commandDelay measureDelay newMeasureDelay "
    "measureCommand emptyAnswerValid")


@contextmanager
def serial_connection(path, protocol, is_probing=False):
    """Opens a serial connection to a device and closes it automatically after use.

    :param path: path to the device
    :type path: string
    :param protocol: an object containing the serial protocol to connect to a device
    :type protocol: namedtuple
    :param is_probing: a flag thet if set to `True` makes the timeouts longer, defaults to False
    :type is_probing: bool, optional
    """

    PROBING_TIMEOUT = 1
    port_config = {
        'baudrate': protocol.baudrate,
        'bytesize': protocol.bytesize,
        'stopbits': protocol.stopbits,
        'parity': protocol.parity,
        'timeout': PROBING_TIMEOUT if is_probing else protocol.timeout,               # longer timeouts for probing
        'writeTimeout': PROBING_TIMEOUT if is_probing else protocol.writeTimeout      # longer timeouts for probing
    }
    connection = serial.Serial(path, **port_config)
    yield connection
    connection.close()


class SerialDriver(Driver):
    """Abstract base class for serial drivers."""

    _protocol = None
    connection_type = 'serial'

    STATUS_CONNECTED = 'connected'
    STATUS_ERROR = 'error'
    STATUS_CONNECTING = 'connecting'
    STATUS_DISCONNECTED = 'disconnected'

    def __init__(self, identifier, device):
        """ Attributes initialization method for `SerialDriver`.

        :param device: path to the device
        :type device: str
        """

        super(SerialDriver, self).__init__(identifier, device)
        self._actions.update({
            'get_status': self._push_status,
        })
        self.device_connection = 'serial'
        self._device_lock = Lock()
        self._status = {'status': self.STATUS_CONNECTING, 'message_title': '', 'message_body': ''}
        self._set_name()

    def _get_raw_response(connection):
        pass

    def _push_status(self):
        """Updates the current status and pushes it to the frontend."""

        self.data['status'] = self._status
        event_manager.device_changed(self)

    def _set_name(self):
        """Tries to build the device's name based on its type and protocol name but falls back on a default name if that doesn't work."""

        try:
            name = ('%s serial %s' % (self._protocol.name, self.device_type)).title()
        except Exception:
            name = 'Unknown Serial Device'
        self.device_name = name

    def _take_measure(self):
        pass

    def _do_action(self, data):
        """Helper function that calls a specific action method on the device.

        :param data: the `_actions` key mapped to the action method we want to call
        :type data: string
        """

        with self._device_lock:
            try:
                self._actions[data['action']](data)
                time.sleep(self._protocol.commandDelay)
            except Exception:
                msg = _(f'An error occurred while performing action "{data}" on "{self.device_name}"')
                _logger.exception(msg)
                self._status = {'status': self.STATUS_ERROR, 'message_title': msg, 'message_body': traceback.format_exc()}
                self._push_status()
            self._status = {'status': self.STATUS_CONNECTED, 'message_title': '', 'message_body': ''}
            self.data['status'] = self._status

    def action(self, data):
        """Establish a connection with the device if needed and have it perform a specific action.

        :param data: the `_actions` key mapped to the action method we want to call
        :type data: string
        """

        if self._connection and self._connection.isOpen():
            self._do_action(data)
        else:
            with serial_connection(self.device_identifier, self._protocol) as connection:
                self._connection = connection
                self._do_action(data)

    def run(self):
        """Continuously gets new measures from the device."""

        try:
            with serial_connection(self.device_identifier, self._protocol) as connection:
                self._connection = connection
                self._status['status'] = self.STATUS_CONNECTED
                self._push_status()
                while not self._stopped.is_set():
                    self._take_measure()
                    time.sleep(self._protocol.newMeasureDelay)
                self._status['status'] = self.STATUS_DISCONNECTED
                self._push_status()
        except Exception:
            msg = 'Error while reading %s' % self.device_name
            _logger.exception(msg)
            self._status = {'status': self.STATUS_ERROR, 'message_title': msg, 'message_body': traceback.format_exc()}
            self._push_status()

```

## File: iot_handlers\drivers\SerialScaleDriver.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import namedtuple
import logging
import re
import serial
import threading
import time

from odoo import http
from odoo.addons.hw_drivers.controllers.proxy import proxy_drivers
from odoo.addons.hw_drivers.event_manager import event_manager
from odoo.addons.hw_drivers.iot_handlers.drivers.SerialBaseDriver import SerialDriver, SerialProtocol, serial_connection


_logger = logging.getLogger(__name__)

# Only needed to ensure compatibility with older versions of Odoo
ACTIVE_SCALE = None
new_weight_event = threading.Event()

ScaleProtocol = namedtuple('ScaleProtocol', SerialProtocol._fields + ('zeroCommand', 'tareCommand', 'clearCommand', 'autoResetWeight'))

# 8217 Mettler-Toledo (Weight-only) Protocol, as described in the scale's Service Manual.
#    e.g. here: https://www.manualslib.com/manual/861274/Mettler-Toledo-Viva.html?page=51#manual
# Our recommended scale, the Mettler-Toledo "Ariva-S", supports this protocol on
# both the USB and RS232 ports, it can be configured in the setup menu as protocol option 3.
# We use the default serial protocol settings, the scale's settings can be configured in the
# scale's menu anyway.
Toledo8217Protocol = ScaleProtocol(
    name='Toledo 8217',
    baudrate=9600,
    bytesize=serial.SEVENBITS,
    stopbits=serial.STOPBITS_ONE,
    parity=serial.PARITY_EVEN,
    timeout=1,
    writeTimeout=1,
    measureRegexp=b"\x02\\s*([0-9.]+)N?\\r",
    statusRegexp=b"\x02\\s*(\\?.)\\r",
    commandDelay=0.2,
    measureDelay=0.5,
    newMeasureDelay=0.2,
    commandTerminator=b'',
    measureCommand=b'W',
    zeroCommand=b'Z',
    tareCommand=b'T',
    clearCommand=b'C',
    emptyAnswerValid=False,
    autoResetWeight=False,
)

# The ADAM scales have their own RS232 protocol, usually documented in the scale's manual
#   e.g at https://www.adamequipment.com/media/docs/Print%20Publications/Manuals/PDF/AZEXTRA/AZEXTRA-UM.pdf
#          https://www.manualslib.com/manual/879782/Adam-Equipment-Cbd-4.html?page=32#manual
# Only the baudrate and label format seem to be configurable in the AZExtra series.
ADAMEquipmentProtocol = ScaleProtocol(
    name='Adam Equipment',
    baudrate=4800,
    bytesize=serial.EIGHTBITS,
    stopbits=serial.STOPBITS_ONE,
    parity=serial.PARITY_NONE,
    timeout=0.2,
    writeTimeout=0.2,
    measureRegexp=rb"\s*([0-9.]+)kg",  # LABEL format 3 + KG in the scale settings, but Label 1/2 should work
    statusRegexp=None,
    commandTerminator=b"\r\n",
    commandDelay=0.2,
    measureDelay=0.5,
    # AZExtra beeps every time you ask for a weight that was previously returned!
    # Adding an extra delay gives the operator a chance to remove the products
    # before the scale starts beeping. Could not find a way to disable the beeps.
    newMeasureDelay=5,
    measureCommand=b'P',
    zeroCommand=b'Z',
    tareCommand=b'T',
    clearCommand=None,  # No clear command -> Tare again
    emptyAnswerValid=True,  # AZExtra does not answer unless a new non-zero weight has been detected
    autoResetWeight=True,  # AZExtra will not return 0 after removing products
)


# Ensures compatibility with older versions of Odoo
class ScaleReadOldRoute(http.Controller):
    @http.route('/hw_proxy/scale_read', type='json', auth='none', cors='*')
    def scale_read(self):
        if ACTIVE_SCALE:
            return {'weight': ACTIVE_SCALE._scale_read_old_route()}
        return None


class ScaleDriver(SerialDriver):
    """Abstract base class for scale drivers."""
    last_sent_value = None

    def __init__(self, identifier, device):
        super(ScaleDriver, self).__init__(identifier, device)
        self.device_type = 'scale'
        self._set_actions()
        self._is_reading = True

        # Ensures compatibility with older versions of Odoo
        # Only the last scale connected is kept
        global ACTIVE_SCALE
        ACTIVE_SCALE = self
        proxy_drivers['scale'] = ACTIVE_SCALE

    # Ensures compatibility with older versions of Odoo
    # and allows using the `ProxyDevice` in the point of sale to retrieve the status
    def get_status(self):
        """Allows `hw_proxy.Proxy` to retrieve the status of the scales"""

        status = self._status
        return {'status': status['status'], 'messages': [status['message_title'], ]}

    def _set_actions(self):
        """Initializes `self._actions`, a map of action keys sent by the frontend to backend action methods."""

        self._actions.update({
            'read_once': self._read_once_action,
            'set_zero': self._set_zero_action,
            'set_tare': self._set_tare_action,
            'clear_tare': self._clear_tare_action,
            'start_reading': self._start_reading_action,
            'stop_reading': self._stop_reading_action,
        })

    def _start_reading_action(self, data):
        """Starts asking for the scale value."""
        self._is_reading = True

    def _stop_reading_action(self, data):
        """Stops asking for the scale value."""
        self._is_reading = False

    def _clear_tare_action(self, data):
        """Clears the scale current tare weight."""

        # if the protocol has no clear tare command, we can just tare again
        clearCommand = self._protocol.clearCommand or self._protocol.tareCommand
        self._connection.write(clearCommand + self._protocol.commandTerminator)

    def _read_once_action(self, data):
        """Reads the scale current weight value and pushes it to the frontend."""

        self._read_weight()
        self.last_sent_value = self.data['value']
        event_manager.device_changed(self)

    def _set_zero_action(self, data):
        """Makes the weight currently applied to the scale the new zero."""

        self._connection.write(self._protocol.zeroCommand + self._protocol.commandTerminator)

    def _set_tare_action(self, data):
        """Sets the scale's current weight value as tare weight."""

        self._connection.write(self._protocol.tareCommand + self._protocol.commandTerminator)

    @staticmethod
    def _get_raw_response(connection):
        """Gets raw bytes containing the updated value of the device.

        :param connection: a connection to the device's serial port
        :type connection: pyserial.Serial
        :return: the raw response to a weight request
        :rtype: str
        """

        answer = []
        while True:
            char = connection.read(1)
            if not char:
                break
            else:
                answer.append(bytes(char))
        return b''.join(answer)

    def _read_weight(self):
        """Asks for a new weight from the scale, checks if it is valid and, if it is, makes it the current value."""

        protocol = self._protocol
        self._connection.write(protocol.measureCommand + protocol.commandTerminator)
        answer = self._get_raw_response(self._connection)
        match = re.search(self._protocol.measureRegexp, answer)
        if match:
            self.data = {
                'value': float(match.group(1)),
                'status': self._status
            }

    # Ensures compatibility with older versions of Odoo
    def _scale_read_old_route(self):
        """Used when the iot app is not installed"""
        with self._device_lock:
            self._read_weight()
        return self.data['value']

    def _take_measure(self):
        """Reads the device's weight value, and pushes that value to the frontend."""

        with self._device_lock:
            self._read_weight()
            if self.data['value'] != self.last_sent_value or self._status['status'] == self.STATUS_ERROR:
                self.last_sent_value = self.data['value']
                event_manager.device_changed(self)


class Toledo8217Driver(ScaleDriver):
    """Driver for the Toldedo 8217 serial scale."""
    _protocol = Toledo8217Protocol

    def __init__(self, identifier, device):
        super(Toledo8217Driver, self).__init__(identifier, device)
        self.device_manufacturer = 'Toledo'

    @classmethod
    def supported(cls, device):
        """Checks whether the device, which port info is passed as argument, is supported by the driver.

        :param device: path to the device
        :type device: str
        :return: whether the device is supported by the driver
        :rtype: bool
        """

        protocol = cls._protocol

        try:
            with serial_connection(device['identifier'], protocol, is_probing=True) as connection:
                connection.write(b'Ehello' + protocol.commandTerminator)
                time.sleep(protocol.commandDelay)
                answer = connection.read(8)
                if answer == b'\x02E\rhello':
                    connection.write(b'F' + protocol.commandTerminator)
                    return True
        except serial.serialutil.SerialTimeoutException:
            pass
        except Exception:
            _logger.exception('Error while probing %s with protocol %s' % (device, protocol.name))
        return False


class AdamEquipmentDriver(ScaleDriver):
    """Driver for the Adam Equipment serial scale."""

    _protocol = ADAMEquipmentProtocol
    priority = 0  # Test the supported method of this driver last, after all other serial drivers

    def __init__(self, identifier, device):
        super(AdamEquipmentDriver, self).__init__(identifier, device)
        self._is_reading = False
        self._last_weight_time = 0
        self.device_manufacturer = 'Adam'

    def _check_last_weight_time(self):
        """The ADAM doesn't make the difference between a value of 0 and "the same value as last time":
        in both cases it returns an empty string.
        With this, unless the weight changes, we give the user `TIME_WEIGHT_KEPT` seconds to log the new weight,
        then change it back to zero to avoid keeping it indefinetely, which could cause issues.
        In any case the ADAM must always go back to zero before it can weight again.
        """

        TIME_WEIGHT_KEPT = 10

        if self.data['value'] is None:
            if time.time() - self._last_weight_time > TIME_WEIGHT_KEPT:
                self.data['value'] = 0
        else:
            self._last_weight_time = time.time()

    def _take_measure(self):
        """Reads the device's weight value, and pushes that value to the frontend."""

        if self._is_reading:
            with self._device_lock:
                self._read_weight()
                self._check_last_weight_time()
                if self.data['value'] != self.last_sent_value or self._status['status'] == self.STATUS_ERROR:
                    self.last_sent_value = self.data['value']
                    event_manager.device_changed(self)
        else:
            time.sleep(0.5)

    # Ensures compatibility with older versions of Odoo
    def _scale_read_old_route(self):
        """Used when the iot app is not installed"""

        time.sleep(3)
        with self._device_lock:
            self._read_weight()
            self._check_last_weight_time()
        return self.data['value']

    @classmethod
    def supported(cls, device):
        """Checks whether the device at `device` is supported by the driver.

        :param device: path to the device
        :type device: str
        :return: whether the device is supported by the driver
        :rtype: bool
        """

        protocol = cls._protocol

        try:
            with serial_connection(device['identifier'], protocol, is_probing=True) as connection:
                connection.write(protocol.measureCommand + protocol.commandTerminator)
                # Checking whether writing to the serial port using the Adam protocol raises a timeout exception is about the only thing we can do.
                return True
        except serial.serialutil.SerialTimeoutException:
            pass
        except Exception:
            _logger.exception('Error while probing %s with protocol %s' % (device, protocol.name))
        return False

```

## File: iot_handlers\interfaces\DisplayInterface_L.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import os
import subprocess

try:
    import screeninfo
except ImportError:
    screeninfo = None
    import RPi.GPIO as GPIO
    from vcgencmd import Vcgencmd

from odoo.addons.hw_drivers.interface import Interface

_logger = logging.getLogger(__name__)


class DisplayInterface(Interface):
    _loop_delay = 0
    connection_type = 'display'

    def get_devices(self):
        display_devices = {}
        dummy_display = {
            'distant_display': {
                'identifier': 'distant_display',
                'name': 'Distant Display',
            }
        }

        if screeninfo is None:
            # On IoT image < 24.10 we don't have screeninfo installed, so we can't get the connected displays
            # We use old method to get the connected display
            hdmi_ports = {'hdmi_0': 2, 'hdmi_1': 7} if 'Pi 4' in GPIO.RPI_INFO.get('TYPE') else {'hdmi_0': 2}
            try:
                for x_screen, port in enumerate(hdmi_ports):
                    if Vcgencmd().display_power_state(hdmi_ports.get(port)) == 'on':
                        display_devices[port] = self._add_device(port, x_screen)
            except subprocess.CalledProcessError:
                _logger.warning('Vcgencmd "display_power_state" method call failed')

            return display_devices or dummy_display

        try:
            os.environ['DISPLAY'] = ':0'
            for x_screen, monitor in enumerate(screeninfo.get_monitors()):
                display_devices[monitor.name] = self._add_device(monitor.name, x_screen)
            return display_devices or dummy_display
        except screeninfo.common.ScreenInfoError:
            # If no display is connected, screeninfo raises an error, we return the distant display
            return dummy_display

    @classmethod
    def _add_device(cls, display_identifier, x_screen):
        """Creates a display_device dict.

        :param display_identifier: the identifier of the display
        :param x_screen: the x screen number
        :return: the display device dict
        """

        return {
            'identifier': display_identifier,
            'name': 'Display - ' + display_identifier,
            'x_screen': str(x_screen),
        }

```

## File: iot_handlers\interfaces\PrinterInterface_L.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from cups import Connection as CupsConnection
from re import sub
from threading import Lock

from odoo.addons.hw_drivers.interface import Interface

conn = CupsConnection()
PPDs = conn.getPPDs()
cups_lock = Lock()  # We can only make one call to Cups at a time


class PrinterInterface(Interface):
    _loop_delay = 120
    connection_type = 'printer'
    printer_devices = {}

    def get_devices(self):
        discovered_devices = {}
        with cups_lock:
            printers = conn.getPrinters()
            devices = conn.getDevices()
            for printer_name, printer in printers.items():
                path = printer.get('device-uri', False)
                if printer_name != self.get_identifier(path):
                    device_class = 'direct' if 'usb' in printer.get('device-uri') else 'network'
                    printer.update({
                        'supported': True,
                        'device-class': device_class,
                        'device-make-and-model': printer_name,  # give name set in Cups
                        'device-id': '',
                    })
                    devices.update({printer_name: printer})

        for path, device in devices.items():
            identifier = self.get_identifier(path)
            device.update({
                'identifier': identifier,
                'url': path,
                'disconnect_counter': 0,
            })
            discovered_devices.update({identifier: device})
        self.printer_devices.update(discovered_devices)
        # Deal with devices which are on the list but were not found during this call of "get_devices"
        # If they aren't detected 3 times consecutively, remove them from the list of available devices
        for device in list(self.printer_devices):
            if not discovered_devices.get(device):
                disconnect_counter = self.printer_devices.get(device).get('disconnect_counter')
                if disconnect_counter >= 2:
                    self.printer_devices.pop(device, None)
                else:
                    self.printer_devices[device].update({'disconnect_counter': disconnect_counter + 1})
        return dict(self.printer_devices)

    def get_identifier(self, path):
        """
        Necessary because the path is not always a valid Cups identifier,
        as it may contain characters typically found in URLs or paths.

          - Removes characters: ':', '/', '.', '\', and space.
          - Removes the exact strings: "uuid=" and "serial=".

        Example 1:
            Input: "ipp://printers/printer1:1234/abcd"
            Output: "ippprintersprinter11234abcd"

        Example 2:
            Input: "uuid=1234-5678-90ab-cdef"
            Output: "1234-5678-90ab-cdef
        """
        return sub(r'[:\/\.\\ ]|(uuid=)|(serial=)', '', path)

```

## File: iot_handlers\interfaces\PrinterInterface_W.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import win32print

from odoo.addons.hw_drivers.interface import Interface

_logger = logging.getLogger(__name__)


class PrinterInterface(Interface):
    _loop_delay = 30
    connection_type = 'printer'

    def get_devices(self):
        printer_devices = {}
        printers = win32print.EnumPrinters(win32print.PRINTER_ENUM_LOCAL)

        for printer in printers:
            identifier = printer[2]
            handle_printer = win32print.OpenPrinter(identifier)
            # The value "2" is the level of detail we want to get from the printer, see:
            # https://learn.microsoft.com/en-us/windows/win32/printdocs/getprinter#parameters
            printer_details = win32print.GetPrinter(handle_printer, 2)
            printer_port = None
            if printer_details:
                # see: https://learn.microsoft.com/en-us/windows/win32/printdocs/printer-info-2#members
                printer_port = printer_details.get('pPortName')
            if printer_port is None:
                _logger.warning('Printer "%s" has no port name. Used dummy port', identifier)
                printer_port = 'IOT_DUMMY_PORT'

            printer_devices[identifier] = {
                'identifier': identifier,
                'printer_handle': handle_printer,
                'port': printer_port,
            }
        return printer_devices

```

## File: iot_handlers\interfaces\SerialInterface.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import platform
from serial.tools.list_ports import comports

from odoo.addons.hw_drivers.interface import Interface


class SerialInterface(Interface):
    connection_type = 'serial'

    def get_devices(self):
        serial_devices = {
            port.device: {'identifier': port.device}
            for port in comports()
            if platform.system() == 'Windows' or port.subsystem != 'amba'
            # RPI 5 uses ttyAMA10 as a console serial port for system messages: odoo interprets it as scale -> avoid it
        }
        return serial_devices

```

## File: iot_handlers\interfaces\USBInterface_L.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from usb import core

from odoo.addons.hw_drivers.interface import Interface


class USBInterface(Interface):
    connection_type = 'usb'

    def get_devices(self):
        """
        USB devices are identified by a combination of their `idVendor` and
        `idProduct`. We can't be sure this combination in unique per equipment.
        To still allow connecting multiple similar equipments, we complete the
        identifier by a counter. The drawbacks are we can't be sure the equipments
        will get the same identifiers after a reboot or a disconnect/reconnect.
        """
        usb_devices = {}
        devs = core.find(find_all=True)
        cpt = 2
        for dev in devs:
            identifier = "usb_%04x:%04x" % (dev.idVendor, dev.idProduct)
            if identifier in usb_devices:
                identifier += '_%s' % cpt
                cpt += 1
            usb_devices[identifier] = dev
        return usb_devices

```

## File: static\src\js\worker.js

```javascript
/* global display_identifier */
    $(function() {
        "use strict";
        // mergedHead will be turned to true the first time we receive something from a new host
        // It allows to transform the <head> only once
        var mergedHead = false;
        var current_client_url = "";

        function longpolling() {
            $.ajax({
                type: 'POST',
                url: window.location.origin + '/point_of_sale/get_serialized_order/' + display_identifier,
                dataType: 'json',
                beforeSend: function(xhr){xhr.setRequestHeader('Content-Type', 'application/json');},
                data: JSON.stringify({jsonrpc: '2.0'}),

                success: function(data) {
                    if (data.result.error) {
                        $('.error-message').text(data.result.error);
                        $('.error-message').removeClass('d-none');
                        setTimeout(longpolling, 5000);
                        return;
                    }
                    if (data.result.rendered_html) {
                        var trimmed = $.trim(data.result.rendered_html);
                        var $parsedHTML = $('<div>').html($.parseHTML(trimmed,true)); // WARNING: the true here will executes any script present in the string to parse
                        var new_client_url = $parsedHTML.find(".resources > base").attr('href');

                        if (!mergedHead || (current_client_url !== new_client_url)) {

                            mergedHead = true;
                            current_client_url = new_client_url;
                            $("head").children().not('.origin').remove();
                            $("head").append($parsedHTML.find(".resources").html());
                        }

                        $(".container-fluid").html($parsedHTML.find('.pos-customer_facing_display').html());
                        $(".container-fluid").attr('class', 'container-fluid').addClass($parsedHTML.find('.pos-customer_facing_display').attr('class'));

                        var d = $('.pos_orderlines_list');
                        d.scrollTop(d.prop("scrollHeight"));

                        // Here we execute the code coming from the pos, apparently $.parseHTML() executes scripts right away,
                        // Since we modify the dom afterwards, the script might not have any effect
                        /* eslint-disable no-undef */
                        if (typeof foreign_js !== 'undefined' && $.isFunction(foreign_js)) {
                            foreign_js();
                        }
                        /* eslint-enable no-undef */
                    }
                    longpolling();
                },

                error: function (jqXHR, status, err) {
                    setTimeout(longpolling, 5000);
                },

                timeout: 30000,
            });
        };

        longpolling();
    });

```

## File: tools\helpers.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import configparser
import contextlib
import datetime
from enum import Enum
from functools import cache
from importlib import util
import io
import json
import logging
import netifaces
from OpenSSL import crypto
import os
from pathlib import Path
import platform
import requests
import secrets
import socket
import subprocess
from urllib.parse import parse_qs
import urllib3
from threading import Thread, Lock
import time
import zipfile

from odoo import http, release, service
from odoo.tools.func import lazy_property
from odoo.tools.misc import file_path

lock = Lock()
_logger = logging.getLogger(__name__)

try:
    import crypt
except ImportError:
    _logger.warning('Could not import library crypt')

#----------------------------------------------------------
# Helper
#----------------------------------------------------------


class CertificateStatus(Enum):
    OK = 1
    NEED_REFRESH = 2
    ERROR = 3


class IoTRestart(Thread):
    """
    Thread to restart odoo server in IoT Box when we must return a answer before
    """
    def __init__(self, delay):
        Thread.__init__(self)
        self.delay = delay

    def run(self):
        time.sleep(self.delay)
        service.server.restart()


if platform.system() == 'Windows':
    writable = contextlib.nullcontext
elif platform.system() == 'Linux':
    @contextlib.contextmanager
    def writable():
        with lock:
            try:
                subprocess.run(["sudo", "mount", "-o", "remount,rw", "/"], check=False)
                subprocess.run(["sudo", "mount", "-o", "remount,rw", "/root_bypass_ramdisks/"], check=False)
                yield
            finally:
                subprocess.run(["sudo", "mount", "-o", "remount,ro", "/"], check=False)
                subprocess.run(["sudo", "mount", "-o", "remount,ro", "/root_bypass_ramdisks/"], check=False)
                subprocess.run(["sudo", "mount", "-o", "remount,rw", "/root_bypass_ramdisks/etc/cups"], check=False)

def access_point():
    return get_ip() == '10.11.12.1'

def start_nginx_server():
    if platform.system() == 'Windows':
        path_nginx = get_path_nginx()
        if path_nginx:
            os.chdir(path_nginx)
            _logger.info('Start Nginx server: %s\\nginx.exe', path_nginx)
            os.popen('nginx.exe')
            os.chdir('..\\server')
    elif platform.system() == 'Linux':
        subprocess.check_call(["sudo", "service", "nginx", "restart"])

def check_certificate():
    """
    Check if the current certificate is up to date or not authenticated
    :return CheckCertificateStatus
    """
    server = get_odoo_server_url()

    if not server:
        _logger.info('Ignoring the nginx certificate check without a connected database')
        return {"status": CertificateStatus.ERROR,
                "error_code": "ERR_IOT_HTTPS_CHECK_NO_SERVER"}

    if platform.system() == 'Windows':
        path = Path(get_path_nginx()).joinpath('conf/nginx-cert.crt')
    elif platform.system() == 'Linux':
        path = Path('/etc/ssl/certs/nginx-cert.crt')

    if not path.exists():
        return {"status": CertificateStatus.NEED_REFRESH}

    try:
        with path.open('r') as f:
            cert = crypto.load_certificate(crypto.FILETYPE_PEM, f.read())
    except EnvironmentError:
        _logger.exception("Unable to read certificate file")
        return {"status": CertificateStatus.ERROR,
                "error_code": "ERR_IOT_HTTPS_CHECK_CERT_READ_EXCEPTION"}

    cert_end_date = datetime.datetime.strptime(cert.get_notAfter().decode('utf-8'), "%Y%m%d%H%M%SZ") - datetime.timedelta(days=10)
    for key in cert.get_subject().get_components():
        if key[0] == b'CN':
            cn = key[1].decode('utf-8')
    if cn == 'OdooTempIoTBoxCertificate' or datetime.datetime.now() > cert_end_date:
        message = 'Your certificate %s must be updated' % cn
        _logger.info(message)
        return {"status": CertificateStatus.NEED_REFRESH}
    else:
        message = 'Your certificate %s is valid until %s' % (cn, cert_end_date)
        _logger.info(message)
        return {"status": CertificateStatus.OK, "message": message}


def check_git_branch():
    """
    Check if the local branch is the same than the connected Odoo DB and
    checkout to match it if needed.
    """
    server = get_odoo_server_url()
    urllib3.disable_warnings()
    http = urllib3.PoolManager(cert_reqs='CERT_NONE')
    try:
        response = http.request(
            'POST',
            server + "/web/webclient/version_info",
            body='{}',
            headers={'Content-type': 'application/json'},
        )

        if response.status == 200:
            git = ['git', '--work-tree=/home/pi/odoo/', '--git-dir=/home/pi/odoo/.git']

            db_branch = json.loads(response.data)['result']['server_serie'].replace('~', '-')
            if not subprocess.check_output(git + ['ls-remote', 'origin', db_branch]):
                db_branch = 'master'

            local_branch = (
                subprocess.check_output(git + ['symbolic-ref', '-q', '--short', 'HEAD']).decode('utf-8').rstrip()
            )
            _logger.info(
                "Current IoT Box local git branch: %s / Associated Odoo database's git branch: %s",
                local_branch,
                db_branch,
            )

            if db_branch != local_branch:
                try:
                    with writable():
                        subprocess.run(git + ['branch', '-m', db_branch], check=True)
                        subprocess.run(git + ['remote', 'set-branches', 'origin', db_branch], check=True)
                        _logger.info("Updating odoo folder to the branch %s", db_branch)
                        subprocess.run(
                            ['/home/pi/odoo/addons/point_of_sale/tools/posbox/configuration/posbox_update.sh'], check=True
                        )
                except subprocess.CalledProcessError:
                    _logger.exception("Failed to update the code with git.")
                finally:
                    odoo_restart()
    except Exception:
        _logger.exception('An error occurred while trying to update the code with git')

def check_image():
    """
    Check if the current image of IoT Box is up to date
    """
    url = 'https://nightly.odoo.com/master/iotbox/SHA1SUMS.txt'
    urllib3.disable_warnings()
    http = urllib3.PoolManager(cert_reqs='CERT_NONE')
    response = http.request('GET', url)
    checkFile = {}
    valueActual = ''
    for line in response.data.decode().split('\n'):
        if line:
            value, name = line.split('  ')
            checkFile.update({value: name})
            if name == 'iotbox-latest.zip':
                valueLastest = value
            elif name == get_img_name():
                valueActual = value
    if valueActual == valueLastest:
        return False
    version = checkFile.get(valueLastest, 'Error').replace('iotboxv', '').replace('.zip', '').split('_')
    return {'major': version[0], 'minor': version[1]}


def save_conf_server(url, token, db_uuid, enterprise_code):
    """
    Save server configurations in odoo.conf
    :param url: The URL of the server
    :param token: The token to authenticate the server
    :param db_uuid: The database UUID
    :param enterprise_code: The enterprise code
    """
    update_conf({
        'remote_server': url,
        'token': token,
        'db_uuid': db_uuid,
        'enterprise_code': enterprise_code,
    })


def generate_password():
    """
    Generate an unique code to secure raspberry pi
    """
    alphabet = 'abcdefghijkmnpqrstuvwxyz23456789'
    password = ''.join(secrets.choice(alphabet) for i in range(12))
    try:
        shadow_password = crypt.crypt(password, crypt.mksalt())
        subprocess.run(('sudo', 'usermod', '-p', shadow_password, 'pi'), check=True)
        with writable():
            subprocess.run(('sudo', 'cp', '/etc/shadow', '/root_bypass_ramdisks/etc/shadow'), check=True)
        return password
    except subprocess.CalledProcessError as e:
        _logger.exception("Failed to generate password: %s", e.output)
        return 'Error: Check IoT log'


def get_certificate_status(is_first=True):
    """
    Will get the HTTPS certificate details if present. Will load the certificate if missing.

    :param is_first: Use to make sure that the recursion happens only once
    :return: (bool, str)
    """
    check_certificate_result = check_certificate()
    certificateStatus = check_certificate_result["status"]

    if certificateStatus == CertificateStatus.ERROR:
        return False, check_certificate_result["error_code"]

    if certificateStatus == CertificateStatus.NEED_REFRESH and is_first:
        certificate_process = load_certificate()
        if certificate_process is not True:
            return False, certificate_process
        return get_certificate_status(is_first=False)  # recursive call to attempt certificate read
    return True, check_certificate_result.get("message",
                                              "The HTTPS certificate was generated correctly")

def get_img_name():
    major, minor = get_version()[1:].split('.')
    return 'iotboxv%s_%s.zip' % (major, minor)

def get_ip():
    interfaces = netifaces.interfaces()
    for interface in interfaces:
        if netifaces.ifaddresses(interface).get(netifaces.AF_INET):
            addr = netifaces.ifaddresses(interface).get(netifaces.AF_INET)[0]['addr']
            if addr != '127.0.0.1':
                return addr

def get_mac_address():
    interfaces = netifaces.interfaces()
    for interface in interfaces:
        if netifaces.ifaddresses(interface).get(netifaces.AF_INET):
            addr = netifaces.ifaddresses(interface).get(netifaces.AF_LINK)[0]['addr']
            if addr != '00:00:00:00:00:00':
                return addr

def get_path_nginx():
    return str(list(Path().absolute().parent.glob('*nginx*'))[0])

def get_ssid():
    ap = subprocess.call(['systemctl', 'is-active', '--quiet', 'hostapd']) # if service is active return 0 else inactive
    if not ap:
        return subprocess.check_output(['grep', '-oP', '(?<=ssid=).*', '/etc/hostapd/hostapd.conf']).decode('utf-8').rstrip()
    process_iwconfig = subprocess.Popen(['iwconfig'], stdout=subprocess.PIPE, stderr=subprocess.STDOUT)
    process_grep = subprocess.Popen(['grep', 'ESSID:"'], stdin=process_iwconfig.stdout, stdout=subprocess.PIPE)
    return subprocess.check_output(['sed', 's/.*"\\(.*\\)"/\\1/'], stdin=process_grep.stdout).decode('utf-8').rstrip()


@cache
def get_odoo_server_url():
    """Get the URL of the linked Odoo database.
    If the IoT Box is in access point mode, it will return ``None`` to avoid
    connecting to the server.

    :return: The URL of the linked Odoo database.
    :rtype: str or None
    """
    return None if access_point() else get_conf('remote_server')


def get_token():
    """:return: The token to authenticate the server"""
    return get_conf('token')


def get_commit_hash():
    return subprocess.run(
        ['git', '--work-tree=/home/pi/odoo/', '--git-dir=/home/pi/odoo/.git', 'rev-parse', '--short', 'HEAD'],
        stdout=subprocess.PIPE,
        check=True,
    ).stdout.decode('ascii').strip()


@cache
def get_version(detailed_version=False):
    if platform.system() == 'Linux':
        image_version = read_file_first_line('/var/odoo/iotbox_version')
    elif platform.system() == 'Windows':
        # updated manually when big changes are made to the windows virtual IoT
        image_version = '23.11'

    version = platform.system()[0] + image_version
    if detailed_version:
        # Note: on windows IoT, the `release.version` finish with the build date
        version += f"-{release.version}"
        if platform.system() == 'Linux':
            version += f'#{get_commit_hash()}'
    return version

def get_wifi_essid():
    wifi_options = []
    process_iwlist = subprocess.Popen(['sudo', 'iwlist', 'wlan0', 'scan'], stdout=subprocess.PIPE, stderr=subprocess.STDOUT)
    process_grep = subprocess.Popen(['grep', 'ESSID:"'], stdin=process_iwlist.stdout, stdout=subprocess.PIPE).stdout.readlines()
    for ssid in process_grep:
        essid = ssid.decode('utf-8').split('"')[1]
        if essid not in wifi_options:
            wifi_options.append(essid)
    return wifi_options


def load_certificate():
    """
    Send a request to Odoo with customer db_uuid and enterprise_code to get a true certificate
    """
    db_uuid = get_conf('db_uuid')
    enterprise_code = get_conf('enterprise_code')
    if not db_uuid:
        return "ERR_IOT_HTTPS_LOAD_NO_CREDENTIAL"

    url = 'https://www.odoo.com/odoo-enterprise/iot/x509'
    data = {
        'params': {
            'db_uuid': db_uuid,
            'enterprise_code': enterprise_code or ''
        }
    }
    urllib3.disable_warnings()
    http = urllib3.PoolManager(cert_reqs='CERT_NONE', retries=urllib3.Retry(4))
    try:
        response = http.request(
            'POST',
            url,
            body = json.dumps(data).encode('utf8'),
            headers = {'Content-type': 'application/json', 'Accept': 'text/plain'}
        )
    except Exception as e:
        _logger.exception("An error occurred while trying to reach odoo.com servers.")
        return "ERR_IOT_HTTPS_LOAD_REQUEST_EXCEPTION\n\n%s" % e

    if response.status != 200:
        return "ERR_IOT_HTTPS_LOAD_REQUEST_STATUS %s\n\n%s" % (response.status, response.reason)

    result = json.loads(response.data.decode()).get('result', {})
    error = result.get('error')
    if error:
        _logger.error("An error received from odoo.com while trying to get the certificate: %s", error)
        return "ERR_IOT_HTTPS_LOAD_REQUEST_NO_RESULT"

    update_conf({'subject': result['subject_cn']})
    if platform.system() == 'Linux':
        with writable():
            Path('/etc/ssl/certs/nginx-cert.crt').write_text(result['x509_pem'])
            Path('/root_bypass_ramdisks/etc/ssl/certs/nginx-cert.crt').write_text(result['x509_pem'])
            Path('/etc/ssl/private/nginx-cert.key').write_text(result['private_key_pem'])
            Path('/root_bypass_ramdisks/etc/ssl/private/nginx-cert.key').write_text(result['private_key_pem'])
    elif platform.system() == 'Windows':
        Path(get_path_nginx()).joinpath('conf/nginx-cert.crt').write_text(result['x509_pem'])
        Path(get_path_nginx()).joinpath('conf/nginx-cert.key').write_text(result['private_key_pem'])
    time.sleep(3)
    if platform.system() == 'Windows':
        odoo_restart(0)
    elif platform.system() == 'Linux':
        start_nginx_server()
    return True


def delete_iot_handlers():
    """Delete all drivers, interfaces and libs if any.
    This is needed to avoid conflicts with the newly downloaded drivers.
    """
    try:
        iot_handlers = Path(file_path(f'hw_drivers/iot_handlers'))
        filenames = [
            f"odoo/addons/hw_drivers/iot_handlers/{file.relative_to(iot_handlers)}"
            for file in iot_handlers.glob('**/*')
            if file.is_file()
        ]
        unlink_file(*filenames)
        _logger.info("Deleted old IoT handlers")
    except OSError:
        _logger.exception('Failed to delete old IoT handlers')

def download_iot_handlers(auto=True):
    """
    Get the drivers from the configured Odoo server
    """
    server = get_odoo_server_url()
    if server:
        urllib3.disable_warnings()
        pm = urllib3.PoolManager(cert_reqs='CERT_NONE')
        server = server + '/iot/get_handlers'
        try:
            resp = pm.request('POST', server, fields={'mac': get_mac_address(), 'auto': auto}, timeout=8)
            if resp.data:
                delete_iot_handlers()
                with writable():
                    path = path_file('odoo', 'addons', 'hw_drivers', 'iot_handlers')
                    zip_file = zipfile.ZipFile(io.BytesIO(resp.data))
                    zip_file.extractall(path)
        except Exception:
            _logger.exception('Could not reach configured server to download IoT handlers')

def compute_iot_handlers_addon_name(handler_kind, handler_file_name):
    return "odoo.addons.hw_drivers.iot_handlers.{handler_kind}.{handler_name}".\
        format(handler_kind=handler_kind, handler_name=handler_file_name.removesuffix('.py'))

def load_iot_handlers():
    """
    This method loads local files: 'odoo/addons/hw_drivers/iot_handlers/drivers' and
    'odoo/addons/hw_drivers/iot_handlers/interfaces'
    And execute these python drivers and interfaces
    """
    for directory in ['interfaces', 'drivers']:
        path = file_path(f'hw_drivers/iot_handlers/{directory}')
        filesList = list_file_by_os(path)
        for file in filesList:
            spec = util.spec_from_file_location(compute_iot_handlers_addon_name(directory, file), str(Path(path).joinpath(file)))
            if spec:
                module = util.module_from_spec(spec)
                try:
                    spec.loader.exec_module(module)
                except Exception:
                    _logger.exception('Unable to load handler file: %s', file)
    lazy_property.reset_all(http.root)

def list_file_by_os(file_list):
    platform_os = platform.system()
    if platform_os == 'Linux':
        return [x.name for x in Path(file_list).glob('*[!W].*')]
    elif platform_os == 'Windows':
        return [x.name for x in Path(file_list).glob('*[!L].*')]


def odoo_restart(delay=0):
    """
    Restart Odoo service
    :param delay: Delay in seconds before restarting the service (Default: 0)
    """
    IR = IoTRestart(delay)
    IR.start()


def path_file(*args):
    """Return the path to the file from IoT Box root or Windows Odoo
    server folder
    :return: The path to the file
    """
    platform_os = platform.system()
    if platform_os == 'Linux':
        return Path("~pi", *args).expanduser() # Path.home() returns odoo user's home instead of pi's
    elif platform_os == 'Windows':
        return Path().absolute().parent.joinpath('server', *args)


def read_file_first_line(filename):
    path = path_file(filename)
    if path.exists():
        with path.open('r') as f:
            return f.readline().strip('\n')


def unlink_file(*filenames):
    with writable():
        for filename in filenames:
            path = path_file(filename)
            if path.exists():
                path.unlink()


def write_file(filename, text, mode='w'):
    """
    This function writes 'text' to 'filename' file for classic files.
    :param filename: The name of the file to write to.
    :param text: The text to write to the file, OR the ConfigParser object to write to the file.
    :param mode: The mode to open the file in (Default: 'w').
    """
    with writable():
        path = path_file(filename)
        with open(path, mode) as f:
            if path.suffix == '.conf' and isinstance(text, configparser.ConfigParser):
                # As we are dealing with conf files, :filename: is a Path object, as it was created by path_file for
                # configparser. More, :text: is assumed to be a ConfigParser object.
                text.write(f)
            else:
                f.write(text)


def download_from_url(download_url, path_to_filename):
    """
    This function downloads from its 'download_url' argument and
    saves the result in 'path_to_filename' file
    The 'path_to_filename' needs to be a valid path + file name
    (Example: 'C:\\Program Files\\Odoo\\downloaded_file.zip')
    """
    try:
        request_response = requests.get(download_url, timeout=60)
        request_response.raise_for_status()
        write_file(path_to_filename, request_response.content, 'wb')
        _logger.info('Downloaded %s from %s', path_to_filename, download_url)
    except Exception:
        _logger.exception('Failed to download from %s', download_url)

def unzip_file(path_to_filename, path_to_extract):
    """
    This function unzips 'path_to_filename' argument to
    the path specified by 'path_to_extract' argument
    and deletes the originally used .zip file
    Example: unzip_file('C:\\Program Files\\Odoo\\downloaded_file.zip', 'C:\\Program Files\\Odoo\\new_folder'))
    Will extract all the contents of 'downloaded_file.zip' to the 'new_folder' location)
    """
    try:
        with writable():
            path = path_file(path_to_filename)
            with zipfile.ZipFile(path) as zip_file:
                zip_file.extractall(path_file(path_to_extract))
            Path(path).unlink()
        _logger.info('Unzipped %s to %s', path_to_filename, path_to_extract)
    except Exception:
        _logger.exception('Failed to unzip %s', path_to_filename)


@cache
def get_hostname():
    """Cache the hostname to avoid multiple calls to socket.gethostname()"""
    return socket.gethostname()


def update_conf(values, section='iot.box'):
    """
    Update odoo.conf with the given key and value.
    :param values: The dictionary of key-value pairs to update the config with.
    :param section: The section to update the key-value pairs in (Default: iot.box).
    """
    _logger.debug("Updating odoo.conf with values: %s", values)
    conf = get_conf()
    get_conf.cache_clear()  # Clear the cache to get the updated config

    if not conf.has_section(section):
        _logger.debug("Creating new section '%s' in odoo.conf", section)
        conf.add_section(section)

    for key, value in values.items():
        conf.set(section, key, value) if value else conf.remove_option(section, key)

    write_file("odoo.conf", conf)


@cache
def get_conf(key=None, section='iot.box'):
    """
    Get the value of the given key from odoo.conf, or the full config if no key is provided.
    :param key: The key to get the value of.
    :param section: The section to get the key from (Default: iot.box).
    :return: The value of the key provided or None if it doesn't exist, or full conf object if no key is provided.
    """
    conf = configparser.ConfigParser()
    conf.read(path_file("odoo.conf"))

    return conf.get(section, key, fallback=None) if key else conf  # Return the key's value or the configparser object


def disconnect_from_server():
    """Disconnect the IoT Box from the server, clears associated caches"""
    get_odoo_server_url.cache_clear()
    update_conf({
        'remote_server': '',
        'token': '',
        'db_uuid': '',
    })


def migrate_old_config_files_to_new_config_file():
    """Migrate old config files to the new odoo.conf"""
    if not get_conf().has_section('iot.box'):
        _logger.info('Migrating old config files to the new odoo.conf')
        iotbox_version = read_file_first_line('/var/odoo/iotbox_version')
        db_uuid = read_file_first_line('odoo-db-uuid.conf')
        enterprise_code = read_file_first_line('odoo-enterprise-code.conf')
        remote_server = read_file_first_line('odoo-remote-server.conf')
        token = read_file_first_line('token')
        subject = read_file_first_line('odoo-subject.conf')

        update_conf({
            'iotbox_version': iotbox_version,
            'remote_server': remote_server,
            'token': token,
            'db_uuid': db_uuid,
            'enterprise_code': enterprise_code,
            'subject': subject,
        })

        if platform.system() == 'Linux':
            wifi_network_path = path_file('wifi_network.txt')
            if wifi_network_path.exists():
                with open(wifi_network_path, encoding="utf-8") as f:
                    wifi_ssid = f.readline().strip('\n')
                    wifi_password = f.readline().strip('\n')
                update_conf({
                    'wifi_ssid': wifi_ssid,
                    'wifi_password': wifi_password,
                })

        get_conf.cache_clear()

        _logger.info('Removing old config files')
        unlink_file('iotbox_version')
        unlink_file('wifi_network.txt')
        unlink_file('odoo-db-uuid.conf')
        unlink_file('odoo-enterprise-code.conf')
        unlink_file('odoo-remote-server.conf')
        unlink_file('token')
        unlink_file('odoo-subject.conf')


def url_is_valid(url):
    """
    Checks whether the provided url is a valid one or not
    :param url: the URL to check
    :return: boolean indicating if the URL is valid.
    """
    try:
        result = urllib3.util.parse_url(url.strip())
        return all([result.scheme in ["http", "https"], result.netloc, result.host != 'localhost'])
    except urllib3.exceptions.LocationParseError:
        return False


def parse_url(url):
    """
    Parses URL params and returns them as a dictionary starting by the url.

    Does not allow multiple params with the same name (e.g. <url>?a=1&a=2 will return the same as <url>?a=1)
    :param url: the URL to parse
    :return: the dictionary containing the URL and params
    """
    if not url_is_valid(url):
        raise ValueError("Invalid URL provided.")

    url = urllib3.util.parse_url(url.strip())
    search_params = {
        key: value[0]
        for key, value in parse_qs(url.query, keep_blank_values=True).items()
    }
    return {
        "url": f"{url.scheme}://{url.netloc}",
        **search_params,
    }

```

## File: views\pos_display.html

```html
<!DOCTYPE html>
<html>
    <head>
        <meta http-equiv="cache-control" content="no-cache" />
        <meta http-equiv="pragma" content="no-cache" />
        <title class="origin">{{ title or "Odoo's IoTBox" }}</title>
        <script class="origin" type="text/javascript" src="/web/static/lib/jquery/jquery.js"></script>
        <link class="origin" rel="stylesheet" href="/web/static/lib/bootstrap/dist/css/bootstrap.css">
        <link class="origin" rel="stylesheet" type="text/css" href="/web/static/src/libs/fontawesome/css/font-awesome.css"/>
        <script type="text/javascript" class="origin">
            var display_identifier = '{{ display_identifier }}';
            {{ cust_js|safe }}
        </script>
        <script>
            $(function() {
                setInterval(() => {
                    fetch('/point_of_sale/iot_devices', {
                        method: 'POST',
                        headers: {
                            'Accept': 'application/json',
                            'Content-Type': 'application/json',
                        },
                        body: JSON.stringify({}),
                    })
                    .then(response => response.json())
                    .then(data => {
                        // group devices by type
                        const grouped_devices = JSON.parse(data.result).iot_device_status.reduce((acc, device) => {
                            acc[device.type] = acc[device.type] || [];
                            acc[device.type].push(device);

                            return acc;
                        }, {})

                        const iot_devices_section = document.querySelectorAll('.iot-devices-section');

                        // Hide or show the 'IoT Devices' part depending on devices
                        iot_devices_section.forEach(e => e.style.display = 'block');
                        if (Object.keys(grouped_devices).length === 0) {
                            iot_devices_section.forEach(e => e.style.display = 'none');
                            return;
                        }

                        // create the html object
                        let iot_devices_html = '';
                        for (const type of Object.keys(grouped_devices)) {
                            iot_devices_html += '<tr><td>' + type + 's</td><td><ul>';
                            // display the first 10 devices
                            for (const device of grouped_devices[type].slice(0, 10)) {
                                iot_devices_html += '<li>' + device.name + '</li>';
                            }
                            // add ... at the end if there are more than 10 devices
                            if (grouped_devices[type].length > 10) {
                                iot_devices_html += '<li>...</li>';
                            }
                            iot_devices_html += '</ul></td></tr>';
                        }
                        $('#iot-devices').html(iot_devices_html);
                    })
                    .catch((error) => {
                        console.error('Error: ', error);
                    });
                }, 60000); // fetch every minute
            });

        </script>
        <style class="origin">
            html, body {
                height: 100%;
            }
        </style>
        <style>
            body {
                background: url('/hw_posbox_homepage/static/img/background-light.svg') no-repeat center center fixed;
                background-size: cover;
                height: 100vh;
            }
            .pos-display-boxes {
                position: absolute;
                right: 20px;
                bottom: 20px;
            }
            .pos-display-box {
                padding: 10px 20px;
                background: rgba(255, 255, 255, 0.17);
                border: 1px solid rgba(255, 255, 255, 0.06);
                box-shadow: 0 0 5px 0 rgba(60, 60, 60, 0.4);
                border-radius: 8px;
                width: 500px;
                margin-top: 20px;
                backdrop-filter: blur(5px);
            }
            .info-text {
                font-size: 15px;
            }
            .iot-devices-section {
                display: none;
            }
            .table {
                --bs-table-bg: none;
            }
        </style>
    </head>
    <body>
        <div class="container-fluid">
            <div class="text-center pt-5">
                <img style="width: 150px" src="/web/static/img/logo2.png" alt="odoo_logo">
                <p class="mt-3" style="font-size: 25px">IoTBox: <span style="text-transform: capitalize">{{ hostname }}</span></p>
            </div>
            <div class="pos-display-boxes">
                {% if pairing_code %}
                    <div class="pos-display-box">
                        <h4 class="text-center mb-3">Pairing Code</h4>
                        <hr/>
                        <h4 class="text-center mb-3">{{ pairing_code }}</h4>
                    </div>
                {% endif %}
                <div class="pos-display-box">
                    <h4 class="text-center mb-3">POS Client display</h4>
                    <h5 class="text-center mb-1">IoT Interfaces</h5>
                    <table class="table table-hover table-sm table-pos-info">
                        <thead>
                            <tr>
                                <th>Type</th>
                                <th>IP</th>
                            </tr>
                        </thead>
                        <tbody>
                            {% for display_iface in display_ifaces -%}
                                <tr>
                                    <td><i class="fa fa-{{ display_iface.icon }}"/> {{ display_iface.essid }}</td>
                                    <td>{{ display_iface.addr }}</td>
                                </tr>
                            {%- endfor %}
                        </tbody>
                    </table>
                    <h5 class="text-center mb-1 iot-devices-section">IoT Devices</h5>
                    <table class="table table-hover table-sm table-pos-info iot-devices-section">
                        <thead>
                            <tr>
                                <th>Type</th>
                                <th>Devices</th>
                            </tr>
                        </thead>
                        <tbody id="iot-devices">
                    </table>
                </div>
            </div>
        </div>
    </body>
</html>

```

