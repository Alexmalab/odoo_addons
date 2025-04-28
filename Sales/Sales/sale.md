# Odoo Module: sale

Category: Sales/Sales

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import controllers
from . import report
from . import wizard
from . import populate

from odoo.api import Environment, SUPERUSER_ID


def _synchronize_cron(cr, registry):
    env = Environment(cr, SUPERUSER_ID, {'active_test': False})
    send_invoice_cron = env.ref('sale.send_invoice_cron', raise_if_not_found=False)
    if send_invoice_cron:
        config = env['ir.config_parameter'].get_param('sale.automatic_invoice', False)
        send_invoice_cron.active = bool(config)

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


{
    'name': 'Sales',
    'version': '1.2',
    'category': 'Sales/Sales',
    'summary': 'Sales internal machinery',
    'description': """
This module contains all the common features of Sales Management and eCommerce.
    """,
    'depends': ['sales_team', 'payment', 'portal', 'utm'],
    'data': [
        'security/sale_security.xml',
        'security/ir.model.access.csv',
        'report/sale_report.xml',
        'report/sale_report_views.xml',
        'report/sale_report_templates.xml',
        'report/invoice_report_templates.xml',
        'report/report_all_channels_sales_views.xml',
        'data/ir_sequence_data.xml',
        'data/mail_data_various.xml',
        'data/mail_template_data.xml',
        'data/mail_templates.xml',
        'data/sale_data.xml',
        'wizard/sale_make_invoice_advance_views.xml',
        'views/sale_views.xml',
        'views/crm_team_views.xml',
        'views/res_partner_views.xml',
        'views/mail_activity_views.xml',
        'views/variant_templates.xml',
        'views/sale_portal_templates.xml',
        'views/sale_onboarding_views.xml',
        'views/res_config_settings_views.xml',
        'views/payment_templates.xml',
        'views/payment_views.xml',
        'views/product_views.xml',
        'views/product_packaging_views.xml',
        'views/utm_campaign_views.xml',
        'wizard/sale_order_cancel_views.xml',
        'wizard/sale_payment_link_views.xml',
    ],
    'demo': [
        'data/product_product_demo.xml',
        'data/sale_demo.xml',
    ],
    'installable': True,
    'auto_install': False,
    'assets': {
        'web.assets_backend': [
            'sale/static/src/scss/sale_onboarding.scss',
            'sale/static/src/scss/product_configurator.scss',
            'sale/static/src/js/sale.js',
            'sale/static/src/js/tours/sale.js',
            'sale/static/src/js/product_configurator_widget.js',
            'sale/static/src/js/sale_order_view.js',
            'sale/static/src/js/product_discount_widget.js',
        ],
        'web.report_assets_common': [
            'sale/static/src/scss/sale_report.scss',
        ],
        'web.assets_frontend': [
            'sale/static/src/scss/sale_portal.scss',
            'sale/static/src/js/sale_portal_sidebar.js',
            'sale/static/src/js/payment_form.js',
        ],
        'web.assets_tests': [
            'sale/static/tests/tours/**/*',
        ],
        'web.qunit_suite_tests': [
            'sale/static/tests/product_configurator_tests.js',
            'sale/static/tests/sales_team_dashboard_tests.js',
        ],
    },
    'post_init_hook': '_synchronize_cron',
    'license': 'LGPL-3',
}

```

## File: controllers\onboarding.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http
from odoo.http import request


class OnboardingController(http.Controller):

    @http.route('/sales/sale_quotation_onboarding_panel', auth='user', type='json')
    def sale_quotation_onboarding(self):
        """ Returns the `banner` for the sale onboarding panel.
            It can be empty if the user has closed it or if he doesn't have
            the permission to see it. """

        company = request.env.company
        if not request.env.is_admin() or \
           company.sale_quotation_onboarding_state == 'closed':
            return {}

        return {
            'html': request.env.ref('sale.sale_quotation_onboarding_panel')._render({
                'company': company,
                'state': company.get_and_update_sale_quotation_onboarding_state()
            })
        }

```

## File: controllers\portal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import binascii

from odoo import fields, http, SUPERUSER_ID, _
from odoo.exceptions import AccessError, MissingError, ValidationError
from odoo.fields import Command
from odoo.http import request

from odoo.addons.payment.controllers import portal as payment_portal
from odoo.addons.payment import utils as payment_utils
from odoo.addons.portal.controllers.mail import _message_post_helper
from odoo.addons.portal.controllers import portal
from odoo.addons.portal.controllers.portal import pager as portal_pager, get_records_pager


class CustomerPortal(portal.CustomerPortal):

    def _prepare_home_portal_values(self, counters):
        values = super()._prepare_home_portal_values(counters)
        partner = request.env.user.partner_id

        SaleOrder = request.env['sale.order']
        if 'quotation_count' in counters:
            values['quotation_count'] = SaleOrder.search_count(self._prepare_quotations_domain(partner)) \
                if SaleOrder.check_access_rights('read', raise_exception=False) else 0
        if 'order_count' in counters:
            values['order_count'] = SaleOrder.search_count(self._prepare_orders_domain(partner)) \
                if SaleOrder.check_access_rights('read', raise_exception=False) else 0

        return values

    def _prepare_quotations_domain(self, partner):
        return [
            ('message_partner_ids', 'child_of', [partner.commercial_partner_id.id]),
            ('state', 'in', ['sent', 'cancel'])
        ]

    def _prepare_orders_domain(self, partner):
        return [
            ('message_partner_ids', 'child_of', [partner.commercial_partner_id.id]),
            ('state', 'in', ['sale', 'done'])
        ]

    #
    # Quotations and Sales Orders
    #

    def _get_sale_searchbar_sortings(self):
        return {
            'date': {'label': _('Order Date'), 'order': 'date_order desc'},
            'name': {'label': _('Reference'), 'order': 'name'},
            'stage': {'label': _('Stage'), 'order': 'state'},
        }

    @http.route(['/my/quotes', '/my/quotes/page/<int:page>'], type='http', auth="user", website=True)
    def portal_my_quotes(self, page=1, date_begin=None, date_end=None, sortby=None, **kw):
        values = self._prepare_portal_layout_values()
        partner = request.env.user.partner_id
        SaleOrder = request.env['sale.order']

        domain = self._prepare_quotations_domain(partner)

        searchbar_sortings = self._get_sale_searchbar_sortings()

        # default sortby order
        if not sortby:
            sortby = 'date'
        sort_order = searchbar_sortings[sortby]['order']

        if date_begin and date_end:
            domain += [('create_date', '>', date_begin), ('create_date', '<=', date_end)]

        # count for pager
        quotation_count = SaleOrder.search_count(domain)
        # make pager
        pager = portal_pager(
            url="/my/quotes",
            url_args={'date_begin': date_begin, 'date_end': date_end, 'sortby': sortby},
            total=quotation_count,
            page=page,
            step=self._items_per_page
        )
        # search the count to display, according to the pager data
        quotations = SaleOrder.search(domain, order=sort_order, limit=self._items_per_page, offset=pager['offset'])
        request.session['my_quotations_history'] = quotations.ids[:100]

        values.update({
            'date': date_begin,
            'quotations': quotations,
            'page_name': 'quote',
            'pager': pager,
            'default_url': '/my/quotes',
            'searchbar_sortings': searchbar_sortings,
            'sortby': sortby,
        })
        return request.render("sale.portal_my_quotations", values)

    @http.route(['/my/orders', '/my/orders/page/<int:page>'], type='http', auth="user", website=True)
    def portal_my_orders(self, page=1, date_begin=None, date_end=None, sortby=None, **kw):
        values = self._prepare_portal_layout_values()
        partner = request.env.user.partner_id
        SaleOrder = request.env['sale.order']

        domain = self._prepare_orders_domain(partner)

        searchbar_sortings = self._get_sale_searchbar_sortings()

        # default sortby order
        if not sortby:
            sortby = 'date'
        sort_order = searchbar_sortings[sortby]['order']

        if date_begin and date_end:
            domain += [('create_date', '>', date_begin), ('create_date', '<=', date_end)]

        # count for pager
        order_count = SaleOrder.search_count(domain)
        # pager
        pager = portal_pager(
            url="/my/orders",
            url_args={'date_begin': date_begin, 'date_end': date_end, 'sortby': sortby},
            total=order_count,
            page=page,
            step=self._items_per_page
        )
        # content according to pager
        orders = SaleOrder.search(domain, order=sort_order, limit=self._items_per_page, offset=pager['offset'])
        request.session['my_orders_history'] = orders.ids[:100]

        values.update({
            'date': date_begin,
            'orders': orders,
            'page_name': 'order',
            'pager': pager,
            'default_url': '/my/orders',
            'searchbar_sortings': searchbar_sortings,
            'sortby': sortby,
        })
        return request.render("sale.portal_my_orders", values)

    @http.route(['/my/orders/<int:order_id>'], type='http', auth="public", website=True)
    def portal_order_page(self, order_id, report_type=None, access_token=None, message=False, download=False, **kw):
        try:
            order_sudo = self._document_check_access('sale.order', order_id, access_token=access_token)
        except (AccessError, MissingError):
            return request.redirect('/my')

        if report_type in ('html', 'pdf', 'text'):
            return self._show_report(model=order_sudo, report_type=report_type, report_ref='sale.action_report_saleorder', download=download)

        # use sudo to allow accessing/viewing orders for public user
        # only if he knows the private token
        # Log only once a day
        if order_sudo:
            # store the date as a string in the session to allow serialization
            now = fields.Date.today().isoformat()
            session_obj_date = request.session.get('view_quote_%s' % order_sudo.id)
            if session_obj_date != now and request.env.user.share and access_token:
                request.session['view_quote_%s' % order_sudo.id] = now
                body = _('Quotation viewed by customer %s', order_sudo.partner_id.name if request.env.user._is_public() else request.env.user.partner_id.name)
                _message_post_helper(
                    "sale.order",
                    order_sudo.id,
                    body,
                    token=order_sudo.access_token,
                    message_type="notification",
                    subtype_xmlid="mail.mt_note",
                    partner_ids=order_sudo.user_id.sudo().partner_id.ids,
                )

        values = {
            'sale_order': order_sudo,
            'message': message,
            'token': access_token,
            'landing_route': '/shop/payment/validate',
            'bootstrap_formatting': True,
            'partner_id': order_sudo.partner_id.id,
            'report_type': 'html',
            'action': order_sudo._get_portal_return_action(),
        }
        if order_sudo.company_id:
            values['res_company'] = order_sudo.company_id

        # Payment values
        if order_sudo.has_to_be_paid():
            logged_in = not request.env.user._is_public()

            acquirers_sudo = request.env['payment.acquirer'].sudo()._get_compatible_acquirers(
                order_sudo.company_id.id,
                order_sudo.partner_id.id,
                currency_id=order_sudo.currency_id.id,
                sale_order_id=order_sudo.id,
            )  # In sudo mode to read the fields of acquirers and partner (if not logged in)
            tokens = request.env['payment.token'].search([
                ('acquirer_id', 'in', acquirers_sudo.ids),
                ('partner_id', '=', order_sudo.partner_id.id)
            ]) if logged_in else request.env['payment.token']

            # Make sure that the partner's company matches the order's company.
            if not payment_portal.PaymentPortal._can_partner_pay_in_company(
                order_sudo.partner_id, order_sudo.company_id
            ):
                acquirers_sudo = request.env['payment.acquirer'].sudo()
                tokens = request.env['payment.token']

            fees_by_acquirer = {
                acquirer: acquirer._compute_fees(
                    order_sudo.amount_total,
                    order_sudo.currency_id,
                    order_sudo.partner_id.country_id,
                ) for acquirer in acquirers_sudo.filtered('fees_active')
            }
            # Prevent public partner from saving payment methods but force it for logged in partners
            # buying subscription products
            show_tokenize_input = logged_in \
                and not request.env['payment.acquirer'].sudo()._is_tokenization_required(
                    sale_order_id=order_sudo.id
                )
            values.update({
                'acquirers': acquirers_sudo,
                'tokens': tokens,
                'fees_by_acquirer': fees_by_acquirer,
                'show_tokenize_input': show_tokenize_input,
                'amount': order_sudo.amount_total,
                'currency': order_sudo.pricelist_id.currency_id,
                'partner_id': order_sudo.partner_id.id,
                'access_token': order_sudo.access_token,
                'transaction_route': order_sudo.get_portal_url(suffix='/transaction'),
                'landing_route': order_sudo.get_portal_url(),
            })

        if order_sudo.state in ('draft', 'sent', 'cancel'):
            history = request.session.get('my_quotations_history', [])
        else:
            history = request.session.get('my_orders_history', [])
        values.update(get_records_pager(history, order_sudo))

        return request.render('sale.sale_order_portal_template', values)

    @http.route(['/my/orders/<int:order_id>/accept'], type='json', auth="public", website=True)
    def portal_quote_accept(self, order_id, access_token=None, name=None, signature=None):
        # get from query string if not on json param
        access_token = access_token or request.httprequest.args.get('access_token')
        try:
            order_sudo = self._document_check_access('sale.order', order_id, access_token=access_token)
        except (AccessError, MissingError):
            return {'error': _('Invalid order.')}

        if not order_sudo.has_to_be_signed():
            return {'error': _('The order is not in a state requiring customer signature.')}
        if not signature:
            return {'error': _('Signature is missing.')}

        try:
            order_sudo.write({
                'signed_by': name,
                'signed_on': fields.Datetime.now(),
                'signature': signature,
            })
            request.env.cr.commit()
        except (TypeError, binascii.Error) as e:
            return {'error': _('Invalid signature data.')}

        if not order_sudo.has_to_be_paid():
            order_sudo.action_confirm()
            order_sudo._send_order_confirmation_mail()

        pdf = request.env.ref('sale.action_report_saleorder').with_user(SUPERUSER_ID)._render_qweb_pdf([order_sudo.id])[0]

        _message_post_helper(
            'sale.order', order_sudo.id, _('Order signed by %s') % (name,),
            attachments=[('%s.pdf' % order_sudo.name, pdf)],
            **({'token': access_token} if access_token else {}))

        query_string = '&message=sign_ok'
        if order_sudo.has_to_be_paid(True):
            query_string += '#allow_payment=yes'
        return {
            'force_refresh': True,
            'redirect_url': order_sudo.get_portal_url(query_string=query_string),
        }

    @http.route(['/my/orders/<int:order_id>/decline'], type='http', auth="public", methods=['POST'], website=True)
    def decline(self, order_id, access_token=None, **post):
        try:
            order_sudo = self._document_check_access('sale.order', order_id, access_token=access_token)
        except (AccessError, MissingError):
            return request.redirect('/my')

        message = post.get('decline_message')

        query_string = False
        if order_sudo.has_to_be_signed() and message:
            order_sudo.action_cancel()
            _message_post_helper('sale.order', order_id, message, **{'token': access_token} if access_token else {})
        else:
            query_string = "&message=cant_reject"

        return request.redirect(order_sudo.get_portal_url(query_string=query_string))


class PaymentPortal(payment_portal.PaymentPortal):

    @http.route('/my/orders/<int:order_id>/transaction', type='json', auth='public')
    def portal_order_transaction(self, order_id, access_token, **kwargs):
        """ Create a draft transaction and return its processing values.

        :param int order_id: The sales order to pay, as a `sale.order` id
        :param str access_token: The access token used to authenticate the request
        :param dict kwargs: Locally unused data passed to `_create_transaction`
        :return: The mandatory values for the processing of the transaction
        :rtype: dict
        :raise: ValidationError if the invoice id or the access token is invalid
        """
        # Check the order id and the access token
        try:
            order_sudo = self._document_check_access('sale.order', order_id, access_token)
        except MissingError as error:
            raise error
        except AccessError:
            raise ValidationError("The access token is invalid.")

        kwargs.update({
            'reference_prefix': None,  # Allow the reference to be computed based on the order
            'partner_id': order_sudo.partner_invoice_id.id,
            'sale_order_id': order_id,  # Include the SO to allow Subscriptions tokenizing the tx
        })
        kwargs.pop('custom_create_values', None)  # Don't allow passing arbitrary create values
        tx_sudo = self._create_transaction(
            custom_create_values={'sale_order_ids': [Command.set([order_id])]}, **kwargs,
        )

        return tx_sudo._get_processing_values()

    # Payment overrides

    @http.route()
    def payment_pay(self, *args, amount=None, sale_order_id=None, access_token=None, **kwargs):
        """ Override of payment to replace the missing transaction values by that of the sale order.

        This is necessary for the reconciliation as all transaction values, excepted the amount,
        need to match exactly that of the sale order.

        :param str amount: The (possibly partial) amount to pay used to check the access token
        :param str sale_order_id: The sale order for which a payment id made, as a `sale.order` id
        :param str access_token: The access token used to authenticate the partner
        :return: The result of the parent method
        :rtype: str
        :raise: ValidationError if the order id is invalid
        """
        # Cast numeric parameters as int or float and void them if their str value is malformed
        amount = self._cast_as_float(amount)
        sale_order_id = self._cast_as_int(sale_order_id)
        if sale_order_id:
            order_sudo = request.env['sale.order'].sudo().browse(sale_order_id).exists()
            if not order_sudo:
                raise ValidationError(_("The provided parameters are invalid."))

            # Check the access token against the order values. Done after fetching the order as we
            # need the order fields to check the access token.
            if not payment_utils.check_access_token(
                access_token, order_sudo.partner_invoice_id.id, amount, order_sudo.currency_id.id
            ):
                raise ValidationError(_("The provided parameters are invalid."))

            kwargs.update({
                'currency_id': order_sudo.currency_id.id,
                'partner_id': order_sudo.partner_invoice_id.id,
                'company_id': order_sudo.company_id.id,
                'sale_order_id': sale_order_id,
            })
        return super().payment_pay(*args, amount=amount, access_token=access_token, **kwargs)

    def _get_custom_rendering_context_values(self, sale_order_id=None, **kwargs):
        """ Override of payment to add the sale order id in the custom rendering context values.

        :param int sale_order_id: The sale order for which a payment id made, as a `sale.order` id
        :return: The extended rendering context values
        :rtype: dict
        """
        rendering_context_values = super()._get_custom_rendering_context_values(**kwargs)
        if sale_order_id:
            rendering_context_values['sale_order_id'] = sale_order_id
        return rendering_context_values

    def _create_transaction(self, *args, sale_order_id=None, custom_create_values=None, **kwargs):
        """ Override of payment to add the sale order id in the custom create values.

        :param int sale_order_id: The sale order for which a payment id made, as a `sale.order` id
        :param dict custom_create_values: Additional create values overwriting the default ones
        :return: The result of the parent method
        :rtype: recordset of `payment.transaction`
        """
        if sale_order_id:
            if custom_create_values is None:
                custom_create_values = {}
            # As this override is also called if the flow is initiated from sale or website_sale, we
            # need not to override whatever value these modules could have already set
            if 'sale_order_ids' not in custom_create_values:  # We are in the payment module's flow
                custom_create_values['sale_order_ids'] = [Command.set([int(sale_order_id)])]

        return super()._create_transaction(
            *args, sale_order_id=sale_order_id, custom_create_values=custom_create_values, **kwargs
        )

```

## File: controllers\variant.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http
from odoo.http import request


class VariantController(http.Controller):
    @http.route(['/sale/get_combination_info'], type='json', auth="user", methods=['POST'])
    def get_combination_info(self, product_template_id, product_id, combination, add_qty, pricelist_id, **kw):
        combination = request.env['product.template.attribute.value'].browse(combination)
        pricelist = self._get_pricelist(pricelist_id)
        ProductTemplate = request.env['product.template']
        if 'context' in kw:
            ProductTemplate = ProductTemplate.with_context(**kw.get('context'))
        product_template = ProductTemplate.browse(int(product_template_id))
        res = product_template._get_combination_info(combination, int(product_id or 0), int(add_qty or 1), pricelist)
        if 'parent_combination' in kw:
            parent_combination = request.env['product.template.attribute.value'].browse(kw.get('parent_combination'))
            if not combination.exists() and product_id:
                product = request.env['product.product'].browse(int(product_id))
                if product.exists():
                    combination = product.product_template_attribute_value_ids
            res.update({
                'is_combination_possible': product_template._is_combination_possible(combination=combination, parent_combination=parent_combination),
                'parent_exclusions': product_template._get_parent_attribute_exclusions(parent_combination=parent_combination)
            })
        return res

    @http.route(['/sale/create_product_variant'], type='json', auth="user", methods=['POST'])
    def create_product_variant(self, product_template_id, product_template_attribute_value_ids, **kwargs):
        return request.env['product.template'].browse(int(product_template_id)).create_product_variant(product_template_attribute_value_ids)

    def _get_pricelist(self, pricelist_id, pricelist_fallback=False):
        return request.env['product.pricelist'].browse(int(pricelist_id or 0))

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import onboarding
from . import portal
from . import variant

```

## File: data\ir_sequence_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <!-- Sequences for sale.order -->
        <record id="seq_sale_order" model="ir.sequence">
            <field name="name">Sales Order</field>
            <field name="code">sale.order</field>
            <field name="prefix">S</field>
            <field name="padding">5</field>
            <field name="company_id" eval="False"/>
        </record>

    </data>
</odoo>

```

## File: data\mail_data_various.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- Activities -->
        <record id="mail_act_sale_upsell" model="mail.activity.type">
            <field name="name">Order Upsell</field>
            <field name="icon">fa-line-chart</field>
            <field name="res_model">sale.order</field>
        </record>

        <!-- Sale-related subtypes for messaging / Chatter -->
        <record id="mt_order_sent" model="mail.message.subtype">
            <field name="name">Quotation sent</field>
            <field name="res_model">sale.order</field>
            <field name="default" eval="False"/>
            <field name="description">Quotation sent</field>
        </record>
        <record id="mt_order_confirmed" model="mail.message.subtype">
            <field name="name">Sales Order Confirmed</field>
            <field name="res_model">sale.order</field>
            <field name="default" eval="False"/>
            <field name="description">Quotation confirmed</field>
        </record>

        <!-- Salesteam-related subtypes for messaging / Chatter -->
        <record id="mt_salesteam_order_sent" model="mail.message.subtype">
            <field name="name">Quotation sent</field>
            <field name="sequence">20</field>
            <field name="res_model">crm.team</field>
            <field name="default" eval="True"/>
            <field name="parent_id" ref="sale.mt_order_sent"/>
            <field name="relation_field">team_id</field>
        </record>
        <record id="mt_salesteam_order_confirmed" model="mail.message.subtype">
            <field name="name">Sales Order Confirmed</field>
            <field name="sequence">21</field>
            <field name="res_model">crm.team</field>
            <field name="default" eval="True"/>
            <field name="parent_id" ref="sale.mt_order_confirmed"/>
            <field name="relation_field">team_id</field>
        </record>
        <record id="mt_salesteam_invoice_created" model="mail.message.subtype">
            <field name="name">Invoice Created</field>
            <field name="sequence">22</field>
            <field name="res_model">crm.team</field>
            <field name="parent_id" ref="account.mt_invoice_created"/>
            <field name="relation_field">team_id</field>
        </record>
        <record id="mt_salesteam_invoice_confirmed" model="mail.message.subtype">
            <field name="name">Invoice Confirmed</field>
            <field name="sequence">23</field>
            <field name="res_model">crm.team</field>
            <field name="parent_id" ref="account.mt_invoice_validated"/>
            <field name="relation_field">team_id</field>
        </record>
    </data>
</odoo>

```

## File: data\mail_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">
    <template id="mail_notification_paynow_online" inherit_id="mail.mail_notification_paynow" name="Quotation: Sign and Pay mail notification template">
        <xpath expr="//t[@t-set='access_name']" position="after">
            <t t-if="record._name == 'sale.order'">
                <t t-set="is_transaction_pending" t-value="record.get_portal_last_transaction().state == 'pending'"/>
                <t t-if="record.has_to_be_signed(include_draft=True)">
                    <t t-if="record.has_to_be_paid()" t-set="access_name">
                        <t t-if="is_transaction_pending">View Quotation</t>
                        <t t-else="">Review, Sign &amp; Pay Quotation</t>
                    </t>
                    <t t-else="" t-set="access_name">Review, Accept &amp; Sign Quotation</t>
                </t>
                <t t-elif="record.has_to_be_paid(include_draft=True) and not is_transaction_pending">
                    <t t-set="access_name">Review, Accept &amp; Pay Quotation</t>
                </t>
                <t t-elif="record.state in ('draft', 'sent')">
                    <t t-set="access_name">View Quotation</t>
                </t>
            </t>
        </xpath>
    </template>
</data></odoo>

```

## File: data\mail_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="email_template_edi_sale" model="mail.template">
            <field name="name">Sales Order: Send by email</field>
            <field name="model_id" ref="sale.model_sale_order"/>
            <field name="subject">{{ object.company_id.name }} {{ object.state in ('draft', 'sent') and (ctx.get('proforma') and 'Proforma' or 'Quotation') or 'Order' }} (Ref {{ object.name or 'n/a' }})</field>
            <field name="email_from">{{ (object.user_id.email_formatted or user.email_formatted) }}</field>
            <field name="partner_to">{{ object.partner_id.id }}</field>
            <field name="body_html" type="html">
<div style="margin: 0px; padding: 0px;">
    <p style="margin: 0px; padding: 0px; font-size: 13px;">
        <t t-set="doc_name" t-value="'quotation' if object.state in ('draft', 'sent') else 'order'"/>
        Hello,
        <br/><br/>
        Your
        <t t-if="ctx.get('proforma')">
            Pro forma invoice for <t t-out="doc_name or ''">quotation</t> <strong t-out="object.name or ''">S00052</strong>
            <t t-if="object.origin">
                (with reference: <t t-out="object.origin or ''"></t> )
            </t>
            amounting in <strong t-out="format_amount(object.amount_total, object.pricelist_id.currency_id) or ''">$ 10.00</strong> is available.
        </t>
        <t t-else="">
            <t t-out="doc_name or ''">quotation</t> <strong t-out="object.name or ''"></strong>
            <t t-if="object.origin">
                (with reference: <t t-out="object.origin or ''">S00052</t> )
            </t>
            amounting in <strong t-out="format_amount(object.amount_total, object.pricelist_id.currency_id) or ''">$ 10.00</strong> is ready for review.
        </t>
        <br/><br/>
        Do not hesitate to contact us if you have any questions.
        <br/>
    </p>
</div>
            </field>
            <field name="report_template" ref="action_report_saleorder"/>
            <field name="report_name">{{ (object.name or '').replace('/','_') }}</field>
            <field name="lang">{{ object.partner_id.lang }}</field>
            <field name="auto_delete" eval="True"/>
        </record>

        <record id="mail_template_sale_confirmation" model="mail.template">
            <field name="name">Sales Order: Confirmation Email</field>
            <field name="model_id" ref="sale.model_sale_order"/>
            <field name="subject">{{ object.company_id.name }} {{ (object.get_portal_last_transaction().state == 'pending') and 'Pending Order' or 'Order' }} (Ref {{ object.name or 'n/a' }})</field>
            <field name="email_from">{{ (object.user_id.email_formatted or user.email_formatted) }}</field>
            <field name="partner_to">{{ object.partner_id.id }}</field>
            <field name="body_html" type="html">
<div style="margin: 0px; padding: 0px;">
    <p style="margin: 0px; padding: 0px; font-size: 12px;">
        Hello,
        <br/><br/>
        <t t-set="transaction" t-value="object.get_portal_last_transaction()"/>
        Your order <strong t-out="object.name or ''">S00049</strong> amounting in <strong t-out="format_amount(object.amount_total, object.currency_id) or ''">$ 10.00</strong>
        <t t-if="object.state == 'sale' or (transaction and transaction.state in ('done', 'authorized'))">
            has been confirmed.<br/>
            Thank you for your trust!
        </t>
        <t t-elif="transaction and transaction.state == 'pending'">
            is pending. It will be confirmed when the payment is received.
            <t t-if="object.reference">
                Your payment reference is <strong t-out="object.reference or ''"></strong>.
            </t>
        </t>
        <br/><br/>
        Do not hesitate to contact us if you have any questions.
        <br/><br/>
    </p>
<t t-if="hasattr(object, 'website_id') and object.website_id">
    <div style="margin: 0px; padding: 0px;">
        <table width="100%" style="color: #454748; font-size: 12px; border-collapse: collapse;">
            <tr style="border-bottom: 2px solid #dee2e6;">
                <td style="width: 150px;"><strong>Products</strong></td>
                <td></td>
                <td width="15%" align="center"><strong>Quantity</strong></td>
                <td width="20%" align="right"><strong>
                <t t-if="object.user_id.has_group('account.group_show_line_subtotals_tax_excluded')">
                    VAT Excl.
                </t>
                <t t-else="">
                    VAT Incl.
                </t>
                </strong></td>
            </tr>
        </table>
        <t t-foreach="object.order_line" t-as="line">
            <t t-if="(not hasattr(line, 'is_delivery') or not line.is_delivery) and line.display_type in ['line_section', 'line_note']">
                <table width="100%" style="color: #454748; font-size: 12px; border-collapse: collapse;">
                    <t t-set="loop_cycle_number" t-value="0" />
                    <tr t-att-style="'background-color: #f2f2f2' if loop_cycle_number % 2 == 0 else 'background-color: #ffffff'">
                        <t t-set="loop_cycle_number" t-value="loop_cycle_number + 1" />
                        <td colspan="4">
                            <t t-if="line.display_type == 'line_section'">
                                <strong t-out="line.name or ''">Taking care of Trees Course</strong>
                            </t>
                            <t t-elif="line.display_type == 'line_note'">
                                <i t-out="line.name or ''">Taking care of Trees Course</i>
                            </t>
                        </td>
                    </tr>
                </table>
            </t>
            <t t-elif="(not hasattr(line, 'is_delivery') or not line.is_delivery)">
                <table width="100%" style="color: #454748; font-size: 12px; border-collapse: collapse;">
                    <t t-set="loop_cycle_number" t-value="0" />
                    <tr t-att-style="'background-color: #f2f2f2' if loop_cycle_number % 2 == 0 else 'background-color: #ffffff'">
                        <t t-set="loop_cycle_number" t-value="loop_cycle_number + 1" />
                        <td style="width: 150px;">
                            <img t-attf-src="/web/image/product.product/{{ line.product_id.id }}/image_128" style="width: 64px; height: 64px; object-fit: contain;" alt="Product image"></img>
                        </td>
                        <td align="left" t-out="line.product_id.name or ''">	Taking care of Trees Course</td>
                        <td width="15%" align="center" t-out="line.product_uom_qty or ''">1</td>
                        <td width="20%" align="right"><strong>
                        <t t-if="object.user_id.has_group('account.group_show_line_subtotals_tax_excluded')">
                            <t t-out="format_amount(line.price_reduce_taxexcl, object.currency_id) or ''">$ 10.00</t>
                        </t>
                        <t t-else="">
                            <t t-out="format_amount(line.price_reduce_taxinc, object.currency_id) or ''">$ 10.00</t>
                        </t>
                        </strong></td>
                    </tr>
                </table>
            </t>
        </t>
    </div>
    <div style="margin: 0px; padding: 0px;" t-if="hasattr(object, 'carrier_id') and object.carrier_id">
        <table width="100%" style="color: #454748; font-size: 12px; border-spacing: 0px 4px;" align="right">
            <tr>
                <td style="width: 60%"/>
                <td style="width: 30%; border-top: 1px solid #dee2e6;" align="right"><strong>Delivery:</strong></td>
                <td style="width: 10%; border-top: 1px solid #dee2e6;" align="right" t-out="format_amount(object.amount_delivery, object.currency_id) or ''">$ 0.00</td>
            </tr>
            <tr>
                <td style="width: 60%"/>
                <td style="width: 30%;" align="right"><strong>SubTotal:</strong></td>
                <td style="width: 10%;" align="right" t-out="format_amount(object.amount_untaxed, object.currency_id) or ''">$ 10.00</td>
            </tr>
        </table>
    </div>
    <div style="margin: 0px; padding: 0px;" t-else="">
        <table width="100%" style="color: #454748; font-size: 12px; border-spacing: 0px 4px;" align="right">
            <tr>
                <td style="width: 60%"/>
                <td style="width: 30%; border-top: 1px solid #dee2e6;" align="right"><strong>SubTotal:</strong></td>
                <td style="width: 10%; border-top: 1px solid #dee2e6;" align="right" t-out="format_amount(object.amount_untaxed, object.currency_id) or ''">$ 10.00</td>
            </tr>
        </table>
    </div>
    <div style="margin: 0px; padding: 0px;">
        <table width="100%" style="color: #454748; font-size: 12px; border-spacing: 0px 4px;" align="right">
            <tr>
                <td style="width: 60%"/>
                <td style="width: 30%;" align="right"><strong>Taxes:</strong></td>
                <td style="width: 10%;" align="right" t-out="format_amount(object.amount_tax, object.currency_id) or ''">$ 0.00</td>
            </tr>
            <tr>
                <td style="width: 60%"/>
                <td style="width: 30%; border-top: 1px solid #dee2e6;" align="right"><strong>Total:</strong></td>
                <td style="width: 10%; border-top: 1px solid #dee2e6;" align="right" t-out="format_amount(object.amount_total, object.currency_id) or ''">$ 10.00</td>
            </tr>
        </table>
    </div>
    <div t-if="object.partner_invoice_id" style="margin: 0px; padding: 0px;">
        <table width="100%" style="color: #454748; font-size: 12px;">
            <tr>
                <td style="padding-top: 10px;">
                    <strong>Bill to:</strong>
                    <t t-out="object.partner_invoice_id.street or ''">1201 S Figueroa St</t>
                    <t t-out="object.partner_invoice_id.city or ''">Los Angeles</t>
                    <t t-out="object.partner_invoice_id.state_id.name or ''">California</t>
                    <t t-out="object.partner_invoice_id.zip or ''">90015</t>
                    <t t-out="object.partner_invoice_id.country_id.name or ''">United States</t>
                </td>
            </tr>
            <tr>
                <td>
                    <strong>Payment Method:</strong>
                    <t t-if="transaction.token_id">
                        <t t-out="transaction.token_id.name or ''"></t>
                    </t>
                    <t t-else="">
                        <t t-out="transaction.acquirer_id.sudo().name or ''"></t>
                    </t>
                    (<t t-out="format_amount(transaction.amount, object.currency_id) or ''">$ 10.00</t>)
                </td>
            </tr>
        </table>
    </div>
    <div t-if="object.partner_shipping_id and not object.only_services" style="margin: 0px; padding: 0px;">
        <table width="100%" style="color: #454748; font-size: 12px;">
            <tr>
                <td>
                    <br/>
                    <strong>Ship to:</strong>
                    <t t-out="object.partner_shipping_id.street or ''">1201 S Figueroa St</t>
                    <t t-out="object.partner_shipping_id.city or ''">Los Angeles</t>
                    <t t-out="object.partner_shipping_id.state_id.name or ''">California</t>
                    <t t-out="object.partner_shipping_id.zip or ''">90015</t>
                    <t t-out="object.partner_shipping_id.country_id.name or ''">United States</t>
                </td>
            </tr>
        </table>
        <table t-if="hasattr(object, 'carrier_id') and object.carrier_id" width="100%" style="color: #454748; font-size: 12px;">
            <tr>
                <td>
                    <strong>Shipping Method:</strong>
                    <t t-out="object.carrier_id.name or ''"></t>
                    <t t-if="object.carrier_id.fixed_price == 0.0">
                        (Free)
                    </t>
                    <t t-else="">
                        (<t t-out="format_amount(object.carrier_id.fixed_price, object.currency_id) or ''">$ 10.00</t>)
                    </t>
                </td>
            </tr>
        </table>
    </div>
</t>
</div></field>
            <field name="report_template" ref="action_report_saleorder"/>
            <field name="report_name">{{ (object.name or '').replace('/','_') }}</field>
            <field name="lang">{{ object.partner_id.lang }}</field>
            <field name="auto_delete" eval="True"/>
        </record>
    </data>
</odoo>

```

## File: data\product_product_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="product.consu_delivery_01" model="product.product">
            <field name="invoice_policy">order</field>
        </record>

        <record id="product.consu_delivery_02" model="product.product">
            <field name="invoice_policy">delivery</field>
        </record>

        <record id="product.consu_delivery_03" model="product.product">
            <field name="invoice_policy">delivery</field>
        </record>

        <record id="product.product_delivery_01" model="product.product">
            <field name="invoice_policy">delivery</field>
        </record>

        <record id="product.product_delivery_02" model="product.product">
            <field name="invoice_policy">delivery</field>
        </record>

        <record id="product.product_order_01" model="product.product">
            <field name="invoice_policy">order</field>
        </record>

        <record id="product.product_delivery_02" model="product.product">
            <field name="invoice_policy">delivery</field>
        </record>

        <record id="product.product_delivery_01" model="product.product">
            <field name="invoice_policy">delivery</field>
        </record>

        <record id="product.product_delivery_02" model="product.product">
            <field name="invoice_policy">delivery</field>
        </record>

        <record id="product.product_product_27" model="product.product">
            <field name="invoice_policy">delivery</field>
        </record>

        <record id="product.product_product_25" model="product.product">
            <field name="invoice_policy">delivery</field>
        </record>

        <record id="product.product_product_24" model="product.product">
            <field name="invoice_policy">order</field>
        </record>

        <record id="product.product_product_22" model="product.product">
            <field name="invoice_policy">delivery</field>
        </record>

        <record id="product.product_product_20" model="product.product">
            <field name="invoice_policy">delivery</field>
        </record>

        <record id="product.product_product_16" model="product.product">
            <field name="invoice_policy">order</field>
        </record>

        <record id="product.product_product_13" model="product.product">
            <field name="invoice_policy">delivery</field>
        </record>

        <record id="product.product_product_12" model="product.product">
            <field name="invoice_policy">delivery</field>
        </record>

        <record id="product.product_product_11b" model="product.product">
            <field name="invoice_policy">delivery</field>
        </record>

        <record id="product.product_product_11" model="product.product">
            <field name="invoice_policy">delivery</field>
        </record>

        <record id="product.product_product_10" model="product.product">
            <field name="invoice_policy">delivery</field>
        </record>

        <record id="product.product_product_9" model="product.product">
            <field name="invoice_policy">delivery</field>
        </record>

        <record id="product.product_product_8" model="product.product">
            <field name="invoice_policy">delivery</field>
        </record>

        <record id="product.product_product_7" model="product.product">
            <field name="invoice_policy">delivery</field>
        </record>

        <record id="product.product_product_6" model="product.product">
            <field name="invoice_policy">delivery</field>
        </record>

        <record id="product.product_product_5" model="product.product">
            <field name="invoice_policy">delivery</field>
        </record>

        <record id="product.product_product_4c" model="product.product">
            <field name="invoice_policy">delivery</field>
        </record>

        <record id="product.product_product_4b" model="product.product">
            <field name="invoice_policy">delivery</field>
        </record>

        <record id="product.product_product_4" model="product.product">
            <field name="invoice_policy">delivery</field>
        </record>

        <record id="product.product_product_3" model="product.product">
            <field name="invoice_policy">delivery</field>
            <field name="expense_policy">cost</field>
        </record>

        <record id="product.product_product_2" model="product.product">
            <field name="invoice_policy">delivery</field>
        </record>

        <record id="product.product_product_1" model="product.product">
            <field name="invoice_policy">delivery</field>
        </record>

        <!-- Expensable products -->
        <record id="product.expense_product" model="product.product">
            <field name="invoice_policy">order</field>
            <field name="expense_policy">sales_price</field>
        </record>

        <record id="product.expense_hotel" model="product.product">
            <field name="invoice_policy">delivery</field>
            <field name="expense_policy">cost</field>
        </record>

    </data>
</odoo>

```

## File: data\sale_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- Share Button in action menu -->
        <record id="model_sale_order_action_share" model="ir.actions.server">
            <field name="name">Share</field>
            <field name="model_id" ref="sale.model_sale_order"/>
            <field name="binding_model_id" ref="sale.model_sale_order"/>
            <field name="binding_view_types">form</field>
            <field name="state">code</field>
            <field name="code">action = records.action_share()</field>
        </record>

        <!-- set default order confirmation template -->
        <record id="default_confirmation_template" model="ir.config_parameter">
            <field name="key">sale.default_confirmation_template</field>
            <field name="value" ref="sale.mail_template_sale_confirmation"/>
        </record>

        <record model="ir.cron" id="send_invoice_cron">
            <field name="name">automatic invoicing: send ready invoice</field>
            <field name="model_id" ref="payment.model_payment_transaction" />
            <field name="state">code</field>
            <field name="code">model._cron_send_invoice()</field>
            <field name="user_id" ref="base.user_root" />
            <field name="interval_number">1</field>
            <field name="interval_type">days</field>
            <field name="numbercall">-1</field>
        </record>

    </data>
</odoo>

```

## File: data\sale_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <!-- Enable EUR currency since it's the currency on the company, pricelist and Sale Orders

            If the currency is not enabled, you cannot pay the demo SO's with a payment link bc the currency
            is disabled.
        -->
        <function model="res.currency" name="action_unarchive">
            <value model="res.currency" search="[('id', '=', obj().env.ref('product.list0').currency_id.id), ('active', '=', False)]"/>
        </function>

        <!-- We want to activate pay and sign by default for easier demoing. -->
        <record id="base.main_company" model="res.company">
            <field name="portal_confirmation_pay" eval="True"/>
        </record>

        <record id="base.user_demo" model="res.users">
            <field eval="[(4, ref('sales_team.group_sale_salesman'))]" name="groups_id"/>
        </record>

        <record model="crm.team" id="sales_team.team_sales_department">
            <field name="use_quotations" eval="True"/>
            <field name="invoiced_target">250000</field>
        </record>

        <record model="crm.team" id="sales_team.crm_team_1">
            <field name="use_quotations" eval="True"/>
            <field name="invoiced_target">40000</field>
        </record>

        <record id="utm_source_sale_order_0" model="utm.source">
            <field name="name">Sale Promotion 1</field>
        </record>

        <record id="sale_order_1" model="sale.order">
            <field name="partner_id" ref="base.res_partner_2"/>
            <field name="partner_invoice_id" ref="base.res_partner_2"/>
            <field name="partner_shipping_id" ref="base.res_partner_2"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="pricelist_id" ref="product.list0"/>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="sale.utm_source_sale_order_0"/>
            <field name="date_order" eval="(DateTime.today() - relativedelta(months=1)).strftime('%Y-%m-%d %H:%M')"/>
        </record>

        <record id="sale_order_line_1" model="sale.order.line">
            <field name="order_id" ref="sale_order_1"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_25').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_25"/>
            <field name="product_uom_qty">3</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">295.00</field>
        </record>

        <record id="sale_order_line_2" model="sale.order.line">
            <field name="order_id" ref="sale_order_1"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_delivery_02').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_delivery_02"/>
            <field name="product_uom_qty">5</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">145.00</field>
        </record>

        <record id="sale_order_line_3" model="sale.order.line">
            <field name="order_id" ref="sale_order_1"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_delivery_01').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_delivery_01"/>
            <field name="product_uom_qty">2</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">65.00</field>
        </record>

        <record id="sale_order_2" model="sale.order">
            <field name="partner_id" ref="base.res_partner_4"/>
            <field name="partner_invoice_id" ref="base.res_partner_address_13"/>
            <field name="partner_shipping_id" ref="base.res_partner_address_13"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="pricelist_id" ref="product.list0"/>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="sale.utm_source_sale_order_0"/>
            <field name="date_order" eval="(DateTime.today() - relativedelta(months=1)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="tag_ids" eval="[(4, ref('sales_team.categ_oppor7'))]"/>
        </record>

        <record id="sale_order_line_4" model="sale.order.line">
            <field name="order_id" ref="sale_order_2"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_1').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_1"/>
            <field name="product_uom_qty">24</field>
            <field name="product_uom" ref="uom.product_uom_hour"/>
            <field name="price_unit">75.00</field>
        </record>

        <record id="sale_order_line_5" model="sale.order.line">
            <field name="order_id" ref="sale_order_2"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_2').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_2"/>
            <field name="product_uom_qty">30</field>
            <field name="product_uom" ref="uom.product_uom_hour"/>
            <field name="price_unit">38.25</field>
        </record>

        <record id="sale_order_3" model="sale.order">
            <field name="partner_id" ref="base.res_partner_4"/>
            <field name="partner_invoice_id" ref="base.res_partner_4"/>
            <field name="partner_shipping_id" ref="base.res_partner_4"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="pricelist_id" ref="product.list0"/>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="sale.utm_source_sale_order_0"/>
            <field name="tag_ids" eval="[(4, ref('sales_team.categ_oppor1')), (4, ref('sales_team.categ_oppor2'))]"/>
        </record>

        <record id="sale_order_line_6" model="sale.order.line">
            <field name="order_id" ref="sale_order_3"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_1').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_1"/>
            <field name="product_uom_qty">10</field>
            <field name="product_uom" ref="uom.product_uom_hour"/>
            <field name="price_unit">30.75</field>
        </record>

        <record id="sale_order_line_7" model="sale.order.line">
            <field name="order_id" ref="sale_order_3"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_delivery_01').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_delivery_01"/>
            <field name="product_uom_qty">1</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">70.00</field>
        </record>

        <record id="sale_order_4" model="sale.order">
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="partner_invoice_id" ref="base.res_partner_address_25"/>
            <field name="partner_shipping_id" ref="base.res_partner_address_25"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="pricelist_id" ref="product.list0"/>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="sale.utm_source_sale_order_0"/>
        </record>

        <record id="sale_order_line_8" model="sale.order.line">
            <field name="order_id" ref="sale_order_4"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_1').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_1"/>
            <field name="product_uom_qty">16</field>
            <field name="product_uom" ref="uom.product_uom_hour"/>
            <field name="price_unit">75.00</field>
        </record>

        <record id="sale_order_line_9" model="sale.order.line">
            <field name="order_id" ref="sale_order_4"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_delivery_02').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_delivery_02"/>
            <field name="product_uom_qty">10</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">45.00</field>
        </record>

        <record id="sale_order_line_10" model="sale.order.line">
            <field name="order_id" ref="sale_order_4"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.consu_delivery_02').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.consu_delivery_02"/>
            <field name="product_uom_qty">3</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">150.00</field>
        </record>

        <record id="sale_order_line_11" model="sale.order.line">
            <field name="order_id" ref="sale_order_4"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_delivery_01').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_delivery_01"/>
            <field name="product_uom_qty">2</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">70.00</field>
        </record>

        <record id="sale_order_5" model="sale.order">
            <field name="partner_id" ref="base.res_partner_2"/>
            <field name="partner_invoice_id" ref="base.res_partner_2"/>
            <field name="partner_shipping_id" ref="base.res_partner_2"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="pricelist_id" ref="product.list0"/>
            <field name="team_id" ref="sales_team.crm_team_1"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="sale.utm_source_sale_order_0"/>
            <field name="date_order" eval="(DateTime.today() - relativedelta(months=1)).strftime('%Y-%m-%d %H:%M')"/>
        </record>

        <record id="sale_order_line_12" model="sale.order.line">
            <field name="order_id" ref="sale_order_5"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_delivery_02').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_delivery_02"/>
            <field name="product_uom_qty">1</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">405.00</field>
        </record>

        <record id="sale_order_6" model="sale.order">
            <field name="partner_id" ref="base.res_partner_18"/>
            <field name="partner_invoice_id" ref="base.res_partner_18"/>
            <field name="partner_shipping_id" ref="base.res_partner_18"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="pricelist_id" ref="product.list0"/>
            <field name="team_id" ref="sales_team.crm_team_1"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="sale.utm_source_sale_order_0"/>
            <field name="tag_ids" eval="[(4, ref('sales_team.categ_oppor6'))]"/>
        </record>

        <record id="sale_order_line_15" model="sale.order.line">
            <field name="order_id" ref="sale_order_6"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_4').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_4"/>
            <field name="product_uom_qty">1</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">750.00</field>
        </record>

        <record id="sale_order_7" model="sale.order">
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="partner_invoice_id" ref="base.res_partner_address_11"/>
            <field name="partner_shipping_id" ref="base.res_partner_address_11"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="pricelist_id" ref="product.list0"/>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="sale.utm_source_sale_order_0"/>
            <field name="tag_ids" eval="[(4, ref('sales_team.categ_oppor4'))]"/>
        </record>

        <record id="sale_order_line_16" model="sale.order.line">
            <field name="order_id" ref="sale_order_7"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_25').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_25"/>
            <field name="product_uom_qty">5</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">295.00</field>
        </record>

        <record id="sale_order_line_17" model="sale.order.line">
            <field name="order_id" ref="sale_order_7"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.consu_delivery_01').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.consu_delivery_01"/>
            <field name="product_uom_qty">1</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">173.00</field>
        </record>

        <record id="sale_order_line_18" model="sale.order.line">
            <field name="order_id" ref="sale_order_7"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_delivery_02').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_delivery_02"/>
            <field name="product_uom_qty">1</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">40.00</field>
        </record>

        <record id="sale_order_line_19" model="sale.order.line">
            <field name="order_id" ref="sale_order_7"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_delivery_01').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_delivery_01"/>
            <field name="product_uom_qty">1</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">18.00</field>
        </record>

        <record id="sale_order_8" model="sale.order">
            <field name="name">Test/001</field>
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="partner_invoice_id" ref="base.res_partner_address_25"/>
            <field name="partner_shipping_id" ref="base.res_partner_address_25"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="pricelist_id" ref="product.list0"/>
            <field name="team_id" ref="sales_team.crm_team_1"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="sale.utm_source_sale_order_0"/>
        </record>

        <record id="sale_order_line_20" model="sale.order.line">
            <field name="order_id" ref="sale_order_8"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_27').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_27"/>
            <field name="product_uom_qty">2</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">110.50</field>
        </record>

        <record id="sale_order_line_21" model="sale.order.line">
            <field name="order_id" ref="sale_order_8"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_12').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_12"/>
            <field name="product_uom_qty">2</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">120.50</field>
        </record>

        <!-- additional demo data for pretty graphs in sales dashboard -->

        <record id="sale_order_9" model="sale.order">
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="partner_invoice_id" ref="base.res_partner_address_25"/>
            <field name="partner_shipping_id" ref="base.res_partner_address_25"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="pricelist_id" ref="product.list0"/>
            <field name="team_id" ref="sales_team.crm_team_1"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="sale.utm_source_sale_order_0"/>
            <field name="date_order" eval="(datetime.now()-relativedelta(days=7)).strftime('%Y-%m-%d %H:%M:%S')"/>
        </record>

        <record id="sale_order_line_22" model="sale.order.line">
            <field name="order_id" ref="sale_order_9"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_27').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_27"/>
            <field name="product_uom_qty">3</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">97.50</field>
        </record>

        <record id="sale_order_line_23" model="sale.order.line">
            <field name="order_id" ref="sale_order_9"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_12').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_12"/>
            <field name="product_uom_qty">3</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">120.50</field>
        </record>

        <record id="sale_order_10" model="sale.order">
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="partner_invoice_id" ref="base.res_partner_address_25"/>
            <field name="partner_shipping_id" ref="base.res_partner_address_25"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="pricelist_id" ref="product.list0"/>
            <field name="team_id" ref="sales_team.crm_team_1"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="sale.utm_source_sale_order_0"/>
            <field name="date_order" eval="(datetime.now()-relativedelta(days=14)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="tag_ids" eval="[(4, ref('sales_team.categ_oppor3'))]"/>
        </record>

        <record id="sale_order_line_24" model="sale.order.line">
            <field name="order_id" ref="sale_order_10"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_25').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_25"/>
            <field name="product_uom_qty">2</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">255.00</field>
        </record>

        <record id="sale_order_line_25" model="sale.order.line">
            <field name="order_id" ref="sale_order_10"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_12').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_12"/>
            <field name="product_uom_qty">2</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">120.50</field>
        </record>


        <record id="sale_order_11" model="sale.order">
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="partner_invoice_id" ref="base.res_partner_address_25"/>
            <field name="partner_shipping_id" ref="base.res_partner_address_25"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="pricelist_id" ref="product.list0"/>
            <field name="team_id" ref="sales_team.crm_team_1"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="sale.utm_source_sale_order_0"/>
            <field name="date_order" eval="(datetime.now()-relativedelta(days=21)).strftime('%Y-%m-%d %H:%M:%S')"/>
        </record>

        <record id="sale_order_line_26" model="sale.order.line">
            <field name="order_id" ref="sale_order_11"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_25').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_25"/>
            <field name="product_uom_qty">3</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">245.00</field>
        </record>

        <record id="sale_order_line_27" model="sale.order.line">
            <field name="order_id" ref="sale_order_11"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_12').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_12"/>
            <field name="product_uom_qty">3</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">120.50</field>
        </record>

        <record id="sale_order_12" model="sale.order">
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="partner_invoice_id" ref="base.res_partner_address_25"/>
            <field name="partner_shipping_id" ref="base.res_partner_address_25"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="pricelist_id" ref="product.list0"/>
            <field name="team_id" ref="sales_team.crm_team_1"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="sale.utm_source_sale_order_0"/>
            <field name="date_order" eval="(datetime.now()-relativedelta(days=28)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="tag_ids" eval="[(4, ref('sales_team.categ_oppor1'))]"/>
        </record>

        <record id="sale_order_line_28" model="sale.order.line">
            <field name="order_id" ref="sale_order_12"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_25').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_25"/>
            <field name="product_uom_qty">1</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">315.00</field>
        </record>

        <record id="sale_order_line_29" model="sale.order.line">
            <field name="order_id" ref="sale_order_12"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_12').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_12"/>
            <field name="product_uom_qty">2</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">120.50</field>
        </record>

        <record id="sale_order_13" model="sale.order">
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="partner_invoice_id" ref="base.res_partner_address_25"/>
            <field name="partner_shipping_id" ref="base.res_partner_address_25"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="pricelist_id" ref="product.list0"/>
            <field name="team_id" ref="sales_team.crm_team_1"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="sale.utm_source_sale_order_0"/>
            <field name="date_order" eval="(datetime.now()-relativedelta(days=35)).strftime('%Y-%m-%d %H:%M:%S')"/>
        </record>

        <record id="sale_order_line_30" model="sale.order.line">
            <field name="order_id" ref="sale_order_13"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_25').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_25"/>
            <field name="product_uom_qty">1</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">295.00</field>
        </record>

        <record id="sale_order_line_31" model="sale.order.line">
            <field name="order_id" ref="sale_order_13"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_12').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_12"/>
            <field name="product_uom_qty">1</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">120.50</field>
        </record>

        <record id="sale_order_14" model="sale.order">
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="partner_invoice_id" ref="base.res_partner_address_25"/>
            <field name="partner_shipping_id" ref="base.res_partner_address_25"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="pricelist_id" ref="product.list0"/>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="sale.utm_source_sale_order_0"/>
            <field name="date_order" eval="(datetime.now()-relativedelta(days=7)).strftime('%Y-%m-%d %H:%M:%S')"/>
        </record>

        <record id="sale_order_line_32" model="sale.order.line">
            <field name="order_id" ref="sale_order_14"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_25').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_25"/>
            <field name="product_uom_qty">4</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">275.00</field>
        </record>

        <record id="sale_order_line_33" model="sale.order.line">
            <field name="order_id" ref="sale_order_14"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_12').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_12"/>
            <field name="product_uom_qty">4</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">120.50</field>
        </record>

        <record id="sale_order_15" model="sale.order">
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="partner_invoice_id" ref="base.res_partner_address_25"/>
            <field name="partner_shipping_id" ref="base.res_partner_address_25"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="pricelist_id" ref="product.list0"/>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="sale.utm_source_sale_order_0"/>
            <field name="date_order" eval="(datetime.now()-relativedelta(days=14)).strftime('%Y-%m-%d %H:%M:%S')"/>
        </record>

        <record id="sale_order_line_34" model="sale.order.line">
            <field name="order_id" ref="sale_order_15"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_25').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_25"/>
            <field name="product_uom_qty">4</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">295.00</field>
        </record>

        <record id="sale_order_line_35" model="sale.order.line">
            <field name="order_id" ref="sale_order_15"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_12').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_12"/>
            <field name="product_uom_qty">3</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">120.50</field>
        </record>

        <record id="sale_order_16" model="sale.order">
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="partner_invoice_id" ref="base.res_partner_address_25"/>
            <field name="partner_shipping_id" ref="base.res_partner_address_25"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="pricelist_id" ref="product.list0"/>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="sale.utm_source_sale_order_0"/>
            <field name="date_order" eval="(datetime.now()-relativedelta(days=21)).strftime('%Y-%m-%d %H:%M:%S')"/>
        </record>

        <record id="sale_order_line_36" model="sale.order.line">
            <field name="order_id" ref="sale_order_16"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_25').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_25"/>
            <field name="product_uom_qty">3</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">275.00</field>
        </record>

        <record id="sale_order_line_37" model="sale.order.line">
            <field name="order_id" ref="sale_order_16"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_12').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_12"/>
            <field name="product_uom_qty">3</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">120.50</field>
        </record>

        <record id="sale_order_17" model="sale.order">
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="partner_invoice_id" ref="base.res_partner_address_25"/>
            <field name="partner_shipping_id" ref="base.res_partner_address_25"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="pricelist_id" ref="product.list0"/>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="sale.utm_source_sale_order_0"/>
            <field name="date_order" eval="(datetime.now()-relativedelta(days=28)).strftime('%Y-%m-%d %H:%M:%S')"/>
        </record>

        <record id="sale_order_line_38" model="sale.order.line">
            <field name="order_id" ref="sale_order_17"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_25').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_25"/>
            <field name="product_uom_qty">2</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">355.00</field>
        </record>

        <record id="sale_order_line_39" model="sale.order.line">
            <field name="order_id" ref="sale_order_17"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_12').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_12"/>
            <field name="product_uom_qty">2</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">120.50</field>
        </record>

        <record id="sale_order_18" model="sale.order">
            <field name="partner_id" ref="base.res_partner_3"/>
            <field name="partner_invoice_id" ref="base.res_partner_address_25"/>
            <field name="partner_shipping_id" ref="base.res_partner_address_25"/>
            <field name="user_id" ref="base.user_demo"/>
            <field name="pricelist_id" ref="product.list0"/>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="campaign_id" ref="utm.utm_campaign_email_campaign_products"/>
            <field name="medium_id" ref="utm.utm_medium_email"/>
            <field name="source_id" ref="sale.utm_source_sale_order_0"/>
            <field name="date_order" eval="(datetime.now()-relativedelta(days=35)).strftime('%Y-%m-%d %H:%M:%S')"/>
        </record>

        <record id="sale_order_line_40" model="sale.order.line">
            <field name="order_id" ref="sale_order_18"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_25').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_25"/>
            <field name="product_uom_qty">2</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">295.00</field>
        </record>

        <record id="sale_order_line_41" model="sale.order.line">
            <field name="order_id" ref="sale_order_18"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_12').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_12"/>
            <field name="product_uom_qty">2</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">120.50</field>
        </record>

        <!-- Confirm some Sales Orders-->
        <function model="sale.order" name="action_confirm" eval="[[ref('sale_order_4')]]"/>
        <function model="sale.order" name="action_confirm" eval="[[ref('sale_order_6')]]"/>
        <function model="sale.order" name="action_confirm" eval="[[ref('sale_order_7')]]"/>
        <function model="sale.order" name="action_confirm" eval="[[ref('sale_order_8')]]"/>
        <function model="sale.order" name="action_confirm" eval="[[ref('sale_order_9')]]"/>
        <function model="sale.order" name="action_confirm" eval="[[ref('sale_order_10')]]"/>
        <function model="sale.order" name="action_confirm" eval="[[ref('sale_order_11')]]"/>
        <function model="sale.order" name="action_confirm" eval="[[ref('sale_order_12')]]"/>
        <function model="sale.order" name="action_confirm" eval="[[ref('sale_order_13')]]"/>
        <function model="sale.order" name="action_confirm" eval="[[ref('sale_order_14')]]"/>
        <function model="sale.order" name="action_confirm" eval="[[ref('sale_order_15')]]"/>
        <function model="sale.order" name="action_confirm" eval="[[ref('sale_order_16')]]"/>
        <function model="sale.order" name="action_confirm" eval="[[ref('sale_order_17')]]"/>
        <function model="sale.order" name="action_confirm" eval="[[ref('sale_order_18')]]"/>

        <!-- Setting date_order in the past for beautiful spread -->
        <record id="sale_order_9" model="sale.order">
            <field name="date_order" eval="datetime.now() - relativedelta(days=7)"/>
        </record>

        <record id="sale_order_11" model="sale.order">
            <field name="date_order" eval="datetime.now() - relativedelta(days=21)"/>
        </record>

        <record id="sale_order_15" model="sale.order">
            <field name="date_order" eval="datetime.now() - relativedelta(days=14)"/>
        </record>

        <record id="sale_order_17" model="sale.order">
            <field name="date_order" eval="datetime.now() - relativedelta(days=28)"/>
        </record>

        <record id="sale_order_18" model="sale.order">
            <field name="date_order" eval="datetime.now() - relativedelta(days=35)"/>
        </record>

        <record id="message_sale_1" model="mail.message">
            <field name="model">sale.order</field>
            <field name="res_id" ref="sale_order_2"/>
            <field name="body">Hi,
I have a question regarding services pricing: I heard of a possible discount for quantities exceeding 25 hours.
Could you confirm, please?</field>
            <field name="message_type">comment</field>
            <field name="author_id" ref="base.partner_demo"/>
        </record>

        <record id="message_sale_2" model="mail.message">
            <field name="model">sale.order</field>
            <field name="res_id" ref="sale_order_2"/>
            <field name="parent_id" ref="message_sale_1"/>
            <field name="body">Hello,
Unfortunately that was a temporary discount that is not available anymore.
Do you still plan to confirm the order based on the quoted prices?
Thanks!</field>
            <field name="message_type">comment</field>
            <field name="author_id" ref="base.partner_root"/>
        </record>

        <record id="message_sale_3" model="mail.message">
            <field name="model">sale.order</field>
            <field name="res_id" ref="sale_order_2"/>
            <field name="parent_id" ref="message_sale_2"/>
            <field name="body">Alright, thanks for the clarification. I will confirm the order as soon as I get my manager's approval.</field>
            <field name="message_type">comment</field>
            <field name="author_id" ref="base.partner_demo"/>
        </record>
        <!-- sale advance demo.. -->
        <!-- Demo Data for Product -->

        <record id="advance_product_0" model="product.product">
            <field name="name">Deposit</field>
            <field name="categ_id" ref="product.product_category_3"/>
            <field name="type">service</field>
            <field name="list_price">150.0</field>
            <field name="invoice_policy">order</field>
            <field name="standard_price">100.0</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="company_id" eval="[]"/>
            <field name="image_1920" type="base64" file="sale/static/img/advance_product_0-image.jpg"/>
            <field name="taxes_id" eval="[]"/>
            <field name="supplier_taxes_id" eval="[]"/>
        </record>

        <record id="base.user_demo" model="res.users">
            <field eval="[(4, ref('sales_team.group_sale_salesman'))]" name="groups_id"/>
        </record>

        <!-- records coming from website_portal_sale, now dead module -->
        <record id="portal_sale_order_1" model="sale.order">
            <field name="partner_id" ref="base.partner_demo_portal"/>
            <field name="partner_invoice_id" ref="base.partner_demo_portal"/>
            <field name="partner_shipping_id" ref="base.partner_demo_portal"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="pricelist_id" ref="product.list0"/>
            <field name="state">sent</field>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="date_order" eval="(DateTime.today() - relativedelta(months=1)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="message_partner_ids" eval="[(4, ref('base.partner_demo_portal'))]"/>
            <field name="tag_ids" eval="[(4, ref('sales_team.categ_oppor4'))]"/>
        </record>

        <record id="portal_sale_order_line_1" model="sale.order.line">
            <field name="order_id" ref="portal_sale_order_1"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_25').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_25"/>
            <field name="product_uom_qty">3</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">295.00</field>
        </record>

        <record id="portal_sale_order_line_2" model="sale.order.line">
            <field name="order_id" ref="portal_sale_order_1"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_delivery_02').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_delivery_02"/>
            <field name="product_uom_qty">5</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">145.00</field>
        </record>

        <record id="portal_sale_order_line_3" model="sale.order.line">
            <field name="order_id" ref="portal_sale_order_1"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_delivery_01').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_delivery_01"/>
            <field name="product_uom_qty">2</field>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">65.00</field>
        </record>

        <record id="portal_sale_order_2" model="sale.order">
            <field name="partner_id" ref="base.partner_demo_portal"/>
            <field name="partner_invoice_id" ref="base.partner_demo_portal"/>
            <field name="partner_shipping_id" ref="base.partner_demo_portal"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="pricelist_id" ref="product.list0"/>
            <field name="team_id" ref="sales_team.team_sales_department"/>
            <field name="date_order" eval="(DateTime.today() - relativedelta(months=1)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="message_partner_ids" eval="[(4, ref('base.partner_demo_portal'))]"/>
        </record>

        <record id="portal_sale_order_line_4" model="sale.order.line">
            <field name="order_id" ref="portal_sale_order_2"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_1').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_1"/>
            <field name="product_uom_qty">24</field>
            <field name="product_uom" ref="uom.product_uom_hour"/>
            <field name="price_unit">75.00</field>
        </record>

        <record id="portal_sale_order_line_5" model="sale.order.line">
            <field name="order_id" ref="portal_sale_order_2"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('product.product_product_2').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="product.product_product_2"/>
            <field name="product_uom_qty">30</field>
            <field name="product_uom" ref="uom.product_uom_hour"/>
            <field name="price_unit">38.25</field>
        </record>

        <record id="product.product_attribute_2" model="product.attribute">
            <field name="display_type">color</field>
        </record>
        <record id="product.product_attribute_3" model="product.attribute">
            <field name="display_type">select</field>
        </record>

        <record id="product.product_attribute_value_3" model="product.attribute.value">
            <field name="html_color">#FFFFFF</field>
        </record>
        <record id="product.product_attribute_value_4" model="product.attribute.value">
            <field name="html_color">#000000</field>
        </record>

        <!-- Confirm some Sales Orders-->
        <function model="sale.order" name="action_confirm" eval="[[ref('portal_sale_order_2')]]"/>

        <record id="portal_sale_order_2" model="sale.order">
            <field name="date_order" eval="datetime.now() - relativedelta(months=1)"/>
        </record>

        <record id="product_attribute_value_7" model="product.attribute.value">
            <field name="name">Custom</field>
            <field name="attribute_id" ref="product.product_attribute_1"/>
            <field name="is_custom">True</field>
            <field name="sequence">3</field>
        </record>

        <record id="product.product_4_attribute_1_product_template_attribute_line" model="product.template.attribute.line">
            <field name="value_ids" eval="[(4,ref('product_attribute_value_7'))]"/>
        </record>

        <!--
        Handle automatically created product.template.attribute.value.
        Check "product.product_4_attribute_1_value_2" for more information about this
        -->
        <function model="ir.model.data" name="_update_xmlids">
            <value model="base" eval="[{
                'xml_id': 'sale.product_4_attribute_1_value_3',
                'record': obj().env.ref('product.product_4_attribute_1_product_template_attribute_line').product_template_value_ids[2],
                'noupdate': True,
            }]"/>
        </function>

        <function model="ir.model.data" name="_update_xmlids">
            <value model="base" eval="[{
                'xml_id': 'sale.product_product_4e',
                'record': obj().env.ref('product.product_product_4_product_template')._get_variant_for_combination(obj().env.ref('sale.product_4_attribute_1_value_3') + obj().env.ref('product.product_4_attribute_2_value_1')),
                'noupdate': True,
            }, {
                'xml_id': 'sale.product_product_4f',
                'record': obj().env.ref('product.product_product_4_product_template')._get_variant_for_combination(obj().env.ref('sale.product_4_attribute_1_value_3') + obj().env.ref('product.product_4_attribute_2_value_2')),
                'noupdate': True,
            },]"/>
        </function>

        <record id="product_product_4e" model="product.product">
            <field name="default_code">DESK0005</field>
            <field name="weight">0.01</field>
        </record>

        <record id="product_product_4f" model="product.product">
            <field name="default_code">DESK0006</field>
            <field name="weight">0.01</field>
        </record>

        <record id="product_template_attribute_exclusion_1" model="product.template.attribute.exclusion">
            <field name="product_tmpl_id" ref="product.product_product_4_product_template" />
            <field name="value_ids" eval="[(6,0,[ref('product.product_4_attribute_2_value_2')])]"/>
        </record>

        <record id="product_template_attribute_exclusion_2" model="product.template.attribute.exclusion">
            <field name="product_tmpl_id" ref="product.product_product_11_product_template" />
            <field name="value_ids" eval="[(6,0,[ref('product.product_11_attribute_1_value_1')])]"/>
        </record>

        <!--
        The "Customizable Desk's Aluminium" attribute value will excude:
        - The "Customizable Desk's Black" attribute
        - The "Office Chair's Steel" attribute
         -->
        <record id="product.product_4_attribute_1_value_2" model="product.template.attribute.value">
            <field name="exclude_for" eval="[(6,0,[ref('sale.product_template_attribute_exclusion_1') ,ref('sale.product_template_attribute_exclusion_2')])]" />
        </record>

        <record id="product_template_attribute_exclusion_3" model="product.template.attribute.exclusion">
            <field name="product_tmpl_id" ref="product.product_product_11_product_template" />
            <field name="value_ids" eval="[(6,0,[ref('product.product_11_attribute_1_value_2')])]"/>
        </record>

        <!--
        The "Customizable Desk's Steel" attribute value will excude:
        - The "Office Chair's Aluminium" attribute
         -->
        <record id="product.product_4_attribute_1_value_1" model="product.template.attribute.value">
            <field name="exclude_for" eval="[(6,0,[ref('sale.product_template_attribute_exclusion_3')])]" />
        </record>

        <!-- Activities of sales order -->
        <record id="sale_activity_2" model="mail.activity">
            <field name="res_id" ref="sale.sale_order_3"/>
            <field name="res_model_id" ref="sale.model_sale_order"/>
            <field name="activity_type_id" ref="mail.mail_activity_data_email"/>
            <field name="date_deadline" eval="DateTime.today().strftime('%Y-%m-%d %H:%M')"/>
            <field name="summary">Answer questions</field>
            <field name="create_uid" ref="base.user_admin"/>
            <field name="user_id" ref="base.user_admin"/>
        </record>
        <record id="sale_activity_3" model="mail.activity">
            <field name="res_id" ref="sale.sale_order_4"/>
            <field name="res_model_id" ref="sale.model_sale_order"/>
            <field name="activity_type_id" ref="sale.mail_act_sale_upsell"/>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(days=5)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="create_uid" ref="base.user_admin"/>
            <field name="user_id" ref="base.user_admin"/>
        </record>
        <record id="sale_activity_4" model="mail.activity">
            <field name="res_id" ref="sale.sale_order_5"/>
            <field name="res_model_id" ref="sale.model_sale_order"/>
            <field name="activity_type_id" ref="mail.mail_activity_data_email"/>
            <field name="date_deadline" eval="DateTime.today().strftime('%Y-%m-%d %H:%M')"/>
            <field name="create_uid" ref="base.user_demo"/>
            <field name="user_id" ref="base.user_demo"/>
        </record>
        <record id="sale_activity_6" model="mail.activity">
            <field name="res_id" ref="sale.sale_order_7"/>
            <field name="res_model_id" ref="sale.model_sale_order"/>
            <field name="activity_type_id" ref="mail.mail_activity_data_todo"/>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(days=5)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="summary">Check delivery requirements</field>
            <field name="create_uid" ref="base.user_admin"/>
            <field name="user_id" ref="base.user_admin"/>
        </record>
        <record id="sale_activity_7" model="mail.activity">
            <field name="res_id" ref="sale.sale_order_10"/>
            <field name="res_model_id" ref="sale.model_sale_order"/>
            <field name="activity_type_id" ref="mail.mail_activity_data_todo"/>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(days=5)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="summary">Confirm Delivery</field>
            <field name="create_uid" ref="base.user_demo"/>
            <field name="user_id" ref="base.user_demo"/>
        </record>
        <record id="sale_activity_8" model="mail.activity">
            <field name="res_id" ref="sale.sale_order_12"/>
            <field name="res_model_id" ref="sale.model_sale_order"/>
            <field name="activity_type_id" ref="mail.mail_activity_data_email"/>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(days=5)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="create_uid" ref="base.user_demo"/>
            <field name="user_id" ref="base.user_demo"/>
        </record>
        <record id="sale_activity_9" model="mail.activity">
            <field name="res_id" ref="sale.sale_order_16"/>
            <field name="res_model_id" ref="sale.model_sale_order"/>
            <field name="activity_type_id" ref="sale.mail_act_sale_upsell"/>
            <field name="date_deadline" eval="DateTime.today().strftime('%Y-%m-%d %H:%M')"/>
            <field name="create_uid" ref="base.user_demo"/>
            <field name="user_id" ref="base.user_demo"/>
        </record>
        <record id="sale_activity_10" model="mail.activity">
            <field name="res_id" ref="sale.portal_sale_order_1"/>
            <field name="res_model_id" ref="sale.model_sale_order"/>
            <field name="activity_type_id" ref="mail.mail_activity_data_todo"/>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(days=5)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="summary">Get quote confirmation</field>
            <field name="create_uid" ref="base.user_admin"/>
            <field name="user_id" ref="base.user_admin"/>
        </record>

    </data>
</odoo>

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError

class AccountMove(models.Model):
    _name = 'account.move'
    _inherit = ['account.move', 'utm.mixin']

    @api.model
    def _get_invoice_default_sale_team(self):
        return self.env['crm.team']._get_default_team_id()

    team_id = fields.Many2one(
        'crm.team', string='Sales Team', default=_get_invoice_default_sale_team,
        ondelete="set null", tracking=True,
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    partner_shipping_id = fields.Many2one(
        'res.partner',
        string='Delivery Address',
        readonly=True,
        states={'draft': [('readonly', False)]},
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]",
        help="Delivery address for current invoice.")

    @api.onchange('partner_shipping_id', 'company_id')
    def _onchange_partner_shipping_id(self):
        """
        Trigger the change of fiscal position when the shipping address is modified.
        """
        delivery_partner_id = self._get_invoice_delivery_partner_id()
        fiscal_position = self.env['account.fiscal.position'].with_company(self.company_id).get_fiscal_position(
            self.partner_id.id, delivery_id=delivery_partner_id)

        if fiscal_position:
            self.fiscal_position_id = fiscal_position

    def unlink(self):
        downpayment_lines = self.mapped('line_ids.sale_line_ids').filtered(lambda line: line.is_downpayment and line.invoice_lines <= self.mapped('line_ids'))
        res = super(AccountMove, self).unlink()
        if downpayment_lines:
            downpayment_lines.unlink()
        return res

    @api.onchange('partner_id')
    def _onchange_partner_id(self):
        # OVERRIDE
        # Recompute 'partner_shipping_id' based on 'partner_id'.
        addr = self.partner_id.address_get(['delivery'])
        self.partner_shipping_id = addr and addr.get('delivery')

        res = super(AccountMove, self)._onchange_partner_id()

        return res

    @api.onchange('invoice_user_id')
    def onchange_user_id(self):
        if self.invoice_user_id and self.invoice_user_id.sale_team_id:
            self.team_id = self.env['crm.team']._get_default_team_id(user_id=self.invoice_user_id.id, domain=[('company_id', '=', self.company_id.id)])

    def _reverse_moves(self, default_values_list=None, cancel=False):
        # OVERRIDE
        if not default_values_list:
            default_values_list = [{} for move in self]
        for move, default_values in zip(self, default_values_list):
            default_values.update({
                'campaign_id': move.campaign_id.id,
                'medium_id': move.medium_id.id,
                'source_id': move.source_id.id,
            })
        return super()._reverse_moves(default_values_list=default_values_list, cancel=cancel)

    def action_post(self):
        #inherit of the function from account.move to validate a new tax and the priceunit of a downpayment
        res = super(AccountMove, self).action_post()
        downpayment_lines = self.line_ids.sale_line_ids.filtered('is_downpayment')
        other_so_lines = downpayment_lines.order_id.order_line - downpayment_lines
        real_invoices = set(other_so_lines.invoice_lines.move_id)
        for dpl in downpayment_lines:
            try:
                dpl.price_unit = sum(
                    l.price_unit if l.move_id.move_type == 'out_invoice' else -l.price_unit
                    for l in dpl.invoice_lines
                    if l.move_id.state == 'posted' and l.move_id not in real_invoices  # don't recompute with the final invoice
                )
                dpl.tax_id = dpl.invoice_lines.tax_ids
            except UserError:
                # a UserError here means the SO was locked, which prevents changing the taxes
                # just ignore the error - this is a nice to have feature and should not be blocking
                pass
        return res

    def _post(self, soft=True):
        # OVERRIDE
        # Auto-reconcile the invoice with payments coming from transactions.
        # It's useful when you have a "paid" sale order (using a payment transaction) and you invoice it later.
        posted = super()._post(soft)

        for invoice in posted.filtered(lambda move: move.is_invoice()):
            payments = invoice.mapped('transaction_ids.payment_id').filtered(lambda x: x.state == 'posted')
            move_lines = payments.line_ids.filtered(lambda line: line.account_internal_type in ('receivable', 'payable') and not line.reconciled)
            for line in move_lines:
                invoice.js_assign_outstanding_line(line.id)
        return posted

    def action_invoice_paid(self):
        # OVERRIDE
        res = super(AccountMove, self).action_invoice_paid()
        todo = set()
        for invoice in self.filtered(lambda move: move.is_invoice()):
            for line in invoice.invoice_line_ids:
                for sale_line in line.sale_line_ids:
                    todo.add((sale_line.order_id, invoice.name))
        for (order, name) in todo:
            order.message_post(body=_("Invoice %s paid", name))
        return res

    def _get_invoice_delivery_partner_id(self):
        # OVERRIDE
        self.ensure_one()
        return self.partner_shipping_id.id or super(AccountMove, self)._get_invoice_delivery_partner_id()

    def _action_invoice_ready_to_be_sent(self):
        # OVERRIDE
        # Make sure the send invoice CRON is called when an invoice becomes ready to be sent by mail.
        res = super()._action_invoice_ready_to_be_sent()

        send_invoice_cron = self.env.ref('sale.send_invoice_cron', raise_if_not_found=False)
        if send_invoice_cron:
            send_invoice_cron._trigger()

        return res

    def _is_downpayment(self):
        # OVERRIDE
        self.ensure_one()
        return self.line_ids.sale_line_ids and all(sale_line.is_downpayment for sale_line in self.line_ids.sale_line_ids) or False

```

## File: models\account_move_line.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _
from odoo.exceptions import UserError
from odoo.tools import float_compare, float_is_zero


class AccountMoveLine(models.Model):
    _inherit = 'account.move.line'

    sale_line_ids = fields.Many2many(
        'sale.order.line',
        'sale_order_line_invoice_rel',
        'invoice_line_id', 'order_line_id',
        string='Sales Order Lines', readonly=True, copy=False)

    def _copy_data_extend_business_fields(self, values):
        # OVERRIDE to copy the 'sale_line_ids' field as well.
        super(AccountMoveLine, self)._copy_data_extend_business_fields(values)
        values['sale_line_ids'] = [(6, None, self.sale_line_ids.ids)]

    def _prepare_analytic_line(self):
        """ Note: This method is called only on the move.line that having an analytic account, and
            so that should create analytic entries.
        """
        values_list = super(AccountMoveLine, self)._prepare_analytic_line()

        # filter the move lines that can be reinvoiced: a cost (negative amount) analytic line without SO line but with a product can be reinvoiced
        move_to_reinvoice = self.env['account.move.line']
        for index, move_line in enumerate(self):
            values = values_list[index]
            if 'so_line' not in values:
                if move_line._sale_can_be_reinvoice():
                    move_to_reinvoice |= move_line

        # insert the sale line in the create values of the analytic entries
        if move_to_reinvoice:
            map_sale_line_per_move = move_to_reinvoice._sale_create_reinvoice_sale_line()

            for values in values_list:
                sale_line = map_sale_line_per_move.get(values.get('move_id'))
                if sale_line:
                    values['so_line'] = sale_line.id

        return values_list

    def _sale_can_be_reinvoice(self):
        """ determine if the generated analytic line should be reinvoiced or not.
            For Vendor Bill flow, if the product has a 'erinvoice policy' and is a cost, then we will find the SO on which reinvoice the AAL
        """
        self.ensure_one()
        if self.sale_line_ids:
            return False
        uom_precision_digits = self.env['decimal.precision'].precision_get('Product Unit of Measure')
        return float_compare(self.credit or 0.0, self.debit or 0.0, precision_digits=uom_precision_digits) != 1 and self.product_id.expense_policy not in [False, 'no']

    def _sale_create_reinvoice_sale_line(self):

        sale_order_map = self._sale_determine_order()

        sale_line_values_to_create = []  # the list of creation values of sale line to create.
        existing_sale_line_cache = {}  # in the sales_price-delivery case, we can reuse the same sale line. This cache will avoid doing a search each time the case happen
        # `map_move_sale_line` is map where
        #   - key is the move line identifier
        #   - value is either a sale.order.line record (existing case), or an integer representing the index of the sale line to create in
        #     the `sale_line_values_to_create` (not existing case, which will happen more often than the first one).
        map_move_sale_line = {}

        for move_line in self:
            sale_order = sale_order_map.get(move_line.id)

            # no reinvoice as no sales order was found
            if not sale_order:
                continue

            # raise if the sale order is not currenlty open
            if sale_order.state != 'sale':
                message_unconfirmed = _('The Sales Order %s linked to the Analytic Account %s must be validated before registering expenses.')
                messages = {
                    'draft': message_unconfirmed,
                    'sent': message_unconfirmed,
                    'done': _('The Sales Order %s linked to the Analytic Account %s is currently locked. You cannot register an expense on a locked Sales Order. Please create a new SO linked to this Analytic Account.'),
                    'cancel': _('The Sales Order %s linked to the Analytic Account %s is cancelled. You cannot register an expense on a cancelled Sales Order.'),
                }
                raise UserError(messages[sale_order.state] % (sale_order.name, sale_order.analytic_account_id.name))

            price = move_line._sale_get_invoice_price(sale_order)

            # find the existing sale.line or keep its creation values to process this in batch
            sale_line = None
            if (
                move_line.product_id.expense_policy == 'sales_price'
                and move_line.product_id.invoice_policy == 'delivery'
                and not self.env.context.get('force_split_lines')
            ):
                # for those case only, we can try to reuse one
                map_entry_key = (sale_order.id, move_line.product_id.id, price)  # cache entry to limit the call to search
                sale_line = existing_sale_line_cache.get(map_entry_key)
                if sale_line:  # already search, so reuse it. sale_line can be sale.order.line record or index of a "to create values" in `sale_line_values_to_create`
                    map_move_sale_line[move_line.id] = sale_line
                    existing_sale_line_cache[map_entry_key] = sale_line
                else:  # search for existing sale line
                    sale_line = self.env['sale.order.line'].search([
                        ('order_id', '=', sale_order.id),
                        ('price_unit', '=', price),
                        ('product_id', '=', move_line.product_id.id),
                        ('is_expense', '=', True),
                    ], limit=1)
                    if sale_line:  # found existing one, so keep the browse record
                        map_move_sale_line[move_line.id] = existing_sale_line_cache[map_entry_key] = sale_line
                    else:  # should be create, so use the index of creation values instead of browse record
                        # save value to create it
                        sale_line_values_to_create.append(move_line._sale_prepare_sale_line_values(sale_order, price))
                        # store it in the cache of existing ones
                        existing_sale_line_cache[map_entry_key] = len(sale_line_values_to_create) - 1  # save the index of the value to create sale line
                        # store it in the map_move_sale_line map
                        map_move_sale_line[move_line.id] = len(sale_line_values_to_create) - 1  # save the index of the value to create sale line

            else:  # save its value to create it anyway
                sale_line_values_to_create.append(move_line._sale_prepare_sale_line_values(sale_order, price))
                map_move_sale_line[move_line.id] = len(sale_line_values_to_create) - 1  # save the index of the value to create sale line

        # create the sale lines in batch
        new_sale_lines = self.env['sale.order.line'].create(sale_line_values_to_create)
        for sol in new_sale_lines:
            sol._onchange_discount()

        # build result map by replacing index with newly created record of sale.order.line
        result = {}
        for move_line_id, unknown_sale_line in map_move_sale_line.items():
            if isinstance(unknown_sale_line, int):  # index of newly created sale line
                result[move_line_id] = new_sale_lines[unknown_sale_line]
            elif isinstance(unknown_sale_line, models.BaseModel):  # already record of sale.order.line
                result[move_line_id] = unknown_sale_line
        return result

    def _sale_determine_order(self):
        """ Get the mapping of move.line with the sale.order record on which its analytic entries should be reinvoiced
            :return a dict where key is the move line id, and value is sale.order record (or None).
        """
        analytic_accounts = self.mapped('analytic_account_id')

        # link the analytic account with its open SO by creating a map: {AA.id: sale.order}, if we find some analytic accounts
        mapping = {}
        if analytic_accounts:  # first, search for the open sales order
            sale_orders = self.env['sale.order'].search([('analytic_account_id', 'in', analytic_accounts.ids), ('state', '=', 'sale')], order='create_date DESC')
            for sale_order in sale_orders:
                mapping[sale_order.analytic_account_id.id] = sale_order

            analytic_accounts_without_open_order = analytic_accounts.filtered(lambda account: not mapping.get(account.id))
            if analytic_accounts_without_open_order:  # then, fill the blank with not open sales orders
                sale_orders = self.env['sale.order'].search([('analytic_account_id', 'in', analytic_accounts_without_open_order.ids)], order='create_date DESC')
            for sale_order in sale_orders:
                mapping[sale_order.analytic_account_id.id] = sale_order

        # map of AAL index with the SO on which it needs to be reinvoiced. Maybe be None if no SO found
        return {move_line.id: mapping.get(move_line.analytic_account_id.id) for move_line in self}

    def _sale_prepare_sale_line_values(self, order, price):
        """ Generate the sale.line creation value from the current move line """
        self.ensure_one()
        last_so_line = self.env['sale.order.line'].search([('order_id', '=', order.id)], order='sequence desc', limit=1)
        last_sequence = last_so_line.sequence + 1 if last_so_line else 100

        fpos = order.fiscal_position_id or order.fiscal_position_id.get_fiscal_position(order.partner_id.id)
        product_taxes = self.product_id.taxes_id.filtered(lambda tax: tax.company_id == order.company_id)
        taxes = fpos.map_tax(product_taxes)

        return {
            'order_id': order.id,
            'name': self.name,
            'sequence': last_sequence,
            'price_unit': price,
            'tax_id': [x.id for x in taxes],
            'discount': 0.0,
            'product_id': self.product_id.id,
            'product_uom': self.product_uom_id.id,
            'product_uom_qty': 0.0,
            'is_expense': True,
        }

    def _sale_get_invoice_price(self, order):
        """ Based on the current move line, compute the price to reinvoice the analytic line that is going to be created (so the
            price of the sale line).
        """
        self.ensure_one()

        unit_amount = self.quantity
        amount = (self.credit or 0.0) - (self.debit or 0.0)

        if self.product_id.expense_policy == 'sales_price':
            return self.product_id.with_context(
                partner=order.partner_id,
                date_order=order.date_order,
                pricelist=order.pricelist_id.id,
                uom=self.product_uom_id.id
            ).price

        uom_precision_digits = self.env['decimal.precision'].precision_get('Product Unit of Measure')
        if float_is_zero(unit_amount, precision_digits=uom_precision_digits):
            return 0.0

        # Prevent unnecessary currency conversion that could be impacted by exchange rate
        # fluctuations
        if self.company_id.currency_id and amount and self.company_id.currency_id == order.currency_id:
            return abs(amount / unit_amount)

        price_unit = abs(amount / unit_amount)
        currency_id = self.company_id.currency_id
        if currency_id and currency_id != order.currency_id:
            price_unit = currency_id._convert(price_unit, order.currency_id, order.company_id, order.date_order or fields.Date.today())
        return price_unit

    def _get_downpayment_lines(self):
        # OVERRIDE
        return self.sale_line_ids.filtered('is_downpayment').invoice_lines.filtered(lambda line: line.move_id._is_downpayment())

```

## File: models\analytic.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class AccountAnalyticLine(models.Model):
    _inherit = "account.analytic.line"

    def _default_sale_line_domain(self):
        """ This is only used for delivered quantity of SO line based on analytic line, and timesheet
            (see sale_timesheet). This can be override to allow further customization.
        """
        return [('qty_delivered_method', '=', 'analytic')]

    so_line = fields.Many2one('sale.order.line', string='Sales Order Item', domain=lambda self: self._default_sale_line_domain())

```

## File: models\crm_team.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError


class CrmTeam(models.Model):
    _inherit = 'crm.team'

    use_quotations = fields.Boolean(string='Quotations', help="Check this box if you send quotations to your customers rather than confirming orders straight away.")
    invoiced = fields.Float(
        compute='_compute_invoiced',
        string='Invoiced This Month', readonly=True,
        help="Invoice revenue for the current month. This is the amount the sales "
                "channel has invoiced this month. It is used to compute the progression ratio "
                "of the current and target revenue on the kanban view.")
    invoiced_target = fields.Float(
        string='Invoicing Target',
        help="Revenue target for the current month (untaxed total of confirmed invoices).")
    quotations_count = fields.Integer(
        compute='_compute_quotations_to_invoice',
        string='Number of quotations to invoice', readonly=True)
    quotations_amount = fields.Float(
        compute='_compute_quotations_to_invoice',
        string='Amount of quotations to invoice', readonly=True)
    sales_to_invoice_count = fields.Integer(
        compute='_compute_sales_to_invoice',
        string='Number of sales to invoice', readonly=True)
    sale_order_count = fields.Integer(compute='_compute_sale_order_count', string='# Sale Orders')

    def _compute_quotations_to_invoice(self):
        query = self.env['sale.order']._where_calc([
            ('team_id', 'in', self.ids),
            ('state', 'in', ['draft', 'sent']),
        ])
        self.env['sale.order']._apply_ir_rules(query, 'read')
        _, where_clause, where_clause_args = query.get_sql()
        select_query = """
            SELECT team_id, count(*), sum(amount_total /
                CASE COALESCE(currency_rate, 0)
                WHEN 0 THEN 1.0
                ELSE currency_rate
                END
            ) as amount_total
            FROM sale_order
            WHERE %s
            GROUP BY team_id
        """ % where_clause
        self.env.cr.execute(select_query, where_clause_args)
        quotation_data = self.env.cr.dictfetchall()
        teams = self.browse()
        for datum in quotation_data:
            team = self.browse(datum['team_id'])
            team.quotations_amount = datum['amount_total']
            team.quotations_count = datum['count']
            teams |= team
        remaining = (self - teams)
        remaining.quotations_amount = 0
        remaining.quotations_count = 0

    def _compute_sales_to_invoice(self):
        sale_order_data = self.env['sale.order'].read_group([
            ('team_id', 'in', self.ids),
            ('invoice_status','=','to invoice'),
        ], ['team_id'], ['team_id'])
        data_map = {datum['team_id'][0]: datum['team_id_count'] for datum in sale_order_data}
        for team in self:
            team.sales_to_invoice_count = data_map.get(team.id,0.0)

    def _compute_invoiced(self):
        if not self:
            return

        query = '''
            SELECT
                move.team_id AS team_id,
                SUM(move.amount_untaxed_signed) AS amount_untaxed_signed
            FROM account_move move
            WHERE move.move_type IN ('out_invoice', 'out_refund', 'out_receipt')
            AND move.payment_state IN ('in_payment', 'paid', 'reversed')
            AND move.state = 'posted'
            AND move.team_id IN %s
            AND move.date BETWEEN %s AND %s
            GROUP BY move.team_id
        '''
        today = fields.Date.today()
        params = [tuple(self.ids), fields.Date.to_string(today.replace(day=1)), fields.Date.to_string(today)]
        self._cr.execute(query, params)

        data_map = dict((v[0], v[1]) for v in self._cr.fetchall())
        for team in self:
            team.invoiced = data_map.get(team.id, 0.0)

    def _compute_sale_order_count(self):
        data_map = {}
        if self.ids:
            sale_order_data = self.env['sale.order'].read_group([
                ('team_id', 'in', self.ids),
                ('state', '!=', 'cancel'),
            ], ['team_id'], ['team_id'])
            data_map = {datum['team_id'][0]: datum['team_id_count'] for datum in sale_order_data}
        for team in self:
            team.sale_order_count = data_map.get(team.id, 0)

    def _graph_get_model(self):
        if self._context.get('in_sales_app'):
            return 'sale.report'
        return super(CrmTeam,self)._graph_get_model()

    def _graph_date_column(self):
        if self._context.get('in_sales_app'):
            return 'date'
        return super(CrmTeam,self)._graph_date_column()

    def _graph_y_query(self):
        if self._context.get('in_sales_app'):
            return 'SUM(price_subtotal)'
        return super(CrmTeam,self)._graph_y_query()

    def _extra_sql_conditions(self):
        if self._context.get('in_sales_app'):
            return "AND state in ('sale', 'done', 'pos_done')"
        return super(CrmTeam,self)._extra_sql_conditions()

    def _graph_title_and_key(self):
        if self._context.get('in_sales_app'):
            return ['', _('Sales: Untaxed Total')] # no more title
        return super(CrmTeam, self)._graph_title_and_key()

    def _compute_dashboard_button_name(self):
        super(CrmTeam,self)._compute_dashboard_button_name()
        if self._context.get('in_sales_app'):
            self.update({'dashboard_button_name': _("Sales Analysis")})

    def action_primary_channel_button(self):
        if self._context.get('in_sales_app'):
            return self.env["ir.actions.actions"]._for_xml_id("sale.action_order_report_so_salesteam")
        return super(CrmTeam, self).action_primary_channel_button()

    def update_invoiced_target(self, value):
        return self.write({'invoiced_target': round(float(value or 0))})

    @api.ondelete(at_uninstall=False)
    def _unlink_except_used_for_sales(self):
        """ If more than 5 active SOs, we consider this team to be actively used.
        5 is some random guess based on "user testing", aka more than testing
        CRM feature and less than use it in real life use cases. """
        SO_COUNT_TRIGGER = 5
        for team in self:
            if team.sale_order_count >= SO_COUNT_TRIGGER:
                raise UserError(
                    _('Team %(team_name)s has %(sale_order_count)s active sale orders. Consider canceling them or archiving the team instead.',
                      team_name=team.name,
                      sale_order_count=team.sale_order_count
                      ))

```

## File: models\payment_acquirer.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class PaymentAcquirer(models.Model):
    _inherit = 'payment.acquirer'

    so_reference_type = fields.Selection(string='Communication',
        selection=[
            ('so_name', 'Based on Document Reference'),
            ('partner', 'Based on Customer ID')], default='so_name',
        help='You can set here the communication type that will appear on sales orders.'
             'The communication will be given to the customer when they choose the payment method.')

```

## File: models\payment_transaction.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import logging

from datetime import datetime
from dateutil import relativedelta

from odoo import api, fields, models, _, SUPERUSER_ID


_logger = logging.getLogger(__name__)


class PaymentTransaction(models.Model):
    _inherit = 'payment.transaction'

    sale_order_ids = fields.Many2many('sale.order', 'sale_order_transaction_rel', 'transaction_id', 'sale_order_id',
                                      string='Sales Orders', copy=False, readonly=True)
    sale_order_ids_nbr = fields.Integer(compute='_compute_sale_order_ids_nbr', string='# of Sales Orders')

    def _compute_sale_order_reference(self, order):
        self.ensure_one()
        if self.acquirer_id.so_reference_type == 'so_name':
            order_reference = order.name
        else:
            # self.acquirer_id.so_reference_type == 'partner'
            identification_number = order.partner_id.id
            order_reference = '%s/%s' % ('CUST', str(identification_number % 97).rjust(2, '0'))

        invoice_journal = self.env['account.journal'].search([('type', '=', 'sale'), ('company_id', '=', self.env.company.id)], limit=1)
        order_reference = invoice_journal._process_reference_for_sale_order(order_reference)

        return order_reference

    @api.depends('sale_order_ids')
    def _compute_sale_order_ids_nbr(self):
        for trans in self:
            trans.sale_order_ids_nbr = len(trans.sale_order_ids)

    def _set_pending(self, state_message=None):
        """ Override of payment to send the quotations automatically. """
        super(PaymentTransaction, self)._set_pending(state_message=state_message)

        for record in self:
            sales_orders = record.sale_order_ids.filtered(lambda so: so.state in ['draft', 'sent'])
            sales_orders.filtered(lambda so: so.state == 'draft').with_context(tracking_disable=True).write({'state': 'sent'})

            if record.acquirer_id.provider == 'transfer':
                for so in record.sale_order_ids:
                    so.reference = record._compute_sale_order_reference(so)
            # send order confirmation mail
            sales_orders._send_order_confirmation_mail()

    def _check_amount_and_confirm_order(self):
        self.ensure_one()
        for order in self.sale_order_ids.filtered(lambda so: so.state in ('draft', 'sent')):
            if order.currency_id.compare_amounts(self.amount, order.amount_total) == 0:
                order.with_context(send_email=True).action_confirm()
            else:
                _logger.warning(
                    '<%s> transaction AMOUNT MISMATCH for order %s (ID %s): expected %r, got %r',
                    self.acquirer_id.provider,order.name, order.id,
                    order.amount_total, self.amount,
                )
                order.message_post(
                    subject=_("Amount Mismatch (%s)", self.acquirer_id.provider),
                    body=_("The order was not confirmed despite response from the acquirer (%s): order total is %r but acquirer replied with %r.") % (
                        self.acquirer_id.provider,
                        order.amount_total,
                        self.amount,
                    )
                )

    def _set_authorized(self, state_message=None):
        """ Override of payment to confirm the quotations automatically. """
        super()._set_authorized(state_message=state_message)
        sales_orders = self.mapped('sale_order_ids').filtered(lambda so: so.state in ('draft', 'sent'))
        for tx in self:
            tx._check_amount_and_confirm_order()

        # send order confirmation mail
        sales_orders._send_order_confirmation_mail()

    def _log_message_on_linked_documents(self, message):
        """ Override of payment to log a message on the sales orders linked to the transaction.

        Note: self.ensure_one()

        :param str message: The message to be logged
        :return: None
        """
        super()._log_message_on_linked_documents(message)
        for order in self.sale_order_ids:
            order.message_post(body=message)

    def _reconcile_after_done(self):
        """ Override of payment to automatically confirm quotations and generate invoices. """
        draft_orders = self.sale_order_ids.filtered(lambda so: so.state in ('draft', 'sent'))
        for tx in self:
            tx._check_amount_and_confirm_order()
        confirmed_sales_orders = draft_orders.filtered(lambda so: so.state in ('sale', 'done'))
        # send order confirmation mail
        confirmed_sales_orders._send_order_confirmation_mail()
        # invoice the sale orders if needed
        self._invoice_sale_orders()
        res = super()._reconcile_after_done()
        if self.env['ir.config_parameter'].sudo().get_param('sale.automatic_invoice') and any(so.state in ('sale', 'done') for so in self.sale_order_ids):
            self.filtered(lambda t: t.sale_order_ids.filtered(lambda so: so.state in ('sale', 'done')))._send_invoice()
        return res

    def _send_invoice(self):
        default_template = self.env['ir.config_parameter'].sudo().get_param('sale.default_invoice_email_template')
        if not default_template:
            return

        template_id = int(default_template)
        template = self.env['mail.template'].browse(template_id)
        for trans in self:
            trans = trans.with_company(trans.acquirer_id.company_id).with_context(
                company_id=trans.acquirer_id.company_id.id,
            )
            invoice_to_send = trans.invoice_ids.filtered(
                lambda i: not i.is_move_sent and i.state == 'posted' and i._is_ready_to_be_sent()
            )
            invoice_to_send.is_move_sent = True # Mark invoice as sent
            for invoice in invoice_to_send:
                lang = template._render_lang(invoice.ids)[invoice.id]
                model_desc = invoice.with_context(lang=lang).type_name
                invoice.with_context(model_description=model_desc).with_user(
                    SUPERUSER_ID
                ).message_post_with_template(
                    template_id=template_id, email_layout_xmlid='mail.mail_notification_paynow')

    def _cron_send_invoice(self):
        """
            Cron to send invoice that where not ready to be send directly after posting
        """
        if not self.env['ir.config_parameter'].sudo().get_param('sale.automatic_invoice'):
            return

        # No need to retrieve old transactions
        retry_limit_date = datetime.now() - relativedelta.relativedelta(days=2)
        # Retrieve all transactions matching the criteria for post-processing
        self.search([
            ('state', '=', 'done'),
            ('is_post_processed', '=', True),
            ('invoice_ids', 'in', self.env['account.move']._search([
                ('is_move_sent', '=', False),
                ('state', '=', 'posted'),
            ])),
            ('sale_order_ids.state', 'in', ('sale', 'done')),
            ('last_state_change', '>=', retry_limit_date),
        ])._send_invoice()

    def _invoice_sale_orders(self):
        if self.env['ir.config_parameter'].sudo().get_param('sale.automatic_invoice'):
            for trans in self.filtered(lambda t: t.sale_order_ids):
                trans = trans.with_company(trans.acquirer_id.company_id)\
                    .with_context(company_id=trans.acquirer_id.company_id.id)
                confirmed_orders = trans.sale_order_ids.filtered(lambda so: so.state in ('sale', 'done'))
                if confirmed_orders:
                    confirmed_orders._force_lines_to_invoice_policy_order()
                    invoices = confirmed_orders._create_invoices()
                    # Setup access token in advance to avoid serialization failure between
                    # edi postprocessing of invoice and displaying the sale order on the portal
                    for invoice in invoices:
                        invoice._portal_ensure_token()
                    trans.invoice_ids = [(6, 0, invoices.ids)]

    @api.model
    def _compute_reference_prefix(self, provider, separator, **values):
        """ Override of payment to compute the reference prefix based on Sales-specific values.

        If the `values` parameter has an entry with 'sale_order_ids' as key and a list of (4, id, O)
        or (6, 0, ids) X2M command as value, the prefix is computed based on the sales order name(s)
        Otherwise, the computation is delegated to the super method.

        :param str provider: The provider of the acquirer handling the transaction
        :param str separator: The custom separator used to separate data references
        :param dict values: The transaction values used to compute the reference prefix. It should
                            have the structure {'sale_order_ids': [(X2M command), ...], ...}.
        :return: The computed reference prefix if order ids are found, the one of `super` otherwise
        :rtype: str
        """
        command_list = values.get('sale_order_ids')
        if command_list:
            # Extract sales order id(s) from the X2M commands
            order_ids = self._fields['sale_order_ids'].convert_to_cache(command_list, self)
            orders = self.env['sale.order'].browse(order_ids).exists()
            if len(orders) == len(order_ids):  # All ids are valid
                return separator.join(orders.mapped('name'))
        return super()._compute_reference_prefix(provider, separator, **values)

    def action_view_sales_orders(self):
        action = {
            'name': _('Sales Order(s)'),
            'type': 'ir.actions.act_window',
            'res_model': 'sale.order',
            'target': 'current',
        }
        sale_order_ids = self.sale_order_ids.ids
        if len(sale_order_ids) == 1:
            action['res_id'] = sale_order_ids[0]
            action['view_mode'] = 'form'
        else:
            action['view_mode'] = 'tree,form'
            action['domain'] = [('id', 'in', sale_order_ids)]
        return action

```

## File: models\product_product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from datetime import timedelta, time
from odoo import fields, models, _, api
from odoo.tools.float_utils import float_round


class ProductProduct(models.Model):
    _inherit = 'product.product'

    sales_count = fields.Float(compute='_compute_sales_count', string='Sold')

    def _compute_sales_count(self):
        r = {}
        self.sales_count = 0
        if not self.user_has_groups('sales_team.group_sale_salesman'):
            return r
        date_from = fields.Datetime.to_string(fields.datetime.combine(fields.datetime.now() - timedelta(days=365),
                                                                      time.min))

        done_states = self.env['sale.report']._get_done_states()

        domain = [
            ('state', 'in', done_states),
            ('product_id', 'in', self.ids),
            ('date', '>=', date_from),
        ]
        for group in self.env['sale.report'].read_group(domain, ['product_id', 'product_uom_qty'], ['product_id']):
            r[group['product_id'][0]] = group['product_uom_qty']
        for product in self:
            if not product.id:
                product.sales_count = 0.0
                continue
            product.sales_count = float_round(r.get(product.id, 0), precision_rounding=product.uom_id.rounding)
        return r

    @api.onchange('type')
    def _onchange_type(self):
        if self._origin and self.sales_count > 0:
            return {'warning': {
                'title': _("Warning"),
                'message': _("You cannot change the product's type because it is already used in sales orders.")
            }}

    def action_view_sales(self):
        action = self.env["ir.actions.actions"]._for_xml_id("sale.report_all_channels_sales_action")
        action['domain'] = [('product_id', 'in', self.ids)]
        action['context'] = {
            'pivot_measures': ['product_uom_qty'],
            'active_id': self._context.get('active_id'),
            'search_default_Sales': 1,
            'active_model': 'sale.report',
            'search_default_filter_order_date': 1,
        }
        return action

    def _get_invoice_policy(self):
        return self.invoice_policy

    def _get_combination_info_variant(self, add_qty=1, pricelist=False, parent_combination=False):
        """Return the variant info based on its combination.
        See `_get_combination_info` for more information.
        """
        self.ensure_one()
        return self.product_tmpl_id._get_combination_info(self.product_template_attribute_value_ids, self.id, add_qty, pricelist, parent_combination)

    def _filter_to_unlink(self):
        domain = [('product_id', 'in', self.ids)]
        lines = self.env['sale.order.line'].read_group(domain, ['product_id'], ['product_id'])
        linked_product_ids = [group['product_id'][0] for group in lines]
        return super(ProductProduct, self - self.browse(linked_product_ids))._filter_to_unlink()


class ProductAttributeCustomValue(models.Model):
    _inherit = "product.attribute.custom.value"

    sale_order_line_id = fields.Many2one('sale.order.line', string="Sales Order Line", required=True, ondelete='cascade')

    _sql_constraints = [
        ('sol_custom_value_unique', 'unique(custom_product_template_attribute_value_id, sale_order_line_id)', "Only one Custom Value is allowed per Attribute Value per Sales Order Line.")
    ]

class ProductPackaging(models.Model):
    _inherit = 'product.packaging'

    sales = fields.Boolean("Sales", default=True, help="If true, the packaging can be used for sales orders")

```

## File: models\product_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json
import logging

from odoo import api, fields, models, _
from odoo.addons.base.models.res_partner import WARNING_MESSAGE, WARNING_HELP
from odoo.exceptions import ValidationError
from odoo.tools.float_utils import float_round

_logger = logging.getLogger(__name__)


class ProductTemplate(models.Model):
    _inherit = 'product.template'

    service_type = fields.Selection([('manual', 'Manually set quantities on order')], string='Track Service',
        help="Manually set quantities on order: Invoice based on the manually entered quantity, without creating an analytic account.\n"
             "Timesheets on contract: Invoice based on the tracked hours on the related timesheet.\n"
             "Create a task and track hours: Create a task on the sales order validation and track the work hours.",
        default='manual')
    sale_line_warn = fields.Selection(WARNING_MESSAGE, 'Sales Order Line', help=WARNING_HELP, required=True, default="no-message")
    sale_line_warn_msg = fields.Text('Message for Sales Order Line')
    expense_policy = fields.Selection(
        [('no', 'No'), ('cost', 'At cost'), ('sales_price', 'Sales price')],
        string='Re-Invoice Expenses',
        default='no',
        help="Expenses and vendor bills can be re-invoiced to a customer."
             "With this option, a validated expense can be re-invoice to a customer at its cost or sales price.")
    visible_expense_policy = fields.Boolean("Re-Invoice Policy visible", compute='_compute_visible_expense_policy')
    sales_count = fields.Float(compute='_compute_sales_count', string='Sold')
    visible_qty_configurator = fields.Boolean("Quantity visible in configurator", compute='_compute_visible_qty_configurator')
    invoice_policy = fields.Selection([
        ('order', 'Ordered quantities'),
        ('delivery', 'Delivered quantities')], string='Invoicing Policy',
        help='Ordered Quantity: Invoice quantities ordered by the customer.\n'
             'Delivered Quantity: Invoice quantities delivered to the customer.',
        default='order')

    def _compute_visible_qty_configurator(self):
        for product_template in self:
            product_template.visible_qty_configurator = True

    @api.depends('name')
    def _compute_visible_expense_policy(self):
        visibility = self.user_has_groups('analytic.group_analytic_accounting')
        for product_template in self:
            product_template.visible_expense_policy = visibility


    @api.onchange('sale_ok')
    def _change_sale_ok(self):
        if not self.sale_ok:
            self.expense_policy = 'no'

    @api.depends('product_variant_ids.sales_count')
    def _compute_sales_count(self):
        for product in self:
            product.sales_count = float_round(sum([p.sales_count for p in product.with_context(active_test=False).product_variant_ids]), precision_rounding=product.uom_id.rounding)


    @api.constrains('company_id')
    def _check_sale_product_company(self):
        """Ensure the product is not being restricted to a single company while
        having been sold in another one in the past, as this could cause issues."""
        target_company = self.company_id
        if target_company:  # don't prevent writing `False`, should always work
            product_data = self.env['product.product'].sudo().with_context(active_test=False).search_read([('product_tmpl_id', 'in', self.ids)], fields=['id'])
            product_ids = list(map(lambda p: p['id'], product_data))
            so_lines = self.env['sale.order.line'].sudo().search_read([('product_id', 'in', product_ids), ('company_id', '!=', target_company.id)], fields=['id', 'product_id'])
            used_products = list(map(lambda sol: sol['product_id'][1], so_lines))
            if so_lines:
                raise ValidationError(_('The following products cannot be restricted to the company'
                                        ' %s because they have already been used in quotations or '
                                        'sales orders in another company:\n%s\n'
                                        'You can archive these products and recreate them '
                                        'with your company restriction instead, or leave them as '
                                        'shared product.') % (target_company.name, ', '.join(used_products)))

    def action_view_sales(self):
        action = self.env["ir.actions.actions"]._for_xml_id("sale.report_all_channels_sales_action")
        action['domain'] = [('product_tmpl_id', 'in', self.ids)]
        action['context'] = {
            'pivot_measures': ['product_uom_qty'],
            'active_id': self._context.get('active_id'),
            'active_model': 'sale.report',
            'search_default_Sales': 1,
            'search_default_filter_order_date': 1,
        }
        return action

    def create_product_variant(self, product_template_attribute_value_ids):
        """ Create if necessary and possible and return the id of the product
        variant matching the given combination for this template.

        Note AWA: Known "exploit" issues with this method:

        - This method could be used by an unauthenticated user to generate a
            lot of useless variants. Unfortunately, after discussing the
            matter with ODO, there's no easy and user-friendly way to block
            that behavior.

            We would have to use captcha/server actions to clean/... that
            are all not user-friendly/overkill mechanisms.

        - This method could be used to try to guess what product variant ids
            are created in the system and what product template ids are
            configured as "dynamic", but that does not seem like a big deal.

        The error messages are identical on purpose to avoid giving too much
        information to a potential attacker:
            - returning 0 when failing
            - returning the variant id whether it already existed or not

        :param product_template_attribute_value_ids: the combination for which
            to get or create variant
        :type product_template_attribute_value_ids: json encoded list of id
            of `product.template.attribute.value`

        :return: id of the product variant matching the combination or 0
        :rtype: int
        """
        combination = self.env['product.template.attribute.value'] \
            .browse(json.loads(product_template_attribute_value_ids))

        return self._create_product_variant(combination, log_warning=True).id or 0

    @api.onchange('type')
    def _onchange_type(self):
        """ Force values to stay consistent with integrity constraints """
        res = super(ProductTemplate, self)._onchange_type()
        if self.type == 'consu':
            if not self.invoice_policy:
                self.invoice_policy = 'order'
            self.service_type = 'manual'
        if self._origin and self.sales_count > 0:
            res['warning'] = {
                'title': _("Warning"),
                'message': _("You cannot change the product's type because it is already used in sales orders.")
            }
        return res

    @api.model
    def get_import_templates(self):
        res = super(ProductTemplate, self).get_import_templates()
        if self.env.context.get('sale_multi_pricelist_product_template'):
            if self.user_has_groups('product.group_sale_pricelist'):
                return [{
                    'label': _('Import Template for Products'),
                    'template': '/product/static/xls/product_template.xls'
                }]
        return res

    def _get_combination_info(self, combination=False, product_id=False, add_qty=1, pricelist=False, parent_combination=False, only_template=False):
        """ Return info about a given combination.

        Note: this method does not take into account whether the combination is
        actually possible.

        :param combination: recordset of `product.template.attribute.value`

        :param product_id: id of a `product.product`. If no `combination`
            is set, the method will try to load the variant `product_id` if
            it exists instead of finding a variant based on the combination.

            If there is no combination, that means we definitely want a
            variant and not something that will have no_variant set.

        :param add_qty: float with the quantity for which to get the info,
            indeed some pricelist rules might depend on it.

        :param pricelist: `product.pricelist` the pricelist to use
            (can be none, eg. from SO if no partner and no pricelist selected)

        :param parent_combination: if no combination and no product_id are
            given, it will try to find the first possible combination, taking
            into account parent_combination (if set) for the exclusion rules.

        :param only_template: boolean, if set to True, get the info for the
            template only: ignore combination and don't try to find variant

        :return: dict with product/combination info:

            - product_id: the variant id matching the combination (if it exists)

            - product_template_id: the current template id

            - display_name: the name of the combination

            - price: the computed price of the combination, take the catalog
                price if no pricelist is given

            - list_price: the catalog price of the combination, but this is
                not the "real" list_price, it has price_extra included (so
                it's actually more closely related to `lst_price`), and it
                is converted to the pricelist currency (if given)

            - has_discounted_price: True if the pricelist discount policy says
                the price does not include the discount and there is actually a
                discount applied (price < list_price), else False
        """
        self.ensure_one()
        # get the name before the change of context to benefit from prefetch
        display_name = self.display_name

        display_image = True
        quantity = self.env.context.get('quantity', add_qty)
        context = dict(self.env.context, quantity=quantity, pricelist=pricelist.id if pricelist else False)
        product_template = self.with_context(context)

        combination = combination or product_template.env['product.template.attribute.value']

        if not product_id and not combination and not only_template:
            combination = product_template._get_first_possible_combination(parent_combination)

        if only_template:
            product = product_template.env['product.product']
        elif product_id and not combination:
            product = product_template.env['product.product'].browse(product_id)
        else:
            product = product_template._get_variant_for_combination(combination)

        if product:
            # We need to add the price_extra for the attributes that are not
            # in the variant, typically those of type no_variant, but it is
            # possible that a no_variant attribute is still in a variant if
            # the type of the attribute has been changed after creation.
            no_variant_attributes_price_extra = [
                ptav.price_extra for ptav in combination.filtered(
                    lambda ptav:
                        ptav.price_extra and
                        ptav not in product.product_template_attribute_value_ids
                )
            ]
            if no_variant_attributes_price_extra:
                product = product.with_context(
                    no_variant_attributes_price_extra=tuple(no_variant_attributes_price_extra)
                )
            list_price = product.price_compute('list_price')[product.id]
            price = product.price if pricelist else list_price
            display_image = bool(product.image_128)
            display_name = product.display_name
            price_extra = (product.price_extra or 0.0 ) + (sum(no_variant_attributes_price_extra) or 0.0)
        else:
            current_attributes_price_extra = [v.price_extra or 0.0 for v in combination]
            product_template = product_template.with_context(current_attributes_price_extra=current_attributes_price_extra)
            price_extra = sum(current_attributes_price_extra)
            list_price = product_template.price_compute('list_price')[product_template.id]
            price = product_template.price if pricelist else list_price
            display_image = bool(product_template.image_128)

            combination_name = combination._get_combination_name()
            if combination_name:
                display_name = "%s (%s)" % (display_name, combination_name)

        if pricelist and pricelist.currency_id != product_template.currency_id:
            list_price = product_template.currency_id._convert(
                list_price, pricelist.currency_id, product_template._get_current_company(pricelist=pricelist),
                fields.Date.today()
            )
            price_extra = product_template.currency_id._convert(
                price_extra, pricelist.currency_id, product_template._get_current_company(pricelist=pricelist),
                fields.Date.today()
            )

        price_without_discount = list_price if pricelist and pricelist.discount_policy == 'without_discount' else price
        has_discounted_price = (pricelist or product_template).currency_id.compare_amounts(price_without_discount, price) == 1

        return {
            'product_id': product.id,
            'product_template_id': product_template.id,
            'display_name': display_name,
            'display_image': display_image,
            'price': price,
            'list_price': list_price,
            'price_extra': price_extra,
            'has_discounted_price': has_discounted_price,
        }

    def _can_be_added_to_cart(self):
        """
        Pre-check to `_is_add_to_cart_possible` to know if product can be sold.
        """
        return self.sale_ok

    def _is_add_to_cart_possible(self, parent_combination=None):
        """
        It's possible to add to cart (potentially after configuration) if
        there is at least one possible combination.

        :param parent_combination: the combination from which `self` is an
            optional or accessory product.
        :type parent_combination: recordset `product.template.attribute.value`

        :return: True if it's possible to add to cart, else False
        :rtype: bool
        """
        self.ensure_one()
        if not self.active or not self._can_be_added_to_cart():
            # for performance: avoid calling `_get_possible_combinations`
            return False
        return next(self._get_possible_combinations(parent_combination), False) is not False

    def _get_current_company_fallback(self, **kwargs):
        """Override: if a pricelist is given, fallback to the company of the
        pricelist if it is set, otherwise use the one from parent method."""
        res = super(ProductTemplate, self)._get_current_company_fallback(**kwargs)
        pricelist = kwargs.get('pricelist')
        return pricelist and pricelist.company_id or res

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import base64

from odoo import api, fields, models, _
from odoo.modules.module import get_module_resource


class ResCompany(models.Model):
    _inherit = "res.company"

    portal_confirmation_sign = fields.Boolean(string='Online Signature', default=True)
    portal_confirmation_pay = fields.Boolean(string='Online Payment')
    quotation_validity_days = fields.Integer(default=30, string="Default Quotation Validity (Days)")

    # sale quotation onboarding
    sale_quotation_onboarding_state = fields.Selection([('not_done', "Not done"), ('just_done', "Just done"), ('done', "Done"), ('closed', "Closed")], string="State of the sale onboarding panel", default='not_done')
    sale_onboarding_order_confirmation_state = fields.Selection([('not_done', "Not done"), ('just_done', "Just done"), ('done', "Done")], string="State of the onboarding confirmation order step", default='not_done')
    sale_onboarding_sample_quotation_state = fields.Selection([('not_done', "Not done"), ('just_done', "Just done"), ('done', "Done")], string="State of the onboarding sample quotation step", default='not_done')

    sale_onboarding_payment_method = fields.Selection([
        ('digital_signature', 'Sign online'),
        ('paypal', 'PayPal'),
        ('stripe', 'Stripe'),
        ('other', 'Pay with another payment acquirer'),
        ('manual', 'Manual Payment'),
    ], string="Sale onboarding selected payment method")

    @api.model
    def action_close_sale_quotation_onboarding(self):
        """ Mark the onboarding panel as closed. """
        self.env.company.sale_quotation_onboarding_state = 'closed'

    @api.model
    def action_open_sale_onboarding_payment_acquirer(self):
        """ Called by onboarding panel above the quotation list."""
        self.env.company.get_chart_of_accounts_or_fail()
        action = self.env["ir.actions.actions"]._for_xml_id("sale.action_open_sale_onboarding_payment_acquirer_wizard")
        return action

    def _mark_payment_onboarding_step_as_done(self):
        """ Override of payment to mark the sale onboarding step as done.

        The payment onboarding step of Sales is only marked as done if it was started from Sales.
        This prevents incorrectly marking the step as done if another module's payment onboarding
        step was marked as done.

        :return: None
        """
        super()._mark_payment_onboarding_step_as_done()
        if self.sale_onboarding_payment_method:  # The onboarding step was started from Sales
            self.set_onboarding_step_done('sale_onboarding_order_confirmation_state')

    def _get_sample_sales_order(self):
        """ Get a sample quotation or create one if it does not exist. """
        # use current user as partner
        partner = self.env.user.partner_id
        company_id = self.env.company.id
        # is there already one?
        sample_sales_order = self.env['sale.order'].search(
            [('company_id', '=', company_id), ('partner_id', '=', partner.id),
             ('state', '=', 'draft')], limit=1)
        if len(sample_sales_order) == 0:
            sample_sales_order = self.env['sale.order'].create({
                'partner_id': partner.id
            })
            # take any existing product or create one
            product = self.env['product.product'].search([], limit=1)
            if len(product) == 0:
                default_image_path = get_module_resource('product', 'static/img', 'product_product_13-image.png')
                product = self.env['product.product'].create({
                    'name': _('Sample Product'),
                    'active': False,
                    'image_1920': base64.b64encode(open(default_image_path, 'rb').read())
                })
                product.product_tmpl_id.write({'active': False})
            self.env['sale.order.line'].create({
                'name': _('Sample Order Line'),
                'product_id': product.id,
                'product_uom_qty': 10,
                'price_unit': 123,
                'order_id': sample_sales_order.id,
                'company_id': sample_sales_order.company_id.id,
            })
        return sample_sales_order

    @api.model
    def action_open_sale_onboarding_sample_quotation(self):
        """ Onboarding step for sending a sample quotation. Open a window to compose an email,
            with the edi_invoice_template message loaded by default. """
        sample_sales_order = self._get_sample_sales_order()
        template = self.env.ref('sale.email_template_edi_sale', False)

        message_composer = self.env['mail.compose.message'].with_context(
            default_use_template=bool(template),
            mark_so_as_sent=True,
            custom_layout='mail.mail_notification_paynow',
            proforma=self.env.context.get('proforma', False),
            force_email=True, mail_notify_author=True
        ).create({
            'res_id': sample_sales_order.id,
            'template_id': template and template.id or False,
            'model': 'sale.order',
            'composition_mode': 'comment'})

        # Simulate the onchange (like trigger in form the view)
        update_values = message_composer._onchange_template_id(template.id, 'comment', 'sale.order', sample_sales_order.id)['value']
        message_composer.write(update_values)

        message_composer._action_send_mail()

        self.set_onboarding_step_done('sale_onboarding_sample_quotation_state')

        self.action_close_sale_quotation_onboarding()

        action = self.env["ir.actions.actions"]._for_xml_id("sale.action_orders")
        action.update({
            'views': [[self.env.ref('sale.view_order_form').id, 'form']],
            'view_mode': 'form',
            'target': 'main',
        })
        return action

    def get_and_update_sale_quotation_onboarding_state(self):
        """ This method is called on the controller rendering method and ensures that the animations
            are displayed only one time. """
        steps = [
            'base_onboarding_company_state',
            'account_onboarding_invoice_layout_state',
            'sale_onboarding_order_confirmation_state',
            'sale_onboarding_sample_quotation_state',
        ]
        return self.get_and_update_onbarding_state('sale_quotation_onboarding_state', steps)

    _sql_constraints = [('check_quotation_validity_days', 'CHECK(quotation_validity_days > 0)', 'Quotation Validity is required and must be greater than 0.')]

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    group_auto_done_setting = fields.Boolean("Lock Confirmed Sales", implied_group='sale.group_auto_done_setting')
    module_sale_margin = fields.Boolean("Margins")
    quotation_validity_days = fields.Integer(related='company_id.quotation_validity_days', string="Default Quotation Validity (Days)", readonly=False)
    use_quotation_validity_days = fields.Boolean("Default Quotation Validity", config_parameter='sale.use_quotation_validity_days')
    group_warning_sale = fields.Boolean("Sale Order Warnings", implied_group='sale.group_warning_sale')
    portal_confirmation_sign = fields.Boolean(related='company_id.portal_confirmation_sign', string='Online Signature', readonly=False)
    portal_confirmation_pay = fields.Boolean(related='company_id.portal_confirmation_pay', string='Online Payment', readonly=False)
    group_sale_delivery_address = fields.Boolean("Customer Addresses", implied_group='sale.group_delivery_invoice_address')
    group_proforma_sales = fields.Boolean(string="Pro-Forma Invoice", implied_group='sale.group_proforma_sales',
        help="Allows you to send pro-forma invoice.")
    default_invoice_policy = fields.Selection([
        ('order', 'Invoice what is ordered'),
        ('delivery', 'Invoice what is delivered')
        ], 'Invoicing Policy',
        default='order',
        default_model='product.template')
    deposit_default_product_id = fields.Many2one(
        'product.product',
        'Deposit Product',
        domain="[('type', '=', 'service')]",
        config_parameter='sale.default_deposit_product_id',
        help='Default product used for payment advances')

    auth_signup_uninvited = fields.Selection([
        ('b2b', 'On invitation'),
        ('b2c', 'Free sign up'),
    ], string='Customer Account', default='b2b', config_parameter='auth_signup.invitation_scope')

    module_delivery = fields.Boolean("Delivery Methods")
    module_delivery_dhl = fields.Boolean("DHL Express Connector")
    module_delivery_fedex = fields.Boolean("FedEx Connector")
    module_delivery_ups = fields.Boolean("UPS Connector")
    module_delivery_usps = fields.Boolean("USPS Connector")
    module_delivery_bpost = fields.Boolean("bpost Connector")
    module_delivery_easypost = fields.Boolean("Easypost Connector")

    module_product_email_template = fields.Boolean("Specific Email")
    module_sale_coupon = fields.Boolean("Coupons & Promotions")
    module_sale_amazon = fields.Boolean("Amazon Sync")

    automatic_invoice = fields.Boolean(
        string="Automatic Invoice",
        help="The invoice is generated automatically and available in the customer portal when the "
             "transaction is confirmed by the payment acquirer.\nThe invoice is marked as paid and "
             "the payment is registered in the payment journal defined in the configuration of the "
             "payment acquirer.\nThis mode is advised if you issue the final invoice at the order "
             "and not after the delivery.",
        config_parameter='sale.automatic_invoice',
    )
    invoice_mail_template_id = fields.Many2one(
        comodel_name='mail.template',
        string='Invoice Email Template',
        domain="[('model', '=', 'account.move')]",
        config_parameter='sale.default_invoice_email_template',
        default=lambda self: self.env.ref('account.email_template_edi_invoice', False)
    )
    confirmation_mail_template_id = fields.Many2one(
        comodel_name='mail.template',
        string='Confirmation Email Template',
        domain="[('model', '=', 'sale.order')]",
        config_parameter='sale.default_confirmation_template',
        help="Email sent to the customer once the order is paid."
    )

    def set_values(self):
        super(ResConfigSettings, self).set_values()
        if self.default_invoice_policy != 'order':
            self.env['ir.config_parameter'].set_param('sale.automatic_invoice', False)

        send_invoice_cron = self.env.ref('sale.send_invoice_cron', raise_if_not_found=False)
        if send_invoice_cron:
            send_invoice_cron.active = self.automatic_invoice

    @api.onchange('use_quotation_validity_days')
    def _onchange_use_quotation_validity_days(self):
        if self.quotation_validity_days <= 0:
            self.quotation_validity_days = self.env['res.company'].default_get(['quotation_validity_days'])['quotation_validity_days']

    @api.onchange('quotation_validity_days')
    def _onchange_quotation_validity_days(self):
        if self.quotation_validity_days <= 0:
            self.quotation_validity_days = self.env['res.company'].default_get(['quotation_validity_days'])['quotation_validity_days']
            return {
                'warning': {'title': "Warning", 'message': "Quotation Validity is required and must be greater than 0."},
            }

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models
from odoo.addons.base.models.res_partner import WARNING_MESSAGE, WARNING_HELP


class ResPartner(models.Model):
    _inherit = 'res.partner'

    sale_order_count = fields.Integer(compute='_compute_sale_order_count', string='Sale Order Count')
    sale_order_ids = fields.One2many('sale.order', 'partner_id', 'Sales Order')
    sale_warn = fields.Selection(WARNING_MESSAGE, 'Sales Warnings', default='no-message', help=WARNING_HELP)
    sale_warn_msg = fields.Text('Message for Sales Order')

    def _compute_sale_order_count(self):
        # retrieve all children partners and prefetch 'parent_id' on them
        all_partners = self.with_context(active_test=False).search([('id', 'child_of', self.ids)])
        all_partners.read(['parent_id'])

        sale_order_groups = self.env['sale.order'].read_group(
            domain=[('partner_id', 'in', all_partners.ids)],
            fields=['partner_id'], groupby=['partner_id']
        )
        partners = self.browse()
        for group in sale_order_groups:
            partner = self.browse(group['partner_id'][0])
            while partner:
                if partner in self:
                    partner.sale_order_count += group['partner_id_count']
                    partners |= partner
                partner = partner.parent_id
        (self - partners).sale_order_count = 0

    def can_edit_vat(self):
        ''' Can't edit `vat` if there is (non draft) issued SO. '''
        can_edit_vat = super(ResPartner, self).can_edit_vat()
        if not can_edit_vat:
            return can_edit_vat
        SaleOrder = self.env['sale.order']
        has_so = SaleOrder.sudo().search([
            ('partner_id', 'child_of', self.commercial_partner_id.id),
            ('state', 'in', ['sent', 'sale', 'done'])
        ], limit=1)
        return can_edit_vat and not bool(has_so)

    def action_view_sale_order(self):
        action = self.env['ir.actions.act_window']._for_xml_id('sale.act_res_partner_2_sale_order')
        all_child = self.with_context(active_test=False).search([('id', 'child_of', self.ids)])
        action["domain"] = [("partner_id", "in", all_child.ids)]
        return action

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import datetime, timedelta
from itertools import groupby
import json

from odoo import api, fields, models, SUPERUSER_ID, _
from odoo.exceptions import AccessError, UserError, ValidationError
from odoo.osv import expression
from odoo.tools import float_is_zero, html_keep_url, is_html_empty

from odoo.addons.payment import utils as payment_utils

class SaleOrder(models.Model):
    _name = "sale.order"
    _inherit = ['portal.mixin', 'mail.thread', 'mail.activity.mixin', 'utm.mixin']
    _description = "Sales Order"
    _order = 'date_order desc, id desc'
    _check_company_auto = True

    def _default_validity_date(self):
        if self.env['ir.config_parameter'].sudo().get_param('sale.use_quotation_validity_days'):
            days = self.env.company.quotation_validity_days
            if days > 0:
                return fields.Date.to_string(datetime.now() + timedelta(days))
        return False

    def _get_default_require_signature(self):
        return self.env.company.portal_confirmation_sign

    def _get_default_require_payment(self):
        return self.env.company.portal_confirmation_pay

    def _compute_amount_total_without_delivery(self):
        self.ensure_one()
        return self.amount_total

    @api.depends('order_line.price_total')
    def _amount_all(self):
        """
        Compute the total amounts of the SO.
        """
        for order in self:
            amount_untaxed = amount_tax = 0.0
            for line in order.order_line:
                amount_untaxed += line.price_subtotal
                amount_tax += line.price_tax
            order.update({
                'amount_untaxed': amount_untaxed,
                'amount_tax': amount_tax,
                'amount_total': amount_untaxed + amount_tax,
            })

    @api.depends('order_line.invoice_lines')
    def _get_invoiced(self):
        # The invoice_ids are obtained thanks to the invoice lines of the SO
        # lines, and we also search for possible refunds created directly from
        # existing invoices. This is necessary since such a refund is not
        # directly linked to the SO.
        for order in self:
            invoices = order.order_line.invoice_lines.move_id.filtered(lambda r: r.move_type in ('out_invoice', 'out_refund'))
            order.invoice_ids = invoices
            order.invoice_count = len(invoices)

    @api.depends('state', 'order_line.invoice_status')
    def _get_invoice_status(self):
        """
        Compute the invoice status of a SO. Possible statuses:
        - no: if the SO is not in status 'sale' or 'done', we consider that there is nothing to
          invoice. This is also the default value if the conditions of no other status is met.
        - to invoice: if any SO line is 'to invoice', the whole SO is 'to invoice'
        - invoiced: if all SO lines are invoiced, the SO is invoiced.
        - upselling: if all SO lines are invoiced or upselling, the status is upselling.
        """
        unconfirmed_orders = self.filtered(lambda so: so.state not in ['sale', 'done'])
        unconfirmed_orders.invoice_status = 'no'
        confirmed_orders = self - unconfirmed_orders
        if not confirmed_orders:
            return
        line_invoice_status_all = [
            (d['order_id'][0], d['invoice_status'])
            for d in self.env['sale.order.line'].read_group([
                    ('order_id', 'in', confirmed_orders.ids),
                    ('is_downpayment', '=', False),
                    ('display_type', '=', False),
                ],
                ['order_id', 'invoice_status'],
                ['order_id', 'invoice_status'], lazy=False)]
        for order in confirmed_orders:
            line_invoice_status = [d[1] for d in line_invoice_status_all if d[0] == order.id]
            if order.state not in ('sale', 'done'):
                order.invoice_status = 'no'
            elif any(invoice_status == 'to invoice' for invoice_status in line_invoice_status):
                order.invoice_status = 'to invoice'
            elif line_invoice_status and all(invoice_status == 'invoiced' for invoice_status in line_invoice_status):
                order.invoice_status = 'invoiced'
            elif line_invoice_status and all(invoice_status in ('invoiced', 'upselling') for invoice_status in line_invoice_status):
                order.invoice_status = 'upselling'
            else:
                order.invoice_status = 'no'

    @api.model
    def get_empty_list_help(self, help):
        self = self.with_context(
            empty_list_help_document_name=_("sale order"),
        )
        return super(SaleOrder, self).get_empty_list_help(help)

    @api.model
    def _default_note_url(self):
        return self.env.company.get_base_url()

    @api.model
    def _default_note(self):
        use_invoice_terms = self.env['ir.config_parameter'].sudo().get_param('account.use_invoice_terms')
        if use_invoice_terms and self.env.company.terms_type == "html":
            baseurl = html_keep_url(self._default_note_url() + '/terms')
            context = {'lang': self.partner_id.lang or self.env.user.lang}
            note = _('Terms & Conditions: %s', baseurl)
            del context
            return note
        return use_invoice_terms and self.env.company.invoice_terms or ''

    @api.model
    def _get_default_team(self):
        return self.env['crm.team']._get_default_team_id()

    @api.onchange('fiscal_position_id', 'company_id')
    def _compute_tax_id(self):
        """
        Trigger the recompute of the taxes if the fiscal position is changed on the SO.
        """
        for order in self:
            order.order_line._compute_tax_id()

    def _search_invoice_ids(self, operator, value):
        if operator == 'in' and value:
            self.env.cr.execute("""
                SELECT array_agg(so.id)
                    FROM sale_order so
                    JOIN sale_order_line sol ON sol.order_id = so.id
                    JOIN sale_order_line_invoice_rel soli_rel ON soli_rel.order_line_id = sol.id
                    JOIN account_move_line aml ON aml.id = soli_rel.invoice_line_id
                    JOIN account_move am ON am.id = aml.move_id
                WHERE
                    am.move_type in ('out_invoice', 'out_refund') AND
                    am.id = ANY(%s)
            """, (list(value),))
            so_ids = self.env.cr.fetchone()[0] or []
            return [('id', 'in', so_ids)]
        elif operator == '=' and not value:
            # special case for [('invoice_ids', '=', False)], i.e. "Invoices is not set"
            #
            # We cannot just search [('order_line.invoice_lines', '=', False)]
            # because it returns orders with uninvoiced lines, which is not
            # same "Invoices is not set" (some lines may have invoices and some
            # doesn't)
            #
            # A solution is making inverted search first ("orders with invoiced
            # lines") and then invert results ("get all other orders")
            #
            # Domain below returns subset of ('order_line.invoice_lines', '!=', False)
            order_ids = self._search([
                ('order_line.invoice_lines.move_id.move_type', 'in', ('out_invoice', 'out_refund'))
            ])
            return [('id', 'not in', order_ids)]
        return ['&', ('order_line.invoice_lines.move_id.move_type', 'in', ('out_invoice', 'out_refund')), ('order_line.invoice_lines.move_id', operator, value)]

    name = fields.Char(string='Order Reference', required=True, copy=False, readonly=True, states={'draft': [('readonly', False)]}, index=True, default=lambda self: _('New'))
    origin = fields.Char(string='Source Document', help="Reference of the document that generated this sales order request.")
    client_order_ref = fields.Char(string='Customer Reference', copy=False)
    reference = fields.Char(string='Payment Ref.', copy=False,
        help='The payment communication of this sale order.')
    state = fields.Selection([
        ('draft', 'Quotation'),
        ('sent', 'Quotation Sent'),
        ('sale', 'Sales Order'),
        ('done', 'Locked'),
        ('cancel', 'Cancelled'),
        ], string='Status', readonly=True, copy=False, index=True, tracking=3, default='draft')
    date_order = fields.Datetime(string='Order Date', required=True, readonly=True, index=True, states={'draft': [('readonly', False)], 'sent': [('readonly', False)]}, copy=False, default=fields.Datetime.now, help="Creation date of draft/sent orders,\nConfirmation date of confirmed orders.")
    validity_date = fields.Date(string='Expiration', readonly=True, copy=False, states={'draft': [('readonly', False)], 'sent': [('readonly', False)]},
                                default=_default_validity_date)
    is_expired = fields.Boolean(compute='_compute_is_expired', string="Is expired")
    require_signature = fields.Boolean('Online Signature', default=_get_default_require_signature, readonly=True,
        states={'draft': [('readonly', False)], 'sent': [('readonly', False)]},
        help='Request a online signature to the customer in order to confirm orders automatically.')
    require_payment = fields.Boolean('Online Payment', default=_get_default_require_payment, readonly=True,
        states={'draft': [('readonly', False)], 'sent': [('readonly', False)]},
        help='Request an online payment to the customer in order to confirm orders automatically.')
    create_date = fields.Datetime(string='Creation Date', readonly=True, index=True, help="Date on which sales order is created.")

    user_id = fields.Many2one(
        'res.users', string='Salesperson', index=True, tracking=2, default=lambda self: self.env.user,
        domain=lambda self: "[('groups_id', '=', {}), ('share', '=', False), ('company_ids', '=', company_id)]".format(
            self.env.ref("sales_team.group_sale_salesman").id
        ),)
    partner_id = fields.Many2one(
        'res.partner', string='Customer', readonly=True,
        states={'draft': [('readonly', False)], 'sent': [('readonly', False)]},
        required=True, change_default=True, index=True, tracking=1,
        domain="[('type', '!=', 'private'), ('company_id', 'in', (False, company_id))]",)
    partner_invoice_id = fields.Many2one(
        'res.partner', string='Invoice Address',
        readonly=True, required=True,
        states={'draft': [('readonly', False)], 'sent': [('readonly', False)], 'sale': [('readonly', False)]},
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]",)
    partner_shipping_id = fields.Many2one(
        'res.partner', string='Delivery Address', readonly=True, required=True,
        states={'draft': [('readonly', False)], 'sent': [('readonly', False)], 'sale': [('readonly', False)]},
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]",)

    pricelist_id = fields.Many2one(
        'product.pricelist', string='Pricelist', check_company=True,  # Unrequired company
        required=True, readonly=True, states={'draft': [('readonly', False)], 'sent': [('readonly', False)]},
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]", tracking=1,
        help="If you change the pricelist, only newly added lines will be affected.")
    currency_id = fields.Many2one(related='pricelist_id.currency_id', depends=["pricelist_id"], store=True, ondelete="restrict")
    analytic_account_id = fields.Many2one(
        'account.analytic.account', 'Analytic Account',
        compute='_compute_analytic_account_id', store=True,
        readonly=False, copy=False, check_company=True,  # Unrequired company
        states={'sale': [('readonly', True)], 'done': [('readonly', True)], 'cancel': [('readonly', True)]},
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]",
        help="The analytic account related to a sales order.")

    order_line = fields.One2many('sale.order.line', 'order_id', string='Order Lines', states={'cancel': [('readonly', True)], 'done': [('readonly', True)]}, copy=True, auto_join=True)

    invoice_count = fields.Integer(string='Invoice Count', compute='_get_invoiced')
    invoice_ids = fields.Many2many("account.move", string='Invoices', compute="_get_invoiced", copy=False, search="_search_invoice_ids")
    invoice_status = fields.Selection([
        ('upselling', 'Upselling Opportunity'),
        ('invoiced', 'Fully Invoiced'),
        ('to invoice', 'To Invoice'),
        ('no', 'Nothing to Invoice')
        ], string='Invoice Status', compute='_get_invoice_status', store=True)

    note = fields.Html('Terms and conditions', default=_default_note)
    terms_type = fields.Selection(related='company_id.terms_type')

    amount_untaxed = fields.Monetary(string='Untaxed Amount', store=True, compute='_amount_all', tracking=5)
    tax_totals_json = fields.Char(compute='_compute_tax_totals_json')
    amount_tax = fields.Monetary(string='Taxes', store=True, compute='_amount_all')
    amount_total = fields.Monetary(string='Total', store=True, compute='_amount_all', tracking=4)
    currency_rate = fields.Float("Currency Rate", compute='_compute_currency_rate', store=True, digits=0, help='The rate of the currency to the currency of rate 1 applicable at the date of the order')

    payment_term_id = fields.Many2one(
        'account.payment.term', string='Payment Terms', check_company=True,  # Unrequired company
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]",)
    fiscal_position_id = fields.Many2one(
        'account.fiscal.position', string='Fiscal Position',
        domain="[('company_id', '=', company_id)]", check_company=True,
        help="Fiscal positions are used to adapt taxes and accounts for particular customers or sales orders/invoices."
        "The default value comes from the customer.")
    tax_country_id = fields.Many2one(
        comodel_name='res.country',
        compute='_compute_tax_country_id',
        # Avoid access error on fiscal position when reading a sale order with company != user.company_ids
        compute_sudo=True,
        help="Technical field to filter the available taxes depending on the fiscal country and fiscal position.")
    company_id = fields.Many2one('res.company', 'Company', required=True, index=True, default=lambda self: self.env.company)
    team_id = fields.Many2one(
        'crm.team', 'Sales Team',
        ondelete="set null", tracking=True,
        change_default=True, default=_get_default_team, check_company=True,  # Unrequired company
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")

    signature = fields.Image('Signature', help='Signature received through the portal.', copy=False, attachment=True, max_width=1024, max_height=1024)
    signed_by = fields.Char('Signed By', help='Name of the person that signed the SO.', copy=False)
    signed_on = fields.Datetime('Signed On', help='Date of the signature.', copy=False)

    commitment_date = fields.Datetime('Delivery Date', copy=False,
                                      states={'done': [('readonly', True)], 'cancel': [('readonly', True)]},
                                      help="This is the delivery date promised to the customer. "
                                           "If set, the delivery order will be scheduled based on "
                                           "this date rather than product lead times.")
    expected_date = fields.Datetime("Expected Date", compute='_compute_expected_date', store=False,  # Note: can not be stored since depends on today()
        help="Delivery date you can promise to the customer, computed from the minimum lead time of the order lines.")
    amount_undiscounted = fields.Float('Amount Before Discount', compute='_compute_amount_undiscounted', digits=0)

    type_name = fields.Char('Type Name', compute='_compute_type_name')

    transaction_ids = fields.Many2many('payment.transaction', 'sale_order_transaction_rel', 'sale_order_id', 'transaction_id',
                                       string='Transactions', copy=False, readonly=True)
    authorized_transaction_ids = fields.Many2many('payment.transaction', compute='_compute_authorized_transaction_ids',
                                                  string='Authorized Transactions', copy=False)
    show_update_pricelist = fields.Boolean(string='Has Pricelist Changed',
                                           help="Technical Field, True if the pricelist was changed;\n"
                                                " this will then display a recomputation button")
    tag_ids = fields.Many2many('crm.tag', 'sale_order_tag_rel', 'order_id', 'tag_id', string='Tags')

    _sql_constraints = [
        ('date_order_conditional_required', "CHECK( (state IN ('sale', 'done') AND date_order IS NOT NULL) OR state NOT IN ('sale', 'done') )", "A confirmed sales order requires a confirmation date."),
    ]

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

    @api.depends('pricelist_id', 'date_order', 'company_id')
    def _compute_currency_rate(self):
        for order in self:
            if not order.company_id:
                order.currency_rate = order.currency_id.with_context(date=order.date_order).rate or 1.0
                continue
            elif order.company_id.currency_id and order.currency_id:  # the following crashes if any one is undefined
                order.currency_rate = self.env['res.currency']._get_conversion_rate(order.company_id.currency_id, order.currency_id, order.company_id, order.date_order)
            else:
                order.currency_rate = 1.0

    def _compute_access_url(self):
        super(SaleOrder, self)._compute_access_url()
        for order in self:
            order.access_url = '/my/orders/%s' % (order.id)

    def _compute_is_expired(self):
        today = fields.Date.today()
        for order in self:
            order.is_expired = order.state == 'sent' and order.validity_date and order.validity_date < today

    @api.depends('order_line.customer_lead', 'date_order', 'order_line.state')
    def _compute_expected_date(self):
        """ For service and consumable, we only take the min dates. This method is extended in sale_stock to
            take the picking_policy of SO into account.
        """
        self.mapped("order_line")  # Prefetch indication
        for order in self:
            dates_list = []
            for line in order.order_line.filtered(lambda x: x.state != 'cancel' and not x._is_delivery() and not x.display_type):
                dt = line._expected_date()
                dates_list.append(dt)
            if dates_list:
                order.expected_date = fields.Datetime.to_string(min(dates_list))
            else:
                order.expected_date = False

    @api.depends_context('lang')
    @api.depends('order_line.tax_id', 'order_line.price_unit', 'amount_total', 'amount_untaxed')
    def _compute_tax_totals_json(self):
        def compute_taxes(order_line):
            price = order_line.price_unit * (1 - (order_line.discount or 0.0) / 100.0)
            order = order_line.order_id
            return order_line.tax_id._origin.compute_all(price, order.currency_id, order_line.product_uom_qty, product=order_line.product_id, partner=order.partner_shipping_id)

        account_move = self.env['account.move']
        for order in self:
            tax_lines_data = account_move._prepare_tax_lines_data_for_totals_from_object(order.order_line, compute_taxes)
            tax_totals = account_move._get_tax_totals(order.partner_id, tax_lines_data, order.amount_total, order.amount_untaxed, order.currency_id)
            order.tax_totals_json = json.dumps(tax_totals)

    @api.depends('transaction_ids')
    def _compute_authorized_transaction_ids(self):
        for trans in self:
            trans.authorized_transaction_ids = trans.transaction_ids.filtered(lambda t: t.state == 'authorized')

    def _compute_amount_undiscounted(self):
        for order in self:
            total = 0.0
            for line in order.order_line:
                total += (line.price_subtotal * 100)/(100-line.discount) if line.discount != 100 else (line.price_unit * line.product_uom_qty)
            order.amount_undiscounted = total

    @api.depends('state')
    def _compute_type_name(self):
        for record in self:
            record.type_name = _('Quotation') if record.state in ('draft', 'sent', 'cancel') else _('Sales Order')

    @api.depends('company_id.account_fiscal_country_id', 'fiscal_position_id.country_id', 'fiscal_position_id.foreign_vat')
    def _compute_tax_country_id(self):
        for record in self:
            if record.fiscal_position_id.foreign_vat:
                record.tax_country_id = record.fiscal_position_id.country_id
            else:
                record.tax_country_id = record.company_id.account_fiscal_country_id

    @api.depends('partner_id', 'date_order')
    def _compute_analytic_account_id(self):
        for order in self:
            if not order.analytic_account_id:
                default_analytic_account = order.env['account.analytic.default'].sudo().account_get(
                    partner_id=order.partner_id.id,
                    user_id=order.env.uid,
                    date=order.date_order,
                    company_id=order.company_id.id,
                )
                order.analytic_account_id = default_analytic_account.analytic_id

    @api.ondelete(at_uninstall=False)
    def _unlink_except_draft_or_cancel(self):
        for order in self:
            if order.state not in ('draft', 'cancel'):
                raise UserError(_('You can not delete a sent quotation or a confirmed sales order. You must first cancel it.'))

    def validate_taxes_on_sales_order(self):
        # Override for correct taxcloud computation
        # when using coupon and delivery
        return True

    def _track_subtype(self, init_values):
        self.ensure_one()
        if 'state' in init_values and self.state == 'sale':
            return self.env.ref('sale.mt_order_confirmed')
        elif 'state' in init_values and self.state == 'sent':
            return self.env.ref('sale.mt_order_sent')
        return super(SaleOrder, self)._track_subtype(init_values)

    @api.onchange('partner_shipping_id', 'partner_id', 'company_id')
    def onchange_partner_shipping_id(self):
        """
        Trigger the change of fiscal position when the shipping address is modified.
        """
        self.fiscal_position_id = self.env['account.fiscal.position'].with_company(self.company_id).get_fiscal_position(self.partner_id.id, self.partner_shipping_id.id)
        return {}

    @api.onchange('partner_id')
    def onchange_partner_id(self):
        """
        Update the following fields when the partner is changed:
        - Pricelist
        - Payment terms
        - Invoice address
        - Delivery address
        - Sales Team
        """
        if not self.partner_id:
            self.update({
                'partner_invoice_id': False,
                'partner_shipping_id': False,
                'fiscal_position_id': False,
            })
            return

        self = self.with_company(self.company_id)

        addr = self.partner_id.address_get(['delivery', 'invoice'])
        partner_user = self.partner_id.user_id or self.partner_id.commercial_partner_id.user_id
        values = {
            'pricelist_id': self.partner_id.property_product_pricelist and self.partner_id.property_product_pricelist.id or False,
            'payment_term_id': self.partner_id.property_payment_term_id and self.partner_id.property_payment_term_id.id or False,
            'partner_invoice_id': addr['invoice'],
            'partner_shipping_id': addr['delivery'],
        }
        user_id = partner_user.id
        if not self.env.context.get('not_self_saleperson'):
            user_id = user_id or self.env.context.get('default_user_id', self.env.uid)
        if user_id and self.user_id.id != user_id:
            values['user_id'] = user_id

        if self.env['ir.config_parameter'].sudo().get_param('account.use_invoice_terms'):
            if self.terms_type == 'html' and self.env.company.invoice_terms_html:
                baseurl = html_keep_url(self.get_base_url() + '/terms')
                context = {'lang': self.partner_id.lang or self.env.user.lang}
                values['note'] = _('Terms & Conditions: %s', baseurl)
                del context
            elif not is_html_empty(self.env.company.invoice_terms):
                values['note'] = self.with_context(lang=self.partner_id.lang).env.company.invoice_terms
        if not self.env.context.get('not_self_saleperson') or not self.team_id:
            default_team = self.env.context.get('default_team_id', False) or self.partner_id.team_id.id
            values['team_id'] = self.env['crm.team'].with_context(
                default_team_id=default_team
            )._get_default_team_id(domain=['|', ('company_id', '=', self.company_id.id), ('company_id', '=', False)], user_id=user_id)
        self.update(values)

    @api.onchange('user_id')
    def onchange_user_id(self):
        if self.user_id:
            default_team = self.env.context.get('default_team_id', False) or self.team_id.id
            self.team_id = self.env['crm.team'].with_context(
                default_team_id=default_team
            )._get_default_team_id(user_id=self.user_id.id, domain=None)

    @api.onchange('partner_id')
    def _onchange_partner_id_warning(self):
        if not self.partner_id:
            return
        partner = self.partner_id

        # If partner has no warning, check its company
        if partner.sale_warn == 'no-message' and partner.parent_id:
            partner = partner.parent_id

        if partner.sale_warn and partner.sale_warn != 'no-message':
            # Block if partner only has warning but parent company is blocked
            if partner.sale_warn != 'block' and partner.parent_id and partner.parent_id.sale_warn == 'block':
                partner = partner.parent_id

            if partner.sale_warn == 'block':
                self.update({'partner_id': False, 'partner_invoice_id': False, 'partner_shipping_id': False, 'pricelist_id': False})

            return {
                'warning': {
                    'title': _("Warning for %s", partner.name),
                    'message': partner.sale_warn_msg,
                }
            }

    @api.onchange('commitment_date', 'expected_date')
    def _onchange_commitment_date(self):
        """ Warn if the commitment dates is sooner than the expected date """
        if (self.commitment_date and self.expected_date and self.commitment_date < self.expected_date):
            return {
                'warning': {
                    'title': _('Requested date is too soon.'),
                    'message': _("The delivery date is sooner than the expected date."
                                 "You may be unable to honor the delivery date.")
                }
            }

    @api.onchange('pricelist_id', 'order_line')
    def _onchange_pricelist_id(self):
        if self.order_line and self.pricelist_id and self._origin.pricelist_id != self.pricelist_id:
            self.show_update_pricelist = True
        else:
            self.show_update_pricelist = False

    def _get_update_prices_lines(self):
        """ Hook to exclude specific lines which should not be updated based on price list recomputation """
        return self.order_line.filtered(lambda line: not line.display_type)

    def update_prices(self):
        self.ensure_one()
        for line in self._get_update_prices_lines():
            line.product_uom_change()
            line.discount = 0  # Force 0 as discount for the cases when _onchange_discount directly returns
            line._onchange_discount()
        self.show_update_pricelist = False
        self.message_post(body=_("Product prices have been recomputed according to pricelist <b>%s<b> ", self.pricelist_id.display_name))

    @api.model
    def create(self, vals):
        if 'company_id' in vals:
            self = self.with_company(vals['company_id'])
        if vals.get('name', _('New')) == _('New'):
            seq_date = None
            if 'date_order' in vals:
                seq_date = fields.Datetime.context_timestamp(self, fields.Datetime.to_datetime(vals['date_order']))
            vals['name'] = self.env['ir.sequence'].next_by_code('sale.order', sequence_date=seq_date) or _('New')

        # Makes sure partner_invoice_id', 'partner_shipping_id' and 'pricelist_id' are defined
        if any(f not in vals for f in ['partner_invoice_id', 'partner_shipping_id', 'pricelist_id']):
            partner = self.env['res.partner'].browse(vals.get('partner_id'))
            addr = partner.address_get(['delivery', 'invoice'])
            vals['partner_invoice_id'] = vals.setdefault('partner_invoice_id', addr['invoice'])
            vals['partner_shipping_id'] = vals.setdefault('partner_shipping_id', addr['delivery'])
            vals['pricelist_id'] = vals.setdefault('pricelist_id', partner.property_product_pricelist.id)
        result = super(SaleOrder, self).create(vals)
        return result

    def _compute_field_value(self, field):
        if field.name == 'invoice_status' and not self.env.context.get('mail_activity_automation_skip'):
            filtered_self = self.filtered(lambda so: (so.user_id or so.partner_id.user_id) and so._origin.invoice_status != 'upselling')
        super()._compute_field_value(field)
        if field.name != 'invoice_status' or self.env.context.get('mail_activity_automation_skip'):
            return

        upselling_orders = filtered_self.filtered(lambda so: so.invoice_status == 'upselling')
        if not upselling_orders:
            return

        upselling_orders._create_upsell_activity()

    def copy_data(self, default=None):
        if default is None:
            default = {}
        if 'order_line' not in default:
            default['order_line'] = [(0, 0, line.copy_data()[0]) for line in self.order_line.filtered(lambda l: not l.is_downpayment)]
        return super(SaleOrder, self).copy_data(default)

    def name_get(self):
        if self._context.get('sale_show_partner_name'):
            res = []
            for order in self:
                name = order.name
                if order.partner_id.name:
                    name = '%s - %s' % (name, order.partner_id.name)
                res.append((order.id, name))
            return res
        return super(SaleOrder, self).name_get()

    @api.model
    def _name_search(self, name, args=None, operator='ilike', limit=100, name_get_uid=None):
        if self._context.get('sale_show_partner_name'):
            if operator == 'ilike' and not (name or '').strip():
                domain = []
            elif operator in ('ilike', 'like', '=', '=like', '=ilike'):
                domain = expression.AND([
                    args or [],
                    ['|', ('name', operator, name), ('partner_id.name', operator, name)]
                ])
                return self._search(domain, limit=limit, access_rights_uid=name_get_uid)
        return super(SaleOrder, self)._name_search(name, args=args, operator=operator, limit=limit, name_get_uid=name_get_uid)

    def _create_upsell_activity(self):
        self and self.activity_unlink(['sale.mail_act_sale_upsell'])
        for order in self:
            ref = "<a href='#' data-oe-model='%s' data-oe-id='%d'>%s</a>"
            order_ref = ref % (order._name, order.id or order._origin.id, order.name)
            customer_ref = ref % (order.partner_id._name, order.partner_id.id, order.partner_id.display_name)
            order.activity_schedule(
                'sale.mail_act_sale_upsell',
                user_id=order.user_id.id or order.partner_id.user_id.id,
                note=_("Upsell %(order)s for customer %(customer)s", order=order_ref, customer=customer_ref))

    def _prepare_invoice(self):
        """
        Prepare the dict of values to create the new invoice for a sales order. This method may be
        overridden to implement custom invoice generation (making sure to call super() to establish
        a clean extension chain).
        """
        self.ensure_one()
        journal = self.env['account.move'].with_context(default_move_type='out_invoice')._get_default_journal()
        if not journal:
            raise UserError(_('Please define an accounting sales journal for the company %s (%s).', self.company_id.name, self.company_id.id))

        invoice_vals = {
            'ref': self.client_order_ref or '',
            'move_type': 'out_invoice',
            'narration': self.note,
            'currency_id': self.pricelist_id.currency_id.id,
            'campaign_id': self.campaign_id.id,
            'medium_id': self.medium_id.id,
            'source_id': self.source_id.id,
            'user_id': self.user_id.id,
            'invoice_user_id': self.user_id.id,
            'team_id': self.team_id.id,
            'partner_id': self.partner_invoice_id.id,
            'partner_shipping_id': self.partner_shipping_id.id,
            'fiscal_position_id': (self.fiscal_position_id or self.fiscal_position_id.get_fiscal_position(self.partner_invoice_id.id)).id,
            'partner_bank_id': self.company_id.partner_id.bank_ids.filtered(lambda bank: bank.company_id.id in (self.company_id.id, False))[:1].id,
            'journal_id': journal.id,  # company comes from the journal
            'invoice_origin': self.name,
            'invoice_payment_term_id': self.payment_term_id.id,
            'payment_reference': self.reference,
            'transaction_ids': [(6, 0, self.transaction_ids.ids)],
            'invoice_line_ids': [],
            'company_id': self.company_id.id,
        }
        return invoice_vals

    def action_quotation_sent(self):
        if self.filtered(lambda so: so.state != 'draft'):
            raise UserError(_('Only draft orders can be marked as sent directly.'))
        for order in self:
            order.message_subscribe(partner_ids=order.partner_id.ids)
        self.write({'state': 'sent'})

    def action_view_invoice(self):
        invoices = self.mapped('invoice_ids')
        action = self.env["ir.actions.actions"]._for_xml_id("account.action_move_out_invoice_type")
        if len(invoices) > 1:
            action['domain'] = [('id', 'in', invoices.ids)]
        elif len(invoices) == 1:
            form_view = [(self.env.ref('account.view_move_form').id, 'form')]
            if 'views' in action:
                action['views'] = form_view + [(state,view) for state,view in action['views'] if view != 'form']
            else:
                action['views'] = form_view
            action['res_id'] = invoices.id
        else:
            action = {'type': 'ir.actions.act_window_close'}

        context = {
            'default_move_type': 'out_invoice',
        }
        if len(self) == 1:
            context.update({
                'default_partner_id': self.partner_id.id,
                'default_partner_shipping_id': self.partner_shipping_id.id,
                'default_invoice_payment_term_id': self.payment_term_id.id or self.partner_id.property_payment_term_id.id or self.env['account.move'].default_get(['invoice_payment_term_id']).get('invoice_payment_term_id'),
                'default_invoice_origin': self.name,
            })
        action['context'] = context
        return action

    def _get_invoice_grouping_keys(self):
        return ['company_id', 'partner_id', 'currency_id']

    @api.model
    def _nothing_to_invoice_error(self):
        return UserError(_(
            "There is nothing to invoice!\n\n"
            "Reason(s) of this behavior could be:\n"
            "- You should deliver your products before invoicing them.\n"
            "- You should modify the invoicing policy of your product: Open the product, go to the "
            "\"Sales\" tab and modify invoicing policy from \"delivered quantities\" to \"ordered "
            "quantities\". For Services, you should modify the Service Invoicing Policy to "
            "'Prepaid'."
        ))

    def _get_invoiceable_lines(self, final=False):
        """Return the invoiceable lines for order `self`."""
        down_payment_line_ids = []
        invoiceable_line_ids = []
        pending_section = None
        precision = self.env['decimal.precision'].precision_get('Product Unit of Measure')

        for line in self.order_line:
            if line.display_type == 'line_section':
                # Only invoice the section if one of its lines is invoiceable
                pending_section = line
                continue
            if line.display_type != 'line_note' and float_is_zero(line.qty_to_invoice, precision_digits=precision):
                continue
            if line.qty_to_invoice > 0 or (line.qty_to_invoice < 0 and final) or line.display_type == 'line_note':
                if line.is_downpayment:
                    # Keep down payment lines separately, to put them together
                    # at the end of the invoice, in a specific dedicated section.
                    down_payment_line_ids.append(line.id)
                    continue
                if pending_section:
                    invoiceable_line_ids.append(pending_section.id)
                    pending_section = None
                invoiceable_line_ids.append(line.id)

        return self.env['sale.order.line'].browse(invoiceable_line_ids + down_payment_line_ids)

    def _create_invoices(self, grouped=False, final=False, date=None):
        """
        Create the invoice associated to the SO.
        :param grouped: if True, invoices are grouped by SO id. If False, invoices are grouped by
                        (partner_invoice_id, currency)
        :param final: if True, refunds will be generated if necessary
        :returns: list of created invoices
        """
        if not self.env['account.move'].check_access_rights('create', False):
            try:
                self.check_access_rights('write')
                self.check_access_rule('write')
            except AccessError:
                return self.env['account.move']

        # 1) Create invoices.
        invoice_vals_list = []
        invoice_item_sequence = 0 # Incremental sequencing to keep the lines order on the invoice.
        for order in self:
            order = order.with_company(order.company_id)
            current_section_vals = None
            down_payments = order.env['sale.order.line']

            invoice_vals = order._prepare_invoice()
            invoiceable_lines = order._get_invoiceable_lines(final)

            if not any(not line.display_type for line in invoiceable_lines):
                continue

            invoice_line_vals = []
            down_payment_section_added = False
            for line in invoiceable_lines:
                if not down_payment_section_added and line.is_downpayment:
                    # Create a dedicated section for the down payments
                    # (put at the end of the invoiceable_lines)
                    invoice_line_vals.append(
                        (0, 0, order._prepare_down_payment_section_line(
                            sequence=invoice_item_sequence,
                        )),
                    )
                    down_payment_section_added = True
                    invoice_item_sequence += 1
                invoice_line_vals.append(
                    (0, 0, line._prepare_invoice_line(
                        sequence=invoice_item_sequence,
                    )),
                )
                invoice_item_sequence += 1

            invoice_vals['invoice_line_ids'] += invoice_line_vals
            invoice_vals_list.append(invoice_vals)

        if not invoice_vals_list:
            raise self._nothing_to_invoice_error()

        # 2) Manage 'grouped' parameter: group by (partner_id, currency_id).
        if not grouped:
            new_invoice_vals_list = []
            invoice_grouping_keys = self._get_invoice_grouping_keys()
            invoice_vals_list = sorted(
                invoice_vals_list,
                key=lambda x: [
                    x.get(grouping_key) for grouping_key in invoice_grouping_keys
                ]
            )
            for grouping_keys, invoices in groupby(invoice_vals_list, key=lambda x: [x.get(grouping_key) for grouping_key in invoice_grouping_keys]):
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

        # As part of the invoice creation, we make sure the sequence of multiple SO do not interfere
        # in a single invoice. Example:
        # SO 1:
        # - Section A (sequence: 10)
        # - Product A (sequence: 11)
        # SO 2:
        # - Section B (sequence: 10)
        # - Product B (sequence: 11)
        #
        # If SO 1 & 2 are grouped in the same invoice, the result will be:
        # - Section A (sequence: 10)
        # - Section B (sequence: 10)
        # - Product A (sequence: 11)
        # - Product B (sequence: 11)
        #
        # Resequencing should be safe, however we resequence only if there are less invoices than
        # orders, meaning a grouping might have been done. This could also mean that only a part
        # of the selected SO are invoiceable, but resequencing in this case shouldn't be an issue.
        if len(invoice_vals_list) < len(self):
            SaleOrderLine = self.env['sale.order.line']
            for invoice in invoice_vals_list:
                sequence = 1
                for line in invoice['invoice_line_ids']:
                    line[2]['sequence'] = SaleOrderLine._get_invoice_line_sequence(new=sequence, old=line[2]['sequence'])
                    sequence += 1

        # Manage the creation of invoices in sudo because a salesperson must be able to generate an invoice from a
        # sale order without "billing" access rights. However, he should not be able to create an invoice from scratch.
        moves = self.env['account.move'].sudo().with_context(default_move_type='out_invoice').create(invoice_vals_list)

        # 4) Some moves might actually be refunds: convert them if the total amount is negative
        # We do this after the moves have been created since we need taxes, etc. to know if the total
        # is actually negative or not
        if final:
            moves.sudo().filtered(lambda m: m.amount_total < 0).action_switch_invoice_into_refund_credit_note()
        for move in moves:
            move.message_post_with_view('mail.message_origin_link',
                values={'self': move, 'origin': move.line_ids.mapped('sale_line_ids.order_id')},
                subtype_id=self.env.ref('mail.mt_note').id
            )
        return moves

    def action_draft(self):
        orders = self.filtered(lambda s: s.state in ['cancel', 'sent'])
        return orders.write({
            'state': 'draft',
            'signature': False,
            'signed_by': False,
            'signed_on': False,
        })

    def action_cancel(self):
        cancel_warning = self._show_cancel_wizard()
        if cancel_warning:
            return {
                'name': _('Cancel Sales Order'),
                'view_mode': 'form',
                'res_model': 'sale.order.cancel',
                'view_id': self.env.ref('sale.sale_order_cancel_view_form').id,
                'type': 'ir.actions.act_window',
                'context': {'default_order_id': self.id},
                'target': 'new'
            }
        return self._action_cancel()

    def _action_cancel(self):
        inv = self.invoice_ids.filtered(lambda inv: inv.state == 'draft')
        inv.button_cancel()
        return self.write({'state': 'cancel', 'show_update_pricelist': False})

    def _show_cancel_wizard(self):
        for order in self:
            if order.invoice_ids.filtered(lambda inv: inv.state == 'draft') and not order._context.get('disable_cancel_warning'):
                return True
        return False

    def _find_mail_template(self, force_confirmation_template=False):
        self.ensure_one()
        template_id = False

        if force_confirmation_template or (self.state == 'sale' and not self.env.context.get('proforma', False)):
            template_id = int(self.env['ir.config_parameter'].sudo().get_param('sale.default_confirmation_template'))
            template_id = self.env['mail.template'].search([('id', '=', template_id)]).id
            if not template_id:
                template_id = self.env['ir.model.data']._xmlid_to_res_id('sale.mail_template_sale_confirmation', raise_if_not_found=False)
        if not template_id:
            template_id = self.env['ir.model.data']._xmlid_to_res_id('sale.email_template_edi_sale', raise_if_not_found=False)

        return template_id

    def action_quotation_send(self):
        ''' Opens a wizard to compose an email, with relevant mail template loaded by default '''
        self.ensure_one()
        template_id = self._find_mail_template()
        lang = self.env.context.get('lang')
        template = self.env['mail.template'].browse(template_id)
        if template.lang:
            lang = template._render_lang(self.ids)[self.id]
        ctx = {
            'default_model': 'sale.order',
            'default_res_id': self.ids[0],
            'default_use_template': bool(template_id),
            'default_template_id': template_id,
            'default_composition_mode': 'comment',
            'mark_so_as_sent': True,
            'custom_layout': "mail.mail_notification_paynow",
            'proforma': self.env.context.get('proforma', False),
            'force_email': True,
            'model_description': self.with_context(lang=lang).type_name,
        }
        return {
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'mail.compose.message',
            'views': [(False, 'form')],
            'view_id': False,
            'target': 'new',
            'context': ctx,
        }

    @api.returns('mail.message', lambda value: value.id)
    def message_post(self, **kwargs):
        if self.env.context.get('mark_so_as_sent'):
            self.filtered(lambda o: o.state == 'draft').with_context(tracking_disable=True).write({'state': 'sent'})
        return super(SaleOrder, self.with_context(mail_post_autofollow=self.env.context.get('mail_post_autofollow', True))).message_post(**kwargs)

    def _sms_get_number_fields(self):
        """ No phone or mobile field is available on sale model. Instead SMS will
        fallback on partner-based computation using ``_sms_get_partner_fields``. """
        return []

    def _sms_get_partner_fields(self):
        return ['partner_id']

    def _send_order_confirmation_mail(self):
        if self.env.su:
            # sending mail in sudo was meant for it being sent from superuser
            self = self.with_user(SUPERUSER_ID)
        for order in self:
            template_id = order._find_mail_template(force_confirmation_template=True)
            if template_id:
                order.with_context(force_send=True).message_post_with_template(template_id, composition_mode='comment', email_layout_xmlid="mail.mail_notification_paynow")

    def action_done(self):
        for order in self:
            tx = order.sudo().transaction_ids._get_last()
            if tx and tx.state == 'pending' and tx.acquirer_id.provider == 'transfer':
                tx._set_done()
                tx.write({'is_post_processed': True})
        return self.write({'state': 'done'})

    def action_unlock(self):
        self.write({'state': 'sale'})

    def _action_confirm(self):
        """ Implementation of additionnal mecanism of Sales Order confirmation.
            This method should be extended when the confirmation should generated
            other documents. In this method, the SO are in 'sale' state (not yet 'done').
        """
        # create an analytic account if at least an expense product
        for order in self:
            if any(expense_policy not in [False, 'no'] for expense_policy in order.order_line.mapped('product_id.expense_policy')):
                if not order.analytic_account_id:
                    order._create_analytic_account()

        return True

    def _prepare_confirmation_values(self):
        return {
            'state': 'sale',
            'date_order': fields.Datetime.now()
        }

    def action_confirm(self):
        if self._get_forbidden_state_confirm() & set(self.mapped('state')):
            raise UserError(_(
                'It is not allowed to confirm an order in the following states: %s'
            ) % (', '.join(self._get_forbidden_state_confirm())))

        for order in self.filtered(lambda order: order.partner_id not in order.message_partner_ids):
            order.message_subscribe([order.partner_id.id])
        self.write(self._prepare_confirmation_values())

        # Context key 'default_name' is sometimes propagated up to here.
        # We don't need it and it creates issues in the creation of linked records.
        context = self._context.copy()
        context.pop('default_name', None)

        self.with_context(context)._action_confirm()
        if self.env.user.has_group('sale.group_auto_done_setting'):
            self.action_done()
        return True

    def _get_forbidden_state_confirm(self):
        return {'done', 'cancel'}

    def _prepare_analytic_account_data(self, prefix=None):
        """
        Prepare method for analytic account data

        :param prefix: The prefix of the to-be-created analytic account name
        :type prefix: string
        :return: dictionary of value for new analytic account creation
        """
        name = self.name
        if prefix:
            name = prefix + ": " + self.name
        return {
            'name': name,
            'code': self.client_order_ref,
            'company_id': self.company_id.id,
            'partner_id': self.partner_id.id
        }

    def _create_analytic_account(self, prefix=None):
        for order in self:
            analytic = self.env['account.analytic.account'].create(order._prepare_analytic_account_data(prefix))
            order.analytic_account_id = analytic

    def has_to_be_signed(self, include_draft=False):
        return (self.state == 'sent' or (self.state == 'draft' and include_draft)) and not self.is_expired and self.require_signature and not self.signature

    def has_to_be_paid(self, include_draft=False):
        transaction = self.get_portal_last_transaction()
        return (self.state == 'sent' or (self.state == 'draft' and include_draft)) and not self.is_expired and self.require_payment and transaction.state != 'done' and self.amount_total

    def _notify_get_groups(self, msg_vals=None):
        """ Give access button to users and portal customer as portal is integrated
        in sale. Customer and portal group have probably no right to see
        the document so they don't have the access button. """
        groups = super(SaleOrder, self)._notify_get_groups(msg_vals=msg_vals)

        self.ensure_one()
        if self.state not in ('draft', 'cancel'):
            for group_name, group_method, group_data in groups:
                if group_name not in ('customer', 'portal'):
                    group_data['has_button_access'] = True

        return groups

    def preview_sale_order(self):
        self.ensure_one()
        return {
            'type': 'ir.actions.act_url',
            'target': 'self',
            'url': self.get_portal_url(),
        }

    def _force_lines_to_invoice_policy_order(self):
        for line in self.order_line:
            if self.state in ['sale', 'done']:
                line.qty_to_invoice = line.product_uom_qty - line.qty_invoiced
            else:
                line.qty_to_invoice = 0

    def payment_action_capture(self):
        """ Capture all transactions linked to this sale order. """
        payment_utils.check_rights_on_recordset(self)
        # In sudo mode because we need to be able to read on acquirer fields.
        self.authorized_transaction_ids.sudo().action_capture()

    def payment_action_void(self):
        """ Void all transactions linked to this sale order. """
        payment_utils.check_rights_on_recordset(self)
        # In sudo mode because we need to be able to read on acquirer fields.
        self.authorized_transaction_ids.sudo().action_void()

    def get_portal_last_transaction(self):
        self.ensure_one()
        return self.transaction_ids._get_last()

    @api.model
    def _get_customer_lead(self, product_tmpl_id):
        return False

    def _get_report_base_filename(self):
        self.ensure_one()
        return '%s %s' % (self.type_name, self.name)

    def _get_portal_return_action(self):
        """ Return the action used to display orders when returning from customer portal. """
        self.ensure_one()
        return self.env.ref('sale.action_quotations_with_onboarding')

    @api.model
    def _prepare_down_payment_section_line(self, **optional_values):
        """
        Prepare the dict of values to create a new down payment section for a sales order line.

        :param optional_values: any parameter that should be added to the returned down payment section
        """
        context = {'lang': self.partner_id.lang}
        down_payments_section_line = {
            'display_type': 'line_section',
            'name': _('Down Payments'),
            'product_id': False,
            'product_uom_id': False,
            'quantity': 0,
            'discount': 0,
            'price_unit': 0,
            'account_id': False
        }
        del context
        if optional_values:
            down_payments_section_line.update(optional_values)
        return down_payments_section_line

    def add_option_to_order_with_taxcloud(self):
        self.ensure_one()

```

## File: models\sale_order_line.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import timedelta

from odoo import api, fields, models, _
from odoo.exceptions import UserError
from odoo.tools.misc import get_lang
from odoo.osv import expression
from odoo.tools import float_is_zero, float_compare, float_round


class SaleOrderLine(models.Model):
    _name = 'sale.order.line'
    _description = 'Sales Order Line'
    _order = 'order_id, sequence, id'
    _check_company_auto = True

    @api.depends('state', 'product_uom_qty', 'qty_delivered', 'qty_to_invoice', 'qty_invoiced')
    def _compute_invoice_status(self):
        """
        Compute the invoice status of a SO line. Possible statuses:
        - no: if the SO is not in status 'sale' or 'done', we consider that there is nothing to
          invoice. This is also hte default value if the conditions of no other status is met.
        - to invoice: we refer to the quantity to invoice of the line. Refer to method
          `_get_to_invoice_qty()` for more information on how this quantity is calculated.
        - upselling: this is possible only for a product invoiced on ordered quantities for which
          we delivered more than expected. The could arise if, for example, a project took more
          time than expected but we decided not to invoice the extra cost to the client. This
          occurs onyl in state 'sale', so that when a SO is set to done, the upselling opportunity
          is removed from the list.
        - invoiced: the quantity invoiced is larger or equal to the quantity ordered.
        """
        precision = self.env['decimal.precision'].precision_get('Product Unit of Measure')
        for line in self:
            if line.state not in ('sale', 'done'):
                line.invoice_status = 'no'
            elif line.is_downpayment and line.untaxed_amount_to_invoice == 0:
                line.invoice_status = 'invoiced'
            elif not float_is_zero(line.qty_to_invoice, precision_digits=precision):
                line.invoice_status = 'to invoice'
            elif line.state == 'sale' and line.product_id.invoice_policy == 'order' and\
                    line.product_uom_qty >= 0.0 and\
                    float_compare(line.qty_delivered, line.product_uom_qty, precision_digits=precision) == 1:
                line.invoice_status = 'upselling'
            elif float_compare(line.qty_invoiced, line.product_uom_qty, precision_digits=precision) >= 0:
                line.invoice_status = 'invoiced'
            else:
                line.invoice_status = 'no'

    def _expected_date(self):
        self.ensure_one()
        order_date = fields.Datetime.from_string(self.order_id.date_order if self.order_id.date_order and self.order_id.state in ['sale', 'done'] else fields.Datetime.now())
        return order_date + timedelta(days=self.customer_lead or 0.0)

    @api.depends('product_uom_qty', 'discount', 'price_unit', 'tax_id')
    def _compute_amount(self):
        """
        Compute the amounts of the SO line.
        """
        for line in self:
            price = line.price_unit * (1 - (line.discount or 0.0) / 100.0)
            taxes = line.tax_id.compute_all(price, line.order_id.currency_id, line.product_uom_qty, product=line.product_id, partner=line.order_id.partner_shipping_id)
            line.update({
                'price_tax': sum(t.get('amount', 0.0) for t in taxes.get('taxes', [])),
                'price_total': taxes['total_included'],
                'price_subtotal': taxes['total_excluded'],
            })

    @api.depends('product_id', 'order_id.state', 'qty_invoiced', 'qty_delivered')
    def _compute_product_updatable(self):
        for line in self:
            if line.state in ['done', 'cancel'] or (line.state == 'sale' and (line.qty_invoiced > 0 or line.qty_delivered > 0)):
                line.product_updatable = False
            else:
                line.product_updatable = True

    # no trigger product_id.invoice_policy to avoid retroactively changing SO
    @api.depends('qty_invoiced', 'qty_delivered', 'product_uom_qty', 'order_id.state')
    def _get_to_invoice_qty(self):
        """
        Compute the quantity to invoice. If the invoice policy is order, the quantity to invoice is
        calculated from the ordered quantity. Otherwise, the quantity delivered is used.
        """
        for line in self:
            if line.order_id.state in ['sale', 'done']:
                if line.product_id.invoice_policy == 'order':
                    line.qty_to_invoice = line.product_uom_qty - line.qty_invoiced
                else:
                    line.qty_to_invoice = line.qty_delivered - line.qty_invoiced
            else:
                line.qty_to_invoice = 0

    @api.depends('invoice_lines.move_id.state', 'invoice_lines.quantity', 'untaxed_amount_to_invoice')
    def _compute_qty_invoiced(self):
        """
        Compute the quantity invoiced. If case of a refund, the quantity invoiced is decreased. Note
        that this is the case only if the refund is generated from the SO and that is intentional: if
        a refund made would automatically decrease the invoiced quantity, then there is a risk of reinvoicing
        it automatically, which may not be wanted at all. That's why the refund has to be created from the SO
        """
        for line in self:
            qty_invoiced = 0.0
            for invoice_line in line._get_invoice_lines():
                if invoice_line.move_id.state != 'cancel' or invoice_line.move_id.payment_state == 'invoicing_legacy':
                    if invoice_line.move_id.move_type == 'out_invoice':
                        qty_invoiced += invoice_line.product_uom_id._compute_quantity(invoice_line.quantity, line.product_uom)
                    elif invoice_line.move_id.move_type == 'out_refund':
                        qty_invoiced -= invoice_line.product_uom_id._compute_quantity(invoice_line.quantity, line.product_uom)
            line.qty_invoiced = qty_invoiced

    def _get_invoice_lines(self):
        self.ensure_one()
        if self._context.get('accrual_entry_date'):
            return self.invoice_lines.filtered(
                lambda l: l.move_id.invoice_date and l.move_id.invoice_date <= self._context['accrual_entry_date']
            )
        else:
            return self.invoice_lines

    @api.depends('price_unit', 'discount')
    def _compute_price_reduce(self):
        for line in self:
            line.price_reduce = line.price_unit * (1.0 - line.discount / 100.0)

    @api.depends('price_total', 'product_uom_qty')
    def _compute_price_reduce_taxinc(self):
        for line in self:
            line.price_reduce_taxinc = line.price_total / line.product_uom_qty if line.product_uom_qty else 0.0

    @api.depends('price_subtotal', 'product_uom_qty')
    def _compute_price_reduce_taxexcl(self):
        for line in self:
            line.price_reduce_taxexcl = line.price_subtotal / line.product_uom_qty if line.product_uom_qty else 0.0

    def _compute_tax_id(self):
        for line in self:
            line = line.with_company(line.company_id)
            fpos = line.order_id.fiscal_position_id or line.order_id.fiscal_position_id.get_fiscal_position(line.order_partner_id.id)
            # If company_id is set, always filter taxes by the company
            taxes = line.product_id.taxes_id.filtered(lambda t: t.company_id == line.env.company)
            line.tax_id = fpos.map_tax(taxes)

    @api.model
    def _prepare_add_missing_fields(self, values):
        """ Deduce missing required fields from the onchange """
        res = {}
        onchange_fields = ['name', 'price_unit', 'product_uom', 'tax_id']
        if values.get('order_id') and values.get('product_id') and any(f not in values for f in onchange_fields):
            line = self.new(values)
            line.product_id_change()
            for field in onchange_fields:
                if field not in values:
                    res[field] = line._fields[field].convert_to_write(line[field], line)
        return res

    @api.model_create_multi
    def create(self, vals_list):
        for values in vals_list:
            if values.get('display_type', self.default_get(['display_type'])['display_type']):
                values.update(product_id=False, price_unit=0, product_uom_qty=0, product_uom=False, customer_lead=0)

            values.update(self._prepare_add_missing_fields(values))

        lines = super().create(vals_list)
        for line in lines:
            if line.product_id and line.order_id.state == 'sale':
                msg = _("Extra line with %s", line.product_id.display_name)
                line.order_id.message_post(body=msg)
                # create an analytic account if at least an expense product
                if line.product_id.expense_policy not in [False, 'no'] and not line.order_id.analytic_account_id:
                    line.order_id._create_analytic_account()
        return lines

    _sql_constraints = [
        ('accountable_required_fields',
            "CHECK(display_type IS NOT NULL OR (product_id IS NOT NULL AND product_uom IS NOT NULL))",
            "Missing required fields on accountable sale order line."),
        ('non_accountable_null_fields',
            "CHECK(display_type IS NULL OR (product_id IS NULL AND price_unit = 0 AND product_uom_qty = 0 AND product_uom IS NULL AND customer_lead = 0))",
            "Forbidden values on non-accountable sale order line"),
    ]

    def _update_line_quantity(self, values):
        orders = self.mapped('order_id')
        for order in orders:
            order_lines = self.filtered(lambda x: x.order_id == order)
            msg = "<b>" + _("The ordered quantity has been updated.") + "</b><ul>"
            for line in order_lines:
                msg += "<li> %s: <br/>" % line.product_id.display_name
                msg += _(
                    "Ordered Quantity: %(old_qty)s -> %(new_qty)s",
                    old_qty=line.product_uom_qty,
                    new_qty=values["product_uom_qty"]
                ) + "<br/>"
                if line.product_id.type in ('consu', 'product'):
                    msg += _("Delivered Quantity: %s", line.qty_delivered) + "<br/>"
                msg += _("Invoiced Quantity: %s", line.qty_invoiced) + "<br/>"
            msg += "</ul>"
            order.message_post(body=msg)

    def write(self, values):
        if 'display_type' in values and self.filtered(lambda line: line.display_type != values.get('display_type')):
            raise UserError(_("You cannot change the type of a sale order line. Instead you should delete the current line and create a new line of the proper type."))

        if 'product_uom_qty' in values:
            precision = self.env['decimal.precision'].precision_get('Product Unit of Measure')
            self.filtered(
                lambda r: r.state == 'sale' and float_compare(r.product_uom_qty, values['product_uom_qty'], precision_digits=precision) != 0)._update_line_quantity(values)

        # Prevent writing on a locked SO.
        protected_fields = self._get_protected_fields()
        if 'done' in self.mapped('order_id.state') and any(f in values.keys() for f in protected_fields):
            protected_fields_modified = list(set(protected_fields) & set(values.keys()))
            fields = self.env['ir.model.fields'].search([
                ('name', 'in', protected_fields_modified), ('model', '=', self._name)
            ])
            raise UserError(
                _('It is forbidden to modify the following fields in a locked order:\n%s')
                % '\n'.join(fields.mapped('field_description'))
            )

        result = super(SaleOrderLine, self).write(values)
        return result

    order_id = fields.Many2one('sale.order', string='Order Reference', required=True, ondelete='cascade', index=True, copy=False)
    name = fields.Text(string='Description', required=True)
    sequence = fields.Integer(string='Sequence', default=10)

    invoice_lines = fields.Many2many('account.move.line', 'sale_order_line_invoice_rel', 'order_line_id', 'invoice_line_id', string='Invoice Lines', copy=False)
    invoice_status = fields.Selection([
        ('upselling', 'Upselling Opportunity'),
        ('invoiced', 'Fully Invoiced'),
        ('to invoice', 'To Invoice'),
        ('no', 'Nothing to Invoice')
        ], string='Invoice Status', compute='_compute_invoice_status', store=True, default='no')
    price_unit = fields.Float('Unit Price', required=True, digits='Product Price', default=0.0)

    price_subtotal = fields.Monetary(compute='_compute_amount', string='Subtotal', store=True)
    price_tax = fields.Float(compute='_compute_amount', string='Total Tax', store=True)
    price_total = fields.Monetary(compute='_compute_amount', string='Total', store=True)

    price_reduce = fields.Float(compute='_compute_price_reduce', string='Price Reduce', digits='Product Price', store=True)
    tax_id = fields.Many2many('account.tax', string='Taxes', context={'active_test': False}, check_company=True)
    price_reduce_taxinc = fields.Monetary(compute='_compute_price_reduce_taxinc', string='Price Reduce Tax inc', store=True)
    price_reduce_taxexcl = fields.Monetary(compute='_compute_price_reduce_taxexcl', string='Price Reduce Tax excl', store=True)

    discount = fields.Float(string='Discount (%)', digits='Discount', default=0.0)

    product_id = fields.Many2one(
        'product.product', string='Product', domain="[('sale_ok', '=', True), '|', ('company_id', '=', False), ('company_id', '=', company_id)]",
        change_default=True, ondelete='restrict', check_company=True)  # Unrequired company
    product_template_id = fields.Many2one(
        'product.template', string='Product Template',
        related="product_id.product_tmpl_id", domain=[('sale_ok', '=', True)])
    product_updatable = fields.Boolean(compute='_compute_product_updatable', string='Can Edit Product', default=True)
    product_uom_qty = fields.Float(string='Quantity', digits='Product Unit of Measure', required=True, default=1.0)
    product_uom = fields.Many2one('uom.uom', string='Unit of Measure', domain="[('category_id', '=', product_uom_category_id)]", ondelete="restrict")
    product_uom_category_id = fields.Many2one(related='product_id.uom_id.category_id')
    product_uom_readonly = fields.Boolean(compute='_compute_product_uom_readonly')
    product_custom_attribute_value_ids = fields.One2many('product.attribute.custom.value', 'sale_order_line_id', string="Custom Values", copy=True)

    # M2M holding the values of product.attribute with create_variant field set to 'no_variant'
    # It allows keeping track of the extra_price associated to those attribute values and add them to the SO line description
    product_no_variant_attribute_value_ids = fields.Many2many('product.template.attribute.value', string="Extra Values", ondelete='restrict')

    qty_delivered_method = fields.Selection([
        ('manual', 'Manual'),
        ('analytic', 'Analytic From Expenses')
    ], string="Method to update delivered qty", compute='_compute_qty_delivered_method', store=True,
        help="According to product configuration, the delivered quantity can be automatically computed by mechanism :\n"
             "  - Manual: the quantity is set manually on the line\n"
             "  - Analytic From expenses: the quantity is the quantity sum from posted expenses\n"
             "  - Timesheet: the quantity is the sum of hours recorded on tasks linked to this sale line\n"
             "  - Stock Moves: the quantity comes from confirmed pickings\n")
    qty_delivered = fields.Float('Delivered Quantity', copy=False, compute='_compute_qty_delivered', inverse='_inverse_qty_delivered', store=True, digits='Product Unit of Measure', default=0.0)
    qty_delivered_manual = fields.Float('Delivered Manually', copy=False, digits='Product Unit of Measure', default=0.0)
    qty_to_invoice = fields.Float(
        compute='_get_to_invoice_qty', string='To Invoice Quantity', store=True,
        digits='Product Unit of Measure')
    qty_invoiced = fields.Float(
        compute='_compute_qty_invoiced', string='Invoiced Quantity', store=True,
        digits='Product Unit of Measure')

    untaxed_amount_invoiced = fields.Monetary("Untaxed Invoiced Amount", compute='_compute_untaxed_amount_invoiced', store=True)
    untaxed_amount_to_invoice = fields.Monetary("Untaxed Amount To Invoice", compute='_compute_untaxed_amount_to_invoice', store=True)

    salesman_id = fields.Many2one(related='order_id.user_id', store=True, string='Salesperson')
    currency_id = fields.Many2one(related='order_id.currency_id', depends=['order_id.currency_id'], store=True, string='Currency')
    company_id = fields.Many2one(related='order_id.company_id', string='Company', store=True, index=True)
    order_partner_id = fields.Many2one(related='order_id.partner_id', store=True, string='Customer', index=True)
    analytic_tag_ids = fields.Many2many(
        'account.analytic.tag', string='Analytic Tags',
        compute='_compute_analytic_tag_ids', store=True, readonly=False,
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    analytic_line_ids = fields.One2many('account.analytic.line', 'so_line', string="Analytic lines")
    is_expense = fields.Boolean('Is expense', help="Is true if the sales order line comes from an expense or a vendor bills")
    is_downpayment = fields.Boolean(
        string="Is a down payment", help="Down payments are made when creating invoices from a sales order."
        " They are not copied when duplicating a sales order.")

    state = fields.Selection(
        related='order_id.state', string='Order Status', copy=False, store=True)

    customer_lead = fields.Float(
        'Lead Time', required=True, default=0.0,
        help="Number of days between the order confirmation and the shipping of the products to the customer")

    display_type = fields.Selection([
        ('line_section', "Section"),
        ('line_note', "Note")], default=False, help="Technical field for UX purpose.")

    product_packaging_id = fields.Many2one('product.packaging', string='Packaging', default=False, domain="[('sales', '=', True), ('product_id','=',product_id)]", check_company=True)
    product_packaging_qty = fields.Float('Packaging Quantity')

    @api.depends('state')
    def _compute_product_uom_readonly(self):
        for line in self:
            line.product_uom_readonly = line.ids and line.state in ['sale', 'done', 'cancel']

    @api.depends('is_expense')
    def _compute_qty_delivered_method(self):
        """ Sale module compute delivered qty for product [('type', 'in', ['consu']), ('service_type', '=', 'manual')]
                - consu + expense_policy : analytic (sum of analytic unit_amount)
                - consu + no expense_policy : manual (set manually on SOL)
                - service (+ service_type='manual', the only available option) : manual

            This is true when only sale is installed: sale_stock redifine the behavior for 'consu' type,
            and sale_timesheet implements the behavior of 'service' + service_type=timesheet.
        """
        for line in self:
            if line.is_expense:
                line.qty_delivered_method = 'analytic'
            else:  # service and consu
                line.qty_delivered_method = 'manual'

    @api.depends('qty_delivered_method', 'qty_delivered_manual', 'analytic_line_ids.so_line', 'analytic_line_ids.unit_amount', 'analytic_line_ids.product_uom_id')
    def _compute_qty_delivered(self):
        """ This method compute the delivered quantity of the SO lines: it covers the case provide by sale module, aka
            expense/vendor bills (sum of unit_amount of AAL), and manual case.
            This method should be overridden to provide other way to automatically compute delivered qty. Overrides should
            take their concerned so lines, compute and set the `qty_delivered` field, and call super with the remaining
            records.
        """
        # compute for analytic lines
        lines_by_analytic = self.filtered(lambda sol: sol.qty_delivered_method == 'analytic')
        mapping = lines_by_analytic._get_delivered_quantity_by_analytic([('amount', '<=', 0.0)])
        for so_line in lines_by_analytic:
            so_line.qty_delivered = mapping.get(so_line.id or so_line._origin.id, 0.0)
        # compute for manual lines
        for line in self:
            if line.qty_delivered_method == 'manual':
                line.qty_delivered = line.qty_delivered_manual or 0.0

    def _get_delivered_quantity_by_analytic(self, additional_domain):
        """ Compute and write the delivered quantity of current SO lines, based on their related
            analytic lines.
            :param additional_domain: domain to restrict AAL to include in computation (required since timesheet is an AAL with a project ...)
        """
        result = {}

        # avoid recomputation if no SO lines concerned
        if not self:
            return result

        # group analytic lines by product uom and so line
        domain = expression.AND([[('so_line', 'in', self.ids)], additional_domain])
        data = self.env['account.analytic.line'].read_group(
            domain,
            ['so_line', 'unit_amount', 'product_uom_id'], ['product_uom_id', 'so_line'], lazy=False
        )

        # convert uom and sum all unit_amount of analytic lines to get the delivered qty of SO lines
        # browse so lines and product uoms here to make them share the same prefetch
        lines = self.browse([item['so_line'][0] for item in data])
        lines_map = {line.id: line for line in lines}
        product_uom_ids = [item['product_uom_id'][0] for item in data if item['product_uom_id']]
        product_uom_map = {uom.id: uom for uom in self.env['uom.uom'].browse(product_uom_ids)}
        for item in data:
            if not item['product_uom_id']:
                continue
            so_line_id = item['so_line'][0]
            so_line = lines_map[so_line_id]
            result.setdefault(so_line_id, 0.0)
            uom = product_uom_map.get(item['product_uom_id'][0])
            if so_line.product_uom.category_id == uom.category_id:
                qty = uom._compute_quantity(item['unit_amount'], so_line.product_uom, rounding_method='HALF-UP')
            else:
                qty = item['unit_amount']
            result[so_line_id] += qty

        return result

    @api.onchange('qty_delivered')
    def _inverse_qty_delivered(self):
        """ When writing on qty_delivered, if the value should be modify manually (`qty_delivered_method` = 'manual' only),
            then we put the value in `qty_delivered_manual`. Otherwise, `qty_delivered_manual` should be False since the
            delivered qty is automatically compute by other mecanisms.
        """
        for line in self:
            if line.qty_delivered_method == 'manual':
                line.qty_delivered_manual = line.qty_delivered
            else:
                line.qty_delivered_manual = 0.0

    @api.onchange('product_id', 'product_uom_qty', 'product_uom')
    def _onchange_suggest_packaging(self):
        # remove packaging if not match the product
        if self.product_packaging_id.product_id != self.product_id:
            self.product_packaging_id = False
        # suggest biggest suitable packaging
        if self.product_id and self.product_uom_qty and self.product_uom:
            self.product_packaging_id = self.product_id.packaging_ids.filtered('sales')._find_suitable_product_packaging(self.product_uom_qty, self.product_uom) or self.product_packaging_id

    @api.onchange('product_packaging_id')
    def _onchange_product_packaging_id(self):
        if self.product_packaging_id and self.product_uom_qty:
            newqty = self.product_packaging_id._check_qty(self.product_uom_qty, self.product_uom, "UP")
            if float_compare(newqty, self.product_uom_qty, precision_rounding=self.product_uom.rounding) != 0:
                return {
                    'warning': {
                        'title': _('Warning'),
                        'message': _(
                            "This product is packaged by %(pack_size).2f %(pack_name)s. You should sell %(quantity).2f %(unit)s.",
                            pack_size=self.product_packaging_id.qty,
                            pack_name=self.product_id.uom_id.name,
                            quantity=newqty,
                            unit=self.product_uom.name
                        ),
                    },
                }

    @api.onchange('product_packaging_id', 'product_uom', 'product_uom_qty')
    def _onchange_update_product_packaging_qty(self):
        if not self.product_packaging_id:
            self.product_packaging_qty = False
        else:
            packaging_uom = self.product_packaging_id.product_uom_id
            packaging_uom_qty = self.product_uom._compute_quantity(self.product_uom_qty, packaging_uom)
            self.product_packaging_qty = float_round(packaging_uom_qty / self.product_packaging_id.qty, precision_rounding=packaging_uom.rounding)

    @api.onchange('product_packaging_qty')
    def _onchange_product_packaging_qty(self):
        if self.product_packaging_id:
            packaging_uom = self.product_packaging_id.product_uom_id
            qty_per_packaging = self.product_packaging_id.qty
            product_uom_qty = packaging_uom._compute_quantity(self.product_packaging_qty * qty_per_packaging, self.product_uom)
            if float_compare(product_uom_qty, self.product_uom_qty, precision_rounding=self.product_uom.rounding) != 0:
                self.product_uom_qty = product_uom_qty

    @api.depends('invoice_lines', 'invoice_lines.price_total', 'invoice_lines.move_id.state', 'invoice_lines.move_id.move_type')
    def _compute_untaxed_amount_invoiced(self):
        """ Compute the untaxed amount already invoiced from the sale order line, taking the refund attached
            the so line into account. This amount is computed as
                SUM(inv_line.price_subtotal) - SUM(ref_line.price_subtotal)
            where
                `inv_line` is a customer invoice line linked to the SO line
                `ref_line` is a customer credit note (refund) line linked to the SO line
        """
        for line in self:
            amount_invoiced = 0.0
            for invoice_line in line._get_invoice_lines():
                if invoice_line.move_id.state == 'posted':
                    invoice_date = invoice_line.move_id.invoice_date or fields.Date.today()
                    if invoice_line.move_id.move_type == 'out_invoice':
                        amount_invoiced += invoice_line.currency_id._convert(invoice_line.price_subtotal, line.currency_id, line.company_id, invoice_date)
                    elif invoice_line.move_id.move_type == 'out_refund':
                        amount_invoiced -= invoice_line.currency_id._convert(invoice_line.price_subtotal, line.currency_id, line.company_id, invoice_date)
            line.untaxed_amount_invoiced = amount_invoiced

    @api.depends('state', 'price_reduce', 'product_id', 'untaxed_amount_invoiced', 'qty_delivered', 'product_uom_qty')
    def _compute_untaxed_amount_to_invoice(self):
        """ Total of remaining amount to invoice on the sale order line (taxes excl.) as
                total_sol - amount already invoiced
            where Total_sol depends on the invoice policy of the product.

            Note: Draft invoice are ignored on purpose, the 'to invoice' amount should
            come only from the SO lines.
        """
        for line in self:
            amount_to_invoice = 0.0
            if line.state in ['sale', 'done']:
                # Note: do not use price_subtotal field as it returns zero when the ordered quantity is
                # zero. It causes problem for expense line (e.i.: ordered qty = 0, deli qty = 4,
                # price_unit = 20 ; subtotal is zero), but when you can invoice the line, you see an
                # amount and not zero. Since we compute untaxed amount, we can use directly the price
                # reduce (to include discount) without using `compute_all()` method on taxes.
                price_subtotal = 0.0
                uom_qty_to_consider = line.qty_delivered if line.product_id.invoice_policy == 'delivery' else line.product_uom_qty
                price_reduce = line.price_unit * (1 - (line.discount or 0.0) / 100.0)
                price_subtotal = price_reduce * uom_qty_to_consider
                if len(line.tax_id.filtered(lambda tax: tax.price_include)) > 0:
                    # As included taxes are not excluded from the computed subtotal, `compute_all()` method
                    # has to be called to retrieve the subtotal without them.
                    # `price_reduce_taxexcl` cannot be used as it is computed from `price_subtotal` field. (see upper Note)
                    price_subtotal = line.tax_id.compute_all(
                        price_reduce,
                        currency=line.order_id.currency_id,
                        quantity=uom_qty_to_consider,
                        product=line.product_id,
                        partner=line.order_id.partner_shipping_id)['total_excluded']
                inv_lines = line._get_invoice_lines()
                if any(inv_lines.mapped(lambda l: l.discount != line.discount)):
                    # In case of re-invoicing with different discount we try to calculate manually the
                    # remaining amount to invoice
                    amount = 0
                    for l in inv_lines:
                        if len(l.tax_ids.filtered(lambda tax: tax.price_include)) > 0:
                            amount += l.tax_ids.compute_all(l.currency_id._convert(l.price_unit, line.currency_id, line.company_id, l.date or fields.Date.today(), round=False) * l.quantity)['total_excluded']
                        else:
                            amount += l.currency_id._convert(l.price_unit, line.currency_id, line.company_id, l.date or fields.Date.today(), round=False) * l.quantity

                    amount_to_invoice = max(price_subtotal - amount, 0)
                else:
                    amount_to_invoice = price_subtotal - line.untaxed_amount_invoiced

            line.untaxed_amount_to_invoice = amount_to_invoice

    @api.depends('product_id', 'order_id.date_order', 'order_id.partner_id')
    def _compute_analytic_tag_ids(self):
        for line in self:
            if not line.display_type and line.state == 'draft':
                default_analytic_account = line.env['account.analytic.default'].sudo().account_get(
                    product_id=line.product_id.id,
                    partner_id=line.order_id.partner_id.id,
                    user_id=self.env.uid,
                    date=line.order_id.date_order,
                    company_id=line.company_id.id,
                )
                line.analytic_tag_ids = default_analytic_account.analytic_tag_ids

    def compute_uom_qty(self, new_qty, stock_move, rounding=True):
        return self.product_uom._compute_quantity(new_qty, stock_move.product_uom, rounding)

    def _get_invoice_line_sequence(self, new=0, old=0):
        """
        Method intended to be overridden in third-party module if we want to prevent the resequencing
        of invoice lines.

        :param int new:   the new line sequence
        :param int old:   the old line sequence

        :return:          the sequence of the SO line, by default the new one.
        """
        return new or old

    def _prepare_invoice_line(self, **optional_values):
        """
        Prepare the dict of values to create the new invoice line for a sales order line.

        :param qty: float quantity to invoice
        :param optional_values: any parameter that should be added to the returned invoice line
        """
        self.ensure_one()
        res = {
            'display_type': self.display_type,
            'sequence': self.sequence,
            'name': self.name,
            'product_id': self.product_id.id,
            'product_uom_id': self.product_uom.id,
            'quantity': self.qty_to_invoice,
            'discount': self.discount,
            'price_unit': self.price_unit,
            'tax_ids': [(6, 0, self.tax_id.ids)],
            'sale_line_ids': [(4, self.id)],
        }
        if self.order_id.analytic_account_id and not self.display_type:
            res['analytic_account_id'] = self.order_id.analytic_account_id.id
        if self.analytic_tag_ids and not self.display_type:
            res['analytic_tag_ids'] = [(6, 0, self.analytic_tag_ids.ids)]
        if optional_values:
            res.update(optional_values)
        if self.display_type:
            res['account_id'] = False
        return res

    def _prepare_procurement_values(self, group_id=False):
        """ Prepare specific key for moves or other components that will be created from a stock rule
        comming from a sale order line. This method could be override in order to add other custom key that could
        be used in move/po creation.
        """
        return {}

    def _get_display_price(self, product):
        # TO DO: move me in master/saas-16 on sale.order
        # awa: don't know if it's still the case since we need the "product_no_variant_attribute_value_ids" field now
        # to be able to compute the full price

        # it is possible that a no_variant attribute is still in a variant if
        # the type of the attribute has been changed after creation.
        no_variant_attributes_price_extra = [
            ptav.price_extra for ptav in self.product_no_variant_attribute_value_ids.filtered(
                lambda ptav:
                    ptav.price_extra and
                    ptav not in product.product_template_attribute_value_ids
            )
        ]
        if no_variant_attributes_price_extra:
            product = product.with_context(
                no_variant_attributes_price_extra=tuple(no_variant_attributes_price_extra)
            )

        if self.order_id.pricelist_id.discount_policy == 'with_discount':
            return product.with_context(pricelist=self.order_id.pricelist_id.id, uom=self.product_uom.id).price
        product_context = dict(self.env.context, partner_id=self.order_id.partner_id.id, date=self.order_id.date_order, uom=self.product_uom.id)

        final_price, rule_id = self.order_id.pricelist_id.with_context(product_context).get_product_price_rule(product or self.product_id, self.product_uom_qty or 1.0, self.order_id.partner_id)
        base_price, currency = self.with_context(product_context)._get_real_price_currency(product, rule_id, self.product_uom_qty, self.product_uom, self.order_id.pricelist_id.id)
        if currency != self.order_id.pricelist_id.currency_id:
            base_price = currency._convert(
                base_price, self.order_id.pricelist_id.currency_id,
                self.order_id.company_id or self.env.company, self.order_id.date_order or fields.Date.today())
        # negative discounts (= surcharge) are included in the display price
        return max(base_price, final_price)

    @api.onchange('product_id')
    def product_id_change(self):
        if not self.product_id:
            return

        if not self.product_uom or (self.product_id.uom_id.id != self.product_uom.id):
            self.update({
                'product_uom': self.product_id.uom_id,
                'product_uom_qty': self.product_uom_qty or 1.0
            })

        self._update_description()
        self._update_taxes()

        product = self.product_id
        if product and product.sale_line_warn != 'no-message':
            if product.sale_line_warn == 'block':
                self.product_id = False
            return {
                'warning': {
                    'title': _("Warning for %s", product.name),
                    'message': product.sale_line_warn_msg,
                }
            }

    def _update_description(self):
        if not self.product_id:
            return

        valid_values = self.product_id.product_tmpl_id.valid_product_template_attribute_line_ids.product_template_value_ids
        # remove the is_custom values that don't belong to this template
        for pacv in self.product_custom_attribute_value_ids:
            if pacv.custom_product_template_attribute_value_id not in valid_values:
                self.product_custom_attribute_value_ids -= pacv

        # remove the no_variant attributes that don't belong to this template
        for ptav in self.product_no_variant_attribute_value_ids:
            if ptav._origin not in valid_values:
                self.product_no_variant_attribute_value_ids -= ptav

        lang = get_lang(self.env, self.order_id.partner_id.lang).code
        product = self.product_id.with_context(
            lang=lang,
        )
        self.update({
            'name': self.with_context(lang=lang).get_sale_order_line_multiline_description_sale(product)
        })

    def _update_taxes(self):
        if not self.product_id:
            return

        self._compute_tax_id()

        if self.order_id.pricelist_id and self.order_id.partner_id:
            product = self.product_id.with_context(
                partner=self.order_id.partner_id,
                quantity=self.product_uom_qty,
                date=self.order_id.date_order,
                pricelist=self.order_id.pricelist_id.id,
                uom=self.product_uom.id
            )
            self.update({
                'price_unit': product._get_tax_included_unit_price(
                    self.company_id,
                    self.order_id.currency_id,
                    self.order_id.date_order,
                    'sale',
                    fiscal_position=self.order_id.fiscal_position_id,
                    product_price_unit=self._get_display_price(product),
                    product_currency=self.order_id.currency_id
                )
            })

    @api.onchange('product_uom', 'product_uom_qty')
    def product_uom_change(self):
        if not self.product_uom or not self.product_id:
            self.price_unit = 0.0
            return
        if self.order_id.pricelist_id and self.order_id.partner_id:
            product = self.product_id.with_context(
                lang=self.order_id.partner_id.lang,
                partner=self.order_id.partner_id,
                quantity=self.product_uom_qty,
                date=self.order_id.date_order,
                pricelist=self.order_id.pricelist_id.id,
                uom=self.product_uom.id,
                fiscal_position=self.env.context.get('fiscal_position')
            )
            self.price_unit = product._get_tax_included_unit_price(
                self.company_id or self.order_id.company_id,
                self.order_id.currency_id,
                self.order_id.date_order,
                'sale',
                fiscal_position=self.order_id.fiscal_position_id,
                product_price_unit=self._get_display_price(product),
                product_currency=self.order_id.currency_id
            )

    def name_get(self):
        result = []
        for so_line in self.sudo():
            name = '%s - %s' % (so_line.order_id.name, so_line.name and so_line.name.split('\n')[0] or so_line.product_id.name)
            if so_line.order_partner_id.ref:
                name = '%s (%s)' % (name, so_line.order_partner_id.ref)
            result.append((so_line.id, name))
        return result

    @api.model
    def _name_search(self, name, args=None, operator='ilike', limit=100, name_get_uid=None):
        if operator in ('ilike', 'like', '=', '=like', '=ilike'):
            args = expression.AND([
                args or [],
                ['|', ('order_id.name', operator, name), ('name', operator, name)]
            ])
            return self._search(args, limit=limit, access_rights_uid=name_get_uid)
        return super(SaleOrderLine, self)._name_search(name, args=args, operator=operator, limit=limit, name_get_uid=name_get_uid)

    def _check_line_unlink(self):
        """
        Check wether a line can be deleted or not.

        Lines cannot be deleted if the order is confirmed; downpayment
        lines who have not yet been invoiced bypass that exception.
        Also, allow deleting UX lines (notes/sections).
        :rtype: recordset sale.order.line
        :returns: set of lines that cannot be deleted
        """
        return self.filtered(lambda line: line.state in ('sale', 'done') and (line.invoice_lines or not line.is_downpayment) and not line.display_type)

    @api.ondelete(at_uninstall=False)
    def _unlink_except_confirmed(self):
        if self._check_line_unlink():
            raise UserError(_('You can not remove an order line once the sales order is confirmed.\nYou should rather set the quantity to 0.'))

    def _get_real_price_currency(self, product, rule_id, qty, uom, pricelist_id):
        """Retrieve the price before applying the pricelist
            :param obj product: object of current product record
            :parem float qty: total quentity of product
            :param tuple price_and_rule: tuple(price, suitable_rule) coming from pricelist computation
            :param obj uom: unit of measure of current order line
            :param integer pricelist_id: pricelist id of sales order"""
        PricelistItem = self.env['product.pricelist.item']
        field_name = 'lst_price'
        currency_id = None
        product_currency = product.currency_id
        if rule_id:
            pricelist_item = PricelistItem.browse(rule_id)
            if pricelist_item.pricelist_id.discount_policy == 'without_discount':
                while pricelist_item.base == 'pricelist' and pricelist_item.base_pricelist_id and pricelist_item.base_pricelist_id.discount_policy == 'without_discount':
                    _price, rule_id = pricelist_item.base_pricelist_id.with_context(uom=uom.id).get_product_price_rule(product, qty, self.order_id.partner_id)
                    pricelist_item = PricelistItem.browse(rule_id)

            if pricelist_item.base == 'standard_price':
                field_name = 'standard_price'
                product_currency = product.cost_currency_id
            elif pricelist_item.base == 'pricelist' and pricelist_item.base_pricelist_id:
                field_name = 'price'
                product = product.with_context(pricelist=pricelist_item.base_pricelist_id.id)
                product_currency = pricelist_item.base_pricelist_id.currency_id
            currency_id = pricelist_item.pricelist_id.currency_id

        if not currency_id:
            currency_id = product_currency
            cur_factor = 1.0
        else:
            if currency_id.id == product_currency.id:
                cur_factor = 1.0
            else:
                cur_factor = currency_id._get_conversion_rate(product_currency, currency_id, self.company_id or self.env.company, self.order_id.date_order or fields.Date.today())

        product_uom = self.env.context.get('uom') or product.uom_id.id
        if uom and uom.id != product_uom:
            # the unit price is in a different uom
            uom_factor = uom._compute_price(1.0, product.uom_id)
        else:
            uom_factor = 1.0

        return product[field_name] * uom_factor * cur_factor, currency_id

    def _get_protected_fields(self):
        return [
            'product_id', 'name', 'price_unit', 'product_uom', 'product_uom_qty',
            'tax_id', 'analytic_tag_ids'
        ]

    def _onchange_product_id_set_customer_lead(self):
        pass

    @api.onchange('product_id', 'price_unit', 'product_uom', 'product_uom_qty', 'tax_id')
    def _onchange_discount(self):
        if not (self.product_id and self.product_uom and
                self.order_id.partner_id and self.order_id.pricelist_id and
                self.order_id.pricelist_id.discount_policy == 'without_discount' and
                self.env.user.has_group('product.group_discount_per_so_line')):
            return

        self.discount = 0.0
        product = self.product_id.with_context(
            lang=self.order_id.partner_id.lang,
            partner=self.order_id.partner_id,
            quantity=self.product_uom_qty,
            date=self.order_id.date_order,
            pricelist=self.order_id.pricelist_id.id,
            uom=self.product_uom.id,
            fiscal_position=self.env.context.get('fiscal_position')
        )

        product_context = dict(self.env.context, partner_id=self.order_id.partner_id.id, date=self.order_id.date_order, uom=self.product_uom.id)

        price, rule_id = self.order_id.pricelist_id.with_context(product_context).get_product_price_rule(self.product_id, self.product_uom_qty or 1.0, self.order_id.partner_id)
        new_list_price, currency = self.with_context(product_context)._get_real_price_currency(product, rule_id, self.product_uom_qty, self.product_uom, self.order_id.pricelist_id.id)

        if new_list_price != 0:
            if self.order_id.pricelist_id.currency_id != currency:
                # we need new_list_price in the same currency as price, which is in the SO's pricelist's currency
                new_list_price = currency._convert(
                    new_list_price, self.order_id.pricelist_id.currency_id,
                    self.order_id.company_id or self.env.company, self.order_id.date_order or fields.Date.today())
            discount = (new_list_price - price) / new_list_price * 100
            if (discount > 0 and new_list_price > 0) or (discount < 0 and new_list_price < 0):
                self.discount = discount

    def _is_delivery(self):
        self.ensure_one()
        return False

    def get_sale_order_line_multiline_description_sale(self, product):
        """ Compute a default multiline description for this sales order line.

        In most cases the product description is enough but sometimes we need to append information that only
        exists on the sale order line itself.
        e.g:
        - custom attributes and attributes that don't create variants, both introduced by the "product configurator"
        - in event_sale we need to know specifically the sales order line as well as the product to generate the name:
          the product is not sufficient because we also need to know the event_id and the event_ticket_id (both which belong to the sale order line).
        """
        return product.get_product_multiline_description_sale() + self._get_sale_order_line_multiline_description_variants()

    def _get_sale_order_line_multiline_description_variants(self):
        """When using no_variant attributes or is_custom values, the product
        itself is not sufficient to create the description: we need to add
        information about those special attributes and values.

        :return: the description related to special variant attributes/values
        :rtype: string
        """
        if not self.product_custom_attribute_value_ids and not self.product_no_variant_attribute_value_ids:
            return ""

        name = "\n"

        custom_ptavs = self.product_custom_attribute_value_ids.custom_product_template_attribute_value_id
        no_variant_ptavs = self.product_no_variant_attribute_value_ids._origin

        # display the no_variant attributes, except those that are also
        # displayed by a custom (avoid duplicate description)
        for ptav in (no_variant_ptavs - custom_ptavs):
            name += "\n" + ptav.display_name

        # Sort the values according to _order settings, because it doesn't work for virtual records in onchange
        custom_values = sorted(self.product_custom_attribute_value_ids, key=lambda r: (r.custom_product_template_attribute_value_id.id, r.id))
        # display the is_custom values
        for pacv in custom_values:
            name += "\n" + pacv.display_name

        return name

    def _is_not_sellable_line(self):
        # True if the line is a computed line (reward, delivery, ...) that user cannot add manually
        return False

```

## File: models\utm_campaign.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models

class UtmCampaign(models.Model):
    _inherit = 'utm.campaign'
    _description = 'UTM Campaign'

    quotation_count = fields.Integer('Quotation Count', groups='sales_team.group_sale_salesman', compute="_compute_quotation_count")
    invoiced_amount = fields.Integer(default=0, compute="_compute_sale_invoiced_amount", string="Revenues generated by the campaign")
    company_id = fields.Many2one('res.company', string='Company', readonly=True, states={'draft': [('readonly', False)], 'refused': [('readonly', False)]}, default=lambda self: self.env.company)
    currency_id = fields.Many2one('res.currency', related='company_id.currency_id', string='Currency')

    def _compute_quotation_count(self):
        quotation_data = self.env['sale.order'].read_group([
            ('campaign_id', 'in', self.ids)],
            ['campaign_id'], ['campaign_id'])
        data_map = {datum['campaign_id'][0]: datum['campaign_id_count'] for datum in quotation_data}
        for campaign in self:
            campaign.quotation_count = data_map.get(campaign.id, 0)

    def _compute_sale_invoiced_amount(self):
        self.env['account.move.line'].flush(['balance', 'move_id', 'account_id', 'exclude_from_invoice_tab'])
        self.env['account.move'].flush(['state', 'campaign_id', 'move_type'])
        query = """SELECT move.campaign_id, -SUM(line.balance) as price_subtotal
                    FROM account_move_line line
                    INNER JOIN account_move move ON line.move_id = move.id
                    WHERE move.state not in ('draft', 'cancel')
                        AND move.campaign_id IN %s
                        AND move.move_type IN ('out_invoice', 'out_refund', 'in_invoice', 'in_refund', 'out_receipt', 'in_receipt')
                        AND line.account_id IS NOT NULL
                        AND NOT line.exclude_from_invoice_tab
                    GROUP BY move.campaign_id
                    """

        self._cr.execute(query, [tuple(self.ids)])
        query_res = self._cr.dictfetchall()

        campaigns = self.browse()
        for datum in query_res:
            campaign = self.browse(datum['campaign_id'])
            campaign.invoiced_amount = datum['price_subtotal']
            campaigns |= campaign
        for campaign in (self - campaigns):
            campaign.invoiced_amount = 0

    def action_redirect_to_quotations(self):
        action = self.env["ir.actions.actions"]._for_xml_id("sale.action_quotations_with_onboarding")
        action['domain'] = [('campaign_id', '=', self.id)]
        action['context'] = {'default_campaign_id': self.id}
        return action

    def action_redirect_to_invoiced(self):
        action = self.env["ir.actions.actions"]._for_xml_id("account.action_move_journal_line")
        invoices = self.env['account.move'].search([('campaign_id', '=', self.id)])
        action['context'] = {
            'create': False,
            'edit': False,
            'view_no_maturity': True
        }
        action['domain'] = [
            ('id', 'in', invoices.ids),
            ('move_type', 'in', ('out_invoice', 'out_refund', 'in_invoice', 'in_refund', 'out_receipt', 'in_receipt')),
            ('state', 'not in', ['draft', 'cancel'])
        ]
        return action

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import analytic
from . import account_move
from . import account_move_line
from . import crm_team
from . import payment_acquirer
from . import payment_transaction
from . import product_product
from . import product_template
from . import res_company
from . import res_config_settings
from . import res_partner
from . import sale_order
from . import sale_order_line
from . import utm_campaign

```

## File: populate\product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.tools import populate


class ProductProduct(models.Model):
    _inherit = "product.product"

    def _populate_get_product_factories(self):
        """Populate the invoice_policy of product.product & product.template models."""
        return super()._populate_get_product_factories() + [
            ("invoice_policy", populate.randomize(['order', 'delivery'], [5, 5]))]

```

## File: populate\product_attribute.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.tools import populate


class ProductAttribute(models.Model):
    _inherit = "product.attribute"

    def _populate_factories(self):

        return super()._populate_factories() + [
            ("display_type", populate.randomize(['radio', 'select', 'color'], [6, 3, 1])),
        ]


class ProductAttributeValue(models.Model):
    _inherit = "product.attribute.value"

    def _populate_factories(self):
        attribute_ids = self.env.registry.populated_models["product.attribute"]
        color_attribute_ids = self.env["product.attribute"].search([
            ("id", "in", attribute_ids),
            ("display_type", "=", "color"),
        ]).ids

        def get_custom_values(iterator, field_group, model_name):
            r = populate.Random('%s+fields+%s' % (model_name, field_group))
            for _, values in enumerate(iterator):
                attribute_id = values.get("attribute_id")
                if attribute_id in color_attribute_ids:
                    values["html_color"] = r.choice(
                        ["#FFFFFF", "#000000", "#FFC300", "#1BC56D", "#FFFF00", "#FF0000"],
                    )
                elif not r.getrandbits(4):
                    values["is_custom"] = True
                yield values

        return super()._populate_factories() + [
            ("_custom_values", get_custom_values),
        ]

```

## File: populate\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import logging

from odoo import models
from odoo.tools import populate, groupby

_logger = logging.getLogger(__name__)


class SaleOrder(models.Model):
    _inherit = "sale.order"
    _populate_sizes = {"small": 100, "medium": 2_000, "large": 20_000}
    _populate_dependencies = ["res.partner", "res.company", "res.users", "product.pricelist"]

    def _populate_factories(self):
        company_ids = self.env.registry.populated_models["res.company"]

        def x_ids_by_company(recordset, with_false=True):
            x_by_company = dict(groupby(recordset, key=lambda x_record: x_record.company_id.id))
            if with_false:
                x_inter_company = self.env[recordset._name].concat(*x_by_company.get(False, []))
            else:
                x_inter_company = self.env[recordset._name]
            return {com: (self.env[recordset._name].concat(*x_records) | x_inter_company).ids for com, x_records in x_by_company.items() if com}

        partners_ids_by_company = x_ids_by_company(self.env["res.partner"].browse(self.env.registry.populated_models["res.partner"]))
        pricelist_ids_by_company = x_ids_by_company(self.env["product.pricelist"].browse(self.env.registry.populated_models["product.pricelist"]))
        user_ids_by_company = x_ids_by_company(self.env["res.users"].browse(self.env.registry.populated_models["res.users"]), with_false=False)

        def get_company_info(iterator, field_name, model_name):
            random = populate.Random("sale_order_company")
            for values in iterator:
                cid = values.get("company_id")
                valid_partner_ids = partners_ids_by_company[cid]
                valid_user_ids = user_ids_by_company[cid]
                valid_pricelist_ids = pricelist_ids_by_company[cid]
                values.update({
                    "partner_id": random.choice(valid_partner_ids),
                    "user_id": random.choice(valid_user_ids),
                    "pricelist_id": random.choice(valid_pricelist_ids),
                })
                yield values

        return [
            ("company_id", populate.randomize(company_ids)),
            ("_company_limited_fields", get_company_info),
            ("require_payment", populate.randomize([True, False])),
            ("require_signature", populate.randomize([True, False])),
        ]


class SaleOrderLine(models.Model):
    _inherit = "sale.order.line"
    _populate_sizes = {"small": 1_000, "medium": 50_000, "large": 100_000}
    _populate_dependencies = ["sale.order", "product.product"]

    def _populate(self, size):
        so_line = super()._populate(size)

        def confirm_sale_order(sample_ratio):
            # Confirm sample_ratio * 100 % of picking
            random = populate.Random('confirm_sale_order')
            order_ids = so_line.order_id.ids
            orders_to_confirm = self.env['sale.order'].browse(random.sample(order_ids, int(len(order_ids) * sample_ratio)))
            _logger.info("Confirm %d sale orders", len(orders_to_confirm))
            orders_to_confirm.action_confirm()
            return orders_to_confirm

        confirm_sale_order(0.50)

        return so_line

    def _populate_factories(self):
        order_ids = self.env.registry.populated_models["sale.order"]
        product_ids = self.env.registry.populated_models["product.product"]
        # If we want more advanced products with multiple variants
        # add a populate dependency on product template and the following lines
        product_ids += self.env["product.product"].search([
            ('product_tmpl_id', 'in', self.env.registry.populated_models["product.template"])
        ]).ids

        self.env['product.product'].browse(product_ids).read(['uom_id'])  # prefetch all uom_id

        def get_product_uom(values, counter, random):
            return self.env['product.product'].browse(values['product_id']).uom_id.id

        # TODO sections & notes (display_type & name)

        return [
            ("order_id", populate.randomize(order_ids)),
            ("product_id", populate.randomize(product_ids)),
            ("product_uom", populate.compute(get_product_uom)),
            ("product_uom_qty", populate.randint(1, 200)),
        ]

```

## File: populate\__init__.py

```python
from . import product_attribute
from . import sale_order
from . import product

```

## File: report\invoice_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class AccountInvoiceReport(models.Model):
    _inherit = 'account.invoice.report'

    team_id = fields.Many2one('crm.team', string='Sales Team')

    def _select(self):
        return super(AccountInvoiceReport, self)._select() + ", move.team_id as team_id"

```

## File: report\invoice_report_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="report_invoice_document_inherit_sale" inherit_id="account.report_invoice_document">
        <xpath expr="//address" position="attributes">
            <attribute name="groups">!sale.group_delivery_invoice_address</attribute>
        </xpath>
        <xpath expr="//address" position="before">
            <t t-if="o.partner_shipping_id and (o.partner_shipping_id != o.partner_id)">
                <t t-set="information_block">
                    <div groups="sale.group_delivery_invoice_address" name="shipping_address_block">
                        <strong>Shipping Address:</strong>
                        <div t-field="o.partner_shipping_id"
                            t-options='{"widget": "contact", "fields": ["address", "name"], "no_marker": True}'/>
                    </div>
                </t>
            </t>
            <div t-field="o.partner_id"
                t-options='{"widget": "contact", "fields": ["address", "name"], "no_marker": True}' groups="sale.group_delivery_invoice_address"/>
        </xpath>
    </template>
</odoo>

```

## File: report\report_all_channels_sales.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, tools


class PosSaleReport(models.Model):
    _name = "report.all.channels.sales"
    _description = "Sales by Channel (All in One)"
    _auto = False

    name = fields.Char('Order Reference', readonly=True)
    partner_id = fields.Many2one('res.partner', 'Partner', readonly=True)
    product_id = fields.Many2one('product.product', string='Product', readonly=True)
    product_tmpl_id = fields.Many2one('product.template', 'Product Template', readonly=True)
    date_order = fields.Datetime(string='Date Order', readonly=True)
    user_id = fields.Many2one('res.users', 'Salesperson', readonly=True)
    categ_id = fields.Many2one('product.category', 'Product Category', readonly=True)
    company_id = fields.Many2one('res.company', 'Company', readonly=True)
    price_total = fields.Float('Total', readonly=True)
    pricelist_id = fields.Many2one('product.pricelist', 'Pricelist', readonly=True)
    country_id = fields.Many2one('res.country', 'Partner Country', readonly=True)
    price_subtotal = fields.Float(string='Price Subtotal', readonly=True)
    product_qty = fields.Float('Product Quantity', readonly=True)
    analytic_account_id = fields.Many2one('account.analytic.account', 'Analytic Account', readonly=True)
    team_id = fields.Many2one('crm.team', 'Sales Team', readonly=True)

    def _so(self):
        so_str = """
                SELECT sol.id AS id,
                    so.name AS name,
                    so.partner_id AS partner_id,
                    sol.product_id AS product_id,
                    pro.product_tmpl_id AS product_tmpl_id,
                    so.date_order AS date_order,
                    so.user_id AS user_id,
                    pt.categ_id AS categ_id,
                    so.company_id AS company_id,
                    sol.price_total / CASE COALESCE(so.currency_rate, 0) WHEN 0 THEN 1.0 ELSE so.currency_rate END AS price_total,
                    so.pricelist_id AS pricelist_id,
                    rp.country_id AS country_id,
                    sol.price_subtotal / CASE COALESCE(so.currency_rate, 0) WHEN 0 THEN 1.0 ELSE so.currency_rate END AS price_subtotal,
                    (sol.product_uom_qty / u.factor * u2.factor) as product_qty,
                    so.analytic_account_id AS analytic_account_id,
                    so.team_id AS team_id

            FROM sale_order_line sol
                    JOIN sale_order so ON (sol.order_id = so.id)
                    LEFT JOIN product_product pro ON (sol.product_id = pro.id)
                    JOIN res_partner rp ON (so.partner_id = rp.id)
                    LEFT JOIN product_template pt ON (pro.product_tmpl_id = pt.id)
                    LEFT JOIN product_pricelist pp ON (so.pricelist_id = pp.id)
                    LEFT JOIN uom_uom u on (u.id=sol.product_uom)
                    LEFT JOIN uom_uom u2 on (u2.id=pt.uom_id)
            WHERE so.state in ('sale','done')
        """
        return so_str

    def _from(self):
        return """(%s)""" % (self._so())

    def _get_main_request(self):
        request = """
            CREATE or REPLACE VIEW %s AS
                SELECT id AS id,
                    name,
                    partner_id,
                    product_id,
                    product_tmpl_id,
                    date_order,
                    user_id,
                    categ_id,
                    company_id,
                    price_total,
                    pricelist_id,
                    analytic_account_id,
                    country_id,
                    team_id,
                    price_subtotal,
                    product_qty
                FROM %s
                AS foo""" % (self._table, self._from())
        return request

    def init(self):
        tools.drop_view_if_exists(self.env.cr, self._table)
        self.env.cr.execute(self._get_main_request())

```

## File: report\report_all_channels_sales_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="report_all_channels_sales_action" model="ir.actions.act_window">
        <field name="name">Sales Analysis</field>
        <field name="res_model">sale.report</field>
        <field name="view_mode">pivot</field>
    </record>
</odoo>

```

## File: report\sale_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import tools
from odoo import api, fields, models


class SaleReport(models.Model):
    _name = "sale.report"
    _description = "Sales Analysis Report"
    _auto = False
    _rec_name = 'date'
    _order = 'date desc'

    @api.model
    def _get_done_states(self):
        return ['sale', 'done', 'paid']

    name = fields.Char('Order Reference', readonly=True)
    date = fields.Datetime('Order Date', readonly=True)
    product_id = fields.Many2one('product.product', 'Product Variant', readonly=True)
    product_uom = fields.Many2one('uom.uom', 'Unit of Measure', readonly=True)
    product_uom_qty = fields.Float('Qty Ordered', readonly=True)
    qty_to_deliver = fields.Float('Qty To Deliver', readonly=True)
    qty_delivered = fields.Float('Qty Delivered', readonly=True)
    qty_to_invoice = fields.Float('Qty To Invoice', readonly=True)
    qty_invoiced = fields.Float('Qty Invoiced', readonly=True)
    partner_id = fields.Many2one('res.partner', 'Customer', readonly=True)
    company_id = fields.Many2one('res.company', 'Company', readonly=True)
    user_id = fields.Many2one('res.users', 'Salesperson', readonly=True)
    price_total = fields.Float('Total', readonly=True)
    price_subtotal = fields.Float('Untaxed Total', readonly=True)
    untaxed_amount_to_invoice = fields.Float('Untaxed Amount To Invoice', readonly=True)
    untaxed_amount_invoiced = fields.Float('Untaxed Amount Invoiced', readonly=True)
    product_tmpl_id = fields.Many2one('product.template', 'Product', readonly=True)
    categ_id = fields.Many2one('product.category', 'Product Category', readonly=True)
    nbr = fields.Integer('# of Lines', readonly=True)
    pricelist_id = fields.Many2one('product.pricelist', 'Pricelist', readonly=True)
    analytic_account_id = fields.Many2one('account.analytic.account', 'Analytic Account', readonly=True)
    team_id = fields.Many2one('crm.team', 'Sales Team', readonly=True)
    country_id = fields.Many2one('res.country', 'Customer Country', readonly=True)
    industry_id = fields.Many2one('res.partner.industry', 'Customer Industry', readonly=True)
    commercial_partner_id = fields.Many2one('res.partner', 'Customer Entity', readonly=True)
    state = fields.Selection([
        ('draft', 'Draft Quotation'),
        ('sent', 'Quotation Sent'),
        ('sale', 'Sales Order'),
        ('done', 'Sales Done'),
        ('cancel', 'Cancelled'),
        ], string='Status', readonly=True)
    weight = fields.Float('Gross Weight', readonly=True)
    volume = fields.Float('Volume', readonly=True)

    discount = fields.Float('Discount %', readonly=True)
    discount_amount = fields.Float('Discount Amount', readonly=True)
    campaign_id = fields.Many2one('utm.campaign', 'Campaign')
    medium_id = fields.Many2one('utm.medium', 'Medium')
    source_id = fields.Many2one('utm.source', 'Source')

    order_id = fields.Many2one('sale.order', 'Order #', readonly=True)

    def _select_sale(self, fields=None):
        if not fields:
            fields = {}
        select_ = """
            min(l.id) as id,
            l.product_id as product_id,
            t.uom_id as product_uom,
            CASE WHEN l.product_id IS NOT NULL THEN sum(l.product_uom_qty / u.factor * u2.factor) ELSE 0 END as product_uom_qty,
            CASE WHEN l.product_id IS NOT NULL THEN sum(l.qty_delivered / u.factor * u2.factor) ELSE 0 END as qty_delivered,
            CASE WHEN l.product_id IS NOT NULL THEN SUM((l.product_uom_qty - l.qty_delivered) / u.factor * u2.factor) ELSE 0 END as qty_to_deliver,
            CASE WHEN l.product_id IS NOT NULL THEN sum(l.qty_invoiced / u.factor * u2.factor) ELSE 0 END as qty_invoiced,
            CASE WHEN l.product_id IS NOT NULL THEN sum(l.qty_to_invoice / u.factor * u2.factor) ELSE 0 END as qty_to_invoice,
            CASE WHEN l.product_id IS NOT NULL THEN sum(l.price_total / CASE COALESCE(s.currency_rate, 0) WHEN 0 THEN 1.0 ELSE s.currency_rate END) ELSE 0 END as price_total,
            CASE WHEN l.product_id IS NOT NULL THEN sum(l.price_subtotal / CASE COALESCE(s.currency_rate, 0) WHEN 0 THEN 1.0 ELSE s.currency_rate END) ELSE 0 END as price_subtotal,
            CASE WHEN l.product_id IS NOT NULL THEN sum(l.untaxed_amount_to_invoice / CASE COALESCE(s.currency_rate, 0) WHEN 0 THEN 1.0 ELSE s.currency_rate END) ELSE 0 END as untaxed_amount_to_invoice,
            CASE WHEN l.product_id IS NOT NULL THEN sum(l.untaxed_amount_invoiced / CASE COALESCE(s.currency_rate, 0) WHEN 0 THEN 1.0 ELSE s.currency_rate END) ELSE 0 END as untaxed_amount_invoiced,
            count(*) as nbr,
            s.name as name,
            s.date_order as date,
            s.state as state,
            s.partner_id as partner_id,
            s.user_id as user_id,
            s.company_id as company_id,
            s.campaign_id as campaign_id,
            s.medium_id as medium_id,
            s.source_id as source_id,
            extract(epoch from avg(date_trunc('day',s.date_order)-date_trunc('day',s.create_date)))/(24*60*60)::decimal(16,2) as delay,
            t.categ_id as categ_id,
            s.pricelist_id as pricelist_id,
            s.analytic_account_id as analytic_account_id,
            s.team_id as team_id,
            p.product_tmpl_id,
            partner.country_id as country_id,
            partner.industry_id as industry_id,
            partner.commercial_partner_id as commercial_partner_id,
            CASE WHEN l.product_id IS NOT NULL THEN sum(p.weight * l.product_uom_qty / u.factor * u2.factor) ELSE 0 END as weight,
            CASE WHEN l.product_id IS NOT NULL THEN sum(p.volume * l.product_uom_qty / u.factor * u2.factor) ELSE 0 END as volume,
            l.discount as discount,
            CASE WHEN l.product_id IS NOT NULL THEN sum((l.price_unit * l.product_uom_qty * l.discount / 100.0 / CASE COALESCE(s.currency_rate, 0) WHEN 0 THEN 1.0 ELSE s.currency_rate END))ELSE 0 END as discount_amount,
            s.id as order_id
        """

        for field in fields.values():
            select_ += field
        return select_

    def _from_sale(self, from_clause=''):
        from_ = """
                sale_order_line l
                      left join sale_order s on (s.id=l.order_id)
                      join res_partner partner on s.partner_id = partner.id
                        left join product_product p on (l.product_id=p.id)
                            left join product_template t on (p.product_tmpl_id=t.id)
                    left join uom_uom u on (u.id=l.product_uom)
                    left join uom_uom u2 on (u2.id=t.uom_id)
                    left join product_pricelist pp on (s.pricelist_id = pp.id)
                %s
        """ % from_clause
        return from_

    def _group_by_sale(self, groupby=''):
        groupby_ = """
            l.product_id,
            l.order_id,
            t.uom_id,
            t.categ_id,
            s.name,
            s.date_order,
            s.partner_id,
            s.user_id,
            s.state,
            s.company_id,
            s.campaign_id,
            s.medium_id,
            s.source_id,
            s.pricelist_id,
            s.analytic_account_id,
            s.team_id,
            p.product_tmpl_id,
            partner.country_id,
            partner.industry_id,
            partner.commercial_partner_id,
            l.discount,
            s.id %s
        """ % (groupby)
        return groupby_

    def _select_additional_fields(self, fields):
        """Hook to return additional fields SQL specification for select part of the table query.

        :param dict fields: additional fields info provided by _query overrides (old API), prefer overriding
            _select_additional_fields instead.
        :returns: mapping field -> SQL computation of the field
        :rtype: dict
        """
        return fields

    def _query(self, with_clause='', fields=None, groupby='', from_clause=''):
        if not fields:
            fields = {}
        sale_report_fields = self._select_additional_fields(fields)
        with_ = ("WITH %s" % with_clause) if with_clause else ""
        return '%s (SELECT %s FROM %s WHERE l.display_type IS NULL GROUP BY %s)' % \
               (with_, self._select_sale(sale_report_fields), self._from_sale(from_clause), self._group_by_sale(groupby))

    def init(self):
        # self._table = sale_report
        tools.drop_view_if_exists(self.env.cr, self._table)
        self.env.cr.execute("""CREATE or REPLACE VIEW %s as (%s)""" % (self._table, self._query()))

class SaleOrderReportProforma(models.AbstractModel):
    _name = 'report.sale.report_saleproforma'
    _description = 'Proforma Report'

    @api.model
    def _get_report_values(self, docids, data=None):
        docs = self.env['sale.order'].browse(docids)
        return {
            'doc_ids': docs.ids,
            'doc_model': 'sale.order',
            'docs': docs,
            'proforma': True
        }

```

## File: report\sale_report.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="action_report_saleorder" model="ir.actions.report">
            <field name="name">Quotation / Order</field>
            <field name="model">sale.order</field>
            <field name="report_type">qweb-pdf</field>
            <field name="report_name">sale.report_saleorder</field>
            <field name="report_file">sale.report_saleorder</field>
            <field name="print_report_name">(object.state in ('draft', 'sent') and 'Quotation - %s' % (object.name)) or 'Order - %s' % (object.name)</field>
            <field name="binding_model_id" ref="model_sale_order"/>
            <field name="binding_type">report</field>
        </record>

        <record id="action_report_pro_forma_invoice" model="ir.actions.report">
            <field name="name">PRO-FORMA Invoice</field>
            <field name="model">sale.order</field>
            <field name="report_type">qweb-pdf</field>
            <field name="report_name">sale.report_saleorder_pro_forma</field>
            <field name="report_file">sale.report_saleorder_pro_forma</field>
            <field name="print_report_name">'PRO-FORMA - %s' % (object.name)</field>
            <field name="binding_model_id" ref="model_sale_order"/>
            <field name="binding_type">report</field>
            <field name="groups_id" eval="[(4, ref('sale.group_proforma_sales'))]"/>
        </record>
    </data>
</odoo>

```

## File: report\sale_report_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<template id="report_saleorder_document">
    <t t-call="web.external_layout">
        <t t-set="doc" t-value="doc.with_context(lang=doc.partner_id.lang)" />
        <t t-set="forced_vat" t-value="doc.fiscal_position_id.foreign_vat"/> <!-- So that it appears in the footer of the report instead of the company VAT if it's set -->
        <t t-set="address">
            <div t-field="doc.partner_id"
                t-options='{"widget": "contact", "fields": ["address", "name"], "no_marker": True}' />
            <p t-if="doc.partner_id.vat"><t t-esc="doc.company_id.account_fiscal_country_id.vat_label or 'Tax ID'"/>: <span t-field="doc.partner_id.vat"/></p>
        </t>
        <t t-if="doc.partner_shipping_id == doc.partner_invoice_id
                             and doc.partner_invoice_id != doc.partner_id
                             or doc.partner_shipping_id != doc.partner_invoice_id">
            <t t-set="information_block">
                <strong t-if="doc.partner_shipping_id == doc.partner_invoice_id">Invoicing and Shipping Address:</strong>
                <strong t-if="doc.partner_shipping_id != doc.partner_invoice_id">Invoicing Address:</strong>
                <div t-field="doc.partner_invoice_id"
                t-options='{"widget": "contact", "fields": ["address", "name", "phone"], "no_marker": True, "phone_icons": True}'/>
                <t t-if="doc.partner_shipping_id != doc.partner_invoice_id">
                    <strong>Shipping Address:</strong>
                    <div t-field="doc.partner_shipping_id"
                        t-options='{"widget": "contact", "fields": ["address", "name", "phone"], "no_marker": True, "phone_icons": True}'/>
                </t>
            </t>
        </t>
        <div class="page">
            <div class="oe_structure"/>

            <h2 class="mt16">
                <t t-if="not (env.context.get('proforma', False) or is_pro_forma)">
                    <span t-if="doc.state not in ['draft','sent']">Order # </span>
                    <span t-if="doc.state in ['draft','sent']">Quotation # </span>
                </t>
                <t t-if="env.context.get('proforma', False) or is_pro_forma">
                    <span>Pro-Forma Invoice # </span>
                </t>
                <span t-field="doc.name"/>
            </h2>

            <div class="row mt32 mb32" id="informations">
                <div t-if="doc.client_order_ref" class="col-auto col-3 mw-100 mb-2">
                    <strong>Your Reference:</strong>
                    <p class="m-0" t-field="doc.client_order_ref"/>
                </div>
                <div t-if="doc.date_order and doc.state not in ['draft','sent']" class="col-auto col-3 mw-100 mb-2">
                    <strong>Order Date:</strong>
                    <p class="m-0" t-field="doc.date_order"/>
                </div>
                <div t-if="doc.date_order and doc.state in ['draft','sent']" class="col-auto col-3 mw-100 mb-2">
                    <strong>Quotation Date:</strong>
                    <p class="m-0" t-field="doc.date_order" t-options='{"widget": "date"}'/>
                </div>
                <div t-if="doc.validity_date and doc.state in ['draft', 'sent']" class="col-auto col-3 mw-100 mb-2" name="expiration_date">
                    <strong>Expiration:</strong>
                    <p class="m-0" t-field="doc.validity_date"/>
                </div>
                <div t-if="doc.user_id.name" class="col-auto col-3 mw-100 mb-2">
                    <strong>Salesperson:</strong>
                    <p class="m-0" t-field="doc.user_id"/>
                </div>
            </div>

            <!-- Is there a discount on at least one line? -->
            <t t-set="display_discount" t-value="any(l.discount for l in doc.order_line)"/>

            <table class="table table-sm o_main_table">
                <!-- In case we want to repeat the header, remove "display: table-row-group" -->
                <thead style="display: table-row-group">
                    <tr>
                        <th name="th_description" class="text-left">Description</th>
                        <th name="th_quantity" class="text-right">Quantity</th>
                        <th name="th_priceunit" class="text-right">Unit Price</th>
                        <th name="th_discount" t-if="display_discount" class="text-right" groups="product.group_discount_per_so_line">
                            <span>Disc.%</span>
                        </th>
                        <th name="th_taxes" class="text-right">Taxes</th>
                        <th name="th_subtotal" class="text-right">
                            <span groups="account.group_show_line_subtotals_tax_excluded">Amount</span>
                            <span groups="account.group_show_line_subtotals_tax_included">Total Price</span>
                        </th>
                    </tr>
                </thead>
                <tbody class="sale_tbody">

                    <t t-set="current_subtotal" t-value="0"/>

                    <t t-foreach="doc.order_line" t-as="line">

                        <t t-set="current_subtotal" t-value="current_subtotal + line.price_subtotal" groups="account.group_show_line_subtotals_tax_excluded"/>
                        <t t-set="current_subtotal" t-value="current_subtotal + line.price_total" groups="account.group_show_line_subtotals_tax_included"/>

                        <tr t-att-class="'bg-200 font-weight-bold o_line_section' if line.display_type == 'line_section' else 'font-italic o_line_note' if line.display_type == 'line_note' else ''">
                            <t t-if="not line.display_type">
                                <td name="td_name"><span t-field="line.name"/></td>
                                <td name="td_quantity" class="text-right">
                                    <span t-field="line.product_uom_qty"/>
                                    <span t-field="line.product_uom"/>
                                </td>
                                <td name="td_priceunit" class="text-right">
                                    <span t-field="line.price_unit"/>
                                </td>
                                <td t-if="display_discount" class="text-right" groups="product.group_discount_per_so_line">
                                    <span t-field="line.discount"/>
                                </td>
                                <td name="td_taxes" class="text-right">
                                    <span t-esc="', '.join(map(lambda x: (x.description or x.name), line.tax_id))"/>
                                </td>
                                <td name="td_subtotal" class="text-right o_price_total">
                                    <span t-field="line.price_subtotal" groups="account.group_show_line_subtotals_tax_excluded"/>
                                    <span t-field="line.price_total" groups="account.group_show_line_subtotals_tax_included"/>
                                </td>
                            </t>
                            <t t-if="line.display_type == 'line_section'">
                                <td name="td_section_line" colspan="99">
                                    <span t-field="line.name"/>
                                </td>
                                <t t-set="current_section" t-value="line"/>
                                <t t-set="current_subtotal" t-value="0"/>
                            </t>
                            <t t-if="line.display_type == 'line_note'">
                                <td name="td_note_line" colspan="99">
                                    <span t-field="line.name"/>
                                </td>
                            </t>
                        </tr>

                        <t t-if="current_section and (line_last or doc.order_line[line_index+1].display_type == 'line_section')">
                            <tr class="is-subtotal text-right">
                                <td name="td_section_subtotal" colspan="99">
                                    <strong class="mr16">Subtotal</strong>
                                    <span
                                        t-esc="current_subtotal"
                                        t-options='{"widget": "monetary", "display_currency": doc.pricelist_id.currency_id}'
                                    />
                                </td>
                            </tr>
                        </t>
                    </t>
                </tbody>
            </table>

            <div class="clearfix" name="so_total_summary">
                <div id="total" class="row" name="total">
                    <div t-attf-class="#{'col-6' if report_type != 'html' else 'col-sm-7 col-md-6'} ml-auto">
                        <table class="table table-sm">
                            <!-- Tax totals -->
                            <t t-set="tax_totals" t-value="json.loads(doc.tax_totals_json)"/>
                            <t t-call="account.document_tax_totals"/>
                        </table>
                    </div>
                </div>
            </div>

            <div t-if="doc.signature" class="mt32 ml64 mr4" name="signature">
                <div class="offset-8">
                    <strong>Signature</strong>
                </div>
                <div class="offset-8">
                    <img t-att-src="image_data_uri(doc.signature)" style="max-height: 4cm; max-width: 8cm;"/>
                </div>
                <div class="offset-8 text-center">
                    <p t-field="doc.signed_by"/>
                </div>
            </div>

            <div class="oe_structure"/>

            <p t-field="doc.note" />
            <p t-if="not is_html_empty(doc.payment_term_id.note)">
                <span t-field="doc.payment_term_id.note"/>
            </p>
            <p id="fiscal_position_remark" t-if="doc.fiscal_position_id and not is_html_empty(doc.fiscal_position_id.sudo().note)">
                <strong>Fiscal Position Remark:</strong>
                <span t-field="doc.fiscal_position_id.sudo().note"/>
            </p>
        </div>
    </t>
</template>


<template id="report_saleorder">
    <t t-call="web.html_container">
        <t t-foreach="docs" t-as="doc">
            <t t-call="sale.report_saleorder_document" t-lang="doc.partner_id.lang"/>
        </t>
    </t>
</template>

<template id="report_saleorder_pro_forma">
    <t t-call="web.html_container">
        <t t-set="is_pro_forma" t-value="True"/>
        <t t-set="docs" t-value="docs.with_context(proforma=True)"/>
        <t t-foreach="docs" t-as="doc">
            <t t-call="sale.report_saleorder_document" t-lang="doc.partner_id.lang"/>
        </t>
    </t>
</template>

</odoo>

```

## File: report\sale_report_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_order_product_pivot" model="ir.ui.view">
         <field name="name">sale.report.pivot</field>
         <field name="model">sale.report</field>
         <field name="arch" type="xml">
             <pivot string="Sales Analysis" sample="1">
                 <field name="team_id" type="col"/>
                 <field name="date" interval="month" type="row"/>
                 <field name="price_subtotal" type="measure"/>
             </pivot>
         </field>
    </record>

    <record id="view_order_product_graph" model="ir.ui.view">
         <field name="name">sale.report.graph</field>
         <field name="model">sale.report</field>
         <field name="arch" type="xml">
             <graph string="Sales Analysis" type="line" sample="1">
                 <field name="date" interval="day"/>
                 <field name="price_subtotal" type="measure"/>
             </graph>
         </field>
    </record>

    <record id="sale_report_view_tree" model="ir.ui.view">
        <field name="name">sale.report.view.tree</field>
        <field name="model">sale.report</field>
        <field name="arch" type="xml">
            <tree string="Sales Analysis">
                <field name="date" widget="date"/>
                <field name="order_id" optional="show"/>
                <field name="partner_id" optional="hide"/>
                <field name="user_id" optional="show" widget="many2one_avatar_user"/>
                <field name="team_id" optional="show"/>
                <field name="company_id" optional="show" groups="base.group_multi_company"/>
                <field name="price_subtotal" optional="hide" sum="Sum of Untaxed Total"/>
                <field name="price_total" optional="show" sum="Sum of Total"/>
                <field name="state" optional="hide"/>
            </tree>
        </field>
    </record>

    <record id="view_order_product_search" model="ir.ui.view">
        <field name="name">sale.report.search</field>
        <field name="model">sale.report</field>
        <field name="arch" type="xml">
            <search string="Sales Analysis">
                <field name="date"/>
                <filter string="Date" name="year" invisible="1" date="date" default_period="this_year"/>
                <filter name="Quotations" string="Quotations" domain="[('state','in', ('draft', 'sent'))]"/>
                <filter name="Sales" string="Sales Orders" domain="[('state','not in',('draft', 'cancel', 'sent'))]"/>
                <separator/>
                <filter name="filter_date" date="date" default_period="this_month"/>
                <filter name="filter_order_date" invisible="1" string="Order Date: Last 365 Days" domain="[('date', '&gt;=', (datetime.datetime.combine(context_today() + relativedelta(days=-365), datetime.time(0,0,0))).strftime('%Y-%m-%d %H:%M:%S'))]"/>
                <separator/>
                <field name="user_id"/>
                <field name="team_id"/>
                <field name="product_id"/>
                <field name="categ_id"/>
                <field name="partner_id"/>
                <field name="country_id"/>
                <field name="industry_id"/>
                <group expand="0" string="Extended Filters">
                    <field name="categ_id" filter_domain="[('categ_id', 'child_of', self)]"/>
                    <field name="company_id" groups="base.group_multi_company"/>
                </group>
                <group expand="1" string="Group By">
                    <filter string="Salesperson" name="User" context="{'group_by':'user_id'}"/>
                    <filter string="Sales Team" name="sales_channel" context="{'group_by':'team_id'}"/>
                    <filter string="Customer" name="Customer" context="{'group_by':'partner_id'}"/>
                    <filter string="Customer Country" name="country_id" context="{'group_by':'country_id'}"/>
                    <filter string="Customer Industry" name="industry_id" context="{'group_by':'industry_id'}"/>
                    <filter string="Product" name="Product" context="{'group_by':'product_id'}"/>
                    <filter string="Product Category" name="Category" context="{'group_by':'categ_id'}"/>
                    <filter name="status" string="Status" context="{'group_by':'state'}"/>
                    <filter string="Company" name="company" groups="base.group_multi_company" context="{'group_by':'company_id'}"/>
                    <separator/>
                    <filter string="Order Date" name="date" context="{'group_by':'date'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="action_order_report_all" model="ir.actions.act_window">
        <field name="name">Sales Analysis</field>
        <field name="res_model">sale.report</field>
        <field name="view_mode">graph,pivot</field>
        <field name="view_id"></field>  <!-- force empty -->
        <field name="search_view_id" ref="view_order_product_search"/>
        <field name="context">{'search_default_Sales':1, 'group_by_no_leaf':1,'group_by':[], 'search_default_filter_order_date': 1}</field>
        <field name="help">This report performs analysis on your quotations and sales orders. Analysis check your sales revenues and sort it by different group criteria (salesman, partner, product, etc.) Use this report to perform analysis on sales not having invoiced yet. If you want to analyse your turnover, you should use the Invoice Analysis report in the Accounting application.</field>
    </record>
</odoo>

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import sale_report
from . import invoice_report
from . import report_all_channels_sales

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_sale_order,sale.order,model_sale_order,sales_team.group_sale_salesman,1,1,1,0
access_sale_order_line,sale.order.line,model_sale_order_line,sales_team.group_sale_salesman,1,1,1,1
access_sale_order_line_readonly,sale.order.line accountant,model_sale_order_line,account.group_account_readonly,1,0,0,0
access_sale_order_line_accountant,sale.order.line accountant,model_sale_order_line,account.group_account_user,1,1,0,0
access_sale_order_portal,sale.order.portal,sale.model_sale_order,base.group_portal,1,0,0,0
access_sale_order_line_portal,sale.order.line.portal,sale.model_sale_order_line,base.group_portal,1,0,0,0
access_account_move_manager,account_move manager,account.model_account_move,sales_team.group_sale_manager,1,0,0,0
access_account_move_salesman,account_move salesman,account.model_account_move,sales_team.group_sale_salesman,1,0,0,0
access_account_move_line_salesman,account_move_line salesman,account.model_account_move_line,sales_team.group_sale_salesman,1,0,0,0
access_account_partial_reconcile_salesman,account_partial_reconcile salesman,account.model_account_partial_reconcile,sales_team.group_sale_salesman,1,0,0,0
access_account_payment_term_salesman,account_payment_term salesman,account.model_account_payment_term,sales_team.group_sale_salesman,1,0,0,0
access_account_account_tag_sale_salesman,account.account.tag.sale.salesman,account.model_account_account_tag,sales_team.group_sale_salesman,1,0,0,0
access_account_account_type_sale_salesman,account.account.type.sale.salesman,account.model_account_account_type,sales_team.group_sale_salesman,1,0,0,0
access_account_analytic_tag_sale_salesman,account.analytic.tag.sale.salesman,analytic.model_account_analytic_tag,sales_team.group_sale_salesman,1,0,0,0
access_account_analytic_account_salesman,account_analytic_account salesman,analytic.model_account_analytic_account,sales_team.group_sale_salesman,1,1,1,0
access_account_invoice_send_salesman,access.account.invoice.send.salesman,account.model_account_invoice_send,sales_team.group_sale_salesman,1,1,1,0
access_sale_order_manager,sale.order.manager,model_sale_order,sales_team.group_sale_manager,1,1,1,1
access_sale_order_readonly,sale.order.accountant,model_sale_order,account.group_account_readonly,1,0,0,0
access_sale_order_accountant,sale.order.accountant,model_sale_order,account.group_account_user,1,1,0,0
access_sale_report_salesman,sale.report,model_sale_report,sales_team.group_sale_salesman,1,1,1,0
access_sale_report_manager,sale.report,model_sale_report,sales_team.group_sale_manager,1,1,1,1
access_sale_account_journal,account.journal sale order.user,account.model_account_journal,sales_team.group_sale_salesman,1,0,0,0
access_res_partner_sale_user,res.partner.sale.user,base.model_res_partner,sales_team.group_sale_salesman,1,0,0,0
access_res_partner_sale_manager,res.partner.sale.manager,base.model_res_partner,sales_team.group_sale_manager,1,1,1,0
access_product_template_sale_user,product.template sale use,product.model_product_template,sales_team.group_sale_salesman,1,0,0,0
access_product_product_sale_user,product.product sale use,product.model_product_product,sales_team.group_sale_salesman,1,0,0,0
access_account_tax_user,account.tax.user,account.model_account_tax,sales_team.group_sale_salesman,1,0,0,0
access_uom_uom_user,uom.uom.user,uom.model_uom_uom,sales_team.group_sale_salesman,1,0,0,0
access_product_pricelist_sale_user,product.pricelist.sale.user,product.model_product_pricelist,sales_team.group_sale_salesman,1,0,0,0
access_account_account_salesman,account_account salesman,account.model_account_account,sales_team.group_sale_salesman,1,0,0,0
access_uom_category_sale_manager,uom.category salemanager,uom.model_uom_category,sales_team.group_sale_manager,1,1,1,1
access_uom_uom_sale_manager,uom.uom salemanager,uom.model_uom_uom,sales_team.group_sale_manager,1,1,1,1
access_product_category_sale_manager,product.category salemanager,product.model_product_category,sales_team.group_sale_manager,1,1,1,1
access_product_supplierinfo_user,product.supplierinfo.user,product.model_product_supplierinfo,sales_team.group_sale_salesman,1,0,0,0
access_product_supplierinfo_sale_manager,product.supplierinfo salemanager,product.model_product_supplierinfo,sales_team.group_sale_manager,1,1,1,1
access_product_pricelist_sale_manager,product.pricelist salemanager,product.model_product_pricelist,sales_team.group_sale_manager,1,1,1,1
access_sale_order_invoicing_payments,sale.order,model_sale_order,account.group_account_invoice,1,1,0,0
access_sale_order_line_invoicing_payments,sale.order.line,model_sale_order_line,account.group_account_invoice,1,1,0,0
access_product_pricelist_item_sale_manager,product.pricelist.item salemanager,product.model_product_pricelist_item,sales_team.group_sale_manager,1,1,1,1
access_product_template_sale_manager,product.template salemanager,model_product_template,sales_team.group_sale_manager,1,1,1,1
access_product_product_sale_manager,product.product salemanager,model_product_product,sales_team.group_sale_manager,1,1,1,1
access_product_attribute_sale_manager,product.attribute manager,product.model_product_attribute,sales_team.group_sale_manager,1,1,1,1
access_product_attribute_value_sale_manager,product.attribute manager value,product.model_product_attribute_value,sales_team.group_sale_manager,1,1,1,1
access_product_product_attribute_sale_manager,product.template.attribute manager value,product.model_product_template_attribute_value,sales_team.group_sale_manager,1,1,1,1
access_product_product_attribute_custom_value_sale_manager,product.attribute.custom value manager,product.model_product_attribute_custom_value,sales_team.group_sale_salesman,1,1,1,1
access_product_template_attribute_exclusion_sale_manager,product.attribute manager filter line,product.model_product_template_attribute_exclusion,sales_team.group_sale_manager,1,1,1,1
access_product_template_attribute_line_sale_manager,product.attribute manager line,product.model_product_template_attribute_line,sales_team.group_sale_manager,1,1,1,1
access_account_tax_sale_manager,account.tax sale manager,account.model_account_tax,sales_team.group_sale_salesman,1,0,0,0
access_account_tax_group_sale_manager,account.tax.group sale manager,account.model_account_tax_group,sales_team.group_sale_salesman,1,0,0,0
access_account_account_sale_manager,account.account sale manager,account.model_account_account,sales_team.group_sale_manager,1,0,0,0
access_mail_activity_type_sale_manager,mail.activity.type.sale.manager,mail.model_mail_activity_type,sales_team.group_sale_manager,1,1,1,1
access_report_all_channels_sales,access_report_all_channels_sales,model_report_all_channels_sales,sales_team.group_sale_manager,1,0,0,0
access_sale_payment_acquirer_onboarding_wizard,access.sale.payment.acquirer.onboarding.wizard,model_sale_payment_acquirer_onboarding_wizard,base.group_system,1,1,1,0
access_sale_advance_payment_inv,access.sale.advance.payment.inv,model_sale_advance_payment_inv,sales_team.group_sale_salesman,1,1,1,0
access_sale_order_cancel,access.sale.order.cancel,model_sale_order_cancel,sales_team.group_sale_salesman,1,1,1,0
access_payment_link_wizard_sale,access.payment.link.wizard.sale,model_payment_link_wizard,sales_team.group_sale_salesman,1,1,1,0

```

## File: security\sale_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

     <record id="group_auto_done_setting" model="res.groups">
        <field name="name">Lock Confirmed Sales</field>
        <field name="category_id" ref="base.module_category_hidden"/>
    </record>

    <record id="group_delivery_invoice_address" model="res.groups">
        <field name="name">Addresses in Sales Orders</field>
        <field name="category_id" ref="base.module_category_hidden"/>
    </record>

    <record id="group_warning_sale" model="res.groups">
        <field name="name">A warning can be set on a product or a customer (Sale)</field>
        <field name="category_id" ref="base.module_category_hidden"/>
    </record>

    <record id="group_proforma_sales" model="res.groups">
        <field name="name">Pro-forma Invoices</field>
        <field name="category_id" ref="base.module_category_hidden"/>
    </record>

    <record model="res.users" id="base.user_root">
        <field eval="[(4,ref('base.group_partner_manager'))]" name="groups_id"/>
    </record>

    <record model="res.users" id="base.user_admin">
        <field eval="[(4,ref('base.group_partner_manager'))]" name="groups_id"/>
    </record>

<data noupdate="1">
    <!-- Multi - Company Rules -->

    <record model="ir.rule" id="sale_order_comp_rule">
        <field name="name">Sales Order multi-company</field>
        <field name="model_id" ref="model_sale_order"/>
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>

    <record model="ir.rule" id="sale_order_line_comp_rule">
        <field name="name">Sales Order Line multi-company</field>
        <field name="model_id" ref="model_sale_order_line"/>
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>

    <record model="ir.rule" id="sale_order_report_comp_rule">
        <field name="name">Sales Order Analysis multi-company</field>
        <field name="model_id" ref="model_sale_report"/>
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>

    <!-- Portal Access Rules -->
    <record id="sale_order_rule_portal" model="ir.rule">
        <field name="name">Portal Personal Quotations/Sales Orders</field>
        <field name="model_id" ref="sale.model_sale_order"/>
        <field name="domain_force">[('message_partner_ids','child_of',[user.commercial_partner_id.id])]</field>
        <field name="groups" eval="[(4, ref('base.group_portal'))]"/>
        <field name="perm_unlink" eval="True"/>
        <field name="perm_write" eval="True"/>
        <field name="perm_read" eval="True"/>
        <field name="perm_create" eval="False"/>
    </record>

    <record id="sale_order_line_rule_portal" model="ir.rule">
        <field name="name">Portal Sales Orders Line</field>
        <field name="model_id" ref="sale.model_sale_order_line"/>
        <field name="domain_force">[('order_id.message_partner_ids','child_of',[user.commercial_partner_id.id])]</field>
        <field name="groups" eval="[(4, ref('base.group_portal'))]"/>
    </record>

    <!-- Multi - Salesmen sales order assignation rules -->

    <record id="sale_order_personal_rule" model="ir.rule">
        <field name="name">Personal Orders</field>
        <field ref="model_sale_order" name="model_id"/>
        <field name="domain_force">['|',('user_id','=',user.id),('user_id','=',False)]</field>
        <field name="groups" eval="[(4, ref('sales_team.group_sale_salesman'))]"/>
    </record>
    <record id="sale_order_see_all" model="ir.rule">
        <field name="name">All Orders</field>
        <field ref="model_sale_order" name="model_id"/>
        <field name="domain_force">[(1,'=',1)]</field>
        <field name="groups" eval="[(4, ref('sales_team.group_sale_salesman_all_leads'))]"/>
    </record>

    <record id="sale_order_report_personal_rule" model="ir.rule">
        <field name="name">Personal Orders Analysis</field>
        <field ref="model_sale_report" name="model_id"/>
        <field name="domain_force">['|',('user_id','=',user.id),('user_id','=',False)]</field>
        <field name="groups" eval="[(4, ref('sales_team.group_sale_salesman'))]"/>
    </record>

    <record id="sale_order_report_see_all" model="ir.rule">
        <field name="name">All Orders Analysis</field>
        <field ref="model_sale_report" name="model_id"/>
        <field name="domain_force">[(1,'=',1)]</field>
        <field name="groups" eval="[(4, ref('sales_team.group_sale_salesman_all_leads'))]"/>
    </record>

    <record id="sale_order_line_personal_rule" model="ir.rule">
        <field name="name">Personal Order Lines</field>
        <field ref="model_sale_order_line" name="model_id"/>
        <field name="domain_force">['|',('salesman_id','=',user.id),('salesman_id','=',False)]</field>
        <field name="groups" eval="[(4, ref('sales_team.group_sale_salesman'))]"/>
    </record>

    <record id="sale_order_line_see_all" model="ir.rule">
        <field name="name">All Orders Lines</field>
        <field ref="model_sale_order_line" name="model_id"/>
        <field name="domain_force">[(1,'=',1)]</field>
        <field name="groups" eval="[(4, ref('sales_team.group_sale_salesman_all_leads'))]"/>
    </record>

    <record id="account_invoice_report_rule_see_personal" model="ir.rule">
        <field name="name">Personal Invoices Analysis</field>
        <field name="model_id" ref="model_account_invoice_report"/>
        <field name="domain_force">['|', ('invoice_user_id', '=', user.id), ('invoice_user_id', '=', False)]</field>
        <field name="groups" eval="[(4, ref('sales_team.group_sale_salesman'))]"/>
    </record>

    <record id="account_invoice_report_rule_see_all" model="ir.rule">
        <field name="name">All Invoices Analysis</field>
        <field name="model_id" ref="model_account_invoice_report"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4, ref('sales_team.group_sale_salesman_all_leads'))]"/>
    </record>

    <!-- Payment transactions and tokens access rules -->

    <record id="payment_transaction_salesman_rule" model="ir.rule">
        <field name="name">Access every payment transaction</field>
        <field name="model_id" ref="payment.model_payment_transaction"/>
        <!-- Reset the domain defined by payment.transaction_user_rule -->
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4, ref('sales_team.group_sale_salesman'))]"/>
    </record>

    <record id="payment_token_salesman_rule" model="ir.rule">
        <field name="name">Access every payment token</field>
        <field name="model_id" ref="payment.model_payment_token"/>
        <!-- Reset the domain defined by payment.token_user_rule -->
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4, ref('sales_team.group_sale_salesman'))]"/>
    </record>

    <!-- Multi - Salesmen invoice and account move assignation rules -->
    <record id="account_invoice_rule_see_personal" model="ir.rule">
        <field name="name">Personal Invoices</field>
        <field name="model_id" ref="model_account_move"/>
        <field name="domain_force">[('move_type', 'in', ('out_invoice', 'out_refund')), '|', ('invoice_user_id', '=', user.id), ('invoice_user_id', '=', False)]</field>
        <field name="groups" eval="[(4, ref('sales_team.group_sale_salesman'))]"/>
    </record>

    <record id="account_invoice_rule_see_all" model="ir.rule">
        <field name="name">All Invoices</field>
        <field name="model_id" ref="model_account_move"/>
        <field name="domain_force">[('move_type', 'in', ('out_invoice', 'out_refund'))]</field>
        <field name="groups" eval="[(4, ref('sales_team.group_sale_salesman_all_leads'))]"/>
    </record>

    <record id="account_invoice_line_rule_see_personal" model="ir.rule">
        <field name="name">Personal Invoice Lines</field>
        <field name="model_id" ref="model_account_move_line"/>
        <field name="domain_force">[('move_id.move_type', 'in', ('out_invoice', 'out_refund')), '|', ('move_id.invoice_user_id', '=', user.id), ('move_id.invoice_user_id', '=', False)]</field>
        <field name="groups" eval="[(4, ref('sales_team.group_sale_salesman'))]"/>
    </record>

    <record id="account_invoice_line_rule_see_all" model="ir.rule">
        <field name="name">All Invoice Lines</field>
        <field name="model_id" ref="model_account_move_line"/>
        <field name="domain_force">[('move_id.move_type', 'in', ('out_invoice', 'out_refund'))]</field>
        <field name="groups" eval="[(4, ref('sales_team.group_sale_salesman_all_leads'))]"/>
    </record>

    <record id="account_invoice_send_rule_see_personal" model="ir.rule">
        <field name="name">Personal Invoice Send and Print</field>
        <field name="model_id" ref="account.model_account_invoice_send"/>
        <field name="domain_force">[('invoice_ids.move_type', 'in', ('out_invoice', 'out_refund')), '|', ('invoice_ids.invoice_user_id', '=', user.id), ('invoice_ids.invoice_user_id', '=', False)]</field>
        <field name="groups" eval="[(4, ref('sales_team.group_sale_salesman'))]"/>
    </record>

    <record id="account_invoice_send_rule_see_all" model="ir.rule">
        <field name="name">All Invoice Send and Print</field>
        <field name="model_id" ref="account.model_account_invoice_send"/>
        <field name="domain_force">[('invoice_ids.move_type', 'in', ('out_invoice', 'out_refund'))]</field>
        <field name="groups" eval="[(4, ref('sales_team.group_sale_salesman_all_leads'))]"/>
    </record>

    <!-- Wizard access rules -->
    <record id="sale_payment_acquirer_onboarding_wizard_rule" model="ir.rule">
        <field name="name">Payment Acquier Onboarding Wizard Rule</field>
        <field name="model_id" ref="model_sale_payment_acquirer_onboarding_wizard"/>
        <field name="domain_force">[('create_uid', '=', user.id)]</field>
    </record>

    <record id="sale_advance_payment_inv_rule" model="ir.rule">
        <field name="name">Sales Advance Payment Invoice Rule</field>
        <field name="model_id" ref="model_sale_advance_payment_inv"/>
        <field name="domain_force">[('create_uid', '=', user.id)]</field>
    </record>

    <record id="sale_order_cancel_rule" model="ir.rule">
        <field name="name">Sales Order Cancel Rule</field>
        <field name="model_id" ref="model_sale_order_cancel"/>
        <field name="domain_force">[('create_uid', '=', user.id)]</field>
    </record>
</data>
</odoo>

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#DA956B"/><stop offset="100%" stop-color="#CC7039"/></linearGradient><path id="d" d="M56.243 52.279c.578 0 1.05.537 1.05 1.193v.978c0 .656-.472 1.194-1.05 1.194H13.532c-.578 0-1.05-.538-1.05-1.194v-35.8c0-.657.472-1.194 1.05-1.194h1.5c.578 0 1.05.537 1.05 1.194v33.629h40.161zM39 23.025l4.981 4.963-6.302 7.25-4.866-5.53c-.411-.467-1.068-.467-1.48 0L20.92 41.423a1.31 1.31 0 0 0-.018 1.68l2.494 2.924c.412.477 1.086.487 1.497.01l7.186-8.165 4.857 5.52a.965.965 0 0 0 1.488 0l9.558-10.86L53 37.664 55 20l-16 3.025z"/><path id="e" d="M56.243 50.279c.578 0 1.05.537 1.05 1.193v.978c0 .656-.472 1.194-1.05 1.194H13.532c-.578 0-1.05-.538-1.05-1.194v-35.8c0-.657.472-1.194 1.05-1.194h1.5c.578 0 1.05.537 1.05 1.194v33.629h40.161zM39 21.025l4.981 4.963-6.302 7.25-4.866-5.53c-.411-.467-1.068-.467-1.48 0L20.92 39.423a1.31 1.31 0 0 0-.018 1.68l2.494 2.924c.412.477 1.086.487 1.497.01l7.186-8.165 4.857 5.52a.965.965 0 0 0 1.488 0l9.558-10.86L53 35.664 55 18l-16 3.025z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M45.243 69H4c-2 0-4-.146-4-4.077V35.315L13 16h3v26.5l15-14.27.974.024L40 21.096l13 14.27-18 18.346h22L45.243 69z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: static\src\js\payment_form.js

```javascript
odoo.define('sale.payment_form', require => {
    'use strict';

    const checkoutForm = require('payment.checkout_form');
    const manageForm = require('payment.manage_form');

    const salePaymentMixin = {

        //--------------------------------------------------------------------------
        // Private
        //--------------------------------------------------------------------------

        /**
         * Add `sale_order_id` to the transaction route params if it is provided.
         *
         * @override method from payment.payment_form_mixin
         * @private
         * @param {string} provider - The provider of the selected payment option's acquirer
         * @param {number} paymentOptionId - The id of the selected payment option
         * @param {string} flow - The online payment flow of the selected payment option
         * @return {object} The extended transaction route params
         */
        _prepareTransactionRouteParams: function (provider, paymentOptionId, flow) {
            const transactionRouteParams = this._super(...arguments);
            return {
                ...transactionRouteParams,
                'sale_order_id': this.txContext.saleOrderId
                    ? parseInt(this.txContext.saleOrderId) : undefined,
            };
        },

    };

    checkoutForm.include(salePaymentMixin);
    manageForm.include(salePaymentMixin);

});

```

## File: static\src\js\product_configurator_widget.js

```javascript
odoo.define('sale.product_configurator', function (require) {
var relationalFields = require('web.relational_fields');
var FieldsRegistry = require('web.field_registry');
var core = require('web.core');
var _t = core._t;

/**
 * The sale.product_configurator widget is a simple widget extending FieldMany2One
 * It allows the development of configuration strategies in other modules through
 * widget extensions.
 *
 *
 * !!! WARNING !!!
 *
 * This widget is only designed for sale_order_line creation/updates.
 * !!! It should only be used on a product_product or product_template field !!!
 */
var ProductConfiguratorWidget = relationalFields.FieldMany2One.extend({
    events: _.extend({}, relationalFields.FieldMany2One.prototype.events, {
        'click .o_edit_product_configuration': '_onEditConfiguration'
    }),

     /**
      * @override
      */
    _render: function () {
        this._super.apply(this, arguments);
        if (this.mode === 'edit' && this.value &&
        (this._isConfigurableProduct() || this._isConfigurableLine())) {
            this._addProductLinkButton();
            this._addConfigurationEditButton();
        } else if (this.mode === 'edit' && this.value) {
            this._addProductLinkButton();
            this.$('.o_edit_product_configuration').hide();
        } else {
            this.$('.o_external_button').hide();
            this.$('.o_edit_product_configuration').hide();
        }
    },

    /**
     * Add button linking to product_id/product_template_id form.
     */
    _addProductLinkButton: function () {
        if (this.$('.o_external_button').length === 0) {
            var $productLinkButton = $('<button>', {
                type: 'button',
                class: 'fa fa-external-link btn btn-secondary o_external_button',
                tabindex: '-1',
                draggable: false,
                'aria-label': _t('External Link'),
                title: _t('External Link')
            });

            var $inputDropdown = this.$('.o_input_dropdown');
            $inputDropdown.after($productLinkButton);
        }
    },

    /**
     * If current product is configurable,
     * Show edit button (in Edit Mode) after the product/product_template
     */
    _addConfigurationEditButton: function () {
        var $inputDropdown = this.$('.o_input_dropdown');

        if ($inputDropdown.length !== 0 &&
            this.$('.o_edit_product_configuration').length === 0) {
            var $editConfigurationButton = $('<button>', {
                type: 'button',
                class: 'fa fa-pencil btn btn-secondary o_edit_product_configuration',
                tabindex: '-1',
                draggable: false,
                'aria-label': _t('Edit Configuration'),
                title: _t('Edit Configuration')
            });

            $inputDropdown.after($editConfigurationButton);
        }
    },

    /**
     * Hook to override with _onEditProductConfiguration
     * to know if edit pencil button has to be put next to the field
     *
     * @private
     */
    _isConfigurableProduct: function () {
        return false;
    },

    /**
     * Hook to override with _onEditProductConfiguration
     * to know if edit pencil button has to be put next to the field
     *
     * @private
     */
    _isConfigurableLine: function () {
        return false;
    },

    /**
     * Override catching changes on product_id or product_template_id.
     * Calls _onTemplateChange in case of product_template change.
     * Calls _onProductChange in case of product change.
     * Shouldn't be overridden by product configurators
     * or only to setup some data for further computation
     * before calling super.
     *
     * @override
     * @param {OdooEvent} ev
     * @param {boolean} ev.data.preventProductIdCheck prevent the product configurator widget
     *     from looping forever when it needs to change the 'product_template_id'
     *
     * @private
     */
    reset: async function (record, ev) {
        await this._super(...arguments);
        if (ev && ev.target === this) {
            if (ev.data.changes && !ev.data.preventProductIdCheck && ev.data.changes.product_template_id) {
                this._onTemplateChange(record.data.product_template_id.data.id, ev.data.dataPointID);
            } else if (ev.data.changes && ev.data.changes.product_id) {
                this._onProductChange(record.data.product_id.data && record.data.product_id.data.id, ev.data.dataPointID).then(wizardOpened => {
                    if (!wizardOpened) {
                        this._onLineConfigured();
                    }
                });
            }
        }
    },

    /**
     * Hook for product_template based configurators
     * (product configurator, matrix, ...).
     *
     * @param {integer} productTemplateId
     * @param {String} dataPointID
     *
     * @private
     */
    _onTemplateChange: function (productTemplateId, dataPointId) {
        return Promise.resolve(false);
    },

    /**
     * Hook for product_product based configurators
     * (event, rental, ...).
     * Should return
     *    true if product has been configured through wizard or
     *        the result of the super call for other wizard extensions
     *    false if the product wasn't configurable through the wizard
     *
     * @param {integer} productId
     * @param {String} dataPointID
     * @returns {Promise<Boolean>} stopPropagation true if a suitable configurator has been found.
     *
     * @private
     */
    _onProductChange: function (productId, dataPointId) {
        return Promise.resolve(false);
    },

    /**
     * Hook for configurator happening after line has been set
     * (options, ...).
     * Allows sale_product_configurator module to apply its options
     * after line configuration has been done.
     *
     * @private
     */
    _onLineConfigured: function () {

    },

    /**
     * Triggered on click of the configuration button.
     * It is only shown in Edit mode,
     * when _isConfigurableProduct or _isConfigurableLine is True.
     *
     * After reflexion, when a line was configured through two wizards,
     * only the line configuration will open.
     *
     * Two hooks are available depending on configurator category:
     * _onEditLineConfiguration : line configurators
     * _onEditProductConfiguration : product configurators
     *
     * @private
     */
    _onEditConfiguration: function () {
        if (this._isConfigurableLine()) {
            this._onEditLineConfiguration();
        } else if (this._isConfigurableProduct()) {
            this._onEditProductConfiguration();
        }
    },

    /**
     * Hook for line configurators (rental, event)
     * on line edition (pencil icon inside product field)
     */
    _onEditLineConfiguration: function () {

    },

    /**
     * Hook for product configurators (matrix, product)
     * on line edition (pencil icon inside product field)
     */
    _onEditProductConfiguration: function () {

    },

    /**
     * Utilities for recordData conversion
     */

    /**
     * Will convert the values contained in the recordData parameter to
     * a list of '4' operations that can be passed as a 'default_' parameter.
     *
     * @param {Object} recordData
     *
     * @private
     */
    _convertFromMany2Many: function (recordData) {
        if (recordData) {
            var convertedValues = [];
            _.each(recordData.res_ids, function (resId) {
                convertedValues.push([4, parseInt(resId)]);
            });

            return convertedValues;
        }

        return null;
    },

    /**
     * Will convert the values contained in the recordData parameter to
     * a list of '0' or '4' operations (based on wether the record is already persisted or not)
     * that can be passed as a 'default_' parameter.
     *
     * @param {Object} recordData
     *
     * @private
     */
    _convertFromOne2Many: function (recordData) {
        if (recordData) {
            var convertedValues = [];
            _.each(recordData.res_ids, function (resId) {
                if (isNaN(resId)) {
                    _.each(recordData.data, function (record) {
                        if (record.ref === resId) {
                            convertedValues.push([0, 0, {
                                custom_product_template_attribute_value_id: record.data.custom_product_template_attribute_value_id.data.id,
                                custom_value: record.data.custom_value
                            }]);
                        }
                    });
                } else {
                    convertedValues.push([4, resId]);
                }
            });

            return convertedValues;
        }

        return null;
    }
});

FieldsRegistry.add('product_configurator', ProductConfiguratorWidget);

return ProductConfiguratorWidget;

});

```

## File: static\src\js\product_discount_widget.js

```javascript
odoo.define('sale.product_discount', function (require) {
    "use strict";

    const BasicFields = require('web.basic_fields');
    const FieldsRegistry = require('web.field_registry');

    /**
     * The sale.product_discount widget is a simple widget extending FieldFloat
     *
     *
     * !!! WARNING !!!
     *
     * This widget is only designed for sale_order_line creation/updates.
     * !!! It should only be used on a discount field !!!
     */
    const ProductDiscountWidget = BasicFields.FieldFloat.extend({

        /**
         * Override changes at a discount.
         *
         * @override
         * @param {OdooEvent} ev
         *
         */
        async reset(record, ev) {
            if (ev && ev.data.changes && ev.data.changes.discount >= 0) {
               this.trigger_up('open_discount_wizard');
            }
            this._super(...arguments);
        },
    });

    FieldsRegistry.add('product_discount', ProductDiscountWidget);

    return ProductDiscountWidget;

});

```

## File: static\src\js\sale.js

```javascript
odoo.define('sale.sales_team_dashboard', function (require) {
"use strict";

var core = require('web.core');
var KanbanRecord = require('web.KanbanRecord');
var session = require('web.session');
var _t = core._t;

KanbanRecord.include({
    events: _.defaults({
        'click .sales_team_target_definition': '_onSalesTeamTargetClick',
    }, KanbanRecord.prototype.events),

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @param {MouseEvent} ev
     */
    _onSalesTeamTargetClick: function (ev) {
        ev.preventDefault();
        var self = this;

        this.$target_input = $('<input>');
        this.$('.o_kanban_primary_bottom:last').html(this.$target_input);
        this.$target_input.focus();
        let preLabel = _t("Set an invoicing target: ");
        let postLabel = _t(" / Month");
        const kanbanBottomBlock = this.$el.find('.o_kanban_primary_bottom.bottom_block:last-child');
        if (this.recordData.currency_id) {
            const currency = session.get_currency(this.recordData.currency_id.res_id);
            if (currency.position === "after") {
                postLabel = ` ${' ' + currency.symbol}${postLabel}`;
            } else {
               preLabel = `${preLabel} ${currency.symbol + ' '}`;
            }
        }
        kanbanBottomBlock.prepend($('<span/>').text(preLabel));
        kanbanBottomBlock.append($('<span/>').text(postLabel));

        this.$target_input.on({
            blur: this._onSalesTeamTargetSet.bind(this),
            keydown: function (ev) {
                if (ev.keyCode === $.ui.keyCode.ENTER) {
                    self._onSalesTeamTargetSet();
                }
            },
        });
    },
    /**
     * Mostly a handler for what happens to the input "this.$target_input"
     *
     * @private
     *
     */
    _onSalesTeamTargetSet: function () {
        var self = this;
        var value = Number(this.$target_input.val());
        if (isNaN(value)) {
            this.displayNotification({ message: _t("Please enter an integer value"), type: 'danger' });
        } else {
            this.trigger_up('kanban_record_update', {
                invoiced_target: value,
                onSuccess: function () {
                    self.trigger_up('reload');
                },
            });
        }
    },
});

});

```

## File: static\src\js\sale_order_view.js

```javascript
odoo.define('sale.SaleOrderView', function (require) {
    "use strict";

    const FormController = require('web.FormController');
    const FormView = require('web.FormView');
    const viewRegistry = require('web.view_registry');
    const Dialog = require('web.Dialog');
    const core = require('web.core');
    const _t = core._t;

    const SaleOrderFormController = FormController.extend({
        custom_events: _.extend({}, FormController.prototype.custom_events, {
            open_discount_wizard: '_onOpenDiscountWizard',
        }),

        // -------------------------------------------------------------------------
        // Handlers
        // -------------------------------------------------------------------------

        /**
         * Handler called if user changes the discount field in the sale order line.
         * The wizard will open only if
         *  (1) Sale order line is 3 or more
         *  (2) First sale order line is changed to discount
         *  (3) Discount is the same in all sale order line
         */
        _onOpenDiscountWizard(ev) {
            const orderLines = this.renderer.state.data.order_line.data.filter(line => !line.data.display_type);
            const recordData = ev.target.recordData;
            if (recordData.discount === orderLines[0].data.discount) return;
            const isEqualDiscount = orderLines.slice(1).every(line => line.data.discount === recordData.discount);
            if (orderLines.length >= 3 && recordData.sequence === orderLines[0].data.sequence && isEqualDiscount) {
                Dialog.confirm(this, _t("Do you want to apply this discount to all order lines?"), {
                    confirm_callback: () => {
                        orderLines.slice(1).forEach((line) => {
                            this.trigger_up('field_changed', {
                                dataPointID: this.renderer.state.id,
                                changes: {order_line: {operation: "UPDATE", id: line.id, data: {discount: orderLines[0].data.discount}}},
                            });
                        });
                    },
                });
            }
        },
    });

    const SaleOrderView = FormView.extend({
        config: _.extend({}, FormView.prototype.config, {
            Controller: SaleOrderFormController,
        }),
    });

    viewRegistry.add('sale_discount_form', SaleOrderView);

    return SaleOrderView;

});

```

## File: static\src\js\sale_portal_sidebar.js

```javascript
odoo.define('sale.SalePortalSidebar', function (require) {
'use strict';

var publicWidget = require('web.public.widget');
var PortalSidebar = require('portal.PortalSidebar');

publicWidget.registry.SalePortalSidebar = PortalSidebar.extend({
    selector: '.o_portal_sale_sidebar',

    /**
     * @constructor
     */
    init: function (parent, options) {
        this._super.apply(this, arguments);
        this.authorizedTextTag = ['em', 'b', 'i', 'u'];
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
        // After singature, automatically open the popup for payment
        if ($.bbq.getState('allow_payment') === 'yes' && this.$('#o_sale_portal_paynow').length) {
            this.$('#o_sale_portal_paynow').trigger('click');
            $.bbq.removeState('allow_payment');
        }
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
        var id = _.uniqueId(prefix);
        this.spyWatched.find($el).attr('id', id);
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
            $bsSidenav = this.$el.find('.bs-sidenav');

        $("#quote_content [id^=quote_header_], #quote_content [id^=quote_]", this.spyWatched).attr("id", "");
        _.each(this.spyWatched.find("#quote_content h2, #quote_content h3"), function (el) {
            var id, text;
            switch (el.tagName.toLowerCase()) {
                case "h2":
                    id = self._setElementId('quote_header_', el);
                    text = self._extractText($(el));
                    if (!text) {
                        break;
                    }
                    lastLI = $("<li class='nav-item'>").append($('<a class="nav-link" style="max-width: 200px;" href="#' + id + '"/>').text(text)).appendTo($bsSidenav);
                    lastUL = false;
                    break;
                case "h3":
                    id = self._setElementId('quote_', el);
                    text = self._extractText($(el));
                    if (!text) {
                        break;
                    }
                    if (lastLI) {
                        if (!lastUL) {
                            lastUL = $("<ul class='nav flex-column'>").appendTo(lastLI);
                        }
                        $("<li class='nav-item'>").append($('<a class="nav-link" style="max-width: 200px;" href="#' + id + '"/>').text(text)).appendTo(lastUL);
                    }
                    break;
            }
            el.setAttribute('data-anchor', true);
        });
        this.trigger_up('widgets_start_request', {$target: $bsSidenav});
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
        _.each($node.contents(), function (el) {
            var current = $(el);
            if ($.trim(current.text())) {
                var tagName = current.prop("tagName");
                if (_.isUndefined(tagName) || (!_.isUndefined(tagName) && _.contains(self.authorizedTextTag, tagName.toLowerCase()))) {
                    rawText.push($.trim(current.text()));
                }
            }
        });
        return rawText.join(' ');
    },
});
});

```

## File: static\src\js\variant_mixin.js

```javascript
odoo.define('sale.VariantMixin', function (require) {
'use strict';

var concurrency = require('web.concurrency');
var core = require('web.core');
var utils = require('web.utils');
var ajax = require('web.ajax');
var session = require('web.session');
var _t = core._t;

var VariantMixin = {
    events: {
        'change .css_attribute_color input': '_onChangeColorAttribute',
        'change .main_product:not(.in_cart) input.js_quantity': 'onChangeAddQuantity',
        'change [data-attribute_exclusions]': 'onChangeVariant'
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * When a variant is changed, this will check:
     * - If the selected combination is available or not
     * - The extra price if applicable
     * - The display name of the product ("Customizable desk (White, Steel)")
     * - The new total price
     * - The need of adding a "custom value" input
     *   If the custom value is the only available value
     *   (defined by its data 'is_single_and_custom'),
     *   the custom value will have it's own input & label
     *
     * 'change' events triggered by the user entered custom values are ignored since they
     * are not relevant
     *
     * @param {MouseEvent} ev
     */
    onChangeVariant: function (ev) {
        var $parent = $(ev.target).closest('.js_product');
        if (!$parent.data('uniqueId')) {
            $parent.data('uniqueId', _.uniqueId());
        }
        this._throttledGetCombinationInfo($parent.data('uniqueId'))(ev);
    },
    /**
     * @see onChangeVariant
     *
     * @private
     * @param {Event} ev
     * @returns {Deferred}
     */
    _getCombinationInfo: function (ev) {
        if ($(ev.target).hasClass('variant_custom_value')) {
            return Promise.resolve();
        }

        const $parent = $(ev.target).closest('.js_product');
        const combination = this.getSelectedVariantValues($parent);
        let parentCombination;

        if ($parent.hasClass('main_product')) {
            parentCombination = $parent.find('ul[data-attribute_exclusions]').data('attribute_exclusions').parent_combination;
            const $optProducts = $parent.parent().find(`[data-parent-unique-id='${$parent.data('uniqueId')}']`);

            for (const optionalProduct of $optProducts) {
                const $currentOptionalProduct = $(optionalProduct);
                const childCombination = this.getSelectedVariantValues($currentOptionalProduct);
                const productTemplateId = parseInt($currentOptionalProduct.find('.product_template_id').val());
                ajax.jsonRpc(this._getUri('/sale/get_combination_info'), 'call', {
                    'product_template_id': productTemplateId,
                    'product_id': this._getProductId($currentOptionalProduct),
                    'combination': childCombination,
                    'add_qty': parseInt($currentOptionalProduct.find('input[name="add_qty"]').val()),
                    'pricelist_id': this.pricelistId || false,
                    'parent_combination': combination,
                    'context': session.user_context,
                }).then((combinationData) => {
                    if (this._shouldIgnoreRpcResult()) {
                        return;
                    }
                    this._onChangeCombination(ev, $currentOptionalProduct, combinationData);
                    this._checkExclusions($currentOptionalProduct, childCombination, combinationData.parent_exclusions);
                });
            }
        } else {
            parentCombination = this.getSelectedVariantValues(
                $parent.parent().find('.js_product.in_cart.main_product')
            );
        }

        return ajax.jsonRpc(this._getUri('/sale/get_combination_info'), 'call', {
            'product_template_id': parseInt($parent.find('.product_template_id').val()),
            'product_id': this._getProductId($parent),
            'combination': combination,
            'add_qty': parseInt($parent.find('input[name="add_qty"]').val()),
            'pricelist_id': this.pricelistId || false,
            'parent_combination': parentCombination,
            'context': session.user_context,
        }).then((combinationData) => {
            if (this._shouldIgnoreRpcResult()) {
                return;
            }
            this._onChangeCombination(ev, $parent, combinationData);
            this._checkExclusions($parent, combination, combinationData.parent_exclusions);
        });
    },

    /**
     * Will add the "custom value" input for this attribute value if
     * the attribute value is configured as "custom" (see product_attribute_value.is_custom)
     *
     * @private
     * @param {MouseEvent} ev
     */
    handleCustomValues: function ($target) {
        var $variantContainer;
        var $customInput = false;
        if ($target.is('input[type=radio]') && $target.is(':checked')) {
            $variantContainer = $target.closest('ul').closest('li');
            $customInput = $target;
        } else if ($target.is('select')) {
            $variantContainer = $target.closest('li');
            $customInput = $target
                .find('option[value="' + $target.val() + '"]');
        }

        if ($variantContainer) {
            if ($customInput && $customInput.data('is_custom') === 'True') {
                var attributeValueId = $customInput.data('value_id');
                var attributeValueName = $customInput.data('value_name');

                if ($variantContainer.find('.variant_custom_value').length === 0
                        || $variantContainer
                              .find('.variant_custom_value')
                              .data('custom_product_template_attribute_value_id') !== parseInt(attributeValueId)) {
                    $variantContainer.find('.variant_custom_value').remove();

                    const previousCustomValue = $customInput.attr("previous_custom_value");
                    var $input = $('<input>', {
                        type: 'text',
                        'data-custom_product_template_attribute_value_id': attributeValueId,
                        'data-attribute_value_name': attributeValueName,
                        class: 'variant_custom_value form-control mt-2'
                    });

                    $input.attr('placeholder', attributeValueName);
                    $input.addClass('custom_value_radio');
                    $variantContainer.append($input);
                    if (previousCustomValue) {
                        $input.val(previousCustomValue);
                    }
                }
            } else {
                $variantContainer.find('.variant_custom_value').remove();
            }
        }
    },

    /**
     * Hack to add and remove from cart with json
     *
     * @param {MouseEvent} ev
     */
    onClickAddCartJSON: function (ev) {
        ev.preventDefault();
        var $link = $(ev.currentTarget);
        var $input = $link.closest('.input-group').find("input");
        var min = parseFloat($input.data("min") || 0);
        var max = parseFloat($input.data("max") || Infinity);
        var previousQty = parseFloat($input.val() || 0, 10);
        var quantity = ($link.has(".fa-minus").length ? -1 : 1) + previousQty;
        var newQty = quantity > min ? (quantity < max ? quantity : max) : min;

        if (newQty !== previousQty) {
            $input.val(newQty).trigger('change');
        }
        return false;
    },

    /**
     * When the quantity is changed, we need to query the new price of the product.
     * Based on the price list, the price might change when quantity exceeds X
     *
     * @param {MouseEvent} ev
     */
    onChangeAddQuantity: function (ev) {
        var $parent;

        if ($(ev.currentTarget).closest('.oe_advanced_configurator_modal').length > 0){
            $parent = $(ev.currentTarget).closest('.oe_advanced_configurator_modal');
        } else if ($(ev.currentTarget).closest('form').length > 0){
            $parent = $(ev.currentTarget).closest('form');
        }  else {
            $parent = $(ev.currentTarget).closest('.o_product_configurator');
        }

        this.triggerVariantChange($parent);
    },

    /**
     * Triggers the price computation and other variant specific changes
     *
     * @param {$.Element} $container
     */
    triggerVariantChange: function ($container) {
        var self = this;
        $container.find('ul[data-attribute_exclusions]').trigger('change');
        $container.find('input.js_variant_change:checked, select.js_variant_change').each(function () {
            self.handleCustomValues($(this));
        });
    },

    /**
     * Will look for user custom attribute values
     * in the provided container
     *
     * @param {$.Element} $container
     * @returns {Array} array of custom values with the following format
     *   {integer} custom_product_template_attribute_value_id
     *   {string} attribute_value_name
     *   {string} custom_value
     */
    getCustomVariantValues: function ($container) {
        var variantCustomValues = [];
        $container.find('.variant_custom_value').each(function (){
            var $variantCustomValueInput = $(this);
            if ($variantCustomValueInput.length !== 0){
                variantCustomValues.push({
                    'custom_product_template_attribute_value_id': $variantCustomValueInput.data('custom_product_template_attribute_value_id'),
                    'attribute_value_name': $variantCustomValueInput.data('attribute_value_name'),
                    'custom_value': $variantCustomValueInput.val(),
                });
            }
        });

        return variantCustomValues;
    },

    /**
     * Will look for attribute values that do not create product variant
     * (see product_attribute.create_variant "dynamic")
     *
     * @param {$.Element} $container
     * @returns {Array} array of attribute values with the following format
     *   {integer} custom_product_template_attribute_value_id
     *   {string} attribute_value_name
     *   {integer} value
     *   {string} attribute_name
     *   {boolean} is_custom
     */
    getNoVariantAttributeValues: function ($container) {
        var noVariantAttributeValues = [];
        var variantsValuesSelectors = [
            'input.no_variant.js_variant_change:checked',
            'select.no_variant.js_variant_change'
        ];

        $container.find(variantsValuesSelectors.join(',')).each(function (){
            var $variantValueInput = $(this);
            var singleNoCustom = $variantValueInput.data('is_single') && !$variantValueInput.data('is_custom');

            if ($variantValueInput.is('select')){
                $variantValueInput = $variantValueInput.find('option[value=' + $variantValueInput.val() + ']');
            }

            if ($variantValueInput.length !== 0 && !singleNoCustom){
                noVariantAttributeValues.push({
                    'custom_product_template_attribute_value_id': $variantValueInput.data('value_id'),
                    'attribute_value_name': $variantValueInput.data('value_name'),
                    'value': $variantValueInput.val(),
                    'attribute_name': $variantValueInput.data('attribute_name'),
                    'is_custom': $variantValueInput.data('is_custom')
                });
            }
        });

        return noVariantAttributeValues;
    },

    /**
     * Will return the list of selected product.template.attribute.value ids
     * For the modal, the "main product"'s attribute values are stored in the
     * "unchanged_value_ids" data
     *
     * @param {$.Element} $container the container to look into
     */
    getSelectedVariantValues: function ($container) {
        var values = [];
        var unchangedValues = $container
            .find('div.oe_unchanged_value_ids')
            .data('unchanged_value_ids') || [];

        var variantsValuesSelectors = [
            'input.js_variant_change:checked',
            'select.js_variant_change'
        ];
        _.each($container.find(variantsValuesSelectors.join(', ')), function (el) {
            values.push(+$(el).val());
        });

        return values.concat(unchangedValues);
    },

    /**
     * Will return a promise:
     *
     * - If the product already exists, immediately resolves it with the product_id
     * - If the product does not exist yet ("dynamic" variant creation), this method will
     *   create the product first and then resolve the promise with the created product's id
     *
     * @param {$.Element} $container the container to look into
     * @param {integer} productId the product id
     * @param {integer} productTemplateId the corresponding product template id
     * @param {boolean} useAjax wether the rpc call should be done using ajax.jsonRpc or using _rpc
     * @returns {Promise} the promise that will be resolved with a {integer} productId
     */
    selectOrCreateProduct: function ($container, productId, productTemplateId, useAjax) {
        var self = this;
        productId = parseInt(productId);
        productTemplateId = parseInt(productTemplateId);
        var productReady = Promise.resolve();
        if (productId) {
            productReady = Promise.resolve(productId);
        } else {
            var params = {
                product_template_id: productTemplateId,
                product_template_attribute_value_ids:
                    JSON.stringify(self.getSelectedVariantValues($container)),
            };

            var route = '/sale/create_product_variant';
            if (useAjax) {
                productReady = ajax.jsonRpc(route, 'call', params);
            } else {
                productReady = this._rpc({route: route, params: params});
            }
        }

        return productReady;
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Will disable attribute value's inputs based on combination exclusions
     * and will disable the "add" button if the selected combination
     * is not available
     *
     * This will check both the exclusions within the product itself and
     * the exclusions coming from the parent product (meaning that this product
     * is an option of the parent product)
     *
     * It will also check that the selected combination does not exactly
     * match a manually archived product
     *
     * @private
     * @param {$.Element} $parent the parent container to apply exclusions
     * @param {Array} combination the selected combination of product attribute values
     * @param {Array} parentExclusions the exclusions induced by the variant selection of the parent product
     * For example chair cannot have steel legs if the parent Desk doesn't have steel legs
     */
    _checkExclusions: function ($parent, combination, parentExclusions) {
        var self = this;
        var combinationData = $parent
            .find('ul[data-attribute_exclusions]')
            .data('attribute_exclusions');

        if (parentExclusions && combinationData.parent_exclusions) {
            combinationData.parent_exclusions = parentExclusions;
        }
        $parent
            .find('option, input, label, .o_variant_pills')
            .removeClass('css_not_available')
            .attr('title', function () { return $(this).data('value_name') || ''; })
            .data('excluded-by', '');

        // exclusion rules: array of ptav
        // for each of them, contains array with the other ptav they exclude
        if (combinationData.exclusions) {
            // browse all the currently selected attributes
            _.each(combination, function (current_ptav) {
                if (combinationData.exclusions.hasOwnProperty(current_ptav)) {
                    // for each exclusion of the current attribute:
                    _.each(combinationData.exclusions[current_ptav], function (excluded_ptav) {
                        // disable the excluded input (even when not already selected)
                        // to give a visual feedback before click
                        self._disableInput(
                            $parent,
                            excluded_ptav,
                            current_ptav,
                            combinationData.mapped_attribute_names
                        );
                    });
                }
            });
        }

        // parent exclusions (tell which attributes are excluded from parent)
        _.each(combinationData.parent_exclusions, function (exclusions, excluded_by){
            // check that the selected combination is in the parent exclusions
            _.each(exclusions, function (ptav) {

                // disable the excluded input (even when not already selected)
                // to give a visual feedback before click
                self._disableInput(
                    $parent,
                    ptav,
                    excluded_by,
                    combinationData.mapped_attribute_names,
                    combinationData.parent_product_name
                );
            });
        });
    },
    /**
     * Extracted to a method to be extendable by other modules
     *
     * @param {$.Element} $parent
     */
    _getProductId: function ($parent) {
        return parseInt($parent.find('.product_id').val());
    },
    /**
     * Will disable the input/option that refers to the passed attributeValueId.
     * This is used for showing the user that some combinations are not available.
     *
     * It will also display a message explaining why the input is not selectable.
     * Based on the "excludedBy" and the "productName" params.
     * e.g: Not available with Color: Black
     *
     * @private
     * @param {$.Element} $parent
     * @param {integer} attributeValueId
     * @param {integer} excludedBy The attribute value that excludes this input
     * @param {Object} attributeNames A dict containing all the names of the attribute values
     *   to show a human readable message explaining why the input is disabled.
     * @param {string} [productName] The parent product. If provided, it will be appended before
     *   the name of the attribute value that excludes this input
     *   e.g: Not available with Customizable Desk (Color: Black)
     */
    _disableInput: function ($parent, attributeValueId, excludedBy, attributeNames, productName) {
        var $input = $parent
            .find('option[value=' + attributeValueId + '], input[value=' + attributeValueId + ']');
        $input.addClass('css_not_available');
        $input.closest('label').addClass('css_not_available');
        $input.closest('.o_variant_pills').addClass('css_not_available');

        if (excludedBy && attributeNames) {
            var $target = $input.is('option') ? $input : $input.closest('label').add($input);
            var excludedByData = [];
            if ($target.data('excluded-by')) {
                excludedByData = JSON.parse($target.data('excluded-by'));
            }

            var excludedByName = attributeNames[excludedBy];
            if (productName) {
                excludedByName = productName + ' (' + excludedByName + ')';
            }
            excludedByData.push(excludedByName);

            $target.attr('title', _.str.sprintf(_t('Not available with %s'), excludedByData.join(', ')));
            $target.data('excluded-by', JSON.stringify(excludedByData));
        }
    },
    /**
     * @see onChangeVariant
     *
     * @private
     * @param {MouseEvent} ev
     * @param {$.Element} $parent
     * @param {Array} combination
     */
    _onChangeCombination: function (ev, $parent, combination) {
        var self = this;
        var $price = $parent.find(".oe_price:first .oe_currency_value");
        var $default_price = $parent.find(".oe_default_price:first .oe_currency_value");
        var $optional_price = $parent.find(".oe_optional:first .oe_currency_value");
        $price.text(self._priceToStr(combination.price));
        $default_price.text(self._priceToStr(combination.list_price));

        var isCombinationPossible = true;
        if (!_.isUndefined(combination.is_combination_possible)) {
            isCombinationPossible = combination.is_combination_possible;
        }
        this._toggleDisable($parent, isCombinationPossible);

        if (combination.has_discounted_price) {
            $default_price
                .closest('.oe_website_sale')
                .addClass("discount");
            $optional_price
                .closest('.oe_optional')
                .removeClass('d-none')
                .css('text-decoration', 'line-through');
            $default_price.parent().removeClass('d-none');
        } else {
            $default_price
                .closest('.oe_website_sale')
                .removeClass("discount");
            $optional_price.closest('.oe_optional').addClass('d-none');
            $default_price.parent().addClass('d-none');
        }

        var rootComponentSelectors = [
            'tr.js_product',
            '.oe_website_sale',
            '.o_product_configurator'
        ];

        // update images only when changing product
        // or when either ids are 'false', meaning dynamic products.
        // Dynamic products don't have images BUT they may have invalid
        // combinations that need to disable the image.
        if (!combination.product_id ||
            !this.last_product_id ||
            combination.product_id !== this.last_product_id) {
            this.last_product_id = combination.product_id;
            self._updateProductImage(
                $parent.closest(rootComponentSelectors.join(', ')),
                combination.display_image,
                combination.product_id,
                combination.product_template_id,
                combination.carousel,
                isCombinationPossible
            );
        }

        $parent
            .find('.product_id')
            .first()
            .val(combination.product_id || 0)
            .trigger('change');

        $parent
            .find('.product_display_name')
            .first()
            .text(combination.display_name);

        $parent
            .find('.js_raw_price')
            .first()
            .text(combination.price)
            .trigger('change');

        this.handleCustomValues($(ev.target));
    },

    /**
     * returns the formatted price
     *
     * @private
     * @param {float} price
     */
    _priceToStr: function (price) {
        var l10n = _t.database.parameters;
        var precision = 2;

        if ($('.decimal_precision').length) {
            precision = parseInt($('.decimal_precision').last().data('precision'));
        }
        var formatted = _.str.sprintf('%.' + precision + 'f', price).split('.');
        formatted[0] = utils.insert_thousand_seps(formatted[0]);
        return formatted.join(l10n.decimal_point);
    },
    /**
     * Returns a throttled `_getCombinationInfo` with a leading and a trailing
     * call, which is memoized per `uniqueId`, and for which previous results
     * are dropped.
     *
     * The uniqueId is needed because on the configurator modal there might be
     * multiple elements triggering the rpc at the same time, and we need each
     * individual product rpc to be executed, but only once per individual
     * product.
     *
     * The leading execution is to keep good reactivity on the first call, for
     * a better user experience. The trailing is because ultimately only the
     * information about the last selected combination is useful. All
     * intermediary rpc can be ignored and are therefore best not done at all.
     *
     * The DropMisordered is to make sure slower rpc are ignored if the result
     * of a newer rpc has already been received.
     *
     * @private
     * @param {string} uniqueId
     * @returns {function}
     */
    _throttledGetCombinationInfo: _.memoize(function (uniqueId) {
        var dropMisordered = new concurrency.DropMisordered();
        var _getCombinationInfo = _.throttle(this._getCombinationInfo.bind(this), 500);
        return function (ev, params) {
            return dropMisordered.add(_getCombinationInfo(ev, params));
        };
    }),
    /**
     * Toggles the disabled class depending on the $parent element
     * and the possibility of the current combination.
     *
     * @private
     * @param {$.Element} $parent
     * @param {boolean} isCombinationPossible
     */
    _toggleDisable: function ($parent, isCombinationPossible) {
        $parent.toggleClass('css_not_available', !isCombinationPossible);
        if ($parent.hasClass('in_cart')) {
            const primaryButton = $parent.parents('.modal-content').find('.modal-footer .btn-primary');
            primaryButton.prop('disabled', !isCombinationPossible);
            primaryButton.toggleClass('disabled', !isCombinationPossible);
        }
    },
    /**
     * Updates the product image.
     * This will use the productId if available or will fallback to the productTemplateId.
     *
     * @private
     * @param {$.Element} $productContainer
     * @param {boolean} displayImage will hide the image if true. It will use the 'invisible' class
     *   instead of d-none to prevent layout change
     * @param {integer} product_id
     * @param {integer} productTemplateId
     */
    _updateProductImage: function ($productContainer, displayImage, productId, productTemplateId) {
        var model = productId ? 'product.product' : 'product.template';
        var modelId = productId || productTemplateId;
        var imageUrl = '/web/image/{0}/{1}/' + (this._productImageField ? this._productImageField : 'image_1024');
        var imageSrc = imageUrl
            .replace("{0}", model)
            .replace("{1}", modelId);

        var imagesSelectors = [
            'span[data-oe-model^="product."][data-oe-type="image"] img:first',
            'img.product_detail_img',
            'span.variant_image img',
            'img.variant_image',
        ];

        var $img = $productContainer.find(imagesSelectors.join(', '));

        if (displayImage) {
            $img.removeClass('invisible').attr('src', imageSrc);
        } else {
            $img.addClass('invisible');
        }
    },

    /**
     * Highlight selected color
     *
     * @private
     * @param {MouseEvent} ev
     */
    _onChangeColorAttribute: function (ev) {
        var $parent = $(ev.target).closest('.js_product');
        $parent.find('.css_attribute_color')
            .removeClass("active")
            .filter(':has(input:checked)')
            .addClass("active");
    },

    /**
     * Return true if the current object has been destroyed.
     * This function has been added as a fix to know if the result of a rpc
     * should be handled. Indeed, "this._rpc()" can not be used as it is not
     * supported by some elements that use this mixin.
     *
     * @private
     */
    _shouldIgnoreRpcResult() {
        return (typeof this.isDestroyed === "function" && this.isDestroyed());
    },

    /**
     * Extension point for website_sale
     *
     * @private
     * @param {string} uri The uri to adapt
     */
    _getUri: function (uri) {
        return uri;
    }
};

return VariantMixin;

});

```

## File: static\src\js\tours\sale.js

```javascript
odoo.define('sale.tour', function(require) {
"use strict";

const {_t} = require('web.core');
const {Markup} = require('web.utils');
var tour = require('web_tour.tour');

tour.register("sale_tour", {
    url: "/web",
    rainbowMan: false,
    sequence: 20,
}, [tour.stepUtils.showAppsMenuItem(), {
    trigger: ".o_app[data-menu-xmlid='sale.sale_menu_root']",
    content: _t("Open Sales app to send your first quotation in a few clicks."),
    position: "right",
    edition: "community"
}, {
    trigger: ".o_app[data-menu-xmlid='sale.sale_menu_root']",
    content: _t("Open Sales app to send your first quotation in a few clicks."),
    position: "bottom",
    edition: "enterprise"
}, {
    trigger: 'a.o_onboarding_step_action.btn[data-method=action_open_base_onboarding_company]',
    extra_trigger: ".o_sale_order",
    content: _t("Start by checking your company's data."),
    position: "bottom",
}, {
    trigger: ".modal-content button[name='action_save_onboarding_company_step']",
    content: _t("Looks good. Let's continue."),
    position: "left",
}, {
    trigger: 'a.o_onboarding_step_action.btn[data-method=action_open_base_document_layout]',
    extra_trigger: ".o_sale_order",
    content: _t("Customize your quotes and orders."),
    position: "bottom",
}, {
    trigger: "button[name='document_layout_save']",
    extra_trigger: ".o_sale_order",
    content: _t("Good job, let's continue."),
    position: "top", // dot NOT move to bottom, it would cause a resize flicker
}, {
    trigger: 'a.o_onboarding_step_action.btn[data-method=action_open_sale_onboarding_payment_acquirer]',
    extra_trigger: ".o_sale_order",
    content: _t("To speed up order confirmation, we can activate electronic signatures or payments."),
    position: "bottom",
}, {
    trigger: "button[name='add_payment_methods']",
    extra_trigger: ".o_sale_order",
    content: _t("Lets keep electronic signature for now."),
    position: "bottom",
}, {
    trigger: 'a.o_onboarding_step_action.btn[data-method=action_open_sale_onboarding_sample_quotation]',
    extra_trigger: ".o_sale_order",
    content: _t("Now, we'll create a sample quote."),
    position: "bottom",
}]);

tour.register("sale_quote_tour", {
        url: "/web#action=sale.action_quotations_with_onboarding&view_type=form",
        rainbowMan: true,
        rainbowManMessage: "<b>Congratulations</b>, your first quotation is sent!<br>Check your email to validate the quote.",
        sequence: 30,
    }, [{
        trigger: ".o_form_editable .o_field_many2one[name='partner_id']",
        extra_trigger: ".o_sale_order",
        content: _t("Write a company name to create one, or see suggestions."),
        position: "right",
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
        content: _t("Click here to add some products or services to your quotation."),
        position: "bottom",
    }, {
        trigger: ".o_field_widget[name='product_id'], .o_field_widget[name='product_template_id']",
        extra_trigger: ".o_sale_order",
        content: _t("Select a product, or create a new one on the fly."),
        position: "right",
        run: function (actions) {
            var $input = this.$anchor.find("input");
            actions.text("DESK0001", $input.length === 0 ? this.$anchor : $input);
            // fake keydown to trigger search
            var keyDownEvent = jQuery.Event("keydown");
            keyDownEvent.which = 42;
            this.$anchor.trigger(keyDownEvent);
            var $descriptionElement = $(".o_form_editable textarea[name='name']");
            // when description changes, we know the product has been created
            $descriptionElement.change(function () {
                $descriptionElement.addClass("product_creation_success");
            });
        },
        id: "product_selection_step"
    }, {
        trigger: ".ui-menu.ui-widget .ui-menu-item a:contains('DESK0001')",
        auto: true,
    }, {
        trigger: ".o_form_editable textarea[name='name'].product_creation_success",
        auto: true,
        run: function () {
        } // wait for product creation
    }, {
        trigger: ".o_field_widget[name='price_unit'] ",
        extra_trigger: ".o_sale_order",
        content: Markup(_t("<b>Set a price</b>.")),
        position: "right",
        run: "text 10.0"
    },
    ...tour.stepUtils.statusbarButtonsSteps("Send by Email", Markup(_t("<b>Send the quote</b> to yourself and check what the customer will receive.")), ".o_statusbar_buttons button[name='action_quotation_send']"),
    {
        trigger: ".modal-footer button.btn-primary",
        auto: true,
    }, {
        trigger: ".modal-footer button[name='action_send_mail']",
        extra_trigger: ".modal-footer button[name='action_send_mail']",
        content: _t("Let's send the quote."),
        position: "bottom",
    }]);

});

```

## File: views\crm_team_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="crm_team_salesteams_view_form" model="ir.ui.view">
            <field name="name">crm.team.form</field>
            <field name="model">crm.team</field>
            <field name="priority">9</field>
            <field name="inherit_id" ref="sales_team.crm_team_view_form"/>
            <field name="arch" type="xml">
                <xpath expr="//div[@name='options_active']" position="inside">
                    <div class="o_row" style="display:inherit">
                        <field name="use_quotations"/><label for="use_quotations"/>
                    </div>
                </xpath>
                <xpath expr="//field[@name='company_id']" position="after">
                    <label for="invoiced_target"/>
                    <div class="o_row">
                        <field name="invoiced_target" widget="monetary" options="{'currency_field': 'currency_id'}"/>
                        <span class="oe_read_only">/ Month</span>
                    </div>
                </xpath>
            </field>
        </record>

        <record id="view_account_invoice_report_search_inherit" model="ir.ui.view">
            <field name="name">account.invoice.report.search.inherit</field>
            <field name="model">account.invoice.report</field>
            <field name="inherit_id" ref="account.view_account_invoice_report_search"/>
            <field name="arch" type="xml">
                <xpath expr="//group/filter[@name='user']" position="after">
                    <filter string="Sales Team" name="sales_channel" domain="[]" context="{'group_by':'team_id'}"/>
                </xpath>
                <xpath expr="//field[@name='invoice_user_id']" position="after">
                    <field name="team_id"/>
                </xpath>
            </field>
        </record>

    <!-- Sales Team Dashboard Views -->
    <record id="action_quotation_form" model="ir.actions.act_window">
        <field name="name">New Quotation</field>
        <field name="res_model">sale.order</field>
        <field name="type">ir.actions.act_window</field>
        <field name="view_mode">form</field>
        <field name="context">{
                'search_default_team_id': [active_id],
                'default_team_id': active_id,
                'default_user_id': uid,
        }
        </field>
        <field name="search_view_id" ref="sale_order_view_search_inherit_quotation"/>
    </record>

    <record id="crm_team_view_kanban_dashboard" model="ir.ui.view">
        <field name="name">crm.team.view.kanban.dashboard.inherit.sale</field>
        <field name="model">crm.team</field>
        <field name="inherit_id" ref="sales_team.crm_team_view_kanban_dashboard"/>
        <field name="arch" type="xml">
        <data>
            <xpath expr="//templates" position="before">
                <field name="use_quotations"/>
                <field name="invoiced_target"/>
            </xpath>

            <xpath expr="//t[@name='second_options']" position="after">
                <div class="row" t-if="record.quotations_count.raw_value">
                    <div class="col overflow-hidden text-left">
                        <a name="%(action_quotations_salesteams)d" type="action" context="{'search_default_draft': True, 'search_default_sent': True}">
                            <field name="quotations_count"/>
                            <t t-if="record.quotations_count.raw_value == 1">Quotation</t>
                            <t t-else="">Quotations</t>
                        </a>
                    </div>
                    <div class="col-auto text-right">
                        <field name="quotations_amount" widget="monetary"/>
                    </div>
                </div>
                <div class="row" name="orders_to_invoice" t-if="record.sales_to_invoice_count.raw_value">
                    <div class="col-8">
                        <a name="%(action_orders_to_invoice_salesteams)d" type="action">
                            <field name="sales_to_invoice_count"/>
                            <t t-if="record.sales_to_invoice_count.raw_value == 1">Order to Invoice</t>
                            <t t-else="">Orders to Invoice</t>
                        </a>
                    </div>
                </div>
            </xpath>

            <xpath expr="//div[hasclass('o_kanban_primary_bottom')]" position="after">
                <t groups="sales_team.group_sale_manager">
                    <div class="col-12 o_kanban_primary_bottom bottom_block">
                        <t t-if="record.invoiced_target.raw_value" class="col-12 o_kanban_primary_bottom">
                            <field name="invoiced" widget="progressbar" title="Invoicing" options="{'current_value': 'invoiced', 'max_value': 'invoiced_target', 'editable': true, 'edit_max_value': true, 'on_change': 'update_invoiced_target'}"/>
                        </t>
                        <t t-if="!record.invoiced_target.raw_value" class="col-12 o_kanban_primary_bottom text-center">
                            <a href="#" class="sales_team_target_definition o_inline_link">Click to define an invoicing target</a>
                        </t>
                    </div>
                </t>
            </xpath>

            <xpath expr="//div[hasclass('o_kanban_manage_view')]" position="inside">
                <div t-if="record.use_quotations.raw_value">
                    <a name="%(action_quotations_salesteams)d" type="action" class="o_quotation_view_button">Quotations</a>
                </div>
                <div>
                    <a name="%(action_orders_salesteams)d" type="action">Sales Orders</a>
                </div>
                <div groups="account.group_account_invoice">
                    <a name="%(action_invoice_salesteams)d" type="action">Invoices</a>
                </div>
            </xpath>

            <xpath expr="//div[hasclass('o_kanban_manage_new')]" position="inside">
                <div t-if="record.use_quotations.raw_value">
                    <a name="%(action_quotation_form)d" type="action">
                        Quotation
                    </a>
                </div>
            </xpath>

            <xpath expr="//div[hasclass('o_kanban_manage_reports')]/div[@name='o_team_kanban_report_separator']" position="before">
                <div t-if="record.use_quotations.raw_value">
                     <a name="%(action_order_report_quotation_salesteam)d" type="action">
                         Quotations
                     </a>
                </div>
                <div name="sales_report">
                    <a name="%(action_order_report_so_salesteam)d" type="action">
                        Sales
                    </a>
                </div>
                <div groups="account.group_account_invoice" name="invoices_report">
                    <a name="%(action_account_invoice_report_salesteam)d" type="action">
                        Invoices
                    </a>
                </div>
            </xpath>

        </data>
        </field>
    </record>

    <record id="sales_team.mail_activity_type_action_config_sales" model="ir.actions.act_window">
        <field name="domain">['|', ('res_model', '=', False), ('res_model', 'in', ['sale.order', 'res.partner', 'product.template', 'product.product'])]</field>
    </record>

</odoo>

```

## File: views\mail_activity_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <!-- Activity types config -->
    <record id="mail_activity_type_action_config_sale" model="ir.actions.act_window">
        <field name="name">Activity Types</field>
        <field name="res_model">mail.activity.type</field>
        <field name="view_mode">tree,form</field>
        <field name="domain">['|', ('res_model', '=', False), ('res_model', '=', 'sale.order')]</field>
        <field name="context">{'default_res_model': 'sale.order'}</field>
    </record>
    <menuitem id="sale_menu_config_activity_type"
        action="mail_activity_type_action_config_sale"
        parent="menu_sale_config"
        groups="base.group_no_one"/>
</odoo>
```

## File: views\payment_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- Include sale-related values in payment checkout form to pass them to the client -->
    <template id="payment_checkout_inherit" inherit_id="payment.checkout">
        <xpath expr="//form[@name='o_payment_checkout']" position="attributes">
            <attribute name="t-att-data-sale-order-id">sale_order_id</attribute>
        </xpath>
    </template>

    <!-- Include sale-related values in payment manage form to pass them to the client -->
    <template id="payment_manage_inherit" inherit_id="payment.manage">
        <xpath expr="//form[@name='o_payment_manage']" position="attributes">
            <attribute name="t-att-data-sale-order-id">sale_order_id</attribute>
        </xpath>
    </template>

</odoo>

```

## File: views\payment_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="acquirer_form_inherit_sale" model="ir.ui.view">
            <field name="name">payment.acquirer.form.inherit.sale.payment</field>
            <field name="model">payment.acquirer</field>
            <field name="inherit_id" ref="payment.payment_acquirer_form"/>
            <field name="arch" type="xml">
                <xpath expr="//group[@name='payment_form']" position="inside">
                    <field name="so_reference_type" attrs="{'invisible': [('provider', '!=', 'transfer')]}"/>
                </xpath>
            </field>
        </record>

        <record id="transaction_form_inherit_sale" model="ir.ui.view">
            <field name="name">payment.transaction.form.inherit.sale.payment</field>
            <field name="model">payment.transaction</field>
            <field name="inherit_id" ref="payment.payment_transaction_form"/>
            <field name="arch" type="xml">
                <xpath expr="//div[hasclass('oe_button_box')]" position="inside">
                    <button name="action_view_sales_orders" type="object"
                            class="oe_stat_button" icon="fa-money"
                            attrs="{'invisible': [('sale_order_ids_nbr', '=', 0)]}">
                        <field name="sale_order_ids_nbr" widget="statinfo" string="Sales Order(s)"/>
                    </button>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\product_packaging_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="product_packaging_form_view_sale" model="ir.ui.view">
        <field name="name">product.packaging.form.view.sale</field>
        <field name="model">product.packaging</field>
        <field name="inherit_id" ref="product.product_packaging_form_view"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='group_product']" position="inside">
                <field name="sales"/>
            </xpath>
        </field>
    </record>

    <record id="product_packaging_tree_view_sale" model="ir.ui.view">
        <field name="name">product.packaging.tree.view.sale</field>
        <field name="model">product.packaging</field>
        <field name="inherit_id" ref="product.product_packaging_tree_view"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='product_uom_id']" position="after">
                <field name="sales" optional="show"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\product_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <record model="ir.ui.view" id="product_template_sale_form_view">
        <field name="name">product.template.sales</field>
        <field name="model">product.template</field>
        <field name="inherit_id" ref="product.product_template_form_view"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='sale']" position="attributes">
                <attribute name="invisible">0</attribute>
            </xpath>
            <field name="product_tooltip" position="after">
                <label for="product_tooltip" string="" attrs="{'invisible': ['|', ('type', 'not in', ('product', 'consu')), ('invoice_policy', '!=', 'order')]}"/>
                <div attrs="{'invisible': ['|', ('type', 'not in', ('product', 'consu')), ('invoice_policy', '!=', 'order')]}" class="font-italic text-muted">
                    You can invoice them before they are delivered.
                </div>
                <label for="product_tooltip" string="" attrs="{'invisible': ['|', ('type', 'not in', ('product', 'consu')), ('invoice_policy', '!=', 'delivery')]}"/>
                <div attrs="{'invisible': ['|', ('type', 'not in', ('product', 'consu')), ('invoice_policy', '!=', 'delivery')]}" class="font-italic text-muted">
                    Invoice after delivery, based on quantities delivered, not ordered.
                </div>
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
        <field name="name">res.config.settings.view.form.inherit.sale</field>
        <field name="model">res.config.settings</field>
        <field name="priority" eval="10"/>
        <field name="inherit_id" ref="base.res_config_settings_view_form" />
        <field name="arch" type="xml">
            <xpath expr="//div[hasclass('settings')]" position="inside">
                <div class="app_settings_block o_not_app" data-string="Sales" string="Sales" data-key="sale_management" groups="sales_team.group_sale_manager">
                    <h2>Product Catalog</h2>
                    <div class="row mt16 o_settings_container" name="catalog_setting_container">
                        <div class="col-12 col-lg-6 o_setting_box" id="variant_options">
                            <div class="o_setting_left_pane">
                                <field name="group_product_variant"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="group_product_variant"/>
                                <a href="https://www.odoo.com/documentation/15.0/applications/sales/sales/products_prices/products/variants.html" title="Documentation" class="o_doc_link" target="_blank"></a>
                                <div class="text-muted">
                                    Sell variants of a product using attributes (size, color, etc.)
                                </div>
                                <div class="content-group" attrs="{'invisible': [('group_product_variant','=',False)]}">
                                    <div class="mt8">
                                        <button name="%(product.attribute_action)d" icon="fa-arrow-right" type="action" string="Attributes" class="btn-link"/>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box" id="product_configurator">
                            <div class="o_setting_left_pane">
                                <field name="module_sale_product_configurator"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="module_sale_product_configurator"/>
                                <div class="text-muted">
                                    Select product attributes and optional products from the sales order
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box" id="product_matrix">
                            <div class="o_setting_left_pane">
                                <field name="module_sale_product_matrix"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="module_sale_product_matrix" string="Variant Grid Entry"/>
                                <div class="text-muted">
                                    Add several variants to an order from a grid
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box" id="uom_settings">
                            <div class="o_setting_left_pane">
                                <field name="group_uom"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="group_uom"/>
                                <div class="text-muted">
                                    Sell and purchase products in different units of measure
                                </div>
                                <div class="content-group" attrs="{'invisible': [('group_uom','=',False)]}">
                                    <div class="mt8">
                                        <button name="%(uom.product_uom_categ_form_action)d" icon="fa-arrow-right" type="action" string="Units of Measure" class="btn-link"/>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box"
                            id="email_template"
                            title="Sending an email is useful if you need to share specific information or content about a product (instructions, rules, links, media, etc.). Create and set the email template from the product detail form (in Sales tab).">
                            <div class="o_setting_left_pane">
                                <field name="module_product_email_template"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="module_product_email_template" string="Deliver Content by Email"/>
                                <div class="text-muted">
                                    Send a product-specific email once the invoice is validated
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box"
                            id="stock_packaging"
                            title="Ability to select a package type in sales orders and to force a quantity that is a multiple of the number of units per package.">
                            <div class="o_setting_left_pane">
                                <field name="group_stock_packaging"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="group_stock_packaging"/>
                                <div class="text-muted">
                                    Sell products by multiple of unit # per package
                                </div>
                            </div>
                        </div>
                    </div>
                    <h2>Pricing</h2>
                    <div class="row mt16 o_settings_container" id="pricing_setting_container">
                      <div class="col-12 col-lg-6 o_setting_box"
                           id="discount_sale_order_lines"
                           title="Apply manual discounts on sales order lines or display discounts computed from pricelists (option to activate in the pricelist configuration).">
                           <div class="o_setting_left_pane">
                               <field name="group_discount_per_so_line"/>
                           </div>
                           <div class="o_setting_right_pane">
                               <label for="group_discount_per_so_line"/>
                               <div class="text-muted">
                                   Grant discounts on sales order lines
                               </div>
                           </div>
                       </div>
                        <div class="col-12 col-lg-6 o_setting_box"
                            id="coupon_settings"
                            title="Boost your sales with two kinds of discount programs: promotions and coupon codes. Specific conditions can be set (products, customers, minimum purchase amount, period). Rewards can be discounts (% or amount) or free products.">
                            <div class="o_setting_left_pane">
                                <field name="module_sale_coupon"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="module_sale_coupon"/>
                                <div class="text-muted" id="sale_coupon">
                                    Manage promotion &amp; coupon programs
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box" id="pricelist_configuration">
                            <div class="o_setting_left_pane">
                                <field name="group_product_pricelist"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="group_product_pricelist"/>
                                <a href="https://www.odoo.com/documentation/15.0/applications/sales/sales/products_prices/prices/pricing.html" title="Documentation" class="o_doc_link" target="_blank"></a>
                                <div class="text-muted">
                                    Set multiple prices per product, automated discounts, etc.
                                </div>
                                <div class="content-group" attrs="{'invisible': [('group_product_pricelist' ,'=', False)]}">
                                    <div class="mt16">
                                        <field name="group_sale_pricelist" invisible="1"/>
                                        <field name="product_pricelist_setting" widget="radio" class="o_light_label"/>
                                    </div>
                                    <div class="mt8">
                                        <button name="%(product.product_pricelist_action2)d" icon="fa-arrow-right" type="action" string="Pricelists" groups="product.group_product_pricelist" class="btn-link"/>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box"
                            id="auth_signup_documents"
                            title=" To send invitations in B2B mode, open a contact or select several ones in list view and click on 'Portal Access Management' option in the dropdown menu *Action*.">
                            <div class="o_setting_left_pane">
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="auth_signup_uninvited"/>
                                <div class="text-muted">
                                    Let your customers log in to see their documents
                                </div>
                                <div class="mt8">
                                    <field name="auth_signup_uninvited" class="o_light_label" widget="radio" options="{'horizontal': true}" required="True"/>
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box"
                            id="show_margins"
                            title="The margin is computed as the sum of product sales prices minus the cost set in their detail form.">
                            <div class="o_setting_left_pane">
                                <field name="module_sale_margin"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="module_sale_margin"/>
                                <div class="text-muted">
                                    Show margins on orders
                                </div>
                            </div>
                        </div>
                    </div>
                    <h2>Quotations &amp; Orders</h2>
                    <div class="row mt16 o_settings_container" name="quotation_order_setting_container">
                        <div class="col-12 col-lg-6 o_setting_box" id="sale_config_online_confirmation_sign">
                            <div class="o_setting_left_pane">
                                <field name="portal_confirmation_sign"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="portal_confirmation_sign"/>
                                <a href="https://www.odoo.com/documentation/15.0/applications/sales/sales/send_quotations/get_signature_to_validate.html" title="Documentation" class="mr-2 o_doc_link" target="_blank"></a>
                                <span class="fa fa-lg fa-building-o" title="Values set here are company-specific." groups="base.group_multi_company"/>
                                <div class="text-muted">
                                    Request an online signature to confirm orders
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box" id="sale_config_online_confirmation_pay">
                            <div class="o_setting_left_pane">
                                <field name="portal_confirmation_pay"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="portal_confirmation_pay"/>
                                <a href="https://www.odoo.com/documentation/15.0/applications/sales/sales/send_quotations/get_paid_to_validate.html" title="Documentation" class="mr-2 o_doc_link" target="_blank"></a>
                                <span class="fa fa-lg fa-building-o" title="Values set here are company-specific." groups="base.group_multi_company"/>
                                <div class="text-muted">
                                    Request an online payment to confirm orders
                                </div>
                                <div class="mt8" attrs="{'invisible': [('portal_confirmation_pay', '=', False)]}">
                                    <button name='%(payment.action_payment_acquirer)d' icon="fa-arrow-right" type="action" string="Payment Acquirers" class="btn-link"/>
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box" id="confirmation_email_setting" attrs="{'invisible': [('portal_confirmation_pay', '=', False) , ('portal_confirmation_sign', '=', False)]}" groups="base.group_no_one">
                            <div class="o_setting_right_pane">
                                <span class="o_form_label">Confirmation Email</span>
                                <div class="text-muted">
                                    Automatic email sent after the customer has signed or paid online
                                </div>
                                <div class="row mt16">
                                    <label for="confirmation_mail_template_id" class="col-lg-4 o_light_label"/>
                                    <field name="confirmation_mail_template_id" class="oe_inline"/>
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box" id="invoice_delivery_addresses">
                            <div class="o_setting_left_pane">
                                <field name="group_sale_delivery_address"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="group_sale_delivery_address"/>
                                <a href="https://www.odoo.com/documentation/15.0/applications/sales/sales/send_quotations/different_addresses.html" title="Documentation" class="o_doc_link" target="_blank"></a>
                                <div class="text-muted">
                                    Select specific invoice and delivery addresses
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box" id="quotation_validity_days">
                            <div class="o_setting_left_pane">
                                <field name="use_quotation_validity_days"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="use_quotation_validity_days"/>
                                <span class="fa fa-lg fa-building-o" title="Values set here are company-specific." aria-label="Values set here are company-specific." groups="base.group_multi_company" role="img"/>
                                <div class="text-muted">
                                    Set a default validity on your quotations
                                </div>
                                <div class="content-group"  attrs="{'invisible': [('use_quotation_validity_days','=',False)]}">
                                    <div class="mt16">
                                        <span class="col-lg-3">Default Limit: <field name="quotation_validity_days" attrs="{'required': [('use_quotation_validity_days', '=', True)]}"/> days</span>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box" id="order_warnings">
                            <div class="o_setting_left_pane">
                                <field name="group_warning_sale"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="group_warning_sale" string="Sale Warnings"/>
                                <div class="text-muted">
                                    Get warnings in orders for products or customers
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box" id="no_edit_order">
                            <div class="o_setting_left_pane">
                                <field name="group_auto_done_setting"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="group_auto_done_setting"/>
                                <div class="text-muted">
                                    No longer edit orders once confirmed
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box" id="proforma_configuration">
                            <div class="o_setting_left_pane">
                                <field name="group_proforma_sales"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="group_proforma_sales"/>
                                <div class="text-muted">
                                    Allows you to send Pro-Forma Invoice to your customers
                                </div>
                            </div>
                        </div>
                    </div>
                    <h2 class="mt32">Shipping</h2>
                    <div class="row mt16 o_settings_container" name="shipping_setting_container">
                        <div class="col-12 col-lg-6 o_setting_box" id="delivery">
                            <div class="o_setting_left_pane">
                                <field name="module_delivery"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="module_delivery"/>
                                <div class="text-muted" id="delivery_carrier">
                                    Compute shipping costs on orders
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box" id="ups">
                            <div class="o_setting_left_pane">
                                <field name="module_delivery_ups" widget="upgrade_boolean"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="module_delivery_ups"/>
                                <div class="text-muted">
                                    Compute shipping costs and ship with UPS
                                </div>
                                <div class="content-group">
                                    <div id="sale_delivery_ups"/>
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box" id="shipping_costs_dhl">
                            <div class="o_setting_left_pane">
                                <field name="module_delivery_dhl" widget="upgrade_boolean"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="module_delivery_dhl"/>
                                <div class="text-muted">
                                    Compute shipping costs and ship with DHL
                                </div>
                                <div class="content-group">
                                    <div id="sale_delivery_dhl"></div>
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box" id="shipping_costs_fedex">
                            <div class="o_setting_left_pane">
                                <field name="module_delivery_fedex" widget="upgrade_boolean"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="module_delivery_fedex"/>
                                <div class="text-muted">
                                    Compute shipping costs and ship with FedEx
                                </div>
                                <div class="content-group">
                                    <div id="sale_delivery_fedex"/>
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box" id="shipping_costs_usps">
                            <div class="o_setting_left_pane">
                                <field name="module_delivery_usps" widget="upgrade_boolean"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="module_delivery_usps"/>
                                <div class="text-muted">
                                    Compute shipping costs and ship with USPS
                                </div>
                                <div class="content-group">
                                    <div id="sale_delivery_usps"/>
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box" id="shipping_costs_bpost">
                            <div class="o_setting_left_pane">
                                <field name="module_delivery_bpost" widget="upgrade_boolean"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="module_delivery_bpost"/>
                                <div class="text-muted">
                                    Compute shipping costs and ship with bpost
                                </div>
                                <div class="content-group">
                                    <div id="sale_delivery_bpost"/>
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box" id="shipping_costs_easypost">
                            <div class="o_setting_left_pane">
                                <field name="module_delivery_easypost" widget="upgrade_boolean"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="module_delivery_easypost"/>
                                <div class="text-muted">
                                    Compute shipping costs and ship with Easypost
                                </div>
                                <div class="content-group">
                                    <div id="sale_delivery_easypost"/>
                                </div>
                            </div>
                        </div>
                    </div>
                    <h2>Invoicing</h2>
                    <div class="row mt16 o_settings_container" name="invoicing_setting_container">
                        <div id="sales_settings_invoicing_policy"
                             class="col-12 col-lg-6 o_setting_box"
                             title="This default value is applied to any new product created. This can be changed in the product detail form.">
                            <div class="o_setting_right_pane">
                                <label for="default_invoice_policy"/>
                                <a href="https://www.odoo.com/documentation/15.0/applications/sales/sales/invoicing/invoicing_policy.html" title="Documentation" class="o_doc_link" target="_blank"></a>
                                <div class="text-muted">
                                    Quantities to invoice from sales orders
                                </div>
                                <div class="content-group">
                                    <div class="mt16">
                                        <field name="default_invoice_policy" class="o_light_label" widget="radio"/>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="col-xs-12 col-md-6 o_setting_box"
                             id="automatic_invoicing"
                             attrs="{'invisible': ['|', ('default_invoice_policy', '!=', 'order'), ('portal_confirmation_pay', '=', False)]}">
                            <div class="o_setting_left_pane">
                                <field name="automatic_invoice"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="automatic_invoice"/>
                                <div class="text-muted">
                                    Generate the invoice automatically when the online payment is confirmed
                                </div>
                                <div  attrs="{'invisible': [('automatic_invoice','=',False)]}" groups="base.group_no_one">
                                    <label for="invoice_mail_template_id" class="o_light_label"/>
                                    <field name="invoice_mail_template_id" class="oe_inline" options="{'no_create': True}"/>
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box" id="down_payments">
                            <div class="o_setting_left_pane"/>
                            <div class="o_setting_right_pane">
                                <span class="o_form_label">Down Payments</span>
                                <a href="https://www.odoo.com/documentation/15.0/applications/sales/sales/invoicing/down_payment.html" title="Documentation" class="o_doc_link" target="_blank"></a>
                                <div class="text-muted">
                                    Product used for down payments
                                </div>
                                <div class="text-muted">
                                    <field name="deposit_default_product_id" context="{'default_detailed_type': 'service'}"/>
                                </div>
                            </div>
                        </div>
                    </div>
                    <h2 class="mt32">Connectors</h2>
                    <div class="row mt16 o_settings_container" id="connectors_setting_container">
                        <div class="col-12 col-lg-6 o_setting_box" id="amazon_connector">
                            <div class="o_setting_left_pane">
                                <field name="module_sale_amazon" widget="upgrade_boolean"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="module_sale_amazon"/>
                                <a href="https://www.odoo.com/documentation/15.0/applications/sales/sales/amazon_connector/setup.html" title="Documentation" class="o_doc_link" target="_blank"></a>
                                <div class="text-muted">
                                    Import Amazon orders and sync deliveries
                                </div>
                                <div class="content-group"
                                     name="amazon_connector"
                                     attrs="{'invisible': [('module_sale_amazon', '=', False)]}"/>
                            </div>
                        </div>
                    </div>
                    <div id="sale_ebay"/>
                </div>
            </xpath>
        </field>
    </record>

    <record id="action_sale_config_settings" model="ir.actions.act_window">
        <field name="name">Settings</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">res.config.settings</field>
        <field name="view_id" ref="res_config_settings_view_form"/>
        <field name="view_mode">form</field>
        <field name="target">inline</field>
        <field name="context">{'module' : 'sale_management', 'bin_size': False}</field>
    </record>

    <menuitem id="menu_sale_general_settings"
        name="Settings"
        parent="menu_sale_config"
        sequence="0"
        action="action_sale_config_settings"
        groups="base.group_system"/>

</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0"?>
<odoo>
        <record id="act_res_partner_2_sale_order" model="ir.actions.act_window">
            <field name="name">Quotations and Sales</field>
            <field name="res_model">sale.order</field>
            <field name="view_mode">tree,form,graph</field>
            <field name="context">{'default_partner_id': active_id}</field>
            <field name="groups_id" eval="[(4, ref('sales_team.group_sale_salesman'))]"/>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Create a new quotation, the first step of a new sale!
              </p><p>
                Once the quotation is confirmed by the customer, it becomes a sales order.<br/> You will be able to create an invoice and collect the payment.
              </p>
            </field>
        </record>

        <!-- Partner kanban view inherte -->
        <record model="ir.ui.view" id="crm_lead_partner_kanban_view">
            <field name="name">res.partner.kanban.saleorder.inherit</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="base.res_partner_kanban_view"/>
            <field name="priority" eval="20"/>
            <field name="groups_id" eval="[(4, ref('sales_team.group_sale_salesman'))]"/>
            <field name="arch" type="xml">
                <field name="mobile" position="after">
                    <field name="sale_order_count"/>
                </field>
                <xpath expr="//span[hasclass('oe_kanban_partner_links')]" position="inside">
                    <span t-if="record.sale_order_count.value>0" class="badge badge-pill"><i class="fa fa-fw fa-usd" role="img" aria-label="Sale orders" title="Sales orders"/><t t-esc="record.sale_order_count.value"/></span>
                </xpath>
            </field>
        </record>

        <record id="res_partner_view_buttons" model="ir.ui.view">
            <field name="name">res.partner.view.buttons</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="base.view_partner_form" />
            <field name="priority" eval="3"/>
            <field name="groups_id" eval="[(4, ref('sales_team.group_sale_salesman'))]"/>
            <field name="arch" type="xml">
                <div name="button_box" position="inside">
                    <button class="oe_stat_button" type="object" name="action_view_sale_order"
                        groups="sales_team.group_sale_salesman"
                        icon="fa-usd">
                        <field string="Sales" name="sale_order_count" widget="statinfo"/>
                    </button>
                </div>
                <page name="internal_notes" position="inside">
                    <group colspan="2" col="2" groups="sale.group_warning_sale">
                        <separator string="Warning on the Sales Order" colspan="4" />
                            <field name="sale_warn" nolabel="1" />
                            <field name="sale_warn_msg" colspan="3" nolabel="1"
                                    attrs="{'required':[('sale_warn','!=', False), ('sale_warn','!=','no-message')], 'invisible':[('sale_warn','in',(False,'no-message'))]}"/>
                    </group>
                </page>
            </field>
        </record>

        <record id="res_partner_view_form_payment_defaultcreditcard" model="ir.ui.view">
            <field name="name">res.partner.view.form.payment.defaultcreditcard</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="payment.view_partners_form_payment_defaultcreditcard"/>
            <field name="arch" type="xml">
                <button name="%(payment.action_payment_token)d" position="attributes">
                    <attribute name="groups">sales_team.group_sale_salesman</attribute>
                </button>
            </field>
        </record>

        <record id="res_partner_view_form_property_inherit" model="ir.ui.view">
            <field name="name">res.partner.view.form.property.inherit</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="account.view_partner_property_form"/>
            <field name="arch" type="xml">
                <group name="fiscal_information" position="attributes">
                    <attribute name="groups">account.group_account_invoice, sales_team.group_sale_salesman</attribute>
                </group>
                <field name="property_payment_term_id" position="attributes">
                    <attribute name="groups">account.group_account_invoice, sales_team.group_sale_salesman</attribute>
                </field>
                <field name="property_supplier_payment_term_id" position="attributes">
                    <attribute name="groups">account.group_account_invoice, sales_team.group_sale_salesman</attribute>
                </field>
            </field>
        </record>
</odoo>

```

## File: views\sale_onboarding_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- ONBOARDING STEPS -->
    <template id="onboarding_quotation_layout_step">
        <t t-call="base.onboarding_step">
            <t t-set="title">Quotation Layout</t>
            <t t-set="description">Customize the look of your quotations.</t>
            <t t-set="done_icon" t-value="'fa-star'" />
            <t t-set="done_text">Looks great!</t>
            <t t-set="btn_text">Customize</t>
            <t t-set="model" t-value="'base.document.layout'" />
            <t t-set="method" t-value="'action_open_base_document_layout'" />
            <t t-set="state" t-value="state.get('account_onboarding_invoice_layout_state')" />
        </t>
    </template>
    <template id="sale_onboarding_order_confirmation_step">
        <t t-call="base.onboarding_step">
            <t t-set="title">Order Confirmation</t>
            <t t-set="description">Choose between electronic signatures or online payments.</t>
            <t t-set="btn_text">Set payments</t>
            <t t-set="method" t-value="'action_open_sale_onboarding_payment_acquirer'" />
            <t t-set="model" t-value="'res.company'" />
            <t t-set="state" t-value="state.get('sale_onboarding_order_confirmation_state')" />
        </t>
    </template>
        <template id="sale_onboarding_sample_quotation_step">
        <t t-call="base.onboarding_step">
            <t t-set="title">Sample Quotation</t>
            <t t-set="description">Send a quotation to test the customer portal.</t>
            <t t-set="btn_text">Send sample</t>
            <t t-set="method" t-value="'action_open_sale_onboarding_sample_quotation'" />
            <t t-set="model" t-value="'res.company'" />
            <t t-set="state" t-value="state.get('sale_onboarding_sample_quotation_state')" />
        </t>
    </template>

    <!-- ONBOARDING PANEL-->
    <template id="sale_quotation_onboarding_panel" name="sale.quotation.onboarding.panel">
        <t t-call="base.onboarding_container">
            <t t-set="classes" t-value="'o_onboarding_violet'" />
            <t t-set="bg_image" t-value="'/sale/static/src/img/sale_quotation_onboarding_bg.jpg'"/>
            <t t-set="close_method" t-value="'action_close_sale_quotation_onboarding'" />
            <t t-set="close_model" t-value="'res.company'" />
            <t t-call="base.onboarding_company_step" name="company_step" />
            <t t-call="sale.onboarding_quotation_layout_step" name="quotation_layout_step" />
            <t t-call="sale.sale_onboarding_order_confirmation_step" name="payment_acquirer_step" />
            <t t-call="sale.sale_onboarding_sample_quotation_step" name="sample_quotation_step" />
        </t>
    </template>
    <!-- ORDER CONFIRMATION -->
    <record id="sale_onboarding_order_confirmation_form" model="ir.ui.view">
        <field name="name">sale.order.confirmation.onboarding.form</field>
        <field name="model">sale.payment.acquirer.onboarding.wizard</field>
        <field name="inherit_id" ref="payment.payment_acquirer_onboarding_wizard_form" />
        <field name="mode">primary</field>
        <field name="priority">1000</field>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='left-column']" position="attributes">
                <attribute name="class">col col-4</attribute>
            </xpath>
        </field>
    </record>
    <record id="action_open_sale_onboarding_payment_acquirer_wizard" model="ir.actions.act_window">
        <field name="name">Choose how to confirm quotations</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">sale.payment.acquirer.onboarding.wizard</field>
        <field name="view_mode">form</field>
        <field name="view_id" ref="sale_onboarding_order_confirmation_form" />
        <field name="target">new</field>
    </record>
</odoo>

```

## File: views\sale_portal_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="portal_my_home_menu_sale" name="Portal layout : sales menu entries" inherit_id="portal.portal_breadcrumbs" priority="20">
        <xpath expr="//ol[hasclass('o_portal_submenu')]" position="inside">
            <li t-if="page_name == 'quote' or sale_order and sale_order.state in ('sent', 'cancel')" t-attf-class="breadcrumb-item #{'active ' if not sale_order else ''}">
                <a t-if="sale_order" t-attf-href="/my/quotes?{{ keep_query() }}">Quotations</a>
                <t t-else="">Quotations</t>
            </li>
            <li t-if="page_name == 'order' or sale_order and sale_order.state not in ('sent', 'cancel')" t-attf-class="breadcrumb-item #{'active ' if not sale_order else ''}">
                <a t-if="sale_order" t-attf-href="/my/orders?{{ keep_query() }}">Sales Orders</a>
                <t t-else="">Sales Orders</t>
            </li>
            <li t-if="sale_order" class="breadcrumb-item active">
                <span t-field="sale_order.type_name"/>
                <t t-esc="sale_order.name"/>
            </li>
        </xpath>
    </template>

    <template id="portal_my_home_sale" name="Show Quotations / Sales Orders" customize_show="True" inherit_id="portal.portal_my_home" priority="20">
        <xpath expr="//div[hasclass('o_portal_docs')]" position="inside">
            <t t-call="portal.portal_docs_entry">
                <t t-set="title">Quotations</t>
                <t t-set="url" t-value="'/my/quotes'"/>
                <t t-set="placeholder_count" t-value="'quotation_count'"/>
            </t>
            <t t-call="portal.portal_docs_entry">
                <t t-set="title">Sales Orders</t>
                <t t-set="url" t-value="'/my/orders'"/>
                <t t-set="placeholder_count" t-value="'order_count'"/>
            </t>
        </xpath>
    </template>

    <template id="portal_my_quotations" name="My Quotations">
        <t t-call="portal.portal_layout">
            <t t-set="breadcrumbs_searchbar" t-value="True"/>

            <t t-call="portal.portal_searchbar">
                <t t-set="title">Quotations</t>
            </t>
            <t t-if="not quotations">
                <p>There are currently no quotations for your account.</p>
            </t>
            <t t-if="quotations" t-call="portal.portal_table">
                <thead>
                    <tr class="active">
                        <th>Quotation #</th>
                        <th class="text-right">Quotation Date</th>
                        <th class="text-right">Valid Until</th>
                        <th class="text-center"/>
                        <th class="text-right">Total</th>
                    </tr>
                </thead>
                <t t-foreach="quotations" t-as="quotation">
                    <tr>
                        <td><a t-att-href="quotation.get_portal_url()"><t t-esc="quotation.name"/></a></td>
                        <td class="text-right"><span t-field="quotation.date_order"/></td>
                        <td class="text-right"><span t-field="quotation.validity_date"/></td>
                        <td class="text-center">
                            <span t-if="quotation.state == 'cancel'" class="badge badge-pill badge-secondary"><i class="fa fa-fw fa-remove"/> Cancelled</span>
                            <span t-if="quotation.is_expired" class="badge badge-pill badge-secondary"><i class="fa fa-fw fa-clock-o"/> Expired</span>
                        </td>
                        <td class="text-right">
                            <span t-field="quotation.amount_total"/>
                        </td>
                    </tr>
                </t>
            </t>
        </t>
    </template>

    <template id="portal_my_orders" name="My Sales Orders">
        <t t-call="portal.portal_layout">
            <t t-set="breadcrumbs_searchbar" t-value="True"/>

            <t t-call="portal.portal_searchbar">
                <t t-set="title">Sales Orders</t>
            </t>
            <t t-if="not orders">
                <p>There are currently no orders for your account.</p>
            </t>
            <t t-if="orders" t-call="portal.portal_table">
                <thead>
                    <tr class="active">
                        <th>
                            <span class='d-none d-md-inline'>Sales Order #</span>
                            <span class='d-block d-md-none'>Ref.</span>
                        </th>
                        <th class="text-right">Order Date</th>
                        <th class="text-center"/>
                        <th class="text-right">Total</th>
                    </tr>
                </thead>
                <t t-foreach="orders" t-as="order">
                    <tr>
                        <td><a t-att-href="order.get_portal_url()"><t t-esc="order.name"/></a></td>
                        <td class="text-right">
                            <span t-field="order.date_order" t-options="{'widget': 'date'}"/>&amp;nbsp;
                            <span class='d-none d-md-inline' t-field="order.date_order" t-options="{'time_only': True}"/>
                        </td>
                        <td class="text-center">
                            <span t-if="order.state == 'done'"  class="badge badge-pill badge-success">
                                <i class="fa fa-fw fa-check" role="img" aria-label="Done" title="Done"/>Done
                            </span>
                        </td>
                        <td class="text-right"><span t-field="order.amount_total"/></td>
                    </tr>
                </t>
            </t>
        </t>
    </template>

    <!-- Complete page of the sale_order -->
    <template id="sale_order_portal_template" name="Sales Order Portal Template" inherit_id="portal.portal_sidebar" primary="True">
        <xpath expr="//div[hasclass('o_portal_sidebar')]" position="inside">
            <t t-set="o_portal_fullwidth_alert" groups="sales_team.group_sale_salesman">
                <t t-call="portal.portal_back_in_edit_mode">
                    <t t-set="backend_url" t-value="'/web#model=%s&amp;id=%s&amp;action=%s&amp;view_type=form' % (sale_order._name, sale_order.id, action.id)"/>
                </t>
            </t>

            <div class="row mt16 o_portal_sale_sidebar">
                <!-- Sidebar -->
                <t t-call="portal.portal_record_sidebar">
                    <t t-set="classes" t-value="'col-lg-auto d-print-none'"/>

                    <t t-set="title">
                        <h2 class="mb-0"><b t-field="sale_order.amount_total" data-id="total_amount"/> </h2>
                    </t>
                    <t t-set="entries">
                        <ul class="list-group list-group-flush flex-wrap flex-row flex-lg-column">
                            <li class="list-group-item flex-grow-1">
                                <a t-if="sale_order.has_to_be_signed(True)" role="button" class="btn btn-primary btn-block mb8" data-toggle="modal" data-target="#modalaccept" href="#">
                                    <i class="fa fa-check"/><t t-if="sale_order.has_to_be_paid(True)"> Sign &amp; Pay</t><t t-else=""> Accept &amp; Sign</t>
                                </a>
                                <a t-elif="sale_order.has_to_be_paid(True)" role="button" id="o_sale_portal_paynow" data-toggle="modal" data-target="#modalaccept" href="#" t-att-class="'btn-block mb8 %s' % ('btn btn-light' if sale_order.transaction_ids else 'btn btn-primary')" >
                                    <i class="fa fa-check"/> <t t-if="not sale_order.signature">Accept &amp; Pay</t><t t-else="">Pay Now</t>
                                </a>
                                <div class="o_download_pdf btn-toolbar flex-sm-nowrap">
                                    <div class="btn-group flex-grow-1 mr-1 mb-1">
                                        <a class="btn btn-secondary btn-block o_download_btn" t-att-href="sale_order.get_portal_url(report_type='pdf', download=True)" title="Download"><i class="fa fa-download"/> Download</a>
                                    </div>
                                    <div class="btn-group flex-grow-1 mb-1">
                                        <a class="btn btn-secondary btn-block o_print_btn o_portal_invoice_print" t-att-href="sale_order.get_portal_url(report_type='pdf')" id="print_invoice_report" title="Print" target="_blank"><i class="fa fa-print"/> Print</a>
                                    </div>
                                </div>
                            </li>

                            <li class="navspy list-group-item pl-0 flex-grow-1" t-ignore="true" role="complementary">
                                <ul class="nav flex-column bs-sidenav"></ul>
                            </li>

                            <t t-if="not sale_order.is_expired and sale_order.state in ['draft', 'sent']">
                                <li t-if="sale_order.validity_date" class="list-group-item">
                                    <small><b class="text-muted">This offer expires on</b></small>
                                    <div t-field="sale_order.validity_date"></div>
                                </li>
                                <li t-if="sale_order.amount_undiscounted - sale_order.amount_untaxed &gt; 0.01" class="list-group-item flex-grow-1">
                                    <small><b class="text-muted">Your advantage</b></small>
                                    <small>
                                        <b t-field="sale_order.amount_undiscounted"
                                            t-options='{"widget": "monetary", "display_currency": sale_order.pricelist_id.currency_id}'
                                            style="text-decoration: line-through"
                                            class="d-block mt-1"
                                            data-id="amount_undiscounted" />
                                    </small>
                                    <t t-if="sale_order.amount_untaxed == sale_order.amount_total">
                                        <h4 t-field="sale_order.amount_total" class="text-success" data-id="total_amount"/>
                                    </t>
                                    <t t-else="">
                                        <h4 t-field="sale_order.amount_untaxed" class="text-success mb-0" data-id="total_untaxed"/>
                                        <small>(<span t-field="sale_order.amount_total" data-id="total_amount"/> Incl. tax)</small>
                                    </t>
                                </li>
                            </t>

                            <li t-if="sale_order.user_id" class="list-group-item flex-grow-1">
                                <div class="small mb-1"><strong class="text-muted">Salesperson</strong></div>
                                <div class="row flex-nowrap">
                                    <div class="col flex-grow-0 pr-2">
                                        <img class="rounded-circle mr4 float-left o_portal_contact_img" t-att-src="image_data_uri(sale_order.user_id.avatar_1024)" alt="Contact"/>
                                    </div>
                                    <div class="col pl-0" style="min-width: 150px">
                                        <span t-field="sale_order.user_id" t-options='{"widget": "contact", "fields": ["name", "phone"], "no_marker": True}'/>
                                        <a href="#discussion" class="small"><i class="fa fa-comment"></i> Send message</a>
                                    </div>
                                </div>
                            </li>
                        </ul>
                    </t>
                </t>

                <!-- Page content -->
                <div id="quote_content" class="col-12 col-lg justify-content-end">

                    <!-- modal relative to the actions sign and pay -->
                    <div role="dialog" class="modal fade" id="modalaccept">
                        <div class="modal-dialog" t-if="sale_order.has_to_be_signed(True)">
                            <form id="accept" method="POST" t-att-data-order-id="sale_order.id" t-att-data-token="sale_order.access_token" class="js_accept_json modal-content js_website_submit_form">
                                <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
                                <header class="modal-header">
                                    <h4 class="modal-title">Validate Order</h4>
                                    <button type="button" class="close" data-dismiss="modal" aria-label="Close">×</button>
                                </header>
                                <main class="modal-body" id="sign-dialog">
                                    <p>
                                        <span>By signing this proposal, I agree to the following terms:</span>
                                        <ul>
                                            <li><span>Accepted on the behalf of:</span> <b t-field="sale_order.partner_id.commercial_partner_id"/></li>
                                            <li><span>For an amount of:</span> <b data-id="total_amount" t-field="sale_order.amount_total"/></li>
                                            <li t-if="sale_order.payment_term_id"><span>With payment terms:</span> <b t-field="sale_order.payment_term_id.note"/></li>
                                        </ul>
                                    </p>
                                    <t t-call="portal.signature_form">
                                        <t t-set="call_url" t-value="sale_order.get_portal_url(suffix='/accept')"/>
                                        <t t-set="default_name" t-value="sale_order.partner_id.name"/>
                                    </t>
                                </main>
                            </form>
                        </div>

                        <div class="modal-dialog" t-if="not sale_order.has_to_be_signed(True) and sale_order.has_to_be_paid(True)">
                            <div class="modal-content">
                                <header class="modal-header">
                                    <h4 class="modal-title">Validate Order</h4>
                                    <button type="button" class="close" data-dismiss="modal" aria-label="Close">×</button>
                                </header>
                                <main class="modal-body" id="sign-dialog">
                                    <p>
                                        <span>By paying this proposal, I agree to the following terms:</span>
                                        <ul>
                                            <li><span>Accepted on the behalf of:</span> <b t-field="sale_order.partner_id.commercial_partner_id"/></li>
                                            <li><span>For an amount of:</span> <b data-id="total_amount" t-field="sale_order.amount_total"/></li>
                                            <li t-if="sale_order.payment_term_id"><span>With payment terms:</span> <b t-field="sale_order.payment_term_id.note"/></li>
                                        </ul>
                                    </p>
                                    <div t-if="acquirers or tokens" id="payment_method" class="text-left">
                                        <h3 class="mb24">Pay with</h3>
                                        <t t-call="payment.checkout"/>
                                    </div>
                                    <div t-else="" class="alert alert-warning">
                                        <strong>No suitable payment option could be found.</strong><br/>
                                        If you believe that it is an error, please contact the website administrator.
                                    </div>
                                </main>
                            </div>
                        </div>
                    </div>

                    <!-- modal relative to the action reject -->
                    <div role="dialog" class="modal fade" id="modaldecline">
                        <div class="modal-dialog">
                            <form id="decline" method="POST" t-attf-action="/my/orders/#{sale_order.id}/decline?access_token=#{sale_order.access_token}" class="modal-content">
                                <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
                                <header class="modal-header">
                                    <h4 class="modal-title">Reject This Quotation</h4>
                                    <button type="button" class="close" data-dismiss="modal" aria-label="Close">×</button>
                                </header>
                                <main class="modal-body">
                                    <p>
                                        Tell us why you are refusing this quotation, this will help us improve our services.
                                    </p>
                                    <textarea rows="4" name="decline_message" required="" placeholder="Your feedback..." class="form-control" />
                                </main>
                                <footer class="modal-footer">
                                    <button type="submit" t-att-id="sale_order.id" class="btn btn-danger"><i class="fa fa-times"></i> Reject</button>
                                    <button type="button" class="btn btn-primary" data-dismiss="modal">Cancel</button>
                                </footer>
                            </form>
                        </div>
                    </div>

                    <!-- status messages -->
                    <div t-if="message == 'sign_ok'" class="alert alert-success alert-dismissable d-print-none" role="status">
                        <button type="button" class="close" data-dismiss="alert" aria-label="Close">×</button>
                        <strong>Thank You!</strong><br/>
                        <t t-if="message == 'sign_ok' and sale_order.state in ['sale', 'done']">Your order has been confirmed.</t>
                        <t t-elif="message == 'sign_ok' and sale_order.has_to_be_paid()">Your order has been signed but still needs to be paid to be confirmed.</t>
                        <t t-else="">Your order has been signed.</t>
                    </div>

                    <div t-if="message == 'cant_reject'" class="alert alert-danger alert-dismissable d-print-none" role="alert">
                        <button type="button" class="close" data-dismiss="alert" aria-label="Close">×</button>
                        Your order is not in a state to be rejected.
                    </div>

                    <t t-if="sale_order.transaction_ids">
                        <t t-call="payment.transaction_status">
                            <t t-set="tx" t-value="sale_order.get_portal_last_transaction()"/>
                        </t>
                    </t>

                    <div t-if="sale_order.state == 'cancel'" class="alert alert-danger alert-dismissable d-print-none" role="alert">
                        <button type="button" class="close" data-dismiss="alert" aria-label="close">×</button>
                        <strong>This quotation has been canceled.</strong> <a role="button" href="#discussion"><i class="fa fa-comment"/> Contact us to get a new quotation.</a>
                    </div>

                    <div t-if="sale_order.is_expired" class="alert alert-warning alert-dismissable d-print-none" role="alert">
                        <button type="button" class="close" data-dismiss="alert" aria-label="close">×</button>
                        <strong>This offer expired!</strong> <a role="button" href="#discussion"><i class="fa fa-comment"/> Contact us to get a new quotation.</a>
                    </div>

                    <!-- main content -->
                    <div t-attf-class="card #{'pb-5' if report_type == 'html' else ''}" id="portal_sale_content">
                        <div t-call="sale.sale_order_portal_content"/>
                    </div>

                    <!-- bottom actions -->
                    <div t-if="sale_order.has_to_be_signed(True) or sale_order.has_to_be_paid(True)" class="row justify-content-center text-center d-print-none pt-1 pb-4">

                        <t t-if="sale_order.has_to_be_signed(True)">
                            <div class="col-sm-auto mt8">
                                <a role="button" class="btn btn-primary" data-toggle="modal" data-target="#modalaccept" href="#"><i class="fa fa-check"/><t t-if="sale_order.has_to_be_paid(True)"> Sign &amp; Pay</t><t t-else=""> Accept &amp; Sign</t></a>
                            </div>
                            <div class="col-sm-auto mt8">
                                <a role="button" class="btn btn-secondary" href="#discussion"><i class="fa fa-comment"/> Feedback</a>
                            </div>
                            <div class="col-sm-auto mt8">
                                <a role="button" class="btn btn-danger" data-toggle="modal" data-target="#modaldecline" href="#"> <i class="fa fa-times"/> Reject</a>
                            </div>
                        </t>
                        <div t-elif="sale_order.has_to_be_paid(True)" class="col-sm-auto mt8">
                            <a role="button" data-toggle="modal" data-target="#modalaccept" href="#" t-att-class="'%s' % ('btn btn-light' if sale_order.transaction_ids else 'btn btn-primary')">
                                <i class="fa fa-check"/> <t t-if="not sale_order.signature">Accept &amp; Pay</t><t t-else="">Pay Now</t>
                            </a>
                        </div>
                    </div>

                    <!-- chatter -->
                    <div id="sale_order_communication" class="mt-4">
                        <h2>History</h2>
                        <t t-call="portal.message_thread">
                            <t t-set="object" t-value="sale_order"/>
                        </t>
                    </div>
                </div><!-- // #quote_content -->
            </div>
        </xpath>
    </template>

    <!--
    Sales Order content : intro, informations, order lines, remarks, descriptions ....
    This template should contains all the printable element of the SO. This is the
    template rendered in PDF with the report engine.
    -->
    <template id="sale_order_portal_content" name="Sales Order Portal Content">
        <!-- Intro -->
        <div id="introduction" t-attf-class="pb-2 pt-3 #{'card-header bg-white' if report_type == 'html' else ''}">
          <h2 class="my-0">
                <t t-esc="sale_order.type_name"/>
                <em t-esc="sale_order.name"/>
            </h2>
        </div>

        <div t-attf-class="#{'card-body' if report_type == 'html' else ''}">
            <!-- Informations -->
            <div id="informations">
                <div t-if="sale_order.transaction_ids and not invoices and sale_order.state in ('sent', 'sale') and portal_confirmation == 'pay' and not success and not error" t-att-data-order-id="sale_order.id">
                    <t t-if="sale_order.transaction_ids">
                        <t t-call="payment.transaction_status">
                            <t t-set="tx" t-value="sale_order.get_portal_last_transaction()"/>
                        </t>
                    </t>
                </div>
                <div class="row" id="so_date">
                    <div class="mb-3 col-6">
                      <t t-if="sale_order.state in ['sale', 'done', 'cancel']">
                        <strong>Order Date:</strong>
                      </t>
                      <t t-else="">
                         <strong>Quotation Date:</strong>
                      </t>
                      <span t-field="sale_order.date_order" t-options='{"widget": "date"}'/>
                    </div>
                    <div class="mb-3 col-6" t-if="sale_order.validity_date and sale_order.state in ['draft', 'sent']">
                        <strong>Expiration Date:</strong> <span t-field="sale_order.validity_date" t-options='{"widget": "date"}'/>
                    </div>
                </div>
                <div class="row">
                    <div class="col-lg-6">
                        <strong t-if="sale_order.partner_shipping_id == sale_order.partner_invoice_id" class="d-block mb-1">Invoicing and Shipping Address:</strong>
                        <strong t-if="sale_order.partner_shipping_id != sale_order.partner_invoice_id" class="d-block mb-1">Invoicing Address:</strong>
                        <address t-field="sale_order.partner_invoice_id" t-options='{"widget": "contact", "fields": ["address", "name"], "no_marker": True}'/>
                    </div>
                     <t t-if="sale_order.partner_shipping_id != sale_order.partner_invoice_id">
                        <div id="shipping_address" class="col-lg-6">
                            <strong class="d-block mb-1">Shipping Address:</strong>
                            <address t-field="sale_order.partner_shipping_id" t-options='{"widget": "contact", "fields": ["address", "name"], "no_marker": True}'/>
                        </div>
                    </t>
                </div>

                <t t-set="invoices" t-value="[i for i in sale_order.invoice_ids if i.state not in ['draft', 'cancel']]"/>
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
                                    <span t-if="i.payment_state in ('paid', 'in_payment')" class="small badge badge-success orders_label_text_align"><i class="fa fa-fw fa-check"/> <b>Paid</b></span>
                                    <span t-elif="i.payment_state == 'reversed'" class="small badge badge-success orders_label_text_align"><i class="fa fa-fw fa-check"/> <b>Reversed</b></span>
                                    <span t-else="" class="small badge badge-info orders_label_text_align"><i class="fa fa-fw fa-clock-o"/> <b>Waiting Payment</b></span>
                                </div>
                            </t>
                        </ul>
                    </div>
                </div>
            </div>

            <section id="details" style="page-break-inside: auto;" class="mt32">
                <h3 id="details">Pricing</h3>

                <t t-set="display_discount" t-value="True in [line.discount > 0 for line in sale_order.order_line]"/>

                <div class="table-responsive">
                    <table t-att-data-order-id="sale_order.id" t-att-data-token="sale_order.access_token" class="table table-sm" id="sales_order_table">
                        <thead class="bg-100">
                            <tr>
                                <th class="text-left">Products</th>
                                <th class="text-right">Quantity</th>
                                <th t-attf-class="text-right {{ 'd-none d-sm-table-cell' if report_type == 'html' else '' }}">Unit Price</th>
                                <th t-if="display_discount" t-attf-class="text-right {{ 'd-none d-sm-table-cell' if report_type == 'html' else '' }}">
                                    <span>Disc.%</span>
                                </th>
                                <th t-attf-class="text-right {{ 'd-none d-md-table-cell' if report_type == 'html' else '' }}">
                                    <span>Taxes</span>
                                </th>
                                <th class="text-right" >
                                    <span groups="account.group_show_line_subtotals_tax_excluded">Amount</span>
                                    <span groups="account.group_show_line_subtotals_tax_included">Total Price</span>
                                </th>
                            </tr>
                        </thead>
                        <tbody class="sale_tbody">

                            <t t-set="current_subtotal" t-value="0"/>

                            <t t-foreach="sale_order.order_line" t-as="line">

                                <t t-set="current_subtotal" t-value="current_subtotal + line.price_subtotal" groups="account.group_show_line_subtotals_tax_excluded"/>
                                <t t-set="current_subtotal" t-value="current_subtotal + line.price_total" groups="account.group_show_line_subtotals_tax_included"/>

                                <tr t-att-class="'bg-200 font-weight-bold o_line_section' if line.display_type == 'line_section' else 'font-italic o_line_note' if line.display_type == 'line_note' else ''">
                                    <t t-if="not line.display_type">
                                        <td id="product_name"><span t-field="line.name"/></td>
                                        <td class="text-right">
                                            <div id="quote_qty">
                                                <span t-field="line.product_uom_qty"/>
                                                <span t-field="line.product_uom"/>
                                            </div>
                                        </td>
                                        <td t-attf-class="text-right {{ 'd-none d-sm-table-cell' if report_type == 'html' else '' }}">
                                            <div
                                                t-if="line.discount &gt;= 0"
                                                t-field="line.price_unit"
                                                t-att-style="line.discount and 'text-decoration: line-through' or None"
                                                t-att-class="(line.discount and 'text-danger' or '') + ' text-right'"
                                            />
                                            <div t-if="line.discount">
                                                <t t-esc="(1-line.discount / 100.0) * line.price_unit" t-options='{"widget": "float", "decimal_precision": "Product Price"}'/>
                                            </div>
                                        </td>
                                        <td t-if="display_discount" t-attf-class="text-right {{ 'd-none d-sm-table-cell' if report_type == 'html' else '' }}">
                                            <strong t-if="line.discount &gt; 0" class="text-info">
                                                <t t-esc="((line.discount % 1) and '%s' or '%d') % line.discount"/>%
                                            </strong>
                                        </td>
                                        <td t-attf-class="text-right {{ 'd-none d-md-table-cell' if report_type == 'html' else '' }}">
                                            <span t-esc="', '.join(map(lambda x: (x.description or x.name), line.tax_id))"/>
                                        </td>
                                        <td class="text-right">
                                            <span class="oe_order_line_price_subtotal" t-field="line.price_subtotal" groups="account.group_show_line_subtotals_tax_excluded"/>
                                            <span class="oe_order_line_price_total" t-field="line.price_total" groups="account.group_show_line_subtotals_tax_included"/>
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

                                <t t-if="current_section and (line_last or sale_order.order_line[line_index+1].display_type == 'line_section')">
                                    <tr class="is-subtotal text-right">
                                        <td colspan="99">
                                            <strong class="mr16">Subtotal</strong>
                                            <span
                                                t-esc="current_subtotal"
                                                t-options='{"widget": "monetary", "display_currency": sale_order.pricelist_id.currency_id}'
                                            />
                                        </td>
                                    </tr>
                                </t>
                            </t>
                        </tbody>
                    </table>
                </div>

                <div id="total" class="row" name="total" style="page-break-inside: avoid;">
                    <div t-attf-class="#{'col-6' if report_type != 'html' else 'col-sm-7 col-md-6'} ml-auto">
                        <t t-call="sale.sale_order_portal_content_totals_table"/>
                    </div>
                </div>
            </section>

            <section t-if="sale_order.signature" id="signature" name="Signature">
                <div class="row mt-4" name="signature">
                    <div t-attf-class="#{'col-3' if report_type != 'html' else 'col-sm-7 col-md-4'} ml-auto text-center">
                        <h5>Signature</h5>
                        <img t-att-src="image_data_uri(sale_order.signature)" style="max-height: 6rem; max-width: 100%;"/>
                        <p t-field="sale_order.signed_by"/>
                    </div>
                </div>
            </section>

            <section id="terms" class="mt-5" t-if="not is_html_empty(sale_order.note)">
                <h3 class="">Terms &amp; Conditions</h3>
                <hr class="mt-0 mb-1"/>
                <t t-if="sale_order.terms_type == 'html'">
                    <!-- Note is plain text. This ensures a clickable link  -->
                    <t t-set="tc_url" t-value="'%s/terms' % (sale_order.get_base_url())"/>
                    <em>Terms &amp; Conditions: <a href="/terms"><t t-esc="tc_url"/></a></em>
                </t>
                <t t-else="">
                    <em t-field="sale_order.note"/>
                </t>
            </section>

            <section class="mt-5" t-if="sale_order.payment_term_id">
                <h3 class="">Payment terms</h3>
                <hr class="mt-0 mb-1"/>
                <span t-field="sale_order.payment_term_id.note"/>
            </section>
        </div>
    </template>

    <template id="sale_order_portal_content_totals_table">
        <table class="table table-sm">
            <t t-set="tax_totals" t-value="json.loads(sale_order.tax_totals_json)"/>
            <t t-call="account.document_tax_totals"/>
        </table>
    </template>
</odoo>

```

## File: views\sale_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

        <!-- Top menu item -->
        <menuitem id="sale_menu_root"
            name="Sales"
            web_icon="sale_management,static/description/icon.png"
            active="False"
            sequence="30"/>

        <menuitem id="sale_order_menu"
            name="Orders"
            parent="sale_menu_root"
            sequence="2"/>

        <menuitem id="report_sales_team"
            name="Sales Teams"
            parent="sale_order_menu"
            groups="sales_team.group_sale_manager"
            action="sales_team.crm_team_action_sales"
            sequence="3"/>

        <menuitem id="res_partner_menu"
            parent="sale_order_menu"
            action="account.res_partner_action_customer"
            sequence="4" groups="sales_team.group_sale_salesman"/>

        <menuitem id="menu_sale_report"
            name="Reporting"
            parent="sale_menu_root"
            sequence="5"
            groups="sales_team.group_sale_manager"/>

        <menuitem id="menu_report_product_all"
            name="Sales"
            action="action_order_report_all"
            parent="menu_sale_report"
            sequence="1"/>

        <menuitem id="menu_sale_config"
            name="Configuration"
            parent="sale_menu_root"
            sequence="35"
            groups="sales_team.group_sale_manager"/>

        <menuitem id="sales_team_config"
            name="Sales Teams"
            parent="sale.menu_sale_config"
            action="sales_team.crm_team_action_config"
            sequence="2"/>

        <menuitem id="menu_sales_config"
            parent="menu_sale_config"
            sequence="4"
            name="Sales Orders"/>

        <menuitem
            id="menu_tag_config"
            name="Tags"
            parent="sale.menu_sales_config"
            action="sales_team.sales_team_crm_tag_action"
            sequence="2"/>

        <record id="product_template_action" model="ir.actions.act_window">
            <field name="name">Products</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">product.template</field>
            <field name="view_mode">kanban,tree,form,activity</field>
            <field name="view_id" ref="product.product_template_kanban_view"/>
            <field name="search_view_id" ref="product.product_template_search_view"/>
            <field name="context">{"search_default_filter_to_sell":1, "sale_multi_pricelist_product_template": 1}</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Create a new product
                </p><p>
                    You must define a product for everything you sell or purchase,
                    whether it's a storable product, a consumable or a service.
                </p>
            </field>
        </record>

        <record id="product_template_form_view" model="ir.ui.view">
            <field name="name">product.template.product.website.form</field>
            <field name="model">product.template</field>
            <field name="inherit_id" ref="product.product_template_form_view"/>
            <field name="arch" type="xml">
                <xpath expr="//page[@name='sales']" position="attributes">
                    <attribute name="invisible">0</attribute>
                </xpath>
            </field>
        </record>

        <menuitem id="product_menu_catalog" name="Products" parent="sale_menu_root" sequence="4" groups="sales_team.group_sale_salesman"/>
        <menuitem id="menu_product" name="Product Variants" parent="product_menu_catalog" sequence="2" groups="base.group_no_one" active="False"/>
        <menuitem action="product_template_action" id="menu_product_template_action" parent="product_menu_catalog" sequence="1" active="False"/>
        <menuitem id="prod_config_main" name="Products" parent="menu_sale_config" sequence="5"/>
        <menuitem id="menu_products" action="product.product_normal_action_sell" parent="product_menu_catalog" groups="product.group_product_variant" sequence="2" active="False"/>
        <menuitem id="next_id_16" name="Units of Measure" parent="menu_sale_config" sequence="6" groups="uom.group_uom" active="False"/>
        <menuitem action="uom.product_uom_form_action" id="menu_product_uom_form_action" parent="next_id_16" sequence="7" groups="base.group_no_one" active="False"/>
        <menuitem action="uom.product_uom_categ_form_action" id="menu_product_uom_categ_form_action" parent="next_id_16" sequence="8" active="False"/>
        <menuitem id="menu_product_pricelist_main" name="Pricelists" parent="product_menu_catalog" action="product.product_pricelist_action2" groups="product.group_product_pricelist" sequence="3" active="False"/>

        <record id="sale_order_view_activity" model="ir.ui.view">
            <field name="name">sale.order.activity</field>
            <field name="model">sale.order</field>
            <field name="arch" type="xml">
                <activity string="Sales Orders">
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

        <record id="view_sale_order_calendar" model="ir.ui.view">
            <field name="name">sale.order.calendar</field>
            <field name="model">sale.order</field>
            <field name="arch" type="xml">
                <calendar string="Sales Orders" date_start="date_order" color="state" hide_time="true" event_limit="5" quick_add="False">
                    <field name="currency_id" invisible="1"/>
                    <field name="partner_id" avatar_field="avatar_128"/>
                    <field name="amount_total" widget="monetary"/>
                    <field name="payment_term_id"/>
                    <field name="state" filters="1" invisible="1"/>
                </calendar>
            </field>
        </record>
        <record model="ir.ui.view" id="view_sale_order_graph">
            <field name="name">sale.order.graph</field>
            <field name="model">sale.order</field>
            <field name="arch" type="xml">
                <graph string="Sales Orders" sample="1">
                    <field name="partner_id"/>
                    <field name="amount_total" type="measure"/>
                </graph>
            </field>
        </record>
        <record model="ir.ui.view" id="view_sale_order_pivot">
            <field name="name">sale.order.pivot</field>
            <field name="model">sale.order</field>
            <field name="arch" type="xml">
                <pivot string="Sales Orders" sample="1">
                    <field name="date_order" type="row"/>
                    <field name="amount_total" type="measure"/>
                </pivot>
            </field>
        </record>

        <!-- Sales Orders Kanban View  -->
        <record model="ir.ui.view" id="view_sale_order_kanban">
            <field name="name">sale.order.kanban</field>
            <field name="model">sale.order</field>
            <field name="arch" type="xml">
                <kanban class="o_kanban_mobile" sample="1" quick_create="false">
                    <field name="name"/>
                    <field name="partner_id"/>
                    <field name="amount_total"/>
                    <field name="date_order"/>
                    <field name="state"/>
                    <field name="currency_id"/>
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
                                    <div class="oe_kanban_bottom_left text-muted">
                                        <span><t t-esc="record.name.value"/> <t t-esc="record.date_order.value"/></span>
                                        <field name="activity_ids" widget="kanban_activity"/>
                                    </div>
                                    <div class="oe_kanban_bottom_right">
                                        <field name="state" widget="label_selection" options="{'classes': {'draft': 'default', 'cancel': 'default', 'done': 'success'}}"/>
                                    </div>
                                </div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="view_order_tree" model="ir.ui.view">
            <field name="name">sale.order.tree</field>
            <field name="model">sale.order</field>
            <field name="priority">2</field>
            <field name="arch" type="xml">
                <tree string="Sales Orders" sample="1" decoration-info="invoice_status == 'to invoice'" decoration-muted="state == 'cancel'">
                    <field name="message_needaction" invisible="1"/>
                    <field name="name" string="Number" readonly="1" decoration-bf="1"/>
                    <field name="date_order" string="Order Date" widget="date" optional="show"/>
                    <field name="commitment_date" optional="hide"/>
                    <field name="expected_date" optional="hide"/>
                    <field name="partner_id" readonly="1"/>
                    <field name="user_id" optional="show" widget="many2one_avatar_user"/>
                    <field name="activity_ids" widget="list_activity" optional="show"/>
                    <field name="team_id" optional="hide"/>
                    <field name="company_id" groups="base.group_multi_company" optional="show" readonly="1"/>
                    <field name="amount_untaxed" sum="Total Tax Excluded" widget="monetary" optional="hide"/>
                    <field name="amount_tax" sum="Tax Total" widget="monetary" optional="hide"/>
                    <field name="amount_total" sum="Total Tax Included" widget="monetary" decoration-bf="1" optional="show"/>
                    <field name="currency_id" invisible="1"/>
                    <field name="invoice_status" decoration-success="invoice_status == 'invoiced'" decoration-info="invoice_status == 'to invoice'" decoration-warning="invoice_status == 'upselling'" widget="badge" optional="show"/>
                    <field name="tag_ids" optional="hide" widget="many2many_tags" options="{'color_field': 'color'}"/>
                    <field name="state" invisible="1"/>
                </tree>
            </field>
        </record>

        <record id="view_quotation_tree" model="ir.ui.view">
            <field name="name">sale.order.tree</field>
            <field name="model">sale.order</field>
            <field name="priority">4</field>
            <field name="arch" type="xml">
                <tree string="Quotation" class="o_sale_order" sample="1" decoration-info="state in ['draft', 'sent']" decoration-muted="state == 'cancel'">
                    <field name="name" string="Number" readonly="1" decoration-bf="1"/>
                    <field name="create_date" string="Creation Date" widget="date" optional="show"/>
                    <field name="commitment_date" widget="date" optional="hide"/>
                    <field name="expected_date" widget="date" optional="hide"/>
                    <field name="partner_id" readonly="1"/>
                    <field name="user_id" widget="many2one_avatar_user" optional="show"/>
                    <field name="activity_ids" widget="list_activity" optional="show"/>
                    <field name="team_id" optional="hide"/>
                    <field name="tag_ids" optional="hide" widget="many2many_tags" options="{'color_field': 'color'}"/>
                    <field name="company_id" groups="base.group_multi_company" optional="show" readonly="1"/>
                    <field name="amount_untaxed" sum="Total Tax Excluded" widget="monetary" optional="hide"/>
                    <field name="amount_tax" sum="Tax Total" widget="monetary" optional="hide"/>
                    <field name="amount_total" sum="Total Tax Included" widget="monetary" decoration-bf="1" optional="show"/>
                    <field name="state" decoration-success="state == 'sale' or state == 'done'" decoration-info="state == 'draft' or state == 'sent'" widget="badge" optional="show"/>
                    <field name="invoice_status" optional="hide"/>
                    <field name="message_needaction" invisible="1"/>
                    <field name="currency_id" invisible="1"/>
                </tree>
            </field>
        </record>
        <record id="view_quotation_tree_with_onboarding" model="ir.ui.view">
            <field name="name">sale.order.tree</field>
            <field name="model">sale.order</field>
            <field name="inherit_id" ref="view_quotation_tree"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <xpath expr="//tree" position="attributes">
                    <attribute name="banner_route">/sales/sale_quotation_onboarding_panel</attribute>
                </xpath>
            </field>
        </record>

        <record id="view_order_form" model="ir.ui.view">
            <field name="name">sale.order.form</field>
            <field name="model">sale.order</field>
            <field name="arch" type="xml">
              <form string="Sales Order" class="o_sale_order" js_class="sale_discount_form">
                <header>
                    <field name="authorized_transaction_ids" invisible="1"/>
                    <button name="payment_action_capture" type="object" data-hotkey="shift+g"
                            string="Capture Transaction" class="oe_highlight"
                            attrs="{'invisible': [('authorized_transaction_ids', '=', [])]}"/>
                    <button name="payment_action_void" type="object"
                            string="Void Transaction" data-hotkey="shift+v"
                            confirm="Are you sure you want to void the authorized transaction? This action can't be undone."
                            attrs="{'invisible': [('authorized_transaction_ids', '=', [])]}"/>
                    <button name="%(sale.action_view_sale_advance_payment_inv)d" string="Create Invoice"
                        type="action" class="btn-primary" data-hotkey="q"
                        attrs="{'invisible': [('invoice_status', '!=', 'to invoice')]}"/>
                    <button name="%(sale.action_view_sale_advance_payment_inv)d" string="Create Invoice"
                        type="action" context="{'default_advance_payment_method': 'percentage'}" data-hotkey="q"
                        attrs="{'invisible': ['|',('invoice_status', '!=', 'no'), ('state', '!=', 'sale')]}"/>
                    <button name="action_quotation_send" string="Send by Email" type="object" states="draft" class="btn-primary" data-hotkey="g"/>
                    <button name="action_quotation_send" type="object" string="Send PRO-FORMA Invoice"
                      groups="sale.group_proforma_sales" class="btn-primary"
                      attrs="{'invisible': ['|', ('state', '!=', 'draft'), ('invoice_count','&gt;=',1)]}" context="{'proforma': True}"/>
                    <button name="action_confirm" id="action_confirm" data-hotkey="v"
                        string="Confirm" class="btn-primary" type="object"
                        attrs="{'invisible': [('state', 'not in', ['sent'])]}"/>
                    <button name="action_confirm" data-hotkey="v"
                        string="Confirm" type="object"
                        attrs="{'invisible': [('state', 'not in', ['draft'])]}"/>
                    <button name="action_quotation_send" type="object" string="Send PRO-FORMA Invoice" groups="sale.group_proforma_sales" attrs="{'invisible': ['|', ('state', '=', 'draft'), ('invoice_count','&gt;=',1)]}" context="{'proforma': True}"/>
                    <button name="action_quotation_send" string="Send by Email" type="object" states="sent,sale" data-hotkey="g"/>
                    <button name="action_cancel" type="object" string="Cancel" attrs="{'invisible': ['|', ('state', 'not in', ['draft', 'sent','sale']), ('id', '=', False)]}" data-hotkey="z"/>
                    <button name="action_draft" states="cancel" type="object" string="Set to Quotation" data-hotkey="w"/>
                    <field name="state" widget="statusbar" statusbar_visible="draft,sent,sale"/>
                </header>
                <sheet>
                    <div class="oe_button_box" name="button_box">
                        <button name="preview_sale_order"
                            type="object"
                            class="oe_stat_button"
                            icon="fa-globe icon">
                            <div class="o_field_widget o_stat_info">
                                <span class="o_stat_text">Customer</span>
                                <span class="o_stat_text">Preview</span>
                            </div>
                        </button>
                        <button name="action_view_invoice"
                            type="object"
                            class="oe_stat_button"
                            icon="fa-pencil-square-o"
                            attrs="{'invisible': [('invoice_count', '=', 0)]}">
                            <field name="invoice_count" widget="statinfo" string="Invoices"/>
                        </button>
                    </div>
                    <div class="oe_title">
                        <h1>
                            <field name="name" readonly="1"/>
                        </h1>
                    </div>
                    <group name="sale_header">
                        <group name="partner_details">
                            <field name="partner_id" widget="res_partner_many2one" context="{'res_partner_search_mode': 'customer', 'show_address': 1, 'show_vat': True}" options='{"always_reload": True}'/>
                            <field name="partner_invoice_id" groups="sale.group_delivery_invoice_address" context="{'default_type':'invoice'}" options='{"always_reload": True}'/>
                            <field name="partner_shipping_id" groups="sale.group_delivery_invoice_address" context="{'default_type':'delivery'}" options='{"always_reload": True}'/>
                        </group>
                        <group name="order_details">
                            <field name="validity_date" attrs="{'invisible': [('state', 'in', ['sale', 'done'])]}"/>
                            <div class="o_td_label" groups="base.group_no_one" attrs="{'invisible': [('state', 'in', ['sale', 'done', 'cancel'])]}">
                                <label for="date_order" string="Quotation Date"/>
                            </div>
                            <field name="date_order" nolabel="1" groups="base.group_no_one" attrs="{'invisible': [('state', 'in', ['sale', 'done', 'cancel'])]}"/>
                            <div class="o_td_label" attrs="{'invisible': [('state', 'in', ['draft', 'sent'])]}">
                                <label for="date_order" string="Order Date"/>
                            </div>
                            <field name="date_order" attrs="{'required': [('state', 'in', ['sale', 'done'])], 'invisible': [('state', 'in', ['draft', 'sent'])]}" nolabel="1"/>
                            <field name="show_update_pricelist" invisible="1"/>
                            <label for="pricelist_id" groups="product.group_product_pricelist"/>
                            <div groups="product.group_product_pricelist" class="o_row">
                                <field name="pricelist_id" options="{'no_open':True,'no_create': True}"/>
                                <button name="update_prices" type="object"
                                    string=" Update Prices"
                                    help="Recompute all prices based on this pricelist"
                                    class="btn-link mb-1 px-0" icon="fa-refresh"
                                    confirm="This will update all unit prices based on the currently set pricelist."
                                    attrs="{'invisible': ['|', ('show_update_pricelist', '=', False), ('state', 'in', ['sale', 'done','cancel'])]}"/>
                            </div>
                            <field name="currency_id" invisible="1"/>
                            <field name="tax_country_id" invisible="1"/>
                            <field name="payment_term_id" options="{'no_open':True,'no_create': True}"/>
                        </group>
                    </group>
                    <notebook>
                        <page string="Order Lines" name="order_lines">
                            <field
                                name="order_line"
                                widget="section_and_note_one2many"
                                mode="tree,kanban"
                                attrs="{'readonly': [('state', 'in', ('done','cancel'))]}"
                            >
                                <form>
                                    <field name="display_type" invisible="1"/>
                                    <!--
                                        We need the sequence field to be here for new lines to be added at the correct position.
                                        TODO: at some point we want to fix this in the framework so that an invisible field is not required.
                                    -->
                                    <field name="sequence" invisible="1"/>
                                    <field name="product_uom_category_id" invisible="1"/>
                                    <group>
                                        <group attrs="{'invisible': [('display_type', '!=', False)]}">
                                            <field name="product_updatable" invisible="1"/>
                                            <field name="product_id"
                                                domain="[('sale_ok', '=', True), '|', ('company_id', '=', False), ('company_id', '=', parent.company_id)]"
                                                context="{'partner_id':parent.partner_id, 'quantity':product_uom_qty, 'pricelist':parent.pricelist_id, 'uom':product_uom, 'company_id': parent.company_id}"
                                                attrs="{
                                                    'readonly': [('product_updatable', '=', False)],
                                                    'required': [('display_type', '=', False)],
                                                }"
                                                force_save="1"
                                                widget="many2one_barcode"
                                               />
                                            <field name="invoice_status" invisible="1"/>
                                            <field name="qty_to_invoice" invisible="1"/>
                                            <field name="qty_delivered_manual" invisible="1"/>
                                            <field name="qty_delivered_method" invisible="1"/>
                                            <field name="price_total" invisible="1"/>
                                            <field name="price_tax" invisible="1"/>
                                            <field name="price_subtotal" invisible="1"/>
                                            <field name="product_uom_readonly" invisible="1"/>
                                            <label for="product_uom_qty"/>
                                            <div class="o_row" name="ordered_qty">
                                                <field
                                                    context="{'partner_id':parent.partner_id, 'quantity':product_uom_qty, 'pricelist':parent.pricelist_id, 'uom':product_uom, 'uom_qty_change':True, 'company_id': parent.company_id}"
                                                    name="product_uom_qty"/>
                                                <field
                                                    name="product_uom"
                                                    force_save="1"
                                                    groups="uom.group_uom"
                                                    class="oe_no_button"
                                                    attrs="{
                                                        'readonly': [('product_uom_readonly', '=', True)],
                                                        'required': [('display_type', '=', False)],
                                                    }"
                                                />
                                            </div>
                                            <label for="qty_delivered" string="Delivered" attrs="{'invisible': [('parent.state', 'not in', ['sale', 'done'])]}"/>
                                            <div name="delivered_qty" attrs="{'invisible': [('parent.state', 'not in', ['sale', 'done'])]}">
                                                <field name="qty_delivered" attrs="{'readonly': [('qty_delivered_method', '!=', 'manual')]}"/>
                                            </div>
                                            <label for="qty_invoiced" string="Invoiced" attrs="{'invisible': [('parent.state', 'not in', ['sale', 'done'])]}"/>
                                            <div name="invoiced_qty" attrs="{'invisible': [('parent.state', 'not in', ['sale', 'done'])]}">
                                                <field name="qty_invoiced" attrs="{'invisible': [('parent.state', 'not in', ['sale', 'done'])]}"/>
                                            </div>
                                            <field name="product_packaging_qty" attrs="{'invisible': ['|', ('product_id', '=', False), ('product_packaging_id', '=', False)]}" groups="product.group_stock_packaging"/>
                                            <field name="product_packaging_id" attrs="{'invisible': [('product_id', '=', False)]}" context="{'default_product_id': product_id, 'tree_view_ref':'product.product_packaging_tree_view', 'form_view_ref':'product.product_packaging_form_view'}" groups="product.group_stock_packaging" />
                                            <field name="price_unit"/>
                                            <field name="tax_id" widget="many2many_tags" options="{'no_create': True}" context="{'search_view_ref': 'account.account_tax_view_search'}" domain="[('type_tax_use','=','sale'), ('company_id','=',parent.company_id), ('country_id', '=', parent.tax_country_id)]"
                                                attrs="{'readonly': [('qty_invoiced', '&gt;', 0)]}"/>
                                            <label for="discount" groups="product.group_discount_per_so_line"/>
                                            <div name="discount" groups="product.group_discount_per_so_line">
                                                <field name="discount" class="oe_inline"/> %%
                                            </div>
                                            <!--
                                                We need the sequence field to be here
                                                because we want to be able to overwrite the default sequence value in the JS
                                                in order for new lines to be added at the correct position.
                                                NOTE: at some point we want to fix this in the framework so that an invisible field is not required.
                                            -->
                                            <field name="sequence" invisible="1"/>
                                        </group>
                                        <group attrs="{'invisible': [('display_type', '!=', False)]}">
                                            <label for="customer_lead"/>
                                            <div name="lead">
                                                <field name="customer_lead" class="oe_inline"/> days
                                            </div>
                                            <field name="analytic_tag_ids" widget="many2many_tags" groups="analytic.group_analytic_tags" options="{'color_field': 'color'}" domain="['|', ('company_id', '=', False), ('company_id', '=', parent.company_id)]"/>
                                        </group>
                                    </group>
                                    <label for="name" string="Description" attrs="{'invisible': [('display_type', '!=', False)]}"/>
                                    <label for="name" string="Section Name (eg. Products, Services)" attrs="{'invisible': [('display_type', '!=', 'line_section')]}"/>
                                    <label for="name" string="Note" attrs="{'invisible': [('display_type', '!=', 'line_note')]}"/>
                                    <field name="name"/>
                                    <div name="invoice_lines" groups="base.group_no_one" attrs="{'invisible': [('display_type', '!=', False)]}">
                                        <label for="invoice_lines"/>
                                        <field name="invoice_lines"/>
                                    </div>
                                    <field name="state" invisible="1"/>
                                    <field name="company_id" invisible="1"/>
                                </form>
                                <tree
                                    string="Sales Order Lines"
                                    editable="bottom"
                                >
                                    <control>
                                        <create name="add_product_control" string="Add a product"/>
                                        <create name="add_section_control" string="Add a section" context="{'default_display_type': 'line_section'}"/>
                                        <create name="add_note_control" string="Add a note" context="{'default_display_type': 'line_note'}"/>
                                    </control>

                                    <field name="sequence" widget="handle" />
                                    <!-- We do not display the type because we don't want the user to be bothered with that information if he has no section or note. -->
                                    <field name="display_type" invisible="1"/>
                                    <field name="product_uom_category_id" invisible="1"/>

                                    <field name="product_updatable" invisible="1"/>
                                    <field
                                        name="product_id"
                                        attrs="{
                                            'readonly': [('product_updatable', '=', False)],
                                            'required': [('display_type', '=', False)],
                                        }"
                                        force_save="1"
                                        context="{
                                            'partner_id': parent.partner_id,
                                            'quantity': product_uom_qty,
                                            'pricelist': parent.pricelist_id,
                                            'uom':product_uom,
                                            'company_id': parent.company_id,
                                            'default_lst_price': price_unit,
                                            'default_description_sale': name
                                        }"
                                        domain="[('sale_ok', '=', True), '|', ('company_id', '=', False), ('company_id', '=', parent.company_id)]"
                                        widget="product_configurator"
                                    />
                                    <field name="product_template_id"
                                      string="Product"
                                      invisible="1"
                                      attrs="{
                                          'readonly': [('product_updatable', '=', False)],
                                          'required': [('display_type', '=', False)],
                                      }"
                                      context="{
                                          'partner_id': parent.partner_id,
                                          'quantity': product_uom_qty,
                                          'pricelist': parent.pricelist_id,
                                          'uom':product_uom,
                                          'company_id': parent.company_id,
                                          'default_list_price': price_unit,
                                          'default_description_sale': name
                                      }"
                                      domain="[('sale_ok', '=', True), '|', ('company_id', '=', False), ('company_id', '=', parent.company_id)]"
                                      widget="product_configurator"/>
                                    <field name="name" widget="section_and_note_text" optional="show"/>
                                    <field
                                        name="analytic_tag_ids"
                                        optional="hide"
                                        groups="analytic.group_analytic_tags"
                                        widget="many2many_tags"
                                        options="{'color_field': 'color'}"
                                        domain="['|', ('company_id', '=', False), ('company_id', '=', parent.company_id)]"
                                    />
                                    <field
                                        name="product_uom_qty"
                                        decoration-info="(not display_type and invoice_status == 'to invoice')" decoration-bf="(not display_type and invoice_status == 'to invoice')"
                                        context="{
                                            'partner_id': parent.partner_id,
                                            'quantity': product_uom_qty,
                                            'pricelist': parent.pricelist_id,
                                            'uom': product_uom,
                                            'company_id': parent.company_id
                                        }"
                                    />
                                    <field
                                        name="qty_delivered"
                                        decoration-info="(not display_type and invoice_status == 'to invoice')" decoration-bf="(not display_type and invoice_status == 'to invoice')"
                                        string="Delivered"
                                        attrs="{
                                            'column_invisible': [('parent.state', 'not in', ['sale', 'done'])],
                                            'readonly': [('qty_delivered_method', '!=', 'manual')]
                                        }"
                                        optional="show"
                                    />
                                    <field name="qty_delivered_manual" invisible="1"/>
                                    <field name="qty_delivered_method" invisible="1"/>
                                    <field
                                        name="qty_invoiced"
                                        decoration-info="(not display_type and invoice_status == 'to invoice')" decoration-bf="(not display_type and invoice_status == 'to invoice')"
                                        string="Invoiced"
                                        attrs="{'column_invisible': [('parent.state', 'not in', ['sale', 'done'])]}"
                                        optional="show"
                                    />
                                    <field name="qty_to_invoice" invisible="1"/>
                                    <field name="product_uom_readonly" invisible="1"/>
                                    <field
                                        name="product_uom"
                                        force_save="1"
                                        string="UoM"
                                        attrs="{
                                            'readonly': [('product_uom_readonly', '=', True)],
                                            'required': [('display_type', '=', False)],
                                        }"
                                        context="{'company_id': parent.company_id}"
                                        groups="uom.group_uom"
                                        options='{"no_open": True}'
                                        optional="show"
                                    />
                                    <field
                                        name="customer_lead"
                                        optional="hide"
                                        attrs="{'readonly': [('parent.state', 'not in', ['draft', 'sent', 'sale'])]}"
                                    />
                                    <field name="product_packaging_qty" attrs="{'invisible': ['|', ('product_id', '=', False), ('product_packaging_id', '=', False)]}" groups="product.group_stock_packaging" optional="show"/>
                                    <field name="product_packaging_id" attrs="{'invisible': [('product_id', '=', False)]}" context="{'default_product_id': product_id, 'tree_view_ref':'product.product_packaging_tree_view', 'form_view_ref':'product.product_packaging_form_view'}" groups="product.group_stock_packaging" optional="show"/>
                                    <field
                                        name="price_unit"
                                        attrs="{'readonly': [('qty_invoiced', '&gt;', 0)]}"
                                    />
                                    <field
                                        name="tax_id"
                                        widget="many2many_tags"
                                        options="{'no_create': True}"
                                        domain="[('type_tax_use','=','sale'),('company_id','=',parent.company_id), ('country_id', '=', parent.tax_country_id)]"
                                        context="{'active_test': True}"
                                        attrs="{'readonly': [('qty_invoiced', '&gt;', 0)]}"
                                        optional="show"
                                    />
                                    <field name="discount" string="Disc.%" groups="product.group_discount_per_so_line" optional="show" widget="product_discount"/>
                                    <field name="price_subtotal" widget="monetary" groups="account.group_show_line_subtotals_tax_excluded"/>
                                    <field name="price_total" widget="monetary" groups="account.group_show_line_subtotals_tax_included"/>
                                    <field name="state" invisible="1"/>
                                    <field name="invoice_status" invisible="1"/>
                                    <field name="currency_id" invisible="1"/>
                                    <field name="price_tax" invisible="1"/>
                                    <field name="company_id" invisible="1"/>
                                </tree>
                                <kanban class="o_kanban_mobile">
                                    <field name="name"/>
                                    <field name="product_id"/>
                                    <field name="product_uom_qty"/>
                                    <field name="product_uom" groups="uom.group_uom"/>
                                    <field name="price_subtotal"/>
                                    <field name="price_total"/>
                                    <field name="price_tax" invisible="1"/>
                                    <field name="price_total" invisible="1"/>
                                    <field name="price_unit"/>
                                    <field name="display_type"/>
                                    <field name="tax_id" invisible="1"/>
                                    <field name="company_id" invisible="1"/>
                                    <templates>
                                        <t t-name="kanban-box">
                                            <div t-attf-class="oe_kanban_card oe_kanban_global_click pl-0 pr-0 {{ record.display_type.raw_value ? 'o_is_' + record.display_type.raw_value : '' }}">
                                                <t t-if="!record.display_type.raw_value">
                                                    <div class="row no-gutters">
                                                        <div class="col-2 pr-3">
                                                            <img t-att-src="kanban_image('product.product', 'image_128', record.product_id.raw_value)" t-att-title="record.product_id.value" t-att-alt="record.product_id.value" style="max-width: 100%;"/>
                                                        </div>
                                                        <div class="col-10">
                                                            <div class="row">
                                                                <div class="col">
                                                                    <strong t-esc="record.product_id.value" />
                                                                </div>
                                                                <div class="col-auto">
                                                                    <t t-set="line_price" t-value="record.price_subtotal.value" groups="account.group_show_line_subtotals_tax_excluded"/>
                                                                    <t t-set="line_price" t-value="record.price_total.value" groups="account.group_show_line_subtotals_tax_included"/>
                                                                    <strong class="float-right text-right" t-esc="line_price" />
                                                                </div>
                                                            </div>
                                                            <div class="row">
                                                                <div class="col-12 text-muted">
                                                                    Quantity:
                                                                    <t t-esc="record.product_uom_qty.value"/>
                                                                    <t t-esc="record.product_uom.value"/>
                                                                </div>
                                                            </div>
                                                            <div class="row">
                                                                <div class="col-12 text-muted">
                                                                    Unit Price:
                                                                    <t t-esc="record.price_unit.value"/>
                                                                </div>
                                                            </div>
                                                        </div>
                                                    </div>
                                                </t>
                                                <t t-if="record.display_type.raw_value === 'line_section' || record.display_type.raw_value === 'line_note'">
                                                    <div class="row">
                                                        <div class="col-12">
                                                            <t t-esc="record.name.value"/>
                                                        </div>
                                                    </div>
                                                </t>
                                            </div>
                                        </t>
                                    </templates>
                                </kanban>
                            </field>
                            <group name="note_group" col="6" class="mt-2 mt-md-0">
                                <group colspan="4">
                                    <field name="note" class="oe-bordered-editor" nolabel="1" placeholder="Terms and conditions..."/>
                                </group>
                                <group class="oe_subtotal_footer oe_right" colspan="2" name="sale_total">
                                    <field name="tax_totals_json" widget="account-tax-totals-field" nolabel="1" colspan="2"/>
                                </group>
                                <div class="oe_clear"/>
                            </group>
                        </page>
                        <page string="Other Info" name="other_information">
                            <group>
                                <group name="sales_person" string="Sales">
                                    <field name="user_id" widget="many2one_avatar_user"/>
                                    <field name="team_id" kanban_view_ref="%(sales_team.crm_team_view_kanban)s" options="{'no_create': True}"/>
                                    <field name="company_id" options="{'no_create': True}" groups="base.group_multi_company"/>
                                    <field name="require_signature"/>
                                    <field name="require_payment"/>
                                    <field name="reference" readonly="1" attrs="{'invisible': [('reference', '=', False)]}"/>
                                    <field name="client_order_ref"/>
                                    <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color', 'no_create_edit': True}"/>
                                </group>
                                <group name="sale_info" string="Invoicing">
                                    <field name="fiscal_position_id" options="{'no_create': True}"/>
                                    <field name="analytic_account_id" context="{'default_partner_id':partner_invoice_id, 'default_name':name}" attrs="{'readonly': [('invoice_count','!=',0),('state','=','sale')]}" groups="analytic.group_analytic_accounting" force_save="1"/>
                                    <field name="invoice_status" states="sale,done" groups="base.group_no_one"/>
                                </group>
                            </group>
                            <group>
                                <group name="sale_shipping">
                                    <label for="commitment_date" string="Delivery Date"/>
                                    <div name="commitment_date_div" class="o_row">
                                        <field name="commitment_date"/>
                                        <span name="expected_date_span" class="text-muted">Expected: <field name="expected_date" widget="date"/></span>
                                    </div>
                                </group>
                                <group string="Tracking" name="sale_reporting">
                                    <group name="technical" colspan="2" class="mb-0">
                                        <field name="origin"/>
                                    </group>
                                    <group name="utm_link" colspan="2" class="mt-0">
                                        <field name="campaign_id"/>
                                        <field name="medium_id"/>
                                        <field name="source_id"/>
                                    </group>
                                </group>
                            </group>
                        </page>
                        <page groups="base.group_no_one" string="Customer Signature" name="customer_signature" attrs="{'invisible': [('require_signature', '=', False), ('signed_by', '=', False), ('signature', '=', False), ('signed_on', '=', False)]}">
                            <group>
                                <field name="signed_by"/>
                                <field name="signed_on"/>
                                <field name="signature" widget="image"/>
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

        <record id="view_sales_order_auto_done_setting" model="ir.ui.view">
            <field name="name">sale.order.form</field>
            <field name="model">sale.order</field>
            <field name="inherit_id" ref="sale.view_order_form"/>
            <field name="groups_id" eval="[(4, ref('sale.group_auto_done_setting'))]"/>
            <field name="arch" type="xml">
                <xpath expr="//form//header//button[@name='action_draft']" position="after">
                   <button name="action_done" type="object" string="Lock" states="sale"
                        help="If the sale is locked, you can not modify it anymore. However, you will still be able to invoice or deliver." groups="sales_team.group_sale_manager"/>
                    <button name="action_unlock" type="object" string="Unlock" states="done" groups="sales_team.group_sale_manager"/>
                </xpath>
            </field>
        </record>

        <record id="view_sales_order_filter" model="ir.ui.view">
            <field name="name">sale.order.list.select</field>
            <field name="model">sale.order</field>
            <field name="priority" eval="15"/>
            <field name="arch" type="xml">
                <search string="Search Sales Order">
                    <field name="name" string="Order" filter_domain="['|', '|', ('name', 'ilike', self), ('client_order_ref', 'ilike', self), ('partner_id', 'child_of', self)]"/>
                    <field name="partner_id" operator="child_of"/>
                    <field name="user_id"/>
                    <field name="team_id" string="Sales Team"/>
                    <field name="order_line" string="Product" filter_domain="[('order_line.product_id', 'ilike', self)]"/>
                    <field name="analytic_account_id" groups="analytic.group_analytic_accounting"/>
                    <!-- We only allow to search on the following sale order line fields (product, name) because the other fields, such as price, quantity, ...
                        will not be searched as often, and if they need to be searched it's usually in the context of products
                        and then they can be searched from the page listing the sale order lines related to a product (from the product itself).
                    -->
                    <filter string="My Orders" domain="[('user_id', '=', uid)]" name="my_sale_orders_filter"/>
                    <filter invisible="1" string="Late Activities" name="activities_overdue"
                        domain="[('my_activity_date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                        help="Show all records which has next action date is before today"/>
                    <filter invisible="1" string="Today Activities" name="activities_today"
                        domain="[('my_activity_date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                    <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                        domain="[('my_activity_date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
                    <group expand="0" string="Group By">
                        <filter string="Salesperson" name="salesperson" domain="[]" context="{'group_by': 'user_id'}"/>
                        <filter name="customer" string="Customer" domain="[]" context="{'group_by': 'partner_id'}"/>
                        <filter string="Order Date" name="order_month" domain="[]" context="{'group_by': 'date_order'}"/>
                    </group>
               </search>
            </field>
        </record>

        <record id="sale_order_view_search_inherit_quotation" model="ir.ui.view">
            <field name="name">sale.order.search.inherit.quotation</field>
            <field name="model">sale.order</field>
            <field name="mode">primary</field>
            <field name="inherit_id" ref="sale.view_sales_order_filter"/>
            <field name="arch" type="xml">
                <xpath expr="//filter[@name='my_sale_orders_filter']" position="replace">
                    <field name="campaign_id"/>
                    <separator/>
                    <filter string="My Quotations" name="my_quotation" domain="[('user_id', '=', uid)]"/>
                    <separator/>
                    <filter string="Quotations" name="draft" domain="[('state','in',('draft', 'sent'))]"/>
                    <filter string="Sales Orders" name="sales" domain="[('state','in',('sale','done'))]"/>
                    <separator/>
                    <filter string="Create Date" name="filter_create_date" date="create_date"/>
                </xpath>
            </field>
        </record>

        <record id="sale_order_view_search_inherit_sale" model="ir.ui.view">
            <field name="name">sale.order.search.inherit.sale</field>
            <field name="model">sale.order</field>
            <field name="mode">primary</field>
            <field name="inherit_id" ref="sale.view_sales_order_filter"/>
            <field name="arch" type="xml">
                <xpath expr="//filter[@name='my_sale_orders_filter']" position="after">
                    <separator/>
                    <filter string="To Invoice" name="to_invoice" domain="[('invoice_status','=','to invoice')]" />
                    <filter string="To Upsell" name="upselling" domain="[('invoice_status','=','upselling')]" />
                    <separator/>
                    <filter string="Order Date" name="order_date" date="date_order"/>
                </xpath>
            </field>
        </record>

        <record id="action_orders" model="ir.actions.act_window">
            <field name="name">Sales Orders</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">sale.order</field>
            <field name="view_mode">tree,kanban,form,calendar,pivot,graph,activity</field>
            <field name="search_view_id" ref="sale_order_view_search_inherit_sale"/>
            <field name="context">{}</field>
            <field name="domain">[('state', 'not in', ('draft', 'sent', 'cancel'))]</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Create a new quotation, the first step of a new sale!
                </p><p>
                    Once the quotation is confirmed, it becomes a sales order.<br/> You will be able to create an invoice and collect the payment.
                </p>
            </field>
        </record>

        <record id="action_sale_order_form_view" model="ir.actions.act_window">
            <field name="name">Sales Order</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">sale.order</field>
            <field name="view_mode">form,tree,kanban,calendar,pivot,graph,activity</field>
            <field name="search_view_id" ref="sale_order_view_search_inherit_sale"/>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Create a new quotation, the first step of a new sale!
                </p><p>
                    Once the quotation is confirmed, it becomes a sales order.<br/> You will be able to create an invoice and collect the payment.
                </p>
            </field>
        </record>

        <record id="sale_order_action_view_order_tree" model="ir.actions.act_window.view">
            <field name="sequence" eval="1"/>
            <field name="view_mode">tree</field>
            <field name="view_id" ref="sale.view_order_tree"/>
            <field name="act_window_id" ref="action_orders"/>
        </record>

        <record id="sale_order_action_view_order_kanban" model="ir.actions.act_window.view">
            <field name="sequence" eval="2"/>
            <field name="view_mode">kanban</field>
            <field name="view_id" ref="sale.view_sale_order_kanban"/>
            <field name="act_window_id" ref="action_orders"/>
        </record>

        <record id="sale_order_action_view_order_form" model="ir.actions.act_window.view">
            <field name="sequence" eval="3"/>
            <field name="view_mode">form</field>
            <field name="view_id" ref="sale.view_order_form"/>
            <field name="act_window_id" ref="action_orders"/>
        </record>

        <record id="sale_order_action_view_order_calendar" model="ir.actions.act_window.view">
            <field name="sequence" eval="4"/>
            <field name="view_mode">calendar</field>
            <field name="view_id" ref="sale.view_sale_order_calendar"/>
            <field name="act_window_id" ref="action_orders"/>
        </record>

        <record id="sale_order_action_view_order_pivot" model="ir.actions.act_window.view">
            <field name="sequence" eval="5"/>
            <field name="view_mode">pivot</field>
            <field name="view_id" ref="sale.view_sale_order_pivot"/>
            <field name="act_window_id" ref="action_orders"/>
        </record>

        <record id="sale_order_action_view_order_graph" model="ir.actions.act_window.view">
            <field name="sequence" eval="6"/>
            <field name="view_mode">graph</field>
            <field name="view_id" ref="sale.view_sale_order_graph"/>
            <field name="act_window_id" ref="action_orders"/>
        </record>

        <menuitem id="menu_sale_order"
            name="Orders"
            action="action_orders"
            parent="sale_order_menu"
            sequence="2" groups="sales_team.group_sale_salesman"/>

        <record id="action_orders_to_invoice" model="ir.actions.act_window">
            <field name="name">Orders to Invoice</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">sale.order</field>
            <field name="view_mode">tree,form,calendar,graph,pivot,kanban,activity</field>
            <field name="context">{'create': False}</field>
            <field name="domain">[('invoice_status','=','to invoice')]</field>
            <field name="search_view_id" ref="view_sales_order_filter"/>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                No orders to invoice found
              </p><p>
                You can select all orders and invoice them in batch,<br/>
                or check every order and invoice them one by one.
              </p>
            </field>
        </record>

        <record id="model_sale_order_action_quotation_sent" model="ir.actions.server">
            <field name="name">Mark Quotation as Sent</field>
            <field name="model_id" ref="sale.model_sale_order"/>
            <field name="binding_model_id" ref="sale.model_sale_order"/>
            <field name="binding_view_types">form,list</field>
            <field name="state">code</field>
            <field name="code">action = records.action_quotation_sent()</field>
        </record>

        <record id="action_accrued_revenue_entry" model="ir.actions.act_window">
            <field name="name">Accrued Revenue Entry</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">account.accrued.orders.wizard</field>
            <field name="view_mode">form</field>
            <field name="binding_model_id" ref="sale.model_sale_order"/>
            <field name="groups_id" eval="[(4, ref('account.group_account_user'))]"/>
            <field name="target">new</field>
        </record>

        <menuitem id="menu_sale_invoicing"
            name="To Invoice"
            parent="sale_menu_root"
            sequence="3" groups="sales_team.group_sale_salesman"/>

        <menuitem id="menu_sale_order_invoice"
            action="action_orders_to_invoice"
            parent="sale.menu_sale_invoicing"
            sequence="2" active="False"/>

        <record id="action_orders_upselling" model="ir.actions.act_window">
            <field name="name">Orders to Upsell</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">sale.order</field>
            <field name="view_mode">tree,form,calendar,graph,pivot,kanban,activity</field>
            <field name="domain">[('invoice_status','=','upselling')]</field>
            <field name="context">{'create': False}</field>
            <field name="search_view_id" ref="view_sales_order_filter"/>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                No orders to upsell found.
              </p><p>
                An order is to upsell when delivered quantities are above initially
                ordered quantities, and the invoicing policy is based on ordered quantities.
              </p><p>
                As an example, if you sell pre-paid hours of services, Odoo recommends you
                to sell extra hours when all ordered hours have been consumed.
              </p>
            </field>
        </record>
        <menuitem action="action_orders_upselling"
            id="menu_sale_order_upselling" parent="sale.menu_sale_invoicing"
            sequence="5" active="False"/>


        <record id="action_quotations_with_onboarding" model="ir.actions.act_window">
            <field name="name">Quotations</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">sale.order</field>
            <field name="view_id" ref="view_quotation_tree_with_onboarding"/>
            <field name="view_mode">tree,kanban,form,calendar,pivot,graph,activity</field>
            <field name="search_view_id" ref="sale_order_view_search_inherit_quotation"/>
            <field name="context">{'search_default_my_quotation': 1}</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Create a new quotation, the first step of a new sale!
              </p><p>
                Once the quotation is confirmed by the customer, it becomes a sales order.<br/> You will be able to create an invoice and collect the payment.
              </p>
            </field>
        </record>

        <record id="action_quotations" model="ir.actions.act_window">
            <field name="name">Quotations</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">sale.order</field>
            <field name="view_mode">tree,kanban,form,calendar,pivot,graph,activity</field>
            <field name="search_view_id" ref="sale_order_view_search_inherit_quotation"/>
            <field name="context">{'search_default_my_quotation': 1}</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Create a new quotation, the first step of a new sale!
              </p><p>
                Once the quotation is confirmed by the customer, it becomes a sales order.<br/> You will be able to create an invoice and collect the payment.
              </p>
            </field>
        </record>

        <record id="sale_order_action_view_quotation_tree" model="ir.actions.act_window.view">
            <field name="sequence" eval="1"/>
            <field name="view_mode">tree</field>
            <field name="view_id" ref="sale.view_quotation_tree"/>
            <field name="act_window_id" ref="action_quotations"/>
        </record>

        <record id="sale_order_action_view_quotation_kanban" model="ir.actions.act_window.view">
            <field name="sequence" eval="2"/>
            <field name="view_mode">kanban</field>
            <field name="view_id" ref="sale.view_sale_order_kanban"/>
            <field name="act_window_id" ref="action_quotations"/>
        </record>

        <record id="sale_order_action_view_quotation_form" model="ir.actions.act_window.view">
            <field name="sequence" eval="3"/>
            <field name="view_mode">form</field>
            <field name="view_id" ref="sale.view_order_form"/>
            <field name="act_window_id" ref="action_quotations"/>
        </record>

        <record id="sale_order_action_view_quotation_calendar" model="ir.actions.act_window.view">
            <field name="sequence" eval="4"/>
            <field name="view_mode">calendar</field>
            <field name="view_id" ref="sale.view_sale_order_calendar"/>
            <field name="act_window_id" ref="action_quotations"/>
        </record>

        <record id="sale_order_action_view_quotation_pivot" model="ir.actions.act_window.view">
            <field name="sequence" eval="5"/>
            <field name="view_mode">pivot</field>
            <field name="view_id" ref="sale.view_sale_order_pivot"/>
            <field name="act_window_id" ref="action_quotations"/>
        </record>

        <record id="sale_order_action_view_quotation_graph" model="ir.actions.act_window.view">
            <field name="sequence" eval="6"/>
            <field name="view_mode">graph</field>
            <field name="view_id" ref="sale.view_sale_order_graph"/>
            <field name="act_window_id" ref="action_quotations"/>
        </record>

        <menuitem id="menu_sale_quotations"
                action="action_quotations_with_onboarding"
                parent="sale_order_menu"
                sequence="1" groups="sales_team.group_sale_salesman"/>

        <record id="view_order_line_tree" model="ir.ui.view">
            <field name="name">sale.order.line.tree</field>
            <field name="model">sale.order.line</field>
            <field name="arch" type="xml">
                <tree string="Sales Order Lines" create="false">
                    <field name="order_id"/>
                    <field name="order_partner_id"/>
                    <field name="name"/>
                    <field name="salesman_id"/>
                    <field name="product_uom_qty" string="Qty"/>
                    <field name="qty_delivered"/>
                    <field name="qty_invoiced"/>
                    <field name="qty_to_invoice"/>
                    <field name="product_uom" string="Unit of Measure" groups="uom.group_uom"/>
                    <field name="price_subtotal" sum="Total" widget="monetary"/>
                    <field name="currency_id" invisible="1"/>
                </tree>
            </field>
        </record>

        <record id="sale_order_line_view_form_readonly" model="ir.ui.view">
            <field name="name">sale.order.line.form.readonly</field>
            <field name="model">sale.order.line</field>
            <field name="arch" type="xml">
                <form string="Sales Order Item">
                    <sheet>
                        <div class="oe_title">
                            <h1>
                                <field name="display_name" readonly="1"/>
                            </h1>
                        </div>
                        <group>
                            <group>
                                <field name="order_id" readonly="1"/>
                                <field name="product_id" readonly="1"/>
                                <field name="name" readonly="1"/>
                                <field name="product_uom_qty" readonly="1"/>
                                <field name="qty_delivered" readonly="1"/>
                                <field name="qty_invoiced"/>
                                <field name="product_uom" readonly="1"/>
                                <field name="company_id" options="{'no_create': True}" groups="base.group_multi_company"/>
                                <field name="order_partner_id" invisible="1"/>
                                <field name="display_type" invisible="1"/>
                                <field name="product_updatable" invisible="1"/>
                            </group>
                            <group>
                                <field name="price_unit" readonly="1"/>
                                <field name="discount" groups="product.group_discount_per_so_line" readonly="1"/>
                                <field name="price_subtotal" widget="monetary"/>
                                <field name="tax_id" widget="many2many_tags" readonly="1"/>
                                <field name="price_tax" widget="monetary"/>
                                <field name="price_total" widget="monetary"/>
                                <field name="currency_id" invisible="1"/>
                            </group>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="view_sales_order_line_filter" model="ir.ui.view">
            <field name="name">sale.order.line.select</field>
            <field name="model">sale.order.line</field>
            <field name="arch" type="xml">
                <search string="Search Sales Order">
                    <filter string="To Invoice" name="to_invoice" domain="[('qty_to_invoice','!=', 0)]"  help="Sales Order Lines ready to be invoiced"/>
                    <separator/>
                    <filter string="My Sales Order Lines" name="my_sales_order_lines" domain="[('salesman_id','=',uid)]" help="Sales Order Lines related to a Sales Order of mine"/>
                    <field name="order_id"/>
                    <field name="order_partner_id" operator="child_of"/>
                    <field name="product_id"/>
                    <field name="salesman_id"/>
                    <group expand="0" string="Group By">
                        <filter string="Product" name="product" domain="[]" context="{'group_by':'product_id'}"/>
                        <filter string="Order" name="order" domain="[]" context="{'group_by':'order_id'}"/>
                        <filter string="Salesperson" name="salesperson" domain="[]" context="{'group_by':'salesman_id'}"/>
                    </group>
                </search>
            </field>
        </record>

        <record id="sale_order_line_view_kanban" model="ir.ui.view">
            <field name="name">sale.order.line.kanban</field>
            <field name="model">sale.order.line</field>
            <field name="arch" type="xml">
                <kanban class="o_kanban_mobile">
                    <templates>
                        <t t-name="kanban-box">
                            <div class="oe_kanban_content oe_kanban_global_click">
                                <div class="row">
                                    <div class="col-12">
                                    <field name="display_name"/>
                                    </div>
                                </div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record model="ir.ui.view" id="product_form_view_sale_order_button">
            <field name="name">product.product.sale.order</field>
            <field name="model">product.product</field>
            <field name="inherit_id" ref="product.product_normal_form_view"/>
            <field name="groups_id" eval="[(4, ref('sales_team.group_sale_salesman'))]"/>
            <field name="arch" type="xml">
                <div name="button_box" position="inside">
                    <button class="oe_stat_button" name="action_view_sales"
                        type="object" icon="fa-signal" groups="sales_team.group_sale_salesman" help="Sold in the last 365 days" attrs="{'invisible': [('sale_ok', '=', False)]}">
                        <div class="o_field_widget o_stat_info">
                            <span class="o_stat_value">
                                <field name="sales_count" widget="statinfo" nolabel="1" class="mr4"/>
                                <field name="uom_name"/>
                            </span>
                            <span class="o_stat_text">Sold</span>
                        </div>
                    </button>
                </div>
                <group name="description" position="after">
                    <group string="Warning when Selling this Product" groups="sale.group_warning_sale">
                        <field name="sale_line_warn" nolabel="1"/>
                        <field name="sale_line_warn_msg" colspan="3" nolabel="1"
                                attrs="{'required':[('sale_line_warn','!=','no-message')],'readonly':[('sale_line_warn','=','no-message')]}"/>
                    </group>
                </group>
            </field>
        </record>

        <record model="ir.ui.view" id="product_template_form_view_sale_order_button">
            <field name="name">product.template.sale.order.button</field>
            <field name="model">product.template</field>
            <field name="inherit_id" ref="product.product_template_only_form_view"/>
            <field name="groups_id" eval="[(4, ref('sales_team.group_sale_salesman'))]"/>
            <field name="arch" type="xml">
                <div name="button_box" position="inside">
                    <button class="oe_stat_button" name="action_view_sales"
                        type="object" icon="fa-signal" groups="sales_team.group_sale_salesman" help="Sold in the last 365 days" attrs="{'invisible': [('sale_ok', '=', False)]}">
                        <div class="o_field_widget o_stat_info">
                            <span class="o_stat_value">
                                <field name="sales_count" widget="statinfo" nolabel="1" class="mr4"/>
                                <field name="uom_name"/>
                            </span>
                            <span class="o_stat_text">Sold</span>
                        </div>
                    </button>
                </div>
                <group name="description" position="after">
                    <group string="Warning when Selling this Product" groups="sale.group_warning_sale">
                        <field name="sale_line_warn" nolabel="1"/>
                        <field name="sale_line_warn_msg" colspan="3" nolabel="1"
                                attrs="{'required':[('sale_line_warn','!=','no-message')],'readonly':[('sale_line_warn','=','no-message')], 'invisible':[('sale_line_warn','=','no-message')]}"/>
                    </group>
                </group>
            </field>
        </record>

        <record model="ir.ui.view" id="product_template_form_view_invoice_policy">
            <field name="name">product.template.invoice.policy</field>
            <field name="model">product.template</field>
            <field name="priority">15</field>
            <field name="inherit_id" ref="product.product_template_form_view"/>
            <field name="arch" type="xml">
                <field name="product_variant_count" position="after">
                    <field name="service_type" widget="radio" invisible="True"/>
                    <field name="visible_expense_policy" invisible="1"/>
                </field>
                <field name="detailed_type" position="after">
                    <field name="invoice_policy" required="1"/>
                    <field name="expense_policy" widget="radio" attrs="{'invisible': [('visible_expense_policy', '=', False)]}"/>
                </field>
            </field>
        </record>

        <!-- Update account invoice search view!-->
        <record id="account_invoice_groupby_inherit" model="ir.ui.view">
            <field name="name">account.move.groupby</field>
            <field name="model">account.move</field>
            <field name="inherit_id" ref="account.view_account_invoice_filter"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='invoice_user_id']" position="after">
                    <field name="team_id"/>
                </xpath>
                <xpath expr="//group/filter[@name='status']" position="after">
                    <filter string="Sales Team" name="sales_channel" domain="[]" context="{'group_by':'team_id'}"/>
                </xpath>
            </field>
        </record>

        <record id="account_invoice_view_tree" model="ir.ui.view">
            <field name="name">account.move.tree.inherit.sale</field>
            <field name="model">account.move</field>
            <field name="inherit_id" ref="account.view_invoice_tree"/>
            <field name="arch" type="xml">
                <field name="invoice_user_id" position="after">
                    <field name="team_id" invisible="context.get('default_move_type') not in ('out_invoice', 'out_refund','out_receipt')" optional="hide"/>
                </field>
            </field>
        </record>

        <!-- Update account invoice !-->
        <record model="ir.ui.view" id="account_invoice_form">
            <field name="name">Account Invoice</field>
            <field name="model">account.move</field>
            <field name="inherit_id" ref="account.view_move_form"/>
            <field name="arch" type="xml">
                <data>
                    <xpath expr="//field[@name='partner_id']" position="after">
                       <field name="partner_shipping_id"
                              groups="sale.group_delivery_invoice_address"
                              attrs="{'invisible': [('move_type', 'not in', ('out_invoice', 'out_refund', 'out_receipt'))]}"/>
                   </xpath>
                    <xpath expr="//group[@name='sale_info_group']/field[@name='invoice_user_id']" position="after">
                        <field name="team_id" kanban_view_ref="%(sales_team.crm_team_view_kanban)s"/>
                    </xpath>
                    <xpath expr="//group[@id='other_tab_group']" position="inside">
                        <group string="Marketing"
                               name="utm_link"
                               groups="base.group_no_one"
                               attrs="{'invisible': [('move_type', 'not in', ('out_invoice', 'out_refund'))]}">
                            <field name="campaign_id"/>
                            <field name="medium_id"/>
                            <field name="source_id"/>
                        </group>
                    </xpath>
                </data>
            </field>
        </record>

        <!-- Update account invoice report  -->
        <record id="account_invoice_report_view_tree" model="ir.ui.view">
            <field name="name">account.invoice.report.view.tree.inherit.sale</field>
            <field name="model">account.invoice.report</field>
            <field name="inherit_id" ref="account.account_invoice_report_view_tree"/>
            <field name="arch" type="xml">
                <field name="invoice_user_id" position="after">
                    <field name="team_id" optional="hide"/>
                </field>
            </field>
        </record>

        <!-- search by Salesteams -->
        <record id="action_orders_salesteams" model="ir.actions.act_window">
            <field name="name">Sales Orders</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">sale.order</field>
            <field name="view_mode">tree,form,calendar,graph,kanban,pivot</field>
            <field name="search_view_id" ref="sale.sale_order_view_search_inherit_sale"/>
            <field name="domain">[('state','not in',('draft','sent','cancel'))]</field>
            <field name="context">{
                    'search_default_team_id': [active_id],
                    'default_team_id': active_id,
                }
            </field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Create a new quotation, the first step of a new sale!
              </p><p>
                Once the quotation is confirmed by the customer, it becomes a sales order.<br/> You will be able to create an invoice and collect the payment.
              </p>
            </field>
        </record>
        <record id="action_orders_to_invoice_salesteams" model="ir.actions.act_window">
            <field name="name">Sales Orders</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">sale.order</field>
            <field name="view_mode">tree,form,calendar,graph,kanban,pivot</field>
            <field name="search_view_id" ref="sale.sale_order_view_search_inherit_sale"/>
            <field name="domain">[('invoice_status','=','to invoice')]</field>
            <field name="context">{
                    'search_default_team_id': [active_id],
                    'default_team_id': active_id,
                }
            </field>
        </record>
        <record id="action_quotations_salesteams" model="ir.actions.act_window">
            <field name="name">Quotations</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">sale.order</field>
            <field name="view_id" ref="sale.view_quotation_tree"/>
            <field name="view_mode">tree,form,calendar,graph,kanban,pivot</field>
            <field name="context">{
                    'search_default_team_id': [active_id],
                    'default_team_id': active_id,
                    'show_address': 1,
                }
            </field>
            <field name="domain">[]</field>
            <field name="search_view_id" ref="sale.sale_order_view_search_inherit_quotation"/>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Create a new quotation, the first step of a new sale!
              </p><p>
                Once the quotation is confirmed by the customer, it becomes a sales order.<br/> You will be able to create an invoice and collect the payment.
              </p>
            </field>
        </record>
        <record id="action_invoice_salesteams" model="ir.actions.act_window">
            <field name="name">Invoices</field>
            <field name="res_model">account.move</field>
            <field name="view_mode">tree,form,kanban</field>
            <field name="view_id" ref="account.view_move_tree"/>
            <field name="domain">[
                ('state', '=', 'posted'),
                ('move_type', 'in', ['out_invoice', 'out_refund'])]</field>
            <field name="context">{
                    'search_default_team_id': [active_id],
                    'default_team_id': active_id,
                    'default_move_type':'out_invoice',
                    'move_type':'out_invoice',
                    'journal_type': 'sale',
                }
            </field>
            <field name="search_view_id" ref="account.view_account_invoice_filter"/>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Create a customer invoice
                </p><p>
                    Create invoices, register payments and keep track of the discussions with your customers.
                </p>
            </field>
        </record>

        <record id="action_invoice_salesteams_view_tree" model="ir.actions.act_window.view">
            <field name="sequence">1</field>
            <field name="view_mode">tree</field>
            <field name="act_window_id" ref="sale.action_invoice_salesteams"/>
        </record>

        <record id="action_invoice_salesteams_view_form" model="ir.actions.act_window.view">
            <field name="sequence">2</field>
            <field name="view_mode">form</field>
            <field name="view_id" ref="account.view_move_form"/>
            <field name="act_window_id" ref="sale.action_invoice_salesteams"/>
        </record>

        <record id="action_order_report_quotation_salesteam" model="ir.actions.act_window">
            <field name="name">Quotations Analysis</field>
            <field name="res_model">sale.report</field>
            <field name="view_mode">graph</field>
            <field name="domain">[('state','=','draft'),('team_id', '=', active_id)]</field>
            <field name="context">{'search_default_order_month':1}</field>
            <field name="help">This report performs analysis on your quotations. Analysis check your sales revenues and sort it by different group criteria (salesman, partner, product, etc.) Use this report to perform analysis on sales not having invoiced yet. If you want to analyse your turnover, you should use the Invoice Analysis report in the Accounting application.</field>
        </record>

        <record id="action_order_report_so_salesteam" model="ir.actions.act_window">
            <field name="name">Sales Analysis</field>
            <field name="res_model">sale.report</field>
            <field name="view_mode">graph</field>
            <field name="domain">[('state','not in',('draft','cancel'))]</field>
            <field name="context">{
                'search_default_Sales': 1,
                'search_default_filter_date': 1,
                'search_default_team_id': [active_id]}</field>
            <field name="help">This report performs analysis on your sales orders. Analysis check your sales revenues and sort it by different group criteria (salesman, partner, product, etc.) Use this report to perform analysis on sales not having invoiced yet. If you want to analyse your turnover, you should use the Invoice Analysis report in the Accounting application.</field>
        </record>

        <record id="action_account_invoice_report_salesteam" model="ir.actions.act_window">
            <field name="name">Invoices Analysis</field>
            <field name="res_model">account.invoice.report</field>
            <field name="view_mode">graph</field>
            <field name="domain">[('state', 'not in', ['draft', 'cancel'])]</field>
            <field name="context">{'search_default_month':1, 'search_default_team_id': [active_id]}</field>
            <field name="help">From this report, you can have an overview of the amount invoiced to your customer. The search tool can also be used to personalise your Invoices reports and so, match this analysis to your needs.</field>
        </record>
</odoo>

```

## File: views\utm_campaign_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<record model="ir.ui.view" id="utm_campaign_view_kanban">
    <field name="name">utm.campaign.view.kanban</field>
    <field name="model">utm.campaign</field>
    <field name="inherit_id" ref="utm.utm_campaign_view_kanban"/>
    <field name="arch" type="xml">
        <xpath expr="//div[@id='utm_statistics']" position="inside">
            <div class="mr-3" title="Revenues" groups="sales_team.group_sale_salesman">
                <field name="currency_id" invisible="True"/>
                <small class="font-weight-bold">
                    <field name="invoiced_amount" widget="monetary" options="{'currency_field': 'currency_id'}"/>
                </small>
            </div>
            <div class="mr-3" title="Quotations" groups="sales_team.group_sale_salesman">
                <i class="fa fa-money text-muted"></i>
                <small class="font-weight-bold">
                    <field name="quotation_count"/>
                </small>
            </div>
        </xpath>
    </field>
</record>

<record model="ir.ui.view" id="utm_campaign_view_form">
    <field name="name">utm.campaign.view.form</field>
    <field name="model">utm.campaign</field>
    <field name="inherit_id" ref="utm.utm_campaign_view_form"/>
    <field name="arch" type="xml">
        <xpath expr="//div[hasclass('oe_button_box')]" position="inside">
            <button name="action_redirect_to_invoiced"
                type="object" class="oe_stat_button order-1" icon="fa-usd" groups="sales_team.group_sale_salesman">
                <field name="invoiced_amount" widget="statinfo" string="Revenues"/>
            </button>
            <button name="action_redirect_to_quotations"
                type="object" class="oe_stat_button order-2" icon="fa-money" groups="sales_team.group_sale_salesman">
                <field name="quotation_count" widget="statinfo" string="Quotations"/>
            </button>
        </xpath>
    </field>
</record>
</odoo>

```

## File: views\variant_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="variants">
        <t t-set="attribute_exclusions" t-value="product._get_attribute_exclusions(parent_combination, parent_name)"/>
        <ul t-attf-class="list-unstyled js_add_cart_variants #{ul_class}" t-att-data-attribute_exclusions="json.dumps(attribute_exclusions)">
            <t t-foreach="product.valid_product_template_attribute_line_ids" t-as="ptal">
                <!-- Attributes selection is hidden if there is only one value available and it's not a custom value -->
                <li t-att-data-attribute_id="ptal.attribute_id.id"
                    t-att-data-attribute_name="ptal.attribute_id.name"
                    t-attf-class="variant_attribute #{'d-none' if len(ptal.product_template_value_ids._only_active()) == 1 and not ptal.product_template_value_ids._only_active()[0].is_custom else ''}">

                    <!-- Used to customize layout if the only available attribute value is custom -->
                    <t t-set="single" t-value="len(ptal.product_template_value_ids._only_active()) == 1"/>
                    <t t-set="single_and_custom" t-value="single and ptal.product_template_value_ids._only_active()[0].is_custom" />
                    <strong t-field="ptal.attribute_id.name" class="attribute_name"/>

                    <t t-if="ptal.attribute_id.display_type == 'select'">
                        <select
                            t-att-data-attribute_id="ptal.attribute_id.id"
                            t-attf-class="custom-select css_attribute_select js_variant_change #{ptal.attribute_id.create_variant} #{'d-none' if single_and_custom else ''}"
                            t-att-name="'ptal-%s' % ptal.id">
                            <t t-foreach="ptal.product_template_value_ids._only_active()" t-as="ptav">
                                <option t-att-value="ptav.id"
                                    t-att-data-value_id="ptav.id"
                                    t-att-data-value_name="ptav.name"
                                    t-att-data-attribute_name="ptav.attribute_id.name"
                                    t-att-data-is_custom="ptav.is_custom"
                                    t-att-selected="ptav in combination"
                                    t-att-data-is_single="single"
                                    t-att-data-is_single_and_custom="single_and_custom">
                                    <span t-field="ptav.name"/>
                                    <t t-call="sale.badge_extra_price"/>
                                </option>
                            </t>
                        </select>
                    </t>

                    <t t-if="ptal.attribute_id.display_type == 'radio'">
                        <ul t-att-data-attribute_id="ptal.attribute_id.id" t-attf-class="list-inline list-unstyled #{'d-none' if single_and_custom else ''}">
                            <t t-foreach="ptal.product_template_value_ids._only_active()" t-as="ptav">
                                <li class="list-inline-item form-group js_attribute_value" style="margin: 0;">
                                    <label class="col-form-label">
                                        <div class="custom-control custom-radio">
                                            <input type="radio"
                                                t-attf-class="custom-control-input js_variant_change #{ptal.attribute_id.create_variant}"
                                                t-att-checked="ptav in combination"
                                                t-att-name="'ptal-%s' % ptal.id"
                                                t-att-value="ptav.id"
                                                t-att-data-value_id="ptav.id"
                                                t-att-data-value_name="ptav.name"
                                                t-att-data-attribute_name="ptav.attribute_id.name"
                                                t-att-data-is_custom="ptav.is_custom"
                                                t-att-data-is_single="single"
                                                t-att-data-is_single_and_custom="single_and_custom" />
                                            <div class="radio_input_value custom-control-label">
                                                <span t-field="ptav.name"/>
                                                <t t-call="sale.badge_extra_price"/>
                                            </div>
                                        </div>
                                    </label>
                                </li>
                            </t>
                        </ul>
                    </t>

                    <t t-if="ptal.attribute_id.display_type == 'pills'">
                        <ul t-att-data-attribute_id="ptal.attribute_id.id"
                            t-attf-class="btn-group-toggle list-inline list-unstyled #{'d-none' if single_and_custom else ''}"
                            data-toggle="buttons">
                            <t t-foreach="ptal.product_template_value_ids._only_active()" t-as="ptav">
                                <li t-attf-class="o_variant_pills btn btn-primary mb-1 list-inline-item js_attribute_value #{'active' if ptav in combination else ''}">
                                    <input type="radio"
                                        t-attf-class="js_variant_change #{ptal.attribute_id.create_variant}"
                                        t-att-checked="ptav in combination"
                                        t-att-name="'ptal-%s' % ptal.id"
                                        t-att-value="ptav.id"
                                        t-att-data-value_id="ptav.id"
                                        t-att-id="ptav.id"
                                        t-att-data-value_name="ptav.name"
                                        t-att-data-attribute_name="ptav.attribute_id.name"
                                        t-att-data-is_custom="ptav.is_custom"
                                        t-att-data-is_single_and_custom="single_and_custom"
                                        t-att-autocomplete="off"/>
                                    <div class="radio_input_value o_variant_pills_input_value">
                                        <span t-field="ptav.name"/>
                                        <t t-call="sale.badge_extra_price"/>
                                    </div>
                                </li>
                            </t>
                        </ul>
                    </t>

                    <t t-if="ptal.attribute_id.display_type == 'color'">
                        <ul t-att-data-attribute_id="ptal.attribute_id.id" t-attf-class="list-inline  #{'d-none' if single_and_custom else ''}">
                            <li t-foreach="ptal.product_template_value_ids._only_active()" t-as="ptav" class="list-inline-item mr-1">
                                <label t-attf-style="background-color:#{ptav.html_color or ptav.product_attribute_value_id.name if not ptav.is_custom else ''}"
                                    t-attf-class="css_attribute_color #{'active' if ptav in combination else ''} #{'custom_value' if ptav.is_custom else ''}">
                                    <input type="radio"
                                        t-attf-class="js_variant_change  #{ptal.attribute_id.create_variant}"
                                        t-att-checked="ptav in combination"
                                        t-att-name="'ptal-%s' % ptal.id"
                                        t-att-value="ptav.id"
                                        t-att-title="ptav.name"
                                        t-att-data-value_id="ptav.id"
                                        t-att-data-value_name="ptav.name"
                                        t-att-data-attribute_name="ptav.attribute_id.name"
                                        t-att-data-is_custom="ptav.is_custom"
                                        t-att-data-is_single="single"
                                        t-att-data-is_single_and_custom="single_and_custom"/>
                                </label>
                            </li>
                        </ul>
                    </t>
                </li>
            </t>
        </ul>
    </template>
    <template id="badge_extra_price" name="Badge Extra Price">
        <t t-set="combination_info_variant" t-value="product._get_combination_info(ptav, pricelist=pricelist)"/>
        <span class="badge badge-pill badge-light border" t-if="combination_info_variant['price_extra']">
        <!--
            price_extra is displayed as catalog price instead of
            price after pricelist because it is impossible to
            compute. Indeed, the pricelist rule might depend on the
            selected variant, so the price_extra will be different
            depending on the selected combination. The price of an
            attribute is therefore variable and it's not very
            accurate to display it.

            To cover some generic cases, the price_extra also
            covers the price-included taxes in e-commerce flows.
            (See the override of `_get_combination_info`)
        -->
        <span class="sign_badge_price_extra" t-esc="combination_info_variant['price_extra'] > 0 and '+' or '-'"/>
        <span t-esc="abs(combination_info_variant['price_extra'])" class="variant_price_extra text-muted font-italic" style="white-space: nowrap;"
            t-options='{
                "widget": "monetary",
                "display_currency": (pricelist or product).currency_id
            }'/>
        </span>
    </template>
</odoo>

```

## File: wizard\account_payment_register.py

```python
# -*- coding: utf-8 -*-

from odoo import models


class AccountPaymentRegister(models.TransientModel):
    _inherit = 'account.payment.register'

    def _create_payment_vals_from_wizard(self):
        vals = super()._create_payment_vals_from_wizard()
        # Make sure the account move linked to generated payment
        # belongs to the expected sales team
        # team_id field on account.payment comes from the `_inherits` on account.move model
        vals.update({'team_id': self.line_ids.move_id[0].team_id.id})
        return vals

```

## File: wizard\mail_compose_message.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class MailComposeMessage(models.TransientModel):
    _inherit = 'mail.compose.message'

    def _action_send_mail(self, auto_commit=False):
        if self.model == 'sale.order':
            self = self.with_context(mailing_document_based=True)
            if self.env.context.get('mark_so_as_sent'):
                self = self.with_context(mail_notify_author=self.env.user.partner_id in self.partner_ids)
        return super(MailComposeMessage, self)._action_send_mail(auto_commit=auto_commit)

```

## File: wizard\payment_acquirer_onboarding_wizard.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class PaymentWizard(models.TransientModel):
    """ Override for the sale quotation onboarding panel. """

    _inherit = 'payment.acquirer.onboarding.wizard'
    _name = 'sale.payment.acquirer.onboarding.wizard'
    _description = 'Sale Payment acquire onboarding wizard'

    def _get_default_payment_method(self):
        return self.env.company.sale_onboarding_payment_method or 'digital_signature'

    payment_method = fields.Selection(selection_add=[
        ('digital_signature', "Electronic signature"),
        ('stripe', "Credit & Debit card (via Stripe)"),
        ('paypal', "PayPal"),
        ('other', "Other payment acquirer"),
        ('manual', "Custom payment instructions"),
    ], default=_get_default_payment_method)
    #

    def _set_payment_acquirer_onboarding_step_done(self):
        """ Override. """
        self.env.company.sudo().set_onboarding_step_done('sale_onboarding_order_confirmation_state')

    def add_payment_methods(self, *args, **kwargs):
        self.env.company.sale_onboarding_payment_method = self.payment_method
        if self.payment_method == 'digital_signature':
            self.env.company.portal_confirmation_sign = True
        if self.payment_method in ('paypal', 'stripe', 'other', 'manual'):
            self.env.company.portal_confirmation_pay = True

        return super(PaymentWizard, self).add_payment_methods(*args, **kwargs)

    def _start_stripe_onboarding(self):
        """ Override of payment to set the sale menu as start menu of the payment onboarding. """
        menu_id = self.env.ref('sale.sale_menu_root').id
        return self.env.company._run_payment_onboarding_step(menu_id)

```

## File: wizard\sale_make_invoice_advance.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import time

from odoo import api, fields, models, _
from odoo.exceptions import UserError


class SaleAdvancePaymentInv(models.TransientModel):
    _name = "sale.advance.payment.inv"
    _description = "Sales Advance Payment Invoice"

    @api.model
    def _count(self):
        return len(self._context.get('active_ids', []))

    @api.model
    def _default_product_id(self):
        product_id = self.env['ir.config_parameter'].sudo().get_param('sale.default_deposit_product_id')
        return self.env['product.product'].browse(int(product_id)).exists()

    @api.model
    def _default_deposit_account_id(self):
        return self._default_product_id()._get_product_accounts()['income']

    @api.model
    def _default_deposit_taxes_id(self):
        return self._default_product_id().taxes_id

    @api.model
    def _default_has_down_payment(self):
        if self._context.get('active_model') == 'sale.order' and self._context.get('active_id', False):
            sale_order = self.env['sale.order'].browse(self._context.get('active_id'))
            return sale_order.order_line.filtered(
                lambda sale_order_line: sale_order_line.is_downpayment
            )

        return False

    @api.model
    def _default_currency_id(self):
        if self._context.get('active_model') == 'sale.order' and self._context.get('active_id', False):
            sale_order = self.env['sale.order'].browse(self._context.get('active_id'))
            return sale_order.currency_id

    advance_payment_method = fields.Selection([
        ('delivered', 'Regular invoice'),
        ('percentage', 'Down payment (percentage)'),
        ('fixed', 'Down payment (fixed amount)')
        ], string='Create Invoice', default='delivered', required=True,
        help="A standard invoice is issued with all the order lines ready for invoicing, \
        according to their invoicing policy (based on ordered or delivered quantity).")
    deduct_down_payments = fields.Boolean('Deduct down payments', default=True)
    has_down_payments = fields.Boolean('Has down payments', default=_default_has_down_payment, readonly=True)
    product_id = fields.Many2one('product.product', string='Down Payment Product', domain=[('type', '=', 'service')],
        default=_default_product_id)
    count = fields.Integer(default=_count, string='Order Count')
    amount = fields.Float('Down Payment Amount', digits='Account', help="The percentage of amount to be invoiced in advance, taxes excluded.")
    currency_id = fields.Many2one('res.currency', string='Currency', default=_default_currency_id)
    fixed_amount = fields.Monetary('Down Payment Amount (Fixed)', help="The fixed amount to be invoiced in advance, taxes excluded.")
    deposit_account_id = fields.Many2one("account.account", string="Income Account", domain=[('deprecated', '=', False)],
        help="Account used for deposits", default=_default_deposit_account_id)
    deposit_taxes_id = fields.Many2many("account.tax", string="Customer Taxes", help="Taxes used for deposits", default=_default_deposit_taxes_id)

    @api.onchange('advance_payment_method')
    def onchange_advance_payment_method(self):
        if self.advance_payment_method == 'percentage':
            amount = self.default_get(['amount']).get('amount')
            return {'value': {'amount': amount}}
        return {}

    def _prepare_invoice_values(self, order, name, amount, so_line):
        invoice_vals = {
            'ref': order.client_order_ref,
            'move_type': 'out_invoice',
            'invoice_origin': order.name,
            'invoice_user_id': order.user_id.id,
            'narration': order.note,
            'partner_id': order.partner_invoice_id.id,
            'fiscal_position_id': (order.fiscal_position_id or order.fiscal_position_id.get_fiscal_position(order.partner_id.id)).id,
            'partner_shipping_id': order.partner_shipping_id.id,
            'currency_id': order.pricelist_id.currency_id.id,
            'payment_reference': order.reference,
            'invoice_payment_term_id': order.payment_term_id.id,
            'partner_bank_id': order.company_id.partner_id.bank_ids[:1].id,
            'team_id': order.team_id.id,
            'campaign_id': order.campaign_id.id,
            'medium_id': order.medium_id.id,
            'source_id': order.source_id.id,
            'invoice_line_ids': [(0, 0, {
                'name': name,
                'price_unit': amount,
                'quantity': 1.0,
                'product_id': self.product_id.id,
                'product_uom_id': so_line.product_uom.id,
                'tax_ids': [(6, 0, so_line.tax_id.ids)],
                'sale_line_ids': [(6, 0, [so_line.id])],
                'analytic_tag_ids': [(6, 0, so_line.analytic_tag_ids.ids)],
                'analytic_account_id': order.analytic_account_id.id if not so_line.display_type and order.analytic_account_id.id else False,
            })],
        }

        return invoice_vals

    def _get_advance_details(self, order):
        context = {'lang': order.partner_id.lang}
        if self.advance_payment_method == 'percentage':
            advance_product_taxes = self.product_id.taxes_id.filtered(lambda tax: tax.company_id == order.company_id)
            if all(order.fiscal_position_id.map_tax(advance_product_taxes).mapped('price_include')):
                amount = order.amount_total * self.amount / 100
            else:
                amount = order.amount_untaxed * self.amount / 100
            name = _("Down payment of %s%%") % (self.amount)
        else:
            amount = self.fixed_amount
            name = _('Down Payment')
        del context

        return amount, name

    def _create_invoice(self, order, so_line, amount):
        if (self.advance_payment_method == 'percentage' and self.amount <= 0.00) or (self.advance_payment_method == 'fixed' and self.fixed_amount <= 0.00):
            raise UserError(_('The value of the down payment amount must be positive.'))

        amount, name = self._get_advance_details(order)

        invoice_vals = self._prepare_invoice_values(order, name, amount, so_line)

        if order.fiscal_position_id:
            invoice_vals['fiscal_position_id'] = order.fiscal_position_id.id

        invoice = self.env['account.move'].with_company(order.company_id)\
            .sudo().create(invoice_vals).with_user(self.env.uid)
        invoice.message_post_with_view('mail.message_origin_link',
                    values={'self': invoice, 'origin': order},
                    subtype_id=self.env.ref('mail.mt_note').id)
        return invoice

    def _prepare_so_line(self, order, analytic_tag_ids, tax_ids, amount):
        context = {'lang': order.partner_id.lang}
        so_values = {
            'name': _('Down Payment: %s') % (time.strftime('%m %Y'),),
            'price_unit': amount,
            'product_uom_qty': 0.0,
            'order_id': order.id,
            'discount': 0.0,
            'product_uom': self.product_id.uom_id.id,
            'product_id': self.product_id.id,
            'analytic_tag_ids': analytic_tag_ids,
            'tax_id': [(6, 0, tax_ids)],
            'is_downpayment': True,
            'sequence': order.order_line and order.order_line[-1].sequence + 1 or 10,
        }
        del context
        return so_values

    def create_invoices(self):
        sale_orders = self.env['sale.order'].browse(self._context.get('active_ids', []))

        if self.advance_payment_method == 'delivered':
            sale_orders._create_invoices(final=self.deduct_down_payments)
        else:
            # Create deposit product if necessary
            if not self.product_id:
                vals = self._prepare_deposit_product()
                self.product_id = self.env['product.product'].create(vals)
                self.env['ir.config_parameter'].sudo().set_param('sale.default_deposit_product_id', self.product_id.id)

            sale_line_obj = self.env['sale.order.line']
            for order in sale_orders:
                amount, name = self._get_advance_details(order)

                if self.product_id.invoice_policy != 'order':
                    raise UserError(_('The product used to invoice a down payment should have an invoice policy set to "Ordered quantities". Please update your deposit product to be able to create a deposit invoice.'))
                if self.product_id.type != 'service':
                    raise UserError(_("The product used to invoice a down payment should be of type 'Service'. Please use another product or update this product."))
                taxes = self.product_id.taxes_id.filtered(lambda r: not order.company_id or r.company_id == order.company_id)
                tax_ids = order.fiscal_position_id.map_tax(taxes).ids
                analytic_tag_ids = []
                for line in order.order_line:
                    analytic_tag_ids = [(4, analytic_tag.id, None) for analytic_tag in line.analytic_tag_ids]

                so_line_values = self._prepare_so_line(order, analytic_tag_ids, tax_ids, amount)
                so_line = sale_line_obj.create(so_line_values)
                self._create_invoice(order, so_line, amount)
        if self._context.get('open_invoices', False):
            return sale_orders.action_view_invoice()
        return {'type': 'ir.actions.act_window_close'}

    def _prepare_deposit_product(self):
        return {
            'name': _('Down payment'),
            'type': 'service',
            'invoice_policy': 'order',
            'property_account_income_id': self.deposit_account_id.id,
            'taxes_id': [(6, 0, self.deposit_taxes_id.ids)],
            'company_id': False,
        }


```

## File: wizard\sale_make_invoice_advance_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="view_sale_advance_payment_inv" model="ir.ui.view">
            <field name="name">Invoice Orders</field>
            <field name="model">sale.advance.payment.inv</field>
            <field name="arch" type="xml">
                <form string="Invoice Sales Order">
                    <p class="oe_grey">
                        Invoices will be created in draft so that you can review
                        them before validation.
                    </p>
                    <group>
                        <field name="count" attrs="{'invisible': [('count','=', 1)]}" readonly="True"/>
                        <field name="advance_payment_method" class="oe_inline" widget="radio"
                            attrs="{'invisible': [('count','&gt;',1)]}"/>
                        <field name="has_down_payments" invisible="1" />
                        <label for="deduct_down_payments" string="" attrs="{'invisible': ['|', ('has_down_payments', '=', False), ('advance_payment_method', '!=', 'delivered')]}"/>
                        <div attrs="{'invisible': ['|', ('has_down_payments', '=', False), ('advance_payment_method', '!=', 'delivered')]}"
                            id="down_payment_details">
                            <field name="deduct_down_payments" nolabel="1"/>
                            <label for="deduct_down_payments"/>
                        </div>
                        <field name="product_id"
                            context="{'default_invoice_policy': 'order'}" class="oe_inline"
                            invisible="1"/>
                        <label for="amount" attrs="{'invisible': [('advance_payment_method', 'not in', ('fixed','percentage'))]}"/>
                        <div attrs="{'invisible': [('advance_payment_method', 'not in', ('fixed','percentage'))]}"
                            id="payment_method_details">
                            <field name="currency_id" invisible="1"/>
                            <field name="fixed_amount"
                                attrs="{'required': [('advance_payment_method', '=', 'fixed')], 'invisible': [('advance_payment_method', '!=','fixed')]}" class="oe_inline"/>
                            <field name="amount"
                                attrs="{'required': [('advance_payment_method', '=', 'percentage')], 'invisible': [('advance_payment_method', '!=', 'percentage')]}" class="oe_inline"/>
                            <span
                                attrs="{'invisible': [('advance_payment_method', '!=', 'percentage')]}" class="oe_inline">%</span>
                        </div>
                        <field name="deposit_account_id" options="{'no_create': True}" class="oe_inline"
                            attrs="{'invisible': ['|', ('advance_payment_method', 'not in', ('fixed', 'percentage')), ('product_id', '!=', False)]}" groups="account.group_account_manager"/>
                        <field name="deposit_taxes_id" class="oe_inline" widget="many2many_tags"
                            domain="[('type_tax_use','=','sale')]"
                            attrs="{'invisible': ['|', ('advance_payment_method', 'not in', ('fixed', 'percentage')), ('product_id', '!=', False)]}"/>
                    </group>
                    <footer>
                        <button name="create_invoices" id="create_invoice_open" string="Create and View Invoice" type="object"
                            context="{'open_invoices': True}" class="btn-primary" data-hotkey="q"/>
                        <button name="create_invoices" id="create_invoice" string="Create Invoice" type="object" data-hotkey="w"/>
                        <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="z"/>
                    </footer>
                </form>
            </field>
        </record>

        <record id="action_view_sale_advance_payment_inv" model="ir.actions.act_window">
            <field name="name">Create invoices</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">sale.advance.payment.inv</field>
            <field name="view_mode">form</field>
            <field name="target">new</field>
            <!-- TODO: check if we need this -->
            <field name="binding_model_id" ref="sale.model_sale_order" />
            <field name="binding_view_types">list</field>
        </record>

</odoo>

```

## File: wizard\sale_order_cancel.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class SaleOrderCancel(models.TransientModel):
    _name = 'sale.order.cancel'
    _description = "Sales Order Cancel"

    order_id = fields.Many2one('sale.order', string='Sale Order', required=True, ondelete='cascade')
    display_invoice_alert = fields.Boolean('Invoice Alert', compute='_compute_display_invoice_alert')

    @api.depends('order_id')
    def _compute_display_invoice_alert(self):
        for wizard in self:
            wizard.display_invoice_alert = bool(wizard.order_id.invoice_ids.filtered(lambda inv: inv.state == 'draft'))

    def action_cancel(self):
        return self.order_id.with_context({'disable_cancel_warning': True}).action_cancel()

```

## File: wizard\sale_order_cancel_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="sale_order_cancel_view_form" model="ir.ui.view">
        <field name="name">sale.order.cancel.form</field>
        <field name="model">sale.order.cancel</field>
        <field name="arch" type="xml">
            <form>
                <field name="order_id" invisible="1"/>
                <field name="display_invoice_alert" invisible="1"/>
                <div attrs="{'invisible': [('display_invoice_alert', '=', False)]}">
                    Draft invoices for this order will be cancelled.
                </div>
                <footer>
                    <button string="Confirm" name="action_cancel" type="object" class="oe_highlight" data-hotkey="q"/>
                    <button string="Cancel" class="btn btn-default" special="cancel" data-hotkey="z"/>
                </footer>
            </form>
        </field>
    </record>
</odoo>

```

## File: wizard\sale_payment_link.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from werkzeug import urls

from odoo import api, models


class PaymentLinkWizard(models.TransientModel):
    _inherit = 'payment.link.wizard'
    _description = 'Generate Sales Payment Link'

    @api.model
    def default_get(self, fields):
        res = super().default_get(fields)
        if res['res_id'] and res['res_model'] == 'sale.order':
            record = self.env[res['res_model']].browse(res['res_id'])
            res.update({
                'description': record.name,
                'amount': record.amount_total - sum(record.invoice_ids.filtered(lambda x: x.state != 'cancel').mapped('amount_total')),
                'currency_id': record.currency_id.id,
                'partner_id': record.partner_invoice_id.id,
                'amount_max': record.amount_total
            })
        return res

    def _get_payment_acquirer_available(self, company_id=None, partner_id=None, currency_id=None, sale_order_id=None):
        """ Select and return the acquirers matching the criteria.

        :param int company_id: The company to which acquirers must belong, as a `res.company` id
        :param int partner_id: The partner making the payment, as a `res.partner` id
        :param int currency_id: The payment currency if known beforehand, as a `res.currency` id
        :param int sale_order_id: The sale order currency if known beforehand, as a `sale.order` id
        :return: The compatible acquirers
        :rtype: recordset of `payment.acquirer`
        """
        return self.env['payment.acquirer'].sudo()._get_compatible_acquirers(
            company_id=company_id or self.company_id.id,
            partner_id=partner_id or self.partner_id.id,
            currency_id=currency_id or self.currency_id.id,
            sale_order_id=sale_order_id or self.res_id,
        )

    def _generate_link(self):
        """ Override of payment to add the sale_order_id in the link. """
        for payment_link in self:
            # The sale_order_id field only makes sense if the document is a sales order
            if payment_link.res_model == 'sale.order':
                related_document = self.env[payment_link.res_model].browse(payment_link.res_id)
                base_url = related_document.get_base_url()
                payment_link.link = f'{base_url}/payment/pay' \
                                    f'?reference={urls.url_quote(payment_link.description)}' \
                                    f'&amount={payment_link.amount}' \
                                    f'&sale_order_id={payment_link.res_id}' \
                                    f'{"&acquirer_id=" + str(payment_link.payment_acquirer_selection) if payment_link.payment_acquirer_selection != "all" else "" }' \
                                    f'&access_token={payment_link.access_token}'
                # Order-related fields are retrieved in the controller
            else:
                super(PaymentLinkWizard, payment_link)._generate_link()

```

## File: wizard\sale_payment_link_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="action_sale_order_generate_link" model="ir.actions.act_window">
        <field name="name">Generate a Payment Link</field>
        <field name="res_model">payment.link.wizard</field>
        <field name="view_mode">form</field>
        <field name="view_id" ref="payment.payment_link_wizard_view_form"/>
        <field name="target">new</field>
        <field name="binding_model_id" ref="model_sale_order"/>
        <field name="binding_view_types">form</field>
    </record>

</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_payment_register
from . import mail_compose_message
from . import payment_acquirer_onboarding_wizard
from . import sale_make_invoice_advance
from . import sale_order_cancel
from . import sale_payment_link

```

