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
    'assets': {
        'web.assets_backend': [
            'hw_posbox_homepage/static/*/**',
        ],
    },
    'installable': False,
    'license': 'LGPL-3',
}

```

## File: controllers\homepage.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json
import subprocess
import threading
import logging
import platform
import jinja2
import os
import sys

from pathlib import Path
from odoo import http, tools
from odoo.addons.hw_drivers.tools import helpers
from odoo.addons.hw_drivers.main import iot_devices
from odoo.addons.web.controllers.home import Home
from odoo.addons.hw_drivers.connection_manager import connection_manager
from odoo.tools.misc import file_path
from odoo.addons.hw_drivers.server_logger import (
    check_and_update_odoo_config_log_to_server_option,
    get_odoo_config_log_to_server_option,
    close_server_log_sender_handler,
)

_logger = logging.getLogger(__name__)

IOT_LOGGING_PREFIX = 'iot-logging-'
INTERFACE_PREFIX = 'interface-'
DRIVER_PREFIX = 'driver-'
AVAILABLE_LOG_LEVELS = ('debug', 'info', 'warning', 'error')
AVAILABLE_LOG_LEVELS_WITH_PARENT = AVAILABLE_LOG_LEVELS + ('parent',)

if hasattr(sys, 'frozen'):
    # When running on compiled windows binary, we don't have access to package loader.
    path = os.path.realpath(os.path.join(os.path.dirname(__file__), '..', 'views'))
    loader = jinja2.FileSystemLoader(path)
else:
    loader = jinja2.PackageLoader('odoo.addons.hw_posbox_homepage', "views")

jinja_env = jinja2.Environment(loader=loader, autoescape=True)
jinja_env.filters["json"] = json.dumps

index_template = jinja_env.get_template('index.html')
logs_template = jinja_env.get_template('logs.html')


class IotBoxOwlHomePage(Home):
    def __init__(self):
        super().__init__()
        self.updating = threading.Lock()

    @http.route()
    def index(self):
        return index_template.render()

    @http.route("/logs")
    def logs_page(self):
        return logs_template.render()

    # ---------------------------------------------------------- #
    # GET methods                                                #
    # -> Always use json.dumps() to return a JSON response       #
    # ---------------------------------------------------------- #
    @http.route('/hw_posbox_homepage/restart_odoo_service', auth='none', type='http', cors='*')
    def odoo_service_restart(self):
        helpers.odoo_restart(0)
        return json.dumps({
            'status': 'success',
            'message': 'Odoo service restarted',
        })

    @http.route('/hw_posbox_homepage/iot_logs', auth='none', type='http', cors='*')
    def get_iot_logs(self):
        with open("/var/log/odoo/odoo-server.log", encoding="utf-8") as file:
            return json.dumps({
                'status': 'success',
                'logs': file.read(),
            })

    @http.route('/hw_posbox_homepage/six_payment_terminal_clear', auth='none', type='http', cors='*')
    def clear_six_terminal(self):
        helpers.update_conf({'six_payment_terminal': ''})
        return json.dumps({
            'status': 'success',
            'message': 'Successfully cleared Six Payment Terminal',
        })

    @http.route('/hw_posbox_homepage/clear_credential', auth='none', type='http', cors='*')
    def clear_credential(self):
        helpers.update_conf({
            'db_uuid': '',
            'enterprise_code': '',
        })
        helpers.odoo_restart(0)
        return json.dumps({
            'status': 'success',
            'message': 'Successfully cleared credentials',
        })

    @http.route('/hw_posbox_homepage/wifi_clear', auth='none', type='http', cors='*')
    def clear_wifi_configuration(self):
        helpers.update_conf({'wifi_ssid': '', 'wifi_password': ''})
        return json.dumps({
            'status': 'success',
            'message': 'Successfully disconnected from wifi',
        })

    @http.route('/hw_posbox_homepage/server_clear', auth='none', type='http', cors='*')
    def clear_server_configuration(self):
        helpers.disconnect_from_server()
        close_server_log_sender_handler()
        return json.dumps({
            'status': 'success',
            'message': 'Successfully disconnected from server',
        })

    @http.route('/hw_posbox_homepage/ping', auth='none', type='http', cors='*')
    def ping(self):
        return json.dumps({
            'status': 'success',
            'message': 'pong',
        })

    @http.route('/hw_posbox_homepage/data', auth="none", type="http", cors='*')
    def get_homepage_data(self):
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

        terminal_id = helpers.get_conf('six_payment_terminal')
        six_terminal = terminal_id or 'Not Configured'

        return json.dumps({
            'db_uuid': helpers.get_conf('db_uuid'),
            'enterprise_code': helpers.get_conf('enterprise_code'),
            'hostname': helpers.get_hostname(),
            'ip': helpers.get_ip(),
            'mac': helpers.get_mac_address(),
            'iot_device_status': iot_device,
            'server_status': helpers.get_odoo_server_url() or 'Not Configured',
            'pairing_code': connection_manager.pairing_code,
            'six_terminal': six_terminal,
            'network_status': network,
            'version': helpers.get_version(),
            'system': platform.system(),
            'is_certificate_ok': is_certificate_ok,
            'certificate_details': certificate_details,
        })

    @http.route('/hw_posbox_homepage/wifi', auth="none", type="http", cors='*')
    def get_available_wifi(self):
        return json.dumps(helpers.get_wifi_essid())

    @http.route('/hw_posbox_homepage/generate_password', auth="none", type="http", cors='*')
    def generate_password(self):
        return json.dumps({
            'password': helpers.generate_password(),
        })

    @http.route('/hw_posbox_homepage/version_info', auth="none", type="http", cors='*')
    def get_version_info(self):
        git = ["git", "--work-tree=/home/pi/odoo/", "--git-dir=/home/pi/odoo/.git"]
        # Check branch name and last commit hash on IoT Box
        current_commit = subprocess.run([*git, "rev-parse", "HEAD"], capture_output=True, check=False, text=True)
        current_branch = subprocess.run(
            [*git, "rev-parse", "--abbrev-ref", "HEAD"], capture_output=True, check=False, text=True
        )
        if current_commit.returncode != 0 or current_branch.returncode != 0:
            return
        current_commit = current_commit.stdout.strip()
        current_branch = current_branch.stdout.strip()

        last_available_commit = subprocess.run(
            [*git, "ls-remote", "origin", current_branch], capture_output=True, check=False, text=True
        )
        if last_available_commit.returncode != 0:
            _logger.error("Failed to retrieve last commit available for branch origin/%s", current_branch)
            return
        last_available_commit = last_available_commit.stdout.split()[0].strip()

        return json.dumps({
            'status': 'success',
            # Checkout requires db to align with its version (=branch)
            'odooIsUpToDate': current_commit == last_available_commit or not bool(helpers.get_odoo_server_url()),
            'imageIsUpToDate': not bool(helpers.check_image()),
            'currentCommitHash': current_commit,
        })

    @http.route('/hw_posbox_homepage/log_levels', auth="none", type="http", cors='*')
    def log_levels(self):
        drivers_list = helpers.list_file_by_os(
            file_path('hw_drivers/iot_handlers/drivers'))
        interfaces_list = helpers.list_file_by_os(
            file_path('hw_drivers/iot_handlers/interfaces'))
        return json.dumps({
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

    @http.route('/hw_posbox_homepage/load_iot_handlers', auth="none", type="http", cors='*')
    def load_iot_log_level(self):
        helpers.download_iot_handlers(False)
        helpers.odoo_restart(0)
        return json.dumps({
            'status': 'success',
            'message': 'IoT Handlers loaded successfully',
        })

    @http.route('/hw_posbox_homepage/clear_iot_handlers', auth="none", type="http", cors='*')
    def clear_iot_handlers(self):
        for directory in ['drivers', 'interfaces']:
            for file in list(Path(file_path(f'hw_drivers/iot_handlers/{directory}')).glob('*')):
                if file.name != '__pycache__':
                    helpers.unlink_file(str(file.relative_to(*file.parts[:3])))

        return json.dumps({
            'status': 'success',
            'message': 'IoT Handlers cleared successfully',
        })

    # ---------------------------------------------------------- #
    # POST methods                                               #
    # -> Never use json.dumps() it will be done automatically    #
    # ---------------------------------------------------------- #
    @http.route('/hw_posbox_homepage/six_payment_terminal_add', auth="none", type="json", methods=['POST'], cors='*')
    def add_six_terminal(self, terminal_id):
        if terminal_id.isdigit():
            helpers.update_conf({'six_payment_terminal': terminal_id})
        else:
            _logger.warning('Ignoring invalid Six TID: "%s". Only digits are allowed', terminal_id)
            return self.clear_six_terminal()
        return {
            'status': 'success',
            'message': 'Successfully saved Six Payment Terminal',
        }

    @http.route('/hw_posbox_homepage/save_credential', auth="none", type="json", methods=['POST'], cors='*')
    def save_credential(self, db_uuid, enterprise_code):
        helpers.update_conf({
            'db_uuid': db_uuid,
            'enterprise_code': enterprise_code,
        })
        helpers.odoo_restart(0)
        return {
            'status': 'success',
            'message': 'Successfully saved credentials',
        }

    @http.route('/hw_posbox_homepage/update_wifi', auth="none", type="json", methods=['POST'], cors='*')
    def update_wifi(self, essid, password, persistent=False):
        persistent = "1" if persistent else ""
        subprocess.check_call([file_path(
            'point_of_sale/tools/posbox/configuration/connect_to_wifi.sh'), essid, password, persistent])
        server = helpers.get_odoo_server_url()

        res_payload = {
            'status': 'success',
            'message': 'Connecting to ' + essid,
            'server': {
                'url': server or 'http://' + helpers.get_ip() + ':8069',
                'message': 'Redirect to Odoo Server' if server else 'Redirect to IoT Box'
            }
        }

        return res_payload

    @http.route('/hw_posbox_homepage/enable_ngrok', auth="none", type="json", methods=['POST'], cors='*')
    def enable_remote_connection(self, auth_token):
        if subprocess.call(['pgrep', 'ngrok']) == 1:
            subprocess.Popen(['ngrok', 'tcp', '--authtoken', auth_token, '--log', '/tmp/ngrok.log', '22'])

        return {
            'status': 'success',
            'auth_token': auth_token,
            'message': 'Ngrok tunnel is now enabled',
        }

    @http.route('/hw_posbox_homepage/connect_to_server', auth="none", type="json", methods=['POST'], cors='*')
    def connect_to_odoo_server(self, token=False, iotname=False):
        if token:
            try:
                if len(token.split('|')) == 4:
                    # Old style token with pipe separators (pre v18 DB)
                    url, token, db_uuid, enterprise_code = token.split('|')
                    configuration = helpers.parse_url(url)
                    helpers.save_conf_server(configuration["url"], token, db_uuid, enterprise_code)
                else:
                    # New token using query params (v18+ DB)
                    configuration = helpers.parse_url(token)
                    helpers.save_conf_server(**configuration)
            except ValueError:
                _logger.warning("Wrong server token: %s", token)
                return {
                    'status': 'failure',
                    'message': 'Invalid URL provided.',
                }
            except (subprocess.CalledProcessError, OSError, Exception):
                return {
                    'status': 'failure',
                    'message': 'Failed to write server configuration files on IoT. Please try again.',
                }

        if iotname and platform.system() == 'Linux' and iotname != helpers.get_hostname():
            subprocess.run([file_path(
                'point_of_sale/tools/posbox/configuration/rename_iot.sh'), iotname], check=False)

        # 1 sec delay for IO operations (save_conf_server)
        helpers.odoo_restart(1)
        return {
            'status': 'success',
            'message': 'Successfully connected to db, IoT will restart to update the configuration.',
        }

    @http.route('/hw_posbox_homepage/log_levels_update', auth="none", type="json", methods=['POST'], cors='*')
    def update_log_level(self, name, value):
        if not name.startswith(IOT_LOGGING_PREFIX) and name != 'log-to-server':
            return {
                'status': 'error',
                'message': 'Invalid logger name',
            }

        need_config_save = False
        if name == 'log-to-server':
            need_config_save |= check_and_update_odoo_config_log_to_server_option(
                value
            )

        name = name[len(IOT_LOGGING_PREFIX):]
        if name == 'root':
            need_config_save |= self._update_logger_level(
                '', value, AVAILABLE_LOG_LEVELS)
        elif name == 'odoo':
            need_config_save |= self._update_logger_level(
                'odoo', value, AVAILABLE_LOG_LEVELS)
            need_config_save |= self._update_logger_level(
                'werkzeug', value if value != 'debug' else 'info', AVAILABLE_LOG_LEVELS)
        elif name.startswith(INTERFACE_PREFIX):
            logger_name = name[len(INTERFACE_PREFIX):]
            need_config_save |= self._update_logger_level(
                logger_name, value, AVAILABLE_LOG_LEVELS_WITH_PARENT, 'interfaces')
        elif name.startswith(DRIVER_PREFIX):
            logger_name = name[len(DRIVER_PREFIX):]
            need_config_save |= self._update_logger_level(
                logger_name, value, AVAILABLE_LOG_LEVELS_WITH_PARENT, 'drivers')
        else:
            _logger.warning('Unhandled iot logger: %s', name)

        if need_config_save:
            with helpers.writable():
                tools.config.save()

        return {
            'status': 'success',
            'message': 'Logger level updated',
        }

    @http.route('/hw_posbox_homepage/update_git_tree', auth="none", type="json", methods=['POST'], cors='*')
    def update_git_tree(self):
        helpers.check_git_branch()
        return {
            'status': 'success',
            'message': 'Successfully updated the IoT Box',
        }

    # ---------------------------------------------------------- #
    # Utils                                                      #
    # ---------------------------------------------------------- #
    def _get_iot_handlers_logger(self, handlers_name, iot_handler_folder_name):
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
        return odoo_addon_handler_path in logging.Logger.manager.loggerDict and \
               logging.getLogger(odoo_addon_handler_path)

```

