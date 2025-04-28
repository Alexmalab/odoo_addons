# Odoo Module: hw_drivers

Category: Hidden

This file contains the source code of the Odoo module.

## File: connection_manager.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import datetime, timedelta
import logging
import subprocess
import requests
from threading import Thread
import time
import urllib3

from odoo.modules.module import get_resource_path
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
            while (datetime.now() < end_time):
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
            elif all(key in result for key in ['url', 'token', 'db_uuid', 'enterprise_code']):
                self._connect_to_server(result['url'], result['token'], result['db_uuid'], result['enterprise_code'])
        except Exception as e:
            _logger.error('Could not reach iot-proxy.odoo.com')
            _logger.error('A error encountered : %s ' % e)

    def _connect_to_server(self, url, token, db_uuid, enterprise_code):
        if db_uuid and enterprise_code:
            helpers.add_credential(db_uuid, enterprise_code)

        # Save DB URL and token
        subprocess.check_call([get_resource_path('point_of_sale', 'tools/posbox/configuration/connect_to_server.sh'), url, '', token, 'noreboot'])
        # Notify the DB, so that the kanban view already shows the IoT Box
        manager.send_alldevices()
        # Restart to checkout the git branch, get a certificate, load the IoT handlers...
        subprocess.check_call(["sudo", "service", "odoo", "restart"])

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
    def __new__(cls, clsname, bases, attrs):
        newclass = super(DriverMetaClass, cls).__new__(cls, clsname, bases, attrs)
        if hasattr(newclass, 'priority'):
            newclass.priority += 1
        else:
            newclass.priority = 0
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
            if device.device_identifier in self.sessions[session]['devices'] and not self.sessions[session]['event'].isSet():
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

from odoo import http


class IoTBoxHttpRequest(http.HttpRequest):
    def dispatch(self):
        if self._is_cors_preflight(http.request.endpoint):
            # Using the PoS in debug mode in v12, the call to '/hw_proxy/handshake' contains the
            # 'X-Debug-Mode' header, which was removed from 'Access-Control-Allow-Headers' in v13.
            # When the code of http.py is not checked out to v12 (i.e. in Community), the connection
            # fails as the header is rejected and none of the devices can be used.
            headers = {
                'Access-Control-Max-Age': 60 * 60 * 24,
                'Access-Control-Allow-Headers': 'Origin, X-Requested-With, Content-Type, Accept, Authorization, X-Debug-Mode'
            }
            return http.Response(status=200, headers=headers)
        return super(IoTBoxHttpRequest, self).dispatch()


class IoTBoxRoot(http.Root):
    def setup_db(self, httprequest):
        # No database on the IoT Box
        pass

    def get_request(self, httprequest):
        # Override HttpRequestwith IoTBoxHttpRequest
        if httprequest.mimetype not in ("application/json", "application/json-rpc"):
            return IoTBoxHttpRequest(httprequest)
        return super(IoTBoxRoot, self).get_request(httprequest)

http.Root = IoTBoxRoot
http.root = IoTBoxRoot()

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

from dbus.mainloop.glib import DBusGMainLoop
import json
import logging
import socket
from threading import Thread
import time
import urllib3

from odoo.addons.hw_drivers.tools import helpers

_logger = logging.getLogger(__name__)

drivers = []
interfaces = {}
iot_devices = {}


