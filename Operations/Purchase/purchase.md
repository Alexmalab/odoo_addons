# Odoo Module: purchase

Category: Operations/Purchase

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models
from . import report

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Purchase',
    'version': '1.2',
    'category': 'Operations/Purchase',
    'sequence': 60,
    'summary': 'Purchase orders, tenders and agreements',
    'description': "",
    'website': 'https://www.odoo.com/page/purchase',
    'depends': ['account'],
    'data': [
        'security/purchase_security.xml',
        'security/ir.model.access.csv',
        'views/account_move_views.xml',
        'data/purchase_data.xml',
        'report/purchase_reports.xml',
        'views/purchase_views.xml',
        'views/res_config_settings_views.xml',
        'views/product_views.xml',
        'views/res_partner_views.xml',
        'views/purchase_template.xml',
        'report/purchase_bill_views.xml',
        'report/purchase_report_views.xml',
        'data/mail_template_data.xml',
        'views/portal_templates.xml',
        'report/purchase_order_templates.xml',
        'report/purchase_quotation_templates.xml',
    ],
    'demo': [
        'data/purchase_demo.xml',
    ],
    'installable': True,
    'auto_install': False,
    'application': True,
    'license': 'LGPL-3',
}

```

## File: controllers\portal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
from collections import OrderedDict

from odoo import http
from odoo.exceptions import AccessError, MissingError
from odoo.http import request
from odoo.tools import image_process
from odoo.tools.translate import _
from odoo.addons.portal.controllers.portal import pager as portal_pager, CustomerPortal
from odoo.addons.web.controllers.main import Binary



class CustomerPortal(CustomerPortal):

    def _prepare_home_portal_values(self):
        values = super(CustomerPortal, self)._prepare_home_portal_values()
        values['purchase_count'] = request.env['purchase.order'].search_count([
            ('state', 'in', ['purchase', 'done', 'cancel'])
        ]) if request.env['purchase.order'].check_access_rights('read', raise_exception=False) else 0
        return values

    def _purchase_order_get_page_view_values(self, order, access_token, **kwargs):
        #
        def resize_to_48(b64source):
            if not b64source:
                b64source = base64.b64encode(Binary().placeholder())
            return image_process(b64source, size=(48, 48))

        values = {
            'order': order,
            'resize_to_48': resize_to_48,
        }
        return self._get_page_view_values(order, access_token, values, 'my_purchases_history', True, **kwargs)

    @http.route(['/my/purchase', '/my/purchase/page/<int:page>'], type='http', auth="user", website=True)
    def portal_my_purchase_orders(self, page=1, date_begin=None, date_end=None, sortby=None, filterby=None, **kw):
        values = self._prepare_portal_layout_values()
        partner = request.env.user.partner_id
        PurchaseOrder = request.env['purchase.order']

        domain = []

        archive_groups = self._get_archive_groups('purchase.order', domain) if values.get('my_details') else []
        if date_begin and date_end:
            domain += [('create_date', '>', date_begin), ('create_date', '<=', date_end)]

        searchbar_sortings = {
            'date': {'label': _('Newest'), 'order': 'create_date desc, id desc'},
            'name': {'label': _('Name'), 'order': 'name asc, id asc'},
            'amount_total': {'label': _('Total'), 'order': 'amount_total desc, id desc'},
        }
        # default sort by value
        if not sortby:
            sortby = 'date'
        order = searchbar_sortings[sortby]['order']

        searchbar_filters = {
            'all': {'label': _('All'), 'domain': [('state', 'in', ['purchase', 'done', 'cancel'])]},
            'purchase': {'label': _('Purchase Order'), 'domain': [('state', '=', 'purchase')]},
            'cancel': {'label': _('Cancelled'), 'domain': [('state', '=', 'cancel')]},
            'done': {'label': _('Locked'), 'domain': [('state', '=', 'done')]},
        }
        # default filter by value
        if not filterby:
            filterby = 'all'
        domain += searchbar_filters[filterby]['domain']

        # count for pager
        purchase_count = PurchaseOrder.search_count(domain)
        # make pager
        pager = portal_pager(
            url="/my/purchase",
            url_args={'date_begin': date_begin, 'date_end': date_end},
            total=purchase_count,
            page=page,
            step=self._items_per_page
        )
        # search the purchase orders to display, according to the pager data
        orders = PurchaseOrder.search(
            domain,
            order=order,
            limit=self._items_per_page,
            offset=pager['offset']
        )
        request.session['my_purchases_history'] = orders.ids[:100]

        values.update({
            'date': date_begin,
            'orders': orders,
            'page_name': 'purchase',
            'pager': pager,
            'archive_groups': archive_groups,
            'searchbar_sortings': searchbar_sortings,
            'sortby': sortby,
            'searchbar_filters': OrderedDict(sorted(searchbar_filters.items())),
            'filterby': filterby,
            'default_url': '/my/purchase',
        })
        return request.render("purchase.portal_my_purchase_orders", values)

    @http.route(['/my/purchase/<int:order_id>'], type='http', auth="public", website=True)
    def portal_my_purchase_order(self, order_id=None, access_token=None, **kw):
        try:
            order_sudo = self._document_check_access('purchase.order', order_id, access_token=access_token)
        except (AccessError, MissingError):
            return request.redirect('/my')

        values = self._purchase_order_get_page_view_values(order_sudo, access_token, **kw)
        if order_sudo.company_id:
            values['res_company'] = order_sudo.company_id
        return request.render("purchase.portal_my_purchase_order", values)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import portal

```

## File: data\mail_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="email_template_edi_purchase" model="mail.template">
            <field name="name">Purchase Order: Send RFQ</field>
            <field name="model_id" ref="purchase.model_purchase_order"/>
            <field name="subject">${object.company_id.name} Order (Ref ${object.name or 'n/a' })</field>
            <field name="partner_to">${object.partner_id.id}</field>
            <field name="body_html" type="html">
<div style="margin: 0px; padding: 0px;">
    <p style="margin: 0px; padding: 0px; font-size: 13px;">
        Dear ${object.partner_id.name}
        % if object.partner_id.parent_id:
            (${object.partner_id.parent_id.name})
        % endif
        <br/><br/>
        Here is in attachment a request for quotation <strong>${object.name}</strong>
        % if object.partner_ref:
            with reference: ${object.partner_ref}
        % endif
        from ${object.company_id.name}.
        <br/><br/>
        If you have any questions, please do not hesitate to contact us.
        <br/><br/>
        Best regards,
    </p>
</div></field>
            <field name="report_template" ref="report_purchase_quotation"/>
            <field name="report_name">RFQ_${(object.name or '').replace('/','_')}</field>
            <field name="lang">${object.partner_id.lang}</field>
            <field name="user_signature" eval="False"/>
            <field name="auto_delete" eval="True"/>
        </record>

        <record id="email_template_edi_purchase_done" model="mail.template">
            <field name="name">Purchase Order: Send PO</field>
            <field name="model_id" ref="purchase.model_purchase_order"/>
            <field name="subject">${object.company_id.name} Order (Ref ${object.name or 'n/a' })</field>
            <field name="partner_to">${object.partner_id.id}</field>
            <field name="body_html" type="html">
<div style="margin: 0px; padding: 0px;">
    <p style="margin: 0px; padding: 0px; font-size: 13px;">
        Dear ${object.partner_id.name}
        % if object.partner_id.parent_id:
            (${object.partner_id.parent_id.name})
        % endif
        <br/><br/>
        Here is in attachment a purchase order <strong>${object.name}</strong>
        % if object.partner_ref:
            with reference: ${object.partner_ref}
        % endif
        amounting in <strong>${format_amount(object.amount_total, object.currency_id)}</strong>
        from ${object.company_id.name}.
        <br/><br/>
        If you have any questions, please do not hesitate to contact us.
        <br/><br/>
        Best regards,
    </p>
</div></field>
            <field name="report_template" ref="action_report_purchase_order"/>
            <field name="report_name">PO_${(object.name or '').replace('/','_')}</field>
            <field name="lang">${object.partner_id.lang}</field>
            <field name="user_signature" eval="False"/>
            <field name="auto_delete" eval="True"/>
        </record>
    </data>
</odoo>

```

## File: data\purchase_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- Purchase-related subtypes for messaging / Chatter -->
        <record id="mt_rfq_confirmed" model="mail.message.subtype">
            <field name="name">RFQ Confirmed</field>
            <field name="default" eval="False"/>
            <field name="res_model">purchase.order</field>
        </record>
        <record id="mt_rfq_approved" model="mail.message.subtype">
            <field name="name">RFQ Approved</field>
            <field name="default" eval="False"/>
            <field name="res_model">purchase.order</field>
        </record>
        <record id="mt_rfq_done" model="mail.message.subtype">
            <field name="name">RFQ Done</field>
            <field name="default" eval="False"/>
            <field name="res_model">purchase.order</field>
        </record>

        <!-- Sequences for purchase.order -->
        <record id="seq_purchase_order" model="ir.sequence">
            <field name="name">Purchase Order</field>
            <field name="code">purchase.order</field>
            <field name="prefix">P</field>
            <field name="padding">5</field>
            <field name="company_id" eval="False"/>
        </record>

        <!-- Share Button in action menu -->
        <record id="model_purchase_order_action_share" model="ir.actions.server">
            <field name="name">Share</field>
            <field name="model_id" ref="purchase.model_purchase_order"/>
            <field name="binding_model_id" ref="purchase.model_purchase_order"/>
            <field name="binding_view_types">form</field>
            <field name="state">code</field>
            <field name="code">action = records.action_share()</field>
        </record>
    </data>
</odoo>

```

## File: data\purchase_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <record id="base.user_demo" model="res.users">
            <field eval="[(4, ref('group_purchase_user'))]" name="groups_id"/>
        </record>

    </data>

    <record id="purchase_order_1" model="purchase.order">
        <field name="partner_id" ref="base.res_partner_1"/>
        <field name="user_id" ref="base.user_admin"/>
        <field name="state">draft</field>
        <field name="order_line" model="purchase.order.line" eval="[(5, 0, 0),
            (0, 0, {
                'product_id': ref('product.product_delivery_01'),
                'name': obj().env.ref('product.product_delivery_01').partner_ref,
                'price_unit': 79.80,
                'product_qty': 15.0,
                'product_uom': ref('uom.product_uom_unit'),
                'date_planned': DateTime.today() + relativedelta(days=3)}),
            (0, 0, {
                'product_id': ref('product.product_product_25'),
                'name': obj().env.ref('product.product_product_25').partner_ref,
                'price_unit': 2868.70,
                'product_qty': 5.0,
                'product_uom': ref('uom.product_uom_unit'),
                'date_planned': DateTime.today() + relativedelta(days=3)}),
            (0, 0, {
                'product_id': ref('product.product_product_27'),
                'name': obj().env.ref('product.product_product_27').partner_ref,
                'price_unit': 3297.20,
                'product_qty': 4.0,
                'product_uom': ref('uom.product_uom_unit'),
                'date_planned': DateTime.today() + relativedelta(days=3)})
        ]"/>
    </record>

    <record id="purchase_order_2" model="purchase.order">
        <field name="partner_id" ref="base.res_partner_3"/>
        <field name="user_id" ref="base.user_admin"/>
        <field name="state">draft</field>
        <field name="order_line" model="purchase.order.line" eval="[(5, 0, 0),
            (0, 0, {
                'product_id': ref('product.product_delivery_02'),
                'name': obj().env.ref('product.product_delivery_02').partner_ref,
                'price_unit': 132.50,
                'product_qty': 20.0,
                'product_uom': ref('uom.product_uom_unit'),
                'date_planned': DateTime.today() + relativedelta(days=1)}),
            (0, 0, {
                'product_id': ref('product.product_delivery_01'),
                'name': obj().env.ref('product.product_delivery_01').partner_ref,
                'price_unit': 89.0,
                'product_qty': 5.0,
                'product_uom': ref('uom.product_uom_unit'),
                'date_planned': DateTime.today() + relativedelta(days=1)}),
        ]"/>
    </record>

    <record id="purchase_order_3" model="purchase.order">
        <field name="partner_id" ref="base.res_partner_12"/>
        <field name="user_id" ref="base.user_admin"/>
        <field name="state">draft</field>
        <field name="order_line" model="purchase.order.line" eval="[(5, 0, 0),
            (0, 0, {
                'product_id': ref('product.product_product_2'),
                'name': obj().env.ref('product.product_product_2').partner_ref,
                'price_unit': 25.50,
                'product_qty': 10.0,
                'product_uom': ref('uom.product_uom_hour'),
                'date_planned': DateTime.today() + relativedelta(days=1)}),
        ]"/>
    </record>

    <record id="purchase_order_4" model="purchase.order">
        <field name="partner_id" ref="base.res_partner_4"/>
        <field name="user_id" ref="base.user_admin"/>
        <field name="state">draft</field>
        <field name="order_line" model="purchase.order.line" eval="[(5, 0, 0),
            (0, 0, {
                'product_id': ref('product.product_delivery_02'),
                'name': obj().env.ref('product.product_delivery_02').partner_ref,
                'price_unit': 85.50,
                'product_qty': 6.0,
                'product_uom': ref('uom.product_uom_unit'),
                'date_planned': DateTime.today() + relativedelta(days=5)}),
            (0, 0, {
                'product_id': ref('product.product_product_20'),
                'name': obj().env.ref('product.product_product_20').partner_ref,
                'price_unit': 1690.0,
                'product_qty': 5.0,
                'product_uom': ref('uom.product_uom_unit'),
                'date_planned': DateTime.today() + relativedelta(days=5)}),
            (0, 0, {
                'product_id': ref('product.product_product_6'),
                'name': obj().env.ref('product.product_product_6').partner_ref,
                'price_unit': 800.0,
                'product_qty': 7.0,
                'product_uom': ref('uom.product_uom_unit'),
                'date_planned': DateTime.today() + relativedelta(days=5)})
        ]"/>
    </record>

    <record id="purchase_order_5" model="purchase.order">
        <field name="partner_id" ref="base.res_partner_2"/>
        <field name="user_id" ref="base.user_admin"/>
        <field name="state">draft</field>
        <field name="order_line" model="purchase.order.line" eval="[(5, 0, 0),
            (0, 0, {
                'product_id': ref('product.product_product_22'),
                'name': obj().env.ref('product.product_product_22').partner_ref,
                'price_unit': 2010.0,
                'product_qty': 3.0,
                'product_uom': ref('uom.product_uom_unit'),
                'date_planned': DateTime.today()}),
            (0, 0, {
                'product_id': ref('product.product_product_24'),
                'name': obj().env.ref('product.product_product_24').partner_ref,
                'price_unit': 876.0,
                'product_qty': 3.0,
                'product_uom': ref('uom.product_uom_unit'),
                'date_planned': DateTime.today()}),
        ]"/>
    </record>

    <record id="purchase_order_6" model="purchase.order">
        <field name="partner_id" ref="base.res_partner_1"/>
        <field name="user_id" ref="base.user_admin"/>
        <field name="state">draft</field>
        <field name="order_line" model="purchase.order.line" eval="[(5, 0, 0),
            (0, 0, {
                'product_id': ref('product.product_delivery_02'),
                'name': obj().env.ref('product.product_delivery_02').partner_ref,
                'price_unit': 58.0,
                'product_qty': 9.0,
                'product_uom': ref('uom.product_uom_unit'),
                'date_planned': DateTime.today()}),
            (0, 0, {
                'product_id': ref('product.product_delivery_01'),
                'name': obj().env.ref('product.product_delivery_01').partner_ref,
                'price_unit': 65.0,
                'product_qty': 3.0,
                'product_uom': ref('uom.product_uom_unit'),
                'date_planned': DateTime.today()}),
            (0, 0, {
                'product_id': ref('product.consu_delivery_01'),
                'name': obj().env.ref('product.consu_delivery_01').partner_ref,
                'price_unit': 154.5,
                'product_qty': 4.0,
                'product_uom': ref('uom.product_uom_unit'),
                'date_planned': DateTime.today()}),
        ]"/>
    </record>

    <record id="purchase_order_7" model="purchase.order">
        <field name="partner_id" ref="base.res_partner_4"/>
        <field name="user_id" ref="base.user_admin"/>
        <field name="state">draft</field>
        <field name="order_line" model="purchase.order.line" eval="[(5, 0, 0),
            (0, 0, {
                'product_id': ref('product.product_product_12'),
                'name': obj().env.ref('product.product_product_12').partner_ref,
                'price_unit': 13.5,
                'product_qty': 5.0,
                'product_uom': ref('uom.product_uom_unit'),
                'date_planned': DateTime.today()}),
            (0, 0, {
                'product_id': ref('product.product_delivery_02'),
                'name': obj().env.ref('product.product_delivery_02').partner_ref,
                'price_unit': 38.0,
                'product_qty': 15.0,
                'product_uom': ref('uom.product_uom_unit'),
                'date_planned': DateTime.today()}),
        ]"/>
    </record>

</odoo>


```

## File: doc\average.rst

```rst
Average price


Normal case:
------------

= When the product is purchased by purchase order and leaves towards a customer with a delivery order.  We assume also 
that accounting entries are generated in real-time.  

- When the products are received, the stock move gets the unit price and UoM of the purchase order in company currency.  The standard price of the product is updated with 
(qty available * standard price + incoming qty * purchase price) / (qty available + incoming qty)
- In the stock journal, the accounting items will be generated based on this (price on move is price of purchase order)
- When a delivery order is made, it is going out at cost price (= average price which was updated during incoming move)
- When generating outgoing accounting entries, this is the total amount based on this average cost price


Case of production: 
-------------------
In case of produced goods, the incoming stock move of the finished product and accounting entries can not just invent a cost price like the sum of the cost price of the parts in the BoM, as cost methods tend to be a lot more complicated than this. 
When the finished good is produced, we will put the cost price from the product.  

In the product form there is a link next to the cost price where the user can update it.  This will also generate accounting entries as the stock will be valued differently.  


Case of no purchase order
-------------------------
When no purchase order is given, the price on the stock move and generated entries is the cost price on the product


Returned Goods / Scrap / ...
----------------------------
For returning goods to supplier, the price on the original purchase order is put on the stock and account moves.  That way, this would have the same effect as cancelling the original in move.  
Scrap is an outgoing move at cost price.   


Negative stock
--------------
If your stock is negative and you receive, the price of the product becomes the price on the purchase order.  

Extra
-----
- standard price, costing method and valuation (real_time or manual) are properties.  This means it is possible to use the same product in different companies with different price, valuation and costing methods. 

- UoMs: On the stock move, the price is in units of the stock move, so this will get converted to the product UoM and reverse

- Currency: On the stock move, the currency is the company currency

```

## File: doc\fifolifo.rst

```rst
FIFO/LIFO

In order to activate FIFO/LIFO, the costing method in the product form should be fifo/lifo.  This is only possible when cost methods are checked under Settings > Purchase



Normal case:
------------

= When product is purchased by purchase order and leaves towards a customer with a delivery order.  We assume also 
that accounting entries are generated in real-time.  

- When product is received, the stock move gets the unit price and UoM of the purchase order
- In the stock journal, the accounting items will be generated based on this.  
- When a delivery order is made, the FIFO/LIFO algorithm is used to check which in moves correspond to this out move.  A weighted average is calculated 
based on the different in moves which would theoretically have gone out according to the FIFO/LIFO algorithm.  This average becomes also the new cost price on the product.  
Technically, these calculated matchings are saved in stock_move_matching which makes further FIFO/LIFO calculations easier.  
- When generating accounting entries, the stock.move.matching table is used, to generate 1 account move line per matching.  That way, one stock move will have one account 
move with multiple account move lines with the amounts from the matchings.   


Case of production: 
-------------------
In case of produced goods, the incoming stock move of the finished product and accounting entries can not just invent a cost price like the sum of the cost price of the parts in the BoM, as costing methods tend to be a lot more complicated than this. 
On the stock move, we will put the cost price of the product.  

Case of no purchase order
-------------------------
When no purchase order is given, the price on the stock move and generated entries is the standard price on the product.  


Returned Goods / Scrap / ...
----------------------------
Returned goods to supplier have the same calculations as a normal out, same with scrap.  


Negative stocks
---------------
When an out move makes the stock (quantity on hand) become negative, the cost price of the product is not updated and stock move matchings will only be created for the moves that can be matched.  (until stock is zero)  If the quantity on hand became negative and afterwards we get an incoming move, the system will try to match the previous outgoing move(s) as much as possible with the incoming move.  These matches will generate also the necessary accounting entries.  


