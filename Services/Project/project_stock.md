# Odoo Module: project_stock

Category: Services/Project

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
    'name': 'Project Stock',
    'version': '1.0',
    'summary': 'Link Stock pickings to Project',
    'category': 'Services/Project',
    'depends': ['stock', 'project'],
    'data': [
        'views/stock_picking_views.xml',
        'views/project_project_views.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\project_project.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, _
from odoo.osv.expression import AND


class ProjectProject(models.Model):
    _inherit = 'project.project'

    def action_open_deliveries(self):
        self.ensure_one()
        return self._get_picking_action(_('From WH'), 'outgoing')

    def action_open_receipts(self):
        self.ensure_one()
        return self._get_picking_action(_('To WH'), 'incoming')

    def action_open_all_pickings(self):
        self.ensure_one()
        return self._get_picking_action(_('Stock Moves'))

    def _get_picking_action(self, action_name, picking_type=None):
        domain = [('project_id', '=', self.id)]
        context = {'default_project_id': self.id}
        if picking_type:
            domain = AND([domain, [('picking_type_id.code', '=', picking_type)]])
            context['restricted_picking_type_code'] = picking_type
            if picking_type == 'outgoing':
                context['default_partner_id'] = self.partner_id.id
        return {
            'name': action_name,
            'type': 'ir.actions.act_window',
            'res_model': 'stock.picking',
            'view_mode': f"list,kanban,form,calendar,{'map' if picking_type == 'outgoing' else 'activity'}",
            'domain': domain,
            'context': context,
            'help': self.env['ir.ui.view']._render_template(
                'stock.help_message_template', {
                    'picking_type_code': picking_type,
                }
            ),
        }

```

## File: models\stock_picking.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class StockPicking(models.Model):
    _inherit = 'stock.picking'

    project_id = fields.Many2one('project.project')

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import project_project
from . import stock_picking

```

## File: views\project_project_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="project_embedded_action_from_wh" model="ir.embedded.actions">
        <field name="parent_res_model">project.project</field>
        <field name="sequence">80</field>
        <field name="name">From WH</field>
        <field name="parent_action_id" ref="project.act_project_project_2_project_task_all"/>
        <field name="python_method">action_open_deliveries</field>
        <field name="groups_ids" eval="[(4, ref('stock.group_stock_user'))]"/>
    </record>

    <record id="project_embedded_action_to_wh" model="ir.embedded.actions">
        <field name="parent_res_model">project.project</field>
        <field name="sequence">90</field>
        <field name="name">To WH</field>
        <field name="parent_action_id" ref="project.act_project_project_2_project_task_all"/>
        <field name="python_method">action_open_receipts</field>
        <field name="groups_ids" eval="[(4, ref('stock.group_stock_user'))]"/>
    </record>

    <record id="project_embedded_action_all_pickings" model="ir.embedded.actions">
        <field name="parent_res_model">project.project</field>
        <field name="sequence">92</field>
        <field name="name">Stock Moves</field>
        <field name="parent_action_id" ref="project.act_project_project_2_project_task_all"/>
        <field name="python_method">action_open_all_pickings</field>
        <field name="groups_ids" eval="[(4, ref('stock.group_stock_user'))]"/>
    </record>
</odoo>

```

## File: views\stock_picking_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_picking_form_inherit_project_stock" model="ir.ui.view">
        <field name="name">stock.picking.form.inherit.project_stock</field>
        <field name="model">stock.picking</field>
        <field name="inherit_id" ref="stock.view_picking_form"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='other_infos']" position="inside">
                <field name="project_id" groups="project.group_project_user"/>
            </xpath>
        </field>
    </record>
</odoo>

```

