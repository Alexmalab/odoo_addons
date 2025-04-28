# Odoo Module: pos_sale

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import report


def _pos_sale_post_init(env):
    env['pos.config']._ensure_downpayment_product()

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


{
    'name': 'POS - Sales',
    'version': '1.1',
    'category': 'Hidden',
    'sequence': 6,
    'summary': 'Link module between Point of Sale and Sales',
    'description': """

This module adds a custom Sales Team for the Point of Sale. This enables you to view and manage your point of sale sales with more ease.
""",
    'depends': ['point_of_sale', 'sale_management'],
    'data': [
        'data/pos_sale_data.xml',
        'security/pos_sale_security.xml',
        'security/ir.model.access.csv',
        'views/point_of_sale_report.xml',
        'views/sale_order_views.xml',
        'views/pos_order_views.xml',
        'views/product_views.xml',
        'views/sales_team_views.xml',
        'views/res_config_settings_views.xml',
        'views/stock_template.xml',
    ],
    'installable': True,
    'auto_install': True,
    'assets': {
        'point_of_sale._assets_pos': [
            'pos_sale/static/src/**/*',
        ],
        'web.assets_tests': [
            'pos_sale/static/tests/tours/**/*',
        ],
    },
    'post_init_hook': '_pos_sale_post_init',
    'license': 'LGPL-3',
}

```

## File: data\pos_sale_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="sales_team.pos_sales_team" model="crm.team">
            <field name="active" eval="True"/>
        </record>

        <record id="pos_sale.default_downpayment_product" model="product.product">
            <field name="name">Down Payment (POS)</field>
            <field name="available_in_pos">False</field>
            <field name="standard_price">0.00</field>
            <field name="list_price">0.00</field>
            <field name="weight">0.00</field>
            <field name="type">service</field>
            <field name="taxes_id" eval="[(5,)]"/>
            <field name="categ_id" ref="point_of_sale.product_category_pos"/>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="purchase_ok">False</field>
        </record>
    </data>
</odoo>

```

## File: models\crm_team.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError, ValidationError
from datetime import datetime
import pytz


class CrmTeam(models.Model):
    _inherit = 'crm.team'

    pos_config_ids = fields.One2many('pos.config', 'crm_team_id', string="Point of Sales")
    pos_sessions_open_count = fields.Integer(string='Open POS Sessions', compute='_compute_pos_sessions_open_count')
    pos_order_amount_total = fields.Float(string="Session Sale Amount", compute='_compute_pos_order_amount_total')

    def _compute_pos_sessions_open_count(self):
        for team in self:
            team.pos_sessions_open_count = self.env['pos.session'].search_count([('config_id.crm_team_id', '=', team.id), ('state', '=', 'opened')])

    def _compute_pos_order_amount_total(self):
        data = self.env['report.pos.order']._read_group([
            ('session_id.state', '=', 'opened'),
            ('config_id.crm_team_id', 'in', self.ids),
        ], ['config_id'], ['price_total:sum'])
        rg_results = {config.id: price_total_sum for config, price_total_sum in data}
        for team in self:
            team.pos_order_amount_total = sum([
                rg_results.get(config.id, 0.0)
                for config in team.pos_config_ids
            ])

```

## File: models\pos_config.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api


class PosConfig(models.Model):
    _inherit = 'pos.config'

    crm_team_id = fields.Many2one(
        'crm.team', string="Sales Team", ondelete="set null",
        help="This Point of sale's sales will be related to this Sales Team.")
    down_payment_product_id = fields.Many2one('product.product',
        string="Down Payment Product",
        help="This product will be used as down payment on a sale order.")

    def _get_special_products(self):
        res = super()._get_special_products()
        return res | self.env['pos.config'].search([]).mapped('down_payment_product_id')

    @api.model
    def _ensure_downpayment_product(self):
        pos_config = self.env.ref('point_of_sale.pos_config_main', raise_if_not_found=False)
        downpayment_product = self.env.ref('pos_sale.default_downpayment_product', raise_if_not_found=False)
        if pos_config and downpayment_product:
            pos_config.write({'down_payment_product_id': downpayment_product.id})

    @api.model
    def load_onboarding_furniture_scenario(self):
        super().load_onboarding_furniture_scenario()
        self._ensure_downpayment_product()

```

## File: models\pos_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.tools import float_compare, float_is_zero, format_date


