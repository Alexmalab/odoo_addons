# Odoo Module: hr_work_entry_holidays

Category: Human Resources/Payroll

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models


def _validate_existing_work_entry(env):
    env['hr.work.entry'].search([])._check_if_error()

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


{
    'name': 'Time Off in Payslips',
    'version': '1.0',
    'category': 'Human Resources/Payroll',
    'sequence': 95,
    'summary': 'Manage Time Off in Payslips',
    'description': """
Manage Time Off in Payslips
============================

This application allows you to integrate time off in payslips.
    """,
    'depends': ['hr_holidays', 'hr_work_entry_contract'],
    'data': [
        'data/hr_payroll_holidays_data.xml',
        'views/hr_leave_views.xml',
        'views/hr_leave_type_views.xml',
    ],
    'demo': ['data/hr_payroll_holidays_demo.xml'],
    'installable': True,
    'auto_install': True,
    'post_init_hook': '_validate_existing_work_entry',
    'license': 'LGPL-3',
}

```

## File: data\hr_payroll_holidays_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="0">

        <record id="hr_holidays.holiday_status_comp" model="hr.leave.type">
            <field name="work_entry_type_id" ref="hr_work_entry_contract.work_entry_type_compensatory"></field>
        </record>

        <record id="hr_holidays.holiday_status_unpaid" model="hr.leave.type">
            <field name="work_entry_type_id" ref="hr_work_entry_contract.work_entry_type_unpaid_leave"></field>
        </record>

        <record id="hr_holidays.holiday_status_sl" model="hr.leave.type">
            <field name="work_entry_type_id" ref="hr_work_entry_contract.work_entry_type_sick_leave"></field>
        </record>

        <record id="hr_holidays.holiday_status_cl" model="hr.leave.type">
            <field name="work_entry_type_id" ref="hr_work_entry_contract.work_entry_type_legal_leave"></field>
        </record>
    </data>
</odoo>

```

## File: data\hr_payroll_holidays_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <!-- add work_entry type to leave type -->
        <record id="hr_holidays.resource_public_time_off_1" model="resource.calendar.leaves">
            <field name="work_entry_type_id" ref="hr_work_entry_contract.work_entry_type_leave"></field>
        </record>
</odoo>

```

## File: models\hr_contract.py

```python
# -*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import pytz

from datetime import date
from odoo import api, models, _
from odoo.exceptions import ValidationError
from odoo.osv.expression import OR


