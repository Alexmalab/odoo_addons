# Odoo Module: payment

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, SUPERUSER_ID

from . import models
from . import controllers
from . import wizards


def reset_payment_provider(cr, registry, provider):
    env = api.Environment(cr, SUPERUSER_ID, {})
    acquirers = env['payment.acquirer'].search([('provider', '=', provider)])
    acquirers.write({
        'view_template_id': acquirers._get_default_view_template_id().id,
        'provider': 'manual',
        'state': 'disabled',
    })

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Payment Acquirer',
    'category': 'Hidden',
    'summary': 'Base Module for Payment Acquirers',
    'version': '1.0',
    'description': """Payment Acquirer Base Module""",
    'depends': ['account'],
    'data': [
        'data/account_data.xml',
        'data/payment_icon_data.xml',
        'data/payment_acquirer_data.xml',
        'data/payment_cron.xml',
        'views/payment_views.xml',
        'views/account_payment_views.xml',
        'views/account_invoice_views.xml',
        'views/payment_acquirer_onboarding_templates.xml',
        'views/payment_templates.xml',
        'views/payment_portal_templates.xml',
        'views/assets.xml',
        'views/res_partner_views.xml',
        'security/ir.model.access.csv',
        'security/payment_security.xml',
        'wizards/payment_link_wizard_views.xml',
    ],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: controllers\portal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import hashlib
import hmac
import logging
from unicodedata import normalize
import psycopg2
import werkzeug

from odoo import http, _
from odoo.http import request
from odoo.osv import expression
from odoo.tools import DEFAULT_SERVER_DATETIME_FORMAT, consteq, ustr
from odoo.tools.float_utils import float_repr
from datetime import datetime, timedelta


_logger = logging.getLogger(__name__)

class PaymentProcessing(http.Controller):

    @staticmethod
    def remove_payment_transaction(transactions):
        tx_ids_list = request.session.get("__payment_tx_ids__", [])
        if transactions:
            for tx in transactions:
                if tx.id in tx_ids_list:
                    tx_ids_list.remove(tx.id)
        else:
            return False
        request.session["__payment_tx_ids__"] = tx_ids_list
        return True

    @staticmethod
    def add_payment_transaction(transactions):
        if not transactions:
            return False
        tx_ids_list = set(request.session.get("__payment_tx_ids__", [])) | set(transactions.ids)
        request.session["__payment_tx_ids__"] = list(tx_ids_list)
        return True

    @staticmethod
    def get_payment_transaction_ids():
        # return the ids and not the recordset, since we might need to
        # sudo the browse to access all the record
        # I prefer to let the controller chose when to access to payment.transaction using sudo
        return request.session.get("__payment_tx_ids__", [])

    @http.route(['/payment/process'], type="http", auth="public", website=True, sitemap=False)
    def payment_status_page(self, **kwargs):
        # When the customer is redirect to this website page,
        # we retrieve the payment transaction list from his session
        tx_ids_list = self.get_payment_transaction_ids()
        payment_transaction_ids = request.env['payment.transaction'].sudo().browse(tx_ids_list).exists()

        render_ctx = {
            'payment_tx_ids': payment_transaction_ids.ids,
        }
        return request.render("payment.payment_process_page", render_ctx)

    @http.route(['/payment/process/poll'], type="json", auth="public")
    def payment_status_poll(self):
        # retrieve the transactions
        tx_ids_list = self.get_payment_transaction_ids()

        payment_transaction_ids = request.env['payment.transaction'].sudo().search([
            ('id', 'in', list(tx_ids_list)),
            ('date', '>=', (datetime.now() - timedelta(days=1)).strftime(DEFAULT_SERVER_DATETIME_FORMAT)),
        ])
        if not payment_transaction_ids:
            return {
                'success': False,
                'error': 'no_tx_found',
            }

        processed_tx = payment_transaction_ids.filtered('is_processed')
        self.remove_payment_transaction(processed_tx)

        # create the returned dictionnary
        result = {
            'success': True,
            'transactions': [],
        }
        # populate the returned dictionnary with the transactions data
        for tx in payment_transaction_ids:
            message_to_display = tx.acquirer_id[tx.state + '_msg'] if tx.state in ['done', 'pending', 'cancel'] else None
            tx_info = {
                'reference': tx.reference,
                'state': tx.state,
                'return_url': tx.return_url,
                'is_processed': tx.is_processed,
                'state_message': tx.state_message,
                'message_to_display': message_to_display,
                'amount': tx.amount,
                'currency': tx.currency_id.name,
                'acquirer_provider': tx.acquirer_id.provider,
            }
            tx_info.update(tx._get_processing_info())
            result['transactions'].append(tx_info)

        tx_to_process = payment_transaction_ids.filtered(lambda x: x.state == 'done' and x.is_processed is False)
        try:
            tx_to_process._post_process_after_done()
        except psycopg2.OperationalError as e:
            request.env.cr.rollback()
            result['success'] = False
            result['error'] = "tx_process_retry"
        except Exception as e:
            request.env.cr.rollback()
            result['success'] = False
            result['error'] = str(e)
            _logger.exception("Error while processing transaction(s) %s, exception \"%s\"", tx_to_process.ids, str(e))

        return result

class WebsitePayment(http.Controller):
    @http.route(['/my/payment_method'], type='http', auth="user", website=True)
    def payment_method(self, **kwargs):
        acquirers = list(request.env['payment.acquirer'].search([
            ('state', 'in', ['enabled', 'test']), ('registration_view_template_id', '!=', False),
            ('payment_flow', '=', 's2s'), ('company_id', '=', request.env.company.id)
        ]))
        partner = request.env.user.partner_id
        payment_tokens = partner.payment_token_ids
        payment_tokens |= partner.commercial_partner_id.sudo().payment_token_ids
        return_url = request.params.get('redirect', '/my/payment_method')
        values = {
            'pms': payment_tokens,
            'acquirers': acquirers,
            'error_message': [kwargs['error']] if kwargs.get('error') else False,
            'return_url': return_url,
            'bootstrap_formatting': True,
            'partner_id': partner.id
        }
        return request.render("payment.pay_methods", values)

    @http.route(['/website_payment/pay'], type='http', auth='public', website=True, sitemap=False)
    def pay(self, reference='', order_id=None, amount=False, currency_id=None, acquirer_id=None, partner_id=False, access_token=None, **kw):
        """
        Generic payment page allowing public and logged in users to pay an arbitrary amount.

        In the case of a public user access, we need to ensure that the payment is made anonymously - e.g. it should not be
        possible to pay for a specific partner simply by setting the partner_id GET param to a random id. In the case where
        a partner_id is set, we do an access_token check based on the payment.link.wizard model (since links for specific
        partners should be created from there and there only). Also noteworthy is the filtering of s2s payment methods -
        we don't want to create payment tokens for public users.

        In the case of a logged in user, then we let access rights and security rules do their job.
        """
        env = request.env
        user = env.user.sudo()
        reference = normalize('NFKD', reference).encode('ascii','ignore').decode('utf-8')
        if partner_id and not access_token:
            raise werkzeug.exceptions.NotFound
        if partner_id and access_token:
            token_ok = request.env['payment.link.wizard'].check_token(access_token, int(partner_id), float(amount), int(currency_id))
            if not token_ok:
                raise werkzeug.exceptions.NotFound

        invoice_id = kw.get('invoice_id')

        # Default values
        values = {
            'amount': 0.0,
            'currency': user.company_id.currency_id,
        }

        # Check sale order
        if order_id:
            try:
                order_id = int(order_id)
                if partner_id:
                    # `sudo` needed if the user is not connected.
                    # A public user woudn't be able to read the sale order.
                    # With `partner_id`, an access_token should be validated, preventing a data breach.
                    order = env['sale.order'].sudo().browse(order_id)
                else:
                    order = env['sale.order'].browse(order_id)
                values.update({
                    'currency': order.currency_id,
                    'amount': order.amount_total,
                    'order_id': order_id
                })
            except:
                order_id = None

        if invoice_id:
            try:
                values['invoice_id'] = int(invoice_id)
            except ValueError:
                invoice_id = None

        # Check currency
        if currency_id:
            try:
                currency_id = int(currency_id)
                values['currency'] = env['res.currency'].browse(currency_id)
            except:
                pass

        # Check amount
        if amount:
            try:
                amount = float(amount)
                values['amount'] = amount
            except:
                pass

        # Check reference
        reference_values = order_id and {'sale_order_ids': [(4, order_id)]} or {}
        values['reference'] = env['payment.transaction']._compute_reference(values=reference_values, prefix=reference)

        # Check acquirer
        acquirers = None
        if order_id and order:
            cid = order.company_id.id
        elif kw.get('company_id'):
            try:
                cid = int(kw.get('company_id'))
            except:
                cid = user.company_id.id
        else:
            cid = user.company_id.id

        # Check partner
        if not user._is_public():
            # NOTE: this means that if the partner was set in the GET param, it gets overwritten here
            # This is something we want, since security rules are based on the partner - assuming the
            # access_token checked out at the start, this should have no impact on the payment itself
            # existing besides making reconciliation possibly more difficult (if the payment partner is
            # not the same as the invoice partner, for example)
            partner_id = user.partner_id.id
        elif partner_id:
            partner_id = int(partner_id)

        values.update({
            'partner_id': partner_id,
            'bootstrap_formatting': True,
            'error_msg': kw.get('error_msg')
        })

        acquirer_domain = ['&', ('state', 'in', ['enabled', 'test']), ('company_id', '=', cid)]
        if partner_id:
            partner = request.env['res.partner'].browse([partner_id])
            acquirer_domain = expression.AND([
            acquirer_domain,
            ['|', ('country_ids', '=', False), ('country_ids', 'in', [partner.sudo().country_id.id])]
        ])
        if acquirer_id:
            acquirers = env['payment.acquirer'].browse(int(acquirer_id))
        if order_id:
            acquirers = env['payment.acquirer'].search(acquirer_domain)
        if not acquirers:
            acquirers = env['payment.acquirer'].search(acquirer_domain)

        # s2s mode will always generate a token, which we don't want for public users
        valid_flows = ['form', 's2s'] if not user._is_public() else ['form']
        values['acquirers'] = [acq for acq in acquirers if acq.payment_flow in valid_flows]
        if partner_id:
            values['pms'] = request.env['payment.token'].search([
                ('acquirer_id', 'in', acquirers.ids),
                ('partner_id', '=', partner_id)
            ])
        else:
            values['pms'] = []


        return request.render('payment.pay', values)

    @http.route(['/website_payment/transaction/<string:reference>/<string:amount>/<string:currency_id>',
                '/website_payment/transaction/v2/<string:amount>/<string:currency_id>/<path:reference>',
                '/website_payment/transaction/v2/<string:amount>/<string:currency_id>/<path:reference>/<int:partner_id>'], type='json', auth='public')
    def transaction(self, acquirer_id, reference, amount, currency_id, partner_id=False, **kwargs):
        acquirer = request.env['payment.acquirer'].browse(acquirer_id)
        order_id = kwargs.get('order_id')
        invoice_id = kwargs.get('invoice_id')

        reference_values = order_id and {'sale_order_ids': [(4, order_id)]} or {}
        reference = request.env['payment.transaction']._compute_reference(values=reference_values, prefix=reference)

        values = {
            'acquirer_id': int(acquirer_id),
            'reference': reference,
            'amount': float(amount),
            'currency_id': int(currency_id),
            'partner_id': partner_id,
            'type': 'form_save' if acquirer.save_token != 'none' and partner_id else 'form',
        }

        if order_id:
            values['sale_order_ids'] = [(6, 0, [order_id])]
        elif invoice_id:
            values['invoice_ids'] = [(6, 0, [invoice_id])]

        reference_values = order_id and {'sale_order_ids': [(4, order_id)]} or {}
        reference_values.update(acquirer_id=int(acquirer_id))
        values['reference'] = request.env['payment.transaction']._compute_reference(values=reference_values, prefix=reference)
        tx = request.env['payment.transaction'].sudo().with_context(lang=None).create(values)
        secret = request.env['ir.config_parameter'].sudo().get_param('database.secret')
        token_str = '%s%s%s' % (tx.id, tx.reference, float_repr(tx.amount, precision_digits=tx.currency_id.decimal_places))
        token = hmac.new(secret.encode('utf-8'), token_str.encode('utf-8'), hashlib.sha256).hexdigest()
        tx.return_url = '/website_payment/confirm?tx_id=%d&access_token=%s' % (tx.id, token)

        PaymentProcessing.add_payment_transaction(tx)

        render_values = {
            'partner_id': partner_id,
            'type': tx.type,
        }

        return acquirer.sudo().render(tx.reference, float(amount), int(currency_id), values=render_values)

    @http.route(['/website_payment/token/<string:reference>/<string:amount>/<string:currency_id>',
                '/website_payment/token/v2/<string:amount>/<string:currency_id>/<path:reference>',
                '/website_payment/token/v2/<string:amount>/<string:currency_id>/<path:reference>/<int:partner_id>'], type='http', auth='public', website=True)
    def payment_token(self, pm_id, reference, amount, currency_id, partner_id=False, return_url=None, **kwargs):
        token = request.env['payment.token'].browse(int(pm_id))
        order_id = kwargs.get('order_id')
        invoice_id = kwargs.get('invoice_id')

        if not token:
            return request.redirect('/website_payment/pay?error_msg=%s' % _('Cannot setup the payment.'))

        values = {
            'acquirer_id': token.acquirer_id.id,
            'reference': reference,
            'amount': float(amount),
            'currency_id': int(currency_id),
            'partner_id': int(partner_id),
            'payment_token_id': int(pm_id),
            'type': 'server2server',
            'return_url': return_url,
        }

        if order_id:
            values['sale_order_ids'] = [(6, 0, [int(order_id)])]
        if invoice_id:
            values['invoice_ids'] = [(6, 0, [int(invoice_id)])]

        tx = request.env['payment.transaction'].sudo().with_context(lang=None).create(values)
        PaymentProcessing.add_payment_transaction(tx)

        try:
            tx.s2s_do_transaction()
            secret = request.env['ir.config_parameter'].sudo().get_param('database.secret')
            token_str = '%s%s%s' % (tx.id, tx.reference, float_repr(tx.amount, precision_digits=tx.currency_id.decimal_places))
            token = hmac.new(secret.encode('utf-8'), token_str.encode('utf-8'), hashlib.sha256).hexdigest()
            tx.return_url = return_url or '/website_payment/confirm?tx_id=%d&access_token=%s' % (tx.id, token)
        except Exception as e:
            _logger.exception(e)
        return request.redirect('/payment/process')

    @http.route(['/website_payment/confirm'], type='http', auth='public', website=True, sitemap=False)
    def confirm(self, **kw):
        tx_id = int(kw.get('tx_id', 0))
        access_token = kw.get('access_token')
        if tx_id:
            if access_token:
                tx = request.env['payment.transaction'].sudo().browse(tx_id)
                secret = request.env['ir.config_parameter'].sudo().get_param('database.secret')
                valid_token_str = '%s%s%s' % (tx.id, tx.reference, float_repr(tx.amount, precision_digits=tx.currency_id.decimal_places))
                valid_token = hmac.new(secret.encode('utf-8'), valid_token_str.encode('utf-8'), hashlib.sha256).hexdigest()
                if not consteq(ustr(valid_token), access_token):
                    raise werkzeug.exceptions.NotFound
            else:
                tx = request.env['payment.transaction'].browse(tx_id)
            if tx.state in ['done', 'authorized']:
                status = 'success'
                message = tx.acquirer_id.done_msg
            elif tx.state == 'pending':
                status = 'warning'
                message = tx.acquirer_id.pending_msg
            else:
                status = 'danger'
                message = tx.state_message or _('An error occured during the processing of this payment')
            PaymentProcessing.remove_payment_transaction(tx)
            return request.render('payment.confirm', {'tx': tx, 'status': status, 'message': message})
        else:
            return request.redirect('/my/home')

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import portal

```

## File: data\account_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="account_payment_method_electronic_in" model="account.payment.method">
        <field name="name">Electronic</field>
        <field name="code">electronic</field>
        <field name="payment_type">inbound</field>
    </record>
</odoo>

```

## File: data\payment_acquirer_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <template id="default_acquirer_button">
        <input type="hidden" name="data_set" t-att-data-action-url="tx_url"/>
        <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
        <t t-if="return_url">
            <input type="hidden" name="return_url" t-att-value="return_url"/>
        </t>
        <input type="hidden" name="reference" t-att-value="reference"/>
        <input type="hidden" name="amount" t-att-value="amount"/>
        <input type="hidden" name="currency" t-att-value="currency.name"/>
    </template>

    <record id="payment_acquirer_buckaroo" model="payment.acquirer">
        <field name="name">Buckaroo</field>
        <field name="display_as">Credit Card (powered by Buckaroo)</field>
        <field name="image_128" type="base64" file="payment_buckaroo/static/src/img/buckaroo_icon.png"/>
        <field name="view_template_id" ref="default_acquirer_button"/>
        <field name="module_id" ref="base.module_payment_buckaroo"/>
        <field name="description" type="html">
            <p>
                A payment gateway to accept online payments via credit cards.
            </p>
            <ul class="list-inline">
                <li class="list-inline-item"><i class="fa fa-check"/>Online Payment</li>
                <li class="list-inline-item"><i class="fa fa-check"/>Payment Status Tracking</li>
            </ul>
        </field>
        <!-- https://www.buckaroo-payments.com/products/payment-methods/ -->
        <field name="payment_icon_ids" eval='[(6, 0, [ref("payment.payment_icon_cc_bancontact"),
                                                        ref("payment.payment_icon_cc_maestro"),
                                                        ref("payment.payment_icon_cc_mastercard"),
                                                        ref("payment.payment_icon_cc_visa"),
                                                        ref("payment.payment_icon_cc_american_express")])]'/>
    </record>

    <record id="payment_acquirer_ingenico" model="payment.acquirer">
        <field name="name">Ingenico</field>
        <field name="display_as">Credit Card (powered by Ingenico)</field>
        <field name="sequence">2</field>
        <field name="view_template_id" ref="default_acquirer_button"/>
        <field name="image_128" type="base64" file="payment_ingenico/static/src/img/ingenico_icon.png"/>
        <field name="module_id" ref="base.module_payment_ingenico"/>
        <field name="description" type="html">
            <p>
                Ingenico Payment Services (formerly Ogone) supports a wide range of payment methods: credit cards, debit cards, bank transfers, Bancontact, iDeal, Giropay.
            </p>
            <ul class="list-inline">
                <li class="list-inline-item"><i class="fa fa-check"/>Online Payment</li>
                <li class="list-inline-item"><i class="fa fa-check"/>Payment Status Tracking</li>
                <li class="list-inline-item"><i class="fa fa-check"/>Subscriptions</li>
                <li class="list-inline-item"><i class="fa fa-check"/>Save Cards</li>
                <li class="list-inline-item"><i class="fa fa-check"/>Embedded Credit Card Form</li>
            </ul>
        </field>
        <!-- https://payment-services.ingenico.com/~/media/files/130806_product_sheet_ingenico_collect_en.ashx?la=en -->
        <field name="payment_icon_ids" eval='[(6, 0, [ref("payment.payment_icon_cc_ideal"),
                                                        ref("payment.payment_icon_cc_bancontact"),
                                                        ref("payment.payment_icon_cc_maestro"),
                                                        ref("payment.payment_icon_cc_mastercard"),
                                                        ref("payment.payment_icon_cc_visa")])]'/>
    </record>

    <record id="payment_acquirer_adyen" model="payment.acquirer">
        <field name="name">Adyen</field>
        <field name="display_as">Credit Card (powered by Adyen)</field>
        <field name="view_template_id" ref="default_acquirer_button"/>
        <field name="image_128" type="base64" file="payment_adyen/static/src/img/adyen_icon.png"/>
        <field name="module_id" ref="base.module_payment_adyen"/>
        <field name="description" type="html">
            <p>
                A payment gateway to accept online payments via credit cards, debit cards and bank transfers.
            </p>
            <ul class="list-inline">
                <li class="list-inline-item"><i class="fa fa-check"/>Online Payment</li>
                <li class="list-inline-item"><i class="fa fa-check"/>Payment Status Tracking</li>
            </ul>
        </field>
        <!-- https://www.adyen.com/payment-methods -->
        <field name="payment_icon_ids" eval='[(6, 0, [ref("payment.payment_icon_cc_bancontact"),
                                                        ref("payment.payment_icon_cc_maestro"),
                                                        ref("payment.payment_icon_cc_mastercard"),
                                                        ref("payment.payment_icon_cc_visa"),
                                                        ref("payment.payment_icon_cc_discover"),
                                                        ref("payment.payment_icon_cc_diners_club_intl"),
                                                        ref("payment.payment_icon_cc_jcb"),
                                                        ref("payment.payment_icon_cc_unionpay")])]'/>
    </record>

    <record id="payment_acquirer_authorize" model="payment.acquirer">
        <field name="name">Authorize.net</field>
        <field name="display_as">Credit Card (powered by Authorize)</field>
        <field name="sequence">3</field>
        <field name="view_template_id" ref="default_acquirer_button"/>
        <field name="image_128" type="base64" file="payment_authorize/static/src/img/authorize_icon.png"/>
        <field name="module_id" ref="base.module_payment_authorize"/>
        <field name="description" type="html">
            <p>
                A payment gateway to accept online payments via credit cards and e-checks.
            </p>
            <ul class="list-inline">
                <li class="list-inline-item"><i class="fa fa-check"/>Online Payment</li>
                <li class="list-inline-item"><i class="fa fa-check"/>Payment Status Tracking</li>
                <li class="list-inline-item"><i class="fa fa-check"/>Subscriptions</li>
                <li class="list-inline-item"><i class="fa fa-check"/>Save Cards</li>
                <li class="list-inline-item"><i class="fa fa-check"/>Manual Capture</li>
                <li class="list-inline-item"><i class="fa fa-check"/>Embedded Credit Card Form</li>
            </ul>
        </field>
        <!-- https://www.authorize.net/solutions/merchantsolutions/onlinemerchantaccount/ -->
        <field name="payment_icon_ids" eval='[(6, 0, [ref("payment.payment_icon_cc_maestro"),
                                                        ref("payment.payment_icon_cc_mastercard"),
                                                        ref("payment.payment_icon_cc_discover"),
                                                        ref("payment.payment_icon_cc_diners_club_intl"),
                                                        ref("payment.payment_icon_cc_jcb"),
                                                        ref("payment.payment_icon_cc_visa")])]'/>
    </record>

    <record id="payment_acquirer_transfer" model="payment.acquirer">
        <field name="name">Wire Transfer</field>
        <field name="sequence">2</field>
        <field name="view_template_id" ref="default_acquirer_button"/>
        <field name="image_128" type="base64" file="payment_transfer/static/src/img/transfer_icon.png"/>
        <field name="module_id" ref="base.module_payment_transfer"/>
        <field name="pending_msg">&lt;i&gt;Pending&lt;/i&gt;... The order will be validated after the payment.</field>
        <field name="description" type="html">
            <p>
                Provide instructions to customers so that they can pay their orders manually.
            </p>
        </field>
    </record>

    <record id="payment_acquirer_sips" model="payment.acquirer">
        <field name="name">Sips</field>
        <field name="display_as">Credit Card (powered by Sips)</field>
        <field name="view_template_id" ref="default_acquirer_button"/>
        <field name="image_128" type="base64" file="payment_sips/static/src/img/sips_icon.png"/>
        <field name="module_id" ref="base.module_payment_sips"/>
        <field name="description" type="html">
            <p>
                A payment gateway from Atos Worldline to accept online payments via credit cards.
            </p>
            <ul class="list-inline">
                <li class="list-inline-item"><i class="fa fa-check"/>Online Payment</li>
                <li class="list-inline-item"><i class="fa fa-check"/>Payment Status Tracking</li>
            </ul>
        </field>
        <!-- http://sips.worldline.com/en-us/home/features/payment-types-and-acquirers.html -->
        <field name="payment_icon_ids" eval='[(6, 0, [ref("payment.payment_icon_cc_maestro"),
                                                        ref("payment.payment_icon_cc_mastercard"),
                                                        ref("payment.payment_icon_cc_discover"),
                                                        ref("payment.payment_icon_cc_diners_club_intl"),
                                                        ref("payment.payment_icon_cc_jcb"),
                                                        ref("payment.payment_icon_cc_american_express"),
                                                        ref("payment.payment_icon_cc_bancontact"),
                                                        ref("payment.payment_icon_cc_unionpay"),
                                                        ref("payment.payment_icon_cc_visa")])]'/>
    </record>

    <record id="payment_acquirer_paypal" model="payment.acquirer">
        <field name="name">PayPal</field>
        <field name="sequence">1</field>
        <field name="view_template_id" ref="default_acquirer_button"/>
        <field name="image_128" type="base64" file="payment_paypal/static/src/img/paypal_icon.png"/>
        <field name="module_id" ref="base.module_payment_paypal"/>
        <field name="description" type="html">
            <p>
                PayPal is the easiest way to accept payments via Paypal or credit cards.
            </p>
            <ul class="list-inline">
                <li class="list-inline-item"><i class="fa fa-check"/>Online Payment</li>
                <li class="list-inline-item"><i class="fa fa-check"/>Payment Status Tracking</li>
            </ul>
        </field>
        <!-- https://www.paypal.com/us/selfhelp/article/Which-credit-cards-can-I-accept-with-PayPal-Merchant-Services-FAQ1525#business -->
        <field name="payment_icon_ids" eval='[(6, 0, [ref("payment.payment_icon_cc_maestro"),
                                                        ref("payment.payment_icon_cc_mastercard"),
                                                        ref("payment.payment_icon_cc_discover"),
                                                        ref("payment.payment_icon_cc_diners_club_intl"),
                                                        ref("payment.payment_icon_cc_jcb"),
                                                        ref("payment.payment_icon_cc_american_express"),
                                                        ref("payment.payment_icon_cc_unionpay"),
                                                        ref("payment.payment_icon_cc_visa")])]'/>
    </record>

    <record id="payment_acquirer_stripe" model="payment.acquirer">
        <field name="name">Stripe</field>
        <field name="display_as">Credit Card (powered by Stripe)</field>
        <field name="view_template_id" ref="default_acquirer_button"/>
        <field name="image_128" type="base64" file="payment_stripe/static/src/img/stripe_icon.png"/>
        <field name="module_id" ref="base.module_payment_stripe"/>
        <field name="description" type="html">
            <p>
                A payment gateway to accept online payments via credit cards.
            </p>
            <ul class="list-inline">
                <li class="list-inline-item"><i class="fa fa-check"/>Online Payment</li>
                <li class="list-inline-item"><i class="fa fa-check"/>Payment Status Tracking</li>
                <li class="list-inline-item"><i class="fa fa-check"/>Subscriptions</li>
                <li class="list-inline-item"><i class="fa fa-check"/>Save Cards</li>
            </ul>
        </field>
        <!--
            https://stripe.com/payments/payment-methods-guide
            https://support.goteamup.com/hc/en-us/articles/115002089349-Which-cards-and-payment-types-can-I-accept-with-Stripe-
        -->
        <field name="payment_icon_ids" eval='[(6, 0, [ref("payment.payment_icon_cc_maestro"),
                                                        ref("payment.payment_icon_cc_mastercard"),
                                                        ref("payment.payment_icon_cc_discover"),
                                                        ref("payment.payment_icon_cc_diners_club_intl"),
                                                        ref("payment.payment_icon_cc_jcb"),
                                                        ref("payment.payment_icon_cc_american_express"),
                                                        ref("payment.payment_icon_cc_visa")])]'/>
    </record>

    <record id="payment_acquirer_payu" model="payment.acquirer">
        <field name="name">PayUmoney</field>
        <field name="display_as">Credit Card (powered by PayUmoney)</field>
        <field name="image_128" type="base64" file="payment_payumoney/static/src/img/payumoney_icon.png"/>
        <field name="view_template_id" ref="default_acquirer_button"/>
        <field name="module_id" ref="base.module_payment_payumoney"/>
        <field name="description" type="html">
            <p>
                PayU India is an online payments solutions company serving the Indian market.
            </p>
            <ul class="list-inline">
                <li class="list-inline-item"><i class="fa fa-check"/>Online Payment</li>
                <li class="list-inline-item"><i class="fa fa-check"/>Payment Status Tracking</li>
            </ul>
        </field>
        <!-- https://www.payumoney.com/selfcare.html?userType=seller
            > Banks & Cards > What options do you have in the Credit Card payment?
         -->
        <field name="payment_icon_ids" eval='[(6, 0, [ref("payment.payment_icon_cc_maestro"),
                                                        ref("payment.payment_icon_cc_mastercard"),
                                                        ref("payment.payment_icon_cc_american_express"),
                                                        ref("payment.payment_icon_cc_visa")])]'/>
    </record>

    <record id="payment_acquirer_payulatam" model="payment.acquirer">
        <field name="name">PayU Latam</field>
        <field name="display_as">Credit Card (powered by PayU Latam)</field>
        <field name="image_128" type="base64" file="payment_payulatam/static/src/img/payulatam_icon.png"/>
        <field name="view_template_id" ref="default_acquirer_button"/>
        <field name="module_id" ref="base.module_payment_payulatam"/>
        <field name="description" type="html">
            <p>
                PayU is a leading financial services provider in Colombia, Argentina, Brazil, Chile, Mexico, Panama, and Peru. It allows merchant to accept local payments with just one account and integration.
            </p>
            <ul class="list-inline">
                <li class="list-inline-item"><i class="fa fa-check"/>Online Payment</li>
                <li class="list-inline-item"><i class="fa fa-check"/>Payment Status Tracking</li>
            </ul>
        </field>
        <!-- https://www.payulatam.com/medios-de-pago/ -->
        <field name="payment_icon_ids" eval='[(6, 0, [ref("payment.payment_icon_cc_diners_club_intl"),
                                                        ref("payment.payment_icon_cc_mastercard"),
                                                        ref("payment.payment_icon_cc_american_express"),
                                                        ref("payment.payment_icon_cc_visa"),
                                                        ref("payment.payment_icon_cc_codensa_easy_credit")])]'/>
    </record>

    <record id="payment_acquirer_alipay" model="payment.acquirer">
        <field name="name">Alipay</field>
        <field name="display_as">Credit Card (powered by Alipay)</field>
        <field name="image_128" type="base64" file="payment_alipay/static/description/icon.png"/>
        <field name="view_template_id" ref="default_acquirer_button"/>
        <field name="module_id" ref="base.module_payment_alipay"/>
        <field name="description" type="html">
            <p>
                Alipay is the most popular online payment platform in China. Chinese consumers can buy online using their Alipay eWallet.
            </p>
            <ul class="list-inline">
                <li><i class="fa fa-check"/>Online Payment</li>
                <li><i class="fa fa-check"/>Payment Status Tracking</li>
            </ul>
        </field>
        <!-- https://intl.alipay.com/ihome/home/about/buy.htm?topic=paymentMethods -->
        <field name="payment_icon_ids" eval='[(6, 0, [ref("payment.payment_icon_cc_jcb"),ref("payment.payment_icon_cc_mastercard"),ref("payment.payment_icon_cc_western_union"),ref("payment.payment_icon_cc_webmoney"),ref("payment.payment_icon_cc_visa")])]'/>
    </record>

    <record id="payment.payment_acquirer_sepa_direct_debit" model="payment.acquirer">
        <field name="name">SEPA Direct Debit</field>
        <field name="view_template_id" ref="default_acquirer_button"/>
        <field name="image_128" type="base64" file="base/static/img/icons/payment_sepa_direct_debit.png"/>
        <field name="module_id" ref="base.module_payment_sepa_direct_debit"/>
        <field name="description" type="html">
            <p>
                SEPA Direct Debit is a Europe-wide Direct Debit system that allows merchants to collect Euro-denominated payments from accounts in the 34 SEPA countries and associated territories.
            </p>
        </field>
    </record>