class PosOrder(models.Model):
    _inherit = 'pos.order'

    currency_rate = fields.Float(compute='_compute_currency_rate', store=True, digits=0, readonly=True)
    crm_team_id = fields.Many2one('crm.team', string="Sales Team", ondelete="set null")
    sale_order_count = fields.Integer(string='Sale Order Count', compute='_count_sale_order', readonly=True, groups="sales_team.group_sale_salesman")

    def _count_sale_order(self):
        for order in self:
            order.sale_order_count = len(order.lines.mapped('sale_order_origin_id'))

    @api.model
    def _complete_values_from_session(self, session, values):
        values = super(PosOrder, self)._complete_values_from_session(session, values)
        values['crm_team_id'] = values['crm_team_id'] if values.get('crm_team_id') else session.config_id.crm_team_id.id
        return values

    @api.depends('date_order', 'company_id')
    def _compute_currency_rate(self):
        for order in self:
            date_order = order.date_order or fields.Datetime.now()
            order.currency_rate = self.env['res.currency']._get_conversion_rate(order.company_id.currency_id, order.currency_id, order.company_id, date_order.date())

    def _prepare_invoice_vals(self):
        invoice_vals = super(PosOrder, self)._prepare_invoice_vals()
        invoice_vals['team_id'] = self.crm_team_id.id
        sale_orders = self.lines.mapped('sale_order_origin_id')
        if sale_orders:
            if sale_orders[0].partner_invoice_id.id != sale_orders[0].partner_shipping_id.id:
                invoice_vals['partner_shipping_id'] = sale_orders[0].partner_shipping_id.id
            else:
                addr = self.partner_id.address_get(['delivery'])
                invoice_vals['partner_shipping_id'] = addr['delivery']
            if sale_orders[0].payment_term_id:
                invoice_vals['invoice_payment_term_id'] = False
            if sale_orders[0].partner_invoice_id != sale_orders[0].partner_id:
                invoice_vals['partner_id'] = sale_orders[0].partner_invoice_id.id
        return invoice_vals

    @api.model
    def sync_from_ui(self, orders):
        data = super().sync_from_ui(orders)
        if len(orders) == 0:
            return data

        order_ids = self.browse([o['id'] for o in data["pos.order"]])
        for order in order_ids:
            for line in order.lines.filtered(lambda l: l.product_id == order.config_id.down_payment_product_id and l.qty != 0 and (l.sale_order_origin_id or l.refunded_orderline_id.sale_order_origin_id)):
                sale_lines = line.sale_order_origin_id.order_line or line.refunded_orderline_id.sale_order_origin_id.order_line
                sale_order_origin = line.sale_order_origin_id or line.refunded_orderline_id.sale_order_origin_id
                if not any(line.display_type and line.is_downpayment for line in sale_lines):
                    self.env['sale.order.line'].create(
                        self.env['sale.advance.payment.inv']._prepare_down_payment_section_values(sale_order_origin)
                    )
                order_reference = line.name

                if order.partner_id.lang and order.partner_id.lang != line.env.lang:
                    line = line.with_context(lang=order.partner_id.lang)

                sale_order_line_description = _("Down payment (ref: %(order_reference)s on \n %(date)s)", order_reference=order_reference, date=format_date(line.env, line.order_id.date_order))
                sale_line = self.env['sale.order.line'].create({
                    'order_id': sale_order_origin.id,
                    'product_id': line.product_id.id,
                    'price_unit': line.price_unit,
                    'product_uom_qty': 0,
                    'tax_id': [(6, 0, line.tax_ids.ids)],
                    'is_downpayment': True,
                    'discount': line.discount,
                    'sequence': sale_lines and sale_lines[-1].sequence + 2 or 10,
                    'name': sale_order_line_description
                })
                line.sale_order_line_id = sale_line

            so_lines = order.lines.mapped('sale_order_line_id')

            if order.state != 'draft':
                # confirm the unconfirmed sale orders that are linked to the sale order lines
                sale_orders = so_lines.mapped('order_id')
                for sale_order in sale_orders.filtered(lambda so: so.state in ['draft', 'sent']):
                    sale_order.action_confirm()

            # update the demand qty in the stock moves related to the sale order line
            # flush the qty_delivered to make sure the updated qty_delivered is used when
            # updating the demand value
            so_lines.flush_recordset(['qty_delivered'])
            # track the waiting pickings
            waiting_picking_ids = set()
            for so_line in so_lines:
                so_line_stock_move_ids = so_line.move_ids.group_id.stock_move_ids
                for stock_move in so_line.move_ids:
                    picking = stock_move.picking_id
                    if not picking.state in ['waiting', 'confirmed', 'assigned']:
                        continue
                    new_qty = so_line.product_uom_qty - so_line.qty_delivered
                    if float_compare(new_qty, 0, precision_rounding=stock_move.product_uom.rounding) <= 0:
                        new_qty = 0
                    stock_move.product_uom_qty = so_line.compute_uom_qty(new_qty, stock_move, False)
                    # If the product is delivered with more than one step, we need to update the quantity of the other steps
                    for move in so_line_stock_move_ids.filtered(lambda m: m.state in ['waiting', 'confirmed', 'assigned'] and m.product_id == stock_move.product_id):
                        move.product_uom_qty = stock_move.product_uom_qty
                        waiting_picking_ids.add(move.picking_id.id)
                    waiting_picking_ids.add(picking.id)

            def is_product_uom_qty_zero(move):
                return float_is_zero(move.product_uom_qty, precision_rounding=move.product_uom.rounding)

            # cancel the waiting pickings if each product_uom_qty of move is zero
            for picking in self.env['stock.picking'].browse(waiting_picking_ids):
                if all(is_product_uom_qty_zero(move) for move in picking.move_ids):
                    picking.action_cancel()
                else:
                    # We make sure that the original picking still has the correct quantity reserved
                    picking.action_assign()

        return data

    def action_view_sale_order(self):
        self.ensure_one()
        linked_orders = self.lines.mapped('sale_order_origin_id')
        return {
            'type': 'ir.actions.act_window',
            'name': _('Linked Sale Orders'),
            'res_model': 'sale.order',
            'view_mode': 'list,form',
            'domain': [('id', 'in', linked_orders.ids)],
        }

    def _get_fields_for_order_line(self):
        fields = super(PosOrder, self)._get_fields_for_order_line()
        fields.extend([
            'sale_order_origin_id',
            'down_payment_details',
            'sale_order_line_id',
        ])
        return fields

    def _prepare_order_line(self, order_line):
        order_line = super()._prepare_order_line(order_line)
        if order_line.get('sale_order_origin_id'):
            order_line['sale_order_origin_id'] = {
                'id': order_line['sale_order_origin_id'][0],
                'name': order_line['sale_order_origin_id'][1],
            }
        if order_line.get('sale_order_line_id'):
            order_line['sale_order_line_id'] = {
                'id': order_line['sale_order_line_id'][0],
            }
        return order_line

    def _get_invoice_lines_values(self, line_values, pos_line):
        inv_line_vals = super()._get_invoice_lines_values(line_values, pos_line)

        if pos_line.sale_order_origin_id:
            origin_line = pos_line.sale_order_line_id
            inv_line_vals["name"] = origin_line.name
            origin_line._set_analytic_distribution(inv_line_vals)

        return inv_line_vals

    def write(self, vals):
        if 'crm_team_id' in vals:
            vals['crm_team_id'] = vals['crm_team_id'] if vals.get('crm_team_id') else self.session_id.crm_team_id.id
        return super().write(vals)


class PosOrderLine(models.Model):
    _inherit = 'pos.order.line'

    sale_order_origin_id = fields.Many2one('sale.order', string="Linked Sale Order")
    sale_order_line_id = fields.Many2one('sale.order.line', string="Source Sale Order Line")
    down_payment_details = fields.Text(string="Down Payment Details")
    qty_delivered = fields.Float(
        string="Delivery Quantity",
        compute="_compute_qty_delivered",
        store=True, readonly=False, copy=False)

    @api.depends('order_id.state', 'order_id.picking_ids', 'order_id.picking_ids.state', 'order_id.picking_ids.move_ids.quantity')
    def _compute_qty_delivered(self):
        for order_line in self:
            if order_line.order_id.state in ['paid', 'done', 'invoiced']:
                outgoing_pickings = order_line.order_id.picking_ids.filtered(
                    lambda pick: pick.state == 'done' and pick.picking_type_code == 'outgoing'
                )

                if outgoing_pickings:
                    moves = outgoing_pickings.move_ids.filtered(
                        lambda m: m.state == 'done' and m.product_id == order_line.product_id
                    )
                    order_line.qty_delivered = sum(moves.mapped('quantity'))
                else:
                    order_line.qty_delivered = 0

    @api.model
    def _load_pos_data_fields(self, config_id):
        params = super()._load_pos_data_fields(config_id)
        params += ['sale_order_origin_id', 'sale_order_line_id', 'down_payment_details']
        return params

    def _launch_stock_rule_from_pos_order_lines(self):
        orders = self.mapped('order_id')
        for order in orders:
            self.env['stock.move'].browse(order.lines.sale_order_line_id.move_ids._rollup_move_origs()).filtered(lambda ml: ml.state not in ['cancel', 'done'])._action_cancel()
        return super()._launch_stock_rule_from_pos_order_lines()

```

## File: models\pos_session.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api


class PosSession(models.Model):
    _inherit = 'pos.session'

    crm_team_id = fields.Many2one('crm.team', related='config_id.crm_team_id', string="Sales Team", readonly=True)

    @api.model
    def _load_pos_data_models(self, config_id):
        data = super()._load_pos_data_models(config_id)
        data += ['sale.order', 'sale.order.line']
        return data

```

## File: models\product_product.py

