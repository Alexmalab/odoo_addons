# Odoo Module: purchase_stock

Category: Inventory/Purchase

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import report

from odoo import api, SUPERUSER_ID


def _create_buy_rules(cr, registry):
    """ This hook is used to add a default buy_pull_id on every warehouse. It is
    necessary if the purchase_stock module is installed after some warehouses
    were already created.
    """
    env = api.Environment(cr, SUPERUSER_ID, {})
    warehouse_ids = env['stock.warehouse'].search([('buy_pull_id', '=', False)])
    warehouse_ids.write({'buy_to_resupply': True})

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Purchase Stock',
    'version': '1.2',
    'category': 'Inventory/Purchase',
    'sequence': 60,
    'summary': 'Purchase Orders, Receipts, Vendor Bills for Stock',
    'description': "",
    'depends': ['stock_account', 'purchase'],
    'data': [
        'security/ir.model.access.csv',
        'data/purchase_stock_data.xml',
        'data/mail_data.xml',
        'views/assets.xml',
        'report/vendor_delay_report.xml',
        'views/purchase_views.xml',
        'views/stock_views.xml',
        'views/stock_rule_views.xml',
        'views/res_config_settings_views.xml',
        'views/res_partner_views.xml',
        'views/stock_production_lot_views.xml',
        'views/product_category_views.xml',
        'report/purchase_report_views.xml',
        'report/purchase_report_templates.xml',
        'report/report_stock_forecasted.xml',
        'report/report_stock_rule.xml',
    ],
    'demo': [
        'data/purchase_stock_demo.xml',
    ],
    'installable': True,
    'auto_install': True,
    'post_init_hook': '_create_buy_rules',
    'license': 'LGPL-3',
}

```

## File: data\mail_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="exception_on_po">
        <div>
          Exception(s) occurred on the purchase order(s):
          <t t-foreach="purchase_order_ids" t-as="purchase_order">
              <a href="#" data-oe-model="purchase.order" t-att-data-oe-id="purchase_order.id"><t t-esc="purchase_order.name"/></a>.
          </t>
          Manual actions may be needed.
          <div class="mt16">
              <p>Exception(s):</p>
              <ul t-foreach="order_exceptions" t-as="exception">
                  <li>
                      <t t-set="order_line" t-value="exception[0]"/>
                      <t t-set="new_qty" t-value="exception[1][0]"/>
                      <t t-set="old_qty" t-value="exception[1][1]"/>
                      <t t-esc="new_qty"/> <t t-esc="order_line.product_uom.name"/> of <t t-esc="order_line.product_id.name"/>
                      ordered instead of <t t-esc="old_qty"/> <t t-esc="order_line.product_uom.name"/>
                  </li>
              </ul>
          </div>
          <div class="mt16" t-if="impacted_pickings">
              <p>Next transfer(s) impacted:</p>
              <ul t-foreach="impacted_pickings" t-as="picking">
                  <li><a href="#" data-oe-model="stock.picking" t-att-data-oe-id="picking.id"><t t-esc="picking.name"/></a></li>
              </ul>
          </div>
        </div>
    </template>

</odoo>

```

## File: data\purchase_stock_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <function model="purchase.order.line" name="_update_qty_received_method" />

		<!--
             Stock rules and routes
        -->
        <record id="route_warehouse0_buy" model='stock.location.route'>
            <field name="name">Buy</field>
            <field name="company_id"></field>
            <field name="sequence">5</field>
        </record>

        <!-- enable purchase on main warehouse -->
        <record id="stock.warehouse0" model="stock.warehouse">
            <field name="buy_to_resupply" eval="True"/>
        </record>

    </data>
</odoo>

```

## File: data\purchase_stock_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <record id="stock.res_company_1" model="res.company">
            <field eval="1.0" name="po_lead"/>
        </record>

        <record id="product.product_delivery_01" model="product.product">
            <field name="route_ids" eval="[(4,ref('route_warehouse0_buy'))]"></field>
        </record>

        <record id="product.product_delivery_02" model="product.product">
            <field name="route_ids" eval="[(4,ref('route_warehouse0_buy'))]"></field>
        </record>

        <record id="product.product_product_9" model="product.product">
            <field name="route_ids" eval="[(4,ref('route_warehouse0_buy'))]"></field>
        </record>

        <record id="product.product_product_12" model="product.product">
            <field name="route_ids" eval="[(4,ref('route_warehouse0_buy'))]"></field>
        </record>

        <record id="product.product_product_13" model="product.product">
            <field name="route_ids" eval="[(4,ref('route_warehouse0_buy'))]"></field>
        </record>

        <record id="product.product_product_16" model="product.product">
            <field name="route_ids" eval="[(4,ref('route_warehouse0_buy'))]"></field>
        </record>

        <record id="product.product_product_20" model="product.product">
            <field name="route_ids" eval="[(4,ref('route_warehouse0_buy'))]"></field>
        </record>

        <record id="purchase_order_8" model="purchase.order">
            <field name="partner_id" ref="base.res_partner_4"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="state">draft</field>
            <field name="date_order" eval="(datetime.now() + relativedelta(days=2)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="date_planned" eval="(datetime.now() + relativedelta(days=2)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="order_line" model="purchase.order.line" eval="[(5, 0, 0),
                (0, 0, {
                    'product_id': ref('product.product_product_25'),
                    'name': obj().env.ref('product.product_product_25').partner_ref,
                    'price_unit': 2864.80,
                    'product_qty': 20.0,
                    'product_uom': ref('uom.product_uom_unit'),
                    'date_planned': DateTime.today()}),
            ]"/>
        </record>

        <function model="purchase.order" name="button_confirm" eval="[[ref('purchase_order_8')]]"/>

    </data>

    <data noupdate="0">

        <record id="stock.stock_warehouse_shop0" model="stock.warehouse">
            <field name="buy_to_resupply" eval="True"/>
        </record>

        <function model="stock.warehouse" name="write">
          <value model="stock.warehouse" search="[('partner_id', '=', ref('stock.res_partner_company_1'))]"/>
          <value eval="{'buy_to_resupply': True}"/>
        </function>

    </data>

</odoo>

```

## File: models\account_invoice.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.tools.float_utils import float_compare, float_is_zero


class AccountMove(models.Model):
    _inherit = 'account.move'

    def _stock_account_prepare_anglo_saxon_in_lines_vals(self):
        ''' Prepare values used to create the journal items (account.move.line) corresponding to the price difference
         lines for vendor bills.

        Example:

        Buy a product having a cost of 9 and a supplier price of 10 and being a storable product and having a perpetual
        valuation in FIFO. The vendor bill's journal entries looks like:

        Account                                     | Debit | Credit
        ---------------------------------------------------------------
        101120 Stock Interim Account (Received)     | 10.0  |
        ---------------------------------------------------------------
        101100 Account Payable                      |       | 10.0
        ---------------------------------------------------------------

        This method computes values used to make two additional journal items:

        ---------------------------------------------------------------
        101120 Stock Interim Account (Received)     |       | 1.0
        ---------------------------------------------------------------
        xxxxxx Price Difference Account             | 1.0   |
        ---------------------------------------------------------------

        :return: A list of Python dictionary to be passed to env['account.move.line'].create.
        '''
        lines_vals_list = []
        price_unit_prec = self.env['decimal.precision'].precision_get('Product Price')

        for move in self:
            if move.move_type not in ('in_invoice', 'in_refund', 'in_receipt') or not move.company_id.anglo_saxon_accounting:
                continue

            move = move.with_company(move.company_id)
            for line in move.invoice_line_ids:
                # Filter out lines being not eligible for price difference.
                if line.product_id.type != 'product' or line.product_id.valuation != 'real_time':
                    continue

                # Retrieve accounts needed to generate the price difference.
                debit_pdiff_account = line.product_id.property_account_creditor_price_difference \
                                or line.product_id.categ_id.property_account_creditor_price_difference_categ
                debit_pdiff_account = move.fiscal_position_id.map_account(debit_pdiff_account)
                if not debit_pdiff_account:
                    continue
                # Retrieve stock valuation moves.
                valuation_stock_moves = self.env['stock.move'].search([
                    ('purchase_line_id', '=', line.purchase_line_id.id),
                    ('state', '=', 'done'),
                    ('product_qty', '!=', 0.0),
                ]) if line.purchase_line_id else self.env['stock.move']
                if move.move_type == 'in_refund':
                    valuation_stock_moves = valuation_stock_moves.filtered(lambda stock_move: stock_move._is_out())
                else:
                    valuation_stock_moves = valuation_stock_moves.filtered(lambda stock_move: stock_move._is_in())

                if line.product_id.cost_method != 'standard' and line.purchase_line_id:
                    po_currency = line.purchase_line_id.currency_id
                    po_company = line.purchase_line_id.company_id

                    if valuation_stock_moves:
                        valuation_price_unit_total, valuation_total_qty = valuation_stock_moves._get_valuation_price_and_qty(line, move.currency_id)
                        valuation_price_unit = valuation_price_unit_total / valuation_total_qty
                        valuation_price_unit = line.product_id.uom_id._compute_price(valuation_price_unit, line.product_uom_id)

                    elif line.product_id.cost_method == 'fifo':
                        # In this condition, we have a real price-valuated product which has not yet been received
                        valuation_price_unit = po_currency._convert(
                            line.purchase_line_id.price_unit, move.currency_id,
                            po_company, move.date, round=False,
                        )
                    else:
                        # For average/fifo/lifo costing method, fetch real cost price from incoming moves.
                        price_unit = line.purchase_line_id.product_uom._compute_price(line.purchase_line_id.price_unit, line.product_uom_id)
                        valuation_price_unit = po_currency._convert(
                            price_unit, move.currency_id,
                            po_company, move.date, round=False
                        )

                else:
                    # Valuation_price unit is always expressed in invoice currency, so that it can always be computed with the good rate
                    price_unit = line.product_id.uom_id._compute_price(line.product_id.standard_price, line.product_uom_id)
                    valuation_date = valuation_stock_moves and max(valuation_stock_moves.mapped('date')) or move.date
                    valuation_price_unit = line.company_currency_id._convert(
                        price_unit, move.currency_id,
                        move.company_id, valuation_date, round=False
                    )


                price_unit = line.price_unit * (1 - (line.discount or 0.0) / 100.0)
                if line.tax_ids:
                    # We do not want to round the price unit since :
                    # - It does not follow the currency precision
                    # - It may include a discount
                    # Since compute_all still rounds the total, we use an ugly workaround:
                    # shift the decimal part using a fixed quantity to avoid rounding issues
                    prec = 1e+6
                    price_unit *= prec
                    price_unit = line.tax_ids.with_context(round=False, force_sign=move._get_tax_force_sign()).compute_all(
                        price_unit, currency=move.currency_id, quantity=1.0, is_refund=move.move_type == 'in_refund')['total_excluded']
                    price_unit /= prec

                price_unit_val_dif = price_unit - valuation_price_unit
                price_subtotal = line.quantity * price_unit_val_dif

                # We consider there is a price difference if:
                # - price unit is not zero with respect to product price decimal precision.
                # - subtotal is not zero with respect to move currency precision.
                # - no discount was applied, as we can't round the price unit anymore
                if (
                    not move.currency_id.is_zero(price_subtotal)
                    and not float_is_zero(price_unit_val_dif, precision_digits=price_unit_prec)
                    and float_compare(line["price_unit"], line.price_unit, precision_digits=price_unit_prec) == 0
                ):

                    # Add price difference account line.
                    vals = {
                        'name': line.name[:64],
                        'move_id': move.id,
                        'partner_id': line.partner_id.id or move.commercial_partner_id.id,
                        'currency_id': line.currency_id.id,
                        'product_id': line.product_id.id,
                        'product_uom_id': line.product_uom_id.id,
                        'quantity': line.quantity,
                        'price_unit': price_unit_val_dif,
                        'price_subtotal': line.quantity * price_unit_val_dif,
                        'account_id': debit_pdiff_account.id,
                        'analytic_account_id': line.analytic_account_id.id,
                        'analytic_tag_ids': [(6, 0, line.analytic_tag_ids.ids)],
                        'exclude_from_invoice_tab': True,
                        'is_anglo_saxon_line': True,
                    }
                    vals.update(line._get_fields_onchange_subtotal(price_subtotal=vals['price_subtotal']))
                    lines_vals_list.append(vals)

                    # Correct the amount of the current line.
                    vals = {
                        'name': line.name[:64],
                        'move_id': move.id,
                        'partner_id': line.partner_id.id or move.commercial_partner_id.id,
                        'currency_id': line.currency_id.id,
                        'product_id': line.product_id.id,
                        'product_uom_id': line.product_uom_id.id,
                        'quantity': line.quantity,
                        'price_unit': -price_unit_val_dif,
                        'price_subtotal': line.quantity * -price_unit_val_dif,
                        'account_id': line.account_id.id,
                        'analytic_account_id': line.analytic_account_id.id,
                        'analytic_tag_ids': [(6, 0, line.analytic_tag_ids.ids)],
                        'exclude_from_invoice_tab': True,
                        'is_anglo_saxon_line': True,
                    }
                    vals.update(line._get_fields_onchange_subtotal(price_subtotal=vals['price_subtotal']))
                    lines_vals_list.append(vals)
        return lines_vals_list

    def _post(self, soft=True):
        # OVERRIDE
        # Create additional price difference lines for vendor bills.
        if self._context.get('move_reverse_cancel'):
            return super()._post(soft)
        self.env['account.move.line'].create(self._stock_account_prepare_anglo_saxon_in_lines_vals())
        return super()._post(soft)

    def _stock_account_get_last_step_stock_moves(self):
        """ Overridden from stock_account.
        Returns the stock moves associated to this invoice."""
        rslt = super(AccountMove, self)._stock_account_get_last_step_stock_moves()
        for invoice in self.filtered(lambda x: x.move_type == 'in_invoice'):
            rslt += invoice.mapped('invoice_line_ids.purchase_line_id.move_ids').filtered(lambda x: x.state == 'done' and x.location_id.usage == 'supplier')
        for invoice in self.filtered(lambda x: x.move_type == 'in_refund'):
            rslt += invoice.mapped('invoice_line_ids.purchase_line_id.move_ids').filtered(lambda x: x.state == 'done' and x.location_dest_id.usage == 'supplier')
        return rslt

