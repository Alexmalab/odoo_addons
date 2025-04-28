# Odoo Module: sale_stock

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models
from . import report
from . import wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Sales and Warehouse Management',
    'version': '1.0',
    'category': 'Hidden',
    'summary': 'Quotation, Sales Orders, Delivery & Invoicing Control',
    'description': """
Manage sales quotations and orders
==================================

This module makes the link between the sales and warehouses management applications.

Preferences
-----------
* Shipping: Choice of delivery at once or partial delivery
* Invoicing: choose how invoices will be paid
* Incoterms: International Commercial terms

""",
    'depends': ['sale', 'stock_account'],
    'data': [
        'security/sale_stock_security.xml',
        'security/ir.model.access.csv',

        'views/sale_order_views.xml',
        'views/stock_route_views.xml',
        'views/res_config_settings_views.xml',
        'views/sale_stock_portal_template.xml',
        'views/stock_lot_views.xml',
        'views/res_users_views.xml',
        'views/stock_picking_views.xml',

        'report/sale_order_report_templates.xml',
        'report/stock_report_deliveryslip.xml',

        'data/mail_templates.xml',
        'data/sale_stock_data.xml',

        'wizard/stock_rules_report_views.xml',
        'wizard/sale_order_cancel_views.xml',
    ],
    'demo': ['data/sale_order_demo.xml'],
    'installable': True,
    'auto_install': True,
    'assets': {
        'web.assets_backend': [
            'sale_stock/static/src/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\portal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from werkzeug.exceptions import NotFound

from odoo import exceptions, SUPERUSER_ID
from odoo.addons.sale.controllers.portal import CustomerPortal
from odoo.http import request, route
from odoo.tools import consteq


class SaleStockPortal(CustomerPortal):

    def _stock_picking_check_access(self, picking_id, access_token=None):
        picking = request.env['stock.picking'].browse([picking_id])
        picking_sudo = picking.sudo()
        try:
            picking.check_access('read')
        except exceptions.AccessError:
            if not access_token or not consteq(picking_sudo.sale_id.access_token, access_token):
                raise
        return picking_sudo

    @route(['/my/picking/pdf/<int:picking_id>'], type='http', auth="public", website=True)
    def portal_my_picking_report(self, picking_id, access_token=None, **kw):
        """ Print delivery slip for customer, using either access rights or access token
        to be sure customer has access """
        try:
            picking_sudo = self._stock_picking_check_access(picking_id, access_token=access_token)
        except (exceptions.AccessError, exceptions.MissingError):
            return NotFound()

        # print report with sudo, since it require access to product, taxes, payment term etc.. and portal does not have those access rights.
        pdf = request.env['ir.actions.report'].sudo()._render_qweb_pdf('stock.action_report_delivery', [picking_sudo.id])[0]
        pdfhttpheaders = [
            ('Content-Type', 'application/pdf'),
            ('Content-Length', len(pdf)),
        ]
        return request.make_response(pdf, headers=pdfhttpheaders)

    @route(['/my/picking/return/pdf/<int:picking_id>'], type='http', auth="public", website=True)
    def portal_my_picking_return_report(self, picking_id, access_token=None, **kw):
        """ Print return label for customer, using either access rights or access token
        to be sure customer has access """
        try:
            picking_sudo = self._stock_picking_check_access(picking_id, access_token=access_token)
        except (exceptions.AccessError, exceptions.MissingError):
            return NotFound()

        pdf = \
        request.env['ir.actions.report'].sudo()._render_qweb_pdf('stock.return_label_report', [picking_sudo.id])[0]
        pdfhttpheaders = [
            ('Content-Type', 'application/pdf'),
            ('Content-Length', len(pdf)),
        ]
        return request.make_response(pdf, headers=pdfhttpheaders)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import portal

```

## File: data\mail_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <template id="exception_on_so">
            <div>
                Exception(s) occurred on the sale order(s):
                <t t-foreach="sale_order_ids" t-as="sale_order">
                    <a href="#" data-oe-model="sale.order" t-att-data-oe-id="sale_order.id"><t t-esc="sale_order.name"/></a>.
                </t>
                Manual actions may be needed.
                <div class="mt16">
                    <p>Exception(s):</p>
                    <ul t-foreach="order_exceptions" t-as="exception">
                        <li>
                            <t t-set="order_line" t-value="exception[0]"/>
                            <t t-set="new_qty" t-value="exception[1][0]"/>
                            <t t-set="old_qty" t-value="exception[1][1]"/>
                                <a href="#" data-oe-model="sale.order" t-att-data-oe-id="order_line.order_id.id"><t t-esc="order_line.order_id.name"/></a>:
                                <t t-esc="new_qty"/> <t t-esc="order_line.product_uom.name"/> of <t t-esc="order_line.product_id.name"/>
                                <t t-if="cancel">
                                    cancelled
                                </t>
                                <t t-if="not cancel">
                                    ordered instead of <t t-esc="old_qty"/> <t t-esc="order_line.product_uom.name"/>
                                </t>
                          </li>
                    </ul>
                </div>
                <div class="mt16" t-if="not cancel and impacted_pickings">
                    <p>Impacted Transfer(s):</p>
                    <ul t-foreach="impacted_pickings" t-as="picking">
                        <li><a href="#" data-oe-model="stock.picking" t-att-data-oe-id="picking.id"><t t-esc="picking.name"/></a></li>
                    </ul>
                </div>
            </div>
        </template>

        <template id="exception_on_picking">
            <div>
                Exception(s) occurred on the picking:
                <a href="#" data-oe-model="stock.picking" t-att-data-oe-id="origin_picking.id"><t t-esc="origin_picking.name"/></a>.
                Manual actions may be needed.
                <div class="mt16">
                    <p>Exception(s):</p>
                    <ul t-foreach="moves_information" t-as="exception">
                        <t t-set="move" t-value="exception[0]"/>
                        <t t-set="new_qty" t-value="exception[1][0]"/>
                        <t t-set="old_qty" t-value="exception[1][1]"/>
                        <li>
                        <t t-esc="new_qty"/> <t t-esc="move.product_uom.name"/>
                        of <t t-esc="move.product_id.display_name"/> processed instead of <t t-esc="old_qty"/> <t t-esc="move.product_uom.name"/></li>
                    </ul>
                </div>
            </div>
        </template>
    </data>
</odoo>

```

## File: data\sale_order_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <record id="sale.sale_order_1" model="sale.order">
            <field name="warehouse_id" ref="stock.warehouse0"/>
        </record>

        <record id="sale.sale_order_2" model="sale.order">
            <field name="warehouse_id" ref="stock.warehouse0"/>
        </record>

        <record id="sale.sale_order_3" model="sale.order">
            <field name="warehouse_id" ref="stock.warehouse0"/>
        </record>

        <record id="sale.sale_order_5" model="sale.order">
            <field name="warehouse_id" ref="stock.warehouse0"/>
        </record>

        <record id="sale.sale_order_6" model="sale.order">
            <field name="warehouse_id" ref="stock.warehouse0"/>
        </record>

        <record id="sale.sale_order_8" model="sale.order">
            <field name="warehouse_id" ref="stock.warehouse0"/>
        </record>

        <!-- Create some new sale orders to have more data for the forecast report -->
        <record id="sale_order_19" model="sale.order">
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="partner_invoice_id" ref="base.res_partner_address_25"/>
            <field name="partner_shipping_id" ref="base.res_partner_address_25"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="sale.utm_source_sale_order_0"/>
            <field name="date_order" eval="(datetime.now() + relativedelta(days=1)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="warehouse_id" ref="stock.warehouse0"/>
        </record>

        <record id="sale_order_line_42" model="sale.order.line">
            <field name="order_id" ref="sale_order_19"/>
            <field name="product_id" ref="product.product_product_25"/>
            <field name="product_uom_qty">5</field>
            <field name="price_unit">295.00</field>
        </record>

        <record id="sale_order_line_43" model="sale.order.line">
            <field name="order_id" ref="sale_order_19"/>
            <field name="product_id" ref="product.product_product_10"/>
            <field name="product_uom_qty">5</field>
            <field name="price_unit">140.00</field>
        </record>

        <record id="sale_order_20" model="sale.order">
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="partner_invoice_id" ref="base.res_partner_address_25"/>
            <field name="partner_shipping_id" ref="base.res_partner_address_25"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="sale.utm_source_sale_order_0"/>
            <field name="date_order" eval="(datetime.now() + relativedelta(days=2)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="warehouse_id" ref="stock.warehouse0"/>
        </record>

        <record id="sale_order_line_44" model="sale.order.line">
            <field name="order_id" ref="sale_order_20"/>
            <field name="product_id" ref="product.product_product_25"/>
            <field name="product_uom_qty">5</field>
            <field name="price_unit">295.00</field>
        </record>

        <record id="sale_order_21" model="sale.order">
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="partner_invoice_id" ref="base.res_partner_address_25"/>
            <field name="partner_shipping_id" ref="base.res_partner_address_25"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="sale.utm_source_sale_order_0"/>
            <field name="date_order" eval="(datetime.now() + relativedelta(days=3)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="warehouse_id" ref="stock.warehouse0"/>
        </record>

        <record id="sale_order_line_45" model="sale.order.line">
            <field name="order_id" ref="sale_order_21"/>
            <field name="product_id" ref="product.product_product_25"/>
            <field name="product_uom_qty">10</field>
            <field name="price_unit">295.00</field>
        </record>

        <record id="sale_order_line_46" model="sale.order.line">
            <field name="order_id" ref="sale_order_21"/>
            <field name="product_id" ref="product.product_product_10"/>
            <field name="product_uom_qty">10</field>
            <field name="price_unit">140.00</field>
        </record>

        <record id="sale_order_22" model="sale.order">
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="partner_invoice_id" ref="base.res_partner_address_25"/>
            <field name="partner_shipping_id" ref="base.res_partner_address_25"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="sale.utm_source_sale_order_0"/>
            <field name="date_order" eval="(datetime.now() + relativedelta(days=3)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="warehouse_id" ref="stock.warehouse0"/>
        </record>

        <record id="sale_order_line_47" model="sale.order.line">
            <field name="order_id" ref="sale_order_22"/>
            <field name="product_id" ref="product.product_product_5"/>
            <field name="product_uom_qty">10</field>
            <field name="price_unit">199.00</field>
        </record>


        <function model="sale.order" name="action_confirm" eval="[[
            ref('sale_order_19'),
            ref('sale_order_20'),
            ref('sale_order_21'),
            ref('sale_order_22'),
        ]]"/>

        <!-- Change date of those sale orders' delivery -->
        <function model="stock.picking" name="write">
            <value model="stock.picking" search="[('sale_id', '=', obj().env.ref('sale_stock.sale_order_19').id)]"/>
            <value eval="{'scheduled_date': (datetime.now() + relativedelta(days=1)).strftime('%Y-%m-%d %H:%M:%S')}"/>
        </function>
        <function model="stock.picking" name="write">
            <value model="stock.picking" search="[('sale_id', '=', obj().env.ref('sale_stock.sale_order_20').id)]"/>
            <value eval="{'scheduled_date': (datetime.now() + relativedelta(days=2)).strftime('%Y-%m-%d %H:%M:%S')}"/>
        </function>
        <function model="stock.picking" name="write">
            <value model="stock.picking" search="[('sale_id', '=', obj().env.ref('sale_stock.sale_order_21').id)]"/>
            <value eval="{'scheduled_date': (datetime.now() + relativedelta(days=3)).strftime('%Y-%m-%d %H:%M:%S')}"/>
        </function>


        <!-- Inventory for new Customizable Desks -->
        <record id="sale.product_product_4f" model="product.product">
            <field name="is_storable" eval="True"/>
        </record>
        <record id="stock_inventory_7e" model="stock.quant">
            <field name="product_id" ref="sale.product_product_4e"/>
            <field name="inventory_quantity">65.0</field>
            <field name="location_id" model="stock.location" eval="obj().env.ref('stock.warehouse0').lot_stock_id.id"/>
        </record>
        <record id="stock_inventory_7f" model="stock.quant">
            <field name="product_id" ref="sale.product_product_4f"/>
            <field name="inventory_quantity">70.0</field>
            <field name="location_id" model="stock.location" eval="obj().env.ref('stock.warehouse0').lot_stock_id.id"/>
        </record>
        <function model="stock.quant" name="action_apply_inventory">
            <function eval="[[('id', 'in', (ref('stock_inventory_7e'),
                                            ref('stock_inventory_7f'),
                                            ))]]" model="stock.quant" name="search"/>
        </function>
    </data>
</odoo>

