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
    'demo':[
        'data/sale_margin_demo.xml',
    ],
    'data':[
        'views/sale_order_views.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\sale_margin_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

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

</odoo>

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class SaleOrder(models.Model):
    _inherit = "sale.order"

    margin = fields.Monetary("Margin", compute='_compute_margin', store=True)
    margin_percent = fields.Float("Margin (%)", compute='_compute_margin', store=True, aggregator="avg")

    @api.depends('order_line.margin', 'amount_untaxed')
    def _compute_margin(self):
        if not all(self._ids):
            for order in self:
                order.margin = sum(order.order_line.mapped('margin'))
                order.margin_percent = order.amount_untaxed and order.margin/order.amount_untaxed
        else:
            # On batch records recomputation (e.g. at install), compute the margins
            # with a single read_group query for better performance.
            # This isn't done in an onchange environment because (part of) the data
            # may not be stored in database (new records or unsaved modifications).
            grouped_order_lines_data = self.env['sale.order.line']._read_group(
                [
                    ('order_id', 'in', self.ids),
                ], ['order_id'], ['margin:sum'])
            mapped_data = {order.id: margin for order, margin in grouped_order_lines_data}
            for order in self:
                order.margin = mapped_data.get(order.id, 0.0)
                order.margin_percent = order.amount_untaxed and order.margin/order.amount_untaxed

```

## File: models\sale_order_line.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class SaleOrderLine(models.Model):
    _inherit = "sale.order.line"

    margin = fields.Float(
        "Margin", compute='_compute_margin',
        digits='Product Price', store=True, groups="base.group_user", precompute=True)
    margin_percent = fields.Float(
        "Margin (%)", compute='_compute_margin', store=True, groups="base.group_user", precompute=True)
    purchase_price = fields.Float(
        string="Cost", compute="_compute_purchase_price",
        digits='Product Price', store=True, readonly=False, copy=False, precompute=True,
        groups="base.group_user")

    @api.depends('product_id', 'company_id', 'currency_id', 'product_uom')
    def _compute_purchase_price(self):
        for line in self:
            if not line.product_id:
                line.purchase_price = 0.0
                continue
            line = line.with_company(line.company_id)

            # Convert the cost to the line UoM
            product_cost = line.product_id.uom_id._compute_price(
                line.product_id.standard_price,
                line.product_uom,
            )

            line.purchase_price = line._convert_to_sol_currency(
                product_cost,
                line.product_id.cost_currency_id)

    @api.depends('price_subtotal', 'product_uom_qty', 'purchase_price')
    def _compute_margin(self):
        for line in self:
            # Find alternative calculation when line is added to order from delivery
            if line.qty_delivered and not line.product_uom_qty:
                calculated_subtotal = line.price_unit * line.qty_delivered
                line.margin = calculated_subtotal - (line.purchase_price * line.qty_delivered)
                line.margin_percent = calculated_subtotal and line.margin / calculated_subtotal
            else:
                line.margin = line.price_subtotal - (line.purchase_price * line.product_uom_qty)
                line.margin_percent = line.price_subtotal and line.margin / line.price_subtotal

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import sale_order
from . import sale_order_line

```

## File: report\sale_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class SaleReport(models.Model):
    _inherit = 'sale.report'

    margin = fields.Float('Margin')

    def _select_additional_fields(self):
        res = super()._select_additional_fields()
        res['margin'] = f"""SUM(l.margin
            / {self._case_value_or_one('s.currency_rate')}
            * {self._case_value_or_one('account_currency_table.rate')})
        """
        return res

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import sale_report

```

## File: views\sale_order_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record model="ir.ui.view" id="sale_margin_sale_order">
        <field name="name">sale.order.margin.view.form</field>
        <field name="model">sale.order</field>
        <field name="inherit_id" ref="sale.view_order_form"/>
        <field name="priority" eval="15"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='tax_totals']" position="after">
                <div class="d-flex float-end" colspan="2" groups="base.group_user">
                    <label for="margin"/>
                    <div>
                        <field name="margin" class="oe_inline"/>
                        <field name="amount_untaxed" invisible="1"/>
                        <span class="oe_inline" invisible="amount_untaxed == 0">
                            (<field name="margin_percent" nolabel="1" class="oe_inline" widget="percentage"/>)
                        </span>
                    </div>
                </div>
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
        <field name="name">sale.order.line.list.margin.view.form</field>
        <field name="model">sale.order</field>
        <field name="inherit_id" ref="sale.view_order_form"/>
        <field name="arch" type="xml">
           <xpath expr="//field[@name='order_line']/list//field[@name='price_unit']" position="after">
                <field name="price_subtotal" column_invisible="True"/>
                <field name="purchase_price" optional="hide"/>
                <field name="margin" optional="hide"/>
                <field name="margin_percent"
                    invisible="price_subtotal == 0"
                    optional="hide" widget="percentage"/>
            </xpath>
        </field>
    </record>

    <record model="ir.ui.view" id="sale_margin_sale_order_pivot">
        <field name="name">sale.order.margin.view.pivot</field>
        <field name="model">sale.order</field>
        <field name="inherit_id" ref="sale.view_sale_order_pivot"/>
        <field name="arch" type="xml">
            <pivot position="inside">
                <field name="margin_percent" invisible="1"/>
            </pivot>
        </field>
    </record>

    <record model="ir.ui.view" id="sale_margin_sale_order_graph">
        <field name="name">sale.order.margin.view.graph</field>
        <field name="model">sale.order</field>
        <field name="inherit_id" ref="sale.view_sale_order_graph"/>
        <field name="arch" type="xml">
            <graph position="inside">
                <field name="margin_percent" invisible="1"/>
            </graph>
        </field>
    </record>

</odoo>

```

