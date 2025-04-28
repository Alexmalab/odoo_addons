# Odoo Module: resource

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from .tests import test_resource_model

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
        'views/resource_views.xml',
        'views/resource.xml',
    ],
    'demo': [
    ],
    'license': 'LGPL-3',
}

```

## File: data\resource_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<data>

    <record id="resource_calendar_std" model="resource.calendar">
        <field name="name">Standard 40 hours/week</field>
        <field name="company_id" ref="base.main_company"/>
    </record>

    <record id="base.main_company" model="res.company">
        <field name="resource_calendar_id" ref="resource_calendar_std"/>
    </record>

    <function
        model="res.company"
        name="_init_data_resource_calendar"
        eval="[]"/>

</data>

<data noupdate="1">

    <record id="resource_calendar_std_35h" model="resource.calendar">
        <field name="name">Standard 35 hours/week</field>
        <field name="company_id" ref="base.main_company"/>
        <field name="hours_per_day">7.0</field>
        <field name="attendance_ids"
            eval="[
                (0, 0, {'name': 'Monday Morning', 'dayofweek': '0', 'hour_from': 8, 'hour_to': 12, 'day_period': 'morning'}),
                (0, 0, {'name': 'Monday Afternoon', 'dayofweek': '0', 'hour_from': 13, 'hour_to': 16, 'day_period': 'afternoon'}),
                (0, 0, {'name': 'Tuesday Morning', 'dayofweek': '1', 'hour_from': 8, 'hour_to': 12, 'day_period': 'morning'}),
                (0, 0, {'name': 'Tuesday Afternoon', 'dayofweek': '1', 'hour_from': 13, 'hour_to': 16, 'day_period': 'afternoon'}),
                (0, 0, {'name': 'Wednesday Morning', 'dayofweek': '2', 'hour_from': 8, 'hour_to': 12, 'day_period': 'morning'}),
                (0, 0, {'name': 'Wednesday Afternoon', 'dayofweek': '2', 'hour_from': 13, 'hour_to': 16, 'day_period': 'afternoon'}),
                (0, 0, {'name': 'Thursday Morning', 'dayofweek': '3', 'hour_from': 8, 'hour_to': 12, 'day_period': 'morning'}),
                (0, 0, {'name': 'Thursday Afternoon', 'dayofweek': '3', 'hour_from': 13, 'hour_to': 16, 'day_period': 'afternoon'}),
                (0, 0, {'name': 'Friday Morning', 'dayofweek': '4', 'hour_from': 8, 'hour_to': 12, 'day_period': 'morning'}),
                (0, 0, {'name': 'Friday Afternoon', 'dayofweek': '4', 'hour_from': 13, 'hour_to': 16, 'day_period': 'afternoon'})
            ]"
        />
    </record>

    <record id="resource_calendar_std_38h" model="resource.calendar">
        <field name="name">Standard 38 hours/week</field>
        <field name="company_id" ref="base.main_company"/>
        <field name="hours_per_day">7.6</field>
        <field name="attendance_ids"
            eval="[
                (0, 0, {'name': 'Monday Morning', 'dayofweek': '0', 'hour_from': 8, 'hour_to': 12, 'day_period': 'morning'}),
                (0, 0, {'name': 'Monday Afternoon', 'dayofweek': '0', 'hour_from': 13, 'hour_to': 16.6, 'day_period': 'afternoon'}),
                (0, 0, {'name': 'Tuesday Morning', 'dayofweek': '1', 'hour_from': 8, 'hour_to': 12, 'day_period': 'morning'}),
                (0, 0, {'name': 'Tuesday Afternoon', 'dayofweek': '1', 'hour_from': 13, 'hour_to': 16.6, 'day_period': 'afternoon'}),
                (0, 0, {'name': 'Wednesday Morning', 'dayofweek': '2', 'hour_from': 8, 'hour_to': 12, 'day_period': 'morning'}),
                (0, 0, {'name': 'Wednesday Afternoon', 'dayofweek': '2', 'hour_from': 13, 'hour_to': 16.6, 'day_period': 'afternoon'}),
                (0, 0, {'name': 'Thursday Morning', 'dayofweek': '3', 'hour_from': 8, 'hour_to': 12, 'day_period': 'morning'}),
                (0, 0, {'name': 'Thursday Afternoon', 'dayofweek': '3', 'hour_from': 13, 'hour_to': 16.6, 'day_period': 'afternoon'}),
                (0, 0, {'name': 'Friday Morning', 'dayofweek': '4', 'hour_from': 8, 'hour_to': 12, 'day_period': 'morning'}),
                (0, 0, {'name': 'Friday Afternoon', 'dayofweek': '4', 'hour_from': 13, 'hour_to': 16.6, 'day_period': 'afternoon'})
            ]"
        />
    </record>

</data>

</odoo>

```

## File: models\resource.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict
import math
from datetime import datetime, time, timedelta
from dateutil.relativedelta import relativedelta
from dateutil.rrule import rrule, DAILY, WEEKLY
from functools import partial
from itertools import chain
from pytz import timezone, utc

from odoo import api, fields, models, _
from odoo.addons.base.models.res_partner import _tz_get
from odoo.exceptions import ValidationError
from odoo.osv import expression
from odoo.tools.float_utils import float_round

