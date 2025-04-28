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
    'name': 'pos_sale',
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
        'point_of_sale.assets': [
            'pos_sale/static/src/css/pos_sale.css',
            'pos_sale/static/src/js/models.js',
            'pos_sale/static/src/js/SetSaleOrderButton.js',
            'pos_sale/static/src/js/OrderManagementScreen/MobileSaleOrderManagementScreen.js',
            'pos_sale/static/src/js/OrderManagementScreen/SaleOrderFetcher.js',
            'pos_sale/static/src/js/OrderManagementScreen/SaleOrderList.js',
            'pos_sale/static/src/js/OrderManagementScreen/SaleOrderManagementControlPanel.js',
            'pos_sale/static/src/js/OrderManagementScreen/SaleOrderManagementScreen.js',
            'pos_sale/static/src/js/OrderManagementScreen/SaleOrderRow.js',
            'pos_sale/static/src/xml/**/*',
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
        ], ['price_total:sum', 'config_id'], ['config_id'])
        rg_results = dict((d['config_id'][0], d['price_total']) for d in data)
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


class PosConfig(models.Model):
    _inherit = 'pos.config'

    crm_team_id = fields.Many2one(
        'crm.team', string="Sales Team", ondelete="set null",
        help="This Point of sale's sales will be related to this Sales Team.")
    down_payment_product_id = fields.Many2one('product.product',
        string="Down Payment Product",
        help="This product will be used as down payment on a sale order.")

```

## File: models\pos_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from functools import lru_cache

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

    @api.depends('pricelist_id.currency_id', 'date_order', 'company_id')
    def _compute_currency_rate(self):
        @lru_cache
        def get_rate(from_currency, to_currency, company, date):
            return self.env['res.currency']._get_conversion_rate(
                from_currency=from_currency,
                to_currency=to_currency,
                company=company,
                date=date,
            )
        for order in self:
            date_order = order.date_order or fields.Datetime.now()
            # date_order is a datetime, but the rates are looked up on a date basis,
            # therefor converting the date_order to a date helps with sharing entries in the lru_cache
            order.currency_rate = get_rate(order.company_id.currency_id, order.pricelist_id.currency_id, order.company_id, date_order.date())

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
                invoice_vals['invoice_payment_term_id'] = sale_orders[0].payment_term_id.id
            if sale_orders[0].partner_invoice_id != sale_orders[0].partner_id:
                invoice_vals['partner_id'] = sale_orders[0].partner_invoice_id.id
        return invoice_vals

    def _create_order_picking(self):
        for line in self.lines.filtered(lambda l: l.product_id == self.config_id.down_payment_product_id and l.qty != 0 and (l.sale_order_origin_id or l.refunded_orderline_id.sale_order_origin_id)):
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

        so_lines = self.lines.mapped('sale_order_line_id')

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
        return super()._create_order_picking()

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
        # NOTE We are not exporting 'sale_order_line_id' because it is being used in any views in the POS App.
        result['down_payment_details'] = bool(orderline.down_payment_details) and orderline.down_payment_details
        result['sale_order_origin_id'] = bool(orderline.sale_order_origin_id) and orderline.sale_order_origin_id.read(fields=['name'])[0]
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
        result['search_params']['domain'] = OR([result['search_params']['domain'], [('id', '=', self.config_id.down_payment_product_id.id)]])
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
            pol.price_unit for pol in self.pos_order_line_ids
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
            if line.order_id.to_ship:
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
        done_states.extend(['paid', 'pos_done', 'invoiced'])
        return done_states

    state = fields.Selection(
        selection_add=[
            ('pos_draft', 'New'),
            ('paid', 'Paid'),
            ('pos_done', 'Posted'),
            ('invoiced', 'Invoiced')
        ],
    )

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
            CASE WHEN pos.state = 'draft' THEN 'pos_draft' WHEN pos.state = 'done' THEN 'pos_done' else pos.state END AS state,
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
            partner.country_id AS country_id,
            partner.industry_id AS industry_id,
            partner.commercial_partner_id AS commercial_partner_id,
            (SUM(p.weight) * l.qty) AS weight,
            (SUM(p.volume) * l.qty) AS volume,
            l.discount AS discount,
            SUM((l.price_unit * l.discount * l.qty / 100.0
                / {self._case_value_or_one('pos.currency_rate')}
                * {self._case_value_or_one('currency_table.rate')}))
            AS discount_amount,
            NULL AS order_id"""

        additional_fields = self._select_additional_fields()
        additional_fields_info = self._fill_pos_fields(additional_fields)
        template = """,
            %s AS %s"""
        for fname, value in additional_fields_info.items():
            select_ += template % (value, fname)
        return select_

    def _fill_pos_fields(self, additional_fields):
        """Hook to fill additional fields for the pos_sale.

        :param values: dictionary of values to fill
        :type values: dict
        """
        return {x: 'NULL' for x in additional_fields}

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
            JOIN {currency_table} ON currency_table.company_id = pos.company_id
            """.format(
            currency_table=self.env['res.currency']._get_query_currency_table(
                {
                    'multi_company': True,
                    'date': {'date_to': fields.Date.today()}
                }),
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
            pos.name,
            pos.date_order,
            pos.partner_id,
            pos.user_id,
            pos.state,
            pos.company_id,
            pos.pricelist_id,
            p.product_tmpl_id,
            partner.country_id,
            partner.industry_id,
            partner.commercial_partner_id,
            u.factor,
            pos.crm_team_id,
            currency_table.rate"""

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

## File: static\src\js\models.js

```javascript
odoo.define('pos_sale.models', function (require) {
    "use strict";

var { Order, Orderline } = require('point_of_sale.models');
const Registries = require('point_of_sale.Registries');


const PosSaleOrder = (Order) => class PosSaleOrder extends Order {
    //@override
    select_orderline(orderline) {
        super.select_orderline(...arguments);
        if (orderline && orderline.product.id === this.pos.config.down_payment_product_id[0]) {
            this.pos.numpadMode = 'price';
        }
    }
    //@override
    _get_ignored_product_ids_total_discount() {
        const productIds = super._get_ignored_product_ids_total_discount(...arguments);
        productIds.push(this.pos.config.down_payment_product_id[0]);
        return productIds;
    }
}
Registries.Model.extend(Order, PosSaleOrder);

const PosSaleOrderline = (Orderline) => class PosSaleOrderline extends Orderline {
  constructor(obj, options) {
      super(...arguments);
      // It is possible that this orderline is initialized using `init_from_JSON`,
      // meaning, it is loaded from localStorage or from export_for_ui. This means
      // that some fields has already been assigned. Therefore, we only set the options
      // when the original value is falsy.
      this.sale_order_origin_id = this.sale_order_origin_id || options.sale_order_origin_id;
      this.sale_order_line_id = this.sale_order_line_id || options.sale_order_line_id;
      this.down_payment_details = this.down_payment_details || options.down_payment_details;
      this.customerNote = this.customerNote || options.customer_note;
  }
  init_from_JSON(json) {
      super.init_from_JSON(...arguments);
      this.sale_order_origin_id = json.sale_order_origin_id;
      this.sale_order_line_id = json.sale_order_line_id;
      this.down_payment_details = json.down_payment_details && JSON.parse(json.down_payment_details);
  }
  export_as_JSON() {
      const json = super.export_as_JSON(...arguments);
      json.sale_order_origin_id = this.sale_order_origin_id;
      json.sale_order_line_id = this.sale_order_line_id;
      json.down_payment_details = this.down_payment_details && JSON.stringify(this.down_payment_details);
      return json;
  }
  get_sale_order(){
      if(this.sale_order_origin_id) {
        let value = {
            'name': this.sale_order_origin_id.name,
            'details': this.down_payment_details || false
        }

        return value;
      }
      return false;
  }
  export_for_printing() {
    var json = super.export_for_printing(...arguments);
    json.down_payment_details =  this.down_payment_details;
    if (this.sale_order_origin_id) {
        json.so_reference = this.sale_order_origin_id.name;
    }
    return json;
  }
  /**
   * Set quantity based on the give sale order line.
   * @param {'sale.order.line'} saleOrderLine
   */
  setQuantityFromSOL(saleOrderLine) {
      if (this.product.type === 'service' && !['sent', 'draft'].includes(this.sale_order_origin_id.state)) {
        this.set_quantity(saleOrderLine.qty_to_invoice);
      } else {
        this.set_quantity(saleOrderLine.product_uom_qty - Math.max(saleOrderLine.qty_delivered, saleOrderLine.qty_invoiced));
      }
  }
}
Registries.Model.extend(Orderline, PosSaleOrderline);

});