```

## File: data\sale_stock_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
    	<record id="stock.route_warehouse0_mto" model="stock.route">
            <field name="sale_selectable">True</field>
        </record>
    </data>
</odoo>

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict

from odoo import models, api
from odoo.tools import float_is_zero, float_compare
from odoo.tools.misc import formatLang


class AccountMove(models.Model):
    _inherit = 'account.move'

    def _stock_account_get_last_step_stock_moves(self):
        """ Overridden from stock_account.
        Returns the stock moves associated to this invoice."""
        rslt = super(AccountMove, self)._stock_account_get_last_step_stock_moves()
        for invoice in self:
            if invoice.move_type not in ['out_invoice', 'out_refund']:
                continue
            if (invoice.move_type == 'out_invoice' or (
                invoice.move_type == 'out_refund' and any(invoice.invoice_line_ids.sale_line_ids.mapped('is_downpayment')))
            ):
                rslt += invoice.mapped('invoice_line_ids.sale_line_ids.move_ids').filtered(lambda x: x.state == 'done' and x.location_dest_id.usage == 'customer')
            else:
                rslt += invoice.mapped('reversed_entry_id.invoice_line_ids.sale_line_ids.move_ids').filtered(lambda x: x.state == 'done' and x.location_id.usage == 'customer')
                # Add refunds generated from the SO
                rslt += invoice.mapped('invoice_line_ids.sale_line_ids.move_ids').filtered(lambda x: x.state == 'done' and x.location_id.usage == 'customer')
        return rslt

    def _get_invoiced_lot_values(self):
        """ Get and prepare data to show a table of invoiced lot on the invoice's report. """
        self.ensure_one()

        res = super(AccountMove, self)._get_invoiced_lot_values()

        if self.state == 'draft' or not self.invoice_date or self.move_type not in ('out_invoice', 'out_refund'):
            return res

        current_invoice_amls = self.invoice_line_ids.filtered(lambda aml: aml.display_type == 'product' and aml.product_id and aml.product_id.type == 'consu' and aml.quantity)
        all_invoices_amls = current_invoice_amls.sale_line_ids.invoice_lines.filtered(lambda aml: aml.move_id.state == 'posted').sorted(lambda aml: (aml.date, aml.move_name, aml.id))
        index = all_invoices_amls.ids.index(current_invoice_amls[:1].id) if current_invoice_amls[:1] in all_invoices_amls else 0
        previous_amls = all_invoices_amls[:index]
        invoiced_qties = current_invoice_amls._get_invoiced_qty_per_product()
        invoiced_products = invoiced_qties.keys()

        if self.move_type == 'out_invoice':
            # filter out the invoices that have been fully refund and re-invoice otherwise, the quantities would be
            # consumed by the reversed invoice and won't be print on the new draft invoice
            previous_amls = previous_amls.filtered(lambda aml: aml.move_id.payment_state != 'reversed')

        previous_qties_invoiced = previous_amls._get_invoiced_qty_per_product()

        if self.move_type == 'out_refund':
            # we swap the sign because it's a refund, and it would print negative number otherwise
            for p in previous_qties_invoiced:
                previous_qties_invoiced[p] = -previous_qties_invoiced[p]
            for p in invoiced_qties:
                invoiced_qties[p] = -invoiced_qties[p]

        qties_per_lot = defaultdict(float)
        previous_qties_delivered = defaultdict(float)
        stock_move_lines = current_invoice_amls.sale_line_ids.move_ids.move_line_ids.filtered(lambda sml: sml.state == 'done' and sml.lot_id).sorted(lambda sml: (sml.date, sml.id))
        for sml in stock_move_lines:
            if sml.product_id not in invoiced_products or not sml._should_show_lot_in_invoice():
                continue
            product = sml.product_id
            product_uom = product.uom_id
            quantity = sml.product_uom_id._compute_quantity(sml.quantity, product_uom)

            # is it a stock return considering the document type (should it be it thought of as positively or negatively?)
            is_stock_return = (
                    self.move_type == 'out_invoice' and (sml.location_id.usage, sml.location_dest_id.usage) == ('customer', 'internal')
                    or
                    self.move_type == 'out_refund' and (sml.location_id.usage, sml.location_dest_id.usage) == ('internal', 'customer')
            )
            if is_stock_return:
                returned_qty = min(qties_per_lot[sml.lot_id], quantity)
                qties_per_lot[sml.lot_id] -= returned_qty
                quantity = returned_qty - quantity

            previous_qty_invoiced = previous_qties_invoiced[product]
            previous_qty_delivered = previous_qties_delivered[product]
            # If we return more than currently delivered (i.e., quantity < 0), we remove the surplus
            # from the previously delivered (and quantity becomes zero). If it's a delivery, we first
            # try to reach the previous_qty_invoiced
            if float_compare(quantity, 0, precision_rounding=product_uom.rounding) < 0 or \
                    float_compare(previous_qty_delivered, previous_qty_invoiced, precision_rounding=product_uom.rounding) < 0:
                previously_done = quantity if is_stock_return else min(previous_qty_invoiced - previous_qty_delivered, quantity)
                previous_qties_delivered[product] += previously_done
                quantity -= previously_done

            qties_per_lot[sml.lot_id] += quantity

        for lot, qty in qties_per_lot.items():
            # access the lot as a superuser in order to avoid an error
            # when a user prints an invoice without having the stock access
            lot = lot.sudo()
            if float_is_zero(invoiced_qties[lot.product_id], precision_rounding=lot.product_uom_id.rounding) \
                    or float_compare(qty, 0, precision_rounding=lot.product_uom_id.rounding) <= 0:
                continue
            invoiced_lot_qty = min(qty, invoiced_qties[lot.product_id])
            invoiced_qties[lot.product_id] -= invoiced_lot_qty
            res.append({
                'product_name': lot.product_id.display_name,
                'quantity': formatLang(self.env, invoiced_lot_qty, dp='Product Unit of Measure'),
                'uom_name': lot.product_uom_id.name,
                'lot_name': lot.name,
                # The lot id is needed by localizations to inherit the method and add custom fields on the invoice's report.
                'lot_id': lot.id,
            })

        return res

    @api.depends('line_ids.sale_line_ids.order_id')
    def _compute_delivery_date(self):
        # EXTENDS 'account'
        super()._compute_delivery_date()
        for move in self:
            sale_order_effective_date = list(filter(None, move.line_ids.sale_line_ids.order_id.mapped('effective_date')))
            effective_date_res = max(sale_order_effective_date) if sale_order_effective_date else False
            # if multiple sale order we take the bigger effective_date
            if effective_date_res:
                move.delivery_date = effective_date_res

    @api.depends('line_ids.sale_line_ids.order_id')
    def _compute_incoterm_location(self):
        super()._compute_incoterm_location()
        for move in self:
            sale_locations = move.line_ids.sale_line_ids.order_id.mapped('incoterm_location')
            incoterm_res = next((incoterm for incoterm in sale_locations if incoterm), False)
            # if multiple purchase order we take an incoterm that is not false
            if incoterm_res:
                move.incoterm_location = incoterm_res

    def _get_anglo_saxon_price_ctx(self):
        ctx = super()._get_anglo_saxon_price_ctx()
        move_is_downpayment = self.invoice_line_ids.filtered(
            lambda line: any(line.sale_line_ids.mapped("is_downpayment"))
        )
        return dict(ctx, move_is_downpayment=move_is_downpayment)


class AccountMoveLine(models.Model):
    _inherit = "account.move.line"

    def _sale_can_be_reinvoice(self):
        self.ensure_one()
        return self.move_type != 'entry' and self.display_type != 'cogs' and super(AccountMoveLine, self)._sale_can_be_reinvoice()

    def _stock_account_get_anglo_saxon_price_unit(self):
        self.ensure_one()
        price_unit = super(AccountMoveLine, self)._stock_account_get_anglo_saxon_price_unit()

        so_line = self.sale_line_ids and self.sale_line_ids[-1] or False
        move_is_downpayment = self.env.context.get("move_is_downpayment")
        if move_is_downpayment is None:
            move_is_downpayment = self.move_id.invoice_line_ids.filtered(
            lambda line: any(line.sale_line_ids.mapped("is_downpayment"))
        )
        if so_line:
            is_line_reversing = False
            if self.move_id.move_type == 'out_refund' and not move_is_downpayment:
                is_line_reversing = True
            qty_to_invoice = self.product_uom_id._compute_quantity(self.quantity, self.product_id.uom_id)
            if self.move_id.move_type == 'out_refund' and move_is_downpayment:
                qty_to_invoice = -qty_to_invoice
            account_moves = so_line.invoice_lines.move_id.filtered(lambda m: m.state == 'posted' and bool(m.reversed_entry_id) == is_line_reversing)

            posted_cogs = self.env['account.move.line'].search([
                ('move_id', 'in', account_moves.ids),
                ('display_type', '=', 'cogs'),
                ('product_id', '=', self.product_id.id),
                ('balance', '>', 0),
            ])
            posted_cogs = posted_cogs.filtered(lambda l: so_line in l.cogs_origin_id.sale_line_ids)
            qty_invoiced = 0
            product_uom = self.product_id.uom_id
            for line in posted_cogs:
                if float_compare(line.quantity, 0, precision_rounding=product_uom.rounding) and line.move_id.move_type == 'out_refund' and any(line.move_id.invoice_line_ids.sale_line_ids.mapped('is_downpayment')):
                    qty_invoiced += line.product_uom_id._compute_quantity(abs(line.quantity), line.product_id.uom_id)
                else:
                    qty_invoiced += line.product_uom_id._compute_quantity(line.quantity, line.product_id.uom_id)
            value_invoiced = sum(posted_cogs.mapped('balance'))
            reversal_moves = self.env['account.move']._search([('reversed_entry_id', 'in', posted_cogs.move_id.ids)])
            reversal_cogs = self.env['account.move.line'].search([
                ('move_id', 'in', reversal_moves),
                ('display_type', '=', 'cogs'),
                ('product_id', '=', self.product_id.id),
                ('balance', '>', 0)
            ])
            for line in reversal_cogs:
                if float_compare(line.quantity, 0, precision_rounding=product_uom.rounding) and line.move_id.move_type == 'out_refund' and any(line.move_id.invoice_line_ids.sale_line_ids.mapped('is_downpayment')):
                    qty_invoiced -= line.product_uom_id._compute_quantity(abs(line.quantity), line.product_id.uom_id)
                else:
                    qty_invoiced -= line.product_uom_id._compute_quantity(line.quantity, line.product_id.uom_id)
            value_invoiced -= sum(reversal_cogs.mapped('balance'))

            product = self.product_id.with_company(self.company_id).with_context(value_invoiced=value_invoiced)
            average_price_unit = product._compute_average_price(qty_invoiced, qty_to_invoice, so_line.move_ids, is_returned=is_line_reversing)
            price_unit = self.product_id.uom_id.with_company(self.company_id)._compute_price(average_price_unit, self.product_uom_id)
        return price_unit

```

## File: models\product_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class ProductTemplate(models.Model):
    _inherit = 'product.template'

    @api.depends('type')
    def _compute_expense_policy(self):
        super()._compute_expense_policy()
        self.filtered(lambda t: t.is_storable).expense_policy = 'no'

    @api.depends('type')
    def _compute_service_type(self):
        super()._compute_service_type()
        self.filtered(lambda t: t.is_storable).service_type = 'manual'

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class company(models.Model):
    _inherit = 'res.company'

    security_lead = fields.Float(
        'Sales Safety Days', default=0.0, required=True,
        help="Margin of error for dates promised to customers. "
             "Products will be scheduled for procurement and delivery "
             "that many days earlier than the actual promised date, to "
             "cope with unexpected delays in the supply chain.")

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    security_lead = fields.Float(related='company_id.security_lead', string="Security Lead Time", readonly=False)
    use_security_lead = fields.Boolean(
        string="Security Lead Time for Sales",
        config_parameter='sale_stock.use_security_lead',
        help="Margin of error for dates promised to customers. Products will be scheduled for delivery that many days earlier than the actual promised date, to cope with unexpected delays in the supply chain.")
    default_picking_policy = fields.Selection([
        ('direct', 'Ship products as soon as available, with back orders'),
        ('one', 'Ship all products at once')
        ], "Picking Policy", default='direct', default_model="sale.order", required=True)

    @api.onchange('use_security_lead')
    def _onchange_use_security_lead(self):
        if not self.use_security_lead:
            self.security_lead = 0.0

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields


class Users(models.Model):
    _inherit = ['res.users']

    property_warehouse_id = fields.Many2one('stock.warehouse', string='Default Warehouse', company_dependent=True, check_company=True)

    def _get_default_warehouse_id(self):
        if self.property_warehouse_id:
            return self.property_warehouse_id
        return super()._get_default_warehouse_id()

    @property
    def SELF_READABLE_FIELDS(self):
        return super().SELF_READABLE_FIELDS + ['property_warehouse_id']

    @property
    def SELF_WRITEABLE_FIELDS(self):
        return super().SELF_WRITEABLE_FIELDS + ['property_warehouse_id']

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json
import logging

from odoo import api, fields, models, _
from odoo.exceptions import UserError
from odoo.tools import float_compare

_logger = logging.getLogger(__name__)