from odoo.tools import date_utils, float_utils
from .resource_mixin import timezone_datetime

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
    else:
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
        res = super(ResourceCalendar, self).default_get(fields)
        if not res.get('name') and res.get('company_id'):
            res['name'] = _('Working Hours of %s') % self.env['res.company'].browse(res['company_id']).name
        if 'attendance_ids' in fields and not res.get('attendance_ids'):
            res['attendance_ids'] = [
                (0, 0, {'name': _('Monday Morning'), 'dayofweek': '0', 'hour_from': 8, 'hour_to': 12, 'day_period': 'morning'}),
                (0, 0, {'name': _('Monday Afternoon'), 'dayofweek': '0', 'hour_from': 13, 'hour_to': 17, 'day_period': 'afternoon'}),
                (0, 0, {'name': _('Tuesday Morning'), 'dayofweek': '1', 'hour_from': 8, 'hour_to': 12, 'day_period': 'morning'}),
                (0, 0, {'name': _('Tuesday Afternoon'), 'dayofweek': '1', 'hour_from': 13, 'hour_to': 17, 'day_period': 'afternoon'}),
                (0, 0, {'name': _('Wednesday Morning'), 'dayofweek': '2', 'hour_from': 8, 'hour_to': 12, 'day_period': 'morning'}),
                (0, 0, {'name': _('Wednesday Afternoon'), 'dayofweek': '2', 'hour_from': 13, 'hour_to': 17, 'day_period': 'afternoon'}),
                (0, 0, {'name': _('Thursday Morning'), 'dayofweek': '3', 'hour_from': 8, 'hour_to': 12, 'day_period': 'morning'}),
                (0, 0, {'name': _('Thursday Afternoon'), 'dayofweek': '3', 'hour_from': 13, 'hour_to': 17, 'day_period': 'afternoon'}),
                (0, 0, {'name': _('Friday Morning'), 'dayofweek': '4', 'hour_from': 8, 'hour_to': 12, 'day_period': 'morning'}),
                (0, 0, {'name': _('Friday Afternoon'), 'dayofweek': '4', 'hour_from': 13, 'hour_to': 17, 'day_period': 'afternoon'})
            ]
        return res

    name = fields.Char(required=True)
    company_id = fields.Many2one(
        'res.company', 'Company',
        default=lambda self: self.env.company)
    attendance_ids = fields.One2many(
        'resource.calendar.attendance', 'calendar_id', 'Working Time',
        copy=True)
    leave_ids = fields.One2many(
        'resource.calendar.leaves', 'calendar_id', 'Time Off')
    global_leave_ids = fields.One2many(
        'resource.calendar.leaves', 'calendar_id', 'Global Time Off',
        domain=[('resource_id', '=', False)], copy=True,
        )
    hours_per_day = fields.Float("Average Hour per Day", default=HOURS_PER_DAY,
                                 help="Average hours per day a resource is supposed to work with this calendar.")
    tz = fields.Selection(
        _tz_get, string='Timezone', required=True,
        default=lambda self: self._context.get('tz') or self.env.user.tz or 'UTC',
        help="This field is used in order to define in which timezone the resources will work.")
    two_weeks_calendar = fields.Boolean(string="Calendar in 2 weeks mode")
    two_weeks_explanation = fields.Char('Explanation', compute="_compute_two_weeks_explanation")

    @api.returns('self', lambda value: value.id)
    def copy(self, default=None):
        self.ensure_one()
        if default is None:
            default = {}
        if not default.get('name'):
            default.update(name=_('%s (copy)') % (self.name))
        return super(ResourceCalendar, self).copy(default)

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
        week_type = _("odd") if int(math.floor((today.toordinal() - 1) / 7) % 2) else _("even")
        first_day = date_utils.start_of(today, 'week')
        last_day = date_utils.end_of(today, 'week')
        self.two_weeks_explanation = "This week (from %s to %s) is an %s week." % (first_day, last_day, week_type)

    def _get_global_attendances(self):
        return self.attendance_ids.filtered(lambda attendance:
            not attendance.date_from and not attendance.date_to
            and not attendance.resource_id and not attendance.display_type)

    def _compute_hours_per_day(self, attendances):
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

        return float_round(hour_count / float(number_of_days), precision_digits=2)

    @api.onchange('attendance_ids', 'two_weeks_calendar')
    def _onchange_hours_per_day(self):
        attendances = self._get_global_attendances()
        self.hours_per_day = self._compute_hours_per_day(attendances)

    def switch_calendar_type(self):
        if not self.two_weeks_calendar:
            self.attendance_ids.unlink()
            self.attendance_ids = [
                (0, 0, {
                    'name': 'Even week',
                    'dayofweek': '0',
                    'sequence': '0',
                    'hour_from': 0,
                    'day_period': 'morning',
                    'week_type': '0',
                    'hour_to': 0,
                    'display_type':
                    'line_section'}),
                (0, 0, {
                    'name': 'Odd week',
                    'dayofweek': '0',
                    'sequence': '25',
                    'hour_from': 0,
                    'day_period':
                    'morning',
                    'week_type': '1',
                    'hour_to': 0,
                    'display_type': 'line_section'}),
            ]

            self.two_weeks_calendar = True
            default_attendance = self.default_get('attendance_ids')['attendance_ids']
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
            self.attendance_ids = self.default_get('attendance_ids')['attendance_ids']
        self._onchange_hours_per_day()

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
    # YTI TODO: Remove me in master
    def _attendance_intervals(self, start_dt, end_dt, resource=None, domain=None, tz=None):
        if resource is None:
            resource = self.env['resource.resource']
        return self._attendance_intervals_batch(
            start_dt, end_dt, resources=resource, domain=domain, tz=tz
        )[resource.id]

    def _attendance_intervals_batch(self, start_dt, end_dt, resources=None, domain=None, tz=None):
        """ Return the attendance intervals in the given datetime range.
            The returned intervals are expressed in specified tz or in the resource's timezone.
        """
        self.ensure_one()
        resources = self.env['resource.resource'] if not resources else resources
        assert start_dt.tzinfo and end_dt.tzinfo
        combine = datetime.combine

        resources_list = list(resources) + [self.env['resource.resource']]
        resource_ids = [r.id for r in resources_list]
        domain = domain if domain is not None else []
        domain = expression.AND([domain, [
            ('calendar_id', '=', self.id),
            ('resource_id', 'in', resource_ids),
            ('display_type', '=', False),
        ]])

        # for each attendance spec, generate the intervals in the date range
        cache_dates = defaultdict(dict)
        cache_deltas = defaultdict(dict)
        result = defaultdict(list)
        for attendance in self.env['resource.calendar.attendance'].search(domain):
            for resource in resources_list:
                # express all dates and times in specified tz or in the resource's timezone
                tz = tz if tz else timezone((resource or self).tz)
                if (tz, start_dt) in cache_dates:
                    start = cache_dates[(tz, start_dt)]
                else:
                    start = start_dt.astimezone(tz)
                    cache_dates[(tz, start_dt)] = start
                if (tz, end_dt) in cache_dates:
                    end = cache_dates[(tz, end_dt)]
                else:
                    end = end_dt.astimezone(tz)
                    cache_dates[(tz, end_dt)] = end

                start = start.date()
                if attendance.date_from:
                    start = max(start, attendance.date_from)
                until = end.date()
                if attendance.date_to:
                    until = min(until, attendance.date_to)
                if attendance.week_type:
                    start_week_type = int(math.floor((start.toordinal()-1)/7) % 2)
                    if start_week_type != int(attendance.week_type):
                        # start must be the week of the attendance
                        # if it's not the case, we must remove one week
                        start = start + relativedelta(weeks=-1)
                weekday = int(attendance.dayofweek)

                if self.two_weeks_calendar and attendance.week_type:
                    days = rrule(WEEKLY, start, interval=2, until=until, byweekday=weekday)
                else:
                    days = rrule(DAILY, start, until=until, byweekday=weekday)

                for day in days:
                    # We need to exclude incorrect days according to re-defined start previously
                    # with weeks=-1 (Note: until is correctly handled)
                    if (self.two_weeks_calendar and attendance.date_from and attendance.date_from > day.date()):
                        continue
                    # attendance hours are interpreted in the resource's timezone
                    hour_from = attendance.hour_from
                    if (tz, day, hour_from) in cache_deltas:
                        dt0 = cache_deltas[(tz, day, hour_from)]
                    else:
                        dt0 = tz.localize(combine(day, float_to_time(hour_from)))
                        cache_deltas[(tz, day, hour_from)] = dt0

                    hour_to = attendance.hour_to
                    if (tz, day, hour_to) in cache_deltas:
                        dt1 = cache_deltas[(tz, day, hour_to)]
                    else:
                        dt1 = tz.localize(combine(day, float_to_time(hour_to)))
                        cache_deltas[(tz, day, hour_to)] = dt1
                    result[resource.id].append((max(cache_dates[(tz, start_dt)], dt0), min(cache_dates[(tz, end_dt)], dt1), attendance))
        return {r.id: Intervals(result[r.id]) for r in resources_list}

    def _leave_intervals(self, start_dt, end_dt, resource=None, domain=None, tz=None):
        if resource is None:
            resource = self.env['resource.resource']
        return self._leave_intervals_batch(
            start_dt, end_dt, resources=resource, domain=domain, tz=tz
        )[resource.id]

    def _leave_intervals_batch(self, start_dt, end_dt, resources=None, domain=None, tz=None):
        """ Return the leave intervals in the given datetime range.
            The returned intervals are expressed in specified tz or in the calendar's timezone.
        """
        resources = self.env['resource.resource'] if not resources else resources
        assert start_dt.tzinfo and end_dt.tzinfo
        self.ensure_one()

        # for the computation, express all datetimes in UTC
        resources_list = list(resources) + [self.env['resource.resource']]
        resource_ids = [r.id for r in resources_list]
        if domain is None:
            domain = [('time_type', '=', 'leave')]
        domain = domain + [
            ('calendar_id', '=', self.id),
            ('resource_id', 'in', resource_ids),
            ('date_from', '<=', datetime_to_string(end_dt)),
            ('date_to', '>=', datetime_to_string(start_dt)),
        ]

        # retrieve leave intervals in (start_dt, end_dt)
        result = defaultdict(lambda: [])
        tz_dates = {}
        for leave in self.env['resource.calendar.leaves'].search(domain):
            for resource in resources_list:
                if leave.resource_id.id not in [False, resource.id]:
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
                dt0 = string_to_datetime(leave.date_from).astimezone(tz)
                dt1 = string_to_datetime(leave.date_to).astimezone(tz)
                result[resource.id].append((max(start, dt0), min(end, dt1), leave))

        return {r.id: Intervals(result[r.id]) for r in resources_list}

    # YTI TODO: Remove me in master
    def _work_intervals(self, start_dt, end_dt, resource=None, domain=None, tz=None):
        if resource is None:
            resource = self.env['resource.resource']
        return self._work_intervals_batch(
            start_dt, end_dt, resources=resource, domain=domain, tz=tz
        )[resource.id]

    def _work_intervals_batch(self, start_dt, end_dt, resources=None, domain=None, tz=None):
        """ Return the effective work intervals between the given datetimes. """
        if not resources:
            resources = self.env['resource.resource']
            resources_list = [resources]
        else:
            resources_list = list(resources)

        attendance_intervals = self._attendance_intervals_batch(start_dt, end_dt, resources, tz=tz)
        leave_intervals = self._leave_intervals_batch(start_dt, end_dt, resources, domain, tz=tz)
        return {
            r.id: (attendance_intervals[r.id] - leave_intervals[r.id]) for r in resources_list
        }

    # --------------------------------------------------
    # Private Methods / Helpers
    # --------------------------------------------------

    def _get_days_data(self, intervals, day_total):
        """
        helper function to compute duration of `intervals`
        expressed in days and hours.
        `day_total` is a dict {date: n_hours} with the number of hours for each day.
        """
        day_hours = defaultdict(float)
        for start, stop, meta in intervals:
            day_hours[start.date()] += (stop - start).total_seconds() / 3600

        # compute number of days as quarters
        days = sum(
            float_utils.round(ROUNDING_FACTOR * day_hours[day] / day_total[day]) / ROUNDING_FACTOR
            for day in day_hours
        )
        return {
            'days': days,
            'hours': sum(day_hours.values()),
        }

    # YTI TODO: Remove me in master
    def _get_day_total(self, from_datetime, to_datetime, resource=None):
        if resource is None:
            resource = self.env['resource.resource']
        return self._get_resources_day_total(from_datetime, to_datetime, resources=resource)[resource.id]

    def _get_resources_day_total(self, from_datetime, to_datetime, resources=None):
        """
        @return dict with hours of attendance in each day between `from_datetime` and `to_datetime`
        """
        self.ensure_one()
        resources = self.env['resource.resource'] if not resources else resources
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

        day_total = self._get_resources_day_total(from_datetime, to_datetime)[False]

        # actual hours per day
        if compute_leaves:
            intervals = self._work_intervals_batch(from_datetime, to_datetime, domain=domain)[False]
        else:
            intervals = self._attendance_intervals_batch(from_datetime, to_datetime)[False]

        return self._get_days_data(intervals, day_total)

    def plan_hours(self, hours, day_dt, compute_leaves=False, domain=None, resource=None):
        """
        `compute_leaves` controls whether or not this method is taking into
        account the global leaves.

        `domain` controls the way leaves are recognized.
        None means default value ('time_type', '=', 'leave')

        Return datetime after having planned hours
        """
        day_dt, revert = make_aware(day_dt)

        # which method to use for retrieving intervals
        if compute_leaves:
            get_intervals = partial(self._work_intervals, domain=domain, resource=resource)
        else:
            get_intervals = self._attendance_intervals

        if hours >= 0:
            delta = timedelta(days=14)
            for n in range(100):
                dt = day_dt + delta * n
                for start, stop, meta in get_intervals(dt, dt + delta):
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
                for start, stop, meta in reversed(get_intervals(dt - delta, dt)):
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
            get_intervals = partial(self._work_intervals, domain=domain)
        else:
            get_intervals = self._attendance_intervals

        if days > 0:
            found = set()
            delta = timedelta(days=14)
            for n in range(100):
                dt = day_dt + delta * n
                for start, stop, meta in get_intervals(dt, dt + delta):
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
                for start, stop, meta in reversed(get_intervals(dt - delta, dt)):
                    found.add(start.date())
                    if len(found) == days:
                        return revert(start)
            return False

        else:
            return revert(day_dt)