</odoo>

```

## File: data\payment_cron.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
  <record model="ir.cron" id="cron_post_process_payment_tx">
    <field name="name">Post process payment transactions</field>
    <field name="model_id" ref="payment.model_payment_transaction"/>
    <field name="state">code</field>
    <field name="code">model._cron_post_process_after_done()</field>
    <field name="user_id" ref="base.user_root"/>
    <field name="interval_number">10</field> <!-- To decide clearly -->
    <field name="interval_type">minutes</field>
    <field name="numbercall">-1</field>
  </record>
</odoo>

```

## File: data\payment_icon_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="payment_icon_cc_visa" model="payment.icon">
        <field name="name">VISA</field>
        <field name="image" type="base64" file="payment/static/img/visa.png"/>
    </record>

    <record id="payment_icon_cc_american_express" model="payment.icon">
        <field name="name">American Express</field>
        <field name="image" type="base64" file="payment/static/img/american_express.png"/>
    </record>

    <record id="payment_icon_cc_cirrus" model="payment.icon">
        <field name="name">Cirrus</field>
        <field name="image" type="base64" file="payment/static/img/cirrus.png"/>
    </record>

    <record id="payment_icon_cc_diners_club_intl" model="payment.icon">
        <field name="name">Diners Club International</field>
        <field name="image" type="base64" file="payment/static/img/diners_club_intl.png"/>
    </record>

    <record id="payment_icon_cc_discover" model="payment.icon">
        <field name="name">Discover</field>
        <field name="image" type="base64" file="payment/static/img/discover.png"/>
    </record>

    <record id="payment_icon_cc_jcb" model="payment.icon">
        <field name="name">JCB</field>
        <field name="image" type="base64" file="payment/static/img/jcb.png"/>
    </record>

    <record id="payment_icon_cc_maestro" model="payment.icon">
        <field name="name">Maestro</field>
        <field name="image" type="base64" file="payment/static/img/maestro.png"/>
    </record>

    <record id="payment_icon_cc_mastercard" model="payment.icon">
        <field name="name">MasterCard</field>
        <field name="image" type="base64" file="payment/static/img/mastercard.png"/>
    </record>

    <record id="payment_icon_cc_unionpay" model="payment.icon">
        <field name="name">UnionPay</field>
        <field name="image" type="base64" file="payment/static/img/unionpay.png"/>
    </record>

    <record id="payment_icon_cc_bancontact" model="payment.icon">
        <field name="name">Bancontact</field>
        <field name="image" type="base64" file="payment/static/img/bancontact.png"/>
    </record>

    <record id="payment_icon_cc_eps" model="payment.icon">
        <field name="name">EPS</field>
        <field name="image" type="base64" file="payment/static/img/eps.png"/>
    </record>

    <record id="payment_icon_cc_giropay" model="payment.icon">
        <field name="name">Giropay</field>
        <field name="image" type="base64" file="payment/static/img/giropay.png"/>
    </record>

    <record id="payment_icon_cc_p24" model="payment.icon">
        <field name="name">P24</field>
        <field name="image" type="base64" file="payment/static/img/p24.png"/>
    </record>

    <record id="payment_icon_cc_codensa_easy_credit" model="payment.icon">
        <field name="name">Codensa Easy Credit</field>
        <field name="image" type="base64" file="payment/static/img/codensa_easy_credit.png"/>
    </record>

    <record id="payment_icon_cc_western_union" model="payment.icon">
        <field name="name">Western Union</field>
        <field name="image" type="base64" file="payment/static/img/western_union.png"/>
    </record>

    <record id="payment_icon_cc_ideal" model="payment.icon">
        <field name="name">iDEAL</field>
        <field name="image" type="base64" file="payment/static/img/ideal.png"/>
    </record>

    <record id="payment_icon_cc_webmoney" model="payment.icon">
        <field name="name">WebMoney</field>
        <field name="image" type="base64" file="payment/static/img/webmoney.png"/>
    </record>
</odoo>

```

## File: models\account_invoice.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


class AccountMove(models.Model):
    _inherit = 'account.move'

    transaction_ids = fields.Many2many('payment.transaction', 'account_invoice_transaction_rel', 'invoice_id', 'transaction_id',
                                       string='Transactions', copy=False, readonly=True)
    authorized_transaction_ids = fields.Many2many('payment.transaction', compute='_compute_authorized_transaction_ids',
                                                  string='Authorized Transactions', copy=False, readonly=True)

    @api.depends('transaction_ids')
    def _compute_authorized_transaction_ids(self):
        for trans in self:
            trans.authorized_transaction_ids = trans.transaction_ids.filtered(lambda t: t.state == 'authorized')

    def get_portal_last_transaction(self):
        self.ensure_one()
        return self.with_context(active_test=False).transaction_ids.get_last_transaction()

    def _create_payment_transaction(self, vals):
        '''Similar to self.env['payment.transaction'].create(vals) but the values are filled with the
        current invoices fields (e.g. the partner or the currency).
        :param vals: The values to create a new payment.transaction.
        :return: The newly created payment.transaction record.
        '''
        # Ensure the currencies are the same.
        currency = self[0].currency_id
        if any([inv.currency_id != currency for inv in self]):
            raise ValidationError(_('A transaction can\'t be linked to invoices having different currencies.'))

        # Ensure the partner are the same.
        partner = self[0].partner_id
        if any([inv.partner_id != partner for inv in self]):
            raise ValidationError(_('A transaction can\'t be linked to invoices having different partners.'))

        # Try to retrieve the acquirer. However, fallback to the token's acquirer.
        acquirer_id = vals.get('acquirer_id')
        acquirer = None
        payment_token_id = vals.get('payment_token_id')

        if payment_token_id:
            payment_token = self.env['payment.token'].sudo().browse(payment_token_id)

            # Check payment_token/acquirer matching or take the acquirer from token
            if acquirer_id:
                acquirer = self.env['payment.acquirer'].browse(acquirer_id)
                if payment_token and payment_token.acquirer_id != acquirer:
                    raise ValidationError(_('Invalid token found! Token acquirer %s != %s') % (
                    payment_token.acquirer_id.name, acquirer.name))
            else:
                acquirer = payment_token.acquirer_id

            if payment_token and payment_token.partner_id != partner:
                raise ValidationError(_(
                    'The transaction was aborted because you are not the customer of this invoice. '
                    'Log in as %s to be able to use this payment method.'
                ) % partner.name)
        # Check an acquirer is there.
        if not acquirer_id and not acquirer:
            raise ValidationError(_('A payment acquirer is required to create a transaction.'))

        if not acquirer:
            acquirer = self.env['payment.acquirer'].browse(acquirer_id)

        # Check a journal is set on acquirer.
        if not acquirer.journal_id:
            raise ValidationError(_('A journal must be specified of the acquirer %s.' % acquirer.name))

        if not acquirer_id and acquirer:
            vals['acquirer_id'] = acquirer.id

        vals.update({
            'amount': sum(self.mapped('amount_residual')),
            'currency_id': currency.id,
            'partner_id': partner.id,
            'invoice_ids': [(6, 0, self.ids)],
        })

        transaction = self.env['payment.transaction'].create(vals)

        # Process directly if payment_token
        if transaction.payment_token_id:
            transaction.s2s_do_transaction()

        return transaction

    def payment_action_capture(self):
        self.authorized_transaction_ids.s2s_capture_transaction()

    def payment_action_void(self):
        self.authorized_transaction_ids.s2s_void_transaction()

```

## File: models\account_journal.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, _
from odoo.exceptions import ValidationError


class AccountJournal(models.Model):
    _inherit = 'account.journal'

    @api.constrains('type')
    def _check_journal_type_change(self):
        acquirer_incompatible_journals = self.filtered(lambda j: j.type not in ('bank', 'cash'))
        if acquirer_incompatible_journals and self.env['payment.acquirer'].search_count([('journal_id', 'in', acquirer_incompatible_journals.ids)]):
            raise ValidationError(_("An acquirer is using this journal. Only bank and cash types are allowed."))

```

## File: models\account_payment.py

```python
# coding: utf-8

import datetime

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError


class AccountPayment(models.Model):
    _inherit = 'account.payment'

    payment_transaction_id = fields.Many2one('payment.transaction', string='Payment Transaction', readonly=True)
    payment_token_id = fields.Many2one(
        'payment.token', string="Saved payment token",
        domain="[('acquirer_id.capture_manually', '=', False), ('company_id', '=', company_id)]",
        help="Note that tokens from acquirers set to only authorize transactions (instead of capturing the amount) are not available.")

    def _get_payment_chatter_link(self):
        self.ensure_one()
        return '<a href=# data-oe-model=account.payment data-oe-id=%d>%s</a>' % (self.id, self.name)

    @api.onchange('partner_id', 'payment_method_id', 'journal_id')
    def _onchange_set_payment_token_id(self):
        res = {}

        if not self.payment_method_code == 'electronic' or not self.partner_id or not self.journal_id:
            self.payment_token_id = False
            return res

        partners = self.partner_id | self.partner_id.commercial_partner_id | self.partner_id.commercial_partner_id.child_ids
        domain = [('partner_id', 'in', partners.ids), ('acquirer_id.journal_id', '=', self.journal_id.id)]
        self.payment_token_id = self.env['payment.token'].search(domain, limit=1)

        res['domain'] = {'payment_token_id': domain}
        return res

    def _prepare_payment_transaction_vals(self):
        self.ensure_one()
        return {
            'amount': self.amount,
            'currency_id': self.currency_id.id,
            'partner_id': self.partner_id.id,
            'partner_country_id': self.partner_id.country_id.id,
            'invoice_ids': [(6, 0, self.invoice_ids.ids)],
            'payment_token_id': self.payment_token_id.id,
            'acquirer_id': self.payment_token_id.acquirer_id.id,
            'payment_id': self.id,
            'type': 'server2server',
        }

    def _create_payment_transaction(self, vals=None):
        for pay in self:
            if pay.payment_transaction_id:
                raise ValidationError(_('A payment transaction already exists.'))
            elif not pay.payment_token_id:
                raise ValidationError(_('A token is required to create a new payment transaction.'))

        transactions = self.env['payment.transaction']
        for pay in self:
            transaction_vals = pay._prepare_payment_transaction_vals()

            if vals:
                transaction_vals.update(vals)

            transaction = self.env['payment.transaction'].create(transaction_vals)
            transactions += transaction

            # Link the transaction to the payment.
            pay.payment_transaction_id = transaction

        return transactions

    def action_validate_invoice_payment(self):
        res = super(AccountPayment, self).action_validate_invoice_payment()
        self.mapped('payment_transaction_id').filtered(lambda x: x.state == 'done' and not x.is_processed)._post_process_after_done()
        return res

    def post(self):
        # Post the payments "normally" if no transactions are needed.
        # If not, let the acquirer updates the state.
        #                                __________            ______________
        #                               | Payments |          | Transactions |
        #                               |__________|          |______________|
        #                                  ||                      |    |
        #                                  ||                      |    |
        #                                  ||                      |    |
        #  __________  no s2s required   __\/______   s2s required |    | s2s_do_transaction()
        # |  Posted  |<-----------------|  post()  |----------------    |
        # |__________|                  |__________|<-----              |
        #                                                |              |
        #                                               OR---------------
        #  __________                    __________      |
        # | Cancelled|<-----------------| cancel() |<-----
        # |__________|                  |__________|

        payments_need_trans = self.filtered(lambda pay: pay.payment_token_id and not pay.payment_transaction_id)
        transactions = payments_need_trans._create_payment_transaction()

        res = super(AccountPayment, self - payments_need_trans).post()

        transactions.s2s_do_transaction()

        return res

```

## File: models\chart_template.py

```python
# -*- coding: utf-8 -*-

from odoo import api, fields, models, _


class AccountChartTemplate(models.Model):
    _inherit = 'account.chart.template'

    def _create_bank_journals(self, company, acc_template_ref):
        res = super(AccountChartTemplate, self)._create_bank_journals(company, acc_template_ref)

        # Try to generate the missing journals
        return res + self.env['payment.acquirer']._create_missing_journal_for_acquirers(company=company)

```

## File: models\ir_http.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class IrHttp(models.AbstractModel):
    _inherit = 'ir.http'

    @classmethod
    def _get_translation_frontend_modules_name(cls):
        mods = super(IrHttp, cls)._get_translation_frontend_modules_name()
        return mods + ['payment']

```

## File: models\payment_acquirer.py