class HrContract(models.Model):
    _inherit = 'hr.contract'
    _description = 'Employee Contract'

    @api.constrains('date_start', 'date_end', 'state')
    def _check_contracts(self):
        self._get_leaves()._check_contracts()

    def _get_leaves(self):
        return self.env['hr.leave'].search([
            ('state', '!=', 'refuse'),
            ('employee_id', 'in', self.mapped('employee_id.id')),
            ('date_from', '<=', max([end or date.max for end in self.mapped('date_end')])),
            ('date_to', '>=', min(self.mapped('date_start'))),
        ])

    # override to add work_entry_type from leave
    def _get_leave_work_entry_type(self, leave):
        if leave.holiday_id:
            return leave.holiday_id.holiday_status_id.work_entry_type_id
        else:
            return leave.work_entry_type_id

    def _get_more_vals_leave_interval(self, interval, leaves):
        result = super()._get_more_vals_leave_interval(interval, leaves)
        for leave in leaves:
            if interval[0] >= leave[0] and interval[1] <= leave[1]:
                result.append(('leave_id', leave[2].holiday_id.id))
        return result

    def _get_interval_leave_work_entry_type(self, interval, leaves, bypassing_codes):
        # returns the work entry time related to the leave that
        # includes the whole interval.
        # Overriden in hr_work_entry_contract_holiday to select the
        # global time off first (eg: Public Holiday > Home Working)
        self.ensure_one()
        if 'work_entry_type_id' in interval[2] and interval[2].work_entry_type_id.code in bypassing_codes:
            return interval[2].work_entry_type_id

        interval_start = interval[0].astimezone(pytz.utc).replace(tzinfo=None)
        interval_stop = interval[1].astimezone(pytz.utc).replace(tzinfo=None)
        including_rcleaves = [l[2] for l in leaves if l[2] and interval_start >= l[2].date_from and interval_stop <= l[2].date_to]
        including_global_rcleaves = [l for l in including_rcleaves if not l.holiday_id]
        including_holiday_rcleaves = [l for l in including_rcleaves if l.holiday_id]
        rc_leave = False

        # Example: In CP200: Long term sick > Public Holidays (which is global)
        if bypassing_codes:
            bypassing_rc_leave = [l for l in including_holiday_rcleaves if l.holiday_id.holiday_status_id.work_entry_type_id.code in bypassing_codes]
        else:
            bypassing_rc_leave = []

        if bypassing_rc_leave:
            rc_leave = bypassing_rc_leave[0]
        elif including_global_rcleaves:
            rc_leave = including_global_rcleaves[0]
        elif including_holiday_rcleaves:
            rc_leave = including_holiday_rcleaves[0]
        if rc_leave:
            return self._get_leave_work_entry_type_dates(rc_leave, interval_start, interval_stop, self.employee_id)
        return self.env.ref('hr_work_entry_contract.work_entry_type_leave')

    def _get_sub_leave_domain(self):
        domain = super()._get_sub_leave_domain()
        return OR([
            domain,
            [('holiday_id.employee_id', 'in', self.employee_id.ids)] # see https://github.com/odoo/enterprise/pull/15091
        ])

    def write(self, vals):
        # Special case when setting a contract as running:
        # If there is already a validated time off over another contract
        # with a different schedule, split the time off, before the
        # _check_contracts raises an issue.
        # If there are existing leaves that are spanned by this new
        # contract, update their resource calendar to the current one.
        if not (vals.get("state") == 'open' or vals.get('kanban_state') == 'done'):
            return super().write(vals)

        specific_contracts = self.env['hr.contract']
        all_new_leave_origin = []
        all_new_leave_vals = []
        leaves_state = {}
        # In case a validation error is thrown due to holiday creation with the new resource calendar (which can
        # increase their duration), we catch this error to display a more meaningful error message.
        try:
            for contract in self:
                if vals.get('state') != 'open' and contract.state != 'draft':
                    # In case the current contract is not in the draft state, the kanban_state transition does not
                    # cause any leave changes.
                    continue
                leaves = contract._get_leaves()
                for leave in leaves:
                    # Get all overlapping contracts but exclude draft contracts that are not included in this transaction.
                    overlapping_contracts = leave._get_overlapping_contracts(contract_states=[
                        ('state', '!=', 'cancel'),
                        ('resource_calendar_id', '!=', False),
                        '|', '|', ('id', 'in', self.ids),
                                  ('state', '!=', 'draft'),
                             ('kanban_state', '=', 'done'),
                    ]).sorted(key=lambda c: {'open': 1, 'close': 2, 'draft': 3, 'cancel': 4}[c.state])
                    if len(overlapping_contracts.resource_calendar_id) <= 1:
                        if overlapping_contracts and leave.resource_calendar_id != overlapping_contracts[0].resource_calendar_id:
                            leave.resource_calendar_id = overlapping_contracts[0].resource_calendar_id
                        continue
                    if leave.id not in leaves_state:
                        leaves_state[leave.id] = leave.state
                    if leave.state != 'refuse':
                        leave.action_refuse()
                    super(HrContract, contract).write(vals)
                    specific_contracts += contract
                    for overlapping_contract in overlapping_contracts:
                        new_request_date_from = max(leave.request_date_from, overlapping_contract.date_start)
                        new_request_date_to = min(leave.request_date_to, overlapping_contract.date_end or date.max)
                        new_leave_vals = leave.copy_data({
                            'request_date_from': new_request_date_from,
                            'request_date_to': new_request_date_to,
                            'state': leaves_state[leave.id],
                        })[0]
                        new_leave = self.env['hr.leave'].new(new_leave_vals)
                        new_leave._compute_date_from_to()
                        new_leave._compute_duration()
                        # Could happen for part-time contract, that time off is not necessary
                        # anymore.
                        if new_leave.date_from < new_leave.date_to:
                            all_new_leave_origin.append(leave)
                            all_new_leave_vals.append(new_leave._convert_to_write(new_leave._cache))
            if all_new_leave_vals:
                new_leaves = self.env['hr.leave'].with_context(
                    tracking_disable=True,
                    mail_activity_automation_skip=True,
                    leave_fast_create=True,
                    leave_skip_state_check=True
                ).create(all_new_leave_vals)
                new_leaves.filtered(lambda l: l.state in 'validate')._validate_leave_request()
                for index, new_leave in enumerate(new_leaves):
                    new_leave.message_post_with_source(
                        'mail.message_origin_link',
                        render_values={'self': new_leave, 'origin': all_new_leave_origin[index]},
                        subtype_xmlid='mail.mt_note',
                    )
        except ValidationError:
            raise ValidationError(_("Changing the contract on this employee changes their working schedule in a period "
                                    "they already took leaves. Changing this working schedule changes the duration of "
                                    "these leaves in such a way the employee no longer has the required allocation for "
                                    "them. Please review these leaves and/or allocations before changing the contract."))
        return super(HrContract, self - specific_contracts).write(vals)

