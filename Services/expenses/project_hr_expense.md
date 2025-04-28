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
    'depends': ['project', 'hr_expense'],
    'data': [
        'views/project_project_views.xml',
    ],
    'demo': [],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\project.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _, _lt

class Project(models.Model):
    _inherit = 'project.project'

    expenses_count = fields.Integer('# Expenses', compute='_compute_expenses_count', groups='hr_expense.group_hr_expense_team_approver')

    @api.depends('analytic_account_id')
    def _compute_expenses_count(self):
        expenses_data = self.env['hr.expense'].read_group([
            ('analytic_account_id', '!=', False),
            ('analytic_account_id', 'in', self.analytic_account_id.ids)
        ],
        ['analytic_account_id'], ['analytic_account_id'])
        mapped_data = {data['analytic_account_id'][0]: data['analytic_account_id_count'] for data in expenses_data}
        for project in self:
            project.expenses_count = mapped_data.get(project.analytic_account_id.id, 0)

    # ----------------------------
    #  Actions
    # ----------------------------

    def action_open_project_expenses(self):
        expenses = self.env['hr.expense'].search([
            ('analytic_account_id', '!=', False),
            ('analytic_account_id', 'in', self.analytic_account_id.ids)
        ])
        action = self.env["ir.actions.actions"]._for_xml_id("hr_expense.hr_expense_actions_all")
        action.update({
            'display_name': _('Expenses'),
            'views': [[False, 'tree'], [False, 'form'], [False, 'kanban'], [False, 'graph'], [False, 'pivot']],
            'context': {'default_analytic_account_id': self.analytic_account_id.id},
            'domain': [('id', 'in', expenses.ids)]
        })
        if(len(expenses) == 1):
            action["views"] = [[False, 'form']]
            action["res_id"] = expenses.id
        return action

    # ----------------------------
    #  Project Updates
    # ----------------------------

    def _get_stat_buttons(self):
        buttons = super(Project, self)._get_stat_buttons()
        if self.user_has_groups('hr_expense.group_hr_expense_team_approver'):
            buttons.append({
                'icon': 'money',
                'text': _lt('Expenses'),
                'number': self.expenses_count,
                'action_type': 'object',
                'action': 'action_open_project_expenses',
                'show': self.expenses_count > 0,
                'sequence': 10,
            })
        return buttons

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import project

```

## File: views\project_project_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="project_project_view_form" model="ir.ui.view">
        <field name="name">project.project.form.inherit</field>
        <field name="model">project.project</field>
        <field name="inherit_id" ref="project.edit_project"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@name='%(project.action_project_task_burndown_chart_report)d']" position="after">
                <button name="action_open_project_expenses" type="object" class="oe_stat_button" icon="fa-money" attrs="{'invisible': [('expenses_count', '=', 0)]}" groups="hr_expense.group_hr_expense_team_approver">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_value">
                            <field name="expenses_count" nolabel="1"/>
                        </span>
                        <span class="o_stat_text">
                            Expenses
                        </span>
                    </div>
                </button>
            </xpath>
        </field>
    </record>

</odoo>

```

