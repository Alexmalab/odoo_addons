# Odoo Module: pos_sale

Category: Hidden

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
            'pos_sale/static/tests/**/*',
        ],
    },
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

        <record model="pos.config" id="point_of_sale.pos_config_main" forcecreate="0">
            <field name="down_payment_product_id" ref="pos_sale.default_downpayment_product"/>
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

from odoo import fields, models
from odoo.osv.expression import OR


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

    def _get_available_product_domain(self):
        domain = super()._get_available_product_domain()
        return OR([domain, [('id', '=', self.down_payment_product_id.id)]])

```

## File: models\pos_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.tools import float_compare, float_is_zero


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
        values.setdefault('crm_team_id', session.config_id.crm_team_id.id)
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
    def create_from_ui(self, orders, draft=False):
        order_ids = super(PosOrder, self).create_from_ui(orders, draft)
        if draft:
            return order_ids

        for order in self.sudo().browse([o['id'] for o in order_ids]):
            for line in order.lines.filtered(lambda l: l.product_id == order.config_id.down_payment_product_id and l.qty != 0 and (l.sale_order_origin_id or l.refunded_orderline_id.sale_order_origin_id)):
                sale_lines = line.sale_order_origin_id.order_line or line.refunded_orderline_id.sale_order_origin_id.order_line
                sale_order_origin = line.sale_order_origin_id or line.refunded_orderline_id.sale_order_origin_id
                sale_line = self.env['sale.order.line'].create({
                    'order_id': sale_order_origin.id,
                    'product_id': line.product_id.id,
                    'price_unit': line.price_unit,
                    'product_uom_qty': 0,
                    'tax_id': [(6, 0, line.tax_ids.ids)],
                    'is_downpayment': True,
                    'discount': line.discount,
                    'sequence': sale_lines and sale_lines[-1].sequence + 1 or 10,
                })
                line.sale_order_line_id = sale_line

            so_lines = order.lines.mapped('sale_order_line_id')

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
                    if picking.state not in ['waiting', 'confirmed', 'assigned']:
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

        return order_ids

    def action_view_sale_order(self):
        self.ensure_one()
        linked_orders = self.lines.mapped('sale_order_origin_id')
        return {
            'type': 'ir.actions.act_window',
            'name': _('Linked Sale Orders'),
            'res_model': 'sale.order',
            'view_mode': 'tree,form',
            'domain': [('id', 'in', linked_orders.ids)],
        }

    def _get_invoice_lines_values(self, line_values, pos_line):
        inv_line_vals = super()._get_invoice_lines_values(line_values, pos_line)

        if pos_line.sale_order_origin_id:
            origin_line = pos_line.sale_order_line_id
            origin_line._set_analytic_distribution(inv_line_vals)

        return inv_line_vals

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

class PosOrderLine(models.Model):
    _inherit = 'pos.order.line'

    sale_order_origin_id = fields.Many2one('sale.order', string="Linked Sale Order")
    sale_order_line_id = fields.Many2one('sale.order.line', string="Source Sale Order Line")
    down_payment_details = fields.Text(string="Down Payment Details")

    def _export_for_ui(self, orderline):
        result = super()._export_for_ui(orderline)
        result['down_payment_details'] = bool(orderline.down_payment_details) and orderline.down_payment_details
        result['sale_order_origin_id'] = bool(orderline.sale_order_origin_id) and orderline.sale_order_origin_id.read(fields=['name'])[0]
        result['sale_order_line_id'] = bool(orderline.sale_order_line_id) and orderline.sale_order_line_id.read(fields=['name'])[0]
        return result

    def _order_line_fields(self, line, session_id=None):
        result = super()._order_line_fields(line, session_id)
        vals = result[2]
        if vals.get('sale_order_origin_id', False):
            vals['sale_order_origin_id'] = vals['sale_order_origin_id']['id']
        if vals.get('sale_order_line_id', False):
            #We need to make sure the order line has not been deleted while the order was being handled in the PoS
            order_line = self.env['sale.order.line'].search([('id', '=', vals['sale_order_line_id']['id'])], limit=1)
            vals['sale_order_line_id'] = order_line.id if order_line else False
        return result

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

from odoo import fields, models
from odoo.osv.expression import OR


class PosSession(models.Model):
    _inherit = 'pos.session'

    crm_team_id = fields.Many2one('crm.team', related='config_id.crm_team_id', string="Sales Team", readonly=True)

    def _loader_params_product_product(self):
        result = super()._loader_params_product_product()
        result['search_params']['fields'].extend(['invoice_policy', 'type'])
        return result

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
    _inherit = 'sale.order'

    pos_order_line_ids = fields.One2many('pos.order.line', 'sale_order_origin_id', string="Order lines Transfered to Point of Sale", readonly=True, groups="point_of_sale.group_pos_user")
    pos_order_count = fields.Integer(string='Pos Order Count', compute='_count_pos_order', readonly=True, groups="point_of_sale.group_pos_user")
    amount_unpaid = fields.Monetary(string='Unpaid Amount', compute='_compute_amount_unpaid', store=True, help="The amount due from the sale order.")

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
            'view_mode': 'tree,form',
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
            order_amount = sum(order.pos_order_line_ids.filtered(lambda pol: pol.sale_order_line_id.is_downpayment).mapped('price_subtotal_incl'))
            order.amount_to_invoice -= order_amount


class SaleOrderLine(models.Model):
    _inherit = 'sale.order.line'

    pos_order_line_ids = fields.One2many('pos.order.line', 'sale_order_line_id', string="Order lines Transfered to Point of Sale", readonly=True, groups="point_of_sale.group_pos_user")

    @api.depends('pos_order_line_ids.qty', 'pos_order_line_ids.order_id.picking_ids', 'pos_order_line_ids.order_id.picking_ids.state')
    def _compute_qty_delivered(self):
        super()._compute_qty_delivered()
        for sale_line in self:
            if all(picking.state == 'done' for picking in sale_line.pos_order_line_ids.order_id.picking_ids):
                sale_line.qty_delivered += sum((self._convert_qty(sale_line, pos_line.qty, 'p2s') for pos_line in sale_line.pos_order_line_ids if sale_line.product_id.type != 'service'), 0)

    @api.depends('pos_order_line_ids.qty')
    def _compute_qty_invoiced(self):
        super()._compute_qty_invoiced()
        for sale_line in self:
            sale_line.qty_invoiced += sum([self._convert_qty(sale_line, pos_line.qty, 'p2s') for pos_line in sale_line.pos_order_line_ids], 0)

    def _get_sale_order_fields(self):
        return ["product_id", "display_name", "price_unit", "product_uom_qty", "tax_id", "qty_delivered", "qty_invoiced", "discount", "qty_to_invoice", "price_total"]

    def read_converted(self):
        field_names = self._get_sale_order_fields()
        results = []
        for sale_line in self:
            if sale_line.product_type:
                product_uom = sale_line.product_id.uom_id
                sale_line_uom = sale_line.product_uom
                item = sale_line.read(field_names)[0]
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
        return super()._create_move_from_pos_order_lines(lines)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import pos_config
from . import pos_order
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
            ('invoiced', 'Invoiced')
        ],
    )

    order_reference = fields.Reference(selection_add=[('pos.order', 'POS Order')])

    def _select_pos(self):
        select_ = f"""
            -MIN(l.id) AS id,
            l.product_id AS product_id,
            t.uom_id AS product_uom,
            SUM(l.qty) AS product_uom_qty,
            SUM(l.qty) AS qty_delivered,
            0 AS qty_to_deliver,
            CASE WHEN pos.state = 'invoiced' THEN SUM(l.qty) ELSE 0 END AS qty_invoiced,
            CASE WHEN pos.state != 'invoiced' THEN SUM(l.qty) ELSE 0 END AS qty_to_invoice,
            SUM(l.price_subtotal_incl)
                / MIN({self._case_value_or_one('pos.currency_rate')})
                * {self._case_value_or_one('currency_table.rate')}
            AS price_total,
            SUM(l.price_subtotal)
                / MIN({self._case_value_or_one('pos.currency_rate')})
                * {self._case_value_or_one('currency_table.rate')}
            AS price_subtotal,
            (CASE WHEN pos.state != 'invoiced' THEN SUM(l.price_subtotal) ELSE 0 END)
                / MIN({self._case_value_or_one('pos.currency_rate')})
                * {self._case_value_or_one('currency_table.rate')}
            AS amount_to_invoice,
            (CASE WHEN pos.state = 'invoiced' THEN SUM(l.price_subtotal) ELSE 0 END)
                / MIN({self._case_value_or_one('pos.currency_rate')})
                * {self._case_value_or_one('currency_table.rate')}
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
            NULL AS analytic_account_id,
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
                * {self._case_value_or_one('currency_table.rate')}))
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
            JOIN {currency_table} ON currency_table.company_id = pos.company_id
            """.format(
            currency_table=self.env['res.currency']._get_query_currency_table(self.env.companies.ids, fields.Date.today())
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
            currency_table.rate,
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

## File: static\src\app\order_management_screen\sale_order_fetcher\sale_order_fetcher.js

```javascript
/** @odoo-module */