```

## File: models\hr_employee_base.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class HrEmployeeBase(models.AbstractModel):
    _inherit = "hr.employee.base"

    def write(self, vals):
        # Prevent the resource calendar of leaves to be updated by a write to
        # employee. When this module is enabled the resource calendar of
        # leaves are determined by those of the contracts.
        return super(HrEmployeeBase, self.with_context(no_leave_resource_calendar_update=True)).write(vals)

```

## File: models\hr_leave.py

```python
# -*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict
from datetime import datetime, time
from dateutil.relativedelta import relativedelta

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError
from odoo.osv.expression import AND
from odoo.tools import format_date


class HrLeaveType(models.Model):
    _inherit = 'hr.leave.type'

    work_entry_type_id = fields.Many2one('hr.work.entry.type', string='Work Entry Type')


class HrLeave(models.Model):
    _inherit = 'hr.leave'

    def _prepare_resource_leave_vals(self):
        vals = super(HrLeave, self)._prepare_resource_leave_vals()
        vals['work_entry_type_id'] = self.holiday_status_id.work_entry_type_id.id
        return vals

    def _get_overlapping_contracts(self, contract_states=None):
        self.ensure_one()
        if contract_states is None:
            contract_states = [
                '|',
                ('state', 'not in', ['draft', 'cancel']),
                '&',
                ('state', '=', 'draft'),
                ('kanban_state', '=', 'done')
            ]
        domain = AND([contract_states, [
            ('employee_id', '=', self.employee_id.id),
            ('date_start', '<=', self.date_to),
            '|',
                ('date_end', '>=', self.date_from),
                ('date_end', '=', False),
        ]])
        return self.env['hr.contract'].sudo().search(domain)

    @api.constrains('date_from', 'date_to')
    def _check_contracts(self):
        """
            A leave cannot be set across multiple contracts.
            Note: a leave can be across multiple contracts despite this constraint.
            It happens if a leave is correctly created (not across multiple contracts) but
            contracts are later modifed/created in the middle of the leave.
        """
        for holiday in self.filtered('employee_id'):
            contracts = holiday._get_overlapping_contracts()
            if len(contracts.resource_calendar_id) > 1:
                state_labels = {e[0]: e[1] for e in contracts._fields['state']._description_selection(self.env)}
                raise ValidationError(
                    _("""A leave cannot be set across multiple contracts with different working schedules.

Please create one time off for each contract.

Time off:
%s

Contracts:
%s""",
                      holiday.display_name,
                      '\n'.join(_(
                          "Contract %s from %s to %s, status: %s",
                          contract.name,
                          format_date(self.env, contract.date_start),
                          format_date(self.env, contract.date_start) if contract.date_end else _("undefined"),
                          state_labels[contract.state]
                      ) for contract in contracts)))

    def _cancel_work_entry_conflict(self):
        """
        Creates a leave work entry for each hr.leave in self.
        Check overlapping work entries with self.
        Work entries completely included in a leave are archived.
        e.g.:
            |----- work entry ----|---- work entry ----|
                |------------------- hr.leave ---------------|
                                    ||
                                    vv
            |----* work entry ****|
                |************ work entry leave --------------|
        """
        if not self:
            return

        # 1. Create a work entry for each leave
        work_entries_vals_list = []
        for leave in self:
            contracts = leave.employee_id.sudo()._get_contracts(leave.date_from, leave.date_to, states=['open', 'close'])
            for contract in contracts:
                # Generate only if it has aleady been generated
                if leave.date_to >= contract.date_generated_from and leave.date_from <= contract.date_generated_to:
                    work_entries_vals_list += contracts._get_work_entries_values(leave.date_from, leave.date_to)

        new_leave_work_entries = self.env['hr.work.entry'].create(work_entries_vals_list)

        if new_leave_work_entries:
            # 2. Fetch overlapping work entries, grouped by employees
            start = min(self.mapped('date_from'), default=False)
            stop = max(self.mapped('date_to'), default=False)
            work_entry_groups = self.env['hr.work.entry']._read_group([
                ('date_start', '<', stop),
                ('date_stop', '>', start),
                ('employee_id', 'in', self.employee_id.ids),
            ], ['employee_id'], ['id:recordset'])
            work_entries_by_employee = {
                employee.id: work_entries
                for employee, work_entries in work_entry_groups
            }

            # 3. Archive work entries included in leaves
            included = self.env['hr.work.entry']
            overlappping = self.env['hr.work.entry']
            for work_entries in work_entries_by_employee.values():
                # Work entries for this employee
                new_employee_work_entries = work_entries & new_leave_work_entries
                previous_employee_work_entries = work_entries - new_leave_work_entries

                # Build intervals from work entries
                leave_intervals = new_employee_work_entries._to_intervals()
                conflicts_intervals = previous_employee_work_entries._to_intervals()

                # Compute intervals completely outside any leave
                # Intervals are outside, but associated records are overlapping.
                outside_intervals = conflicts_intervals - leave_intervals

                overlappping |= self.env['hr.work.entry']._from_intervals(outside_intervals)
                included |= previous_employee_work_entries - overlappping
            overlappping.write({'leave_id': False})
            included.write({'active': False})

    def write(self, vals):
        if not self:
            return True
        skip_check = not bool({'employee_id', 'state', 'request_date_from', 'request_date_to'} & vals.keys())
        employee_ids = self.employee_id.ids
        if 'employee_id' in vals and vals['employee_id']:
            employee_ids += [vals['employee_id']]
        # We check a whole day before and after the interval of the earliest
        # request_date_from and latest request_date_end because date_{from,to}
        # can lie in this range due to time zone reasons.
        # (We can't use date_from and date_to as they are not yet computed at
        # this point.)
        start_dates = self.filtered('request_date_from').mapped('request_date_from') + [fields.Date.to_date(vals.get('request_date_from', False)) or datetime.max.date()]
        stop_dates = self.filtered('request_date_to').mapped('request_date_to') + [fields.Date.to_date(vals.get('request_date_to', False)) or datetime.min.date()]
        start = datetime.combine(min(start_dates) - relativedelta(days=1), time.min)
        stop = datetime.combine(max(stop_dates) + relativedelta(days=1), time.max)
        with self.env['hr.work.entry']._error_checking(start=start, stop=stop, skip=skip_check, employee_ids=employee_ids):
            return super().write(vals)

    @api.model_create_multi
    def create(self, vals_list):
        if any(vals.get('holiday_type', 'employee') == 'employee' and not vals.get('multi_employee', False) and not vals.get('employee_id', False) for vals in vals_list):
            raise ValidationError(_("There is no employee set on the time off. Please make sure you're logged in the correct company."))
        employee_ids = {v['employee_id'] for v in vals_list if v.get('employee_id')}
        # We check a whole day before and after the interval of the earliest
        # request_date_from and latest request_date_end because date_{from,to}
        # can lie in this range due to time zone reasons.
        # (We can't use date_from and date_to as they are not yet computed at
        # this point.)
        start_dates = [fields.Date.to_date(v.get('request_date_from')) for v in vals_list if v.get('request_date_from')]
        stop_dates = [fields.Date.to_date(v.get('request_date_to')) for v in vals_list if v.get('request_date_to')]
        start = datetime.combine(min(start_dates, default=datetime.max.date()) - relativedelta(days=1), time.min)
        stop = datetime.combine(max(stop_dates, default=datetime.min.date()) + relativedelta(days=1), time.max)
        with self.env['hr.work.entry']._error_checking(start=start, stop=stop, employee_ids=employee_ids):
            return super().create(vals_list)

    def action_confirm(self):
        start = min(self.mapped('date_from'), default=False)
        stop = max(self.mapped('date_to'), default=False)
        with self.env['hr.work.entry']._error_checking(start=start, stop=stop, employee_ids=self.employee_id.ids):
            return super().action_confirm()

    def _get_leaves_on_public_holiday(self):
        return super()._get_leaves_on_public_holiday().filtered(
            lambda l: l.holiday_status_id.work_entry_type_id.code not in ['LEAVE110', 'LEAVE210', 'LEAVE280'])

    def _validate_leave_request(self):
        super(HrLeave, self)._validate_leave_request()
        self.sudo()._cancel_work_entry_conflict()  # delete preexisting conflicting work_entries
        return True

    def action_refuse(self):
        """
        Override to archive linked work entries and recreate attendance work entries
        where the refused leave was.
        """
        res = super(HrLeave, self).action_refuse()
        self._regen_work_entries()
        return res

    def _action_user_cancel(self, reason):
        res = super()._action_user_cancel(reason)
        self.sudo()._regen_work_entries()
        return res

    def _regen_work_entries(self):
        """
        Called when the leave is refused or cancelled to regenerate the work entries properly for that period.
        """
        work_entries = self.env['hr.work.entry'].sudo().search([('leave_id', 'in', self.ids)])

        work_entries.write({'active': False})
        # Re-create attendance work entries
        vals_list = []
        for work_entry in work_entries:
            vals_list += work_entry.contract_id._get_work_entries_values(work_entry.date_start, work_entry.date_stop)
        self.env['hr.work.entry'].create(vals_list)

    def _compute_can_cancel(self):
        super()._compute_can_cancel()

        cancellable_leaves = self.filtered('can_cancel')
        work_entries = self.env['hr.work.entry'].sudo().search([('state', '=', 'validated'), ('leave_id', 'in', cancellable_leaves.ids)])
        leave_ids = work_entries.mapped('leave_id').ids

        for leave in cancellable_leaves:
            leave.can_cancel = leave.id not in leave_ids

```

