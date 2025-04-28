# Odoo Module: project_hr_expense

Category: Services/expenses

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
    'name': 'Project Expenses',
    'version': '1.0',
    'category': 'Services/expenses',
    'summary': 'Project expenses',
    'description': 'Bridge created to add the number of expenses linked to an AA to a project form',
    'depends': ['project_account', 'hr_expense'],
    'data': [
        'views/project_project_views.xml',
    ],
    'demo': [
        'data/project_hr_expense_demo.xml',
    ],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\project_hr_expense_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- Expense Sheet -->
        <record id="transportation_expense_sheet" model="hr.expense.sheet">
            <field name="name">Transportation Expense</field>
            <field name="employee_id" ref="hr.employee_al"/>
        </record>
        <record id="restaurant_expense_sheet" model="hr.expense.sheet">
            <field name="name">Restaurant Expense</field>
            <field name="employee_id" ref="hr.employee_al"/>
        </record>

        <record id="hr_expense.travel_admin_by_car_expense" model="hr.expense">
            <field name="analytic_distribution" eval="{ref('project.analytic_office_design'): 100}"/>
        </record>
        <record id="hr_expense.travel_demo_by_car_expense" model="hr.expense">
            <field name="analytic_distribution" eval="{ref('project.analytic_office_design'): 100}"/>
        </record>
        <record id="transportation_expense" model="hr.expense">
            <field name="name">Transportation</field>
            <field name="employee_id" ref="hr.employee_al"/>
            <field name="analytic_distribution" eval="{ref('project.analytic_construction'): 100}"/>
            <field name="product_id" ref="hr_expense.expense_product_travel_accommodation"/>
            <field name="total_amount_currency" eval="240.0"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="date" eval="time.strftime('%Y-%m-12')"/>
            <field name="sheet_id" ref="transportation_expense_sheet"/>
        </record>
        <record id="restaurant_expense" model="hr.expense">
            <field name="name">Restaurant</field>
            <field name="employee_id" ref="hr.employee_al"/>
            <field name="analytic_distribution" eval="{ref('project.analytic_construction'): 100}"/>
            <field name="product_id" ref="hr_expense.expense_product_meal"/>
            <field name="total_amount_currency" eval="320.0"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="date" eval="time.strftime('%Y-%m-10')"/>
            <field name="sheet_id" ref="restaurant_expense_sheet"/>
        </record>

        <function name="action_submit_sheet" model="hr.expense.sheet">
            <value model="hr.expense.sheet" eval="[ref('transportation_expense_sheet'), ref('restaurant_expense_sheet')]"/>
        </function>

        <function name="action_approve_expense_sheets" model="hr.expense.sheet">
            <value model="hr.expense.sheet" eval="[ref('transportation_expense_sheet'), ref('restaurant_expense_sheet')]"/>
        </function>
    </data>
</odoo>

```

## File: models\hr_expense.py

```python
from odoo import api, models


class HrExpense(models.Model):
    _inherit = 'hr.expense'

    def _compute_analytic_distribution(self):
        project_id = self.env.context.get('project_id')
        if not project_id:
            super()._compute_analytic_distribution()
        else:
            analytic_distribution = self.env['project.project'].browse(project_id)._get_analytic_distribution()
            for expense in self:
                expense.analytic_distribution = expense.analytic_distribution or analytic_distribution

    @api.model_create_multi
    def create(self, vals_list):
        project_id = self.env.context.get('project_id')
        if project_id:
            analytic_distribution = self.env['project.project'].browse(project_id)._get_analytic_distribution()
            if analytic_distribution:
                for vals in vals_list:
                    vals['analytic_distribution'] = vals.get('analytic_distribution', analytic_distribution)
        return super().create(vals_list)