Inter-company
-------------
cost price, costing method and valuation (real_time or manual_periodic) are properties (are different according to the company) => when you receive/ship goods, it depends on the company of the stock move as what costing method needs to be used. 

Not possible to create a stock move between two locations of different companies.  You need a transit location in between.  This new constraint removes the requirement for a currency or two prices on one stock move.  

UoMs: As quantities need to be matched between outgoing and ingoing stock moves which can have different UoMs, it will take all these conversions into account

Currency: On the stock move, the currency is the company currency

```

## File: migrations\9.0.1.2\pre-create-properties.py

```python
# -*- coding: utf-8 -*-

def convert_field(cr, model, field, target_model):
    table = model.replace('.', '_')

    cr.execute("""SELECT 1
                    FROM information_schema.columns
                   WHERE table_name = %s
                     AND column_name = %s
               """, (table, field))
    if not cr.fetchone():
        return

    cr.execute("SELECT id FROM ir_model_fields WHERE model=%s AND name=%s", (model, field))
    [fields_id] = cr.fetchone()

    cr.execute("""
        INSERT INTO ir_property(name, type, fields_id, company_id, res_id, value_reference)
        SELECT %(field)s, 'many2one', %(fields_id)s, company_id, CONCAT('{model},', id),
               CONCAT('{target_model},', {field})
          FROM {table} t
         WHERE {field} IS NOT NULL
           AND NOT EXISTS(SELECT 1
                            FROM ir_property
                           WHERE fields_id=%(fields_id)s
                             AND company_id=t.company_id
                             AND res_id=CONCAT('{model},', t.id))
    """.format(**locals()), locals())

    cr.execute('ALTER TABLE "{0}" DROP COLUMN "{1}" CASCADE'.format(table, field))

def migrate(cr, version):
    convert_field(cr, 'res.partner', 'property_purchase_currency_id', 'res.currency')
    convert_field(cr, 'product.template',
                  'property_account_creditor_price_difference', 'account.account')

```

## File: models\account_invoice.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _


class AccountMove(models.Model):
    _inherit = 'account.move'

    purchase_vendor_bill_id = fields.Many2one('purchase.bill.union', store=False, readonly=True,
        states={'draft': [('readonly', False)]},
        string='Auto-complete',
        help="Auto-complete from a past bill / purchase order.")
    purchase_id = fields.Many2one('purchase.order', store=False, readonly=True,
        states={'draft': [('readonly', False)]},
        string='Purchase Order',
        help="Auto-complete from a past purchase order.")

    def _get_invoice_reference(self):
        self.ensure_one()
        vendor_refs = [ref for ref in set(self.line_ids.mapped('purchase_line_id.order_id.partner_ref')) if ref]
        if self.ref:
            return [ref for ref in self.ref.split(', ') if ref and ref not in vendor_refs] + vendor_refs
        return vendor_refs

    @api.onchange('purchase_vendor_bill_id', 'purchase_id')
    def _onchange_purchase_auto_complete(self):
        ''' Load from either an old purchase order, either an old vendor bill.

        When setting a 'purchase.bill.union' in 'purchase_vendor_bill_id':
        * If it's a vendor bill, 'invoice_vendor_bill_id' is set and the loading is done by '_onchange_invoice_vendor_bill'.
        * If it's a purchase order, 'purchase_id' is set and this method will load lines.

        /!\ All this not-stored fields must be empty at the end of this function.
        '''
        if self.purchase_vendor_bill_id.vendor_bill_id:
            self.invoice_vendor_bill_id = self.purchase_vendor_bill_id.vendor_bill_id
            self._onchange_invoice_vendor_bill()
        elif self.purchase_vendor_bill_id.purchase_order_id:
            self.purchase_id = self.purchase_vendor_bill_id.purchase_order_id
        self.purchase_vendor_bill_id = False

        if not self.purchase_id:
            return

        # Copy partner.
        self.partner_id = self.purchase_id.partner_id
        self.fiscal_position_id = self.purchase_id.fiscal_position_id
        self.invoice_payment_term_id = self.purchase_id.payment_term_id
        self.currency_id = self.purchase_id.currency_id
        self.company_id = self.purchase_id.company_id

        # Copy purchase lines.
        po_lines = self.purchase_id.order_line - self.line_ids.mapped('purchase_line_id')
        new_lines = self.env['account.move.line']
        for line in po_lines.filtered(lambda l: not l.display_type):
            new_line = new_lines.new(line._prepare_account_move_line(self))
            new_line.account_id = new_line._get_computed_account()
            new_line._onchange_price_subtotal()
            new_lines += new_line
        new_lines._onchange_mark_recompute_taxes()

        # Compute invoice_origin.
        origins = set(self.line_ids.mapped('purchase_line_id.order_id.name'))
        self.invoice_origin = ','.join(list(origins))

        # Compute ref.
        refs = self._get_invoice_reference()
        self.ref = ', '.join(refs)

        # Compute invoice_payment_ref.
        if len(refs) == 1:
            self.invoice_payment_ref = refs[0]

        self.purchase_id = False
        self._onchange_currency()
        self.invoice_partner_bank_id = self.bank_partner_id.bank_ids and self.bank_partner_id.bank_ids[0]

    @api.onchange('partner_id', 'company_id')
    def _onchange_partner_id(self):
        res = super(AccountMove, self)._onchange_partner_id()
        if self.partner_id and\
                self.type in ['in_invoice', 'in_refund'] and\
                self.currency_id != self.partner_id.property_purchase_currency_id and\
                self.partner_id.property_purchase_currency_id.id:
            if not self.env.context.get('default_journal_id'):
                journal_domain = [
                    ('type', '=', 'purchase'),
                    ('company_id', '=', self.company_id.id),
                    ('currency_id', '=', self.partner_id.property_purchase_currency_id.id),
                ]
                default_journal_id = self.env['account.journal'].search(journal_domain, limit=1)
                if default_journal_id:
                    self.journal_id = default_journal_id
            if self.env.context.get('default_currency_id'):
                self.currency_id = self.env.context['default_currency_id']
            if self.partner_id.property_purchase_currency_id:
                self.currency_id = self.partner_id.property_purchase_currency_id
        return res

    @api.model_create_multi
    def create(self, vals_list):
        # OVERRIDE
        moves = super(AccountMove, self).create(vals_list)
        for move in moves:
            if move.reversed_entry_id:
                continue
            purchase = move.line_ids.mapped('purchase_line_id.order_id')
            if not purchase:
                continue
            refs = ["<a href=# data-oe-model=purchase.order data-oe-id=%s>%s</a>" % tuple(name_get) for name_get in purchase.name_get()]
            message = _("This vendor bill has been created from: %s") % ','.join(refs)
            move.message_post(body=message)
        return moves

    def write(self, vals):
        # OVERRIDE
        old_purchases = [move.mapped('line_ids.purchase_line_id.order_id') for move in self]
        res = super(AccountMove, self).write(vals)
        for i, move in enumerate(self):
            new_purchases = move.mapped('line_ids.purchase_line_id.order_id')
            if not new_purchases:
                continue
            diff_purchases = new_purchases - old_purchases[i]
            if diff_purchases:
                refs = ["<a href=# data-oe-model=purchase.order data-oe-id=%s>%s</a>" % tuple(name_get) for name_get in diff_purchases.name_get()]
                message = _("This vendor bill has been modified from: %s") % ','.join(refs)
                move.message_post(body=message)
        return res


class AccountMoveLine(models.Model):
    """ Override AccountInvoice_line to add the link to the purchase order line it is related to"""
    _inherit = 'account.move.line'

    purchase_line_id = fields.Many2one('purchase.order.line', 'Purchase Order Line', ondelete='set null', index=True)

    def _copy_data_extend_business_fields(self, values):
        # OVERRIDE to copy the 'purchase_line_id' field as well.
        super(AccountMoveLine, self)._copy_data_extend_business_fields(values)
        values['purchase_line_id'] = self.purchase_line_id.id

```

## File: models\product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import timedelta
from odoo import api, fields, models, _
from odoo.addons.base.models.res_partner import WARNING_MESSAGE, WARNING_HELP
from odoo.tools.float_utils import float_round


class ProductTemplate(models.Model):
    _name = 'product.template'
    _inherit = 'product.template'

    property_account_creditor_price_difference = fields.Many2one(
        'account.account', string="Price Difference Account", company_dependent=True,
        help="This account is used in automated inventory valuation to "\
             "record the price difference between a purchase order and its related vendor bill when validating this vendor bill.")
    purchased_product_qty = fields.Float(compute='_compute_purchased_product_qty', string='Purchased')
    purchase_method = fields.Selection([
        ('purchase', 'On ordered quantities'),
        ('receive', 'On received quantities'),
    ], string="Control Policy", help="On ordered quantities: Control bills based on ordered quantities.\n"
        "On received quantities: Control bills based on received quantities.", default="receive")
    purchase_line_warn = fields.Selection(WARNING_MESSAGE, 'Purchase Order Line', help=WARNING_HELP, required=True, default="no-message")
    purchase_line_warn_msg = fields.Text('Message for Purchase Order Line')

    def _compute_purchased_product_qty(self):
        for template in self:
            template.purchased_product_qty = float_round(sum([p.purchased_product_qty for p in template.product_variant_ids]), precision_rounding=template.uom_id.rounding)

    @api.model
    def get_import_templates(self):
        res = super(ProductTemplate, self).get_import_templates()
        if self.env.context.get('purchase_product_template'):
            return [{
                'label': _('Import Template for Products'),
                'template': '/purchase/static/xls/product_purchase.xls'
            }]
        return res

    def action_view_po(self):
        action = self.env.ref('purchase.action_purchase_order_report_all').read()[0]
        action['domain'] = ['&', ('state', 'in', ['purchase', 'done']), ('product_tmpl_id', 'in', self.ids)]
        action['context'] = {
            'graph_measure': 'qty_ordered',
            'search_default_orders': 1,
            'time_ranges': {'field': 'date_approve', 'range': 'last_365_days'}
        }
        return action


class ProductProduct(models.Model):
    _name = 'product.product'
    _inherit = 'product.product'

    purchased_product_qty = fields.Float(compute='_compute_purchased_product_qty', string='Purchased')

    def _compute_purchased_product_qty(self):
        date_from = fields.Datetime.to_string(fields.datetime.now() - timedelta(days=365))
        domain = [
            ('state', 'in', ['purchase', 'done']),
            ('product_id', 'in', self.ids),
            ('date_order', '>', date_from)
        ]
        order_lines = self.env['purchase.order.line'].read_group(domain, ['product_id', 'product_uom_qty'], ['product_id'])
        purchased_data = dict([(data['product_id'][0], data['product_uom_qty']) for data in order_lines])
        for product in self:
            if not product.id:
                product.purchased_product_qty = 0.0
                continue
            product.purchased_product_qty = float_round(purchased_data.get(product.id, 0), precision_rounding=product.uom_id.rounding)

    def action_view_po(self):
        action = self.env.ref('purchase.action_purchase_order_report_all').read()[0]
        action['domain'] = ['&', ('state', 'in', ['purchase', 'done']), ('product_id', 'in', self.ids)]
        action['context'] = {
            'search_default_last_year_purchase': 1,
            'search_default_status': 1, 'search_default_order_month': 1,
            'graph_measure': 'qty_ordered'
        }
        return action


class ProductCategory(models.Model):
    _inherit = "product.category"

    property_account_creditor_price_difference_categ = fields.Many2one(
        'account.account', string="Price Difference Account",
        company_dependent=True,
        help="This account will be used to value price difference between purchase price and accounting cost.")


class ProductSupplierinfo(models.Model):
    _inherit = "product.supplierinfo"

    @api.onchange('name')
    def _onchange_name(self):
        self.currency_id = self.name.property_purchase_currency_id.id or self.env.company.currency_id.id

