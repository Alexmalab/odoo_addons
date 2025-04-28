# Odoo Module: hw_posbox_homepage

Category: Sales/Point Of Sale

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
    'category': 'Sales/Point Of Sale',
    'sequence': 6,
    'website': 'https://www.odoo.com/page/point-of-sale-hardware',
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
    'depends': ['hw_proxy'],
    'installable': False,
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import json
import jinja2
import subprocess
import socket
import sys
import netifaces
import odoo
from odoo import http
import os
from odoo.tools import misc
from pathlib import Path
import threading

from odoo.addons.hw_proxy.controllers import main as hw_proxy
from odoo.addons.web.controllers import main as web
from odoo.modules.module import get_resource_path
from odoo.addons.hw_drivers.tools import helpers
from odoo.addons.hw_drivers.controllers.driver import iot_devices
from odoo.http import Response

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
driver_list_template = jinja_env.get_template('driver_list.html')
remote_connect_template = jinja_env.get_template('remote_connect.html')
configure_wizard_template = jinja_env.get_template('configure_wizard.html')
six_payment_terminal_template = jinja_env.get_template('six_payment_terminal.html')
list_credential_template = jinja_env.get_template('list_credential.html')
upgrade_page_template = jinja_env.get_template('upgrade_page.html')

class IoTboxHomepage(web.Home):
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
        ssid = helpers.get_ssid()
        wired = subprocess.check_output(['cat', '/sys/class/net/eth0/operstate']).decode('utf-8').strip('\n')
        if wired == 'up':
            network = 'Ethernet'
        elif ssid:
            if helpers.access_point():
                network = 'Wifi access point'
            else:
                network = 'Wifi : ' + ssid
        else:
            network = 'Not Connected'

        iot_device = []
        for device in iot_devices:
            iot_device.append({
                'name': iot_devices[device].device_name + ' : ' + str(iot_devices[device].data['value']),
                'type': iot_devices[device].device_type.replace('_', ' '),
                'message': iot_devices[device].device_identifier + iot_devices[device].get_message()
            })

        return {
            'hostname': hostname,
            'ip': helpers.get_ip(),
            'mac': helpers.get_mac_address(),
            'iot_device_status': iot_device,
            'server_status': helpers.get_odoo_server_url() or 'Not Configured',
            'six_terminal': self.get_six_terminal(),
            'network_status': network,
            'version': helpers.get_version(),
            }

    @http.route('/', type='http', auth='none')
    def index(self):
        wifi = Path.home() / 'wifi_network.txt'
        remote_server = Path.home() / 'odoo-remote-server.conf'
        if (wifi.exists() == False or remote_server.exists() == False) and helpers.access_point():
            return "<meta http-equiv='refresh' content='0; url=http://" + helpers.get_ip() + ":8069/steps'>"
        else:
            return homepage_template.render(self.get_homepage_data())

    @http.route('/list_drivers', type='http', auth='none', website=True)
    def list_drivers(self):
        drivers_list = []
        for driver in os.listdir(get_resource_path('hw_drivers', 'drivers')):
            if driver != '__pycache__':
                drivers_list.append(driver)
        return driver_list_template.render({
            'title': "Odoo's IoT Box - Drivers list",
            'breadcrumb': 'Drivers list',
            'drivers_list': drivers_list,
            'server': helpers.get_odoo_server_url()
        })

    @http.route('/load_drivers', type='http', auth='none', website=True)
    def load_drivers(self):
        helpers.download_drivers(False)
        subprocess.check_call(["sudo", "service", "odoo", "restart"])
        return "<meta http-equiv='refresh' content='20; url=http://" + helpers.get_ip() + ":8069/list_drivers'>"

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
        helpers.add_credential(db_uuid, enterprise_code)
        subprocess.check_call(["sudo", "service", "odoo", "restart"])
        return "<meta http-equiv='refresh' content='20; url=http://" + helpers.get_ip() + ":8069'>"

    @http.route('/clear_credential', type='http', auth='none', cors='*', csrf=False)
    def clear_credential(self):
        helpers.unlink_file('odoo-db-uuid.conf')
        helpers.unlink_file('odoo-enterprise-code.conf')
        subprocess.check_call(["sudo", "service", "odoo", "restart"])
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
        return "<meta http-equiv='refresh' content='0; url=http://" + helpers.get_ip() + ":8069'>"

    @http.route('/drivers_clear', type='http', auth='none', cors='*', csrf=False)
    def clear_drivers_list(self):
        for driver in os.listdir(get_resource_path('hw_drivers', 'drivers')):
            if driver != '__pycache__':
                helpers.unlink_file(get_resource_path('hw_drivers', 'drivers', driver))
        return "<meta http-equiv='refresh' content='0; url=http://" + helpers.get_ip() + ":8069/list_drivers'>"

    @http.route('/server_connect', type='http', auth='none', cors='*', csrf=False)
    def connect_to_server(self, token, iotname):
        if token:
            credential = token.split('|')
            url = credential[0]
            token = credential[1]
            if len(credential) > 2:
                # IoT Box send token with db_uuid and enterprise_code only since V13
                db_uuid = credential[2]
                enterprise_code = credential[3]
                helpers.add_credential(db_uuid, enterprise_code)
        else:
            url = helpers.get_odoo_server_url()
            token = helpers.get_token()
        reboot = 'reboot'
        subprocess.check_call([get_resource_path('point_of_sale', 'tools/posbox/configuration/connect_to_server.sh'), url, iotname, token, reboot])
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
            subprocess.Popen(['ngrok', 'tcp', '-authtoken', auth_token, '-log', '/tmp/ngrok.log', '22'])
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
        helpers.write_file('odoo-six-payment-terminal.conf', terminal_id)
        subprocess.check_call(["sudo", "service", "odoo", "restart"])
        return 'http://' + helpers.get_ip() + ':8069'

    @http.route('/six_payment_terminal_clear', type='http', auth='none', cors='*', csrf=False)
    def clear_six_payment_terminal(self):
        helpers.unlink_file('odoo-six-payment-terminal.conf')
        subprocess.check_call(["sudo", "service", "odoo", "restart"])
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
            _logger.error('A error encountered : %s ' % e)
            return Response(str(e), status=500)

    @http.route('/hw_proxy/perform_flashing_download_raspbian', type='http', auth='none')
    def perform_flashing_download_raspbian(self):
        try:
            response = subprocess.check_output(['sudo', 'bash', '-c', '. /home/pi/odoo/addons/point_of_sale/tools/posbox/configuration/upgrade.sh; download_raspbian']).decode().split('\n')[-2]
            if response == 'Error_Raspbian_Download':
                raise Exception(response)
            return Response('success', status=200)
        except subprocess.CalledProcessError as e:
            raise Exception(e.output)
        except Exception as e:
            self.clean_partition()
            _logger.error('A error encountered : %s ' % e)
            return Response(str(e), status=500)

    @http.route('/hw_proxy/perform_flashing_copy_raspbian', type='http', auth='none')
    def perform_flashing_copy_raspbian(self):
        try:
            response = subprocess.check_output(['sudo', 'bash', '-c', '. /home/pi/odoo/addons/point_of_sale/tools/posbox/configuration/upgrade.sh; copy_raspbian']).decode().split('\n')[-2]
            if response == 'Error_Iotbox_Download':
                raise Exception(response)
            return Response('success', status=200)
        except subprocess.CalledProcessError as e:
            raise Exception(e.output)
        except Exception as e:
            self.clean_partition()
            _logger.error('A error encountered : %s ' % e)
            return Response(str(e), status=500)

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