```

## File: models\product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo.osv import expression


class ProductTemplate(models.Model):
    _name = 'product.template'
    _inherit = 'product.template'

    @api.model
    def _get_buy_route(self):
        buy_route = self.env.ref('purchase_stock.route_warehouse0_buy', raise_if_not_found=False)
        if buy_route:
            return self.env['stock.location.route'].search([('id', '=', buy_route.id)]).ids
        return []

    route_ids = fields.Many2many(default=lambda self: self._get_buy_route())


class ProductProduct(models.Model):
    _name = 'product.product'
    _inherit = 'product.product'

    purchase_order_line_ids = fields.One2many('purchase.order.line', 'product_id', help='Technical: used to compute quantities.')

    def _get_quantity_in_progress(self, location_ids=False, warehouse_ids=False):
        if not location_ids:
            location_ids = []
        if not warehouse_ids:
            warehouse_ids = []

        qty_by_product_location, qty_by_product_wh = super()._get_quantity_in_progress(location_ids, warehouse_ids)
        domain = []
        rfq_domain = [
            ('state', 'in', ('draft', 'sent', 'to approve')),
            ('product_id', 'in', self.ids)
        ]
        if location_ids:
            domain = expression.AND([rfq_domain, [
                '|',
                    ('order_id.picking_type_id.default_location_dest_id', 'in', location_ids),
                    '&',
                        ('move_dest_ids', '=', False),
                        ('orderpoint_id.location_id', 'in', location_ids)
            ]])
        if warehouse_ids:
            wh_domain = expression.AND([rfq_domain, [
                '|',
                    ('order_id.picking_type_id.warehouse_id', 'in', warehouse_ids),
                    '&',
                        ('move_dest_ids', '=', False),
                        ('orderpoint_id.warehouse_id', 'in', warehouse_ids)
            ]])
            domain = expression.OR([domain, wh_domain])
        groups = self.env['purchase.order.line'].read_group(domain,
            ['product_id', 'product_qty', 'order_id', 'product_uom', 'orderpoint_id'],
            ['order_id', 'product_id', 'product_uom', 'orderpoint_id'], lazy=False)
        for group in groups:
            if group.get('orderpoint_id'):
                location = self.env['stock.warehouse.orderpoint'].browse(group['orderpoint_id'][:1]).location_id
            else:
                order = self.env['purchase.order'].browse(group['order_id'][0])
                location = order.picking_type_id.default_location_dest_id
            product = self.env['product.product'].browse(group['product_id'][0])
            uom = self.env['uom.uom'].browse(group['product_uom'][0])
            product_qty = uom._compute_quantity(group['product_qty'], product.uom_id, round=False)
            qty_by_product_location[(product.id, location.id)] += product_qty
            qty_by_product_wh[(product.id, location.get_warehouse().id)] += product_qty
        return qty_by_product_location, qty_by_product_wh

```

## File: models\purchase.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, fields, models, SUPERUSER_ID, _
from odoo.tools.float_utils import float_compare, float_round
from datetime import datetime
from dateutil.relativedelta import relativedelta
from odoo.exceptions import UserError

from odoo.addons.purchase.models.purchase import PurchaseOrder as Purchase


class PurchaseOrder(models.Model):
    _inherit = 'purchase.order'

    @api.model
    def _default_picking_type(self):
        return self._get_picking_type(self.env.context.get('company_id') or self.env.company.id)

    incoterm_id = fields.Many2one('account.incoterms', 'Incoterm', states={'done': [('readonly', True)]}, help="International Commercial Terms are a series of predefined commercial terms used in international transactions.")

    picking_count = fields.Integer(compute='_compute_picking', string='Picking count', default=0, store=True)
    picking_ids = fields.Many2many('stock.picking', compute='_compute_picking', string='Receptions', copy=False, store=True)

    picking_type_id = fields.Many2one('stock.picking.type', 'Deliver To', states=Purchase.READONLY_STATES, required=True, default=_default_picking_type, domain="['|', ('warehouse_id', '=', False), ('warehouse_id.company_id', '=', company_id)]",
        help="This will determine operation type of incoming shipment")
    default_location_dest_id_usage = fields.Selection(related='picking_type_id.default_location_dest_id.usage', string='Destination Location Type',
        help="Technical field used to display the Drop Ship Address", readonly=True)
    group_id = fields.Many2one('procurement.group', string="Procurement Group", copy=False)
    is_shipped = fields.Boolean(compute="_compute_is_shipped")
    effective_date = fields.Datetime("Effective Date", compute='_compute_effective_date', store=True, copy=False,
        help="Completion date of the first receipt order.")
    on_time_rate = fields.Float(related='partner_id.on_time_rate', compute_sudo=False)

    @api.depends('order_line.move_ids.picking_id')
    def _compute_picking(self):
        for order in self:
            pickings = order.order_line.mapped('move_ids.picking_id')
            order.picking_ids = pickings
            order.picking_count = len(pickings)

    @api.depends('picking_ids.date_done')
    def _compute_effective_date(self):
        for order in self:
            pickings = order.picking_ids.filtered(lambda x: x.state == 'done' and x.location_dest_id.usage == 'internal' and x.date_done)
            order.effective_date = min(pickings.mapped('date_done'), default=False)

    @api.depends('picking_ids', 'picking_ids.state')
    def _compute_is_shipped(self):
        for order in self:
            if order.picking_ids and all(x.state in ['done', 'cancel'] for x in order.picking_ids):
                order.is_shipped = True
            else:
                order.is_shipped = False

    @api.onchange('picking_type_id')
    def _onchange_picking_type_id(self):
        if self.picking_type_id.default_location_dest_id.usage != 'customer':
            self.dest_address_id = False

    @api.onchange('company_id')
    def _onchange_company_id(self):
        p_type = self.picking_type_id
        if not(p_type and p_type.code == 'incoming' and (p_type.warehouse_id.company_id == self.company_id or not p_type.warehouse_id)):
            self.picking_type_id = self._get_picking_type(self.company_id.id)

    # --------------------------------------------------
    # CRUD
    # --------------------------------------------------

    def write(self, vals):
        if vals.get('order_line') and self.state == 'purchase':
            for order in self:
                pre_order_line_qty = {order_line: order_line.product_qty for order_line in order.mapped('order_line')}
        res = super(PurchaseOrder, self).write(vals)
        if vals.get('order_line') and self.state == 'purchase':
            for order in self:
                to_log = {}
                for order_line in order.order_line:
                    if pre_order_line_qty.get(order_line, False) and float_compare(pre_order_line_qty[order_line], order_line.product_qty, precision_rounding=order_line.product_uom.rounding) > 0:
                        to_log[order_line] = (order_line.product_qty, pre_order_line_qty[order_line])
                if to_log:
                    order._log_decrease_ordered_quantity(to_log)
        return res

    # --------------------------------------------------
    # Actions
    # --------------------------------------------------

    def button_approve(self, force=False):
        result = super(PurchaseOrder, self).button_approve(force=force)
        self._create_picking()
        return result

    def button_cancel(self):
        for order in self:
            for move in order.order_line.mapped('move_ids'):
                if move.state == 'done':
                    raise UserError(_('Unable to cancel purchase order %s as some receptions have already been done.') % (order.name))
            # If the product is MTO, change the procure_method of the closest move to purchase to MTS.
            # The purpose is to link the po that the user will manually generate to the existing moves's chain.
            if order.state in ('draft', 'sent', 'to approve', 'purchase'):
                for order_line in order.order_line:
                    order_line.move_ids._action_cancel()
                    if order_line.move_dest_ids:
                        move_dest_ids = order_line.move_dest_ids
                        if order_line.propagate_cancel:
                            move_dest_ids._action_cancel()
                        else:
                            move_dest_ids.write({'procure_method': 'make_to_stock'})
                            move_dest_ids._recompute_state()

            for pick in order.picking_ids.filtered(lambda r: r.state != 'cancel'):
                pick.action_cancel()

            order.order_line.write({'move_dest_ids':[(5,0,0)]})

        return super(PurchaseOrder, self).button_cancel()

    def action_view_picking(self):
        """ This function returns an action that display existing picking orders of given purchase order ids. When only one found, show the picking immediately.
        """
        result = self.env["ir.actions.actions"]._for_xml_id('stock.action_picking_tree_all')
        # override the context to get rid of the default filtering on operation type
        result['context'] = {'default_partner_id': self.partner_id.id, 'default_origin': self.name, 'default_picking_type_id': self.picking_type_id.id}
        pick_ids = self.mapped('picking_ids')
        # choose the view_mode accordingly
        if not pick_ids or len(pick_ids) > 1:
            result['domain'] = "[('id','in',%s)]" % (pick_ids.ids)
        elif len(pick_ids) == 1:
            res = self.env.ref('stock.view_picking_form', False)
            form_view = [(res and res.id or False, 'form')]
            if 'views' in result:
                result['views'] = form_view + [(state,view) for state,view in result['views'] if view != 'form']
            else:
                result['views'] = form_view
            result['res_id'] = pick_ids.id
        return result

    def _prepare_invoice(self):
        invoice_vals = super()._prepare_invoice()
        invoice_vals['invoice_incoterm_id'] = self.incoterm_id.id
        return invoice_vals

    # --------------------------------------------------
    # Business methods
    # --------------------------------------------------

    def _log_decrease_ordered_quantity(self, purchase_order_lines_quantities):

        def _keys_in_sorted(move):
            """ sort by picking and the responsible for the product the
            move.
            """
            return (move.picking_id.id, move.product_id.responsible_id.id)

        def _keys_in_groupby(move):
            """ group by picking and the responsible for the product the
            move.
            """
            return (move.picking_id, move.product_id.responsible_id)

        def _render_note_exception_quantity_po(order_exceptions):
            order_line_ids = self.env['purchase.order.line'].browse([order_line.id for order in order_exceptions.values() for order_line in order[0]])
            purchase_order_ids = order_line_ids.mapped('order_id')
            move_ids = self.env['stock.move'].concat(*rendering_context.keys())
            impacted_pickings = move_ids.mapped('picking_id')._get_impacted_pickings(move_ids) - move_ids.mapped('picking_id')
            values = {
                'purchase_order_ids': purchase_order_ids,
                'order_exceptions': order_exceptions.values(),
                'impacted_pickings': impacted_pickings,
            }
            return self.env.ref('purchase_stock.exception_on_po')._render(values=values)

        documents = self.env['stock.picking']._log_activity_get_documents(purchase_order_lines_quantities, 'move_ids', 'DOWN', _keys_in_sorted, _keys_in_groupby)
        filtered_documents = {}
        for (parent, responsible), rendering_context in documents.items():
            if parent._name == 'stock.picking':
                if parent.state in ['cancel', 'done']:
                    continue
            filtered_documents[(parent, responsible)] = rendering_context
        self.env['stock.picking']._log_activity(_render_note_exception_quantity_po, filtered_documents)

    def _get_destination_location(self):
        self.ensure_one()
        if self.dest_address_id:
            return self.dest_address_id.property_stock_customer.id
        return self.picking_type_id.default_location_dest_id.id

    @api.model
    def _get_picking_type(self, company_id):
        picking_type = self.env['stock.picking.type'].search([('code', '=', 'incoming'), ('warehouse_id.company_id', '=', company_id)])
        if not picking_type:
            picking_type = self.env['stock.picking.type'].search([('code', '=', 'incoming'), ('warehouse_id', '=', False)])
        return picking_type[:1]

    def _prepare_picking(self):
        if not self.group_id:
            self.group_id = self.group_id.create({
                'name': self.name,
                'partner_id': self.partner_id.id
            })
        if not self.partner_id.property_stock_supplier.id:
            raise UserError(_("You must set a Vendor Location for this partner %s", self.partner_id.name))
        return {
            'picking_type_id': self.picking_type_id.id,
            'partner_id': self.partner_id.id,
            'user_id': False,
            'date': self.date_order,
            'origin': self.name,
            'location_dest_id': self._get_destination_location(),
            'location_id': self.partner_id.property_stock_supplier.id,
            'company_id': self.company_id.id,
        }

    def _create_picking(self):
        StockPicking = self.env['stock.picking']
        for order in self.filtered(lambda po: po.state in ('purchase', 'done')):
            if any(product.type in ['product', 'consu'] for product in order.order_line.product_id):
                order = order.with_company(order.company_id)
                pickings = order.picking_ids.filtered(lambda x: x.state not in ('done', 'cancel'))
                if not pickings:
                    res = order._prepare_picking()
                    picking = StockPicking.with_user(SUPERUSER_ID).create(res)
                else:
                    picking = pickings[0]
                moves = order.order_line._create_stock_moves(picking)
                moves = moves.filtered(lambda x: x.state not in ('done', 'cancel'))._action_confirm()
                seq = 0
                for move in sorted(moves, key=lambda move: move.date):
                    seq += 5
                    move.sequence = seq
                moves._action_assign()
                picking.message_post_with_view('mail.message_origin_link',
                    values={'self': picking, 'origin': order},
                    subtype_id=self.env.ref('mail.mt_note').id)
        return True

    def _add_picking_info(self, activity):
        """Helper method to add picking info to the Date Updated activity when
        vender updates date_planned of the po lines.
        """
        validated_picking = self.picking_ids.filtered(lambda p: p.state == 'done')
        if validated_picking:
            activity.note += _("<p>Those dates couldn’t be modified accordingly on the receipt %s which had already been validated.</p>") % validated_picking[0].name
        elif not self.picking_ids:
            activity.note += _("<p>Corresponding receipt not found.</p>")
        else:
            activity.note += _("<p>Those dates have been updated accordingly on the receipt %s.</p>") % self.picking_ids[0].name

    def _create_update_date_activity(self, updated_dates):
        activity = super()._create_update_date_activity(updated_dates)
        self._add_picking_info(activity)

    def _update_update_date_activity(self, updated_dates, activity):
        # remove old picking info to update it
        note_lines = activity.note.split('<p>')
        note_lines.pop()
        activity.note = '<p>'.join(note_lines)
        super()._update_update_date_activity(updated_dates, activity)
        self._add_picking_info(activity)

    @api.model
    def _get_orders_to_remind(self):
        """When auto sending reminder mails, don't send for purchase order with
        validated receipts."""
        return super()._get_orders_to_remind().filtered(lambda p: not p.effective_date)