```python
from odoo import models, api


class ProductProduct(models.Model):
    _inherit = 'product.product'

    @api.model
    def _load_pos_data_fields(self, config_id):
        params = super()._load_pos_data_fields(config_id)
        params += ['invoice_policy', 'optional_product_ids', 'type']
        return params

    def get_product_info_pos(self, price, quantity, pos_config_id):
        res = super().get_product_info_pos(price, quantity, pos_config_id)

        # Optional products
        res['optional_products'] = [
            {'name': p.name, 'price': min(p.product_variant_ids.mapped('lst_price'))}
            for p in self.optional_product_ids.filtered_domain(self._optional_product_pos_domain())
        ]

        return res

    def has_optional_product_in_pos(self):
        self.ensure_one()
        return bool(self.optional_product_ids.filtered_domain(self._optional_product_pos_domain()))

    def _optional_product_pos_domain(self):
        return [
            *self.env['product.product']._check_company_domain(self.env.company),
            ['sale_ok', '=', True],
            ['available_in_pos', '=', True],
        ]

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    pos_crm_team_id = fields.Many2one(related='pos_config_id.crm_team_id', readonly=False, string='Sales Team (PoS)')
    pos_down_payment_product_id = fields.Many2one(related='pos_config_id.down_payment_product_id', readonly=False)

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _


class SaleOrder(models.Model):
    _name = 'sale.order'
    _inherit = ['sale.order', 'pos.load.mixin']

    pos_order_line_ids = fields.One2many('pos.order.line', 'sale_order_origin_id', string="Order lines Transfered to Point of Sale", readonly=True, groups="point_of_sale.group_pos_user")
    pos_order_count = fields.Integer(string='Pos Order Count', compute='_count_pos_order', readonly=True, groups="point_of_sale.group_pos_user")
    amount_unpaid = fields.Monetary(
        string="Amount To Pay In POS",
        help="Amount left to pay in POS to avoid double payment or double invoicing.",
        compute='_compute_amount_unpaid',
        store=True,
    )

    @api.model
    def _load_pos_data_domain(self, data):
        return [['pos_order_line_ids.order_id.state', '=', 'draft']]

    @api.model
    def _load_pos_data_fields(self, config_id):
        return ['name', 'state', 'user_id', 'order_line', 'partner_id', 'pricelist_id', 'fiscal_position_id', 'amount_total', 'amount_untaxed', 'amount_unpaid',
            'picking_ids', 'partner_shipping_id', 'partner_invoice_id', 'date_order']

    def _count_pos_order(self):
        for order in self:
            linked_orders = order.pos_order_line_ids.mapped('order_id')
            order.pos_order_count = len(linked_orders)

    def action_view_pos_order(self):
        self.ensure_one()
        linked_orders = self.pos_order_line_ids.mapped('order_id')
        return {
            'type': 'ir.actions.act_window',
            'name': _('Linked POS Orders'),
            'res_model': 'pos.order',
            'view_mode': 'list,form',
            'domain': [('id', 'in', linked_orders.ids)],
        }

    @api.depends('order_line', 'amount_total', 'order_line.invoice_lines.parent_state', 'order_line.invoice_lines.price_total', 'order_line.pos_order_line_ids')
    def _compute_amount_unpaid(self):
        for sale_order in self:
            total_invoice_paid = sum(sale_order.order_line.filtered(lambda l: not l.display_type).mapped('invoice_lines').filtered(lambda l: l.parent_state != 'cancel').mapped('price_total'))
            total_pos_paid = sum(sale_order.order_line.filtered(lambda l: not l.display_type).mapped('pos_order_line_ids.price_subtotal_incl'))
            sale_order.amount_unpaid = sale_order.amount_total - (total_invoice_paid + total_pos_paid)

    @api.depends('order_line.pos_order_line_ids')
    def _compute_amount_to_invoice(self):
        super()._compute_amount_to_invoice()
        for order in self:
            if order.invoice_status == 'invoiced':
                continue
            # We need to account for the downpayment paid in POS with and without invoice
            order_amount = sum(order.sudo().pos_order_line_ids.filtered(lambda pol: pol.sale_order_line_id.is_downpayment).mapped('price_subtotal_incl'))
            order.amount_to_invoice -= order_amount

    @api.depends('order_line.pos_order_line_ids')
    def _compute_amount_invoiced(self):
        super()._compute_amount_invoiced()
        for order in self:
            if order.invoice_status == 'invoiced':
                continue
            # We need to account for the downpayment paid in POS with and without invoice
            order_amount = sum(order.sudo().pos_order_line_ids.filtered(lambda pol: pol.order_id.state in ['paid', 'done', 'invoiced'] and pol.sale_order_line_id.is_downpayment).mapped('price_subtotal_incl'))
            order.amount_invoiced += order_amount


class SaleOrderLine(models.Model):
    _name = 'sale.order.line'
    _inherit = ['sale.order.line', 'pos.load.mixin']

    pos_order_line_ids = fields.One2many('pos.order.line', 'sale_order_line_id', string="Order lines Transfered to Point of Sale", readonly=True, groups="point_of_sale.group_pos_user")

    @api.model
    def _load_pos_data_domain(self, data):
        return [('order_id', 'in', [order['id'] for order in data['sale.order']['data']])]

    @api.model
    def _load_pos_data_fields(self, config_id):
        return ['discount', 'display_name', 'price_total', 'price_unit', 'product_id', 'product_uom_qty', 'qty_delivered',
            'qty_invoiced', 'qty_to_invoice', 'display_type', 'name', 'tax_id', 'is_downpayment']

    @api.depends('pos_order_line_ids.qty', 'pos_order_line_ids.order_id.picking_ids', 'pos_order_line_ids.order_id.picking_ids.state')
    def _compute_qty_delivered(self):
        super()._compute_qty_delivered()
        for sale_line in self:
            pos_lines = sale_line.pos_order_line_ids.filtered(lambda order_line: order_line.order_id.state not in ['cancel', 'draft'])
            if all(picking.state == 'done' for picking in pos_lines.order_id.picking_ids):
                sale_line.qty_delivered += sum((self._convert_qty(sale_line, pos_line.qty, 'p2s') for pos_line in pos_lines if sale_line.product_id.type != 'service'), 0)

    @api.depends('pos_order_line_ids.qty')
    def _compute_qty_invoiced(self):
        super()._compute_qty_invoiced()
        for sale_line in self:
            pos_lines = sale_line.pos_order_line_ids.filtered(lambda order_line: order_line.order_id.state not in ['cancel', 'draft'])
            sale_line.qty_invoiced += sum([self._convert_qty(sale_line, pos_line.qty, 'p2s') for pos_line in pos_lines], 0)

    def _get_sale_order_fields(self):
        return ["product_id", "display_name", "price_unit", "product_uom_qty", "tax_id", "qty_delivered", "qty_invoiced", "discount", "qty_to_invoice", "price_total", "is_downpayment"]

    def read_converted(self):
        field_names = self._get_sale_order_fields()
        results = []
        for sale_line in self:
            if sale_line.product_type or (sale_line.is_downpayment and sale_line.price_unit != 0):
                product_uom = sale_line.product_id.uom_id
                sale_line_uom = sale_line.product_uom
                item = sale_line.read(field_names, load=False)[0]
                if sale_line.product_id.tracking != 'none':
                    item['lot_names'] = sale_line.move_ids.move_line_ids.lot_id.mapped('name')
                if product_uom == sale_line_uom:
                    results.append(item)
                    continue
                item['product_uom_qty'] = self._convert_qty(sale_line, item['product_uom_qty'], 's2p')
                item['qty_delivered'] = self._convert_qty(sale_line, item['qty_delivered'], 's2p')
                item['qty_invoiced'] = self._convert_qty(sale_line, item['qty_invoiced'], 's2p')
                item['qty_to_invoice'] = self._convert_qty(sale_line, item['qty_to_invoice'], 's2p')
                item['price_unit'] = sale_line_uom._compute_price(item['price_unit'], product_uom)
                results.append(item)

            elif sale_line.display_type == 'line_note':
                if results:
                    if results[-1].get('customer_note'):
                        results[-1]['customer_note'] += "--" + sale_line.name
                    else:
                        results[-1]['customer_note'] = sale_line.name


        return results

    @api.model
    def _convert_qty(self, sale_line, qty, direction):
        """Converts the given QTY based on the given SALE_LINE and DIR.

        if DIR='s2p': convert from sale line uom to product uom
        if DIR='p2s': convert from product uom to sale line uom
        """
        product_uom = sale_line.product_id.uom_id
        sale_line_uom = sale_line.product_uom
        if direction == 's2p':
            return sale_line_uom._compute_quantity(qty, product_uom, False)
        elif direction == 'p2s':
            return product_uom._compute_quantity(qty, sale_line_uom, False)

    def unlink(self):
        # do not delete downpayment lines created from pos
        pos_downpayment_lines = self.filtered(lambda line: line.is_downpayment and line.sudo().pos_order_line_ids)
        return super(SaleOrderLine, self - pos_downpayment_lines).unlink()

    @api.depends('pos_order_line_ids')
    def _compute_untaxed_amount_invoiced(self):
        super()._compute_untaxed_amount_invoiced()
        for line in self:
            line.untaxed_amount_invoiced += sum(line.pos_order_line_ids.mapped('price_subtotal'))

    def _get_downpayment_line_price_unit(self, invoices):
        return super()._get_downpayment_line_price_unit(invoices) + sum(
            pol.price_unit for pol in self.sudo().pos_order_line_ids
        )

```

