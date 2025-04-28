# Odoo Module: hw_posbox_homepage

Category: Sales/Point of Sale

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'IoT Box Homepage',
    'category': 'Sales/Point of Sale',
    'sequence': 6,
    'website': 'https://www.odoo.com/app/point-of-sale-hardware',
    'summary': 'A homepage for the IoT Box',
    'description': """
IoT Box Homepage
================

This module overrides Odoo web interface to display a simple
Homepage that explains what's the iotbox and shows the status,
and where to find documentation.

If you activate this module, you won't be able to access the 
regular Odoo interface anymore.

""",
    'installable': False,
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json
import jinja2
import platform
import logging
import os
from pathlib import Path
import socket
import subprocess
import sys
import threading

from odoo import http, service, tools
from odoo.http import Response, request
from odoo.modules.module import get_resource_path
from odoo.addons.hw_drivers.connection_manager import connection_manager
from odoo.addons.hw_drivers.main import iot_devices
from odoo.addons.hw_drivers.tools import helpers
from odoo.addons.hw_drivers.server_logger import check_and_update_odoo_config_log_to_server_option, close_server_log_sender_handler, get_odoo_config_log_to_server_option
from odoo.addons.web.controllers.home import Home

_logger = logging.getLogger(__name__)


#----------------------------------------------------------
# Controllers
#----------------------------------------------------------

if hasattr(sys, 'frozen'):
    # When running on compiled windows binary, we don't have access to package loader.
    path = os.path.realpath(os.path.join(os.path.dirname(__file__), '..', 'views'))
    loader = jinja2.FileSystemLoader(path)
else:
    loader = jinja2.PackageLoader('odoo.addons.hw_posbox_homepage', "views")

jinja_env = jinja2.Environment(loader=loader, autoescape=True)
jinja_env.filters["json"] = json.dumps

homepage_template = jinja_env.get_template('homepage.html')
server_config_template = jinja_env.get_template('server_config.html')
wifi_config_template = jinja_env.get_template('wifi_config.html')
handler_list_template = jinja_env.get_template('handler_list.html')
remote_connect_template = jinja_env.get_template('remote_connect.html')
configure_wizard_template = jinja_env.get_template('configure_wizard.html')
six_payment_terminal_template = jinja_env.get_template('six_payment_terminal.html')
list_credential_template = jinja_env.get_template('list_credential.html')
upgrade_page_template = jinja_env.get_template('upgrade_page.html')

