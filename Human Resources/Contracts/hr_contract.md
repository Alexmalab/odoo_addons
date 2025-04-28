# Odoo Module: hr_contract

Category: Human Resources/Contracts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Employee Contracts',
    'version': '1.0',
    'category': 'Human Resources/Contracts',
    'sequence': 335,
    'description': """
Add all information on the employee form to manage contracts.
=============================================================

    * Contract
    * Place of Birth,
    * Medical Examination Date
    * Company Vehicle

You can assign several contracts per employee.
    """,
    'website': 'https://www.odoo.com/page/employees',
    'depends': ['hr'],
    'data': [
        'security/security.xml',
        'security/ir.model.access.csv',
        'data/hr_contract_data.xml',
        'views/hr_contract_views.xml',
        'views/assets.xml',
        'wizard/hr_departure_wizard_views.xml',
    ],
    'demo': ['data/hr_contract_demo.xml'],
    'installable': True,
    'auto_install': False,
    'application': True,
    'license': 'LGPL-3',
}

```

## File: data\hr_contract_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- Structure Type -->
        <record id="structure_type_employee" model="hr.payroll.structure.type">
            <field name="name">Employee</field>
            <field name="country_id" eval="False"/>
        </record>
        <record id="structure_type_worker" model="hr.payroll.structure.type">
            <field name="name">Worker</field>
            <field name="country_id" eval="False"/>
        </record>
        <record id="structure_type_employee_cp200_pfi" model="hr.payroll.structure.type">
            <field name="name">CP200 PFI: Belgian Employee</field>
            <field name="default_resource_calendar_id" ref="resource.resource_calendar_std_38h"/>
            <field name="country_id" ref="base.be"/>
        </record>
        <record id="structure_type_employee_cp200" model="hr.payroll.structure.type">
            <field name="name">CP200: Belgian Employee</field>
            <field name="default_resource_calendar_id" ref="resource.resource_calendar_std_38h"/>
            <field name="country_id" ref="base.be"/>
        </record>

        <!-- Contract-related subtypes for messaging / Chatter -->
        <record id="mt_contract_pending" model="mail.message.subtype">
            <field name="name">To Renew</field>
            <field name="res_model">hr.contract</field>
            <field name="default" eval="True"/>
            <field name="description">Contract about to expire</field>
        </record>
        <record id="mt_contract_close" model="mail.message.subtype">
            <field name="name">Expired</field>
            <field name="res_model">hr.contract</field>
            <field name="default" eval="False"/>
            <field name="description">Contract expired</field>
        </record>
        <!-- Department-related (parent) subtypes for messaging / Chatter -->
        <record id="mt_department_contract_pending" model="mail.message.subtype">
            <field name="name">Contract to Renew</field>
            <field name="res_model">hr.department</field>
            <field name="default" eval="False"/>
            <field name="parent_id" ref="mt_contract_pending"/>
            <field name="relation_field">department_id</field>
            <field name="description">Contract about to expire</field>
        </record>

        <!-- Expired Soon -->
        <record id="ir_cron_data_contract_update_state" model="ir.cron">
            <field name="name">HR Contract: update state</field>
            <field name="model_id" ref="model_hr_contract"/>
            <field name="type">ir.actions.server</field>
            <field name="state">code</field>
            <field name="code">model.with_context(from_cron=True).update_state()</field>
            <field name="interval_number">1</field>
            <field name="interval_type">days</field>
            <field name="numbercall">-1</field>
        </record>
    </data>
</odoo>

```

