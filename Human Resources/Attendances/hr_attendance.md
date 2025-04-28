# Odoo Module: hr_attendance

Category: Human Resources/Attendances

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models
from . import report

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
        'report/hr_attendance_report_views.xml',
        'views/hr_department_view.xml',
        'views/hr_employee_view.xml',
        'views/res_config_settings_views.xml',
    ],
    'demo': [
        'data/hr_attendance_demo.xml'
    ],
    'installable': True,
    'application': True,
    'assets': {
        'web.assets_backend': [
            'hr_attendance/static/src/**/*',
            'hr_attendance/static/src/xml/**/*',
        ],
        'web.qunit_suite_tests': [
            ('after', 'web/static/tests/legacy/views/kanban_tests.js', 'hr_attendance/static/tests/hr_attendance_tests.js'),
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http
from odoo.http import request


class HrAttendance(http.Controller):
    @http.route('/hr_attendance/kiosk_keepalive', auth='user', type='json')
    def kiosk_keepalive(self):
        request.session.touch()
        return {}

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
    <data noupdate="True">
        <record id="base.user_demo" model="res.users">
            <field name="groups_id" eval="[(4, ref('hr_attendance.group_hr_attendance_user'))]"/>
        </record>

        <record id="hr.employee_al" model="hr.employee">
            <field name="pin">0000</field>
            <field name="barcode">123</field>
        </record>

        <record id="hr.employee_admin" model="hr.employee">
            <field name="pin">0000</field>
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

from collections import defaultdict
from datetime import datetime, timedelta
from operator import itemgetter

import pytz
from odoo import models, fields, api, exceptions, _
from odoo.tools import format_datetime
from odoo.osv.expression import AND, OR
from odoo.tools.float_utils import float_is_zero
from odoo.exceptions import AccessError


class HrAttendance(models.Model):
    _name = "hr.attendance"
    _description = "Attendance"
    _order = "check_in desc"

    def _default_employee(self):
        return self.env.user.employee_id

    employee_id = fields.Many2one('hr.employee', string="Employee", default=_default_employee, required=True, ondelete='cascade', index=True)
    department_id = fields.Many2one('hr.department', string="Department", related="employee_id.department_id",
        readonly=True)
    check_in = fields.Datetime(string="Check In", default=fields.Datetime.now, required=True)
    check_out = fields.Datetime(string="Check Out")
    worked_hours = fields.Float(string='Worked Hours', compute='_compute_worked_hours', store=True, readonly=True)

    def name_get(self):
        result = []
        for attendance in self:
            if not attendance.check_out:
                result.append((attendance.id, _("%(empl_name)s from %(check_in)s") % {
                    'empl_name': attendance.employee_id.name,
                    'check_in': format_datetime(self.env, attendance.check_in, dt_format=False),
                }))
            else:
                result.append((attendance.id, _("%(empl_name)s from %(check_in)s to %(check_out)s") % {
                    'empl_name': attendance.employee_id.name,
                    'check_in': format_datetime(self.env, attendance.check_in, dt_format=False),
                    'check_out': format_datetime(self.env, attendance.check_out, dt_format=False),
                }))
        return result

    @api.depends('check_in', 'check_out')
    def _compute_worked_hours(self):
        for attendance in self:
            if attendance.check_out and attendance.check_in:
                delta = attendance.check_out - attendance.check_in
                attendance.worked_hours = delta.total_seconds() / 3600.0
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
                raise exceptions.ValidationError(_("Cannot create new attendance record for %(empl_name)s, the employee was already checked in on %(datetime)s") % {
                    'empl_name': attendance.employee_id.name,
                    'datetime': format_datetime(self.env, attendance.check_in, dt_format=False),
                })

            if not attendance.check_out:
                # if our attendance is "open" (no check_out), we verify there is no other "open" attendance
                no_check_out_attendances = self.env['hr.attendance'].search([
                    ('employee_id', '=', attendance.employee_id.id),
                    ('check_out', '=', False),
                    ('id', '!=', attendance.id),
                ], order='check_in desc', limit=1)
                if no_check_out_attendances:
                    raise exceptions.ValidationError(_("Cannot create new attendance record for %(empl_name)s, the employee hasn't checked out since %(datetime)s") % {
                        'empl_name': attendance.employee_id.name,
                        'datetime': format_datetime(self.env, no_check_out_attendances.check_in, dt_format=False),
                    })
            else:
                # we verify that the latest attendance with check_in time before our check_out time
                # is the same as the one before our check_in time computed before, otherwise it overlaps
                last_attendance_before_check_out = self.env['hr.attendance'].search([
                    ('employee_id', '=', attendance.employee_id.id),
                    ('check_in', '<', attendance.check_out),
                    ('id', '!=', attendance.id),
                ], order='check_in desc', limit=1)
                if last_attendance_before_check_out and last_attendance_before_check_in != last_attendance_before_check_out:
                    raise exceptions.ValidationError(_("Cannot create new attendance record for %(empl_name)s, the employee was already checked in on %(datetime)s") % {
                        'empl_name': attendance.employee_id.name,
                        'datetime': format_datetime(self.env, last_attendance_before_check_out.check_in, dt_format=False),
                    })

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
            expected_attendances = emp._get_expected_attendances(start, stop, domain=AND([self._get_overtime_leave_domain(), [('company_id', 'in', [False, emp.company_id.id])]]))

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
                        # Compute start and end time for that day
                        planned_start_dt, planned_end_dt = False, False
                        planned_work_duration = 0
                        for calendar_attendance in working_times[attendance_date]:
                            planned_start_dt = min(planned_start_dt, calendar_attendance[0]) if planned_start_dt else calendar_attendance[0]
                            planned_end_dt = max(planned_end_dt, calendar_attendance[1]) if planned_end_dt else calendar_attendance[1]
                            planned_work_duration += (calendar_attendance[1] - calendar_attendance[0]).total_seconds() / 3600.0
                        # Count time before, during and after 'working hours'
                        pre_work_time, work_duration, post_work_time = 0, 0, 0

                        for attendance in attendances:
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
                                work_duration += (min(planned_end_dt, local_check_out) - max(planned_start_dt, local_check_in)).total_seconds() / 3600.0
                            # There is an overtime at the end of the day
                            if local_check_out > planned_end_dt:
                                post_work_time += (local_check_out - max(planned_end_dt, local_check_in)).total_seconds() / 3600.0

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
                elif overtime:
                    overtime_to_unlink |= overtime
        self.env['hr.attendance.overtime'].sudo().create(overtime_vals_list)
        overtime_to_unlink.sudo().unlink()

    @api.model_create_multi
    def create(self, vals_list):
        res = super().create(vals_list)
        res._update_overtime()
        return res

    def write(self, vals):
        if vals.get('employee_id') and \
            vals['employee_id'] not in self.env.user.employee_ids.ids and \
            not self.env.user.has_group('hr_attendance.group_hr_attendance_user'):
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
        super(HrAttendance, self).unlink()
        self._update_overtime(attendances_dates)

    @api.returns('self', lambda value: value.id)
    def copy(self):
        raise exceptions.UserError(_('You cannot duplicate an attendance.'))

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

    attendance_ids = fields.One2many(
        'hr.attendance', 'employee_id', groups="hr_attendance.group_hr_attendance_user,hr.group_hr_user")
    last_attendance_id = fields.Many2one(
        'hr.attendance', compute='_compute_last_attendance_id', store=True,
        groups="hr_attendance.group_hr_attendance_kiosk,hr_attendance.group_hr_attendance,hr.group_hr_user")
    last_check_in = fields.Datetime(
        related='last_attendance_id.check_in', store=True,
        groups="hr_attendance.group_hr_attendance_user,hr.group_hr_user")
    last_check_out = fields.Datetime(
        related='last_attendance_id.check_out', store=True,
        groups="hr_attendance.group_hr_attendance_user,hr.group_hr_user")
    attendance_state = fields.Selection(
        string="Attendance Status", compute='_compute_attendance_state',
        selection=[('checked_out', "Checked out"), ('checked_in', "Checked in")],
        groups="hr_attendance.group_hr_attendance_kiosk,hr_attendance.group_hr_attendance,hr.group_hr_user")
    hours_last_month = fields.Float(
        compute='_compute_hours_last_month', groups="hr_attendance.group_hr_attendance_user,hr.group_hr_user")
    hours_today = fields.Float(
        compute='_compute_hours_today',
        groups="hr_attendance.group_hr_attendance_kiosk,hr_attendance.group_hr_attendance,hr.group_hr_user")
    hours_last_month_display = fields.Char(
        compute='_compute_hours_last_month', groups="hr_attendance.group_hr_attendance_user,hr.group_hr_user")
    overtime_ids = fields.One2many(
        'hr.attendance.overtime', 'employee_id', groups="hr_attendance.group_hr_attendance_user,hr.group_hr_user")
    total_overtime = fields.Float(
        compute='_compute_total_overtime', compute_sudo=True,
        groups="hr_attendance.group_hr_attendance_kiosk,hr_attendance.group_hr_attendance,hr.group_hr_user")

    @api.depends('overtime_ids.duration', 'attendance_ids')
    def _compute_total_overtime(self):
        for employee in self:
            if employee.company_id.hr_attendance_overtime:
                employee.total_overtime = float_round(sum(employee.overtime_ids.mapped('duration')), 2)
            else:
                employee.total_overtime = 0

    @api.depends('user_id.im_status', 'attendance_state')
    def _compute_presence_state(self):
        """
        Override to include checkin/checkout in the presence state
        Attendance has the second highest priority after login
        """
        super()._compute_presence_state()
        employees = self.filtered(lambda e: e.hr_presence_state != 'present')
        employee_to_check_working = self.filtered(lambda e: e.attendance_state == 'checked_out'
                                                            and e.hr_presence_state == 'to_define')
        working_now_list = employee_to_check_working._get_employee_working_now()
        for employee in employees:
            if employee.attendance_state == 'checked_out' and employee.hr_presence_state == 'to_define' and \
                    employee.id not in working_now_list:
                employee.hr_presence_state = 'absent'
            elif employee.attendance_state == 'checked_in':
                employee.hr_presence_state = 'present'

    def _compute_hours_last_month(self):
        now = fields.Datetime.now()
        now_utc = pytz.utc.localize(now)
        for employee in self:
            tz = pytz.timezone(employee.tz or 'UTC')
            now_tz = now_utc.astimezone(tz)
            start_tz = now_tz + relativedelta(months=-1, day=1, hour=0, minute=0, second=0, microsecond=0)
            start_naive = start_tz.astimezone(pytz.utc).replace(tzinfo=None)
            end_tz = now_tz + relativedelta(day=1, hour=0, minute=0, second=0, microsecond=0)
            end_naive = end_tz.astimezone(pytz.utc).replace(tzinfo=None)

            attendances = self.env['hr.attendance'].search([
                ('employee_id', '=', employee.id),
                '&',
                ('check_in', '<=', end_naive),
                ('check_out', '>=', start_naive),
            ])

            hours = 0
            for attendance in attendances:
                check_in = max(attendance.check_in, start_naive)
                check_out = min(attendance.check_out, end_naive)
                hours += (check_out - check_in).total_seconds() / 3600.0

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
            ])

            worked_hours = 0
            for attendance in attendances:
                delta = (attendance.check_out or now) - max(attendance.check_in, start_naive)
                worked_hours += delta.total_seconds() / 3600.0
            employee.hours_today = worked_hours

    @api.depends('attendance_ids')
    def _compute_last_attendance_id(self):
        for employee in self:
            employee.last_attendance_id = self.env['hr.attendance'].search([
                ('employee_id', '=', employee.id),
            ], limit=1)

    @api.depends('last_attendance_id.check_in', 'last_attendance_id.check_out', 'last_attendance_id')
    def _compute_attendance_state(self):
        for employee in self:
            att = employee.last_attendance_id.sudo()
            employee.attendance_state = att and not att.check_out and 'checked_in' or 'checked_out'

    @api.model
    def attendance_scan(self, barcode):
        """ Receive a barcode scanned from the Kiosk Mode and change the attendances of corresponding employee.
            Returns either an action or a warning.
        """
        employee = self.sudo().search([('barcode', '=', barcode)], limit=1)
        if employee:
            return employee._attendance_action('hr_attendance.hr_attendance_action_kiosk_mode')
        return {'warning': _("No employee corresponding to Badge ID '%(barcode)s.'") % {'barcode': barcode}}

    def attendance_manual(self, next_action, entered_pin=None):
        self.ensure_one()
        attendance_user_and_no_pin = self.user_has_groups(
            'hr_attendance.group_hr_attendance_user,'
            '!hr_attendance.group_hr_attendance_use_pin')
        can_check_without_pin = attendance_user_and_no_pin or (self.user_id == self.env.user and entered_pin is None)
        if can_check_without_pin or entered_pin is not None and entered_pin == self.sudo().pin:
            return self._attendance_action(next_action)
        if not self.user_has_groups('hr_attendance.group_hr_attendance_user'):
            return {'warning': _('To activate Kiosk mode without pin code, you must have access right as an Officer or above in the Attendance app. Please contact your administrator.')}
        return {'warning': _('Wrong PIN')}

    def _attendance_action(self, next_action):
        """ Changes the attendance of the employee.
            Returns an action to the check in/out message,
            next_action defines which menu the check in/out message should return to. ("My Attendances" or "Kiosk Mode")
        """
        self.ensure_one()
        employee = self.sudo()
        action_message = self.env["ir.actions.actions"]._for_xml_id("hr_attendance.hr_attendance_action_greeting_message")
        action_message['previous_attendance_change_date'] = employee.last_attendance_id and (employee.last_attendance_id.check_out or employee.last_attendance_id.check_in) or False
        action_message['employee_name'] = employee.name
        action_message['barcode'] = employee.barcode
        action_message['next_action'] = next_action
        action_message['hours_today'] = employee.hours_today
        action_message['kiosk_delay'] = employee.company_id.attendance_kiosk_delay * 1000

        if employee.user_id:
            modified_attendance = employee.with_user(employee.user_id).sudo()._attendance_action_change()
        else:
            modified_attendance = employee._attendance_action_change()
        action_message['attendance'] = modified_attendance.read()[0]
        action_message['show_total_overtime'] = employee.company_id.hr_attendance_overtime
        action_message['total_overtime'] = employee.total_overtime
        # Overtime have an unique constraint on the day, no need for limit=1
        action_message['overtime_today'] = self.env['hr.attendance.overtime'].sudo().search([
            ('employee_id', '=', employee.id), ('date', '=', fields.Date.context_today(self)), ('adjustment', '=', False)]).duration or 0
        return {'action': action_message}

    def _attendance_action_change(self):
        """ Check In/Check Out action
            Check In: create a new attendance record
            Check Out: modify check_out field of appropriate attendance record
        """
        self.ensure_one()
        action_date = fields.Datetime.now()

        if self.attendance_state != 'checked_in':
            vals = {
                'employee_id': self.id,
                'check_in': action_date,
            }
            return self.env['hr.attendance'].create(vals)
        attendance = self.env['hr.attendance'].search([('employee_id', '=', self.id), ('check_out', '=', False)], limit=1)
        if attendance:
            attendance.check_out = action_date
        else:
            raise exceptions.UserError(_('Cannot perform check out on %(empl_name)s, could not find corresponding check in. '
                'Your attendances have probably been modified manually by human resources.') % {'empl_name': self.sudo().name, })
        return attendance

    @api.model
    def read_group(self, domain, fields, groupby, offset=0, limit=None, orderby=False, lazy=True):
        if 'pin' in groupby or 'pin' in self.env.context.get('group_by', '') or self.env.context.get('no_group_by'):
            raise exceptions.UserError(_('Such grouping is not allowed.'))
        return super(HrEmployee, self).read_group(domain, fields, groupby, offset=offset, limit=limit, orderby=orderby, lazy=lazy)

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
        groups="hr_attendance.group_hr_attendance_kiosk,hr_attendance.group_hr_attendance")
    hours_today = fields.Float(related='employee_id.hours_today', readonly=True,
        groups="hr_attendance.group_hr_attendance_kiosk,hr_attendance.group_hr_attendance")
    last_attendance_id = fields.Many2one(related='employee_id.last_attendance_id', readonly=True,
        groups="hr_attendance.group_hr_attendance_kiosk,hr_attendance.group_hr_attendance")
    total_overtime = fields.Float(related='employee_id.total_overtime', readonly=True,
        groups="hr_attendance.group_hr_attendance_kiosk,hr_attendance.group_hr_attendance")

    def action_employee_kiosk_confirm(self):
        self.ensure_one()
        return {
            'type': 'ir.actions.client',
            'name': 'Confirm',
            'tag': 'hr_attendance_kiosk_confirm',
            'employee_id': self.id,
            'employee_name': self.name,
            'employee_state': self.attendance_state,
            'employee_hours_today': self.hours_today,
        }