class IoTboxHomepage(Home):
    def __init__(self):
        super(IoTboxHomepage,self).__init__()
        self.updating = threading.Lock()

    def clean_partition(self):
        subprocess.check_call(['sudo', 'bash', '-c', '. /home/pi/odoo/addons/point_of_sale/tools/posbox/configuration/upgrade.sh; cleanup'])

    def get_six_terminal(self):
        terminal_id = helpers.read_file_first_line('odoo-six-payment-terminal.conf')
        return terminal_id or 'Not Configured'

    def get_homepage_data(self):
        hostname = str(socket.gethostname())
        if platform.system() == 'Linux':
            ssid = helpers.get_ssid()
            wired = helpers.read_file_first_line('/sys/class/net/eth0/operstate')
        else:
            wired = 'up'
        if wired == 'up':
            network = 'Ethernet'
        elif ssid:
            if helpers.access_point():
                network = 'Wifi access point'
            else:
                network = 'Wifi : ' + ssid
        else:
            network = 'Not Connected'

        is_certificate_ok, certificate_details = helpers.get_certificate_status()

        iot_device = []
        for device in iot_devices:
            iot_device.append({
                'name': iot_devices[device].device_name + ' : ' + str(iot_devices[device].data['value']),
                'type': iot_devices[device].device_type.replace('_', ' '),
                'identifier': iot_devices[device].device_identifier,
            })

        return {
            'hostname': hostname,
            'ip': helpers.get_ip(),
            'mac': helpers.get_mac_address(),
            'iot_device_status': iot_device,
            'server_status': helpers.get_odoo_server_url() or 'Not Configured',
            'pairing_code': connection_manager.pairing_code,
            'six_terminal': self.get_six_terminal(),
            'network_status': network,
            'version': helpers.get_version(),
            'system': platform.system(),
            'is_certificate_ok': is_certificate_ok,
            'certificate_details': certificate_details,
            }

    @http.route('/', type='http', auth='none')
    def index(self):
        wifi = Path.home() / 'wifi_network.txt'
        remote_server = Path.home() / 'odoo-remote-server.conf'
        if (wifi.exists() == False or remote_server.exists() == False) and helpers.access_point():
            return "<meta http-equiv='refresh' content='0; url=http://" + helpers.get_ip() + ":8069/steps'>"
        else:
            return homepage_template.render(self.get_homepage_data())

    @http.route('/list_handlers', type='http', auth='none', website=True, csrf=False, save_session=False)
    def list_handlers(self, **post):
        AVAILABLE_LOG_LEVELS = ('debug', 'info', 'warning', 'error')
        if request.httprequest.method == 'POST':
            need_config_save = False  # If the config file needed to be saved at the end

            # Check and update "send logs to server"
            need_config_save |= check_and_update_odoo_config_log_to_server_option(
                post.get('log-to-server') == 'on' # we use .get() as if the HTML checkbox is unchecked, no value is given in the POST request
            )

            # Check and update logging levels
            IOT_LOGGING_PREFIX = 'iot-logging-'
            INTERFACE_PREFIX = 'interface-'
            DRIVER_PREFIX = 'driver-'
            AVAILABLE_LOG_LEVELS_WITH_PARENT = AVAILABLE_LOG_LEVELS + ('parent',)
            for post_request_key, log_level_or_parent in post.items():
                if not post_request_key.startswith(IOT_LOGGING_PREFIX):
                    # probably a new post request payload argument not related to logging
                    continue
                post_request_key = post_request_key[len(IOT_LOGGING_PREFIX):]

                if post_request_key == 'root':
                    need_config_save |= self._update_logger_level('', log_level_or_parent, AVAILABLE_LOG_LEVELS)
                elif post_request_key == 'odoo':
                    need_config_save |= self._update_logger_level('odoo', log_level_or_parent, AVAILABLE_LOG_LEVELS)
                    need_config_save |= self._update_logger_level('werkzeug', log_level_or_parent if log_level_or_parent != 'debug' else 'info', AVAILABLE_LOG_LEVELS)
                elif post_request_key.startswith(INTERFACE_PREFIX):
                    logger_name = post_request_key[len(INTERFACE_PREFIX):]
                    need_config_save |= self._update_logger_level(logger_name, log_level_or_parent, AVAILABLE_LOG_LEVELS_WITH_PARENT, 'interfaces')
                elif post_request_key.startswith(DRIVER_PREFIX):
                    logger_name = post_request_key[len(DRIVER_PREFIX):]
                    need_config_save |= self._update_logger_level(logger_name, log_level_or_parent, AVAILABLE_LOG_LEVELS_WITH_PARENT, 'drivers')
                else:
                    _logger.warning('Unhandled iot logger: %s', post_request_key)

            # Update and save the config file (in case of IoT box reset)
            if need_config_save:
                with helpers.writable():
                    tools.config.save()
        drivers_list = helpers.list_file_by_os(get_resource_path('hw_drivers', 'iot_handlers', 'drivers'))
        interfaces_list = helpers.list_file_by_os(get_resource_path('hw_drivers', 'iot_handlers', 'interfaces'))
        return handler_list_template.render({
            'title': "Odoo's IoT Box - Handlers list",
            'breadcrumb': 'Handlers list',
            'drivers_list': drivers_list,
            'interfaces_list': interfaces_list,
            'server': helpers.get_odoo_server_url(),
            'is_log_to_server_activated': get_odoo_config_log_to_server_option(),
            'root_logger_log_level': self._get_logger_effective_level_str(logging.getLogger()),
            'odoo_current_log_level': self._get_logger_effective_level_str(logging.getLogger('odoo')),
            'recommended_log_level': 'warning',
            'available_log_levels': AVAILABLE_LOG_LEVELS,
            'drivers_logger_info': self._get_iot_handlers_logger(drivers_list, 'drivers'),
            'interfaces_logger_info': self._get_iot_handlers_logger(interfaces_list, 'interfaces'),
        })

    @http.route('/load_iot_handlers', type='http', auth='none', website=True)
    def load_iot_handlers(self):
        helpers.download_iot_handlers(False)
        helpers.odoo_restart(0)
        return "<meta http-equiv='refresh' content='20; url=http://" + helpers.get_ip() + ":8069/list_handlers'>"

    @http.route('/list_credential', type='http', auth='none', website=True)
    def list_credential(self):
        return list_credential_template.render({
            'title': "Odoo's IoT Box - List credential",
            'breadcrumb': 'List credential',
            'db_uuid': helpers.read_file_first_line('odoo-db-uuid.conf'),
            'enterprise_code': helpers.read_file_first_line('odoo-enterprise-code.conf'),
        })

    @http.route('/save_credential', type='http', auth='none', cors='*', csrf=False)
    def save_credential(self, db_uuid, enterprise_code):
        helpers.write_file('odoo-db-uuid.conf', db_uuid)
        helpers.write_file('odoo-enterprise-code.conf', enterprise_code)
        helpers.odoo_restart(0)
        return "<meta http-equiv='refresh' content='20; url=http://" + helpers.get_ip() + ":8069'>"

    @http.route('/clear_credential', type='http', auth='none', cors='*', csrf=False)
    def clear_credential(self):
        helpers.unlink_file('odoo-db-uuid.conf')
        helpers.unlink_file('odoo-enterprise-code.conf')
        helpers.odoo_restart(0)
        return "<meta http-equiv='refresh' content='20; url=http://" + helpers.get_ip() + ":8069'>"

    @http.route('/wifi', type='http', auth='none', website=True)
    def wifi(self):
        return wifi_config_template.render({
            'title': 'Wifi configuration',
            'breadcrumb': 'Configure Wifi',
            'loading_message': 'Connecting to Wifi',
            'ssid': helpers.get_wifi_essid(),
        })

    @http.route('/wifi_connect', type='http', auth='none', cors='*', csrf=False)
    def connect_to_wifi(self, essid, password, persistent=False):
        if persistent:
                persistent = "1"
        else:
                persistent = ""

        subprocess.check_call([get_resource_path('point_of_sale', 'tools/posbox/configuration/connect_to_wifi.sh'), essid, password, persistent])
        server = helpers.get_odoo_server_url()
        res_payload = {
            'message': 'Connecting to ' + essid,
        }
        if server:
            res_payload['server'] = {
                'url': server,
                'message': 'Redirect to Odoo Server'
            }
        else:
            res_payload['server'] = {
                'url': 'http://' + helpers.get_ip() + ':8069',
                'message': 'Redirect to IoT Box'
            }

        return json.dumps(res_payload)

    @http.route('/wifi_clear', type='http', auth='none', cors='*', csrf=False)
    def clear_wifi_configuration(self):
        helpers.unlink_file('wifi_network.txt')
        return "<meta http-equiv='refresh' content='0; url=http://" + helpers.get_ip() + ":8069'>"

    @http.route('/server_clear', type='http', auth='none', cors='*', csrf=False)
    def clear_server_configuration(self):
        helpers.unlink_file('odoo-remote-server.conf')
        close_server_log_sender_handler()
        return "<meta http-equiv='refresh' content='0; url=http://" + helpers.get_ip() + ":8069'>"

    @http.route('/handlers_clear', type='http', auth='none', cors='*', csrf=False)
    def clear_handlers_list(self):
        for directory in ['drivers', 'interfaces']:
            for file in list(Path(get_resource_path('hw_drivers', 'iot_handlers', directory)).glob('*')):
                if file.name != '__pycache__':
                    helpers.unlink_file(str(file.relative_to(*file.parts[:3])))
        return "<meta http-equiv='refresh' content='0; url=http://" + helpers.get_ip() + ":8069/list_handlers'>"

    @http.route('/server_connect', type='http', auth='none', cors='*', csrf=False)
    def connect_to_server(self, token, iotname):
        if token:
            credential = token.split('|')
            url = credential[0]
            token = credential[1]
            db_uuid = credential[2]
            enterprise_code = credential[3]
            helpers.save_conf_server(url, token, db_uuid, enterprise_code)
        else:
            url = helpers.get_odoo_server_url()
            token = helpers.get_token()
        if iotname and platform.system() == 'Linux':
            subprocess.check_call([get_resource_path('point_of_sale', 'tools/posbox/configuration/rename_iot.sh'), iotname])
        helpers.odoo_restart(5)
        return 'http://' + helpers.get_ip() + ':8069'

    @http.route('/steps', type='http', auth='none', cors='*', csrf=False)
    def step_by_step_configure_page(self):
        return configure_wizard_template.render({
            'title': 'Configure IoT Box',
            'breadcrumb': 'Configure IoT Box',
            'loading_message': 'Configuring your IoT Box',
            'ssid': helpers.get_wifi_essid(),
            'server': helpers.get_odoo_server_url() or '',
            'hostname': subprocess.check_output('hostname').decode('utf-8').strip('\n'),
        })

    @http.route('/step_configure', type='http', auth='none', cors='*', csrf=False)
    def step_by_step_configure(self, token, iotname, essid, password, persistent=False):
        if token:
            url = token.split('|')[0]
            token = token.split('|')[1]
        else:
            url = ''
        subprocess.check_call([get_resource_path('point_of_sale', 'tools/posbox/configuration/connect_to_server_wifi.sh'), url, iotname, token, essid, password, persistent])
        return url

    # Set server address
    @http.route('/server', type='http', auth='none', website=True)
    def server(self):
        return server_config_template.render({
            'title': 'IoT -> Odoo server configuration',
            'breadcrumb': 'Configure Odoo Server',
            'hostname': subprocess.check_output('hostname').decode('utf-8').strip('\n'),
            'server_status': helpers.get_odoo_server_url() or 'Not configured yet',
            'loading_message': 'Configure Domain Server'
        })

    # Get password
    @http.route('/hw_posbox_homepage/password', type='json', auth='none', methods=['POST'])
    def view_password(self):
        return helpers.generate_password()

    @http.route('/remote_connect', type='http', auth='none', cors='*')
    def remote_connect(self):
        """
        Establish a link with a customer box trough internet with a ssh tunnel
        1 - take a new auth_token on https://dashboard.ngrok.com/
        2 - copy past this auth_token on the IoT Box : http://IoT_Box:8069/remote_connect
        3 - check on ngrok the port and url to get access to the box
        4 - you can connect to the box with this command : ssh -p port -v pi@url
        """
        return remote_connect_template.render({
            'title': 'Remote debugging',
            'breadcrumb': 'Remote Debugging',
        })

    @http.route('/enable_ngrok', type='http', auth='none', cors='*', csrf=False)
    def enable_ngrok(self, auth_token):
        if subprocess.call(['pgrep', 'ngrok']) == 1:
            subprocess.Popen(['ngrok', 'tcp', '--authtoken', auth_token, '--log', '/tmp/ngrok.log', '22'])
            return 'starting with ' + auth_token
        else:
            return 'already running'

    @http.route('/six_payment_terminal', type='http', auth='none', cors='*', csrf=False)
    def six_payment_terminal(self):
        return six_payment_terminal_template.render({
            'title': 'Six Payment Terminal',
            'breadcrumb': 'Six Payment Terminal',
            'terminalId': self.get_six_terminal(),
        })

    @http.route('/six_payment_terminal_add', type='http', auth='none', cors='*', csrf=False)
    def add_six_payment_terminal(self, terminal_id):
        if terminal_id.isdigit():
            helpers.write_file('odoo-six-payment-terminal.conf', terminal_id)
            service.server.restart()
        else:
            _logger.warning('Ignoring invalid Six TID: "%s". Only digits are allowed', terminal_id)
            self.clear_six_payment_terminal()
        return 'http://' + helpers.get_ip() + ':8069'

    @http.route('/six_payment_terminal_clear', type='http', auth='none', cors='*', csrf=False)
    def clear_six_payment_terminal(self):
        helpers.unlink_file('odoo-six-payment-terminal.conf')
        service.server.restart()
        return "<meta http-equiv='refresh' content='0; url=http://" + helpers.get_ip() + ":8069'>"

    @http.route('/hw_proxy/upgrade', type='http', auth='none', )
    def upgrade(self):
        commit = subprocess.check_output(["git", "--work-tree=/home/pi/odoo/", "--git-dir=/home/pi/odoo/.git", "log", "-1"]).decode('utf-8').replace("\n", "<br/>")
        flashToVersion = helpers.check_image()
        actualVersion = helpers.get_version()
        if flashToVersion:
            flashToVersion = '%s.%s' % (flashToVersion.get('major', ''), flashToVersion.get('minor', ''))
        return upgrade_page_template.render({
            'title': "Odoo's IoTBox - Software Upgrade",
            'breadcrumb': 'IoT Box Software Upgrade',
            'loading_message': 'Updating IoT box',
            'commit': commit,
            'flashToVersion': flashToVersion,
            'actualVersion': actualVersion,
        })

    @http.route('/hw_proxy/perform_upgrade', type='http', auth='none')
    def perform_upgrade(self):
        self.updating.acquire()
        os.system('/home/pi/odoo/addons/point_of_sale/tools/posbox/configuration/posbox_update.sh')
        self.updating.release()
        return 'SUCCESS'

    @http.route('/hw_proxy/get_version', type='http', auth='none')
    def check_version(self):
        return helpers.get_version()

    @http.route('/hw_proxy/perform_flashing_create_partition', type='http', auth='none')
    def perform_flashing_create_partition(self):
        try:
            response = subprocess.check_output(['sudo', 'bash', '-c', '. /home/pi/odoo/addons/point_of_sale/tools/posbox/configuration/upgrade.sh; create_partition']).decode().split('\n')[-2]
            if response in ['Error_Card_Size', 'Error_Upgrade_Already_Started']:
                raise Exception(response)
            return Response('success', status=200)
        except subprocess.CalledProcessError as e:
            raise Exception(e.output)
        except Exception as e:
            _logger.exception("Flashing create partition failed")
            return Response(str(e), status=500)

    @http.route('/hw_proxy/perform_flashing_download_raspios', type='http', auth='none')
    def perform_flashing_download_raspios(self):
        try:
            response = subprocess.check_output(['sudo', 'bash', '-c', '. /home/pi/odoo/addons/point_of_sale/tools/posbox/configuration/upgrade.sh; download_raspios']).decode().split('\n')[-2]
            if response == 'Error_Raspios_Download':
                raise Exception(response)
            return Response('success', status=200)
        except subprocess.CalledProcessError as e:
            raise Exception(e.output)
        except Exception as e:
            self.clean_partition()
            _logger.exception("Flashing download raspios failed")
            return Response(str(e), status=500)

    @http.route('/hw_proxy/perform_flashing_copy_raspios', type='http', auth='none')
    def perform_flashing_copy_raspios(self):
        try:
            response = subprocess.check_output(['sudo', 'bash', '-c', '. /home/pi/odoo/addons/point_of_sale/tools/posbox/configuration/upgrade.sh; copy_raspios']).decode().split('\n')[-2]
            if response == 'Error_Iotbox_Download':
                raise Exception(response)
            return Response('success', status=200)
        except subprocess.CalledProcessError as e:
            raise Exception(e.output)
        except Exception as e:
            self.clean_partition()
            _logger.exception("Flashing copy raspios failed")
            return Response(str(e), status=500)

    @http.route('/iot_restart_odoo_or_reboot', type='json', auth='none', cors='*', csrf=False)
    def iot_restart_odoo_or_reboot(self, action):
        """ Reboots the IoT Box / restarts Odoo on it depending on chosen 'action' argument"""
        try:
            if action == 'restart_odoo':
                helpers.odoo_restart(3)
            else:
                subprocess.call(['sudo', 'reboot'])
            return 'success'
        except Exception as e:
            _logger.exception("Failed to restart/reboot the IoT")
            return str(e)

    def _get_logger_effective_level_str(self, logger):
        return logging.getLevelName(logger.getEffectiveLevel()).lower()

    def _get_iot_handler_logger(self, handler_name, handler_folder_name):
        """
        Get Odoo Iot logger given an IoT handler name
        :param handler_name: name of the IoT handler
        :param handler_folder_name: IoT handler folder name (interfaces or drivers)
        :return: logger if any, False otherwise
        """
        odoo_addon_handler_path = helpers.compute_iot_handlers_addon_name(handler_folder_name, handler_name)
        return odoo_addon_handler_path in logging.Logger.manager.loggerDict.keys() and \
               logging.getLogger(odoo_addon_handler_path)

    def _update_logger_level(self, logger_name, new_level, available_log_levels, handler_folder=False):
        """
        Update (if necessary) Odoo's configuration and logger to the given logger_name to the given level.
        The responsibility of saving the config file is not managed here.
        :param logger_name: name of the logging logger to change level
        :param new_level: new log level to set for this logger
        :param available_log_levels: iterable of logs levels allowed (for initial check)
        :param handler_folder: optional string of the IoT handler folder name ('interfaces' or 'drivers')
        :return: wherever some changes were performed or not on the config
        """
        if new_level not in available_log_levels:
            _logger.warning('Unknown level to set on logger %s: %s', logger_name, new_level)
            return False

        if handler_folder:
            logger = self._get_iot_handler_logger(logger_name, handler_folder)
            if not logger:
                _logger.warning('Unable to change log level for logger %s as logger missing', logger_name)
                return False
            logger_name = logger.name

        ODOO_TOOL_CONFIG_HANDLER_NAME = 'log_handler'
        LOG_HANDLERS = tools.config[ODOO_TOOL_CONFIG_HANDLER_NAME]
        LOGGER_PREFIX = logger_name + ':'
        IS_NEW_LEVEL_PARENT = new_level == 'parent'

        if not IS_NEW_LEVEL_PARENT:
            intended_to_find = LOGGER_PREFIX + new_level.upper()
            if intended_to_find in LOG_HANDLERS:
                # There is nothing to do, the entry is already inside
                return False

        # We remove every occurrence for the given logger
        log_handlers_without_logger = [
            log_handler for log_handler in LOG_HANDLERS if not log_handler.startswith(LOGGER_PREFIX)
        ]

        if IS_NEW_LEVEL_PARENT:
            # We must check that there is no existing entries using this logger (whatever the level)
            if len(log_handlers_without_logger) == len(LOG_HANDLERS):
                return False

        # We add if necessary new logger entry
        # If it is "parent" it means we want it to inherit from the parent logger.
        # In order to do this we have to make sure that no entries for the logger exists in the
        # `log_handler` (which is the case at this point as long as we don't re-add an entry)
        tools.config[ODOO_TOOL_CONFIG_HANDLER_NAME] = log_handlers_without_logger
        new_level_upper_case = new_level.upper()
        if not IS_NEW_LEVEL_PARENT:
            new_entry = [LOGGER_PREFIX + new_level_upper_case]
            tools.config[ODOO_TOOL_CONFIG_HANDLER_NAME] += new_entry
            _logger.debug('Adding to odoo config log_handler: %s', new_entry)

        # Update the logger dynamically
        real_new_level = logging.NOTSET if IS_NEW_LEVEL_PARENT else new_level_upper_case
        _logger.debug('Change logger %s level to %s', logger_name, real_new_level)
        logging.getLogger(logger_name).setLevel(real_new_level)
        return True

    def _get_iot_handlers_logger(self, handlers_name, iot_handler_folder_name):
        """
        :param handlers_name: List of IoT handler string to search the loggers of
        :param iot_handler_folder_name: name of the handler folder ('interfaces' or 'drivers')
        :return:
        {
            <iot_handler_name_1> : {
                'level': <logger_level_1>,
                'is_using_parent_level': <logger_use_parent_level_or_not_1>,
                'parent_name': <logger_parent_name_1>,
            },
            ...
        }
        """
        handlers_loggers_level = dict()
        for handler_name in handlers_name:
            handler_logger = self._get_iot_handler_logger(handler_name, iot_handler_folder_name)
            if not handler_logger:
                # Might happen if the file didn't define a logger (or not init yet)
                handlers_loggers_level[handler_name] = False
                _logger.debug('Unable to find logger for handler %s', handler_name)
                continue
            logger_parent = handler_logger.parent
            handlers_loggers_level[handler_name] = {
                'level': self._get_logger_effective_level_str(handler_logger),
                'is_using_parent_level': handler_logger.level == logging.NOTSET,
                'parent_name': logger_parent.name,
                'parent_level': self._get_logger_effective_level_str(logger_parent),
            }
        return handlers_loggers_level

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: views\configure_wizard.html