```

## File: models\purchase.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import datetime
from dateutil.relativedelta import relativedelta

from odoo import api, fields, models, SUPERUSER_ID, _
from odoo.osv import expression
from odoo.tools import DEFAULT_SERVER_DATETIME_FORMAT
from odoo.tools.float_utils import float_compare
from odoo.exceptions import AccessError, UserError, ValidationError
from odoo.tools.misc import formatLang, get_lang


class PurchaseOrder(models.Model):
    _name = "purchase.order"
    _inherit = ['mail.thread', 'mail.activity.mixin', 'portal.mixin']
    _description = "Purchase Order"
    _order = 'date_order desc, id desc'

    def _default_currency_id(self):
        company_id = self.env.context.get('force_company') or self.env.context.get('company_id') or self.env.company.id
        return self.env['res.company'].browse(company_id).currency_id

    @api.depends('order_line.price_total')
    def _amount_all(self):
        for order in self:
            amount_untaxed = amount_tax = 0.0
            for line in order.order_line:
                line._compute_amount()
                amount_untaxed += line.price_subtotal
                amount_tax += line.price_tax
            currency = order.currency_id or order.partner_id.property_purchase_currency_id or self.env.company.currency_id
            order.update({
                'amount_untaxed': currency.round(amount_untaxed),
                'amount_tax': currency.round(amount_tax),
                'amount_total': amount_untaxed + amount_tax,
            })

    @api.depends('state', 'order_line.qty_invoiced', 'order_line.qty_received', 'order_line.product_qty')
    def _get_invoiced(self):
        precision = self.env['decimal.precision'].precision_get('Product Unit of Measure')
        for order in self:
            if order.state not in ('purchase', 'done'):
                order.invoice_status = 'no'
                continue

            if any(
                float_compare(
                    line.qty_invoiced,
                    line.product_qty if line.product_id.purchase_method == 'purchase' else line.qty_received,
                    precision_digits=precision,
                )
                == -1
                for line in order.order_line.filtered(lambda l: not l.display_type)
            ):
                order.invoice_status = 'to invoice'
            elif (
                all(
                    float_compare(
                        line.qty_invoiced,
                        line.product_qty if line.product_id.purchase_method == "purchase" else line.qty_received,
                        precision_digits=precision,
                    )
                    >= 0
                    for line in order.order_line.filtered(lambda l: not l.display_type)
                )
                and order.invoice_ids
            ):
                order.invoice_status = 'invoiced'
            else:
                order.invoice_status = 'no'

    @api.depends('order_line.invoice_lines.move_id')
    def _compute_invoice(self):
        for order in self:
            invoices = order.mapped('order_line.invoice_lines.move_id')
            order.invoice_ids = invoices
            order.invoice_count = len(invoices)

    READONLY_STATES = {
        'purchase': [('readonly', True)],
        'done': [('readonly', True)],
        'cancel': [('readonly', True)],
    }

    name = fields.Char('Order Reference', required=True, index=True, copy=False, default='New')
    origin = fields.Char('Source Document', copy=False,
        help="Reference of the document that generated this purchase order "
             "request (e.g. a sales order)")
    partner_ref = fields.Char('Vendor Reference', copy=False,
        help="Reference of the sales order or bid sent by the vendor. "
             "It's used to do the matching when you receive the "
             "products as this reference is usually written on the "
             "delivery order sent by your vendor.")
    date_order = fields.Datetime('Order Date', required=True, states=READONLY_STATES, index=True, copy=False, default=fields.Datetime.now,\
        help="Depicts the date where the Quotation should be validated and converted into a purchase order.")
    date_approve = fields.Datetime('Confirmation Date', readonly=1, index=True, copy=False)
    partner_id = fields.Many2one('res.partner', string='Vendor', required=True, states=READONLY_STATES, change_default=True, tracking=True, domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]", help="You can find a vendor by its Name, TIN, Email or Internal Reference.")
    dest_address_id = fields.Many2one('res.partner', domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]", string='Drop Ship Address', states=READONLY_STATES,
        help="Put an address if you want to deliver directly from the vendor to the customer. "
             "Otherwise, keep empty to deliver to your own company.")
    currency_id = fields.Many2one('res.currency', 'Currency', required=True, states=READONLY_STATES,
        default=_default_currency_id)
    state = fields.Selection([
        ('draft', 'RFQ'),
        ('sent', 'RFQ Sent'),
        ('to approve', 'To Approve'),
        ('purchase', 'Purchase Order'),
        ('done', 'Locked'),
        ('cancel', 'Cancelled')
    ], string='Status', readonly=True, index=True, copy=False, default='draft', tracking=True)
    order_line = fields.One2many('purchase.order.line', 'order_id', string='Order Lines', states={'cancel': [('readonly', True)], 'done': [('readonly', True)]}, copy=True)
    notes = fields.Text('Terms and Conditions')

    invoice_count = fields.Integer(compute="_compute_invoice", string='Bill Count', copy=False, default=0, store=True)
    invoice_ids = fields.Many2many('account.move', compute="_compute_invoice", string='Bills', copy=False, store=True)
    invoice_status = fields.Selection([
        ('no', 'Nothing to Bill'),
        ('to invoice', 'Waiting Bills'),
        ('invoiced', 'Fully Billed'),
    ], string='Billing Status', compute='_get_invoiced', store=True, readonly=True, copy=False, default='no')

    # There is no inverse function on purpose since the date may be different on each line
    date_planned = fields.Datetime(string='Receipt Date', index=True)

    amount_untaxed = fields.Monetary(string='Untaxed Amount', store=True, readonly=True, compute='_amount_all', tracking=True)
    amount_tax = fields.Monetary(string='Taxes', store=True, readonly=True, compute='_amount_all')
    amount_total = fields.Monetary(string='Total', store=True, readonly=True, compute='_amount_all')

    fiscal_position_id = fields.Many2one('account.fiscal.position', string='Fiscal Position', domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    payment_term_id = fields.Many2one('account.payment.term', 'Payment Terms', domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    incoterm_id = fields.Many2one('account.incoterms', 'Incoterm', states={'done': [('readonly', True)]}, help="International Commercial Terms are a series of predefined commercial terms used in international transactions.")

    product_id = fields.Many2one('product.product', related='order_line.product_id', string='Product', readonly=False)
    user_id = fields.Many2one(
        'res.users', string='Purchase Representative', index=True, tracking=True,
        default=lambda self: self.env.user, check_company=True)
    company_id = fields.Many2one('res.company', 'Company', required=True, index=True, states=READONLY_STATES, default=lambda self: self.env.company.id)
    currency_rate = fields.Float("Currency Rate", compute='_compute_currency_rate', compute_sudo=True, store=True, readonly=True, help='Ratio between the purchase order currency and the company currency')

    @api.constrains('company_id', 'order_line')
    def _check_order_line_company_id(self):
        for order in self:
            companies = order.order_line.product_id.company_id
            if companies and companies != order.company_id:
                bad_products = order.order_line.product_id.filtered(lambda p: p.company_id and p.company_id != order.company_id)
                raise ValidationError((_("Your quotation contains products from company %s whereas your quotation belongs to company %s. \n Please change the company of your quotation or remove the products from other companies (%s).") % (', '.join(companies.mapped('display_name')), order.company_id.display_name, ', '.join(bad_products.mapped('display_name')))))

    def _compute_access_url(self):
        super(PurchaseOrder, self)._compute_access_url()
        for order in self:
            order.access_url = '/my/purchase/%s' % (order.id)

    @api.model
    def _name_search(self, name, args=None, operator='ilike', limit=100, name_get_uid=None):
        args = args or []
        domain = []
        if name:
            domain = ['|', ('name', operator, name), ('partner_ref', operator, name)]
        purchase_order_ids = self._search(expression.AND([domain, args]), limit=limit, access_rights_uid=name_get_uid)
        return models.lazy_name_get(self.browse(purchase_order_ids).with_user(name_get_uid))

    @api.depends('date_order', 'currency_id', 'company_id', 'company_id.currency_id')
    def _compute_currency_rate(self):
        for order in self:
            order.currency_rate = self.env['res.currency']._get_conversion_rate(order.company_id.currency_id, order.currency_id, order.company_id, order.date_order)

    @api.depends('name', 'partner_ref')
    def name_get(self):
        result = []
        for po in self:
            name = po.name
            if po.partner_ref:
                name += ' (' + po.partner_ref + ')'
            if self.env.context.get('show_total_amount') and po.amount_total:
                name += ': ' + formatLang(self.env, po.amount_total, currency_obj=po.currency_id)
            result.append((po.id, name))
        return result

    @api.model
    def create(self, vals):
        company_id = vals.get('company_id', self.default_get(['company_id'])['company_id'])
        if vals.get('name', 'New') == 'New':
            seq_date = None
            if 'date_order' in vals:
                seq_date = fields.Datetime.context_timestamp(self, fields.Datetime.to_datetime(vals['date_order']))
            vals['name'] = self.env['ir.sequence'].with_context(force_company=company_id).next_by_code('purchase.order', sequence_date=seq_date) or '/'
        return super(PurchaseOrder, self.with_context(company_id=company_id)).create(vals)

    def write(self, vals):
        res = super(PurchaseOrder, self).write(vals)
        if vals.get('date_planned'):
            self.order_line.filtered(lambda line: not line.display_type).date_planned = vals['date_planned']
        return res

    def unlink(self):
        for order in self:
            if not order.state == 'cancel':
                raise UserError(_('In order to delete a purchase order, you must cancel it first.'))
        return super(PurchaseOrder, self).unlink()

    def copy(self, default=None):
        ctx = dict(self.env.context)
        ctx.pop('default_product_id', None)
        self = self.with_context(ctx)
        new_po = super(PurchaseOrder, self).copy(default=default)
        for line in new_po.order_line:
            if new_po.date_planned and not line.display_type:
                line.date_planned = new_po.date_planned
            elif line.product_id:
                seller = line.product_id._select_seller(
                    partner_id=line.partner_id, quantity=line.product_qty,
                    date=line.order_id.date_order and line.order_id.date_order.date(), uom_id=line.product_uom)
                line.date_planned = line._get_date_planned(seller)
        return new_po

    def _track_subtype(self, init_values):
        self.ensure_one()
        if 'state' in init_values and self.state == 'purchase':
            if init_values['state'] == 'to approve':
                return self.env.ref('purchase.mt_rfq_approved')
            return self.env.ref('purchase.mt_rfq_confirmed')
        elif 'state' in init_values and self.state == 'to approve':
            return self.env.ref('purchase.mt_rfq_confirmed')
        elif 'state' in init_values and self.state == 'done':
            return self.env.ref('purchase.mt_rfq_done')
        return super(PurchaseOrder, self)._track_subtype(init_values)

    @api.onchange('partner_id', 'company_id')
    def onchange_partner_id(self):
        # Ensures all properties and fiscal positions
        # are taken with the company of the order
        # if not defined, force_company doesn't change anything.
        self = self.with_context(force_company=self.company_id.id)
        if not self.partner_id:
            self.fiscal_position_id = False
            self.currency_id = self.env.company.currency_id.id
        else:
            self.fiscal_position_id = self.env['account.fiscal.position'].get_fiscal_position(self.partner_id.id)
            self.payment_term_id = self.partner_id.property_supplier_payment_term_id.id
            self.currency_id = self.partner_id.property_purchase_currency_id.id or self.env.company.currency_id.id
        return {}

    @api.onchange('fiscal_position_id')
    def _compute_tax_id(self):
        """
        Trigger the recompute of the taxes if the fiscal position is changed on the PO.
        """
        for order in self:
            order.order_line._compute_tax_id()

    @api.onchange('partner_id')
    def onchange_partner_id_warning(self):
        if not self.partner_id or not self.env.user.has_group('purchase.group_warning_purchase'):
            return
        warning = {}
        title = False
        message = False

        partner = self.partner_id

        # If partner has no warning, check its company
        if partner.purchase_warn == 'no-message' and partner.parent_id:
            partner = partner.parent_id

        if partner.purchase_warn and partner.purchase_warn != 'no-message':
            # Block if partner only has warning but parent company is blocked
            if partner.purchase_warn != 'block' and partner.parent_id and partner.parent_id.purchase_warn == 'block':
                partner = partner.parent_id
            title = _("Warning for %s") % partner.name
            message = partner.purchase_warn_msg
            warning = {
                'title': title,
                'message': message
            }
            if partner.purchase_warn == 'block':
                self.update({'partner_id': False})
            return {'warning': warning}
        return {}

    def action_rfq_send(self):
        '''
        This function opens a window to compose an email, with the edi purchase template message loaded by default
        '''
        self.ensure_one()
        ir_model_data = self.env['ir.model.data']
        try:
            if self.env.context.get('send_rfq', False):
                template_id = ir_model_data.get_object_reference('purchase', 'email_template_edi_purchase')[1]
            else:
                template_id = ir_model_data.get_object_reference('purchase', 'email_template_edi_purchase_done')[1]
        except ValueError:
            template_id = False
        try:
            compose_form_id = ir_model_data.get_object_reference('mail', 'email_compose_message_wizard_form')[1]
        except ValueError:
            compose_form_id = False
        ctx = dict(self.env.context or {})
        ctx.update({
            'default_model': 'purchase.order',
            'active_model': 'purchase.order',
            'active_id': self.ids[0],
            'default_res_id': self.ids[0],
            'default_use_template': bool(template_id),
            'default_template_id': template_id,
            'default_composition_mode': 'comment',
            'custom_layout': "mail.mail_notification_paynow",
            'force_email': True,
            'mark_rfq_as_sent': True,
        })

        # In the case of a RFQ or a PO, we want the "View..." button in line with the state of the
        # object. Therefore, we pass the model description in the context, in the language in which
        # the template is rendered.
        lang = self.env.context.get('lang')
        if {'default_template_id', 'default_model', 'default_res_id'} <= ctx.keys():
            template = self.env['mail.template'].browse(ctx['default_template_id'])
            if template and template.lang:
                lang = template._render_template(template.lang, ctx['default_model'], ctx['default_res_id'])

        self = self.with_context(lang=lang)
        if self.state in ['draft', 'sent']:
            ctx['model_description'] = _('Request for Quotation')
        else:
            ctx['model_description'] = _('Purchase Order')

        return {
            'name': _('Compose Email'),
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'mail.compose.message',
            'views': [(compose_form_id, 'form')],
            'view_id': compose_form_id,
            'target': 'new',
            'context': ctx,
        }

    @api.returns('mail.message', lambda value: value.id)
    def message_post(self, **kwargs):
        if self.env.context.get('mark_rfq_as_sent'):
            self.filtered(lambda o: o.state == 'draft').write({'state': 'sent'})
        return super(PurchaseOrder, self.with_context(mail_post_autofollow=True)).message_post(**kwargs)

    def print_quotation(self):
        self.write({'state': "sent"})
        return self.env.ref('purchase.report_purchase_quotation').report_action(self)

    def button_approve(self, force=False):
        self.write({'state': 'purchase', 'date_approve': fields.Datetime.now()})
        self.filtered(lambda p: p.company_id.po_lock == 'lock').write({'state': 'done'})
        return {}

    def button_draft(self):
        self.write({'state': 'draft'})
        return {}

    def button_confirm(self):
        for order in self:
            if order.state not in ['draft', 'sent']:
                continue
            order._add_supplier_to_product()
            # Deal with double validation process
            if order.company_id.po_double_validation == 'one_step'\
                    or (order.company_id.po_double_validation == 'two_step'\
                        and order.amount_total < self.env.company.currency_id._convert(
                            order.company_id.po_double_validation_amount, order.currency_id, order.company_id, order.date_order or fields.Date.today()))\
                    or order.user_has_groups('purchase.group_purchase_manager'):
                order.button_approve()
            else:
                order.write({'state': 'to approve'})
        return True

    def button_cancel(self):
        for order in self:
            for inv in order.invoice_ids:
                if inv and inv.state not in ('cancel', 'draft'):
                    raise UserError(_("Unable to cancel this purchase order. You must first cancel the related vendor bills."))

        self.write({'state': 'cancel'})

    def button_unlock(self):
        self.write({'state': 'purchase'})

    def button_done(self):
        self.write({'state': 'done'})

    def _add_supplier_to_product(self):
        # Add the partner in the supplier list of the product if the supplier is not registered for
        # this product. We limit to 10 the number of suppliers for a product to avoid the mess that
        # could be caused for some generic products ("Miscellaneous").
        for line in self.order_line:
            # Do not add a contact as a supplier
            partner = self.partner_id if not self.partner_id.parent_id else self.partner_id.parent_id
            if line.product_id and partner not in line.product_id.seller_ids.mapped('name') and len(line.product_id.seller_ids) <= 10:
                # Convert the price in the right currency.
                currency = partner.property_purchase_currency_id or self.env.company.currency_id
                price = self.currency_id._convert(line.price_unit, currency, line.company_id, line.date_order or fields.Date.today(), round=False)
                # Compute the price for the template's UoM, because the supplier's UoM is related to that UoM.
                if line.product_id.product_tmpl_id.uom_po_id != line.product_uom:
                    default_uom = line.product_id.product_tmpl_id.uom_po_id
                    price = line.product_uom._compute_price(price, default_uom)

                supplierinfo = {
                    'name': partner.id,
                    'sequence': max(line.product_id.seller_ids.mapped('sequence')) + 1 if line.product_id.seller_ids else 1,
                    'min_qty': 0.0,
                    'price': price,
                    'currency_id': currency.id,
                    'delay': 0,
                }
                # In case the order partner is a contact address, a new supplierinfo is created on
                # the parent company. In this case, we keep the product name and code.
                seller = line.product_id._select_seller(
                    partner_id=line.partner_id,
                    quantity=line.product_qty,
                    date=line.order_id.date_order and line.order_id.date_order.date(),
                    uom_id=line.product_uom)
                if seller:
                    supplierinfo['product_name'] = seller.product_name
                    supplierinfo['product_code'] = seller.product_code
                vals = {
                    'seller_ids': [(0, 0, supplierinfo)],
                }
                try:
                    line.product_id.write(vals)
                except AccessError:  # no write access rights -> just ignore
                    break

    def action_view_invoice(self):
        '''
        This function returns an action that display existing vendor bills of given purchase order ids.
        When only one found, show the vendor bill immediately.
        '''
        action = self.env.ref('account.action_move_in_invoice_type')
        result = action.read()[0]
        create_bill = self.env.context.get('create_bill', False)
        # override the context to get rid of the default filtering
        result['context'] = {
            'default_type': 'in_invoice',
            'default_company_id': self.company_id.id,
            'default_purchase_id': self.id,
            'default_partner_id': self.partner_id.id,
        }
        # Invoice_ids may be filtered depending on the user. To ensure we get all
        # invoices related to the purchase order, we read them in sudo to fill the
        # cache.
        self.sudo()._read(['invoice_ids'])
        # choose the view_mode accordingly
        if len(self.invoice_ids) > 1 and not create_bill:
            result['domain'] = "[('id', 'in', " + str(self.invoice_ids.ids) + ")]"
        else:
            res = self.env.ref('account.view_move_form', False)
            form_view = [(res and res.id or False, 'form')]
            if 'views' in result:
                result['views'] = form_view + [(state,view) for state,view in action['views'] if view != 'form']
            else:
                result['views'] = form_view
            # Do not set an invoice_id if we want to create a new bill.
            if not create_bill:
                result['res_id'] = self.invoice_ids.id or False
        result['context']['default_invoice_origin'] = self.name
        result['context']['default_ref'] = self.partner_ref
        return result


class PurchaseOrderLine(models.Model):
    _name = 'purchase.order.line'
    _description = 'Purchase Order Line'
    _order = 'order_id, sequence, id'

    name = fields.Text(string='Description', required=True)
    sequence = fields.Integer(string='Sequence', default=10)
    product_qty = fields.Float(string='Quantity', digits='Product Unit of Measure', required=True)
    product_uom_qty = fields.Float(string='Total Quantity', compute='_compute_product_uom_qty', store=True)
    date_planned = fields.Datetime(string='Scheduled Date', index=True)
    taxes_id = fields.Many2many('account.tax', string='Taxes', domain=['|', ('active', '=', False), ('active', '=', True)])
    product_uom = fields.Many2one('uom.uom', string='Unit of Measure', domain="[('category_id', '=', product_uom_category_id)]")
    product_uom_category_id = fields.Many2one(related='product_id.uom_id.category_id')
    product_id = fields.Many2one('product.product', string='Product', domain=[('purchase_ok', '=', True)], change_default=True)
    product_type = fields.Selection(related='product_id.type', readonly=True)
    price_unit = fields.Float(string='Unit Price', required=True, digits='Product Price')

    price_subtotal = fields.Monetary(compute='_compute_amount', string='Subtotal', store=True)
    price_total = fields.Monetary(compute='_compute_amount', string='Total', store=True)
    price_tax = fields.Float(compute='_compute_amount', string='Tax', store=True)

    order_id = fields.Many2one('purchase.order', string='Order Reference', index=True, required=True, ondelete='cascade')
    account_analytic_id = fields.Many2one('account.analytic.account', string='Analytic Account')
    analytic_tag_ids = fields.Many2many('account.analytic.tag', string='Analytic Tags')
    company_id = fields.Many2one('res.company', related='order_id.company_id', string='Company', store=True, readonly=True)
    state = fields.Selection(related='order_id.state', store=True, readonly=False)

    invoice_lines = fields.One2many('account.move.line', 'purchase_line_id', string="Bill Lines", readonly=True, copy=False)

    # Replace by invoiced Qty
    qty_invoiced = fields.Float(compute='_compute_qty_invoiced', string="Billed Qty", digits='Product Unit of Measure', store=True)

    qty_received_method = fields.Selection([('manual', 'Manual')], string="Received Qty Method", compute='_compute_qty_received_method', store=True,
        help="According to product configuration, the recieved quantity can be automatically computed by mechanism :\n"
             "  - Manual: the quantity is set manually on the line\n"
             "  - Stock Moves: the quantity comes from confirmed pickings\n")
    qty_received = fields.Float("Received Qty", compute='_compute_qty_received', inverse='_inverse_qty_received', compute_sudo=True, store=True, digits='Product Unit of Measure')
    qty_received_manual = fields.Float("Manual Received Qty", digits='Product Unit of Measure', copy=False)

    partner_id = fields.Many2one('res.partner', related='order_id.partner_id', string='Partner', readonly=True, store=True)
    currency_id = fields.Many2one(related='order_id.currency_id', store=True, string='Currency', readonly=True)
    date_order = fields.Datetime(related='order_id.date_order', string='Order Date', readonly=True)

    display_type = fields.Selection([
        ('line_section', "Section"),
        ('line_note', "Note")], default=False, help="Technical field for UX purpose.")

    _sql_constraints = [
        ('accountable_required_fields',
            "CHECK(display_type IS NOT NULL OR (product_id IS NOT NULL AND product_uom IS NOT NULL AND date_planned IS NOT NULL))",
            "Missing required fields on accountable purchase order line."),
        ('non_accountable_null_fields',
            "CHECK(display_type IS NULL OR (product_id IS NULL AND price_unit = 0 AND product_uom_qty = 0 AND product_uom IS NULL AND date_planned is NULL))",
            "Forbidden values on non-accountable purchase order line"),
    ]

    @api.depends('product_qty', 'price_unit', 'taxes_id')
    def _compute_amount(self):
        for line in self:
            vals = line._prepare_compute_all_values()
            taxes = line.taxes_id.compute_all(
                vals['price_unit'],
                vals['currency_id'],
                vals['product_qty'],
                vals['product'],
                vals['partner'])
            line.update({
                'price_tax': sum(t.get('amount', 0.0) for t in taxes.get('taxes', [])),
                'price_total': taxes['total_included'],
                'price_subtotal': taxes['total_excluded'],
            })

    def _prepare_compute_all_values(self):
        # Hook method to returns the different argument values for the
        # compute_all method, due to the fact that discounts mechanism
        # is not implemented yet on the purchase orders.
        # This method should disappear as soon as this feature is
        # also introduced like in the sales module.
        self.ensure_one()
        return {
            'price_unit': self.price_unit,
            'currency_id': self.order_id.currency_id,
            'product_qty': self.product_qty,
            'product': self.product_id,
            'partner': self.order_id.partner_id,
        }

    def _compute_tax_id(self):
        for line in self:
            fpos = line.order_id.fiscal_position_id or line.order_id.partner_id.with_context(force_company=line.company_id.id).property_account_position_id
            # If company_id is set in the order, always filter taxes by the company
            taxes = line.product_id.supplier_taxes_id.filtered(lambda r: r.company_id == line.order_id.company_id)
            line.taxes_id = fpos.map_tax(taxes, line.product_id, line.order_id.partner_id) if fpos else taxes

    @api.depends('invoice_lines.move_id.state', 'invoice_lines.quantity')
    def _compute_qty_invoiced(self):
        for line in self:
            qty = 0.0
            for inv_line in line.invoice_lines:
                if inv_line.move_id.state not in ['cancel']:
                    if inv_line.move_id.type == 'in_invoice':
                        qty += inv_line.product_uom_id._compute_quantity(inv_line.quantity, line.product_uom)
                    elif inv_line.move_id.type == 'in_refund':
                        qty -= inv_line.product_uom_id._compute_quantity(inv_line.quantity, line.product_uom)
            line.qty_invoiced = qty

    @api.depends('product_id')
    def _compute_qty_received_method(self):
        for line in self:
            if line.product_id and line.product_id.type in ['consu', 'service']:
                line.qty_received_method = 'manual'
            else:
                line.qty_received_method = False

    @api.depends('qty_received_method', 'qty_received_manual')
    def _compute_qty_received(self):
        for line in self:
            if line.qty_received_method == 'manual':
                line.qty_received = line.qty_received_manual or 0.0
            else:
                line.qty_received = 0.0

    @api.onchange('qty_received')
    def _inverse_qty_received(self):
        """ When writing on qty_received, if the value should be modify manually (`qty_received_method` = 'manual' only),
            then we put the value in `qty_received_manual`. Otherwise, `qty_received_manual` should be False since the
            received qty is automatically compute by other mecanisms.
        """
        for line in self:
            if line.qty_received_method == 'manual':
                line.qty_received_manual = line.qty_received
            else:
                line.qty_received_manual = 0.0

    @api.model
    def create(self, values):
        if values.get('display_type', self.default_get(['display_type'])['display_type']):
            values.update(product_id=False, price_unit=0, product_uom_qty=0, product_uom=False, date_planned=False)

        order_id = values.get('order_id')
        if 'date_planned' not in values:
            order = self.env['purchase.order'].browse(order_id)
            if order.date_planned:
                values['date_planned'] = order.date_planned
        line = super(PurchaseOrderLine, self).create(values)
        if line.order_id.state == 'purchase':
            msg = _("Extra line with %s ") % (line.product_id.display_name,)
            line.order_id.message_post(body=msg)
        return line

    def write(self, values):
        if 'display_type' in values and self.filtered(lambda line: line.display_type != values.get('display_type')):
            raise UserError(_("You cannot change the type of a purchase order line. Instead you should delete the current line and create a new line of the proper type."))

        if 'product_qty' in values:
            for line in self:
                if line.order_id.state == 'purchase':
                    line.order_id.message_post_with_view('purchase.track_po_line_template',
                                                         values={'line': line, 'product_qty': values['product_qty']},
                                                         subtype_id=self.env.ref('mail.mt_note').id)
        return super(PurchaseOrderLine, self).write(values)

    def unlink(self):
        for line in self:
            if line.order_id.state in ['purchase', 'done']:
                raise UserError(_('Cannot delete a purchase order line which is in state \'%s\'.') % (line.state,))
        return super(PurchaseOrderLine, self).unlink()

    @api.model
    def _get_date_planned(self, seller, po=False):
        """Return the datetime value to use as Schedule Date (``date_planned``) for
           PO Lines that correspond to the given product.seller_ids,
           when ordered at `date_order_str`.

           :param Model seller: used to fetch the delivery delay (if no seller
                                is provided, the delay is 0)
           :param Model po: purchase.order, necessary only if the PO line is
                            not yet attached to a PO.
           :rtype: datetime
           :return: desired Schedule Date for the PO line
        """
        date_order = po.date_order if po else self.order_id.date_order
        if date_order:
            return date_order + relativedelta(days=seller.delay if seller else 0)
        else:
            return datetime.today() + relativedelta(days=seller.delay if seller else 0)

    @api.onchange('product_id')
    def onchange_product_id(self):
        if not self.product_id:
            return

        # Reset date, price and quantity since _onchange_quantity will provide default values
        self.date_planned = datetime.today().strftime(DEFAULT_SERVER_DATETIME_FORMAT)
        self.price_unit = self.product_qty = 0.0

        self._product_id_change()

        self._suggest_quantity()
        self._onchange_quantity()

    def _product_id_change(self):
        if not self.product_id:
            return

        self.product_uom = self.product_id.uom_po_id or self.product_id.uom_id
        product_lang = self.product_id.with_context(
            lang=get_lang(self.env, self.partner_id.lang).code,
            partner_id=self.partner_id.id,
            company_id=self.company_id.id,
        )
        self.name = self._get_product_purchase_description(product_lang)

        self._compute_tax_id()

    @api.onchange('product_id')
    def onchange_product_id_warning(self):
        if not self.product_id or not self.env.user.has_group('purchase.group_warning_purchase'):
            return
        warning = {}
        title = False
        message = False

        product_info = self.product_id

        if product_info.purchase_line_warn != 'no-message':
            title = _("Warning for %s") % product_info.name
            message = product_info.purchase_line_warn_msg
            warning['title'] = title
            warning['message'] = message
            if product_info.purchase_line_warn == 'block':
                self.product_id = False
            return {'warning': warning}
        return {}

    @api.onchange('product_qty', 'product_uom')
    def _onchange_quantity(self):
        if not self.product_id or self.invoice_lines:
            return
        params = {'order_id': self.order_id}
        seller = self.product_id._select_seller(
            partner_id=self.partner_id,
            quantity=self.product_qty,
            date=self.order_id.date_order and self.order_id.date_order.date(),
            uom_id=self.product_uom,
            params=params)

        if seller or not self.date_planned:
            self.date_planned = self._get_date_planned(seller).strftime(DEFAULT_SERVER_DATETIME_FORMAT)

        if not seller:
            if self.product_id.seller_ids.filtered(lambda s: s.name.id == self.partner_id.id):
                self.price_unit = 0.0
            return

        price_unit = self.env['account.tax']._fix_tax_included_price_company(seller.price, self.product_id.supplier_taxes_id, self.taxes_id, self.company_id) if seller else 0.0
        if price_unit and seller and self.order_id.currency_id and seller.currency_id != self.order_id.currency_id:
            price_unit = seller.currency_id._convert(
                price_unit, self.order_id.currency_id, self.order_id.company_id, self.date_order or fields.Date.today())

        if seller and self.product_uom and seller.product_uom != self.product_uom:
            price_unit = seller.product_uom._compute_price(price_unit, self.product_uom)

        self.price_unit = price_unit

        default_names = []
        vendors = self.product_id._prepare_sellers({})
        for vendor in vendors:
            product_ctx = {'seller_id': vendor.id, 'lang': get_lang(self.env, self.partner_id.lang).code}
            default_names.append(self._get_product_purchase_description(self.product_id.with_context(product_ctx)))

        if (self.name in default_names or not self.name):
            product_ctx = {'seller_id': seller.id, 'lang': get_lang(self.env, self.partner_id.lang).code}
            self.name = self._get_product_purchase_description(self.product_id.with_context(product_ctx))

    @api.depends('product_uom', 'product_qty', 'product_id.uom_id')
    def _compute_product_uom_qty(self):
        for line in self:
            if line.product_id and line.product_id.uom_id != line.product_uom:
                line.product_uom_qty = line.product_uom._compute_quantity(line.product_qty, line.product_id.uom_id)
            else:
                line.product_uom_qty = line.product_qty

    def _suggest_quantity(self):
        '''
        Suggest a minimal quantity based on the seller
        '''
        if not self.product_id:
            return
        seller_min_qty = self.product_id.seller_ids\
            .filtered(lambda r: r.name == self.order_id.partner_id and (not r.product_id or r.product_id == self.product_id))\
            .sorted(key=lambda r: r.min_qty)
        if seller_min_qty:
            self.product_qty = seller_min_qty[0].min_qty or 1.0
            self.product_uom = seller_min_qty[0].product_uom
        else:
            self.product_qty = 1.0

    def _get_product_purchase_description(self, product_lang):
        self.ensure_one()
        name = product_lang.display_name
        if product_lang.description_purchase:
            name += '\n' + product_lang.description_purchase

        return name

    def _prepare_account_move_line(self, move):
        self.ensure_one()
        if self.product_id.purchase_method == 'purchase':
            qty = self.product_qty - self.qty_invoiced
        else:
            qty = self.qty_received - self.qty_invoiced
        if float_compare(qty, 0.0, precision_rounding=self.product_uom.rounding) <= 0:
            qty = 0.0

        return {
            'name': '%s: %s' % (self.order_id.name, self.name),
            'move_id': move.id,
            'currency_id': move.currency_id.id,
            'purchase_line_id': self.id,
            'date_maturity': move.invoice_date_due,
            'product_uom_id': self.product_uom.id,
            'product_id': self.product_id.id,
            'price_unit': self.price_unit,
            'quantity': qty,
            'partner_id': move.commercial_partner_id.id,
            'analytic_account_id': self.account_analytic_id.id,
            'analytic_tag_ids': [(6, 0, self.analytic_tag_ids.ids)],
            'tax_ids': [(6, 0, self.taxes_id.ids)],
            'display_type': self.display_type,
        }

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models

class Company(models.Model):
    _inherit = 'res.company'

    po_lead = fields.Float(string='Purchase Lead Time', required=True,
        help="Margin of error for vendor lead times. When the system "
             "generates Purchase Orders for procuring products, "
             "they will be scheduled that many days earlier "
             "to cope with unexpected vendor delays.", default=0.0)

    po_lock = fields.Selection([
        ('edit', 'Allow to edit purchase orders'),
        ('lock', 'Confirmed purchase orders are not editable')
        ], string="Purchase Order Modification", default="edit",
        help='Purchase Order Modification used when you want to purchase order editable after confirm')

    po_double_validation = fields.Selection([
        ('one_step', 'Confirm purchase orders in one step'),
        ('two_step', 'Get 2 levels of approvals to confirm a purchase order')
        ], string="Levels of Approvals", default='one_step',
        help="Provide a double validation mechanism for purchases")

    po_double_validation_amount = fields.Monetary(string='Double validation amount', default=5000,
        help="Minimum amount for which a double validation is required")

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    lock_confirmed_po = fields.Boolean("Lock Confirmed Orders", default=lambda self: self.env.company.po_lock == 'lock')
    po_lock = fields.Selection(related='company_id.po_lock', string="Purchase Order Modification *", readonly=False)
    po_order_approval = fields.Boolean("Purchase Order Approval", default=lambda self: self.env.company.po_double_validation == 'two_step')
    po_double_validation = fields.Selection(related='company_id.po_double_validation', string="Levels of Approvals *", readonly=False)
    po_double_validation_amount = fields.Monetary(related='company_id.po_double_validation_amount', string="Minimum Amount", currency_field='company_currency_id', readonly=False)
    company_currency_id = fields.Many2one('res.currency', related='company_id.currency_id', string="Company Currency", readonly=True,
        help='Utility field to express amount currency')
    default_purchase_method = fields.Selection([
        ('purchase', 'Ordered quantities'),
        ('receive', 'Received quantities'),
        ], string="Bill Control", default_model="product.template",
        help="This default value is applied to any new product created. "
        "This can be changed in the product detail form.", default="receive")
    group_warning_purchase = fields.Boolean("Purchase Warnings", implied_group='purchase.group_warning_purchase')
    module_account_3way_match = fields.Boolean("3-way matching: purchases, receptions and bills")
    module_purchase_requisition = fields.Boolean("Purchase Agreements")
    module_purchase_product_matrix = fields.Boolean("Purchase Grid Entry")
    po_lead = fields.Float(related='company_id.po_lead', readonly=False)
    use_po_lead = fields.Boolean(
        string="Security Lead Time for Purchase",
        config_parameter='purchase.use_po_lead',
        help="Margin of error for vendor lead times. When the system generates Purchase Orders for reordering products,they will be scheduled that many days earlier to cope with unexpected vendor delays.")

    @api.onchange('use_po_lead')
    def _onchange_use_po_lead(self):
        if not self.use_po_lead:
            self.po_lead = 0.0

    def set_values(self):
        super(ResConfigSettings, self).set_values()
        self.po_lock = 'lock' if self.lock_confirmed_po else 'edit'
        self.po_double_validation = 'two_step' if self.po_order_approval else 'one_step'

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo.addons.base.models.res_partner import WARNING_MESSAGE, WARNING_HELP


class res_partner(models.Model):
    _name = 'res.partner'
    _inherit = 'res.partner'

    def _compute_purchase_order_count(self):
        # retrieve all children partners and prefetch 'parent_id' on them
        all_partners = self.with_context(active_test=False).search([('id', 'child_of', self.ids)])
        all_partners.read(['parent_id'])

        purchase_order_groups = self.env['purchase.order'].read_group(
            domain=[('partner_id', 'in', all_partners.ids)],
            fields=['partner_id'], groupby=['partner_id']
        )
        partners = self.browse()
        for group in purchase_order_groups:
            partner = self.browse(group['partner_id'][0])
            while partner:
                if partner in self:
                    partner.purchase_order_count += group['partner_id_count']
                    partners |= partner
                partner = partner.parent_id
        (self - partners).purchase_order_count = 0

    def _compute_supplier_invoice_count(self):
        # retrieve all children partners and prefetch 'parent_id' on them
        all_partners = self.with_context(active_test=False).search([('id', 'child_of', self.ids)])
        all_partners.read(['parent_id'])

        supplier_invoice_groups = self.env['account.move'].read_group(
            domain=[('partner_id', 'in', all_partners.ids),
                    ('type', 'in', ('in_invoice', 'in_refund'))],
            fields=['partner_id'], groupby=['partner_id']
        )
        partners = self.browse()
        for group in supplier_invoice_groups:
            partner = self.browse(group['partner_id'][0])
            while partner:
                if partner in self:
                    partner.supplier_invoice_count += group['partner_id_count']
                    partners |= partner
                partner = partner.parent_id
        (self - partners).supplier_invoice_count = 0

    @api.model
    def _commercial_fields(self):
        return super(res_partner, self)._commercial_fields()

    property_purchase_currency_id = fields.Many2one(
        'res.currency', string="Supplier Currency", company_dependent=True,
        help="This currency will be used, instead of the default one, for purchases from the current partner")
    purchase_order_count = fields.Integer(compute='_compute_purchase_order_count', string='Purchase Order Count')
    supplier_invoice_count = fields.Integer(compute='_compute_supplier_invoice_count', string='# Vendor Bills')
    purchase_warn = fields.Selection(WARNING_MESSAGE, 'Purchase Order', help=WARNING_HELP, default="no-message")
    purchase_warn_msg = fields.Text('Message for Purchase Order')

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_invoice
from . import purchase
from . import product
from . import res_company
from . import res_config_settings
from . import res_partner

```

