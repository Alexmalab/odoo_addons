# Odoo Module: sale_mrp

Category: Hidden

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
    'name': 'Sales and MRP Management',
    'version': '1.0',
    'category': 'Hidden',
    'description': """
This module provides facility to the user to install mrp and sales modulesat a time.
====================================================================================

It is basically used when we want to keep track of production orders generated
from sales order. It adds sales name and sales Reference on production order.
    """,
    'depends': ['mrp', 'sale_stock'],
    'data': [
        'security/ir.model.access.csv',
        'views/mrp_production_views.xml',
        'views/sale_order_views.xml',
        'views/sale_portal_templates.xml'
    ],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-

from odoo import models


class AccountMoveLine(models.Model):
    _inherit = "account.move.line"

    def _stock_account_get_anglo_saxon_price_unit(self):
        price_unit = super(AccountMoveLine, self)._stock_account_get_anglo_saxon_price_unit()

        so_line = self.sale_line_ids and self.sale_line_ids[-1] or False
        if so_line:
            # We give preference to the bom in the stock moves for the sale order lines
            # If there are changes in BOMs between the stock moves creation and the
            # invoice validation a wrong price will be taken
            boms = so_line.move_ids.filtered(lambda m: m.state != 'cancel').mapped('bom_line_id.bom_id').filtered(lambda b: b.type == 'phantom')
            if boms:
                bom = boms.filtered(lambda b: b.product_id == so_line.product_id or b.product_tmpl_id == so_line.product_id.product_tmpl_id)
                if not bom:
                    # In the case where the product has no direct component in its bom, it won't be present in the stock moves boms.
                    # We then take the first bom of the product.
                    bom = self.env['mrp.bom']._bom_find(products=so_line.product_id, company_id=so_line.company_id.id, bom_type='phantom')[so_line.product_id]
                is_line_reversing = self.move_id.move_type == 'out_refund'
                qty_to_invoice = self.product_uom_id._compute_quantity(self.quantity, self.product_id.uom_id)
                account_moves = so_line.invoice_lines.move_id.filtered(lambda m: m.state == 'posted' and bool(m.reversed_entry_id) == is_line_reversing)
                posted_invoice_lines = account_moves.line_ids.filtered(lambda l: l.display_type == 'cogs' and l.product_id == self.product_id and l.balance > 0)
                qty_invoiced = sum([x.product_uom_id._compute_quantity(x.quantity, x.product_id.uom_id) for x in posted_invoice_lines])
                reversal_cogs = posted_invoice_lines.move_id.reversal_move_id.line_ids.filtered(lambda l: l.display_type == 'cogs' and l.product_id == self.product_id and l.balance > 0)
                qty_invoiced -= sum([line.product_uom_id._compute_quantity(line.quantity, line.product_id.uom_id) for line in reversal_cogs])

                moves = so_line.move_ids
                average_price_unit = 0
                components_qty = so_line._get_bom_component_qty(bom)
                storable_components = self.env['product.product'].search([('id', 'in', list(components_qty.keys())), ('type', '=', 'product')])
                for product in storable_components:
                    factor = components_qty[product.id]['qty']
                    prod_moves = moves.filtered(lambda m: m.product_id == product)
                    prod_qty_invoiced = factor * qty_invoiced
                    prod_qty_to_invoice = factor * qty_to_invoice
                    product = product.with_company(self.company_id)
                    average_price_unit += factor * product._compute_average_price(prod_qty_invoiced, prod_qty_to_invoice, prod_moves, is_returned=is_line_reversing)
                price_unit = average_price_unit / bom.product_qty or price_unit
                price_unit = self.product_id.uom_id._compute_price(price_unit, self.product_uom_id)
        return price_unit

