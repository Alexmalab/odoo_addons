# Odoo Module: payment_stripe

Category: Accounting/Payment Acquirers

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import controllers
from odoo.addons.payment.models.payment_acquirer import create_missing_journal_for_acquirers
from odoo.addons.payment import reset_payment_provider

def uninstall_hook(cr, registry):
    reset_payment_provider(cr, registry, 'stripe')

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-

{
    'name': 'Stripe Payment Acquirer',
    'category': 'Accounting/Payment Acquirers',
    'sequence': 380,
    'summary': 'Payment Acquirer: Stripe Implementation',
    'version': '1.0',
    'description': """Stripe Payment Acquirer""",
    'depends': ['payment'],
    'data': [
        'views/payment_views.xml',
        'views/payment_stripe_templates.xml',
        'data/payment_acquirer_data.xml',
    ],
    'images': ['static/description/icon.png'],
    'installable': True,
    'application': True,
    'post_init_hook': 'create_missing_journal_for_acquirers',
    'uninstall_hook': 'uninstall_hook',
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
import json
import logging
import pprint
import werkzeug

from odoo import http
from odoo.http import request

_logger = logging.getLogger(__name__)


class StripeController(http.Controller):
    _success_url = '/payment/stripe/success'
    _cancel_url = '/payment/stripe/cancel'

    @http.route(['/payment/stripe/success', '/payment/stripe/cancel'], type='http', auth='public')
    def stripe_success(self, **kwargs):
        request.env['payment.transaction'].sudo().form_feedback(kwargs, 'stripe')
        return werkzeug.utils.redirect('/payment/process')

    @http.route(['/payment/stripe/s2s/create_json_3ds'], type='json', auth='public', csrf=False)
    def stripe_s2s_create_json_3ds(self, verify_validity=False, **kwargs):
        if not kwargs.get('partner_id'):
            kwargs = dict(kwargs, partner_id=request.env.user.partner_id.id)
        token = request.env['payment.acquirer'].browse(int(kwargs.get('acquirer_id'))).with_context(stripe_manual_payment=True).s2s_process(kwargs)

        if not token:
            res = {
                'result': False,
            }
            return res

        res = {
            'result': True,
            'id': token.id,
            'short_name': token.short_name,
            '3d_secure': False,
            'verified': False,
        }

        if verify_validity != False:
            token.validate()
            res['verified'] = token.verified

        return res

    @http.route('/payment/stripe/s2s/create_setup_intent', type='json', auth='public', csrf=False)
    def stripe_s2s_create_setup_intent(self, acquirer_id, **kwargs):
        acquirer = request.env['payment.acquirer'].browse(int(acquirer_id))
        res = acquirer.with_context(stripe_manual_payment=True)._create_setup_intent(kwargs)
        return res.get('client_secret')

    @http.route('/payment/stripe/s2s/process_payment_intent', type='json', auth='public', csrf=False)
    def stripe_s2s_process_payment_intent(self, **post):
        return request.env['payment.transaction'].sudo().form_feedback(post, 'stripe')

    @http.route('/payment/stripe/s2s/process_payment_error', type='json', auth='public', csrf=False)
    def stripe_s2s_process_payment_error(self, **post):
        transaction_sudo = request.env['payment.transaction'].sudo().search([('reference', '=', post['reference']),
                                                                            ('provider', '=', 'stripe'),
                                                                            ('stripe_payment_intent_secret', '=', post['stripe_payment_intent_secret'])])
        transaction_sudo.write({'state': 'error', 'state_message': post['error']})

    @http.route('/payment/stripe/webhook', type='json', auth='public', csrf=False)
    def stripe_webhook(self, **kwargs):
        data = json.loads(request.httprequest.data)
        request.env['payment.acquirer'].sudo()._handle_stripe_webhook(data)
        return 'OK'
```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-

from . import main

```

## File: data\payment_acquirer_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="payment.payment_acquirer_stripe" model="payment.acquirer">
            <field name="name">Stripe</field>
            <field name="image_128" type="base64" file="payment_stripe/static/src/img/stripe_icon.png"/>
            <field name="provider">stripe</field>
            <field name="company_id" ref="base.main_company"/>
            <field name="view_template_id" ref="stripe_form"/>
            <field name="registration_view_template_id" ref="stripe_s2s_form"/>
        </record>
    </data>
</odoo>

```

## File: models\payment.py

```python
# coding: utf-8

from collections import namedtuple
from datetime import datetime
from hashlib import sha1, sha256
import hmac
import json
import logging
import requests
import pprint
from requests.exceptions import HTTPError
from werkzeug import urls

from odoo import api, fields, models, _
from odoo.http import request
from odoo.tools.float_utils import float_round
from odoo.tools import consteq
from odoo.exceptions import ValidationError

from odoo.addons.payment_stripe.controllers.main import StripeController

_logger = logging.getLogger(__name__)

# The following currencies are integer only, see https://stripe.com/docs/currencies#zero-decimal
INT_CURRENCIES = [
    u'BIF', u'XAF', u'XPF', u'CLP', u'KMF', u'DJF', u'GNF', u'JPY', u'MGA', u'PYG', u'RWF', u'KRW',
    u'VUV', u'VND', u'XOF'
]
STRIPE_SIGNATURE_AGE_TOLERANCE = 600  # in seconds


class PaymentAcquirerStripe(models.Model):
    _inherit = 'payment.acquirer'

    provider = fields.Selection(selection_add=[
        ('stripe', 'Stripe')
    ], ondelete={'stripe': 'set default'})
    stripe_secret_key = fields.Char(required_if_provider='stripe', groups='base.group_user')
    stripe_publishable_key = fields.Char(required_if_provider='stripe', groups='base.group_user')
    stripe_webhook_secret = fields.Char(
        string='Stripe Webhook Secret', groups='base.group_user',
        help="If you enable webhooks, this secret is used to verify the electronic "
             "signature of events sent by Stripe to Odoo. Failing to set this field in Odoo "
             "will disable the webhook system for this acquirer entirely.")
    stripe_image_url = fields.Char(
        "Checkout Image URL", groups='base.group_user',
        help="A relative or absolute URL pointing to a square image of your "
             "brand or product. As defined in your Stripe profile. See: "
             "https://stripe.com/docs/checkout")

    def stripe_form_generate_values(self, tx_values):
        self.ensure_one()

        base_url = self.get_base_url()
        stripe_session_data = {
            'line_items[][amount]': int(tx_values['amount'] if tx_values['currency'].name in INT_CURRENCIES else float_round(tx_values['amount'] * 100, 2)),
            'line_items[][currency]': tx_values['currency'].name,
            'line_items[][quantity]': 1,
            'line_items[][name]': tx_values['reference'],
            'client_reference_id': tx_values['reference'],
            'success_url': urls.url_join(base_url, StripeController._success_url) + '?reference=%s' % urls.url_quote_plus(tx_values['reference']),
            'cancel_url': urls.url_join(base_url, StripeController._cancel_url) + '?reference=%s' % urls.url_quote_plus(tx_values['reference']),
            'payment_intent_data[description]': tx_values['reference'],
            'customer_email': tx_values.get('partner_email') or tx_values.get('billing_partner_email') or None,
        }
        if tx_values['type'] == 'form_save':
            stripe_session_data['payment_intent_data[setup_future_usage]'] = 'off_session'

        self._add_available_payment_method_types(stripe_session_data, tx_values)

        tx_values['session_id'] = self.with_context(stripe_manual_payment=True)._create_stripe_session(stripe_session_data)

        return tx_values

    @api.model
    def _add_available_payment_method_types(self, stripe_session_data, tx_values):
        """
        Add payment methods available for the given transaction

        :param stripe_session_data: dictionary to add the payment method types to
        :param tx_values: values of the transaction to consider the payment method types for
        """
        PMT = namedtuple('PaymentMethodType', ['name', 'countries', 'currencies', 'recurrence'])
        all_payment_method_types = [
            PMT('card', [], [], 'recurring'),
            PMT('ideal', ['nl'], ['eur'], 'punctual'),
            PMT('bancontact', ['be'], ['eur'], 'punctual'),
            PMT('eps', ['at'], ['eur'], 'punctual'),
            PMT('giropay', ['de'], ['eur'], 'punctual'),
            PMT('p24', ['pl'], ['eur', 'pln'], 'punctual'),
        ]

        existing_icons = [(icon.name or '').lower() for icon in self.env['payment.icon'].search([])]
        linked_icons = [(icon.name or '').lower() for icon in self.payment_icon_ids]

        # We don't filter out pmt in the case the icon doesn't exist at all as it would be **implicit** exclusion
        icon_filtered = filter(lambda pmt: pmt.name == 'card' or
                                           pmt.name in linked_icons or
                                           pmt.name not in existing_icons, all_payment_method_types)
        country = (tx_values['billing_partner_country'].code or 'no_country').lower()
        pmt_country_filtered = filter(lambda pmt: not pmt.countries or country in pmt.countries, icon_filtered)
        currency = (tx_values.get('currency').name or 'no_currency').lower()
        pmt_currency_filtered = filter(lambda pmt: not pmt.currencies or currency in pmt.currencies, pmt_country_filtered)
        pmt_recurrence_filtered = filter(lambda pmt: tx_values.get('type') != 'form_save' or pmt.recurrence == 'recurring',
                                    pmt_currency_filtered)

        available_payment_method_types = map(lambda pmt: pmt.name, pmt_recurrence_filtered)

        for idx, payment_method_type in enumerate(available_payment_method_types):
            stripe_session_data[f'payment_method_types[{idx}]'] = payment_method_type

    def _stripe_request(self, url, data=False, method='POST', idempotency_key=None):
        self.ensure_one()
        url = urls.url_join(self._get_stripe_api_url(), url)
        headers = {
            'AUTHORIZATION': 'Bearer %s' % self.sudo().stripe_secret_key,
            'Stripe-Version': '2019-05-16',  # SetupIntent need a specific version
        }
        if method == 'POST' and idempotency_key:
            headers['Idempotency-Key'] = idempotency_key
        resp = requests.request(method, url, data=data, headers=headers)
        # Stripe can send 4XX errors for payment failure (not badly-formed requests)
        # check if error `code` is present in 4XX response and raise only if not
        # cfr https://stripe.com/docs/error-codes
        # these can be made customer-facing, as they usually indicate a problem with the payment
        # (e.g. insufficient funds, expired card, etc.)
        # if the context key `stripe_manual_payment` is set then these errors will be raised as ValidationError,
        # otherwise, they will be silenced, and the will be returned no matter the status.
        # This key should typically be set for payments in the present and unset for automated payments
        # (e.g. through crons)
        if not resp.ok and self._context.get('stripe_manual_payment') and (400 <= resp.status_code < 500 and resp.json().get('error', {}).get('code')):
            try:
                resp.raise_for_status()
            except HTTPError:
                _logger.error(resp.text)
                stripe_error = resp.json().get('error', {}).get('message', '')
                error_msg = " " + (_("Stripe gave us the following info about the problem: '%s'", stripe_error))
                raise ValidationError(error_msg)
        return resp.json()

    def _create_stripe_session(self, kwargs):
        self.ensure_one()
        resp = self._stripe_request('checkout/sessions', kwargs)
        if resp.get('payment_intent') and kwargs.get('client_reference_id'):
            tx = self.env['payment.transaction'].sudo().search([('reference', '=', kwargs['client_reference_id'])])
            tx.stripe_payment_intent = resp['payment_intent']
        if 'id' not in resp and 'error' in resp:
            _logger.error(resp['error']['message'])
        return resp['id']

    def _create_setup_intent(self, kwargs):
        self.ensure_one()
        params = {
            'usage': 'off_session',
        }
        _logger.info('_stripe_create_setup_intent: Sending values to stripe, values:\n%s', pprint.pformat(params))

        res = self._stripe_request('setup_intents', params)

        _logger.info('_stripe_create_setup_intent: Values received:\n%s', pprint.pformat(res))
        return res

    @api.model
    def _get_stripe_api_url(self):
        return 'https://api.stripe.com/v1/'

    @api.model
    def stripe_s2s_form_process(self, data):
        if 'card' in data and not data.get('card'):
            # coming back from a checkout payment and iDeal (or another non-card pm)
            # can't save the token if it's not a card
            # note that in the case of a s2s payment, 'card' wont be
            # in the data dict because we need to fetch it from the stripe server
            _logger.info('unable to save card info from Stripe since the payment was not done with a card')
            return self.env['payment.token']
        last4 = data.get('card', {}).get('last4')
        if not last4:
            # PM was created with a setup intent, need to get last4 digits through
            # yet another call -_-
            acquirer_id = self.env['payment.acquirer'].browse(int(data['acquirer_id']))
            pm = data.get('payment_method')
            res = acquirer_id._stripe_request('payment_methods/%s' % pm, data=False, method='GET')
            last4 = res.get('card', {}).get('last4', '****')

        payment_token = self.env['payment.token'].sudo().create({
            'acquirer_id': int(data['acquirer_id']),
            'partner_id': int(data['partner_id']),
            'stripe_payment_method': data.get('payment_method'),
            'name': 'XXXXXXXXXXXX%s' % last4,
            'acquirer_ref': data.get('customer')
        })
        return payment_token

    def _get_feature_support(self):
        """Get advanced feature support by provider.

        Each provider should add its technical in the corresponding
        key for the following features:
            * tokenize: support saving payment data in a payment.tokenize
                        object
        """
        res = super(PaymentAcquirerStripe, self)._get_feature_support()
        res['tokenize'].append('stripe')
        return res

    def _handle_stripe_webhook(self, data):
        """Process a webhook payload from Stripe.

        Post-process a webhook payload to act upon the matching payment.transaction
        record in Odoo.
        """
        wh_type = data.get('type')
        if wh_type != 'checkout.session.completed':
            _logger.info('unsupported webhook type %s, ignored', wh_type)
            return False

        _logger.info('handling %s webhook event from stripe', wh_type)

        stripe_object = data.get('data', {}).get('object')
        if not stripe_object:
            raise ValidationError('Stripe Webhook data does not conform to the expected API.')
        if wh_type == 'checkout.session.completed':
            return self._handle_checkout_webhook(stripe_object)
        return False

    def _verify_stripe_signature(self):
        """
        :return: true if and only if signature matches hash of payload calculated with secret
        :raises ValidationError: if signature doesn't match
        """
        if not self.stripe_webhook_secret:
            raise ValidationError('webhook event received but webhook secret is not configured')
        signature = request.httprequest.headers.get('Stripe-Signature')
        body = request.httprequest.data

        sign_data = {k: v for (k, v) in [s.split('=') for s in signature.split(',')]}
        event_timestamp = int(sign_data['t'])
        if datetime.utcnow().timestamp() - event_timestamp > STRIPE_SIGNATURE_AGE_TOLERANCE:
            _logger.error('stripe event is too old, event is discarded')
            raise ValidationError('event timestamp older than tolerance')

        signed_payload = "%s.%s" % (event_timestamp, body.decode('utf-8'))

        actual_signature = sign_data['v1']
        expected_signature = hmac.new(self.stripe_webhook_secret.encode('utf-8'),
                                      signed_payload.encode('utf-8'),
                                      sha256).hexdigest()

        if not consteq(expected_signature, actual_signature):
            _logger.error(
                'incorrect webhook signature from Stripe, check if the webhook signature '
                'in Odoo matches to one in the Stripe dashboard')
            raise ValidationError('incorrect webhook signature')

        return True

    def _handle_checkout_webhook(self, checkout_object: dir):
        """
        Process a checkout.session.completed Stripe web hook event,
        mark related payment successful

        :param checkout_object: provided in the request body
        :return: True if and only if handling went well, False otherwise
        :raises ValidationError: if input isn't usable
        """
        tx_reference = checkout_object.get('client_reference_id')
        data = {'reference': tx_reference}
        try:
            odoo_tx = self.env['payment.transaction']._stripe_form_get_tx_from_data(data)
        except ValidationError as e:
            _logger.info('Received notification for tx %s. Skipped it because of %s', tx_reference, e)
            return False

        PaymentAcquirerStripe._verify_stripe_signature(odoo_tx.acquirer_id)

        url = 'payment_intents/%s' % odoo_tx.stripe_payment_intent
        stripe_tx = odoo_tx.acquirer_id._stripe_request(url)

        if 'error' in stripe_tx:
            error = stripe_tx['error']
            raise ValidationError("Could not fetch Stripe payment intent related to %s because of %s; see %s" % (
                odoo_tx, error['message'], error['doc_url']))

        if stripe_tx.get('charges') and stripe_tx.get('charges').get('total_count'):
            charge = stripe_tx.get('charges').get('data')[0]
            data.update(charge)

        return odoo_tx.form_feedback(data, 'stripe')


class PaymentTransactionStripe(models.Model):
    _inherit = 'payment.transaction'

    stripe_payment_intent = fields.Char(string='Stripe Payment Intent ID', readonly=True)
    stripe_payment_intent_secret = fields.Char(string='Stripe Payment Intent Secret', readonly=True)

    def _get_processing_info(self):
        res = super()._get_processing_info()
        if self.acquirer_id.provider == 'stripe':
            stripe_info = {
                'stripe_payment_intent': self.stripe_payment_intent,
                'stripe_payment_intent_secret': self.stripe_payment_intent_secret,
                'stripe_publishable_key': self.acquirer_id.stripe_publishable_key,
            }
            res.update(stripe_info)
        return res

    def form_feedback(self, data, acquirer_name):
        if data.get('reference') and acquirer_name == 'stripe':
            transaction = self.env['payment.transaction'].search([('reference', '=', data['reference'])])

            url = 'payment_intents/%s' % transaction.stripe_payment_intent
            resp = transaction.acquirer_id._stripe_request(url)
            if resp.get('charges') and resp.get('charges').get('total_count'):
                resp = resp.get('charges').get('data')[0]

            data.update(resp)
            _logger.info('Stripe: entering form_feedback with post data %s' % pprint.pformat(data))
        return super(PaymentTransactionStripe, self).form_feedback(data, acquirer_name)

    def _stripe_create_payment_intent(self, acquirer_ref=None, email=None):
        if not self.payment_token_id.stripe_payment_method:
            # old token before using sca, need to fetch data from the api
            self.payment_token_id._stripe_sca_migrate_customer()

        charge_params = {
            'amount': int(self.amount if self.currency_id.name in INT_CURRENCIES else float_round(self.amount * 100, 2)),
            'currency': self.currency_id.name.lower(),
            'off_session': True,
            'confirm': True,
            'payment_method': self.payment_token_id.stripe_payment_method,
            'customer': self.payment_token_id.acquirer_ref,
            "description": self.reference,
        }
        if not self.env.context.get('off_session'):
            charge_params.update(setup_future_usage='off_session', off_session=False)
        _logger.info('_stripe_create_payment_intent: Sending values to stripe, values:\n%s', pprint.pformat(charge_params))
        # Create an idempotency key using the hash of the transaction reference and the database UUID
        database_uuid = self.env['ir.config_parameter'].sudo().get_param('database.uuid')
        idempotency_key = sha1((database_uuid + self.reference).encode("utf-8")).hexdigest()
        res = self.acquirer_id._stripe_request('payment_intents', charge_params, idempotency_key=idempotency_key)
        if res.get('charges') and res.get('charges').get('total_count'):
            res = res.get('charges').get('data')[0]

        _logger.info('_stripe_create_payment_intent: Values received:\n%s', pprint.pformat(res))
        return res

    def stripe_s2s_do_transaction(self, **kwargs):
        self.ensure_one()
        result = self._stripe_create_payment_intent(acquirer_ref=self.payment_token_id.acquirer_ref, email=self.partner_email)
        return self._stripe_s2s_validate_tree(result)

    def _create_stripe_refund(self):

        refund_params = {
            'charge': self.acquirer_reference,
            'amount': int(float_round(self.amount * 100, 2)), # by default, stripe refund the full amount (we don't really need to specify the value)
            'metadata[reference]': self.reference,
        }

        _logger.info('_create_stripe_refund: Sending values to stripe URL, values:\n%s', pprint.pformat(refund_params))
        # Create an idempotency key using the hash of the transaction reference and the database UUID
        database_uuid = self.env['ir.config_parameter'].sudo().get_param('database.uuid')
        idempotency_key = sha1((database_uuid + self.reference + 'refunds').encode("utf-8")).hexdigest()
        res = self.acquirer_id._stripe_request('refunds', refund_params, idempotency_key=idempotency_key)
        _logger.info('_create_stripe_refund: Values received:\n%s', pprint.pformat(res))

        return res

    def stripe_s2s_do_refund(self, **kwargs):
        self.ensure_one()
        result = self._create_stripe_refund()
        return self._stripe_s2s_validate_tree(result)

    @api.model
    def _stripe_form_get_tx_from_data(self, data):
        """ Given a data dict coming from stripe, verify it and find the related
        transaction record. """
        reference = data.get('reference')
        if not reference:
            stripe_error = data.get('error', {}).get('message', '')
            _logger.error('Stripe: invalid reply received from stripe API, looks like '
                          'the transaction failed. (error: %s)', stripe_error or 'n/a')
            error_msg = _("We're sorry to report that the transaction has failed.")
            if stripe_error:
                error_msg += " " + (_("Stripe gave us the following info about the problem: '%s'") %
                                    stripe_error)
            error_msg += " " + _("Perhaps the problem can be solved by double-checking your "
                                 "credit card details, or contacting your bank?")
            raise ValidationError(error_msg)

        tx = self.search([('reference', '=', reference)])
        if not tx:
            error_msg = _('Stripe: no order found for reference %s', reference)
            _logger.error(error_msg)
            raise ValidationError(error_msg)
        elif len(tx) > 1:
            error_msg = _('Stripe: %(count)s orders found for reference %(reference)s', count=len(tx), reference=reference)
            _logger.error(error_msg)
            raise ValidationError(error_msg)
        return tx[0]

    def _stripe_s2s_validate_tree(self, tree):
        self.ensure_one()
        if self.state not in ("draft", "pending"):
            _logger.info('Stripe: trying to validate an already validated tx (ref %s)', self.reference)
            return True

        status = tree.get('status')
        tx_id = tree.get('id')
        tx_secret = tree.get("client_secret")
        pi_id = tree.get('payment_intent')
        vals = {
            "date": fields.datetime.now(),
            "acquirer_reference": tx_id,
            "stripe_payment_intent": pi_id or tx_id,
            "stripe_payment_intent_secret": tx_secret
        }
        if status == 'succeeded':
            self.write(vals)
            self._set_transaction_done()
            self.execute_callback()
            if self.type == 'form_save':
                s2s_data = {
                    'customer': tree.get('customer'),
                    'payment_method': tree.get('payment_method'),
                    'card': tree.get('payment_method_details').get('card'),
                    'acquirer_id': self.acquirer_id.id,
                    'partner_id': self.partner_id.id
                }
                token = self.acquirer_id.stripe_s2s_form_process(s2s_data)
                self.payment_token_id = token.id
            if self.payment_token_id:
                self.payment_token_id.verified = True
            return True
        if status in ('processing', 'requires_action'):
            self.write(vals)
            self._set_transaction_pending()
            return True
        if status == 'requires_payment_method':
            self._set_transaction_cancel()
            self.acquirer_id._stripe_request('payment_intents/%s/cancel' % self.stripe_payment_intent)
            return False
        else:
            error = tree.get("failure_message") or tree.get('error', {}).get('message')
            self._set_transaction_error(error)
            return False

    def _stripe_form_get_invalid_parameters(self, data):
        invalid_parameters = []
        if data.get('amount') != int(self.amount if self.currency_id.name in INT_CURRENCIES else float_round(self.amount * 100, 2)):
            invalid_parameters.append(('Amount', data.get('amount'), self.amount * 100))
        if data.get('currency') and data.get('currency').upper() != self.currency_id.name:
            invalid_parameters.append(('Currency', data.get('currency'), self.currency_id.name))
        if data.get('payment_intent') and data.get('payment_intent') != self.stripe_payment_intent:
            invalid_parameters.append(('Payment Intent', data.get('payment_intent'), self.stripe_payment_intent))
        return invalid_parameters

    def _stripe_form_validate(self, data):
        return self._stripe_s2s_validate_tree(data)


class PaymentTokenStripe(models.Model):
    _inherit = 'payment.token'

    stripe_payment_method = fields.Char('Payment Method ID')

    @api.model
    def stripe_create(self, values):
        if values.get('stripe_payment_method') and not values.get('acquirer_ref'):
            partner_id = self.env['res.partner'].browse(values.get('partner_id'))
            payment_acquirer = self.env['payment.acquirer'].browse(values.get('acquirer_id'))

            # create customer to stipe
            customer_data = {
                'email': partner_id.email
            }
            cust_resp = payment_acquirer._stripe_request('customers', customer_data)

            # link customer with payment method
            api_url_payment_method = 'payment_methods/%s/attach' % values['stripe_payment_method']
            method_data = {
                'customer': cust_resp.get('id')
            }
            payment_acquirer._stripe_request(api_url_payment_method, method_data)
            return {
                'acquirer_ref': cust_resp['id'],
            }
        return values

    def _stripe_sca_migrate_customer(self):
        """Migrate a token from the old implementation of Stripe to the SCA one.

        In the old implementation, it was possible to create a valid charge just by
        giving the customer ref to ask Stripe to use the default source (= default
        card). Since we have a one-to-one matching between a saved card, this used to
        work well - but now we need to specify the payment method for each call and so
        we have to contact stripe to get the default source for the customer and save it
        in the payment token.
        This conversion will happen once per token, the first time it gets used following
        the installation of the module."""
        self.ensure_one()
        url = "customers/%s" % (self.acquirer_ref)
        data = self.acquirer_id._stripe_request(url, method="GET")
        sources = data.get('sources', {}).get('data', [])
        pm_ref = False
        if sources:
            if len(sources) > 1:
                _logger.warning('stripe sca customer conversion: there should be a single saved source per customer!')
            pm_ref = sources[0].get('id')
        else:
            url = 'payment_methods'
            params = {
                'type': 'card',
                'customer': self.acquirer_ref,
            }
            payment_methods = self.acquirer_id._stripe_request(url, params, method='GET')
            cards = payment_methods.get('data', [])
            if len(cards) > 1:
                _logger.warning('stripe sca customer conversion: there should be a single saved source per customer!')
            pm_ref = cards and cards[0].get('id')
        if not pm_ref:
            raise ValidationError(_('Unable to convert Stripe customer for SCA compatibility. Is there at least one card for this customer in the Stripe backend?'))
        self.stripe_payment_method = pm_ref
        _logger.info('converted old customer ref to sca-compatible record for payment token %s', self.id)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import payment

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#94B6C8"/><stop offset="100%" stop-color="#6A9EBA"/></linearGradient><path id="d" d="M19.25 46h2.714v1.212c0 .188.152.343.339.343h26.394a.342.342 0 0 0 .339-.343V37.5H32.5c.333-3.333-.165-5.5-1.494-6.5h18.03v-3.212a.342.342 0 0 0-.339-.343H29.765c.361-1.01.57-1.924.627-2.742h18.644c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742V46zm-3.79 9.642h-1.037V55h2.833v.642h-1.037v2.832h-.76v-2.832zM17.422 55h1.07l.809 2.389h.01L20.075 55h1.07v3.474h-.711v-2.462h-.01l-.847 2.462h-.586l-.848-2.438h-.01v2.438h-.711V55zm5.66-27.992c4.256 1.523 6.896 3.327 6.896 7.649 0 2.612-.901 4.633-2.64 6.001-1.553 1.244-3.852 1.897-6.616 1.897-3.48 0-6.834-1.057-8.635-2.084l.932-5.814c2.112 1.244 5.342 2.207 7.299 2.207 1.584 0 2.454-.59 2.454-1.616 0-1.058-.901-1.742-3.603-2.706-4.193-1.523-6.771-3.327-6.771-7.556 0-2.332.838-4.26 2.453-5.597 1.553-1.275 3.728-1.959 6.337-1.959 3.696 0 6.367 1.027 7.671 1.649l-.931 5.752c-1.647-.808-4.038-1.71-6.368-1.71-1.273 0-1.988.497-1.988 1.368 0 1.026 1.243 1.68 3.51 2.519z"/><path id="e" d="M19.25 44h2.714v1.212c0 .188.152.343.339.343h26.394a.342.342 0 0 0 .339-.343V35.5H32.5c.333-3.333-.165-5.5-1.494-6.5h18.03v-3.212a.342.342 0 0 0-.339-.343H29.765c.361-1.01.57-1.924.627-2.742h18.644c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742V44zm-3.79 9.642h-1.037V53h2.833v.642h-1.037v2.832h-.76v-2.832zM17.422 53h1.07l.809 2.389h.01L20.075 53h1.07v3.474h-.711v-2.462h-.01l-.847 2.462h-.586l-.848-2.438h-.01v2.438h-.711V53zm5.66-27.992c4.256 1.523 6.896 3.327 6.896 7.649 0 2.612-.901 4.633-2.64 6.001-1.553 1.244-3.852 1.897-6.616 1.897-3.48 0-6.834-1.057-8.635-2.084l.932-5.814c2.112 1.244 5.342 2.207 7.299 2.207 1.584 0 2.454-.59 2.454-1.616 0-1.058-.901-1.742-3.603-2.706-4.193-1.523-6.771-3.327-6.771-7.556 0-2.332.838-4.26 2.453-5.597 1.553-1.275 3.728-1.959 6.337-1.959 3.696 0 6.367 1.027 7.671 1.649l-.931 5.752c-1.647-.808-4.038-1.71-6.368-1.71-1.273 0-1.988.497-1.988 1.368 0 1.026 1.243 1.68 3.51 2.519z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M4 69c-2 0-4-1-4-4V33.916L14 17.5l13.818 5.203h9.432L48 23l3.576 1.968-.236 22.01L39.224 69H4z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: static\src\js\payment_form.js

```javascript
odoo.define('payment_stripe.payment_form', function (require) {
"use strict";

var ajax = require('web.ajax');
var core = require('web.core');
var Dialog = require('web.Dialog');
var PaymentForm = require('payment.payment_form');

var qweb = core.qweb;
var _t = core._t;

ajax.loadXML('/payment_stripe/static/src/xml/stripe_templates.xml', qweb);

PaymentForm.include({

    willStart: function () {
        return this._super.apply(this, arguments).then(function () {
            return ajax.loadJS("https://js.stripe.com/v3/");
        })
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

     /**
     * called to create payment method object for credit card/debit card.
     *
     * @private
     * @param {Object} stripe
     * @param {Object} formData
     * @param {Object} card
     * @param {Boolean} addPmEvent
     * @returns {Promise}
     */
    _createPaymentMethod: function (stripe, formData, card, addPmEvent) {
        if (addPmEvent) {
            return this._rpc({
                route: '/payment/stripe/s2s/create_setup_intent',
                params: {'acquirer_id': formData.acquirer_id}
            }).then(function(intent_secret) {
                return stripe.handleCardSetup(intent_secret, card);
            });
        } else {
            return stripe.createPaymentMethod({
                type: 'card',
                card: card,
            });
        }
    },

    /**
     * called when clicking on pay now or add payment event to create token for credit card/debit card.
     *
     * @private
     * @param {Event} ev
     * @param {DOMElement} checkedRadio
     * @param {Boolean} addPmEvent
     */
    _createStripeToken: function (ev, $checkedRadio, addPmEvent) {
        var self = this;
        if (ev.type === 'submit') {
            var button = $(ev.target).find('*[type="submit"]')[0]
        } else {
            var button = ev.target;
        }
        this.disableButton(button);
        var acquirerID = this.getAcquirerIdFromRadio($checkedRadio);
        var acquirerForm = this.$('#o_payment_add_token_acq_' + acquirerID);
        var inputsForm = $('input', acquirerForm);
        if (this.options.partnerId === undefined) {
            console.warn('payment_form: unset partner_id when adding new token; things could go wrong');
        }

        var formData = self.getFormData(inputsForm);
        var stripe = this.stripe;
        var card = this.stripe_card_element;
        if (card._invalid) {
            // if we don't enable the button again, at this point the button is displaying the 'loading' animation
            // and since we break the workflow here it gives the impression that something is happening, but it isn't
            self.enableButton(button);
            return;
        }
        this._createPaymentMethod(stripe, formData, card, addPmEvent).then(function(result) {
            if (result.error) {
                return Promise.reject({"message": {"data": { "arguments": [result.error.message]}}});
            } else {
                const paymentMethod = addPmEvent ? result.setupIntent.payment_method : result.paymentMethod.id;
                _.extend(formData, {"payment_method": paymentMethod});
                return self._rpc({
                    route: formData.data_set,
                    params: formData,
                });
            }
        }).then(function(result) {
            if (addPmEvent) {
                if (formData.return_url) {
                    window.location = formData.return_url;
                } else {
                    window.location.reload();
                }
            } else {
                $checkedRadio.val(result.id);
                self.el.submit();
            }
        }).guardedCatch(function (error) {
            // We don't want to open the Error dialog since
            // we already have a container displaying the error
            if (error.event) {
                error.event.preventDefault();
            }
            // if the rpc fails, pretty obvious
            self.enableButton(button);
            self.displayError(
                _t('Unable to save card'),
                _t("We are not able to add your payment method at the moment. ") +
                    self._parseError(error)
            );
        });
    },
    /**
     * called when clicking a Stripe radio if configured for s2s flow; instanciates the card and bind it to the widget.
     *
     * @private
     * @param {DOMElement} checkedRadio
     */
    _bindStripeCard: function ($checkedRadio) {
        var acquirerID = this.getAcquirerIdFromRadio($checkedRadio);
        var acquirerForm = this.$('#o_payment_add_token_acq_' + acquirerID);
        var inputsForm = $('input', acquirerForm);
        var formData = this.getFormData(inputsForm);
        var stripe = Stripe(formData.stripe_publishable_key);
        var element = stripe.elements();
        var card = element.create('card', {hidePostalCode: true});
        // use more specific css selector so that '#card-element' is found inside the selected stripe card, otherwise
        // this won't happen, and the card will be mounted to the first element found.
        card.mount(`#o_payment_add_token_acq_${acquirerID} #card-element`);
        card.on('ready', function(ev) {
            card.focus();
        });
        card.addEventListener('change', function (event) {
            var displayError = document.getElementById('card-errors');
            displayError.textContent = '';
            if (event.error) {
                displayError.textContent = event.error.message;
            }
        });
        this.stripe = stripe;
        this.stripe_card_element = card;
    },
    /**
     * destroys the card element and any stripe instance linked to the widget.
     *
     * @private
     */
    _unbindStripeCard: function () {
        if (this.stripe_card_element) {
            this.stripe_card_element.destroy();
        }
        this.stripe = undefined;
        this.stripe_card_element = undefined;
    },
    /**
     * @override
     */
    updateNewPaymentDisplayStatus: function () {
        var $checkedRadio = this.$('input[type="radio"]:checked');

        if ($checkedRadio.length !== 1) {
            return;
        }
        var provider = $checkedRadio.data('provider')
        if (provider === 'stripe') {
            // always re-init stripe (in case of multiple acquirers for stripe, make sure the stripe instance is using the right key)
            this._unbindStripeCard();
            if (this.isNewPaymentRadio($checkedRadio)) {
                this._bindStripeCard($checkedRadio);
            }
        }
        return this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    payEvent: function (ev) {
        ev.preventDefault();
        var $checkedRadio = this.$('input[type="radio"]:checked');

        // first we check that the user has selected a stripe as s2s payment method
        if ($checkedRadio.length === 1 && this.isNewPaymentRadio($checkedRadio) && $checkedRadio.data('provider') === 'stripe') {
            return this._createStripeToken(ev, $checkedRadio);
        } else {
            return this._super.apply(this, arguments);
        }
    },
    /**
     * @override
     */
    addPmEvent: function (ev) {
        ev.stopPropagation();
        ev.preventDefault();
        var $checkedRadio = this.$('input[type="radio"]:checked');

        // first we check that the user has selected a stripe as add payment method
        if ($checkedRadio.length === 1 && this.isNewPaymentRadio($checkedRadio) && $checkedRadio.data('provider') === 'stripe') {
            return this._createStripeToken(ev, $checkedRadio, true);
        } else {
            return this._super.apply(this, arguments);
        }
    },
});
});

