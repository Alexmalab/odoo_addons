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
        'views/stock_views.xml',
        'views/res_config_settings_views.xml',
        'views/sale_stock_portal_template.xml',
        'views/stock_production_lot_views.xml',
        'views/res_users_views.xml',
        'report/report_stock_forecasted.xml',
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
            'sale_stock/static/src/js/**/*',
        ],
        'web.assets_qweb': [
            'sale_stock/static/src/xml/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\portal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import exceptions, SUPERUSER_ID
from odoo.addons.sale.controllers.portal import CustomerPortal
from odoo.http import request, route
from odoo.tools import consteq


class SaleStockPortal(CustomerPortal):

    def _stock_picking_check_access(self, picking_id, access_token=None):
        picking = request.env['stock.picking'].browse([picking_id])
        picking_sudo = picking.sudo()
        try:
            picking.check_access_rights('read')
            picking.check_access_rule('read')
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
        except exceptions.AccessError:
            return request.redirect('/my')

        # print report as SUPERUSER, since it require access to product, taxes, payment term etc.. and portal does not have those access rights.
        pdf = request.env.ref('stock.action_report_delivery').with_user(SUPERUSER_ID)._render_qweb_pdf([picking_sudo.id])[0]
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
            <field name="pricelist_id" ref="product.list0"/>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="sale.utm_source_sale_order_0"/>
            <field name="date_order" eval="(datetime.now() + relativedelta(days=1)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="warehouse_id" ref="stock.warehouse0"/>
        </record>

        <record id="sale_order_line_42" model="sale.order.line">
            <field name="order_id" ref="sale_order_19"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_25').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_25"/>
            <field name="product_uom_qty">5</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">295.00</field>
        </record>

        <record id="sale_order_line_43" model="sale.order.line">
            <field name="order_id" ref="sale_order_19"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_10').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_10"/>
            <field name="product_uom_qty">5</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">140.00</field>
        </record>

        <record id="sale_order_20" model="sale.order">
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="partner_invoice_id" ref="base.res_partner_address_25"/>
            <field name="partner_shipping_id" ref="base.res_partner_address_25"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="pricelist_id" ref="product.list0"/>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="sale.utm_source_sale_order_0"/>
            <field name="date_order" eval="(datetime.now() + relativedelta(days=2)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="warehouse_id" ref="stock.warehouse0"/>
        </record>

        <record id="sale_order_line_44" model="sale.order.line">
            <field name="order_id" ref="sale_order_20"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_25').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_25"/>
            <field name="product_uom_qty">5</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">295.00</field>
        </record>

        <record id="sale_order_21" model="sale.order">
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="partner_invoice_id" ref="base.res_partner_address_25"/>
            <field name="partner_shipping_id" ref="base.res_partner_address_25"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="pricelist_id" ref="product.list0"/>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="sale.utm_source_sale_order_0"/>
            <field name="date_order" eval="(datetime.now() + relativedelta(days=3)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="warehouse_id" ref="stock.warehouse0"/>
        </record>

        <record id="sale_order_line_45" model="sale.order.line">
            <field name="order_id" ref="sale_order_21"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_25').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_25"/>
            <field name="product_uom_qty">10</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">295.00</field>
        </record>

        <record id="sale_order_line_46" model="sale.order.line">
            <field name="order_id" ref="sale_order_21"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_10').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_10"/>
            <field name="product_uom_qty">10</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">140.00</field>
        </record>

        <record id="sale_order_22" model="sale.order">
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="partner_invoice_id" ref="base.res_partner_address_25"/>
            <field name="partner_shipping_id" ref="base.res_partner_address_25"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="pricelist_id" ref="product.list0"/>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="sale.utm_source_sale_order_0"/>
            <field name="date_order" eval="(datetime.now() + relativedelta(days=3)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="warehouse_id" ref="stock.warehouse0"/>
        </record>

        <record id="sale_order_line_47" model="sale.order.line">
            <field name="order_id" ref="sale_order_22"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_5').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_5"/>
            <field name="product_uom_qty">10</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
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
    	<record id="stock.route_warehouse0_mto" model="stock.location.route">
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

from odoo import fields, models
from odoo.tools import float_is_zero, float_compare
from odoo.tools.misc import formatLang


class AccountMove(models.Model):
    _inherit = 'account.move'

    def _stock_account_get_last_step_stock_moves(self):
        """ Overridden from stock_account.
        Returns the stock moves associated to this invoice."""
        rslt = super(AccountMove, self)._stock_account_get_last_step_stock_moves()
        for invoice in self.filtered(lambda x: x.move_type == 'out_invoice'):
            rslt += invoice.mapped('invoice_line_ids.sale_line_ids.move_ids').filtered(lambda x: x.state == 'done' and x.location_dest_id.usage == 'customer')
        for invoice in self.filtered(lambda x: x.move_type == 'out_refund'):
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

        current_invoice_amls = self.invoice_line_ids.filtered(lambda aml: not aml.display_type and aml.product_id and aml.product_id.type in ('consu', 'product') and aml.quantity)
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
            if sml.product_id not in invoiced_products or 'customer' not in {sml.location_id.usage, sml.location_dest_id.usage}:
                continue
            product = sml.product_id
            product_uom = product.uom_id
            qty_done = sml.product_uom_id._compute_quantity(sml.qty_done, product_uom)

            # is it a stock return considering the document type (should it be it thought of as positively or negatively?)
            is_stock_return = (
                    self.move_type == 'out_invoice' and (sml.location_id.usage, sml.location_dest_id.usage) == ('customer', 'internal')
                    or
                    self.move_type == 'out_refund' and (sml.location_id.usage, sml.location_dest_id.usage) == ('internal', 'customer')
            )
            if is_stock_return:
                returned_qty = min(qties_per_lot[sml.lot_id], qty_done)
                qties_per_lot[sml.lot_id] -= returned_qty
                qty_done = returned_qty - qty_done

            previous_qty_invoiced = previous_qties_invoiced[product]
            previous_qty_delivered = previous_qties_delivered[product]
            # If we return more than currently delivered (i.e., qty_done < 0), we remove the surplus
            # from the previously delivered (and qty_done becomes zero). If it's a delivery, we first
            # try to reach the previous_qty_invoiced
            if float_compare(qty_done, 0, precision_rounding=product_uom.rounding) < 0 or \
                    float_compare(previous_qty_delivered, previous_qty_invoiced, precision_rounding=product_uom.rounding) < 0:
                previously_done = qty_done if is_stock_return else min(previous_qty_invoiced - previous_qty_delivered, qty_done)
                previous_qties_delivered[product] += previously_done
                qty_done -= previously_done

            qties_per_lot[sml.lot_id] += qty_done

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


class AccountMoveLine(models.Model):
    _inherit = "account.move.line"

    def _sale_can_be_reinvoice(self):
        self.ensure_one()
        return not self.is_anglo_saxon_line and super(AccountMoveLine, self)._sale_can_be_reinvoice()

    def _stock_account_get_anglo_saxon_price_unit(self):
        self.ensure_one()
        price_unit = super(AccountMoveLine, self)._stock_account_get_anglo_saxon_price_unit()

        so_line = self.sale_line_ids and self.sale_line_ids[-1] or False
        if so_line:
            is_line_reversing = self.move_id.move_type == 'out_refund'
            qty_to_invoice = self.product_uom_id._compute_quantity(self.quantity, self.product_id.uom_id)
            account_moves = so_line.invoice_lines.move_id.filtered(lambda m: m.state == 'posted' and bool(m.reversed_entry_id) == is_line_reversing)

            posted_cogs = account_moves.line_ids.filtered(lambda l: l.is_anglo_saxon_line and l.product_id == self.product_id and l.balance > 0)
            qty_invoiced = sum([line.product_uom_id._compute_quantity(line.quantity, line.product_id.uom_id) for line in posted_cogs])
            value_invoiced = sum(posted_cogs.mapped('balance'))

            reversal_cogs = posted_cogs.move_id.reversal_move_id.line_ids.filtered(lambda l: l.is_anglo_saxon_line and l.product_id == self.product_id and l.balance > 0)
            qty_invoiced -= sum([line.product_uom_id._compute_quantity(line.quantity, line.product_id.uom_id) for line in reversal_cogs])
            value_invoiced -= sum(reversal_cogs.mapped('balance'))

            product = self.product_id.with_company(self.company_id).with_context(is_returned=is_line_reversing, value_invoiced=value_invoiced)
            average_price_unit = product._compute_average_price(qty_invoiced, qty_to_invoice, so_line.move_ids)
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

    @api.onchange('type')
    def _onchange_type(self):
        """ We want to prevent storable product to be expensed, since it make no sense as when confirm
            expenses, the product is already out of our stock.
        """
        res = super(ProductTemplate, self)._onchange_type()
        if self.type == 'product':
            self.expense_policy = 'no'
            self.service_type = 'manual'
        return res

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
    group_display_incoterm = fields.Boolean("Incoterms", implied_group='sale_stock.group_display_incoterm')
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
        # !!! Any change to the following search domain should probably
        # be also applied in sale_stock/models/sale_order.py/_init_column.
        return self.env['stock.warehouse'].search([('company_id', '=', self.env.company.id)], limit=1)

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
from datetime import timedelta
from collections import defaultdict

from odoo import api, fields, models, _
from odoo.tools import float_compare, float_round
from odoo.exceptions import UserError


_logger = logging.getLogger(__name__)


class SaleOrder(models.Model):
    _inherit = "sale.order"

    @api.model
    def _default_warehouse_id(self):
        # !!! Any change to the default value may have to be repercuted
        # on _init_column() below.
        return self.env.user._get_default_warehouse_id()

    incoterm = fields.Many2one(
        'account.incoterms', 'Incoterm',
        help="International Commercial Terms are a series of predefined commercial terms used in international transactions.")
    picking_policy = fields.Selection([
        ('direct', 'As soon as possible'),
        ('one', 'When all products are ready')],
        string='Shipping Policy', required=True, readonly=True, default='direct',
        states={'draft': [('readonly', False)], 'sent': [('readonly', False)]}
        ,help="If you deliver all products at once, the delivery order will be scheduled based on the greatest "
        "product lead time. Otherwise, it will be based on the shortest.")
    warehouse_id = fields.Many2one(
        'stock.warehouse', string='Warehouse',
        required=True, readonly=True, states={'draft': [('readonly', False)], 'sent': [('readonly', False)]},
        default=_default_warehouse_id, check_company=True)
    picking_ids = fields.One2many('stock.picking', 'sale_id', string='Transfers')
    delivery_count = fields.Integer(string='Delivery Orders', compute='_compute_picking_ids')
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
        value = field.convert_to_column(value, self)
        if value is not None:
            _logger.debug("Table '%s': setting default value of new column %s to %r",
                self._table, column_name, value)
            query = 'UPDATE "%s" SET "%s"=%s WHERE "%s" IS NULL' % (
                self._table, column_name, field.column_format, column_name)
            self._cr.execute(query, (value,))

    @api.depends('picking_ids.date_done')
    def _compute_effective_date(self):
        for order in self:
            pickings = order.picking_ids.filtered(lambda x: x.state == 'done' and x.location_dest_id.usage == 'customer')
            dates_list = [date for date in pickings.mapped('date_done') if date]
            order.effective_date = min(dates_list, default=False)

    @api.depends('picking_policy')
    def _compute_expected_date(self):
        super(SaleOrder, self)._compute_expected_date()
        for order in self:
            dates_list = []
            for line in order.order_line.filtered(lambda x: x.state != 'cancel' and not x._is_delivery() and not x.display_type):
                dt = line._expected_date()
                dates_list.append(dt)
            if dates_list:
                expected_date = min(dates_list) if order.picking_policy == 'direct' else max(dates_list)
                order.expected_date = fields.Datetime.to_string(expected_date)

    @api.model
    def create(self, vals):
        if 'warehouse_id' not in vals and 'company_id' in vals:
            user = self.env['res.users'].browse(vals.get('user_id', False))
            vals['warehouse_id'] = user.with_company(vals.get('company_id'))._get_default_warehouse_id().id
        return super().create(vals)

    def write(self, values):
        if values.get('order_line') and self.state == 'sale':
            for order in self:
                pre_order_line_qty = {order_line: order_line.product_uom_qty for order_line in order.mapped('order_line') if not order_line.is_expense}

        if values.get('partner_shipping_id'):
            new_partner = self.env['res.partner'].browse(values.get('partner_shipping_id'))
            for record in self:
                picking = record.mapped('picking_ids').filtered(lambda x: x.state not in ('done', 'cancel'))
                addresses = (record.partner_shipping_id.display_name, new_partner.display_name)
                message = _("""The delivery address has been changed on the Sales Order<br/>
                        From <strong>"%s"</strong> To <strong>"%s"</strong>,
                        You should probably update the partner on this document.""") % addresses
                picking.activity_schedule('mail.mail_activity_data_warning', note=message, user_id=self.env.user.id)

        if values.get('commitment_date'):
            # protagate commitment_date as the deadline of the related stock move.
            # TODO: Log a note on each down document
            self.order_line.move_ids.date_deadline = fields.Datetime.to_datetime(values.get('commitment_date'))

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
                    documents = {k:v for k, v in documents.items() if k[0].state != 'cancel'}
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

    @api.onchange('company_id')
    def _onchange_company_id(self):
        if self.company_id:
            warehouse_id = self.env['ir.default'].get_model_defaults('sale.order').get('warehouse_id')
            self.warehouse_id = warehouse_id or self.user_id.with_company(self.company_id.id)._get_default_warehouse_id().id

    @api.onchange('user_id')
    def onchange_user_id(self):
        super().onchange_user_id()
        if self.state in ['draft','sent']:
            self.warehouse_id = self.user_id.with_company(self.company_id.id)._get_default_warehouse_id().id

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
                    'Do not forget to change the partner on the following delivery orders: %s'
                ) % (','.join(pickings.mapped('name')))
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
        action['context'] = dict(self._context, default_partner_id=self.partner_id.id, default_picking_type_id=picking_id.picking_type_id.id, default_origin=self.name, default_group_id=picking_id.group_id.id)
        return action

    def _prepare_invoice(self):
        invoice_vals = super(SaleOrder, self)._prepare_invoice()
        invoice_vals['invoice_incoterm_id'] = self.incoterm.id
        return invoice_vals

    @api.model
    def _get_customer_lead(self, product_tmpl_id):
        super(SaleOrder, self)._get_customer_lead(product_tmpl_id)
        return product_tmpl_id.sale_delay

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
            return self.env.ref('sale_stock.exception_on_so')._render(values=values)

        self.env['stock.picking']._log_activity(_render_note_exception_quantity_so, documents)

    def _show_cancel_wizard(self):
        res = super(SaleOrder, self)._show_cancel_wizard()
        for order in self:
            if any(picking.state == 'done' for picking in order.picking_ids) and not order._context.get('disable_cancel_warning'):
                return True
        return res