```

## File: models\mrp_bom.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, models
from odoo.exceptions import UserError


class MrpBom(models.Model):
    _inherit = 'mrp.bom'

    def toggle_active(self):
        self.filtered(lambda bom: bom.active)._ensure_bom_is_free()
        return super().toggle_active()

    def write(self, vals):
        if 'phantom' in self.mapped('type') and vals.get('type', 'phantom') != 'phantom':
            self._ensure_bom_is_free()
        return super().write(vals)

    def unlink(self):
        self._ensure_bom_is_free()
        return super().unlink()

    def _ensure_bom_is_free(self):
        product_ids = []
        for bom in self:
            if bom.type != 'phantom':
                continue
            product_ids += bom.product_id.ids or bom.product_tmpl_id.product_variant_ids.ids
        if not product_ids:
            return
        lines = self.env['sale.order.line'].sudo().search([
            ('state', '=', 'sale'),
            ('invoice_status', 'in', ('no', 'to invoice')),
            ('product_id', 'in', product_ids),
            ('move_ids.state', '!=', 'cancel'),
        ])
        if lines:
            product_names = ', '.join(lines.product_id.mapped('display_name'))
            raise UserError(_('As long as there are some sale order lines that must be delivered/invoiced and are '
                              'related to these bills of materials, you can not remove them.\n'
                              'The error concerns these products: %s', product_names))

```

## File: models\mrp_production.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _


class MrpProduction(models.Model):
    _inherit = 'mrp.production'

    sale_order_count = fields.Integer(
        "Count of Source SO",
        compute='_compute_sale_order_count',
        groups='sales_team.group_sale_salesman')

    @api.depends('procurement_group_id.mrp_production_ids.move_dest_ids.group_id.sale_id')
    def _compute_sale_order_count(self):
        for production in self:
            production.sale_order_count = len(production.procurement_group_id.mrp_production_ids.move_dest_ids.group_id.sale_id)

    def action_view_sale_orders(self):
        self.ensure_one()
        sale_order_ids = self.procurement_group_id.mrp_production_ids.move_dest_ids.group_id.sale_id.ids
        action = {
            'res_model': 'sale.order',
            'type': 'ir.actions.act_window',
        }
        if len(sale_order_ids) == 1:
            action.update({
                'view_mode': 'form',
                'res_id': sale_order_ids[0],
            })
        else:
            action.update({
                'name': _("Sources Sale Orders of %s", self.name),
                'domain': [('id', 'in', sale_order_ids)],
                'view_mode': 'tree,form',
            })
        return action

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _


class SaleOrder(models.Model):
    _inherit = 'sale.order'

    mrp_production_count = fields.Integer(
        "Count of MO generated",
        compute='_compute_mrp_production_ids',
        groups='mrp.group_mrp_user')
    mrp_production_ids = fields.Many2many(
        'mrp.production',
        compute='_compute_mrp_production_ids',
        string='Manufacturing orders associated with this sales order.',
        groups='mrp.group_mrp_user')

    @api.depends('procurement_group_id.stock_move_ids.created_production_id.procurement_group_id.mrp_production_ids')
    def _compute_mrp_production_ids(self):
        data = self.env['procurement.group']._read_group([('sale_id', 'in', self.ids)], ['sale_id'], ['id:recordset'])
        mrp_productions = {}
        for sale, procurement_groups in data:
            mrp_productions[sale.id] = procurement_groups.stock_move_ids.created_production_id.procurement_group_id.mrp_production_ids | procurement_groups.mrp_production_ids
        for sale in self:
            mrp_production_ids = mrp_productions.get(sale.id, self.env['mrp.production'])
            sale.mrp_production_count = len(mrp_production_ids)
            sale.mrp_production_ids = mrp_production_ids

    def action_view_mrp_production(self):
        self.ensure_one()
        action = {
            'res_model': 'mrp.production',
            'type': 'ir.actions.act_window',
        }
        if len(self.mrp_production_ids) == 1:
            action.update({
                'view_mode': 'form',
                'res_id': self.mrp_production_ids.id,
            })
        else:
            action.update({
                'name': _("Manufacturing Orders Generated by %s", self.name),
                'domain': [('id', 'in', self.mrp_production_ids.ids)],
                'view_mode': 'tree,form',
            })
        return action

