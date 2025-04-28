# Odoo Module: sale_project_account

Category: Services/account

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
    'name': 'Project Sales Accounting',
    'version': '1.0',
    'category': 'Services/account',
    'summary': 'Project sales accounting',
    'description': 'Bridge created to add the number of vendor bills linked to an AA to a project form',
    'depends': ['sale_timesheet', 'account'],
    'data': [
        'views/project_project_views.xml',
    ],
    'demo': [],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-

from odoo import models


class AccountMoveLine(models.Model):
    _inherit = 'account.move.line'

    def _compute_analytic_account_id(self):
        # when a project creates an aml, it adds an analytic account to it. the following filter is to save this
        # analytic account from being overridden by analytic default rules and lack thereof
        project_amls = self.filtered(lambda aml: aml.analytic_account_id and any(aml.sale_line_ids.project_id))
        super(AccountMoveLine, self - project_amls)._compute_analytic_account_id()

```

## File: models\project.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _, _lt

class Project(models.Model):
    _inherit = 'project.project'

    vendor_bill_count = fields.Integer(related='analytic_account_id.vendor_bill_count', groups='account.group_account_readonly')

    # ----------------------------
    # Actions
    # ----------------------------

    def action_open_project_vendor_bills(self):
        purchase_types = self.env['account.move'].get_purchase_types()
        vendor_bills = self.env['account.move'].search([
            ('line_ids.analytic_account_id', '!=', False),
            ('line_ids.analytic_account_id', 'in', self.analytic_account_id.ids),
            ('move_type', 'in', purchase_types),
        ])
        action_window = {
            'name': _('Vendor Bills'),
            'type': 'ir.actions.act_window',
            'res_model': 'account.move',
            'views': [[False, 'tree'], [False, 'form'], [False, 'kanban']],
            'domain': [('id', 'in', vendor_bills.ids)],
            'context': {
                'create': False,
            }
        }
        if len(vendor_bills) == 1:
            action_window['views'] = [[False, 'form']]
            action_window['res_id'] = vendor_bills.id
        return action_window

    # ----------------------------
    #  Project Updates
    # ----------------------------

    def _get_stat_buttons(self):
        buttons = super(Project, self)._get_stat_buttons()
        if self.user_has_groups('account.group_account_readonly'):
            buttons.append({
                'icon': 'pencil-square-o',
                'text': _lt('Vendor Bills'),
                'number': self.vendor_bill_count,
                'action_type': 'object',
                'action': 'action_open_project_vendor_bills',
                'show': self.vendor_bill_count > 0,
                'sequence': 14,
            })
        return buttons

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import project
from . import account_move

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
                <button name="action_open_project_vendor_bills" type="object" class="oe_stat_button" icon="fa-pencil-square-o" attrs="{'invisible': [('vendor_bill_count', '=', 0)]}" groups="account.group_account_readonly">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_value">
                            <field name="vendor_bill_count" nolabel="1"/>
                        </span>
                        <span class="o_stat_text">
                            Vendor Bills
                        </span>
                    </div>
                </button>
            </xpath>
        </field>
    </record>

</odoo>

```