```python
# coding: utf-8
from collections import defaultdict
import hashlib
import hmac
import logging
from datetime import datetime
from dateutil import relativedelta
import pprint
import psycopg2

from odoo import api, exceptions, fields, models, _, SUPERUSER_ID
from odoo.tools import consteq, float_round, image_process, ustr
from odoo.addons.base.models import ir_module
from odoo.exceptions import ValidationError
from odoo.tools.misc import DEFAULT_SERVER_DATETIME_FORMAT
from odoo.tools.misc import formatLang
from odoo.http import request
from odoo.osv import expression

_logger = logging.getLogger(__name__)


def _partner_format_address(address1=False, address2=False):
    return ' '.join((address1 or '', address2 or '')).strip()


def _partner_split_name(partner_name):
    return [' '.join(partner_name.split()[:-1]), ' '.join(partner_name.split()[-1:])]


def create_missing_journal_for_acquirers(cr, registry):
    env = api.Environment(cr, SUPERUSER_ID, {})
    env['payment.acquirer']._create_missing_journal_for_acquirers()


class PaymentAcquirer(models.Model):
    """ Acquirer Model. Each specific acquirer can extend the model by adding
    its own fields, using the acquirer_name as a prefix for the new fields.
    Using the required_if_provider='<name>' attribute on fields it is possible
    to have required fields that depend on a specific acquirer.

    Each acquirer has a link to an ir.ui.view record that is a template of
    a button used to display the payment form. See examples in ``payment_ingenico``
    and ``payment_paypal`` modules.

    Methods that should be added in an acquirer-specific implementation:

     - ``<name>_form_generate_values(self, reference, amount, currency,
       partner_id=False, partner_values=None, tx_custom_values=None)``:
       method that generates the values used to render the form button template.
     - ``<name>_get_form_action_url(self):``: method that returns the url of
       the button form. It is used for example in ecommerce application if you
       want to post some data to the acquirer.
     - ``<name>_compute_fees(self, amount, currency_id, country_id)``: computes
       the fees of the acquirer, using generic fields defined on the acquirer
       model (see fields definition).

    Each acquirer should also define controllers to handle communication between
    OpenERP and the acquirer. It generally consists in return urls given to the
    button form and that the acquirer uses to send the customer back after the
    transaction, with transaction details given as a POST request.
    """
    _name = 'payment.acquirer'
    _description = 'Payment Acquirer'
    _order = 'module_state, state, sequence, name'

    def _get_default_view_template_id(self):
        return self.env.ref('payment.default_acquirer_button', raise_if_not_found=False)

    name = fields.Char('Name', required=True, translate=True)
    color = fields.Integer('Color', compute='_compute_color', store=True)
    display_as = fields.Char('Displayed as', translate=True, help="How the acquirer is displayed to the customers.")
    description = fields.Html('Description')
    sequence = fields.Integer('Sequence', default=10, help="Determine the display order")
    provider = fields.Selection(
        selection=[('manual', 'Custom Payment Form')], string='Provider',
        default='manual', required=True)
    company_id = fields.Many2one(
        'res.company', 'Company',
        default=lambda self: self.env.company.id, required=True)
    view_template_id = fields.Many2one(
        'ir.ui.view', 'Form Button Template',
        default=_get_default_view_template_id,
        help="This template renders the acquirer button with all necessary values.\n"
        "It is rendered with qWeb with the following evaluation context:\n"
        "tx_url: transaction URL to post the form\n"
        "acquirer: payment.acquirer browse record\n"
        "user: current user browse record\n"
        "reference: the transaction reference number\n"
        "currency: the transaction currency browse record\n"
        "amount: the transaction amount, a float\n"
        "partner: the buyer partner browse record, not necessarily set\n"
        "partner_values: specific values about the buyer, for example coming from a shipping form\n"
        "tx_values: transaction values\n"
        "context: the current context dictionary")
    registration_view_template_id = fields.Many2one(
        'ir.ui.view', 'S2S Form Template', domain=[('type', '=', 'qweb')],
        help="Template for method registration")
    state = fields.Selection([
        ('disabled', 'Disabled'),
        ('enabled', 'Enabled'),
        ('test', 'Test Mode')], required=True, default='disabled', copy=False,
        help="""In test mode, a fake payment is processed through a test
             payment interface. This mode is advised when setting up the
             acquirer. Watch out, test and production modes require
             different credentials.""")
    capture_manually = fields.Boolean(string="Capture Amount Manually",
        help="Capture the amount from Odoo, when the delivery is completed.")
    journal_id = fields.Many2one(
        'account.journal', 'Payment Journal', domain="[('type', 'in', ['bank', 'cash']), ('company_id', '=', company_id)]",
        help="""Journal where the successful transactions will be posted""")
    check_validity = fields.Boolean(string="Verify Card Validity",
        help="""Trigger a transaction of 1 currency unit and its refund to check the validity of new credit cards entered in the customer portal.
        Without this check, the validity will be verified at the very first transaction.""")
    country_ids = fields.Many2many(
        'res.country', 'payment_country_rel',
        'payment_id', 'country_id', 'Countries',
        help="This payment gateway is available for selected countries. If none is selected it is available for all countries.")

    pre_msg = fields.Html(
        'Help Message', translate=True,
        help='Message displayed to explain and help the payment process.')
    auth_msg = fields.Html(
        'Authorize Message', translate=True,
        default=lambda s: _('Your payment has been authorized.'),
        help='Message displayed if payment is authorized.')
    pending_msg = fields.Html(
        'Pending Message', translate=True,
        default=lambda s: _('Your payment has been successfully processed but is waiting for approval.'),
        help='Message displayed, if order is in pending state after having done the payment process.')
    done_msg = fields.Html(
        'Done Message', translate=True,
        default=lambda s: _('Your payment has been successfully processed. Thank you!'),
        help='Message displayed, if order is done successfully after having done the payment process.')
    cancel_msg = fields.Html(
        'Cancel Message', translate=True,
        default=lambda s: _('Your payment has been cancelled.'),
        help='Message displayed, if order is cancel during the payment process.')
    save_token = fields.Selection([
        ('none', 'Never'),
        ('ask', 'Let the customer decide'),
        ('always', 'Always')],
        string='Save Cards', default='none',
        help="This option allows customers to save their credit card as a payment token and to reuse it for a later purchase. "
             "If you manage subscriptions (recurring invoicing), you need it to automatically charge the customer when you "
             "issue an invoice.")
    token_implemented = fields.Boolean('Saving Card Data supported', compute='_compute_feature_support', search='_search_is_tokenized')
    authorize_implemented = fields.Boolean('Authorize Mechanism Supported', compute='_compute_feature_support')
    fees_implemented = fields.Boolean('Fees Computation Supported', compute='_compute_feature_support')
    fees_active = fields.Boolean('Add Extra Fees')
    fees_dom_fixed = fields.Float('Fixed domestic fees')
    fees_dom_var = fields.Float('Variable domestic fees (in percents)')
    fees_int_fixed = fields.Float('Fixed international fees')
    fees_int_var = fields.Float('Variable international fees (in percents)')
    qr_code = fields.Boolean('Use SEPA QR Code')

    # TDE FIXME: remove that brol
    module_id = fields.Many2one('ir.module.module', string='Corresponding Module')
    module_state = fields.Selection(selection=ir_module.STATES, string='Installation State', related='module_id.state', store=True)
    module_to_buy = fields.Boolean(string='Odoo Enterprise Module', related='module_id.to_buy', readonly=True, store=False)

    image_128 = fields.Image("Image", max_width=128, max_height=128)

    payment_icon_ids = fields.Many2many('payment.icon', string='Supported Payment Icons')
    payment_flow = fields.Selection(selection=[('form', 'Redirection to the acquirer website'),
        ('s2s','Payment from Odoo')],
        default='form', required=True, string='Payment Flow',
        help="""Note: Subscriptions does not take this field in account, it uses server to server by default.""")
    inbound_payment_method_ids = fields.Many2many('account.payment.method', related='journal_id.inbound_payment_method_ids', readonly=False)

    @api.onchange('payment_flow')
    def _onchange_payment_flow(self):
        electronic = self.env.ref('payment.account_payment_method_electronic_in')
        if self.token_implemented and self.payment_flow == 's2s':
            if electronic not in self.inbound_payment_method_ids:
                self.inbound_payment_method_ids = [(4, electronic.id)]
        elif electronic in self.inbound_payment_method_ids:
            self.inbound_payment_method_ids = [(2, electronic.id)]

    @api.onchange('state')
    def onchange_state(self):
        """Disable dashboard display for test acquirer journal."""
        self.journal_id.update({'show_on_dashboard': self.state == 'enabled'})

    def _search_is_tokenized(self, operator, value):
        tokenized = self._get_feature_support()['tokenize']
        if (operator, value) in [('=', True), ('!=', False)]:
            return [('provider', 'in', tokenized)]
        return [('provider', 'not in', tokenized)]

    @api.depends('provider')
    def _compute_feature_support(self):
        feature_support = self._get_feature_support()
        for acquirer in self:
            acquirer.fees_implemented = acquirer.provider in feature_support['fees']
            acquirer.authorize_implemented = acquirer.provider in feature_support['authorize']
            acquirer.token_implemented = acquirer.provider in feature_support['tokenize']

    @api.depends('state', 'module_state')
    def _compute_color(self):
        for acquirer in self:
            if acquirer.module_id and not acquirer.module_state == 'installed':
                acquirer.color = 4  # blue
            elif acquirer.state == 'disabled':
                acquirer.color = 3  # yellow
            elif acquirer.state == 'test':
                acquirer.color = 2  # orange
            elif acquirer.state == 'enabled':
                acquirer.color = 7  # green

    def _check_required_if_provider(self):
        """ If the field has 'required_if_provider="<provider>"' attribute, then it
        required if record.provider is <provider>. """
        field_names = []
        enabled_acquirers = self.filtered(lambda acq: acq.state in ['enabled', 'test'])
        for k, f in self._fields.items():
            provider = getattr(f, 'required_if_provider', None)
            if provider and any(
                acquirer.provider == provider and not acquirer[k]
                for acquirer in enabled_acquirers
            ):
                ir_field = self.env['ir.model.fields']._get(self._name, k)
                field_names.append(ir_field.field_description)
        if field_names:
            raise ValidationError(_("Required fields not filled: %s") % ", ".join(field_names))

    def get_base_url(self):
        self.ensure_one()
        # priority is always given to url_root
        # from the request
        url = ''
        if request:
            url = request.httprequest.url_root

        if not url and 'website_id' in self and self.website_id:
            url = self.website_id._get_http_domain()

        return url or self.env['ir.config_parameter'].sudo().get_param('web.base.url')

    def _get_feature_support(self):
        """Get advanced feature support by provider.

        Each provider should add its technical in the corresponding
        key for the following features:
            * fees: support payment fees computations
            * authorize: support authorizing payment (separates
                         authorization and capture)
            * tokenize: support saving payment data in a payment.tokenize
                        object
        """
        return dict(authorize=[], tokenize=[], fees=[])

    def _prepare_account_journal_vals(self):
        '''Prepare the values to create the acquirer's journal.
        :return: a dictionary to create a account.journal record.
        '''
        self.ensure_one()
        account_vals = self.company_id.chart_template_id._prepare_transfer_account_for_direct_creation(self.name, self.company_id)
        account = self.env['account.account'].create(account_vals)
        inbound_payment_method_ids = []
        if self.token_implemented and self.payment_flow == 's2s':
            inbound_payment_method_ids.append((4, self.env.ref('payment.account_payment_method_electronic_in').id))
        return {
            'name': self.name,
            'code': self.name.upper(),
            'sequence': 999,
            'type': 'bank',
            'company_id': self.company_id.id,
            'default_debit_account_id': account.id,
            'default_credit_account_id': account.id,
            # Show the journal on dashboard if the acquirer is published on the website.
            'show_on_dashboard': self.state == 'enabled',
            # Don't show payment methods in the backend.
            'inbound_payment_method_ids': inbound_payment_method_ids,
            'outbound_payment_method_ids': [],
        }

    def _get_acquirer_journal_domain(self):
        """Returns a domain for finding a journal corresponding to an acquirer"""
        self.ensure_one()
        code_cutoff = self.env['account.journal']._fields['code'].size
        return [
            ('name', '=', self.name),
            ('code', '=', self.name.upper()[:code_cutoff]),
            ('company_id', '=', self.company_id.id),
        ]

    @api.model
    def _create_missing_journal_for_acquirers(self, company=None):
        '''Create the journal for active acquirers.
        We want one journal per acquirer. However, we can't create them during the 'create' of the payment.acquirer
        because every acquirers are defined on the 'payment' module but is active only when installing their own module
        (e.g. payment_paypal for Paypal). We can't do that in such modules because we have no guarantee the chart template
        is already installed.
        '''
        # Search for installed acquirers modules that have no journal for the current company.
        # If this method is triggered by a post_init_hook, the module is 'to install'.
        # If the trigger comes from the chart template wizard, the modules are already installed.
        company = company or self.env.company
        acquirers = self.env['payment.acquirer'].search([
            ('module_state', 'in', ('to install', 'installed')),
            ('journal_id', '=', False),
            ('company_id', '=', company.id),
        ])

        # Here we will attempt to first create the journal since the most common case (first
        # install) is to successfully to create the journal for the acquirer, in the case of a
        # reinstall (least common case), the creation will fail because of a unique constraint
        # violation, this is ok as we catch the error and then perform a search if need be
        # and assign the existing journal to our reinstalled acquirer. It is better to ask for
        # forgiveness than to ask for permission as this saves us the overhead of doing a select
        # that would be useless in most cases.
        Journal = journals = self.env['account.journal']
        for acquirer in acquirers.filtered(lambda l: not l.journal_id and l.company_id.chart_template_id):
            try:
                with self.env.cr.savepoint():
                    journal = Journal.create(acquirer._prepare_account_journal_vals())
            except psycopg2.IntegrityError as e:
                if e.pgcode == psycopg2.errorcodes.UNIQUE_VIOLATION:
                    journal = Journal.search(acquirer._get_acquirer_journal_domain(), limit=1)
                else:
                    raise
            acquirer.journal_id = journal
            journals += journal
        return journals

    @api.model
    def create(self, vals):
        record = super(PaymentAcquirer, self).create(vals)
        record._check_required_if_provider()
        return record

    def write(self, vals):
        result = super(PaymentAcquirer, self).write(vals)
        self._check_required_if_provider()
        return result

    def get_acquirer_extra_fees(self, amount, currency_id, country_id):
        extra_fees = {
            'currency_id': currency_id
        }
        acquirers = self.filtered(lambda x: x.fees_active)
        for acq in acquirers:
            custom_method_name = '%s_compute_fees' % acq.provider
            if hasattr(acq, custom_method_name):
                fees = getattr(acq, custom_method_name)(amount, currency_id, country_id)
                extra_fees[acq] = fees
        return extra_fees

    def get_form_action_url(self):
        """ Returns the form action URL, for form-based acquirer implementations. """
        if hasattr(self, '%s_get_form_action_url' % self.provider):
            return getattr(self, '%s_get_form_action_url' % self.provider)()
        return False

    def _get_available_payment_input(self, partner=None, company=None):
        """ Generic (model) method that fetches available payment mechanisms
        to use in all portal / eshop pages that want to use the payment form.

        It contains

         * acquirers: record set of both form and s2s acquirers;
         * pms: record set of stored credit card data (aka payment.token)
                connected to a given partner to allow customers to reuse them """
        if not company:
            company = self.env.company
        if not partner:
            partner = self.env.user.partner_id

        domain = expression.AND([
            ['&', ('state', 'in', ['enabled', 'test']), ('company_id', '=', company.id)],
            ['|', ('country_ids', '=', False), ('country_ids', 'in', [partner.country_id.id])]
        ])
        active_acquirers = self.search(domain)
        acquirers = active_acquirers.filtered(lambda acq: (acq.payment_flow == 'form' and acq.view_template_id) or
                                                               (acq.payment_flow == 's2s' and acq.registration_view_template_id))
        return {
            'acquirers': acquirers,
            'pms': self.env['payment.token'].search([
                ('partner_id', '=', partner.id),
                ('acquirer_id', 'in', acquirers.ids)]),
        }

    def render(self, reference, amount, currency_id, partner_id=False, values=None):
        """ Renders the form template of the given acquirer as a qWeb template.
        :param string reference: the transaction reference
        :param float amount: the amount the buyer has to pay
        :param currency_id: currency id
        :param dict partner_id: optional partner_id to fill values
        :param dict values: a dictionary of values for the transction that is
        given to the acquirer-specific method generating the form values

        All templates will receive:

         - acquirer: the payment.acquirer browse record
         - user: the current user browse record
         - currency_id: id of the transaction currency
         - amount: amount of the transaction
         - reference: reference of the transaction
         - partner_*: partner-related values
         - partner: optional partner browse record
         - 'feedback_url': feedback URL, controler that manage answer of the acquirer (without base url) -> FIXME
         - 'return_url': URL for coming back after payment validation (wihout base url) -> FIXME
         - 'cancel_url': URL if the client cancels the payment -> FIXME
         - 'error_url': URL if there is an issue with the payment -> FIXME
         - context: Odoo context

        """
        if values is None:
            values = {}

        if not self.view_template_id:
            return None

        values.setdefault('return_url', '/payment/process')
        # reference and amount
        values.setdefault('reference', reference)
        amount = float_round(amount, 2)
        values.setdefault('amount', amount)

        # currency id
        currency_id = values.setdefault('currency_id', currency_id)
        if currency_id:
            currency = self.env['res.currency'].browse(currency_id)
        else:
            currency = self.env.company.currency_id
        values['currency'] = currency

        # Fill partner_* using values['partner_id'] or partner_id argument
        partner_id = values.get('partner_id', partner_id)
        billing_partner_id = values.get('billing_partner_id', partner_id)
        if partner_id:
            partner = self.env['res.partner'].browse(partner_id)
            if partner_id != billing_partner_id:
                billing_partner = self.env['res.partner'].browse(billing_partner_id)
            else:
                billing_partner = partner
            values.update({
                'partner': partner,
                'partner_id': partner_id,
                'partner_name': partner.name,
                'partner_lang': partner.lang,
                'partner_email': partner.email,
                'partner_zip': partner.zip,
                'partner_city': partner.city,
                'partner_address': _partner_format_address(partner.street, partner.street2),
                'partner_country_id': partner.country_id.id or self.env.company.country_id.id,
                'partner_country': partner.country_id,
                'partner_phone': partner.phone,
                'partner_state': partner.state_id,
                'billing_partner': billing_partner,
                'billing_partner_id': billing_partner_id,
                'billing_partner_name': billing_partner.name,
                'billing_partner_commercial_company_name': billing_partner.commercial_company_name,
                'billing_partner_lang': billing_partner.lang,
                'billing_partner_email': billing_partner.email,
                'billing_partner_zip': billing_partner.zip,
                'billing_partner_city': billing_partner.city,
                'billing_partner_address': _partner_format_address(billing_partner.street, billing_partner.street2),
                'billing_partner_country_id': billing_partner.country_id.id,
                'billing_partner_country': billing_partner.country_id,
                'billing_partner_phone': billing_partner.phone,
                'billing_partner_state': billing_partner.state_id,
            })
        if values.get('partner_name'):
            values.update({
                'partner_first_name': _partner_split_name(values.get('partner_name'))[0],
                'partner_last_name': _partner_split_name(values.get('partner_name'))[1],
            })
        if values.get('billing_partner_name'):
            values.update({
                'billing_partner_first_name': _partner_split_name(values.get('billing_partner_name'))[0],
                'billing_partner_last_name': _partner_split_name(values.get('billing_partner_name'))[1],
            })

        # Fix address, country fields
        if not values.get('partner_address'):
            values['address'] = _partner_format_address(values.get('partner_street', ''), values.get('partner_street2', ''))
        if not values.get('partner_country') and values.get('partner_country_id'):
            values['country'] = self.env['res.country'].browse(values.get('partner_country_id'))
        if not values.get('billing_partner_address'):
            values['billing_address'] = _partner_format_address(values.get('billing_partner_street', ''), values.get('billing_partner_street2', ''))
        if not values.get('billing_partner_country') and values.get('billing_partner_country_id'):
            values['billing_country'] = self.env['res.country'].browse(values.get('billing_partner_country_id'))

        # compute fees
        fees_method_name = '%s_compute_fees' % self.provider
        if hasattr(self, fees_method_name):
            fees = getattr(self, fees_method_name)(values['amount'], values['currency_id'], values.get('partner_country_id'))
            values['fees'] = float_round(fees, 2)

        # call <name>_form_generate_values to update the tx dict with acqurier specific values
        cust_method_name = '%s_form_generate_values' % (self.provider)
        if hasattr(self, cust_method_name):
            method = getattr(self, cust_method_name)
            values = method(values)

        values.update({
            'tx_url':  self._context.get(
                'tx_url', self.with_context(form_action_url_values=values).get_form_action_url()
            ),
            'submit_class': self._context.get('submit_class', 'btn btn-link'),
            'submit_txt': self._context.get('submit_txt'),
            'acquirer': self,
            'user': self.env.user,
            'context': self._context,
            'type': values.get('type') or 'form',
        })

        _logger.info('payment.acquirer.render: <%s> values rendered for form payment:\n%s', self.provider, pprint.pformat(values))
        return self.view_template_id.render(values, engine='ir.qweb')

    def get_s2s_form_xml_id(self):
        if self.registration_view_template_id:
            model_data = self.env['ir.model.data'].search([('model', '=', 'ir.ui.view'), ('res_id', '=', self.registration_view_template_id.id)])
            return ('%s.%s') % (model_data.module, model_data.name)
        return False

    def s2s_process(self, data):
        cust_method_name = '%s_s2s_form_process' % (self.provider)
        if not self.s2s_validate(data):
            return False
        if hasattr(self, cust_method_name):
            # As this method may be called in JSON and overridden in various addons
            # let us raise interesting errors before having stranges crashes
            if not data.get('partner_id'):
                raise ValueError(_('Missing partner reference when trying to create a new payment token'))
            method = getattr(self, cust_method_name)
            return method(data)
        return True

    def s2s_validate(self, data):
        cust_method_name = '%s_s2s_form_validate' % (self.provider)
        if hasattr(self, cust_method_name):
            method = getattr(self, cust_method_name)
            return method(data)
        return True

    def button_immediate_install(self):
        # TDE FIXME: remove that brol
        if self.module_id and self.module_state != 'installed':
            self.module_id.button_immediate_install()
            return {
                'type': 'ir.actions.client',
                'tag': 'reload',
            }

class PaymentIcon(models.Model):
    _name = 'payment.icon'
    _description = 'Payment Icon'

    name = fields.Char(string='Name')
    acquirer_ids = fields.Many2many('payment.acquirer', string="Acquirers", help="List of Acquirers supporting this payment icon.")
    image = fields.Binary(
        "Image", help="This field holds the image used for this payment icon, limited to 1024x1024px")

    image_payment_form = fields.Binary(
        "Image displayed on the payment form", attachment=True)

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            if 'image' in vals:
                image = ustr(vals['image'] or '').encode('utf-8')
                vals['image_payment_form'] = image_process(image, size=(45,30))
                vals['image'] = image_process(image, size=(64,64))
        return super(PaymentIcon, self).create(vals_list)

    def write(self, vals):
        if 'image' in vals:
            image = ustr(vals['image'] or '').encode('utf-8')
            vals['image_payment_form'] = image_process(image, size=(45,30))
            vals['image'] = image_process(image, size=(64,64))
        return super(PaymentIcon, self).write(vals)

class PaymentTransaction(models.Model):
    """ Transaction Model. Each specific acquirer can extend the model by adding
    its own fields.

    Methods that can be added in an acquirer-specific implementation:

     - ``<name>_create``: method receiving values used when creating a new
       transaction and that returns a dictionary that will update those values.
       This method can be used to tweak some transaction values.

    Methods defined for convention, depending on your controllers:

     - ``<name>_form_feedback(self, data)``: method that handles the data coming
       from the acquirer after the transaction. It will generally receives data
       posted by the acquirer after the transaction.
    """
    _name = 'payment.transaction'
    _description = 'Payment Transaction'
    _order = 'id desc'
    _rec_name = 'reference'

    @api.model
    def _lang_get(self):
        return self.env['res.lang'].get_installed()

    @api.model
    def _get_default_partner_country_id(self):
        return self.env.company.country_id.id

    date = fields.Datetime('Validation Date', readonly=True)
    acquirer_id = fields.Many2one('payment.acquirer', string='Acquirer', readonly=True, required=True)
    provider = fields.Selection(string='Provider', related='acquirer_id.provider', readonly=True)
    type = fields.Selection([
        ('validation', 'Validation of the bank card'),
        ('server2server', 'Server To Server'),
        ('form', 'Form'),
        ('form_save', 'Form with tokenization')], 'Type',
        default='form', required=True, readonly=True)
    state = fields.Selection([
        ('draft', 'Draft'),
        ('pending', 'Pending'),
        ('authorized', 'Authorized'),
        ('done', 'Done'),
        ('cancel', 'Canceled'),
        ('error', 'Error'),],
        string='Status', copy=False, default='draft', required=True, readonly=True)
    state_message = fields.Text(string='Message', readonly=True,
                                help='Field used to store error and/or validation messages for information')
    amount = fields.Monetary(string='Amount', currency_field='currency_id', required=True, readonly=True)
    fees = fields.Monetary(string='Fees', currency_field='currency_id', readonly=True,
                           help='Fees amount; set by the system because depends on the acquirer')
    currency_id = fields.Many2one('res.currency', 'Currency', required=True, readonly=True)
    reference = fields.Char(string='Reference', required=True, readonly=True, index=True,
                            help='Internal reference of the TX')
    acquirer_reference = fields.Char(string='Acquirer Reference', readonly=True, help='Reference of the TX as stored in the acquirer database')
    # duplicate partner / transaction data to store the values at transaction time
    partner_id = fields.Many2one('res.partner', 'Customer', tracking=True)
    partner_name = fields.Char('Partner Name')
    partner_lang = fields.Selection(_lang_get, 'Language', default=lambda self: self.env.lang)
    partner_email = fields.Char('Email')
    partner_zip = fields.Char('Zip')
    partner_address = fields.Char('Address')
    partner_city = fields.Char('City')
    partner_country_id = fields.Many2one('res.country', 'Country', default=_get_default_partner_country_id, required=True)
    partner_phone = fields.Char('Phone')
    html_3ds = fields.Char('3D Secure HTML')

    callback_model_id = fields.Many2one('ir.model', 'Callback Document Model', groups="base.group_system")
    callback_res_id = fields.Integer('Callback Document ID', groups="base.group_system")
    callback_method = fields.Char('Callback Method', groups="base.group_system")
    callback_hash = fields.Char('Callback Hash', groups="base.group_system")

    # Fields used for user redirection & payment post processing
    return_url = fields.Char('Return URL after payment')
    is_processed = fields.Boolean('Has the payment been post processed', default=False)

    # Fields used for payment.transaction traceability.

    payment_token_id = fields.Many2one('payment.token', 'Payment Token', readonly=True,
                                       domain="[('acquirer_id', '=', acquirer_id)]")

    payment_id = fields.Many2one('account.payment', string='Payment', readonly=True)
    invoice_ids = fields.Many2many('account.move', 'account_invoice_transaction_rel', 'transaction_id', 'invoice_id',
        string='Invoices', copy=False, readonly=True,
        domain=[('type', 'in', ('out_invoice', 'out_refund', 'in_invoice', 'in_refund'))])
    invoice_ids_nbr = fields.Integer(compute='_compute_invoice_ids_nbr', string='# of Invoices')

    _sql_constraints = [
        ('reference_uniq', 'unique(reference)', 'Reference must be unique!'),
    ]

    @api.depends('invoice_ids')
    def _compute_invoice_ids_nbr(self):
        for trans in self:
            trans.invoice_ids_nbr = len(trans.invoice_ids)

    def _prepare_account_payment_vals(self):
        self.ensure_one()
        return {
            'amount': self.amount,
            'payment_type': 'inbound' if self.amount > 0 else 'outbound',
            'currency_id': self.currency_id.id,
            'partner_id': self.partner_id.id,
            'partner_type': 'customer',
            'invoice_ids': [(6, 0, self.invoice_ids.ids)],
            'journal_id': self.acquirer_id.journal_id.id,
            'company_id': self.acquirer_id.company_id.id,
            'payment_method_id': self.env.ref('payment.account_payment_method_electronic_in').id,
            'payment_token_id': self.payment_token_id and self.payment_token_id.id or None,
            'payment_transaction_id': self.id,
            'communication': self.reference,
        }

    def get_last_transaction(self):
        transactions = self.filtered(lambda t: t.state != 'draft')
        return transactions and transactions[0] or transactions

    def _get_processing_info(self):
        """ Extensible method for providers if they need specific fields/info regarding a tx in the payment processing page. """
        return dict()

    def _get_payment_transaction_sent_message(self):
        self.ensure_one()
        if self.payment_token_id:
            message = _('A transaction %s with %s initiated using %s credit card.')
            message_vals = (self.reference, self.acquirer_id.name, self.payment_token_id.name)
        elif self.provider in ('manual', 'transfer'):
            message = _('The customer has selected %s to pay this document.')
            message_vals = (self.acquirer_id.name)
        else:
            message = _('A transaction %s with %s initiated.')
            message_vals = (self.reference, self.acquirer_id.name)
        if self.provider not in ('manual', 'transfer'):
            message += ' ' + _('Waiting for payment confirmation...')
        return message % message_vals

    def _get_payment_transaction_received_message(self):
        self.ensure_one()
        amount = formatLang(self.env, self.amount, currency_obj=self.currency_id)
        message_vals = [self.reference, self.acquirer_id.name, amount]
        if self.state == 'pending':
            message = _('The transaction %s with %s for %s is pending.')
        elif self.state == 'authorized':
            message = _('The transaction %s with %s for %s has been authorized. Waiting for capture...')
        elif self.state == 'done':
            message = _('The transaction %s with %s for %s has been confirmed. The related payment is posted: %s')
            message_vals.append(self.payment_id._get_payment_chatter_link())
        elif self.state == 'cancel' and self.state_message:
            message = _('The transaction %s with %s for %s has been cancelled with the following message: %s')
            message_vals.append(self.state_message)
        elif self.state == 'error' and self.state_message:
            message = _('The transaction %s with %s for %s has return failed with the following error message: %s')
            message_vals.append(self.state_message)
        else:
            message = _('The transaction %s with %s for %s has been cancelled.')
        return message % tuple(message_vals)

    def _log_payment_transaction_sent(self):
        '''Log the message saying the transaction has been sent to the remote server to be
        processed by the acquirer.
        '''
        for trans in self:
            post_message = trans._get_payment_transaction_sent_message()
            for inv in trans.invoice_ids:
                inv.message_post(body=post_message)

    def _log_payment_transaction_received(self):
        '''Log the message saying a response has been received from the remote server and some
        additional informations like the old/new state, the reference of the payment... etc.
        :param old_state:       The state of the transaction before the response.
        :param add_messages:    Optional additional messages to log like the capture status.
        '''
        for trans in self.filtered(lambda t: t.provider not in ('manual', 'transfer')):
            post_message = trans._get_payment_transaction_received_message()
            for inv in trans.invoice_ids:
                inv.message_post(body=post_message)

    def _filter_transaction_state(self, allowed_states, target_state):
        """Divide a set of transactions according to their state.

        :param tuple(string) allowed_states: tuple of allowed states for the target state (strings)
        :param string target_state: target state for the filtering
        :return: tuple of transactions divided by their state, in that order
                    tx_to_process: tx that were in the allowed states
                    tx_already_processed: tx that were already in the target state
                    tx_wrong_state: tx that were not in the allowed state for the transition
        :rtype: tuple(recordset)
        """
        tx_to_process = self.filtered(lambda tx: tx.state in allowed_states)
        tx_already_processed = self.filtered(lambda tx: tx.state == target_state)
        tx_wrong_state = self -tx_to_process - tx_already_processed
        return (tx_to_process, tx_already_processed, tx_wrong_state)

    def _set_transaction_pending(self):
        '''Move the transaction to the pending state(e.g. Wire Transfer).'''
        allowed_states = ('draft',)
        target_state = 'pending'
        (tx_to_process, tx_already_processed, tx_wrong_state) = self._filter_transaction_state(allowed_states, target_state)
        for tx in tx_already_processed:
            _logger.info('Trying to write the same state twice on tx (ref: %s, state: %s' % (tx.reference, tx.state))
        for tx in tx_wrong_state:
            _logger.warning('Processed tx with abnormal state (ref: %s, target state: %s, previous state %s, expected previous states: %s)' % (tx.reference, target_state, tx.state, allowed_states))

        tx_to_process.write({
            'state': target_state,
            'date': fields.Datetime.now(),
            'state_message': '',
        })
        tx_to_process._log_payment_transaction_received()

    def _set_transaction_authorized(self):
        '''Move the transaction to the authorized state(e.g. Authorize).'''
        allowed_states = ('draft', 'pending')
        target_state = 'authorized'
        (tx_to_process, tx_already_processed, tx_wrong_state) = self._filter_transaction_state(allowed_states, target_state)
        for tx in tx_already_processed:
            _logger.info('Trying to write the same state twice on tx (ref: %s, state: %s' % (tx.reference, tx.state))
        for tx in tx_wrong_state:
            _logger.warning('Processed tx with abnormal state (ref: %s, target state: %s, previous state %s, expected previous states: %s)' % (tx.reference, target_state, tx.state, allowed_states))
        tx_to_process.write({
            'state': target_state,
            'date': fields.Datetime.now(),
            'state_message': '',
        })
        tx_to_process._log_payment_transaction_received()

    def _set_transaction_done(self):
        '''Move the transaction's payment to the done state(e.g. Paypal).'''
        allowed_states = ('draft', 'authorized', 'pending', 'error')
        target_state = 'done'
        (tx_to_process, tx_already_processed, tx_wrong_state) = self._filter_transaction_state(allowed_states, target_state)
        for tx in tx_already_processed:
            _logger.info('Trying to write the same state twice on tx (ref: %s, state: %s' % (tx.reference, tx.state))
        for tx in tx_wrong_state:
            _logger.warning('Processed tx with abnormal state (ref: %s, target state: %s, previous state %s, expected previous states: %s)' % (tx.reference, target_state, tx.state, allowed_states))

        tx_to_process.write({
            'state': target_state,
            'date': fields.Datetime.now(),
            'state_message': '',
        })

    def _reconcile_after_transaction_done(self):
        # Validate invoices automatically upon the transaction is posted.
        invoices = self.mapped('invoice_ids').filtered(lambda inv: inv.state == 'draft')
        invoices.post()

        # Create & Post the payments.
        payments = defaultdict(lambda: self.env['account.payment'])
        for trans in self:
            if trans.payment_id:
                payments[trans.acquirer_id.company_id.id] += trans.payment_id
                continue

            payment_vals = trans._prepare_account_payment_vals()
            payment = self.env['account.payment'].create(payment_vals)
            payments[trans.acquirer_id.company_id.id] += payment

            # Track the payment to make a one2one.
            trans.payment_id = payment

        for company in payments:
            payments[company].with_context(force_company=company, company_id=company).post()

    def _set_transaction_cancel(self):
        '''Move the transaction's payment to the cancel state(e.g. Paypal).'''
        allowed_states = ('draft', 'authorized')
        target_state = 'cancel'
        (tx_to_process, tx_already_processed, tx_wrong_state) = self._filter_transaction_state(allowed_states, target_state)
        for tx in tx_already_processed:
            _logger.info('Trying to write the same state twice on tx (ref: %s, state: %s' % (tx.reference, tx.state))
        for tx in tx_wrong_state:
            _logger.warning('Processed tx with abnormal state (ref: %s, target state: %s, previous state %s, expected previous states: %s)' % (tx.reference, target_state, tx.state, allowed_states))

        # Cancel the existing payments.
        tx_to_process.mapped('payment_id').cancel()

        tx_to_process.write({'state': target_state, 'date': fields.Datetime.now()})
        tx_to_process._log_payment_transaction_received()

    def _set_transaction_error(self, msg):
        '''Move the transaction to the error state (Third party returning error e.g. Paypal).'''
        allowed_states = ('draft', 'authorized', 'pending')
        target_state = 'error'
        (tx_to_process, tx_already_processed, tx_wrong_state) = self._filter_transaction_state(allowed_states, target_state)
        for tx in tx_already_processed:
            _logger.info('Trying to write the same state twice on tx (ref: %s, state: %s' % (tx.reference, tx.state))
        for tx in tx_wrong_state:
            _logger.warning('Processed tx with abnormal state (ref: %s, target state: %s, previous state %s, expected previous states: %s)' % (tx.reference, target_state, tx.state, allowed_states))

        tx_to_process.write({
            'state': target_state,
            'date': fields.Datetime.now(),
            'state_message': msg,
        })
        self._log_payment_transaction_received()

    def _post_process_after_done(self):
        self._reconcile_after_transaction_done()
        self._log_payment_transaction_received()
        self.write({'is_processed': True})
        return True

    def _cron_post_process_after_done(self):
        if not self:
            ten_minutes_ago = datetime.now() - relativedelta.relativedelta(minutes=10)
            # we don't want to forever try to process a transaction that doesn't go through
            retry_limit_date = datetime.now() - relativedelta.relativedelta(days=2)
            # we retrieve all the payment tx that need to be post processed
            self = self.search([('state', '=', 'done'),
                                ('is_processed', '=', False),
                                ('date', '<=', ten_minutes_ago),
                                ('date', '>=', retry_limit_date),
                            ])
        for tx in self:
            try:
                tx._post_process_after_done()
                self.env.cr.commit()
            except Exception as e:
                _logger.exception("Transaction post processing failed")
                self.env.cr.rollback()

    @api.model
    def _compute_reference_prefix(self, values):
        if values and values.get('invoice_ids'):
            many_list = self.resolve_2many_commands('invoice_ids', values['invoice_ids'], fields=['name'])
            return ','.join(dic['name'] for dic in many_list)
        return None

    @api.model
    def _compute_reference(self, values=None, prefix=None):
        '''Compute a unique reference for the transaction.
        If prefix:
            prefix-\d+
        If some invoices:
            <inv_number_0>.number,<inv_number_1>,...,<inv_number_n>-x
        If some sale orders:
            <so_name_0>.number,<so_name_1>,...,<so_name_n>-x
        Else:
            tx-\d+
        :param values: values used to create a new transaction.
        :param prefix: custom transaction prefix.
        :return: A unique reference for the transaction.
        '''
        if not prefix:
            if values:
                prefix = self._compute_reference_prefix(values)
            else:
                prefix = 'tx'

        # Fetch the last reference
        # E.g. If the last reference is SO42-5, this query will return '-5'
        self._cr.execute('''
                SELECT CAST(SUBSTRING(reference FROM '-\d+$') AS INTEGER) AS suffix
                FROM payment_transaction WHERE reference LIKE %s ORDER BY suffix
            ''', [prefix + '-%'])
        query_res = self._cr.fetchone()
        if query_res:
            # Increment the last reference by one
            suffix = '%s' % (-query_res[0] + 1)
        else:
            # Start a new indexing from 1
            suffix = '1'

        return '%s-%s' % (prefix, suffix)

    def action_view_invoices(self):
        action = {
            'name': _('Invoices'),
            'type': 'ir.actions.act_window',
            'res_model': 'account.move',
            'target': 'current',
        }
        invoice_ids = self.invoice_ids.ids
        if len(invoice_ids) == 1:
            invoice = invoice_ids[0]
            action['res_id'] = invoice
            action['view_mode'] = 'form'
            form_view = [(self.env.ref('account.view_move_form').id, 'form')]
            if 'views' in action:
                action['views'] = form_view + [(state,view) for state,view in action['views'] if view != 'form']
            else:
                action['views'] = form_view
        else:
            action['view_mode'] = 'tree,form'
            action['domain'] = [('id', 'in', invoice_ids)]
        return action

    @api.constrains('state', 'acquirer_id')
    def _check_authorize_state(self):
        failed_tx = self.filtered(lambda tx: tx.state == 'authorized' and tx.acquirer_id.provider not in self.env['payment.acquirer']._get_feature_support()['authorize'])
        if failed_tx:
            raise exceptions.ValidationError(_('The %s payment acquirers are not allowed to manual capture mode!' % failed_tx.mapped('acquirer_id.name')))

    @api.model
    def create(self, values):
        # call custom create method if defined
        acquirer = self.env['payment.acquirer'].browse(values['acquirer_id'])
        if values.get('partner_id'):
            partner = self.env['res.partner'].browse(values['partner_id'])

            values.update({
                'partner_name': partner.name,
                'partner_lang': partner.lang or self.env.user.lang,
                'partner_email': partner.email,
                'partner_zip': partner.zip,
                'partner_address': _partner_format_address(partner.street or '', partner.street2 or ''),
                'partner_city': partner.city,
                'partner_country_id': partner.country_id.id or self._get_default_partner_country_id(),
                'partner_phone': partner.phone,
            })

        # compute fees
        custom_method_name = '%s_compute_fees' % acquirer.provider
        if hasattr(acquirer, custom_method_name):
            fees = getattr(acquirer, custom_method_name)(
                values.get('amount', 0.0), values.get('currency_id'), values['partner_country_id'])
            values['fees'] = fees

        # custom create
        custom_method_name = '%s_create' % acquirer.provider
        if hasattr(self, custom_method_name):
            values.update(getattr(self, custom_method_name)(values))

        if not values.get('reference'):
            values['reference'] = self._compute_reference(values=values)

        # Default value of reference is
        tx = super(PaymentTransaction, self).create(values)

        # Generate callback hash if it is configured on the tx; avoid generating unnecessary stuff
        # (limited sudo env for checking callback presence, must work for manual transactions too)
        tx_sudo = tx.sudo()
        if tx_sudo.callback_model_id and tx_sudo.callback_res_id and tx_sudo.callback_method:
            tx.write({'callback_hash': tx._generate_callback_hash()})

        return tx

    def _generate_callback_hash(self):
        self.ensure_one()
        secret = self.env['ir.config_parameter'].sudo().get_param('database.secret')
        token = '%s%s%s' % (self.callback_model_id.model,
                            self.callback_res_id,
                            self.sudo().callback_method)
        return hmac.new(secret.encode('utf-8'), token.encode('utf-8'), hashlib.sha256).hexdigest()

    # --------------------------------------------------
    # FORM RELATED METHODS
    # --------------------------------------------------

    @api.model
    def form_feedback(self, data, acquirer_name):
        invalid_parameters, tx = None, None

        tx_find_method_name = '_%s_form_get_tx_from_data' % acquirer_name
        if hasattr(self, tx_find_method_name):
            tx = getattr(self, tx_find_method_name)(data)

        # TDE TODO: form_get_invalid_parameters from model to multi
        invalid_param_method_name = '_%s_form_get_invalid_parameters' % acquirer_name
        if hasattr(self, invalid_param_method_name):
            invalid_parameters = getattr(tx, invalid_param_method_name)(data)

        if invalid_parameters:
            _error_message = '%s: incorrect tx data:\n' % (acquirer_name)
            for item in invalid_parameters:
                _error_message += '\t%s: received %s instead of %s\n' % (item[0], item[1], item[2])
            _logger.error(_error_message)
            return False

        # TDE TODO: form_validate from model to multi
        feedback_method_name = '_%s_form_validate' % acquirer_name
        if hasattr(self, feedback_method_name):
            return getattr(tx, feedback_method_name)(data)

        return True

    # --------------------------------------------------
    # SERVER2SERVER RELATED METHODS
    # --------------------------------------------------

    def s2s_do_transaction(self, **kwargs):
        custom_method_name = '%s_s2s_do_transaction' % self.acquirer_id.provider
        for trans in self:
            trans._log_payment_transaction_sent()
            if hasattr(trans, custom_method_name):
                return getattr(trans, custom_method_name)(**kwargs)

    def s2s_do_refund(self, **kwargs):
        custom_method_name = '%s_s2s_do_refund' % self.acquirer_id.provider
        if hasattr(self, custom_method_name):
            return getattr(self, custom_method_name)(**kwargs)

    def s2s_capture_transaction(self, **kwargs):
        custom_method_name = '%s_s2s_capture_transaction' % self.acquirer_id.provider
        if hasattr(self, custom_method_name):
            return getattr(self, custom_method_name)(**kwargs)

    def s2s_void_transaction(self, **kwargs):
        custom_method_name = '%s_s2s_void_transaction' % self.acquirer_id.provider
        if hasattr(self, custom_method_name):
            return getattr(self, custom_method_name)(**kwargs)

    def s2s_get_tx_status(self):
        """ Get the tx status. """
        invalid_param_method_name = '_%s_s2s_get_tx_status' % self.acquirer_id.provider
        if hasattr(self, invalid_param_method_name):
            return getattr(self, invalid_param_method_name)()
        return True

    def execute_callback(self):
        res = None
        for transaction in self:
            # limited sudo env, only for checking callback presence, not for running it!
            # manual transactions have no callback, and can pass without being run by admin user
            tx_sudo = transaction.sudo()
            if not (tx_sudo.callback_model_id and tx_sudo.callback_res_id and tx_sudo.callback_method):
                continue

            valid_token = transaction._generate_callback_hash()
            if not consteq(ustr(valid_token), transaction.callback_hash):
                _logger.warning("Invalid callback signature for transaction %d" % (transaction.id))
                continue

            record = self.env[transaction.callback_model_id.model].browse(transaction.callback_res_id).exists()
            if record:
                res = getattr(record, transaction.callback_method)(transaction)
            else:
                _logger.warning("Did not found record %s.%s for callback of transaction %d" % (transaction.callback_model_id.model, transaction.callback_res_id, transaction.id))
        return res

    def action_capture(self):
        if any([t.state != 'authorized' for t in self]):
            raise ValidationError(_('Only transactions having the capture status can be captured.'))
        for tx in self:
            tx.s2s_capture_transaction()

    def action_void(self):
        if any([t.state != 'authorized' for t in self]):
            raise ValidationError(_('Only transactions having the capture status can be voided.'))
        for tx in self:
            tx.s2s_void_transaction()


class PaymentToken(models.Model):
    _name = 'payment.token'
    _order = 'partner_id, id desc'
    _description = 'Payment Token'

    name = fields.Char('Name', help='Name of the payment token')
    short_name = fields.Char('Short name', compute='_compute_short_name')
    partner_id = fields.Many2one('res.partner', 'Partner', required=True)
    acquirer_id = fields.Many2one('payment.acquirer', 'Acquirer Account', required=True)
    company_id = fields.Many2one(related='acquirer_id.company_id', store=True, index=True)
    acquirer_ref = fields.Char('Acquirer Ref.', required=True)
    active = fields.Boolean('Active', default=True)
    payment_ids = fields.One2many('payment.transaction', 'payment_token_id', 'Payment Transactions')
    verified = fields.Boolean(string='Verified', default=False)

    @api.model
    def create(self, values):
        # call custom create method if defined
        if values.get('acquirer_id'):
            acquirer = self.env['payment.acquirer'].browse(values['acquirer_id'])

            # custom create
            custom_method_name = '%s_create' % acquirer.provider
            if hasattr(self, custom_method_name):
                values.update(getattr(self, custom_method_name)(values))
                # remove all non-model fields used by (provider)_create method to avoid warning
                fields_wl = set(self._fields) & set(values)
                values = {field: values[field] for field in fields_wl}
        return super(PaymentToken, self).create(values)
    """
        @TBE: stolen shamelessly from there https://www.paypal.com/us/selfhelp/article/why-is-there-a-$1.95-charge-on-my-card-statement-faq554
        Most of them are ~1.50€s
        TODO: See this with @AL & @DBO
    """
    VALIDATION_AMOUNTS = {
        'CAD': 2.45,
        'EUR': 1.50,
        'GBP': 1.00,
        'JPY': 200,
        'AUD': 2.00,
        'NZD': 3.00,
        'CHF': 3.00,
        'HKD': 15.00,
        'SEK': 15.00,
        'DKK': 12.50,
        'PLN': 6.50,
        'NOK': 15.00,
        'HUF': 400.00,
        'CZK': 50.00,
        'BRL': 4.00,
        'MYR': 10.00,
        'MXN': 20.00,
        'ILS': 8.00,
        'PHP': 100.00,
        'TWD': 70.00,
        'THB': 70.00
        }

    @api.model
    def validate(self, **kwargs):
        """
            This method allow to verify if this payment method is valid or not.
            It does this by withdrawing a certain amount and then refund it right after.
        """
        currency = self.partner_id.currency_id

        if self.VALIDATION_AMOUNTS.get(currency.name):
            amount = self.VALIDATION_AMOUNTS.get(currency.name)
        else:
            # If we don't find the user's currency, then we set the currency to EUR and the amount to 1€50.
            currency = self.env['res.currency'].search([('name', '=', 'EUR')])
            amount = 1.5

        if len(currency) != 1:
            _logger.error("Error 'EUR' currency not found for payment method validation!")
            return False

        reference = "VALIDATION-%s-%s" % (self.id, datetime.now().strftime('%y%m%d_%H%M%S'))
        tx = self.env['payment.transaction'].sudo().create({
            'amount': amount,
            'acquirer_id': self.acquirer_id.id,
            'type': 'validation',
            'currency_id': currency.id,
            'reference': reference,
            'payment_token_id': self.id,
            'partner_id': self.partner_id.id,
            'partner_country_id': self.partner_id.country_id.id,
            'state_message': _('This Transaction was automatically processed & refunded in order to validate a new credit card.'),
        })

        kwargs.update({'3d_secure': True})
        tx.s2s_do_transaction(**kwargs)

        # if 3D secure is called, then we do not refund right now
        if not tx.html_3ds:
            tx.s2s_do_refund()

        return tx

    @api.depends('name')
    def _compute_short_name(self):
        for token in self:
            token.short_name = token.name.replace('XXXXXXXXXXXX', '***')

    def get_linked_records(self):
        """ This method returns a dict containing all the records linked to the payment.token (e.g Subscriptions),
            the key is the id of the payment.token and the value is an array that must follow the scheme below.

            {
                token_id: [
                    'description': The model description (e.g 'Sale Subscription'),
                    'id': The id of the record,
                    'name': The name of the record,
                    'url': The url to access to this record.
                ]
            }
        """
        return {r.id:[] for r in self}

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ResCompany(models.Model):
    _inherit = 'res.company'

    payment_acquirer_onboarding_state = fields.Selection([('not_done', "Not done"), ('just_done', "Just done"), ('done', "Done")], string="State of the onboarding payment acquirer step", default='not_done')
    # YTI FIXME: Check if it's really needed on the company. Should be enough on the wizard
    payment_onboarding_payment_method = fields.Selection([
        ('paypal', "PayPal"),
        ('stripe', "Stripe"),
        ('manual', "Manual"),
        ('other', "Other"),
    ], string="Selected onboarding payment method")

    @api.model
    def action_open_payment_onboarding_payment_acquirer(self):
        """ Called by onboarding panel above the customer invoice list."""
        # Fail if there are no existing accounts
        self.env.company.get_chart_of_accounts_or_fail()

        action = self.env.ref('payment.action_open_payment_onboarding_payment_acquirer_wizard').read()[0]
        return action

    def get_account_invoice_onboarding_steps_states_names(self):
        """ Override. """
        steps = super(ResCompany, self).get_account_invoice_onboarding_steps_states_names()
        return steps + ['payment_acquirer_onboarding_state']

```