## File: data\hr_contract_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="base.user_demo" model="res.users">
        <field name="groups_id" eval="[(4, ref('hr_contract.group_hr_contract_manager'))]"/>
    </record>

    <record id="hr_contract_admin" model="hr.contract">
        <field name="name">Mitchell Admin Contract</field>
        <field name="date_start" eval="time.strftime('%Y')+'-1-1'"/>
        <field name="employee_id" ref="hr.employee_admin"/>
        <field name="job_id" model="hr.job"
            eval="obj().env.ref('hr.employee_admin').job_id.id"/>
        <field name="department_id" model="hr.department"
            eval="obj().env.ref('hr.employee_admin').department_id.id"/>
        <field eval="7540.0" name="wage"/>
        <field name="state">draft</field>
        <field name="kanban_state">normal</field>
    </record>

    <record id="hr_contract_al" model="hr.contract">
        <field name="name">Ronnie Hart Contract</field>
        <field name="date_start" eval="time.strftime('%Y')+'-1-1'"/>
        <field name="employee_id" ref="hr.employee_al"/>
        <field name="job_id" model="hr.job"
            eval="obj().env.ref('hr.employee_al').job_id.id"/>
        <field name="department_id" model="hr.department"
            eval="obj().env.ref('hr.employee_al').department_id.id"/>
        <field eval="4000.0" name="wage"/>
        <field name="state">open</field>
        <field name="kanban_state">normal</field>
    </record>

    <record id="hr_contract_mit" model="hr.contract">
        <field name="name">Marketing Executive Contract</field>
        <field name="date_start" eval="time.strftime('%Y')+'-3-1'"/>
        <field name="employee_id" ref="hr.employee_mit"/>
        <field name="job_id" model="hr.job"
            eval="obj().env.ref('hr.employee_mit').job_id.id"/>
        <field name="department_id" model="hr.department"
            eval="obj().env.ref('hr.employee_mit').department_id.id"/>
        <field eval="4500.0" name="wage"/>
        <field name="state">open</field>
        <field name="kanban_state">normal</field>
    </record>

    <record id="hr_contract_stw" model="hr.contract">
        <field name="name">Randall Lewis Contract</field>
        <field name="date_start" eval="time.strftime('%Y')+'-2-1'"/>
        <field name="date_end" eval="time.strftime('%Y')+'-12-1'"/>
        <field name="employee_id" ref="hr.employee_stw"/>
        <field name="job_id" model="hr.job"
            eval="obj().env.ref('hr.employee_stw').job_id.id"/>
        <field name="department_id" model="hr.department"
            eval="obj().env.ref('hr.employee_stw').department_id.id"/>
        <field eval="4500.0" name="wage"/>
        <field name="state">open</field>
        <field name="kanban_state">normal</field>
    </record>

    <record id="hr_contract_qdp" model="hr.contract">
        <field name="name">Demo Contract</field>
        <field name="date_start" eval="time.strftime('%Y')+'-3-1'"/>
        <field name="employee_id" ref="hr.employee_qdp"/>
        <field name="job_id" model="hr.job"
            eval="obj().env.ref('hr.employee_qdp').job_id.id"/>
        <field name="department_id" model="hr.department"
            eval="obj().env.ref('hr.employee_qdp').department_id.id"/>
        <field eval="3750.0" name="wage"/>
        <field name="state">draft</field>
        <field name="kanban_state">normal</field>
    </record>

    <record id="hr_contract_han" model="hr.contract">
        <field name="name">Walter Horton Contract</field>
        <field name="date_start" eval="time.strftime('%Y')+'-3-1'"/>
        <field name="employee_id" ref="hr.employee_han"/>
        <field name="job_id" model="hr.job"
            eval="obj().env.ref('hr.employee_han').job_id.id"/>
        <field name="department_id" model="hr.department"
            eval="obj().env.ref('hr.employee_han').department_id.id"/>
        <field eval="4600.0" name="wage"/>
        <field name="state">open</field>
        <field name="kanban_state">normal</field>
    </record>

    <record id="hr_contract_niv" model="hr.contract">
        <field name="name">Sharlene Rhodes Contract</field>
        <field name="date_start" eval="time.strftime('%Y-%m')+'-1'"/>
        <field name="date_end" eval="time.strftime('%Y')+'-12-1'"/>
        <field name="employee_id" ref="hr.employee_niv"/>
        <field name="job_id" model="hr.job"
            eval="obj().env.ref('hr.employee_niv').job_id.id"/>
        <field name="department_id" model="hr.department"
            eval="obj().env.ref('hr.employee_niv').department_id.id"/>
        <field eval="4000.0" name="wage"/>
        <field name="state">draft</field>
        <field name="kanban_state">normal</field>
    </record>

    <record id="hr_contract_jth" model="hr.contract">
        <field name="name">Toni Jimenez</field>
        <field name="date_start" eval="time.strftime('%Y-%m')+'-1'"/>
        <field name="date_end" eval="time.strftime('%Y')+'-12-1'"/>
        <field name="employee_id" ref="hr.employee_jth"/>
        <field name="job_id" model="hr.job"
            eval="obj().env.ref('hr.employee_jth').job_id.id"/>
        <field name="department_id" model="hr.department"
            eval="obj().env.ref('hr.employee_jth').department_id.id"/>
        <field eval="4200.0" name="wage"/>
        <field name="state">draft</field>
        <field name="kanban_state">normal</field>
    </record>

    <record id="hr_contract_chs" model="hr.contract">
        <field name="name">Jennie Fletcher Contract</field>
        <field name="date_start" eval="time.strftime('%Y-%m')+'-1'"/>
        <field name="employee_id" ref="hr.employee_chs"/>
        <field name="job_id" model="hr.job"
            eval="obj().env.ref('hr.employee_chs').job_id.id"/>
        <field name="department_id" model="hr.department"
            eval="obj().env.ref('hr.employee_chs').department_id.id"/>
        <field eval="3750.0" name="wage"/>
        <field name="state">cancel</field>
        <field name="kanban_state">normal</field>
    </record>

    <record id="hr_contract_jve" model="hr.contract">
        <field name="name">Paul Williams Contract</field>
        <field name="date_start" eval="time.strftime('%Y-%m')+'-1'"/>
        <field name="employee_id" ref="hr.employee_jve"/>
        <field name="job_id" model="hr.job"
            eval="obj().env.ref('hr.employee_jve').job_id.id"/>
        <field name="department_id" model="hr.department"
            eval="obj().env.ref('hr.employee_jve').department_id.id"/>
        <field eval="3950.0" name="wage"/>
        <field name="state">cancel</field>
        <field name="kanban_state">normal</field>
    </record>

    <record id="hr_contract_fme" model="hr.contract">
        <field name="name">Keith Byrd Contract</field>
        <field name="date_start" eval="'2015-1-1'"/>
        <field name="date_end" eval="time.strftime('%Y-%m-%d')"/>
        <field name="employee_id" ref="hr.employee_fme"/>
        <field name="job_id" model="hr.job"
            eval="obj().env.ref('hr.employee_fme').job_id.id"/>
        <field name="department_id" model="hr.department"
            eval="obj().env.ref('hr.employee_fme').department_id.id"/>
        <field eval="3650.0" name="wage"/>
        <field name="state">open</field>
        <field name="kanban_state">blocked</field>
    </record>

    <record id="hr_contract_fpi" model="hr.contract">
        <field name="name">Audrey Peterson Contract</field>
        <field name="date_start" eval="'2015-1-1'"/>
        <field name="date_end" eval="'2017-12-1'"/>
        <field name="employee_id" ref="hr.employee_fpi"/>
        <field name="job_id" model="hr.job"
            eval="obj().env.ref('hr.employee_fpi').job_id.id"/>
        <field name="department_id" model="hr.department"
            eval="obj().env.ref('hr.employee_fpi').department_id.id"/>
        <field eval="3750.0" name="wage"/>
        <field name="state">close</field>
        <field name="kanban_state">normal</field>
    </record>

    <record id="hr_contract_vad" model="hr.contract">
        <field name="name">Tina Williamson Contract</field>
        <field name="date_start" eval="'2015-1-1'"/>
        <field name="date_end" eval="'2018-2-1'"/>
        <field name="employee_id" ref="hr.employee_vad"/>
        <field name="job_id" model="hr.job"
            eval="obj().env.ref('hr.employee_vad').job_id.id"/>
        <field name="department_id" model="hr.department"
            eval="obj().env.ref('hr.employee_vad').department_id.id"/>
        <field eval="3750.0" name="wage"/>
        <field name="state">close</field>
        <field name="kanban_state">normal</field>
    </record>

