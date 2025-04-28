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

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import stock_picking
from . import stock_move
from . import purchase_order

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
                    type="object" icon="fa-truck" attrs="{'invisible': [('subcontracting_resupply_picking_count', '=', 0)]}" groups="stock.group_stock_user">
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
                    type="object" icon="fa-credit-card" attrs="{'invisible': [('subcontracting_source_purchase_count', '=', 0)]}" groups="stock.group_stock_user">
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