```

## File: models\project_project.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json

from odoo import models
from odoo.osv import expression


class Project(models.Model):
    _inherit = 'project.project'

    # ----------------------------
    #  Actions
    # ----------------------------

    def _get_expense_action(self, domain=None, expense_ids=None):
        if not domain and not expense_ids:
            return {}
        action = self.env["ir.actions.actions"]._for_xml_id("hr_expense.hr_expense_actions_all")
        action.update({
            'display_name': self.env._('Expenses'),
            'views': [[False, 'list'], [False, 'form'], [False, 'kanban'], [False, 'graph'], [False, 'pivot']],
            'context': {'project_id': self.id},
            'domain': domain or [('id', 'in', expense_ids)],
        })
        if not self.env.context.get('from_embedded_action') and len(expense_ids) == 1:
            action["views"] = [[False, 'form']]
            action["res_id"] = expense_ids[0]
        return action

    def _get_add_purchase_items_domain(self):
        return expression.AND([
            super()._get_add_purchase_items_domain(),
            [('expense_id', '=', False)],
        ])

    def action_profitability_items(self, section_name, domain=None, res_id=False):
        if section_name == 'expenses':
            return self._get_expense_action(domain, [res_id] if res_id else [])
        return super().action_profitability_items(section_name, domain, res_id)

    def action_open_project_expenses(self):
        self.ensure_one()
        return self._get_expense_action(domain=[('analytic_distribution', 'in', self.account_id.ids)])

    # ----------------------------
    #  Project Update
    # ----------------------------

    def _get_profitability_labels(self):
        labels = super()._get_profitability_labels()
        labels['expenses'] = self.env._('Expenses')
        return labels

    def _get_profitability_sequence_per_invoice_type(self):
        sequence_per_invoice_type = super()._get_profitability_sequence_per_invoice_type()
        sequence_per_invoice_type['expenses'] = 13
        return sequence_per_invoice_type

    def _get_already_included_profitability_invoice_line_ids(self):
        # As both purchase orders and expenses (paid by employee) create vendor bills,
        # we need to make sure they are exclusive in the profitability report.
        move_line_ids = super()._get_already_included_profitability_invoice_line_ids()
        query = self.env['account.move.line'].sudo()._search([
            ('move_id.expense_sheet_id', '!=', False),
            ('id', 'not in', move_line_ids),
        ])
        return move_line_ids + list(query)

    def _get_expenses_profitability_items(self, with_action=True):
        if not self.account_id:
            return {}
        can_see_expense = with_action and self.env.user.has_group('hr_expense.group_hr_expense_team_approver')

        expenses_read_group = self.env['hr.expense']._read_group(
            [
                ('sheet_id.state', 'in', ['post', 'done']),
                ('analytic_distribution', 'in', self.account_id.ids),
            ],
            groupby=['currency_id'],
            aggregates=['id:array_agg', 'untaxed_amount_currency:sum'],
        )
        if not expenses_read_group:
            return {}
        expense_ids = []
        amount_billed = 0.0
        for currency, ids, untaxed_amount_currency_sum in expenses_read_group:
            if can_see_expense:
                expense_ids.extend(ids)
            amount_billed += currency._convert(
                from_amount=untaxed_amount_currency_sum,
                to_currency=self.currency_id,
                company=self.company_id,
            )

        section_id = 'expenses'
        expense_profitability_items = {
            'costs': {'id': section_id, 'sequence': self._get_profitability_sequence_per_invoice_type()[section_id], 'billed': -amount_billed, 'to_bill': 0.0},
        }
        if can_see_expense:
            args = [section_id, [('id', 'in', expense_ids)]]
            if len(expense_ids) == 1:
                args.append(expense_ids[0])
            action = {'name': 'action_profitability_items', 'type': 'object', 'args': json.dumps(args)}
            expense_profitability_items['costs']['action'] = action
        return expense_profitability_items

    def _get_profitability_aal_domain(self):
        return expression.AND([
            super()._get_profitability_aal_domain(),
            ['|', ('move_line_id', '=', False), ('move_line_id.expense_id', '=', False)],
        ])

    def _get_profitability_items(self, with_action=True):
        profitability_data = super()._get_profitability_items(with_action)
        expenses_data = self._get_expenses_profitability_items(with_action)
        if expenses_data:
            if 'revenues' in expenses_data:
                revenues = profitability_data['revenues']
                revenues['data'].append(expenses_data['revenues'])
                revenues['total'] = {k: revenues['total'][k] + expenses_data['revenues'][k] for k in ['invoiced', 'to_invoice']}
            costs = profitability_data['costs']
            costs['data'].append(expenses_data['costs'])
            costs['total'] = {k: costs['total'][k] + expenses_data['costs'][k] for k in ['billed', 'to_bill']}
        return profitability_data

```

## File: models\__init__.py

```python
from . import hr_expense
from . import project_project

```

## File: views\project_project_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="project_embedded_action_hr_expenses" model="ir.embedded.actions">
        <field name="parent_res_model">project.project</field>
        <field name="sequence">77</field>
        <field name="name">Expenses</field>
        <field name="parent_action_id" ref="project.act_project_project_2_project_task_all"/>
        <field name="python_method">action_open_project_expenses</field>
        <field name="context">{"from_embedded_action": true}</field>
        <field name="groups_ids" eval="[(4, ref('hr_expense.group_hr_expense_user'))]"/>
    </record>
</odoo>

```