```

## File: static\src\js\payment_processing.js

```javascript
odoo.define('payment_stripe.processing', function (require) {
'use strict';

var ajax = require('web.ajax');
var rpc = require('web.rpc')
var publicWidget = require('web.public.widget');

var PaymentProcessing = publicWidget.registry.PaymentProcessing;

return PaymentProcessing.include({
    init: function () {
        this._super.apply(this, arguments);
        this._authInProgress = false;
    },
    willStart: function () {
        return this._super.apply(this, arguments).then(function () {
            return ajax.loadJS("https://js.stripe.com/v3/");
        })
    },
    _stripeAuthenticate: function (tx) {
        var stripe = Stripe(tx.stripe_publishable_key);
        return stripe.handleCardPayment(tx.stripe_payment_intent_secret)
        .then(function(result) {
            if (result.error) {
                return rpc.query({
                    route: '/payment/stripe/s2s/process_payment_error',
                    params: _.extend({}, {reference: tx.reference,
                        stripe_payment_intent_secret: tx.stripe_payment_intent_secret,
                        error: result.error.message})
                }).then(()=> Promise.reject({"message": {"data": { "message": result.error.message}}}));
            }
            return rpc.query({
                route: '/payment/stripe/s2s/process_payment_intent',
                params: _.extend({}, result.paymentIntent, {reference: tx.reference}),
            });
        }).then(function() {
            window.location = '/payment/process';
        }).guardedCatch(function () {
            this._authInProgress = false;
        });
    },
    processPolledData: function(transactions) {
        this._super.apply(this, arguments);
        for (var itx=0; itx < transactions.length; itx++) {
            var tx = transactions[itx];
            if (tx.acquirer_provider === 'stripe' && tx.state === 'pending' && tx.stripe_payment_intent_secret && !this._authInProgress) {
                this._authInProgress = true;
                this._stripeAuthenticate(tx);
            }
        }
    },
});
});
```

## File: static\src\js\stripe.js

```javascript
odoo.define('payment_stripe.stripe', function (require) {
"use strict";

var ajax = require('web.ajax');
var core = require('web.core');

var qweb = core.qweb;
var _t = core._t;

ajax.loadXML('/payment_stripe/static/src/xml/stripe_templates.xml', qweb);

if ($.blockUI) {
    // our message needs to appear above the modal dialog
    $.blockUI.defaults.baseZ = 2147483647; //same z-index as StripeCheckout
    $.blockUI.defaults.css.border = '0';
    $.blockUI.defaults.css["background-color"] = '';
    $.blockUI.defaults.overlayCSS["opacity"] = '0.9';
}

require('web.dom_ready');
if (!$('.o_payment_form').length) {
    return Promise.reject("DOM doesn't contain '.o_payment_form'");
}

var observer = new MutationObserver(function (mutations, observer) {
    for (var i = 0; i < mutations.length; ++i) {
        for (var j = 0; j < mutations[i].addedNodes.length; ++j) {
            if (mutations[i].addedNodes[j].tagName.toLowerCase() === "form" && mutations[i].addedNodes[j].getAttribute('provider') === 'stripe') {
                _redirectToStripeCheckout($(mutations[i].addedNodes[j]));
            }
        }
    }
});

function displayError(message) {
    var wizard = $(qweb.render('stripe.error', {'msg': message || _t('Payment error')}));
    wizard.appendTo($('body')).modal({'keyboard': true});
    if ($.blockUI) {
        $.unblockUI();
    }
    $("#o_payment_form_pay").removeAttr('disabled');
}


function _redirectToStripeCheckout(providerForm) {
    // Open Checkout with further options
    if ($.blockUI) {
        var msg = _t("Just one more second, We are redirecting you to Stripe...");
        $.blockUI({
            'message': '<h2 class="text-white"><img src="/web/static/src/img/spin.png" class="fa-pulse"/>' +
                    '    <br />' + msg +
                    '</h2>'
        });
    }

    var paymentForm = $('.o_payment_form');
    if (!paymentForm.find('i').length) {
        paymentForm.append('<i class="fa fa-spinner fa-spin"/>');
        paymentForm.attr('disabled', 'disabled');
    }

    var _getStripeInputValue = function (name) {
        return providerForm.find('input[name="' + name + '"]').val();
    };

    var stripe = Stripe(_getStripeInputValue('stripe_key'));

    stripe.redirectToCheckout({
        sessionId: _getStripeInputValue('session_id')
    }).then(function (result) {
        if (result.error) {
            displayError(result.error.message);
        }
    });
}

$.getScript("https://js.stripe.com/v3/", function (data, textStatus, jqxhr) {
    observer.observe(document.body, {childList: true});
    _redirectToStripeCheckout($('form[provider="stripe"]'));
});
});

