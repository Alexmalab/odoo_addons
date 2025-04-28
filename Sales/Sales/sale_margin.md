# Odoo Module: sale_margin

Category: Sales/Sales

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models      # noqa
from . import report      # noqa

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Margins in Sales Orders',
    'version':'1.0',
    'category': 'Sales/Sales',
    'description': """
This module adds the 'Margin' on sales order.
=============================================

This gives the profitability by calculating the difference between the Unit
Price and Cost Price.
    """,
    'depends':['sale_management'],
    'demo':['data/sale_margin_demo.xml'],
    'data':['security/ir.model.access.csv','views/sale_margin_view.xml'],
    'license': 'LGPL-3',
}

```

## File: data\sale_margin_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <record id="sale.sale_order_line_1" model="sale.order.line">
            <field name="purchase_price">2870.00</field>
        </record>

        <record id="sale.sale_order_line_2" model="sale.order.line">
            <field name="purchase_price">126.00</field>
        </record>

        <record id="sale.sale_order_line_3" model="sale.order.line">
            <field name="purchase_price">60.00</field>
        </record>

        <record id="sale.sale_order_line_12" model="sale.order.line">
            <field name="purchase_price">390.00</field>
        </record>

        <record id="sale.sale_order_line_16" model="sale.order.line">
            <field name="purchase_price">2870.00</field>
        </record>

        <record id="sale.sale_order_line_17" model="sale.order.line">
            <field name="purchase_price">155.00</field>
        </record>

        <record id="sale.sale_order_line_18" model="sale.order.line">
            <field name="purchase_price">35.00</field>
        </record>

        <record id="sale.sale_order_line_19" model="sale.order.line">
            <field name="purchase_price">13.00</field>
        </record>

    </data>
</odoo>

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo.tools.sql import column_exists, create_column


class SaleOrderLine(models.Model):
    _inherit = "sale.order.line"

    def _auto_init(self):
        if not column_exists(self.env.cr, "sale_order_line", "margin"):
            # By creating the column 'margin' manually we steer clear of hefty data computation.
            create_column(self.env.cr, "sale_order_line", "margin", "NUMERIC")
        return super()._auto_init()

    margin = fields.Float(compute='_product_margin', digits='Product Price', store=True)
    purchase_price = fields.Float(string='Cost', digits='Product Price')

    def _compute_margin(self, order_id, product_id, product_uom_id):
        frm_cur = self.env.company.currency_id
        to_cur = order_id.pricelist_id.currency_id
        purchase_price = product_id.standard_price
        if product_uom_id != product_id.uom_id:
            purchase_price = product_id.uom_id._compute_price(purchase_price, product_uom_id)
        price = frm_cur._convert(
            purchase_price, to_cur, order_id.company_id or self.env.company,
            order_id.date_order or fields.Date.today(), round=False)
        return price

    @api.model
    def _get_purchase_price(self, pricelist, product, product_uom, date):
        frm_cur = self.env.company.currency_id
        to_cur = pricelist.currency_id
        purchase_price = product.standard_price
        if product_uom != product.uom_id:
            purchase_price = product.uom_id._compute_price(purchase_price, product_uom)
        price = frm_cur._convert(
            purchase_price, to_cur,
            self.order_id.company_id or self.env.company,
            date or fields.Date.today(), round=False)
        return {'purchase_price': price}

    @api.onchange('product_id', 'product_uom')
    def product_id_change_margin(self):
        if not self.order_id.pricelist_id or not self.product_id or not self.product_uom:
            return
        self.purchase_price = self._compute_margin(self.order_id, self.product_id, self.product_uom)

    @api.onchange('product_id')
    def product_id_change(self):
        # VFE FIXME : bugfix for matrix, the purchase_price will be changed to a computed field in master.
        res = super(SaleOrderLine, self).product_id_change()
        self.product_id_change_margin()
        return res

    @api.model
    def create(self, vals):
        vals.update(self._prepare_add_missing_fields(vals))

        # Calculation of the margin for programmatic creation of a SO line. It is therefore not
        # necessary to call product_id_change_margin manually
        if 'purchase_price' not in vals and ('display_type' not in vals or not vals['display_type']):
            order_id = self.env['sale.order'].browse(vals['order_id'])
            product_id = self.env['product.product'].browse(vals['product_id'])
            product_uom_id = self.env['uom.uom'].browse(vals['product_uom'])

            vals['purchase_price'] = self._compute_margin(order_id, product_id, product_uom_id)

        return super(SaleOrderLine, self).create(vals)

    @api.depends('product_id', 'purchase_price', 'product_uom_qty', 'price_unit', 'price_subtotal')
    def _product_margin(self):
        for line in self:
            currency = line.order_id.pricelist_id.currency_id
            price = line.purchase_price
            margin = line.price_subtotal - (price * line.product_uom_qty)
            line.margin = currency.round(margin) if currency else margin

class SaleOrder(models.Model):
    _inherit = "sale.order"

    margin = fields.Monetary(compute='_product_margin', help="It gives profitability by calculating the difference between the Unit Price and the cost.", currency_field='currency_id', store=True)

    @api.depends('order_line.margin')
    def _product_margin(self):
        if not all(self._ids):
            for order in self:
                order.margin = sum(order.order_line.filtered(lambda r: r.state != 'cancel').mapped('margin'))
        else:
            self.env["sale.order.line"].flush(['margin', 'state'])
            # On batch records recomputation (e.g. at install), compute the margins
            # with a single read_group query for better performance.
            # This isn't done in an onchange environment because (part of) the data
            # may not be stored in database (new records or unsaved modifications).
            grouped_order_lines_data = self.env['sale.order.line'].read_group(
                [
                    ('order_id', 'in', self.ids),
                    ('state', '!=', 'cancel'),
                ], ['margin', 'order_id'], ['order_id'])
            mapped_data = {m['order_id'][0]: m['margin'] for m in grouped_order_lines_data}
            for order in self:
                order.margin = mapped_data.get(order.id, 0.0)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import sale_order

```