class ResourceCalendarAttendance(models.Model):
    _name = "resource.calendar.attendance"
    _description = "Work Detail"
    _order = 'week_type, dayofweek, hour_from'

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
    calendar_id = fields.Many2one("resource.calendar", string="Resource's Calendar", required=True, ondelete='cascade')
    day_period = fields.Selection([('morning', 'Morning'), ('afternoon', 'Afternoon')], required=True, default='morning')
    resource_id = fields.Many2one('resource.resource', 'Resource')
    week_type = fields.Selection([
        ('1', 'Odd week'),
        ('0', 'Even week')
        ], 'Week Even/Odd', default=False)
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
        self.hour_to = min(self.hour_to, 23.99)
        self.hour_to = max(self.hour_to, 0.0)

        # avoid wrong order
        self.hour_to = max(self.hour_to, self.hour_from)


class ResourceResource(models.Model):
    _name = "resource.resource"
    _description = "Resources"

    @api.model
    def default_get(self, fields):
        res = super(ResourceResource, self).default_get(fields)
        if not res.get('calendar_id') and res.get('company_id'):
            company = self.env['res.company'].browse(res['company_id'])
            res['calendar_id'] = company.resource_calendar_id.id
        return res

    name = fields.Char(required=True)
    active = fields.Boolean(
        'Active', default=True, tracking=True,
        help="If the active field is set to False, it will allow you to hide the resource record without removing it.")
    company_id = fields.Many2one('res.company', string='Company', default=lambda self: self.env.company)
    resource_type = fields.Selection([
        ('user', 'Human'),
        ('material', 'Material')], string='Resource Type',
        default='user', required=True)
    user_id = fields.Many2one('res.users', string='User', help='Related user name for the resource to manage its access.')
    time_efficiency = fields.Float(
        'Efficiency Factor', default=100, required=True,
        help="This field is used to calculate the the expected duration of a work order at this work center. For example, if a work order takes one hour and the efficiency factor is 100%, then the expected duration will be one hour. If the efficiency factor is 200%, however the expected duration will be 30 minutes.")
    calendar_id = fields.Many2one(
        "resource.calendar", string='Working Time',
        default=lambda self: self.env.company.resource_calendar_id,
        required=True,
        help="Define the schedule of resource")
    tz = fields.Selection(
        _tz_get, string='Timezone', required=True,
        default=lambda self: self._context.get('tz') or self.env.user.tz or 'UTC',
        help="This field is used in order to define in which timezone the resources will work.")

    _sql_constraints = [
        ('check_time_efficiency', 'CHECK(time_efficiency>0)', 'Time efficiency must be strictly positive'),
    ]

    @api.constrains('time_efficiency')
    def _check_time_efficiency(self):
        for record in self:
            if record.time_efficiency == 0:
                raise ValidationError(_('The efficiency factor cannot be equal to 0.'))

    @api.model_create_multi
    def create(self, vals_list):
        for values in vals_list:
            if values.get('company_id') and not values.get('calendar_id'):
                values['calendar_id'] = self.env['res.company'].browse(values['company_id']).resource_calendar_id.id
            if not values.get('tz'):
                # retrieve timezone on user or calendar
                tz = (self.env['res.users'].browse(values.get('user_id')).tz or
                      self.env['resource.calendar'].browse(values.get('calendar_id')).tz)
                if tz:
                    values['tz'] = tz
        return super(ResourceResource, self).create(vals_list)

    @api.returns('self', lambda value: value.id)
    def copy(self, default=None):
        self.ensure_one()
        if default is None:
            default = {}
        if not default.get('name'):
            default.update(name=_('%s (copy)') % (self.name))
        return super(ResourceResource, self).copy(default)

    @api.onchange('company_id')
    def _onchange_company_id(self):
        if self.company_id:
            self.calendar_id = self.company_id.resource_calendar_id.id

    @api.onchange('user_id')
    def _onchange_user_id(self):
        if self.user_id:
            self.tz = self.user_id.tz

    def _get_work_interval(self, start, end):
        """ Return interval's start datetime for interval closest to start. And interval's end datetime for interval closest to end.
            If none is found return None
            Note: this method is used in enterprise (forecast and planning)

            :start: datetime
            :end: datetime
            :return: (datetime|None, datetime|None)
        """
        start_datetime = timezone_datetime(start)
        end_datetime = timezone_datetime(end)
        resource_mapping = {}
        for resource in self:
            work_intervals = sorted(
                resource.calendar_id._work_intervals_batch(start_datetime, end_datetime, resource)[resource.id],
                key=lambda x: x[0]
            )
            if work_intervals:
                resource_mapping[resource.id] = (work_intervals[0][0].astimezone(utc), work_intervals[-1][1].astimezone(utc))
            else:
                resource_mapping[resource.id] = (None, None)
        return resource_mapping

    def _get_unavailable_intervals(self, start, end):
        """ Compute the intervals during which employee is unavailable with hour granularity between start and end
            Note: this method is used in enterprise (forecast and planning)

        """
        start_datetime = timezone_datetime(start)
        end_datetime = timezone_datetime(end)
        resource_mapping = {}
        calendar_mapping = defaultdict(lambda: self.env['resource.resource'])
        for resource in self:
            calendar_mapping[resource.calendar_id] |= resource

        for calendar, resources in calendar_mapping.items():
            resources_work_intervals = calendar._work_intervals_batch(start_datetime, end_datetime, resources)
            for resource in resources:
                resource_work_intervals = resources_work_intervals[resource.id]
                resource_work_intervals = [(start, stop) for start, stop, meta in resource_work_intervals]
                # start + flatten(intervals) + end
                resource_work_intervals = [start_datetime] + list(chain.from_iterable(resource_work_intervals)) + [end_datetime]
                # put it back to UTC
                resource_work_intervals = list(map(lambda dt: dt.astimezone(utc), resource_work_intervals))
                # pick groups of two
                resource_work_intervals = list(zip(resource_work_intervals[0::2], resource_work_intervals[1::2]))
                resource_mapping[resource.id] = resource_work_intervals
        return resource_mapping