```

## File: models\sale_order_line.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models
from odoo.tools import float_compare


class SaleOrderLine(models.Model):
    _inherit = 'sale.order.line'

    @api.depends('product_uom_qty', 'qty_delivered', 'product_id', 'state')
    def _compute_qty_to_deliver(self):
        """The inventory widget should now be visible in more cases if the product is consumable."""
        super(SaleOrderLine, self)._compute_qty_to_deliver()
        for line in self:
            # Hide the widget for kits since forecast doesn't support them.
            boms = self.env['mrp.bom']
            if line.state == 'sale':
                boms = line.move_ids.mapped('bom_line_id.bom_id')
            elif line.state in ['draft', 'sent'] and line.product_id:
                boms = boms._bom_find(line.product_id, company_id=line.company_id.id, bom_type='phantom')[line.product_id]
            relevant_bom = boms.filtered(lambda b: b.type == 'phantom' and
                    (b.product_id == line.product_id or
                    (b.product_tmpl_id == line.product_id.product_tmpl_id and not b.product_id)))
            if relevant_bom:
                line.display_qty_widget = False
                continue
            if line.state == 'draft' and line.product_type == 'consu':
                components = line.product_id.get_components()
                if components and components != [line.product_id.id]:
                    line.display_qty_widget = True

    def _compute_qty_delivered(self):
        super(SaleOrderLine, self)._compute_qty_delivered()
        for order_line in self:
            if order_line.qty_delivered_method == 'stock_move':
                boms = order_line.move_ids.filtered(lambda m: m.state != 'cancel').mapped('bom_line_id.bom_id')
                dropship = any(m._is_dropshipped() for m in order_line.move_ids)
                # We fetch the BoMs of type kits linked to the order_line,
                # the we keep only the one related to the finished produst.
                # This bom should be the only one since bom_line_id was written on the moves
                relevant_bom = boms.filtered(lambda b: b.type == 'phantom' and
                        (b.product_id == order_line.product_id or
                        (b.product_tmpl_id == order_line.product_id.product_tmpl_id and not b.product_id)))
                if not relevant_bom:
                    relevant_bom = boms._bom_find(order_line.product_id, company_id=order_line.company_id.id, bom_type='phantom')[order_line.product_id]
                if relevant_bom:
                    # not written on a move coming from a PO: all moves (to customer) must be done
                    # and the returns must be delivered back to the customer
                    # FIXME: if the components of a kit have different suppliers, multiple PO
                    # are generated. If one PO is confirmed and all the others are in draft, receiving
                    # the products for this PO will set the qty_delivered. We might need to check the
                    # state of all PO as well... but sale_mrp doesn't depend on purchase.
                    if dropship:
                        moves = order_line.move_ids.filtered(lambda m: m.state != 'cancel')
                        if any((m.location_dest_id.usage == 'customer' and m.state != 'done')
                               or (m.location_dest_id.usage != 'customer'
                               and m.state == 'done'
                               and float_compare(m.quantity,
                                                 sum(sub_m.product_uom._compute_quantity(sub_m.quantity, m.product_uom) for sub_m in m.returned_move_ids if sub_m.state == 'done'),
                                                 precision_rounding=m.product_uom.rounding) > 0)
                               for m in moves) or not moves:
                            order_line.qty_delivered = 0
                        else:
                            order_line.qty_delivered = order_line.product_uom_qty
                        continue
                    moves = order_line.move_ids.filtered(lambda m: m.state == 'done' and not m.scrapped)
                    filters = {
                        'incoming_moves': lambda m: m.location_dest_id.usage == 'customer' and (not m.origin_returned_move_id or (m.origin_returned_move_id and m.to_refund)),
                        'outgoing_moves': lambda m: m.location_id.usage == 'customer' and m.to_refund
                    }
                    order_qty = order_line.product_uom._compute_quantity(order_line.product_uom_qty, relevant_bom.product_uom_id)
                    qty_delivered = moves._compute_kit_quantities(order_line.product_id, order_qty, relevant_bom, filters)
                    order_line.qty_delivered += relevant_bom.product_uom_id._compute_quantity(qty_delivered, order_line.product_uom)

                # If no relevant BOM is found, fall back on the all-or-nothing policy. This happens
                # when the product sold is made only of kits. In this case, the BOM of the stock moves
                # do not correspond to the product sold => no relevant BOM.
                elif boms:
                    # if the move is ingoing, the product **sold** has delivered qty 0
                    if all(m.state == 'done' and m.location_dest_id.usage == 'customer' for m in order_line.move_ids):
                        order_line.qty_delivered = order_line.product_uom_qty
                    else:
                        order_line.qty_delivered = 0.0

    def compute_uom_qty(self, new_qty, stock_move, rounding=True):
        #check if stock move concerns a kit
        if stock_move.bom_line_id:
            return new_qty * stock_move.bom_line_id.product_qty
        return super(SaleOrderLine, self).compute_uom_qty(new_qty, stock_move, rounding)

    def _get_bom_component_qty(self, bom):
        bom_quantity = self.product_id.uom_id._compute_quantity(1, bom.product_uom_id, rounding_method='HALF-UP')
        boms, lines = bom.explode(self.product_id, bom_quantity)
        components = {}
        for line, line_data in lines:
            product = line.product_id.id
            uom = line.product_uom_id
            qty = line_data['qty']
            if components.get(product, False):
                if uom.id != components[product]['uom']:
                    from_uom = uom
                    to_uom = self.env['uom.uom'].browse(components[product]['uom'])
                    qty = from_uom._compute_quantity(qty, to_uom)
                components[product]['qty'] += qty
            else:
                # To be in the uom reference of the product
                to_uom = self.env['product.product'].browse(product).uom_id
                if uom.id != to_uom.id:
                    from_uom = uom
                    qty = from_uom._compute_quantity(qty, to_uom)
                components[product] = {'qty': qty, 'uom': to_uom.id}
        return components

    @api.model
    def _get_incoming_outgoing_moves_filter(self):
        """ Method to be override: will get incoming moves and outgoing moves.

        :return: Dictionary with incoming moves and outgoing moves
        :rtype: dict
        """
        return {
            'incoming_moves': lambda m: m.location_dest_id.usage == 'customer' and \
                        (not m.origin_returned_move_id or (m.origin_returned_move_id and m.to_refund)),
            'outgoing_moves': lambda m: m.location_dest_id.usage != 'customer' and m.to_refund
        }

    def _get_qty_procurement(self, previous_product_uom_qty=False):
        self.ensure_one()
        # Specific case when we change the qty on a SO for a kit product.
        # We don't try to be too smart and keep a simple approach: we use the quantity of entire
        # kits that are currently in delivery
        bom = self.env['mrp.bom'].sudo()._bom_find(self.product_id, bom_type='phantom', company_id=self.company_id.id)[self.product_id]
        if bom:
            moves = self.move_ids.filtered(lambda r: r.state != 'cancel' and not r.scrapped)
            filters = self._get_incoming_outgoing_moves_filter()
            order_qty = previous_product_uom_qty.get(self.id, 0) if previous_product_uom_qty else self.product_uom_qty
            order_qty = self.product_uom._compute_quantity(order_qty, bom.product_uom_id)
            qty = moves._compute_kit_quantities(self.product_id, order_qty, bom, filters)
            return bom.product_uom_id._compute_quantity(qty, self.product_uom)
        return super(SaleOrderLine, self)._get_qty_procurement(previous_product_uom_qty=previous_product_uom_qty)

```

