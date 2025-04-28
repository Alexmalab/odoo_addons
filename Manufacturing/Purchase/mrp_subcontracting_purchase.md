# Odoo Module: mrp_subcontracting_purchase

Category: Manufacturing/Purchase

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
    'name': 'Purchase and Subcontracting Management',
    'version': '0.1',
    'category': 'Manufacturing/Purchase',
    'description': """
This bridge module adds some smart buttons between Purchase and Subcontracting
    """,
    'depends': ['mrp_subcontracting', 'purchase_mrp'],
    'data': [
        'views/purchase_order_views.xml',
        'views/stock_picking_views.xml',
    ],
    'demo': [
        'data/mrp_subcontracting_purchase_demo.xml',
    ],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\mrp_subcontracting_purchase_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="purchase_order_subcontracting" model="purchase.order">
            <field name="partner_id" ref="base.partner_demo_portal"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="state">purchase</field>
            <field name="order_line" model="purchase.order.line" eval="[(5, 0, 0),
                (0, 0, {
                    'product_id': ref('product.product_delivery_02'),
                    'price_unit': 10.0,
                    'name': 'Office Lamp',
                    'product_qty': 5.0,
                    'date_planned': time.strftime('%Y-%m-%d')}),
            ]"/>
        </record>
    </data>
</odoo>

```

## File: models\account_move_line.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.tools import float_is_zero


class AccountMoveLine(models.Model):
    _inherit = 'account.move.line'

    def _get_price_unit_val_dif_and_relevant_qty(self):
        price_unit_val_dif, relevant_qty = super()._get_price_unit_val_dif_and_relevant_qty()
        if self.product_id.cost_method == 'standard' and self.purchase_line_id:
            components_cost = 0
            subcontract_production = self.purchase_line_id.move_ids._get_subcontract_production()
            components_cost -= sum(subcontract_production.move_raw_ids.stock_valuation_layer_ids.mapped('value'))
            qty = sum(mo.product_uom_id._compute_quantity(mo.qty_producing, self.product_uom_id) for mo in subcontract_production if mo.state == 'done')
            if not float_is_zero(qty, precision_rounding=self.product_uom_id.rounding):
                price_unit_val_dif = price_unit_val_dif + components_cost / qty
        return price_unit_val_dif, relevant_qty

    def _get_valued_in_moves(self):
        res = super()._get_valued_in_moves()
        # subcontracted move valuations are not linked to the PO move but its orig move (the MO finished move)
        res |= res.filtered(lambda m: m.is_subcontract).move_orig_ids
        return res

```

## File: models\purchase_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields, api


class PurchaseOrder(models.Model):
    _inherit = 'purchase.order'

    subcontracting_resupply_picking_count = fields.Integer(
        "Count of Subcontracting Resupply", compute='_compute_subcontracting_resupply_picking_count',
        help="Count of Subcontracting Resupply for component")

    @api.depends('order_line.move_ids')
    def _compute_subcontracting_resupply_picking_count(self):
        for purchase in self:
            purchase.subcontracting_resupply_picking_count = len(purchase._get_subcontracting_resupplies())

    def action_view_subcontracting_resupply(self):
        return self._get_action_view_picking(self._get_subcontracting_resupplies())

    def _get_subcontracting_resupplies(self):
        moves_subcontracted = self.order_line.move_ids.filtered(lambda m: m.is_subcontract)
        subcontracted_productions = moves_subcontracted.move_orig_ids.production_id
        return subcontracted_productions.picking_ids

    def _get_mrp_productions(self, **kwargs):
        productions = super()._get_mrp_productions(**kwargs)
        if kwargs.get('remove_archived_picking_types', True):
            productions = productions.filtered(lambda production: production.with_context(active_test=False).picking_type_id.active)
        return productions

```

## File: models\stock_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class StockMove(models.Model):
    _inherit = 'stock.move'

    def _is_purchase_return(self):
        res = super()._is_purchase_return()
        return res or self._is_subcontract_return()

```

## File: models\stock_picking.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields, api, _


class StockPicking(models.Model):
    _inherit = 'stock.picking'

    subcontracting_source_purchase_count = fields.Integer(
        "Number of subcontracting PO Source", compute='_compute_subcontracting_source_purchase_count',
        help="Number of subcontracting Purchase Order Source")

    @api.depends('move_ids.move_dest_ids.raw_material_production_id')
    def _compute_subcontracting_source_purchase_count(self):
        for picking in self:
            picking.subcontracting_source_purchase_count = len(picking._get_subcontracting_source_purchase())

    def action_view_subcontracting_source_purchase(self):
        purchase_order_ids = self._get_subcontracting_source_purchase().ids
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
                'name': _("Source PO of %s", self.name),
                'domain': [('id', 'in', purchase_order_ids)],
                'view_mode': 'tree,form',
            })
        return action

    def _get_subcontracting_source_purchase(self):
        moves_subcontracted = self.move_ids.move_dest_ids.raw_material_production_id.move_finished_ids.move_dest_ids.filtered(lambda m: m.is_subcontract)
        return moves_subcontracted.purchase_line_id.order_id

```

## File: models\stock_rule.py

```python
from odoo import models, _