</odoo>
```

## File: models\hr_contract.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import threading

from datetime import date
from dateutil.relativedelta import relativedelta

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError

from odoo.osv import expression

import logging
_logger = logging.getLogger(__name__)


class Contract(models.Model):
    _name = 'hr.contract'
    _description = 'Contract'
    _inherit = ['mail.thread', 'mail.activity.mixin']

    name = fields.Char('Contract Reference', required=True)
    active = fields.Boolean(default=True)
    structure_type_id = fields.Many2one('hr.payroll.structure.type', string="Salary Structure Type")
    employee_id = fields.Many2one('hr.employee', string='Employee', tracking=True, domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    department_id = fields.Many2one('hr.department', compute='_compute_employee_contract', store=True, readonly=False,
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]", string="Department")
    job_id = fields.Many2one('hr.job', compute='_compute_employee_contract', store=True, readonly=False,
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]", string='Job Position')
    date_start = fields.Date('Start Date', required=True, default=fields.Date.today, tracking=True,
        help="Start date of the contract.")
    date_end = fields.Date('End Date', tracking=True,
        help="End date of the contract (if it's a fixed-term contract).")
    trial_date_end = fields.Date('End of Trial Period',
        help="End date of the trial period (if there is one).")
    resource_calendar_id = fields.Many2one(
        'resource.calendar', 'Working Schedule', compute='_compute_employee_contract', store=True, readonly=False,
        default=lambda self: self.env.company.resource_calendar_id.id, copy=False, index=True,
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    wage = fields.Monetary('Wage', required=True, tracking=True, help="Employee's monthly gross wage.")
    notes = fields.Text('Notes')
    state = fields.Selection([
        ('draft', 'New'),
        ('open', 'Running'),
        ('close', 'Expired'),
        ('cancel', 'Cancelled')
    ], string='Status', group_expand='_expand_states', copy=False,
       tracking=True, help='Status of the contract', default='draft')
    company_id = fields.Many2one('res.company', compute='_compute_employee_contract', store=True, readonly=False,
        default=lambda self: self.env.company, required=True)
    company_country_id = fields.Many2one('res.country', string="Company country", related='company_id.country_id', readonly=True)

    """
        kanban_state:
            * draft + green = "Incoming" state (will be set as Open once the contract has started)
            * open + red = "Pending" state (will be set as Closed once the contract has ended)
            * red = Shows a warning on the employees kanban view
    """
    kanban_state = fields.Selection([
        ('normal', 'Grey'),
        ('done', 'Green'),
        ('blocked', 'Red')
    ], string='Kanban State', default='normal', tracking=True, copy=False)
    currency_id = fields.Many2one(string="Currency", related='company_id.currency_id', readonly=True)
    permit_no = fields.Char('Work Permit No', related="employee_id.permit_no", readonly=False)
    visa_no = fields.Char('Visa No', related="employee_id.visa_no", readonly=False)
    visa_expire = fields.Date('Visa Expire Date', related="employee_id.visa_expire", readonly=False)
    hr_responsible_id = fields.Many2one('res.users', 'HR Responsible', tracking=True,
        help='Person responsible for validating the employee\'s contracts.')
    calendar_mismatch = fields.Boolean(compute='_compute_calendar_mismatch')
    first_contract_date = fields.Date(related='employee_id.first_contract_date')

    @api.depends('employee_id.resource_calendar_id', 'resource_calendar_id')
    def _compute_calendar_mismatch(self):
        for contract in self:
            contract.calendar_mismatch = contract.resource_calendar_id != contract.employee_id.resource_calendar_id

    def _expand_states(self, states, domain, order):
        return [key for key, val in self._fields['state'].selection]

    @api.depends('employee_id')
    def _compute_employee_contract(self):
        for contract in self.filtered('employee_id'):
            contract.job_id = contract.employee_id.job_id
            contract.department_id = contract.employee_id.department_id
            contract.resource_calendar_id = contract.employee_id.resource_calendar_id
            contract.company_id = contract.employee_id.company_id

    @api.onchange('company_id')
    def _onchange_company_id(self):
        if self.company_id:
            structure_types = self.env['hr.payroll.structure.type'].search([
                '|',
                ('country_id', '=', self.company_id.country_id.id),
                ('country_id', '=', False)])
            if structure_types:
                self.structure_type_id = structure_types[0]
            elif self.structure_type_id not in structure_types:
                self.structure_type_id = False

    @api.onchange('structure_type_id')
    def _onchange_structure_type_id(self):
        if self.structure_type_id.default_resource_calendar_id:
            self.resource_calendar_id = self.structure_type_id.default_resource_calendar_id

    @api.constrains('employee_id', 'state', 'kanban_state', 'date_start', 'date_end')
    def _check_current_contract(self):
        """ Two contracts in state [incoming | open | close] cannot overlap """
        for contract in self.filtered(lambda c: (c.state not in ['draft', 'cancel'] or c.state == 'draft' and c.kanban_state == 'done') and c.employee_id):
            domain = [
                ('id', '!=', contract.id),
                ('employee_id', '=', contract.employee_id.id),
                ('company_id', '=', contract.company_id.id),
                '|',
                    ('state', 'in', ['open', 'close']),
                    '&',
                        ('state', '=', 'draft'),
                        ('kanban_state', '=', 'done') # replaces incoming
            ]

            if not contract.date_end:
                start_domain = []
                end_domain = ['|', ('date_end', '>=', contract.date_start), ('date_end', '=', False)]
            else:
                start_domain = [('date_start', '<=', contract.date_end)]
                end_domain = ['|', ('date_end', '>', contract.date_start), ('date_end', '=', False)]

            domain = expression.AND([domain, start_domain, end_domain])
            if self.search_count(domain):
                raise ValidationError(_('An employee can only have one contract at the same time. (Excluding Draft and Cancelled contracts)'))

    @api.constrains('date_start', 'date_end')
    def _check_dates(self):
        if self.filtered(lambda c: c.date_end and c.date_start > c.date_end):
            raise ValidationError(_('Contract start date must be earlier than contract end date.'))

    @api.model
    def update_state(self):
        from_cron = 'from_cron' in self.env.context
        contracts = self.search([
            ('state', '=', 'open'), ('kanban_state', '!=', 'blocked'),
            '|',
            '&',
            ('date_end', '<=', fields.Date.to_string(date.today() + relativedelta(days=7))),
            ('date_end', '>=', fields.Date.to_string(date.today() + relativedelta(days=1))),
            '&',
            ('visa_expire', '<=', fields.Date.to_string(date.today() + relativedelta(days=60))),
            ('visa_expire', '>=', fields.Date.to_string(date.today() + relativedelta(days=1))),
        ])

        for contract in contracts:
            contract.activity_schedule(
                'mail.mail_activity_data_todo', contract.date_end,
                _("The contract of %s is about to expire.", contract.employee_id.name),
                user_id=contract.hr_responsible_id.id or self.env.uid)

        if contracts:
            contracts._safe_write_for_cron({'kanban_state': 'blocked'}, from_cron)

        contracts_to_close = self.search([
            ('state', '=', 'open'),
            '|',
            ('date_end', '<=', fields.Date.to_string(date.today())),
            ('visa_expire', '<=', fields.Date.to_string(date.today())),
        ])

        if contracts_to_close:
            contracts_to_close._safe_write_for_cron({'state': 'close'}, from_cron)

        contracts_to_open = self.search([('state', '=', 'draft'), ('kanban_state', '=', 'done'), ('date_start', '<=', fields.Date.to_string(date.today())),])

        if contracts_to_open:
            contracts_to_open._safe_write_for_cron({'state': 'open'}, from_cron)

        contract_ids = self.search([('date_end', '=', False), ('state', '=', 'close'), ('employee_id', '!=', False)])
        # Ensure all closed contract followed by a new contract have a end date.
        # If closed contract has no closed date, the work entries will be generated for an unlimited period.
        for contract in contract_ids:
            next_contract = self.search([
                ('employee_id', '=', contract.employee_id.id),
                ('state', 'not in', ['cancel', 'draft']),
                ('date_start', '>', contract.date_start)
            ], order="date_start asc", limit=1)
            if next_contract:
                contract._safe_write_for_cron({'date_end': next_contract.date_start - relativedelta(days=1)}, from_cron)
                continue
            next_contract = self.search([
                ('employee_id', '=', contract.employee_id.id),
                ('date_start', '>', contract.date_start)
            ], order="date_start asc", limit=1)
            if next_contract:
                contract._safe_write_for_cron({'date_end': next_contract.date_start - relativedelta(days=1)}, from_cron)

        return True

    def _safe_write_for_cron(self, vals, from_cron=False):
        if from_cron:
            auto_commit = not getattr(threading.current_thread(), 'testing', False)
            for contract in self:
                try:
                    with self.env.cr.savepoint():
                        contract.write(vals)
                except ValidationError as e:
                    _logger.warning(e)
                else:
                    if auto_commit:
                        self.env.cr.commit()
        else:
            self.write(vals)

    def _assign_open_contract(self):
        for contract in self:
            contract.employee_id.sudo().write({'contract_id': contract.id})

    def _get_contract_wage(self):
        self.ensure_one()
        return self[self._get_contract_wage_field()]

    def _get_contract_wage_field(self):
        self.ensure_one()
        return 'wage'

    def write(self, vals):
        res = super(Contract, self).write(vals)
        if vals.get('state') == 'open':
            self._assign_open_contract()
        if vals.get('state') == 'close':
            for contract in self.filtered(lambda c: not c.date_end):
                contract.date_end = max(date.today(), contract.date_start)

        calendar = vals.get('resource_calendar_id')
        if calendar:
            self.filtered(lambda c: c.state == 'open' or (c.state == 'draft' and c.kanban_state == 'done')).mapped('employee_id').write({'resource_calendar_id': calendar})

        if 'state' in vals and 'kanban_state' not in vals:
            self.write({'kanban_state': 'normal'})

        return res

    @api.model
    def create(self, vals):
        contracts = super(Contract, self).create(vals)
        if vals.get('state') == 'open':
            contracts._assign_open_contract()
        open_contracts = contracts.filtered(lambda c: c.state == 'open' or c.state == 'draft' and c.kanban_state == 'done')
        # sync contract calendar -> calendar employee
        for contract in open_contracts.filtered(lambda c: c.employee_id and c.resource_calendar_id):
            contract.employee_id.resource_calendar_id = contract.resource_calendar_id
        return contracts

    def _track_subtype(self, init_values):
        self.ensure_one()
        if 'state' in init_values and self.state == 'open' and 'kanban_state' in init_values and self.kanban_state == 'blocked':
            return self.env.ref('hr_contract.mt_contract_pending')
        elif 'state' in init_values and self.state == 'close':
            return self.env.ref('hr_contract.mt_contract_close')
        return super(Contract, self)._track_subtype(init_values)

```