```

## File: models\ir_ui_menu.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class IrUiMenu(models.Model):
    _inherit = 'ir.ui.menu'

    def _load_menus_blacklist(self):
        res = super()._load_menus_blacklist()
        if self.env.user.has_group('hr_attendance.group_hr_attendance_user'):
            res.append(self.env.ref('hr_attendance.menu_hr_attendance_attendances_overview').id)
        return res

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models
from odoo.osv.expression import OR


class ResCompany(models.Model):
    _inherit = 'res.company'

    hr_attendance_overtime = fields.Boolean(string="Count Extra Hours")
    overtime_start_date = fields.Date(string="Extra Hours Starting Date")
    overtime_company_threshold = fields.Integer(string="Tolerance Time In Favor Of Company", default=0)
    overtime_employee_threshold = fields.Integer(string="Tolerance Time In Favor Of Employee", default=0)

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

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    group_attendance_use_pin = fields.Boolean(
        string='Employee PIN',
        implied_group="hr_attendance.group_hr_attendance_use_pin")
    hr_attendance_overtime = fields.Boolean(
        string="Count Extra Hours", readonly=False)
    overtime_start_date = fields.Date(string="Extra Hours Starting Date", readonly=False)
    overtime_company_threshold = fields.Integer(
        string="Tolerance Time In Favor Of Company", readonly=False)
    overtime_employee_threshold = fields.Integer(
        string="Tolerance Time In Favor Of Employee", readonly=False)
    attendance_kiosk_mode = fields.Selection(related='company_id.attendance_kiosk_mode', readonly=False)
    attendance_barcode_source = fields.Selection(related='company_id.attendance_barcode_source', readonly=False)
    attendance_kiosk_delay = fields.Integer(related='company_id.attendance_kiosk_delay', readonly=False)

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

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields


class User(models.Model):
    _inherit = ['res.users']

    hours_last_month = fields.Float(related='employee_id.hours_last_month')
    hours_last_month_display = fields.Char(related='employee_id.hours_last_month_display')
    attendance_state = fields.Selection(related='employee_id.attendance_state')
    last_check_in = fields.Datetime(related='employee_id.last_attendance_id.check_in')
    last_check_out = fields.Datetime(related='employee_id.last_attendance_id.check_out')
    total_overtime = fields.Float(related='employee_id.total_overtime')

    @property
    def SELF_READABLE_FIELDS(self):
        return super().SELF_READABLE_FIELDS + [
            'hours_last_month',
            'hours_last_month_display',
            'attendance_state',
            'last_check_in',
            'last_check_out',
            'total_overtime'
        ]

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import res_config_settings
from . import hr_attendance
from . import hr_attendance_overtime
from . import hr_employee
from . import hr_employee_public
from . import ir_ui_menu
from . import res_company
from . import res_users

```

