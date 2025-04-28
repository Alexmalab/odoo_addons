# Odoo Module: purchase

Category: Inventory/Purchase

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models
from . import report
from . import populate

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Purchase',
    'version': '1.2',
    'category': 'Inventory/Purchase',
    'sequence': 35,
    'summary': 'Purchase orders, tenders and agreements',
    'website': 'https://www.odoo.com/app/purchase',
    'depends': ['account'],
    'data': [
        'security/purchase_security.xml',
        'security/ir.model.access.csv',
        'data/digest_data.xml',
        'views/account_move_views.xml',
        'data/purchase_data.xml',
        'data/ir_cron_data.xml',
        'report/purchase_reports.xml',
        'views/purchase_views.xml',
        'views/res_config_settings_views.xml',
        'views/product_views.xml',
        'views/res_partner_views.xml',
        'report/purchase_bill_views.xml',
        'report/purchase_report_views.xml',
        'data/mail_templates.xml',
        'data/mail_template_data.xml',
        'views/portal_templates.xml',
        'report/purchase_order_templates.xml',
        'report/purchase_quotation_templates.xml',
        'views/product_packaging_views.xml',
        'views/analytic_account_views.xml',
    ],
    'demo': [
        'data/purchase_demo.xml',
    ],
    'installable': True,
    'application': True,
    'assets': {
        'web.assets_backend': [
            'purchase/static/src/product_catalog/**/*',
            'purchase/static/src/toaster_button/*',
            'purchase/static/src/views/*.js',
            'purchase/static/src/js/tours/purchase.js',
            'purchase/static/src/js/tours/purchase_steps.js',
            'purchase/static/src/**/*.xml',
        ],
        'web.assets_frontend': [
            'purchase/static/src/js/purchase_datetimepicker.js',
            'purchase/static/src/js/purchase_portal_sidebar.js',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\portal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
from collections import OrderedDict
from datetime import datetime

from odoo import http
from odoo.exceptions import AccessError, MissingError
from odoo.http import request, Response
from odoo.tools import image_process
from odoo.tools.translate import _
from odoo.addons.portal.controllers import portal
from odoo.addons.portal.controllers.portal import pager as portal_pager


class CustomerPortal(portal.CustomerPortal):

    def _prepare_home_portal_values(self, counters):
        values = super()._prepare_home_portal_values(counters)
        PurchaseOrder = request.env['purchase.order']
        if 'rfq_count' in counters:
            values['rfq_count'] = PurchaseOrder.search_count([
                ('state', 'in', ['sent'])
            ]) if PurchaseOrder.check_access_rights('read', raise_exception=False) else 0
        if 'purchase_count' in counters:
            values['purchase_count'] = PurchaseOrder.search_count([
                ('state', 'in', ['purchase', 'done', 'cancel'])
            ]) if PurchaseOrder.check_access_rights('read', raise_exception=False) else 0
        return values

    def _get_purchase_searchbar_sortings(self):
        return {
            'date': {'label': _('Newest'), 'order': 'create_date desc, id desc'},
            'name': {'label': _('Name'), 'order': 'name asc, id asc'},
            'amount_total': {'label': _('Total'), 'order': 'amount_total desc, id desc'},
        }

    def _render_portal(self, template, page, date_begin, date_end, sortby, filterby, domain, searchbar_filters, default_filter, url, history, page_name, key):
        values = self._prepare_portal_layout_values()
        PurchaseOrder = request.env['purchase.order']

        if date_begin and date_end:
            domain += [('create_date', '>', date_begin), ('create_date', '<=', date_end)]

        searchbar_sortings = self._get_purchase_searchbar_sortings()
        # default sort
        if not sortby:
            sortby = 'date'
        order = searchbar_sortings[sortby]['order']

        if searchbar_filters:
            # default filter
            if not filterby:
                filterby = default_filter
            domain += searchbar_filters[filterby]['domain']

        # count for pager
        count = PurchaseOrder.search_count(domain)

        # make pager
        pager = portal_pager(
            url=url,
            url_args={'date_begin': date_begin, 'date_end': date_end, 'sortby': sortby, 'filterby': filterby},
            total=count,
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
        request.session[history] = orders.ids[:100]

        values.update({
            'date': date_begin,
            key: orders,
            'page_name': page_name,
            'pager': pager,
            'searchbar_sortings': searchbar_sortings,
            'sortby': sortby,
            'searchbar_filters': OrderedDict(sorted(searchbar_filters.items())),
            'filterby': filterby,
            'default_url': url,
        })
        return request.render(template, values)

    def _purchase_order_get_page_view_values(self, order, access_token, **kwargs):
        #
        def resize_to_48(source):
            if not source:
                source = request.env['ir.binary']._placeholder()
            else:
                source = base64.b64decode(source)
            return base64.b64encode(image_process(source, size=(48, 48)))

        values = {
            'order': order,
            'resize_to_48': resize_to_48,
            'report_type': 'html',
        }
        if order.state in ('sent'):
            history = 'my_rfqs_history'
        else:
            history = 'my_purchases_history'
        return self._get_page_view_values(order, access_token, values, history, False, **kwargs)

    @http.route(['/my/rfq', '/my/rfq/page/<int:page>'], type='http', auth="user", website=True)
    def portal_my_requests_for_quotation(self, page=1, date_begin=None, date_end=None, sortby=None, filterby=None, **kw):
        return self._render_portal(
            "purchase.portal_my_purchase_rfqs",
            page, date_begin, date_end, sortby, filterby,
            [('state', '=', 'sent')],
            {},
            None,
            "/my/rfq",
            'my_rfqs_history',
            'rfq',
            'rfqs'
        )

    @http.route(['/my/purchase', '/my/purchase/page/<int:page>'], type='http', auth="user", website=True)
    def portal_my_purchase_orders(self, page=1, date_begin=None, date_end=None, sortby=None, filterby=None, **kw):
        return self._render_portal(
            "purchase.portal_my_purchase_orders",
            page, date_begin, date_end, sortby, filterby,
            [],
            {
                'all': {'label': _('All'), 'domain': [('state', 'in', ['purchase', 'done', 'cancel'])]},
                'purchase': {'label': _('Purchase Order'), 'domain': [('state', '=', 'purchase')]},
                'cancel': {'label': _('Cancelled'), 'domain': [('state', '=', 'cancel')]},
                'done': {'label': _('Locked'), 'domain': [('state', '=', 'done')]},
            },
            'all',
            "/my/purchase",
            'my_purchases_history',
            'purchase',
            'orders'
        )

    @http.route(['/my/purchase/<int:order_id>'], type='http', auth="public", website=True)
    def portal_my_purchase_order(self, order_id=None, access_token=None, **kw):
        try:
            order_sudo = self._document_check_access('purchase.order', order_id, access_token=access_token)
        except (AccessError, MissingError):
            return request.redirect('/my')

        report_type = kw.get('report_type')
        if report_type in ('html', 'pdf', 'text'):
            return self._show_report(model=order_sudo, report_type=report_type, report_ref='purchase.action_report_purchase_order', download=kw.get('download'))

        confirm_type = kw.get('confirm')
        if confirm_type == 'reminder':
            order_sudo.confirm_reminder_mail(kw.get('confirmed_date'))
        if confirm_type == 'reception':
            order_sudo._confirm_reception_mail()

        values = self._purchase_order_get_page_view_values(order_sudo, access_token, **kw)
        update_date = kw.get('update')
        if order_sudo.company_id:
            values['res_company'] = order_sudo.company_id
        if update_date == 'True':
            return request.render("purchase.portal_my_purchase_order_update_date", values)
        return request.render("purchase.portal_my_purchase_order", values)

    @http.route(['/my/purchase/<int:order_id>/update'], type='json', auth="public", website=True)
    def portal_my_purchase_order_update_dates(self, order_id=None, access_token=None, **kw):
        """User update scheduled date on purchase order line.
        """
        try:
            order_sudo = self._document_check_access('purchase.order', order_id, access_token=access_token)
        except (AccessError, MissingError):
            return request.redirect('/my')

        updated_dates = []
        for id_str, date_str in kw.items():
            try:
                line_id = int(id_str)
            except ValueError:
                return request.redirect(order_sudo.get_portal_url())
            line = order_sudo.order_line.filtered(lambda l: l.id == line_id)
            if not line:
                return request.redirect(order_sudo.get_portal_url())

            try:
                updated_date = line._convert_to_middle_of_day(datetime.strptime(date_str, '%Y-%m-%d'))
            except ValueError:
                continue

            updated_dates.append((line, updated_date))

        if updated_dates:
            order_sudo._update_date_planned_for_lines(updated_dates)
        return Response(status=204)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import portal

```

## File: data\digest_data.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<odoo>
    <data>
        <record id="digest_tip_purchase_0" model="digest.tip">
            <field name="name">Tip: How to keep late receipts under control?</field>
            <field name="sequence">100</field>
            <field name="group_id" ref="purchase.group_purchase_user" />
            <field name="tip_description" type="html">
<div>
    <p class="tip_title">Tip: How to keep late receipts under control?</p>
    <p class="tip_content">When creating a purchase order, have a look at the vendor's <i>On Time Delivery</i> rate: the percentage of products shipped on time. If it is too low, activate the <i>automated reminders</i>. A few days before the due shipment, Odoo will send the vendor an email to ask confirmation of shipment dates and keep you informed in case of any delays. To get the vendor's performance statistics, click on the OTD rate.</p>
    <img src="https://download.odoocdn.com/digests/purchase/static/src/img/milk-OTDPurchase.gif" width="540" class="illustration_border" />
</div>
            </field>
        </record>
        <record id="digest_tip_purchase_1" model="digest.tip">
            <field name="name">Tip: Never miss a purchase order</field>
            <field name="sequence">2000</field>
            <field name="group_id" ref="purchase.group_purchase_user" />
            <field name="tip_description" type="html">
<div>
    <p class="tip_title">Tip: Never miss a purchase order</p>
    <p class="tip_content">When sending a purchase order by email, Odoo asks the vendor to acknowledge the reception of the order. When the vendor acknowledges the order by clicking on a button in the email, the information is added on the purchase order. Use filters to track orders that have not been acknowledged.</p>
</div>
            </field>
        </record>
    </data>
</odoo>

```

## File: data\ir_cron_data.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<odoo noupdate="1">
    <record forcecreate="True" id="purchase_send_reminder_mail" model="ir.cron">
        <field name="name">Purchase reminder</field>
        <field name="user_id" ref="base.user_root" />
        <field name="interval_number">1</field>
        <field name="interval_type">days</field>
        <field name="numbercall">-1</field>
        <field name="doall">1</field>
        <field name="model_id" ref="model_purchase_order"/>
        <field name="state">code</field>
        <field name="code">model._send_reminder_mail()</field>
    </record>
</odoo>

```

## File: data\mail_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">

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

    <template id="track_po_line_qty_received_template">
        <div>
            <strong>The received quantity has been updated.</strong>
            <ul>
                <li><t t-esc="line.product_id.display_name"/>:</li>
                Received Quantity: <t t-esc="line.qty_received" /> -&gt; <t t-esc="float(qty_received)"/><br/>
            </ul>
        </div>
    </template>

</data></odoo>

```

## File: data\mail_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="email_template_edi_purchase" model="mail.template">
            <field name="name">Purchase: Request For Quotation</field>
            <field name="model_id" ref="purchase.model_purchase_order"/>
            <field name="subject">{{ object.company_id.name }} Order (Ref {{ object.name or 'n/a' }})</field>
            <field name="partner_to">{{ object.partner_id.id }}</field>
            <field name="description">Sent manually to vendor to request a quotation</field>
            <field name="body_html" type="html">
<div style="margin: 0px; padding: 0px;">
    <p style="margin: 0px; padding: 0px; font-size: 13px;">
        Dear <t t-out="object.partner_id.name or ''">Brandon Freeman</t>
        <t t-if="object.partner_id.parent_id">
            (<t t-out="object.partner_id.parent_id.name or ''">Azure Interior</t>)
        </t>
        <br/><br/>
        Here is in attachment a request for quotation <span style="font-weight:bold;" t-out="object.name or ''">P00015</span>
        <t t-if="object.partner_ref">
            with reference: <t t-out="object.partner_ref or ''">REF_XXX</t>
        </t>
        from <t t-out="object.company_id.name or ''">YourCompany</t>.
        <br/><br/>
        If you have any questions, please do not hesitate to contact us.
        <br/><br/>
        Best regards,
        <t t-if="not is_html_empty(object.user_id.signature)">
            <br/><br/>
            <t t-out="object.user_id.signature or ''">--<br/>Mitchell Admin</t>
        </t>
    </p>
</div></field>
            <field name="report_template_ids" eval="[(4, ref('purchase.report_purchase_quotation'))]"/>
            <field name="lang">{{ object.partner_id.lang }}</field>
            <field name="auto_delete" eval="True"/>
        </record>

        <record id="email_template_edi_purchase_done" model="mail.template">
            <field name="name">Purchase: Purchase Order</field>
            <field name="model_id" ref="purchase.model_purchase_order"/>
            <field name="subject">{{ object.company_id.name }} Order (Ref {{ object.name or 'n/a' }})</field>
            <field name="partner_to">{{ object.partner_id.id }}</field>
            <field name="description">Sent to vendor with the purchase order in attachment</field>
            <field name="body_html" type="html">
<div style="margin: 0px; padding: 0px;">
    <p style="margin: 0px; padding: 0px; font-size: 13px;">
        Dear <t t-out="object.partner_id.name or ''">Brandon Freeman</t>
        <t t-if="object.partner_id.parent_id">
            (<t t-out="object.partner_id.parent_id.name or ''">Azure Interior</t>)
        </t>
        <br/><br/>
        Here is in attachment a purchase order <span style="font-weight:bold;" t-out="object.name or ''">P00015</span>
        <t t-if="object.partner_ref">
            with reference: <t t-out="object.partner_ref or ''">REF_XXX</t>
        </t>
        amounting in <span style="font-weight:bold;" t-out="format_amount(object.amount_total, object.currency_id) or ''">$ 10.00</span>
        from <t t-out="object.company_id.name or ''">YourCompany</t>. 
        <br/><br/>
        <t t-if="object.date_planned">
            The receipt is expected for <span style="font-weight:bold;" t-out="format_date(object.date_planned) or ''">05/05/2021</span>.
            <br/><br/>
            Could you please acknowledge the receipt of this order?
        </t>
        <t t-if="not is_html_empty(object.user_id.signature)">
            <br/><br/>
            <t t-out="object.user_id.signature or ''">--<br/>Mitchell Admin</t>
        </t>
        <br/><br/>
    </p>
</div></field>
            <field name="report_template_ids" eval="[(4, ref('purchase.action_report_purchase_order'))]"/>
            <field name="lang">{{ object.partner_id.lang }}</field>
            <field name="auto_delete" eval="True"/>
        </record>

        <record id="email_template_edi_purchase_reminder" model="mail.template">
            <field name="name">Purchase: Vendor Reminder</field>
            <field name="model_id" ref="purchase.model_purchase_order"/>
            <field name="email_from">{{ (object.user_id.email_formatted or user.email_formatted) }}</field>
            <field name="subject">{{ object.company_id.name }} Order (Ref {{ object.name or 'n/a' }})</field>
            <field name="partner_to">{{ object.partner_id.id }}</field>
            <field name="description">Sent to vendors before expected arrival, based on the purchase order setting</field>
            <field name="body_html" type="html">
<div style="margin: 0px; padding: 0px;">
    <p style="margin: 0px; padding: 0px; font-size: 13px;">
        Dear <t t-out="object.partner_id.name or ''">Brandon Freeman</t>
        <t t-if="object.partner_id.parent_id">
            (<t t-out="object.partner_id.parent_id.name or ''">Azure Interior</t>)
        </t>
        <br/><br/>
        Here is a reminder that the delivery of the purchase order <span style="font-weight:bold;" t-out="object.name or ''">P00015</span>
        <t t-if="object.partner_ref">
            <span style="font-weight:bold;">(<t t-out="object.partner_ref or ''">REF_XXX</t>)</span>
        </t>
        is expected for 
        <t t-if="object.date_planned">
            <span style="font-weight:bold;" t-out="format_date(object.date_planned) or ''">05/05/2021</span>.
        </t>
         <t t-else="">
            <span style="font-weight:bold;">undefined</span>.
        </t>
        Could you please confirm it will be delivered on time?
        <t t-if="not is_html_empty(object.user_id.signature)">
            <br/><br/>
            <t t-out="object.user_id.signature or ''">--<br/>Mitchell Admin</t>
        </t>
        <br/><br/>
    </p>
</div></field>
            <field name="report_template_ids" eval="[(4, ref('purchase.action_report_purchase_order'))]"/>
            <field name="lang">{{ object.partner_id.lang }}</field>
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
        <record id="mt_rfq_sent" model="mail.message.subtype">
            <field name="name">RFQ Sent</field>
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

        <!-- Default value for company_dependant field -->
        <record forcecreate="True" id="receipt_reminder_email" model="ir.property">
            <field name="name">receipt_reminder_email</field>
            <field name="type" eval="'boolean'"/>
            <field name="fields_id" search="[('model','=','res.partner'),('name','=','receipt_reminder_email')]"/>
            <field eval="False" name="value"/>
        </record>
        <record forcecreate="True" id="reminder_date_before_receipt" model="ir.property">
            <field name="name">reminder_date_before_receipt</field>
            <field name="type" eval="'integer'"/>
            <field name="fields_id" search="[('model','=','res.partner'),('name','=','reminder_date_before_receipt')]"/>
            <field eval="1" name="value"/>
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
            <field name="groups_id" eval="[(3, ref('purchase.group_purchase_manager'))]"/>
        </record>

        <record id="base.res_partner_1" model="res.partner">
            <field name="receipt_reminder_email">True</field>
        </record>

        <record id="base.res_partner_2" model="res.partner">
            <field name="receipt_reminder_email">True</field>
        </record>

        <record id="base.res_partner_12" model="res.partner">
            <field name="receipt_reminder_email">True</field>
        </record>

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
                    'price_unit': 286.70,
                    'product_qty': 5.0,
                    'product_uom': ref('uom.product_uom_unit'),
                    'date_planned': DateTime.today() + relativedelta(days=3)}),
                (0, 0, {
                    'product_id': ref('product.product_product_27'),
                    'name': obj().env.ref('product.product_product_27').partner_ref,
                    'price_unit': 99.00,
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
        <function model="purchase.order" name="write">
            <value eval="[ref('purchase_order_4')]"/>
            <value eval="{'state': 'sent'}"/>
        </function>

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
                    'price_unit': 130.5,
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

        <record id="purchase_order_8" model="purchase.order">
            <field name="partner_id" ref="base.res_partner_1"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="state">purchase</field>
            <field name="create_date" eval="DateTime.today() - relativedelta(days=20)"/>
            <field name="date_order" eval="DateTime.today() - relativedelta(days=5)"/>
            <field name="date_approve" eval="DateTime.today() - relativedelta(days=9)"/>
            <field name="order_line" model="purchase.order.line" eval="[(5, 0, 0),
                (0, 0, {
                    'product_id': ref('product.product_product_16'),
                    'name': 'Drawer Black',
                    'price_unit': 280.80,
                    'product_qty': 15.0,
                    'product_uom': ref('uom.product_uom_dozen'),
                    'date_planned': time.strftime('%Y-%m-%d')}),
                (0, 0, {
                    'product_id': ref('product.product_product_20'),
                    'name': 'Flipover',
                    'price_unit': 450.70,
                    'product_qty': 5.0,
                    'product_uom': ref('uom.product_uom_dozen'),
                    'date_planned': time.strftime('%Y-%m-%d')})
            ]"/>
        </record>

        <record id="purchase_order_9" model="purchase.order">
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="state">purchase</field>
            <field name="create_date" eval="DateTime.today() - relativedelta(days=20)"/>
            <field name="date_order" eval="DateTime.today() - relativedelta(days=15)"/>
            <field name="date_approve" eval="DateTime.today() - relativedelta(days=5)"/>
            <field name="order_line" model="purchase.order.line" eval="[(5, 0, 0),
                (0, 0, {
                    'product_id': ref('product.product_product_8'),
                    'name': 'Large Desk',
                    'price_unit': 500.00,
                    'product_qty': 20.0,
                    'product_uom': ref('uom.product_uom_dozen'),
                    'date_planned': time.strftime('%Y-%m-%d')}),
                (0, 0, {
                    'product_id': ref('product.product_product_5'),
                    'name': 'Corner Desk Right Sit',
                    'price_unit': 500.0,
                    'product_qty': 5.0,
                    'product_uom': ref('uom.product_uom_dozen'),
                    'date_planned': time.strftime('%Y-%m-%d')}),
            ]"/>
        </record>

        <record id="purchase_order_10" model="purchase.order">
            <field name="partner_id" ref="base.res_partner_12"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="state">purchase</field>
            <field name="create_date" eval="DateTime.today() - relativedelta(days=20)"/>
            <field name="date_order" eval="DateTime.today() - relativedelta(days=15)"/>
            <field name="date_approve" eval="DateTime.today() - relativedelta(days=18)"/>
            <field name="order_line" model="purchase.order.line" eval="[(5, 0, 0),
                (0, 0, {
                    'product_id': ref('product.product_product_12'),
                    'name': 'Office Chair Black',
                    'price_unit': 250.50,
                    'product_qty': 10.0,
                    'product_uom': ref('uom.product_uom_dozen'),
                    'date_planned': time.strftime('%Y-%m-%d')}),
            ]"/>
        </record>

        <record id="purchase_activity_1" model="mail.activity">
            <field name="res_id" ref="purchase.purchase_order_2"/>
            <field name="res_model_id" ref="purchase.model_purchase_order"/>
            <field name="activity_type_id" ref="mail.mail_activity_data_email"/>
            <field name="date_deadline" eval="DateTime.today().strftime('%Y-%m-%d %H:%M')"/>
            <field name="summary">Send specifications</field>
            <field name="create_uid" ref="base.user_admin"/>
            <field name="user_id" ref="base.user_admin"/>
        </record>
        <record id="purchase_activity_2" model="mail.activity">
            <field name="res_id" ref="purchase.purchase_order_5"/>
            <field name="res_model_id" ref="purchase.model_purchase_order"/>
            <field name="activity_type_id" ref="mail.mail_activity_data_todo"/>
            <field name="date_deadline" eval="DateTime.today().strftime('%Y-%m-%d %H:%M')"/>
            <field name="summary">Get approval</field>
            <field name="create_uid" ref="base.user_admin"/>
            <field name="user_id" ref="base.user_admin"/>
        </record>
        <record id="purchase_activity_3" model="mail.activity">
            <field name="res_id" ref="purchase.purchase_order_6"/>
            <field name="res_model_id" ref="purchase.model_purchase_order"/>
            <field name="activity_type_id" ref="mail.mail_activity_data_todo"/>
            <field name="date_deadline" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="summary">Check optional products</field>
            <field name="create_uid" ref="base.user_admin"/>
            <field name="user_id" ref="base.user_admin"/>
        </record>
        <record id="purchase_activity_4" model="mail.activity">
            <field name="res_id" ref="purchase.purchase_order_7"/>
            <field name="res_model_id" ref="purchase.model_purchase_order"/>
            <field name="activity_type_id" ref="mail.mail_activity_data_todo"/>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(days=5)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="summary">Check competitors</field>
            <field name="create_uid" ref="base.user_admin"/>
            <field name="user_id" ref="base.user_admin"/>
        </record>

    </data>
</odoo>


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
import difflib
import logging
import time
from markupsafe import Markup

from odoo import api, fields, models, Command, _

_logger = logging.getLogger(__name__)

TOLERANCE = 0.02  # tolerance applied to the total when searching for a matching purchase order


