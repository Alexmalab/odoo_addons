# Odoo Module: project_mrp

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
    'name': "MRP Project",
    'version': '1.0',
    'summary': "Monitor MRP using project",
    'description': "",
    'category': 'Services/Project',
    'depends': ['mrp_account', 'project'],
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

from odoo import fields, models, _, _lt


class Project(models.Model):
    _inherit = "project.project"

    production_count = fields.Integer(related="analytic_account_id.production_count", groups='mrp.group_mrp_user')
    workorder_count = fields.Integer(related="analytic_account_id.workorder_count", groups='mrp.group_mrp_user')
    bom_count = fields.Integer(related="analytic_account_id.bom_count", groups='mrp.group_mrp_user')

    def action_view_mrp_production(self):
        self.ensure_one()
        action = self.analytic_account_id.action_view_mrp_production()
        if self.production_count > 1:
            action["view_mode"] = 'tree,form,kanban,calendar,pivot,graph'
        return action

    def action_view_mrp_bom(self):
        self.ensure_one()
        action = self.analytic_account_id.action_view_mrp_bom()
        if self.bom_count > 1:
            action['view_mode'] = 'tree,form,kanban'
        return action

    def action_view_workorder(self):
        self.ensure_one()
        action = self.analytic_account_id.action_view_workorder()
        if self.workorder_count > 1:
            action['view_mode'] = 'tree,form,kanban,calendar,pivot,graph'
        return action

    # ----------------------------
    #  Project Updates
    # ----------------------------

    def _get_stat_buttons(self):
        buttons = super(Project, self)._get_stat_buttons()
        if self.user_has_groups('mrp.group_mrp_user'):
            buttons.extend([{
                'icon': 'wrench',
                'text': _lt('Manufacturing Orders'),
                'number': self.production_count,
                'action_type': 'object',
                'action': 'action_view_mrp_production',
                'show': self.production_count > 0,
                'sequence': 19,
            },
            {
                'icon': 'cog',
                'text': _lt('Work Orders'),
                'number': self.workorder_count,
                'action_type': 'object',
                'action': 'action_view_workorder',
                'show': self.workorder_count > 0,
                'sequence': 20,
            },
            {
                'icon': 'flask',
                'text': _lt('Bills of Materials'),
                'number': self.bom_count,
                'action_type': 'object',
                'action': 'action_view_mrp_bom',
                'show': self.bom_count > 0,
                'sequence': 21,
            }])
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
        <field name="inherit_id" ref="project.edit_project" />
        <field eval="14" name="priority"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@name='action_view_analytic_account_entries']" position="after">
                <button class="oe_stat_button" type="object" name="action_view_mrp_production"
                    icon="fa-wrench" attrs="{'invisible': [('production_count', '=', 0)]}"
                    groups="mrp.group_mrp_user">
                    <field string="Manufacturing Orders" name="production_count" widget="statinfo"/>
                </button>
                <button class="oe_stat_button" type="object" name="action_view_workorder"
                    icon="fa-cog" attrs="{'invisible': [('workorder_count', '=', 0)]}"
                    groups="mrp.group_mrp_user">
                    <field string="Work Orders" name="workorder_count" widget="statinfo"/>
                </button>
                <button class="oe_stat_button" type="object" name="action_view_mrp_bom"
                    icon="fa-flask" attrs="{'invisible': [('bom_count', '=', 0)]}"
                    groups="mrp.group_mrp_user">
                    <field string="Bills of Materials" name="bom_count" widget="statinfo"/>
                </button>
            </xpath>
        </field>
    </record>
</odoo>

```