```

## File: static\src\xml\stripe_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="stripe.error">
        <div role="dialog" class="modal fade">
            <div class="modal-dialog">
                <div class="modal-content">
                    <header class="modal-header">
                        <h4 class="modal-title">Error</h4>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">×</button>
                    </header>
                    <main class="modal-body">
                        <t t-esc="msg"></t>
                    </main>
                    <footer class="modal-footer">
                        <a role="button" href="#" class="btn btn-link btn-sm" data-dismiss="modal">Close</a>
                    </footer>
                </div>
            </div>
        </div>
    </t>
</templates>

```

## File: views\payment_stripe_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <template id="stripe_form">
            <input type="hidden" name="data_set" t-att-data-action-url="tx_url" data-remove-me=""/>
            <input type='hidden' name='session_id' t-att-value='session_id'/>
            <input type="hidden" name="stripe_key" t-att-value="acquirer.stripe_publishable_key"/>
            <script type="text/javascript">
                odoo.define(function (require) {
                    var ajax = require('web.ajax');
                    ajax.loadJS("/payment_stripe/static/src/js/stripe.js");
                });
            </script>
        </template>

        <template id="stripe_s2s_form">
            <input type="hidden" name="data_set" value="/payment/stripe/s2s/create_json_3ds"/>
            <input type="hidden" name="acquirer_id" t-att-value="id"/>
            <input type="hidden" name="stripe_publishable_key" t-att-value="acq.sudo().stripe_publishable_key"/>
            <input type="hidden" name="currency_id" t-att-value="currency_id"/>
            <input t-if="return_url" type="hidden" name="return_url" t-att-value="return_url"/>
            <input t-if="partner_id" type="hidden" name="partner_id" t-att-value="partner_id"/>
            <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
            <div id="payment-form">
                <div id="card-element" class="m-3"/>
                <div id="card-errors" class="m-3 text-danger"/>
            </div>
        </template>

        <template id="assets_frontend" inherit_id="web.assets_frontend">
            <xpath expr="script[last()]" position="after">
                <script type="text/javascript" src="/payment_stripe/static/src/js/payment_form.js"></script>
                <script type="text/javascript" src="/payment_stripe/static/src/js/payment_processing.js"></script>

            </xpath>
        </template>
    </data>
</odoo>

```

## File: views\payment_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="acquirer_form_stripe" model="ir.ui.view">
        <field name="name">payment.acquirer.form.inherit</field>
        <field name="model">payment.acquirer</field>
        <field name="inherit_id" ref="payment.acquirer_form"/>
        <field name="arch" type="xml">
            <xpath expr='//group[@name="acquirer"]' position='inside'>
                <group attrs="{'invisible': [('provider', '!=', 'stripe')]}">
                    <field name="stripe_secret_key" attrs="{'required':[ ('provider', '=', 'stripe'), ('state', '!=', 'disabled')]}" password="True"/>
                    <field name="stripe_publishable_key" attrs="{'required':[ ('provider', '=', 'stripe'), ('state', '!=', 'disabled')]}" password="True"/>
                    <field name="stripe_webhook_secret" password="True"/>
                </group>
            </xpath>
            <xpath expr='//group[@name="acquirer_config"]' position='after'>
                <group attrs="{'invisible': [('provider', '!=', 'stripe')]}">
                    <field name="stripe_image_url"/>
                </group>
            </xpath>
        </field>
    </record>
</odoo>

```

