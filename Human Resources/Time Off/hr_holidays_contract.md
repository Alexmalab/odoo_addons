# Odoo Module: hr_holidays_contract

Category: Human Resources/Time off

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.


{
    'name': 'Hr Holidays Contract',
    'version': '1.0',
    'category': 'Human Resources/Time off',
    'summary': 'Bridge module between time off and contract',
    'description': """
        Bridge module to manage time off based on contracts.
    """,
    'depends': ['hr_holidays', 'hr_contract'],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\hr_contract.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import date
from odoo import api, models, _
from odoo.exceptions import ValidationError


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
# Part of Odoo. See LICENSE file for full copyright and licensing details.


from odoo import api, models, _
from odoo.exceptions import ValidationError
from odoo.osv.expression import AND
from odoo.tools import format_date


class HrLeave(models.Model):
    _inherit = 'hr.leave'

    def _compute_resource_calendar_id(self):
        super()._compute_resource_calendar_id()
        for leave in self.filtered(lambda l: l.employee_id):
            # We use the request dates to find the contracts, because date_from
            # and date_to are not set yet at this point. Since these dates are
            # used to get the contracts for which these leaves apply and
            # contract start- and end-dates are just dates (and not datetimes)
            # these dates are comparable.
            if leave.employee_id:
                contracts = self.env['hr.contract'].search([
                    '|', ('state', 'in', ['open', 'close']),
                         '&', ('state', '=', 'draft'),
                              ('kanban_state', '=', 'done'),
                    ('employee_id', '=', leave.employee_id.id),
                    ('date_start', '<=', leave.request_date_to),
                    '|', ('date_end', '=', False),
                         ('date_end', '>=', leave.request_date_from),
                ])
                if contracts:
                    # If there are more than one contract they should all have the
                    # same calendar, otherwise a constraint is violated.
                    leave.resource_calendar_id = contracts[:1].resource_calendar_id

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
%(time_off)s

Contracts:
%(contracts)s""",
                      time_off=holiday.display_name,
                      contracts='\n'.join(_(
                          "Contract %(contract)s from %(start_date)s to %(end_date)s, status: %(status)s",
                          contract=contract.name,
                          start_date=format_date(self.env, contract.date_start),
                          end_date=format_date(self.env, contract.date_end) if contract.date_end else _("undefined"),
                          status=state_labels[contract.state]
                      ) for contract in contracts)))

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_contract
from . import hr_leave
from . import hr_employee_base

```

