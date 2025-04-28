# Odoo Module: hr_attendance

Category: Human Resources/Attendances

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models


def post_init_hook(env):
    env['res.company']._check_hr_presence_control(True)


def uninstall_hook(env):
    env['res.company']._check_hr_presence_control(False)

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.


{
    'name': 'Attendances',
    'version': '2.0',
    'category': 'Human Resources/Attendances',
    'sequence': 240,
    'summary': 'Track employee attendance',
    'description': """
This module aims to manage employee's attendances.
==================================================

Keeps account of the attendances of the employees on the basis of the
actions(Check in/Check out) performed by them.
       """,
    'website': 'https://www.odoo.com/app/employees',
    'depends': ['hr', 'barcodes'],
    'data': [
        'data/hr_attendance_data.xml',
        'security/hr_attendance_security.xml',
        'security/ir.model.access.csv',
        'views/hr_attendance_view.xml',
        'views/hr_attendance_overtime_view.xml',
        'views/hr_department_view.xml',
        'views/hr_employee_view.xml',
        'views/res_config_settings_views.xml',
        'views/hr_attendance_kiosk_templates.xml'
    ],
    'demo': [
        'data/hr_attendance_demo.xml'
    ],
    'installable': True,
    'application': True,
    'assets': {
        'web.assets_backend': [
            'hr_attendance/static/src/**/*.js',
            'hr_attendance/static/src/**/*.xml',
            'hr_attendance/static/src/scss/views/*.scss'
        ],
        'web.qunit_suite_tests': [
            'hr_attendance/static/tests/hr_attendance_mock_server.js',
        ],
        'web.qunit_mobile_suite_tests': [
            'hr_attendance/static/tests/hr_attendance_mock_server.js',
        ],
        'hr_attendance.assets_public_attendance': [
            # Define attendance variables (takes priority)
            'hr_attendance/static/src/scss/kiosk/primary_variables.scss',

            # Front-end libraries
            ('include', 'web._assets_helpers'),
            ('include', 'web._assets_primary_variables'),
            'hr_attendance/static/src/scss/kiosk/bootstrap_overridden.scss',
            ('include', 'web._assets_frontend_helpers'),
            'web/static/lib/jquery/jquery.js',
            'web/static/src/scss/pre_variables.scss',
            'web/static/lib/bootstrap/scss/_variables.scss',
            'web/static/lib/bootstrap/scss/_variables-dark.scss',
            'web/static/lib/bootstrap/scss/_maps.scss',
            ('include', 'web._assets_bootstrap_frontend'),
            ('include', 'web._assets_bootstrap_backend'),
            '/web/static/lib/odoo_ui_icons/*',
            '/web/static/lib/bootstrap/scss/_functions.scss',
            '/web/static/lib/bootstrap/scss/_mixins.scss',
            '/web/static/lib/bootstrap/scss/utilities/_api.scss',
            'web/static/src/libs/fontawesome/css/font-awesome.css',
            ('include', 'web._assets_core'),

            # Public Kiosk app and its components
            "hr_attendance/static/src/public_kiosk/**/*",
            'hr_attendance/static/src/components/**/*',

            'hr_attendance/static/src/scss/kiosk/hr_attendance.scss',
            "web/static/src/views/fields/formatters.js",

            # document link
            "web/static/src/session.js",
            "web/static/src/views/widgets/standard_widget_props.js",
            "web/static/src/views/widgets/documentation_link/*",

            # Barcode reader utils
            "barcodes/static/src/components/barcode_scanner.js",
            "barcodes/static/src/components/barcode_scanner.xml",
            "barcodes/static/src/components/barcode_scanner.scss",
            "barcodes/static/src/barcode_service.js",

        ]
    },
    'license': 'LGPL-3',
    'post_init_hook': 'post_init_hook',
    'uninstall_hook': 'uninstall_hook',
}

```

## File: controllers\main.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.service.common import exp_version
from odoo import http, _
from odoo.http import request
from odoo.osv import expression
from odoo.tools import float_round, py_to_js_locale, SQL
from odoo.tools.image import image_data_uri

import datetime

class HrAttendance(http.Controller):
    @staticmethod
    def _get_company(token):
        company = request.env['res.company'].sudo().search([('attendance_kiosk_key', '=', token)])
        return company

    @staticmethod
    def _get_user_attendance_data(employee):
        response = {}
        if employee:
            response = {
                'id': employee.id,
                'hours_today': float_round(employee.hours_today, precision_digits=2),
                'hours_previously_today': float_round(employee.hours_previously_today, precision_digits=2),
                'last_attendance_worked_hours': float_round(employee.last_attendance_worked_hours, precision_digits=2),
                'last_check_in': employee.last_check_in,
                'attendance_state': employee.attendance_state,
                'display_systray': employee.company_id.attendance_from_systray,
            }
        return response

    @staticmethod
    def _get_employee_info_response(employee):
        response = {}
        if employee:
            response = {
                **HrAttendance._get_user_attendance_data(employee),
                'employee_name': employee.name,
                'employee_avatar': employee.image_256 and image_data_uri(employee.image_256),
                'total_overtime': float_round(employee.total_overtime, precision_digits=2),
                'kiosk_delay': employee.company_id.attendance_kiosk_delay * 1000,
                'attendance': {'check_in': employee.last_attendance_id.check_in,
                               'check_out': employee.last_attendance_id.check_out},
                'overtime_today': request.env['hr.attendance.overtime'].sudo().search([
                    ('employee_id', '=', employee.id), ('date', '=', datetime.date.today()),
                    ('adjustment', '=', False)]).duration or 0,
                'use_pin': employee.company_id.attendance_kiosk_use_pin,
                'display_overtime': employee.company_id.hr_attendance_display_overtime
            }
        return response

    @staticmethod
    def _get_geoip_response(mode, latitude=False, longitude=False):
        return {
            'city': request.geoip.city.name or _('Unknown'),
            'country_name': request.geoip.country.name or request.geoip.continent.name or _('Unknown'),
            'latitude': latitude or request.geoip.location.latitude or False,
            'longitude': longitude or request.geoip.location.longitude or False,
            'ip_address': request.geoip.ip,
            'browser': request.httprequest.user_agent.browser,
            'mode': mode
        }

    @http.route('/hr_attendance/kiosk_mode_menu/<int:company_id>', auth='user', type='http')
    def kiosk_menu_item_action(self, company_id):
        if request.env.user.has_group("hr_attendance.group_hr_attendance_manager"):
            # Auto log out will prevent users from forgetting to log out of their session
            # before leaving the kiosk mode open to the public. This is a prevention security
            # measure.
            if self.has_password():
                request.session.logout(keep_db=True)
            return request.redirect(request.env['res.company'].browse(company_id).attendance_kiosk_url)
        else:
            return request.not_found()

    @http.route('/hr_attendance/kiosk_keepalive', auth='user', type='json')
    def kiosk_keepalive(self):
        request.session.touch()
        return {}

    @http.route(["/hr_attendance/<token>"], type='http', auth='public', website=True, sitemap=True)
    def open_kiosk_mode(self, token, from_trial_mode=False):
        company = self._get_company(token)
        if not company:
            return request.not_found()
        else:
            department_list = [{'id': dep["id"],
                                 'name': dep["name"],
                                 'count': dep["total_employee"]
                                 } for dep in request.env['hr.department'].sudo().search_read(domain=[('company_id', '=', company.id)],
                                                                                              fields=["id",
                                                                                                      "name",
                                                                                                      "total_employee"])]
            has_password = self.has_password()
            if not from_trial_mode and has_password:
                request.session.logout(keep_db=True)
            if (from_trial_mode or not has_password):
                kiosk_mode = "settings"
            else:
                kiosk_mode = company.attendance_kiosk_mode
            version_info = exp_version()
            return request.render(
                'hr_attendance.public_kiosk_mode',
                {
                    'kiosk_backend_info': {
                        'token': token,
                        'company_id': company.id,
                        'company_name': company.name,
                        'departments': department_list,
                        'kiosk_mode': kiosk_mode,
                        'from_trial_mode': from_trial_mode,
                        'barcode_source': company.attendance_barcode_source,
                        'lang': py_to_js_locale(company.partner_id.lang or company.env.lang),
                        'server_version_info': version_info.get('server_version_info'),
                    },
                }
            )

    @http.route('/hr_attendance/attendance_employee_data', type="json", auth="public")
    def employee_attendance_data(self, token, employee_id):
        company = self._get_company(token)
        if company:
            employee = request.env['hr.employee'].sudo().browse(employee_id)
            if employee.company_id == company:
                return self._get_employee_info_response(employee)
        return {}

    @http.route('/hr_attendance/attendance_barcode_scanned', type="json", auth="public")
    def scan_barcode(self, token, barcode):
        company = self._get_company(token)
        if company:
            employee = request.env['hr.employee'].sudo().search([('barcode', '=', barcode), ('company_id', '=', company.id)], limit=1)
            if employee:
                employee._attendance_action_change(self._get_geoip_response('kiosk'))
                return self._get_employee_info_response(employee)
        return {}

    def manual_selection(self, token, employee_id, pin_code):
        return self.manual_selection_with_geolocation(token, employee_id, pin_code)

    @http.route('/hr_attendance/manual_selection', type="json", auth="public")
    def manual_selection_with_geolocation(self, token, employee_id, pin_code, latitude=False, longitude=False):
        company = self._get_company(token)
        if company:
            employee = request.env['hr.employee'].sudo().browse(employee_id)
            if employee.company_id == company and ((not company.attendance_kiosk_use_pin) or (employee.pin == pin_code)):
                employee.sudo()._attendance_action_change(self._get_geoip_response('kiosk', latitude=latitude, longitude=longitude))
                return self._get_employee_info_response(employee)
        return {}

    @http.route('/hr_attendance/employees_infos', type="json", auth="public")
    def employees_infos(self, token, limit, offset, domain):
        company = self._get_company(token)
        if company:
            domain = expression.AND([domain, [('company_id', '=', company.id)]])
            employees = request.env['hr.employee'].sudo().search_fetch(domain, ['id', 'display_name', 'job_id'],
                limit=limit, offset=offset, order="name, id")
            employees_data = [{
                'id': employee.id,
                'display_name': employee.display_name,
                'job_id': employee.job_id.name,
                'avatar': image_data_uri(employee.avatar_128)
            } for employee in employees]
            return {'records': employees_data, 'length': request.env['hr.employee'].sudo().search_count(domain)}
        return []

    @http.route('/hr_attendance/systray_check_in_out', type="json", auth="user")
    def systray_attendance(self, latitude=False, longitude=False):
        employee = request.env.user.employee_id
        geo_ip_response = self._get_geoip_response(mode='systray',
                                                  latitude=latitude,
                                                  longitude=longitude)
        employee._attendance_action_change(geo_ip_response)
        return self._get_employee_info_response(employee)

    @http.route('/hr_attendance/attendance_user_data', type="json", auth="user", readonly=True)
    def user_attendance_data(self):
        employee = request.env.user.employee_id
        return self._get_user_attendance_data(employee)

    def has_password(self):
        # With this method we try to know whether it's the user is on trial mode or not.
        # We assume that in trial, people have not configured their password yet and their password should be empty.
        request.env.cr.execute(
            SQL('''
                SELECT COUNT(password)
                  FROM res_users
                 WHERE id=%(user_id)s
                   AND password IS NOT NULL
                 LIMIT 1
                ''', user_id=request.env.user.id))
        return bool(request.env.cr.fetchone()[0])

    @http.route('/hr_attendance/is_fresh_db', type="json", auth="public")
    def is_fresh_db(self, token):
        company = self._get_company(token)
        if company:
            users = request.env['res.users'].sudo().search([])
            return len(users) == 1 and not users[0].employee_id.barcode
        return False

    @http.route('/hr_attendance/set_user_barcode', type="json", auth="public")
    def set_user_barcode(self, token, barcode):
        company = self._get_company(token)
        if company and self.is_fresh_db(token):
            request.env.user.employee_id.barcode = barcode
            return True
        return False

    @http.route('/hr_attendance/set_settings', type="json", auth="public")
    def set_attendance_settings(self, token, mode):
        company = self._get_company(token)
        if company:
            request.env.user.company_id.attendance_kiosk_mode = mode

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-

from . import main

```

## File: data\hr_attendance_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="hr_attendance_check_out_cron" model="ir.cron">
            <field name="name">Attendance: Automatically check-out employees</field>
            <field name="model_id" ref="model_hr_attendance"/>
            <field name="state">code</field>
            <field name="code">model._cron_auto_check_out()</field>
            <field name="interval_number">4</field>
            <field name="interval_type">hours</field>
        </record>

        <record id="hr_attendance_absence_cron" model="ir.cron">
            <field name="name">Attendance: Detect Absences for employees</field>
            <field name="model_id" ref="model_hr_attendance"/>
            <field name="state">code</field>
            <field name="code">model._cron_absence_detection()</field>
            <field name="interval_number">4</field>
            <field name="interval_type">hours</field>
        </record>
    </data>
</odoo>

```

## File: data\hr_attendance_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="base.user_demo" model="res.users">
            <field name="groups_id" eval="[(3, ref('hr_attendance.group_hr_attendance_manager'))]"/>
        </record>

        <record id="hr.employee_al" model="hr.employee">
            <field name="barcode">123</field>
        </record>

        <record id="hr.employee_admin" model="hr.employee">
            <field name="barcode">456</field>
        </record>

        <record id="attendance_root1" model="hr.attendance">
            <field eval="(datetime.now()+relativedelta(months=-1, days=-1)).strftime('%Y-%m-%d 08:00:24')" name="check_in"/>
            <field eval="(datetime.now()+relativedelta(months=-1, days=-1)).strftime('%Y-%m-%d 12:01:33')" name="check_out"/>
            <field name="employee_id" ref="hr.employee_admin"/>
        </record>

        <record id="attendance_root2" model="hr.attendance">
            <field eval="(datetime.now()+relativedelta(months=-1, days=-1)).strftime('%Y-%m-%d 13:02:58')" name="check_in"/>
            <field eval="(datetime.now()+relativedelta(months=-1, days=-1)).strftime('%Y-%m-%d 18:09:22')" name="check_out"/>
            <field name="employee_id" ref="hr.employee_admin"/>
        </record>

        <record id="attendance1" model="hr.attendance">
            <field eval="(datetime.now()+relativedelta(months=-1)).strftime('%Y-%m-01 08:21')" name="check_in"/>
            <field eval="(datetime.now()+relativedelta(months=-1)).strftime('%Y-%m-01 15:51')" name="check_out"/>
            <field name="employee_id" ref="hr.employee_qdp"/>
        </record>

        <record id="attendance2" model="hr.attendance">
            <field eval="(datetime.now()+relativedelta(months=-1)).strftime('%Y-%m-02 08:47')" name="check_in"/>
            <field eval="(datetime.now()+relativedelta(months=-1)).strftime('%Y-%m-02 15:53')" name="check_out"/>
            <field name="employee_id" ref="hr.employee_qdp"/>
        </record>

        <record id="attendance3" model="hr.attendance">
            <field eval="(datetime.now()+relativedelta(months=-1)).strftime('%Y-%m-03 08:32')" name="check_in"/>
            <field eval="(datetime.now()+relativedelta(months=-1)).strftime('%Y-%m-03 15:22')" name="check_out"/>
            <field name="employee_id" ref="hr.employee_qdp"/>
        </record>

        <record id="attendance4" model="hr.attendance">
            <field eval="(datetime.now()+relativedelta(months=-1)).strftime('%Y-%m-04 08:01')" name="check_in"/>
            <field eval="(datetime.now()+relativedelta(months=-1)).strftime('%Y-%m-04 16:21')" name="check_out"/>
            <field name="employee_id" ref="hr.employee_qdp"/>
        </record>

        <record id="attendance5" model="hr.attendance">
            <field eval="(datetime.now()+relativedelta(months=-1)).strftime('%Y-%m-05 10:10')" name="check_in"/>
            <field eval="(datetime.now()+relativedelta(months=-1)).strftime('%Y-%m-05 14:42')" name="check_out"/>
            <field name="employee_id" ref="hr.employee_qdp"/>
        </record>

        <record id="attendance6" model="hr.attendance">
            <field eval="(datetime.now()+relativedelta(months=-1)).strftime('%Y-%m-06 10:10')" name="check_in"/>
            <field eval="(datetime.now()+relativedelta(months=-1)).strftime('%Y-%m-06 17:34')" name="check_out"/>
            <field name="employee_id" ref="hr.employee_qdp"/>
        </record>

        <record id="attendance7" model="hr.attendance">
            <field eval="(datetime.now()+relativedelta(months=-1)).strftime('%Y-%m-07 08:21')" name="check_in"/>
            <field eval="(datetime.now()+relativedelta(months=-1)).strftime('%Y-%m-07 17:29')" name="check_out"/>
            <field name="employee_id" ref="hr.employee_qdp"/>
        </record>

        <record id="attendance8" model="hr.attendance">
            <field eval="(datetime.now()+relativedelta(months=-1)).strftime('%Y-%m-08 09:21')" name="check_in"/>
            <field eval="(datetime.now()+relativedelta(months=-1)).strftime('%Y-%m-08 14:54')" name="check_out"/>
            <field name="employee_id" ref="hr.employee_qdp"/>
        </record>

        <record id="attendance9" model="hr.attendance">
            <field eval="(datetime.now()+relativedelta(months=-1)).strftime('%Y-%m-09 10:32')" name="check_in"/>
            <field eval="(datetime.now()+relativedelta(months=-1)).strftime('%Y-%m-09 17:31')" name="check_out"/>
            <field name="employee_id" ref="hr.employee_qdp"/>
        </record>

        <record id="attendance10" model="hr.attendance">
            <field eval="(datetime.now()+relativedelta(months=-1)).strftime('%Y-%m-10 08:00')" name="check_in"/>
            <field eval="(datetime.now()+relativedelta(months=-1)).strftime('%Y-%m-10 17:00')" name="check_out"/>
            <field name="employee_id" ref="hr.employee_qdp"/>
        </record>
    </data>
</odoo>

```

## File: data\scenarios\hr_attendance_scenario.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- Calendar -->
        <record id="resource_calendar_std_38h" model="resource.calendar" forcecreate="1">
            <field name="name">Standard 32 hours/week (4 work days, friday free)</field>
            <field name="company_id" eval="False"/>
            <field name="hours_per_day">8</field>
            <field name="attendance_ids"
                eval="[(5, 0, 0),
                    (0, 0, {'name': 'Monday Morning', 'dayofweek': '0', 'hour_from': 8, 'hour_to': 12, 'day_period': 'morning'}),
                    (0, 0, {'name': 'Monday Lunch', 'dayofweek': '0', 'hour_from': 12, 'hour_to': 13, 'day_period': 'lunch'}),
                    (0, 0, {'name': 'Monday Afternoon', 'dayofweek': '0', 'hour_from': 13, 'hour_to': 17, 'day_period': 'afternoon'}),
                    (0, 0, {'name': 'Tuesday Morning', 'dayofweek': '1', 'hour_from': 8, 'hour_to': 12, 'day_period': 'morning'}),
                    (0, 0, {'name': 'Tuesday Lunch', 'dayofweek': '1', 'hour_from': 12, 'hour_to': 13, 'day_period': 'lunch'}),
                    (0, 0, {'name': 'Tuesday Afternoon', 'dayofweek': '1', 'hour_from': 13, 'hour_to': 17, 'day_period': 'afternoon'}),
                    (0, 0, {'name': 'Wednesday Morning', 'dayofweek': '2', 'hour_from': 8, 'hour_to': 12, 'day_period': 'morning'}),
                    (0, 0, {'name': 'Wednesday Lunch', 'dayofweek': '2', 'hour_from': 12, 'hour_to': 13, 'day_period': 'lunch'}),
                    (0, 0, {'name': 'Wednesday Afternoon', 'dayofweek': '2', 'hour_from': 13, 'hour_to': 17, 'day_period': 'afternoon'}),
                    (0, 0, {'name': 'Thursday Morning', 'dayofweek': '3', 'hour_from': 8, 'hour_to': 12, 'day_period': 'morning'}),
                    (0, 0, {'name': 'Thursday Lunch', 'dayofweek': '3', 'hour_from': 12, 'hour_to': 13, 'day_period': 'lunch'}),
                    (0, 0, {'name': 'Thursday Afternoon', 'dayofweek': '3', 'hour_from': 13, 'hour_to': 17, 'day_period': 'afternoon'}),
                ]"
            />
        </record>

        <!-- Employee -->
        <record id="hr.employee_mw" model="hr.employee" forcecreate="1">
            <field name="barcode">123</field>
        </record>

        <record id="hr.employee_eg" model="hr.employee" forcecreate="1">
            <field name="resource_calendar_id" ref="resource_calendar_std_38h"/>
            <field name="barcode">456</field>
        </record>

        <record id="hr.employee_sj" model="hr.employee" forcecreate="1">
            <field name="pin">789</field>
        </record>
    </data>
