# Odoo Module: payment_flutterwave

Category: Accounting/Payment Providers

This file contains the source code of the Odoo module.

## File: const.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# The currencies supported by Flutterwave, in ISO 4217 format.
# See https://flutterwave.com/us/support/general/what-are-the-currencies-accepted-on-flutterwave.
# Last website update: June 2022.
# Last seen online: 24 November 2022.
SUPPORTED_CURRENCIES = [
    'GBP',
    'CAD',
    'XAF',
    'CLP',
    'COP',
    'EGP',
    'EUR',
    'GHS',
    'GNF',
    'KES',
    'MWK',
    'MAD',
    'NGN',
    'RWF',
    'SLL',
    'STD',
    'ZAR',
    'TZS',
    'UGX',
    'USD',
    'XOF',
    'ZMW',
]

# Mapping of transaction states to Flutterwave payment statuses.
PAYMENT_STATUS_MAPPING = {
    'pending': ['pending', 'pending auth'],
    'done': ['successful'],
    'cancel': ['cancelled'],
    'error': ['failed'],
}

# The codes of the payment methods to activate when Flutterwave is activated.
DEFAULT_PAYMENT_METHODS_CODES = [
    # Primary payment methods.
    'card',
    'mpesa',
    # Brand payment methods.
    'visa',
    'mastercard',
    'amex',
    'discover',
]

PAYMENT_METHODS_MAPPING = {
    'bank_transfer': 'banktransfer',
}

```

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

from odoo.addons.payment import setup_provider, reset_payment_provider


def post_init_hook(env):
    setup_provider(env, 'flutterwave')


def uninstall_hook(env):
    reset_payment_provider(env, 'flutterwave')

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': "Payment Provider: Flutterwave",
    'version': '1.0',
    'category': 'Accounting/Payment Providers',
    'sequence': 350,
    'summary': "A Nigerian payment provider covering several African countries.",
    'description': " ",  # Non-empty string to avoid loading the README file.
    'depends': ['payment'],
    'data': [
        'views/payment_flutterwave_templates.xml',
        'views/payment_provider_views.xml',

        'data/payment_provider_data.xml',
    ],
    'assets': {
        'web.assets_frontend': [
            'payment_flutterwave/static/src/js/payment_form.js',
        ],
    },
    'post_init_hook': 'post_init_hook',
    'uninstall_hook': 'uninstall_hook',
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import hmac
import json
import logging
import pprint

from werkzeug.exceptions import Forbidden

from odoo import http
from odoo.exceptions import ValidationError
from odoo.http import request


_logger = logging.getLogger(__name__)


class FlutterwaveController(http.Controller):
    _return_url = '/payment/flutterwave/return'
    _auth_return_url = '/payment/flutterwave/auth_return'
    _webhook_url = '/payment/flutterwave/webhook'

    @http.route(_return_url, type='http', methods=['GET'], auth='public')
    def flutterwave_return_from_checkout(self, **data):
        """ Process the notification data sent by Flutterwave after redirection from checkout.

        :param dict data: The notification data.
        """
        _logger.info("Handling redirection from Flutterwave with data:\n%s", pprint.pformat(data))

        # Handle the notification data.
        if data.get('status') != 'cancelled':
            request.env['payment.transaction'].sudo()._handle_notification_data('flutterwave', data)
        else:  # The customer cancelled the payment by clicking on the close button.
            pass  # Don't try to process this case because the transaction id was not provided.

        # Redirect the user to the status page.
        return request.redirect('/payment/status')

    @http.route(_auth_return_url, type='http', methods=['GET'], auth='public')
    def flutterwave_return_from_authorization(self, response):
        """ Process the response sent by Flutterwave after authorization.

        :param str response: The stringified JSON response.
        """
        data = json.loads(response)
        return self.flutterwave_return_from_checkout(**data)

    @http.route(_webhook_url, type='http', methods=['POST'], auth='public', csrf=False)
    def flutterwave_webhook(self):
        """ Process the notification data sent by Flutterwave to the webhook.

        :return: An empty string to acknowledge the notification.
        :rtype: str
        """
        data = request.get_json_data()
        _logger.info("Notification received from Flutterwave with data:\n%s", pprint.pformat(data))

        if data['event'] == 'charge.completed':
            try:
                # Check the origin of the notification.
                tx_sudo = request.env['payment.transaction'].sudo()._get_tx_from_notification_data(
                    'flutterwave', data['data']
                )
                signature = request.httprequest.headers.get('verif-hash')
                self._verify_notification_signature(signature, tx_sudo)

                # Handle the notification data.
                notification_data = data['data']
                tx_sudo._handle_notification_data('flutterwave', notification_data)
            except ValidationError:  # Acknowledge the notification to avoid getting spammed.
                _logger.exception("Unable to handle the notification data; skipping to acknowledge")
        return request.make_json_response('')

    @staticmethod
    def _verify_notification_signature(received_signature, tx_sudo):
        """ Check that the received signature matches the expected one.

        :param dict received_signature: The signature received with the notification data.
        :param recordset tx_sudo: The sudoed transaction referenced by the notification data, as a
                                  `payment.transaction` record.
        :return: None
        :raise Forbidden: If the signatures don't match.
        """
        # Check for the received signature.
        if not received_signature:
            _logger.warning("Received notification with missing signature.")
            raise Forbidden()

        # Compare the received signature with the expected signature.
        expected_signature = tx_sudo.provider_id.flutterwave_webhook_secret
        if not hmac.compare_digest(received_signature, expected_signature):
            _logger.warning("Received notification with invalid signature.")
            raise Forbidden()

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\neutralize.sql

```sql
-- disable flutterwave payment provider
UPDATE payment_provider
   SET flutterwave_public_key = NULL,
       flutterwave_secret_key = NULL,
       flutterwave_webhook_secret = NULL;