## File: views\driver_list.html

```html
{% extends "layout.html" %}
{% from "loading.html" import loading_block_ui %}
{% block head %}
    <script>
        $(document).ready(function () {
            $('#load_driver_btn').on('click', function(e){
                e.preventDefault();
                $('.loading-block').removeClass('o_hide');
                $.ajax({
                    url: '/load_drivers',
                }).done(function () {
                    $('.message-status').html('Driver loaded successfully <br> Refreshing page');
                    setTimeout(function () {
                        location.reload(true);
                    }, 20000);
                }).fail(function () {
                    setTimeout(function () {
                        location.reload(true);
                    }, 20000);
                });
            });
        });
    </script>
{% endblock %}
{% block content %}
    <h2 class="text-center text-green">Drivers list</h2>
    <table align="center" cellpadding="3">
        {% for driver in drivers_list -%}
            <tr><td>{{ driver }}</td></tr>
        {%- endfor %}
    </table>
    {% if server %}
        <div style="margin-top: 20px;" class="text-center">
            <a id="load_driver_btn" class="btn" href='/load_drivers'>Load drivers</a>
        </div>
        <div class="text-center font-small" style="margin: 10px auto;">
            You can clear the drivers configuration
            <form style="display: inline-block;margin-left: 4px;" action='/drivers_clear'>
                <input class="btn btn-sm" type="submit" value="Clear"/>
            </form>
        </div>
    {% endif %}
    {{ loading_block_ui('Loading Drivers') }}
{% endblock %}

```