import { registry } from "@web/core/registry";
import { EventBus } from "@odoo/owl";

class SaleOrderFetcher extends EventBus {
    static serviceDependencies = ["orm", "pos"];
    constructor({ orm, pos }) {
        super();
        this.currentPage = 1;
        this.ordersToShow = [];
        this.totalCount = 0;
        this.orm = orm;
        this.pos = pos;
    }

    /**
     * for nPerPage = 10
     * +--------+----------+
     * | nItems | lastPage |
     * +--------+----------+
     * |     2  |       1  |
     * |    10  |       1  |
     * |    11  |       2  |
     * |    30  |       3  |
     * |    35  |       4  |
     * +--------+----------+
     */
    get lastPage() {
        const nItems = this.totalCount;
        return Math.trunc(nItems / (this.nPerPage + 1)) + 1;
    }
    /**
     * Calling this methods populates the `ordersToShow` then trigger `update` event.
     * @related get
     *
     * NOTE: This is tightly-coupled with pagination. So if the current page contains all
     * active orders, it will not fetch anything from the server but only sets `ordersToShow`
     * to the active orders that fits the current page.
     */
    async fetch() {
        // Show orders from the backend.
        const offset = this.nPerPage + (this.currentPage - 1 - 1) * this.nPerPage;
        const limit = this.nPerPage;
        this.ordersToShow = await this._fetch(limit, offset);

        this.trigger("update");
    }
    /**
     * This returns the orders from the backend that needs to be shown.
     * If the order is already in cache, the full information about that
     * order is not fetched anymore, instead, we use info from cache.
     *
     * @param {number} limit
     * @param {number} offset
     */
    async _fetch(limit, offset) {
        const sale_orders = await this._getOrderIdsForCurrentPage(limit, offset);

        this.totalCount = sale_orders.length;
        return sale_orders;
    }
    async _getOrderIdsForCurrentPage(limit, offset) {
        const domain = [["currency_id", "=", this.pos.currency.id]].concat(
            this.searchDomain || []
        );

        this.pos.set_synch("connecting");
        const saleOrders = await this.orm.searchRead(
            "sale.order",
            domain,
            [
                "name",
                "partner_id",
                "amount_total",
                "date_order",
                "state",
                "user_id",
                "amount_unpaid",
            ],
            { offset, limit }
        );

        this.pos.set_synch("connected");
        return saleOrders;
    }

    nextPage() {
        if (this.currentPage < this.lastPage) {
            this.currentPage += 1;
            this.fetch();
        }
    }
    prevPage() {
        if (this.currentPage > 1) {
            this.currentPage -= 1;
            this.fetch();
        }
    }
    /**
     * @param {integer|undefined} id id of the cached order
     * @returns {Array<models.Order>}
     */
    get(id) {
        return this.ordersToShow;
    }
    setSearchDomain(searchDomain) {
        this.searchDomain = searchDomain;
    }
    setNPerPage(val) {
        this.nPerPage = val;
    }
    setPage(page) {
        this.currentPage = page;
    }
}