## File: models\hr_contract_type.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class HrPayrollStructureType(models.Model):
    _name = 'hr.payroll.structure.type'
    _description = 'Contract Type'

    name = fields.Char('Contract Type')
    default_resource_calendar_id = fields.Many2one(
        'resource.calendar', 'Default Working Hours',
        default=lambda self: self.env.company.resource_calendar_id)
    country_id = fields.Many2one('res.country', string='Country', default=lambda self: self.env.company.country_id)

```

## File: models\hr_employee.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo.osv import expression


class Employee(models.Model):
    _inherit = "hr.employee"

    vehicle = fields.Char(string='Company Vehicle', groups="hr.group_hr_user")
    contract_ids = fields.One2many('hr.contract', 'employee_id', string='Employee Contracts')
    contract_id = fields.Many2one('hr.contract', string='Current Contract',
        groups="hr.group_hr_user",domain="[('company_id', '=', company_id)]", help='Current contract of the employee')
    calendar_mismatch = fields.Boolean(related='contract_id.calendar_mismatch')
    contracts_count = fields.Integer(compute='_compute_contracts_count', string='Contract Count')
    contract_warning = fields.Boolean(string='Contract Warning', store=True, compute='_compute_contract_warning', groups="hr.group_hr_user")
    first_contract_date = fields.Date(compute='_compute_first_contract_date', groups="hr.group_hr_user")

    def _get_first_contracts(self):
        self.ensure_one()
        return self.sudo().contract_ids.filtered(lambda c: c.state != 'cancel')

    @api.depends('contract_ids.state', 'contract_ids.date_start')
    def _compute_first_contract_date(self):
        for employee in self:
            contracts = employee._get_first_contracts()
            if contracts:
                employee.first_contract_date = min(contracts.mapped('date_start'))
            else:
                employee.first_contract_date = False

    @api.depends('contract_id', 'contract_id.state', 'contract_id.kanban_state')
    def _compute_contract_warning(self):
        for employee in self:
            employee.contract_warning = not employee.contract_id or employee.contract_id.kanban_state == 'blocked' or employee.contract_id.state != 'open'

    def _compute_contracts_count(self):
        # read_group as sudo, since contract count is displayed on form view
        contract_data = self.env['hr.contract'].sudo().read_group([('employee_id', 'in', self.ids)], ['employee_id'], ['employee_id'])
        result = dict((data['employee_id'][0], data['employee_id_count']) for data in contract_data)
        for employee in self:
            employee.contracts_count = result.get(employee.id, 0)

    def _get_contracts(self, date_from, date_to, states=['open'], kanban_state=False):
        """
        Returns the contracts of the employee between date_from and date_to
        """
        state_domain = [('state', 'in', states)]
        if kanban_state:
            state_domain = expression.AND([state_domain, [('kanban_state', 'in', kanban_state)]])

        return self.env['hr.contract'].search(
            expression.AND([[('employee_id', 'in', self.ids)],
            state_domain,
            [('date_start', '<=', date_to),
                '|',
                    ('date_end', '=', False),
                    ('date_end', '>=', date_from)]]))

    def _get_incoming_contracts(self, date_from, date_to):
        return self._get_contracts(date_from, date_to, states=['draft'], kanban_state=['done'])

    @api.model
    def _get_all_contracts(self, date_from, date_to, states=['open']):
        """
        Returns the contracts of all employees between date_from and date_to
        """
        return self.search([])._get_contracts(date_from, date_to, states=states)

    def write(self, vals):
        res = super(Employee, self).write(vals)
        if vals.get('contract_id'):
            for employee in self:
                employee.resource_calendar_id.transfer_leaves_to(employee.contract_id.resource_calendar_id, employee.resource_id)
                employee.resource_calendar_id = employee.contract_id.resource_calendar_id
        return res

```

