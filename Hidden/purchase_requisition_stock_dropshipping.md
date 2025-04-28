# Odoo Module: purchase_requisition_stock_dropshipping

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Purchase Requisition Stock Dropshipping',
    'version': '1.0',
    'category': 'Hidden',
    'summary': 'Purchase Requisition, Stock, Dropshipping',
    'description': """
This module makes the link between the purchase requisition and dropshipping applications.

Set shipping address on purchase orders created from purchase agreements
and link with originating sale order.
""",
    'depends': ['purchase_requisition_stock', 'stock_dropshipping'],
    'data': [
        'views/purchase_views.xml',
    ],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\purchase.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class PurchaseOrder(models.Model):
    _inherit = 'purchase.order'

    @api.onchange('requisition_id')
    def _onchange_requisition_id(self):
        super()._onchange_requisition_id()
        if self.requisition_id and self.requisition_id.procurement_group_id:
            self.group_id = self.requisition_id.procurement_group_id.id
            if self.group_id.sale_id.partner_shipping_id:
                self.dest_address_id = self.group_id.sale_id.partner_shipping_id.id

```

## File: models\purchase_requisition.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class PurchaseRequisition(models.Model):
    _inherit = 'purchase.requisition'

    def _prepare_tender_values(self, product_id, product_qty, product_uom, location_id, name, origin, company_id, values):
        res = super()._prepare_tender_values(product_id, product_qty, product_uom, location_id, name, origin, company_id, values)
        if 'sale_line_id' in values:
            res['line_ids'][0][2]['sale_line_id'] = values['sale_line_id']
        return res


class PurchaseRequisitionLine(models.Model):
    _inherit = "purchase.requisition.line"

    sale_line_id = fields.Many2one('sale.order.line', string="Origin Sale Order Line")

    def _prepare_purchase_order_line(self, name, product_qty=0.0, price_unit=0.0, taxes_ids=False):
        res = super()._prepare_purchase_order_line(name, product_qty, price_unit, taxes_ids)
        if self.sale_line_id:
            res['sale_line_id'] = self.sale_line_id.id
        return res

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import purchase
from . import purchase_requisition

```

## File: views\purchase_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="purchase_order_view_form_inherit" model="ir.ui.view">
            <field name="name">purchase.order.form.inherit</field>
            <field name="model">purchase.order</field>
            <field name="inherit_id" ref="purchase.purchase_order_form"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='origin']" position="after">
                    <field name="group_id" invisible="1"/>
                </xpath>
                <xpath expr="//field[@name='order_line']/tree//field[@name='price_subtotal']" position="after">
                    <field name="sale_line_id" invisible="1"/>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