export const saleOrderFetcherService = {
    dependencies: SaleOrderFetcher.serviceDependencies,
    start(env, deps) {
        return new SaleOrderFetcher(deps);
    },
};

registry.category("services").add("sale_order_fetcher", saleOrderFetcherService);

```

## File: static\src\app\order_management_screen\sale_order_list\sale_order_list.js

```javascript
/** @odoo-module */

import { Component, useState } from "@odoo/owl";
import { SaleOrderRow } from "@pos_sale/app/order_management_screen/sale_order_row/sale_order_row";
import { useService } from "@web/core/utils/hooks";

/**
 * @props {models.Order} [initHighlightedOrder] initially highligted order
 * @props {Array<models.Order>} orders
 */
export class SaleOrderList extends Component {
    static components = { SaleOrderRow };
    static template = "pos_sale.SaleOrderList";

    setup() {
        this.ui = useState(useService("ui"));
        this.state = useState({ highlightedOrder: this.props.initHighlightedOrder || null });
    }
    get highlightedOrder() {
        return this.state.highlightedOrder;
    }
    _onClickOrder(order) {
        this.state.highlightedOrder = order;
    }
}

```

## File: static\src\app\order_management_screen\sale_order_list\sale_order_list.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="pos_sale.SaleOrderList">
        <div class="orders overflow-y-auto">
            <div class="header-row d-flex text-bg-700 fw-bolder" t-att-class="{ 'd-none': ui.isSmall }">
                <div class="col name p-2">Order</div>
                <div class="col date p-2">Date</div>
                <div class="col customer p-2">Customer</div>
                <div class="col salesman p-2">Salesperson</div>
                <div class="col total p-2">Total</div>
                <div class="col state p-2">State</div>
            </div>
            <div class="order-list">
                <t t-foreach="props.orders" t-as="order" t-key="order.id">
                    <SaleOrderRow
                        onClickSaleOrder.bind="props.onClickSaleOrder"
                        order="order"
                        highlightedOrder="highlightedOrder" />
                </t>
            </div>
        </div>
    </t>

</templates>

```

## File: static\src\app\order_management_screen\sale_order_management_control_panel\sale_order_management_control_panel.js

```javascript
/** @odoo-module */

import { useAutofocus, useService } from "@web/core/utils/hooks";
import { Component, useState } from "@odoo/owl";
import { usePos } from "@point_of_sale/app/store/pos_hook";

// NOTE: These are constants so that they are only instantiated once
// and they can be used efficiently by the OrderManagementControlPanel.
const VALID_SEARCH_TAGS = new Set(["date", "customer", "client", "name", "order"]);
const FIELD_MAP = {
    date: "date_order",
    customer: "partner_id.complete_name",
    client: "partner_id.complete_name",
    name: "name",
    order: "name",
};
const SEARCH_FIELDS = ["name", "partner_id.complete_name", "date_order"];

/**
 * @emits search
 */
export class SaleOrderManagementControlPanel extends Component {
    static template = "pos_sale.SaleOrderManagementControlPanel";

    setup() {
        this.pos = usePos();
        this.ui = useState(useService("ui"));
        this.saleOrderFetcher = useService("sale_order_fetcher");
        useAutofocus();

        const currentPartner = this.pos.get_order().get_partner();
        if (currentPartner) {
            this.pos.orderManagement.searchString = `"${currentPartner.name}"`;
        }
        this.saleOrderFetcher.setSearchDomain(this._computeDomain());
    }
    onInputKeydown(event) {
        if (event.key === "Enter") {
            this.props.onSearch(this._computeDomain());
        }
    }
    get showPageControls() {
        return this.saleOrderFetcher.lastPage > 1;
    }
    get pageNumber() {
        const currentPage = this.saleOrderFetcher.currentPage;
        const lastPage = this.saleOrderFetcher.lastPage;
        return isNaN(lastPage) ? "" : `(${currentPage}/${lastPage})`;
    }
    get validSearchTags() {
        return VALID_SEARCH_TAGS;
    }
    get fieldMap() {
        return FIELD_MAP;
    }
    get searchFields() {
        return SEARCH_FIELDS;
    }
    /**
     * E.g. 1
     * ```
     *   searchString = 'Customer 1'
     *   result = [
     *      '|',
     *      '|',
     *      ['pos_reference', 'ilike', '%Customer 1%'],
     *      ['partner_id.complete_name', 'ilike', '%Customer 1%'],
     *      ['date_order', 'ilike', '%Customer 1%']
     *   ]
     * ```
     *
     * E.g. 2
     * ```
     *   searchString = 'date: 2020-05'
     *   result = [
     *      ['date_order', 'ilike', '%2020-05%']
     *   ]
     * ```
     *
     * E.g. 3
     * ```
     *   searchString = 'customer: Steward, date: 2020-05-01'
     *   result = [
     *      ['partner_id.complete_name', 'ilike', '%Steward%'],
     *      ['date_order', 'ilike', '%2020-05-01%']
     *   ]
     * ```
     */
    _computeDomain() {
        let domain = [
            ["state", "!=", "cancel"],
            ["invoice_status", "!=", "invoiced"],
        ];
        const input = this.pos.orderManagement.searchString.trim();
        if (!input) {
            return domain;
        }

        let searchConditions;
        let isQuoted = false;
        if (input.startsWith('"') && input.endsWith('"')) {
            searchConditions = [input.slice(1, -1)];
            isQuoted = true;
        } else {
            searchConditions = input.split(/[,&]\s*/);
        }

        if (searchConditions.length === 1) {
            const cond = searchConditions[0].split(/:\s*/);
            if (cond.length === 1 || isQuoted) {
                domain = domain.concat(Array(this.searchFields.length - 1).fill("|"));
                domain = domain.concat(
                    this.searchFields.map((field) => [field, "ilike", `%${cond[0]}%`])
                );
                return domain;
            }
        }

        for (const cond of searchConditions) {
            const [tag, value] = cond.split(/:\s*/);
            if (!this.validSearchTags.has(tag)) {
                continue;
            }
            domain.push([this.fieldMap[tag], "ilike", `%${value}%`]);
        }
        return domain;
    }
    clearSearch() {
        this.pos.orderManagement.searchString = "";
        this.onInputKeydown({ key: "Enter" });
    }
}

```