## File: models\resource.py

```python
# -*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from datetime import datetime

from odoo import models, fields, api
from odoo.osv.expression import AND


class ResourceCalendar(models.Model):
    _inherit = 'resource.calendar'

    def transfer_leaves_to(self, other_calendar, resources=None, from_date=None):
        """
            Transfer some resource.calendar.leaves from 'self' to another calendar 'other_calendar'.
            Transfered leaves linked to `resources` (or all if `resources` is None) and starting
            after 'from_date' (or today if None).
        """
        from_date = from_date or fields.Datetime.now().replace(hour=0, minute=0, second=0, microsecond=0)
        domain = [
            ('calendar_id', 'in', self.ids),
            ('date_from', '>=', from_date),
        ]
        domain = AND([domain, [('resource_id', 'in', resources.ids)]]) if resources else domain

        self.env['resource.calendar.leaves'].search(domain).write({
            'calendar_id': other_calendar.id,
        })


```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields, api, _


class User(models.Model):
    _inherit = ['res.users']

    vehicle = fields.Char(related="employee_id.vehicle")
    bank_account_id = fields.Many2one(related="employee_id.bank_account_id")

    def __init__(self, pool, cr):
        """ Override of __init__ to add access rights.
            Access rights are disabled by default, but allowed
            on some specific fields defined in self.SELF_{READ/WRITE}ABLE_FIELDS.
        """
        contract_readable_fields = [
            'vehicle',
            'bank_account_id',
        ]
        init_res = super(User, self).__init__(pool, cr)
        # duplicate list to avoid modifying the original reference
        pool[self._name].SELF_READABLE_FIELDS = pool[self._name].SELF_READABLE_FIELDS + contract_readable_fields
        return init_res

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_employee
from . import hr_contract
from . import res_users
from . import resource
from . import hr_contract_type

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_hr_resource_manager,hr.employee.resource.manager,resource.model_resource_resource,hr.group_hr_manager,1,1,1,1
access_hr_resource_calendar_user,hr.employee.resource.calendar.user,resource.model_resource_calendar,hr.group_hr_user,1,1,1,1
access_hr_resource_calendar_attendance_user,hr.employee.resource.calendar.attendance.user,resource.model_resource_calendar_attendance,hr.group_hr_user,1,1,1,1
access_hr_contract_manager,hr.contract.manager,model_hr_contract,hr_contract.group_hr_contract_manager,1,1,1,1
access_hr_payroll_structure_type_hr_contract_manager,hr.payroll.structure.type.contract.manager,model_hr_payroll_structure_type,hr_contract.group_hr_contract_manager,1,1,1,1

