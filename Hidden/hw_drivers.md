# Odoo Module: hw_drivers

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import controllers
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
    'website': 'https://www.odoo.com/page/iot',
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
#!/usr/bin/python3
import logging
import time
from threading import Thread, Event, Lock
from traceback import format_exc

from usb import core
from gatt import DeviceManager as Gatt_DeviceManager
import subprocess
import json
from re import sub, finditer
import urllib3
import os
import socket
import sys
from importlib import util
import v4l2
from fcntl import ioctl
from cups import Connection as cups_connection
from glob import glob
from base64 import b64decode
from pathlib import Path
import requests
import socket
import ctypes
from datetime import datetime, timedelta

from odoo import http, _
from odoo.modules.module import get_resource_path
from odoo.addons.hw_drivers.tools import helpers
from odoo.http import request

_logger = logging.getLogger(__name__)


#----------------------------------------------------------
# Controllers
#----------------------------------------------------------

class StatusController(http.Controller):
    @http.route('/hw_drivers/action', type='json', auth='none', cors='*', csrf=False, save_session=False)
    def action(self, session_id, device_id, data):
        """
        This route is called when we want to make a action with device (take picture, printing,...)
        We specify in data from which session_id that action is called
        And call the action of specific device
        """
        iot_device = iot_devices.get(device_id)
        if iot_device:
            iot_device.data['owner'] = session_id
            data = json.loads(data)
            iot_device.action(data)
            return True
        return False

    @http.route('/hw_drivers/check_certificate', type='http', auth='none', cors='*', csrf=False, save_session=False)
    def check_certificate(self):
        """
        This route is called when we want to check if certificate is up-to-date
        Used in cron.daily
        """
        helpers.check_certificate()

    @http.route('/hw_drivers/event', type='json', auth='none', cors='*', csrf=False, save_session=False)
    def event(self, listener):
        """
        listener is a dict in witch there are a sessions_id and a dict of device_id to listen
        """
        req = event_manager.add_request(listener)

        # Search for previous events and remove events older than 5 seconds
        oldest_time = time.time() - 5
        for event in list(event_manager.events):
            if event['time'] < oldest_time:
                del event_manager.events[0]
                continue
            if event['device_id'] in listener['devices'] and event['time'] > listener['last_event']:
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
                m.send_alldevices()
                image = get_resource_path('hw_drivers', 'static/img', 'True.jpg')
                helpers.odoo_restart(3)
            except subprocess.CalledProcessError as e:
                _logger.error('A error encountered : %s ' % e.output)
        if os.path.isfile(image):
            with open(image, 'rb') as f:
                return f.read()

#----------------------------------------------------------
# Log Exceptions
#----------------------------------------------------------

class ExceptionLogger:
    """
    Redirect Exceptions to the logger to keep track of them in the log file.
    """

    def __init__(self):
        self.logger = logging.getLogger()

    def write(self, message):
        if message != '\n':
            self.logger.error(message)

    def flush(self):
        pass

sys.stderr = ExceptionLogger()

#----------------------------------------------------------
# Drivers
#----------------------------------------------------------

drivers = []
bt_devices = {}
socket_devices = {}
iot_devices = {}

class DriverMetaClass(type):
    def __new__(cls, clsname, bases, attrs):
        newclass = super(DriverMetaClass, cls).__new__(cls, clsname, bases, attrs)
        # Some drivers must be tried only when all the others have been ruled out. These are kept at the bottom of the list.
        if newclass.is_tested_last:
            drivers.append(newclass)
        else:
            drivers.insert(0, newclass)
        return newclass

class Driver(Thread, metaclass=DriverMetaClass):
    """
    Hook to register the driver into the drivers list
    """
    connection_type = ""
    is_tested_last = False

    def __init__(self, device):
        super(Driver, self).__init__()
        self.dev = device
        self.data = {'value': ''}
        self.gatt_device = False
        self._device_manufacturer = ''

    @property
    def device_name(self):
        return self._device_name

    @property
    def device_identifier(self):
        return self.dev.identifier

    @property
    def device_manufacturer(self):
        return self._device_manufacturer

    @property
    def device_connection(self):
        """
        On specific driver override this method to give connection type of device
        return string
        possible value : direct - network - bluetooth - serial - hdmi
        """
        return self._device_connection

    @property
    def device_type(self):
        """
        On specific driver override this method to give type of device
        return string
        possible value : printer - camera - keyboard - scanner - display - device
        """
        return self._device_type

    @classmethod
    def supported(cls, device):
        """
        On specific driver override this method to check if device is supported or not
        return True or False
        """
        pass

    def get_message(self):
        return ''

    def action(self, data):
        """
        On specific driver override this method to make a action with device (take picture, printing,...)
        """
        raise NotImplementedError()

    def disconnect(self):
        del iot_devices[self.device_identifier]