class PurchaseOrderLine(models.Model):
    _inherit = 'purchase.order.line'

    qty_received_method = fields.Selection(selection_add=[('stock_moves', 'Stock Moves')])

    move_ids = fields.One2many('stock.move', 'purchase_line_id', string='Reservation', readonly=True, copy=False)
    orderpoint_id = fields.Many2one('stock.warehouse.orderpoint', 'Orderpoint', copy=False, index=True)
    move_dest_ids = fields.One2many('stock.move', 'created_purchase_line_id', 'Downstream Moves')
    product_description_variants = fields.Char('Custom Description')
    propagate_cancel = fields.Boolean('Propagate cancellation', default=True)

    def _compute_qty_received_method(self):
        super(PurchaseOrderLine, self)._compute_qty_received_method()
        for line in self.filtered(lambda l: not l.display_type):
            if line.product_id.type in ['consu', 'product']:
                line.qty_received_method = 'stock_moves'

    @api.depends('move_ids.state', 'move_ids.product_uom_qty', 'move_ids.product_uom')
    def _compute_qty_received(self):
        from_stock_lines = self.filtered(lambda order_line: order_line.qty_received_method == 'stock_moves')
        super(PurchaseOrderLine, self - from_stock_lines)._compute_qty_received()
        for line in self:
            if line.qty_received_method == 'stock_moves':
                total = 0.0
                # In case of a BOM in kit, the products delivered do not correspond to the products in
                # the PO. Therefore, we can skip them since they will be handled later on.
                for move in line.move_ids.filtered(lambda m: m.product_id == line.product_id):
                    if move.state == 'done':
                        if move._is_purchase_return():
                            if move.to_refund:
                                total -= move.product_uom._compute_quantity(move.product_uom_qty, line.product_uom, rounding_method='HALF-UP')
                        elif move.origin_returned_move_id and move.origin_returned_move_id._is_dropshipped() and not move._is_dropshipped_returned():
                            # Edge case: the dropship is returned to the stock, no to the supplier.
                            # In this case, the received quantity on the PO is set although we didn't
                            # receive the product physically in our stock. To avoid counting the
                            # quantity twice, we do nothing.
                            pass
                        else:
                            total += move.product_uom._compute_quantity(move.product_uom_qty, line.product_uom, rounding_method='HALF-UP')
                line._track_qty_received(total)
                line.qty_received = total

    @api.model_create_multi
    def create(self, vals_list):
        lines = super(PurchaseOrderLine, self).create(vals_list)
        lines.filtered(lambda l: l.order_id.state == 'purchase')._create_or_update_picking()
        return lines

    def write(self, values):
        for line in self.filtered(lambda l: not l.display_type):
            # PO date_planned overrides any PO line date_planned values
            if values.get('date_planned'):
                new_date = fields.Datetime.to_datetime(values['date_planned'])
                self._update_move_date_deadline(new_date)
        lines = self.filtered(lambda l: l.order_id.state == 'purchase')
        previous_product_uom_qty = {line.id: line.product_uom_qty for line in lines}
        previous_product_qty = {line.id: line.product_qty for line in lines}
        result = super(PurchaseOrderLine, self).write(values)
        if 'price_unit' in values:
            for line in lines:
                # Avoid updating kit components' stock.move
                moves = line.move_ids.filtered(lambda s: s.state not in ('cancel', 'done') and s.product_id == line.product_id)
                moves.write({'price_unit': line._get_stock_move_price_unit()})
        if 'product_qty' in values:
            lines = lines.filtered(lambda l: float_compare(previous_product_qty[l.id], l.product_qty, precision_rounding=l.product_uom.rounding) != 0)
            lines.with_context(previous_product_qty=previous_product_uom_qty)._create_or_update_picking()
        return result

    def unlink(self):
        self.move_ids._action_cancel()

        ppg_cancel_lines = self.filtered(lambda line: line.propagate_cancel)
        ppg_cancel_lines.move_dest_ids._action_cancel()

        not_ppg_cancel_lines = self.filtered(lambda line: not line.propagate_cancel)
        not_ppg_cancel_lines.move_dest_ids.write({'procure_method': 'make_to_stock'})
        not_ppg_cancel_lines.move_dest_ids._recompute_state()

        return super().unlink()

    # --------------------------------------------------
    # Business methods
    # --------------------------------------------------

    def _update_move_date_deadline(self, new_date):
        """ Updates corresponding move picking line deadline dates that are not yet completed. """
        moves_to_update = self.move_ids.filtered(lambda m: m.state not in ('done', 'cancel'))
        if not moves_to_update:
            moves_to_update = self.move_dest_ids.filtered(lambda m: m.state not in ('done', 'cancel'))
        for move in moves_to_update:
            move.date_deadline = new_date + relativedelta(days=move.company_id.po_lead)

    def _create_or_update_picking(self):
        for line in self:
            if line.product_id and line.product_id.type in ('product', 'consu'):
                # Prevent decreasing below received quantity
                if float_compare(line.product_qty, line.qty_received, line.product_uom.rounding) < 0:
                    raise UserError(_('You cannot decrease the ordered quantity below the received quantity.\n'
                                      'Create a return first.'))

                if float_compare(line.product_qty, line.qty_invoiced, line.product_uom.rounding) == -1:
                    # If the quantity is now below the invoiced quantity, create an activity on the vendor bill
                    # inviting the user to create a refund.
                    line.invoice_lines[0].move_id.activity_schedule(
                        'mail.mail_activity_data_warning',
                        note=_('The quantities on your purchase order indicate less than billed. You should ask for a refund.'))

                # If the user increased quantity of existing line or created a new line
                pickings = line.order_id.picking_ids.filtered(lambda x: x.state not in ('done', 'cancel') and x.location_dest_id.usage in ('internal', 'transit', 'customer'))
                picking = pickings and pickings[0] or False
                if not picking:
                    res = line.order_id._prepare_picking()
                    picking = self.env['stock.picking'].create(res)

                moves = line._create_stock_moves(picking)
                moves._action_confirm()._action_assign()

    def _get_stock_move_price_unit(self):
        self.ensure_one()
        line = self[0]
        order = line.order_id
        price_unit = line.price_unit
        price_unit_prec = self.env['decimal.precision'].precision_get('Product Price')
        if line.taxes_id:
            qty = line.product_qty or 1
            price_unit = line.taxes_id.with_context(round=False).compute_all(
                price_unit, currency=line.order_id.currency_id, quantity=qty, product=line.product_id, partner=line.order_id.partner_id
            )['total_void']
            price_unit = float_round(price_unit / qty, precision_digits=price_unit_prec)
        if line.product_uom.id != line.product_id.uom_id.id:
            price_unit *= line.product_uom.factor / line.product_id.uom_id.factor
        if order.currency_id != order.company_id.currency_id:
            price_unit = order.currency_id._convert(
                price_unit, order.company_id.currency_id, self.company_id, self.date_order or fields.Date.today(), round=False)
        return price_unit

    def _prepare_stock_moves(self, picking):
        """ Prepare the stock moves data for one order line. This function returns a list of
        dictionary ready to be used in stock.move's create()
        """
        self.ensure_one()
        res = []
        if self.product_id.type not in ['product', 'consu']:
            return res

        price_unit = self._get_stock_move_price_unit()
        qty = self._get_qty_procurement()

        move_dests = self.move_dest_ids
        if not move_dests:
            move_dests = self.move_ids.move_dest_ids.filtered(lambda m: m.state != 'cancel' and not m.location_dest_id.usage == 'supplier')

        if not move_dests:
            qty_to_attach = 0
            qty_to_push = self.product_qty - qty
        else:
            move_dests_initial_demand = self.product_id.uom_id._compute_quantity(
                sum(move_dests.filtered(lambda m: m.state != 'cancel' and not m.location_dest_id.usage == 'supplier').mapped('product_qty')),
                self.product_uom, rounding_method='HALF-UP')
            qty_to_attach = min(self.product_qty, move_dests_initial_demand) - qty
            qty_to_push = self.product_qty - move_dests_initial_demand

        if float_compare(qty_to_attach, 0.0, precision_rounding=self.product_uom.rounding) > 0:
            product_uom_qty, product_uom = self.product_uom._adjust_uom_quantities(qty_to_attach, self.product_id.uom_id)
            res.append(self._prepare_stock_move_vals(picking, price_unit, product_uom_qty, product_uom))
        if float_compare(qty_to_push, 0.0, precision_rounding=self.product_uom.rounding) > 0:
            product_uom_qty, product_uom = self.product_uom._adjust_uom_quantities(qty_to_push, self.product_id.uom_id)
            extra_move_vals = self._prepare_stock_move_vals(picking, price_unit, product_uom_qty, product_uom)
            extra_move_vals['move_dest_ids'] = False  # don't attach
            res.append(extra_move_vals)
        return res

    def _get_qty_procurement(self):
        self.ensure_one()
        qty = 0.0
        outgoing_moves, incoming_moves = self._get_outgoing_incoming_moves()
        for move in outgoing_moves:
            qty -= move.product_uom._compute_quantity(move.product_uom_qty, self.product_uom, rounding_method='HALF-UP')
        for move in incoming_moves:
            qty += move.product_uom._compute_quantity(move.product_uom_qty, self.product_uom, rounding_method='HALF-UP')
        return qty

    def _check_orderpoint_picking_type(self):
        warehouse_loc = self.order_id.picking_type_id.warehouse_id.view_location_id
        dest_loc = self.move_dest_ids.location_id or self.orderpoint_id.location_id
        if warehouse_loc and dest_loc and dest_loc.get_warehouse() and not warehouse_loc.parent_path in dest_loc[0].parent_path:
            raise UserError(_('For the product %s, the warehouse of the operation type (%s) is inconsistent with the location (%s) of the reordering rule (%s). Change the operation type or cancel the request for quotation.',
                              self.product_id.display_name, self.order_id.picking_type_id.display_name, self.orderpoint_id.location_id.display_name, self.orderpoint_id.display_name))

    def _prepare_stock_move_vals(self, picking, price_unit, product_uom_qty, product_uom):
        self.ensure_one()
        self._check_orderpoint_picking_type()
        product = self.product_id.with_context(lang=self.order_id.dest_address_id.lang or self.env.user.lang)
        description_picking = product._get_description(self.order_id.picking_type_id)
        if self.product_description_variants:
            description_picking += "\n" + self.product_description_variants
        date_planned = self.date_planned or self.order_id.date_planned
        return {
            # truncate to 2000 to avoid triggering index limit error
            # TODO: remove index in master?
            'name': (self.product_id.display_name or '')[:2000],
            'product_id': self.product_id.id,
            'date': date_planned,
            'date_deadline': date_planned + relativedelta(days=self.order_id.company_id.po_lead),
            'location_id': self.order_id.partner_id.property_stock_supplier.id,
            'location_dest_id': (self.orderpoint_id and not (self.move_ids | self.move_dest_ids)) and self.orderpoint_id.location_id.id or self.order_id._get_destination_location(),
            'picking_id': picking.id,
            'partner_id': self.order_id.dest_address_id.id,
            'move_dest_ids': [(4, x) for x in self.move_dest_ids.ids],
            'state': 'draft',
            'purchase_line_id': self.id,
            'company_id': self.order_id.company_id.id,
            'price_unit': price_unit,
            'picking_type_id': self.order_id.picking_type_id.id,
            'group_id': self.order_id.group_id.id,
            'origin': self.order_id.name,
            'description_picking': description_picking,
            'propagate_cancel': self.propagate_cancel,
            'warehouse_id': self.order_id.picking_type_id.warehouse_id.id,
            'product_uom_qty': product_uom_qty,
            'product_uom': product_uom.id,
            'sequence': self.sequence,
        }

    @api.model
    def _prepare_purchase_order_line_from_procurement(self, product_id, product_qty, product_uom, company_id, values, po):
        line_description = ''
        if values.get('product_description_variants'):
            line_description = values['product_description_variants']
        supplier = values.get('supplier')
        res = self._prepare_purchase_order_line(product_id, product_qty, product_uom, company_id, supplier, po)
        # We need to keep the vendor name set in _prepare_purchase_order_line. To avoid redundancy
        # in the line name, we add the line_description only if different from the product name.
        # This way, we shoud not lose any valuable information.
        if line_description and product_id.name != line_description:
            res['name'] += '\n' + line_description
        res['move_dest_ids'] = [(4, x.id) for x in values.get('move_dest_ids', [])]
        res['orderpoint_id'] = values.get('orderpoint_id', False) and values.get('orderpoint_id').id
        res['propagate_cancel'] = values.get('propagate_cancel')
        res['product_description_variants'] = values.get('product_description_variants')
        return res

    def _create_stock_moves(self, picking):
        values = []
        for line in self.filtered(lambda l: not l.display_type):
            for val in line._prepare_stock_moves(picking):
                values.append(val)
            line.move_dest_ids.created_purchase_line_id = False

        return self.env['stock.move'].create(values)

    def _find_candidate(self, product_id, product_qty, product_uom, location_id, name, origin, company_id, values):
        """ Return the record in self where the procument with values passed as
        args can be merged. If it returns an empty record then a new line will
        be created.
        """
        description_picking = ''
        if values.get('product_description_variants'):
            description_picking = values['product_description_variants']
        lines = self.filtered(
            lambda l: l.propagate_cancel == values['propagate_cancel']
            and (l.orderpoint_id == values['orderpoint_id'] if values['orderpoint_id'] and not values['move_dest_ids'] else True)
        )

        # In case 'product_description_variants' is in the values, we also filter on the PO line
        # name. This way, we can merge lines with the same description. To do so, we need the
        # product name in the context of the PO partner.
        if lines and values.get('product_description_variants'):
            partner = self.mapped('order_id.partner_id')[:1]
            product_lang = product_id.with_context(
                lang=partner.lang,
                partner_id=partner.id,
            )
            name = product_lang.display_name
            if product_lang.description_purchase:
                name += '\n' + product_lang.description_purchase
            lines = lines.filtered(lambda l: l.name == name + '\n' + description_picking)
            if lines:
                return lines[0]

        return lines and lines[0] or self.env['purchase.order.line']

    def _get_outgoing_incoming_moves(self):
        outgoing_moves = self.env['stock.move']
        incoming_moves = self.env['stock.move']

        for move in self.move_ids.filtered(lambda r: r.state != 'cancel' and not r.scrapped and self.product_id == r.product_id):
            if move.location_dest_id.usage == "supplier" and move.to_refund:
                outgoing_moves |= move
            elif move.location_dest_id.usage != "supplier":
                if not move.origin_returned_move_id or (move.origin_returned_move_id and move.to_refund):
                    incoming_moves |= move

        return outgoing_moves, incoming_moves

    def _update_date_planned(self, updated_date):
        move_to_update = self.move_ids.filtered(lambda m: m.state not in ['done', 'cancel'])
        if not self.move_ids or move_to_update:  # Only change the date if there is no move done or none
            super()._update_date_planned(updated_date)
        if move_to_update:
            self._update_move_date_deadline(updated_date)

    @api.model
    def _update_qty_received_method(self):
        """Update qty_received_method for old PO before install this module."""
        self.search(['!', ('state', 'in', ['purchase', 'done'])])._compute_qty_received_method()

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResCompany(models.Model):
    _inherit = 'res.company'

    days_to_purchase = fields.Float(
        string='Days to Purchase',
        help="Days needed to confirm a PO, define when a PO should be validated")

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    module_stock_dropshipping = fields.Boolean("Dropshipping")
    days_to_purchase = fields.Float(
        related='company_id.days_to_purchase', readonly=False)

    is_installed_sale = fields.Boolean(string="Is the Sale Module Installed")

    def get_values(self):
        res = super(ResConfigSettings, self).get_values()
        res.update(
            is_installed_sale=self.env['ir.module.module'].search([('name', '=', 'sale'), ('state', '=', 'installed')]).id,
        )
        return res

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import timedelta, datetime, time
from collections import defaultdict

