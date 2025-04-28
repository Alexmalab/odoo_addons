# Odoo Module: purchase_requisition_stock

Category: Inventory/Purchase

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Purchase Requisition Stock',
    'version': '1.2',
    'category': 'Inventory/Purchase',
    'sequence': 70,
    'depends': ['purchase_requisition', 'purchase_stock'],
    'demo': [
        'data/purchase_requisition_stock_demo.xml'
        ],
    'data': [
        'security/ir.model.access.csv',
        'data/purchase_requisition_stock_data.xml',
        'views/purchase_views.xml',
        'views/purchase_requisition_views.xml',
    ],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\purchase_requisition_stock_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
         <function
            model="ir.default" name="set"
            eval="('purchase.requisition', 'warehouse_id', ref('stock.warehouse0'))"/>
    </data>
</odoo>

```

## File: data\purchase_requisition_stock_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="purchase_requisition.bo_requisition" model="purchase.requisition">
            <field name="warehouse_id" ref="stock.stock_warehouse_shop0"/>
        </record>
    </data>
</odoo>

```

## File: models\purchase.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class PurchaseOrder(models.Model):
    _inherit = 'purchase.order'

    on_time_rate_perc = fields.Float(string="OTD", compute="_compute_on_time_rate_perc")

    @api.depends('on_time_rate')
    def _compute_on_time_rate_perc(self):
        for po in self:
            if po.on_time_rate >= 0:
                po.on_time_rate_perc = po.on_time_rate / 100
            else:
                po.on_time_rate_perc = -1

    @api.onchange('requisition_id')
    def _onchange_requisition_id(self):
        super(PurchaseOrder, self)._onchange_requisition_id()
        if self.requisition_id:
            self.picking_type_id = self.requisition_id.picking_type_id.id


class PurchaseOrderLine(models.Model):
    _inherit = 'purchase.order.line'

    on_time_rate_perc = fields.Float(string="OTD", related="order_id.on_time_rate_perc")

```

## File: models\purchase_requisition.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class PurchaseRequisition(models.Model):
    _inherit = 'purchase.requisition'

    def _default_picking_type_id(self):
        return self.env['stock.picking.type'].search([('warehouse_id.company_id', '=', self.env.company.id), ('code', '=', 'incoming')], limit=1)

    warehouse_id = fields.Many2one('stock.warehouse', string='Warehouse', domain="[('company_id', '=', company_id)]")
    picking_type_id = fields.Many2one(
        'stock.picking.type', 'Operation Type', required=True, default=_default_picking_type_id,
        domain="['|',('warehouse_id', '=', False), ('warehouse_id.company_id', '=', company_id)]")


class PurchaseRequisitionLine(models.Model):
    _inherit = "purchase.requisition.line"

    move_dest_id = fields.Many2one('stock.move', 'Downstream Move')

    def _prepare_purchase_order_line(self, name, product_qty=0.0, price_unit=0.0, taxes_ids=False):
        res = super(PurchaseRequisitionLine, self)._prepare_purchase_order_line(name, product_qty, price_unit, taxes_ids)
        res['move_dest_ids'] = self.move_dest_id and [(4, self.move_dest_id.id)] or []
        return res

```

## File: models\stock.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class StockRule(models.Model):
    _inherit = 'stock.rule'

    def _prepare_purchase_order(self, company_id, origins, values):
        res = super(StockRule, self)._prepare_purchase_order(company_id, origins, values)
        values = values[0]
        res['partner_ref'] = values['supplier'].purchase_requisition_id.name
        res['requisition_id'] = values['supplier'].purchase_requisition_id.id
        if values['supplier'].purchase_requisition_id.currency_id:
            res['currency_id'] = values['supplier'].purchase_requisition_id.currency_id.id
        return res

    def _make_po_get_domain(self, company_id, values, partner):
        domain = super(StockRule, self)._make_po_get_domain(company_id, values, partner)
        if 'supplier' in values and values['supplier'].purchase_requisition_id:
            domain += (
                ('requisition_id', '=', values['supplier'].purchase_requisition_id.id),
            )
        return domain