</odoo>
```

## File: models\hr_attendance.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import pytz

from calendar import monthrange
from collections import defaultdict
from datetime import datetime, timedelta
from dateutil.relativedelta import relativedelta
from operator import itemgetter
from pytz import timezone
from random import randint

from odoo.http import request
from odoo import models, fields, api, exceptions, _
from odoo.addons.resource.models.utils import Intervals
from odoo.osv.expression import AND, OR
from odoo.tools.float_utils import float_is_zero
from odoo.exceptions import AccessError
from odoo.tools import convert, format_duration, format_time, format_datetime
from odoo.tools.float_utils import float_compare

def get_google_maps_url(latitude, longitude):
    return "https://maps.google.com?q=%s,%s" % (latitude, longitude)


class HrAttendance(models.Model):
    _name = "hr.attendance"
    _description = "Attendance"
    _order = "check_in desc"
    _inherit = "mail.thread"

    def _default_employee(self):
        return self.env.user.employee_id

    employee_id = fields.Many2one('hr.employee', string="Employee", default=_default_employee, required=True,
        ondelete='cascade', index=True, group_expand='_read_group_employee_id')
    department_id = fields.Many2one('hr.department', string="Department", related="employee_id.department_id",
        readonly=True)
    manager_id = fields.Many2one(comodel_name='hr.employee', related="employee_id.parent_id", readonly=True,
        export_string_translation=False)
    check_in = fields.Datetime(string="Check In", default=fields.Datetime.now, required=True, tracking=True)
    check_out = fields.Datetime(string="Check Out", tracking=True)
    worked_hours = fields.Float(string='Worked Hours', compute='_compute_worked_hours', store=True, readonly=True)
    color = fields.Integer(compute='_compute_color')
    overtime_hours = fields.Float(string="Over Time", compute='_compute_overtime_hours', store=True)
    overtime_status = fields.Selection(selection=[('to_approve', "To Approve"),
                                                  ('approved', "Approved"),
                                                  ('refused', "Refused")], compute="_compute_overtime_status", store=True, tracking=True, readonly=False)
    validated_overtime_hours = fields.Float(string="Extra Hours", compute='_compute_validated_overtime_hours', store=True, readonly=False, tracking=True)
    no_validated_overtime_hours = fields.Boolean(compute='_compute_no_validated_overtime_hours')
    in_latitude = fields.Float(string="Latitude", digits=(10, 7), readonly=True, aggregator=None)
    in_longitude = fields.Float(string="Longitude", digits=(10, 7), readonly=True, aggregator=None)
    in_country_name = fields.Char(string="Country", help="Based on IP Address", readonly=True)
    in_city = fields.Char(string="City", readonly=True)
    in_ip_address = fields.Char(string="IP Address", readonly=True)
    in_browser = fields.Char(string="Browser", readonly=True)
    in_mode = fields.Selection(string="Mode",
                               selection=[('kiosk', "Kiosk"),
                                          ('systray', "Systray"),
                                          ('manual', "Manual"),
                                          ('technical', 'Technical')],
                               readonly=True,
                               default='manual')
    out_latitude = fields.Float(digits=(10, 7), readonly=True, aggregator=None)
    out_longitude = fields.Float(digits=(10, 7), readonly=True, aggregator=None)
    out_country_name = fields.Char(help="Based on IP Address", readonly=True)
    out_city = fields.Char(readonly=True)
    out_ip_address = fields.Char(readonly=True)
    out_browser = fields.Char(readonly=True)
    out_mode = fields.Selection(selection=[('kiosk', "Kiosk"),
                                           ('systray', "Systray"),
                                           ('manual', "Manual"),
                                           ('technical', 'Technical'),
                                           ('auto_check_out', 'Automatic Check-Out')],
                                readonly=True,
                                default='manual')
    expected_hours = fields.Float(compute="_compute_expected_hours", store=True, aggregator="sum")

    @api.depends("worked_hours", "overtime_hours")
    def _compute_expected_hours(self):
        for attendance in self:
            attendance.expected_hours = attendance.worked_hours - attendance.overtime_hours

    def _compute_color(self):
        for attendance in self:
            if attendance.check_out:
                attendance.color = 1 if attendance.worked_hours > 16 or attendance.out_mode == 'technical' else 0
            else:
                attendance.color = 1 if attendance.check_in < (datetime.today() - timedelta(days=1)) else 10

    @api.depends('worked_hours')
    def _compute_overtime_hours(self):
        att_progress_values = dict()
        negative_overtime_attendances = defaultdict(lambda: False)
        if self.employee_id:
            self.env['hr.attendance'].flush_model(['worked_hours'])
            self.env['hr.attendance.overtime'].flush_model(['duration'])
            self.env.cr.execute('''
                WITH employee_time_zones AS (
                    SELECT employee.id AS employee_id,
                           calendar.tz AS timezone
                      FROM hr_employee employee
                INNER JOIN resource_calendar calendar
                        ON calendar.id = employee.resource_calendar_id
                )
                SELECT att.id AS att_id,
                       att.worked_hours AS att_wh,
                       ot.id AS ot_id,
                       ot.duration AS ot_d,
                       ot.date AS od,
                       att.check_in AS ad
                  FROM hr_attendance att
            INNER JOIN employee_time_zones etz
                    ON att.employee_id = etz.employee_id
            INNER JOIN hr_attendance_overtime ot
                    ON date_trunc('day',
                                  CAST(att.check_in
                                           AT TIME ZONE 'utc'
                                           AT TIME ZONE etz.timezone
                                  as date)) = date_trunc('day', ot.date)
                   AND att.employee_id = ot.employee_id
                   AND att.employee_id IN %s
                   AND ot.adjustment IS false
              ORDER BY att.check_in DESC
            ''', (tuple(self.employee_id.ids),))
            a = self.env.cr.dictfetchall()
            grouped_dict = dict()
            for row in a:
                if row['ot_id'] and row['att_wh']:
                    if row['ot_id'] not in grouped_dict:
                        grouped_dict[row['ot_id']] = {'attendances': [(row['att_id'], row['att_wh'])], 'overtime_duration': row['ot_d']}
                    else:
                        grouped_dict[row['ot_id']]['attendances'].append((row['att_id'], row['att_wh']))

            for overtime in grouped_dict:
                overtime_reservoir = grouped_dict[overtime]['overtime_duration']
                if overtime_reservoir > 0:
                    for attendance in grouped_dict[overtime]['attendances']:
                        if overtime_reservoir > 0:
                            sub_time = attendance[1] - overtime_reservoir
                            if sub_time < 0:
                                att_progress_values[attendance[0]] = 0
                                overtime_reservoir -= attendance[1]
                            else:
                                att_progress_values[attendance[0]] = float(((attendance[1] - overtime_reservoir) / attendance[1]) * 100)
                                overtime_reservoir = 0
                        else:
                            att_progress_values[attendance[0]] = 100
                elif overtime_reservoir < 0 and grouped_dict[overtime]['attendances']:
                    att_id = grouped_dict[overtime]['attendances'][0][0]
                    att_progress_values[att_id] = overtime_reservoir
                    negative_overtime_attendances[att_id] = True
        for attendance in self:
            if negative_overtime_attendances[attendance.id]:
                attendance.overtime_hours = att_progress_values.get(attendance.id, 0)
            else:
                attendance.overtime_hours = attendance.worked_hours * ((100 - att_progress_values.get(attendance.id, 100)) / 100)

    @api.depends('employee_id', 'overtime_status', 'overtime_hours')
    def _compute_validated_overtime_hours(self):
        no_validation = self.filtered(lambda a: a.employee_id.company_id.attendance_overtime_validation == 'no_validation')
        with_validation = self - no_validation

        for attendance in with_validation:
            if attendance.overtime_status == 'to_approve':
                attendance.validated_overtime_hours = attendance.overtime_hours
            elif attendance.overtime_status == 'refused':
                attendance.validated_overtime_hours = 0

        for attendance in no_validation:
            attendance.validated_overtime_hours = attendance.overtime_hours

    @api.depends('validated_overtime_hours')
    def _compute_no_validated_overtime_hours(self):
        for attendance in self:
            attendance.no_validated_overtime_hours = not float_compare(attendance.validated_overtime_hours, 0.0, precision_digits=5)

    @api.depends('employee_id')
    def _compute_overtime_status(self):
        for attendance in self:
            if not attendance.overtime_status:
                attendance.overtime_status = "to_approve" if attendance.employee_id.company_id.attendance_overtime_validation == 'by_manager' else "approved"

    @api.depends('employee_id', 'check_in', 'check_out')
    def _compute_display_name(self):
        tz = request.httprequest.cookies.get('tz') if request else None
        for attendance in self:
            if not attendance.check_out:
                attendance.display_name = _(
                    "From %s",
                    format_time(self.env, attendance.check_in, time_format=None, tz=tz, lang_code=self.env.lang),
                )
            else:
                attendance.display_name = _(
                    "%(worked_hours)s (%(check_in)s-%(check_out)s)",
                    worked_hours=format_duration(attendance.worked_hours),
                    check_in=format_time(self.env, attendance.check_in, time_format=None, tz=tz, lang_code=self.env.lang),
                    check_out=format_time(self.env, attendance.check_out, time_format=None, tz=tz, lang_code=self.env.lang),
                )

    def _get_employee_calendar(self):
        self.ensure_one()
        return self.employee_id.resource_calendar_id or self.employee_id.company_id.resource_calendar_id

    @api.depends('check_in', 'check_out')
    def _compute_worked_hours(self):
        """ Computes the worked hours of the attendance record.
            The worked hours of resource with flexible calendar is computed as the difference
            between check_in and check_out, without taking into account the lunch_interval"""
        for attendance in self:
            if attendance.check_out and attendance.check_in and attendance.employee_id:
                calendar = attendance._get_employee_calendar()
                resource = attendance.employee_id.resource_id
                tz = timezone(resource.tz) if not calendar else timezone(calendar.tz)
                check_in_tz = attendance.check_in.astimezone(tz)
                check_out_tz = attendance.check_out.astimezone(tz)
                lunch_intervals = []
                if not attendance.employee_id.is_flexible:
                    lunch_intervals = attendance.employee_id._employee_attendance_intervals(check_in_tz, check_out_tz, lunch=True)
                attendance_intervals = Intervals([(check_in_tz, check_out_tz, attendance)]) - lunch_intervals
                delta = sum((i[1] - i[0]).total_seconds() for i in attendance_intervals)
                attendance.worked_hours = delta / 3600.0
            else:
                attendance.worked_hours = False

    @api.constrains('check_in', 'check_out')
    def _check_validity_check_in_check_out(self):
        """ verifies if check_in is earlier than check_out. """
        for attendance in self:
            if attendance.check_in and attendance.check_out:
                if attendance.check_out < attendance.check_in:
                    raise exceptions.ValidationError(_('"Check Out" time cannot be earlier than "Check In" time.'))

    @api.constrains('check_in', 'check_out', 'employee_id')
    def _check_validity(self):
        """ Verifies the validity of the attendance record compared to the others from the same employee.
            For the same employee we must have :
                * maximum 1 "open" attendance record (without check_out)
                * no overlapping time slices with previous employee records
        """
        for attendance in self:
            # we take the latest attendance before our check_in time and check it doesn't overlap with ours
            last_attendance_before_check_in = self.env['hr.attendance'].search([
                ('employee_id', '=', attendance.employee_id.id),
                ('check_in', '<=', attendance.check_in),
                ('id', '!=', attendance.id),
            ], order='check_in desc', limit=1)
            if last_attendance_before_check_in and last_attendance_before_check_in.check_out and last_attendance_before_check_in.check_out > attendance.check_in:
                raise exceptions.ValidationError(_("Cannot create new attendance record for %(empl_name)s, the employee was already checked in on %(datetime)s",
                                                   empl_name=attendance.employee_id.name,
                                                   datetime=format_datetime(self.env, attendance.check_in, dt_format=False)))

            if not attendance.check_out:
                # if our attendance is "open" (no check_out), we verify there is no other "open" attendance
                no_check_out_attendances = self.env['hr.attendance'].search([
                    ('employee_id', '=', attendance.employee_id.id),
                    ('check_out', '=', False),
                    ('id', '!=', attendance.id),
                ], order='check_in desc', limit=1)
                if no_check_out_attendances:
                    raise exceptions.ValidationError(_("Cannot create new attendance record for %(empl_name)s, the employee hasn't checked out since %(datetime)s",
                                                       empl_name=attendance.employee_id.name,
                                                       datetime=format_datetime(self.env, no_check_out_attendances.check_in, dt_format=False)))
            else:
                # we verify that the latest attendance with check_in time before our check_out time
                # is the same as the one before our check_in time computed before, otherwise it overlaps
                last_attendance_before_check_out = self.env['hr.attendance'].search([
                    ('employee_id', '=', attendance.employee_id.id),
                    ('check_in', '<', attendance.check_out),
                    ('id', '!=', attendance.id),
                ], order='check_in desc', limit=1)
                if last_attendance_before_check_out and last_attendance_before_check_in != last_attendance_before_check_out:
                    raise exceptions.ValidationError(_("Cannot create new attendance record for %(empl_name)s, the employee was already checked in on %(datetime)s",
                                                       empl_name=attendance.employee_id.name,
                                                       datetime=format_datetime(self.env, last_attendance_before_check_out.check_in, dt_format=False)))

    @api.model
    def _get_day_start_and_day(self, employee, dt):
        #Returns a tuple containing the datetime in naive UTC of the employee's start of the day
        # and the date it was for that employee
        if not dt.tzinfo:
            date_employee_tz = pytz.utc.localize(dt).astimezone(pytz.timezone(employee._get_tz()))
        else:
            date_employee_tz = dt
        start_day_employee_tz = date_employee_tz.replace(hour=0, minute=0, second=0)
        return (start_day_employee_tz.astimezone(pytz.utc).replace(tzinfo=None), start_day_employee_tz.date())

    def _get_attendances_dates(self):
        # Returns a dictionnary {employee_id: set((datetimes, dates))}
        attendances_emp = defaultdict(set)
        for attendance in self.filtered(lambda a: a.check_in):
            check_in_day_start = attendance._get_day_start_and_day(attendance.employee_id, attendance.check_in)
            attendances_emp[attendance.employee_id].add(check_in_day_start)
            if attendance.check_out:
                check_out_day_start = attendance._get_day_start_and_day(attendance.employee_id, attendance.check_out)
                attendances_emp[attendance.employee_id].add(check_out_day_start)
        return attendances_emp

    def _get_overtime_leave_domain(self):
        return []

    def _update_overtime(self, employee_attendance_dates=None):
        if employee_attendance_dates is None:
            employee_attendance_dates = self._get_attendances_dates()

        overtime_to_unlink = self.env['hr.attendance.overtime']
        overtime_vals_list = []
        affected_employees = self.env['hr.employee']
        for emp, attendance_dates in employee_attendance_dates.items():
            # get_attendances_dates returns the date translated from the local timezone without tzinfo,
            # and contains all the date which we need to check for overtime
            attendance_domain = []
            for attendance_date in attendance_dates:
                attendance_domain = OR([attendance_domain, [
                    ('check_in', '>=', attendance_date[0]), ('check_in', '<', attendance_date[0] + timedelta(hours=24)),
                ]])
            attendance_domain = AND([[('employee_id', '=', emp.id)], attendance_domain])

            # Attendances per LOCAL day
            attendances_per_day = defaultdict(lambda: self.env['hr.attendance'])
            all_attendances = self.env['hr.attendance'].search(attendance_domain)
            for attendance in all_attendances:
                check_in_day_start = attendance._get_day_start_and_day(attendance.employee_id, attendance.check_in)
                attendances_per_day[check_in_day_start[1]] += attendance

            # As _attendance_intervals_batch and _leave_intervals_batch both take localized dates we need to localize those date
            start = pytz.utc.localize(min(attendance_dates, key=itemgetter(0))[0])
            stop = pytz.utc.localize(max(attendance_dates, key=itemgetter(0))[0] + timedelta(hours=24))

            # Retrieve expected attendance intervals
            calendar = emp.resource_calendar_id or emp.company_id.resource_calendar_id
            expected_attendances = emp._employee_attendance_intervals(start, stop)

            # working_times = {date: [(start, stop)]}
            working_times = defaultdict(lambda: [])
            for expected_attendance in expected_attendances:
                # Exclude resource.calendar.attendance
                working_times[expected_attendance[0].date()].append(expected_attendance[:2])

            overtimes = self.env['hr.attendance.overtime'].sudo().search([
                ('employee_id', '=', emp.id),
                ('date', 'in', [day_data[1] for day_data in attendance_dates]),
                ('adjustment', '=', False),
            ])

            company_threshold = emp.company_id.overtime_company_threshold / 60.0
            employee_threshold = emp.company_id.overtime_employee_threshold / 60.0

            for day_data in attendance_dates:
                attendance_date = day_data[1]
                attendances = attendances_per_day.get(attendance_date, self.browse())
                unfinished_shifts = attendances.filtered(lambda a: not a.check_out)
                overtime_duration = 0
                overtime_duration_real = 0
                # Overtime is not counted if any shift is not closed or if there are no attendances for that day,
                # this could happen when deleting attendances.
                if not unfinished_shifts and attendances:
                    # The employee is working flexible hours
                    if emp.is_flexible:
                        work_duration = 0
                        for attendance in attendances:
                            local_check_in = pytz.utc.localize(attendance.check_in)
                            local_check_out = pytz.utc.localize(attendance.check_out)
                            work_duration += (local_check_out - local_check_in).total_seconds() / 3600.0
                        # In case of fully flexible employee, no overtime is computed
                        if not emp.is_fully_flexible:
                            overtime_duration = work_duration - emp.resource_id.calendar_id.hours_per_day
                            overtime_duration_real = overtime_duration

                    # The employee usually doesn't work on that day
                    elif not working_times[attendance_date]:
                        # User does not have any resource_calendar_attendance for that day (week-end for example)
                        overtime_duration = sum(attendances.mapped('worked_hours'))
                        overtime_duration_real = overtime_duration
                    # The employee usually work on that day
                    else:
                        # Count time before, during and after 'working hours'
                        pre_work_time, work_duration, post_work_time, planned_work_duration = attendances._get_pre_post_work_time(emp, working_times, attendance_date)
                        # Overtime within the planned work hours + overtime before/after work hours is > company threshold
                        overtime_duration = work_duration - planned_work_duration
                        if pre_work_time > company_threshold:
                            overtime_duration += pre_work_time
                        if post_work_time > company_threshold:
                            overtime_duration += post_work_time
                        # Global overtime including the thresholds
                        overtime_duration_real = sum(attendances.mapped('worked_hours')) - planned_work_duration

                overtime = overtimes.filtered(lambda o: o.date == attendance_date)
                if not float_is_zero(overtime_duration, 2) or unfinished_shifts:
                    # Do not create if any attendance doesn't have a check_out, update if exists
                    if unfinished_shifts:
                        overtime_duration = 0
                    if not overtime and overtime_duration:
                        overtime_vals_list.append({
                            'employee_id': emp.id,
                            'date': attendance_date,
                            'duration': overtime_duration,
                            'duration_real': overtime_duration_real,
                        })
                    elif overtime:
                        overtime.sudo().write({
                            'duration': overtime_duration,
                            'duration_real': overtime_duration
                        })
                        affected_employees |= overtime.employee_id
                elif overtime:
                    overtime_to_unlink |= overtime
        created_overtimes = self.env['hr.attendance.overtime'].sudo().create(overtime_vals_list)
        employees_worked_hours_to_compute = (affected_employees.ids +
                                             created_overtimes.employee_id.ids +
                                             overtime_to_unlink.employee_id.ids)
        overtime_to_unlink.sudo().unlink()
        to_recompute = self.search([('employee_id', 'in', employees_worked_hours_to_compute)])
        self.env.add_to_compute(self._fields['overtime_hours'],
                                to_recompute)
        self.env.add_to_compute(self._fields['validated_overtime_hours'],
                                to_recompute)
        self.env.add_to_compute(self._fields['expected_hours'],
                                to_recompute)

    def _get_pre_post_work_time(self, employee, working_times, attendance_date):
        pre_work_time, work_duration, post_work_time = 0, 0, 0
        company_threshold = employee.company_id.overtime_company_threshold / 60.0
        employee_threshold = employee.company_id.overtime_employee_threshold / 60.0
        # Compute start and end time for that day
        planned_start_dt, planned_end_dt = False, False
        planned_work_duration = 0
        for calendar_attendance in working_times[attendance_date]:
            planned_start_dt = min(planned_start_dt, calendar_attendance[0]) if planned_start_dt else calendar_attendance[0]
            planned_end_dt = max(planned_end_dt, calendar_attendance[1]) if planned_end_dt else calendar_attendance[1]
            planned_work_duration += (calendar_attendance[1] - calendar_attendance[0]).total_seconds() / 3600.0
        for attendance in self:
            # consider check_in as planned_start_dt if within threshold
            # if delta_in < 0: Checked in after supposed start of the day
            # if delta_in > 0: Checked in before supposed start of the day
            local_check_in = pytz.utc.localize(attendance.check_in)
            delta_in = (planned_start_dt - local_check_in).total_seconds() / 3600.0

            # Started before or after planned date within the threshold interval
            if (delta_in > 0 and delta_in <= company_threshold) or\
                (delta_in < 0 and abs(delta_in) <= employee_threshold):
                local_check_in = planned_start_dt
            local_check_out = pytz.utc.localize(attendance.check_out)

            # same for check_out as planned_end_dt
            delta_out = (local_check_out - planned_end_dt).total_seconds() / 3600.0
            # if delta_out < 0: Checked out before supposed start of the day
            # if delta_out > 0: Checked out after supposed start of the day

            # Finised before or after planned date within the threshold interval
            if (delta_out > 0 and delta_out <= company_threshold) or\
                (delta_out < 0 and abs(delta_out) <= employee_threshold):
                local_check_out = planned_end_dt

            # There is an overtime at the start of the day
            if local_check_in < planned_start_dt:
                pre_work_time += (min(planned_start_dt, local_check_out) - local_check_in).total_seconds() / 3600.0
            # Interval inside the working hours -> Considered as working time
            if local_check_in <= planned_end_dt and local_check_out >= planned_start_dt:
                start_dt = max(planned_start_dt, local_check_in)
                stop_dt = min(planned_end_dt, local_check_out)
                work_duration += (stop_dt - start_dt).total_seconds() / 3600.0
                # remove lunch time from work duration
                lunch_intervals = employee._employee_attendance_intervals(start_dt, stop_dt, lunch=True)
                work_duration -= sum((i[1] - i[0]).total_seconds() / 3600.0 for i in lunch_intervals)

            # There is an overtime at the end of the day
            if local_check_out > planned_end_dt:
                post_work_time += (local_check_out - max(planned_end_dt, local_check_in)).total_seconds() / 3600.0
        return pre_work_time, work_duration, post_work_time, planned_work_duration

    @api.model_create_multi
    def create(self, vals_list):
        res = super().create(vals_list)
        res._update_overtime()
        return res

    def write(self, vals):
        if vals.get('employee_id') and \
            vals['employee_id'] not in self.env.user.employee_ids.ids and \
            not self.env.user.has_group('hr_attendance.group_hr_attendance_officer'):
            raise AccessError(_("Do not have access, user cannot edit the attendances that are not his own."))
        attendances_dates = self._get_attendances_dates()
        result = super(HrAttendance, self).write(vals)
        if any(field in vals for field in ['employee_id', 'check_in', 'check_out']):
            # Merge attendance dates before and after write to recompute the
            # overtime if the attendances have been moved to another day
            for emp, dates in self._get_attendances_dates().items():
                attendances_dates[emp] |= dates
            self._update_overtime(attendances_dates)
        return result

    def unlink(self):
        attendances_dates = self._get_attendances_dates()
        res = super().unlink()
        self._update_overtime(attendances_dates)
        return res

    def copy(self, default=None):
        raise exceptions.UserError(_('You cannot duplicate an attendance.'))

    def action_in_attendance_maps(self):
        self.ensure_one()
        return {
            'type': 'ir.actions.act_url',
            'url': get_google_maps_url(self.in_latitude, self.in_longitude),
            'target': 'new'
        }

    def action_out_attendance_maps(self):
        self.ensure_one()
        return {
            'type': 'ir.actions.act_url',
            'url': get_google_maps_url(self.out_latitude, self.out_longitude),
            'target': 'new'
        }

    def get_kiosk_url(self):
        return self.get_base_url() + "/hr_attendance/" + self.env.company.attendance_kiosk_key

    @api.model
    def has_demo_data(self):
        if not self.env.user.has_group("hr_attendance.group_hr_attendance_manager"):
            return True
        # This record only exists if the scenario has been already launched
        demo_tag = self.env.ref('hr_attendance.resource_calendar_std_38h', raise_if_not_found=False)
        return bool(demo_tag) or bool(self.env['ir.module.module'].search_count([('demo', '=', True)]))

    def _load_demo_data(self):
        if self.has_demo_data():
            return
        self.env['hr.employee']._load_scenario()
        # Load employees, schedules, departments and partners
        convert.convert_file(self.env, 'hr_attendance', 'data/scenarios/hr_attendance_scenario.xml', None, mode='init', kind='data')

        employee_sj = self.env.ref('hr.employee_sj')
        employee_mw = self.env.ref('hr.employee_mw')
        employee_eg = self.env.ref('hr.employee_eg')

        # Retrieve employee from xml file
        # Calculate attendances records for the previous month and the current until today
        now = datetime.now()
        previous_month_datetime = (now - relativedelta(months=1))
        date_range = now.day + monthrange(previous_month_datetime.year, previous_month_datetime.month)[1]
        city_coordinates = (50.27, 5.31)
        city_coordinates_exception = (51.01, 2.82)
        city_dict = {
                    'latitude': city_coordinates_exception[0],
                    'longitude': city_coordinates_exception[1],
                    'city': 'Rellemstraat'
                }
        city_exception_dict = {
            'latitude': city_coordinates[0],
            'longitude': city_coordinates[1],
            'city': 'Waillet'
        }
        attendance_values = []
        for i in range(1, date_range):
            check_in_date = now.replace(hour=6, minute=0, second=randint(0, 59)) + timedelta(days=-i, minutes=randint(-2, 3))
            if check_in_date.weekday() not in range(0, 5):
                continue
            check_out_date = now.replace(hour=10, minute=0, second=randint(0, 59)) + timedelta(days=-i, minutes=randint(-2, -1))
            check_in_date_after_lunch = now.replace(hour=11, minute=0, second=randint(0, 59)) + timedelta(days=-i, minutes=randint(-2, -1))
            check_out_date_after_lunch = now.replace(hour=15, minute=0, second=randint(0, 59)) + timedelta(days=-i, minutes=randint(1, 3))

            # employee_eg doesn't work on friday
            eg_data = []
            if check_in_date.weekday() != 4:
                # employee_eg will compensate her work's hours between weeks.
                if check_in_date.isocalendar().week % 2:
                    employee_eg_hours = {
                        'check_in_date': check_in_date + timedelta(hours=1),
                        'check_out_date': check_out_date,
                        'check_in_date_after_lunch': check_in_date_after_lunch,
                        'check_out_date_after_lunch': check_out_date_after_lunch + timedelta(hours=-1),
                    }
                else:
                    employee_eg_hours = {
                        'check_in_date': check_in_date,
                        'check_out_date': check_out_date,
                        'check_in_date_after_lunch': check_in_date_after_lunch,
                        'check_out_date_after_lunch': check_out_date_after_lunch + timedelta(hours=1, minutes=30),
                    }
                eg_data = [{
                    'employee_id': employee_eg.id,
                    'check_in': employee_eg_hours['check_in_date'],
                    'check_out': employee_eg_hours['check_out_date'],
                    'in_mode': "kiosk",
                    'out_mode': "kiosk"
                }, {
                    'employee_id': employee_eg.id,
                    'check_in': employee_eg_hours['check_in_date_after_lunch'],
                    'check_out': employee_eg_hours['check_out_date_after_lunch'],
                    'in_mode': "kiosk",
                    'out_mode': "kiosk",
                }]

            # calculate GPS coordination for employee_mw (systray attendance)
            if randint(1, 10) == 1:
                city_data = city_exception_dict
            else:
                city_data = city_dict
            mw_data = [{
                'employee_id': employee_mw.id,
                'check_in': check_in_date,
                'check_out': check_out_date,
                'in_mode': "systray",
                'out_mode': "systray",
                'in_longitude': city_data['longitude'],
                'out_longitude': city_data['longitude'],
                'in_latitude': city_data['latitude'],
                'out_latitude': city_data['latitude'],
                'in_city': city_data['city'],
                'out_city': city_data['city'],
                'in_ip_address': "127.0.0.1",
                'out_ip_address': "127.0.0.1",
                'in_browser': 'chrome',
                'out_browser': 'chrome'
            }, {
                'employee_id': employee_mw.id,
                'check_in': check_in_date_after_lunch,
                'check_out': check_out_date_after_lunch,
                'in_mode': "systray",
                'out_mode': "systray",
                'in_longitude': city_data['longitude'],
                'out_longitude': city_data['longitude'],
                'in_latitude': city_data['latitude'],
                'out_latitude': city_data['latitude'],
                'in_city': city_data['city'],
                'out_city': city_data['city'],
                'in_ip_address': "127.0.0.1",
                'out_ip_address': "127.0.0.1",
                'in_browser': 'chrome',
                'out_browser': 'chrome'
            }]
            sj_data = [{
                'employee_id': employee_sj.id,
                'check_in': check_in_date + timedelta(minutes=randint(-10, -5)),
                'check_out': check_out_date,
                'in_mode': "manual",
                'out_mode': "manual"
            }, {
                'employee_id': employee_sj.id,
                'check_in': check_in_date_after_lunch,
                'check_out': check_out_date_after_lunch + timedelta(hours=1, minutes=randint(-20, 10)),
                'in_mode': "manual",
                'out_mode': "manual"
            }]
            attendance_values.extend(eg_data + mw_data + sj_data)
        self.env['hr.attendance'].create(attendance_values)
        return {
            'type': 'ir.actions.client',
            'tag': 'reload',
        }

    def action_try_kiosk(self):
        if not self.env.user.has_group("hr_attendance.group_hr_attendance_manager"):
            return {
                    'type': 'ir.actions.client',
                    'tag': 'display_notification',
                    'params': {
                        'message': _("You don't have the rights to execute that action."),
                        'type': 'info',
                    }
            }
        return {
            'type': 'ir.actions.act_url',
            'target': 'self',
            'url': self.env.company.attendance_kiosk_url + '?from_trial_mode=True'
        }

    def _read_group_employee_id(self, resources, domain):
        user_domain = self.env.context.get('user_domain')
        employee_domain = [('company_id', 'in', self.env.context.get('allowed_company_ids', []))]
        if not self.env.user.has_group('hr_attendance.group_hr_attendance_manager'):
            employee_domain = AND([employee_domain, [('attendance_manager_id', '=', self.env.user.id)]])
        if not user_domain:
            return self.env['hr.employee'].search(employee_domain)
        else:
            employee_name_domain = []
            for leaf in user_domain:
                if len(leaf) == 3 and leaf[0] == 'employee_id':
                    employee_name_domain.append([('name', leaf[1], leaf[2])])
            return resources | self.env['hr.employee'].search(AND([OR(employee_name_domain), employee_domain]))

    def action_approve_overtime(self):
        self.write({
            'overtime_status': 'approved'
        })

    def action_refuse_overtime(self):
        self.write({
            'overtime_status': 'refused'
        })

    def _cron_auto_check_out(self):
        to_verify = self.env['hr.attendance'].search(
            [('check_out', '=', False),
             ('employee_id.company_id.auto_check_out', '=', True)]
        )

        if not to_verify:
            return

        previous_duration = self.env['hr.attendance']._read_group(
            domain=[
                ('employee_id', 'in', to_verify.mapped('employee_id').ids),
                ('check_in', '>', (fields.Datetime.now() - relativedelta(days=1)).replace(hour=0, minute=0, second=0)),
                ('check_out', '!=', False)], groupby=['check_in:day', 'employee_id'], aggregates=['worked_hours:sum'])

        mapped_previous_duration = defaultdict(lambda: defaultdict(float))
        for rec in previous_duration:
            mapped_previous_duration[rec[1]][rec[0].date()] += rec[2]

        all_companies = to_verify.employee_id.company_id

        for company in all_companies:
            max_tol = company.auto_check_out_tolerance
            to_verify_company = to_verify.filtered(lambda a: a.employee_id.company_id.id == company.id)

            # Attendances where Last open attendance worked time + previously worked time on that day + tolerance greater than the planned worked hours in his calendar
            to_check_out = to_verify_company.filtered(lambda a: (fields.Datetime.now() - a.check_in).seconds / 3600 + mapped_previous_duration[a.employee_id][a.check_in.date()] - max_tol > (sum(a.employee_id.resource_calendar_id.attendance_ids.filtered(lambda att: att.dayofweek == str(a.check_in.weekday())).mapped('duration_hours'))))
            body = _('This attendance was automatically checked out because the employee exceeded the allowed time for their scheduled work hours.')

            for att in to_check_out:
                delta_duration = max(1, (sum(att.employee_id.resource_calendar_id.attendance_ids.filtered(lambda a: a.dayofweek == str(att.check_in.weekday())).mapped('duration_hours')) + max_tol - mapped_previous_duration[att.employee_id][att.check_in.date()]) * 3600)
                att.write({
                    "check_out": att.check_in + relativedelta(seconds=delta_duration),
                    "out_mode": "auto_check_out"
                })
                att.message_post(body=body)

    def _cron_absence_detection(self):
        """
        Objective is to create technical attendances on absence days to have negative overtime created for that day
        """
        yesterday = datetime.today().replace(hour=0, minute=0, second=0) - relativedelta(days=1)
        companies = self.env['res.company'].search([('absence_management', '=', True)])
        if not companies:
            return

        checked_in_employees = self.env['hr.attendance.overtime'].search([('date', '=', yesterday),
                                                                          ('adjustment', '=', False)]).employee_id

        technical_attendances_vals = []
        absent_employees = self.env['hr.employee'].search([('id', 'not in', checked_in_employees.ids),
                                                           ('company_id', 'in', companies.ids)])
        for emp in absent_employees:
            local_day_start = pytz.utc.localize(yesterday).astimezone(pytz.timezone(emp._get_tz()))
            technical_attendances_vals.append({
                'check_in': local_day_start.strftime('%Y-%m-%d %H:%M:%S'),
                'check_out': (local_day_start + relativedelta(seconds=1)).strftime('%Y-%m-%d %H:%M:%S'),
                'in_mode': 'technical',
                'out_mode': 'technical',
                'employee_id': emp.id
            })

        technical_attendances = self.env['hr.attendance'].create(technical_attendances_vals)
        to_unlink = technical_attendances.filtered(lambda a: a.overtime_hours == 0)

        body = _('This attendance was automatically created to cover an unjustified absence on that day.')
        for technical_attendance in technical_attendances - to_unlink:
            technical_attendance.message_post(body=body)

        to_unlink.unlink()

```