## File: models\stock_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class StockMove(models.Model):
    _inherit = 'stock.move'

    def _prepare_procurement_values(self):
        res = super()._prepare_procurement_values()
        res['analytic_account_id'] = self.sale_line_id.order_id.analytic_account_id
        if self.sale_line_id.order_id.analytic_account_id:
            res['analytic_distribution'] = {self.sale_line_id.order_id.analytic_account_id.id: 100}
        return res


class StockMoveLine(models.Model):
    _inherit = 'stock.move.line'

    def _compute_sale_price(self):
        kit_lines = self.filtered(lambda move_line: move_line.move_id.bom_line_id.bom_id.type == 'phantom')
        for move_line in kit_lines:
            unit_price = move_line.product_id.list_price
            qty = move_line.product_uom_id._compute_quantity(move_line.quantity, move_line.product_id.uom_id)
            move_line.sale_price = unit_price * qty
        super(StockMoveLine, self - kit_lines)._compute_sale_price()

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_move
from . import mrp_production
from . import sale_order
from . import sale_order_line
from . import stock_move
from . import mrp_bom

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_mrp_bom_user,mrp.bom,mrp.model_mrp_bom,sales_team.group_sale_salesman,1,0,0,0
access_sale_order_manufacturing_user,sale.order manufacturing.user,sale.model_sale_order,mrp.group_mrp_user,1,1,0,0
access_sale_order_line_manufacturing_user,sale.order.line manufacturing.user,sale.model_sale_order_line,mrp.group_mrp_user,1,1,0,0
access_mrp_production_salesman,mrp.production salesman,mrp.model_mrp_production,sales_team.group_sale_salesman,1,1,1,0
access_mrp_production_workcenter_line_salesman,mrp.workorder salesman,mrp.model_mrp_workorder,sales_team.group_sale_salesman,1,0,1,0
access_mrp_bom_line_salesman,mrp.bom.line,mrp.model_mrp_bom_line,sales_team.group_sale_salesman,1,0,0,0