## File: models\stock_picking.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class StockPicking(models.Model):
    _inherit = 'stock.picking'

    def _create_move_from_pos_order_lines(self, lines):
        lines_to_unreserve = self.env['pos.order.line']
        for line in lines:
            if line.order_id.shipping_date:
                continue
            if any(wh != line.order_id.config_id.warehouse_id for wh in line.sale_order_line_id.move_ids.location_id.warehouse_id):
                continue
            lines_to_unreserve |= line
        lines_to_unreserve.sale_order_line_id.move_ids.filtered(lambda ml: ml.state not in ['cancel', 'done'])._do_unreserve()
        return super()._create_move_from_pos_order_lines(lines.filtered(lambda l: not l.sale_order_line_id or (l.sale_order_line_id.has_valued_move_ids() or not l.sale_order_line_id.move_ids)))

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import pos_config
from . import pos_order
from . import product_product
from . import crm_team
from . import pos_session
from . import sale_order
from . import stock_picking
from . import res_config_settings

```

## File: report\sale_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class SaleReport(models.Model):
    _inherit = "sale.report"

    @api.model
    def _get_done_states(self):
        done_states = super()._get_done_states()
        done_states.extend(['paid', 'invoiced', 'done'])
        return done_states

    state = fields.Selection(
        selection_add=[
            ('paid', 'Paid'),
            ('invoiced', 'Invoiced'),
            ('done', 'Posted')
        ],
    )

    order_reference = fields.Reference(selection_add=[('pos.order', 'POS Order')])

    def _select_pos(self):
        select_ = f"""
            -MIN(l.id) AS id,
            l.product_id AS product_id,
            NULL AS line_invoice_status,
            t.uom_id AS product_uom,
            SUM(l.qty) AS product_uom_qty,
            SUM(l.qty_delivered) AS qty_delivered,
            SUM(l.qty - l.qty_delivered) AS qty_to_deliver,
            CASE WHEN pos.state = 'invoiced' THEN SUM(l.qty) ELSE 0 END AS qty_invoiced,
            CASE WHEN pos.state != 'invoiced' THEN SUM(l.qty) ELSE 0 END AS qty_to_invoice,
            AVG(l.price_unit)
                / MIN({self._case_value_or_one('pos.currency_rate')})
                * {self._case_value_or_one('account_currency_table.rate')}
            AS price_unit,
            SUM(l.price_subtotal_incl)
                / MIN({self._case_value_or_one('pos.currency_rate')})
                * {self._case_value_or_one('account_currency_table.rate')}
            AS price_total,
            SUM(l.price_subtotal)
                / MIN({self._case_value_or_one('pos.currency_rate')})
                * {self._case_value_or_one('account_currency_table.rate')}
            AS price_subtotal,
            (CASE WHEN pos.state != 'invoiced' THEN SUM(l.price_subtotal) ELSE 0 END)
                / MIN({self._case_value_or_one('pos.currency_rate')})
                * {self._case_value_or_one('account_currency_table.rate')}
            AS amount_to_invoice,
            (CASE WHEN pos.state = 'invoiced' THEN SUM(l.price_subtotal) ELSE 0 END)
                / MIN({self._case_value_or_one('pos.currency_rate')})
                * {self._case_value_or_one('account_currency_table.rate')}
            AS amount_invoiced,
            count(*) AS nbr,
            pos.name AS name,
            pos.date_order AS date,
            (CASE WHEN pos.state = 'done' THEN 'sale' ELSE pos.state END) AS state,
            NULL as invoice_status,
            pos.partner_id AS partner_id,
            pos.user_id AS user_id,
            pos.company_id AS company_id,
            NULL AS campaign_id,
            NULL AS medium_id,
            NULL AS source_id,
            t.categ_id AS categ_id,
            pos.pricelist_id AS pricelist_id,
            pos.crm_team_id AS team_id,
            p.product_tmpl_id,
            partner.commercial_partner_id AS commercial_partner_id,
            partner.country_id AS country_id,
            partner.industry_id AS industry_id,
            partner.state_id AS state_id,
            partner.zip AS partner_zip,
            (SUM(p.weight) * l.qty) AS weight,
            (SUM(p.volume) * l.qty) AS volume,
            l.discount AS discount,
            SUM((l.price_unit * l.discount * l.qty / 100.0
                / {self._case_value_or_one('pos.currency_rate')}
                * {self._case_value_or_one('account_currency_table.rate')}))
            AS discount_amount,
            concat('pos.order', ',', pos.id) AS order_reference"""

        additional_fields = self._select_additional_fields()
        additional_fields_info = self._fill_pos_fields(additional_fields)
        template = """,
            %s AS %s"""
        for fname, value in additional_fields_info.items():
            select_ += template % (value, fname)
        return select_

    def _available_additional_pos_fields(self):
        """Hook to replace the additional fields from sale with the one from pos_sale."""
        return {
            'warehouse_id': 'picking.warehouse_id',
        }

    def _fill_pos_fields(self, additional_fields):
        """Hook to fill additional fields for the pos_sale.

        :param values: dictionary of values to fill
        :type values: dict
        """
        filled_fields = {x: 'NULL' for x in additional_fields}
        for fname, value in self._available_additional_pos_fields().items():
            if fname in additional_fields:
                filled_fields[fname] = value
        return filled_fields

    def _from_pos(self):
        currency_table = self.env['res.currency']._get_simple_currency_table(self.env.companies)
        return """
            pos_order_line l
            JOIN pos_order pos ON l.order_id = pos.id
            LEFT JOIN res_partner partner ON (pos.partner_id=partner.id OR pos.partner_id = NULL)
            LEFT JOIN product_product p ON l.product_id=p.id
            LEFT JOIN product_template t ON p.product_tmpl_id=t.id
            LEFT JOIN uom_uom u ON u.id=t.uom_id
            LEFT JOIN pos_session session ON session.id = pos.session_id
            LEFT JOIN pos_config config ON config.id = session.config_id
            LEFT JOIN stock_picking_type picking ON picking.id = config.picking_type_id
            JOIN {currency_table} ON account_currency_table.company_id = pos.company_id
            """.format(
            currency_table=self.env.cr.mogrify(currency_table).decode(self.env.cr.connection.encoding),
            )

    def _where_pos(self):
        return """
            l.sale_order_line_id IS NULL"""

    def _group_by_pos(self):
        return """
            l.order_id,
            l.product_id,
            l.price_unit,
            l.discount,
            l.qty,
            t.uom_id,
            t.categ_id,
            pos.id,
            pos.name,
            pos.date_order,
            pos.partner_id,
            pos.user_id,
            pos.state,
            pos.company_id,
            pos.pricelist_id,
            p.product_tmpl_id,
            partner.commercial_partner_id,
            partner.country_id,
            partner.industry_id,
            partner.state_id,
            partner.zip,
            u.factor,
            pos.crm_team_id,
            account_currency_table.rate,
            picking.warehouse_id"""

    def _query(self):
        res = super()._query()
        return res + f"""UNION ALL (
            SELECT {self._select_pos()}
            FROM {self._from_pos()}
            WHERE {self._where_pos()}
            GROUP BY {self._group_by_pos()}
            )
        """

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
access_crm_team_pos_manager,crm.team.pos.manager,model_crm_team,point_of_sale.group_pos_manager,1,1,1,1

```