## File: report\hr_attendance_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, tools


class HRAttendanceReport(models.Model):
    _name = "hr.attendance.report"
    _description = "Attendance Statistics"
    _auto = False

    department_id = fields.Many2one('hr.department', string="Department", readonly=True)
    employee_id = fields.Many2one('hr.employee', string="Employee", readonly=True)
    company_id = fields.Many2one('res.company', string="Company", readonly=True)
    check_in = fields.Date("Check In", readonly=True)
    worked_hours = fields.Float("Hours Worked", readonly=True)
    overtime_hours = fields.Float("Extra Hours", readonly=True)

    @api.model
    def _select(self):
        return """
            SELECT
                hra.id,
                hr_employee.department_id,
                hra.employee_id,
                hr_employee.company_id,
                hra.check_in,
                hra.worked_hours,
                coalesce(ot.duration, 0) as overtime_hours
        """

    @api.model
    def _from(self):
        return """
            FROM (
                SELECT
                    id,
                    row_number() over (partition by employee_id, CAST(check_in AS DATE)) as ot_check,
                    employee_id,
                    CAST(check_in
                            at time zone 'utc'
                            at time zone
                                (SELECT calendar.tz FROM resource_calendar as calendar
                                INNER JOIN hr_employee as employee ON employee.id = hr_attendance.employee_id
                                WHERE calendar.id = employee.resource_calendar_id)
                    as DATE) as check_in,
                    worked_hours
                FROM
                    hr_attendance
                ) as hra
        """

    def _join(self):
        return """
            LEFT JOIN hr_employee ON hr_employee.id = hra.employee_id
            LEFT JOIN hr_attendance_overtime ot
                ON hra.ot_check = 1
                AND ot.employee_id = hra.employee_id
                AND ot.date = hra.check_in
                AND ot.adjustment = FALSE
        """

    def init(self):
        tools.drop_view_if_exists(self.env.cr, self._table)

        self.env.cr.execute("""
            CREATE OR REPLACE VIEW %s AS (
                %s
                %s
                %s
            )
        """ % (self._table, self._select(), self._from(), self._join())
        )

```

## File: report\hr_attendance_report_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hr_attendance_report_view_search" model="ir.ui.view">
        <field name="name">hr.attendance.report.view.search</field>
        <field name="model">hr.attendance.report</field>
        <field name="arch" type="xml">
            <search string="HR Attendance Search">
                <field name="employee_id"/>
                <field name="department_id" operator="child_of"/>
                <filter name="check_in" string="Check In" date="check_in" default_period="last_month"/>
                <group expand="0" string="Group By">
                    <filter string="Employee" name="groupby_employee" context="{'group_by': 'employee_id'}"/>
                    <filter string="Check In" name="groupby_check_in" context="{'group_by': 'check_in'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="hr_attendance_report_view_pivot" model="ir.ui.view">
        <field name="name">hr.attendance.report.view.pivot</field>
        <field name="model">hr.attendance.report</field>
        <field name="arch" type="xml">
            <pivot string="Attendance" js_class="attendance_report_pivot">
                <field name="employee_id" type="row"/>
                <field name="check_in" type="col"/>
                <field name="worked_hours" type="measure" widget="float_time"/>
                <field name="overtime_hours" type="measure" widget="float_time"/>
            </pivot>
        </field>
    </record>

    <record id="hr_attendance_report_view_graph" model="ir.ui.view">
        <field name="name">hr.attendance.report.view.graph</field>
        <field name="model">hr.attendance.report</field>
        <field name="arch" type="xml">
            <graph string="Attendance Statistics" stacked="0" js_class="attendance_report_graph">
                <field name="employee_id"/>
                <field name="check_in"/>
                <field name="overtime_hours" type="measure" />
                <field name="worked_hours" type="measure" />
            </graph>
        </field>
    </record>

    <record id="hr_attendance_report_action" model="ir.actions.act_window">
        <field name="name">Attendance Analysis</field>
        <field name="res_model">hr.attendance.report</field>
        <field name="view_mode">graph,pivot</field>
        <field name="search_view_id" ref="hr_attendance_report_view_search"/>
        <field name="context">{'group_by': ['check_in:day', 'employee_id'], 'search_default_check_in': '1'}</field>
    </record>

    <record id="hr_attendance_report_action_filtered" model="ir.actions.act_window">
        <field name="name">Attendance Analysis</field>
        <field name="res_model">hr.attendance.report</field>
        <field name="view_mode">graph,pivot</field>
        <field name="search_view_id" ref="hr_attendance_report_view_search"/>
        <field name="context">{
            'group_by': ['check_in:day', 'employee_id'],
            'search_default_department_id': [active_id]}
        </field>
    </record>

    <menuitem
        id="menu_hr_attendance_report"
        name="Reporting"
        sequence="30"
        parent="hr_attendance.menu_hr_attendance_root"
        action="hr_attendance_report_action"
        groups="hr_attendance.group_hr_attendance_user"/>
</odoo>

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_attendance_report

```

## File: security\hr_attendance_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record model="ir.module.category" id="base.module_category_human_resources_attendances">
        <field name="sequence">14</field>
    </record>

    <record id="group_hr_attendance_kiosk" model="res.groups">
        <field name="name">User : Only kiosk mode</field>
        <field name="category_id" ref="base.module_category_human_resources_employees"/>
        <field name="comment">The user will be able to open the kiosk mode and validate the employee PIN.</field>
        <field name="implied_ids" eval="[(6, 0, [ref('base.group_user')])]"/>
    </record>

    <record id="hr.group_hr_user" model="res.groups">
        <field name="implied_ids" eval="[(4, ref('hr_attendance.group_hr_attendance_kiosk'))]"/>
    </record>

    <record id="group_hr_attendance" model="res.groups">
        <field name="name">User</field>
        <field name="category_id" ref="base.module_category_human_resources_attendances"/>
        <field name="comment">The user will gain access to the human resources attendance menu, enabling him to manage his own attendance.</field>
    </record>

    <record id="group_hr_attendance_user" model="res.groups">
        <field name="name">Officer : Manage all attendances</field>
        <field name="category_id" ref="base.module_category_human_resources_attendances"/>
        <field name="implied_ids" eval="[(4, ref('hr.group_hr_user')), (4, ref('hr_attendance.group_hr_attendance'))]"/>
    </record>

    <record id="group_hr_attendance_manager" model="res.groups">
        <field name="name">Administrator</field>
        <field name="category_id" ref="base.module_category_human_resources_attendances"/>
        <field name="implied_ids" eval="[(4, ref('hr_attendance.group_hr_attendance_user'))]"/>
        <field name="users" eval="[(4, ref('base.user_root')), (4, ref('base.user_admin'))]"/>
    </record>

    <record id="base.default_user" model="res.users">
        <field name="groups_id" eval="[(4,ref('hr_attendance.group_hr_attendance_manager'))]"/>
    </record>

    <record id="group_hr_attendance_use_pin" model="res.groups">
        <field name="name">Enable PIN use</field>
        <field name="category_id" ref="base.module_category_hidden"/>
        <field name="comment">The user will have to enter his PIN to check in and out manually at the company screen.</field>
    </record>

    <data noupdate="1">

        <record id="hr_attendance_rule_employee_company" model="ir.rule">
            <field name="name">Employee multi company rule</field>
            <field name="model_id" ref="model_hr_attendance"/>
            <field name="global" eval="True"/>
            <field name="domain_force">['|',('employee_id.company_id','=',False),('employee_id.company_id', 'in', company_ids)]</field>
        </record>

        <record id="hr_attendance_rule_attendance_manager" model="ir.rule">
            <field name="name">attendance officer: full access</field>
            <field name="model_id" ref="model_hr_attendance"/>
            <field name="domain_force">[(1,'=',1)]</field>
            <field name="groups" eval="[(4,ref('hr_attendance.group_hr_attendance_user'))]"/>
        </record>

        <record id="hr_attendance_rule_attendance_manual" model="ir.rule">
            <field name="name">Manual Attendance: own attendances</field>
            <field name="model_id" ref="model_hr_attendance"/>
            <field name="domain_force">[('employee_id.user_id', '=', user.id)]</field>
            <field name="perm_read" eval="1"/>
            <field name="perm_write" eval="0"/>
            <field name="perm_create" eval="1"/>
            <field name="perm_unlink" eval="0"/>
            <field name="groups" eval="[(4,ref('hr_attendance.group_hr_attendance'))]"/>
        </record>

        <record id="hr_attendance_rule_attendance_employee" model="ir.rule">
            <field name="name">user: read own attendance only</field>
            <field name="model_id" ref="model_hr_attendance"/>
            <field name="domain_force">[('employee_id.user_id','=',user.id)]</field>
            <field name="perm_read" eval="1"/>
            <field name="perm_write" eval="0"/>
            <field name="perm_create" eval="0"/>
            <field name="perm_unlink" eval="0"/>
            <field name="groups" eval="[(4,ref('base.group_user'))]"/>
        </record>

        <!-- Overtime -->
        <record id="hr_attendance_overtime_rule_employee_company" model="ir.rule">
            <field name="name">Employee multi company rule</field>
            <field name="model_id" ref="model_hr_attendance_overtime"/>
            <field name="global" eval="True"/>
            <field name="domain_force">['|',('employee_id.company_id','=',False),('employee_id.company_id', 'in', company_ids)]</field>
        </record>

        <record id="hr_attendance_rule_attendance_overtime_manager" model="ir.rule">
            <field name="name">attendance officer: full access</field>
            <field name="model_id" ref="model_hr_attendance_overtime"/>
            <field name="domain_force">[(1,'=',1)]</field>
            <field name="groups" eval="[(4,ref('hr_attendance.group_hr_attendance_user'))]"/>
        </record>

        <record id="hr_attendance_rule_attendance_overtime_employee" model="ir.rule">
            <field name="name">user: read and modify own attendance only</field>
            <field name="model_id" ref="model_hr_attendance_overtime"/>
            <field name="domain_force">[('employee_id.user_id', '=', user.id)]</field>
            <field name="perm_read" eval="1"/>
            <field name="perm_write" eval="0"/>
            <field name="perm_create" eval="0"/>
            <field name="perm_unlink" eval="0"/>
            <field name="groups" eval="[(4,ref('base.group_user'))]"/>
        </record>

        <!-- Report -->
        <record id="hr_attendance_report_rule_multi_company" model="ir.rule">
            <field name="name">Attendance report multi company rule</field>
            <field name="model_id" ref="model_hr_attendance_report"/>
            <field name="domain_force">[('company_id', 'in', company_ids)]</field>
        </record>
    </data>