```

## File: data\payment_provider_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="payment.payment_provider_flutterwave" model="payment.provider">
        <field name="code">flutterwave</field>
        <field name="redirect_form_view_id" ref="redirect_form"/>
        <field name="allow_tokenization">True</field>
    </record>

</odoo>

```

## File: models\payment_provider.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import pprint

import requests
from werkzeug.urls import url_join

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError

from odoo.addons.payment_flutterwave import const


_logger = logging.getLogger(__name__)


class PaymentProvider(models.Model):
    _inherit = 'payment.provider'

    code = fields.Selection(
        selection_add=[('flutterwave', "Flutterwave")], ondelete={'flutterwave': 'set default'}
    )
    flutterwave_public_key = fields.Char(
        string="Flutterwave Public Key",
        help="The key solely used to identify the account with Flutterwave.",
        required_if_provider='flutterwave',
    )
    flutterwave_secret_key = fields.Char(
        string="Flutterwave Secret Key",
        required_if_provider='flutterwave',
        groups='base.group_system',
    )
    flutterwave_webhook_secret = fields.Char(
        string="Flutterwave Webhook Secret",
        required_if_provider='flutterwave',
        groups='base.group_system',
    )

    #=== COMPUTE METHODS ===#

    def _compute_feature_support_fields(self):
        """ Override of `payment` to enable additional features. """
        super()._compute_feature_support_fields()
        self.filtered(lambda p: p.code == 'flutterwave').update({
            'support_tokenization': True,
        })

    # === BUSINESS METHODS ===#

    @api.model
    def _get_compatible_providers(self, *args, is_validation=False, **kwargs):
        """ Override of `payment` to filter out Flutterwave providers for validation operations. """
        providers = super()._get_compatible_providers(*args, is_validation=is_validation, **kwargs)

        if is_validation:
            providers = providers.filtered(lambda p: p.code != 'flutterwave')

        return providers

    def _get_supported_currencies(self):
        """ Override of `payment` to return the supported currencies. """
        supported_currencies = super()._get_supported_currencies()
        if self.code == 'flutterwave':
            supported_currencies = supported_currencies.filtered(
                lambda c: c.name in const.SUPPORTED_CURRENCIES
            )
        return supported_currencies

    def _flutterwave_make_request(self, endpoint, payload=None, method='POST'):
        """ Make a request to Flutterwave API at the specified endpoint.

        Note: self.ensure_one()

        :param str endpoint: The endpoint to be reached by the request.
        :param dict payload: The payload of the request.
        :param str method: The HTTP method of the request.
        :return The JSON-formatted content of the response.
        :rtype: dict
        :raise ValidationError: If an HTTP error occurs.
        """
        self.ensure_one()

        url = url_join('https://api.flutterwave.com/v3/', endpoint)
        headers = {'Authorization': f'Bearer {self.flutterwave_secret_key}'}
        try:
            if method == 'GET':
                response = requests.get(url, params=payload, headers=headers, timeout=10)
            else:
                response = requests.post(url, json=payload, headers=headers, timeout=10)
            try:
                response.raise_for_status()
            except requests.exceptions.HTTPError:
                _logger.exception(
                    "Invalid API request at %s with data:\n%s", url, pprint.pformat(payload),
                )
                raise ValidationError("Flutterwave: " + _(
                    "The communication with the API failed. Flutterwave gave us the following "
                    "information: '%s'", response.json().get('message', '')
                ))
        except (requests.exceptions.ConnectionError, requests.exceptions.Timeout):
            _logger.exception("Unable to reach endpoint at %s", url)
            raise ValidationError(
                "Flutterwave: " + _("Could not establish the connection to the API.")
            )
        return response.json()

    def _get_default_payment_method_codes(self):
        """ Override of `payment` to return the default payment method codes. """
        default_codes = super()._get_default_payment_method_codes()
        if self.code != 'flutterwave':
            return default_codes
        return const.DEFAULT_PAYMENT_METHODS_CODES

