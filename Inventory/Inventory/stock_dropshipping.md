# Odoo Module: stock_dropshipping

Category: Inventory/Inventory

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import wizard

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
    'data': [
        'data/stock_data.xml',
        'views/sale_order_views.xml',
        'views/stock_picking_views.xml',
        'views/purchase_order_views.xml'
    ],
    'installable': True,
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

        <record id="route_drop_shipping" model='stock.route'>
            <field name="name">Dropship</field>
            <field name="sequence">20</field>
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

from odoo import api, models, fields


class PurchaseOrder(models.Model):
    _inherit = 'purchase.order'

    dropship_picking_count = fields.Integer("Dropship Count", compute='_compute_incoming_picking_count')

    @api.depends('picking_ids.is_dropship')
    def _compute_incoming_picking_count(self):
        super()._compute_incoming_picking_count()
        for order in self:
            dropship_count = len(order.picking_ids.filtered(lambda p: p.is_dropship))
            order.incoming_picking_count -= dropship_count
            order.dropship_picking_count = dropship_count

    def action_view_picking(self):
        return self._get_action_view_picking(self.picking_ids.filtered(lambda p: not p.is_dropship))

    def action_view_dropship(self):
        return self._get_action_view_picking(self.picking_ids.filtered(lambda p: p.is_dropship))