## File: report\sale_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class SaleReport(models.Model):
    _inherit = 'sale.report'

    margin = fields.Float('Margin')

    def _query(self, with_clause='', fields={}, groupby='', from_clause=''):
        fields['margin'] = ", SUM(l.margin / CASE COALESCE(s.currency_rate, 0) WHEN 0 THEN 1.0 ELSE s.currency_rate END) AS margin"
        return super(SaleReport, self)._query(with_clause, fields, groupby, from_clause)

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import sale_report

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink

```

## File: views\sale_margin_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record model="ir.ui.view" id="sale_margin_sale_order">
        <field name="name">sale.order.margin.view.form</field>
        <field name="model">sale.order</field>
        <field name="inherit_id" ref="sale.view_order_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='amount_total']" position="after">
                    <field name="margin" groups="base.group_user"/>
            </xpath>
        </field>
    </record>

    <record model="ir.ui.view" id="sale_margin_sale_order_line">
        <field name="name">sale.order.line.margin.view.form</field>
        <field name="model">sale.order</field>
        <field name="inherit_id" ref="sale.view_order_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='order_line']/form//field[@name='price_unit']" position="after">
                <field name="purchase_price" groups="base.group_user"/>
            </xpath>
        </field>
    </record>

    <record model="ir.ui.view" id="sale_margin_sale_order_line_form">
        <field name="name">sale.order.line.tree.margin.view.form</field>
        <field name="model">sale.order</field>
        <field name="inherit_id" ref="sale.view_order_form"/>
        <field name="arch" type="xml">
           <xpath expr="//field[@name='order_line']/tree//field[@name='price_unit']" position="after">
                <field name="purchase_price" optional="hide" groups="base.group_user"/>
            </xpath>
        </field>
    </record>

</odoo>

```