## File: models\hr_attendance_overtime.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields


class HrAttendanceOvertime(models.Model):
    _name = "hr.attendance.overtime"
    _description = "Attendance Overtime"
    _rec_name = 'employee_id'
    _order = 'date desc'

    def _default_employee(self):
        return self.env.user.employee_id

    employee_id = fields.Many2one(
        'hr.employee', string="Employee", default=_default_employee,
        required=True, ondelete='cascade', index=True)
    company_id = fields.Many2one(related='employee_id.company_id')

    date = fields.Date(string='Day')
    duration = fields.Float(string='Extra Hours', default=0.0, required=True)
    duration_real = fields.Float(
        string='Extra Hours (Real)', default=0.0,
        help="Extra-hours including the threshold duration")
    adjustment = fields.Boolean(default=False)

    def init(self):
        # Allows only 1 overtime record per employee per day unless it's an adjustment
        self.env.cr.execute("""
            CREATE UNIQUE INDEX IF NOT EXISTS hr_attendance_overtime_unique_employee_per_day
            ON %s (employee_id, date)
            WHERE adjustment is false""" % (self._table))

```

## File: models\hr_employee.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import pytz
from dateutil.relativedelta import relativedelta

from odoo import models, fields, api, exceptions, _
from odoo.tools import float_round


class HrEmployee(models.Model):
    _inherit = "hr.employee"

    attendance_manager_id = fields.Many2one(
        'res.users', store=True, readonly=False,
        domain="[('share', '=', False), ('company_ids', 'in', company_id)]",
        groups="hr_attendance.group_hr_attendance_manager",
        help="The user set in Attendance will access the attendance of the employee through the dedicated app and will be able to edit them.")
    attendance_ids = fields.One2many(
        'hr.attendance', 'employee_id', groups="hr_attendance.group_hr_attendance_officer,hr.group_hr_user")
    last_attendance_id = fields.Many2one(
        'hr.attendance', compute='_compute_last_attendance_id', store=True,
        groups="hr_attendance.group_hr_attendance_officer,hr.group_hr_user")
    last_check_in = fields.Datetime(
        related='last_attendance_id.check_in', store=True,
        groups="hr_attendance.group_hr_attendance_officer,hr.group_hr_user", tracking=False)
    last_check_out = fields.Datetime(
        related='last_attendance_id.check_out', store=True,
        groups="hr_attendance.group_hr_attendance_officer,hr.group_hr_user", tracking=False)
    attendance_state = fields.Selection(
        string="Attendance Status", compute='_compute_attendance_state',
        selection=[('checked_out', "Checked out"), ('checked_in', "Checked in")],
        groups="hr_attendance.group_hr_attendance_officer,hr.group_hr_user")
    hours_last_month = fields.Float(
        compute='_compute_hours_last_month', groups="hr_attendance.group_hr_attendance_officer,hr.group_hr_user")
    hours_today = fields.Float(
        compute='_compute_hours_today',
        groups="hr_attendance.group_hr_attendance_officer,hr.group_hr_user")
    hours_previously_today = fields.Float(
        compute='_compute_hours_today',
        groups="hr_attendance.group_hr_attendance_officer,hr.group_hr_user")
    last_attendance_worked_hours = fields.Float(
        compute='_compute_hours_today',
        groups="hr_attendance.group_hr_attendance_officer,hr.group_hr_user")
    hours_last_month_display = fields.Char(
        compute='_compute_hours_last_month', groups="hr.group_hr_user")
    overtime_ids = fields.One2many(
        'hr.attendance.overtime', 'employee_id', groups="hr_attendance.group_hr_attendance_officer,hr.group_hr_user")
    total_overtime = fields.Float(
        compute='_compute_total_overtime', compute_sudo=True, groups="hr_attendance.group_hr_attendance_officer,hr.group_hr_user")

    @api.model_create_multi
    def create(self, vals_list):
        officer_group = self.env.ref('hr_attendance.group_hr_attendance_officer', raise_if_not_found=False)
        group_updates = []
        for vals in vals_list:
            if officer_group and vals.get('attendance_manager_id'):
                group_updates.append((4, vals['attendance_manager_id']))
        if group_updates:
            officer_group.sudo().write({'users': group_updates})
        return super().create(vals_list)

    def write(self, values):
        old_officers = self.env['res.users']
        if 'attendance_manager_id' in values:
            old_officers = self.attendance_manager_id
            # Officer was added
            if values['attendance_manager_id']:
                officer = self.env['res.users'].browse(values['attendance_manager_id'])
                officers_group = self.env.ref('hr_attendance.group_hr_attendance_officer', raise_if_not_found=False)
                if officers_group and not officer.has_group('hr_attendance.group_hr_attendance_officer'):
                    officer.sudo().write({'groups_id': [(4, officers_group.id)]})

        res = super(HrEmployee, self).write(values)
        old_officers.sudo()._clean_attendance_officers()

        return res

    @api.depends('overtime_ids.duration', 'attendance_ids', 'attendance_ids.overtime_status')
    def _compute_total_overtime(self):
        mapped_validated_overtimes = dict(self.env['hr.attendance']._read_group(
            domain=[('overtime_status', '=', 'approved')],
            groupby=['employee_id'],
            aggregates=['validated_overtime_hours:sum']
        ))

        mapped_overtime_adjustments = dict(self.env['hr.attendance.overtime']._read_group(
            domain=[('adjustment', '=', True)],
            groupby=['employee_id'],
            aggregates=['duration:sum']
        ))

        for employee in self:
            employee.total_overtime = mapped_validated_overtimes.get(employee, 0) + mapped_overtime_adjustments.get(employee, 0)

    def _compute_hours_last_month(self):
        """
        Compute hours in the current month, if we are the 15th of october, will compute hours from 1 oct to 15 oct
        """
        now = fields.Datetime.now()
        now_utc = pytz.utc.localize(now)
        for employee in self:
            tz = pytz.timezone(employee.tz or 'UTC')
            now_tz = now_utc.astimezone(tz)
            start_tz = now_tz.replace(day=1, hour=0, minute=0, second=0, microsecond=0)
            start_naive = start_tz.astimezone(pytz.utc).replace(tzinfo=None)
            end_tz = now_tz
            end_naive = end_tz.astimezone(pytz.utc).replace(tzinfo=None)

            hours = sum(
                att.worked_hours or 0
                for att in employee.attendance_ids.filtered(
                    lambda att: att.check_in >= start_naive and att.check_out and att.check_out <= end_naive
                )
            )

            employee.hours_last_month = round(hours, 2)
            employee.hours_last_month_display = "%g" % employee.hours_last_month

    def _compute_hours_today(self):
        now = fields.Datetime.now()
        now_utc = pytz.utc.localize(now)
        for employee in self:
            # start of day in the employee's timezone might be the previous day in utc
            tz = pytz.timezone(employee.tz)
            now_tz = now_utc.astimezone(tz)
            start_tz = now_tz + relativedelta(hour=0, minute=0)  # day start in the employee's timezone
            start_naive = start_tz.astimezone(pytz.utc).replace(tzinfo=None)

            attendances = self.env['hr.attendance'].search([
                ('employee_id', 'in', employee.ids),
                ('check_in', '<=', now),
                '|', ('check_out', '>=', start_naive), ('check_out', '=', False),
            ], order='check_in asc')
            hours_previously_today = 0
            worked_hours = 0
            attendance_worked_hours = 0
            for attendance in attendances:
                delta = (attendance.check_out or now) - max(attendance.check_in, start_naive)
                attendance_worked_hours = delta.total_seconds() / 3600.0
                worked_hours += attendance_worked_hours
                hours_previously_today += attendance_worked_hours
            employee.last_attendance_worked_hours = attendance_worked_hours
            hours_previously_today -= attendance_worked_hours
            employee.hours_previously_today = hours_previously_today
            employee.hours_today = worked_hours

    @api.depends('attendance_ids')
    def _compute_last_attendance_id(self):
        for employee in self:
            employee.last_attendance_id = self.env['hr.attendance'].search([
                ('employee_id', 'in', employee.ids),
            ], order="check_in desc", limit=1)

    @api.depends('last_attendance_id.check_in', 'last_attendance_id.check_out', 'last_attendance_id')
    def _compute_attendance_state(self):
        for employee in self:
            att = employee.last_attendance_id.sudo()
            employee.attendance_state = att and not att.check_out and 'checked_in' or 'checked_out'

    def _attendance_action_change(self, geo_information=None):
        """ Check In/Check Out action
            Check In: create a new attendance record
            Check Out: modify check_out field of appropriate attendance record
        """
        self.ensure_one()
        action_date = fields.Datetime.now()

        if self.attendance_state != 'checked_in':
            if geo_information:
                vals = {
                    'employee_id': self.id,
                    'check_in': action_date,
                    **{'in_%s' % key: geo_information[key] for key in geo_information}
                }
            else:
                vals = {
                    'employee_id': self.id,
                    'check_in': action_date,
                }
            return self.env['hr.attendance'].create(vals)
        attendance = self.env['hr.attendance'].search([('employee_id', '=', self.id), ('check_out', '=', False)], limit=1)
        if attendance:
            if geo_information:
                attendance.write({
                    'check_out': action_date,
                    **{'out_%s' % key: geo_information[key] for key in geo_information}
                })
            else:
                attendance.write({
                    'check_out': action_date
                })
        else:
            raise exceptions.UserError(_(
                'Cannot perform check out on %(empl_name)s, could not find corresponding check in. '
                'Your attendances have probably been modified manually by human resources.',
                empl_name=self.sudo().name))
        return attendance

    def action_open_last_month_attendances(self):
        self.ensure_one()
        return {
            "type": "ir.actions.act_window",
            "name": _("Attendances This Month"),
            "res_model": "hr.attendance",
            "views": [[self.env.ref('hr_attendance.hr_attendance_employee_simple_tree_view').id, "list"]],
            "context": {
                "create": 0
            },
            "domain": [('employee_id', '=', self.id),
                       ('check_in', ">=", fields.datetime.today().replace(day=1, hour=0, minute=0))]
        }

    def action_open_last_month_overtime(self):
        self.ensure_one()
        return {
            "type": "ir.actions.act_window",
            "name": _("Attendances This Month"),
            "res_model": "hr.attendance",
            "views": [[self.env.ref('hr_attendance.hr_attendance_validated_hours_employee_simple_tree_view').id, "list"]],
            "context": {
                "create": 0
            },
            "domain": [('employee_id', '=', self.id), ('overtime_status', '=', 'approved')]
        }

```

## File: models\hr_employee_base.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class HrEmployeeBase(models.AbstractModel):
    _inherit = "hr.employee.base"

    @api.depends("user_id.im_status", "attendance_state")
    def _compute_presence_state(self):
        """
        Override to include checkin/checkout in the presence state
        Attendance has the second highest priority after login
        """
        super()._compute_presence_state()
        employees = self.filtered(lambda e: e.hr_presence_state != "present")
        employee_to_check_working = self.filtered(lambda e: e.attendance_state == "checked_out"
                                                            and e.hr_presence_state == "out_of_working_hour")
        working_now_list = employee_to_check_working._get_employee_working_now()
        for employee in employees:
            if employee.attendance_state == "checked_out" and employee.hr_presence_state == "out_of_working_hour" and \
                    employee.id in working_now_list:
                employee.hr_presence_state = "absent"
            elif employee.attendance_state == "checked_in":
                employee.hr_presence_state = "present"

    def _compute_presence_icon(self):
        res = super()._compute_presence_icon()
        # All employee must chek in or check out. Everybody must have an icon
        for employee in self:
            employee.show_hr_icon_display = employee.company_id.hr_presence_control_attendance or bool(employee.user_id)
        return res

```

## File: models\hr_employee_public.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models

class HrEmployeePublic(models.Model):
    _inherit = 'hr.employee.public'

    # These are required for manual attendance
    attendance_state = fields.Selection(related='employee_id.attendance_state', readonly=True,
        groups="hr_attendance.group_hr_attendance_officer")
    hours_today = fields.Float(related='employee_id.hours_today', readonly=True,
        groups="hr_attendance.group_hr_attendance_officer")
    last_attendance_id = fields.Many2one(related='employee_id.last_attendance_id', readonly=True,
        groups="hr_attendance.group_hr_attendance_officer")
    total_overtime = fields.Float(related='employee_id.total_overtime', readonly=True,
        groups="hr_attendance.group_hr_attendance_officer")
    attendance_manager_id = fields.Many2one(related='employee_id.attendance_manager_id',
        groups="hr_attendance.group_hr_attendance_officer")
    last_check_in = fields.Datetime(related='employee_id.last_check_in',
        groups="hr_attendance.group_hr_attendance_officer")
    last_check_out = fields.Datetime(related='employee_id.last_check_out',
        groups="hr_attendance.group_hr_attendance_officer")

```

## File: models\res_company.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api
from odoo.osv.expression import OR

import uuid
from werkzeug.urls import url_join


class ResCompany(models.Model):
    _inherit = 'res.company'

    def _default_company_token(self):
        return str(uuid.uuid4())

    overtime_company_threshold = fields.Integer(string="Tolerance Time In Favor Of Company", default=0)
    overtime_employee_threshold = fields.Integer(string="Tolerance Time In Favor Of Employee", default=0)
    hr_attendance_display_overtime = fields.Boolean(string="Display Extra Hours")
    attendance_kiosk_mode = fields.Selection([
        ('barcode', 'Barcode / RFID'),
        ('barcode_manual', 'Barcode / RFID and Manual Selection'),
        ('manual', 'Manual Selection'),
    ], string='Attendance Mode', default='barcode_manual')
    attendance_barcode_source = fields.Selection([
        ('scanner', 'Scanner'),
        ('front', 'Front Camera'),
        ('back', 'Back Camera'),
    ], string='Barcode Source', default='front')
    attendance_kiosk_delay = fields.Integer(default=10)
    attendance_kiosk_key = fields.Char(default=lambda s: uuid.uuid4().hex, copy=False, groups='hr_attendance.group_hr_attendance_manager')
    attendance_kiosk_url = fields.Char(compute="_compute_attendance_kiosk_url")
    attendance_kiosk_use_pin = fields.Boolean(string='Employee PIN Identification')
    attendance_from_systray = fields.Boolean(string='Attendance From Systray', default=True)
    attendance_overtime_validation = fields.Selection([
        ('no_validation', 'Automatically Approved'),
        ('by_manager', 'Approved by Manager'),
    ], string='Extra Hours Validation', default='no_validation')
    auto_check_out = fields.Boolean(string="Automatic Check Out", default=False)
    auto_check_out_tolerance = fields.Float(default=2, export_string_translation=False)
    absence_management = fields.Boolean(string="Absence Management", default=False)

    @api.depends("attendance_kiosk_key")
    def _compute_attendance_kiosk_url(self):
        for company in self:
            company.attendance_kiosk_url = url_join(self.env['res.company'].get_base_url(), '/hr_attendance/%s' % company.attendance_kiosk_key)

    # ---------------------------------------------------------
    # ORM Overrides
    # ---------------------------------------------------------
    def _init_column(self, column_name):
        """ Initialize the value of the given column for existing rows.
            Overridden here because we need to generate different access tokens
            and by default _init_column calls the default method once and applies
            it for every record.
        """
        if column_name != 'attendance_kiosk_key':
            super(ResCompany, self)._init_column(column_name)
        else:
            self.env.cr.execute("SELECT id FROM %s WHERE attendance_kiosk_key IS NULL" % self._table)
            attendance_ids = self.env.cr.dictfetchall()
            values_args = [(attendance_id['id'], self._default_company_token()) for attendance_id in attendance_ids]
            query = """
                UPDATE {table}
                SET attendance_kiosk_key = vals.token
                FROM (VALUES %s) AS vals(id, token)
                WHERE {table}.id = vals.id
            """.format(table=self._table)
            self.env.cr.execute_values(query, values_args)

    def write(self, vals):
        search_domains = []  # Overtime to generate
        # Also recompute if the threshold have changed
        if 'overtime_company_threshold' in vals or 'overtime_employee_threshold' in vals:
            for company in self:
                # If we modify the thresholds only
                if (vals.get('overtime_company_threshold') != company.overtime_company_threshold) or\
                    (vals.get('overtime_employee_threshold') != company.overtime_employee_threshold):
                    search_domains.append([('employee_id.company_id', '=', company.id)])

        res = super().write(vals)
        if search_domains:
            self.env['hr.attendance'].search(OR(search_domains))._update_overtime()

        return res

    def _regenerate_attendance_kiosk_key(self):
        self.ensure_one()
        self.write({
            'attendance_kiosk_key': uuid.uuid4().hex
        })

    def _check_hr_presence_control(self, at_install):
        companies = self.env.companies
        for company in companies:
            if at_install and company.hr_presence_control_login:
                company.hr_presence_control_attendance = True
            if not at_install and company.hr_presence_control_attendance:
                company.hr_presence_control_login = True

    def _action_open_kiosk_mode(self):
        return {
            'type': 'ir.actions.act_url',
            'target': 'self',
            'url': f'/hr_attendance/kiosk_mode_menu/{self.env.company.id}',
        }

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    overtime_company_threshold = fields.Integer(
        string="Tolerance Time In Favor Of Company", readonly=False)
    overtime_employee_threshold = fields.Integer(
        string="Tolerance Time In Favor Of Employee", readonly=False)
    hr_attendance_display_overtime = fields.Boolean(related='company_id.hr_attendance_display_overtime', readonly=False)
    attendance_kiosk_mode = fields.Selection(related='company_id.attendance_kiosk_mode', readonly=False)
    attendance_barcode_source = fields.Selection(related='company_id.attendance_barcode_source', readonly=False)
    attendance_kiosk_delay = fields.Integer(related='company_id.attendance_kiosk_delay', readonly=False)
    attendance_kiosk_url = fields.Char(related='company_id.attendance_kiosk_url')
    attendance_kiosk_use_pin = fields.Boolean(related='company_id.attendance_kiosk_use_pin', readonly=False)
    attendance_from_systray = fields.Boolean(related="company_id.attendance_from_systray", readonly=False)
    attendance_overtime_validation = fields.Selection(related="company_id.attendance_overtime_validation", readonly=False)
    auto_check_out = fields.Boolean(related="company_id.auto_check_out", readonly=False)
    auto_check_out_tolerance = fields.Float(related="company_id.auto_check_out_tolerance", readonly=False)
    absence_management = fields.Boolean(related="company_id.absence_management", readonly=False)

    @api.model
    def get_values(self):
        res = super(ResConfigSettings, self).get_values()
        company = self.env.company
        res.update({
            'overtime_company_threshold': company.overtime_company_threshold,
            'overtime_employee_threshold': company.overtime_employee_threshold,
        })
        return res

    def set_values(self):
        super().set_values()
        company = self.env.company
        # Done this way to have all the values written at the same time,
        # to avoid recomputing the overtimes several times with
        # invalid company configurations
        fields_to_check = [
            'overtime_company_threshold',
            'overtime_employee_threshold',
        ]
        if any(self[field] != company[field] for field in fields_to_check):
            company.write({field: self[field] for field in fields_to_check})

    def regenerate_kiosk_key(self):
        if self.env.user.has_group("hr_attendance.group_hr_attendance_manager"):
            self.company_id._regenerate_attendance_kiosk_key()

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields, _