```html
{% extends "layout.html" %}
{% from "loading.html" import loading_block_ui %}
{% block head %}
    <script>
        $(document).ready(function () {
            function changePage(key) {
                $('.progressbar li[data-key=' + key + ']').prevAll().addClass('completed');
                $('.progressbar li[data-key=' + key + ']').nextAll().removeClass('active completed');
                $('.progressbar li[data-key=' + key + ']').addClass('active').removeClass('completed');
                $('.config-steps.active').removeClass('active').addClass('o_hide');
                $('.config-steps[data-key=' + key + ']').removeClass('o_hide').addClass('active');
            }
            $('.next-btn').on('click', function (ev) {
                changePage($(ev.target).data('key'));
            });
            $('#config-form').submit(function(e){
                e.preventDefault();
                $('.loading-block').removeClass('o_hide');
                $.ajax({
                    url: '/step_configure',
                    type: 'post',
                    data: $('#config-form').serialize(),
                }).done(function (url) {
                    $('.loading-block').addClass('o_hide');
                    changePage('done');
                    if(url) {
                        if ($('#iotname')[0].defaultValue == $('#iotname')[0].value){
                            var cpt = 30;
                        }else{
                            var cpt = 100;
                        }
                        setInterval(function(){
                            if(cpt === 0){
                                window.location = url
                            } else {
                                $('.redirect-message').html('You will be redirected to <a href="'+ url +'">' + url + '</a> in <b>' + cpt + '</b> seconds');
                                --cpt;
                            }
                        } , 1000);
                    }
                }).fail(function () {
                    $('.error-message').text('Error in submitting data');
                    $('.loading-block').addClass('o_hide');
                });
            });
        });
    </script>
    <style>
        .config-steps .title {
            font-weight: bold;
            margin-bottom: 10px;
        }
        .progressbar {
            counter-reset: step;
            z-index: 1;
            position: relative;
            display: inline-block;
            width: 100%;
            padding: 0;
        }
        .progressbar li{
            list-style-type: none;
            float: left;
            width: 33.33%;
            position:relative;
            text-align: center;
            font-size: 0.8rem;
        }
        .progressbar li:before {
            content:counter(step);
            counter-increment: step;
            height:30px;
            width:30px;
            line-height: 30px;
            border: 2px solid #ddd;
            display:block;
            text-align: center;
            margin: 0 auto 6px auto;
            border-radius: 50%;
            background-color: white;
            color: #ddd;
            font-size: 1rem;
        }
        .progressbar li:after {
            content:'';
            position: absolute;
            width:100%;
            height:2px;
            background-color: #ddd;
            top: 15px;
            left: -50%;
            z-index: -1;
        }
        .progressbar li:first-child:after {
            content:none;
        }
        .progressbar li.active, .progressbar li.completed {
            color:#875A7B;
        }
        .progressbar li:last-child:before {
            content: '✔';
        }
        .progressbar li.active:before {
            border-color:#875A7B;
            background-color:#875A7B;
            color: #fff;
        }
        .progressbar li.completed:before{
            border-color:#875A7B;
            background-color: #fff;
            color: #875A7B;
        }
        .progressbar li.active + li:after{
            background-color:#875A7B;
        }
        .footer-buttons {
            display: inline-block;
            width: 100%;
            margin-top: 20px;
        }
    </style>
{% endblock %}
{% block content %}
    <h2 class="text-center">Configure IoT Box</h2>
    <ul class="progressbar">
        <li class="active" data-key="server">Connect to Odoo</li>
        <li data-key="wifi">Connect to Internet</li>
        <li data-key="done">Done</li>
    </ul>
    <form id="config-form" style="margin-top: 20px;" action='/step_configure' method='POST'>
        <div>
            <div class="config-steps active" data-key="server">
                <table align="center">
                    <tr>
                        <td>IoT Box Name</td>
                        <td><input type="text" id="iotname" name="iotname" value="{{ hostname }}"></td>
                    </tr>
                    <tr>
                        <td>Server token</td>
                        <td><input type="text" name="token" value="{{ server }}" placeholder="Paste your copied token here"></td>
                    </tr>
                </table>
                <div class="text-center font-small" style="margin: 10px auto;">
                    Server token is not mandatory for the community version.
                </div>
                <div class="footer-buttons">
                    <a class="btn next-btn" style="float: right" data-key="wifi">Next</a>
                </div>
            </div>
            <div class="config-steps wifi-step o_hide" data-key="wifi">
                <table align="center">
                    <tr>
                        <td>Wifi Network</td>
                        <td>
                            <select name="essid">
                                {% for id in ssid -%}
                                    <option value="{{ id }}">{{ id }}</option>
                                {%- endfor %}
                            </select>
                        </td>
                    </tr>
                    <tr>
                        <td>Password</td>
                        <td><input type="password" name="password" placeholder="Optional"/></td>
                    </tr>
                    <input type="hidden" name="persistent" value="True"/>
                </table>
                <div class="footer-buttons">
                    <a class="btn next-btn" data-key="server">Previous</a>
                    <input class="btn" style="float: right" type="submit" value="Connect"/>
                </div>
            </div>
            <div class="config-steps o_hide" data-key="done">
                <h3 class="text-center" style="margin: 0;">✔ Nice! Your configuration is done.</h3>
                <p class="text-center redirect-message" />
            </div>
        </div>
        {{ loading_block_ui(loading_message) }}
    </form>
{% endblock %}

```