## File: security\pos_sale_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <record id="pos_sale_rule_pos_channel_pos_manager" model="ir.rule">
        <field name="name">POS Sales Team</field>
        <field name="model_id" ref="sales_team.model_crm_team"/>
        <field name="domain_force">[(1,'=',1)]</field>
        <field name="groups" eval="[(4, ref('point_of_sale.group_pos_manager'))]"/>
    </record>
</odoo>

```

## File: static\src\overrides\components\control_buttons\control_buttons.js

```javascript
/** @odoo-module */

import { patch } from "@web/core/utils/patch";
import { ControlButtons } from "@point_of_sale/app/screens/product_screen/control_buttons/control_buttons";
import { SelectCreateDialog } from "@web/views/view_dialogs/select_create_dialog";

patch(ControlButtons.prototype, {
    onClickQuotation() {
        this.dialog.add(SelectCreateDialog, {
            resModel: "sale.order",
            noCreate: true,
            multiSelect: false,
            domain: [
                ["state", "!=", "cancel"],
                ["invoice_status", "!=", "invoiced"],
                ["currency_id", "=", this.pos.currency.id],
            ],
            onSelected: async (resIds) => {
                await this.pos.onClickSaleOrder(resIds[0]);
            },
        });
    },
});

```

## File: static\src\overrides\components\control_buttons\control_buttons.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_sale.ControlButtons" t-inherit="point_of_sale.ControlButtons" t-inherit-mode="extension">
        <xpath expr="//t[@t-if='props.showRemainingButtons']/div/OrderlineNoteButton" position="after">
            <button t-att-class="buttonClass" t-on-click="() => this.onClickQuotation()">
                <i class="fa fa-link me-1" role="img" aria-label="Set Sale Order" title="Set Sale Order" /> Quotation/Order
            </button>
        </xpath>
    </t>
</templates>

```

## File: static\src\overrides\components\orderline\orderline.js

```javascript
import { Orderline } from "@point_of_sale/app/generic_components/orderline/orderline";
import { patch } from "@web/core/utils/patch";

patch(Orderline, {
    props: {
        ...Orderline.props,
        line: {
            ...Orderline.props.line,
            shape: {
                ...Orderline.props.line.shape,
                so_reference: { type: String, optional: true },
                details: {
                    type: Array,
                    optional: true,
                    element: {
                        type: Object,
                        shape: {
                            product_uom_qty: Number,
                            product_name: String,
                            total: String,
                        },
                    },
                },
            },
        },
    },
});

```

## File: static\src\overrides\components\orderline\orderline.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_sale.Orderline" t-inherit="point_of_sale.Orderline" t-inherit-mode="extension">
        <xpath expr="//ul[hasclass('info-list')]" position="inside" >
            <t t-if="line.so_reference">
                <li>
                    <i class="fa fa-shopping-basket me-1" role="img" aria-label="SO" title="SO"/>
                    <t t-esc="line.so_reference" />
                </li>
                <table t-if="line.details" class="sale-order-info">
                    <tr t-foreach="line.details" t-as="soLine" t-key="soLine_index">
                        <td class="text-truncate"><t t-esc="soLine.product_uom_qty"/>x</td>
                        <td class="text-truncate" t-esc="soLine.product_name" />
                        <td class="text-truncate">: </td>
                        <td t-if="!props.basic_receipt" class="text-truncate"><t t-esc="soLine.total" /> (tax incl.)</td>
                    </tr>
                </table>
            </t>
        </xpath>
    </t>
</templates>

```

## File: static\src\overrides\components\product_info_popup\product_info_popup.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_sale.ProductInfoPopup" t-inherit="point_of_sale.ProductInfoPopup" t-inherit-mode="extension">
        <xpath expr="//div[hasclass('financials-order')]" position="before">
            <div class="section-optional-product mt-3 mb-4 pb-4 border-bottom text-start" t-if="props.info.productInfo.optional_products.length > 0">
                <h3 class="section-title">Optional Products:</h3>
                <div class="section-optional-product-body" t-foreach="props.info.productInfo.optional_products" t-as="optional" t-key="optional.name">
                    <span>
                        <a class="text-primary text-decoration-underline" t-esc="optional.name" t-on-click="() => this.searchProduct(optional.name)"/>
 from
                        <t t-esc="env.utils.formatCurrency(optional.price)"/>
                    </span>
                </div>
            </div>
        </xpath>
    </t>
</templates>

```

## File: static\src\overrides\models\data_service_options.js

```javascript
import { DataServiceOptions } from "@point_of_sale/app/models/data_service_options";
import { patch } from "@web/core/utils/patch";

patch(DataServiceOptions.prototype, {
    get databaseTable() {
        return {
            ...super.databaseTable,
            "sale.order": {
                key: "id",
                condition: (record) => {
                    return record.models["pos.order.line"].find(
                        (l) => l.sale_order_origin_id?.id === record.id
                    );
                },
            },
            "sale.order.line": {
                key: "id",
                condition: (record) => {
                    return record.models["pos.order.line"].find(
                        (l) => l.sale_order_line_id?.id === record.id
                    );
                },
            },
        };
    },
});

```

## File: static\src\overrides\models\pos_order.js

```javascript
import { PosOrder } from "@point_of_sale/app/models/pos_order";
import { patch } from "@web/core/utils/patch";

patch(PosOrder.prototype, {
    //@override
    _get_ignored_product_ids_total_discount() {
        const productIds = super._get_ignored_product_ids_total_discount(...arguments);
        if (this.config.down_payment_product_id) {
            productIds.push(this.config.down_payment_product_id.id);
        }
        return productIds;
    },
});

```

## File: static\src\overrides\models\pos_order_line.js