from odoo import api, fields, models


class ResPartner(models.Model):
    _inherit = 'res.partner'

    purchase_line_ids = fields.One2many('purchase.order.line', 'partner_id', string="Purchase Lines")
    on_time_rate = fields.Float(
        "On-Time Delivery Rate", compute='_compute_on_time_rate',
        help="Over the past x days; the number of products received on time divided by the number of ordered products."\
            "x is either the System Parameter purchase_stock.on_time_delivery_days or the default 365")

    @api.depends('purchase_line_ids')
    def _compute_on_time_rate(self):
        date_order_days_delta = int(self.env['ir.config_parameter'].sudo().get_param('purchase_stock.on_time_delivery_days', default='365'))
        order_lines = self.env['purchase.order.line'].search([
            ('partner_id', 'in', self.ids),
            ('date_order', '>', fields.Date.today() - timedelta(date_order_days_delta)),
            ('qty_received', '!=', 0),
            ('order_id.state', 'in', ['done', 'purchase']),
            ('product_id', 'in', self.env['product.product'].sudo()._search([('type', '!=', 'service')]))
        ])
        lines_qty_done = defaultdict(lambda: 0)
        moves = self.env['stock.move'].search([
            ('purchase_line_id', 'in', order_lines.ids),
            ('state', '=', 'done')])
        # Fetch fields from db and put them in cache.
        order_lines.read(['date_planned', 'partner_id', 'product_uom_qty'], load='')
        moves.read(['purchase_line_id', 'date'], load='')
        moves = moves.filtered(lambda m: m.date.date() <= m.purchase_line_id.date_planned.date())
        for move, qty_done in zip(moves, moves.mapped('quantity_done')):
            lines_qty_done[move.purchase_line_id.id] += qty_done
        partner_dict = {}
        for line in order_lines:
            on_time, ordered = partner_dict.get(line.partner_id, (0, 0))
            ordered += line.product_uom_qty
            on_time += lines_qty_done[line.id]
            partner_dict[line.partner_id] = (on_time, ordered)
        seen_partner = self.env['res.partner']
        for partner, numbers in partner_dict.items():
            seen_partner |= partner
            on_time, ordered = numbers
            partner.on_time_rate = on_time / ordered * 100 if ordered else -1   # use negative number to indicate no data
        (self - seen_partner).on_time_rate = -1