## File: views\handler_list.html

```html
{% extends "layout.html" %}
{% from "loading.html" import loading_block_ui %}
{% block head %}
    <script>
        $(document).ready(function () {
            $('#load_handler_btn').on('click', function(e){
                e.preventDefault();
                $('.loading-block').removeClass('o_hide');
                $.ajax({
                    url: '/load_iot_handlers',
                }).done(function () {
                    $('.message-status').html('Handlers loaded successfully <br> Refreshing page');
                    setTimeout(function () {
                        location.reload(true);
                    }, 25000);
                }).fail(function () {
                    setTimeout(function () {
                        location.reload(true);
                    }, 25000);
                });
            });
        });
    </script>
{% endblock %}
{% block content %}
    <h2 class="text-center text-green">Logging</h2>
    <form method="post">
        <input type="checkbox" id="log-to-server" name="log-to-server" {{ "checked" if is_log_to_server_activated }} />
        <label for="log-to-server">IoT logs automatically send to server logs</label>
        <div style="text-align: left;display:block;">
            <label for="iot-logging-root">Root log level (Current value: {{root_logger_log_level}}):</label>
            <select name="iot-logging-root" id="iot-logging-root">
            {% for log_level in available_log_levels %}
                <option value="{{ log_level }}" {{ "selected" if log_level == root_logger_log_level }}>
                    {{ log_level | capitalize }} {{ "(Recommended)" if log_level == recommended_log_level }}
                </option>
            {% endfor %}
            </select>

            <label for="iot-logging-odoo">Odoo log level (Current value: {{odoo_current_log_level}}):</label>
            <select name="iot-logging-odoo" id="iot-logging-odoo">
            {% for log_level in available_log_levels %}
                <option value="{{ log_level }}" {{ "selected" if log_level == odoo_current_log_level }}>
                    {{ log_level | capitalize }} {{ "(Recommended)" if log_level == recommended_log_level }}
                </option>
            {% endfor %}
            </select>
        </div>

        <h2 class="text-center text-green">Interfaces list</h2>
        <table align="center" width="80%" cellpadding="3">
            <tr>
                <th>Name</th>
                <th>Log Level</th>
            </tr>
            {% for interface in interfaces_list -%}
                <tr>
                    <td>{{ interface }}</td>
                    <td>
                        {% set interface_logger_info = interfaces_logger_info[interface] %}
                        {% if interface_logger_info != False %}
                            <select name="iot-logging-interface-{{interface}}">
                                <option value="parent" {{ "selected" if interface_logger_info.is_using_parent_level }}>
                                    Same as {{ interface_logger_info.parent_name | capitalize }} ({{ interface_logger_info.parent_level | capitalize }})
                                </option>
                                <option style="font-size: 1pt; background-color: black;" disabled>&nbsp;</option>
                                {% for log_level in available_log_levels %}
                                <option value="{{ log_level }}" {{ "selected" if not interface_logger_info.is_using_parent_level and log_level == interface_logger_info.level }}>
                                    {{ log_level | capitalize }}
                                </option>
                                {% endfor %}
                            </select>
                        {% else %}
                            <span class="font-small">Logger uninitialised</span>
                        {% endif %}
                    </td>
                </tr>
            {%- endfor %}
        </table>

        <h2 class="text-center text-green">Drivers list</h2>
        <table align="center" width="80%" cellpadding="3">
            <tr>
                <th>Name</th>
                <th>Log Level</th>
            </tr>
            {% for driver in drivers_list -%}
                <tr>
                    <td>{{ driver }}</td>
                    <td>
                        {% set driver_logger_info = drivers_logger_info[driver] %}
                        {% if driver_logger_info != False %}
                            <select name="iot-logging-driver-{{driver}}">
                                <option value="parent" {{ "selected" if driver_logger_info.is_using_parent_level }}>
                                    Same as {{ driver_logger_info.parent_name | capitalize }} ({{ driver_logger_info.parent_level | capitalize }})
                                </option>
                                <option style="font-size: 1pt; background-color: black;" disabled>&nbsp;</option>
                                {% for log_level in available_log_levels %}
                                <option value="{{ log_level }}" {{ "selected" if not driver_logger_info.is_using_parent_level and log_level == driver_logger_info.level }}>
                                    {{ log_level | capitalize }}
                                </option>
                                {% endfor %}
                            </select>
                        {% else %}
                            <span class="font-small">Logger uninitialised</span>
                        {% endif %}
                    </td>
                </tr>
            {%- endfor %}
        </table>

        <div style="margin-top: 20px;" class="text-center">
            {% if server %}
                <a id="load_handler_btn" class="btn" href='/load_iot_handlers'>Load handlers</a>
            {% endif %}
            <input class="btn" type="submit" value="Update Logs Configuration"/>
        </div>
    </form>
    {% if server %}
        <div class="text-center font-small" style="margin: 10px auto;">
            You can clear the handlers configuration
            <form style="display: inline-block;margin-left: 4px;" action='/handlers_clear'>
                <input class="btn btn-sm" type="submit" value="Clear"/>
            </form>
        </div>
    {% endif %}
    {{ loading_block_ui('Loading Handlers') }}
{% endblock %}

```