class PurchaseOrderLine(models.Model):
    _inherit = 'purchase.order.line'

    def _prepare_stock_moves(self, picking):
        res = super(PurchaseOrderLine, self)._prepare_stock_moves(picking)
        for re in res:
            re['sale_line_id'] = self.sale_line_id.id
            if self.order_id.dest_address_id:
                # In a dropshipping context we do not need the description of the purchase order or it will be displayed
                # in Delivery slip report and it may be confusing for the customer to see several times the same text (product name + description_picking).
                product = self.product_id.with_context(lang=self.order_id.dest_address_id.lang or self.env.user.lang)
                re['description_picking'] = product._get_description(
                    self.env['stock.picking.type'].browse(re['picking_type_id'])
                )
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
                'code': 'dropship',
                'default_location_src_id': self.env.ref('stock.stock_location_suppliers').id,
                'default_location_dest_id': self.env.ref('stock.stock_location_customers').id,
                'sequence_code': 'DS',
                'use_existing_lots': False,
            })
        if dropship_vals:
            self.env['stock.picking.type'].create(dropship_vals)

    @api.model
    def create_missing_dropship_picking_type(self):
        company_ids = self.env['res.company'].search([])
        company_has_dropship_picking_type = (
            self.env['stock.picking.type'].search([("code", "=", "dropship")]).company_id
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
                'location_dest_id': customer_location.id,
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

from odoo import models, fields, api


class SaleOrder(models.Model):
    _inherit = 'sale.order'

    dropship_picking_count = fields.Integer("Dropship Count", compute='_compute_picking_ids')

    @api.depends('picking_ids.is_dropship')
    def _compute_picking_ids(self):
        super()._compute_picking_ids()
        for order in self:
            dropship_count = len(order.picking_ids.filtered(lambda p: p.is_dropship))
            order.delivery_count -= dropship_count
            order.dropship_picking_count = dropship_count

    def action_view_delivery(self):
        return self._get_action_view_picking(self.picking_ids.filtered(lambda p: not p.is_dropship))

    def action_view_dropship(self):
        return self._get_action_view_picking(self.picking_ids.filtered(lambda p: p.is_dropship))


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

    @api.depends('purchase_line_count')
    def _compute_product_updatable(self):
        super()._compute_product_updatable()
        if self.env.user.has_group('purchase.group_purchase_user'):
            for line in self:
                if line.purchase_line_count > 0:
                    line.product_updatable = False

```

## File: models\stock.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, fields
from odoo.osv import expression


class StockRule(models.Model):
    _inherit = 'stock.rule'

    @api.model
    def _get_procurements_to_merge_groupby(self, procurement):
        """ Do not group purchase order line if they are linked to different
        sale order line. The purpose is to compute the delivered quantities.
        """
        return procurement.values.get('sale_line_id'), super(StockRule, self)._get_procurements_to_merge_groupby(procurement)

    def _get_partner_id(self, values, rule):
        route = self.env.ref('stock_dropshipping.route_drop_shipping', raise_if_not_found=False)
        if route and rule.route_id == route:
            return False
        return super()._get_partner_id(values, rule)


class ProcurementGroup(models.Model):
    _inherit = "procurement.group"

    @api.model
    def _get_rule_domain(self, locations, values):
        if 'sale_line_id' in values and values.get('company_id'):
            return [('location_dest_id', 'in', locations.ids), ('action', '!=', 'push'), ('company_id', '=', values['company_id'].id)]
        else:
            return super()._get_rule_domain(locations, values)

class StockPicking(models.Model):
    _inherit = 'stock.picking'

    is_dropship = fields.Boolean("Is a Dropship", compute='_compute_is_dropship')

    @api.depends('location_dest_id.usage', 'location_id.usage')
    def _compute_is_dropship(self):
        for picking in self:
            picking.is_dropship = picking.location_dest_id.usage == 'customer' and picking.location_id.usage == 'supplier'

    def _is_to_external_location(self):
        self.ensure_one()
        return super()._is_to_external_location() or self.is_dropship

class StockPickingType(models.Model):
    _inherit = 'stock.picking.type'

    code = fields.Selection(
        selection_add=[('dropship', 'Dropship')], ondelete={'dropship': lambda recs: recs.write({'code': 'outgoing', 'active': False})})

    @api.depends('default_location_src_id', 'default_location_dest_id')
    def _compute_warehouse_id(self):
        super()._compute_warehouse_id()
        for picking_type in self:
            if picking_type.default_location_src_id.usage == 'supplier' and picking_type.default_location_dest_id.usage == 'customer':
                picking_type.warehouse_id = False

    @api.depends('code')
    def _compute_show_picking_type(self):
        super()._compute_show_picking_type()
        for record in self:
            if record.code == "dropship":
                record.show_picking_type = True


class StockLot(models.Model):
    _inherit = 'stock.lot'

    def _compute_last_delivery_partner_id(self):
        super()._compute_last_delivery_partner_id()
        for lot in self:
            if lot.delivery_count > 0:
                last_delivery = max(lot.delivery_ids, key=lambda d: d.date_done)
                if last_delivery.is_dropship:
                    lot.last_delivery_partner_id = last_delivery.sale_id.partner_id

    def _get_outgoing_domain(self):
        res = super()._get_outgoing_domain()
        return expression.OR([res, [
            ('location_dest_id.usage', '=', 'customer'),
            ('location_id.usage', '=', 'supplier'),
        ]])

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

## File: views\purchase_order_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="purchase_order_form_inherit_stock_dropshipping" model="ir.ui.view">
        <field name="name">purchase.order.form.inherit.stock.dropshipping</field>
        <field name="model">purchase.order</field>
        <field name="inherit_id" ref="purchase.purchase_order_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[hasclass('oe_button_box')]/button[@name='action_view_picking']" position="before">
                <button type="object"
                    name="action_view_dropship"
                    class="oe_stat_button"
                    icon="fa-truck"
                    invisible="dropship_picking_count == 0" groups="stock.group_stock_user">
                    <field name="dropship_picking_count" widget="statinfo" string="Dropship"/>
                </button>
            </xpath>
            <xpath expr="//field[@name='picking_type_id']" position="attributes">
                <attribute name="domain">
                    [('code', 'in', ('incoming', 'dropship')), '|', ('warehouse_id', '=', False), ('warehouse_id.company_id', '=', company_id)]
                </attribute>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\sale_order_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="view_order_form_inherit_sale_stock" model="ir.ui.view">
            <field name="name">sale.order.form.sale.dropshipping</field>
            <field name="model">sale.order</field>
            <field name="inherit_id" ref="sale.view_order_form"/>
            <field name="arch" type="xml">
                <xpath expr="//button[@name='action_view_delivery']" position="before">
                    <t groups="purchase.group_purchase_user">
                        <button type="object"
                            name="action_view_dropship"
                            class="oe_stat_button"
                            icon="fa-truck"
                            invisible="dropship_picking_count == 0" groups="stock.group_stock_user">
                            <field name="dropship_picking_count" widget="statinfo" string="Dropship"/>
                        </button>
                    </t>
                </xpath>
           </field>
        </record>
    </data>
</odoo>

```

## File: views\stock_picking_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_picking_internal_search_inherit_stock_dropshipping" model="ir.ui.view">
        <field name="name">stock.picking.search</field>
        <field name="model">stock.picking</field>
        <field name="inherit_id" ref="stock.view_picking_internal_search"/>
        <field name="arch" type="xml">
            <xpath expr="//filter[@name='internal']" position="after">
                <filter name="dropships" string="Dropships" domain="[('picking_type_code', '=', 'dropship')]"/>
            </xpath>
        </field>
    </record>

    <record id="action_picking_tree_dropship" model="ir.actions.act_window">
        <field name="name">Dropships</field>
        <field name="res_model">stock.picking</field>
        <field name="view_mode">tree,kanban,form,calendar</field>
        <field name="domain"></field>
        <field name="context">{'contact_display': 'partner_address', 'default_company_id': allowed_company_ids[0], 'restricted_picking_type_code': 'dropship', 'search_default_dropships': 1}</field>
        <field name="search_view_id" ref="stock.view_picking_internal_search"/>
    </record>

    <menuitem id="dropship_picking" name="Dropships" parent="stock.menu_stock_transfers"
        action="action_picking_tree_dropship" sequence="30"
        groups="stock.group_stock_manager,stock.group_stock_user"/>
</odoo>

```

## File: wizard\product_replenish.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.osv import expression


class ProductReplenish(models.TransientModel):
    _inherit = 'product.replenish'

    def _get_allowed_route_domain(self):
        domains = super()._get_allowed_route_domain()
        return expression.AND([domains, [('id', '!=', self.env.ref('stock_dropshipping.route_drop_shipping', raise_if_not_found=False).id)]])

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import product_replenish

```