## File: static\src\app\order_management_screen\sale_order_management_control_panel\sale_order_management_control_panel.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="pos_sale.SaleOrderManagementControlPanel">
        <div class="control-panel d-flex justify-content-between align-items-center m-1 p-2 gap-2">
            <div class="d-flex">
                <button class="item button back btn btn-lg btn-secondary" t-on-click="() => pos.closeScreen()">
                    <span class="search-icon d-flex align-items-center">
                        <i class="fa fa-angle-double-left"/>
                        <span t-if="!ui.isSmall" class="ms-2"> Back</span>
                    </span>
                </button>
            </div>
            <div class="item search-box d-flex flex-grow-1 ms-3">
                <div class="input-group">
                    <span class="icon input-group-text">
                        <i class="oi oi-search" />
                    </span>
                    <input type="text" class="form-control form-control-lg" t-ref="autofocus" t-model="pos.orderManagement.searchString" t-on-keydown="onInputKeydown" placeholder="E.g. customer: Steward, date: 2020-05-09" />
                    <button type="button" class="clear  btn btn-secondary" t-on-click="() => this.clearSearch()">
                        <i class="oi oi-close" />
                    </button>
                </div> 
            </div>
            <div t-if="showPageControls" class="item item d-flex align-items-center ms-3 gap-2">
                <div class="page-controls input-group">
                    <button class="previous btn btn-lg btn-secondary" t-on-click="() => this.props.onPrevPage">
                        <i class="oi oi-chevron-left" role="img" aria-label="Previous Order List" title="Previous Order List"></i>
                    </button>
                    <button class="next btn btn-lg btn-secondary" t-on-click="() => this.props.onNextPage">
                        <i class="oi oi-chevron-right" role="img" aria-label="Next Order List" title="Next Order List"></i>
                    </button>
                </div>
                <div class="page">
                    <span><t t-esc="pageNumber" /></span>
                </div>
            </div>
            <div t-else="" class="item"></div>
        </div>
    </t>

</templates>