## File: views\homepage.html

```html
{% extends "layout.html" %}
{% from "loading.html" import loading_block_ui %}
{% block head %}
<style>
    .btn-sm-restart {
        display: flex;
        min-width: 100%;
        justify-content: center;
    }
    .item-restart {
        display: flex;
        flex-direction: column;
        margin-left: auto;
        max-width: 100%;
    }
    .text-green-primary {
        display: flex;
        width: 100%;
        margin-top: 30px;
        margin-bottom: 30px;
        justify-content: center;
    }
    table {
        width: 100%;
        border-collapse: collapse;
    }
    table tr {
        border-bottom: 1px solid #f1f1f1;
    }
    table tr:last-child {
        border-width: 0px;
    }
    table td {
        padding: 8px;
        border-left: 1px solid #f1f1f1;
    }
    table td:first-child {
        border-left: 0;
    }
    td.heading {
        font-weight: bold;
        vertical-align: top;
        width: 30%;
        text-align: left;
    }
    .device-status {
        margin: 6px 0;
    }
    .device-status .identifier {
        font-size: 0.8rem;
        max-width: 350px;
    }
    .device-status .indicator {
        margin-left: 4px;
        font-size: 0.7rem;
        text-transform: uppercase;
    }
    .device-status .device {
        font-weight: 500;
    }
    .collapse .collapsible{
        border: 1px solid #f1f1f1;
    }
    .collapse .title {
        position: relative;
        color: #00a09d;
        cursor: pointer;
        padding: 8px;
    }
    .collapse  .active, .collapse .title:hover {
        background-color: #f1f1f1;
        color: #006d6b;
    }
    .collapse  .content {
        padding: 0 8px;
        max-height: 0;
        overflow: hidden;
        transition: max-height 0.2s ease-out;
    }
    .arrow-down::after {
        content: '\25bc';
        padding: 0 10px;
        position: absolute;
        right: 0;
    }
    .arrow-up::after {
        content: '\25b2';
        padding: 0 10px;
        position: absolute;
        right: 0;
    }
    .warn-tr {
        color: #856404;
        background-color: #fff3cd;
        border: 2px solid #f3e4ce;
    }
</style>
<script>
    $(document).ready(function () {
        $('.collapsible .title').on('click', function (ev) {
            $(ev.target).toggleClass('active');
            $(ev.target).toggleClass('arrow-down arrow-up');
            var content = $(ev.target).next('.content');
            var maxHeight = ( content.css('max-height') === '0px' ? content.prop('scrollHeight') : 0) + 'px';
            content.css('max-height', maxHeight);
        });
    });

    function display_error_and_clear_interval(interval, xhrStatus, thrownError) {
        /// Displays the error message and stops sending requests to the server
        if (interval) {
            clearInterval(interval);
        }
        $('.loading-block').addClass('o_hide');
        $('.error-message').text(xhrStatus + ": " + thrownError);
    }

    function restart_odoo_or_reboot(action) {
        /// Call restart method on server, then ping it until restarting is finished
        /// If an error is encountered, display it and stop
        $('.loading-block').removeClass('o_hide');
        $('.message-title').text('Restarting');
        $('.message-status').text('Please wait');
        $.ajax({
            url: '/iot_restart_odoo_or_reboot/',
            type: 'post',
            contentType: 'application/json',
            data: JSON.stringify({ params: {action: action} }),
            timeout: 15000,
        }).done(function(data) {
            if (data.result == 'success') {
                const interval = setInterval(function() {
                    $.ajax({
                        url:'/',
                        timeout: 4000
                    }).done(function() {
                        location.reload();
                    }).fail(function(xhr, textStatus, thrownError) {
                        if (xhr.status) {
                            display_error_and_clear_interval(interval, xhr.status, thrownError);
                        }
                    })
                }, 4000)
                setTimeout(function(){
                    display_error_and_clear_interval(interval, '0', 'timeout');
                }, 600000);
            }
            else {
                display_error_and_clear_interval(interval, 'Error', data.result);
            }
        }).fail(function(xhr, textStatus, thrownError) {
            display_error_and_clear_interval(null, xhr.status, thrownError);
        })
    }
</script>
{% endblock %}
{% block content %}
    <div class="collapse item-restart">
        <div class="collapsible item-restart">
            <div class="title arrow-down">Restart</div>
            <div class="content">
                <div class="device-status">
                    {% if system == "Linux" %}
                        <button class="btn btn-sm btn-sm-restart" onclick="restart_odoo_or_reboot('reboot_iot_box')">Reboot the IoT Box</button>
                    {% endif %}
                    <button class="btn btn-sm btn-sm-restart" onclick="restart_odoo_or_reboot('restart_odoo')">Restart Odoo service</button>
                </div>
            </div>
        </div>
    </div>
    <h2 class="text-center text-green">Your IoT Box is up and running</h2>
    <table align="center" cellpadding="3">
        <tr>
            <td class="heading">Name</td>
            <td> {{ hostname }} {% if system == "Linux" %}<a class="btn btn-sm float-right" href='/server'>configure</a>{% endif %}</td>
        </tr>
        <tr>
            <td class="heading">Version</td>
            <td> {{ version }} {% if system == "Linux" %}<a class="btn btn-sm float-right" href='/hw_proxy/upgrade/'>update</a>{% endif %}</td>
        </tr>
        <tr>
            <td class="heading">IP Address</td>
            <td>{{ ip }}</a></td>
        </tr>
        <tr>
            <td class="heading">Mac Address</td>
            <td> {{ mac }}</td>
        </tr>
        <tr>
            <td class="heading">Network</td>
            <td>{{ network_status }} {% if system == "Linux" %}<a class="btn btn-sm float-right" href='/wifi'>configure wifi</a>{% endif %}</td>
        </tr>
        <tr>
            <td class="heading">Server</td>
            <td><a href='{{ server_status }}' target=_blank>{{ server_status }}<a class="btn btn-sm float-end" href='/server'>configure</a></td>
        </tr>
        <tr class="{{ 'warn-tr' if not is_certificate_ok }}">
            <td class="heading">HTTPS certificate</td>
            <td>
                {% if is_certificate_ok %}
                    <details>
                        <summary>OK</summary>
                        <code>{{ certificate_details }}</code>
                    </details>
                {% else %}
                    Error code:
                    {% set error_code = certificate_details.split(' ') | first | replace("_", "-") | lower %}
                    {% set doc_url = 'https://www.odoo.com/documentation/master/applications/productivity/iot/config/https_certificate_iot.html#' ~ error_code %}
                    <a target="_blank" class="btn btn-sm float-end" href="{{ doc_url }}">help</a>
                    <br/>
                    <code style="white-space: pre-wrap;">{{ certificate_details }}</code>
                {% endif %}
            </td>
        </tr>
        {% if server_status != "Not Configured" %}
        <tr>
            <td class="heading">Six payment terminal</td>
            <td>{{ six_terminal }} <a class="btn btn-sm float-end" href='/six_payment_terminal'>configure</a></td>
        </tr>
        {% endif %}
        {% if pairing_code %}
        <tr>
            <td class="heading">Pairing code</td>
            <td>{{ pairing_code }}</td>
        </tr>
        {% endif %}
        <tr>
            <td class="heading">IOT Device</td>
            <td>
                <div class="collapse">
                    {% if iot_device_status|length == 0 %}
                        No Device Found
                    {% endif %}
                    {% for iot_devices in iot_device_status|groupby('type') %}
                        <div class="collapsible">
                            <div class="title arrow-down">{{ iot_devices.grouper|capitalize }}s</div>
                            <div class="content">
                                {% for device in iot_devices.list %}
                                    <div class="device-status">
                                        <span class="device">{{ device['name'] }}</span>
                                        <div class="identifier">{{ device['identifier'] }}</div>
                                    </div>
                                {% endfor %}
                            </div>
                        </div>
                    {% endfor %}
                </div>
                <br><center><a class="btn btn-sm" href='/list_handlers'>handlers list</a></center>
            </td>
        </tr>
    </table>
    <div style="margin: 20px auto 10px auto;" class="text-center">
        <a class="btn" href='/point_of_sale/display'>POS Display</a>
        {% if system == "Linux" %}
            <a class="btn" style="margin-left: 10px;" href='/remote_connect'>Remote Debug</a>
            <a target="_blank" class="btn" style="margin-left: 10px;" href="http://{{ ip }}:631">Printers server</a>
        {% endif %}
        {% if server_status != "Not Configured" %}
        <a class="btn" style="margin-left: 10px;" href='/list_credential'>Credential</a>
        {% endif %}
    </div>
    {{ loading_block_ui(loading_message) }}
{% endblock %}

```

