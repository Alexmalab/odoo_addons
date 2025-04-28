# Odoo Module: hr_contract

Category: Human Resources/Contracts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import report
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
    'website': 'https://www.odoo.com/app/employees',
    'depends': ['hr'],
    'data': [
        'security/security.xml',
        'security/ir.model.access.csv',
        'data/hr_contract_data.xml',
        'report/hr_contract_history_report_views.xml',
        'views/hr_contract_views.xml',
        'views/hr_employee_views.xml',
        'views/hr_employee_public_views.xml',
        'views/resource_calendar_views.xml',
        'wizard/hr_departure_wizard_views.xml',
    ],
    'demo': ['data/hr_contract_demo.xml'],
    'installable': True,
    'auto_install': False,
    'application': True,
    'assets': {
        'web.assets_backend': [
            'hr_contract/static/src/**/*',
        ],
    },
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

    <record id="hr_contract_admin_new" model="hr.contract">
        <field name="name">Contract For Mitchell Admin</field>
        <field name="date_start" eval="time.strftime('%Y-%m')+'-1'"/>
        <field name="date_end" eval="time.strftime('%Y')+'-12-31'"/>
        <field name="structure_type_id" ref="hr_contract.structure_type_employee"/>
        <field name="employee_id" ref="hr.employee_admin"/>
        <field name="notes">This is Mitchell Admin's contract</field>
        <field eval="5500.0" name="wage"/>
        <field name="state">open</field>
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

    <record id="hr_contract_fpi_previous" model="hr.contract">
        <field name="name">Audrey Peterson Contract</field>
        <field name="date_start" eval="'2014-1-1'"/>
        <field name="date_end" eval="'2014-12-31'"/>
        <field name="employee_id" ref="hr.employee_fpi"/>
        <field name="job_id" model="hr.job"
            eval="obj().env.ref('hr.employee_fpi').job_id.id"/>
        <field name="department_id" model="hr.department"
            eval="obj().env.ref('hr.employee_fpi').department_id.id"/>
        <field eval="3700.0" name="wage"/>
        <field name="state">close</field>
        <field name="kanban_state">normal</field>
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
        help="Start date of the contract.", index=True)
    date_end = fields.Date('End Date', tracking=True,
        help="End date of the contract (if it's a fixed-term contract).")
    trial_date_end = fields.Date('End of Trial Period',
        help="End date of the trial period (if there is one).")
    resource_calendar_id = fields.Many2one(
        'resource.calendar', 'Working Schedule', compute='_compute_employee_contract', store=True, readonly=False,
        default=lambda self: self.env.company.resource_calendar_id.id, copy=False, index=True,
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    wage = fields.Monetary('Wage', required=True, tracking=True, help="Employee's monthly gross wage.")
    contract_wage = fields.Monetary('Contract Wage', compute='_compute_contract_wage')
    notes = fields.Html('Notes')
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
    country_code = fields.Char(related='company_country_id.code', readonly=True)
    contract_type_id = fields.Many2one('hr.contract.type', "Contract Type")

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
    visa_expire = fields.Date('Visa Expiration Date', related="employee_id.visa_expire", readonly=False)
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
                raise ValidationError(
                    _(
                        'An employee can only have one contract at the same time. (Excluding Draft and Cancelled contracts).\n\nEmployee: %(employee_name)s',
                        employee_name=contract.employee_id.name
                    )
                )

    @api.constrains('date_start', 'date_end')
    def _check_dates(self):
        for contract in self:
            if contract.date_end and contract.date_start > contract.date_end:
                raise ValidationError(_(
                    'Contract %(contract)s: start date (%(start)s) must be earlier than contract end date (%(end)s).',
                    contract=contract.name, start=contract.date_start, end=contract.date_end,
                ))

    def _get_employee_vals_to_update(self):
        self.ensure_one()
        vals = {'contract_id': self.id}
        if self.job_id and self.job_id != self.employee_id.job_id:
            vals['job_id'] = self.job_id.id
        if self.department_id:
            vals['department_id'] = self.department_id.id
        return vals

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
            vals = contract._get_employee_vals_to_update()
            contract.employee_id.sudo().write(vals)

    @api.depends('wage')
    def _compute_contract_wage(self):
        for contract in self:
            contract.contract_wage = contract._get_contract_wage()

    def _get_contract_wage(self):
        self.ensure_one()
        return self[self._get_contract_wage_field()]

    def _get_contract_wage_field(self):
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
            self.filtered(
                lambda c: c.state == 'open' or (c.state == 'draft' and c.kanban_state == 'done' and c.employee_id.contracts_count == 1)
            ).employee_id.resource_calendar_id = calendar

        if 'state' in vals and 'kanban_state' not in vals:
            self.write({'kanban_state': 'normal'})

        return res

    @api.model
    def create(self, vals):
        contracts = super(Contract, self).create(vals)
        if vals.get('state') == 'open':
            contracts._assign_open_contract()
        open_contracts = contracts.filtered(
            lambda c: c.state == 'open' or (c.state == 'draft' and c.kanban_state == 'done' and c.employee_id.contracts_count == 1)
        )
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

    def action_open_contract_form(self):
        self.ensure_one()
        return {
            "type": "ir.actions.act_window",
            "res_model": "hr.contract",
            "views": [[False, "form"]],
            "res_id": self.id,
        }

```

## File: models\hr_contract_type.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ContractType(models.Model):
    _name = 'hr.contract.type'
    _description = 'Contract Type'

    name = fields.Char(required=True)

class HrPayrollStructureType(models.Model):
    _name = 'hr.payroll.structure.type'
    _description = 'Salary Structure Type'

    name = fields.Char('Salary Structure Type')
    default_resource_calendar_id = fields.Many2one(
        'resource.calendar', 'Default Working Hours',
        default=lambda self: self.env.company.resource_calendar_id)
    country_id = fields.Many2one('res.country', string='Country', default=lambda self: self.env.company.country_id)

```

## File: models\hr_employee.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import date, datetime, time
from pytz import timezone

from odoo import _, api, fields, models
from odoo.osv import expression
from odoo.addons.resource.models.resource import Intervals
from odoo.exceptions import UserError


class Employee(models.Model):
    _inherit = "hr.employee"

    vehicle = fields.Char(string='Company Vehicle', groups="hr.group_hr_user")
    contract_ids = fields.One2many('hr.contract', 'employee_id', string='Employee Contracts')
    contract_id = fields.Many2one(
        'hr.contract', string='Current Contract', groups="hr.group_hr_user",
        domain="[('company_id', '=', company_id), ('employee_id', '=', id)]", help='Current contract of the employee', copy=False)
    calendar_mismatch = fields.Boolean(related='contract_id.calendar_mismatch')
    contracts_count = fields.Integer(compute='_compute_contracts_count', string='Contract Count')
    contract_warning = fields.Boolean(string='Contract Warning', store=True, compute='_compute_contract_warning', groups="hr.group_hr_user")
    first_contract_date = fields.Date(compute='_compute_first_contract_date', groups="hr.group_hr_user", store=True)

    def _get_first_contracts(self):
        self.ensure_one()
        return self.sudo().contract_ids.filtered(lambda c: c.state != 'cancel')

    def _get_first_contract_date(self, no_gap=True):
        self.ensure_one()
        def remove_gap(contracts):
            # We do not consider a gap of more than 4 days to be a same occupation
            # contracts are considered to be ordered correctly
            if not contracts:
                return self.env['hr.contract']
            if len(contracts) == 1:
                return contracts
            current_contract = contracts[0]
            older_contracts = contracts[1:]
            current_date = current_contract.date_start
            for i, other_contract in enumerate(older_contracts):
                # Consider current_contract.date_end being false as an error and cut the loop
                gap = (current_date - (other_contract.date_end or date(2100, 1, 1))).days
                current_date = other_contract.date_start
                if gap >= 4:
                    return older_contracts[0:i] + current_contract
            return older_contracts + current_contract

        contracts = self._get_first_contracts().sorted('date_start', reverse=True)
        if no_gap:
            contracts = remove_gap(contracts)
        return min(contracts.mapped('date_start')) if contracts else False

    @api.depends('contract_ids.state', 'contract_ids.date_start', 'contract_ids.active')
    def _compute_first_contract_date(self):
        for employee in self:
            employee.first_contract_date = employee._get_first_contract_date()

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
        return self.search(['|', ('active', '=', True), ('active', '=', False)])._get_contracts(date_from, date_to, states=states)

    def _get_expected_attendances(self, date_from, date_to, domain=None):
        self.ensure_one()
        valid_contracts = self.sudo()._get_contracts(date_from, date_to, states=['open', 'close'])
        if not valid_contracts:
            return super()._get_expected_attendances(date_from, date_to, domain)
        employee_tz = timezone(self.tz) if self.tz else None
        duration_data = Intervals()
        for contract in valid_contracts:
            contract_start = datetime.combine(contract.date_start, time.min, employee_tz)
            contract_end = datetime.combine(contract.date_end or date.max, time.max, employee_tz)
            calendar = contract.resource_calendar_id or contract.company_id.resource_calendar_id
            contract_intervals = calendar._work_intervals_batch(
                                    max(date_from, contract_start),
                                    min(date_to, contract_end),
                                    tz=employee_tz,
                                    domain=domain,
                                    resources=self.resource_id)[self.resource_id.id]
            duration_data = duration_data | contract_intervals
        return duration_data


    def write(self, vals):
        res = super(Employee, self).write(vals)
        if vals.get('contract_id'):
            for employee in self:
                employee.resource_calendar_id.transfer_leaves_to(employee.contract_id.resource_calendar_id, employee.resource_id)
                employee.resource_calendar_id = employee.contract_id.resource_calendar_id
        return res
    
    @api.ondelete(at_uninstall=False)
    def _unlink_except_open_contract(self):
        if any(contract.state == 'open' for contract in self.contract_ids):
            raise UserError(_('You cannot delete an employee with a running contract.'))

    def action_open_contract_history(self):
        self.ensure_one()
        action = self.env["ir.actions.actions"]._for_xml_id('hr_contract.hr_contract_history_view_form_action')
        action['res_id'] = self.id
        return action

```

## File: models\hr_employee_public.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class HrEmployeePublic(models.Model):
    _inherit = "hr.employee.public"

    first_contract_date = fields.Date(related='employee_id.first_contract_date', groups="base.group_user")

```

## File: models\resource.py

```python
# -*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import datetime

from odoo import fields, models
from odoo.osv.expression import AND


class ResourceCalendar(models.Model):
    _inherit = 'resource.calendar'

    contracts_count = fields.Integer("# Contracts using it", compute='_compute_contracts_count', groups="hr_contract.group_hr_contract_manager")

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

    def _compute_contracts_count(self):
        count_data = self.env['hr.contract'].read_group(
            [('resource_calendar_id', 'in', self.ids)],
            ['resource_calendar_id'],
            ['resource_calendar_id'])
        mapped_counts = {cd['resource_calendar_id'][0]: cd['resource_calendar_id_count'] for cd in count_data}
        for calendar in self:
            calendar.contracts_count = mapped_counts.get(calendar.id, 0)

    def action_open_contracts(self):
        self.ensure_one()
        action = self.env["ir.actions.actions"]._for_xml_id("hr_contract.action_hr_contract")
        action.update({'domain': [('resource_calendar_id', '=', self.id)]})
        return action

```

## File: models\resource_calendar_leaves.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict
from datetime import datetime
from pytz import timezone, utc

from odoo import models


class ResourceCalendarLeaves(models.Model):
    _inherit = 'resource.calendar.leaves'

    def _compute_calendar_id(self):
        def date2datetime(date, tz):
            dt = datetime.fromordinal(date.toordinal())
            return tz.localize(dt).astimezone(utc).replace(tzinfo=None)

        contracts = self.env['hr.contract'].search([
            ('state', '=', 'open'),
            ('employee_id.resource_id', 'in', self.resource_id.ids),
        ])

        CalendarLeaves = self.env['resource.calendar.leaves']
        leaves_by_resource_id = defaultdict(lambda: CalendarLeaves, {False: CalendarLeaves})
        for leave in self:
            leaves_by_resource_id[leave.resource_id.id] += leave
        # pass leaves without resource_id to super
        remaining = leaves_by_resource_id.pop(False)

        for resource_id, leaves in leaves_by_resource_id.items():
            contract = contracts.filtered_domain([('employee_id.resource_id', '=', resource_id)])
            if not contract:
                remaining += leaves
                continue
            tz = timezone(contract.resource_calendar_id.tz or 'UTC')
            start_dt = date2datetime(contract.date_start, tz)
            end_dt = date2datetime(contract.date_end, tz) if contract.date_end else datetime.max
            # only modify leaves that fall under the active contract
            leaves.filtered(
                lambda leave: start_dt <= leave.date_from < end_dt
            ).calendar_id = contract.resource_calendar_id

        super(ResourceCalendarLeaves, remaining)._compute_calendar_id()

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

    @property
    def SELF_READABLE_FIELDS(self):
        return super().SELF_READABLE_FIELDS + ['vehicle', 'bank_account_id']

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_employee
from . import hr_employee_public
from . import hr_contract
from . import res_users
from . import resource
from . import resource_calendar_leaves
from . import hr_contract_type

```

## File: report\hr_contract_history.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, tools, _
from collections import defaultdict


class ContractHistory(models.Model):
    _name = 'hr.contract.history'
    _description = 'Contract history'
    _auto = False
    _order = 'is_under_contract'

    # Even though it would have been obvious to use the reference contract's id as the id of the
    # hr.contract.history model, it turned out it was a bad idea as this id could change (for instance if a
    # new contract is created with a later start date). The hr.contract.history is instead closely linked
    # to the employee. That's why we will use this id (employee_id) as the id of the hr.contract.history.
    contract_id = fields.Many2one('hr.contract', readonly=True)

    display_name = fields.Char(compute='_compute_display_name')
    name = fields.Char('Contract Name', readonly=True)
    date_hired = fields.Date('Hire Date', readonly=True)
    date_start = fields.Date('Start Date', readonly=True)
    date_end = fields.Date('End Date', readonly=True)
    employee_id = fields.Many2one('hr.employee', string='Employee', readonly=True)
    active_employee = fields.Boolean('Active Employee', readonly=True)
    is_under_contract = fields.Boolean('Is Currently Under Contract', readonly=True)
    department_id = fields.Many2one('hr.department', string='Department', readonly=True)
    structure_type_id = fields.Many2one('hr.payroll.structure.type', string='Salary Structure Type', readonly=True)
    hr_responsible_id = fields.Many2one('res.users', string='HR Responsible', readonly=True)
    job_id = fields.Many2one('hr.job', string='Job Position', readonly=True)
    state = fields.Selection([
        ('draft', 'New'),
        ('open', 'Running'),
        ('close', 'Expired'),
        ('cancel', 'Cancelled')
    ], string='Status', readonly=True)
    resource_calendar_id = fields.Many2one('resource.calendar', string="Working Schedule", readonly=True)
    wage = fields.Monetary('Wage', help="Employee's monthly gross wage.", readonly=True, group_operator="avg")
    company_id = fields.Many2one('res.company', string='Company', readonly=True)
    company_country_id = fields.Many2one('res.country', string="Company country", related='company_id.country_id', readonly=True)
    country_code = fields.Char(related='company_country_id.code', readonly=True)
    currency_id = fields.Many2one(string='Currency', related='company_id.currency_id', readonly=True)
    contract_type_id = fields.Many2one('hr.contract.type', 'Contract Type', readonly=True)
    contract_ids = fields.One2many('hr.contract', string='Contracts', compute='_compute_contract_ids', readonly=True)
    contract_count = fields.Integer(compute='_compute_contract_count', string="# Contracts")
    under_contract_state = fields.Selection([
        ('done', 'Under Contract'),
        ('blocked', 'Not Under Contract')
    ], string='Contractual Status', compute='_compute_under_contract_state')
    activity_state = fields.Selection(related='contract_id.activity_state')

    @api.depends('contract_ids')
    def _compute_contract_count(self):
        for history in self:
            history.contract_count = len(history.contract_ids)

    @api.depends('is_under_contract')
    def _compute_under_contract_state(self):
        for history in self:
            history.under_contract_state = 'done' if history.is_under_contract else 'blocked'

    @api.depends('employee_id.name')
    def _compute_display_name(self):
        for history in self:
            history.display_name = _("%s's Contracts History", history.employee_id.name)

    @api.model
    def _get_fields(self):
        return ','.join('contract.%s' % name for name, field in self._fields.items()
                        if field.store 
                        and field.type not in ['many2many', 'one2many', 'related']
                        and field.name not in ['id', 'contract_id', 'employee_id', 'date_hired', 'is_under_contract', 'active_employee'])

    def init(self):
        tools.drop_view_if_exists(self.env.cr, self._table)
        # Reference contract is the one with the latest start_date.
        self.env.cr.execute("""CREATE or REPLACE VIEW %s AS (
            WITH contract_information AS (
                SELECT DISTINCT employee_id,
                                company_id,
                                FIRST_VALUE(id) OVER w_partition AS id,
                                MAX(CASE
                                    WHEN state='open' THEN 1
                                    WHEN state='draft' AND kanban_state='done' THEN 1
                                    ELSE 0 END) OVER w_partition AS is_under_contract
                FROM   hr_contract AS contract
                WHERE  contract.state <> 'cancel'
                AND contract.active = true
                WINDOW w_partition AS (
                    PARTITION BY contract.employee_id, contract.company_id
                    ORDER BY
                        CASE
                            WHEN contract.state = 'open' THEN 0
                            WHEN contract.state = 'draft' THEN 1
                            WHEN contract.state = 'close' THEN 2
                            ELSE 3 END,
                        contract.date_start DESC
                    RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
                )
            )
            SELECT     employee.id AS id,
                       employee.id AS employee_id,
                       employee.active AS active_employee,
                       contract.id AS contract_id,
                       contract_information.is_under_contract::bool AS is_under_contract,
                       employee.first_contract_date AS date_hired,
                       %s
            FROM       hr_contract AS contract
            INNER JOIN contract_information ON contract.id = contract_information.id
            RIGHT JOIN hr_employee AS employee
                ON  contract_information.employee_id = employee.id
                AND contract.company_id = employee.company_id
            WHERE   employee.employee_type IN ('employee', 'student', 'trainee')
        )""" % (self._table, self._get_fields()))

    @api.depends('employee_id.contract_ids')
    def _compute_contract_ids(self):
        sorted_contracts = self.mapped('employee_id.contract_ids').sorted('date_start', reverse=True)

        mapped_employee_contracts = defaultdict(lambda: self.env['hr.contract'])
        for contract in sorted_contracts:
            if contract.state != 'cancel':
                mapped_employee_contracts[contract.employee_id] |= contract

        for history in self:
            history.contract_ids = mapped_employee_contracts[history.employee_id]

    def hr_contract_view_form_new_action(self):
        self.ensure_one()
        action = self.env['ir.actions.actions']._for_xml_id('hr_contract.action_hr_contract')
        action.update({
            'context': {'default_employee_id': self.employee_id.id},
            'view_mode': 'form',
            'view_id': self.env.ref('hr_contract.hr_contract_view_form').id,
            'views': [(self.env.ref('hr_contract.hr_contract_view_form').id, 'form')],
        })
        return action

```

## File: report\hr_contract_history_report_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hr_contract_history_view_search" model="ir.ui.view">
        <field name="name">hr.contract.history.search</field>
        <field name="model">hr.contract.history</field>
        <field name="arch" type="xml">
            <search string="Search Reference Contracts">
                <field name="name"/>
                <field name="employee_id"/>
                <field name="job_id"/>
                <field name="department_id" operator="child_of"/>
                <field name="resource_calendar_id"/>
                <field name="state"/>
                <field name="is_under_contract"/>
                <filter string="Running Contracts" name="open_contracts" domain="[('state', '=', 'open')]"/>
                <filter string="Contracts to Review" name="contract_to_review" domain="['|', ('state', 'in', ['draft', 'close', 'cancel']), ('is_under_contract', '!=', True)]"/>
                <filter string="No Contracts" name="no_contracts" domain="[('contract_id', '=', False)]"/>
                <filter string="Currently Under Contract" name="currently_under_contract" domain="[('is_under_contract', '=', True)]"/>
                <filter string="Active Employees" name="active_employees" domain="[('active_employee', '=', True)]"/>
                <group expand="0" string="Group By">
                    <filter string="Job Position" name="job" domain="[]" context="{'group_by': 'job_id'}"/>
                    <filter string="Status" name='group_by_state' domain="[]" context="{'group_by': 'state'}"/>
                    <filter string="Reference Working Time" name="group_by_resource_calendar_id" domain="[]" context="{'group_by': 'resource_calendar_id'}"/>
                    <filter string="Salary Structure Type" name="group_by_structure_type_id" domain="[]" context="{'group_by': 'structure_type_id'}"/>
                </group>
                <searchpanel>
                    <field name="company_id" icon="fa-building" enable_counters="1"/>
                    <field name="department_id" icon="fa-users" enable_counters="1"/>
                    <field name="state" icon="fa-tasks" enable_counters="1"/>
                </searchpanel>
            </search>
        </field>
    </record>
    <record id="hr_contract_history_view_form_action" model="ir.actions.act_window">
        <field name="name">Contracts</field>
        <field name="res_model">hr.contract.history</field>
        <field name="view_mode">form</field>
        <field name="context">{'search_default_active_employees': 1}</field>
    </record>
    <record id="hr_contract_history_view_form" model="ir.ui.view">
        <field name="name">hr.contract.history.form</field>
        <field name="model">hr.contract.history</field>
        <field name="arch" type="xml">
            <form string="Contract History"
                  create="false"
                  edit="false"
                  delete="false"
                  duplicate="false"
                  import="false">
                <header>
                    <button name="hr_contract_view_form_new_action" string="Create" type="object" groups="hr_contract.group_hr_contract_manager" class="btn-primary"/>
                </header>
                <sheet>
                    <div class="oe_button_box" name="button_box"/>
                    <h1>
                        <div class="d-flex justify-content-start">
                            <div>
                                <field name="display_name"/>
                            </div>
                            <div class="pl-3">
                                <field name="under_contract_state" widget="state_selection" readonly="1"/>
                            </div>
                        </div>
                    </h1>
                    <h2>
                        <field name="employee_id"/>
                    </h2>
                    <group>
                        <group>
                            <field name="contract_id" invisible="1"/>
                            <field name="company_country_id" invisible="1"/>
                            <field name="country_code" invisible="1"/>
                            <field name="structure_type_id"/>
                            <field name="resource_calendar_id"/>
                            <field name="currency_id" invisible="1"/>
                            <field name="wage" invisible="1"/>
                        </group>
                        <group>
                            <field name="department_id"/>
                            <field name="job_id"/>
                        </group>
                    </group>
                    <notebook>
                        <page string="Contract History" name="contract_history">
                            <field name="contract_ids" widget="one2many" readonly="0">
                                <tree string="Current Contracts"
                                      decoration-primary="state == 'open'"
                                      decoration-muted="state == 'close'"
                                      decoration-bf="id == parent.contract_id"
                                      default_order = "date_start desc, state desc"
                                      editable="bottom"
                                      no_open="1"
                                      create="0" delete="0">
                                    <button name="action_open_contract_form" type="object" icon="fa-external-link"/>
                                    <field name="id" invisible="1"/>
                                    <field name="name" string="Contract Name"/>
                                    <field name="date_start"/>
                                    <field name="date_end"/>
                                    <field name="resource_calendar_id"/>
                                    <field name="currency_id" invisible="1"/>
                                    <field name="wage" string="Monthly Wage"/>
                                    <field name="state" widget="badge" decoration-info="state == 'draft'" decoration-warning="state == 'close'" decoration-success="state == 'open'"/>
                                </tree>
                            </field>
                        </page>
                        <page string="Employee Information" name="contract_others">
                            <group>
                                <field name="date_hired"/>
                                <field name="hr_responsible_id"/>
                                <field name="company_id"/>
                            </group>
                        </page>
                    </notebook>
                </sheet>
            </form>
        </field>
    </record>
    <record id="hr_contract_history_view_list_action" model="ir.actions.act_window">
        <field name="name">Employees</field>
        <field name="res_model">hr.contract.history</field>
        <field name="view_mode">tree,kanban,form</field>
        <field name="search_view_id" ref="hr_contract_history_view_search"/>
        <field name="context">{'search_default_active_employees': 1}</field>
    </record>
    <record id="hr_contract_history_to_review_view_list_action" model="ir.actions.act_window">
        <field name="name">Contracts to Review</field>
        <field name="res_model">hr.contract.history</field>
        <field name="view_mode">tree,form</field>
        <field name="search_view_id" ref="hr_contract_history_view_search"/>
        <field name="context">
            {
                'search_default_to_review': 1,
                'search_default_active_employees': 1
            }
        </field>
    </record>
    <record id="hr_contract_history_view_list" model="ir.ui.view">
        <field name="name">hr.contract.history.list</field>
        <field name="model">hr.contract.history</field>
        <field name="arch" type="xml">
            <tree string="Contracts"
                  default_order = 'is_under_contract, date_start desc'
                  create="false"
                  edit="false"
                  delete="false"
                  duplicate="false"
                  import="false">
                <field name="employee_id" widget="many2one_avatar_employee"/>
                <field name="date_hired"/>
                <field name="is_under_contract" invisible="1"/>
                <field name="name"/>
                <field name="date_start"/>
                <field string="Reference Working Time" name="resource_calendar_id" optional="hide"/>
                <field name="under_contract_state" widget="state_selection" optional="hide"/>
                <field name="structure_type_id" optional="hide"/>
                <field name="currency_id" invisible="1"/>
                <field name="wage" optional="hide"/>
                <field name="state"
                       widget="badge"
                       decoration-info="state == 'draft'"
                       decoration-warning="state == 'close'"
                       decoration-success="state == 'open'"/>
                <field name="contract_count"/>
            </tree>
        </field>
    </record>
    <record id="hr_contract_history_view_kanban" model="ir.ui.view">
        <field name="name">hr.contract.history.view.kanban</field>
        <field name="model">hr.contract.history</field>
        <field name="arch" type="xml">
            <kanban class="o_kanban_small_column" default_order="date_end" sample="1" create="false">
                <field name="employee_id"/>
                <field name="activity_state"/>
                <field name="state"/>
                <progressbar field="activity_state" colors='{"planned": "success", "today": "warning", "overdue": "danger"}'/>
                <templates>
                    <t t-name="kanban-box">
                        <div class="oe_kanban_card oe_kanban_global_click">
                            <div class="oe_kanban_content">
                                <div class="o_hr_contract_state">
                                    <strong class="o_kanban_record_title">
                                        <field name="display_name"/>
                                    </strong>
                                </div>
                                <div class="text-muted o_kanban_record_subtitle o_hr_contract_job_id">
                                    <field name="job_id"/>
                                </div>
                                <div class="oe_kanban_bottom_right">
                                    <span class="float-right">
                                        <field name="employee_id" widget="many2one_avatar_employee"/>
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

    <menuitem
        id="hr_menu_contract_history"
        action="hr_contract_history_view_list_action"
        parent="hr.menu_hr_employee_payroll"
        name="Contracts"
        sequence="4"
        groups="hr_contract.group_hr_contract_manager"/>
</odoo>

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_contract_history

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_hr_resource_manager,hr.employee.resource.manager,resource.model_resource_resource,hr.group_hr_manager,1,1,1,1
access_hr_resource_calendar_user,hr.employee.resource.calendar.user,resource.model_resource_calendar,hr.group_hr_user,1,1,1,1
access_hr_resource_calendar_attendance_user,hr.employee.resource.calendar.attendance.user,resource.model_resource_calendar_attendance,hr.group_hr_user,1,1,1,1
access_hr_contract_manager,hr.contract.manager,model_hr_contract,hr_contract.group_hr_contract_manager,1,1,1,1
access_hr_contract_type_manager,hr.contract.type.manager,model_hr_contract_type,hr_contract.group_hr_contract_manager,1,1,1,1
access_hr_contract_history_manager,hr.contract.history.manager,model_hr_contract_history,hr_contract.group_hr_contract_manager,1,1,1,1
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

        <record id="ir_rule_hr_contract_history_multi_company" model="ir.rule">
            <field name="name">HR Contract History: Multi Company</field>
            <field name="model_id" ref="model_hr_contract_history"/>
            <field name="domain_force">['|', ('employee_id.company_id', '=', False), ('employee_id.company_id', 'in', company_ids)]</field>
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

## File: static\description\icon.svg

```svg
<svg id="Layer_1" data-name="Layer 1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 70 70">
  <defs>
    <mask id="mask" x="0" y="0" width="70" height="70" maskUnits="userSpaceOnUse">
      <g id="b">
        <path id="a" d="M4,0H65c4,0,5,1,5,5V65c0,4-1,5-5,5H4c-3,0-4-1-4-5V5C0,1,1,0,4,0Z" fill="#fff" fill-rule="evenodd"/>
      </g>
    </mask>
    <linearGradient id="linear-gradient" x1="-915.85" y1="568.8" x2="-916.85" y2="567.8" gradientTransform="matrix(70, 0, 0, -70, 64179.63, 39816)" gradientUnits="userSpaceOnUse">
      <stop offset="0" stop-color="#cdc484"/>
      <stop offset="1" stop-color="#b5aa59"/>
    </linearGradient>
  </defs>
  <g mask="url(#mask)">
    <g>
      <path d="M0,0H70V70H0Z" fill-rule="evenodd" fill="url(#linear-gradient)"/>
      <path d="M4,1H65c2.67,0,4.33.67,5,2V0H0V3C.67,1.67,2,1,4,1Z" fill="#fff" fill-opacity="0.38" fill-rule="evenodd"/>
      <path d="M4,69a3.66,3.66,0,0,1-4-4V32.67L19.61,15.56,29.94,25,28,32.67c1.92-.12,12.08-.62,19-.59,4.12,0,7.17,1.52,8.27,1.64V53.56h3L58,56,51.24,69Z" fill="#393939" fill-rule="evenodd" opacity="0.32" style="isolation: isolate"/>
      <path d="M4,69H65c2.67,0,4.33-1,5-3v4H0V66A3.92,3.92,0,0,0,4,69Z" fill-opacity="0.38" fill-rule="evenodd"/>
      <g opacity="0.4">
        <g>
          <path d="M49.53,37.69H44a.75.75,0,0,1,0-1.5h5.56a.75.75,0,0,1,0,1.5Z"/>
          <path d="M53.3,40.85H41.36a.75.75,0,0,1,0-1.5H53.3a.75.75,0,0,1,0,1.5Z"/>
          <path d="M53.3,47.2H41.36a.75.75,0,0,1,0-1.5H53.3a.75.75,0,0,1,0,1.5Z"/>
          <path d="M53.37,44H43.27a.75.75,0,0,1,0-1.5h10.1a.75.75,0,0,1,0,1.5Z"/>
          <path d="M41.27,44.05a.75.75,0,0,0,0-1.5.75.75,0,0,0,0,1.5Z"/>
          <path d="M58.8,54.43H57V36.8a3.49,3.49,0,0,0-3.35-3.62H36.26a.76.76,0,0,0-.21,0c-2.51.71-2.51,2.55-2.51,4.32v.59a.75.75,0,0,0,.22.53.76.76,0,0,0,.53.23h0l3.12,0V55.59a3.53,3.53,0,0,0,1.06,2.52A3.61,3.61,0,0,0,40,59h0a3.4,3.4,0,0,0,.84.11H56.28a.8.8,0,0,0,.26,0,3.53,3.53,0,0,0,3-3.47v-.41A.75.75,0,0,0,58.8,54.43ZM37.41,37.36H35c0-1.7.1-2.31,1.3-2.68h.08a1,1,0,0,1,.44.14,1.13,1.13,0,0,1,.55,1Zm1.5,18.23V35.8a2.63,2.63,0,0,0-.25-1.12h15a2,2,0,0,1,1.85,2.12V54.43H54.09l0-1.4a3.11,3.11,0,0,0,1-2.28,3.15,3.15,0,0,0-5.45-2.16H41.34a.75.75,0,0,0,0,1.5h7.52a2.84,2.84,0,0,0-.08.66,3,3,0,0,0,.1.73H41.34a.75.75,0,0,0,0,1.5h8.27v1.45h-7.4a.74.74,0,0,0-.54.23.73.73,0,0,0-.21.56c.09,1.56-.16,2.07-1.13,2.3a2,2,0,0,1-1.42-1.93Zm13-3.18a1.66,1.66,0,1,1,1.66-1.66A1.66,1.66,0,0,1,51.94,52.41Zm0,1.5a3.51,3.51,0,0,0,.66-.07l0,1.38-.15-.13a.74.74,0,0,0-.89,0l-.42.27V53.78A2.88,2.88,0,0,0,51.94,53.91ZM56,57.61l-13.39,0A3.9,3.9,0,0,0,43,55.93h6.63v.79a.75.75,0,0,0,.4.66.74.74,0,0,0,.77,0l1.12-.74.92.75a.78.78,0,0,0,.48.17.7.7,0,0,0,.31-.07.77.77,0,0,0,.44-.67v-.85h4A2,2,0,0,1,56,57.61Z"/>
        </g>
        <g>
          <path d="M34.3,40.88a2.77,2.77,0,0,1-2.76-2.79v-.57a7.54,7.54,0,0,1,.86-4.05L29,32.63a11,11,0,0,1-12.74,0l-4.06,1a5.48,5.48,0,0,0-4.14,5.31v2.12a2.73,2.73,0,0,0,2.73,2.73H34.47a2.66,2.66,0,0,0,.94-.18V40.87Z"/>
          <path d="M22.63,14.62h0a9.12,9.12,0,0,0,0,18.23h.19a9.12,9.12,0,0,0-.19-18.23Z"/>
        </g>
      </g>
      <g>
        <g>
          <path d="M49.53,36.07H44a.75.75,0,0,1,0-1.5h5.56a.75.75,0,0,1,0,1.5Z" fill="#fff"/>
          <path d="M53.3,39.23H41.36a.75.75,0,0,1,0-1.5H53.3a.75.75,0,0,1,0,1.5Z" fill="#fff"/>
          <path d="M53.3,45.58H41.36a.75.75,0,0,1,0-1.5H53.3a.75.75,0,0,1,0,1.5Z" fill="#fff"/>
          <path d="M53.37,42.41H43.27a.75.75,0,0,1,0-1.5h10.1a.75.75,0,1,1,0,1.5Z" fill="#fff"/>
          <path d="M41.27,42.43a.75.75,0,0,0,0-1.5.75.75,0,0,0,0,1.5Z" fill="#fff"/>
          <path d="M58.8,52.81H57V35.19a3.5,3.5,0,0,0-3.35-3.63H36.26a.76.76,0,0,0-.21,0c-2.51.72-2.51,2.55-2.51,4.32v.59a.79.79,0,0,0,.22.54.75.75,0,0,0,.53.22h3.12V54a3.51,3.51,0,0,0,1.06,2.52,3.42,3.42,0,0,0,1.57.88h0a3.41,3.41,0,0,0,.84.12H56.28a.82.82,0,0,0,.26-.06,3.52,3.52,0,0,0,3-3.47v-.41A.76.76,0,0,0,58.8,52.81ZM37.41,35.75H35c0-1.7.1-2.31,1.3-2.69h.08a1.1,1.1,0,0,1,.44.15,1.13,1.13,0,0,1,.55,1ZM38.91,54V34.18a2.63,2.63,0,0,0-.25-1.12h15a2,2,0,0,1,1.85,2.13V52.81H54.09l0-1.39A3.15,3.15,0,1,0,49.65,47H41.34a.75.75,0,0,0,0,1.5h7.52a3,3,0,0,0-.08.66,2.86,2.86,0,0,0,.1.73H41.34a.75.75,0,0,0,0,1.5h8.27v1.45h-7.4a.74.74,0,0,0-.54.23.73.73,0,0,0-.21.56c.09,1.56-.16,2.07-1.13,2.31A2,2,0,0,1,38.91,54Zm13-3.18a1.66,1.66,0,1,1,1.66-1.66A1.66,1.66,0,0,1,51.94,50.79Zm0,1.5a2.84,2.84,0,0,0,.66-.07l0,1.38-.15-.13a.77.77,0,0,0-.89,0l-.42.28V52.17A3.26,3.26,0,0,0,51.94,52.29ZM56,56l-13.39-.06A3.9,3.9,0,0,0,43,54.31h6.63v.79a.76.76,0,0,0,1.17.63L51.9,55l.92.75a.77.77,0,0,0,.48.16.7.7,0,0,0,.31-.07.76.76,0,0,0,.44-.66v-.86h4A2,2,0,0,1,56,56Z" fill="#fff"/>
        </g>
        <g>
          <path d="M34.3,39.26a2.75,2.75,0,0,1-2.76-2.79V35.9a7.54,7.54,0,0,1,.86-4L29,31a11,11,0,0,1-12.74,0L12.2,32a5.48,5.48,0,0,0-4.14,5.31v2.12a2.73,2.73,0,0,0,2.73,2.73H34.47a2.66,2.66,0,0,0,.94-.17V39.25Z" fill="#fff"/>
          <path d="M22.63,13h0a9.12,9.12,0,0,0,0,18.23h.19A9.12,9.12,0,0,0,22.63,13Z" fill="#fff"/>
        </g>
      </g>
    </g>
  </g>
</svg>

```

## File: views\hr_contract_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

        <record id="hr_hr_employee_view_form2" model="ir.ui.view">
            <field name="name">hr.hr.employee.view.form2</field>
            <field name="model">hr.employee</field>
            <field name="inherit_id" ref="hr.view_employee_form"/>
            <field name="arch" type="xml">
                <data>
                    <div name="button_box" position="inside">
                        <field name="contract_warning" invisible="1"/>
                        <button name="action_open_contract_history"
                            class="oe_stat_button"
                            icon="fa-book"
                            type="object"
                            groups="hr_contract.group_hr_contract_manager"
                            attrs="{'invisible' : [('employee_type', 'not in', ['employee', 'student', 'trainee'])]}">
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
                        <field name="first_contract_date"
                               attrs="{'invisible' : ['|', ('employee_type', 'not in', ['employee', 'student']), ('first_contract_date', '=', False)]}"
                               readonly="1"/>
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
                    <field name="job_id" position="before">
                        <field name="contract_id" groups="hr_contract.group_hr_contract_manager"/>
                    </field>
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
                    <filter string="Not Running" name="not_running" domain="[('state', '!=', 'open')]"/>
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
                        <filter string="Status" name="group_by_state" domain="[]" context="{'group_by': 'state'}"/>
                        <filter string="Employee" name="group_by_employee" domain="[]" context="{'group_by': 'employee_id'}"/>
                        <filter string="Start Date" name="group_by_date_start" domain="[]" context="{'group_by': 'date_start'}"/>
                        <filter string="Job Position" name="group_by_job" domain="[]" context="{'group_by': 'job_id'}"/>
                        <filter string="Working Schedule" name="group_by_resource_calendar_id" domain="[]" context="{'group_by': 'resource_calendar_id'}"/>
                        <filter string="Salary Structure Type" name="group_by_structure_type_id" domain="[]" context="{'group_by': 'structure_type_id'}"/>
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
                            <h2>
                                <field name="company_id" groups="base.group_multi_company" invisible="1"/>
                            </h2>
                        </div>
                        <group name="top_info">
                            <group name="top_info_left">
                                <field name="active" invisible="1"/>
                                <field name="employee_id"/>
                                <field name="date_start" string="Contract Start Date"/>
                                <field name="date_end" string="Contract End Date"/>
                                <field name="company_country_id" invisible="1"/>
                                <field name="country_code" invisible="1"/>
                                <field name="structure_type_id" domain="['|', ('country_id', '=', False), ('country_id', '=', company_country_id)]"/>
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
                            </group>
                            <group name="top_info_right">
                                <field name="department_id"/>
                                <field name="job_id"/>
                                <field name="contract_type_id"/>
                                <field name="hr_responsible_id" required="1"/>
                            </group>
                        </group>
                        <notebook>
                            <page string="Contract Details" name="other">
                                <group><group name="contract_details"/></group>
                                <group name="notes_group" string="Notes">
                                    <field name="notes" nolabel="1"/>
                                </group>
                            </page>
                            <page string="Salary Information" name="information">
                                <group name="salary_info">
                                    <group name="salary">
                                        <label for="wage"/>
                                        <div class="o_row" name="wage">
                                            <field name="wage" nolabel="1"/>
                                            <span>/ month</span>
                                        </div>
                                    </group>
                                    <group name="yearly_advantages"/>
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
                <tree string="Contracts" multi_edit="1" sample="1" default_order='date_start ASC'>
                    <field name="name" readonly="1"/>
                    <field name="employee_id" readonly="1" widget="many2one_avatar_employee"/>
                    <field name="job_id"/>
                    <field name="date_start" readonly="1"/>
                    <field name="date_end" readonly="1"/>
                    <field name="resource_calendar_id" optional="show"/>
                    <field name="structure_type_id" optional="show"/>
                    <field name="state" widget="badge" decoration-info="state == 'draft'" decoration-warning="state == 'close'" decoration-success="state == 'open'"/>
                    <field name="wage" invisible="1"/>
                    <field name="message_needaction" invisible="1"/>
                    <field name="company_id" groups="base.group_multi_company" readonly="1" optional="show"/>
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
                                <div class="text-muted o_kanban_record_subtitle o_hr_contract_job_id" name="div_job_id">
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
                            <img t-att-src="activity_image('hr.employee', 'avatar_128', record.employee_id.raw_value)" t-att-title="record.employee_id.value" t-att-alt="record.employee_id.value"/>
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
            <field name="context">{'search_default_group_by_state': 1}</field>
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

</odoo>

```

## File: views\hr_employee_public_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hr_employee_public_view_form" model="ir.ui.view">
        <field name="name">hr.employee.public.form</field>
        <field name="model">hr.employee.public</field>
        <field name="inherit_id" ref="hr.hr_employee_public_view_form"/>
        <field name="arch" type="xml">
            <field name="employee_type" position="replace"/>
            <field name="user_id" position="replace"/>
            <group name="location" position="after">
                <group string="Status" name="status">
                    <field name="employee_type"/>
                    <field name="first_contract_date"/>
                    <field name="user_id" string="Related User"/>
                </group>
            </group>
        </field>
    </record>
</odoo>

```

## File: views\hr_employee_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_employee_tree" model="ir.ui.view">
        <field name="name">hr.employee.tree</field>
        <field name="model">hr.employee</field>
        <field name="inherit_id" ref="hr.view_employee_tree"></field>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='work_email']" position="after">
                <field name="first_contract_date" optional="hide"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\resource_calendar_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="resource_calendar_view_tree" model="ir.ui.view">
        <field name="name">resource.calendar.view.tree.inherit.hr.contract</field>
        <field name="model">resource.calendar</field>
        <field name="inherit_id" ref="resource.view_resource_calendar_tree"/>
        <field name="arch" type="xml">
            <field name="company_id" position="after">
                <field name="contracts_count"/>
            </field>
        </field>
    </record>

    <record id="resource_calendar_view_form" model="ir.ui.view">
        <field name="name">resource.calendar.view.form.inherit.hr.contract</field>
        <field name="model">resource.calendar</field>
        <field name="inherit_id" ref="resource.resource_calendar_form"/>
        <field name="arch" type="xml">
            <div name="button_box" position="inside">
                <button class="oe_stat_button" name="action_open_contracts"
                        type="object" icon="fa-book" groups="hr_contract.group_hr_contract_manager">
                    <field name="contracts_count" string="Contracts" widget="statinfo"/>
                </button>
            </div>
        </field>
    </record>
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

    set_date_end = fields.Boolean(string="Set Contract End Date", default=True,
        help="Set the end date on the current contract.")

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
            <xpath expr="//div[@id='activities_label']" position="attributes">
                <attribute name="invisible">0</attribute>
            </xpath>
            <xpath expr="//div[@id='activities']" position="attributes">
                <attribute name="invisible">0</attribute>
            </xpath>
            <xpath expr="//field[@name='departure_date']" position="replace">
                <field name="departure_date" string="Contract End Date"/>
            </xpath>
            <xpath expr="//div[@id='activities']" position="inside">
                <div><field name="set_date_end"/><label for="set_date_end" string="Contract"/></div>
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