## File: controllers\main.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import os
import subprocess
import threading

from odoo import http
from odoo.http import Response
from odoo.addons.hw_drivers.tools import helpers
from odoo.addons.web.controllers.home import Home

_logger = logging.getLogger(__name__)


class IoTboxHomepage(Home):
    def __init__(self):
        super(IoTboxHomepage,self).__init__()
        self.updating = threading.Lock()

    def clean_partition(self):
        subprocess.check_call(['sudo', 'bash', '-c', '. /home/pi/odoo/addons/point_of_sale/tools/posbox/configuration/upgrade.sh; cleanup'])

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

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main
from . import homepage

```

## File: static\img\background-light.svg

```svg
<svg width="1920" height="1080" viewBox="0 0 1920 1080" xmlns="http://www.w3.org/2000/svg">
<path d="M3.51001 1080H76.35L1153.55 0H3.51001V1080Z" fill="url(#o_app_switcher_gradient_01)"/>
<path d="M76.35 1080H842.98L1920 0.18V0H1153.55L76.35 1080Z" fill="url(#o_app_switcher_gradient_02)"/>
<path d="M1920 0.180176L842.98 1080H1063.11L1920 220.88V0.180176Z" fill="url(#o_app_switcher_gradient_03)"/>
<path d="M1920 1080V220.88L1063.11 1080H1920Z" fill="url(#o_app_switcher_gradient_04)"/>
<rect width="1920" height="1080" fill="url(#o_app_switcher_gradient_05)" fill-opacity="0.25"/>
<rect width="1920" height="1080" fill="#E9E6F9" fill-opacity="0.25"/>
<defs>
<linearGradient id="o_app_switcher_gradient_01" x1="-222.43" y1="727.19" x2="904.26" y2="-76.67" gradientUnits="userSpaceOnUse">
<stop offset="0.1" stop-color="white"/>
<stop offset="0.36" stop-color="#FEFEFE"/>
<stop offset="0.68" stop-color="#EAE7F9"/>
<stop offset="1" stop-color="#E4E9F7"/>
</linearGradient>
<linearGradient id="o_app_switcher_gradient_02" x1="407.23" y1="1021.82" x2="1848.47" y2="-153.08" gradientUnits="userSpaceOnUse">
<stop offset="0.32" stop-color="#FEFEFE"/>
<stop offset="0.66" stop-color="#EAE7F9"/>
<stop offset="1" stop-color="#E5E2F6"/>
</linearGradient>
<linearGradient id="o_app_switcher_gradient_03" x1="1142.33" y1="846.57" x2="1951.83" y2="136.16" gradientUnits="userSpaceOnUse">
<stop offset="0.15" stop-color="white"/>
<stop offset="0.51" stop-color="#F7F0FD"/>
<stop offset="0.85" stop-color="#F0E7F9"/>
</linearGradient>
<linearGradient id="o_app_switcher_gradient_04" x1="1409.74" y1="1071" x2="2070.98" y2="526.01" gradientUnits="userSpaceOnUse">
<stop offset="0.45" stop-color="white"/>
<stop offset="0.88" stop-color="#F7F0FD"/>
<stop offset="1" stop-color="#ECE5F8"/>
</linearGradient>
<radialGradient id="o_app_switcher_gradient_05" cx="0" cy="0" r="1" gradientUnits="userSpaceOnUse" gradientTransform="translate(960 540) rotate(90) scale(540 960)">
<stop stop-color="#9996A9" stop-opacity="0.53"/>
<stop offset="1" stop-color="#7A768F"/>
</radialGradient>
</defs>
</svg>