#----------------------------------------------------------
# Device manager
#----------------------------------------------------------

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
        expired_sessions = [session for session in self.sessions if now - self.sessions[session]['time_request'] > max_time]
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
            'device_id': device.device_identifier,
            'time': time.time(),
            'request_data': json.loads(request.params['data']) if request else None,
        }
        self.events.append(event)
        for session in self.sessions:
            if device.device_identifier in self.sessions[session]['devices'] and not self.sessions[session]['event'].isSet():
                self.sessions[session]['result'] = event
                self.sessions[session]['event'].set()


class IoTDevice(object):

    def __init__(self, dev, connection_type):
        self.dev = dev
        self.connection_type = connection_type

event_manager = EventManager()

#----------------------------------------------------------
# ConnectionManager
#----------------------------------------------------------

class ConnectionManager(Thread):
    def __init__(self):
        super(ConnectionManager, self).__init__()
        self.pairing_code = False
        self.pairing_uuid = False

    def run(self):
        if not helpers.get_odoo_server_url():
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

        urllib3.disable_warnings()
        req = requests.post('https://iot-proxy.odoo.com/odoo-enterprise/iot/connect-box', json=data, verify=False)
        result = req.json().get('result', {})

        if all(key in result for key in ['pairing_code', 'pairing_uuid']):
            self.pairing_code = result['pairing_code']
            self.pairing_uuid = result['pairing_uuid']
        elif all(key in result for key in ['url', 'token', 'db_uuid', 'enterprise_code']):
            self._connect_to_server(result['url'], result['token'], result['db_uuid'], result['enterprise_code'])

    def _connect_to_server(self, url, token, db_uuid, enterprise_code):
        if db_uuid and enterprise_code:
            helpers.add_credential(db_uuid, enterprise_code)

        # Save DB URL and token
        subprocess.check_call([get_resource_path('point_of_sale', 'tools/posbox/configuration/connect_to_server.sh'), url, '', token, 'noreboot'])
        # Notify the DB, so that the kanban view already shows the IoT Box
        m.send_alldevices()
        # Restart to checkout the git branch, get a certificate, load the IoT handlers...
        subprocess.check_call(["sudo", "service", "odoo", "restart"])

    def _refresh_displays(self):
        """Refresh all displays to hide the pairing code"""
        for d in iot_devices:
            if iot_devices[d].device_type == 'display':
                iot_devices[d].action({
                    'action': 'display_refresh'
                })

#----------------------------------------------------------
# Manager
#----------------------------------------------------------