class User(models.Model):
    _inherit = ['res.users']

    hours_last_month = fields.Float(related='employee_id.hours_last_month')
    hours_last_month_display = fields.Char(related='employee_id.hours_last_month_display')
    attendance_state = fields.Selection(related='employee_id.attendance_state')
    last_check_in = fields.Datetime(related='employee_id.last_attendance_id.check_in')
    last_check_out = fields.Datetime(related='employee_id.last_attendance_id.check_out')
    total_overtime = fields.Float(related='employee_id.total_overtime')
    attendance_manager_id = fields.Many2one(related='employee_id.attendance_manager_id', readonly=False)
    display_extra_hours = fields.Boolean(related='company_id.hr_attendance_display_overtime')

    @property
    def SELF_READABLE_FIELDS(self):
        return super().SELF_READABLE_FIELDS + [
            'hours_last_month',
            'hours_last_month_display',
            'attendance_state',
            'last_check_in',
            'last_check_out',
            'total_overtime',
            'attendance_manager_id',
            'display_extra_hours',
        ]

    def _clean_attendance_officers(self):
        attendance_officers = self.env['hr.employee'].search(
            [('attendance_manager_id', 'in', self.ids)]).attendance_manager_id
        officers_to_remove_ids = self - attendance_officers
        if officers_to_remove_ids:
            self.env.ref('hr_attendance.group_hr_attendance_officer').users = [(3, user.id) for user in
                                                                               officers_to_remove_ids]
    def action_open_last_month_attendances(self):
        self.ensure_one()
        return {
            "type": "ir.actions.act_window",
            "name": _("Attendances This Month"),
            "res_model": "hr.attendance",
            "views": [[self.env.ref('hr_attendance.hr_attendance_employee_simple_tree_view').id, "list"]],
            "context": {
                "create": 0
            },
            "domain": [('employee_id', '=', self.employee_id.id),
                       ('check_in', ">=", fields.datetime.today().replace(day=1, hour=0, minute=0))]
        }

    def action_open_last_month_overtime(self):
        self.ensure_one()
        return {
            "type": "ir.actions.act_window",
            "name": _("Overtime"),
            "res_model": "hr.attendance.overtime",
            "views": [[False, "list"]],
            "context": {
                "create": 0
            },
            "domain": [('employee_id', '=', self.employee_id.id)]
        }

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import res_config_settings
from . import hr_attendance
from . import hr_attendance_overtime
from . import hr_employee_base
from . import hr_employee
from . import hr_employee_public
from . import res_company
from . import res_users

```

## File: security\hr_attendance_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record model="ir.module.category" id="base.module_category_human_resources_attendances">
        <field name="sequence">14</field>
    </record>

    <record id="group_hr_attendance_own_reader" model="res.groups">
        <field name="name">User: Read his own attendances</field>
        <field name="category_id" ref="base.module_category_hidden"/>
        <field name="comment">The user will have access to his own attendances on his user / employee profile</field>
    </record>

    <record id="base.group_user" model="res.groups">
        <field name="implied_ids" eval="[(4, ref('hr_attendance.group_hr_attendance_own_reader'))]"/>
    </record>

    <record id="group_hr_attendance_officer" model="res.groups">
        <field name="name">Officer: Manage attendances</field>
        <field name="category_id" ref="base.module_category_hidden"/>
        <field name="comment">The user will have access to the attendance records and reporting of employees where he's set as an attendance manager</field>
    </record>

    <record id="group_hr_attendance_manager" model="res.groups">
        <field name="name">Administrator</field>
        <field name="category_id" ref="base.module_category_human_resources_attendances"/>
        <field name="users" eval="[(4, ref('base.user_root')), (4, ref('base.user_admin'))]"/>
        <field name="implied_ids" eval="[(4, ref('hr_attendance.group_hr_attendance_officer'))]"/>
    </record>

    <record id="base.default_user" model="res.users">
        <field name="groups_id" eval="[(4, ref('hr_attendance.group_hr_attendance_manager'))]"/>
    </record>

    <data noupdate="1">

        <!-- Attendances -->
        <record id="hr_attendance_rule_employee_company" model="ir.rule">
            <field name="name">Employee multi company rule</field>
            <field name="model_id" ref="model_hr_attendance"/>
            <field name="global" eval="True"/>
            <field name="domain_force">['|',('employee_id.company_id','=',False),('employee_id.company_id', 'in', company_ids)]</field>
        </record>

        <record id="hr_attendance_rule_attendance_admin" model="ir.rule">
            <field name="name">Attendance Administrator: Full access</field>
            <field name="model_id" ref="model_hr_attendance"/>
            <field name="domain_force">[(1,'=',1)]</field>
            <field name="groups" eval="[(4, ref('hr_attendance.group_hr_attendance_manager'))]"/>
        </record>

        <record id="hr_attendance_rule_attendance_manager_restrict" model="ir.rule">
            <field name="name">Attendance Officer: Restrict Attendances to managed employees</field>
            <field name="model_id" ref="model_hr_attendance"/>
            <field name="domain_force">
                [
                '|',
                '&amp;',
                 ('employee_id.attendance_manager_id', '=', user.id),
                 ('employee_id.user_id', '=', user.id),
                '&amp;',
                ('employee_id.user_id', '!=', user.id),
                ('employee_id.attendance_manager_id', '=', user.id)
                ]
            </field>
            <field name="groups" eval="[(4, ref('hr_attendance.group_hr_attendance_officer'))]"/>
            <field name="perm_create" eval="1"/>
            <field name="perm_write" eval="1"/>
            <field name="perm_unlink" eval="1"/>
            <field name="perm_read" eval="1"/>
        </record>

        <record id="hr_attendance_rule_attendance_simple_user" model="ir.rule">
            <field name="name">Attendance base user: Read his own attendances in other apps</field>
            <field name="model_id" ref="model_hr_attendance"/>
            <field name="domain_force">[('employee_id.user_id', '=', user.id)]</field>
            <field name="groups" eval="[(4, ref('hr_attendance.group_hr_attendance_own_reader'))]"/>
            <field name="perm_create" eval="0"/>
            <field name="perm_write" eval="0"/>
            <field name="perm_unlink" eval="0"/>
            <field name="perm_read" eval="1"/>
        </record>

        <!-- Overtime -->
        <record id="hr_attendance_overtime_rule_employee_company" model="ir.rule">
            <field name="name">Employee multi company rule</field>
            <field name="model_id" ref="model_hr_attendance_overtime"/>
            <field name="global" eval="True"/>
            <field name="domain_force">['|',('employee_id.company_id','=',False),('employee_id.company_id', 'in', company_ids)]</field>
        </record>

        <record id="hr_attendance_rule_attendance_officer_overtime_restrict" model="ir.rule">
            <field name="name">Attendance Officer: Restrict Overtime to managed employees</field>
            <field name="model_id" ref="model_hr_attendance_overtime"/>
            <field name="domain_force">
                [
                '|',
                '&amp;',
                 ('employee_id.attendance_manager_id', '=', user.id),
                 ('employee_id.user_id', '=', user.id),
                '&amp;',
                ('employee_id.user_id', '!=', user.id),
                ('employee_id.attendance_manager_id', '=', user.id)
                ]
            </field>
            <field name="groups" eval="[(4, ref('hr_attendance.group_hr_attendance_officer'))]"/>
            <field name="perm_create" eval="1"/>
            <field name="perm_write" eval="1"/>
            <field name="perm_unlink" eval="1"/>
            <field name="perm_read" eval="1"/>
        </record>

        <record id="hr_attendance_rule_attendance_overtime_admin" model="ir.rule">
            <field name="name">Attendance Admin: full access</field>
            <field name="model_id" ref="model_hr_attendance_overtime"/>
            <field name="domain_force">[(1,'=',1)]</field>
            <field name="groups" eval="[(4, ref('hr_attendance.group_hr_attendance_manager'))]"/>

        </record>

        <record id="hr_attendance_rule_attendance_overtime_simple_user" model="ir.rule">
            <field name="name">Attendance base user: Read his own overtime</field>
            <field name="model_id" ref="model_hr_attendance_overtime"/>
            <field name="domain_force">[('employee_id.user_id', '=', user.id)]</field>
            <field name="groups" eval="[(4, ref('hr_attendance.group_hr_attendance_own_reader'))]"/>
            <field name="perm_create" eval="0"/>
            <field name="perm_write" eval="0"/>
            <field name="perm_unlink" eval="0"/>
            <field name="perm_read" eval="1"/>
        </record>
    </data>
</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_hr_attendance_admin,hr.attendance.admin,model_hr_attendance,group_hr_attendance_manager,1,1,1,1
access_hr_attendance_admin_overtime,hr.attendance.admin.overtime,model_hr_attendance_overtime,group_hr_attendance_manager,1,1,1,1
access_hr_attendance_officer,hr.attendance.officer,model_hr_attendance,group_hr_attendance_officer,1,1,1,1
access_hr_attendance_officer_overtime,hr.attendance.officer.overtime,model_hr_attendance_overtime,group_hr_attendance_officer,1,1,1,1
access_hr_attendance_user,hr.attendance.user,model_hr_attendance,group_hr_attendance_own_reader,1,0,0,0
access_hr_attendance_overtime_user,hr.attendance.overtime.user,model_hr_attendance_overtime,group_hr_attendance_own_reader,1,0,0,0

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><circle cx="18" cy="16" r="10" fill="#FBB945"/><path d="M10.215 9.722c.45-.557.96-1.066 1.518-1.515l7.479 5.999c.985.79 1.053 2.274.16 3.168-.892.893-2.375.825-3.164-.161l-5.993-7.49Z" fill="#fff"/><circle cx="38" cy="24" r="5" fill="#953B24"/><path d="M16 31h26a4 4 0 0 1 4 4v5a4 4 0 0 1-4 4H16V31Z" fill="#953B24"/><path d="M4 31h16c7.18 0 13 5.82 13 13H17C9.82 44 4 38.18 4 31Z" fill="#FBB945"/></svg>

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

## File: static\img\tablet-cam.svg

```svg
<svg width="232" height="137" viewBox="0 0 232 137" fill="none" xmlns="http://www.w3.org/2000/svg">
<path d="M10 2.5H222C226.142 2.5 229.5 5.85786 229.5 10V127C229.5 131.142 226.142 134.5 222 134.5H10C5.85787 134.5 2.5 131.142 2.5 127V10C2.5 5.85786 5.85786 2.5 10 2.5Z" fill="#714B67" fill-opacity="0.75" stroke="#714B67" stroke-width="5"/>
<mask id="path-3-inside-1_11_84" fill="white">
<rect x="129" width="17.7895" height="26" rx="4" transform="rotate(90 129 0)"/>
</mask>
<rect x="129" width="17.7895" height="26" rx="4" transform="rotate(90 129 0)" fill="#303030" stroke="#373737" stroke-width="12" mask="url(#path-3-inside-1_11_84)"/>
<circle cx="118.053" cy="9.12278" r="2.10526" transform="rotate(90 118.053 9.12278)" fill="#F58787" stroke="#232323" stroke-width="4"/>
<circle cx="118.053" cy="9.12282" r="1.36842" transform="rotate(90 118.053 9.12282)" fill="#454545"/>
<rect x="110.754" y="7.29822" width="3.64912" height="0.912281" rx="0.45614" transform="rotate(90 110.754 7.29822)" fill="#57E888"/>
<rect x="50" y="31" width="132" height="75" rx="7" fill="white"/>
<path d="M96.9254 83H96V99H96.9254V83Z" fill="black"/>
<path d="M97.8507 83H97.3881V99H97.8507V83Z" fill="black"/>
<path d="M100.164 83H98.7761V99H100.164V83Z" fill="black"/>
<path d="M102.015 83H101.09V99H102.015V83Z" fill="black"/>
<path d="M103.866 83H103.403V99H103.866V83Z" fill="black"/>
<path d="M105.254 83H104.791V99H105.254V83Z" fill="black"/>
<path d="M107.104 83H106.179V99H107.104V83Z" fill="black"/>
<path d="M108.955 83H108.493V99H108.955V83Z" fill="black"/>
<path d="M110.343 83H109.881V99H110.343V83Z" fill="black"/>
<path d="M112.194 83H111.269V99H112.194V83Z" fill="black"/>
<path d="M114.045 83H113.582V99H114.045V83Z" fill="black"/>
<path d="M115.433 83H114.97V99H115.433V83Z" fill="black"/>
<path d="M117.284 83H116.358V99H117.284V83Z" fill="black"/>
<path d="M119.134 83H118.672V99H119.134V83Z" fill="black"/>
<path d="M120.522 83H120.06V99H120.522V83Z" fill="black"/>
<path d="M122.373 83H121.448V99H122.373V83Z" fill="black"/>
<path d="M124.687 83H123.299V99H124.687V83Z" fill="black"/>
<path d="M125.612 83H125.149V99H125.612V83Z" fill="black"/>
<path d="M127.463 83H126.537V99H127.463V83Z" fill="black"/>
<path d="M129.313 83H127.925V99H129.313V83Z" fill="black"/>
<path d="M130.701 83H130.239V99H130.701V83Z" fill="black"/>
<path d="M132.552 83H131.627V99H132.552V83Z" fill="black"/>
<path d="M134.403 83H133.94V99H134.403V83Z" fill="black"/>
<path d="M135.791 83H135.328V99H135.791V83Z" fill="black"/>
<path d="M137.642 83H136.716V99H137.642V83Z" fill="black"/>
<path d="M139.493 83H139.03V99H139.493V83Z" fill="black"/>
<path d="M140.881 83H140.418V99H140.881V83Z" fill="black"/>
<path d="M142.731 83H141.806V99H142.731V83Z" fill="black"/>
<path d="M144.582 83H144.119V99H144.582V83Z" fill="black"/>
<path d="M145.97 83H145.507V99H145.97V83Z" fill="black"/>
<path d="M148.746 83H146.896V99H148.746V83Z" fill="black"/>
<path d="M149.672 83H149.209V99H149.672V83Z" fill="black"/>
<path d="M150.597 83H150.134V99H150.597V83Z" fill="black"/>
<path d="M152.91 83H151.985V99H152.91V83Z" fill="black"/>
<path d="M155.687 83H154.299V99H155.687V83Z" fill="black"/>
<path d="M156.612 83H156.149V99H156.612V83Z" fill="black"/>
<path d="M158 83H157.075V99H158V83Z" fill="black"/>
<path d="M50 38C50 34.134 53.134 31 57 31H88V106H57C53.134 106 50 102.866 50 99V38Z" fill="#714B67"/>
<rect x="96" y="41" width="41" height="10" rx="5" fill="#714B67" fill-opacity="0.6"/>
<rect x="96" y="60" width="20" height="5" rx="2.5" fill="#714B67" fill-opacity="0.2"/>
<rect x="118" y="60" width="32" height="5" rx="2.5" fill="#714B67" fill-opacity="0.2"/>
<rect x="96" y="69" width="25" height="5" rx="2.5" fill="#714B67" fill-opacity="0.2"/>
<rect x="126" y="69" width="16" height="5" rx="2.5" fill="#714B67" fill-opacity="0.2"/>
<rect x="147" y="69" width="25" height="5" rx="2.5" fill="#714B67" fill-opacity="0.2"/>
<path d="M69 40C70.4896 40 71.9137 40.2906 73.2723 40.8717C74.631 41.4528 75.8013 42.2344 76.7835 43.2165C77.7656 44.1987 78.5472 45.369 79.1283 46.7277C79.7094 48.0863 80 49.5104 80 51C80 52.4814 79.7115 53.9014 79.1345 55.26C78.5575 56.6187 77.7779 57.7891 76.7958 58.7712C75.8136 59.7533 74.6432 60.537 73.2846 61.1222C71.926 61.7074 70.4978 62 69 62C67.5022 62 66.074 61.7094 64.7154 61.1283C63.3568 60.5472 62.1884 59.7636 61.2104 58.7773C60.2323 57.7911 59.4528 56.6207 58.8717 55.2662C58.2906 53.9116 58 52.4896 58 51C58 49.5104 58.2906 48.0863 58.8717 46.7277C59.4528 45.369 60.2344 44.1987 61.2165 43.2165C62.1987 42.2344 63.369 41.4528 64.7277 40.8717C66.0863 40.2906 67.5104 40 69 40ZM76.5993 56.5859C77.8188 54.9081 78.4286 53.0461 78.4286 51C78.4286 49.7232 78.1789 48.5037 77.6797 47.3415C77.1804 46.1793 76.5093 45.1767 75.6663 44.3337C74.8233 43.4907 73.8207 42.8196 72.6585 42.3203C71.4963 41.8211 70.2768 41.5714 69 41.5714C67.7232 41.5714 66.5037 41.8211 65.3415 42.3203C64.1793 42.8196 63.1767 43.4907 62.3337 44.3337C61.4907 45.1767 60.8196 46.1793 60.3203 47.3415C59.8211 48.5037 59.5714 49.7232 59.5714 51C59.5714 53.0461 60.1812 54.9081 61.4007 56.5859C61.9408 53.9096 63.1931 52.5714 65.1574 52.5714C66.2295 53.619 67.5104 54.1429 69 54.1429C70.4896 54.1429 71.7705 53.619 72.8426 52.5714C74.8069 52.5714 76.0592 53.9096 76.5993 56.5859ZM73.7143 48.6429C73.7143 47.3415 73.2539 46.2305 72.3331 45.3097C71.4124 44.389 70.3013 43.9286 69 43.9286C67.6987 43.9286 66.5876 44.389 65.6669 45.3097C64.7461 46.2305 64.2857 47.3415 64.2857 48.6429C64.2857 49.9442 64.7461 51.0552 65.6669 51.976C66.5876 52.8968 67.6987 53.3571 69 53.3571C70.3013 53.3571 71.4124 52.8968 72.3331 51.976C73.2539 51.0552 73.7143 49.9442 73.7143 48.6429Z" fill="white"/>
</svg>

```

## File: static\img\tablet-pin.svg

```svg
<svg width="232" height="137" viewBox="0 0 232 137" fill="none" xmlns="http://www.w3.org/2000/svg">
<path d="M10 2.5H222C226.142 2.5 229.5 5.85786 229.5 10V127C229.5 131.142 226.142 134.5 222 134.5H10C5.85787 134.5 2.5 131.142 2.5 127V10C2.5 5.85786 5.85786 2.5 10 2.5Z" fill="#714B67" fill-opacity="0.75" stroke="#714B67" stroke-width="5"/>
<g clip-path="url(#clip0_11_116)">
<rect x="14" y="51" width="204" height="35" rx="4" fill="white"/>
<rect x="24" y="55.5" width="26" height="26" rx="4" fill="#D9CFD7"/>
<rect x="60" y="58.5" width="43" height="7" rx="3.5" fill="#714B67" fill-opacity="0.2"/>
<rect x="107" y="58.5" width="88" height="7" rx="3.5" fill="#714B67" fill-opacity="0.2"/>
<rect x="60" y="71.5" width="52" height="7" rx="3.5" fill="#714B67" fill-opacity="0.2"/>
<rect x="122" y="71.5" width="35" height="7" rx="3.5" fill="#714B67" fill-opacity="0.2"/>
</g>
<g clip-path="url(#clip1_11_116)">
<rect x="14" y="90" width="204" height="35" rx="4" fill="white"/>
<rect x="24" y="94.5" width="26" height="26" rx="4" fill="#714B67" fill-opacity="0.75"/>
<rect x="60" y="97.5" width="43" height="7" rx="3.5" fill="#714B67" fill-opacity="0.2"/>
<rect x="107" y="97.5" width="88" height="7" rx="3.5" fill="#714B67" fill-opacity="0.2"/>
<rect x="60" y="110.5" width="52" height="7" rx="3.5" fill="#714B67" fill-opacity="0.2"/>
<rect x="122" y="110.5" width="35" height="7" rx="3.5" fill="#714B67" fill-opacity="0.2"/>
</g>
<g clip-path="url(#clip2_11_116)">
<rect x="14" y="12" width="204" height="35" rx="4" fill="white"/>
<rect x="24" y="16.5" width="26" height="26" rx="4" fill="#714B67" fill-opacity="0.6"/>
<rect x="60" y="19.5" width="43" height="7" rx="3.5" fill="#714B67" fill-opacity="0.2"/>
<rect x="107" y="19.5" width="88" height="7" rx="3.5" fill="#714B67" fill-opacity="0.2"/>
<rect x="60" y="32.5" width="52" height="7" rx="3.5" fill="#714B67" fill-opacity="0.2"/>
<rect x="122" y="32.5" width="35" height="7" rx="3.5" fill="#714B67" fill-opacity="0.2"/>
</g>
<defs>
<clipPath id="clip0_11_116">
<rect x="14" y="51" width="204" height="35" rx="4" fill="white"/>
</clipPath>
<clipPath id="clip1_11_116">
<rect x="14" y="90" width="204" height="35" rx="4" fill="white"/>
</clipPath>
<clipPath id="clip2_11_116">
<rect x="14" y="12" width="204" height="35" rx="4" fill="white"/>
</clipPath>
</defs>
</svg>