class StockMove(models.Model):
    _inherit = 'stock.move'

    requisition_line_ids = fields.One2many('purchase.requisition.line', 'move_dest_id')

    def _get_upstream_documents_and_responsibles(self, visited):
        # People without purchase rights should be able to do this operation
        requisition_lines_sudo = self.sudo().requisition_line_ids
        if requisition_lines_sudo:
            return [(requisition_line.requisition_id, requisition_line.requisition_id.user_id, visited) for requisition_line in requisition_lines_sudo if requisition_line.requisition_id.state not in ('done', 'cancel')]
        else:
            return super(StockMove, self)._get_upstream_documents_and_responsibles(visited)


class Orderpoint(models.Model):
    _inherit = "stock.warehouse.orderpoint"

    def _quantity_in_progress(self):
        res = super(Orderpoint, self)._quantity_in_progress()
        for op in self:
            for pr in self.env['purchase.requisition'].search([('state', '=', 'draft'), ('origin', '=', op.name)]):
                for prline in pr.line_ids.filtered(lambda l: l.product_id.id == op.product_id.id and not l.move_dest_id):
                    res[op.id] += prline.product_uom_id._compute_quantity(prline.product_qty, op.product_uom, round=False)
        return res

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import purchase
from . import stock
from . import purchase_requisition

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_purchase_requisition_stock_manager,purchase.requisition,purchase_requisition.model_purchase_requisition,stock.group_stock_manager,1,0,1,0
access_purchase_requisition_line_stock_manager,purchase.requisition.line,purchase_requisition.model_purchase_requisition_line,stock.group_stock_manager,1,0,1,0


```

## File: views\purchase_requisition_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="view_purchase_requisition_form_inherit" model="ir.ui.view">
            <field name="name">purchase.requisition.form.inherit</field>
            <field name="model">purchase.requisition</field>
            <field name="inherit_id" ref="purchase_requisition.view_purchase_requisition_form" />
            <field name="arch" type="xml">
                <field name="origin" position="after">
                    <field name="picking_type_id" options="{'no_open': True, 'no_create': True}" groups="stock.group_adv_location" readonly="state != 'draft'"/>
                </field>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\purchase_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="purchase_order_form_inherit_purchase_requisition_stock" model="ir.ui.view">
        <field name="name">purchase.order.form.inherit.purchase.requisition.stock</field>
        <field name="model">purchase.order</field>
        <field name="inherit_id" ref="purchase_requisition.purchase_order_form_inherit"/>
        <field name="arch" type="xml">
            <xpath expr="//page[@name='alternative_pos']//field[@name='alternative_po_ids']//field[@name='partner_id']" position="after">
                <field name='on_time_rate_perc' widget="percentage" invisible="on_time_rate_perc &lt; 0"/>
            </xpath>
        </field>
    </record>

    <record id="purchase_order_line_compare_tree_inherit_purchase_requisition_stock" model="ir.ui.view">
        <field name="name">purchase.order.line.compare.tree.purchase.requisition.stock</field>
        <field name="model">purchase.order.line</field>
        <field name="inherit_id" ref="purchase_requisition.purchase_order_line_compare_tree"/>
        <field name="arch" type="xml">
            <field name="partner_id" position="after">
                <field name='on_time_rate_perc' widget="percentage" invisible="on_time_rate_perc &lt; 0"/>
            </field>
        </field>
    </record>
</odoo>

```

## File: wizard\purchase_requisition_create_alternative.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, Command, models


class PurchaseRequisitionCreateAlternative(models.TransientModel):
    _inherit = 'purchase.requisition.create.alternative'

    def _get_alternative_values(self):
        vals = super(PurchaseRequisitionCreateAlternative, self)._get_alternative_values()
        vals.update({
            'picking_type_id': self.origin_po_id.picking_type_id.id,
            'group_id': self.origin_po_id.group_id.id,
        })
        return vals

    @api.model
    def _get_alternative_line_value(self, order_line):
        res_line = super()._get_alternative_line_value(order_line)
        if order_line.move_dest_ids:
            res_line['move_dest_ids'] = [Command.set(order_line.move_dest_ids.ids)]

        return res_line

```

## File: wizard\__init__.py

```python
from . import purchase_requisition_create_alternative

```