```

## File: static\src\js\SetSaleOrderButton.js

```javascript
odoo.define('pos_sale.SetSaleOrderButton', function(require) {
    'use strict';

    const PosComponent = require('point_of_sale.PosComponent');
    const ProductScreen = require('point_of_sale.ProductScreen');
    const { useListener } = require("@web/core/utils/hooks");
    const Registries = require('point_of_sale.Registries');
    const { isConnectionError } = require('point_of_sale.utils');
    const { Gui } = require('point_of_sale.Gui');

    class SetSaleOrderButton extends PosComponent {
        setup() {
            super.setup();
            useListener('click', this.onClick);
        }
        get currentOrder() {
            return this.env.pos.get_order();
        }
        async onClick() {
          try {
              // ping the server, if no error, show the screen
              // Use rpc from services which resolves even when this
              // component is destroyed (removed together with the popup).
              await this.env.services.rpc({
                  model: 'sale.order',
                  method: 'browse',
                  args: [[]],
                  kwargs: { context: this.env.session.user_context },
              });
              // LegacyComponent doesn't work the same way as before.
              // We need to use Gui here to show the screen. This will work
              // because ui methods in Gui is bound to the root component.
              const screen = this.env.isMobile ? 'MobileSaleOrderManagementScreen' : 'SaleOrderManagementScreen';
              Gui.showScreen(screen);
          } catch (error) {
              if (isConnectionError(error)) {
                  this.showPopup('ErrorPopup', {
                      title: this.env._t('Network Error'),
                      body: this.env._t('Cannot access order management screen if offline.'),
                  });
              } else {
                  throw error;
              }
          }
        }
    }
    SetSaleOrderButton.template = 'SetSaleOrderButton';

    ProductScreen.addControlButton({
        component: SetSaleOrderButton,
        condition: function() {
            return true;
        },
    });

    Registries.Component.add(SetSaleOrderButton);

    return SetSaleOrderButton;
});

