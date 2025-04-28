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
    'depends': [
        'sales_team',
        'account_payment',  # -> account, payment, portal
        'utm',
    ],
    'data': [
        'security/ir.model.access.csv',
        'security/res_groups.xml',
        'security/ir_rules.xml',

        'report/account_invoice_report_views.xml',
        'report/ir_actions_report_templates.xml',
        'report/ir_actions_report.xml',
        'report/sale_report_views.xml',

        'data/ir_sequence_data.xml',
        'data/mail_activity_type_data.xml',
        'data/mail_message_subtype_data.xml',
        'data/mail_template_data.xml',
        'data/sale_data.xml',

        'wizard/account_accrued_orders_wizard_views.xml',
        'wizard/payment_link_wizard_views.xml',
        'wizard/sale_make_invoice_advance_views.xml',
        'wizard/sale_order_cancel_views.xml',

        # Define sale order views before their references
        'views/sale_order_views.xml',

        'views/account_views.xml',
        'views/crm_team_views.xml',
        'views/mail_activity_views.xml',
        'views/payment_templates.xml',
        'views/payment_views.xml',
        'views/product_packaging_views.xml',
        'views/product_views.xml',
        'views/res_config_settings_views.xml',
        'views/res_partner_views.xml',
        'views/variant_templates.xml',
        'views/sale_onboarding_views.xml',
        'views/sale_order_line_views.xml',
        'views/sale_portal_templates.xml',
        'views/utm_campaign_views.xml',

        'views/sale_menus.xml',  # Last because referencing actions defined in previous files
    ],
    'demo': [
        'data/product_demo.xml',
        'data/sale_demo.xml',
    ],
    'installable': True,
    'assets': {
        'web.assets_backend': [
            'sale/static/src/scss/sale_onboarding.scss',
            'sale/static/src/scss/product_configurator.scss',
            'sale/static/src/js/sale_progressbar_field.js',
            'sale/static/src/js/tours/sale.js',
            'sale/static/src/js/product_discount_field.js',
            'sale/static/src/js/sale_product_field.js',
            'sale/static/src/xml/**/*',
        ],
        'web.assets_frontend': [
            'sale/static/src/scss/sale_portal.scss',
            'sale/static/src/js/sale_portal_sidebar.js',
            'sale/static/src/js/sale_portal.js',
            'sale/static/src/js/payment_form.js',
        ],
        'web.assets_tests': [
            'sale/static/tests/tours/**/*',
        ],
        'web.qunit_suite_tests': [
            'sale/static/tests/**/*',
            ('remove', 'sale/static/tests/tours/**/*')
        ],
        'web.report_assets_common': [
            'sale/static/src/scss/sale_report.scss',
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
            'html': request.env['ir.qweb']._render('sale.sale_quotation_onboarding_panel', {
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
from odoo.addons.portal.controllers.portal import pager as portal_pager


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

    def _get_sale_searchbar_sortings(self):
        return {
            'date': {'label': _('Order Date'), 'order': 'date_order desc'},
            'name': {'label': _('Reference'), 'order': 'name'},
            'stage': {'label': _('Stage'), 'order': 'state'},
        }

    def _prepare_sale_portal_rendering_values(
        self, page=1, date_begin=None, date_end=None, sortby=None, quotation_page=False, **kwargs
    ):
        SaleOrder = request.env['sale.order']

        if not sortby:
            sortby = 'date'

        partner = request.env.user.partner_id
        values = self._prepare_portal_layout_values()

        if quotation_page:
            url = "/my/quotes"
            domain = self._prepare_quotations_domain(partner)
        else:
            url = "/my/orders"
            domain = self._prepare_orders_domain(partner)

        searchbar_sortings = self._get_sale_searchbar_sortings()

        sort_order = searchbar_sortings[sortby]['order']

        if date_begin and date_end:
            domain += [('create_date', '>', date_begin), ('create_date', '<=', date_end)]

        pager_values = portal_pager(
            url=url,
            total=SaleOrder.search_count(domain),
            page=page,
            step=self._items_per_page,
            url_args={'date_begin': date_begin, 'date_end': date_end, 'sortby': sortby},
        )
        orders = SaleOrder.search(domain, order=sort_order, limit=self._items_per_page, offset=pager_values['offset'])

        values.update({
            'date': date_begin,
            'quotations': orders.sudo() if quotation_page else SaleOrder,
            'orders': orders.sudo() if not quotation_page else SaleOrder,
            'page_name': 'quote' if quotation_page else 'order',
            'pager': pager_values,
            'default_url': url,
            'searchbar_sortings': searchbar_sortings,
            'sortby': sortby,
        })

        return values

    @http.route(['/my/quotes', '/my/quotes/page/<int:page>'], type='http', auth="user", website=True)
    def portal_my_quotes(self, **kwargs):
        values = self._prepare_sale_portal_rendering_values(quotation_page=True, **kwargs)
        request.session['my_quotations_history'] = values['quotations'].ids[:100]
        return request.render("sale.portal_my_quotations", values)

    @http.route(['/my/orders', '/my/orders/page/<int:page>'], type='http', auth="user", website=True)
    def portal_my_orders(self, **kwargs):
        values = self._prepare_sale_portal_rendering_values(quotation_page=False, **kwargs)
        request.session['my_orders_history'] = values['orders'].ids[:100]
        return request.render("sale.portal_my_orders", values)

    @http.route(['/my/orders/<int:order_id>'], type='http', auth="public", website=True)
    def portal_order_page(self, order_id, report_type=None, access_token=None, message=False, download=False, **kw):
        try:
            order_sudo = self._document_check_access('sale.order', order_id, access_token=access_token)
        except (AccessError, MissingError):
            return request.redirect('/my')

        if report_type in ('html', 'pdf', 'text'):
            return self._show_report(model=order_sudo, report_type=report_type, report_ref='sale.action_report_saleorder', download=download)

        if request.env.user.share and access_token:
            # If a public/portal user accesses the order with the access token
            # Log a note on the chatter.
            today = fields.Date.today().isoformat()
            session_obj_date = request.session.get('view_quote_%s' % order_sudo.id)
            if session_obj_date != today:
                # store the date as a string in the session to allow serialization
                request.session['view_quote_%s' % order_sudo.id] = today
                # The "Quotation viewed by customer" log note is an information
                # dedicated to the salesman and shouldn't be translated in the customer/website lgg
                context = {'lang': order_sudo.user_id.partner_id.lang or order_sudo.company_id.partner_id.lang}
                msg = _('Quotation viewed by customer %s', order_sudo.partner_id.name if request.env.user._is_public() else request.env.user.partner_id.name)
                del context
                _message_post_helper(
                    "sale.order",
                    order_sudo.id,
                    message=msg,
                    token=order_sudo.access_token,
                    message_type="notification",
                    subtype_xmlid="mail.mt_note",
                    partner_ids=order_sudo.user_id.sudo().partner_id.ids,
                )

        backend_url = f'/web#model={order_sudo._name}'\
                      f'&id={order_sudo.id}'\
                      f'&action={order_sudo._get_portal_return_action().id}'\
                      f'&view_type=form'
        values = {
            'sale_order': order_sudo,
            'message': message,
            'report_type': 'html',
            'backend_url': backend_url,
            'res_company': order_sudo.company_id,  # Used to display correct company logo
        }

        # Payment values
        if order_sudo._has_to_be_paid():
            values.update(self._get_payment_values(order_sudo))

        if order_sudo.state in ('draft', 'sent', 'cancel'):
            history_session_key = 'my_quotations_history'
        else:
            history_session_key = 'my_orders_history'

        values = self._get_page_view_values(
            order_sudo, access_token, values, history_session_key, False)

        return request.render('sale.sale_order_portal_template', values)

    def _get_payment_values(self, order_sudo):
        """ Return the payment-specific QWeb context values.

        :param recordset order_sudo: The sales order being paid, as a `sale.order` record.
        :return: The payment-specific values.
        :rtype: dict
        """
        logged_in = not request.env.user._is_public()
        providers_sudo = request.env['payment.provider'].sudo()._get_compatible_providers(
            order_sudo.company_id.id,
            order_sudo.partner_id.id,
            order_sudo.amount_total,
            currency_id=order_sudo.currency_id.id,
            sale_order_id=order_sudo.id,
        )  # In sudo mode to read the fields of providers and partner (if not logged in)
        tokens = request.env['payment.token'].search([
            ('provider_id', 'in', providers_sudo.ids),
            ('partner_id', '=', order_sudo.partner_id.id)
        ]) if logged_in else request.env['payment.token']
        # Make sure that the partner's company matches the order's company.
        if not payment_portal.PaymentPortal._can_partner_pay_in_company(
            order_sudo.partner_id, order_sudo.company_id
        ):
            providers_sudo = request.env['payment.provider'].sudo()
            tokens = request.env['payment.token']
        fees_by_provider = {
            provider: provider._compute_fees(
                order_sudo.amount_total,
                order_sudo.currency_id,
                order_sudo.partner_id.country_id,
            ) for provider in providers_sudo.filtered('fees_active')
        }
        return {
            'providers': providers_sudo,
            'tokens': tokens,
            'fees_by_provider': fees_by_provider,
            'show_tokenize_input': PaymentPortal._compute_show_tokenize_input_mapping(
                providers_sudo, logged_in=logged_in, sale_order_id=order_sudo.id
            ),
            'amount': order_sudo.amount_total,
            'currency': order_sudo.pricelist_id.currency_id,
            'partner_id': order_sudo.partner_id.id,
            'access_token': order_sudo.access_token,
            'transaction_route': order_sudo.get_portal_url(suffix='/transaction'),
            'landing_route': order_sudo.get_portal_url(),
        }

    @http.route(['/my/orders/<int:order_id>/accept'], type='json', auth="public", website=True)
    def portal_quote_accept(self, order_id, access_token=None, name=None, signature=None):
        # get from query string if not on json param
        access_token = access_token or request.httprequest.args.get('access_token')
        try:
            order_sudo = self._document_check_access('sale.order', order_id, access_token=access_token)
        except (AccessError, MissingError):
            return {'error': _('Invalid order.')}

        if not order_sudo._has_to_be_signed():
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

        if not order_sudo._has_to_be_paid():
            order_sudo.action_confirm()
            order_sudo._send_order_confirmation_mail()

        pdf = request.env['ir.actions.report'].sudo()._render_qweb_pdf('sale.action_report_saleorder', [order_sudo.id])[0]

        _message_post_helper(
            'sale.order',
            order_sudo.id,
            _('Order signed by %s', name),
            attachments=[('%s.pdf' % order_sudo.name, pdf)],
            token=access_token,
        )

        query_string = '&message=sign_ok'
        if order_sudo._has_to_be_paid(True):
            query_string += '#allow_payment=yes'
        return {
            'force_refresh': True,
            'redirect_url': order_sudo.get_portal_url(query_string=query_string),
        }

    @http.route(['/my/orders/<int:order_id>/decline'], type='http', auth="public", methods=['POST'], website=True)
    def portal_quote_decline(self, order_id, access_token=None, decline_message=None, **kwargs):
        try:
            order_sudo = self._document_check_access('sale.order', order_id, access_token=access_token)
        except (AccessError, MissingError):
            return request.redirect('/my')

        if order_sudo._has_to_be_signed() and decline_message:
            order_sudo._action_cancel()
            _message_post_helper(
                'sale.order',
                order_sudo.id,
                decline_message,
                token=access_token,
            )
            redirect_url = order_sudo.get_portal_url()
        else:
            redirect_url = order_sudo.get_portal_url(query_string="&message=cant_reject")

        return request.redirect(redirect_url)


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
            raise ValidationError(_("The access token is invalid."))

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
        rendering_context_values = super()._get_custom_rendering_context_values(
            sale_order_id=sale_order_id, **kwargs
        )
        if sale_order_id:
            rendering_context_values['sale_order_id'] = sale_order_id

            # Interrupt the payment flow if the sales order has been canceled.
            order_sudo = request.env['sale.order'].sudo().browse(sale_order_id)
            if order_sudo.state == 'cancel':
                rendering_context_values['amount'] = 0.0
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

import json

from odoo import http
from odoo.http import request


class VariantController(http.Controller):
    @http.route(['/sale/get_combination_info'], type='json', auth="user", methods=['POST'])
    def get_combination_info(self, product_template_id, product_id, combination, add_qty, pricelist_id, **kw):
        combination = request.env['product.template.attribute.value'].browse(combination)
        pricelist = self._get_pricelist(pricelist_id)
        cids = request.httprequest.cookies.get('cids', str(request.env.user.company_id.id))
        allowed_company_ids = [int(cid) for cid in cids.split(',')]
        ProductTemplate = request.env['product.template'].with_context(allowed_company_ids=allowed_company_ids)
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
        return request.env['product.template'].browse(int(product_template_id)).create_product_variant(json.loads(product_template_attribute_value_ids))

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
<odoo noupdate="1">

    <record id="seq_sale_order" model="ir.sequence">
        <field name="name">Sales Order</field>
        <field name="code">sale.order</field>
        <field name="prefix">S</field>
        <field name="padding">5</field>
        <field name="company_id" eval="False"/>
    </record>

</odoo>

```

## File: data\mail_activity_type_data.xml

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
    </data>
</odoo>

```

## File: data\mail_message_subtype_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
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

## File: data\mail_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="email_template_edi_sale" model="mail.template">
            <field name="name">Sales: Send Quotation</field>
            <field name="model_id" ref="sale.model_sale_order"/>
            <field name="subject">{{ object.company_id.name }} {{ object.state in ('draft', 'sent') and (ctx.get('proforma') and 'Proforma' or 'Quotation') or 'Order' }} (Ref {{ object.name or 'n/a' }})</field>
            <field name="email_from">{{ (object.user_id.email_formatted or object.company_id.email_formatted or user.email_formatted) }}</field>
            <field name="partner_to">{{ object.partner_id.id }}</field>
            <field name="description">Used by salespeople when they send quotations or proforma to prospects</field>
            <field name="body_html" type="html">
<div style="margin: 0px; padding: 0px;">
    <p style="margin: 0px; padding: 0px; font-size: 13px;">
        <t t-set="doc_name" t-value="'quotation' if object.state in ('draft', 'sent') else 'order'"/>
        Hello,
        <br/><br/>
        Your
        <t t-if="ctx.get('proforma')">
            Pro forma invoice for <t t-out="doc_name or ''">quotation</t> <span style="font-weight: bold;"  t-out="object.name or ''">S00052</span>
            <t t-if="object.origin">
                (with reference: <t t-out="object.origin or ''"></t> )
            </t>
            amounting in <span style="font-weight: bold;"  t-out="format_amount(object.amount_total, object.pricelist_id.currency_id) or ''">$ 10.00</span> is available.
        </t>
        <t t-else="">
            <t t-out="doc_name or ''">quotation</t> <span style="font-weight: bold;" t-out="object.name or ''"></span>
            <t t-if="object.origin">
                (with reference: <t t-out="object.origin or ''">S00052</t> )
            </t>
            amounting in <span style="font-weight: bold;" t-out="format_amount(object.amount_total, object.pricelist_id.currency_id) or ''">$ 10.00</span> is ready for review.
        </t>
        <br/><br/>
        Do not hesitate to contact us if you have any questions.
        <t t-if="not is_html_empty(object.user_id.signature)">
            <br/><br/>
            <t t-out="object.user_id.signature or ''">--<br/>Mitchell Admin</t>
        </t>
        <br/><br/>
    </p>
</div>
            </field>
            <field name="report_template" ref="action_report_saleorder"/>
            <field name="report_name">{{ (object.name or '').replace('/','_') }}</field>
            <field name="lang">{{ object.partner_id.lang }}</field>
            <field name="auto_delete" eval="True"/>
        </record>

        <record id="mail_template_sale_confirmation" model="mail.template">
            <field name="name">Sales: Order Confirmation</field>
            <field name="model_id" ref="sale.model_sale_order"/>
            <field name="subject">{{ object.company_id.name }} {{ (object.get_portal_last_transaction().state == 'pending') and 'Pending Order' or 'Order' }} (Ref {{ object.name or 'n/a' }})</field>
            <field name="email_from">{{ (object.user_id.email_formatted or object.company_id.email_formatted or user.email_formatted) }}</field>
            <field name="partner_to">{{ object.partner_id.id }}</field>
            <field name="description">Sent to customers on order confirmation</field>
            <field name="body_html" type="html">
<div style="margin: 0px; padding: 0px;">
    <p style="margin: 0px; padding: 0px; font-size: 12px;">
        Hello,
        <br/><br/>
        <t t-set="transaction" t-value="object.get_portal_last_transaction()"/>
        Your order <span style="font-weight:bold;" t-out="object.name or ''">S00049</span> amounting in <span style="font-weight:bold;" t-out="format_amount(object.amount_total, object.currency_id) or ''">$ 10.00</span>
        <t t-if="object.state == 'sale' or (transaction and transaction.state in ('done', 'authorized'))">
            has been confirmed.<br/>
            Thank you for your trust!
        </t>
        <t t-elif="transaction and transaction.state == 'pending'">
            is pending. It will be confirmed when the payment is received.
            <t t-if="object.reference">
                Your payment reference is <span style="font-weight:bold;" t-out="object.reference or ''"></span>.
            </t>
        </t>
        <br/><br/>
        Do not hesitate to contact us if you have any questions.
        <t t-if="not is_html_empty(object.user_id.signature)">
            <br/><br/>
            <t t-out="object.user_id.signature or ''">--<br/>Mitchell Admin</t>
        </t>
        <br/><br/>
    </p>
<t t-if="hasattr(object, 'website_id') and object.website_id">
    <div style="margin: 0px; padding: 0px;">
        <table width="100%" style="color: #454748; font-size: 12px; border-collapse: collapse;">
            <tr style="border-bottom: 2px solid #dee2e6;">
                <td style="width: 150px;"><span style="font-weight:bold;">Products</span></td>
                <td></td>
                <td width="15%" align="center"><span style="font-weight:bold;">Quantity</span></td>
                <td width="20%" align="right"><span style="font-weight:bold;">
                <t t-if="object.user_id.has_group('account.group_show_line_subtotals_tax_excluded')">
                    VAT Excl.
                </t>
                <t t-else="">
                    VAT Incl.
                </t>
                </span></td>
            </tr>
        </table>
        <t t-foreach="object.order_line" t-as="line">
            <t t-if="(not hasattr(line, 'is_delivery') or not line.is_delivery) and line.display_type in ['line_section', 'line_note']">
                <table width="100%" style="color: #454748; font-size: 12px; border-collapse: collapse;">
                    <t t-set="loop_cycle_number" t-value="loop_cycle_number or 0" />
                    <tr t-att-style="'background-color: #f2f2f2' if loop_cycle_number % 2 == 0 else 'background-color: #ffffff'">
                        <t t-set="loop_cycle_number" t-value="loop_cycle_number + 1" />
                        <td colspan="4">
                            <t t-if="line.display_type == 'line_section'">
                                <span style="font-weight:bold;" t-out="line.name or ''">Taking care of Trees Course</span>
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
                    <t t-set="loop_cycle_number" t-value="loop_cycle_number or 0" />
                    <tr t-att-style="'background-color: #f2f2f2' if loop_cycle_number % 2 == 0 else 'background-color: #ffffff'">
                        <t t-set="loop_cycle_number" t-value="loop_cycle_number + 1" />
                        <td style="width: 150px;">
                            <img t-attf-src="/web/image/product.product/{{ line.product_id.id }}/image_128" style="width: 64px; height: 64px; object-fit: contain;" alt="Product image"></img>
                        </td>
                        <td align="left" t-out="line.product_id.name or ''">	Taking care of Trees Course</td>
                        <td width="15%" align="center" t-out="line.product_uom_qty or ''">1</td>
                        <td width="20%" align="right"><span style="font-weight:bold;">
                        <t t-if="object.user_id.has_group('account.group_show_line_subtotals_tax_excluded')">
                            <t t-out="format_amount(line.price_reduce_taxexcl, object.currency_id) or ''">$ 10.00</t>
                        </t>
                        <t t-else="">
                            <t t-out="format_amount(line.price_reduce_taxinc, object.currency_id) or ''">$ 10.00</t>
                        </t>
                        </span></td>
                    </tr>
                </table>
            </t>
        </t>
    </div>
    <div style="margin: 0px; padding: 0px;" t-if="hasattr(object, 'carrier_id') and object.carrier_id">
        <table width="100%" style="color: #454748; font-size: 12px; border-spacing: 0px 4px;" align="right">
            <tr>
                <td style="width: 60%"/>
                <td style="width: 30%; border-top: 1px solid #dee2e6;" align="right"><span style="font-weight:bold;">Delivery:</span></td>
                <td style="width: 10%; border-top: 1px solid #dee2e6;" align="right" t-out="format_amount(object.amount_delivery, object.currency_id) or ''">$ 0.00</td>
            </tr>
            <tr>
                <td style="width: 60%"/>
                <td style="width: 30%;" align="right"><span style="font-weight:bold;">SubTotal:</span></td>
                <td style="width: 10%;" align="right" t-out="format_amount(object.amount_untaxed, object.currency_id) or ''">$ 10.00</td>
            </tr>
        </table>
    </div>
    <div style="margin: 0px; padding: 0px;" t-else="">
        <table width="100%" style="color: #454748; font-size: 12px; border-spacing: 0px 4px;" align="right">
            <tr>
                <td style="width: 60%"/>
                <td style="width: 30%; border-top: 1px solid #dee2e6;" align="right"><span style="font-weight:bold;">SubTotal:</span></td>
                <td style="width: 10%; border-top: 1px solid #dee2e6;" align="right" t-out="format_amount(object.amount_untaxed, object.currency_id) or ''">$ 10.00</td>
            </tr>
        </table>
    </div>
    <div style="margin: 0px; padding: 0px;">
        <table width="100%" style="color: #454748; font-size: 12px; border-spacing: 0px 4px;" align="right">
            <tr>
                <td style="width: 60%"/>
                <td style="width: 30%;" align="right"><span style="font-weight:bold;">Taxes:</span></td>
                <td style="width: 10%;" align="right" t-out="format_amount(object.amount_tax, object.currency_id) or ''">$ 0.00</td>
            </tr>
            <tr>
                <td style="width: 60%"/>
                <td style="width: 30%; border-top: 1px solid #dee2e6;" align="right"><span style="font-weight:bold;">Total:</span></td>
                <td style="width: 10%; border-top: 1px solid #dee2e6;" align="right" t-out="format_amount(object.amount_total, object.currency_id) or ''">$ 10.00</td>
            </tr>
        </table>
    </div>
    <div t-if="object.partner_invoice_id" style="margin: 0px; padding: 0px;">
        <table width="100%" style="color: #454748; font-size: 12px;">
            <tr>
                <td style="padding-top: 10px;">
                    <span style="font-weight:bold;">Bill to:</span>
                    <t t-out="object.partner_invoice_id.street or ''">1201 S Figueroa St</t>
                    <t t-out="object.partner_invoice_id.city or ''">Los Angeles</t>
                    <t t-out="object.partner_invoice_id.state_id.name or ''">California</t>
                    <t t-out="object.partner_invoice_id.zip or ''">90015</t>
                    <t t-out="object.partner_invoice_id.country_id.name or ''">United States</t>
                </td>
            </tr>
            <tr>
                <td>
                    <span style="font-weight:bold;">Payment Method:</span>
                    <t t-if="transaction.token_id">
                        <t t-out="transaction.token_id.display_name or ''"></t>
                    </t>
                    <t t-else="">
                        <t t-out="transaction.provider_id.sudo().name or ''"></t>
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
                    <span style="font-weight:bold;">Ship to:</span>
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
                    <span style="font-weight:bold;">Shipping Method:</span>
                    <t t-out="object.carrier_id.name or ''"></t>
                    <t t-if="object.amount_delivery == 0.0">
                        (Free)
                    </t>
                    <t t-else="">
                        (<t t-out="format_amount(object.amount_delivery, object.currency_id) or ''">$ 10.00</t>)
                    </t>
                </td>
            </tr>
            <tr t-if="object.carrier_id.carrier_description">
                <td>
                    <strong>Shipping Description:</strong>
                    <t t-out="object.carrier_id.carrier_description"/>
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

        <record id="sale.mail_template_sale_cancellation" model="mail.template">
            <field name="name">Sales: Order Cancellation</field>
            <field name="model_id" ref="sale.model_sale_order"/>
            <field name="subject">{{ object.company_id.name }} {{ object.type_name }} Cancelled (Ref {{ object.name or 'n/a' }})</field>
            <field name="email_from">{{ (object.user_id.email_formatted or object.company_id.email_formatted or user.email_formatted) }}</field>
            <field name="partner_to">{{ object.partner_id.id }}</field>
            <field name="description">Sent automatically to customers when you cancel an order</field>
            <field name="body_html" type="html">
<div style="margin: 0px; padding: 0px;">
    <p style="margin: 0px; padding: 0px; font-size: 13px;">
        <t t-set="doc_name" t-value="object.type_name"/>
        Dear <t t-out="object.partner_id.name or ''">user</t>,
        <br/><br/>
        Please be advised that your
        <t t-out="doc_name or ''">quotation</t> <strong t-out="object.name or ''">S00052</strong>
        <t t-if="object.origin">
            (with reference: <t t-out="object.origin or ''">S00052</t> )
        </t>
        has been cancelled. Therefore, you should not be charged further for this order.
        If any refund is necessary, this will be executed at best convenience.
        <br/><br/>
        Do not hesitate to contact us if you have any questions.
        <br/>
    </p>
</div>
            </field>
            <field name="lang">{{ object.partner_id.lang }}</field>
            <field name="auto_delete" eval="True"/>
        </record>

    </data>
</odoo>

```

## File: data\product_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="product.consu_delivery_01" model="product.product">
        <field name="invoice_policy">order</field>
    </record>

    <record id="product.consu_delivery_02" model="product.product">
        <field name="invoice_policy">delivery</field>
    </record>

    <record id="product.consu_delivery_03" model="product.product">
        <field name="invoice_policy">delivery</field>
    </record>

    <record id="product.product_order_01" model="product.product">
        <field name="invoice_policy">order</field>
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
</odoo>

```

## File: data\sale_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <!-- TODO remove as it is already the fallback in _find_mail_template -->
    <record id="default_confirmation_template" model="ir.config_parameter">
        <field name="key">sale.default_confirmation_template</field>
        <field name="value" ref="sale.mail_template_sale_confirmation"/>
    </record>

    <record id="default_invoice_email_template" model="ir.config_parameter">
        <field name="key">sale.default_invoice_email_template</field>
        <field name="value" ref="account.email_template_edi_invoice"/>
    </record>

    <record id="send_invoice_cron" model="ir.cron">
        <field name="name">automatic invoicing: send ready invoice</field>
        <field name="model_id" ref="payment.model_payment_transaction" />
        <field name="state">code</field>
        <field name="code">model._cron_send_invoice()</field>
        <field name="user_id" ref="base.user_root" />
        <field name="interval_number">1</field>
        <field name="interval_type">days</field>
        <field name="numbercall">-1</field>
    </record>

</odoo>

```

## File: data\sale_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

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
        <field name="product_id" ref="product.product_product_25"/>
        <field name="product_uom_qty">3</field>
        <field name="price_unit">295.00</field>
    </record>

    <record id="sale_order_line_2" model="sale.order.line">
        <field name="order_id" ref="sale_order_1"/>
        <field name="product_id" ref="product.product_delivery_02"/>
        <field name="product_uom_qty">5</field>
        <field name="price_unit">145.00</field>
    </record>

    <record id="sale_order_line_3" model="sale.order.line">
        <field name="order_id" ref="sale_order_1"/>
        <field name="product_id" ref="product.product_delivery_01"/>
        <field name="product_uom_qty">2</field>
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
        <field name="product_id" ref="product.product_product_1"/>
        <field name="product_uom_qty">24</field>
        <field name="price_unit">75.00</field>
    </record>

    <record id="sale_order_line_5" model="sale.order.line">
        <field name="order_id" ref="sale_order_2"/>
        <field name="product_id" ref="product.product_product_2"/>
        <field name="product_uom_qty">30</field>
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
        <field name="product_id" ref="product.product_product_1"/>
        <field name="product_uom_qty">10</field>
        <field name="price_unit">30.75</field>
    </record>

    <record id="sale_order_line_7" model="sale.order.line">
        <field name="order_id" ref="sale_order_3"/>
        <field name="product_id" ref="product.product_delivery_01"/>
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
        <field name="product_id" ref="product.product_product_1"/>
        <field name="product_uom_qty">16</field>
        <field name="price_unit">75.00</field>
    </record>

    <record id="sale_order_line_9" model="sale.order.line">
        <field name="order_id" ref="sale_order_4"/>
        <field name="product_id" ref="product.product_delivery_02"/>
        <field name="product_uom_qty">10</field>
        <field name="price_unit">45.00</field>
    </record>

    <record id="sale_order_line_10" model="sale.order.line">
        <field name="order_id" ref="sale_order_4"/>
        <field name="product_id" ref="product.consu_delivery_02"/>
        <field name="product_uom_qty">3</field>
        <field name="price_unit">150.00</field>
    </record>

    <record id="sale_order_line_11" model="sale.order.line">
        <field name="order_id" ref="sale_order_4"/>
        <field name="product_id" ref="product.product_delivery_01"/>
        <field name="product_uom_qty">2</field>
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
        <field name="product_id" ref="product.product_delivery_02"/>
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
        <field name="product_id" ref="product.product_product_4"/>
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
        <field name="product_id" ref="product.product_product_25"/>
        <field name="product_uom_qty">5</field>
        <field name="price_unit">295.00</field>
    </record>

    <record id="sale_order_line_17" model="sale.order.line">
        <field name="order_id" ref="sale_order_7"/>
        <field name="product_id" ref="product.consu_delivery_01"/>
        <field name="price_unit">173.00</field>
    </record>

    <record id="sale_order_line_18" model="sale.order.line">
        <field name="order_id" ref="sale_order_7"/>
        <field name="product_id" ref="product.product_delivery_02"/>
        <field name="price_unit">40.00</field>
    </record>

    <record id="sale_order_line_19" model="sale.order.line">
        <field name="order_id" ref="sale_order_7"/>
        <field name="product_id" ref="product.product_delivery_01"/>
        <field name="price_unit">18.00</field>
    </record>

    <record id="sale_order_8" model="sale.order">
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
        <field name="product_id" ref="product.product_product_27"/>
        <field name="product_uom_qty">2</field>
        <field name="price_unit">110.50</field>
    </record>

    <record id="sale_order_line_21" model="sale.order.line">
        <field name="order_id" ref="sale_order_8"/>
        <field name="product_id" ref="product.product_product_12"/>
        <field name="product_uom_qty">2</field>
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
        <field name="product_id" ref="product.product_product_27"/>
        <field name="product_uom_qty">3</field>
        <field name="price_unit">97.50</field>
    </record>

    <record id="sale_order_line_23" model="sale.order.line">
        <field name="order_id" ref="sale_order_9"/>
        <field name="product_id" ref="product.product_product_12"/>
        <field name="product_uom_qty">3</field>
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
        <field name="product_id" ref="product.product_product_25"/>
        <field name="product_uom_qty">2</field>
        <field name="price_unit">255.00</field>
    </record>

    <record id="sale_order_line_25" model="sale.order.line">
        <field name="order_id" ref="sale_order_10"/>
        <field name="product_id" ref="product.product_product_12"/>
        <field name="product_uom_qty">2</field>
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
        <field name="product_id" ref="product.product_product_25"/>
        <field name="product_uom_qty">3</field>
        <field name="price_unit">245.00</field>
    </record>

    <record id="sale_order_line_27" model="sale.order.line">
        <field name="order_id" ref="sale_order_11"/>
        <field name="product_id" ref="product.product_product_12"/>
        <field name="product_uom_qty">3</field>
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
        <field name="product_id" ref="product.product_product_25"/>
        <field name="price_unit">315.00</field>
    </record>

    <record id="sale_order_line_29" model="sale.order.line">
        <field name="order_id" ref="sale_order_12"/>
        <field name="product_id" ref="product.product_product_12"/>
        <field name="product_uom_qty">2</field>
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
        <field name="product_id" ref="product.product_product_25"/>
        <field name="price_unit">295.00</field>
    </record>

    <record id="sale_order_line_31" model="sale.order.line">
        <field name="order_id" ref="sale_order_13"/>
        <field name="product_id" ref="product.product_product_12"/>
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
        <field name="product_id" ref="product.product_product_25"/>
        <field name="product_uom_qty">4</field>
        <field name="price_unit">275.00</field>
    </record>

    <record id="sale_order_line_33" model="sale.order.line">
        <field name="order_id" ref="sale_order_14"/>
        <field name="product_id" ref="product.product_product_12"/>
        <field name="product_uom_qty">4</field>
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
        <field name="product_id" ref="product.product_product_25"/>
        <field name="product_uom_qty">4</field>
        <field name="price_unit">295.00</field>
    </record>

    <record id="sale_order_line_35" model="sale.order.line">
        <field name="order_id" ref="sale_order_15"/>
        <field name="product_id" ref="product.product_product_12"/>
        <field name="product_uom_qty">3</field>
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
        <field name="product_id" ref="product.product_product_25"/>
        <field name="product_uom_qty">3</field>
        <field name="price_unit">275.00</field>
    </record>

    <record id="sale_order_line_37" model="sale.order.line">
        <field name="order_id" ref="sale_order_16"/>
        <field name="product_id" ref="product.product_product_12"/>
        <field name="product_uom_qty">3</field>
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
        <field name="product_id" ref="product.product_product_25"/>
        <field name="product_uom_qty">2</field>
        <field name="price_unit">355.00</field>
    </record>

    <record id="sale_order_line_39" model="sale.order.line">
        <field name="order_id" ref="sale_order_17"/>
        <field name="product_id" ref="product.product_product_12"/>
        <field name="product_uom_qty">2</field>
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
        <field name="product_id" ref="product.product_product_25"/>
        <field name="product_uom_qty">2</field>
        <field name="price_unit">295.00</field>
    </record>

    <record id="sale_order_line_41" model="sale.order.line">
        <field name="order_id" ref="sale_order_18"/>
        <field name="product_id" ref="product.product_product_12"/>
        <field name="product_uom_qty">2</field>
        <field name="price_unit">120.50</field>
    </record>

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
        <field name="product_id" ref="product.product_product_25"/>
        <field name="product_uom_qty">3</field>
        <field name="price_unit">295.00</field>
    </record>

    <record id="portal_sale_order_line_2" model="sale.order.line">
        <field name="order_id" ref="portal_sale_order_1"/>
        <field name="product_id" ref="product.product_delivery_02"/>
        <field name="product_uom_qty">5</field>
        <field name="price_unit">145.00</field>
    </record>

    <record id="portal_sale_order_line_3" model="sale.order.line">
        <field name="order_id" ref="portal_sale_order_1"/>
        <field name="product_id" ref="product.product_delivery_01"/>
        <field name="product_uom_qty">2</field>
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
        <field name="product_id" ref="product.product_product_1"/>
        <field name="product_uom_qty">24</field>
        <field name="price_unit">75.00</field>
    </record>

    <record id="portal_sale_order_line_5" model="sale.order.line">
        <field name="order_id" ref="portal_sale_order_2"/>
        <field name="product_id" ref="product.product_product_2"/>
        <field name="product_uom_qty">30</field>
        <field name="price_unit">38.25</field>
    </record>

    <!-- Confirm some Sales Orders-->
    <function model="sale.order" name="action_confirm" eval="[[
        ref('sale_order_4'),
        ref('sale_order_6'),
        ref('sale_order_7'),
        ref('sale_order_8'),
        ref('sale_order_9'),
        ref('sale_order_10'),
        ref('sale_order_11'),
        ref('sale_order_12'),
        ref('sale_order_13'),
        ref('sale_order_14'),
        ref('sale_order_15'),
        ref('sale_order_16'),
        ref('sale_order_17'),
        ref('sale_order_18'),
        ref('portal_sale_order_2'),
    ]]"/>

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

    <record id="portal_sale_order_2" model="sale.order">
        <field name="date_order" eval="datetime.now() - relativedelta(months=1)"/>
    </record>

    <!-- Mail messages in SO's chatter -->
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
        <field name="body">
Alright, thanks for the clarification. I will confirm the order as soon as I get my manager's approval.
        </field>
        <field name="message_type">comment</field>
        <field name="author_id" ref="base.partner_demo"/>
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

</odoo>

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError
from odoo.tools import groupby


class AccountMove(models.Model):
    _name = 'account.move'
    _inherit = ['account.move', 'utm.mixin']

    team_id = fields.Many2one(
        'crm.team', string='Sales Team',
        compute='_compute_team_id', store=True, readonly=False,
        ondelete="set null", tracking=True,
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")

    # UTMs - enforcing the fact that we want to 'set null' when relation is unlinked
    campaign_id = fields.Many2one(ondelete='set null')
    medium_id = fields.Many2one(ondelete='set null')
    source_id = fields.Many2one(ondelete='set null')
    sale_order_count = fields.Integer(compute="_compute_origin_so_count", string='Sale Order Count')

    def unlink(self):
        downpayment_lines = self.mapped('line_ids.sale_line_ids').filtered(lambda line: line.is_downpayment and line.invoice_lines <= self.mapped('line_ids'))
        res = super(AccountMove, self).unlink()
        if downpayment_lines:
            downpayment_lines.unlink()
        return res

    @api.depends('invoice_user_id')
    def _compute_team_id(self):
        applicable_moves = self.filtered(
            lambda move:
                move.is_sale_document(include_receipts=True)
        )

        for ((user_id, company_id), moves) in groupby(
            applicable_moves,
            key=lambda m: (m.invoice_user_id.id, m.company_id.id)
        ):
            self.env['account.move'].concat(*moves).team_id = self.env['crm.team'].with_context(
                allowed_company_ids=[company_id]
            )._get_default_team_id(
                user_id=user_id,
            )

    @api.depends('line_ids.sale_line_ids')
    def _compute_origin_so_count(self):
        for move in self:
            move.sale_order_count = len(move.line_ids.sale_line_ids.order_id)

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
        down_payment_lines = self.line_ids.filtered('is_downpayment')
        for line in down_payment_lines:

            if not line.sale_line_ids.display_type:
                line.sale_line_ids._compute_name()

        downpayment_lines = self.line_ids.sale_line_ids.filtered(lambda l: l.is_downpayment and not l.display_type)
        other_so_lines = downpayment_lines.order_id.order_line - downpayment_lines
        real_invoices = set(other_so_lines.invoice_lines.move_id)
        for dpl in downpayment_lines:
            try:
                dpl.price_unit = dpl._get_downpayment_line_price_unit(real_invoices)
                dpl.tax_id = dpl.invoice_lines.tax_ids
            except UserError:
                # a UserError here means the SO was locked, which prevents changing the taxes
                # just ignore the error - this is a nice to have feature and should not be blocking
                pass
        return res

    def button_draft(self):
        res = super().button_draft()

        self.line_ids.filtered('is_downpayment').sale_line_ids.filtered(
            lambda sol: not sol.display_type)._compute_name()

        return res

    def button_cancel(self):
        res = super().button_cancel()

        self.line_ids.filtered('is_downpayment').sale_line_ids.filtered(
            lambda sol: not sol.display_type)._compute_name()

        return res

    def _post(self, soft=True):
        # OVERRIDE
        # Auto-reconcile the invoice with payments coming from transactions.
        # It's useful when you have a "paid" sale order (using a payment transaction) and you invoice it later.
        posted = super()._post(soft)

        for invoice in posted.filtered(lambda move: move.is_invoice()):
            payments = invoice.mapped('transaction_ids.payment_id').filtered(lambda x: x.state == 'posted')
            move_lines = payments.line_ids.filtered(lambda line: line.account_type in ('asset_receivable', 'liability_payable') and not line.reconciled)
            for line in move_lines:
                invoice.js_assign_outstanding_line(line.id)
        return posted

    def _invoice_paid_hook(self):
        # OVERRIDE
        res = super(AccountMove, self)._invoice_paid_hook()
        todo = set()
        for invoice in self.filtered(lambda move: move.is_invoice()):
            for line in invoice.invoice_line_ids:
                for sale_line in line.sale_line_ids:
                    todo.add((sale_line.order_id, invoice.name))
        for (order, name) in todo:
            order.message_post(body=_("Invoice %s paid", name))
        return res

    def _action_invoice_ready_to_be_sent(self):
        # OVERRIDE
        # Make sure the send invoice CRON is called when an invoice becomes ready to be sent by mail.
        res = super()._action_invoice_ready_to_be_sent()

        send_invoice_cron = self.env.ref('sale.send_invoice_cron', raise_if_not_found=False)
        if send_invoice_cron:
            send_invoice_cron._trigger()

        return res

    def action_view_source_sale_orders(self):
        self.ensure_one()
        source_orders = self.line_ids.sale_line_ids.order_id
        result = self.env['ir.actions.act_window']._for_xml_id('sale.action_orders')
        if len(source_orders) > 1:
            result['domain'] = [('id', 'in', source_orders.ids)]
        elif len(source_orders) == 1:
            result['views'] = [(self.env.ref('sale.view_order_form', False).id, 'form')]
            result['res_id'] = source_orders.id
        else:
            result = {'type': 'ir.actions.act_window_close'}
        return result

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

    is_downpayment = fields.Boolean()
    sale_line_ids = fields.Many2many(
        'sale.order.line',
        'sale_order_line_invoice_rel',
        'invoice_line_id', 'order_line_id',
        string='Sales Order Lines', readonly=True, copy=False)

    def _copy_data_extend_business_fields(self, values):
        # OVERRIDE to copy the 'sale_line_ids' field as well.
        super(AccountMoveLine, self)._copy_data_extend_business_fields(values)
        values['sale_line_ids'] = [(6, None, self.sale_line_ids.ids)]

    def _prepare_analytic_lines(self):
        """ Note: This method is called only on the move.line that having an analytic distribution, and
            so that should create analytic entries.
        """
        values_list = super(AccountMoveLine, self)._prepare_analytic_lines()

        # filter the move lines that can be reinvoiced: a cost (negative amount) analytic line without SO line but with a product can be reinvoiced
        move_to_reinvoice = self.env['account.move.line']
        if len(values_list) > 0:
            for index, move_line in enumerate(self):
                values = values_list[index]
                if 'so_line' not in values:
                    if move_line._sale_can_be_reinvoice():
                        move_to_reinvoice |= move_line

        # insert the sale line in the create values of the analytic entries
        if move_to_reinvoice.filtered(lambda aml: not aml.move_id.reversed_entry_id and aml.product_id):  # only if the move line is not a reversal one
            map_sale_line_per_move = move_to_reinvoice._sale_create_reinvoice_sale_line()
            for values in values_list:
                sale_line = map_sale_line_per_move.get(values.get('move_line_id'))
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

            # raise if the sale order is not currently open
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
        mapping = {}
        for move_line in self:
            if move_line.analytic_distribution:
                distribution_json = move_line.analytic_distribution
                sale_order = self.env['sale.order'].search([('analytic_account_id', 'in', list(int(account_id) for account_id in distribution_json.keys())),
                                                            ('state', '=', 'sale')], order='create_date ASC', limit=1)
                if sale_order:
                    mapping[move_line.id] = sale_order
                else:
                    sale_order = self.env['sale.order'].search([('analytic_account_id', 'in', list(int(account_id) for account_id in distribution_json.keys()))], order='create_date ASC', limit=1)
                    mapping[move_line.id] = sale_order

        # map of AAL index with the SO on which it needs to be reinvoiced. Maybe be None if no SO found
        return mapping

    def _sale_prepare_sale_line_values(self, order, price):
        """ Generate the sale.line creation value from the current move line """
        self.ensure_one()
        last_so_line = self.env['sale.order.line'].search([('order_id', '=', order.id)], order='sequence desc', limit=1)
        last_sequence = last_so_line.sequence + 1 if last_so_line else 100

        fpos = order.fiscal_position_id or order.fiscal_position_id._get_fiscal_position(order.partner_id)
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
            return order.pricelist_id._get_product_price(
                self.product_id,
                1.0,
                uom=self.product_uom_id,
                date=order.date_order,
            )

        uom_precision_digits = self.env['decimal.precision'].precision_get('Product Unit of Measure')
        if float_is_zero(unit_amount, precision_digits=uom_precision_digits):
            return 0.0

        # Prevent unnecessary currency conversion that could be impacted by exchange rate
        # fluctuations
        if self.company_id.currency_id and amount and self.company_id.currency_id == order.currency_id:
            return self.company_id.currency_id.round(abs(amount / unit_amount))

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


class AccountAnalyticApplicability(models.Model):
    _inherit = 'account.analytic.applicability'
    _description = "Analytic Plan's Applicabilities"

    business_domain = fields.Selection(
        selection_add=[
            ('sale_order', 'Sale Order'),
        ],
        ondelete={'sale_order': 'cascade'},
    )

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
        sale_order_data = self.env['sale.order']._read_group([
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
            sale_order_data = self.env['sale.order']._read_group([
                ('team_id', 'in', self.ids),
                ('state', '!=', 'cancel'),
            ], ['team_id'], ['team_id'])
            data_map = {datum['team_id'][0]: datum['team_id_count'] for datum in sale_order_data}
        for team in self:
            team.sale_order_count = data_map.get(team.id, 0)

    def _in_sale_scope(self):
        return self.env.context.get('in_sales_app')

    def _graph_get_model(self):
        if self._in_sale_scope():
            return 'sale.report'
        return super()._graph_get_model()

    def _graph_date_column(self):
        if self._in_sale_scope():
            return 'date'
        return super()._graph_date_column()

    def _graph_get_table(self, GraphModel):
        if self._in_sale_scope():
            # For a team not shared between company, we make sure the amounts are expressed
            # in the currency of the team company and not converted to the current company currency,
            # as the amounts of the sale report are converted in the currency
            # of the current company (for multi-company reporting, see #83550)
            GraphModel = GraphModel.with_company(self.company_id)
            return f"({GraphModel._table_query}) AS {GraphModel._table}"
        return super()._graph_get_table(GraphModel)

    def _graph_y_query(self):
        if self._in_sale_scope():
            return 'SUM(price_subtotal)'
        return super()._graph_y_query()

    def _extra_sql_conditions(self):
        if self._in_sale_scope():
            return "AND state in ('sale', 'done', 'pos_done')"
        return super()._extra_sql_conditions()

    def _graph_title_and_key(self):
        if self._in_sale_scope():
            return ['', _('Sales: Untaxed Total')] # no more title
        return super()._graph_title_and_key()

    def _compute_dashboard_button_name(self):
        super(CrmTeam,self)._compute_dashboard_button_name()
        if self._in_sale_scope():
            self.dashboard_button_name = _("Sales Analysis")

    def action_primary_channel_button(self):
        if self._in_sale_scope():
            return self.env["ir.actions.actions"]._for_xml_id("sale.action_order_report_so_salesteam")
        return super().action_primary_channel_button()

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

## File: models\payment_provider.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class PaymentProvider(models.Model):
    _inherit = 'payment.provider'

    so_reference_type = fields.Selection(string='Communication',
        selection=[
            ('so_name', 'Based on Document Reference'),
            ('partner', 'Based on Customer ID')], default='so_name',
        help='You can set here the communication type that will appear on sales orders.'
             'The communication will be given to the customer when they choose the payment method.')

```

## File: models\payment_transaction.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import logging

from datetime import datetime
from dateutil import relativedelta

from odoo import _, api, Command, fields, models, SUPERUSER_ID
from odoo.tools import format_amount, str2bool

_logger = logging.getLogger(__name__)


class PaymentTransaction(models.Model):
    _inherit = 'payment.transaction'

    sale_order_ids = fields.Many2many('sale.order', 'sale_order_transaction_rel', 'transaction_id', 'sale_order_id',
                                      string='Sales Orders', copy=False, readonly=True)
    sale_order_ids_nbr = fields.Integer(compute='_compute_sale_order_ids_nbr', string='# of Sales Orders')

    def _compute_sale_order_reference(self, order):
        self.ensure_one()
        if self.provider_id.so_reference_type == 'so_name':
            order_reference = order.name
        else:
            # self.provider_id.so_reference_type == 'partner'
            identification_number = order.partner_id.id
            order_reference = '%s/%s' % ('CUST', str(identification_number % 97).rjust(2, '0'))

        invoice_journal = self.env['account.journal'].search([('type', '=', 'sale'), ('company_id', '=', self.env.company.id)], limit=1)
        if invoice_journal:
            order_reference = invoice_journal._process_reference_for_sale_order(order_reference)

        return order_reference

    @api.depends('sale_order_ids')
    def _compute_sale_order_ids_nbr(self):
        for trans in self:
            trans.sale_order_ids_nbr = len(trans.sale_order_ids)

    def _set_pending(self, state_message=None):
        """ Override of `payment` to send the quotations automatically.

        :param str state_message: The reason for which the transaction is set in 'pending' state.
        :return: updated transactions.
        :rtype: `payment.transaction` recordset.
        """
        txs_to_process = super()._set_pending(state_message=state_message)

        for tx in txs_to_process:  # Consider only transactions that are indeed set pending.
            sales_orders = tx.sale_order_ids.filtered(lambda so: so.state in ['draft', 'sent'])
            sales_orders.filtered(
                lambda so: so.state == 'draft'
            ).with_context(tracking_disable=True).action_quotation_sent()

            if tx.provider_id.code == 'custom':
                for so in tx.sale_order_ids:
                    so.reference = tx._compute_sale_order_reference(so)
            # send order confirmation mail.
            sales_orders._send_order_confirmation_mail()

        return txs_to_process

    def _check_amount_and_confirm_order(self):
        """ Confirm the sales order based on the amount of a transaction.

        Confirm the sales orders only if the transaction amount is equal to the total amount of the
        sales orders. Neither partial payments nor grouped payments (paying multiple sales orders in
        one transaction) are not supported.

        :return: The confirmed sales orders.
        :rtype: a `sale.order` recordset
        """
        confirmed_orders = self.env['sale.order']
        for tx in self:
            # We only support the flow where exactly one quotation is linked to a transaction and
            # vice versa.
            if len(tx.sale_order_ids) == 1:
                quotation = tx.sale_order_ids.filtered(lambda so: so.state in ('draft', 'sent'))
                if quotation and len(quotation.transaction_ids.filtered(
                    lambda tx: tx.state in ('authorized', 'done')  # Only consider confirmed tx
                )) == 1:
                    # Check if the SO is fully paid
                    if quotation.currency_id.compare_amounts(tx.amount, quotation.amount_total) == 0:
                        quotation.with_context(send_email=True).action_confirm()
                        confirmed_orders |= quotation
                    else:
                        _logger.warning(
                            '<%(provider)s> transaction AMOUNT MISMATCH for order %(so_name)s '
                            '(ID %(so_id)s): expected %(so_amount)s, got %(tx_amount)s', {
                                'provider': tx.provider_code,
                                'so_name': quotation.name,
                                'so_id': quotation.id,
                                'so_amount': format_amount(
                                    quotation.env, quotation.amount_total, quotation.currency_id
                                ),
                                'tx_amount': format_amount(tx.env, tx.amount, tx.currency_id),
                            },
                        )
        return confirmed_orders

    def _set_authorized(self, state_message=None):
        """ Override of payment to confirm the quotations automatically. """
        super()._set_authorized(state_message=state_message)
        confirmed_orders = self._check_amount_and_confirm_order()
        confirmed_orders._send_order_confirmation_mail()

    def _log_message_on_linked_documents(self, message):
        """ Override of payment to log a message on the sales orders linked to the transaction.

        Note: self.ensure_one()

        :param str message: The message to be logged
        :return: None
        """
        super()._log_message_on_linked_documents(message)
        self = self.with_user(SUPERUSER_ID)  # Log messages as 'OdooBot'
        for order in self.sale_order_ids:
            order.message_post(body=message)

    def _reconcile_after_done(self):
        """ Override of payment to automatically confirm quotations and generate invoices. """
        confirmed_orders = self._check_amount_and_confirm_order()
        confirmed_orders._send_order_confirmation_mail()

        auto_invoice = str2bool(
            self.env['ir.config_parameter'].sudo().get_param('sale.automatic_invoice'))
        if auto_invoice:
            # Invoice the sale orders in self instead of in confirmed_orders to create the invoice
            # even if only a partial payment was made.
            self._invoice_sale_orders()
        super()._reconcile_after_done()
        if auto_invoice:
            # Must be called after the super() call to make sure the invoice are correctly posted.
            self._send_invoice()

    def _send_invoice(self):
        template_id = self.env['ir.config_parameter'].sudo().get_param(
            'sale.default_invoice_email_template'
        )
        if not template_id:
            return
        template_id = int(template_id)
        template = self.env['mail.template'].browse(template_id)
        for tx in self:
            tx = tx.with_company(tx.company_id).with_context(
                company_id=tx.company_id.id,
            )
            invoice_to_send = tx.invoice_ids.filtered(
                lambda i: not i.is_move_sent and i.state == 'posted' and i._is_ready_to_be_sent()
            )
            invoice_to_send.is_move_sent = True # Mark invoice as sent
            for invoice in invoice_to_send:
                lang = template._render_lang(invoice.ids)[invoice.id]
                model_desc = invoice.with_context(lang=lang).type_name
                invoice.with_context(model_description=model_desc).with_user(
                    SUPERUSER_ID
                ).message_post_with_template(
                    template_id=template_id,
                    email_layout_xmlid='mail.mail_notification_layout_with_responsible_signature',
                )

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
        for tx in self.filtered(lambda tx: tx.sale_order_ids):
            # Create invoices
            tx = tx.with_company(tx.company_id).with_context(company_id=tx.company_id.id)
            confirmed_orders = tx.sale_order_ids.filtered(lambda so: so.state in ('sale', 'done'))
            if confirmed_orders:
                confirmed_orders._force_lines_to_invoice_policy_order()
                invoices = confirmed_orders.with_context(
                    raise_if_nothing_to_invoice=False
                )._create_invoices()
                # Setup access token in advance to avoid serialization failure between
                # edi postprocessing of invoice and displaying the sale order on the portal
                for invoice in invoices:
                    invoice._portal_ensure_token()
                tx.invoice_ids = [Command.set(invoices.ids)]

    @api.model
    def _compute_reference_prefix(self, provider_code, separator, **values):
        """ Override of payment to compute the reference prefix based on Sales-specific values.

        If the `values` parameter has an entry with 'sale_order_ids' as key and a list of (4, id, O)
        or (6, 0, ids) X2M command as value, the prefix is computed based on the sales order name(s)
        Otherwise, the computation is delegated to the super method.

        :param str provider_code: The code of the provider handling the transaction
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
        return super()._compute_reference_prefix(provider_code, separator, **values)

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

    sales_count = fields.Float(compute='_compute_sales_count', string='Sold', digits='Product Unit of Measure')

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
        for group in self.env['sale.report']._read_group(domain, ['product_id', 'product_uom_qty'], ['product_id']):
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
        lines = self.env['sale.order.line']._read_group(domain, ['product_id'], ['product_id'])
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

from collections import defaultdict

import json
import logging

from odoo import api, fields, models, _
from odoo.addons.base.models.res_partner import WARNING_MESSAGE, WARNING_HELP
from odoo.exceptions import ValidationError
from odoo.tools.float_utils import float_round

_logger = logging.getLogger(__name__)


class ProductTemplate(models.Model):
    _inherit = 'product.template'

    service_type = fields.Selection(
        [('manual', 'Manually set quantities on order')], string='Track Service',
        compute='_compute_service_type', store=True, readonly=False, precompute=True,
        help="Manually set quantities on order: Invoice based on the manually entered quantity, without creating an analytic account.\n"
             "Timesheets on contract: Invoice based on the tracked hours on the related timesheet.\n"
             "Create a task and track hours: Create a task on the sales order validation and track the work hours.")
    sale_line_warn = fields.Selection(WARNING_MESSAGE, 'Sales Order Line', help=WARNING_HELP, required=True, default="no-message")
    sale_line_warn_msg = fields.Text('Message for Sales Order Line')
    expense_policy = fields.Selection(
        [('no', 'No'),
         ('cost', 'At cost'),
         ('sales_price', 'Sales price')
        ], string='Re-Invoice Expenses', default='no',
        compute='_compute_expense_policy', store=True, readonly=False,
        help="Expenses and vendor bills can be re-invoiced to a customer."
             "With this option, a validated expense can be re-invoice to a customer at its cost or sales price.")
    visible_expense_policy = fields.Boolean("Re-Invoice Policy visible", compute='_compute_visible_expense_policy')
    sales_count = fields.Float(compute='_compute_sales_count', string='Sold', digits='Product Unit of Measure')
    visible_qty_configurator = fields.Boolean("Quantity visible in configurator", compute='_compute_visible_qty_configurator')
    invoice_policy = fields.Selection(
        [('order', 'Ordered quantities'),
         ('delivery', 'Delivered quantities')], string='Invoicing Policy',
        compute='_compute_invoice_policy', store=True, readonly=False, precompute=True,
        help='Ordered Quantity: Invoice quantities ordered by the customer.\n'
             'Delivered Quantity: Invoice quantities delivered to the customer.')

    def _compute_visible_qty_configurator(self):
        for product_template in self:
            product_template.visible_qty_configurator = True

    @api.depends('name')
    def _compute_visible_expense_policy(self):
        visibility = self.user_has_groups('analytic.group_analytic_accounting')
        for product_template in self:
            product_template.visible_expense_policy = visibility

    @api.depends('sale_ok')
    def _compute_expense_policy(self):
        self.filtered(lambda t: not t.sale_ok).expense_policy = 'no'

    @api.depends('product_variant_ids.sales_count')
    def _compute_sales_count(self):
        for product in self:
            product.sales_count = float_round(sum([p.sales_count for p in product.with_context(active_test=False).product_variant_ids]), precision_rounding=product.uom_id.rounding)

    @api.constrains('company_id')
    def _check_sale_product_company(self):
        """Ensure the product is not being restricted to a single company while
        having been sold in another one in the past, as this could cause issues."""
        products_by_company = defaultdict(lambda: self.env['product.template'])
        for product in self:
            if not product.product_variant_ids or not product.company_id:
                # No need to check if the product has just being created (`product_variant_ids` is
                # still empty) or if we're writing `False` on its company (should always work.)
                continue
            products_by_company[product.company_id] |= product

        for target_company, products in products_by_company.items():
            subquery_products = self.env['product.product'].sudo().with_context(active_test=False)._search([('product_tmpl_id', 'in', products.ids)])
            so_lines = self.env['sale.order.line'].sudo().search_read(
                [('product_id', 'in', subquery_products), ('company_id', '!=', target_company.id)],
                fields=['id', 'product_id']
            )
            if so_lines:
                used_products = [sol['product_id'][1] for sol in so_lines]
                raise ValidationError(_('The following products cannot be restricted to the company'
                                        ' %(company)s because they have already been used in quotations or '
                                        'sales orders in another company:\n%(used_products)s\n'
                                        'You can archive these products and recreate them '
                                        'with your company restriction instead, or leave them as '
                                        'shared product.', company=target_company.name, used_products=', '.join(used_products)))

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
        :type product_template_attribute_value_ids: list of id
            of `product.template.attribute.value`

        :return: id of the product variant matching the combination or 0
        :rtype: int
        """
        combination = self.env['product.template.attribute.value'] \
            .browse(product_template_attribute_value_ids)

        return self._create_product_variant(combination, log_warning=True).id or 0

    @api.onchange('type')
    def _onchange_type(self):
        res = super(ProductTemplate, self)._onchange_type()
        if self._origin and self.sales_count > 0:
            res['warning'] = {
                'title': _("Warning"),
                'message': _("You cannot change the product's type because it is already used in sales orders.")
            }
        return res

    @api.depends('type')
    def _compute_service_type(self):
        self.filtered(lambda t: t.type == 'consu' or not t.service_type).service_type = 'manual'

    @api.depends('type')
    def _compute_invoice_policy(self):
        self.filtered(lambda t: t.type == 'consu' or not t.invoice_policy).invoice_policy = 'order'

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
        product_template = self

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
            if pricelist:
                price = pricelist._get_product_price(product, quantity)
            else:
                price = list_price
            display_image = bool(product.image_128)
            display_name = product.display_name
            price_extra = (product.price_extra or 0.0) + (sum(no_variant_attributes_price_extra) or 0.0)
        else:
            current_attributes_price_extra = [v.price_extra or 0.0 for v in combination]
            product_template = product_template.with_context(current_attributes_price_extra=current_attributes_price_extra)
            price_extra = sum(current_attributes_price_extra)
            list_price = product_template.price_compute('list_price')[product_template.id]
            if pricelist:
                price = pricelist._get_product_price(product_template, quantity)
            else:
                price = list_price
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
        ('other', 'Pay with another payment provider'),
        ('manual', 'Manual Payment'),
    ], string="Sale onboarding selected payment method")

    @api.model
    def action_close_sale_quotation_onboarding(self):
        """ Mark the onboarding panel as closed. """
        self.env.company.sale_quotation_onboarding_state = 'closed'

    @api.model
    def action_open_sale_onboarding_payment_provider(self):
        """ Called by onboarding panel above the quotation list."""
        self.env.company.get_chart_of_accounts_or_fail()
        action = self.env["ir.actions.actions"]._for_xml_id("sale.action_open_sale_payment_provider_onboarding_wizard")
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
                default_image_path = get_module_resource('product', 'static/img', 'product_product_13-image.jpg')
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
            default_email_layout_xmlid='mail.mail_notification_layout_with_responsible_signature',
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
        return self._get_and_update_onboarding_state('sale_quotation_onboarding_state', steps)

    _sql_constraints = [('check_quotation_validity_days', 'CHECK(quotation_validity_days > 0)', 'Quotation Validity is required and must be greater than 0.')]

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    # Defaults
    default_invoice_policy = fields.Selection([
        ('order', 'Invoice what is ordered'),
        ('delivery', 'Invoice what is delivered')
        ], 'Invoicing Policy',
        default='order',
        default_model='product.template')

    # Groups
    group_auto_done_setting = fields.Boolean(
        string="Lock Confirmed Sales", implied_group='sale.group_auto_done_setting')
    group_proforma_sales = fields.Boolean(
        string="Pro-Forma Invoice", implied_group='sale.group_proforma_sales',
        help="Allows you to send pro-forma invoice.")
    group_warning_sale = fields.Boolean("Sale Order Warnings", implied_group='sale.group_warning_sale')

    # Config params
    automatic_invoice = fields.Boolean(
        string="Automatic Invoice",
        help="The invoice is generated automatically and available in the customer portal when the "
             "transaction is confirmed by the payment provider.\nThe invoice is marked as paid and "
             "the payment is registered in the payment journal defined in the configuration of the "
             "payment provider.\nThis mode is advised if you issue the final invoice at the order "
             "and not after the delivery.",
        config_parameter='sale.automatic_invoice',
    )
    deposit_default_product_id = fields.Many2one(
        'product.product',
        'Deposit Product',
        domain="[('type', '=', 'service')]",
        config_parameter='sale.default_deposit_product_id',
        help='Default product used for payment advances')

    invoice_mail_template_id = fields.Many2one(
        comodel_name='mail.template',
        string='Invoice Email Template',
        domain="[('model', '=', 'account.move')]",
        config_parameter='sale.default_invoice_email_template',
        help="Email sent to the customer once the invoice is available.",
    )

    use_quotation_validity_days = fields.Boolean(
        "Default Quotation Validity", config_parameter='sale.use_quotation_validity_days')

    # Company setup
    quotation_validity_days = fields.Integer(
        related='company_id.quotation_validity_days', string="Default Quotation Validity (Days)", readonly=False)
    portal_confirmation_sign = fields.Boolean(
        related='company_id.portal_confirmation_sign', string='Online Signature', readonly=False)
    portal_confirmation_pay = fields.Boolean(
        related='company_id.portal_confirmation_pay', string='Online Payment', readonly=False)

    # Modules
    module_delivery = fields.Boolean("Delivery Methods")
    module_delivery_bpost = fields.Boolean("bpost Connector")
    module_delivery_dhl = fields.Boolean("DHL Express Connector")
    module_delivery_easypost = fields.Boolean("Easypost Connector")
    module_delivery_sendcloud = fields.Boolean("Sendcloud Connector")
    module_delivery_fedex = fields.Boolean("FedEx Connector")
    module_delivery_ups = fields.Boolean("UPS Connector")
    module_delivery_usps = fields.Boolean("USPS Connector")

    module_product_email_template = fields.Boolean("Specific Email")
    module_sale_amazon = fields.Boolean("Amazon Sync")
    module_sale_loyalty = fields.Boolean("Coupons & Loyalty")
    module_sale_margin = fields.Boolean("Margins")

    #=== ONCHANGE METHODS ===#

    @api.onchange('use_quotation_validity_days')
    def _onchange_use_quotation_validity_days(self):
        if self.quotation_validity_days <= 0:
            self.quotation_validity_days = self.env['res.company'].default_get(['quotation_validity_days'])['quotation_validity_days']

    @api.onchange('quotation_validity_days')
    def _onchange_quotation_validity_days(self):
        if self.quotation_validity_days <= 0:
            self.quotation_validity_days = self.env['res.company'].default_get(['quotation_validity_days'])['quotation_validity_days']
            return {
                'warning': {'title': _("Warning"), 'message': _("Quotation Validity is required and must be greater than 0.")},
            }

    #=== CRUD METHODS ===#

    def set_values(self):
        super().set_values()
        if self.default_invoice_policy != 'order':
            self.env['ir.config_parameter'].set_param('sale.automatic_invoice', False)

        send_invoice_cron = self.env.ref('sale.send_invoice_cron', raise_if_not_found=False)
        if send_invoice_cron and send_invoice_cron.active != self.automatic_invoice:
            send_invoice_cron.active = self.automatic_invoice

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo.addons.base.models.res_partner import WARNING_MESSAGE, WARNING_HELP
from odoo.osv import expression

class ResPartner(models.Model):
    _inherit = 'res.partner'

    sale_order_count = fields.Integer(compute='_compute_sale_order_count', string='Sale Order Count')
    sale_order_ids = fields.One2many('sale.order', 'partner_id', 'Sales Order')
    sale_warn = fields.Selection(WARNING_MESSAGE, 'Sales Warnings', default='no-message', help=WARNING_HELP)
    sale_warn_msg = fields.Text('Message for Sales Order')

    @api.model
    def _get_sale_order_domain_count(self):
        return []

    def _compute_sale_order_count(self):
        # retrieve all children partners
        all_partners = self.with_context(active_test=False).search([('id', 'child_of', self.ids)])

        sale_order_groups = self.env['sale.order']._read_group(
            domain=expression.AND([self._get_sale_order_domain_count(), [('partner_id', 'in', all_partners.ids)]]),
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

from datetime import timedelta
from itertools import groupby
from markupsafe import Markup

from odoo import api, fields, models, SUPERUSER_ID, _
from odoo.exceptions import AccessError, UserError, ValidationError
from odoo.fields import Command
from odoo.osv import expression
from odoo.tools import float_is_zero, format_amount, format_date, html_keep_url, is_html_empty
from odoo.tools.sql import create_index

from odoo.addons.payment import utils as payment_utils

READONLY_FIELD_STATES = {
    state: [('readonly', True)]
    for state in {'sale', 'done', 'cancel'}
}

LOCKED_FIELD_STATES = {
    state: [('readonly', True)]
    for state in {'done', 'cancel'}
}

INVOICE_STATUS = [
    ('upselling', 'Upselling Opportunity'),
    ('invoiced', 'Fully Invoiced'),
    ('to invoice', 'To Invoice'),
    ('no', 'Nothing to Invoice')
]


class SaleOrder(models.Model):
    _name = 'sale.order'
    _inherit = ['portal.mixin', 'mail.thread', 'mail.activity.mixin', 'utm.mixin']
    _description = "Sales Order"
    _order = 'date_order desc, id desc'
    _check_company_auto = True

    _sql_constraints = [
        ('date_order_conditional_required',
         "CHECK((state IN ('sale', 'done') AND date_order IS NOT NULL) OR state NOT IN ('sale', 'done'))",
         "A confirmed sales order requires a confirmation date."),
    ]

    @property
    def _rec_names_search(self):
        if self._context.get('sale_show_partner_name'):
            return ['name', 'partner_id.name']
        return ['name']

    #=== FIELDS ===#

    name = fields.Char(
        string="Order Reference",
        required=True, copy=False, readonly=True,
        index='trigram',
        states={'draft': [('readonly', False)]},
        default=lambda self: _('New'))

    company_id = fields.Many2one(
        comodel_name='res.company',
        required=True, index=True,
        default=lambda self: self.env.company)
    partner_id = fields.Many2one(
        comodel_name='res.partner',
        string="Customer",
        required=True, readonly=False, change_default=True, index=True,
        tracking=1,
        states=READONLY_FIELD_STATES,
        domain="[('type', '!=', 'private'), ('company_id', 'in', (False, company_id))]")
    state = fields.Selection(
        selection=[
            ('draft', "Quotation"),
            ('sent', "Quotation Sent"),
            ('sale', "Sales Order"),
            ('done', "Locked"),
            ('cancel', "Cancelled"),
        ],
        string="Status",
        readonly=True, copy=False, index=True,
        tracking=3,
        default='draft')

    client_order_ref = fields.Char(string="Customer Reference", copy=False)
    create_date = fields.Datetime(  # Override of default create_date field from ORM
        string="Creation Date", index=True, readonly=True)
    commitment_date = fields.Datetime(
        string="Delivery Date", copy=False,
        states=LOCKED_FIELD_STATES,
        help="This is the delivery date promised to the customer. "
             "If set, the delivery order will be scheduled based on "
             "this date rather than product lead times.")
    date_order = fields.Datetime(
        string="Order Date",
        required=True, readonly=False, copy=False,
        states=READONLY_FIELD_STATES,
        help="Creation date of draft/sent orders,\nConfirmation date of confirmed orders.",
        default=fields.Datetime.now)
    origin = fields.Char(
        string="Source Document",
        help="Reference of the document that generated this sales order request")
    reference = fields.Char(
        string="Payment Ref.",
        help="The payment communication of this sale order.",
        copy=False)

    require_signature = fields.Boolean(
        string="Online Signature",
        compute='_compute_require_signature',
        store=True, readonly=False, precompute=True,
        states=READONLY_FIELD_STATES,
        help="Request a online signature and/or payment to the customer in order to confirm orders automatically.")
    require_payment = fields.Boolean(
        string="Online Payment",
        compute='_compute_require_payment',
        store=True, readonly=False, precompute=True,
        states=READONLY_FIELD_STATES)

    signature = fields.Image(
        string="Signature",
        copy=False, attachment=True, max_width=1024, max_height=1024)
    signed_by = fields.Char(
        string="Signed By", copy=False)
    signed_on = fields.Datetime(
        string="Signed On", copy=False)

    validity_date = fields.Date(
        string="Expiration",
        compute='_compute_validity_date',
        store=True, readonly=False, copy=False, precompute=True,
        states=READONLY_FIELD_STATES)

    # Partner-based computes
    note = fields.Html(
        string="Terms and conditions",
        compute='_compute_note',
        store=True, readonly=False, precompute=True)

    partner_invoice_id = fields.Many2one(
        comodel_name='res.partner',
        string="Invoice Address",
        compute='_compute_partner_invoice_id',
        store=True, readonly=False, required=True, precompute=True,
        states=LOCKED_FIELD_STATES,
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    partner_shipping_id = fields.Many2one(
        comodel_name='res.partner',
        string="Delivery Address",
        compute='_compute_partner_shipping_id',
        store=True, readonly=False, required=True, precompute=True,
        states=LOCKED_FIELD_STATES,
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]",)

    fiscal_position_id = fields.Many2one(
        comodel_name='account.fiscal.position',
        string="Fiscal Position",
        compute='_compute_fiscal_position_id',
        store=True, readonly=False, precompute=True, check_company=True,
        help="Fiscal positions are used to adapt taxes and accounts for particular customers or sales orders/invoices."
            "The default value comes from the customer.",
        domain="[('company_id', '=', company_id)]")
    payment_term_id = fields.Many2one(
        comodel_name='account.payment.term',
        string="Payment Terms",
        compute='_compute_payment_term_id',
        store=True, readonly=False, precompute=True, check_company=True,  # Unrequired company
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    pricelist_id = fields.Many2one(
        comodel_name='product.pricelist',
        string="Pricelist",
        compute='_compute_pricelist_id',
        store=True, readonly=False, precompute=True, check_company=True, required=True,  # Unrequired company
        states=READONLY_FIELD_STATES,
        tracking=1,
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]",
        help="If you change the pricelist, only newly added lines will be affected.")
    currency_id = fields.Many2one(
        related='pricelist_id.currency_id',
        depends=["pricelist_id"],
        store=True, precompute=True, ondelete="restrict")
    currency_rate = fields.Float(
        string="Currency Rate",
        compute='_compute_currency_rate',
        digits=0,
        store=True, precompute=True)
    user_id = fields.Many2one(
        comodel_name='res.users',
        string="Salesperson",
        compute='_compute_user_id',
        store=True, readonly=False, precompute=True, index=True,
        tracking=2,
        domain=lambda self: "[('groups_id', '=', {}), ('share', '=', False), ('company_ids', '=', company_id)]".format(
            self.env.ref("sales_team.group_sale_salesman").id
        ))
    team_id = fields.Many2one(
        comodel_name='crm.team',
        string="Sales Team",
        compute='_compute_team_id',
        store=True, readonly=False, precompute=True, ondelete="set null",
        change_default=True, check_company=True,  # Unrequired company
        tracking=True,
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")

    # Lines and line based computes
    order_line = fields.One2many(
        comodel_name='sale.order.line',
        inverse_name='order_id',
        string="Order Lines",
        states=LOCKED_FIELD_STATES,
        copy=True, auto_join=True)

    amount_untaxed = fields.Monetary(string="Untaxed Amount", store=True, compute='_compute_amounts', tracking=5)
    amount_tax = fields.Monetary(string="Taxes", store=True, compute='_compute_amounts')
    amount_total = fields.Monetary(string="Total", store=True, compute='_compute_amounts', tracking=4)

    invoice_count = fields.Integer(string="Invoice Count", compute='_get_invoiced')
    invoice_ids = fields.Many2many(
        comodel_name='account.move',
        string="Invoices",
        compute='_get_invoiced',
        search='_search_invoice_ids',
        copy=False)
    invoice_status = fields.Selection(
        selection=INVOICE_STATUS,
        string="Invoice Status",
        compute='_compute_invoice_status',
        store=True)

    # Payment fields
    transaction_ids = fields.Many2many(
        comodel_name='payment.transaction',
        relation='sale_order_transaction_rel', column1='sale_order_id', column2='transaction_id',
        string="Transactions",
        copy=False, readonly=True)
    authorized_transaction_ids = fields.Many2many(
        comodel_name='payment.transaction',
        string="Authorized Transactions",
        compute='_compute_authorized_transaction_ids',
        copy=False)

    # UTMs - enforcing the fact that we want to 'set null' when relation is unlinked
    campaign_id = fields.Many2one(ondelete='set null')
    medium_id = fields.Many2one(ondelete='set null')
    source_id = fields.Many2one(ondelete='set null')

    # Followup ?
    analytic_account_id = fields.Many2one(
        comodel_name='account.analytic.account',
        string="Analytic Account",
        copy=False, check_company=True,  # Unrequired company
        states=READONLY_FIELD_STATES,
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    tag_ids = fields.Many2many(
        comodel_name='crm.tag',
        relation='sale_order_tag_rel', column1='order_id', column2='tag_id',
        string="Tags")

    # Remaining non stored computed fields (hide/make fields readonly, ...)
    amount_undiscounted = fields.Float(
        string="Amount Before Discount",
        compute='_compute_amount_undiscounted', digits=0)
    country_code = fields.Char(related='company_id.account_fiscal_country_id.code', string="Country code")
    expected_date = fields.Datetime(
        string="Expected Date",
        compute='_compute_expected_date', store=False,  # Note: can not be stored since depends on today()
        help="Delivery date you can promise to the customer, computed from the minimum lead time of the order lines.")
    is_expired = fields.Boolean(string="Is Expired", compute='_compute_is_expired')
    partner_credit_warning = fields.Text(
        compute='_compute_partner_credit_warning')
    tax_country_id = fields.Many2one(
        comodel_name='res.country',
        compute='_compute_tax_country_id',
        # Avoid access error on fiscal position when reading a sale order with company != user.company_ids
        compute_sudo=True)  # used to filter available taxes depending on the fiscal country and position
    tax_totals = fields.Binary(compute='_compute_tax_totals', exportable=False)
    terms_type = fields.Selection(related='company_id.terms_type')
    type_name = fields.Char(string="Type Name", compute='_compute_type_name')

    # Remaining ux fields (not computed, not stored)

    show_update_fpos = fields.Boolean(
        string="Has Fiscal Position Changed", store=False)  # True if the fiscal position was changed
    show_update_pricelist = fields.Boolean(
        string="Has Pricelist Changed", store=False)  # True if the pricelist was changed


    def init(self):
        create_index(self._cr, 'sale_order_date_order_id_idx', 'sale_order', ["date_order desc", "id desc"])

    #=== COMPUTE METHODS ===#

    @api.depends('company_id')
    def _compute_require_signature(self):
        for order in self:
            order.require_signature = order.company_id.portal_confirmation_sign

    @api.depends('company_id')
    def _compute_require_payment(self):
        for order in self:
            order.require_payment = order.company_id.portal_confirmation_pay

    @api.depends('company_id')
    def _compute_validity_date(self):
        enabled_feature = bool(self.env['ir.config_parameter'].sudo().get_param('sale.use_quotation_validity_days'))
        if not enabled_feature:
            self.validity_date = False
            return

        today = fields.Date.context_today(self)
        for order in self:
            days = order.company_id.quotation_validity_days
            if days > 0:
                order.validity_date = today + timedelta(days)
            else:
                order.validity_date = False

    @api.depends('partner_id')
    def _compute_note(self):
        use_invoice_terms = self.env['ir.config_parameter'].sudo().get_param('account.use_invoice_terms')
        if not use_invoice_terms:
            return
        for order in self:
            order = order.with_company(order.company_id)
            if order.terms_type == 'html' and self.env.company.invoice_terms_html:
                baseurl = html_keep_url(order._get_note_url() + '/terms')
                context = {'lang': order.partner_id.lang or self.env.user.lang}
                order.note = _('Terms & Conditions: %s', baseurl)
                del context
            elif not is_html_empty(self.env.company.invoice_terms):
                order.note = order.with_context(lang=order.partner_id.lang).env.company.invoice_terms

    @api.model
    def _get_note_url(self):
        return self.env.company.get_base_url()

    @api.depends('partner_id')
    def _compute_partner_invoice_id(self):
        for order in self:
            order.partner_invoice_id = order.partner_id.address_get(['invoice'])['invoice'] if order.partner_id else False

    @api.depends('partner_id')
    def _compute_partner_shipping_id(self):
        for order in self:
            order.partner_shipping_id = order.partner_id.address_get(['delivery'])['delivery'] if order.partner_id else False

    @api.depends('partner_shipping_id', 'partner_id', 'company_id')
    def _compute_fiscal_position_id(self):
        """
        Trigger the change of fiscal position when the shipping address is modified.
        """
        cache = {}
        for order in self:
            if not order.partner_id:
                order.fiscal_position_id = False
                continue
            key = (order.company_id.id, order.partner_id.id, order.partner_shipping_id.id)
            if key not in cache:
                cache[key] = self.env['account.fiscal.position'].with_company(
                    order.company_id
                )._get_fiscal_position(order.partner_id, order.partner_shipping_id)
            order.fiscal_position_id = cache[key]

    @api.depends('partner_id')
    def _compute_payment_term_id(self):
        for order in self:
            order = order.with_company(order.company_id)
            order.payment_term_id = order.partner_id.property_payment_term_id

    @api.depends('partner_id')
    def _compute_pricelist_id(self):
        for order in self:
            if not order.partner_id:
                order.pricelist_id = False
                continue
            order = order.with_company(order.company_id)
            order.pricelist_id = order.partner_id.property_product_pricelist

    @api.depends('currency_id', 'date_order', 'company_id')
    def _compute_currency_rate(self):
        cache = {}
        for order in self:
            order_date = (order.date_order or fields.Datetime.now()).date()
            if not order.company_id:
                order.currency_rate = order.currency_id.with_context(date=order_date).rate or 1.0
                continue
            elif not order.currency_id:
                order.currency_rate = 1.0
            else:
                key = (order.company_id.id, order_date, order.currency_id.id)
                if key not in cache:
                    cache[key] = self.env['res.currency']._get_conversion_rate(
                        from_currency=order.company_id.currency_id,
                        to_currency=order.currency_id,
                        company=order.company_id,
                        date=order_date,
                    )
                order.currency_rate = cache[key]

    @api.depends('partner_id')
    def _compute_user_id(self):
        for order in self:
            if order.partner_id and not (order._origin.id and order.user_id):
                # Recompute the salesman on partner change
                #   * if partner is set (is required anyway, so it will be set sooner or later)
                #   * if the order is not saved or has no salesman already
                order.user_id = (
                    order.partner_id.user_id
                    or order.partner_id.commercial_partner_id.user_id
                    or (self.user_has_groups('sales_team.group_sale_salesman') and self.env.user)
                )

    @api.depends('partner_id', 'user_id')
    def _compute_team_id(self):
        cached_teams = {}
        for order in self:
            default_team_id = self.env.context.get('default_team_id', False) or order.partner_id.team_id.id or order.team_id.id
            user_id = order.user_id.id
            company_id = order.company_id.id
            key = (default_team_id, user_id, company_id)
            if key not in cached_teams:
                cached_teams[key] = self.env['crm.team'].with_context(
                    default_team_id=default_team_id
                )._get_default_team_id(
                    user_id=user_id, domain=[('company_id', 'in', [company_id, False])])
            order.team_id = cached_teams[key]

    @api.depends('order_line.price_subtotal', 'order_line.price_tax', 'order_line.price_total')
    def _compute_amounts(self):
        """Compute the total amounts of the SO."""
        for order in self:
            order = order.with_company(order.company_id)
            order_lines = order.order_line.filtered(lambda x: not x.display_type)

            if order.company_id.tax_calculation_rounding_method == 'round_globally':
                tax_results = order.env['account.tax']._compute_taxes([
                    line._convert_to_tax_base_line_dict()
                    for line in order_lines
                ])
                totals = tax_results['totals']
                amount_untaxed = totals.get(order.currency_id, {}).get('amount_untaxed', 0.0)
                amount_tax = totals.get(order.currency_id, {}).get('amount_tax', 0.0)
            else:
                amount_untaxed = sum(order_lines.mapped('price_subtotal'))
                amount_tax = sum(order_lines.mapped('price_tax'))

            order.amount_untaxed = amount_untaxed
            order.amount_tax = amount_tax
            order.amount_total = order.amount_untaxed + order.amount_tax

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
        return [
            ('order_line.invoice_lines.move_id.move_type', 'in', ('out_invoice', 'out_refund')),
            ('order_line.invoice_lines.move_id', operator, value),
        ]

    @api.depends('state', 'order_line.invoice_status')
    def _compute_invoice_status(self):
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

    @api.depends('order_line.customer_lead', 'date_order', 'state')
    def _compute_expected_date(self):
        """ For service and consumable, we only take the min dates. This method is extended in sale_stock to
            take the picking_policy of SO into account.
        """
        self.mapped("order_line")  # Prefetch indication
        for order in self:
            if order.state == 'cancel':
                order.expected_date = False
                continue
            dates_list = order.order_line.filtered(
                lambda line: not line.display_type and not line._is_delivery()
            ).mapped(lambda line: line and line._expected_date())
            if dates_list:
                order.expected_date = order._select_expected_date(dates_list)
            else:
                order.expected_date = False

    def _select_expected_date(self, expected_dates):
        self.ensure_one()
        return min(expected_dates)

    def _compute_is_expired(self):
        today = fields.Date.today()
        for order in self:
            order.is_expired = order.state == 'sent' and order.validity_date and order.validity_date < today

    @api.depends('company_id', 'fiscal_position_id')
    def _compute_tax_country_id(self):
        for record in self:
            if record.fiscal_position_id.foreign_vat:
                record.tax_country_id = record.fiscal_position_id.country_id
            else:
                record.tax_country_id = record.company_id.account_fiscal_country_id

    @api.depends('company_id', 'partner_id', 'amount_total')
    def _compute_partner_credit_warning(self):
        for order in self:
            order.with_company(order.company_id)
            order.partner_credit_warning = ''
            show_warning = order.state in ('draft', 'sent') and \
                           order.company_id.account_use_credit_limit
            if show_warning:
                order_sudo = order.sudo()
                current_credit = order_sudo.partner_id.commercial_partner_id.credit
                order.partner_credit_warning = self.env['account.move']._build_credit_warning_message(
                    record=order_sudo,
                    updated_credit=current_credit + order.amount_total / order.currency_rate,
                )

    @api.depends_context('lang')
    @api.depends('order_line.tax_id', 'order_line.price_unit', 'amount_total', 'amount_untaxed', 'currency_id')
    def _compute_tax_totals(self):
        for order in self:
            order = order.with_company(order.company_id)
            order_lines = order.order_line.filtered(lambda x: not x.display_type)
            order.tax_totals = order.env['account.tax']._prepare_tax_totals(
                [x._convert_to_tax_base_line_dict() for x in order_lines],
                order.currency_id or order.company_id.currency_id,
            )

    @api.depends('state')
    def _compute_type_name(self):
        for record in self:
            if record.state in ('draft', 'sent', 'cancel'):
                record.type_name = _("Quotation")
            else:
                record.type_name = _("Sales Order")

    # portal.mixin override
    def _compute_access_url(self):
        super()._compute_access_url()
        for order in self:
            order.access_url = f'/my/orders/{order.id}'

    #=== CONSTRAINT METHODS ===#

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

    #=== ONCHANGE METHODS ===#

    @api.onchange('commitment_date', 'expected_date')
    def _onchange_commitment_date(self):
        """ Warn if the commitment dates is sooner than the expected date """
        if self.commitment_date and self.expected_date and self.commitment_date < self.expected_date:
            return {
                'warning': {
                    'title': _('Requested date is too soon.'),
                    'message': _("The delivery date is sooner than the expected date."
                                 " You may be unable to honor the delivery date.")
                }
            }

    @api.onchange('fiscal_position_id')
    def _onchange_fpos_id_show_update_fpos(self):
        if self.order_line and (
            not self.fiscal_position_id
            or (self.fiscal_position_id and self._origin.fiscal_position_id != self.fiscal_position_id)
        ):
            self.show_update_fpos = True

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
                self.partner_id = False

            return {
                'warning': {
                    'title': _("Warning for %s", partner.name),
                    'message': partner.sale_warn_msg,
                }
            }

    @api.onchange('pricelist_id')
    def _onchange_pricelist_id_show_update_prices(self):
        if self.order_line and self.pricelist_id and self._origin.pricelist_id != self.pricelist_id:
            self.show_update_pricelist = True

    #=== CRUD METHODS ===#

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            if 'company_id' in vals:
                self = self.with_company(vals['company_id'])
            if vals.get('name', _("New")) == _("New"):
                seq_date = fields.Datetime.context_timestamp(
                    self, fields.Datetime.to_datetime(vals['date_order'])
                ) if 'date_order' in vals else None
                vals['name'] = self.env['ir.sequence'].next_by_code(
                    'sale.order', sequence_date=seq_date) or _("New")

        return super().create(vals_list)

    def copy_data(self, default=None):
        if default is None:
            default = {}
        if 'order_line' not in default:
            default['order_line'] = [
                Command.create(line.copy_data()[0])
                for line in self.order_line.filtered(lambda l: not l.is_downpayment)
            ]
        return super().copy_data(default)

    @api.ondelete(at_uninstall=False)
    def _unlink_except_draft_or_cancel(self):
        for order in self:
            if order.state not in ('draft', 'cancel'):
                raise UserError(_(
                    "You can not delete a sent quotation or a confirmed sales order."
                    " You must first cancel it."))

    #=== ACTION METHODS ===#

    def action_draft(self):
        orders = self.filtered(lambda s: s.state in ['cancel', 'sent'])
        return orders.write({
            'state': 'draft',
            'signature': False,
            'signed_by': False,
            'signed_on': False,
        })

    def action_quotation_send(self):
        """ Opens a wizard to compose an email, with relevant mail template loaded by default """
        self.ensure_one()
        self.order_line._validate_analytic_distribution()
        lang = self.env.context.get('lang')
        mail_template = self._find_mail_template()
        if mail_template and mail_template.lang:
            lang = mail_template._render_lang(self.ids)[self.id]
        ctx = {
            'default_model': 'sale.order',
            'default_res_id': self.id,
            'default_use_template': bool(mail_template),
            'default_template_id': mail_template.id if mail_template else None,
            'default_composition_mode': 'comment',
            'mark_so_as_sent': True,
            'default_email_layout_xmlid': 'mail.mail_notification_layout_with_responsible_signature',
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

    def _find_mail_template(self):
        """ Get the appropriate mail template for the current sales order based on its state.

        If the SO is confirmed, we return the mail template for the sale confirmation.
        Otherwise, we return the quotation email template.

        :return: The correct mail template based on the current status
        :rtype: record of `mail.template` or `None` if not found
        """
        self.ensure_one()
        if self.env.context.get('proforma') or self.state not in ('sale', 'done'):
            return self.env.ref('sale.email_template_edi_sale', raise_if_not_found=False)
        else:
            return self._get_confirmation_template()

    def _get_confirmation_template(self):
        """ Get the mail template sent on SO confirmation (or for confirmed SO's).

        :return: `mail.template` record or None if default template wasn't found
        """
        return self.env.ref('sale.mail_template_sale_confirmation', raise_if_not_found=False)

    def action_quotation_sent(self):
        """ Mark the given draft quotation(s) as sent.

        :raise: UserError if any given SO is not in draft state.
        """
        if self.filtered(lambda so: so.state != 'draft'):
            raise UserError(_("Only draft orders can be marked as sent directly."))

        for order in self:
            order.message_subscribe(partner_ids=order.partner_id.ids)

        self.write({'state': 'sent'})

    def action_confirm(self):
        """ Confirm the given quotation(s) and set their confirmation date.

        If the corresponding setting is enabled, also locks the Sale Order.

        :return: True
        :rtype: bool
        :raise: UserError if trying to confirm locked or cancelled SO's
        """
        if self._get_forbidden_state_confirm() & set(self.mapped('state')):
            raise UserError(_(
                "It is not allowed to confirm an order in the following states: %s",
                ", ".join(self._get_forbidden_state_confirm()),
            ))

        self.order_line._validate_analytic_distribution()

        for order in self:
            order.validate_taxes_on_sales_order()
            if order.partner_id in order.message_partner_ids:
                continue
            order.message_subscribe([order.partner_id.id])

        self.write(self._prepare_confirmation_values())

        # Context key 'default_name' is sometimes propagated up to here.
        # We don't need it and it creates issues in the creation of linked records.
        context = self._context.copy()
        context.pop('default_name', None)

        self.with_context(context)._action_confirm()

        if self[:1].create_uid.has_group('sale.group_auto_done_setting'):
            # Public user can confirm SO, so we check the group on any record creator.
            self.action_done()

        return True

    def _get_forbidden_state_confirm(self):
        return {'done', 'cancel'}

    def _prepare_confirmation_values(self):
        """ Prepare the sales order confirmation values.

        Note: self can contain multiple records.

        :return: Sales Order confirmation values
        :rtype: dict
        """
        return {
            'state': 'sale',
            'date_order': fields.Datetime.now()
        }

    def _action_confirm(self):
        """ Implementation of additional mechanism of Sales Order confirmation.
            This method should be extended when the confirmation should generated
            other documents. In this method, the SO are in 'sale' state (not yet 'done').
        """
        # create an analytic account if at least an expense product
        for order in self:
            if any(expense_policy not in [False, 'no'] for expense_policy in order.order_line.product_id.mapped('expense_policy')):
                if not order.analytic_account_id:
                    order._create_analytic_account()

    def _send_order_confirmation_mail(self):
        if not self:
            return

        if self.env.su:
            # sending mail in sudo was meant for it being sent from superuser
            self = self.with_user(SUPERUSER_ID)

        for sale_order in self:
            mail_template = sale_order._get_confirmation_template()
            if not mail_template:
                continue
            sale_order.with_context(force_send=True).message_post_with_template(
                mail_template.id,
                composition_mode='comment',
                email_layout_xmlid='mail.mail_notification_layout_with_responsible_signature',
            )

    def action_done(self):
        self.write({'state': 'done'})

    def action_unlock(self):
        self.write({'state': 'sale'})

    def action_cancel(self):
        """ Cancel SO after showing the cancel wizard when needed. (cfr :meth:`_show_cancel_wizard`)

        For post-cancel operations, please only override :meth:`_action_cancel`.

        note: self.ensure_one() if the wizard is shown.
        """
        cancel_warning = self._show_cancel_wizard()
        if cancel_warning:
            self.ensure_one()
            template_id = self.env['ir.model.data']._xmlid_to_res_id(
                'sale.mail_template_sale_cancellation', raise_if_not_found=False
            )
            lang = self.env.context.get('lang')
            template = self.env['mail.template'].browse(template_id)
            if template.lang:
                lang = template._render_lang(self.ids)[self.id]
            ctx = {
                'default_use_template': bool(template_id),
                'default_template_id': template_id,
                'default_order_id': self.id,
                'mark_so_as_canceled': True,
                'default_email_layout_xmlid': "mail.mail_notification_layout_with_responsible_signature",
                'model_description': self.with_context(lang=lang).type_name,
            }
            return {
                'name': _('Cancel %s', self.type_name),
                'view_mode': 'form',
                'res_model': 'sale.order.cancel',
                'view_id': self.env.ref('sale.sale_order_cancel_view_form').id,
                'type': 'ir.actions.act_window',
                'context': ctx,
                'target': 'new'
            }
        else:
            return self._action_cancel()

    def _action_cancel(self):
        inv = self.invoice_ids.filtered(lambda inv: inv.state == 'draft')
        inv.button_cancel()
        return self.write({'state': 'cancel'})

    def _show_cancel_wizard(self):
        """ Decide whether the sale.order.cancel wizard should be shown to cancel specified orders.

        :return: True if there is any non-draft order in the given orders
        :rtype: bool
        """
        if self.env.context.get('disable_cancel_warning'):
            return False
        return any(so.state != 'draft' for so in self)

    def action_preview_sale_order(self):
        self.ensure_one()
        return {
            'type': 'ir.actions.act_url',
            'target': 'self',
            'url': self.get_portal_url(),
        }

    def action_update_taxes(self):
        self.ensure_one()

        self._recompute_taxes()

        if self.partner_id:
            self.message_post(body=_(
                "Product taxes have been recomputed according to fiscal position %s.",
                self.fiscal_position_id._get_html_link() if self.fiscal_position_id else "",
            ))

    def _recompute_taxes(self):
        lines_to_recompute = self.order_line.filtered(lambda line: not line.display_type)
        lines_to_recompute._compute_tax_id()
        self.show_update_fpos = False

    def action_update_prices(self):
        self.ensure_one()

        self._recompute_prices()

        if self.pricelist_id:
            self.message_post(body=_(
                "Product prices have been recomputed according to pricelist %s.",
                self.pricelist_id._get_html_link(),
            ))

    def _recompute_prices(self):
        lines_to_recompute = self._get_update_prices_lines()
        lines_to_recompute.invalidate_recordset(['pricelist_item_id'])
        lines_to_recompute._compute_price_unit()
        # Special case: we want to overwrite the existing discount on _recompute_prices call
        # i.e. to make sure the discount is correctly reset
        # if pricelist discount_policy is different than when the price was first computed.
        lines_to_recompute.discount = 0.0
        lines_to_recompute._compute_discount()
        self.show_update_pricelist = False

    # INVOICING #

    def _prepare_invoice(self):
        """
        Prepare the dict of values to create the new invoice for a sales order. This method may be
        overridden to implement custom invoice generation (making sure to call super() to establish
        a clean extension chain).
        """
        self.ensure_one()

        return {
            'ref': self.client_order_ref or '',
            'move_type': 'out_invoice',
            'narration': self.note,
            'currency_id': self.currency_id.id,
            'campaign_id': self.campaign_id.id,
            'medium_id': self.medium_id.id,
            'source_id': self.source_id.id,
            'team_id': self.team_id.id,
            'partner_id': self.partner_invoice_id.id,
            'partner_shipping_id': self.partner_shipping_id.id,
            'fiscal_position_id': (self.fiscal_position_id or self.fiscal_position_id._get_fiscal_position(self.partner_invoice_id)).id,
            'invoice_origin': self.name,
            'invoice_payment_term_id': self.payment_term_id.id,
            'invoice_user_id': self.user_id.id,
            'payment_reference': self.reference,
            'transaction_ids': [Command.set(self.transaction_ids.ids)],
            'company_id': self.company_id.id,
            'invoice_line_ids': [],
            'user_id': self.user_id.id,
        }

    def action_view_invoice(self):
        invoices = self.mapped('invoice_ids')
        action = self.env['ir.actions.actions']._for_xml_id('account.action_move_out_invoice_type')
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

    def _nothing_to_invoice_error_message(self):
        return _(
            "There is nothing to invoice!\n\n"
            "Reason(s) of this behavior could be:\n"
            "- You should deliver your products before invoicing them.\n"
            "- You should modify the invoicing policy of your product: Open the product, go to the "
            "\"Sales\" tab and modify invoicing policy from \"delivered quantities\" to \"ordered "
            "quantities\". For Services, you should modify the Service Invoicing Policy to "
            "'Prepaid'."
        )

    def _get_update_prices_lines(self):
        """ Hook to exclude specific lines which should not be updated based on price list recomputation """
        return self.order_line.filtered(lambda line: not line.display_type)

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
        """ Create invoice(s) for the given Sales Order(s).

        :param bool grouped: if True, invoices are grouped by SO id.
            If False, invoices are grouped by keys returned by :meth:`_get_invoice_grouping_keys`
        :param bool final: if True, refunds will be generated if necessary
        :param date: unused parameter
        :returns: created invoices
        :rtype: `account.move` recordset
        :raises: UserError if one of the orders has no invoiceable lines.
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
            order = order.with_company(order.company_id).with_context(lang=order.partner_invoice_id.lang)

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
                        Command.create(
                            order._prepare_down_payment_section_line(sequence=invoice_item_sequence)
                        ),
                    )
                    down_payment_section_added = True
                    invoice_item_sequence += 1
                invoice_line_vals.append(
                    Command.create(
                        line._prepare_invoice_line(sequence=invoice_item_sequence)
                    ),
                )
                invoice_item_sequence += 1

            invoice_vals['invoice_line_ids'] += invoice_line_vals
            invoice_vals_list.append(invoice_vals)

        if not invoice_vals_list and self._context.get('raise_if_nothing_to_invoice', True):
            raise UserError(self._nothing_to_invoice_error_message())

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
            for _grouping_keys, invoices in groupby(invoice_vals_list, key=lambda x: [x.get(grouping_key) for grouping_key in invoice_grouping_keys]):
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
            move.message_post_with_view(
                'mail.message_origin_link',
                values={'self': move, 'origin': move.line_ids.sale_line_ids.order_id},
                subtype_id=self.env['ir.model.data']._xmlid_to_res_id('mail.mt_note'))
        return moves

    # MAIL #

    @api.returns('mail.message', lambda value: value.id)
    def message_post(self, **kwargs):
        if self.env.context.get('mark_so_as_sent'):
            self.filtered(lambda o: o.state == 'draft').with_context(tracking_disable=True).write({'state': 'sent'})
        return super(SaleOrder, self.with_context(
            mail_post_autofollow=self.env.context.get('mail_post_autofollow', True),
            lang=self.partner_id.lang,
        )).message_post(**kwargs)

    def _notify_get_recipients_groups(self, msg_vals=None):
        """ Give access button to users and portal customer as portal is integrated
        in sale. Customer and portal group have probably no right to see
        the document so they don't have the access button. """
        groups = super()._notify_get_recipients_groups(msg_vals=msg_vals)
        if not self:
            return groups

        self.ensure_one()
        if self._context.get('proforma'):
            for group in [g for g in groups if g[0] in ('portal_customer', 'portal', 'follower', 'customer')]:
                group[2]['has_button_access'] = False
            return groups
        local_msg_vals = dict(msg_vals or {})

        # portal customers have full access (existence not granted, depending on partner_id)
        try:
            customer_portal_group = next(group for group in groups if group[0] == 'portal_customer')
        except StopIteration:
            pass
        else:
            access_opt = customer_portal_group[2].setdefault('button_access', {})
            is_tx_pending = self.get_portal_last_transaction().state == 'pending'
            if self._has_to_be_signed(include_draft=True):
                if self._has_to_be_paid():
                    access_opt['title'] = _("View Quotation") if is_tx_pending else _("Sign & Pay Quotation")
                else:
                    access_opt['title'] = _("Accept & Sign Quotation")
            elif self._has_to_be_paid(include_draft=True) and not is_tx_pending:
                access_opt['title'] = _("Accept & Pay Quotation")
            elif self.state in ('draft', 'sent'):
                access_opt['title'] = _("View Quotation")

        # enable followers that have access through portal
        follower_group = next(group for group in groups if group[0] == 'follower')
        follower_group[2]['active'] = True
        follower_group[2]['has_button_access'] = True
        access_opt = follower_group[2].setdefault('button_access', {})
        if self.state in ('draft', 'sent'):
            access_opt['title'] = _("View Quotation")
        else:
            access_opt['title'] = _("View Order")
        access_opt['url'] = self._notify_get_action_link('view', **local_msg_vals)

        return groups

    def _notify_by_email_prepare_rendering_context(self, message, msg_vals=False, model_description=False,
                                                   force_email_company=False, force_email_lang=False):
        render_context = super()._notify_by_email_prepare_rendering_context(
            message, msg_vals, model_description=model_description,
            force_email_company=force_email_company, force_email_lang=force_email_lang
        )
        lang_code = render_context.get('lang')
        subtitles = [
            render_context['record'].name,
        ]

        if self.amount_total:
            # Do not show the price in subtitles if zero (e.g. e-commerce orders are created empty)
            subtitles.append(
                format_amount(self.env, self.amount_total, self.currency_id, lang_code=lang_code),
            )

        if self.validity_date and self.state in ['draft', 'sent']:
            formatted_date = format_date(self.env, self.validity_date, lang_code=lang_code)
            subtitles.append(_("Expires on %(date)s", date=formatted_date))

        render_context['subtitles'] = subtitles
        return render_context

    def _sms_get_number_fields(self):
        """ No phone or mobile field is available on sale model. Instead SMS will
        fallback on partner-based computation using ``_sms_get_partner_fields``. """
        return []

    def _sms_get_partner_fields(self):
        return ['partner_id']

    def _track_subtype(self, init_values):
        self.ensure_one()
        if 'state' in init_values and self.state == 'sale':
            return self.env.ref('sale.mt_order_confirmed')
        elif 'state' in init_values and self.state == 'sent':
            return self.env.ref('sale.mt_order_sent')
        return super()._track_subtype(init_values)

    # PAYMENT #

    def _force_lines_to_invoice_policy_order(self):
        for line in self.order_line:
            if self.state in ['sale', 'done']:
                line.qty_to_invoice = line.product_uom_qty - line.qty_invoiced
            else:
                line.qty_to_invoice = 0

    def payment_action_capture(self):
        """ Capture all transactions linked to this sale order. """
        payment_utils.check_rights_on_recordset(self)
        # In sudo mode because we need to be able to read on provider fields.
        self.authorized_transaction_ids.sudo().action_capture()

    def payment_action_void(self):
        """ Void all transactions linked to this sale order. """
        payment_utils.check_rights_on_recordset(self)
        # In sudo mode because we need to be able to read on provider fields.
        self.authorized_transaction_ids.sudo().action_void()

    def get_portal_last_transaction(self):
        self.ensure_one()
        return self.transaction_ids._get_last()

    def _get_order_lines_to_report(self):
        down_payment_lines = self.order_line.filtered(lambda line:
            line.is_downpayment
            and not line.display_type
            and not line._get_downpayment_state()
        )

        def show_line(line):
            if not line.is_downpayment:
                return True
            elif line.display_type and down_payment_lines:
                return True  # Only show the down payment section if down payments were posted
            elif line in down_payment_lines:
                return True  # Only show posted down payments
            else:
                return False

        return self.order_line.filtered(show_line)

    def _get_default_payment_link_values(self):
        self.ensure_one()
        return {
            'description': self.name,
            'amount': self.amount_total - sum(self.invoice_ids.filtered(lambda x: x.state != 'cancel' and x.invoice_line_ids.sale_line_ids.order_id == self).mapped('amount_total')),
            'currency_id': self.currency_id.id,
            'partner_id': self.partner_invoice_id.id,
            'amount_max': self.amount_total,
        }

    # PORTAL #

    def _has_to_be_signed(self, include_draft=False):
        return (self.state == 'sent' or (self.state == 'draft' and include_draft)) and not self.is_expired and self.require_signature and not self.signature

    def _has_to_be_paid(self, include_draft=False):
        transaction = self.get_portal_last_transaction()
        return (self.state == 'sent' or (self.state == 'draft' and include_draft)) and not self.is_expired and self.require_payment and transaction.state != 'done' and self.amount_total

    def _get_portal_return_action(self):
        """ Return the action used to display orders when returning from customer portal. """
        self.ensure_one()
        return self.env.ref('sale.action_quotations_with_onboarding')

    def _get_report_base_filename(self):
        self.ensure_one()
        return '%s %s' % (self.type_name, self.name)

    #=== CORE METHODS OVERRIDES ===#

    @api.model
    def get_empty_list_help(self, help_msg):
        self = self.with_context(
            empty_list_help_document_name=_("sale order"),
        )
        return super().get_empty_list_help(help_msg)

    def _compute_field_value(self, field):
        if field.name != 'invoice_status' or self.env.context.get('mail_activity_automation_skip'):
            return super()._compute_field_value(field)

        filtered_self = self.filtered(
            lambda so: so.ids
                and (so.user_id or so.partner_id.user_id)
                and so._origin.invoice_status != 'upselling')
        super()._compute_field_value(field)

        upselling_orders = filtered_self.filtered(lambda so: so.invoice_status == 'upselling')
        upselling_orders._create_upsell_activity()

    def name_get(self):
        if self._context.get('sale_show_partner_name'):
            res = []
            for order in self:
                name = order.name
                if order.partner_id.name:
                    name = '%s - %s' % (name, order.partner_id.name)
                res.append((order.id, name))
            return res
        return super().name_get()

    #=== BUSINESS METHODS ===#

    def _create_upsell_activity(self):
        if not self:
            return

        self.activity_unlink(['sale.mail_act_sale_upsell'])
        for order in self:
            order_ref = order._get_html_link()
            customer_ref = order.partner_id._get_html_link()
            order.activity_schedule(
                'sale.mail_act_sale_upsell',
                user_id=order.user_id.id or order.partner_id.user_id.id,
                note=_("Upsell %(order)s for customer %(customer)s", order=order_ref, customer=customer_ref))

    def _prepare_analytic_account_data(self, prefix=None):
        """ Prepare SO analytic account creation values.

        :param str prefix: The prefix of the to-be-created analytic account name
        :return: `account.analytic.account` creation values
        :rtype: dict
        """
        self.ensure_one()
        name = self.name
        if prefix:
            name = prefix + ": " + self.name
        plan = self.env['account.analytic.plan'].sudo().search([
            '|', ('company_id', '=', self.company_id.id), ('company_id', '=', False)
        ], limit=1)
        if not plan:
            plan = self.env['account.analytic.plan'].sudo().create({
                'name': 'Default',
                'company_id': self.company_id.id,
            })
        return {
            'name': name,
            'code': self.client_order_ref,
            'company_id': self.company_id.id,
            'plan_id': plan.id,
            'partner_id': self.partner_id.id,
        }

    def _create_analytic_account(self, prefix=None):
        """ Create a new analytic account for the given orders.

        :param str prefix: if specified, the account name will be '<prefix>: <so_reference>'.
            If not, the account name will be the Sales Order reference.
        :return: None
        """
        for order in self:
            analytic = self.env['account.analytic.account'].create(order._prepare_analytic_account_data(prefix))
            order.analytic_account_id = analytic

    def _prepare_down_payment_section_line(self, **optional_values):
        """ Prepare the values to create a new down payment section.

        :param dict optional_values: any parameter that should be added to the returned down payment section
        :return: `account.move.line` creation values
        :rtype: dict
        """
        self.ensure_one()
        context = {'lang': self.partner_id.lang}
        down_payments_section_line = {
            'display_type': 'line_section',
            'name': _("Down Payments"),
            'product_id': False,
            'product_uom_id': False,
            'quantity': 0,
            'discount': 0,
            'price_unit': 0,
            'account_id': False,
            **optional_values
        }
        del context
        return down_payments_section_line

    #=== HOOKS ===#

    def add_option_to_order_with_taxcloud(self):
        self.ensure_one()

    def validate_taxes_on_sales_order(self):
        # Override for correct taxcloud computation
        # when using coupon and delivery
        return True

    #=== TOOLING ===#

    def _get_lang(self):
        self.ensure_one()

        if self.partner_id.lang and not self.partner_id.is_public:
            return self.partner_id.lang

        return self.env.lang

```

## File: models\sale_order_line.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict
from datetime import timedelta

from odoo import api, fields, models, _
from odoo.exceptions import UserError
from odoo.fields import Command
from odoo.osv import expression
from odoo.tools import float_is_zero, float_compare, float_round


class SaleOrderLine(models.Model):
    _name = 'sale.order.line'
    _inherit = 'analytic.mixin'
    _description = "Sales Order Line"
    _rec_names_search = ['name', 'order_id.name']
    _order = 'order_id, sequence, id'
    _check_company_auto = True

    _sql_constraints = [
        ('accountable_required_fields',
            "CHECK(display_type IS NOT NULL OR (product_id IS NOT NULL AND product_uom IS NOT NULL))",
            "Missing required fields on accountable sale order line."),
        ('non_accountable_null_fields',
            "CHECK(display_type IS NULL OR (product_id IS NULL AND price_unit = 0 AND product_uom_qty = 0 AND product_uom IS NULL AND customer_lead = 0))",
            "Forbidden values on non-accountable sale order line"),
    ]

    # Fields are ordered according by tech & business logics
    # and computed fields are defined after their dependencies.
    # This reduces execution stacks depth when precomputing fields
    # on record creation (and is also a good ordering logic imho)

    order_id = fields.Many2one(
        comodel_name='sale.order',
        string="Order Reference",
        required=True, ondelete='cascade', index=True, copy=False)
    sequence = fields.Integer(string="Sequence", default=10)

    # Order-related fields
    company_id = fields.Many2one(
        related='order_id.company_id',
        store=True, index=True, precompute=True)
    currency_id = fields.Many2one(
        related='order_id.currency_id',
        depends=['order_id.currency_id'],
        store=True, precompute=True)
    order_partner_id = fields.Many2one(
        related='order_id.partner_id',
        string="Customer",
        store=True, index=True, precompute=True)
    salesman_id = fields.Many2one(
        related='order_id.user_id',
        string="Salesperson",
        store=True, precompute=True)
    state = fields.Selection(
        related='order_id.state',
        string="Order Status",
        copy=False, store=True, precompute=True)

    # Fields specifying custom line logic
    display_type = fields.Selection(
        selection=[
            ('line_section', "Section"),
            ('line_note', "Note"),
        ],
        default=False)
    is_downpayment = fields.Boolean(
        string="Is a down payment",
        help="Down payments are made when creating invoices from a sales order."
            " They are not copied when duplicating a sales order.")
    is_expense = fields.Boolean(
        string="Is expense",
        help="Is true if the sales order line comes from an expense or a vendor bills")

    # Generic configuration fields
    product_id = fields.Many2one(
        comodel_name='product.product',
        string="Product",
        change_default=True, ondelete='restrict', check_company=True, index='btree_not_null',
        domain="[('sale_ok', '=', True), '|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    product_template_id = fields.Many2one(
        string="Product Template",
        comodel_name='product.template',
        compute='_compute_product_template_id',
        readonly=False,
        search='_search_product_template_id',
        # previously related='product_id.product_tmpl_id'
        # not anymore since the field must be considered editable for product configurator logic
        # without modifying the related product_id when updated.
        domain=[('sale_ok', '=', True)])
    product_uom_category_id = fields.Many2one(related='product_id.uom_id.category_id', depends=['product_id'])

    product_custom_attribute_value_ids = fields.One2many(
        comodel_name='product.attribute.custom.value', inverse_name='sale_order_line_id',
        string="Custom Values",
        compute='_compute_custom_attribute_values',
        store=True, readonly=False, precompute=True, copy=True)
    # M2M holding the values of product.attribute with create_variant field set to 'no_variant'
    # It allows keeping track of the extra_price associated to those attribute values and add them to the SO line description
    product_no_variant_attribute_value_ids = fields.Many2many(
        comodel_name='product.template.attribute.value',
        string="Extra Values",
        compute='_compute_no_variant_attribute_values',
        store=True, readonly=False, precompute=True, ondelete='restrict')

    name = fields.Text(
        string="Description",
        compute='_compute_name',
        store=True, readonly=False, required=True, precompute=True)

    product_uom_qty = fields.Float(
        string="Quantity",
        compute='_compute_product_uom_qty',
        digits='Product Unit of Measure', default=1.0,
        store=True, readonly=False, required=True, precompute=True)
    product_uom = fields.Many2one(
        comodel_name='uom.uom',
        string="Unit of Measure",
        compute='_compute_product_uom',
        store=True, readonly=False, precompute=True, ondelete='restrict',
        domain="[('category_id', '=', product_uom_category_id)]")

    # Pricing fields
    tax_id = fields.Many2many(
        comodel_name='account.tax',
        string="Taxes",
        compute='_compute_tax_id',
        store=True, readonly=False, precompute=True,
        context={'active_test': False},
        check_company=True)

    # Tech field caching pricelist rule used for price & discount computation
    pricelist_item_id = fields.Many2one(
        comodel_name='product.pricelist.item',
        compute='_compute_pricelist_item_id')

    price_unit = fields.Float(
        string="Unit Price",
        compute='_compute_price_unit',
        digits='Product Price',
        store=True, readonly=False, required=True, precompute=True)

    discount = fields.Float(
        string="Discount (%)",
        compute='_compute_discount',
        digits='Discount',
        store=True, readonly=False, precompute=True)

    # The price_reduce field should not be used for amounts computations
    # because of its digits precision. It will be removed in next version.
    price_reduce = fields.Float(
        string="Price Reduce",
        compute='_compute_price_reduce',
        digits='Product Price',
        store=True, precompute=True)
    price_subtotal = fields.Monetary(
        string="Subtotal",
        compute='_compute_amount',
        store=True, precompute=True)
    price_tax = fields.Float(
        string="Total Tax",
        compute='_compute_amount',
        store=True, precompute=True)
    price_total = fields.Monetary(
        string="Total",
        compute='_compute_amount',
        store=True, precompute=True)
    price_reduce_taxexcl = fields.Monetary(
        string="Price Reduce Tax excl",
        compute='_compute_price_reduce_taxexcl',
        store=True, precompute=True)
    price_reduce_taxinc = fields.Monetary(
        string="Price Reduce Tax incl",
        compute='_compute_price_reduce_taxinc',
        store=True, precompute=True)

    # Logistics/Delivery fields
    product_packaging_id = fields.Many2one(
        comodel_name='product.packaging',
        string="Packaging",
        compute='_compute_product_packaging_id',
        store=True, readonly=False, precompute=True,
        domain="[('sales', '=', True), ('product_id','=',product_id)]",
        check_company=True)
    product_packaging_qty = fields.Float(
        string="Packaging Quantity",
        compute='_compute_product_packaging_qty',
        store=True, readonly=False, precompute=True)

    customer_lead = fields.Float(
        string="Lead Time",
        compute='_compute_customer_lead',
        store=True, readonly=False, required=True, precompute=True,
        help="Number of days between the order confirmation and the shipping of the products to the customer")

    qty_delivered_method = fields.Selection(
        selection=[
            ('manual', "Manual"),
            ('analytic', "Analytic From Expenses"),
        ],
        string="Method to update delivered qty",
        compute='_compute_qty_delivered_method',
        store=True, precompute=True,
        help="According to product configuration, the delivered quantity can be automatically computed by mechanism :\n"
             "  - Manual: the quantity is set manually on the line\n"
             "  - Analytic From expenses: the quantity is the quantity sum from posted expenses\n"
             "  - Timesheet: the quantity is the sum of hours recorded on tasks linked to this sale line\n"
             "  - Stock Moves: the quantity comes from confirmed pickings\n")
    qty_delivered = fields.Float(
        string="Delivery Quantity",
        compute='_compute_qty_delivered',
        default=0.0,
        digits='Product Unit of Measure',
        store=True, readonly=False, copy=False)

    # Analytic & Invoicing fields
    qty_invoiced = fields.Float(
        string="Invoiced Quantity",
        compute='_compute_qty_invoiced',
        digits='Product Unit of Measure',
        store=True)
    qty_to_invoice = fields.Float(
        string="Quantity To Invoice",
        compute='_compute_qty_to_invoice',
        digits='Product Unit of Measure',
        store=True)

    analytic_line_ids = fields.One2many(
        comodel_name='account.analytic.line', inverse_name='so_line',
        string="Analytic lines")

    invoice_lines = fields.Many2many(
        comodel_name='account.move.line',
        relation='sale_order_line_invoice_rel', column1='order_line_id', column2='invoice_line_id',
        string="Invoice Lines",
        copy=False)
    invoice_status = fields.Selection(
        selection=[
            ('upselling', "Upselling Opportunity"),
            ('invoiced', "Fully Invoiced"),
            ('to invoice', "To Invoice"),
            ('no', "Nothing to Invoice"),
        ],
        string="Invoice Status",
        compute='_compute_invoice_status',
        store=True)

    untaxed_amount_invoiced = fields.Monetary(
        string="Untaxed Invoiced Amount",
        compute='_compute_untaxed_amount_invoiced',
        store=True)
    untaxed_amount_to_invoice = fields.Monetary(
        string="Untaxed Amount To Invoice",
        compute='_compute_untaxed_amount_to_invoice',
        store=True)

    # Technical computed fields for UX purposes (hide/make fields readonly, ...)
    product_type = fields.Selection(related='product_id.detailed_type', depends=['product_id'])
    product_updatable = fields.Boolean(
        string="Can Edit Product",
        compute='_compute_product_updatable')
    product_uom_readonly = fields.Boolean(
        compute='_compute_product_uom_readonly')

    #=== COMPUTE METHODS ===#

    @api.depends('product_id')
    def _compute_product_template_id(self):
        for line in self:
            line.product_template_id = line.product_id.product_tmpl_id

    def _search_product_template_id(self, operator, value):
        return [('product_id.product_tmpl_id', operator, value)]

    @api.depends('product_id')
    def _compute_custom_attribute_values(self):
        for line in self:
            if not line.product_id:
                line.product_custom_attribute_value_ids = False
                continue
            if not line.product_custom_attribute_value_ids:
                continue
            valid_values = line.product_id.product_tmpl_id.valid_product_template_attribute_line_ids.product_template_value_ids
            # remove the is_custom values that don't belong to this template
            for pacv in line.product_custom_attribute_value_ids:
                if pacv.custom_product_template_attribute_value_id not in valid_values:
                    line.product_custom_attribute_value_ids -= pacv

    @api.depends('product_id')
    def _compute_no_variant_attribute_values(self):
        for line in self:
            if not line.product_id:
                line.product_no_variant_attribute_value_ids = False
                continue
            if not line.product_no_variant_attribute_value_ids:
                continue
            valid_values = line.product_id.product_tmpl_id.valid_product_template_attribute_line_ids.product_template_value_ids
            # remove the no_variant attributes that don't belong to this template
            for ptav in line.product_no_variant_attribute_value_ids:
                if ptav._origin not in valid_values:
                    line.product_no_variant_attribute_value_ids -= ptav

    @api.depends('product_id')
    def _compute_name(self):
        for line in self:
            if not line.product_id:
                continue
            lang = line.order_id._get_lang()
            if lang != self.env.lang:
                line = line.with_context(lang=lang)
            name = line._get_sale_order_line_multiline_description_sale()
            if line.is_downpayment and not line.display_type:
                context = {'lang': lang}
                dp_state = line._get_downpayment_state()
                if dp_state == 'draft':
                    name = _("%(line_description)s (Draft)", line_description=name)
                elif dp_state == 'cancel':
                    name = _("%(line_description)s (Canceled)", line_description=name)
                del context
            line.name = name

    def _get_sale_order_line_multiline_description_sale(self):
        """ Compute a default multiline description for this sales order line.

        In most cases the product description is enough but sometimes we need to append information that only
        exists on the sale order line itself.
        e.g:
        - custom attributes and attributes that don't create variants, both introduced by the "product configurator"
        - in event_sale we need to know specifically the sales order line as well as the product to generate the name:
          the product is not sufficient because we also need to know the event_id and the event_ticket_id (both which belong to the sale order line).
        """
        self.ensure_one()
        return self.product_id.get_product_multiline_description_sale() + self._get_sale_order_line_multiline_description_variants()

    def _get_sale_order_line_multiline_description_variants(self):
        """When using no_variant attributes or is_custom values, the product
        itself is not sufficient to create the description: we need to add
        information about those special attributes and values.

        :return: the description related to special variant attributes/values
        :rtype: string
        """
        if not self.product_custom_attribute_value_ids and not self.product_no_variant_attribute_value_ids:
            return ""

        name = ""

        custom_ptavs = self.product_custom_attribute_value_ids.custom_product_template_attribute_value_id
        no_variant_ptavs = self.product_no_variant_attribute_value_ids._origin

        # display the no_variant attributes, except those that are also
        # displayed by a custom (avoid duplicate description)
        for ptav in (no_variant_ptavs - custom_ptavs):
            name += "\n" + ptav.display_name

        # Sort the values according to _order settings, because it doesn't work for virtual records in onchange
        sorted_custom_ptav = self.product_custom_attribute_value_ids.custom_product_template_attribute_value_id.sorted()
        for patv in sorted_custom_ptav:
            pacv = self.product_custom_attribute_value_ids.filtered(lambda pcav: pcav.custom_product_template_attribute_value_id == patv)
            name += "\n" + pacv.display_name

        return name

    @api.depends('display_type', 'product_id', 'product_packaging_qty')
    def _compute_product_uom_qty(self):
        for line in self:
            if line.display_type:
                line.product_uom_qty = 0.0
                continue

            if not line.product_packaging_id:
                continue
            packaging_uom = line.product_packaging_id.product_uom_id
            qty_per_packaging = line.product_packaging_id.qty
            product_uom_qty = packaging_uom._compute_quantity(
                line.product_packaging_qty * qty_per_packaging, line.product_uom)
            if float_compare(product_uom_qty, line.product_uom_qty, precision_rounding=line.product_uom.rounding) != 0:
                line.product_uom_qty = product_uom_qty

    @api.depends('product_id')
    def _compute_product_uom(self):
        for line in self:
            if not line.product_uom or (line.product_id.uom_id.id != line.product_uom.id):
                line.product_uom = line.product_id.uom_id

    @api.depends('product_id', 'company_id')
    def _compute_tax_id(self):
        taxes_by_product_company = defaultdict(lambda: self.env['account.tax'])
        lines_by_company = defaultdict(lambda: self.env['sale.order.line'])
        cached_taxes = {}
        for line in self:
            lines_by_company[line.company_id] += line
        for product in self.product_id:
            for tax in product.taxes_id:
                taxes_by_product_company[(product, tax.company_id)] += tax
        for company, lines in lines_by_company.items():
            for line in lines.with_company(company):
                taxes = taxes_by_product_company[(line.product_id, company)]
                if not line.product_id or not taxes:
                    # Nothing to map
                    line.tax_id = False
                    continue
                fiscal_position = line.order_id.fiscal_position_id
                cache_key = (fiscal_position.id, company.id, tuple(taxes.ids))
                cache_key += line._get_custom_compute_tax_cache_key()
                if cache_key in cached_taxes:
                    result = cached_taxes[cache_key]
                else:
                    result = fiscal_position.map_tax(taxes)
                    cached_taxes[cache_key] = result
                # If company_id is set, always filter taxes by the company
                line.tax_id = result

    def _get_custom_compute_tax_cache_key(self):
        """Hook method to be able to set/get cached taxes while computing them"""
        return tuple()

    @api.depends('product_id', 'product_uom', 'product_uom_qty')
    def _compute_pricelist_item_id(self):
        for line in self:
            if not line.product_id or line.display_type or not line.order_id.pricelist_id:
                line.pricelist_item_id = False
            else:
                line.pricelist_item_id = line.order_id.pricelist_id._get_product_rule(
                    line.product_id,
                    line.product_uom_qty or 1.0,
                    uom=line.product_uom,
                    date=line._get_order_date(),
                )

    @api.depends('product_id', 'product_uom', 'product_uom_qty')
    def _compute_price_unit(self):
        for line in self:
            # check if there is already invoiced amount. if so, the price shouldn't change as it might have been
            # manually edited
            if line.qty_invoiced > 0 or (line.product_id.expense_policy == 'cost' and line.is_expense):
                continue
            if not line.product_uom or not line.product_id:
                line.price_unit = 0.0
            else:
                line = line.with_company(line.company_id)
                price = line._get_display_price()
                line.price_unit = line.product_id._get_tax_included_unit_price_from_price(
                    price,
                    line.currency_id or line.order_id.currency_id,
                    product_taxes=line.product_id.taxes_id.filtered(
                        lambda tax: tax.company_id == line.env.company
                    ),
                    fiscal_position=line.order_id.fiscal_position_id,
                )

    def _get_order_date(self):
        self.ensure_one()
        return self.order_id.date_order

    def _get_display_price(self):
        """Compute the displayed unit price for a given line.

        Overridden in custom flows:
        * where the price is not specified by the pricelist
        * where the discount is not specified by the pricelist

        Note: self.ensure_one()
        """
        self.ensure_one()

        pricelist_price = self._get_pricelist_price()

        if self.order_id.pricelist_id.discount_policy == 'with_discount':
            return pricelist_price

        if not self.pricelist_item_id:
            # No pricelist rule found => no discount from pricelist
            return pricelist_price

        base_price = self._get_pricelist_price_before_discount()

        # negative discounts (= surcharge) are included in the display price
        return max(base_price, pricelist_price)

    def _get_pricelist_price(self):
        """Compute the price given by the pricelist for the given line information.

        :return: the product sales price in the order currency (without taxes)
        :rtype: float
        """
        self.ensure_one()
        self.product_id.ensure_one()

        pricelist_rule = self.pricelist_item_id
        order_date = self._get_order_date() or fields.Date.today()
        product = self.product_id.with_context(**self._get_product_price_context())
        qty = self.product_uom_qty or 1.0
        uom = self.product_uom or self.product_id.uom_id
        currency = self.currency_id or self.order_id.company_id.currency_id

        price = pricelist_rule._compute_price(
            product, qty, uom, order_date, currency=currency)

        return price

    def _get_product_price_context(self):
        """Gives the context for product price computation.

        :return: additional context to consider extra prices from attributes in the base product price.
        :rtype: dict
        """
        self.ensure_one()
        res = {}

        # It is possible that a no_variant attribute is still in a variant if
        # the type of the attribute has been changed after creation.
        no_variant_attributes_price_extra = [
            ptav.price_extra for ptav in self.product_no_variant_attribute_value_ids.filtered(
                lambda ptav:
                    ptav.price_extra and
                    ptav not in self.product_id.product_template_attribute_value_ids
            )
        ]
        if no_variant_attributes_price_extra:
            res['no_variant_attributes_price_extra'] = tuple(no_variant_attributes_price_extra)

        return res

    def _get_pricelist_price_before_discount(self):
        """Compute the price used as base for the pricelist price computation.

        :return: the product sales price in the order currency (without taxes)
        :rtype: float
        """
        self.ensure_one()
        self.product_id.ensure_one()

        pricelist_rule = self.pricelist_item_id
        order_date = self._get_order_date() or fields.Date.today()
        product = self.product_id.with_context(**self._get_product_price_context())
        qty = self.product_uom_qty or 1.0
        uom = self.product_uom

        if pricelist_rule:
            pricelist_item = pricelist_rule
            if pricelist_item.pricelist_id.discount_policy == 'without_discount':
                # Find the lowest pricelist rule whose pricelist is configured
                # to show the discount to the customer.
                while pricelist_item.base == 'pricelist' and pricelist_item.base_pricelist_id.discount_policy == 'without_discount':
                    rule_id = pricelist_item.base_pricelist_id._get_product_rule(
                        product, qty, uom=uom, date=order_date)
                    pricelist_item = self.env['product.pricelist.item'].browse(rule_id)

            pricelist_rule = pricelist_item

        price = pricelist_rule._compute_base_price(
            product,
            qty,
            uom,
            order_date,
            target_currency=self.currency_id,
        )

        return price

    @api.depends('product_id', 'product_uom', 'product_uom_qty')
    def _compute_discount(self):
        for line in self:
            if not line.product_id or line.display_type:
                line.discount = 0.0

            if not (
                line.order_id.pricelist_id
                and line.order_id.pricelist_id.discount_policy == 'without_discount'
            ):
                continue

            line.discount = 0.0

            if not line.pricelist_item_id:
                # No pricelist rule was found for the product
                # therefore, the pricelist didn't apply any discount/change
                # to the existing sales price.
                continue

            line = line.with_company(line.company_id)
            pricelist_price = line._get_pricelist_price()
            base_price = line._get_pricelist_price_before_discount()

            if base_price != 0:  # Avoid division by zero
                discount = (base_price - pricelist_price) / base_price * 100
                if (discount > 0 and base_price > 0) or (discount < 0 and base_price < 0):
                    # only show negative discounts if price is negative
                    # otherwise it's a surcharge which shouldn't be shown to the customer
                    line.discount = discount

    @api.depends('price_unit', 'discount')
    def _compute_price_reduce(self):
        for line in self:
            line.price_reduce = line.price_unit * (1.0 - line.discount / 100.0)

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
            taxes=self.tax_id,
            price_unit=self.price_unit,
            quantity=self.product_uom_qty,
            discount=self.discount,
            price_subtotal=self.price_subtotal,
        )

    @api.depends('product_uom_qty', 'discount', 'price_unit', 'tax_id')
    def _compute_amount(self):
        """
        Compute the amounts of the SO line.
        """
        for line in self:
            tax_results = self.env['account.tax'].with_company(line.company_id)._compute_taxes(
                [line._convert_to_tax_base_line_dict()]
            )
            totals = list(tax_results['totals'].values())[0]
            amount_untaxed = totals['amount_untaxed']
            amount_tax = totals['amount_tax']

            line.update({
                'price_subtotal': amount_untaxed,
                'price_tax': amount_tax,
                'price_total': amount_untaxed + amount_tax,
            })

    @api.depends('price_subtotal', 'product_uom_qty')
    def _compute_price_reduce_taxexcl(self):
        for line in self:
            line.price_reduce_taxexcl = line.price_subtotal / line.product_uom_qty if line.product_uom_qty else 0.0

    @api.depends('price_total', 'product_uom_qty')
    def _compute_price_reduce_taxinc(self):
        for line in self:
            line.price_reduce_taxinc = line.price_total / line.product_uom_qty if line.product_uom_qty else 0.0

    @api.depends('product_id', 'product_uom_qty', 'product_uom')
    def _compute_product_packaging_id(self):
        for line in self:
            # remove packaging if not match the product
            if line.product_packaging_id.product_id != line.product_id:
                line.product_packaging_id = False
            # suggest biggest suitable packaging matching the SO's company
            if line.product_id and line.product_uom_qty and line.product_uom:
                suggested_packaging = line.product_id.packaging_ids\
                        .filtered(lambda p: p.sales and (p.product_id.company_id <= p.company_id <= line.company_id))\
                        ._find_suitable_product_packaging(line.product_uom_qty, line.product_uom)
                line.product_packaging_id = suggested_packaging or line.product_packaging_id

    @api.depends('product_packaging_id', 'product_uom', 'product_uom_qty')
    def _compute_product_packaging_qty(self):
        for line in self:
            if not line.product_packaging_id:
                line.product_packaging_qty = False
            else:
                packaging_uom = line.product_packaging_id.product_uom_id
                packaging_uom_qty = line.product_uom._compute_quantity(line.product_uom_qty, packaging_uom)
                line.product_packaging_qty = float_round(
                    packaging_uom_qty / line.product_packaging_id.qty,
                    precision_rounding=packaging_uom.rounding)

    # This computed default is necessary to have a clean computation inheritance
    # (cf sale_stock) instead of simply removing the default and specifying
    # the compute attribute & method in sale_stock.
    def _compute_customer_lead(self):
        self.customer_lead = 0.0

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

    @api.depends(
        'qty_delivered_method',
        'analytic_line_ids.so_line',
        'analytic_line_ids.unit_amount',
        'analytic_line_ids.product_uom_id')
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

    def _get_downpayment_state(self):
        self.ensure_one()

        if self.display_type:
            return ''

        invoice_lines = self._get_invoice_lines()
        if self.invoice_status == 'invoiced' and not invoice_lines:
            return ''
        if all(line.parent_state == 'draft' for line in invoice_lines):
            return 'draft'
        if all(line.parent_state == 'cancel' for line in invoice_lines):
            return 'cancel'

        return ''

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
            ['so_line', 'unit_amount', 'product_uom_id', 'move_line_id:count_distinct'], ['product_uom_id', 'so_line'], lazy=False
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
            # avoid counting unit_amount twice when dealing with multiple analytic lines on the same move line
            if item['move_line_id'] == 1 and item['__count'] > 1:
                qty = item['unit_amount'] / item['__count']
            else:
                qty = item['unit_amount']
            if so_line.product_uom.category_id == uom.category_id:
                qty = uom._compute_quantity(qty, so_line.product_uom, rounding_method='HALF-UP')
            result[so_line_id] += qty

        return result

    @api.depends('invoice_lines.move_id.state', 'invoice_lines.quantity')
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

    # no trigger product_id.invoice_policy to avoid retroactively changing SO
    @api.depends('qty_invoiced', 'qty_delivered', 'product_uom_qty', 'state')
    def _compute_qty_to_invoice(self):
        """
        Compute the quantity to invoice. If the invoice policy is order, the quantity to invoice is
        calculated from the ordered quantity. Otherwise, the quantity delivered is used.
        """
        for line in self:
            if line.state in ['sale', 'done'] and not line.display_type:
                if line.product_id.invoice_policy == 'order':
                    line.qty_to_invoice = line.product_uom_qty - line.qty_invoiced
                else:
                    line.qty_to_invoice = line.qty_delivered - line.qty_invoiced
            else:
                line.qty_to_invoice = 0

    @api.depends('state', 'product_uom_qty', 'qty_delivered', 'qty_to_invoice', 'qty_invoiced')
    def _compute_invoice_status(self):
        """
        Compute the invoice status of a SO line. Possible statuses:
        - no: if the SO is not in status 'sale' or 'done', we consider that there is nothing to
          invoice. This is also the default value if the conditions of no other status is met.
        - to invoice: we refer to the quantity to invoice of the line. Refer to method
          `_compute_qty_to_invoice()` for more information on how this quantity is calculated.
        - upselling: this is possible only for a product invoiced on ordered quantities for which
          we delivered more than expected. The could arise if, for example, a project took more
          time than expected but we decided not to invoice the extra cost to the client. This
          occurs only in state 'sale', so that when a SO is set to done, the upselling opportunity
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
                if invoice_line.move_id.state == 'posted' or invoice_line.move_id.payment_state == 'invoicing_legacy':
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
                        currency=line.currency_id,
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

    @api.depends('order_id.partner_id', 'product_id')
    def _compute_analytic_distribution(self):
        for line in self:
            if not line.display_type:
                distribution = line.env['account.analytic.distribution.model']._get_distribution({
                    "product_id": line.product_id.id,
                    "product_categ_id": line.product_id.categ_id.id,
                    "partner_id": line.order_id.partner_id.id,
                    "partner_category_id": line.order_id.partner_id.category_id.ids,
                    "company_id": line.company_id.id,
                })
                line.analytic_distribution = distribution or line.analytic_distribution

    @api.depends('product_id', 'state', 'qty_invoiced', 'qty_delivered')
    def _compute_product_updatable(self):
        for line in self:
            if line.state in ['done', 'cancel'] or (line.state == 'sale' and (line.qty_invoiced > 0 or line.qty_delivered > 0)):
                line.product_updatable = False
            else:
                line.product_updatable = True

    @api.depends('state')
    def _compute_product_uom_readonly(self):
        for line in self:
            # line.ids checks whether it's a new record not yet saved
            line.product_uom_readonly = line.ids and line.state in ['sale', 'done', 'cancel']

    #=== CONSTRAINT METHODS ===#

    #=== ONCHANGE METHODS ===#

    @api.onchange('product_id')
    def _onchange_product_id_warning(self):
        if not self.product_id:
            return

        product = self.product_id
        if product.sale_line_warn != 'no-message':
            if product.sale_line_warn == 'block':
                self.product_id = False

            return {
                'warning': {
                    'title': _("Warning for %s", product.name),
                    'message': product.sale_line_warn_msg,
                }
            }

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

    #=== CRUD METHODS ===#
    def _add_precomputed_values(self, vals_list):
        """ In case an editable precomputed field is provided in the create values
        without being rounded, we have to 'manually' round it otherwise it won't be,
        because those field values are kept 'as is'.

        This is a temporary fix until the problem is fixed in the ORM.
        """
        for vals in vals_list:
            for fname in ('discount', 'product_uom_qty'):
                if fname in vals:
                    vals[fname] = self._fields[fname].convert_to_cache(vals[fname], self)
        return super()._add_precomputed_values(vals_list)

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            if vals.get('display_type') or self.default_get(['display_type']).get('display_type'):
                vals['product_uom_qty'] = 0.0

        lines = super().create(vals_list)
        for line in lines:
            if line.product_id and line.state == 'sale':
                msg = _("Extra line with %s", line.product_id.display_name)
                line.order_id.message_post(body=msg)
                # create an analytic account if at least an expense product
                if line.product_id.expense_policy not in [False, 'no'] and not line.order_id.analytic_account_id:
                    line.order_id._create_analytic_account()
        return lines

    def write(self, values):
        if 'display_type' in values and self.filtered(lambda line: line.display_type != values.get('display_type')):
            raise UserError(_("You cannot change the type of a sale order line. Instead you should delete the current line and create a new line of the proper type."))

        if 'product_uom_qty' in values:
            precision = self.env['decimal.precision'].precision_get('Product Unit of Measure')
            self.filtered(
                lambda r: r.state == 'sale' and float_compare(r.product_uom_qty, values['product_uom_qty'], precision_digits=precision) != 0)._update_line_quantity(values)

        # Prevent writing on a locked SO.
        protected_fields = self._get_protected_fields()
        if 'done' in self.mapped('state') and any(f in values.keys() for f in protected_fields):
            protected_fields_modified = list(set(protected_fields) & set(values.keys()))

            if 'name' in protected_fields_modified and all(self.mapped('is_downpayment')):
                protected_fields_modified.remove('name')

            fields = self.env['ir.model.fields'].sudo().search([
                ('name', 'in', protected_fields_modified), ('model', '=', self._name)
            ])
            if fields:
                raise UserError(
                    _('It is forbidden to modify the following fields in a locked order:\n%s')
                    % '\n'.join(fields.mapped('field_description'))
                )

        result = super().write(values)

        # Don't recompute the package_id if we are setting the quantity of the items and the quantity of packages
        if 'product_uom_qty' in values and 'product_packaging_qty' in values and 'product_packaging_id' not in values:
            self.env.remove_to_compute(self._fields['product_packaging_id'], self)

        return result

    def _get_protected_fields(self):
        """ Give the fields that should not be modified on a locked SO.

        :returns: list of field names
        :rtype: list
        """
        return [
            'product_id', 'name', 'price_unit', 'product_uom', 'product_uom_qty',
            'tax_id', 'analytic_distribution'
        ]

    def _update_line_quantity(self, values):
        orders = self.mapped('order_id')
        for order in orders:
            order_lines = self.filtered(lambda x: x.order_id == order)
            msg = "<b>" + _("The ordered quantity has been updated.") + "</b><ul>"
            for line in order_lines:
                if 'product_id' in values and values['product_id'] != line.product_id.id:
                    # tracking is meaningless if the product is changed as well.
                    continue
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

    def _check_line_unlink(self):
        """ Check whether given lines can be deleted or not.

        * Lines cannot be deleted if the order is confirmed.
        * Down payment lines who have not yet been invoiced bypass that exception.
        * Sections and Notes can always be deleted.

        :returns: Sales Order Lines that cannot be deleted
        :rtype: `sale.order.line` recordset
        """
        return self.filtered(
            lambda line:
                line.state in ('sale', 'done')
                and (line.invoice_lines or not line.is_downpayment)
                and not line.display_type
        )

    @api.ondelete(at_uninstall=False)
    def _unlink_except_confirmed(self):
        if self._check_line_unlink():
            raise UserError(_("You can not remove an order line once the sales order is confirmed.\nYou should rather set the quantity to 0."))

    #=== BUSINESS METHODS ===#

    def _expected_date(self):
        self.ensure_one()
        if self.state in ['sale', 'done'] and self.order_id.date_order:
            order_date = self.order_id.date_order
        else:
            order_date = fields.Datetime.now()
        return order_date + timedelta(days=self.customer_lead or 0.0)

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
        """Prepare the values to create the new invoice line for a sales order line.

        :param optional_values: any parameter that should be added to the returned invoice line
        :rtype: dict
        """
        self.ensure_one()
        res = {
            'display_type': self.display_type or 'product',
            'sequence': self.sequence,
            'name': self.name,
            'product_id': self.product_id.id,
            'product_uom_id': self.product_uom.id,
            'quantity': self.qty_to_invoice,
            'discount': self.discount,
            'price_unit': self.price_unit,
            'tax_ids': [Command.set(self.tax_id.ids)],
            'sale_line_ids': [Command.link(self.id)],
            'is_downpayment': self.is_downpayment,
        }
        analytic_account_id = self.order_id.analytic_account_id.id
        if self.analytic_distribution and not self.display_type:
            res['analytic_distribution'] = self.analytic_distribution
        if analytic_account_id and not self.display_type:
            analytic_account_id = str(analytic_account_id)
            if 'analytic_distribution' in res:
                res['analytic_distribution'][analytic_account_id] = res['analytic_distribution'].get(analytic_account_id, 0) + 100
            else:
                res['analytic_distribution'] = {analytic_account_id: 100}
        if optional_values:
            res.update(optional_values)
        if self.display_type:
            res['account_id'] = False
        return res

    def _prepare_procurement_values(self, group_id=False):
        """ Prepare specific key for moves or other components that will be created from a stock rule
        coming from a sale order line. This method could be override in order to add other custom key that could
        be used in move/po creation.
        """
        return {}

    def _validate_analytic_distribution(self):
        for line in self.filtered(lambda l: not l.display_type and l.state in ['draft', 'sent']):
            line._validate_distribution(**{
                'product': line.product_id.id,
                'business_domain': 'sale_order',
                'company_id': line.company_id.id,
            })

    def _get_downpayment_line_price_unit(self, invoices):
        return sum(
            l.price_unit if l.move_id.move_type == 'out_invoice' else -l.price_unit
            for l in self.invoice_lines
            if l.move_id.state == 'posted' and l.move_id not in invoices  # don't recompute with the final invoice
        )

    #=== CORE METHODS OVERRIDES ===#

    def name_get(self):
        result = []
        for so_line in self.sudo():
            name = '%s - %s' % (so_line.order_id.name, so_line.name and so_line.name.split('\n')[0] or so_line.product_id.name)
            if so_line.order_partner_id.ref:
                name = '%s (%s)' % (name, so_line.order_partner_id.ref)
            result.append((so_line.id, name))
        return result

    #=== HOOKS ===#

    def _is_delivery(self):
        self.ensure_one()
        return False

    def _is_not_sellable_line(self):
        # True if the line is a computed line (reward, delivery, ...) that user cannot add manually
        return False

    def _sellable_lines_domain(self):
        return [('is_downpayment', '=', False)]

```

## File: models\utm_campaign.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models

class UtmCampaign(models.Model):
    _inherit = 'utm.campaign'
    _description = 'UTM Campaign'

    quotation_count = fields.Integer('Quotation Count',
        compute="_compute_quotation_count", compute_sudo=True, groups='sales_team.group_sale_salesman')
    invoiced_amount = fields.Integer(string="Revenues generated by the campaign",
        compute="_compute_sale_invoiced_amount", compute_sudo=True, groups='sales_team.group_sale_salesman')
    company_id = fields.Many2one('res.company', string='Company', readonly=True, states={'draft': [('readonly', False)], 'refused': [('readonly', False)]}, default=lambda self: self.env.company)
    currency_id = fields.Many2one('res.currency', related='company_id.currency_id', string='Currency')

    def _compute_quotation_count(self):
        quotation_data = self.env['sale.order']._read_group([
            ('campaign_id', 'in', self.ids)],
            ['campaign_id'], ['campaign_id'])
        data_map = {datum['campaign_id'][0]: datum['campaign_id_count'] for datum in quotation_data}
        for campaign in self:
            campaign.quotation_count = data_map.get(campaign.id, 0)

    def _compute_sale_invoiced_amount(self):
        self.env['account.move.line'].flush_model(['balance', 'move_id', 'account_id', 'display_type'])
        self.env['account.move'].flush_model(['state', 'campaign_id', 'move_type'])
        query = """SELECT move.campaign_id, -SUM(line.balance) as price_subtotal
                    FROM account_move_line line
                    INNER JOIN account_move move ON line.move_id = move.id
                    WHERE move.state not in ('draft', 'cancel')
                        AND move.campaign_id IN %s
                        AND move.move_type IN ('out_invoice', 'out_refund', 'in_invoice', 'in_refund', 'out_receipt', 'in_receipt')
                        AND line.account_id IS NOT NULL
                        AND line.display_type = 'product'
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
from . import payment_provider
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
        if 'product.template' in self.env.registry.populated_models:
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

## File: report\account_invoice_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class AccountInvoiceReport(models.Model):
    _inherit = 'account.invoice.report'

    team_id = fields.Many2one(comodel_name='crm.team', string="Sales Team")

    def _select(self):
        return super()._select() + ", move.team_id as team_id"

```

## File: report\account_invoice_report_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_account_invoice_report_search_inherit" model="ir.ui.view">
        <field name="name">account.invoice.report.search.inherit</field>
        <field name="model">account.invoice.report</field>
        <field name="inherit_id" ref="account.view_account_invoice_report_search"/>
        <field name="arch" type="xml">
            <filter name="user" position="after">
                <filter string="Sales Team" name="sales_channel" context="{'group_by':'team_id'}"/>
            </filter>
            <field name="invoice_user_id" position="after">
                <field name="team_id"/>
            </field>
        </field>
    </record>

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

## File: report\ir_actions_report.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

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

</odoo>

```

## File: report\ir_actions_report_templates.xml

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
            <p t-if="doc.partner_id.vat"><t t-out="doc.company_id.account_fiscal_country_id.vat_label or 'Tax ID'"/>: <span t-field="doc.partner_id.vat"/></p>
        </t>
        <t t-if="doc.partner_shipping_id == doc.partner_invoice_id
                             and doc.partner_invoice_id != doc.partner_id
                             or doc.partner_shipping_id != doc.partner_invoice_id">
            <t t-set="information_block">
                <strong>
                    <t t-if="doc.partner_shipping_id == doc.partner_invoice_id">
                        Invoicing and Shipping Address:
                    </t>
                    <t t-else="">
                        Invoicing Address:
                    </t>
                </strong>
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

            <h2 class="mt-4">
                <span t-if="env.context.get('proforma', False) or is_pro_forma">Pro-Forma Invoice # </span>
                <span t-elif="doc.state in ['draft','sent']">Quotation # </span>
                <span t-else="">Order # </span>
                <span t-field="doc.name"/>
            </h2>

            <div class="row mt-4 mb-4" id="informations">
                <div t-if="doc.client_order_ref" class="col-auto col-3 mw-100 mb-2" name="informations_reference">
                    <strong>Your Reference:</strong>
                    <p class="m-0" t-field="doc.client_order_ref"/>
                </div>
                <div t-if="doc.date_order" class="col-auto col-3 mw-100 mb-2" name="informations_date">
                    <strong t-if="doc.state in ['draft', 'sent']">Quotation Date:</strong>
                    <strong t-else="">Order Date:</strong>
                    <p class="m-0" t-field="doc.date_order" t-options='{"widget": "date"}'/>
                </div>
                <div t-if="doc.validity_date and doc.state in ['draft', 'sent']"
                    class="col-auto col-3 mw-100 mb-2"
                    name="expiration_date">
                    <strong>Expiration:</strong>
                    <p class="m-0" t-field="doc.validity_date"/>
                </div>
                <div t-if="doc.user_id.name" class="col-auto col-3 mw-100 mb-2">
                    <strong>Salesperson:</strong>
                    <p class="m-0" t-field="doc.user_id"/>
                </div>
            </div>

            <!-- Is there a discount on at least one line? -->
            <t t-set="lines_to_report" t-value="doc._get_order_lines_to_report()"/>
            <t t-set="display_discount" t-value="any(l.discount for l in lines_to_report)"/>

            <table class="table table-sm o_main_table table-borderless mt-4">
                <!-- In case we want to repeat the header, remove "display: table-row-group" -->
                <thead style="display: table-row-group">
                    <tr>
                        <th name="th_description" class="text-start">Description</th>
                        <th name="th_quantity" class="text-end">Quantity</th>
                        <th name="th_priceunit" class="text-end">Unit Price</th>
                        <th name="th_discount" t-if="display_discount" class="text-end">
                            <span>Disc.%</span>
                        </th>
                        <th name="th_taxes" class="text-end">Taxes</th>
                        <th name="th_subtotal" class="text-end">
                            <span groups="account.group_show_line_subtotals_tax_excluded">Amount</span>
                            <span groups="account.group_show_line_subtotals_tax_included">Total Price</span>
                        </th>
                    </tr>
                </thead>
                <tbody class="sale_tbody">

                    <t t-set="current_subtotal" t-value="0"/>

                    <t t-foreach="lines_to_report" t-as="line">

                        <t t-set="current_subtotal" t-value="current_subtotal + line.price_subtotal" groups="account.group_show_line_subtotals_tax_excluded"/>
                        <t t-set="current_subtotal" t-value="current_subtotal + line.price_total" groups="account.group_show_line_subtotals_tax_included"/>

                        <tr t-att-class="'bg-200 fw-bold o_line_section' if line.display_type == 'line_section' else 'fst-italic o_line_note' if line.display_type == 'line_note' else ''">
                            <t t-if="not line.display_type">
                                <td name="td_name"><span t-field="line.name"/></td>
                                <td name="td_quantity" class="text-end">
                                    <span t-field="line.product_uom_qty"/>
                                    <span t-field="line.product_uom"/>
                                </td>
                                <td name="td_priceunit" class="text-end">
                                    <span t-field="line.price_unit"/>
                                </td>
                                <td t-if="display_discount" class="text-end">
                                    <span t-field="line.discount"/>
                                </td>
                                <t t-set="taxes" t-value="', '.join([(tax.description or tax.name) for tax in line.tax_id])"/>
                                <td name="td_taxes" t-attf-class="text-end {{ 'text-nowrap' if len(taxes) &lt; 10 else '' }}">
                                    <span t-out="taxes">Tax 15%</span>
                                </td>
                                <td t-if="not line.is_downpayment" name="td_subtotal" class="text-end o_price_total">
                                    <span t-field="line.price_subtotal" groups="account.group_show_line_subtotals_tax_excluded"/>
                                    <span t-field="line.price_total" groups="account.group_show_line_subtotals_tax_included"/>
                                </td>
                            </t>
                            <t t-elif="line.display_type == 'line_section'">
                                <td name="td_section_line" colspan="99">
                                    <span t-field="line.name"/>
                                </td>
                                <t t-set="current_section" t-value="line"/>
                                <t t-set="current_subtotal" t-value="0"/>
                            </t>
                            <t t-elif="line.display_type == 'line_note'">
                                <td name="td_note_line" colspan="99">
                                    <span t-field="line.name"/>
                                </td>
                            </t>
                        </tr>

                        <t t-if="current_section and (line_last or lines_to_report[line_index+1].display_type == 'line_section') and not line.is_downpayment">
                            <tr class="is-subtotal text-end">
                                <td name="td_section_subtotal" colspan="99">
                                    <strong class="mr16">Subtotal</strong>
                                    <span
                                        t-out="current_subtotal"
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
                    <div t-attf-class="#{'col-6' if report_type != 'html' else 'col-sm-7 col-md-6'} ms-auto">
                        <table class="table table-sm table-borderless">
                            <!-- Tax totals -->
                            <t t-set="tax_totals" t-value="doc.tax_totals"/>
                            <t t-call="account.document_tax_totals"/>
                        </table>
                    </div>
                </div>
            </div>

            <div t-if="doc.signature" class="mt-4 ml64 mr4" name="signature">
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

            <div>
                <p t-field="doc.note" name="order_note"/>
                <p t-if="not is_html_empty(doc.payment_term_id.note)">
                    <span t-field="doc.payment_term_id.note"/>
                </p>
                <p t-if="doc.fiscal_position_id and not is_html_empty(doc.fiscal_position_id.sudo().note)"
                    id="fiscal_position_remark">
                    <strong>Fiscal Position Remark:</strong>
                    <span t-field="doc.fiscal_position_id.sudo().note"/>
                </p>
            </div>
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

## File: report\sale_report.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class SaleReport(models.Model):
    _name = "sale.report"
    _description = "Sales Analysis Report"
    _auto = False
    _rec_name = 'date'
    _order = 'date desc'

    @api.model
    def _get_done_states(self):
        return ['sale', 'done']

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

    discount = fields.Float('Discount %', readonly=True, group_operator="avg")
    discount_amount = fields.Float('Discount Amount', readonly=True)
    campaign_id = fields.Many2one('utm.campaign', 'Campaign', readonly=True)
    medium_id = fields.Many2one('utm.medium', 'Medium', readonly=True)
    source_id = fields.Many2one('utm.source', 'Source', readonly=True)

    order_id = fields.Many2one('sale.order', 'Order #', readonly=True)

    def _with_sale(self):
        return ""

    def _select_sale(self):
        select_ = f"""
            MIN(l.id) AS id,
            l.product_id AS product_id,
            t.uom_id AS product_uom,
            CASE WHEN l.product_id IS NOT NULL THEN SUM(l.product_uom_qty / u.factor * u2.factor) ELSE 0 END AS product_uom_qty,
            CASE WHEN l.product_id IS NOT NULL THEN SUM(l.qty_delivered / u.factor * u2.factor) ELSE 0 END AS qty_delivered,
            CASE WHEN l.product_id IS NOT NULL THEN SUM((l.product_uom_qty - l.qty_delivered) / u.factor * u2.factor) ELSE 0 END AS qty_to_deliver,
            CASE WHEN l.product_id IS NOT NULL THEN SUM(l.qty_invoiced / u.factor * u2.factor) ELSE 0 END AS qty_invoiced,
            CASE WHEN l.product_id IS NOT NULL THEN SUM(l.qty_to_invoice / u.factor * u2.factor) ELSE 0 END AS qty_to_invoice,
            CASE WHEN l.product_id IS NOT NULL THEN SUM(l.price_total
                / {self._case_value_or_one('s.currency_rate')}
                * {self._case_value_or_one('currency_table.rate')}
                ) ELSE 0
            END AS price_total,
            CASE WHEN l.product_id IS NOT NULL THEN SUM(l.price_subtotal
                / {self._case_value_or_one('s.currency_rate')}
                * {self._case_value_or_one('currency_table.rate')}
                ) ELSE 0
            END AS price_subtotal,
            CASE WHEN l.product_id IS NOT NULL THEN SUM(l.untaxed_amount_to_invoice
                / {self._case_value_or_one('s.currency_rate')}
                * {self._case_value_or_one('currency_table.rate')}
                ) ELSE 0
            END AS untaxed_amount_to_invoice,
            CASE WHEN l.product_id IS NOT NULL THEN SUM(l.untaxed_amount_invoiced
                / {self._case_value_or_one('s.currency_rate')}
                * {self._case_value_or_one('currency_table.rate')}
                ) ELSE 0
            END AS untaxed_amount_invoiced,
            COUNT(*) AS nbr,
            s.name AS name,
            s.date_order AS date,
            s.state AS state,
            s.partner_id AS partner_id,
            s.user_id AS user_id,
            s.company_id AS company_id,
            s.campaign_id AS campaign_id,
            s.medium_id AS medium_id,
            s.source_id AS source_id,
            t.categ_id AS categ_id,
            s.pricelist_id AS pricelist_id,
            s.analytic_account_id AS analytic_account_id,
            s.team_id AS team_id,
            p.product_tmpl_id,
            partner.country_id AS country_id,
            partner.industry_id AS industry_id,
            partner.commercial_partner_id AS commercial_partner_id,
            CASE WHEN l.product_id IS NOT NULL THEN SUM(p.weight * l.product_uom_qty / u.factor * u2.factor) ELSE 0 END AS weight,
            CASE WHEN l.product_id IS NOT NULL THEN SUM(p.volume * l.product_uom_qty / u.factor * u2.factor) ELSE 0 END AS volume,
            l.discount AS discount,
            CASE WHEN l.product_id IS NOT NULL THEN SUM(l.price_unit * l.product_uom_qty * l.discount / 100.0
                / {self._case_value_or_one('s.currency_rate')}
                * {self._case_value_or_one('currency_table.rate')}
                ) ELSE 0
            END AS discount_amount,
            s.id AS order_id"""

        additional_fields_info = self._select_additional_fields()
        template = """,
            %s AS %s"""
        for fname, query_info in additional_fields_info.items():
            select_ += template % (query_info, fname)

        return select_

    def _case_value_or_one(self, value):
        return f"""CASE COALESCE({value}, 0) WHEN 0 THEN 1.0 ELSE {value} END"""

    def _select_additional_fields(self):
        """Hook to return additional fields SQL specification for select part of the table query.

        :returns: mapping field -> SQL computation of field, will be converted to '_ AS _field' in the final table definition
        :rtype: dict
        """
        return {}

    def _from_sale(self):
        return """
            sale_order_line l
            LEFT JOIN sale_order s ON s.id=l.order_id
            JOIN res_partner partner ON s.partner_id = partner.id
            LEFT JOIN product_product p ON l.product_id=p.id
            LEFT JOIN product_template t ON p.product_tmpl_id=t.id
            LEFT JOIN uom_uom u ON u.id=l.product_uom
            LEFT JOIN uom_uom u2 ON u2.id=t.uom_id
            JOIN {currency_table} ON currency_table.company_id = s.company_id
            """.format(
            currency_table=self.env['res.currency']._get_query_currency_table(
                {
                    'multi_company': True,
                    'date': {'date_to': fields.Date.today()}
                }),
            )

    def _where_sale(self):
        return """
            l.display_type IS NULL"""

    def _group_by_sale(self):
        return """
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
            s.id,
            currency_table.rate"""

    def _query(self):
        with_ = self._with_sale()
        return f"""
            {"WITH" + with_ + "(" if with_ else ""}
            SELECT {self._select_sale()}
            FROM {self._from_sale()}
            WHERE {self._where_sale()}
            GROUP BY {self._group_by_sale()}
            {")" if with_ else ""}
        """

    @property
    def _table_query(self):
        return self._query()

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
                <filter string="Quotations" name="Quotations" domain="[('state','in', ('draft', 'sent'))]"/>
                <filter string="Sales Orders" name="Sales" domain="[('state','not in',('draft', 'cancel', 'sent'))]"/>
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
                    <filter string="Status" name="status" context="{'group_by':'state'}"/>
                    <filter string="Company" name="company" groups="base.group_multi_company" context="{'group_by':'company_id'}"/>
                    <separator/>
                    <filter string="Order Date" name="date" context="{'group_by':'date'}"
                            invisible="context.get('sale_report_view_hide_date')"/>
                    <filter string="Order Date" name="group_by_date_day" context="{'group_by':'date:day'}"
                            invisible="not context.get('sale_report_view_hide_date')"/>
                </group>
            </search>
        </field>
    </record>

    <record id="action_order_report_all" model="ir.actions.act_window">
        <field name="name">Sales Analysis</field>
        <field name="res_model">sale.report</field>
        <field name="view_mode">graph,pivot,tree</field>
        <field name="view_id"></field>  <!-- force empty -->
        <field name="search_view_id" ref="view_order_product_search"/>
        <field name="domain">[('state', '!=', 'cancel')]</field>
        <field name="context">{'search_default_Sales':1, 'group_by_no_leaf':1,'group_by':[], 'search_default_filter_order_date': 1}</field>
        <field name="help">This report performs analysis on your quotations and sales orders. Analysis check your sales revenues and sort it by different group criteria (salesman, partner, product, etc.) Use this report to perform analysis on sales not having invoiced yet. If you want to analyse your turnover, you should use the Invoice Analysis report in the Accounting application.</field>
    </record>

    <record id="report_all_channels_sales_action" model="ir.actions.act_window">
        <field name="name">Sales Analysis</field>
        <field name="res_model">sale.report</field>
        <field name="view_mode">pivot,tree,graph</field>
    </record>

    <record id="action_order_report_quotation_salesteam" model="ir.actions.act_window">
        <field name="name">Quotations Analysis</field>
        <field name="res_model">sale.report</field>
        <field name="view_mode">graph,tree</field>
        <field name="domain">[('state','=','draft'),('team_id', '=', active_id)]</field>
        <field name="context">{'search_default_order_month':1}</field>
        <field name="help">This report performs analysis on your quotations. Analysis check your sales revenues and sort it by different group criteria (salesman, partner, product, etc.) Use this report to perform analysis on sales not having invoiced yet. If you want to analyse your turnover, you should use the Invoice Analysis report in the Accounting application.</field>
    </record>

    <record id="action_order_report_so_salesteam" model="ir.actions.act_window">
        <field name="name">Sales Analysis</field>
        <field name="res_model">sale.report</field>
        <field name="view_mode">graph,tree</field>
        <field name="domain">[('state','not in',('draft','cancel'))]</field>
        <field name="context">{
            'search_default_Sales': 1,
            'search_default_filter_date': 1,
            'search_default_team_id': [active_id]}</field>
        <field name="help">This report performs analysis on your sales orders. Analysis check your sales revenues and sort it by different group criteria (salesman, partner, product, etc.) Use this report to perform analysis on sales not having invoiced yet. If you want to analyse your turnover, you should use the Invoice Analysis report in the Accounting application.</field>
    </record>
</odoo>

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_invoice_report
from . import sale_report

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_account_account_salesman,account_account salesman,account.model_account_account,sales_team.group_sale_salesman,1,0,0,0
access_account_analytic_account_salesman,account_analytic_account salesman,analytic.model_account_analytic_account,sales_team.group_sale_salesman,1,1,1,0
access_account_account_tag_sale_salesman,account.account.tag.sale.salesman,account.model_account_account_tag,sales_team.group_sale_salesman,1,0,0,0
access_account_invoice_send_salesman,access.account.invoice.send.salesman,account.model_account_invoice_send,sales_team.group_sale_salesman,1,1,1,0
access_sale_account_journal,account.journal sale order.user,account.model_account_journal,sales_team.group_sale_salesman,1,0,0,0
access_account_move_salesman,account_move salesman,account.model_account_move,sales_team.group_sale_salesman,1,0,0,0
access_account_move_line_salesman,account_move_line salesman,account.model_account_move_line,sales_team.group_sale_salesman,1,0,0,0
access_account_partial_reconcile_salesman,account_partial_reconcile salesman,account.model_account_partial_reconcile,sales_team.group_sale_salesman,1,0,0,0
access_account_payment_term_salesman,account_payment_term salesman,account.model_account_payment_term,sales_team.group_sale_salesman,1,0,0,0
access_account_tax_user,account.tax.user,account.model_account_tax,sales_team.group_sale_salesman,1,0,0,0
access_account_tax_group_sale_manager,account.tax.group sale manager,account.model_account_tax_group,sales_team.group_sale_salesman,1,0,0,0

access_res_partner_sale_user,res.partner.sale.user,base.model_res_partner,sales_team.group_sale_salesman,1,0,0,0
access_res_partner_sale_manager,res.partner.sale.manager,base.model_res_partner,sales_team.group_sale_manager,1,1,1,0

access_mail_activity_type_sale_manager,mail.activity.type.sale.manager,mail.model_mail_activity_type,sales_team.group_sale_manager,1,1,1,1

access_product_category_sale_manager,product.category salemanager,product.model_product_category,sales_team.group_sale_manager,1,1,1,1
access_product_pricelist_sale_user,product.pricelist.sale.user,product.model_product_pricelist,sales_team.group_sale_salesman,1,0,0,0
access_product_pricelist_sale_manager,product.pricelist salemanager,product.model_product_pricelist,sales_team.group_sale_manager,1,1,1,1
access_product_pricelist_item_sale_manager,product.pricelist.item salemanager,product.model_product_pricelist_item,sales_team.group_sale_manager,1,1,1,1
access_product_attribute_sale_manager,product.attribute manager,product.model_product_attribute,sales_team.group_sale_manager,1,1,1,1
access_product_attribute_value_sale_manager,product.attribute manager value,product.model_product_attribute_value,sales_team.group_sale_manager,1,1,1,1
access_product_product_attribute_sale_manager,product.template.attribute manager value,product.model_product_template_attribute_value,sales_team.group_sale_manager,1,1,1,1
access_product_product_attribute_custom_value_sale_user,product.attribute.custom value salesman,product.model_product_attribute_custom_value,sales_team.group_sale_salesman,1,1,1,1
access_product_supplierinfo_user,product.supplierinfo.user,product.model_product_supplierinfo,sales_team.group_sale_salesman,1,0,0,0
access_product_supplierinfo_sale_manager,product.supplierinfo salemanager,product.model_product_supplierinfo,sales_team.group_sale_manager,1,1,1,1
access_product_template_attribute_exclusion_sale_manager,product.attribute manager filter line,product.model_product_template_attribute_exclusion,sales_team.group_sale_manager,1,1,1,1
access_product_template_attribute_line_sale_manager,product.attribute manager line,product.model_product_template_attribute_line,sales_team.group_sale_manager,1,1,1,1
access_product_tag_sale_manager,product.tag.sale.manager,product.model_product_tag,sales_team.group_sale_manager,1,1,1,1
access_product_product_sale_user,product.product sale use,product.model_product_product,sales_team.group_sale_salesman,1,0,0,0
access_product_product_sale_manager,product.product salemanager,model_product_product,sales_team.group_sale_manager,1,1,1,1
access_product_template_sale_user,product.template sale use,product.model_product_template,sales_team.group_sale_salesman,1,0,0,0
access_product_template_sale_manager,product.template salemanager,model_product_template,sales_team.group_sale_manager,1,1,1,1

access_uom_uom_user,uom.uom.user,uom.model_uom_uom,sales_team.group_sale_salesman,1,0,0,0
access_uom_category_sale_manager,uom.category salemanager,uom.model_uom_category,sales_team.group_sale_manager,1,1,1,1
access_uom_uom_sale_manager,uom.uom salemanager,uom.model_uom_uom,sales_team.group_sale_manager,1,1,1,1

access_sale_order_portal,sale.order.portal,sale.model_sale_order,base.group_portal,1,0,0,0
access_sale_order_readonly,sale.order.accountant,model_sale_order,account.group_account_readonly,1,0,0,0
access_sale_order_invoicing_payments,sale.order,model_sale_order,account.group_account_invoice,1,1,0,0
access_sale_order_accountant,sale.order.accountant,model_sale_order,account.group_account_user,1,1,0,0
access_sale_order,sale.order,model_sale_order,sales_team.group_sale_salesman,1,1,1,0
access_sale_order_manager,sale.order.manager,model_sale_order,sales_team.group_sale_manager,1,1,1,1

access_sale_order_line_portal,sale.order.line.portal,sale.model_sale_order_line,base.group_portal,1,0,0,0
access_sale_order_line_readonly,sale.order.line accountant,model_sale_order_line,account.group_account_readonly,1,0,0,0
access_sale_order_line_invoicing_payments,sale.order.line,model_sale_order_line,account.group_account_invoice,1,1,0,0
access_sale_order_line_accountant,sale.order.line accountant,model_sale_order_line,account.group_account_user,1,1,0,0
access_sale_order_line,sale.order.line,model_sale_order_line,sales_team.group_sale_salesman,1,1,1,1

access_sale_report_salesman,sale.report,model_sale_report,sales_team.group_sale_salesman,1,0,0,0

access_payment_link_wizard_sale,access.payment.link.wizard.sale,payment.model_payment_link_wizard,sales_team.group_sale_salesman,1,1,1,0
access_sale_payment_provider_onboarding_wizard,access.sale.payment.provider.onboarding.wizard,model_sale_payment_provider_onboarding_wizard,base.group_system,1,1,1,0
access_sale_advance_payment_inv,access.sale.advance.payment.inv,model_sale_advance_payment_inv,sales_team.group_sale_salesman,1,1,1,0
access_sale_order_cancel,access.sale.order.cancel,model_sale_order_cancel,sales_team.group_sale_salesman,1,1,1,0

```

## File: security\ir_rules.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <!-- Multi - Company Rules -->
    <record id="sale_order_comp_rule" model="ir.rule">
        <field name="name">Sales Order multi-company</field>
        <field name="model_id" ref="model_sale_order"/>
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>

    <record id="sale_order_line_comp_rule" model="ir.rule">
        <field name="name">Sales Order Line multi-company</field>
        <field name="model_id" ref="model_sale_order_line"/>
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>

    <record id="sale_order_report_comp_rule" model="ir.rule">
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

    <!-- Wizard access rules -->
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

    <record id="sale_payment_provider_onboarding_wizard_rule" model="ir.rule">
        <field name="name">Payment Provider Onboarding Wizard Rule</field>
        <field name="model_id" ref="model_sale_payment_provider_onboarding_wizard"/>
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

</odoo>

```

## File: security\res_groups.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

     <record id="group_auto_done_setting" model="res.groups">
        <field name="name">Lock Confirmed Sales</field>
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
         * @param {string} code - The code of the selected payment option's provider
         * @param {number} paymentOptionId - The id of the selected payment option
         * @param {string} flow - The online payment flow of the selected payment option
         * @return {object} The extended transaction route params
         */
        _prepareTransactionRouteParams: function (code, paymentOptionId, flow) {
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

## File: static\src\js\product_discount_field.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { useService } from "@web/core/utils/hooks";
import { FloatField } from "@web/views/fields/float/float_field";
import { _lt } from "@web/core/l10n/translation";
import { ConfirmationDialog } from "@web/core/confirmation_dialog/confirmation_dialog";

 /**
    * Dialog called if user changes a value in the sale order line.
    * The wizard will open only if
    *  (1) Sale order line is 3 or more
    *  (2) First sale order line is changed
    *  (3) value is the same in all other sale order line
 */

export class ProductDiscountField extends FloatField {
    setup() {
        super.setup();
        this.dialogService = useService("dialog");
    }

    onChange(ev) {
        if (!("order_line" in this.props.record.model.root.data)) {
            return;
        }
        const x2mList = this.props.record.model.root.data.order_line;
        const orderLines = x2mList.records.filter(line => !line.data.display_type);

        if (orderLines.length < 3) {
            return;
        }

        const isFirstOrderLine = this.props.record.data.id === orderLines[0].data.id;
        if (isFirstOrderLine && sameValue(orderLines)) {
            this.dialogService.add(ConfirmationDialog, {
                body: _lt("Do you want to apply this value to all lines ?"),
                confirm: () => {
                    const commands = orderLines.slice(1).map((line) => {
                        return {
                            operation: "UPDATE",
                            record: line,
                            data: {["discount"]: this.props.value},
                        };
                    });

                    x2mList.applyCommands('order_line', commands);
                },
            });
        }
    }
}

// TODO remove this function, no need to export it anymore.
export function sameValue(orderLines) {
    const compareValue = orderLines[1].data.discount;
    return orderLines.slice(1).every(line => line.data.discount === compareValue);
}


ProductDiscountField.components = { ConfirmationDialog };
ProductDiscountField.template = "sale.ProductDiscountField";
ProductDiscountField.displayName = _lt("Disc.%");

registry.category("fields").add("sol_discount", ProductDiscountField)

```

## File: static\src\js\sale_portal.js

```javascript
/** @odoo-module */

import publicWidget from 'web.public.widget';
import "portal.portal"; // force dependencies

publicWidget.registry.PortalHomeCounters.include({
    /**
     * @override
     */
    _getCountersAlwaysDisplayed() {
        return this._super(...arguments).concat(['quotation_count', 'order_count']);
    },
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
        // After signature, automatically open the popup for payment
        if ($.bbq.getState('allow_payment') === 'yes' && this.$('#o_sale_portal_paynow').length) {
            this.el.querySelector('#o_sale_portal_paynow').click();
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

## File: static\src\js\sale_product_field.js

```javascript
/** @odoo-module **/

import { registry } from '@web/core/registry';
import { Many2OneField } from '@web/views/fields/many2one/many2one_field';
import { useEffect } from '@odoo/owl';

export class SaleOrderLineProductField extends Many2OneField {

    setup() {
        super.setup();
        let isMounted = false;
        let isInternalUpdate = false;
        const super_update = this.update;
        this.update = (recordlist) => {
            isInternalUpdate = true;
            super_update(recordlist);
        };
        if (this.props.canQuickCreate) {
            this.quickCreate = (name) => {
                isInternalUpdate = true;
                return this.props.update([false, name]);
            };
        }
        useEffect(value => {
            if (!isMounted) {
                isMounted = true;
            } else if (value && isInternalUpdate) {
                // we don't want to trigger product update when update comes from an external sources,
                // such as an onchange, or the product configuration dialog itself
                if (this.props.relation === 'product.template') {
                    this._onProductTemplateUpdate();
                } else {
                    this._onProductUpdate();
                }
            }
            isInternalUpdate = false;
        }, () => [Array.isArray(this.value) && this.value[0]]);
    }

    get isProductClickable() {
        // product form should be accessible if the widget field is readonly
        // or if the line cannot be edited (e.g. locked SO)
        return (
            this.props.record.isReadonly(this.props.name)
            || this.props.record.model.root.isReadonly
            && this.props.record.model.root.activeFields.order_line
            && this.props.record.model.root.isReadonly('order_line')
        )
    }
    get hasExternalButton() {
        // Keep external button, even if field is specified as 'no_open' so that the user is not
        // redirected to the product when clicking on the field content
        const res = super.hasExternalButton;
        return res || (!!this.props.value && !this.state.isFloating);
    }
    get hasConfigurationButton() {
        return this.isConfigurableLine || this.isConfigurableTemplate;
    }
    get isConfigurableLine() { return false; }
    get isConfigurableTemplate() { return false; }

    get configurationButtonHelp() {
        return this.env._t("Edit Configuration");
    }

    get configurationButtonIcon() {
        return 'btn btn-secondary fa ' + this.configurationButtonFAIcon();
    }

    configurationButtonFAIcon() {
        return 'fa-pencil';
    }

    onClick(ev) {
        // Override to get internal link to products in SOL that cannot be edited
        if (this.props.readonly) {
            ev.stopPropagation();
            this.openAction();
        }
        else {
            super.onClick(ev);
        }
    }

    async _onProductTemplateUpdate() { }
    async _onProductUpdate() { } // event_booth_sale, event_sale, sale_renting

    onEditConfiguration() {
        if (this.isConfigurableLine) {
            this._editLineConfiguration();
        } else {
            this._editProductConfiguration();
        }
    }
    _editLineConfiguration() { } // event_booth_sale, event_sale, sale_renting
    _editProductConfiguration() { } // sale_product_configurator, sale_product_matrix

}

SaleOrderLineProductField.template = "sale.SaleProductField";

registry.category("fields").add("sol_product_many2one", SaleOrderLineProductField);

```

## File: static\src\js\sale_progressbar_field.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { useService } from "@web/core/utils/hooks";
import { KanbanProgressBarField } from "@web/views/fields/progress_bar/kanban_progress_bar_field";

const { useEffect } = owl;

/**
 * A custom Component for the view of sales teams on the kanban view in the CRM app.
 *
 * The wanted behavior is to show a progress bar when an invoicing target is defined or show
 * a link redirecting to the record's form view otherwise.
 */
export class SaleProgressBarField extends KanbanProgressBarField {
    /**
     * Anything used by the component is defined on the setup method.
     */
    setup() {
        super.setup();

        this.actionService = useService("action");
        this.orm = useService("orm");

        useEffect(() => {
            this.state.isInvoicingTargetDefined = this.props.record.data[this.props.maxValueField];
        });
    }

    /**
     * Display the form view of the record on click.
     */
    async defineInvoicingTarget() {
        const { resId, resModel } = this.props.record;
        const action = await this.orm.call(resModel, "get_formview_action", [[resId]]);
        this.actionService.doAction(action, { props: { mode: "edit" } });
    }
}

/**
 * Define the template name used on the component.
 */
SaleProgressBarField.template = "sale.SaleProgressBarField";

registry.category("fields").add("sales_team_progressbar", SaleProgressBarField);

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
        'click .o_variant_pills': '_onChangePillsAttribute',
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
        if(!$parent.length){
            return Promise.resolve();
        }
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
                    ...this._getOptionalCombinationInfoParam($currentOptionalProduct),
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
            ...this._getOptionalCombinationInfoParam($parent),
        }).then((combinationData) => {
            if (this._shouldIgnoreRpcResult()) {
                return;
            }
            this._onChangeCombination(ev, $parent, combinationData);
            this._checkExclusions($parent, combination, combinationData.parent_exclusions);
        });
    },

    /**
     * Hook to add optional info to the combination info call.
     *
     * @param {$.Element} $product
     */
    _getOptionalCombinationInfoParam($product) {
        return {};
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
        $container.find('ul[data-attribute_exclusions]').trigger('change');
        $container.find('input.js_variant_change:checked, select.js_variant_change').each(function () {
            VariantMixin.handleCustomValues($(this));
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
        productId = parseInt(productId);
        productTemplateId = parseInt(productTemplateId);
        var productReady = Promise.resolve();
        if (productId) {
            productReady = Promise.resolve(productId);
        } else {
            var params = {
                product_template_id: productTemplateId,
                product_template_attribute_value_ids:
                    JSON.stringify(VariantMixin.getSelectedVariantValues($container)),
            };

            var route = '/sale/create_product_variant';
            if (useAjax) {
                productReady = ajax.jsonRpc(route, 'call', params);
            } else if (Boolean(this._rpc)) {
                // HACK to combine owl and non owl calls
                productReady = this._rpc({route: route, params: params});
            } else {
                productReady = this.rpc(route, params);
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
        // combination exclusions: array of array of ptav
        // for example a product with 3 variation and one specific variation is disabled (archived)
        //  requires the first 2 to be selected for the third to be disabled
        if (combinationData.archived_combinations) {
            combinationData.archived_combinations.forEach((excludedCombination) => {
                const ptavCommon = excludedCombination.filter((ptav) => combination.includes(ptav));
                if (
                    !!ptavCommon
                    && (combination.length === excludedCombination.length)
                    && (ptavCommon.length === combination.length)
                ) {
                    // Selected combination is archived, all attributes must be disabled from each other
                    combination.forEach((ptav) => {
                        combination.forEach((ptavOther) => {
                            if (ptav === ptavOther) {
                                return;
                            }
                            self._disableInput(
                                $parent,
                                ptav,
                                ptavOther,
                                combinationData.mapped_attribute_names,
                            );
                        })
                    })
                } else if (
                    !!ptavCommon
                    && (combination.length === excludedCombination.length)
                    && (ptavCommon.length === (combination.length - 1))
                ) {
                    // In this case we only need to disable the remaining ptav
                    const disabledPtav = excludedCombination.find((ptav) => !combination.includes(ptav));
                    excludedCombination.forEach((ptav) => {
                        if (ptav === disabledPtav) {
                            return;
                        }
                        self._disableInput(
                            $parent,
                            disabledPtav,
                            ptav,
                            combinationData.mapped_attribute_names,
                        )
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

        if (combination.has_discounted_price && !combination.compare_list_price) {
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

    _onChangePillsAttribute: function (ev) {
        const radio = ev.target.closest('.o_variant_pills').querySelector("input");
        radio.click();  // Trigger onChangeVariant.
        var $parent = $(ev.target).closest('.js_product');
        $parent.find('.o_variant_pills')
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

const { markup } = owl;

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
    skip_trigger: 'a[data-method=action_open_base_onboarding_company].o_onboarding_step_action__done',
}, {
    trigger: 'input[id=street]',
    content: _t("Complete your company's data"),
    position: "bottom",
    skip_trigger: 'a[data-method=action_open_base_onboarding_company].o_onboarding_step_action__done',
}, {
    trigger: ".modal-content button[name='action_save_onboarding_company_step']",
    content: _t("Looks good. Let's continue."),
    position: "left",
    skip_trigger: 'a[data-method=action_open_base_onboarding_company].o_onboarding_step_action__done',
}, {
    trigger: 'a.o_onboarding_step_action.btn[data-method=action_open_base_document_layout]',
    extra_trigger: ".o_sale_order",
    content: _t("Customize your quotes and orders."),
    position: "bottom",
    skip_trigger: 'a[data-method=action_open_base_document_layout].o_onboarding_step_action__done',
}, {
    trigger: "button[name='document_layout_save']",
    extra_trigger: ".o_sale_order",
    content: _t("Good job, let's continue."),
    position: "top", // dot NOT move to bottom, it would cause a resize flicker
    skip_trigger: 'a[data-method=action_open_base_document_layout].o_onboarding_step_action__done',
}, {
    trigger: 'a.o_onboarding_step_action.btn[data-method=action_open_sale_onboarding_payment_provider]',
    extra_trigger: ".o_sale_order",
    content: _t("To speed up order confirmation, we can activate electronic signatures or payments."),
    position: "bottom",
    skip_trigger: 'a[data-method=action_open_sale_onboarding_payment_provider].o_onboarding_step_action__done',
}, {
    trigger: "button[name='add_payment_methods']",
    extra_trigger: ".o_sale_order",
    content: _t("Lets keep electronic signature for now."),
    position: "bottom",
    skip_trigger: 'a[data-method=action_open_sale_onboarding_payment_provider].o_onboarding_step_action__done',
}, {
    trigger: 'a.o_onboarding_step_action.btn[data-method=action_open_sale_onboarding_sample_quotation]',
    extra_trigger: ".o_sale_order",
    content: _t("Now, we'll create a sample quote."),
    position: "bottom",
}]);

tour.register("sale_quote_tour", {
        url: "/web#action=sale.action_quotations_with_onboarding&view_type=form",
        rainbowMan: true,
        rainbowManMessage: markup(_t("<b>Congratulations</b>, your first quotation is sent!<br>Check your email to validate the quote.")),
        sequence: 30,
    }, [{
        trigger: ".o_field_res_partner_many2one[name='partner_id']",
        extra_trigger: ".o_sale_order",
        content: _t("Write a company name to create one, or see suggestions."),
        position: "right",
        run: function (actions) {
            actions.text("Agrolait", this.$anchor.find("input"));
        },
    }, {
        trigger: ".ui-menu-item > a:contains('Agrolait')",
        auto: true,
        in_modal: false,
    }, {
        trigger: ".o_field_x2many_list_row_add > a",
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
        trigger: "a:contains('DESK0001')",
        auto: true,
    }, {
        trigger: ".o_field_widget[name='price_unit'] ",
        extra_trigger: ".fa-arrow-right",  // Wait for product creation
        content: Markup(_t("<b>Set a price</b>.")),
        position: "right",
        run: "text 10.0"
    },
    ...tour.stepUtils.statusbarButtonsSteps("Send by Email", Markup(_t("<b>Send the quote</b> to yourself and check what the customer will receive.")), ".o_statusbar_buttons button[name='action_quotation_send']"),
    {
        trigger: ".modal-footer button[name='action_send_mail']",
        extra_trigger: ".modal-footer button[name='action_send_mail']",
        content: _t("Let's send the quote."),
        position: "bottom",
    }]);

});

```

## File: static\src\xml\product_discount_field.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates xml:space="preserve">
    <t t-name="sale.ProductDiscountField" t-inherit="web.FloatField" t-inherit-mode="primary" owl="1">
        <xpath expr="//input" position="attributes">
            <attribute name="t-on-change">onChange</attribute>
        </xpath>
    </t>
</templates>

```

## File: static\src\xml\sales_team_progress_bar_template.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates xml:space="preserve">

    <t t-name="sale.SaleProgressBarField" owl="1">
        <t t-if="state.isInvoicingTargetDefined">
            <t t-call="web.ProgressBarField"/>
        </t>
        <t t-else="">
            <!-- TODO is this class needed here ? -->
            <a t-on-click.prevent="defineInvoicingTarget" href="#" class="sale_progressbar_form_link">
                Click to define an invoicing target
            </a>
        </t>
    </t>

</templates>

```

## File: static\src\xml\sale_product_field.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates xml:space="preserve">

    <t t-name="sale.SaleProductField" t-inherit="web.Many2OneField" t-inherit-mode="primary" owl="1">
        <!-- Make the product label clickable (to open its form view) when the user cannot
            access it through the external button (because the product/line is readonly) -->
        <xpath expr="//t[@t-if='!props.canOpen']" position="attributes">
            <attribute name="t-if">
                !isProductClickable
            </attribute>
        </xpath>
        <!-- Show configuration button for custom lines/products -->
        <xpath expr="//t[@t-if='hasExternalButton']" position="before">
            <t t-if="hasConfigurationButton">
                <button
                    type="button"
                    t-att-class="configurationButtonIcon"
                    tabindex="-1"
                    draggable="false"
                    t-att-aria-label="configurationButtonHelp"
                    t-att-data-tooltip="configurationButtonHelp"
                    t-on-click="onEditConfiguration"/>
            </t>
        </xpath>
    </t>

</templates>

```

## File: views\account_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <record id="account_invoice_groupby_inherit" model="ir.ui.view">
        <field name="name">account.move.groupby</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_account_invoice_filter"/>
        <field name="arch" type="xml">
            <field name="invoice_user_id" position="after">
                <field name="team_id"/>
            </field>
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

    <record model="ir.ui.view" id="account_invoice_form">
        <field name="name">Account Invoice</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_move_form"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='sale_info_group']/field[@name='invoice_user_id']" position="after">
                <field name="team_id" kanban_view_ref="%(sales_team.crm_team_view_kanban)s"/>
            </xpath>
            <xpath expr="//group[@id='other_tab_group']" position="inside">
                <group string="Marketing"
                        name="utm_link"
                        groups="base.group_no_one"
                        attrs="{'invisible': [('move_type', 'not in', ('out_invoice', 'out_refund'))]}">
                    <field name="campaign_id" options="{'create_name_field': 'title', 'always_reload': True}"/>
                    <field name="medium_id"/>
                    <field name="source_id"/>
                </group>
            </xpath>
            <xpath expr="//div[@name='button_box']" position="inside">
                <button class="oe_stat_button"
                        name="action_view_source_sale_orders"
                        type="object"
                        icon="fa-pencil-square-o"
                        attrs="{'invisible': ['|', ('sale_order_count', '=', 0), ('move_type', 'not in', ('out_invoice', 'out_refund', 'in_invoice', 'in_refund', 'out_receipt', 'in_receipt'))]}">
                    <field string="Sale Orders" name="sale_order_count" widget="statinfo"/>
                </button>
            </xpath>
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

</odoo>

```

## File: views\crm_team_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- Dedicated actions used in views below -->
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

    <!-- crm.team views (& actions) -->

    <record id="crm_team_salesteams_view_form" model="ir.ui.view">
        <field name="name">crm.team.form</field>
        <field name="model">crm.team</field>
        <field name="priority">9</field>
        <field name="inherit_id" ref="sales_team.crm_team_view_form"/>
        <field name="arch" type="xml">
            <div name="options_active" position="inside">
                <div class="o_row" style="display:inherit">
                    <field name="use_quotations"/><label for="use_quotations"/>
                </div>
            </div>
            <field name="company_id" position="after">
                <label for="invoiced_target"/>
                <div class="o_row">
                    <field name="invoiced_target" widget="monetary" options="{'currency_field': 'currency_id'}" class="oe_inline"/>
                    <span class="flex-grow-1">/ Month</span>
                </div>
            </field>
        </field>
    </record>

    <record id="crm_team_view_kanban_dashboard" model="ir.ui.view">
        <field name="name">crm.team.view.kanban.dashboard.inherit.sale</field>
        <field name="model">crm.team</field>
        <field name="inherit_id" ref="sales_team.crm_team_view_kanban_dashboard"/>
        <field name="arch" type="xml">
            <xpath expr="//templates" position="before">
                <field name="use_quotations"/>
                <field name="invoiced_target"/>
            </xpath>

            <xpath expr="//t[@name='second_options']" position="after">
                <div class="row" t-if="record.quotations_count.raw_value">
                    <div class="col">
                        <a name="%(action_quotations_salesteams)d" type="action" context="{'search_default_draft': True, 'search_default_sent': True}">
                            <field name="quotations_count" class="me-1"/>
                            <t t-if="record.quotations_count.raw_value == 1">Quotation</t>
                            <t t-else="">Quotations</t>
                        </a>
                    </div>
                    <div class="col-auto text-truncate">
                        <field name="quotations_amount" widget="monetary"/>
                    </div>
                </div>
                <div class="row" name="orders_to_invoice" t-if="record.sales_to_invoice_count.raw_value">
                    <div class="col-8">
                        <a name="%(action_orders_to_invoice_salesteams)d" type="action">
                            <field name="sales_to_invoice_count" class="me-1"/>
                            <t t-if="record.sales_to_invoice_count.raw_value == 1">Order to Invoice</t>
                            <t t-else="">Orders to Invoice</t>
                        </a>
                    </div>
                </div>
            </xpath>

            <xpath expr="//div[hasclass('o_kanban_primary_bottom')]" position="after">
                <t groups="sales_team.group_sale_manager">
                    <div class="col-12 o_kanban_primary_bottom bottom_block">
                        <field name="invoiced" widget="sales_team_progressbar" title="Invoicing" options="{'current_value': 'invoiced', 'max_value': 'invoiced_target', 'editable': true, 'edit_max_value': true, 'on_change': 'update_invoiced_target'}"/>
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

            <div name="o_team_kanban_report_separator" position="before">
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
            </div>
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

    <record id="mail_activity_type_action_config_sale" model="ir.actions.act_window">
        <field name="name">Activity Types</field>
        <field name="res_model">mail.activity.type</field>
        <field name="view_mode">tree,form</field>
        <field name="domain">['|', ('res_model', '=', False), ('res_model', '=', 'sale.order')]</field>
        <field name="context">{'default_res_model': 'sale.order'}</field>
    </record>

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

    <record id="payment_provider_form" model="ir.ui.view">
        <field name="name">payment.provider.form.inherit.sale</field>
        <field name="model">payment.provider</field>
        <field name="inherit_id" ref="payment.payment_provider_form"/>
        <field name="arch" type="xml">
            <group name="payment_form" position="inside">
                <field name="so_reference_type" attrs="{'invisible': [('code', '!=', 'custom')]}"/>
            </group>
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
            <group name="group_product" position="inside">
                <field name="sales"/>
            </group>
        </field>
    </record>

    <record id="product_packaging_tree_view_sale" model="ir.ui.view">
        <field name="name">product.packaging.tree.view.sale</field>
        <field name="model">product.packaging</field>
        <field name="inherit_id" ref="product.product_packaging_tree_view"/>
        <field name="arch" type="xml">
            <field name="product_uom_id" position="after">
                <field name="sales" optional="show"/>
            </field>
        </field>
    </record>

</odoo>

```

## File: views\product_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <record id="product_template_form_view" model="ir.ui.view">
        <field name="name">product.template.form.view.inherit.sale</field>
        <field name="model">product.template</field>
        <field name="inherit_id" ref="product.product_template_form_view"/>
        <field name="arch" type="xml">
            <page name="sales" position="attributes">
                <attribute name="invisible">0</attribute>
            </page>
            <field name="product_variant_count" position="after">
                <field name="service_type" widget="radio" invisible="True"/>
                <field name="visible_expense_policy" invisible="1"/>
            </field>
            <field name="detailed_type" position="after">
                <field name="invoice_policy" required="1"/>
                <field name="expense_policy" widget="radio" attrs="{'invisible': [('visible_expense_policy', '=', False)]}"/>
            </field>
            <field name="product_tooltip" position="after">
                <label for="product_tooltip" string=""
                    attrs="{'invisible': ['|', ('type', 'not in', ('product', 'consu')), ('invoice_policy', '!=', 'order')]}"/>
                <div attrs="{'invisible': ['|', ('type', 'not in', ('product', 'consu')), ('invoice_policy', '!=', 'order')]}"
                    class="fst-italic text-muted">
                    You can invoice them before they are delivered.
                </div>
                <label for="product_tooltip" string=""
                    attrs="{'invisible': ['|', ('type', 'not in', ('product', 'consu')), ('invoice_policy', '!=', 'delivery')]}"/>
                <div attrs="{'invisible': ['|', ('type', 'not in', ('product', 'consu')), ('invoice_policy', '!=', 'delivery')]}"
                    class="fst-italic text-muted">
                    Invoice after delivery, based on quantities delivered, not ordered.
                </div>
            </field>
        </field>
    </record>

    <!-- TODO VFE inherit product_template_form_view and factorize both views -->
    <record id="product_form_view_sale_order_button" model="ir.ui.view">
        <field name="name">product.product.sale.order</field>
        <field name="model">product.product</field>
        <field name="inherit_id" ref="product.product_normal_form_view"/>
        <field name="arch" type="xml">
            <div name="button_box" position="inside">
                <button class="oe_stat_button"
                    name="action_view_sales"
                    type="object"
                    icon="fa-signal"
                    help="Sold in the last 365 days"
                    groups="sales_team.group_sale_salesman"
                    attrs="{'invisible': [('sale_ok', '=', False)]}">
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
                <t groups="sales_team.group_sale_salesman">
                    <group string="Warning when Selling this Product" groups="sale.group_warning_sale">
                        <field name="sale_line_warn" nolabel="1" colspan="2"/>
                        <field name="sale_line_warn_msg" colspan="2" nolabel="1" placeholder="Type a message..."
                                attrs="{'required':[('sale_line_warn','!=','no-message')],'readonly':[('sale_line_warn','=','no-message')]}"/>
                    </group>
                </t>
            </group>
        </field>
    </record>

    <record id="product_template_form_view_sale_order_button" model="ir.ui.view">
        <field name="name">product.template.sale.order.button</field>
        <field name="model">product.template</field>
        <field name="inherit_id" ref="product.product_template_only_form_view"/>
        <field name="arch" type="xml">
            <div name="button_box" position="inside">
                <button class="oe_stat_button"
                    name="action_view_sales"
                    type="object" icon="fa-signal"
                    help="Sold in the last 365 days"
                    groups="sales_team.group_sale_salesman"
                    attrs="{'invisible': [('sale_ok', '=', False)]}">
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
                <t groups="sales_team.group_sale_salesman">
                    <group string="Warning when Selling this Product" groups="sale.group_warning_sale">
                        <field name="sale_line_warn" string="Warning"/>
                        <field name="sale_line_warn_msg" string="Message" placeholder="Type a message..."
                                attrs="{'required':[('sale_line_warn','!=','no-message')],'readonly':[('sale_line_warn','=','no-message')], 'invisible':[('sale_line_warn','=','no-message')]}"/>
                    </group>
                </t>
            </group>
        </field>
    </record>

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
        <field name="inherit_id" ref="base.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[hasclass('settings')]" position="inside">
                <div class="app_settings_block o_not_app"
                    string="Sales"
                    data-string="Sales"
                    data-key="sale_management"
                    groups="sales_team.group_sale_manager">
                    <h2>Product Catalog</h2>
                    <div class="row mt16 o_settings_container" name="catalog_setting_container">
                        <div class="col-12 col-lg-6 o_setting_box" id="variant_options">
                            <div class="o_setting_left_pane">
                                <field name="group_product_variant"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="group_product_variant"/>
                                <a href="https://www.odoo.com/documentation/16.0/applications/sales/sales/products_prices/products/variants.html" title="Documentation" class="o_doc_link" target="_blank"></a>
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
                            title="Boost your sales with multiple kinds of programs: Coupons, Promotions, Gift Card, Loyalty. Specific conditions can be set (products, customers, minimum purchase amount, period). Rewards can be discounts (% or amount) or free products.">
                            <div class="o_setting_left_pane">
                                <field name="module_loyalty"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="module_loyalty" string="Discounts, Loyalty &amp; Gift Card"/>
                                <div class="text-muted" id="sale_coupon">
                                    Manage Promotions, coupons, loyalty cards, Gift cards &amp; eWallet
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box" id="pricelist_configuration">
                            <div class="o_setting_left_pane">
                                <field name="group_product_pricelist"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="group_product_pricelist"/>
                                <a href="https://www.odoo.com/documentation/16.0/applications/sales/sales/products_prices/prices/pricing.html" title="Documentation" class="o_doc_link" target="_blank"></a>
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
                                <a href="https://www.odoo.com/documentation/16.0/applications/sales/sales/send_quotations/get_signature_to_validate.html" title="Documentation" class="me-2 o_doc_link" target="_blank"></a>
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
                                <a href="https://www.odoo.com/documentation/16.0/applications/sales/sales/send_quotations/get_paid_to_validate.html" title="Documentation" class="me-2 o_doc_link" target="_blank"></a>
                                <span class="fa fa-lg fa-building-o" title="Values set here are company-specific." groups="base.group_multi_company"/>
                                <div class="text-muted">
                                    Request an online payment to confirm orders
                                </div>
                                <div class="mt8" attrs="{'invisible': [('portal_confirmation_pay', '=', False)]}">
                                    <button name='%(payment.action_payment_provider)d' icon="fa-arrow-right" type="action" string="Payment Providers" class="btn-link"/>
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
                        <div class="col-12 col-lg-6 o_setting_box" id="shipping_costs_sendcloud">
                            <div class="o_setting_left_pane">
                                <field name="module_delivery_sendcloud" widget="upgrade_boolean"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="module_delivery_sendcloud"/>
                                <div class="text-muted">
                                    Compute shipping costs and ship with Sendcloud
                                </div>
                                <div class="content-group">
                                    <div id="sale_delivery_sendcloud"/>
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
                                <a href="https://www.odoo.com/documentation/16.0/applications/sales/sales/invoicing/invoicing_policy.html" title="Documentation" class="o_doc_link" target="_blank"></a>
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
                                    <label for="invoice_mail_template_id" class="o_light_label me-2"/>
                                    <field name="invoice_mail_template_id" class="oe_inline" options="{'no_create': True}"/>
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box" id="down_payments">
                            <div class="o_setting_left_pane"/>
                            <div class="o_setting_right_pane">
                                <span class="o_form_label">Down Payments</span>
                                <a href="https://www.odoo.com/documentation/16.0/applications/sales/sales/invoicing/down_payment.html" title="Documentation" class="o_doc_link" target="_blank"></a>
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
                                <a href="https://www.odoo.com/documentation/16.0/applications/sales/sales/amazon_connector/setup.html" title="Documentation" class="o_doc_link" target="_blank"></a>
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

</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0"?>
<odoo>

    <record id="act_res_partner_2_sale_order" model="ir.actions.act_window">
        <field name="name">Quotations and Sales</field>
        <field name="res_model">sale.order</field>
        <field name="view_mode">tree,kanban,form,graph</field>
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

    <record id="crm_lead_partner_kanban_view" model="ir.ui.view">
        <field name="name">res.partner.kanban.saleorder.inherit</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="base.res_partner_kanban_view"/>
        <field name="priority" eval="20"/>
        <field name="arch" type="xml">
            <field name="mobile" position="after">
                <field name="sale_order_count" groups="sales_team.group_sale_salesman"/>
            </field>
            <xpath expr="//div[hasclass('oe_kanban_bottom_left')]" position="inside">
                <a t-if="record.sale_order_count.value>0" data-type="object" data-name="action_view_sale_order"
                   groups="sales_team.group_sale_salesman"
                   href="#" class="oe_kanban_action oe_kanban_action_a me-1">
                    <span class="badge rounded-pill">
                        <i class="fa fa-fw fa-usd" role="img" aria-label="Sale orders" title="Sales orders"/>
                        <t t-out="record.sale_order_count.value"/>
                    </span>
                </a>
            </xpath>
        </field>
    </record>

    <record id="res_partner_view_buttons" model="ir.ui.view">
        <field name="name">res.partner.view.buttons</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="base.view_partner_form" />
        <field name="priority" eval="3"/>
        <field name="arch" type="xml">
            <div name="button_box" position="inside">
                <button class="oe_stat_button" type="object" name="action_view_sale_order"
                    groups="sales_team.group_sale_salesman"
                    icon="fa-usd">
                    <field string="Sales" name="sale_order_count" widget="statinfo"/>
                </button>
            </div>
            <xpath expr="//page[@name='internal_notes']//field[@name='comment']" position="after">
                <group groups="sales_team.group_sale_salesman">
                    <group groups="sale.group_warning_sale" col="2">
                        <separator string="Warning on the Sales Order" colspan="2"/>
                        <field name="sale_warn" nolabel="1" colspan="2" required="1"/>
                        <field name="sale_warn_msg" nolabel="1" string="Message" placeholder="Type a message..." colspan="2"
                                attrs="{'required':[('sale_warn','!=', False), ('sale_warn','!=','no-message')], 'invisible':[('sale_warn','in',(False,'no-message'))]}"/>
                    </group>
                </group>
            </xpath>
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

## File: views\sale_menus.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <menuitem id="sale_menu_root"
        name="Sales"
        web_icon="sale_management,static/description/icon.svg"
        active="False"
        sequence="30">

        <menuitem id="sale_order_menu"
            name="Orders"
            sequence="10">

            <menuitem id="menu_sale_quotations"
                action="action_quotations_with_onboarding"
                groups="sales_team.group_sale_salesman"
                sequence="10"/>

            <menuitem id="menu_sale_order"
                name="Orders"
                action="action_orders"
                groups="sales_team.group_sale_salesman"
                sequence="20"/>

            <menuitem id="report_sales_team"
                name="Sales Teams"
                action="sales_team.crm_team_action_sales"
                groups="sales_team.group_sale_manager"
                sequence="30"/>

            <menuitem id="res_partner_menu"
                action="account.res_partner_action_customer"
                groups="sales_team.group_sale_salesman"
                sequence="40"/>

        </menuitem>

        <menuitem id="menu_sale_invoicing"
            name="To Invoice"
            groups="sales_team.group_sale_salesman"
            sequence="20">

            <menuitem id="menu_sale_order_invoice"
                action="action_orders_to_invoice"
                sequence="10"/>

            <menuitem id="menu_sale_order_upselling"
                action="action_orders_upselling"
                sequence="20"/>

        </menuitem>

        <menuitem id="product_menu_catalog"
            name="Products"
            groups="sales_team.group_sale_salesman"
            sequence="30">

            <menuitem id="menu_product_template_action"
                action="product_template_action"
                sequence="10"/>
            <menuitem id="menu_products"
                action="product.product_normal_action_sell"
                groups="product.group_product_variant"
                sequence="20"/>
            <menuitem id="menu_product_pricelist_main"
                name="Pricelists"
                action="product.product_pricelist_action2"
                groups="product.group_product_pricelist"
                sequence="30"/>

        </menuitem>

        <menuitem id="menu_sale_report"
            name="Reporting"
            action="action_order_report_all"
            groups="sales_team.group_sale_manager"
            sequence="40"/>

        <menuitem id="menu_sale_config"
            name="Configuration"
            groups="sales_team.group_sale_manager"
            sequence="50">

            <menuitem id="menu_sale_general_settings"
                name="Settings"
                sequence="10"
                action="action_sale_config_settings"
                groups="base.group_system"/>

            <menuitem id="sales_team_config"
                name="Sales Teams"
                action="sales_team.crm_team_action_config"
                sequence="20"/>

            <menuitem id="menu_sales_config"
                sequence="30"
                name="Sales Orders">

                <menuitem id="menu_tag_config"
                    name="Tags"
                    action="sales_team.sales_team_crm_tag_action"
                    sequence="10"/>

            </menuitem>

            <menuitem id="prod_config_main"
                name="Products"
                sequence="40">

                <menuitem id="menu_product_attribute_action"
                    action="product.attribute_action"
                    groups="product.group_product_variant"
                    sequence="10"/>

            </menuitem>

            <menuitem id="next_id_16"
                name="Units of Measure"
                groups="uom.group_uom"
                sequence="50">

                <menuitem id="menu_product_uom_form_action"
                    action="uom.product_uom_form_action"
                    groups="base.group_no_one"
                    sequence="10"/>

                <menuitem id="menu_product_uom_categ_form_action"
                    action="uom.product_uom_categ_form_action"
                    sequence="20"/>

            </menuitem>

            <menuitem id="sale_menu_config_activity_type"
                action="mail_activity_type_action_config_sale"
                groups="base.group_no_one"
                sequence="60"
                />
        </menuitem>
    </menuitem>

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
            <t t-set="method" t-value="'action_open_sale_onboarding_payment_provider'" />
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
            <t t-call="sale.sale_onboarding_order_confirmation_step" name="payment_provider_step" />
            <t t-call="sale.sale_onboarding_sample_quotation_step" name="sample_quotation_step" />
        </t>
    </template>
    <!-- ORDER CONFIRMATION -->
    <record id="action_open_sale_payment_provider_onboarding_wizard" model="ir.actions.act_window">
        <field name="name">Choose how to confirm quotations</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">sale.payment.provider.onboarding.wizard</field>
        <field name="view_mode">form</field>
        <field name="view_id" ref="payment.payment_provider_onboarding_wizard_form" />
        <field name="target">new</field>
    </record>
</odoo>

```

## File: views\sale_order_line_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

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
                <filter string="To Invoice"
                    name="to_invoice"
                    domain="[('qty_to_invoice', '!=', 0)]"
                    help="Sales Order Lines ready to be invoiced"/>
                <separator/>
                <filter string="My Sales Order Lines"
                    name="my_sales_order_lines"
                    domain="[('salesman_id','=',uid)]"
                    help="Sales Order Lines related to a Sales Order of mine"/>
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

</odoo>

```

## File: views\sale_order_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- VIEWS -->

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

    <record id="view_sale_order_graph" model="ir.ui.view">
        <field name="name">sale.order.graph</field>
        <field name="model">sale.order</field>
        <field name="arch" type="xml">
            <graph string="Sales Orders" sample="1">
                <field name="partner_id"/>
                <field name="amount_total" type="measure"/>
            </graph>
        </field>
    </record>

    <record id="view_sale_order_pivot" model="ir.ui.view">
        <field name="name">sale.order.pivot</field>
        <field name="model">sale.order</field>
        <field name="arch" type="xml">
            <pivot string="Sales Orders" sample="1">
                <field name="date_order" type="row"/>
                <field name="amount_total" type="measure"/>
            </pivot>
        </field>
    </record>

    <record id="view_sale_order_kanban" model="ir.ui.view">
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
                <progressbar field="activity_state"
                    colors='{"planned": "success", "today": "warning", "overdue": "danger"}'/>
                <templates>
                    <t t-name="kanban-box">
                        <div t-attf-class="oe_kanban_card oe_kanban_global_click">
                            <div class="o_kanban_record_top mb16">
                                <div class="o_kanban_record_headings mt4">
                                    <strong class="o_kanban_record_title">
                                        <span t-out="record.partner_id.value"/>
                                    </strong>
                                </div>
                                <strong>
                                    <field name="amount_total" widget="monetary"/>
                                </strong>
                            </div>
                            <div class="o_kanban_record_bottom">
                                <div class="oe_kanban_bottom_left text-muted">
                                    <span>
                                        <t t-out="record.name.value"/> <t t-out="record.date_order.value"/>
                                    </span>
                                    <field name="activity_ids" widget="kanban_activity"/>
                                </div>
                                <div class="oe_kanban_bottom_right">
                                    <field name="state"
                                        widget="label_selection"
                                        options="{'classes': {'draft': 'default', 'cancel': 'default', 'done': 'success'}}"/>
                                </div>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <!-- TODO VFE factorize core fields between quotation & SO views -->
    <record id="view_order_tree" model="ir.ui.view">
        <field name="name">sale.order.tree</field>
        <field name="model">sale.order</field>
        <field name="priority">2</field>
        <field name="arch" type="xml">
            <tree string="Sales Orders" sample="1"
                decoration-info="invoice_status == 'to invoice'"
                decoration-muted="state == 'cancel'">
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
                <field name="invoice_status"
                    decoration-success="invoice_status == 'invoiced'"
                    decoration-info="invoice_status == 'to invoice'"
                    decoration-warning="invoice_status == 'upselling'"
                    widget="badge" optional="show"/>
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
            <tree string="Quotation" class="o_sale_order" sample="1"
                decoration-info="state in ['draft', 'sent']"
                decoration-muted="state == 'cancel'">
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
                <field name="company_id" groups="!base.group_multi_company" invisible="1"/>
                <field name="amount_untaxed" sum="Total Tax Excluded" widget="monetary" optional="hide"/>
                <field name="amount_tax" sum="Tax Total" widget="monetary" optional="hide"/>
                <field name="amount_total" sum="Total Tax Included" widget="monetary" decoration-bf="1" optional="show"/>
                <field name="state"
                    decoration-success="state == 'sale' or state == 'done'"
                    decoration-info="state == 'draft' or state == 'sent'"
                    widget="badge" optional="show"/>
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
            <form string="Sales Order" class="o_sale_order">
            <header>
                <field name="authorized_transaction_ids" invisible="1"/>
                <button name="payment_action_capture" type="object" data-hotkey="shift+g"
                        string="Capture Transaction" class="oe_highlight"
                        attrs="{'invisible': [('authorized_transaction_ids', '=', [])]}"/>
                <button name="payment_action_void" type="object"
                        string="Void Transaction" data-hotkey="shift+v"
                        confirm="Are you sure you want to void the authorized transaction? This action can't be undone."
                        attrs="{'invisible': [('authorized_transaction_ids', '=', [])]}"/>
                <button id="create_invoice" name="%(sale.action_view_sale_advance_payment_inv)d" string="Create Invoice"
                    type="action" class="btn-primary" data-hotkey="q"
                    attrs="{'invisible': [('invoice_status', '!=', 'to invoice')]}"/>
                <button id="create_invoice_percentage" name="%(sale.action_view_sale_advance_payment_inv)d" string="Create Invoice"
                    type="action" context="{'default_advance_payment_method': 'percentage'}" data-hotkey="q"
                    attrs="{'invisible': ['|',('invoice_status', '!=', 'no'), ('state', '!=', 'sale')]}"/>
                <button name="action_quotation_send" string="Send by Email" type="object" states="draft" class="btn-primary" data-hotkey="g" context="{'validate_analytic': True}"/>
                <button name="action_quotation_send" type="object" string="Send PRO-FORMA Invoice"
                    groups="sale.group_proforma_sales" class="btn-primary"
                    attrs="{'invisible': ['|', ('state', '!=', 'draft'), ('invoice_count','&gt;=',1)]}" context="{'proforma': True, 'validate_analytic': True}"/>
                <button name="action_confirm" id="action_confirm" data-hotkey="v"
                    string="Confirm" class="btn-primary" type="object" context="{'validate_analytic': True}"
                    attrs="{'invisible': [('state', 'not in', ['sent'])]}"/>
                <button name="action_confirm" data-hotkey="v"
                    string="Confirm" type="object" context="{'validate_analytic': True}"
                    attrs="{'invisible': [('state', 'not in', ['draft'])]}"/>
                <button name="action_quotation_send" type="object" string="Send PRO-FORMA Invoice" groups="sale.group_proforma_sales" attrs="{'invisible': ['|', ('state', '=', 'draft'), ('invoice_count','&gt;=',1)]}" context="{'proforma': True, 'validate_analytic': True}"/>
                <button name="action_quotation_send" string="Send by Email" type="object" states="sent,sale" data-hotkey="g" context="{'validate_analytic': True}"/>
                <button name="action_cancel" type="object" string="Cancel" attrs="{'invisible': ['|', ('state', 'not in', ['draft', 'sent','sale']), ('id', '=', False)]}" data-hotkey="z"/>
                <button name="action_draft" states="cancel" type="object" string="Set to Quotation" data-hotkey="w"/>
                <field name="state" widget="statusbar" statusbar_visible="draft,sent,sale"/>
            </header>
            <div
                 class="alert alert-warning mb-0" role="alert"
                 attrs="{'invisible': [('partner_credit_warning', '=', '')]}">
                <field name="partner_credit_warning"/>
            </div>
            <sheet>
                <div class="oe_button_box" name="button_box">
                    <button name="action_view_invoice"
                        type="object"
                        class="oe_stat_button"
                        icon="fa-pencil-square-o"
                        attrs="{'invisible': [('invoice_count', '=', 0)]}">
                        <field name="invoice_count" widget="statinfo" string="Invoices"/>
                    </button>
                    <button name="action_preview_sale_order"
                        type="object"
                        class="oe_stat_button"
                        icon="fa-globe icon">
                        <div class="o_field_widget o_stat_info">
                            <span class="o_stat_text">Customer</span>
                            <span class="o_stat_text">Preview</span>
                        </div>
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
                        <field name="partner_invoice_id" groups="account.group_delivery_invoice_address" context="{'default_type':'invoice'}" options='{"always_reload": True}'/>
                        <field name="partner_shipping_id" groups="account.group_delivery_invoice_address" context="{'default_type':'delivery'}" options='{"always_reload": True}'/>
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
                        <field name="date_order" attrs="{'invisible': [('state', 'in', ['draft', 'sent'])]}" nolabel="1"/>
                        <field name="show_update_pricelist" invisible="1"/>
                        <label for="pricelist_id" groups="product.group_product_pricelist"/>
                        <div groups="product.group_product_pricelist" class="o_row">
                            <field name="pricelist_id" options="{'no_open':True,'no_create': True}"/>
                            <button name="action_update_prices" type="object"
                                string=" Update Prices"
                                help="Recompute all prices based on this pricelist"
                                class="btn-link mb-1 px-0" icon="fa-refresh"
                                confirm="This will update all unit prices based on the currently set pricelist."
                                attrs="{'invisible': ['|', ('show_update_pricelist', '=', False), ('state', 'in', ['sale', 'done', 'cancel'])]}"/>
                        </div>
                        <field name="company_id" invisible="1"/>
                        <field name="currency_id" invisible="1"/>
                        <field name="pricelist_id" invisible="1" groups="!product.group_product_pricelist"/>
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
                                        <field name="product_type" invisible="1"/>
                                        <field name="invoice_status" invisible="1"/>
                                        <field name="qty_to_invoice" invisible="1"/>
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
                                            <field name="product_uom" invisible="1" groups="!uom.group_uom"/>
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
                                            <field name="discount" class="oe_inline"/> %
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
                                        <field name="analytic_distribution" widget="analytic_distribution"
                                           groups="analytic.group_analytic_accounting"
                                           options="{'product_field': 'product_id', 'business_domain': 'sale_order'}"/>
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
                                <field name="product_type" invisible="1"/>
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
                                    options="{
                                        'no_open': True,
                                    }"
                                    domain="[('sale_ok', '=', True), '|', ('company_id', '=', False), ('company_id', '=', parent.company_id)]"
                                    widget="sol_product_many2one"
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
                                    options="{
                                        'no_open': True,
                                    }"
                                    domain="[('sale_ok', '=', True), '|', ('company_id', '=', False), ('company_id', '=', parent.company_id)]"
                                    widget="sol_product_many2one"/>
                                <field name="name" widget="section_and_note_text" optional="show"/>
                                <field name="analytic_distribution" widget="analytic_distribution"
                                           optional="hide"
                                           groups="analytic.group_analytic_accounting"
                                           options="{'product_field': 'product_id', 'business_domain': 'sale_order'}"/>
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
                                <field name="product_uom" invisible="1" groups="!uom.group_uom"/>
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
                                <field name="discount" string="Disc.%" groups="product.group_discount_per_so_line" optional="show" widget="sol_discount"/>
                                <field name="is_downpayment" invisible="1"/>
                                <field name="price_subtotal" widget="monetary" groups="account.group_show_line_subtotals_tax_excluded" attrs="{'invisible': [('is_downpayment', '=', True)]}"/>
                                <field name="price_total" widget="monetary" groups="account.group_show_line_subtotals_tax_included" attrs="{'invisible': [('is_downpayment', '=', True)]}"/>
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
                                <field name="product_uom"/>
                                <field name="price_subtotal"/>
                                <field name="price_total"/>
                                <field name="price_tax" invisible="1"/>
                                <field name="price_total" invisible="1"/>
                                <field name="price_unit"/>
                                <field name="display_type"/>
                                <field name="tax_id" invisible="1"/>
                                <field name="company_id" invisible="1"/>
                                <field name="product_updatable" invisible="1"/>
                                <field name="qty_invoiced" invisible="1"/>
                                <field name="product_uom_readonly" invisible="1"/>
                                <templates>
                                    <t t-name="kanban-box">
                                        <div t-attf-class="oe_kanban_card oe_kanban_global_click ps-0 pe-0 {{ record.display_type.raw_value ? 'o_is_' + record.display_type.raw_value : '' }}">
                                            <t t-if="!record.display_type.raw_value">
                                                <div class="row g-0">
                                                    <div class="col-2 pe-3">
                                                        <img t-att-src="kanban_image('product.product', 'image_128', record.product_id.raw_value)" t-att-title="record.product_id.value" t-att-alt="record.product_id.value" style="max-width: 100%;"/>
                                                    </div>
                                                    <div class="col-10">
                                                        <div class="row">
                                                            <div class="col">
                                                                <strong t-out="record.product_id.value" />
                                                            </div>
                                                            <div class="col-auto">
                                                                <t t-set="line_price" t-value="record.price_subtotal.value" groups="account.group_show_line_subtotals_tax_excluded"/>
                                                                <t t-set="line_price" t-value="record.price_total.value" groups="account.group_show_line_subtotals_tax_included"/>
                                                                <strong class="float-end text-end" t-out="line_price" />
                                                            </div>
                                                        </div>
                                                        <div class="row">
                                                            <div class="col-12 text-muted">
                                                                Quantity:
                                                                <t t-out="record.product_uom_qty.value"/> <t t-out="record.product_uom.value"/>
                                                            </div>
                                                        </div>
                                                        <div class="row">
                                                            <div class="col-12 text-muted">
                                                                Unit Price:
                                                                <t t-out="record.price_unit.value"/>
                                                            </div>
                                                        </div>
                                                    </div>
                                                </div>
                                            </t>
                                            <t t-if="record.display_type.raw_value === 'line_section' || record.display_type.raw_value === 'line_note'">
                                                <div class="row">
                                                    <div class="col-12">
                                                        <t t-out="record.name.value"/>
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
                                <field  colspan="2" name="note" nolabel="1" placeholder="Terms and conditions..."/>
                            </group>
                            <group class="oe_subtotal_footer oe_right" colspan="2" name="sale_total">
                                <field name="tax_totals" widget="account-tax-totals-field" nolabel="1" colspan="2" readonly="1"/>
                            </group>
                            <div class="clearfix"/>
                        </group>
                    </page>
                    <page string="Other Info" name="other_information">
                        <group>
                            <group name="sales_person" string="Sales">
                                <field name="user_id" widget="many2one_avatar_user"/>
                                <field name="team_id" kanban_view_ref="%(sales_team.crm_team_view_kanban)s" options="{'no_create': True}"/>
                                <field name="company_id" options="{'no_create': True}" groups="base.group_multi_company"/>
                                <label for="require_signature" string="Online confirmation"/>
                                <div>
                                    <field name="require_signature" class="oe_inline"/>
                                    <span>Signature</span>
                                    <field name="require_payment" class="oe_inline ms-3"/>
                                    <span>Payment</span>
                                </div>
                                <field name="reference" readonly="1" attrs="{'invisible': [('reference', '=', False)]}"/>
                                <field name="client_order_ref"/>
                                <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color', 'no_create_edit': True}"/>
                            </group>
                            <group name="sale_info" string="Invoicing and Payments">
                                <field name="show_update_fpos" invisible="1"/>
                                <label for="fiscal_position_id"/>
                                <div class="o_row">
                                    <field name="fiscal_position_id" options="{'no_create': True}"/>
                                    <button name="action_update_taxes" type="object"
                                        string=" Update Taxes"
                                        help="Recompute all taxes based on this fiscal position"
                                        class="btn-link mb-1 px-0" icon="fa-refresh"
                                        confirm="This will update all taxes based on the currently selected fiscal position."
                                        attrs="{'invisible': ['|', ('show_update_fpos', '=', False), ('state', 'in', ['sale', 'done','cancel'])]}"/>
                                </div>
                                <field name="partner_invoice_id" groups="!account.group_delivery_invoice_address" invisible="1"/>
                                <field name="analytic_account_id" context="{'default_partner_id':partner_invoice_id, 'default_name':name}" attrs="{'readonly': [('invoice_count','!=',0),('state','=','sale')]}" groups="analytic.group_analytic_accounting" force_save="1"/>
                                <field name="invoice_status" states="sale,done" groups="base.group_no_one"/>
                                <!-- test_event_configurator -->
                                <field name="invoice_status" invisible="1" groups="!base.group_no_one"/>
                            </group>
                        </group>
                        <group>
                            <group name="sale_shipping">
                                <label for="commitment_date" string="Delivery Date"/>
                                <div name="commitment_date_div" class="o_row">
                                    <field name="commitment_date"/>
                                    <span name="expected_date_span" class="text-muted">Expected: <field name="expected_date" class="oe_inline" widget="date"/></span>
                                </div>
                            </group>
                            <group string="Tracking" name="sale_reporting">
                                <group name="technical" colspan="2" class="mb-0">
                                    <field name="origin"/>
                                </group>
                                <group name="utm_link" colspan="2" class="mt-0">
                                    <field name="campaign_id" options="{'create_name_field': 'title', 'always_reload': True}"/>
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
        <field name="arch" type="xml">
            <button name="action_draft" position="after">
                <t groups="sale.group_auto_done_setting">
                    <button name="action_done" type="object" string="Lock"
                        states="sale"
                        help="If the sale is locked, you can not modify it anymore. However, you will still be able to invoice or deliver." groups="sales_team.group_sale_manager"/>
                    <button name="action_unlock" type="object" string="Unlock"
                        states="done"
                        groups="sales_team.group_sale_manager"/>
                </t>
            </button>
        </field>
    </record>

    <record id="view_sales_order_filter" model="ir.ui.view">
        <field name="name">sale.order.list.select</field>
        <field name="model">sale.order</field>
        <field name="priority" eval="15"/>
        <field name="arch" type="xml">
            <search string="Search Sales Order">
                <field name="name" string="Order"
                    filter_domain="['|', '|', ('name', 'ilike', self), ('client_order_ref', 'ilike', self), ('partner_id', 'child_of', self)]"/>
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
            <filter name="my_sale_orders_filter" position="replace">
                <field name="campaign_id"/>
                <separator/>
                <filter string="My Quotations" name="my_quotation" domain="[('user_id', '=', uid)]"/>
                <separator/>
                <filter string="Quotations" name="draft" domain="[('state','in',('draft', 'sent'))]"/>
                <filter string="Sales Orders" name="sales" domain="[('state','in',('sale','done'))]"/>
                <separator/>
                <filter string="Create Date" name="filter_create_date" date="create_date"/>
            </filter>
        </field>
    </record>

    <record id="sale_order_view_search_inherit_sale" model="ir.ui.view">
        <field name="name">sale.order.search.inherit.sale</field>
        <field name="model">sale.order</field>
        <field name="mode">primary</field>
        <field name="inherit_id" ref="sale.view_sales_order_filter"/>
        <field name="arch" type="xml">
            <filter name="my_sale_orders_filter" position="after">
                <separator/>
                <filter string="To Invoice" name="to_invoice" domain="[('invoice_status','=','to invoice')]" />
                <filter string="To Upsell" name="upselling" domain="[('invoice_status','=','upselling')]" />
                <separator/>
                <filter string="Order Date" name="order_date" date="date_order"/>
            </filter>
        </field>
    </record>

    <!-- ACTIONS (WINDOW) -->

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

    <!-- ACTIONS (SERVER) -->

    <record id="model_sale_order_action_quotation_sent" model="ir.actions.server">
        <field name="name">Mark Quotation as Sent</field>
        <field name="model_id" ref="sale.model_sale_order"/>
        <field name="binding_model_id" ref="sale.model_sale_order"/>
        <field name="binding_view_types">form,list</field>
        <field name="state">code</field>
        <field name="code">action = records.action_quotation_sent()</field>
    </record>

    <record id="model_sale_order_action_share" model="ir.actions.server">
        <field name="name">Share</field>
        <field name="model_id" ref="sale.model_sale_order"/>
        <field name="binding_model_id" ref="sale.model_sale_order"/>
        <field name="binding_view_types">form</field>
        <field name="state">code</field>
        <field name="code">action = records.action_share()</field>
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
                <t t-out="sale_order.name"/>
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
            <t t-if="quotations" t-call="portal.portal_table">
                <thead>
                    <tr class="active">
                        <th>Quotation #</th>
                        <th class="text-end">Quotation Date</th>
                        <th class="text-end">Valid Until</th>
                        <th class="text-center"/>
                        <th class="text-end">Total</th>
                    </tr>
                </thead>
                <t t-foreach="quotations" t-as="quotation">
                    <tr>
                        <td><a t-att-href="quotation.get_portal_url()"><t t-out="quotation.name"/></a></td>
                        <td class="text-end"><span t-field="quotation.date_order"/></td>
                        <td class="text-end"><span t-field="quotation.validity_date"/></td>
                        <td class="text-center">
                            <span t-if="quotation.state == 'cancel'" class="badge rounded-pill text-bg-secondary">
                                <i class="fa fa-fw fa-remove"/> Cancelled</span>
                            <span t-if="quotation.is_expired" class="badge rounded-pill text-bg-secondary">
                                <i class="fa fa-fw fa-clock-o"/> Expired</span>
                        </td>
                        <td class="text-end">
                            <span t-field="quotation.amount_total"/>
                        </td>
                    </tr>
                </t>
            </t>
            <p t-else="">There are currently no quotations for your account.</p>
        </t>
    </template>

    <template id="portal_my_orders" name="My Sales Orders">
        <t t-call="portal.portal_layout">
            <t t-set="breadcrumbs_searchbar" t-value="True"/>

            <t t-call="portal.portal_searchbar">
                <t t-set="title">Sales Orders</t>
            </t>
            <t t-if="orders" t-call="portal.portal_table">
                <thead>
                    <tr class="active">
                        <th>
                            <span class='d-none d-md-inline'>Sales Order #</span>
                            <span class='d-block d-md-none'>Ref.</span>
                        </th>
                        <th class="text-end">Order Date</th>
                        <th class="text-center"/>
                        <th class="text-end">Total</th>
                    </tr>
                </thead>
                <t t-foreach="orders" t-as="order">
                    <tr>
                        <td><a t-att-href="order.get_portal_url()"><t t-out="order.name"/></a></td>
                        <td class="text-end">
                            <span t-field="order.date_order" t-options="{'widget': 'date'}"/>&amp;nbsp;
                            <span class='d-none d-md-inline' t-field="order.date_order" t-options="{'time_only': True}"/>
                        </td>
                        <td class="text-center">
                            <span t-if="order.state == 'done'"  class="badge rounded-pill text-bg-success">
                                <i class="fa fa-fw fa-check" role="img" aria-label="Done" title="Done"/>Done
                            </span>
                        </td>
                        <td class="text-end"><span t-field="order.amount_total"/></td>
                    </tr>
                </t>
            </t>
            <p t-else="">There are currently no orders for your account.</p>
        </t>
    </template>

    <!-- Complete page of the sale_order -->
    <template id="sale_order_portal_template" name="Sales Order Portal Template" inherit_id="portal.portal_sidebar" primary="True">
        <xpath expr="//div[hasclass('o_portal_sidebar')]" position="inside">
            <t t-set="o_portal_fullwidth_alert" groups="sales_team.group_sale_salesman">
                <!-- Uses backend_url provided in rendering values -->
                <t t-call="portal.portal_back_in_edit_mode"/>
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
                            <li class="list-group-item d-grid align-content-start">
                                <a t-if="sale_order._has_to_be_signed(True)" role="button" class="btn btn-primary mb8" data-bs-toggle="modal" data-bs-target="#modalaccept" href="#">
                                    <i class="fa fa-check"/><t t-if="sale_order._has_to_be_paid(True)"> Sign &amp; Pay</t><t t-else=""> Accept &amp; Sign</t>
                                </a>
                                <a t-elif="sale_order._has_to_be_paid(True)" role="button" id="o_sale_portal_paynow" data-bs-toggle="modal" data-bs-target="#modalaccept" href="#" t-att-class="'mb8 %s' % ('btn btn-light' if sale_order.transaction_ids else 'btn btn-primary')" >
                                    <i class="fa fa-check"/> <t t-if="not sale_order.signature">Accept &amp; Pay</t><t t-else="">Pay Now</t>
                                </a>
                                <div class="o_download_pdf btn-toolbar flex-sm-nowrap">
                                    <div class="btn-group flex-grow-1 me-1 mb-1">
                                        <a class="btn btn-secondary o_download_btn" t-att-href="sale_order.get_portal_url(report_type='pdf', download=True)" title="Download"><i class="fa fa-download"/> Download</a>
                                    </div>
                                    <div class="btn-group flex-grow-1 mb-1">
                                        <a class="btn btn-secondary o_print_btn o_portal_invoice_print" t-att-href="sale_order.get_portal_url(report_type='pdf')" id="print_invoice_report" title="Print" target="_blank"><i class="fa fa-print"/> Print</a>
                                    </div>
                                </div>
                            </li>

                            <li class="navspy list-group-item ps-0 flex-grow-1" t-ignore="true" role="complementary">
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
                                <div class="small mb-1">
                                    <strong class="text-muted">Salesperson</strong>
                                </div>
                                <div class="row flex-nowrap">
                                    <div class="col flex-grow-0 pe-2">
                                        <img class="rounded-circle mr4 float-start o_portal_contact_img"
                                            t-att-src="image_data_uri(sale_order.user_id.avatar_128)"
                                            alt="Contact"/>
                                    </div>
                                    <div class="col ps-0" style="min-width: 150px">
                                        <span t-field="sale_order.user_id"
                                            t-options='{"widget": "contact", "fields": ["name", "phone"], "no_marker": True}'/>
                                        <a href="#discussion" class="small">
                                            <i class="fa fa-comment"></i> Send message
                                        </a>
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
                        <div class="modal-dialog" t-if="sale_order._has_to_be_signed(True)">
                            <form id="accept" method="POST" t-att-data-order-id="sale_order.id" t-att-data-token="sale_order.access_token" class="js_accept_json modal-content js_website_submit_form">
                                <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
                                <header class="modal-header">
                                    <h4 class="modal-title">Validate Order</h4>
                                    <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                                </header>
                                <main class="modal-body" id="sign-dialog">
                                    <p>
                                        <span>By signing this proposal, I agree to the following terms:</span>
                                        <ul>
                                            <li>
                                                <span>Accepted on the behalf of:</span> <b t-field="sale_order.partner_id.commercial_partner_id"/>
                                            </li>
                                            <li>
                                                <span>For an amount of:</span> <b data-id="total_amount" t-field="sale_order.amount_total"/>
                                            </li>
                                            <li t-if="sale_order.payment_term_id">
                                                <span>With payment terms:</span> <b t-field="sale_order.payment_term_id.note"/>
                                            </li>
                                        </ul>
                                    </p>
                                    <t t-call="portal.signature_form">
                                        <t t-set="call_url" t-value="sale_order.get_portal_url(suffix='/accept')"/>
                                        <t t-set="default_name" t-value="sale_order.partner_id.name"/>
                                    </t>
                                </main>
                            </form>
                        </div>

                        <div class="modal-dialog" t-if="not sale_order._has_to_be_signed(True) and sale_order._has_to_be_paid(True)">
                            <div class="modal-content">
                                <header class="modal-header">
                                    <h4 class="modal-title">Validate Order</h4>
                                    <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                                </header>
                                <main class="modal-body" id="sign-dialog">
                                    <p>
                                        <span>By paying this proposal, I agree to the following terms:</span>
                                        <ul>
                                            <li>
                                                <span>Accepted on the behalf of:</span> <b t-field="sale_order.partner_id.commercial_partner_id"/>
                                            </li>
                                            <li>
                                                <span>For an amount of:</span> <b data-id="total_amount" t-field="sale_order.amount_total"/>
                                            </li>
                                            <li t-if="sale_order.payment_term_id">
                                                <span>With payment terms:</span> <b t-field="sale_order.payment_term_id.note"/>
                                            </li>
                                        </ul>
                                    </p>
                                    <div t-if="providers or tokens" id="payment_method" class="text-start">
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
                                    <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                                </header>
                                <main class="modal-body">
                                    <p>
                                        Tell us why you are refusing this quotation, this will help us improve our services.
                                    </p>
                                    <textarea rows="4" name="decline_message" required="" placeholder="Your feedback..." class="form-control" />
                                </main>
                                <footer class="modal-footer">
                                    <button type="submit" t-att-id="sale_order.id" class="btn btn-danger">
                                        <i class="fa fa-times"></i> Reject
                                    </button>
                                    <button type="button" class="btn btn-primary" data-bs-dismiss="modal">
                                        Cancel
                                    </button>
                                </footer>
                            </form>
                        </div>
                    </div>

                    <!-- status messages -->
                    <div t-if="message == 'sign_ok'" class="alert alert-success alert-dismissible d-print-none" role="status">
                        <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
                        <strong>Thank You!</strong><br/>
                        <t t-if="message == 'sign_ok' and sale_order.state in ['sale', 'done']">
                            Your order has been confirmed.
                        </t>
                        <t t-elif="message == 'sign_ok' and sale_order._has_to_be_paid()">
                            Your order has been signed but still needs to be paid to be confirmed.
                        </t>
                        <t t-else="">Your order has been signed.</t>
                    </div>

                    <div t-if="message == 'cant_reject'" class="alert alert-danger alert-dismissible d-print-none" role="alert">
                        <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
                        Your order is not in a state to be rejected.
                    </div>

                    <t t-if="sale_order.transaction_ids">
                        <t t-call="payment.transaction_status">
                            <t t-set="tx" t-value="sale_order.get_portal_last_transaction()"/>
                        </t>
                    </t>

                    <div t-if="sale_order.state == 'cancel'" class="alert alert-danger alert-dismissible d-print-none" role="alert">
                        <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="close"></button>
                        <strong>This quotation has been canceled.</strong> <a role="button" href="#discussion"><i class="fa fa-comment"/> Contact us to get a new quotation.</a>
                    </div>

                    <div t-if="sale_order.is_expired" class="alert alert-warning alert-dismissible d-print-none" role="alert">
                        <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="close"></button>
                        <strong>This offer expired!</strong> <a role="button" href="#discussion"><i class="fa fa-comment"/> Contact us to get a new quotation.</a>
                    </div>

                    <!-- main content -->
                    <div t-attf-class="card #{'pb-5' if report_type == 'html' else ''}" id="portal_sale_content">
                        <div t-call="sale.sale_order_portal_content"/>
                    </div>

                    <!-- bottom actions -->
                    <div t-if="sale_order._has_to_be_signed(True) or sale_order._has_to_be_paid(True)" class="row justify-content-center text-center d-print-none pt-1 pb-4">

                        <t t-if="sale_order._has_to_be_signed(True)">
                            <div class="col-sm-auto mt8">
                                <a role="button" class="btn btn-primary" data-bs-toggle="modal" data-bs-target="#modalaccept" href="#"><i class="fa fa-check"/><t t-if="sale_order._has_to_be_paid(True)"> Sign &amp; Pay</t><t t-else=""> Accept &amp; Sign</t></a>
                            </div>
                            <div class="col-sm-auto mt8">
                                <a role="button" class="btn btn-secondary" href="#discussion"><i class="fa fa-comment"/> Feedback</a>
                            </div>
                            <div class="col-sm-auto mt8">
                                <a role="button" class="btn btn-danger" data-bs-toggle="modal" data-bs-target="#modaldecline" href="#"> <i class="fa fa-times"/> Reject</a>
                            </div>
                        </t>
                        <div t-elif="sale_order._has_to_be_paid(True)" class="col-sm-auto mt8">
                            <a role="button" data-bs-toggle="modal" data-bs-target="#modalaccept" href="#" t-att-class="'%s' % ('btn btn-light' if sale_order.transaction_ids else 'btn btn-primary')">
                                <i class="fa fa-check"/> <t t-if="not sale_order.signature">Accept &amp; Pay</t><t t-else="">Pay Now</t>
                            </a>
                        </div>
                    </div>

                    <!-- chatter -->
                    <div id="sale_order_communication" class="mt-4">
                        <h2>History</h2>
                        <t t-call="portal.message_thread"/>
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
                <t t-out="sale_order.type_name"/>
                <em t-out="sale_order.name"/>
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
                <div class="row" id="invoicing_shipping_address">
                    <div class="col-lg-6">
                        <strong class="d-block mb-1">
                            <t t-if="sale_order.partner_shipping_id == sale_order.partner_invoice_id">
                                Invoicing and Shipping Address:
                            </t>
                            <t t-else="">
                                Invoicing Address:
                            </t>
                        </strong>
                        <address t-field="sale_order.partner_invoice_id"
                            t-options='{"widget": "contact", "fields": ["address", "name"], "no_marker": True}'/>
                    </div>
                    <div t-if="sale_order.partner_shipping_id != sale_order.partner_invoice_id"
                        id="shipping_address"
                        class="col-lg-6">
                        <strong class="d-block mb-1">
                            Shipping Address:
                        </strong>
                        <address t-field="sale_order.partner_shipping_id"
                            t-options='{"widget": "contact", "fields": ["address", "name"], "no_marker": True}'/>
                    </div>
                </div>

                <t t-set="invoices" t-value="sale_order.invoice_ids.filtered(lambda i: i.state not in ['draft', 'cancel'])"/>
                <div t-if="invoices" class="row">
                    <div class="col">
                        <strong class="d-block mb-1">Invoices</strong>
                        <ul class="list-group mb-4">
                            <t t-foreach="invoices" t-as="i">
                                <t t-set="report_url" t-value="i.get_portal_url(report_type='pdf', download=True)"/>
                                <t t-set="authorized_tx_ids" t-value="i.transaction_ids.filtered(lambda tx: tx.state == 'authorized')"/>
                                <div class="d-flex flex-wrap align-items-center justify-content-between">
                                    <div>
                                        <a t-att-href="report_url">
                                            <span t-out="i.name"/>
                                        </a>
                                        <div class="small d-lg-inline-block">Date: <span class="text-muted" t-field="i.invoice_date"/></div>
                                    </div>
                                    <span t-if="i.payment_state in ('paid', 'in_payment')" class="small badge text-bg-success orders_label_text_align">
                                        <i class="fa fa-fw fa-check"/> <b>Paid</b>
                                    </span>
                                    <span t-elif="i.payment_state == 'reversed'" class="small badge text-bg-success orders_label_text_align">
                                        <i class="fa fa-fw fa-check"/> <b>Reversed</b>
                                    </span>
                                    <span t-elif="authorized_tx_ids" class="small badge text-bg-success orders_label_text_align">
                                        <i class="fa fa-fw fa-check"/> <b>Authorized</b>
                                    </span>
                                    <span t-else="" class="small badge text-bg-info orders_label_text_align">
                                        <i class="fa fa-fw fa-clock-o"/> <b>Waiting Payment</b>
                                    </span>
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
                                <th class="text-start" id="product_name_header">Products</th>
                                <th class="text-end">Quantity</th>
                                <th t-attf-class="text-end {{ 'd-none d-sm-table-cell' if report_type == 'html' else '' }}">
                                    Unit Price
                                </th>
                                <th t-if="display_discount" t-attf-class="text-end {{ 'd-none d-sm-table-cell' if report_type == 'html' else '' }}">
                                    <span>Disc.%</span>
                                </th>
                                <th t-attf-class="text-end {{ 'd-none d-md-table-cell' if report_type == 'html' else '' }}">
                                    <span>Taxes</span>
                                </th>
                                <th class="text-end" >
                                    <span groups="account.group_show_line_subtotals_tax_excluded">Amount</span>
                                    <span groups="account.group_show_line_subtotals_tax_included">Total Price</span>
                                </th>
                            </tr>
                        </thead>
                        <tbody class="sale_tbody">

                            <t t-set="current_subtotal" t-value="0"/>
                            <t t-set="lines_to_report" t-value="sale_order._get_order_lines_to_report()"/>

                            <t t-foreach="lines_to_report" t-as="line">

                                    <t t-set="current_subtotal" t-value="current_subtotal + line.price_subtotal" groups="account.group_show_line_subtotals_tax_excluded"/>
                                    <t t-set="current_subtotal" t-value="current_subtotal + line.price_total" groups="account.group_show_line_subtotals_tax_included"/>

                                <tr t-att-class="'bg-200 fw-bold o_line_section' if line.display_type == 'line_section' else 'fst-italic o_line_note' if line.display_type == 'line_note' else ''">
                                    <t t-if="not line.display_type">
                                        <td id="product_name">
                                            <span t-field="line.name"/>
                                        </td>
                                        <td class="text-end">
                                            <div id="quote_qty">
                                                <span t-field="line.product_uom_qty"/>
                                                <span t-field="line.product_uom"/>
                                            </div>
                                        </td>
                                        <td t-attf-class="text-end {{ 'd-none d-sm-table-cell' if report_type == 'html' else '' }}">
                                            <div
                                                t-if="line.discount &gt;= 0"
                                                t-field="line.price_unit"
                                                t-att-style="line.discount and 'text-decoration: line-through' or None"
                                                t-att-class="(line.discount and 'text-danger' or '') + ' text-end'"
                                            />
                                            <div t-if="line.discount">
                                                <t t-out="(1-line.discount / 100.0) * line.price_unit" t-options='{"widget": "float", "decimal_precision": "Product Price"}'/>
                                            </div>
                                        </td>
                                        <td t-if="display_discount" t-attf-class="text-end {{ 'd-none d-sm-table-cell' if report_type == 'html' else '' }}">
                                            <strong t-if="line.discount &gt; 0" class="text-info">
                                                <t t-out="((line.discount % 1) and '%s' or '%d') % line.discount"/>%
                                            </strong>
                                        </td>
                                        <td t-attf-class="text-end {{ 'd-none d-md-table-cell' if report_type == 'html' else '' }}">
                                            <span t-out="', '.join(map(lambda x: (x.description or x.name), line.tax_id))"/>
                                        </td>
                                        <td t-if="not line.is_downpayment" class="text-end">
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
                                <tr t-if="current_section and (line_last or lines_to_report[line_index+1].display_type == 'line_section') and not line.is_downpayment"
                                    class="is-subtotal text-end">
                                    <td colspan="99">
                                        <strong class="mr16">Subtotal</strong>
                                        <span t-out="current_subtotal"
                                            t-options='{"widget": "monetary", "display_currency": sale_order.pricelist_id.currency_id}'
                                        />
                                    </td>
                                </tr>
                            </t>
                        </tbody>
                    </table>
                </div>

                <div id="total" class="row" name="total" style="page-break-inside: avoid;">
                    <div t-attf-class="#{'col-6' if report_type != 'html' else 'col-sm-7 col-md-6'} ms-auto">
                        <t t-call="sale.sale_order_portal_content_totals_table"/>
                    </div>
                </div>
            </section>

            <section t-if="sale_order.signature" id="signature" name="Signature">
                <div class="row mt-4" name="signature">
                    <div t-attf-class="#{'col-3' if report_type != 'html' else 'col-sm-7 col-md-4'} ms-auto text-center">
                        <h5>Signature</h5>
                        <img t-att-src="image_data_uri(sale_order.signature)" style="max-height: 6rem; max-width: 100%;"/>
                        <p t-field="sale_order.signed_by"/>
                    </div>
                </div>
            </section>

            <section t-if="not is_html_empty(sale_order.note)" id="terms" class="mt-5">
                <h3 class="">Terms &amp; Conditions</h3>
                <hr class="mt-0 mb-1"/>
                <t t-if="sale_order.terms_type == 'html'">
                    <!-- Note is plain text. This ensures a clickable link  -->
                    <t t-set="tc_url" t-value="'%s/terms' % (sale_order.get_base_url())"/>
                    <em>Terms &amp; Conditions: <a href="/terms"><t t-out="tc_url"/></a></em>
                </t>
                <t t-else="">
                    <em t-field="sale_order.note"/>
                </t>
            </section>

            <section t-if="sale_order.payment_term_id" class="mt-5">
                <h3 class="">Payment terms</h3>
                <hr class="mt-0 mb-1"/>
                <span t-field="sale_order.payment_term_id.note"/>
            </section>
        </div>
    </template>

    <template id="sale_order_portal_content_totals_table">
        <table class="table table-sm">
            <t t-set="tax_totals" t-value="sale_order.tax_totals"/>
            <t t-call="account.document_tax_totals"/>
        </table>
    </template>
</odoo>

```

## File: views\utm_campaign_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="utm_campaign_view_kanban" model="ir.ui.view">
        <field name="name">utm.campaign.view.kanban</field>
        <field name="model">utm.campaign</field>
        <field name="inherit_id" ref="utm.utm_campaign_view_kanban"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='utm_statistics']" position="inside">
                <div class="me-3" title="Revenues" groups="sales_team.group_sale_salesman">
                    <field name="currency_id" invisible="True"/>
                    <small class="fw-bold">
                        <field name="invoiced_amount" widget="monetary" options="{'currency_field': 'currency_id'}"/>
                    </small>
                </div>
                <div class="me-3" title="Quotations" groups="sales_team.group_sale_salesman">
                    <i class="fa fa-money text-muted"></i> <small class="fw-bold"><field name="quotation_count"/></small>
                </div>
            </xpath>
        </field>
    </record>

    <record id="utm_campaign_view_form" model="ir.ui.view">
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
        <ul t-attf-class="list-unstyled js_add_cart_variants mb-0 #{ul_class}" t-att-data-attribute_exclusions="json.dumps(attribute_exclusions)">
            <t t-foreach="product.valid_product_template_attribute_line_ids" t-as="ptal">
                <!-- Attributes selection is hidden if there is only one value available and it's not a custom value -->
                <li t-att-data-attribute_id="ptal.attribute_id.id"
                    t-att-data-attribute_name="ptal.attribute_id.name"
                    t-att-data-attribute_display_type="ptal.attribute_id.display_type"
                    t-attf-class="variant_attribute #{'d-none' if len(ptal.product_template_value_ids._only_active()) == 1 and not ptal.product_template_value_ids._only_active()[0].is_custom else ''}">

                    <!-- Used to customize layout if the only available attribute value is custom -->
                    <t t-set="single" t-value="len(ptal.product_template_value_ids._only_active()) == 1"/>
                    <t t-set="single_and_custom" t-value="single and ptal.product_template_value_ids._only_active()[0].is_custom" />
                    <strong t-field="ptal.attribute_id.name" class="attribute_name"/>

                    <t t-if="ptal.attribute_id.display_type == 'select'">
                        <select
                            t-att-data-attribute_id="ptal.attribute_id.id"
                            t-attf-class="form-select css_attribute_select o_wsale_product_attribute js_variant_change #{ptal.attribute_id.create_variant} #{'d-none' if single_and_custom else ''}"
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
                        <ul t-att-data-attribute_id="ptal.attribute_id.id" t-attf-class="list-inline list-unstyled o_wsale_product_attribute #{'d-none' if single_and_custom else ''}">
                            <t t-foreach="ptal.product_template_value_ids._only_active()" t-as="ptav">
                                <li class="list-inline-item mb-3 js_attribute_value" style="margin: 0;">
                                    <label class="col-form-label">
                                        <div class="form-check">
                                            <input type="radio"
                                                t-attf-class="form-check-input js_variant_change #{ptal.attribute_id.create_variant}"
                                                t-att-checked="ptav in combination"
                                                t-att-name="'ptal-%s' % ptal.id"
                                                t-att-value="ptav.id"
                                                t-att-data-value_id="ptav.id"
                                                t-att-data-value_name="ptav.name"
                                                t-att-data-attribute_name="ptav.attribute_id.name"
                                                t-att-data-is_custom="ptav.is_custom"
                                                t-att-data-is_single="single"
                                                t-att-data-is_single_and_custom="single_and_custom" />
                                            <div class="radio_input_value form-check-label">
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
                            t-attf-class="btn-group-toggle list-inline list-unstyled o_wsale_product_attribute #{'d-none' if single_and_custom else ''}"
                            data-bs-toggle="buttons">
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
                                    <label class="radio_input_value o_variant_pills_input_value"
                                           t-att-for="ptav.id">
                                        <span t-field="ptav.name"/>
                                        <t t-call="sale.badge_extra_price"/>
                                    </label>
                                </li>
                            </t>
                        </ul>
                    </t>

                    <t t-if="ptal.attribute_id.display_type == 'color'">
                        <ul t-att-data-attribute_id="ptal.attribute_id.id" t-attf-class="list-inline o_wsale_product_attribute #{'d-none' if single_and_custom else ''}">
                            <li t-foreach="ptal.product_template_value_ids._only_active()" t-as="ptav" class="list-inline-item me-1">
                                <label t-attf-style="background-color:#{ptav.html_color or ptav.product_attribute_value_id.name if not ptav.is_custom else ''}"
                                    t-attf-class="css_attribute_color #{'active' if ptav in combination else ''} #{'custom_value' if ptav.is_custom else ''} #{'transparent' if (not ptav.is_custom and not ptav.html_color) else ''}">
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
        <span class="badge rounded-pill text-bg-light border" t-if="combination_info_variant['price_extra']">
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
        <span class="sign_badge_price_extra" t-out="combination_info_variant['price_extra'] > 0 and '+' or '-'"/>
        <span t-out="abs(combination_info_variant['price_extra'])" class="variant_price_extra text-muted fst-italic" style="white-space: nowrap;"
            t-options='{
                "widget": "monetary",
                "display_currency": (pricelist or product).currency_id
            }'/>
        </span>
    </template>
</odoo>

```

## File: wizard\account_accrued_orders_wizard_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <record id="action_accrued_revenue_entry" model="ir.actions.act_window">
        <field name="name">Accrued Revenue Entry</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">account.accrued.orders.wizard</field>
        <field name="view_mode">form</field>
        <field name="binding_model_id" ref="sale.model_sale_order"/>
        <field name="groups_id" eval="[(4, ref('account.group_account_user'))]"/>
        <field name="target">new</field>
    </record>

</odoo>

```

## File: wizard\account_payment_register.py

```python
# -*- coding: utf-8 -*-

from odoo import models


class AccountPaymentRegister(models.TransientModel):
    _inherit = 'account.payment.register'

    def _create_payment_vals_from_wizard(self, batch_result):
        vals = super()._create_payment_vals_from_wizard(batch_result)
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

## File: wizard\payment_link_wizard.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from werkzeug import urls

from odoo import api, models


class PaymentLinkWizard(models.TransientModel):
    _inherit = 'payment.link.wizard'
    _description = 'Generate Sales Payment Link'

    def _get_payment_provider_available(self, res_model, res_id, **kwargs):
        """ Select and return the providers matching the criteria.

        :param str res_model: active model
        :param int res_id: id of 'active_model' record
        :return: The compatible providers
        :rtype: recordset of `payment.provider`
        """
        if res_model == 'sale.order':
            kwargs['sale_order_id'] = res_id
        return super()._get_payment_provider_available(**kwargs)

    def _get_additional_link_values(self):
        """ Override of `payment` to add `sale_order_id` to the payment link values.

        The other values related to the sales order are directly read from the sales order.

        Note: self.ensure_one()

        :return: The additional payment link values.
        :rtype: dict
        """
        res = super()._get_additional_link_values()
        if self.res_model != 'sale.order':
            return res

        # Order-related fields are retrieved in the controller
        return {
            'sale_order_id': self.res_id,
        }

```

## File: wizard\payment_link_wizard_views.xml

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

## File: wizard\payment_provider_onboarding_wizard.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class PaymentWizard(models.TransientModel):
    """ Override for the sale quotation onboarding panel. """

    _inherit = 'payment.provider.onboarding.wizard'
    _name = 'sale.payment.provider.onboarding.wizard'
    _description = 'Sale Payment provider onboarding wizard'

    def _get_default_payment_method(self):
        return self.env.company.sale_onboarding_payment_method or 'digital_signature'

    payment_method = fields.Selection(selection_add=[
        ('digital_signature', "Electronic signature"),
        ('stripe', "Credit & Debit card (via Stripe)"),
        ('paypal', "PayPal"),
        ('manual', "Custom payment instructions"),
    ], default=_get_default_payment_method)
    #

    def _set_payment_provider_onboarding_step_done(self):
        """ Override. """
        self.env.company.sudo().set_onboarding_step_done('sale_onboarding_order_confirmation_state')

    def add_payment_methods(self):
        self.env.company.sale_onboarding_payment_method = self.payment_method
        if self.payment_method == 'digital_signature':
            self.env.company.portal_confirmation_sign = True
        if self.payment_method in ('paypal', 'stripe', 'other', 'manual'):
            self.env.company.portal_confirmation_pay = True

        return super().add_payment_methods()

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
from odoo.fields import Command
from odoo.tools import float_is_zero


class SaleAdvancePaymentInv(models.TransientModel):
    _name = 'sale.advance.payment.inv'
    _description = "Sales Advance Payment Invoice"

    advance_payment_method = fields.Selection(
        selection=[
            ('delivered', "Regular invoice"),
            ('percentage', "Down payment (percentage)"),
            ('fixed', "Down payment (fixed amount)"),
        ],
        string="Create Invoice",
        default='delivered',
        required=True,
        help="A standard invoice is issued with all the order lines ready for invoicing,"
            "according to their invoicing policy (based on ordered or delivered quantity).")
    count = fields.Integer(string="Order Count", compute='_compute_count')
    sale_order_ids = fields.Many2many(
        'sale.order', default=lambda self: self.env.context.get('active_ids'))

    # Down Payment logic
    has_down_payments = fields.Boolean(
        string="Has down payments", compute="_compute_has_down_payments")
    deduct_down_payments = fields.Boolean(string="Deduct down payments", default=True)

    # New Down Payment
    product_id = fields.Many2one(
        comodel_name='product.product',
        string="Down Payment Product",
        domain=[('type', '=', 'service')],
        compute='_compute_product_id',
        readonly=False,
        store=True)
    amount = fields.Float(
        string="Down Payment Amount",
        help="The percentage of amount to be invoiced in advance, taxes excluded.")
    fixed_amount = fields.Monetary(
        string="Down Payment Amount (Fixed)",
        help="The fixed amount to be invoiced in advance, taxes excluded.")
    currency_id = fields.Many2one(
        comodel_name='res.currency',
        compute='_compute_currency_id',
        store=True)
    company_id = fields.Many2one(
        comodel_name='res.company',
        compute='_compute_company_id',
        store=True)

    # Only used when there is no down payment product available
    #  to setup the down payment product
    deposit_account_id = fields.Many2one(
        comodel_name='account.account',
        string="Income Account",
        domain=[('deprecated', '=', False)],
        help="Account used for deposits")
    deposit_taxes_id = fields.Many2many(
        comodel_name='account.tax',
        string="Customer Taxes",
        domain=[('type_tax_use', '=', 'sale')],
        help="Taxes used for deposits")

    #=== COMPUTE METHODS ===#

    @api.depends('sale_order_ids')
    def _compute_count(self):
        for wizard in self:
            wizard.count = len(wizard.sale_order_ids)

    @api.depends('sale_order_ids')
    def _compute_has_down_payments(self):
        for wizard in self:
            wizard.has_down_payments = bool(
                wizard.sale_order_ids.order_line.filtered('is_downpayment')
            )

    # next computed fields are only used for down payments invoices and therefore should only
    # have a value when 1 unique SO is invoiced through the wizard
    @api.depends('sale_order_ids')
    def _compute_currency_id(self):
        self.currency_id = False
        for wizard in self:
            if wizard.count == 1:
                wizard.currency_id = wizard.sale_order_ids.currency_id

    @api.depends('sale_order_ids')
    def _compute_company_id(self):
        self.company_id = False
        for wizard in self:
            if wizard.count == 1:
                wizard.company_id = wizard.sale_order_ids.company_id

    @api.depends('company_id')  # 'dumb' depends to trigger the computation
    def _compute_product_id(self):
        self.product_id = False
        dp_product_id = int(self.env['ir.config_parameter'].sudo().get_param(
            'sale.default_deposit_product_id'))
        if not dp_product_id:
            return
        for wizard in self:
            if wizard.count == 1:
                wizard.product_id = dp_product_id

    #=== ONCHANGE METHODS ===#

    @api.onchange('advance_payment_method')
    def _onchange_advance_payment_method(self):
        if self.advance_payment_method == 'percentage':
            amount = self.default_get(['amount']).get('amount')
            return {'value': {'amount': amount}}

    #=== CONSTRAINT METHODS ===#

    @api.constrains('advance_payment_method', 'amount', 'fixed_amount')
    def _check_amount_is_positive(self):
        for wizard in self:
            if wizard.advance_payment_method == 'percentage' and wizard.amount <= 0.00:
                raise UserError(_('The value of the down payment amount must be positive.'))
            elif wizard.advance_payment_method == 'fixed' and wizard.fixed_amount <= 0.00:
                raise UserError(_('The value of the down payment amount must be positive.'))

    @api.constrains('product_id')
    def _check_down_payment_product_is_valid(self):
        for wizard in self:
            if wizard.count > 1 or not wizard.product_id:
                continue
            if wizard.product_id.invoice_policy != 'order':
                raise UserError(_(
                    "The product used to invoice a down payment should have an invoice policy"
                    "set to \"Ordered quantities\"."
                    " Please update your deposit product to be able to create a deposit invoice."))
            if wizard.product_id.type != 'service':
                raise UserError(_(
                    "The product used to invoice a down payment should be of type 'Service'."
                    " Please use another product or update this product."))

    #=== ACTION METHODS ===#

    def create_invoices(self):
        self._create_invoices(self.sale_order_ids)

        if self.env.context.get('open_invoices'):
            return self.sale_order_ids.action_view_invoice()

        return {'type': 'ir.actions.act_window_close'}

    #=== BUSINESS METHODS ===#

    def _create_invoices(self, sale_orders):
        self.ensure_one()
        if self.advance_payment_method == 'delivered':
            return sale_orders._create_invoices(final=self.deduct_down_payments)
        else:
            self.sale_order_ids.ensure_one()
            self = self.with_company(self.company_id)
            order = self.sale_order_ids

            # Create deposit product if necessary
            if not self.product_id:
                self.product_id = self.env['product.product'].create(
                    self._prepare_down_payment_product_values()
                )
                self.env['ir.config_parameter'].sudo().set_param(
                    'sale.default_deposit_product_id', self.product_id.id)

            # Create down payment section if necessary
            if not any(line.display_type and line.is_downpayment for line in order.order_line):
                self.env['sale.order.line'].create(
                    self._prepare_down_payment_section_values(order)
                )

            down_payment_so_line = self.env['sale.order.line'].create(
                self._prepare_so_line_values(order)
            )

            invoice = self.env['account.move'].sudo().create(
                self._prepare_invoice_values(order, down_payment_so_line)
            ).with_user(self.env.uid)  # Unsudo the invoice after creation

            invoice.message_post_with_view(
                'mail.message_origin_link',
                values={'self': invoice, 'origin': order},
                subtype_id=self.env.ref('mail.mt_note').id)

            return invoice

    def _prepare_down_payment_product_values(self):
        self.ensure_one()
        return {
            'name': _('Down payment'),
            'type': 'service',
            'invoice_policy': 'order',
            'company_id': False,
            'property_account_income_id': self.deposit_account_id.id,
            'taxes_id': [Command.set(self.deposit_taxes_id.ids)],
        }

    def _prepare_down_payment_section_values(self, order):
        context = {'lang': order.partner_id.lang}

        so_values = {
            'name': _('Down Payments'),
            'product_uom_qty': 0.0,
            'order_id': order.id,
            'display_type': 'line_section',
            'is_downpayment': True,
            'sequence': order.order_line and order.order_line[-1].sequence + 1 or 10,
        }

        del context
        return so_values

    def _prepare_so_line_values(self, order):
        self.ensure_one()
        analytic_distribution = {}
        amount_total = sum(order.order_line.mapped("price_total"))
        if not float_is_zero(amount_total, precision_rounding=self.currency_id.rounding):
            for line in order.order_line:
                distrib_dict = line.analytic_distribution or {}
                for account, distribution in distrib_dict.items():
                    analytic_distribution[account] = distribution * line.price_total + analytic_distribution.get(account, 0)
            for account, distribution_amount in analytic_distribution.items():
                analytic_distribution[account] = distribution_amount/amount_total
        context = {'lang': order.partner_id.lang}
        so_values = {
            'name': _('Down Payment: %s (Draft)', time.strftime('%m %Y')),
            'price_unit': self._get_down_payment_amount(order),
            'product_uom_qty': 0.0,
            'order_id': order.id,
            'discount': 0.0,
            'product_id': self.product_id.id,
            'analytic_distribution': analytic_distribution,
            'is_downpayment': True,
            'sequence': order.order_line and order.order_line[-1].sequence + 1 or 10,
        }
        del context
        return so_values

    def _get_down_payment_amount(self, order):
        self.ensure_one()
        if self.advance_payment_method == 'percentage':
            advance_product_taxes = self.product_id.taxes_id.filtered(lambda tax: tax.company_id == order.company_id)
            if all(order.fiscal_position_id.map_tax(advance_product_taxes).mapped('price_include')):
                amount = order.amount_total * self.amount / 100
            else:
                amount = order.amount_untaxed * self.amount / 100
        else:  # Fixed amount
            amount = self.fixed_amount
        return amount

    def _prepare_invoice_values(self, order, so_line):
        self.ensure_one()
        return {
            **order._prepare_invoice(),
            'invoice_line_ids': [
                Command.create(
                    so_line._prepare_invoice_line(
                        name=self._get_down_payment_description(order),
                        quantity=1.0,
                    )
                )
            ],
        }

    def _get_down_payment_description(self, order):
        self.ensure_one()
        context = {'lang': order.partner_id.lang}
        if self.advance_payment_method == 'percentage':
            name = _("Down payment of %s%%", self.amount)
        else:
            name = _('Down Payment')
        del context

        return name

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
                    <field name="sale_order_ids" invisible="1"/>
                    <field name="has_down_payments" invisible="1"/>
                    <field name="count" attrs="{'invisible': [('count', '=', 1)]}"/>
                    <field name="advance_payment_method" class="oe_inline"
                        widget="radio"
                        attrs="{'invisible': [('count', '&gt;', 1)]}"/>
                    <label for="deduct_down_payments"
                        string=""
                        attrs="{'invisible': ['|', ('has_down_payments', '=', False), ('advance_payment_method', '!=', 'delivered')]}"/>
                    <div attrs="{'invisible': ['|', ('has_down_payments', '=', False), ('advance_payment_method', '!=', 'delivered')]}"
                        id="down_payment_details">
                        <field name="deduct_down_payments" nolabel="1"/>
                        <label for="deduct_down_payments"/>
                    </div>
                </group>
                <group name="down_payment_specification"
                    attrs="{'invisible': [('advance_payment_method', 'not in', ('fixed', 'percentage'))]}">
                    <field name="company_id" invisible="1"/>
                    <field name="product_id" invisible="1"/>
                    <label for="fixed_amount" attrs="{'invisible': [('advance_payment_method', '!=', 'fixed')]}"/>
                    <label for="amount" attrs="{'invisible': [('advance_payment_method', '!=', 'percentage')]}"/>
                    <div id="payment_method_details">
                        <field name="currency_id" invisible="1"/>
                        <field name="fixed_amount"
                            attrs="{'required': [('advance_payment_method', '=', 'fixed')],
                                'invisible': [('advance_payment_method', '!=', 'fixed')]}"
                            class="oe_inline"/>
                        <field name="amount"
                            attrs="{'required': [('advance_payment_method', '=', 'percentage')],
                                'invisible': [('advance_payment_method', '!=', 'percentage')]}"
                            class="oe_inline"/>
                        <span attrs="{'invisible': [('advance_payment_method', '!=', 'percentage')]}"
                            class="oe_inline">%</span>
                    </div>
                    <field name="deposit_account_id"
                        options="{'no_create': True}"
                        attrs="{'invisible': [('product_id', '!=', False)]}"
                        groups="account.group_account_manager"/>
                    <field name="deposit_taxes_id"
                        widget="many2many_tags"
                        attrs="{'invisible': [('product_id', '!=', False)]}"/>
                </group>
                <footer>
                    <button name="create_invoices" type="object"
                        id="create_invoice_open"
                        string="Create and View Invoice"
                        context="{'open_invoices': True}"
                        class="btn-primary" data-hotkey="q"/>
                    <button name="create_invoices" type="object"
                        id="create_invoice"
                        string="Create Invoice"
                        data-hotkey="w"/>
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
        <field name="binding_model_id" ref="sale.model_sale_order"/>
        <field name="binding_view_types">list</field>
    </record>

</odoo>

```

## File: wizard\sale_order_cancel.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.exceptions import UserError


class SaleOrderCancel(models.TransientModel):
    _name = 'sale.order.cancel'
    _inherit = 'mail.composer.mixin'
    _description = "Sales Order Cancel"

    @api.model
    def _default_email_from(self):
        if self.env.user.email:
            return self.env.user.email_formatted
        raise UserError(_("Unable to post message, please configure the sender's email address."))

    @api.model
    def _default_author_id(self):
        return self.env.user.partner_id

    # origin
    email_from = fields.Char(string="From", default=_default_email_from)
    author_id = fields.Many2one(
        'res.partner',
        string="Author",
        index=True,
        ondelete='set null',
        default=_default_author_id,
    )

    # recipients
    recipient_ids = fields.Many2many(
        'res.partner',
        string="Recipients",
        compute='_compute_recipient_ids',
        readonly=False,
    )
    order_id = fields.Many2one('sale.order', string="Sale Order", required=True, ondelete='cascade')
    display_invoice_alert = fields.Boolean(
        string="Invoice Alert",
        compute='_compute_display_invoice_alert',
        compute_sudo=True,
    )

    @api.depends('order_id')
    def _compute_recipient_ids(self):
        for wizard in self:
            wizard.recipient_ids = wizard.order_id.partner_id \
                                   | wizard.order_id.message_partner_ids \
                                   - wizard.author_id

    @api.depends('order_id')
    def _compute_display_invoice_alert(self):
        for wizard in self:
            wizard.display_invoice_alert = bool(
                wizard.order_id.invoice_ids.filtered(lambda inv: inv.state == 'draft')
            )

    @api.depends('order_id')
    def _compute_subject(self):
        for wizard in self:
            if wizard.template_id:
                wizard.subject = wizard.template_id._render_field(
                    'subject',
                    wizard.order_id.ids,
                    post_process=True,
                    compute_lang=True,
                )[wizard.order_id.id]

    @api.depends('order_id')
    def _compute_body(self):
        for wizard in self:
            if wizard.template_id:
                wizard.body = wizard.template_id._render_field(
                    'body_html',
                    wizard.order_id.ids,
                    post_process=True,
                    compute_lang=True,
                )[wizard.order_id.id]

    def action_send_mail_and_cancel(self):
        self.ensure_one()
        self.order_id.message_post(
            subject=self.subject,
            body=self.body,
            message_type='comment',
            email_from=self.email_from,
            email_layout_xmlid='mail.mail_notification_light',
            partner_ids=self.recipient_ids.ids,
        )
        return self.action_cancel()

    def action_cancel(self):
        return self.order_id.with_context(disable_cancel_warning=True).action_cancel()

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
                <group col="1">
                    <field name="render_model" invisible="1"/>
                    <field name="order_id" invisible="1"/>
                    <field name="template_id" invisible="1"/>
                    <field name="display_invoice_alert" invisible="1"/>
                    <div col="2"
                         class="alert alert-warning"
                         role="alert">
                        <span>Are you sure you want to cancel this order? <br/></span>
                        <span id="display_invoice_alert"
                              attrs="{'invisible': [('display_invoice_alert', '=', False)]}">
                            Draft invoices for this order will be cancelled. <br/>
                        </span>
                    </div>
                    <group col="2">
                        <field name="recipient_ids"
                               widget="many2many_tags_email"
                               context="{'force_email': True,
                                         'show_email': True,
                                         'no_create_edit': True}"/>
                    </group>
                    <group col="2">
                        <field name="subject" placeholder="Subject"/>
                    </group>
                    <field name="body"
                           class="oe-bordered-editor"
                           options="{'style-inline': true}"/>
                </group>
                <footer>
                    <button string="Send and cancel"
                            name="action_send_mail_and_cancel"
                            type="object"
                            class="btn-primary"/>
                    <button string="Cancel"
                            name="action_cancel"
                            type="object"
                            class="btn-primary mx-1"/>
                    <button string="Discard"
                            class="btn-secondary"
                            special="cancel"/>
                </footer>
            </form>
        </field>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_payment_register
from . import mail_compose_message
from . import payment_provider_onboarding_wizard
from . import payment_link_wizard
from . import sale_make_invoice_advance
from . import sale_order_cancel

```