class ResourceCalendarLeaves(models.Model):
    _name = "resource.calendar.leaves"
    _description = "Resource Time Off Detail"
    _order = "date_from"

    name = fields.Char('Reason')
    company_id = fields.Many2one(
        'res.company', related='calendar_id.company_id', string="Company",
        readonly=True, store=True)
    calendar_id = fields.Many2one('resource.calendar', 'Working Hours')
    date_from = fields.Datetime('Start Date', required=True)
    date_to = fields.Datetime('End Date', required=True)
    resource_id = fields.Many2one(
        "resource.resource", 'Resource',
        help="If empty, this is a generic time off for the company. If a resource is set, the time off is only for this resource")
    time_type = fields.Selection([('leave', 'Time Off'), ('other', 'Other')], default='leave',
                                 help="Whether this should be computed as a time off or as work time (eg: formation)")

    @api.constrains('date_from', 'date_to')
    def check_dates(self):
        if self.filtered(lambda leave: leave.date_from > leave.date_to):
            raise ValidationError(_('The start date of the time off must be earlier end date.'))

    @api.onchange('resource_id')
    def onchange_resource(self):
        if self.resource_id:
            self.calendar_id = self.resource_id.calendar_id

```

## File: models\resource_mixin.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict
from pytz import utc

from odoo import api, fields, models


def timezone_datetime(time):
    if not time.tzinfo:
        time = time.replace(tzinfo=utc)
    return time


class ResourceMixin(models.AbstractModel):
    _name = "resource.mixin"
    _description = 'Resource Mixin'

    resource_id = fields.Many2one(
        'resource.resource', 'Resource',
        auto_join=True, index=True, ondelete='restrict', required=True)
    company_id = fields.Many2one(
        'res.company', 'Company',
        default=lambda self: self.env.company,
        index=True, related='resource_id.company_id', store=True, readonly=False)
    resource_calendar_id = fields.Many2one(
        'resource.calendar', 'Working Hours',
        default=lambda self: self.env.company.resource_calendar_id,
        index=True, related='resource_id.calendar_id', store=True, readonly=False)
    tz = fields.Selection(
        string='Timezone', related='resource_id.tz', readonly=False,
        help="This field is used in order to define in which timezone the resources will work.")

    @api.model
    def create(self, values):
        if not values.get('resource_id'):
            resource_vals = {'name': values.get(self._rec_name)}
            tz = (values.pop('tz', False) or
                  self.env['resource.calendar'].browse(values.get('resource_calendar_id')).tz)
            if tz:
                resource_vals['tz'] = tz
            resource = self.env['resource.resource'].create(resource_vals)
            values['resource_id'] = resource.id
        return super(ResourceMixin, self).create(values)

    def copy_data(self, default=None):
        if default is None:
            default = {}
        resource = self.resource_id.copy()
        default['resource_id'] = resource.id
        default['company_id'] = resource.company_id.id
        default['resource_calendar_id'] = resource.calendar_id.id
        return super(ResourceMixin, self).copy_data(default)

    # YTI TODO: Remove me in master
    def _get_work_days_data(self, from_datetime, to_datetime, compute_leaves=True, calendar=None, domain=None):
        self.ensure_one()
        return self._get_work_days_data_batch(
            from_datetime,
            to_datetime,
            compute_leaves=compute_leaves,
            calendar=calendar,
            domain=domain
        )[self.id]

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

        mapped_resources = defaultdict(lambda: self.env['resource.resource'])
        for record in self:
            mapped_resources[calendar or record.resource_calendar_id] |= record.resource_id

        for calendar, calendar_resources in mapped_resources.items():
            day_total = calendar._get_resources_day_total(from_datetime, to_datetime, calendar_resources)

            # actual hours per day
            if compute_leaves:
                intervals = calendar._work_intervals_batch(from_datetime, to_datetime, calendar_resources, domain)
            else:
                intervals = calendar._attendance_intervals_batch(from_datetime, to_datetime, calendar_resources)

            for calendar_resource in calendar_resources:
                result[calendar_resource.id] = calendar._get_days_data(intervals[calendar_resource.id], day_total[calendar_resource.id])

        # convert "resource: result" into "employee: result"
        return {mapped_employees[r.id]: result[r.id] for r in resources} 

    # YTI TODO: Remove me in master
    def _get_leave_days_data(self, from_datetime, to_datetime, calendar=None, domain=None):
        self.ensure_one()
        return self._get_leave_days_data_batch(
            from_datetime,
            to_datetime,
            calendar=calendar,
            domain=domain
        )[self.id]

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
            day_total = calendar._get_resources_day_total(from_datetime, to_datetime, calendar_resources)

            # compute actual hours per day
            attendances = calendar._attendance_intervals_batch(from_datetime, to_datetime, calendar_resources)
            leaves = calendar._leave_intervals_batch(from_datetime, to_datetime, calendar_resources, domain)

            for calendar_resource in calendar_resources:
                result[calendar_resource.id] = calendar._get_days_data(
                    attendances[calendar_resource.id] & leaves[calendar_resource.id],
                    day_total[calendar_resource.id]
                )

        # convert "resource: result" into "employee: result"
        return {mapped_employees[r.id]: result[r.id] for r in resources}

    def list_work_time_per_day(self, from_datetime, to_datetime, calendar=None, domain=None):
        """
            By default the resource calendar is used, but it can be
            changed using the `calendar` argument.

            `domain` is used in order to recognise the leaves to take,
            None means default value ('time_type', '=', 'leave')

            Returns a list of tuples (day, hours) for each day
            containing at least an attendance.
        """
        resource = self.resource_id
        calendar = calendar or self.resource_calendar_id

        # naive datetimes are made explicit in UTC
        if not from_datetime.tzinfo:
            from_datetime = from_datetime.replace(tzinfo=utc)
        if not to_datetime.tzinfo:
            to_datetime = to_datetime.replace(tzinfo=utc)

        intervals = calendar._work_intervals_batch(from_datetime, to_datetime, resource, domain)[resource.id]
        result = defaultdict(float)
        for start, stop, meta in intervals:
            result[start.date()] += (stop - start).total_seconds() / 3600
        return sorted(result.items())

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
        for company in self:
            company.resource_calendar_id = self.env['resource.calendar'].create({
                'name': _('Standard 40 hours/week'),
                'company_id': company.id
            }).id

    @api.model
    def create(self, values):
        company = super(ResCompany, self).create(values)
        if not company.resource_calendar_id:
            company.sudo()._create_resource_calendar()
        # calendar created from form view: no company_id set because record was still not created
        if not company.resource_calendar_id.company_id:
            company.resource_calendar_id.company_id = company.id
        return company

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

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import resource
from . import resource_mixin
from . import res_company
from . import res_users

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
access_resource_test_all,resource.test.all,model_resource_test,base.group_user,1,1,1,1
```

