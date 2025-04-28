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
    'description': "",
    'website': 'https://www.odoo.com/page/purchase',
    'depends': ['account'],
    'data': [
        'security/purchase_security.xml',
        'security/ir.model.access.csv',
        'data/digest_data.xml',
        'views/assets.xml',
        'views/account_move_views.xml',
        'data/purchase_data.xml',
        'data/ir_cron_data.xml',
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
    'qweb': [
        "static/src/xml/purchase_dashboard.xml",
        "static/src/xml/purchase_toaster_button.xml",
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
from datetime import datetime

from odoo import http
from odoo.exceptions import AccessError, MissingError
from odoo.http import request, Response
from odoo.tools import image_process
from odoo.tools.translate import _
from odoo.addons.portal.controllers.portal import pager as portal_pager, CustomerPortal
from odoo.addons.web.controllers.main import Binary


class CustomerPortal(CustomerPortal):

    def _prepare_home_portal_values(self, counters):
        values = super()._prepare_home_portal_values(counters)
        if 'purchase_count' in counters:
            values['purchase_count'] = request.env['purchase.order'].search_count([
                ('state', 'in', ['purchase', 'done', 'cancel'])
            ]) if request.env['purchase.order'].check_access_rights('read', raise_exception=False) else 0
        return values

    def _purchase_order_get_page_view_values(self, order, access_token, **kwargs):
        #
        def resize_to_48(b64source):
            if not b64source:
                b64source = base64.b64encode(Binary.placeholder())
            return image_process(b64source, size=(48, 48))

        values = {
            'order': order,
            'resize_to_48': resize_to_48,
        }
        return self._get_page_view_values(order, access_token, values, 'my_purchases_history', False, **kwargs)

    @http.route(['/my/purchase', '/my/purchase/page/<int:page>'], type='http', auth="user", website=True)
    def portal_my_purchase_orders(self, page=1, date_begin=None, date_end=None, sortby=None, filterby=None, **kw):
        values = self._prepare_portal_layout_values()
        PurchaseOrder = request.env['purchase.order']

        domain = []

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
            url_args={'date_begin': date_begin, 'date_end': date_end, 'sortby': sortby, 'filterby': filterby},
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

    @http.route(['/my/purchase/<int:order_id>/update'], type='http', methods=['POST'], auth="public", website=True)
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
    <img src="/purchase/static/src/img/OTDPurchase.gif" class="illustration_border" />
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
        % if object.date_planned:
            The receipt is expected for <strong>${format_date(object.date_planned)}</strong>.
            <br/><br/>
            Could you please acknowledge the receipt of this order?
        % endif
    </p>
</div></field>
            <field name="report_template" ref="action_report_purchase_order"/>
            <field name="report_name">PO_${(object.name or '').replace('/','_')}</field>
            <field name="lang">${object.partner_id.lang}</field>
            <field name="auto_delete" eval="True"/>
        </record>

        <record id="email_template_edi_purchase_reminder" model="mail.template">
            <field name="name">Purchase Order: Vendor Reminder</field>
            <field name="model_id" ref="purchase.model_purchase_order"/>
            <field name="email_from">${(object.user_id.email_formatted or user.email_formatted) |safe}</field>
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
        Here is a reminder that the delivery of the purchase order <strong>${object.name}</strong>
        % if object.partner_ref:
            <strong>(${object.partner_ref})</strong>
        % endif 
        is expected for 
        % if object.date_planned:
            <strong>${format_date(object.date_planned)}</strong>.
        % else:
            <strong>undefined</strong>.
        % endif
        Could you please confirm it will be delivered on time?
    </p>
</div></field>
            <field name="report_template" ref="action_report_purchase_order"/>
            <field name="report_name">PO_${(object.name or '').replace('/','_')}</field>
            <field name="lang">${object.partner_id.lang}</field>
            <field name="auto_delete" eval="True"/>
        </record>

    <template id="mail_notification_confirm" inherit_id="mail.mail_notification_paynow" name="Purchase: Confirmation mail notification template">
        <xpath expr="//t[@t-set='access_name']" position="after">
            <t t-if="record._name == 'purchase.order'">
                <t t-if="record.state == 'purchase' and not record.env.context.get('is_reminder')">
                    <t t-set="access_name">Confirm</t>
                    <t t-set="access_url" t-value="record.get_confirm_url(confirm_type='reception')"/>
                </t>
            </t>
        </xpath>
        <xpath expr="//a[@t-att-href='access_url']" position="replace">
            <t t-if="record._name == 'purchase.order' and record.env.context.get('is_reminder')">
                <a t-att-href="record.get_confirm_url(confirm_type='reminder')"
                    style="margin-right: 10px; background-color: #875A7B; padding: 8px 16px 8px 16px; text-decoration: none; color: #fff; border-radius: 5px; font-size:13px;">
                    Yes
                </a>
                <a t-att-href="record.get_update_url()"
                    style="margin-left: 10px; background-color: #875A7B; padding: 8px 16px 8px 16px; text-decoration: none; color: #fff; border-radius: 5px; font-size:13px;">
                    No, Update Dates
                </a>
                <div>&amp;nbsp;</div>
                <div style="margin: 0px; padding: 0px; font-size:13px; text-align: left;">
                    If you have any questions, please do not hesitate to contact us.
                    <div>&amp;nbsp;</div>
                    Best regards,
                </div>
            </t>
            <t t-else="">
                <a t-att-href="access_url"
                    style="margin-left: 10px; background-color: #875A7B; padding: 8px 16px 8px 16px; text-decoration: none; color: #fff; border-radius: 5px; font-size:13px;">
                    <t t-esc="access_name"/>
                </a>
            </t>
        </xpath>
    </template>

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
            <field eval="[(4, ref('group_purchase_user'))]" name="groups_id"/>
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

        # Copy data from PO
        invoice_vals = self.purchase_id.with_company(self.purchase_id.company_id)._prepare_invoice()
        invoice_vals['currency_id'] = self.line_ids and self.currency_id or invoice_vals.get('currency_id')
        del invoice_vals['ref']
        self.update(invoice_vals)

        # Copy purchase lines.
        po_lines = self.purchase_id.order_line - self.line_ids.mapped('purchase_line_id')
        new_lines = self.env['account.move.line']
        sequence = max(self.line_ids.mapped('sequence')) + 1 if self.line_ids else 10
        for line in po_lines.filtered(lambda l: not l.display_type):
            line_vals = line._prepare_account_move_line(self)
            line_vals.update({'sequence': sequence})
            new_line = new_lines.new(line_vals)
            sequence += 1
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

        # Compute payment_reference.
        if len(refs) == 1:
            self.payment_reference = refs[0]

        self.purchase_id = False
        self._onchange_currency()

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
                    ('type', '=', 'purchase'),
                    ('company_id', '=', self.company_id.id),
                    ('currency_id', '=', currency_id.id),
                ]
                default_journal_id = self.env['account.journal'].search(journal_domain, limit=1)
                if default_journal_id:
                    self.journal_id = default_journal_id

            self.currency_id = currency_id
            self._onchange_currency()

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
    purchase_order_id = fields.Many2one('purchase.order', 'Purchase Order', related='purchase_line_id.order_id', readonly=True)

    def _copy_data_extend_business_fields(self, values):
        # OVERRIDE to copy the 'purchase_line_id' field as well.
        super(AccountMoveLine, self)._copy_data_extend_business_fields(values)
        values['purchase_line_id'] = self.purchase_line_id.id

```

## File: models\mail_compose_message.py

```python
# -*- coding: utf-8 -*-
# purches Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class MailComposeMessage(models.TransientModel):
    _inherit = 'mail.compose.message'

    def send_mail(self, auto_commit=False):
        if self.env.context.get('mark_rfq_as_sent') and self.model == 'purchase.order':
            self = self.with_context(mail_notify_author=self.env.user.partner_id in self.partner_ids)
        return super(MailComposeMessage, self).send_mail(auto_commit=auto_commit)

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
    purchase_line_warn = fields.Selection(WARNING_MESSAGE, 'Purchase Order Line Warning', help=WARNING_HELP, required=True, default="no-message")
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
        action = self.env["ir.actions.actions"]._for_xml_id("purchase.action_purchase_order_report_all")
        action['domain'] = ['&', ('state', 'in', ['purchase', 'done']), ('product_tmpl_id', 'in', self.ids)]
        action['context'] = {
            'graph_measure': 'qty_ordered',
            'search_default_later_than_a_year_ago': True
        }
        return action


class ProductProduct(models.Model):
    _name = 'product.product'
    _inherit = 'product.product'

    purchased_product_qty = fields.Float(compute='_compute_purchased_product_qty', string='Purchased')

    def _compute_purchased_product_qty(self):
        date_from = fields.Datetime.to_string(fields.Date.context_today(self) - relativedelta(years=1))
        domain = [
            ('order_id.state', 'in', ['purchase', 'done']),
            ('product_id', 'in', self.ids),
            ('order_id.date_approve', '>=', date_from)
        ]
        order_lines = self.env['purchase.order.line'].read_group(domain, ['product_id', 'product_uom_qty'], ['product_id'])
        purchased_data = dict([(data['product_id'][0], data['product_uom_qty']) for data in order_lines])
        for product in self:
            if not product.id:
                product.purchased_product_qty = 0.0
                continue
            product.purchased_product_qty = float_round(purchased_data.get(product.id, 0), precision_rounding=product.uom_id.rounding)

    def action_view_po(self):
        action = self.env["ir.actions.actions"]._for_xml_id("purchase.action_purchase_order_report_all")
        action['domain'] = ['&', ('state', 'in', ['purchase', 'done']), ('product_id', 'in', self.ids)]
        action['context'] = {
            'graph_measure': 'qty_ordered',
            'search_default_later_than_a_year_ago': True
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
from datetime import datetime, time
from dateutil.relativedelta import relativedelta
from itertools import groupby
from pytz import timezone, UTC
from werkzeug.urls import url_encode

from odoo import api, fields, models, _
from odoo.osv import expression
from odoo.tools import DEFAULT_SERVER_DATETIME_FORMAT
from odoo.tools.float_utils import float_is_zero
from odoo.exceptions import AccessError, UserError, ValidationError
from odoo.tools.misc import formatLang, get_lang, format_amount


class PurchaseOrder(models.Model):
    _name = "purchase.order"
    _inherit = ['portal.mixin', 'mail.thread', 'mail.activity.mixin']
    _description = "Purchase Order"
    _order = 'priority desc, id desc'

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

    READONLY_STATES = {
        'purchase': [('readonly', True)],
        'done': [('readonly', True)],
        'cancel': [('readonly', True)],
    }

    name = fields.Char('Order Reference', required=True, index=True, copy=False, default='New')
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
    date_order = fields.Datetime('Order Deadline', required=True, states=READONLY_STATES, index=True, copy=False, default=fields.Datetime.now,
        help="Depicts the date within which the Quotation should be confirmed and converted into a purchase order.")
    date_approve = fields.Datetime('Confirmation Date', readonly=1, index=True, copy=False)
    partner_id = fields.Many2one('res.partner', string='Vendor', required=True, states=READONLY_STATES, change_default=True, tracking=True, domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]", help="You can find a vendor by its Name, TIN, Email or Internal Reference.")
    dest_address_id = fields.Many2one('res.partner', domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]", string='Drop Ship Address', states=READONLY_STATES,
        help="Put an address if you want to deliver directly from the vendor to the customer. "
             "Otherwise, keep empty to deliver to your own company.")
    currency_id = fields.Many2one('res.currency', 'Currency', required=True, states=READONLY_STATES,
        default=lambda self: self.env.company.currency_id.id)
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
    date_planned = fields.Datetime(
        string='Receipt Date', index=True, copy=False, compute='_compute_date_planned', store=True, readonly=False,
        help="Delivery date promised by vendor. This date is used to determine expected arrival of products.")
    date_calendar_start = fields.Datetime(compute='_compute_date_calendar_start', readonly=True, store=True)

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

    mail_reminder_confirmed = fields.Boolean("Reminder Confirmed", default=False, readonly=True, copy=False, help="True if the reminder email is confirmed by the vendor.")
    mail_reception_confirmed = fields.Boolean("Reception Confirmed", default=False, readonly=True, copy=False, help="True if PO reception is confirmed by the vendor.")

    receipt_reminder_email = fields.Boolean('Receipt Reminder Email', related='partner_id.receipt_reminder_email', readonly=False)
    reminder_date_before_receipt = fields.Integer('Days Before Receipt', related='partner_id.reminder_date_before_receipt', readonly=False)

    @api.constrains('company_id', 'order_line')
    def _check_order_line_company_id(self):
        for order in self:
            companies = order.order_line.product_id.company_id
            if companies and companies != order.company_id:
                bad_products = order.order_line.product_id.filtered(lambda p: p.company_id and p.company_id != order.company_id)
                raise ValidationError(_(
                    "Your quotation contains products from company %(product_company)s whereas your quotation belongs to company %(quote_company)s. \n Please change the company of your quotation or remove the products from other companies (%(bad_products)s).",
                    product_company=', '.join(companies.mapped('display_name')),
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

    @api.model
    def _name_search(self, name, args=None, operator='ilike', limit=100, name_get_uid=None):
        args = args or []
        domain = []
        if name:
            domain = ['|', ('name', operator, name), ('partner_ref', operator, name)]
        return self._search(expression.AND([domain, args]), limit=limit, access_rights_uid=name_get_uid)

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
                order.date_planned = fields.Datetime.to_string(min(dates_list))
            else:
                order.date_planned = False

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

    @api.model
    def create(self, vals):
        company_id = vals.get('company_id', self.default_get(['company_id'])['company_id'])
        # Ensures default picking type and currency are taken from the right company.
        self_comp = self.with_company(company_id)
        if vals.get('name', 'New') == 'New':
            seq_date = None
            if 'date_order' in vals:
                seq_date = fields.Datetime.context_timestamp(self, fields.Datetime.to_datetime(vals['date_order']))
            vals['name'] = self_comp.env['ir.sequence'].next_by_code('purchase.order', sequence_date=seq_date) or '/'
        vals, partner_vals = self._write_partner_values(vals)
        res = super(PurchaseOrder, self_comp).create(vals)
        if partner_vals:
            res.sudo().write(partner_vals)  # Because the purchase user doesn't have write on `res.partner`
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
            if line.product_id:
                seller = line.product_id._select_seller(
                    partner_id=line.partner_id, quantity=line.product_qty,
                    date=line.order_id.date_order and line.order_id.date_order.date(), uom_id=line.product_uom)
                line.date_planned = line._get_date_planned(seller)
        return new_po

    def _must_delete_date_planned(self, field_name):
        # To be overridden
        return field_name == 'order_line'

    def onchange(self, values, field_name, field_onchange):
        """Override onchange to NOT to update all date_planned on PO lines when
        date_planned on PO is updated by the change of date_planned on PO lines.
        """
        result = super(PurchaseOrder, self).onchange(values, field_name, field_onchange)
        if self._must_delete_date_planned(field_name) and 'value' in result:
            already_exist = [ol[1] for ol in values.get('order_line', []) if ol[1]]
            for line in result['value'].get('order_line', []):
                if line[0] < 2 and 'date_planned' in line[2] and line[1] in already_exist:
                    del line[2]['date_planned']
        return result

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

    def _get_report_base_filename(self):
        self.ensure_one()
        return 'Purchase Order-%s' % (self.name)

    @api.onchange('partner_id', 'company_id')
    def onchange_partner_id(self):
        # Ensures all properties and fiscal positions
        # are taken with the company of the order
        # if not defined, with_company doesn't change anything.
        self = self.with_company(self.company_id)
        if not self.partner_id:
            self.fiscal_position_id = False
            self.currency_id = self.env.company.currency_id.id
        else:
            self.fiscal_position_id = self.env['account.fiscal.position'].get_fiscal_position(self.partner_id.id)
            self.payment_term_id = self.partner_id.property_supplier_payment_term_id.id
            self.currency_id = self.partner_id.property_purchase_currency_id.id or self.env.company.currency_id.id
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

    @api.returns('mail.message', lambda value: value.id)
    def message_post(self, **kwargs):
        if self.env.context.get('mark_rfq_as_sent'):
            self.filtered(lambda o: o.state == 'draft').write({'state': 'sent'})
        return super(PurchaseOrder, self.with_context(mail_post_autofollow=True)).message_post(**kwargs)

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
        moves.filtered(lambda m: m.currency_id.round(m.amount_total) < 0).action_switch_invoice_into_refund_credit_note()

        return self.action_view_invoice(moves)

    def _prepare_invoice(self):
        """Prepare the dict of values to create the new invoice for a purchase order.
        """
        self.ensure_one()
        move_type = self._context.get('default_move_type', 'in_invoice')
        journal = self.env['account.move'].with_context(default_move_type=move_type)._get_default_journal()
        if not journal:
            raise UserError(_('Please define an accounting purchase journal for the company %s (%s).') % (self.company_id.name, self.company_id.id))

        partner_invoice_id = self.partner_id.address_get(['invoice'])['invoice']
        partner_bank_id = self.partner_id.commercial_partner_id.bank_ids.filtered_domain(['|', ('company_id', '=', False), ('company_id', '=', self.company_id.id)])[:1]
        invoice_vals = {
            'ref': self.partner_ref or '',
            'move_type': move_type,
            'narration': self.notes,
            'currency_id': self.currency_id.id,
            'invoice_user_id': self.user_id and self.user_id.id or self.env.user.id,
            'partner_id': partner_invoice_id,
            'fiscal_position_id': (self.fiscal_position_id or self.fiscal_position_id.get_fiscal_position(partner_invoice_id)).id,
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
            # Invoice_ids may be filtered depending on the user. To ensure we get all
            # invoices related to the purchase order, we read them in sudo to fill the
            # cache.
            self.sudo()._read(['invoice_ids'])
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

        # Get list of translation values
        Translation = self.env['ir.translation']
        list_old_value_char = []
        list_new_value_char = []
        field_name = 'ir.model.fields.selection,name'
        for lang in self.env['res.lang'].search_read([], ['code']):
            list_old_value_char.append(Translation._get_source(field_name, 'model', lang['code'], source='RFQ'))
            list_new_value_char.append(Translation._get_source(field_name, 'model', lang['code'], source='RFQ Sent'))

        # This query is brittle since it depends on the label values of a selection field
        # not changing, but we don't have a direct time tracker of when a state changes
        query = """SELECT COUNT(1)
                   FROM mail_tracking_value v
                   LEFT JOIN mail_message m ON (v.mail_message_id = m.id)
                   JOIN purchase_order po ON (po.id = m.res_id)
                   WHERE m.create_date >= %s
                     AND m.model = 'purchase.order'
                     AND m.message_type = 'notification'
                     AND v.old_value_char IN %s
                     AND v.new_value_char IN %s
                     AND po.company_id = %s;
                """

        self.env.cr.execute(query, (one_week_ago, tuple(list_old_value_char), tuple(list_new_value_char), self.env.company.id))
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
        # 'total last 7 days' takes into account exchange rate and current company's currency's precision. Min of currency precision
        # is taken to easily extract it from query.
        # This is done via SQL for scalability reasons
        query = """SELECT AVG(COALESCE(po.amount_total / NULLIF(po.currency_rate, 0), po.amount_total)),
                          AVG(extract(epoch from age(po.date_approve,po.create_date)/(24*60*60)::decimal(16,2))),
                          SUM(CASE WHEN po.date_approve >= %s THEN COALESCE(po.amount_total / NULLIF(po.currency_rate, 0), po.amount_total) ELSE 0 END)
                   FROM purchase_order po
                   JOIN res_company comp ON (po.company_id = comp.id)
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
                    order.with_context(is_reminder=True).message_post_with_template(template.id, email_layout_xmlid="mail.mail_notification_paynow", composition_mode='comment')

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
                email_values={'email_to': self.env.user.email, 'recipient_ids': []},
                notif_layout="mail.mail_notification_paynow")
            return {'toast_message': _("A sample email has been sent to %s.") % self.env.user.email}

    @api.model
    def _get_orders_to_remind(self):
        """When auto sending a reminder mail, only send for unconfirmed purchase
        order and not all products are service."""
        return self.search([
            ('receipt_reminder_email', '=', True),
            ('state', 'in', ['purchase', 'done']),
            ('mail_reminder_confirmed', '=', False)
        ]).filtered(lambda p: p.mapped('order_line.product_id.product_tmpl_id.type') != ['service'])

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
                date = confirmed_date or self.date_planned.date()
                order.message_post(body=_("%(name)s confirmed the receipt will take place on %(date)s.", name=order.partner_id.name, date=date))

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
                order.message_post(body=_("The order receipt has been acknowledged by %(name)s.", name=order.partner_id.name))

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

    def _create_update_date_activity(self, updated_dates):
        note = _('<p> %s modified receipt dates for the following products:</p>') % self.partner_id.name
        for line, date in updated_dates:
            note += _('<p> &nbsp; - %s from %s to %s </p>') % (line.product_id.display_name, line.date_planned.date(), date.date())
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
            activity.note += _('<p> &nbsp; - %s from %s to %s </p>') % (line.product_id.display_name, line.date_planned.date(), date.date())

    def _write_partner_values(self, vals):
        partner_values = {}
        if 'receipt_reminder_email' in vals:
            partner_values['receipt_reminder_email'] = vals.pop('receipt_reminder_email')
        if 'reminder_date_before_receipt' in vals:
            partner_values['reminder_date_before_receipt'] = vals.pop('reminder_date_before_receipt')
        return vals, partner_values


class PurchaseOrderLine(models.Model):
    _name = 'purchase.order.line'
    _description = 'Purchase Order Line'
    _order = 'order_id, sequence, id'

    name = fields.Text(string='Description', required=True)
    sequence = fields.Integer(string='Sequence', default=10)
    product_qty = fields.Float(string='Quantity', digits='Product Unit of Measure', required=True)
    product_uom_qty = fields.Float(string='Total Quantity', compute='_compute_product_uom_qty', store=True)
    date_planned = fields.Datetime(string='Delivery Date', index=True,
        help="Delivery date expected from vendor. This date respectively defaults to vendor pricelist lead time then today's date.")
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
    account_analytic_id = fields.Many2one('account.analytic.account', store=True, string='Analytic Account', compute='_compute_account_analytic_id', readonly=False)
    analytic_tag_ids = fields.Many2many('account.analytic.tag', store=True, string='Analytic Tags', compute='_compute_analytic_tag_ids', readonly=False)
    company_id = fields.Many2one('res.company', related='order_id.company_id', string='Company', store=True, readonly=True)
    state = fields.Selection(related='order_id.state', store=True, readonly=False)

    invoice_lines = fields.One2many('account.move.line', 'purchase_line_id', string="Bill Lines", readonly=True, copy=False)

    # Replace by invoiced Qty
    qty_invoiced = fields.Float(compute='_compute_qty_invoiced', string="Billed Qty", digits='Product Unit of Measure', store=True)

    qty_received_method = fields.Selection([('manual', 'Manual')], string="Received Qty Method", compute='_compute_qty_received_method', store=True,
        help="According to product configuration, the received quantity can be automatically computed by mechanism :\n"
             "  - Manual: the quantity is set manually on the line\n"
             "  - Stock Moves: the quantity comes from confirmed pickings\n")
    qty_received = fields.Float("Received Qty", compute='_compute_qty_received', inverse='_inverse_qty_received', compute_sudo=True, store=True, digits='Product Unit of Measure')
    qty_received_manual = fields.Float("Manual Received Qty", digits='Product Unit of Measure', copy=False)
    qty_to_invoice = fields.Float(compute='_compute_qty_invoiced', string='To Invoice Quantity', store=True, readonly=True,
                                  digits='Product Unit of Measure')

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
            line = line.with_company(line.company_id)
            fpos = line.order_id.fiscal_position_id or line.order_id.fiscal_position_id.get_fiscal_position(line.order_id.partner_id.id)
            # filter taxes by company
            taxes = line.product_id.supplier_taxes_id.filtered(lambda r: r.company_id == line.env.company)
            line.taxes_id = fpos.map_tax(taxes, line.product_id, line.order_id.partner_id)

    @api.depends('invoice_lines.move_id.state', 'invoice_lines.quantity', 'qty_received', 'product_uom_qty', 'order_id.state')
    def _compute_qty_invoiced(self):
        for line in self:
            # compute qty_invoiced
            qty = 0.0
            for inv_line in line.invoice_lines:
                if inv_line.move_id.state not in ['cancel']:
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
                msg = _("Extra line with %s ") % (line.product_id.display_name,)
                line.order_id.message_post(body=msg)
        return lines

    def write(self, values):
        if 'display_type' in values and self.filtered(lambda line: line.display_type != values.get('display_type')):
            raise UserError(_("You cannot change the type of a purchase order line. Instead you should delete the current line and create a new line of the proper type."))

        if 'product_qty' in values:
            for line in self:
                if line.order_id.state == 'purchase':
                    line.order_id.message_post_with_view('purchase.track_po_line_template',
                                                         values={'line': line, 'product_qty': values['product_qty']},
                                                         subtype_id=self.env.ref('mail.mt_note').id)
        if 'qty_received' in values:
            for line in self:
                line._track_qty_received(values['qty_received'])
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
            date_planned = date_order + relativedelta(days=seller.delay if seller else 0)
        else:
            date_planned = datetime.today() + relativedelta(days=seller.delay if seller else 0)
        return self._convert_to_middle_of_day(date_planned)

    @api.depends('product_id', 'date_order')
    def _compute_account_analytic_id(self):
        for rec in self:
            if not rec.display_type:
                default_analytic_account = rec.env['account.analytic.default'].sudo().account_get(
                    product_id=rec.product_id.id,
                    partner_id=rec.order_id.partner_id.id,
                    user_id=rec.env.uid,
                    date=rec.date_order,
                    company_id=rec.company_id.id,
                )
                if default_analytic_account:
                    rec.account_analytic_id = default_analytic_account.analytic_id

    @api.depends('product_id', 'date_order')
    def _compute_analytic_tag_ids(self):
        for rec in self:
            if not rec.display_type:
                default_analytic_account = rec.env['account.analytic.default'].sudo().account_get(
                    product_id=rec.product_id.id,
                    partner_id=rec.order_id.partner_id.id,
                    user_id=rec.env.uid,
                    date=rec.date_order,
                    company_id=rec.company_id.id,
                )
                if default_analytic_account:
                    rec.analytic_tag_ids = default_analytic_account.analytic_tag_ids

    @api.onchange('product_id', 'company_id')
    def onchange_product_id(self):
        if not self.product_id or not self.company_id:
            return

        # Reset date, price and quantity since _onchange_quantity will provide default values
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
            title = _("Warning for %s", product_info.name)
            message = product_info.purchase_line_warn_msg
            warning['title'] = title
            warning['message'] = message
            if product_info.purchase_line_warn == 'block':
                self.product_id = False
            return {'warning': warning}
        return {}

    @api.onchange('product_qty', 'product_uom', 'company_id')
    def _onchange_quantity(self):
        if not self.product_id or self.invoice_lines or not self.company_id:
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

        # If not seller, use the standard price. It needs a proper currency conversion.
        if not seller:
            po_line_uom = self.product_uom or self.product_id.uom_po_id
            price_unit = self.env['account.tax']._fix_tax_included_price_company(
                self.product_id.uom_id._compute_price(self.product_id.standard_price, po_line_uom),
                self.product_id.supplier_taxes_id,
                self.taxes_id,
                self.company_id,
            )
            if price_unit and self.order_id.currency_id and self.order_id.company_id.currency_id != self.order_id.currency_id:
                price_unit = self.order_id.company_id.currency_id._convert(
                    price_unit,
                    self.order_id.currency_id,
                    self.order_id.company_id,
                    self.date_order or fields.Date.today(),
                )

            self.price_unit = price_unit
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

    def _prepare_account_move_line(self, move=False):
        self.ensure_one()
        aml_currency = move and move.currency_id or self.currency_id
        date = move and move.date or fields.Date.today()
        res = {
            'display_type': self.display_type,
            'sequence': self.sequence,
            'name': '%s: %s' % (self.order_id.name, self.name),
            'product_id': self.product_id.id,
            'product_uom_id': self.product_uom.id,
            'quantity': self.qty_to_invoice,
            'price_unit': self.currency_id._convert(self.price_unit, aml_currency, self.company_id, date, round=False),
            'tax_ids': [(6, 0, self.taxes_id.ids)],
            'analytic_account_id': self.account_analytic_id.id,
            'analytic_tag_ids': [(6, 0, self.analytic_tag_ids.ids)],
            'purchase_line_id': self.id,
        }
        if not move:
            return res

        if self.currency_id == move.company_id.currency_id:
            currency = False
        else:
            currency = move.currency_id

        res.update({
            'move_id': move.id,
            'currency_id': currency and currency.id or False,
            'date_maturity': move.invoice_date_due,
            'partner_id': move.partner_id.id,
        })
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
        partner = supplier.name
        uom_po_qty = product_uom._compute_quantity(product_qty, product_id.uom_po_id)
        # _select_seller is used if the supplier have different price depending
        # the quantities ordered.
        today = fields.Date.today()
        seller = product_id.with_company(company_id)._select_seller(
            partner_id=partner,
            quantity=uom_po_qty,
            date=po.date_order and max(po.date_order.date(), today) or today,
            uom_id=product_id.uom_po_id)

        taxes = product_id.supplier_taxes_id
        fpos = po.fiscal_position_id
        taxes_id = fpos.map_tax(taxes, product_id, seller.name)
        if taxes_id:
            taxes_id = taxes_id.filtered(lambda x: x.company_id.id == company_id.id)

        price_unit = self.env['account.tax']._fix_tax_included_price_company(seller.price, product_id.supplier_taxes_id, taxes_id, company_id) if seller else 0.0
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

        return {
            'name': name,
            'product_qty': uom_po_qty,
            'product_id': product_id.id,
            'product_uom': product_id.uom_po_id.id,
            'price_unit': price_unit,
            'date_planned': date_planned,
            'taxes_id': [(6, 0, taxes_id.ids)],
            'order_id': po.id,
        }

    def _convert_to_middle_of_day(self, date):
        """Return a datetime which is the noon of the input date(time) according
        to order user's time zone, convert to UTC time.
        """
        tz = timezone(self.order_id.user_id.tz or self.company_id.partner_id.tz or 'UTC')
        date = date.astimezone(tz) # date is UTC, applying the offset could change the day
        return tz.localize(datetime.combine(date, time(12))).astimezone(UTC).replace(tzinfo=None)

    def _update_date_planned(self, updated_date):
        self.date_planned = updated_date

    def _track_qty_received(self, new_qty):
        self.ensure_one()
        if new_qty != self.qty_received and self.order_id.state == 'purchase':
            self.order_id.message_post_with_view(
                'purchase.track_po_line_qty_received_template',
                values={'line': self, 'qty_received': new_qty},
                subtype_id=self.env.ref('mail.mt_note').id
            )

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

    group_send_reminder = fields.Boolean("Receipt Reminder", implied_group='purchase.group_send_reminder', default=True,
        help="Allow automatically send email to remind your vendor the receipt date")

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
                    ('move_type', 'in', ('in_invoice', 'in_refund'))],
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

    receipt_reminder_email = fields.Boolean('Receipt Reminder', default=False, company_dependent=True,
        help="Automatically send a confirmation email to the vendor X days before the expected receipt date, asking him to confirm the exact date.")
    reminder_date_before_receipt = fields.Integer('Days Before Receipt', default=1, company_dependent=True,
        help="Number of days to send reminder email before the promised receipt date")

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
from . import mail_compose_message

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
        return purchase_bills_union_ids

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
                <div t-if="o.date_order" class="col-3 bm-2">
                    <strong>Order Date:</strong>
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

import re

from odoo import api, fields, models, tools
from odoo.exceptions import UserError
from odoo.osv.expression import AND, expression


class PurchaseReport(models.Model):
    _name = "purchase.report"
    _description = "Purchase Report"
    _auto = False
    _order = 'date_order desc, price_total desc'

    date_order = fields.Datetime('Order Date', readonly=True, help="Depicts the date when the Quotation should be validated and converted into a purchase order.")
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
    user_id = fields.Many2one('res.users', 'Purchase Representative', readonly=True)
    delay = fields.Float('Days to Confirm', digits=(16, 2), readonly=True, group_operator='avg', help="Amount of time between purchase approval and order by date.")
    delay_pass = fields.Float('Days to Receive', digits=(16, 2), readonly=True, group_operator='avg',
                              help="Amount of time between date planned and order by date for each purchase order line.")
    avg_days_to_purchase = fields.Float(
        'Average Days to Purchase', digits=(16, 2), readonly=True, store=False,  # needs store=False to prevent showing up as a 'measure' option
        help="Amount of time between purchase approval and document creation date. Due to a hack needed to calculate this, \
              every record will show the same average value, therefore only use this as an aggregated value with group_operator=avg")
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
                    po.currency_id,
                    t.uom_id as product_uom,
                    extract(epoch from age(po.date_approve,po.date_order))/(24*60*60)::decimal(16,2) as delay,
                    extract(epoch from age(l.date_planned,po.date_order))/(24*60*60)::decimal(16,2) as delay_pass,
                    count(*) as nbr_lines,
                    sum(l.price_total / COALESCE(po.currency_rate, 1.0))::decimal(16,2) * currency_table.rate as price_total,
                    (sum(l.product_qty * l.price_unit / COALESCE(po.currency_rate, 1.0))/NULLIF(sum(l.product_qty/line_uom.factor*product_uom.factor),0.0))::decimal(16,2) * currency_table.rate as price_average,
                    partner.country_id as country_id,
                    partner.commercial_partner_id as commercial_partner_id,
                    analytic_account.id as account_analytic_id,
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
                left join uom_uom line_uom on (line_uom.id=l.product_uom)
                left join uom_uom product_uom on (product_uom.id=t.uom_id)
                left join account_analytic_account analytic_account on (l.account_analytic_id = analytic_account.id)
                left join {currency_table} ON currency_table.company_id = po.company_id
        """.format(
            currency_table=self.env['res.currency']._get_query_currency_table({'multi_company': True, 'date': {'date_to': fields.Date.today()}}),
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
                po.id,
                currency_table.rate
        """
        return group_by_str

    @api.model
    def read_group(self, domain, fields, groupby, offset=0, limit=None, orderby=False, lazy=True):
        """ This is a hack to allow us to correctly calculate the average of PO specific date values since
            the normal report query result will duplicate PO values across its PO lines during joins and
            lead to incorrect aggregation values.

            Only the AVG operator is supported for avg_days_to_purchase.
        """
        avg_days_to_purchase = next((field for field in fields if re.search(r'\bavg_days_to_purchase\b', field)), False)

        if avg_days_to_purchase:
            fields.remove(avg_days_to_purchase)
            if any(field.split(':')[1].split('(')[0] != 'avg' for field in [avg_days_to_purchase] if field):
                raise UserError("Value: 'avg_days_to_purchase' should only be used to show an average. If you are seeing this message then it is being accessed incorrectly.")

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
        if not res and avg_days_to_purchase:
            res = [{}]

        if avg_days_to_purchase:
            self.check_access_rights('read')
            query = """ SELECT AVG(days_to_purchase.po_days_to_purchase)::decimal(16,2) AS avg_days_to_purchase
                          FROM (
                              SELECT extract(epoch from age(po.date_approve,po.create_date))/(24*60*60) AS po_days_to_purchase
                              FROM purchase_order po
                              WHERE po.id IN (
                                  SELECT "purchase_report"."order_id" FROM %s WHERE %s)
                              ) AS days_to_purchase
                    """

            subdomain = AND([domain, [('company_id', '=', self.env.company.id), ('date_approve', '!=', False)]])
            subtables, subwhere, subparams = expression(subdomain, self).query.get_sql()

            self.env.cr.execute(query % (subtables, subwhere), subparams)
            res[0].update({
                '__count': 1,
                avg_days_to_purchase.split(':')[0]: self.env.cr.fetchall()[0][0],
            })
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
                <pivot string="Purchase Analysis" disable_linking="True" display_quantity="true" sample="1">
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
                <graph string="Purchase Orders Statistics" type="line" sample="1" disable_linking="1">
                    <field name="date_approve" interval="day" type="col"/>
                    <field name="untaxed_total" type="measure"/>
                </graph>
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

    <menuitem id="purchase_report" name="Reporting" parent="purchase.menu_purchase_root" sequence="99" groups="purchase.group_purchase_manager" action="action_purchase_order_report_all"/>

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
access_account_type,account.acount.type.purchase.user,account.model_account_account_type,group_purchase_user,1,0,0,0
account_partial_reconcile,account.partial.reconcile.purchase.user,account.model_account_partial_reconcile,group_purchase_user,1,0,0,0
access_res_partner_purchase_manager,res.partner.purchase.manager,base.model_res_partner,group_purchase_manager,1,1,1,0
access_uom_category_purchase_manager,uom.category purchase_manager,uom.model_uom_category,purchase.group_purchase_manager,1,1,1,1
access_uom_uom_purchase_manager,uom.uom purchase_manager,uom.model_uom_uom,purchase.group_purchase_manager,1,1,1,1
access_product_category_purchase_manager,product.category purchase_manager,product.model_product_category,purchase.group_purchase_manager,1,1,1,1
access_product_template_purchase_manager,product.template purchase_manager,product.model_product_template,purchase.group_purchase_manager,1,1,1,1
access_product_packaging_purchase_manager,product.packaging purchase_manager,product.model_product_packaging,purchase.group_purchase_manager,1,1,1,1
access_product_supplierinfo_purchase_manager,product.supplierinfo purchase_manager,product.model_product_supplierinfo,purchase.group_purchase_manager,1,1,1,1
access_product_pricelist_item_purchase_manager,product.pricelist.item purchase_manager,product.model_product_pricelist_item,purchase.group_purchase_manager,1,1,1,1
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
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#94B6C8"/><stop offset="100%" stop-color="#6A9EBA"/></linearGradient><path id="d" d="M53.551 19.955H16.223c-2.07 0-3.742 1.64-3.742 3.663v26.864c0 2.022 1.673 3.663 3.742 3.663h37.328c2.07 0 3.742-1.64 3.742-3.663V23.618c0-2.023-1.672-3.663-3.742-3.663zm-36.86 3.663h36.393c.257 0 .467.206.467.458v3.205H16.223v-3.205c0-.252.21-.458.467-.458zm36.393 26.864H16.69a.464.464 0 0 1-.467-.458V37.05h37.328v12.974c0 .252-.21.458-.467.458zM27.42 42.85v3.053c0 .503-.42.916-.934.916h-5.601a.928.928 0 0 1-.934-.916V42.85c0-.504.42-.916.934-.916h5.601c.514 0 .934.412.934.916zm14.937 0v3.053c0 .503-.42.916-.934.916h-10.58a.928.928 0 0 1-.934-.916V42.85c0-.504.42-.916.934-.916h10.58c.514 0 .934.412.934.916z"/><path id="e" d="M53.551 17.955H16.223c-2.07 0-3.742 1.64-3.742 3.663v26.864c0 2.022 1.673 3.663 3.742 3.663h37.328c2.07 0 3.742-1.64 3.742-3.663V21.618c0-2.023-1.672-3.663-3.742-3.663zm-36.86 3.663h36.393c.257 0 .467.206.467.458v3.205H16.223v-3.205c0-.252.21-.458.467-.458zm36.393 26.864H16.69a.464.464 0 0 1-.467-.458V35.05h37.328v12.974c0 .252-.21.458-.467.458zM27.42 40.85v3.053c0 .503-.42.916-.934.916h-5.601a.928.928 0 0 1-.934-.916V40.85c0-.504.42-.916.934-.916h5.601c.514 0 .934.412.934.916zm14.937 0v3.053c0 .503-.42.916-.934.916h-10.58a.928.928 0 0 1-.934-.916V40.85c0-.504.42-.916.934-.916h10.58c.514 0 .934.412.934.916z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M43.95 70H4c-2 0-4-.149-4-4.163V38.095l13.403-18.86L56 20.04l1.006 29.996L43.95 70z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-opacity=".3" fill-rule="nonzero" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: static\src\js\purchase_dashboard.js

```javascript
odoo.define('purchase.dashboard', function (require) {
"use strict";

/**
 * This file defines the Purchase Dashboard view (alongside its renderer, model
 * and controller). This Dashboard is added to the top of list and kanban Purchase
 * views, it extends both views with essentially the same code except for
 * _onDashboardActionClicked function so we can apply filters without changing our
 * current view.
 */

var core = require('web.core');
var ListController = require('web.ListController');
var ListModel = require('web.ListModel');
var ListRenderer = require('web.ListRenderer');
var ListView = require('web.ListView');
var KanbanController = require('web.KanbanController');
var KanbanModel = require('web.KanbanModel');
var KanbanRenderer = require('web.KanbanRenderer');
var KanbanView = require('web.KanbanView');
var SampleServer = require('web.SampleServer');
var view_registry = require('web.view_registry');

var QWeb = core.qweb;

// Add mock of method 'retrieve_dashboard' in SampleServer, so that we can have
// the sample data in empty purchase kanban and list view
let dashboardValues;
SampleServer.mockRegistry.add('purchase.order/retrieve_dashboard', () => {
    return Object.assign({}, dashboardValues);
});


//--------------------------------------------------------------------------
// List View
//--------------------------------------------------------------------------

var PurchaseListDashboardRenderer = ListRenderer.extend({
    events:_.extend({}, ListRenderer.prototype.events, {
        'click .o_dashboard_action': '_onDashboardActionClicked',
    }),
    /**
     * @override
     * @private
     * @returns {Promise}
     */
    _renderView: function () {
        var self = this;
        return this._super.apply(this, arguments).then(function () {
            var values = self.state.dashboardValues;
            var purchase_dashboard = QWeb.render('purchase.PurchaseDashboard', {
                values: values,
            });
            self.$el.prepend(purchase_dashboard);
        });
    },

    /**
     * @private
     * @param {MouseEvent}
     */
    _onDashboardActionClicked: function (e) {
        e.preventDefault();
        var $action = $(e.currentTarget);
        this.trigger_up('dashboard_open_action', {
            action_name: $action.attr('name')+"_list",
            action_context: $action.attr('context'),
        });
    },
});

var PurchaseListDashboardModel = ListModel.extend({
    /**
     * @override
     */
    init: function () {
        this.dashboardValues = {};
        this._super.apply(this, arguments);
    },

    /**
     * @override
     */
    __get: function (localID) {
        var result = this._super.apply(this, arguments);
        if (_.isObject(result)) {
            result.dashboardValues = this.dashboardValues[localID];
        }
        return result;
    },
    /**
     * @override
     * @returns {Promise}
     */
    __load: function () {
        return this._loadDashboard(this._super.apply(this, arguments));
    },
    /**
     * @override
     * @returns {Promise}
     */
    __reload: function () {
        return this._loadDashboard(this._super.apply(this, arguments));
    },

    /**
     * @private
     * @param {Promise} super_def a promise that resolves with a dataPoint id
     * @returns {Promise -> string} resolves to the dataPoint id
     */
    _loadDashboard: function (super_def) {
        var self = this;
        var dashboard_def = this._rpc({
            model: 'purchase.order',
            method: 'retrieve_dashboard',
        });
        return Promise.all([super_def, dashboard_def]).then(function(results) {
            var id = results[0];
            dashboardValues = results[1];
            self.dashboardValues[id] = dashboardValues;
            return id;
        });
    },
});

var PurchaseListDashboardController = ListController.extend({
    custom_events: _.extend({}, ListController.prototype.custom_events, {
        dashboard_open_action: '_onDashboardOpenAction',
    }),

    /**
     * @private
     * @param {OdooEvent} e
     */
    _onDashboardOpenAction: function (e) {
        return this.do_action(e.data.action_name,
            {additional_context: JSON.parse(e.data.action_context)});
    },
});

var PurchaseListDashboardView = ListView.extend({
    config: _.extend({}, ListView.prototype.config, {
        Model: PurchaseListDashboardModel,
        Renderer: PurchaseListDashboardRenderer,
        Controller: PurchaseListDashboardController,
    }),
});

//--------------------------------------------------------------------------
// Kanban View
//--------------------------------------------------------------------------

var PurchaseKanbanDashboardRenderer = KanbanRenderer.extend({
    events:_.extend({}, KanbanRenderer.prototype.events, {
        'click .o_dashboard_action': '_onDashboardActionClicked',
    }),
    /**
     * @override
     * @private
     * @returns {Promise}
     */
    _render: function () {
        var self = this;
        return this._super.apply(this, arguments).then(function () {
            var values = self.state.dashboardValues;
            var purchase_dashboard = QWeb.render('purchase.PurchaseDashboard', {
                values: values,
            });
            self.$el.parent().find(".o_purchase_dashboard").remove();
            self.$el.before(purchase_dashboard);
        });
    },

    /**
     * @private
     * @param {MouseEvent}
     */
    _onDashboardActionClicked: function (e) {
        e.preventDefault();
        var $action = $(e.currentTarget);
        this.trigger_up('dashboard_open_action', {
            action_name: $action.attr('name')+"_kanban",
            action_context: $action.attr('context'),
        });
    },
});

var PurchaseKanbanDashboardModel = KanbanModel.extend({
    /**
     * @override
     */
    init: function () {
        this.dashboardValues = {};
        this._super.apply(this, arguments);
    },

    /**
     * @override
     */
    __get: function (localID) {
        var result = this._super.apply(this, arguments);
        if (_.isObject(result)) {
            result.dashboardValues = this.dashboardValues[localID];
        }
        return result;
    },
    /**
     * @override
     * @returns {Promise}
     */
    __load: function () {
        return this._loadDashboard(this._super.apply(this, arguments));
    },
    /**
     * @override
     * @returns {Promise}
     */
    __reload: function () {
        return this._loadDashboard(this._super.apply(this, arguments));
    },

    /**
     * @private
     * @param {Promise} super_def a promise that resolves with a dataPoint id
     * @returns {Promise -> string} resolves to the dataPoint id
     */
    _loadDashboard: function (super_def) {
        var self = this;
        var dashboard_def = this._rpc({
            model: 'purchase.order',
            method: 'retrieve_dashboard',
        });
        return Promise.all([super_def, dashboard_def]).then(function(results) {
            var id = results[0];
            dashboardValues = results[1];
            self.dashboardValues[id] = dashboardValues;
            return id;
        });
    },
});

var PurchaseKanbanDashboardController = KanbanController.extend({
    custom_events: _.extend({}, KanbanController.prototype.custom_events, {
        dashboard_open_action: '_onDashboardOpenAction',
    }),

    /**
     * @private
     * @param {OdooEvent} e
     */
    _onDashboardOpenAction: function (e) {
        return this.do_action(e.data.action_name,
            {additional_context: JSON.parse(e.data.action_context)});
    },
});

var PurchaseKanbanDashboardView = KanbanView.extend({
    config: _.extend({}, KanbanView.prototype.config, {
        Model: PurchaseKanbanDashboardModel,
        Renderer: PurchaseKanbanDashboardRenderer,
        Controller: PurchaseKanbanDashboardController,
    }),
});

view_registry.add('purchase_list_dashboard', PurchaseListDashboardView);
view_registry.add('purchase_kanban_dashboard', PurchaseKanbanDashboardView);

return {
    PurchaseListDashboardModel: PurchaseListDashboardModel,
    PurchaseListDashboardRenderer: PurchaseListDashboardRenderer,
    PurchaseListDashboardController: PurchaseListDashboardController,
    PurchaseKanbanDashboardModel: PurchaseKanbanDashboardModel,
    PurchaseKanbanDashboardRenderer: PurchaseKanbanDashboardRenderer,
    PurchaseKanbanDashboardController: PurchaseKanbanDashboardController
};

});

```

## File: static\src\js\purchase_datetimepicker.js

```javascript
$(function () {
    $('input.o-purchase-datetimepicker').datetimepicker();
    $('input.o-purchase-datetimepicker').on("hide.datetimepicker", function () {
        $(this).parents('form').submit();
    });
})

```

## File: static\src\js\purchase_toaster_button.js

```javascript
odoo.define('purchase.ToasterButton', function (require) {
    'use strict';

    const widgetRegistry = require('web.widget_registry');
    const Widget = require('web.Widget');


    const ToasterButton = Widget.extend({
        template: 'purchase.ToasterButton',
        xmlDependencies: ['/purchase/static/src/xml/purchase_toaster_button.xml'],
        events: Object.assign({}, Widget.prototype.events, {
            'click .fa-info-circle': '_onClickButton',
        }),

        init: function (parent, data, node) {
            this._super(...arguments);
            this.button_name = node.attrs.button_name;
            this.title = node.attrs.title;
            this.id = data.res_id;
            this.model = data.model;
        },

        //--------------------------------------------------------------------------
        // Handlers
        //--------------------------------------------------------------------------
        _onClickButton: function (ev) {
            this._rpc({
                method: this.button_name,
                model: this.model,
                args: [[this.id]],
            }).then(res => {
                if (res) {
                    this.do_notify(false, res.toast_message);
                }
            })
        },
    });

    widgetRegistry.add('toaster_button', ToasterButton);

    return ToasterButton;
});

```

## File: static\src\js\tours\purchase.js

```javascript
odoo.define('purchase.purchase_steps', function (require) {
"use strict";

var core = require('web.core');

var PurchaseAdditionalTourSteps = core.Class.extend({

    _get_purchase_stock_steps: function () {
        return [
            {
                auto: true, // Useless final step to trigger congratulation message
                trigger: ".o_purchase_order",
            },
        ];
    },

});

return PurchaseAdditionalTourSteps;

});

odoo.define('purchase.tour', function(require) {
"use strict";

var core = require('web.core');
var tour = require('web_tour.tour');

var _t = core._t;
var PurchaseAdditionalTourSteps = require('purchase.purchase_steps');

tour.register('purchase_tour' , {
    url: "/web",
    sequence: 40,
}, [tour.stepUtils.showAppsMenuItem(), {
    trigger: '.o_app[data-menu-xmlid="purchase.menu_purchase_root"]',
    content: _t("Let's try the Purchase app to manage the flow from purchase to reception and invoice control."),
    position: 'right',
    edition: 'community'
}, {
    trigger: '.o_app[data-menu-xmlid="purchase.menu_purchase_root"]',
    content: _t("Let's try the Purchase app to manage the flow from purchase to reception and invoice control."),
    position: 'bottom',
    edition: 'enterprise'
}, {
    trigger: ".o_list_button_add",
    extra_trigger: ".o_purchase_order",
    content: _t("Let's create your first request for quotation."),
    position: "bottom",
}, {
    trigger: ".o_form_editable .o_field_many2one[name='partner_id']",
    extra_trigger: ".o_purchase_order",
    content: _t("Search a vendor name, or create one on the fly."),
    position: "bottom",
    run: function (actions) {
        actions.text("Agrolait", this.$anchor.find("input"));
    },
}, {
    trigger: ".ui-menu-item > a",
    auto: true,
    in_modal: false,
}, {
    trigger: ".o_field_x2many_list_row_add > a",
    extra_trigger: ".o_field_many2one[name='partner_id'] .o_external_button",
    content: _t("Add some products or services to your quotation."),
    position: "bottom",
}, {
    trigger: ".o_field_widget[name=product_id], .o_field_widget[name=product_template_id]",
    extra_trigger: ".o_purchase_order",
    content: _t("Select a product, or create a new one on the fly."),
    position: "right",
    run: function (actions) {
        var $input = this.$anchor.find('input');
        actions.text("DESK0001", $input.length === 0 ? this.$anchor : $input);
        // fake keydown to trigger search
        var keyDownEvent = jQuery.Event("keydown");
        keyDownEvent.which = 42;
        this.$anchor.trigger(keyDownEvent);
        var $descriptionElement = $('.o_form_editable textarea[name="name"]');
        // when description changes, we know the product has been created
        $descriptionElement.change(function () {
            $descriptionElement.addClass('product_creation_success');
        });
    },
}, {
    trigger: '.ui-menu.ui-widget .ui-menu-item a:contains("DESK0001")',
    auto: true,
}, {
    trigger: '.o_form_editable textarea[name="name"].product_creation_success',
    auto: true,
    run: function () {} // wait for product creation
}, {
    trigger: ".o_form_editable input[name='product_qty'] ",
    extra_trigger: ".o_purchase_order",
    content: _t("Indicate the product quantity you want to order."),
    position: "right",
    run: 'text 12.0'
},
...tour.stepUtils.statusbarButtonsSteps('Send by Email', _t("Send the request for quotation to your vendor."), ".o_statusbar_buttons button[name='action_rfq_send']"),
{
    trigger: ".modal-content",
    auto: true,
    run: function(actions){
        // Check in case user must add email to vendor
        var $input = $(".modal-content input[name='email']");
        if ($input.length) {
            actions.text("agrolait@example.com", $input);
            actions.click($(".modal-footer button"));
        }
    }
}, {
    trigger: ".modal-footer button[name='action_send_mail']",
    extra_trigger: ".modal-footer button[name='action_send_mail']",
    content: _t("Send the request for quotation to your vendor."),
    position: "left",
    run: 'click',
}, {
    trigger: ".ui-sortable-handle",
    extra_trigger: ".o_purchase_order",
    content: _t("Click here to edit or move the quotation line."),
    position: "bottom",
    run: 'click',
}, {
    trigger: ".o_form_editable input[name='price_unit']",
    extra_trigger: ".o_form_editable input[name='price_unit']",
    content: _t("Once you get the price from the vendor, you can complete the purchase order with the right price."),
    position: "right",
    run: 'text 200.00'
}, {
    auto: true,
    trigger: ".o_purchase_order",
    run: 'click',
}, ...tour.stepUtils.statusbarButtonsSteps('Confirm Order', _t("Confirm your purchase.")),
...new PurchaseAdditionalTourSteps()._get_purchase_stock_steps(),
]);

});

```

## File: static\src\xml\purchase_dashboard.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <!-- This template is for a table at the top of purchase views that shows some KPIs. -->
    <t t-name="purchase.PurchaseDashboard">
        <div class="o_purchase_dashboard container">
        <div class="row">
            <div class="col-sm-5">
            <table class="table table-sm">
                <!-- thead needed to avoid list view rendering error for some reason -->
                <thead>
                    <tr>
                    <!-- can't use th tag due to list rendering error when no values in list... -->
                        <td class="o_text">
                            <div>All RFQs</div>
                        </td>
                        <td class="o_main o_dashboard_action" title="All Draft RFQs" name="purchase.purchase_action_dashboard" context='{"search_default_draft_rfqs": true}'>
                            <a href="#"><t t-esc="values['all_to_send']"/><br/>To Send</a>
                        </td>
                        <td class="o_main o_dashboard_action" title="All Waiting RFQs" name="purchase.purchase_action_dashboard" context='{"search_default_waiting_rfqs": true}'>
                            <a href="#"><t t-esc="values['all_waiting']"/><br/>Waiting</a>
                        </td>
                        <td class="o_main o_dashboard_action" title="All Late RFQs" name="purchase.purchase_action_dashboard" context='{"search_default_late_rfqs": true}'>
                            <a href="#"><t t-esc="values['all_late']"/><br/>Late</a>
                        </td>
                    </tr>
                </thead>
                <tbody>
                    <tr>
                        <td class="o_text">
                            <div>My RFQs</div>
                        </td>
                        <td class="o_main o_dashboard_action" title="My Draft RFQs" name="purchase.purchase_action_dashboard" context='{"search_default_draft_rfqs": true, "search_default_my_purchases": true}'>
                            <a href="#"><t t-esc="values['my_to_send']"/></a>
                        </td>
                        <td class="o_main o_dashboard_action" title="My Waiting RFQs" name="purchase.purchase_action_dashboard" context='{"search_default_waiting_rfqs": true, "search_default_my_purchases": true}'>
                            <a href="#"><t t-esc="values['my_waiting']"/></a>
                        </td>
                        <td class="o_main o_dashboard_action" title="My Late RFQs" name="purchase.purchase_action_dashboard" context='{"search_default_late_rfqs": true, "search_default_my_purchases": true}'>
                            <a href="#"><t t-esc="values['my_late']"/></a>
                        </td>
                    </tr>
                </tbody>
            </table></div>

            <div class="col-sm-7">
            <table class="table table-sm">
                <!-- thead needed to avoid list view rendering error for some reason -->
                <thead>
                    <tr>
                        <!-- can't use th tag due to list rendering error when no values in list... -->
                        <td class="o_text">Avg Order Value (<t t-esc="values['company_currency_symbol']"/>)</td>
                        <td><span><t t-esc="values['all_avg_order_value']"/></span></td>
                        <td class="o_text">Purchased Last 7 Days (<t t-esc="values['company_currency_symbol']"/>)</td>
                        <td><span><t t-esc="values['all_total_last_7_days']"/></span></td>
                    </tr>
                </thead>
                <tbody>
                    <tr>
                        <td class="o_text">Lead Time to Purchase</td>
                        <td><span><t t-esc="values['all_avg_days_to_purchase']"/> &#160;Days</span></td>
                        <td class="o_text">RFQs Sent Last 7 Days</td>
                        <td><span><t t-esc="values['all_sent_rfqs']"/></span></td>
                    </tr>
                </tbody>
            </table></div>
        </div></div>
    </t>
</templates>

```

## File: static\src\xml\purchase_toaster_button.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<template>

    <t t-name="purchase.ToasterButton">
        <button t-att-name="widget.button_name" type="button" class="btn oe_inline btn-link" t-att-title="widget.title">
            <i class="fa fa-fw o_button_icon fa-info-circle"></i>
        </button>
    </t>

</template>

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
                <label for="purchase_vendor_bill_id" string="Auto-Complete" class="oe_edit_only"
                       attrs="{'invisible': ['|', ('state','!=','draft'), ('move_type', '!=', 'in_invoice')]}" />
                <field name="purchase_vendor_bill_id" nolabel="1"
                       attrs="{'invisible': ['|', ('state','!=','draft'), ('move_type', '!=', 'in_invoice')]}"
                       class="oe_edit_only"
                       domain="partner_id and [('company_id', '=', company_id), ('partner_id.commercial_partner_id', '=', commercial_partner_id)] or [('company_id', '=', company_id)]"
                       placeholder="Select a purchase order or an old bill"
                       context="{'show_total_amount': True}"
                       options="{'no_create': True, 'no_open': True}"/>
            </field>
            <label name="invoice_vendor_bill_id_label" position="attributes">
                <attribute name="invisible">1</attribute>
            </label>
            <field name="invoice_vendor_bill_id" position="attributes">
                <attribute name="invisible">1</attribute>
            </field>
            <!-- Add link to purchase_line_id to account.move.line -->
            <xpath expr="//field[@name='invoice_line_ids']/tree/field[@name='company_id']" position="after">
                <field name="purchase_line_id" invisible="1"/>
                <field name="purchase_order_id" attrs="{'column_invisible': [('parent.move_type', '!=', 'in_invoice')]}" optional="hide"/>
            </xpath>
            <xpath expr="//field[@name='line_ids']/tree/field[@name='company_id']" position="after">
                <field name="purchase_line_id" invisible="1"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\assets.xml

```xml
<?xml version="1.0"?>
<odoo>
    <template id="assets_backend" name="purchase assets" inherit_id="web.assets_backend">
        <xpath expr="." position="inside">
            <link rel="stylesheet" type="text/scss" href="/purchase/static/src/scss/purchase.scss"/>
            <script type="text/javascript" src="/purchase/static/src/js/purchase_dashboard.js"></script>
            <script type="text/javascript" src="/purchase/static/src/js/purchase_toaster_button.js"></script>
        </xpath>
        <xpath expr="script[last()]" position="after">
            <script type="text/javascript" src="/purchase/static/src/js/tours/purchase.js"></script>
        </xpath>
    </template>

    <template id="assets_frontend" name="purchase assets" inherit_id="web.assets_frontend">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/purchase/static/src/js/purchase_datetimepicker.js"></script>
        </xpath>
    </template>
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

  <template id="portal_my_home_purchase" name="Show Purchase Orders" customize_show="True" inherit_id="portal.portal_my_home" priority="25">
    <xpath expr="//div[hasclass('o_portal_docs')]" position="inside">
        <t t-call="portal.portal_docs_entry">
            <t t-set="title">Purchase Orders</t>
            <t t-set="url" t-value="'/my/purchase'"/>
            <t t-set="placeholder_count" t-value="'purchase_count'"/>
        </t>
    </xpath>
  </template>

  <template id="portal_my_purchase_orders" name="Portal: My Purchase Orders">
    <t t-call="portal.portal_layout">
      <t t-set="breadcrumbs_searchbar" t-value="True"/>
      <t t-call="portal.portal_searchbar"/>
      <t t-if="orders" t-call="portal.portal_table">
        <thead>
          <tr class="active">
            <th>Purchase Orders #</th>
            <th class="text-right">Confirmation Date</th>
            <th></th>
            <th class="text-right">Total</th>
          </tr>
        </thead>
        <tbody>
          <t t-foreach="orders" t-as="order">
            <tr>
              <td>
                <a t-att-href="order.get_portal_url()">
                  <t t-esc="order.name"/>
               </a>
              </td>
              <td class="text-right">
                <span t-field="order.date_approve"/>
              </td>
              <td>
                <t t-if="order.invoice_status == 'to invoice'">
                  <span class="badge badge-info"><i class="fa fa-fw fa-file-text"/> Waiting for Bill</span>
                </t>
                <t t-if="order.state == 'cancel'">
                  <span class="badge badge-secondary"><i class="fa fa-fw fa-remove"/> Cancelled</span>
                </t>
              </td>
              <td class="text-right">
                <span t-field="order.amount_total" t-options='{"widget": "monetary", "display_currency": order.currency_id}'/>
              </td>
            </tr>
          </t>
        </tbody>
      </t>
    </t>
  </template>

  <template id="portal_my_purchase_order" name="Portal: My Purchase Order">
    <t t-call="portal.portal_layout">
      <t t-set="purchase_order" t-value="order"/>
      <t t-set="o_portal_fullwidth_alert" groups="purchase.group_purchase_manager">
        <t t-call="portal.portal_back_in_edit_mode">
          <t t-set="backend_url" t-value="'/web#return_label=Website&amp;model=%s&amp;id=%s&amp;action=%s&amp;view_type=form' % (purchase_order._name, purchase_order.id, purchase_order.env.ref('purchase.purchase_rfq').id)"/>
        </t>
      </t>
      <div id="optional_placeholder"></div>
      <div class="container">
        <div class="row mt16 o_portal_purchase_sidebar">
          <!-- Sidebar -->
          <t t-call="portal.portal_record_sidebar">
            <t t-set="classes" t-value="'col-lg-auto d-print-none'"/>
              <t t-set="title">
                <h2 class="mb-0">
                  <b t-field="purchase_order.amount_total" data-id="total_amount"/>
                </h2>
              </t>
            <t t-set="entries">
              <ul class="list-group list-group-flush flex-wrap flex-row flex-lg-column">
                <li class="list-group-item flex-grow-1">
                  <div class="o_download_pdf btn-toolbar flex-sm-nowrap">
                    <div class="btn-group flex-grow-1 mr-1 mb-1">
                      <a class="btn btn-secondary btn-block o_download_btn" t-att-href="purchase_order.get_portal_url(report_type='pdf', download=True)" title="Download"><i class="fa fa-download"/> Download</a>
                    </div>
                    <div class="btn-group flex-grow-1 mb-1">
                        <a class="btn btn-secondary btn-block o_print_btn o_portal_invoice_print" t-att-href="purchase_order.get_portal_url(report_type='pdf')" id="print_invoice_report" title="Print" target="_blank"><i class="fa fa-print"/> Print</a>
                    </div>
                  </div>
                </li>

                <li t-if="purchase_order.user_id" class="list-group-item flex-grow-1">
                  <div class="small mb-1">
                    <strong class="text-muted">Purchase Representative</strong>
                  </div>
                  <div class="row flex-nowrap">
                    <div class="col flex-grow-0 pr-2">
                      <img class="rounded-circle mr4 float-left o_portal_contact_img" t-if="purchase_order.user_id.image_1024" t-att-src="image_data_uri(purchase_order.user_id.image_1024)" alt="Contact"/>
                      <img class="rounded-circle mr4 float-left o_portal_contact_img" t-if="not purchase_order.user_id.image_1024" src="/web/static/src/img/placeholder.png" alt="Contact"/>
                    </div>
                    <div class="col pl-0" style="min-width: 150px">
                      <span t-field="purchase_order.user_id" t-options='{"widget": "contact", "fields": ["name", "phone"], "no_marker": True}'/>
                      <a href="#discussion" class="small"><i class="fa fa-comment"></i> Send message</a>
                    </div>
                  </div>
                </li>
              </ul>
            </t>
          </t>
          <div class=" col-lg col-12 justify-content-end w-100">
            <div class= "card pb-5">
              <div class="card-header bg-white pb-1">
                <div class="row">
                  <div class="col-lg-12">
                    <h2 class="font-weight-normal">
                      <t t-if="order.state in ['draft', 'sent']">Request for Quotation</t>
                      <t t-else="1">
                        Purchase Order
                      </t>
                      <span class="font-italic" t-esc="order.name"/>
                    </h2>
                  </div>
                </div>
              </div>
              <div class="card-body">
                <div class="mb-4">
                  <strong class="d-block mb-1">From:</strong>
                  <address t-field="order.company_id.partner_id" t-options='{"widget": "contact", "fields": ["address", "name"], "no_marker": True}'/>
                  <strong>Confirmation Date:</strong> <span t-field="order.date_approve" t-options='{"widget": "date"}'/><br/>
                  <div t-att-class="'d-inline' if order.date_planned else 'd-none'">
                    <strong>Receipt Date:</strong><span class="ml-1" t-field="order.date_planned" t-options='{"widget": "date"}'/>
                  </div>
                </div>
                <h3 class="font-weight-normal">Pricing</h3>
                <div class="table-responsive">
                  <table class="table table-sm">
                    <thead class="bg-100">
                      <tr>
                        <th>Products</th>
                        <th class="text-right d-none d-sm-table-cell" t-if="order.state in ['purchase', 'done']">Unit Price</th>
                        <th class="text-right">Quantity</th>
                        <th class="text-right" t-if="order.state in ['purchase', 'done']">Subtotal</th>
                      </tr>
                    </thead>
                    <tbody>
                      <t t-foreach="order.order_line" t-as="ol">
                        <tr t-att-class="'bg-200 font-weight-bold o_line_section' if ol.display_type == 'line_section' else 'font-italic o_line_note' if ol.display_type == 'line_note' else ''">
                          <t t-if="not ol.display_type">
                            <td>
                              <img t-att-src="image_data_uri(resize_to_48(ol.product_id.image_1024))" alt="Product" class="d-none d-lg-inline"/>
                              <span t-esc="ol.name"/>
                            </td>
                            <td class="text-right d-none d-sm-table-cell" t-if="order.state in ['purchase', 'done']">
                              <span t-field="ol.price_unit" t-options='{"widget": "monetary", "display_currency": order.currency_id}'/>
                            </td>
                            <td class="text-right">
                              <span t-esc="ol.product_qty"/>
                            </td>
                            <td class="text-right" t-if="order.state in ['purchase', 'done']">
                              <span t-field="ol.price_subtotal" t-options='{"widget": "monetary", "display_currency": order.currency_id}'/>
                            </td>
                          </t>
                          <t t-if="ol.display_type == 'line_section'">
                              <td colspan="99">
                                  <span t-field="ol.name"/>
                              </td>
                              <t t-set="current_section" t-value="line"/>
                              <t t-set="current_subtotal" t-value="0"/>
                          </t>
                          <t t-if="ol.display_type == 'line_note'">
                              <td colspan="99">
                                  <span t-field="ol.name"/>
                              </td>
                          </t>
                        </tr>
                      </t>
                    </tbody>
                  </table>
                </div>
                <div class="row" t-if="order.state in ['purchase', 'done']">
                  <div class="col-sm-7 col-md-5 ml-auto">
                    <table class="table table-sm">
                      <tbody>
                        <tr>
                          <td>
                            <strong>Untaxed Amount:</strong>
                          </td>
                          <td class="text-right">
                            <span t-field="order.amount_untaxed" t-options='{"widget": "monetary", "display_currency": order.currency_id}'/>
                          </td>
                        </tr>
                        <tr>
                          <td>
                            <strong>Taxes:</strong>
                          </td>
                          <td class="text-right">
                            <span t-field="order.amount_tax" t-options='{"widget": "monetary", "display_currency": order.currency_id}'/>
                          </td>
                        </tr>
                        <tr>
                          <td>
                            <strong>Total:</strong>
                          </td>
                          <td class="text-right">
                            <span t-field="order.amount_total" t-options='{"widget": "monetary", "display_currency": order.currency_id}'/>
                          </td>
                        </tr>
                      </tbody>
                    </table>
                  </div>
                </div>
              </div>
            </div>
            <div id="purchase_order_communication" class="mt-4">
              <h2>History</h2>
              <t t-call="portal.message_thread">
                <t t-set="object" t-value="purchase_order"/>
              </t>
            </div>
          </div>
        </div>
      </div>
      <div class="oe_structure mb32"/>
    </t>
  </template>

  <template id="portal_my_purchase_order_update_date" name="Portal: My Purchase Order Update Dates" inherit_id="purchase.portal_my_purchase_order" primary="True">
    <xpath expr="//div[hasclass('card-body')]" position="replace">
      <div class="card-body">
        <div class="mb-4">
          <strong>Confirmation Date:</strong> <span t-field="order.date_approve" t-options='{"widget": "date"}'/><br/>
          <div t-att-class="'d-inline' if order.date_planned else 'd-none'">
            <strong>Receipt Date:</strong><span class="ml-1" t-field="order.date_planned" t-options='{"widget": "date"}'/>
          </div>
        </div>
        <h3 class="font-weight-normal">Pricing</h3>
          <div class="table-responsive">
            <table class="table table-sm">
              <thead class="bg-100">
                <tr>
                  <th>Products</th>
                  <th class="text-right d-none d-sm-table-cell">Unit Price</th>
                  <th class="text-right">Quantity</th>
                  <th class="text-right">Scheduled Date</th>
                  <th class="text-right" style="color:#3aadaa"><strong>Update Dates Here</strong></th>
                  <th class="text-right">Subtotal</th>
                </tr>
              </thead>
              <tbody>
                <t t-foreach="order.order_line" t-as="ol">
                  <t t-if="not ol.display_type">
                    <tr>
                      <td>
                        <img t-att-src="image_data_uri(resize_to_48(ol.product_id.image_1024))" alt="Product" class="d-none d-lg-inline"/>
                        <span t-esc="ol.name"/>
                      </td>
                      <td class="text-right d-none d-sm-table-cell">
                        <span t-field="ol.price_unit" t-options='{"widget": "monetary", "display_currency": order.currency_id}'/>
                      </td>
                      <td class="text-right">
                        <span t-esc="ol.product_qty"/>
                      </td>
                      <td class="text-right">
                        <span t-esc="ol.date_planned.date()"/>
                      </td>
                      <td class="text-right">
                        <form t-attf-action="/my/purchase/#{order.id}/update?access_token=#{order.access_token}" method="post">
                          <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
                          <div class="container">
                            <div class="form-group">
                              <div class="input-group date">
                                  <input type="text" class="form-control datetimepicker-input o-purchase-datetimepicker" t-attf-id="datetimepicker_#{ol.id}" t-att-name="ol.id"
                                    data-toggle="datetimepicker" data-date-format="YYYY-MM-DD" t-attf-data-target="#datetimepicker_#{ol.id}"/>
                              </div>
                            </div>
                          </div>
                        </form>
                      </td>
                      <td class="text-right">
                        <span t-field="ol.price_subtotal" t-options='{"widget": "monetary", "display_currency": order.currency_id}'/>
                      </td>
                    </tr>
                  </t>
                  <t t-if="ol.display_type">
                    <tr>
                      <td colspan="99">
                        <span t-esc="ol.name"/>
                      </td>
                    </tr>
                  </t>
                </t>
              </tbody>
            </table>
          </div>
          <div class="row">
            <div class="col-sm-7 col-md-5 ml-auto">
              <table class="table table-sm">
                <tbody>
                  <tr>
                    <td>
                      <strong>Untaxed Amount:</strong>
                    </td>
                    <td class="text-right">
                      <span t-field="order.amount_untaxed" t-options='{"widget": "monetary", "display_currency": order.currency_id}'/>
                    </td>
                  </tr>
                  <tr>
                    <td>
                      <strong>Taxes:</strong>
                    </td>
                    <td class="text-right">
                      <span t-field="order.amount_tax" t-options='{"widget": "monetary", "display_currency": order.currency_id}'/>
                    </td>
                  </tr>
                  <tr>
                    <td>
                      <strong>Total:</strong>
                    </td>
                    <td class="text-right">
                      <span t-field="order.amount_total" t-options='{"widget": "monetary", "display_currency": order.currency_id}'/>
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
      </div>
    </xpath>
  </template>

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
                <xpath expr="//field[@name='company_id']" position="attributes">
                    <attribute name="readonly">0</attribute>
                    <attribute name="optional">hide</attribute>
                </xpath>
                <xpath expr="//field[@name='min_qty']" position="attributes">
                    <attribute name="optional">hide</attribute>
                </xpath>
                <xpath expr="//field[@name='name']" position="attributes">
                    <attribute name="readonly">0</attribute>
                </xpath>
                <xpath expr="//field[@name='product_id']" position="attributes">
                    <attribute name="readonly">0</attribute>
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
                    <attribute name="invisible">0</attribute>
                </xpath>
                <group name="purchase" position="before">
                    <field name="seller_ids" context="{'default_product_tmpl_id':context.get('product_tmpl_id',active_id), 'product_template_invisible_variant': True, 'tree_view_ref':'purchase.product_supplierinfo_tree_view2'}" nolabel="1" attrs="{'invisible': [('product_variant_count','&gt;',1)], 'readonly': [('product_variant_count','&gt;',1)]}"/>
                    <field name="variant_seller_ids" context="{'default_product_tmpl_id': context.get('product_tmpl_id', active_id), 'tree_view_ref':'purchase.product_supplierinfo_tree_view2'}" nolabel="1" attrs="{'invisible': [('product_variant_count','&lt;=',1)], 'readonly': [('product_variant_count','&lt;=',1)]}"/>
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
                    <field name="property_account_creditor_price_difference_categ" domain="[('deprecated','=',False)]" groups="account.group_account_readonly"/>
                </field>
            </field>
        </record>

        <record id="view_product_account_purchase_ok_form" model="ir.ui.view">
            <field name="name">product.template.account.purchase.ok.form.inherit</field>
            <field name="model">product.template</field>
            <field name="inherit_id" ref="account.product_template_form_view"/>
            <field name="arch" type="xml">
                <field name="property_account_expense_id" position="attributes">
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
                    <field name="property_account_creditor_price_difference" domain="[('deprecated','=',False)]" attrs="{'readonly':[('purchase_ok', '=', 0)]}" groups="account.group_account_readonly"/>
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

    <template id="track_po_line_qty_received_template">
        <div>
            <strong>The received quantity has been updated.</strong>
            <ul>
                <li><t t-esc="line.product_id.display_name"/>:</li>
                Received Quantity: <t t-esc="line.qty_received" /> -&gt; <t t-esc="float(qty_received)"/><br/>
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
              action="uom.product_uom_form_action" id="menu_purchase_uom_form_action"
              parent="purchase.menu_unit_of_measure_in_config_purchase" sequence="5" groups="uom.group_uom"/>

        <menuitem
             action="uom.product_uom_categ_form_action" id="menu_purchase_uom_categ_form_action"
             parent="purchase.menu_unit_of_measure_in_config_purchase" sequence="10" groups="uom.group_uom"/>

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
                <pivot string="Purchase Order" display_quantity="True" sample="1">
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
                    <button name="action_rfq_send" states="draft" string="Send by Email" type="object" context="{'send_rfq':True}" class="oe_highlight"/>
                    <button name="print_quotation" string="Print RFQ" type="object" states="draft" class="oe_highlight" groups="base.group_user"/>
                    <button name="button_confirm" type="object" states="sent" string="Confirm Order" class="oe_highlight" id="bid_confirm"/>
                    <button name="button_approve" type="object" states='to approve' string="Approve Order" class="oe_highlight" groups="purchase.group_purchase_manager"/>
                    <button name="action_create_invoice" string="Create Bill" type="object" class="oe_highlight" context="{'create_bill':True}" attrs="{'invisible': ['|', ('state', 'not in', ('purchase', 'done')), ('invoice_status', 'in', ('no', 'invoiced'))]}"/>
                    <button name="action_rfq_send" states="sent" string="Re-Send by Email" type="object" context="{'send_rfq':True}"/>
                    <button name="print_quotation" string="Print RFQ" type="object" states="sent" groups="base.group_user"/>
                    <button name="button_confirm" type="object" states="draft" string="Confirm Order" id="draft_confirm"/>
                    <button name="action_rfq_send" states="purchase" string="Send PO by Email" type="object" context="{'send_rfq':False}"/>
                    <button name="confirm_reminder_mail" string="Confirm Receipt Date" type="object" attrs="{'invisible': ['|','|', ('state', 'not in', ('purchase', 'done')), ('mail_reminder_confirmed', '=', True), ('date_planned', '=', False)]}" groups="base.group_no_one"/>
                    <button name="action_create_invoice" string="Create Bill" type="object" context="{'create_bill':True}" attrs="{'invisible': ['|', '|', ('state', 'not in', ('purchase', 'done')), ('invoice_status', 'not in', ('no', 'invoiced')), ('order_line', '=', [])]}"/>
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
                            <field name="priority" widget="priority" class="mr-3"/>
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
                            <field name="id" invisible="1"/>
                        </group>
                        <group>
                            <field name="date_order" attrs="{'invisible': [('state','in',('purchase','done'))]}"/>
                            <label for="date_approve" attrs="{'invisible': [('state','not in',('purchase','done'))]}"/>
                            <div name="date_approve" attrs="{'invisible': [('state','not in',('purchase','done'))]}" class="o_row">
                                <field name="date_approve"/>
                                <field name="mail_reception_confirmed" invisible="1"/>
                                <span class="text-muted" attrs="{'invisible': [('mail_reception_confirmed','=', False)]}">(confirmed by vendor)</span>
                            </div>
                            <label for="date_planned"/>
                            <div name="date_planned_div" class="o_row">
                                <field name="date_planned" attrs="{'readonly': [('state', 'not in', ('draft', 'sent', 'to approve', 'purchase'))]}"/>
                                <field name="mail_reminder_confirmed" invisible="1"/>
                                <span class="text-muted" attrs="{'invisible': [('mail_reminder_confirmed', '=', False)]}">(confirmed by vendor)</span>
                            </div>
                            <label for="receipt_reminder_email" invisible='1'/>
                            <div name="reminder" class="o_row" groups='purchase.group_send_reminder' title="Automatically send a confirmation email to the vendor X days before the expected receipt date, asking him to confirm the exact date.">
                                <field name="receipt_reminder_email"/>
                                <span>Ask confirmation</span>
                                <div class="o_row oe_inline" attrs="{'invisible': [('receipt_reminder_email', '=', False)]}">
                                    <field name="reminder_date_before_receipt" class="oe_inline"/>
                                    day(s) before 
                                    <widget name='toaster_button' button_name="send_reminder_preview" title="Preview the reminder email by sending it to yourself." attrs="{'invisible': [('id', '=', False)]}"/>
                                </div>
                            </div>
                            <field name="origin" attrs="{'invisible': [('origin','=',False)]}"/>
                        </group>
                    </group>
                    <notebook>
                        <page string="Products" name="products">
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
                                    <field name="date_planned" optional="hide" attrs="{'required': [('display_type', '=', False)]}" force_save="1"/>
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
                                            </group>
                                            <group colspan="12">
                                            <notebook>
                                                <page string="Notes" name="notes">
                                                    <field name="name"/>
                                                </page>
                                                <page string="Invoices and Incoming Shipments" name="invoices_incoming_shiptments">
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
                                <group name="other_info">
                                    <field name="user_id" domain="[('share', '=', False)]" widget="many2one_avatar_user"/>
                                    <field name="company_id" groups="base.group_multi_company" options="{'no_create': True}"/>
                                </group>
                                <group name="invoice_info">
                                    <field name="invoice_status" attrs="{'invisible': [('state', 'in', ('draft', 'sent', 'to approve', 'cancel'))]}"/>
                                    <field name="payment_term_id" attrs="{'readonly': ['|', ('invoice_status','=', 'invoiced'), ('state', '=', 'done')]}" options="{'no_create': True}"/>
                                    <field name="fiscal_position_id" options="{'no_create': True}" attrs="{'readonly': ['|', ('invoice_status','=', 'invoiced'), ('state', '=', 'done')]}"/>
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
                <kanban class="o_kanban_mobile" js_class="purchase_kanban_dashboard" sample="1" quick_create="false">
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
                                    <div class="o_kanban_record_headings ml-1">
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
            <field name="priority" eval="1"/>
            <field name="arch" type="xml">
                <tree string="Purchase Order" multi_edit="1"
                      decoration-muted="state=='cancel'" decoration-info="state in ('wait','confirmed')" sample="1">
                    <field name="priority" optional="show" widget="priority" nolabel="1"/>
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
        
        <!-- Unfortunately we want the purchase kpis table to only show up in some list views,
             so we have to duplicate code to support both view versions -->
        <record id="purchase_order_kpis_tree" model="ir.ui.view">
            <field name="name">purchase.order.inherit.purchase.order.tree</field>
            <field name="model">purchase.order</field>
            <field name="priority" eval="10"/>
            <field name="arch" type="xml">
                <tree string="Purchase Order" multi_edit="1" class="o_purchase_order"
                      js_class="purchase_list_dashboard" sample="1">
                    <header>
                        <button name="action_create_invoice" type="object" string="Create Bills"/>
                    </header>
                    <field name="priority" optional="show" widget="priority" nolabel="1"/>
                    <field name="partner_ref" optional="hide"/>
                    <field name="name" string="Reference" readonly="1" decoration-bf="1"/>
                    <field name="date_approve" invisible="context.get('quotation_only', False)" optional="show"/>
                    <field name="partner_id" readonly="1"/>
                    <field name="company_id" readonly="1" options="{'no_create': True}"
                        groups="base.group_multi_company" optional="show"/>
                    <field name="date_planned" invisible="context.get('quotation_only', False)" optional="show"/>
                    <field name="user_id" optional="show" widget="many2one_avatar_user"/>
                    <field name="date_order" attrs="{'invisible': ['|', '|', ('state', '=', 'purchase'), ('state', '=', 'done'), ('state', '=', 'cancel')]}"
                        invisible="not context.get('quotation_only', False)" widget="remaining_days" optional="show"/>
                    <field name="activity_ids" widget="list_activity" optional="show"/>
                    <field name="origin" optional="show"/>
                    <field name="amount_untaxed" sum="Total Untaxed amount" string="Untaxed" widget="monetary" optional="hide"/>
                    <field name="amount_total" sum="Total amount" widget="monetary" optional="show" decoration-bf="1"/>
                    <field name="currency_id" invisible="1"/>
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
                <tree decoration-muted="state=='cancel'"
                    decoration-info="state in ('wait','confirmed')"
                    string="Purchase Order"
                    class="o_purchase_order"
                    sample="1">
                    <header>
                        <button name="action_create_invoice" type="object" string="Create Bills"/>
                    </header>
                    <field name="priority" optional="show" widget="priority" nolabel="1"/>
                    <field name="partner_ref" optional="hide"/>
                    <field name="name" string="Reference" readonly="1" decoration-bf="1"/>
                    <field name="date_approve" widget="date" invisible="context.get('quotation_only', False)" optional="show"/>
                    <field name="partner_id"/>
                    <field name="company_id" groups="base.group_multi_company" options="{'no_create': True}" optional="show"/>
                    <field name="date_planned" invisible="context.get('quotation_only', False)" optional="show"/>
                    <field name="user_id" widget="many2one_avatar_user" optional="show"/>
                    <field name="date_order" invisible="not context.get('quotation_only', False)" optional="show"/>
                    <field name="activity_ids" widget="list_activity" optional="show"/>
                    <field name="origin" optional="show"/>
                    <field name="amount_untaxed" sum="Total Untaxed amount" string="Untaxed" widget="monetary" optional="hide"/>
                    <field name="amount_total" sum="Total amount" widget="monetary" optional="show" decoration-bf="1"/>
                    <field name="currency_id" invisible="1"/>
                    <field name="state" invisible="1"/>
                    <field name="invoice_status" widget="badge" decoration-success="invoice_status == 'invoiced'" decoration-info="invoice_status == 'to invoice'" optional="show"/>
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

        <record id="purchase_rfq" model="ir.actions.act_window">
            <field name="name">Requests for Quotation</field>
            <field name="type">ir.actions.act_window</field>
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
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">purchase.order</field>
            <field name="view_mode">tree,kanban,form,pivot,graph,calendar,activity</field>
            <field name="view_id" ref="purchase_order_view_tree"/>
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
                    <field name="currency_id" invisible="1"/>
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
    
    <!-- Dashboard action buttons -->
    <!-- Dashboard action buttons: End in List view -->
    <record id="purchase_action_dashboard_list" model="ir.actions.act_window">
        <field name="name">Requests for Quotation</field>
        <field name="res_model">purchase.order</field>
        <field name="view_mode">form,tree,kanban,pivot,graph,calendar,activity</field>
        <field name="view_id" ref="purchase.purchase_order_kpis_tree"/>
        <field name="search_view_id" ref="view_purchase_order_filter"/>
        <field name="target">main</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                No RFQs to display
            </p>
        </field>
    </record>

    <!-- Dashboard action buttons: End in Kanban view-->
    <record id="purchase_action_dashboard_kanban" model="ir.actions.act_window">
        <field name="name">Requests for Quotation</field>
        <field name="res_model">purchase.order</field>
        <field name="view_mode">form,tree,kanban,pivot,graph,calendar,activity</field>
        <field name="view_id" ref="purchase.view_purchase_order_kanban"/>
        <field name="search_view_id" ref="view_purchase_order_filter"/>
        <field name="target">main</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                No RFQs to display
            </p>
        </field>
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
            <xpath expr="//div[hasclass('settings')]" position="inside">
                <div class="app_settings_block" data-string="Purchase" string="Purchase" data-key="purchase" groups="purchase.group_purchase_manager">
                    <field name="po_double_validation" invisible="1"/>
                    <field name="company_currency_id" invisible="1"/>
                    <field name="po_lock" invisible="1"/>
                    <h2>Orders</h2>
                    <div class="row mt16 o_settings_container" name="purchase_setting_container">
                        <div class="col-12 col-lg-6 o_setting_box" id="po_order_approval">
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
                        <div class="col-12 col-lg-6 o_setting_box" id="automatic_lock_confirmed_orders">
                            <div class="o_setting_left_pane">
                                <field name="lock_confirmed_po"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="lock_confirmed_po"/>
                                <div class="text-muted">
                                    Automatically lock confirmed orders to prevent editing
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box" id="get_order_warnings">
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
                        <div class="col-12 col-lg-6 o_setting_box"
                            id="manage_purchase_agreements"
                            title="Calls for tenders are used when you want to generate requests for quotations to several vendors for a given set of products. You can configure per product if you directly do a Request for Quotation to one vendor or if you want a Call for Tenders to compare offers from several vendors.">
                            <div class="o_setting_left_pane">
                                <field name="module_purchase_requisition"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="module_purchase_requisition"/>
                                <a href="https://www.odoo.com/documentation/14.0/applications/inventory_and_mrp/purchase/manage_deals/agreements.html" title="Documentation" class="o_doc_link" target="_blank"></a>
                                <div class="text-muted">
                                    Manage your purchase agreements (call for tenders, blanket orders)
                                </div>
                                <div class="content-group" attrs="{'invisible': [('module_purchase_requisition', '=', False)]}">
                                    <div id="use_purchase_requisition"/>
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box" id="auto_receipt_reminder">
                            <div class="o_setting_left_pane">
                                <field name="group_send_reminder"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="group_send_reminder"/>
                                <div class="text-muted">
                                    Automatically remind the receipt date to your vendors
                                </div>
                            </div>
                        </div>
                    </div>
                    <h2>Invoicing</h2>
                    <div class="row mt16 o_settings_container" name="invoicing_settings_container">
                        <div class="col-12 col-lg-6 o_setting_box" id="quantities_billed_vendor" title="This default value is applied to any new product created. This can be changed in the product detail form.">
                            <div class="o_setting_left_pane"/>
                            <div class="o_setting_right_pane">
                                <label for="default_purchase_method"/>
                                <a href="https://www.odoo.com/documentation/14.0/applications/inventory_and_mrp/purchase/manage_deals/control_bills.html" title="Documentation" class="o_doc_link" target="_blank"></a>
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
                        <div class="col-12 col-lg-6 o_setting_box"
                            id="three_way_matching"
                            title="If enabled, activates 3-way matching on vendor bills : the items must be received in order to pay the invoice.">
                            <div class="o_setting_left_pane">
                                <field name="module_account_3way_match" string="3-way matching" widget="upgrade_boolean"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="module_account_3way_match"/>
                                <a href="https://www.odoo.com/documentation/14.0/applications/inventory_and_mrp/purchase/manage_deals/control_bills.html" title="Documentation" class="o_doc_link" target="_blank"></a>
                                <div class="text-muted">
                                    Make sure you only pay bills for which you received the goods you ordered
                                </div>
                            </div>
                        </div>
                    </div>
                    <h2>Products</h2>
                    <div class="row mt16 o_settings_container" name="matrix_setting_container">
                        <div class="col-12 col-lg-6 o_setting_box" id="variant_options">
                            <div class="o_setting_left_pane">
                                <field name="group_product_variant"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="group_product_variant"/>
                                    <a href="https://www.odoo.com/documentation/14.0/applications/sales/sales/products_prices/products/variants.html" title="Documentation" class="o_doc_link" target="_blank"></a>
                                <div class="text-muted">
                                    Purchase variants of a product using attributes (size, color, etc.)
                                </div>
                                <div class="content-group" attrs="{'invisible': [('group_product_variant','=',False)]}">
                                    <div class="mt8">
                                        <button name="%(product.attribute_action)d" icon="fa-arrow-right" type="action" string="Attributes" class="btn-link"/>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box"
                            id="product_matrix"
                            title="If installed, the product variants will be added to purchase orders through a grid entry.">
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
            <field name="arch" type="xml">
                <group name="purchase" position="inside">
                    <label for="receipt_reminder_email" groups='purchase.group_send_reminder'/>
                    <div name="receipt_reminder" class="o_row" groups='purchase.group_send_reminder'>
                        <field name="receipt_reminder_email"/>
                        <div attrs="{'invisible': [('receipt_reminder_email', '=', False)]}">
                            <field name="reminder_date_before_receipt" class="oe_inline"/>
                            <span> day(s) before</span>
                        </div>
                    </div>
                    <field name="property_purchase_currency_id" options="{'no_create': True, 'no_open': True}" groups='base.group_multi_currency'/>
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
                <xpath expr="//span[hasclass('oe_kanban_partner_links')]" position="inside">
                    <span t-if="record.purchase_order_count.value>0" class="badge badge-pill"><i class="fa fa-fw fa-shopping-cart" role="img" aria-label="Shopping cart" title="Shopping cart"/><t t-esc="record.purchase_order_count.value"/></span>
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