## File: models\hr_work_entry.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict
from dateutil.relativedelta import relativedelta

from odoo import api, fields, models


class HrWorkEntry(models.Model):
    _inherit = 'hr.work.entry'

    leave_id = fields.Many2one('hr.leave', string='Time Off')
    leave_state = fields.Selection(related='leave_id.state')

    def _is_duration_computed_from_calendar(self):
        return super()._is_duration_computed_from_calendar() or bool(not self.work_entry_type_id and self.leave_id)

    def write(self, vals):
        if 'state' in vals and vals['state'] == 'cancelled':
            self.mapped('leave_id').filtered(lambda l: l.state != 'refuse').action_refuse()
        return super().write(vals)

    def _reset_conflicting_state(self):
        super()._reset_conflicting_state()
        attendances = self.filtered(lambda w: w.work_entry_type_id and not w.work_entry_type_id.is_leave)
        attendances.write({'leave_id': False})

    def _check_if_error(self):
        res = super()._check_if_error()
        conflict_with_leaves = self._compute_conflicts_leaves_to_approve()
        return res or conflict_with_leaves

    def _compute_conflicts_leaves_to_approve(self):
        if not self:
            return False

        self.flush_recordset(['date_start', 'date_stop', 'employee_id', 'active'])
        self.env['hr.leave'].flush_model(['date_from', 'date_to', 'state', 'employee_id'])

        query = """
            SELECT
                b.id AS work_entry_id,
                l.id AS leave_id
            FROM hr_work_entry b
            INNER JOIN hr_leave l ON b.employee_id = l.employee_id
            WHERE
                b.active = TRUE AND
                b.id IN %s AND
                l.date_from < b.date_stop AND
                l.date_to > b.date_start AND
                l.state IN ('confirm', 'validate1');
        """
        self.env.cr.execute(query, [tuple(self.ids)])
        conflicts = self.env.cr.dictfetchall()
        for res in conflicts:
            self.browse(res.get('work_entry_id')).write({
                'state': 'conflict',
                'leave_id': res.get('leave_id')
            })
        return bool(conflicts)

    def action_approve_leave(self):
        self.ensure_one()
        if self.leave_id:
            # Already confirmed once
            if self.leave_id.state == 'validate1':
                self.leave_id.action_validate()
            # Still in confirmed state
            else:
                self.leave_id.action_approve()
                # If double validation, still have to validate it again
                if self.leave_id.validation_type == 'both':
                    self.leave_id.action_validate()

    def action_refuse_leave(self):
        self.ensure_one()
        leave_sudo = self.leave_id.sudo()
        if leave_sudo:
            leave_sudo.action_refuse()

    @api.model
    def _get_leaves_duration_between_two_dates(self, employee_id, date_from, date_to):
        date_from += relativedelta(hour=0, minute=0, second=0)
        date_to += relativedelta(hour=23, minute=59, second=59)
        leaves_work_entries = self.env['hr.work.entry'].search([
            ('employee_id', '=', employee_id.id),
            ('date_start', '>=', date_from),
            ('date_stop', '<=', date_to),
            ('state', '!=', 'cancelled'),
            ('leave_id', '!=', False),
            ('leave_state', '=', 'validate'),
        ])
        entries_by_leave_type = defaultdict(lambda: self.env['hr.work.entry'])
        for work_entry in leaves_work_entries:
            entries_by_leave_type[work_entry.leave_id.holiday_status_id] |= work_entry

        durations_by_leave_type = {}
        for leave_type, work_entries in entries_by_leave_type.items():
            durations_by_leave_type[leave_type] = sum(work_entries.mapped('duration'))
        return durations_by_leave_type