class Manager(Thread):

    def load_drivers(self):
        """
        This method loads local files: 'odoo/addons/hw_drivers/drivers'
        And execute these python drivers
        """
        helpers.download_drivers()
        path = get_resource_path('hw_drivers', 'drivers')
        driversList = os.listdir(path)
        self.devices = {}
        for driver in driversList:
            path_file = os.path.join(path, driver)
            spec = util.spec_from_file_location(driver, path_file)
            if spec:
                module = util.module_from_spec(spec)
                spec.loader.exec_module(module)
        http.addons_manifest = {}
        http.root = http.Root()

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
                'version': helpers.get_version()
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
            data = {
                'params': {
                    'iot_box' : iot_box,
                    'devices' : devices_list,
                }
            }
            # disable certifiacte verification
            urllib3.disable_warnings()
            http = urllib3.PoolManager(cert_reqs='CERT_NONE')
            try:
                http.request(
                    'POST',
                    server + "/iot/setup",
                    body = json.dumps(data).encode('utf8'),
                    headers = {'Content-type': 'application/json', 'Accept': 'text/plain'}
                )
            except Exception as e:
                _logger.error('Could not reach configured server')
                _logger.error('A error encountered : %s ' % e)
        else:
            _logger.warning('Odoo server not set')

    def get_connected_displays(self):
        display_devices = {}

        displays = subprocess.check_output(['tvservice', '-l']).decode()
        x_screen = 0
        for match in finditer('Display Number (\d), type HDMI (\d)', displays):
            display_id, hdmi_id = match.groups()
            tvservice_output = subprocess.check_output(['tvservice', '-nv', display_id]).decode().rstrip()
            if tvservice_output:
                display_name = tvservice_output.split('=')[1]
                display_identifier = sub('[^a-zA-Z0-9 ]+', '', display_name).replace(' ', '_') + "_" + str(hdmi_id)
                iot_device = IoTDevice({
                    'identifier': display_identifier,
                    'name': display_name,
                    'x_screen': str(x_screen),
                }, 'display')
                display_devices[display_identifier] = iot_device
                x_screen += 1

        if not len(display_devices):
            # No display connected, create "fake" device to be accessed from another computer
            display_devices['distant_display'] = IoTDevice({
                'identifier': "distant_display",
                'name': "Distant Display",
            }, 'display')

        return display_devices

    def serial_loop(self):
        serial_devices = {}
        for identifier in glob('/dev/serial/by-path/*'):
            iot_device = IoTDevice({'identifier': identifier, }, 'serial')
            serial_devices[identifier] = iot_device
        return serial_devices

    def usb_loop(self):
        """
        Loops over the connected usb devices, assign them an identifier, instantiate
        an `IoTDevice` for them.

        USB devices are identified by a combination of their `idVendor` and
        `idProduct`. We can't be sure this combination in unique per equipment.
        To still allow connecting multiple similar equipments, we complete the
        identifier by a counter. The drawbacks are we can't be sure the equipments
        will get the same identifiers after a reboot or a disconnect/reconnect.

        :return: a dict of the `IoTDevices` instances indexed by their identifier.
        """
        usb_devices = {}
        devs = core.find(find_all=True)
        cpt = 2
        for dev in devs:
            dev.identifier =  "usb_%04x:%04x" % (dev.idVendor, dev.idProduct)
            if dev.identifier in usb_devices:
                dev.identifier += '_%s' % cpt
                cpt += 1
            iot_device = IoTDevice(dev, 'usb')
            usb_devices[dev.identifier] = iot_device
        return usb_devices

    def video_loop(self):
        camera_devices = {}
        videos = glob('/dev/video*')
        for video in videos:
            with open(video, 'w') as path:
                dev = v4l2.v4l2_capability()
                ioctl(path, v4l2.VIDIOC_QUERYCAP, dev)
                dev.interface = video
                dev.identifier = dev.bus_info.decode('utf-8')
                iot_device = IoTDevice(dev, 'video')
                camera_devices[dev.identifier] = iot_device
        return camera_devices

    def printer_loop(self):
        printer_devices = {}
        with cups_lock:
            devices = conn.getDevices()
        for path in devices:
            if 'uuid=' in path:
                serial = sub('[^a-zA-Z0-9 ]+', '', path.split('uuid=')[1])
            elif 'serial=' in path:
                serial = sub('[^a-zA-Z0-9 ]+', '', path.split('serial=')[1])
            else:
                serial = sub('[^a-zA-Z0-9 ]+', '', path)
            devices[path]['identifier'] = serial
            devices[path]['url'] = path
            iot_device = IoTDevice(devices[path], 'printer')
            printer_devices[serial] = iot_device
        return printer_devices

    def run(self):
        """
        Thread that will check connected/disconnected device, load drivers if needed and contact the odoo server with the updates
        """
        helpers.check_git_branch()
        helpers.check_certificate()
        updated_devices = {}
        self.send_alldevices()
        self.load_drivers()
        # The list of devices doesn't change after the Raspberry has booted
        display_devices = self.get_connected_displays()
        cpt = 0
        while 1:
            try:
                updated_devices = self.usb_loop()
                updated_devices.update(self.video_loop())
                updated_devices.update(mpdm.devices)
                updated_devices.update(display_devices)
                updated_devices.update(bt_devices)
                updated_devices.update(socket_devices)
                updated_devices.update(self.serial_loop())
                if cpt % 40 == 0:
                    printer_devices = self.printer_loop()
                    cpt = 0
                updated_devices.update(printer_devices)
                cpt += 1
                added = updated_devices.keys() - self.devices.keys()
                removed = self.devices.keys() - updated_devices.keys()
                self.devices = updated_devices
                send_devices = False
                for path in [device_rm for device_rm in removed if device_rm in iot_devices]:
                    iot_devices[path].disconnect()
                    _logger.info('Device %s is now disconnected', path)
                    send_devices = True
                for path in [device_add for device_add in added if device_add not in iot_devices]:
                    for driverclass in [d for d in drivers if d.connection_type == self.devices[path].connection_type]:
                        if driverclass.supported(device = updated_devices[path].dev):
                            _logger.info('Device %s is now connected', path)
                            d = driverclass(device = updated_devices[path].dev)
                            d.daemon = True
                            iot_devices[path] = d
                            # Start the thread after creating the iot_devices entry so the
                            # thread can assume the iot_devices entry will exist while it's
                            # running, at least until the `disconnect` above gets triggered
                            # when `removed` is not empty. Threads are currently not
                            # explicitly terminated when that happens, so the results can
                            # be undefined.
                            d.start()
                            send_devices = True
                            break
                if send_devices:
                    self.send_alldevices()
                time.sleep(3)
            except:
                # No matter what goes wrong, the Manager loop needs to keep running
                _logger.error(format_exc())


class GattBtManager(Gatt_DeviceManager):

    def device_discovered(self, device):
        path = "bt_%s" % (device.mac_address,)
        if path not in bt_devices:
            device.manager = self
            device.identifier = path
            iot_device = IoTDevice(device, 'bluetooth')
            bt_devices[path] = iot_device

class BtManager(Thread):

    def run(self):
        dm = GattBtManager(adapter_name='hci0')
        for device in [device_con for device_con in dm.devices() if device_con.is_connected()]:
            device.disconnect()
        dm.start_discovery()
        dm.run()