## File: report\purchase_bill.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, tools
from odoo.osv import expression
from odoo.tools import formatLang

class PurchaseBillUnion(models.Model):
    _name = 'purchase.bill.union'
    _auto = False
    _description = 'Purchases & Bills Union'
    _order = "date desc, name desc"

    name = fields.Char(string='Reference', readonly=True)
    reference = fields.Char(string='Source', readonly=True)
    partner_id = fields.Many2one('res.partner', string='Vendor', readonly=True)
    date = fields.Date(string='Date', readonly=True)
    amount = fields.Float(string='Amount', readonly=True)
    currency_id = fields.Many2one('res.currency', string='Currency', readonly=True)
    company_id = fields.Many2one('res.company', 'Company', readonly=True)
    vendor_bill_id = fields.Many2one('account.move', string='Vendor Bill', readonly=True)
    purchase_order_id = fields.Many2one('purchase.order', string='Purchase Order', readonly=True)

    def init(self):
        tools.drop_view_if_exists(self.env.cr, 'purchase_bill_union')
        self.env.cr.execute("""
            CREATE OR REPLACE VIEW purchase_bill_union AS (
                SELECT
                    id, name, ref as reference, partner_id, date, amount_untaxed as amount, currency_id, company_id,
                    id as vendor_bill_id, NULL as purchase_order_id
                FROM account_move
                WHERE
                    type='in_invoice' and state = 'posted'
            UNION
                SELECT
                    -id, name, partner_ref as reference, partner_id, date_order::date as date, amount_untaxed as amount, currency_id, company_id,
                    NULL as vendor_bill_id, id as purchase_order_id
                FROM purchase_order
                WHERE
                    state in ('purchase', 'done') AND
                    invoice_status in ('to invoice', 'no')
            )""")

    def name_get(self):
        result = []
        for doc in self:
            name = doc.name or ''
            if doc.reference:
                name += ' - ' + doc.reference
            amount = doc.amount
            if doc.purchase_order_id and doc.purchase_order_id.invoice_status == 'no':
                amount = 0.0
            name += ': ' + formatLang(self.env, amount, monetary=True, currency_obj=doc.currency_id)
            result.append((doc.id, name))
        return result

    @api.model
    def _name_search(self, name, args=None, operator='ilike', limit=100, name_get_uid=None):
        args = args or []
        domain = []
        if name:
            domain = ['|', ('name', operator, name), ('reference', operator, name)]
        purchase_bills_union_ids = self._search(expression.AND([domain, args]), limit=limit, access_rights_uid=name_get_uid)
        return self.browse(purchase_bills_union_ids).name_get()

```

## File: report\purchase_bill_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

   <record id="view_purchase_bill_union_filter" model="ir.ui.view">
        <field name="name">purchase.bill.union.select</field>
        <field name="model">purchase.bill.union</field>
        <field name="arch" type="xml">
            <search string="Search Reference Document">
                <field name="name" string="Reference" filter_domain="['|', ('name','ilike',self), ('reference','=like',str(self)+'%')]"/>
                <field name="amount"/>
                <separator/>
                <field name="partner_id" operator="child_of"/>
                <separator/>
                <filter name="purchase_orders" string="Purchase Orders" domain="[('purchase_order_id', '!=', False)]"/>
                <filter name="vendor_bills" string="Vendor Bills" domain="[('vendor_bill_id', '!=', False)]"/>
            </search>
        </field>
    </record>

    <record id="view_purchase_bill_union_tree" model="ir.ui.view">
        <field name="name">purchase.bill.union.tree</field>
        <field name="model">purchase.bill.union</field>
        <field name="arch" type="xml">
            <tree string="Reference Document">
                <field name="name"/>
                <field name="reference"/>
                <field name="partner_id"/>
                <field name="date"/>
                <field name="amount"/>
                <field name="currency_id" invisible="1"/>
                <field name="company_id" groups="base.group_multi_company" options="{'no_create': True}"/>
            </tree>
        </field>
    </record>

</odoo>

```

## File: report\purchase_order_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<template id="report_purchaseorder_document">
    <t t-call="web.external_layout">
        <t t-set="o" t-value="o.with_context(lang=o.partner_id.lang)"/>
        <t t-set="address">
            <div t-field="o.partner_id"
            t-options='{"widget": "contact", "fields": ["address", "name", "phone"], "no_marker": True, "phone_icons": True}'/>
            <p t-if="o.partner_id.vat"><t t-esc="o.company_id.country_id.vat_label or 'Tax ID'"/>: <span t-field="o.partner_id.vat"/></p>
        </t>
        <t t-if="o.dest_address_id">
            <t t-set="information_block">
                <strong>Shipping address:</strong>
                <div t-if="o.dest_address_id">
                    <div t-field="o.dest_address_id"
                        t-options='{"widget": "contact", "fields": ["address", "name", "phone"], "no_marker": True, "phone_icons": True}' name="purchase_shipping_address"/>
                </div>

            </t>
        </t>
        <div class="page">
            <div class="oe_structure"/>

            <h2 t-if="o.state == 'draft'">Request for Quotation #<span t-field="o.name"/></h2>
            <h2 t-if="o.state in ['sent', 'to approve']">Purchase Order #<span t-field="o.name"/></h2>
            <h2 t-if="o.state in ['purchase', 'done']">Purchase Order #<span t-field="o.name"/></h2>
            <h2 t-if="o.state == 'cancel'">Cancelled Purchase Order #<span t-field="o.name"/></h2>

            <div id="informations" class="row mt32 mb32">
                <div t-if="o.user_id" class="col-3 bm-2">
                    <strong>Purchase Representative:</strong>
                    <p t-field="o.user_id" class="m-0"/>
                </div>
                <div t-if="o.partner_ref" class="col-3 bm-2">
                    <strong>Your Order Reference:</strong>
                    <p t-field="o.partner_ref" class="m-0"/>
                </div>
                <div t-if="o.state in ['purchase','done'] and o.date_approve" class="col-3 bm-2">
                    <strong>Order Date:</strong>
                    <p t-field="o.date_approve" class="m-0"/>
                </div>
                <div t-elif="o.date_order" class="col-3 bm-2">
                    <strong >Order Deadline:</strong>
                    <p t-field="o.date_order" class="m-0"/>
                </div>
            </div>

            <table class="table table-sm o_main_table">
                <thead>
                    <tr>
                        <th name="th_description"><strong>Description</strong></th>
                        <th name="th_taxes"><strong>Taxes</strong></th>
                        <th name="th_date_req" class="text-center"><strong>Date Req.</strong></th>
                        <th name="th_quantity" class="text-right"><strong>Qty</strong></th>
                        <th name="th_price_unit" class="text-right"><strong>Unit Price</strong></th>
                        <th name="th_amount" class="text-right"><strong>Amount</strong></th>
                    </tr>
                </thead>
                <tbody>
                    <t t-set="current_subtotal" t-value="0"/>
                    <t t-foreach="o.order_line" t-as="line">
                        <t t-set="current_subtotal" t-value="current_subtotal + line.price_subtotal" groups="account.group_show_line_subtotals_tax_excluded"/>
                        <t t-set="current_subtotal" t-value="current_subtotal + line.price_total" groups="account.group_show_line_subtotals_tax_included"/>

                        <tr t-att-class="'bg-200 font-weight-bold o_line_section' if line.display_type == 'line_section' else 'font-italic o_line_note' if line.display_type == 'line_note' else ''">
                            <t t-if="not line.display_type">
                                <td id="product">
                                    <span t-field="line.name"/>
                                </td>
                                <td name="td_taxes">
                                    <span t-esc="', '.join(map(lambda x: x.name, line.taxes_id))"/>
                                </td>
                                <td class="text-center">
                                    <span t-field="line.date_planned"/>
                                </td>
                                <td class="text-right">
                                    <span t-field="line.product_qty"/>
                                    <span t-field="line.product_uom.name" groups="uom.group_uom"/>
                                </td>
                                <td class="text-right">
                                    <span t-field="line.price_unit"/>
                                </td>
                                <td class="text-right">
                                    <span t-field="line.price_subtotal"
                                        t-options='{"widget": "monetary", "display_currency": o.currency_id}'/>
                                </td>
                            </t>
                            <t t-if="line.display_type == 'line_section'">
                                <td colspan="99" id="section">
                                    <span t-field="line.name"/>
                                </td>
                                <t t-set="current_section" t-value="line"/>
                                <t t-set="current_subtotal" t-value="0"/>
                            </t>
                            <t t-if="line.display_type == 'line_note'">
                                <td colspan="99" id="note">
                                    <span t-field="line.name"/>
                                </td>
                            </t>
                        </tr>
                        <t t-if="current_section and (line_last or o.order_line[line_index+1].display_type == 'line_section')">
                            <tr class="is-subtotal text-right">
                                <td colspan="99" id="subtotal">
                                    <strong class="mr16">Subtotal</strong>
                                    <span
                                        t-esc="current_subtotal"
                                        t-options='{"widget": "monetary", "display_currency": o.currency_id}'
                                    />
                                </td>
                            </tr>
                        </t>
                    </t>
                </tbody>
            </table>

            <div id="total" class="row justify-content-end">
                <div class="col-4">
                    <table class="table table-sm">
                        <tr class="border-black">
                            <td name="td_subtotal_label"><strong>Subtotal</strong></td>
                            <td class="text-right">
                                <span t-field="o.amount_untaxed"
                                    t-options='{"widget": "monetary", "display_currency": o.currency_id}'/>
                            </td>
                        </tr>
                        <tr>
                            <td name="td_taxes_label">Taxes</td>
                            <td class="text-right">
                                <span t-field="o.amount_tax"
                                    t-options='{"widget": "monetary", "display_currency": o.currency_id}'/>
                            </td>
                        </tr>
                        <tr class="border-black o_total">
                            <td name="td_amount_total_label"><strong>Total</strong></td>
                            <td class="text-right">
                                <span t-field="o.amount_total"
                                    t-options='{"widget": "monetary", "display_currency": o.currency_id}'/>
                            </td>
                        </tr>
                    </table>
                </div>
            </div>

            <p t-field="o.notes"/>
            <div class="oe_structure"/>
        </div>
    </t>
