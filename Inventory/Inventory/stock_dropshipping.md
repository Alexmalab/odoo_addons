# Odoo Module: stock_dropshipping

Category: Inventory/Inventory

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
    'name': 'Drop Shipping',
    'version': '1.0',
    'category': 'Inventory/Inventory',
    'summary': 'Drop Shipping',
    'description': """
Manage drop shipping orders
===========================

This module adds a pre-configured Drop Shipping operation type
as well as a procurement route that allow configuring Drop
Shipping products and orders.

When drop shipping is used the goods are directly transferred
from vendors to customers (direct delivery) without
going through the retailer's warehouse. In this case no
internal transfer document is needed.

""",
    'depends': ['sale_purchase_stock'],
    'data': ['data/stock_data.xml', 'views/sale_order_views.xml'],
    'installable': True,
    'auto_install': False,
    'license': 'LGPL-3',
}

```

## File: data\stock_data.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data noupdate="1">
        <function model="res.company" name="create_missing_dropship_sequence"/>
        <function model="res.company" name="create_missing_dropship_picking_type"/>

        <record id="route_drop_shipping" model='stock.location.route'>
            <field name="name">Dropship</field>
            <field name="sequence">3</field>
            <field name="company_id"></field>
            <field name="sale_selectable" eval="True"/>
            <field name="product_selectable" eval="True"/>
            <field name="product_categ_selectable" eval="True"/>
        </record>

        <function model="res.company" name="create_missing_dropship_sequence"/>
        <function model="res.company" name="create_missing_dropship_picking_type"/>
        <function model="res.company" name="create_missing_dropship_rule"/>
    </data>
</odoo>

```

## File: models\purchase.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class PurchaseOrderLine(models.Model):
    _inherit = 'purchase.order.line'

    def _prepare_stock_moves(self, picking):
        res = super(PurchaseOrderLine, self)._prepare_stock_moves(picking)
        for re in res:
            re['sale_line_id'] = self.sale_line_id.id
        return res

    def _find_candidate(self, product_id, product_qty, product_uom, location_id, name, origin, company_id, values):
        # if this is defined, this is a dropshipping line, so no
        # this is to correctly map delivered quantities to the so lines
        lines = self.filtered(lambda po_line: po_line.sale_line_id.id == values['sale_line_id']) if values.get('sale_line_id') else self
        return super(PurchaseOrderLine, lines)._find_candidate(product_id, product_qty, product_uom, location_id, name, origin, company_id, values)

    @api.model
    def _prepare_purchase_order_line_from_procurement(self, product_id, product_qty, product_uom, company_id, values, po):
        res = super()._prepare_purchase_order_line_from_procurement(product_id, product_qty, product_uom, company_id, values, po)
        res['sale_line_id'] = values.get('sale_line_id', False)
        return res

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class ResCompany(models.Model):
    _inherit = 'res.company'

    # -------------------------------------------------------------------------
    # Sequences
    # -------------------------------------------------------------------------
    def _create_dropship_sequence(self):
        dropship_vals = []
        for company in self:
            dropship_vals.append({
                'name': 'Dropship (%s)' % company.name,
                'code': 'stock.dropshipping',
                'company_id': company.id,
                'prefix': 'DS/',
                'padding': 5,
            })
        if dropship_vals:
            self.env['ir.sequence'].create(dropship_vals)

    @api.model
    def create_missing_dropship_sequence(self):
        company_ids = self.env['res.company'].search([])
        company_has_dropship_seq = self.env['ir.sequence'].search([('code', '=', 'stock.dropshipping')]).mapped('company_id')
        company_todo_sequence = company_ids - company_has_dropship_seq
        company_todo_sequence._create_dropship_sequence()

    def _create_per_company_sequences(self):
        super(ResCompany, self)._create_per_company_sequences()
        self._create_dropship_sequence()

    # -------------------------------------------------------------------------
    # Picking types
    # -------------------------------------------------------------------------
    def _create_dropship_picking_type(self):
        dropship_vals = []
        for company in self:
            sequence = self.env['ir.sequence'].search([
                ('code', '=', 'stock.dropshipping'),
                ('company_id', '=', company.id),
            ])
            dropship_vals.append({
                'name': 'Dropship',
                'company_id': company.id,
                'warehouse_id': False,
                'sequence_id': sequence.id,
                'code': 'incoming',
                'default_location_src_id': self.env.ref('stock.stock_location_suppliers').id,
                'default_location_dest_id': self.env.ref('stock.stock_location_customers').id,
                'sequence_code': 'DS',
            })
        if dropship_vals:
            self.env['stock.picking.type'].create(dropship_vals)

    @api.model
    def create_missing_dropship_picking_type(self):
        company_ids = self.env['res.company'].search([])
        company_has_dropship_picking_type = (
            self.env['stock.picking.type']
            .search([
                ('default_location_src_id.usage', '=', 'supplier'),
                ('default_location_dest_id.usage', '=', 'customer'),
            ])
            .mapped('company_id')
        )
        company_todo_picking_type = company_ids - company_has_dropship_picking_type
        company_todo_picking_type._create_dropship_picking_type()

    def _create_per_company_picking_types(self):
        super(ResCompany, self)._create_per_company_picking_types()
        self._create_dropship_picking_type()

    # -------------------------------------------------------------------------
    # Stock rules
    # -------------------------------------------------------------------------
    def _create_dropship_rule(self):
        dropship_route = self.env.ref('stock_dropshipping.route_drop_shipping')
        supplier_location = self.env.ref('stock.stock_location_suppliers')
        customer_location = self.env.ref('stock.stock_location_customers')

        dropship_vals = []
        for company in self:
            dropship_picking_type = self.env['stock.picking.type'].search([
                ('company_id', '=', company.id),
                ('default_location_src_id.usage', '=', 'supplier'),
                ('default_location_dest_id.usage', '=', 'customer'),
            ], limit=1, order='sequence')
            if not dropship_picking_type:
                continue
            dropship_vals.append({
                'name': '%s → %s' % (supplier_location.name, customer_location.name),
                'action': 'buy',
                'location_id': customer_location.id,
                'location_src_id': supplier_location.id,
                'procure_method': 'make_to_stock',
                'route_id': dropship_route.id,
                'picking_type_id': dropship_picking_type.id,
                'company_id': company.id,
            })
        if dropship_vals:
            self.env['stock.rule'].create(dropship_vals)

    @api.model
    def create_missing_dropship_rule(self):
        dropship_route = self.env.ref('stock_dropshipping.route_drop_shipping')

        company_ids = self.env['res.company'].search([])
        company_has_dropship_rule = self.env['stock.rule'].search([('route_id', '=', dropship_route.id)]).mapped('company_id')
        company_todo_rule = company_ids - company_has_dropship_rule
        company_todo_rule._create_dropship_rule()

    def _create_per_company_rules(self):
        super(ResCompany, self)._create_per_company_rules()
        self._create_dropship_rule()