```

## File: static\img\tablet-rfid.svg

```svg
<svg width="232" height="137" viewBox="0 0 232 137" fill="none" xmlns="http://www.w3.org/2000/svg">
<path d="M10 2.5H222C226.142 2.5 229.5 5.85786 229.5 10V127C229.5 131.142 226.142 134.5 222 134.5H10C5.85787 134.5 2.5 131.142 2.5 127V10C2.5 5.85786 5.85786 2.5 10 2.5Z" fill="#714B67" fill-opacity="0.75" stroke="#714B67" stroke-width="5"/>
<path d="M98.7909 104.627V110.134H94V95H99.8098C104.629 95 107.039 96.4907 107.039 99.4717C107.039 101.225 106.036 102.581 104.031 103.54L109.198 110.134H103.764L100.004 104.627H98.7909ZM98.7909 101.552H99.6885C101.362 101.552 102.199 100.921 102.199 99.6582C102.199 98.6159 101.378 98.0951 99.737 98.0951H98.7909V101.552Z" fill="white"/>
<path d="M115.529 110.134H110.811V95H121.29V98.2813H115.529V101.169H120.841V104.451H115.529V110.134Z" fill="white"/>
<path d="M124.201 110.134V95H129.016V110.134H124.201Z" fill="white"/>
<path d="M147.913 102.246C147.913 104.772 147.096 106.718 145.463 108.084C143.838 109.45 141.549 110.134 138.598 110.134H132.861V95H138.998C141.844 95 144.04 95.6211 145.585 96.8631C147.137 98.1054 147.913 99.8996 147.913 102.246ZM142.94 102.391C142.94 101.004 142.617 99.9755 141.97 99.306C141.331 98.6368 140.357 98.3022 139.047 98.3022H137.652V106.78H138.719C140.175 106.78 141.242 106.421 141.921 105.703C142.6 104.979 142.94 103.874 142.94 102.391Z" fill="white"/>
<g clip-path="url(#clip0_11_36)">
<path d="M85.0402 66.9514C84.9849 66.3632 84.9112 65.8914 84.8989 65.4134C84.8313 62.4723 84.8497 59.5312 84.7145 56.5962C84.5363 52.7544 86.3122 50.2912 89.7227 48.6675C93.6616 46.7864 97.5514 44.795 101.361 42.6688C104.034 41.1738 106.517 39.3478 109.116 37.7241C113.08 35.2548 117.338 33.5146 122.033 33.0857C129.044 32.4423 135.404 34.2744 140.972 38.5451C147.08 43.2264 150.736 49.4518 151.424 57.0987C152.414 68.1278 148.1 76.7919 138.821 82.8886C134.642 85.6275 129.941 86.902 124.946 87C121.677 87.0613 118.586 86.1728 115.544 85.0576C111.722 83.6545 108.219 81.6202 104.704 79.6043C100.095 76.9634 95.3638 74.5799 90.3433 72.803C90.0054 72.6804 89.6797 72.5028 89.3294 72.3373C90.3863 71.3692 91.388 70.4378 92.408 69.531C92.5125 69.433 92.7214 69.4268 92.8873 69.4268C93.9443 69.4207 95.0012 69.4452 96.0581 69.4207C97.6435 69.3839 98.639 68.3791 98.6144 66.8105C98.553 62.4968 98.4731 58.1832 98.3809 53.8757C98.3441 52.1049 97.8279 51.6208 96.052 51.5902C95.118 51.5779 94.1839 51.5841 93.2499 51.6331C91.517 51.725 90.6014 52.7054 90.626 54.4272C90.6506 56.1428 90.7305 57.8585 90.7182 59.5741C90.7182 60.0582 90.5891 60.5974 90.3556 61.0201C89.2004 63.1647 87.6272 64.9661 85.7223 66.4796C85.538 66.6266 85.3352 66.7492 85.0341 66.9575L85.0402 66.9514ZM125.437 43.4776C116.263 43.4592 108.895 50.763 108.852 59.9356C108.809 68.9857 116.244 76.4304 125.333 76.4426C134.446 76.4549 141.906 69.0837 141.93 60.0337C141.961 50.8978 134.581 43.4899 125.431 43.4715L125.437 43.4776Z" fill="white"/>
<path d="M170.486 83.1704C169.484 83.0724 168.987 82.7538 168.747 82.1165C168.489 81.4241 168.661 80.8236 169.177 80.3028C171.014 78.4401 172.557 76.363 173.866 74.102C180.539 62.5458 178.407 47.4604 168.802 38.1714C168.409 37.7915 168.089 37.2216 167.991 36.6947C167.874 36.0574 168.28 35.5121 168.901 35.2364C169.57 34.9423 170.246 35.0158 170.726 35.5734C172.323 37.4238 174.068 39.1885 175.433 41.1983C184.109 53.9798 182.487 71.3508 171.666 82.38C171.285 82.7721 170.738 83.005 170.486 83.1704Z" fill="white"/>
<path d="M173.073 60.2358C172.987 66.4857 170.523 72.5701 165.631 77.6803C165.207 78.1215 164.753 78.4585 164.095 78.3972C163.444 78.3359 162.989 78.0112 162.737 77.423C162.448 76.7367 162.608 76.124 163.124 75.6031C164.71 74.01 166.031 72.2209 167.1 70.2479C171.795 61.5838 170.302 50.714 163.438 43.6246C163.216 43.3979 162.983 43.1774 162.762 42.9445C162.043 42.1909 162.018 41.2534 162.688 40.5855C163.321 39.9605 164.359 39.9115 164.999 40.6284C166.344 42.148 167.751 43.643 168.87 45.328C171.703 49.5804 173.042 54.3168 173.067 60.2358L173.073 60.2358Z" fill="white"/>
<path d="M156.193 71.743C156.494 71.2957 156.746 70.7994 157.108 70.4011C160.967 66.2223 162.362 61.3694 161.115 55.8181C160.482 52.9811 159.062 50.5486 157.022 48.4653C156.721 48.1589 156.432 47.7852 156.303 47.3869C156.07 46.6639 156.432 45.9408 157.084 45.5793C157.686 45.2423 158.466 45.3587 159.044 45.9163C161.846 48.6124 163.696 51.866 164.421 55.671C165.631 62.0618 164.009 67.6928 159.579 72.4967C159.382 72.7111 159.173 72.9317 158.94 73.1033C158.405 73.5138 157.803 73.5689 157.207 73.2626C156.641 72.9685 156.34 72.4844 156.193 71.7369L156.193 71.743Z" fill="white"/>
<path d="M139.067 59.9601C139.091 67.4354 132.959 73.5689 125.431 73.5995C117.903 73.624 111.746 67.5457 111.709 60.0582C111.672 52.4174 117.75 46.3513 125.443 46.3329C132.934 46.3146 139.036 52.4235 139.067 59.9662L139.067 59.9601Z" fill="white"/>
<path d="M72.7606 76.4487C85.751 76.4487 96.1231 67.6182 96.756 54.8182C96.756 54.8182 96.2129 53.2091 95.1463 53.2091C94.0797 53.2091 93.5365 54.8182 93.5365 54.8182C92.9097 65.5778 83.7048 72.7478 72.7606 72.7478C61.8164 72.7478 52.1749 63.5385 52.1749 52.2213C52.1749 40.9041 61.4108 31.6948 72.7606 31.6948C81.6769 31.6948 89.0694 36.1683 91.9268 44.0909C91.9268 44.0909 92.9999 45.1637 94.0731 44.6273C95.1463 44.0909 94.6097 42.4818 94.6097 42.4818C91.1378 33.3337 83.1394 27.9939 72.7606 27.9939C59.3646 27.9939 48.4634 38.8637 48.4634 52.2213C48.4634 65.5789 59.3646 76.4487 72.7606 76.4487Z" fill="white"/>
</g>
<rect x="202.316" y="47" width="27.4737" height="42" rx="2" fill="#303030" stroke="#373737" stroke-width="4"/>
<circle cx="216.154" cy="64.3081" r="8.15891" fill="#232323"/>
<path d="M217.764 67.697C217.757 67.697 217.679 67.6487 217.532 67.552C217.385 67.4553 217.189 67.3586 216.943 67.2619C216.698 67.1652 216.449 67.1169 216.198 67.1169C215.947 67.1169 215.698 67.1652 215.453 67.2619C215.207 67.3586 215.012 67.4553 214.867 67.552C214.722 67.6487 214.643 67.697 214.632 67.697C214.562 67.697 214.381 67.552 214.089 67.2619C213.797 66.9718 213.651 66.792 213.651 66.7224C213.651 66.6721 213.671 66.6276 213.709 66.5889C214.011 66.2911 214.39 66.0572 214.846 65.887C215.303 65.7168 215.753 65.6317 216.198 65.6317C216.643 65.6317 217.093 65.7168 217.55 65.887C218.006 66.0572 218.385 66.2911 218.687 66.5889C218.725 66.6276 218.745 66.6721 218.745 66.7224C218.745 66.792 218.599 66.9718 218.307 67.2619C218.015 67.552 217.834 67.697 217.764 67.697ZM219.348 66.119C219.306 66.119 219.261 66.1036 219.215 66.0726C218.689 65.6665 218.201 65.3678 217.753 65.1763C217.304 64.9849 216.786 64.8892 216.198 64.8892C215.869 64.8892 215.54 64.9317 215.209 65.0168C214.878 65.1019 214.59 65.2044 214.344 65.3243C214.099 65.4442 213.879 65.5641 213.686 65.6839C213.493 65.8038 213.34 65.9063 213.228 65.9914C213.116 66.0765 213.056 66.119 213.048 66.119C212.982 66.119 212.804 65.974 212.514 65.6839C212.224 65.3939 212.079 65.214 212.079 65.1444C212.079 65.098 212.098 65.0555 212.137 65.0168C212.648 64.5063 213.266 64.1099 213.993 63.8275C214.721 63.5452 215.455 63.404 216.198 63.404C216.941 63.404 217.675 63.5452 218.402 63.8275C219.13 64.1099 219.748 64.5063 220.259 65.0168C220.298 65.0555 220.317 65.098 220.317 65.1444C220.317 65.214 220.172 65.3939 219.882 65.6839C219.592 65.974 219.414 66.119 219.348 66.119ZM220.92 64.5469C220.878 64.5469 220.835 64.5295 220.793 64.4947C220.1 63.8875 219.382 63.4301 218.637 63.1227C217.893 62.8152 217.08 62.6615 216.198 62.6615C215.316 62.6615 214.503 62.8152 213.759 63.1227C213.014 63.4301 212.296 63.8875 211.603 64.4947C211.561 64.5295 211.518 64.5469 211.476 64.5469C211.41 64.5469 211.231 64.4019 210.939 64.1118C210.647 63.8217 210.501 63.6419 210.501 63.5723C210.501 63.522 210.52 63.4775 210.559 63.4388C211.282 62.7195 212.143 62.1626 213.141 61.7681C214.139 61.3736 215.158 61.1763 216.198 61.1763C217.238 61.1763 218.257 61.3736 219.255 61.7681C220.253 62.1626 221.114 62.7195 221.837 63.4388C221.876 63.4775 221.895 63.522 221.895 63.5723C221.895 63.6419 221.749 63.8217 221.457 64.1118C221.165 64.4019 220.986 64.5469 220.92 64.5469Z" fill="white"/>
<rect x="213.228" y="78.2263" width="6.45614" height="1.61404" rx="0.807018" fill="#57E888"/>
<defs>
<clipPath id="clip0_11_36">
<rect width="132" height="59" fill="white" transform="translate(181 87) rotate(-180)"/>
</clipPath>
</defs>
</svg>

```

## File: static\src\components\attendance_menu\attendance_menu.js

```javascript
/* @odoo-module */


import { Component, useState } from "@odoo/owl";
import { Dropdown } from "@web/core/dropdown/dropdown";
import { DropdownItem } from "@web/core/dropdown/dropdown_item";
import { useDropdownState } from "@web/core/dropdown/dropdown_hooks";
import { deserializeDateTime } from "@web/core/l10n/dates";
import { rpc } from "@web/core/network/rpc";
import { registry } from "@web/core/registry";
import { useService } from "@web/core/utils/hooks";
import { isIosApp } from "@web/core/browser/feature_detection";
const { DateTime } = luxon;

export class ActivityMenu extends Component {
    static components = {Dropdown, DropdownItem};
    static props = [];
    static template = "hr_attendance.attendance_menu";

    setup() {
        this.ui = useState(useService("ui"));
        this.employee = false;
        this.state = useState({
            checkedIn: false,
            isDisplayed: false
        });
        this.date_formatter = registry.category("formatters").get("float_time")
        this.dropdown = useDropdownState();
        // load data but do not wait for it to render to prevent from delaying
        // the whole webclient
        this.searchReadEmployee();
    }

    async searchReadEmployee(){
        const result = await rpc("/hr_attendance/attendance_user_data");
        this.employee = result;
        if (this.employee.id) {
            this.hoursToday = this.date_formatter(
                this.employee.hours_today
            );
            this.hoursPreviouslyToday = this.date_formatter(
                this.employee.hours_previously_today
            );
            this.lastAttendanceWorkedHours = this.date_formatter(
                this.employee.last_attendance_worked_hours
            );
            this.lastCheckIn = deserializeDateTime(this.employee.last_check_in).toLocaleString(DateTime.TIME_SIMPLE);
            this.state.checkedIn = this.employee.attendance_state === "checked_in";
            this.isFirstAttendance = this.employee.hours_previously_today === 0;
            this.state.isDisplayed = this.employee.display_systray
        } else {
            this.state.isDisplayed = false
        }
    }

    async signInOut() {
        this.dropdown.close();
        if (!isIosApp()) { // iOS app lacks permissions to call `getCurrentPosition`
            navigator.geolocation.getCurrentPosition(
                async ({coords: {latitude, longitude}}) => {
                    await rpc("/hr_attendance/systray_check_in_out", {
                        latitude,
                        longitude
                    })
                    await this.searchReadEmployee()
                },
                async err => {
                    await rpc("/hr_attendance/systray_check_in_out")
                    await this.searchReadEmployee()
                },
                {
                    enableHighAccuracy: true,
                }
            )
        } else {
            await rpc("/hr_attendance/systray_check_in_out")
            await this.searchReadEmployee()
        }
    }
}

export const systrayAttendance = {
    Component: ActivityMenu,
};

registry
    .category("systray")
    .add("hr_attendance.attendance_menu", systrayAttendance, { sequence: 101 });

```

## File: static\src\components\attendance_menu\attendance_menu.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">

<t t-name="hr_attendance.attendance_menu">
    <t t-if="this.state.isDisplayed">
        <Dropdown position="'bottom-end'" beforeOpen.bind="searchReadEmployee" menuClass="`p-2 pb-3`" state="dropdown">
            <button>
                <i class="fa fa-circle" t-attf-class="text-{{ this.state.checkedIn ? 'success' : 'danger' }}" role="img" aria-label="Attendance"/>
            </button>
            <t t-set-slot="content">
                <div class="o_att_menu_container d-flex flex-column gap-4">
                    <div class="d-flex flex-column gap-3">
                        <div t-if="this.state.checkedIn" class="d-flex flex-wrap gap-3">
                            <div t-if="!this.isFirstAttendance" class="att_container flex-grow-1 flex-shrink-0">
                                <small class="d-block text-muted">Before <t t-esc="this.lastCheckIn"/></small>
                                <div t-esc="this.hoursPreviouslyToday" class="fs-3 text-info text-end"/>
                            </div>
                            <div class="att_container flex-grow-1 flex-shrink-0">
                                <small class="d-block text-muted">Since <t t-esc="this.lastCheckIn"/></small>
                                <div t-esc="this.lastAttendanceWorkedHours" t-attf-class="fs-3 text-info {{ !this.isFirstAttendance ? 'text-end' : '' }}"/>
                            </div>
                        </div>
                        <div t-if="!this.isFirstAttendance"
                            class="att_container d-flex flex-column"
                            t-att-class="this.state.checkedIn ? 'p-3 bg-100 rounded' : ''">
                            <div class="d-flex" t-att-class="this.state.checkedIn ? 'align-items-center justify-content-between' : 'flex-column'">
                                <small class="text-muted">Total today</small>
                                <h2 t-esc="this.hoursToday" class="mb-0 fs-2"/>
                            </div>
                            <button t-on-click="() => this.signInOut()" class="flex-basis-100 mt-3" t-attf-class="btn btn-{{ this.state.checkedIn ? 'warning' : 'success' }}">
                                <span t-if="!this.state.checkedIn">Check in</span>
                                <span t-else="">Check out</span>
                                <i t-attf-class="fa fa-sign-{{ this.state.checkedIn ? 'out' : 'in' }} ms-1"/>
                            </button>
                        </div>
                    </div>
                    <button t-if="this.isFirstAttendance" t-on-click="() => this.signInOut()" t-attf-class="btn btn-{{ this.state.checkedIn ? 'warning' : 'success' }}">
                        <span t-if="!this.state.checkedIn">Check in</span>
                        <span t-else="">Check out</span>
                        <i t-attf-class="fa fa-sign-{{ this.state.checkedIn ? 'out' : 'in' }} ms-1"/>
                    </button>
                </div>
            </t>
        </Dropdown>
    </t>
</t>

</templates>

```

## File: static\src\components\card_layout\card_layout.js

```javascript
/** @odoo-module **/

import { Component, useState, onWillUnmount } from "@odoo/owl";

const { DateTime } = luxon;
export class CardLayout extends Component {
    static template = "hr_attendance.CardLayout";
    static props = {
        slots: Object,
        fromTrialMode: { type: Boolean, optional: true },
        companyImageUrl: { type: String },
        kioskReturn: { type: Function },
    };
    static defaultProps = {
        kioskModeClasses: "",
    };

    setup() {
        this.state = useState(this.getDateTime());
        this.timeInterval = setInterval(() => {
            Object.assign(this.state, this.getDateTime());
        }, 1000);
        onWillUnmount(() => {
            clearInterval(this.timeInterval);
        });
    }

    getDateTime() {
        const now = DateTime.now();
        return {
            dayOfWeek: now.toFormat("cccc"),
            date: now.toLocaleString({
                ...DateTime.DATE_FULL,
                weekday: undefined,
            }),
            time: now.toLocaleString(DateTime.TIME_SIMPLE),
        };
    }
}

```

## File: static\src\components\card_layout\card_layout.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

<t t-name="hr_attendance.CardLayout">
    <div class="o_attendance_background d-flex flex-column h-100">
        <t t-if="env.isSmall">
            <div class="p-2 d-flex justify-content-between">
                <button
                    t-on-click="props.kioskReturn"
                    class="o_hr_attendance_back_button btn btn-secondary rounded-pill"
                    t-if="this.props.fromTrialMode">
                    <i class="oi oi-chevron-left" role="img" aria-label="Go back" title="Go back"/>
                </button>
                <t t-call="hr_attendance.companyHeader">
                    <t t-set="companyImageUrl" t-value="this.props.companyImageUrl"/>
                    <t t-set="companyName" t-value="this.props.companyName"/>
                </t>
            </div>
            <div class="o_hr_kiosk_mode_top d-flex flex-column-reverse flex-sm-row justify-content-between mx-4 my-3">
                <div class="o_hr_kiosk_mode_top_time d-flex flex-column-reverse flex-sm-row align-items-center justify-content-center mt-2 mt-sm-0">
                    <span class="me-0 me-sm-2 display-6" t-esc="state.time"/>
                    <div class="d-flex flex-sm-column gap-1 gap-sm-0 small">
                        <span t-esc="state.dayOfWeek"/>
                        <span t-esc="state.date"/>
                    </div>
                </div>
            </div>
        </t>
        <t t-else="">
            <div class="p-2 d-flex justify-content-between">
                <div class="m-2">
                    <button
                        t-on-click="props.kioskReturn"
                        class="o_hr_attendance_back_button btn btn-secondary rounded-pill"
                        t-if="this.props.fromTrialMode">
                        <i class="oi oi-chevron-left" role="img" aria-label="Go back" title="Go back"/>
                    </button>
                </div>
                <div class="o_hr_kiosk_mode_top d-flex flex-column-reverse flex-sm-row justify-content-between mx-4 my-3">
                    <div class="o_hr_kiosk_mode_top_time d-flex flex-column-reverse flex-sm-row align-items-center justify-content-center mt-2 mt-sm-0">
                        <span class="me-0 me-sm-2 display-6" t-esc="state.time"/>
                        <div class="d-flex flex-sm-column gap-1 gap-sm-0 small">
                            <span t-esc="state.dayOfWeek"/>
                            <span t-esc="state.date"/>
                        </div>
                    </div>
                </div>
                <t t-call="hr_attendance.companyHeader">
                    <t t-set="companyImageUrl" t-value="this.props.companyImageUrl"/>
                    <t t-set="companyName" t-value="this.props.companyName"/>
                </t>
            </div>
        </t>
        <div t-attf-class="o_hr_attendance_kiosk_mode d-flex flex-column w-100 h-100 {{props.kioskModeClasses}} overflow-auto px-3">
            <t t-slot="default" />
        </div>
    </div>
</t>

</templates>

```

## File: static\src\components\check_in_out\check_in_out.js

```javascript
/** @odoo-module **/

import { Component } from "@odoo/owl";
import { useService } from "@web/core/utils/hooks";
import { useDebounced } from "@web/core/utils/timing";

export class CheckInOut extends Component {
    static template = "hr_attendance.CheckInOut";
    static props = {
        checkedIn: Boolean,
        employeeId: Number,
        nextAction: String,
    };

    setup() {
        this.actionService = useService("action");
        this.orm = useService("orm");
        this.notification = useService("notification");

        this.onClickSignInOut = useDebounced(this.signInOut, 200, { immediate: true });
    }

    async signInOut() {
        navigator.geolocation.getCurrentPosition(
            ({coords: {latitude, longitude}}) => {
                this.orm.call("hr.employee", "update_last_position", [
                    [this.props.employeeId],
                    latitude,
                    longitude
                ])
            },
            err => {
                this.orm.call("hr.employee", "update_last_position", [
                    [this.props.employeeId],
                    false,
                    false
                ])
            })
        const result = await this.orm.call("hr.employee", "attendance_manual", [
            [this.props.employeeId],
            this.props.nextAction,
        ]);
        if (result.action) {
            this.actionService.doAction(result.action);
        } else if (result.warning) {
            this.notification.add(result.warning, {type: "danger"});
        }
    }
}

```

## File: static\src\components\check_in_out\check_in_out.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

<t t-name="hr_attendance.CheckInOut">
    <div class="flex-grow-1">
        <button t-on-click="() => this.onClickSignInOut()" t-attf-class="o_hr_attendance_sign_in_out_icon btn btn-{{ props.checkedIn ? 'warning' : 'success' }} align-self-center px-5 py-3 mt-4 mb-2">
            <span class="align-middle fs-2 me-3 text-white" t-if="!props.checkedIn">Check IN</span>
            <i t-attf-class="fa fa-4x fa-sign-{{ props.checkedIn ? 'out' : 'in' }} align-middle"/>
            <span class="align-middle fs-2 ms-3" t-if="props.checkedIn">Check OUT</span>
        </button>
    </div>
</t>

</templates>

```

## File: static\src\components\greetings\greetings.js

```javascript
/** @odoo-module **/

import {Component, onWillDestroy} from "@odoo/owl";
import { registry } from "@web/core/registry";
import { deserializeDateTime } from "@web/core/l10n/dates";

export class KioskGreetings extends Component {
    static template = "hr_attendance.public_kiosk_greetings";
    static props = {
        employeeData: { type: Object },
        kioskReturn: { type: Function },
    };

    setup() {
        this.formatDateTime = registry.category("formatters").get("datetime");
        this.formatFloatTime = registry.category("formatters").get("float_time");
        this.employeeName = this.props.employeeData.employee_name;
        this.employeeAvatar = this.props.employeeData.employee_avatar;
        this.hoursToday = this.formatFloatTime(this.props.employeeData.hours_today);
        this.attendance = this.props.employeeData.attendance;
        this.check_in_time = this.formatDateTime(this.attendance.check_in && deserializeDateTime(this.attendance.check_in));
        this.check_out_time = this.formatDateTime(this.attendance.check_out && deserializeDateTime(this.attendance.check_out));
        this.kiosk_delay = setTimeout(() => {
            this.props.kioskReturn(true)
        }, this.props.employeeData.kiosk_delay)
        if (this.props.employeeData.display_overtime){
            this.overtimeToday = this.formatFloatTime(this.props.employeeData.overtime_today);
            this.totalOvertime = this.formatFloatTime(this.props.employeeData.total_overtime);
        }
        onWillDestroy(() => clearTimeout(this.kiosk_delay));
    }
}

```