class SaleOrder(models.Model):
    _inherit = "sale.order"

    incoterm = fields.Many2one(
        'account.incoterms', 'Incoterm',
        help="International Commercial Terms are a series of predefined commercial terms used in international transactions.")
    incoterm_location = fields.Char(string='Incoterm Location')
    picking_policy = fields.Selection([
        ('direct', 'As soon as possible'),
        ('one', 'When all products are ready')],
        string='Shipping Policy', required=True, default='direct',
        help="If you deliver all products at once, the delivery order will be scheduled based on the greatest "
        "product lead time. Otherwise, it will be based on the shortest.")
    warehouse_id = fields.Many2one(
        'stock.warehouse', string='Warehouse',
        compute='_compute_warehouse_id', store=True, readonly=False, precompute=True,
        check_company=True)
    picking_ids = fields.One2many('stock.picking', 'sale_id', string='Transfers')
    delivery_count = fields.Integer(string='Delivery Orders', compute='_compute_picking_ids')
    delivery_status = fields.Selection([
        ('pending', 'Not Delivered'),
        ('started', 'Started'),
        ('partial', 'Partially Delivered'),
        ('full', 'Fully Delivered'),
    ], string='Delivery Status', compute='_compute_delivery_status', store=True,
       help="Blue: Not Delivered/Started\n\
            Orange: Partially Delivered\n\
            Green: Fully Delivered")
    procurement_group_id = fields.Many2one('procurement.group', 'Procurement Group', copy=False)
    effective_date = fields.Datetime("Effective Date", compute='_compute_effective_date', store=True, help="Completion date of the first delivery order.")
    expected_date = fields.Datetime( help="Delivery date you can promise to the customer, computed from the minimum lead time of "
                                          "the order lines in case of Service products. In case of shipping, the shipping policy of "
                                          "the order will be taken into account to either use the minimum or maximum lead time of "
                                          "the order lines.")
    json_popover = fields.Char('JSON data for the popover widget', compute='_compute_json_popover')
    show_json_popover = fields.Boolean('Has late picking', compute='_compute_json_popover')

    def _init_column(self, column_name):
        """ Ensure the default warehouse_id is correctly assigned

        At column initialization, the ir.model.fields for res.users.property_warehouse_id isn't created,
        which means trying to read the property field to get the default value will crash.
        We therefore enforce the default here, without going through
        the default function on the warehouse_id field.
        """
        if column_name != "warehouse_id":
            return super(SaleOrder, self)._init_column(column_name)
        field = self._fields[column_name]
        default = self.env['stock.warehouse'].search([('company_id', '=', self.env.company.id)], limit=1)
        value = field.convert_to_write(default, self)
        value = field.convert_to_column_insert(value, self)
        if value is not None:
            _logger.debug("Table '%s': setting default value of new column %s to %r",
                self._table, column_name, value)
            query = f'UPDATE "{self._table}" SET "{column_name}" = %s WHERE "{column_name}" IS NULL'
            self._cr.execute(query, (value,))

    @api.depends('picking_ids.date_done')
    def _compute_effective_date(self):
        for order in self:
            pickings = order.picking_ids.filtered(lambda x: x.state == 'done' and x.location_dest_id.usage == 'customer')
            dates_list = [date for date in pickings.mapped('date_done') if date]
            order.effective_date = min(dates_list, default=False)

    @api.depends('picking_ids', 'picking_ids.state')
    def _compute_delivery_status(self):
        for order in self:
            if not order.picking_ids or all(p.state == 'cancel' for p in order.picking_ids):
                order.delivery_status = False
            elif all(p.state in ['done', 'cancel'] for p in order.picking_ids):
                order.delivery_status = 'full'
            elif any(p.state == 'done' for p in order.picking_ids) and any(
                    l.qty_delivered for l in order.order_line):
                order.delivery_status = 'partial'
            elif any(p.state == 'done' for p in order.picking_ids):
                order.delivery_status = 'started'
            else:
                order.delivery_status = 'pending'

    @api.depends('picking_policy')
    def _compute_expected_date(self):
        super(SaleOrder, self)._compute_expected_date()

    def _select_expected_date(self, expected_dates):
        if self.picking_policy == "direct":
            return super()._select_expected_date(expected_dates)
        return max(expected_dates)

    @api.constrains('warehouse_id', 'state', 'order_line')
    def _check_warehouse(self):
        """ Ensure that the warehouse is set in case of storable products """
        orders_without_wh = self.filtered(lambda order: order.state not in ('draft', 'cancel') and not order.warehouse_id)
        company_ids_with_wh = {group['company_id'][0] for group in self.env['stock.warehouse'].read_group(domain=[('company_id', 'in', orders_without_wh.mapped('company_id').ids)], fields=['id:recordset'], groupby=['company_id'])} if orders_without_wh else {}
        other_company = set()
        for order_line in orders_without_wh.order_line:
            if order_line.product_id.type != 'consu':
                continue
            if order_line.route_id.company_id and order_line.route_id.company_id != order_line.company_id:
                other_company.add(order_line.route_id.company_id.id)
                continue
            if order_line.order_id.company_id.id in company_ids_with_wh:
                raise UserError(_('You must set a warehouse on your sale order to proceed.'))
            self.env['stock.warehouse'].with_company(order_line.order_id.company_id)._warehouse_redirect_warning()
        other_company_warehouses = self.env['stock.warehouse'].search([('company_id', 'in', list(other_company))])
        if any(c not in other_company_warehouses.company_id.ids for c in other_company):
            raise UserError(_("You must have a warehouse for line using a delivery in different company."))

    def write(self, values):
        if values.get('order_line') and self.state == 'sale':
            for order in self:
                pre_order_line_qty = {order_line: order_line.product_uom_qty for order_line in order.mapped('order_line') if not order_line.is_expense}

        if values.get('partner_shipping_id') and self._context.get('update_delivery_shipping_partner'):
            for order in self:
                order.picking_ids.partner_id = values.get('partner_shipping_id')
        elif values.get('partner_shipping_id'):
            new_partner = self.env['res.partner'].browse(values.get('partner_shipping_id'))
            for record in self:
                picking = record.mapped('picking_ids').filtered(lambda x: x.state not in ('done', 'cancel'))
                message = _("""The delivery address has been changed on the Sales Order<br/>
                        From <strong>"%(old_address)s"</strong> to <strong>"%(new_address)s"</strong>,
                        You should probably update the partner on this document.""",
                            old_address=record.partner_shipping_id.display_name, new_address=new_partner.display_name)
                picking.activity_schedule('mail.mail_activity_data_warning', note=message, user_id=self.env.user.id)

        if 'commitment_date' in values:
            # protagate commitment_date as the deadline of the related stock move.
            # TODO: Log a note on each down document
            deadline_datetime = values.get('commitment_date')
            for order in self:
                order.order_line.move_ids.date_deadline = deadline_datetime or order.expected_date

        res = super(SaleOrder, self).write(values)
        if values.get('order_line') and self.state == 'sale':
            rounding = self.env['decimal.precision'].precision_get('Product Unit of Measure')
            for order in self:
                to_log = {}
                for order_line in order.order_line:
                    if order_line.display_type:
                        continue
                    if float_compare(order_line.product_uom_qty, pre_order_line_qty.get(order_line, 0.0), precision_rounding=order_line.product_uom.rounding or rounding) < 0:
                        to_log[order_line] = (order_line.product_uom_qty, pre_order_line_qty.get(order_line, 0.0))
                if to_log:
                    documents = self.env['stock.picking'].sudo()._log_activity_get_documents(to_log, 'move_ids', 'UP')
                    documents = {k: v for k, v in documents.items() if k[0].state != 'cancel'}
                    order._log_decrease_ordered_quantity(documents)
        return res

    def _compute_json_popover(self):
        for order in self:
            late_stock_picking = order.picking_ids.filtered(lambda p: p.delay_alert_date)
            order.json_popover = json.dumps({
                'popoverTemplate': 'sale_stock.DelayAlertWidget',
                'late_elements': [{
                        'id': late_move.id,
                        'name': late_move.display_name,
                        'model': 'stock.picking',
                    } for late_move in late_stock_picking
                ]
            })
            order.show_json_popover = bool(late_stock_picking)

    def _action_confirm(self):
        self.order_line._action_launch_stock_rule()
        return super(SaleOrder, self)._action_confirm()

    @api.depends('picking_ids')
    def _compute_picking_ids(self):
        for order in self:
            order.delivery_count = len(order.picking_ids)

    @api.depends('user_id', 'company_id')
    def _compute_warehouse_id(self):
        for order in self:
            default_warehouse_id = self.env['ir.default'].with_company(
                order.company_id.id)._get_model_defaults('sale.order').get('warehouse_id')
            if order.state in ['draft', 'sent'] or not order.ids:
                # Should expect empty
                if default_warehouse_id is not None:
                    order.warehouse_id = default_warehouse_id
                else:
                    order.warehouse_id = order.user_id.with_company(order.company_id.id)._get_default_warehouse_id()

    @api.onchange('partner_shipping_id')
    def _onchange_partner_shipping_id(self):
        res = {}
        pickings = self.picking_ids.filtered(
            lambda p: p.state not in ['done', 'cancel'] and p.partner_id != self.partner_shipping_id
        )
        if pickings:
            res['warning'] = {
                'title': _('Warning!'),
                'message': _(
                    'Do not forget to change the partner on the following delivery orders: %s',
                    ','.join(pickings.mapped('name')))
            }
        return res

    def action_view_delivery(self):
        return self._get_action_view_picking(self.picking_ids)

    def _action_cancel(self):
        documents = None
        for sale_order in self:
            if sale_order.state == 'sale' and sale_order.order_line:
                sale_order_lines_quantities = {order_line: (order_line.product_uom_qty, 0) for order_line in sale_order.order_line}
                documents = self.env['stock.picking'].with_context(include_draft_documents=True)._log_activity_get_documents(sale_order_lines_quantities, 'move_ids', 'UP')
        self.picking_ids.filtered(lambda p: p.state != 'done').action_cancel()
        if documents:
            filtered_documents = {}
            for (parent, responsible), rendering_context in documents.items():
                if parent._name == 'stock.picking':
                    if parent.state == 'cancel':
                        continue
                filtered_documents[(parent, responsible)] = rendering_context
            self._log_decrease_ordered_quantity(filtered_documents, cancel=True)
        return super()._action_cancel()

    def _get_action_view_picking(self, pickings):
        '''
        This function returns an action that display existing delivery orders
        of given sales order ids. It can either be a in a list or in a form
        view, if there is only one delivery order to show.
        '''
        action = self.env["ir.actions.actions"]._for_xml_id("stock.action_picking_tree_all")

        if len(pickings) > 1:
            action['domain'] = [('id', 'in', pickings.ids)]
        elif pickings:
            form_view = [(self.env.ref('stock.view_picking_form').id, 'form')]
            if 'views' in action:
                action['views'] = form_view + [(state,view) for state,view in action['views'] if view != 'form']
            else:
                action['views'] = form_view
            action['res_id'] = pickings.id
        # Prepare the context.
        picking_id = pickings.filtered(lambda l: l.picking_type_id.code == 'outgoing')
        if picking_id:
            picking_id = picking_id[0]
        else:
            picking_id = pickings[0]
        action['context'] = dict(default_partner_id=self.partner_id.id, default_picking_type_id=picking_id.picking_type_id.id, default_origin=self.name, default_group_id=picking_id.group_id.id)
        return action

    def _prepare_invoice(self):
        invoice_vals = super(SaleOrder, self)._prepare_invoice()
        invoice_vals['invoice_incoterm_id'] = self.incoterm.id
        return invoice_vals

    def _log_decrease_ordered_quantity(self, documents, cancel=False):

        def _render_note_exception_quantity_so(rendering_context):
            order_exceptions, visited_moves = rendering_context
            visited_moves = list(visited_moves)
            visited_moves = self.env[visited_moves[0]._name].concat(*visited_moves)
            order_line_ids = self.env['sale.order.line'].browse([order_line.id for order in order_exceptions.values() for order_line in order[0]])
            sale_order_ids = order_line_ids.mapped('order_id')
            impacted_pickings = visited_moves.filtered(lambda m: m.state not in ('done', 'cancel')).mapped('picking_id')
            values = {
                'sale_order_ids': sale_order_ids,
                'order_exceptions': order_exceptions.values(),
                'impacted_pickings': impacted_pickings,
                'cancel': cancel
            }
            return self.env['ir.qweb']._render('sale_stock.exception_on_so', values)

        self.env['stock.picking']._log_activity(_render_note_exception_quantity_so, documents)