class SocketManager(Thread):

    def __init__(self):
        super(SocketManager, self).__init__()
        self.open_socket(9000)

    def open_socket(self, port):
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        self.sock.bind(('', port))
        self.sock.listen()

    @staticmethod
    def create_socket_device(dev, addr):
        """Creates a socket_devices entry that wraps the socket.

        The Manager thread will detect it being added and instantiate a corresponding
        Driver in iot_devices based on the results of the `supported` call.
        """
        _logger.debug("Creating new socket_device")
        iot_device = IoTDevice(type('', (), {'dev': dev}), 'socket')
        socket_devices[addr] = iot_device

    @staticmethod
    def replace_socket_device(dev, addr):
        """Replaces an existing socket_devices entry.

        The socket contained in the socket_devices entry is also used by the Driver
        thread defined in iot_devices that's reading and writing from it. The Driver
        thread can modify both socket_devices and iot_devices. The Manager thread can
        update iot_devices based on changes in socket_devices. In order to clean up
        the existing connection, it'll be necessary to actively close it at the TCP
        level, wait for the Driver thread to terminate in response to that, and for the
        Manager to do any iot_devices related cleanup in response.

        After this the new connection can replace the old one.
        """
        driver_thread = iot_devices.get(addr)

        if not driver_thread:
            _logger.warning("Found socket_device entry {} with no corresponding iot_device".format(addr))
            dev.close()
            return

        old_dev = socket_devices[addr].dev.dev
        _logger.debug("Closing socket: {}".format(old_dev))
        # Actively close the existing connection and do not allow receiving further
        # data. This will result in a currently blocking recv call returning b'' and
        # subsequent recv calls raising an OSError about a bad file descriptor.
        old_dev.shutdown(socket.SHUT_RD)
        old_dev.close()

        _logger.debug("Waiting for driver thread to finish")
        driver_thread.join()
        _logger.debug("Driver thread finished")

        # Shutting down the socket will result in the corresponding IngenicoDriver
        # thread terminating and removing the corresponding entries in socket_devices
        # and iot_devices. However, if we create a new socket device too soon,
        # the `devices` attribute of the Manager thread will not have registered that
        # the old socket device is gone yet. As a result, the keys of `updated_devices`
        # and `devices` might be exactly the same, which means no difference will be
        # detected and no new IngenicoDriver thread will be created. To avoid this, we
        # wait for `devices` to update first, and only after that do we create a new
        # socket device.
        _logger.debug("Waiting for Manager.devices to be updated")
        while addr in m.devices:
            time.sleep(1)
        _logger.debug("Manager.devices is updated")

        SocketManager.create_socket_device(dev, addr)

    def run(self):
        while True:
            try:
                dev, addr = self.sock.accept()
                _logger.debug("Accepted new socket connection")
                if not addr:
                    _logger.warning("Socket accept returned no address")
                    continue

                if addr[0] not in socket_devices:
                    self.create_socket_device(dev, addr[0])
                else:
                    # This can happen if the device power cycled or a network cable
                    # was temporarily unplugged: if the device tries to connect again
                    # we might still have the old connection open and it needs to be
                    # cleaned up.
                    self.replace_socket_device(dev, addr[0])
            except OSError:
                pass


class MPDManager(Thread):
    def __init__(self):
        super(MPDManager, self).__init__()
        self.devices = {}
        self.mpd_session = ctypes.c_void_p()

    def run(self):
        eftapi.EFT_CreateSession(ctypes.byref(self.mpd_session))
        eftapi.EFT_PutDeviceId(self.mpd_session, terminal_id.encode())
        while True:
            if self.terminal_connected(terminal_id):
                self.devices[terminal_id] = IoTDevice(terminal_id, 'mpd')
            elif terminal_id in self.devices:
                self.devices = {}
            time.sleep(20)

    def terminal_connected(self, terminal_id):
        eftapi.EFT_QueryStatus(self.mpd_session)
        eftapi.EFT_Complete(self.mpd_session, 1)  # Needed to read messages from driver
        device_status = ctypes.c_long()
        eftapi.EFT_GetDeviceStatusCode(self.mpd_session, ctypes.byref(device_status))
        return device_status.value in [0, 1]


conn = cups_connection()
PPDs = conn.getPPDs()
printers = conn.getPrinters()
cups_lock = Lock()  # We can only make one call to Cups at a time

mpdm = MPDManager()
terminal_id = helpers.read_file_first_line('odoo-six-payment-terminal.conf')
if terminal_id:
    try:
        subprocess.check_output(["pidof", "eftdvs"])  # Check if MPD server is running
    except subprocess.CalledProcessError:
        subprocess.Popen(["eftdvs", "/ConfigDir", "/usr/share/eftdvs/"])  # Start MPD server
    eftapi = ctypes.CDLL("eftapi.so")  # Library given by Six
    mpdm.daemon = True
    mpdm.start()
else:
    try:
        subprocess.check_call(["pkill", "-9", "eftdvs"])  # Check if MPD server is running
    except subprocess.CalledProcessError:
        pass

cm = ConnectionManager()
cm.daemon = True
cm.start()

m = Manager()
m.daemon = True
m.start()

bm = BtManager()
bm.daemon = True
bm.start()

sm = SocketManager()
sm.daemon = True
sm.start()

```

## File: controllers\__init__.py

```python
from . import driver
```

## File: drivers\DisplayDriver.py

```python
import jinja2
import json
import logging
import netifaces as ni
import os
from pathlib import Path
import subprocess
import threading
import time
import urllib3

