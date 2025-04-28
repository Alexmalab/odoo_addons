# Odoo Module: project_purchase

Category: Services/Project

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
    'name': "Project Purchase",
    'version': '1.0',
    'summary': "Monitor purchase in project",
    'description': "",
    'category': 'Services/Project',
    'depends': ['purchase', 'project'],
    'data': [
        'views/project_views.xml',
    ],
    'application': False,
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
    _inherit = "project.project"

    purchase_orders_count = fields.Integer('# Purchase Orders', compute='_compute_purchase_orders_count', groups='purchase.group_purchase_user')

    @api.depends('analytic_account_id')
    def _compute_purchase_orders_count(self):
        purchase_orders_data = self.env['purchase.order.line'].read_group([
            ('account_analytic_id', '!=', False),
            ('account_analytic_id', 'in', self.analytic_account_id.ids)
        ], ['account_analytic_id', 'order_id:count_distinct'], ['account_analytic_id'])
        mapped_data = dict([(data['account_analytic_id'][0], data['order_id']) for data in purchase_orders_data])
        for project in self:
            project.purchase_orders_count = mapped_data.get(project.analytic_account_id.id, 0)

    # ----------------------------
    #  Actions
    # ----------------------------

    def action_open_project_purchase_orders(self):
        purchase_orders = self.env['purchase.order'].search([
            ('order_line.account_analytic_id', '!=', False),
            ('order_line.account_analytic_id', 'in', self.analytic_account_id.ids)
        ])
        action_window = {
            'name': _('Purchase Orders'),
            'type': 'ir.actions.act_window',
            'res_model': 'purchase.order',
            'views': [[False, 'tree'], [False, 'form']],
            'domain': [('id', 'in', purchase_orders.ids)],
            'context': {
                'create': False,
            }
        }
        if len(purchase_orders) == 1:
            action_window['views'] = [[False, 'form']]
            action_window['res_id'] = purchase_orders.id
        return action_window

    # ----------------------------
    #  Project Updates
    # ----------------------------

    def _get_stat_buttons(self):
        buttons = super(Project, self)._get_stat_buttons()
        if self.user_has_groups('purchase.group_purchase_user'):
            buttons.append({
                'icon': 'credit-card',
                'text': _lt('Purchase Orders'),
                'number': self.purchase_orders_count,
                'action_type': 'object',
                'action': 'action_open_project_purchase_orders',
                'show': self.purchase_orders_count > 0,
                'sequence': 13,
            })
        return buttons

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import project

```

## File: views\project_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="project_project_form_view_inherited" model="ir.ui.view">
        <field name="name">project.project.view.inherited</field>
        <field name="model">project.project</field>
        <field name="inherit_id" ref="project.edit_project"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@name='%(project.action_project_task_burndown_chart_report)d']" position="after">
                <button name="action_open_project_purchase_orders" type="object" class="oe_stat_button" icon="fa-credit-card" attrs="{'invisible': [('purchase_orders_count', '=', 0)]}" groups="purchase.group_purchase_user">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_value">
                            <field name="purchase_orders_count" nolabel="1"/>
                        </span>
                        <span class="o_stat_text">
                            Purchase Orders
                        </span>
                    </div>
                </button>
            </xpath>
        </field>
    </record>
</odoo>

```