class HrWorkEntryType(models.Model):
    _inherit = 'hr.work.entry.type'
    _description = 'HR Work Entry Type'

    leave_type_ids = fields.One2many(
        'hr.leave.type', 'work_entry_type_id', string='Time Off Type',
        help="Work entry used in the payslip.")

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import hr_contract
from . import hr_employee_base
from . import hr_leave
from . import hr_work_entry

```

## File: views\hr_leave_type_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="view_holiday_status_normal_tree" model="ir.ui.view">
        <field name="model">hr.leave.type</field>
        <field name="inherit_id" ref="hr_holidays.view_holiday_status_normal_tree"/>
        <field name="arch" type="xml">
            <field name="allocation_validation_type" position="after">
                <field name="work_entry_type_id" optional="show"/>
            </field>
        </field>
    </record>
</odoo>

```

## File: views\hr_leave_views.xml

```xml
<?xml version='1.0' encoding='UTF-8' ?>
<odoo>
    <!-- Work entry type on hr leave -->
    <record id="work_entry_type_leave_form_inherit" model="ir.ui.view">
        <field name="name">work_entry.type.leave.form.inherit</field>
        <field name="model">hr.leave.type</field>
        <field name="inherit_id" ref="hr_holidays.edit_holiday_status_form"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='visual']" position="before">
                <group>
                    <group name="payroll" string="Payroll">
                        <field name="work_entry_type_id"/>
                    </group>
                </group>
            </xpath>
        </field>
    </record>

    <record id="hr_leave_view_tree_inherit_work_entry" model="ir.ui.view">
        <field name="name">hr.holidays.view.tree.inherit.work.entry</field>
        <field name="model">hr.leave</field>
        <field name="inherit_id" ref="hr_holidays.hr_leave_view_tree"/>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <xpath expr="//tree" position="attributes">
                <attribute name="default_order">date_from</attribute>
            </xpath>
        </field>
    </record>

</odoo>

```