```

## File: security\security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record model="ir.module.category" id="base.module_category_human_resources_contracts">
            <field name="description">Helps you manage your contracts.</field>
            <field name="sequence">10</field>
        </record>

        <record id="hr_contract.group_hr_contract_manager" model="res.groups">
            <field name="name">Administrator</field>
            <field name="category_id" ref="base.module_category_human_resources_contracts"/>
            <field name="implied_ids" eval="[(4, ref('base.group_user')), (4, ref('hr.group_hr_user'))]"/>
            <field name="users" eval="[(4, ref('base.user_root')), (4, ref('base.user_admin'))]"/>
        </record>

        <record id="base.default_user" model="res.users">
            <field name="groups_id" eval="[(4,ref('hr_contract.group_hr_contract_manager'))]"/>
        </record>

        <record id="ir_rule_hr_contract_multi_company" model="ir.rule">
            <field name="name">HR Contract: Multi Company</field>
            <field name="model_id" ref="model_hr_contract"/>
            <field name="domain_force">[('company_id', 'in', company_ids)]</field>
        </record>

        <record id="ir_rule_hr_payroll_structure_type_multi_company" model="ir.rule">
            <field name="name">HR Payroll Structure Type: Multi Company</field>
            <field name="model_id" ref="model_hr_payroll_structure_type"/>
            <field name="global" eval="True"/>
            <field name="domain_force">['|', ('country_id', '=', False), ('country_id', 'in', user.env.companies.mapped('country_id').ids)]</field>
        </record>

    </data>
</odoo>

```

## File: views\assets.xml

```xml
<odoo>
    <template id="assets_backend" inherit_id="web.assets_backend">
        <xpath expr="." position="inside">
            <link rel="stylesheet" type="text/scss" href="/hr_contract/static/src/scss/state_selection.scss"/>
            <link rel="stylesheet" type="text/scss" href="/hr_contract/static/src/scss/calendar_mismatch.scss"/>
        </xpath>
    </template>
</odoo>

```