## File: models\res_partner.py

```python
# coding: utf-8

from odoo import api, fields, models


class res_partner(models.Model):
    _name = 'res.partner'
    _inherit = 'res.partner'

    payment_token_ids = fields.One2many('payment.token', 'partner_id', 'Payment Tokens')
    payment_token_count = fields.Integer('Count Payment Token', compute='_compute_payment_token_count')

    @api.depends('payment_token_ids')
    def _compute_payment_token_count(self):
        payment_data = self.env['payment.token'].read_group([
            ('partner_id', 'in', self.ids)], ['partner_id'], ['partner_id'])
        mapped_data = dict([(payment['partner_id'][0], payment['partner_id_count']) for payment in payment_data])
        for partner in self:
            partner.payment_token_count = mapped_data.get(partner.id, 0)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import payment_acquirer
from . import account_invoice
from . import account_journal
from . import res_partner
from . import account_payment
from . import chart_template
from . import ir_http
from . import res_company

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
payment_acquirer_all,payment.acquirer.all,model_payment_acquirer,,1,0,0,0
payment_acquirer_system,payment.acquirer.system,model_payment_acquirer,base.group_system,1,1,1,1
payment_transaction_all,payment.transaction.all,model_payment_transaction,,1,0,0,0
payment_transaction_user,payment.transaction.user,model_payment_transaction,base.group_user,1,1,1,0
payment_transaction_system,payment.transaction.system,model_payment_transaction,base.group_system,1,1,1,1
payment_method_all,payment.token.all,model_payment_token,,1,0,0,0
payment_method_user,payment.token.user,model_payment_token,base.group_user,1,1,1,1
payment_method_portal,payment.token.portal,model_payment_token,base.group_portal,1,1,1,1
payment_method_system,payment.token.system,model_payment_token,base.group_system,1,1,1,1
payment_icon_all,payment.icon.all,model_payment_icon,,1,0,0,0
payment_icon_user,payment.icon.user,model_payment_icon,base.group_user,1,1,1,0
payment_icon_system,payment.icon.system,model_payment_icon,base.group_system,1,1,1,1
```

## File: security\payment_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <record id="payment_transaction_user_rule" model="ir.rule">
            <field name="name">Access own payment transaction only</field>
            <field name="model_id" ref="payment.model_payment_transaction"/>
            <field name="domain_force">[
                '|',
                    ('partner_id','=',False),
                    ('partner_id','=',user.partner_id.id)
                ]</field>
            <field name="groups" eval="[(4, ref('base.group_user')), (4, ref('base.group_portal')), (4, ref('base.group_public'))]"/>
        </record>
        <record id="payment_token_user_rule" model="ir.rule">
            <field name="name">Access own payment tokens only</field>
            <field name="model_id" ref="payment.model_payment_token"/>
            <field name="domain_force">['|', ('partner_id','=',user.partner_id.id), ('partner_id', '=', user.partner_id.commercial_partner_id.id)]</field>
            <field name="groups" eval="[(4, ref('base.group_user')), (4, ref('base.group_portal')), (4, ref('base.group_public'))]"/>
        </record>

    </data>
</odoo>

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="98.616%"><stop offset="0%" stop-color="#797C79"/><stop offset="100%" stop-color="#545554"/></linearGradient><path id="d" d="M19.25 34.621C20.983 35.541 22.9 36 25 36c4 0 7-1.667 9-5h15.036v-3.212a.342.342 0 0 0-.339-.343H35c.347-.502.423-1.416.228-2.742h13.808c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742V34.62zm29.447 12.934a.342.342 0 0 0 .339-.343V37.5H21.964v9.712c0 .188.152.343.339.343h26.394zm-18.614-5.713v2.285a.683.683 0 0 1-.677.685h-4.062a.683.683 0 0 1-.677-.685v-2.285c0-.377.304-.686.677-.686h4.062c.373 0 .677.309.677.686zm10.834 0v2.285a.683.683 0 0 1-.677.685h-7.674a.683.683 0 0 1-.677-.685v-2.285c0-.377.305-.686.677-.686h7.674c.372 0 .677.309.677.686zM22.874 19.77c0 2.517 8.146 1.879 8.146 6.663 0 2.292-1.783 4.25-4.681 4.657v1.897c0 .265-.237.48-.53.48h-1.763c-.292 0-.53-.215-.53-.48v-1.928c-1.74-.268-3.293-.997-4.353-1.917a.447.447 0 0 1-.058-.634l1.34-1.625a.566.566 0 0 1 .757-.086c1.098.8 2.517 1.435 3.873 1.435 1.58 0 2.299-.853 2.299-1.646 0-2.342-8.147-1.834-8.147-6.772 0-2.107 1.705-3.811 4.29-4.342V13.48c0-.265.237-.48.53-.48h1.763c.292 0 .529.215.529.48v1.888c1.418.149 2.954.653 4.025 1.511.184.148.23.391.113.587l-1.038 1.74c-.152.255-.516.33-.775.161-.985-.642-2.193-1.132-3.371-1.132-1.47 0-2.42.603-2.42 1.535z"/><path id="e" d="M19.25 32.621C20.983 33.541 22.9 34 25 34c4 0 7-1.667 9-5h15.036v-3.212a.342.342 0 0 0-.339-.343H35c.347-.502.423-1.416.228-2.742h13.808c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742V32.62zm29.447 12.934a.342.342 0 0 0 .339-.343V35.5H21.964v9.712c0 .188.152.343.339.343h26.394zm-18.614-5.713v2.285a.683.683 0 0 1-.677.685h-4.062a.683.683 0 0 1-.677-.685v-2.285c0-.377.304-.686.677-.686h4.062c.373 0 .677.309.677.686zm10.834 0v2.285a.683.683 0 0 1-.677.685h-7.674a.683.683 0 0 1-.677-.685v-2.285c0-.377.305-.686.677-.686h7.674c.372 0 .677.309.677.686zM22.874 17.77c0 2.517 8.146 1.879 8.146 6.663 0 2.292-1.783 4.25-4.681 4.657v1.897c0 .265-.237.48-.53.48h-1.763c-.292 0-.53-.215-.53-.48v-1.928c-1.74-.268-3.293-.997-4.353-1.917a.447.447 0 0 1-.058-.634l1.34-1.625a.566.566 0 0 1 .757-.086c1.098.8 2.517 1.435 3.873 1.435 1.58 0 2.299-.853 2.299-1.646 0-2.342-8.147-1.834-8.147-6.772 0-2.107 1.705-3.811 4.29-4.342V11.48c0-.265.237-.48.53-.48h1.763c.292 0 .529.215.529.48v1.888c1.418.149 2.954.653 4.025 1.511.184.148.23.391.113.587l-1.038 1.74c-.152.255-.516.33-.775.161-.985-.642-2.193-1.132-3.371-1.132-1.47 0-2.42.603-2.42 1.535z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M4 69c-2 0-4-1-4-4V36.095l23.6-24.837 5.552 6.125-3.147 3.581 4.02 6.176 5.257-4.413L48 23l3.377 2.017v21.868L32.142 69H4z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: static\lib\jquery.payment\jquery.payment.js