## File: static\src\components\greetings\greetings.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="hr_attendance.EmployeeBadge">
        <div class="o_hr_attendance_user_badge text-center">
            <img
                class="o_hr_attendance_employee_badge img rounded-circle"
                t-attf-src="{{employeeAvatar}}"
                t-attf-height="{{ employeeAvatarHeight or '120'}}"/>
        </div>
    </t>

    <t t-name="hr_attendance.public_kiosk_greetings">
        <t t-if="this.attendance">
            <div class="o_hr_kiosk_mode_main d-flex flex-grow-1 justify-content-center align-items-center">
                <div class="o_hr_attendance_kiosk_card card rounded-3">
                    <div class="card-body rounded-3">
                        <div class="o_hr_attendance_kiosk_card_wrapper d-flex flex-column align-items-center justify-content-center">
                            <div class="o_hr_attendance_kiosk_card_main p-3 mx-3 mx-xl-0 text-center">
                                <t t-call="hr_attendance.EmployeeBadge">
                                    <t t-set="employeeAvatar" t-value="this.employeeAvatar"/>
                                </t>
                                <h5 class="text-muted mt-2 mb-0">
                                    <t t-if="attendance.check_out" >Goodbye</t>
                                    <t t-else="">Welcome</t>
                                </h5>
                                <h2><t t-esc="this.employeeName"/></h2>
                            </div>
                            <div class="o_hr_attendance_kiosk_card_bottom d-flex flex-column w-100">
                                <div class="alert alert-info text-center p-3" role="status">
                                    <t t-if="attendance.check_out">
                                        Checked out at <strong><t t-esc="this.check_out_time"/></strong>
                                    </t>
                                    <t t-else="">
                                        Checked in at <strong><t t-esc="this.check_in_time"/></strong>
                                    </t>
                                </div>
                                <div class="d-flex flex-column w-100">
                                    <div
                                        class="alert alert-info text-center p-3"
                                        t-if="this.hoursToday != '00:00'"
                                        role="status">
                                        <t t-if="attendance.check_out">
                                            Hours Today: <strong><t t-esc="this.hoursToday"/></strong>
                                        </t>
                                        <t t-else="">
                                            Hours Previously Today: <strong><t t-esc="this.hoursToday"/></strong>
                                        </t>
                                    </div>
                                    <div t-if="attendance.check_out and this.overtimeToday" class="alert alert-warning d-flex flex-column gap-2 mb-0 p-2 p-xl-3 border border-warning text-center">
                                        <div class="small fw-bolder text-uppercase">
                                            Extra hours
                                        </div>
                                        <div class="d-flex gap-2">
                                            <div class="d-flex flex-column w-100 mb-0 ">
                                                <span class="text-nowrap">Today</span>
                                                <span class="fs-6"><strong><t t-esc="this.overtimeToday"/></strong></span>
                                            </div>
                                            <div t-if="this.totalOvertime" class="d-flex flex-column w-100 mb-0 ">
                                                <span class="text-nowrap">Total</span>
                                                <span class="fs-6"><strong><t t-esc="this.totalOvertime"/></strong></span>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            <div class="o_hr_kiosk_mode_bottom my-4 mx-auto">
                <button class="btn btn-primary btn-lg rounded-pill px-5" t-on-click="this.props.kioskReturn">
                    OK
                </button>
            </div>
        </t>
        <t t-else="">
            <div class="o_hr_kiosk_mode_main d-flex flex-grow-1 justify-content-center align-items-center">
                <div class="alert alert-warning px-5 text-center" role="alert">
                    <h4 class="alert-heading mt-3">Invalid request</h4>
                    <p>Please return to the main menu.</p>
                </div>
            </div>
            <div class="o_hr_kiosk_mode_bottom my-4 mx-auto">
                <button class="btn btn-primary btn-lg rounded-pill px-5" t-on-click="this.props.kioskReturn">
                    <i class="oi oi-chevron-left me-2"/>
                    <span>Go back</span>
                </button>
            </div>
        </t>
    </t>
</templates>

```

## File: static\src\components\kiosk_barcode\kiosk_barcode.js

```javascript
/** @odoo-module **/

import { BarcodeScanner } from "@barcodes/components/barcode_scanner";
import { BarcodeDialog } from '@web/core/barcode/barcode_dialog';
import { isDisplayStandalone } from "@web/core/browser/feature_detection";

export class KioskBarcodeScanner extends BarcodeScanner {
    static props = {
        ...BarcodeScanner.props,
        barcodeSource: String,
        token: String,
    };
    static template = "hr_attendance.BarcodeScanner";
    setup() {
        super.setup();
        this.isDisplayStandalone = isDisplayStandalone();
        this.scanBarcode = () => scanBarcode(this.env, this.facingMode, this.props.token);
    }

    get facingMode() {
        if (this.props.barcodeSource == "front") {
            return "user";
        }
        return super.facingMode;
    }

    get installURL() {
        const url = `hr_attendance/${this.props.token}`;
        return `/scoped_app?app_id=hr_attendance&path=${encodeURIComponent(url)}`;
    }
}

/**
 * Opens the BarcodeScanning dialog and begins code detection using the device's camera.
 *
 * @returns {Promise<string>} resolves when a {qr,bar}code has been detected
 */
export async function scanBarcode(env, facingMode = "environment", token) {
    let res;
    let rej;
    const promise = new Promise((resolve, reject) => {
        res = resolve;
        rej = reject;
    });
    env.services.dialog.add(BarcodeDialog, {
        facingMode,
        token: token,
        onResult: (result) => res(result),
        onError: (error) => rej(error),
    });
    return promise;
}

```

## File: static\src\components\manual_selection\manual_selection.js

```javascript
/** @odoo-module **/

import { Component, useState, onWillStart } from "@odoo/owl";

import { Domain } from "@web/core/domain";
import { Dropdown } from "@web/core/dropdown/dropdown";
import { DropdownItem } from "@web/core/dropdown/dropdown_item";
import { _t } from "@web/core/l10n/translation";
import { rpc } from "@web/core/network/rpc";
import { Pager } from "@web/core/pager/pager";
import { MEDIAS_BREAKPOINTS, SIZES } from "@web/core/ui/ui_service";
import { useService } from "@web/core/utils/hooks";

export class KioskManualSelection extends Component {
    static template = "hr_attendance.public_kiosk_manual_selection";
    static components = {
        Dropdown,
        DropdownItem,
        Pager,
    };
    static props = {
        displayBackButton: { type: Boolean },
        token: { type: String },
        departments: { type: Array },
        onSelectEmployee: { type: Function },
        onClickBack: { type: Function },
    };

    setup() {
        this.orm = useService("orm");
        let limit = this.calculateLimit();
        this.state = useState({
            employeesData: {
                count: 0,
                records: [],
            },
            offset: 0,
            limit: limit,
            searchInput: "",
            searchDomain: [],
            departmentDomain: [],
        });
        this.departmentName = _t("All departments");
        onWillStart(async () => {
            await this._fetchEmployeeData();
        })
    }

    calculateLimit() {
        // This function calculates the maximum number of employee cards that can fit on the screen based on his size,
        // font size, and the number of cards per row.
        let employeeCardPerLine = 1;
        let fontSizeMultiplication = 1;
        let searchBarHeight = 0;
        // for small screen the searchbar is higher
        if (screen.width <= MEDIAS_BREAKPOINTS[SIZES.SM].maxWidth){
            searchBarHeight += 38;
        } else if(screen.width <= MEDIAS_BREAKPOINTS[SIZES.MD].maxWidth){
            employeeCardPerLine = 2;
        } else if(screen.width <= MEDIAS_BREAKPOINTS[SIZES.LG].maxWidth){
            fontSizeMultiplication *= 1.25;
            employeeCardPerLine = 2;
        } else if (screen.width <= MEDIAS_BREAKPOINTS[SIZES.XL].maxWidth){
            fontSizeMultiplication *= 1.25;
            if (screen.width < 1400){ //grid breakpoint xxl
                employeeCardPerLine = 3;
            } else {
                employeeCardPerLine = 4;
            }
        } else {
            employeeCardPerLine = 4;
            if (screen.width <= 2560) {
                fontSizeMultiplication *= 1.35;
            } else {
                fontSizeMultiplication *= 2;
            }
        }
        let employeeCardHeight = 150 * fontSizeMultiplication;
        searchBarHeight += 62 * fontSizeMultiplication;
        let availableScreen = screen.height - searchBarHeight;
        return Math.trunc(availableScreen / employeeCardHeight) * employeeCardPerLine;
    }

    async _onPagerChanged({ offset, limit }) {
        this.state.offset = offset;
        this.state.limit = limit;
        await this._fetchEmployeeData();
    }

    async _fetchEmployeeData() {
        const domain = Domain.and([this.state.departmentDomain, this.state.searchDomain]).toList();
        const results = await rpc("/hr_attendance/employees_infos", {
            token: this.props.token,
            limit: this.state.limit,
            offset: this.state.offset,
            domain: domain,
        });
        this.state.employeesData.records = results.records;
        this.state.employeesData.count = results.length;
    }

    async onDepartmentClick(departmentId = false){
        if (this.env.isSmall) {
            if (departmentId){
                const selectedDepartment = this.props.departments.find((department) => department.id === departmentId);
                this.departmentName = selectedDepartment.name;
            } else {
                this.departmentName = _t("All departments");
            }
        }
        if (departmentId){
            this.state.departmentDomain = [['department_id', '=', departmentId]];
        } else {
            this.state.departmentDomain = [];
        }
        this.state.offset = 0;
        await this._fetchEmployeeData();
    }

    async onSearchInput(ev) {
        const searchInput = ev.target.value;
        if (searchInput.length){
            this.state.searchDomain = [['name', 'ilike', searchInput]];
        }else{
            this.state.searchDomain = [];
        }
        this.state.offset = 0;
        await this._fetchEmployeeData();
    }
}

```

## File: static\src\components\manual_selection\manual_selection.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="hr_attendance.public_kiosk_manual_selection">
        <div class="position-absolute top-0 start-0 w-100 h-100">
            <div class="d-flex gap-2 p-2 bg-white" style="top: 0px;">
                    <button
                        t-on-click="() => this.props.onClickBack()"
                        class="o_hr_attendance_back_button btn btn-secondary rounded-pill d-flex flex-row align-items-center"
                        t-if="this.props.displayBackButton">
                        <i class="oi oi-chevron-left me-1" role="img" aria-label="Go back" title="Go back"/>
                        Back
                    </button>
                <div class="o_control_panel_main d-flex justify-content-between align-items-lg-center flex-grow-1">
                    <div class="o_cp_searchview d-flex input-group h-100">
                        <div class="d-flex flex-row align-items-center rounded-pill border flex-grow-1">
                            <i class="o_searchview_icon d-print-none oi oi-search m-2" aria-label="Search..." title="Search..."/>
                            <div class="o_searchview_input_container position-relative w-100">
                                <input t-on-input="onSearchInput" type="text" class="d-print-none border-0" style="width: 95%;" placeholder="Search..."/>
                            </div>
                        </div>
                    </div>
                    <div class="p-2">
                        <Pager
                            t-if="state.employeesData.count > state.limit"
                            total="state.employeesData.count"
                            offset="state.offset"
                            limit="state.limit"
                            onUpdate.bind="_onPagerChanged"/>
                    </div>
                </div>
            </div>
            <div class="d-flex h-100 flex-column flex-md-row bg-300">
                <t t-if="!env.isSmall">
                    <div class="o_hr_kiosk_sidebar ps-2 py-2 overflow-auto">
                        <div class="list-group">
                            <a t-on-click="() => this.onDepartmentClick()" class="list-group-item py-3 list-group-item-action text-start">
                                All
                            </a>
                            <t t-foreach="this.props.departments" t-as="dep" t-key="dep.id">
                                <a t-on-click="() => this.onDepartmentClick(dep.id)" class="list-group-item py-3 list-group-item-action d-flex justify-content-between align-items-center">
                                    <span class="text-truncate"><t t-esc="dep.name"/></span>
                                    <small class="badge bg-secondary rounded-pill ms-3"><t t-esc="dep.count"/></small>
                                </a>
                            </t>
                        </div>
                    </div>
                </t>
                <t t-else="">
                    <Dropdown>
                        <button class="btn btn-light m-2 me-auto align-self-start">
                            <span id="departmentButton" ><i class="fa fa-users me-2"/><t t-esc="departmentName"/></span>
                            <i class="fa fa-caret-down ms-2"/>
                        </button>
                        <t t-set-slot="content">
                            <a t-on-click="() => this.onDepartmentClick()" class="d-flex text-start text-reset p-3">
                                All
                            </a>
                            <DropdownItem
                                t-foreach="this.props.departments"
                                t-as="dep"
                                t-key="dep.id"
                                class="'py-3 d-flex justify-content-between align-items-center'"
                                onSelected="() => this.onDepartmentClick(dep.id)">
                                <span class="text-truncate"><t t-esc="dep.name"/></span>
                                <small class="badge bg-secondary rounded-pill ms-3"><t t-esc="dep.count"/></small>
                            </DropdownItem>
                        </t>
                    </Dropdown>
                </t>

                <div class="o_hr_kiosk_manual_selection w-100 p-2 pt-0 pt-md-2 bg-300 overflow-auto">
                    <div class="row row-cols-1 row-cols-md-2 row-cols-xl-3 row-cols-xxl-4 g-2">
                        <t t-foreach="this.state.employeesData.records" t-as="employee" t-key="employee.id">
                            <div t-on-click="() => this.props.onSelectEmployee(employee.id)" class="col">
                                <div class="card d-flex flex-column align-items-center h-100 p-3">
                                    <img class="rounded-circle" alt="Employee Avatar" loading="lazy" t-attf-src="{{employee.avatar}}"/>
                                    <div class="d-flex flex-column align-items-center mt-2">
                                        <h6 class="mb-0" t-esc="employee.display_name"/>
                                        <p class="small text-muted text-center mb-0" t-if="employee.job_id" t-esc="employee.job_id"/>
                                    </div>
                                </div>
                            </div>
                        </t>
                    </div>
                </div>
            </div>
        </div>
    </t>
</templates>

```

## File: static\src\components\pin_code\pin_code.js

```javascript
/** @odoo-module **/

import { Component, onWillStart, useState, onWillDestroy } from "@odoo/owl";
import { browser } from "@web/core/browser/browser";

export class KioskPinCode extends Component {
    static template = "hr_attendance.KioskPinConfirm";
    static props = {
        employeeData: { type: Object },
        onClickBack: { type: Function },
        onPinConfirm: { type: Function },
    };

    setup() {
        this.padButtons = [
            ...Array.from({ length: 9 }, (_, i) => [i + 1]), // [[1], ..., [9]]
            ["C", "btn-warning"],
            [0],
            ["OK", "btn-primary"],
        ];
        this.state = useState({
            codePin: "",
        });
        this.lockPad = false;
        this.checkedIn = this.props.employeeData.attendance_state === 'checked_in';

        const onKeyDown = async (ev) => {
            const allowedKeys = [...Array(10).keys()].reduce((acc, value) => { // { from '0': '0' ... to '9': '9' }
                acc[value] = value;
                return acc;
            }, {
                'Delete': 'C',
                'Enter': 'OK',
                'Backspace': null,
            });
            const key = ev.key;

            if (!Object.keys(allowedKeys).includes(key)) {
                return;
            }

            ev.preventDefault();
            ev.stopPropagation();

            if (allowedKeys[key] !== null) {
                await this.onClickPadButton(allowedKeys[key]);
            }
            else {
                this.state.codePin = this.state.codePin.substring(0, this.state.codePin.length - 1);
            }
        }
        browser.addEventListener('keydown', onKeyDown);
        onWillStart(() => browser.addEventListener('keydown', onKeyDown))
        onWillDestroy(() => browser.removeEventListener('keydown', onKeyDown));
    }

    async onClickPadButton(value) {
        if (this.lockPad) {
            return;
        }
        if (value === "C") {
            this.state.codePin = "";
        } else if (value === "OK") {
            this.lockPad = true;
            await this.props.onPinConfirm(this.props.employeeData.id, this.state.codePin)
            this.state.codePin = "";
            this.lockPad = false;
        } else {
            this.state.codePin += value;
        }
    }
}

```

## File: static\src\components\pin_code\pin_code.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

<t t-name="hr_attendance.KioskPinConfirm">
    <t t-if="this.props.employeeData">
        <div class="o_hr_kiosk_mode_main d-flex flex-grow-1 justify-content-center align-items-center">
            <div class="card rounded-3">
                <div class="card-body rounded-3 text-center">
                    <t t-call="hr_attendance.EmployeeBadge">
                        <t t-set="employeeAvatar" t-value="this.props.employeeData.employee_avatar"/>
                        <t t-set="employeeAvatarHeight" t-value="'60'"/>
                    </t>
                    <h3 class="mt-2 mb-1"><t t-esc="this.props.employeeData.employee_name"/></h3>
                    <h5 class="text-muted my-0">
                        Please enter your PIN to
                        <t t-if="checkedIn">check out</t>
                        <t t-else="">check in</t>
                    </h5>
                    <div class="o_hr_kiosk_mode_code mb-2">
                        <div class="row">
                            <div class="col-md-8 offset-md-2 o_hr_attendance_pin_pad">
                                <div class="row g-0 my-2" >
                                    <input t-att-value="state.codePin" class="o_hr_attendance_PINbox py-0 border-0 bg-white text-center fs-3" type="password" disabled="true"/>
                                </div>
                                <div class="row g-2">
                                    <div class="col-4" t-foreach="padButtons" t-as="btn" t-key="btn[0]">
                                        <a href="#" t-on-click="() => this.onClickPadButton(btn[0])" t-attf-class="o_hr_attendance_PINbox_button btn {{btn[1]? btn[1] : 'btn-secondary'}} d-flex align-items-center justify-content-center py-2 py-xl-3 rounded-1">
                                            <t t-esc="btn[0]"/>
                                        </a>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <div class="o_hr_kiosk_mode_bottom my-4 mx-auto">
            <button class="btn btn-light btn-lg rounded-pill px-5" t-on-click="() => this.props.onClickBack()">
                <i class="oi oi-chevron-left me-2"/>
                <span>Go back</span>
            </button>
        </div>
    </t>
    <t t-else="">
        <div class="alert alert-danger mx-3" role="alert">
            <h4 class="alert-heading">Error: could not find corresponding employee.</h4>
            <p>Please return to the main menu.</p>
        </div>
        <a role="button" class="oe_attendance_sign_in_out" aria-label="Sign out" title="Sign out"/>
    </t>
</t>

</templates>

```

## File: static\src\public_kiosk\public_kiosk_app.js

```javascript
import { App, whenReady, Component, useState, onWillStart } from "@odoo/owl";
import { CardLayout } from "@hr_attendance/components/card_layout/card_layout";
import { KioskManualSelection } from "@hr_attendance/components/manual_selection/manual_selection";
import { makeEnv, startServices } from "@web/env";
import { getTemplate } from "@web/core/templates";
import { _t } from "@web/core/l10n/translation";
import { MainComponentsContainer } from "@web/core/main_components_container";
import { rpc } from "@web/core/network/rpc";
import { useService, useBus } from "@web/core/utils/hooks";
import { url } from "@web/core/utils/urls";
import { KioskGreetings } from "@hr_attendance/components/greetings/greetings";
import { KioskPinCode } from "@hr_attendance/components/pin_code/pin_code";
import { KioskBarcodeScanner } from "@hr_attendance/components/kiosk_barcode/kiosk_barcode";
import { browser } from "@web/core/browser/browser";
import { isIosApp } from "@web/core/browser/feature_detection";
import { DocumentationLink } from "@web/views/widgets/documentation_link/documentation_link";
import { session } from "@web/session";

class kioskAttendanceApp extends Component{
    static template = "hr_attendance.public_kiosk_app";
    static props = {
        token: { type: String },
        companyId: { type: Number },
        companyName: { type: String },
        departments: { type: Array },
        kioskMode: { type: String },
        barcodeSource: { type: String },
        fromTrialMode: { type: Boolean },
    };
    static components = {
        KioskBarcodeScanner,
        CardLayout,
        KioskManualSelection,
        KioskGreetings,
        KioskPinCode,
        MainComponentsContainer,
        DocumentationLink,
    };

    setup() {
        this.barcode = useService("barcode");
        this.notification = useService("notification");
        this.companyImageUrl = url("/web/binary/company_logo", {
            company: this.props.companyId,
        });
        this.state = useState({
            barcode: false,
            barcodeIsSet: false,
            active_display: "settings",
            displayDemoMessage: browser.localStorage.getItem("hr_attendance.ShowDemoMessage") !== "false",
        });
        this.lockScanner = false;
        if (this.props.kioskMode === 'settings' || this.props.fromTrialMode){
            this.manualKioskMode = false;
            useBus(this.barcode.bus, "barcode_scanned", (ev) => this.onBarcodeScanned(ev.detail.barcode));
        }
        else if (this.props.kioskMode !== 'manual') {
            useBus(this.barcode.bus, "barcode_scanned", (ev) => this.onBarcodeScanned(ev.detail.barcode));
            this.state.active_display = "main";
            this.manualKioskMode = false;
        } else {
            this.manualKioskMode = true;
            this.state.active_display = "manual";
        }
        onWillStart( async () => {
            this.isFreshDb = await rpc("/hr_attendance/is_fresh_db", { token: this.props.token });
        });
    }

    async setBadgeID() {
        let barcode = this.state.barcode;
        if (barcode) {
            const result = await rpc("/hr_attendance/set_user_barcode", { token: this.props.token, barcode, });
            if (result) {
                this.notification.add(_t("Your badge Id is now set, you can scan your badge."), { type: 'success', });
            } else {
                this.notification.add(_t("Your badge has already been set."), { type: 'danger', });
            }
            this.state.barcodeIsSet = true;
        }
    }

    switchDisplay(screen) {
        const displays = ["main", "greet", "manual", "pin", "settings"];
        if (displays.includes(screen)) {
            this.state.active_display = screen;
        } else {
            this.state.active_display = "main";
        }
    }

    async setSetting(mode) {
        await rpc("/hr_attendance/set_settings", {
            token: this.props.token,
            mode: mode,
        });
        this.props.kioskMode = mode;
        if (mode !== "manual") {
            this.manualKioskMode = false;
            this.state.active_display = "main";
            this.props.kioskMode = mode;
        } else {
            this.manualKioskMode = true;
            this.state.active_display = "manual";
            this.props.kioskMode = "manual";
        }
    }

    async kioskConfirm(employeeId){
        const employee = await rpc('attendance_employee_data',
            {
                'token': this.props.token,
                'employee_id': employeeId
            })
        if (employee && employee.employee_name){
            if (employee.use_pin){
                this.employeeData = employee
                this.switchDisplay('pin')
            }else{
                await this.onManualSelection(employeeId, false)
            }
        }
    }

    kioskReturn() {
        if (this.state.active_display === "settings"){
            history.back();
        } else if (
            (["manual", "barcode"].includes(this.props.kioskMode) ||
                (this.props.kioskMode === "barcode_manual" &&
                    this.state.active_display === "main")) &&
            this.props.fromTrialMode
        ) {
            this.switchDisplay("settings");
        } else if (this.props.kioskMode === 'manual') {
            this.switchDisplay("manual");
        } else {
            this.switchDisplay("main");
        }
    }

    displayNotification(text){
        this.notification.add(text, { type: "danger" });
    }

    async makeRpcWithGeolocation(route, params) {
        if (!isIosApp()) { // iOS app lacks permissions to call `getCurrentPosition`
            return new Promise((resolve) => {
                navigator.geolocation.getCurrentPosition(
                    async ({ coords: { latitude, longitude } }) => {
                        const result = await rpc(route, {
                            ...params,
                            latitude,
                            longitude,
                        });
                        resolve(result);
                    },
                    async (err) => {
                        const result = await rpc(route, {
                            ...params
                        });
                        resolve(result);
                    },
                    { enableHighAccuracy: true }
                );
            });
        }
        else {
            return rpc(route, {...params})
        }
    }

    async onManualSelection(employeeId, enteredPin) {
        const result = await this.makeRpcWithGeolocation('manual_selection',
            {
                'token': this.props.token,
                'employee_id': employeeId,
                'pin_code': enteredPin
            })
        if (result && result.attendance) {
            this.employeeData = result
            this.switchDisplay('greet')
        }else{
            if (enteredPin){
                this.displayNotification(_t("Wrong Pin"))
            }
        }
    }

    async onBarcodeScanned(barcode){
        if (this.lockScanner || this.state.active_display !== 'main') {
            return;
        }
        this.lockScanner = true;
        const result = await rpc('attendance_barcode_scanned',
            {
                'barcode': barcode,
                'token': this.props.token
            })
        if (result && result.employee_name) {
            this.employeeData = result
            this.switchDisplay('greet')
        }else{
            this.displayNotification(_t("No employee corresponding to Badge ID '%(barcode)s.'", { barcode }))
        }
        this.lockScanner = false
    }

    removeDemoMessage() {
        this.state.displayDemoMessage = false;
        browser.localStorage.setItem("hr_attendance.ShowDemoMessage", "false");
        return;
    }
}

export async function createPublicKioskAttendance(document, kiosk_backend_info) {
    await whenReady();
    const env = makeEnv();
    await startServices(env);
    session.server_version_info = kiosk_backend_info.server_version_info;
    const app = new App(kioskAttendanceApp, {
        getTemplate,
        env: env,
        props:
            {
                token : kiosk_backend_info.token,
                companyId: kiosk_backend_info.company_id,
                companyName: kiosk_backend_info.company_name,
                departments: kiosk_backend_info.departments,
                kioskMode: kiosk_backend_info.kiosk_mode,
                barcodeSource: kiosk_backend_info.barcode_source,
                fromTrialMode: kiosk_backend_info.from_trial_mode,
            },
        dev: env.debug,
        translateFn: _t,
        translatableAttributes: ["data-tooltip"],
    });
    return app.mount(document.body);
}
export default { kioskAttendanceApp, createPublicKioskAttendance };

