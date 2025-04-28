# Odoo Module: hr_attendance

Category: Human Resources/Attendances

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
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
            'hr_attendance/static/src/**/*',
        ],
        'web.qunit_suite_tests': [
            'hr_attendance/static/tests/hr_attendance_mock_server.js',
        ],
        'web.qunit_mobile_suite_tests': [
            'hr_attendance/static/tests/hr_attendance_mock_server.js',
        ],
        'hr_attendance.assets_public_attendance': [
            # Front-end libraries
            ('include', 'web._assets_helpers'),
            ('include', 'web._assets_frontend_helpers'),
            'web/static/lib/jquery/jquery.js',
            'web/static/src/scss/pre_variables.scss',
            'web/static/lib/bootstrap/scss/_variables.scss',
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
            "hr_attendance/static/src/hr_attendance.scss",
            'hr_attendance/static/src/components/**/*',
            "web/static/src/views/fields/formatters.js",

            # Barcode reader utils
            "web/static/src/webclient/barcode/barcode_scanner.js",
            "web/static/src/webclient/barcode/barcode_scanner.xml",
            "web/static/src/webclient/barcode/barcode_scanner.scss",
            "web/static/src/webclient/barcode/crop_overlay.js",
            "web/static/src/webclient/webclient_layout.scss",
            "web/static/src/webclient/barcode/crop_overlay.xml",
            "web/static/src/webclient/barcode/crop_overlay.scss",
            "web/static/src/webclient/barcode/ZXingBarcodeDetector.js",
            "barcodes/static/src/components/barcode_scanner.js",
            "barcodes/static/src/components/barcode_scanner.xml",
            "barcodes/static/src/components/barcode_scanner.scss",
            "barcodes/static/src/barcode_service.js",

            # Kanban view mock
            "web/static/src/views/kanban/kanban_controller.scss",
            "web/static/src/search/search_panel/search_panel.scss",
            "web/static/src/search/control_panel/control_panel.scss",
        ]
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http, _
from odoo.http import request
from odoo.tools import float_round
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
                'employee_avatar': employee.image_256,
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

    @http.route('/hr_attendance/kiosk_mode_menu', auth='user', type='http')
    def kiosk_menu_item_action(self):
        # better use route with company_id suffix
        if request.env.user.user_has_groups("hr_attendance.group_hr_attendance_manager"):
            # Auto log out will prevent users from forgetting to log out of their session
            # before leaving the kiosk mode open to the public. This is a prevention security
            # measure.
            request.session.logout(keep_db=True)
            return request.redirect(request.env.company.attendance_kiosk_url)
        else:
            return request.not_found()

    @http.route('/hr_attendance/kiosk_mode_menu/<int:company_id>', auth='user', type='http')
    def kiosk_menu_item_action2(self, company_id):
        request.update_context(allowed_company_ids=[company_id])
        return self.kiosk_menu_item_action()

    @http.route('/hr_attendance/kiosk_keepalive', auth='user', type='json')
    def kiosk_keepalive(self):
        request.session.touch()
        return {}

    @http.route(["/hr_attendance/<token>"], type='http', auth='public', website=True, sitemap=True)
    def open_kiosk_mode(self, token):
        company = self._get_company(token)
        if not company:
            return request.not_found()
        else:
            employee_list = [{"id": e["id"],
                              "name": e["name"],
                              "avatar": image_data_uri(e["avatar_256"]),
                              "job": e["job_id"][1] if e["job_id"] else False,
                              "department": {"id": e["department_id"][0] if e["department_id"] else False,
                                             "name": e["department_id"][1] if e["department_id"] else False
                                             }
                              } for e in request.env['hr.employee'].sudo().search_read(domain=[('company_id', '=', company.id)],
                                                                                       fields=["id",
                                                                                               "name",
                                                                                               "avatar_256",
                                                                                               "job_id",
                                                                                               "department_id"])]
            departement_list = [{'id': dep["id"],
                                 'name': dep["name"],
                                 'count': dep["total_employee"]
                                 } for dep in request.env['hr.department'].sudo().search_read(domain=[('company_id', '=', company.id)],
                                                                                              fields=["id",
                                                                                                      "name",
                                                                                                      "total_employee"])]
            request.session.logout(keep_db=True)
            return request.render(
                'hr_attendance.public_kiosk_mode',
                {
                    'kiosk_backend_info': {
                        'token': token,
                        'company_id': company.id,
                        'company_name': company.name,
                        'employees': employee_list,
                        'departments': departement_list,
                        'kiosk_mode': company.attendance_kiosk_mode,
                        'barcode_source': company.attendance_barcode_source,
                        'lang': company.partner_id.lang,
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

    @http.route('/hr_attendance/systray_check_in_out', type="json", auth="user")
    def systray_attendance(self, latitude=False, longitude=False):
        employee = request.env.user.employee_id
        geo_ip_response = self._get_geoip_response(mode='systray',
                                                  latitude=latitude,
                                                  longitude=longitude)
        employee._attendance_action_change(geo_ip_response)
        return self._get_employee_info_response(employee)

    @http.route('/hr_attendance/attendance_user_data', type="json", auth="user")
    def user_attendance_data(self):
        employee = request.env.user.employee_id
        return self._get_user_attendance_data(employee)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-

from . import main

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

## File: models\hr_attendance.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import pytz

from collections import defaultdict
from datetime import datetime, timedelta
from operator import itemgetter
from pytz import timezone

from odoo.http import request
from odoo import models, fields, api, exceptions, _
from odoo.addons.resource.models.utils import Intervals
from odoo.osv.expression import AND, OR
from odoo.tools.float_utils import float_is_zero
from odoo.exceptions import AccessError
from odoo.tools import format_duration, format_time, format_datetime

def get_google_maps_url(latitude, longitude):
    return "https://maps.google.com?q=%s,%s" % (latitude, longitude)


class HrAttendance(models.Model):
    _name = "hr.attendance"
    _description = "Attendance"
    _order = "check_in desc"
    _inherit = "mail.thread"

    def _default_employee(self):
        return self.env.user.employee_id

    employee_id = fields.Many2one('hr.employee', string="Employee", default=_default_employee, required=True, ondelete='cascade', index=True)
    department_id = fields.Many2one('hr.department', string="Department", related="employee_id.department_id",
        readonly=True)
    check_in = fields.Datetime(string="Check In", default=fields.Datetime.now, required=True, tracking=True)
    check_out = fields.Datetime(string="Check Out", tracking=True)
    worked_hours = fields.Float(string='Worked Hours', compute='_compute_worked_hours', store=True, readonly=True)
    color = fields.Integer(compute='_compute_color')
    overtime_hours = fields.Float(string="Over Time", compute='_compute_overtime_hours', store=True)
    in_latitude = fields.Float(string="Latitude", digits=(10, 7), readonly=True)
    in_longitude = fields.Float(string="Longitude", digits=(10, 7), readonly=True)
    in_country_name = fields.Char(string="Country", help="Based on IP Address", readonly=True)
    in_city = fields.Char(string="City", readonly=True)
    in_ip_address = fields.Char(string="IP Address", readonly=True)
    in_browser = fields.Char(string="Browser", readonly=True)
    in_mode = fields.Selection(string="Mode",
                               selection=[('kiosk', "Kiosk"),
                                          ('systray', "Systray"),
                                          ('manual', "Manual")],
                               readonly=True,
                               default='manual')
    out_latitude = fields.Float(digits=(10, 7), readonly=True)
    out_longitude = fields.Float(digits=(10, 7), readonly=True)
    out_country_name = fields.Char(help="Based on IP Address", readonly=True)
    out_city = fields.Char(readonly=True)
    out_ip_address = fields.Char(readonly=True)
    out_browser = fields.Char(readonly=True)
    out_mode = fields.Selection(selection=[('kiosk', "Kiosk"),
                                           ('systray', "Systray"),
                                           ('manual', "Manual")],
                                readonly=True,
                                default='manual')

    def _compute_color(self):
        for attendance in self:
            if attendance.check_out:
                attendance.color = 1 if attendance.worked_hours > 16 else 0
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
                    "%s : (%s-%s)",
                    format_duration(attendance.worked_hours),
                    format_time(self.env, attendance.check_in, time_format=None, tz=tz, lang_code=self.env.lang),
                    format_time(self.env, attendance.check_out, time_format=None, tz=tz, lang_code=self.env.lang),
                )

    def _get_employee_calendar(self):
        self.ensure_one()
        return self.employee_id.resource_calendar_id or self.employee_id.company_id.resource_calendar_id

    @api.depends('check_in', 'check_out')
    def _compute_worked_hours(self):
        for attendance in self:
            if attendance.check_out and attendance.check_in and attendance.employee_id:
                calendar = attendance._get_employee_calendar()
                resource = attendance.employee_id.resource_id
                tz = timezone(calendar.tz)
                check_in_tz = attendance.check_in.astimezone(tz)
                check_out_tz = attendance.check_out.astimezone(tz)
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
        for attendance in self.filtered(lambda a: a.employee_id.company_id.hr_attendance_overtime and a.check_in):
            check_in_day_start = attendance._get_day_start_and_day(attendance.employee_id, attendance.check_in)
            if check_in_day_start[0] < datetime.combine(attendance.employee_id.company_id.overtime_start_date, datetime.min.time()):
                continue
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
                    # The employee usually doesn't work on that day
                    if not working_times[attendance_date]:
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
        self.env.add_to_compute(self._fields['overtime_hours'],
                                self.search([('employee_id', 'in', employees_worked_hours_to_compute)]))

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

    @api.returns('self', lambda value: value.id)
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
        compute='_compute_hours_last_month')
    overtime_ids = fields.One2many(
        'hr.attendance.overtime', 'employee_id', groups="hr_attendance.group_hr_attendance_officer,hr.group_hr_user")
    total_overtime = fields.Float(
        compute='_compute_total_overtime', compute_sudo=True)

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

    @api.depends('overtime_ids.duration', 'attendance_ids')
    def _compute_total_overtime(self):
        for employee in self:
            if employee.company_id.hr_attendance_overtime:
                employee.total_overtime = float_round(sum(employee.overtime_ids.mapped('duration')), 2)
            else:
                employee.total_overtime = 0

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
                ('employee_id', '=', employee.id),
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
                ('employee_id', '=', employee.id),
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
            "views": [[self.env.ref('hr_attendance.hr_attendance_employee_simple_tree_view').id, "tree"]],
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
            "name": _("Overtime"),
            "res_model": "hr.attendance.overtime",
            "views": [[False, "tree"]],
            "context": {
                "create": 0
            },
            "domain": [('employee_id', '=', self.id)]
        }