## File: views\hr_contract_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

        <record id="act_hr_employee_2_hr_contract" model="ir.actions.act_window">
            <field name="name">Contracts</field>
            <field name="res_model">hr.contract</field>
            <field name="view_mode">kanban,tree,form</field>
            <field name="context">{
                'search_default_employee_id': [active_id],
                'default_employee_id': active_id,
                'search_default_group_by_state': 1
            }</field>
        </record>

        <record id="hr_hr_employee_view_form2" model="ir.ui.view">
            <field name="name">hr.hr.employee.view.form2</field>
            <field name="model">hr.employee</field>
            <field name="inherit_id" ref="hr.view_employee_form"/>
            <field name="arch" type="xml">
                <data>
                    <div name="button_box" position="inside">
                        <field name="contract_warning" invisible="1"/>
                        <button name="%(act_hr_employee_2_hr_contract)d"
                            class="oe_stat_button"
                            icon="fa-book"
                            type="action"
                            groups="hr_contract.group_hr_contract_manager">
                            <div attrs="{'invisible' : [('first_contract_date', '=', False)]}" class="o_stat_info">
                                <span class="o_stat_text text-success" attrs="{'invisible' : [('contract_warning', '=', True)]}" title="In Contract Since"> In Contract Since</span>
                                <span class="o_stat_value text-success" attrs="{'invisible' : [('contract_warning', '=', True)]}">
                                    <field name="first_contract_date" readonly="1"/>
                                </span>
                                <span class="o_stat_text text-danger" attrs="{'invisible' : [('contract_warning', '=', False)]}" title="In Contract Since">
                                    In Contract Since
                                </span>
                                <span class="o_stat_value text-danger" attrs="{'invisible' : [('contract_warning', '=', False)]}">
                                    <field name="first_contract_date" readonly="1"/>
                                </span>
                            </div>
                            <div attrs="{'invisible' : [('first_contract_date', '!=', False)]}" class="o_stat_info">
                                <span class="o_stat_value text-danger">
                                   <field name="contracts_count"/>
                                </span>
                                <span class="o_stat_text text-danger">
                                    Contracts
                                </span>
                            </div>
                        </button>
                    </div>
                    <xpath expr="//field[@name='user_id']" position="before">
                        <field name="first_contract_date" attrs="{'invisible' : [('first_contract_date', '=', False)]}" readonly="1"/>
                    </xpath>
                    <xpath expr="//field[@name='bank_account_id']" position="replace">
                        <field name="bank_account_id" context="{'display_partner':True}" attrs="{'invisible' : [('address_home_id', '=', False)]}"/>
                    </xpath>
                    <xpath expr="//field[@name='resource_calendar_id']" position="replace">
                        <field name="calendar_mismatch" invisible="1"/>
                        <label for="resource_calendar_id"/>
                        <div>
                            <field name="resource_calendar_id" required="1" nolabel="1"/>
                            <span attrs="{'invisible': [('calendar_mismatch', '=', False)]}"
                                class="fa fa-exclamation-triangle text-danger o_calendar_warning pl-3">
                            </span>
                            <span class="o_calendar_warning_tooltip text-danger">
                                Calendar Mismatch : The employee's calendar does not match its current contract calendar. This could lead to unexpected behaviors.
                            </span>
                        </div>
                    </xpath>
                </data>
            </field>
        </record>

        <record id="hr_employee_view_search" model="ir.ui.view">
            <field name="name">hr.employee.view.search</field>
            <field name="model">hr.employee</field>
            <field name="inherit_id" ref="hr.view_employee_filter"/>
            <field name="arch" type="xml">
                <data>
                    <xpath expr="//filter[@name='message_needaction']" position="after">
                        <separator/>
                        <filter string="Contract Warning" name="with_contract_warning" domain="[('contract_warning', '=', True)]"/>
                    </xpath>
                </data>
            </field>
        </record>

        <record id="hr_user_view_form" model="ir.ui.view">
        <field name="name">hr.user.preferences.view.form.contract.inherit</field>
        <field name="model">res.users</field>
        <field name="inherit_id" ref="hr.res_users_view_form_profile"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='employee_bank_account_id']" position="replace">
                <field name="employee_bank_account_id" context="{'display_partner':True}" attrs="{'readonly': [('can_edit', '=', False)]}"/>
            </xpath>
        </field>
    </record>

        <record id="hr_contract_view_search" model="ir.ui.view">
            <field name="name">hr.contract.search</field>
            <field name="model">hr.contract</field>
            <field name="arch" type="xml">
                <search string="Search Contract">
                    <field name="name" string="Contract"/>
                    <field name="date_start"/>
                    <field name="date_end"/>
                    <field name="employee_id"/>
                    <field name="job_id"/>
                    <field name="department_id" operator="child_of"/>
                    <field name="resource_calendar_id"/>
                    <filter string="Running" name="running" domain="[('state', '=', 'open')]"/>
                    <filter string="Need Action" name="to_renew" domain="['&amp;', ('state', '=', 'open'), ('kanban_state', '=', 'blocked')]"/>
                    <separator />
                    <filter string="Employed" name="current_employee" domain="[('employee_id.active', '=', True)]"/>
                    <separator/>
                    <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
                    <separator/>
                    <filter string="Late Activities" name="activities_overdue"
                        domain="[('my_activity_date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                        help="Show all records which have a next action date before today"/>
                    <filter string="Today Activities" name="activities_today"
                        domain="[('my_activity_date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                    <filter string="Future Activities" name="activities_upcoming_all"
                        domain="[('my_activity_date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
                    <group expand="0" string="Group By">
                        <filter string="Employee" name="employee" domain="[]" context="{'group_by': 'employee_id'}"/>
                        <filter string="Job Position" name="job" domain="[]" context="{'group_by': 'job_id'}"/>
                        <filter string="Status" name='group_by_state' domain="[]" context="{'group_by': 'state'}"/>
                    </group>
                </search>
            </field>
        </record>

        <record id="hr_contract_view_form" model="ir.ui.view">
            <field name="name">hr.contract.form</field>
            <field name="model">hr.contract</field>
            <field name="arch" type="xml">
                <form string="Current Contract">
                    <header>
                        <field name="state" widget="statusbar" options="{'clickable': '1'}"/>
                    </header>
                    <sheet>
                    <div class="oe_button_box" name="button_box"/>
                    <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                    <div class="oe_title pr-0" name="title">
                        <h1 class="d-flex flex-row justify-content-between">
                            <field name="name" class="text-truncate" placeholder="Contract Reference"/>
                            <field name="kanban_state" widget="state_selection"/>
                        </h1>
                    </div>
                    <group>
                        <group>
                            <field name="active" invisible="1"/>
                            <field name="employee_id"/>
                            <field name="department_id"/>
                            <field name="job_id"/>
                            <field name="company_id" groups="base.group_multi_company"/>
                            <field name="company_country_id" invisible="1"/>
                            <field name="structure_type_id" domain="['|', ('country_id', '=', False), ('country_id', '=', company_country_id)]"/>
                        </group>
                        <group name="duration_group">
                            <field name="date_start"/>
                            <field name="first_contract_date" attrs="{'invisible': ['|', ('first_contract_date', '=', False), ('first_contract_date', '=', 'date_start')]}"/>
                            <field name="date_end"/>
                            <field name="calendar_mismatch" invisible="1"/>
                            <label for="resource_calendar_id"/>
                            <div>
                                <field name="resource_calendar_id" required="1" nolabel="1"/>
                                <span attrs="{'invisible': ['|', ('calendar_mismatch', '=', False), ('state', '!=', 'open')]}"
                                    class="fa fa-exclamation-triangle text-danger o_calendar_warning pl-3">
                                </span>
                                <span class="o_calendar_warning_tooltip text-danger">
                                    Calendar Mismatch : The employee's calendar does not match this contract's calendar. This could lead to unexpected behaviors.
                                </span>
                            </div>
                            <field name="hr_responsible_id"/>
                        </group>
                    </group>
                    <notebook>
                        <page string="Contract Details" name="other">
                            <group name="notes_group" string="Notes">
                                <field name="notes" nolabel="1"/>
                            </group>
                        </page>
                        <page string="Salary Information" name="information">
                            <group name="main_info">
                                <group name="salary_and_advantages" string="Monthly Advantages in Cash">
                                    <label for="wage"/>
                                    <div class="o_row" name="wage">
                                        <field name="wage" nolabel="1"/>
                                        <span>/ month</span>
                                    </div>
                                </group>
                                <group string="Yearly Advantages" name="yearly_advantages"/>
                            </group>
                        </page>
                    </notebook>
                    </sheet>
                    <div class="oe_chatter">
                        <field name="message_follower_ids" groups="base.group_user"/>
                        <field name="activity_ids"/>
                        <field name="message_ids"/>
                    </div>
                </form>
            </field>
        </record>

        <record id="hr_contract_view_tree" model="ir.ui.view">
            <field name="name">hr.contract.tree</field>
            <field name="model">hr.contract</field>
            <field name="arch" type="xml">
                <tree string="Contracts" multi_edit="1" sample="1">
                    <field name="name" readonly="1"/>
                    <field name="employee_id" readonly="1" widget="many2one_avatar_employee"/>
                    <field name="job_id"/>
                    <field name="resource_calendar_id"/>
                    <field name="date_start" readonly="1"/>
                    <field name="date_end" readonly="1"/>
                    <field name="state" widget="badge" decoration-info="state == 'draft'" decoration-warning="state == 'close'" decoration-success="state == 'open'"/>
                    <field name="kanban_state" widget="state_selection" readonly="1"/>
                    <field name="wage" invisible="1"/>
                    <field name="message_needaction" invisible="1"/>
                    <field name="company_id" groups="base.group_multi_company" readonly="1"/>
                </tree>
            </field>
        </record>

        <record id="hr_contract_view_kanban" model="ir.ui.view">
            <field name="name">hr.contract.kanban</field>
            <field name="model">hr.contract</field>
            <field name="arch" type="xml">
                <kanban class="o_kanban_small_column" default_order="date_end" sample="1">
                    <field name="employee_id"/>
                    <field name="activity_state"/>
                    <field name="state"/>
                    <progressbar field="activity_state" colors='{"planned": "success", "today": "warning", "overdue": "danger"}'/>
                    <templates>
                    <t t-name="kanban-box">
                        <div class="oe_kanban_card oe_kanban_global_click">
                            <div class="o_dropdown_kanban dropdown" t-if="!selection_mode" groups="base.group_user">
                                <a class="dropdown-toggle o-no-caret btn" role="button" data-toggle="dropdown" data-display="static" href="#" aria-label="Dropdown menu" title="Dropdown menu">
                                    <span class="fa fa-ellipsis-v"/>
                                </a>
                                <div class="dropdown-menu" role="menu">
                                    <t t-if="widget.editable"><a role="menuitem" type="edit" class="dropdown-item">Edit Contract</a></t>
                                    <t t-if="widget.deletable"><a role="menuitem" type="delete" class="dropdown-item">Delete</a></t>
                                </div>
                            </div>
                            <div class="oe_kanban_content">
                                <div class="o_hr_contract_state">
                                    <strong class="o_kanban_record_title">
                                        <field name="name"/>
                                    </strong>
                                </div>
                                <div class="text-muted o_kanban_record_subtitle o_hr_contract_job_id">
                                    <field name="job_id"/>
                                </div>
                                <div class="oe_kanban_bottom_right">
                                    <span class="float-right">
                                        <field name="employee_id" widget="many2one_avatar_employee"/>
                                    </span>
                                    <span class="float-right mr4">
                                        <field name="kanban_state" widget="state_selection"/>
                                    </span>
                                </div>
                            </div>
                            <div class="oe_clear"></div>
                        </div>
                    </t>
                    </templates>
                </kanban>
            </field>
         </record>

        <record id="hr_contract_view_activity" model="ir.ui.view">
            <field name="name">hr.contract.activity</field>
            <field name="model">hr.contract</field>
            <field name="arch" type="xml">
                <activity string="Contracts">
                    <field name="employee_id"/>
                    <templates>
                        <div t-name="activity-box">
                            <img t-att-src="activity_image('hr.employee', 'image_128', record.employee_id.raw_value)" t-att-title="record.employee_id.value" t-att-alt="record.employee_id.value"/>
                            <div>
                                <field name="name" display="full"/>
                                <field name="job_id" muted="1" display="full"/>
                            </div>
                        </div>
                    </templates>
                </activity>
            </field>
        </record>

        <record id="action_hr_contract" model="ir.actions.act_window">
            <field name="name">Contracts</field>
            <field name="res_model">hr.contract</field>
            <field name="view_mode">kanban,tree,form,activity</field>
            <field name="domain">[('employee_id', '!=', False)]</field>
            <field name="context">{'search_default_current':1, 'search_default_group_by_state': 1}</field>
            <field name="search_view_id" ref="hr_contract_view_search"/>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Create a new contract
              </p>
            </field>
        </record>

        <menuitem
            id="menu_human_resources_configuration_contract"
            name="Contracts"
            parent="hr.menu_human_resources_configuration"
            sequence="3"/>

        <menuitem
            id="hr_menu_contract"
            name="Contracts"
            action="action_hr_contract"
            parent="hr.menu_hr_employee_payroll"
            sequence="4"
            groups="hr_contract.group_hr_contract_manager"/>