```

## File: static\src\js\OrderManagementScreen\MobileSaleOrderManagementScreen.js

```javascript
odoo.define('point_of_sale.MobileSaleOrderManagementScreen', function (require) {
    const SaleOrderManagementScreen = require('pos_sale.SaleOrderManagementScreen');
    const Registries = require('point_of_sale.Registries');
    const { useListener } = require("@web/core/utils/hooks");

    const { useState } = owl;

    const MobileSaleOrderManagementScreen = (SaleOrderManagementScreen) => {
        class MobileSaleOrderManagementScreen extends SaleOrderManagementScreen {
            setup() {
                super.setup();
                useListener('click-order', this._onShowDetails)
                this.mobileState = useState({ showDetails: false });
            }
            _onShowDetails() {
                this.mobileState.showDetails = true;
            }
        }
        MobileSaleOrderManagementScreen.template = 'MobileSaleOrderManagementScreen';
        return MobileSaleOrderManagementScreen;
    };

    Registries.Component.addByExtending(MobileSaleOrderManagementScreen, SaleOrderManagementScreen);

    return MobileSaleOrderManagementScreen;
});

```

## File: static\src\js\OrderManagementScreen\SaleOrderFetcher.js

```javascript
odoo.define('pos_sale.SaleOrderFetcher', function (require) {
    'use strict';

    const { Gui } = require('point_of_sale.Gui');
    const { isConnectionError } = require('point_of_sale.utils');

    const { EventBus } = owl;

    class SaleOrderFetcher extends EventBus {
        constructor() {
            super();
            this.currentPage = 1;
            this.ordersToShow = [];
            this.totalCount = 0;
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
        get orderFields(){
          return ['name', 'partner_id', 'amount_total', 'date_order', 'state', 'user_id', 'amount_unpaid'] 
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
            try {
                let limit, offset;
                // Show orders from the backend.
                offset =
                    this.nPerPage +
                    (this.currentPage - 1 - 1) *
                        this.nPerPage;
                limit = this.nPerPage;
                this.ordersToShow = await this._fetch(limit, offset);

                this.trigger('update');
            } catch (error) {
                if (isConnectionError(error)) {
                    Gui.showPopup('ErrorPopup', {
                        title: this.comp.env._t('Network Error'),
                        body: this.comp.env._t('Unable to fetch orders if offline.'),
                    });
                    Gui.setSyncStatus('error');
                } else {
                    throw error;
                }
            }
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
            let domain = [['currency_id', '=', this.comp.env.pos.currency.id]].concat(this.searchDomain || []);
            const saleOrders = await this.rpc({
                model: 'sale.order',
                method: 'search_read',
                args: [domain, this.orderFields, offset, limit],
                context: this.comp.env.session.user_context,
            });

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
        setComponent(comp) {
            this.comp = comp;
            return this;
        }
        setNPerPage(val) {
            this.nPerPage = val;
        }
        setPage(page) {
            this.currentPage = page;
        }

        async rpc() {
            Gui.setSyncStatus('connecting');
            const result = await this.comp.rpc(...arguments);
            Gui.setSyncStatus('connected');
            return result;
        }
    }

    return new SaleOrderFetcher();
});

```

## File: static\src\js\OrderManagementScreen\SaleOrderList.js

```javascript
odoo.define('pos_sale.SaleOrderList', function (require) {
    'use strict';

    const { useListener } = require("@web/core/utils/hooks");
    const PosComponent = require('point_of_sale.PosComponent');
    const Registries = require('point_of_sale.Registries');

    const { useState } = owl;

    /**
     * @props {models.Order} [initHighlightedOrder] initially highligted order
     * @props {Array<models.Order>} orders
     */
    class SaleOrderList extends PosComponent {
        setup() {
            super.setup();
            useListener('click-order', this._onClickOrder);
            this.state = useState({ highlightedOrder: this.props.initHighlightedOrder || null });
        }
        get highlightedOrder() {
            return this.state.highlightedOrder;
        }
        _onClickOrder({ detail: order }) {
            this.state.highlightedOrder = order;
        }
    }
    SaleOrderList.template = 'SaleOrderList';

    Registries.Component.add(SaleOrderList);

    return SaleOrderList;
});

```

## File: static\src\js\OrderManagementScreen\SaleOrderManagementControlPanel.js

```javascript
odoo.define('pos_sale.SaleOrderManagementControlPanel', function (require) {
    'use strict';

    const { useAutofocus, useListener } = require("@web/core/utils/hooks");
    const PosComponent = require('point_of_sale.PosComponent');
    const Registries = require('point_of_sale.Registries');
    const SaleOrderFetcher = require('pos_sale.SaleOrderFetcher');
    const contexts = require('point_of_sale.PosContext');

    const { useState } = owl;

    // NOTE: These are constants so that they are only instantiated once
    // and they can be used efficiently by the OrderManagementControlPanel.
    const VALID_SEARCH_TAGS = new Set(['date', 'customer', 'client', 'name', 'order']);
    const FIELD_MAP = {
        date: 'date_order',
        customer: 'partner_id.display_name',
        client: 'partner_id.display_name',
        name: 'name',
        order: 'name',
    };
    const SEARCH_FIELDS = ['name', 'partner_id.display_name', 'date_order'];

    /**
     * @emits close-screen
     * @emits prev-page
     * @emits next-page
     * @emits search
     */
    class SaleOrderManagementControlPanel extends PosComponent {
        setup() {
            super.setup();
            this.orderManagementContext = useState(contexts.orderManagement);
            useListener('clear-search', this._onClearSearch);
            useAutofocus();

            let currentPartner = this.env.pos.get_order().get_partner();
            if (currentPartner) {
                this.orderManagementContext.searchString = currentPartner.name;
            }
            SaleOrderFetcher.setSearchDomain(this._computeDomain());
        }
        onInputKeydown(event) {
            if (event.key === 'Enter') {
                this.trigger('search', this._computeDomain());
            }
        }
        get showPageControls() {
            return SaleOrderFetcher.lastPage > 1;
        }
        get pageNumber() {
            const currentPage = SaleOrderFetcher.currentPage;
            const lastPage = SaleOrderFetcher.lastPage;
            return isNaN(lastPage) ? '' : `(${currentPage}/${lastPage})`;
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
         *      ['partner_id.display_name', 'ilike', '%Customer 1%'],
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
         *      ['partner_id.display_name', 'ilike', '%Steward%'],
         *      ['date_order', 'ilike', '%2020-05-01%']
         *   ]
         * ```
         */
        _computeDomain() {
            let domain = [['state', '!=', 'cancel'],['invoice_status', '!=', 'invoiced']];
            const input = this.orderManagementContext.searchString.trim();
            if (!input) return domain;

            const searchConditions = this.orderManagementContext.searchString.split(/[,&]\s*/);
            if (searchConditions.length === 1) {
                let cond = searchConditions[0].split(/:\s*/);
                if (cond.length === 1) {
                  domain = domain.concat(Array(this.searchFields.length - 1).fill('|'));
                  domain = domain.concat(this.searchFields.map((field) => [field, 'ilike', `%${cond[0]}%`]));
                  return domain;
                }
            }

            for (let cond of searchConditions) {
                let [tag, value] = cond.split(/:\s*/);
                if (!this.validSearchTags.has(tag)) continue;
                domain.push([this.fieldMap[tag], 'ilike', `%${value}%`]);
            }
            return domain;
        }
        _onClearSearch() {
            this.orderManagementContext.searchString = '';
            this.onInputKeydown({ key: 'Enter' });
        }
    }
    SaleOrderManagementControlPanel.template = 'SaleOrderManagementControlPanel';

    Registries.Component.add(SaleOrderManagementControlPanel);

    return SaleOrderManagementControlPanel;
});

```

## File: static\src\js\OrderManagementScreen\SaleOrderManagementScreen.js

```javascript
odoo.define('pos_sale.SaleOrderManagementScreen', function (require) {
    'use strict';

    const { sprintf } = require('web.utils');
    const { parse } = require('web.field_utils');
    const { _t } = require('@web/core/l10n/translation');
    const { useListener } = require("@web/core/utils/hooks");
    const ControlButtonsMixin = require('point_of_sale.ControlButtonsMixin');
    const NumberBuffer = require('point_of_sale.NumberBuffer');
    const Registries = require('point_of_sale.Registries');
    const SaleOrderFetcher = require('pos_sale.SaleOrderFetcher');
    const IndependentToOrderScreen = require('point_of_sale.IndependentToOrderScreen');
    const contexts = require('point_of_sale.PosContext');
    const utils = require('web.utils');
    const { Orderline } = require('point_of_sale.models');

    const { onMounted, onWillUnmount, useState } = owl;

    /**
     * ID getter to take into account falsy many2one value.
     * @param {[id: number, display_name: string] | false} fieldVal many2one field value
     * @returns {number | false}
     */
    function getId(fieldVal) {
        return fieldVal && fieldVal[0];
    }

    class SaleOrderManagementScreen extends ControlButtonsMixin(IndependentToOrderScreen) {
        setup() {
            super.setup();
            useListener('close-screen', this.close);
            useListener('click-sale-order', this._onClickSaleOrder);
            useListener('next-page', this._onNextPage);
            useListener('prev-page', this._onPrevPage);
            useListener('search', this._onSearch);

            SaleOrderFetcher.setComponent(this);
            this.orderManagementContext = useState(contexts.orderManagement);

            onMounted(this.onMounted);
            onWillUnmount(this.onWillUnmount);
        }
        onMounted() {
            SaleOrderFetcher.on('update', this, this.render);

            // calculate how many can fit in the screen.
            // It is based on the height of the header element.
            // So the result is only accurate if each row is just single line.
            const flexContainer = this.el.querySelector('.flex-container');
            const cpEl = this.el.querySelector('.control-panel');
            const headerEl = this.el.querySelector('.header-row');
            const val = Math.trunc(
                (flexContainer.offsetHeight - cpEl.offsetHeight - headerEl.offsetHeight) /
                    headerEl.offsetHeight
            );
            SaleOrderFetcher.setNPerPage(val);

            // Fetch the order after mounting so that order management screen
            // is shown while fetching.
            setTimeout(() => SaleOrderFetcher.fetch(), 0);
        }
        onWillUnmount() {
            SaleOrderFetcher.off('update', this);
        }
        get selectedPartner() {
            const order = this.orderManagementContext.selectedOrder;
            return order ? order.get_partner() : null;
        }
        get orders() {
            return SaleOrderFetcher.get();
        }
        async _setNumpadMode(event) {
            const { mode } = event.detail;
            this.numpadMode = mode;
            NumberBuffer.reset();
        }
        _onNextPage() {
            SaleOrderFetcher.nextPage();
        }
        _onPrevPage() {
            SaleOrderFetcher.prevPage();
        }
        _onSearch({ detail: domain }) {
            SaleOrderFetcher.setSearchDomain(domain);
            SaleOrderFetcher.setPage(1);
            SaleOrderFetcher.fetch();
        }
        _getSaleOrderOrigin(order) {
            for (const line of order.get_orderlines()) {
                if (line.sale_order_origin_id) {
                    return line.sale_order_origin_id
                }
            }
            return false;
        }
        async _onClickSaleOrder(event) {
            const clickedOrder = event.detail;
            const { confirmed, payload: selectedOption } = await this.showPopup('SelectionPopup',
                {
                    title: this.env._t('What do you want to do?'),
                    list: [{id:"0", label: this.env._t("Apply a down payment"), item: false}, {id:"1", label: this.env._t("Settle the order"), item: true}],
                });

            if(confirmed){
              let currentPOSOrder = this.env.pos.get_order();
              let sale_order = await this._getSaleOrder(clickedOrder.id);
              const currentSaleOrigin = this._getSaleOrderOrigin(currentPOSOrder);
              const currentSaleOriginId = currentSaleOrigin && currentSaleOrigin.id;

              if (currentSaleOriginId) {
                const linkedSO = await this._getSaleOrder(currentSaleOriginId);
                if (
                    getId(linkedSO.partner_id) !== getId(sale_order.partner_id) ||
                    getId(linkedSO.partner_invoice_id) !== getId(sale_order.partner_invoice_id) ||
                    getId(linkedSO.partner_shipping_id) !== getId(sale_order.partner_shipping_id)
                ) {
                    currentPOSOrder = this.env.pos.add_new_order();
                    this.showNotification(this.env._t("A new order has been created."));
                }
              }

              let order_partner = this.env.pos.db.get_partner_by_id(sale_order.partner_id[0])
              if(order_partner){
                currentPOSOrder.set_partner(order_partner);
              } else {
                try {
                    await this.env.pos._loadPartners([sale_order.partner_id[0]]);
                }
                catch (_error){
                    const title = this.env._t('Customer loading error');
                    const body = _.str.sprintf(this.env._t('There was a problem in loading the %s customer.'), sale_order.partner_id[1]);
                    await this.showPopup('ErrorPopup', { title, body });
                }
                currentPOSOrder.set_partner(this.env.pos.db.get_partner_by_id(sale_order.partner_id[0]));
              }

              let orderFiscalPos = sale_order.fiscal_position_id ? this.env.pos.fiscal_positions.find(
                  (position) => position.id === sale_order.fiscal_position_id[0]
              )
              : false;
              if (orderFiscalPos){
                  currentPOSOrder.fiscal_position = orderFiscalPos;
              }
              let orderPricelist = sale_order.pricelist_id ? this.env.pos.pricelists.find(
                  (pricelist) => pricelist.id === sale_order.pricelist_id[0]
              )
              : false;
              if (orderPricelist){
                  currentPOSOrder.set_pricelist(orderPricelist);
              }

              if (selectedOption){
                // settle the order
                let lines = sale_order.order_line;
                let product_to_add_in_pos = lines.filter(line => !this.env.pos.db.get_product_by_id(line.product_id[0])).map(line => line.product_id[0]);
                if (product_to_add_in_pos.length){
                    const { confirmed } = await this.showPopup('ConfirmPopup', {
                        title: this.env._t('Products not available in POS'),
                        body:
                            this.env._t(
                                'Some of the products in your Sale Order are not available in POS, do you want to import them?'
                            ),
                        confirmText: this.env._t('Yes'),
                        cancelText: this.env._t('No'),
                    });
                    if (confirmed){
                        await this.env.pos._addProducts(product_to_add_in_pos);
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
                    let line = lines[i];
                    if (!this.env.pos.db.get_product_by_id(line.product_id[0])){
                        continue;
                    }

                    const line_values = {
                        pos: this.env.pos,
                        order: this.env.pos.get_order(),
                        product: this.env.pos.db.get_product_by_id(line.product_id[0]),
                        description: line.product_id[1],
                        price: line.price_unit,
                        tax_ids: orderFiscalPos ? undefined : line.tax_id,
                        price_automatically_set: true,
                        price_manually_set: false,
                        sale_order_origin_id: clickedOrder,
                        sale_order_line_id: line,
                        customer_note: line.customer_note,
                    };
                    let new_line = Orderline.create({}, line_values);

                    if (
                        new_line.get_product().tracking !== 'none' &&
                        (this.env.pos.picking_type.use_create_lots || this.env.pos.picking_type.use_existing_lots) &&
                        line.lot_names.length > 0
                    ) {
                        // Ask once when `useLoadedLots` is undefined, then reuse it's value on the succeeding lines.
                        const { confirmed } =
                            useLoadedLots === undefined
                                ? await this.showPopup('ConfirmPopup', {
                                      title: this.env._t('SN/Lots Loading'),
                                      body: this.env._t(
                                          'Do you want to load the SN/Lots linked to the Sales Order?'
                                      ),
                                      confirmText: this.env._t('Yes'),
                                      cancelText: this.env._t('No'),
                                  })
                                : { confirmed: useLoadedLots };
                        useLoadedLots = confirmed;
                        if (useLoadedLots) {
                            new_line.setPackLotLines({
                                modifiedPackLotLines: [],
                                newPackLotLines: (line.lot_names || []).map((name) => ({ lot_name: name })),
                            });
                        }
                    }
                    new_line.setQuantityFromSOL(line);
                    new_line.set_unit_price(line.price_unit);
                    new_line.set_discount(line.discount);
                    const product = this.env.pos.db.get_product_by_id(line.product_id[0]);
                    const product_unit = product.get_unit();
                    if (product_unit && !product.get_unit().is_pos_groupable) {
                        //loop for value of quantity
                        let remaining_quantity  = new_line.quantity;
                        while (!utils.float_is_zero(remaining_quantity, 6)) {
                            let splitted_line = Orderline.create({}, line_values);
                            splitted_line.set_quantity(Math.min(remaining_quantity, 1.0), true);
                            splitted_line.set_discount(line.discount);
                            remaining_quantity -= splitted_line.quantity;
                            this.env.pos.get_order().add_orderline(splitted_line);
                        }
                    }
                    else {
                        this.env.pos.get_order().add_orderline(new_line);
                    }
                }
              }
              else {
                // apply a downpayment
                if (this.env.pos.config.down_payment_product_id){

                    let lines = sale_order.order_line;
                    let tab = [];

                    for (let i=0; i<lines.length; i++) {
                        tab[i] = {
                            'product_name': lines[i].product_id[1],
                            'product_uom_qty': lines[i].product_uom_qty,
                            'price_unit': lines[i].price_unit,
                            'total': lines[i].price_total,
                        };
                    }
                    let down_payment_product = this.env.pos.db.get_product_by_id(this.env.pos.config.down_payment_product_id[0]);
                    if (!down_payment_product) {
                        await this.env.pos._addProducts([this.env.pos.config.down_payment_product_id[0]]);
                        down_payment_product = this.env.pos.db.get_product_by_id(this.env.pos.config.down_payment_product_id[0]);
                    }
                    let down_payment_tax = this.env.pos.taxes_by_id[down_payment_product.taxes_id] || false ;
                    let down_payment;
                    if (down_payment_tax) {
                        down_payment = down_payment_tax.price_include ? sale_order.amount_total : sale_order.amount_untaxed;
                    }
                    else{
                        down_payment = sale_order.amount_total;
                    }

                    const { confirmed, payload } = await this.showPopup('NumberPopup', {
                        title: sprintf(this.env._t("Percentage of %s"), this.env.pos.format_currency(sale_order.amount_total)),
                        startingValue: 0,
                    });
                    if (confirmed){
                        down_payment = down_payment * parse.float(payload) / 100;
                    }

                    if (down_payment > sale_order.amount_unpaid) {
                        const errorBody = sprintf(
                            this.env._t("You have tried to charge a down payment of %s but only %s remains to be paid, %s will be applied to the purchase order line."),
                            this.env.pos.format_currency(down_payment),
                            this.env.pos.format_currency(sale_order.amount_unpaid),
                            sale_order.amount_unpaid > 0 ? this.env.pos.format_currency(sale_order.amount_unpaid) : this.env.pos.format_currency(0),
                        );
                        await this.showPopup('ErrorPopup', { title: _t('Error amount too high'), body: errorBody });
                        down_payment = sale_order.amount_unpaid > 0 ? sale_order.amount_unpaid : 0;
                    }

                    let new_line = Orderline.create({}, {
                        pos: this.env.pos,
                        order: this.env.pos.get_order(),
                        product: down_payment_product,
                        price: down_payment,
                        price_automatically_set: true,
                        sale_order_origin_id: clickedOrder,
                        down_payment_details: tab,
                    });
                    new_line.set_unit_price(down_payment);
                    this.env.pos.get_order().add_orderline(new_line);
                }
                else {
                    const title = this.env._t('No down payment product');
                    const body = this.env._t(
                        "It seems that you didn't configure a down payment product in your point of sale.\
                        You can go to your point of sale configuration to choose one."
                    );
                    await this.showPopup('ErrorPopup', { title, body });
                }
              }

              this.close();
            }

        }

        async _getSaleOrder(id) {
            const sale_order = await this.rpc({
                model: 'sale.order',
                method: 'read',
                args: [[id],['order_line', 'partner_id', 'pricelist_id', 'fiscal_position_id', 'amount_total', 'amount_untaxed', 'amount_unpaid', 'partner_shipping_id', 'partner_invoice_id']],
                context: this.env.session.user_context,
            });

            const sale_lines = await this._getSOLines(sale_order[0].order_line);
            sale_order[0].order_line = sale_lines;

            return sale_order[0];
        }

        async _getSOLines(ids) {
          let so_lines = await this.rpc({
              model: 'sale.order.line',
              method: 'read_converted',
              args: [ids],
              context: this.env.session.user_context,
          });
          return so_lines;
        }

    }
    SaleOrderManagementScreen.template = 'SaleOrderManagementScreen';
    SaleOrderManagementScreen.hideOrderSelector = true;

    Registries.Component.add(SaleOrderManagementScreen);

    return SaleOrderManagementScreen;
});

```

## File: static\src\js\OrderManagementScreen\SaleOrderRow.js

```javascript
odoo.define('pos_sale.SaleOrderRow', function (require) {
    'use strict';

    const PosComponent = require('point_of_sale.PosComponent');
    const Registries = require('point_of_sale.Registries');
    const utils = require('web.utils');
    const { deserializeDateTime } = require("@web/core/l10n/dates");

    /**
     * @props {models.Order} order
     * @props columns
     * @emits click-order
     */
    class SaleOrderRow extends PosComponent {
        get order() {
            return this.props.order;
        }
        get highlighted() {
            const highlightedOrder = this.props.highlightedOrder;
            return !highlightedOrder ? false : highlightedOrder.backendId === this.props.order.backendId;
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
            return this.env.pos.format_currency(this.order.amount_total);
        }
        /**
         * Returns true if the order has unpaid amount, but the unpaid amount
         * should not be the same as the total amount.
         * @returns {boolean}
         */
        get showAmountUnpaid() {
            const isFullAmountUnpaid = utils.float_is_zero(Math.abs(this.order.amount_total - this.order.amount_unpaid), this.env.pos.currency.decimal_places);
            return !isFullAmountUnpaid && !utils.float_is_zero(this.order.amount_unpaid, this.env.pos.currency.decimal_places);
        }
        get amountUnpaidRepr() {
            return this.env.pos.format_currency(this.order.amount_unpaid);
        }
        get state() {
            let state_mapping = {
              'draft': this.env._t('Quotation'),
              'sent': this.env._t('Quotation Sent'),
              'sale': this.env._t('Sales Order'),
              'done': this.env._t('Locked'),
              'cancel': this.env._t('Cancelled'),
            };

            return state_mapping[this.order.state];
        }
        get salesman() {
            const salesman = this.order.user_id;
            return salesman ? salesman[1] : null;
        }
    }
    SaleOrderRow.template = 'SaleOrderRow';

    Registries.Component.add(SaleOrderRow);

    return SaleOrderRow;
});

```

## File: static\src\xml\SetSaleOrderButton.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="SetSaleOrderButton" owl="1">
        <div class="control-button o_sale_order_button">
            <i class="fa fa-link" role="img" aria-label="Set Sale Order"
               title="Set Sale Order" /> Quotation/Order
        </div>
    </t>

</templates>

```

## File: static\src\xml\OrderManagementScreen\MobileSaleOrderManagementScreen.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <div t-name="MobileSaleOrderManagementScreen" class="screen-full-width" owl="1">
    <div class="order-management-screen screen" t-att-class="{ oe_hidden: !props.isShown }">
        <div t-if="mobileState.showDetails" class="leftpane">
            <OrderDetails order="orderManagementContext.selectedOrder" />
            <div class="pads">
                <div class="control-buttons">
                    <t t-foreach="controlButtons" t-as="cb" t-key="cb.name">
                        <t t-component="cb.component" t-key="cb.name" />
                    </t>
                </div>
                <div class="subpads">
                    <ActionpadWidget partner="selectedPartner" />
                    <NumpadWidget />
                </div>
            </div>
            <div class="back-to-list" t-on-click="() => { mobileState.showDetails = false; }">
                <span>Back to list</span>
            </div>
        </div>
        <div t-else="" class="rightpane">
            <div class="flex-container">
                <SaleOrderManagementControlPanel />
                <SaleOrderList orders="orders" initHighlightedOrder="orderManagementContext.selectedOrder" />
            </div>
        </div>
    </div>
    </div>

</templates>

```

## File: static\src\xml\OrderManagementScreen\SaleOrderList.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="SaleOrderList" owl="1">
        <div class="orders">
            <div class="header-row" t-att-class="{ oe_hidden: env.isMobile }">
                <div class="col name">Order</div>
                <div class="col date">Date</div>
                <div class="col customer">Customer</div>
                <div class="col salesman">Salesperson</div>
                <div class="col end total">Total</div>
                <div class="col state">State</div>
            </div>
            <div class="order-list">
                <t t-foreach="props.orders" t-as="order" t-key="order.id">
                    <SaleOrderRow order="order" highlightedOrder="highlightedOrder" />
                </t>
            </div>
        </div>
    </t>

</templates>

```

## File: static\src\xml\OrderManagementScreen\SaleOrderManagementControlPanel.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="SaleOrderManagementControlPanel" owl="1">
        <div class="control-panel">
            <div class="item button back" t-on-click="() => this.trigger('close-screen')">
                <i class="fa fa-angle-double-left"></i>
                <span t-if="!env.isMobile"> Back</span>
            </div>
            <div class="item search-box">
                <span class="icon">
                    <i class="fa fa-search" />
                </span>
                <input type="text" t-ref="autofocus" t-model="orderManagementContext.searchString" t-on-keydown="onInputKeydown" placeholder="E.g. customer: Steward, date: 2020-05-09" />
                <span class="clear" t-on-click="() => this.trigger('clear-search')">
                    <i class="fa fa-remove" />
                </span>
            </div>
            <div t-if="showPageControls" class="item">
                <div class="page-controls">
                    <div class="previous" t-on-click="() => this.trigger('prev-page')">
                        <i class="fa fa-fw fa-caret-left" role="img" aria-label="Previous Order List" title="Previous Order List"></i>
                    </div>
                    <div class="next" t-on-click="() => this.trigger('next-page')">
                        <i class="fa fa-fw fa-caret-right" role="img" aria-label="Next Order List" title="Next Order List"></i>
                    </div>
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

## File: static\src\xml\OrderManagementScreen\SaleOrderManagementScreen.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="SaleOrderManagementScreen" owl="1">
        <div class="order-management-screen screen" t-att-class="{ oe_hidden: !props.isShown }">
                <div class="rightpane">
                    <div class="flex-container">
                        <SaleOrderManagementControlPanel />
                        <SaleOrderList orders="orders" initHighlightedOrder="orderManagementContext.selectedOrder" />
                    </div>
                </div>
        </div>
    </t>

</templates>

```

## File: static\src\xml\OrderManagementScreen\SaleOrderRow.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="SaleOrderRow" owl="1">
        <div class="order-row"
        t-att-class="{ highlight: highlighted }"
        t-on-click="() => this.trigger('click-sale-order', props.order)">
            <div class="col name">
                <div t-if="env.isMobile">Order</div>
                <div><t t-esc="name"/></div>
            </div>
            <div class="col date">
                <div t-if="env.isMobile">Date</div>
                <div><t t-esc="date"/></div>
            </div>
            <div class="col partner">
                <div t-if="env.isMobile">Customer</div>
                <div><t t-esc="partner"/></div>
            </div>
            <div class="col salesman">
                <div t-if="env.isMobile">Salesman</div>
                <div><t t-esc="salesman"/></div>
            </div>
            <div class="col end total">
                <div t-if="env.isMobile">Total</div>
                <div class="flex-container">
                    <div class="self-end">
                        <t t-esc="total"/>
                    </div>
                    <div t-if="showAmountUnpaid" class="self-end text-gray">
                        (left: <t t-esc="amountUnpaidRepr"/>)
                    </div>
                </div>
            </div>
            <div class="col state">
                <div t-if="env.isMobile">State</div>
                <div><t t-esc="state"/></div>
            </div>
        </div>
    </t>

</templates>

```

## File: static\src\xml\ProductScreen\Orderline.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="Orderline" t-inherit="point_of_sale.Orderline" t-inherit-mode="extension" owl="1">
        <xpath expr="//ul[hasclass('info-list')]" position="inside">
            <t t-if="props.line.get_sale_order()">
                <li class="info orderline-sale-order">
                    <i class="fa fa-basket" role="img" aria-label="SO" title="SO"/>
                    <t t-esc="props.line.get_sale_order().name" />
                </li>
                <table t-if="props.line.get_sale_order().details" class="sale-order-info">
                    <t t-foreach='props.line.get_sale_order().details' t-as='line' t-key='line_index'>
                        <tr>
                            <td><t t-esc="line['product_uom_qty']"/>x</td>
                            <td style="max-width: 275px;">
                                <t t-esc="line['product_name']" />
                            </td>
                            <td>:</td>
                            <td><t t-esc="env.pos.format_currency(line['total'])" /> (tax incl.)</td>
                        </tr>
                    </t>
                </table>
            </t>
        </xpath>
    </t>

</templates>

```

## File: static\src\xml\ReceiptScreen\OrderReceipt.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="OrderReceipt" t-inherit="point_of_sale.OrderLinesReceipt" t-inherit-mode="extension" owl="1">
        <xpath expr="//t[@t-foreach]" position="inside">
            <div class="pos-receipt-left-padding" t-if="line.so_reference">From <t t-esc="line.so_reference"/></div>
            <div class="pos-receipt-left-padding" t-if="line.down_payment_details">
                <table class="sale-order-info">
                    <tr t-foreach='line.down_payment_details' t-as='line' t-key='line_index'>
                        <td><t t-esc="line['product_uom_qty']" />x</td>
                        <td style="max-width: 200px;">
                            <t t-esc="line['product_name']" />
                        </td>
                    </tr>
                </table>
            </div>
        </xpath>
    </t>

</templates>

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
                                <t t-if="sale_order_line.product_id != down_payment_product">
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
                        attrs="{'invisible': [('sale_order_count', '=', 0)]}" groups="sales_team.group_sale_salesman">
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
            <div id="pos_accounting_section" position="after">
                <h2>Sales</h2>
                <div class="row mt16 o_settings_container">
                    <div class="col-12 col-lg-6 o_setting_box">
                        <div class="o_setting_right_pane">
                            <span class="o_form_label">Sales Team</span>
                            <div class="text-muted">
                                Sales are reported to the following sales team
                            </div>
                            <div class="content-group mt16">
                                <field name="pos_crm_team_id" kanban_view_ref="%(sales_team.crm_team_view_kanban)s" domain="['|', ('company_id', '=', company_id), ('company_id', '=', False)]"/>
                            </div>
                        </div>
                    </div>
                    <div class="col-12 col-lg-6 o_setting_box">
                        <div class="o_setting_right_pane">
                            <span class="o_form_label">Down Payment Product</span>
                            <div class="text-muted">
                                This product will be applied when down payment is made
                            </div>
                            <div class="content-group mt16">
                                <field
                                    name="pos_down_payment_product_id"
                                    domain="[('type', '=', 'service'), '|', ('company_id', '=', company_id), ('company_id', '=', False)]"
                                    context="{'default_detailed_type': 'service', 'default_taxes_id': False }"
                                />
                            </div>
                        </div>
                    </div>
                </div>
            </div>
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
                <field name="crm_team_id" kanban_view_ref="%(sales_team.crm_team_view_kanban)s"/>
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
                <field name="crm_team_id" kanban_view_ref="%(sales_team.crm_team_view_kanban)s"/>
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
                        attrs="{'invisible': [('pos_order_count', '=', 0)]}" groups="point_of_sale.group_pos_user">
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

