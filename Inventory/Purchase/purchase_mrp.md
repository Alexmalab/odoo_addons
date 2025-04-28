# Odoo Module: purchase_mrp

Category: Inventory/Purchase

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import report

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


{
    'name': 'Purchase and MRP Management',
    'version': '1.0',
    'category': 'Inventory/Purchase',
    'description': """
This module provides facility to the user to install mrp and purchase modules at a time.
========================================================================================

It is basically used when we want to keep track of production orders generated
from purchase order.
    """,
    'data': [
        'views/mrp_bom_views.xml',
        'views/purchase_order_views.xml',
        'views/mrp_production_views.xml',
        'views/stock_orderpoint_views.xml',
        'security/ir.model.access.csv',
    ],
    'demo': [
        'data/purchase_mrp_demo.xml',
    ],
    'depends': ['mrp', 'purchase_stock'],
    'installable': True,
    'auto_install': True,
    'assets': {
        'web.assets_backend': [
            'mrp/static/src/**/*.js',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\purchase_mrp_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="mrp.product_product_computer_desk_leg" model="product.product">
            <field name="route_ids" eval="[Command.link(ref('purchase_stock.route_warehouse0_buy'))]"></field>
        </record>
    </data>
</odoo>

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class AccountMoveLine(models.Model):
    _inherit = "account.move.line"

    def _get_stock_valuation_layers(self, move):
        """ Do not handle the invoice correction for kit. It has to be done
        manually """
        layers = super()._get_stock_valuation_layers(move)
        return layers.filtered(lambda svl: svl.product_id == self.product_id)

```

## File: models\mrp_bom.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError
from odoo.tools import float_compare


class MrpBom(models.Model):
    _inherit = 'mrp.bom'

    @api.constrains('product_id', 'product_tmpl_id', 'bom_line_ids', 'byproduct_ids', 'operation_ids')
    def _check_bom_lines(self):
        res = super()._check_bom_lines()
        for bom in self:
            if all(not bl.cost_share for bl in bom.bom_line_ids):
                continue
            if any(bl.cost_share < 0 for bl in bom.bom_line_ids):
                raise UserError(_("Components cost share have to be positive or equals to zero."))
            if float_compare(sum(bom.bom_line_ids.mapped('cost_share')), 100, precision_digits=2) != 0:
                raise UserError(_("The total cost share for a BoM's component have to be 100"))
        return res


class MrpBomLine(models.Model):
    _inherit = 'mrp.bom.line'

    cost_share = fields.Float(
        "Cost Share (%)", digits=(5, 2),  # decimal = 2 is important for rounding calculations!!
        help="The percentage of the component repartition cost when purchasing a kit."
             "The total of all components' cost have to be equal to 100.")

    def _get_cost_share(self):
        self.ensure_one()
        if self.cost_share:
            return self.cost_share / 100
        bom = self.bom_id
        bom_lines_without_cost_share = bom.bom_line_ids.filtered(lambda bl: not bl.cost_share)
        return 1 / len(bom_lines_without_cost_share)

```

## File: models\mrp_production.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, Command, fields, models, _


class MrpProduction(models.Model):
    _inherit = 'mrp.production'

    purchase_order_count = fields.Integer(
        "Count of generated PO",
        compute='_compute_purchase_order_count',
        groups='purchase.group_purchase_user')

    @api.depends('procurement_group_id.stock_move_ids.created_purchase_line_ids.order_id', 'procurement_group_id.stock_move_ids.move_orig_ids.purchase_line_id.order_id')
    def _compute_purchase_order_count(self):
        for production in self:
            production.purchase_order_count = len(production.procurement_group_id.stock_move_ids.created_purchase_line_ids.order_id |
                                                  production.procurement_group_id.stock_move_ids.move_orig_ids.purchase_line_id.order_id)

    def action_view_purchase_orders(self):
        self.ensure_one()
        purchase_order_ids = (self.procurement_group_id.stock_move_ids.created_purchase_line_ids.order_id | self.procurement_group_id.stock_move_ids.move_orig_ids.purchase_line_id.order_id).ids
        action = {
            'res_model': 'purchase.order',
            'type': 'ir.actions.act_window',
        }
        if len(purchase_order_ids) == 1:
            action.update({
                'view_mode': 'form',
                'res_id': purchase_order_ids[0],
            })
        else:
            action.update({
                'name': _("Purchase Order generated from %s", self.name),
                'domain': [('id', 'in', purchase_order_ids)],
                'view_mode': 'tree,form',
            })
        return action

    def _get_document_iterate_key(self, move_raw_id):
        iterate_key = super(MrpProduction, self)._get_document_iterate_key(move_raw_id)
        if not iterate_key and move_raw_id.created_purchase_line_ids:
            iterate_key = 'created_purchase_line_ids'
        return iterate_key

    def _prepare_merge_orig_links(self):
        origs = super()._prepare_merge_orig_links()
        for move in self.move_raw_ids:
            if not move.move_orig_ids or not move.created_purchase_line_ids:
                continue
            origs[move.bom_line_id.id].setdefault('created_purchase_line_ids', set()).update(move.created_purchase_line_ids.ids)
        for vals in origs.values():
            if vals.get('created_purchase_line_ids'):
                vals['created_purchase_line_ids'] = [Command.set(vals['created_purchase_line_ids'])]
            else:
                vals['created_purchase_line_ids'] = []
        return origs

```

## File: models\purchase.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict

from odoo import api, fields, models, _
from odoo.tools import OrderedSet


class PurchaseOrder(models.Model):
    _inherit = 'purchase.order'

    mrp_production_count = fields.Integer(
        "Count of MO Source",
        compute='_compute_mrp_production_count',
        groups='mrp.group_mrp_user')

    @api.depends('order_line.move_dest_ids.group_id.mrp_production_ids')
    def _compute_mrp_production_count(self):
        for purchase in self:
            purchase.mrp_production_count = len(purchase._get_mrp_productions())

    def _get_mrp_productions(self, **kwargs):
        return self.order_line.move_dest_ids.group_id.mrp_production_ids | self.order_line.move_ids.move_dest_ids.group_id.mrp_production_ids

    def action_view_mrp_productions(self):
        self.ensure_one()
        mrp_production_ids = self._get_mrp_productions().ids
        action = {
            'res_model': 'mrp.production',
            'type': 'ir.actions.act_window',
        }
        if len(mrp_production_ids) == 1:
            action.update({
                'view_mode': 'form',
                'res_id': mrp_production_ids[0],
            })
        else:
            action.update({
                'name': _("Manufacturing Source of %s", self.name),
                'domain': [('id', 'in', mrp_production_ids)],
                'view_mode': 'tree,form',
            })
        return action


class PurchaseOrderLine(models.Model):
    _inherit = 'purchase.order.line'

    def _compute_qty_received(self):
        kit_lines = self.env['purchase.order.line']
        lines_stock = self.filtered(lambda l: l.qty_received_method == 'stock_moves' and l.move_ids and l.state != 'cancel')
        product_by_company = defaultdict(OrderedSet)
        for line in lines_stock:
            product_by_company[line.company_id].add(line.product_id.id)
        kits_by_company = {
            company: self.env['mrp.bom']._bom_find(self.env['product.product'].browse(product_ids), company_id=company.id, bom_type='phantom')
            for company, product_ids in product_by_company.items()
        }
        for line in lines_stock:
            kit_bom = kits_by_company[line.company_id].get(line.product_id)
            if kit_bom:
                moves = line.move_ids.filtered(lambda m: m.state == 'done' and not m.scrapped)
                order_qty = line.product_uom._compute_quantity(line.product_uom_qty, kit_bom.product_uom_id)
                filters = {
                    'incoming_moves': lambda m: m.location_id.usage == 'supplier' and (not m.origin_returned_move_id or (m.origin_returned_move_id and m.to_refund)),
                    'outgoing_moves': lambda m: m.location_id.usage != 'supplier' and m.to_refund
                }
                line.qty_received = moves._compute_kit_quantities(line.product_id, order_qty, kit_bom, filters)
                kit_lines += line
        super(PurchaseOrderLine, self - kit_lines)._compute_qty_received()

    def _get_upstream_documents_and_responsibles(self, visited):
        return [(self.order_id, self.order_id.user_id, visited)]

    def _get_qty_procurement(self):
        self.ensure_one()
        # Specific case when we change the qty on a PO for a kit product.
        # We don't try to be too smart and keep a simple approach: we compare the quantity before
        # and after update, and return the difference. We don't take into account what was already
        # sent, or any other exceptional case.
        bom = self.env['mrp.bom'].sudo()._bom_find(self.product_id, bom_type='phantom')[self.product_id]
        if bom and 'previous_product_qty' in self.env.context:
            return self.env.context['previous_product_qty'].get(self.id, 0.0)
        return super()._get_qty_procurement()

    def _get_move_dests_initial_demand(self, move_dests):
        kit_bom = self.env['mrp.bom']._bom_find(self.product_id, bom_type='phantom')[self.product_id]
        if kit_bom:
            filters = {'incoming_moves': lambda m: True, 'outgoing_moves': lambda m: False}
            return move_dests._compute_kit_quantities(self.product_id, self.product_qty, kit_bom, filters)
        return super()._get_move_dests_initial_demand(move_dests)

```

## File: models\stock_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, models, fields
from odoo.tools.float_utils import float_is_zero, float_round
from odoo.exceptions import UserError


class StockMove(models.Model):
    _inherit = 'stock.move'

    def _prepare_phantom_move_values(self, bom_line, product_qty, quantity_done):
        vals = super(StockMove, self)._prepare_phantom_move_values(bom_line, product_qty, quantity_done)
        if self.purchase_line_id:
            vals['purchase_line_id'] = self.purchase_line_id.id
        return vals

    def _get_price_unit(self):
        if self.product_id == self.purchase_line_id.product_id or not self.bom_line_id or self._should_ignore_pol_price():
            return super()._get_price_unit()
        line = self.purchase_line_id
        # price_unit here with uom of product
        kit_price_unit = line._get_gross_price_unit()
        bom_line = self.bom_line_id
        bom = bom_line.bom_id
        if line.currency_id != self.company_id.currency_id:
            kit_price_unit = line.currency_id._convert(kit_price_unit, self.company_id.currency_id, self.company_id, fields.Date.context_today(self), round=False)
        cost_share = self.bom_line_id._get_cost_share()
        uom_factor = 1.0
        kit_product = bom.product_id or bom.product_tmpl_id

        # Convert uom from product_uom to bom_uom for kit product
        uom_factor = bom.product_uom_id._compute_quantity(uom_factor, kit_product.uom_id)

        # Convert uom from bom_line_uom to product_uom for bom_line
        uom_factor = bom_line.product_id.uom_id._compute_quantity(uom_factor, bom_line.product_uom_id)

        return (kit_price_unit * cost_share * uom_factor * bom.product_qty / bom_line.product_qty)

    def _get_valuation_price_and_qty(self, related_aml, to_curr):
        valuation_price_unit_total, valuation_total_qty = super()._get_valuation_price_and_qty(related_aml, to_curr)
        boms = self.env['mrp.bom']._bom_find(related_aml.product_id, company_id=related_aml.company_id.id, bom_type='phantom')
        if related_aml.product_id in boms:
            kit_bom = boms[related_aml.product_id]
            order_qty = related_aml.product_id.uom_id._compute_quantity(related_aml.quantity, kit_bom.product_uom_id)
            filters = {
                'incoming_moves': lambda m: m.location_id.usage == 'supplier' and (not m.origin_returned_move_id or (m.origin_returned_move_id and m.to_refund)),
                'outgoing_moves': lambda m: m.location_id.usage != 'supplier' and m.to_refund
            }
            valuation_total_qty = self._compute_kit_quantities(related_aml.product_id, order_qty, kit_bom, filters)
            valuation_total_qty = kit_bom.product_uom_id._compute_quantity(valuation_total_qty, related_aml.product_id.uom_id)
            if float_is_zero(valuation_total_qty, precision_rounding=related_aml.product_uom_id.rounding or related_aml.product_id.uom_id.rounding):
                raise UserError(_('Odoo is not able to generate the anglo saxon entries. The total valuation of %s is zero.', related_aml.product_id.display_name))
        return valuation_price_unit_total, valuation_total_qty

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_move
from . import purchase
from . import mrp_production
from . import mrp_bom
from . import stock_move

```

## File: report\mrp_report_bom_structure.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models
from odoo.tools import float_compare

class ReportBomStructure(models.AbstractModel):
    _inherit = 'report.mrp.report_bom_structure'

    @api.model
    def _format_route_info(self, rules, rules_delay, warehouse, product, bom, quantity):
        res = super()._format_route_info(rules, rules_delay, warehouse, product, bom, quantity)
        if self._is_buy_route(rules, product, bom):
            buy_rules = [rule for rule in rules if rule.action == 'buy']
            supplier = product._select_seller(quantity=quantity, uom_id=product.uom_id)
            if not supplier:
                # If no vendor found for the right quantity, we still want to display a vendor for the lead times
                supplier = product._select_seller(quantity=None, uom_id=product.uom_id)
            parent_bom = self.env.context.get('parent_bom')
            purchase_lead = parent_bom.company_id.days_to_purchase + parent_bom.company_id.po_lead if parent_bom and parent_bom.company_id else 0
            if supplier:
                qty_supplier_uom = product.uom_id._compute_quantity(quantity, supplier.product_uom)
                return {
                    'route_type': 'buy',
                    'route_name': buy_rules[0].route_id.display_name,
                    'route_detail': supplier.display_name,
                    'lead_time': supplier.delay + rules_delay + purchase_lead,
                    'supplier_delay': supplier.delay + rules_delay + purchase_lead,
                    'supplier': supplier,
                    'route_alert': float_compare(qty_supplier_uom, supplier.min_qty, precision_rounding=product.uom_id.rounding) < 0,
                    'qty_checked': quantity,
                }
        return res

    @api.model
    def _is_resupply_rules(self, rules, bom):
        return super()._is_resupply_rules(rules, bom) or any(rule.action == 'buy' for rule in rules)

    @api.model
    def _is_buy_route(self, rules, product, bom):
        return any(rule for rule in rules if rule.action == 'buy' and product.seller_ids)

    @api.model
    def _get_resupply_availability(self, route_info, components):
        if route_info.get('route_type') == 'buy':
            supplier_delay = route_info.get('supplier_delay', 0)
            return ('estimated', supplier_delay)
        return super()._get_resupply_availability(route_info, components)

```

## File: report\mrp_report_mo_overview.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models

class ReportMoOverview(models.AbstractModel):
    _inherit = 'report.mrp.report_mo_overview'

    def _get_extra_replenishments(self, product):
        res = super()._get_extra_replenishments(product)
        domain = [('state', 'in', ['draft', 'sent', 'to approve']), ('product_id', '=', product.id)]
        warehouse_id = self.env['stock.warehouse']._get_warehouse_id_from_context()
        if warehouse_id:
            domain += [('order_id.picking_type_id.warehouse_id', '=', warehouse_id)]
        po_lines = self.env['purchase.order.line'].search(domain, order='date_planned, id')

        for po_line in po_lines:
            line_qty = po_line.product_qty
            # Need to fetch every move connected to a manufacturing order from this PO line. This can happen when:
            # - Multiple MOs are linked to a single PO line (e.g. Same MTO component for multiple MO).
            # - A MO has a backorder / is splitted.
            dest_moves = self.env['stock.move'].browse(po_line.move_dest_ids._rollup_move_dests())
            for move in dest_moves:
                if not move.raw_material_production_id:
                    continue
                prod_qty = min(line_qty, move.product_uom._compute_quantity(move.product_uom_qty, po_line.product_uom))
                res.append(self._format_extra_replenishment(po_line, prod_qty, move.raw_material_production_id.id))
                line_qty -= prod_qty
            if line_qty:
                res.append(self._format_extra_replenishment(po_line, line_qty))

        return res

    def _format_extra_replenishment(self, po_line, quantity, production_id=False):
        po = po_line.order_id
        price = po_line.taxes_id.with_context(round=False).compute_all(
            po_line.price_unit, currency=po.currency_id, quantity=quantity, product=po_line.product_id, partner=po.partner_id
        )['total_void']
        return {
            '_name': 'purchase.order',
            'id': po.id,
            'cost': price,
            'quantity': quantity,
            'uom': po_line.product_uom,
            'production_id': production_id
        }

    def _get_replenishment_receipt(self, doc_in, components):
        res = super()._get_replenishment_receipt(doc_in, components)
        if doc_in._name == 'purchase.order':
            if doc_in.state != 'purchase':
                return self._format_receipt_date('estimated', doc_in.date_planned)
            in_pickings = doc_in.picking_ids.filtered(lambda p: p.state not in ('done', 'cancel'))
            planned_date = max(in_pickings.mapped('scheduled_date')) if in_pickings else doc_in.date_planned
            return self._format_receipt_date('expected', planned_date)
        return res

    def _get_resupply_data(self, rules, rules_delay, quantity, uom_id, product, production):
        res = super()._get_resupply_data(rules, rules_delay, quantity, uom_id, product, production)
        if any(rule for rule in rules if rule.action == 'buy' and product.seller_ids):
            supplier = product._select_seller(quantity=quantity, uom_id=product.uom_id)
            if supplier:
                return {
                    'delay': supplier.delay + rules_delay,
                    'cost': supplier.price * uom_id._compute_quantity(quantity, supplier.product_uom),
                    'currency': supplier.currency_id,
                }
        return res

    def _is_doc_in_done(self, doc_in):
        if doc_in._name == 'purchase.order':
            return doc_in.state == 'purchase' and all(move.state in ('done', 'cancel') for move in doc_in.order_line.move_ids)
        return super()._is_doc_in_done(doc_in)

    def _get_origin(self, move):
        if move.purchase_line_id:
            return move.purchase_line_id.order_id
        return super()._get_origin(move)

    def _get_replenishment_mo_cost(self, product, quantity, uom_id, currency, move_in=False):
        if move_in and move_in.purchase_line_id:
            po_line = move_in.purchase_line_id
            po = po_line.order_id
            price = po_line.taxes_id.with_context(round=False).compute_all(
                po_line.price_unit, currency=po.currency_id, quantity=uom_id._compute_quantity(quantity, move_in.purchase_line_id.product_uom),
                product=po_line.product_id, partner=po.partner_id
            )['total_void']
            price = po_line.currency_id._convert(price, currency, (move_in.company_id or self.env.company), fields.Date.today())
            return currency.round(price)
        return super()._get_replenishment_mo_cost(product, quantity, uom_id, currency, move_in)

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import mrp_report_bom_structure
from . import mrp_report_mo_overview

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_mrp_bom_purchase_user,mrp.bom,mrp.model_mrp_bom,purchase.group_purchase_user,1,0,0,0
access_mrp_bom_line_purchase_user,mrp.bom.line,mrp.model_mrp_bom_line,purchase.group_purchase_user,1,0,0,0

```

## File: static\src\components\bom_overview_line\mrp_bom_overview_line.js

```javascript
/** @odoo-module **/

import { patch } from "@web/core/utils/patch";
import { BomOverviewLine } from "@mrp/components/bom_overview_line/mrp_bom_overview_line";

patch(BomOverviewLine.prototype, {
    /**
     * @override
     */
    async goToRoute(routeType) {
        if (routeType == "buy") {
            return this.goToAction(this.data.link_id, this.data.link_model);
        }
        return super.goToRoute(...arguments);
    }
});

```

## File: views\mrp_bom_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>
    <record id="mrp_bom_form_view" model="ir.ui.view">
        <field name="name">mrp.bom.view.form.inherited.purchase.mrp</field>
        <field name="model">mrp.bom</field>
        <field name="inherit_id" ref="mrp.mrp_bom_form_view"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='bom_line_ids']/tree//field[@name='manual_consumption']" position="after">
                <field name="cost_share" optional="hidden" column_invisible="parent.type != 'phantom'"/>
            </xpath>
        </field>
    </record>
</data></odoo>
```

## File: views\mrp_production_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>
    <record id="mrp_production_form_view_purchase" model="ir.ui.view">
        <field name="name">mrp.production.inherited.form.purchase</field>
        <field name="model">mrp.production</field>
        <field name="priority">32</field>
        <field name="inherit_id" ref="mrp.mrp_production_form_view"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@name='action_view_mrp_production_childs']" position="before">
                <button class="oe_stat_button" name="action_view_purchase_orders" type="object" icon="fa-credit-card" groups="purchase.group_purchase_user" invisible="purchase_order_count == 0">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_value"><field name="purchase_order_count"/></span>
                        <span class="o_stat_text">Purchases</span>
                    </div>
                </button>
            </xpath>
        </field>
    </record>
</data></odoo>

```

## File: views\purchase_order_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>
    <record id="purchase_order_form_mrp" model="ir.ui.view">
        <field name="name">purchase.order.inherited.form.mrp</field>
        <field name="model">purchase.order</field>
        <field name="inherit_id" ref="purchase.purchase_order_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='button_box']" position="inside">
                <button class="oe_stat_button" name="action_view_mrp_productions" type="object" icon="fa-wrench" invisible="mrp_production_count == 0" groups="mrp.group_mrp_user">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_value"><field name="mrp_production_count"/></span>
                        <span class="o_stat_text">Manufacturing</span>
                    </div>
                </button>
            </xpath>
        </field>
    </record>
</data></odoo>

```

## File: views\stock_orderpoint_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_warehouse_orderpoint_tree_editable" model="ir.ui.view">
        <field name="name">stock.warehouse.orderpoint.tree.editable.inherit.show_route</field>
        <field name="model">stock.warehouse.orderpoint</field>
        <field name="inherit_id" ref="stock.view_warehouse_orderpoint_tree_editable"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='route_id']" position="attributes">
                <attribute name="optional">show</attribute>
            </xpath>
        </field>
    </record>
</odoo>

```