class SaleOrderLine(models.Model):
    _inherit = 'sale.order.line'

    qty_delivered_method = fields.Selection(selection_add=[('stock_move', 'Stock Moves')])
    route_id = fields.Many2one('stock.location.route', string='Route', domain=[('sale_selectable', '=', True)], ondelete='restrict', check_company=True)
    move_ids = fields.One2many('stock.move', 'sale_line_id', string='Stock Moves')
    product_type = fields.Selection(related='product_id.detailed_type')
    virtual_available_at_date = fields.Float(compute='_compute_qty_at_date', digits='Product Unit of Measure')
    scheduled_date = fields.Datetime(compute='_compute_qty_at_date')
    forecast_expected_date = fields.Datetime(compute='_compute_qty_at_date')
    free_qty_today = fields.Float(compute='_compute_qty_at_date', digits='Product Unit of Measure')
    qty_available_today = fields.Float(compute='_compute_qty_at_date')
    warehouse_id = fields.Many2one(related='order_id.warehouse_id')
    qty_to_deliver = fields.Float(compute='_compute_qty_to_deliver', digits='Product Unit of Measure')
    is_mto = fields.Boolean(compute='_compute_is_mto')
    display_qty_widget = fields.Boolean(compute='_compute_qty_to_deliver')

    @api.depends('product_type', 'product_uom_qty', 'qty_delivered', 'state', 'move_ids', 'product_uom')
    def _compute_qty_to_deliver(self):
        """Compute the visibility of the inventory widget."""
        for line in self:
            line.qty_to_deliver = line.product_uom_qty - line.qty_delivered
            if line.state in ('draft', 'sent', 'sale') and line.product_type == 'product' and line.product_uom and line.qty_to_deliver > 0:
                if line.state == 'sale' and not line.move_ids:
                    line.display_qty_widget = False
                else:
                    line.display_qty_widget = True
            else:
                line.display_qty_widget = False

    @api.depends(
        'product_id', 'customer_lead', 'product_uom_qty', 'product_uom', 'order_id.commitment_date',
        'move_ids', 'move_ids.forecast_expected_date', 'move_ids.forecast_availability')
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
            for move in line.move_ids
            if move.product_id == line.product_id
        }
        all_moves = self.env['stock.move'].browse(all_move_ids)
        forecast_expected_date_per_move = dict(all_moves.mapped(lambda m: (m.id, m.forecast_expected_date)))
        # If the state is already in sale the picking is created and a simple forecasted quantity isn't enough
        # Then used the forecasted data of the related stock.move
        for line in self.filtered(lambda l: l.state == 'sale'):
            if not line.display_qty_widget:
                continue
            moves = line.move_ids.filtered(lambda m: m.product_id == line.product_id)
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
                line.qty_available_today += move.product_uom._compute_quantity(move.reserved_availability, line.product_uom)
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
            product_qties = lines.mapped('product_id').with_context(to_date=scheduled_date, warehouse=warehouse).read([
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

    @api.depends('product_id', 'route_id', 'order_id.warehouse_id', 'product_id.route_ids')
    def _compute_is_mto(self):
        """ Verify the route of the product based on the warehouse
            set 'is_available' at True if the product availibility in stock does
            not need to be verified, which is the case in MTO, Cross-Dock or Drop-Shipping
        """
        self.is_mto = False
        for line in self:
            if not line.display_qty_widget:
                continue
            product = line.product_id
            product_routes = line.route_id or (product.route_ids + product.categ_id.total_route_ids)

            # Check MTO
            mto_route = line.order_id.warehouse_id.mto_pull_id.route_id
            if not mto_route:
                try:
                    mto_route = self.env['stock.warehouse']._find_global_route('stock.route_warehouse0_mto', _('Make To Order'))
                except UserError:
                    # if route MTO not found in ir_model_data, we treat the product as in MTS
                    pass

            if mto_route and mto_route in product_routes:
                line.is_mto = True
            else:
                line.is_mto = False

    @api.depends('product_id')
    def _compute_qty_delivered_method(self):
        """ Stock module compute delivered qty for product [('type', 'in', ['consu', 'product'])]
            For SO line coming from expense, no picking should be generate: we don't manage stock for
            thoses lines, even if the product is a storable.
        """
        super(SaleOrderLine, self)._compute_qty_delivered_method()

        for line in self:
            if not line.is_expense and line.product_id.type in ['consu', 'product']:
                line.qty_delivered_method = 'stock_move'

    @api.depends('move_ids.state', 'move_ids.scrapped', 'move_ids.quantity_done', 'move_ids.product_uom')
    def _compute_qty_delivered(self):
        super(SaleOrderLine, self)._compute_qty_delivered()

        for line in self:  # TODO: maybe one day, this should be done in SQL for performance sake
            if line.qty_delivered_method == 'stock_move':
                qty = 0.0
                outgoing_moves, incoming_moves = line._get_outgoing_incoming_moves()
                for move in outgoing_moves:
                    if move.state != 'done':
                        continue
                    qty += move.product_uom._compute_quantity(move.quantity_done, line.product_uom, rounding_method='HALF-UP')
                for move in incoming_moves:
                    if move.state != 'done':
                        continue
                    qty -= move.product_uom._compute_quantity(move.quantity_done, line.product_uom, rounding_method='HALF-UP')
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
            lines._action_launch_stock_rule(previous_product_uom_qty)
        if 'customer_lead' in values and self.state == 'sale' and not self.order_id.commitment_date:
            # Propagate deadline on related stock move
            self.move_ids.date_deadline = self.order_id.date_order + timedelta(days=self.customer_lead or 0.0)
        return res

    @api.depends('order_id.state')
    def _compute_invoice_status(self):
        def check_moves_state(moves):
            # All moves states are either 'done' or 'cancel', and there is at least one 'done'
            at_least_one_done = False
            for move in moves:
                if move.state not in ['done', 'cancel']:
                    return False
                at_least_one_done = at_least_one_done or move.state == 'done'
            return at_least_one_done
        super(SaleOrderLine, self)._compute_invoice_status()
        for line in self:
            # We handle the following specific situation: a physical product is partially delivered,
            # but we would like to set its invoice status to 'Fully Invoiced'. The use case is for
            # products sold by weight, where the delivered quantity rarely matches exactly the
            # quantity ordered.
            if line.order_id.state == 'done'\
                    and line.invoice_status == 'no'\
                    and line.product_id.type in ['consu', 'product']\
                    and line.product_id.invoice_policy == 'delivery'\
                    and line.move_ids \
                    and check_moves_state(line.move_ids):
                line.invoice_status = 'invoiced'

    @api.depends('move_ids')
    def _compute_product_updatable(self):
        for line in self:
            if not line.move_ids.filtered(lambda m: m.state != 'cancel'):
                super(SaleOrderLine, line)._compute_product_updatable()
            else:
                line.product_updatable = False

    @api.onchange('product_id')
    def _onchange_product_id_set_customer_lead(self):
        self.customer_lead = self.product_id.sale_delay

    def _prepare_procurement_values(self, group_id=False):
        """ Prepare specific key for moves or other components that will be created from a stock rule
        comming from a sale order line. This method could be override in order to add other custom key that could
        be used in move/po creation.
        """
        values = super(SaleOrderLine, self)._prepare_procurement_values(group_id)
        self.ensure_one()
        # Use the delivery date if there is else use date_order and lead time
        date_deadline = self.order_id.commitment_date or (self.order_id.date_order + timedelta(days=self.customer_lead or 0.0))
        date_planned = date_deadline - timedelta(days=self.order_id.company_id.security_lead)
        values.update({
            'group_id': group_id,
            'sale_line_id': self.id,
            'date_planned': date_planned,
            'date_deadline': date_deadline,
            'route_ids': self.route_id,
            'warehouse_id': self.order_id.warehouse_id or False,
            'partner_id': self.order_id.partner_shipping_id.id,
            'product_description_variants': self.with_context(lang=self.order_id.partner_id.lang)._get_sale_order_line_multiline_description_variants(),
            'company_id': self.order_id.company_id,
            'product_packaging_id': self.product_packaging_id,
            'sequence': self.sequence,
        })
        return values

    def _get_qty_procurement(self, previous_product_uom_qty=False):
        self.ensure_one()
        qty = 0.0
        outgoing_moves, incoming_moves = self._get_outgoing_incoming_moves()
        for move in outgoing_moves:
            qty += move.product_uom._compute_quantity(move.product_uom_qty, self.product_uom, rounding_method='HALF-UP')
        for move in incoming_moves:
            qty -= move.product_uom._compute_quantity(move.product_uom_qty, self.product_uom, rounding_method='HALF-UP')
        return qty

    def _get_outgoing_incoming_moves(self):
        outgoing_moves = self.env['stock.move']
        incoming_moves = self.env['stock.move']

        moves = self.move_ids.filtered(lambda r: r.state != 'cancel' and not r.scrapped and self.product_id == r.product_id)
        if self._context.get('accrual_entry_date'):
            moves = moves.filtered(lambda r: fields.Date.context_today(r, r.date) <= self._context['accrual_entry_date'])

        for move in moves:
            if move.location_dest_id.usage == "customer":
                if not move.origin_returned_move_id or (move.origin_returned_move_id and move.to_refund):
                    outgoing_moves |= move
            elif move.location_dest_id.usage != "customer" and move.to_refund:
                incoming_moves |= move

        return outgoing_moves, incoming_moves

    def _get_procurement_group(self):
        return self.order_id.procurement_group_id

    def _prepare_procurement_group_vals(self):
        return {
            'name': self.order_id.name,
            'move_type': self.order_id.picking_policy,
            'sale_id': self.order_id.id,
            'partner_id': self.order_id.partner_shipping_id.id,
        }

    def _action_launch_stock_rule(self, previous_product_uom_qty=False):
        """
        Launch procurement group run method with required/custom fields genrated by a
        sale order line. procurement group will launch '_run_pull', '_run_buy' or '_run_manufacture'
        depending on the sale order line product rule.
        """
        if self._context.get("skip_procurement"):
            return True
        precision = self.env['decimal.precision'].precision_get('Product Unit of Measure')
        procurements = []
        for line in self:
            line = line.with_company(line.company_id)
            if line.state != 'sale' or not line.product_id.type in ('consu','product'):
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
            product_qty, procurement_uom = line_uom._adjust_uom_quantities(product_qty, quant_uom)
            procurements.append(self.env['procurement.group'].Procurement(
                line.product_id, product_qty, procurement_uom,
                line.order_id.partner_shipping_id.property_stock_customer,
                line.product_id.display_name, line.order_id.name, line.order_id.company_id, values))
        if procurements:
            procurement_group = self.env['procurement.group']
            if self.env.context.get('import_file'):
                procurement_group = procurement_group.with_context(import_file=False)
            procurement_group.run(procurements)

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
        line_products = self.filtered(lambda l: l.product_id.type in ['product', 'consu'])
        if line_products.mapped('qty_delivered') and float_compare(values['product_uom_qty'], max(line_products.mapped('qty_delivered')), precision_digits=precision) == -1:
            raise UserError(_('You cannot decrease the ordered quantity of a sale order line below its delivered quantity.\n'
                              'Create a return in your inventory first.'))
        super(SaleOrderLine, self)._update_line_quantity(values)

```

## File: models\stock.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict

from odoo import api, fields, models, _
from odoo.tools.sql import column_exists, create_column


class StockLocationRoute(models.Model):
    _inherit = "stock.location.route"
    sale_selectable = fields.Boolean("Selectable on Sales Order Line")


class StockMove(models.Model):
    _inherit = "stock.move"
    sale_line_id = fields.Many2one('sale.order.line', 'Sale Line', index=True)

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
        return self.sale_line_id.order_id or res

    def _assign_picking_post_process(self, new=False):
        super(StockMove, self)._assign_picking_post_process(new=new)
        if new:
            picking_id = self.mapped('picking_id')
            sale_order_ids = self.mapped('sale_line_id.order_id')
            for sale_order_id in sale_order_ids:
                picking_id.message_post_with_view(
                    'mail.message_origin_link',
                    values={'self': picking_id, 'origin': sale_order_id},
                    subtype_id=self.env.ref('mail.mt_note').id)


class ProcurementGroup(models.Model):
    _inherit = 'procurement.group'

    sale_id = fields.Many2one('sale.order', 'Sale Order')


class StockRule(models.Model):
    _inherit = 'stock.rule'

    def _get_custom_move_fields(self):
        fields = super(StockRule, self)._get_custom_move_fields()
        fields += ['sale_line_id', 'partner_id', 'sequence']
        return fields


class StockPicking(models.Model):
    _inherit = 'stock.picking'

    sale_id = fields.Many2one(related="group_id.sale_id", string="Sales Order", store=True, readonly=False)

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
        for move in self.move_lines:
            sale_order = move.picking_id.sale_id
            # Creates new SO line only when pickings linked to a sale order and
            # for moves with qty. done and not already linked to a SO line.
            if not sale_order or move.location_dest_id.usage != 'customer' or move.sale_line_id or not move.quantity_done:
                continue
            product = move.product_id
            so_line_vals = {
                'move_ids': [(4, move.id, 0)],
                'name': product.display_name,
                'order_id': sale_order.id,
                'product_id': product.id,
                'product_uom_qty': 0,
                'qty_delivered': move.quantity_done,
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
        note summarize the real proccessed quantity and promote a
        manual action.

        :param dict moves: a dict with a move as key and tuple with
        new and old quantity as value. eg: {move_1 : (4, 5)}
        """

        def _keys_in_sorted(sale_line):
            """ sort by order_id and the sale_person on the order """
            return (sale_line.order_id.id, sale_line.order_id.user_id.id)

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
            return self.env.ref('sale_stock.exception_on_picking')._render(values=values)

        documents = self._log_activity_get_documents(moves, 'sale_line_id', 'DOWN', _keys_in_sorted, _keys_in_groupby)
        self._log_activity(_render_note_exception_quantity, documents)

        return super(StockPicking, self)._log_less_quantities_than_expected(moves)

class ProductionLot(models.Model):
    _inherit = 'stock.production.lot'

    sale_order_ids = fields.Many2many('sale.order', string="Sales Orders", compute='_compute_sale_order_ids')
    sale_order_count = fields.Integer('Sale order count', compute='_compute_sale_order_ids')

    @api.depends('name')
    def _compute_sale_order_ids(self):
        sale_orders = defaultdict(lambda: self.env['sale.order'])
        for move_line in self.env['stock.move.line'].search([('lot_id', 'in', self.ids), ('state', '=', 'done')]):
            move = move_line.move_id
            if move.picking_id.location_dest_id.usage == 'customer' and move.sale_line_id.order_id:
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

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_move
from . import res_company
from . import sale_order
from . import res_config_settings
from . import stock
from . import product_template
from . import res_users

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
        domain = self._product_sale_domain(product_template_ids, product_variant_ids)
        so_lines = self.env['sale.order.line'].search(domain)
        out_sum = 0
        if so_lines:
            product_uom = so_lines[0].product_id.uom_id
            quantities = so_lines.mapped(lambda line: line.product_uom._compute_quantity(line.product_uom_qty, product_uom))
            out_sum = sum(quantities)
        res['draft_sale_qty'] = out_sum
        res['draft_sale_orders'] = so_lines.mapped("order_id").sorted(key=lambda so: so.name)
        res['draft_sale_orders_matched'] = self.env.context.get('sale_line_to_match_id') in so_lines.ids
        res['qty']['out'] += out_sum
        return res

    def _product_sale_domain(self, product_template_ids, product_variant_ids):
        domain = [('state', 'in', ['draft', 'sent'])]
        if product_template_ids:
            domain += [('product_template_id', 'in', product_template_ids)]
        elif product_variant_ids:
            domain += [('product_id', 'in', product_variant_ids)]
        warehouse_id = self.env.context.get('warehouse', False)
        if warehouse_id:
            domain += [('warehouse_id', '=', warehouse_id)]
        return domain

```

## File: report\report_stock_forecasted.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="sale_report_product_product_replenishment" inherit_id="stock.report_product_product_replenishment">
        <xpath expr="//tr[@name='draft_picking_out']" position="after">
            <tr t-if="docs['draft_sale_qty']" name="draft_so_out" t-attf-class="#{docs['draft_sale_orders_matched'] and 'o_grid_match'}">
                <td colspan="2">Quotations</td>
                <td t-esc="-docs['draft_sale_qty']" t-options="{'widget': 'float', 'precision': precision}" class="text-right"/>
                <td>
                    <t t-foreach="docs['draft_sale_orders']" t-as="sale_order">
                        <t t-if="sale_order_index > 0">|</t>
                        <a t-attf-href="#" t-esc="sale_order.name"
                            class="font-weight-bold" view-type="form"
                            t-att-res-model="sale_order._name"
                            t-att-res-id="sale_order.id"/>
                    </t>
                </td>
            </tr>
        </xpath>
        <xpath expr="//td[@name='usedby_cell']" position="inside">
            <span t-if="line['move_out'] and line['move_out'].picking_id and line['move_out'].picking_id.sale_id">
                | <span t-esc="line['move_out'].picking_id.sale_id.amount_untaxed" t-options="{'widget': 'monetary', 'display_currency': line['move_out'].picking_id.sale_id.currency_id}" class="font-weight-bold"/>
                | <span t-esc="line['move_out'].picking_id.sale_id.partner_id.name"/>
            </span>
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
    def _get_routes(self, data):
        res = super(ReportStockRule, self)._get_routes(data)
        if data.get('so_route_ids'):
            res = self.env['stock.location.route'].browse(data['so_route_ids']) | res
        return res

```

## File: report\sale_order_report_templates.xml

```xml
<?xml version="1.0"?>
<odoo>
    <template id="report_saleorder_document_inherit_sale_stock" inherit_id="sale.report_saleorder_document">
        <xpath expr="//div[@name='expiration_date']" position="after">
            <div class="col-auto col-3 mw-100 mb-2" t-if="doc.incoterm" groups="sale_stock.group_display_incoterm">
                <strong>Incoterm:</strong>
                <p class="m-0" t-field="doc.incoterm.code"/>
            </div>
        </xpath>
    </template>

    <template id="report_invoice_document_inherit_sale_stock" inherit_id="account.report_invoice_document">
        <xpath expr="//div[@name='reference']" position="after">
            <div class="col-auto col-3 mw-100 mb-2" t-if="o.invoice_incoterm_id" groups="sale_stock.group_display_incoterm" name="invoice_incoterm_id">
                <strong>Incoterm:</strong>
                <p class="m-0" t-field="o.invoice_incoterm_id.code"/>
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

    def _group_by_sale(self, groupby=''):
        res = super()._group_by_sale(groupby)
        res += """,s.warehouse_id"""
        return res

    def _select_additional_fields(self, fields):
        fields['warehouse_id'] = ", s.warehouse_id as warehouse_id"
        return super()._select_additional_fields(fields)

```

## File: report\stock_report_deliveryslip.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="report_delivery_document_inherit_sale_stock" inherit_id="stock.report_delivery_document">
        <xpath expr="//div[@name='div_sched_date']" position="after">
            <div class="col-auto justify-content-end" t-if="o.sudo().sale_id.client_order_ref">
                <strong>Customer Reference:</strong>
                <p t-field="o.sudo().sale_id.client_order_ref"/>
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
from . import report_stock_forecasted
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
        <record id="group_display_incoterm" model="res.groups">
            <field name="name">Display incoterms on Sales Order and related invoices</field>
            <field name="category_id" ref="base.module_category_hidden"/>
        </record>

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

## File: static\src\js\qty_at_date_widget.js

```javascript
odoo.define('sale_stock.QtyAtDateWidget', function (require) {
"use strict";

var core = require('web.core');
var QWeb = core.qweb;

var Widget = require('web.Widget');
var widget_registry = require('web.widget_registry');
var utils = require('web.utils');

var _t = core._t;
var time = require('web.time');

var QtyAtDateWidget = Widget.extend({
    template: 'sale_stock.qtyAtDate',
    events: _.extend({}, Widget.prototype.events, {
        'click .fa-area-chart': '_onClickButton',
    }),

    /**
     * @override
     * @param {Widget|null} parent
     * @param {Object} params
     */
    init: function (parent, params) {
        this.data = params.data;
        this.fields = params.fields;
        this._updateData();
        this._super(parent);
    },

    start: function () {
        var self = this;
        return this._super.apply(this, arguments).then(function () {
            self._setPopOver();
        });
    },

    _updateData: function() {
        // add some data to simplify the template
        if (this.data.scheduled_date) {
            var qty_to_deliver = utils.round_decimals(this.data.qty_to_deliver, this.fields.qty_to_deliver.digits[1]);
            if (this.data.state === 'sale') {
                this.data.will_be_fulfilled = utils.round_decimals(this.data.free_qty_today, this.fields.free_qty_today.digits[1]) >= qty_to_deliver
            } else {
                this.data.will_be_fulfilled = utils.round_decimals(this.data.virtual_available_at_date, this.fields.virtual_available_at_date.digits[1]) >= qty_to_deliver
            }
            this.data.will_be_late = this.data.forecast_expected_date && this.data.forecast_expected_date > this.data.scheduled_date;
            if (['draft', 'sent'].includes(this.data.state)){
                // Moves aren't created yet, then the forecasted is only based on virtual_available of quant
                this.data.forecasted_issue = !this.data.will_be_fulfilled && !this.data.is_mto;
            } else {
                // Moves are created, using the forecasted data of related moves
                this.data.forecasted_issue = !this.data.will_be_fulfilled || this.data.will_be_late;
            }
        }
    },
    
    updateState: function (state) {
        this.$el.popover('dispose');
        var candidate = state.data[this.getParent().currentRow];
        if (candidate) {
            this.data = candidate.data;
            this._updateData();
            this.renderElement();
            this._setPopOver();
        }
    },
    /**
     * Redirect to the product graph view.
     *
     * @private
     * @param {MouseEvent} event
     * @returns {Promise} action loaded
     */
    async _openForecast(ev) {
        ev.stopPropagation();
        // TODO: in case of kit product, the forecast view should show the kit's components (get_component)
        // The forecast_report doesn't not allow for now multiple products 
        var action = await this._rpc({
            model: 'product.product',
            method: 'action_product_forecast_report',
            args: [[this.data.product_id.data.id]]
        });
        action.context = {
            active_model: 'product.product',
            active_id: this.data.product_id.data.id,
            warehouse: this.data.warehouse_id && this.data.warehouse_id.res_id,
            move_to_match_ids: this.data.move_ids.res_ids,
            sale_line_to_match_id: this.data.id,
        };
        return this.do_action(action);
    },

    _getContent() {
        if (!this.data.scheduled_date) {
            return;
        }
        this.data.delivery_date = this.data.scheduled_date.clone().add(this.getSession().getTZOffset(this.data.scheduled_date), 'minutes').format(time.getLangDateFormat());
        if (this.data.forecast_expected_date) {
            this.data.forecast_expected_date_str = this.data.forecast_expected_date.clone().add(this.getSession().getTZOffset(this.data.forecast_expected_date), 'minutes').format(time.getLangDateFormat());
        }
        const $content = $(QWeb.render('sale_stock.QtyDetailPopOver', {
            data: this.data,
        }));
        $content.on('click', '.action_open_forecast', this._openForecast.bind(this));
        return $content;
    },
    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------
    /**
     * Set a bootstrap popover on the current QtyAtDate widget that display available
     * quantity.
     */
    _setPopOver() {
        const $content = this._getContent();
        if (!$content) {
            return;
        }
        const options = {
            content: $content,
            html: true,
            placement: 'left',
            title: _t('Availability'),
            trigger: 'focus',
            delay: {'show': 0, 'hide': 100 },
        };
        this.$el.popover(options);
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------
    _onClickButton: function () {
        // We add the property special click on the widget link.
        // This hack allows us to trigger the popover (see _setPopOver) without
        // triggering the _onRowClicked that opens the order line form view.
        this.$el.find('.fa-area-chart').prop('special_click', true);
    },
});

widget_registry.add('qty_at_date_widget', QtyAtDateWidget);

return QtyAtDateWidget;
});

```

## File: static\src\xml\sale_stock.xml

```xml
<templates>
    <div t-name="sale_stock.qtyAtDate">
        <div t-att-class="!widget.data.display_qty_widget ? 'invisible' : ''">
            <a tabindex="0" t-attf-class="fa fa-area-chart {{ widget.data.forecasted_issue ? 'text-danger' : 'text-primary' }}"/>
        </div>
    </div>

    <div t-name="sale_stock.QtyDetailPopOver">
        <table class="table table-borderless table-sm">
            <tbody>
                <t t-if="!data.is_mto and ['draft', 'sent'].includes(data.state)">
                    <tr>
                        <td><strong>Forecasted Stock</strong><br /><small>On <span t-esc="data.delivery_date"/></small></td>
                        <td><b t-esc='data.virtual_available_at_date'/>
                        <t t-esc='data.product_uom.data.display_name'/></td>
                    </tr>
                    <tr>
                        <td><strong>Available</strong><br /><small>All planned operations included</small></td>
                        <td><b t-esc='data.free_qty_today' t-att-class="!data.will_be_fulfilled ? 'text-danger': ''"/>
                        <t t-esc='data.product_uom.data.display_name'/></td>
                    </tr>
                </t>
                <t t-elif="data.is_mto and ['draft', 'sent'].includes(data.state)">
                    <tr>
                        <td><strong>Expected Delivery</strong></td>
                        <td class="oe-right"><span t-esc="data.delivery_date"/></td>
                    </tr>
                    <tr>
                        <p>This product is replenished on demand.</p>
                    </tr>
                </t>
                <t t-elif="data.state == 'sale'">
                    <tr>
                        <td>
                            <strong>Reserved</strong><br/>
                        </td>
                        <td style="min-width: 50px; text-align: right;">
                            <b t-esc='data.qty_available_today'/> <t t-esc='data.product_uom.data.display_name'/>
                        </td>
                    </tr>
                    <tr t-if="data.qty_available_today &lt; data.qty_to_deliver">
                        <td>
                            <span t-if="data.will_be_fulfilled and data.forecast_expected_date_str">
                                Remaining demand available at <b t-esc="data.forecast_expected_date_str" t-att-class="data.scheduled_date &lt; data.forecast_expected_date ? 'text-danger' : ''"/>
                            </span>
                            <span t-elif="!data.will_be_fulfilled and data.forecast_expected_date_str" class="text-danger">
                                No enough future availaibility
                            </span>
                            <span t-elif="!data.will_be_fulfilled" class="text-danger">
                                No future availaibility
                            </span>
                            <span t-else="">
                                Available in stock
                            </span>
                        </td>
                    </tr>
                </t>
            </tbody>
        </table>
        <button t-if="!data.is_mto" class="text-left btn btn-link action_open_forecast"
            type="button">
            <i class="fa fa-fw o_button_icon fa-arrow-right"></i>
            View Forecast
        </button>
    </div>

    <div t-name="sale_stock.DelayAlertWidget">
        <p>The delivery
            <t t-foreach="late_elements" t-as="late_element">
                <a t-esc="late_element.name" href="#" t-att-element-id="late_element.id" t-att-element-model="model"/>,
            </t> will be late.
        </p>
    </div>
</templates>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="res_config_settings_view_form_sale" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.sale.stock.sale</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="sale.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='ups']" position="after">
                <div class="col-12 col-lg-6 o_setting_box" id="display_incoterms_setting">
                    <div class="o_setting_left_pane">
                        <field name="group_display_incoterm"/>
                    </div>
                    <div class="o_setting_right_pane">
                        <label for="group_display_incoterm"/>
                        <div class="text-muted">
                            Display incoterms on orders &amp; invoices
                        </div>
                        <div class="content-group" attrs="{'invisible': [('group_display_incoterm','=',False)]}">
                            <div class="mt8">
                                <button name="%(account.action_incoterms_tree)d" icon="fa-arrow-right" type="action" string="Incoterms" class="btn-link"/>
                            </div>
                        </div>
                    </div>
                </div>
            </xpath>
        </field>
    </record>

    <record id="res_config_settings_view_form_stock" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.sale.stock.stock</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="stock.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <div id="warning_info" position="after">
                <div class="col-12 col-lg-6 o_setting_box">
                    <div class="o_setting_right_pane">
                        <label for="default_picking_policy"/>
                        <div class="text-muted">
                            When to start shipping
                        </div>
                        <div class="content-group">
                            <div class="mt16">
                                <field name="default_picking_policy" class="o_light_label" widget="selection"/>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            <xpath expr="//h2[@id='schedule_info']" position="attributes">
                <attribute name="invisible">0</attribute>
            </xpath>
            <div id="sale_security_lead" position="replace">
                <div class="col-12 col-lg-6 o_setting_box" title="Margin of error for dates promised to customers. Products will be scheduled for procurement and delivery that many days earlier than the actual promised date, to cope with unexpected delays in the supply chain.">
                    <div class="o_setting_left_pane">
                        <field name="use_security_lead"/>
                    </div>
                    <div class="o_setting_right_pane">
                        <label for="use_security_lead"/>
                        <a href="https://www.odoo.com/documentation/15.0/applications/inventory_and_mrp/inventory/management/planning/scheduled_dates.html" title="Documentation" class="mr-2 o_doc_link" target="_blank"></a>
                        <span class="fa fa-lg fa-building-o" title="Values set here are company-specific." role="img" aria-label="Values set here are company-specific." groups="base.group_multi_company"/>
                        <div class="text-muted">
                            Schedule deliveries earlier to avoid delays
                        </div>
                        <div class="content-group">
                            <div class="mt16" attrs="{'invisible': [('use_security_lead','=',False)]}">
                                <span>Move forward expected delivery dates by <field name="security_lead" class="oe_inline"/> days</span>
                            </div>
                        </div>
                    </div>
                </div>
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
                        attrs="{'invisible': [('delivery_count', '=', 0)]}" groups="stock.group_stock_user">
                        <field name="delivery_count" widget="statinfo" string="Delivery"/>
                    </button>
                </xpath>
                <xpath expr="//group[@name='sale_shipping']" position="attributes">
                    <attribute name="groups"></attribute><!-- Remove the res.group on the group and set it on the field directly-->
                    <attribute name="string">Delivery</attribute>
                </xpath>
                <xpath expr="//label[@for='commitment_date']" position="before">
                    <field name="warehouse_id" options="{'no_create': True}" groups="stock.group_stock_multi_warehouses" force_save="1"/>
                    <field name="incoterm" options="{'no_open': True, 'no_create': True}" groups="sale_stock.group_display_incoterm"/>
                    <field name="picking_policy" required="True"/>
                </xpath>
                <xpath expr="//span[@name='expected_date_span']" position="attributes">
                    <attribute name="attrs">
                        {'invisible': [('effective_date', '!=', False), ('commitment_date', '!=', False)]}
                    </attribute>
                </xpath>
                <xpath expr="//div[@name='commitment_date_div']" position="replace">
                    <div class="o_row">
                        <field name="commitment_date"/>
                        <span class="text-muted" attrs="{'invisible': [('effective_date', '!=', False), ('commitment_date', '!=', False)]}">Expected: <field name="expected_date" widget="date"/></span>
                    </div>
                    <field name="effective_date" attrs="{'invisible': [('effective_date', '=', False)]}"/>
                </xpath>
                <xpath expr="//page[@name='other_information']//field[@name='expected_date']" position="after">
                    <field name="show_json_popover" invisible="1"/>
                    <field string=" " name="json_popover" widget="stock_rescheduling_popover" attrs="{'invisible': [('show_json_popover', '=', False)]}"/>
                </xpath>
                <xpath expr="//field[@name='order_line']/form/group/group/field[@name='analytic_tag_ids']" position="before">
                    <field name="route_id" groups="stock.group_adv_location" options="{'no_create': True}"/>
                </xpath>
                <xpath expr="//field[@name='order_line']/tree/field[@name='analytic_tag_ids']" position="after">
                    <field name="route_id" groups="stock.group_adv_location" options="{'no_create': True}" optional="hide"/>
                </xpath>
           </field>
        </record>

        <record id="view_quotation_tree" model="ir.ui.view">
            <field name="name">sale.order.tree.inherit.sale.stock</field>
            <field name="model">sale.order</field>
            <field name="inherit_id" ref="sale.view_quotation_tree"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='tag_ids']" position="after">
                    <field name="warehouse_id" options="{'no_create': True}" groups="stock.group_stock_multi_warehouses" optional="hide" />
                </xpath>
            </field>
        </record>

        <record id="view_order_tree" model="ir.ui.view">
            <field name="name">sale.order.tree.inherit.sale.stock</field>
            <field name="model">sale.order</field>
            <field name="inherit_id" ref="sale.view_order_tree"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='team_id']" position="after">
                    <field name="warehouse_id" options="{'no_create': True}" groups="stock.group_stock_multi_warehouses" optional="hide"/>
                </xpath>
            </field>
        </record>

        <record id="view_order_line_tree_inherit_sale_stock" model="ir.ui.view">
            <field name="name">sale.order.line.tree.sale.stock.location</field>
            <field name="inherit_id" ref="sale.view_order_line_tree"/>
            <field name="model">sale.order.line</field>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='price_subtotal']" position="before">
                    <field name="route_id" groups="stock.group_adv_location" options="{'no_create': True}"/>
                </xpath>
            </field>
        </record>

        <record id="view_order_form_inherit_sale_stock_qty" model="ir.ui.view">
            <field name="name">sale.order.line.tree.sale.stock.qty</field>
            <field name="inherit_id" ref="sale.view_order_form"/>
            <field name="model">sale.order</field>
            <field name="arch" type="xml">
                <xpath expr="//page/field[@name='order_line']/form/group/group/div[@name='ordered_qty']/field[@name='product_uom']" position="after">
                    <!-- below fields are used in the widget qty_at_date_widget -->
                    <field name="product_type" invisible="1"/>
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
                    <widget name="qty_at_date_widget" width="0.1"/>
                </xpath>
                <xpath expr="//page/field[@name='order_line']/tree/field[@name='qty_delivered']" position="after">
                    <!-- below fields are used in the widget qty_at_date_widget -->
                    <field name="product_type" invisible="1"/>
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
    <template id="sale_order_portal_content_inherit_sale_stock" name="Orders Shipping Followup" inherit_id="sale.sale_order_portal_content">
        <xpath expr="//div[@id='so_date']" position="after">
            <div class="row" t-if="sale_order.incoterm">
                <div class="mb-3 col-6 ml-auto">
                    <strong>Incoterm: </strong> <span t-field="sale_order.incoterm"/>
                </div>
            </div>
        </xpath>
        <xpath expr="//div[@id='informations']" position="inside">
            <t t-set="delivery_orders" t-value="sale_order.picking_ids.filtered(lambda picking: picking.picking_type_id.code == 'outgoing')"/>
            <t t-if="delivery_orders">
                <div>
                    <strong>Delivery Orders</strong>
                </div>
                <div>
                    <t t-foreach="delivery_orders" t-as="i">
                        <t t-set="delivery_report_url" t-value="'/my/picking/pdf/%s?%s' % (i.id, keep_query())"/>
                        <div class="d-flex flex-wrap align-items-center justify-content-between o_sale_stock_picking">
                            <div>
                                <a t-att-href="delivery_report_url">
                                    <span t-esc="i.name"/>
                                </a>
                                <div class="small d-lg-inline-block ml-3">Date:
                                    <span class="text-muted" t-field="i.date_done" t-options="{'date_only': True}"/>
                                    <span t-if="i.state in ['draft', 'waiting', 'confirmed', 'assigned']" class="text-muted" t-field="i.scheduled_date" t-options="{'date_only': True}"/>
                                </div>
                            </div>
                            <span t-if="i.state == 'done'" class="small badge badge-success orders_label_text_align"><i class="fa fa-fw fa-truck"/> <b>Shipped</b></span>
                            <span t-if="i.state == 'cancel'" class="small badge badge-danger orders_label_text_align"><i class="fa fa-fw fa-times"/> <b>Cancelled</b></span>
                            <span t-if="i.state in ['draft', 'waiting', 'confirmed', 'assigned']" class="small badge badge-info orders_label_text_align"><i class="fa fa-fw fa-clock-o"/> <b>Preparation</b></span>
                        </div>
                    </t>
                </div>
            </t>
        </xpath>
  </template>

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
                <button class="oe_stat_button" name="action_view_so"
                        type="object" icon="fa-pencil-square-o" help="Sale Orders"
                        attrs="{'invisible': ['|', ('sale_order_count', '=', 0), ('display_complete', '=', False)]}">
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
                    <field name="sale_order_ids" widget="many2many" readonly="True"
                           attrs="{'invisible': ['|', ('sale_order_ids', '=', []), ('display_complete', '=', True)]}">
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