```

## File: static\src\app\order_management_screen\sale_order_management_screen\sale_order_management_screen.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { sprintf } from "@web/core/utils/strings";
import { parseFloat } from "@web/views/fields/parsers";
import { floatIsZero } from "@web/core/utils/numbers";
import { useBus, useService } from "@web/core/utils/hooks";
import { registry } from "@web/core/registry";
import { ControlButtonsMixin } from "@point_of_sale/app/utils/control_buttons_mixin";
import { Orderline } from "@point_of_sale/app/store/models";

import { SelectionPopup } from "@point_of_sale/app/utils/input_popups/selection_popup";
import { ErrorPopup } from "@point_of_sale/app/errors/popups/error_popup";
import { ConfirmPopup } from "@point_of_sale/app/utils/confirm_popup/confirm_popup";
import { NumberPopup } from "@point_of_sale/app/utils/input_popups/number_popup";

import { SaleOrderList } from "@pos_sale/app/order_management_screen/sale_order_list/sale_order_list";
import { SaleOrderManagementControlPanel } from "@pos_sale/app/order_management_screen/sale_order_management_control_panel/sale_order_management_control_panel";
import { Component, onMounted, useRef } from "@odoo/owl";
import { usePos } from "@point_of_sale/app/store/pos_hook";

/**
 * ID getter to take into account falsy many2one value.
 * @param {[id: number, display_name: string] | false} fieldVal many2one field value
 * @returns {number | false}
 */
function getId(fieldVal) {
    return fieldVal && fieldVal[0];
}

export class SaleOrderManagementScreen extends ControlButtonsMixin(Component) {
    static storeOnOrder = false;
    static components = { SaleOrderList, SaleOrderManagementControlPanel };
    static template = "pos_sale.SaleOrderManagementScreen";

    setup() {
        super.setup();
        this.pos = usePos();
        this.popup = useService("popup");
        this.orm = useService("orm");
        this.root = useRef("root");
        this.numberBuffer = useService("number_buffer");
        this.saleOrderFetcher = useService("sale_order_fetcher");
        this.notification = useService("pos_notification");

        useBus(this.saleOrderFetcher, "update", this.render);

        onMounted(this.onMounted);
    }
    onMounted() {
        // calculate how many can fit in the screen.
        // It is based on the height of the header element.
        // So the result is only accurate if each row is just single line.
        const flexContainer = this.root.el.querySelector(".flex-container");
        const cpEl = this.root.el.querySelector(".control-panel");
        const headerEl = this.root.el.querySelector(".header-row");
        const val = Math.trunc(
            (flexContainer.offsetHeight - cpEl.offsetHeight - headerEl.offsetHeight) /
                headerEl.offsetHeight
        );
        this.saleOrderFetcher.setNPerPage(val);
        this.saleOrderFetcher.fetch();
    }
    _getSaleOrderOrigin(order) {
        for (const line of order.get_orderlines()) {
            if (line.sale_order_origin_id) {
                return line.sale_order_origin_id;
            }
        }
        return false;
    }
    get selectedPartner() {
        const order = this.pos.orderManagement.selectedOrder;
        return order ? order.get_partner() : null;
    }
    get orders() {
        return this.saleOrderFetcher.get();
    }
    async _setNumpadMode(event) {
        const { mode } = event.detail;
        this.numpadMode = mode;
        this.numberBuffer.reset();
    }
    onNextPage() {
        this.saleOrderFetcher.nextPage();
    }
    onPrevPage() {
        this.saleOrderFetcher.prevPage();
    }
    onSearch(domain) {
        this.saleOrderFetcher.setSearchDomain(domain);
        this.saleOrderFetcher.setPage(1);
        this.saleOrderFetcher.fetch();
    }
    async onClickSaleOrder(clickedOrder) {
        const { confirmed, payload: selectedOption } = await this.popup.add(SelectionPopup, {
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

        if (confirmed) {
            let currentPOSOrder = this.pos.get_order();
            const sale_order = await this._getSaleOrder(clickedOrder.id);
            clickedOrder.shipping_date = this.pos.config.ship_later && sale_order.shipping_date;

            const currentSaleOrigin = this._getSaleOrderOrigin(currentPOSOrder);
            const currentSaleOriginId = currentSaleOrigin && currentSaleOrigin.id;

            if (currentSaleOriginId) {
                const linkedSO = await this._getSaleOrder(currentSaleOriginId);
                if (
                    getId(linkedSO.partner_id) !== getId(sale_order.partner_id) ||
                    getId(linkedSO.partner_invoice_id) !== getId(sale_order.partner_invoice_id) ||
                    getId(linkedSO.partner_shipping_id) !== getId(sale_order.partner_shipping_id)
                ) {
                    currentPOSOrder = this.pos.add_new_order();
                    this.notification.add(_t("A new order has been created."), 4000);
                }
            }

            const order_partner = this.pos.db.get_partner_by_id(sale_order.partner_id[0]);
            if (order_partner) {
                currentPOSOrder.set_partner(order_partner);
            } else {
                try {
                    await this.pos._loadPartners([sale_order.partner_id[0]]);
                } catch {
                    const title = _t("Customer loading error");
                    const body = _t(
                        "There was a problem in loading the %s customer.",
                        sale_order.partner_id[1]
                    );
                    await this.popup.add(ErrorPopup, { title, body });
                }
                currentPOSOrder.set_partner(
                    this.pos.db.get_partner_by_id(sale_order.partner_id[0])
                );
            }
            const orderFiscalPos = sale_order.fiscal_position_id
                ? this.pos.fiscal_positions.find(
                      (position) => position.id === sale_order.fiscal_position_id[0]
                  )
                : false;
            if (orderFiscalPos) {
                currentPOSOrder.fiscal_position = orderFiscalPos;
            }
            const orderPricelist = sale_order.pricelist_id
                ? this.pos.pricelists.find(
                      (pricelist) => pricelist.id === sale_order.pricelist_id[0]
                  )
                : false;
            if (orderPricelist) {
                currentPOSOrder.set_pricelist(orderPricelist);
            }

            if (selectedOption == "settle") {
                // settle the order
                const lines = sale_order.order_line;
                const product_to_add_in_pos = lines
                    .filter((line) => !this.pos.db.get_product_by_id(line.product_id[0]))
                    .map((line) => line.product_id[0]);
                if (product_to_add_in_pos.length) {
                    const { confirmed } = await this.popup.add(ConfirmPopup, {
                        title: _t("Products not available in POS"),
                        body: _t(
                            "Some of the products in your Sale Order are not available in POS, do you want to import them?"
                        ),
                        confirmText: _t("Yes"),
                        cancelText: _t("No"),
                    });
                    if (confirmed) {
                        await this.pos._addProducts(product_to_add_in_pos);
                    }
                }

                /**
                 * This variable will have 3 values, `undefined | false | true`.
                 * Initially, it is `undefined`. When looping thru each sale.order.line,
                 * when a line comes with lots (`.lot_names`), we use these lot names
                 * as the pack lot of the generated pos.order.line. We ask the user
                 * if he wants to use the lots that come with the sale.order.lines to
                 * be used on the corresponding pos.order.line only once. So, once the
                 * `useLoadedLots` becomes true, it will be true for the succeeding lines,
                 * and vice versa.
                 */
                let useLoadedLots;

                for (var i = 0; i < lines.length; i++) {
                    const line = lines[i];
                    if (!this.pos.db.get_product_by_id(line.product_id[0])) {
                        continue;
                    }

                    const line_values = {
                        pos: this.pos,
                        order: this.pos.get_order(),
                        product: this.pos.db.get_product_by_id(line.product_id[0]),
                        description: line.name,
                        price: line.price_unit,
                        tax_ids: orderFiscalPos ? undefined : line.tax_id,
                        price_manually_set: false,
                        price_type: "automatic",
                        sale_order_origin_id: clickedOrder,
                        sale_order_line_id: line,
                        customer_note: line.customer_note,
                    };
                    const new_line = new Orderline({ env: this.env }, line_values);

                    if (
                        new_line.get_product().tracking !== "none" &&
                        (this.pos.picking_type.use_create_lots ||
                            this.pos.picking_type.use_existing_lots) &&
                        line.lot_names.length > 0
                    ) {
                        // Ask once when `useLoadedLots` is undefined, then reuse it's value on the succeeding lines.
                        const { confirmed } =
                            useLoadedLots === undefined
                                ? await this.popup.add(ConfirmPopup, {
                                      title: _t("SN/Lots Loading"),
                                      body: _t(
                                          "Do you want to load the SN/Lots linked to the Sales Order?"
                                      ),
                                      confirmText: _t("Yes"),
                                      cancelText: _t("No"),
                                  })
                                : { confirmed: useLoadedLots };
                        useLoadedLots = confirmed;
                        if (useLoadedLots) {
                            new_line.setPackLotLines({
                                modifiedPackLotLines: [],
                                newPackLotLines: (line.lot_names || []).map((name) => ({
                                    lot_name: name,
                                })),
                            });
                        }
                    }
                    new_line.setQuantityFromSOL(line);
                    new_line.set_unit_price(line.price_unit);
                    new_line.set_discount(line.discount);
                    const product = this.pos.db.get_product_by_id(line.product_id[0]);
                    const product_unit = product.get_unit();
                    if (product_unit && !product.get_unit().is_pos_groupable) {
                        let remaining_quantity = new_line.quantity;
                        while (!floatIsZero(remaining_quantity, 6)) {
                            const splitted_line = new Orderline({ env: this.env }, line_values);
                            splitted_line.set_quantity(Math.min(remaining_quantity, 1.0), true);
                            splitted_line.set_discount(line.discount);
                            this.pos.get_order().add_orderline(splitted_line);
                            remaining_quantity -= splitted_line.quantity;
                        }
                    } else {
                        this.pos.get_order().add_orderline(new_line);
                    }
                }
            } else {
                // apply a downpayment
                if (this.pos.config.down_payment_product_id) {
                    let down_payment_product = this.pos.db.get_product_by_id(
                        this.pos.config.down_payment_product_id[0]
                    );
                    if (!down_payment_product) {
                        await this.pos._addProducts([this.pos.config.down_payment_product_id[0]]);
                        down_payment_product = this.pos.db.get_product_by_id(
                            this.pos.config.down_payment_product_id[0]
                        );
                    }

                    let down_payment;
                    let popupTitle = "";
                    let popupInputSuffix = "";
                    const popupTotalDue = sale_order.amount_unpaid;
                    let getInputBufferReminder = () => false;
                    const popupSubtitle = _t("Due balance: %s");
                    if (selectedOption == "dpAmount") {
                        popupTitle = _t("Down Payment");
                        popupInputSuffix = this.pos.currency.symbol;
                    } else {
                        popupTitle = _t("Down Payment");
                        popupInputSuffix = "%";
                        getInputBufferReminder = (buffer) => {
                            if (buffer && buffer.length > 0) {
                                const percentage = parseFloat(buffer);
                                if (isNaN(percentage)) {
                                    return false;
                                }
                                return this.env.utils.formatCurrency(
                                    (popupTotalDue * percentage) / 100
                                );
                            } else {
                                return false;
                            }
                        };
                    }
                    const { confirmed, payload } = await this.popup.add(NumberPopup, {
                        title: popupTitle,
                        subtitle: sprintf(
                            popupSubtitle,
                            this.env.utils.formatCurrency(popupTotalDue)
                        ),
                        inputSuffix: popupInputSuffix,
                        startingValue: 0,
                        getInputBufferReminder,
                    });

                    if (!confirmed) {
                        return;
                    }
                    if (selectedOption == "dpAmount") {
                        down_payment = parseFloat(payload);
                    } else {
                        down_payment = (popupTotalDue * parseFloat(payload)) / 100;
                    }

                    if (down_payment > sale_order.amount_unpaid) {
                        const errorBody = _t(
                            "You have tried to charge a down payment of %s but only %s remains to be paid, %s will be applied to the purchase order line.",
                            this.env.utils.formatCurrency(down_payment),
                            this.env.utils.formatCurrency(sale_order.amount_unpaid),
                            sale_order.amount_unpaid > 0
                                ? this.env.utils.formatCurrency(sale_order.amount_unpaid)
                                : this.env.utils.formatCurrency(0)
                        );
                        await this.popup.add(ErrorPopup, {
                            title: _t("Error amount too high"),
                            body: errorBody,
                        });
                        down_payment = sale_order.amount_unpaid > 0 ? sale_order.amount_unpaid : 0;
                    }
                    this._createDownpaymentLines(
                        sale_order,
                        down_payment,
                        clickedOrder,
                        down_payment_product
                    );
                } else {
                    const title = _t("No down payment product");
                    const body = _t(
                        "It seems that you didn't configure a down payment product in your point of sale. You can go to your point of sale configuration to choose one."
                    );
                    await this.popup.add(ErrorPopup, { title, body });
                }
            }

            this.pos.closeScreen();
        }
    }

    _createDownpaymentLines(sale_order, total_down_payment, clickedOrder, down_payment_product) {
        //This function will create all the downpaymentlines. We will create one downpayment line per unique tax combination
        const percentage = total_down_payment / sale_order.amount_total;
        const grouped = {};
        sale_order.order_line.forEach((obj) => {
            const sortedTaxes = obj.tax_id.slice().sort((a, b) => a - b);
            const key = sortedTaxes.join(",");
            if (!grouped[key]) {
                grouped[key] = [];
            }
            grouped[key].push(obj);
        });

        // We need one unique line for the fixed amount taxes
        let fixed_taxes_downpayment = 0;
        const fixed_taxes_tab = [];
        const down_payment_line_to_create = [];

        Object.keys(grouped).forEach((key) => {
            const group = grouped[key];
            const tab = group.map((line) => ({
                product_name: line.product_id[1],
                product_uom_qty: line.product_uom_qty,
                price_unit: line.price_unit,
                total: line.price_total,
            }));

            // We compute the values for the fixed taxes downpayment
            const fixed_taxes = group[0].tax_id.filter(
                (id) => this.pos.taxes_by_id[id].amount_type === "fixed"
            );
            const total_qty = group.reduce((total, line) => (total += line.product_uom_qty), 0);
            fixed_taxes.forEach((tax_id) => {
                const tax = this.pos.taxes_by_id[tax_id];
                fixed_taxes_downpayment += tax.amount * total_qty * percentage;
                fixed_taxes_tab.push(tab);
            });

            // We need to remove the amount of the fixed tax as they will have a separate line
            const fixed_tax_total_amount = fixed_taxes.reduce((total, tax_id) => {
                const tax = this.pos.taxes_by_id[tax_id];
                return total + tax.amount;
            }, 0);
            const total_price = group.reduce(
                (total, line) =>
                    (total += line.price_total - line.product_uom_qty * fixed_tax_total_amount),
                0
            );
            const down_payment_line_price = total_price * percentage;
            // We apply the taxes and keep the same price
            const taxes_to_apply = group[0].tax_id
                .filter((id) => this.pos.taxes_by_id[id].amount_type !== "fixed")
                .map((id) => {
                    return { ...this.pos.taxes_by_id[id], price_include: true };
                });
            const tax_res = this.pos.compute_all(
                taxes_to_apply,
                down_payment_line_price,
                1,
                this.pos.currency.rounding
            );
            let new_price = tax_res["total_excluded"];
            new_price += tax_res.taxes
                .filter((tax) => this.pos.taxes_by_id[tax.id].price_include)
                .reduce((sum, tax) => (sum += tax.amount), 0);
            down_payment_line_to_create.push({
                price: new_price,
                tab: tab,
                tax_ids: group[0].tax_id.filter(
                    (id) => this.pos.taxes_by_id[id].amount_type !== "fixed"
                ),
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
            this.pos.get_order().add_orderline(
                new Orderline(
                    { env: this.env },
                    {
                        pos: this.pos,
                        order: this.pos.get_order(),
                        product: down_payment_product,
                        price: down_payment_line.price,
                        price_type: "automatic",
                        sale_order_origin_id: clickedOrder,
                        down_payment_details: down_payment_line.tab,
                        tax_ids: down_payment_line.tax_ids,
                    }
                )
            );
        }
    }

    async _getSaleOrder(id) {
        const [sale_order] = await this.orm.read(
            "sale.order",
            [id],
            [
                "order_line",
                "partner_id",
                "pricelist_id",
                "fiscal_position_id",
                "amount_total",
                "amount_untaxed",
                "amount_unpaid",
                "picking_ids",
                "partner_shipping_id",
                "partner_invoice_id",
            ]
        );

        const sale_lines = await this._getSOLines(sale_order.order_line);
        sale_order.order_line = sale_lines;

        return sale_order;
    }

    async _getSOLines(ids) {
        const so_lines = await this.orm.call("sale.order.line", "read_converted", [ids]);
        return so_lines;
    }
}

registry.category("pos_screens").add("SaleOrderManagementScreen", SaleOrderManagementScreen);

```