</template>

<template id="report_purchaseorder">
    <t t-call="web.html_container">
        <t t-foreach="docs" t-as="o">
            <t t-call="purchase.report_purchaseorder_document" t-lang="o.partner_id.lang"/>
        </t>
    </t>
</template>
</odoo>

```

## File: report\purchase_quotation_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<template id="report_purchasequotation_document">
    <t t-call="web.external_layout">
        <t t-set="o" t-value="o.with_context(lang=o.partner_id.lang)"/>
        <t t-set="address">
            <div t-field="o.partner_id"
            t-options='{"widget": "contact", "fields": ["address", "name", "phone"], "no_marker": True, "phone_icons": True}'/>
            <p t-if="o.partner_id.vat"><t t-esc="o.company_id.country_id.vat_label or 'Tax ID'"/>: <span t-field="o.partner_id.vat"/></p>
        </t>
        <t t-if="o.dest_address_id">
            <t t-set="information_block">
                <strong>Shipping address:</strong>
                <div t-field="o.dest_address_id"
                    t-options='{"widget": "contact", "fields": ["address", "name", "phone"], "no_marker": True, "phone_icons": True}' name="purchase_shipping_address"/>
            </t>
        </t>
        <div class="page">
            <div class="oe_structure"/>

            <h2>Request for Quotation <span t-field="o.name"/></h2>

            <table class="table table-sm">
                <thead style="display: table-row-group">
                    <tr>
                        <th name="th_description"><strong>Description</strong></th>
                        <th name="th_expected_date" class="text-center"><strong>Expected Date</strong></th>
                        <th name="th_quantity" class="text-right"><strong>Qty</strong></th>
                    </tr>
                </thead>
                <tbody>
                    <t t-foreach="o.order_line" t-as="order_line">
                        <tr t-att-class="'bg-200 font-weight-bold o_line_section' if order_line.display_type == 'line_section' else 'font-italic o_line_note' if order_line.display_type == 'line_note' else ''">
                            <t t-if="not order_line.display_type">
                                <td id="product">
                                    <span t-field="order_line.name"/>
                                </td>
                                <td class="text-center">
                                    <span t-field="order_line.date_planned"/>
                                </td>
                                <td class="text-right">
                                    <span t-field="order_line.product_qty"/>
                                    <span t-field="order_line.product_uom" groups="uom.group_uom"/>
                                </td>
                            </t>
                            <t t-else="">
                                <td colspan="99" id="section">
                                    <span t-field="order_line.name"/>
                                </td>
                            </t>
                        </tr>
                    </t>
                </tbody>
            </table>

            <p t-field="o.notes"/>

            <div class="oe_structure"/>
        </div>
    </t>
</template>

<template id="report_purchasequotation">
    <t t-call="web.html_container">
        <t t-foreach="docs" t-as="o">
            <t t-call="purchase.report_purchasequotation_document" t-lang="o.partner_id.lang"/>
        </t>
    </t>
</template>
</odoo>

```

## File: report\purchase_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

#
# Please note that these reports are not multi-currency !!!
#

from odoo import api, fields, models, tools


class PurchaseReport(models.Model):
    _name = "purchase.report"
    _description = "Purchase Report"
    _auto = False
    _order = 'date_order desc, price_total desc'

    date_order = fields.Datetime('Order Date', readonly=True, help="Date on which this document has been created")
    state = fields.Selection([
        ('draft', 'Draft RFQ'),
        ('sent', 'RFQ Sent'),
        ('to approve', 'To Approve'),
        ('purchase', 'Purchase Order'),
        ('done', 'Done'),
        ('cancel', 'Cancelled')
    ], 'Order Status', readonly=True)
    product_id = fields.Many2one('product.product', 'Product', readonly=True)
    partner_id = fields.Many2one('res.partner', 'Vendor', readonly=True)
    date_approve = fields.Datetime('Confirmation Date', readonly=True)
    product_uom = fields.Many2one('uom.uom', 'Reference Unit of Measure', required=True)
    company_id = fields.Many2one('res.company', 'Company', readonly=True)
    currency_id = fields.Many2one('res.currency', 'Currency', readonly=True)
    user_id = fields.Many2one('res.users', 'Purchase Representative', readonly=True)
    delay = fields.Float('Days to Confirm', digits=(16, 2), readonly=True, group_operator='avg')
    delay_pass = fields.Float('Days to Receive', digits=(16, 2), readonly=True, group_operator='avg')
    price_total = fields.Float('Total', readonly=True)
    price_average = fields.Float('Average Cost', readonly=True, group_operator="avg")
    nbr_lines = fields.Integer('# of Lines', readonly=True)
    category_id = fields.Many2one('product.category', 'Product Category', readonly=True)
    product_tmpl_id = fields.Many2one('product.template', 'Product Template', readonly=True)
    country_id = fields.Many2one('res.country', 'Partner Country', readonly=True)
    fiscal_position_id = fields.Many2one('account.fiscal.position', string='Fiscal Position', readonly=True)
    account_analytic_id = fields.Many2one('account.analytic.account', 'Analytic Account', readonly=True)
    commercial_partner_id = fields.Many2one('res.partner', 'Commercial Entity', readonly=True)
    weight = fields.Float('Gross Weight', readonly=True)
    volume = fields.Float('Volume', readonly=True)
    order_id = fields.Many2one('purchase.order', 'Order', readonly=True)
    untaxed_total = fields.Float('Untaxed Total', readonly=True)
    qty_ordered = fields.Float('Qty Ordered', readonly=True)
    qty_received = fields.Float('Qty Received', readonly=True)
    qty_billed = fields.Float('Qty Billed', readonly=True)
    qty_to_be_billed = fields.Float('Qty to be Billed', readonly=True)

    def init(self):
        # self._table = sale_report
        tools.drop_view_if_exists(self.env.cr, self._table)
        self.env.cr.execute("""CREATE or REPLACE VIEW %s as (
            %s
            FROM ( %s )
            %s
            )""" % (self._table, self._select(), self._from(), self._group_by()))

    def _select(self):
        select_str = """
            WITH currency_rate as (%s)
                SELECT
                    po.id as order_id,
                    min(l.id) as id,
                    po.date_order as date_order,
                    po.state,
                    po.date_approve,
                    po.dest_address_id,
                    po.partner_id as partner_id,
                    po.user_id as user_id,
                    po.company_id as company_id,
                    po.fiscal_position_id as fiscal_position_id,
                    l.product_id,
                    p.product_tmpl_id,
                    t.categ_id as category_id,
                    po.currency_id,
                    t.uom_id as product_uom,
                    extract(epoch from age(po.date_approve,po.date_order))/(24*60*60)::decimal(16,2) as delay,
                    extract(epoch from age(l.date_planned,po.date_order))/(24*60*60)::decimal(16,2) as delay_pass,
                    count(*) as nbr_lines,
                    sum(l.price_total / COALESCE(po.currency_rate, 1.0))::decimal(16,2) as price_total,
                    (sum(l.product_qty * l.price_unit / COALESCE(po.currency_rate, 1.0))/NULLIF(sum(l.product_qty/line_uom.factor*product_uom.factor),0.0))::decimal(16,2) as price_average,
                    partner.country_id as country_id,
                    partner.commercial_partner_id as commercial_partner_id,
                    analytic_account.id as account_analytic_id,
                    sum(p.weight * l.product_qty/line_uom.factor*product_uom.factor) as weight,
                    sum(p.volume * l.product_qty/line_uom.factor*product_uom.factor) as volume,
                    sum(l.price_subtotal / COALESCE(po.currency_rate, 1.0))::decimal(16,2) as untaxed_total,
                    sum(l.product_qty / line_uom.factor * product_uom.factor) as qty_ordered,
                    sum(l.qty_received / line_uom.factor * product_uom.factor) as qty_received,
                    sum(l.qty_invoiced / line_uom.factor * product_uom.factor) as qty_billed,
                    case when t.purchase_method = 'purchase' 
                         then sum(l.product_qty / line_uom.factor * product_uom.factor) - sum(l.qty_invoiced / line_uom.factor * product_uom.factor)
                         else sum(l.qty_received / line_uom.factor * product_uom.factor) - sum(l.qty_invoiced / line_uom.factor * product_uom.factor)
                    end as qty_to_be_billed
        """ % self.env['res.currency']._select_companies_rates()
        return select_str

    def _from(self):
        from_str = """
            purchase_order_line l
                join purchase_order po on (l.order_id=po.id)
                join res_partner partner on po.partner_id = partner.id
                    left join product_product p on (l.product_id=p.id)
                        left join product_template t on (p.product_tmpl_id=t.id)
                left join uom_uom line_uom on (line_uom.id=l.product_uom)
                left join uom_uom product_uom on (product_uom.id=t.uom_id)
                left join account_analytic_account analytic_account on (l.account_analytic_id = analytic_account.id)
                left join currency_rate cr on (cr.currency_id = po.currency_id and
                    cr.company_id = po.company_id and
                    cr.date_start <= coalesce(po.date_order, now()) and
                    (cr.date_end is null or cr.date_end > coalesce(po.date_order, now())))
        """
        return from_str

    def _group_by(self):
        group_by_str = """
            GROUP BY
                po.company_id,
                po.user_id,
                po.partner_id,
                line_uom.factor,
                po.currency_id,
                l.price_unit,
                po.date_approve,
                l.date_planned,
                l.product_uom,
                po.dest_address_id,
                po.fiscal_position_id,
                l.product_id,
                p.product_tmpl_id,
                t.categ_id,
                po.date_order,
                po.state,
                line_uom.uom_type,
                line_uom.category_id,
                t.uom_id,
                t.purchase_method,
                line_uom.id,
                product_uom.factor,
                partner.country_id,
                partner.commercial_partner_id,
                analytic_account.id,
                po.id
        """
        return group_by_str

```

## File: report\purchase_reports.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <report 
            string="Purchase Order" 
            id="action_report_purchase_order" 
            model="purchase.order" 
            report_type="qweb-pdf"
            name="purchase.report_purchaseorder" 
            file="purchase.report_purchaseorder"
            print_report_name="
                (object.state in ('draft', 'sent') and 'Request for Quotation - %s' % (object.name) or
                'Purchase Order - %s' % (object.name))"
        />

        <report 
            string="Request for Quotation" 
            id="report_purchase_quotation" 
            model="purchase.order" 
            report_type="qweb-pdf"
            name="purchase.report_purchasequotation" 
            file="purchase.report_purchasequotation"
            print_report_name="'Request for Quotation - %s' % (object.name)"
        />
    </data>
</odoo>

```

## File: report\purchase_report_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record model="ir.ui.view" id="view_purchase_order_pivot">
            <field name="name">product.month.pivot</field>
            <field name="model">purchase.report</field>
            <field name="arch" type="xml">
                <pivot string="Purchase Analysis" disable_linking="True" display_quantity="true">
                    <field name="category_id" type="row"/>
                    <field name="order_id" type="measure"/>
                    <field name="untaxed_total" type="measure"/>
                    <field name="price_total" type="measure"/>
                </pivot>
            </field>
        </record>
        <record model="ir.ui.view" id="view_purchase_order_graph">
            <field name="name">product.month.graph</field>
            <field name="model">purchase.report</field>
            <field name="arch" type="xml">
                <graph string="Purchase Orders Statistics" type="line">
                    <field name="date_approve" interval="day" type="col"/>
                    <field name="untaxed_total" type="measure"/>
                </graph>
            </field>
        </record>

        <!-- Custom reports (aka filters) -->
        <record id="filter_purchase_order_monthly_purchases" model="ir.filters">
            <field name="name">Monthly Purchases</field>
            <field name="model_id">purchase.report</field>
            <field name="domain">[('state','!=','cancel')]</field>
            <field name="user_id" eval="False"/>
        </record>
        <record id="filter_purchase_order_price_per_supplier" model="ir.filters">
            <field name="name">Price Per Vendor</field>
            <field name="model_id">purchase.report</field>
            <field name="domain">[('state','!=','draft'),('state','!=','cancel')]</field>
            <field name="user_id" eval="False"/>
            <field name="context">{'group_by': ['partner_id'], 'col_group_by': ['product_id'], 'measures': ['price_average']}</field>
        </record>
        <record id="filter_purchase_order_average_delivery_time" model="ir.filters">
            <field name="name">Average Delivery Time</field>
            <field name="model_id">purchase.report</field>
            <field name="domain">[('state','!=','draft'),('state','!=','cancel')]</field>
            <field name="user_id" eval="False"/>
            <field name="context">{'group_by': ['partner_id'], 'measures': ['delay_pass']}</field>
        </record>


        <record id="view_purchase_order_search" model="ir.ui.view">
        <field name="name">report.purchase.order.search</field>
        <field name="model">purchase.report</field>
        <field name="arch" type="xml">
            <search string="Purchase Orders">
                <filter string="Requests for Quotation" name="quotes" domain="[('state','in',('draft','sent'))]"/>
                <filter string="Purchase Orders" name="orders" domain="[('state','!=','draft'), ('state','!=','sent'), ('state','!=','cancel')]"/>
                <field name="partner_id"/>
                <field name="product_id"/>
                <group expand="0" string="Extended Filters">
                    <field name="user_id"/>
                    <field name="company_id" groups="base.group_multi_company"/>
                    <field name="date_order"/>
                    <field name="date_approve"/>
                    <field name="category_id" filter_domain="[('category_id', 'child_of', self)]"/>
                </group>
                <group expand="1" string="Group By">
                    <filter string="Vendor" name="group_partner_id" context="{'group_by':'partner_id'}"/>
                    <filter string="Vendor Country" name="country_id" context="{'group_by':'country_id'}"/>
                    <filter string="Purchase Representative" name="user_id" context="{'group_by':'user_id'}"/>
                    <filter string="Product" name="group_product_id" context="{'group_by':'product_id'}"/>
                    <filter string="Product Category" name="group_category_id" context="{'group_by':'category_id'}"/>
                    <filter string="Status" name="status" context="{'group_by':'state'}"/>
                    <filter string="Company" name="company" context="{'group_by':'company_id'}" groups="base.group_multi_company"/>
                    <separator/>
                    <filter string="Order Date" name="order_month" context="{'group_by': 'date_order:month'}"/>
                    <filter string="Confirmation Date" name="group_date_approve_month" context="{'group_by': 'date_approve:month'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="action_purchase_order_report_all" model="ir.actions.act_window">
        <field name="name">Purchase Analysis</field>
        <field name="res_model">purchase.report</field>
        <field name="view_mode">graph,pivot</field>
        <field name="view_id"></field>  <!-- force empty -->
        <field name="help">Purchase Analysis allows you to easily check and analyse your company purchase history and performance. From this menu you can track your negotiation performance, the delivery performance of your vendors, etc.</field>
        <field name="target">current</field>
    </record>

    <menuitem id="purchase_report" name="Reporting" parent="purchase.menu_purchase_root" sequence="99"/>

    <menuitem id="menu_report_purchase" name="Purchase"  action="action_purchase_order_report_all" parent="purchase_report" sequence="1" groups="purchase.group_purchase_manager"/>

</odoo>

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import purchase_report
from . import purchase_bill

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_purchase_order,purchase.order,model_purchase_order,group_purchase_user,1,1,1,1
access_purchase_order_manager,purchase.order,model_purchase_order,group_purchase_manager,1,1,1,1
access_purchase_order_invoicing_payments,purchase.order,model_purchase_order,account.group_account_invoice,1,1,0,0
access_purchase_order_portal,purchase.order.portal,purchase.model_purchase_order,base.group_portal,1,0,0,0
access_purchase_order_line,purchase.order.line user,model_purchase_order_line,group_purchase_user,1,1,1,1
access_purchase_order_line_manager,purchase.order.line,model_purchase_order_line,group_purchase_manager,1,1,1,1
access_purchase_order_line_invoicing_payments,purchase.order.line,model_purchase_order_line,account.group_account_invoice,1,1,0,0
access_purchase_order_line_portal,purchase.order.line.portal,purchase.model_purchase_order_line,base.group_portal,1,0,0,0
access_account_tax_purchase_user,account.tax,account.model_account_tax,group_purchase_user,1,0,0,0
access_account_tax_purchase_user_manager,account.tax,account.model_account_tax,group_purchase_manager,1,0,0,0
access_product_product_purchase_user,product.product.purchase.user,product.model_product_product,group_purchase_user,1,0,0,0
access_product_product_purchase_manager,product.product purchase_manager,product.model_product_product,purchase.group_purchase_manager,1,1,1,1
access_product_template_purchase_user,product.template purchase_user,product.model_product_template,group_purchase_user,1,0,0,0
access_account_move_purchase,account_move purchase,account.model_account_move,group_purchase_user,1,1,1,1
access_account_move_purchase_manager,account_move purchase manager,account.model_account_move,group_purchase_manager,1,0,0,0
access_account_fiscal_position_purchase_user,account.fiscal.position purchase,account.model_account_fiscal_position,group_purchase_user,1,0,0,0
access_res_partner_purchase_user,res.partner purchase,base.model_res_partner,group_purchase_user,1,0,0,0
access_account_journal,account.journal,account.model_account_journal,group_purchase_user,1,0,0,0
access_account_journal_manager,account.journal,account.model_account_journal,group_purchase_manager,1,0,0,0
access_account_move_line,account.move.line,account.model_account_move_line,group_purchase_user,1,1,1,0
access_account_analytic_line,account.analytic.line,account.model_account_analytic_line,group_purchase_user,1,0,0,0
access_account_type,account.acount.type.purchase.user,account.model_account_account_type,group_purchase_user,1,0,0,0
account_partial_reconcile,account.partial.reconcile.purchase.user,account.model_account_partial_reconcile,group_purchase_user,1,0,0,0
access_res_partner_purchase_manager,res.partner.purchase.manager,base.model_res_partner,group_purchase_manager,1,1,1,0
access_uom_category_purchase_manager,uom.category purchase_manager,uom.model_uom_category,purchase.group_purchase_manager,1,1,1,1
access_uom_uom_purchase_manager,uom.uom purchase_manager,uom.model_uom_uom,purchase.group_purchase_manager,1,1,1,1
access_product_category_purchase_manager,product.category purchase_manager,product.model_product_category,purchase.group_purchase_manager,1,1,1,1
access_product_template_purchase_manager,product.template purchase_manager,product.model_product_template,purchase.group_purchase_manager,1,1,1,1
access_product_packaging_purchase_manager,product.packaging purchase_manager,product.model_product_packaging,purchase.group_purchase_manager,1,1,1,1
access_product_supplierinfo_purchase_manager,product.supplierinfo purchase_manager,product.model_product_supplierinfo,purchase.group_purchase_manager,1,1,1,1
access_product_group_res_partner_purchase_manager,res_partner group_purchase_manager,base.model_res_partner,purchase.group_purchase_manager,1,1,1,0
access_product_pricelist_item_purchase_manager,product.pricelist.item purchase_manager,product.model_product_pricelist_item,purchase.group_purchase_manager,1,1,1,1
access_account_account_purchase_manager,account.account purchase manager,account.model_account_account,purchase.group_purchase_manager,1,0,0,0
access_account_journal_purchase_manager,account.journal purchase manager,account.model_account_journal,purchase.group_purchase_manager,1,0,0,0
access_purchase_bill_union,access_purchase_bill_union,model_purchase_bill_union,purchase.group_purchase_user,1,0,0,0
access_report_purchase_order,purchase.stock.report,model_purchase_report,purchase.group_purchase_manager,1,0,0,0
access_report_purchase_order_user,purchase.stock.report user,model_purchase_report,purchase.group_purchase_user,1,0,0,0