```

## File: models\sale.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class SaleOrderLine(models.Model):
    _inherit = 'sale.order.line'

    def _compute_is_mto(self):
        super(SaleOrderLine, self)._compute_is_mto()
        for line in self:
            if not line.display_qty_widget or line.is_mto:
                continue
            product_routes = line.route_id or (line.product_id.route_ids + line.product_id.categ_id.total_route_ids)
            for pull_rule in product_routes.mapped('rule_ids'):
                if pull_rule.picking_type_id.sudo().default_location_src_id.usage == 'supplier' and\
                        pull_rule.picking_type_id.sudo().default_location_dest_id.usage == 'customer':
                    line.is_mto = True
                    break

    def _get_qty_procurement(self, previous_product_uom_qty):
        # People without purchase rights should be able to do this operation
        purchase_lines_sudo = self.sudo().purchase_line_ids
        if purchase_lines_sudo.filtered(lambda r: r.state != 'cancel'):
            qty = 0.0
            for po_line in purchase_lines_sudo.filtered(lambda r: r.state != 'cancel'):
                qty += po_line.product_uom._compute_quantity(po_line.product_qty, self.product_uom, rounding_method='HALF-UP')
            return qty
        else:
            return super(SaleOrderLine, self)._get_qty_procurement(previous_product_uom_qty=previous_product_uom_qty)

```

## File: models\stock.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class StockRule(models.Model):
    _inherit = 'stock.rule'

    @api.model
    def _get_procurements_to_merge_groupby(self, procurement):
        """ Do not group purchase order line if they are linked to different
        sale order line. The purpose is to compute the delivered quantities.
        """
        return procurement.values.get('sale_line_id'), super(StockRule, self)._get_procurements_to_merge_groupby(procurement)

    @api.model
    def _get_procurements_to_merge_sorted(self, procurement):
        return procurement.values.get('sale_line_id'), super(StockRule, self)._get_procurements_to_merge_sorted(procurement)


class ProcurementGroup(models.Model):
    _inherit = "procurement.group"

    @api.model
    def _get_rule_domain(self, location, values):
        if 'sale_line_id' in values and values.get('company_id'):
            return [('location_id', '=', location.id), ('action', '!=', 'push'), ('company_id', '=', values['company_id'].id)]
        else:
            return super(ProcurementGroup, self)._get_rule_domain(location, values)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import purchase
from . import res_company
from . import sale
from . import stock

```

## File: views\sale_order_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data>
        <record id="view_order_form_inherit_sale_stock" model="ir.ui.view">
            <field name="name">sale.order.form.sale.dropshipping</field>
            <field name="model">sale.order</field>
            <field name="inherit_id" ref="sale.view_order_form"/>
            <field name="groups_id" eval="[(4, ref('purchase.group_purchase_user'))]"/>
            <field name="arch" type="xml">
                <xpath expr="//page/field[@name='order_line']/form//field[@name='product_updatable']" position="after">
                    <field name="purchase_line_count" invisible="1"/>
                </xpath>
                <xpath expr="//page/field[@name='order_line']/form//field[@name='product_id']" position="attributes">
                   <attribute name="attrs">{'readonly': ['|', ('product_updatable', '=', False), ('purchase_line_count', '&gt;', 0)], 'required': [('display_type', '=', False)],}</attribute>
                </xpath>
                <xpath expr="//page/field[@name='order_line']/tree/field[@name='product_updatable']" position="after">
                    <field name="purchase_line_count" invisible="1"/>
                </xpath>
                <xpath expr="//page/field[@name='order_line']/tree/field[@name='product_id']" position="attributes">
                    <attribute name="attrs">{'readonly': ['|', ('product_updatable', '=', False), ('purchase_line_count', '&gt;', 0)], 'required': [('display_type', '=', False)],}</attribute>
                </xpath>
           </field>
        </record>
    </data>
</odoo>

```