```

## File: models\stock.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.tools.float_utils import float_round, float_is_zero
from odoo.exceptions import UserError
from odoo.osv.expression import AND

class StockPicking(models.Model):
    _inherit = 'stock.picking'

    purchase_id = fields.Many2one('purchase.order', related='move_lines.purchase_line_id.order_id',
        string="Purchase Orders", readonly=True)


class StockMove(models.Model):
    _inherit = 'stock.move'

    purchase_line_id = fields.Many2one('purchase.order.line',
        'Purchase Order Line', ondelete='set null', index=True, readonly=True)
    created_purchase_line_id = fields.Many2one('purchase.order.line',
        'Created Purchase Order Line', ondelete='set null', readonly=True, copy=False, index=True)

    @api.model
    def _prepare_merge_moves_distinct_fields(self):
        distinct_fields = super(StockMove, self)._prepare_merge_moves_distinct_fields()
        distinct_fields += ['purchase_line_id', 'created_purchase_line_id']
        return distinct_fields

    @api.model
    def _prepare_merge_move_sort_method(self, move):
        move.ensure_one()
        keys_sorted = super(StockMove, self)._prepare_merge_move_sort_method(move)
        keys_sorted += [move.purchase_line_id.id, move.created_purchase_line_id.id]
        return keys_sorted

    def _get_price_unit(self):
        """ Returns the unit price for the move"""
        self.ensure_one()
        if not self.origin_returned_move_id and self.purchase_line_id and self.product_id.id == self.purchase_line_id.product_id.id:
            price_unit_prec = self.env['decimal.precision'].precision_get('Product Price')
            line = self.purchase_line_id
            order = line.order_id
            price_unit = line.price_unit
            if line.taxes_id:
                qty = line.product_qty or 1
                price_unit = line.taxes_id.with_context(round=False).compute_all(price_unit, currency=line.order_id.currency_id, quantity=qty)['total_void']
                price_unit = float_round(price_unit / qty, precision_digits=price_unit_prec)
            if line.product_uom.id != line.product_id.uom_id.id:
                price_unit *= line.product_uom.factor / line.product_id.uom_id.factor
            if order.currency_id != order.company_id.currency_id:
                # The date must be today, and not the date of the move since the move move is still
                # in assigned state. However, the move date is the scheduled date until move is
                # done, then date of actual move processing. See:
                # https://github.com/odoo/odoo/blob/2f789b6863407e63f90b3a2d4cc3be09815f7002/addons/stock/models/stock_move.py#L36
                price_unit = order.currency_id._convert(
                    price_unit, order.company_id.currency_id, order.company_id, fields.Date.context_today(self), round=False)
            return price_unit
        return super(StockMove, self)._get_price_unit()

    def _generate_valuation_lines_data(self, partner_id, qty, debit_value, credit_value, debit_account_id, credit_account_id, description):
        """ Overridden from stock_account to support amount_currency on valuation lines generated from po
        """
        self.ensure_one()

        rslt = super(StockMove, self)._generate_valuation_lines_data(partner_id, qty, debit_value, credit_value, debit_account_id, credit_account_id, description)
        if self.purchase_line_id:
            purchase_currency = self.purchase_line_id.currency_id
            if purchase_currency != self.company_id.currency_id:
                if(self.purchase_line_id.product_id.cost_method == 'standard'):
                    purchase_price_unit = self.purchase_line_id.product_id.cost_currency_id._convert(
                        self.purchase_line_id.product_id.standard_price,
                        purchase_currency,
                        self.company_id,
                        self.date,
                        round=False,
                    )
                else:
                    # Do not use price_unit since we want the price tax excluded. And by the way, qty
                    # is in the UOM of the product, not the UOM of the PO line.
                    purchase_price_unit = (
                        self.purchase_line_id.price_subtotal / self.purchase_line_id.product_uom_qty
                        if self.purchase_line_id.product_uom_qty
                        else self.purchase_line_id.price_unit
                    )
                currency_move_valuation = purchase_currency.round(purchase_price_unit * abs(qty))
                rslt['credit_line_vals']['amount_currency'] = rslt['credit_line_vals']['credit'] and -currency_move_valuation or currency_move_valuation
                rslt['credit_line_vals']['currency_id'] = purchase_currency.id
                rslt['debit_line_vals']['amount_currency'] = rslt['debit_line_vals']['credit'] and -currency_move_valuation or currency_move_valuation
                rslt['debit_line_vals']['currency_id'] = purchase_currency.id
        return rslt

    def _prepare_extra_move_vals(self, qty):
        vals = super(StockMove, self)._prepare_extra_move_vals(qty)
        vals['purchase_line_id'] = self.purchase_line_id.id
        return vals

    def _prepare_move_split_vals(self, uom_qty):
        vals = super(StockMove, self)._prepare_move_split_vals(uom_qty)
        vals['purchase_line_id'] = self.purchase_line_id.id
        return vals

    def _prepare_procurement_values(self):
        proc_values = super()._prepare_procurement_values()
        if self.restrict_partner_id:
            proc_values['supplierinfo_name'] = self.restrict_partner_id
            self.restrict_partner_id = False
        return proc_values

    def _clean_merged(self):
        super(StockMove, self)._clean_merged()
        self.write({'created_purchase_line_id': False})

    def _get_upstream_documents_and_responsibles(self, visited):
        if self.created_purchase_line_id and self.created_purchase_line_id.state not in ('done', 'cancel'):
            return [(self.created_purchase_line_id.order_id, self.created_purchase_line_id.order_id.user_id, visited)]
        elif self.purchase_line_id and self.purchase_line_id.state not in ('done', 'cancel'):
            return[(self.purchase_line_id.order_id, self.purchase_line_id.order_id.user_id, visited)]
        else:
            return super(StockMove, self)._get_upstream_documents_and_responsibles(visited)

    def _get_related_invoices(self):
        """ Overridden to return the vendor bills related to this stock move.
        """
        rslt = super(StockMove, self)._get_related_invoices()
        rslt += self.mapped('picking_id.purchase_id.invoice_ids').filtered(lambda x: x.state == 'posted')
        return rslt


    def _get_source_document(self):
        res = super()._get_source_document()
        return self.purchase_line_id.order_id or res

    def _get_valuation_price_and_qty(self, related_aml, to_curr):
        valuation_price_unit_total = 0
        valuation_total_qty = 0
        for val_stock_move in self:
            # In case val_stock_move is a return move, its valuation entries have been made with the
            # currency rate corresponding to the original stock move
            valuation_date = val_stock_move.origin_returned_move_id.date or val_stock_move.date
            svl = val_stock_move.with_context(active_test=False).mapped('stock_valuation_layer_ids').filtered(
                lambda l: l.quantity)
            layers_qty = sum(svl.mapped('quantity'))
            layers_values = sum(svl.mapped('value'))
            valuation_price_unit_total += related_aml.company_currency_id._convert(
                layers_values, to_curr, related_aml.company_id, valuation_date, round=False,
            )
            valuation_total_qty += layers_qty
        if float_is_zero(valuation_total_qty, precision_rounding=related_aml.product_uom_id.rounding or related_aml.product_id.uom_id.rounding):
            raise UserError(
                _('Odoo is not able to generate the anglo saxon entries. The total valuation of %s is zero.') % related_aml.product_id.display_name)
        return valuation_price_unit_total, valuation_total_qty

    def _is_purchase_return(self):
        self.ensure_one()
        return self.location_dest_id.usage == "supplier" or (
                self.location_dest_id.usage == "internal"
                and self.location_id.usage != "supplier"
                and self.warehouse_id
                and self.location_dest_id not in self.env["stock.location"].search([("id", "child_of", self.warehouse_id.view_location_id.id)])
        )


class StockWarehouse(models.Model):
    _inherit = 'stock.warehouse'

    buy_to_resupply = fields.Boolean('Buy to Resupply', default=True,
                                     help="When products are bought, they can be delivered to this warehouse")
    buy_pull_id = fields.Many2one('stock.rule', 'Buy rule')

    def _get_global_route_rules_values(self):
        rules = super(StockWarehouse, self)._get_global_route_rules_values()
        location_id = self.in_type_id.default_location_dest_id
        rules.update({
            'buy_pull_id': {
                'depends': ['reception_steps', 'buy_to_resupply'],
                'create_values': {
                    'action': 'buy',
                    'picking_type_id': self.in_type_id.id,
                    'group_propagation_option': 'none',
                    'company_id': self.company_id.id,
                    'route_id': self._find_global_route('purchase_stock.route_warehouse0_buy', _('Buy')).id,
                    'propagate_cancel': self.reception_steps != 'one_step',
                },
                'update_values': {
                    'active': self.buy_to_resupply,
                    'name': self._format_rulename(location_id, False, 'Buy'),
                    'location_id': location_id.id,
                    'propagate_cancel': self.reception_steps != 'one_step',
                }
            }
        })
        return rules

    def _get_all_routes(self):
        routes = super(StockWarehouse, self)._get_all_routes()
        routes |= self.filtered(lambda self: self.buy_to_resupply and self.buy_pull_id and self.buy_pull_id.route_id).mapped('buy_pull_id').mapped('route_id')
        return routes

    def get_rules_dict(self):
        result = super(StockWarehouse, self).get_rules_dict()
        for warehouse in self:
            result[warehouse.id].update(warehouse._get_receive_rules_dict())
        return result

    def _get_routes_values(self):
        routes = super(StockWarehouse, self)._get_routes_values()
        routes.update(self._get_receive_routes_values('buy_to_resupply'))
        return routes

    def _update_name_and_code(self, name=False, code=False):
        res = super(StockWarehouse, self)._update_name_and_code(name, code)
        warehouse = self[0]
        #change the buy stock rule name
        if warehouse.buy_pull_id and name:
            warehouse.buy_pull_id.write({'name': warehouse.buy_pull_id.name.replace(warehouse.name, name, 1)})
        return res


class ReturnPicking(models.TransientModel):
    _inherit = "stock.return.picking"

    def _prepare_move_default_values(self, return_line, new_picking):
        vals = super(ReturnPicking, self)._prepare_move_default_values(return_line, new_picking)
        vals['purchase_line_id'] = return_line.move_id.purchase_line_id.id
        return vals


class Orderpoint(models.Model):
    _inherit = "stock.warehouse.orderpoint"

    show_supplier = fields.Boolean('Show supplier column', compute='_compute_show_suppplier')
    supplier_id = fields.Many2one(
        'product.supplierinfo', string='Vendor', check_company=True,
        domain="['|', ('product_id', '=', product_id), '&', ('product_id', '=', False), ('product_tmpl_id', '=', product_tmpl_id)]")

    @api.depends('product_id.purchase_order_line_ids.product_qty', 'product_id.purchase_order_line_ids.state')
    def _compute_qty(self):
        """ Extend to add more depends values """
        return super()._compute_qty()

    @api.depends('route_id')
    def _compute_show_suppplier(self):
        buy_route = []
        for res in self.env['stock.rule'].search_read([('action', '=', 'buy')], ['route_id']):
            buy_route.append(res['route_id'][0])
        for orderpoint in self:
            orderpoint.show_supplier = orderpoint.route_id.id in buy_route

    def action_view_purchase(self):
        """ This function returns an action that display existing
        purchase orders of given orderpoint.
        """
        result = self.env['ir.actions.act_window']._for_xml_id('purchase.purchase_rfq')

        # Remvove the context since the action basically display RFQ and not PO.
        result['context'] = {}
        order_line_ids = self.env['purchase.order.line'].search([('orderpoint_id', '=', self.id)])
        purchase_ids = order_line_ids.mapped('order_id')

        result['domain'] = "[('id','in',%s)]" % (purchase_ids.ids)

        return result

    def _get_replenishment_order_notification(self):
        self.ensure_one()
        domain = [('orderpoint_id', 'in', self.ids)]
        if self.env.context.get('written_after'):
            domain = AND([domain, [('write_date', '>', self.env.context.get('written_after'))]])
        order = self.env['purchase.order.line'].search(domain, limit=1).order_id
        if order:
            action = self.env.ref('purchase.action_rfq_form')
            return {
                'type': 'ir.actions.client',
                'tag': 'display_notification',
                'params': {
                    'title': _('The following replenishment order has been generated'),
                    'message': '%s',
                    'links': [{
                        'label': order.display_name,
                        'url': f'#action={action.id}&id={order.id}&model=purchase.order',
                    }],
                    'sticky': False,
                }
            }
        return super()._get_replenishment_order_notification()

    def _prepare_procurement_values(self, date=False, group=False):
        values = super()._prepare_procurement_values(date=date, group=group)
        values['supplierinfo_id'] = self.supplier_id
        return values

    def _quantity_in_progress(self):
        res = super()._quantity_in_progress()
        qty_by_product_location, dummy = self.product_id._get_quantity_in_progress(self.location_id.ids)
        for orderpoint in self:
            product_qty = qty_by_product_location.get((orderpoint.product_id.id, orderpoint.location_id.id), 0.0)
            product_uom_qty = orderpoint.product_id.uom_id._compute_quantity(product_qty, orderpoint.product_uom, round=False)
            res[orderpoint.id] += product_uom_qty
        return res

    def _set_default_route_id(self):
        route_id = self.env['stock.rule'].search([
            ('action', '=', 'buy')
        ]).route_id
        orderpoint_wh_supplier = self.filtered(lambda o: o.product_id.seller_ids)
        if route_id and orderpoint_wh_supplier:
            orderpoint_wh_supplier.route_id = route_id[0].id
        return super()._set_default_route_id()


class ProductionLot(models.Model):
    _inherit = 'stock.production.lot'

    purchase_order_ids = fields.Many2many('purchase.order', string="Purchase Orders", compute='_compute_purchase_order_ids', readonly=True, store=False)
    purchase_order_count = fields.Integer('Purchase order count', compute='_compute_purchase_order_ids')

    @api.depends('name')
    def _compute_purchase_order_ids(self):
        for lot in self:
            stock_moves = self.env['stock.move.line'].search([
                ('lot_id', '=', lot.id),
                ('state', '=', 'done')
            ]).mapped('move_id')
            stock_moves = stock_moves.search([('id', 'in', stock_moves.ids)]).filtered(
                lambda move: move.picking_id.location_id.usage == 'supplier' and move.state == 'done')
            lot.purchase_order_ids = stock_moves.mapped('purchase_line_id.order_id')
            lot.purchase_order_count = len(lot.purchase_order_ids)

    def action_view_po(self):
        self.ensure_one()
        action = self.env["ir.actions.actions"]._for_xml_id("purchase.purchase_form_action")
        action['domain'] = [('id', 'in', self.mapped('purchase_order_ids.id'))]
        action['context'] = dict(self._context, create=False)
        return action


class ProcurementGroup(models.Model):
    _inherit = 'procurement.group'

    @api.model
    def run(self, procurements, raise_user_error=True):
        wh_by_comp = dict()
        for procurement in procurements:
            routes = procurement.values.get('route_ids')
            if routes and any(r.action == 'buy' for r in routes.rule_ids):
                company = procurement.company_id
                if company not in wh_by_comp:
                    wh_by_comp[company] = self.env['stock.warehouse'].search([('company_id', '=', company.id)])
                wh = wh_by_comp[company]
                procurement.values['route_ids'] |= wh.reception_route_id
        return super().run(procurements, raise_user_error=raise_user_error)

```

## File: models\stock_rule.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict
from datetime import datetime
from dateutil.relativedelta import relativedelta
from itertools import groupby

from odoo import api, fields, models, SUPERUSER_ID, _
from odoo.addons.stock.models.stock_rule import ProcurementException