</odoo>

```

## File: wizard\hr_departure_wizard.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _
from odoo.exceptions import UserError


class HrDepartureWizard(models.TransientModel):
    _inherit = 'hr.departure.wizard'

    set_date_end = fields.Boolean(string="Set Contract End Date", default=True)

    def action_register_departure(self):
        """If set_date_end is checked, set the departure date as the end date to current running contract,
        and cancel all draft contracts"""
        current_contract = self.employee_id.contract_id
        if current_contract and current_contract.date_start > self.departure_date:
            raise UserError(_("Departure date can't be earlier than the start date of current contract."))

        super(HrDepartureWizard, self).action_register_departure()
        if self.set_date_end:
            self.employee_id.contract_ids.filtered(lambda c: c.state == 'draft').write({'state': 'cancel'})
            if current_contract:
                self.employee_id.contract_id.write({'date_end': self.departure_date})

```

## File: wizard\hr_departure_wizard_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hr_departure_wizard_view_form" model="ir.ui.view">
        <field name="name">hr.departure.wizard.view.form.extend</field>
        <field name="model">hr.departure.wizard</field>
        <field name="inherit_id" ref="hr.hr_departure_wizard_view_form" />
        <field name="arch" type="xml">
            <xpath expr="//field[@name='departure_date']" position="replace">
                <field name="departure_date" string="Contract End Date"/>
            </xpath>
            <xpath expr="//group[@id='date']" position="inside">
                <field name="set_date_end" string="Set end date to current contract"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_departure_wizard

```