```javascript
import { PosOrderline } from "@point_of_sale/app/models/pos_order_line";
import { formatCurrency } from "@point_of_sale/app/models/utils/currency";
import { patch } from "@web/core/utils/patch";

patch(PosOrderline.prototype, {
    setup(_defaultObj) {
        super.setup(...arguments);
        // It is possible that this orderline is initialized using server data,
        // meaning, it is loaded from localStorage or from server. This means
        // that some fields has already been assigned. Therefore, we only set the options
        // when the original value is falsy.
        if (this.sale_order_origin_id?.shipping_date) {
            this.order_id.setShippingDate(this.sale_order_origin_id.shipping_date);
        }
    },
    get_sale_order() {
        if (this.sale_order_origin_id) {
            const value = {
                name: this.sale_order_origin_id.name,
                details: this.down_payment_details || false,
            };

            return value;
        }
        return false;
    },
    getDisplayData() {
        let down_payment_details = [];

        // FIXME: This is a hack to handle the case where the down_payment_details is a stringified JSON.
        try {
            down_payment_details = JSON.parse(this.down_payment_details);
        } catch {
            down_payment_details = this.down_payment_details;
        }
        return {
            ...super.getDisplayData(),
            so_reference: this.sale_order_origin_id?.name,
            details: down_payment_details?.map?.((detail) => ({
                product_uom_qty: detail.product_uom_qty,
                product_name: detail.product_name,
                total: formatCurrency(detail.total, this.currency),
            })),
        };
    },
    /**
     * Set quantity based on the give sale order line.
     * @param {'sale.order.line'} saleOrderLine
     */
    async setQuantityFromSOL(saleOrderLine) {
        if (!saleOrderLine.has_valued_move_ids) {
            this.set_quantity(saleOrderLine.product_uom_qty);
        } else if (
            this.product_id.type === "service" &&
            !["sent", "draft"].includes(this.sale_order_origin_id.state)
        ) {
            this.set_quantity(saleOrderLine.qty_to_invoice);
        } else {
            this.set_quantity(
                saleOrderLine.product_uom_qty -
                    Math.max(saleOrderLine.qty_delivered, saleOrderLine.qty_invoiced)
            );
        }
    },
});

```

## File: static\src\overrides\models\pos_store.js