from odoo import http
from odoo.addons.hw_drivers.tools import helpers
from odoo.addons.hw_drivers.controllers.driver import Driver, event_manager, iot_devices

try:
    from odoo.addons.hw_drivers.controllers.driver import cm
except:
    cm = None

path = os.path.realpath(os.path.join(os.path.dirname(__file__), '../views'))
loader = jinja2.FileSystemLoader(path)

jinja_env = jinja2.Environment(loader=loader, autoescape=True)
jinja_env.filters["json"] = json.dumps

pos_display_template = jinja_env.get_template('pos_display.html')

_logger = logging.getLogger(__name__)


class DisplayDriver(Driver):
    connection_type = 'display'

    def __init__(self, device):
        super(DisplayDriver, self).__init__(device)
        self._device_type = 'display'
        self._device_connection = 'hdmi'
        self._device_name = device['name']
        self.event_data = threading.Event()
        self.owner = False
        self.rendered_html = ''
        if self.device_identifier != 'distant_display':
            self._x_screen = device.get('x_screen', '0')
            self.load_url()

    @property
    def device_identifier(self):
        return self.dev['identifier']

    @classmethod
    def supported(cls, device):
        return True  # All devices with connection_type == 'display' are supported

    @classmethod
    def get_default_display(cls):
        displays = list(filter(lambda d: iot_devices[d].device_type == 'display', iot_devices))
        return len(displays) and iot_devices[displays[0]]

    def action(self, data):
        if data.get('action') == "update_url" and self.device_identifier != 'distant_display':
            self.update_url(data.get('url'))
        elif data.get('action') == "display_refresh" and self.device_identifier != 'distant_display':
            self.call_xdotools('F5')
        elif data.get('action') == "take_control":
            self.take_control(self.data['owner'], data.get('html'))
        elif data.get('action') == "customer_facing_display":
            self.update_customer_facing_display(self.data['owner'], data.get('html'))
        elif data.get('action') == "get_owner":
            self.data = {
                'value': '',
                'owner': self.owner,
            }
            event_manager.device_changed(self)

    def run(self):
        while self.device_identifier != 'distant_display':
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
                response = http.request('GET', "%s/iot/box/%s/screen_url" % (helpers.get_odoo_server_url(), helpers.get_mac_address()))
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

        with open(os.path.join(os.path.dirname(__file__), "../static/src/js/worker.js")) as js:
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
            'pairing_code': cm and cm.pairing_code,
        })

```

## File: drivers\KeyboardUSBDriver.py

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
import re
import subprocess
import time
from threading import Lock
import usb
import urllib3
try:
    from queue import Queue, Empty
except ImportError:
    from Queue import Queue, Empty  # pylint: disable=deprecated-module

from odoo import http, _
from odoo.addons.hw_proxy.controllers.main import drivers as old_drivers
from odoo.addons.hw_drivers.tools import helpers
from odoo.addons.hw_drivers.controllers.driver import event_manager, Driver, iot_devices

_logger = logging.getLogger(__name__)
xlib = ctypes.cdll.LoadLibrary('libX11.so.6')


class KeyboardUSBDriver(Driver):
    connection_type = 'usb'
    keyboard_layout_groups = []
    available_layouts = []

    def __init__(self, device):
        if not hasattr(KeyboardUSBDriver, 'display'):
            os.environ['XAUTHORITY'] = "/run/lightdm/pi/xauthority"
            KeyboardUSBDriver.display = xlib.XOpenDisplay(bytes(":0.0", "utf-8"))

        super(KeyboardUSBDriver, self).__init__(device)
        self._device_type = 'keyboard'
        self._device_connection = 'direct'
        self._device_name = self._set_name()

        # from https://github.com/xkbcommon/libxkbcommon/blob/master/test/evdev-scancodes.h
        self._scancode_to_modifier = {
            42: 'left_shift',
            54: 'right_shift',
            58: 'caps_lock',
            69: 'num_lock',
            100: 'alt_gr', # right alt
        }
        self._tracked_modifiers = {modifier: False for modifier in self._scancode_to_modifier.values()}

        self.load_layout()

        if not KeyboardUSBDriver.available_layouts:
            KeyboardUSBDriver.load_layouts_list()
        KeyboardUSBDriver.send_layouts_list()

        for device in [evdev.InputDevice(path) for path in evdev.list_devices()]:
            if (self.dev.idVendor == device.info.vendor) and (self.dev.idProduct == device.info.product):
                self.input_device = device

        device_name = self._device_name.lower()
        if 'barcode' in device_name or 'scanner' in device_name or 'reader' in device_name or self.dev.interface_protocol == '0':
            self._device_type = 'scanner'
            self._barcodes = Queue()
            self._current_barcode = ''
            self.input_device.grab()
            self.read_barcode_lock = Lock()

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
            if usb.__version__ == '1.0.0b1':
                manufacturer = usb.util.get_string(self.dev, 256, self.dev.iManufacturer)
                product = usb.util.get_string(self.dev, 256, self.dev.iProduct)
            else:
                manufacturer = usb.util.get_string(self.dev, self.dev.iManufacturer)
                product = usb.util.get_string(self.dev, self.dev.iProduct)
            return re.sub(r"[^\w \-+/*&]", '', "%s - %s" % (manufacturer, product))
        except ValueError as e:
            _logger.warning(e)
            return _('Unknown input device')

    def action(self, data):
        if data.get('action', False) == 'update_layout':
            layout = {
                'layout': data.get('layout'),
                'variant': data.get('variant'),
            }
            self._change_keyboard_layout(layout)
            self.save_layout(layout)
        else:
            self.data['value'] = ''
            event_manager.device_changed(self)

    def run(self):
        key_input = self._barcode_scanner_input if self._device_type == "scanner" else self._keyboard_input

        try:
            for event in self.input_device.read_loop():
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
                        key_input(data.scancode)

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


old_drivers['scanner'] = KeyboardUSBDriver

class KeyboardUSBController(http.Controller):
    @http.route('/hw_proxy/scanner', type='json', auth='none', cors='*')
    def get_barcode(self):
        scanners = [iot_devices[d] for d in iot_devices if iot_devices[d].device_type == "scanner"]
        if scanners:
            return scanners[0].read_next_barcode()
        time.sleep(5)
        return None

```