## File: static\src\app\order_management_screen\sale_order_management_screen\sale_order_management_screen.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="pos_sale.SaleOrderManagementScreen">
        <div class="order-management-screen screen h-100 bg-100 overflow-auto" t-att-class="{ 'd-none': !props.isShown }" t-ref="root">
                <div class="rightpane">
                    <div class="flex-container flex-container d-flex flex-column h-100">
                        <SaleOrderManagementControlPanel
                            onSearch.bind="onSearch"
                            onPrevPage.bind="onPrevPage"
                            onNextPage.bind="onNextPage" />
                        <SaleOrderList
                            onClickSaleOrder.bind="onClickSaleOrder"
                            orders="orders"
                            initHighlightedOrder="pos.orderManagement.selectedOrder" />
                    </div>
                </div>
        </div>
    </t>

</templates>

```

## File: static\src\app\order_management_screen\sale_order_row\sale_order_row.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { Component, useState } from "@odoo/owl";
import { useService } from "@web/core/utils/hooks";
import { deserializeDateTime } from "@web/core/l10n/dates";

/**
 * @props {models.Order} order
 * @props columns
 * @emits click-order
 */
export class SaleOrderRow extends Component {
    static template = "pos_sale.SaleOrderRow";

    setup() {
        this.ui = useState(useService("ui"));
    }
    get order() {
        return this.props.order;
    }
    get highlighted() {
        const highlightedOrder = this.props.highlightedOrder;
        return !highlightedOrder
            ? false
            : highlightedOrder.backendId === this.props.order.backendId;
    }

    // Column getters //

    get name() {
        return this.order.name;
    }
    get date() {
        return deserializeDateTime(this.order.date_order).toFormat("yyyy-MM-dd HH:mm a");
    }
    get partner() {
        const partner = this.order.partner_id;
        return partner ? partner[1] : null;
    }
    get total() {
        return this.env.utils.formatCurrency(this.order.amount_total);
    }
    /**
     * Returns true if the order has unpaid amount, but the unpaid amount
     * should not be the same as the total amount.
     * @returns {boolean}
     */
    get showAmountUnpaid() {
        return this.order.amount_total != this.order.amount_unpaid;
    }
    get amountUnpaidRepr() {
        return this.env.utils.formatCurrency(this.order.amount_unpaid);
    }
    get state() {
        const state_mapping = {
            draft: _t("Quotation"),
            sent: _t("Quotation Sent"),
            sale: _t("Sales Order"),
            done: _t("Locked"),
            cancel: _t("Cancelled"),
        };

        return state_mapping[this.order.state];
    }
    get salesman() {
        const salesman = this.order.user_id;
        return salesman ? salesman[1] : null;
    }
}

```

