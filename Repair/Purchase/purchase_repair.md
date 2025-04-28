# Odoo Module: purchase_repair

Category: Repair/Purchase

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
    'name': 'Purchase Repair',
    'summary': 'Keep track of linked purchase and repair orders',
    'version': '1.0',
    'category': 'Repair/Purchase',
    'license': 'LGPL-3',
    'depends': ['repair', 'purchase_stock'],
    'data': [
        'views/purchase_views.xml',
        'views/repair_views.xml',
    ],
    'auto_install': True,
    'installable': True,
}

```

## File: models\purchase_order.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, fields, models, _


class PurchaseOrder(models.Model):
    _inherit = 'purchase.order'

    repair_count = fields.Integer(string='Count of source repairs', compute='_compute_repair_count', groups='stock.group_stock_user')

    @api.depends('order_line.move_dest_ids.repair_id')
    def _compute_repair_count(self):
        for purchase in self:
            purchase.repair_count = len(purchase.order_line.move_dest_ids.repair_id)

    def action_view_repair_orders(self):
        self.ensure_one()
        repair_ids = self.order_line.move_dest_ids.repair_id
        action = {
            'type': 'ir.actions.act_window',
            'res_model': 'repair.order',
            'views': [[False, 'form']]
        }
        if self.repair_count == 1:
            action['res_id'] = repair_ids.id
        elif self.repair_count > 1:
            action['name'] = _("Repair Source of %s", self.name)
            action['views'] = [[False, 'list']]
            action['domain'] = [('id', 'in', repair_ids.ids)]
        return action

```

## File: models\repair_order.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, models, fields, _


class RepairOrder(models.Model):
    _inherit = 'repair.order'

    purchase_count = fields.Integer(string="Count of generated POs", compute="_compute_purchase_count", groups="purchase.group_purchase_user")

    @api.depends('move_ids.created_purchase_line_ids.order_id')
    def _compute_purchase_count(self):
        for repair in self:
            repair.purchase_count = len(repair.move_ids.created_purchase_line_ids.order_id)

    def action_view_purchase_orders(self):
        self.ensure_one()
        purchase_ids = self.move_ids.created_purchase_line_ids.order_id
        action = {
            'type': 'ir.actions.act_window',
            'res_model': 'purchase.order',
            'views': [[False, 'form']],
        }
        if self.purchase_count == 1:
            action['res_id'] = purchase_ids.id
        elif self.purchase_count > 1:
            action['name'] = _('Purchase Orders')
            action['views'] = [[False, 'list']]
            action['domain'] = [('id', 'in', purchase_ids.ids)]
        return action

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import purchase_order
from . import repair_order

```

## File: views\purchase_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="purchase_order_form_inherit" model="ir.ui.view">
        <field name="name">purchase.order.form.inherit</field>
        <field name="model">purchase.order</field>
        <field name="inherit_id" ref="purchase.purchase_order_form"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@name='action_view_invoice']" position="after">
                <button
                    class="oe_stat_button" name="action_view_repair_orders" type="object"
                    icon="fa-wrench" groups="stock.group_stock_user"
                    invisible="repair_count == 0">
                    <div class="o_field_widget o_stat_info">
                        <field name="repair_count" string="Repair Orders" widget="statinfo"/>
                    </div>
                </button>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\repair_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_repair_order_form_inherit" model="ir.ui.view">
        <field name="name">repair.order.form.inherit</field>
        <field name="model">repair.order</field>
        <field name="inherit_id" ref="repair.view_repair_order_form"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@name='action_view_sale_order']" position="after">
                <button
                    class="oe_stat_button" name="action_view_purchase_orders" type="object"
                    icon="fa-credit-card" groups="purchase.group_purchase_user"
                    invisible="purchase_count == 0 or state == 'draft'">
                    <div class="o_field_widget o_stat_info">
                        <field name="purchase_count" string="Purchase Orders" widget="statinfo"/>
                    </div>
                </button>
            </xpath>
        </field>
    </record>
</odoo>

```