## File: drivers\PrinterDriver.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from base64 import b64decode
from cups import IPPError, IPP_PRINTER_IDLE, IPP_PRINTER_PROCESSING, IPP_PRINTER_STOPPED
import dbus
import logging
import netifaces as ni
import os
import io
import base64
import re
import subprocess
import tempfile
from PIL import Image, ImageOps

from odoo import http, _
from odoo.addons.hw_drivers.controllers.driver import event_manager, Driver, PPDs, conn, printers, cups_lock, iot_devices
from odoo.addons.hw_drivers.tools import helpers
from odoo.addons.hw_proxy.controllers.main import drivers as old_drivers

try:
    from odoo.addons.hw_drivers.controllers.driver import cm
except:
    cm = None

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

def cups_notification_handler(message, uri, device_id, state, reason, accepting_jobs):
    if device_id in iot_devices:
        reason = reason if reason != 'none' else None
        state_value = {
            IPP_PRINTER_IDLE: 'connected',
            IPP_PRINTER_PROCESSING: 'processing',
            IPP_PRINTER_STOPPED: 'stopped'
        }
        iot_devices[device_id].update_status(state_value[state], message, reason)

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

    def __init__(self, device):
        super(PrinterDriver, self).__init__(device)
        self._device_type = 'printer'
        self._device_connection = self.dev['device-class'].lower()
        self._device_name = self.dev['device-make-and-model']
        self.state = {
            'status': 'connecting',
            'message': 'Connecting to printer',
            'reason': None,
        }
        self.send_status()

        self.receipt_protocol = 'star' if 'STR_T' in self.dev['device-id'] else 'escpos'
        if 'direct' in self._device_connection and any(cmd in self.dev['device-id'] for cmd in ['CMD:STAR;', 'CMD:ESC/POS;']):
            self.print_status()

    @classmethod
    def supported(cls, device):
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
                if device['identifier'] not in printers:
                    conn.setPrinterInfo(device['identifier'], device['device-make-and-model'])
                    conn.enablePrinter(device['identifier'])
                    conn.acceptJobs(device['identifier'])
                    conn.setPrinterUsersAllowed(device['identifier'], ['all'])
                    conn.addPrinterOptionDefault(device['identifier'], "usb-no-reattach", "true")
                    conn.addPrinterOptionDefault(device['identifier'], "usb-unidir", "true")
                else:
                    device['device-make-and-model'] = printers[device['identifier']]['printer-info']
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
        return re.sub("[\(].*?[\)]", "", device_model).strip()

    @classmethod
    def get_status(cls):
        status = 'connected' if any(iot_devices[d].device_type == "printer" and iot_devices[d].device_connection == 'direct' for d in iot_devices) else 'disconnected'
        return {'status': status, 'messages': ''}

    @property
    def device_identifier(self):
        return self.dev['identifier']

    def action(self, data):
        if data.get('action') == 'cashbox':
            self.open_cashbox()
        elif data.get('action') == 'print_receipt':
            self.print_receipt(base64.b64decode(data['receipt']))
        else:
            self.print_raw(b64decode(data['document']))

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

    def print_receipt(self, receipt):
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

    def format_escpos(self, im):
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

        _logger.error('done')
        return raster_data + RECEIPT_PRINTER_COMMANDS['escpos']['cut']

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

        code = cm and cm.pairing_code
        if code:
            pairing_code = '\nPairing Code:\n%s\n' % code

        commands = RECEIPT_PRINTER_COMMANDS[self.receipt_protocol]
        title = commands['title'] % b'IoTBox Status'
        self.print_raw(commands['center'] + title + b'\n' + wlan.encode() + mac.encode() + ip.encode() + homepage.encode() + pairing_code.encode() + commands['cut'])

    def open_cashbox(self):
        """Sends a signal to the current printer to open the connected cashbox."""
        commands = RECEIPT_PRINTER_COMMANDS[self.receipt_protocol]
        for drawer in commands['drawers']:
            self.print_raw(drawer)