```

## File: models\hr_employee_base.py

```python
# -*- coding: utf-8 -*-
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
                                                            and e.hr_presence_state == "to_define")
        working_now_list = employee_to_check_working._get_employee_working_now()
        for employee in employees:
            if employee.attendance_state == "checked_out" and employee.hr_presence_state == "to_define" and \
                    employee.id in working_now_list:
                employee.hr_presence_state = "absent"
            elif employee.attendance_state == "checked_in":
                employee.hr_presence_state = "present"

    def _compute_presence_icon(self):
        res = super()._compute_presence_icon()
        # All employee must chek in or check out. Everybody must have an icon
        self.filtered(lambda employee: not employee.show_hr_icon_display).show_hr_icon_display = True
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
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api
from odoo.osv.expression import OR
import uuid
from werkzeug.urls import url_join


class ResCompany(models.Model):
    _inherit = 'res.company'

    def _default_company_token(self):
        return str(uuid.uuid4())

    hr_attendance_overtime = fields.Boolean(string="Count Extra Hours")
    overtime_start_date = fields.Date(string="Extra Hours Starting Date")
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
        search_domain = False  # Overtime to generate
        delete_domain = False  # Overtime to delete

        overtime_enabled_companies = self.filtered('hr_attendance_overtime')
        # Prevent any further logic if we are disabling the feature
        is_disabling_overtime = False
        # If we disable overtime
        if 'hr_attendance_overtime' in vals and not vals['hr_attendance_overtime'] and overtime_enabled_companies:
            delete_domain = [('company_id', 'in', overtime_enabled_companies.ids)]
            vals['overtime_start_date'] = False
            is_disabling_overtime = True

        start_date = vals.get('hr_attendance_overtime') and vals.get('overtime_start_date')
        # Also recompute if the threshold have changed
        if not is_disabling_overtime and (
            start_date or 'overtime_company_threshold' in vals or 'overtime_employee_threshold' in vals):
            for company in self:
                # If we modify the thresholds only
                if start_date == company.overtime_start_date and \
                    (vals.get('overtime_company_threshold') != company.overtime_company_threshold) or\
                    (vals.get('overtime_employee_threshold') != company.overtime_employee_threshold):
                    search_domain = OR([search_domain, [('employee_id.company_id', '=', company.id)]])
                # If we enabled the overtime with a start date
                elif not company.overtime_start_date and start_date:
                    search_domain = OR([search_domain, [
                        ('employee_id.company_id', '=', company.id),
                        ('check_in', '>=', start_date)]])
                # If we move the start date into the past
                elif start_date and company.overtime_start_date > start_date:
                    search_domain = OR([search_domain, [
                        ('employee_id.company_id', '=', company.id),
                        ('check_in', '>=', start_date),
                        ('check_in', '<=', company.overtime_start_date)]])
                # If we move the start date into the future
                elif start_date and company.overtime_start_date < start_date:
                    delete_domain = OR([delete_domain, [
                        ('company_id', '=', company.id),
                        ('date', '<', start_date)]])

        res = super().write(vals)
        if delete_domain:
            self.env['hr.attendance.overtime'].search(delete_domain).unlink()
        if search_domain:
            self.env['hr.attendance'].search(search_domain)._update_overtime()

        return res

    def _regenerate_attendance_kiosk_key(self):
        self.ensure_one()
        self.write({
            'attendance_kiosk_key': uuid.uuid4().hex
        })

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

    hr_attendance_overtime = fields.Boolean(
        string="Count Extra Hours", readonly=False)
    overtime_start_date = fields.Date(string="Extra Hours Starting Date", readonly=False)
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

    @api.model
    def get_values(self):
        res = super(ResConfigSettings, self).get_values()
        company = self.env.company
        res.update({
            'hr_attendance_overtime': company.hr_attendance_overtime,
            'overtime_start_date': company.overtime_start_date,
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
            'hr_attendance_overtime',
            'overtime_start_date',
            'overtime_company_threshold',
            'overtime_employee_threshold',
        ]
        if any(self[field] != company[field] for field in fields_to_check):
            company.write({field: self[field] for field in fields_to_check})

    def regenerate_kiosk_key(self):
        if self.user_has_groups("hr_attendance.group_hr_attendance_manager"):
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
            "views": [[self.env.ref('hr_attendance.hr_attendance_employee_simple_tree_view').id, "tree"]],
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
            "views": [[False, "tree"]],
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

## File: static\src\components\attendance_menu\attendance_menu.js

```javascript
/* @odoo-module */


import { Component, useState } from "@odoo/owl";
import { Dropdown } from "@web/core/dropdown/dropdown";
import { DropdownItem } from "@web/core/dropdown/dropdown_item";
import { deserializeDateTime } from "@web/core/l10n/dates";
import { registry } from "@web/core/registry";
import { useService } from "@web/core/utils/hooks";
import { isIosApp } from "@web/core/browser/feature_detection";
const { DateTime } = luxon;

export class ActivityMenu extends Component {
    static components = {Dropdown, DropdownItem};
    static props = [];
    static template = "hr_attendance.attendance_menu";

    setup() {
        this.rpc = useService("rpc");
        this.ui = useState(useService("ui"));
        this.userService = useService("user");
        this.employee = false;
        this.state = useState({
            checkedIn: false,
            isDisplayed: false
        });
        this.date_formatter = registry.category("formatters").get("float_time")
        // load data but do not wait for it to render to prevent from delaying
        // the whole webclient
        this.searchReadEmployee();
    }

    async searchReadEmployee(){
        const result = await this.rpc("/hr_attendance/attendance_user_data");
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
        // to close the dropdown
        document.body.click()
        // iOS app lacks permissions to call `getCurrentPosition`
        if (!isIosApp()) {
            navigator.geolocation.getCurrentPosition(
                async ({coords: {latitude, longitude}}) => {
                    await this.rpc("/hr_attendance/systray_check_in_out", {
                        latitude,
                        longitude
                    })
                    await this.searchReadEmployee()
                },
                async err => {
                    await this.rpc("/hr_attendance/systray_check_in_out")
                    await this.searchReadEmployee()
                },
                {
                    enableHighAccuracy: true,
                }
            )
        } else {
            await this.rpc("/hr_attendance/systray_check_in_out")
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
        <Dropdown position="'bottom-end'" beforeOpen.bind="searchReadEmployee" menuClass="`p-2 pb-3`">
            <t t-set-slot="toggler">
                <i class="fa fa-circle" t-attf-class="text-{{ this.state.checkedIn ? 'success' : 'danger' }}" role="img" aria-label="Attendance"/>
            </t>
            <t t-set-slot="default">
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

import { Component } from "@odoo/owl";

export class CardLayout extends Component {}

CardLayout.template = "hr_attendance.CardLayout";
CardLayout.props = {
    kioskModeClasses: { type: String, optional: true },
    slots: Object,
};
CardLayout.defaultProps = {
    kioskModeClasses: "",
};

```

## File: static\src\components\card_layout\card_layout.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

<t t-name="hr_attendance.CardLayout">
    <div class="o_hr_attendance_kiosk_mode_container o_attendance_background d-flex flex-column align-items-center justify-content-center h-100 text-center position-relative">
        <span class="o_hr_attendance_kiosk_backdrop position-absolute top-0 start-0 end-0 bottom-0 bg-black-25"/>
        <div t-attf-class="o_hr_attendance_kiosk_mode flex-grow-1 flex-md-grow-0 card pb-3 px-0 px-lg-5 {{props.kioskModeClasses}}" style="margin:15px;">
            <div class="card-body d-flex flex-column p-0 p-md-4">
                <t t-slot="default" />
            </div>
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

CheckInOut.template = "hr_attendance.CheckInOut";
CheckInOut.props = {
    checkedIn: Boolean,
    employeeId: Number,
    nextAction: String,
};

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

KioskGreetings.template = "hr_attendance.public_kiosk_greetings";
KioskGreetings.props = {
    employeeData : {type: Object},
    kioskReturn: {type: Function}
};

```

## File: static\src\components\greetings\greetings.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="hr_attendance.public_kiosk_greetings">
        <t t-if="this.attendance">
            <t t-call="hr_attendance.EmployeeBadge">
                <t t-set="employeeAvatar" t-value="this.employeeAvatar"/>
            </t>
            <div t-if="attendance.check_out" class="flex-grow-1">
                <h1 class="mt-5">Goodbye <t t-esc="this.employeeName"/>!</h1>
                <div class="alert alert-info fs-2 mx-3" role="status">
                    Checked out at <b><t t-esc="this.check_out_time"/></b>
                </div>
                <div class="alert alert-info fs-2 mx-3" role="status">
                    <b>Hours Today : <t t-esc="this.hoursToday"/></b>
                    <t t-if="this.overtimeToday">
                        <br/>
                        <b>
                            Extra hours today: <span t-esc="this.overtimeToday"/>
                        </b>
                    </t>
                </div>
                <t t-if="this.totalOvertime">
                    <div class="alert alert-info h3 mx-3" role="status">
                        Total extra hours: <span t-esc="this.totalOvertime"/>
                    </div>
                </t>
            </div>
            <div t-else="" class="flex-grow-1">
                <h1 class="mt-5 mb0">Welcome <t t-esc="this.employeeName"/>!</h1>
                <div class="alert alert-info fs-2 mx-3" role="status">
                    Checked in at <b><t t-esc="this.check_in_time"/></b>
                </div>
                <t t-if="this.hoursToday">
                    <div class="alert alert-info fs-2 mx-3" role="status">
                        <b>
                            Hours Previously Today: <span t-esc="this.hoursToday"/>
                        </b>
                    </div>
                </t>
            </div>
            <div class="flex-grow-1">
                <button class="o_hr_attendance_button_dismiss align-self-center btn btn-primary btn-lg px-5 py-3" t-on-click="this.props.kioskReturn">
                    <span class="fs-2" t-if="attendance.check_out">Goodbye</span>
                    <span class="fs-2" t-else="">OK</span>
                </button>
            </div>
        </t>
        <t t-else="">
            <div class="flex-grow-1">
                <div class="alert alert-warning mt-5 mx-3" role="alert">
                    <h4 class="alert-heading">Invalid request</h4>
                    <p>Please return to the main menu.</p>
                </div>
            </div>
            <div class="flex-grow-1">
                <button class="o_hr_attendance_button_dismiss btn btn-primary btn-lg fs-2 px-5 py-3" t-on-click="this.props.kioskReturn">
                    <i class="oi oi-chevron-left me-2"/>
                    <span class="fs-2">Go back</span>
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

export class KioskBarcodeScanner extends BarcodeScanner {
    get facingMode() {
        if (this.props.barcodeSource == "front") {
            return "user";
        }
        return super.facingMode;
    }
}
KioskBarcodeScanner.props = {
    ...BarcodeScanner.props,
    barcodeSource: String,
};

```

## File: static\src\components\manual_selection\manual_selection.js

```javascript
/** @odoo-module **/

import {Component, useState} from "@odoo/owl";
import { Dropdown } from "@web/core/dropdown/dropdown";
import { DropdownItem } from "@web/core/dropdown/dropdown_item";

export class KioskManualSelection extends Component {
    setup() {
        this.state = useState({
            displayedEmployees : this.props.employees,
            searchInput: ""
        });
        this.displayedEmployees = this.props.employees
    }
    onDepartementClick(dep_id){
        if (dep_id){
            this.state.displayedEmployees = this.props.employees.filter(item => item.department.id === dep_id)
        }else{
            this.state.displayedEmployees = this.props.employees
        }
    }

    onSearchInput(ev) {
        const searchInput = ev.target.value;
        if (searchInput.length){
            this.state.displayedEmployees = this.props.employees.filter(item => item.name.toLowerCase().includes(searchInput.toLowerCase()))
        }else{
            this.state.displayedEmployees = this.props.employees
        }
    }
}

KioskManualSelection.template = "hr_attendance.public_kiosk_manual_selection";
KioskManualSelection.components = {
    Dropdown,
    DropdownItem
}
KioskManualSelection.props = {
    employees : {type : Array},
    displayBackButton : {type : Boolean},
    departments: {type : Array},
    onSelectEmployee : {type : Function},
    onClickBack : {type : Function}
};

```

## File: static\src\components\manual_selection\manual_selection.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="hr_attendance.employeeListElement">
        <div class="d-flex position-relative py-2 overflow-visible o_treeEntry">
    <div class="o_media_left position-relative">
        <div class="o_media_object d-block rounded border border-info">
            <img t-attf-src="{{avatar}}" height="80" width="80"/>
        </div>
    </div>
    <div class="d-flex flex-grow-1 align-items-center justify-content-between position-relative px-3">
        <div class="d-flex flex-column align-items-start">
            <h5 class="o_media_heading m-0"><t t-esc="employeeName"/></h5>
            <small class="text-muted fw-bold"><t t-esc="jobTitle"/></small>
        </div>
    </div>
    </div>
    </t>
    <t t-name="hr_attendance.public_kiosk_manual_selection">
        <t t-if="!this.props.displayBackButton">
            <button t-on-click="() => this.props.onClickBack()" class="o_hr_attendance_back_button o_hr_attendance_back_button_md btn btn-secondary d-none d-md-inline-flex align-items-center position-absolute top-0 start-0 rounded-circle m-5">
            <i class="oi fa-2x fa-fw oi-chevron-left me-1" role="img" aria-label="Go back" title="Go back"/>
        </button>
        </t>
            <div class="o_kanban_view o_view_controller o_action">
                <div class="o_control_panel d-flex flex-column gap-3 gap-lg-1 px-3 pt-2 pb-3" style="top: 0px;">
                    <div class="o_control_panel_main d-flex flex-wrap flex-lg-nowrap justify-content-between align-items-lg-start gap-3 flex-grow-1">
                       <div class="o_cp_searchview d-flex input-group">
                           <div class="o_searchview form-control d-print-contents d-flex align-items-center py-1 border-end-0">
                               <i class="o_searchview_icon d-print-none oi oi-search me-2" aria-label="Search..." title="Search..."/>
                               <div class="o_searchview_input_container d-flex flex-grow-1 flex-wrap gap-1" style="position: relative;">
                                   <input t-on-input="onSearchInput" type="text" class="o_searchview_input d-print-none flex-grow-1 w-auto border-0" placeholder="Search..."/>
                               </div>
                           </div>
                       </div>
                    </div>
                </div>
                <div class="o_content o_component_with_search_panel">
                <div class="o_search_panel flex-grow-0 flex-shrink-0 h-100 pb-5 bg-view overflow-auto pe-1 ps-3" style="max-height:500px;">
                    <section class="o_search_panel_section o_search_panel_category">
                        <header class="o_search_panel_section_header pt-4 pb-2 text-uppercase cursor-default">
                            <i class="fa fa-users o_search_panel_section_icon text-odoo me-2"/>
                            <b>Department</b>
                        </header>
                        <ul class="list-group d-block o_search_panel_field">
                            <li class="o_search_panel_category_value list-group-item py-1 cursor-pointer border-0 pe-0 ps-0">
                                <header class="list-group-item list-group-item-action d-flex align-items-center px-0 py-lg-0 border-0 text-900 fw-bold">
                                    <div t-on-click="() => this.onDepartementClick(false)" class="o_search_panel_label d-flex align-items-center overflow-hidden w-100 cursor-pointer mb-0 o_with_counters">
                                        <span class="o_search_panel_label_title text-truncate">All</span>
                                    </div>
                                </header>
                            </li>
                            <t t-foreach="this.props.departments" t-as="dep" t-key="dep.id">
                                <li class="o_search_panel_category_value list-group-item py-1 cursor-pointer border-0 pe-0 ps-0">
                                <header class="list-group-item list-group-item-action d-flex align-items-center px-0 py-lg-0 border-0">
                                    <div t-on-click="() => this.onDepartementClick(dep.id)" class="o_search_panel_label d-flex align-items-center overflow-hidden w-100 cursor-pointer mb-0 o_with_counters">
                                        <span class="o_search_panel_label_title text-truncate"><t t-esc="dep.name"/></span>
                                    </div>
                                    <small class="o_search_panel_counter text-muted mx-2 fw-bold"><t t-esc="dep.count"/></small>
                                </header>
                                </li>
                            </t>
                        </ul>
                    </section>
                </div>
                <div class="o_kanban_renderer o_renderer d-flex o_kanban_ungrouped align-content-start flex-wrap justify-content-start" style="max-width:1050px;max-height:500px;overflow-y:scroll;">
                <t t-foreach="this.state.displayedEmployees" t-as="employee" t-key="employee.id">
                <div t-on-click="() => this.props.onSelectEmployee(employee.id)" role="article" class="o_kanban_record d-flex oe_kanban_global_click flex-md-shrink-1 flex-shrink-0">
                    <div class="oe_kanban_global_click">
                        <div class="o_kanban_image">
                            <img alt="Employee" loading="lazy" t-attf-src="{{employee.avatar}}"/>
                        </div>
                        <div class="oe_kanban_details">
                            <div id="textbox">
                                <strong><span><t t-esc="employee.name"/></span></strong>
                            </div>
                            <ul>
                                <li t-if="employee.job"><span><t t-esc="employee.job"/></span></li>
                            </ul>
                        </div>
                    </div>
                </div>
                </t>
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

KioskPinCode.props = {
    employeeData : {type: Object},
    onClickBack: {type: Function},
    onPinConfirm: {type: Function}
}

KioskPinCode.template = "hr_attendance.KioskPinConfirm";

```

## File: static\src\components\pin_code\pin_code.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

<t t-name="hr_attendance.EmployeeBadge">
    <div t-if="employeeAvatar" class="o_attendance_background d-flex align-items-end justify-content-center flex-grow-1 pt-5 pt-md-4 bg-odoo">
        <img class="o_hr_attendance_employee_badge img rounded-circle" t-attf-src="data:image/png;base64,{{employeeAvatar}}" height="80"/>
    </div>
</t>

<t t-name="hr_attendance.KioskPinConfirm">
        <button t-on-click="() => this.props.onClickBack()" class="o_hr_attendance_back_button btn btn-block btn-secondary btn-lg d-block d-md-none py-5">
            <i class="oi oi-chevron-left me-2"/> Go back
        </button>
        <t t-call="hr_attendance.EmployeeBadge">
            <t t-set="employeeAvatar" t-value="this.props.employeeData.employee_avatar"/>
        </t>
        <button t-on-click="() => this.props.onClickBack()" class="o_hr_attendance_back_button o_hr_attendance_back_button_md btn btn-secondary d-none d-md-inline-flex align-items-center position-absolute top-0 start-0 rounded-circle m-5">
            <i class="oi fa-2x fa-fw oi-chevron-left me-1" role="img" aria-label="Go back" title="Go back"/>
        </button>

        <div t-if="this.props.employeeData" class="flex-grow-1">
            <h1 class="mt-5 mb8" t-esc="this.props.employeeData.employee_name"/>
            <h3 class="mt8 mb24"><t t-if="!checkedIn">Welcome!</t><t t-else="">Want to check out?</t></h3>
            <h3 class="mt-4 mb0 text-muted">Please enter your PIN to <b t-if="checkedIn">check out</b><b t-else="">check in</b></h3>
            <div class="row">
                <div class="col-md-8 offset-md-2 o_hr_attendance_pin_pad">
                    <div class="row g-0" >
                        <input t-att-value="state.codePin" class="o_hr_attendance_PINbox col-12 mb8 mt8 border-0 bg-white fs-1 text-center" type="password" disabled="true"/>
                    </div>
                    <div class="row g-0">
                        <t t-foreach="padButtons" t-as="btn" t-key="btn[0]">
                            <div class="col-4 p-1">
                                <a href="#" t-on-click="() => this.onClickPadButton(btn[0])" t-attf-class="o_hr_attendance_PINbox_button btn {{btn[1]? btn[1] : 'btn-secondary border'}} btn-block btn-lg d-flex align-items-center justify-content-center">
                                    <t t-esc="btn[0]"/>
                                </a>
                            </div>
                        </t>
                    </div>
                </div>
            </div>
        </div>
        <div t-else="" class="alert alert-danger mx-3" role="alert">
            <h4 class="alert-heading">Error: could not find corresponding employee.</h4>
            <p>Please return to the main menu.</p>
        </div>
        <a role="button" class="oe_attendance_sign_in_out" aria-label="Sign out" title="Sign out"/>
</t>

</templates>

```

## File: static\src\public_kiosk\public_kiosk_app.js

```javascript
/** @odoo-module **/

import {App, whenReady, Component, useState} from "@odoo/owl";
import { CardLayout } from "@hr_attendance/components/card_layout/card_layout";
import { KioskManualSelection } from "@hr_attendance/components/manual_selection/manual_selection";
import { makeEnv, startServices } from "@web/env";
import { templates } from "@web/core/assets";
import { isIosApp } from "@web/core/browser/feature_detection";
import { _t } from "@web/core/l10n/translation";
import { MainComponentsContainer } from "@web/core/main_components_container";
import { useService, useBus } from "@web/core/utils/hooks";
import { url } from "@web/core/utils/urls";
import {KioskGreetings} from "@hr_attendance/components/greetings/greetings";
import {KioskPinCode} from "@hr_attendance/components/pin_code/pin_code";
import {KioskBarcodeScanner} from "@hr_attendance/components/kiosk_barcode/kiosk_barcode";

class kioskAttendanceApp extends Component{
    static props = {
        token: {type : String},
        companyId: {type : Number},
        companyName: {type : String},
        employees: {type : Array},
        departments: {type : Array},
        kioskMode: {type : String},
        barcodeSource: {type : String}
    };
    static components = {
        KioskBarcodeScanner,
        CardLayout,
        KioskManualSelection,
        KioskGreetings,
        KioskPinCode,
        MainComponentsContainer,
    };

    setup() {
        this.rpc = useService("rpc");
        this.barcode = useService("barcode");
        this.notification = useService("notification");
        this.companyImageUrl = url("/web/binary/company_logo", {
            company: this.props.companyId,
        });
        this.lockScanner = false;
        if (this.props.kioskMode !== 'manual'){
            useBus(this.barcode.bus, "barcode_scanned", (ev) => this.onBarcodeScanned(ev.detail.barcode));
            this.state = useState({active_display: "main"});
            this.manualKioskMode = false
        }else{
            this.manualKioskMode = true
            this.state = useState({active_display: "manual"});
        }
    }

    switchDisplay(screen) {
        const displays = ["main", "greet", "manual", "pin"]
        if (displays.includes(screen)){
            this.state.active_display = screen;
        }else{
            this.state.active_display = "main";
        }
    }

    async kioskConfirm(employeeId){
        const employee = await this.rpc('attendance_employee_data',
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
        if (this.props.kioskMode !== 'manual'){
            this.switchDisplay('main')
        }else{
            this.switchDisplay('manual')
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
                        const result = await this.rpc(route, {
                            ...params,
                            latitude,
                            longitude,
                        });
                        resolve(result);
                    },
                    async (err) => {
                        const result = await this.rpc(route, {
                            ...params
                        });
                        resolve(result);
                    },
                    { enableHighAccuracy: true }
                );
            });
        }
        else {
            return this.rpc(route, {...params})
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
        const result = await this.rpc('attendance_barcode_scanned',
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
}

kioskAttendanceApp.template = "hr_attendance.public_kiosk_app";

export async function createPublicKioskAttendance(document, kiosk_backend_info) {
    await whenReady();
    const env = makeEnv();
    await startServices(env);
    const app = new App(kioskAttendanceApp, {
        templates,
        env: env,
        props:
            {
                token : kiosk_backend_info.token,
                companyId: kiosk_backend_info.company_id,
                companyName: kiosk_backend_info.company_name,
                employees: kiosk_backend_info.employees,
                departments: kiosk_backend_info.departments,
                kioskMode: kiosk_backend_info.kiosk_mode,
                barcodeSource: kiosk_backend_info.barcode_source,
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
    <h2 class="mb-2"><small>Welcome to</small></h2>
    <img t-att-src="companyImageUrl" alt="Company Logo" class="o_hr_attendance_kiosk_company_image align-self-center img img-fluid mb-3" width="200"/>
</t>

<t t-name="hr_attendance.public_kiosk_app">
    <MainComponentsContainer/>
    <CardLayout>
        <t t-if="this.state.active_display === 'main'">
            <t t-call="hr_attendance.companyHeader">
                <t t-set="companyImageUrl" t-value="companyImageUrl"/>
                <t t-set="companyName" t-value="this.props.companyName"/>
            </t>
            <t t-if="this.props.kioskMode !== 'manual'">
                <div class="col-md-5 mt-5 mb-5 mb-md-0 align-self-center">
                    <KioskBarcodeScanner barcodeSource="this.props.barcodeSource" onBarcodeScanned="(ev) => this.onBarcodeScanned(ev)"/>
                    <h6 class="mt-2 text-muted">Scan your badge</h6>
                </div>
            </t>
            <t t-if="this.props.kioskMode !== 'barcode'">
                <div class="mt-5 align-self-center">
                    <button t-on-click="() => this.switchDisplay('manual')"  class="o_hr_attendance_button_employees btn btn-link">
                        Identify Manually
                    </button>
                </div>
            </t>
        </t>
        <t t-if="this.state.active_display === 'manual'">
            <t t-call="hr_attendance.companyHeader">
                <t t-set="companyImageUrl" t-value="companyImageUrl"/>
                <t t-set="companyName" t-value="this.props.companyName"/>
            </t>
            <KioskManualSelection employees="this.props.employees" displayBackButton="this.manualKioskMode" departments="this.props.departments" onSelectEmployee="(e) => this.kioskConfirm(e)" onClickBack="() => this.kioskReturn()"/>
        </t>
        <t t-if="this.state.active_display === 'greet'">
            <KioskGreetings employeeData="this.employeeData" kioskReturn="() => this.kioskReturn(true)"/>
        </t>
        <t t-if="this.state.active_display === 'pin'">
            <KioskPinCode employeeData="this.employeeData" onPinConfirm="(id, pin) => this.onManualSelection(id, pin)" onClickBack="() => this.kioskReturn()"/>
        </t>
    </CardLayout>
</t>
</templates>

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
                <meta name="apple-mobile-web-app-capable" content="yes"/>
                <meta name="mobile-web-app-capable" content="yes"/>
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
            <body class="o_web_client o_hr_attendance_kiosk_body">
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
        <field name="name">hr.attendance.overtime.tree</field>
        <field name="model">hr.attendance.overtime</field>
        <field name="arch" type="xml">
            <tree edit="0" create="0">
                <field name="date"/>
                <field name="employee_id"/>
                <field name="duration" widget="float_time"/>
            </tree>
        </field>
    </record>

    <record id="view_attendance_overtime_search" model="ir.ui.view">
        <field name="name">hr.attendance.overtime.search</field>
        <field name="model">hr.attendance.overtime</field>
        <field name="arch" type="xml">
            <search>
                <field name="employee_id"/>
                <field name="duration" filter_domain="[('duration', '>=', self)]"/>
            </search>
        </field>
    </record>

    <record id="hr_attendance_overtime_action" model="ir.actions.act_window">
        <field name="name">Extra Hours</field>
        <field name="res_model">hr.attendance.overtime</field>
        <field name="view_mode">tree</field>
    </record>
</odoo>

```

## File: views\hr_attendance_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- views -->

    <record id="view_attendance_tree" model="ir.ui.view">
        <field name="name">hr.attendance.tree</field>
        <field name="model">hr.attendance</field>
        <field name="arch" type="xml">
            <tree string="Employee attendances" decoration-success="color == 10" decoration-danger="color == 1" sample="1" duplicate="false">
                <field name="employee_id" widget="many2one_avatar_user"/>
                <field name="check_in"/>
                <field name="check_out" options="{}"/>
                <field name="worked_hours" string="Work Hours" widget="float_time"/>
                <field name="overtime_hours" string="Over Time" optional="show" widget="float_time"/>
                <field name="color" column_invisible="1"/>
                <field name="in_latitude" optional="hidden"/>
                <field name="in_longitude" optional="hidden"/>
                <field name="out_latitude" optional="hidden"/>
                <field name="out_longitude" optional="hidden"/>
                <field name="in_country_name" optional="hidden"/>
                <field name="out_country_name" optional="hidden"/>
                <field name="in_mode" optional="hidden"/>
                <field name="out_mode" optional="hidden"/>
                <field name="in_city" optional="hidden"/>
                <field name="out_city" optional="hidden"/>
                <field name="create_uid" optional="hidden"/>
                <field name="write_uid" optional="hidden"/>
                <field name="write_date" optional="hidden"/>
            </tree>
        </field>
    </record>

    <record id="view_hr_attendance_kanban" model="ir.ui.view">
        <field name="name">hr.attendance.kanban</field>
        <field name="model">hr.attendance</field>
        <field name="arch" type="xml">
            <kanban class="o_kanban_mobile" sample="1">
                <field name="employee_id"/>
                <field name="check_in"/>
                <field name="check_out"/>
                <templates>
                    <t t-name="kanban-box">
                        <div t-attf-class="oe_kanban_global_click">
                            <div class="o_kanban_record_title">
                                <field name="employee_id" widget="many2one_avatar_user" options="{'display_avatar_name': True}" class="fs-5 fw-bold"/>
                            </div>
                            <hr class="mt4 mb8"/>
                            <div class="o_kanban_record_subtitle">
                                <i class="fa fa-calendar" aria-label="Period" role="img" title="Period"></i>
                                <t t-esc="record.check_in.value"/>
                                - <t t-esc="record.check_out.value"/>
                            </div>
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
                <sheet>
                    <group>
                        <group colspan="2">
                            <group col="1">
                                <field name="employee_id" widget="many2one_avatar_user"/>
                                <field name="check_in" options="{'rounding': 0}"/>
                                <field name="check_out" options="{'rounding': 0}"/>
                            </group>
                            <group col="2">
                                <field name="worked_hours" widget="float_time"/>
                                <field name="overtime_hours" widget="float_time" string="Extra Hours"/>
                            </group>
                        </group>
                        <separator string="Check In"/>
                        <group colspan="2">
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
                        <separator string="Check Out" invisible="not check_out"/>
                        <group colspan="2" invisible="not check_out">
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
                <div class="oe_chatter">
                    <field name="message_ids"/>
                </div>
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
                <field name="overtime_hours" type="measure" widget="float_time"/>
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
                        domain="['|', ('worked_hours', '&gt;=', 16), '&amp;', ('check_out', '=', False), ('check_in', '&lt;=',  (context_today() - datetime.timedelta(days=1)).strftime('%Y-%m-%d'))]"                />
                <separator/>
                <filter string="Check In" name="check_in_filter" date="check_in"/>
                <filter string="Last 7 days" name="last_week" domain="[(
                    'check_in','&gt;=', (
                        context_today() + datetime.timedelta(days=-7)
                        )
                    )]"/>
                <filter string="Last 3 Months" invisible="1" name="last_three_months" domain="[(
                    'check_in','&gt;=', (
                        context_today() + datetime.timedelta(days=-90)
                        )
                    )]"/>
                <group expand="0" string="Group By">
                    <filter string="Check In" name="groupby_name" context="{'group_by': 'check_in:week'}"/>
                    <filter string="Employee" name="employee" context="{'group_by': 'employee_id'}"/>
                    <filter string="Check Out" name="groupby_check_out" context="{'group_by': 'check_out'}"/>
                </group>
            </search>
        </field>
    </record>

    <!-- actions -->

    <record id="hr_attendance_action" model="ir.actions.act_window">
        <field name="name">Attendances</field>
        <field name="res_model">hr.attendance</field>
        <field name="view_mode">tree,form</field>
        <field name="context">
            {
                "search_default_employee": 1
            }
        </field>
        <field name="search_view_id" ref="hr_attendance_view_filter"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                No attendance records found
            </p><p>
                The attendance records of your employees will be displayed here.
            </p>
        </field>
    </record>

    <record id="hr_attendance_reporting" model="ir.actions.act_window">
        <field name="name">Attendances</field>
        <field name="res_model">hr.attendance</field>
        <field name="view_mode">graph,pivot</field>
        <field name="search_view_id" ref="hr_attendance_view_filter"/>
        <field name="context">
            {
                "search_default_groupby_name" : 1,
                "search_default_employee": 1,
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

    <record id="hr_attendance_action_greeting_message" model="ir.actions.client">
        <field name="name">Message</field>
        <field name="tag">hr_attendance_greeting_message</field>
    </record>

    <record model="ir.actions.server" id="open_kiosk_url">
        <field name="name">Open Kiosk Url</field>
        <field name="model_id" ref="hr_attendance.model_res_company"/>
        <field name="binding_model_id" ref="hr_attendance.model_res_company"/>
        <field name="state">code</field>
        <field name="code">
            action = model._action_open_kiosk_mode()
        </field>
        <field name="groups_id" eval="[(4, ref('hr_attendance.group_hr_attendance_manager'))]"/>
    </record>

    <!-- Menus -->

    <menuitem id="menu_hr_attendance_root" name="Attendances" sequence="205" groups="hr_attendance.group_hr_attendance_officer" web_icon="hr_attendance,static/description/icon.png"/>

    <menuitem id="menu_hr_attendance_kiosk_no_user_mode" name="Kiosk Mode" parent="menu_hr_attendance_root" sequence="10" groups="hr_attendance.group_hr_attendance_manager" action="open_kiosk_url"/>

    <menuitem id="menu_hr_attendance_reporting" name="Reporting" parent="menu_hr_attendance_root" sequence="15" groups="hr_attendance.group_hr_attendance_officer" action="hr_attendance_reporting"/>

    <menuitem id="menu_hr_attendance_view_attendances" name="Overview" parent="menu_hr_attendance_root" sequence="5" groups="hr_attendance.group_hr_attendance_officer" action="hr_attendance_action"/>
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
                    <a role="menuitem" class="dropdown-item" name="%(hr_attendance_reporting)d" type="action" groups="hr_attendance.group_hr_attendance_officer">
                        Attendances
                    </a>
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
                        groups="base.group_user"
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
                <field name="hours_today"/>
                <field name="total_overtime"/>
                <field name="id"/>
                <templates>
                    <t t-name="kanban-box">
                    <div class="oe_kanban_global_click">
                        <div class="o_kanban_image">
                            <img t-att-src="kanban_image('hr.employee.public', 'avatar_128', record.id.raw_value)" alt="Employee"/>
                        </div>
                        <div class="oe_kanban_details">
                            <div id="textbox">
                                <div class="float-end" t-if="record.attendance_state.raw_value == 'checked_in'">
                                    <span id="oe_hr_attendance_status" class="fa fa-circle text-success me-1" role="img" aria-label="Available" title="Available"></span>
                                </div>
                                <div class="float-end" t-if="record.attendance_state.raw_value == 'checked_out'">
                                    <span id="oe_hr_attendance_status" class="fa fa-circle text-warning me-1"
                                          role="img" aria-label="Not available" title="Not available">
                                    </span>
                                </div>
                                <strong>
                                    <field name="name"/>
                                </strong>
                            </div>
                            <ul>
                                <li t-if="record.job_id.raw_value"><field name="job_id"/></li>
                                <li t-if="record.work_location_id.raw_value"><field name="work_location_id"/></li>
                            </ul>
                        </div>
                    </div>
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
        <field name="name">hr.employee.tree.leave</field>
        <field name="model">hr.employee</field>
        <field name="inherit_id" ref="hr.view_employee_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='work_location_id']" position="after">
                <field name="attendance_manager_id" optional="hide" string="Attendance" widget="many2one_avatar_user"/>
            </xpath>
        </field>
    </record>

    <record id="hr_attendance_employee_simple_tree_view" model="ir.ui.view">
        <field name="name">hr.attendance.tree</field>
        <field name="model">hr.attendance</field>
        <field name="arch" type="xml">
            <tree sample="1">
                <field name="check_in"/>
                <field name="check_out"/>
                <field name="worked_hours" string="Work Hours" widget="float_time"/>
            </tree>
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
                        </setting>
                        <setting string="Attendances from Backend" company_dependent="1" help="Allow Users to Check in/out from Odoo.">
                            <field name="attendance_from_systray" required="1"/>
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
                        <setting title="Kiosk Mode Adress" help="Use this url to access your kiosk mode from any device. Warning, anybody with the link can access your kiosk.">
                            <field name="attendance_kiosk_url" class="o_hr_kiosk_url_media w-100" style="width:100% !important;" widget="CopyClipboardURL"/>
                            <br/>
                            <span>
                                If your address is compromised, you can refresh it to generate a new one.
                            </span>
                            <br/>
                            <button name="regenerate_kiosk_key" type="object" string="Generate a new Kiosk Mode URL" class="btn-link" icon="fa-refresh"/>
                        </setting>
                    </block>
                    <block title="Extra Hours" name="overtime_settings">
                        <setting title="Activate the count of employees' extra hours." string="Count of Extra Hours" company_dependent="1" help="Compare attendance with working hours set on employee.">
                            <field name="hr_attendance_overtime"/>
                            <div class="mt16" invisible="not hr_attendance_overtime" required="hr_attendance_overtime">
                                <div class="mt16 row" title="Count of extra hours is considered from this date. Potential extra hours prior to this date are not considered.">
                                    <label for="overtime_start_date" string="Start from" class="o_light_label col-lg-3"/>
                                    <field name="overtime_start_date" class="col-lg-3 w-75" required="hr_attendance_overtime" />
                                </div>
                                <br/>
                                <label for="overtime_company_threshold" class="o_form_label">
                                    Tolerance Time In Favor Of Company
                                </label>
                                <div class="text-muted">
                                    Allow a period of time (around working hours) where extra time will not be counted, in benefit of the company
                                </div>
                                <span>Time Period </span><field name="overtime_company_threshold" class="text-center"
                                    required="hr_attendance_overtime" style="width: 10%; min-width: 4rem;"/><span> Minutes</span>
                                <br/>
                                <br/>
                                <label for="overtime_employee_threshold" class="o_form_label">
                                    Tolerance Time In Favor Of Employee
                                </label>
                                <div class="text-muted">
                                    Allow a period of time (around working hours) where extra time will not be deducted, in benefit of the employee
                                </div>
                                <span>Time Period </span><field name="overtime_employee_threshold" class="text-center"
                                    required="hr_attendance_overtime" style="width: 10%; min-width: 4rem;"/><span> Minutes</span>
                            </div>
                        </setting>
                        <setting title="Display Extra Hours." string="Display Extra Hours" invisible="not hr_attendance_overtime" company_dependent="1" help="Display Extra Hours in Kiosk mode and on User profile.">
                            <field name="hr_attendance_display_overtime"/>
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