</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_hr_attendance_officer,hr.attendance.user,model_hr_attendance,group_hr_attendance_user,1,1,1,1
access_hr_attendance_user,hr.attendance.system.user,model_hr_attendance,group_hr_attendance,1,0,1,0
access_hr_attendance_system_user,hr.attendance.system.user,model_hr_attendance,base.group_user,1,0,0,0
access_hr_attendance_overtime_user,hr.attendance.overtime.user,model_hr_attendance_overtime,hr_attendance.group_hr_attendance_user,1,1,1,1
access_hr_attendance_overtime_system_user,hr.attendance.overtime.system.user,model_hr_attendance_overtime,base.group_user,1,0,0,0
access_hr_attendance_report_user,hr.attendance.report.user,model_hr_attendance_report,hr_attendance.group_hr_attendance_user,1,1,1,1

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#94B6C8"/><stop offset="100%" stop-color="#6A9EBA"/></linearGradient><path id="d" d="M28.278 17.235c5.193 0 9.404 4.271 9.404 9.54 0 5.27-4.21 9.541-9.404 9.541s-9.404-4.271-9.404-9.54c0-5.27 4.21-9.541 9.404-9.541zm6.018 19.492c0-.413-9.178 1.847-12.6-.65l-4.19 1.063c-2.512.638-4.275 2.927-4.275 5.554v2.209c0 1.58 1.264 2.862 2.822 2.862h15.466c-1.852-6.34 2.777-10.625 2.777-11.038zM45.5 33.794c-6.466 0-11.706 5.24-11.706 11.706 0 6.466 5.24 11.706 11.706 11.706 6.466 0 11.706-5.24 11.706-11.706 0-6.466-5.24-11.706-11.706-11.706zm2.695 16.525l-4.163-3.025a.57.57 0 0 1-.231-.458v-7.944c0-.312.255-.566.566-.566h2.266c.311 0 .566.254.566.566v6.5l2.997 2.18a.566.566 0 0 1 .123.793l-1.33 1.831a.57.57 0 0 1-.794.123z"/><path id="e" d="M28.278 15.235c5.193 0 9.404 4.271 9.404 9.54 0 5.27-4.21 9.541-9.404 9.541s-9.404-4.271-9.404-9.54c0-5.27 4.21-9.541 9.404-9.541zm6.018 19.492c0-.413-9.178 1.847-12.6-.65l-4.19 1.063c-2.512.638-4.275 2.927-4.275 5.554v2.209c0 1.58 1.264 2.862 2.822 2.862h15.466c-1.852-6.34 2.777-10.625 2.777-11.038zM45.5 31.794c-6.466 0-11.706 5.24-11.706 11.706 0 6.466 5.24 11.706 11.706 11.706 6.466 0 11.706-5.24 11.706-11.706 0-6.466-5.24-11.706-11.706-11.706zm2.695 16.525l-4.163-3.025a.57.57 0 0 1-.231-.458v-7.944c0-.312.255-.566.566-.566h2.266c.311 0 .566.254.566.566v6.5l2.997 2.18a.566.566 0 0 1 .123.793l-1.33 1.831a.57.57 0 0 1-.794.123z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M42.488 69H4c-2 0-4-1-4-4V41.348l20.938-22.35 14.703 11.46-4.201 5.183-1.252 7.42 6.406-7.111L46 33l9.958 15.844L42.488 69z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: static\src\js\attendance_report_views.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { graphView } from "@web/views/graph/graph_view";
import { pivotView } from "@web/views/pivot/pivot_view";

const viewRegistry = registry.category("views");

/**
 * Open the hr.attendance instead of the report list view.
 */
function openView(component, domain, views, context) {
    component.actionService.doAction({
        type: "ir.actions.act_window",
        name: component.model.metaData.title,
        res_model: 'hr.attendance',
        views: [[false, "list"], [false, "form"]],
        view_mode: "list",
        target: "current",
        context,
        domain,
    });
}

export class AttendanceReportGraphController extends graphView.Controller {
    /**
     * @override
     */
    openView(domain, views, context) {
        openView(this, domain, views, context);
    }
}

viewRegistry.add("attendance_report_graph", {
    ...graphView,
    Controller: AttendanceReportGraphController
});

export class AttendanceReportPivotController extends pivotView.Controller {
    /**
     * @override
     */
    openView(domain, views, context) {
        openView(this, domain, views, context);
    }
}

viewRegistry.add("attendance_report_pivot", {
    ...pivotView,
    Controller: AttendanceReportPivotController
});

```

## File: static\src\js\greeting_message.js

```javascript
odoo.define('hr_attendance.greeting_message', function (require) {
"use strict";

var AbstractAction = require('web.AbstractAction');
var core = require('web.core');
var time = require('web.time');
var field_utils = require('web.field_utils');

var _t = core._t;


var GreetingMessage = AbstractAction.extend({
    contentTemplate: 'HrAttendanceGreetingMessage',

    events: {
        "click .o_hr_attendance_button_dismiss": function() { this.do_action(this.next_action, {clear_breadcrumbs: true}); },
    },

    init: function(parent, action) {
        var self = this;
        this._super.apply(this, arguments);
        this.activeBarcode = true;
        this.kioskDelay = action.kiosk_delay;

        // if no correct action given (due to an erroneous back or refresh from the browser), we set the dismiss button to return
        // to the (likely) appropriate menu, according to the user access rights
        if(!action.attendance) {
            this.activeBarcode = false;
            this.getSession().user_has_group('hr_attendance.group_hr_attendance_user').then(function(has_group) {
                if(has_group) {
                    self.next_action = 'hr_attendance.hr_attendance_action_kiosk_mode';
                } else {
                    self.next_action = 'hr_attendance.hr_attendance_action_my_attendances';
                }
            });
            return;
        }

        this.next_action = action.next_action || 'hr_attendance.hr_attendance_action_my_attendances';
        // no listening to barcode scans if we aren't coming from the kiosk mode (and thus not going back to it with next_action)
        if (this.next_action != 'hr_attendance.hr_attendance_action_kiosk_mode' && this.next_action.tag != 'hr_attendance_kiosk_mode') {
            this.activeBarcode = false;
        }

        this.attendance = action.attendance;
        // We receive the check in/out times in UTC
        // This widget only deals with display, which should be in browser's TimeZone
        this.attendance.check_in = this.attendance.check_in && moment.utc(this.attendance.check_in).local();
        this.attendance.check_out = this.attendance.check_out && moment.utc(this.attendance.check_out).local();
        this.previous_attendance_change_date = action.previous_attendance_change_date && moment.utc(action.previous_attendance_change_date).local();

        // check in/out times displayed in the greeting message template.
        this.format_time = time.getLangTimeFormat();
        this.attendance.check_in_time = this.attendance.check_in && this.attendance.check_in.format(this.format_time);
        this.attendance.check_out_time = this.attendance.check_out && this.attendance.check_out.format(this.format_time);

        // extra hours amount displayed in the greeting message template.
        this.show_total_overtime = action.show_total_overtime;
        this.total_overtime_float = action.total_overtime; // Used for comparison in template
        this.total_overtime = field_utils.format.float_time(this.total_overtime_float);
        this.today_overtime_float = action.overtime_today;
        this.today_overtime = field_utils.format.float_time(this.today_overtime_float);

        if (action.hours_today) {
            var duration = moment.duration(action.hours_today, "hours");
            this.hours_today = duration.hours() + ' hours, ' + duration.minutes() + ' minutes';
        }

        this.employee_name = action.employee_name;
        this.attendanceBarcode = action.barcode;
    },

    start: function() {
        if (this.attendance) {
            this.attendance.check_out ? this.farewell_message() : this.welcome_message();
        }
        if (this.activeBarcode) {
            core.bus.on('barcode_scanned', this, this._onBarcodeScanned);
        }
        return this._super.apply(this, arguments);
    },

    welcome_message: function() {
        var self = this;
        var now = this.attendance.check_in.clone();
        if (this.kioskDelay > 0) {
            this.return_to_main_menu = setTimeout( function() { self.do_action(self.next_action, {clear_breadcrumbs: true}); }, this.kioskDelay);
        }

        if (now.hours() < 5) {
            this.$('.o_hr_attendance_message_message').append(_t("Good night"));
        } else if (now.hours() < 12) {
            if (now.hours() < 8 && Math.random() < 0.3) {
                if (Math.random() < 0.75) {
                    this.$('.o_hr_attendance_message_message').append(_t("The early bird catches the worm"));
                } else {
                    this.$('.o_hr_attendance_message_message').append(_t("First come, first served"));
                }
            } else {
                this.$('.o_hr_attendance_message_message').append(_t("Good morning"));
            }
        } else if (now.hours() < 17){
            this.$('.o_hr_attendance_message_message').append(_t("Good afternoon"));
        } else if (now.hours() < 23){
            this.$('.o_hr_attendance_message_message').append(_t("Good evening"));
        } else {
            this.$('.o_hr_attendance_message_message').append(_t("Good night"));
        }
        if(this.previous_attendance_change_date){
            var last_check_out_date = this.previous_attendance_change_date.clone();
            if(now - last_check_out_date > 24*7*60*60*1000){
                this.$('.o_hr_attendance_random_message').html(_t("Glad to have you back, it's been a while!"));
            } else {
                if(Math.random() < 0.02){
                    this.$('.o_hr_attendance_random_message').html(_t("If a job is worth doing, it is worth doing well!"));
                }
            }
        }
    },

    farewell_message: function() {
        var self = this;
        var now = this.attendance.check_out.clone();
        if (this.kioskDelay > 0) {
            this.return_to_main_menu = setTimeout( function() { self.do_action(self.next_action, {clear_breadcrumbs: true}); }, this.kioskDelay);
        }

        if(this.previous_attendance_change_date){
            var last_check_in_date = this.previous_attendance_change_date.clone();
            if(now - last_check_in_date > 1000*60*60*12){
                this.$('.o_hr_attendance_warning_message').show().append(_t("<b>Warning! Last check in was over 12 hours ago.</b><br/>If this isn't right, please contact Human Resource staff"));
                if (this.return_to_main_menu) {
                    clearTimeout(this.return_to_main_menu);
                }
                this.activeBarcode = false;
            } else if(now - last_check_in_date > 1000*60*60*8){
                this.$('.o_hr_attendance_random_message').html(_t("Another good day's work! See you soon!"));
            }
        }

        if (now.hours() < 12) {
            this.$('.o_hr_attendance_message_message').append(_t("Have a good day!"));
        } else if (now.hours() < 14) {
            this.$('.o_hr_attendance_message_message').append(_t("Have a nice lunch!"));
            if (Math.random() < 0.05) {
                this.$('.o_hr_attendance_random_message').html(_t("Eat breakfast as a king, lunch as a merchant and supper as a beggar"));
            } else if (Math.random() < 0.06) {
                this.$('.o_hr_attendance_random_message').html(_t("An apple a day keeps the doctor away"));
            }
        } else if (now.hours() < 17) {
            this.$('.o_hr_attendance_message_message').append(_t("Have a good afternoon"));
        } else {
            if (now.hours() < 18 && Math.random() < 0.2) {
                this.$('.o_hr_attendance_message_message').append(_t("Early to bed and early to rise, makes a man healthy, wealthy and wise"));
            } else {
                this.$('.o_hr_attendance_message_message').append(_t("Have a good evening"));
            }
        }
    },

    _onBarcodeScanned: function(barcode) {
        var self = this;
        if (this.attendanceBarcode !== barcode){
            if (this.return_to_main_menu) {  // in case of multiple scans in the greeting message view, delete the timer, a new one will be created.
                clearTimeout(this.return_to_main_menu);
            }
            core.bus.off('barcode_scanned', this, this._onBarcodeScanned);
            const kioskDelay = this.kioskDelay;
            this._rpc({
                    model: 'hr.employee',
                    method: 'attendance_scan',
                    args: [barcode, ],
                })
                .then(function (result) {
                    if (result.action) {
                        self.do_action(result.action);
                    } else if (result.warning) {
                        self.displayNotification({ title: result.warning, type: 'danger' });
                        if (kioskDelay > 0) {
                            setTimeout( function() { self.do_action(self.next_action, {clear_breadcrumbs: true}); }, kioskDelay);
                        }
                    }
                }, function () {
                    if (kioskDelay > 0) {
                        setTimeout( function() { self.do_action(self.next_action, {clear_breadcrumbs: true}); }, kioskDelay);
                    }
                });
        }
    },

    destroy: function () {
        core.bus.off('barcode_scanned', this, this._onBarcodeScanned);
        clearTimeout(this.return_to_main_menu);
        this._super.apply(this, arguments);
    },
});

core.action_registry.add('hr_attendance_greeting_message', GreetingMessage);

return GreetingMessage;

});