```

## File: models\sale_order_line.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import timedelta
from collections import defaultdict

from odoo import api, fields, models, _
from odoo.osv import expression
from odoo.tools import float_compare
from odoo.exceptions import UserError


class SaleOrderLine(models.Model):
    _inherit = 'sale.order.line'

    qty_delivered_method = fields.Selection(selection_add=[('stock_move', 'Stock Moves')])
    route_id = fields.Many2one('stock.route', string='Route', domain=[('sale_selectable', '=', True)], ondelete='restrict')
    move_ids = fields.One2many('stock.move', 'sale_line_id', string='Stock Moves')
    virtual_available_at_date = fields.Float(compute='_compute_qty_at_date', digits='Product Unit of Measure')
    scheduled_date = fields.Datetime(compute='_compute_qty_at_date')
    forecast_expected_date = fields.Datetime(compute='_compute_qty_at_date')
    free_qty_today = fields.Float(compute='_compute_qty_at_date', digits='Product Unit of Measure')
    qty_available_today = fields.Float(compute='_compute_qty_at_date')
    warehouse_id = fields.Many2one('stock.warehouse', compute='_compute_warehouse_id', store=True)
    qty_to_deliver = fields.Float(compute='_compute_qty_to_deliver', digits='Product Unit of Measure')
    is_mto = fields.Boolean(compute='_compute_is_mto')
    display_qty_widget = fields.Boolean(compute='_compute_qty_to_deliver')
    is_storable = fields.Boolean(related='product_id.is_storable')
    customer_lead = fields.Float(
        compute='_compute_customer_lead', store=True, readonly=False, precompute=True,
        inverse='_inverse_customer_lead')

    @api.depends('route_id', 'order_id.warehouse_id', 'product_packaging_id', 'product_id')
    def _compute_warehouse_id(self):
        for line in self:
            line.warehouse_id = line.order_id.warehouse_id
            if line.route_id:
                domain = [
                    ('location_dest_id', '=', line.order_id.partner_shipping_id.property_stock_customer.id),
                    ('action', '!=', 'push'),
                ]
                # prefer rules on the route itself even if they pull from a different warehouse than the SO's
                rules = sorted(
                    self.env['stock.rule'].search(
                        domain=expression.AND([[('route_id', '=', line.route_id.id)], domain]),
                        order='route_sequence, sequence'
                    ),
                    # if there are multiple rules on the route, prefer those that pull from the SO's warehouse
                    # or those that are not warehouse specific
                    key=lambda rule: 0 if rule.location_src_id.warehouse_id in (False, line.order_id.warehouse_id) else 1
                )
                if rules:
                    line.warehouse_id = rules[0].location_src_id.warehouse_id

    @api.depends('is_storable', 'product_uom_qty', 'qty_delivered', 'state', 'move_ids', 'product_uom')
    def _compute_qty_to_deliver(self):
        """Compute the visibility of the inventory widget."""
        for line in self:
            line.qty_to_deliver = line.product_uom_qty - line.qty_delivered
            if line.state in ('draft', 'sent', 'sale') and line.is_storable and line.product_uom and line.qty_to_deliver > 0:
                if line.state == 'sale' and not line.move_ids:
                    line.display_qty_widget = False
                else:
                    line.display_qty_widget = True
            else:
                line.display_qty_widget = False

    @api.depends(
        'product_id', 'customer_lead', 'product_uom_qty', 'product_uom', 'order_id.commitment_date',
        'move_ids', 'move_ids.forecast_expected_date', 'move_ids.forecast_availability',
        'warehouse_id')
    def _compute_qty_at_date(self):
        """ Compute the quantity forecasted of product at delivery date. There are
        two cases:
         1. The quotation has a commitment_date, we take it as delivery date
         2. The quotation hasn't commitment_date, we compute the estimated delivery
            date based on lead time"""
        treated = self.browse()
        all_move_ids = {
            move.id
            for line in self
            if line.state == 'sale'
            for move in line.move_ids | self.env['stock.move'].browse(line.move_ids._rollup_move_origs())
            if move.product_id == line.product_id
        }
        all_moves = self.env['stock.move'].browse(all_move_ids)
        forecast_expected_date_per_move = dict(all_moves.mapped(lambda m: (m.id, m.forecast_expected_date)))
        # If the state is already in sale the picking is created and a simple forecasted quantity isn't enough
        # Then used the forecasted data of the related stock.move
        for line in self.filtered(lambda l: l.state == 'sale'):
            if not line.display_qty_widget:
                continue
            moves = line.move_ids | self.env['stock.move'].browse(line.move_ids._rollup_move_origs())
            moves = moves.filtered(
                lambda m: m.product_id == line.product_id and m.state not in ('cancel', 'done'))
            line.forecast_expected_date = max(
                (
                    forecast_expected_date_per_move[move.id]
                    for move in moves
                    if forecast_expected_date_per_move[move.id]
                ),
                default=False,
            )
            line.qty_available_today = 0
            line.free_qty_today = 0
            for move in moves:
                line.qty_available_today += move.product_uom._compute_quantity(move.quantity, line.product_uom)
                line.free_qty_today += move.product_id.uom_id._compute_quantity(move.forecast_availability, line.product_uom)
            line.scheduled_date = line.order_id.commitment_date or line._expected_date()
            line.virtual_available_at_date = False
            treated |= line

        qty_processed_per_product = defaultdict(lambda: 0)
        grouped_lines = defaultdict(lambda: self.env['sale.order.line'])
        # We first loop over the SO lines to group them by warehouse and schedule
        # date in order to batch the read of the quantities computed field.
        for line in self.filtered(lambda l: l.state in ('draft', 'sent')):
            if not (line.product_id and line.display_qty_widget):
                continue
            grouped_lines[(line.warehouse_id.id, line.order_id.commitment_date or line._expected_date())] |= line

        for (warehouse, scheduled_date), lines in grouped_lines.items():
            product_qties = lines.mapped('product_id').with_context(to_date=scheduled_date, warehouse_id=warehouse).read([
                'qty_available',
                'free_qty',
                'virtual_available',
            ])
            qties_per_product = {
                product['id']: (product['qty_available'], product['free_qty'], product['virtual_available'])
                for product in product_qties
            }
            for line in lines:
                line.scheduled_date = scheduled_date
                qty_available_today, free_qty_today, virtual_available_at_date = qties_per_product[line.product_id.id]
                line.qty_available_today = qty_available_today - qty_processed_per_product[line.product_id.id]
                line.free_qty_today = free_qty_today - qty_processed_per_product[line.product_id.id]
                line.virtual_available_at_date = virtual_available_at_date - qty_processed_per_product[line.product_id.id]
                line.forecast_expected_date = False
                product_qty = line.product_uom_qty
                if line.product_uom and line.product_id.uom_id and line.product_uom != line.product_id.uom_id:
                    line.qty_available_today = line.product_id.uom_id._compute_quantity(line.qty_available_today, line.product_uom)
                    line.free_qty_today = line.product_id.uom_id._compute_quantity(line.free_qty_today, line.product_uom)
                    line.virtual_available_at_date = line.product_id.uom_id._compute_quantity(line.virtual_available_at_date, line.product_uom)
                    product_qty = line.product_uom._compute_quantity(product_qty, line.product_id.uom_id)
                qty_processed_per_product[line.product_id.id] += product_qty
            treated |= lines
        remaining = (self - treated)
        remaining.virtual_available_at_date = False
        remaining.scheduled_date = False
        remaining.forecast_expected_date = False
        remaining.free_qty_today = False
        remaining.qty_available_today = False

    @api.depends('product_id', 'route_id', 'warehouse_id', 'product_id.route_ids')
    def _compute_is_mto(self):
        """ Verify the route of the product based on the warehouse
            set 'is_available' at True if the product availability in stock does
            not need to be verified, which is the case in MTO, Cross-Dock or Drop-Shipping
        """
        self.is_mto = False
        for line in self:
            if not line.display_qty_widget:
                continue
            product = line.product_id
            product_routes = line.route_id or (product.route_ids + product.categ_id.total_route_ids)

            # Check MTO
            mto_route = line.warehouse_id.mto_pull_id.route_id
            if not mto_route:
                try:
                    mto_route = self.env['stock.warehouse']._find_or_create_global_route('stock.route_warehouse0_mto', _('Replenish on Order (MTO)'), create=False)
                except UserError:
                    # if route MTO not found in ir_model_data, we treat the product as in MTS
                    pass

            if mto_route and mto_route in product_routes:
                line.is_mto = True
            else:
                line.is_mto = False

    @api.depends('product_id')
    def _compute_qty_delivered_method(self):
        """ Stock module compute delivered qty for product [('type', '=', 'consu')]
            For SO line coming from expense, no picking should be generate: we don't manage stock for
            those lines, even if the product is a storable.
        """
        super(SaleOrderLine, self)._compute_qty_delivered_method()

        for line in self:
            if not line.is_expense and line.product_id.type == 'consu':
                line.qty_delivered_method = 'stock_move'

    @api.depends('move_ids.state', 'move_ids.scrapped', 'move_ids.quantity', 'move_ids.product_uom')
    def _compute_qty_delivered(self):
        super(SaleOrderLine, self)._compute_qty_delivered()

        for line in self:  # TODO: maybe one day, this should be done in SQL for performance sake
            if line.qty_delivered_method == 'stock_move':
                qty = 0.0
                outgoing_moves, incoming_moves = line._get_outgoing_incoming_moves()
                for move in outgoing_moves:
                    if move.state != 'done':
                        continue
                    qty += move.product_uom._compute_quantity(move.quantity, line.product_uom, rounding_method='HALF-UP')
                for move in incoming_moves:
                    if move.state != 'done':
                        continue
                    qty -= move.product_uom._compute_quantity(move.quantity, line.product_uom, rounding_method='HALF-UP')
                line.qty_delivered = qty

    @api.model_create_multi
    def create(self, vals_list):
        lines = super(SaleOrderLine, self).create(vals_list)
        lines.filtered(lambda line: line.state == 'sale')._action_launch_stock_rule()
        return lines

    def write(self, values):
        lines = self.env['sale.order.line']
        if 'product_uom_qty' in values:
            lines = self.filtered(lambda r: r.state == 'sale' and not r.is_expense)

        if 'product_packaging_id' in values:
            self.move_ids.filtered(
                lambda m: m.state not in ['cancel', 'done']
            ).product_packaging_id = values['product_packaging_id']

        previous_product_uom_qty = {line.id: line.product_uom_qty for line in lines}
        res = super(SaleOrderLine, self).write(values)
        if lines:
            lines._action_launch_stock_rule(previous_product_uom_qty=previous_product_uom_qty)
        return res

    @api.depends('move_ids')
    def _compute_product_updatable(self):
        super()._compute_product_updatable()
        for line in self:
            if line.move_ids.filtered(lambda m: m.state != 'cancel'):
                line.product_updatable = False

    @api.depends('product_id')
    def _compute_customer_lead(self):
        super()._compute_customer_lead() # Reset customer_lead when the product is modified
        for line in self:
            line.customer_lead = line.product_id.sale_delay

    def _inverse_customer_lead(self):
        for line in self:
            if line.state == 'sale' and not line.order_id.commitment_date:
                # Propagate deadline on related stock move
                line.move_ids.date_deadline = line.order_id.date_order + timedelta(days=line.customer_lead or 0.0)

    def _prepare_procurement_values(self, group_id=False):
        """ Prepare specific key for moves or other components that will be created from a stock rule
        coming from a sale order line. This method could be override in order to add other custom key that could
        be used in move/po creation.
        """
        values = super(SaleOrderLine, self)._prepare_procurement_values(group_id)
        self.ensure_one()
        # Use the delivery date if there is else use date_order and lead time
        date_deadline = self.order_id.commitment_date or self._expected_date()
        date_planned = date_deadline - timedelta(days=self.order_id.company_id.security_lead)
        values.update({
            'group_id': group_id,
            'sale_line_id': self.id,
            'date_planned': date_planned,
            'date_deadline': date_deadline,
            'route_ids': self.route_id,
            'warehouse_id': self.warehouse_id,
            'partner_id': self.order_id.partner_shipping_id.id,
            'location_final_id': self._get_location_final(),
            'product_description_variants': self.with_context(lang=self.order_id.partner_id.lang)._get_sale_order_line_multiline_description_variants(),
            'company_id': self.order_id.company_id,
            'product_packaging_id': self.product_packaging_id,
            'sequence': self.sequence,
            'never_product_template_attribute_value_ids': self.product_no_variant_attribute_value_ids,
        })
        return values

    def _get_location_final(self):
        # Can be overriden for inter-company transactions.
        self.ensure_one()
        return self.order_id.partner_shipping_id.property_stock_customer

    def _get_qty_procurement(self, previous_product_uom_qty=False):
        self.ensure_one()
        qty = 0.0
        outgoing_moves, incoming_moves = self._get_outgoing_incoming_moves(strict=False)
        for move in outgoing_moves:
            qty_to_compute = move.quantity if move.state == 'done' else move.product_uom_qty
            qty += move.product_uom._compute_quantity(qty_to_compute, self.product_uom, rounding_method='HALF-UP')
        for move in incoming_moves:
            qty_to_compute = move.quantity if move.state == 'done' else move.product_uom_qty
            qty -= move.product_uom._compute_quantity(qty_to_compute, self.product_uom, rounding_method='HALF-UP')
        return qty

    def _get_outgoing_incoming_moves(self, strict=True):
        """ Return the outgoing and incoming moves of the sale order line.
            @param strict: If True, only consider the moves that are strictly delivered to the customer (old behavior).
                           If False, consider the moves that were created through the initial rule of the delivery route,
                           to support the new push mechanism.
        """
        outgoing_moves_ids = set()
        incoming_moves_ids = set()

        moves = self.move_ids.filtered(lambda r: r.state != 'cancel' and not r.scrapped and self.product_id == r.product_id)
        if moves and not strict:
            # The first move created was the one created from the intial rule that started it all.
            sorted_moves = moves.sorted('id')
            triggering_rule_ids = []
            seen_wh_ids = set()
            for move in sorted_moves:
                if move.warehouse_id.id not in seen_wh_ids:
                    triggering_rule_ids.append(move.rule_id.id)
                    seen_wh_ids.add(move.warehouse_id.id)
        if self._context.get('accrual_entry_date'):
            moves = moves.filtered(lambda r: fields.Date.context_today(r, r.date) <= self._context['accrual_entry_date'])

        for move in moves:
            if (strict and move.location_dest_id._is_outgoing()) or \
               (not strict and move.rule_id.id in triggering_rule_ids and (move.location_final_id or move.location_dest_id)._is_outgoing()):
                if not move.origin_returned_move_id or (move.origin_returned_move_id and move.to_refund):
                    outgoing_moves_ids.add(move.id)
            elif move.location_id._is_outgoing() and move.to_refund:
                incoming_moves_ids.add(move.id)

        return self.env['stock.move'].browse(outgoing_moves_ids), self.env['stock.move'].browse(incoming_moves_ids)

    def _get_procurement_group(self):
        return self.order_id.procurement_group_id

    def _prepare_procurement_group_vals(self):
        return {
            'name': self.order_id.name,
            'move_type': self.order_id.picking_policy,
            'sale_id': self.order_id.id,
            'partner_id': self.order_id.partner_shipping_id.id,
        }

    def _create_procurements(self, product_qty, procurement_uom, origin, values):
        self.ensure_one()
        return [self.env['procurement.group'].Procurement(
            self.product_id, product_qty, procurement_uom, self._get_location_final(),
            self.product_id.display_name, origin, self.order_id.company_id, values)]

    def _action_launch_stock_rule(self, previous_product_uom_qty=False):
        """
        Launch procurement group run method with required/custom fields generated by a
        sale order line. procurement group will launch '_run_pull', '_run_buy' or '_run_manufacture'
        depending on the sale order line product rule.
        """
        if self._context.get("skip_procurement"):
            return True
        precision = self.env['decimal.precision'].precision_get('Product Unit of Measure')
        procurements = []
        for line in self:
            line = line.with_company(line.company_id)
            if line.state != 'sale' or line.order_id.locked or line.product_id.type != 'consu':
                continue
            qty = line._get_qty_procurement(previous_product_uom_qty)
            if float_compare(qty, line.product_uom_qty, precision_digits=precision) == 0:
                continue

            group_id = line._get_procurement_group()
            if not group_id:
                group_id = self.env['procurement.group'].create(line._prepare_procurement_group_vals())
                line.order_id.procurement_group_id = group_id
            else:
                # In case the procurement group is already created and the order was
                # cancelled, we need to update certain values of the group.
                updated_vals = {}
                if group_id.partner_id != line.order_id.partner_shipping_id:
                    updated_vals.update({'partner_id': line.order_id.partner_shipping_id.id})
                if group_id.move_type != line.order_id.picking_policy:
                    updated_vals.update({'move_type': line.order_id.picking_policy})
                if updated_vals:
                    group_id.write(updated_vals)

            values = line._prepare_procurement_values(group_id=group_id)
            product_qty = line.product_uom_qty - qty

            line_uom = line.product_uom
            quant_uom = line.product_id.uom_id
            origin = f'{line.order_id.name} - {line.order_id.client_order_ref}' if line.order_id.client_order_ref else line.order_id.name
            product_qty, procurement_uom = line_uom._adjust_uom_quantities(product_qty, quant_uom)
            procurements += line._create_procurements(product_qty, procurement_uom, origin, values)
        if procurements:
            self.env['procurement.group'].run(procurements)

        # This next block is currently needed only because the scheduler trigger is done by picking confirmation rather than stock.move confirmation
        orders = self.mapped('order_id')
        for order in orders:
            pickings_to_confirm = order.picking_ids.filtered(lambda p: p.state not in ['cancel', 'done'])
            if pickings_to_confirm:
                # Trigger the Scheduler for Pickings
                pickings_to_confirm.action_confirm()
        return True

    def _update_line_quantity(self, values):
        precision = self.env['decimal.precision'].precision_get('Product Unit of Measure')
        line_products = self.filtered(lambda l: l.product_id.type == 'consu')
        if line_products.mapped('qty_delivered') and float_compare(values['product_uom_qty'], max(line_products.mapped('qty_delivered')), precision_digits=precision) == -1:
            raise UserError(_('The ordered quantity of a sale order line cannot be decreased below the amount already delivered. Instead, create a return in your inventory.'))
        super(SaleOrderLine, self)._update_line_quantity(values)

    #=== HOOKS ===#

    def _get_action_add_from_catalog_extra_context(self, order):
        extra_context = super()._get_action_add_from_catalog_extra_context(order)
        extra_context.update(warehouse_id=order.warehouse_id.id)
        return extra_context

    def _get_product_catalog_lines_data(self, **kwargs):
        """ Override of `sale` to add the delivered quantity.

        :rtype: dict
        :return: A dict with the following structure:
            {
                'deliveredQty': float,
                'quantity': float,
                'price': float,
                'readOnly': bool,
            }
        """
        res = super()._get_product_catalog_lines_data(**kwargs)
        res['deliveredQty'] = sum(
            self.mapped(
                lambda line: line.product_uom._compute_quantity(
                    qty=line.qty_delivered,
                    to_unit=line.product_id.uom_id,
                )
            )
        )
        return res

```