```javascript
// Generated by CoffeeScript 1.7.1
(function() {
  var $, cardFromNumber, cardFromType, cards, defaultFormat, formatBackCardNumber, formatBackExpiry, formatCardNumber, formatExpiry, formatForwardExpiry, formatForwardSlashAndSpace, hasTextSelected, luhnCheck, reFormatCVC, reFormatCardNumber, reFormatExpiry, reFormatNumeric, replaceFullWidthChars, restrictCVC, restrictCardNumber, restrictExpiry, restrictNumeric, safeVal, setCardType,
    __slice = [].slice,
    __indexOf = [].indexOf || function(item) { for (var i = 0, l = this.length; i < l; i++) { if (i in this && this[i] === item) return i; } return -1; };

  $ = window.jQuery || window.Zepto || window.$;

  $.payment = {};

  $.payment.fn = {};

  $.fn.payment = function() {
    var args, method;
    method = arguments[0], args = 2 <= arguments.length ? __slice.call(arguments, 1) : [];
    return $.payment.fn[method].apply(this, args);
  };

  defaultFormat = /(\d{1,4})/g;

  $.payment.cards = cards = [
    {
      type: 'maestro',
      patterns: [5018, 502, 503, 506, 56, 58, 639, 6220, 67],
      format: defaultFormat,
      length: [12, 13, 14, 15, 16, 17, 18, 19],
      cvcLength: [3],
      luhn: true
    }, {
      type: 'forbrugsforeningen',
      patterns: [600],
      format: defaultFormat,
      length: [16],
      cvcLength: [3],
      luhn: true
    }, {
      type: 'dankort',
      patterns: [5019],
      format: defaultFormat,
      length: [16],
      cvcLength: [3],
      luhn: true
    }, {
      type: 'visa',
      patterns: [4],
      format: defaultFormat,
      length: [13, 16],
      cvcLength: [3],
      luhn: true
    }, {
      type: 'mastercard',
      patterns: [51, 52, 53, 54, 55, 22, 23, 24, 25, 26, 27],
      format: defaultFormat,
      length: [16],
      cvcLength: [3],
      luhn: true
    }, {
      type: 'amex',
      patterns: [34, 37],
      format: /(\d{1,4})(\d{1,6})?(\d{1,5})?/,
      length: [15],
      cvcLength: [3, 4],
      luhn: true
    }, {
      type: 'dinersclub',
      patterns: [30, 36, 38, 39],
      format: /(\d{1,4})(\d{1,6})?(\d{1,4})?/,
      length: [14],
      cvcLength: [3],
      luhn: true
    }, {
      type: 'discover',
      patterns: [60, 64, 65, 622],
      format: defaultFormat,
      length: [16],
      cvcLength: [3],
      luhn: true
    }, {
      type: 'unionpay',
      patterns: [62, 88],
      format: defaultFormat,
      length: [16, 17, 18, 19],
      cvcLength: [3],
      luhn: false
    }, {
      type: 'jcb',
      patterns: [35],
      format: defaultFormat,
      length: [16],
      cvcLength: [3],
      luhn: true
    }
  ];

  cardFromNumber = function(num) {
    var card, p, pattern, _i, _j, _len, _len1, _ref;
    num = (num + '').replace(/\D/g, '');
    for (_i = 0, _len = cards.length; _i < _len; _i++) {
      card = cards[_i];
      _ref = card.patterns;
      for (_j = 0, _len1 = _ref.length; _j < _len1; _j++) {
        pattern = _ref[_j];
        p = pattern + '';
        if (num.substr(0, p.length) === p) {
          return card;
        }
      }
    }
  };

  cardFromType = function(type) {
    var card, _i, _len;
    for (_i = 0, _len = cards.length; _i < _len; _i++) {
      card = cards[_i];
      if (card.type === type) {
        return card;
      }
    }
  };

  luhnCheck = function(num) {
    var digit, digits, odd, sum, _i, _len;
    odd = true;
    sum = 0;
    digits = (num + '').split('').reverse();
    for (_i = 0, _len = digits.length; _i < _len; _i++) {
      digit = digits[_i];
      digit = parseInt(digit, 10);
      if ((odd = !odd)) {
        digit *= 2;
      }
      if (digit > 9) {
        digit -= 9;
      }
      sum += digit;
    }
    return sum % 10 === 0;
  };

  hasTextSelected = function($target) {
    var _ref;
    if (($target.prop('selectionStart') != null) && $target.prop('selectionStart') !== $target.prop('selectionEnd')) {
      return true;
    }
    if ((typeof document !== "undefined" && document !== null ? (_ref = document.selection) != null ? _ref.createRange : void 0 : void 0) != null) {
      if (document.selection.createRange().text) {
        return true;
      }
    }
    return false;
  };

  safeVal = function(value, $target) {
    var currPair, cursor, digit, error, last, prevPair;
    try {
      cursor = $target.prop('selectionStart');
    } catch (_error) {
      error = _error;
      cursor = null;
    }
    last = $target.val();
    $target.val(value);
    if (cursor !== null && $target.is(":focus")) {
      if (cursor === last.length) {
        cursor = value.length;
      }
      if (last !== value) {
        prevPair = last.slice(cursor - 1, +cursor + 1 || 9e9);
        currPair = value.slice(cursor - 1, +cursor + 1 || 9e9);
        digit = value[cursor];
        if (/\d/.test(digit) && prevPair === ("" + digit + " ") && currPair === (" " + digit)) {
          cursor = cursor + 1;
        }
      }
      $target.prop('selectionStart', cursor);
      return $target.prop('selectionEnd', cursor);
    }
  };

  replaceFullWidthChars = function(str) {
    var chars, chr, fullWidth, halfWidth, idx, value, _i, _len;
    if (str == null) {
      str = '';
    }
    fullWidth = '\uff10\uff11\uff12\uff13\uff14\uff15\uff16\uff17\uff18\uff19';
    halfWidth = '0123456789';
    value = '';
    chars = str.split('');
    for (_i = 0, _len = chars.length; _i < _len; _i++) {
      chr = chars[_i];
      idx = fullWidth.indexOf(chr);
      if (idx > -1) {
        chr = halfWidth[idx];
      }
      value += chr;
    }
    return value;
  };

  reFormatNumeric = function(e) {
    var $target;
    $target = $(e.currentTarget);
    return setTimeout(function() {
      var value;
      value = $target.val();
      value = replaceFullWidthChars(value);
      value = value.replace(/\D/g, '');
      return safeVal(value, $target);
    });
  };

  reFormatCardNumber = function(e) {
    var $target;
    $target = $(e.currentTarget);
    return setTimeout(function() {
      var value;
      value = $target.val();
      value = replaceFullWidthChars(value);
      value = $.payment.formatCardNumber(value);
      return safeVal(value, $target);
    });
  };

  formatCardNumber = function(e) {
    var $target, card, digit, length, re, upperLength, value;
    digit = String.fromCharCode(e.which);
    if (!/^\d+$/.test(digit)) {
      return;
    }
    $target = $(e.currentTarget);
    value = $target.val();
    card = cardFromNumber(value + digit);
    length = (value.replace(/\D/g, '') + digit).length;
    upperLength = 16;
    if (card) {
      upperLength = card.length[card.length.length - 1];
    }
    if (length >= upperLength) {
      return;
    }
    if (($target.prop('selectionStart') != null) && $target.prop('selectionStart') !== value.length) {
      return;
    }
    if (card && card.type === 'amex') {
      re = /^(\d{4}|\d{4}\s\d{6})$/;
    } else {
      re = /(?:^|\s)(\d{4})$/;
    }
    if (re.test(value)) {
      e.preventDefault();
      return setTimeout(function() {
        return $target.val(value + ' ' + digit);
      });
    } else if (re.test(value + digit)) {
      e.preventDefault();
      return setTimeout(function() {
        return $target.val(value + digit + ' ');
      });
    }
  };

  formatBackCardNumber = function(e) {
    var $target, value;
    $target = $(e.currentTarget);
    value = $target.val();
    if (e.which !== 8) {
      return;
    }
    if (($target.prop('selectionStart') != null) && $target.prop('selectionStart') !== value.length) {
      return;
    }
    if (/\d\s$/.test(value)) {
      e.preventDefault();
      return setTimeout(function() {
        return $target.val(value.replace(/\d\s$/, ''));
      });
    } else if (/\s\d?$/.test(value)) {
      e.preventDefault();
      return setTimeout(function() {
        return $target.val(value.replace(/\d$/, ''));
      });
    }
  };

  reFormatExpiry = function(e) {
    var $target;
    $target = $(e.currentTarget);
    return setTimeout(function() {
      var value;
      value = $target.val();
      value = replaceFullWidthChars(value);
      value = $.payment.formatExpiry(value);
      return safeVal(value, $target);
    });
  };

  formatExpiry = function(e) {
    var $target, digit, val;
    digit = String.fromCharCode(e.which);
    if (!/^\d+$/.test(digit)) {
      return;
    }
    $target = $(e.currentTarget);
    val = $target.val() + digit;
    if (/^\d$/.test(val) && (val !== '0' && val !== '1')) {
      e.preventDefault();
      return setTimeout(function() {
        return $target.val("0" + val + " / ");
      });
    } else if (/^\d\d$/.test(val)) {
      e.preventDefault();
      return setTimeout(function() {
        var m1, m2;
        m1 = parseInt(val[0], 10);
        m2 = parseInt(val[1], 10);
        if (m2 > 2 && m1 !== 0) {
          return $target.val("0" + m1 + " / " + m2);
        } else {
          return $target.val("" + val + " / ");
        }
      });
    }
  };

  formatForwardExpiry = function(e) {
    var $target, digit, val;
    digit = String.fromCharCode(e.which);
    if (!/^\d+$/.test(digit)) {
      return;
    }
    $target = $(e.currentTarget);
    val = $target.val();
    if (/^\d\d$/.test(val)) {
      return $target.val("" + val + " / ");
    }
  };

  formatForwardSlashAndSpace = function(e) {
    var $target, val, which;
    which = String.fromCharCode(e.which);
    if (!(which === '/' || which === ' ')) {
      return;
    }
    $target = $(e.currentTarget);
    val = $target.val();
    if (/^\d$/.test(val) && val !== '0') {
      return $target.val("0" + val + " / ");
    }
  };

  formatBackExpiry = function(e) {
    var $target, value;
    $target = $(e.currentTarget);
    value = $target.val();
    if (e.which !== 8) {
      return;
    }
    if (($target.prop('selectionStart') != null) && $target.prop('selectionStart') !== value.length) {
      return;
    }
    if (/\d\s\/\s$/.test(value)) {
      e.preventDefault();
      return setTimeout(function() {
        return $target.val(value.replace(/\d\s\/\s$/, ''));
      });
    }
  };

  reFormatCVC = function(e) {
    var $target;
    $target = $(e.currentTarget);
    return setTimeout(function() {
      var value;
      value = $target.val();
      value = replaceFullWidthChars(value);
      value = value.replace(/\D/g, '').slice(0, 4);
      return safeVal(value, $target);
    });
  };

  restrictNumeric = function(e) {
    var input;
    if (e.metaKey || e.ctrlKey) {
      return true;
    }
    if (e.which === 32) {
      return false;
    }
    if (e.which === 0) {
      return true;
    }
    if (e.which < 33) {
      return true;
    }
    input = String.fromCharCode(e.which);
    return !!/[\d\s]/.test(input);
  };

  restrictCardNumber = function(e) {
    var $target, card, digit, value;
    $target = $(e.currentTarget);
    digit = String.fromCharCode(e.which);
    if (!/^\d+$/.test(digit)) {
      return;
    }
    if (hasTextSelected($target)) {
      return;
    }
    value = ($target.val() + digit).replace(/\D/g, '');
    card = cardFromNumber(value);
    if (card) {
      return value.length <= card.length[card.length.length - 1];
    } else {
      return value.length <= 16;
    }
  };

  restrictExpiry = function(e) {
    var $target, digit, value;
    $target = $(e.currentTarget);
    digit = String.fromCharCode(e.which);
    if (!/^\d+$/.test(digit)) {
      return;
    }
    if (hasTextSelected($target)) {
      return;
    }
    value = $target.val() + digit;
    value = value.replace(/\D/g, '');
    if (value.length > 6) {
      return false;
    }
  };

  restrictCVC = function(e) {
    var $target, digit, val;
    $target = $(e.currentTarget);
    digit = String.fromCharCode(e.which);
    if (!/^\d+$/.test(digit)) {
      return;
    }
    if (hasTextSelected($target)) {
      return;
    }
    val = $target.val() + digit;
    return val.length <= 4;
  };

  setCardType = function(e) {
    var $target, allTypes, card, cardType, val;
    $target = $(e.currentTarget);
    val = $target.val();
    cardType = $.payment.cardType(val) || 'unknown';
    if (!$target.hasClass(cardType)) {
      allTypes = (function() {
        var _i, _len, _results;
        _results = [];
        for (_i = 0, _len = cards.length; _i < _len; _i++) {
          card = cards[_i];
          _results.push(card.type);
        }
        return _results;
      })();
      $target.removeClass('unknown');
      $target.removeClass(allTypes.join(' '));
      $target.addClass(cardType);
      $target.toggleClass('identified', cardType !== 'unknown');
      return $target.trigger('payment.cardType', cardType);
    }
  };

  $.payment.fn.formatCardCVC = function() {
    this.on('keypress', restrictNumeric);
    this.on('keypress', restrictCVC);
    this.on('paste', reFormatCVC);
    this.on('change', reFormatCVC);
    this.on('input', reFormatCVC);
    return this;
  };

  $.payment.fn.formatCardExpiry = function() {
    this.on('keypress', restrictNumeric);
    this.on('keypress', restrictExpiry);
    this.on('keypress', formatExpiry);
    this.on('keypress', formatForwardSlashAndSpace);
    this.on('keypress', formatForwardExpiry);
    this.on('keydown', formatBackExpiry);
    this.on('change', reFormatExpiry);
    this.on('input', reFormatExpiry);
    return this;
  };

  $.payment.fn.formatCardNumber = function() {
    this.on('keypress', restrictNumeric);
    this.on('keypress', restrictCardNumber);
    this.on('keypress', formatCardNumber);
    this.on('keydown', formatBackCardNumber);
    this.on('keyup', setCardType);
    this.on('paste', reFormatCardNumber);
    this.on('change', reFormatCardNumber);
    this.on('input', reFormatCardNumber);
    this.on('input', setCardType);
    return this;
  };

  $.payment.fn.restrictNumeric = function() {
    this.on('keypress', restrictNumeric);
    this.on('paste', reFormatNumeric);
    this.on('change', reFormatNumeric);
    this.on('input', reFormatNumeric);
    return this;
  };

  $.payment.fn.cardExpiryVal = function() {
    return $.payment.cardExpiryVal($(this).val());
  };

  $.payment.cardExpiryVal = function(value) {
    var month, prefix, year, _ref;
    _ref = value.split(/[\s\/]+/, 2), month = _ref[0], year = _ref[1];
    if ((year != null ? year.length : void 0) === 2 && /^\d+$/.test(year)) {
      prefix = (new Date).getFullYear();
      prefix = prefix.toString().slice(0, 2);
      year = prefix + year;
    }
    month = parseInt(month, 10);
    year = parseInt(year, 10);
    return {
      month: month,
      year: year
    };
  };

  $.payment.validateCardNumber = function(num) {
    var card, _ref;
    num = (num + '').replace(/\s+|-/g, '');
    if (!/^\d+$/.test(num)) {
      return false;
    }
    card = cardFromNumber(num);
    if (!card) {
      return false;
    }
    return (_ref = num.length, __indexOf.call(card.length, _ref) >= 0) && (card.luhn === false || luhnCheck(num));
  };

  $.payment.validateCardExpiry = function(month, year) {
    var currentTime, expiry, _ref;
    if (typeof month === 'object' && 'month' in month) {
      _ref = month, month = _ref.month, year = _ref.year;
    }
    if (!(month && year)) {
      return false;
    }
    month = $.trim(month);
    year = $.trim(year);
    if (!/^\d+$/.test(month)) {
      return false;
    }
    if (!/^\d+$/.test(year)) {
      return false;
    }
    if (!((1 <= month && month <= 12))) {
      return false;
    }
    if (year.length === 2) {
      if (year < 70) {
        year = "20" + year;
      } else {
        year = "19" + year;
      }
    }
    if (year.length !== 4) {
      return false;
    }
    expiry = new Date(year, month);
    currentTime = new Date;
    expiry.setMonth(expiry.getMonth() - 1);
    expiry.setMonth(expiry.getMonth() + 1, 1);
    return expiry > currentTime;
  };

  $.payment.validateCardCVC = function(cvc, type) {
    var card, _ref;
    cvc = $.trim(cvc);
    if (!/^\d+$/.test(cvc)) {
      return false;
    }
    card = cardFromType(type);
    if (card != null) {
      return _ref = cvc.length, __indexOf.call(card.cvcLength, _ref) >= 0;
    } else {
      return cvc.length >= 3 && cvc.length <= 4;
    }
  };

  $.payment.cardType = function(num) {
    var _ref;
    if (!num) {
      return null;
    }
    return ((_ref = cardFromNumber(num)) != null ? _ref.type : void 0) || null;
  };

  $.payment.formatCardNumber = function(num) {
    var card, groups, upperLength, _ref;
    num = num.replace(/\D/g, '');
    card = cardFromNumber(num);
    if (!card) {
      return num;
    }
    upperLength = card.length[card.length.length - 1];
    num = num.slice(0, upperLength);
    if (card.format.global) {
      return (_ref = num.match(card.format)) != null ? _ref.join(' ') : void 0;
    } else {
      groups = card.format.exec(num);
      if (groups == null) {
        return;
      }
      groups.shift();
      groups = $.grep(groups, function(n) {
        return n;
      });
      return groups.join(' ');
    }
  };

  $.payment.formatExpiry = function(expiry) {
    var mon, parts, sep, year;
    parts = expiry.match(/^\D*(\d{1,2})(\D+)?(\d{1,4})?/);
    if (!parts) {
      return '';
    }
    mon = parts[1] || '';
    sep = parts[2] || '';
    year = parts[3] || '';
    if (year.length > 0) {
      sep = ' / ';
    } else if (sep === ' /') {
      mon = mon.substring(0, 1);
      sep = '';
    } else if (mon.length === 2 || sep.length > 0) {
      sep = ' / ';
    } else if (mon.length === 1 && (mon !== '0' && mon !== '1')) {
      mon = "0" + mon;
      sep = ' / ';
    }
    return mon + sep + year;
  };

}).call(this);

```

## File: static\src\js\payment_form.js

```javascript
odoo.define('payment.payment_form', function (require) {
"use strict";

var core = require('web.core');
var Dialog = require('web.Dialog');
var publicWidget = require('web.public.widget');

var _t = core._t;

publicWidget.registry.PaymentForm = publicWidget.Widget.extend({
    selector: '.o_payment_form',
    events: {
        'submit': 'onSubmit',
        'click #o_payment_form_pay': 'payEvent',
        'click #o_payment_form_add_pm': 'addPmEvent',
        'click button[name="delete_pm"]': 'deletePmEvent',
        'click .o_payment_form_pay_icon_more': 'onClickMorePaymentIcon',
        'click .o_payment_acquirer_select': 'radioClickEvent',
    },

    /**
     * @override
     */
    start: function () {
        this._adaptPayButton();
        window.addEventListener('pageshow', function (event) {
            if (event.persisted) {
                window.location.reload();
            }
        });
        var self = this;
        return this._super.apply(this, arguments).then(function () {
            self.options = _.extend(self.$el.data(), self.options);
            self.updateNewPaymentDisplayStatus();
            $('[data-toggle="tooltip"]').tooltip();
        });
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {string} title
     * @param {string} message
     */
    displayError: function (title, message) {
        var $checkedRadio = this.$('input[type="radio"]:checked'),
            acquirerID = this.getAcquirerIdFromRadio($checkedRadio[0]);
        var $acquirerForm;
        if (this.isNewPaymentRadio($checkedRadio[0])) {
            $acquirerForm = this.$('#o_payment_add_token_acq_' + acquirerID);
        }
        else if (this.isFormPaymentRadio($checkedRadio[0])) {
            $acquirerForm = this.$('#o_payment_form_acq_' + acquirerID);
        }

        if ($checkedRadio.length === 0) {
            return new Dialog(null, {
                title: _t('Error: ') + _.str.escapeHTML(title),
                size: 'medium',
                $content: "<p>" + (_.str.escapeHTML(message) || "") + "</p>" ,
                buttons: [
                {text: _t('Ok'), close: true}]}).open();
        } else {
            // removed if exist error message
            this.$('#payment_error').remove();
            var messageResult = '<div class="alert alert-danger mb4" id="payment_error">';
            if (title != '') {
                messageResult = messageResult + '<b>' + _.str.escapeHTML(title) + ':</b><br/>';
            }
            messageResult = messageResult + _.str.escapeHTML(message) + '</div>';
            $acquirerForm.append(messageResult);
        }
    },
    hideError: function() {
        this.$('#payment_error').remove();
    },
    /**
     * @private
     * @param {DOMElement} element
     */
    getAcquirerIdFromRadio: function (element) {
        return $(element).data('acquirer-id');
    },
    /**
     * @private
     * @param {jQuery} $form
     */
    getFormData: function ($form) {
        var unindexed_array = $form.serializeArray();
        var indexed_array = {};

        $.map(unindexed_array, function (n, i) {
            indexed_array[n.name] = n.value;
        });
        return indexed_array;
    },
    /**
     * @private
     * @param {DOMElement} element
     */
    isFormPaymentRadio: function (element) {
        return $(element).data('form-payment') === 'True';
    },
    /**
     * @private
     * @param {DOMElement} element
     */
    isNewPaymentRadio: function (element) {
        return $(element).data('s2s-payment') === 'True';
    },
    /**
     * @private
     */
    updateNewPaymentDisplayStatus: function () {
        var checked_radio = this.$('input[type="radio"]:checked');
        // we hide all the acquirers form
        this.$('[id*="o_payment_add_token_acq_"]').addClass('d-none');
        this.$('[id*="o_payment_form_acq_"]').addClass('d-none');
        if (checked_radio.length !== 1) {
            return;
        }
        checked_radio = checked_radio[0];
        var acquirer_id = this.getAcquirerIdFromRadio(checked_radio);

        // if we clicked on an add new payment radio, display its form
        if (this.isNewPaymentRadio(checked_radio)) {
            this.$('#o_payment_add_token_acq_' + acquirer_id).removeClass('d-none');
        }
        else if (this.isFormPaymentRadio(checked_radio)) {
            this.$('#o_payment_form_acq_' + acquirer_id).removeClass('d-none');
        }
    },

    disableButton: function (button) {
        $("body").block({overlayCSS: {backgroundColor: "#000", opacity: 0, zIndex: 1050}, message: false});
        $(button).attr('disabled', true);
        $(button).children('.fa-lock').removeClass('fa-lock');
        $(button).prepend('<span class="o_loader"><i class="fa fa-refresh fa-spin"></i>&nbsp;</span>');
    },

    enableButton: function (button) {
        $('body').unblock();
        $(button).attr('disabled', false);
        $(button).children('.fa').addClass('fa-lock');
        $(button).find('span.o_loader').remove();
    },
    _parseError: function(e) { 
        if (e.message.data.arguments[1]) {
            return e.message.data.arguments[0] + e.message.data.arguments[1];
        }
        return e.message.data.arguments[0];
    },
    _adaptPayButton: function () {
        var $payButton = $("#o_payment_form_pay");
        var disabledReasons = $payButton.data('disabled_reasons') || {};
        $payButton.prop('disabled', _.contains(disabledReasons, true));
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {Event} ev
     */
    payEvent: function (ev) {
        ev.preventDefault();
        var form = this.el;
        var checked_radio = this.$('input[type="radio"]:checked');
        var self = this;
        if (ev.type === 'submit') {
            var button = $(ev.target).find('*[type="submit"]')[0]
        } else {
            var button = ev.target;
        }

        // first we check that the user has selected a payment method
        if (checked_radio.length === 1) {
            checked_radio = checked_radio[0];

            // we retrieve all the input inside the acquirer form and 'serialize' them to an indexed array
            var acquirer_id = this.getAcquirerIdFromRadio(checked_radio);
            var acquirer_form = false;
            if (this.isNewPaymentRadio(checked_radio)) {
                acquirer_form = this.$('#o_payment_add_token_acq_' + acquirer_id);
            } else {
                acquirer_form = this.$('#o_payment_form_acq_' + acquirer_id);
            }
            var inputs_form = $('input', acquirer_form);
            var ds = $('input[name="data_set"]', acquirer_form)[0];

            // if the user is adding a new payment
            if (this.isNewPaymentRadio(checked_radio)) {
                if (this.options.partnerId === undefined) {
                    console.warn('payment_form: unset partner_id when adding new token; things could go wrong');
                }
                var form_data = this.getFormData(inputs_form);
                var wrong_input = false;

                inputs_form.toArray().forEach(function (element) {
                    //skip the check of non visible inputs
                    if ($(element).attr('type') == 'hidden') {
                        return true;
                    }
                    $(element).closest('div.form-group').removeClass('o_has_error').find('.form-control, .custom-select').removeClass('is-invalid');
                    $(element).siblings( ".o_invalid_field" ).remove();
                    //force check of forms validity (useful for Firefox that refill forms automatically on f5)
                    $(element).trigger("focusout");
                    if (element.dataset.isRequired && element.value.length === 0) {
                            $(element).closest('div.form-group').addClass('o_has_error').find('.form-control, .custom-select').addClass('is-invalid');
                            $(element).closest('div.form-group').append('<div style="color: red" class="o_invalid_field" aria-invalid="true">' + _.str.escapeHTML("The value is invalid.") + '</div>');
                            wrong_input = true;
                    }
                    else if ($(element).closest('div.form-group').hasClass('o_has_error')) {
                        wrong_input = true;
                        $(element).closest('div.form-group').append('<div style="color: red" class="o_invalid_field" aria-invalid="true">' + _.str.escapeHTML("The value is invalid.") + '</div>');
                    }
                });

                if (wrong_input) {
                    return;
                }

                this.disableButton(button);
                // do the call to the route stored in the 'data_set' input of the acquirer form, the data must be called 'create-route'
                return this._rpc({
                    route: ds.dataset.createRoute,
                    params: form_data,
                }).then(function (data) {
                    // if the server has returned true
                    if (data.result) {
                        // and it need a 3DS authentication
                        if (data['3d_secure'] !== false) {
                            // then we display the 3DS page to the user
                            $("body").html(data['3d_secure']);
                        }
                        else {
                            checked_radio.value = data.id; // set the radio value to the new card id
                            form.submit();
                            return new Promise(function () {});
                        }
                    }
                    // if the server has returned false, we display an error
                    else {
                        if (data.error) {
                            self.displayError(
                                '',
                                data.error);
                        } else { // if the server doesn't provide an error message
                            self.displayError(
                                _t('Server Error'),
                                _t('e.g. Your credit card details are wrong. Please verify.'));
                        }
                    }
                    // here we remove the 'processing' icon from the 'add a new payment' button
                    self.enableButton(button);
                }).guardedCatch(function (error) {
                    error.event.preventDefault();
                    // if the rpc fails, pretty obvious
                    self.enableButton(button);

                    self.displayError(
                        _t('Server Error'),
                        _t("We are not able to add your payment method at the moment.") +
                            self._parseError(error)
                    );
                });
            }
            // if the user is going to pay with a form payment, then
            else if (this.isFormPaymentRadio(checked_radio)) {
                this.disableButton(button);
                var $tx_url = this.$el.find('input[name="prepare_tx_url"]');
                // if there's a prepare tx url set
                if ($tx_url.length === 1) {
                    // if the user wants to save his credit card info
                    var form_save_token = acquirer_form.find('input[name="o_payment_form_save_token"]').prop('checked');
                    // then we call the route to prepare the transaction
                    return this._rpc({
                        route: $tx_url[0].value,
                        params: {
                            'acquirer_id': parseInt(acquirer_id),
                            'save_token': form_save_token,
                            'access_token': self.options.accessToken,
                            'success_url': self.options.successUrl,
                            'error_url': self.options.errorUrl,
                            'callback_method': self.options.callbackMethod,
                            'order_id': self.options.orderId,
                            'invoice_id': self.options.invoiceId,
                        },
                    }).then(function (result) {
                        if (result) {
                            // if the server sent us the html form, we create a form element
                            var newForm = document.createElement('form');
                            newForm.setAttribute("method", self._get_redirect_form_method());
                            newForm.setAttribute("provider", checked_radio.dataset.provider);
                            newForm.hidden = true; // hide it
                            newForm.innerHTML = result; // put the html sent by the server inside the form
                            var action_url = $(newForm).find('input[name="data_set"]').data('actionUrl');
                            newForm.setAttribute("action", action_url); // set the action url
                            $(document.getElementsByTagName('body')[0]).append(newForm); // append the form to the body
                            $(newForm).find('input[data-remove-me]').remove(); // remove all the input that should be removed
                            if(action_url) {
                                newForm.submit(); // and finally submit the form
                                return new Promise(function () {});
                            }
                        }
                        else {
                            self.displayError(
                                _t('Server Error'),
                                _t("We are not able to redirect you to the payment form.")
                            );
                            self.enableButton(button);
                        }
                    }).guardedCatch(function (error) {
                        error.event.preventDefault();
                        self.displayError(
                            _t('Server Error'),
                            _t("We are not able to redirect you to the payment form.") + " " +
                                self._parseError(error)
                        );
                        self.enableButton(button);
                    });
                }
                else {
                    // we append the form to the body and send it.
                    this.displayError(
                        _t("Cannot setup the payment"),
                        _t("We're unable to process your payment.")
                    );
                    self.enableButton(button);
                }
            }
            else {  // if the user is using an old payment then we just submit the form
                this.disableButton(button);
                form.submit();
                return new Promise(function () {});
            }
        }
        else {
            this.displayError(
                _t('No payment method selected'),
                _t('Please select a payment method.')
            );
            this.enableButton(button);
        }
    },
    /**
     * Return the HTTP method to be used by the redirect form.
     *
     * @private
     * @return {string} The HTTP method, "post" by default
     */
    _get_redirect_form_method: function () {
        return "post";
    },
    /**
     * Called when clicking on the button to add a new payment method.
     *
     * @private
     * @param {Event} ev
     */
    addPmEvent: function (ev) {
        ev.stopPropagation();
        ev.preventDefault();
        var checked_radio = this.$('input[type="radio"]:checked');
        var self = this;
        if (ev.type === 'submit') {
            var button = $(ev.target).find('*[type="submit"]')[0]
        } else {
            var button = ev.target;
        }

        // we check if the user has selected a 'add a new payment' option
        if (checked_radio.length === 1 && this.isNewPaymentRadio(checked_radio[0])) {
            // we retrieve which acquirer is used
            checked_radio = checked_radio[0];
            var acquirer_id = this.getAcquirerIdFromRadio(checked_radio);
            var acquirer_form = this.$('#o_payment_add_token_acq_' + acquirer_id);
            // we retrieve all the input inside the acquirer form and 'serialize' them to an indexed array
            var inputs_form = $('input', acquirer_form);
            var form_data = this.getFormData(inputs_form);
            var ds = $('input[name="data_set"]', acquirer_form)[0];
            var wrong_input = false;

            inputs_form.toArray().forEach(function (element) {
                //skip the check of non visible inputs
                if ($(element).attr('type') == 'hidden') {
                    return true;
                }
                $(element).closest('div.form-group').removeClass('o_has_error').find('.form-control, .custom-select').removeClass('is-invalid');
                $(element).siblings( ".o_invalid_field" ).remove();
                //force check of forms validity (useful for Firefox that refill forms automatically on f5)
                $(element).trigger("focusout");
                if (element.dataset.isRequired && element.value.length === 0) {
                        $(element).closest('div.form-group').addClass('o_has_error').find('.form-control, .custom-select').addClass('is-invalid');
                        var message = '<div style="color: red" class="o_invalid_field" aria-invalid="true">' + _.str.escapeHTML("The value is invalid.") + '</div>';
                        $(element).closest('div.form-group').append(message);
                        wrong_input = true;
                }
                else if ($(element).closest('div.form-group').hasClass('o_has_error')) {
                    wrong_input = true;
                    var message = '<div style="color: red" class="o_invalid_field" aria-invalid="true">' + _.str.escapeHTML("The value is invalid.") + '</div>';
                    $(element).closest('div.form-group').append(message);
                }
            });

            if (wrong_input) {
                return;
            }
            // We add a 'processing' icon into the 'add a new payment' button
            $(button).attr('disabled', true);
            $(button).children('.fa-plus-circle').removeClass('fa-plus-circle');
            $(button).prepend('<span class="o_loader"><i class="fa fa-refresh fa-spin"></i>&nbsp;</span>');

            // do the call to the route stored in the 'data_set' input of the acquirer form, the data must be called 'create-route'
            this._rpc({
                route: ds.dataset.createRoute,
                params: form_data,
            }).then(function (data) {
                // if the server has returned true
                if (data.result) {
                    // and it need a 3DS authentication
                    if (data['3d_secure'] !== false) {
                        // then we display the 3DS page to the user
                        $("body").html(data['3d_secure']);
                    }
                    // if it doesn't require 3DS
                    else {
                        // we just go to the return_url or reload the page
                        if (form_data.return_url) {
                            window.location = form_data.return_url;
                        }
                        else {
                            window.location.reload();
                        }
                    }
                }
                // if the server has returned false, we display an error
                else {
                    if (data.error) {
                        self.displayError(
                            '',
                            data.error);
                    } else { // if the server doesn't provide an error message
                        self.displayError(
                            _t('Server Error'),
                            _t('e.g. Your credit card details are wrong. Please verify.'));
                    }
                }
                // here we remove the 'processing' icon from the 'add a new payment' button
                $(button).attr('disabled', false);
                $(button).children('.fa').addClass('fa-plus-circle');
                $(button).find('span.o_loader').remove();
            }).guardedCatch(function (error) {
                error.event.preventDefault();
                // if the rpc fails, pretty obvious
                $(button).attr('disabled', false);
                $(button).children('.fa').addClass('fa-plus-circle');
                $(button).find('span.o_loader').remove();

                self.displayError(
                    _t('Server error'),
                    _t("We are not able to add your payment method at the moment.") +
                        self._parseError(error)
                );
            });
        }
        else {
            this.displayError(
                _t('No payment method selected'),
                _t('Please select the option to add a new payment method.')
            );
        }
    },
    /**
     * Called when submitting the form (e.g. through the Return key).
     * We need to check whether we are paying or adding a new pm and dispatch
     * to the correct method.
     *
     * @private
     * @param {Event} ev
     */
    onSubmit: function(ev) {
        ev.stopPropagation();
        ev.preventDefault();
        var button = $(ev.target).find('*[type="submit"]')[0]
        if (button.id === 'o_payment_form_pay') {
            return this.payEvent(ev);
        } else if (button.id === 'o_payment_form_add_pm') {
            return this.addPmEvent(ev);
        }
        return;
    },
    /**
     * Called when clicking on a button to delete a payment method.
     *
     * @private
     * @param {Event} ev
     */
    deletePmEvent: function (ev) {
        ev.stopPropagation();
        ev.preventDefault();
        var self = this;
        var pm_id = parseInt(ev.currentTarget.value);

        var tokenDelete = function () {
            self._rpc({
                model: 'payment.token',
                method: 'unlink',
                args: [pm_id],
            }).then(function (result) {
                if (result === true) {
                    ev.target.closest('div').remove();
                }
            }, function () {
                self.displayError(
                    _t('Server Error'),
                    _t("We are not able to delete your payment method at the moment.")
                );
            });
        };

        this._rpc({
            model: 'payment.token',
            method: 'get_linked_records',
            args: [pm_id],
        }).then(function (result) {
            if (result[pm_id].length > 0) {
                // if there's records linked to this payment method
                var content = '';
                result[pm_id].forEach(function (sub) {
                    content += '<p><a href="' + sub.url + '" title="' + sub.description + '">' + sub.name + '</a></p>';
                });

                content = $('<div>').html('<p>' + _t('This card is currently linked to the following records:') + '</p>' + content);
                // Then we display the list of the records and ask the user if he really want to remove the payment method.
                new Dialog(self, {
                    title: _t('Warning!'),
                    size: 'medium',
                    $content: content,
                    buttons: [
                    {text: _t('Confirm Deletion'), classes: 'btn-primary', close: true, click: tokenDelete},
                    {text: _t('Cancel'), close: true}]
                }).open();
            }
            else {
                // if there's no records linked to this payment method, then we delete it
                tokenDelete();
            }
        }, function (err, event) {
            self.displayError(
                _t('Server Error'),
                _t("We are not able to delete your payment method at the moment.") + err.data.message
            );
        });
    },
    /**
     * Called when clicking on 'and more' to show more payment icon.
     *
     * @private
     * @param {Event} ev
     */
    onClickMorePaymentIcon: function (ev) {
        ev.preventDefault();
        ev.stopPropagation();
        var $listItems = $(ev.currentTarget).parents('ul').children('li');
        var $moreItem = $(ev.currentTarget).parents('li');
        $listItems.removeClass('d-none');
        $moreItem.addClass('d-none');
    },
    /**
     * Called when clicking on a radio button.
     *
     * @private
     * @param {Event} ev
     */
    radioClickEvent: function (ev) {
        // radio button checked when we click on entire zone(body) of the payment acquirer
        $(ev.currentTarget).find('input[type="radio"]').prop("checked", true);
        this.updateNewPaymentDisplayStatus();
    },
});
return publicWidget.registry.PaymentForm;
});

```