```javascript
import { _t } from "@web/core/l10n/translation";
import { sprintf } from "@web/core/utils/strings";
import { parseFloat } from "@web/views/fields/parsers";
import { SelectionPopup } from "@point_of_sale/app/utils/input_popups/selection_popup";
import { AlertDialog } from "@web/core/confirmation_dialog/confirmation_dialog";
import { NumberPopup } from "@point_of_sale/app/utils/input_popups/number_popup";
import { ask, makeAwaitable } from "@point_of_sale/app/store/make_awaitable_dialog";
import { enhancedButtons } from "@point_of_sale/app/generic_components/numpad/numpad";
import { patch } from "@web/core/utils/patch";
import { PosStore } from "@point_of_sale/app/store/pos_store";
import { compute_price_force_price_include } from "@point_of_sale/app/models/utils/tax_utils";

patch(PosStore.prototype, {
    async onClickSaleOrder(clickedOrderId) {
        const selectedOption = await makeAwaitable(this.dialog, SelectionPopup, {
            title: _t("What do you want to do?"),
            list: [
                { id: "0", label: _t("Settle the order"), item: "settle" },
                {
                    id: "1",
                    label: _t("Apply a down payment (percentage)"),
                    item: "dpPercentage",
                },
                {
                    id: "2",
                    label: _t("Apply a down payment (fixed amount)"),
                    item: "dpAmount",
                },
            ],
        });
        if (!selectedOption) {
            return;
        }
        const sale_order = await this._getSaleOrder(clickedOrderId);

        const currentSaleOrigin = this.get_order()
            .get_orderlines()
            .find((line) => line.sale_order_origin_id)?.sale_order_origin_id;
        if (currentSaleOrigin?.id) {
            const linkedSO = await this._getSaleOrder(currentSaleOrigin.id);
            if (
                linkedSO.partner_id?.id !== sale_order.partner_id?.id ||
                linkedSO.partner_invoice_id?.id !== sale_order.partner_invoice_id?.id ||
                linkedSO.partner_shipping_id?.id !== sale_order.partner_shipping_id?.id
            ) {
                this.add_new_order({
                    partner_id: sale_order.partner_id,
                });
                this.notification.add(_t("A new order has been created."));
            }
        }
        const orderFiscalPos =
            sale_order.fiscal_position_id &&
            this.models["account.fiscal.position"].find(
                (position) => position.id === sale_order.fiscal_position_id
            );
        if (orderFiscalPos) {
            this.get_order().update({
                fiscal_position_id: orderFiscalPos,
            });
        }
        if (sale_order.partner_id) {
            this.get_order().set_partner(sale_order.partner_id);
        }
        selectedOption == "settle"
            ? await this.settleSO(sale_order, orderFiscalPos)
            : await this.downPaymentSO(sale_order, selectedOption == "dpPercentage");
        this.selectOrderLine(this.get_order(), this.get_order().lines.at(-1));
    },
    async _getSaleOrder(id) {
        const sale_order = (await this.data.read("sale.order", [id]))[0];
        const orderlines = this.models["sale.order.line"].readMany(sale_order.raw.order_line);
        sale_order.order_line = orderlines;
        return sale_order;
    },
    async settleSO(sale_order, orderFiscalPos) {
        if (sale_order.pricelist_id) {
            this.get_order().set_pricelist(sale_order.pricelist_id);
        }
        let useLoadedLots = false;
        let userWasAskedAboutLoadedLots = false;
        let previousProductLine = null;

        const converted_lines = await this.data.call("sale.order.line", "read_converted", [
            sale_order.order_line.map((l) => l.id),
        ]);

        for (let i = 0; i < sale_order.order_line.length; ++i) {
            const line = sale_order.order_line[i];
            if (line.display_type === "line_note") {
                if (previousProductLine) {
                    const previousNote = previousProductLine.customer_note;
                    previousProductLine.customer_note = previousNote
                        ? previousNote + "--" + line.name
                        : line.name;
                }
                continue;
            }

            if (line.is_downpayment) {
                line.product_id = this.config.down_payment_product_id;
            }

            const newLineValues = {
                product_id: line.product_id,
                qty: line.product_uom_qty,
                price_unit: line.price_unit,
                price_type: "automatic",
                tax_ids:
                    orderFiscalPos || !line.tax_id
                        ? undefined
                        : line.tax_id.map((t) => ["link", t]),
                sale_order_origin_id: sale_order,
                sale_order_line_id: line,
                customer_note: line.customer_note,
                description: line.name,
                order_id: this.get_order(),
            };
            if (line.display_type === "line_section") {
                continue;
            }
            const newLine = await this.addLineToCurrentOrder(newLineValues, {}, false);
            previousProductLine = newLine;

            const converted_line = converted_lines[i];
            if (
                newLine.get_product().tracking !== "none" &&
                (this.pickingType.use_create_lots || this.pickingType.use_existing_lots) &&
                converted_line.lot_names.length > 0
            ) {
                if (!useLoadedLots && !userWasAskedAboutLoadedLots) {
                    useLoadedLots = await ask(this.dialog, {
                        title: _t("SN/Lots Loading"),
                        body: _t("Do you want to load the SN/Lots linked to the Sales Order?"),
                    });
                    userWasAskedAboutLoadedLots = true;
                }
                if (useLoadedLots) {
                    newLine.setPackLotLines({
                        modifiedPackLotLines: [],
                        newPackLotLines: (converted_line.lot_names || []).map((name) => ({
                            lot_name: name,
                        })),
                    });
                }
            }

            line.has_valued_move_ids = await this.data.call(
                "sale.order.line",
                "has_valued_move_ids",
                [line.id]
            );
            newLine.setQuantityFromSOL(line);
            newLine.set_unit_price(line.price_unit);
            newLine.set_discount(line.discount);

            const product_unit = line.product_id.uom_id;
            if (product_unit && !product_unit.is_pos_groupable) {
                let remaining_quantity = newLine.qty;
                newLine.delete();
                while (!this.env.utils.floatIsZero(remaining_quantity)) {
                    const splitted_line = this.models["pos.order.line"].create({
                        ...newLineValues,
                    });
                    splitted_line.set_quantity(Math.min(remaining_quantity, 1.0), true);
                    splitted_line.set_discount(line.discount);
                    remaining_quantity -= splitted_line.qty;
                }
            }
        }
    },
    async downPaymentSO(sale_order, isPercentage) {
        if (!this.config.down_payment_product_id && this.config.raw.down_payment_product_id) {
            await this.data.read("product.product", [this.config.raw.down_payment_product_id]);
        }
        if (!this.config.down_payment_product_id) {
            this.dialog.add(AlertDialog, {
                title: _t("No down payment product"),
                body: _t(
                    "It seems that you didn't configure a down payment product in your point of sale. You can go to your point of sale configuration to choose one."
                ),
            });
            return;
        }
        const payload = await makeAwaitable(this.dialog, NumberPopup, {
            title: _t("Down Payment"),
            subtitle: sprintf(
                _t("Due balance: %s"),
                this.env.utils.formatCurrency(sale_order.amount_unpaid)
            ),
            buttons: enhancedButtons(),
            formatDisplayedValue: (x) => (isPercentage ? `% ${x}` : x),
            feedback: (buffer) =>
                isPercentage && buffer
                    ? `(${this.env.utils.formatCurrency(
                          (sale_order.amount_unpaid * parseFloat(buffer)) / 100
                      )})`
                    : "",
        });
        if (!payload) {
            return;
        }
        const userValue = parseFloat(payload);
        let proposed_down_payment = userValue;
        if (isPercentage) {
            const down_payment_tax = this.models["account.tax"].get(
                this.config.down_payment_product_id.taxes_id
            );
            const percentageBase =
                !down_payment_tax || down_payment_tax.price_include
                    ? sale_order.amount_unpaid
                    : sale_order.amount_untaxed;
            proposed_down_payment = (percentageBase * userValue) / 100;
        }
        if (proposed_down_payment > sale_order.amount_unpaid) {
            this.dialog.add(AlertDialog, {
                title: _t("Error amount too high"),
                body: _t(
                    "You have tried to charge a down payment of %s but only %s remains to be paid, %s will be applied to the purchase order line.",
                    this.env.utils.formatCurrency(proposed_down_payment),
                    this.env.utils.formatCurrency(sale_order.amount_unpaid),
                    this.env.utils.formatCurrency(sale_order.amount_unpaid || 0)
                ),
            });
            proposed_down_payment = sale_order.amount_unpaid || 0;
        }
        this._createDownpaymentLines(sale_order, proposed_down_payment);
    },
    async _createDownpaymentLines(sale_order, total_down_payment) {
        //This function will create all the downpaymentlines. We will create one downpayment line per unique tax combination
        const percentage = total_down_payment / sale_order.amount_total;
        const grouped = Object.groupBy(
            sale_order.order_line.filter((ol) => ol.product_id),
            (ol) => ol.tax_id.map((tax_id) => tax_id.id).sort((a, b) => a - b)
        );

        // We need one unique line for the fixed amount taxes
        let fixed_taxes_downpayment = 0;
        const fixed_taxes_tab = [];
        const down_payment_line_to_create = [];

        Object.keys(grouped).forEach(async (key) => {
            const group = grouped[key];

            // We compute the values for the fixed taxes downpayment
            const fixed_taxes = group[0].tax_id.filter((tax) => tax.amount_type === "fixed");
            const total_qty = group.reduce((total, line) => (total += line.product_uom_qty), 0);
            fixed_taxes.forEach((tax) => {
                fixed_taxes_downpayment += tax.amount * total_qty * percentage;
                fixed_taxes_tab.push(group);
            });

            // We need to remove the amount of the fixed tax as they will have a separate line
            const fixed_tax_total_amount = fixed_taxes.reduce((total, tax) => {
                return total + tax.amount;
            }, 0);
            const total_price = group.reduce(
                (total, line) =>
                    (total += line.price_total - line.product_uom_qty * fixed_tax_total_amount),
                0
            );
            const down_payment_line_price = total_price * percentage;
            const taxes_to_apply = group[0].tax_id.filter((tax) => tax.amount_type !== "fixed");
            // We apply the taxes and keep the same price
            const new_price = compute_price_force_price_include(
                taxes_to_apply,
                down_payment_line_price,
                this.config.down_payment_product_id,
                this.config._product_default_values,
                this.company,
                this.currency,
                this.models
            );
            down_payment_line_to_create.push({
                price: new_price,
                tab: group,
                tax_ids: taxes_to_apply,
            });
        });
        if (fixed_taxes_downpayment !== 0) {
            // We try to merge the fixed taxes in one line that has no tax if possible
            const line = down_payment_line_to_create.find((line) => !line.tax_ids.length);
            if (line) {
                line.price += fixed_taxes_downpayment;
            } else {
                down_payment_line_to_create.push({
                    price: fixed_taxes_downpayment,
                    tab: fixed_taxes_tab.flat(),
                    tax_ids: [],
                });
            }
        }
        for (const down_payment_line of down_payment_line_to_create) {
            this.addLineToCurrentOrder({
                pos: this,
                order: this.get_order(),
                product_id: this.config.down_payment_product_id,
                price: down_payment_line.price,
                price_unit: down_payment_line.price,
                price_type: "automatic",
                sale_order_origin_id: sale_order,
                down_payment_details: down_payment_line.tab
                    .filter(
                        (line) =>
                            line.product_id &&
                            line.product_id.id !== this.config.down_payment_product_id.id
                    )
                    .map((line) => ({
                        product_name: line.product_id.display_name,
                        product_uom_qty: line.product_uom_qty,
                        price_unit: line.price_unit,
                        total: line.price_total,
                    })),
                tax_ids: [["link", ...down_payment_line.tax_ids]],
            });
        }
    },
    selectOrderLine(order, line) {
        super.selectOrderLine(...arguments);
        if (
            line &&
            this.config.down_payment_product_id &&
            line.product_id.id === this.config.down_payment_product_id.id
        ) {
            this.numpadMode = "price";
        }
    },
});

```