## File: models\stock.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict

from odoo import api, fields, models, _
from odoo.tools.sql import column_exists, create_column


class StockRoute(models.Model):
    _inherit = "stock.route"
    sale_selectable = fields.Boolean("Selectable on Sales Order Line")


class StockMove(models.Model):
    _inherit = "stock.move"
    sale_line_id = fields.Many2one('sale.order.line', 'Sale Line', index='btree_not_null')

    @api.model
    def _prepare_merge_moves_distinct_fields(self):
        distinct_fields = super(StockMove, self)._prepare_merge_moves_distinct_fields()
        distinct_fields.append('sale_line_id')
        return distinct_fields

    def _get_related_invoices(self):
        """ Overridden from stock_account to return the customer invoices
        related to this stock move.
        """
        rslt = super(StockMove, self)._get_related_invoices()
        invoices = self.mapped('picking_id.sale_id.invoice_ids').filtered(lambda x: x.state == 'posted')
        rslt += invoices
        #rslt += invoices.mapped('reverse_entry_ids')
        return rslt

    def _get_source_document(self):
        res = super()._get_source_document()
        return self.sudo().sale_line_id.order_id or res

    def _get_sale_order_lines(self):
        """ Return all possible sale order lines for one stock move. """
        self.ensure_one()
        return (self + self.browse(self._rollup_move_origs() | self._rollup_move_dests())).sale_line_id

    def _assign_picking_post_process(self, new=False):
        super(StockMove, self)._assign_picking_post_process(new=new)
        if new:
            picking_id = self.mapped('picking_id')
            sale_order_ids = self.mapped('sale_line_id.order_id')
            for sale_order_id in sale_order_ids:
                picking_id.message_post_with_source(
                    'mail.message_origin_link',
                    render_values={'self': picking_id, 'origin': sale_order_id},
                    subtype_xmlid='mail.mt_note',
                )

    def _get_all_related_sm(self, product):
        return super()._get_all_related_sm(product) | self.filtered(lambda m: m.sale_line_id.product_id == product)


class StockMoveLine(models.Model):
    _inherit = "stock.move.line"

    def _should_show_lot_in_invoice(self):
        return 'customer' in {self.location_id.usage, self.location_dest_id.usage}


class ProcurementGroup(models.Model):
    _inherit = 'procurement.group'

    sale_id = fields.Many2one('sale.order', 'Sale Order')


class StockRule(models.Model):
    _inherit = 'stock.rule'

    def _get_custom_move_fields(self):
        fields = super(StockRule, self)._get_custom_move_fields()
        fields += ['sale_line_id', 'partner_id', 'sequence', 'to_refund']
        return fields


class StockPicking(models.Model):
    _inherit = 'stock.picking'

    sale_id = fields.Many2one('sale.order', compute="_compute_sale_id", inverse="_set_sale_id", string="Sales Order", store=True, index='btree_not_null')

    @api.depends('group_id')
    def _compute_sale_id(self):
        for picking in self:
            picking.sale_id = picking.group_id.sale_id

    def _set_sale_id(self):
        if self.group_id:
            self.group_id.sale_id = self.sale_id
        else:
            if self.sale_id:
                vals = {
                    'sale_id': self.sale_id.id,
                    'name': self.sale_id.name,
                }
            else:
                vals = {}

            pg = self.env['procurement.group'].create(vals)
            self.group_id = pg

    def _auto_init(self):
        """
        Create related field here, too slow
        when computing it afterwards through _compute_related.

        Since group_id.sale_id is created in this module,
        no need for an UPDATE statement.
        """
        if not column_exists(self.env.cr, 'stock_picking', 'sale_id'):
            create_column(self.env.cr, 'stock_picking', 'sale_id', 'int4')
        return super()._auto_init()

    def _action_done(self):
        res = super()._action_done()
        sale_order_lines_vals = []
        for move in self.move_ids:
            sale_order = move.picking_id.sale_id
            # Creates new SO line only when pickings linked to a sale order and
            # for moves with qty. done and not already linked to a SO line.
            if not sale_order or move.sale_line_id or not move.picked or not (
                (move.location_dest_id.usage in ['customer', 'transit'] and not move.move_dest_ids)
                or (move.location_id.usage == 'customer' and move.to_refund)
            ):
                continue
            product = move.product_id
            quantity = move.quantity
            if move.to_refund:
                quantity *= -1

            so_line_vals = {
                'move_ids': [(4, move.id, 0)],
                'name': product.display_name,
                'order_id': sale_order.id,
                'product_id': product.id,
                'product_uom_qty': 0,
                'qty_delivered': quantity,
                'product_uom': move.product_uom.id,
            }
            if product.invoice_policy == 'delivery':
                # Check if there is already a SO line for this product to get
                # back its unit price (in case it was manually updated).
                so_line = sale_order.order_line.filtered(lambda sol: sol.product_id == product)
                if so_line:
                    so_line_vals['price_unit'] = so_line[0].price_unit
            elif product.invoice_policy == 'order':
                # No unit price if the product is invoiced on the ordered qty.
                so_line_vals['price_unit'] = 0
            sale_order_lines_vals.append(so_line_vals)

        if sale_order_lines_vals:
            self.env['sale.order.line'].with_context(skip_procurement=True).create(sale_order_lines_vals)
        return res

    def _log_less_quantities_than_expected(self, moves):
        """ Log an activity on sale order that are linked to moves. The
        note summarize the real processed quantity and promote a
        manual action.

        :param dict moves: a dict with a move as key and tuple with
        new and old quantity as value. eg: {move_1 : (4, 5)}
        """

        def _keys_in_groupby(sale_line):
            """ group by order_id and the sale_person on the order """
            return (sale_line.order_id, sale_line.order_id.user_id)

        def _render_note_exception_quantity(moves_information):
            """ Generate a note with the picking on which the action
            occurred and a summary on impacted quantity that are
            related to the sale order where the note will be logged.

            :param moves_information dict:
            {'move_id': ['sale_order_line_id', (new_qty, old_qty)], ..}

            :return: an html string with all the information encoded.
            :rtype: str
            """
            origin_moves = self.env['stock.move'].browse([move.id for move_orig in moves_information.values() for move in move_orig[0]])
            origin_picking = origin_moves.mapped('picking_id')
            values = {
                'origin_moves': origin_moves,
                'origin_picking': origin_picking,
                'moves_information': moves_information.values(),
            }
            return self.env['ir.qweb']._render('sale_stock.exception_on_picking', values)

        documents = self.sudo()._log_activity_get_documents(moves, 'sale_line_id', 'DOWN', _keys_in_groupby)
        self._log_activity(_render_note_exception_quantity, documents)

        return super(StockPicking, self)._log_less_quantities_than_expected(moves)

    def _can_return(self):
        self.ensure_one()
        return super()._can_return() or self.sale_id


class StockLot(models.Model):
    _inherit = 'stock.lot'

    sale_order_ids = fields.Many2many('sale.order', string="Sales Orders", compute='_compute_sale_order_ids')
    sale_order_count = fields.Integer('Sale order count', compute='_compute_sale_order_ids')

    @api.depends('name')
    def _compute_sale_order_ids(self):
        sale_orders = defaultdict(lambda: self.env['sale.order'])
        for move_line in self.env['stock.move.line'].search([('lot_id', 'in', self.ids), ('state', '=', 'done')]):
            move = move_line.move_id
            if move.picking_id.location_dest_id.usage in ('customer', 'transit') and move.sale_line_id.order_id:
                sale_orders[move_line.lot_id.id] |= move.sale_line_id.order_id
        for lot in self:
            lot.sale_order_ids = sale_orders[lot.id]
            lot.sale_order_count = len(lot.sale_order_ids)

    def action_view_so(self):
        self.ensure_one()
        action = self.env["ir.actions.actions"]._for_xml_id("sale.action_orders")
        action['domain'] = [('id', 'in', self.mapped('sale_order_ids.id'))]
        action['context'] = dict(self._context, create=False)
        return action