```

## File: views\mrp_production_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>
    <record id="mrp_production_form_view_sale" model="ir.ui.view">
        <field name="name">mrp.production.inherited.form.sale</field>
        <field name="model">mrp.production</field>
        <field name="priority">64</field>
        <field name="inherit_id" ref="mrp.mrp_production_form_view"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@name='action_view_mrp_production_childs']" position="before">
                <button class="oe_stat_button" name="action_view_sale_orders" type="object" icon="fa-dollar" invisible="sale_order_count == 0" groups="sales_team.group_sale_salesman">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_value"><field name="sale_order_count"/></span>
                        <span class="o_stat_text">Sale</span>
                    </div>
                </button>
            </xpath>
        </field>
    </record>
</data></odoo>
```

## File: views\sale_order_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>
    <record id="sale_order_form_mrp" model="ir.ui.view">
        <field name="name">sale.order.inherited.form.mrp</field>
        <field name="model">sale.order</field>
        <field name="inherit_id" ref="sale.view_order_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='button_box']" position="inside">
                <button class="oe_stat_button" name="action_view_mrp_production" type="object" icon="fa-wrench" invisible="mrp_production_count == 0" groups="mrp.group_mrp_user">
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

## File: views\sale_portal_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="sale_order_portal_content_inherit_sale_mrp"
        name="Orders MOs"
        inherit_id="sale.sale_order_portal_content">
        <xpath expr="//div[@id='informations']" position="inside">
            <t t-set="mrp_productions" t-value="sale_order.mrp_production_ids"/>
            <t t-if="mrp_productions">
                <div>
                    <strong>Manufacturing Orders</strong>
                </div>
                <div>
                    <t t-foreach="mrp_productions" t-as="mo">
                        <div class="d-flex flex-wrap align-items-center justify-content-between">
                            <div>
                                <span t-out="mo.name"/>
                                <div class="small d-lg-inline-block ms-3">
                                    Date:
                                    <span class="text-muted"
                                        t-field="mo.date_finished"
                                        t-options="{'date_only': True}"/>
                                    <span t-if="mo.state in ['draft', 'confirmed', 'progress', 'to_close']"
                                        class="text-muted"
                                        t-field="mo.date_finished"
                                        t-options="{'date_only': True}"/>
                                </div>
                            </div>
                            <span t-if="mo.state in ['to_close', 'done']"
                                class="small badge text-bg-success orders_label_text_align">
                                <i class="fa fa-fw fa-check"/> <b>Manufactured</b>
                            </span>
                            <span t-elif="mo.state == 'cancel'"
                                class="small badge text-bg-danger orders_label_text_align">
                                <i class="fa fa-fw fa-times"/> <b>Cancelled</b>
                            </span>
                            <span t-elif="mo.state in ['confirmed']"
                                class="small badge text-bg-info orders_label_text_align">
                                <i class="fa fa-fw fa-clock-o"/> <b>Confirmed</b>
                            </span>
                            <span t-elif="mo.state in ['progress']"
                                class="small badge text-bg-warning orders_label_text_align">
                                <i class="fa fa-fw fa-clock-o"/> <b>In progress</b>
                            </span>
                        </div>
                    </t>
                </div>
            </t>
        </xpath>
    </template>

</odoo>

```

