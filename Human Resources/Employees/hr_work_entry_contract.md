# Odoo Module: hr_work_entry_contract

Category: Human Resources/Employees

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import wizard

```

## File: __manifest__.py

```python
#-*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Work Entries - Contract',
    'category': 'Human Resources/Employees',
    'sequence': 39,
    'summary': 'Manage work entries',
    'installable': True,
    'depends': [
        'hr_work_entry',
        'hr_contract',
    ],
    'data': [
        'security/hr_work_entry_security.xml',
        'security/ir.model.access.csv',
        'data/hr_work_entry_data.xml',
        'data/ir_cron_data.xml',
        'views/hr_work_entry_views.xml',
        'views/hr_contract_views.xml',
        'wizard/hr_work_entry_regeneration_wizard_views.xml',
    ],
    'demo': [
        'data/hr_work_entry_demo.xml',
    ],
    'assets': {
        'web.assets_backend': [
            'hr_work_entry_contract/static/src/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\hr_work_entry_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="0">

        <!-- Work Entry Type -->
         <record id="work_entry_type_leave" model="hr.work.entry.type">
            <field name="name">Generic Time Off</field>
            <field name="code">LEAVE100</field>
            <field name="color">3</field>
            <field name="is_leave">True</field>
        </record>

        <record id="work_entry_type_compensatory" model="hr.work.entry.type">
            <field name="name">Compensatory Time Off</field>
            <field name="code">LEAVE105</field>
            <field name="color">3</field>
            <field name="is_leave">True</field>
        </record>

        <record id="work_entry_type_home_working" model="hr.work.entry.type">
            <field name="name">Home Working</field>
            <field name="code">WORK110</field>
            <field name="color">2</field>
            <field name="is_leave">True</field>
        </record>

        <record id="work_entry_type_unpaid_leave" model="hr.work.entry.type">
            <field name="name">Unpaid</field>
            <field name="color">5</field>
            <field name="code">LEAVE90</field>
            <field name="is_leave">True</field>
        </record>

        <record id="work_entry_type_sick_leave" model="hr.work.entry.type">
            <field name="name">Sick Time Off</field>
            <field name="code">LEAVE110</field>
            <field name="is_leave">True</field>
            <field name="color">5</field>
        </record>

         <record id="work_entry_type_legal_leave" model="hr.work.entry.type">
            <field name="name">Paid Time Off</field>
            <field name="code">LEAVE120</field>
            <field name="is_leave">True</field>
            <field name="color">5</field>
        </record>
    </data>
    <data noupdate="1">
        <record id="hr_work_entry.work_entry_type_attendance" model="hr.work.entry.type">
            <field name="is_leave">False</field>
        </record>

    </data>
</odoo>

```

## File: data\hr_work_entry_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

        <!-- Work entries -->
        <record id="work_entry_type_extra_hours" model="hr.work.entry.type">
            <field name="name">Extra Hours</field>
            <field name="code">WORK300</field>
            <field name="color">8</field>
        </record>

        <record id="work_entry_type_long_leave" model="hr.work.entry.type">
            <field name="name">Long Term Time Off</field>
            <field name="code">LEAVE200</field>
            <field name="is_leave">True</field>
            <field name="color">4</field>
        </record>

</odoo>

```

## File: data\ir_cron_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="ir_cron_generate_missing_work_entries" model="ir.cron">
            <field name="name">Generate Missing Work Entries</field>
            <field name="model_id" ref="hr_contract.model_hr_contract"/>
            <field name="state">code</field>
            <field name="code">model._cron_generate_missing_work_entries()</field>
            <field name="active" eval="True"/>
            <field name="user_id" ref="base.user_root"/>
            <field name="interval_number">1</field>
            <field name="interval_type">days</field>
        </record>
    </data>
</odoo>

```

## File: models\hr_contract.py

```python
# -*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import itertools
from collections import defaultdict
from datetime import datetime, date, time
import pytz

from dateutil.relativedelta import relativedelta

from odoo import api, fields, models, _
from odoo.addons.resource.models.utils import string_to_datetime, Intervals
from odoo.osv import expression
from odoo.tools import ormcache, format_list
from odoo.exceptions import UserError

from .hr_work_intervals import WorkIntervals


class HrContract(models.Model):
    _inherit = 'hr.contract'
    _description = 'Employee Contract'

    date_generated_from = fields.Datetime(string='Generated From', readonly=True, required=True,
        default=lambda self: datetime.now().replace(hour=0, minute=0, second=0, microsecond=0), copy=False)
    date_generated_to = fields.Datetime(string='Generated To', readonly=True, required=True,
        default=lambda self: datetime.now().replace(hour=0, minute=0, second=0, microsecond=0), copy=False)
    last_generation_date = fields.Date(string='Last Generation Date', readonly=True)
    work_entry_source = fields.Selection([('calendar', 'Working Schedule')], required=True, default='calendar', help='''
        Defines the source for work entries generation

        Working Schedule: Work entries will be generated from the working hours below.
        Attendances: Work entries will be generated from the employee's attendances. (requires Attendance app)
        Planning: Work entries will be generated from the employee's planning. (requires Planning app)
    '''
    )
    work_entry_source_calendar_invalid = fields.Boolean(compute='_compute_work_entry_source_calendar_invalid')

    @api.depends('work_entry_source', 'resource_calendar_id')
    def _compute_work_entry_source_calendar_invalid(self):
        for contract in self:
            contract.work_entry_source_calendar_invalid = contract.work_entry_source == 'calendar' and not contract.resource_calendar_id

    @ormcache('self.structure_type_id')
    def _get_default_work_entry_type_id(self):
        attendance = self.env.ref('hr_work_entry.work_entry_type_attendance', raise_if_not_found=False)
        return attendance.id if attendance else False

    def _get_leave_work_entry_type_dates(self, leave, date_from, date_to, employee):
        return self._get_leave_work_entry_type(leave)

    def _get_leave_work_entry_type(self, leave):
        return leave.work_entry_type_id

    # Is used to add more values, for example planning_slot_id
    def _get_more_vals_attendance_interval(self, interval):
        return []

    # Is used to add more values, for example leave_id (in hr_work_entry_holidays)
    def _get_more_vals_leave_interval(self, interval, leaves):
        return []

    def _get_bypassing_work_entry_type_codes(self):
        return []

    def _get_interval_leave_work_entry_type(self, interval, leaves, bypassing_codes):
        # returns the work entry time related to the leave that
        # includes the whole interval.
        # Overriden in hr_work_entry_contract_holiday to select the
        # global time off first (eg: Public Holiday > Home Working)
        self.ensure_one()
        for leave in leaves:
            if interval[0] >= leave[0] and interval[1] <= leave[1] and leave[2]:
                interval_start = interval[0].astimezone(pytz.utc).replace(tzinfo=None)
                interval_stop = interval[1].astimezone(pytz.utc).replace(tzinfo=None)
                return self._get_leave_work_entry_type_dates(leave[2], interval_start, interval_stop, self.employee_id)
        return self.env.ref('hr_work_entry_contract.work_entry_type_leave')

    def _get_sub_leave_domain(self):
        return [('calendar_id', 'in', [False] + self.resource_calendar_id.ids)]

    def _get_leave_domain(self, start_dt, end_dt):
        domain = [
            ('resource_id', 'in', [False] + self.employee_id.resource_id.ids),
            ('date_from', '<=', end_dt),
            ('date_to', '>=', start_dt),
            ('company_id', 'in', [False, self.company_id.id]),
        ]
        return expression.AND([domain, self._get_sub_leave_domain()])

    def _get_resource_calendar_leaves(self, start_dt, end_dt):
        return self.env['resource.calendar.leaves'].search(self._get_leave_domain(start_dt, end_dt))

    def _get_attendance_intervals(self, start_dt, end_dt):
        # {resource: intervals}
        employees_by_calendar = defaultdict(lambda: self.env['hr.employee'])
        for contract in self:
            if contract.work_entry_source != 'calendar':
                continue
            employees_by_calendar[contract.resource_calendar_id] |= contract.employee_id
        result = dict()
        for calendar, employees in employees_by_calendar.items():
            result.update(calendar._attendance_intervals_batch(
                start_dt,
                end_dt,
                resources=employees.resource_id,
                tz=pytz.timezone(calendar.tz)
            ))
        return result

    def _get_lunch_intervals(self, start_dt, end_dt):
        # {resource: intervals}
        employees_by_calendar = defaultdict(lambda: self.env['hr.employee'])
        for contract in self:
            employees_by_calendar[contract.resource_calendar_id] |= contract.employee_id
        result = {}
        for calendar, employees in employees_by_calendar.items():
            result.update(calendar._attendance_intervals_batch(
                start_dt,
                end_dt,
                resources=employees.resource_id,
                tz=pytz.timezone(calendar.tz),
                lunch=True,
            ))
        return result

    def _get_interval_work_entry_type(self, interval):
        self.ensure_one()
        if 'work_entry_type_id' in interval[2] and interval[2].work_entry_type_id[:1]:
            return interval[2].work_entry_type_id[:1]
        return self.env['hr.work.entry.type'].browse(self._get_default_work_entry_type_id())

    def _get_valid_leave_intervals(self, attendances, interval):
        self.ensure_one()
        return [interval]

    def _get_contract_work_entries_values(self, date_start, date_stop):
        start_dt = pytz.utc.localize(date_start) if not date_start.tzinfo else date_start
        end_dt = pytz.utc.localize(date_stop) if not date_stop.tzinfo else date_stop
        contract_vals = []
        bypassing_work_entry_type_codes = self._get_bypassing_work_entry_type_codes()

        attendances_by_resource = self._get_attendance_intervals(start_dt, end_dt)

        resource_calendar_leaves = self._get_resource_calendar_leaves(start_dt, end_dt)
        # {resource: resource_calendar_leaves}
        leaves_by_resource = defaultdict(lambda: self.env['resource.calendar.leaves'])
        for leave in resource_calendar_leaves:
            leaves_by_resource[leave.resource_id.id] |= leave

        tz_dates = {}
        for contract in self:
            employee = contract.employee_id
            calendar = contract.resource_calendar_id
            resource = employee.resource_id
            # if the contract is fully flexible, we refer to the employee's timezone
            tz = pytz.timezone(resource.tz) if contract._is_fully_flexible() else pytz.timezone(calendar.tz)
            attendances = attendances_by_resource[resource.id]

            # Other calendars: In case the employee has declared time off in another calendar
            # Example: Take a time off, then a credit time.
            resources_list = [self.env['resource.resource'], resource]
            result = defaultdict(lambda: [])
            for leave in itertools.chain(leaves_by_resource[False], leaves_by_resource[resource.id]):
                for resource in resources_list:
                    # Global time off is not for this calendar, can happen with multiple calendars in self
                    if resource and leave.calendar_id and leave.calendar_id != calendar and not leave.resource_id:
                        continue
                    tz = tz if tz else pytz.timezone((resource or contract).tz)
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
                    leave_start_dt = max(start, dt0)
                    leave_end_dt = min(end, dt1)
                    leave_interval = (leave_start_dt, leave_end_dt, leave)
                    leave_interval = contract._get_valid_leave_intervals(attendances, leave_interval)
                    if leave_interval:
                        result[resource.id] += leave_interval
            mapped_leaves = {r.id: WorkIntervals(result[r.id]) for r in resources_list}
            leaves = mapped_leaves[resource.id]

            real_attendances = attendances - leaves
            if contract.has_static_work_entries() or not leaves:
                # Empty leaves means empty real_leaves
                real_leaves = attendances - real_attendances
            elif not calendar:
                # If fully flexible working schedule is defined
                real_leaves = leaves
            else:
                # In the case of attendance based contracts use regular attendances to generate leave intervals
                static_attendances = calendar._attendance_intervals_batch(
                    start_dt, end_dt, resources=resource, tz=tz)[resource.id]
                real_leaves = static_attendances & leaves

            if not contract.has_static_work_entries():
                # An attendance based contract might have an invalid planning, by definition it may not happen with
                # static work entries.
                # Creating overlapping slots for example might lead to a single work entry.
                # In that case we still create both work entries to indicate a problem (conflicting W E).
                split_attendances = []
                for attendance in real_attendances:
                    if attendance[2] and len(attendance[2]) > 1:
                        split_attendances += [(attendance[0], attendance[1], a) for a in attendance[2]]
                    else:
                        split_attendances += [attendance]
                real_attendances = split_attendances

            # A leave period can be linked to several resource.calendar.leave
            split_leaves = []
            for leave_interval in leaves:
                if leave_interval[2] and len(leave_interval[2]) > 1:
                    split_leaves += [(leave_interval[0], leave_interval[1], l) for l in leave_interval[2]]
                else:
                    split_leaves += [(leave_interval[0], leave_interval[1], leave_interval[2])]
            leaves = split_leaves

            # Attendances
            for interval in real_attendances:
                work_entry_type = contract._get_interval_work_entry_type(interval)
                # All benefits generated here are using datetimes converted from the employee's timezone
                contract_vals += [dict([
                    ('name', "%s: %s" % (work_entry_type.name, employee.name)),
                    ('date_start', interval[0].astimezone(pytz.utc).replace(tzinfo=None)),
                    ('date_stop', interval[1].astimezone(pytz.utc).replace(tzinfo=None)),
                    ('work_entry_type_id', work_entry_type.id),
                    ('employee_id', employee.id),
                    ('contract_id', contract.id),
                    ('company_id', contract.company_id.id),
                    ('state', 'draft'),
                ] + contract._get_more_vals_attendance_interval(interval))]

            leaves_over_attendances = WorkIntervals(leaves) & real_leaves
            for interval in real_leaves:
                # Could happen when a leave is configured on the interface on a day for which the
                # employee is not supposed to work, i.e. no attendance_ids on the calendar.
                # In that case, do try to generate an empty work entry, as this would raise a
                # sql constraint error
                if interval[0] == interval[1]:  # if start == stop
                    continue
                leaves_over_interval = [l for l in leaves_over_attendances if l[0] >= interval[0] and l[1] <= interval[1]]
                for leave_interval in [(l[0], l[1], interval[2]) for l in leaves_over_interval]:
                    leave_entry_type = contract._get_interval_leave_work_entry_type(leave_interval, leaves, bypassing_work_entry_type_codes)
                    interval_leaves = [leave for leave in leaves if leave[2].work_entry_type_id.id == leave_entry_type.id]
                    interval_start = leave_interval[0].astimezone(pytz.utc).replace(tzinfo=None)
                    interval_stop = leave_interval[1].astimezone(pytz.utc).replace(tzinfo=None)
                    contract_vals += [dict([
                        ('name', "%s%s" % (leave_entry_type.name + ": " if leave_entry_type else "", employee.name)),
                        ('date_start', interval_start),
                        ('date_stop', interval_stop),
                        ('work_entry_type_id', leave_entry_type.id),
                        ('employee_id', employee.id),
                        ('company_id', contract.company_id.id),
                        ('state', 'draft'),
                        ('contract_id', contract.id),
                    ] + contract._get_more_vals_leave_interval(interval, interval_leaves))]
        return contract_vals

    def _get_work_entries_values(self, date_start, date_stop):
        """
        Generate a work_entries list between date_start and date_stop for one contract.
        :return: list of dictionnary.
        """
        if isinstance(date_start, datetime):
            contract_vals = self._get_contract_work_entries_values(date_start, date_stop)
        else:
            contract_vals = []
            contracts_by_tz = defaultdict(lambda: self.env['hr.contract'])
            for contract in self:
                contracts_by_tz[contract.resource_calendar_id.tz] += contract
            for contract_tz, contracts in contracts_by_tz.items():
                tz = pytz.timezone(contract_tz) if contract_tz else pytz.utc
                contract_vals += contracts._get_contract_work_entries_values(
                    tz.localize(date_start),
                    tz.localize(date_stop))

        # {contract_id: ([dates_start], [dates_stop])}
        mapped_contract_dates = defaultdict(lambda: ([], []))
        for x in contract_vals:
            mapped_contract_dates[x['contract_id']][0].append(x['date_start'])
            mapped_contract_dates[x['contract_id']][1].append(x['date_stop'])

        for contract in self:
            # If we generate work_entries which exceeds date_start or date_stop, we change boundaries on contract
            if contract_vals:
                #Handle empty work entries for certain contracts, could happen on an attendance based contract
                #NOTE: this does not handle date_stop or date_start not being present in vals
                dates_stop = mapped_contract_dates[contract.id][1]
                if dates_stop:
                    date_stop_max = max(dates_stop)
                    if date_stop_max > contract.date_generated_to:
                        contract.date_generated_to = date_stop_max

                dates_start = mapped_contract_dates[contract.id][0]
                if dates_start:
                    date_start_min = min(dates_start)
                    if date_start_min < contract.date_generated_from:
                        contract.date_generated_from = date_start_min

        return contract_vals

    def has_static_work_entries(self):
        # Static work entries as in the same are to be generated each month
        # Useful to differentiate attendance based contracts from regular ones
        self.ensure_one()
        return self.work_entry_source == 'calendar'

    def generate_work_entries(self, date_start, date_stop, force=False):
        # Generate work entries between 2 dates (datetime.date)
        # To correctly englobe the period, the start and end periods are converted
        # using the calendar timezone.
        assert not isinstance(date_start, datetime)
        assert not isinstance(date_stop, datetime)

        date_start = datetime.combine(fields.Datetime.to_datetime(date_start), datetime.min.time())
        date_stop = datetime.combine(fields.Datetime.to_datetime(date_stop), datetime.max.time())

        contracts_by_company_tz = defaultdict(lambda: self.env['hr.contract'])
        for contract in self:
            contracts_by_company_tz[(
                contract.company_id,
                (contract.resource_calendar_id or contract.employee_id.resource_calendar_id).tz
            )] += contract
        utc = pytz.timezone('UTC')
        new_work_entries = self.env['hr.work.entry']
        for (company, contract_tz), contracts in contracts_by_company_tz.items():
            tz = pytz.timezone(contract_tz) if contract_tz else pytz.utc
            date_start_tz = tz.localize(date_start).astimezone(utc).replace(tzinfo=None)
            date_stop_tz = tz.localize(date_stop).astimezone(utc).replace(tzinfo=None)
            new_work_entries += contracts.with_company(company).sudo()._generate_work_entries(
                date_start_tz, date_stop_tz, force=force)
        return new_work_entries

    def _generate_work_entries(self, date_start, date_stop, force=False):
        # Generate work entries between 2 dates (datetime.datetime)
        # This method considers that the dates are correctly localized
        # based on the target timezone
        assert isinstance(date_start, datetime)
        assert isinstance(date_stop, datetime)
        self = self.with_context(tracking_disable=True)
        canceled_contracts = self.filtered(lambda c: c.state == 'cancel')
        if canceled_contracts:
            raise UserError(
                _("Sorry, generating work entries from cancelled contracts is not allowed.")
                + "\n%s" % (format_list(self.env, canceled_contracts.mapped("name"))),
            )
        vals_list = []
        self.write({'last_generation_date': fields.Date.today()})

        intervals_to_generate = defaultdict(lambda: self.env['hr.contract'])
        # In case the date_generated_from == date_generated_to, move it to the date_start to
        # avoid trying to generate several months/years of history for old contracts for which
        # we've never generated the work entries.
        self.filtered(lambda c: c.date_generated_from == c.date_generated_to).write({
            'date_generated_from': date_start,
            'date_generated_to': date_start,
        })
        utc = pytz.timezone('UTC')
        for contract in self:
            contract_tz = (contract.resource_calendar_id or contract.employee_id.resource_calendar_id).tz
            tz = pytz.timezone(contract_tz) if contract_tz else pytz.utc
            contract_start = tz.localize(fields.Datetime.to_datetime(contract.date_start)).astimezone(utc).replace(tzinfo=None)
            contract_stop = datetime.combine(fields.Datetime.to_datetime(contract.date_end or datetime.max.date()),
                                             datetime.max.time())
            if contract.date_end:
                contract_stop = tz.localize(contract_stop).astimezone(utc).replace(tzinfo=None)
            if date_start > contract_stop or date_stop < contract_start:
                continue
            date_start_work_entries = max(date_start, contract_start)
            date_stop_work_entries = min(date_stop, contract_stop)
            if force:
                intervals_to_generate[(date_start_work_entries, date_stop_work_entries)] |= contract
                continue

            # For each contract, we found each interval we must generate
            # In some cases we do not want to set the generated dates beforehand, since attendance based work entries
            #  is more dynamic, we want to update the dates within the _get_work_entries_values function
            last_generated_from = min(contract.date_generated_from, contract_stop)
            if last_generated_from > date_start_work_entries:
                contract.date_generated_from = date_start_work_entries
                intervals_to_generate[(date_start_work_entries, last_generated_from)] |= contract

            last_generated_to = max(contract.date_generated_to, contract_start)
            if last_generated_to < date_stop_work_entries:
                contract.date_generated_to = date_stop_work_entries
                intervals_to_generate[(last_generated_to, date_stop_work_entries)] |= contract

        for interval, contracts in intervals_to_generate.items():
            date_from, date_to = interval
            vals_list.extend(contracts._get_work_entries_values(date_from, date_to))

        if not vals_list:
            return self.env['hr.work.entry']

        return self.env['hr.work.entry'].create(vals_list)

    def _remove_work_entries(self):
        ''' Remove all work_entries that are outside contract period (function used after writing new start or/and end date) '''
        all_we_to_unlink = self.env['hr.work.entry']
        for contract in self:
            date_start = fields.Datetime.to_datetime(contract.date_start)
            if contract.date_generated_from < date_start:
                we_to_remove = self.env['hr.work.entry'].search([('date_stop', '<=', date_start), ('contract_id', '=', contract.id)])
                if we_to_remove:
                    contract.date_generated_from = date_start
                    all_we_to_unlink |= we_to_remove
            if not contract.date_end:
                continue
            date_end = datetime.combine(contract.date_end, datetime.max.time())
            if contract.date_generated_to > date_end:
                we_to_remove = self.env['hr.work.entry'].search([('date_start', '>=', date_end), ('contract_id', '=', contract.id)])
                if we_to_remove:
                    contract.date_generated_to = date_end
                    all_we_to_unlink |= we_to_remove
        all_we_to_unlink.unlink()

    def _cancel_work_entries(self):
        if not self:
            return
        domain = [('state', '!=', 'validated')]
        for contract in self:
            date_start = fields.Datetime.to_datetime(contract.date_start)
            contract_domain = [
                ('contract_id', '=', contract.id),
                ('date_start', '>=', date_start),
            ]
            if contract.date_end:
                date_end = datetime.combine(contract.date_end, datetime.max.time())
                contract_domain += [('date_stop', '<=', date_end)]
            domain = expression.AND([domain, contract_domain])
        work_entries = self.env['hr.work.entry'].sudo().search(domain)
        if work_entries:
            work_entries.sudo().unlink()

    def write(self, vals):
        result = super(HrContract, self).write(vals)
        if vals.get('date_end') or vals.get('date_start'):
            self.sudo()._remove_work_entries()
        if vals.get('state') in ['draft', 'cancel']:
            self._cancel_work_entries()
        dependendant_fields = self._get_fields_that_recompute_we()
        salary_simulation = self.env.context.get('salary_simulation')
        if not salary_simulation and any(key in dependendant_fields for key in vals.keys()):
            for contract in self:
                date_from = max(contract.date_start, contract.date_generated_from.date())
                date_to = min(contract.date_end or date.max, contract.date_generated_to.date())
                if date_from != date_to and self.employee_id:
                    contract._recompute_work_entries(date_from, date_to)
        return result

    def _recompute_work_entries(self, date_from, date_to):
        self.ensure_one()
        if self.employee_id:
            wizard = self.env['hr.work.entry.regeneration.wizard'].create({
                'employee_ids': [(4, self.employee_id.id)],
                'date_from': date_from,
                'date_to': date_to,
            })
            wizard.with_context(work_entry_skip_validation=True).regenerate_work_entries()

    def _get_fields_that_recompute_we(self):
        # Returns the fields that should recompute the work entries
        return ['resource_calendar_id', 'work_entry_source']

    @api.model
    def _cron_generate_missing_work_entries(self):
        # retrieve contracts for the current month
        today = fields.Date.today()
        start = datetime.combine(today + relativedelta(day=1), time.min)
        stop = datetime.combine(today + relativedelta(months=1, day=31), time.max)
        all_contracts = self.env['hr.employee']._get_all_contracts(
            start, stop, states=['open', 'close'])
        # determine contracts to do (the ones whose generated dates have open periods this month)
        contracts_todo = all_contracts.filtered(lambda c:\
            (c.date_generated_from > start or c.date_generated_to < stop) and\
            (not c.last_generation_date or c.last_generation_date < today))
        if not contracts_todo:
            return
        countract_todo_count = len(contracts_todo)
        # Filter contracts by company, work entries generation is not supposed to be called on
        # contracts from differents companies, as we will retrieve the resource.calendar.leave
        # and we don't want to mix everything up. The other contracts will be treated when the
        # cron is re-triggered
        contracts_todo = contracts_todo.filtered(lambda c: c.company_id == contracts_todo[0].company_id)
        # generate a batch of work entries
        BATCH_SIZE = 100
        # Since attendance based are more volatile for their work entries generation
        # it can happen that the date_generated_from and date_generated_to fields are not
        # pushed to start and stop
        # It is more interesting for batching to process statically generated work entries first
        # since we get benefits from having multiple contracts on the same calendar
        contracts_todo = contracts_todo.sorted(key=lambda c: 1 if c.has_static_work_entries() else 100)
        contracts_todo = contracts_todo[:BATCH_SIZE].generate_work_entries(
            start.date(), stop.date(), False)
        # if necessary, retrigger the cron to generate more work entries
        if countract_todo_count > BATCH_SIZE:
            self.env.ref('hr_work_entry_contract.ir_cron_generate_missing_work_entries')._trigger()

```

## File: models\hr_employee.py

```python
# -*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class HrEmployee(models.Model):
    _inherit = 'hr.employee'

    def generate_work_entries(self, date_start, date_stop, force=False):
        date_start = fields.Date.to_date(date_start)
        date_stop = fields.Date.to_date(date_stop)

        if self:
            current_contracts = self._get_contracts(date_start, date_stop, states=['open', 'close'])
        else:
            current_contracts = self._get_all_contracts(date_start, date_stop, states=['open', 'close'])

        return current_contracts.generate_work_entries(date_start, date_stop, force=force)

```

## File: models\hr_work_entry.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import pytz

from collections import defaultdict
from itertools import chain

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError
from odoo.addons.hr_work_entry_contract.models.hr_work_intervals import WorkIntervals


class HrWorkEntry(models.Model):
    _inherit = 'hr.work.entry'

    contract_id = fields.Many2one('hr.contract', string="Contract", required=True)
    employee_id = fields.Many2one(domain=[('contract_ids.state', 'in', ('open', 'pending'))])
    work_entry_source = fields.Selection(related='contract_id.work_entry_source')

    def init(self):
        # FROM 7s by query to 2ms (with 2.6 millions entries)
        self.env.cr.execute("""
            CREATE INDEX IF NOT EXISTS hr_work_entry_contract_date_start_stop_idx
            ON hr_work_entry(contract_id, date_start, date_stop)
            WHERE state in ('draft', 'validated');
        """)

    def _init_column(self, column_name):
        if column_name != 'contract_id':
            super()._init_column(column_name)
        else:
            self.env.cr.execute("""
                UPDATE hr_work_entry AS _hwe
                SET contract_id = result.contract_id
                FROM (
                    SELECT
                        hc.id AS contract_id,
                        array_agg(hwe.id) AS entry_ids
                    FROM
                        hr_work_entry AS hwe
                    LEFT JOIN
                        hr_contract AS hc
                    ON
                        hwe.employee_id=hc.employee_id AND
                        hc.state in ('open', 'close') AND
                        hwe.date_start >= hc.date_start AND
                        hwe.date_stop < COALESCE(hc.date_end + integer '1', '9999-12-31 23:59:59')
                    WHERE
                        hwe.contract_id IS NULL
                    GROUP BY
                        hwe.employee_id, hc.id
                ) AS result
                WHERE _hwe.id = ANY(result.entry_ids)
            """)

    def _get_duration_is_valid(self):
        return self.work_entry_type_id and self.work_entry_type_id.is_leave

    @api.onchange('employee_id', 'date_start', 'date_stop')
    def _onchange_contract_id(self):
        vals = {
            'employee_id': self.employee_id.id,
            'date_start': self.date_start,
            'date_stop': self.date_stop,
        }
        try:
            res = self._set_current_contract(vals)
        except ValidationError:
            return
        if res.get('contract_id'):
            self.contract_id = res.get('contract_id')

    @api.depends('date_start', 'duration')
    def _compute_date_stop(self):
        for work_entry in self:
            if work_entry._get_duration_is_valid():
                calendar = work_entry.contract_id.resource_calendar_id
                if not calendar:
                    continue
                work_entry.date_stop = calendar.plan_hours(work_entry.duration, work_entry.date_start, compute_leaves=True)
                continue
            super(HrWorkEntry, work_entry)._compute_date_stop()

    def _is_duration_computed_from_calendar(self):
        self.ensure_one()
        return self._get_duration_is_valid()

    def _get_duration_batch(self):
        super_work_entries = self.env['hr.work.entry']
        result = {}
        # {(date_start, date_stop): {calendar: employees}}
        mapped_periods = defaultdict(lambda: defaultdict(lambda: self.env['hr.employee']))
        for work_entry in self:
            if not work_entry.date_start or not work_entry.date_stop or not work_entry._is_duration_computed_from_calendar() or not work_entry.employee_id:
                super_work_entries |= work_entry
                continue
            date_start = work_entry.date_start
            date_stop = work_entry.date_stop
            calendar = work_entry.contract_id.resource_calendar_id
            if not calendar:
                result[work_entry.id] = 0.0
                continue
            employee = work_entry.contract_id.employee_id
            mapped_periods[(date_start, date_stop)][calendar] |= employee

        # {(date_start, date_stop): {calendar: {'hours': foo}}}
        mapped_contract_data = defaultdict(lambda: defaultdict(lambda: {'hours': 0.0}))
        for (date_start, date_stop), employees_by_calendar in mapped_periods.items():
            for calendar, employees in employees_by_calendar.items():
                mapped_contract_data[(date_start, date_stop)][calendar] = employees._get_work_days_data_batch(
                    date_start, date_stop, compute_leaves=False, calendar=calendar)
        result = super(HrWorkEntry, super_work_entries)._get_duration_batch()
        for work_entry in self - super_work_entries:
            date_start = work_entry.date_start
            date_stop = work_entry.date_stop
            calendar = work_entry.contract_id.resource_calendar_id
            employee = work_entry.contract_id.employee_id
            result[work_entry.id] = mapped_contract_data[(date_start, date_stop)][calendar][employee.id]['hours'] if calendar else 0.0
        return result

    @api.model
    def _set_current_contract(self, vals):
        if not vals.get('contract_id') and vals.get('date_start') and vals.get('date_stop') and vals.get('employee_id'):
            contract_start = fields.Datetime.to_datetime(vals.get('date_start')).date()
            contract_end = fields.Datetime.to_datetime(vals.get('date_stop')).date()
            employee = self.env['hr.employee'].browse(vals.get('employee_id'))
            contracts = employee._get_contracts(contract_start, contract_end, states=['open', 'pending', 'close'])
            if not contracts:
                raise ValidationError(_(
                    "%(employee)s does not have a contract from %(date_start)s to %(date_end)s.",
                    employee=employee.name,
                    date_start=contract_start,
                    date_end=contract_end,
                ))
            elif len(contracts) > 1:
                raise ValidationError(_("%(employee)s has multiple contracts from %(date_start)s to %(date_end)s. A work entry cannot overlap multiple contracts.",
                                        employee=employee.name, date_start=contract_start, date_end=contract_end))
            return dict(vals, contract_id=contracts[0].id)
        return vals

    @api.model_create_multi
    def create(self, vals_list):
        vals_list = [self._set_current_contract(vals) for vals in vals_list]
        work_entries = super().create(vals_list)
        return work_entries

    def _check_if_error(self):
        res = super()._check_if_error()
        outside_calendar = self._mark_leaves_outside_schedule()
        return res or outside_calendar

    def _get_leaves_entries_outside_schedule(self):
        return self.filtered(lambda w: w.work_entry_type_id.is_leave and w.state not in ('validated', 'cancelled'))

    def _mark_leaves_outside_schedule(self):
        """
        Check leave work entries in `self` which are completely outside
        the contract's theoretical calendar schedule. Mark them as conflicting.
        :return: leave work entries completely outside the contract's calendar
        """
        work_entries = self._get_leaves_entries_outside_schedule()
        entries_by_calendar = defaultdict(lambda: self.env['hr.work.entry'])
        for work_entry in work_entries:
            calendar = work_entry.contract_id.resource_calendar_id
            entries_by_calendar[calendar] |= work_entry

        outside_entries = self.env['hr.work.entry']
        for calendar, entries in entries_by_calendar.items():
            datetime_start = min(entries.mapped('date_start'))
            datetime_stop = max(entries.mapped('date_stop'))

            calendar_intervals = calendar._attendance_intervals_batch(pytz.utc.localize(datetime_start), pytz.utc.localize(datetime_stop))[False]
            entries_intervals = entries._to_intervals()
            overlapping_entries = self._from_intervals(entries_intervals & calendar_intervals)
            outside_entries |= entries - overlapping_entries
        outside_entries.write({'state': 'conflict'})
        return bool(outside_entries)

    def _to_intervals(self):
        return WorkIntervals((w.date_start.replace(tzinfo=pytz.utc), w.date_stop.replace(tzinfo=pytz.utc), w) for w in self)

    @api.model
    def _from_intervals(self, intervals):
        return self.browse(chain.from_iterable(recs.ids for start, end, recs in intervals))


class HrWorkEntryType(models.Model):
    _inherit = 'hr.work.entry.type'
    _description = 'HR Work Entry Type'

    is_leave = fields.Boolean(
        default=False, string="Time Off", help="Allow the work entry type to be linked with time off types.")

```

## File: models\hr_work_intervals.py

```python
# -*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from itertools import chain


def _boundaries(intervals, opening, closing):
    """ Iterate on the boundaries of intervals. """
    for start, stop, recs in intervals:
        if start < stop:
            yield (start, opening, recs)
            yield (stop, closing, recs)


class WorkIntervals(object):
    """
        This class is a modified copy of the ``Intervals`` class in the resource module.
        A generic solution to handle intervals should probably be developped the day a similar
        class is needed elsewhere.

        This implementation differs from the resource implementation in its management
        of two continuous intervals. Here, continuous intervals are not merged together
        while they are merged in resource.
        e.g.:
        In resource: (1, 4, rec1) and (4, 10, rec2) are merged into (1, 10, rec1 | rec2)
        Here: they remain two different intervals.
        To implement this behaviour, the main implementation change is the way boundaries are sorted.
    """
    def __init__(self, intervals=()):
        self._items = []
        if intervals:
            # normalize the representation of intervals
            append = self._items.append
            starts = []
            recses = []
            for value, flag, recs in sorted(_boundaries(sorted(intervals), 'start', 'stop'), key=lambda i: i[0]):
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
        return WorkIntervals(chain(self._items, other._items))

    def __and__(self, other):
        """ Return the intersection of two sets of intervals. """
        return self._merge(other, False)

    def __sub__(self, other):
        """ Return the difference of two sets of intervals. """
        return self._merge(other, True)

    def _merge(self, other, difference):
        """ Return the difference or intersection of two sets of intervals. """
        result = WorkIntervals()
        append = result._items.append

        # using 'self' and 'other' below forces normalization
        bounds1 = _boundaries(sorted(self), 'start', 'stop')
        bounds2 = _boundaries(sorted(other), 'switch', 'switch')

        start = None                    # set by start/stop
        recs1 = None                    # set by start
        enabled = difference            # changed by switch
        for value, flag, recs in sorted(chain(bounds1, bounds2), key=lambda i: i[0]):
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

```

## File: models\resource_calendar.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models

class ResourceCalendar(models.Model):
    _inherit = 'resource.calendar'

    def _get_global_attendances(self):
        return super()._get_global_attendances().filtered(lambda a: not a.work_entry_type_id.is_leave)

```

## File: models\__init__.py

```python
# -*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_contract
from . import hr_employee
from . import hr_work_entry
from . import hr_work_intervals
from . import resource_calendar

```

## File: security\hr_work_entry_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="ir_rule_hr_work_entry_multi_company" model="ir.rule">
        <field name="name">HR Work Entry Contract: Multi Company</field>
        <field name="model_id" ref="model_hr_work_entry"/>
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>

</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_hr_work_entry_regeneration_wizard,access_hr_work_entry_regeneration_wizard,model_hr_work_entry_regeneration_wizard,hr.group_hr_manager,1,1,1,1
```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M0 12.5h50V38a4 4 0 0 1-4 4H0V12.5Z" fill="#985184"/><path d="M4 20.5a4 4 0 0 1-4-4V12a4 4 0 0 1 4-4h46v8.5a4 4 0 0 1-4 4H4Z" fill="#2EBCFA"/><path d="M0 16h50v4a4 4 0 0 1-4 4H4a4 4 0 0 1-4-4v-4Z" fill="#144496"/><path d="M25 28a9 9 0 0 0-9 9h10a9 9 0 0 0 9-9H25Z" fill="#fff"/><circle cx="25" cy="19" r="6" fill="#fff"/></svg>

```

## File: static\src\components\work_entry_source_field\work_entry_source_field.js

```javascript
import { registry } from "@web/core/registry";
import { RadioField, radioField } from "@web/views/fields/radio/radio_field";
import { _t } from "@web/core/l10n/translation";

export class WorkEntrySourceField extends RadioField {
    static template = "hr_work_entry_contract.WorkEntrySourceField";

    get isFullyFlexible() {
        return !this.props.record.data.resource_calendar_id;
    }

    get tooltipWarning() {
        return JSON.stringify({
            "text" : _t("Invalid option: For fully flexible calendars, the work entry source cannot be 'Working Hours'."),
        })
    }
}

export const workEntrySourceField = {
    ...radioField,
    component: WorkEntrySourceField,
    fieldDependencies: [
        { name: "resource_calendar_id", type: "many2one", relation: "resource.calendar" },
    ],
};

registry.category("fields").add("work_entry_source_field", workEntrySourceField);

```

## File: static\src\components\work_entry_source_field\work_entry_source_field.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="hr_work_entry_contract.WorkEntrySourceField" t-inherit="web.RadioField">
        <xpath expr="//input[hasclass('o_radio_input')]" position="attributes">
            <attribute name="t-att-disabled">props.readonly or (item[0] === 'calendar' and isFullyFlexible)</attribute>
        </xpath>

        <xpath expr="//div[hasclass('o_radio_item')]" position="inside">
            <span t-if="props.readonly or (item[0] === 'calendar' and item[0] === value and isFullyFlexible)"
                class="fa fa-exclamation-triangle text-danger o_work_entry_source_warning ms-3"
                data-tooltip-template="hr_work_entry_contract.WorkEntrySourceWarning"
                t-att-data-tooltip-info='tooltipWarning'/>
        </xpath>

    </t>
</templates>

```

## File: static\src\views\work_entry_hook.js

```javascript
/** @odoo-module **/

import { serializeDate } from "@web/core/l10n/dates";
import { useService } from "@web/core/utils/hooks";

export function useWorkEntry({ getEmployeeIds, getRange, onClose}) {
    const orm = useService("orm");
    const action = useService("action");

    return {
        onRegenerateWorkEntries: () => {
            const { start, end } = getRange();
            action.doAction('hr_work_entry_contract.hr_work_entry_regeneration_wizard_action', {
                additionalContext: {
                    default_employee_ids: getEmployeeIds(),
                    date_start: serializeDate(start),
                    date_end: serializeDate(end),
                }, onClose: onClose
            });
        },
        generateWorkEntries: () => {
            const { start, end } = getRange();
            return orm.call(
                'hr.employee',
                'generate_work_entries',
                [[], serializeDate(start), serializeDate(end)]
            );
        },
    }
}

```

## File: static\src\views\work_entry_calendar\work_entry_calendar.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates>
    <t t-name="hr_work_entry_contract.calendar.controlButtons" t-inherit="web.CalendarController.controlButtons" t-inherit-mode="primary">
        <xpath expr="." position="inside">
            <span class="btn-regenerate-work-entries">
                <button class="btn btn-secondary ms-3" t-on-click="onRegenerateWorkEntries">
                    Regenerate Work Entries
                </button>
            </span>
        </xpath>
    </t>
</templates>

```

## File: static\src\views\work_entry_calendar\work_entry_calendar_controller.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { CalendarController } from '@web/views/calendar/calendar_controller';
import { WorkEntryCalendarModel } from "@hr_work_entry_contract/views/work_entry_calendar/work_entry_calendar_model";
import { calendarView } from "@web/views/calendar/calendar_view";
import { useWorkEntry } from "@hr_work_entry_contract/views/work_entry_hook";

export class WorkEntryCalendarController extends CalendarController {
    setup() {
        super.setup(...arguments);
        const { onRegenerateWorkEntries } = useWorkEntry({
            getEmployeeIds: this.getEmployeeIds.bind(this),
            getRange: this.model.computeRange.bind(this.model),
            onClose: this.model.load.bind(this.model)
        });
        this.onRegenerateWorkEntries = onRegenerateWorkEntries;
    }

    getEmployeeIds() {
        return [...new Set(Object.values(this.model.records).map(rec => rec.rawRecord.employee_id[0]))];
    }
}

export const WorkEntryCalendarView = {
    ...calendarView,
    Controller: WorkEntryCalendarController,
    Model: WorkEntryCalendarModel,
    buttonTemplate: "hr_work_entry_contract.calendar.controlButtons",
}

registry.category("views").add("work_entries_calendar", WorkEntryCalendarView);

```

## File: static\src\views\work_entry_calendar\work_entry_calendar_model.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { CalendarModel } from "@web/views/calendar/calendar_model";
import { useWorkEntry } from "@hr_work_entry_contract/views/work_entry_hook";

const { DateTime } = luxon;

export class WorkEntryCalendarModel extends CalendarModel {
    setup() {
        super.setup(...arguments);
        const { generateWorkEntries } = useWorkEntry({ getRange: this.computeRange.bind(this) });
        this.generateWorkEntries = generateWorkEntries;
    }

    computeRange() {
        const { scale, date } = this.meta;
        const start = date.startOf(scale);
        const end = date.endOf(scale);
        return { start, end };
    }

    makeFilterAll() {
        return {
            ...super.makeFilterAll(...arguments),
            label: _t("Everybody's work entries"),
            active: true,
        };
    }

    async updateData(data) {
        const { start } = this.computeRange();
        const shouldGenerateWorkEntries = start <= DateTime.now();
        if (shouldGenerateWorkEntries) {
            await this.generateWorkEntries();
        }
        await super.updateData(data);
    }
}

```

## File: static\src\xml\hr_work_entry_source_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="hr_work_entry_contract.WorkEntrySourceWarning">
        <div class="o-tooltip px-1 py-2">
            <p style="font-size: 12px;" class="text-danger"
                t-esc="text"/>
        </div>
    </t>
</templates>

```

## File: views\hr_contract_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hr_contract_view_form_inherit_work_entry" model="ir.ui.view">
        <field name="name">hr.contract.view.form.inherit.work.entry</field>
        <field name="model">hr.contract</field>
        <field name="inherit_id" ref="hr_contract.hr_contract_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='resource_calendar_warning']" position="replace">
                <div id="resource_calendar_warning" class="d-flex align-items-center">
                    <!-- resource calendar is only required when work_entry_source is set to 'calendar' -->
                    <field name="resource_calendar_id"
                        groups="!hr_contract.group_hr_contract_manager"
                        placeholder="Fully Flexible"
                        required="work_entry_source == 'calendar'"
                        options="{'no_open': True, 'no_create': True}"/>
                    <field name="resource_calendar_id"
                        groups="hr_contract.group_hr_contract_manager"
                        placeholder="Fully Flexible"
                        required="work_entry_source == 'calendar'"/>
                    <widget
                        name="contract_warning_tooltip"
                        invisible="not calendar_mismatch or state != 'open'"/>
                </div>
                <!-- Makes no sense to display only one possibility -->
                <field name="work_entry_source" widget="work_entry_source_field" invisible="1"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\hr_work_entry_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hr_work_entry_contract_view_calendar_inherit" model="ir.ui.view">
        <field name="name">hr.work.entry.contract.view.calendar.inherit</field>
        <field name="model">hr.work.entry</field>
        <field name="inherit_id" ref="hr_work_entry.hr_work_entry_view_calendar"/>
        <field name="arch" type="xml">
            <xpath expr="//calendar" position="attributes">
                <attribute name="js_class">work_entries_calendar</attribute>
            </xpath>
        </field>
    </record>

    <record id="hr_work_entry_contract_view_form_inherit" model="ir.ui.view">
        <field name="name">hr.work.entry.contract.view.form.inherit</field>
        <field name="model">hr.work.entry</field>
        <field name="inherit_id" ref="hr_work_entry.hr_work_entry_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//sheet" position="before">
                <div invisible="state != 'conflict'">
                    <div class="alert alert-warning" role="alert" invisible="work_entry_type_id" name="work_entry_undefined">
                        This work entry cannot be validated. The work entry type is undefined.
                    </div>
                </div>
            </xpath>
        </field>
    </record>

    <record id="hr_work_entry_contract_type_view_form_inherit" model="ir.ui.view">
        <field name="name">hr.work.entry.type.contract.view.form.inherit</field>
        <field name="model">hr.work.entry.type</field>
        <field name="inherit_id" ref="hr_work_entry.hr_work_entry_type_view_form"/>
        <field name="arch" type="xml">
            <group name="time_off" position="inside">
                <field name="is_leave"/>
            </group>
        </field>
    </record>

</odoo>

```

## File: wizard\hr_work_entry_regeneration_wizard.py

```python
# -*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError

from dateutil.relativedelta import relativedelta

class HrWorkEntryRegenerationWizard(models.TransientModel):
    _name = 'hr.work.entry.regeneration.wizard'
    _description = 'Regenerate Employee Work Entries'

    earliest_available_date = fields.Date('Earliest date', compute='_compute_earliest_available_date')
    earliest_available_date_message = fields.Char(readonly=True, store=False, default='')
    latest_available_date = fields.Date('Latest date', compute='_compute_latest_available_date')
    latest_available_date_message = fields.Char(readonly=True, store=False, default='')
    date_from = fields.Date('From', required=True, default=lambda self: self.env.context.get('date_start'))
    date_to = fields.Date('To', required=True, compute='_compute_date_to', store=True,
            readonly=False, default=lambda self: self.env.context.get('date_end'))
    employee_ids = fields.Many2many('hr.employee', string='Employees',
                                    domain=lambda self: [('company_id', 'in', self.env.companies.ids)], required=True)
    validated_work_entry_ids = fields.Many2many('hr.work.entry', string='Work Entries Within Interval',
                                   compute='_compute_validated_work_entry_ids')
    search_criteria_completed = fields.Boolean(compute='_compute_search_criteria_completed')
    valid = fields.Boolean(compute='_compute_valid')

    @api.depends('date_from')
    def _compute_date_to(self):
        for wizard in self:
            wizard.date_to = wizard.date_from and wizard.date_from + relativedelta(months=+1, day=1, days=-1)

    @api.depends('employee_ids')
    def _compute_earliest_available_date(self):
        for wizard in self:
            dates = wizard.employee_ids.contract_ids.mapped('date_generated_from')
            wizard.earliest_available_date = min(dates) if dates else None

    @api.depends('employee_ids')
    def _compute_latest_available_date(self):
        for wizard in self:
            dates = wizard.employee_ids.contract_ids.mapped('date_generated_to')
            wizard.latest_available_date = max(dates) if dates else None

    @api.depends('date_from', 'date_to', 'employee_ids')
    def _compute_validated_work_entry_ids(self):
        for wizard in self:
            validated_work_entry_ids = self.env['hr.work.entry']
            if wizard.search_criteria_completed:
                search_domain = [('employee_id', 'in', wizard.employee_ids.ids),
                                 ('date_start', '>=', wizard.date_from),
                                 ('date_stop', '<=', wizard.date_to),
                                 ('state', '=', 'validated')]
                validated_work_entry_ids = self.env['hr.work.entry'].search(search_domain, order="date_start")
            wizard.validated_work_entry_ids = validated_work_entry_ids

    @api.depends('validated_work_entry_ids')
    def _compute_valid(self):
        for wizard in self:
            wizard.valid = wizard.search_criteria_completed and len(wizard.validated_work_entry_ids) == 0

    @api.depends('date_from', 'date_to', 'employee_ids')
    def _compute_search_criteria_completed(self):
        for wizard in self:
            wizard.search_criteria_completed = wizard.date_from and wizard.date_to and wizard.employee_ids and wizard.earliest_available_date and wizard.latest_available_date

    @api.onchange('date_from', 'date_to', 'employee_ids')
    def _check_dates(self):
        for wizard in self:
            wizard.earliest_available_date_message = ''
            wizard.latest_available_date_message = ''
            if wizard.search_criteria_completed:
                if wizard.date_from > wizard.date_to:
                    date_from = wizard.date_from
                    wizard.date_from = wizard.date_to
                    wizard.date_to = date_from
                if wizard.earliest_available_date and wizard.date_from < wizard.earliest_available_date:
                    wizard.date_from = wizard.earliest_available_date
                    wizard.earliest_available_date_message = 'The earliest available date is {date}' \
                        .format(date=self._date_to_string(wizard.earliest_available_date))
                if wizard.latest_available_date and wizard.date_to > wizard.latest_available_date:
                    wizard.date_to = wizard.latest_available_date
                    wizard.latest_available_date_message = 'The latest available date is {date}' \
                        .format(date=self._date_to_string(wizard.latest_available_date))

    @api.model
    def _date_to_string(self, date):
        if not date:
            return ''
        user_date_format = self.env['res.lang']._get_data(code=self.env.user.lang).date_format
        return date.strftime(user_date_format)

    def _work_entry_fields_to_nullify(self):
        return ['active']

    def regenerate_work_entries(self):
        self.ensure_one()
        if not self.env.context.get('work_entry_skip_validation'):
            if not self.valid:
                raise ValidationError(_("In order to regenerate the work entries, you need to provide the wizard with an employee_id, a date_from and a date_to. In addition to that, the time interval defined by date_from and date_to must not contain any validated work entries."))

            if self.date_from < self.earliest_available_date or self.date_to > self.latest_available_date:
                raise ValidationError(_("The from date must be >= '%(earliest_available_date)s' and the to date must be <= '%(latest_available_date)s', which correspond to the generated work entries time interval.", earliest_available_date=self._date_to_string(self.earliest_available_date), latest_available_date=self._date_to_string(self.latest_available_date)))

        date_from = max(self.date_from, self.earliest_available_date) if self.earliest_available_date else self.date_from
        date_to = min(self.date_to, self.latest_available_date) if self.latest_available_date else self.date_to
        work_entries = self.env['hr.work.entry'].search([
            ('employee_id', 'in', self.employee_ids.ids),
            ('date_stop', '>=', date_from),
            ('date_start', '<=', date_to),
            ('state', '!=', 'validated')])

        write_vals = {field: False for field in self._work_entry_fields_to_nullify()}
        work_entries.write(write_vals)
        self.employee_ids.generate_work_entries(date_from, date_to, True)

```

## File: wizard\hr_work_entry_regeneration_wizard_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hr_work_entry_regeneration_wizard" model="ir.ui.view">
        <field name="name">hr_work_entry_regeneration_wizard</field>
        <field name="model">hr.work.entry.regeneration.wizard</field>
        <field name="arch" type="xml">
            <form string="Regenerate Employee Work Entries">
                <group>
                    <group>
                        <field name="employee_ids" widget="many2many_avatar_user"/>
                        <label for="date_from"></label>
                        <div name="date_from">
                            <div class="text-info" invisible="earliest_available_date_message == ''">
                                <i class="fa fa-info-circle me-1" title="Hint"/>
                                <field name="earliest_available_date_message" nolabel="1"/>
                            </div >
                            <field name="date_from"/>
                        </div>
                        <label for="date_to"></label>
                        <div name="date_to">
                            <div class="text-info" invisible="latest_available_date_message == ''">
                                <i class="fa fa-info-circle me-1" title="Hint"/>
                                <field name="latest_available_date_message" nolabel="1"/>
                            </div>
                            <field name="date_to"/>
                        </div>
                    </group>
                </group>
                <div>
                    <span class="text-muted">Warning: The work entry regeneration will delete all manual changes on the selected period.</span>
                </div>
                <field name="search_criteria_completed" invisible="1"/>
                <field name="valid" invisible="1"/>
                <div invisible="not search_criteria_completed or valid">
                    <div class="text-danger"><i class="fa fa-exclamation-triangle me-1" title="Warning"/>You are not allowed to regenerate validated work entries</div>
                    <field name="validated_work_entry_ids" widget="many2many" nolabel="1">
                        <list string="Work Entries"
                              default_order = "date_start"

                              editable="bottom"
                              no_open="1" decoration-danger="state == 'validated'">
                            <field name="state" column_invisible="True"/>
                            <field name="date_start"/>
                            <field name="date_stop"/>
                            <field name="work_entry_type_id"/>
                            <field name="state"/>
                        </list>
                    </field>
                </div>
                <footer>
                    <button name="regenerate_work_entries"
                            string="Regenerate Work Entries" data-hotkey="q"
                            class="btn btn-primary oe_highlight"
                            type="object"
                            invisible="not search_criteria_completed or not valid"/>
                    <button name="regenerate_work_entries_disabled"
                            string="Regenerate Work Entries"
                            class="btn btn-primary disabled"
                            invisible="search_criteria_completed and valid"/>
                    <button name="cancel_button" string="Cancel" class="btn-secondary" special="cancel" data-hotkey="x"/>
                </footer>
            </form>
        </field>
    </record>
    <record id="hr_work_entry_regeneration_wizard_action" model="ir.actions.act_window">
        <field name="name">Work Entry Regeneration</field>
        <field name="res_model">hr.work.entry.regeneration.wizard</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_work_entry_regeneration_wizard

```