## File: security\resource_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data noupdate="1">

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
</data>
</odoo>

```

## File: static\src\js\section_fields_backend.js

```javascript

odoo.define('resource.section_backend', function (require) {
// The goal of this file is to contain JS hacks related to allowing
// section on resource calendar.

"use strict";
var FieldOne2Many = require('web.relational_fields').FieldOne2Many;
var fieldRegistry = require('web.field_registry');
var ListRenderer = require('web.ListRenderer');

var SectionListRenderer = ListRenderer.extend({
    /**
     * We want section to take the whole line (except handle and trash)
     * to look better and to hide the unnecessary fields.
     *
     * @override
     */
    _renderBodyCell: function (record, node, index, options) {
        var $cell = this._super.apply(this, arguments);

        var isSection = record.data.display_type === 'line_section';

        if (isSection) {
            if (node.attrs.widget === "handle") {
                return $cell;
            } else if (node.attrs.name === "name") {
                var nbrColumns = this._getNumberOfCols();
                if (this.handleField) {
                    nbrColumns--;
                }
                if (this.addTrashIcon) {
                    nbrColumns--;
                }
                $cell.attr('colspan', nbrColumns);
            } else {
                return $cell.addClass('o_hidden');
            }
        }

        return $cell;
    },
    /**
     * We add the o_is_{display_type} class to allow custom behaviour both in JS and CSS.
     *
     * @override
     */
    _renderRow: function (record, index) {
        var $row = this._super.apply(this, arguments);

        if (record.data.display_type) {
            $row.addClass('o_is_' + record.data.display_type);
        }

        return $row;
    },
    /**
     * We want to add .o_section_list_view on the table to have stronger CSS.
     *
     * @override
     * @private
     */
    _renderView: function () {
        var self = this;
        return this._super.apply(this, arguments).then(function () {
            self.$('.o_list_table').addClass('o_section_list_view');
        });
    },
});

// We create a custom widget because this is the cleanest way to do it:
// to be sure this custom code will only impact selected fields having the widget
// and not applied to any other existing ListRenderer.
var SectionFieldOne2Many = FieldOne2Many.extend({
    /**
     * We want to use our custom renderer for the list.
     *
     * @override
     */
    _getRenderer: function () {
        if (this.view.arch.tag === 'tree') {
            return SectionListRenderer;
        }
        return this._super.apply(this, arguments);
    },
});

fieldRegistry.add('section_one2many', SectionFieldOne2Many);

});