class AccountMove(models.Model):
    _inherit = 'account.move'

    purchase_vendor_bill_id = fields.Many2one('purchase.bill.union', store=False, readonly=False,
        string='Auto-complete',
        help="Auto-complete from a past bill / purchase order.")
    purchase_id = fields.Many2one('purchase.order', store=False, readonly=False,
        string='Purchase Order',
        help="Auto-complete from a past purchase order.")
    purchase_order_count = fields.Integer(compute="_compute_origin_po_count", string='Purchase Order Count')

    def _get_invoice_reference(self):
        self.ensure_one()
        vendor_refs = [ref for ref in set(self.invoice_line_ids.mapped('purchase_line_id.order_id.partner_ref')) if ref]
        if self.ref:
            return [ref for ref in self.ref.split(', ') if ref and ref not in vendor_refs] + vendor_refs
        return vendor_refs

    @api.onchange('purchase_vendor_bill_id', 'purchase_id')
    def _onchange_purchase_auto_complete(self):
        r''' Load from either an old purchase order, either an old vendor bill.

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

        # Copy data from PO
        invoice_vals = self.purchase_id.with_company(self.purchase_id.company_id)._prepare_invoice()
        has_invoice_lines = bool(self.invoice_line_ids.filtered(lambda x: x.display_type not in ('line_note', 'line_section')))
        new_currency_id = self.currency_id if has_invoice_lines else invoice_vals.get('currency_id')
        del invoice_vals['ref'], invoice_vals['payment_reference']
        del invoice_vals['company_id']  # avoid recomputing the currency
        if self.move_type == invoice_vals['move_type']:
            del invoice_vals['move_type'] # no need to be updated if it's same value, to avoid recomputes
        self.update(invoice_vals)
        self.currency_id = new_currency_id

        # Copy purchase lines.
        po_lines = self.purchase_id.order_line - self.invoice_line_ids.mapped('purchase_line_id')
        for line in po_lines.filtered(lambda l: not l.display_type):
            self.invoice_line_ids += self.env['account.move.line'].new(
                line._prepare_account_move_line(self)
            )

        # Compute invoice_origin.
        origins = set(self.invoice_line_ids.mapped('purchase_line_id.order_id.name'))
        self.invoice_origin = ','.join(list(origins))

        # Compute ref.
        refs = self._get_invoice_reference()
        self.ref = ', '.join(refs)

        # Compute payment_reference.
        if not self.payment_reference:
            if len(refs) == 1:
                self.payment_reference = refs[0]
            elif len(refs) > 1:
                self.payment_reference = refs[-1]

        # Copy company_id (only changes if the id is of a child company (branch))
        if self.company_id != self.purchase_id.company_id:
            self.company_id = self.purchase_id.company_id

        self.purchase_id = False

    @api.onchange('partner_id', 'company_id')
    def _onchange_partner_id(self):
        res = super(AccountMove, self)._onchange_partner_id()

        currency_id = (
                self.partner_id.property_purchase_currency_id
                or self.env['res.currency'].browse(self.env.context.get("default_currency_id"))
                or self.currency_id
        )

        if self.partner_id and self.move_type in ['in_invoice', 'in_refund'] and self.currency_id != currency_id:
            if not self.env.context.get('default_journal_id'):
                journal_domain = [
                    *self.env['account.journal']._check_company_domain(self.company_id),
                    ('type', '=', 'purchase'),
                    ('currency_id', '=', currency_id.id),
                ]
                default_journal_id = self.env['account.journal'].search(journal_domain, limit=1)
                if default_journal_id:
                    self.journal_id = default_journal_id

            self.currency_id = currency_id

        return res

    @api.depends('line_ids.purchase_line_id')
    def _compute_origin_po_count(self):
        for move in self:
            move.purchase_order_count = len(move.line_ids.purchase_line_id.order_id)

    def action_view_source_purchase_orders(self):
        self.ensure_one()
        source_orders = self.line_ids.purchase_line_id.order_id
        result = self.env['ir.actions.act_window']._for_xml_id('purchase.purchase_form_action')
        if len(source_orders) > 1:
            result['domain'] = [('id', 'in', source_orders.ids)]
        elif len(source_orders) == 1:
            result['views'] = [(self.env.ref('purchase.purchase_order_form', False).id, 'form')]
            result['res_id'] = source_orders.id
        else:
            result = {'type': 'ir.actions.act_window_close'}
        return result

    @api.model_create_multi
    def create(self, vals_list):
        # OVERRIDE
        moves = super(AccountMove, self).create(vals_list)
        for move in moves:
            if move.reversed_entry_id:
                continue
            purchases = move.line_ids.purchase_line_id.order_id
            if not purchases:
                continue
            refs = [purchase._get_html_link() for purchase in purchases]
            message = _("This vendor bill has been created from: ") + Markup(',').join(refs)
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
                refs = [purchase._get_html_link() for purchase in diff_purchases]
                message = _("This vendor bill has been modified from: ") + Markup(',').join(refs)
                move.message_post(body=message)
        return res

    def _find_matching_subset_po_lines(self, po_lines_with_amount, goal_total, timeout):
        """Finds the purchase order lines adding up to the goal amount.

        The problem of finding the subset of `po_lines_with_amount` which sums up to `goal_total` reduces to
        the 0-1 Knapsack problem. The dynamic programming approach to solve this problem is most of the time slower
        than this because identical sub-problems don't arise often enough. It returns the list of purchase order lines
        which sum up to `goal_total` or an empty list if multiple or no solutions were found.

        :param po_lines_with_amount: a dict (str: float|recordset) containing:
            * line: an `purchase.order.line`
            * amount_to_invoice: the remaining amount to be invoiced of the line
        :param goal_total: the total amount to match with a subset of purchase order lines
        :param timeout: the max time the line matching algorithm can take before timing out
        :return: list of `purchase.order.line` whose remaining sum matches `goal_total`
        """
        def find_matching_subset_po_lines(lines, goal):
            if time.time() - start_time > timeout:
                raise TimeoutError
            solutions = []
            for i, line in enumerate(lines):
                if line['amount_to_invoice'] < goal - TOLERANCE:
                    # The amount to invoice of the current purchase order line is less than the amount we still need on
                    # the vendor bill.
                    # We try finding purchase order lines that match the remaining vendor bill amount minus the amount
                    # to invoice of the current purchase order line. We only look in the purchase order lines that we
                    # haven't passed yet.
                    sub_solutions = find_matching_subset_po_lines(lines[i + 1:], goal - line['amount_to_invoice'])
                    # We add all possible sub-solutions' purchase order lines in a tuple together with our current
                    # purchase order line.
                    solutions.extend((line['line'], *solution) for solution in sub_solutions)
                elif goal - TOLERANCE <= line['amount_to_invoice'] <= goal + TOLERANCE:
                    # The amount to invoice of the current purchase order line matches the remaining vendor bill amount.
                    # We add this purchase order line to our list of solutions.
                    solutions.append([line['line']])
                if len(solutions) > 1:
                    # More than one solution was found. We can't know for sure which is the correct one, so we don't
                    # return any solution.
                    return []
            return solutions
        start_time = time.time()
        try:
            subsets = find_matching_subset_po_lines(
                sorted(po_lines_with_amount, key=lambda line: line['amount_to_invoice'], reverse=True),
                goal_total
            )
            return subsets[0] if subsets else []
        except TimeoutError:
            _logger.warning("Timed out during search of a matching subset of purchase order lines")
            return []

    def _find_matching_po_and_inv_lines(self, po_lines, inv_lines, timeout):
        """Finds purchase order lines that match some of the invoice lines.

        We try to find a purchase order line for every invoice line matching on the unit price and having at least
        the same quantity to invoice.

        :param po_lines: list of purchase order lines that can be matched
        :param inv_lines: list of invoice lines to be matched
        :param timeout: how long this function can run before we consider it too long
        :return: a tuple (list, list) containing:
            * matched 'purchase.order.line'
            * tuple of purchase order line ids and their matched 'account.move.line'
        """
        # Sort the invoice lines by unit price and quantity to speed up matching
        invoice_lines = sorted(inv_lines, key=lambda line: (line.price_unit, line.quantity), reverse=True)
        # Sort the purchase order lines by unit price and remaining quantity to speed up matching
        purchase_lines = sorted(
            po_lines,
            key=lambda line: (line.price_unit, line.product_qty - line.qty_invoiced),
            reverse=True
        )
        matched_po_lines = []
        matched_inv_lines = []
        try:
            start_time = time.time()
            for invoice_line in invoice_lines:
                # There are no purchase order lines left. We are done matching.
                if not purchase_lines:
                    break
                # A dict of purchase lines mapping to a diff score for the name
                purchase_line_candidates = {}
                for purchase_line in purchase_lines:
                    if time.time() - start_time > timeout:
                        raise TimeoutError

                    # The lists are sorted by unit price descendingly.
                    # When the unit price of the purchase line is lower than the unit price of the invoice line,
                    # we cannot get a match anymore.
                    if purchase_line.price_unit < invoice_line.price_unit:
                        break

                    if (invoice_line.price_unit == purchase_line.price_unit
                            and invoice_line.quantity <= purchase_line.product_qty - purchase_line.qty_invoiced):
                        # The current purchase line is a possible match for the current invoice line.
                        # We calculate the name match ratio and continue with other possible matches.
                        #
                        # We could match on more fields coming from an EDI invoice, but that requires extending the
                        # account.move.line model with the extra matching fields and extending the EDI extraction
                        # logic to fill these new fields.
                        purchase_line_candidates[purchase_line] = difflib.SequenceMatcher(
                            None, invoice_line.name, purchase_line.name).ratio()

                if len(purchase_line_candidates) > 0:
                    # We take the best match based on the name.
                    purchase_line_match = max(purchase_line_candidates, key=purchase_line_candidates.get)
                    if purchase_line_match:
                        # We found a match. We remove the purchase order line so it does not get matched twice.
                        purchase_lines.remove(purchase_line_match)
                        matched_po_lines.append(purchase_line_match)
                        matched_inv_lines.append((purchase_line_match.id, invoice_line))

            return (matched_po_lines, matched_inv_lines)

        except TimeoutError:
            _logger.warning('Timed out during search of matching purchase order lines')
            return ([], [])

    def _set_purchase_orders(self, purchase_orders, force_write=True):
        """Link the given purchase orders to this vendor bill and add their lines as invoice lines.

        :param purchase_orders: a list of purchase orders to be linked to this vendor bill
        :param force_write: whether to delete all existing invoice lines before adding the vendor bill lines
        """
        with self.env.cr.savepoint():
            with self._get_edi_creation() as invoice:
                if force_write and invoice.line_ids:
                    invoice.invoice_line_ids = [Command.clear()]
                for purchase_order in purchase_orders:
                    invoice.invoice_line_ids = [Command.create({
                        'display_type': 'line_section',
                        'name': _('From %s', purchase_order.name)
                    })]
                    invoice.purchase_id = purchase_order
                    invoice._onchange_purchase_auto_complete()

    def _match_purchase_orders(self, po_references, partner_id, amount_total, from_ocr, timeout):
        """Tries to match open purchase order lines with this invoice given the information we have.

        :param po_references: a list of potential purchase order references/names
        :param partner_id: the vendor id inferred from the vendor bill
        :param amount_total: the total amount of the vendor bill
        :param from_ocr: indicates whether this vendor bill was created from an OCR scan (less reliable)
        :param timeout: the max time the line matching algorithm can take before timing out
        :return: tuple (str, recordset, dict) containing:
            * the match method:
                * `total_match`: purchase order reference(s) and total amounts match perfectly
                * `subset_total_match`: a subset of the referenced purchase orders' lines matches the total amount of
                    this invoice (OCR only)
                * `po_match`: only the purchase order reference matches (OCR only)
                * `subset_match`: a subset of the referenced purchase orders' lines matches a subset of the invoice
                    lines based on unit prices (EDI only)
                * `no_match`: no result found
            * recordset of `purchase.order.line` containing purchase order lines matched with an invoice line
            * list of tuple containing every `purchase.order.line` id and its related `account.move.line`
        """

        common_domain = [
            ('company_id', '=', self.company_id.id),
            ('state', 'in', ('purchase', 'done')),
            ('invoice_status', 'in', ('to invoice', 'no'))
        ]

        matching_purchase_orders = self.env['purchase.order']

        # We have purchase order references in our vendor bill and a total amount.
        if po_references and amount_total:
            # We first try looking for purchase orders whose names match one of the purchase order references in the
            # vendor bill.
            matching_purchase_orders |= self.env['purchase.order'].search(
                common_domain + [('name', 'in', po_references)])

            if not matching_purchase_orders:
                # If not found, we try looking for purchase orders whose `partner_ref` field matches one of the
                # purchase order references in the vendor bill.
                matching_purchase_orders |= self.env['purchase.order'].search(
                    common_domain + [('partner_ref', 'in', po_references)])

            if matching_purchase_orders:
                # We found matching purchase orders and are extracting all purchase order lines together with their
                # amounts still to be invoiced.
                po_lines = [line for line in matching_purchase_orders.order_line if line.product_qty]
                po_lines_with_amount = [{
                    'line': line,
                    'amount_to_invoice': (1 - line.qty_invoiced / line.product_qty) * line.price_total,
                } for line in po_lines]

                # If the sum of all remaining amounts to be invoiced for these purchase orders' lines is within a
                # tolerance from the vendor bill total, we have a total match. We return all purchase order lines
                # summing up to this vendor bill's total (could be from multiple purchase orders).
                if (amount_total - TOLERANCE
                        < sum(line['amount_to_invoice'] for line in po_lines_with_amount)
                        < amount_total + TOLERANCE):
                    return 'total_match', matching_purchase_orders.order_line, None

                elif from_ocr:
                    # The invoice comes from an OCR scan.
                    # We try to match the invoice total with purchase order lines.
                    matching_po_lines = self._find_matching_subset_po_lines(
                        po_lines_with_amount, amount_total, timeout)
                    if matching_po_lines:
                        return 'subset_total_match', self.env['purchase.order.line'].union(*matching_po_lines), None
                    else:
                        # We did not find a match for the invoice total.
                        # We return all purchase order lines based only on the purchase order reference(s) in the
                        # vendor bill.
                        return 'po_match', matching_purchase_orders.order_line, None

                else:
                    # We have an invoice from an EDI document, so we try to match individual invoice lines with
                    # individual purchase order lines from referenced purchase orders.
                    matching_po_lines, matching_inv_lines = self._find_matching_po_and_inv_lines(
                        po_lines, self.invoice_line_ids, timeout)

                    if matching_po_lines:
                        # We found a subset of purchase order lines that match a subset of the vendor bill lines.
                        # We return the matching purchase order lines and vendor bill lines.
                        return ('subset_match',
                                self.env['purchase.order.line'].union(*matching_po_lines),
                                matching_inv_lines)

        # As a last resort we try matching a purchase order by vendor and total amount.
        if partner_id and amount_total:
            purchase_id_domain = common_domain + [
                ('partner_id', 'child_of', [partner_id]),
                ('amount_total', '>=', amount_total - TOLERANCE),
                ('amount_total', '<=', amount_total + TOLERANCE)
            ]
            matching_purchase_orders = self.env['purchase.order'].search(purchase_id_domain)
            if len(matching_purchase_orders) == 1:
                # We found exactly one match on vendor and total amount (within tolerance).
                # We return all purchase order lines of the purchase order whose total amount matched our vendor bill.
                return 'total_match', matching_purchase_orders.order_line, None

        # We couldn't find anything, so we return no lines.
        return ('no_match', matching_purchase_orders.order_line, None)

    def _find_and_set_purchase_orders(self, po_references, partner_id, amount_total, from_ocr=False, timeout=10):
        """Finds related purchase orders that (partially) match the vendor bill and links the matching lines on this
        vendor bill.

        :param po_references: a list of potential purchase order references/names
        :param partner_id: the vendor id matched on the vendor bill
        :param amount_total: the total amount of the vendor bill
        :param from_ocr: indicates whether this vendor bill was created from an OCR scan (less reliable)
        :param timeout: the max time the line matching algorithm can take before timing out
        """
        self.ensure_one()

        method, matched_po_lines, matched_inv_lines = self._match_purchase_orders(
            po_references, partner_id, amount_total, from_ocr, timeout
        )

        if method in ('total_match', 'po_match'):
            # The purchase order reference(s) and total amounts match perfectly or there is only one purchase order
            # reference that matches with an OCR invoice. We replace the invoice lines with the purchase order lines.
            self._set_purchase_orders(matched_po_lines.order_id, force_write=True)

        elif method == 'subset_total_match':
            # A subset of the referenced purchase order lines matches the total amount of this invoice.
            # We keep the invoice lines, but add all the lines from the partially matched purchase orders:
            #   * "naively" matched purchase order lines keep their quantity
            #   * unmatched purchase order lines are added with their quantity set to 0
            self._set_purchase_orders(matched_po_lines.order_id, force_write=False)

            with self._get_edi_creation() as invoice:
                unmatched_lines = invoice.invoice_line_ids.filtered(
                    lambda l: l.purchase_line_id and l.purchase_line_id not in matched_po_lines)
                invoice.invoice_line_ids = [Command.update(line.id, {'quantity': 0}) for line in unmatched_lines]

        elif method == 'subset_match':
            # A subset of the referenced purchase order lines matches a subset of the invoice lines.
            # We add the purchase order lines, but adjust the quantity to the quantities in the invoice.
            # The original invoice lines that correspond with a purchase order line are removed.
            self._set_purchase_orders(matched_po_lines.order_id, force_write=False)

            with self._get_edi_creation() as invoice:
                unmatched_lines = invoice.invoice_line_ids.filtered(
                    lambda l: l.purchase_line_id and l.purchase_line_id not in matched_po_lines)
                invoice.invoice_line_ids = [Command.delete(line.id) for line in unmatched_lines]

                # We remove the original matched invoice lines and apply their quantities and taxes to the matched
                # purchase order lines.
                inv_and_po_lines = list(map(lambda line: (
                        invoice.invoice_line_ids.filtered(
                            lambda l: l.purchase_line_id and l.purchase_line_id.id == line[0]),
                        invoice.invoice_line_ids.filtered(
                            lambda l: l in line[1])
                    ),
                    matched_inv_lines
                ))
                invoice.invoice_line_ids = [
                    Command.update(po_line.id, {'quantity': inv_line.quantity, 'tax_ids': inv_line.tax_ids})
                    for po_line, inv_line in inv_and_po_lines
                ]
                invoice.invoice_line_ids = [Command.delete(inv_line.id) for dummy, inv_line in inv_and_po_lines]

                # If there are lines left not linked to a purchase order, we add a header
                unmatched_lines = invoice.invoice_line_ids.filtered(lambda l: not l.purchase_line_id)
                if len(unmatched_lines) > 0:
                    invoice.invoice_line_ids = [Command.create({
                        'display_type': 'line_section',
                        'name': _('From Electronic Document'),
                        'sequence': -1,
                    })]



class AccountMoveLine(models.Model):
    """ Override AccountInvoice_line to add the link to the purchase order line it is related to"""
    _inherit = 'account.move.line'

    purchase_line_id = fields.Many2one('purchase.order.line', 'Purchase Order Line', ondelete='set null', index='btree_not_null')
    purchase_order_id = fields.Many2one('purchase.order', 'Purchase Order', related='purchase_line_id.order_id', readonly=True)

    def _copy_data_extend_business_fields(self, values):
        # OVERRIDE to copy the 'purchase_line_id' field as well.
        super(AccountMoveLine, self)._copy_data_extend_business_fields(values)
        values['purchase_line_id'] = self.purchase_line_id.id

```

## File: models\account_tax.py

```python
# -*- coding: utf-8 -*-

from odoo import models


class AccountTax(models.Model):
    _inherit = "account.tax"

    def _hook_compute_is_used(self, taxes_to_compute):
        # OVERRIDE in order to fetch taxes used in purchase

        used_taxes = super()._hook_compute_is_used(taxes_to_compute)
        taxes_to_compute -= used_taxes

        if taxes_to_compute:
            self.env['purchase.order.line'].flush_model(['taxes_id'])
            self.env.cr.execute("""
                SELECT id
                FROM account_tax
                WHERE EXISTS(
                    SELECT 1
                    FROM account_tax_purchase_order_line_rel AS pur
                    WHERE account_tax_id IN %s
                    AND account_tax.id = pur.account_tax_id
                )
            """, [tuple(taxes_to_compute)])

            used_taxes.update([tax[0] for tax in self.env.cr.fetchall()])

        return used_taxes

```

## File: models\analytic_account.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _


class AccountAnalyticAccount(models.Model):
    _inherit = 'account.analytic.account'

    purchase_order_count = fields.Integer("Purchase Order Count", compute='_compute_purchase_order_count')

    @api.depends('line_ids')
    def _compute_purchase_order_count(self):
        for account in self:
            account.purchase_order_count = self.env['purchase.order'].search_count([
                ('order_line.invoice_lines.analytic_line_ids.account_id', '=', account.id)
            ])

    def action_view_purchase_orders(self):
        self.ensure_one()
        purchase_orders = self.env['purchase.order'].search([
            ('order_line.invoice_lines.analytic_line_ids.account_id', '=', self.id)
        ])
        result = {
            "type": "ir.actions.act_window",
            "res_model": "purchase.order",
            "domain": [['id', 'in', purchase_orders.ids]],
            "name": _("Purchase Orders"),
            'view_mode': 'tree,form',
        }
        if len(purchase_orders) == 1:
            result['view_mode'] = 'form'
            result['res_id'] = purchase_orders.id
        return result

```

## File: models\analytic_applicability.py

```python
# -*- coding: utf-8 -*-

from odoo import fields, models


class AccountAnalyticApplicability(models.Model):
    _inherit = 'account.analytic.applicability'
    _description = "Analytic Plan's Applicabilities"

    business_domain = fields.Selection(
        selection_add=[
            ('purchase_order', 'Purchase Order'),
        ],
        ondelete={'purchase_order': 'cascade'},
    )

```

## File: models\product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.addons.base.models.res_partner import WARNING_MESSAGE, WARNING_HELP
from odoo.tools.float_utils import float_round
from dateutil.relativedelta import relativedelta


class ProductTemplate(models.Model):
    _name = 'product.template'
    _inherit = 'product.template'

    purchased_product_qty = fields.Float(compute='_compute_purchased_product_qty', string='Purchased', digits='Product Unit of Measure')
    purchase_method = fields.Selection([
        ('purchase', 'On ordered quantities'),
        ('receive', 'On received quantities'),
    ], string="Control Policy", compute='_compute_purchase_method', default='receive', store=True, readonly=False,
        help="On ordered quantities: Control bills based on ordered quantities.\n"
            "On received quantities: Control bills based on received quantities.")
    purchase_line_warn = fields.Selection(WARNING_MESSAGE, 'Purchase Order Line Warning', help=WARNING_HELP, required=True, default="no-message")
    purchase_line_warn_msg = fields.Text('Message for Purchase Order Line')

    @api.depends('detailed_type')
    def _compute_purchase_method(self):
        default_purchase_method = self.env['product.template'].default_get(['purchase_method']).get('purchase_method')
        for product in self:
            if product.detailed_type == 'service':
                product.purchase_method = 'purchase'
            else:
                product.purchase_method = default_purchase_method

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
        action = self.env["ir.actions.actions"]._for_xml_id("purchase.action_purchase_history")
        action['domain'] = ['&', ('state', 'in', ['purchase', 'done']), ('product_id', 'in', self.product_variant_ids.ids)]
        action['display_name'] = _("Purchase History for %s", self.display_name)
        return action


class ProductProduct(models.Model):
    _name = 'product.product'
    _inherit = 'product.product'

    purchased_product_qty = fields.Float(compute='_compute_purchased_product_qty', string='Purchased',
        digits='Product Unit of Measure')

    def _compute_purchased_product_qty(self):
        date_from = fields.Datetime.to_string(fields.Date.context_today(self) - relativedelta(years=1))
        domain = [
            ('order_id.state', 'in', ['purchase', 'done']),
            ('product_id', 'in', self.ids),
            ('order_id.date_approve', '>=', date_from)
        ]
        order_lines = self.env['purchase.order.line']._read_group(domain, ['product_id'], ['product_uom_qty:sum'])
        purchased_data = {product.id: qty for product, qty in order_lines}
        for product in self:
            if not product.id:
                product.purchased_product_qty = 0.0
                continue
            product.purchased_product_qty = float_round(purchased_data.get(product.id, 0), precision_rounding=product.uom_id.rounding)

    def action_view_po(self):
        action = self.env["ir.actions.actions"]._for_xml_id("purchase.action_purchase_history")
        action['domain'] = ['&', ('state', 'in', ['purchase', 'done']), ('product_id', 'in', self.ids)]
        action['display_name'] = _("Purchase History for %s", self.display_name)
        return action


class ProductSupplierinfo(models.Model):
    _inherit = "product.supplierinfo"

    @api.onchange('partner_id')
    def _onchange_partner_id(self):
        self.currency_id = self.partner_id.property_purchase_currency_id.id or self.env.company.currency_id.id

    def _get_filtered_supplier(self, company_id, product_id, params):
        if params and 'order_id' in params and params['order_id'].company_id:
            company_id = params['order_id'].company_id
        return super()._get_filtered_supplier(company_id, product_id, params)

class ProductPackaging(models.Model):
    _inherit = 'product.packaging'

    purchase = fields.Boolean("Purchase", default=True, help="If true, the packaging can be used for purchase orders")

```

## File: models\purchase_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from collections import defaultdict
from datetime import datetime
from dateutil.relativedelta import relativedelta
from pytz import timezone

from markupsafe import escape, Markup
from werkzeug.urls import url_encode

from odoo import api, Command, fields, models, _
from odoo.osv import expression
from odoo.tools import format_amount, format_date, formatLang, groupby
from odoo.tools.float_utils import float_is_zero
from odoo.exceptions import UserError, ValidationError


class PurchaseOrder(models.Model):
    _name = "purchase.order"
    _inherit = ['portal.mixin', 'product.catalog.mixin', 'mail.thread', 'mail.activity.mixin']
    _description = "Purchase Order"
    _rec_names_search = ['name', 'partner_ref']
    _order = 'priority desc, id desc'

    @api.depends('order_line.price_total')
    def _amount_all(self):
        for order in self:
            order_lines = order.order_line.filtered(lambda x: not x.display_type)

            if order.company_id.tax_calculation_rounding_method == 'round_globally':
                tax_results = self.env['account.tax']._compute_taxes([
                    line._convert_to_tax_base_line_dict()
                    for line in order_lines
                ])
                totals = tax_results['totals']
                amount_untaxed = totals.get(order.currency_id, {}).get('amount_untaxed', 0.0)
                amount_tax = totals.get(order.currency_id, {}).get('amount_tax', 0.0)
            else:
                amount_untaxed = sum(order_lines.mapped('price_subtotal'))
                amount_tax = sum(order_lines.mapped('price_tax'))

            if order.currency_id.compare_amounts(order.amount_untaxed, amount_untaxed):
                order.amount_untaxed = amount_untaxed
            order.amount_tax = amount_tax
            order.amount_total = order.amount_untaxed + order.amount_tax

    @api.depends('state', 'order_line.qty_to_invoice')
    def _get_invoiced(self):
        precision = self.env['decimal.precision'].precision_get('Product Unit of Measure')
        for order in self:
            if order.state not in ('purchase', 'done'):
                order.invoice_status = 'no'
                continue

            if any(
                not float_is_zero(line.qty_to_invoice, precision_digits=precision)
                for line in order.order_line.filtered(lambda l: not l.display_type)
            ):
                order.invoice_status = 'to invoice'
            elif (
                all(
                    float_is_zero(line.qty_to_invoice, precision_digits=precision)
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

    name = fields.Char('Order Reference', required=True, index='trigram', copy=False, default='New')
    priority = fields.Selection(
        [('0', 'Normal'), ('1', 'Urgent')], 'Priority', default='0', index=True)
    origin = fields.Char('Source Document', copy=False,
        help="Reference of the document that generated this purchase order "
             "request (e.g. a sales order)")
    partner_ref = fields.Char('Vendor Reference', copy=False,
        help="Reference of the sales order or bid sent by the vendor. "
             "It's used to do the matching when you receive the "
             "products as this reference is usually written on the "
             "delivery order sent by your vendor.")
    date_order = fields.Datetime('Order Deadline', required=True, index=True, copy=False, default=fields.Datetime.now,
        help="Depicts the date within which the Quotation should be confirmed and converted into a purchase order.")
    date_approve = fields.Datetime('Confirmation Date', readonly=True, index=True, copy=False)
    partner_id = fields.Many2one('res.partner', string='Vendor', required=True, change_default=True, tracking=True, check_company=True, help="You can find a vendor by its Name, TIN, Email or Internal Reference.")
    dest_address_id = fields.Many2one('res.partner', check_company=True, string='Dropship Address',
        help="Put an address if you want to deliver directly from the vendor to the customer. "
             "Otherwise, keep empty to deliver to your own company.")
    currency_id = fields.Many2one('res.currency', 'Currency', required=True,
        default=lambda self: self.env.company.currency_id.id)
    state = fields.Selection([
        ('draft', 'RFQ'),
        ('sent', 'RFQ Sent'),
        ('to approve', 'To Approve'),
        ('purchase', 'Purchase Order'),
        ('done', 'Locked'),
        ('cancel', 'Cancelled')
    ], string='Status', readonly=True, index=True, copy=False, default='draft', tracking=True)
    order_line = fields.One2many('purchase.order.line', 'order_id', string='Order Lines', copy=True)
    notes = fields.Html('Terms and Conditions')

    invoice_count = fields.Integer(compute="_compute_invoice", string='Bill Count', copy=False, default=0, store=True)
    invoice_ids = fields.Many2many('account.move', compute="_compute_invoice", string='Bills', copy=False, store=True)
    invoice_status = fields.Selection([
        ('no', 'Nothing to Bill'),
        ('to invoice', 'Waiting Bills'),
        ('invoiced', 'Fully Billed'),
    ], string='Billing Status', compute='_get_invoiced', store=True, readonly=True, copy=False, default='no')
    date_planned = fields.Datetime(
        string='Expected Arrival', index=True, copy=False, compute='_compute_date_planned', store=True, readonly=False,
        help="Delivery date promised by vendor. This date is used to determine expected arrival of products.")
    date_calendar_start = fields.Datetime(compute='_compute_date_calendar_start', readonly=True, store=True)

    amount_untaxed = fields.Monetary(string='Untaxed Amount', store=True, readonly=True, compute='_amount_all', tracking=True)
    tax_totals = fields.Binary(compute='_compute_tax_totals', exportable=False)
    amount_tax = fields.Monetary(string='Taxes', store=True, readonly=True, compute='_amount_all')
    amount_total = fields.Monetary(string='Total', store=True, readonly=True, compute='_amount_all')

    fiscal_position_id = fields.Many2one('account.fiscal.position', string='Fiscal Position', domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    tax_country_id = fields.Many2one(
        comodel_name='res.country',
        compute='_compute_tax_country_id',
        # Avoid access error on fiscal position, when reading a purchase order with company != user.company_ids
        compute_sudo=True,
        help="Technical field to filter the available taxes depending on the fiscal country and fiscal position.")
    tax_calculation_rounding_method = fields.Selection(
        related='company_id.tax_calculation_rounding_method',
        string='Tax calculation rounding method', readonly=True)
    payment_term_id = fields.Many2one('account.payment.term', 'Payment Terms', domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    incoterm_id = fields.Many2one('account.incoterms', 'Incoterm', help="International Commercial Terms are a series of predefined commercial terms used in international transactions.")

    product_id = fields.Many2one('product.product', related='order_line.product_id', string='Product')
    user_id = fields.Many2one(
        'res.users', string='Buyer', index=True, tracking=True,
        default=lambda self: self.env.user, check_company=True)
    company_id = fields.Many2one('res.company', 'Company', required=True, index=True, default=lambda self: self.env.company.id)
    country_code = fields.Char(related='company_id.account_fiscal_country_id.code', string="Country code")
    currency_rate = fields.Float("Currency Rate", compute='_compute_currency_rate', compute_sudo=True, store=True, readonly=True, help='Ratio between the purchase order currency and the company currency')

    mail_reminder_confirmed = fields.Boolean("Reminder Confirmed", default=False, readonly=True, copy=False, help="True if the reminder email is confirmed by the vendor.")
    mail_reception_confirmed = fields.Boolean("Reception Confirmed", default=False, readonly=True, copy=False, help="True if PO reception is confirmed by the vendor.")

    receipt_reminder_email = fields.Boolean('Receipt Reminder Email', compute='_compute_receipt_reminder_email')
    reminder_date_before_receipt = fields.Integer('Days Before Receipt', compute='_compute_receipt_reminder_email')

    @api.constrains('company_id', 'order_line')
    def _check_order_line_company_id(self):
        for order in self:
            invalid_companies = order.order_line.product_id.company_id.filtered(
                lambda c: order.company_id not in c._accessible_branches()
            )
            if invalid_companies:
                bad_products = order.order_line.product_id.filtered(
                    lambda p: p.company_id and p.company_id in invalid_companies
                )
                raise ValidationError(_(
                    "Your quotation contains products from company %(product_company)s whereas your quotation belongs to company %(quote_company)s. \n Please change the company of your quotation or remove the products from other companies (%(bad_products)s).",
                    product_company=', '.join(invalid_companies.sudo().mapped('display_name')),
                    quote_company=order.company_id.display_name,
                    bad_products=', '.join(bad_products.mapped('display_name')),
                ))

    def _compute_access_url(self):
        super(PurchaseOrder, self)._compute_access_url()
        for order in self:
            order.access_url = '/my/purchase/%s' % (order.id)

    @api.depends('state', 'date_order', 'date_approve')
    def _compute_date_calendar_start(self):
        for order in self:
            order.date_calendar_start = order.date_approve if (order.state in ['purchase', 'done']) else order.date_order

    @api.depends('date_order', 'currency_id', 'company_id', 'company_id.currency_id')
    def _compute_currency_rate(self):
        for order in self:
            order.currency_rate = self.env['res.currency']._get_conversion_rate(order.company_id.currency_id, order.currency_id, order.company_id, order.date_order)

    @api.depends('order_line.date_planned')
    def _compute_date_planned(self):
        """ date_planned = the earliest date_planned across all order lines. """
        for order in self:
            dates_list = order.order_line.filtered(lambda x: not x.display_type and x.date_planned).mapped('date_planned')
            if dates_list:
                order.date_planned = min(dates_list)
            else:
                order.date_planned = False

    @api.depends('name', 'partner_ref', 'amount_total', 'currency_id')
    @api.depends_context('show_total_amount')
    def _compute_display_name(self):
        for po in self:
            name = po.name
            if po.partner_ref:
                name += ' (' + po.partner_ref + ')'
            if self.env.context.get('show_total_amount') and po.amount_total:
                name += ': ' + formatLang(self.env, po.amount_total, currency_obj=po.currency_id)
            po.display_name = name

    @api.depends('company_id', 'partner_id')
    def _compute_receipt_reminder_email(self):
        for order in self:
            order.receipt_reminder_email = order.partner_id.with_company(order.company_id).receipt_reminder_email
            order.reminder_date_before_receipt = order.partner_id.with_company(order.company_id).reminder_date_before_receipt

    @api.depends_context('lang')
    @api.depends('order_line.taxes_id', 'order_line.price_subtotal', 'amount_total', 'amount_untaxed')
    def _compute_tax_totals(self):
        for order in self:
            order_lines = order.order_line.filtered(lambda x: not x.display_type)
            order.tax_totals = self.env['account.tax']._prepare_tax_totals(
                [x._convert_to_tax_base_line_dict() for x in order_lines],
                order.currency_id or order.company_id.currency_id,
            )

    @api.depends('company_id.account_fiscal_country_id', 'fiscal_position_id.country_id', 'fiscal_position_id.foreign_vat')
    def _compute_tax_country_id(self):
        for record in self:
            if record.fiscal_position_id.foreign_vat:
                record.tax_country_id = record.fiscal_position_id.country_id
            else:
                record.tax_country_id = record.company_id.account_fiscal_country_id

    @api.onchange('date_planned')
    def onchange_date_planned(self):
        if self.date_planned:
            self.order_line.filtered(lambda line: not line.display_type).date_planned = self.date_planned

    def write(self, vals):
        vals, partner_vals = self._write_partner_values(vals)
        res = super().write(vals)
        if partner_vals:
            self.partner_id.sudo().write(partner_vals)  # Because the purchase user doesn't have write on `res.partner`
        return res

    @api.model_create_multi
    def create(self, vals_list):
        orders = self.browse()
        partner_vals_list = []
        for vals in vals_list:
            company_id = vals.get('company_id', self.default_get(['company_id'])['company_id'])
            # Ensures default picking type and currency are taken from the right company.
            self_comp = self.with_company(company_id)
            if vals.get('name', 'New') == 'New':
                seq_date = None
                if 'date_order' in vals:
                    seq_date = fields.Datetime.context_timestamp(self, fields.Datetime.to_datetime(vals['date_order']))
                vals['name'] = self_comp.env['ir.sequence'].next_by_code('purchase.order', sequence_date=seq_date) or '/'
            vals, partner_vals = self._write_partner_values(vals)
            partner_vals_list.append(partner_vals)
            orders |= super(PurchaseOrder, self_comp).create(vals)
        for order, partner_vals in zip(orders, partner_vals_list):
            if partner_vals:
                order.sudo().write(partner_vals)  # Because the purchase user doesn't have write on `res.partner`
        return orders

    @api.ondelete(at_uninstall=False)
    def _unlink_if_cancelled(self):
        for order in self:
            if not order.state == 'cancel':
                raise UserError(_('In order to delete a purchase order, you must cancel it first.'))

    def copy(self, default=None):
        ctx = dict(self.env.context)
        ctx.pop('default_product_id', None)
        self = self.with_context(ctx)
        new_po = super(PurchaseOrder, self).copy(default=default)
        for line in new_po.order_line:
            if line.product_id:
                seller = line.product_id._select_seller(
                    partner_id=line.partner_id, quantity=line.product_qty,
                    date=line.order_id.date_order and line.order_id.date_order.date(), uom_id=line.product_uom)
                line.date_planned = line._get_date_planned(seller)
        return new_po

    def _must_delete_date_planned(self, field_name):
        # To be overridden
        return field_name == 'order_line'

    def onchange(self, values, field_names, fields_spec):
        """
        Override onchange to NOT update all date_planned on PO lines when
        date_planned on PO is updated by the change of date_planned on PO lines.
        """
        result = super().onchange(values, field_names, fields_spec)
        if any(self._must_delete_date_planned(field) for field in field_names) and 'value' in result:
            for line in result['value'].get('order_line', []):
                if line[0] == Command.UPDATE and 'date_planned' in line[2]:
                    del line[2]['date_planned']
        return result

    def _get_report_base_filename(self):
        self.ensure_one()
        return 'Purchase Order-%s' % (self.name)

    @api.onchange('partner_id', 'company_id')
    def onchange_partner_id(self):
        # Ensures all properties and fiscal positions
        # are taken with the company of the order
        # if not defined, with_company doesn't change anything.
        self = self.with_company(self.company_id)
        default_currency = self._context.get("default_currency_id")
        if not self.partner_id:
            self.fiscal_position_id = False
            self.currency_id = default_currency or self.env.company.currency_id.id
        else:
            self.fiscal_position_id = self.env['account.fiscal.position']._get_fiscal_position(self.partner_id)
            self.payment_term_id = self.partner_id.property_supplier_payment_term_id.id
            self.currency_id = default_currency or self.partner_id.property_purchase_currency_id.id or self.env.company.currency_id.id
            if self.partner_id.buyer_id:
                self.user_id = self.partner_id.buyer_id
        return {}

    @api.onchange('fiscal_position_id', 'company_id')
    def _compute_tax_id(self):
        """
        Trigger the recompute of the taxes if the fiscal position is changed on the PO.
        """
        self.order_line._compute_tax_id()

    @api.onchange('partner_id')
    def onchange_partner_id_warning(self):
        if not self.partner_id or not self.env.user.has_group('purchase.group_warning_purchase'):
            return

        partner = self.partner_id

        # If partner has no warning, check its company
        if partner.purchase_warn == 'no-message' and partner.parent_id:
            partner = partner.parent_id

        if partner.purchase_warn and partner.purchase_warn != 'no-message':
            # Block if partner only has warning but parent company is blocked
            if partner.purchase_warn != 'block' and partner.parent_id and partner.parent_id.purchase_warn == 'block':
                partner = partner.parent_id
            title = _("Warning for %s", partner.name)
            message = partner.purchase_warn_msg
            warning = {
                'title': title,
                'message': message
            }
            if partner.purchase_warn == 'block':
                self.update({'partner_id': False})
            return {'warning': warning}
        return {}

    # ------------------------------------------------------------
    # MAIL.THREAD
    # ------------------------------------------------------------

    @api.returns('mail.message', lambda value: value.id)
    def message_post(self, **kwargs):
        if self.env.context.get('mark_rfq_as_sent'):
            self.filtered(lambda o: o.state == 'draft').write({'state': 'sent'})
        po_ctx = {'mail_post_autofollow': self.env.context.get('mail_post_autofollow', True)}
        if self.env.context.get('mark_rfq_as_sent') and 'notify_author' not in kwargs:
            kwargs['notify_author'] = self.env.user.partner_id.id in (kwargs.get('partner_ids') or [])
        return super(PurchaseOrder, self.with_context(**po_ctx)).message_post(**kwargs)

    def _notify_get_recipients_groups(self, message, model_description, msg_vals=None):
        """ Tweak 'view document' button for portal customers, calling directly
        routes for confirm specific to PO model. """
        groups = super()._notify_get_recipients_groups(
            message, model_description, msg_vals=msg_vals
        )
        if not self:
            return groups

        self.ensure_one()
        try:
            customer_portal_group = next(group for group in groups if group[0] == 'portal_customer')
        except StopIteration:
            pass
        else:
            access_opt = customer_portal_group[2].setdefault('button_access', {})
            if self.env.context.get('is_reminder'):
                access_opt['title'] = _('View')
                actions = customer_portal_group[2].setdefault('actions', list())
                actions.extend([
                    {'url': self.get_confirm_url(confirm_type='reminder'), 'title': _('Accept')},
                    {'url': self.get_update_url(), 'title': _('Update Dates')},
                ])
            else:
                access_opt['title'] = _('Confirm')
                access_opt['url'] = self.get_confirm_url(confirm_type='reception')

        return groups

    def _notify_by_email_prepare_rendering_context(self, message, msg_vals=False, model_description=False,
                                                   force_email_company=False, force_email_lang=False):
        render_context = super()._notify_by_email_prepare_rendering_context(
            message, msg_vals, model_description=model_description,
            force_email_company=force_email_company, force_email_lang=force_email_lang
        )
        subtitles = [render_context['record'].name]
        # don't show price on RFQ mail
        if self.state not in ['draft', 'sent']:
            if self.date_order:
                subtitles.append(_('%(amount)s due\N{NO-BREAK SPACE}%(date)s',
                                   amount=format_amount(self.env, self.amount_total, self.currency_id, lang_code=render_context.get('lang')),
                                   date=format_date(self.env, self.date_order, date_format='short', lang_code=render_context.get('lang'))
                                   ))
            else:
                subtitles.append(format_amount(self.env, self.amount_total, self.currency_id, lang_code=render_context.get('lang')))
        render_context['subtitles'] = subtitles
        return render_context

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
        elif 'state' in init_values and self.state == 'sent':
            return self.env.ref('purchase.mt_rfq_sent')
        return super(PurchaseOrder, self)._track_subtype(init_values)

    # ------------------------------------------------------------
    # ACTIONS
    # ------------------------------------------------------------

    def action_rfq_send(self):
        '''
        This function opens a window to compose an email, with the edi purchase template message loaded by default
        '''
        self.ensure_one()
        ir_model_data = self.env['ir.model.data']
        try:
            if self.env.context.get('send_rfq', False):
                template_id = ir_model_data._xmlid_lookup('purchase.email_template_edi_purchase')[1]
            else:
                template_id = ir_model_data._xmlid_lookup('purchase.email_template_edi_purchase_done')[1]
        except ValueError:
            template_id = False
        try:
            compose_form_id = ir_model_data._xmlid_lookup('mail.email_compose_message_wizard_form')[1]
        except ValueError:
            compose_form_id = False
        ctx = dict(self.env.context or {})
        ctx.update({
            'default_model': 'purchase.order',
            'default_res_ids': self.ids,
            'default_template_id': template_id,
            'default_composition_mode': 'comment',
            'default_email_layout_xmlid': "mail.mail_notification_layout_with_responsible_signature",
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
                lang = template._render_lang([ctx['default_res_id']])[ctx['default_res_id']]

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

    def print_quotation(self):
        self.write({'state': "sent"})
        return self.env.ref('purchase.report_purchase_quotation').report_action(self)

    def button_approve(self, force=False):
        self = self.filtered(lambda order: order._approval_allowed())
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
            order.order_line._validate_analytic_distribution()
            order._add_supplier_to_product()
            # Deal with double validation process
            if order._approval_allowed():
                order.button_approve()
            else:
                order.write({'state': 'to approve'})
            if order.partner_id not in order.message_partner_ids:
                order.message_subscribe([order.partner_id.id])
        return True

    def button_cancel(self):
        for order in self:
            for inv in order.invoice_ids:
                if inv and inv.state not in ('cancel', 'draft'):
                    raise UserError(_("Unable to cancel this purchase order. You must first cancel the related vendor bills."))

        self.write({'state': 'cancel', 'mail_reminder_confirmed': False})

    def button_unlock(self):
        self.write({'state': 'purchase'})

    def button_done(self):
        self.write({'state': 'done', 'priority': '0'})

    def _prepare_supplier_info(self, partner, line, price, currency):
        # Prepare supplierinfo data when adding a product
        return {
            'partner_id': partner.id,
            'sequence': max(line.product_id.seller_ids.mapped('sequence')) + 1 if line.product_id.seller_ids else 1,
            'min_qty': 1.0,
            'price': price,
            'currency_id': currency.id,
            'discount': line.discount,
            'delay': 0,
        }

    def _add_supplier_to_product(self):
        # Add the partner in the supplier list of the product if the supplier is not registered for
        # this product. We limit to 10 the number of suppliers for a product to avoid the mess that
        # could be caused for some generic products ("Miscellaneous").
        for line in self.order_line:
            # Do not add a contact as a supplier
            partner = self.partner_id if not self.partner_id.parent_id else self.partner_id.parent_id
            already_seller = (partner | self.partner_id) & line.product_id.seller_ids.mapped('partner_id')
            if line.product_id and not already_seller and len(line.product_id.seller_ids) <= 10:
                # Convert the price in the right currency.
                currency = partner.property_purchase_currency_id or self.env.company.currency_id
                price = self.currency_id._convert(line.price_unit, currency, line.company_id, line.date_order or fields.Date.today(), round=False)
                # Compute the price for the template's UoM, because the supplier's UoM is related to that UoM.
                if line.product_id.product_tmpl_id.uom_po_id != line.product_uom:
                    default_uom = line.product_id.product_tmpl_id.uom_po_id
                    price = line.product_uom._compute_price(price, default_uom)

                supplierinfo = self._prepare_supplier_info(partner, line, price, currency)
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
                # supplier info should be added regardless of the user access rights
                line.product_id.product_tmpl_id.sudo().write(vals)

    def action_create_invoice(self):
        """Create the invoice associated to the PO.
        """
        precision = self.env['decimal.precision'].precision_get('Product Unit of Measure')

        # 1) Prepare invoice vals and clean-up the section lines
        invoice_vals_list = []
        sequence = 10
        for order in self:
            if order.invoice_status != 'to invoice':
                continue

            order = order.with_company(order.company_id)
            pending_section = None
            # Invoice values.
            invoice_vals = order._prepare_invoice()
            # Invoice line values (keep only necessary sections).
            for line in order.order_line:
                if line.display_type == 'line_section':
                    pending_section = line
                    continue
                if not float_is_zero(line.qty_to_invoice, precision_digits=precision):
                    if pending_section:
                        line_vals = pending_section._prepare_account_move_line()
                        line_vals.update({'sequence': sequence})
                        invoice_vals['invoice_line_ids'].append((0, 0, line_vals))
                        sequence += 1
                        pending_section = None
                    line_vals = line._prepare_account_move_line()
                    line_vals.update({'sequence': sequence})
                    invoice_vals['invoice_line_ids'].append((0, 0, line_vals))
                    sequence += 1
            invoice_vals_list.append(invoice_vals)

        if not invoice_vals_list:
            raise UserError(_('There is no invoiceable line. If a product has a control policy based on received quantity, please make sure that a quantity has been received.'))

        # 2) group by (company_id, partner_id, currency_id) for batch creation
        new_invoice_vals_list = []
        for grouping_keys, invoices in groupby(invoice_vals_list, key=lambda x: (x.get('company_id'), x.get('partner_id'), x.get('currency_id'))):
            origins = set()
            payment_refs = set()
            refs = set()
            ref_invoice_vals = None
            for invoice_vals in invoices:
                if not ref_invoice_vals:
                    ref_invoice_vals = invoice_vals
                else:
                    ref_invoice_vals['invoice_line_ids'] += invoice_vals['invoice_line_ids']
                origins.add(invoice_vals['invoice_origin'])
                payment_refs.add(invoice_vals['payment_reference'])
                refs.add(invoice_vals['ref'])
            ref_invoice_vals.update({
                'ref': ', '.join(refs)[:2000],
                'invoice_origin': ', '.join(origins),
                'payment_reference': len(payment_refs) == 1 and payment_refs.pop() or False,
            })
            new_invoice_vals_list.append(ref_invoice_vals)
        invoice_vals_list = new_invoice_vals_list

        # 3) Create invoices.
        moves = self.env['account.move']
        AccountMove = self.env['account.move'].with_context(default_move_type='in_invoice')
        for vals in invoice_vals_list:
            moves |= AccountMove.with_company(vals['company_id']).create(vals)

        # 4) Some moves might actually be refunds: convert them if the total amount is negative
        # We do this after the moves have been created since we need taxes, etc. to know if the total
        # is actually negative or not
        moves.filtered(lambda m: m.currency_id.round(m.amount_total) < 0).action_switch_move_type()

        return self.action_view_invoice(moves)

    def _prepare_invoice(self):
        """Prepare the dict of values to create the new invoice for a purchase order.
        """
        self.ensure_one()
        move_type = self._context.get('default_move_type', 'in_invoice')

        partner_invoice = self.env['res.partner'].browse(self.partner_id.address_get(['invoice'])['invoice'])
        partner_bank_id = self.partner_id.commercial_partner_id.bank_ids.filtered_domain(['|', ('company_id', '=', False), ('company_id', '=', self.company_id.id)])[:1]

        invoice_vals = {
            'ref': self.partner_ref or '',
            'move_type': move_type,
            'narration': self.notes,
            'currency_id': self.currency_id.id,
            'partner_id': partner_invoice.id,
            'fiscal_position_id': (self.fiscal_position_id or self.fiscal_position_id._get_fiscal_position(partner_invoice)).id,
            'payment_reference': self.partner_ref or '',
            'partner_bank_id': partner_bank_id.id,
            'invoice_origin': self.name,
            'invoice_payment_term_id': self.payment_term_id.id,
            'invoice_line_ids': [],
            'company_id': self.company_id.id,
        }
        return invoice_vals

    def action_view_invoice(self, invoices=False):
        """This function returns an action that display existing vendor bills of
        given purchase order ids. When only one found, show the vendor bill
        immediately.
        """
        if not invoices:
            self.invalidate_model(['invoice_ids'])
            invoices = self.invoice_ids

        result = self.env['ir.actions.act_window']._for_xml_id('account.action_move_in_invoice_type')
        # choose the view_mode accordingly
        if len(invoices) > 1:
            result['domain'] = [('id', 'in', invoices.ids)]
        elif len(invoices) == 1:
            res = self.env.ref('account.view_move_form', False)
            form_view = [(res and res.id or False, 'form')]
            if 'views' in result:
                result['views'] = form_view + [(state, view) for state, view in result['views'] if view != 'form']
            else:
                result['views'] = form_view
            result['res_id'] = invoices.id
        else:
            result = {'type': 'ir.actions.act_window_close'}

        return result

    @api.model
    def retrieve_dashboard(self):
        """ This function returns the values to populate the custom dashboard in
            the purchase order views.
        """
        self.check_access_rights('read')

        result = {
            'all_to_send': 0,
            'all_waiting': 0,
            'all_late': 0,
            'my_to_send': 0,
            'my_waiting': 0,
            'my_late': 0,
            'all_avg_order_value': 0,
            'all_avg_days_to_purchase': 0,
            'all_total_last_7_days': 0,
            'all_sent_rfqs': 0,
            'company_currency_symbol': self.env.company.currency_id.symbol
        }

        one_week_ago = fields.Datetime.to_string(fields.Datetime.now() - relativedelta(days=7))

        query = """SELECT COUNT(1)
                   FROM mail_message m
                   JOIN purchase_order po ON (po.id = m.res_id)
                   WHERE m.create_date >= %s
                     AND m.model = 'purchase.order'
                     AND m.message_type = 'notification'
                     AND m.subtype_id = %s
                     AND po.company_id = %s;
                """

        self.env.cr.execute(query, (one_week_ago, self.env.ref('purchase.mt_rfq_sent').id, self.env.company.id))
        res = self.env.cr.fetchone()
        result['all_sent_rfqs'] = res[0] or 0

        # easy counts
        po = self.env['purchase.order']
        result['all_to_send'] = po.search_count([('state', '=', 'draft')])
        result['my_to_send'] = po.search_count([('state', '=', 'draft'), ('user_id', '=', self.env.uid)])
        result['all_waiting'] = po.search_count([('state', '=', 'sent'), ('date_order', '>=', fields.Datetime.now())])
        result['my_waiting'] = po.search_count([('state', '=', 'sent'), ('date_order', '>=', fields.Datetime.now()), ('user_id', '=', self.env.uid)])
        result['all_late'] = po.search_count([('state', 'in', ['draft', 'sent', 'to approve']), ('date_order', '<', fields.Datetime.now())])
        result['my_late'] = po.search_count([('state', 'in', ['draft', 'sent', 'to approve']), ('date_order', '<', fields.Datetime.now()), ('user_id', '=', self.env.uid)])

        # Calculated values ('avg order value', 'avg days to purchase', and 'total last 7 days') note that 'avg order value' and
        # 'total last 7 days' takes into account exchange rate and current company's currency's precision.
        # This is done via SQL for scalability reasons
        query = """SELECT AVG(COALESCE(po.amount_total / NULLIF(po.currency_rate, 0), po.amount_total)),
                          AVG(extract(epoch from age(po.date_approve,po.create_date)/(24*60*60)::decimal(16,2))),
                          SUM(CASE WHEN po.date_approve >= %s THEN COALESCE(po.amount_total / NULLIF(po.currency_rate, 0), po.amount_total) ELSE 0 END)
                   FROM purchase_order po
                   WHERE po.state in ('purchase', 'done')
                     AND po.company_id = %s
                """
        self._cr.execute(query, (one_week_ago, self.env.company.id))
        res = self.env.cr.fetchone()
        result['all_avg_days_to_purchase'] = round(res[1] or 0, 2)
        currency = self.env.company.currency_id
        result['all_avg_order_value'] = format_amount(self.env, res[0] or 0, currency)
        result['all_total_last_7_days'] = format_amount(self.env, res[2] or 0, currency)

        return result

    def _send_reminder_mail(self, send_single=False):
        if not self.user_has_groups('purchase.group_send_reminder'):
            return

        template = self.env.ref('purchase.email_template_edi_purchase_reminder', raise_if_not_found=False)
        if template:
            orders = self if send_single else self._get_orders_to_remind()
            for order in orders:
                date = order.date_planned
                if date and (send_single or (date - relativedelta(days=order.reminder_date_before_receipt)).date() == datetime.today().date()):
                    if send_single:
                        return order._send_reminder_open_composer(template.id)
                    else:
                        order.with_context(is_reminder=True).message_post_with_source(
                            template,
                            email_layout_xmlid="mail.mail_notification_layout_with_responsible_signature",
                            subtype_xmlid='mail.mt_comment',
                        )

    def send_reminder_preview(self):
        self.ensure_one()
        if not self.user_has_groups('purchase.group_send_reminder'):
            return

        template = self.env.ref('purchase.email_template_edi_purchase_reminder', raise_if_not_found=False)
        if template and self.env.user.email and self.id:
            template.with_context(is_reminder=True).send_mail(
                self.id,
                force_send=True,
                raise_exception=False,
                email_layout_xmlid="mail.mail_notification_layout_with_responsible_signature",
                email_values={'email_to': self.env.user.email, 'recipient_ids': []},
            )
            return {'toast_message': escape(_("A sample email has been sent to %s.", self.env.user.email))}

    def _send_reminder_open_composer(self,template_id):
        self.ensure_one()
        try:
            compose_form_id = self.env['ir.model.data']._xmlid_lookup('mail.email_compose_message_wizard_form')[1]
        except ValueError:
            compose_form_id = False
        ctx = dict(self.env.context or {})
        ctx.update({
            'default_model': 'purchase.order',
            'default_res_ids': self.ids,
            'default_template_id': template_id,
            'default_composition_mode': 'comment',
            'default_email_layout_xmlid': "mail.mail_notification_layout_with_responsible_signature",
            'force_email': True,
            'mark_rfq_as_sent': True,
        })
        lang = self.env.context.get('lang')
        if {'default_template_id', 'default_model', 'default_res_id'} <= ctx.keys():
            template = self.env['mail.template'].browse(ctx['default_template_id'])
            if template and template.lang:
                lang = template._render_lang([ctx['default_res_id']])[ctx['default_res_id']]
        self = self.with_context(lang=lang)
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

    @api.model
    def _get_orders_to_remind(self):
        """When auto sending a reminder mail, only send for unconfirmed purchase
        order and not all products are service."""
        return self.search([
            ('partner_id', '!=', False),
            ('state', 'in', ['purchase', 'done']),
            ('mail_reminder_confirmed', '=', False)
        ]).filtered(lambda p: p.partner_id.with_company(p.company_id).receipt_reminder_email and\
            p.mapped('order_line.product_id.product_tmpl_id.type') != ['service'])

    def _default_order_line_values(self):
        default_data = super()._default_order_line_values()
        new_default_data = self.env['purchase.order.line']._get_product_catalog_lines_data()
        return {**default_data, **new_default_data}

    def _get_action_add_from_catalog_extra_context(self):
        return {
            **super()._get_action_add_from_catalog_extra_context(),
            'display_uom': self.env.user.has_group('uom.group_uom'),
            'precision': self.env['decimal.precision'].precision_get('Product Unit of Measure'),
            'product_catalog_currency_id': self.currency_id.id,
            'product_catalog_digits': self.order_line._fields['price_unit'].get_digits(self.env),
            'search_default_seller_ids': self.partner_id.name,
        }

    def _get_product_catalog_domain(self):
        return expression.AND([super()._get_product_catalog_domain(), [('purchase_ok', '=', True)]])

    def _get_product_catalog_order_data(self, products, **kwargs):
        res = super()._get_product_catalog_order_data(products, **kwargs)
        for product in products:
            res[product.id] |= self._get_product_price_and_data(product)
        return res

    def _get_product_catalog_record_lines(self, product_ids):
        grouped_lines = defaultdict(lambda: self.env['purchase.order.line'])
        for line in self.order_line:
            if line.display_type or line.product_id.id not in product_ids:
                continue
            grouped_lines[line.product_id] |= line
        return grouped_lines

    def _get_product_price_and_data(self, product):
        """ Fetch the product's data used by the purchase's catalog.

        :return: the product's price and, if applicable, the minimum quantity to
                 buy and the product's packaging data.
        :rtype: dict
        """
        self.ensure_one()
        product_infos = {
            'price': product.standard_price,
            'uom': {
                'display_name': product.uom_id.display_name,
                'id': product.uom_id.id,
            },
        }
        if product.purchase_line_warn_msg:
            product_infos['warning'] = product.purchase_line_warn_msg
        if product.purchase_line_warn == "block":
            product_infos['readOnly'] = True
        if product.uom_id != product.uom_po_id:
            product_infos['purchase_uom'] = {
                'display_name': product.uom_po_id.display_name,
                'id': product.uom_po_id.id,
            }
        params = {'order_id': self}
        # Check if there is a price and a minimum quantity for the order's vendor.
        seller = product._select_seller(
            partner_id=self.partner_id,
            quantity=None,
            date=self.date_order and self.date_order.date(),
            uom_id=product.uom_id,
            ordered_by='min_qty',
            params=params
        )
        if seller:
            product_infos.update(
                price=seller.price_discounted,
                min_qty=seller.min_qty,
            )
        # Check if the product uses some packaging.
        packaging = self.env['product.packaging'].search(
            [('product_id', '=', product.id), ('purchase', '=', True)], limit=1
        )
        if packaging:
            qty = packaging.product_uom_id._compute_quantity(packaging.qty, product.uom_po_id)
            product_infos.update(
                packaging={
                    'id': packaging.id,
                    'name': packaging.display_name,
                    'qty': qty,
                }
            )
        return product_infos

    def get_confirm_url(self, confirm_type=None):
        """Create url for confirm reminder or purchase reception email for sending
        in mail."""
        if confirm_type in ['reminder', 'reception']:
            param = url_encode({
                'confirm': confirm_type,
                'confirmed_date': self.date_planned and self.date_planned.date(),
            })
            return self.get_portal_url(query_string='&%s' % param)
        return self.get_portal_url()

    def get_update_url(self):
        """Create portal url for user to update the scheduled date on purchase
        order lines."""
        update_param = url_encode({'update': 'True'})
        return self.get_portal_url(query_string='&%s' % update_param)

    def confirm_reminder_mail(self, confirmed_date=False):
        for order in self:
            if order.state in ['purchase', 'done'] and not order.mail_reminder_confirmed:
                order.mail_reminder_confirmed = True
                date_planned = order.get_localized_date_planned(confirmed_date).date()
                order.message_post(body=_("%s confirmed the receipt will take place on %s.", order.partner_id.name, date_planned))

    def _approval_allowed(self):
        """Returns whether the order qualifies to be approved by the current user"""
        self.ensure_one()
        return (
            self.company_id.po_double_validation == 'one_step'
            or (self.company_id.po_double_validation == 'two_step'
                and self.amount_total < self.env.company.currency_id._convert(
                    self.company_id.po_double_validation_amount, self.currency_id, self.company_id,
                    self.date_order or fields.Date.today()))
            or self.user_has_groups('purchase.group_purchase_manager'))

    def _confirm_reception_mail(self):
        for order in self:
            if order.state in ['purchase', 'done'] and not order.mail_reception_confirmed:
                order.mail_reception_confirmed = True
                order.message_post(body=_("The order receipt has been acknowledged by %s.", order.partner_id.name))

    def get_localized_date_planned(self, date_planned=False):
        """Returns the localized date planned in the timezone of the order's user or the
        company's partner or UTC if none of them are set."""
        self.ensure_one()
        date_planned = date_planned or self.date_planned
        if not date_planned:
            return False
        if isinstance(date_planned, str):
            date_planned = fields.Datetime.from_string(date_planned)
        tz = self.get_order_timezone()
        return date_planned.astimezone(tz)

    def get_order_timezone(self):
        """ Returns the timezone of the order's user or the company's partner
        or UTC if none of them are set. """
        self.ensure_one()
        return timezone(self.user_id.tz or self.company_id.partner_id.tz or 'UTC')

    def _update_date_planned_for_lines(self, updated_dates):
        # create or update the activity
        activity = self.env['mail.activity'].search([
            ('summary', '=', _('Date Updated')),
            ('res_model_id', '=', 'purchase.order'),
            ('res_id', '=', self.id),
            ('user_id', '=', self.user_id.id)], limit=1)
        if activity:
            self._update_update_date_activity(updated_dates, activity)
        else:
            self._create_update_date_activity(updated_dates)

        # update the date on PO line
        for line, date in updated_dates:
            line._update_date_planned(date)

    def _update_order_line_info(self, product_id, quantity, **kwargs):
        """ Update purchase order line information for a given product or create
        a new one if none exists yet.
        :param int product_id: The product, as a `product.product` id.
        :return: The unit price of the product, based on the pricelist of the
                 purchase order and the quantity selected.
        :rtype: float
        """
        self.ensure_one()
        product_packaging_qty = kwargs.get('product_packaging_qty', False)
        product_packaging_id = kwargs.get('product_packaging_id', False)
        pol = self.order_line.filtered(lambda line: line.product_id.id == product_id)
        if pol:
            if product_packaging_qty:
                pol.product_packaging_id = product_packaging_id
                pol.product_packaging_qty = product_packaging_qty
            elif quantity != 0:
                pol.product_qty = quantity
            elif self.state in ['draft', 'sent']:
                price_unit = self._get_product_price_and_data(pol.product_id)['price']
                pol.unlink()
                return price_unit
            else:
                pol.product_qty = 0
        elif quantity > 0:
            pol = self.env['purchase.order.line'].create({
                'order_id': self.id,
                'product_id': product_id,
                'product_qty': quantity,
                'sequence': ((self.order_line and self.order_line[-1].sequence + 1) or 10),  # put it at the end of the order
            })
            seller = pol.product_id._select_seller(
                partner_id=pol.partner_id,
                quantity=pol.product_qty,
                date=pol.order_id.date_order and pol.order_id.date_order.date() or fields.Date.context_today(pol),
                uom_id=pol.product_uom)
            if seller:
                # Fix the PO line's price on the seller's one.
                pol.price_unit = seller.price_discounted
        return pol.price_unit_discounted

    def _create_update_date_activity(self, updated_dates):
        note = Markup('<p>%s</p>\n') % _('%s modified receipt dates for the following products:', self.partner_id.name)
        for line, date in updated_dates:
            note += Markup('<p> - %s</p>\n') % _(
                '%(product)s from %(original_receipt_date)s to %(new_receipt_date)s',
                product=line.product_id.display_name,
                original_receipt_date=line.date_planned.date(),
                new_receipt_date=date.date()
            )
        activity = self.activity_schedule(
            'mail.mail_activity_data_warning',
            summary=_("Date Updated"),
            user_id=self.user_id.id
        )
        # add the note after we post the activity because the note can be soon
        # changed when updating the date of the next PO line. So instead of
        # sending a mail with incomplete note, we send one with no note.
        activity.note = note
        return activity

    def _update_update_date_activity(self, updated_dates, activity):
        for line, date in updated_dates:
            activity.note += Markup('<p> - %s</p>\n') %  _(
                '%(product)s from %(original_receipt_date)s to %(new_receipt_date)s',
                product=line.product_id.display_name,
                original_receipt_date=line.date_planned.date(),
                new_receipt_date=date.date()
            )

    def _write_partner_values(self, vals):
        partner_values = {}
        if 'receipt_reminder_email' in vals:
            partner_values['receipt_reminder_email'] = vals.pop('receipt_reminder_email')
        if 'reminder_date_before_receipt' in vals:
            partner_values['reminder_date_before_receipt'] = vals.pop('reminder_date_before_receipt')
        return vals, partner_values

    def _is_readonly(self):
        """ Return whether the purchase order is read-only or not based on the state.
        A purchase order is considered read-only if its state is 'cancel'.

        :return: Whether the purchase order is read-only or not.
        :rtype: bool
        """
        self.ensure_one()
        return self.state == 'cancel'

```

## File: models\purchase_order_line.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from datetime import datetime, time
from dateutil.relativedelta import relativedelta
from pytz import UTC

from odoo import api, fields, models, _
from odoo.tools import DEFAULT_SERVER_DATETIME_FORMAT, get_lang
from odoo.tools.float_utils import float_compare, float_round
from odoo.exceptions import UserError


class PurchaseOrderLine(models.Model):
    _name = 'purchase.order.line'
    _inherit = 'analytic.mixin'
    _description = 'Purchase Order Line'
    _order = 'order_id, sequence, id'

    name = fields.Text(
        string='Description', required=True, compute='_compute_price_unit_and_date_planned_and_name', store=True, readonly=False)
    sequence = fields.Integer(string='Sequence', default=10)
    product_qty = fields.Float(string='Quantity', digits='Product Unit of Measure', required=True,
                               compute='_compute_product_qty', store=True, readonly=False)
    product_uom_qty = fields.Float(string='Total Quantity', compute='_compute_product_uom_qty', store=True)
    date_planned = fields.Datetime(
        string='Expected Arrival', index=True,
        compute="_compute_price_unit_and_date_planned_and_name", readonly=False, store=True,
        help="Delivery date expected from vendor. This date respectively defaults to vendor pricelist lead time then today's date.")
    discount = fields.Float(
        string="Discount (%)",
        compute='_compute_price_unit_and_date_planned_and_name',
        digits='Discount',
        store=True, readonly=False)
    taxes_id = fields.Many2many('account.tax', string='Taxes', context={'active_test': False})
    product_uom = fields.Many2one('uom.uom', string='Unit of Measure', domain="[('category_id', '=', product_uom_category_id)]")
    product_uom_category_id = fields.Many2one(related='product_id.uom_id.category_id')
    product_id = fields.Many2one('product.product', string='Product', domain=[('purchase_ok', '=', True)], change_default=True, index='btree_not_null')
    product_type = fields.Selection(related='product_id.detailed_type', readonly=True)
    price_unit = fields.Float(
        string='Unit Price', required=True, digits='Product Price',
        compute="_compute_price_unit_and_date_planned_and_name", readonly=False, store=True)
    price_unit_discounted = fields.Float('Unit Price (Discounted)', compute='_compute_price_unit_discounted')

    price_subtotal = fields.Monetary(compute='_compute_amount', string='Subtotal', store=True)
    price_total = fields.Monetary(compute='_compute_amount', string='Total', store=True)
    price_tax = fields.Float(compute='_compute_amount', string='Tax', store=True)

    order_id = fields.Many2one('purchase.order', string='Order Reference', index=True, required=True, ondelete='cascade')

    company_id = fields.Many2one('res.company', related='order_id.company_id', string='Company', store=True, readonly=True)
    state = fields.Selection(related='order_id.state', store=True)

    invoice_lines = fields.One2many('account.move.line', 'purchase_line_id', string="Bill Lines", readonly=True, copy=False)

    # Replace by invoiced Qty
    qty_invoiced = fields.Float(compute='_compute_qty_invoiced', string="Billed Qty", digits='Product Unit of Measure', store=True)

    qty_received_method = fields.Selection([('manual', 'Manual')], string="Received Qty Method", compute='_compute_qty_received_method', store=True,
        help="According to product configuration, the received quantity can be automatically computed by mechanism:\n"
             "  - Manual: the quantity is set manually on the line\n"
             "  - Stock Moves: the quantity comes from confirmed pickings\n")
    qty_received = fields.Float("Received Qty", compute='_compute_qty_received', inverse='_inverse_qty_received', compute_sudo=True, store=True, digits='Product Unit of Measure')
    qty_received_manual = fields.Float("Manual Received Qty", digits='Product Unit of Measure', copy=False)
    qty_to_invoice = fields.Float(compute='_compute_qty_invoiced', string='To Invoice Quantity', store=True, readonly=True,
                                  digits='Product Unit of Measure')

    partner_id = fields.Many2one('res.partner', related='order_id.partner_id', string='Partner', readonly=True, store=True, index='btree_not_null')
    currency_id = fields.Many2one(related='order_id.currency_id', store=True, string='Currency', readonly=True)
    date_order = fields.Datetime(related='order_id.date_order', string='Order Date', readonly=True)
    date_approve = fields.Datetime(related="order_id.date_approve", string='Confirmation Date', readonly=True)
    product_packaging_id = fields.Many2one('product.packaging', string='Packaging', domain="[('purchase', '=', True), ('product_id', '=', product_id)]", check_company=True,
                                           compute="_compute_product_packaging_id", store=True, readonly=False)
    product_packaging_qty = fields.Float('Packaging Quantity', compute="_compute_product_packaging_qty", store=True, readonly=False)
    tax_calculation_rounding_method = fields.Selection(
        related='company_id.tax_calculation_rounding_method',
        string='Tax calculation rounding method', readonly=True)
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

    @api.depends('product_qty', 'price_unit', 'taxes_id', 'discount')
    def _compute_amount(self):
        for line in self:
            tax_results = self.env['account.tax']._compute_taxes([line._convert_to_tax_base_line_dict()])
            totals = next(iter(tax_results['totals'].values()))
            amount_untaxed = totals['amount_untaxed']
            amount_tax = totals['amount_tax']

            line.update({
                'price_subtotal': amount_untaxed,
                'price_tax': amount_tax,
                'price_total': amount_untaxed + amount_tax,
            })

    def _convert_to_tax_base_line_dict(self):
        """ Convert the current record to a dictionary in order to use the generic taxes computation method
        defined on account.tax.

        :return: A python dictionary.
        """
        self.ensure_one()
        return self.env['account.tax']._convert_to_tax_base_line_dict(
            self,
            partner=self.order_id.partner_id,
            currency=self.order_id.currency_id,
            product=self.product_id,
            taxes=self.taxes_id,
            price_unit=self.price_unit,
            quantity=self.product_qty,
            discount=self.discount,
            price_subtotal=self.price_subtotal,
        )

    def _compute_tax_id(self):
        for line in self:
            line = line.with_company(line.company_id)
            fpos = line.order_id.fiscal_position_id or line.order_id.fiscal_position_id._get_fiscal_position(line.order_id.partner_id)
            # filter taxes by company
            taxes = line.product_id.supplier_taxes_id._filter_taxes_by_company(line.company_id)
            line.taxes_id = fpos.map_tax(taxes)

    @api.depends('discount', 'price_unit')
    def _compute_price_unit_discounted(self):
        for line in self:
            line.price_unit_discounted = line.price_unit * (1 - line.discount / 100)

    @api.depends('invoice_lines.move_id.state', 'invoice_lines.quantity', 'qty_received', 'product_uom_qty', 'order_id.state')
    def _compute_qty_invoiced(self):
        for line in self:
            # compute qty_invoiced
            qty = 0.0
            for inv_line in line._get_invoice_lines():
                if inv_line.move_id.state not in ['cancel'] or inv_line.move_id.payment_state == 'invoicing_legacy':
                    if inv_line.move_id.move_type == 'in_invoice':
                        qty += inv_line.product_uom_id._compute_quantity(inv_line.quantity, line.product_uom)
                    elif inv_line.move_id.move_type == 'in_refund':
                        qty -= inv_line.product_uom_id._compute_quantity(inv_line.quantity, line.product_uom)
            line.qty_invoiced = qty

            # compute qty_to_invoice
            if line.order_id.state in ['purchase', 'done']:
                if line.product_id.purchase_method == 'purchase':
                    line.qty_to_invoice = line.product_qty - line.qty_invoiced
                else:
                    line.qty_to_invoice = line.qty_received - line.qty_invoiced
            else:
                line.qty_to_invoice = 0

    def _get_invoice_lines(self):
        self.ensure_one()
        if self._context.get('accrual_entry_date'):
            return self.invoice_lines.filtered(
                lambda l: l.move_id.invoice_date and l.move_id.invoice_date <= self._context['accrual_entry_date']
            )
        else:
            return self.invoice_lines

    @api.depends('product_id', 'product_id.type')
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

    @api.model_create_multi
    def create(self, vals_list):
        for values in vals_list:
            if values.get('display_type', self.default_get(['display_type'])['display_type']):
                values.update(product_id=False, price_unit=0, product_uom_qty=0, product_uom=False, date_planned=False)
            else:
                values.update(self._prepare_add_missing_fields(values))

        lines = super().create(vals_list)
        for line in lines:
            if line.product_id and line.order_id.state == 'purchase':
                msg = _("Extra line with %s ", line.product_id.display_name)
                line.order_id.message_post(body=msg)
        return lines

    def write(self, values):
        if 'display_type' in values and self.filtered(lambda line: line.display_type != values.get('display_type')):
            raise UserError(_("You cannot change the type of a purchase order line. Instead you should delete the current line and create a new line of the proper type."))

        if 'product_qty' in values:
            precision = self.env['decimal.precision'].precision_get('Product Unit of Measure')
            for line in self:
                if (
                    line.order_id.state == "purchase"
                    and float_compare(line.product_qty, values["product_qty"], precision_digits=precision) != 0
                ):
                    line.order_id.message_post_with_source(
                        'purchase.track_po_line_template',
                        render_values={'line': line, 'product_qty': values['product_qty']},
                        subtype_xmlid='mail.mt_note',
                    )

        if 'qty_received' in values:
            for line in self:
                line._track_qty_received(values['qty_received'])
        return super(PurchaseOrderLine, self).write(values)

    @api.ondelete(at_uninstall=False)
    def _unlink_except_purchase_or_done(self):
        for line in self:
            if line.order_id.state in ['purchase', 'done']:
                state_description = {state_desc[0]: state_desc[1] for state_desc in self._fields['state']._description_selection(self.env)}
                raise UserError(_('Cannot delete a purchase order line which is in state %r.', state_description.get(line.state)))

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

    @api.depends('product_id', 'order_id.partner_id')
    def _compute_analytic_distribution(self):
        for line in self:
            if not line.display_type:
                distribution = self.env['account.analytic.distribution.model']._get_distribution({
                    "product_id": line.product_id.id,
                    "product_categ_id": line.product_id.categ_id.id,
                    "partner_id": line.order_id.partner_id.id,
                    "partner_category_id": line.order_id.partner_id.category_id.ids,
                    "company_id": line.company_id.id,
                })
                line.analytic_distribution = distribution or line.analytic_distribution

    @api.onchange('product_id')
    def onchange_product_id(self):
        # TODO: Remove when onchanges are replaced with computes
        if not self.product_id or (self.env.context.get('origin_po_id') and self.product_qty):
            return

        # Reset date, price and quantity since _onchange_quantity will provide default values
        self.price_unit = self.product_qty = 0.0

        self._product_id_change()

        self._suggest_quantity()

    def _product_id_change(self):
        if not self.product_id:
            return

        self.product_uom = self.product_id.uom_po_id or self.product_id.uom_id
        product_lang = self.product_id.with_context(
            lang=get_lang(self.env, self.partner_id.lang).code,
            partner_id=None,
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
            title = _("Warning for %s", product_info.name)
            message = product_info.purchase_line_warn_msg
            warning['title'] = title
            warning['message'] = message
            if product_info.purchase_line_warn == 'block':
                self.product_id = False
            return {'warning': warning}
        return {}

    @api.depends('product_qty', 'product_uom', 'company_id')
    def _compute_price_unit_and_date_planned_and_name(self):
        for line in self:
            if not line.product_id or line.invoice_lines or not line.company_id:
                continue
            params = line._get_select_sellers_params()
            seller = line.product_id._select_seller(
                partner_id=line.partner_id,
                quantity=line.product_qty,
                date=line.order_id.date_order and line.order_id.date_order.date() or fields.Date.context_today(line),
                uom_id=line.product_uom,
                params=params)

            if seller or not line.date_planned:
                line.date_planned = line._get_date_planned(seller).strftime(DEFAULT_SERVER_DATETIME_FORMAT)

            # If not seller, use the standard price. It needs a proper currency conversion.
            if not seller:
                line.discount = 0
                unavailable_seller = line.product_id.seller_ids.filtered(
                    lambda s: s.partner_id == line.order_id.partner_id)
                if not unavailable_seller and line.price_unit and line.product_uom == line._origin.product_uom:
                    # Avoid to modify the price unit if there is no price list for this partner and
                    # the line has already one to avoid to override unit price set manually.
                    continue
                po_line_uom = line.product_uom or line.product_id.uom_po_id
                price_unit = line.env['account.tax']._fix_tax_included_price_company(
                    line.product_id.uom_id._compute_price(line.product_id.standard_price, po_line_uom),
                    line.product_id.supplier_taxes_id,
                    line.taxes_id,
                    line.company_id,
                )
                price_unit = line.product_id.cost_currency_id._convert(
                    price_unit,
                    line.currency_id,
                    line.company_id,
                    line.date_order or fields.Date.context_today(line),
                    False
                )
                line.price_unit = float_round(price_unit, precision_digits=max(line.currency_id.decimal_places, self.env['decimal.precision'].precision_get('Product Price')))

            elif seller:
                price_unit = line.env['account.tax']._fix_tax_included_price_company(seller.price, line.product_id.supplier_taxes_id, line.taxes_id, line.company_id) if seller else 0.0
                price_unit = seller.currency_id._convert(price_unit, line.currency_id, line.company_id, line.date_order or fields.Date.context_today(line), False)
                price_unit = float_round(price_unit, precision_digits=max(line.currency_id.decimal_places, self.env['decimal.precision'].precision_get('Product Price')))
                line.price_unit = seller.product_uom._compute_price(price_unit, line.product_uom)
                line.discount = seller.discount or 0.0

            # record product names to avoid resetting custom descriptions
            default_names = []
            vendors = line.product_id._prepare_sellers(params=params)
            product_ctx = {'seller_id': None, 'partner_id': None, 'lang': get_lang(line.env, line.partner_id.lang).code}
            default_names.append(line._get_product_purchase_description(line.product_id.with_context(product_ctx)))
            for vendor in vendors:
                product_ctx = {'seller_id': vendor.id, 'lang': get_lang(line.env, line.partner_id.lang).code}
                default_names.append(line._get_product_purchase_description(line.product_id.with_context(product_ctx)))
            if not line.name or line.name in default_names:
                product_ctx = {'seller_id': seller.id, 'lang': get_lang(line.env, line.partner_id.lang).code}
                line.name = line._get_product_purchase_description(line.product_id.with_context(product_ctx))

    @api.depends('product_id', 'product_qty', 'product_uom')
    def _compute_product_packaging_id(self):
        for line in self:
            # remove packaging if not match the product
            if line.product_packaging_id.product_id != line.product_id:
                line.product_packaging_id = False
            # suggest biggest suitable packaging matching the PO's company
            if line.product_id and line.product_qty and line.product_uom:
                suggested_packaging = line.product_id.packaging_ids\
                        .filtered(lambda p: p.purchase and (p.product_id.company_id <= p.company_id <= line.company_id))\
                        ._find_suitable_product_packaging(line.product_qty, line.product_uom)
                line.product_packaging_id = suggested_packaging or line.product_packaging_id

    @api.onchange('product_packaging_id')
    def _onchange_product_packaging_id(self):
        if self.product_packaging_id and self.product_qty:
            newqty = self.product_packaging_id._check_qty(self.product_qty, self.product_uom, "UP")
            if float_compare(newqty, self.product_qty, precision_rounding=self.product_uom.rounding) != 0:
                return {
                    'warning': {
                        'title': _('Warning'),
                        'message': _(
                            "This product is packaged by %(pack_size).2f %(pack_name)s. You should purchase %(quantity).2f %(unit)s.",
                            pack_size=self.product_packaging_id.qty,
                            pack_name=self.product_id.uom_id.name,
                            quantity=newqty,
                            unit=self.product_uom.name
                        ),
                    },
                }

    @api.depends('product_packaging_id', 'product_uom', 'product_qty')
    def _compute_product_packaging_qty(self):
        self.product_packaging_qty = 0
        for line in self:
            if not line.product_packaging_id:
                continue
            line.product_packaging_qty = line.product_packaging_id._compute_qty(line.product_qty, line.product_uom)

    @api.depends('product_packaging_qty')
    def _compute_product_qty(self):
        for line in self:
            if line.product_packaging_id:
                packaging_uom = line.product_packaging_id.product_uom_id
                qty_per_packaging = line.product_packaging_id.qty
                product_qty = packaging_uom._compute_quantity(line.product_packaging_qty * qty_per_packaging, line.product_uom)
                if float_compare(product_qty, line.product_qty, precision_rounding=line.product_uom.rounding) != 0:
                    line.product_qty = product_qty

    @api.depends('product_uom', 'product_qty', 'product_id.uom_id')
    def _compute_product_uom_qty(self):
        for line in self:
            if line.product_id and line.product_id.uom_id != line.product_uom:
                line.product_uom_qty = line.product_uom._compute_quantity(line.product_qty, line.product_id.uom_id)
            else:
                line.product_uom_qty = line.product_qty

    def _get_gross_price_unit(self):
        self.ensure_one()
        price_unit = self.price_unit
        if self.discount:
            price_unit = price_unit * (1 - self.discount / 100)
        if self.taxes_id:
            qty = self.product_qty or 1
            price_unit = self.taxes_id.with_context(round=False, round_base=False).compute_all(price_unit, currency=self.order_id.currency_id, quantity=qty, product=self.product_id)['total_void']
            price_unit = price_unit / qty
        if self.product_uom.id != self.product_id.uom_id.id:
            price_unit *= self.product_uom.factor / self.product_id.uom_id.factor
        return price_unit

    def action_add_from_catalog(self):
        order = self.env['purchase.order'].browse(self.env.context.get('order_id'))
        return order.action_add_from_catalog()

    def action_purchase_history(self):
        self.ensure_one()
        action = self.env["ir.actions.actions"]._for_xml_id("purchase.action_purchase_history")
        action['domain'] = [('state', 'in', ['purchase', 'done']), ('product_id', '=', self.product_id.id)]
        action['display_name'] = _("Purchase History for %s", self.product_id.display_name)
        action['context'] = {
            'search_default_partner_id': self.partner_id.id
        }

        return action

    def _suggest_quantity(self):
        '''
        Suggest a minimal quantity based on the seller
        '''
        if not self.product_id:
            return
        seller_min_qty = self.product_id.seller_ids\
            .filtered(lambda r: r.partner_id == self.order_id.partner_id and (not r.product_id or r.product_id == self.product_id))\
            .sorted(key=lambda r: r.min_qty)
        if seller_min_qty:
            self.product_qty = seller_min_qty[0].min_qty or 1.0
            self.product_uom = seller_min_qty[0].product_uom
        else:
            self.product_qty = 1.0

    def _get_product_catalog_lines_data(self):
        """ Return information about purchase order lines in `self`.

        If `self` is empty, this method returns only the default value(s) needed for the product
        catalog. In this case, the quantity that equals 0.

        Otherwise, it returns a quantity and a price based on the product of the POL(s) and whether
        the product is read-only or not.

        A product is considered read-only if the order is considered read-only (see
        ``PurchaseOrder._is_readonly`` for more details) or if `self` contains multiple records
        or if it has purchase_line_warn == "block".

        Note: This method cannot be called with multiple records that have different products linked.

        :raise odoo.exceptions.ValueError: ``len(self.product_id) != 1``
        :rtype: dict
        :return: A dict with the following structure:
            {
                'quantity': float,
                'price': float,
                'readOnly': bool,
                'uom': dict,
                'purchase_uom': dict,
                'packaging': dict,
                'warning': String,
            }
        """
        if len(self) == 1:
            catalog_info = self.order_id._get_product_price_and_data(self.product_id)
            uom = {
                'display_name': self.product_id.uom_id.display_name,
                'id': self.product_id.uom_id.id,
            }
            catalog_info.update(
                quantity=self.product_qty,
                price=self.price_unit * (1 - self.discount / 100),
                readOnly=self.order_id._is_readonly(),
                uom=uom,
            )
            if self.product_id.uom_id != self.product_uom:
                catalog_info['purchase_uom'] = {
                'display_name': self.product_uom.display_name,
                'id': self.product_uom.id,
            }
            if self.product_packaging_id:
                packaging = self.product_packaging_id
                catalog_info['packaging'] = {
                    'id': packaging.id,
                    'name': packaging.display_name,
                    'qty': packaging.product_uom_id._compute_quantity(packaging.qty, self.product_uom),
                }
            return catalog_info
        elif self:
            self.product_id.ensure_one()
            order_line = self[0]
            catalog_info = order_line.order_id._get_product_price_and_data(order_line.product_id)
            catalog_info['quantity'] = sum(self.mapped(
                lambda line: line.product_uom._compute_quantity(
                    qty=line.product_qty,
                    to_unit=line.product_id.uom_id,
            )))
            catalog_info['readOnly'] = True
            return catalog_info
        return {'quantity': 0}

    def _get_product_purchase_description(self, product_lang):
        self.ensure_one()
        name = product_lang.display_name
        if product_lang.description_purchase:
            name += '\n' + product_lang.description_purchase

        return name

    def _prepare_account_move_line(self, move=False):
        self.ensure_one()
        aml_currency = move and move.currency_id or self.currency_id
        date = move and move.date or fields.Date.today()
        res = {
            'display_type': self.display_type or 'product',
            'name': '%s: %s' % (self.order_id.name, self.name),
            'product_id': self.product_id.id,
            'product_uom_id': self.product_uom.id,
            'quantity': self.qty_to_invoice,
            'discount': self.discount,
            'price_unit': self.currency_id._convert(self.price_unit, aml_currency, self.company_id, date, round=False),
            'tax_ids': [(6, 0, self.taxes_id.ids)],
            'purchase_line_id': self.id,
        }
        if self.analytic_distribution and not self.display_type:
            res['analytic_distribution'] = self.analytic_distribution
        return res

    @api.model
    def _prepare_add_missing_fields(self, values):
        """ Deduce missing required fields from the onchange """
        res = {}
        onchange_fields = ['name', 'price_unit', 'product_qty', 'product_uom', 'taxes_id', 'date_planned']
        if values.get('order_id') and values.get('product_id') and any(f not in values for f in onchange_fields):
            line = self.new(values)
            line.onchange_product_id()
            for field in onchange_fields:
                if field not in values:
                    res[field] = line._fields[field].convert_to_write(line[field], line)
        return res

    @api.model
    def _prepare_purchase_order_line(self, product_id, product_qty, product_uom, company_id, supplier, po):
        partner = supplier.partner_id
        uom_po_qty = product_uom._compute_quantity(product_qty, product_id.uom_po_id, rounding_method='HALF-UP')
        # _select_seller is used if the supplier have different price depending
        # the quantities ordered.
        today = fields.Date.today()
        seller = product_id.with_company(company_id)._select_seller(
            partner_id=partner,
            quantity=uom_po_qty,
            date=po.date_order and max(po.date_order.date(), today) or today,
            uom_id=product_id.uom_po_id)

        product_taxes = product_id.supplier_taxes_id.filtered(lambda x: x.company_id in company_id.parent_ids)
        taxes = po.fiscal_position_id.map_tax(product_taxes)

        price_unit = self.env['account.tax']._fix_tax_included_price_company(
            seller.price, product_taxes, taxes, company_id) if seller else 0
        if price_unit and seller and po.currency_id and seller.currency_id != po.currency_id:
            price_unit = seller.currency_id._convert(
                price_unit, po.currency_id, po.company_id, po.date_order or fields.Date.today())

        product_lang = product_id.with_prefetch().with_context(
            lang=partner.lang,
            partner_id=partner.id,
        )
        name = product_lang.with_context(seller_id=seller.id).display_name
        if product_lang.description_purchase:
            name += '\n' + product_lang.description_purchase

        date_planned = self.order_id.date_planned or self._get_date_planned(seller, po=po)
        discount = seller.discount or 0.0

        return {
            'name': name,
            'product_qty': uom_po_qty,
            'product_id': product_id.id,
            'product_uom': product_id.uom_po_id.id,
            'price_unit': price_unit,
            'date_planned': date_planned,
            'taxes_id': [(6, 0, taxes.ids)],
            'order_id': po.id,
            'discount': discount,
        }

    def _convert_to_middle_of_day(self, date):
        """Return a datetime which is the noon of the input date(time) according
        to order user's time zone, convert to UTC time.
        """
        return self.order_id.get_order_timezone().localize(datetime.combine(date, time(12))).astimezone(UTC).replace(tzinfo=None)

    def _update_date_planned(self, updated_date):
        self.date_planned = updated_date

    def _track_qty_received(self, new_qty):
        self.ensure_one()
        # don't track anything when coming from the accrued expense entry wizard, as it is only computing fields at a past date to get relevant amounts
        # and doesn't actually change anything to the current record
        if  self.env.context.get('accrual_entry_date'):
            return
        if new_qty != self.qty_received and self.order_id.state == 'purchase':
            self.order_id.message_post_with_source(
                'purchase.track_po_line_qty_received_template',
                render_values={'line': self, 'qty_received': new_qty},
                subtype_xmlid='mail.mt_note',
            )

    def _validate_analytic_distribution(self):
        for line in self:
            if line.display_type:
                continue
            line._validate_distribution(
                product=line.product_id.id,
                business_domain='purchase_order',
                company_id=line.company_id.id,
            )

    def _get_select_sellers_params(self):
        self.ensure_one()
        return {
            "order_id": self.order_id,
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
    company_currency_id = fields.Many2one('res.currency', related='company_id.currency_id', string="Company Currency", readonly=True)
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

    group_send_reminder = fields.Boolean("Receipt Reminder", implied_group='purchase.group_send_reminder', default=True,
        help="Allow automatically send email to remind your vendor the receipt date")

    @api.onchange('use_po_lead')
    def _onchange_use_po_lead(self):
        if not self.use_po_lead:
            self.po_lead = 0.0

    @api.onchange('group_product_variant')
    def _onchange_group_product_variant_purchase(self):
        """If the user disables the product variants -> disable the product configurator as well"""
        if self.module_purchase_product_matrix and not self.group_product_variant:
            self.module_purchase_product_matrix = False

    @api.onchange('module_purchase_product_matrix')
    def _onchange_module_purchase_product_matrix(self):
        """The product variant grid requires the product variants activated
        If the user enables the product configurator -> enable the product variants as well"""
        if self.module_purchase_product_matrix and not self.group_product_variant:
            self.group_product_variant = True

    def set_values(self):
        super().set_values()
        po_lock = 'lock' if self.lock_confirmed_po else 'edit'
        po_double_validation = 'two_step' if self.po_order_approval else 'one_step'
        if self.po_lock != po_lock:
            self.po_lock = po_lock
        if self.po_double_validation != po_double_validation:
            self.po_double_validation = po_double_validation

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
        all_partners = self.with_context(active_test=False).search_fetch(
            [('id', 'child_of', self.ids)],
            ['parent_id'],
        )
        purchase_order_groups = self.env['purchase.order']._read_group(
            domain=[('partner_id', 'in', all_partners.ids)],
            groupby=['partner_id'], aggregates=['__count'],
        )
        self_ids = set(self._ids)

        self.purchase_order_count = 0
        for partner, count in purchase_order_groups:
            while partner:
                if partner.id in self_ids:
                    partner.purchase_order_count += count
                partner = partner.parent_id

    @api.model
    def _commercial_fields(self):
        return super(res_partner, self)._commercial_fields()

    property_purchase_currency_id = fields.Many2one(
        'res.currency', string="Supplier Currency", company_dependent=True,
        help="This currency will be used, instead of the default one, for purchases from the current partner")
    purchase_order_count = fields.Integer(compute='_compute_purchase_order_count', string='Purchase Order Count')
    purchase_warn = fields.Selection(WARNING_MESSAGE, 'Purchase Order Warning', help=WARNING_HELP, default="no-message")
    purchase_warn_msg = fields.Text('Message for Purchase Order')

    receipt_reminder_email = fields.Boolean('Receipt Reminder', default=False, company_dependent=True,
        help="Automatically send a confirmation email to the vendor X days before the expected receipt date, asking him to confirm the exact date.")
    reminder_date_before_receipt = fields.Integer('Days Before Receipt', default=1, company_dependent=True,
        help="Number of days to send reminder email before the promised receipt date")
    buyer_id = fields.Many2one('res.users', string='Buyer')

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_invoice
from . import account_tax
from . import analytic_account
from . import analytic_applicability
from . import purchase_order
from . import purchase_order_line
from . import product
from . import res_company
from . import res_config_settings
from . import res_partner

```

## File: populate\purchase.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
from datetime import datetime, timedelta

from odoo import models
from odoo.tools import populate, groupby
from odoo.addons.stock.populate.stock import COMPANY_NB_WITH_STOCK

_logger = logging.getLogger(__name__)


class ResCompany(models.Model):
    _inherit = 'res.company'

    def _populate_factories(self):
        return super()._populate_factories() + [
            ('po_lead', populate.randint(0, 2))
        ]


class PurchaseOrder(models.Model):
    _inherit = 'purchase.order'

    _populate_sizes = {'small': 100, 'medium': 1_500, 'large': 25_000}
    _populate_dependencies = ['res.partner']

    def _populate_factories(self):
        now = datetime.now()

        company_ids = self.env.registry.populated_models['res.company'][:COMPANY_NB_WITH_STOCK]
        all_partners = self.env['res.partner'].browse(self.env.registry.populated_models['res.partner'])
        partners_by_company = dict(groupby(all_partners, key=lambda par: par.company_id.id))
        partners_inter_company = self.env['res.partner'].concat(*partners_by_company.get(False, []))
        partners_by_company = {com: self.env['res.partner'].concat(*partners) | partners_inter_company for com, partners in partners_by_company.items() if com}

        def get_date_order(values, counter, random):
            # 95.45 % of picking scheduled between (-5, 10) days and follow a gauss distribution (only +-15% PO is late)
            delta = random.gauss(5, 5)
            return now + timedelta(days=delta)

        def get_date_planned(values, counter, random):
            # 95 % of PO Receipt Date between (1, 16) days after the order deadline and follow a exponential distribution
            delta = random.expovariate(5) + 1
            return values['date_order'] + timedelta(days=delta)

        def get_partner_id(values, counter, random):
            return random.choice(partners_by_company[values['company_id']]).id

        def get_currency_id(values, counter, random):
            company = self.env['res.company'].browse(values['company_id'])
            return company.currency_id.id

        return [
            ('company_id', populate.randomize(company_ids)),
            ('date_order', populate.compute(get_date_order)),
            ('date_planned', populate.compute(get_date_planned)),
            ('partner_id', populate.compute(get_partner_id)),
            ('currency_id', populate.compute(get_currency_id)),
        ]


class PurchaseOrderLine(models.Model):
    _inherit = 'purchase.order.line'

    _populate_sizes = {'small': 500, 'medium': 7_500, 'large': 125_000}
    _populate_dependencies = ['purchase.order', 'product.product']

    def _populate_factories(self):

        purchase_order_ids = self.env.registry.populated_models['purchase.order']
        product_ids = self.env.registry.populated_models['product.product']

        def get_product_uom(values, counter, random):
            product = self.env['product.product'].browse(values['product_id'])
            return product.uom_id.id

        def get_date_planned(values, counter, random):
            po = self.env['purchase.order'].browse(values['order_id'])
            return po.date_planned

        return [
            ('order_id', populate.iterate(purchase_order_ids)),
            ('name', populate.constant("PO-line-{counter}")),
            ('product_id', populate.randomize(product_ids)),
            ('product_uom', populate.compute(get_product_uom)),
            ('taxes_id', populate.constant(False)),  # to avoid slow _prepare_add_missing_fields
            ('date_planned', populate.compute(get_date_planned)),
            ('product_qty', populate.randint(1, 10)),
            ('price_unit', populate.randint(10, 100)),
        ]

```

## File: populate\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import purchase

```

## File: report\purchase_bill.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, tools
from odoo.tools import formatLang

class PurchaseBillUnion(models.Model):
    _name = 'purchase.bill.union'
    _auto = False
    _description = 'Purchases & Bills Union'
    _order = "date desc, name desc"
    _rec_names_search = ['name', 'reference']

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
                    move_type='in_invoice' and state = 'posted'
            UNION
                SELECT
                    -id, name, partner_ref as reference, partner_id, date_order::date as date, amount_untaxed as amount, currency_id, company_id,
                    NULL as vendor_bill_id, id as purchase_order_id
                FROM purchase_order
                WHERE
                    state in ('purchase', 'done') AND
                    invoice_status in ('to invoice', 'no')
            )""")

    @api.depends('currency_id', 'reference', 'amount', 'purchase_order_id')
    @api.depends_context('show_total_amount')
    def _compute_display_name(self):
        for doc in self:
            name = doc.name or ''
            if doc.reference:
                name += ' - ' + doc.reference
            amount = doc.amount
            if doc.purchase_order_id and doc.purchase_order_id.invoice_status == 'no':
                amount = 0.0
            name += ': ' + formatLang(self.env, amount, monetary=True, currency_obj=doc.currency_id)
            doc.display_name = name

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
                <field name="name" string="Reference" filter_domain="['|', ('name','ilike',self), ('reference','=like',self+'%')]"/>
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
                <field name="currency_id" column_invisible="True"/>
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
                 t-options='{"widget": "contact", "fields": ["address", "name", "phone", "vat"], "no_marker": True, "phone_icons": True}'/>
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

            <div class="mt-4">
                <h2 t-if="o.state in ['draft', 'sent', 'to approve']">Request for Quotation #<span t-field="o.name"/></h2>
                <h2 t-if="o.state in ['purchase', 'done']">Purchase Order #<span t-field="o.name"/></h2>
                <h2 t-if="o.state == 'cancel'">Cancelled Purchase Order #<span t-field="o.name"/></h2>
            </div>

            <div id="informations" class="row mt-4 mb32">
                <div t-if="o.user_id" class="col-3 bm-2">
                    <strong>Buyer:</strong>
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
                    <strong>Order Deadline:</strong>
                    <p t-field="o.date_order" class="m-0"/>
                </div>
            </div>

            <table class="table table-sm o_main_table table-borderless mt-4">
                <thead style="display: table-row-group">
                    <tr>
                        <th name="th_description"><strong>Description</strong></th>
                        <th name="th_date_req" class="text-center"><strong>Date Req.</strong></th>
                        <th name="th_quantity" class="text-end"><strong>Qty</strong></th>
                        <th name="th_price_unit" class="text-end"><strong>Unit Price</strong></th>
                        <th name="th_discount" class="text-end"><strong>Disc.</strong></th>
                        <th name="th_taxes" class="text-end"><strong>Taxes</strong></th>
                        <th name="th_subtotal" class="text-end">
                            <strong>Amount</strong>
                        </th>
                    </tr>
                </thead>
                <tbody>
                    <t t-set="current_subtotal" t-value="0"/>
                    <t t-foreach="o.order_line" t-as="line">
                        <t t-set="current_subtotal" t-value="current_subtotal + line.price_subtotal"/>

                        <tr t-att-class="'bg-200 fw-bold o_line_section' if line.display_type == 'line_section' else 'fst-italic o_line_note' if line.display_type == 'line_note' else ''">
                            <t t-if="not line.display_type">
                                <td id="product">
                                    <span t-field="line.name"/>
                                </td>
                                <td class="text-center">
                                    <span t-field="line.date_planned"/>
                                </td>
                                <td class="text-end">
                                    <span t-field="line.product_qty"/>
                                    <span t-field="line.product_uom.name" groups="uom.group_uom"/>
                                    <span t-if="line.product_packaging_id">
                                        (<span t-field="line.product_packaging_qty" t-options='{"widget": "integer"}'/> <span t-field="line.product_packaging_id"/>)
                                    </span>
                                </td>
                                <td class="text-end">
                                    <span t-field="line.price_unit"/>
                                </td>
                                <td class="text-end">
                                    <span class="text-align-bottom"><span t-field="line.discount"/>%</span>
                                </td>
                                <t t-set="taxes" t-value="', '.join([(tax.invoice_label or tax.name) for tax in line.taxes_id])"/>
                                <td name="td_taxes" t-attf-class="text-end {{ 'text-nowrap' if len(taxes) &lt; 10 else '' }}">
                                    <span t-out="taxes">Tax 15%</span>
                                </td>
                                <td class="text-end">
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
                            <tr class="is-subtotal text-end">
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
                    <table class="table table-sm table-borderless">
                        <t t-set="tax_totals" t-value="o.tax_totals"/>
                        <t t-call="purchase.document_tax_totals"/>
                    </table>
                </div>
            </div>

            <p t-field="o.notes" class="mt-4"/>
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

 <!-- Allow edits (e.g. studio) without changing the often inherited base template -->
<template id="document_tax_totals" inherit_id="account.document_tax_totals_template" primary="True"></template>

</odoo>

```

## File: report\purchase_quotation_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<template id="report_purchasequotation_document">
    <t t-call="web.external_layout">
        <t t-set="o" t-value="o.with_context(lang=o.partner_id.lang)"/>
        <t t-set="forced_vat" t-value="o.fiscal_position_id.foreign_vat"/> <!-- So that it appears in the footer of the report instead of the company VAT if it's set -->
        <t t-set="address">
            <div t-field="o.partner_id"
                 t-options='{"widget": "contact", "fields": ["address", "name", "phone", "vat"], "no_marker": True, "phone_icons": True}'/>
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

            <h2 class="mt-4">Request for Quotation <span t-field="o.name"/></h2>

            <table class="table table-sm mt-4">
                <thead style="display: table-row-group">
                    <tr>
                        <th name="th_description"><strong>Description</strong></th>
                        <th name="th_expected_date" class="text-center"><strong>Expected Date</strong></th>
                        <th name="th_quantity" class="text-end"><strong>Qty</strong></th>
                    </tr>
                </thead>
                <tbody>
                    <t t-foreach="o.order_line" t-as="order_line">
                        <tr t-att-class="'bg-200 fw-bold o_line_section' if order_line.display_type == 'line_section' else 'fst-italic o_line_note' if order_line.display_type == 'line_note' else ''">
                            <t t-if="not order_line.display_type">
                                <td id="product">
                                    <span t-field="order_line.name"/>
                                </td>
                                <td class="text-center">
                                    <span t-field="order_line.date_planned"/>
                                </td>
                                <td class="text-end">
                                    <span t-field="order_line.product_qty"/>
                                    <span t-field="order_line.product_uom" groups="uom.group_uom"/>
                                    <span t-if="order_line.product_packaging_id">
                                        (<span t-field="order_line.product_packaging_qty" t-options='{"widget": "integer"}'/> <span t-field="order_line.product_packaging_id"/>)
                                    </span>
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

            <p t-field="o.notes" class="mt-4"/>

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

from odoo import fields, models, api


class PurchaseReport(models.Model):
    _name = "purchase.report"
    _description = "Purchase Report"
    _auto = False
    _order = 'date_order desc, price_total desc'

    date_order = fields.Datetime('Order Date', readonly=True)
    state = fields.Selection([
        ('draft', 'Draft RFQ'),
        ('sent', 'RFQ Sent'),
        ('to approve', 'To Approve'),
        ('purchase', 'Purchase Order'),
        ('done', 'Done'),
        ('cancel', 'Cancelled')
    ], 'Status', readonly=True)
    product_id = fields.Many2one('product.product', 'Product', readonly=True)
    partner_id = fields.Many2one('res.partner', 'Vendor', readonly=True)
    date_approve = fields.Datetime('Confirmation Date', readonly=True)
    product_uom = fields.Many2one('uom.uom', 'Reference Unit of Measure', required=True)
    company_id = fields.Many2one('res.company', 'Company', readonly=True)
    currency_id = fields.Many2one('res.currency', 'Currency', readonly=True)
    user_id = fields.Many2one('res.users', 'Buyer', readonly=True)
    delay = fields.Float('Days to Confirm', digits=(16, 2), readonly=True, group_operator='avg', help="Amount of time between purchase approval and order by date.")
    delay_pass = fields.Float('Days to Receive', digits=(16, 2), readonly=True, group_operator='avg',
                              help="Amount of time between date planned and order by date for each purchase order line.")
    price_total = fields.Float('Total', readonly=True)
    price_average = fields.Float('Average Cost', readonly=True, group_operator="avg", digits='Product Price')
    nbr_lines = fields.Integer('# of Lines', readonly=True)
    category_id = fields.Many2one('product.category', 'Product Category', readonly=True)
    product_tmpl_id = fields.Many2one('product.template', 'Product Template', readonly=True)
    country_id = fields.Many2one('res.country', 'Partner Country', readonly=True)
    fiscal_position_id = fields.Many2one('account.fiscal.position', string='Fiscal Position', readonly=True)
    commercial_partner_id = fields.Many2one('res.partner', 'Commercial Entity', readonly=True)
    weight = fields.Float('Gross Weight', readonly=True)
    volume = fields.Float('Volume', readonly=True)
    order_id = fields.Many2one('purchase.order', 'Order', readonly=True)
    untaxed_total = fields.Float('Untaxed Total', readonly=True)
    qty_ordered = fields.Float('Qty Ordered', readonly=True)
    qty_received = fields.Float('Qty Received', readonly=True)
    qty_billed = fields.Float('Qty Billed', readonly=True)
    qty_to_be_billed = fields.Float('Qty to be Billed', readonly=True)

    @property
    def _table_query(self):
        ''' Report needs to be dynamic to take into account multi-company selected + multi-currency rates '''
        return '%s %s %s %s' % (self._select(), self._from(), self._where(), self._group_by())

    def _select(self):
        select_str = """
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
                    c.currency_id,
                    t.uom_id as product_uom,
                    extract(epoch from age(po.date_approve,po.date_order))/(24*60*60)::decimal(16,2) as delay,
                    extract(epoch from age(l.date_planned,po.date_order))/(24*60*60)::decimal(16,2) as delay_pass,
                    count(*) as nbr_lines,
                    sum(l.price_total / COALESCE(po.currency_rate, 1.0))::decimal(16,2) * currency_table.rate as price_total,
                    (sum(l.product_qty * l.price_unit / COALESCE(po.currency_rate, 1.0))/NULLIF(sum(l.product_qty/line_uom.factor*product_uom.factor),0.0))::decimal(16,2) * currency_table.rate as price_average,
                    partner.country_id as country_id,
                    partner.commercial_partner_id as commercial_partner_id,
                    sum(p.weight * l.product_qty/line_uom.factor*product_uom.factor) as weight,
                    sum(p.volume * l.product_qty/line_uom.factor*product_uom.factor) as volume,
                    sum(l.price_subtotal / COALESCE(po.currency_rate, 1.0))::decimal(16,2) * currency_table.rate as untaxed_total,
                    sum(l.product_qty / line_uom.factor * product_uom.factor) as qty_ordered,
                    sum(l.qty_received / line_uom.factor * product_uom.factor) as qty_received,
                    sum(l.qty_invoiced / line_uom.factor * product_uom.factor) as qty_billed,
                    case when t.purchase_method = 'purchase' 
                         then sum(l.product_qty / line_uom.factor * product_uom.factor) - sum(l.qty_invoiced / line_uom.factor * product_uom.factor)
                         else sum(l.qty_received / line_uom.factor * product_uom.factor) - sum(l.qty_invoiced / line_uom.factor * product_uom.factor)
                    end as qty_to_be_billed
        """
        return select_str

    def _from(self):
        from_str = """
            FROM
            purchase_order_line l
                join purchase_order po on (l.order_id=po.id)
                join res_partner partner on po.partner_id = partner.id
                    left join product_product p on (l.product_id=p.id)
                        left join product_template t on (p.product_tmpl_id=t.id)
                left join res_company C ON C.id = po.company_id
                left join uom_uom line_uom on (line_uom.id=l.product_uom)
                left join uom_uom product_uom on (product_uom.id=t.uom_id)
                left join {currency_table} ON currency_table.company_id = po.company_id
        """.format(
            currency_table=self.env['res.currency']._get_query_currency_table(self.env.companies.ids, fields.Date.today())
        )
        return from_str

    def _where(self):
        return """
            WHERE
                l.display_type IS NULL
        """

    def _group_by(self):
        group_by_str = """
            GROUP BY
                po.company_id,
                po.user_id,
                po.partner_id,
                line_uom.factor,
                c.currency_id,
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
                po.id,
                currency_table.rate
        """
        return group_by_str

    @api.model
    def read_group(self, domain, fields, groupby, offset=0, limit=None, orderby=False, lazy=True):
        """
        This is a hack to allow us to correctly calculate the average price of product.
        """
        if 'price_average:avg' in fields:
            fields.extend(['aggregated_qty_ordered:array_agg(qty_ordered)'])
            fields.extend(['aggregated_price_average:array_agg(price_average)'])

        res = []
        if fields:
            res = super(PurchaseReport, self).read_group(domain, fields, groupby, offset=offset, limit=limit, orderby=orderby, lazy=lazy)

        if 'price_average:avg' in fields:
            qties = 'aggregated_qty_ordered'
            special_field = 'aggregated_price_average'
            for data in res:
                if data[special_field] and data[qties]:
                    total_unit_cost = sum(float(value) * float(qty) for value, qty in zip(data[special_field], data[qties]) if qty and value)
                    total_qty_ordered = sum(float(qty) for qty in data[qties] if qty)
                    data['price_average'] = (total_unit_cost / total_qty_ordered) if total_qty_ordered else 0
                del data[special_field]
                del data[qties]

        return res

```

## File: report\purchase_reports.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="action_report_purchase_order" model="ir.actions.report">
            <field name="name">Purchase Order</field>
            <field name="model">purchase.order</field>
            <field name="report_type">qweb-pdf</field>
            <field name="report_name">purchase.report_purchaseorder</field>
            <field name="report_file">purchase.report_purchaseorder</field>
            <field name="print_report_name">
                (object.state in ('draft', 'sent') and 'Request for Quotation - %s' % (object.name) or
                'Purchase Order - %s' % (object.name))</field>
            <field name="binding_model_id" ref="model_purchase_order"/>
            <field name="binding_type">report</field>
        </record>

        <record id="report_purchase_quotation" model="ir.actions.report">
            <field name="name">Request for Quotation</field>
            <field name="model">purchase.order</field>
            <field name="report_type">qweb-pdf</field>
            <field name="report_name">purchase.report_purchasequotation</field>
            <field name="report_file">purchase.report_purchasequotation</field>
            <field name="print_report_name">'Request for Quotation - %s' % (object.name)</field>
            <field name="binding_model_id" ref="model_purchase_order"/>
            <field name="binding_type">report</field>
        </record>
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
                <pivot string="Purchase Analysis" display_quantity="1" sample="1">
                    <field name="category_id" type="row"/>
                    <field name="order_id" type="row"/>
                    <field name="untaxed_total" type="measure"/>
                    <field name="price_total" type="measure"/>
                </pivot>
            </field>
        </record>
        <record model="ir.ui.view" id="view_purchase_order_graph">
            <field name="name">product.month.graph</field>
            <field name="model">purchase.report</field>
            <field name="arch" type="xml">
                <graph string="Purchase Analysis" type="line" sample="1">
                    <field name="date_approve" interval="day"/>
                    <field name="untaxed_total" type="measure"/>
                </graph>
            </field>
        </record>

        <record id="purchase_report_view_tree" model="ir.ui.view">
            <field name="name">purchase.report.view.tree</field>
            <field name="model">purchase.report</field>
            <field name="arch" type="xml">
                <tree string="Purchase Analysis">
                    <field name="date_order" widget="date"/>
                    <field name="order_id" optional="show"/>
                    <field name="partner_id" optional="show"/>
                    <field name="product_id" optional="show"/>
                    <field name="category_id" optional="show"/>
                    <field name="user_id" optional="show" widget="many2one_avatar_user"/>
                    <field name="company_id" optional="show" groups="base.group_multi_company"/>
                    <field name="qty_ordered" optional="hide" sum="Sum of Qty Ordered"/>
                    <field name="qty_received" optional="hide" sum="Sum of Qty Received"/>
                    <field name="qty_billed" optional="hide" sum="Sum of Qty Billed"/>
                    <field name="currency_id" optional="show" column_invisible="True"/>
                    <field name="untaxed_total" optional="hide" widget="monetary" sum="Sum of Untaxed Total"/>
                    <field name="price_total" optional="show" widget="monetary" sum="Sum of Total"/>
                    <field name="state" optional="show"/>
                </tree>
            </field>
        </record>

        <record id="view_purchase_order_search" model="ir.ui.view">
        <field name="name">report.purchase.order.search</field>
        <field name="model">purchase.report</field>
        <field name="arch" type="xml">
            <search string="Purchase Orders">
                <filter string="Requests for Quotation" name="quotes" domain="[('state','in',('draft','sent'))]"/>
                <filter string="Purchase Orders" name="orders" domain="[('state','!=','draft'), ('state','!=','sent'), ('state','!=','cancel')]"/>
                <filter string="Confirmation Date Last Year" name="later_than_a_year_ago" domain="[('date_approve', '&gt;=', ((context_today()-relativedelta(years=1)).strftime('%Y-%m-%d')))]"/>
                <filter name="filter_date_order" date="date_order"/>
                <filter name="filter_date_approve" date="date_approve" default_period="this_month"/>
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
                    <filter string="Buyer" name="user_id" context="{'group_by':'user_id'}"/>
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
        <field name="context" eval="{
                'search_default_orders': 1,
                'search_default_filter_date_approve': 1}"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                No Purchase Analysis
            </p><p>
                This analysis allows you to easily check and analyse your company purchase history and performance.
                You can track your negotiation performance, the delivery performance of your vendors, etc
            </p>
        </field>
        <field name="target">current</field>
    </record>

    <menuitem id="purchase_report_main" name="Reporting" parent="purchase.menu_purchase_root" sequence="99" groups="purchase.group_purchase_manager"/>
    <menuitem id="purchase_report" name="Purchase" parent="purchase.purchase_report_main" sequence="99" groups="purchase.group_purchase_manager" action="action_purchase_order_report_all"/>

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
access_purchase_order_invoicing_payments_readonly,purchase.order,model_purchase_order,account.group_account_readonly,1,0,0,0
access_purchase_order_invoicing_payments,purchase.order,model_purchase_order,account.group_account_invoice,1,1,0,0
access_purchase_order_portal,purchase.order.portal,purchase.model_purchase_order,base.group_portal,1,0,0,0
access_purchase_order_line,purchase.order.line user,model_purchase_order_line,group_purchase_user,1,1,1,1
access_purchase_order_line_manager,purchase.order.line,model_purchase_order_line,group_purchase_manager,1,1,1,1
access_purchase_order_line_invoicing_payments_readonly,purchase.order.line,model_purchase_order_line,account.group_account_readonly,1,0,0,0
access_purchase_order_line_invoicing_payments,purchase.order.line,model_purchase_order_line,account.group_account_invoice,1,1,0,0
access_purchase_order_line_portal,purchase.order.line.portal,purchase.model_purchase_order_line,base.group_portal,1,0,0,0
access_account_tax_purchase_user,account.tax,account.model_account_tax,group_purchase_user,1,0,0,0
access_account_tag_purchase_user,account.account.tag,account.model_account_account_tag,group_purchase_user,1,0,0,0
access_account_tax_purchase_user_manager,account.tax,account.model_account_tax,group_purchase_manager,1,0,0,0
access_product_product_purchase_user,product.product.purchase.user,product.model_product_product,group_purchase_user,1,0,0,0
access_product_product_purchase_manager,product.product purchase_manager,product.model_product_product,purchase.group_purchase_manager,1,1,1,1
access_product_template_purchase_user,product.template purchase_user,product.model_product_template,group_purchase_user,1,0,0,0
access_account_fiscal_position_purchase_user,account.fiscal.position purchase,account.model_account_fiscal_position,group_purchase_user,1,0,0,0
access_res_partner_purchase_user,res.partner purchase,base.model_res_partner,group_purchase_user,1,0,0,0
access_account_journal,account.journal,account.model_account_journal,group_purchase_user,1,0,0,0
access_account_journal_manager,account.journal,account.model_account_journal,group_purchase_manager,1,0,0,0
access_account_move,account.move,account.model_account_move,group_purchase_user,1,1,1,1
access_account_move_line_manager,account.move.line,account.model_account_move_line,group_purchase_manager,1,1,1,1
access_account_move_line,account.move.line,account.model_account_move_line,group_purchase_user,1,1,1,0
access_account_analytic_line,account.analytic.line,account.model_account_analytic_line,group_purchase_user,1,0,0,0
account_partial_reconcile,account.partial.reconcile.purchase.user,account.model_account_partial_reconcile,group_purchase_user,1,0,0,0
access_res_partner_purchase_manager,res.partner.purchase.manager,base.model_res_partner,group_purchase_manager,1,1,1,0
access_uom_category_purchase_manager,uom.category purchase_manager,uom.model_uom_category,purchase.group_purchase_manager,1,1,1,1
access_uom_uom_purchase_manager,uom.uom purchase_manager,uom.model_uom_uom,purchase.group_purchase_manager,1,1,1,1
access_product_category_purchase_manager,product.category purchase_manager,product.model_product_category,purchase.group_purchase_manager,1,1,1,1
access_product_template_purchase_manager,product.template purchase_manager,product.model_product_template,purchase.group_purchase_manager,1,1,1,1
access_product_packaging_purchase_manager,product.packaging purchase_manager,product.model_product_packaging,purchase.group_purchase_manager,1,1,1,1
access_product_supplierinfo_purchase_manager,product.supplierinfo purchase_manager,product.model_product_supplierinfo,purchase.group_purchase_manager,1,1,1,1
access_product_pricelist_item_purchase_manager,product.pricelist.item purchase_manager,product.model_product_pricelist_item,purchase.group_purchase_manager,1,1,1,1
access_product_tag_purchase_manager,product.tag.purchase.manager,product.model_product_tag,purchase.group_purchase_manager,1,1,1,1
access_account_account_purchase_manager,account.account purchase manager,account.model_account_account,purchase.group_purchase_manager,1,0,0,0
access_purchase_bill_union,access_purchase_bill_union,model_purchase_bill_union,purchase.group_purchase_user,1,0,0,0
access_report_purchase_order,purchase.stock.report,model_purchase_report,purchase.group_purchase_manager,1,0,0,0
access_report_purchase_order_user,purchase.stock.report user,model_purchase_report,purchase.group_purchase_user,1,0,0,0

```

## File: security\purchase_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data>

    <record model="ir.module.category" id="base.module_category_inventory_purchase">
        <field name="description">Helps you manage your purchase-related processes such as requests for quotations, supplier bills, etc...</field>
        <field name="sequence">8</field>
    </record>

    <record id="group_purchase_user" model="res.groups">
        <field name="name">User</field>
        <field name="implied_ids" eval="[(4, ref('base.group_user'))]"/>
        <field name="category_id" ref="base.module_category_inventory_purchase"/>
    </record>

    <record id="group_purchase_manager" model="res.groups">
        <field name="name">Administrator</field>
        <field name="category_id" ref="base.module_category_inventory_purchase"/>
        <field name="implied_ids" eval="[(4, ref('group_purchase_user'))]"/>
        <field name="users" eval="[(4, ref('base.user_root')), (4, ref('base.user_admin'))]"/>
    </record>

    <record id="group_warning_purchase" model="res.groups">
        <field name="name">A warning can be set on a product or a customer (Purchase)</field>
        <field name="category_id" ref="base.module_category_hidden"/>
    </record>

    <record id="group_send_reminder" model="res.groups">
        <field name="name">Send an automatic reminder email to confirm delivery</field>
        <field name="category_id" ref="base.module_category_hidden"/>
    </record>

</data>
<data noupdate="1">
    <record model="res.groups" id="base.group_user">
        <field name="implied_ids" eval="[(4, ref('purchase.group_send_reminder'))]"/>
    </record>

    <record id="base.default_user" model="res.users">
        <field name="groups_id" eval="[(4,ref('purchase.group_purchase_manager'))]"/>
    </record>

    <record model="ir.rule" id="purchase_order_comp_rule">
        <field name="name">Purchase Order multi-company</field>
        <field name="model_id" ref="model_purchase_order"/>
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>

    <record model="ir.rule" id="purchase_order_line_comp_rule">
        <field name="name">Purchase Order Line multi-company</field>
        <field name="model_id" ref="model_purchase_order_line"/>
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
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
        <field name="domain_force">[('move_id.move_type', 'in', ('in_invoice', 'in_refund', 'in_receipt'))]</field>
        <field name="groups" eval="[(4, ref('purchase.group_purchase_user'))]"/>
    </record>
    <record id="purchase_user_account_move_rule" model="ir.rule">
        <field name="name">Purchase User Account Move</field>
        <field name="model_id" ref="account.model_account_move"/>
        <field name="domain_force">[('move_type', 'in', ('in_invoice', 'in_refund', 'in_receipt'))]</field>
        <field name="groups" eval="[(4, ref('purchase.group_purchase_user'))]"/>
    </record>
    <record id="portal_purchase_order_line_rule" model="ir.rule">
        <field name="name">Portal Purchase Order Lines</field>
        <field name="model_id" ref="purchase.model_purchase_order_line"/>
        <field name="domain_force">['|',('order_id.message_partner_ids','child_of',[user.commercial_partner_id.id]),('order_id.partner_id','child_of',[user.commercial_partner_id.id])]</field>
        <field name="groups" eval="[(4, ref('base.group_portal'))]"/>
    </record>
    <record model="ir.rule" id="purchase_bill_union_comp_rule">
        <field name="name">Purchases &amp; Bills Union multi-company</field>
        <field name="model_id" ref="model_purchase_bill_union"/>
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
    </record>

    <record id="purchase_order_report_comp_rule" model="ir.rule">
        <field name="name">Purchase Order Report multi-company</field>
        <field name="model_id" ref="model_purchase_report"/>
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>

</data>
</odoo>

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M0 12h50v26a4 4 0 0 1-4 4H0V12Z" fill="#985184"/><path d="M4 21a4 4 0 0 1-4-4v-5a4 4 0 0 1 4-4h46v9a4 4 0 0 1-4 4H4Z" fill="#1AD3BB"/><path d="M0 16h50v4a4 4 0 0 1-4 4H4a4 4 0 0 1-4-4v-4Z" fill="#005E7A"/></svg>

```

## File: static\src\img\calculator.svg

```svg
<svg width="64" height="64" viewBox="0 0 64 64" fill="none" xmlns="http://www.w3.org/2000/svg">
<path d="M15.5901 20.9763V55.6806C15.5901 59.2894 15.3694 58.7419 18.3453 60.3903C19.3255 60.9319 19.7052 61.188 20.2056 61.4412C20.6282 61.6371 21.0912 61.7299 21.5567 61.712C19.9642 61.503 19.7111 60.5875 19.7111 60.5875V23.4989C19.7111 22.0978 19.7111 21.1176 20.3969 20.2375L16.4614 17.9062C15.8887 18.8275 15.5867 19.8914 15.5901 20.9763Z" fill="#FBDBD0"/>
<path d="M44.1896 2.73225C42.7502 1.90217 41.9554 2.25246 39.6153 3.59472C31.7413 8.10716 20.8856 14.3651 19.3049 15.2747C18.1531 15.8969 17.1763 16.7988 16.4644 17.8974L20.3881 20.2198C20.8143 19.7122 21.3351 19.2923 21.9217 18.9836L45.523 5.4168C46.9948 4.56613 48.022 4.79278 48.337 6.00551C48.337 4.63088 45.8644 3.69774 44.1896 2.73225Z" fill="white"/>
<path d="M45.1904 48.3924C44.4251 48.8339 29.2158 57.6057 22.5899 61.3881C21.3477 62.0975 19.5669 61.4912 19.5669 60.5433V23.4341C19.5669 21.356 19.5669 20.1962 21.7922 18.9276L45.4877 5.30487C47.2833 4.27168 48.4136 4.82508 48.4136 6.80903V43.0146C48.4136 46.1818 46.1736 47.8243 45.1904 48.3924Z" fill="white"/>
<path d="M45.9057 21.2854L36.6718 26.7397C36.4694 26.8587 36.3451 27.076 36.3451 27.3108V36.3239C36.3416 36.4279 36.4232 36.515 36.5272 36.5184C36.5671 36.5197 36.6064 36.5083 36.6394 36.4858L45.3494 31.3199C45.7872 31.0616 46.0567 30.5919 46.0588 30.0836V21.3737C46.0607 21.3184 46.0174 21.2721 45.9622 21.2702C45.9423 21.2695 45.9226 21.2748 45.9057 21.2854Z" fill="#C1DBF6"/>
<path d="M33.5428 28.2763L24.3089 33.7307C24.1065 33.8482 23.982 34.0647 23.9822 34.2988V43.306C23.9787 43.41 24.0602 43.4971 24.1642 43.5005C24.2042 43.5018 24.2435 43.4904 24.2765 43.4679L32.9982 38.3167C33.4374 38.0569 33.707 37.5848 33.7076 37.0745V28.3763C33.7226 28.3231 33.6916 28.2679 33.6384 28.2529C33.6047 28.2434 33.5684 28.2523 33.5428 28.2763Z" fill="#C1DBF6"/>
<path d="M45.9057 34.0809L36.6718 39.5353C36.4694 39.6529 36.3449 39.8693 36.3451 40.1034V49.1106C36.3416 49.2146 36.4232 49.3017 36.5272 49.3051C36.5671 49.3064 36.6064 49.295 36.6394 49.2725L45.3611 44.1213C45.8003 43.8615 46.0699 43.3894 46.0705 42.8792V34.1663C46.0757 34.1113 46.0354 34.0624 45.9803 34.0572C45.9533 34.0546 45.9263 34.0632 45.9057 34.0809Z" fill="#C1DBF6"/>
<path d="M33.5428 41.0689L24.3089 46.5232C24.1065 46.6422 23.9822 46.8595 23.9822 47.0943V56.1015C23.9787 56.2055 24.0602 56.2926 24.1642 56.296C24.2042 56.2973 24.2435 56.2859 24.2765 56.2634L32.9982 51.1122C33.4342 50.848 33.6992 50.374 33.6959 49.8641V41.1513C33.6917 41.0945 33.6424 41.0519 33.5856 41.056C33.5706 41.0571 33.5559 41.0615 33.5428 41.0689Z" fill="#C1DBF6"/>
<path d="M41.8759 12.9581L40.2894 13.8912C40.2163 13.9326 40.1713 14.0103 40.1716 14.0943V16.7435C40.1711 16.8735 40.2762 16.9794 40.4062 16.9799C40.449 16.98 40.4911 16.9685 40.5278 16.9466L42.1144 16.0135C42.1874 15.972 42.2324 15.8944 42.2321 15.8104V13.1612C42.2326 13.0311 42.1276 12.9253 41.9975 12.9248C41.9547 12.9246 41.9127 12.9361 41.8759 12.9581ZM41.5404 15.8311L40.7781 16.2961C40.7427 16.3197 40.6574 16.3226 40.6574 16.249V14.3769C40.6574 14.3034 40.7251 14.268 40.7751 14.2386L41.508 13.7882C41.5905 13.7441 41.667 13.7558 41.667 13.8383V15.6397C41.667 15.7339 41.6788 15.7604 41.5404 15.8311Z" fill="#374874"/>
<path fill-rule="evenodd" clip-rule="evenodd" d="M28.8467 33.3942C28.973 33.3942 29.0753 33.4965 29.0753 33.6227V38.6925C29.0753 38.8188 28.973 38.9211 28.8467 38.9211C28.7205 38.9211 28.6181 38.8188 28.6181 38.6925V33.6227C28.6181 33.4965 28.7205 33.3942 28.8467 33.3942Z" fill="#374874"/>
<path fill-rule="evenodd" clip-rule="evenodd" d="M30.8348 35.2242C30.8949 35.3351 30.8537 35.4739 30.7427 35.534L27.0325 37.5453C26.9215 37.6054 26.7828 37.5642 26.7226 37.4532C26.6625 37.3423 26.7037 37.2035 26.8146 37.1434L30.5249 35.1321C30.6359 35.072 30.7746 35.1132 30.8348 35.2242Z" fill="#374874"/>
<path fill-rule="evenodd" clip-rule="evenodd" d="M27.2921 48.1447C27.3615 48.0393 27.5033 48.0101 27.6087 48.0795L30.3205 49.8647C30.4259 49.9341 30.4551 50.0759 30.3857 50.1813C30.3163 50.2867 30.1745 50.3159 30.0691 50.2465L27.3574 48.4613C27.2519 48.3919 27.2227 48.2501 27.2921 48.1447Z" fill="#374874"/>
<path fill-rule="evenodd" clip-rule="evenodd" d="M30.6547 46.1501C30.7649 46.2116 30.8044 46.3509 30.7428 46.4611L27.6727 51.9596C27.6111 52.0698 27.4719 52.1093 27.3617 52.0477C27.2515 51.9862 27.212 51.8469 27.2735 51.7367L30.3437 46.2382C30.4052 46.128 30.5445 46.0885 30.6547 46.1501Z" fill="#374874"/>
<path d="M40.6867 40.2152C40.7089 39.7601 40.9432 39.3417 41.3196 39.0849C41.667 38.8789 41.9495 39.0496 41.9495 39.4647C41.9262 39.9195 41.6936 40.338 41.3196 40.5979C40.9693 40.7922 40.6867 40.6215 40.6867 40.2035V40.2152ZM41.9377 43.2324C41.9135 43.6862 41.6811 44.1034 41.3078 44.3627C40.9575 44.5687 40.675 44.398 40.675 43.9829C40.6953 43.5273 40.93 43.1081 41.3078 42.8526C41.6669 42.6437 41.9377 42.8144 41.9377 43.2324Z" fill="#374874"/>
<path fill-rule="evenodd" clip-rule="evenodd" d="M43.7104 40.2872C43.775 40.3957 43.7394 40.536 43.631 40.6006L39.2156 43.2292C39.1072 43.2937 38.9669 43.2581 38.9023 43.1497C38.8377 43.0412 38.8733 42.9009 38.9818 42.8364L43.3971 40.2078C43.5056 40.1432 43.6459 40.1788 43.7104 40.2872Z" fill="#374874"/>
<path fill-rule="evenodd" clip-rule="evenodd" d="M43.7104 27.5647C43.775 27.6732 43.7394 27.8134 43.631 27.878L39.2156 30.5066C39.1072 30.5712 38.9669 30.5356 38.9023 30.4271C38.8377 30.3186 38.8733 30.1784 38.9818 30.1138L43.3971 27.4852C43.5056 27.4206 43.6459 27.4562 43.7104 27.5647Z" fill="#374874"/>
<path d="M42.7756 2.28587C43.2317 2.28587 43.6684 2.43167 44.1895 2.73224C45.8644 3.69771 48.337 4.63085 48.337 6.00546C48.3331 5.9903 48.3277 5.97716 48.3235 5.96232C48.3809 6.20797 48.4135 6.48765 48.4135 6.80907V43.0146C48.4135 46.1819 46.1735 47.8243 45.1903 48.3924C44.425 48.834 29.2158 57.6057 22.5899 61.3882C22.2348 61.5909 21.8361 61.6832 21.4471 61.6937C21.4841 61.6996 21.5182 61.7069 21.5567 61.7119C21.5189 61.7134 21.4812 61.7141 21.4434 61.7141C21.0163 61.7141 20.5939 61.6212 20.2056 61.4412C19.7052 61.188 19.3254 60.9319 18.3452 60.3903C15.3693 58.742 15.5901 59.2894 15.5901 55.6806V20.9763C15.5867 19.8915 15.8886 18.8275 16.4614 17.9062L16.4643 17.8973C17.1763 16.7988 18.153 15.8969 19.3048 15.2747C20.8855 14.3651 31.7413 8.10715 39.6153 3.59469C41.1081 2.73842 41.972 2.28584 42.7756 2.28587ZM42.7756 1.37158C41.7078 1.37154 40.7062 1.91494 39.1604 2.80159C32.6208 6.54934 24.0517 11.4854 20.3925 13.5931L18.8589 14.4764C17.5809 15.1693 16.4877 16.1801 15.6971 17.4001L15.6336 17.498L15.6266 17.519C15.0004 18.5647 14.672 19.7586 14.6758 20.9791V55.6806C14.6758 55.9304 14.6747 56.1601 14.6737 56.3719C14.6672 57.7864 14.6639 58.4964 15.0132 59.1456C15.3827 59.8324 16.0172 60.173 17.0685 60.7372C17.3135 60.8687 17.59 61.0172 17.9022 61.1901C18.3541 61.4398 18.6806 61.6303 18.9429 61.7834C19.2558 61.9661 19.503 62.1103 19.7927 62.257L19.8068 62.2641L19.821 62.2706C20.3259 62.5047 20.8869 62.6284 21.4434 62.6284C21.4929 62.6284 21.5423 62.6274 21.5918 62.6255L21.5929 62.6024C22.1164 62.5707 22.6155 62.4264 23.0432 62.1821C29.8225 58.312 45.4919 49.2739 45.6471 49.1844C46.7542 48.5447 49.3277 46.6636 49.3277 43.0146V6.80904C49.3277 6.49842 49.3015 6.20214 49.2497 5.92582C49.1929 4.32884 47.4271 3.4082 45.7186 2.51736C45.3356 2.31771 44.9738 2.12909 44.646 1.94013C43.9643 1.54696 43.3874 1.3716 42.7756 1.37158Z" fill="#374874"/>
<path d="M44.5869 10.5179V15.3512C44.5865 15.4858 44.5148 15.6101 44.3985 15.6779L23.3934 27.8406V22.8366C23.3931 22.8073 44.5869 10.5179 44.5869 10.5179ZM44.5869 10.0607C44.5077 10.0607 44.4285 10.0813 44.3576 10.1224C44.3576 10.1224 39.0591 13.1948 33.7605 16.2709C31.1112 17.809 28.4618 19.348 26.4749 20.5043C25.4811 21.0826 24.6529 21.5652 24.0733 21.9042C23.7828 22.074 23.5546 22.2079 23.399 22.2997C23.0694 22.4943 22.933 22.5748 22.9363 22.8423L22.9363 27.8406C22.9363 28.004 23.0235 28.155 23.1651 28.2367C23.2358 28.2774 23.3146 28.2978 23.3934 28.2978C23.4725 28.2978 23.5517 28.2773 23.6225 28.2362L44.6276 16.0735C44.884 15.9239 45.0431 15.648 45.0441 15.3527V10.5178C45.0441 10.3544 44.9568 10.2033 44.8151 10.1217C44.7445 10.081 44.6657 10.0607 44.5869 10.0607Z" fill="#374874"/>
</svg>

```

## File: static\src\img\rfq.svg

```svg
<svg width="64" height="64" viewBox="0 0 64 64" fill="none" xmlns="http://www.w3.org/2000/svg">
<path d="M21.3095 17.3993L22.0328 51.6869C22.0328 51.6869 21.8805 60.315 29.4658 62.0791L30.178 60.0212L33.6656 59.5619L34.4012 57.47L37.9639 57.0515L38.6927 54.9867L42.2732 54.4366L42.912 52.4792L46.4268 51.9162L47.2035 49.905L50.3861 49.5137C50.3861 49.5137 44.5997 49.9724 43.8764 44.5353C43.1531 39.0982 43.8764 12.9918 43.8764 12.9918C43.8764 12.9918 43.684 2.78861 34.7389 1.92065L15.3412 11.8767C15.3412 11.8767 21.3095 11.6206 21.3095 17.3993Z" fill="white"/>
<path d="M13.6426 16.8207C13.6426 16.8207 13.064 11.3236 17.1144 11.4683C21.1649 11.6129 21.3096 17.3993 21.3096 17.3993L13.6179 21.4496L13.6426 16.8207Z" fill="#FBDBD0"/>
<path d="M34.739 1.92076C43.684 2.78873 43.8764 12.9918 43.8764 12.9918C43.8764 12.9918 43.1531 39.0983 43.8764 44.5354C44.4953 49.1877 48.8209 49.5233 50.0569 49.5233C50.2654 49.5233 50.3861 49.5137 50.3861 49.5137L47.2036 49.9051L46.4269 51.9162L42.912 52.4793L42.2733 54.4367L38.6928 54.9868L37.9639 57.0516L34.4012 57.4701L33.6657 59.562L30.1781 60.0213L29.4658 62.0792C21.8805 60.3151 22.0328 51.687 22.0328 51.687L21.3095 17.3993L13.6178 21.4497L13.6425 16.8207C13.6425 16.8207 13.2557 13.1286 15.3467 11.8767C15.3455 11.8767 15.3413 11.8768 15.3413 11.8768L15.4025 11.8454C15.4688 11.8078 15.537 11.7723 15.6083 11.7397L34.739 1.92076ZM50.3861 49.5137H50.3873H50.3861ZM34.739 1.00647C34.5942 1.00647 34.451 1.04085 34.3215 1.10736L15.2077 10.9176C15.1284 10.9546 15.0501 10.9947 14.9695 11.04L14.9244 11.0631C14.8808 11.0854 14.8397 11.1108 14.8011 11.139C12.4487 12.6286 12.6827 16.3406 12.728 16.8608L12.7035 21.4448C12.7018 21.766 12.8687 22.0645 13.1432 22.2312C13.2887 22.3195 13.4531 22.364 13.6178 22.364C13.7639 22.364 13.9102 22.3291 14.0438 22.2587L20.4266 18.8976L21.1184 51.6915C21.1162 51.9413 21.1256 54.131 22.0509 56.5484C23.3553 59.956 25.8477 62.1765 29.2587 62.9698C29.3281 62.9859 29.3977 62.9937 29.4663 62.9937C29.8481 62.9937 30.2 62.7532 30.3298 62.3783L30.8573 60.854L33.785 60.4685C34.1266 60.4235 34.4139 60.1903 34.5282 59.8653L35.0744 58.3116L38.0705 57.9597C38.4169 57.919 38.7099 57.6848 38.826 57.356L39.3727 55.8073L42.412 55.3404C42.7529 55.288 43.0354 55.0483 43.1424 54.7204L43.6079 53.2938L46.5714 52.8191C46.8927 52.7676 47.1625 52.5492 47.2797 52.2457L47.859 50.7457L50.4857 50.4227C50.9439 50.373 51.3009 49.985 51.3009 49.5138C51.3009 49.0293 50.9246 48.6328 50.4481 48.6015C50.4279 48.6001 50.4075 48.5994 50.387 48.5994C50.3755 48.5994 50.364 48.5997 50.3524 48.6001C50.3384 48.6006 50.3245 48.6014 50.3107 48.6025C50.2928 48.6037 50.2013 48.609 50.0568 48.609C48.6388 48.609 45.2863 48.2004 44.7827 44.4148C44.0772 39.1117 44.7831 13.2775 44.7903 13.0172C44.7907 13.003 44.7908 12.9888 44.7905 12.9746C44.7884 12.863 44.7249 10.2091 43.4994 7.41458C41.8326 3.61397 38.8338 1.39952 34.8272 1.01079C34.7978 1.00789 34.7684 1.00647 34.739 1.00647Z" fill="#374874"/>
<path fill-rule="evenodd" clip-rule="evenodd" d="M39.7046 18.6847C39.7654 18.7953 39.7249 18.9343 39.6143 18.995L26.333 26.2861C26.2223 26.3469 26.0834 26.3064 26.0226 26.1958C25.9619 26.0851 26.0023 25.9461 26.113 25.8854L39.3943 18.5943C39.5049 18.5335 39.6439 18.574 39.7046 18.6847Z" fill="#374874"/>
<path fill-rule="evenodd" clip-rule="evenodd" d="M39.7046 25.9873C39.7654 26.0979 39.7249 26.2369 39.6143 26.2976L26.333 33.5887C26.2223 33.6495 26.0834 33.609 26.0226 33.4984C25.9619 33.3877 26.0023 33.2488 26.113 33.188L39.3943 25.8969C39.5049 25.8362 39.6439 25.8766 39.7046 25.9873Z" fill="#374874"/>
<path fill-rule="evenodd" clip-rule="evenodd" d="M39.7046 33.2899C39.7654 33.4005 39.7249 33.5395 39.6143 33.6002L26.333 40.8913C26.2223 40.9521 26.0834 40.9116 26.0226 40.801C25.9619 40.6903 26.0023 40.5514 26.113 40.4906L39.3943 33.1995C39.5049 33.1388 39.6439 33.1792 39.7046 33.2899Z" fill="#374874"/>
<path fill-rule="evenodd" clip-rule="evenodd" d="M39.7046 40.5925C39.7654 40.7032 39.7249 40.8421 39.6143 40.9029L26.333 48.194C26.2223 48.2547 26.0834 48.2143 26.0226 48.1036C25.9619 47.9929 26.0023 47.854 26.113 47.7932L39.3943 40.5021C39.5049 40.4414 39.6439 40.4818 39.7046 40.5925Z" fill="#374874"/>
</svg>

```

## File: static\src\js\purchase_datetimepicker.js

```javascript
/** @odoo-module */
import PublicWidget from "@web/legacy/js/public/public_widget";

export const PurchaseDatePicker = PublicWidget.Widget.extend({
    selector: ".o-purchase-datetimepicker",
    init() {
        this._super(...arguments);
        this.rpc = this.bindService("rpc");
    },
    start() {
        this.call("datetime_picker", "create", {
            target: this.el,
            onChange: (newDate) => {
                const { accessToken, orderId, lineId } = this.el.dataset;
                this.rpc(`/my/purchase/${orderId}/update?access_token=${accessToken}`, {
                    [lineId]: newDate.toISODate(),
                });
            },
            pickerProps: {
                type: "date",
                value: luxon.DateTime.fromISO(this.el.dataset.value),
            },
        }).enable();
    },
});

PublicWidget.registry.PurchaseDatePicker = PurchaseDatePicker;

```

## File: static\src\js\purchase_portal_sidebar.js

```javascript
/** @odoo-module **/

import publicWidget from "@web/legacy/js/public/public_widget";
import PortalSidebar from "@portal/js/portal_sidebar";
import { uniqueId } from "@web/core/utils/functions";

publicWidget.registry.PurchasePortalSidebar = PortalSidebar.extend({
    selector: ".o_portal_purchase_sidebar",

    /**
     * @constructor
     */
    init: function (parent, options) {
        this._super.apply(this, arguments);
        this.authorizedTextTag = ["em", "b", "i", "u"];
        this.spyWatched = $('body[data-target=".navspy"]');
    },
    /**
     * @override
     */
    start: function () {
        var def = this._super.apply(this, arguments);
        var $spyWatcheElement = this.$el.find('[data-id="portal_sidebar"]');
        this._setElementId($spyWatcheElement);
        // Nav Menu ScrollSpy
        this._generateMenu();
        return def;
    },

    //--------------------------------------------------------------------------
    // Private
    //---------------------------------------------------------------------------

    /**
     * create an unique id and added as a attribute of spyWatched element
     *
     * @private
     * @param {string} prefix
     * @param {Object} $el
     *
     */
    _setElementId: function (prefix, $el) {
        var id = uniqueId(prefix);
        this.spyWatched.find($el).attr("id", id);
        return id;
    },
    /**
     * generate the new spy menu
     *
     * @private
     *
     */
    _generateMenu: function () {
        var self = this,
            lastLI = false,
            lastUL = null,
            $bsSidenav = this.$el.find(".bs-sidenav");

        $("#quote_content [id^=quote_header_], #quote_content [id^=quote_]", this.spyWatched).attr(
            "id",
            ""
        );
        this.spyWatched
            .find("#quote_content h2, #quote_content h3")
            .toArray()
            .forEach((el) => {
                var id, text;
                switch (el.tagName.toLowerCase()) {
                    case "h2":
                        id = self._setElementId("quote_header_", el);
                        text = self._extractText($(el));
                        if (!text) {
                            break;
                        }
                        lastLI = $("<li class='nav-item'>")
                            .append(
                                $(
                                    '<a class="nav-link p-0" style="max-width: 200px;" href="#' +
                                        id +
                                        '"/>'
                                ).text(text)
                            )
                            .appendTo($bsSidenav);
                        lastUL = false;
                        break;
                    case "h3":
                        id = self._setElementId("quote_", el);
                        text = self._extractText($(el));
                        if (!text) {
                            break;
                        }
                        if (lastLI) {
                            if (!lastUL) {
                                lastUL = $("<ul class='nav flex-column'>").appendTo(lastLI);
                            }
                            $("<li class='nav-item'>")
                                .append(
                                    $(
                                        '<a class="nav-link p-0" style="max-width: 200px;" href="#' +
                                            id +
                                            '"/>'
                                    ).text(text)
                                )
                                .appendTo(lastUL);
                        }
                        break;
                }
                el.setAttribute("data-anchor", true);
            });
        this.trigger_up("widgets_start_request", { $target: $bsSidenav });
    },
    /**
     * extract text of menu title for sidebar
     *
     * @private
     * @param {Object} $node
     *
     */
    _extractText: function ($node) {
        var self = this;
        var rawText = [];
        Array.from($node.contents()).forEach((el) => {
            var current = $(el);
            if ($.trim(current.text())) {
                var tagName = current.prop("tagName");
                if (
                    typeof tagName === "undefined" ||
                    (typeof tagName !== "undefined" &&
                        self.authorizedTextTag.includes(tagName.toLowerCase()))
                ) {
                    rawText.push($.trim(current.text()));
                }
            }
        });
        return rawText.join(" ");
    },
});

```

## File: static\src\js\tours\purchase.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { registry } from "@web/core/registry";
import { stepUtils } from "@web_tour/tour_service/tour_utils";

import PurchaseAdditionalTourSteps from "@purchase/js/tours/purchase_steps";

registry.category("web_tour.tours").add("purchase_tour", {
    url: "/web",
    sequence: 40,
    steps: () => [
        stepUtils.showAppsMenuItem(),
        {
            trigger: '.o_app[data-menu-xmlid="purchase.menu_purchase_root"]',
            content: _t(
                "Let's try the Purchase app to manage the flow from purchase to reception and invoice control."
            ),
            position: "right",
            edition: "community",
        },
        {
            trigger: '.o_app[data-menu-xmlid="purchase.menu_purchase_root"]',
            content: _t(
                "Let's try the Purchase app to manage the flow from purchase to reception and invoice control."
            ),
            position: "bottom",
            edition: "enterprise",
        },
        {
            trigger: ".o_list_button_add",
            extra_trigger: ".o_purchase_order",
            content: _t("Let's create your first request for quotation."),
            position: "bottom",
        },
        {
            trigger: ".o_form_editable .o_field_res_partner_many2one[name='partner_id']",
            extra_trigger: ".o_purchase_order",
            content: _t("Search a vendor name, or create one on the fly."),
            position: "bottom",
            run: function (actions) {
                actions.text("Azure Interior", this.$anchor.find("input"));
            },
        },
        {
            trigger: ".ui-menu-item > a:contains('Azure Interior')",
            auto: true,
            in_modal: false,
        },
        {
            trigger: ".o_field_x2many_list_row_add > a",
            content: _t("Add some products or services to your quotation."),
            position: "bottom",
        },
        {
            trigger: ".o_field_widget[name=product_id], .o_field_widget[name=product_template_id]",
            extra_trigger: ".o_purchase_order",
            content: _t("Select a product, or create a new one on the fly."),
            position: "right",
            run: function (actions) {
                var $input = this.$anchor.find("input");
                actions.text("DESK0001", $input.length === 0 ? this.$anchor : $input);
            },
        },
        {
            trigger: "a:contains('DESK0001')",
            auto: true,
        },
        {
            trigger: "td[name='name'][data-tooltip*='DESK0001']",
            auto: true,
            run: function () {}, // wait for product creation
        },
        {
            trigger: "div.o_field_widget[name='product_qty'] input ",
            extra_trigger: ".o_purchase_order",
            content: _t("Indicate the product quantity you want to order."),
            position: "right",
            run: "text 12.0",
        },
        ...stepUtils.statusbarButtonsSteps(
            "Send by Email",
            _t("Send the request for quotation to your vendor."),
            ".o_statusbar_buttons .o_arrow_button_current[name='action_rfq_send']"
        ),
        {
            trigger: ".modal-content",
            auto: true,
            run: function (actions) {
                // Check in case user must add email to vendor
                var $input = $(".modal-content input[name='email']");
                if ($input.length) {
                    actions.text("agrolait@example.com", $input);
                    actions.click($(".modal-footer button"));
                }
            },
        },
        {
            trigger: ".modal-footer button[name='action_send_mail']",
            extra_trigger: ".modal-footer button[name='action_send_mail']",
            content: _t("Send the request for quotation to your vendor."),
            position: "left",
            run: "click",
        },
        {
            content: "Select price",
            trigger: 'tbody tr.o_data_row .o_list_number[name="price_unit"]',
        },
        {
            trigger: 'tbody tr.o_data_row .o_list_number[name="price_unit"] input',
            extra_trigger: ".o_purchase_order",
            content: _t(
                "Once you get the price from the vendor, you can complete the purchase order with the right price."
            ),
            position: "right",
            run: "text 200.00",
        },
        {
            auto: true,
            trigger: ".o_purchase_order",
            run: "click",
        },
        ...stepUtils.statusbarButtonsSteps("Confirm Order", _t("Confirm your purchase.")),
        ...new PurchaseAdditionalTourSteps()._get_purchase_stock_steps(),
    ],
});

```

## File: static\src\js\tours\purchase_steps.js

```javascript
/** @odoo-module **/

class PurchaseAdditionalTourSteps {
    _get_purchase_stock_steps() {
        return [
            {
                auto: true, // Useless final step to trigger congratulation message
                trigger: ".o_purchase_order",
            },
        ];
    }
}

export default PurchaseAdditionalTourSteps;

```

## File: static\src\product_catalog\kanban_record.js

```javascript
/** @odoo-module */
import { ProductCatalogKanbanRecord } from "@product/product_catalog/kanban_record";
import { ProductCatalogPurchaseOrderLine } from "./purchase_order_line/purchase_order_line";
import { patch } from "@web/core/utils/patch";
import { useService } from "@web/core/utils/hooks";
import { useSubEnv } from "@odoo/owl";

patch(ProductCatalogKanbanRecord.prototype, {
    setup() {
        super.setup();
        this.orm = useService("orm");
        useSubEnv({
            updatePackagingQuantity: this.updatePackagingQuantity.bind(this),
        });
    },

    get orderLineComponent() {
        if (this.env.orderResModel === "purchase.order") {
            return ProductCatalogPurchaseOrderLine;
        }
        return super.orderLineComponent;
    },

    addProduct() {
        if (this.productCatalogData.quantity === 0 && this.productCatalogData.min_qty) {
            super.addProduct(this.productCatalogData.min_qty);
        } else {
            super.addProduct(...arguments);
        }
    },

    async updatePackagingQuantity(packaging) {
        const productPackagingQty =
            Math.floor(this.productCatalogData.quantity / packaging.qty) + 1;
        this.productCatalogData.quantity = productPackagingQty * packaging.qty;
        const price = await this.rpc("/product/catalog/update_order_line_info", {
            order_id: this.env.orderId,
            product_id: this.env.productId,
            product_packaging_id: packaging.id,
            product_packaging_qty: productPackagingQty,
            res_model: this.env.orderResModel,
        });
        this.productCatalogData.price = parseFloat(price);
    },
});

```

## File: static\src\product_catalog\purchase_order_line\purchase_order_line.js

```javascript
/** @odoo-module */
import { ProductCatalogOrderLine } from "@product/product_catalog/order_line/order_line";

export class ProductCatalogPurchaseOrderLine extends ProductCatalogOrderLine {
    static template = "ProductCatalogPurchaseOrderLine";
    static props = {
        ...ProductCatalogPurchaseOrderLine.props,
        min_qty: { type: Number, optional: true },
        packaging: { type: Object, optional: true },
        purchase_uom: { type: Object, optional: true },
        uom: Object,
    };

    get highlightUoM() {
        return true;
    }

    addPackagingQty() {
        this.env.updatePackagingQuantity(this.props.packaging);
    }
}

```

## File: static\src\product_catalog\purchase_order_line\purchase_order_line.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">
    <t t-name="ProductCatalogPurchaseOrderLine"
       t-inherit="product.ProductCatalogOrderLine"
       t-inherit-mode="primary">
        <xpath expr="//span[hasclass('o_product_catalog_price')]" position="attributes">
            <attribute name="t-if">!this.env.displayUoM</attribute>
        </xpath>
        <xpath expr="//span[hasclass('o_product_catalog_price')]" position="after">
            <span class="o_product_catalog_price" t-if="env.displayUoM">
                Price: <t t-out="price"/>
                / <span t-att-class="{'fw-bold text-primary': props.purchase_uom}">
                    <t t-if="props.purchase_uom" t-esc="props.purchase_uom.display_name"/>
                    <t t-else="" t-esc="props.uom.display_name"/>
                </span>
            </span>
        </xpath>
        <xpath expr="//div[hasclass('o_product_catalog_quantity')]" position="inside">
            <button t-if="props.packaging"
                    class="o_product_catalog_package_quantity btn btn-primary border"
                    t-on-click.stop="addPackagingQty">
                <i class="fa fa-cube"/>
            </button>
        </xpath>
    </t>
</templates>

```

## File: static\src\toaster_button\toaster_button_widget.js

```javascript
/** @odoo-module */
import { registry } from "@web/core/registry";
import { useService } from "@web/core/utils/hooks";
import { Component } from "@odoo/owl";

class ButtonWithNotification extends Component {
    setup() {
        this.orm = useService("orm");
        this.notification = useService("notification");
    }

    async onClick() {
        const result = await this.orm.call(this.props.record.resModel, this.props.method, [
            this.props.record.resId,
        ]);
        const message = result.toast_message;
        this.notification.add(message, { type: "success" });
    }
}
ButtonWithNotification.template = "purchase.ButtonWithNotification";

export const buttonWithNotification = {
    component: ButtonWithNotification,
    extractProps: ({ attrs }) => {
        return {
            method: attrs.button_name,
            title: attrs.title,
        };
    },
};
registry.category("view_widgets").add("toaster_button", buttonWithNotification);

```

## File: static\src\toaster_button\toaster_button_widget.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<template>

    <t t-name="purchase.ButtonWithNotification">
        <button t-on-click="onClick" t-att-name="props.method" type="button" class="btn oe_inline btn-link" t-att-title="props.title">
            <i class="fa fa-fw o_button_icon fa-info-circle"></i>
        </button>
    </t>

</template>

```

## File: static\src\views\purchase_dashboard.js

```javascript
/** @odoo-module */
import { useService } from "@web/core/utils/hooks";
import { Component, onWillStart } from "@odoo/owl";

export class PurchaseDashBoard extends Component {
    setup() {
        this.orm = useService("orm");
        this.action = useService("action");

        onWillStart(async () => {
            this.purchaseData = await this.orm.call("purchase.order", "retrieve_dashboard");
        });
    }

    /**
     * This method clears the current search query and activates
     * the filters found in `filter_name` attibute from button pressed
     */
    setSearchContext(ev) {
        const filter_name = ev.currentTarget.getAttribute("filter_name");
        const filters = filter_name.split(",");
        const searchItems = this.env.searchModel.getSearchItems((item) =>
            filters.includes(item.name)
        );
        this.env.searchModel.query = [];
        for (const item of searchItems) {
            this.env.searchModel.toggleSearchItem(item.id);
        }
    }
}

PurchaseDashBoard.template = "purchase.PurchaseDashboard";

```

## File: static\src\views\purchase_dashboard.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-name="purchase.PurchaseDashboard">
        <div class="o_purchase_dashboard container-fluid py-4 border-bottom bg-view">
            <div class="row justify-content-between gap-3 gap-lg-0">
                <div class="col-12 col-lg-5 col-xl-5 col-xxl-4 flex-grow-1 flex-lg-grow-0 flex-shrink-0">
                    <div class="grid gap-4">
                        <div class="g-col-3 g-col-sm-2 d-flex align-items-center py-2 justify-content-end text-end justify-content-lg-start text-lg-start text-break">
                            All RFQs
                        </div>
                        <div class="g-col-9 g-col-sm-10 grid gap-1">
                            <div class="g-col-4 p-0" t-on-click="setSearchContext" title="All Draft RFQs" filter_name="draft_rfqs">
                                <a href="#" class="btn btn-primary w-100 h-100 border-0 rounded-0 text-capitalize text-break fw-normal">
                                    <div class="fs-2" t-out="purchaseData['all_to_send']"/>To Send
                                </a>
                            </div>
                            <div class="g-col-4 p-0" t-on-click="setSearchContext" title="All Waiting RFQs" filter_name="waiting_rfqs">
                                <a href="#" class="btn btn-primary w-100 h-100 border-0 rounded-0 text-capitalize text-break fw-normal">
                                    <div class="fs-2" t-out="purchaseData['all_waiting']"/>Waiting
                                </a>
                            </div>
                            <div class="g-col-4 p-0" t-on-click="setSearchContext" title="All Late RFQs" filter_name="late_rfqs">
                                <a href="#" class="btn btn-primary w-100 h-100 border-0 rounded-0 text-capitalize text-break fw-normal">
                                    <div class="fs-2" t-out="purchaseData['all_late']"/>Late
                                </a>
                            </div>
                        </div>
                    </div>
                    <div class="grid gap-4">
                        <div class="g-col-3 g-col-sm-2 d-flex align-items-center py-2 justify-content-end text-end justify-content-lg-start text-lg-start text-break">
                            My RFQs
                        </div>
                        <div class="g-col-9 g-col-sm-10 grid gap-2">
                            <div class="g-col-4 p-0" t-on-click="setSearchContext" title="My Draft RFQs" filter_name="draft_rfqs,my_purchases">
                                <a href="#" class="btn btn-light d-flex align-items-center w-100 h-100 p-0 border-0 bg-100 fw-normal">
                                    <div class="w-100 p-2" t-out="purchaseData['my_to_send']"/>
                                </a>
                            </div>
                            <div class="g-col-4 p-0" t-on-click="setSearchContext" title="My Waiting RFQs" filter_name="waiting_rfqs,my_purchases">
                                <a href="#" class="btn btn-light d-flex align-items-center w-100 h-100 p-0 border-0 bg-100 fw-normal">
                                    <div class="w-100 p-2" t-out="purchaseData['my_waiting']"/>
                                </a>
                            </div>
                            <div class="g-col-4 p-0" t-on-click="setSearchContext" title="My Late RFQs" filter_name="late_rfqs,my_purchases">
                                <a href="#" class="btn btn-light d-flex align-items-center w-100 h-100 p-0 border-0 bg-100 fw-normal">
                                    <div class="w-100 p-2" t-out="purchaseData['my_late']"/>
                                </a>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="col-12 col-lg-7 col-xl-6 col-xxl-5 flex-shrink-0">
                    <div class="d-flex flex-column justify-content-between gap-2 h-100">
                        <div class="grid gap-2 h-100">
                            <div class="g-col-6 g-col-md-6 grid gap-1 gap-md-4">
                                <div class="g-col-12 g-col-sm-4 g-col-lg-6 d-flex align-items-center justify-content-center text-center justify-content-md-end text-md-end mt-4 mt-sm-0 text-break">
                                    Avg Order Value
                                </div>
                                <div class="g-col-12 g-col-sm-8 g-col-lg-5 d-flex align-items-center justify-content-center py-2 bg-100">
                                    <span><t t-out="purchaseData['all_avg_order_value']"/></span>
                                </div>
                            </div>
                            <div class="g-col-6 g-col-md-6 grid gap-1 gap-md-4">
                                <div class="g-col-12 g-col-sm-4 g-col-lg-6 d-flex align-items-center py-2 justify-content-center text-center justify-content-md-end text-md-end mt-4 mt-sm-0 text-break">
                                    Purchased Last 7 Days
                                </div>
                                <div class="g-col-12 g-col-sm-8 g-col-lg-6 d-flex align-items-center justify-content-center py-2 bg-100">
                                    <span><t t-out="purchaseData['all_total_last_7_days']"/></span>
                                </div>
                            </div>
                        </div>
                        <div class="grid gap-2 h-100">
                            <div class="g-col-6 g-col-md-6 grid gap-1 gap-md-4">
                                <div class="g-col-12 g-col-sm-4 g-col-lg-6 d-flex align-items-center justify-content-center text-center justify-content-md-end text-md-end mt-4 mt-sm-0 text-break">
                                    Lead Time to Purchase
                                </div>
                                <div class="g-col-12 g-col-sm-8 g-col-lg-5 d-flex align-items-center justify-content-center py-2 bg-100">
                                    <span><t t-out="purchaseData['all_avg_days_to_purchase']"/> &#160;Days</span>
                                </div>
                            </div>
                            <div class="g-col-6 g-col-md-6 grid gap-1 gap-md-4">
                                <div class="g-col-12 g-col-md-4 g-col-sm-4 g-col-lg-6 d-flex align-items-center justify-content-center text-center justify-content-md-end text-md-end mt-4 mt-sm-0 text-break">
                                    RFQs Sent Last 7 Days
                                </div>
                                <div class="g-col-12 g-col-sm-8 g-col-lg-6 d-flex align-items-center justify-content-center py-2 bg-100">
                                    <span><t t-out="purchaseData['all_sent_rfqs']"/></span>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </t>
</templates>

```

## File: static\src\views\purchase_kanbanview.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { kanbanView } from '@web/views/kanban/kanban_view';
import { KanbanRenderer } from '@web/views/kanban/kanban_renderer';
import { PurchaseDashBoard } from '@purchase/views/purchase_dashboard';


export class PurchaseDashBoardKanbanRenderer extends KanbanRenderer {};

PurchaseDashBoardKanbanRenderer.template = 'purchase.PurchaseKanbanView';
PurchaseDashBoardKanbanRenderer.components= Object.assign({}, KanbanRenderer.components, {PurchaseDashBoard})

export const PurchaseDashBoardKanbanView = {
    ...kanbanView,
    Renderer: PurchaseDashBoardKanbanRenderer,
};

registry.category("views").add("purchase_dashboard_kanban", PurchaseDashBoardKanbanView);

```

## File: static\src\views\purchase_kanbanview.xml

```xml
<templates>
    <t t-name="purchase.PurchaseKanbanView" t-inherit="web.KanbanRenderer" t-inherit-mode="primary">
        <xpath expr="//div[hasclass('o_kanban_renderer')]" position="before">
            <PurchaseDashBoard />
        </xpath>
    </t>
</templates>

```

## File: static\src\views\purchase_listview.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { listView } from "@web/views/list/list_view";
import { ListRenderer } from "@web/views/list/list_renderer";
import { PurchaseDashBoard } from '@purchase/views/purchase_dashboard';

export class PurchaseDashBoardRenderer extends ListRenderer {};

PurchaseDashBoardRenderer.template = 'purchase.PurchaseListView';
PurchaseDashBoardRenderer.components= Object.assign({}, ListRenderer.components, {PurchaseDashBoard})

export const PurchaseDashBoardListView = {
    ...listView,
    Renderer: PurchaseDashBoardRenderer,
};

registry.category("views").add("purchase_dashboard_list", PurchaseDashBoardListView);

```

## File: static\src\views\purchase_listview.xml

```xml
<templates>
    <t t-name="purchase.PurchaseListView" t-inherit="web.ListRenderer" t-inherit-mode="primary">
        <xpath expr="//div[hasclass('o_list_renderer')]" position="before">
            <PurchaseDashBoard />
        </xpath>
    </t>
</templates>

```

## File: views\account_move_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_move_form_inherit_purchase" model="ir.ui.view">
        <field name="name">account.move.inherit.purchase</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_move_form"/>
        <field name="arch" type="xml">
            <!-- Auto-complete could be done from either a bill either a purchase order -->
            <field name="invoice_vendor_bill_id" position="after">
                <t groups="purchase.group_purchase_user">
                    <field name="purchase_id" invisible="1" readonly="state != 'draft'"/>
                    <label for="purchase_vendor_bill_id" string="Auto-Complete" class="oe_edit_only"
                        invisible="state != 'draft' or move_type != 'in_invoice'" />
                    <field name="purchase_vendor_bill_id" nolabel="1"
                        invisible="state != 'draft' or move_type != 'in_invoice'"
                        readonly="state != 'draft'"
                        class="oe_edit_only"
                        domain="partner_id and [('partner_id.commercial_partner_id', '=', commercial_partner_id)] or []"
                        placeholder="Select a purchase order or an old bill"
                        context="{'show_total_amount': True}"
                        options="{'no_create': True, 'no_open': True}"/>
                </t>
            </field>
            <label name="invoice_vendor_bill_id_label" position="attributes">
                <attribute name="groups">!purchase.group_purchase_user</attribute>
            </label>
            <field name="invoice_vendor_bill_id" position="attributes">
                <attribute name="groups">!purchase.group_purchase_user</attribute>
            </field>
            <!-- Add link to purchase_line_id to account.move.line -->
            <xpath expr="//field[@name='invoice_line_ids']/tree/field[@name='company_id']" position="after">
                <field name="purchase_line_id" groups="purchase.group_purchase_user" column_invisible="True"/>
                <field name="purchase_order_id" groups="purchase.group_purchase_user" column_invisible="parent.move_type != 'in_invoice'" optional="hide"/>
            </xpath>
            <xpath expr="//field[@name='line_ids']/tree/field[@name='company_id']" position="after">
                <field name="purchase_line_id" groups="purchase.group_purchase_user" column_invisible="True"/>
            </xpath>
            <xpath expr="//div[@name='button_box']" position="inside">
                <button class="oe_stat_button"
                        name="action_view_source_purchase_orders"
                        type="object"
                        groups="purchase.group_purchase_user"
                        icon="fa-pencil-square-o"
                        invisible="purchase_order_count == 0 or move_type not in ('out_invoice', 'out_refund', 'in_invoice', 'in_refund', 'out_receipt', 'in_receipt')">
                    <field string="Purchases" name="purchase_order_count" widget="statinfo"/>
                </button>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\analytic_account_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="account_analytic_account_view_form_purchase" model="ir.ui.view">
        <field name="name">account.analytic.account.form.purchase</field>
        <field name="model">account.analytic.account</field>
        <field name="inherit_id" ref="analytic.view_account_analytic_account_form"/>
        <field eval="10" name="priority"/>
        <field name="arch" type="xml">
            <div name="button_box" position="inside">
                <button class="oe_stat_button" type="object" name="action_view_purchase_orders"
                    icon="fa-credit-card" invisible="purchase_order_count == 0"
                    groups="purchase.group_purchase_user">
                    <field string="Purchase Orders" name="purchase_order_count" widget="statinfo"/>
                </button>
            </div>
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
      <li t-if="page_name == 'rfq' or order and order.state == 'sent'" t-attf-class="breadcrumb-item #{'active ' if not order else ''}">
        <a t-if="order" t-attf-href="/my/rfq?{{ keep_query() }}">Requests for Quotation</a>
        <t t-else="">Requests for Quotation</t>
      </li>
      <li t-if="page_name == 'purchase' or order and order.state != 'sent'" t-attf-class="breadcrumb-item #{'active ' if not order else ''}">
        <a t-if="order" t-attf-href="/my/purchase?{{ keep_query() }}">Purchase Orders</a>
        <t t-else="">Purchase Orders</t>
      </li>
      <li t-if="order" class="breadcrumb-item active">
        <t t-esc="order.name"/>
      </li>
    </xpath>
  </template>

  <template id="portal_my_home_purchase" name="Show Requests for Quotation / Purchase Orders" customize_show="True" inherit_id="portal.portal_my_home" priority="25">
    <xpath expr="//div[hasclass('o_portal_docs')]" position="before">
          <t t-set="portal_vendor_category_enable" t-value="True"/>
      </xpath>
      <div id="portal_vendor_category" position="inside">
          <t t-call="portal.portal_docs_entry">
              <t t-set="icon" t-value="'/purchase/static/src/img/rfq.svg'"/>
              <t t-set="text">Follow your Requests for Quotation</t>
              <t t-set="title">Requests for Quotation</t>
              <t t-set="url" t-value="'/my/rfq'"/>
              <t t-set="placeholder_count" t-value="'rfq_count'"/>
          </t>
          <t t-call="portal.portal_docs_entry">
              <t t-set="icon" t-value="'/purchase/static/src/img/calculator.svg'"/>
              <t t-set="text">Follow orders you have to fulfill</t>
              <t t-set="title">Our Orders</t>
              <t t-set="url" t-value="'/my/purchase'"/>
              <t t-set="placeholder_count" t-value="'purchase_count'"/>
          </t>
      </div>
  </template>

  <template id="portal_my_purchase_rfqs" name="My Requests For Quotation">
      <t t-call="portal.portal_layout">
          <t t-set="breadcrumbs_searchbar" t-value="True"/>

          <t t-call="portal.portal_searchbar">
              <t t-set="title" >Requests For Quotation</t>
          </t>
          <t t-if="not rfqs">
              <p class="alert alert-warning">There are currently no requests for quotation for your account.</p>
          </t>
          <t t-if="rfqs" t-call="portal.portal_table">
              <thead>
                  <tr class="active">
                    <th>
                        <span class='d-none d-md-inline'>Request for Quotation #</span>
                        <span class='d-block d-md-none'>Ref.</span>
                    </th>
                      <th class="text-end">Order Deadline</th>
                      <th class="text-end">Total</th>
                  </tr>
              </thead>
              <t t-foreach="rfqs" t-as="rfq">
                  <tr>
                      <td><a t-att-href="rfq.get_portal_url()"><t t-esc="rfq.name"/></a></td>
                      <td class="text-end">
                          <span t-field="rfq.date_order" t-options="{'widget': 'date'}"/>&amp;nbsp;
                          <span class='d-none d-md-inline' t-field="rfq.date_order" t-options="{'time_only': True}"/>
                      </td>
                      <td class="text-end">
                          <span t-field="rfq.amount_total"/>
                      </td>
                  </tr>
              </t>
          </t>
      </t>
  </template>

  <template id="portal_my_purchase_orders" name="My Purchase Orders">
      <t t-call="portal.portal_layout">
          <t t-set="breadcrumbs_searchbar" t-value="True"/>

          <t t-call="portal.portal_searchbar">
              <t t-set="title">Purchase Orders</t>
          </t>
          <t t-if="not orders">
              <p class="alert alert-warning">There are currently no purchase orders for your account.</p>
          </t>
          <t t-if="orders" t-call="portal.portal_table">
              <thead>
                  <tr class="active">
                      <th>
                          <span class='d-none d-md-inline'>Purchase Order #</span>
                          <span class='d-block d-md-none'>Ref.</span>
                      </th>
                      <th class="text-end">
                          <span class='d-none d-md-inline'>Confirmation Date</span>
                          <span class='d-block d-md-none'>Confirmation</span>
                      </th>
                      <th class="text-center"/>
                      <th class="text-end">Total</th>
                  </tr>
              </thead>
              <t t-foreach="orders" t-as="order">
                  <tr>
                      <td><a t-att-href="order.get_portal_url()"><t t-esc="order.name"/></a></td>
                      <td class="text-end">
                          <span t-field="order.date_approve" t-options="{'widget': 'date'}"/>&amp;nbsp;
                          <span class='d-none d-md-inline' t-field="order.date_approve" t-options="{'time_only': True}"/>
                      </td>
                      <td class="text-center">
                          <span t-if="order.invoice_status == 'to invoice'" class="badge rounded-pill text-bg-info">
                              <i class="fa fa-fw fa-file-text" role="img" aria-label="Waiting for Bill" title="Waiting for Bill"></i><span class="d-none d-md-inline"> Waiting for Bill</span>
                          </span>
                          <span t-if="order.state == 'cancel'" class="badge rounded-pill text-bg-secondary">
                              <i class="fa fa-fw fa-remove" role="img" aria-label="Cancelled" title="Cancelled"></i><span class="d-none d-md-inline"> Cancelled</span>
                          </span>
                          <span t-if="order.state == 'done'" class="badge rounded-pill text-bg-success">
                              <i class="fa fa-fw fa-check" role="img" aria-label="Done" title="Done"></i><span class="d-none d-md-inline"> Done</span>
                          </span>
                      </td>
                      <td class="text-end"><span t-field="order.amount_total"/></td>
                  </tr>
              </t>
          </t>
      </t>
  </template>

  <template id="portal_my_purchase_order" name="Purchase Order" inherit_id="portal.portal_sidebar" primary="True">
      <xpath expr="//div[hasclass('o_portal_sidebar')]" position="inside">
          <t t-set="o_portal_fullwidth_alert" groups="purchase.group_purchase_manager">
              <t t-call="portal.portal_back_in_edit_mode">
                  <t t-set="backend_url" t-value="'/web#model=%s&amp;id=%s&amp;action=%s&amp;view_type=form' % (order._name, order.id, order.env.ref('purchase.purchase_rfq').id)"/>
              </t>
          </t>

          <div class="row o_portal_purchase_sidebar">
              <!-- Sidebar -->
              <t t-call="portal.portal_record_sidebar">
                  <t t-set="classes" t-value="'col-lg-3 col-xl-4 d-print-none'"/>

                  <t t-set="title">
                      <h2 t-field="order.amount_total" data-id="total_amount"/>
                  </t>
                  <t t-set="entries">
                      <div class="d-flex flex-column gap-4">
                          <div class="flex-grow-1 order-md-last order-lg-first">
                              <div class="o_download_pdf btn-toolbar flex-sm-nowrap">
                                  <div class="btn-group flex-grow-1 mb-1">
                                      <a class="btn btn-secondary btn-block o_print_btn o_portal_invoice_print" t-att-href="order.get_portal_url(report_type='pdf')" id="print_invoice_report" title="View Details" target="_blank"><i class="fa fa-print"/> View Details</a>
                                  </div>
                              </div>
                          </div>

                          <div class="navspy flex-grow-1 ps-0" t-ignore="true" role="complementary">
                              <ul class="nav flex-column bs-sidenav"></ul>
                          </div>

                          <div t-if="order.user_id">
                              <h6><small class="text-muted">Purchase Representative</small></h6>
                              <div class="o_portal_contact_details d-flex flex-column gap-2">
                                <div class="d-flex justify-content-start align-items-center gap-2">
                                    <img class="o_avatar o_portal_contact_img rounded" t-att-src="image_data_uri(order.user_id.avatar_128)" alt="Contact"/>
                                    <div>
                                        <h6 class="mb-0" t-out="order.user_id.name"/>
                                        <a href="#discussion" class="small fw-bold">Send message</a>
                                    </div>
                                </div>
                                <div t-field="order.user_id" t-options='{"widget": "contact", "fields": ["phone"], "no_marker": False}'/>
                            </div>
                          </div>
                      </div>
                  </t>
              </t>

              <!-- Page content -->
              <div id="quote_content" class="o_portal_content col-12 col-lg-9 col-xl-8">

                  <!-- status messages -->
                  <div t-if="order.state == 'cancel'" class="alert alert-danger alert-dismissable d-print-none" role="alert">
                      <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="close"></button>
                      <strong>This purchase has been canceled.</strong>
                  </div>

                  <!-- main content -->
                  <div id="portal_purchase_content">
                      <div t-call="purchase.purchase_order_portal_content"/>
                  </div>

                  <!-- chatter -->
                  <hr/>
                  <div id="purchase_order_communication">
                      <h3>Communication history</h3>
                      <t t-call="portal.message_thread"/>
                  </div>
              </div><!-- // #quote_content -->
          </div>
      </xpath>
  </template>

  <template id="purchase_order_portal_content" name="Purchase Order Portal Content">
      <!-- Intro -->
      <div id="introduction" class="mt-5 mt-lg-0 pb-2 pt-0">
        <h2 class="my-0">
          <t t-if="order.state in ['draft', 'sent']">Request for Quotation</t>
          <t t-else="1">
            Purchase Order
          </t>
          <em t-esc="order.name"/>
        </h2>
      </div>

      <div>
          <!-- Informations -->
          <div id="informations">
              <div class="row" id="po_date">
                  <div class="mb-3 col-6">
                    <t t-if="order.state in ['draft', 'sent']">
                      <strong>Request For Quotation Date:</strong>
                    </t>
                    <t t-if="order.state in ['purchase', 'done', 'cancel']">
                      <strong>Order Date:</strong>
                    </t>
                    <span t-field="order.date_order" t-options='{"widget": "date"}'/>
                  </div>
              </div>
              <div class="row">
                  <div class="col-lg-6">
                    <strong class="d-block mb-1">From:</strong>
                    <address t-field="order.company_id.partner_id" t-options='{"widget": "contact", "fields": ["address", "name"], "no_marker": True}'/>
                    <strong>Confirmation Date:</strong> <span t-field="order.date_approve" t-options='{"widget": "date"}'/><br/>
                    <div t-att-class="'d-inline' if order.date_planned else 'd-none'">
                      <strong>Receipt Date:</strong><span class="ms-1" t-field="order.date_planned" t-options='{"widget": "date"}'/>
                    </div>
                  </div>
              </div>

              <t t-set="invoices" t-value="[i for i in order.invoice_ids if i.state not in ['draft', 'cancel']]"/>
              <div t-if="invoices" class="row">
                  <div class="col">
                      <strong class="d-block mb-1">Invoices</strong>
                      <ul class="list-group mb-4">
                          <t t-foreach="invoices" t-as="i">
                              <t t-set="report_url" t-value="i.get_portal_url(report_type='pdf', download=True)"/>
                              <div class="d-flex flex-wrap align-items-center justify-content-between">
                                  <div>
                                      <a t-att-href="report_url">
                                          <span t-esc="i.name"/>
                                      </a>
                                      <div class="small d-lg-inline-block">Date: <span class="text-muted" t-field="i.invoice_date"/></div>
                                  </div>
                                  <span t-if="i.payment_state in ('paid', 'in_payment')" class="badge text-bg-success small"><i class="fa fa-fw fa-check"/>Paid</span>
                                  <span t-elif="i.payment_state == 'reversed'" class="badge text-bg-success small"><i class="fa fa-fw fa-check"/>Reversed</span>
                                  <span t-else="" class="small badge text-bg-primary"><i class="fa fa-fw fa-clock-o"/>Waiting Payment</span>
                              </div>
                          </t>
                      </ul>
                  </div>
              </div>
          </div>

          <section id="details" style="page-break-inside: auto;" class="mt32">
              <h3 id="details">Pricing</h3>

              <div class="table-responsive">
                <table t-att-data-order-id="order.id" t-att-data-token="order.access_token" class="table table-sm" id="purchase_order_table">
                    <thead class="bg-100">
                        <tr>
                            <th class="text-start">Products</th>
                            <th class="text-end">Quantity</th>
                            <th t-if="update_dates" class="text-end">Scheduled Date</th>
                            <th t-if="not update_dates and order.state in ['purchase', 'done']" t-attf-class="text-end {{ 'd-none d-sm-table-cell' if report_type == 'html' else '' }}">Unit Price</th>
                            <th t-if="not update_dates and order.state in ['purchase', 'done']" t-attf-class="text-end {{ 'd-none d-md-table-cell' if report_type == 'html' else '' }}">
                                <span>Taxes</span>
                            </th>
                            <th class="text-end" t-if="order.state in ['purchase', 'done']" >
                                <span>Amount</span>
                            </th>
                        </tr>
                    </thead>
                    <tbody class="purchase_tbody">

                          <t t-set="current_subtotal" t-value="0"/>

                          <t t-foreach="order.order_line" t-as="line">

                              <t t-set="current_subtotal" t-value="current_subtotal + line.price_subtotal"/>

                            <tr t-att-class="'bg-200 fw-bold o_line_section' if line.display_type == 'line_section' else 'fst-italic text-break' if line.display_type == 'line_note' else ''">
                                <t t-if="not line.display_type">
                                    <td id="product_name">
                                        <img t-att-src="image_data_uri(resize_to_48(line.product_id.image_1024))" alt="Product" class="d-none d-lg-inline"/>
                                        <span t-field="line.name"/>
                                    </td>
                                    <td class="text-end">
                                        <div id="quote_qty">
                                            <span t-field="line.product_qty"/>
                                            <span t-field="line.product_uom" groups="uom.group_uom"/>
                                        </div>
                                    </td>
                                    <td t-if="update_dates" class="text-end">
                                        <div class="container">
                                          <div class="mb-3">
                                            <div class="input-group date">
                                                <input type="text"
                                                    class="form-control datetimepicker-input o-purchase-datetimepicker text-end"
                                                    t-att-data-access-token="order.access_token"
                                                    t-att-data-order-id="order.id"
                                                    t-att-data-line-id="line.id"
                                                    t-att-data-value="line.date_planned.isoformat()"
                                                />
                                            </div>
                                          </div>
                                        </div>
                                    </td>
                                    <td t-if="not update_dates and order.state in ['purchase', 'done']" t-attf-class="text-end {{ 'd-none d-sm-table-cell' if report_type == 'html' else '' }}">
                                        <div
                                            t-field="line.price_unit"
                                            class="text-end"
                                        />
                                    </td>
                                    <td t-if="not update_dates and order.state in ['purchase', 'done']" t-attf-class="text-end {{ 'd-none d-md-table-cell' if report_type == 'html' else '' }}">
                                        <span t-esc="', '.join(map(lambda x: (x.description or x.name), line.taxes_id))"/>
                                    </td>
                                    <td class="text-end" t-if="not update_dates and order.state in ['purchase', 'done']">
                                        <span class="oe_order_line_price_subtotal" t-field="line.price_subtotal"/>
                                    </td>
                                </t>
                                <t t-if="line.display_type == 'line_section'">
                                    <td colspan="99">
                                        <span t-field="line.name"/>
                                    </td>
                                    <t t-set="current_section" t-value="line"/>
                                    <t t-set="current_subtotal" t-value="0"/>
                                </t>
                                <t t-if="line.display_type == 'line_note'">
                                    <td colspan="99">
                                        <span t-field="line.name"/>
                                    </td>
                                </t>
                            </tr>

                            <t t-if="current_section and (line_last or order.order_line[line_index+1].display_type == 'line_section') and order.state in ['purchase', 'done']">
                                <tr class="is-subtotal text-end">
                                    <td colspan="99">
                                        <strong class="mr16">Subtotal</strong>
                                        <span
                                            t-esc="current_subtotal"
                                            t-options='{"widget": "monetary", "display_currency": order.currency_id}'
                                        />
                                    </td>
                                </tr>
                            </t>
                        </t>
                    </tbody>
                </table>
              </div>

              <div id="total" t-if="order.state in ['purchase', 'done']" class="row" name="total" style="page-break-inside: avoid;">
                  <div t-attf-class="#{'col-4' if report_type != 'html' else 'col-sm-7 col-md-5'} ms-auto">
                      <t t-call="purchase.purchase_order_portal_content_totals_table"/>
                  </div>
              </div>
          </section>

          <section id="terms" class="mt-5" t-if="order.notes">
              <h3 class="">Terms &amp; Conditions</h3>
              <hr class="mt-0 mb-1"/>
              <em t-field="order.notes"/>
          </section>

          <section class="mt-5" t-if="order.payment_term_id">
              <h3 class="">Payment terms</h3>
              <hr class="mt-0 mb-1"/>
              <span t-field="order.payment_term_id"/>
          </section>
      </div>
  </template>

  <template id="purchase_order_portal_content_totals_table">
      <table class="table table-sm">
          <t t-set="tax_totals" t-value="order.tax_totals"/>
          <t t-call="purchase.document_tax_totals"/>
      </table>
  </template>

  <template id="portal_my_purchase_order_update_date" name="Portal: My Purchase Order Update Dates" inherit_id="purchase.portal_my_purchase_order" primary="True">
    <xpath expr="////div[@id='portal_purchase_content']" position="replace">
      <div id="portal_purchase_content">
        <t t-set="update_dates" t-value="True"/>
        <div t-call="purchase.purchase_order_portal_content"/>
      </div>
    </xpath>
  </template>

</odoo>

```

## File: views\product_packaging_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="product_packaging_form_view_purchase" model="ir.ui.view">
        <field name="name">product.packaging.form.view.purchase</field>
        <field name="model">product.packaging</field>
        <field name="inherit_id" ref="product.product_packaging_form_view"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='group_product']" position="inside">
                <field name="purchase"/>
            </xpath>
        </field>
    </record>

    <record id="product_packaging_tree_view_purchase" model="ir.ui.view">
        <field name="name">product.packaging.tree.view.purchase</field>
        <field name="model">product.packaging</field>
        <field name="inherit_id" ref="product.product_packaging_tree_view"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='product_uom_id']" position="after">
                <field name="purchase" optional="show"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\product_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

        <!-- Product Suppliers-->
        <record id="product_supplierinfo_tree_view2" model="ir.ui.view">
            <field name="name">product.supplierinfo.tree.view2</field>
            <field name="model">product.supplierinfo</field>
            <field name="inherit_id" ref="product.product_supplierinfo_tree_view"/>
            <field name="mode">primary</field>
            <field name="priority" eval="1000"/>
            <field name="arch" type="xml">
                <xpath expr="//tree" position="attributes">
                    <attribute name="editable">bottom</attribute>
                </xpath>
                <xpath expr="//tree" position="inside">
                    <field name="company_id" column_invisible="True"/>
                </xpath>
                <xpath expr="//field[@name='company_id']" position="attributes">
                    <attribute name="readonly">0</attribute>
                    <attribute name="optional">hide</attribute>
                </xpath>
                <xpath expr="//field[@name='partner_id']" position="attributes">
                    <attribute name="readonly">0</attribute>
                </xpath>
                <xpath expr="//field[@name='product_id']" position="attributes">
                    <attribute name="readonly">0</attribute>
                    <attribute name="options">{'no_create': True, 'no_open': True}</attribute>
                </xpath>
                <xpath expr="//field[@name='delay']" position="attributes">
                    <attribute name="optional">show</attribute>
                </xpath>
            </field>
        </record>

        <record id="view_product_supplier_inherit" model="ir.ui.view">
            <field name="name">product.template.supplier.form.inherit</field>
            <field name="model">product.template</field>
            <field name="inherit_id" ref="product.product_template_form_view"/>
            <field name="arch" type="xml">
                <xpath expr="//page[@name='purchase']" position="attributes">
                    <attribute name="invisible" remove="1" separator=" or "></attribute>
                    <attribute name="groups">purchase.group_purchase_user</attribute>
                </xpath>
                <group name="purchase" position="before">
                    <field name="seller_ids" context="{'default_product_tmpl_id': id, 'product_template_invisible_variant': True, 'tree_view_ref':'purchase.product_supplierinfo_tree_view2'}" nolabel="1" invisible="product_variant_count &gt; 1" readonly="product_variant_count &gt; 1"/>
                    <field name="variant_seller_ids" context="{'model': 'product.template', 'active_id': id, 'tree_view_ref':'purchase.product_supplierinfo_tree_view2'}" nolabel="1" invisible="product_variant_count &lt;= 1" readonly="product_variant_count &lt;= 1"/>
                </group>
                <group name="bill" position="attributes">
                    <attribute name="groups">purchase.group_purchase_manager</attribute>
                </group>
                <group name="bill" position="inside">
                    <field name="purchase_method" widget="radio"/>
                </group>
                <group name="purchase" position="inside">
                    <group col="1">
                        <group string="Purchase Description">
                           <field name="description_purchase" nolabel="1" colspan="2"
                                placeholder="This note is added to purchase orders."/>
                        </group>
                        <group string="Warning when Purchasing this Product" groups="purchase.group_warning_purchase">
                            <field name="purchase_line_warn" nolabel="1" colspan="2"/>
                            <field name="purchase_line_warn_msg" colspan="2" nolabel="1" placeholder="Type a message..."
                                invisible="purchase_line_warn == 'no-message'"
                                readonly="purchase_line_warn == 'no-message'"
                                required="purchase_line_warn != 'no-message'"/>
                        </group>
                    </group>
                </group>
            </field>
        </record>

        <record id="view_product_product_supplier_inherit" model="ir.ui.view">
            <field name="name">product.product.form</field>
            <field name="model">product.product</field>
            <field name="inherit_id" ref="product.product_normal_form_view"/>
            <field name="arch" type="xml">
                <field name="seller_ids" position="attributes">
                    <attribute name="context">{'default_product_tmpl_id': product_tmpl_id, 'product_template_invisible_variant': True, 'tree_view_ref':'purchase.product_supplierinfo_tree_view2'}</attribute>
                </field>
                <field name="variant_seller_ids" position="attributes">
                    <attribute name="context">{'model': 'product.product', 'active_id': id, 'tree_view_ref':'purchase.product_supplierinfo_tree_view2'}</attribute>
                </field>
            </field>
        </record>

        <record id="view_product_account_purchase_ok_form" model="ir.ui.view">
            <field name="name">product.template.account.purchase.ok.form.inherit</field>
            <field name="model">product.template</field>
            <field name="inherit_id" ref="account.product_template_form_view"/>
            <field name="arch" type="xml">
                <field name='supplier_taxes_id' position="replace" >
                     <field name="supplier_taxes_id" colspan="2" widget="many2many_tags" readonly="purchase_ok == 0" context="{'default_type_tax_use':'purchase'}"/>
                </field>
            </field>
        </record>

        <record id="view_product_template_purchase_buttons_from" model="ir.ui.view">
            <field name="name">product.template.purchase.button.inherit</field>
            <field name="model">product.template</field>
            <field name="inherit_id" ref="product.product_template_only_form_view"/>
            <field name="arch" type="xml">
                <div name="button_box" position="inside">
                    <button class="oe_stat_button" name="action_view_po"
                        groups="purchase.group_purchase_user"
                        type="object" icon="fa-credit-card" invisible="not purchase_ok" help="Purchased in the last 365 days">
                        <div class="o_field_widget o_stat_info">
                            <span class="o_stat_value d-flex gap-1">
                                <field name="purchased_product_qty" widget="statinfo" nolabel="1" class="oe_inline"/>
                                <field name="uom_name"  class="oe_inline"/>
                            </span>
                            <span class="o_stat_text">Purchased</span>
                        </div>
                    </button>
                </div>
            </field>
        </record>

        <record id="product_normal_form_view_inherit_purchase" model="ir.ui.view">
            <field name="name">product.product.purchase.order</field>
            <field name="model">product.product</field>
            <field name="inherit_id" ref="product.product_normal_form_view"/>
            <field name="arch" type="xml">
                <div name="button_box" position="inside">
                    <button class="oe_stat_button" name="action_view_po"
                        groups="purchase.group_purchase_user"
                        type="object" icon="fa-credit-card" invisible="not purchase_ok" help="Purchased in the last 365 days">
                        <div class="o_field_widget o_stat_info">
                            <span class="o_stat_value d-flex gap-1">
                                <field name="purchased_product_qty" widget="statinfo" nolabel="1" class="oe_inline"/>
                                <field name="uom_name"  class="oe_inline"/>
                            </span>
                            <span class="o_stat_text">Purchased</span>
                        </div>
                    </button>
                </div>
            </field>
        </record>

        <record id="product_template_search_view_purchase" model="ir.ui.view">
            <field name="name">product.template.search.purchase</field>
            <field name="model">product.template</field>
            <field name="inherit_id" ref="product.product_template_search_view"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='categ_id']" position="after">
                    <field string="Vendor" name="seller_ids"/>
                </xpath>
            </field>
        </record>

        <!-- Product catalog -->
        <record id="product_view_kanban_catalog_purchase_only" model="ir.ui.view">
            <field name="name">product.view.kanban.catalog.purchase</field>
            <field name="model">product.product</field>
            <field name="inherit_id" ref="product.product_view_kanban_catalog"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <field name="default_code" position="after">
                    <field name="uom_po_id" invisible="1"/>
                </field>
            </field>
        </record>

        <record id="product_view_search_catalog" model="ir.ui.view">
            <field name="name">product.view.search.catalog.inherit.purchase</field>
            <field name="model">product.product</field>
            <field name="inherit_id" ref="product.product_view_search_catalog"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='product_tmpl_id']" position="after">
                    <field name="seller_ids" string="Vendor"/>
                </xpath>
            </field>
        </record>

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
            sequence="135"/>

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
            parent="menu_purchase_config" sequence="30"/>

        <record model="ir.ui.menu" id="menu_product_in_config_purchase">
            <field name="groups_id" eval="False"/>
        </record>

        <menuitem id="menu_product_attribute_action"
            action="product.attribute_action" name="Product Attributes"
            parent="purchase.menu_product_in_config_purchase"  groups="product.group_product_variant" sequence="1"/>

        <menuitem
            action="product.product_category_action_form" id="menu_product_category_config_purchase"
            parent="purchase.menu_product_in_config_purchase" sequence="3" />

         <menuitem
            id="menu_unit_of_measure_in_config_purchase" name="Units of Measures"
            parent="menu_purchase_config" sequence="40" groups="uom.group_uom" />

        <menuitem
              action="uom.product_uom_form_action" id="menu_purchase_uom_form_action" active="False"
              parent="purchase.menu_unit_of_measure_in_config_purchase" sequence="5" groups="base.group_no_one"/>

        <menuitem
             action="uom.product_uom_categ_form_action" id="menu_purchase_uom_categ_form_action"
             parent="purchase.menu_unit_of_measure_in_config_purchase" sequence="10" groups="uom.group_uom"/>

    <!-- Products Control Menu -->
    <menuitem id="menu_purchase_products" name="Products" parent="purchase.menu_purchase_root" sequence="5"/>

    <record id="product_normal_action_puchased" model="ir.actions.act_window">
        <field name="name">Products</field>
        <field name="res_model">product.template</field>
        <field name="view_mode">kanban,tree,form,activity</field>
        <field name="context">{"search_default_filter_to_purchase":1, "purchase_product_template": 1}</field>
        <field name="search_view_id" ref="product.product_template_search_view"/>
        <field name="view_id" eval="False"/> <!-- Force empty -->
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            No product found. Let's create one!
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
                <calendar string="Calendar View" date_start="date_calendar_start" color="partner_id" hide_time="true" event_limit="5" create="false">
                    <field name="currency_id" invisible="1"/>
                    <field name="partner_ref"/>
                    <field name="amount_total" widget="monetary"/>
                    <field name="partner_id" filters="1"/>
                </calendar>
            </field>
        </record>
        <record model="ir.ui.view" id="purchase_order_pivot">
            <field name="name">purchase.order.pivot</field>
            <field name="model">purchase.order</field>
            <field name="arch" type="xml">
                <pivot string="Purchase Order" display_quantity="1" sample="1">
                    <field name="partner_id" type="row"/>
                    <field name="amount_total" type="measure"/>
                </pivot>
            </field>
        </record>
        <record model="ir.ui.view" id="purchase_order_graph">
            <field name="name">purchase.order.graph</field>
            <field name="model">purchase.order</field>
            <field name="arch" type="xml">
                <graph string="Purchase Order" sample="1">
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
                    <button name="action_rfq_send" invisible="state != 'draft'" string="Send by Email" type="object" context="{'send_rfq':True}" class="oe_highlight" data-hotkey="g"/>
                    <button name="print_quotation" string="Print RFQ" type="object" invisible="state != 'draft'" class="oe_highlight" groups="base.group_user" data-hotkey="k"/>
                    <button name="button_confirm" type="object" invisible="state != 'sent'" string="Confirm Order" context="{'validate_analytic': True}" class="oe_highlight" id="bid_confirm" data-hotkey="q"/>
                    <button name="button_approve" type="object" invisible="state != 'to approve'" string="Approve Order" class="oe_highlight" groups="purchase.group_purchase_manager" data-hotkey="z"/>
                    <button name="action_create_invoice" string="Create Bill" type="object" class="oe_highlight" context="{'create_bill':True}" invisible="state not in ('purchase', 'done') or invoice_status in ('no', 'invoiced')" data-hotkey="w"/>
                    <button name="action_rfq_send" invisible="state != 'sent'" string="Re-Send by Email" type="object" context="{'send_rfq':True}" data-hotkey="g"/>
                    <button name="print_quotation" string="Print RFQ" type="object" invisible="state != 'sent'" groups="base.group_user" data-hotkey="k"/>
                    <button name="button_confirm" type="object" invisible="state != 'draft'" context="{'validate_analytic': True}" string="Confirm Order" id="draft_confirm" data-hotkey="q"/>
                    <button name="action_rfq_send" invisible="state != 'purchase'" string="Send PO by Email" type="object" context="{'send_rfq':False}" data-hotkey="g"/>
                    <button name="confirm_reminder_mail" string="Confirm Receipt Date" type="object" invisible="state not in ('purchase', 'done') or mail_reminder_confirmed or not date_planned" groups="base.group_no_one" data-hotkey="o"/>
                    <button name="action_create_invoice" string="Create Bill" type="object" context="{'create_bill':True}" invisible="state not in ('purchase', 'done') or invoice_status not in ('no', 'invoiced') or not order_line" data-hotkey="w"/>
                    <button name="button_draft" invisible="state != 'cancel'" string="Set to Draft" type="object" data-hotkey="o"/>
                    <button name="button_cancel" invisible="state not in ('draft', 'to approve', 'sent', 'purchase')" string="Cancel" type="object" data-hotkey="x" />
                    <button name="button_done" type="object" string="Lock" invisible="state != 'purchase'" data-hotkey="l"/>
                    <button name="button_unlock" type="object" string="Unlock" invisible="state != 'done'" groups="purchase.group_purchase_manager" data-hotkey="l"/>
                    <field name="state" widget="statusbar" statusbar_visible="draft,sent,purchase" readonly="1"/>
                </header>
                <sheet>
                    <div class="oe_button_box" name="button_box">
                        <button type="object"  name="action_view_invoice"
                            class="oe_stat_button"
                            icon="fa-pencil-square-o" invisible="invoice_count == 0 or state in ('draft', 'sent', 'to approve')">
                            <field name="invoice_count" widget="statinfo" string="Vendor Bills"/>
                            <field name='invoice_ids' invisible="1"/>
                        </button>
                    </div>
                    <div class="oe_title">
                        <span class="o_form_label" invisible="state not in ('draft', 'sent')">Request for Quotation </span>
                        <span class="o_form_label" invisible="state in ('draft', 'sent')">Purchase Order </span>
                        <h1 class="d-flex">
                            <field name="priority" widget="priority" class="me-3"/>
                            <field name="name" readonly="1"/>
                        </h1>
                    </div>
                    <group>
                        <group>
                            <field name="partner_id" widget="res_partner_many2one" context="{'res_partner_search_mode': 'supplier', 'show_vat': True}"
                                placeholder="Name, TIN, Email, or Reference"
                             readonly="state in ['cancel', 'done', 'purchase']"/>
                            <field name="partner_ref"/>
                            <field name="currency_id" groups="base.group_multi_currency" force_save="1" readonly="state in ['cancel', 'done', 'purchase']"/>
                            <field name="id" invisible="1"/>
                            <field name="company_id" invisible="1" readonly="state in ['cancel', 'done', 'purchase']"/>
                            <field name="currency_id" invisible="1" readonly="state in ['cancel', 'done', 'purchase']" groups="!base.group_multi_currency"/>
                            <field name="tax_calculation_rounding_method" invisible="1"/>
                        </group>
                        <group>
                            <field name="date_order" invisible="state in ('purchase', 'done')" readonly="state in ['cancel', 'done', 'purchase']"/>
                            <label for="date_approve" invisible="state not in ('purchase', 'done')"/>
                            <div name="date_approve" invisible="state not in ('purchase', 'done')" class="o_row">
                                <field name="date_approve"/>
                                <field name="mail_reception_confirmed" invisible="1"/>
                                <span class="text-muted" invisible="not mail_reception_confirmed">(confirmed by vendor)</span>
                            </div>
                            <label for="date_planned"/>
                            <div name="date_planned_div" class="o_row">
                                <field name="date_planned" readonly="state not in ('draft', 'sent', 'to approve', 'purchase')"/>
                                <field name="mail_reminder_confirmed" invisible="1"/>
                                <span class="text-muted" invisible="not mail_reminder_confirmed">(confirmed by vendor)</span>
                            </div>
                            <label for="receipt_reminder_email" class="d-none" groups="purchase.group_send_reminder"/>
                            <div name="reminder" class="o_row" groups='purchase.group_send_reminder' title="Automatically send a confirmation email to the vendor X days before the expected receipt date, asking him to confirm the exact date.">
                                <field name="receipt_reminder_email"/>
                                <span>Ask confirmation</span>
                                <div class="o_row oe_inline" invisible="not receipt_reminder_email">
                                    <field name="reminder_date_before_receipt"/>
                                    day(s) before
                                    <widget name='toaster_button' button_name="send_reminder_preview" title="Preview the reminder email by sending it to yourself." invisible="not id"/>
                                </div>
                            </div>
                        </group>
                    </group>
                    <notebook>
                        <page string="Products" name="products">
                            <field name="tax_country_id" invisible="1"/>
                            <field name="order_line"
                                widget="section_and_note_one2many"
                                mode="tree,kanban"
                                context="{'default_state': 'draft'}"
                                readonly="state in ('done', 'cancel')">
                                <tree string="Purchase Order Lines" editable="bottom">
                                    <control>
                                        <create name="add_product_control" string="Add a product"/>
                                        <create name="add_section_control" string="Add a section" context="{'default_display_type': 'line_section'}"/>
                                        <create name="add_note_control" string="Add a note" context="{'default_display_type': 'line_note'}"/>
                                        <button name="action_add_from_catalog" string="Catalog" type="object" class="px-4 btn-link" context="{'order_id': parent.id}"/>
                                    </control>
                                    <field name="tax_calculation_rounding_method" column_invisible="True"/>
                                    <field name="display_type" column_invisible="True"/>
                                    <field name="company_id" column_invisible="True"/>
                                    <field name="currency_id" column_invisible="True"/>
                                    <field name="state" column_invisible="True"/>
                                    <field name="product_type" column_invisible="True"/>
                                    <field name="product_uom" column_invisible="True" groups="!uom.group_uom"/>
                                    <field name="product_uom_category_id" column_invisible="True"/>
                                    <field name="invoice_lines" column_invisible="True"/>
                                    <field name="sequence" widget="handle"/>
                                    <field
                                        name="product_id"
                                        readonly="state in ('purchase', 'to approve', 'done', 'cancel')"
                                        required="not display_type"
                                        width="35%"
                                        context="{'partner_id': parent.partner_id, 'quantity': product_qty, 'company_id': parent.company_id, 'use_partner_name': False}"
                                        force_save="1" domain="[('purchase_ok', '=', True), '|', ('company_id', '=', False), ('company_id', 'parent_of', parent.company_id)]"/>
                                    <field name="name" widget="section_and_note_text"/>
                                    <field name="date_planned" optional="hide" required="not display_type" force_save="1"/>
                                    <field name="analytic_distribution" widget="analytic_distribution"
                                           optional="hide"
                                           groups="analytic.group_analytic_accounting"
                                           options="{'product_field': 'product_id', 'business_domain': 'purchase_order', 'amount_field': 'price_subtotal'}"/>
                                    <field name="product_qty"/>
                                    <field name="qty_received_manual" column_invisible="True"/>
                                    <field name="qty_received_method" column_invisible="True"/>
                                    <field name="qty_received" string="Received" column_invisible="parent.state not in ('purchase', 'done')" readonly="qty_received_method != 'manual'" optional="show"/>
                                    <field name="qty_invoiced" string="Billed" column_invisible="parent.state not in ('purchase', 'done')" optional="show"/>
                                    <field name="product_uom" string="UoM" groups="uom.group_uom"
                                        readonly="state in ('purchase', 'done', 'cancel')"
                                        required="not display_type"
                                        options="{'no_open': True}"
                                        force_save="1" optional="show"/>
                                    <field name="product_packaging_qty" invisible="not product_id or not product_packaging_id" groups="product.group_stock_packaging" optional="show"/>
                                    <field name="product_packaging_id" invisible="not product_id" context="{'default_product_id': product_id, 'tree_view_ref':'product.product_packaging_tree_view', 'form_view_ref':'product.product_packaging_form_view'}" groups="product.group_stock_packaging" optional="show"/>
                                    <field name="price_unit" readonly="qty_invoiced != 0"/>
                                    <button name="action_purchase_history" type="object" icon="fa-history" title="Purchase History" invisible="not id"/>
                                    <field name="taxes_id" widget="many2many_tags" domain="[('type_tax_use', '=', 'purchase'), ('company_id', 'parent_of', parent.company_id), ('country_id', '=', parent.tax_country_id), ('active', '=', True)]" context="{'default_type_tax_use': 'purchase', 'search_view_ref': 'account.account_tax_view_search'}" options="{'no_create': True}" optional="show"/>
                                    <field name="discount" string="Disc.%" readonly="qty_invoiced != 0" optional="hide"/>
                                    <field name="price_subtotal" string="Tax excl."/>
                                    <field name="price_total"
                                           string="Tax incl."
                                           column_invisible="parent.tax_calculation_rounding_method == 'round_globally'"
                                           optional="hide"/>
                                </tree>
                                <form string="Purchase Order Line">
                                        <field name="tax_calculation_rounding_method" invisible="1"/>
                                        <field name="state" invisible="1"/>
                                        <field name="display_type" invisible="1"/>
                                        <field name="company_id" invisible="1"/>
                                        <group invisible="display_type">
                                            <group>
                                                <field name="product_uom_category_id" invisible="1"/>
                                                <field name="product_id"
                                                       context="{'partner_id': parent.partner_id}"
                                                       widget="many2one_barcode"
                                                       domain="[('purchase_ok', '=', True), '|', ('company_id', '=', False), ('company_id', '=', parent.company_id)]"
                                                       readonly="state in ('purchase', 'to approve', 'done', 'cancel')"
                                                />
                                                <label for="product_qty"/>
                                                <div class="o_row">
                                                    <field name="product_qty"/>
                                                    <field name="product_uom" groups="uom.group_uom" required="not display_type"/>
                                                </div>
                                                <field name="qty_received_method" invisible="1"/>
                                                <field name="qty_received" string="Received Quantity" invisible="parent.state not in ('purchase', 'done')" readonly="qty_received_method != 'manual'"/>
                                                <field name="qty_invoiced" string="Billed Quantity" invisible="parent.state not in ('purchase', 'done')"/>
                                                <field name="product_packaging_qty" invisible="not product_id or not product_packaging_id" groups="product.group_stock_packaging"/>
                                                <field name="product_packaging_id" invisible="not product_id" context="{'default_product_id': product_id, 'tree_view_ref':'product.product_packaging_tree_view', 'form_view_ref':'product.product_packaging_form_view'}" groups="product.group_stock_packaging" />
                                                <field name="price_unit"/>
                                                <field name="discount"/>
                                                <field name="taxes_id" widget="many2many_tags" domain="[('type_tax_use', '=', 'purchase'), ('company_id', 'parent_of', parent.company_id), ('country_id', '=', parent.tax_country_id)]" options="{'no_create': True}"/>
                                            </group>
                                            <group>
                                                <field name="date_planned" widget="date" required="not display_type"/>
                                                <field name="analytic_distribution" widget="analytic_distribution"
                                                       groups="analytic.group_analytic_accounting"
                                                       options="{'product_field': 'product_id', 'business_domain': 'purchase_order'}"/>
                                            </group>
                                            <group>
                                            <notebook colspan="4">
                                                <page string="Notes" name="notes">
                                                    <field name="name"/>
                                                </page>
                                                <page string="Invoices and Incoming Shipments" name="invoices_incoming_shiptments">
                                                    <field name="invoice_lines"/>
                                                </page>
                                            </notebook>
                                            </group>
                                        </group>
                                        <label for="name" string="Section Name (eg. Products, Services)" invisible="display_type != 'line_section'"/>
                                        <label for="name" string="Note" invisible="display_type != 'line_note'"/>
                                        <field name="name" nolabel="1"  invisible="not display_type"/>
                                </form>
                                <kanban class="o_kanban_mobile">
                                     <field name="name"/>
                                     <field name="product_id"/>
                                     <field name="product_qty"/>
                                     <field name="product_uom" groups="uom.group_uom"/>
                                     <field name="price_subtotal"/>
                                     <field name="price_tax"/>
                                     <field name="price_total"/>
                                     <field name="price_unit"/>
                                     <field name="discount"/>
                                     <field name="display_type"/>
                                     <field name="taxes_id"/>
                                     <field name="tax_calculation_rounding_method"/>
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
                                                                 <span>
                                                                     Tax excl.: <t t-esc="record.price_subtotal.value" class="float-end text-end"/>
                                                                 </span>
                                                             </strong>
                                                         </div>
                                                     </div>
                                                     <div class="row">
                                                         <div class="col-8 text-muted">
                                                             <span>
                                                                 Quantity:
                                                                 <t t-esc="record.product_qty.value"/>
                                                                 <small> <t t-esc="record.product_uom.value" groups="uom.group_uom"/></small>
                                                             </span>
                                                         </div>
                                                         <div class="col-4" t-if="record.tax_calculation_rounding_method.raw_value === 'round_per_line'">
                                                             <strong>
                                                                <span>
                                                                    Tax incl.: <t t-esc="record.price_total.value"/>
                                                                </span>
                                                             </strong>
                                                         </div>
                                                     </div>
                                                     <div class="row">
                                                         <div class="col-12 text-muted">
                                                             <span>
                                                                 Unit Price:
                                                                 <field name="price_unit"/>
                                                             </span>
                                                         </div>
                                                     </div>
                                                     <div class="row" t-if="record.discount.raw_value">
                                                         <div class="col-12 text-muted">
                                                            <span>
                                                                Discount: <t t-out="record.discount.value"/>%
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
                            <group>
                                <group>
                                    <field colspan="2" name="notes" nolabel="1" placeholder="Define your terms and conditions ..."/>
                                </group>
                                <group class="oe_subtotal_footer">
                                    <field name="tax_totals" widget="account-tax-totals-field" nolabel="1" colspan="2" readonly="1"/>
                                </group>
                            </group>
                            <div class="clearfix"/>
                        </page>
                        <page string="Other Information" name="purchase_delivery_invoice">
                            <group>
                                <group name="other_info">
                                    <field name="user_id" domain="[('share', '=', False)]" widget="many2one_avatar_user"/>
                                    <field name="company_id" groups="base.group_multi_company" options="{'no_create': True}" readonly="state in ['cancel', 'done', 'purchase']"/>
                                    <field name="origin"/>
                                </group>
                                <group name="invoice_info">
                                    <field name="invoice_status" invisible="state in ('draft', 'sent', 'to approve', 'cancel')"/>
                                    <field name="payment_term_id" readonly="invoice_status == 'invoiced' or state == 'done'" options="{'no_create': True}"/>
                                    <field name="fiscal_position_id" options="{'no_create': True}" readonly="invoice_status == 'invoiced' or state == 'done'"/>
                                </group>
                            </group>
                        </page>
                    </notebook>
                </sheet>
                <div class="oe_chatter">
                    <field name="message_follower_ids"/>
                    <field name="activity_ids"/>
                    <field name="message_ids"/>
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
                    <field name="origin"/>
                    <filter name="my_purchases" string="My Purchases" domain="[('user_id', '=', uid)]"/>
                    <filter string="Starred" name="starred" domain="[('priority', '=', '1')]"/>
                    <separator/>
                    <filter name="draft" string="RFQs" domain="[('state', 'in', ('draft', 'sent', 'to approve'))]"/>
                    <separator/>
                    <filter name="approved" string="Purchase Orders" domain="[('state', 'in', ('purchase', 'done'))]"/>
                    <filter name="to_approve" string="To Approve" domain="[('state', '=', 'to approve')]"/>
                    <separator/>
                    <filter name="order_date" string="Order Date" date="date_order"/>
                    <filter name="draft_rfqs" string="Draft RFQs" domain="[('state', '=', 'draft')]"/>
                    <filter name="waiting_rfqs" string="Waiting RFQs" domain="[('state', '=', 'sent'), ('date_order', '&gt;=', datetime.datetime.now())]"/>
                    <filter name="late_rfqs" string="Late RFQs" domain="[('state', 'in', ['draft', 'sent', 'to approve']),('date_order', '&lt;', datetime.datetime.now())]"/>
                    <separator/>
                    <filter invisible="1" string="Late Activities" name="activities_overdue"
                        domain="[('my_activity_date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                        help="Show all records which has next action date is before today"/>
                    <filter invisible="1" string="Today Activities" name="activities_today"
                        domain="[('my_activity_date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                    <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                        domain="[('my_activity_date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
                    <separator/>
                    <filter string="Warnings" name="activities_exception"
                        domain="[('activity_exception_decoration', '!=', False)]"/>
                    <group expand="0" string="Group By">
                        <filter string="Vendor" name="vendor" domain="[]" context="{'group_by': 'partner_id'}"/>
                        <filter string="Buyer" name="representative" domain="[]" context="{'group_by': 'user_id'}"/>
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
                    <filter string="Starred" name="starred" domain="[('priority', '=', '1')]"/>
                    <separator/>
                    <filter name="unconfirmed" string="Not Acknowledged" domain="[('mail_reception_confirmed', '=', False), ('state', '=', 'purchase')]"/>
                    <filter name="not_invoiced" string="Waiting Bills" domain="[('invoice_status', '=', 'to invoice')]" help="Purchase orders that include lines not invoiced."/>
                    <filter name="invoiced" string="Bills Received" domain="[('invoice_status', '=', 'invoiced')]" help="Purchase orders that have been invoiced."/>
                    <separator/>
                    <filter name="order_date" string="Order Date" date="date_order"/>
                    <separator/>
                    <filter invisible="1" string="Late Activities" name="activities_overdue"
                        domain="[('my_activity_date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                        help="Show all records which has next action date is before today"/>
                    <filter invisible="1" string="Today Activities" name="activities_today"
                        domain="[('my_activity_date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                    <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                        domain="[('my_activity_date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
                    <separator/>
                    <filter string="Warnings" name="activities_exception"
                        domain="[('activity_exception_decoration', '!=', False)]"/>
                    <group expand="0" string="Group By">
                        <filter string="Vendor" name="vendor" domain="[]" context="{'group_by': 'partner_id'}"/>
                        <filter string="Buyer" name="representative" domain="[]" context="{'group_by': 'user_id'}"/>
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
                <kanban class="o_kanban_mobile" js_class="purchase_dashboard_kanban" sample="1" quick_create="false">
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
                                    <field name="priority" widget="priority"/>
                                    <div class="o_kanban_record_headings ms-1">
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

        <record id="purchase_order_view_kanban_without_dashboard" model="ir.ui.view">
            <field name="name">purchase.order.view.kanban.without.dashboard</field>
            <field name="model">purchase.order</field>
            <field name="inherit_id" ref="purchase.view_purchase_order_kanban"/>
            <field name="mode">primary</field>
            <field name="priority">20</field>
            <field name="arch" type="xml">
                <xpath expr="//kanban" position="attributes">
                    <attribute name="js_class"/>
                </xpath>
            </field>
        </record>

        <record id="purchase_order_tree" model="ir.ui.view">
            <field name="name">purchase.order.tree</field>
            <field name="model">purchase.order</field>
            <field name="priority" eval="1"/>
            <field name="arch" type="xml">
                <tree string="Purchase Order"
                      decoration-muted="state=='cancel'" sample="1">
                    <field name="priority" optional="show" widget="priority" nolabel="1"/>
                    <field name="partner_ref" optional="hide"/>
                    <field name="name" string="Reference" readonly="1" decoration-info="state in ('draft','sent')"/>
                    <field name="date_order" column_invisible="not context.get('quotation_only', False)" readonly="state in ['cancel', 'done', 'purchase']" optional="show"/>
                    <field name="date_approve" column_invisible="context.get('quotation_only', False)" optional="show"/>
                    <field name="partner_id" readonly="1"/>
                    <field name="company_id" readonly="1" options="{'no_create': True}"
                        groups="base.group_multi_company" optional="show"/>
                    <field name="user_id" optional="show"/>
                    <field name="origin" optional="show"/>
                    <field name="amount_untaxed" sum="Total Untaxed amount" string="Untaxed" widget="monetary" optional="hide"/>
                    <field name="amount_total" sum="Total amount" widget="monetary" optional="show"/>
                    <field name="currency_id" column_invisible="True" readonly="state in ['cancel', 'done', 'purchase']"/>
                    <field name="state" optional="show"/>
                    <field name="date_planned" column_invisible="context.get('quotation_only', False)" optional="show"/>
                    <field name="invoice_status" column_invisible="context.get('quotation_only', False)" optional="hide"/>
                    <field name="activity_exception_decoration" widget="activity_exception"/>
                </tree>
            </field>
        </record>

        <!-- Unfortunately we want the purchase kpis table to only show up in some list views,
             so we have to duplicate code to support both view versions -->
        <record id="purchase_order_kpis_tree" model="ir.ui.view">
            <field name="name">purchase.order.inherit.purchase.order.tree</field>
            <field name="model">purchase.order</field>
            <field name="priority" eval="10"/>
            <field name="arch" type="xml">
                <tree string="Purchase Order" decoration-info="state in ['draft', 'sent']"
                decoration-muted="state == 'cancel'"
                class="o_purchase_order" js_class="purchase_dashboard_list" sample="1">
                    <header>
                        <button name="action_create_invoice" type="object" string="Create Bills"/>
                    </header>
                    <field name="priority" optional="show" widget="priority" nolabel="1"/>
                    <field name="partner_ref" optional="hide"/>
                    <field name="name" string="Reference" readonly="1" decoration-bf="1"/>
                    <field name="date_approve" column_invisible="context.get('quotation_only', False)" optional="show"/>
                    <field name="partner_id" readonly="1"/>
                    <field name="company_id" readonly="1" options="{'no_create': True}"
                        groups="base.group_multi_company" optional="show"/>
                    <field name="company_id" groups="!base.group_multi_company" column_invisible="True" readonly="state in ['cancel', 'done', 'purchase']"/>
                    <field name="date_planned" column_invisible="context.get('quotation_only', False)" optional="show"/>
                    <field name="user_id" optional="show" widget="many2one_avatar_user"/>
                    <field name="date_order"
                        invisible="state == 'purchase' or state == 'done' or state == 'cancel'"
                        column_invisible="not context.get('quotation_only', False)"
                        readonly="state in ['cancel', 'done', 'purchase']" widget="remaining_days" optional="show"/>
                    <field name="activity_ids" widget="list_activity" optional="show"/>
                    <field name="origin" optional="show"/>
                    <field name="amount_untaxed" sum="Total Untaxed amount" string="Untaxed" widget="monetary" optional="hide"/>
                    <field name="amount_total" sum="Total amount" widget="monetary" optional="show" decoration-bf="state in ['purchase', 'done']"/>
                    <field name="currency_id" column_invisible="True" readonly="state in ['cancel', 'done', 'purchase']"/>
                    <field name="state" optional="show" widget="badge" decoration-success="state == 'purchase' or state == 'done'"
                        decoration-warning="state == 'to approve'" decoration-info="state == 'draft' or state == 'sent'"/>
                    <field name="invoice_status" optional="hide"/>
                </tree>
            </field>
        </record>

        <record id="purchase_order_view_tree" model="ir.ui.view">
            <field name="name">purchase.order.view.tree</field>
            <field name="model">purchase.order</field>
            <field name="arch" type="xml">
                <tree decoration-muted="state == 'cancel'"
                    decoration-info="invoice_status == 'to invoice'"
                    string="Purchase Order"
                    class="o_purchase_order"
                    sample="1">
                    <header>
                        <button name="action_create_invoice" type="object" string="Create Bills"/>
                    </header>
                    <field name="priority" optional="show" widget="priority" nolabel="1"/>
                    <field name="partner_ref" optional="hide"/>
                    <field name="name" string="Reference" readonly="1" decoration-bf="1" decoration-info="state in ('draft','sent')"/>
                    <field name="date_approve" widget="date" column_invisible="context.get('quotation_only', False)" optional="show"/>
                    <field name="partner_id" readonly="state in ['cancel', 'done', 'purchase']"/>
                    <field name="company_id" groups="base.group_multi_company" options="{'no_create': True}" optional="show" readonly="state in ['cancel', 'done', 'purchase']"/>
                    <field name="company_id" groups="!base.group_multi_company" column_invisible="True" readonly="state in ['cancel', 'done', 'purchase']"/>
                    <field name="user_id" widget="many2one_avatar_user" optional="show"/>
                    <field name="date_order" column_invisible="not context.get('quotation_only', False)" readonly="state in ['cancel', 'done', 'purchase']" optional="show"/>
                    <field name="activity_ids" widget="list_activity" optional="show"/>
                    <field name="origin" optional="show"/>
                    <field name="amount_untaxed" sum="Total Untaxed amount" string="Untaxed" widget="monetary" optional="hide"/>
                    <field name="amount_total" sum="Total amount" widget="monetary" optional="show" decoration-bf="1"/>
                    <field name="currency_id" column_invisible="True" readonly="state in ['cancel', 'done', 'purchase']"/>
                    <field name="state" column_invisible="True"/>
                    <field name="invoice_status" widget="badge" decoration-success="invoice_status == 'invoiced'" decoration-info="invoice_status == 'to invoice'" optional="show"/>
                    <field name="date_planned" column_invisible="context.get('quotation_only', False)" optional="show"/>
                </tree>
            </field>
        </record>

        <record id="purchase_order_view_activity" model="ir.ui.view">
            <field name="name">purchase.order.activity</field>
            <field name="model">purchase.order</field>
            <field name="arch" type="xml">
                <activity string="Purchase Order">
                    <field name="currency_id"/>
                    <templates>
                        <div t-name="activity-box" class="d-block">
                            <div class="d-flex justify-content-between">
                                <field name="name" display="full" class="o_text_block o_text_bold"/>
                                <div class="m-1"/>
                                <field name="amount_total" widget="monetary"/>
                            </div>
                            <div class="d-flex justify-content-between">
                                <field name="partner_id" muted="1" display="full" class="o_text_block"/>
                                <div class="m-1"/>
                                <field name="state" widget="badge"
                                   decoration-success="state == 'purchase' or state == 'done'"
                                   decoration-warning="state == 'to approve'"
                                   decoration-info="state == 'draft' or state == 'sent'"/>
                            </div>
                        </div>
                    </templates>
                </activity>
            </field>
        </record>

        <record id="purchase_rfq" model="ir.actions.act_window">
            <field name="name">Requests for Quotation</field>
            <field name="res_model">purchase.order</field>
            <field name="view_mode">tree,kanban,form,pivot,graph,calendar,activity</field>
            <field name="view_id" ref="purchase_order_kpis_tree"/>
            <field name="domain">[]</field>
            <field name="search_view_id" ref="view_purchase_order_filter"/>
            <field name="context">{'quotation_only': True}</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                No request for quotation found. Let's create one!
              </p><p>
                Requests for quotation are documents that will be sent to your suppliers to request prices for different products you consider buying.
                Once an agreement has been found with the supplier, they will be confirmed and turned into purchase orders.
              </p>
            </field>
        </record>
        <menuitem action="purchase_rfq" id="menu_purchase_rfq"
            parent="menu_procurement_management"
            sequence="0"/>

        <record id="purchase_form_action" model="ir.actions.act_window">
            <field name="name">Purchase Orders</field>
            <field name="res_model">purchase.order</field>
            <field name="view_mode">tree,kanban,form,pivot,graph,calendar,activity</field>
            <field name="view_ids" eval="[(5, 0, 0),
                (0, 0, {'view_mode': 'tree', 'view_id': ref('purchase.purchase_order_view_tree')}),
                (0, 0, {'view_mode': 'kanban', 'view_id': ref('purchase.purchase_order_view_kanban_without_dashboard')}),
            ]"/>
            <field name="domain">[('state','in',('purchase', 'done'))]</field>
            <field name="search_view_id" ref="purchase_order_view_search"/>
            <field name="context">{}</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                No purchase order found. Let's create one!
              </p><p>
                Once you ordered your products to your supplier, confirm your request for quotation and it will turn into a purchase order.
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
                    <field name="currency_id" column_invisible="True"/>
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
                        <label for="order_id"/>
                        <h1>
                            <field name="order_id" class="oe_inline"/>
                            <label string="," for="date_order" invisible="not date_order"/>
                            <field name="date_order" class="oe_inline"/>
                        </h1>
                        <label for="partner_id"/>
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
                                <field name="analytic_distribution" widget="analytic_distribution"
                                       groups="analytic.group_analytic_accounting"
                                       options="{'product_field': 'product_id', 'business_domain': 'purchase_order'}"/>
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
                    <field name="partner_id" string="Vendor"/>
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

    <record id="purchase_history_tree" model="ir.ui.view">
        <field name="name">purchase.history.tree</field>
        <field name="model">purchase.order.line</field>
        <field name="arch" type="xml">
            <tree create="false" default_order="order_id asc">
                <field name="order_id" widget="many2one"/>
                <field name="date_approve" widget="date"/>
                <field name="partner_id"/>
                <field name="product_uom_qty"/>
                <field name="price_unit"/>
            </tree>
        </field>
    </record>

    <record id="action_purchase_history" model="ir.actions.act_window">
        <field name="res_model">purchase.order.line</field>
        <field name="view_mode">tree</field>
        <field name="view_id" ref="purchase.purchase_history_tree"/>
        <field name="context">{}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                No purchase order were made for this product yet!
            </p>
        </field>
    </record>

    <record id="action_purchase_batch_bills" model="ir.actions.server">
        <field name="name">Create Vendor Bills</field>
        <field name="model_id" ref="purchase.model_purchase_order"/>
        <field name="binding_model_id" ref="purchase.model_purchase_order"/>
        <field name='groups_id' eval="[(4, ref('account.group_account_invoice'))]"/>
        <field name="state">code</field>
        <field name="code">
            if records:
                action = records.action_create_invoice()
        </field>
    </record>

    <record id="action_purchase_send_reminder" model="ir.actions.server">
        <field name="name">Send Reminder</field>
        <field name="model_id" ref="purchase.model_purchase_order"/>
        <field name="binding_model_id" ref="purchase.model_purchase_order"/>
        <field name='groups_id' eval="[(4, ref('purchase.group_send_reminder'))]"/>
        <field name="binding_view_types">form</field>
        <field name="state">code</field>
        <field name="code">
            if records:
                action = records._send_reminder_mail(send_single=True)
        </field>
    </record>

    <record id="action_accrued_expense_entry" model="ir.actions.act_window">
        <field name="name">Accrued Expense Entry</field>
        <field name="res_model">account.accrued.orders.wizard</field>
        <field name="view_mode">form</field>
        <field name="binding_model_id" ref="purchase.model_purchase_order"/>
        <field name="groups_id" eval="[(4, ref('account.group_account_user'))]"/>
        <field name="target">new</field>
    </record>

    <record id="action_rfq_form" model="ir.actions.act_window">
        <field name="name">Requests for Quotation</field>
        <field name="res_model">purchase.order</field>
        <field name="view_mode">form</field>
        <field name="view_id" ref="purchase.purchase_order_form"/>
        <field name="search_view_id" ref="view_purchase_order_filter"/>
        <field name="target">main</field>
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
            <xpath expr="//form" position="inside">
                <app data-string="Purchase" string="Purchase" name="purchase" groups="purchase.group_purchase_manager">
                    <field name="po_double_validation" invisible="1"/>
                    <field name="company_currency_id" invisible="1"/>
                    <field name="po_lock" invisible="1"/>
                    <block title="Orders" name="purchase_setting_container">
                        <setting id="po_order_approval" company_dependent="1" help="Request managers to approve orders above a minimum amount">
                            <field name="po_order_approval"/>
                            <div class="content-group" invisible="not po_order_approval">
                                <div class="row mt16">
                                    <label for="po_double_validation_amount" class="col-lg-4 o_light_label"/>
                                    <field name="po_double_validation_amount"/>
                                </div>
                            </div>
                        </setting>
                        <setting id="automatic_lock_confirmed_orders" help="Automatically lock confirmed orders to prevent editing">
                            <field name="lock_confirmed_po"/>
                        </setting>
                        <setting id="get_order_warnings" string="Warnings" help="Get warnings in orders for products or vendors">
                            <field name="group_warning_purchase"/>
                        </setting>
                        <setting id="manage_purchase_agreements" title="Calls for tenders are when you want to generate requests for quotations with several vendors for a given set of products to compare offers." documentation="/applications/inventory_and_mrp/purchase/manage_deals/agreements.html" help="Manage your purchase agreements (call for tenders, blanket orders)">
                            <field name="module_purchase_requisition"/>
                            <div class="content-group" invisible="not module_purchase_requisition">
                                <div id="use_purchase_requisition"/>
                            </div>
                        </setting>
                        <setting id="auto_receipt_reminder" help="Automatically remind the receipt date to your vendors">
                            <field name="group_send_reminder"/>
                        </setting>
                    </block>
                    <block title="Invoicing" name="invoicing_settings_container">
                        <setting id="quantities_billed_vendor" title="This default value is applied to any new product created. This can be changed in the product detail form." documentation="/applications/inventory_and_mrp/purchase/manage_deals/control_bills.html" help="Quantities billed by vendors">
                            <field name="default_purchase_method" class="o_light_label" widget="radio"/>
                        </setting>
                        <setting id="three_way_matching" title="If enabled, activates 3-way matching on vendor bills : the items must be received in order to pay the invoice." documentation="/applications/inventory_and_mrp/purchase/manage_deals/control_bills.html" help="Make sure you only pay bills for which you received the goods you ordered">
                            <field name="module_account_3way_match" string="3-way matching" widget="upgrade_boolean"/>
                        </setting>
                    </block>
                    <block title="Products" name="matrix_setting_container">
                        <setting id="variant_options" help="Purchase variants of a product using attributes (size, color, etc.)" documentation="/applications/sales/sales/products_prices/products/variants.html">
                            <field name="group_product_variant"/>
                            <div class="content-group" invisible="not group_product_variant">
                                <div class="mt8">
                                    <button name="%(product.attribute_action)d" icon="oi-arrow-right" type="action" string="Attributes" class="btn-link"/>
                                </div>
                            </div>
                        </setting>
                        <setting id="product_matrix" title="If installed, the product variants will be added to purchase orders through a grid entry." string="Variant Grid Entry" help="Add several variants to the purchase order from a grid">
                            <field name="module_purchase_product_matrix"/>
                        </setting>
                        <setting id="stock_packaging_purchase" help="Purchase products by multiple of unit # per package" title="Ability to select a package type in purchase orders and to force a quantity that is a multiple of the number of units per package.">
                            <field name="group_stock_packaging"/>
                        </setting>
                        <setting id="sell_purchase_uom" help="Sell and purchase products in different units of measure"
                                 documentation="/applications/inventory_and_mrp/inventory/management/products/uom.html">
                            <field name="group_uom"/>
                            <div class="content-group">
                                <div class="mt8" invisible="not group_uom">
                                    <button name="%(uom.product_uom_categ_form_action)d" icon="oi-arrow-right"
                                            type="action" string="Units Of Measure" class="btn-link"/>
                                </div>
                            </div>
                        </setting>
                    </block>
                </app>
            </xpath>
        </field>
    </record>

    <record id="action_purchase_configuration" model="ir.actions.act_window">
        <field name="name">Settings</field>
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
            <field name="arch" type="xml">
                <group name="purchase" position="inside">
                    <div name="receipt_reminder" colspan="2" class="o_checkbox_optional_field" groups='purchase.group_send_reminder'>
                        <label for="receipt_reminder_email"/>
                        <field name="receipt_reminder_email"/>
                        <div invisible="not receipt_reminder_email">
                            <field name="reminder_date_before_receipt" class="oe_inline"/>
                            <span> day(s) before</span>
                        </div>
                    </div>
                    <field name="property_purchase_currency_id" options="{'no_create': True, 'no_open': True}" groups='base.group_multi_currency'/>
                </group>
                <field name="property_supplier_payment_term_id" position="before">
                    <field name="buyer_id" domain="[('share', '=', False)]" widget="many2one_avatar_user"/>
                </field>
            </field>
    </record>

    <record id="act_res_partner_2_purchase_order" model="ir.actions.act_window">
            <field name="name">RFQs and Purchases</field>
            <field name="res_model">purchase.order</field>
            <field name="view_mode">tree,kanban,form,graph</field>
            <field name="view_ids" eval="[(5, 0, 0),
                (0, 0, {'view_mode': 'tree', 'view_id': ref('purchase.purchase_order_tree')}),
                (0, 0, {'view_mode': 'kanban', 'view_id': ref('purchase.purchase_order_view_kanban_without_dashboard')}),
            ]"/>
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
            <field name="arch" type="xml">
                <field name="mobile" position="after">
                    <field name="purchase_order_count" groups="purchase.group_purchase_user"/>
                </field>
                <xpath expr="//div[hasclass('oe_kanban_bottom_left')]" position="inside">
                    <a t-if="record.purchase_order_count.value>0" data-type="action" data-name="%(purchase.act_res_partner_2_purchase_order)d"
                        groups="purchase.group_purchase_user"
                        href="#" class="oe_kanban_action oe_kanban_action_a me-1">
                        <span class="badge rounded-pill">
                            <i class="fa fa-fw fa-credit-card" role="img" aria-label="Purchases" title="Purchases"/>
                            <t t-out="record.purchase_order_count.value"/>
                        </span>
                    </a>
                </xpath>
            </field>
        </record>

    <record id="act_res_partner_2_supplier_invoices" model="ir.actions.act_window">
            <field name="name">Vendor Bills</field>
            <field name="res_model">account.move</field>
            <field name="view_mode">tree,form,graph</field>
            <field name="domain">[('move_type','in',('in_invoice', 'in_refund'))]</field>
            <field name="context">{'search_default_partner_id': active_id, 'default_move_type': 'in_invoice', 'default_partner_id': active_id}</field>
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
            <field name="arch" type="xml">
                <div name="button_box" position="inside">
                    <button class="oe_stat_button" name="%(purchase.act_res_partner_2_purchase_order)d" type="action"
                        groups="purchase.group_purchase_user"
                        icon="fa-credit-card">
                        <field string="Purchases" name="purchase_order_count" widget="statinfo"/>
                    </button>
                </div>
                <page name="internal_notes" position="inside">
                    <group groups="purchase.group_purchase_user">
                        <group groups="purchase.group_warning_purchase" col="2">
                            <separator string="Warning on the Purchase Order" colspan="2"/>
                            <field name="purchase_warn" nolabel="1" colspan="2" required="1"/>
                            <field name="purchase_warn_msg" colspan="2" placeholder="Type a message..." nolabel="1"
                                invisible="purchase_warn in (False, 'no-message')"
                                required="purchase_warn and purchase_warn != 'no-message'"/>
                        </group>
                    </group>
                </page>
            </field>
        </record>

        <record id="res_partner_view_purchase_account_buttons" model="ir.ui.view">
            <field name="name">res.partner.view.purchase.account.buttons</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="base.view_partner_form" />
            <field name="priority" eval="12"/>
            <field name="arch" type="xml">
                <div name="button_box" position="inside">
                </div>
            </field>
        </record>
</odoo>

```