```

## File: models\stock_warehouse.py

```python
from odoo import models

class Warehouse(models.Model):
    _inherit = "stock.warehouse"

    def _get_routes_values(self):
        routes = super()._get_routes_values()
        if routes.get('crossdock_route_id'):
            routes['crossdock_route_id']['route_update_values']['sale_selectable'] = True
            routes['crossdock_route_id']['route_create_values']['sale_selectable'] = True
        return routes

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_move
from . import product_template
from . import res_company
from . import res_config_settings
from . import res_users
from . import sale_order
from . import sale_order_line
from . import stock
from . import stock_warehouse

```

## File: report\report_stock_rule.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class ReportStockRule(models.AbstractModel):
    _inherit = 'report.stock.report_stock_rule'

    @api.model
    def _get_routes(self, data):
        res = super(ReportStockRule, self)._get_routes(data)
        if data.get('so_route_ids'):
            res = self.env['stock.route'].browse(data['so_route_ids']) | res
        return res

```

## File: report\sale_order_report_templates.xml

```xml
<?xml version="1.0"?>
<odoo>
    <template id="report_saleorder_document_inherit_sale_stock" inherit_id="sale.report_saleorder_document">
        <xpath expr="//div[@name='expiration_date']" position="after">
            <div class="col" t-if="doc.incoterm">
                <strong>Incoterm</strong>
                <div t-if="doc.incoterm_location">
                    <span t-field="doc.incoterm.code"/> <br/>
                    <span t-field="doc.incoterm_location"/>
                </div>
                <div t-else="" t-field="doc.incoterm.code"/>
            </div>
        </xpath>
    </template>
</odoo>

```

## File: report\sale_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class SaleReport(models.Model):
    _inherit = "sale.report"

    warehouse_id = fields.Many2one('stock.warehouse', 'Warehouse', readonly=True)

    def _select_additional_fields(self):
        res = super()._select_additional_fields()
        res['warehouse_id'] = "s.warehouse_id"
        return res

    def _group_by_sale(self):
        res = super()._group_by_sale()
        res += """,
            s.warehouse_id"""
        return res

```

## File: report\stock_forecasted.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class StockForecasted(models.AbstractModel):
    _inherit = 'stock.forecasted_product_product'

    def _prepare_report_line(self, quantity, move_out=None, move_in=None, replenishment_filled=True, product=False, reserved_move=False, in_transit=False, read=True):
        line = super()._prepare_report_line(quantity, move_out, move_in, replenishment_filled, product, reserved_move, in_transit, read)

        if not move_out or not move_out.picking_id or not move_out.picking_id.sale_id:
            return line

        picking = move_out.picking_id
        # If read is False, line['move_out'] is a stock.move record and will trigger a record update
        if read:
            line['move_out'].update({
                'picking_id': {
                    'id': picking.id,
                    'priority': picking.priority,
                    'sale_id': {
                        'id': picking.sale_id.id,
                        'amount_untaxed': picking.sale_id.amount_untaxed,
                        'currency_id': picking.sale_id.currency_id.read(fields=['id', 'name'])[0],
                        'partner_id': picking.sale_id.partner_id.read(fields=['id', 'name'])[0],
                    }
                }
            })
        return line

    def _get_report_header(self, product_template_ids, product_ids, wh_location_ids):
        res = super()._get_report_header(product_template_ids, product_ids, wh_location_ids)
        domain = self._product_sale_domain(product_template_ids, product_ids)
        so_lines = self.env['sale.order.line'].sudo().search(domain)
        out_sum = 0
        if so_lines:
            product_uom = so_lines[0].product_id.uom_id
            quantities = so_lines.mapped(lambda line: line.product_uom._compute_quantity(line.product_uom_qty, product_uom))
            out_sum = sum(quantities)
        res['draft_sale_qty'] = out_sum
        res['draft_sale_orders'] = so_lines.mapped("order_id").sorted(key=lambda so: so.name).read(fields=['id', 'name'])
        res['draft_sale_orders_matched'] = self.env.context.get('sale_line_to_match_id') in so_lines.ids
        res['qty']['out'] += out_sum
        return res

    def _product_sale_domain(self, product_template_ids, product_ids):
        domain = [('state', 'in', ['draft', 'sent'])]
        if product_template_ids:
            domain += [('product_template_id', 'in', product_template_ids)]
        elif product_ids:
            domain += [('product_id', 'in', product_ids)]
        warehouse_id = self.env['stock.warehouse']._get_warehouse_id_from_context()
        if warehouse_id:
            domain += [('warehouse_id', '=', warehouse_id)]
        return domain

```

## File: report\stock_report_deliveryslip.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="report_delivery_document_inherit_sale_stock" inherit_id="stock.report_delivery_document">
        <xpath expr="//div[@name='div_sched_date']" position="after">
            <div class="col col-3 mw-100 mb-2" t-if="o.sudo().sale_id.client_order_ref">
                <strong>Customer Reference</strong>
                <div t-field="o.sudo().sale_id.client_order_ref" class="m-0">Customer reference</div>
            </div>
            <div class="col col-3" t-if="o.sudo().sale_id.incoterm">
                <strong>Incoterm</strong>
                <div t-if="o.sudo().sale_id.incoterm_location" t-out="'%s %s' % (o.sudo().sale_id.incoterm.code, o.sudo().sale_id.incoterm_location)">Incoterm details</div>
                <div t-else="" t-field="o.sudo().sale_id.incoterm.display_name">Incoterm details</div>
            </div>
        </xpath>
    </template>
</odoo>

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import sale_report
from . import stock_forecasted
from . import report_stock_rule

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_stock_picking_salesman,stock_picking salesman,stock.model_stock_picking,sales_team.group_sale_salesman,1,1,1,0
access_stock_move_salesman,stock_move salesman,stock.model_stock_move,sales_team.group_sale_salesman,1,1,1,0
access_stock_move_manager,stock_move manager,stock.model_stock_move,sales_team.group_sale_manager,1,1,1,1
access_sale_order_stock_worker,sale.order stock worker,model_sale_order,stock.group_stock_user,1,1,0,0
access_sale_order_line_stock_worker,sale.order.line stock worker,model_sale_order_line,stock.group_stock_user,1,1,0,0
access_stock_picking_sales,stock.picking.sales,stock.model_stock_picking,sales_team.group_sale_manager,1,1,1,1
access_stock_picking_portal,stock.picking,stock.model_stock_picking,base.group_portal,1,0,0,0
access_product_packaging_user,product.packaging.user,product.model_product_packaging,sales_team.group_sale_salesman,1,1,1,0
access_stock_warehouse_user,stock.warehouse.user,stock.model_stock_warehouse,sales_team.group_sale_salesman,1,0,0,0
access_stock_location_user,stock.location.user,stock.model_stock_location,sales_team.group_sale_salesman,1,0,0,0
access_product_packaging_sale_manager,product.packaging salemanager,product.model_product_packaging,sales_team.group_sale_manager,1,1,1,1
access_stock_warehouse_orderpoint_sale_salesman,stock.warehouse.orderpoint,stock.model_stock_warehouse_orderpoint,sales_team.group_sale_salesman,1,0,0,0
access_account_partial_reconcile,account.partial.reconcile,account.model_account_partial_reconcile,stock.group_stock_manager,1,1,1,1
access_account_journal,account.journal,account.model_account_journal,stock.group_stock_manager,1,1,0,0
access_stock_location_sale_manager,stock.location sale manager,stock.model_stock_location,sales_team.group_sale_manager,1,0,0,0
access_stock_rule_salemanager,stock_rule salemanager,stock.model_stock_rule,sales_team.group_sale_manager,1,1,1,1
access_stock_rule,stock.rule.flow,stock.model_stock_rule,sales_team.group_sale_salesman,1,0,0,0
access_stock_package_type_salesman,stock_package_type salesman,stock.model_stock_package_type,sales_team.group_sale_salesman,1,0,0,0

```

## File: security\sale_stock_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <!-- Stock Portal Access Rules -->
        <record id="stock_picking_rule_portal" model="ir.rule">
            <field name="name">Portal Follower Transfers</field>
            <field name="model_id" ref="stock.model_stock_picking"/>
            <field name="domain_force">['|', '|', ('message_partner_ids', 'in', [user.partner_id.id]), ('partner_id', '=', user.partner_id.id), ('sale_id.partner_id', '=', user.partner_id.id)]</field>
            <field name="groups" eval="[(4, ref('base.group_portal'))]"/>
        </record>
    </data>
</odoo>

```

## File: static\src\product_catalog\kanban_record.js

```javascript
/** @odoo-module */
import { ProductCatalogKanbanRecord } from "@product/product_catalog/kanban_record";
import { ProductCatalogSaleOrderLine } from "./sale_order_line/sale_order_line";
import { patch } from "@web/core/utils/patch";

patch(ProductCatalogKanbanRecord.prototype, {
    updateQuantity(quantity) {
        if (this.env.orderResModel !== "sale.order" || this.productCatalogData.productType == "service") {
            super.updateQuantity(...arguments);
        } else if (
            this.productCatalogData.quantity === this.productCatalogData.deliveredQty &&
            quantity < this.productCatalogData.quantity
        ) {
            // This condition is only triggered when the product was already at the minimum quantity
            // possible, as stated in the sale_stock module, then the user inputs a quantity lower
            // than this limit, in this case we need the record to forcefully update the record.
            this.props.record.load();
            this.props.record.model.notify();
        } else {
            super.updateQuantity(Math.max(quantity, this.productCatalogData.deliveredQty));
        }
    },

    get orderLineComponent() {
        if (this.env.orderResModel === "sale.order") {
            return ProductCatalogSaleOrderLine;
        }
        return super.orderLineComponent;
    },
});

```

## File: static\src\product_catalog\sale_order_line\sale_order_line.js

```javascript
/** @odoo-module */

import { _t } from "@web/core/l10n/translation";
import { ProductCatalogOrderLine } from "@product/product_catalog/order_line/order_line";

export class ProductCatalogSaleOrderLine extends ProductCatalogOrderLine {
    static props = {
        ...ProductCatalogOrderLine.props,
        deliveredQty: Number,
    }

    get disableRemove() {
        return this.props.quantity === this.props.deliveredQty;
    }

    get disabledButtonTooltip() {
        if (this.disableRemove) {
            return _t("The ordered quantity cannot be decreased below the amount already delivered. Instead, create a return in your inventory.");
        }
        return super.disabledButtonTooltip;
    }
}

```

## File: static\src\sale_stock_forecasted\forecasted_details.js

```javascript
/** @odoo-module **/
import { formatMonetary } from "@web/views/fields/formatters";
import { patch } from "@web/core/utils/patch";

import { ForecastedDetails } from "@stock/stock_forecasted/forecasted_details";

patch(ForecastedDetails.prototype, {
    _formatMonetary(num, currencyId){
        return formatMonetary(num,{ currencyId: currencyId});
    }
});

```

## File: static\src\sale_stock_forecasted\forecasted_details.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="sale_stock.ForecastedDetails" t-inherit="stock.ForecastedDetails" t-inherit-mode="extension">
        <xpath expr="//tr[@name='draft_picking_out']" position="after">
            <tr t-if="props.docs.draft_sale_qty" name="draft_so_out" t-attf-class="#{props.docs.draft_sale_orders_matched and 'o_grid_match table-info'}">
                <td colspan="2">Quotations</td>
                <td t-out="_formatFloat(-props.docs.draft_sale_qty)" class="text-end"/>
                <td>
                    <t t-foreach="props.docs.draft_sale_orders" t-as="sale_order" t-key="sale_order_index">
                        <t t-if="sale_order_index > 0"> | </t>
                        <a t-attf-href="#" t-out="sale_order.name"
                            class="fw-bold"
                            t-on-click.prevent="() => this.props.openView('sale.order', 'form', sale_order.id)"/>
                    </t>
                </td>
            </tr>
        </xpath>
        <xpath expr="//td[@name='usedby_cell']" position="inside">
            <span t-if="line.move_out and line.move_out.picking_id and line.move_out.picking_id.sale_id">
                | <span t-out="_formatMonetary(line.move_out.picking_id.sale_id.amount_untaxed, line.move_out.picking_id.sale_id.currency_id.id)" class="fw-bold"/>
                | <span t-out="line.move_out.picking_id.sale_id.partner_id.name"/>
            </span>
        </xpath>
    </t>
</templates>

```

## File: static\src\widgets\qty_at_date_widget.js

```javascript
/** @odoo-module **/

import { formatDateTime } from "@web/core/l10n/dates";
import { localization } from "@web/core/l10n/localization";
import { registry } from "@web/core/registry";
import { useService } from "@web/core/utils/hooks";
import { usePopover } from "@web/core/popover/popover_hook";
import { Component, onWillRender } from "@odoo/owl";
import { standardWidgetProps } from "@web/views/widgets/standard_widget_props";

