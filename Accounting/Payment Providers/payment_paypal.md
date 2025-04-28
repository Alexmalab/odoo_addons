# Odoo Module: payment_paypal

Category: Accounting/Payment Providers

This file contains the source code of the Odoo module.

## File: const.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# ISO 4217 codes of currencies supported by PayPal
# See https://developer.paypal.com/docs/reports/reference/paypal-supported-currencies/.
# Last seen on: 22 September 2022.
SUPPORTED_CURRENCIES = (
    'AUD',
    'BRL',
    'CAD',
    'CNY',
    'CZK',
    'DKK',
    'EUR',
    'HKD',
    'HUF',
    'ILS',
    'JPY',
    'MYR',
    'MXN',
    'TWD',
    'NZD',
    'NOK',
    'PHP',
    'PLN',
    'GBP',
    'RUB',
    'SGD',
    'SEK',
    'CHF',
    'THB',
    'USD',
)

# The codes of the payment methods to activate when Paypal is activated.
DEFAULT_PAYMENT_METHOD_CODES = {
    # Primary payment methods.
    'paypal',
}

# Mapping of transaction states to PayPal payment statuses.
# See https://developer.paypal.com/docs/api/orders/v2/#definition-capture_status.
# See https://developer.paypal.com/api/rest/webhooks/event-names/#orders.
PAYMENT_STATUS_MAPPING = {
    'pending': (
        'PENDING',
        'CREATED',
        'APPROVED',  # The buyer approved a checkout order.
    ),
    'done': (
        'COMPLETED',
        'CAPTURED',
    ),
    'cancel': (
        'DECLINED',
        'DENIED',
        'VOIDED',
    ),
    'error': ('FAILED',),
}

# Events which are handled by the webhook.
# See https://developer.paypal.com/api/rest/webhooks/event-names/
HANDLED_WEBHOOK_EVENTS = [
    'CHECKOUT.ORDER.COMPLETED',
    'CHECKOUT.ORDER.APPROVED',
    'CHECKOUT.PAYMENT-APPROVAL.REVERSED',
]

```

## File: utils.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

def get_normalized_email_account(provider):
    """ Remove unicode characters, such as \u200b coming from pasted emails, from the provider's
    paypal email account.

    :return: The normalized address of the paypal email account of the provider.
    :rtype: str
    """
    return provider.paypal_email_account.encode('ascii', 'ignore').decode('utf-8')

```

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

from odoo.addons.payment import setup_provider, reset_payment_provider


def post_init_hook(env):
    setup_provider(env, 'paypal')