## File: views\stock_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <menuitem id="menu_aftersale" name="After-Sale"
            groups="sales_team.group_sale_salesman"
            parent="sale.sale_menu_root" sequence="5" />
        <menuitem id="menu_invoiced" name="Invoicing" parent="menu_aftersale" sequence="1"/>

        <record id="stock_location_route_view_form_inherit_sale_stock" model="ir.ui.view">
            <field name="name">stock.location.route.form</field>
            <field name="inherit_id" ref="stock.stock_location_route_form_view"/>
            <field name="model">stock.location.route</field>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='warehouse_ids']" position="after">
                    <br/><field name="sale_selectable" string="Sales Order Lines"/>
                </xpath>
            </field>
        </record>

        <record id="product_template_view_form_inherit_sale" model="ir.ui.view">
            <field name="name">product.template.inherit.form</field>
            <field name="inherit_id" ref="sale.product_template_form_view_sale_order_button"/>
            <field name="model">product.template</field>
            <field name="arch" type="xml">
                <xpath expr="//button[@name='action_view_sales']" position="replace" />
            </field>
        </record>

        <record id="product_template_view_form_inherit_stock" model="ir.ui.view">
            <field name="name">product.template.inherit.form</field>
            <field name="inherit_id" ref="stock.product_template_form_view_procurement_button"/>
            <field name="model">product.template</field>
            <field name="arch" type="xml">
                <button name="action_view_orderpoints" position="after">
                    <button class="oe_stat_button" name="action_view_sales"
                        type="object" icon="fa-signal" attrs="{'invisible': [('sale_ok', '=', False)]}"
                        groups="sales_team.group_sale_salesman" help="Sold in the last 365 days">
                        <div class="o_field_widget o_stat_info">
                            <span class="o_stat_value">
                                <field name="sales_count" widget="statinfo" nolabel="1" class="mr4"/>
                                <field name="uom_name"/>
                            </span>
                            <span class="o_stat_text">Sold</span>
                        </div>
                    </button>
                </button>
            </field>
        </record>
        <record id="stock.view_stock_product_tree" model="ir.ui.view">
            <field name="groups_id" eval="[(4, ref('sales_team.group_sale_salesman'))]" />
        </record>
        <record id="stock.view_stock_product_template_tree" model="ir.ui.view">
            <field name="groups_id" eval="[(4, ref('sales_team.group_sale_salesman'))]" />
        </record>
        <record id="stock.product_template_kanban_stock_view" model="ir.ui.view">
            <field name="groups_id" eval="[(4, ref('sales_team.group_sale_salesman'))]" />
        </record>
    </data>
</odoo>

```

## File: wizard\sale_make_invoice_advance.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class SaleAdvancePaymentInv(models.TransientModel):
    _inherit = "sale.advance.payment.inv"

    def _prepare_invoice_values(self, order, name, amount, so_line):
        invoice_vals = super()._prepare_invoice_values(order, name, amount, so_line)
        invoice_vals['invoice_incoterm_id'] = order.incoterm.id
        return invoice_vals

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
            wizard.display_delivery_alert = bool(any(picking.state == 'done' for picking in wizard.order_id.picking_ids))

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
            <field name="display_invoice_alert" position="after">
                <field name="display_delivery_alert" invisible="1"/>
                <div attrs="{'invisible': [('display_delivery_alert', '=', False)]}">
                    Some products have already been delivered. Returns can be created from the Delivery Orders.
                </div>
            </field>
        </field>
    </record>
</odoo>

```

## File: wizard\stock_rules_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class StockRulesReport(models.TransientModel):
    _inherit = 'stock.rules.report'

    so_route_ids = fields.Many2many('stock.location.route', string='Apply specific routes',
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
from . import sale_make_invoice_advance

```