export class QtyAtDatePopover extends Component {
    static template = "sale_stock.QtyAtDatePopover";
    static props = {
        record: Object,
        calcData: Object,
        close: Function,
    };
    setup() {
        this.actionService = useService("action");
    }

    openForecast() {
        this.actionService.doAction("stock.stock_forecasted_product_product_action", {
            additionalContext: {
                active_model: 'product.product',
                active_id: this.props.record.data.product_id[0],
                warehouse_id: this.props.record.data.warehouse_id && this.props.record.data.warehouse_id[0],
                move_to_match_ids: this.props.record.data.move_ids.records.map(record => record.resId),
                sale_line_to_match_id: this.props.record.resId,
            },
        });
    }
}


export class QtyAtDateWidget extends Component {
    static components = { Popover: QtyAtDatePopover };
    static template = "sale_stock.QtyAtDate";
    static props = {...standardWidgetProps};
    setup() {
        this.popover = usePopover(this.constructor.components.Popover, { position: "top" });
        this.calcData = {};
        onWillRender(() => {
            this.initCalcData();
        })
    }

    initCalcData() {
        // calculate data not in record
        const { data } = this.props.record;
        if (data.scheduled_date) {
            // TODO: might need some round_decimals to avoid errors
            if (data.state === 'sale') {
                this.calcData.will_be_fulfilled = data.free_qty_today >= data.qty_to_deliver;
            } else {
                this.calcData.will_be_fulfilled = data.virtual_available_at_date >= data.qty_to_deliver;
            }
            this.calcData.will_be_late = data.forecast_expected_date && data.forecast_expected_date > data.scheduled_date;
            if (['draft', 'sent'].includes(data.state)) {
                // Moves aren't created yet, then the forecasted is only based on virtual_available of quant
                this.calcData.forecasted_issue = !this.calcData.will_be_fulfilled && !data.is_mto;
            } else {
                // Moves are created, using the forecasted data of related moves
                this.calcData.forecasted_issue = !this.calcData.will_be_fulfilled || this.calcData.will_be_late;
            }
        }
    }

    updateCalcData() {
        // popup specific data
        const { data } = this.props.record;
        if (!data.scheduled_date) {
            return;
        }
        this.calcData.delivery_date = formatDateTime(data.scheduled_date, { format: localization.dateFormat });
        if (data.forecast_expected_date) {
            this.calcData.forecast_expected_date_str = formatDateTime(data.forecast_expected_date, { format: localization.dateFormat });
        }
    }

    showPopup(ev) {
        this.updateCalcData();
        this.popover.open(ev.currentTarget, {
            record: this.props.record,
            calcData: this.calcData,
        });
    }
}

export const qtyAtDateWidget = {
    component: QtyAtDateWidget,
};
registry.category("view_widgets").add("qty_at_date_widget", qtyAtDateWidget);

```

## File: static\src\widgets\qty_at_date_widget.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<template id="template" xml:space="preserve">

    <t t-name="sale_stock.QtyAtDate">
        <a t-att-tabindex="props.record.data.display_qty_widget ? '0' : '-1'"
            t-on-click="showPopup"
            t-att-class="!props.record.data.display_qty_widget ? 'invisible' : ''"
            t-attf-class="fa fa-area-chart cursor-pointer {{ calcData.forecasted_issue ? 'text-danger' : '' }}"
        />
    </t>

    <t t-name="sale_stock.QtyAtDatePopover">
        <div class="p-2">
        <h6>Availability</h6>
        <table class="table table-borderless table-sm">
            <tbody>
                <t t-if="!props.record.data.is_mto and ['draft', 'sent'].includes(props.record.data.state)">
                    <tr>
                        <td><strong>Forecasted Stock</strong><br/><small>On <span t-out="props.calcData.delivery_date"/></small></td>
                        <td class="text-end"><b t-out='props.record.data.virtual_available_at_date'/> <t t-out='props.record.data.product_uom[1]'/></td>
                    </tr>
                    <tr>
                        <td><strong>Available</strong><br /><small>All planned operations included</small></td>
                        <td class="text-end"><b t-out='props.record.data.free_qty_today' t-att-class="!props.calcData.will_be_fulfilled ? 'text-danger': ''"/> <t t-out='props.record.data.product_uom[1]'/></td>
                    </tr>
                </t>
                <t t-elif="props.record.data.is_mto and ['draft', 'sent'].includes(props.record.data.state)">
                    <tr>
                        <td><strong>Expected Delivery</strong></td>
                        <td class="oe-right"><span t-out="props.calcData.delivery_date"/></td>
                    </tr>
                    <tr>
                        <p>This product is replenished on demand.</p>
                    </tr>
                </t>
                <t t-elif="props.record.data.state == 'sale'">
                    <tr>
                        <td>
                            <strong>Reserved</strong><br/>
                        </td>
                        <td style="min-width: 50px; text-align: right;">
                            <b t-out='props.record.data.qty_available_today'/> <t t-out='props.record.data.product_uom[1]'/>
                        </td>
                    </tr>
                    <tr t-if="props.record.data.qty_available_today &lt; props.record.data.qty_to_deliver">
                        <td>
                            <span t-if="props.calcData.will_be_fulfilled and props.calcData.forecast_expected_date_str">
                                Remaining demand available at <b t-out="props.calcData.forecast_expected_date_str" t-att-class="props.record.data.scheduled_date &lt; props.record.data.forecast_expected_date ? 'text-danger' : ''"/>
                            </span>
                            <span t-elif="!props.calcData.will_be_fulfilled and props.calcData.forecast_expected_date_str" class="text-danger">
                                Not enough future availability
                            </span>
                            <span t-elif="!props.calcData.will_be_fulfilled" class="text-danger">
                                No future availability
                            </span>
                            <span t-else="">
                                Available in stock
                            </span>
                        </td>
                    </tr>
                </t>
            </tbody>
        </table>
        <button t-if="!props.record.data.is_mto" class="text-start btn btn-link"
            type="button" t-on-click="openForecast">
            <i class="oi oi-fw o_button_icon oi-arrow-right"></i>
            View Forecast
        </button>
        </div>
    </t>
</template>

```

## File: static\src\xml\delay_alert.xml

```xml
<templates>
    <div t-name="sale_stock.DelayAlertWidget" class="m-1">
        <p>The delivery
            <t t-foreach="props.late_elements" t-as="late_element" t-key="late_element.id">
                <a t-esc="late_element.name" href="#" t-on-click="openElement" t-att-element-id="late_element.id" t-att-element-model="late_element.model"/>,
            </t> will be late.
        </p>
    </div>
</templates>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="res_config_settings_view_form_stock" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.sale.stock.stock</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="stock.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <setting id="warning_info" position="after">
                <setting help="When to start shipping">
                    <field name="default_picking_policy" class="o_light_label w-auto" widget="selection"/>
                </setting>
            </setting>
            <xpath expr="//block[@id='schedule_info']" position="attributes">
                <attribute name="invisible">0</attribute>
            </xpath>
            <div id="sale_security_lead" position="replace">
                <setting company_dependent="1" help="Schedule deliveries earlier to avoid delays" documentation="/applications/inventory_and_mrp/inventory/management/planning/scheduled_dates.html" title="Margin of error for dates promised to customers. Products will be scheduled for procurement and delivery that many days earlier than the actual promised date, to cope with unexpected delays in the supply chain.">
                    <field name="use_security_lead"/>
                    <div class="content-group">
                        <div class="mt16" invisible="not use_security_lead">
                            <span>Move forward expected delivery dates by <field name="security_lead" class="oe_inline"/> days</span>
                        </div>
                    </div>
                </setting>
            </div>
        </field>
    </record>

</odoo>

```

## File: views\res_users_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="res_users_view_form_preferences" model="ir.ui.view">
            <field name="name">res.users.preferences.form.inherit</field>
            <field name="model">res.users</field>
            <field name="inherit_id" ref="base.view_users_form_simple_modif"/>
            <field name="arch" type="xml">
                <group name="signature" position="after">
                    <group name="Warehouses">
                        <field name="property_warehouse_id" groups="stock.group_stock_multi_warehouses"/>
                    </group>
                </group>
            </field>
        </record>

        <record id="res_users_view_simple_form" model="ir.ui.view">
            <field name="name">res.users.simple.form.inherit</field>
            <field name="model">res.users</field>
            <field name="inherit_id" ref="base.view_users_simple_form"/>
            <field name="arch" type="xml">
                <group name="phone_numbers" position="after">
                    <group name="Warehouses">
                        <field name="property_warehouse_id" groups="stock.group_stock_multi_warehouses"/>
                    </group>
                </group>
            </field>
        </record>

        <record id="res_users_view_form" model="ir.ui.view">
            <field name="name">res.users.form.inherit</field>
            <field name="model">res.users</field>
            <field name="inherit_id" ref="base.view_users_form"/>
            <field name="arch" type="xml">
                <group name="messaging" position="after">
                    <group name="Warehouses" string="Inventory" groups="stock.group_stock_multi_warehouses">
                        <field name="property_warehouse_id" groups="stock.group_stock_multi_warehouses"/>
                    </group>
                </group>
            </field>
        </record>

    </data>
</odoo>
```

## File: views\sale_order_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data>
        <record id="view_order_form_inherit_sale_stock" model="ir.ui.view">
            <field name="name">sale.order.form.sale.stock</field>
            <field name="model">sale.order</field>
            <field name="inherit_id" ref="sale.view_order_form"/>
            <field name="arch" type="xml">
                <xpath expr="//button[@name='action_view_invoice']" position="before">
                    <button type="object"
                        name="action_view_delivery"
                        class="oe_stat_button"
                        icon="fa-truck"
                        invisible="delivery_count == 0" groups="stock.group_stock_user">
                        <field name="delivery_count" widget="statinfo" string="Delivery"/>
                    </button>
                </xpath>
                <xpath expr="//group[@name='sale_shipping']" position="attributes">
                    <attribute name="groups"></attribute><!-- Remove the res.group on the group and set it on the field directly-->
                    <attribute name="string">Delivery</attribute>
                </xpath>
                <xpath expr="//label[@for='commitment_date']" position="before">
                    <field name="warehouse_id" invisible="1" readonly="state == 'sale'"/>  <!-- needed for js logic -->
                    <field name="warehouse_id" options="{'no_create': True}" force_save="1" readonly="state == 'sale'"/>
                    <field name="incoterm" options="{'no_open': True, 'no_create': True}"/>
                    <field name="incoterm_location"/>
                    <field name="picking_policy" required="True" readonly="state not in ['draft', 'sent']"/>
                </xpath>
                <xpath expr="//span[@name='expected_date_span']" position="attributes">
                    <attribute name="invisible">effective_date and commitment_date</attribute>
                </xpath>
                <xpath expr="//div[@name='commitment_date_div']" position="replace">
                    <div class="o_row">
                        <field name="commitment_date"/>
                        <span class="text-muted" invisible="effective_date and commitment_date">Expected: <field name="expected_date" class="oe_inline"/></span>
                    </div>
                    <field name="effective_date" invisible="not effective_date"/>
                </xpath>
                <field name="effective_date" position="after">
                    <field name="delivery_status" invisible="state != 'sale'"/>
                </field>
                <xpath expr="//page[@name='other_information']//field[@name='expected_date']" position="after">
                    <field name="show_json_popover" invisible="1"/>
                    <field string=" " name="json_popover" widget="stock_rescheduling_popover" invisible="not show_json_popover"/>
                </xpath>
                <xpath expr="//field[@name='order_line']/form/group/group/field[@name='analytic_distribution']" position="before">
                    <field name="route_id" groups="stock.group_adv_location" options="{'no_create': True}"/>
                </xpath>
                <xpath expr="//field[@name='order_line']/list/field[@name='analytic_distribution']" position="after">
                    <field name="route_id" groups="stock.group_adv_location" options="{'no_create': True}" optional="hide"/>
                </xpath>
           </field>
        </record>

        <record id="sale_order_tree" model="ir.ui.view">
            <field name="name">sale.order.list.inherit.sale.stock</field>
            <field name="model">sale.order</field>
            <field name="inherit_id" ref="sale.sale_order_tree"/>
            <field name="arch" type="xml">
                <field name="tag_ids" position="after">
                    <field name="warehouse_id"
                           options="{'no_create': True}"
                           groups="stock.group_stock_multi_warehouses"
                           optional="hide" readonly="state == 'sale'"/>
                </field>
            </field>
        </record>

        <record id="view_order_tree" model="ir.ui.view">
            <field name="name">sale.order.list.inherit.sale.stock</field>
            <field name="model">sale.order</field>
            <field name="inherit_id" ref="sale.view_order_tree"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='commitment_date']" position="attributes">
                    <attribute name="decoration-danger">
                        (
                            (
                                commitment_date &lt; effective_date
                                or commitment_date &lt; datetime.datetime.combine(
                                    datetime.date.today(),
                                    datetime.time(0,0,0)
                                ).to_utc().strftime('%Y-%m-%d %H:%M:%S')
                            )
                            and delivery_status in ['pending', 'started']
                            and effective_date &lt;= commitment_date
                            or delivery_status == 'partial'
                        )
                    </attribute>
                </xpath>
                <field name="invoice_status" position="before">
                    <field name="effective_date" column_invisible="True"/>
                    <field name="delivery_status"
                        decoration-success="delivery_status == 'full'"
                        decoration-warning="delivery_status == 'partial'"
                        decoration-info="delivery_status in ['pending', 'started']"
                        widget="badge"
                        optional="hide"/>
                </field>
            </field>
        </record>

        <record id="view_order_line_tree_inherit_sale_stock" model="ir.ui.view">
            <field name="name">sale.order.line.list.sale.stock.location</field>
            <field name="inherit_id" ref="sale.view_order_line_tree"/>
            <field name="model">sale.order.line</field>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='price_subtotal']" position="before">
                    <field name="route_id" groups="stock.group_adv_location" options="{'no_create': True}"/>
                </xpath>
            </field>
        </record>

        <record id="view_order_form_inherit_sale_stock_qty" model="ir.ui.view">
            <field name="name">sale.order.line.list.sale.stock.qty</field>
            <field name="inherit_id" ref="sale.view_order_form"/>
            <field name="model">sale.order</field>
            <field name="arch" type="xml">
                <xpath expr="//page/field[@name='order_line']/form/group/group/div[@name='ordered_qty']/field[@name='product_uom']" position="after">
                    <!-- below fields are used in the widget qty_at_date_widget -->
                    <field name="virtual_available_at_date" invisible="1"/>
                    <field name="qty_available_today" invisible="1"/>
                    <field name="free_qty_today" invisible="1"/>
                    <field name="scheduled_date" invisible="1"/>
                    <field name="forecast_expected_date" invisible="1"/>
                    <field name="warehouse_id" invisible="1"/>
                    <field name="move_ids" invisible="1"/>
                    <field name="qty_to_deliver" invisible="1"/>
                    <field name="is_mto" invisible="1"/>
                    <field name="display_qty_widget" invisible="1"/>
                    <widget name="qty_at_date_widget"/>
                </xpath>
                <xpath expr="//page/field[@name='order_line']/list/field[@name='qty_delivered']" position="after">
                    <!-- below fields are used in the widget qty_at_date_widget -->
                    <field name="virtual_available_at_date" column_invisible="True"/>
                    <field name="qty_available_today" column_invisible="True"/>
                    <field name="free_qty_today" column_invisible="True"/>
                    <field name="scheduled_date" column_invisible="True"/>
                    <field name="forecast_expected_date" column_invisible="True"/>
                    <field name="warehouse_id" column_invisible="True"/>
                    <field name="move_ids" column_invisible="True"/>
                    <field name="qty_to_deliver" column_invisible="True"/>
                    <field name="is_mto" column_invisible="True"/>
                    <field name="display_qty_widget" column_invisible="True"/>
                    <widget name="qty_at_date_widget" width="20px"/>
                </xpath>
            </field>
        </record>

        </data>
</odoo>

```