class StockRule(models.Model):
    _inherit = 'stock.rule'

    action = fields.Selection(selection_add=[
        ('buy', 'Buy')
    ], ondelete={'buy': 'cascade'})

    def _get_message_dict(self):
        message_dict = super(StockRule, self)._get_message_dict()
        dummy, destination, dummy = self._get_message_values()
        message_dict.update({
            'buy': _('When products are needed in <b>%s</b>, <br/> '
                     'a request for quotation is created to fulfill the need.<br/>'
                     'Note: This rule will be used in combination with the rules<br/>'
                     'of the reception route(s)') % (destination)
        })
        return message_dict

    @api.depends('action')
    def _compute_picking_type_code_domain(self):
        remaining = self.browse()
        for rule in self:
            if rule.action == 'buy':
                rule.picking_type_code_domain = 'incoming'
            else:
                remaining |= rule
        super(StockRule, remaining)._compute_picking_type_code_domain()

    @api.onchange('action')
    def _onchange_action(self):
        if self.action == 'buy':
            self.location_src_id = False

    @api.model
    def _run_buy(self, procurements):
        procurements_by_po_domain = defaultdict(list)
        errors = []
        for procurement, rule in procurements:

            # Get the schedule date in order to find a valid seller
            procurement_date_planned = fields.Datetime.from_string(procurement.values['date_planned'])
            schedule_date = (procurement_date_planned - relativedelta(days=procurement.company_id.po_lead))

            supplier = False
            if procurement.values.get('supplierinfo_id'):
                supplier = procurement.values['supplierinfo_id']
            else:
                supplier = procurement.product_id.with_company(procurement.company_id.id)._select_seller(
                    partner_id=procurement.values.get("supplierinfo_name"),
                    quantity=procurement.product_qty,
                    date=max(schedule_date.date(), fields.Date.today()),
                    uom_id=procurement.product_uom)

            # Fall back on a supplier for which no price may be defined. Not ideal, but better than
            # blocking the user.
            supplier = supplier or procurement.product_id._prepare_sellers(False).filtered(
                lambda s: not s.company_id or s.company_id == procurement.company_id
            )[:1]

            if not supplier:
                msg = _('There is no matching vendor price to generate the purchase order for product %s (no vendor defined, minimum quantity not reached, dates not valid, ...). Go on the product form and complete the list of vendors.') % (procurement.product_id.display_name)
                errors.append((procurement, msg))

            partner = supplier.name
            # we put `supplier_info` in values for extensibility purposes
            procurement.values['supplier'] = supplier
            procurement.values['propagate_cancel'] = rule.propagate_cancel

            domain = rule._make_po_get_domain(procurement.company_id, procurement.values, partner)
            procurements_by_po_domain[domain].append((procurement, rule))

        if errors:
            raise ProcurementException(errors)

        for domain, procurements_rules in procurements_by_po_domain.items():
            # Get the procurements for the current domain.
            # Get the rules for the current domain. Their only use is to create
            # the PO if it does not exist.
            procurements, rules = zip(*procurements_rules)

            # Get the set of procurement origin for the current domain.
            origins = set([p.origin for p in procurements])
            # Check if a PO exists for the current domain.
            po = self.env['purchase.order'].sudo().search([dom for dom in domain], limit=1)
            company_id = procurements[0].company_id
            if not po:
                # We need a rule to generate the PO. However the rule generated
                # the same domain for PO and the _prepare_purchase_order method
                # should only uses the common rules's fields.
                vals = rules[0]._prepare_purchase_order(company_id, origins, [p.values for p in procurements])
                # The company_id is the same for all procurements since
                # _make_po_get_domain add the company in the domain.
                # We use SUPERUSER_ID since we don't want the current user to be follower of the PO.
                # Indeed, the current user may be a user without access to Purchase, or even be a portal user.
                po = self.env['purchase.order'].with_company(company_id).with_user(SUPERUSER_ID).create(vals)
            else:
                # If a purchase order is found, adapt its `origin` field.
                if po.origin:
                    missing_origins = origins - set(po.origin.split(', '))
                    if missing_origins:
                        po.write({'origin': po.origin + ', ' + ', '.join(missing_origins)})
                else:
                    po.write({'origin': ', '.join(origins)})

            procurements_to_merge = self._get_procurements_to_merge(procurements)
            procurements = self._merge_procurements(procurements_to_merge)

            po_lines_by_product = {}
            grouped_po_lines = groupby(po.order_line.filtered(lambda l: not l.display_type and l.product_uom == l.product_id.uom_po_id).sorted(lambda l: l.product_id.id), key=lambda l: l.product_id.id)
            for product, po_lines in grouped_po_lines:
                po_lines_by_product[product] = self.env['purchase.order.line'].concat(*list(po_lines))
            po_line_values = []
            for procurement in procurements:
                po_lines = po_lines_by_product.get(procurement.product_id.id, self.env['purchase.order.line'])
                po_line = po_lines._find_candidate(*procurement)

                if po_line:
                    # If the procurement can be merge in an existing line. Directly
                    # write the new values on it.
                    vals = self._update_purchase_order_line(procurement.product_id,
                        procurement.product_qty, procurement.product_uom, company_id,
                        procurement.values, po_line)
                    po_line.write(vals)
                else:
                    # If it does not exist a PO line for current procurement.
                    # Generate the create values for it and add it to a list in
                    # order to create it in batch.
                    partner = procurement.values['supplier'].name
                    po_line_values.append(self.env['purchase.order.line']._prepare_purchase_order_line_from_procurement(
                        procurement.product_id, procurement.product_qty,
                        procurement.product_uom, procurement.company_id,
                        procurement.values, po))
            self.env['purchase.order.line'].sudo().create(po_line_values)

    def _get_lead_days(self, product):
        """Add the company security lead time, days to purchase and the supplier
        delay to the cumulative delay and cumulative description. The days to
        purchase and company lead time are always displayed for onboarding
        purpose in order to indicate that those options are available.
        """
        delay, delay_description = super()._get_lead_days(product)
        bypass_delay_description = self.env.context.get('bypass_delay_description')
        buy_rule = self.filtered(lambda r: r.action == 'buy')
        seller = product.with_company(buy_rule.company_id)._select_seller(quantity=None)
        if not buy_rule or not seller:
            return delay, delay_description
        buy_rule.ensure_one()
        supplier_delay = seller[0].delay
        if supplier_delay and not bypass_delay_description:
            delay_description += '<tr><td>%s</td><td class="text-right">+ %d %s</td></tr>' % (_('Vendor Lead Time'), supplier_delay, _('day(s)'))
        security_delay = buy_rule.picking_type_id.company_id.po_lead
        if not bypass_delay_description:
            delay_description += '<tr><td>%s</td><td class="text-right">+ %d %s</td></tr>' % (_('Purchase Security Lead Time'), security_delay, _('day(s)'))
        days_to_purchase = buy_rule.company_id.days_to_purchase
        if not bypass_delay_description:
            delay_description += '<tr><td>%s</td><td class="text-right">+ %d %s</td></tr>' % (_('Days to Purchase'), days_to_purchase, _('day(s)'))
        return delay + supplier_delay + security_delay + days_to_purchase, delay_description

    @api.model
    def _get_procurements_to_merge_groupby(self, procurement):
        # Do not group procument from different orderpoint. 1. _quantity_in_progress
        # directly depends from the orderpoint_id on the line. 2. The stock move
        # generated from the order line has the orderpoint's location as
        # destination location. In case of move_dest_ids those two points are not
        # necessary anymore since those values are taken from destination moves.
        return procurement.product_id, procurement.product_uom, procurement.values['propagate_cancel'],\
            procurement.values.get('product_description_variants'),\
            (procurement.values.get('orderpoint_id') and not procurement.values.get('move_dest_ids')) and procurement.values['orderpoint_id']

    @api.model
    def _get_procurements_to_merge_sorted(self, procurement):
        return procurement.product_id.id, procurement.product_uom.id, procurement.values['propagate_cancel'],\
            procurement.values.get('product_description_variants'),\
            (procurement.values.get('orderpoint_id') and not procurement.values.get('move_dest_ids')) and procurement.values['orderpoint_id']

    @api.model
    def _get_procurements_to_merge(self, procurements):
        """ Get a list of procurements values and create groups of procurements
        that would use the same purchase order line.
        params procurements_list list: procurements requests (not ordered nor
        sorted).
        return list: procurements requests grouped by their product_id.
        """
        procurements_to_merge = []

        for k, procurements in groupby(sorted(procurements, key=self._get_procurements_to_merge_sorted), key=self._get_procurements_to_merge_groupby):
            procurements_to_merge.append(list(procurements))
        return procurements_to_merge

    @api.model
    def _merge_procurements(self, procurements_to_merge):
        """ Merge the quantity for procurements requests that could use the same
        order line.
        params similar_procurements list: list of procurements that have been
        marked as 'alike' from _get_procurements_to_merge method.
        return a list of procurements values where values of similar_procurements
        list have been merged.
        """
        merged_procurements = []
        for procurements in procurements_to_merge:
            quantity = 0
            move_dest_ids = self.env['stock.move']
            orderpoint_id = self.env['stock.warehouse.orderpoint']
            for procurement in procurements:
                if procurement.values.get('move_dest_ids'):
                    move_dest_ids |= procurement.values['move_dest_ids']
                if not orderpoint_id and procurement.values.get('orderpoint_id'):
                    orderpoint_id = procurement.values['orderpoint_id']
                quantity += procurement.product_qty
            # The merged procurement can be build from an arbitrary procurement
            # since they were mark as similar before. Only the quantity and
            # some keys in values are updated.
            values = dict(procurement.values)
            values.update({
                'move_dest_ids': move_dest_ids,
                'orderpoint_id': orderpoint_id,
            })
            merged_procurement = self.env['procurement.group'].Procurement(
                procurement.product_id, quantity, procurement.product_uom,
                procurement.location_id, procurement.name, procurement.origin,
                procurement.company_id, values
            )
            merged_procurements.append(merged_procurement)
        return merged_procurements

    def _update_purchase_order_line(self, product_id, product_qty, product_uom, company_id, values, line):
        partner = values['supplier'].name
        procurement_uom_po_qty = product_uom._compute_quantity(product_qty, product_id.uom_po_id)
        seller = product_id.with_company(company_id)._select_seller(
            partner_id=partner,
            quantity=line.product_qty + procurement_uom_po_qty,
            date=line.order_id.date_order and line.order_id.date_order.date(),
            uom_id=product_id.uom_po_id)

        price_unit = self.env['account.tax']._fix_tax_included_price_company(seller.price, line.product_id.supplier_taxes_id, line.taxes_id, company_id) if seller else 0.0
        if price_unit and seller and line.order_id.currency_id and seller.currency_id != line.order_id.currency_id:
            price_unit = seller.currency_id._convert(
                price_unit, line.order_id.currency_id, line.order_id.company_id, fields.Date.today())

        res = {
            'product_qty': line.product_qty + procurement_uom_po_qty,
            'price_unit': price_unit,
            'move_dest_ids': [(4, x.id) for x in values.get('move_dest_ids', [])]
        }
        orderpoint_id = values.get('orderpoint_id')
        if orderpoint_id:
            res['orderpoint_id'] = orderpoint_id.id
        return res

    def _prepare_purchase_order(self, company_id, origins, values):
        """ Create a purchase order for procuremets that share the same domain
        returned by _make_po_get_domain.
        params values: values of procurements
        params origins: procuremets origins to write on the PO
        """
        purchase_date = min([fields.Datetime.from_string(value['date_planned']) - relativedelta(days=int(value['supplier'].delay)) for value in values])

        purchase_date = (purchase_date - relativedelta(days=company_id.po_lead))


        # Since the procurements are grouped if they share the same domain for
        # PO but the PO does not exist. In this case it will create the PO from
        # the common procurements values. The common values are taken from an
        # arbitrary procurement. In this case the first.
        values = values[0]
        partner = values['supplier'].name

        fpos = self.env['account.fiscal.position'].with_company(company_id).get_fiscal_position(partner.id)

        gpo = self.group_propagation_option
        group = (gpo == 'fixed' and self.group_id.id) or \
                (gpo == 'propagate' and values.get('group_id') and values['group_id'].id) or False

        return {
            'partner_id': partner.id,
            'user_id': False,
            'picking_type_id': self.picking_type_id.id,
            'company_id': company_id.id,
            'currency_id': partner.with_company(company_id).property_purchase_currency_id.id or company_id.currency_id.id,
            'dest_address_id': values.get('partner_id', False),
            'origin': ', '.join(origins),
            'payment_term_id': partner.with_company(company_id).property_supplier_payment_term_id.id,
            'date_order': purchase_date,
            'fiscal_position_id': fpos.id,
            'group_id': group
        }

    def _make_po_get_domain(self, company_id, values, partner):
        gpo = self.group_propagation_option
        group = (gpo == 'fixed' and self.group_id) or \
                (gpo == 'propagate' and 'group_id' in values and values['group_id']) or False

        domain = (
            ('partner_id', '=', partner.id),
            ('state', '=', 'draft'),
            ('picking_type_id', '=', self.picking_type_id.id),
            ('company_id', '=', company_id.id),
            ('user_id', '=', False),
        )
        delta_days = self.env['ir.config_parameter'].sudo().get_param('purchase_stock.delta_days_merge')
        if delta_days is not False:
            procurement_date = fields.Date.to_date(values['date_planned']) - relativedelta(days=int(values['supplier'].delay) + company_id.po_lead)
            delta_days = int(delta_days)
            domain += (
                ('date_order', '<=', datetime.combine(procurement_date + relativedelta(days=delta_days), datetime.max.time())),
                ('date_order', '>=', datetime.combine(procurement_date - relativedelta(days=delta_days), datetime.min.time()))
            )
        if group:
            domain += (('group_id', '=', group.id),)
        return domain

    def _push_prepare_move_copy_values(self, move_to_copy, new_date):
        res = super(StockRule, self)._push_prepare_move_copy_values(move_to_copy, new_date)
        res['purchase_line_id'] = None
        return res

    def _get_stock_move_values(self, product_id, product_qty, product_uom, location_id, name, origin, company_id, values):
        move_values = super()._get_stock_move_values(product_id, product_qty, product_uom, location_id, name, origin, company_id, values)
        if values.get('supplierinfo_name'):
            move_values['restrict_partner_id'] = values['supplierinfo_name'].id
        elif values.get('supplierinfo_id'):
            partner = values['supplierinfo_id'].name
            move_values['restrict_partner_id'] = partner.id
        return move_values

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_invoice
from . import product
from . import purchase
from . import res_company
from . import res_config_settings
from . import res_partner
from . import stock
from . import stock_rule

```

## File: report\purchase_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re

from odoo import api, fields, models
from odoo.exceptions import UserError
from odoo.osv.expression import expression


class PurchaseReport(models.Model):
    _inherit = "purchase.report"

    picking_type_id = fields.Many2one('stock.warehouse', 'Warehouse', readonly=True)
    avg_receipt_delay = fields.Float(
        'Average Receipt Delay', digits=(16, 2), readonly=True, store=False,  # needs store=False to prevent showing up as a 'measure' option
        help="Amount of time between expected and effective receipt date. Due to a hack needed to calculate this, \
              every record will show the same average value, therefore only use this as an aggregated value with group_operator=avg")
    effective_date = fields.Datetime(string="Effective Date")

    def _select(self):
        return super(PurchaseReport, self)._select() + ", spt.warehouse_id as picking_type_id, po.effective_date as effective_date"

    def _from(self):
        return super(PurchaseReport, self)._from() + " left join stock_picking_type spt on (spt.id=po.picking_type_id)"

    def _group_by(self):
        return super(PurchaseReport, self)._group_by() + ", spt.warehouse_id, effective_date"

    @api.model
    def read_group(self, domain, fields, groupby, offset=0, limit=None, orderby=False, lazy=True):
        """ This is a hack to allow us to correctly calculate the average of PO specific date values since
            the normal report query result will duplicate PO values across its PO lines during joins and
            lead to incorrect aggregation values.

            Only the AVG operator is supported for avg_receipt_delay.
        """
        avg_receipt_delay = next((field for field in fields if re.search(r'\bavg_receipt_delay\b', field)), False)

        if avg_receipt_delay:
            fields.remove(avg_receipt_delay)
            if any(field.split(':')[1].split('(')[0] != 'avg' for field in [avg_receipt_delay] if field):
                raise UserError("Value: 'avg_receipt_delay' should only be used to show an average. If you are seeing this message then it is being accessed incorrectly.")

        res = []
        if fields:
            res = super(PurchaseReport, self).read_group(domain, fields, groupby, offset=offset, limit=limit, orderby=orderby, lazy=lazy)

        if not res and avg_receipt_delay:
            res = [{}]

        if avg_receipt_delay:
            query = """ SELECT AVG(receipt_delay.po_receipt_delay)::decimal(16,2) AS avg_receipt_delay
                          FROM (
                              SELECT extract(epoch from age(po.effective_date, po.date_planned))/(24*60*60) AS po_receipt_delay
                              FROM purchase_order po
                              WHERE po.id IN (
                                  SELECT "purchase_report"."order_id" FROM %s WHERE %s)
                              ) AS receipt_delay
                    """

            subdomain = domain + [('company_id', '=', self.env.company.id), ('effective_date', '!=', False)]
            subtables, subwhere, subparams = expression(subdomain, self).query.get_sql()

            self.env.cr.execute(query % (subtables, subwhere), subparams)
            res[0].update({
                '__count': 1,
                avg_receipt_delay.split(':')[0]: self.env.cr.fetchall()[0][0],
            })
        return res

```

## File: report\purchase_report_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="report_purchaseorder_document" inherit_id="purchase.report_purchaseorder_document">
        <xpath expr="//t[@t-if='o.dest_address_id']" position="after">
            <t t-else="">
                <t t-set="information_block">
                    <strong>Shipping address:</strong>
                    <div t-if="o.picking_type_id and o.picking_type_id.warehouse_id">
                        <span t-field="o.picking_type_id.warehouse_id.name"/>
                        <div t-field="o.picking_type_id.warehouse_id.partner_id" t-options='{"widget": "contact", "fields": ["address", "phone"], "no_marker": True, "phone_icons": True}'/>
                    </div>
                </t>
            </t>
        </xpath>
        <xpath expr="//div[@t-if='o.date_order']" position="after">
            <div t-if="o.incoterm_id" class="col-3 bm-2">
                <strong>Incoterm:</strong>
                <p t-field="o.incoterm_id.code" class="m-0"/>
            </div>
        </xpath>
    </template>

    <template id="report_purchasequotation_document" inherit_id="purchase.report_purchasequotation_document">
        <xpath expr="//t[@t-if='o.dest_address_id']" position="after">
            <t t-else="">
                <t t-set="information_block">
                    <strong>Shipping address:</strong>
                    <div t-if="o.picking_type_id and o.picking_type_id.warehouse_id">
                        <span t-field="o.picking_type_id.warehouse_id.name"/>
                        <div t-field="o.picking_type_id.warehouse_id.partner_id" t-options='{"widget": "contact", "fields": ["address", "phone"], "no_marker": True, "phone_icons": True}'/>
                    </div>
                </t>
            </t>
        </xpath>
        <xpath expr="//span[@t-field='o.name']/.." position="after">
            <div id="informations" class="row mt16 mb16">
                <div t-if="o.incoterm_id" class="col-3 bm-2">
                    <strong>Incoterm:</strong>
                    <p t-field="o.incoterm_id.code" class="m-0"/>
                </div>
            </div>
        </xpath>
    </template>