## File: static\src\app\order_management_screen\sale_order_row\sale_order_row.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="pos_sale.SaleOrderRow">
        <div class="order-row"
        t-att-class="{ highlight: highlighted }"
        t-on-click="() => this.props.onClickSaleOrder(props.order)">
            <div class="col name p-2">
                <div t-if="ui.isSmall">Order</div>
                <div><t t-esc="name"/></div>
            </div>
            <div class="col date p-2">
                <div t-if="ui.isSmall">Date</div>
                <div><t t-esc="date"/></div>
            </div>
            <div class="col partner p-2">
                <div t-if="ui.isSmall">Customer</div>
                <div><t t-esc="partner"/></div>
            </div>
            <div class="col salesman p-2">
                <div t-if="ui.isSmall">Salesman</div>
                <div><t t-esc="salesman"/></div>
            </div>
            <div class="col end total p-2">
                <div t-if="ui.isSmall">Total</div>
                <div class="flex-container d-flex flex-column h-100">
                    <div >
                        <t t-esc="total"/>
                    </div>
                    <div t-if="showAmountUnpaid" class="text-muted">
                        (left: <t t-esc="amountUnpaidRepr"/>)
                    </div>
                </div>
            </div>
            <div class="col state p-2">
                <div t-if="ui.isSmall">State</div>
                <div><t t-esc="state"/></div>
            </div>
        </div>
    </t>

</templates>

```

## File: static\src\app\set_sale_order_button\set_sale_order_button.js

```javascript
/** @odoo-module */

import { ProductScreen } from "@point_of_sale/app/screens/product_screen/product_screen";
import { usePos } from "@point_of_sale/app/store/pos_hook";
import { Component } from "@odoo/owl";

export class SetSaleOrderButton extends Component {
    static template = "pos_sale.SetSaleOrderButton";
    setup() {
        this.pos = usePos();
    }
    async click() {
        this.pos.showScreen("SaleOrderManagementScreen");
    }
}

ProductScreen.addControlButton({ component: SetSaleOrderButton });