```

## File: static\src\app\Homepage.js

```javascript
/* global owl */

import { SingleData } from "./components/SingleData.js";
import { FooterButtons } from "./components/FooterButtons.js";
import { ServerDialog } from "./components/dialog/ServerDialog.js";
import { WifiDialog } from "./components/dialog/WifiDialog.js";
import useStore from "./hooks/useStore.js";
import { UpdateDialog } from "./components/dialog/UpdateDialog.js";
import { DeviceDialog } from "./components/dialog/DeviceDialog.js";
import { SixDialog } from "./components/dialog/SixDialog.js";
import { LoadingFullScreen } from "./components/LoadingFullScreen.js";
import { IconButton } from "./components/IconButton.js";

const { Component, xml, useState, onWillStart } = owl;

export class Homepage extends Component {
    static props = {};
    static components = {
        SingleData,
        FooterButtons,
        ServerDialog,
        WifiDialog,
        UpdateDialog,
        DeviceDialog,
        SixDialog,
        LoadingFullScreen,
        IconButton,
    };

    setup() {
        this.store = useStore();
        this.state = useState({ loading: true, waitRestart: false });
        this.data = useState({});
        this.store.advanced = localStorage.getItem("showAdvanced") === "true";
        this.store.dev = new URLSearchParams(window.location.search).has("debug");

        onWillStart(async () => {
            await this.loadInitialData();
        });

        setInterval(() => {
            this.loadInitialData();
        }, 10000);
    }

    async loadInitialData() {
        try {
            const data = await this.store.rpc({
                url: "/hw_posbox_homepage/data",
            });

            if (data.system === "Linux") {
                this.store.isLinux = true;
            }

            this.data = data;
            this.store.base = data;
            this.state.loading = false;
            this.store.update = new Date().getTime();
        } catch {
            console.warn("Error while fetching data");
        }
    }

    async restartOdooService() {
        try {
            await this.store.rpc({
                url: "/hw_posbox_homepage/restart_odoo_service",
            });

            this.state.waitRestart = true;
        } catch {
            console.warn("Error while restarting Odoo Service");
        }
    }

    toggleAdvanced() {
        this.store.advanced = !this.store.advanced;
        localStorage.setItem("showAdvanced", this.store.advanced);
    }

    static template = xml`
    <LoadingFullScreen t-if="this.state.waitRestart">
        <t t-set-slot="body">
           Restarting IoT Box, please wait...
        </t>
    </LoadingFullScreen>

    <div t-if="!this.state.loading" class="w-100 d-flex flex-column align-items-center justify-content-center" style="background-color: #F1F1F1; height: 100vh">
        <div class="bg-white p-4 rounded overflow-auto position-relative" style="width: 100%; max-width: 600px;">
            <div class="position-absolute end-0 top-0 mt-3 me-4 d-flex gap-1">
                <IconButton onClick.bind="toggleAdvanced" icon="this.store.advanced ? 'fa-cog' : 'fa-cogs'" />
                <IconButton onClick.bind="restartOdooService" icon="'fa-power-off'" />
            </div>
            <div class="d-flex mb-4 flex-column align-items-center justify-content-center">
                <h4 class="text-center m-0">IoT Box - <t t-esc="this.data.hostname" /></h4>
            </div>
            <div t-if="this.store.advanced" t-att-class="'alert ' + (this.data.is_certificate_ok === true ? 'alert-info' : 'alert-warning')" role="alert">
                <p class="m-0 fw-bold">HTTPS Certificate</p>
                <small>
                    <t t-if="this.data.is_certificate_ok === true">Status: </t>
                    <t t-else="">Error Code: </t>
                    <t t-esc="this.data.certificate_details" />
                </small>
            </div>
            <SingleData name="'Name'" value="this.data.hostname" icon="'fa-id-card'">
				<t t-set-slot="button">
					<ServerDialog t-if="this.store.isLinux" />
				</t>
			</SingleData>
            <SingleData t-if="this.store.advanced" name="'Version'" value="this.data.version" icon="'fa-microchip'">
                <t t-set-slot="button">
                    <UpdateDialog />
                </t>
            </SingleData>
            <SingleData t-if="this.store.advanced" name="'IP address'" value="this.data.ip" icon="'fa-globe'" />
            <SingleData t-if="this.store.advanced" name="'MAC address'" value="this.data.mac.toUpperCase()" icon="'fa-address-card'" />
            <SingleData t-if="this.store.isLinux" name="'Internet Status'" value="this.data.network_status"  icon="'fa-wifi'">
                <t t-set-slot="button">
                    <WifiDialog />
                </t>
            </SingleData>
            <SingleData name="'Odoo database connected'" value="this.data.server_status" icon="'fa-link'">
				<t t-set-slot="button">
					<ServerDialog />
				</t>
			</SingleData>
            <SingleData t-if="this.data.pairing_code" name="'Pairing Code'" value="this.data.pairing_code" icon="'fa-code'"/>
            <SingleData  t-if="this.store.advanced" name="'Six terminal'" value="this.data.six_terminal" icon="'fa-money'">
                <t t-set-slot="button">
                    <SixDialog />
                </t>
            </SingleData>
            <SingleData name="'Devices'" value="this.data.iot_device_status.length + ' devices'" icon="'fa-plug'">
                <t t-set-slot="button">
                    <DeviceDialog />
                </t>
            </SingleData>

            <hr class="mt-5" />
            <FooterButtons />
            <div class="d-flex justify-content-center gap-2 mt-2">
                <a href="https://www.odoo.com/fr_FR/help" target="_blank" class="link-primary">Help</a>
                <a href="https://www.odoo.com/documentation/master/applications/general/iot.html" target="_blank" class="link-primary">Documentation</a>
            </div>
        </div>
    </div>
    <div t-else="" class="w-100 d-flex align-items-center justify-content-center" style="background-color: #F1F1F1; height: 100vh">
        <div class="spinner-border" role="status">
            <span class="visually-hidden">Loading...</span>
        </div>
    </div>
  `;
}

```

## File: static\src\app\main.js

```javascript
/* global owl */

import { Homepage } from "./Homepage.js";
import Store from "./store.js";

const { mount, reactive } = owl;

function createStore() {
    return reactive(new Store());
}

mount(Homepage, document.body, {
    env: {
        store: createStore(),
    },
});

```

## File: static\src\app\store.js

```javascript
export default class Store {
    constructor() {
        this.setup();
    }
    setup() {
        this.url = "";
        this.base = {};
        this.update = 0;
        this.isLinux = false;
        this.advanced = false;
    }