```

## File: static\src\js\kiosk_confirm.js

```javascript
odoo.define('hr_attendance.kiosk_confirm', function (require) {
"use strict";

var AbstractAction = require('web.AbstractAction');
var core = require('web.core');
var field_utils = require('web.field_utils');
var QWeb = core.qweb;

const session = require('web.session');


var KioskConfirm = AbstractAction.extend({
    events: {
        "click .o_hr_attendance_back_button": function () { this.do_action(this.next_action, {clear_breadcrumbs: true}); },
        "click .o_hr_attendance_sign_in_out_icon": _.debounce(function () {
            var self = this;
            this._rpc({
                    model: 'hr.employee',
                    method: 'attendance_manual',
                    args: [[this.employee_id], this.next_action],
                    context: session.user_context,
                })
                .then(function(result) {
                    if (result.action) {
                        self.do_action(result.action);
                    } else if (result.warning) {
                        self.displayNotification({ title: result.warning, type: 'danger' });
                    }
                });
        }, 200, true),
        'click .o_hr_attendance_pin_pad_button_0': function() { this.$('.o_hr_attendance_PINbox').val(this.$('.o_hr_attendance_PINbox').val() + 0); },
        'click .o_hr_attendance_pin_pad_button_1': function() { this.$('.o_hr_attendance_PINbox').val(this.$('.o_hr_attendance_PINbox').val() + 1); },
        'click .o_hr_attendance_pin_pad_button_2': function() { this.$('.o_hr_attendance_PINbox').val(this.$('.o_hr_attendance_PINbox').val() + 2); },
        'click .o_hr_attendance_pin_pad_button_3': function() { this.$('.o_hr_attendance_PINbox').val(this.$('.o_hr_attendance_PINbox').val() + 3); },
        'click .o_hr_attendance_pin_pad_button_4': function() { this.$('.o_hr_attendance_PINbox').val(this.$('.o_hr_attendance_PINbox').val() + 4); },
        'click .o_hr_attendance_pin_pad_button_5': function() { this.$('.o_hr_attendance_PINbox').val(this.$('.o_hr_attendance_PINbox').val() + 5); },
        'click .o_hr_attendance_pin_pad_button_6': function() { this.$('.o_hr_attendance_PINbox').val(this.$('.o_hr_attendance_PINbox').val() + 6); },
        'click .o_hr_attendance_pin_pad_button_7': function() { this.$('.o_hr_attendance_PINbox').val(this.$('.o_hr_attendance_PINbox').val() + 7); },
        'click .o_hr_attendance_pin_pad_button_8': function() { this.$('.o_hr_attendance_PINbox').val(this.$('.o_hr_attendance_PINbox').val() + 8); },
        'click .o_hr_attendance_pin_pad_button_9': function() { this.$('.o_hr_attendance_PINbox').val(this.$('.o_hr_attendance_PINbox').val() + 9); },
        'click .o_hr_attendance_pin_pad_button_C': function() { this.$('.o_hr_attendance_PINbox').val(''); },
        'click .o_hr_attendance_pin_pad_button_ok': function() { this._send_pin_debounced() },
    },

    _sendPin: function() {
        var self = this;
        this.$('.o_hr_attendance_pin_pad_button_ok').attr("disabled", "disabled");
        this._rpc({
            model: 'hr.employee',
            method: 'attendance_manual',
            args: [[this.employee_id], this.next_action, this.$('.o_hr_attendance_PINbox').val()],
            context: session.user_context,
        })
        .then(function(result) {
            self.pin_is_send = true
            if (result.action) {
                self.do_action(result.action);
            } else if (result.warning) {
                self.displayNotification({ title: result.warning, type: 'danger' });
                self.$('.o_hr_attendance_PINbox').val('');
                setTimeout( function() { self.$('.o_hr_attendance_pin_pad_button_ok').removeAttr("disabled"); }, 500);
            }
        });
    },

    init: function (parent, action) {
        this._super.apply(this, arguments);
        this.next_action = 'hr_attendance.hr_attendance_action_kiosk_mode';
        this.employee_id = action.employee_id;
        this.employee_name = action.employee_name;
        this.employee_state = action.employee_state;
        this.employee_hours_today = field_utils.format.float_time(action.employee_hours_today);

        this.pin_is_send = false
        this._send_pin_debounced = _.debounce(this._sendPin, 200, true);

        window.addEventListener("keydown", (ev) => {
            const allowedKeys = ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'Backspace', 'Enter', 'Delete'];
            const key = ev.key;

            if (!allowedKeys.includes(key)) {
                return;
            }

            ev.preventDefault();
            ev.stopPropagation();

            const pinBox = this.$('.o_hr_attendance_PINbox');

            if (key.length == 1) {
                pinBox.val(pinBox.val() + key);
            }
            else if (key == 'Enter' && !this.pin_is_send) {
                this._send_pin_debounced();
            }
            else if (key == 'Backspace') {
                pinBox.val(pinBox.val().substring(0, pinBox.val().length - 1));
            }
            else {  // Delete
                pinBox.val('');
            }
        });
    },

    start: function () {
        var self = this;
        this.getSession().user_has_group('hr_attendance.group_hr_attendance_use_pin').then(function(has_group){
            self.use_pin = has_group;
            self.$el.html(QWeb.render("HrAttendanceKioskConfirm", {widget: self}));
            self.start_clock();
        });
        return self._super.apply(this, arguments);
    },

    start_clock: function () {
        this.clock_start = setInterval(function() {this.$(".o_hr_attendance_clock").text(new Date().toLocaleTimeString(navigator.language, {hour: '2-digit', minute:'2-digit', second:'2-digit'}));}, 500);
        // First clock refresh before interval to avoid delay
        this.$(".o_hr_attendance_clock").show().text(new Date().toLocaleTimeString(navigator.language, {hour: '2-digit', minute:'2-digit', second:'2-digit'}));
    },

    destroy: function () {
        clearInterval(this.clock_start);
        this._super.apply(this, arguments);
    },
});

core.action_registry.add('hr_attendance_kiosk_confirm', KioskConfirm);

return KioskConfirm;

});