```

## File: static\src\public_kiosk\public_kiosk_app.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

<t t-name="hr_attendance.companyHeader">
    <img t-att-src="companyImageUrl" alt="Company Logo" class="o_hr_attendance_kiosk_company_image align-self-center"/>
</t>

<t t-name="hr_attendance.BarcodeScanner" t-inherit="barcodes.BarcodeScanner" t-inherit-mode="primary">
    <xpath expr="//div[hasclass('o_barcode_mobile_container')]" position="replace">
        <div class="o_hr_attendance_kiosk d-flex justify-content-around" t-att-class="{'position-relative' : !isDisplayStandalone}">
            <button t-if="isBarcodeScannerSupported" t-on-click="openMobileScanner" class="o_mobile_barcode btn btn-light btn-lg p-5 rounded-3" t-att-class="{'position-absolute' : !isDisplayStandalone}">
                <i class="fa fa-3x fa-barcode mb-3"/>
                <span class="d-block">Scan your badge</span>
            </button>
            <a t-if="!isDisplayStandalone" class="o_hr_attendance_install_btn btn btn-secondary d-flex align-items-center justify-content-center fw-bolder position-relative" t-att-href="installURL" target="_blank">Install</a>
        </div>
    </xpath>
</t>

<t t-name="hr_attendance.PublicKiosksSettingScreen">
    <div class="align-self-center" t-att-class="{'h2 mt-5': !env.isSmall, 'my-3 text-center h5': env.isSmall}">
        Choose how to record attendances
    </div>
    <div class="o_hr_kiosk_mode_main d-flex m-auto" t-att-class="{'gap-5': !env.isSmall, 'gap-3 flex-column': env.isSmall}">
        <div class="d-flex flex-column align-items-center text-center gap-2" t-att-style="!env.isSmall ? 'max-width: min-content;' : ''">
            <button
                t-on-click="() => this.setSetting('manual')"
                class="btn btn-light"
                t-att-class="{
                    'd-flex flex-row rounded-2 ps-2 align-items-center gap-2' : env.isSmall,
                    'btn-lg rounded-3': !env.isSmall
                }">
                <t t-if="env.isSmall">
                    <img src="/hr_attendance/static/img/tablet-pin.svg" alt="Manual Attendance" width="40%"/>
                    <h6 class="m-0">Manually (optional PIN)</h6>
                </t>
                <t t-else="">
                    <img src="/hr_attendance/static/img/tablet-pin.svg" height="170px" alt="Manual Attendance"/>
                </t>
            </button>
            <t t-if="!env.isSmall">
                <h5 class="m-0">Select on Tablet</h5>
                <em>with optional PIN code</em>
            </t>
        </div>
        <div class="d-flex flex-column align-items-center text-center gap-2" t-att-style="!env.isSmall ? 'max-width: min-content;' : ''">
            <button
                t-on-click="() => this.setSetting('barcode')"
                class="btn btn-light"
                t-att-class="{
                    'flex-row d-flex rounded-2 ps-2 align-items-center gap-2' : env.isSmall,
                    'btn-lg rounded-3': !env.isSmall
                }">
                <t t-if="env.isSmall">
                    <img src="/hr_attendance/static/img/tablet-cam.svg" alt="Attendance with barcode" width="40%"/>
                    <h6 class="m-0">Badge with Barcode</h6>
                </t>
                <t t-else="">
                    <img src="/hr_attendance/static/img/tablet-cam.svg" height="170px" alt="Attendance with barcode"/>
                </t>
            </button>
            <t t-if="!env.isSmall">
                <h5 class="m-0 w-75">Badge with Barcode on Tablet</h5>
            </t>
        </div>
        <div class="d-flex flex-column align-items-center text-center gap-2" t-att-style="!env.isSmall ? 'max-width: min-content;' : ''">
            <button
                t-on-click="() => this.setSetting('barcode_manual')"
                class="btn btn-light"
                t-att-class="{
                    'flex-row d-flex rounded-2 ps-2 align-items-center gap-2' : env.isSmall,
                    'btn-lg rounded-3': !env.isSmall
                }">
                <t t-if="env.isSmall">
                    <img src="/hr_attendance/static/img/tablet-rfid.svg" alt="Manual Attendance or with barcode" width="40%"/>
                    <h6 class="m-0">RFID Token with reader</h6>
                </t>
                <t t-else="">
                    <img src="/hr_attendance/static/img/tablet-rfid.svg" height="170px"
                    alt="Manual Attendance or with barcode"/>
                </t>
            </button>
            <t t-if="!env.isSmall">
                <h5 class="m-0 w-75">RFID Token with reader on tablet</h5>
            </t>
        </div>
    </div>
    <div class="o_hr_kiosk_mode_bottom my-5 mx-auto">
        Powered by <img t-att-height="env.isSmall ? 21 : 32" src="/web/static/img/logo.png" alt="Odoo Logo"/>
    </div>
</t>

<t t-name="hr_attendance.NoBadgeEmployeeScreen">
    <div class="alert alert-info align-items-center align-self-center d-flex gap-2" t-att-class="{'flex-wrap': env.isSmall}" style="width: fit-content;" role="alert">
        <span class="m-0" t-att-class="{'text-align-center w-100': env.isSmall}" t-att-style="!env.isSmall? 'min-width: fit-content;' : ''">
            No Badge defined on employees. Set one to test.
        </span>
        <div class="input-group">
            <input id="manual_barcode"
                t-model="state.barcode"
                type="text"
                name="barcode"
                t-ref="demobadgeID"
                class="input-group-text text-start"
                placeholder="e.g. 1102021021"
                t-att-class="{'flex-grow-1': env.isSmall}"
                t-att-style="env.isSmall? 'width: 15ch;' : ''"/>
            <button type="button" class="btn btn-primary" t-on-click="setBadgeID" t-att-class="{'flex-shrink-0': env.isSmall}">
                Set my badge
            </button>
        </div>
    </div>
</t>

<t t-name="hr_attendance.public_kiosk_app">
    <MainComponentsContainer/>
    <CardLayout fromTrialMode="this.props.fromTrialMode" companyImageUrl="this.companyImageUrl" kioskReturn.bind="kioskReturn">
        <t t-if="this.state.active_display === 'settings'">
            <t t-call="hr_attendance.PublicKiosksSettingScreen"/>
        </t>
        <t t-if="this.state.active_display === 'main'">
            <t t-if="isFreshDb and !state.barcodeIsSet">
                <t t-call="hr_attendance.NoBadgeEmployeeScreen"/>
            </t>
            <t t-elif="state.displayDemoMessage">
                <div
                    class="alert alert-info flex-row justify-content-between d-flex"
                    t-att-class="{'align-self-center col-6': !env.isSmall}"
                    t-att-style="!env.isSmall? 'min-width: fit-content;' : ''">
                        <div>
                            Connect an RFID reader, and scan a token.
                            <DocumentationLink path="'/applications/hr/attendances/hardware.html'" label.translate="'Documentation'" alertLink="true"/>
                        </div>
                        <button t-on-click="removeDemoMessage" type="button" class="btn-close float-end ms-2" title="Close"/>
                </div>
            </t>
            <div class="o_hr_kiosk_mode_main d-flex flex-column flex-sm-row gap-3 m-auto">
                <t t-if="this.props.kioskMode !== 'manual'">
                <KioskBarcodeScanner
                    token="this.props.token"
                    barcodeSource="this.props.barcodeSource"
                    onBarcodeScanned="(ev) => this.onBarcodeScanned(ev)"/>
                </t>
            </div>
            <div class="o_hr_kiosk_mode_bottom d-flex flex-column flex-md-row gap-2 align-items-center justify-content-between my-5">
                <t t-if="this.props.kioskMode !== 'barcode'">
                    <button t-on-click="() => this.switchDisplay('manual')"  class="btn btn-light btn-lg rounded-3">
                        <i class="fa fa-user-o me-2"/>
                        <span >Identify Manually</span>
                    </button>
                    <span>
                        Powered by <img height="32" src="/web/static/img/logo.png" alt="Odoo Logo"/>
                    </span>
                </t>
                <t t-else="">
                    <div class="o_hr_kiosk_mode_bottom mx-auto">
                        Powered by <img height="32" src="/web/static/img/logo.png" alt="Odoo Logo"/>
                    </div>
                </t>
            </div>
        </t>
        <t t-if="this.state.active_display === 'manual'">
            <KioskManualSelection
                displayBackButton="!this.manualKioskMode || this.props.fromTrialMode"
                departments="this.props.departments"
                onSelectEmployee="(e) => this.kioskConfirm(e)"
                onClickBack="() => this.kioskReturn()"
                token="this.props.token"/>
        </t>
        <t t-if="this.state.active_display === 'greet'">
            <KioskGreetings employeeData="this.employeeData" kioskReturn="() => this.kioskReturn(true)"/>
        </t>
        <t t-if="this.state.active_display === 'pin'">
            <KioskPinCode
                employeeData="this.employeeData"
                onPinConfirm="(id, pin) => this.onManualSelection(id, pin)"
                onClickBack="() => this.kioskReturn()"/>
        </t>
    </CardLayout>
</t>
</templates>

```

## File: static\src\views\attendance_helper_view.js

```javascript
import { user } from "@web/core/user";
import { useService } from "@web/core/utils/hooks";

import { Component, onWillStart, useState } from "@odoo/owl";

export class AttendanceActionHelper extends Component {
    static template = "hr_attendance.AttendanceActionHelper";
    static props = ["noContentHelp"];
    setup() {
        this.orm = useService("orm");
        this.actionService = useService("action");
        this.state = useState({
            hasDemoData: false,
        });
        onWillStart(async () => {
            this.hasAttendanceRight = await user.hasGroup("hr_attendance.group_hr_attendance_manager");
            if (this.hasAttendanceRight){
                this.state.hasDemoData = await this.orm.call("hr.attendance", "has_demo_data", []);
            }
        });
    }

    loadAttendanceScenario() {
        this.actionService.doAction("hr_attendance.action_load_demo_data");
    }

    LoadTryKiosk() {
        this.actionService.doAction("hr_attendance.action_try_kiosk");
    }
};

```

## File: static\src\views\attendance_helper_view.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <t t-name="hr_attendance.AttendanceActionHelper">
        <div class="o_view_nocontent">
            <div class="o_nocontent_help">
                <t t-if="hasAttendanceRight">
                    <t t-if="env.isSmall">
                        <t t-call="hr_attendance.AttendanceActionHelperMobileScreen"/>
                    </t>
                    <t t-else="">
                        <t t-call="hr_attendance.AttendanceActionHelperRegularScreen"/>
                    </t>
                </t>
                <t t-else="">
                    <p class="o_view_nocontent_empty_folder">
                        No attendance records found
                    </p>
                    <p>
                        The attendance records of your employees will be displayed here.
                    </p>
                </t>
            </div>
        </div>
    </t>

    <t t-name="hr_attendance.AttendanceActionHelperMobileScreen">
        <div class="d-flex flex-column align py-2 gap-4 oe_view_nocontent_attendance_mobile">
            <h3>
                Ready to track attendances ?
            </h3>
            <a type="object" t-on-click="() => this.LoadTryKiosk()">
                <img src="/hr_attendance/static/img/mock-tablet.png" height="180" class="mb-2"/>
                <div class="row justify-content-center mt-1">
                    <div class="btn btn-primary d-block col-6">
                        Try the kiosk
                    </div>
                </div>
            </a>
            <div>
                <img src="/hr_attendance/static/img/attendance_dot.gif" height="180" class="mb-2"/>
                <h3 class="mt-2">
                    <em>Try the top
                        <i class="fa fa-circle text-danger" role="img" aria-label="Attendance"/>
                        icon (e.g for work from home)
                    </em>
                </h3>
            </div>
            <t t-if="!state.hasDemoData">
                <div class="d-flex gap-3 align-items-center or-separator">
                    <hr class="flex-grow-1"/>
                    or
                    <hr class="flex-grow-1"/>
                </div>
            <div class="px-2">
                <h3 class="pb-1">
                    Try the backend and reporting:
                </h3>
                <div class="row justify-content-center">
                    <a type="object" class="btn btn-secondary d-block col-6" t-on-click="() => this.loadAttendanceScenario()">
                        Load sample data
                    </a>
                </div>
            </div>
        </t>
        </div>
    </t>

    <t t-name="hr_attendance.AttendanceActionHelperRegularScreen">
        <p class="oe_view_nocontent_attendance">
            <h3 class="py-2">
                Ready to track attendances ?
            </h3>
            <div class="d-flex align py-2">
                <a class="mx-5" type="object" t-on-click="() => this.LoadTryKiosk()">
                    <img src="/hr_attendance/static/img/mock-tablet.png" height="180" class="mb-2"/>
                    <div class="row justify-content-center mt-1">
                        <div class="btn btn-primary d-block col-6">
                            Try the kiosk
                        </div>
                    </div>
                </a>
                <div class="mx-5" >
                        <img src="/hr_attendance/static/img/attendance_dot.gif" height="180" class="mb-2"/>
                    <h3 class="mt-2"><em>Try the top
                        <i class="fa fa-circle text-danger" role="img" aria-label="Attendance"/>
                    icon (e.g for work from home)</em></h3>
                </div>
            </div>
            <t t-if="!state.hasDemoData">
                    <div class="d-flex gap-3 align-items-center or-separator">
                        <hr class="flex-grow-1"/>
                        or
                        <hr class="flex-grow-1"/>
                    </div>
                <div class="px-2 py-2">
                        <h3 class="m-2 py-2">
                            Try the backend and reporting:
                        </h3>
                    <div class="row justify-content-center">
                        <a type="object" class="btn btn-secondary d-block col-2" t-on-click="() => this.loadAttendanceScenario()">
                            Load sample data
                        </a>
                    </div>
                </div>
            </t>
        </p>
    </t>
</odoo>

```

## File: static\src\views\attendance_list_view.js

```javascript
import { registry } from "@web/core/registry";

import { listView } from "@web/views/list/list_view";
import { ListRenderer } from "@web/views/list/list_renderer";
import { AttendanceActionHelper } from "@hr_attendance/views/attendance_helper_view";

export class AttendanceListRenderer extends ListRenderer {
    static template = "hr_attendance.AttendanceListRenderer";
    static components = {
        ...AttendanceListRenderer.components,
        AttendanceActionHelper,
    };

    /** @override **/
    get showNoContentHelper() {
        // Rows's length need to be lower than 6 to avoid nocontent overlapping
        return super.showNoContentHelper && this.props.list.count < 6 ;
    }
};

export class AttendanceListModel extends listView.Model {

    /** @override **/
    async load(params = {}) {
        const activeDomainParam = params.domain?.some((index) => Array.isArray(index) && index[0] == "employee_id.active");
        if (!activeDomainParam) {
            params.domain?.push(["employee_id.active", "=", true]);
        }
        return super.load(params);
    }
}

export const attendanceListView = {
    ...listView,
    Renderer: AttendanceListRenderer,
    Model: AttendanceListModel,
};

registry.category("views").add("attendance_list_view", attendanceListView);

```

## File: static\src\views\attendance_list_view.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <t t-name="hr_attendance.AttendanceListRenderer" t-inherit="web.ListRenderer" t-inherit-mode="primary">
        <t t-call="web.ActionHelper" position="replace">
        </t>
        <table position="after">
            <t t-if="showNoContentHelper">
                <AttendanceActionHelper noContentHelp="props.noContentHelp"/>
            </t>
        </table>
    </t>
</odoo>

```

## File: views\hr_attendance_kiosk_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="hr_attendance.public_kiosk_mode" name="Attendance Kiosk">
        <t t-call="web.layout">
            <t t-set="html_data" t-value="{'lang': kiosk_backend_info['lang']}"/>
            <t t-set="head">
                <title>Attendance Kiosk</title>
                <meta http-equiv="X-UA-Compatible" content="IE=edge"/>
                <meta http-equiv="content-type" content="text/html, charset=utf-8" />
                <meta name="viewport" content="width=device-width, initial-scale=1, user-scalable=no"/>
                <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
                <t t-call-assets="hr_attendance.assets_public_attendance" t-js="false"/>
                <t t-call-assets="hr_attendance.assets_public_attendance" t-css="false"/>
                <t t-call="web.conditional_assets_tests">
                    <t t-set="ignore_missing_deps" t-value="True"/>
                </t>

                <script type="text/javascript">
                    odoo.define("hr_attendance.public_kiosk_app", ["@hr_attendance/public_kiosk/public_kiosk_app"], function (require) {
                    var { createPublicKioskAttendance } = require("@hr_attendance/public_kiosk/public_kiosk_app");
                    createPublicKioskAttendance(document, <t t-out="json.dumps(kiosk_backend_info)"/>);
                    });
                </script>
            </t>
            <t t-set="body">
            </t>
            <body class="o_web_client o_hr_attendance_kiosk_body position-relative">
            </body>
        </t>
    </template>
</odoo>

```

## File: views\hr_attendance_overtime_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_attendance_overtime_tree" model="ir.ui.view">
        <field name="name">hr.attendance.overtime.list</field>
        <field name="model">hr.attendance.overtime</field>
        <field name="arch" type="xml">
            <list edit="0" create="0">
                <field name="date"/>
                <field name="employee_id"/>
                <field name="duration" widget="float_time"/>
            </list>
        </field>
    </record>

    <record id="view_attendance_overtime_search" model="ir.ui.view">
        <field name="name">hr.attendance.overtime.search</field>
        <field name="model">hr.attendance.overtime</field>
        <field name="arch" type="xml">
            <search>
                <field name="employee_id"/>
                <field name="duration" filter_domain="[('duration', '>=', self)]"/>
                <filter string="Last 3 Months" invisible="1" name="last_three_months" domain="[(
                'date','&gt;=', (
                    context_today() + relativedelta(months=-3)
                    )
                )]"/>
                <group expand="0" string="Group By">
                    <filter string="Date" name="groupby_date" context="{'group_by': 'date:week'}"/>
                    <filter string="Employee" name="employee" context="{'group_by': 'employee_id'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="view_attendance_overtime_graph" model="ir.ui.view">
        <field name="name">hr.attendance.overtime.graph</field>
        <field name="model">hr.attendance.overtime</field>
        <field name="arch" type="xml">
            <graph string="Overtime" type="bar" stacked="0" sample="1">
                <field name="employee_id" type="row"/>
                <field name="date" interval="week" type="col"/>
                <field name="duration" type="measure" widget="float_time"/>
            </graph>
        </field>
    </record>

    <record id="hr_attendance_overtime_view_pivot" model="ir.ui.view">
        <field name="name">hr.attendance.overtime.pivot</field>
        <field name="model">hr.attendance.overtime</field>
        <field name="arch" type="xml">
            <pivot string="Worked Hours">
                <field name="employee_id" type="row"/>
                <field name="date" type="col" interval="month"/>
                <field name="duration" type="measure" widget="float_time"/>
            </pivot>
        </field>
    </record>

    <record id="hr_attendance_overtime_action" model="ir.actions.act_window">
        <field name="name">Extra Hours</field>
        <field name="res_model">hr.attendance.overtime</field>
        <field name="view_mode">graph,pivot,list</field>
        <field name="context">
            {
                "search_default_groupby_date": 1,
                "search_default_employee": 1,
                "search_default_last_three_months": 1
            }
        </field>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                No overtime records found
            </p><p>
                The overtime records of your employees will be displayed here.
            </p>
        </field>

    </record>

</odoo>

```