## File: views\layout.html

```html
<!DOCTYPE html>
<html>
    <head>
        <meta http-equiv="cache-control" content="no-cache" />
        <meta http-equiv="pragma" content="no-cache" />
        <title>{{ title or "Odoo's IoT Box" }}</title>
        <script type="text/javascript" src="/web/static/lib/jquery/jquery.js"></script>
        <style>
            body {
                width: 600px;
                margin: 30px auto;
                font-family: sans-serif;
                text-align: justify;
                color: #6B6B6B;
                background-color: #f1f1f1;
            }
            .text-green {
                color: #28a745;
            }
            .text-red {
                color: #dc3545;
            }
            .text-blue {
                color: #007bff;
            }
            .text-center {
                text-align: center;
            }
            .float-end {
                float: right;
            }
            .btn {
                display: inline-block;
                padding: 8px 15px;
                border: 1px solid #dadada;
                border-radius: 3px;
                font-weight: bold;
                font-size: 0.8rem;
                background: #fff;
                color: #00a09d;
                cursor: pointer;
            }
            .btn-sm {
                padding: 4px 8px;
                font-size: 1.0rem;
                font-weight: normal;
            }
            .btn:hover {
                background-color: #f1f1f1;
            }
            a {
                text-decoration: none;
                color: #00a09d;
            }
            a:hover {
                color: #006d6b;
            }
            .container {
                padding: 10px 20px;
                background: #ffffff;
                border-radius: 8px;
                box-shadow: 0 1px 1px 0 rgba(0, 0, 0, 0.17);
            }
            .breadcrumb {
                margin-bottom: 10px;
                font-size: 0.9rem;
            }
            input[type="text"], input[type="password"] {
                padding: 6px 12px;
                font-size: 1rem;
                border: 1px solid #ccc;
                border-radius: 3px;
                color: inherit;
            }
            input::placeholder {
                color: #ccc;
                opacity: 1; /* Firefox */
            }
            select {
                padding: 6px 12px;
                font-size: 1rem;
                border: 1px solid #ccc;
                border-radius: 3px;
                color: inherit;
                background: #ffffff;
                width: 100%;
            }
            .o_hide {
                display: none;
            }
            .font-small {
                font-size: 0.8rem;
            }
            .footer {
                margin-top: 12px;
                text-align: right;
            }
            .footer a {
                margin-left: 8px;
            }
            .loading-block {
                position: absolute;
                background-color: #0a060661;
                top: 0;
                left: 0;
                right: 0;
                bottom: 0;
                z-index: 9999;
            }
            .loading-message-block {
                text-align: center;
                position: absolute;
                top: 50%;
                left: 50%;
                -webkit-transform: translate(-50%, -50%);
                transform: translate(-50%, -50%);
            }
            .loading-message {
                font-size: 14px;
                line-height: 20px;
                color:white
            }
            @keyframes spin {
                from {transform:rotate(0deg);}
                to {transform:rotate(360deg);}
            }
        </style>
        {% block head %}{% endblock %}
    </head>
    <body>
        {%if breadcrumb %}
            <div class="breadcrumb"><a href="/">Home</a> / <span>{{ breadcrumb }}</span></div>
        {% endif %}
        <div class="container">
            {% block content %}{% endblock %}
            <p class="error-message text-red" style="text-align: right;" />
        </div>
        <div class="footer">
            <a href='https://www.odoo.com/help'>Help</a>
            <a href='https://www.odoo.com/documentation/16.0/applications/productivity/iot.html'>Documentation</a>
        </div>
    </body>
</html>

```