class PrinterController(http.Controller):

    @http.route('/hw_proxy/default_printer_action', type='json', auth='none', cors='*')
    def default_printer_action(self, data):
        printer = next((d for d in iot_devices if iot_devices[d].device_type == 'printer' and iot_devices[d].device_connection == 'direct'), None)
        if printer:
            iot_devices[printer].action(data)
            return True
        return False

old_drivers['printer'] = PrinterDriver

```

## File: drivers\SerialBaseDriver.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import time
import traceback
from collections import namedtuple
from threading import Lock
from contextlib import contextmanager

import serial

from odoo.tools.translate import _
from odoo.addons.hw_drivers.controllers.driver import Driver, event_manager

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

    def __init__(self, device):
        """ Attributes initialization method for `SerialDriver`.

        :param device: path to the device
        :type device: str
        """

        super().__init__(device)
        self._actions = {
            'get_status': self._push_status,
        }
        self._device_connection = 'serial'
        self._device_lock = Lock()
        self._status = {'status': self.STATUS_CONNECTING, 'message_title': '', 'message_body': ''}
        self._set_name()

    @property
    def device_identifier(self):
        return self.dev['identifier']

    def _get_raw_response(connection):
        pass

    def _push_status(self):
        """Updates the current status and pushes it to the frontend."""

        self.data['status'] = self._status
        event_manager.device_changed(self)

    def _set_name(self):
        """Tries to build the device's name based on its type and protocol name but falls back on a default name if that doesn't work."""

        try:
            name = ('%s serial %s' % (self._protocol.name, self._device_type)).title()
        except Exception:
            name = 'Unknown Serial Device'
        self._device_name = name

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
            msg = _('An error occured while performing action %s on %s') % (data, self.device_name)
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
            with serial_connection(self.dev['identifier'], self._protocol) as connection:
                self._connection = connection
                self._do_action(data)

    def run(self):
        """Continuously gets new measures from the device."""

        try:
            with serial_connection(self.dev['identifier'], self._protocol) as connection:
                self._connection = connection
                self._status['status'] = self.STATUS_CONNECTED
                self._push_status()
                while True:
                    self._take_measure()
                    time.sleep(self._protocol.newMeasureDelay)
        except Exception:
            msg = _('Error while reading %s') % self.device_name
            _logger.exception(msg)
            self._status = {'status': self.STATUS_ERROR, 'message_title': msg, 'message_body': traceback.format_exc()}
            self._push_status()

```

## File: drivers\SerialScaleDriver.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


import threading
import logging
import re
import time
from collections import namedtuple

import serial

from odoo import http
from odoo.addons.hw_proxy.controllers.main import drivers as old_drivers
from odoo.addons.hw_drivers.controllers.driver import event_manager
from odoo.addons.hw_drivers.drivers.SerialBaseDriver import SerialDriver, SerialProtocol, serial_connection


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
    measureRegexp=b"\s*([0-9.]+)kg",  # LABEL format 3 + KG in the scale settings, but Label 1/2 should work
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

    def __init__(self, device):
        self._device_type = 'scale'
        super().__init__(device)
        self._set_actions()
        self._is_reading = True

        # Ensures compatibility with older versions of Odoo
        # Only the last scale connected is kept
        global ACTIVE_SCALE
        ACTIVE_SCALE = self
        old_drivers['scale'] = ACTIVE_SCALE

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

    def __init__(self, device):
        super().__init__(device)
        self._device_manufacturer = 'Toledo'

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
    is_tested_last = True

    def __init__(self, device):
        super().__init__(device)
        self._is_reading = False
        self._last_weight_time = 0
        self._device_manufacturer = 'Adam'

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

## File: static\src\js\worker.js