```

## File: security\purchase_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data noupdate="0">

    <record model="ir.module.category" id="base.module_category_operations_purchase">
        <field name="description">Helps you manage your purchase-related processes such as requests for quotations, supplier bills, etc...</field>
        <field name="sequence">8</field>
    </record>

    <record id="group_purchase_user" model="res.groups">
        <field name="name">User</field>
        <field name="implied_ids" eval="[(4, ref('base.group_user'))]"/>
        <field name="category_id" ref="base.module_category_operations_purchase"/>
    </record>

    <record id="group_purchase_manager" model="res.groups">
        <field name="name">Administrator</field>
        <field name="category_id" ref="base.module_category_operations_purchase"/>
        <field name="implied_ids" eval="[(4, ref('group_purchase_user'))]"/>
        <field name="users" eval="[(4, ref('base.user_root')), (4, ref('base.user_admin'))]"/>
    </record>

    <record id="group_warning_purchase" model="res.groups">
        <field name="name">A warning can be set on a product or a customer (Purchase)</field>
        <field name="category_id" ref="base.module_category_hidden"/>
    </record>

</data>
<data noupdate="1">
    <record id="base.default_user" model="res.users">
        <field name="groups_id" eval="[(4,ref('purchase.group_purchase_manager'))]"/>
    </record>

    <record model="ir.rule" id="purchase_order_comp_rule">
        <field name="name">Purchase Order multi-company</field>
        <field name="model_id" ref="model_purchase_order"/>
        <field name="global" eval="True"/>
        <field name="domain_force">['|',('company_id','=',False),('company_id', 'in', company_ids)]</field>
    </record>

    <record model="ir.rule" id="purchase_order_line_comp_rule">
        <field name="name">Purchase Order Line multi-company</field>
        <field name="model_id" ref="model_purchase_order_line"/>
        <field name="global" eval="True"/>
        <field name="domain_force">['|',('company_id','=',False),('company_id', 'in', company_ids)]</field>
    </record>

    <record id="portal_purchase_order_user_rule" model="ir.rule">
        <field name="name">Portal Purchase Orders</field>
        <field name="model_id" ref="purchase.model_purchase_order"/>
        <field name="domain_force">['|', ('message_partner_ids','child_of',[user.commercial_partner_id.id]),('partner_id', 'child_of', [user.commercial_partner_id.id])]</field>
        <field name="groups" eval="[(4, ref('base.group_portal'))]"/>
        <field name="perm_unlink" eval="1"/>
        <field name="perm_write" eval="1"/>
        <field name="perm_read" eval="1"/>
        <field name="perm_create" eval="0"/>
    </record>

    <record id="purchase_user_account_move_line_rule" model="ir.rule">
        <field name="name">Purchase User Account Move Line</field>
        <field name="model_id" ref="account.model_account_move_line"/>
        <field name="domain_force">[('move_id.type', 'in', ('in_invoice', 'in_refund', 'in_receipt'))]</field>
        <field name="groups" eval="[(4, ref('purchase.group_purchase_user'))]"/>
    </record>
    <record id="purchase_user_account_move_rule" model="ir.rule">
        <field name="name">Purchase User Account Move</field>
        <field name="model_id" ref="account.model_account_move"/>
        <field name="domain_force">[('type', 'in', ('in_invoice', 'in_refund', 'in_receipt'))]</field>
        <field name="groups" eval="[(4, ref('purchase.group_purchase_user'))]"/>
    </record>
    <record id="portal_purchase_order_line_rule" model="ir.rule">
        <field name="name">Portal Purhcase Orders Line</field>
        <field name="model_id" ref="purchase.model_purchase_order_line"/>
        <field name="domain_force">['|',('order_id.message_partner_ids','child_of',[user.commercial_partner_id.id]),('order_id.partner_id','child_of',[user.commercial_partner_id.id])]</field>
        <field name="groups" eval="[(4, ref('base.group_portal'))]"/>
    </record>
    <record model="ir.rule" id="purchase_bill_union_comp_rule">
        <field name="name">Purchases &amp; Bills Union multi-company</field>
        <field name="model_id" ref="model_purchase_bill_union"/>
        <field name="global" eval="True"/>
        <field name="domain_force">['|',('company_id','=',False),('company_id', 'in', company_ids)]</field>
    </record>

    <record id="purchase_order_report_comp_rule" model="ir.rule">
        <field name="name">Purchase Order Report multi-company</field>
        <field name="model_id" ref="model_purchase_report"/>
        <field name="global" eval="True"/>
        <field name="domain_force">['|',('company_id','=',False),('company_id', 'in', company_ids)]</field>
    </record>

</data>
</odoo>

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#94B6C8"/><stop offset="100%" stop-color="#6A9EBA"/></linearGradient><path id="d" d="M53.551 19.955H16.223c-2.07 0-3.742 1.64-3.742 3.663v26.864c0 2.022 1.673 3.663 3.742 3.663h37.328c2.07 0 3.742-1.64 3.742-3.663V23.618c0-2.023-1.672-3.663-3.742-3.663zm-36.86 3.663h36.393c.257 0 .467.206.467.458v3.205H16.223v-3.205c0-.252.21-.458.467-.458zm36.393 26.864H16.69a.464.464 0 0 1-.467-.458V37.05h37.328v12.974c0 .252-.21.458-.467.458zM27.42 42.85v3.053c0 .503-.42.916-.934.916h-5.601a.928.928 0 0 1-.934-.916V42.85c0-.504.42-.916.934-.916h5.601c.514 0 .934.412.934.916zm14.937 0v3.053c0 .503-.42.916-.934.916h-10.58a.928.928 0 0 1-.934-.916V42.85c0-.504.42-.916.934-.916h10.58c.514 0 .934.412.934.916z"/><path id="e" d="M53.551 17.955H16.223c-2.07 0-3.742 1.64-3.742 3.663v26.864c0 2.022 1.673 3.663 3.742 3.663h37.328c2.07 0 3.742-1.64 3.742-3.663V21.618c0-2.023-1.672-3.663-3.742-3.663zm-36.86 3.663h36.393c.257 0 .467.206.467.458v3.205H16.223v-3.205c0-.252.21-.458.467-.458zm36.393 26.864H16.69a.464.464 0 0 1-.467-.458V35.05h37.328v12.974c0 .252-.21.458-.467.458zM27.42 40.85v3.053c0 .503-.42.916-.934.916h-5.601a.928.928 0 0 1-.934-.916V40.85c0-.504.42-.916.934-.916h5.601c.514 0 .934.412.934.916zm14.937 0v3.053c0 .503-.42.916-.934.916h-10.58a.928.928 0 0 1-.934-.916V40.85c0-.504.42-.916.934-.916h10.58c.514 0 .934.412.934.916z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M43.95 70H4c-2 0-4-.149-4-4.163V38.095l13.403-18.86L56 20.04l1.006 29.996L43.95 70z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-opacity=".3" fill-rule="nonzero" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: views\account_move_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_move_form_inherit_purchase" model="ir.ui.view">
        <field name="name">account.move.inherit.purchase</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_move_form"/>
        <field name="groups_id" eval="[(4, ref('purchase.group_purchase_user'))]"/>
        <field name="arch" type="xml">
            <!-- Auto-complete could be done from either a bill either a purchase order -->
            <field name="invoice_vendor_bill_id" position="after">
                <field name="purchase_id" invisible="1"/>
                <field name="purchase_vendor_bill_id"
                       attrs="{'invisible': ['|', '|', ('state','not in',['draft']), ('state', '=', 'purchase'), ('type', '!=', 'in_invoice')]}"
                       class="oe_edit_only"
                       domain="partner_id and [('company_id', '=', company_id), ('partner_id','child_of', [partner_id])] or [('company_id', '=', company_id)]"
                       placeholder="Select a purchase order or an old bill"
                       context="{'show_total_amount': True}"
                       options="{'no_create': True, 'no_open': True}"/>
            </field>
            <field name="invoice_vendor_bill_id" position="attributes">
                <attribute name="invisible">1</attribute>
            </field>
            <!-- Add link to purchase_line_id to account.move.line -->
            <xpath expr="//field[@name='invoice_line_ids']/tree/field[@name='company_id']" position="after">
                <field name="purchase_line_id" invisible="1"/>
            </xpath>
            <xpath expr="//field[@name='line_ids']/tree/field[@name='company_id']" position="after">
                <field name="purchase_line_id" invisible="1"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\portal_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

  <template id="portal_my_home_menu_purchase" name="Portal layout : purchase menu entries" inherit_id="portal.portal_breadcrumbs" priority="25">
    <xpath expr="//ol[hasclass('o_portal_submenu')]" position="inside">
      <li t-if="page_name == 'purchase' or purchase_order" t-attf-class="breadcrumb-item #{'active ' if not purchase_order else ''}">
        <a t-if="purchase_order" t-attf-href="/my/purchase?{{ keep_query() }}">Purchase Orders</a>
        <t t-else="">Purchase Orders</t>
      </li>
      <li t-if="purchase_order" class="breadcrumb-item active">
        <t t-esc="purchase_order.name"/>
      </li>
    </xpath>
  </template>

  <template id="portal_my_home_purchase" name="Portal My Home : purchase entry" inherit_id="portal.portal_my_home" priority="25">
    <xpath expr="//div[hasclass('o_portal_docs')]" position="inside">
        <t t-if="purchase_count" t-call="portal.portal_docs_entry">
            <t t-set="title">Purchase Orders</t>
            <t t-set="url" t-value="'/my/purchase'"/>
            <t t-set="count" t-value="purchase_count"/>
        </t>
    </xpath>
  </template>

  <template id="portal_my_purchase_orders" name="Portal: My Purchase Orders">
    <t t-call="portal.portal_layout">
      <t t-call="portal.portal_searchbar">
        <t t-set="title">Purchase Orders</t>
      </t>
      <t t-if="orders" t-call="portal.portal_table">
        <thead>
          <tr class="active">
            <th>Purchase Orders #</th>
            <th>Order Date</th>
            <th></th>
            <th>Total</th>
          </tr>
        </thead>
        <tbody>
          <t t-foreach="orders" t-as="order">
            <tr>
              <td><a t-attf-href="/my/purchase/#{order.id}?#{keep_query()}"><t t-esc="order.name"/></a></td>
              <td><span t-field="order.date_order"/></td>
              <td>
                <t t-if="order.invoice_status == 'to invoice'">
                  <span class="badge badge-info"><i class="fa fa-fw fa-file-text"/> Waiting for Bill</span>
                </t>
                <t t-if="order.state == 'cancel'">
                  <span class="badge badge-secondary"><i class="fa fa-fw fa-remove"/> Cancelled</span>
                </t>
              </td>
              <td><span t-field="order.amount_total" t-options='{"widget": "monetary", "display_currency": order.currency_id}'/></td>
            </tr>
          </t>
        </tbody>
      </t>
    </t>
  </template>

  <template id="portal_my_purchase_order" name="Portal: My Purchase Order">
    <t t-call="portal.portal_layout">
      <t t-set="purchase_order" t-value="order"/>
      <div id="optional_placeholder"></div>
      <div class="container">
          <div class="card">
            <div class="card-header">
              <div class="row">
                <div class="col-lg-12">
                  <h4>
                    <t t-if="order.state in ['draft', 'sent']">
                      Request for Quotation
                    </t>
                    <t t-else="1">
                      Purchase Order
                    </t>
                    <span t-esc="order.name"/>
                  </h4>
                </div>
              </div>
            </div>
            <div class="card-body">
              <div class="mb8">
                  <strong>Date:</strong> <span t-field="order.date_order" t-options='{"widget": "date"}'/>
              </div>
              <div class="row">
                <div class="col-lg-6">
                  <strong>Product</strong>
                </div>
                <div class="col-lg-2 text-right">
                  <strong>Unit Price</strong>
                </div>
                <div class="col-lg-2 text-right">
                  <strong>Quantity</strong>
                </div>
                <div class="col-lg-2 text-right">
                  <strong>Subtotal</strong>
                </div>
              </div>
              <t t-set="current_subtotal" t-value="0"/>
              <t t-foreach="order.order_line" t-as="ol">
                <t t-set="current_subtotal" t-value="current_subtotal + ol.price_subtotal" groups="account.group_show_line_subtotals_tax_excluded"/>
                <t t-set="current_subtotal" t-value="current_subtotal + ol.price_total" groups="account.group_show_line_subtotals_tax_included"/>
                <div t-if="not ol.display_type" class="row purchases_vertical_align">
                  <div class="col-lg-1 text-center">
                      <img t-att-src="image_data_uri(resize_to_48(ol.product_id.image_128))" alt="Product"/>
                  </div>
                  <div id='product_name' class="col-lg-5">
                    <span t-esc="ol.name"/>
                  </div>
                  <div class="col-lg-2 text-right">
                    <span t-field="ol.price_unit" t-options='{"widget": "monetary", "display_currency": order.currency_id}'/>
                  </div>
                  <div class="col-lg-2 text-right">
                      <span t-esc="ol.product_qty"/>
                  </div>
                  <div class="col-lg-2 text-right">
                    <span t-field="ol.price_subtotal" t-options='{"widget": "monetary", "display_currency": order.currency_id}'/>
                  </div>
                </div>
                <t t-if="ol.display_type == 'line_section'">
                    <div class="col-lg-12 bg-200">
                        <strong t-esc="ol.name"/>
                    </div>
                    <t t-set="current_section" t-value="ol"/>
                    <t t-set="current_subtotal" t-value="0"/>
                </t>
                <t t-elif="ol.display_type == 'line_note'">
                    <div class="col-lg-12 font-italic">
                        <span t-esc="ol.name"/>
                    </div>
                </t>
                <t t-if="current_section and (ol_last or order.order_line[ol_index+1].display_type == 'line_section')">
                  <div class="row">
                    <div class="col-lg-10 text-right">Subtotal</div>
                    <div class="col-lg-2 text-right">
                      <span
                            t-esc="current_subtotal"
                            t-options='{"widget": "monetary", "display_currency": order.currency_id}'
                          />
                    </div>
                  </div>
                </t>
              </t>

              <hr/>

              <div class="row">
                <div class="col-lg-12 text-right">
                  <div class="row">
                    <div class="col-lg-10 text-right">
                      Untaxed Amount:
                    </div>
                    <div class="col-lg-2 text-right">
                      <span t-field="order.amount_untaxed" t-options='{"widget": "monetary", "display_currency": order.currency_id}'/>
                    </div>
                  </div>
                  <div class="row">
                    <div class="col-lg-10 text-right">
                      Taxes:
                    </div>
                    <div class="col-lg-2 text-right">
                      <span t-field="order.amount_tax" t-options='{"widget": "monetary", "display_currency": order.currency_id}'/>
                    </div>
                  </div>
                  <div class="row">
                    <div class="col-lg-10 text-right">
                      <strong>Total:</strong>
                    </div>
                    <div class="col-lg-2 text-right">
                      <strong><span t-field="order.amount_total" t-options='{"widget": "monetary", "display_currency": order.currency_id}'/></strong>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
      </div>
      <div class="oe_structure mb32"/>
    </t>
  </template>

</odoo>

```

## File: views\product_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

        <!-- Product Suppliers-->

        <record id="view_product_supplier_inherit" model="ir.ui.view">
            <field name="name">product.template.supplier.form.inherit</field>
            <field name="model">product.template</field>
            <field name="inherit_id" ref="product.product_template_form_view"/>
            <field name="arch" type="xml">
                <xpath expr="//page[@name='purchase']" position="attributes">
                    <attribute name="invisible">0</attribute>
                </xpath>
                <group name="purchase" position="before">
                    <field name="seller_ids" context="{'default_product_tmpl_id':context.get('product_tmpl_id',active_id), 'product_template_invisible_variant': True}" nolabel="1" attrs="{'invisible': [('product_variant_count','&gt;',1)]}"/>
                    <field name="variant_seller_ids" context="{'default_product_tmpl_id': context.get('product_tmpl_id', active_id)}" nolabel="1" attrs="{'invisible': [('product_variant_count','&lt;=',1)]}"/>
                </group>
                <group name="bill" position="attributes">
                    <attribute name="groups">purchase.group_purchase_manager</attribute>
                </group>
                <group name="bill" position="inside">
                    <field name="purchase_method" widget="radio"/>
                </group>
                <page name="purchase" position="inside">
                    <group string="Purchase Description">
                       <field name="description_purchase" nolabel="1"
                            placeholder="This note is added to purchase orders."/>
                    </group>
                    <group string="Warning when Purchasing this Product" groups="purchase.group_warning_purchase">
                        <field name="purchase_line_warn" nolabel="1"/>
                        <field name="purchase_line_warn_msg" colspan="3" nolabel="1"
                                attrs="{'required':[('purchase_line_warn','!=','no-message')],'readonly':[('purchase_line_warn','=','no-message')], 'invisible':[('purchase_line_warn','=','no-message')]}"/>
                    </group>
                </page>
            </field>
        </record>

        <record id="view_category_property_form" model="ir.ui.view">
            <field name="name">product.category.property.form.inherit.stock</field>
            <field name="model">product.category</field>
            <field name="inherit_id" ref="account.view_category_property_form"/>
            <field name="arch" type="xml">
                <field name="property_account_income_categ_id" position="before">
                    <field name="property_account_creditor_price_difference_categ" domain="[('deprecated','=',False)]" groups="account.group_account_user"/>
                </field>
            </field>
        </record>

        <record id="view_product_account_purchase_ok_form" model="ir.ui.view">
            <field name="name">product.template.account.purchase.ok.form.inherit</field>
            <field name="model">product.template</field>
            <field name="inherit_id" ref="account.product_template_form_view"/>
            <field name="arch" type="xml">
                <field name="property_account_expense_id" position="attributes">
                    <attribute name="domain">[('deprecated','=',False)]</attribute>
                    <attribute name="attrs">{'readonly': [('purchase_ok', '=', 0)]}</attribute>
                </field>
                <field name='supplier_taxes_id' position="replace" >
                     <field name="supplier_taxes_id" colspan="2" widget="many2many_tags" attrs="{'readonly':[('purchase_ok','=',0)]}" context="{'default_type_tax_use':'purchase'}"/>
                </field>
            </field>
        </record>

        <record id="view_product_template_purchase_buttons_from" model="ir.ui.view">
            <field name="name">product.template.purchase.button.inherit</field>
            <field name="model">product.template</field>
            <field name="inherit_id" ref="product.product_template_only_form_view"/>
            <field name="groups_id" eval="[(4, ref('purchase.group_purchase_user'))]"/>
            <field name="arch" type="xml">
                <div name="button_box" position="inside">
                    <button class="oe_stat_button" name="action_view_po"
                        type="object" icon="fa-shopping-cart" attrs="{'invisible': [('purchase_ok', '=', False)]}" help="Purchased in the last 365 days">
                        <div class="o_field_widget o_stat_info">
                            <span class="o_stat_value">
                                <field name="purchased_product_qty" widget="statinfo" nolabel="1" class="mr4"/>
                                <field name="uom_name"/>
                            </span>
                            <span class="o_stat_text">Purchased</span>
                        </div>
                    </button>
                </div>
            </field>
        </record>

        <record id="product_template_form_view" model="ir.ui.view">
            <field name="name">product.normal.form.inherit.stock</field>
            <field name="model">product.template</field>
            <field name="inherit_id" ref="account.product_template_form_view"/>
            <field name="arch" type="xml">
                <field name="property_account_expense_id" position="after">
                    <field name="property_account_creditor_price_difference" domain="[('deprecated','=',False)]" attrs="{'readonly':[('purchase_ok', '=', 0)]}" groups="account.group_account_user"/>
                </field>
            </field>
        </record>

        <record id="product_normal_form_view_inherit_purchase" model="ir.ui.view">
            <field name="name">product.product.purchase.order</field>
            <field name="model">product.product</field>
            <field name="inherit_id" ref="product.product_normal_form_view"/>
            <field name="groups_id" eval="[(4, ref('purchase.group_purchase_user'))]"/>
            <field name="arch" type="xml">
                <div name="button_box" position="inside">
                    <button class="oe_stat_button" name="action_view_po"
                        type="object" icon="fa-shopping-cart" attrs="{'invisible': [('purchase_ok', '=', False)]}" help="Purchased in the last 365 days">
                        <div class="o_field_widget o_stat_info">
                            <span class="o_stat_value">
                                <field name="purchased_product_qty" widget="statinfo" nolabel="1" class="mr4"/>
                                <field name="uom_name"/>
                            </span>
                            <span class="o_stat_text">Purchased</span>
                        </div>
                    </button>
                </div>
            </field>
        </record>

</odoo>

```

## File: views\purchase_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="track_po_line_template">
        <div>
            <strong>The ordered quantity has been updated.</strong>
            <ul>
                <li><t t-esc="line.product_id.display_name"/>:</li>
                Ordered Quantity: <t t-esc="line.product_qty" /> -&gt; <t t-esc="float(product_qty)"/><br/>
                <t t-if='line.order_id.product_id.type in ("consu", "product")'>
                    Received Quantity: <t t-esc="line.qty_received" /><br/>
                </t>
                Billed Quantity: <t t-esc="line.qty_invoiced"/>
            </ul>
        </div>
    </template>

</odoo>

```