## File: views\homepage.html

```html
{% extends "layout.html" %}
{% block head %}
<style>
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
    .device-status .message {
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
</script>
{% endblock %}
{% block content %}
    <h2 class="text-center text-green">Your IoT Box is up and running</h2>
    <table align="center" cellpadding="3">
        <tr>
            <td class="heading">Name</td>
            <td> {{ hostname }} <a class="btn btn-sm float-right" href='/server'>configure</a></td>
        </tr>
        <tr>
            <td class="heading">Version</td>
            <td> {{ version }} <a class="btn btn-sm float-right" href='/hw_proxy/upgrade/'>update</a></td>
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
            <td>{{ network_status }} <a class="btn btn-sm float-right" href='/wifi'>configure wifi</a></td>
        </tr>
        <tr>
            <td class="heading">Server</td>
            <td><a href='{{ server_status }}' target=_blank>{{ server_status }}<a class="btn btn-sm float-right" href='/server'>configure</a></td>
        </tr>
        {% if server_status != "Not Configured" %}
        <tr>
            <td class="heading">Six payment terminal</td>
            <td>{{ six_terminal }} <a class="btn btn-sm float-right" href='/six_payment_terminal'>configure</a></td>
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
                                        <div class="message">{{ device['message'] }}</div>
                                    </div>
                                {% endfor %}
                            </div>
                        </div>
                    {% endfor %}
                </div>
                <br><center><a class="btn btn-sm" href='/list_drivers'>drivers list</a></center>
            </td>
        </tr>
    </table>
    <div style="margin: 20px auto 10px auto;" class="text-center">
        <a class="btn" href='/point_of_sale/display'>POS Display</a>
        <a class="btn" style="margin-left: 10px;" href='/remote_connect'>Remote Debug</a>
        <a target="_blank" class="btn" style="margin-left: 10px;" href="http://{{ ip }}:631">Printers server</a>
        {% if server_status != "Not Configured" %}
        <a class="btn" style="margin-left: 10px;" href='/list_credential'>Credential</a>
        {% endif %}
    </div>
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
            .float-right {
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
            <a href='https://www.odoo.com/documentation/13.0/applications/productivity/iot.html'>Documentation</a>
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
            <img src="/web/static/src/img/spin.png" style="animation: spin 4s infinite linear;" alt="Loading...">
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
                <td>Terminal ID</td>
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
                        var cpt = 110;
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
                        await $.ajax({url: '/hw_proxy/perform_flashing_download_raspbian/'}).promise();
                        $('.message-status').text('Download file for new boot partition.');
                        await $.ajax({url: '/hw_proxy/perform_flashing_copy_raspbian/'}).promise();
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
        the <a href='http://nightly.odoo.com/master/posbox/iotbox/iotbox-latest.zip'>latest image</a>. The upgrade
        procedure is explained into to the
        <a href='https://www.odoo.com/documentation/13.0/applications/productivity/iot.html'>IoTBox manual</a>
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