```

## File: static\src\app\set_sale_order_button\set_sale_order_button.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="pos_sale.SetSaleOrderButton">
        <div class="control-button o_sale_order_button btn btn-light rounded-0 fw-bolder" t-on-click="() => this.click()">
            <i class="fa fa-link me-1" role="img" aria-label="Set Sale Order"
               title="Set Sale Order" /> Quotation/Order
        </div>
    </t>

</templates>

```

## File: static\src\overrides\components\product_screen\product_screen.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="pos_sale.ProductScreen" t-inherit="point_of_sale.ProductScreen" t-inherit-mode="extension">
		<xpath expr="//Orderline" position="inside" >
            <t t-if="line.get_sale_order()">
                <li class="info orderline-sale-order ms-2">
                    <i class="fa fa-shopping-basket me-1" role="img" aria-label="SO" title="SO"/>
                    <t t-esc="line.get_sale_order().name" />
                </li>
                <table t-if="line.get_sale_order().details" class="sale-order-info ms-2">
                    <tr t-foreach="line.get_sale_order()?.details" t-as="soLine" t-key="soLine_index">
                        <td class="text-truncate"><t t-esc="soLine.product_uom_qty"/>x</td>
                        <td class="text-truncate" style="max-width: 275px;"
                            t-esc="soLine.product_name" />
                        <td class="text-truncate">: </td>
                        <td class="text-truncate"><t t-esc="env.utils.formatCurrency(soLine.total)" /> (tax incl.)</td>
                    </tr>
                </table>
            </t>
        </xpath>
    </t>

</templates>

```

## File: static\src\overrides\components\receipt_screen\order_receipt\order_receipt.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="pos_sale.OrderReceipt" t-inherit="point_of_sale.OrderReceipt" t-inherit-mode="extension">
        <xpath expr="//Orderline" position="inside">
            <div class="pos-receipt-left-padding" t-if="line.so_reference">From <t t-esc="line.so_reference"/></div>
            <div class="pos-receipt-left-padding" t-if="line.down_payment_details">
                <table class="sale-order-info ms-2 text-truncate">
                    <tr t-foreach='line.down_payment_details' t-as='line' t-key='line_index'>
                        <td class="text-truncate"><t t-esc="line['product_uom_qty']" />x</td>
                        <td class="text-truncate" style="max-width: 200px;">
                            <t t-esc="line['product_name']" />
                        </td>
                    </tr>
                </table>
            </div>
        </xpath>
    </t>

</templates>

```

## File: static\src\overrides\models\models.js

```javascript
/** @odoo-module */

import { Order, Orderline } from "@point_of_sale/app/store/models";
import { patch } from "@web/core/utils/patch";

patch(Order.prototype, {
    //@override
    select_orderline(orderline) {
        super.select_orderline(...arguments);
        if (orderline && orderline.product.id === this.pos.config.down_payment_product_id[0]) {
            this.pos.numpadMode = "price";
        }
    },
    //@override
    _get_ignored_product_ids_total_discount() {
        const productIds = super._get_ignored_product_ids_total_discount(...arguments);
        productIds.push(this.pos.config.down_payment_product_id[0]);
        return productIds;
    },
});

patch(Orderline.prototype, {
    setup(_defaultObj, options) {
        super.setup(...arguments);
        // It is possible that this orderline is initialized using `init_from_JSON`,
        // meaning, it is loaded from localStorage or from export_for_ui. This means
        // that some fields has already been assigned. Therefore, we only set the options
        // when the original value is falsy.
        this.sale_order_origin_id = this.sale_order_origin_id || options.sale_order_origin_id;
        this.sale_order_line_id = this.sale_order_line_id || options.sale_order_line_id;
        this.down_payment_details = this.down_payment_details || options.down_payment_details;
        this.customerNote = this.customerNote || options.customer_note;
        if (this.sale_order_origin_id && this.sale_order_origin_id.shipping_date) {
            this.order.setShippingDate(this.sale_order_origin_id.shipping_date);
        }
    },
    init_from_JSON(json) {
        super.init_from_JSON(...arguments);
        this.sale_order_origin_id = json.sale_order_origin_id;
        this.sale_order_line_id = json.sale_order_line_id;
        this.down_payment_details =
            json.down_payment_details && JSON.parse(json.down_payment_details);
    },
    export_as_JSON() {
        const json = super.export_as_JSON(...arguments);
        json.sale_order_origin_id = this.sale_order_origin_id;
        json.sale_order_line_id = this.sale_order_line_id;
        json.down_payment_details =
            this.down_payment_details && JSON.stringify(this.down_payment_details);
        return json;
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
        return {
            ...super.getDisplayData(),
            down_payment_details: this.down_payment_details,
            so_reference: this.sale_order_origin_id?.name,
        };
    },
    /**
     * Set quantity based on the give sale order line.
     * @param {'sale.order.line'} saleOrderLine
     */
    setQuantityFromSOL(saleOrderLine) {
        if (this.product.type === "service" && !['sent', 'draft'].includes(this.sale_order_origin_id.state)) {
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
/** @odoo-module */

import { PosStore } from "@point_of_sale/app/store/pos_store";
import { patch } from "@web/core/utils/patch";

patch(PosStore.prototype, {
    async setup(...args) {
        this.orderManagement = { searchString: "", selectedOrder: null };
        return await super.setup(...args);
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
                            <span class="o_stat_value">
                                <field name="sale_order_count" widget="statinfo" nolabel="1" class="mr4" /> Transfered<br/>
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
                            context="{'default_detailed_type': 'service', 'default_taxes_id': False }"
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
        <field name="view_mode">tree,form</field>
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
                <div class="row" t-if="record.pos_sessions_open_count.raw_value" groups="point_of_sale.group_pos_user">
                    <div class="col-8">
                        <a name="%(pos_session_action_from_crm_team)d" type="action">
                            <field name="pos_sessions_open_count"/>
                            <t t-if="record.pos_sessions_open_count.raw_value == 1">Session Running</t>
                            <t t-else="">Sessions Running</t>
                        </a>
                    </div>
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
                            <span class="o_stat_value">
                                <field name="pos_order_count" widget="statinfo" nolabel="1" class="mr4" /> Transfered<br/>
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