</odoo>

```

## File: report\purchase_report_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="purchase_report_view_search" model="ir.ui.view">
        <field name="name">purchase.report.search.stock</field>
        <field name="model">purchase.report</field>
        <field name="inherit_id" ref="purchase.view_purchase_order_search"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='user_id']" position="after">
                <field name="picking_type_id"/>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: report\report_stock_forecasted.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class ReplenishmentReport(models.AbstractModel):
    _inherit = 'report.stock.report_product_product_replenishment'

    def _compute_draft_quantity_count(self, product_template_ids, product_variant_ids, wh_location_ids):
        res = super()._compute_draft_quantity_count(product_template_ids, product_variant_ids, wh_location_ids)
        domain = [('state', 'in', ['draft', 'sent', 'to approve'])]
        domain += self._product_purchase_domain(product_template_ids, product_variant_ids)
        warehouse_id = self.env.context.get('warehouse', False)
        if warehouse_id:
            domain += [('order_id.picking_type_id.warehouse_id', '=', warehouse_id)]
        po_lines = self.env['purchase.order.line'].read_group(domain, ['product_uom_qty'], 'product_id')
        in_sum = sum(line['product_uom_qty'] for line in po_lines)

        res['draft_purchase_qty'] = in_sum
        res['qty']['in'] += in_sum
        return res

    def _product_purchase_domain(self, product_template_ids, product_variant_ids):
        if product_variant_ids:
            return [('product_id', 'in', product_variant_ids)]
        elif product_template_ids:
            products = self.env['product.product'].search_read(
                [('product_tmpl_id', 'in', product_template_ids)], ['id']
            )
            product_ids = [product['id'] for product in products]
            return [('product_id', 'in', product_ids)]

```

## File: report\report_stock_forecasted.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="purchase_report_product_product_replenishment" inherit_id="stock.report_product_product_replenishment">
        <xpath expr="//tr[@name='draft_picking_in']" position="after">
            <tr t-if="docs['draft_purchase_qty']" name="draft_po_in">
                <td colspan="2">Draft PO</td>
                <td t-esc="docs['draft_purchase_qty']" class="text-right" t-options='{"widget": "float", "decimal_precision": "Product Unit of Measure"}'/>
                <td t-esc="docs['uom']" groups="uom.group_uom"/>
            </tr>
        </xpath>
    </template>
</odoo>

```

## File: report\report_stock_rule.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class ReportStockRule(models.AbstractModel):
    _inherit = 'report.stock.report_stock_rule'

    @api.model
    def _get_rule_loc(self, rule, product_id):
        """ We override this method to handle buy rules which do not have a location_src_id.
        """
        res = super(ReportStockRule, self)._get_rule_loc(rule, product_id)
        if rule.action == 'buy':
            res['source'] = self.env.ref('stock.stock_location_suppliers')
        return res

```

## File: report\report_stock_rule.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo>
    <template id="purchase_report_stock_rule" inherit_id="stock.report_stock_rule">
        <xpath expr="//div[hasclass('o_report_stock_rule_rule')]/t" position="before">
            <t t-if="rule[0].action == 'buy'">
                <t t-if="rule[1] == 'origin'">
                    <t t-call="stock.report_stock_rule_left_arrow"/>
                </t>
            </t>
        </xpath>
        <xpath expr="//div[hasclass('o_report_stock_rule_rule')]/t[last()]" position="after">
            <t t-if="rule[0].action == 'buy'">
                <t t-if="rule[1] == 'destination'">
                    <t t-call="stock.report_stock_rule_right_arrow"/>
                </t>
            </t>
        </xpath>
        <xpath expr="//div[hasclass('o_report_stock_rule_rule_name')]/span" position="before">
            <t t-if="rule[0].action == 'buy'">
                <i class="fa fa-shopping-cart fa-fw" t-attf-style="color: #{color};"/>
            </t>
        </xpath>
        <xpath expr="//div[hasclass('o_report_stock_rule_legend')]" position="inside">
            <div class="o_report_stock_rule_legend_line">
                <div class="o_report_stock_rule_legend_label">Buy</div>
                <div class="o_report_stock_rule_legend_symbol">
                    <div class="fa fa-shopping-cart fa-fw" t-attf-style="color: #{color};"/>
                </div>
            </div>
        </xpath>
    </template>
</odoo>
```

## File: report\vendor_delay_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, tools
from odoo.exceptions import UserError
from odoo.osv.expression import expression


class VendorDelayReport(models.Model):
    _name = "vendor.delay.report"
    _description = "Vendor Delay Report"
    _auto = False

    partner_id = fields.Many2one('res.partner', 'Vendor', readonly=True)
    product_id = fields.Many2one('product.product', 'Product', readonly=True)
    category_id = fields.Many2one('product.category', 'Product Category', readonly=True)
    date = fields.Datetime('Effective Date', readonly=True)
    qty_total = fields.Float('Total Quantity', readonly=True)
    qty_on_time = fields.Float('On-Time Quantity', readonly=True)
    on_time_rate = fields.Float('On-Time Delivery Rate', readonly=True)

    def init(self):
        tools.drop_view_if_exists(self.env.cr, 'vendor_delay_report')
        self.env.cr.execute("""
CREATE OR replace VIEW vendor_delay_report AS(
SELECT m.id                     AS id,
       m.date                   AS date,
       m.purchase_line_id       AS purchase_line_id,
       m.product_id             AS product_id,
       Min(pc.id)               AS category_id,
       Min(po.partner_id)       AS partner_id,
       Min(m.product_qty)       AS qty_total,
       Sum(CASE
             WHEN (m.state = 'done' and pol.date_planned::date >= m.date::date) THEN (ml.qty_done / ml_uom.factor * pt_uom.factor)
             ELSE 0
           END)                 AS qty_on_time
FROM   stock_move m
       JOIN purchase_order_line pol
         ON pol.id = m.purchase_line_id
       JOIN purchase_order po
         ON po.id = pol.order_id
       JOIN product_product p
         ON p.id = m.product_id
       JOIN product_template pt
         ON pt.id = p.product_tmpl_id
       JOIN uom_uom pt_uom
         ON pt_uom.id = pt.uom_id
       JOIN product_category pc
         ON pc.id = pt.categ_id
       LEFT JOIN stock_move_line ml
         ON ml.move_id = m.id
       LEFT JOIN uom_uom ml_uom
         ON ml_uom.id = ml.product_uom_id
GROUP  BY m.id
)""")

    @api.model
    def read_group(self, domain, fields, groupby, offset=0, limit=None, orderby=False, lazy=True):
        if all('on_time_rate' not in field for field in fields):
            res = super().read_group(domain, fields, groupby, offset=offset, limit=limit, orderby=orderby, lazy=lazy)
            return res

        for field in fields:
            if 'on_time_rate' not in field:
                continue

            fields.remove(field)

            agg = field.split(':')[1:]
            if agg and agg[0] != 'sum':
                raise NotImplementedError('Aggregate functions other than \':sum\' are not allowed.')

            qty_total = field.replace('on_time_rate', 'qty_total')
            if qty_total not in fields:
                fields.append(qty_total)
            qty_on_time = field.replace('on_time_rate', 'qty_on_time')
            if qty_on_time not in fields:
                fields.append(qty_on_time)
            break

        res = super().read_group(domain, fields, groupby, offset=offset, limit=limit, orderby=orderby, lazy=lazy)

        for group in res:
            if group['qty_total'] == 0:
                on_time_rate = 100
            else:
                on_time_rate = group['qty_on_time'] / group['qty_total'] * 100
            group.update({'on_time_rate': on_time_rate})

        return res

```

## File: report\vendor_delay_report.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="vendor_delay_report_filter" model="ir.ui.view">
        <field name="name">vendor.delay.report.search</field>
        <field name="model">vendor.delay.report</field>
        <field name="arch" type="xml">
            <search string="On-time Delivery">
                <field name="partner_id"/>
                <field name="product_id"/>
                <filter string="Effective Date Last Year" name="later_than_a_year_ago"  domain="[('date', '&gt;=', ((context_today()-relativedelta(years=1)).strftime('%Y-%m-%d')))]"/>
            </search>
            </field>
    </record>

    <record id="vendor_delay_report_view_graph" model="ir.ui.view">
        <field name="name">vendor.delay.report.view.graph</field>
        <field name="model">vendor.delay.report</field>
        <field name="arch" type="xml">
            <graph string="On-Time Delivery" type="bar" sample="1" disable_linking="1">
                <field name="product_id" type="row"/>
                <field name="on_time_rate" type="measure"/>
            </graph>
        </field>
    </record>

    <record id="action_purchase_vendor_delay_report" model="ir.actions.act_window">
        <field name="name">On-time Delivery</field>
        <field name="res_model">vendor.delay.report</field>
        <field name="view_mode">graph</field>
        <field name="search_view_id" ref="vendor_delay_report_filter"/>
        <field name="help">Vendor On-time Delivery analysis</field>
        <field name="target">current</field>
        <field name="context">{'search_default_later_than_a_year_ago':1}</field>
    </record>
</odoo>

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import purchase_report
from . import report_stock_forecasted
from . import report_stock_rule
from . import vendor_delay_report

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_purchase_order_stock_worker,purchase.order,model_purchase_order,stock.group_stock_user,1,0,0,0
access_purchase_order_line_stock_worker,purchase.order.line,model_purchase_order_line,stock.group_stock_user,1,0,0,0
access_stock_location_purchase_user,stock.location,stock.model_stock_location,purchase.group_purchase_user,1,0,0,0
access_stock_warehouse_purchase_user,stock.warehouse,stock.model_stock_warehouse,purchase.group_purchase_user,1,0,0,0
access_stock_picking_purchase_user,stock.picking,stock.model_stock_picking,purchase.group_purchase_user,1,1,1,1
access_stock_move_purchase_user,stock.move,stock.model_stock_move,purchase.group_purchase_user,1,1,1,0
access_stock_location_purchase_user_manager,stock.location,stock.model_stock_location,purchase.group_purchase_manager,1,0,0,0
access_stock_warehouse_purchase_user_manager,stock.warehouse,stock.model_stock_warehouse,purchase.group_purchase_manager,1,0,0,0
access_stock_picking_purchase_user_manager,stock.picking,stock.model_stock_picking,purchase.group_purchase_manager,1,1,1,1
access_stock_move_purchase_user_manager,stock.move,stock.model_stock_move,purchase.group_purchase_manager,1,1,1,1
access_stock_warehouse_orderpoint_manager,stock.warehouse.orderpoint,stock.model_stock_warehouse_orderpoint,purchase.group_purchase_manager,1,0,0,0
access_stock_warehouse_orderpoint_user,stock.warehouse.orderpoint,stock.model_stock_warehouse_orderpoint,purchase.group_purchase_user,1,0,0,0
access_report_purchase_order,vendor.delay.report,model_vendor_delay_report,purchase.group_purchase_manager,1,0,0,0
access_report_purchase_order_user,vendor.delay.report user,model_vendor_delay_report,purchase.group_purchase_user,1,0,0,0

```

## File: static\src\js\tours\purchase_stock.js

```javascript
odoo.define('purchase_stock.purchase_steps', function (require) {
"use strict";

var core = require('web.core');

var _t = core._t;
var PurchaseAdditionalTourSteps = require('purchase.purchase_steps');

PurchaseAdditionalTourSteps.include({

    init: function() {
        this._super.apply(this, arguments);
    },

    _get_purchase_stock_steps: function () {
        this._super.apply(this, arguments);
        return [{
            trigger: ".oe_button_box button[name='action_view_picking']",
            extra_trigger: ".oe_button_box button[name='action_view_picking']",
            content: _t("Receive the ordered products."),
            position: "bottom",
            run: 'click',
        }, {
            trigger: ".o_statusbar_buttons button[name='button_validate']",
            content: _t("Validate the receipt of all ordered products."),
            position: "bottom",
            run: 'click',
        }, {
            trigger: ".modal-footer .btn-primary",
            extra_trigger: ".modal-dialog",
            content: _t("Process all the receipt quantities."),
            position: "bottom",
        }, {
            trigger: ".o_back_button a, .breadcrumb-item:not('.active'):last",
            content: _t('Go back to the purchase order to generate the vendor bill.'),
            position: 'bottom',
        }, {
            trigger: ".o_statusbar_buttons button[name='action_create_invoice']",
            content: _t("Generate the draft vendor bill."),
            position: "bottom",
            run: 'click',
        }
        ];
    }
});

return PurchaseAdditionalTourSteps;

});