## File: views\list_credential.html

```html
{% extends "layout.html" %}
{% from "loading.html" import loading_block_ui %}
{% block head %}
    <script>
        var _onQueryDone = function () {
            $('.message-status').html('Updated configuration <br> Refreshing page');
            setTimeout(function () {
                location.reload();
            }, 30000);
        }
        $(document).ready(function () {
            $('#list-credential').submit(function(e){
                e.preventDefault();
                $('.loading-block').removeClass('o_hide');
                $.ajax({
                    url: '/save_credential',
                    type: 'post',
                    data: $('#list-credential').serialize(),
                }).always(_onQueryDone);
            });
        $('#credential-clear').submit(function(e){
            e.preventDefault();
            $('.loading-block').removeClass('o_hide');
            $.ajax({
                url: '/clear_credential',
                type: 'get',
            }).always(_onQueryDone);
        });
        });
    </script>
{% endblock %}
{% block content %}
    <h2 class="text-center">List Credential</h2>
    <p>
        Set the DB UUID and your Contract Number you want to use.
    </p>
    <form id="list-credential" action='/save_credential' method='POST'>
        <table align="center">
            <tr>
                <td>DB uuid</td>
                <td><input type="text" name="db_uuid" value="{{ db_uuid }}"></td>
            </tr>
            <tr>
                <td>Contract Number</td>
                <td><input type="text" name="enterprise_code" value="{{ enterprise_code }}"></td>
            </tr>
            <tr>
                <td/>
                <td><input class="btn" type="submit" value="Save"/></td>
            </tr>
        </table>
        {{ loading_block_ui(loading_message) }}
    </form>
    {% if db_uuid or enterprise_code %}
        <p class="text-center font-small">
            Current DB uuid: <strong>{{ db_uuid }}</strong>
        </p>
        <p class="text-center font-small">
            Current Contract Number: <strong>{{ enterprise_code }}</strong>
        </p>
        <div class="text-center font-small" style="margin: 10px auto;">
            You can clear the credential configuration
            <form id="credential-clear" style="display: inline-block;margin-left: 4px;">
                <input class="btn btn-sm" type="submit" value="Clear"/>
            </form>
        </div>
    {% endif %}
{% endblock %}

```

## File: views\loading.html

```html
{% macro loading_block_ui(message) %}
<div class="loading-block o_hide">
    <div class="loading-message-block">
        <div style="height: 50px">
            <img src="/web/static/img/spin.png" style="animation: spin 4s infinite linear;" alt="Loading...">
        </div>
        <br>
        <div class="loading-message">
            <span class="message-title">Please wait..</span><br>
            <span class="message-status">{{ message }}</span>
        </div>
    </div>
</div>
{% endmacro %}

```

## File: views\remote_connect.html

```html
{% extends "layout.html" %}
{% block head %}
    <script>
        $(function () {
            var upgrading = false;
            $('#enable_debug').click(function () {
                var auth_token = $('#auth_token').val();
                if (auth_token == "") {
                    alert('Please provide an authentication token.');
                } else {
                    $.ajax({
                        url: '/enable_ngrok',
                        data: {
                            'auth_token': auth_token
                        }
                    }).always(function (response) {
                        if (response === 'already running') {
                            alert('Remote debugging already activated.');
                        } else {
                            $('#auth_token').attr('disabled','disabled');
                            $('#enable_debug').html('Enabled remote debugging');
                            $('#enable_debug').removeAttr('href', '')
                            $('#enable_debug').off('click');
                        }
                    });
                }
            });
        });
        $(function() {
            $('.view-password').click(function() {
                $.ajax({
                    url:'/hw_posbox_homepage/password',
                    type: 'post',
                    contentType: 'application/json',
                    data: JSON.stringify({}),
                }).done(function(password) {
                    $('.password').val(password['result']);
                });
            });
        });
    </script>
{% endblock %}
{% block content %}
    <h2 class="text-center">Remote Debugging</h2>
    <p class='text-red'>
        This allows someone who give a ngrok authtoken to gain remote access to your IoT Box, and
        thus your entire local network. Only enable this for someone
        you trust.
    </p>
    <div class='text-center'>
        <input class="password" type="text" placeholder="pi user password" disabled>
        <a class="btn view-password" style="margin: 18px auto;"href="#">Generate password</a><br/>
        <input type="text" id="auth_token" size="42" placeholder="Authentication Token"/> <br/>
        <a class="btn" style="margin: 18px auto;" id="enable_debug" href="#">Enable Remote Debugging</a>
    </div>
{% endblock %}

```

## File: views\server_config.html

```html
{% extends "layout.html" %}
{% from "loading.html" import loading_block_ui %}
{% block head %}
    <script>
        $(document).ready(function () {
            $('#server-config').submit(function(e){
                e.preventDefault();
                $('.loading-block').removeClass('o_hide');
                $.ajax({
                    url: '/server_connect',
                    type: 'post',
                    data: $('#server-config').serialize(),
                }).fail(function () {
                    $('.message-status').html('Configure Domain Server <br> Redirect to IoT Box');
                    if ($('#iotname')[0].defaultValue == $('#iotname')[0].value){
                        var rebootTime = 30000;
                    }else{
                        var rebootTime = 100000;
                    }
                    setTimeout(function () {
                            location.reload(true);
                    }, rebootTime);
                });
            });
        });
    </script>
{% endblock %}
{% block content %}
    <h2 class="text-center">Configure Odoo Server</h2>
    <p>
        Paste the token from the Connect wizard in your Odoo instance in the Server Token field.  If you change the IoT Box Name,
        your IoT Box will need a reboot.
    </p>
    <form id="server-config" action='/server_connect' method='POST'>
        <table align="center">
            <tr>
                <td>IoT Box Name</td>
                <td><input type="text" id="iotname" name="iotname" value="{{ hostname }}"></td>
            </tr>
            <tr>
                <td>Server Token</td>
                <td><input type="text" name="token"></td>
            </tr>
            <tr>
                <td/>
                <td><input class="btn" type="submit" value="Connect"/></td>
            </tr>
        </table>
        <p class="text-center font-small">
            Your current server <strong>{{ server_status }}</strong>
        </p>
        {{ loading_block_ui(loading_message) }}
    </form>
    <div class="text-center font-small" style="margin: 10px auto;">
        You can clear the server configuration
        <form style="display: inline-block;margin-left: 4px;" action='/server_clear'>
            <input class="btn btn-sm" type="submit" value="Clear"/>
        </form>
    </div>
{% endblock %}

```

## File: views\six_payment_terminal.html