## File: views\point_of_sale_report.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="report_invoice_document" inherit_id="account.report_invoice_document">
        <xpath expr="//td[@name='account_invoice_line_name']" position="inside">
            <t t-set="pos_orders" t-value="o.sudo().pos_order_ids"/>
            <t t-if="pos_orders">
                <div style="margin-left:15px; font-style: italic">
                    <t t-set="down_payment_product" t-value="pos_orders[0].config_id.down_payment_product_id"/>
                    <t t-if="line.product_id == down_payment_product">
                        <t t-set="sale_orders" t-value="pos_orders[0].lines.filtered(lambda l: l.product_id == down_payment_product and l.price_subtotal_incl == line.price_total).mapped('sale_order_origin_id')"/>
                        <t t-if="sale_orders">
                            <t t-set="sale_order" t-value="sale_orders[0]"/>
                            <t t-foreach="sale_order.order_line" t-as="sale_order_line">
                                <t t-if="sale_order_line.product_id != down_payment_product and sale_order_line.tax_id == line.tax_ids">
                                    <div>
                                        <span style="margin-right: 5px;"><t t-esc="int(sale_order_line.product_uom_qty)"/>x</span>
                                        <span t-esc="sale_order_line.name" />
                                        <span style="margin: 0px 5px;">:</span>
                                        <t t-esc="sale_order_line.price_total" t-options='{"widget": "monetary", "display_currency": o.currency_id}'/>
                                    </div>
                                </t>
                            </t>
                            <div style="font-weight: bold">From <t t-esc="sale_order.name"/> </div>
                        </t>
                    </t>
                </div>
            </t>
        </xpath>
    </template>
</odoo>

```

## File: views\pos_order_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data>
        <record id="view_pos_order_form_inherit_pos_sale" model="ir.ui.view">
            <field name="name">pos.order.form.pos.sale</field>
            <field name="model">pos.order</field>
            <field name="inherit_id" ref="point_of_sale.view_pos_pos_form"/>
            <field name="arch" type="xml">
                <xpath expr="//button[@name='action_view_invoice']" position="before">
                    <button type="object"
                        name="action_view_sale_order"
                        class="oe_stat_button"
                        icon="fa-shopping-basket"
                        invisible="sale_order_count == 0" groups="sales_team.group_sale_salesman">
                        <div class="o_field_widget o_stat_info">
                            <span class="o_stat_value d-flex">
                                <field name="sale_order_count" widget="statinfo" nolabel="1" class="mr4" /> Transferred<br/>
                                from Sale
                            </span>
                        </div>
                    </button>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\product_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- TODO: remove this view in master -->
    <record id="product_template_form_view" model="ir.ui.view">
        <field name="name">product.template.form.inherit</field>
        <field name="model">product.template</field>
        <field name="inherit_id" ref="sale.product_template_form_view"/>
        <field name="arch" type="xml">
            <field name="invoice_policy" position="attributes">
                <attribute name="invisible" separator="or" add="type == 'combo'"/>
            </field>
        </field>
    </record>

</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.pos_sale</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="point_of_sale.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <block id="pos_accounting_section" position="after">
                <block title="Sales">
                    <setting string="Sales Team" help="Sales are reported to the following sales team">
                        <field name="pos_crm_team_id" context="{'kanban_view_ref': 'sales_team.crm_team_view_kanban'}" domain="['|', ('company_id', '=', company_id), ('company_id', '=', False)]"/>
                    </setting>
                    <setting id="down_payment_product" string="Down Payment Product" help="This product will be applied when down payment is made">
                        <field
                            name="pos_down_payment_product_id"
                            domain="[('type', '=', 'service'), '|', ('company_id', '=', company_id), ('company_id', '=', False)]"
                            context="{
                                'default_type': 'service',
                                'default_taxes_id': False,
                            }"
                        />
                    </setting>
                </block>
            </block>
        </field>
    </record>
</odoo>

```

## File: views\sales_team_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_pos_session_search_inherit_pos_sale" model="ir.ui.view">
        <field name="name">pos.session.search.view</field>
        <field name="model">pos.session</field>
        <field name="inherit_id" ref="point_of_sale.view_pos_session_search"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='config_id']" position="after">
                <field name="crm_team_id" context="{'kanban_view_ref': 'sales_team.crm_team_view_kanban'}"/>
            </xpath>
        </field>
    </record>

    <record id="pos_session_action_from_crm_team" model="ir.actions.act_window">
        <field name="name">Open Sessions</field>
        <field name="binding_model_id" ref="sales_team.model_crm_team"/>
        <field name="res_model">pos.session</field>
        <field name="view_mode">list,form</field>
        <field name="context">{'search_default_open_sessions': True, 'search_default_crm_team_id': active_id}</field>
    </record>

    <record id="view_pos_config_search_inherit_pos_sale" model="ir.ui.view">
        <field name="name">pos.config.search.view</field>
        <field name="model">pos.config</field>
        <field name="inherit_id" ref="point_of_sale.view_pos_config_search"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='name']" position="after">
                <field name="crm_team_id" context="{'kanban_view_ref': 'sales_team.crm_team_view_kanban'}"/>
            </xpath>
        </field>
    </record>

    <record id="crm_team_view_kanban_dashboard" model="ir.ui.view"> 
        <field name="name">crm.team.view.kanban.dashboard.inherit.pos</field>
        <field name="model">crm.team</field>
        <field name="inherit_id" ref="sales_team.crm_team_view_kanban_dashboard"/>
        <field name="arch" type="xml">
        <data>
            <xpath expr="//t[@name='first_options']" position="before">
                <div class="row" groups="point_of_sale.group_pos_user">
                    <a class="col-8" name="%(pos_session_action_from_crm_team)d" type="action">
                        <field class="me-1" name="pos_sessions_open_count"/>
                        <t t-if="record.pos_sessions_open_count.raw_value == 1">Session Running</t>
                        <t t-else="">Sessions Running</t>
                    </a>
                    <div class="col-4 text-end">
                        <field name="pos_order_amount_total" widget="monetary"/>
                    </div>
                </div>
            </xpath>

        </data>
        </field>
    </record>

</odoo>

```

## File: views\sale_order_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data>
        <record id="view_order_form_inherit_pos_sale" model="ir.ui.view">
            <field name="name">sale.order.form.pos.sale</field>
            <field name="model">sale.order</field>
            <field name="inherit_id" ref="sale.view_order_form"/>
            <field name="arch" type="xml">
                <xpath expr="//button[@name='action_view_invoice']" position="before">
                    <button type="object"
                        name="action_view_pos_order"
                        class="oe_stat_button"
                        icon="fa-shopping-basket"
                        invisible="pos_order_count == 0" groups="point_of_sale.group_pos_user">
                        <div class="o_field_widget o_stat_info">
                            <span class="o_stat_value d-flex">
                                <field name="pos_order_count" widget="statinfo" nolabel="1" class="mr4" /> Transferred<br/>
                                to POS
                            </span>
                        </div>
                    </button>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\stock_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="message_body" inherit_id="stock.message_body">
        <xpath expr="//t[contains(@t-if, 'move.state')]" position="inside">
            <t groups="point_of_sale.group_pos_user">
                <t t-set="pos_orders" t-value="move.sale_line_id.pos_order_line_ids.mapped('order_id')" />
                <t t-if="len(pos_orders)">
                    <li>
                        Delivered from
                        <t t-foreach="pos_orders" t-as="pos_order">
                            <a href="#" t-att-data-oe-model="pos_order._name" t-att-data-oe-id="pos_order.id">
                                <t t-esc="pos_order.display_name" />
                            </a><span t-if="pos_orders.ids[-1:] != pos_order.ids">, </span>
                        </t>
                    </li>
                </t>
            </t>
        </xpath>
    </template>
</odoo>

```