```

## File: views\assets.xml

```xml
<?xml version="1.0"?>
<odoo>
    <template id="assets_backend" name="purchase stock assets" inherit_id="web.assets_backend">
        <xpath expr="script[last()]" position="after">
            <script type="text/javascript" src="/purchase_stock/static/src/js/tours/purchase_stock.js"></script>
        </xpath>
    </template>
</odoo>

```

## File: views\product_category_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="product_category_view_form" model="ir.ui.view">
        <field name="name">product.category.view.form.inherit.purchase.stock</field>
        <field name="model">product.category</field>
        <field name="inherit_id" ref="purchase.view_category_property_form"/>
        <field name="arch" type="xml">
            <field name="property_account_creditor_price_difference_categ" position="attributes">
                <attribute name="attrs">{'invisible':[('property_valuation', '=', 'manual_periodic')]}</attribute>
            </field>
        </field>
    </record>
</odoo>

```

## File: views\purchase_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="purchase_order_view_form_inherit" model="ir.ui.view">
        <field name="name">purchase.order.form.inherit</field>
        <field name="model">purchase.order</field>
        <field name="inherit_id" ref="purchase.purchase_order_form"/>
        <field name="arch" type="xml">
            <xpath expr="//header/button[@name='action_rfq_send']" position="after">
                <button name="action_view_picking" string="Receive Products" class="oe_highlight" type="object" attrs="{'invisible': ['|', '|' , ('is_shipped', '=', True), ('state','not in', ('purchase','done')), ('picking_count', '=', 0)]}"/>
            </xpath>
            <xpath expr="//header/button[@name='confirm_reminder_mail']" position="attributes">
                <attribute name="attrs">{'invisible': ['|', '|', '|', ('state', 'not in', ('purchase', 'done')), ('mail_reminder_confirmed', '=', True), ('date_planned', '=', False), ('effective_date', '!=', False)]}</attribute>
            </xpath>
            <xpath expr="//div[hasclass('oe_button_box')]" position="inside">
                <button type="object"
                    name="action_view_picking"
                    class="oe_stat_button"
                    icon="fa-truck" attrs="{'invisible':[('picking_ids','=',[])]}">
                    <field name="picking_count" widget="statinfo" string="Receipt" help="Incoming Shipments"/>
                    <field name="picking_ids" invisible="1"/>
                </button>
            </xpath>
            <xpath expr="//field[@name='currency_id']" position="after">
                <field name="is_shipped" invisible="1"/>
            </xpath>
            <xpath expr="//field[@name='order_line']/tree//field[@name='date_planned']" position="after">
                <field name="move_dest_ids" invisible="1"/>
            </xpath>
            <xpath expr="//div[@name='date_planned_div']" position="inside">
                <button name="%(action_purchase_vendor_delay_report)d" class="oe_link" type="action" context="{'search_default_partner_id': partner_id}" attrs="{'invisible': ['|', ('state', 'in', ['purchase', 'done']), ('partner_id', '=', False)]}">
                    <span attrs="{'invisible': [('on_time_rate', '&lt;', 0)]}"><field name="on_time_rate" widget="integer" class="oe_inline"/>% On-Time Delivery</span>
                    <span attrs="{'invisible': [('on_time_rate', '&gt;=', 0)]}">No On-time Delivery Data</span>
                </button>
            </xpath>
            <xpath expr="//div[@name='reminder']" position="attributes">
                <attribute name="attrs">{'invisible': [('effective_date', '!=', False)]}</attribute>
            </xpath>
            <xpath expr="//div[@name='reminder']" position="after">
                <field name="effective_date" attrs="{'invisible': [('effective_date', '=', False)]}"/>
            </xpath>
            <xpath expr="//field[@name='order_line']/form//field[@name='invoice_lines']" position="after">
                <field name="move_ids"/>
            </xpath>
            <xpath expr="//field[@name='order_line']/form//field[@name='account_analytic_id']" position="before">
                <field name="propagate_cancel" groups="base.group_no_one"/>
            </xpath>
            <xpath expr="//field[@name='order_line']/tree//field[@name='qty_received']" position="attributes">
                <attribute name="attrs">{'column_invisible': [('parent.state', 'not in', ('purchase', 'done'))], 'readonly': [('product_type', 'in', ('consu', 'product'))]}</attribute>
            </xpath>
            <xpath expr="//page[@name='purchase_delivery_invoice']/group/group" position="inside">
                <field name="picking_type_id" domain="[('code','=','incoming'), '|', ('warehouse_id', '=', False), ('warehouse_id.company_id', '=', company_id)]" options="{'no_create': True}" groups="stock.group_stock_multi_locations"/>
                <field name="dest_address_id" groups="stock.group_stock_multi_locations" attrs="{'invisible': [('default_location_dest_id_usage', '!=', 'customer')], 'required': [('default_location_dest_id_usage', '=', 'customer')]}"/>
                <field name="default_location_dest_id_usage" invisible="1"/>
                <field name="incoterm_id"/>
            </xpath>
        </field>
    </record>

    <record id="purchase_order_line_view_form_inherit" model="ir.ui.view">
        <field name="name">purchase.order.line.form.inherit</field>
        <field name="model">purchase.order.line</field>
        <field name="inherit_id" ref="purchase.purchase_order_line_form2"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='invoice_lines']" position="after">
                <separator string="Stock Moves"/>
                <field name="move_ids"/>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

	<record id="res_config_settings_view_form_purchase" model="ir.ui.view">
		<field name="name">res.config.settings.view.form.inherit.purchase</field>
		<field name="model">res.config.settings</field>
		<field name="priority" eval="25"/>
		<field name="inherit_id" ref="purchase.res_config_settings_view_form_purchase"/>
		<field name="arch" type="xml">
			<xpath expr="//div[@data-key='purchase']" position="inside">
				<field name="is_installed_sale" invisible="1"/>
				<h2 attrs="{'invisible': [('is_installed_sale', '=', False)]}">Logistics</h2>
				<div class="row mt16 o_settings_container" name="request_vendor_setting_container">
					<div class="col-12 col-lg-6 o_setting_box" attrs="{'invisible': [('is_installed_sale', '=', False)]}" title="This adds a dropshipping route to apply on products in order to request your vendors to deliver to your customers. A product to dropship will generate a purchase request for quotation once the sales order confirmed. This is a on-demand flow. The requested delivery address will be the customer delivery address and not your warehouse.">
						<div class="o_setting_left_pane">
							<field name="module_stock_dropshipping"/>
						</div>
						<div class="o_setting_right_pane">
							<label for="module_stock_dropshipping"/>
							<a href="https://www.odoo.com/documentation/14.0/applications/inventory_and_mrp/inventory/management/delivery/dropshipping.html" title="Documentation" class="o_doc_link" target="_blank"></a>
							<div class="text-muted">
								Request your vendors to deliver to your customers
							</div>
						</div>
					</div>
				</div>
			</xpath>
		</field>
	</record>

	<record id="res_config_settings_view_form_stock" model="ir.ui.view">
		<field name="name">res.config.settings.view.form.inherit.purchase.stock</field>
		<field name="model">res.config.settings</field>
		<field name="inherit_id" ref="stock.res_config_settings_view_form"/>
		<field name="arch" type="xml">
			<xpath expr="//h2[@id='schedule_info']" position="attributes">
				<attribute name="invisible">0</attribute>
			</xpath>
			<div id="purchase_po_lead" position="replace">
				<div class="col-12 col-lg-6 o_setting_box"
                                    title="Margin of error for vendor lead times. When the system generates Purchase Orders for reordering products,they will be scheduled that many days earlier to cope with unexpected vendor delays."
                                    name="schedule_receivings_setting_container">
					<div class="o_setting_left_pane">
						<field name="use_po_lead"/>
					</div>
					<div class="o_setting_right_pane">
						<label for="use_po_lead"/>
						<a href="https://www.odoo.com/documentation/14.0/applications/inventory_and_mrp/inventory/management/planning/scheduled_dates.html" title="Documentation" class="mr-2 o_doc_link" target="_blank"></a>
						<span class="fa fa-lg fa-building-o" title="Values set here are company-specific." groups="base.group_multi_company"/>
						<div class="text-muted">
							Schedule receivings earlier to avoid delays
						</div>
						<div class="content-group">
							<div class="mt16" attrs="{'invisible': [('use_po_lead','=',False)]}">
								<span>Move forward expected delivery dates by <field name="po_lead" class="oe_inline"/> days</span>
							</div>
						</div>
					</div>
				</div>
				<div class="col-12 col-lg-6 o_setting_box">
					<div class="o_setting_right_pane">
						<label for="days_to_purchase"/>
						<span class="fa fa-lg fa-building-o" title="Values set here are company-specific." aria-label="Values set here are company-specific." groups="base.group_multi_company"/>
						<div class="text-muted">
							Days needed to confirm a PO, define when a PO should be validated
						</div>
						<div class="content-group">
							<div class="mt16">
								<span><field name="days_to_purchase" class="oe_inline"/> days</span>
							</div>
						</div>
					</div>
				</div>
			</div>
		</field>
	</record>

</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="res_partner_view_purchase_buttons_inherit" model="ir.ui.view">
            <field name="name">res.partner.purchase.stock.form.inherit</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="purchase.res_partner_view_purchase_buttons"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='purchase_order_count']/.." position="after">
                    <button class="oe_stat_button" name="%(action_purchase_vendor_delay_report)d" type="action"
                        groups="purchase.group_purchase_user"
                        icon="fa-truck"
                        context="{'search_default_partner_id': id}">
                        <div class="o_form_field o_stat_info">
                            <div class="o_row" attrs="{'invisible': [('on_time_rate', '&lt;', 0)]}">
                                <span class="o_stat_value">
                                    <field string="On-time Rate" name="on_time_rate" widget="integer"/>
                                </span>
                                <span class="o_stat_value">%</span>
                            </div>
                            <div class="o_stat_value" attrs="{'invisible': [('on_time_rate', '&gt;=', 0)]}">
                                No data yet
                            </div>
                            <span class="o_stat_text">On-time Rate</span>
                        </div>
                    </button>
                </xpath>
            </field>
    </record>
</odoo>

```

## File: views\stock_production_lot_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="stock_production_lot_view_form" model="ir.ui.view">
        <field name="name">stock.production.lot.view.form</field>
        <field name="model">stock.production.lot</field>
        <field name="inherit_id" ref="stock.view_production_lot_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[hasclass('oe_button_box')]/button" position="before">
                <button class="oe_stat_button" name="action_view_po"
                        type="object" icon="fa-shopping-cart" help="Purchase Orders"
                        attrs="{'invisible': ['|', ('purchase_order_count', '=', 0), ('display_complete', '=', False)]}">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_value">
                            <field name="purchase_order_count" widget="statinfo" nolabel="1" class="mr4"/>
                        </span>
                        <span class="o_stat_text">Purchases</span>
                    </div>
                </button>
            </xpath>
            <xpath expr="//group[@name='main_group']" position="after">
                <group>
                    <field name="purchase_order_ids" widget="many2many" readonly="True"
                           attrs="{'invisible': ['|', ('purchase_order_ids', '=', []), ('display_complete', '=', True)]}">
                        <tree>
                            <field name="name"/>
                            <field name="partner_id"/>
                            <field name="date_order"/>
                            <field name="state" invisible="1"/>
                        </tree>
                    </field>
                </group>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\stock_rule_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <!-- Stock rules -->
        <record id="view_stock_rule_form_stock_inherit_purchase_stock" model="ir.ui.view">
            <field name="name">stock.rule.form.stock.inherit.purchase_stock</field>
            <field name="model">stock.rule</field>
            <field name="inherit_id" ref="stock.view_stock_rule_form"/>
            <field name="arch" type="xml">
                <field name="location_src_id" position="attributes">
                    <attribute name="attrs">{'required': [('action', 'in', ['pull', 'push', 'pull_push'])], 'invisible': [('action', '=', 'buy')]}</attribute>
                </field>
            </field>
        </record>

    </data>
</odoo>

```

## File: views\stock_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="stock_move_purchase" model="ir.ui.view">
            <field name="name">stock.move.form</field>
            <field name="model">stock.move</field>
            <field name="inherit_id" ref="stock.view_move_form"/>
            <field name="arch" type="xml">
                <xpath expr="//group[@name='origin_grp']" position="inside">
                    <field name="purchase_line_id"/>
                </xpath>
            </field>
        </record>

        <record id="view_warehouse_inherited" model="ir.ui.view">
            <field name="name">Stock Warehouse Inherited</field>
            <field name="model">stock.warehouse</field>
            <field name="inherit_id" ref="stock.view_warehouse"/>
            <field name="arch" type="xml">
                 <xpath expr="//field[@name='resupply_wh_ids']" position="before">
                    <field name="buy_to_resupply" />
                </xpath>
                <xpath expr="//group[@name='group_resupply']" position="attributes">
                    <attribute name="attrs">{}</attribute>
                </xpath>
            </field>
        </record>

        <record id="view_warehouse_orderpoint_tree_editable_inherited_mrp" model="ir.ui.view">
            <field name="name">stock.warehouse.orderpoint.tree.editable.inherit.mrp</field>
            <field name="model">stock.warehouse.orderpoint</field>
            <field name="inherit_id" ref="stock.view_warehouse_orderpoint_tree_editable"/>
            <field name="arch" type="xml">
                <field name="route_id" position="after">
                    <field name="show_supplier" invisible="1"/>
                    <field name="supplier_id" optional="hide" attrs="{'invisible': [('show_supplier', '=', False)]}" options="{'no_create': True}"/>
                </field>
            </field>
        </record>

</odoo>

```