## File: views\hr_attendance_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- views -->

    <record id="view_attendance_tree" model="ir.ui.view">
        <field name="name">hr.attendance.list</field>
        <field name="model">hr.attendance</field>
        <field name="arch" type="xml">
            <list
                js_class="attendance_list_view"
                string="Employee attendances"
                decoration-success="color == 10"
                decoration-danger="color == 1"
                duplicate="false">
                <header>
                    <button class="btn-secondary" string="Approve Extra Hours" name="action_approve_overtime" type="object"/>
                    <button class="btn-secondary" string="Refuse Extra Hours" name="action_refuse_overtime" type="object"/>
                </header>
                <field name="employee_id" widget="many2one_avatar_user"/>
                <field name="check_in"/>
                <field name="check_out" options="{}"/>
                <field name="worked_hours" string="Worked Hours" widget="float_time"/>
                <field name="overtime_hours" string="Worked Extra Hours" optional="show" widget="float_time"/>
                <field name="validated_overtime_hours" string="Extra Hours" optional="show" widget="float_time"/>
                <field name="overtime_status" optional="hidden" widget="badge" decoration-warning="overtime_status == 'to_approve'" decoration-success="overtime_status == 'approved'" decoration-danger="overtime_status == 'refused'"/>
                <field name="in_mode" string="Input Mode (In)" optional="hidden"/>
                <field name="out_mode" string="Input Mode (Out)" optional="hidden"/>
                <field name="in_latitude" string="Latitude (In)" optional="hidden"/>
                <field name="in_longitude" string="Longitude (In)" optional="hidden"/>
                <field name="out_latitude" string="Latitude (Out)" optional="hidden"/>
                <field name="out_longitude" string="Longitude (Out)" optional="hidden"/>
                <field name="in_city" string="City (In)" optional="hidden"/>
                <field name="out_city" string="City (Out)" optional="hidden"/>
                <field name="in_country_name" string="Country (In)" optional="hidden"/>
                <field name="out_country_name" string="Country (Out)" optional="hidden"/>
                <field name="create_uid" optional="hidden"/>
                <field name="write_uid" optional="hidden"/>
                <field name="write_date" optional="hidden"/>
                <field name="color" column_invisible="1"/>
            </list>
        </field>
    </record>

    <record id="view_hr_attendance_kanban" model="ir.ui.view">
        <field name="name">hr.attendance.kanban</field>
        <field name="model">hr.attendance</field>
        <field name="arch" type="xml">
            <kanban class="o_kanban_mobile" sample="1">
                <field name="check_in"/>
                <field name="check_out"/>
                <templates>
                    <t t-name="card">
                        <field name="employee_id" widget="many2one_avatar_user" options="{'display_avatar_name': True}" class="fs-5 fw-bold mb-2"/>
                        <hr class="mt4 mb8"/>
                        <div>
                            <i class="fa fa-calendar me-1" aria-label="Period" role="img" title="Period"></i>
                            <field name="check_in" /> - <field name="check_out" />
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="hr_attendance_view_form" model="ir.ui.view">
        <field name="name">hr.attendance.form</field>
        <field name="model">hr.attendance</field>
        <field name="arch" type="xml">
            <form string="Employee attendances" duplicate="false">
                <header>
                    <field name="overtime_status" widget="statusbar" invisible="no_validated_overtime_hours" readonly="0" options="{'clickable': '1'}"/>
                    <field name="overtime_status" widget="statusbar" invisible="not no_validated_overtime_hours" statusbar_visible="approved,refused" readonly="0" options="{'clickable': '1'}"/>
                </header>
                <sheet>
                    <group>
                        <group colspan="2">
                            <group col="1">
                                <field name="employee_id" widget="many2one_avatar_user"/>
                                <field name="check_in" options="{'rounding': 0}"/>
                                <field name="check_out" options="{'rounding': 0}" placeholder="Currently Working"/>
                            </group>
                            <group col="2">
                                <field name="worked_hours" string="Worked Time" widget="float_time"/>
                                <field name="overtime_hours" widget="float_time" string="Worked Extra Hours" invisible="overtime_hours == validated_overtime_hours"/>
                                <label for="validated_overtime_hours"/>
                                <div class="o_row">
                                <field
                                    name="validated_overtime_hours"
                                    class="o_hr_narrow_field-4"
                                    widget="float_time"
                                    string="Extra Hours"
                                    readonly="overtime_status == 'refused'"
                                    />
                                    <button class="oe_stat_button" string="Approve" invisible="overtime_status not in ['to_approve', 'refused'] or no_validated_overtime_hours" name="action_approve_overtime" icon="fa-check" type="object"/>
                                    <button class="oe_stat_button" string="Refuse" invisible="overtime_status not in ['to_approve', 'approved'] or no_validated_overtime_hours" name="action_refuse_overtime" icon="fa-times" type="object"/>
                                </div>
                            </group>
                        </group>
                        <separator string="Check In"/>
                        <group name="check_in_group" colspan="2">
                            <group>
                                <group>
                                    <field name="in_mode"/>
                                    <field name="in_ip_address" invisible="in_mode == 'manual'"/>
                                    <field name="in_browser" invisible="in_mode == 'manual'"/>
                                </group>
                            </group>
                            <group invisible="in_mode == 'manual'">
                                <label for="in_country_name" string="Localisation"/>
                                <div class="o_row" name="in_location_info">
                                    <span>
                                        <field name="in_country_name"/>
                                    </span>
                                    <span>
                                        <field name="in_city" invisible="in_city == 'Unknown'" nolabel="1"/>
                                    </span>
                                </div>

                                <label for="in_latitude" string="GPS Coordinates"/>
                                <div>
                                    <div class="o_row">
                                        <span>
                                            <field name="in_latitude"/>
                                        </span>,
                                        <span>
                                            <field name="in_longitude" nolabel="1"/>
                                        </span>
                                    </div>
                                    <button
                                        name="action_in_attendance_maps"
                                        type="object"
                                        class="btn btn-link ps-0 pt-0 pb-2"
                                        icon="oi-arrow-right"
                                        string="View on Maps"
                                        colspan="2"/>
                                </div>
                            </group>
                        </group>
                        <separator string="Check Out" name="check_out_separator" invisible="not check_out"/>
                        <group colspan="2" name="check_out_group" invisible="not check_out">
                            <group>
                                <group>
                                    <field name="out_mode" string="Mode"/>
                                    <field name="out_ip_address" string="IP Address" invisible="in_mode == 'manual'"/>
                                    <field name="out_browser" string="Browser" invisible="in_mode == 'manual'"/>
                                </group>
                            </group>
                            <group invisible="out_mode == 'manual'">
                                <label for="out_country_name" string="Localisation"/>
                                <div class="o_row" name="out_location_info" >
                                    <span>
                                        <field name="out_country_name"/>
                                    </span>
                                    <span>
                                        <field name="out_city" invisible="out_city == 'Unknown'" nolabel="1"/>
                                    </span>
                                </div>

                                <label for="out_latitude" string="GPS Coordinates"/>
                                <div>
                                    <div class="o_row">
                                        <span>
                                            <field name="out_latitude"/>
                                        </span>,
                                        <span>
                                            <field name="out_longitude" nolabel="1"/>
                                        </span>
                                    </div>
                                    <button
                                        name="action_out_attendance_maps"
                                        type="object"
                                        class="btn btn-link ps-0 pt-0 pb-2"
                                        icon="oi-arrow-right"
                                        string="View on Maps"
                                        colspan="2"/>
                                </div>
                            </group>
                        </group>
                    </group>
                </sheet>
                <chatter/>
            </form>
        </field>
    </record>

    <record id="hr_attendance_view_graph" model="ir.ui.view">
        <field name="name">hr.attendance.graph</field>
        <field name="model">hr.attendance</field>
        <field name="arch" type="xml">
            <graph string="Worked Hours" type="line" stacked="0" sample="1">
                <field name="employee_id" type="row"/>
                <field name="check_in" interval="week" type="col"/>
                <field name="overtime_hours" widget="float_time"/>
                <field name="worked_hours" type="measure" widget="float_time"/>
            </graph>
        </field>
    </record>

    <record id="hr_attendance_view_pivot" model="ir.ui.view">
        <field name="name">hr.attendance.pivot</field>
        <field name="model">hr.attendance</field>
        <field name="arch" type="xml">
            <pivot string="Worked Hours">
                <field name="employee_id" type="row"/>
                <field name="check_in" type="col" interval="month"/>
                <field name="worked_hours" type="measure" widget="float_time"/>
                <field name="expected_hours" type="measure" widget="float_time"/>
                <field name="overtime_hours" string="Difference" type="measure" widget="float_time"/>
                <field name="validated_overtime_hours" string="Balance" type="measure" widget="float_time"/>
            </pivot>
        </field>
    </record>

    <record id="hr_attendance_view_filter" model="ir.ui.view">
        <field name="name">hr_attendance_view_filter</field>
        <field name="model">hr.attendance</field>
        <field name="arch" type="xml">
            <search string="Hr Attendance Search">
                <field name="employee_id"/>
                <field name="department_id" operator="child_of"/>
                <field name="check_in"/>
                <filter string="My Attendances" name="myattendances" domain="[('employee_id.user_id', '=', uid)]" />
                <filter string="My Team" name="myteam" domain="[('employee_id.parent_id.user_id', '=', uid)]"/>
                <separator/>
                <filter string="At Work" name="nocheckout" domain="[('check_out', '=', False)]" />
                <filter string="Errors" name="errors"
                        domain="['|', ('worked_hours', '&gt;=', 16), '&amp;', ('check_out', '=', False), ('check_in', '&lt;=',  (context_today() - datetime.timedelta(days=1)).strftime('%Y-%m-%d'))]"/>
                <filter string="Automatically Checked-Out" name="auto_check"
                        domain="[('out_mode', '=', 'auto_check_out')]"/>
                <separator/>
                <filter string="Date" name="check_in_filter" date="check_in"/>
                <separator/>
                <filter
                    string="Active Employees"
                    name="activeemployees"
                    domain="[('employee_id.active', '=', True)]"/>
                <filter
                    string="Archived Employees"
                    name="archivedemployees"
                    domain="[('employee_id.active', '=', False)]"/>
                <separator invisible="1"/>
                <filter string="Last 3 Months" invisible="1" name="last_three_months" domain="[(
                    'check_in','&gt;=', (
                        context_today() + datetime.timedelta(days=-90)
                        )
                    )]"/>
                <group expand="0" string="Group By">
                    <filter string="Employee" name="employee" context="{'group_by': 'employee_id'}"/>
                    <filter string="Departement" name="departement" context="{'group_by': 'department_id'}"/>
                    <filter string="Manager" name="manager" context="{'group_by': 'manager_id'}"/>
                    <filter string="Method" name="groupby_mode_in" context="{'group_by': 'in_mode'}"/>
                    <filter string="Date" name="groupby_name" context="{'group_by': 'check_in:month'}"/>
                </group>
            </search>
        </field>
    </record>

    <!-- actions -->

    <record id="action_try_kiosk" model="ir.actions.server">
        <field name="name">Try kiosk</field>
        <field name="model_id" ref="hr_attendance.model_hr_attendance"/>
        <field name="state">code</field>
        <field name="code">action = model.action_try_kiosk()</field>
    </record>

    <record id="action_load_demo_data" model="ir.actions.server">
        <field name="name">Load demo data</field>
        <field name="model_id" ref="hr_attendance.model_hr_attendance"/>
        <field name="state">code</field>
        <field name="code">action = model._load_demo_data()</field>
    </record>

    <record id="hr_attendance_action" model="ir.actions.act_window">
        <field name="name">Attendances</field>
        <field name="path">attendances</field>
        <field name="res_model">hr.attendance</field>
        <field name="view_mode">list,form</field>
        <field name="context">
            {
                "search_default_groupby_name": 1,
                "search_default_employee": 2
            }
        </field>
        <field name="search_view_id" ref="hr_attendance_view_filter"/>
        <field name="help">
        </field>
    </record>

    <record id="hr_attendance_reporting" model="ir.actions.act_window">
        <field name="name">Attendances</field>
        <field name="res_model">hr.attendance</field>
        <field name="view_mode">pivot,graph</field>
        <field name="search_view_id" ref="hr_attendance_view_filter"/>
        <field name="context">
            {
                "search_default_groupby_name" : 1,
                "search_default_employee": 2,
                "search_default_activeemployees": 1,
                "search_default_last_three_months": 1
            }
        </field>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                No attendance records found
            </p><p>
                The attendance reporting of your employees will be displayed here.
            </p>
        </field>
    </record>

    <record id="hr_attendance_management_view_filter" model="ir.ui.view">
        <field name="name">hr_attendance_management_view_filter</field>
        <field name="model">hr.attendance</field>
        <field name="arch" type="xml">
            <search>
                <field name="employee_id"/>
                <field name="department_id" operator="child_of"/>
                <filter string="My Attendances" name="myattendances" domain="[('employee_id.user_id', '=', uid)]" />
                <filter string="My Team" name="myteam" domain="[('employee_id.parent_id.user_id', '=', uid)]"/>
                <separator/>
                <filter string="To Approve" name="to_approve" domain="[('overtime_status','=', 'to_approve'), ('overtime_hours', '!=', 0)]"/>
                <filter string="Approved" name="approved" domain="[('overtime_status','=', 'approved')]"/>
                <filter string="Refused" name="refused" domain="[('overtime_status','=', 'refused')]"/>
                <separator/>
                <filter
                    string="Active Employees"
                    name="activeemployees"
                    domain="[('employee_id.active', '=', True)]"/>
                <filter
                    string="Archived Employees"
                    name="archivedemployees"
                    domain="[('employee_id.active', '=', False)]"/>
                <group string="Group By">
                    <filter string="Employee" name="employee" context="{'group_by': 'employee_id'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="view_attendance_tree_management" model="ir.ui.view">
        <field name="name">hr.attendance.list</field>
        <field name="model">hr.attendance</field>
        <field name="arch" type="xml">
            <list string="Employee attendances" create="0">
                <header>
                    <button class="btn-secondary" string="Approve" name="action_approve_overtime" type="object"/>
                    <button class="btn-secondary" string="Refuse" name="action_refuse_overtime" type="object"/>
                </header>
                <field name="employee_id" widget="many2one_avatar_user" readonly="1"/>
                <field name="check_in" readonly="1"/>
                <field name="check_out" readonly="1"/>
                <field name="worked_hours" readonly="1" string="Worked Time" widget="float_time"/>
                <field name="overtime_hours" string="Worked Extra Hours" widget="float_time"/>
                <field name="validated_overtime_hours" readonly="overtime_status == 'refused'" string="Extra Hours" widget="float_time"/>
                <field name="overtime_status" widget="badge" decoration-warning="overtime_status == 'to_approve'" decoration-success="overtime_status == 'approved'" decoration-danger="overtime_status == 'refused'"/>
                <button class="oe_stat_button" string="Approve" invisible="overtime_status not in ['to_approve', 'refused']" name="action_approve_overtime" icon="fa-check" type="object"/>
                <button class="oe_stat_button" string="Refuse" invisible="overtime_status not in ['to_approve', 'approved']" name="action_refuse_overtime" icon="fa-times" type="object"/>
            </list>
        </field>
    </record>

    <record id="hr_attendance_management_action" model="ir.actions.act_window">
        <field name="name">Management</field>
        <field name="res_model">hr.attendance</field>
        <field name="view_mode">list,form</field>
        <field name="search_view_id" ref="hr_attendance_management_view_filter"/>
        <field name="view_id" ref="view_attendance_tree_management"/>
        <field name="context">
            {
                "search_default_to_approve" : 1,
                "search_default_activeemployees": 1,
            }
        </field>
        <field name="domain">[('check_out', '!=', False)]</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                No attendance records found
            </p><p>
                The attendance reporting of your employees will be displayed here.
            </p>
        </field>
    </record>

    <record id="hr_attendance_action_greeting_message" model="ir.actions.client">
        <field name="name">Message</field>
        <field name="tag">hr_attendance_greeting_message</field>
    </record>

    <!-- not used anymore, will be removed in master -->
    <record id="hr_attendance_action_install_kiosk_pwa" model="ir.actions.client">
        <field name="name">Attendance Kiosk</field>
        <field name="target">new</field>
        <field name="res_model">hr.attendance</field>
        <field name="tag">install_kiosk_pwa</field>
        <field name="context">{'app_id': 'hr_attendance', 'footer': false}</field>
    </record>

    <record model="ir.actions.server" id="open_kiosk_url">
        <field name="name">Open Kiosk Url</field>
        <field name="model_id" ref="hr_attendance.model_res_company"/>
        <field name="path">attendance-kiosk</field>
        <field name="binding_model_id" ref="hr_attendance.model_res_company"/>
        <field name="state">code</field>
        <field name="code">
            action = model._action_open_kiosk_mode()
        </field>
        <field name="groups_id" eval="[(4, ref('hr_attendance.group_hr_attendance_manager'))]"/>
    </record>

    <!-- Menus -->

    <menuitem id="menu_hr_attendance_root" name="Attendances" sequence="205" groups="hr_attendance.group_hr_attendance_officer" web_icon="hr_attendance,static/description/icon.png"/>

    <menuitem id="menu_action_open_form" name="Kiosk Mode" action="open_kiosk_url" parent="menu_hr_attendance_root" sequence="10" groups="hr_attendance.group_hr_attendance_manager"/>

    <menuitem id="menu_hr_attendance_reporting" name="Reporting" action="hr_attendance_reporting" parent="menu_hr_attendance_root" sequence="15" groups="hr_attendance.group_hr_attendance_officer"/>

    <menuitem id="menu_hr_attendance_view_attendances" name="Overview" parent="menu_hr_attendance_root" sequence="5" groups="hr_attendance.group_hr_attendance_officer" action="hr_attendance_action"/>

    <menuitem id="menu_hr_attendance_view_attendances_management" name="Management" parent="menu_hr_attendance_root" sequence="6" groups="hr_attendance.group_hr_attendance_officer" action="hr_attendance_management_action"/>
</odoo>

```

## File: views\hr_department_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hr_department_view_kanban" model="ir.ui.view">
        <field name="name">hr.department.kanban.inherit</field>
        <field name="model">hr.department</field>
        <field name="inherit_id" ref="hr.hr_department_view_kanban"/>
        <field name="arch" type="xml">
            <data>
                <xpath expr="//div[hasclass('o_kanban_manage_reports')]" position="inside">
                    <div role="menuitem">
                        <a class="dropdown-item" name="%(hr_attendance_reporting)d" type="action" groups="hr_attendance.group_hr_attendance_officer">
                            Attendances
                        </a>
                    </div>
                </xpath>
            </data>
        </field>
    </record>
</odoo>

```

## File: views\hr_employee_view.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="view_employee_form_inherit_hr_attendance" model="ir.ui.view">
        <field name="name">hr.employee</field>
        <field name="model">hr.employee</field>
        <field name="inherit_id" ref="hr.view_employee_form"/>
        <field name="priority">110</field>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='button_box']" position="inside">
                <field name="attendance_state" invisible="1"/>
                <field name="hours_last_month" groups="hr_attendance.group_hr_attendance_officer" invisible="1"/>
                <button name="action_open_last_month_attendances"
                        class="oe_stat_button"
                        icon="fa-clock-o"
                        type="object"
                        groups="hr_attendance.group_hr_attendance_officer"
                        invisible="hours_last_month == 0"
                        help="Worked hours this month">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_value">
                            <field name="hours_last_month_display" widget="float_time"/> Hours
                        </span>
                        <span class="o_stat_text">
                            This Month
                        </span>
                    </div>
                </button>
                <button name="action_open_last_month_overtime"
                        class="oe_stat_button"
                        icon="fa-history"
                        type="object"
                        invisible="total_overtime == 0.0"
                        groups="hr_attendance.group_hr_attendance_officer">
                    <div class="o_stat_info">
                        <span class="o_stat_value text-success" invisible="total_overtime &lt; 0">
                            <field name="total_overtime" widget="float_time"/>
                        </span>
                        <span class="o_stat_value text-danger" invisible="total_overtime &gt;= 0">
                            <field name="total_overtime" widget="float_time"/>
                        </span>
                        <span class="o_stat_text">Extra Hours</span>
                    </div>
                </button>
            </xpath>
            <xpath expr="//group[@name='managers']" position="inside">
                <field name="attendance_manager_id" string="Attendance" widget="many2one_avatar_user"/>
            </xpath>
            <xpath expr="//group[@name='managers']" position="attributes">
                <attribute name="invisible">0</attribute>
            </xpath>
        </field>
    </record>

    <record id="hr_user_view_form" model="ir.ui.view">
        <field name="name">hr.user.preferences.view.form.attendance.inherit</field>
        <field name="model">res.users</field>
        <field name="inherit_id" ref="hr.res_users_view_form_profile"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='button_box']" position="inside">
                <field name="employee_ids" invisible="1"/>
                <button name="action_open_last_month_attendances"
                        class="oe_stat_button"
                        icon="fa-calendar"
                        type="object"
                        groups="base.group_user"
                        help="Worked hours this month">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_value">
                            <field name="hours_last_month_display" widget="float_time"/> Hours
                        </span>
                        <span class="o_stat_text">
                            This Month
                        </span>
                    </div>
                </button>
            </xpath>
            <xpath expr="//div[@name='button_box']" position="inside">
                <field name="display_extra_hours" invisible="1"/>
                <button name="action_open_last_month_overtime"
                        class="oe_stat_button"
                        icon="fa-history"
                        type="object"
                        invisible="total_overtime == 0.0 or not display_extra_hours"
                        groups="hr.group_hr_user"
                        help="Amount of extra hours">
                    <div class="o_stat_info">
                        <span class="o_stat_value text-success" invisible="total_overtime &lt; 0">
                            <field name="total_overtime" widget="float_time"/>
                        </span>
                        <span class="o_stat_value text-danger" invisible="total_overtime &gt;= 0">
                            <field name="total_overtime" widget="float_time"/>
                        </span>
                        <span class="o_stat_text">Extra Hours</span>
                    </div>
                </button>
            </xpath>
            <xpath expr="//group[@name='managers']" position="inside">
                <field name="attendance_manager_id" string="Attendance" widget="many2one_avatar_user"/>
            </xpath>
            <xpath expr="//group[@name='managers']" position="attributes">
                <attribute name="invisible">0</attribute>
            </xpath>
        </field>
    </record>

    <!-- employee kanban view specifically for hr_attendance (to check in/out) -->
    <record id="hr_employees_view_kanban" model="ir.ui.view">
        <field name="name">hr.employee.kanban</field>
        <field name="model">hr.employee</field>
        <field name="priority">99</field>
        <field name="arch" type="xml">
            <kanban create="false">
                <field name="attendance_state"/>
                <templates>
                    <t t-name="card" class="flex-row">
                        <aside>
                            <field name="avatar_128" widget="image" alt="Employee" class="mb-0"/>
                        </aside>
                        <main class="w-100 ms-2">
                            <div>
                                <div class="float-end" t-if="record.attendance_state.raw_value == 'checked_in'">
                                    <span class="fa fa-circle text-success me-1" role="img" aria-label="Available" title="Available"></span>
                                </div>
                                <div class="float-end" t-if="record.attendance_state.raw_value == 'checked_out'">
                                    <span class="fa fa-circle text-warning me-1"
                                          role="img" aria-label="Not available" title="Not available">
                                    </span>
                                </div>
                                <field class="fw-bolder" name="name"/>
                            </div>
                            <field t-if="record.job_id.raw_value" name="job_id"/>
                            <field t-if="record.work_location_id.raw_value" name="work_location_id"/>
                        </main>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="hr_employee_attendance_action_kanban" model="ir.actions.act_window">
        <field name="name">Employees</field>
        <field name="res_model">hr.employee.public</field>
        <field name="view_mode">kanban</field>
        <field name="view_id" ref="hr_employees_view_kanban"/>
        <field name="target">fullscreen</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a new employee
            </p><p>
                Add a few employees to be able to select an employee here and perform his check in / check out.
                To create employees go to the Employees menu.
            </p>
        </field>
    </record>

    <record id="view_employee_tree_inherit_leave" model="ir.ui.view">
        <field name="name">hr.employee.list.leave</field>
        <field name="model">hr.employee</field>
        <field name="inherit_id" ref="hr.view_employee_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='work_location_id']" position="after">
                <field name="attendance_manager_id" optional="hide" string="Attendance" widget="many2one_avatar_user"/>
            </xpath>
        </field>
    </record>

    <record id="hr_attendance_employee_simple_tree_view" model="ir.ui.view">
        <field name="name">hr.attendance.list</field>
        <field name="model">hr.attendance</field>
        <field name="arch" type="xml">
            <list sample="1">
                <field name="check_in"/>
                <field name="check_out"/>
                <field name="worked_hours" string="Work Hours" widget="float_time"/>
            </list>
        </field>
    </record>

    <record id="hr_attendance_validated_hours_employee_simple_tree_view" model="ir.ui.view">
        <field name="name">hr.attendance.list</field>
        <field name="model">hr.attendance</field>
        <field name="arch" type="xml">
            <list sample="1">
                <field name="check_in"/>
                <field name="check_out"/>
                <field name="worked_hours" string="Worked Hours" widget="float_time"/>
                <field name="validated_overtime_hours" string="Extra Hours" widget="float_time"/>
            </list>
        </field>
    </record>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.hr.attendance</field>
        <field name="model">res.config.settings</field>
        <field name="priority" eval="80"/>
        <field name="inherit_id" ref="base.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//form" position="inside">
                <app data-string="Attendances" string="Attendances" name="hr_attendance" groups="hr_attendance.group_hr_attendance_manager">
                    <block title="Modes" name="kiosk_mode_setting_container">
                        <setting string="Kiosk Mode" company_dependent="1" help="Define the way the user will be identified by the application.">
                            <field name="attendance_kiosk_mode" required="1" class="w-75"/>
                            <div>
                                <a href="https://www.odoo.com/documentation/master/applications/hr/attendances/hardware.html"
                                    target="_blank"><i class="fa fa-fw fa-arrow-right"/>Installation Manual</a>
                            </div>
                        </setting>
                        <setting string="Attendances from Backend" company_dependent="1" help="Allow Users to Check in/out from Odoo.">
                            <field name="attendance_from_systray" required="1"/>
                        </setting>
                        <setting string="Automatic Check-Out" company_dependent="1" help="Automatically Check-Out Employees based on their working schedule with an additional tolerance.">
                            <field name="auto_check_out"/>
                            <div invisible="not auto_check_out">
                                <span class="me-2">Tolerance</span><field name="auto_check_out_tolerance" class="text-center" style="width: 5ch;"/><span class="ms-2">Hours</span>
                            </div>
                        </setting>
                        <setting string="Absence Management" company_dependent="1" help="If checked, days not covered by an attendance will be visible in the Report.">
                            <field name="absence_management"/>
                        </setting>
                    </block>
                    <block title="Kiosk Settings">
                        <setting invisible="attendance_kiosk_mode == 'manual'" company_dependent="1" help="Define the camera used for the barcode scan.">
                            <field name="attendance_barcode_source" required="1"/>
                        </setting>
                        <setting string="Display Time" company_dependent="1" help="Choose how long the greeting message will be displayed.">
                            <field name="attendance_kiosk_delay" required="1" class="text-center" style="width: 10%; min-width: 4rem;"/><span> seconds</span>
                        </setting>
                        <setting title="Set PIN codes in the employee detail form (in HR Settings tab)." invisible="attendance_kiosk_mode == 'barcode'" help="Use PIN codes (defined on the Employee's profile) to check-in.">
                            <field name="attendance_kiosk_use_pin"/>
                        </setting>
                        <setting title="Kiosk Mode Adress" help="Use this url to access your kiosk mode from any device.">
                            <field name="attendance_kiosk_url" class="o_hr_kiosk_url_media w-100" style="width:100% !important;" widget="CopyClipboardURL"/>
                            <button name="regenerate_kiosk_key" type="object" string="Generate new URL" class="btn-link" icon="fa-refresh"/>
                        </setting>
                    </block>
                    <block title="Extra Hours" name="overtime_settings">
                        <setting company_dependent="1">
                            <div class="mt16">
                                <label for="overtime_company_threshold" class="o_form_label">
                                    Tolerance Time In Favor Of Company
                                </label>
                                <div class="text-muted">
                                    Allow a period of time (around working hours) where extra time will not be counted, in benefit of the company
                                </div>
                                <span class="me-2">Time Period</span><field name="overtime_company_threshold" class="text-center" style="width: 5ch;"/><span class="ms-2">Minutes</span>
                                <br/>
                                <br/>
                                <label for="overtime_employee_threshold" class="o_form_label">
                                    Tolerance Time In Favor Of Employee
                                </label>
                                <div class="text-muted">
                                    Allow a period of time (around working hours) where extra time will not be deducted, in benefit of the employee
                                </div>
                                <span class="me-2">Time Period </span><field name="overtime_employee_threshold" class="text-center" style="width: 5ch;"/><span> Minutes</span>
                            </div>
                        </setting>
                        <setting title="Display Extra Hours." string="Display Extra Hours" company_dependent="1" help="Display Extra Hours in Kiosk mode and on User profile.">
                            <field name="hr_attendance_display_overtime"/>
                        </setting>
                        <setting title="Extra Hours Validation" string="Extra Hours Validation" company_dependent="1" help="Can be converted as Time Off (cfr Time Off configuration).">
                            <field name="attendance_overtime_validation" widget="radio"/>
                        </setting>
                    </block>
                </app>
            </xpath>
        </field>
    </record>

    <record id="action_hr_attendance_settings" model="ir.actions.act_window">
        <field name="name">Settings</field>
        <field name="res_model">res.config.settings</field>
        <field name="view_mode">form</field>
        <field name="target">inline</field>
        <field name="context">{'module' : 'hr_attendance', 'bin_size': False}</field>
    </record>

    <menuitem id="hr_attendance.menu_hr_attendance_settings" name="Configuration" parent="menu_hr_attendance_root"
        sequence="99" action="action_hr_attendance_settings" groups="hr_attendance.group_hr_attendance_manager"/>
</odoo>

```