class Manager(Thread):
    def send_alldevices(self):
        """
        This method send IoT Box and devices informations to Odoo database
        """
        server = helpers.get_odoo_server_url()
        if server:
            subject = helpers.read_file_first_line('odoo-subject.conf')
            if subject:
                domain = helpers.get_ip().replace('.', '-') + subject.strip('*')
            else:
                domain = helpers.get_ip()
            iot_box = {
                'name': socket.gethostname(),
                'identifier': helpers.get_mac_address(),
                'ip': domain,
                'token': helpers.get_token(),
                'version': helpers.get_version(),
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
            data = {'params': {'iot_box': iot_box, 'devices': devices_list,}}
            # disable certifiacte verification
            urllib3.disable_warnings()
            http = urllib3.PoolManager(cert_reqs='CERT_NONE')
            try:
                http.request(
                    'POST',
                    server + "/iot/setup",
                    body=json.dumps(data).encode('utf8'),
                    headers={
                        'Content-type': 'application/json',
                        'Accept': 'text/plain',
                    },
                )
            except Exception as e:
                _logger.error('Could not reach configured server')
                _logger.error('A error encountered : %s ' % e)
        else:
            _logger.warning('Odoo server not set')

    def run(self):
        """
        Thread that will load interfaces and drivers and contact the odoo server with the updates
        """

        _logger.info("IoT Box Image version: %s", helpers.get_version())
        helpers.check_git_branch()
        is_certificate_ok, certificate_details = helpers.get_certificate_status()
        if not is_certificate_ok:
            _logger.warning("An error happened when trying to get the HTTPS certificate: %s",
                            certificate_details)

        # We first add the IoT Box to the connected DB because IoT handlers cannot be downloaded if
        # the identifier of the Box is not found in the DB. So add the Box to the DB.
        self.send_alldevices()
        helpers.download_iot_handlers()
        helpers.load_iot_handlers()

        # Start the interfaces
        for interface in interfaces.values():
            try:
                i = interface()
                i.daemon = True
                i.start()
            except Exception as e:
                _logger.error("Error in %s: %s", str(interface), e)

        # Check every 3 secondes if the list of connected devices has changed and send the updated
        # list to the connected DB.
        self.previous_iot_devices = []
        while 1:
            try:
                if iot_devices != self.previous_iot_devices:
                    self.send_alldevices()
                    self.previous_iot_devices = iot_devices.copy()
                time.sleep(3)
            except:
                # No matter what goes wrong, the Manager loop needs to keep running
                _logger.error(format_exc())


# Must be started from main thread
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
import json
import logging
import os
import subprocess
import time

from odoo import http, tools
from odoo.http import send_file
from odoo.modules.module import get_resource_path

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
            iot_device.action(data)
            return True
        return False

    @http.route('/hw_drivers/check_certificate', type='http', auth='none', cors='*', csrf=False, save_session=False)
    def check_certificate(self):
        """
        This route is called when we want to check if certificate is up-to-date
        Used in cron.daily
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
                return event

        # Wait for new event
        if req['event'].wait(50):
            req['event'].clear()
            req['result']['session_id'] = req['session_id']
            return req['result']

    @http.route('/hw_drivers/box/connect', type='http', auth='none', cors='*', csrf=False, save_session=False)
    def connect_box(self, token):
        """
        This route is called when we want that a IoT Box will be connected to a Odoo DB
        token is a base 64 encoded string and have 2 argument separate by |
        1 - url of odoo DB
        2 - token. This token will be compared to the token of Odoo. He have 1 hour lifetime
        """
        server = helpers.get_odoo_server_url()
        image = get_resource_path('hw_drivers', 'static/img', 'False.jpg')
        if not server:
            credential = b64decode(token).decode('utf-8').split('|')
            url = credential[0]
            token = credential[1]
            if len(credential) > 2:
                # IoT Box send token with db_uuid and enterprise_code only since V13
                db_uuid = credential[2]
                enterprise_code = credential[3]
                helpers.add_credential(db_uuid, enterprise_code)
            try:
                subprocess.check_call([get_resource_path('point_of_sale', 'tools/posbox/configuration/connect_to_server.sh'), url, '', token, 'noreboot'])
                manager.send_alldevices()
                image = get_resource_path('hw_drivers', 'static/img', 'True.jpg')
                helpers.odoo_restart(3)
            except subprocess.CalledProcessError as e:
                _logger.error('A error encountered : %s ' % e.output)
        if os.path.isfile(image):
            with open(image, 'rb') as f:
                return f.read()

    @http.route('/hw_drivers/download_logs', type='http', auth='none', cors='*', csrf=False, save_session=False)
    def download_logs(self):
        """
        Downloads the log file
        """
        if tools.config['logfile']:
            res = send_file(tools.config['logfile'], mimetype="text/plain", as_attachment=True)
            res.headers['Cache-Control'] = 'no-cache'
            return res

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

## File: iot_handlers\drivers\DisplayDriver.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import jinja2
import json
import logging
import netifaces as ni
import os
import subprocess
import threading
import time

import urllib3

from odoo import http
from odoo.addons.hw_drivers.connection_manager import connection_manager
from odoo.addons.hw_drivers.driver import Driver
from odoo.addons.hw_drivers.event_manager import event_manager
from odoo.addons.hw_drivers.main import iot_devices
from odoo.addons.hw_drivers.tools import helpers

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
        while self.device_identifier != 'distant_display' and not self._stopped.isSet():
            time.sleep(60)
            if self.url != 'http://localhost:8069/point_of_sale/display/' + self.device_identifier:
                # Refresh the page every minute
                self.call_xdotools('F5')

    def update_url(self, url=None):
        os.environ['DISPLAY'] = ":0." + self._x_screen
        os.environ['XAUTHORITY'] = '/run/lightdm/pi/xauthority'
        firefox_env = os.environ.copy()
        firefox_env['HOME'] = '/tmp/' + self._x_screen
        self.url = url or 'http://localhost:8069/point_of_sale/display/' + self.device_identifier
        new_window = subprocess.call(['xdotool', 'search', '--onlyvisible', '--screen', self._x_screen, '--class', 'Firefox'])
        subprocess.Popen(['firefox', self.url], env=firefox_env)
        if new_window:
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
            subprocess.call(['xdotool', 'search', '--sync', '--onlyvisible', '--screen', self._x_screen, '--class', 'Firefox', 'key', keystroke])
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

        with open(os.path.join(os.path.dirname(__file__), "../../static/src/js/worker.js")) as js:
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

        if not display_identifier:
            display_identifier = DisplayDriver.get_default_display().device_identifier

        return pos_display_template.render({
            'title': "Odoo -- Point of Sale",
            'breadcrumb': 'POS Client display',
            'cust_js': cust_js,
            'display_ifaces': display_ifaces,
            'display_identifier': display_identifier,
            'pairing_code': connection_manager.pairing_code,
        })

```

## File: iot_handlers\drivers\KeyboardUSBDriver.py

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

from odoo import http, _
from odoo.addons.hw_drivers.controllers.proxy import proxy_drivers
from odoo.addons.hw_drivers.driver import Driver
from odoo.addons.hw_drivers.event_manager import event_manager
from odoo.addons.hw_drivers.main import iot_devices
from odoo.addons.hw_drivers.tools import helpers

_logger = logging.getLogger(__name__)
xlib = ctypes.cdll.LoadLibrary('libX11.so.6')


class KeyboardUSBDriver(Driver):
    connection_type = 'usb'
    keyboard_layout_groups = []
    available_layouts = []

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
                self.input_device = evdev_device

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
            except Exception as e:
                _logger.error('Could not reach configured server')
                _logger.error('A error encountered : %s ' % e)

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
            return re.sub(r"[^\w \-+/*&]", '', "%s - %s" % (manufacturer, product))
        except ValueError as e:
            _logger.warning(e)
            return _('Unknown input device')

    def run(self):
        try:
            for event in self.input_device.read_loop():
                if self._stopped.isSet():
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
        file_path = Path.home() / 'odoo-keyboard-layouts.conf'
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
        file_path = Path.home() / 'odoo-keyboard-layouts.conf'
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

        file_path = Path.home() / 'odoo-keyboard-is-scanner.conf'
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
        file_path = Path.home() / 'odoo-keyboard-is-scanner.conf'
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
            self.input_device.grab()
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

## File: iot_handlers\drivers\PrinterDriver.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from base64 import b64decode
from cups import IPPError, IPP_PRINTER_IDLE, IPP_PRINTER_PROCESSING, IPP_PRINTER_STOPPED
import dbus
import io
import logging
import netifaces as ni
import os
from PIL import Image, ImageOps
import re
import subprocess
import tempfile
from uuid import getnode as get_mac

from odoo import http
from odoo.addons.hw_drivers.connection_manager import connection_manager
from odoo.addons.hw_drivers.controllers.proxy import proxy_drivers
from odoo.addons.hw_drivers.driver import Driver
from odoo.addons.hw_drivers.event_manager import event_manager
from odoo.addons.hw_drivers.iot_handlers.interfaces.PrinterInterface import PPDs, conn, cups_lock
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
        if any(x in device['url'] for x in protocol) and device['device-make-and-model'] != 'Unknown' or 'direct' in device['device-class']:
            model = cls.get_device_model(device)
            ppdFile = ''
            for ppd in PPDs:
                if model and model in PPDs[ppd]['ppd-product']:
                    ppdFile = ppd
                    break
            with cups_lock:
                if ppdFile:
                    conn.addPrinter(name=device['identifier'], ppdname=ppdFile, device=device['url'])
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
        status = 'connected' if any(iot_devices[d].device_type == "printer" and iot_devices[d].device_connection == 'direct' for d in iot_devices) else 'disconnected'
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

    def print_raw(self, data):
        process = subprocess.Popen(["lp", "-d", self.device_identifier], stdin=subprocess.PIPE)
        process.communicate(data)
        if process.returncode != 0:
            # The stderr isn't meaningful so we don't log it ('No such file or directory')
            _logger.error('Printing failed: printer with the identifier "%s" could not be found',
                          self.device_identifier)

    def print_receipt(self, data):
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
        commands = RECEIPT_PRINTER_COMMANDS[self.receipt_protocol]
        for drawer in commands['drawers']:
            self.print_raw(drawer)

    def _action_default(self, data):
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

from odoo import _
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

        try:
            with self._device_lock:
                self._actions[data['action']](data)
                time.sleep(self._protocol.commandDelay)
        except Exception:
            msg = _('An error occurred while performing action %s on %s') % (data, self.device_name)
            _logger.exception(msg)
            self._status = {'status': self.STATUS_ERROR, 'message_title': msg, 'message_body': traceback.format_exc()}
            self._push_status()

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
                while not self._stopped.isSet():
                    self._take_measure()
                    time.sleep(self._protocol.newMeasureDelay)
        except Exception:
            msg = _('Error while reading %s', self.device_name)
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

## File: iot_handlers\interfaces\DisplayInterface.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from re import sub, finditer
import subprocess
import RPi.GPIO as GPIO
import logging

from odoo.addons.hw_drivers.interface import Interface


_logger = logging.getLogger(__name__)

try:
    from vcgencmd import Vcgencmd
except ImportError:
    Vcgencmd = None
    _logger.warning('Could not import library vcgencmd')


class DisplayInterface(Interface):
    _loop_delay = 0
    connection_type = 'display'

    def get_devices(self):

        # If no display connected, create "fake" device to be accessed from another computer
        display_devices = {
             'distant_display' : {
                  'name': "Distant Display",
             },
        }

        if Vcgencmd:
            return self.get_devices_vcgencmd() or display_devices
        else:
            return self.get_devices_tvservice() or display_devices


    def get_devices_tvservice(self):
        display_devices = {}
        displays = subprocess.check_output(['tvservice', '-l']).decode()
        x_screen = 0
        for match in finditer(r'Display Number (\d), type HDMI (\d)', displays):
            display_id, hdmi_id = match.groups()
            tvservice_output = subprocess.check_output(['tvservice', '-nv', display_id]).decode().strip()
            if tvservice_output:
                display_name = tvservice_output.split('=')[1]
                display_identifier = sub('[^a-zA-Z0-9 ]+', '', display_name).replace(' ', '_') + "_" + str(hdmi_id)
                iot_device = {
                    'identifier': display_identifier,
                    'name': display_name,
                    'x_screen': str(x_screen),
                }
                display_devices[display_identifier] = iot_device
                x_screen += 1

        return display_devices

    def get_devices_vcgencmd(self):
        """
        With the new IoT build 23_11 which uses Raspi OS Bookworm,
        tvservice is no longer usable.
        vcgencmd returns the display power state as on or off of the display whose ID is passed as the parameter.
        The display ID for the preceding three methods are determined by the following table.

        Display        ID
        Main LCD        0
        Secondary LCD   1
        HDMI 0          2
        Composite       3
        HDMI 1          7
        """
        display_devices = {}
        x_screen = 0
        hdmi_port = {'hdmi_0' : 2} # HDMI 0
        rpi_type = GPIO.RPI_INFO.get('TYPE')
        # Check if it is a RPI 3B+ beacause he response on for booth hdmi port
        if 'Pi 4' in rpi_type:
            hdmi_port.update({'hdmi_1': 7}) # HDMI 1

        try:
            for hdmi in hdmi_port:
                power_state_hdmi = Vcgencmd().display_power_state(hdmi_port.get(hdmi))
                if power_state_hdmi == 'on':
                    iot_device = {
                        'identifier': hdmi,
                        'name': 'Display hdmi ' + str(x_screen),
                        'x_screen': str(x_screen),
                    }
                    display_devices[hdmi] = iot_device
                    x_screen += 1
        except subprocess.CalledProcessError:
            _logger.warning('Vcgencmd "display_power_state" method call failed')

        return display_devices

```

## File: iot_handlers\interfaces\PrinterInterface.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from cups import Connection as cups_connection
from re import sub
from threading import Lock

from odoo.addons.hw_drivers.interface import Interface

conn = cups_connection()
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
                    printer.update({'supported': True}) # these printers are automatically supported
                    device_class = 'network'
                    if 'usb' in printer.get('device-uri'):
                        device_class = 'direct'
                    printer.update({'device-class': device_class})
                    printer.update({'device-make-and-model': printer_name}) # give name setted in Cups
                    printer.update({'device-id': ''})
                    devices.update({printer_name: printer})
        for path, device in devices.items():
            identifier = self.get_identifier(path)
            device.update({'identifier': identifier})
            device.update({'url': path})
            device.update({'disconnect_counter': 0})
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

## File: iot_handlers\interfaces\SerialInterface.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from glob import glob

from odoo.addons.hw_drivers.interface import Interface


class SerialInterface(Interface):
    connection_type = 'serial'

    def get_devices(self):
        serial_devices = {}
        for identifier in glob('/dev/serial/by-path/*'):
            serial_devices[identifier] = {
                'identifier': identifier
            }
        return serial_devices

```

## File: iot_handlers\interfaces\USBInterface.py

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

import datetime
from enum import Enum
from importlib import util
import io
import json
import logging
import netifaces
from OpenSSL import crypto
import os
from pathlib import Path
import subprocess
import urllib3
import zipfile
from threading import Thread
import time

from odoo import _, http
from odoo.modules.module import get_resource_path

_logger = logging.getLogger(__name__)

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
        subprocess.check_call(["sudo", "service", "odoo", "restart"])

def access_point():
    return get_ip() == '10.11.12.1'

def add_credential(db_uuid, enterprise_code):
    write_file('odoo-db-uuid.conf', db_uuid)
    write_file('odoo-enterprise-code.conf', enterprise_code)

def check_certificate():
    """
    Check if the current certificate is up to date or not authenticated
    :return CheckCertificateStatus
    """
    server = get_odoo_server_url()
    if not server:
        return {"status": CertificateStatus.ERROR,
                "error_code": "ERR_IOT_HTTPS_CHECK_NO_SERVER"}

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
        message = _('Your certificate %s must be updated') % (cn)
        _logger.info(message)
        return {"status": CertificateStatus.NEED_REFRESH}
    else:
        message = _('Your certificate %s is valid until %s') % (cn, cert_end_date)
        _logger.info(message)
        return {"status": CertificateStatus.OK, "message": message}

def check_git_branch():
    """
    Check if the local branch is the same than the connected Odoo DB and
    checkout to match it if needed.
    """
    server = get_odoo_server_url()
    if server:
        urllib3.disable_warnings()
        http = urllib3.PoolManager(cert_reqs='CERT_NONE')
        try:
            response = http.request(
                'POST',
                server + "/web/webclient/version_info",
                body = '{}',
                headers = {'Content-type': 'application/json'}
            )

            if response.status == 200:
                git = ['git', '--work-tree=/home/pi/odoo/', '--git-dir=/home/pi/odoo/.git']

                db_branch = json.loads(response.data)['result']['server_serie'].replace('~', '-')
                if not subprocess.check_output(git + ['ls-remote', 'origin', db_branch]):
                    db_branch = 'master'

                local_branch = subprocess.check_output(git + ['symbolic-ref', '-q', '--short', 'HEAD']).decode('utf-8').rstrip()
                _logger.info("Current IoT Box local git branch: %s / Associated Odoo database's git branch: %s", local_branch, db_branch)

                if db_branch != local_branch:
                    subprocess.call(["sudo", "mount", "-o", "remount,rw", "/"])
                    subprocess.check_call(["rm", "-rf", "/home/pi/odoo/addons/hw_drivers/iot_handlers/drivers/*"])
                    subprocess.check_call(["rm", "-rf", "/home/pi/odoo/addons/hw_drivers/iot_handlers/interfaces/*"])
                    subprocess.check_call(git + ['branch', '-m', db_branch])
                    subprocess.check_call(git + ['remote', 'set-branches', 'origin', db_branch])
                    os.system('/home/pi/odoo/addons/point_of_sale/tools/posbox/configuration/posbox_update.sh')
                    subprocess.call(["sudo", "mount", "-o", "remount,ro", "/"])
                    subprocess.call(["sudo", "mount", "-o", "remount,rw", "/root_bypass_ramdisks/etc/cups"])

        except Exception as e:
            _logger.error('Could not reach configured server')
            _logger.error('A error encountered : %s ' % e)

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
    major, minor = get_version().split('.')
    return 'iotboxv%s_%s.zip' % (major, minor)

def get_ip():
    while True:
        try:
            return netifaces.ifaddresses('eth0')[netifaces.AF_INET][0]['addr']
        except KeyError:
            pass

        try:
            return netifaces.ifaddresses('wlan0')[netifaces.AF_INET][0]['addr']
        except KeyError:
            pass

        _logger.warning("Couldn't get IP, sleeping and retrying.")
        time.sleep(5)

def get_mac_address():
    while True:
        try:
            return netifaces.ifaddresses('eth0')[netifaces.AF_LINK][0]['addr']
        except KeyError:
            pass

        try:
            return netifaces.ifaddresses('wlan0')[netifaces.AF_LINK][0]['addr']
        except KeyError:
            pass

        _logger.warning("Couldn't get MAC address, sleeping and retrying.")
        time.sleep(5)

def get_ssid():
    ap = subprocess.call(['systemctl', 'is-active', '--quiet', 'hostapd']) # if service is active return 0 else inactive
    if not ap:
        return subprocess.check_output(['grep', '-oP', '(?<=ssid=).*', '/etc/hostapd/hostapd.conf']).decode('utf-8').rstrip()
    process_iwconfig = subprocess.Popen(['iwconfig'], stdout=subprocess.PIPE, stderr=subprocess.STDOUT)
    process_grep = subprocess.Popen(['grep', 'ESSID:"'], stdin=process_iwconfig.stdout, stdout=subprocess.PIPE)
    return subprocess.check_output(['sed', 's/.*"\\(.*\\)"/\\1/'], stdin=process_grep.stdout).decode('utf-8').rstrip()

def get_odoo_server_url():
    ap = subprocess.call(['systemctl', 'is-active', '--quiet', 'hostapd']) # if service is active return 0 else inactive
    if not ap:
        return False
    return read_file_first_line('odoo-remote-server.conf')

def get_token():
    return read_file_first_line('token')

def get_version():
    return subprocess.check_output(['cat', '/var/odoo/iotbox_version']).decode().rstrip()

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
    db_uuid = read_file_first_line('odoo-db-uuid.conf')
    enterprise_code = read_file_first_line('odoo-enterprise-code.conf')
    if not (db_uuid and enterprise_code):
        return "ERR_IOT_HTTPS_LOAD_NO_CREDENTIAL"

    url = 'https://www.odoo.com/odoo-enterprise/iot/x509'
    data = {
        'params': {
            'db_uuid': db_uuid,
            'enterprise_code': enterprise_code
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

    result = json.loads(response.data.decode('utf8'))['result']
    if not result:
        return "ERR_IOT_HTTPS_LOAD_REQUEST_NO_RESULT"

    write_file('odoo-subject.conf', result['subject_cn'])
    subprocess.call(["sudo", "mount", "-o", "remount,rw", "/"])
    subprocess.call(["sudo", "mount", "-o", "remount,rw", "/root_bypass_ramdisks/"])
    Path('/etc/ssl/certs/nginx-cert.crt').write_text(result['x509_pem'])
    Path('/root_bypass_ramdisks/etc/ssl/certs/nginx-cert.crt').write_text(result['x509_pem'])
    Path('/etc/ssl/private/nginx-cert.key').write_text(result['private_key_pem'])
    Path('/root_bypass_ramdisks/etc/ssl/private/nginx-cert.key').write_text(result['private_key_pem'])
    subprocess.call(["sudo", "mount", "-o", "remount,ro", "/"])
    subprocess.call(["sudo", "mount", "-o", "remount,ro", "/root_bypass_ramdisks/"])
    subprocess.call(["sudo", "mount", "-o", "remount,rw", "/root_bypass_ramdisks/etc/cups"])
    subprocess.check_call(["sudo", "service", "nginx", "restart"])
    return True

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
            resp = pm.request('POST', server, fields={'mac': get_mac_address(), 'auto': auto})
            if resp.data:
                subprocess.call(["sudo", "mount", "-o", "remount,rw", "/"])
                drivers_path = Path.home() / 'odoo/addons/hw_drivers/iot_handlers'
                zip_file = zipfile.ZipFile(io.BytesIO(resp.data))
                zip_file.extractall(drivers_path)
                subprocess.call(["sudo", "mount", "-o", "remount,ro", "/"])
                subprocess.call(["sudo", "mount", "-o", "remount,rw", "/root_bypass_ramdisks/etc/cups"])
        except Exception as e:
            _logger.error('Could not reach configured server')
            _logger.error('A error encountered : %s ' % e)

def compute_iot_handlers_addon_name(handler_kind, handler_file_name):
    # TODO: replace with `removesuffix` (for Odoo version using an IoT image that use Python >= 3.9)
    return "odoo.addons.hw_drivers.iot_handlers.{handler_kind}.{handler_name}".\
        format(handler_kind=handler_kind, handler_name=handler_file_name.replace('.py', ''))

def load_iot_handlers():
    """
    This method loads local files: 'odoo/addons/hw_drivers/iot_handlers/drivers' and
    'odoo/addons/hw_drivers/iot_handlers/interfaces'
    And execute these python drivers and interfaces
    """
    for directory in ['interfaces', 'drivers']:
        path = get_resource_path('hw_drivers', 'iot_handlers', directory)
        filesList = os.listdir(path)
        for file in filesList:
            path_file = os.path.join(path, file)
            spec = util.spec_from_file_location(compute_iot_handlers_addon_name(directory, file), path_file)
            if spec:
                module = util.module_from_spec(spec)
                spec.loader.exec_module(module)
    http.addons_manifest = {}
    http.root = http.Root()

def odoo_restart(delay):
    IR = IoTRestart(delay)
    IR.start()

def read_file_first_line(filename):
    path = Path.home() / filename
    path = Path('/home/pi/' + filename)
    if path.exists():
        with path.open('r') as f:
            return f.readline().strip('\n')
    return ''

def unlink_file(filename):
    subprocess.call(["sudo", "mount", "-o", "remount,rw", "/"])
    path = Path.home() / filename
    if path.exists():
        path.unlink()
    subprocess.call(["sudo", "mount", "-o", "remount,ro", "/"])
    subprocess.call(["sudo", "mount", "-o", "remount,rw", "/root_bypass_ramdisks/etc/cups"])

def write_file(filename, text):
    subprocess.call(["sudo", "mount", "-o", "remount,rw", "/"])
    path = Path.home() / filename
    path.write_text(text)
    subprocess.call(["sudo", "mount", "-o", "remount,ro", "/"])
    subprocess.call(["sudo", "mount", "-o", "remount,rw", "/root_bypass_ramdisks/etc/cups"])

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
        <link class="origin" rel="stylesheet" href="/web/static/lib/bootstrap/css/bootstrap.css">
        <script class="origin" type="text/javascript" src="/web/static/lib/bootstrap/js/bootstrap.min.js"></script>
        <link rel="stylesheet" type="text/css" href="/web/static/lib/fontawesome/css/font-awesome.css"/>
        <script type="text/javascript" class="origin">
            var display_identifier = '{{ display_identifier }}';
            {{ cust_js|safe }}
        </script>
        <style class="origin">
            html, body {
                height: 100%;
            }
        </style>
        <style>
            body {
                background: linear-gradient(to right bottom, #77717e, #c9a8a9);
                height: 100vh;
            }
            .pos-display-boxes {
                position: absolute;
                right: 20px;
                bottom: 20px;
            }
            .pos-display-box {
                padding: 10px 20px;
                background: rgba(0, 0, 0, 0.17);
                border: 1px solid rgba(0, 0, 0, 0.06);
                box-shadow: 1px 1px 0px 0px rgba(60, 60, 60, 0.4);
                color: #fff;
                border-radius: 8px;
                width: 500px;
                margin-top: 20px;
            }
            .pos-display-box hr {
                background-color: #fff;
            }
            .info-text {
                font-size: 15px;
            }
            .table-pos-info {
                color: #fff;
            }
        </style>
    </head>
    <body>
        <div class="container-fluid">
            <div class="text-center pt-5">
                <img style="width: 150px;" src="/web/static/img/logo_inverse_white_206px.png">
                <p class="mt-3" style="color: #fff;font-size: 30px;">IoTBox</p>
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
                    <table class="table table-hover table-sm table-pos-info">
                        <thead>
                            <tr>
                                <th>Interface</th>
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
                    <p class="mb-2 info-text">
                        <i class="fa fa-info-circle mr-1"></i>The customer cart will be displayed here once a Point of Sale session is started.
                    </p>
                    <p class="mb-2 info-text">
                        <i class="fa fa-info-circle mr-1"></i>Odoo version 11 or above is required.
                    </p>
                    <div class="error-message alert alert-danger mb-2 d-none" role="alert" />
                </div>
            </div>
        </div>
    </body>
</html>

```