## File: static\src\js\payment_portal.js

```javascript
$(function () {

    $('input#cc_number').payment('formatCardNumber');
    $('input#cc_cvc').payment('formatCardCVC');
    $('input#cc_expiry').payment('formatCardExpiry')

    $('input#cc_number').on('focusout', function (e) {
        var valid_value = $.payment.validateCardNumber(this.value);
        var card_type = $.payment.cardType(this.value);
        if (card_type) {
            $(this).parent('.form-group').children('.card_placeholder').removeClass().addClass('card_placeholder ' + card_type);
            $(this).parent('.form-group').children('input[name="cc_brand"]').val(card_type)
        }
        else {
            $(this).parent('.form-group').children('.card_placeholder').removeClass().addClass('card_placeholder');
        }
        if (valid_value) {
            $(this).parent('.form-group').addClass('o_has_success').find('.form-control, .custom-select').addClass('is-valid');
            $(this).parent('.form-group').removeClass('o_has_error').find('.form-control, .custom-select').removeClass('is-invalid');
            $(this).siblings('.o_invalid_field').remove();
        }
        else {
            $(this).parent('.form-group').addClass('o_has_error').find('.form-control, .custom-select').addClass('is-invalid');
            $(this).parent('.form-group').removeClass('o_has_success').find('.form-control, .custom-select').removeClass('is-valid');
        }
    });

    $('input#cc_cvc').on('focusout', function (e) {
        var cc_nbr = $(this).parents('.oe_cc').find('#cc_number').val();
        var card_type = $.payment.cardType(cc_nbr);
        var valid_value = $.payment.validateCardCVC(this.value, card_type);
        if (valid_value) {
            $(this).parent('.form-group').addClass('o_has_success').find('.form-control, .custom-select').addClass('is-valid');
            $(this).parent('.form-group').removeClass('o_has_error').find('.form-control, .custom-select').removeClass('is-invalid');
            $(this).siblings('.o_invalid_field').remove();
        }
        else {
            $(this).parent('.form-group').addClass('o_has_error').find('.form-control, .custom-select').addClass('is-invalid');
            $(this).parent('.form-group').removeClass('o_has_success').find('.form-control, .custom-select').removeClass('is-valid');
        }
    });

    $('input#cc_expiry').on('focusout', function (e) {
        var expiry_value = $.payment.cardExpiryVal(this.value);
        var month = expiry_value.month || '';
        var year = expiry_value.year || '';
        var valid_value = $.payment.validateCardExpiry(month, year);
        if (valid_value) {
            $(this).parent('.form-group').addClass('o_has_success').find('.form-control, .custom-select').addClass('is-valid');
            $(this).parent('.form-group').removeClass('o_has_error').find('.form-control, .custom-select').removeClass('is-invalid');
            $(this).siblings('.o_invalid_field').remove();
        }
        else {
            $(this).parent('.form-group').addClass('o_has_error').find('.form-control, .custom-select').addClass('is-invalid');
            $(this).parent('.form-group').removeClass('o_has_success').find('.form-control, .custom-select').removeClass('is-valid');
        }
    });

    $('select[name="pm_acquirer_id"]').on('change', function() {
        var acquirer_id = $(this).val();
        $('.acquirer').addClass('d-none');
        $('.acquirer[data-acquirer-id="'+acquirer_id+'"]').removeClass('d-none');
    });

});

```

## File: static\src\js\payment_processing.js

```javascript
odoo.define('payment.processing', function (require) {
    'use strict';

    var publicWidget = require('web.public.widget');
    var ajax = require('web.ajax');
    var core = require('web.core');

    var _t = core._t;

    $.blockUI.defaults.css.border = '0';
    $.blockUI.defaults.css["background-color"] = '';
    $.blockUI.defaults.overlayCSS["opacity"] = '0.9';

    publicWidget.registry.PaymentProcessing = publicWidget.Widget.extend({
        selector: '.o_payment_processing',
        xmlDependencies: ['/payment/static/src/xml/payment_processing.xml'],

        _pollCount: 0,

        start: function() {
            this.displayLoading();
            this.poll();
            return this._super.apply(this, arguments);
        },
        /* Methods */
        startPolling: function () {
            var timeout = 3000;
            //
            if(this._pollCount >= 10 && this._pollCount < 20) {
                timeout = 10000;
            }
            else if(this._pollCount >= 20) {
                timeout = 30000;
            }
            //
            setTimeout(this.poll.bind(this), timeout);
            this._pollCount ++;
        },
        poll: function () {
            var self = this;
            ajax.jsonRpc('/payment/process/poll', 'call', {}).then(function(data) {
                if(data.success === true) {
                    self.processPolledData(data.transactions);
                }
                else {
                    switch(data.error) {
                    case "tx_process_retry":
                        break;
                    case "no_tx_found":
                        self.displayContent("payment.no_tx_found", {});
                        break;
                    default: // if an exception is raised
                        self.displayContent("payment.exception", {exception_msg: data.error});
                        break;
                    }
                }
                self.startPolling();

            }).guardedCatch(function() {
                self.displayContent("payment.rpc_error", {});
                self.startPolling();
            });
        },
        processPolledData: function (transactions) {
            var render_values = {
                'tx_draft': [],
                'tx_pending': [],
                'tx_authorized': [],
                'tx_done': [],
                'tx_cancel': [],
                'tx_error': [],
            };

            if (transactions.length > 0 && ['transfer', 'sepa_direct_debit'].indexOf(transactions[0].acquirer_provider) >= 0) {
                window.location = transactions[0].return_url;
                return;
            }

            // group the transaction according to their state
            transactions.forEach(function (tx) {
                var key = 'tx_' + tx.state;
                if(key in render_values) {
                    render_values[key].push(tx);
                }
            });

            function countTxInState(states) {
                var nbTx = 0;
                for (var prop in render_values) {
                    if (states.indexOf(prop) > -1 && render_values.hasOwnProperty(prop)) {
                        nbTx += render_values[prop].length;
                    }
                }
                return nbTx;
            }
                       
            /*
            * When the server sends the list of monitored transactions, it tries to post-process 
            * all the successful ones. If it succeeds or if the post-process has already been made, 
            * the transaction is removed from the list of monitored transactions and won't be 
            * included in the next response. We assume that successful and post-process 
            * transactions should always prevail on others, regardless of their number or state.
            */
            if (render_values['tx_done'].length === 1 &&
                render_values['tx_done'][0].is_processed) {
                    window.location = render_values['tx_done'][0].return_url;
                    return;
            }
            // If there are multiple transactions monitored, display them all to the customer. If
            // there is only one transaction monitored, redirect directly the customer to the
            // landing route.
            if(countTxInState(['tx_done', 'tx_error', 'tx_pending', 'tx_authorized']) === 1) {
                // We don't want to redirect customers to the landing page when they have a pending
                // transaction. The successful transactions are dealt with before.
                var tx = render_values['tx_authorized'][0] || render_values['tx_error'][0];
                if (tx) {
                    window.location = tx.return_url;
                    return;
                }
            }

            this.displayContent("payment.display_tx_list", render_values);
        },
        displayContent: function (xmlid, render_values) {
            var html = core.qweb.render(xmlid, render_values);
            $.unblockUI();
            this.$el.find('.o_payment_processing_content').html(html);
        },
        displayLoading: function () {
            var msg = _t("We are processing your payment, please wait ...");
            $.blockUI({
                'message': '<h2 class="text-white"><img src="/web/static/src/img/spin.png" class="fa-pulse"/>' +
                    '    <br />' + msg +
                    '</h2>'
            });
        },
    });
});

```

## File: static\src\js\payment_transaction_portal.js

```javascript
// This file is kept to avoid breaking assets compilation on instance that didn't run "-u payment".
// TBE TODO: Remove me in master
```

## File: static\src\xml\payment_processing.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="payment" xml:space="preserve">
    <!-- The templates here as rendered by 'payment_processing.js', you can also take
            a look at payment_templates.xml (xmlid: payment_process_page) for more infos-->
    <t t-name="payment.display_tx_list">
        <div>
            <!-- Error transactions -->
            <div t-if="tx_error.length > 0">
                <h1>Payments failed</h1>
                <ul class="list-group">
                    <t t-foreach="tx_error" t-as="tx">
                        <li class="list-group-item">
                            <h4 class="list-group-item-heading mb5">
                                <t t-esc="tx['reference']"/>
                                <span class="badge pull-right"><t t-esc="tx['amount']"/> <t t-esc="tx['currency']"/></span>
                            </h4>
                            <small class="list-group-item-text">
                                An error occured during the processing of this payment.<br/>
                                <strong>Reason:</strong> <t t-esc="tx['state_message']"/>
                            </small>
                        </li>
                    </t>
                </ul>
            </div>
            <div t-if="tx_done.length > 0 || tx_authorized.length > 0 || tx_pending.length > 0">
                <h1>Payments received</h1>
                <div class="list-group">
                    <!-- Done transactions -->
                    <t t-foreach="tx_done" t-as="tx">
                        <a t-att-href="tx['return_url']" class="list-group-item">
                            <h4 class="list-group-item-heading mb5">
                                <t t-esc="tx['reference']"/>
                                <span class="badge pull-right"><t t-esc="tx['amount']"/> <t t-esc="tx['currency']"/></span>
                            </h4>
                            <small class="list-group-item-text">
                                <t t-if="!tx['is_processed']">
                                    Your order is being processed, please wait ... <i class="fa fa-cog fa-spin"/>
                                </t>
                                <t t-else="">
                                    Your order has been processed.<br/>
                                    Click here to be redirected to the confirmation page.
                                </t>
                            </small>
                        </a>
                    </t>
                    <!-- Pending transactions -->
                    <t t-foreach="tx_pending" t-as="tx">
                        <a t-att-href="tx['return_url']" class="list-group-item">
                            <h4 class="list-group-item-heading mb5">
                                <t t-esc="tx['reference']"/>
                                <span class="badge pull-right"><t t-esc="tx['amount']"/> <t t-esc="tx['currency']"/></span>
                            </h4>
                            <small class="list-group-item-text">
                                <t t-if="tx['message_to_display']">
                                    <t t-raw="tx['message_to_display']"/>
                                </t>
                                <t t-else="">
                                    Your payment is in pending state.<br/>
                                    You will be notified when the payment is fully confirmed.<br/>
                                    You can click here to be redirected to the confirmation page.
                                </t>
                            </small>
                        </a>
                    </t>
                    <!-- Authorized transactions -->
                    <t t-foreach="tx_authorized" t-as="tx">
                        <li class="list-group-item">
                            <h4 class="list-group-item-heading mb5">
                                <t t-esc="tx['reference']"/>
                                <span class="badge pull-right"><t t-esc="tx['amount']"/> <t t-esc="tx['currency']"/></span>
                            </h4>
                            <small class="list-group-item-text">
                                <t t-if="tx['message_to_display']">
                                    <t t-raw="tx['message_to_display']"/>
                                </t>
                                <t t-else="">
                                    Your payment has been received but need to be confirmed manually.<br/>
                                    You will be notified when the payment is confirmed.
                                </t>
                            </small>
                        </li>
                    </t>
                </div>
            </div>
            <!-- Draft transactions -->
            <div t-if="tx_draft.length > 0">
                <h1>Waiting for payment</h1>
                <ul class="list-group">
                    <t t-foreach="tx_draft" t-as="tx">
                        <li class="list-group-item">
                            <h4 class="list-group-item-heading mb5">
                                <t t-esc="tx['reference']"/>
                                <span class="badge pull-right"><t t-esc="tx['amount']"/> <t t-esc="tx['currency']"/></span>
                            </h4>
                            <small class="list-group-item-text">
                                <t t-if="tx['message_to_display']">
                                    <t t-raw="tx['message_to_display']"/>
                                </t>
                                <t t-else="">
                                    We are waiting for the payment acquirer to confirm the payment.
                                </t>
                            </small>
                        </li>
                    </t>
                </ul>
            </div>
            <!-- Cancel transactions -->
            <div t-if="tx_cancel.length > 0">
                <h1>Cancelled payments</h1>
                <ul class="list-group">
                    <t t-foreach="tx_cancel" t-as="tx">
                        <li class="list-group-item">
                            <h4 class="list-group-item-heading mb5">
                                <t t-esc="tx['reference']"/>
                                <span class="badge pull-right"><t t-esc="tx['amount']"/> <t t-esc="tx['currency']"/></span>
                            </h4>
                            <small class="list-group-item-text">
                                This transaction has been cancelled.<br/>
                                No payment has been processed.
                            </small>
                        </li>
                    </t>
                </ul>
            </div>
        </div>
    </t>

    <t t-name="payment.no_tx_found">
        <div class="text-center">
            <p>We are not able to find your payment, but don't worry.</p>
            <p>You should receive an email confirming your payment in a few minutes.</p>
            <p>If the payment hasn't been confirmed you can contact us.</p>
        </div>
    </t>

    <t t-name="payment.rpc_error">
        <div class="text-center">
            <p><strong>Server error:</strong> Unable to contact the Odoo server.</p>
            <p>Please wait ... <i class="fa fa-refresh fa-spin"></i></p>
        </div>
    </t>

    <t t-name="payment.exception">
        <div class="text-center">
            <h2>Internal server error</h2>
            <pre><t t-esc="exception_msg"/></pre>
        </div>
    </t>

</templates>

```

## File: views\account_invoice_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data>
        <record id="account_invoice_view_form_inherit_payment" model="ir.ui.view">
            <field name="name">account.move.view.form.inherit.payment</field>
            <field name="model">account.move</field>
            <field name="inherit_id" ref="account.view_move_form"/>
            <field name="arch" type="xml">
                <!--
                The user must capture/void the authorized transactions before registering a new payment.
                -->
                <xpath expr="//button[@id='account_invoice_payment_btn']" position="attributes">
                    <attribute name="attrs">{'invisible': ['|', '|', '|', ('state', '!=', 'posted'), ('invoice_payment_state', '!=', 'not_paid'), ('type', 'not in', ('out_invoice', 'out_refund', 'in_invoice', 'in_refund', 'out_receipt', 'in_receipt')), ('authorized_transaction_ids', '!=', [])]}</attribute>
                </xpath>
                <xpath expr="//button[@id='account_invoice_payment_btn']" position="after">
                    <field name="authorized_transaction_ids" invisible="1"/>
                    <button name="payment_action_capture" type="object"
                            string="Capture Transaction" class="oe_highlight"
                            attrs="{'invisible': ['|', '|', ('type', 'not in', ('out_invoice', 'out_refund', 'in_invoice', 'in_refund')), ('state', '!=', 'posted'), ('authorized_transaction_ids', '=', [])]}"/>
                    <button name="payment_action_void" type="object"
                            string="Void Transaction"
                            confirm="Are you sure you want to void the authorized transaction? This action can't be undone."
                            attrs="{'invisible': ['|', '|', ('type', 'not in', ('out_invoice', 'out_refund', 'in_invoice', 'in_refund')), ('state', '!=', 'posted'), ('authorized_transaction_ids', '=', [])]}"/>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\account_payment_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data>
        <record id="view_account_payment_form_inherit_payment" model="ir.ui.view">
                <field name="name">view.account.payment.form.inherit.payment</field>
                <field name="model">account.payment</field>
                <field name="inherit_id" ref="account.view_account_payment_form"/>
                <field name="arch" type="xml">
                    <xpath expr='//group[2]' position="inside">
                        <field name="payment_transaction_id" groups="base.group_no_one" attrs="{'invisible': [('payment_method_code', '!=', 'electronic')]}"/>
                    </xpath>
                    <field name="payment_method_id" position="after">
                        <field name="payment_method_code" invisible="1"/>
                        <field name="payment_token_id" options="{'no_create': True}"
                            attrs="{'invisible': [('payment_method_code', '!=', 'electronic')], 'readonly': [('state', '!=', 'draft')]}"/>
                    </field>
                </field>
        </record>
    </data>
</odoo>

```

## File: views\assets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="assets_frontend" inherit_id="web.assets_frontend">
        <xpath expr="link[last()]" position="after">
            <link rel="stylesheet" type="text/scss" href="/payment/static/src/scss/portal_payment.scss"/>
            <link rel="stylesheet" type="text/scss" href="/payment/static/src/scss/payment_form.scss"/>
        </xpath>
        <xpath expr="script[last()]" position="after">
            <script type="text/javascript" src="/payment/static/lib/jquery.payment/jquery.payment.js"></script>
            <script type="text/javascript" src="/payment/static/src/js/payment_portal.js"></script>
            <script type="text/javascript" src="/payment/static/src/js/payment_transaction_portal.js"></script>
            <script type="text/javascript" src="/payment/static/src/js/payment_form.js"></script>
            <script type="text/javascript" src="/payment/static/src/js/payment_processing.js"></script>
        </xpath>
    </template>
</odoo>