```html
{% extends "layout.html" %}
{% from "loading.html" import loading_block_ui %}
{% block head %}
    <script>
        var _onQueryDone = function () {
            $('.message-status').html('Updated configuration <br> Refreshing page');
            setTimeout(function () {
                location.reload();
            }, 30000);
        }
        $(document).ready(function () {
            $('#terminal-id').submit(function(e){
                e.preventDefault();
                $('.loading-block').removeClass('o_hide');
                $.ajax({
                    url: '/six_payment_terminal_add',
                    type: 'post',
                    data: $('#terminal-id').serialize(),
                }).always(_onQueryDone);
            });
            $('#terminal-clear').submit(function(e){
                e.preventDefault();
                $('.loading-block').removeClass('o_hide');
                $.ajax({
                    url: '/six_payment_terminal_clear',
                    type: 'get',
                }).always(_onQueryDone);
            });
        });
    </script>
{% endblock %}
{% block content %}
    <h2 class="text-center">Six Payment Terminal</h2>
    <p>
        Set the Terminal ID (TID) of the terminal you want to use.
    </p>
    <form id="terminal-id" action='/six_payment_terminal_add' method='POST'>
        <table align="center">
            <tr>
                <td>Terminal ID (digits only)</td>
                <td><input type="text" name="terminal_id"></td>
            </tr>
            <tr>
                <td/>
                <td><input class="btn" type="submit" value="Connect"/></td>
            </tr>
        </table>
        {{ loading_block_ui(loading_message) }}
    </form>
    {% if terminalId %}
        <p class="text-center font-small">
            Current Terminal Id: <strong>{{ terminalId }}</strong>
        </p>
        <div class="text-center font-small" style="margin: 10px auto;">
            You can clear the terminal configuration
            <form id="terminal-clear" style="display: inline-block;margin-left: 4px;">
                <input class="btn btn-sm" type="submit" value="Clear"/>
            </form>
        </div>
    {% endif %}
{% endblock %}

```

## File: views\upgrade_page.html

```html
{% extends "layout.html" %}
{% from "loading.html" import loading_block_ui %}
{% block head %}
    <script type="text/javascript" src="/web/static/lib/jquery/jquery.js"></script>
    <script>
        $(function() {
            var updating = false;
            $('#upgrade').click(function() {
                if (!updating) {
                    updating = true;
                    $('.loading-block').removeClass('o_hide');
                    $.ajax({
                        url:'/hw_proxy/perform_upgrade/'
                    }).done(function() {
                        $('.message-title').text('Upgrade successful');
                        var cpt = 25;
                        setInterval(function() {
                            --cpt;
                            if (cpt === 0) {location.reload();}
                            $('.message-status').text('Restarting the IoTBox.  Available in ' + cpt);
                        } , 1000);
                    }).fail(function() {
                        $('.error-message').text('Upgrade Failed');
                    });
                }
            });
            $('#flash').click(async function() {
                if (confirm('Are you sure you want to flash your IoT Box?\nThe box will be unavailable for ~30 min\nDo not turn off the box or close this page during the flash.\nThis page will reaload when your box is ready.')) {
                    $('.loading-block').removeClass('o_hide');
                    $('.message-title').text('IoTBox perform a self flashing it take a lot of time (~30min).');
                    $('.message-status').text('Prepare space for IoTBox.');
                    try {
                        await $.ajax({url: '/hw_proxy/perform_flashing_create_partition/'}).promise();
                        $('.message-status').text('Prepare new boot partition.');
                        await $.ajax({url: '/hw_proxy/perform_flashing_download_raspios/'}).promise();
                        $('.message-status').text('Download file for new boot partition.');
                        await $.ajax({url: '/hw_proxy/perform_flashing_copy_raspios/'}).promise();
                        $('.message-status').text('Prepare to restart and installation of the new version of the IoT Box.');
                        setTimeout(function() {
                        $('.message-status').text('The auto flash is almost finished - the page will be automatically reloaded');
                        setInterval(function() {
                            $.ajax({
                                url: '/hw_proxy/get_version',
                                timeout: 4000,
                            }).done(function(version) {
                                if (version == {{ flashToVersion }}) {
                                    window.location = '/';
                                }
                            });
                        } , 2000);
                    }, 240000);
                    }
                    catch(error) {
                        $('.loading-block').addClass('o_hide');
                        $('.error-message').text(error.responseText);
                    }
                }
            });
        });
    </script>
    <style>
        .commit-details {
            background: #f1f1f1;
            padding: 10px 10px 0 10px;
            border-radius: 5px;
        }
    </style>
{% endblock %}
{% block content %}
    <h2 class="text-center">IoT Box Software Upgrade</h2>
    <p>
        This tool will help you perform an upgrade of the IoTBox's software over the internet.
        However the preferred method to upgrade the IoTBox is to flash the sd-card with
        the <a href='https://nightly.odoo.com/master/iotbox/iotbox-latest.zip'>latest image</a>. The upgrade
        procedure is explained into to the
        <a href='https://www.odoo.com/documentation/16.0/applications/productivity/iot.html'>IoTBox manual</a>
    </p>
    <p>
        To upgrade the IoTBox, click on the upgrade button. The upgrade will take a few minutes. <b>Do not reboot</b> the IoTBox during the upgrade.
    </p>
    <div class="commit-details">
        <div style="padding-bottom: 5px; font-weight: bold;">
            Latest patch:
        </div>
        <pre style="margin: 0;padding: 15px 0; overflow: auto;">{{ commit|safe }}</pre>
    </div>
    <div class="text-center" style="margin: 15px auto;">
        {% if flashToVersion %}
            <a class="btn" href='#' id='flash'>Upgrade to {{ flashToVersion }}</a>
        {% else %}
            <a class="btn" href='#' id='upgrade'>Upgrade</a>
        {% endif %}
    </div>
    {{ loading_block_ui(loading_message) }}
{% endblock %}

```

## File: views\wifi_config.html

```html
{% extends "layout.html" %}
{% from "loading.html" import loading_block_ui %}
{% block head %}
    <script>
        $(document).ready(function () {
            $('#wifi-config').submit(function(e){
                e.preventDefault();
                $('.loading-block').removeClass('o_hide');
                $.ajax({
                    url: '/wifi_connect',
                    type: 'post',
                    data: $('#wifi-config').serialize(),
                }).done(function (message) {
                    var data = JSON.parse(message);
                    var message = data.message;
                    if (data.server) {
                        message += '<br>'+ data.server.message;
                        setTimeout(function () {
                            window.location = data.server.url;
                        }, 30000);
                    }
                    $('.message-status').html(message);
                }).fail(function () {
                    $('.error-message').text('Error in connecting to wifi');
                    $('.loading-block').addClass('o_hide');
                });
            });
        });
    </script>
{% endblock %}
{% block content %}
    <h2 class="text-center">Configure Wifi</h2>
    <p>
        Here you can configure how the iotbox should connect to wireless networks.
        Currently only Open and WPA networks are supported. When enabling the persistent checkbox,
        the chosen network will be saved and the iotbox will attempt to connect to it every time it boots.
    </p>
    <form id="wifi-config" action='/wifi_connect' method='POST'>
        <table align="center">
            <tr>
                <td>ESSID</td>
                <td>
                    <select name="essid">
                        {% for id in ssid -%}
                            <option value="{{ id }}">{{ id }}</option>
                        {%- endfor %}
                    </select>
                </td>
            </tr>
            <tr>
                <td>Password</td>
                <td><input type="password" name="password" placeholder="optional"/></td>
            </tr>
            <tr>
                <td>Persistent</td>
                <td><input type="checkbox" name="persistent"/></td>
            </tr>
            <tr>
                <td/>
                <td><input class="btn" type="submit" value="Connect"/></td>
            </tr>
        </table>
    </form>
    <div class="text-center font-small" style="margin: 10px auto;">
        You can clear the persistent configuration
        <form style="display: inline-block;margin-left: 4px;" action='/wifi_clear'>
            <input class="btn btn-sm" type="submit" value="Clear"/>
        </form>
    </div>
    {{ loading_block_ui(loading_message) }}
{% endblock %}

```