```

## File: views\resource.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="assets_backend" name="resource assets" inherit_id="web.assets_backend">
        <xpath expr="." position="inside">
            <link rel="stylesheet" href="/resource/static/src/scss/section_backend.scss"/>
            <script type="text/javascript" src="/resource/static/src/js/section_fields_backend.js"></script>
        </xpath>
    </template>
</odoo>

```

## File: views\resource_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- RESOURCE.RESOURCE -->
    <record id="view_resource_resource_search" model="ir.ui.view">
        <field name="name">resource.resource.search</field>
        <field name="model">resource.resource</field>
        <field name="arch" type="xml">
            <search string="Search Resource">
               <field name="name" string="Resource"/>
               <field name="resource_type"/>
               <field name="user_id"/>
               <field name="calendar_id"/>
               <field name="company_id" groups="base.group_multi_company"/>
               <filter string="Inactive" name="inactive" domain="[('active','=',False)]"/>
               <group expand="0" string="Group By">
                    <filter string="User" name="user" domain="[]" context="{'group_by':'user_id'}"/>
                    <filter string="Type" name="type" domain="[]" context="{'group_by':'resource_type'}"/>
                    <filter string="Company" name="company" domain="[]" context="{'group_by':'company_id'}" groups="base.group_multi_company"/>
                    <filter string="Working Period" name="working_period" domain="[]" context="{'group_by':'calendar_id'}"/>
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
                <group>
                    <group name="user_details">
                        <field name="name"/>
                        <field name="user_id" attrs="{'required':[('resource_type','=','user')], 'readonly':[('resource_type','=','material')]}"/>
                        <field name="company_id" options="{'no_create': True}" groups="base.group_multi_company"/>
                        <field name="active"/>
                    </group>
                    <group name="resource_details">
                        <field name="resource_type" />
                        <field name="calendar_id"/>
                        <field name="tz"/>
                        <field name="time_efficiency"/>
                    </group>
                </group>
            </sheet>
            </form>
        </field>
    </record>

    <record id="resource_resource_tree" model="ir.ui.view">
        <field name="name">resource.resource.tree</field>
        <field name="model">resource.resource</field>
        <field name="arch" type="xml">
            <tree string="Resources">
                <field name="name" />
                <field name="user_id" />
                <field name="company_id" groups="base.group_multi_company"/>
                <field name="resource_type" />
                <field name="calendar_id"/>
                <field name="time_efficiency"/>
            </tree>
        </field>
    </record>

    <record id="action_resource_resource_tree" model="ir.actions.act_window">
        <field name="name">Resources</field>
        <field name="res_model">resource.resource</field>
        <field name="view_mode">tree,form</field>
        <field name="context">{}</field>
        <field name="search_view_id" ref="view_resource_resource_search"/>
        <field name="help">Resources allow you to create and manage resources that should be involved in a specific project phase. You can also set their efficiency level and workload based on their weekly working hours.</field>
    </record>

    <record id="resource_resource_action_from_calendar" model="ir.actions.act_window">
        <field name="name">Resources</field>
        <field name="res_model">resource.resource</field>
        <field name="view_mode">tree,form</field>
        <field name="context">{
            'default_calendar_id': active_id,
            'search_default_calendar_id': active_id}</field>
        <field name="search_view_id" ref="view_resource_resource_search"/>
        <field name="help">Resources allow you to create and manage resources that should be involved in a specific project phase. You can also set their efficiency level and workload based on their weekly working hours.</field>
    </record>

    <!-- RESOURCE.CALENDAR.LEAVES -->
    <record id="view_resource_calendar_leaves_search" model="ir.ui.view">
        <field name="name">resource.calendar.leaves.search</field>
        <field name="model">resource.calendar.leaves</field>
        <field name="arch" type="xml">
            <search string="Search Working Period Time Off">
                <field name="name"/>
                <field name="resource_id"/>
                <field name="company_id" groups="base.group_multi_company"/>
                <field name="calendar_id"/>
                <group expand="0" string="Group By">
                    <filter string="Resource" name="resource" domain="[]" context="{'group_by':'resource_id'}"/>
                    <filter string="Company" name="company" domain="[]" context="{'group_by':'company_id'}" groups="base.group_multi_company"/>
                    <filter string="Leave Date" name="leave_month" domain="[]" context="{'group_by':'date_from'}" help="Starting Date of Leave"/>
                </group>
           </search>
        </field>
    </record>

    <record id="view_resource_calendar" model="ir.ui.view">
        <field name="name">resource.calendar.leaves.calendar</field>
        <field name="model">resource.calendar.leaves</field>
        <field name="arch" type="xml">
            <calendar date_start="date_from" date_stop="date_to" mode="month" string="Resource" color="resource_id" event_limit="5">
                <field name="resource_id" avatar_field="image_128"/>
                <field name="company_id"/>
                <field name="name"/>
            </calendar>
        </field>
    </record>

    <record id="resource_calendar_leave_form" model="ir.ui.view">
        <field name="name">resource.calendar.leaves.form</field>
        <field name="model">resource.calendar.leaves</field>
        <field name="arch" type="xml">
            <form string="Leave Detail">
            <sheet>
                <group>
                    <group name="leave_details">
                        <field name="name" string="Reason"/>
                        <field name="calendar_id"/>
                        <field name="company_id" options="{'no_create': True}" groups="base.group_multi_company" attrs="{'invisible':[('calendar_id','=',False)]}"/>
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
        <field name="name">resource.calendar.leaves.tree</field>
        <field name="model">resource.calendar.leaves</field>
        <field name="arch" type="xml">
            <tree string="Leave Detail">
                <field name="name" string="Reason"/>
                <field name="resource_id" />
                <field name="company_id" groups="base.group_multi_company"/>
                <field name="calendar_id" />
                <field name="date_from" />
                <field name="date_to" />
            </tree>
        </field>
    </record>

    <record id="action_resource_calendar_leave_tree" model="ir.actions.act_window">
        <field name="name">Resource Time Off</field>
        <field name="res_model">resource.calendar.leaves</field>
        <field name="view_mode">tree,form,calendar</field>
        <field name="search_view_id" ref="view_resource_calendar_leaves_search"/>
    </record>

    <record id="resource_calendar_leaves_action_from_calendar" model="ir.actions.act_window">
        <field name="name">Resource Time Off</field>
        <field name="res_model">resource.calendar.leaves</field>
        <field name="view_mode">tree,form,calendar</field>
        <field name="context">{
            'default_calendar_id': active_id,
            'search_default_calendar_id': active_id}</field>
        <field name="search_view_id" ref="view_resource_calendar_leaves_search"/>
    </record>

    <act_window
        id="resource_calendar_closing_days"
        name="Closing Days"
        res_model="resource.calendar.leaves"
        binding_model="resource.calendar"
        binding_views="form"
        view_mode="calendar,tree,form"
        context="{'default_calendar_id': active_id}"
        domain="[('calendar_id','=',active_id), ('resource_id','=',False)]"/>

    <act_window
        id="resource_calendar_resources_leaves"
        name="Resources Time Off"
        res_model="resource.calendar.leaves"
        binding_model="resource.calendar"
        binding_views="form"
        view_mode="calendar,tree,form"
        context="{'default_calendar_id': active_id}"
        domain="[('calendar_id','=',active_id), ('resource_id','!=',False)]"/>

    <!-- RESOURCE.CALENDAR.ATTENDANCE -->
    <record id="view_resource_calendar_attendance_tree" model="ir.ui.view">
        <field name="name">resource.calendar.attendance.tree</field>
        <field name="model">resource.calendar.attendance</field>
        <field name="arch" type="xml">
            <tree string="Working Time" editable="top">
                <field name="sequence" widget="handle"/>
                <field name="display_type" invisible="1"/>
                <field name="name"/>
                <field name="dayofweek"/>
                <field name="day_period"/>
                <field name="hour_from" widget="float_time"/>
                <field name="hour_to" widget="float_time"/>
                <field name="date_from"/>
                <field name="date_to"/>
                <field name="week_type" readonly="1" force_save="1" groups="base.group_no_one"/>
            </tree>
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
                </group>
                </sheet>
            </form>
        </field>
    </record>

    <!-- RESOURCE.CALENDAR -->
    <record id="view_resource_calendar_search" model="ir.ui.view">
        <field name="name">resource.calendar.search</field>
        <field name="model">resource.calendar</field>
        <field name="arch" type="xml">
            <search string="Search Working Time">
               <field name="name" string="Working Time"/>
               <field name="company_id" groups="base.group_multi_company"/>
           </search>
        </field>
    </record>

    <record id="resource_calendar_form" model="ir.ui.view">
        <field name="name">resource.calendar.form</field>
        <field name="model">resource.calendar</field>
        <field name="arch" type="xml">
            <form string="Working Time">
                <sheet string="Working Time">
                    <div class="oe_button_box" name="button_box">
                        <button name="%(resource_calendar_leaves_action_from_calendar)d" type="action"
                                string="Time Off" icon="fa-plane"
                                class="oe_stat_button"
                                groups="base.group_no_one"/>
                        <button name="%(resource_resource_action_from_calendar)d" type="action"
                                string="Work Resources" icon="fa-cogs"
                                class="oe_stat_button"
                                groups="base.group_user"/>
                    </div>
                    <h1>
                        <field name="name"/>
                    </h1>
                    <group name="resource_details">
                        <group>
                            <field name="company_id" groups="base.group_multi_company"/>
                            <field name="hours_per_day" widget="float_time"/>
                            <field name="tz"/>
                        </group>
                    </group>
                    <notebook>
                        <page string="Working Hours" name="working_hours">
                        <button name="switch_calendar_type" attrs="{'invisible':[('two_weeks_calendar', '=', True)]}" string="Switch to 2 weeks calendar" type="object" confirm="Are you sure you want to switch this calendar to 2 weeks calendar ? All entries will be lost"/>
                        <button name="switch_calendar_type" attrs="{'invisible':[('two_weeks_calendar', '=', False)]}" string="Switch to 1 week calendar" type="object" confirm="Are you sure you want to switch this calendar to 1 week calendar ? All entries will be lost"/>
                        <field name="two_weeks_calendar" invisible="1"/>

                        <group attrs="{'invisible':[('two_weeks_calendar', '=', False)]}">
                            <field name="two_weeks_explanation" nolabel="1"/>
                        </group>
                            <field name="attendance_ids" widget="section_one2many"/>
                        </page>
                        <page string="Global Time Off" name="global_leaves">
                            <field name="global_leave_ids">
                                <tree editable="top">
                                    <field name="name"/>
                                    <field name="date_from"/>
                                    <field name="date_to"/>
                                </tree>
                            </field>
                        </page>
                    </notebook>
                </sheet>
            </form>
        </field>
    </record>

    <record id="view_resource_calendar_tree" model="ir.ui.view">
        <field name="name">resource.calendar.tree</field>
        <field name="model">resource.calendar</field>
        <field name="arch" type="xml">
            <tree string="Working Time">
                <field name="name" string="Working Time"/>
                <field name="company_id" groups="base.group_multi_company"/>
            </tree>
        </field>
    </record>

    <record id="action_resource_calendar_form" model="ir.actions.act_window">
        <field name="name">Working Times</field>
        <field name="res_model">resource.calendar</field>
        <field name="view_mode">tree,form</field>
        <field name="view_id" eval="False"/>
        <field name="search_view_id" ref="view_resource_calendar_search"/>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Define working hours and time table that could be scheduled to your project members
          </p>
        </field>
    </record>

    <!-- MENU ITEMS -->
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

