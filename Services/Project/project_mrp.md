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
    'category': 'Services/Project',
    'depends': ['mrp', 'project'],
    'data': [
        'views/mrp_bom_views.xml',
        'views/mrp_production_views.xml',
        'views/project_project_views.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\mrp_bom.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class MrpBom(models.Model):
    _inherit = 'mrp.bom'

    project_id = fields.Many2one('project.project')

```

## File: models\mrp_production.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class MrpProduction(models.Model):
    _inherit = 'mrp.production'

    project_id = fields.Many2one('project.project', compute='_compute_project_id', readonly=False, store=True)

    @api.depends('bom_id')
    def _compute_project_id(self):
        if not self.env.context.get('from_project_action'):
            for production in self:
                production.project_id = production.bom_id.project_id

    def action_generate_bom(self):
        action = super().action_generate_bom()
        action['context']['default_project_id'] = self.project_id.id
        return action

```

## File: models\project_project.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _


class ProjectProject(models.Model):
    _inherit = 'project.project'

    bom_count = fields.Integer(compute='_compute_bom_count', groups='mrp.group_mrp_user', export_string_translation=False)
    production_count = fields.Integer(compute='_compute_production_count', groups='mrp.group_mrp_user', export_string_translation=False)

    def _compute_bom_count(self):
        bom_count_per_project = dict(
            self.env['mrp.bom']._read_group(
                [('project_id', 'in', self.ids)],
                ['project_id'], ['__count']
            )
        )
        for project in self:
            project.bom_count = bom_count_per_project.get(project)

    def _compute_production_count(self):
        production_count_per_project = dict(
            self.env['mrp.production']._read_group(
                [('project_id', 'in', self.ids)],
                ['project_id'], ['__count']
            )
        )
        for project in self:
            project.production_count = production_count_per_project.get(project)

    def action_view_mrp_bom(self):
        self.ensure_one()
        action = {
            'type': 'ir.actions.act_window',
            'res_model': 'mrp.bom',
            'domain': [('project_id', '=', self.id)],
            'name': self.env._('Bills of Materials'),
            'view_mode': 'list,kanban,form',
            'context': {'default_project_id': self.id},
            'help': "<p class='o_view_nocontent_smiling_face'>%s</p><p>%s</p>" % (
                _("No bill of materials found. Let's create one."),
                _("Bills of materials allow you to define the list of required raw materials used to make a finished "
                    "product; through a manufacturing order or a pack of products."),
            ),
        }
        boms = self.env['mrp.bom'].search([('project_id', '=', self.id)])
        if not self.env.context.get('from_embedded_action', False) and len(boms) == 1:
            action['views'] = [[False, 'form']]
            action['res_id'] = boms.id
        return action

    def action_view_mrp_production(self):
        self.ensure_one()
        action = self.env['ir.actions.actions']._for_xml_id('mrp.mrp_production_action')
        action['domain'] = [('project_id', '=', self.id)]
        action['context'] = {'default_project_id': self.id, 'from_project_action': True}
        productions = self.env['mrp.production'].search([('project_id', '=', self.id)])
        if not self.env.context.get('from_embedded_action', False) and len(productions) == 1:
            action['views'] = [[False, 'form']]
            action['res_id'] = productions.id
        return action

    def _get_stat_buttons(self):
        buttons = super()._get_stat_buttons()
        if self.env.user.has_group('mrp.group_mrp_user'):
            self_sudo = self.sudo()
            buttons.extend([{
                'icon': 'flask',
                'text': self.env._('Bills of Materials'),
                'number': self_sudo.bom_count,
                'action_type': 'object',
                'action': 'action_view_mrp_bom',
                'show': self_sudo.bom_count > 0,
                'sequence': 35,
            },
            {
                'icon': 'wrench',
                'text': self.env._('Manufacturing Orders'),
                'number': self_sudo.production_count,
                'action_type': 'object',
                'action': 'action_view_mrp_production',
                'show': self_sudo.production_count > 0,
                'sequence': 46,
            }])
        return buttons

```

## File: models\stock.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class StockRule(models.Model):
    _inherit = 'stock.rule'

    def _prepare_mo_vals(self, product_id, product_qty, product_uom, location_id, name, origin, company_id, values, bom):
        res = super()._prepare_mo_vals(product_id, product_qty, product_uom, location_id, name, origin, company_id, values, bom)
        if values.get('project_id'):
            res['project_id'] = values.get('project_id')
        return res


class StockMove(models.Model):
    _inherit = 'stock.move'

    def _prepare_procurement_values(self):
        res = super()._prepare_procurement_values()
        if res.get('group_id') and len(res['group_id'].mrp_production_ids) == 1:
            res['project_id'] = res['group_id'].mrp_production_ids.project_id.id
        return res

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import mrp_bom
from . import mrp_production
from . import project_project
from . import stock

```

## File: views\mrp_bom_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="mrp_bom_form_view_inherited_project_mrp" model="ir.ui.view">
        <field name="name">mrp.bom.form.inherited.project_mrp</field>
        <field name="model">mrp.bom</field>
        <field name="inherit_id" ref="mrp.mrp_bom_form_view"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='picking_type_id']" position="after">
                <field name="project_id" groups="project.group_project_user"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\mrp_production_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_production_tree_view_inherit_project_mrp" model="ir.ui.view">
        <field name="name">mrp.production.list.view.inherit.project_mrp</field>
        <field name="model">mrp.production</field>
        <field name="inherit_id" ref="mrp.mrp_production_tree_view"/>
        <field name="arch" type="xml">
            <field name="name" position="after">
                <field name="project_id" optional="hide" groups="project.group_project_user"/>
            </field>
        </field>
    </record>

    <record id="mrp_production_form_view_inherit_project_mrp" model="ir.ui.view">
        <field name="name">mrp.production.view.inherited</field>
        <field name="model">mrp.production</field>
        <field name="inherit_id" ref="mrp.mrp_production_form_view"/>
        <field name="arch" type="xml">
            <field name="bom_id" position="attributes">
                <attribute name="context">{
                    'default_product_tmpl_id': product_tmpl_id,
                    'default_project_id': project_id,
                }</attribute>
            </field>
            <xpath expr="//page[@name='miscellaneous']//field[@name='date_deadline']" position="after">
                <t groups="project.group_project_user">
                    <field name="project_id" groups="project.group_project_user"/>
                </t>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\project_project_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="project_embedded_action_bills_of_materials" model="ir.embedded.actions">
        <field name="parent_res_model">project.project</field>
        <field name="sequence">95</field>
        <field name="name">Bills of Materials</field>
        <field name="parent_action_id" ref="project.act_project_project_2_project_task_all"/>
        <field name="python_method">action_view_mrp_bom</field>
        <field name="context">{"from_embedded_action": true}</field>
        <field name="groups_ids" eval="[(4, ref('mrp.group_mrp_user'))]"/>
    </record>

    <record id="project_embedded_action_manufacturing_orders" model="ir.embedded.actions">
        <field name="parent_res_model">project.project</field>
        <field name="sequence">100</field>
        <field name="name">Manufacturing Orders</field>
        <field name="parent_action_id" ref="project.act_project_project_2_project_task_all"/>
        <field name="python_method">action_view_mrp_production</field>
        <field name="context">{"from_embedded_action": true}</field>
        <field name="groups_ids" eval="[(4, ref('mrp.group_mrp_user'))]"/>
    </record>
</odoo>

```