```javascript
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
                        if (typeof foreign_js !== 'undefined' && $.isFunction(foreign_js)) {
                            foreign_js();
                        }
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

import netifaces
from pathlib import Path
import datetime
from OpenSSL import crypto
import urllib3
import io
import json
import logging
import os
import subprocess
import zipfile
from threading import Thread
import time

from odoo import _
from odoo.modules.module import get_resource_path

_logger = logging.getLogger(__name__)

#----------------------------------------------------------
# Helper
#----------------------------------------------------------

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
    """
    server = get_odoo_server_url()
    if server:
        path = Path('/etc/ssl/certs/nginx-cert.crt')
        if path.exists():
            with path.open('r') as f:
                cert = crypto.load_certificate(crypto.FILETYPE_PEM, f.read())
                cert_end_date = datetime.datetime.strptime(cert.get_notAfter().decode('utf-8'), "%Y%m%d%H%M%SZ") - datetime.timedelta(days=10)
                for key in cert.get_subject().get_components():
                    if key[0] == b'CN':
                        cn = key[1].decode('utf-8')
                if cn == 'OdooTempIoTBoxCertificate' or datetime.datetime.now() > cert_end_date:
                    _logger.info(_('Your certificate %s must be updated') % (cn))
                    load_certificate()
                else:
                    _logger.info(_('Your certificate %s is valid until %s') % (cn, cert_end_date))
        else:
            load_certificate()

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

                if db_branch != local_branch:
                    subprocess.check_call(["sudo", "mount", "-o", "remount,rw", "/"])
                    subprocess.check_call(["rm", "-rf", "/home/pi/odoo/addons/hw_drivers/drivers/*"])
                    subprocess.check_call(git + ['branch', '-m', db_branch])
                    subprocess.check_call(git + ['remote', 'set-branches', 'origin', db_branch])
                    os.system('/home/pi/odoo/addons/point_of_sale/tools/posbox/configuration/posbox_update.sh')
                    subprocess.check_call(["sudo", "mount", "-o", "remount,ro", "/"])
                    subprocess.check_call(["sudo", "mount", "-o", "remount,rw", "/root_bypass_ramdisks/etc/cups"])

        except Exception as e:
            _logger.error('Could not reach configured server')
            _logger.error('A error encountered : %s ' % e)

def check_image():
    """
    Check if the current image of IoT Box is up to date
    """
    url = 'http://nightly.odoo.com/master/posbox/iotbox/SHA1SUMS.txt'
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
    return subprocess.check_output(['cat', '/home/pi/iotbox_version']).decode().rstrip()

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
    if db_uuid and enterprise_code:
        url = 'https://www.odoo.com/odoo-enterprise/iot/x509'
        data = {
            'params': {
                'db_uuid': db_uuid,
                'enterprise_code': enterprise_code
            }
        }
        urllib3.disable_warnings()
        http = urllib3.PoolManager(cert_reqs='CERT_NONE')
        response = http.request(
            'POST',
            url,
            body = json.dumps(data).encode('utf8'),
            headers = {'Content-type': 'application/json', 'Accept': 'text/plain'}
        )
        result = json.loads(response.data.decode('utf8'))['result']
        if result:
            write_file('odoo-subject.conf', result['subject_cn'])
            subprocess.check_call(["sudo", "mount", "-o", "remount,rw", "/"])
            subprocess.check_call(["sudo", "mount", "-o", "remount,rw", "/root_bypass_ramdisks/"])
            Path('/etc/ssl/certs/nginx-cert.crt').write_text(result['x509_pem'])
            Path('/root_bypass_ramdisks/etc/ssl/certs/nginx-cert.crt').write_text(result['x509_pem'])
            Path('/etc/ssl/private/nginx-cert.key').write_text(result['private_key_pem'])
            Path('/root_bypass_ramdisks/etc/ssl/private/nginx-cert.key').write_text(result['private_key_pem'])
            subprocess.check_call(["sudo", "mount", "-o", "remount,ro", "/"])
            subprocess.check_call(["sudo", "mount", "-o", "remount,ro", "/root_bypass_ramdisks/"])
            subprocess.check_call(["sudo", "mount", "-o", "remount,rw", "/root_bypass_ramdisks/etc/cups"])
            subprocess.check_call(["sudo", "service", "nginx", "restart"])

def download_drivers(auto=True):
    """
    Get the drivers from the configured Odoo server
    """
    server = get_odoo_server_url()
    if server:
        urllib3.disable_warnings()
        pm = urllib3.PoolManager(cert_reqs='CERT_NONE')
        server = server + '/iot/get_drivers'
        try:
            resp = pm.request('POST', server, fields={'mac': get_mac_address(), 'auto': auto})
            if resp.data:
                subprocess.check_call(["sudo", "mount", "-o", "remount,rw", "/"])
                drivers_path = Path.home() / 'odoo/addons/hw_drivers/drivers'
                zip_file = zipfile.ZipFile(io.BytesIO(resp.data))
                zip_file.extractall(drivers_path)
                subprocess.check_call(["sudo", "mount", "-o", "remount,ro", "/"])
                subprocess.check_call(["sudo", "mount", "-o", "remount,rw", "/root_bypass_ramdisks/etc/cups"])
        except Exception as e:
            _logger.error('Could not reach configured server')
            _logger.error('A error encountered : %s ' % e)

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
    subprocess.check_call(["sudo", "mount", "-o", "remount,rw", "/"])
    path = Path.home() / filename
    if path.exists():
        path.unlink()
    subprocess.check_call(["sudo", "mount", "-o", "remount,ro", "/"])
    subprocess.check_call(["sudo", "mount", "-o", "remount,rw", "/root_bypass_ramdisks/etc/cups"])

def write_file(filename, text):
    subprocess.check_call(["sudo", "mount", "-o", "remount,rw", "/"])
    path = Path.home() / filename
    path.write_text(text)
    subprocess.check_call(["sudo", "mount", "-o", "remount,ro", "/"])
    subprocess.check_call(["sudo", "mount", "-o", "remount,rw", "/root_bypass_ramdisks/etc/cups"])

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
                <img style="width: 150px;" src="/web/static/src/img/logo_inverse_white_206px.png">
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