def uninstall_hook(env):
    reset_payment_provider(env, 'paypal')

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Payment Provider: Paypal',
    'version': '2.0',
    'category': 'Accounting/Payment Providers',
    'sequence': 350,
    'summary': "An American payment provider for online payments all over the world.",
    'description': " ",  # Non-empty string to avoid loading the README file.
    'depends': ['payment'],
    'data': [
        'views/payment_form_templates.xml',
        'views/payment_provider_views.xml',
        'views/payment_transaction_views.xml',

        'data/payment_provider_data.xml',
    ],
    'post_init_hook': 'post_init_hook',
    'uninstall_hook': 'uninstall_hook',
    'assets': {
        'web.assets_frontend': [
            'payment_paypal/static/src/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json
import logging
import pprint

from werkzeug.exceptions import Forbidden

from odoo import _, http
from odoo.exceptions import ValidationError
from odoo.http import request

from odoo.addons.payment import utils as payment_utils
from odoo.addons.payment_paypal import const


_logger = logging.getLogger(__name__)


class PaypalController(http.Controller):
    _complete_url = '/payment/paypal/complete_order'
    _webhook_url = '/payment/paypal/webhook/'

    @http.route(_complete_url, type='json', auth='public', methods=['POST'])
    def paypal_complete_order(self, provider_id, order_id, reference=None):
        """ Make a capture request and handle the notification data.

        :param int provider_id: The provider handling the transaction, as a `payment.provider` id.
        :param string order_id: The order id provided by PayPal to identify the order.
        :param str reference: The reference of the transaction used to generate idempotency key.
        :return: None
        """
        provider_sudo = request.env['payment.provider'].browse(provider_id).sudo()
        idempotency_key = None
        if reference:
            tx_sudo = request.env['payment.transaction'].sudo()._get_tx_from_notification_data(
                'paypal', {'reference_id': reference}
            )
            idempotency_key = payment_utils.generate_idempotency_key(
                tx_sudo, scope='payment_request_controller'
            )
        response = provider_sudo._paypal_make_request(
            f'/v2/checkout/orders/{order_id}/capture', idempotency_key=idempotency_key
        )
        normalized_response = self._normalize_paypal_data(response)
        tx_sudo = request.env['payment.transaction'].sudo()._get_tx_from_notification_data(
            'paypal', normalized_response
        )
        tx_sudo._handle_notification_data('paypal', normalized_response)

    @http.route(_webhook_url, type='http', auth='public', methods=['POST'], csrf=False)
    def paypal_webhook(self):
        """ Process the notification data sent by PayPal to the webhook.

        See https://developer.paypal.com/docs/api/webhooks/v1/.

        :return: An empty string to acknowledge the notification.
        :rtype: str
        """
        data = request.get_json_data()
        if data.get('event_type') in const.HANDLED_WEBHOOK_EVENTS:
            normalized_data = self._normalize_paypal_data(
                data.get('resource'), from_webhook=True
            )
            _logger.info("Notification received from PayPal with data:\n%s", pprint.pformat(data))
            try:
                # Check the origin and integrity of the notification.
                tx_sudo = request.env['payment.transaction'].sudo()._get_tx_from_notification_data(
                    'paypal', normalized_data
                )
                self._verify_notification_origin(data, tx_sudo)

                # Handle the notification data.
                tx_sudo._handle_notification_data('paypal', normalized_data)
            except ValidationError:  # Acknowledge the notification to avoid getting spammed.
                _logger.warning(
                    "Unable to handle the notification data; skipping to acknowledge.",
                    exc_info=True,
                )
        return request.make_json_response('')

    def _normalize_paypal_data(self, data, from_webhook=False):
        """ Normalize the payment data received from PayPal.

        The payment data received from PayPal has a different format depending on whether the data
        come from the payment request response, or from the webhook.

        :param dict data: The data to normalize.
        :param bool from_webhook: Whether the data come from the webhook.
        :return: The normalized data.
        :rtype: dict
        """
        purchase_unit = data['purchase_units'][0]
        result = {
            'payment_source': data['payment_source'].keys(),
            'reference_id': purchase_unit.get('reference_id')
        }
        if from_webhook:
            result.update({
                **purchase_unit,
                'txn_type': data.get('intent'),
                'id': data.get('id'),
                'status': data.get('status'),
            })
        else:
            if captured := purchase_unit.get('payments', {}).get('captures'):
                result.update({
                    **captured[0],
                    'txn_type': 'CAPTURE',
                })
            else:
                raise ValidationError("PayPal: " + _("Invalid response format, can't normalize."))
        return result

    def _verify_notification_origin(self, notification_data, tx_sudo):
        """ Check that the notification was sent by PayPal.

        See https://developer.paypal.com/docs/api/webhooks/v1/#verify-webhook-signature_post.

        :param dict notification_data: The notification data
        :param recordset tx_sudo: The sudoed transaction referenced in the notification data, as a
                                  `payment.transaction` record
        :return: None
        :raise Forbidden: If the notification origin can't be verified.
        """
        headers = request.httprequest.headers
        data = json.dumps({
            'transmission_id': headers.get('PAYPAL-TRANSMISSION-ID'),
            'transmission_time': headers.get('PAYPAL-TRANSMISSION-TIME'),
            'cert_url': headers.get('PAYPAL-CERT-URL'),
            'auth_algo': headers.get('PAYPAL-AUTH-ALGO'),
            'transmission_sig': headers.get('PAYPAL-TRANSMISSION-SIG'),
            'webhook_id': tx_sudo.provider_id.paypal_webhook_id,
            'webhook_event': notification_data,
        })
        verification = tx_sudo.provider_id._paypal_make_request(
            '/v1/notifications/verify-webhook-signature', data=data
        )
        if verification.get('verification_status') != 'SUCCESS':
            _logger.warning("Received notification that was not verified by PayPal.")
            raise Forbidden()

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\neutralize.sql

```sql
-- disable paypal payment provider
UPDATE payment_provider
   SET paypal_email_account = NULL,
       paypal_client_id = NULL,
       paypal_client_secret = NULL;

```

## File: data\payment_provider_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="payment.payment_provider_paypal" model="payment.provider">
        <field name="code">paypal</field>
    </record>

</odoo>

```

## File: models\payment_provider.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json
import logging
import pprint
import requests

from datetime import timedelta
from werkzeug import urls

from odoo import _, fields, models
from odoo.exceptions import UserError, ValidationError

from odoo.addons.payment_paypal import const
from odoo.addons.payment_paypal.controllers.main import PaypalController


_logger = logging.getLogger(__name__)


class PaymentProvider(models.Model):
    _inherit = 'payment.provider'

    code = fields.Selection(
        selection_add=[('paypal', "PayPal")], ondelete={'paypal': 'set default'}
    )
    paypal_email_account = fields.Char(
        string="Email",
        help="The public business email solely used to identify the account with PayPal",
        required_if_provider='paypal',
        default=lambda self: self.env.company.email,
    )
    paypal_client_id = fields.Char(string="PayPal Client ID", required_if_provider='paypal')
    paypal_client_secret = fields.Char(string="PayPal Client Secret", groups='base.group_system')
    paypal_access_token = fields.Char(
        string="PayPal Access Token",
        help="The short-lived token used to access Paypal APIs",
        groups='base.group_system',
    )
    paypal_access_token_expiry = fields.Datetime(
        string="PayPal Access Token Expiry",
        help="The moment at which the access token becomes invalid.",
        default='1970-01-01',
        groups='base.group_system',
    )
    paypal_webhook_id = fields.Char(string="PayPal Webhook ID")

    # === ACTION METHODS === #

    def action_paypal_create_webhook(self):
        """ Create a new webhook.

        Note: This action only works for instances using a public URL.

        :return: None
        :raise UserError: If the base URL is not in HTTPS.
        """
        base_url = self.get_base_url()
        if 'localhost' in base_url:
            raise UserError(
                "PayPal: " + _("You must have an HTTPS connection to generate a webhook.")
            )
        data = {
            'url': urls.url_join(base_url, PaypalController._webhook_url),
            'event_types': [{'name': event_type} for event_type in const.HANDLED_WEBHOOK_EVENTS]
        }
        webhook_data = self._paypal_make_request('/v1/notifications/webhooks', json_payload=data)
        self.paypal_webhook_id = webhook_data.get('id')

    #=== BUSINESS METHODS ===#

    def _paypal_make_request(
        self, endpoint, data=None, json_payload=None, auth=None, is_refresh_token_request=False,
        idempotency_key=None,
    ):
        """ Make a request to Paypal API at the specified endpoint.

        Note: self.ensure_one()

        :param str endpoint: The endpoint to be reached by the request.
        :param dict data: The string payload of the request.
        :param dict json_payload: The JSON-formatted payload of the request.
        :param tuple auth: The authentication data.
        :param bool is_refresh_token_request: Whether the request is for refreshing the access
                                              token.
        :param str idempotency_key: The idempotency key to pass in the request.
        :return: The JSON-formatted content of the response.
        :rtype: dict
        :raise ValidationError: If an HTTP error occurs.
        """
        url = self._paypal_get_api_url() + endpoint
        headers = {'Content-Type': 'application/json'}  # PayPal always wants JSON content-type.
        if idempotency_key:
            headers['PayPal-Request-Id'] = idempotency_key
        if not is_refresh_token_request:
            headers['Authorization'] = f'Bearer {self._paypal_fetch_access_token()}'
        try:
            response = requests.post(
                url, headers=headers, data=data, json=json_payload, auth=auth, timeout=10
            )
            try:
                response.raise_for_status()
            except requests.exceptions.HTTPError:
                payload = data or json_payload
                # PayPal errors https://developer.paypal.com/api/rest/reference/orders/v2/errors/
                _logger.exception(
                    "Invalid API request at %s with data:\n%s", url, pprint.pformat(payload)
                )
                msg = response.json().get('message', '')
                raise ValidationError(
                    "PayPal: " + _("The communication with the API failed. Details: %s", msg)
                )
        except (requests.exceptions.ConnectionError, requests.exceptions.Timeout):
            _logger.exception("Unable to reach endpoint at %s", url)
            raise ValidationError("PayPal: " + _("Could not establish the connection to the API."))
        return response.json()

    def _paypal_fetch_access_token(self):
        """ Generate a new access token if it's expired, otherwise return the existing access token.

        :return: A valid access token.
        :rtype: str
        :raise ValidationError: If the access token can not be fetched.
        """
        if fields.Datetime.now() > self.paypal_access_token_expiry - timedelta(minutes=5):
            response_content = self._paypal_make_request(
                '/v1/oauth2/token',
                data={'grant_type': 'client_credentials'},
                auth=(self.paypal_client_id, self.paypal_client_secret),
                is_refresh_token_request=True,
            )
            access_token = response_content['access_token']
            if not access_token:
                raise ValidationError("PayPal: " + _("Could not generate a new access token."))
            self.write({
                'paypal_access_token': access_token,
                'paypal_access_token_expiry': fields.Datetime.now() + timedelta(
                    seconds=response_content['expires_in']
                ),
            })
        return self.paypal_access_token

    # === BUSINESS METHODS - GETTERS === #

    def _get_supported_currencies(self):
        """ Override of `payment` to return the supported currencies. """
        supported_currencies = super()._get_supported_currencies()
        if self.code == 'paypal':
            supported_currencies = supported_currencies.filtered(
                lambda c: c.name in const.SUPPORTED_CURRENCIES
            )
        return supported_currencies

    def _paypal_get_api_url(self):
        """ Return the API URL according to the provider state.

        Note: self.ensure_one()

        :return: The API URL
        :rtype: str
        """
        self.ensure_one()

        if self.state == 'enabled':
            return 'https://api-m.paypal.com'
        else:
            return 'https://api-m.sandbox.paypal.com'

    def _get_default_payment_method_codes(self):
        """ Override of `payment` to return the default payment method codes. """
        default_codes = super()._get_default_payment_method_codes()
        if self.code != 'paypal':
            return default_codes
        return const.DEFAULT_PAYMENT_METHOD_CODES

    def _paypal_get_inline_form_values(self, currency=None):
        """ Return a serialized JSON of the required values to render the inline form.

        Note: `self.ensure_one()`

        :param res.currency currency: The transaction currency.
        :return: The JSON serial of the required values to render the inline form.
        :rtype: str
        """
        inline_form_values = {
            'provider_id': self.id,
            'client_id': self.paypal_client_id,
            'currency_code': currency and currency.name,
        }
        return json.dumps(inline_form_values)

```

## File: models\payment_transaction.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import pprint

from odoo import _, fields, models
from odoo.exceptions import ValidationError

from odoo.addons.payment import utils as payment_utils
from odoo.addons.payment_paypal import utils as paypal_utils
from odoo.addons.payment_paypal.const import PAYMENT_STATUS_MAPPING

_logger = logging.getLogger(__name__)


class PaymentTransaction(models.Model):
    _inherit = 'payment.transaction'

    # See https://developer.paypal.com/docs/api-basics/notifications/ipn/IPNandPDTVariables/
    # this field has no use in Odoo except for debugging
    paypal_type = fields.Char(string="PayPal Transaction Type")

    def _get_specific_processing_values(self, processing_values):
        """ Override of `payment` to return the Paypal-specific processing values.

        Note: self.ensure_one() from `_get_processing_values`

        :param dict processing_values: The generic and specific processing values of the
                                       transaction.
        :return: The dict of provider-specific processing values
        :rtype: dict
        """
        res = super()._get_specific_processing_values(processing_values)
        if self.provider_code != 'paypal':
            return res

        payload = self._paypal_prepare_order_payload()

        _logger.info(
            "Sending '/checkout/orders' request for transaction with reference %s:\n%s",
            self.reference, pprint.pformat(payload)
        )
        idempotency_key = payment_utils.generate_idempotency_key(
            self, scope='payment_request_order'
        )
        order_data = self.provider_id._paypal_make_request(
            '/v2/checkout/orders', json_payload=payload, idempotency_key=idempotency_key
        )
        _logger.info(
            "Response of '/checkout/orders' request for transaction with reference %s:\n%s",
            self.reference, pprint.pformat(order_data)
        )
        return {'order_id': order_data['id']}

    def _paypal_prepare_order_payload(self):
        """ Prepare the payload for the Paypal create order request.

        :return: The requested payload to create a Paypal order.
        :rtype: dict
        """
        country_code = self.partner_country_id.code or self.company_id.country_id.code
        partner_first_name, partner_last_name = payment_utils.split_partner_name(self.partner_name)
        payload = {
            'intent': 'CAPTURE',
            'purchase_units': [
                {
                    'reference_id': self.reference,
                    'description': f'{self.company_id.name}: {self.reference}',
                    'amount': {
                        'currency_code': self.currency_id.name,
                        'value': self.amount,
                    },
                    'payee':  {
                        'display_data': {
                            'business_email':  self.provider_id.company_id.email,
                            'brand_name': self.provider_id.company_id.name,
                        },
                        'email_address': paypal_utils.get_normalized_email_account(self.provider_id)
                    },
                },
            ],
            'payment_source': {
                'paypal': {
                    'experience_context': {
                        'shipping_preference': 'NO_SHIPPING',
                    },
                    'name': {
                        'given_name': partner_first_name,
                        'surname': partner_last_name,
                    },
                    'address': {
                        'address_line_1': self.partner_address,
                        'admin_area_1': self.partner_state_id.name,
                        'admin_area_2': self.partner_city,
                        'postal_code': self.partner_zip,
                        'country_code': country_code,
                    },
                },
            },
        }
        # PayPal does not accept None set to fields and to avoid users getting errors when email
        # is not set on company we will add it conditionally since its not a required field.
        if self.partner_email:
            payload['payment_source']['paypal']['email_address'] = self.partner_email

        if company_email := self.provider_id.company_id.email:
            payload['purchase_units'][0]['payee']['display_data']['business_email'] = company_email

        return payload

    def _get_tx_from_notification_data(self, provider_code, notification_data):
        """ Override of `payment` to find the transaction based on Paypal data.

        :param str provider_code: The code of the provider that handled the transaction.
        :param dict notification_data: The notification data sent by the provider.
        :return: The transaction if found.
        :rtype: payment.transaction
        :raise ValidationError: If the data match no transaction.
        """
        tx = super()._get_tx_from_notification_data(provider_code, notification_data)
        if provider_code != 'paypal' or len(tx) == 1:
            return tx

        reference = notification_data.get('reference_id')
        tx = self.search([('reference', '=', reference), ('provider_code', '=', 'paypal')])
        if not tx:
            raise ValidationError(
                "PayPal: " + _("No transaction found matching reference %s.", reference)
            )
        return tx

    def _process_notification_data(self, notification_data):
        """ Override of `payment` to process the transaction based on Paypal data.

        Note: self.ensure_one()

        :param dict notification_data: The notification data sent by the provider.
        :return: None
        :raise ValidationError: If inconsistent data were received.
        """
        super()._process_notification_data(notification_data)
        if self.provider_code != 'paypal':
            return

        if not notification_data:
            self._set_canceled(state_message=_("The customer left the payment page."))
            return

        amount = notification_data.get('amount').get('value')
        currency_code = notification_data.get('amount').get('currency_code')
        assert amount and currency_code, "PayPal: missing amount or currency"
        assert self.currency_id.compare_amounts(float(amount), self.amount) == 0, \
            "PayPal: mismatching amounts"
        assert currency_code == self.currency_id.name, "PayPal: mismatching currency codes"

        # Update the provider reference.
        txn_id = notification_data.get('id')
        txn_type = notification_data.get('txn_type')
        if not all((txn_id, txn_type)):
            raise ValidationError(
                "PayPal: " + _(
                    "Missing value for txn_id (%(txn_id)s) or txn_type (%(txn_type)s).",
                    txn_id=txn_id, txn_type=txn_type
                )
            )
        self.provider_reference = txn_id
        self.paypal_type = txn_type

        # Force PayPal as the payment method if it exists.
        self.payment_method_id = self.env['payment.method'].search(
            [('code', '=', 'paypal')], limit=1
        ) or self.payment_method_id

        # Update the payment state.
        payment_status = notification_data.get('status')

        if payment_status in PAYMENT_STATUS_MAPPING['pending']:
            self._set_pending(state_message=notification_data.get('pending_reason'))
        elif payment_status in PAYMENT_STATUS_MAPPING['done']:
            self._set_done()
        elif payment_status in PAYMENT_STATUS_MAPPING['cancel']:
            self._set_canceled()
        else:
            _logger.info(
                "received data with invalid payment status (%s) for transaction with reference %s",
                payment_status, self.reference
            )
            self._set_error(
                "PayPal: " + _("Received data with invalid payment status: %s", payment_status)
            )

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import payment_provider
from . import payment_transaction

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="m17.287 44.574.74-4.623-1.649-.038H8.5l5.475-34.116a.449.449 0 0 1 .445-.373h13.282c4.41 0 7.453.901 9.042 2.682.745.835 1.219 1.707 1.448 2.668.241 1.007.245 2.211.01 3.68l-.017.107v.94l.745.415c.627.327 1.126.702 1.508 1.13.637.714 1.05 1.622 1.224 2.698.18 1.106.12 2.423-.174 3.913-.34 1.715-.89 3.208-1.632 4.43a9.172 9.172 0 0 1-2.584 2.784c-.986.687-2.157 1.21-3.48 1.543-1.284.329-2.747.494-4.35.494h-1.035a3.17 3.17 0 0 0-2.02.73 3.063 3.063 0 0 0-1.054 1.85l-.078.415-1.308 8.15-.06.298c-.015.095-.042.142-.082.174a.221.221 0 0 1-.136.05h-6.382Z" fill="#253B80"/><path d="M39.638 14.671a23.1 23.1 0 0 1-.136.766c-1.752 8.839-7.744 11.892-15.398 11.892h-3.897c-.936 0-1.725.668-1.87 1.576L16.34 41.342l-.565 3.525A.986.986 0 0 0 16.76 46h6.912c.818 0 1.514-.585 1.642-1.378l.069-.345 1.3-8.117.084-.445a1.654 1.654 0 0 1 1.643-1.38h1.034c6.696 0 11.939-2.673 13.47-10.406.64-3.23.31-5.927-1.384-7.824-.513-.572-1.149-1.047-1.892-1.434Z" fill="#179BD7"/><path d="M37.804 13.953a13.964 13.964 0 0 0-1.704-.372 22.002 22.002 0 0 0-3.435-.246h-10.41c-.257 0-.5.057-.719.16-.48.227-.837.673-.923 1.22l-2.215 13.787-.064.402a1.883 1.883 0 0 1 1.871-1.575h3.897c7.654 0 13.647-3.055 15.398-11.893.053-.261.097-.516.136-.765a9.436 9.436 0 0 0-1.44-.597 12.1 12.1 0 0 0-.392-.121Z" fill="#222D65"/><path d="M20.614 14.715a1.63 1.63 0 0 1 .923-1.219c.22-.103.462-.16.718-.16h10.411c1.233 0 2.385.08 3.435.246a14.023 14.023 0 0 1 2.097.491c.517.169.998.368 1.44.598.522-3.267-.003-5.49-1.8-7.505C35.855 4.95 32.28 4 27.706 4H14.423c-.935 0-1.732.668-1.876 1.577L7.014 40.044a1.128 1.128 0 0 0 1.126 1.297h8.2l2.06-12.839 2.214-13.787Z" fill="#253B80"/></svg>

```

## File: static\src\js\payment_button.js

```javascript
/** @odoo-module **/

import paymentButton from '@payment/js/payment_button';

paymentButton.include({

    /**
     * Hide the disabled PayPal button and show the enabled one.
     *
     * @override method from @payment/js/payment_button
     * @private
     * @return {void}
     */
    _setEnabled() {
        if (!this.paymentButton.dataset.isPaypal) {
            this._super();
            return;
        }

        document.getElementById('o_paypal_disabled_button').classList.add('d-none');
        document.getElementById('o_paypal_enabled_button').classList.remove('d-none');
    },

    /**
     * Hide the enabled PayPal button and show the disabled one.
     *
     * @override method from @payment/js/payment_button
     * @private
     * @return {void}
     */
    _disable() {
        if (!this.paymentButton.dataset.isPaypal) {
            this._super();
            return;
        }

        document.getElementById('o_paypal_disabled_button').classList.remove('d-none');
        document.getElementById('o_paypal_enabled_button').classList.add('d-none');
    },

    /**
     * Disable the generic behavior that would hide the Paypal button container.
     *
     * @override method from @payment/js/payment_button
     * @private
     * @return {void}
     */
    _hide() {
        if (!this.paymentButton.dataset.isPaypal) {
            this._super();
        }
    },

    /**
     * Disable the generic behavior that would show the Paypal button container.
     *
     * @override method from @payment/js/payment_button
     * @private
     * @return {void}
     */
    _show() {
        if (!this.paymentButton.dataset.isPaypal) {
            this._super();
        }
    },

});

```

## File: static\src\js\payment_form.js

```javascript
/* global paypal */

import { loadJS } from '@web/core/assets';
import { _t } from '@web/core/l10n/translation';
import { rpc, RPCError } from '@web/core/network/rpc';

import paymentForm from '@payment/js/payment_form';

paymentForm.include({
    inlineFormValues: undefined,
    paypalColor: 'blue',
    selectedOptionId: undefined,
    paypalData: undefined,

    // #=== DOM MANIPULATION ===#

    /**
     * Hides paypal button container if the expanded inline form is another provider.
     *
     * @private
     * @param {HTMLInputElement} radio - The radio button linked to the payment option.
     * @return {void}
     */
    async _expandInlineForm(radio) {
        const providerCode = this._getProviderCode(radio);
        if (providerCode !== 'paypal') {
            document.getElementById('o_paypal_button')?.classList.add('d-none'); // TODO Compatibility layer; to remove in master.
            document.getElementById('o_paypal_button_container')?.classList.add('d-none');
        }
        this._super(...arguments);
    },

    /**
     * Prepare the inline form of Paypal for direct payment.
     *
     * The PayPal SDK creates payment buttons based on the client_id and the currency of the order.
     *
     * Two payment buttons are created: one enabled and one disabled. The enabled button is shown
     * when the user is allowed to click on it, and the disabled button is shown otherwise. This
     * trick is necessary as the PayPal SDK does not provide a way to disable the button after it
     * has been created.
     *
     * The created buttons are saved and reused when switching between different payment methods to
     * avoid recreating the buttons.
     *
     * @override method from @payment/js/payment_form
     * @private
     * @param {number} providerId - The id of the selected payment option's provider.
     * @param {string} providerCode - The code of the selected payment option's provider.
     * @param {number} paymentOptionId - The id of the selected payment option
     * @param {string} paymentMethodCode - The code of the selected payment method, if any.
     * @param {string} flow - The online payment flow of the selected payment option.
     * @return {void}
     */
    async _prepareInlineForm(providerId, providerCode, paymentOptionId, paymentMethodCode, flow) {
        if (providerCode !== 'paypal') {
            this._super(...arguments);
            return;
        }

        this._hideInputs();
        this._setPaymentFlow('direct');
        document.getElementById('o_paypal_loading').classList.remove('d-none');
        // Check if instantiation of the component is needed.
        this.paypalData ??= {}; // Store the component of each instantiated payment method.
        if (this.selectedOptionId && this.selectedOptionId !== paymentOptionId) {
            this.paypalData[this.selectedOptionId]['enabledButton'].hide()
            this.paypalData[this.selectedOptionId]['disabledButton']?.hide() // TODO Compatibility layer; remove the ? in master.
        }
        const currentPayPalData = this.paypalData[paymentOptionId]
        if (currentPayPalData && this.selectedOptionId !== paymentOptionId) {
            const paypalSDKURL = this.paypalData[paymentOptionId]['sdkURL']
            const enabledButton = this.paypalData[paymentOptionId]['enabledButton']
            const disabledButton = this.paypalData[paymentOptionId]['disabledButton']
            await loadJS(paypalSDKURL);
            enabledButton?.show();
            disabledButton?.show(); // TODO Compatibility layer; remove the ? in master.
        }
        else if (!currentPayPalData) {
            this.paypalData[paymentOptionId] = {}
            const radio = document.querySelector('input[name="o_payment_radio"]:checked');
            if (radio) {
                this.inlineFormValues = JSON.parse(radio.dataset['paypalInlineFormValues']);
                this.paypalColor = radio.dataset['paypalColor']
            }

            // https://developer.paypal.com/sdk/js/configuration/#link-queryparameters
            const { client_id, currency_code } = this.inlineFormValues
            const paypalSDKURL = `https://www.paypal.com/sdk/js?client-id=${
                client_id}&components=buttons&currency=${currency_code}&intent=capture`
            this.paypalData[paymentOptionId]['sdkURL'] = paypalSDKURL;
            await loadJS(paypalSDKURL);

            // Create the two PayPal buttons. See https://developer.paypal.com/sdk/js/reference.
            const enabledButton = paypal.Buttons({
                fundingSource: paypal.FUNDING.PAYPAL,
                style: { // https://developer.paypal.com/sdk/js/reference/#link-style
                    color: this.paypalColor,
                    label: 'paypal',
                    disableMaxWidth: true,
                    borderRadius: 6,
                },
                createOrder: this._paypalOnClick.bind(this),
                onApprove: this._paypalOnApprove.bind(this),
                onCancel: this._paypalOnCancel.bind(this),
                onError: this._paypalOnError.bind(this),
            });
            const enabledButtonContainer = document.getElementById('o_paypal_enabled_button');
            if (enabledButtonContainer) {
                enabledButton.render('#o_paypal_enabled_button');
            } else {
                enabledButton.render('#o_paypal_button');  // TODO Compatibility layer; to remove in master.
            }
            this.paypalData[paymentOptionId]['enabledButton'] = enabledButton;

            const disabledButtonContainer = document.getElementById('o_paypal_disabled_button');
            if (disabledButtonContainer) { // TODO Compatibility layer; to remove in master.
                const disabledButton = paypal.Buttons({
                    fundingSource: paypal.FUNDING.PAYPAL,
                    style: { // https://developer.paypal.com/sdk/js/reference/#link-style
                        color: 'silver',
                        label: 'paypal',
                        disableMaxWidth: true,
                        borderRadius: 6,
                    },
                    onInit: (data, actions) => actions.disable(),  // Permanently disable the button.
                });
                disabledButton.render('#o_paypal_disabled_button');
                this.paypalData[paymentOptionId]['disabledButton'] = disabledButton;
            }
        }
        document.getElementById('o_paypal_loading').classList.add('d-none');
        document.getElementById('o_paypal_button')?.classList.remove('d-none'); // TODO Compatibility layer; to remove in master.
        document.getElementById('o_paypal_button_container')?.classList.remove('d-none');  // TODO Compatibility layer; remove the ? in master.
        this.selectedOptionId = paymentOptionId;
    },

    // #=== PAYMENT FLOW ===#

    /**
     * Handle the click event of the component and initiate the payment.
     *
     * @private
     * @return {void}
     */
    async _paypalOnClick() {
        await this._submitForm(new Event("PayPalClickEvent"));
        return this.paypalData[this.selectedOptionId].paypalOrderId;
    },

    _processDirectFlow(providerCode, paymentOptionId, paymentMethodCode, processingValues) {
        if (providerCode !== 'paypal') {
            this._super(...arguments);
            return;
        }
        this.paypalData[paymentOptionId].paypalOrderId = processingValues['order_id'];
        this.paypalData[paymentOptionId].paypalTxRef = processingValues['reference'];
    },

    /**
     * Handle the approval event of the component and complete the payment.
     *
     * @private
     * @param {object} data - The data returned by PayPal on approving the order.
     * @return {void}
     */
    async _paypalOnApprove(data) {
        const orderID = data.orderID;
        const { provider_id } = this.inlineFormValues

        await rpc('/payment/paypal/complete_order', {
            'provider_id': provider_id,
            'order_id': orderID,
            'reference': this.paypalData[this.selectedOptionId].paypalTxRef,
        }).then(() => {
            // Close the PayPal buttons that were rendered
            this.paypalData[this.selectedOptionId]['enabledButton'].close();
            window.location = '/payment/status';
        }).catch(error => {
            if (error instanceof RPCError) {
                this._displayErrorDialog(_t("Payment processing failed"), error.data.message);
                this._enableButton(); // The button has been disabled before initiating the flow.
            }
            return Promise.reject(error);
        })
    },

    /**
     * Handle the cancel event of the component.
     * @private
     * @return {void}
     */
    _paypalOnCancel() {
        this._enableButton();
    },

    /**
     * Handle the error event of the component.
     * @private
     * @param {object} error - The error in the component.
     * @return {void}
     */
    _paypalOnError(error) {
        const message = error.message
        this._enableButton();
        // Paypal throws an error if the popup is closed before it can load;
        // this case should be treated as an onCancel event.
        if (message !== "Detected popup close" && !(error instanceof RPCError)) {
            this._displayErrorDialog(_t("Payment processing failed"), message);
        }
    },
});

```

## File: views\payment_form_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="payment_paypal.method_form" inherit_id="payment.method_form">
        <input name="o_payment_radio" position="attributes">
            <attribute name="t-att-data-paypal-inline-form-values">
                provider_sudo._paypal_get_inline_form_values(currency)
            </attribute>
            <attribute name="t-att-data-paypal-color">
                "blue"
            </attribute>
        </input>
    </template>

    <template id="payment_submit_button_inherit" inherit_id="payment.submit_button">
        <button name="o_payment_submit_button" position="before">
            <div
                id="o_paypal_button_container" name="o_payment_submit_button" data-is-paypal="true"
            >
                <div id="o_paypal_enabled_button" class="d-none"/>
                <div id="o_paypal_disabled_button"/>
            </div>
            <div id="o_paypal_loading" class="d-flex justify-content-center d-none">
                <div class="spinner-border"/>
            </div>
        </button>
    </template>

</odoo>

```

## File: views\payment_provider_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_provider_form" model="ir.ui.view">
        <field name="name">PayPal Provider Form</field>
        <field name="model">payment.provider</field>
        <field name="inherit_id" ref="payment.payment_provider_form"/>
        <field name="arch" type="xml">
            <group name="provider_credentials" position='inside'>
                <group invisible="code != 'paypal'">
                    <field
                        name="paypal_email_account"
                        required="code == 'paypal' and state != 'disabled'"
                    />
                    <field
                        name="paypal_client_id"
                        string="Client ID"
                        required="code == 'paypal' and state != 'disabled'"
                    />
                    <field
                        name="paypal_client_secret"
                        string="Client Secret"
                        password="True"
                        required="code == 'paypal' and state != 'disabled'"
                    />
                    <label for="paypal_webhook_id" string="Webhook ID"/>
                    <div class="o_row" col="2">
                        <field name="paypal_webhook_id"/>
                        <button
                            string="Generate your webhook"
                            type="object"
                            name="action_paypal_create_webhook"
                            class="btn-primary"
                            invisible="paypal_webhook_id"
                        />
                    </div>
                    <widget
                        name="documentation_link"
                        path="/applications/finance/payment_providers/paypal.html"
                        label="How to configure your paypal account?"
                        colspan="2"
                    />
                </group>
            </group>
        </field>
    </record>

</odoo>

```

## File: views\payment_transaction_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_transaction_form" model="ir.ui.view">
        <field name="name">PayPal Transaction Form</field>
        <field name="model">payment.transaction</field>
        <field name="inherit_id" ref="payment.payment_transaction_form"/>
        <field name="arch" type="xml">
            <field name="provider_reference" position="after">
                <field name="paypal_type"
                       readonly="1"
                       invisible="provider_code != 'paypal'"
                       groups="base.group_no_one"/>
            </field>
        </field>
    </record>

</odoo>

```