```

## File: views\payment_acquirer_onboarding_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- onboarding step -->
    <template id="onboarding_payment_acquirer_step">
        <t t-call="base.onboarding_step">
            <t t-set="title">Payment Method</t>
            <t t-set="description">Choose your default customer payment method.</t>
            <t t-set="btn_text">Set payments</t>
            <t t-set="done_text">Payment method set!</t>
            <t t-set="method" t-value="'action_open_payment_onboarding_payment_acquirer'" />
            <t t-set="model" t-value="'res.company'" />
            <t t-set="state" t-value="state.get('payment_acquirer_onboarding_state')" />
        </t>
    </template>
    <!-- add it in the account invoice onboarding panel -->
    <template id="account_invoice_payment_acquirer_onboarding"
              name="account invoice payment acquirer onboarding"
              inherit_id="account.account_invoice_onboarding_panel">
        <xpath expr="//t[@name='sample_invoice_step']" position="before">
            <t t-call="payment.onboarding_payment_acquirer_step" name="payment_acquirer_step" />
         </xpath>
    </template>
    <!--PAYMENT ACQUIRER-->
    <record model="ir.ui.view" id="payment_acquirer_onboarding_wizard_form">
        <field name="name">payment.acquirer.onboarding.wizard.form</field>
        <field name="model">payment.acquirer.onboarding.wizard</field>
        <field name="arch" type="xml">
            <form string="Choose a payment method" class="o_onboarding_payment_acquirer_wizard">
                <div class="container">
                    <div class="row align-items-start">
                        <div class="col col-4" name="left-column">
                            <group>
                                <field name="payment_method" widget="radio" nolabel="1"/>
                            </group>
                        </div>
                        <div class="col" name="right-column">
                            <div attrs="{'invisible': [('payment_method', '!=', 'paypal')]}">
                                <group>
                                    <field name="paypal_user_type" widget="radio" nolabel="1"/>
                                    <field name="paypal_email_account" attrs="{'required': [('payment_method', '=', 'paypal')]}" string="Email"/>
                                    <field name="paypal_seller_account" attrs="{'invisible': [('paypal_user_type', '=', 'new_user')], 'required': [('paypal_user_type', '!=', 'new_user'), ('payment_method', '=', 'paypal')]}" />
                                    <field name="paypal_pdt_token" password="True" attrs="{'invisible': [('paypal_user_type', '=', 'new_user')], 'required': [('paypal_user_type', '!=', 'new_user'), ('payment_method', '=', 'paypal')]}" />
                                </group>
                                <p attrs="{'invisible': [('paypal_user_type', '!=', 'new_user')]}">
                                    <span>Start selling directly without an account; an email will be sent by Paypal to create your new account and collect your payments.</span>
                                </p>
                                <p attrs="{'invisible': [('paypal_user_type', '=', 'new_user')]}">
                                    <a href="https://www.odoo.com/documentation/13.0/applications/general/payment_acquirers/paypal.html" target="_blank">
                                        <span class="fa fa-arrow-right"> How to configure your PayPal account</span>
                                    </a>
                                </p>
                            </div>
                            <div attrs="{'invisible': [('payment_method', '!=', 'stripe')]}">
                                <group>
                                    <field name="stripe_secret_key" password="True"
                                       attrs="{'required': [('payment_method', '=', 'stripe')]}" />
                                    <field name="stripe_publishable_key" password="True"
                                       attrs="{'required': [('payment_method', '=', 'stripe')]}" />
                                </group>
                                <p>
                                    <a href="https://dashboard.stripe.com/account/apikeys" target="_blank">
                                        <span class="fa fa-arrow-right"> Get my Stripe keys</span>
                                    </a>
                                </p>
                            </div>

                            <div attrs="{'invisible': [('payment_method', '!=', 'other')]}">
                                <button type="object" name="action_onboarding_other_payment_acquirer">
                                    Check here
                                </button>
                                to choose another payment method.
                            </div>
                            <div attrs="{'invisible': [('payment_method', '!=', 'manual')]}">
                                <group>
                                    <field name="manual_name" attrs="{'required': [('payment_method', '=', 'manual')]}"/>
                                    <field name="journal_name" attrs="{'required': [('payment_method', '=', 'manual')]}"/>
                                    <field name="acc_number" attrs="{'required': [('payment_method', '=', 'manual')]}"/>
                                    <field name="manual_post_msg" attrs="{'required': [('payment_method', '=', 'manual')]}"/>
                                </group>
                            </div>
                        </div>
                    </div>
                </div>
                <footer>
                    <button name="add_payment_methods" string="Apply" class="oe_highlight"
                            type="object" />
                    <button special="cancel" string="Cancel" />
                </footer>
            </form>
        </field>
    </record>
    <record id="action_open_payment_onboarding_payment_acquirer_wizard" model="ir.actions.act_window">
        <field name="name">Choose a payment method</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">payment.acquirer.onboarding.wizard</field>
        <field name="view_mode">form</field>
        <field name="view_id" ref="payment.payment_acquirer_onboarding_wizard_form" />
        <field name="target">new</field>
    </record>
</odoo>

```

## File: views\payment_portal_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
    <template id="pay_methods" name="Payment Methods">
        <t t-call="portal.frontend_layout">
            <t t-set="additional_title">Payment Methods</t>
            <div class="wrap">
                <div class="container">
                  <div class="row">
                        <div class="col-md-6">
                            <ol class="breadcrumb mt8">
                                <li class="breadcrumb-item"><a href="/my/home"><i class="fa fa-home" role="img" aria-label="Home" title="Home"/></a></li>
                                <li class="breadcrumb-item">Payment Methods</li>
                            </ol>
                        </div>
                    </div>
                    <div class="clearfix">
                        <div class="row">
                            <div class="col-md-8">
                                <h3>Manage Payment Methods</h3>
                                <t t-call="payment.payment_tokens_list">
                                    <t t-set="acquirers" t-value="acquirers"/>
                                    <t t-set="mode" t-value="'manage'"/>
                                </t>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </t>
    </template>

    <!-- TDE FIXME: was customize_show=True -->
    <template id="pay_meth_link" inherit_id="portal.portal_layout">
        <xpath expr="//div[hasclass('o_portal_my_details')]" position="inside">
            <t t-if="request.env['payment.acquirer'].search([('state', 'in', ['enabled', 'test']), ('registration_view_template_id', '!=', False), ('payment_flow', '=', 's2s'), ('company_id', '=', request.env.company.id)])">
                <div class='manage_payment_method mt16'><a href="/my/payment_method">Manage payment methods</a></div>
            </t>
        </xpath>
    </template>

    <template id="pay">
        <t t-call="portal.frontend_layout">
            <t t-set="additional_title">Payment</t>
            <div class="wrap">
                <div class="container o_website_payment">
                    <h1>Payment</h1>
                    <div class="row mt32 mb32">
                        <div class="col-lg-7">
                            <p><b>Reference:</b> <t t-esc="reference"/></p>
                            <p><b>Amount:</b> <t t-esc="amount" t-options="{'widget': 'monetary', 'display_currency': currency}"/></p>
                            <t t-call="payment.payment_tokens_list" t-if="reference and amount and currency">
                                <t t-set="mode" t-value="'payment'"/>
                                <t t-set="_overriden_reference" t-value="reference"/>
                                <t t-set="reference" t-value="quote_plus(reference).replace('+','%20')"/>
                                <t t-set="prepare_tx_url" t-value="'/website_payment/transaction/v2/' + str(amount) + '/' + str(currency.id) + '/' + reference + (('/' + str(partner_id)) if partner_id else '')"/>
                                <t t-set="form_action" t-value="'/website_payment/token/v2/' + str(amount) + '/' + str(currency.id) + '/' + reference + (('/' + str(partner_id)) if partner_id else '')"/>
                                <t t-set="reference" t-value="_overriden_reference"/>
                            </t>
                            <div t-if="not acquirers" class="alert alert-danger" role="alert">
                                <p>No payment acquirer found.</p>
                                <p>Please configure a payment acquirer.</p>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </t>
    </template>

    <template id="confirm">
        <t t-call="portal.frontend_layout">
            <t t-set="additional_title">Payment</t>
            <div class="wrap">
                <div class="container o_website_payment">
                    <h1>Payment</h1>
                    <div class="row">
                        <div class="col-lg-6">
                            <div>
                                <div t-attf-class="alert alert-#{status}" t-raw="message" role="status"/>
                                <div class="form-group row">
                                    <label for="form_partner_name" class="col-md-3 col-form-label">From</label>
                                    <div class="col-md-9">
                                        <span name="form_partner_name" class="form-control" t-esc="tx.partner_name"/>
                                    </div>
                                </div>
                                <div class="form-group row">
                                    <label for="form_reference" class="col-md-3 col-form-label">Reference</label>
                                    <div class="col-md-9">
                                        <span name="form_reference" class="form-control" t-esc="tx.reference"/>
                                    </div>
                                </div>
                                <div class="form-group row">
                                    <label for="form_amount" class="col-md-3 col-form-label">Amount</label>
                                    <div class="col-md-9">
                                        <span name="form_amount" class="form-control" t-esc="tx.amount" t-options="{'widget': 'monetary', 'display_currency': tx.currency_id}"/>
                                    </div>
                                </div>
                                <div t-if="(tx.acquirer_id.qr_code == True) and (tx.currency_id.name == 'EUR')">
                                    <div t-if="tx.acquirer_id.journal_id.bank_account_id.qr_code_valid">
                                        <h3>Or scan me with your banking app.</h3>
                                        <img class="border border-dark rounded" t-att-src="tx.acquirer_id.journal_id.bank_account_id.build_qr_code_base64(tx.amount,tx.reference)"/>
                                    </div>
                                    <div t-if="(tx.acquirer_id.journal_id.bank_account_id.qr_code_valid == False)">
                                        <h3>The SEPA QR Code informations are not set correctly.</h3>
                                    </div>
                                </div>
                                <div class="col-md-9 offset-md-3 text-muted mt16 pl-md-2 pl-0"><!-- FIXME should be in a row... -->
                                    <span t-field="tx.acquirer_id.image_128" t-att-title="tx.acquirer_id.name" role="img" t-att-aria-label="tx.acquirer_id.name" t-options='{"widget": "image", "style":"max-width: 60px; display: inline-block"}'/>
                                    <span>Processed by <t t-esc="tx.acquirer_id.name"/>.</span>
                                </div>
                                <div>
                                    <a role="button" t-attf-class="btn btn-#{status} float-right" href="/my/home"><i class="fa fa-arrow-circle-right"/> Back to My Account</a>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </t>
    </template>

    <template id="payment_confirmation_status">
        <!--
        Inform the portal user about the transaction status.

        To be called with the 'payment_tx_id' value.
        -->
        <div t-if="payment_tx_id and payment_tx_id.state == 'pending'" class="alert alert-warning alert-dismissable" role="status">
            <button type="button" class="close" data-dismiss="alert" aria-label="Close">&amp;times;</button>
            <span t-if='payment_tx_id.acquirer_id.pending_msg' t-raw="payment_tx_id.acquirer_id.pending_msg"/>
            <span t-if='thanks_msg' t-raw="thanks_msg"/>
            <div t-if="payment_tx_id.acquirer_id.provider == 'transfer' and reference">
                <b>Communication: </b><span t-esc='reference'/>
            </div>
            <div t-if="(payment_tx_id.acquirer_id.qr_code == True) and (payment_tx_id.currency_id.name == 'EUR')">
                <div t-if="payment_tx_id.acquirer_id.journal_id.bank_account_id.qr_code_valid">
                    <h3>Or scan me with your banking app.</h3>
                    <img class="border border-dark rounded" t-att-src="payment_tx_id.acquirer_id.journal_id.bank_account_id.build_qr_code_base64(payment_tx_id.amount,payment_tx_id.reference)"/>
                </div>
                <div t-if="(payment_tx_id.acquirer_id.journal_id.bank_account_id.qr_code_valid == False)">
                    <h3>The SEPA QR Code informations are not set correctly.</h3>
                </div>
            </div>
        </div>
        <div t-if="payment_tx_id and payment_tx_id.state == 'authorized' and payment_tx_id.acquirer_id.authorize_implemented" class="alert alert-success alert-dismissable" role="alert">
            <button type="button" class="close" data-dismiss="alert" title="Dismiss" aria-label="Dismiss">&amp;times;</button>
            <!-- Your payment has been authorized. -->
            <span t-if='payment_tx_id.acquirer_id.auth_msg' t-raw="payment_tx_id.acquirer_id.auth_msg"/>
        </div>
        <div t-if="payment_tx_id and payment_tx_id.state == 'done'" class="alert alert-success alert-dismissable">
            <button type="button" class="close" data-dismiss="alert" title="Dismiss" aria-label="Dismiss">&amp;times;</button>
            <span t-if='payment_tx_id.acquirer_id.done_msg' t-raw="payment_tx_id.acquirer_id.done_msg"/>
            <span t-if='thanks_msg' t-raw="thanks_msg"/>
            <div t-if="thanks_msg and payment_tx_id.acquirer_id.provider == 'transfer' and reference">
                <b>Communication: </b><span t-esc='reference'/>
            </div>
        </div>
        <div t-if="payment_tx_id and payment_tx_id.state == 'cancel'" class="alert alert-danger alert-dismissable">
            <button type="button" class="close" data-dismiss="alert" title="Dismiss" aria-label="Dismiss">&amp;times;</button>
            <span t-if='payment_tx_id.acquirer_id.cancel_msg' t-raw="payment_tx_id.acquirer_id.cancel_msg"/>
        </div>
        <div t-if="payment_tx_id and payment_tx_id.state_message">
            <span t-esc="payment_tx_id.state_message"/>
        </div>
    </template>

    </data>
</odoo>

```

## File: views\payment_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="assets_backend" name="Payment Assets Backend" inherit_id="web.assets_backend">
        <xpath expr="." position="inside">
            <link rel="stylesheet" type="text/scss" href="/payment/static/src/scss/payment_acquirer.scss"/>
        </xpath>
    </template>

    <template id="payment_tokens_list" name="Payment Tokens list">
        <!--
        Variables description:
            - 'submit_txt' the text displayed inside the submit button
            - 'submit_class' the css classes to style the submit button
            - 'icon_class' font awesome class (e.g. 'fa-trash', 'fa-lock')
            - 'form_action' the URI to the page that will handle the form values given for server2server
            - 'pms' the tokens
            - 'checked_pm_id' the payment token that should be checked (for radio buttons)
            - 'mode' can take two values, either 'payment' or 'manage'. 'manage' displays the add a new card and delete buttons. 'payment'
                display a form that is used to pay and send the information to the form action url.
            - 'acquirers' the list of both server2server and form payment acquirers
            - 'verify_validity' if we need to verify if the payment method is valid when adding a new one
            - 'prepare_tx_url' the url of the route which will handle the creation of a transaction for a form base payment (handles if the transaction is form or form_save)
            - 'show_manage_btn' if True, a button is added in the footer to manage payment methods
        -->
        <form t-if="pms or acquirers" method="post" class="o_payment_form mt-3 clearfix"
                t-att-action="form_action if form_action else '#'"
                t-att-data-success-url="success_url or ''"
                t-att-data-error-url="error_url or ''"
                t-att-data-access-token="access_token or ''"
                t-att-data-partner-id="partner_id"
                t-att-data-callback-method="callback_method or ''"
                t-att-data-order-id="order_id or ''"
                t-att-data-invoice-id="invoice_id or ''"
                t-att-data-mode="mode">
            <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
            <input type="hidden" t-if="prepare_tx_url" name="prepare_tx_url" t-att-value="prepare_tx_url"/>
            <input type="hidden" t-if="order_id" name="order_id" t-att-value="order_id"/>
            <input type="hidden" t-if="invoice_id" name="invoice_id" t-att-value="invoice_id"/>
            <!-- s2s form submission -->
            <input type="hidden" t-if="access_token" name="access_token" t-att-value="access_token"/>
            <input type="hidden" t-if="success_url" name="success_url" t-att-value="success_url"/>
            <input type="hidden" t-if="error_url" name="error_url" t-att-value="error_url"/>
            <input type="hidden" t-if="callback_method" name="callback_method" t-att-value="callback_method"/>

            <div class="card">
                <t t-set="acquirers_count" t-value="len(acquirers) if acquirers else 0"/>
                <t t-set="pms_count" t-value="len(pms) if pms else 0"/>
                <t t-set="MAX_BRAND_LINE" t-value="3"/>
                <t t-foreach="acquirers" t-as="acq">
                    <div class="card-body o_payment_acquirer_select">
                        <label>
                            <t t-if="acq.payment_flow == 'form'">
                                <input type="radio" t-att-data-acquirer-id="acq.id"
                                       t-att-data-form-payment="true"
                                       t-att-data-provider="acq.provider"
                                       t-att-class="'d-none' if (acquirers_count==1 and pms_count==0) else ''"
                                       name="pm_id" t-attf-value="form_{{acq.id}}"
                                       t-att-checked="acquirers_count==1 and pms_count==0 or acquirers[0] == acq"/>
                            </t>
                            <t t-else="acq.payment_flow == 's2s'">
                                <input type="radio" t-att-data-acquirer-id="acq.id"
                                       t-att-data-s2s-payment="true"
                                       t-att-data-provider="acq.provider"
                                       name="pm_id" t-attf-value="new_{{acq.id}}"
                                       t-att-class="'d-none' if (acquirers_count==1 and pms_count==0) else ''"
                                       t-att-checked="acquirers_count==1 and pms_count==0 or acquirers[0] == acq"/>
                            </t>
                            <span class="payment_option_name">
                              <t t-esc="acq.display_as or acq.name"/>
                              <div t-if="acq.state == 'test'" class="badge-pill badge-warning float-right" style="margin-left:5px">
                                Test Mode
                              </div>
                            </span>
                            <t t-if="acq_extra_fees and acq_extra_fees.get(acq)">
                                <span class="badge badge-pill badge-secondary"> + <t t-esc="acq_extra_fees[acq]" t-options='{"widget": "monetary", "display_currency": acq_extra_fees["currency_id"]}'/> Fee </span>
                            </t>
                            <t t-elif="acq.fees_active">
                                <small class="text-muted">(Some fees may apply)</small>
                            </t>
                        </label>
                        <ul class="float-right list-inline payment_icon_list">
                            <t t-set="i" t-value="0"/>
                            <t t-foreach="acq.payment_icon_ids" t-as="pm_icon">
                                <li t-attf-class="list-inline-item#{'' if (i &lt; MAX_BRAND_LINE) else ' d-none'}">
                                    <span t-field="pm_icon.image_payment_form"
                                          t-options='{"widget": "image", "alt-field": "name"}'/>
                                </li>
                                <li t-if="i==MAX_BRAND_LINE" style="display:block;" class="list-inline-item">
                                    <span class="float-right more_option text-info">
                                        <a href="#" class="o_payment_form_pay_icon_more" data-toggle="tooltip" t-att-title="', '.join([opt.name for opt in acq.payment_icon_ids[MAX_BRAND_LINE:]])">and more</a>
                                    </span>
                                </li>
                                <t t-set="i" t-value="i+1"/>
                            </t>
                        </ul>
                        <div t-raw="acq.pre_msg" class="text-muted ml-3"/>
                    </div>
                    <t t-if="acq.payment_flow == 'form'">
                        <div t-attf-id="o_payment_form_acq_{{acq.id}}"
                             t-attf-class="d-none {{'card-footer' if acq.save_token == 'ask' else ''}}">
                            <label t-if="acq.save_token == 'ask'">
                                <input type="checkbox" name="o_payment_form_save_token" data-remove-me=""/>
                                Save my payment data
                            </label>
                            <t t-if="acq.save_token == 'always'">
                                <input type="checkbox" name="o_payment_form_save_token" checked="'checked'" class="o_hidden" data-remove-me=""/>
                            </t>
                        </div>
                    </t>
                    <t t-else="acq.payment_flow == 's2s'">
                        <div t-attf-id="o_payment_add_token_acq_{{acq.id}}"
                             t-attf-class="card-footer {{'d-none' if(acquirers_count &gt; 1 and pms_count==0 and acquirers[0]!=acq) else 'd-none' if pms_count &gt;0 else ''}}">
                            <div class="clearfix">
                                <input type="hidden" t-if="(verify_validity==True or mode == 'manage') and acq.check_validity" name="verify_validity" t-att-value="acq.check_validity"/>
                                <t t-call="{{acq.sudo().get_s2s_form_xml_id()}}">
                                    <t t-set="id" t-value="acq.id"/>
                                    <t t-set="partner_id" t-value="partner_id"/>
                                    <t t-if="not return_url" t-set="return_url" t-value="''"/>
                                </t>
                            </div>
                        </div>
                    </t>
                </t>
                <t t-foreach="pms" t-as="pm">
                    <t t-if="not verify_validity or (pm.acquirer_id.check_validity and pm.verified) or not pm.acquirer_id.check_validity">
                        <div class="card-body o_payment_acquirer_select">
                            <label>
                                <input t-if="mode == 'payment'" type="radio" name="pm_id" t-att-value="pm.id" t-att-checked="checked_pm_id == pm.id"/>
                                <span class="payment_option_name" t-esc="pm.name"/>
                                <t t-if="pm.verified">
                                    <i class="fa fa-check text-success" title="This payment method is verified by our system." role="img" aria-label="Ok"></i>
                                </t>
                                <t t-else="">
                                    <i class="fa fa-check text-muted" title="This payment method has not been verified by our system." role="img" aria-label="Not verified"></i>
                                </t>
                            </label>
                            <button t-if="mode == 'manage'" name="delete_pm" t-att-value="pm.id" class="btn btn-primary btn-sm float-right">
                                <i class="fa fa-trash"></i> Delete
                            </button>
                        </div>
                    </t>
                </t>
            </div>
            <div t-if='back_button_txt' class="float-left mt-2">
                <a role="button" t-att-href="back_button_link or '#'" t-att-class="back_button_class or 'btn btn-lg btn-secondary'">
                    <i t-if="back_button_icon_class" t-attf-class="fa {{back_button_icon_class}}"/>
                    <t t-esc="back_button_txt"/>
                </a>
            </div>
            <div class="float-right mt-2">
                <button t-if="mode == 'payment'" id="o_payment_form_pay" type="submit" t-att-class="submit_class if submit_class else 'btn btn-primary btn-lg mb8 mt8'" disabled="true">
                    <t t-if="submit_txt">
                        <i t-if="icon_class and not icon_right" t-attf-class="fa {{icon_class}}"/>
                        <t t-esc="submit_txt"/>
                        <i t-if="icon_class and icon_right" t-attf-class="fa {{icon_class}}"/>
                    </t>
                    <t t-else="">
                        <i class="fa fa-lock"/> Pay
                    </t>
                </button>
                <t t-if="show_manage_btn">
                    <a class="btn btn-link mb8 mt8" href="/my/payment_method">Manage your payment methods</a>
                </t>
                <button t-if="mode == 'manage' and list(filter(lambda x: x.payment_flow == 's2s', acquirers))" type="submit" id="o_payment_form_add_pm" class="btn btn-primary btn-lg mb8 mt8">
                    <i class="fa fa-plus-circle"/> Add new card
                </button>
            </div>
        </form>
    </template>

    <template id="payment_process_page" name="Payment processing page">
        <t t-call="portal.frontend_layout">
            <div class="wrap">
                <div class="container o_website_payment">
                    <div class="o_payment_processing">
                        <div class="row">
                            <div class="o_payment_processing_content col-sm-6 col-sm-offset-3">
                                <!-- The content here is generated in JS -->
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </t>
    </template>
</odoo>

```