```

## File: models\payment_token.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class PaymentToken(models.Model):
    _inherit = 'payment.token'

    flutterwave_customer_email = fields.Char(
        help="The email of the customer at the time the token was created.", readonly=True
    )

```

## File: models\payment_transaction.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import pprint

from werkzeug import urls

from odoo import _, models
from odoo.exceptions import UserError, ValidationError

from odoo.addons.payment import utils as payment_utils
from odoo.addons.payment_flutterwave import const
from odoo.addons.payment_flutterwave.controllers.main import FlutterwaveController


_logger = logging.getLogger(__name__)


class PaymentTransaction(models.Model):
    _inherit = 'payment.transaction'

    def _get_specific_processing_values(self, processing_values):
        """ Override of payment to redirect pending token-flow transactions.

        If the financial institution insists on 3-D Secure authentication, this
        override will redirect the user to the provided authorization page.

        Note: `self.ensure_one()`
        """
        res = super()._get_specific_processing_values(processing_values)
        if self._flutterwave_is_authorization_pending():
            res['redirect_form_html'] = self.env['ir.qweb']._render(
                self.provider_id.redirect_form_view_id.id,
                {'api_url': self.provider_reference},
            )
        return res

    def _get_specific_rendering_values(self, processing_values):
        """ Override of payment to return Flutterwave-specific rendering values.

        Note: self.ensure_one() from `_get_processing_values`

        :param dict processing_values: The generic and specific processing values of the transaction
        :return: The dict of provider-specific processing values.
        :rtype: dict
        """
        res = super()._get_specific_rendering_values(processing_values)
        if self.provider_code != 'flutterwave':
            return res

        # Initiate the payment and retrieve the payment link data.
        base_url = self.provider_id.get_base_url()
        payload = {
            'tx_ref': self.reference,
            'amount': self.amount,
            'currency': self.currency_id.name,
            'redirect_url': urls.url_join(base_url, FlutterwaveController._return_url),
            'customer': {
                'email': self.partner_email,
                'name': self.partner_name,
                'phonenumber': self.partner_phone,
            },
            'customizations': {
                'title': self.company_id.name,
                'logo': urls.url_join(base_url, f'web/image/res.company/{self.company_id.id}/logo'),
            },
            'payment_options': const.PAYMENT_METHODS_MAPPING.get(
                self.payment_method_code, self.payment_method_code
            ),
        }
        payment_link_data = self.provider_id._flutterwave_make_request('payments', payload=payload)

        # Extract the payment link URL and embed it in the redirect form.
        rendering_values = {
            'api_url': payment_link_data['data']['link'],
        }
        return rendering_values

    def _send_payment_request(self):
        """ Override of payment to send a payment request to Flutterwave.

        Note: self.ensure_one()

        :return: None
        :raise UserError: If the transaction is not linked to a token.
        """
        super()._send_payment_request()
        if self.provider_code != 'flutterwave':
            return

        # Prepare the payment request to Flutterwave.
        if not self.token_id:
            raise UserError("Flutterwave: " + _("The transaction is not linked to a token."))

        first_name, last_name = payment_utils.split_partner_name(self.partner_name)
        base_url = self.provider_id.get_base_url()
        data = {
            'token': self.token_id.provider_ref,
            'email': self.token_id.flutterwave_customer_email,
            'amount': self.amount,
            'currency': self.currency_id.name,
            'country': self.company_id.country_id.code,
            'tx_ref': self.reference,
            'first_name': first_name,
            'last_name': last_name,
            'ip': payment_utils.get_customer_ip_address(),
            'redirect_url': urls.url_join(base_url, FlutterwaveController._auth_return_url),
        }

        # Make the payment request to Flutterwave.
        response_content = self.provider_id._flutterwave_make_request(
            'tokenized-charges', payload=data
        )

        # Handle the payment request response.
        _logger.info(
            "payment request response for transaction with reference %s:\n%s",
            self.reference, pprint.pformat(response_content)
        )
        self._handle_notification_data('flutterwave', response_content['data'])

    def _get_tx_from_notification_data(self, provider_code, notification_data):
        """ Override of payment to find the transaction based on Flutterwave data.

        :param str provider_code: The code of the provider that handled the transaction.
        :param dict notification_data: The notification data sent by the provider.
        :return: The transaction if found.
        :rtype: recordset of `payment.transaction`
        :raise ValidationError: If inconsistent data were received.
        :raise ValidationError: If the data match no transaction.
        """
        tx = super()._get_tx_from_notification_data(provider_code, notification_data)
        if provider_code != 'flutterwave' or len(tx) == 1:
            return tx

        reference = notification_data.get('tx_ref') or notification_data.get('txRef')
        if not reference:
            raise ValidationError("Flutterwave: " + _("Received data with missing reference."))

        tx = self.search([('reference', '=', reference), ('provider_code', '=', 'flutterwave')])
        if not tx:
            raise ValidationError(
                "Flutterwave: " + _("No transaction found matching reference %s.", reference)
            )
        return tx

    def _process_notification_data(self, notification_data):
        """ Override of payment to process the transaction based on Flutterwave data.

        Note: self.ensure_one()

        :param dict notification_data: The notification data sent by the provider.
        :return: None
        :raise ValidationError: If inconsistent data were received.
        """
        super()._process_notification_data(notification_data)
        if self.provider_code != 'flutterwave':
            return

        # Verify the notification data.
        verification_response_content = self.provider_id._flutterwave_make_request(
            'transactions/verify_by_reference', payload={'tx_ref': self.reference}, method='GET'
        )
        verified_data = verification_response_content['data']

        # Update the provider reference.
        self.provider_reference = verified_data['id']

        # Update payment method.
        payment_method_type = verified_data.get('payment_type', '')
        if payment_method_type == 'card':
            payment_method_type = verified_data.get('card', {}).get('type').lower()
        payment_method = self.env['payment.method']._get_from_code(
            payment_method_type, mapping=const.PAYMENT_METHODS_MAPPING
        )
        self.payment_method_id = payment_method or self.payment_method_id

        # Update the payment state.
        payment_status = verified_data['status'].lower()
        if payment_status in const.PAYMENT_STATUS_MAPPING['pending']:
            auth_url = notification_data.get('meta', {}).get('authorization', {}).get('redirect')
            if auth_url:
                # will be set back to the actual value after moving away from pending
                self.provider_reference = auth_url
            self._set_pending()
        elif payment_status in const.PAYMENT_STATUS_MAPPING['done']:
            self._set_done()
            has_token_data = 'token' in verified_data.get('card', {})
            if self.tokenize and has_token_data:
                self._flutterwave_tokenize_from_notification_data(verified_data)
        elif payment_status in const.PAYMENT_STATUS_MAPPING['cancel']:
            self._set_canceled()
        elif payment_status in const.PAYMENT_STATUS_MAPPING['error']:
            self._set_error(_(
                "An error occurred during the processing of your payment (status %s). Please try "
                "again.", payment_status
            ))
        else:
            _logger.warning(
                "Received data with invalid payment status (%s) for transaction with reference %s.",
                payment_status, self.reference
            )
            self._set_error("Flutterwave: " + _("Unknown payment status: %s", payment_status))

    def _flutterwave_tokenize_from_notification_data(self, notification_data):
        """ Create a new token based on the notification data.

        Note: self.ensure_one()

        :param dict notification_data: The notification data sent by the provider.
        :return: None
        """
        self.ensure_one()

        token = self.env['payment.token'].create({
            'provider_id': self.provider_id.id,
            'payment_method_id': self.payment_method_id.id,
            'payment_details': notification_data['card']['last_4digits'],
            'partner_id': self.partner_id.id,
            'provider_ref': notification_data['card']['token'],
            'flutterwave_customer_email': notification_data['customer']['email'],
        })
        self.write({
            'token_id': token,
            'tokenize': False,
        })
        _logger.info(
            "created token with id %(token_id)s for partner with id %(partner_id)s from "
            "transaction with reference %(ref)s",
            {
                'token_id': token.id,
                'partner_id': self.partner_id.id,
                'ref': self.reference,
            },
        )

    def _flutterwave_is_authorization_pending(self):
        return self.filtered_domain([
            ('provider_code', '=', 'flutterwave'),
            ('operation', '=', 'online_token'),
            ('state', '=', 'pending'),
            ('provider_reference', 'ilike', 'https'),
        ])

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import payment_provider
from . import payment_token
from . import payment_transaction

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M43.105 50h.972v-3.182h1.108V46H42v.818h1.105V50Zm2.775 0h.858v-2.547h.053L47.663 50h.555l.871-2.547h.056V50H50v-4h-1.108l-.924 2.714h-.05L46.99 46h-1.11v4Z" fill="#D1D5DB"/><path d="M4 15.425c0-2.209.644-4.089 2.027-5.428l2.385 2.35c-2.647 2.608-.334 10.715 7.227 18.164 7.56 7.448 15.79 9.728 18.436 7.12l2.386 2.35c-4.484 4.417-14.668 1.268-23.207-7.12C7.339 27.01 4 20.406 4 15.424Z" fill="#009A46"/><path d="M19.05 42c-2.242 0-4.15-.634-5.51-1.997l2.385-2.35c2.648 2.608 10.876.329 18.436-7.12 7.561-7.449 9.875-15.555 7.227-18.164l2.385-2.35c4.484 4.418 1.288 14.452-7.226 22.864C30.807 38.71 24.082 42 19.049 42Z" fill="#FF5805"/><path d="M37.51 29.522c-1.455-4.112-4.412-8.506-8.323-12.36C20.648 8.75 10.463 5.625 5.98 10.019c-.31.329-.024 1.104.62 1.739.644.634 1.455.916 1.765.61 2.647-2.608 10.876-.329 18.437 7.12 3.577 3.525 6.248 7.473 7.536 11.115 1.121 3.195 1.026 5.78-.286 7.073-.31.305-.048 1.08.62 1.738.668.658 1.455.917 1.765.611 2.29-2.279 2.671-6.015 1.073-10.503Z" fill="#F5AFCB"/><path d="M43.95 10.02c-2.29-2.256-6.083-2.632-10.638-1.08-4.174 1.433-8.634 4.346-12.545 8.2-8.539 8.412-11.71 18.446-7.227 22.863.31.306 1.121.047 1.765-.61.644-.659.93-1.434.62-1.74-2.647-2.608-.334-10.714 7.227-18.163 3.577-3.525 7.584-6.157 11.257-7.425 3.244-1.105 5.867-1.01 7.18.282.31.305 1.096.046 1.764-.611.668-.658.93-1.387.596-1.716Z" fill="#FF9B00"/></svg>