    async rpc({ url, method = "GET", params = {} }) {
        if (method === "POST") {
            const response = await fetch(url, {
                method,
                headers: {
                    "Content-Type": "application/json; charset=utf-8",
                },
                body: JSON.stringify({
                    params,
                }),
            });

            const data = await response.json();
            return data.result;
        } else if (method === "GET") {
            const response = await fetch(url);
            return await response.json();
        }

        return false;
    }
}

```

## File: static\src\app\components\FooterButtons.js

```javascript
/* global owl */

import useStore from "../hooks/useStore.js";
import { CredentialDialog } from "./dialog/CredentialDialog.js";
import { HandlerDialog } from "./dialog/HandlerDialog.js";
import { RemoteDebugDialog } from "./dialog/RemoteDebugDialog.js";

const { Component, xml } = owl;

export class FooterButtons extends Component {
    static props = {};
    static components = {
        RemoteDebugDialog,
        HandlerDialog,
        CredentialDialog,
    };

    setup() {
        this.store = useStore();
    }

    static template = xml`
    <div class="w-100 d-flex align-items-cente gap-2 justify-content-center">
        <a t-if="this.store.isLinux" class="btn btn-primary btn-sm" t-att-href="'http://' + this.store.base.ip + '/point_of_sale/display'" target="_blank">PoS Display</a>
        <a t-if="this.store.isLinux" class="btn btn-primary btn-sm" t-att-href="'http://' + this.store.base.ip + ':631'" target="_blank">Printer Server</a>
        <RemoteDebugDialog t-if="this.store.advanced and this.store.isLinux" />
        <CredentialDialog t-if="this.store.advanced" />
        <HandlerDialog t-if="this.store.advanced" />
    </div>
  `;
}

```

## File: static\src\app\components\IconButton.js

```javascript
/* global owl */

import useStore from "../hooks/useStore.js";

const { Component, xml } = owl;

export class IconButton extends Component {
    static props = {
        onClick: Function,
        icon: String,
    };

    setup() {
        this.store = useStore();
    }

    static template = xml`
    <div class="cursor-pointer bg-primary rounded text-white d-flex align-items-center justify-content-center" style="width: 25px; height: 25px; font-size: 12px" t-on-click="this.props.onClick">
        <i class="fa" t-att-class="this.props.icon" aria-hidden="true"></i>
    </div>
  `;
}

```

## File: static\src\app\components\LoadingFullScreen.js

```javascript
/* global owl */

import useStore from "../hooks/useStore.js";

const { Component, xml, onMounted } = owl;

export class LoadingFullScreen extends Component {
    static props = {
        slots: Object,
    };

    setup() {
        this.store = useStore();

        // We delay the RPC verification for 10 seconds to be sure that the Odoo service
        // was already restarted
        onMounted(() => {
            setTimeout(() => {
                setInterval(async () => {
                    try {
                        await this.store.rpc({
                            url: "/hw_posbox_homepage/ping",
                        });
                        window.location.reload();
                    } catch {
                        console.warn("Odoo service is probably rebooting.");
                    }
                }, 750);
            }, 10000);
        });
    }

    static template = xml`
    <div class="position-fixed top-0 start-0 bg-white vh-100 w-100 justify-content-center align-items-center d-flex flex-column gap-3" style="z-index: 9999">
        <div class="spinner-border" role="status">
            <span class="visually-hidden">Loading...</span>
        </div>
        <t t-slot="body" />
    </div>
  `;
}

```

## File: static\src\app\components\SingleData.js

```javascript
/* global owl */

const { Component, xml } = owl;

export class SingleData extends Component {
    static props = {
        name: String,
        value: String,
        icon: { type: String, optional: true },
        style: { type: String, optional: true },
        slots: { type: Object, optional: true },
        btnName: { type: String, optional: true },
        btnAction: { type: Function, optional: true },
    };
    static defaultProps = {
        style: "primary",
    };