## File: views\sale_stock_portal_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="sale_order_portal_content_inherit_sale_stock"
        name="Orders Shipping Followup"
        inherit_id="sale.sale_order_portal_content">
        <tbody id="sale_info_table" position="inside">
            <tr t-if="sale_order.incoterm">
                <th class="pb-0">Incoterm:</th>
                <td class="w-100 pb-0 text-wrap">
                    <p t-if="sale_order.incoterm_location">
                        <span t-field="sale_order.incoterm.code"/> <br/>
                        <span t-field="sale_order.incoterm_location"/>
                    </p>
                    <p t-else="" t-field="sale_order.incoterm.code" class="m-0"/>
                </td>
            </tr>
        </tbody>

        <div id="sale_invoices" position="after">
            <t t-set="delivery_orders" t-value="sale_order.picking_ids.filtered(lambda picking: picking.picking_type_id.code == 'outgoing')"/>
            <t t-set="latest_delivery_orders" t-value="delivery_orders.sorted('date', reverse=True)[:3]"/>
            <div t-if="delivery_orders" class="col-12 col-lg-6 mb-4">
                <h5 class="mb-1">Last Delivery Orders</h5>
                <hr class="mt-1 mb-2"/>
                <div class="d-flex flex-column gap-2">
                    <t t-foreach="latest_delivery_orders" t-as="picking">
                        <t t-set="delivery_report_url"
                           t-value="'/my/picking/pdf/%s?%s' % (picking.id, keep_query())"/>
                        <div name="delivery_order"
                            class="d-flex flex-column">
                            <div name="delivery_details" class="d-flex align-items-center justify-content-between">
                                <a t-att-href="delivery_report_url">
                                    <span t-esc="picking.name"/>
                                </a>
                                <div t-if="picking.state == 'done'">
                                    <span class="small badge rounded-pill text-bg-success orders_label_text_align">
                                        <i class="fa fa-fw fa-truck"/> Shipped
                                    </span>
                                    <a class="badge rounded-pill text-bg-secondary orders_label_text_align" target="_blank"
                                        t-att-href="'/my/picking/return/pdf/%s?%s' % (picking.id, keep_query())">
                                        RETURN
                                    </a>
                                </div>
                                <span t-elif="picking.state == 'cancel'"
                                    class="small badge rounded-pill text-bg-danger orders_label_text_align">
                                    <i class="fa fa-fw fa-times"/>Cancelled
                                </span>
                                <span t-elif="picking.state in ['draft', 'waiting', 'confirmed', 'assigned']"
                                    class="small badge rounded-pill text-bg-info orders_label_text_align">
                                    <i class="fa fa-fw fa-clock-o"/>Preparation
                                </span>
                            </div>
                            <div class="small d-lg-inline-block" t-if="picking.date_done or picking.scheduled_date">
                                Date:
                                <span class="text-muted"
                                        t-field="picking.date_done"
                                        t-options="{'date_only': True}"/>
                                <span t-if="picking.state in ['draft', 'waiting', 'confirmed', 'assigned']"
                                        class="text-muted"
                                        t-field="picking.scheduled_date"
                                        t-options="{'date_only': True}"/>
                            </div>
                        </div>
                    </t>
                </div>
            </div>
            <t t-set="returns" t-value="delivery_orders.return_ids.sorted('date', reverse=True)"/>
            <div t-if="returns" class="col-12 col-lg-6 mb-4">
                <h4 class="mb-1">Returns</h4>
                <hr class="mt-1 mb-2"/>
                <t t-foreach="returns" t-as="picking">
                    <t t-set="delivery_report_url"
                        t-value="'/my/picking/pdf/%s?%s' % (picking.id, keep_query())"/>
                    <div name="return">
                        <div name="return_details" class="d-flex justify-content-between align-items-center">
                            <a t-att-href="delivery_report_url">
                                <span t-esc="picking.name"/>
                            </a>
                            <span t-if="picking.state == 'done'"
                                class="small badge rounded-pill text-bg-success orders_label_text_align">
                                <i class="fa fa-fw fa-truck"/> Received
                            </span>
                            <span t-elif="picking.state == 'cancel'"
                                class="small badge rounded-pill text-bg-danger orders_label_text_align">
                                <i class="fa fa-fw fa-times"/> Cancelled
                            </span>
                            <span t-elif="picking.state in ['draft', 'waiting', 'confirmed', 'assigned']"
                                class="small badge rounded-pill text-bg-info orders_label_text_align">
                                <i class="fa fa-fw fa-clock-o"/> Awaiting arrival
                            </span>
                        </div>
                        <div class="small d-lg-inline-block">
                            Date:
                            <span class="text-muted"
                                t-field="picking.date_done"
                                t-options="{'date_only': True}"/>
                            <span t-if="picking.state in ['draft', 'waiting', 'confirmed', 'assigned']"
                                class="text-muted"
                                t-field="picking.scheduled_date"
                                t-options="{'date_only': True}"/>
                        </div>
                    </div>
                </t>
            </div>
        </div>
    </template>

</odoo>

```

## File: views\stock_lot_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="stock_production_lot_view_form" model="ir.ui.view">
        <field name="name">stock.production.lot.view.form</field>
        <field name="model">stock.lot</field>
        <field name="inherit_id" ref="stock.view_production_lot_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[hasclass('oe_button_box')]/button" position="before">
                <button class="oe_stat_button" name="action_view_so"
                        type="object" icon="fa-pencil-square-o" help="Sale Orders"
                        invisible="sale_order_count == 0 or not display_complete">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_value">
                            <field name="sale_order_count" widget="statinfo" nolabel="1" class="mr4"/>
                        </span>
                        <span class="o_stat_text">Sales</span>
                    </div>
                </button>
            </xpath>
            <xpath expr="//group[@name='main_group']" position="after">
                <group>
                    <field name="sale_order_ids" widget="many2many"
                           readonly="True"
                           invisible="not sale_order_ids or display_complete">
                        <list>
                            <field name="name" readonly="state != 'draft'"/>
                            <field name="partner_id" readonly="state in ['cancel', 'sale']"/>
                            <field name="date_order" readonly="state in ['cancel', 'sale']"/>
                            <field name="state" column_invisible="True"/>
                        </list>
                    </field>
                </group>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\stock_picking_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="sale_stock_help_message_template" inherit_id="stock.help_message_template">
        <xpath expr="//p[@name='top_help_message']/t[@name='delivery_message']" position="replace">
            <t t-if="picking_type_code == 'outgoing'">
                No delivery yet! Automate them with sales orders.
            </t>
        </xpath>
        <xpath expr="//p[@name='bottom_help_message']/t[@name='delivery_message']" position="replace">
            <t t-if="picking_type_code == 'outgoing'">
                Reduce back orders with reservations, locations management, smart
                replenishment propositions, shipping connectors, barcode scanner, etc.
            </t>
        </xpath>
        <xpath expr="//p[@name='bottom_help_message']" position="after">
            <a t-if="picking_type_code == 'outgoing'" class="btn btn-secondary" name="sale.action_orders" type="action">Sales Orders</a>
        </xpath>
    </template>
    <record model="ir.ui.view" id="view_picking_form">
    <field name="name">sale.stock.view.picking.form</field>
        <field name="model">stock.picking</field>
        <field name="inherit_id" ref="stock.view_picking_form"></field>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='group_id']" position="after">
                <field name="sale_id"/>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\stock_route_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="stock_location_route_view_form_inherit_sale_stock" model="ir.ui.view">
        <field name="name">stock.route.form</field>
        <field name="model">stock.route</field>
        <field name="inherit_id" ref="stock.stock_location_route_form_view"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='route_selector']/group[last()]" position="inside">
                <field name="sale_selectable" string="Sales Order Lines"/>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: wizard\sale_order_cancel.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class SaleOrderCancel(models.TransientModel):
    _inherit = 'sale.order.cancel'

    display_delivery_alert = fields.Boolean('Delivery Alert', compute='_compute_display_delivery_alert')

    @api.depends('order_id')
    def _compute_display_delivery_alert(self):
        for wizard in self:
            out_pickings = wizard.order_id.picking_ids.filtered(lambda p: p.picking_type_code == 'outgoing')
            wizard.display_delivery_alert = bool(any(picking.state == 'done' for picking in out_pickings))

```

## File: wizard\sale_order_cancel_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="sale_order_cancel_view_form_inherit" model="ir.ui.view">
        <field name="name">sale.order.cancel.form.inherit</field>
        <field name="model">sale.order.cancel</field>
        <field name="inherit_id" ref="sale.sale_order_cancel_view_form"/>
        <field name="arch" type="xml">
            <span id="display_invoice_alert" position="after">
                <field name="display_delivery_alert" invisible="1"/>
                <span invisible="not display_delivery_alert">
                    Some deliveries are already done. Returns can be created from the Delivery Orders.
                </span>
            </span>
        </field>
    </record>
</odoo>

```

## File: wizard\stock_return_picking.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class ReturnPicking(models.TransientModel):
    _inherit = 'stock.return.picking'

    def _get_proc_values(self, line):
        vals = super()._get_proc_values(line)
        vals['sale_line_id'] = line.move_id.sale_line_id.id
        return vals

```

## File: wizard\stock_rules_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class StockRulesReport(models.TransientModel):
    _inherit = 'stock.rules.report'

    so_route_ids = fields.Many2many('stock.route', string='Apply specific routes',
        domain="[('sale_selectable', '=', True)]", help="Choose to apply SO lines specific routes.")

    def _prepare_report_data(self):
        data = super(StockRulesReport, self)._prepare_report_data()
        data['so_route_ids'] = self.so_route_ids.ids
        return data

```

## File: wizard\stock_rules_report_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_stock_rules_report_sale" model="ir.ui.view">
        <field name="name">Stock Rules Report Sale</field>
        <field name="model">stock.rules.report</field>
        <field name="inherit_id" ref="stock.view_stock_rules_report"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='warehouse_ids']" position="after">
                <field name="so_route_ids" widget="many2many_tags" groups="stock.group_adv_location"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import stock_rules_report
from . import sale_order_cancel
from . import stock_return_picking

```