```

## File: static\src\js\payment_form.js

```javascript
/** @odoo-module */

import paymentForm from '@payment/js/payment_form';

paymentForm.include({

    /**
     * Allow forcing redirect to authorization url for Flutterwave token flow.
     *
     * @override method from @payment/js/payment_form
     * @private
     * @param {string} providerCode - The code of the selected payment option's provider.
     * @param {number} paymentOptionId - The id of the selected payment option.
     * @param {string} paymentMethodCode - The code of the selected payment method, if any.
     * @param {object} processingValues - The processing values of the transaction.
     * @return {void}
     */
    _processTokenFlow(providerCode, paymentOptionId, paymentMethodCode, processingValues) {
        if (providerCode === 'flutterwave' && processingValues.redirect_form_html) {
            // Authorization uses POST instead of GET
            const redirect_form_html = processingValues.redirect_form_html.replace(
                /method="get"/, 'method="post"'
            )
            this._processRedirectFlow(providerCode, paymentOptionId, paymentMethodCode, {
                ...processingValues,
                redirect_form_html,
            });
        } else {
            this._super(...arguments);
        }
    }
});

```

## File: views\payment_flutterwave_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="redirect_form">
        <form t-att-action="api_url" method="get"/>
    </template>

</odoo>

```

## File: views\payment_provider_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_provider_form" model="ir.ui.view">
        <field name="name">Flutterwave Provider Form</field>
        <field name="model">payment.provider</field>
        <field name="inherit_id" ref="payment.payment_provider_form"/>
        <field name="arch" type="xml">
            <group name="provider_credentials" position="inside">
                <group invisible="code != 'flutterwave'"
                       name="flutterwave_credentials">
                    <field name="flutterwave_public_key"
                           string="Public Key"
                           required="code == 'flutterwave' and state != 'disabled'"/>
                    <field name="flutterwave_secret_key"
                           string="Secret Key"
                           required="code == 'flutterwave' and state != 'disabled'"
                           password="True"/>
                    <field name="flutterwave_webhook_secret"
                           string="Webhook Secret"
                           required="code == 'flutterwave' and state != 'disabled'"
                           password="True"/>
                </group>
            </group>
        </field>
    </record>

</odoo>

```