```

## File: static\src\js\kiosk_mode.js

```javascript
odoo.define('hr_attendance.kiosk_mode', function (require) {
"use strict";

var AbstractAction = require('web.AbstractAction');
var ajax = require('web.ajax');
var core = require('web.core');
var Session = require('web.session');

var QWeb = core.qweb;


var KioskMode = AbstractAction.extend({
    events: {
        "click .o_hr_attendance_button_employees": function() {
            this.do_action('hr_attendance.hr_employee_attendance_action_kanban', {
                additional_context: {'no_group_by': true},
            });
        },
    },

    start: function () {
        var self = this;
        core.bus.on('barcode_scanned', this, this._onBarcodeScanned);
        self.session = Session;
        const company_id = this.session.user_context.allowed_company_ids[0];
        var def = this._rpc({
                model: 'res.company',
                method: 'search_read',
                args: [[['id', '=', company_id]], ['name', 'attendance_kiosk_mode', 'attendance_barcode_source']],
            })
            .then(function (companies){
                self.company_name = companies[0].name;
                self.company_image_url = self.session.url('/web/image', {model: 'res.company', id: company_id, field: 'logo',});
                self.kiosk_mode = companies[0].attendance_kiosk_mode;
                self.barcode_source = companies[0].attendance_barcode_source;
                self.$el.html(QWeb.render("HrAttendanceKioskMode", {widget: self}));
                self.start_clock();
            });
        // Make a RPC call every day to keep the session alive
        self._interval = window.setInterval(this._callServer.bind(this), (60*60*1000*24));
        return Promise.all([def, this._super.apply(this, arguments)]);
    },

    on_attach_callback: function () {
        // Stop the bus_service to avoid notifications in kiosk mode
        this.call('bus_service', 'stop');
        $('body').find('.o_ChatWindowHeader_commandClose').click();
    },

    _onBarcodeScanned: function(barcode) {
        var self = this;
        core.bus.off('barcode_scanned', this, this._onBarcodeScanned);
        this._rpc({
                model: 'hr.employee',
                method: 'attendance_scan',
                args: [barcode, ],
            })
            .then(function (result) {
                if (result.action) {
                    self.do_action(result.action);
                } else if (result.warning) {
                    self.displayNotification({ title: result.warning, type: 'danger' });
                    core.bus.on('barcode_scanned', self, self._onBarcodeScanned);
                }
            }, function () {
                core.bus.on('barcode_scanned', self, self._onBarcodeScanned);
            });
    },

    start_clock: function() {
        this.clock_start = setInterval(function() {this.$(".o_hr_attendance_clock").text(new Date().toLocaleTimeString(navigator.language, {hour: '2-digit', minute:'2-digit', second:'2-digit'}));}, 500);
        // First clock refresh before interval to avoid delay
        this.$(".o_hr_attendance_clock").show().text(new Date().toLocaleTimeString(navigator.language, {hour: '2-digit', minute:'2-digit', second:'2-digit'}));
    },

    destroy: function () {
        core.bus.off('barcode_scanned', this, this._onBarcodeScanned);
        clearInterval(this.clock_start);
        clearInterval(this._interval);
        this._super.apply(this, arguments);
    },

    _callServer: function () {
        // Make a call to the database to avoid the auto close of the session
        return ajax.rpc("/hr_attendance/kiosk_keepalive", {});
    },

});

core.action_registry.add('hr_attendance_kiosk_mode', KioskMode);

return KioskMode;

});

```

## File: static\src\js\my_attendances.js

```javascript
odoo.define('hr_attendance.my_attendances', function (require) {
"use strict";

var AbstractAction = require('web.AbstractAction');
var core = require('web.core');
var field_utils = require('web.field_utils');

const session = require('web.session');

var MyAttendances = AbstractAction.extend({
    contentTemplate: 'HrAttendanceMyMainMenu',
    events: {
        "click .o_hr_attendance_sign_in_out_icon": _.debounce(function() {
            this.update_attendance();
        }, 200, true),
    },

    willStart: function () {
        var self = this;

        var def = this._rpc({
                model: 'hr.employee',
                method: 'search_read',
                args: [[['user_id', '=', this.getSession().uid]], ['attendance_state', 'name', 'hours_today']],
                context: session.user_context,
            })
            .then(function (res) {
                self.employee = res.length && res[0];
                if (res.length) {
                    self.hours_today = field_utils.format.float_time(self.employee.hours_today);
                }
            });

        return Promise.all([def, this._super.apply(this, arguments)]);
    },

    update_attendance: function () {
        var self = this;
        this._rpc({
                model: 'hr.employee',
                method: 'attendance_manual',
                args: [[self.employee.id], 'hr_attendance.hr_attendance_action_my_attendances'],
                context: session.user_context,
            })
            .then(function(result) {
                if (result.action) {
                    self.do_action(result.action);
                } else if (result.warning) {
                    self.displayNotification({ title: result.warning, type: 'danger' });
                }
            });
    },
});

core.action_registry.add('hr_attendance_my_attendances', MyAttendances);

return MyAttendances;

});

