# Odoo Module: resource

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Resource',
    'version': '1.1',
    'category': 'Hidden',
    'description': """
Module for resource management.
===============================

A resource represent something that can be scheduled (a developer on a task or a
work center on manufacturing orders). This module manages a resource calendar
associated to every resource. It also manages the leaves of every resource.
    """,
    'depends': ['base', 'web'],
    'data': [
        'data/resource_data.xml',
        'security/ir.model.access.csv',
        'security/resource_security.xml',
        'views/resource_resource_views.xml',
        'views/resource_calendar_leaves_views.xml',
        'views/resource_calendar_attendance_views.xml',
        'views/resource_calendar_views.xml',
        'views/menuitems.xml',
    ],
    'demo': [
        'data/resource_demo.xml',
    ],
    'assets': {
        'web.assets_backend': [
            'resource/static/src/**/*',
        ],
        'web.assets_unit_tests': [
            'resource/static/tests/**/*',
            ('remove', 'resource/static/tests/components/**/*'),
        ],
        'web.qunit_suite_tests': [
            'resource/static/tests/components/*.js',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\resource_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="resource_calendar_std" model="resource.calendar">
            <field name="name">Standard 40 hours/week</field>
            <field name="company_id" ref="base.main_company"/>
            <field name="full_time_required_hours">40</field>
        </record>

        <record id="base.main_company" model="res.company">
            <field name="resource_calendar_id" ref="resource_calendar_std"/>
        </record>

        <function
            model="res.company"
            name="_init_data_resource_calendar"
            eval="[]"/>
    </data>
</odoo>

```

## File: data\resource_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="resource_calendar_std_35h" model="resource.calendar">
            <field name="name">Standard 35 hours/week</field>
            <field name="company_id" ref="base.main_company"/>
            <field name="attendance_ids"
                eval="[(5, 0, 0),
                    (0, 0, {'name': 'Monday Morning', 'dayofweek': '0', 'hour_from': 8, 'hour_to': 12, 'day_period': 'morning'}),
                    (0, 0, {'name': 'Monday Lunch', 'dayofweek': '0', 'hour_from': 12, 'hour_to': 13, 'day_period': 'lunch'}),
                    (0, 0, {'name': 'Monday Afternoon', 'dayofweek': '0', 'hour_from': 13, 'hour_to': 16, 'day_period': 'afternoon'}),
                    (0, 0, {'name': 'Tuesday Morning', 'dayofweek': '1', 'hour_from': 8, 'hour_to': 12, 'day_period': 'morning'}),
                    (0, 0, {'name': 'Tuesday Lunch', 'dayofweek': '1', 'hour_from': 12, 'hour_to': 13, 'day_period': 'lunch'}),
                    (0, 0, {'name': 'Tuesday Afternoon', 'dayofweek': '1', 'hour_from': 13, 'hour_to': 16, 'day_period': 'afternoon'}),
                    (0, 0, {'name': 'Wednesday Morning', 'dayofweek': '2', 'hour_from': 8, 'hour_to': 12, 'day_period': 'morning'}),
                    (0, 0, {'name': 'Wednesday Lunch', 'dayofweek': '2', 'hour_from': 12, 'hour_to': 13, 'day_period': 'lunch'}),
                    (0, 0, {'name': 'Wednesday Afternoon', 'dayofweek': '2', 'hour_from': 13, 'hour_to': 16, 'day_period': 'afternoon'}),
                    (0, 0, {'name': 'Thursday Morning', 'dayofweek': '3', 'hour_from': 8, 'hour_to': 12, 'day_period': 'morning'}),
                    (0, 0, {'name': 'Thursday Lunch', 'dayofweek': '3', 'hour_from': 12, 'hour_to': 13, 'day_period': 'lunch'}),
                    (0, 0, {'name': 'Thursday Afternoon', 'dayofweek': '3', 'hour_from': 13, 'hour_to': 16, 'day_period': 'afternoon'}),
                    (0, 0, {'name': 'Friday Morning', 'dayofweek': '4', 'hour_from': 8, 'hour_to': 12, 'day_period': 'morning'}),
                    (0, 0, {'name': 'Friday Lunch', 'dayofweek': '4', 'hour_from': 12, 'hour_to': 13, 'day_period': 'lunch'}),
                    (0, 0, {'name': 'Friday Afternoon', 'dayofweek': '4', 'hour_from': 13, 'hour_to': 16, 'day_period': 'afternoon'})
                ]"
            />
            <field name="full_time_required_hours">35</field>
        </record>

        <record id="resource_calendar_std_38h" model="resource.calendar">
            <field name="name">Standard 38 hours/week</field>
            <field name="company_id" eval="False"/>
            <field name="hours_per_day">7.6</field>
            <field name="attendance_ids"
                eval="[(5, 0, 0),
                    (0, 0, {'name': 'Monday Morning', 'dayofweek': '0', 'hour_from': 8, 'hour_to': 12, 'day_period': 'morning'}),
                    (0, 0, {'name': 'Monday Lunch', 'dayofweek': '0', 'hour_from': 12, 'hour_to': 13, 'day_period': 'lunch'}),
                    (0, 0, {'name': 'Monday Afternoon', 'dayofweek': '0', 'hour_from': 13, 'hour_to': 16.6, 'day_period': 'afternoon'}),
                    (0, 0, {'name': 'Tuesday Morning', 'dayofweek': '1', 'hour_from': 8, 'hour_to': 12, 'day_period': 'morning'}),
                    (0, 0, {'name': 'Tuesday Lunch', 'dayofweek': '1', 'hour_from': 12, 'hour_to': 13, 'day_period': 'lunch'}),
                    (0, 0, {'name': 'Tuesday Afternoon', 'dayofweek': '1', 'hour_from': 13, 'hour_to': 16.6, 'day_period': 'afternoon'}),
                    (0, 0, {'name': 'Wednesday Morning', 'dayofweek': '2', 'hour_from': 8, 'hour_to': 12, 'day_period': 'morning'}),
                    (0, 0, {'name': 'Wednesday Lunch', 'dayofweek': '2', 'hour_from': 12, 'hour_to': 13, 'day_period': 'lunch'}),
                    (0, 0, {'name': 'Wednesday Afternoon', 'dayofweek': '2', 'hour_from': 13, 'hour_to': 16.6, 'day_period': 'afternoon'}),
                    (0, 0, {'name': 'Thursday Morning', 'dayofweek': '3', 'hour_from': 8, 'hour_to': 12, 'day_period': 'morning'}),
                    (0, 0, {'name': 'Thursday Lunch', 'dayofweek': '3', 'hour_from': 12, 'hour_to': 13, 'day_period': 'lunch'}),
                    (0, 0, {'name': 'Thursday Afternoon', 'dayofweek': '3', 'hour_from': 13, 'hour_to': 16.6, 'day_period': 'afternoon'}),
                    (0, 0, {'name': 'Friday Morning', 'dayofweek': '4', 'hour_from': 8, 'hour_to': 12, 'day_period': 'morning'}),
                    (0, 0, {'name': 'Friday Lunch', 'dayofweek': '4', 'hour_from': 12, 'hour_to': 13, 'day_period': 'lunch'}),
                    (0, 0, {'name': 'Friday Afternoon', 'dayofweek': '4', 'hour_from': 13, 'hour_to': 16.6, 'day_period': 'afternoon'})
                ]"
            />
            <field name="full_time_required_hours">38</field>
        </record>

        <record id="resource_calendar_flex_40h" model="resource.calendar">
            <field name="name">Flexible 40 hours/week</field>
            <field name="company_id" ref="base.main_company"/>
            <field name="hours_per_day">8</field>
            <field name="flexible_hours" eval="True"/>
            <field name="full_time_required_hours">40</field>
        </record>

    </data>
</odoo>

```

## File: models\resource_calendar.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import itertools

from collections import defaultdict
from datetime import datetime, timedelta, time
from functools import partial
from itertools import chain

from dateutil.relativedelta import relativedelta
from dateutil.rrule import rrule, DAILY
from pytz import timezone, utc

from odoo import api, fields, models, _
from odoo.addons.base.models.res_partner import _tz_get
from odoo.exceptions import ValidationError
from odoo.osv import expression
from odoo.tools.float_utils import float_round

from odoo.tools import date_utils, ormcache
from .utils import Intervals, float_to_time, make_aware, datetime_to_string, string_to_datetime
from odoo.addons.hr_work_entry_contract.models.hr_work_intervals import WorkIntervals


class ResourceCalendar(models.Model):
    """ Calendar model for a resource. It has

     - attendance_ids: list of resource.calendar.attendance that are a working
                       interval in a given weekday.
     - leave_ids: list of leaves linked to this calendar. A leave can be general
                  or linked to a specific resource, depending on its resource_id.

    All methods in this class use intervals. An interval is a tuple holding
    (begin_datetime, end_datetime). A list of intervals is therefore a list of
    tuples, holding several intervals of work or leaves. """
    _name = "resource.calendar"
    _description = "Resource Working Time"

    @api.model
    def default_get(self, fields):
        res = super().default_get(fields)
        if not res.get('name') and res.get('company_id'):
            res['name'] = _('Working Hours of %s', self.env['res.company'].browse(res['company_id']).name)
        if 'attendance_ids' in fields and not res.get('attendance_ids'):
            company_id = res.get('company_id', self.env.company.id)
            company = self.env['res.company'].browse(company_id)
            company_attendance_ids = company.resource_calendar_id.attendance_ids
            if not company.resource_calendar_id.two_weeks_calendar and company_attendance_ids:
                res['attendance_ids'] = [
                    (0, 0, {
                        'name': attendance.name,
                        'dayofweek': attendance.dayofweek,
                        'hour_from': attendance.hour_from,
                        'hour_to': attendance.hour_to,
                        'day_period': attendance.day_period,
                        'date_from': attendance.date_from,
                        'date_to': attendance.date_to,
                    })
                    for attendance in company_attendance_ids
                ]
            else:
                res['attendance_ids'] = [
                    (0, 0, {'name': _('Monday Morning'), 'dayofweek': '0', 'hour_from': 8, 'hour_to': 12, 'day_period': 'morning'}),
                    (0, 0, {'name': _('Monday Lunch'), 'dayofweek': '0', 'hour_from': 12, 'hour_to': 13, 'day_period': 'lunch'}),
                    (0, 0, {'name': _('Monday Afternoon'), 'dayofweek': '0', 'hour_from': 13, 'hour_to': 17, 'day_period': 'afternoon'}),
                    (0, 0, {'name': _('Tuesday Morning'), 'dayofweek': '1', 'hour_from': 8, 'hour_to': 12, 'day_period': 'morning'}),
                    (0, 0, {'name': _('Tuesday Lunch'), 'dayofweek': '1', 'hour_from': 12, 'hour_to': 13, 'day_period': 'lunch'}),
                    (0, 0, {'name': _('Tuesday Afternoon'), 'dayofweek': '1', 'hour_from': 13, 'hour_to': 17, 'day_period': 'afternoon'}),
                    (0, 0, {'name': _('Wednesday Morning'), 'dayofweek': '2', 'hour_from': 8, 'hour_to': 12, 'day_period': 'morning'}),
                    (0, 0, {'name': _('Wednesday Lunch'), 'dayofweek': '2', 'hour_from': 12, 'hour_to': 13, 'day_period': 'lunch'}),
                    (0, 0, {'name': _('Wednesday Afternoon'), 'dayofweek': '2', 'hour_from': 13, 'hour_to': 17, 'day_period': 'afternoon'}),
                    (0, 0, {'name': _('Thursday Morning'), 'dayofweek': '3', 'hour_from': 8, 'hour_to': 12, 'day_period': 'morning'}),
                    (0, 0, {'name': _('Thursday Lunch'), 'dayofweek': '3', 'hour_from': 12, 'hour_to': 13, 'day_period': 'lunch'}),
                    (0, 0, {'name': _('Thursday Afternoon'), 'dayofweek': '3', 'hour_from': 13, 'hour_to': 17, 'day_period': 'afternoon'}),
                    (0, 0, {'name': _('Friday Morning'), 'dayofweek': '4', 'hour_from': 8, 'hour_to': 12, 'day_period': 'morning'}),
                    (0, 0, {'name': _('Friday Lunch'), 'dayofweek': '4', 'hour_from': 12, 'hour_to': 13, 'day_period': 'lunch'}),
                    (0, 0, {'name': _('Friday Afternoon'), 'dayofweek': '4', 'hour_from': 13, 'hour_to': 17, 'day_period': 'afternoon'})
                ]
        return res

    name = fields.Char(required=True)
    active = fields.Boolean("Active", default=True,
                            help="If the active field is set to false, it will allow you to hide the Working Time without removing it.")
    company_id = fields.Many2one(
        'res.company', 'Company', domain=lambda self: [('id', 'in', self.env.companies.ids)],
        default=lambda self: self.env.company)
    attendance_ids = fields.One2many(
        'resource.calendar.attendance', 'calendar_id', 'Working Time',
        compute='_compute_attendance_ids', store=True, readonly=False, copy=True)
    leave_ids = fields.One2many(
        'resource.calendar.leaves', 'calendar_id', 'Time Off')
    global_leave_ids = fields.One2many(
        'resource.calendar.leaves', 'calendar_id', 'Global Time Off',
        compute='_compute_global_leave_ids', store=True, readonly=False,
        domain=[('resource_id', '=', False)], copy=True,
    )
    hours_per_day = fields.Float("Average Hour per Day", store=True, compute="_compute_hours_per_day", digits=(2, 2), readonly=False,
                                 help="Average hours per day a resource is supposed to work with this calendar.")
    tz = fields.Selection(
        _tz_get, string='Timezone', required=True,
        default=lambda self: self._context.get('tz') or self.env.user.tz or self.env.ref('base.user_admin').tz or 'UTC',
        help="This field is used in order to define in which timezone the resources will work.")
    tz_offset = fields.Char(compute='_compute_tz_offset', string='Timezone offset')
    two_weeks_calendar = fields.Boolean(string="Calendar in 2 weeks mode")
    two_weeks_explanation = fields.Char('Explanation', compute="_compute_two_weeks_explanation")
    flexible_hours = fields.Boolean(string="Flexible Hours",
                                    help="When enabled, it will allow employees to work flexibly, without relying on the company's working schedule (working hours).")
    full_time_required_hours = fields.Float(string="Company Full Time", help="Number of hours to work on the company schedule to be considered as fulltime.")

    @api.depends('attendance_ids', 'attendance_ids.hour_from', 'attendance_ids.hour_to', 'two_weeks_calendar', 'flexible_hours')
    def _compute_hours_per_day(self):
        for calendar in self:
            if calendar.flexible_hours:
                continue
            attendances = calendar._get_global_attendances()
            calendar.hours_per_day = calendar._get_hours_per_day(attendances)

    @api.depends('company_id')
    def _compute_attendance_ids(self):
        for calendar in self.filtered(lambda c: not c._origin or c._origin.company_id != c.company_id and c.company_id):
            company_calendar = calendar.company_id.resource_calendar_id
            calendar.update({
                'two_weeks_calendar': company_calendar.two_weeks_calendar,
                'tz': company_calendar.tz,
                'attendance_ids': [(5, 0, 0)] + [
                    (0, 0, attendance._copy_attendance_vals()) for attendance in company_calendar.attendance_ids if not attendance.resource_id]
            })

    @api.depends('company_id')
    def _compute_global_leave_ids(self):
        for calendar in self.filtered(lambda c: not c._origin or c._origin.company_id != c.company_id):
            calendar.update({
                'global_leave_ids': [(5, 0, 0)] + [
                    (0, 0, leave._copy_leave_vals()) for leave in calendar.company_id.resource_calendar_id.global_leave_ids]
            })

    @api.depends('tz')
    def _compute_tz_offset(self):
        for calendar in self:
            calendar.tz_offset = datetime.now(timezone(calendar.tz or 'GMT')).strftime('%z')

    def copy_data(self, default=None):
        vals_list = super().copy_data(default=default)
        return [dict(vals, name=self.env._("%s (copy)", calendar.name)) for calendar, vals in zip(self, vals_list)]

    @api.constrains('attendance_ids')
    def _check_attendance_ids(self):
        for resource in self:
            if (resource.two_weeks_calendar and
                    resource.attendance_ids.filtered(lambda a: a.display_type == 'line_section') and
                    not resource.attendance_ids.sorted('sequence')[0].display_type):
                raise ValidationError(_("In a calendar with 2 weeks mode, all periods need to be in the sections."))

    @api.depends('two_weeks_calendar')
    def _compute_two_weeks_explanation(self):
        today = fields.Date.today()
        week_type = self.env['resource.calendar.attendance'].get_week_type(today)
        week_type_str = _("second") if week_type else _("first")
        first_day = date_utils.start_of(today, 'week')
        last_day = date_utils.end_of(today, 'week')
        self.two_weeks_explanation = _(
            "The current week (from %(first_day)s to %(last_day)s) corresponds to week number %(number)s.",
            first_day=first_day,
            last_day=last_day,
            number=week_type_str,
        )

    def _get_global_attendances(self):
        return self.attendance_ids.filtered(lambda attendance:
            attendance.day_period != 'lunch'
            and not attendance.date_from and not attendance.date_to
            and not attendance.resource_id and not attendance.display_type)

    def _get_hours_per_day(self, attendances):
        """
        Calculate the average hours worked per workday.
        """
        if not attendances:
            return 0

        hour_count = 0.0
        for attendance in attendances:
            hour_count += attendance.hour_to - attendance.hour_from

        if self.two_weeks_calendar:
            number_of_days = len(set(attendances.filtered(lambda cal: cal.week_type == '1').mapped('dayofweek')))
            number_of_days += len(set(attendances.filtered(lambda cal: cal.week_type == '0').mapped('dayofweek')))
        else:
            number_of_days = len(set(attendances.mapped('dayofweek')))

        if not number_of_days:
            return 0

        return float_round(hour_count / float(number_of_days), precision_digits=2)

    def switch_calendar_type(self):
        if not self.two_weeks_calendar:
            self.attendance_ids.unlink()
            self.attendance_ids = [
                (0, 0, {
                    'name': 'First week',
                    'dayofweek': '0',
                    'sequence': '0',
                    'hour_from': 0,
                    'day_period': 'morning',
                    'week_type': '0',
                    'hour_to': 0,
                    'display_type':
                    'line_section'}),
                (0, 0, {
                    'name': 'Second week',
                    'dayofweek': '0',
                    'sequence': '25',
                    'hour_from': 0,
                    'day_period': 'morning',
                    'week_type': '1',
                    'hour_to': 0,
                    'display_type': 'line_section'}),
            ]

            self.two_weeks_calendar = True
            default_attendance = self.default_get(['attendance_ids'])['attendance_ids']
            for idx, att in enumerate(default_attendance):
                att[2]["week_type"] = '0'
                att[2]["sequence"] = idx + 1
            self.attendance_ids = default_attendance
            for idx, att in enumerate(default_attendance):
                att[2]["week_type"] = '1'
                att[2]["sequence"] = idx + 26
            self.attendance_ids = default_attendance
        else:
            self.two_weeks_calendar = False
            self.attendance_ids.unlink()
            self.attendance_ids = self.default_get(['attendance_ids'])['attendance_ids']

    @api.onchange('attendance_ids')
    def _onchange_attendance_ids(self):
        if not self.two_weeks_calendar:
            return

        even_week_seq = self.attendance_ids.filtered(lambda att: att.display_type == 'line_section' and att.week_type == '0')
        odd_week_seq = self.attendance_ids.filtered(lambda att: att.display_type == 'line_section' and att.week_type == '1')
        if len(even_week_seq) != 1 or len(odd_week_seq) != 1:
            raise ValidationError(_("You can't delete section between weeks."))

        even_week_seq = even_week_seq.sequence
        odd_week_seq = odd_week_seq.sequence

        for line in self.attendance_ids.filtered(lambda att: att.display_type is False):
            if even_week_seq > odd_week_seq:
                line.week_type = '1' if even_week_seq > line.sequence else '0'
            else:
                line.week_type = '0' if odd_week_seq > line.sequence else '1'

    def _check_overlap(self, attendance_ids):
        """ attendance_ids correspond to attendance of a week,
            will check for each day of week that there are no superimpose. """
        result = []
        for attendance in attendance_ids.filtered(lambda att: not att.date_from and not att.date_to):
            # 0.000001 is added to each start hour to avoid to detect two contiguous intervals as superimposing.
            # Indeed Intervals function will join 2 intervals with the start and stop hour corresponding.
            result.append((int(attendance.dayofweek) * 24 + attendance.hour_from + 0.000001, int(attendance.dayofweek) * 24 + attendance.hour_to, attendance))

        if len(Intervals(result)) != len(result):
            raise ValidationError(_("Attendances can't overlap."))

    @api.constrains('attendance_ids')
    def _check_attendance(self):
        # Avoid superimpose in attendance
        for calendar in self:
            attendance_ids = calendar.attendance_ids.filtered(lambda attendance: not attendance.resource_id and attendance.display_type is False)
            if calendar.two_weeks_calendar:
                calendar._check_overlap(attendance_ids.filtered(lambda attendance: attendance.week_type == '0'))
                calendar._check_overlap(attendance_ids.filtered(lambda attendance: attendance.week_type == '1'))
            else:
                calendar._check_overlap(attendance_ids)

    # --------------------------------------------------
    # Computation API
    # --------------------------------------------------

    def _attendance_intervals_batch(self, start_dt, end_dt, resources=None, domain=None, tz=None, lunch=False):
        assert start_dt.tzinfo and end_dt.tzinfo
        self.ensure_one()
        if not resources:
            resources = self.env['resource.resource']
            resources_list = [resources]
        else:
            resources_list = list(resources) + [self.env['resource.resource']]
        resource_ids = [r.id for r in resources_list]
        domain = domain if domain is not None else []
        domain = expression.AND([domain, [
            ('calendar_id', '=', self.id),
            ('resource_id', 'in', resource_ids),
            ('display_type', '=', False),
            ('day_period', '!=' if not lunch else '=', 'lunch'),
        ]])

        attendances = self.env['resource.calendar.attendance'].search(domain)
        # Since we only have one calendar to take in account
        # Group resources per tz they will all have the same result
        resources_per_tz = defaultdict(list)
        for resource in resources_list:
            resources_per_tz[tz or timezone((resource or self).tz)].append(resource)
        # Resource specific attendances
        attendance_per_resource = defaultdict(lambda: self.env['resource.calendar.attendance'])
        # Calendar attendances per day of the week
        # * 7 days per week * 2 for two week calendars
        attendances_per_day = [self.env['resource.calendar.attendance']] * 7 * 2
        weekdays = set()
        for attendance in attendances:
            if attendance.resource_id:
                attendance_per_resource[attendance.resource_id] |= attendance
            weekday = int(attendance.dayofweek)
            weekdays.add(weekday)
            if self.two_weeks_calendar:
                weektype = int(attendance.week_type)
                attendances_per_day[weekday + 7 * weektype] |= attendance
            else:
                attendances_per_day[weekday] |= attendance
                attendances_per_day[weekday + 7] |= attendance

        start = start_dt.astimezone(utc)
        end = end_dt.astimezone(utc)
        bounds_per_tz = {
            tz: (start_dt.astimezone(tz), end_dt.astimezone(tz))
            for tz in resources_per_tz.keys()
        }
        # Use the outer bounds from the requested timezones
        for tz, bounds in bounds_per_tz.items():
            start = min(start, bounds[0].replace(tzinfo=utc))
            end = max(end, bounds[1].replace(tzinfo=utc))
        # Generate once with utc as timezone
        days = rrule(DAILY, start.date(), until=end.date(), byweekday=weekdays)
        ResourceCalendarAttendance = self.env['resource.calendar.attendance']
        base_result = []
        per_resource_result = defaultdict(list)
        for day in days:
            week_type = ResourceCalendarAttendance.get_week_type(day)
            attendances = attendances_per_day[day.weekday() + 7 * week_type]
            for attendance in attendances:
                if (attendance.date_from and day.date() < attendance.date_from) or\
                    (attendance.date_to and attendance.date_to < day.date()):
                    continue
                day_from = datetime.combine(day, float_to_time(attendance.hour_from))
                day_to = datetime.combine(day, float_to_time(attendance.hour_to))
                if attendance.resource_id:
                    per_resource_result[attendance.resource_id].append((day_from, day_to, attendance))
                else:
                    base_result.append((day_from, day_to, attendance))


        # Copy the result localized once per necessary timezone
        # Strictly speaking comparing start_dt < time or start_dt.astimezone(tz) < time
        # should always yield the same result. however while working with dates it is easier
        # if all dates have the same format
        result_per_tz = {
            tz: [(max(bounds_per_tz[tz][0], tz.localize(val[0])),
                min(bounds_per_tz[tz][1], tz.localize(val[1])),
                val[2])
                    for val in base_result]
            for tz in resources_per_tz.keys()
        }
        result_per_resource_id = dict()
        for tz, resources in resources_per_tz.items():
            res = result_per_tz[tz]
            res_intervals = WorkIntervals(res)
            for resource in resources:
                if resource and resource._is_flexible():
                    duration_days = (end - start).days + (0.5 if (end - start).total_seconds() / 3600 < resource.calendar_id.hours_per_day else 1)
                    if resource.calendar_id and resource.calendar_id.flexible_hours:
                        duration_hours = duration_days * resource.calendar_id.hours_per_day
                    else:
                        duration_hours = (end - start).total_seconds() / 3600
                    # If the resource is flexible, return the whole period from start_dt to end_dt with a dummy attendance
                    dummy_attendance = self.env['resource.calendar.attendance'].new({
                        'duration_hours': duration_hours,
                        'duration_days': duration_days,
                    })
                    result_per_resource_id[resource.id] = WorkIntervals([(start, end, dummy_attendance)])
                elif resource in per_resource_result:
                    resource_specific_result = [(max(bounds_per_tz[tz][0], tz.localize(val[0])), min(bounds_per_tz[tz][1], tz.localize(val[1])), val[2])
                        for val in per_resource_result[resource]]
                    result_per_resource_id[resource.id] = WorkIntervals(itertools.chain(res, resource_specific_result))
                else:
                    result_per_resource_id[resource.id] = res_intervals
        return result_per_resource_id

    def _handle_flexible_leave_interval(self, dt0, dt1, leave):
        """Hook method to handle flexible leave intervals. Can be overridden in other modules."""
        tz = dt0.tzinfo  # Get the timezone information from dt0
        dt0 = datetime.combine(dt0.date(), time.min).replace(tzinfo=tz)
        dt1 = datetime.combine(dt1.date(), time.max).replace(tzinfo=tz)
        return dt0, dt1

    def _leave_intervals(self, start_dt, end_dt, resource=None, domain=None, tz=None):
        if resource is None:
            resource = self.env['resource.resource']
        return self._leave_intervals_batch(
            start_dt, end_dt, resources=resource, domain=domain, tz=tz
        )[resource.id]

    def _leave_intervals_batch(self, start_dt, end_dt, resources=None, domain=None, tz=None, any_calendar=False):
        """ Return the leave intervals in the given datetime range.
            The returned intervals are expressed in specified tz or in the calendar's timezone.
        """
        assert start_dt.tzinfo and end_dt.tzinfo
        self.ensure_one()

        if not resources:
            resources = self.env['resource.resource']
            resources_list = [resources]
        else:
            resources_list = list(resources) + [self.env['resource.resource']]
        if domain is None:
            domain = [('time_type', '=', 'leave')]
        if not any_calendar:
            domain = domain + [('calendar_id', 'in', [False, self.id])]
        # for the computation, express all datetimes in UTC
        # Public leave don't have a resource_id
        domain = domain + [
            ('resource_id', 'in', [False] + [r.id for r in resources_list]),
            ('date_from', '<=', datetime_to_string(end_dt)),
            ('date_to', '>=', datetime_to_string(start_dt)),
        ]

        # retrieve leave intervals in (start_dt, end_dt)
        result = defaultdict(lambda: [])
        tz_dates = {}
        all_leaves = self.env['resource.calendar.leaves'].search(domain)
        for leave in all_leaves:
            leave_resource = leave.resource_id
            leave_company = leave.company_id
            leave_date_from = leave.date_from
            leave_date_to = leave.date_to
            for resource in resources_list:
                if leave_resource.id not in [False, resource.id] or (not leave_resource and resource and resource.company_id != leave_company):
                    continue
                tz = tz if tz else timezone((resource or self).tz)
                if (tz, start_dt) in tz_dates:
                    start = tz_dates[(tz, start_dt)]
                else:
                    start = start_dt.astimezone(tz)
                    tz_dates[(tz, start_dt)] = start
                if (tz, end_dt) in tz_dates:
                    end = tz_dates[(tz, end_dt)]
                else:
                    end = end_dt.astimezone(tz)
                    tz_dates[(tz, end_dt)] = end
                dt0 = string_to_datetime(leave_date_from).astimezone(tz)
                dt1 = string_to_datetime(leave_date_to).astimezone(tz)
                if leave_resource and leave_resource._is_flexible():
                    dt0, dt1 = self._handle_flexible_leave_interval(dt0, dt1, leave)
                result[resource.id].append((max(start, dt0), min(end, dt1), leave))

        return {r.id: Intervals(result[r.id]) for r in resources_list}

    def _work_intervals_batch(self, start_dt, end_dt, resources=None, domain=None, tz=None, compute_leaves=True):
        """ Return the effective work intervals between the given datetimes. """
        if not resources:
            resources = self.env['resource.resource']
            resources_list = [resources]
        else:
            resources_list = list(resources) + [self.env['resource.resource']]

        attendance_intervals = self._attendance_intervals_batch(start_dt, end_dt, resources, tz=tz or self.env.context.get("employee_timezone"))
        if compute_leaves:
            leave_intervals = self._leave_intervals_batch(start_dt, end_dt, resources, domain, tz=tz)
            return {
                r.id: (attendance_intervals[r.id] - leave_intervals[r.id]) for r in resources_list
            }
        else:
            return {
                r.id: attendance_intervals[r.id] for r in resources_list
            }

    def _unavailable_intervals(self, start_dt, end_dt, resource=None, domain=None, tz=None):
        if resource is None:
            resource = self.env['resource.resource']
        return self._unavailable_intervals_batch(
            start_dt, end_dt, resources=resource, domain=domain, tz=tz
        )[resource.id]

    def _unavailable_intervals_batch(self, start_dt, end_dt, resources=None, domain=None, tz=None):
        """ Return the unavailable intervals between the given datetimes. """
        if not resources:
            resources = self.env['resource.resource']
            resources_list = [resources]
        else:
            resources_list = list(resources)

        resources_work_intervals = self._work_intervals_batch(start_dt, end_dt, resources, domain, tz)
        result = {}
        for resource in resources_list:
            if resource and resource._is_fully_flexible():
                continue
            work_intervals = [(start, stop) for start, stop, meta in resources_work_intervals[resource.id]]
            # start + flatten(intervals) + end
            work_intervals = [start_dt] + list(chain.from_iterable(work_intervals)) + [end_dt]
            # put it back to UTC
            work_intervals = list(map(lambda dt: dt.astimezone(utc), work_intervals))
            # pick groups of two
            work_intervals = list(zip(work_intervals[0::2], work_intervals[1::2]))
            result[resource.id] = work_intervals
        return result

    # --------------------------------------------------
    # Private Methods / Helpers
    # --------------------------------------------------

    def _get_attendance_intervals_days_data(self, attendance_intervals):
        """
        helper function to compute duration of `intervals` that have
        'resource.calendar.attendance' records as payload (3rd element in tuple).
        expressed in days and hours.

        resource.calendar.attendance records have durations associated
        with them so this method merely calculates the proportion that is
        covered by the intervals.
        """
        day_hours = defaultdict(float)
        day_days = defaultdict(float)
        for start, stop, meta in attendance_intervals:
            # If the interval covers only a part of the original attendance, we
            # take durations in days proportionally to what is left of the interval.
            interval_hours = (stop - start).total_seconds() / 3600
            if len(self) == 1 and self.flexible_hours:
                day_hours[start.date()] += meta.duration_hours
                day_days[start.date()] += meta.duration_days
            else:
                day_hours[start.date()] += interval_hours
                day_days[start.date()] += sum(meta.mapped('duration_days')) * interval_hours / sum(meta.mapped('duration_hours'))

        return {
            # Round the number of days to the closest 16th of a day.
            'days': float_round(sum(day_days[day] for day in day_days), precision_rounding=0.001),
            'hours': sum(day_hours.values()),
        }

    def _get_days_data(self, intervals, day_total):
        """
        helper function to compute duration of `intervals`
        expressed in days and hours.
        `day_total` is a dict {date: n_hours} with the number of hours for each day.
        """
        day_hours = defaultdict(float)
        for start, stop, meta in intervals:
            day_hours[start.date()] += (stop - start).total_seconds() / 3600

        # compute number of days the hours span over
        days = float_round(sum(
            day_hours[day] / day_total[day] if day_total[day] else 0
            for day in day_hours
        ), precision_rounding=0.001)
        return {
            'days': days,
            'hours': sum(day_hours.values()),
        }

    def _get_resources_day_total(self, from_datetime, to_datetime, resources=None):
        """
        @return dict with hours of attendance in each day between `from_datetime` and `to_datetime`
        """
        self.ensure_one()
        if not resources:
            resources = self.env['resource.resource']
            resources_list = [resources]
        else:
            resources_list = list(resources) + [self.env['resource.resource']]
        # total hours per day:  retrieve attendances with one extra day margin,
        # in order to compute the total hours on the first and last days
        from_full = from_datetime - timedelta(days=1)
        to_full = to_datetime + timedelta(days=1)
        intervals = self._attendance_intervals_batch(from_full, to_full, resources=resources)

        result = defaultdict(lambda: defaultdict(float))
        for resource in resources_list:
            day_total = result[resource.id]
            for start, stop, meta in intervals[resource.id]:
                day_total[start.date()] += (stop - start).total_seconds() / 3600
        return result

    def _get_closest_work_time(self, dt, match_end=False, resource=None, search_range=None, compute_leaves=True):
        """Return the closest work interval boundary within the search range.
        Consider only starts of intervals unless `match_end` is True. It will then only consider
        ends of intervals.
        :param dt: reference datetime
        :param match_end: wether to search for the begining of an interval or the end.
        :param search_range: time interval considered. Defaults to the entire day of `dt`
        :rtype: datetime | None
        """
        def interval_dt(interval):
            return interval[1 if match_end else 0]

        tz = resource.tz if resource else self.tz
        if resource is None:
            resource = self.env['resource.resource']

        if not dt.tzinfo or search_range and not (search_range[0].tzinfo and search_range[1].tzinfo):
            raise ValueError('Provided datetimes needs to be timezoned')

        dt = dt.astimezone(timezone(tz))

        if not search_range:
            range_start = dt + relativedelta(hour=0, minute=0, second=0)
            range_end = dt + relativedelta(days=1, hour=0, minute=0, second=0)
        else:
            range_start, range_end = search_range

        if not range_start <= dt <= range_end:
            return None
        work_intervals = sorted(
            self._work_intervals_batch(range_start, range_end, resource, compute_leaves=compute_leaves)[resource.id],
            key=lambda i: abs(interval_dt(i) - dt),
        )
        return interval_dt(work_intervals[0]) if work_intervals else None

    def _get_unusual_days(self, start_dt, end_dt, company_id=False):
        if not self:
            return {}
        self.ensure_one()
        if not start_dt.tzinfo:
            start_dt = start_dt.replace(tzinfo=utc)
        if not end_dt.tzinfo:
            end_dt = end_dt.replace(tzinfo=utc)

        domain = []
        if company_id:
            domain = [('company_id', 'in', (company_id.id, False))]
        works = {d[0].date() for d in self._work_intervals_batch(start_dt, end_dt, domain=domain)[False]}
        return {fields.Date.to_string(day.date()): (day.date() not in works) for day in rrule(DAILY, start_dt, until=end_dt)}

    # --------------------------------------------------
    # External API
    # --------------------------------------------------

    def get_work_hours_count(self, start_dt, end_dt, compute_leaves=True, domain=None):
        """
            `compute_leaves` controls whether or not this method is taking into
            account the global leaves.

            `domain` controls the way leaves are recognized.
            None means default value ('time_type', '=', 'leave')

            Counts the number of work hours between two datetimes.
        """
        self.ensure_one()
        # Set timezone in UTC if no timezone is explicitly given
        if not start_dt.tzinfo:
            start_dt = start_dt.replace(tzinfo=utc)
        if not end_dt.tzinfo:
            end_dt = end_dt.replace(tzinfo=utc)

        if compute_leaves:
            intervals = self._work_intervals_batch(start_dt, end_dt, domain=domain)[False]
        else:
            intervals = self._attendance_intervals_batch(start_dt, end_dt)[False]

        return sum(
            (stop - start).total_seconds() / 3600
            for start, stop, meta in intervals
        )

    def get_work_duration_data(self, from_datetime, to_datetime, compute_leaves=True, domain=None):
        """
            Get the working duration (in days and hours) for a given period, only
            based on the current calendar. This method does not use resource to
            compute it.

            `domain` is used in order to recognise the leaves to take,
            None means default value ('time_type', '=', 'leave')

            Returns a dict {'days': n, 'hours': h} containing the
            quantity of working time expressed as days and as hours.
        """
        # naive datetimes are made explicit in UTC
        from_datetime, dummy = make_aware(from_datetime)
        to_datetime, dummy = make_aware(to_datetime)

        # actual hours per day
        if compute_leaves:
            intervals = self._work_intervals_batch(from_datetime, to_datetime, domain=domain)[False]
        else:
            intervals = self._attendance_intervals_batch(from_datetime, to_datetime, domain=domain)[False]

        return self._get_attendance_intervals_days_data(intervals)

    def plan_hours(self, hours, day_dt, compute_leaves=False, domain=None, resource=None):
        """
        `compute_leaves` controls whether or not this method is taking into
        account the global leaves.

        `domain` controls the way leaves are recognized.
        None means default value ('time_type', '=', 'leave')

        Return datetime after having planned hours
        """
        day_dt, revert = make_aware(day_dt)

        if resource is None:
            resource = self.env['resource.resource']

        # which method to use for retrieving intervals
        if compute_leaves:
            get_intervals = partial(self._work_intervals_batch, domain=domain, resources=resource)
            resource_id = resource.id
        else:
            get_intervals = self._attendance_intervals_batch
            resource_id = False

        if hours >= 0:
            delta = timedelta(days=14)
            for n in range(100):
                dt = day_dt + delta * n
                for start, stop, meta in get_intervals(dt, dt + delta)[resource_id]:
                    interval_hours = (stop - start).total_seconds() / 3600
                    if hours <= interval_hours:
                        return revert(start + timedelta(hours=hours))
                    hours -= interval_hours
            return False
        else:
            hours = abs(hours)
            delta = timedelta(days=14)
            for n in range(100):
                dt = day_dt - delta * n
                for start, stop, meta in reversed(get_intervals(dt - delta, dt)[resource_id]):
                    interval_hours = (stop - start).total_seconds() / 3600
                    if hours <= interval_hours:
                        return revert(stop - timedelta(hours=hours))
                    hours -= interval_hours
            return False

    def plan_days(self, days, day_dt, compute_leaves=False, domain=None):
        """
        `compute_leaves` controls whether or not this method is taking into
        account the global leaves.

        `domain` controls the way leaves are recognized.
        None means default value ('time_type', '=', 'leave')

        Returns the datetime of a days scheduling.
        """
        day_dt, revert = make_aware(day_dt)

        # which method to use for retrieving intervals
        if compute_leaves:
            get_intervals = partial(self._work_intervals_batch, domain=domain)
        else:
            get_intervals = self._attendance_intervals_batch

        if days > 0:
            found = set()
            delta = timedelta(days=14)
            for n in range(100):
                dt = day_dt + delta * n
                for start, stop, meta in get_intervals(dt, dt + delta)[False]:
                    found.add(start.date())
                    if len(found) == days:
                        return revert(stop)
            return False

        elif days < 0:
            days = abs(days)
            found = set()
            delta = timedelta(days=14)
            for n in range(100):
                dt = day_dt - delta * n
                for start, stop, meta in reversed(get_intervals(dt - delta, dt)[False]):
                    found.add(start.date())
                    if len(found) == days:
                        return revert(start)
            return False

        else:
            return revert(day_dt)

    def _get_max_number_of_hours(self, start, end):
        self.ensure_one()
        if not self.attendance_ids:
            return 0
        mapped_data = defaultdict(lambda: 0)
        for attendance in self.attendance_ids.filtered(lambda a: a.day_period != 'lunch' and ((not a.date_from or not a.date_to) or (a.date_from <= end.date() and a.date_to >= start.date()))):
            mapped_data[(attendance.week_type, attendance.dayofweek)] += attendance.hour_to - attendance.hour_from
        return max(mapped_data.values())

    def _works_on_date(self, date):
        self.ensure_one()

        working_days = self._get_working_hours()
        dayofweek = str(date.weekday())
        if self.two_weeks_calendar:
            weektype = str(self.env['resource.calendar.attendance'].get_week_type(date))
            return working_days[weektype][dayofweek]
        return working_days[False][dayofweek]

    @ormcache('self.id')
    def _get_working_hours(self):
        self.ensure_one()

        working_days = defaultdict(lambda: defaultdict(lambda: False))
        for attendance in self.attendance_ids:
            working_days[attendance.week_type][attendance.dayofweek] = True
        return working_days

```

## File: models\resource_calendar_attendance.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import math

from odoo import api, fields, models, _


class ResourceCalendarAttendance(models.Model):
    _name = "resource.calendar.attendance"
    _description = "Work Detail"
    _order = 'sequence, week_type, dayofweek, hour_from'

    name = fields.Char(required=True)
    dayofweek = fields.Selection([
        ('0', 'Monday'),
        ('1', 'Tuesday'),
        ('2', 'Wednesday'),
        ('3', 'Thursday'),
        ('4', 'Friday'),
        ('5', 'Saturday'),
        ('6', 'Sunday')
        ], 'Day of Week', required=True, index=True, default='0')
    date_from = fields.Date(string='Starting Date')
    date_to = fields.Date(string='End Date')
    hour_from = fields.Float(string='Work from', required=True, index=True,
        help="Start and End time of working.\n"
             "A specific value of 24:00 is interpreted as 23:59:59.999999.")
    hour_to = fields.Float(string='Work to', required=True)
    # For the hour duration, the compute function is used to compute the value
    # unambiguously, while the duration in days is computed for the default
    # value based on the day_period but can be manually overridden.
    duration_hours = fields.Float(compute='_compute_duration_hours', string='Duration (hours)')
    duration_days = fields.Float(compute='_compute_duration_days', string='Duration (days)', store=True, readonly=False)
    calendar_id = fields.Many2one("resource.calendar", string="Resource's Calendar", required=True, ondelete='cascade')
    day_period = fields.Selection([
        ('morning', 'Morning'),
        ('lunch', 'Break'),
        ('afternoon', 'Afternoon')], required=True, default='morning')
    resource_id = fields.Many2one('resource.resource', 'Resource')
    week_type = fields.Selection([
        ('1', 'Second'),
        ('0', 'First')
        ], 'Week Number', default=False)
    two_weeks_calendar = fields.Boolean("Calendar in 2 weeks mode", related='calendar_id.two_weeks_calendar')
    display_type = fields.Selection([
        ('line_section', "Section")], default=False, help="Technical field for UX purpose.")
    sequence = fields.Integer(default=10,
        help="Gives the sequence of this line when displaying the resource calendar.")

    @api.onchange('hour_from', 'hour_to')
    def _onchange_hours(self):
        # avoid negative or after midnight
        self.hour_from = min(self.hour_from, 23.99)
        self.hour_from = max(self.hour_from, 0.0)
        self.hour_to = min(self.hour_to, 24)
        self.hour_to = max(self.hour_to, 0.0)

        # avoid wrong order
        self.hour_to = max(self.hour_to, self.hour_from)

    @api.model
    def get_week_type(self, date):
        # week_type is defined by
        #  * counting the number of days from January 1 of year 1
        #    (extrapolated to dates prior to the first adoption of the Gregorian calendar)
        #  * converted to week numbers and then the parity of this number is asserted.
        # It ensures that an even week number always follows an odd week number. With classical week number,
        # some years have 53 weeks. Therefore, two consecutive odd week number follow each other (53 --> 1).
        return int(math.floor((date.toordinal() - 1) / 7) % 2)

    @api.depends('hour_from', 'hour_to')
    def _compute_duration_hours(self):
        for attendance in self:
            attendance.duration_hours = (attendance.hour_to - attendance.hour_from) if attendance.day_period != 'lunch' else 0

    @api.depends('day_period', 'duration_hours')
    def _compute_duration_days(self):
        for attendance in self:
            if attendance.day_period == 'lunch':
                attendance.duration_days = 0
            else:
                attendance.duration_days = 0.5 if attendance.duration_hours <= attendance.calendar_id.hours_per_day * 3 / 4 else 1

    @api.depends('week_type')
    def _compute_display_name(self):
        super()._compute_display_name()
        this_week_type = str(self.get_week_type(fields.Date.context_today(self)))
        section_names = {'0': _('First week'), '1': _('Second week')}
        section_info = {True: _('this week'), False: _('other week')}
        for record in self.filtered(lambda l: l.display_type == 'line_section'):
            section_name = f"{section_names[record.week_type]} ({section_info[this_week_type == record.week_type]})"
            record.display_name = section_name

    def _copy_attendance_vals(self):
        self.ensure_one()
        return {
            'name': self.name,
            'dayofweek': self.dayofweek,
            'date_from': self.date_from,
            'date_to': self.date_to,
            'hour_from': self.hour_from,
            'hour_to': self.hour_to,
            'day_period': self.day_period,
            'week_type': self.week_type,
            'display_type': self.display_type,
            'sequence': self.sequence,
        }

```

## File: models\resource_calendar_leaves.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import datetime, time
from dateutil.relativedelta import relativedelta
from pytz import timezone, utc

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


class ResourceCalendarLeaves(models.Model):
    _name = "resource.calendar.leaves"
    _description = "Resource Time Off Detail"
    _order = "date_from"

    def default_get(self, fields_list):
        res = super().default_get(fields_list)
        if 'date_from' in fields_list and 'date_to' in fields_list and not res.get('date_from') and not res.get('date_to'):
            # Then we give the current day and we search the begin and end hours for this day in resource.calendar of the current company
            today = fields.Datetime.now()
            user_tz = timezone(self.env.user.tz or self._context.get('tz') or self.company_id.resource_calendar_id.tz or 'UTC')
            date_from = user_tz.localize(datetime.combine(today, time.min))
            date_to = user_tz.localize(datetime.combine(today, time.max))
            intervals = self.env.company.resource_calendar_id._work_intervals_batch(date_from.replace(tzinfo=utc), date_to.replace(tzinfo=utc))[False]
            if intervals:  # Then we stop and return the dates given in parameter
                list_intervals = [(start, stop) for start, stop, records in intervals]  # Convert intervals in interval list
                date_from = list_intervals[0][0]  # We take the first date in the interval list
                date_to = list_intervals[-1][1]  # We take the last date in the interval list
            res.update(
                date_from=date_from.astimezone(utc).replace(tzinfo=None),
                date_to=date_to.astimezone(utc).replace(tzinfo=None)
            )
        return res

    name = fields.Char('Reason')
    company_id = fields.Many2one(
        'res.company', string="Company", readonly=True, store=True,
        default=lambda self: self.env.company, compute='_compute_company_id')
    calendar_id = fields.Many2one(
        'resource.calendar', "Working Hours",
        compute='_compute_calendar_id', store=True, readonly=False,
        domain="[('company_id', 'in', [company_id, False])]",
        check_company=True, index=True,
    )
    date_from = fields.Datetime('Start Date', required=True)
    date_to = fields.Datetime('End Date', compute="_compute_date_to", readonly=False, required=True, store=True)
    resource_id = fields.Many2one(
        "resource.resource", 'Resource', index=True,
        help="If empty, this is a generic time off for the company. If a resource is set, the time off is only for this resource")
    time_type = fields.Selection([('leave', 'Time Off'), ('other', 'Other')], default='leave',
                                 help="Whether this should be computed as a time off or as work time (eg: formation)")

    @api.depends('resource_id.calendar_id')
    def _compute_calendar_id(self):
        for leave in self.filtered('resource_id'):
            leave.calendar_id = leave.resource_id.calendar_id

    @api.depends('calendar_id')
    def _compute_company_id(self):
        for leave in self:
            leave.company_id = leave.calendar_id.company_id or self.env.company

    @api.depends('date_from')
    def _compute_date_to(self):
        user_tz = timezone(self.env.user.tz or self._context.get('tz') or self.company_id.resource_calendar_id.tz or 'UTC')
        for leave in self:
            if not leave.date_from or (leave.date_to and leave.date_to > leave.date_from):
                continue
            local_date_from = utc.localize(leave.date_from).astimezone(user_tz)
            local_date_to = local_date_from + relativedelta(hour=23, minute=59, second=59)
            leave.date_to = local_date_to.astimezone(utc).replace(tzinfo=None)

    @api.constrains('date_from', 'date_to')
    def check_dates(self):
        if self.filtered(lambda leave: leave.date_from > leave.date_to):
            raise ValidationError(_('The start date of the time off must be earlier than the end date.'))

    def _copy_leave_vals(self):
        self.ensure_one()
        return {
            'name': self.name,
            'date_from': self.date_from,
            'date_to': self.date_to,
            'time_type': self.time_type,
        }

```

## File: models\resource_mixin.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict
from pytz import utc

from odoo import api, fields, models
from .utils import timezone_datetime


class ResourceMixin(models.AbstractModel):
    _name = "resource.mixin"
    _description = 'Resource Mixin'

    resource_id = fields.Many2one(
        'resource.resource', 'Resource',
        auto_join=True, index=True, ondelete='restrict', required=True)
    company_id = fields.Many2one(
        'res.company', 'Company',
        default=lambda self: self.env.company,
        index=True, related='resource_id.company_id', precompute=True, store=True, readonly=False)
    resource_calendar_id = fields.Many2one(
        'resource.calendar', 'Working Hours',
        default=lambda self: self.env.company.resource_calendar_id,
        index=True, related='resource_id.calendar_id', store=True, readonly=False)
    tz = fields.Selection(
        string='Timezone', related='resource_id.tz', readonly=False,
        help="This field is used in order to define in which timezone the resources will work.")

    @api.model_create_multi
    def create(self, vals_list):
        resources_vals_list = []
        calendar_ids = [vals['resource_calendar_id'] for vals in vals_list if vals.get('resource_calendar_id')]
        calendars_tz = {calendar.id: calendar.tz for calendar in self.env['resource.calendar'].browse(calendar_ids)}
        for vals in vals_list:
            if not vals.get('resource_id'):
                resources_vals_list.append(
                    self._prepare_resource_values(
                        vals,
                        vals.pop('tz', False) or calendars_tz.get(vals.get('resource_calendar_id'))
                    )
                )
        if resources_vals_list:
            resources = self.env['resource.resource'].create(resources_vals_list)
            resources_iter = iter(resources.ids)
            for vals in vals_list:
                if not vals.get('resource_id'):
                    vals['resource_id'] = next(resources_iter)
        return super(ResourceMixin, self.with_context(check_idempotence=True)).create(vals_list)

    def _prepare_resource_values(self, vals, tz):
        resource_vals = {'name': vals.get(self._rec_name)}
        if tz:
            resource_vals['tz'] = tz
        company_id = vals.get('company_id', self.env.company.id)
        if company_id:
            resource_vals['company_id'] = company_id
        calendar_id = vals.get('resource_calendar_id')
        if calendar_id:
            resource_vals['calendar_id'] = calendar_id
        return resource_vals

    def copy_data(self, default=None):
        default = dict(default or {})
        vals_list = super().copy_data(default=default)

        resource_default = {}
        if 'company_id' in default:
            resource_default['company_id'] = default['company_id']
        if 'resource_calendar_id' in default:
            resource_default['calendar_id'] = default['resource_calendar_id']
        resources = [record.resource_id for record in self]
        resources_to_copy = self.env['resource.resource'].concat(*resources)
        new_resources = resources_to_copy.copy(resource_default)
        for resource, vals in zip(new_resources, vals_list):
            vals['resource_id'] = resource.id
            vals['company_id'] = resource.company_id.id
            vals['resource_calendar_id'] = resource.calendar_id.id
        return vals_list

    def _get_calendars(self, date_from=None):
        return { resource.id: resource.resource_calendar_id or False for resource in self }

    def _get_work_days_data_batch(self, from_datetime, to_datetime, compute_leaves=True, calendar=None, domain=None):
        """
            By default the resource calendar is used, but it can be
            changed using the `calendar` argument.

            `domain` is used in order to recognise the leaves to take,
            None means default value ('time_type', '=', 'leave')

            Returns a dict {'days': n, 'hours': h} containing the
            quantity of working time expressed as days and as hours.
        """
        resources = self.mapped('resource_id')
        mapped_employees = {e.resource_id.id: e.id for e in self}
        result = {}

        # naive datetimes are made explicit in UTC
        from_datetime = timezone_datetime(from_datetime)
        to_datetime = timezone_datetime(to_datetime)

        if calendar:
            mapped_resources = {calendar: self.resource_id}
        else:
            calendar_by_resource = self._get_calendars(from_datetime)
            mapped_resources = defaultdict(lambda: self.env['resource.resource'])
            for resource in self:
                mapped_resources[calendar_by_resource[resource.id]] |= resource.resource_id

        for calendar, calendar_resources in mapped_resources.items():
            if not calendar:
                for calendar_resource in calendar_resources:
                    result[calendar_resource.id] = {'days': 0, 'hours': 0}
                continue

            # actual hours per day
            if compute_leaves:
                intervals = calendar._work_intervals_batch(from_datetime, to_datetime, calendar_resources, domain)
            else:
                intervals = calendar._attendance_intervals_batch(from_datetime, to_datetime, calendar_resources)

            for calendar_resource in calendar_resources:
                result[calendar_resource.id] = calendar._get_attendance_intervals_days_data(intervals[calendar_resource.id])

        # convert "resource: result" into "employee: result"
        return {mapped_employees[r.id]: result[r.id] for r in resources}

    def _get_leave_days_data_batch(self, from_datetime, to_datetime, calendar=None, domain=None):
        """
            By default the resource calendar is used, but it can be
            changed using the `calendar` argument.

            `domain` is used in order to recognise the leaves to take,
            None means default value ('time_type', '=', 'leave')

            Returns a dict {'days': n, 'hours': h} containing the number of leaves
            expressed as days and as hours.
        """
        resources = self.mapped('resource_id')
        mapped_employees = {e.resource_id.id: e.id for e in self}
        result = {}

        # naive datetimes are made explicit in UTC
        from_datetime = timezone_datetime(from_datetime)
        to_datetime = timezone_datetime(to_datetime)

        mapped_resources = defaultdict(lambda: self.env['resource.resource'])
        for record in self:
            mapped_resources[calendar or record.resource_calendar_id] |= record.resource_id

        for calendar, calendar_resources in mapped_resources.items():
            # handle fully flexible resources by returning the length of the whole interval
            # since we do not take into account leaves for fully flexible resources
            if not calendar:
                days = (to_datetime - from_datetime).days
                hours = (to_datetime - from_datetime).total_seconds() / 3600
                for calendar_resource in calendar_resources:
                    result[calendar_resource.id] = {'days': days, 'hours': hours}
                continue

            # compute actual hours per day
            attendances = calendar._attendance_intervals_batch(from_datetime, to_datetime, calendar_resources)
            leaves = calendar._leave_intervals_batch(from_datetime, to_datetime, calendar_resources, domain)

            for calendar_resource in calendar_resources:
                result[calendar_resource.id] = calendar._get_attendance_intervals_days_data(
                    attendances[calendar_resource.id] & leaves[calendar_resource.id]
                )

        # convert "resource: result" into "employee: result"
        return {mapped_employees[r.id]: result[r.id] for r in resources}

    def _adjust_to_calendar(self, start, end):
        resource_results = self.resource_id._adjust_to_calendar(start, end)
        # change dict keys from resources to associated records.
        return {
            record: resource_results[record.resource_id]
            for record in self
        }

    def _list_work_time_per_day(self, from_datetime, to_datetime, calendar=None, domain=None):
        """
            By default the resource calendar is used, but it can be
            changed using the `calendar` argument.

            `domain` is used in order to recognise the leaves to take,
            None means default value ('time_type', '=', 'leave')

            Returns a list of tuples (day, hours) for each day
            containing at least an attendance.
        """
        result = {}
        records_by_calendar = defaultdict(lambda: self.env[self._name])
        for record in self:
            records_by_calendar[calendar or record.resource_calendar_id or record.company_id.resource_calendar_id] += record

        # naive datetimes are made explicit in UTC
        if not from_datetime.tzinfo:
            from_datetime = from_datetime.replace(tzinfo=utc)
        if not to_datetime.tzinfo:
            to_datetime = to_datetime.replace(tzinfo=utc)
        compute_leaves = self.env.context.get('compute_leaves', True)

        for calendar, records in records_by_calendar.items():
            resources = self.resource_id
            all_intervals = calendar._work_intervals_batch(from_datetime, to_datetime, resources, domain, compute_leaves=compute_leaves)
            for record in records:
                intervals = all_intervals[record.resource_id.id]
                record_result = defaultdict(float)
                for start, stop, meta in intervals:
                    if calendar.flexible_hours:
                        record_result[start.date()] = meta.duration_hours
                    else:
                        record_result[start.date()] += (stop - start).total_seconds() / 3600
                result[record.id] = sorted(record_result.items())
        return result

    def list_leaves(self, from_datetime, to_datetime, calendar=None, domain=None):
        """
            By default the resource calendar is used, but it can be
            changed using the `calendar` argument.

            `domain` is used in order to recognise the leaves to take,
            None means default value ('time_type', '=', 'leave')

            Returns a list of tuples (day, hours, resource.calendar.leaves)
            for each leave in the calendar.
        """
        resource = self.resource_id
        calendar = calendar or self.resource_calendar_id

        # naive datetimes are made explicit in UTC
        if not from_datetime.tzinfo:
            from_datetime = from_datetime.replace(tzinfo=utc)
        if not to_datetime.tzinfo:
            to_datetime = to_datetime.replace(tzinfo=utc)

        attendances = calendar._attendance_intervals_batch(from_datetime, to_datetime, resource)[resource.id]
        leaves = calendar._leave_intervals_batch(from_datetime, to_datetime, resource, domain)[resource.id]
        result = []
        for start, stop, leave in (leaves & attendances):
            hours = (stop - start).total_seconds() / 3600
            result.append((start.date(), hours, leave))
        return result

```

## File: models\resource_resource.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict
from dateutil.relativedelta import relativedelta
from pytz import timezone

from odoo import api, fields, models
from odoo.addons.base.models.res_partner import _tz_get

from .utils import timezone_datetime, make_aware, Intervals


class ResourceResource(models.Model):
    _name = "resource.resource"
    _description = "Resources"
    _order = "name"

    @api.model
    def default_get(self, fields):
        res = super().default_get(fields)
        if not res.get('calendar_id') and res.get('company_id'):
            company = self.env['res.company'].browse(res['company_id'])
            res['calendar_id'] = company.resource_calendar_id.id
        return res

    name = fields.Char(required=True)
    active = fields.Boolean(
        'Active', default=True,
        help="If the active field is set to False, it will allow you to hide the resource record without removing it.")
    company_id = fields.Many2one('res.company', string='Company', default=lambda self: self.env.company)
    resource_type = fields.Selection([
        ('user', 'Human'),
        ('material', 'Material')], string='Type',
        default='user', required=True)
    user_id = fields.Many2one('res.users', string='User', help='Related user name for the resource to manage its access.')
    avatar_128 = fields.Image(compute='_compute_avatar_128')
    share = fields.Boolean(related='user_id.share')
    email = fields.Char(related='user_id.email')
    phone = fields.Char(related='user_id.phone')

    time_efficiency = fields.Float(
        'Efficiency Factor', default=100, required=True,
        help="This field is used to calculate the expected duration of a work order at this work center. For example, if a work order takes one hour and the efficiency factor is 100%, then the expected duration will be one hour. If the efficiency factor is 200%, however the expected duration will be 30 minutes.")
    calendar_id = fields.Many2one(
        "resource.calendar", string='Working Time',
        default=lambda self: self.env.company.resource_calendar_id,
        domain="[('company_id', '=', company_id)]",
        help="Define the working schedule of the resource. If not set, the resource will have fully flexible working hours.")
    tz = fields.Selection(
        _tz_get, string='Timezone', required=True,
        default=lambda self: self._context.get('tz') or self.env.user.tz or 'UTC')

    _sql_constraints = [
        ('check_time_efficiency', 'CHECK(time_efficiency>0)', 'Time efficiency must be strictly positive'),
    ]

    @api.depends('user_id')
    def _compute_avatar_128(self):
        for resource in self:
            resource.avatar_128 = resource.user_id.avatar_128

    @api.model_create_multi
    def create(self, vals_list):
        for values in vals_list:
            if values.get('company_id') and not 'calendar_id' in values:
                values['calendar_id'] = self.env['res.company'].browse(values['company_id']).resource_calendar_id.id
            if not values.get('tz'):
                # retrieve timezone on user or calendar
                tz = (self.env['res.users'].browse(values.get('user_id')).tz or
                      self.env['resource.calendar'].browse(values.get('calendar_id')).tz)
                if tz:
                    values['tz'] = tz
        return super().create(vals_list)

    def copy_data(self, default=None):
        vals_list = super().copy_data(default=default)
        return [dict(vals, name=self.env._("%s (copy)", resource.name)) for resource, vals in zip(self, vals_list)]

    def write(self, values):
        if self.env.context.get('check_idempotence') and len(self) == 1:
            values = {
                fname: value
                for fname, value in values.items()
                if self._fields[fname].convert_to_write(self[fname], self) != value
            }
        if not values:
            return True
        return super().write(values)

    @api.onchange('company_id')
    def _onchange_company_id(self):
        if self.company_id:
            self.calendar_id = self.company_id.resource_calendar_id.id

    @api.onchange('user_id')
    def _onchange_user_id(self):
        if self.user_id:
            self.tz = self.user_id.tz

    def _get_work_interval(self, start, end):
        # Deprecated method. Use `_adjust_to_calendar` instead
        return self._adjust_to_calendar(start, end)

    def _adjust_to_calendar(self, start, end, compute_leaves=True):
        """Adjust the given start and end datetimes to the closest effective hours encoded
        in the resource calendar. Only attendances in the same day as `start` and `end` are
        considered (respectively). If no attendance is found during that day, the closest hour
        is None.
        e.g. simplified example:
             given two attendances: 8am-1pm and 2pm-5pm, given start=9am and end=6pm
             resource._adjust_to_calendar(start, end)
             >>> {resource: (8am, 5pm)}
        :return: Closest matching start and end of working periods for each resource
        :rtype: dict(resource, tuple(datetime | None, datetime | None))
        """
        start, revert_start_tz = make_aware(start)
        end, revert_end_tz = make_aware(end)
        result = {}
        for resource in self:
            resource_tz = timezone(resource.tz)
            start, end = start.astimezone(resource_tz), end.astimezone(resource_tz)
            search_range = [
                start + relativedelta(hour=0, minute=0, second=0),
                end + relativedelta(days=1, hour=0, minute=0, second=0),
            ]
            calendar = resource.calendar_id or resource.company_id.resource_calendar_id or self.env.company.resource_calendar_id
            calendar_start = calendar._get_closest_work_time(start, resource=resource, search_range=search_range,
                                                                         compute_leaves=compute_leaves)
            search_range[0] = start
            calendar_end = calendar._get_closest_work_time(max(start, end), match_end=True,
                                                                       resource=resource, search_range=search_range,
                                                                       compute_leaves=compute_leaves)
            result[resource] = (
                calendar_start and revert_start_tz(calendar_start),
                calendar_end and revert_end_tz(calendar_end),
            )
        return result

    def _get_unavailable_intervals(self, start, end):
        """ Compute the intervals during which employee is unavailable with hour granularity between start and end
            Note: this method is used in enterprise (forecast and planning)

        """
        start_datetime = timezone_datetime(start)
        end_datetime = timezone_datetime(end)
        resource_mapping = {}
        calendar_mapping = defaultdict(lambda: self.env['resource.resource'])
        for resource in self:
            calendar_mapping[resource.calendar_id or resource.company_id.resource_calendar_id] |= resource

        for calendar, resources in calendar_mapping.items():
            if not calendar:
                continue
            resources_unavailable_intervals = calendar._unavailable_intervals_batch(start_datetime, end_datetime, resources, tz=timezone(calendar.tz))
            resource_mapping.update(resources_unavailable_intervals)
        return resource_mapping

    def _get_calendars_validity_within_period(self, start, end, default_company=None):
        """ Gets a dict of dict with resource's id as first key and resource's calendar as secondary key
            The value is the validity interval of the calendar for the given resource.

            Here the validity interval for each calendar is the whole interval but it's meant to be overriden in further modules
            handling resource's employee contracts.
        """
        assert start.tzinfo and end.tzinfo
        resource_calendars_within_period = defaultdict(lambda: defaultdict(Intervals))  # keys are [resource id:integer][calendar:self.env['resource.calendar']]
        default_calendar = default_company and default_company.resource_calendar_id or self.env.company.resource_calendar_id
        if not self:
            # if no resource, add the company resource calendar.
            resource_calendars_within_period[False][default_calendar] = Intervals([(start, end, self.env['resource.calendar.attendance'])])
        for resource in self:
            calendar = resource.calendar_id or resource.company_id.resource_calendar_id or default_calendar
            resource_calendars_within_period[resource.id][calendar] = Intervals([(start, end, self.env['resource.calendar.attendance'])])
        return resource_calendars_within_period

    def _get_valid_work_intervals(self, start, end, calendars=None, compute_leaves=True):
        """ Gets the valid work intervals of the resource following their calendars between ``start`` and ``end``

            This methods handle the eventuality of a resource having multiple resource calendars, see _get_calendars_validity_within_period method
            for further explanation.

            For flexible calendars and fully flexible resources: -> return the whole interval
        """
        assert start.tzinfo and end.tzinfo
        resource_calendar_validity_intervals = {}
        calendar_resources = defaultdict(lambda: self.env['resource.resource'])
        resource_work_intervals = defaultdict(Intervals)
        calendar_work_intervals = dict()

        resource_calendar_validity_intervals = self.sudo()._get_calendars_validity_within_period(start, end)
        for resource in self:
            # For each resource, retrieve its calendar and their validity intervals
            for calendar in resource_calendar_validity_intervals[resource.id]:
                calendar_resources[calendar] |= resource
        for calendar in (calendars or []):
            calendar_resources[calendar] |= self.env['resource.resource']
        for calendar, resources in calendar_resources.items():
            # for fully flexible resource, return the whole interval
            if not calendar:
                for resource in resources:
                    resource_work_intervals[resource.id] |= Intervals([(start, end, self.env['resource.calendar.attendance'])])
                continue
            # For each calendar used by the resources, retrieve the work intervals for every resources using it
            work_intervals_batch = calendar._work_intervals_batch(start, end, resources=resources, compute_leaves=compute_leaves)
            for resource in resources:
                # Make the conjunction between work intervals and calendar validity
                resource_work_intervals[resource.id] |= work_intervals_batch[resource.id] & resource_calendar_validity_intervals[resource.id][calendar]
            calendar_work_intervals[calendar.id] = work_intervals_batch[False]

        return resource_work_intervals, calendar_work_intervals

    def _is_fully_flexible(self):
        """ employee has a fully flexible schedule has no working calendar set """
        self.ensure_one()
        return not self.calendar_id

    def _is_flexible(self):
        """ An employee is considered flexible if the field flexible_hours is True on the calendar
            or the employee is not assigned any calendar, in which case is considered as Fully flexible.
        """
        self.ensure_one()
        return self._is_fully_flexible() or (self.calendar_id and self.calendar_id.flexible_hours)

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _


class ResCompany(models.Model):
    _inherit = 'res.company'

    resource_calendar_ids = fields.One2many(
        'resource.calendar', 'company_id', 'Working Hours')
    resource_calendar_id = fields.Many2one(
        'resource.calendar', 'Default Working Hours', ondelete='restrict')

    @api.model
    def _init_data_resource_calendar(self):
        self.search([('resource_calendar_id', '=', False)])._create_resource_calendar()

    def _create_resource_calendar(self):
        vals_list = [
            company._prepare_resource_calendar_values()
            for company in self
        ]
        resource_calendars = self.env['resource.calendar'].create(vals_list)
        for company, calendar in zip(self, resource_calendars):
            company.resource_calendar_id = calendar

    def _prepare_resource_calendar_values(self):
        self.ensure_one()
        return {
            'name': _('Standard 40 hours/week'),
            'company_id': self.id,
        }

    @api.model_create_multi
    def create(self, vals_list):
        companies = super().create(vals_list)
        companies_without_calendar = companies.filtered(lambda c: not c.resource_calendar_id)
        if companies_without_calendar:
            companies_without_calendar.sudo()._create_resource_calendar()
        # calendar created from form view: no company_id set because record was still not created
        for company in companies:
            if not company.resource_calendar_id.company_id:
                company.resource_calendar_id.company_id = company.id
        return companies

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResUsers(models.Model):
    _inherit = 'res.users'

    resource_ids = fields.One2many(
        'resource.resource', 'user_id', 'Resources')
    resource_calendar_id = fields.Many2one(
        'resource.calendar', 'Default Working Hours',
        related='resource_ids.calendar_id', readonly=False)

    def write(self, vals):
        rslt = super().write(vals)

        # If the timezone of the admin user gets set on their first login, also update the timezone of the default working calendar
        if (vals.get('tz') and len(self) == 1 and not self.env.user.login_date
            and self.env.user == self.env.ref('base.user_admin', False) and self == self.env.user):
            if self.resource_calendar_id:
                self.resource_calendar_id.tz = vals['tz']
            else:
                self.env.ref('resource.resource_calendar_std', False).tz = vals['tz']

        return rslt

```

## File: models\utils.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import math
from datetime import time
from itertools import chain
from pytz import utc

from odoo import fields
from odoo.osv.expression import normalize_domain, is_leaf, NOT_OPERATOR
from odoo.tools.float_utils import float_round

# Default hour per day value. The one should
# only be used when the one from the calendar
# is not available.
HOURS_PER_DAY = 8
# This will generate 16th of days
ROUNDING_FACTOR = 16


def make_aware(dt):
    """ Return ``dt`` with an explicit timezone, together with a function to
        convert a datetime to the same (naive or aware) timezone as ``dt``.
    """
    if dt.tzinfo:
        return dt, lambda val: val.astimezone(dt.tzinfo)
    return dt.replace(tzinfo=utc), lambda val: val.astimezone(utc).replace(tzinfo=None)


def string_to_datetime(value):
    """ Convert the given string value to a datetime in UTC. """
    return utc.localize(fields.Datetime.from_string(value))


def datetime_to_string(dt):
    """ Convert the given datetime (converted in UTC) to a string value. """
    return fields.Datetime.to_string(dt.astimezone(utc))


def float_to_time(hours):
    """ Convert a number of hours into a time object. """
    if hours == 24.0:
        return time.max
    fractional, integral = math.modf(hours)
    return time(int(integral), int(float_round(60 * fractional, precision_digits=0)), 0)


def _boundaries(intervals, opening, closing):
    """ Iterate on the boundaries of intervals. """
    for start, stop, recs in intervals:
        if start < stop:
            yield (start, opening, recs)
            yield (stop, closing, recs)

def filter_domain_leaf(domain, field_check, field_name_mapping=None):
    """
    filter_domain_lead only keep the leaves of a domain that verify a given check. Logical operators that involves
    a leaf that is undetermined (because it does not pass the check) are ignored.

    each operator is a logic gate:
    - '&' and '|' take two entries and can be ignored if one of them (or the two of them) is undetermined
    -'!' takes one entry and can be ignored if this entry is undetermined

    params:
        - domain: the domain that needs to be filtered
        - field_check: the function that the field name used in the leaf needs to verify to keep the leaf
        - field_name_mapping: dictionary of the form {'field_name': 'new_field_name', ...}. Occurences of 'field_name'
          in the first element of domain leaves will be replaced by 'new_field_name'. This is usefull when adapting a
          domain from one model to another when some field names do not match the names of the corresponding fields in
          the new model.
    returns: The filtered version of the domain
    """
    domain = normalize_domain(domain)
    field_name_mapping = field_name_mapping or {}

    stack = [] # stack of elements (leaf or operator) to conserve (reversing it gives a domain)
    ignored_elems = [] # history of ignored elements in the domain (not added to the stack)
    # if the top of the stack ignored_elems is:
    # - True: indicates that the last browsed elem has been ignored
    # - False: indicates that the last browsed elem has been added to the stack
    # When an operator is applied to some elements, they are removed from the ignored_elems stack
    # (and replaced by the ignored_elems flag of the operator)
    while domain:
        next_elem = domain.pop() # Browsing the domain backward simplifies the filtering
        if is_leaf(next_elem):
            field_name, op, value = next_elem
            if field_check(field_name):
                field_name = field_name_mapping.get(field_name, field_name)
                stack.append((field_name, op, value))
                ignored_elems.append(False)
            else:
                ignored_elems.append(True)
        elif next_elem == NOT_OPERATOR:
            ignore_operation = ignored_elems.pop()
            if not ignore_operation:
                stack.append(NOT_OPERATOR)
                ignored_elems.append(False)
            else:
                ignored_elems.append(True)
        else: # OR/AND operation
            ignore_operand1 = ignored_elems.pop()
            ignore_operand2 = ignored_elems.pop()
            if not ignore_operand1 and not ignore_operand2:
                stack.append(next_elem)
                ignored_elems.append(False)
            elif ignore_operand1 and ignore_operand2:
                ignored_elems.append(True)
            else:
                ignored_elems.append(False) # the AND/OR operation is replaced by one of its operand which cannot be ignored
    return list(reversed(stack))

class Intervals(object):
    """ Collection of ordered disjoint intervals with some associated records.
        Each interval is a triple ``(start, stop, records)``, where ``records``
        is a recordset.
    """
    def __init__(self, intervals=()):
        self._items = []
        if intervals:
            # normalize the representation of intervals
            append = self._items.append
            starts = []
            recses = []
            for value, flag, recs in sorted(_boundaries(intervals, 'start', 'stop')):
                if flag == 'start':
                    starts.append(value)
                    recses.append(recs)
                else:
                    start = starts.pop()
                    if not starts:
                        append((start, value, recses[0].union(*recses)))
                        recses.clear()

    def __bool__(self):
        return bool(self._items)

    def __len__(self):
        return len(self._items)

    def __iter__(self):
        return iter(self._items)

    def __reversed__(self):
        return reversed(self._items)

    def __or__(self, other):
        """ Return the union of two sets of intervals. """
        return Intervals(chain(self._items, other._items))

    def __and__(self, other):
        """ Return the intersection of two sets of intervals. """
        return self._merge(other, False)

    def __sub__(self, other):
        """ Return the difference of two sets of intervals. """
        return self._merge(other, True)

    def _merge(self, other, difference):
        """ Return the difference or intersection of two sets of intervals. """
        result = Intervals()
        append = result._items.append

        # using 'self' and 'other' below forces normalization
        bounds1 = _boundaries(self, 'start', 'stop')
        bounds2 = _boundaries(other, 'switch', 'switch')

        start = None                    # set by start/stop
        recs1 = None                    # set by start
        enabled = difference            # changed by switch
        for value, flag, recs in sorted(chain(bounds1, bounds2)):
            if flag == 'start':
                start = value
                recs1 = recs
            elif flag == 'stop':
                if enabled and start < value:
                    append((start, value, recs1))
                start = None
            else:
                if not enabled and start is not None:
                    start = value
                if enabled and start is not None and start < value:
                    append((start, value, recs1))
                enabled = not enabled

        return result

def sum_intervals(intervals):
    """ Sum the intervals duration (unit : hour)"""
    return sum(
        (stop - start).total_seconds() / 3600
        for start, stop, meta in intervals
    )

def timezone_datetime(time):
    if not time.tzinfo:
        time = time.replace(tzinfo=utc)
    return time

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_company
from . import res_users
from . import resource_calendar
from . import resource_calendar_attendance
from . import resource_calendar_leaves
from . import resource_mixin
from . import resource_resource
from . import utils

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_resource_calendar_user,resource.calendar.user,model_resource_calendar,base.group_user,1,0,0,0
access_resource_calendar_system,resource.calendar.system,model_resource_calendar,base.group_system,1,1,1,1
access_resource_calendar_attendance_user,resource.calendar.attendance.user,model_resource_calendar_attendance,base.group_user,1,0,0,0
access_resource_calendar_attendance_system,resource.calendar.attendance.system,model_resource_calendar_attendance,base.group_system,1,1,1,1
access_resource_resource,resource.resource,model_resource_resource,base.group_system,1,0,0,0
access_resource_resource_all,resource.resource all,model_resource_resource,base.group_user,1,0,0,0
access_resource_calendar_leaves_user,resource.calendar.leaves,model_resource_calendar_leaves,base.group_user,1,1,1,1
access_resource_calendar_leaves,resource.calendar.leaves,model_resource_calendar_leaves,base.group_system,1,1,1,1

```

## File: security\resource_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record model="ir.rule" id="resource_calendar_leaves_rule_group_user_create">
        <field name="name">resource.calendar.leaves: employee reads own or global</field>
        <field name="model_id" ref="model_resource_calendar_leaves"/>
        <field name="groups" eval="[(4, ref('base.group_user'))]"/>
        <field name="domain_force">['|', ('resource_id', '=', False), ('resource_id.user_id', 'in', [False, user.id])]</field>
		<field name="perm_write" eval="False"/>
		<field name="perm_create" eval="False"/>
		<field name="perm_unlink" eval="False"/>
    </record>

    <record model="ir.rule" id="resource_calendar_leaves_rule_group_user_modify">
        <field name="name">resource.calendar.leaves: employee modifies own</field>
        <field name="model_id" ref="model_resource_calendar_leaves"/>
        <field name="groups" eval="[(4, ref('base.group_user'))]"/>
        <field name="domain_force">[('resource_id', '!=', False), ('resource_id.user_id', 'in', [False, user.id])]</field>
		<field name="perm_read" eval="False"/>
    </record>

    <record model="ir.rule" id="resource_calendar_leaves_rule_group_admin_modify">
        <field name="name">resource.calendar.leaves: admin modifies global</field>
        <field name="model_id" ref="model_resource_calendar_leaves"/>
        <field name="groups" eval="[(4, ref('base.group_erp_manager'))]"/>
        <field name="domain_force">[('resource_id', '=', False)]</field>
        <field name="perm_read" eval="False"/>
    </record>

    <record id="resource_resource_multi_company" model="ir.rule">
        <field name="name">resource.resource multi-company</field>
        <field name="model_id" ref="model_resource_resource"/>
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
    </record>

    <record model="ir.rule" id="resource_calendar_leaves_rule_multi_company">
        <field name="name">resource.calendar.leaves: multi-company rule</field>
        <field name="model_id" ref="model_resource_calendar_leaves"/>
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
    </record>
</odoo>

```

## File: static\src\section_list_renderer.js

```javascript
/** @odoo-module */

import { ListRenderer } from "@web/views/list/list_renderer";
import { useEffect } from "@odoo/owl";

export class SectionListRenderer extends ListRenderer {
    setup() {
        super.setup();

        this.displayType = "line_section";
        this.titleField = "title";

        useEffect(
            (table) => {
                if (table) {
                    table.classList.add("o_section_list_view");
                }
            },
            () => [this.tableRef.el]
        );
    }

    getColumns(record) {
        const columns = super.getColumns(record);
        if (this.isSection(record)) {
            return this.getSectionColumns(columns);
        }
        return columns;
    }

    getRowClass(record) {
        const classNames = super.getRowClass(record).split(" ");
        if (this.isSection(record)) {
            classNames.push(`o_is_${this.displayType}`, `fw-bold`);
        }
        return classNames.join(" ");
    }

    getSectionColumns(columns) {
        const sectionColumns = columns.filter((col) => col.widget === "handle");
        let colspan = columns.length - sectionColumns.length;
        if (this.activeActions.onDelete) {
            colspan++;
        }
        const titleCol = columns.find(
            (col) => col.type === "field" && col.name === this.titleField
        );
        sectionColumns.push({ ...titleCol, colspan });
        return sectionColumns;
    }

    isSection(record) {
        return record.data.display_type === this.displayType;
    }
}
SectionListRenderer.recordRowTemplate = "resource.SectionListRenderer.RecordRow";

```

## File: static\src\section_list_renderer.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="resource.SectionListRenderer.RecordRow" t-inherit="web.ListRenderer.RecordRow">
        <xpath expr="//t[@t-if='displayOptionalFields or hasX2ManyAction']" position="attributes">
            <attribute name="t-if">(displayOptionalFields or hasX2ManyAction) and !isSection(record)</attribute>
        </xpath>
    </t>

</templates>

```

## File: static\src\section_one2many_field.js

```javascript
/** @odoo-module */

import { SectionListRenderer } from "./section_list_renderer";
import { registry } from "@web/core/registry";
import { X2ManyField, x2ManyField } from "@web/views/fields/x2many/x2many_field";

class SectionOneToManyField extends X2ManyField {
    static components = {
        ...X2ManyField.components,
        ListRenderer: SectionListRenderer,
    };
    static defaultProps = {
        ...X2ManyField.defaultProps,
        editable: "bottom",
    };
}

registry.category("fields").add("section_one2many", {
    ...x2ManyField,
    component: SectionOneToManyField,
    additionalClasses: [...x2ManyField.additionalClasses || [], "o_field_one2many"],
});

```

## File: static\src\views\form_with_html_expander\form_controller_with_html_expander.js

```javascript
import { useState } from "@odoo/owl";
import { FormController } from "@web/views/form/form_controller";

export class FormControllerWithHTMLExpander extends FormController {
    static template = "resource.FormViewWithHtmlExpander";

    setup() {
        super.setup();
        this.htmlExpanderState = useState({ reload: true });
        const oldOnNotebookPageChange = this.onNotebookPageChange;
        this.onNotebookPageChange = (notebookId, page) => {
            oldOnNotebookPageChange(notebookId, page);
            if (page && !this.htmlExpanderState.reload) {
                this.htmlExpanderState.reload = true;
            }
        };
    }

    get modelParams() {
        const modelParams = super.modelParams;
        const onRootLoaded = modelParams.hooks.onRootLoaded;
        modelParams.hooks.onRootLoaded = async () => {
            if (onRootLoaded) {
                onRootLoaded();
            }
            this.htmlExpanderState.reload = true;
        };
        return modelParams;
    }

    notifyHTMLFieldExpanded() {
        this.htmlExpanderState.reload = false;
    }

    async onRecordSaved(record, changes) {
        super.onRecordSaved(record, changes);
        this.htmlExpanderState.reload = true;
    }
}

```

## File: static\src\views\form_with_html_expander\form_controller_with_html_expander.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="resource.FormViewWithHtmlExpander" t-inherit="web.FormView">
        <xpath expr="//Layout/t[@t-component='props.Renderer']" position="attributes">
            <attribute name="reloadHtmlFieldHeight">htmlExpanderState.reload</attribute>
            <attribute name="notifyHtmlExpander.bind">notifyHTMLFieldExpanded</attribute>
        </xpath>
    </t>

</templates>

```

## File: static\src\views\form_with_html_expander\form_renderer_with_html_expander.js

```javascript
import { useService } from "@web/core/utils/hooks";
import { FormRenderer } from "@web/views/form/form_renderer";
import { useRef, useEffect } from "@odoo/owl";

export class FormRendererWithHtmlExpander extends FormRenderer {
    static props = {
        ...FormRenderer.props,
        reloadHtmlFieldHeight: { type: Boolean, optional: true },
        notifyHtmlExpander: { type: Function, optional: true },
    };
    static defaultProps = {
        ...FormRenderer.defaultProps,
        reloadHtmlFieldHeight: true,
        notifyHtmlExpander: () => {},
    };

    setup() {
        super.setup();
        if (!this.uiService) {
            // Should be defined in FormRenderer
            this.uiService = useService("ui");
        }
        const ref = useRef("compiled_view_root");
        useEffect(
            (el, size) => {
                if (el && this._canExpandHTMLField(size)) {
                    const descriptionField = el.querySelector(this.htmlFieldQuerySelector);
                    if (descriptionField) {
                        const containerEL = descriptionField.closest(
                            this.getHTMLFieldContainerQuerySelector
                        );
                        const editor = descriptionField.querySelector(".note-editable");
                        const elementToResize = editor || descriptionField;
                        const { top, bottom } = elementToResize.getBoundingClientRect();
                        const { bottom: containerBottom } = containerEL.getBoundingClientRect();
                        const { paddingTop, paddingBottom } = window.getComputedStyle(containerEL);
                        const nonEditableHeight =
                            containerBottom -
                            bottom +
                            parseInt(paddingTop) +
                            parseInt(paddingBottom);
                        const minHeight =
                            document.documentElement.clientHeight - top - nonEditableHeight;
                        elementToResize.style.minHeight = `${minHeight}px`;
                    }
                }
                this.props.notifyHtmlExpander();
            },
            () => [ref.el, this.uiService.size, this.props.reloadHtmlFieldHeight]
        );
    }

    get htmlFieldQuerySelector() {
        return ".o_field_html[name=description]";
    }

    get getHTMLFieldContainerQuerySelector() {
        return ".o_form_sheet";
    }

    _canExpandHTMLField(size) {
        return size === 6;
    }
}

```

## File: static\src\views\form_with_html_expander\form_view_with_html_expander.js

```javascript
import { registry } from "@web/core/registry";
import { formView } from "@web/views/form/form_view";
import { FormRendererWithHtmlExpander } from "./form_renderer_with_html_expander";
import { FormControllerWithHTMLExpander } from "./form_controller_with_html_expander";

export const formViewWithHtmlExpander = {
    ...formView,
    Controller: FormControllerWithHTMLExpander,
    Renderer: FormRendererWithHtmlExpander,
};

registry.category("views").add("form_description_expander", formViewWithHtmlExpander);

```

## File: views\menuitems.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem id="menu_resource_config" name="Resource"
        parent="base.menu_custom"
        sequence="30"/>
    <menuitem id="menu_resource_calendar"
        parent="menu_resource_config"
        action="action_resource_calendar_form"
        sequence="1"/>
    <menuitem id="menu_view_resource_calendar_leaves_search"
        parent="menu_resource_config"
        action="action_resource_calendar_leave_tree"
        sequence="2"/>
    <menuitem id="menu_resource_resource"
        parent="menu_resource_config"
        action="action_resource_resource_tree"
        sequence="3"/>
</odoo>

```

## File: views\resource_calendar_attendance_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_resource_calendar_attendance_tree" model="ir.ui.view">
        <field name="name">resource.calendar.attendance.list</field>
        <field name="model">resource.calendar.attendance</field>
        <field name="arch" type="xml">
            <list string="Working Time" editable="bottom" default_order="sequence, week_type, dayofweek, hour_from">
                <field name="sequence" widget="handle"/>
                <field name="display_type" column_invisible="True"/>
                <field name="display_name" width="1" string=" " invisible="display_type != 'line_section'"/>
                <field name="name" invisible="display_type == 'line_section'"/>
                <field name="dayofweek"/>
                <field name="day_period"/>
                <field name="hour_from" widget="float_time"/>
                <field name="hour_to" widget="float_time"/>
                <field name="duration_days" optional="show"/>
                <field name="date_from" optional="hide"/>
                <field name="date_to" optional="hide"/>
                <field name="week_type" readonly="1" force_save="1" optional="hide"/>
            </list>
        </field>
    </record>

    <record id="view_resource_calendar_attendance_form" model="ir.ui.view">
        <field name="name">resource.calendar.attendance.form</field>
        <field name="model">resource.calendar.attendance</field>
        <field name="arch" type="xml">
            <form string="Working Time">
                <sheet>
                <group>
                    <field name="name"/>
                    <field name="date_from"/>
                    <field name="date_to"/>
                    <field name="dayofweek"/>
                    <label for="hour_from" string="Hours"/>
                    <div class="o_row">
                        <field name="hour_from" widget="float_time"/> -
                        <field name="hour_to" widget="float_time"/>
                    </div>
                    <field name="day_period"/>
                    <field name="duration_days"/>
                </group>
                </sheet>
            </form>
        </field>
    </record>
</odoo>

```

## File: views\resource_calendar_leaves_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_resource_calendar_leaves_search" model="ir.ui.view">
        <field name="name">resource.calendar.leaves.search</field>
        <field name="model">resource.calendar.leaves</field>
        <field name="arch" type="xml">
            <search string="Search Working Period Time Off">
                <field name="name"/>
                <field name="resource_id"/>
                <field name="company_id" groups="base.group_multi_company"/>
                <field name="calendar_id"/>
                <filter name="filter_date" date="date_from" default_period="year" string="Period"/>
                <group expand="0" string="Group By">
                    <filter string="Resource" name="resource" domain="[]" context="{'group_by':'resource_id'}"/>
                    <filter string="Company" name="company" domain="[]" context="{'group_by':'company_id'}" groups="base.group_multi_company"/>
                    <filter string="Date" name="leave_month" domain="[]" context="{'group_by':'date_from'}" help="Starting Date of Time Off"/>
                </group>
           </search>
        </field>
    </record>

    <record id="view_resource_calendar" model="ir.ui.view">
        <field name="name">resource.calendar.leaves.calendar</field>
        <field name="model">resource.calendar.leaves</field>
        <field name="arch" type="xml">
            <calendar date_start="date_from" date_stop="date_to" mode="month" string="Resource" color="resource_id" event_limit="5">
                <field name="resource_id" avatar_field="image_128" filters="1"/>
                <field name="company_id"/>
                <field name="name"/>
            </calendar>
        </field>
    </record>

    <record id="resource_calendar_leave_form" model="ir.ui.view">
        <field name="name">resource.calendar.leaves.form</field>
        <field name="model">resource.calendar.leaves</field>
        <field name="arch" type="xml">
            <form string="Time Off Detail">
            <field name="company_id" invisible="1"/>
            <sheet>
                <group>
                    <group name="leave_details">
                        <field name="name" string="Reason"/>
                        <field name="calendar_id"/>
                        <field name="company_id" options="{'no_create': True}" groups="base.group_multi_company" invisible="not calendar_id"/>
                        <field name="resource_id"/>
                    </group>
                    <group name="leave_dates">
                       <field name="date_from"/>
                       <field name="date_to"/>
                    </group>
                </group>
            </sheet>
            </form>
        </field>
    </record>

    <record id="resource_calendar_leave_tree" model="ir.ui.view">
        <field name="name">resource.calendar.leaves.list</field>
        <field name="model">resource.calendar.leaves</field>
        <field name="priority">1</field>
        <field name="arch" type="xml">
            <list string="Time Off Detail">
                <field name="name" string="Reason"/>
                <field name="resource_id" />
                <field name="company_id" groups="base.group_multi_company" optional="hide"/>
                <field name="calendar_id"/>
                <field name="date_from" />
                <field name="date_to" />
            </list>
        </field>
    </record>

    <record id="action_resource_calendar_leave_tree" model="ir.actions.act_window">
        <field name="name">Resource Time Off</field>
        <field name="res_model">resource.calendar.leaves</field>
        <field name="view_mode">list,form,calendar</field>
        <field name="search_view_id" ref="view_resource_calendar_leaves_search"/>
    </record>

    <record id="resource_calendar_leaves_action_from_calendar" model="ir.actions.act_window">
        <field name="name">Resource Time Off</field>
        <field name="res_model">resource.calendar.leaves</field>
        <field name="view_mode">list,form,calendar</field>
        <field name="context">{
            'default_calendar_id': active_id,
            'search_default_calendar_id': active_id}</field>
        <field name="search_view_id" ref="view_resource_calendar_leaves_search"/>
    </record>

    <record id="resource_calendar_closing_days" model="ir.actions.act_window">
        <field name="name">Closing Days</field>
        <field name="res_model">resource.calendar.leaves</field>
        <field name="view_mode">calendar,list,form</field>
        <field name="domain">[('calendar_id','=',active_id), ('resource_id','=',False)]</field>
        <field name="context">{'default_calendar_id': active_id}</field>
        <field name="binding_model_id" ref="model_resource_calendar"/>
        <field name="binding_view_types">form</field>
    </record>

    <record id="resource_calendar_resources_leaves" model="ir.actions.act_window">
        <field name="name">Resources Time Off</field>
        <field name="res_model">resource.calendar.leaves</field>
        <field name="view_mode">calendar,list,form</field>
        <field name="domain">[('calendar_id','=',active_id), ('resource_id','!=',False)]</field>
        <field name="context">{'default_calendar_id': active_id}</field>
        <field name="binding_model_id" ref="model_resource_calendar"/>
        <field name="binding_view_types">form</field>
    </record>
</odoo>

```

## File: views\resource_calendar_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_resource_calendar_search" model="ir.ui.view">
        <field name="name">resource.calendar.search</field>
        <field name="model">resource.calendar</field>
        <field name="arch" type="xml">
            <search string="Search Working Time">
               <field name="name" string="Working Time"/>
               <field name="company_id" groups="base.group_multi_company"/>
               <separator/>
               <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
           </search>
        </field>
    </record>

    <record id="resource_calendar_form" model="ir.ui.view">
        <field name="name">resource.calendar.form</field>
        <field name="model">resource.calendar</field>
        <field name="arch" type="xml">
            <form string="Working Time">
                <field name="two_weeks_calendar" invisible="1"/>
                <header>
                    <button name="switch_calendar_type" invisible="two_weeks_calendar or flexible_hours" string="Switch to 2 weeks calendar" type="object"
                        confirm="Are you sure you want to switch to a 2-week calendar? All work entries will be lost."
                        confirm-label="Switch"/>
                    <button name="switch_calendar_type" invisible="not two_weeks_calendar" string="Switch to 1 week calendar" type="object"
                        confirm="Are you sure you want to switch to a 1-week calendar? All work entries will be lost."
                        confirm-label="Switch"/>
                </header>
                <sheet string="Working Time">
                    <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                    <div class="oe_button_box" name="button_box">
                        <button name="%(resource_calendar_leaves_action_from_calendar)d" type="action"
                                icon="fa-plane"
                                class="oe_stat_button"
                                groups="base.group_no_one">
                                <div class="o_field_widget o_stat_info">
                                    <span class="o_stat_text">Time Off</span>
                                </div>
                        </button>
                        <button name="%(resource_resource_action_from_calendar)d" type="action"
                                icon="fa-cogs"
                                class="oe_stat_button"
                                groups="base.group_user">
                                <div class="o_field_widget o_stat_info">
                                    <span class="o_stat_text">Work Resources</span>
                                </div>
                        </button>
                    </div>
                    <div class="oe_title">
                        <h1><field name="name"/></h1>
                    </div>
                    <group name="resource_details" col="2">
                        <group name="resource_working_hours">
                            <field name="flexible_hours"/>
                            <label for="full_time_required_hours" invisible="flexible_hours"/>
                            <label for="full_time_required_hours" string="Hours per Week" invisible="not flexible_hours"/>
                            <div>
                                <field name="full_time_required_hours" style="width: 10%" widget="float_time"/>
                                <span> hours/week</span>
                            </div>
                            <field name="hours_per_day" widget="float_time" readonly="not flexible_hours"/>
                        </group>
                        <group name="resource_company">
                            <field name="active" invisible="1"/>
                            <field name="company_id" groups="base.group_multi_company"/>
                            <field name="tz" widget="timezone_mismatch" options="{'tz_offset_field': 'tz_offset', 'mismatch_title': 'Timezone Mismatch : This timezone is different from that of your browser.\nPlease, be mindful of this when setting the working hours or the time off.'}" />
                            <field name="tz_offset" invisible="1"/>
                        </group>
                    </group>
                    <notebook>
                        <page string="Working Hours" name="working_hours" invisible="flexible_hours">
                            <field name="two_weeks_explanation" invisible="not two_weeks_calendar"/>
                            <field name="attendance_ids" widget="section_one2many"/>
                        </page>
                    </notebook>
                </sheet>
            </form>
        </field>
    </record>

    <record id="view_resource_calendar_tree" model="ir.ui.view">
        <field name="name">resource.calendar.list</field>
        <field name="model">resource.calendar</field>
        <field name="arch" type="xml">
            <list string="Working Time">
                <field name="name" string="Working Time"/>
                <field name="company_id" groups="base.group_multi_company"/>
            </list>
        </field>
    </record>

    <record id="action_resource_calendar_form" model="ir.actions.act_window">
        <field name="name">Working Schedules</field>
        <field name="res_model">resource.calendar</field>
        <field name="view_mode">list,form</field>
        <field name="domain">['|', ('company_id', '=', False), ('company_id', 'in', allowed_company_ids)]</field>
        <field name="view_id" eval="False"/>
        <field name="search_view_id" ref="view_resource_calendar_search"/>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Define working hours and time table that could be scheduled to your project members
          </p>
        </field>
    </record>
</odoo>

```

## File: views\resource_resource_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- RESOURCE.RESOURCE -->
    <record id="view_resource_resource_search" model="ir.ui.view">
        <field name="name">resource.resource.search</field>
        <field name="model">resource.resource</field>
        <field name="arch" type="xml">
            <search string="Search Resource">
               <field name="name" />
               <field name="resource_type"/>
               <field name="user_id"/>
               <field name="calendar_id"/>
               <field name="company_id" groups="base.group_multi_company"/>
               <filter string="Human" name="human" domain="[('resource_type','=', 'user')]"/>
               <filter string="Material" name="material" domain="[('resource_type','=', 'material')]"/>
               <separator />
               <filter string="Archived" name="inactive" domain="[('active','=',False)]"/>
               <group expand="0" string="Group By">
                    <filter string="User" name="user" domain="[]" context="{'group_by':'user_id'}"/>
                    <filter string="Type" name="type" domain="[]" context="{'group_by':'resource_type'}"/>
                    <filter string="Company" name="company" domain="[]" context="{'group_by':'company_id'}" groups="base.group_multi_company"/>
                    <filter string="Working Time" name="working_period" domain="[]" context="{'group_by':'calendar_id'}"/>
                </group>
           </search>
        </field>
    </record>

    <record id="resource_resource_form" model="ir.ui.view">
        <field name="name">resource.resource.form</field>
        <field name="model">resource.resource</field>
        <field name="arch" type="xml">
            <form string="Resource">
            <sheet>
                <field name="active" invisible="1" />
                <field name="company_id" invisible="1"/>
                <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                <group>
                    <group name="user_details">
                        <field name="name"/>
                        <field name="user_id" invisible="resource_type == 'material'" required="resource_type == 'user'"/>
                        <field name="resource_type" />
                    </group>
                    <group name="resource_details">
                        <field name="company_id" options="{'no_create': True}" groups="base.group_multi_company"/>
                        <field name="calendar_id" placeholder="Fully Flexible"/>
                        <field name="tz"/>
                        <field name="time_efficiency"/>
                    </group>
                </group>
            </sheet>
            </form>
        </field>
    </record>

    <record id="resource_resource_tree" model="ir.ui.view">
        <field name="name">resource.resource.list</field>
        <field name="model">resource.resource</field>
        <field name="arch" type="xml">
            <list string="Resources" multi_edit="1" default_order="name">
                <field name="company_id" column_invisible="True"/>
                <field name="name" />
                <field name="user_id" />
                <field name="company_id" groups="base.group_multi_company" options="{'no_create': True}" optional="show"/>
                <field name="calendar_id" optional="show" />
                <field name="tz" optional="hide" />
                <field name="resource_type" optional="show" />
                <field name="time_efficiency"/>
            </list>
        </field>
    </record>

    <record id="action_resource_resource_tree" model="ir.actions.act_window">
        <field name="name">Resources</field>
        <field name="res_model">resource.resource</field>
        <field name="view_mode">list,form</field>
        <field name="context">{}</field>
        <field name="search_view_id" ref="view_resource_resource_search"/>
        <field name="help">Resources allow you to create and manage resources that should be involved in a specific project phase. You can also set their efficiency level and workload based on their weekly working hours.</field>
    </record>

    <record id="resource_resource_action_from_calendar" model="ir.actions.act_window">
        <field name="name">Resources</field>
        <field name="res_model">resource.resource</field>
        <field name="view_mode">list,form</field>
        <field name="context">{
            'default_calendar_id': active_id,
            'search_default_calendar_id': active_id}</field>
        <field name="search_view_id" ref="view_resource_resource_search"/>
        <field name="help">Resources allow you to create and manage resources that should be involved in a specific project phase. You can also set their efficiency level and workload based on their weekly working hours.</field>
    </record>
</odoo>

```