## File: views\purchase_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

        <!-- Top menu item -->
        <menuitem name="Purchase"
            id="menu_purchase_root"
            groups="group_purchase_manager,group_purchase_user"
            web_icon="purchase,static/description/icon.png"
            sequence="25"/>

        <menuitem id="menu_procurement_management" name="Orders"
            parent="menu_purchase_root" sequence="1" />

        <!--Supplier menu-->
        <menuitem id="menu_procurement_management_supplier_name" name="Vendors"
            parent="menu_procurement_management"
            action="account.res_partner_action_supplier" sequence="15"/>

        <menuitem id="menu_purchase_config" name="Configuration" parent="menu_purchase_root" sequence="100" groups="group_purchase_manager"/>

        <menuitem
           action="product.product_supplierinfo_type_action" id="menu_product_pricelist_action2_purchase"
           parent="menu_purchase_config" sequence="1"/>

        <menuitem
            id="menu_product_in_config_purchase" name="Products"
            parent="menu_purchase_config" sequence="30" groups="base.group_no_one"/>

        <menuitem
            action="product.product_category_action_form" id="menu_product_category_config_purchase"
            parent="purchase.menu_product_in_config_purchase" sequence="1" />

        <menuitem
             action="uom.product_uom_categ_form_action" id="menu_purchase_uom_categ_form_action"
             parent="purchase.menu_product_in_config_purchase" sequence="10" />

        <menuitem
              action="uom.product_uom_form_action" id="menu_purchase_uom_form_action"
              parent="purchase.menu_product_in_config_purchase" sequence="5"/>

    <!-- Products Control Menu -->
    <menuitem id="menu_purchase_products" name="Products" parent="purchase.menu_purchase_root" sequence="5"/>

    <record id="product_normal_action_puchased" model="ir.actions.act_window">
        <field name="name">Products</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">product.template</field>
        <field name="view_mode">kanban,tree,form,activity</field>
        <field name="context">{"search_default_filter_to_purchase":1, "purchase_product_template": 1}</field>
        <field name="search_view_id" ref="product.product_template_search_view"/>
        <field name="view_id" eval="False"/> <!-- Force empty -->
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Create a new product
          </p><p>
            You must define a product for everything you sell or purchase,
            whether it's a storable product, a consumable or a service.
          </p>
        </field>
    </record>

      <!-- Product menu-->
      <menuitem name="Products" id="menu_procurement_partner_contact_form" action="product_normal_action_puchased"
          parent="menu_purchase_products" sequence="20"/>

        <record id="product_product_action" model="ir.actions.act_window">
            <field name="name">Product Variants</field>
            <field name="res_model">product.product</field>
            <field name="view_mode">tree,kanban,form,activity</field>
            <field name="search_view_id" ref="product.product_search_form_view"/>
            <field name="context">{"search_default_filter_to_purchase": 1}</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Create a new product variant
              </p><p>
                You must define a product for everything you sell or purchase,
                whether it's a storable product, a consumable or a service.
              </p>
            </field>
        </record>

        <menuitem id="product_product_menu" name="Product Variants" action="product_product_action"
            parent="menu_purchase_products" sequence="21" groups="product.group_product_variant"/>

        <record model="ir.ui.view" id="purchase_order_calendar">
            <field name="name">purchase.order.calendar</field>
            <field name="model">purchase.order</field>
            <field name="priority" eval="2"/>
            <field name="arch" type="xml">
                <calendar string="Calendar View" date_start="date_planned" color="partner_id" hide_time="true" event_limit="5">
                    <field name="currency_id" invisible="1"/>
                    <field name="name"/>
                    <field name="partner_ref"/>
                    <field name="amount_total" widget="monetary"/>
                    <field name="partner_id"/>
                </calendar>
            </field>
        </record>
        <record model="ir.ui.view" id="purchase_order_pivot">
            <field name="name">purchase.order.pivot</field>
            <field name="model">purchase.order</field>
            <field name="arch" type="xml">
                <pivot string="Purchase Order" display_quantity="True">
                    <field name="partner_id" type="row"/>
                    <field name="amount_total" type="measure"/>
                </pivot>
            </field>
        </record>
        <record model="ir.ui.view" id="purchase_order_graph">
            <field name="name">purchase.order.graph</field>
            <field name="model">purchase.order</field>
            <field name="arch" type="xml">
                <graph string="Purchase Order">
                    <field name="partner_id"/>
                    <field name="amount_total" type="measure"/>
                </graph>
            </field>
        </record>

        <record id="purchase_order_form" model="ir.ui.view">
            <field name="name">purchase.order.form</field>
            <field name="model">purchase.order</field>
            <field name="arch" type="xml">
                <form string="Purchase Order" class="o_purchase_order">
                <header>
                    <button name="action_rfq_send" states="draft" string="Send by Email" type="object" context="{'send_rfq':True}" class="oe_highlight"/>
                    <button name="print_quotation" string="Print RFQ" type="object" states="draft" class="oe_highlight" groups="base.group_user"/>
                    <button name="button_confirm" type="object" states="sent" string="Confirm Order" class="oe_highlight" id="bid_confirm"/>
                    <button name="button_approve" type="object" states='to approve' string="Approve Order" class="oe_highlight" groups="purchase.group_purchase_manager"/>
                    <button name="action_view_invoice" string="Create Bill" type="object" class="oe_highlight" context="{'create_bill':True}" attrs="{'invisible': ['|', ('state', 'not in', ('purchase', 'done')), ('invoice_status', 'in', ('no', 'invoiced'))]}"/>
                    <button name="action_rfq_send" states="sent" string="Re-Send by Email" type="object" context="{'send_rfq':True}"/>
                    <button name="print_quotation" string="Print RFQ" type="object" states="sent" groups="base.group_user"/>
                    <button name="button_confirm" type="object" states="draft" string="Confirm Order" id="draft_confirm"/>
                    <button name="action_rfq_send" states="purchase" string="Send PO by Email" type="object" context="{'send_rfq':False}"/>
                    <button name="action_view_invoice" string="Create Bill" type="object" context="{'create_bill':True}" attrs="{'invisible': ['|', '|', ('state', 'not in', ('purchase', 'done')), ('invoice_status', 'not in', ('no', 'invoiced')), ('order_line', '=', [])]}"/>
                    <button name="button_draft" states="cancel" string="Set to Draft" type="object" />
                    <button name="button_cancel" states="draft,to approve,sent,purchase" string="Cancel" type="object" />
                    <button name="button_done" type="object" string="Lock" states="purchase"/>
                    <button name="button_unlock" type="object" string="Unlock" states="done" groups="purchase.group_purchase_manager"/>
                    <field name="state" widget="statusbar" statusbar_visible="draft,sent,purchase" readonly="1"/>
                </header>
                <sheet>
                    <div class="oe_button_box" name="button_box">
                        <button type="object"  name="action_view_invoice"
                            class="oe_stat_button"
                            icon="fa-pencil-square-o" attrs="{'invisible':['|', ('invoice_count', '=', 0), ('state', 'in', ('draft','sent','to approve'))]}">
                            <field name="invoice_count" widget="statinfo" string="Vendor Bills"/>
                            <field name='invoice_ids' invisible="1"/>
                        </button>
                    </div>
                    <div class="oe_title">
                        <span class="o_form_label" attrs="{'invisible': [('state','not in',('draft','sent'))]}">Request for Quotation </span>
                        <span class="o_form_label" attrs="{'invisible': [('state','in',('draft','sent'))]}">Purchase Order </span>
                        <h1>
                            <field name="name" readonly="1"/>
                        </h1>
                    </div>
                    <group>
                        <group>
                            <field name="partner_id" widget="res_partner_many2one" context="{'res_partner_search_mode': 'supplier', 'show_vat': True}"
                                placeholder="Name, TIN, Email, or Reference"
                            />
                            <field name="partner_ref"/>
                            <field name="currency_id" groups="base.group_multi_currency" force_save="1"/>
                        </group>
                        <group>
                            <field name="date_order" attrs="{'invisible': [('state','in',('purchase','done'))]}"/>
                            <field name="date_approve" attrs="{'invisible': [('state','not in',('purchase','done'))]}"/>
                            <field name="origin" attrs="{'invisible': [('origin','=',False)]}"/>
                            <field name="company_id" groups="base.group_multi_company" options="{'no_create': True}"/>
                        </group>
                    </group>
                    <notebook>
                        <page string="Products">
                            <field name="order_line"
                                widget="section_and_note_one2many"
                                mode="tree,kanban"
                                context="{'default_state': 'draft'}"
                                attrs="{'readonly': [('state', 'in', ('done', 'cancel'))]}">
                                <tree string="Purchase Order Lines" editable="bottom">
                                    <control>
                                        <create name="add_product_control" string="Add a product"/>
                                        <create name="add_section_control" string="Add a section" context="{'default_display_type': 'line_section'}"/>
                                        <create name="add_note_control" string="Add a note" context="{'default_display_type': 'line_note'}"/>
                                    </control>
                                    <field name="display_type" invisible="1"/>
                                    <field name="currency_id" invisible="1"/>
                                    <field name="state" invisible="1" readonly="1"/>
                                    <field name="product_type" invisible="1"/>
                                    <field name="product_uom_category_id" invisible="1"/>
                                    <field name="invoice_lines" invisible="1"/>
                                    <field name="sequence" widget="handle"/>
                                    <field
                                        name="product_id"
                                        attrs="{
                                            'readonly': [('state', 'in', ('purchase', 'to approve','done', 'cancel'))],
                                            'required': [('display_type', '=', False)],
                                        }"
                                        context="{'partner_id':parent.partner_id, 'quantity':product_qty,'uom':product_uom, 'company_id': parent.company_id}"
                                        force_save="1" domain="[('purchase_ok', '=', True), '|', ('company_id', '=', False), ('company_id', '=', parent.company_id)]"/>
                                    <field name="name" widget="section_and_note_text"/>
                                    <field name="date_planned" optional="hide" attrs="{'required': [('display_type', '=', False)], 'readonly': [('parent.date_planned', '!=', False)]}"/>
                                    <field name="account_analytic_id" optional="hide" context="{'default_partner_id':parent.partner_id}" groups="analytic.group_analytic_accounting" domain="['|', ('company_id', '=', False), ('company_id', '=', parent.company_id)]"/>
                                    <field name="analytic_tag_ids" optional="hide" groups="analytic.group_analytic_tags" widget="many2many_tags" options="{'color_field': 'color'}" domain="['|', ('company_id', '=', False), ('company_id', '=', parent.company_id)]"/>
                                    <field name="product_qty"/>
                                    <field name="qty_received_manual" invisible="1"/>
                                    <field name="qty_received_method" invisible="1"/>
                                    <field name="qty_received" string="Received" attrs="{'column_invisible': [('parent.state', 'not in', ('purchase', 'done'))], 'readonly': [('qty_received_method', '!=', 'manual')]}" optional="show"/>
                                    <field name="qty_invoiced" string="Billed" attrs="{'column_invisible': [('parent.state', 'not in', ('purchase', 'done'))]}" optional="show"/>
                                    <field name="product_uom" string="UoM" groups="uom.group_uom"
                                        attrs="{
                                            'readonly': [('state', 'in', ('purchase', 'done', 'cancel'))],
                                            'required': [('display_type', '=', False)]
                                        }"
                                        force_save="1" optional="show"/>
                                    <field name="price_unit" attrs="{'readonly': [('invoice_lines', '!=', [])]}"/>
                                    <field name="taxes_id" widget="many2many_tags" domain="[('type_tax_use','=','purchase'), ('company_id', '=', parent.company_id)]" context="{'default_type_tax_use': 'purchase', 'search_view_ref': 'account.account_tax_view_search'}" options="{'no_create': True}" optional="show"/>
                                    <field name="price_subtotal" widget="monetary"/>
                                    <field name="price_total" invisible="1"/>
                                    <field name="price_tax" invisible="1"/>
                                </tree>
                                <form string="Purchase Order Line">
                                        <field name="state" invisible="1"/>
                                        <field name="display_type" invisible="1"/>
                                        <group attrs="{'invisible': [('display_type', '!=', False)]}">
                                            <group>
                                                <field name="product_uom_category_id" invisible="1"/>
                                                <field name="product_id"
                                                       context="{'partner_id': parent.partner_id}"
                                                       widget="many2one_barcode"
                                                       domain="[('purchase_ok', '=', True), '|', ('company_id', '=', False), ('company_id', '=', parent.company_id)]"
                                                />
                                                <label for="product_qty"/>
                                                <div class="o_row">
                                                    <field name="product_qty"/>
                                                    <field name="product_uom" groups="uom.group_uom" attrs="{'required': [('display_type', '=', False)]}"/>
                                                </div>
                                                <field name="qty_received_method" invisible="1"/>
                                                <field name="qty_received" string="Received Quantity" attrs="{'invisible': [('parent.state', 'not in', ('purchase', 'done'))], 'readonly': [('qty_received_method', '!=', 'manual')]}"/>
                                                <field name="qty_invoiced" string="Billed Quantity" attrs="{'invisible': [('parent.state', 'not in', ('purchase', 'done'))]}"/>
                                                <field name="price_unit"/>
                                                <field name="taxes_id" widget="many2many_tags" domain="[('type_tax_use', '=', 'purchase'), ('company_id', '=', parent.company_id)]" options="{'no_create': True}"/>
                                            </group>
                                            <group>
                                                <field name="date_planned" widget="date" attrs="{'required': [('display_type', '=', False)]}"/>
                                                <field name="account_analytic_id" colspan="2" domain="['|', ('company_id', '=', False), ('company_id', '=', parent.company_id)]" groups="analytic.group_analytic_accounting"/>
                                                <field name="analytic_tag_ids" groups="analytic.group_analytic_tags" domain="['|', ('company_id', '=', False), ('company_id', '=', parent.company_id)]" widget="many2many_tags" options="{'color_field': 'color'}"/>
                                                <field name="company_id" groups="base.group_multi_company" options="{'no_create': True}"/>
                                            </group>
                                            <group colspan="12">
                                            <notebook>
                                                <page string="Notes">
                                                    <field name="name"/>
                                                </page>
                                                <page string="Invoices and Incoming Shipments">
                                                    <field name="invoice_lines"/>
                                                </page>
                                            </notebook>
                                            </group>
                                        </group>
                                        <label for="name" string="Section Name (eg. Products, Services)" attrs="{'invisible': [('display_type', '!=', 'line_section')]}"/>
                                        <label for="name" string="Note" attrs="{'invisible': [('display_type', '!=', 'line_note')]}"/>
                                        <field name="name" nolabel="1"  attrs="{'invisible': [('display_type', '=', False)]}"/>
                                 </form>
                                 <kanban class="o_kanban_mobile">
                                     <field name="name"/>
                                     <field name="product_id"/>
                                     <field name="product_qty"/>
                                     <field name="product_uom" groups="uom.group_uom"/>
                                     <field name="price_subtotal"/>
                                     <field name="price_tax" invisible="1"/>
                                     <field name="price_total" invisible="1"/>
                                     <field name="price_unit"/>
                                     <field name="display_type"/>
                                     <field name="taxes_id" invisible="1"/>
                                     <templates>
                                         <t t-name="kanban-box">
                                             <div t-attf-class="oe_kanban_card oe_kanban_global_click {{ record.display_type.raw_value ? 'o_is_' + record.display_type.raw_value : '' }}">
                                                 <t t-if="!record.display_type.raw_value">
                                                     <div class="row">
                                                         <div class="col-8">
                                                             <strong>
                                                                 <span t-esc="record.product_id.value"/>
                                                             </strong>
                                                         </div>
                                                         <div class="col-4">
                                                             <strong>
                                                                 <span t-esc="record.price_subtotal.value" class="float-right text-right"/>
                                                             </strong>
                                                         </div>
                                                     </div>
                                                     <div class="row">
                                                         <div class="col-12 text-muted">
                                                             <span>
                                                                 Quantity:
                                                                 <t t-esc="record.product_qty.value"/>
                                                                 <t t-esc="record.product_uom.value"/>
                                                             </span>
                                                         </div>
                                                     </div>
                                                     <div class="row">
                                                         <div class="col-12 text-muted">
                                                             <span>
                                                                 Unit Price:
                                                                 <t t-esc="record.price_unit.value"/>
                                                             </span>
                                                         </div>
                                                     </div>
                                                 </t>
                                                 <div
                                                     t-elif="record.display_type.raw_value === 'line_section' || record.display_type.raw_value === 'line_note'"
                                                     class="row">
                                                     <div class="col-12">
                                                         <span t-esc="record.name.value"/>
                                                     </div>
                                                 </div>
                                             </div>
                                         </t>
                                     </templates>
                                 </kanban>
                            </field>
                            <group class="oe_subtotal_footer oe_right">
                                <field name="amount_untaxed" widget="monetary" options="{'currency_field': 'currency_id'}"/>
                                <field name="amount_tax" widget="monetary" options="{'currency_field': 'currency_id'}"/>
                                <div class="oe_subtotal_footer_separator oe_inline">
                                    <label for="amount_total"/>
                                </div>
                                <field name="amount_total" nolabel="1" class="oe_subtotal_footer_separator" widget="monetary" options="{'currency_field': 'currency_id'}"/>
                            </group>
                            <field name="notes" class="oe_inline" placeholder="Define your terms and conditions ..."/>
                            <div class="oe_clear"/>
                        </page>
                        <page string="Other Information" name="purchase_delivery_invoice">
                            <group>
                                <group name="planning">
                                    <label for="date_planned"/>
                                    <div>
                                        <field name="date_planned" attrs="{'readonly': [('state', 'not in', ('draft', 'sent'))]}"/>
                                    </div>
                                </group>
                                <group name="other_info">
                                    <field name="user_id"/>
                                    <field name="invoice_status" attrs="{'invisible': [('state', 'in', ('draft', 'sent', 'to approve', 'cancel'))]}"/>
                                    <field name="payment_term_id" attrs="{'readonly': ['|', ('invoice_status','=', 'invoiced'), ('state', '=', 'done')]}" options="{'no_create': True}"/>
                                    <field name="fiscal_position_id" options="{'no_create': True}" attrs="{'readonly': ['|', ('invoice_status','=', 'invoiced'), ('state', '=', 'done')]}"/>
                                </group>
                            </group>
                        </page>
                    </notebook>
                </sheet>
                <div class="oe_chatter">
                    <field name="message_follower_ids" widget="mail_followers"/>
                    <field name="activity_ids" widget="mail_activity"/>
                    <field name="message_ids" widget="mail_thread"/>
                </div>
                </form>
            </field>
        </record>

       <record id="view_purchase_order_filter" model="ir.ui.view">
            <field name="name">request.quotation.select</field>
            <field name="model">purchase.order</field>
            <field name="arch" type="xml">
                <search string="Search Purchase Order">
                    <field name="name" string="Order"
                        filter_domain="['|', '|', ('name', 'ilike', self), ('partner_ref', 'ilike', self), ('partner_id', 'child_of', self)]"/>
                    <field name="partner_id" operator="child_of"/>
                    <field name="user_id"/>
                    <field name="product_id"/>
                    <filter name="my_purchases" string="My Purchases" domain="[('user_id', '=', uid)]"/>
                    <separator/>
                    <filter name="draft" string="RFQs" domain="[('state', 'in', ('draft', 'sent', 'to approve'))]"/>
                    <filter name="approved" string="Purchase Orders" domain="[('state', 'in', ('purchase', 'done'))]"/>
                    <filter name="to_approve" string="To Approve" domain="[('state', '=', 'to approve')]"/>
                    <separator/>
                    <filter name="order_date" string="Order Date" date="date_order"/>
                    <separator/>
                    <filter invisible="1" string="Late Activities" name="activities_overdue"
                        domain="[('activity_ids.date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                        help="Show all records which has next action date is before today"/>
                    <filter invisible="1" string="Today Activities" name="activities_today"
                        domain="[('activity_ids.date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                    <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                        domain="[('activity_ids.date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
                    <separator/>
                    <filter string="Warnings" name="activities_exception"
                        domain="[('activity_exception_decoration', '!=', False)]"/>
                    <group expand="0" string="Group By">
                        <filter string="Vendor" name="vendor" domain="[]" context="{'group_by': 'partner_id'}"/>
                        <filter string="Purchase Representative" name="representative" domain="[]" context="{'group_by': 'user_id'}"/>
                        <filter string="Order Date" name="order_date" domain="[]" context="{'group_by': 'date_order'}"/>
                    </group>
                </search>
            </field>
        </record>

        <record id="purchase_order_view_search" model="ir.ui.view">
            <field name="name">purchase.order.select</field>
            <field name="model">purchase.order</field>
            <field name="arch" type="xml">
                <search string="Search Purchase Order">
                    <field name="name" string="Order"
                        filter_domain="['|', '|', ('name', 'ilike', self), ('partner_ref', 'ilike', self), ('partner_id', 'child_of', self)]"/>
                    <field name="partner_id" operator="child_of"/>
                    <field name="user_id"/>
                    <field name="product_id"/>
                    <filter name="my_Orders" string="My Orders" domain="[('user_id', '=', uid)]"/>
                    <separator/>
                    <filter name="not_invoiced" string="Waiting Bills" domain="[('invoice_status', '=', 'to invoice')]" help="Purchase orders that include lines not invoiced."/>
                    <filter name="invoiced" string="Bills Received" domain="[('invoice_status', '=', 'invoiced')]" help="Purchase orders that have been invoiced."/>
                    <separator/>
                    <filter name="order_date" string="Order Date" date="date_order"/>
                    <separator/>
                    <filter invisible="1" string="Late Activities" name="activities_overdue"
                        domain="[('activity_ids.date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                        help="Show all records which has next action date is before today"/>
                    <filter invisible="1" string="Today Activities" name="activities_today"
                        domain="[('activity_ids.date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                    <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                        domain="[('activity_ids.date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
                    <separator/>
                    <filter string="Warnings" name="activities_exception"
                        domain="[('activity_exception_decoration', '!=', False)]"/>
                    <group expand="0" string="Group By">
                        <filter string="Vendor" name="vendor" domain="[]" context="{'group_by': 'partner_id'}"/>
                        <filter string="Purchase Representative" name="representative" domain="[]" context="{'group_by': 'user_id'}"/>
                        <filter string="Order Date" name="order_date" domain="[]" context="{'group_by': 'date_order'}"/>
                    </group>
                </search>
            </field>
        </record>


        <!-- Purchase Orders Kanban View  -->
        <record model="ir.ui.view" id="view_purchase_order_kanban">
            <field name="name">purchase.order.kanban</field>
            <field name="model">purchase.order</field>
            <field name="arch" type="xml">
                <kanban class="o_kanban_mobile">
                    <field name="name"/>
                    <field name="partner_id" readonly="1"/>
                    <field name="amount_total"/>
                    <field name="state"/>
                    <field name="date_order"/>
                    <field name="currency_id" readonly="1"/>
                    <field name="activity_state"/>
                    <progressbar field="activity_state" colors='{"planned": "success", "today": "warning", "overdue": "danger"}'/>
                    <templates>
                        <t t-name="kanban-box">
                            <div t-attf-class="oe_kanban_card oe_kanban_global_click">
                                <div class="o_kanban_record_top mb16">
                                    <div class="o_kanban_record_headings mt4">
                                        <strong class="o_kanban_record_title"><span><t t-esc="record.partner_id.value"/></span></strong>
                                    </div>
                                    <strong><field name="amount_total" widget="monetary"/></strong>
                                </div>
                                <div class="o_kanban_record_bottom">
                                    <div class="oe_kanban_bottom_left">
                                        <span><t t-esc="record.name.value"/> <t t-esc="record.date_order.value and record.date_order.value.split(' ')[0] or False"/></span>
                                        <field name="activity_ids" widget="kanban_activity"/>
                                    </div>
                                    <div class="oe_kanban_bottom_right">
                                        <field name="state" widget="label_selection" options="{'classes': {'draft': 'default', 'cancel': 'default', 'done': 'success', 'approved': 'warning'}}"/>
                                    </div>
                                </div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="purchase_order_tree" model="ir.ui.view">
            <field name="name">purchase.order.tree</field>
            <field name="model">purchase.order</field>
            <field name="arch" type="xml">
                <tree string="Purchase Order" multi_edit="1" decoration-bf="message_unread==True"
                      decoration-muted="state=='cancel'" decoration-info="state in ('wait','confirmed')" class="o_purchase_order">
                    <field name="message_unread" invisible="1"/>
                    <field name="partner_ref" optional="hide"/>
                    <field name="name" string="Reference" readonly="1"/>
                    <field name="date_order" invisible="not context.get('quotation_only', False)" optional="show"/>
                    <field name="date_approve" invisible="context.get('quotation_only', False)" optional="show"/>
                    <field name="partner_id" readonly="1"/>
                    <field name="company_id" readonly="1" options="{'no_create': True}"
                        groups="base.group_multi_company" optional="show"/>
                    <field name="date_planned" invisible="context.get('quotation_only', False)" optional="show"/>
                    <field name="user_id" optional="show"/>
                    <field name="origin" optional="show"/>
                    <field name="amount_untaxed" sum="Total Untaxed amount" string="Untaxed" widget="monetary" optional="hide"/>
                    <field name="amount_total" sum="Total amount" widget="monetary" optional="show"/>
                    <field name="currency_id" invisible="1"/>
                    <field name="state" optional="show"/>
                    <field name="invoice_status" optional="hide"/>
                    <field name="activity_exception_decoration" widget="activity_exception"/>
                </tree>
            </field>
        </record>

        <record id="purchase_order_view_tree" model="ir.ui.view">
            <field name="name">purchase.order.view.tree</field>
            <field name="model">purchase.order</field>
            <field name="arch" type="xml">
                <tree decoration-bf="message_unread==True" decoration-muted="state=='cancel'" decoration-info="state in ('wait','confirmed')" string="Purchase Order" class="o_purchase_order">
                    <field name="message_unread" invisible="1"/>
                    <field name="partner_ref" optional="hide"/>
                    <field name="name" string="Reference" readonly="1"/>
                    <field name="date_order" invisible="not context.get('quotation_only', False)" optional="show"/>
                    <field name="date_approve" invisible="context.get('quotation_only', False)" optional="show"/>
                    <field name="partner_id"/>
                    <field name="company_id" groups="base.group_multi_company" options="{'no_create': True}" optional="show"/>
                    <field name="date_planned" invisible="context.get('quotation_only', False)" optional="show"/>
                    <field name="user_id" optional="show"/>
                    <field name="origin" optional="show"/>
                    <field name="amount_untaxed" sum="Total Untaxed amount" string="Untaxed" widget="monetary" optional="hide"/>
                    <field name="amount_total" sum="Total amount" widget="monetary" optional="show"/>
                    <field name="currency_id" invisible="1"/>
                    <field name="state" invisible="1"/>
                    <field name="invoice_status" optional="show"/>
                </tree>
            </field>
        </record>

        <record id="purchase_order_view_activity" model="ir.ui.view">
            <field name="name">purchase.order.activity</field>
            <field name="model">purchase.order</field>
            <field name="arch" type="xml">
                <activity string="Purchase Order">
                    <templates>
                        <div t-name="activity-box">
                            <div>
                                <field name="name" display="full"/>
                                <field name="partner_id" muted="1" display="full"/>
                            </div>
                        </div>
                    </templates>
                </activity>
            </field>
        </record>

        <record id="purchase_order_action_generic" model="ir.actions.act_window">
            <field name="name">Purchase Orders</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">purchase.order</field>
            <field name="domain">[]</field>
            <field name="view_id" ref="purchase_order_form"/>
        </record>

        <record id="purchase_rfq" model="ir.actions.act_window">
            <field name="name">Requests for Quotation</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">purchase.order</field>
            <field name="view_mode">tree,kanban,form,pivot,graph,calendar,activity</field>
            <field name="view_id" ref="purchase_order_tree"/>
            <field name="domain">[]</field>
            <field name="context">{}</field>
            <field name="search_view_id" ref="view_purchase_order_filter"/>
            <field name="context">{'quotation_only': True}</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Create a request for quotation
              </p><p>
                The quotation contains the history of the discussion
                you had with your vendor.
              </p>
            </field>
        </record>
        <menuitem action="purchase_rfq" id="menu_purchase_rfq"
            parent="menu_procurement_management"
            sequence="0"/>

        <record id="purchase_form_action" model="ir.actions.act_window">
            <field name="name">Purchase Orders</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">purchase.order</field>
            <field name="view_mode">tree,kanban,form,pivot,graph,calendar,activity</field>
            <field name="view_id" ref="purchase_order_view_tree"/>
            <field name="domain">[('state','in',('purchase', 'done'))]</field>
            <field name="search_view_id" ref="purchase_order_view_search"/>
            <field name="context">{}</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Create a quotation
              </p><p>
                It will be converted into a purchase order.
              </p>
            </field>
        </record>
        <menuitem action="purchase_form_action" id="menu_purchase_form_action" parent="menu_procurement_management" sequence="6"/>

        <record id="purchase_order_line_tree" model="ir.ui.view">
            <field name="name">purchase.order.line.tree</field>
            <field name="model">purchase.order.line</field>
            <field name="arch" type="xml">
                <tree string="Purchase Order Lines" create="false">
                    <field name="order_id"/>
                    <field name="name"/>
                    <field name="partner_id" string="Vendor" />
                    <field name="product_id"/>
                    <field name="price_unit"/>
                    <field name="product_qty"/>
                    <field name="product_uom" groups="uom.group_uom"/>
                    <field name="price_subtotal" widget="monetary"/>
                    <field name="date_planned"  widget="date"/>
                </tree>
            </field>
        </record>

        <record id="purchase_order_line_form2" model="ir.ui.view">
            <field name="name">purchase.order.line.form2</field>
            <field name="model">purchase.order.line</field>
            <field name="priority" eval="20"/>
            <field name="arch" type="xml">
                <form string="Purchase Order Line" create="false">
                    <sheet>
                        <label for="order_id" class="oe_edit_only"/>
                        <h1>
                            <field name="order_id" class="oe_inline"/>
                            <label string="," for="date_order" attrs="{'invisible':[('date_order','=',False)]}"/>
                            <field name="date_order" class="oe_inline"/>
                        </h1>
                        <label for="partner_id" class="oe_edit_only"/>
                        <h2><field name="partner_id"/></h2>
                        <group>
                            <group>
                                <field name="product_id" readonly="1"/>
                                <label for="product_qty"/>
                                <div class="o_row">
                                    <field name="product_qty" readonly="1"/>
                                    <field name="product_uom" readonly="1" groups="uom.group_uom"/>
                                </div>
                                <field name="price_unit"/>
                            </group>
                            <group>
                                <field name="taxes_id" widget="many2many_tags"
                                    domain="[('type_tax_use', '=', 'purchase')]"/>
                                <field name="date_planned" widget="date" readonly="1"/>
                                <field name="company_id" groups="base.group_multi_company" options="{'no_create': True}"/>
                                <field name="account_analytic_id" colspan="4" groups="analytic.group_analytic_accounting"/>
                            </group>
                        </group>
                        <field name="name"/>
                        <separator string="Manual Invoices"/>
                        <field name="invoice_lines"/>
                    </sheet>
                </form>
            </field>
        </record>
          <record id="purchase_order_line_search" model="ir.ui.view">
            <field name="name">purchase.order.line.search</field>
            <field name="model">purchase.order.line</field>
            <field name="arch" type="xml">
                <search string="Search Purchase Order">
                    <field name="order_id"/>
                    <field name="product_id"/>
                    <field name="partner_id" string="Vendor" filter_domain="[('partner_id', 'child_of', self)]"/>
                    <filter name="hide_cancelled" string="Hide cancelled lines" domain="[('state', '!=', 'cancel')]"/>
                    <group expand="0" string="Group By">
                        <filter name="groupby_supplier" string="Vendor" domain="[]" context="{'group_by' : 'partner_id'}" />
                        <filter name="groupby_product" string="Product" domain="[]" context="{'group_by' : 'product_id'}" />
                        <filter string="Order Reference" name="order_reference" domain="[]" context="{'group_by' :'order_id'}"/>
                        <filter string="Status" name="status" domain="[]" context="{'group_by' : 'state'}" />
                    </group>
                </search>
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
        <field name="inherit_id" ref="base.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[hasclass('settings')]" position="inside">
                <div class="app_settings_block" data-string="Purchase" string="Purchase" data-key="purchase" groups="purchase.group_purchase_manager">
                    <field name="po_double_validation" invisible="1"/>
                    <field name="company_currency_id" invisible="1"/>
                    <field name="po_lock" invisible="1"/>
                    <h2>Orders</h2>
                    <div class="row mt16 o_settings_container" name="purchase_setting_container">
                        <div class="col-12 col-lg-6 o_setting_box">
                            <div class="o_setting_left_pane">
                                <field name="po_order_approval"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="po_order_approval"/>
                                <span class="fa fa-lg fa-building-o" title="Values set here are company-specific." aria-label="Values set here are company-specific." groups="base.group_multi_company" role="img"/>
                                <div class="text-muted">
                                    Request managers to approve orders above a minimum amount
                                </div>
                                <div class="content-group" attrs="{'invisible': [('po_order_approval', '=', False)]}">
                                    <div class="row mt16">
                                        <label for="po_double_validation_amount" class="col-lg-4 o_light_label"/>
                                        <field name="po_double_validation_amount"/>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box">
                            <div class="o_setting_left_pane">
                                <field name="lock_confirmed_po"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="lock_confirmed_po"/>
                                <div class="text-muted">
                                    Automatically lock confirmed orders to prevent edition
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box">
                            <div class="o_setting_left_pane">
                                <field name="group_warning_purchase"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="group_warning_purchase" string="Warnings"/>
                                <div class="text-muted">
                                    Get warnings in orders for products or vendors
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box" title="Calls for tenders are used when you want to generate requests for quotations to several vendors for a given set of products. You can configure per product if you directly do a Request for Quotation to one vendor or if you want a Call for Tenders to compare offers from several vendors.">
                            <div class="o_setting_left_pane">
                                <field name="module_purchase_requisition"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="module_purchase_requisition"/>
                                <div class="text-muted">
                                    Manage your purchase agreements (call for tenders, blanket orders)
                                </div>
                                <div class="content-group" attrs="{'invisible': [('module_purchase_requisition', '=', False)]}">
                                    <div id="use_purchase_requisition"/>
                                </div>
                            </div>
                        </div>
                    </div>
                    <h2>Invoicing</h2>
                    <div class="row mt16 o_settings_container">
                        <div class="col-12 col-lg-6 o_setting_box" title="This default value is applied to any new product created. This can be changed in the product detail form.">
                            <div class="o_setting_left_pane"/>
                            <div class="o_setting_right_pane">
                                <label for="default_purchase_method"/>
                                <div class="text-muted">
                                    Quantities billed by vendors
                                </div>
                                <div class="content-group">
                                    <div class="mt16">
                                        <field name="default_purchase_method" class="o_light_label" widget="radio"/>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box" title="If enabled, activates 3-way matching on vendor bills : the items must be received in order to pay the invoice.">
                            <div class="o_setting_left_pane">
                                <field name="module_account_3way_match" string="3-way matching" widget="upgrade_boolean"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="module_account_3way_match"/>
                                <div class="text-muted">
                                    Make sure you only pay bills for which you received the goods you ordered
                                </div>
                            </div>
                        </div>
                    </div>
                    <h2>Products</h2>
                    <div class="row mt16 o_settings_container">
                        <div class="col-12 col-lg-6 o_setting_box" title="If installed, the product variants will be added to purchase orders through a grid entry.">
                            <div class="o_setting_left_pane">
                                <field name="module_purchase_product_matrix"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="module_purchase_product_matrix" string="Variant Grid Entry"/>
                                <div class="text-muted">
                                    Add several variants to the purchase order from a grid
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </xpath>
        </field>
    </record>

    <record id="action_purchase_configuration" model="ir.actions.act_window">
        <field name="name">Settings</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">res.config.settings</field>
        <field name="view_mode">form</field>
        <field name="target">inline</field>
        <field name="context">{'module' : 'purchase', 'bin_size': False}</field>
    </record>

    <menuitem id="menu_purchase_general_settings" name="Settings" parent="menu_purchase_config"
        sequence="0" action="action_purchase_configuration" groups="base.group_system"/>

</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_partner_property_form" model="ir.ui.view">
            <field name="name">res.partner.purchase.property.form.inherit</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="base.view_partner_form"/>
            <field name="priority">36</field>
            <field name="groups_id" eval="[(4, ref('base.group_multi_currency'))]"/>
            <field name="arch" type="xml">
                <group name="purchase" position="inside">
                    <field name="property_purchase_currency_id" options="{'no_create': True, 'no_open': True}"/>
                </group>
            </field>
    </record>
    <record id="act_res_partner_2_purchase_order" model="ir.actions.act_window">
            <field name="name">RFQs and Purchases</field>
            <field name="res_model">purchase.order</field>
            <field name="view_mode">tree,form,graph</field>
            <field name="context">{'search_default_partner_id': active_id, 'default_partner_id': active_id}</field>
            <field name="groups_id" eval="[(4, ref('purchase.group_purchase_user'))]"/>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    This vendor has no purchase order. Create a new RfQ
                </p><p>
                    The request for quotation is the first step of the purchases flow. Once
                    converted into a purchase order, you will be able to control the receipt
                    of the products and the vendor bill.
                </p>
            </field>
        </record>

        <!-- Partner kanban view inherited -->
        <record model="ir.ui.view" id="purchase_partner_kanban_view">
            <field name="name">res.partner.kanban.purchaseorder.inherit</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="base.res_partner_kanban_view"/>
            <field name="groups_id" eval="[(4, ref('purchase.group_purchase_user'))]"/>
            <field name="arch" type="xml">
                <field name="mobile" position="after">
                    <field name="purchase_order_count"/>
                </field>
                <xpath expr="//div[hasclass('oe_kanban_partner_links')]" position="inside">
                    <span t-if="record.purchase_order_count.value>0" class="badge badge-pill"><i class="fa fa-fw fa-shopping-cart" role="img" aria-label="Shopping cart" title="Shopping cart"/><t t-esc="record.purchase_order_count.value"/></span>
                </xpath>
            </field>
        </record>

    <record id="act_res_partner_2_supplier_invoices" model="ir.actions.act_window">
            <field name="name">Vendor Bills</field>
            <field name="res_model">account.move</field>
            <field name="view_mode">tree,form,graph</field>
            <field name="domain">[('type','in',('in_invoice', 'in_refund'))]</field>
            <field name="context">{'search_default_partner_id': active_id, 'default_type': 'in_invoice', 'default_partner_id': active_id}</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Record a new vendor bill
                </p><p>
                    Vendors bills can be pre-generated based on purchase
                    orders or receipts. This allows you to control bills
                    you receive from your vendor according to the draft
                    document in Odoo.
                </p>
            </field>
        </record>

        <record id="res_partner_view_purchase_buttons" model="ir.ui.view">
            <field name="name">res.partner.view.purchase.buttons</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="base.view_partner_form" />
            <field name="priority" eval="9"/>
            <field name="groups_id" eval="[(4, ref('purchase.group_purchase_user'))]"/>
            <field name="arch" type="xml">
                <div name="button_box" position="inside">
                    <button class="oe_stat_button" name="%(purchase.act_res_partner_2_purchase_order)d" type="action"
                        groups="purchase.group_purchase_user"
                        icon="fa-shopping-cart">
                        <field string="Purchases" name="purchase_order_count" widget="statinfo"/>
                    </button>
                </div>
                <page name="internal_notes" position="inside">
                    <group colspan="2" col="2" groups="purchase.group_warning_purchase">
                        <separator string="Warning on the Purchase Order" colspan="4"/>
                        <field name="purchase_warn" nolabel="1" />
                        <field name="purchase_warn_msg" colspan="3" nolabel="1"
                                attrs="{'required':[('purchase_warn', '!=', False), ('purchase_warn','!=','no-message')],'readonly':[('purchase_warn','=','no-message')]}"/>
                    </group>
                </page>
            </field>
        </record>

        <record id="res_partner_view_purchase_account_buttons" model="ir.ui.view">
            <field name="name">res.partner.view.purchase.account.buttons</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="base.view_partner_form" />
            <field name="priority" eval="12"/>
            <field name="groups_id" eval="[(4, ref('account.group_account_invoice'))]"/>
            <field name="arch" type="xml">
                <div name="button_box" position="inside">
                    <button class="oe_stat_button" name="%(purchase.act_res_partner_2_supplier_invoices)d" type="action"
                        icon="fa-pencil-square-o" help="Vendor Bills">
                        <field string="Vendor Bills" name="supplier_invoice_count" widget="statinfo"/>
                    </button>
                </div>
            </field>
        </record>
</odoo>

```