```

## File: static\src\xml\attendance.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<template xml:space="preserve">
    <t t-name="PresenceIndicator">
        <div id="oe_hr_attendance_status" class="fa fa-circle me-1 text-400 " role="img" aria-label="Available" title="Available">
        </div>
    </t>

    <t t-name="HrAttendanceCardLayout">
        <div class="o_hr_attendance_kiosk_mode_container o_home_menu_background d-flex flex-column align-items-center justify-content-center h-100 text-center">
            <span class="o_hr_attendance_kiosk_backdrop position-absolute top-0 start-0 end-0 bottom-0 bg-black-25"/>
            <div class="o_hr_attendance_clock bg-black-50 p-3 py-md-2 m-0 mt-md-5 me-md-5 h2 text-white font-monospace"/>
            <div t-attf-class="o_hr_attendance_kiosk_mode flex-grow-1 flex-md-grow-0 card pb-3 px-0 px-lg-5 {{kioskModeClasses ? kioskModeClasses : '' }}">
                <div class="card-body d-flex flex-column p-0 p-md-4">
                    <t t-out="bodyContent"></t>
                </div>
            </div>
        </div>
    </t>

    <t t-name="HrAttendanceUserBadge">
        <div class="o_hr_attendance_user_badge o_home_menu_background d-flex align-items-end justify-content-center flex-grow-1 pt-5 pt-md-4 bg-odoo">
            <img class="img rounded-circle mb-n5" t-attf-src="/web/image?model=hr.employee.public&amp;field=avatar_128&amp;id=#{userId}" t-att-title="userName" height="80" t-att-alt="userName"/>
        </div>
    </t>

    <t t-name="HrAttendanceCheckInOutButtons">
        <div class="flex-grow-1">
            <button t-attf-class="o_hr_attendance_sign_in_out_icon btn btn-{{ checked_in ? 'warning' : 'success' }} align-self-center px-5 py-3 mt-4 mb-2">
                <span class="align-middle fs-2 me-3 text-white" t-if="!checked_in">Check IN</span>
                <i t-attf-class="fa fa-4x fa-sign-{{ checked_in ? 'out' : 'in' }} align-middle"/>
                <span class="align-middle fs-2 ms-3" t-if="checked_in">Check OUT</span>
            </button>
        </div>
    </t>

    <t t-name="HrAttendanceKioskMode">
        <t t-call="HrAttendanceCardLayout">
            <t t-set="kioskModeClasses" t-translation="off">o_barcode_main pt-5</t>
            <t t-set="bodyContent">
                <h2 class="mb-2"><small>Welcome to</small> <t t-esc="widget.company_name"/></h2>
                <img t-attf-src="{{widget.company_image_url}}" alt="Company Logo" class="o_hr_attendance_kiosk_company_image align-self-center img img-fluid mb-3" width="200"/>
                <div class="o_hr_attendance_kiosk_welcome_row d-flex flex-column pb-5">
                    <div class="col-md-5 mt-5 mb-5 mb-md-0 align-self-center" t-if="widget.kiosk_mode != 'manual'">
                        <img src="/barcodes/static/img/barcode.png" alt="Barcode" style="width: 115px;height: 60px"/>
                        <h6 class="mt-2 text-muted">Scan your badge</h6>
                    </div>
                    <div class="mt-5 align-self-end" t-if="widget.kiosk_mode == 'barcode_manual'">
                        <button class="o_hr_attendance_button_employees btn btn-link">
                            Identify Manually
                        </button>
                    </div>
                    <div class="mt-5 align-self-center" t-if="widget.kiosk_mode == 'manual'">
                        <button class="o_hr_attendance_button_employees btn btn-primary px-5 py-3 mt-4 mb-2">
                            <span class="fs-2">Identify Manually</span>
                        </button>
                    </div>
                </div>
            </t>
        </t>
    </t>

    <t t-name="HrAttendanceMyMainMenu">
        <t t-call="HrAttendanceCardLayout">
            <t t-set="bodyContent">
                <t t-if="widget.employee">
                    <t t-set="checked_in" t-value="widget.employee.attendance_state=='checked_in'"/>

                    <t t-call="HrAttendanceUserBadge">
                        <t t-set="userId" t-value="widget.employee.id"/>
                        <t t-set="userName" t-value="widget.employee.name"/>
                    </t>

                    <div class="flex-grow-1">
                        <h1 class="mt-5" t-esc="widget.employee.name"/>
                        <h3><t t-if="!checked_in">Welcome!</t><t t-else="">Want to check out?</t></h3>
                        <h4 class="mt0 mb0 text-muted" t-if="checked_in">Today's work hours: <span t-esc="widget.hours_today"/></h4>
                    </div>

                    <t t-call="HrAttendanceCheckInOutButtons"/>
                </t>
                <div class="alert alert-warning" t-else="">
                    <b>Warning</b> : Your user should be linked to an employee to use attendance.<br/> Please contact your administrator.
                </div>
            </t>
        </t>
    </t>

    <t t-name="HrAttendanceKioskConfirm">
        <t t-call="HrAttendanceCardLayout">
            <t t-set="bodyContent">
                <t t-set="checked_in" t-value="widget.employee_state=='checked_in'"/>

                <button class="o_hr_attendance_back_button btn btn-block btn-secondary btn-lg d-block d-md-none py-5">
                    <i class="fa fa-chevron-left me-2"/> Go back
                </button>

                <t t-if="widget.employee_id" t-call="HrAttendanceUserBadge">
                    <t t-set="userId" t-value="widget.employee_id"/>
                    <t t-set="userName" t-value="widget.employee_name"/>
                </t>

                <button class="o_hr_attendance_back_button o_hr_attendance_back_button_md btn btn-secondary d-none d-md-inline-flex align-items-center position-absolute top-0 start-0 rounded-circle">
                    <i class="fa fa-2x fa-fw fa-chevron-left me-1" role="img" aria-label="Go back" title="Go back"/>
                </button>

                <div t-if="widget.employee_id" class="flex-grow-1">
                    <h1 class="mt-5 mb8"><t t-esc="widget.employee_name"/></h1>
                    <h3 class="mt8 mb24"><t t-if="!checked_in">Welcome!</t><t t-else="">Want to check out?</t></h3>
                    <h4 class="mt0 mb0 text-muted" t-if="checked_in">Today's work hours: <span t-esc="widget.employee_hours_today"/></h4>

                    <t t-if="!widget.use_pin" t-call="HrAttendanceCheckInOutButtons"/>

                    <t t-else="">
                        <h3 class="mt-4 mb0 text-muted">Please enter your PIN to <b t-if="checked_in">check out</b><b t-else="">check in</b></h3>
                        <div class="row">
                            <div class="col-md-8 offset-md-2 o_hr_attendance_pin_pad">
                                <div class="row g-0" >
                                    <div class="col-12 mb8 mt8">
                                        <input class="o_hr_attendance_PINbox border-0 bg-white fs-1 text-center" type="password" disabled="true"/>
                                    </div>
                                </div>
                                <div class="row g-0">
                                    <t t-foreach="['1', '2', '3', '4', '5', '6', '7', '8', '9', ['C', 'btn-warning'], '0', ['ok', 'btn-primary']]" t-as="btn_name">
                                        <div class="col-4 p-1">
                                            <a href="#" t-attf-class="o_hr_attendance_PINbox_button btn {{btn_name[1]? btn_name[1] : 'btn-secondary border'}} btn-block btn-lg {{ 'o_hr_attendance_pin_pad_button_' + btn_name[0] }} d-flex align-items-center justify-content-center">
                                                <t t-esc="btn_name[0]"/>
                                            </a>
                                        </div>
                                    </t>
                                </div>
                            </div>
                        </div>
                    </t>
                </div>
                <div t-else="" class="alert alert-danger mx-3" role="alert">
                    <h4 class="alert-heading">Error: could not find corresponding employee.</h4>
                    <p>Please return to the main menu.</p>
                </div>
                <a role="button" class="oe_attendance_sign_in_out" aria-label="Sign out" title="Sign out"/>
            </t>
        </t>
    </t>

    <t t-name="HrAttendanceGreetingMessage">
        <t t-call="HrAttendanceCardLayout">
            <t t-set="bodyContent">
                <t t-set="checked_in" t-value="widget.employee_state=='checked_in'"/>

                <t t-if="widget.attendance">
                    <t t-call="HrAttendanceUserBadge">
                        <t t-set="userId" t-value="widget.attendance.employee_id[0]"/>
                        <t t-set="userName" t-value="widget.employee_name"/>
                    </t>

                    <div t-if="widget.attendance.check_out" class="flex-grow-1">
                        <h1 class="mt-5">Goodbye <t t-esc="widget.employee_name"/>!</h1>
                        <h2 class="o_hr_attendance_message_message mt4 mb24"/>
                        <div class="alert alert-info fs-2 mx-3" role="status">
                            Checked out at <b><t t-esc="widget.attendance.check_out_time"/></b>
                            <br/><b><t t-esc="widget.hours_today"/></b>
                        </div>
                        <t t-if="widget.show_total_overtime">
                            <div t-att-class="'alert ' + (widget.today_overtime_float &gt;= 0 ? 'alert-success' : 'alert-danger') + ' h3 mx-3'" role="status">
                                Extra hours today:
                                <span t-esc="widget.today_overtime"/>
                            </div>
                            <t t-if="widget.total_overtime_float &gt; 0">
                                Total extra hours: <span t-esc="widget.total_overtime"/>
                            </t>
                        </t>
                        <h3 class="o_hr_attendance_random_message fst-italic mb24"/>
                        <div class="o_hr_attendance_warning_message mt24 alert alert-warning" style="display:none" role="alert"/>
                    </div>
                    <div t-else="" class="flex-grow-1">
                        <h1 class="mt-5 mb0">Welcome <t t-esc="widget.employee_name"/>!</h1>
                        <h2 class="o_hr_attendance_message_message mt4 mb24"/>
                        <div class="alert alert-info fs-2 mx-3" role="status">
                            Checked in at <b><t t-esc="widget.attendance.check_in_time"/></b>
                        </div>
                        <h3 class="o_hr_attendance_random_message mb24"/>
                        <div class="o_hr_attendance_warning_message mt24 alert alert-warning" style="display:none" role="alert"/>
                    </div>
                    <div class="flex-grow-1">
                        <button class="o_hr_attendance_button_dismiss align-self-center btn btn-primary btn-lg px-5 py-3">
                            <span class="fs-2" t-if="widget.attendance.check_out">Goodbye</span>
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
                        <button class="o_hr_attendance_button_dismiss btn btn-primary btn-lg fs-2 px-5 py-3">
                            <i class="fa fa-chevron-left me-2"/>
                            <span class="fs-2">Go back</span>
                        </button>
                    </div>
                </t>
            </t>
        </t>
    </t>
</template>

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
            <tree string="Employee attendances" editable="bottom" sample="1">
                <field name="employee_id"/>
                <field name="check_in"/>
                <field name="check_out"/>
                <field name="worked_hours" string="Work Hours" widget="float_time"/>
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
                            <div>
                                <img t-att-src="kanban_image('hr.employee', 'avatar_128', record.employee_id.raw_value)" t-att-title="record.employee_id.value" t-att-alt="record.employee_id.value" class="oe_kanban_avatar o_image_24_cover mr4"/>
                                <span class="o_kanban_record_title">
                                    <strong><t t-esc="record.employee_id.value"/></strong>
                                </span>
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
            <form string="Employee attendances">
                <sheet>
                    <group>
                        <field name="employee_id"/>
                        <field name="check_in"/>
                        <field name="check_out"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="hr_attendance_view_filter" model="ir.ui.view">
        <field name="name">hr_attendance_view_filter</field>
        <field name="model">hr.attendance</field>
        <field name="arch" type="xml">
            <search string="Hr Attendance Search">
                <field name="employee_id"/>
                <field name="department_id" operator="child_of"/>
                <filter string="My Attendances" name="myattendances" domain="[('employee_id.user_id', '=', uid)]" />
                <separator/>
                <filter string="Check In" name="check_in_filter" date="check_in" default_period="last_month"/>
                <filter string="No Check Out" name="nocheckout" domain="[('check_out', '=', False)]" />
                <group expand="0" string="Group By">
                    <filter string="Employee" name="employee" context="{'group_by': 'employee_id'}"/>
                    <filter string="Check In" name="groupby_name" context="{'group_by': 'check_in'}"/>
                    <filter string="Check Out" name="groupby_check_out" context="{'group_by': 'check_out'}"/>
                </group>
            </search>
        </field>
    </record>

    <!-- actions -->

    <record id="hr_attendance_action" model="ir.actions.act_window">
        <field name="name">Attendances</field>
        <field name="res_model">hr.attendance</field>
        <field name="view_mode">tree,kanban,form</field>
        <field name="context">{"search_default_today":1}</field>
        <field name="search_view_id" ref="hr_attendance_view_filter" />
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                No attendance records found
            </p><p>
                The attendance records of your employees will be displayed here.
            </p>
        </field>
    </record>

    <record id="hr_attendance_action_employee" model="ir.actions.act_window">
        <field name="name">Attendances</field>
        <field name="res_model">hr.attendance</field>
        <field name="view_mode">tree,form</field>
        <field name="context">{'create': False}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                No attendance records to display
            </p><p>
                The attendance records of your employees will be displayed here.
            </p>
        </field>
    </record>

    <record id="hr_attendance_action_overview" model="ir.actions.act_window">
        <field name="name">Attendances</field>
        <field name="res_model">hr.attendance</field>
        <field name="view_mode">tree</field>
        <field name="context">{'create': False}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                No attendance records to display
            </p><p>
                Your attendance records will be displayed here.
            </p>
        </field>
    </record>

    <record id="hr_attendance_action_kiosk_mode" model="ir.actions.client">
        <field name="name">Attendances</field>
        <field name="tag">hr_attendance_kiosk_mode</field>
        <field name="target">fullscreen</field>
    </record>

    <record id="hr_attendance_action_my_attendances" model="ir.actions.client">
        <field name="name">Attendance</field>
        <field name="tag">hr_attendance_my_attendances</field>
        <field name="target">main</field>
    </record>

    <record id="hr_attendance_action_greeting_message" model="ir.actions.client">
        <field name="name">Message</field>
        <field name="tag">hr_attendance_greeting_message</field>
    </record>

    <!-- Menus -->

    <menuitem id="menu_hr_attendance_root" name="Attendances" sequence="205" groups="hr_attendance.group_hr_attendance,hr_attendance.group_hr_attendance_kiosk" web_icon="hr_attendance,static/description/icon.svg"/>

    <menuitem id="menu_hr_attendance_my_attendances" name="Check In / Check Out" parent="menu_hr_attendance_root" sequence="1" groups="hr_attendance.group_hr_attendance" action="hr_attendance_action_my_attendances"/>

    <menuitem id="menu_hr_attendance_attendances_overview" name="Attendances" parent="menu_hr_attendance_root" sequence="1" groups="hr_attendance.group_hr_attendance" action="hr_attendance_action_overview"/>

    <menuitem id="menu_hr_attendance_kiosk_no_user_mode" name="Kiosk Mode" parent="menu_hr_attendance_root" sequence="10" groups="hr_attendance.group_hr_attendance_kiosk" action="hr_attendance_action_kiosk_mode"/>

    <menuitem id="menu_hr_attendance_view_attendances" name="Attendances" parent="menu_hr_attendance_root" sequence="10" groups="hr_attendance.group_hr_attendance_user" action="hr_attendance_action"/>
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
                    <a role="menuitem" class="dropdown-item" name="%(hr_attendance_report_action_filtered)d" type="action" groups="hr_attendance.group_hr_attendance_user">
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
                <button name="%(hr_attendance_action)d"
                        class="oe_stat_button"
                        icon="fa-clock-o"
                        type="action"
                        context="{'search_default_employee_id': id, 'search_default_check_in_filter': '1'}"
                        groups="hr_attendance.group_hr_attendance_user"
                        help="Worked hours last month">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_value">
                            <field name="hours_last_month_display" widget="float_time"/> Hours
                        </span>
                        <span class="o_stat_text">
                            Last Month
                        </span>
                    </div>
                </button>
                <button name="%(hr_attendance_overtime_action)d"
                        class="oe_stat_button"
                        icon="fa-history"
                        type="action"
                        attrs="{'invisible': [('total_overtime', '=', 0.0)]}"
                        context="{'search_default_employee_id': active_id}"
                        groups="hr_attendance.group_hr_attendance_user">
                    <div class="o_stat_info">
                        <span class="o_stat_value text-success" attrs="{'invisible': [('total_overtime', '&lt;', 0)]}">
                            <field name="total_overtime" widget="float_time"/>
                        </span>
                        <span class="o_stat_value text-danger" attrs="{'invisible': [('total_overtime', '&gt;=', 0)]}">
                            <field name="total_overtime" widget="float_time"/>
                        </span>
                        <span class="o_stat_text">Extra Hours</span>
                    </div>
                </button>
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
                <button name="%(hr_attendance_action_employee)d"
                        class="oe_stat_button"
                        icon="fa-calendar"
                        type="action"
                        context="{'search_default_employee_id': employee_ids, 'search_default_check_in_filter': '1'}"
                        groups="base.group_user"
                        help="Worked hours last month">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_value">
                            <field name="hours_last_month_display" widget="float_time"/> Hours
                        </span>
                        <span class="o_stat_text">
                            Last Month
                        </span>
                    </div>
                </button>
            </xpath>
            <xpath expr="//div[@name='button_box']" position="inside">
                <button name="%(hr_attendance_overtime_action)d"
                        class="oe_stat_button"
                        icon="fa-history"
                        type="action"
                        attrs="{'invisible': [('total_overtime', '=', 0.0)]}"
                        context="{'search_default_employee_id': employee_ids}"
                        groups="base.group_user"
                        help="Amount of extra hours">
                    <div class="o_stat_info">
                        <span class="o_stat_value text-success" attrs="{'invisible': [('total_overtime', '&lt;', 0)]}">
                            <field name="total_overtime" widget="float_time"/>
                        </span>
                        <span class="o_stat_value text-danger" attrs="{'invisible': [('total_overtime', '&gt;=', 0)]}">
                            <field name="total_overtime" widget="float_time"/>
                        </span>
                        <span class="o_stat_text">Extra Hours</span>
                    </div>
                </button>
            </xpath>
        </field>
    </record>

    <!-- employee kanban view specifically for hr_attendance (to check in/out) -->
    <record id="hr_employees_view_kanban" model="ir.ui.view">
        <field name="name">hr.employee.kanban</field>
        <field name="model">hr.employee</field>
        <field name="priority">99</field>
        <field name="arch" type="xml">
            <kanban class="o_hr_employee_attendance_kanban" create="false" action="action_employee_kiosk_confirm" type="object">
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
            <xpath expr="//div[hasclass('settings')]" position="inside">
                <div class="app_settings_block" data-string="Attendances" string="Attendances" data-key="hr_attendance" groups="hr_attendance.group_hr_attendance_manager">
                    <h2>Check-In/Out in Kiosk Mode</h2>
                    <div class="row mt16 o_settings_container" name="kiosk_mode_setting_container">
                        <div class="col-12 col-lg-6 o_setting_box">
                            <div class="o_setting_right_pane">
                                <label for="attendance_kiosk_mode"/>
                                <span class="fa fa-lg fa-building-o" title="Values set here are company-specific." role="img" aria-label="Values set here are company-specific." groups="base.group_multi_company"/>
                                <div class="row">
                                    <div class="text-muted col-lg-10">
                                        Define the way the user will be identified by the application.
                                    </div>
                                </div>
                                <div class="content-group">
                                    <div class="mt16">
                                        <field name="attendance_kiosk_mode" required="1" class="w-75"/>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box">
                            <div class="o_setting_right_pane">
                                <label for="attendance_kiosk_delay" string="Display Time"/>
                                <span class="fa fa-lg fa-building-o" title="Values set here are company-specific." role="img" aria-label="Values set here are company-specific." groups="base.group_multi_company"/>
                                <div class="row">
                                    <div class="text-muted col-lg-10">
                                        Choose how long the greeting message will be displayed.
                                    </div>
                                </div>
                                <div class="content-group">
                                    <div class="mt16">
                                        <field name="attendance_kiosk_delay" required="1" class="text-center" style="width: 10%; min-width: 4rem;"/> seconds
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="row mt16 o_settings_container" name="pincode_setting_container">
                        <div class="col-12 col-lg-6 o_setting_box" attrs="{'invisible': [('attendance_kiosk_mode', '=', 'manual')]}">
                            <div class="o_setting_right_pane">
                                <label for="attendance_barcode_source"/>
                                <span class="fa fa-lg fa-building-o" title="Values set here are company-specific." role="img" aria-label="Values set here are company-specific." groups="base.group_multi_company"/>
                                <div class="row">
                                    <div class="text-muted col-lg-10">
                                        Define the camera used for the barcode scan.
                                    </div>
                                </div>
                                <div class="content-group">
                                    <div class="mt16">
                                        <field name="attendance_barcode_source" required="1"/>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box" title="Set PIN codes in the employee detail form (in HR Settings tab)." attrs="{'invisible': [('attendance_kiosk_mode', '=', 'barcode')]}">
                            <div class="o_setting_left_pane">
                                <field name="group_attendance_use_pin"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="group_attendance_use_pin"/>
                                <div class="text-muted col-lg-10">
                                    Use PIN codes (defined on the Employee's profile) to check-in.
                                </div>
                            </div>
                        </div>
                    </div>
                    <h2>Extra Hours</h2>
                    <div class="row mt16 o_settings_container" name="overtime_settings">
                        <div class="col-12 col-lg-6 o_setting_box">
                            <div class="o_setting_left_pane" title="Activate the count of employees' extra hours.">
                                <field name="hr_attendance_overtime"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="hr_attendance_overtime" class="o_form_label">Count of Extra Hours</label>
                                <span class="fa fa-lg fa-building-o" title="Values set here are company-specific." role="img" aria-label="Values set here are company-specific." groups="base.group_multi_company"/>
                                <div class="text-muted">
                                    Compare attendance with working hours set on employee.
                                </div>
                                <div class="mt16" attrs="{'invisible': [('hr_attendance_overtime', '=', False)],
                                                          'required': [('hr_attendance_overtime', '=', True)]}">
                                    <div class="mt16 row" title="Count of extra hours is considered from this date. Potential extra hours prior to this date are not considered.">
                                        <label for="overtime_start_date" string="Start from" class="col-3 col-lg-3 o_light_label"/>
                                        <field name="overtime_start_date" class="col-lg-3 p-0" attrs="{'required': [('hr_attendance_overtime', '=', True)]}" />
                                    </div>
                                    <br/>
                                    <label for="overtime_company_threshold" class="o_form_label">
                                        Tolerance Time In Favor Of Company
                                    </label>
                                    <div class="text-muted">
                                        Allow a period of time (around working hours) where extra time will not be counted, in benefit of the company
                                    </div>
                                    <span>Time Period </span><field name="overtime_company_threshold" class="text-center"
                                        attrs="{'required': [('hr_attendance_overtime', '=', True)]}" style="width: 10%; min-width: 4rem;"/><span> Minutes</span>
                                    <br/>
                                    <br/>
                                    <label for="overtime_employee_threshold" class="o_form_label">
                                        Tolerance Time In Favor Of Employee
                                    </label>
                                    <div class="text-muted">
                                        Allow a period of time (around working hours) where extra time will not be deducted, in benefit of the employee
                                    </div>
                                    <span>Time Period </span><field name="overtime_employee_threshold" class="text-center"
                                        attrs="{'required': [('hr_attendance_overtime', '=', True)]}" style="width: 10%; min-width: 4rem;"/><span> Minutes</span>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </xpath>
        </field>
    </record>

    <record id="action_hr_attendance_settings" model="ir.actions.act_window">
        <field name="name">Settings</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">res.config.settings</field>
        <field name="view_mode">form</field>
        <field name="target">inline</field>
        <field name="context">{'module' : 'hr_attendance', 'bin_size': False}</field>
    </record>

    <menuitem id="hr_attendance.menu_hr_attendance_settings" name="Configuration" parent="menu_hr_attendance_root"
        sequence="99" action="action_hr_attendance_settings" groups="hr_attendance.group_hr_attendance_manager"/>
</odoo>

```