    get valueIsURL() {
        const expression =
            /((([A-Za-z]{3,9}:(?:\/\/)?)(?:[-;:&=+$,\w]+@)?[A-Za-z0-9.-]+|(?:www.|[-;:&=+$,\w]+@)[A-Za-z0-9.-]+)((?:\/[+~%/.\w-_]*)?\??(?:[-+=&;%@.\w_]*)#?(?:[\w]*))?)/;

        const regex = new RegExp(expression);
        if (this.props.value.match(regex)) {
            return true;
        } else {
            return false;
        }
    }

    static template = xml`
    <div class="w-100 d-flex justify-content-between align-items-center bg-light rounded ps-2 pe-3 py-1 mb-2 gap-2">
        <div t-att-class="this.props.style === 'primary' ? 'odoo-bg-primary' : 'odoo-bg-secondary'" class="rounded" style="width: 7px !important; height: 50px" />
        <div class="flex-grow-1 overflow-hidden">
            <h6 class="m-0">
                <i t-if="this.props.icon" class="me-2 fa" t-att-class="this.props.icon" aria-hidden="true"></i>
                <t t-esc="this.props.name" />
            </h6>
            <p t-if="!this.valueIsURL" class="m-0 text-secondary one-line" t-esc="this.props.value" />
            <a t-if="this.valueIsURL" t-att-href="this.props.value" target="_blank" class="m-0 text-secondary one-line" t-esc="this.props.value" />
        </div>
        <div t-if="this.props.btnName">
            <button class="btn btn-primary btn-sm" t-esc="this.props.btnName" t-on-click="() => this.props.btnAction()" />
        </div>
        <t t-if="this.props.slots and this.props.slots['button']" t-slot="button" />
    </div>
  `;
}

```

## File: static\src\app\components\dialog\BootstrapDialog.js

```javascript
/* global owl */

const { Component, xml, useEffect, useRef } = owl;

export class BootstrapDialog extends Component {
    static props = {
        identifier: String,
        slots: Object,
        btnName: { type: String, optional: true },
        onOpen: { type: Function, optional: true },
        onClose: { type: Function, optional: true },
    };

    setup() {
        this.dialog = useRef("dialog");

        useEffect(
            () => {
                if (!this.dialog || !this.dialog.el) {
                    return;
                }

                if (this.props.onOpen) {
                    this.dialog.el.addEventListener("show.bs.modal", this.props.onOpen);
                }

                if (this.props.onClose) {
                    this.dialog.el.addEventListener("hide.bs.modal", this.props.onClose);
                }

                return () => {
                    this.dialog.el.removeEventListener("show.bs.modal", this.props.onOpen);
                    this.dialog.el.removeEventListener("hide.bs.modal", this.props.onClose);
                };
            },
            () => [this.dialog]
        );
    }

    static template = xml`
        <button type="button" class="btn btn-primary btn-sm" data-bs-toggle="modal" t-att-data-bs-target="'#'+this.props.identifier" t-esc="this.props.btnName" />
        <div t-ref="dialog" t-att-id="this.props.identifier" class="modal modal-dialog-scrollable fade" tabindex="-1" aria-hidden="true">
            <div class="modal-dialog">
                <div class="modal-content">
                    <div class="modal-header">
                        <t t-slot="header" />
                    </div>
                    <div class="modal-body position-relative" style="max-height: 70vh; min-height: 40vh;">
                        <t t-slot="body" />
                    </div>
                    <div class="modal-footer">
                        <t t-slot="footer" />
                    </div>
                </div>
            </div>
        </div>
    `;
}

```

## File: static\src\app\components\dialog\CredentialDialog.js

```javascript
/* global owl */

import useStore from "../../hooks/useStore.js";
import { BootstrapDialog } from "./BootstrapDialog.js";
import { LoadingFullScreen } from "../LoadingFullScreen.js";

const { Component, xml, useState, toRaw } = owl;

export class CredentialDialog extends Component {
    static props = {};
    static components = { BootstrapDialog, LoadingFullScreen };

    setup() {
        this.store = toRaw(useStore());
        this.state = useState({ waitRestart: false });
        this.form = useState({
            db_uuid: this.store.base.db_uuid,
            enterprise_code: "",
        });
    }

    async connectToServer() {
        try {
            if (!this.form.db_uuid || !this.form.enterprise_code) {
                return;
            }

            const data = await this.store.rpc({
                url: "/hw_posbox_homepage/save_credential",
                method: "POST",
                params: this.form,
            });

            if (data.status === "success") {
                this.state.waitRestart = true;
            }
        } catch {
            console.warn("Error while fetching data");
        }
    }

    async clearConfiguration() {
        try {
            const data = await this.store.rpc({
                url: "/hw_posbox_homepage/clear_credential",
            });

            if (data.status === "success") {
                this.state.waitRestart = true;
            }
        } catch {
            console.warn("Error while clearing configuration");
        }
    }

    static template = xml`
        <LoadingFullScreen t-if="this.state.waitRestart">
            <t t-set-slot="body">
                Your IoT Box is currently processing your request. Please wait.
            </t>
        </LoadingFullScreen>

        <BootstrapDialog identifier="'credential-configuration'" btnName="'Credential'">
            <t t-set-slot="header">
                Configure credential
            </t>
            <t t-set-slot="body">
                <div class="alert alert-info fs-6" role="alert">
                    Set the DB UUID and your Contract Number you want to use.
                </div>
                <div class="mt-3">
                    <div class="input-group-sm mb-3">
                        <label for="iotname">DB uuid</label>
                        <input name="iotname" type="text" class="form-control" t-model="this.form.db_uuid" />
                        <small t-if="!this.form.db_uuid" class="text-danger">Please enter a correct db UUID</small>
                    </div>
                    <div class="input-group-sm mb-3">
                        <label for="token">Contract number</label>
                        <input name="token" type="text" class="form-control" t-model="this.form.enterprise_code" />
                        <small t-if="!this.form.enterprise_code" class="text-danger">Please enter a contract number</small>
                    </div>
                    <div class="d-flex justify-content-end gap-2">
                        <button type="submit" class="btn btn-warning btn-sm" t-on-click="connectToServer">Connect</button>
                    </div>
                </div>
            </t>
            <t t-set-slot="footer">
                <button class="btn btn-danger btn-sm" t-on-click="clearConfiguration">Clear configuration</button>
                <button type="button" class="btn btn-primary btn-sm" data-bs-dismiss="modal">Close</button>
            </t>
        </BootstrapDialog>
    `;
}

```

## File: static\src\app\components\dialog\DeviceDialog.js

```javascript
/* global owl */

import useStore from "../../hooks/useStore.js";
import { SingleData } from "../SingleData.js";
import { BootstrapDialog } from "./BootstrapDialog.js";

const { Component, xml, useState } = owl;

export class DeviceDialog extends Component {
    static props = {};
    static components = { BootstrapDialog, SingleData };

    setup() {
        this.store = useStore();
        this.state = useState({
            loading: false,
        });
    }

    onClose() {
        this.state.initialization = [];
        this.state.handlerData = {};
    }

    get devices() {
        // Put blackbox first in the list
        return this.store.base.iot_device_status.sort((a, b) =>
            a.type === "fiscal data module" ? -1 : 1
        );
    }

    static template = xml`
        <BootstrapDialog identifier="'device-list'" btnName="'Show'">
            <t t-set-slot="header">
                Devices list
            </t>
            <t t-set-slot="body">
                <div t-if="this.store.base.iot_device_status.length === 0" class="alert alert-warning fs-6" role="alert">
                    No devices found.
                </div>
                <div>
                    <t t-foreach="devices" t-as="device" t-key="device.identifier">
                        <SingleData name="'Device: ' + device.name.slice(0, 40) +  device.name.length > 40 ? '...' : ''" value="device.type + ' - ' + device.identifier.slice(0, 50) + '...'" style="'secondary'" />
                    </t>
                </div>
            </t>
            <t t-set-slot="footer">
                <button type="button" class="btn btn-primary btn-sm" data-bs-dismiss="modal">Close</button>
            </t>
        </BootstrapDialog>
    `;
}

```

## File: static\src\app\components\dialog\HandlerDialog.js

```javascript
/* global owl */

import useStore from "../../hooks/useStore.js";
import { BootstrapDialog } from "./BootstrapDialog.js";
import { LoadingFullScreen } from "../LoadingFullScreen.js";

const { Component, xml, useState } = owl;

export class HandlerDialog extends Component {
    static props = {};
    static components = { BootstrapDialog, LoadingFullScreen };

    setup() {
        this.store = useStore();
        this.state = useState({
            initialization: true,
            waitRestart: false,
            loading: false,
            handlerData: {},
            globalLogger: {},
        });
    }

    onClose() {
        this.state.initialization = [];
        this.state.handlerData = {};
    }

    async getHandlerData() {
        try {
            const data = await this.store.rpc({
                url: "/hw_posbox_homepage/log_levels",
            });
            this.state.handlerData = data;
            this.state.globalLogger = {
                "iot-logging-root": data.root_logger_log_level,
                "iot-logging-odoo": data.odoo_current_log_level,
            };
            this.state.initialization = false;
        } catch {
            console.warn("Error while fetching data");
        }
    }

    async onChange(name, value) {
        try {
            await this.store.rpc({
                url: "/hw_posbox_homepage/log_levels_update",
                method: "POST",
                params: {
                    name: name,
                    value: value,
                },
            });
        } catch {
            console.warn("Error while saving data");
        }
    }

    static template = xml`
        <LoadingFullScreen t-if="this.state.waitRestart">
            <t t-set-slot="body">
                Processing your request, please wait...
            </t>
        </LoadingFullScreen>

        <BootstrapDialog identifier="'handler-configuration'" btnName="'Log level'" onOpen.bind="getHandlerData" onClose.bind="onClose">
            <t t-set-slot="header">
                Handler logging
            </t>
            <t t-set-slot="body">
                <div t-if="this.state.initialization" class="position-absolute top-0 start-0 bg-white h-100 w-100 justify-content-center align-items-center d-flex flex-column gap-3" style="z-index: 9999; min-height: 300px">
                    <div class="spinner-border" role="status">
                        <span class="visually-hidden">Loading...</span>
                    </div>
                    <p>Currently scanning for initialized drivers and interfaces...</p>
                </div>
                <t t-else="">
                    <div class="mb-3">
                        <h5>Global logs level</h5>
                        <div class="form-check mb-3">
                            <input name="log-to-server"
                                id="log-to-server"
                                class="form-check-input cursor-pointer"
                                type="checkbox"
                                t-att-checked="this.state.handlerData.is_log_to_server_activated"
                                t-on-change="(ev) => this.onChange(ev.target.name, ev.target.checked)" />
                            <label class="form-check-label cursor-pointer" for="log-to-server">IoT logs automatically send to server logs</label>
                        </div>
                        <div t-foreach="Object.entries(this.state.globalLogger)" t-as="global" t-key="global[0]" class="input-group input-group-sm mb-3">
                            <label class="input-group-text w-50" t-att-for="global[0]" t-esc="global[0]" />
                            <select t-att-name="global[0]"
                                t-if="global[1]"
                                class="form-select"
                                t-on-change="(ev) => this.onChange(ev.target.name, ev.target.value)"
                                t-att-id="global[0]"
                                t-att-value="global[1]">
                                <option value="parent">Same as Odoo</option>
                                <option value="info">Info</option>
                                <option value="debug">Debug</option>
                                <option value="warning">Warning</option>
                                <option value="error">Error</option>
                            </select>
                            <input t-else="" type="text" class="form-control" aria-label="Text input with dropdown button" disabled="true" placeholder="Logger uninitialised" />
                        </div>
                    </div>
                    <div class="mb-3">
                        <h5>Interfaces logs level</h5>
                        <div t-foreach="Object.entries(this.state.handlerData.interfaces_logger_info)" t-as="interface" t-key="interface[0]" class="input-group input-group-sm mb-3">
                            <label class="input-group-text w-50" t-att-for="interface[0]" t-esc="interface[0]" />
                            <select t-att-name="'iot-logging-interface-'+interface[0]"
                                t-if="interface[1]"
                                class="form-select"
                                t-on-change="(ev) => this.onChange(ev.target.name, ev.target.value)"
                                t-att-id="interface[0]"
                                t-att-value="interface[1].is_using_parent_level ? 'parent' : interface[1].level">
                                <option value="parent">Same as Odoo</option>
                                <option value="info">Info</option>
                                <option value="debug">Debug</option>
                                <option value="warning">Warning</option>
                                <option value="error">Error</option>
                            </select>
                            <input t-else="" type="text" class="form-control" aria-label="Text input with dropdown button" disabled="true" placeholder="Logger uninitialised" />
                        </div>
                    </div>
                    <div class="mb-3">
                        <h5>Drivers logs level</h5>
                        <div t-foreach="Object.entries(this.state.handlerData.drivers_logger_info)" t-as="drivers" t-key="drivers[0]" class="input-group input-group-sm mb-3">
                            <label class="input-group-text w-50" t-att-for="drivers[0]" t-esc="drivers[0]" />
                            <select t-att-name="'iot-logging-driver-'+drivers[0]"
                                t-if="drivers[1]"
                                class="form-select"
                                t-on-change="(ev) => this.onChange(ev.target.name, ev.target.value)"
                                t-att-id="drivers[0]"
                                t-att-value="drivers[1].is_using_parent_level ? 'parent' : drivers[1].level">
                                <option value="parent">Same as Odoo</option>
                                <option value="info">Info</option>
                                <option value="debug">Debug</option>
                                <option value="warning">Warning</option>
                                <option value="error">Error</option>
                            </select>
                            <input t-else="" type="text" class="form-control" aria-label="Text input with dropdown button" disabled="true" placeholder="Logger uninitialised" />
                        </div>
                    </div>
                </t>
            </t>
            <t t-set-slot="footer">
                <button type="button" class="btn btn-primary btn-sm" data-bs-dismiss="modal">Close</button>
            </t>
        </BootstrapDialog>
    `;
}

```

## File: static\src\app\components\dialog\RemoteDebugDialog.js

```javascript
/* global owl */

import useStore from "../../hooks/useStore.js";
import { LoadingFullScreen } from "../LoadingFullScreen.js";
import { BootstrapDialog } from "./BootstrapDialog.js";

const { Component, xml, useState } = owl;

export class RemoteDebugDialog extends Component {
    static props = {};
    static components = { BootstrapDialog, LoadingFullScreen };

    setup() {
        this.store = useStore();
        this.state = useState({
            password: "",
            loading: false,
            ngrok: false,
            ngrokToken: "",
        });
    }

    async generatePassword() {
        try {
            this.state.loading = true;

            const data = await this.store.rpc({
                url: "/hw_posbox_homepage/generate_password",
            });

            this.state.password = data.password;
            this.state.loading = false;
        } catch {
            console.warn("Error while fetching data");
        }
    }

    async connectToRemoteDebug() {
        if (!this.state.ngrokToken) {
            return;
        }

        try {
            const data = await this.store.rpc({
                url: "/hw_posbox_homepage/enable_ngrok",
                method: "POST",
                params: {
                    auth_token: this.state.ngrokToken,
                },
            });

            if (data.status === "success") {
                this.state.ngrok = true;
            }
        } catch {
            console.warn("Error while enabling remote debugging");
        }
    }

    static template = xml`
        <LoadingFullScreen t-if="this.state.waitRestart">
            <t t-set-slot="body">
                Processing your request, please wait...
            </t>
        </LoadingFullScreen>

        <BootstrapDialog identifier="'remote-debug-configuration'" btnName="'Remote debug'">
            <t t-set-slot="header">
                Remote Debugging
            </t>
            <t t-set-slot="body">
                <div class="alert alert-warning fs-6" role="alert">
                    This allows someone who give a ngrok authtoken to gain remote access to your IoT Box,
                    and thus your entire local network. Only enable this for someone you trust.
                </div>
                <div class="d-flex flex-row gap-2 mb-4">
                    <input placeholder="Password" t-att-value="this.state.password" class="form-control" readonly="readonly" />
                    <button class="btn btn-primary btn-sm" t-on-click="generatePassword">
                        <div t-if="this.state.loading" class="spinner-border spinner-border-sm" role="status">
                            <span class="visually-hidden">Loading...</span>
                        </div>
                        <t t-else="">Generate</t>
                    </button>
                </div>
                <input t-model="this.state.ngrokToken" placeholder="Authentication token" class="form-control" />
                <div class="d-flex justify-content-end gap-2">
                    <button type="submit" class="btn btn-primary mt-2 btn-sm" t-on-click="connectToRemoteDebug">Enable remote debugging</button>
                </div>
                <div t-if="this.state.ngrok" class="alert alert-success fs-6 mt-2" role="alert">
                    Your IoT Box is now accessible from the internet.
                </div>
            </t>
            <t t-set-slot="footer">
                <button type="button" class="btn btn-primary btn-sm" data-bs-dismiss="modal">Close</button>
            </t>
        </BootstrapDialog>
    `;
}

```

## File: static\src\app\components\dialog\ServerDialog.js

```javascript
/* global owl */

import useStore from "../../hooks/useStore.js";
import { BootstrapDialog } from "./BootstrapDialog.js";
import { LoadingFullScreen } from "../LoadingFullScreen.js";

const { Component, xml, useState, toRaw } = owl;

export class ServerDialog extends Component {
    static props = {};
    static components = { BootstrapDialog, LoadingFullScreen };

    setup() {
        this.store = toRaw(useStore());
        this.state = useState({ waitRestart: false, loading: false, error: null });
        this.form = useState({
            token: "",
            iotname: this.store.base.hostname,
        });
    }

    async connectToServer() {
        this.state.loading = true;
        this.state.error = null;
        try {
            const data = await this.store.rpc({
                url: "/hw_posbox_homepage/connect_to_server",
                method: "POST",
                params: this.form,
            });

            if (data.status === "success") {
                this.state.waitRestart = true;
            } else {
                this.state.error = data.message;
            }
        } catch {
            console.warn("Error while fetching data");
        }
        this.state.loading = false;
    }

    async clearConfiguration() {
        try {
            const data = await this.store.rpc({
                url: "/hw_posbox_homepage/server_clear",
            });

            if (data.status === "success") {
                this.state.waitRestart = true;
            }
        } catch {
            console.warn("Error while clearing configuration");
        }
    }

    static template = xml`
        <LoadingFullScreen t-if="this.state.waitRestart">
            <t t-set-slot="body">
                Processing your request please wait...
            </t>
        </LoadingFullScreen>

        <BootstrapDialog identifier="'server-configuration'" btnName="'Configure'">
            <t t-set-slot="header">
                Configure Odoo Server
            </t>
            <t t-set-slot="body">
                <div class="alert alert-warning fs-6" role="alert">
                    Paste the token from the Connect wizard in your Odoo instance in the Server Token field.
                    If you change the IoT Box Name, your IoT Box will need a reboot.
                </div>
                <div class="mt-3">
                    <div class="input-group-sm mb-3" t-if="this.store.isLinux">
                        <label for="iotname">IoT Name</label>
                        <input name="iotname" type="text" class="form-control" t-model="this.form.iotname" />
                        <small t-if="!this.form.iotname" class="text-danger">Please enter a correct name</small>
                    </div>
                    <div class="input-group-sm mb-3">
                        <label for="token">Server Token</label>
                        <input name="token" type="text" class="form-control" t-model="this.form.token" />
                        <small t-if="!this.form.token" class="text-danger">Please enter a server token</small>
                    </div>
                    <div class="d-flex justify-content-end gap-2">
                        <button type="submit" class="btn btn-warning btn-sm" t-att-disabled="this.state.loading" t-on-click="connectToServer">Connect</button>
                    </div>
                    <small t-if="this.state.error" class="text-danger" t-esc="this.state.error"/>
                </div>
            </t>
            <t t-set-slot="footer">
                <div class="d-flex justify-content-between w-100">
                    <div style="font-size: 13px;">
                        <p class="m-0">Your current server is:<br/> <strong t-esc="this.store.base.server_status" /></p>
                    </div>
                    <div class="d-flex gap-2">
                        <button class="btn btn-danger btn-sm" t-on-click="clearConfiguration">Clear configuration</button>
                        <button type="button" class="btn btn-primary btn-sm" data-bs-dismiss="modal">Close</button>
                    </div>
                </div>
            </t>
        </BootstrapDialog>
    `;
}

```

## File: static\src\app\components\dialog\SixDialog.js

```javascript
/* global owl */

import useStore from "../../hooks/useStore.js";
import { BootstrapDialog } from "./BootstrapDialog.js";
import { LoadingFullScreen } from "../LoadingFullScreen.js";

const { Component, xml, useState, toRaw } = owl;

export class SixDialog extends Component {
    static props = {};
    static components = { BootstrapDialog, LoadingFullScreen };

    setup() {
        this.store = toRaw(useStore());
        this.state = useState({ waitRestart: false });
        this.form = useState({ terminal_id: this.store.base.six_terminal });
    }

    async connectToServer() {
        try {
            if (!this.form.terminal_id) {
                return;
            }

            const data = await this.store.rpc({
                url: "/hw_posbox_homepage/six_payment_terminal_add",
                method: "POST",
                params: this.form,
            });

            if (data.status === "success") {
                this.state.waitRestart = true;
            }
        } catch {
            console.warn("Error while fetching data");
        }
    }

    async clearConfiguration() {
        try {
            const data = await this.store.rpc({
                url: "/hw_posbox_homepage/six_payment_terminal_clear",
            });

            if (data.status === "success") {
                this.state.waitRestart = true;
            }
        } catch {
            console.warn("Error while clearing configuration");
        }
    }

    static template = xml`
        <LoadingFullScreen t-if="this.state.waitRestart">
            <t t-set-slot="body">
                Your IoT Box is currently processing your request. Please wait.
            </t>
        </LoadingFullScreen>

        <BootstrapDialog identifier="'six-configuration'" btnName="'Configure'">
            <t t-set-slot="header">
                Configure Six Terminal
            </t>
            <t t-set-slot="body">
                <div class="alert alert-warning fs-6" role="alert">
                    Set the Terminal ID (TID) of the terminal you want to use.
                </div>
                <div class="mt-3">
                    <div class="input-group-sm mb-3">
                        <label for="iotname">Terminal ID (Digit only)</label>
                        <input name="iotname" type="text" class="form-control" t-model="this.form.terminal_id" />
                        <small t-if="!this.form.terminal_id" class="text-danger">Please enter a correct terminal ID</small>
                    </div>
                    <div class="d-flex justify-content-end gap-2">
                        <button class="btn btn-danger btn-sm" t-on-click="clearConfiguration">Clear configuration</button>
                        <button type="submit" class="btn btn-warning btn-sm" t-on-click="connectToServer">Connect</button>
                    </div>
                </div>
            </t>
            <t t-set-slot="footer">
                <button type="button" class="btn btn-primary btn-sm" data-bs-dismiss="modal">Close</button>
            </t>
        </BootstrapDialog>
    `;
}

```

## File: static\src\app\components\dialog\UpdateDialog.js

```javascript
/* global owl */

import useStore from "../../hooks/useStore.js";
import { BootstrapDialog } from "./BootstrapDialog.js";
import { LoadingFullScreen } from "../LoadingFullScreen.js";

const { Component, xml, useState } = owl;

export class UpdateDialog extends Component {
    static props = {};
    static components = { BootstrapDialog, LoadingFullScreen };

    setup() {
        this.store = useStore();
        this.state = useState({
            initialization: true,
            loading: false,
            waitRestart: false,
            odooIsUpToDate: false,
            imageIsUpToDate: false,
            currentCommitHash: "",
        });
    }

    onClose() {
        this.state.initialization = [];
    }

    async getVersionInfo() {
        if (!this.store.isLinux) {
            this.state.odooIsUpToDate = true;
            this.state.imageIsUpToDate = true;
            this.state.initialization = false;
            return
        }
        try {
            const data = await this.store.rpc({
                url: "/hw_posbox_homepage/version_info",
            });

            this.state.odooIsUpToDate = data.odooIsUpToDate;
            this.state.imageIsUpToDate = data.imageIsUpToDate;
            this.state.currentCommitHash = data.currentCommitHash;
            this.state.initialization = false;
        } catch {
            console.warn("Error while fetching version info");
        }
    }

    async updateGitTree() {
        this.state.waitRestart = true;
        try {
            const data = await this.store.rpc({
                url: "/hw_posbox_homepage/update_git_tree",
                method: "POST",
            });
            if (data.status === "success") {
                this.state.isUpToDate = true;
            }
        } catch {
            console.warn("Error while updating IoT Box.");
        }
    }

    async forceUpdateIotHandlers() {
        this.state.waitRestart = true;
        try {
            await this.store.rpc({
                url: "/hw_posbox_homepage/load_iot_handlers",
            });
        } catch {
            console.warn("Error while downloading handlers from db.");
        }
    }

    get everythingIsUpToDate() {
        return this.state.odooIsUpToDate && this.state.imageIsUpToDate;
    }

    static template = xml`
        <LoadingFullScreen t-if="this.state.waitRestart">
            <t t-set-slot="body">
                Updating your device, please wait...
            </t>
        </LoadingFullScreen>

        <BootstrapDialog identifier="'update-configuration'" btnName="'Update'" onOpen.bind="getVersionInfo" onClose.bind="onClose">
            <t t-set-slot="header">
                <div>
                    Update
                    <a href="https://www.odoo.com/documentation/17.0/applications/general/iot/config/updating_iot.html" class="fa fa-question-circle text-decoration-none text-dark" target="_blank"></a>
                </div>
            </t>
            <t t-set-slot="body">
                <div t-if="this.state.initialization" class="position-absolute top-0 start-0 bg-white h-100 w-100 justify-content-center align-items-center d-flex flex-column gap-3" style="z-index: 9999">
                    <div class="spinner-border" role="status">
                        <span class="visually-hidden">Loading...</span>
                    </div>
                    <p>Currently fetching update data...</p>
                </div>

                <div class="mb-3" t-if="this.store.isLinux">
                    <h6>Operating System Update</h6>
                    <div t-if="this.state.imageIsUpToDate" class="text-success px-2 small">
                        Operating system is up to date
                    </div>
                    <div t-else="" class="alert alert-warning small mb-0">
                        A new version of the operating system is available, see:
                        <a href="https://www.odoo.com/documentation/17.0/applications/general/iot/config/updating_iot.html#flashing-the-sd-card-on-iot-box" target="_blank" class="alert-link">
                            Flashing the SD Card on IoT Box
                        </a>
                    </div>
                    <div t-if="this.store.dev" class="alert alert-light small">
                        <a href="https://nightly.odoo.com/master/iotbox/" target="_blank" class="alert-link">
                            Current: <t t-esc="this.store.base.version"/>
                        </a>
                    </div>
                </div>

                <div class="mb-3" t-if="this.store.isLinux">
                    <h6>IoT Box Update</h6>
                    <div t-if="this.state.odooIsUpToDate" class="text-success px-2 small">
                        IoT Box is up to date.
                    </div>
                    <div t-else="" class="d-flex justify-content-between align-items-center alert alert-warning small">
                        A new version of the IoT Box is available
                        <button class="btn btn-primary btn-sm" t-on-click="updateGitTree">Update</button>
                    </div>
                    <div t-if="this.store.dev" class="alert alert-light small">
                        Current: 
                        <a t-att-href="'https://github.com/odoo/odoo/commit/' + this.state.currentCommitHash" target="_blank" class="alert-link">
                            <t t-esc="this.state.currentCommitHash"/>
                        </a>
                    </div>
                </div>

                <h6>Drivers Update</h6>
                <div class="d-flex gap-2">
                    <button class="btn btn-secondary btn-sm" t-on-click="forceUpdateIotHandlers">
                        Force Drivers Update
                    </button>
                </div>
            </t>
            <t t-set-slot="footer">
                <button 
                    type="button"
                    t-att-class="'btn btn-sm ' + (this.everythingIsUpToDate ? 'btn-primary' : 'btn-secondary')"
                    data-bs-dismiss="modal">
                    Close
                </button>
            </t>
        </BootstrapDialog>
    `;
}

```

## File: static\src\app\components\dialog\WifiDialog.js

```javascript
/* global owl */

import useStore from "../../hooks/useStore.js";
import { BootstrapDialog } from "./BootstrapDialog.js";
import { LoadingFullScreen } from "../LoadingFullScreen.js";

const { Component, xml, useState } = owl;

export class WifiDialog extends Component {
    static props = {};
    static components = { BootstrapDialog, LoadingFullScreen };

    setup() {
        this.store = useStore();
        this.state = useState({
            scanning: true,
            loading: false,
            waitRestart: false,
            availableWifi: [],
        });
        this.form = useState({
            essid: "",
            password: "",
            persistent: false,
        });
    }

    onClose() {
        this.state.availableWifi = [];
        this.state.scanning = true;
        this.form.essid = "";
        this.form.password = "";
        this.form.persistent = false;
    }

    async getWifiNetworks() {
        try {
            const data = await this.store.rpc({
                url: "/hw_posbox_homepage/wifi",
            });

            this.state.availableWifi = data;
            this.state.scanning = false;
        } catch {
            console.warn("Error while fetching data");
        }
    }

    async connectToWifi() {
        if (!this.form.essid || !this.form.password) {
            return;
        }

        const data = await this.store.rpc({
            url: "/hw_posbox_homepage/update_wifi",
            method: "POST",
            params: this.form,
        });

        if (data.status === "success") {
            this.state.waitRestart = true;
        }
    }

    async clearConfiguration() {
        try {
            const data = await this.store.rpc({
                url: "/hw_posbox_homepage/wifi_clear",
            });

            if (data.status === "success") {
                this.state.waitRestart = true;
            }
        } catch {
            console.warn("Error while clearing configuration");
        }
    }

    static template = xml`
        <LoadingFullScreen t-if="this.state.waitRestart">
            <t t-set-slot="body">
                Processing your request please wait...
            </t>
        </LoadingFullScreen>

        <BootstrapDialog identifier="'wifi-configuration'" btnName="'Configure'" onOpen.bind="getWifiNetworks" onClose.bind="onClose">
            <t t-set-slot="header">
                Configure WIFI
            </t>
            <t t-set-slot="body">
                <div t-if="this.state.scanning" class="position-absolute top-0 start-0 bg-white h-100 w-100 justify-content-center align-items-center d-flex flex-column gap-3" style="z-index: 9999">
                    <div class="spinner-border" role="status">
                        <span class="visually-hidden">Loading...</span>
                    </div>
                    <p>Currently scanning for available networks...</p>
                </div>

                <div class="alert alert-warning fs-6" role="alert">
                    Here you can configure how the iotbox should connect to wireless networks.
                    Currently only Open and WPA networks are supported. When enabling the persistent checkbox,
                    the chosen network will be saved and the iotbox will attempt to connect to it every time it boots.
                </div>
                <div class="mt-3">
                    <div class="mb-3">
                        <label for="wifi-ssid">WIFI SSID</label>
                        <select name="essid" class="form-control" id="wifi-ssid" t-model="this.form.essid">
                            <option>Choose...</option>
                            <option t-foreach="this.state.availableWifi.filter((wifi) => wifi)" t-as="wifi" t-key="wifi" t-att-value="wifi">
                                <t t-esc="wifi" />
                            </option>
                        </select>
                        <small t-if="!this.form.essid" class="text-danger">Please select a network</small>
                    </div>

                    <div class="mb-3">
                        <label for="wifi-ssid">WIFI Password</label>
                        <input name="password" type="password" class="form-control" aria-label="Username" aria-describedby="basic-addon1" t-model="this.form.password" />
                        <small t-if="!this.form.password" class="text-danger">Please enter a password</small>
                    </div>

                    <div class="form-check">
                        <input name="persistent" class="form-check-input" type="checkbox" value="" id="persistent" t-model="this.form.persistent" />
                        <label class="form-check-label" for="persistent">
                            Persistent
                        </label>
                    </div>

                    <div class="d-flex justify-content-end gap-2">
                        <button type="submit" class="btn btn-danger btn-sm" t-on-click="clearConfiguration">Clear</button>
                        <button type="submit" class="btn btn-warning btn-sm" t-on-click="connectToWifi">Connect</button>
                    </div>
                </div>
            </t>
            <t t-set-slot="footer">
                <button type="button" class="btn btn-primary btn-sm" data-bs-dismiss="modal">Close</button>
            </t>
        </BootstrapDialog>
    `;
}

```

## File: static\src\app\hooks\useStore.js

```javascript
/* global owl */

const { useState, useEnv } = owl;

export default function useStore() {
    const env = useEnv();
    return useState(env.store);
}

```

## File: views\index.html

```html
<!DOCTYPE html>
<html>
    <head>
        <meta http-equiv="cache-control" content="no-cache" />
        <meta http-equiv="pragma" content="no-cache" />
        <meta name="viewport" content="width=device-width,initial-scale=1">

        <title>Odoo's IoT Box</title>

        <script src="/web/static/lib/owl/owl.js"></script>
        <script src="/web/static/lib/bootstrap/js/dist/dom/data.js"></script>
        <script src="/web/static/lib/bootstrap/js/dist/dom/event-handler.js"></script>
        <script src="/web/static/lib/bootstrap/js/dist/dom/manipulator.js"></script>
        <script src="/web/static/lib/bootstrap/js/dist/dom/selector-engine.js"></script>
        <script src="/web/static/lib/bootstrap/js/dist/base-component.js"></script>
        <script src="/web/static/lib/bootstrap/js/dist/alert.js"></script>
        <script src="/web/static/lib/bootstrap/js/dist/button.js"></script>
        <script src="/web/static/lib/bootstrap/js/dist/carousel.js"></script>
        <script src="/web/static/lib/bootstrap/js/dist/collapse.js"></script>
        <script src="/web/static/lib/bootstrap/js/dist/dropdown.js"></script>
        <script src="/web/static/lib/bootstrap/js/dist/modal.js"></script>
        <script src="/web/static/lib/bootstrap/js/dist/offcanvas.js"></script>
        <script src="/web/static/lib/bootstrap/js/dist/tooltip.js"></script>
        <script src="/web/static/lib/bootstrap/js/dist/popover.js"></script>
        <script src="/web/static/lib/bootstrap/js/dist/scrollspy.js"></script>
        <script src="/web/static/lib/bootstrap/js/dist/tab.js"></script>
        <script src="/web/static/lib/bootstrap/js/dist/toast.js"></script>

        <link href="/web/static/lib/bootstrap/dist/css/bootstrap.css" rel="stylesheet"/>
        <link href="/hw_posbox_homepage/static/src/app/css/override.css" rel="stylesheet"/>
        <link href="/web/static/src/libs/fontawesome/css/font-awesome.css" rel="stylesheet"/>
    </head>
    <body>
        <script src="/hw_posbox_homepage/static/src/app/main.js" type="module"></script>
    </body>
</html>

```

## File: views\logs.html

```html
<!DOCTYPE html>
<html>
    <head>
        <meta http-equiv="cache-control" content="no-cache" />
        <meta http-equiv="pragma" content="no-cache" />

        <title>Odoo's IoT Box Logs</title>

        <script>
            async function getLogs() {
                const result = await fetch('/hw_posbox_homepage/iot_logs');
                const data = await result.json();
                document.getElementById('logs').innerText = data.logs;
                document.getElementById('logs').scrollTop = document.getElementById('logs').scrollHeight;
            }
            document.addEventListener("DOMContentLoaded", function(event) {
                setInterval(getLogs, 1000);
            });
        </script>
    </head>
    <body style="width: 100%; background-color: black; color: white;">
        <pre id="logs"></pre>
    </body>
</html>

```