## File: views\payment_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record id="acquirer_form" model="ir.ui.view">
            <field name="name">payment.acquirer.form</field>
            <field name="model">payment.acquirer</field>
            <field name="arch" type="xml">
                <form string="Payment Acquirer">
                    <field name="fees_implemented" invisible='1'/>
                    <field name="token_implemented" invisible='1'/>
                    <field name="authorize_implemented" invisible="1"/>
                    <sheet>
                        <field name="module_id" invisible="1"/>
                        <field name="module_state" invisible="1"/>
                        <field name="module_to_buy" invisible="1"/>
                        <field name="inbound_payment_method_ids" invisible="1"/>
                        <field name="image_128" widget="image" class="oe_avatar"/>
                        <widget name="web_ribbon" title="Disabled" bg_color="bg-danger" attrs="{'invisible': [('state', '!=', 'disabled')]}"/>
                        <widget name="web_ribbon" title="Test Mode" bg_color="bg-warning" attrs="{'invisible': [('state', '!=', 'test')]}"/>
                        <div class="oe_title">
                            <h1><field name="name" placeholder="Name"/></h1>
                            <div attrs="{'invisible': ['|', ('module_state', '=', 'installed'), ('module_id', '=', False)]}">
                                <a attrs="{'invisible': [('module_to_buy', '=', False)]}" href="https://odoo.com/pricing?utm_source=db&amp;utm_medium=module" class="btn btn-info" role="button">Upgrade</a>
                                <button attrs="{'invisible': [('module_to_buy', '=', True)]}" type="object" class="btn btn-primary" name="button_immediate_install" string="Install"/>
                            </div>
                        </div>
                        <div attrs="{'invisible': ['|', ('module_state', '=', 'installed'), ('module_id', '=', False)]}">
                            <div class="o_payment_acquirer_desc">
                                <field name="description"/>
                            </div>
                        </div>
                        <group>
                            <group name="payment_state">
                                <field name="provider" groups="base.group_no_one" attrs="{'invisible': [('module_id', '!=', False), ('module_state', '!=', 'installed')]}"/>
                                <field name="state" widget="radio" attrs="{'invisible': [('module_state', '=', 'uninstalled')]}"/>
                                <field name="company_id" groups="base.group_multi_company" options='{"no_open":True}'/>
                            </group>
                        </group>
                        <notebook attrs="{'invisible': ['&amp;', ('module_id', '!=', False), ('module_state', '!=', 'installed')]}">
                            <page string="Credentials" name="acquirer_credentials" attrs="{'invisible': ['provider', '=', 'manual']}">
                                <group name="acquirer">
                                </group>
                            </page>
                            <page string="Configuration">
                                <group name="acquirer_config">
                                    <group string="Payment Form" name="payment_form">
                                        <field name="display_as" placeholder="If not defined, the acquirer name will be used."/>
                                        <field name="payment_icon_ids" widget="many2many_tags"/>
                                        <field name="save_token" widget="radio" attrs="{'invisible': ['|', ('token_implemented', '=', False), ('payment_flow', '=', 's2s')]}"/>
                                        <field name="capture_manually" attrs="{'invisible': [('authorize_implemented', '=', False)]}"/>
                                        <field name="payment_flow" widget="radio" attrs="{'invisible': [('token_implemented', '=', False)]}"/>
                                        <field name="view_template_id" groups="base.group_no_one"/>
                                        <field name="registration_view_template_id" groups="base.group_no_one" attrs="{'invisible': [('payment_flow', '!=', 's2s')]}"/>
                                        <field name="check_validity" attrs="{'invisible': [('payment_flow', '!=', 's2s')]}" groups="base.group_no_one"/>
                                        <field name="qr_code" attrs="{'invisible': [('provider', '!=', 'transfer')]}"/>
                                    </group>
                                    <group string="Availability" name="availability">
                                        <field name="country_ids" widget="many2many_tags" placeholder="Select countries. Leave empty to use everywhere."/>
                                    </group>
                                    <group string="Payment Followup" name="payment_followup">
                                        <field name="journal_id" context="{'default_type': 'bank'}"
                                          attrs="{'required': [('state', '!=', 'disabled'), ('provider', 'not in', ['manual', 'transfer'])]}"/>
                                    </group>
                                </group>
                            </page>
                            <page string="Fees" attrs="{'invisible': [('fees_implemented', '=', False)]}">
                                <group name="payment_fees">
                                    <field name="fees_active"/>
                                    <field name="fees_dom_fixed" attrs="{'invisible': [('fees_active', '=', False)]}"/>
                                    <field name="fees_dom_var" attrs="{'invisible': [('fees_active', '=', False)]}"/>
                                    <field name="fees_int_fixed" attrs="{'invisible': [('fees_active', '=', False)]}"/>
                                    <field name="fees_int_var" attrs="{'invisible': [('fees_active', '=', False)]}"/>
                                </group>
                            </page>
                            <page string="Messages" attrs="{'invisible': [('module_id', '=', True), ('module_state', '!=', 'installed')]}">
                                <group>
                                    <field name="pre_msg"/>
                                    <field name="auth_msg" attrs="{'invisible': [('authorize_implemented', '=', False)]}"/>
                                    <field name="pending_msg"/>
                                    <field name="done_msg"/>
                                    <field name="cancel_msg"/>
                                </group>
                            </page>
                        </notebook>
                    </sheet>
                </form>
            </field>
        </record>


        <!-- Acquirer Kanban View -->
        <record id="acquirer_kanban" model="ir.ui.view">
            <field name="name">payment.acquirer.kanban</field>
            <field name="model">payment.acquirer</field>
            <field name="arch" type="xml">
                <kanban quick_create="false" create="true" class="o_kanban_payment_acquirer o_kanban_dashboard">
                    <field name="id"/>
                    <field name="name"/>
                    <field name="description"/>
                    <field name="provider"/>
                    <field name="module_id"/>
                    <field name="module_state"/>
                    <field name="module_to_buy"/>
                    <field name="color"/>
                    <templates>
                        <t t-name="kanban-box">
                            <t t-set="installed" t-value="!record.module_id.value || (record.module_id.value &amp;&amp; record.module_state.raw_value === 'installed')"/>
                            <t t-set="to_buy" t-value="record.module_to_buy.raw_value === true"/>
                            <div t-attf-class="oe_kanban_global_click #{kanban_color(record.color.raw_value)}">
                                <div class="o_payment_acquirer_desc">
                                    <div class="o_kanban_image">
                                        <img type="open" t-att-src="kanban_image('payment.acquirer', 'image_128', record.id.raw_value)" alt="Acquirer"/>
                                    </div>
                                    <h3 class="mt4"><t t-esc="record.name.value"/></h3>
                                    <t t-if="record.description.raw_value" t-raw="record.description.raw_value"/>
                                </div>
                                <div class="o_payment_acquirer_bottom">
                                    <t t-if="installed">
                                        <field name="state" widget="label_selection" options="{'classes': {'enabled': 'success', 'test': 'warning', 'disabled' : 'danger'}}"/>
                                    </t>
                                    <button t-if="!installed and !selection_mode and !to_buy" type="object" class="btn btn-secondary float-right" name="button_immediate_install">Install</button>
                                    <t t-if="!installed and to_buy">
                                        <button href="https://odoo.com/pricing?utm_source=db&amp;utm_medium=module" class="btn btn-info btn-sm float_right">Upgrade</button>
                                        <span class="badge badge-primary oe_inline o_enterprise_label">Enterprise</span>
                                    </t>
                                    <button t-if="installed and record.state.raw_value == 'disabled' and !selection_mode" type="edit" class="btn btn-primary float-right">Activate</button>
                                    <button t-if="installed and record.state.raw_value in ['enabled', 'test'] and !selection_mode" type="edit" class="btn btn-primary float-right">Configure</button>
                                </div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="acquirer_list" model="ir.ui.view">
            <field name="name">payment.acquirer.list</field>
            <field name="model">payment.acquirer</field>
            <field name="arch" type="xml">
                <tree string="Payment Acquirers">
                    <field name="sequence" widget="handle"/>
                    <field name="name"/>
                    <field name="provider"/>
                    <field name="state"/>
                    <field name="country_ids" widget="many2many_tags" optional="hide"/>
                    <field name="company_id" groups="base.group_multi_company"/>
                </tree>
            </field>
        </record>

        <record id="acquirer_search" model="ir.ui.view">
            <field name="name">payment.acquirer.search</field>
            <field name="model">payment.acquirer</field>
            <field name="arch" type="xml">
                <search>
                    <field name="name" string="Acquirer" filter_domain="['|', ('name','ilike',self), ('description','ilike',self)]"/>
                    <field name="provider"/>
                    <filter name="acquirer_installed" string="Installed" domain="[('provider', '!=', 'manual')]"/>
                    <group expand="0" string="Group By">
                        <filter string="Provider" name="provider" domain="[]" context="{'group_by': 'provider'}"/>
                        <filter string="State" name="state" domain="[]" context="{'group_by': 'state'}"/>
                    </group>
                </search>
            </field>
        </record>

        <record id="action_payment_acquirer" model="ir.actions.act_window">
            <field name="name">Payment Acquirers</field>
            <field name="res_model">payment.acquirer</field>
            <field name='view_mode'>kanban,tree,form</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Create a new payment acquirer
                </p>
            </field>
        </record>

        <menuitem
            action='action_payment_acquirer'
            id='payment_acquirer_menu'
            parent='account.root_payment_menu'
            sequence='10' />

        <!-- Payment transactions -->
        <record id="transaction_form" model="ir.ui.view">
            <field name="name">payment.transaction.form</field>
            <field name="model">payment.transaction</field>
            <field name="arch" type="xml">
                <form string="Payment Transactions" create="false" edit="false">
                    <header>
                        <button type="object" name="action_capture" states="authorized" string="Capture Transaction" class="oe_highlight"/>
                        <button type="object" name="action_void" states="authorized" string="Void Transaction"
                                confirm="Are you sure you want to void the authorized transaction? This action can't be undone."/>
                        <field name="state" widget="statusbar"/>
                    </header>
                    <sheet>
                        <div class="oe_button_box" name="button_box">
                            <button name="action_view_invoices" type="object"
                                    class="oe_stat_button" icon="fa-money"
                                    attrs="{'invisible': [('invoice_ids_nbr', '=', 0)]}">
                                <field name="invoice_ids_nbr" widget="statinfo" string="Invoice(s)"/>
                            </button>
                        </div>
                        <group>
                            <group>
                                <field name="reference"/>
                                <field name="payment_id"/>
                                <label for="amount"/>
                                <div class="o_row">
                                    <field name="amount"/>
                                    <field name="currency_id" options="{'no_open': True, 'no_create': True}"/>
                                </div>
                                <field name="fees" readonly="1"/>
                                <label for="partner_id"/>
                                <div>
                                    <field name="partner_id" context="{'show_address': 1}" options="{'always_reload': True}"/>
                                    <div name="partner_details" class="o_address_format" attrs="{'invisible': [('partner_id', '!=', False)]}">
                                        <field name="partner_name" placeholder="Name" attrs="{'readonly': [('partner_id', '!=', False)]}" class="o_address_street"/>
                                        <field name="partner_address" placeholder="Address" attrs="{'readonly': [('partner_id', '!=', False)]}" class="o_address_street"/>
                                        <field name="partner_city" placeholder="City" attrs="{'readonly': [('partner_id', '!=', False)]}" class="o_address_city"/>
                                        <field name="partner_zip" placeholder="ZIP" attrs="{'readonly': [('partner_id', '!=', False)]}" class="o_address_zip"/>
                                        <field name="partner_country_id" placeholder="Country" attrs="{'readonly': [('partner_id', '!=', False)]}" class="o_address_country"/>
                                        <field name="partner_lang" attrs="{'readonly': [('partner_id', '!=', False)]}" class="o_address_street"/>
                                        <field name="partner_email" placeholder="E-mail" attrs="{'readonly': [('partner_id', '!=', False)]}" class="o_address_street"/>
                                    </div>
                                </div>
                            </group>
                            <group>
                                <field name="acquirer_id"/>
                                <field name="provider" invisible="1"/>
                                <field name="payment_token_id" options="{'no_create': True}"/>
                                <field name="acquirer_reference" readonly="1"/>
                                <field name="date"/>
                            </group>
                        </group>
                        <group string="Message">
                            <field name="state_message" nolabel="1"/>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="transaction_list" model="ir.ui.view">
            <field name="name">payment.transaction.list</field>
            <field name="model">payment.transaction</field>
            <field name="arch" type="xml">
                <tree string="Payment Transactions" create="false">
                    <field name="reference"/>
                    <field name="create_date"/>
                    <field name="acquirer_id"/>
                    <field name="partner_id"/>
                    <field name="partner_name"/>
                    <field name="amount"/>
                    <field name="fees"/>
                    <field name="state"/>
                </tree>
            </field>
        </record>

        <record id="transaction_view_kanban" model="ir.ui.view">
            <field name="name">payment.transaction.kanban</field>
            <field name="model">payment.transaction</field>
            <field name="arch" type="xml">
                <kanban class="o_kanban_mobile" create="false">
                    <templates>
                        <t t-name="kanban-box">
                            <div t-attf-class="oe_kanban_content oe_kanban_global_click">
                                <div class="row">
                                    <div class="col-6">
                                        <strong><field name="reference"/></strong>
                                    </div>
                                    <div class="col-6">
                                        <span><field name="partner_name"/></span>
                                    </div>
                                    <div class="col-6">
                                        <span class="float-right">
                                            <field name="amount" widget="monetary"/>
                                        </span>
                                    </div>
                                </div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="transaction" model="ir.ui.view">
            <field name="name">payment.transaction.search</field>
            <field name="model">payment.transaction</field>
            <field name="arch" type="xml">
                <search>
                    <field name="reference"/>
                    <field name="acquirer_id"/>
                    <field name="partner_id"/>
                    <field name="partner_name"/>
                </search>
            </field>
        </record>

        <record id="action_payment_transaction" model="ir.actions.act_window">
            <field name="name">Payment Transactions</field>
            <field name="res_model">payment.transaction</field>
            <field name='view_mode'>tree,kanban,form</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Create a new payment transaction
              </p>
            </field>
        </record>

        <menuitem
            action='action_payment_transaction'
            id='payment_transaction_menu'
            parent='account.root_payment_menu'
            groups="base.group_no_one"
            sequence='20' />

        <!-- Payment Tokens -->
        <record model='ir.ui.view' id='payment_token_tree_view'>
            <field name='name'>payment.token.tree</field>
            <field name='model'>payment.token</field>
            <field name='arch' type='xml'>
                <tree string='Payment Tokens'>
                    <field name="name"/>
                    <field name="active"/>
                    <field name='partner_id' />
                    <field name='acquirer_id' readonly='1'/>
                    <field name='acquirer_ref' readonly='1'/>
                </tree>
            </field>
        </record>

        <record id='payment_token_view_search' model='ir.ui.view'>
            <field name='name'>payment.token.search</field>
            <field name='model'>payment.token</field>
            <field name='arch' type='xml'>
                <search string='Payment Tokens'>
                    <field name='partner_id'/>
                    <separator/>
                    <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
                </search>
            </field>
        </record>

        <record id="action_payment_tx_ids" model="ir.actions.act_window">
            <field name="name">Payment Transactions</field>
            <field name="res_model">payment.transaction</field>
            <field name="view_mode">tree,form</field>
            <field name="domain">[('payment_token_id','=', active_id)]</field>
            <field name="context">{'create': False}</field>
        </record>

        <record model='ir.ui.view' id='payment_token_form_view'>
            <field name='name'>payment.token.form</field>
            <field name='model'>payment.token</field>
            <field name='arch' type='xml'>
                <form string='Payment Tokens' create='false' editable='bottom'>
                    <sheet>
                        <div class="oe_button_box" name="button_box">
                            <button class="oe_stat_button" name="%(action_payment_tx_ids)d"
                                type="action" icon="fa-money" string="Payments">
                            </button>
                        </div>
                        <group>
                            <field name="name"/>
                            <field name='partner_id' />
                        </group>
                        <group>
                            <field name="active"/>
                            <field name='acquirer_id'/>
                            <field name='acquirer_ref'/>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <record model='ir.actions.act_window' id='payment_token_action'>
            <field name='name'>Saved Payment Data</field>
            <field name='res_model'>payment.token</field>
            <field name='view_mode'>tree,form</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Create a saved payment data
                </p>
            </field>
        </record>

        <menuitem
            action='payment_token_action'
            id='payment_token_menu'
            parent='account.root_payment_menu'
            groups='base.group_no_one'/>

        <!-- Payment icons -->
        <record model="ir.ui.view" id="payment_icon_form_view">
            <field name="name">payment.icon.form</field>
            <field name="model">payment.icon</field>
            <field name="arch" type="xml">
                <form string="Payment Icon">
                    <sheet>
                        <field name="image" widget="image" class="oe_avatar"/>
                        <div class="oe_title">
                            <h1><field name="name" placeholder="Name"/></h1>
                        </div>
                        <notebook>
                            <page string="Acquirers list">
                                <field nolabel="1" name="acquirer_ids"/>
                            </page>
                        </notebook>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="action_payment_icon" model="ir.actions.act_window">
            <field name="name">Payment Icons</field>
            <field name="res_model">payment.icon</field>
            <field name="view_mode">tree,form</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Create a payment icon
                </p>
            </field>
        </record>

        <menuitem
            action="action_payment_icon"
            id="payment_icon_menu"
            parent="account.root_payment_menu"
            groups="base.group_no_one"/>
    </data>
</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data>

        <!-- Add creditcard to res.partner -->
        <record id="view_partners_form_payment_defaultcreditcard" model="ir.ui.view">
                <field name="name">view.res.partner.form.payment.defaultcreditcard</field>
                <field name="model">res.partner</field>
                <field name="inherit_id" ref="base.view_partner_form"/>
                <field name="priority" eval="15"/>
                <field name="arch" type="xml">
                    <div name="button_box" position="inside">
                        <button type="action" class="oe_stat_button"
                                icon="fa-credit-card"
                                name="%(payment.payment_token_action)d"
                                context="{'search_default_partner_id': active_id, 'create': False, 'edit': False}"
                                attrs="{'invisible': [('payment_token_count', '=', 0)]}">
                            <div class="o_form_field o_stat_info">
                                <span class="o_stat_value">
                                    <field name="payment_token_count" widget="statinfo" nolabel="1"/>
                                </span>
                                <span class="o_stat_text">Credit Cards</span>
                            </div>
                        </button>
                    </div>
                </field>
        </record>
    </data>
</odoo>

```

## File: wizards\payment_acquirer_onboarding_wizard.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, fields, _
from odoo.exceptions import UserError


class PaymentWizard(models.TransientModel):
    _name = 'payment.acquirer.onboarding.wizard'
    _description = 'Payment acquire onboarding wizard'

    payment_method = fields.Selection([
        ('paypal', "PayPal"),
        ('stripe', "Credit card (via Stripe)"),
        ('other', "Other payment acquirer"),
        ('manual', "Custom payment instructions"),
    ], string="Payment Method", default=lambda self: self._get_default_payment_acquirer_onboarding_value('payment_method'))

    paypal_user_type = fields.Selection([
        ('new_user', "I don't have a Paypal account"),
        ('existing_user', 'I have a Paypal account')], string="Paypal User Type", default='new_user')
    paypal_email_account = fields.Char("Email", default=lambda self: self._get_default_payment_acquirer_onboarding_value('paypal_email_account'))
    paypal_seller_account = fields.Char("Merchant Account ID", default=lambda self: self._get_default_payment_acquirer_onboarding_value('paypal_seller_account'))
    paypal_pdt_token = fields.Char("PDT Identity Token", default=lambda self: self._get_default_payment_acquirer_onboarding_value('paypal_pdt_token'))

    stripe_secret_key = fields.Char(default=lambda self: self._get_default_payment_acquirer_onboarding_value('stripe_secret_key'))
    stripe_publishable_key = fields.Char(default=lambda self: self._get_default_payment_acquirer_onboarding_value('stripe_publishable_key'))

    manual_name = fields.Char("Method",  default=lambda self: self._get_default_payment_acquirer_onboarding_value('manual_name'))
    journal_name = fields.Char("Bank Name", default=lambda self: self._get_default_payment_acquirer_onboarding_value('journal_name'))
    acc_number = fields.Char("Account Number",  default=lambda self: self._get_default_payment_acquirer_onboarding_value('acc_number'))
    manual_post_msg = fields.Html("Payment Instructions")

    @api.onchange('journal_name', 'acc_number')
    def _set_manual_post_msg_value(self):
        self.manual_post_msg = _('<h3>Please make a payment to: </h3><ul><li>Bank: %s</li><li>Account Number: %s</li><li>Account Holder: %s</li></ul>') %\
                               (self.journal_name or _("Bank") , self.acc_number or _("Account"), self.env.company.name)

    _payment_acquirer_onboarding_cache = {}
    _data_fetched = False

    def _get_manual_payment_acquirer(self, env=None):
        if env is None:
            env = self.env
        module_id = env.ref('base.module_payment_transfer').id
        return env['payment.acquirer'].search([('module_id', '=', module_id),
            ('company_id', '=', env.company.id)], limit=1)

    def _get_default_payment_acquirer_onboarding_value(self, key):
        if not self.env.is_admin():
            raise UserError(_("Only administrators can access this data."))

        if self._data_fetched:
            return self._payment_acquirer_onboarding_cache.get(key, '')

        self._data_fetched = True

        self._payment_acquirer_onboarding_cache['payment_method'] = self.env.company.payment_onboarding_payment_method

        installed_modules = self.env['ir.module.module'].sudo().search([
            ('name', 'in', ('payment_paypal', 'payment_stripe')),
            ('state', '=', 'installed'),
        ]).mapped('name')

        if 'payment_paypal' in installed_modules:
            acquirer = self.env.ref('payment.payment_acquirer_paypal')
            self._payment_acquirer_onboarding_cache['paypal_email_account'] = acquirer['paypal_email_account'] or self.env.user.email or ''
            self._payment_acquirer_onboarding_cache['paypal_seller_account'] = acquirer['paypal_seller_account']
            self._payment_acquirer_onboarding_cache['paypal_pdt_token'] = acquirer['paypal_pdt_token']

        if 'payment_stripe' in installed_modules:
            acquirer = self.env.ref('payment.payment_acquirer_stripe')
            self._payment_acquirer_onboarding_cache['stripe_secret_key'] = acquirer['stripe_secret_key']
            self._payment_acquirer_onboarding_cache['stripe_publishable_key'] = acquirer['stripe_publishable_key']

        manual_payment = self._get_manual_payment_acquirer()
        journal = manual_payment.journal_id

        self._payment_acquirer_onboarding_cache['manual_name'] = manual_payment['name']
        self._payment_acquirer_onboarding_cache['manual_post_msg'] = manual_payment['pending_msg']
        self._payment_acquirer_onboarding_cache['journal_name'] = journal.name if journal.name != "Bank" else ""
        self._payment_acquirer_onboarding_cache['acc_number'] = journal.bank_acc_number

        return self._payment_acquirer_onboarding_cache.get(key, '')

    def _install_module(self, module_name):
        module = self.env['ir.module.module'].sudo().search([('name', '=', module_name)])
        if module.state not in ('installed', 'to install', 'to upgrade'):
            module.button_immediate_install()

    def _on_save_payment_acquirer(self):
        self._install_module('account_payment')

    def add_payment_methods(self):
        """ Install required payment acquiers, configure them and mark the
            onboarding step as done."""

        if self.payment_method == 'paypal':
            self._install_module('payment_paypal')

        if self.payment_method == 'stripe':
            self._install_module('payment_stripe')

        if self.payment_method  in ('paypal', 'stripe', 'manual', 'other'):

            self._on_save_payment_acquirer()

            self.env.company.payment_onboarding_payment_method = self.payment_method

            # create a new env including the freshly installed module(s)
            new_env = api.Environment(self.env.cr, self.env.uid, self.env.context)

            if self.payment_method == 'paypal':
                new_env.ref('payment.payment_acquirer_paypal').write({
                    'paypal_email_account': self.paypal_email_account,
                    'paypal_seller_account': self.paypal_seller_account,
                    'paypal_pdt_token': self.paypal_pdt_token,
                    'state': 'enabled',
                })
            if self.payment_method == 'stripe':
                new_env.ref('payment.payment_acquirer_stripe').write({
                    'stripe_secret_key': self.stripe_secret_key,
                    'stripe_publishable_key': self.stripe_publishable_key,
                    'state': 'enabled',
                })
            if self.payment_method == 'manual':
                manual_acquirer = self._get_manual_payment_acquirer(new_env)
                if not manual_acquirer:
                    raise UserError(_(
                        'No manual payment method could be found for this company. ' +
                        'Please create one from the Payment Acquirer menu.'
                    ))
                manual_acquirer.name = self.manual_name
                manual_acquirer.pending_msg = self.manual_post_msg
                manual_acquirer.state = 'enabled'

                journal = manual_acquirer.journal_id
                if journal:
                    journal.name = self.journal_name
                    journal.bank_acc_number = self.acc_number
                else:
                    raise UserError(_("You have to set a journal for your payment acquirer %s." % (self.manual_name,)))

            # delete wizard data immediately to get rid of residual credentials
            self.unlink()
        # the user clicked `apply` and not cancel so we can assume this step is done.
        self._set_payment_acquirer_onboarding_step_done()
        return {'type': 'ir.actions.act_window_close'}

    def _set_payment_acquirer_onboarding_step_done(self):
        self.env.company.sudo().set_onboarding_step_done('payment_acquirer_onboarding_state')

    def action_onboarding_other_payment_acquirer(self):
        self._set_payment_acquirer_onboarding_step_done()
        action = self.env.ref('payment.action_payment_acquirer').read()[0]
        return action

```

## File: wizards\payment_link_wizard.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import hashlib
import hmac

from werkzeug import urls

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError
from odoo.tools import ustr, consteq, float_compare


class PaymentLinkWizard(models.TransientModel):
    _name = "payment.link.wizard"
    _description = "Generate Payment Link"

    @api.model
    def default_get(self, fields):
        res = super(PaymentLinkWizard, self).default_get(fields)
        res_id = self._context.get('active_id')
        res_model = self._context.get('active_model')
        res.update({'res_id': res_id, 'res_model': res_model})
        amount_field = 'amount_residual' if res_model == 'account.move' else 'amount_total'
        if res_id and res_model == 'account.move':
            record = self.env[res_model].browse(res_id)
            res.update({
                'description': record.invoice_payment_ref,
                'amount': record[amount_field],
                'currency_id': record.currency_id.id,
                'partner_id': record.partner_id.id,
                'amount_max': record[amount_field],
            })
        return res

    res_model = fields.Char('Related Document Model', required=True)
    res_id = fields.Integer('Related Document ID', required=True)
    amount = fields.Monetary(currency_field='currency_id', required=True)
    amount_max = fields.Monetary(currency_field='currency_id')
    currency_id = fields.Many2one('res.currency')
    partner_id = fields.Many2one('res.partner')
    partner_email = fields.Char(related='partner_id.email')
    link = fields.Char(string='Payment Link', compute='_compute_values')
    description = fields.Char('Payment Ref')
    access_token = fields.Char(compute='_compute_values')
    company_id = fields.Many2one('res.company', compute='_compute_company')

    @api.onchange('amount', 'description')
    def _onchange_amount(self):
        if float_compare(self.amount_max, self.amount, precision_rounding=self.currency_id.rounding or 0.01) == -1:
            raise ValidationError(_("Please set an amount smaller than %s.") % (self.amount_max))
        if self.amount <= 0:
            raise ValidationError(_("The value of the payment amount must be positive."))

    @api.depends('amount', 'description', 'partner_id', 'currency_id')
    def _compute_values(self):
        secret = self.env['ir.config_parameter'].sudo().get_param('database.secret')
        for payment_link in self:
            token_str = '%s%s%s' % (payment_link.partner_id.id, payment_link.amount, payment_link.currency_id.id)
            payment_link.access_token = hmac.new(secret.encode('utf-8'), token_str.encode('utf-8'), hashlib.sha256).hexdigest()
        # must be called after token generation, obvsly - the link needs an up-to-date token
        self._generate_link()

    @api.depends('res_model', 'res_id')
    def _compute_company(self):
        for link in self:
            record = self.env[link.res_model].browse(link.res_id)
            link.company_id = record.company_id if 'company_id' in record else False

    def _generate_link(self):
        for payment_link in self:
            record = self.env[payment_link.res_model].browse(payment_link.res_id)
            link = ('%s/website_payment/pay?reference=%s&amount=%s&currency_id=%s'
                    '&partner_id=%s&access_token=%s') % (
                        record.get_base_url(),
                        urls.url_quote_plus(payment_link.description),
                        payment_link.amount,
                        payment_link.currency_id.id,
                        payment_link.partner_id.id,
                        payment_link.access_token
                    )
            if payment_link.company_id:
                link += '&company_id=%s' % payment_link.company_id.id
            if payment_link.res_model == 'account.move':
                link += '&invoice_id=%s' % payment_link.res_id
            payment_link.link = link

    @api.model
    def check_token(self, access_token, partner_id, amount, currency_id):
        secret = self.env['ir.config_parameter'].sudo().get_param('database.secret')
        token_str = '%s%s%s' % (partner_id, amount, currency_id)
        correct_token = hmac.new(secret.encode('utf-8'), token_str.encode('utf-8'), hashlib.sha256).hexdigest()
        if consteq(ustr(access_token), correct_token):
            return True
        return False
```

## File: wizards\payment_link_wizard_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="payment_link_wizard_view_form" model="ir.ui.view">
        <field name="name">payment.link.wizard.form</field>
        <field name="model">payment.link.wizard</field>
        <field name="arch" type="xml">
            <form string="Generate Payment Link">
                <group>
                    <field name="res_id" invisible="1"/>
                    <field name="res_model" invisible="1"/>
                    <field name="partner_id" invisible="1"/>
                    <field name="partner_email" invisible="1"/>
                    <field name="amount_max" invisible="1"/>
                    <field name="description"/>
                    <field name="amount"/>
                    <field name="currency_id" invisible="1"/>
                    <field name="access_token" invisible="1"/>
                </group>
                <group>
                    <field name="link" readonly="1" widget="CopyClipboardChar"/>
                </group>
                <group attrs="{'invisible':[('partner_email', '!=', False)]}">
                    <p class="alert alert-warning font-weight-bold" role="alert">This partner has no email, which may cause issues with some payment acquirers. Setting an email for this partner is advised.</p>
                </group>
                <footer>
                    <button string="Close" class="btn-primary" special="cancel"/>
                </footer>
            </form>
        </field>
    </record>

    <act_window
        name="Generate a Payment Link"
        res_model="payment.link.wizard"
        binding_model="account.move"
        binding_views="form"
        view_mode="form"
        target="new"
        view_id="payment_link_wizard_view_form"
        id="action_invoice_order_generate_link"
    />
</odoo>

```

## File: wizards\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import payment_acquirer_onboarding_wizard
from . import payment_link_wizard

```