class StockRule(models.Model):
    _inherit = 'stock.rule'

    def _get_lead_days(self, product, **values):
        """For subcontracting, we need to consider both vendor lead time and
        manufacturing lead time, and DTPMO (Days To Prepare MO).
        Subcontracting delay =
            max(Vendor lead time, Manufacturing lead time + DTPMO) + Days to Purchase + Purchase security lead time
        """
        bypass_delay_description = self.env.context.get('bypass_delay_description')
        buy_rule = self.filtered(lambda r: r.action == 'buy')
        seller = 'supplierinfo' in values and values['supplierinfo'] or product.with_company(buy_rule.company_id)._select_seller(quantity=None)
        if not buy_rule or not seller:
            return super()._get_lead_days(product, **values)
        seller = seller[0]
        bom = self.env['mrp.bom'].sudo()._bom_subcontract_find(
            product,
            company_id=buy_rule.picking_type_id.company_id.id,
            bom_type='subcontract',
            subcontractor=seller.partner_id)
        if not bom:
            return super()._get_lead_days(product, **values)

        delays, delay_description = super(StockRule, self - buy_rule)._get_lead_days(product, **values)
        extra_delays, extra_delay_description = super(StockRule, buy_rule.with_context(ignore_vendor_lead_time=True, ignore_global_visibility_days=True))._get_lead_days(product, **values)
        if seller.delay >= bom.produce_delay + bom.days_to_prepare_mo:
            delays['total_delay'] += seller.delay
            delays['purchase_delay'] += seller.delay
            if not bypass_delay_description:
                delay_description.append((_('Vendor Lead Time'), _('+ %d day(s)', seller.delay)))
        else:
            manufacture_delay = bom.produce_delay
            delays['total_delay'] += manufacture_delay
            # set manufacture_delay to purchase_delay so that PO can be created with correct date
            delays['purchase_delay'] += manufacture_delay
            if not bypass_delay_description:
                delay_description.append((_('Manufacturing Lead Time'), _('+ %d day(s)', manufacture_delay)))
            days_to_order = bom.days_to_prepare_mo
            delays['total_delay'] += days_to_order
            # add dtpmo to purchase_delay so that PO can be created with correct date
            delays['purchase_delay'] += days_to_order
            if not bypass_delay_description:
                extra_delay_description.append((_('Days to Supply Components'), _('+ %d day(s)', days_to_order)))

        for key, value in extra_delays.items():
            delays[key] += value
        return delays, delay_description + extra_delay_description

```

## File: models\stock_valuation_layer.py

```python


from odoo import models


class StockValuationLayer(models.Model):
    _inherit = 'stock.valuation.layer'

    def _get_layer_price_unit(self):
        """ For a subcontracted product, we want a way to get the subcontracting cost (the price on the PO)
            This override deducts the value of subcomponents from the layer price.
        """
        components_price = 0
        production = self.stock_move_id.production_id
        if production.subcontractor_id and production.state == 'done':
            # each layer has a quantity and price for each move, to get the correct component price for each move
            # we need to get the components used for each quantity
            for move in production.move_raw_ids:
                components_price += sum(move.sudo().stock_valuation_layer_ids.mapped('value')) / production.product_uom_qty
        # the move valuation is negative (out moves) therefore we we add the negative components_price instead of subtracting
        return super()._get_layer_price_unit() + components_price

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_move_line
from . import purchase_order
from . import stock_move
from . import stock_picking
from . import stock_rule
from . import stock_valuation_layer

```

## File: report\mrp_report_bom_structure.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class ReportBomStructure(models.AbstractModel):
    _inherit = 'report.mrp.report_bom_structure'

    @api.model
    def _is_buy_route(self, rules, product, bom):
        return super()._is_buy_route(rules, product, bom) and (not bom or bom.type != 'subcontract')

    @api.model
    def _get_resupply_availability(self, route_info, components):
        resupply_state, resupply_delay = super()._get_resupply_availability(route_info, components)
        if route_info.get('route_type') == 'subcontract' and resupply_delay:
            # always add `Purchase security lead days` and `Days to Purchase`
            extra_delay = route_info['bom'].company_id.po_lead + route_info['bom'].company_id.days_to_purchase
            route_info['lead_time'] += extra_delay
            route_info['manufacture_delay'] += extra_delay
            subcontract_delay = resupply_delay + extra_delay
            return ('estimated', subcontract_delay)
        return (resupply_state, resupply_delay)

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import mrp_report_bom_structure

```

## File: views\purchase_order_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="purchase_order_form_mrp_subcontracting_purchase" model="ir.ui.view">
        <field name="name">purchase.order.inherited.form.mrp.subcontracting.purchase</field>
        <field name="model">purchase.order</field>
        <field name="inherit_id" ref="purchase.purchase_order_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[hasclass('oe_button_box')]/button[@name='action_view_picking']" position="before">
                <button 
                    class="oe_stat_button" name="action_view_subcontracting_resupply" 
                    type="object" icon="fa-truck" invisible="subcontracting_resupply_picking_count == 0" groups="stock.group_stock_user">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_value"><field name="subcontracting_resupply_picking_count"/></span>
                        <span class="o_stat_text">Resupply</span>
                    </div>
                </button>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\stock_picking_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="stock_picking_form_mrp_subcontracting" model="ir.ui.view">
        <field name="name">stock.picking.inherited.form.mrp.subcontracting</field>
        <field name="model">stock.picking</field>
        <field name="inherit_id" ref="stock.view_picking_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[hasclass('oe_button_box')]" position="inside">
                <button 
                    class="oe_stat_button" name="action_view_subcontracting_source_purchase" 
                    type="object" icon="fa-credit-card" invisible="subcontracting_source_purchase_count == 0" groups="stock.group_stock_user">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_value"><field name="subcontracting_source_purchase_count"/></span>
                        <span class="o_stat_text">Source PO</span>
                    </div>
                </button>
            </xpath>
        </field>
    </record>
</odoo>

```

