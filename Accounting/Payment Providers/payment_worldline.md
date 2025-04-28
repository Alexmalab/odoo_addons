# Odoo Module: payment_worldline

Category: Accounting/Payment Providers

This file contains the source code of the Odoo module.

## File: const.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# The codes of the payment methods to activate when Worldline is activated.
DEFAULT_PAYMENT_METHOD_CODES = {
    # Primary payment methods.
    'card',
}

# Mapping of payment method codes to Worldline codes.
# See https://docs.direct.worldline-solutions.com/en/payment-methods-and-features/index.
PAYMENT_METHODS_MAPPING = {
    'alipay_plus': 5405,
    'amex': 2,
    'bancontact': 3012,
    'bizum': 5001,
    'cartes_bancaires': 130,
    'cofidis': 3012,
    'diners': 132,
    'discover': 128,
    'eps': 5406,
    'floa_bank': 5139,
    'ideal': 809,
    'jcb': 125,
    'klarna': 3301,
    'maestro': 117,
    'mastercard': 3,
    'mbway': 5908,
    'multibanco': 5500,
    'p24': 3124,
    'paypal': 840,
    'post_finance_pay': 3203,
    'twint': 5407,
    'upi': 56,
    'visa': 1,
    'wechat_pay': 5404,
}

# The payment methods that involve a redirection to 3rd parties by Worldline.
REDIRECT_PAYMENT_METHODS = {
    'alipay_plus',
    'bizum',
    'eps',
    'floa_bank',
    'ideal',
    'klarna',
    'mbway',
    'multibanco',
    'p24',
    'paypal',
    'post_finance_pay',
    'twint',
    'wechat_pay',
}

# Mapping of transaction states to Worldline's payment statuses.
# See https://docs.direct.worldline-solutions.com/en/integration/api-developer-guide/statuses.
PAYMENT_STATUS_MAPPING = {
    'pending': (
        'CREATED', 'REDIRECTED', 'AUTHORIZATION_REQUESTED', 'PENDING_CAPTURE', 'CAPTURE_REQUESTED'
    ),
    'done': ('CAPTURED',),
    'cancel': ('CANCELLED',),
    'declined': ('REJECTED', 'REJECTED_CAPTURE'),
}

# Mapping of response codes indicating Worldline handled the request
# See https://apireference.connect.worldline-solutions.com/s2sapi/v1/en_US/json/response-codes.html.
VALID_RESPONSE_CODES = {
    200: 'Successful',
    201: 'Created',
    402: 'Payment Rejected',
}

```

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

from odoo.addons.payment import setup_provider, reset_payment_provider


def post_init_hook(env):
    setup_provider(env, 'worldline')


def uninstall_hook(env):
    reset_payment_provider(env, 'worldline')

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': "Payment Provider: Worldline",
    'category': 'Accounting/Payment Providers',
    'sequence': 350,
    'summary': "A French payment provider covering several European countries.",
    'description': " ",  # Non-empty string to avoid loading the README file.
    'depends': ['payment'],
    'data': [
        'views/payment_provider_views.xml',
        'views/payment_worldline_templates.xml',

        'data/payment_provider_data.xml',
    ],
    'assets': {
        'web.assets_frontend': [
            'payment_worldline/static/src/js/payment_form.js',
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

from base64 import b64encode
import hashlib
import hmac
import logging
import pprint

from werkzeug.exceptions import Forbidden

from odoo import http
from odoo.exceptions import ValidationError
from odoo.http import request


_logger = logging.getLogger(__name__)


class WorldlineController(http.Controller):
    _return_url = '/payment/worldline/return'
    _webhook_url = '/payment/worldline/webhook'

    @http.route(_return_url, type='http', auth='public', methods=['GET'])
    def worldline_return_from_checkout(self, **data):
        """ Process the notification data sent by Worldline after redirection.

        :param dict data: The notification data, including the provider id appended to the URL in
                          `_get_specific_rendering_values`.
        """
        _logger.info("Handling redirection from Worldline with data:\n%s", pprint.pformat(data))

        provider_id = int(data['provider_id'])
        provider_sudo = request.env['payment.provider'].sudo().browse(provider_id).exists()
        if not provider_sudo or provider_sudo.code != 'worldline':
            _logger.warning("Received payment data with invalid provider id.")
            raise Forbidden()

        # Fetch the checkout session data from Worldline.
        checkout_session_data = provider_sudo._worldline_make_request(
            f'hostedcheckouts/{data["hostedCheckoutId"]}', method='GET'
        )
        _logger.info(
            "Response of '/hostedcheckouts/<hostedCheckoutId>' request:\n%s",
            pprint.pformat(checkout_session_data)
        )
        notification_data = checkout_session_data.get('createdPaymentOutput', {})

        # Handle the notification data.
        tx_sudo = request.env['payment.transaction'].sudo()._get_tx_from_notification_data(
            'worldline', notification_data
        )
        tx_sudo._handle_notification_data('worldline', notification_data)
        return request.redirect('/payment/status')

    @http.route(_webhook_url, type='http', auth='public', methods=['POST'], csrf=False)
    def worldline_webhook(self):
        """ Process the notification data sent by Worldline to the webhook.

        See https://docs.direct.worldline-solutions.com/en/integration/api-developer-guide/webhooks.

        :return: An empty string to acknowledge the notification.
        :rtype: str
        """
        notification_data = request.get_json_data()
        _logger.info(
            "Notification received from Worldline with data:\n%s", pprint.pformat(notification_data)
        )
        try:
            # Check the integrity of the notification.
            tx_sudo = request.env['payment.transaction'].sudo()._get_tx_from_notification_data(
                'worldline', notification_data
            )
            received_signature = request.httprequest.headers.get('X-GCS-Signature')
            request_data = request.httprequest.data
            self._verify_notification_signature(request_data, received_signature, tx_sudo)

            # Handle the notification data.
            tx_sudo._handle_notification_data('worldline', notification_data)
        except ValidationError:  # Acknowledge the notification to avoid getting spammed.
            _logger.exception("Unable to handle the notification data; skipping to acknowledge.")

        return request.make_json_response('')  # Acknowledge the notification.

    @staticmethod
    def _verify_notification_signature(request_data, received_signature, tx_sudo):
        """ Check that the received signature matches the expected one.

        :param dict|bytes request_data: The request data.
        :param str received_signature: The signature to compare with the expected signature.
        :param payment.transaction tx_sudo: The sudoed transaction referenced by the notification
                                            data.
        :return: None
        :raise Forbidden: If the signatures don't match.
        """
        # Retrieve the received signature from the payload.
        if not received_signature:
            _logger.warning("Received notification with missing signature.")
            raise Forbidden()

        # Compare the received signature with the expected signature computed from the payload.
        webhook_secret = tx_sudo.provider_id.worldline_webhook_secret
        expected_signature = b64encode(
            hmac.new(webhook_secret.encode(), request_data, hashlib.sha256).digest()
        )
        if not hmac.compare_digest(received_signature.encode(), expected_signature):
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
-- disable worldline payment provider
UPDATE payment_provider
   SET worldline_pspid = NULL,
       worldline_api_key = NULL,
       worldline_api_secret = NULL,
       worldline_webhook_key = NULL,
       worldline_webhook_secret = NULL;

```

## File: data\payment_provider_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="payment.payment_provider_worldline" model="payment.provider">
        <field name="code">worldline</field>
        <field name="redirect_form_view_id" ref="redirect_form"/>
        <field name="allow_tokenization">True</field>
    </record>

</odoo>

```

## File: models\payment_provider.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
import hashlib
import hmac
import logging
import pprint
from wsgiref.handlers import format_date_time

import requests

from odoo import _, fields, models
from odoo.exceptions import ValidationError
from odoo.fields import Datetime

from odoo.addons.payment_worldline import const


_logger = logging.getLogger(__name__)


class PaymentProvider(models.Model):
    _inherit = 'payment.provider'

    code = fields.Selection(
        selection_add=[('worldline', "Worldline")], ondelete={'worldline': 'set default'}
    )
    worldline_pspid = fields.Char(string="Worldline PSPID", required_if_provider='worldline')
    worldline_api_key = fields.Char(string="Worldline API Key", required_if_provider='worldline')
    worldline_api_secret = fields.Char(
        string="Worldline API Secret", required_if_provider='worldline'
    )
    worldline_webhook_key = fields.Char(
        string="Worldline Webhook Key", required_if_provider='worldline'
    )
    worldline_webhook_secret = fields.Char(
        string="Worldline Webhook Secret", required_if_provider='worldline'
    )

    # === COMPUTE METHODS === #

    def _compute_feature_support_fields(self):
        """ Override of `payment` to enable additional features. """
        super()._compute_feature_support_fields()
        self.filtered(lambda p: p.code == 'worldline').update({
            'support_tokenization': True,
        })

    # === BUSINESS METHODS === #

    def _worldline_make_request(self, endpoint, payload=None, method='POST', idempotency_key=None):
        """ Make a request to Worldline API at the specified endpoint.

        Note: self.ensure_one()

        :param str endpoint: The endpoint to be reached by the request.
        :param dict payload: The payload of the request.
        :param str method: The HTTP method of the request.
        :param str idempotency_key: The idempotency key to pass in the request.
        :return: The JSON-formatted content of the response.
        :rtype: dict
        :raise ValidationError: If an HTTP error occurs.
        """
        self.ensure_one()

        api_url = self._worldline_get_api_url()
        url = f'{api_url}/v2/{self.worldline_pspid}/{endpoint}'
        content_type = 'application/json; charset=utf-8' if method == 'POST' else ''
        dt = format_date_time(Datetime.now().timestamp())  # Datetime in locale-independent RFC1123
        signature = self._worldline_calculate_signature(
            method, endpoint, content_type, dt, idempotency_key=idempotency_key
        )
        authorization_header = f'GCS v1HMAC:{self.worldline_api_key}:{signature}'
        headers = {
            'Authorization': authorization_header,
            'Date': dt,
            'Content-Type': content_type,
        }
        if method == 'POST' and idempotency_key:
            headers['X-GCS-Idempotence-Key'] = idempotency_key
        try:
            response = requests.request(method, url, json=payload, headers=headers, timeout=10)
            try:
                if response.status_code not in const.VALID_RESPONSE_CODES:
                    response.raise_for_status()
            except requests.exceptions.HTTPError:
                _logger.exception(
                    "Invalid API request at %s with data:\n%s", url, pprint.pformat(payload)
                )
                msg = ', '.join(
                    [error.get('message', '') for error in response.json().get('errors', [])]
                )
                raise ValidationError(
                    "Worldline: " + _("The communication with the API failed. Details: %s", msg)
                )
        except (requests.exceptions.ConnectionError, requests.exceptions.Timeout):
            _logger.exception("Unable to reach endpoint at %s", url)
            raise ValidationError(
                "Worldline: " + _("Could not establish the connection to the API.")
            )
        return response.json()

    def _worldline_get_api_url(self):
        """ Return the URL of the API corresponding to the provider's state.

        :return: The API URL.
        :rtype: str
        """
        if self.state == 'enabled':
            return 'https://payment.direct.worldline-solutions.com'
        else:  # 'test'
            return 'https://payment.preprod.direct.worldline-solutions.com'

    def _worldline_calculate_signature(
        self, method, endpoint, content_type, dt_rfc, idempotency_key=None
    ):
        """ Compute the signature for the provided data.

        See https://docs.direct.worldline-solutions.com/en/integration/api-developer-guide/authentication.

        :param str method: The HTTP method of the request
        :param str endpoint: The endpoint to be reached by the request.
        :param str content_type: The 'Content-Type' header of the request.
        :param datetime.datetime dt_rfc: The timestamp of the request, in RFC1123 format.
        :param str idempotency_key: The idempotency key to pass in the request.
        :return: The calculated signature.
        :rtype: str
        """
        # specific order required: method, content_type, date, custom headers, endpoint
        values_to_sign = [method, content_type, dt_rfc]
        if idempotency_key:
            values_to_sign.append(f'x-gcs-idempotence-key:{idempotency_key}')
        values_to_sign.append(f'/v2/{self.worldline_pspid}/{endpoint}')

        signing_str = '\n'.join(values_to_sign) + '\n'
        signature = hmac.new(
            self.worldline_api_secret.encode(), signing_str.encode(), hashlib.sha256
        )
        return base64.b64encode(signature.digest()).decode('utf-8')

    def _get_default_payment_method_codes(self):
        """ Override of `payment` to return the default payment method codes. """
        default_codes = super()._get_default_payment_method_codes()
        if self.code != 'worldline':
            return default_codes
        return const.DEFAULT_PAYMENT_METHOD_CODES

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
from odoo.addons.payment_worldline import const
from odoo.addons.payment_worldline.controllers.main import WorldlineController


_logger = logging.getLogger(__name__)


class PaymentTransaction(models.Model):
    _inherit = 'payment.transaction'

    def _compute_reference(self, provider_code, prefix=None, separator='-', **kwargs):
        """ Override of `payment` to ensure that Worldline requirement for references is satisfied.

        Worldline requires for references to be at most 30 characters long.

        :param str provider_code: The code of the provider handling the transaction.
        :param str prefix: The custom prefix used to compute the full reference.
        :param str separator: The custom separator used to separate the prefix from the suffix.
        :return: The unique reference for the transaction.
        :rtype: str
        """
        reference = super()._compute_reference(
            provider_code, prefix=prefix, separator=separator, **kwargs
        )
        if provider_code != 'worldline':
            return reference

        if len(reference) <= 30:  # Worldline transaction merchantReference is limited to 30 chars
            return reference

        prefix = payment_utils.singularize_reference_prefix(prefix='WL')
        return super()._compute_reference(
            provider_code, prefix=prefix, separator=separator, **kwargs
        )

    def _get_specific_processing_values(self, processing_values):
        """ Override of `payment` to redirect failed token-flow transactions.

        If the financial institution insists on user authentication,
        this override will reset the transaction, and switch the flow to redirect.

        Note: self.ensure_one() from `_get_processing_values`.

        :param dict processing_values: The generic processing values of the transaction.
        :return: The dict of provider-specific processing values.
        :rtype: dict
        """
        res = super()._get_specific_processing_values(processing_values)
        if (
            self.provider_code == 'worldline'
            and self.operation == 'online_token'
            and self.state == 'error'
            and self.state_message.endswith('AUTHORIZATION_REQUESTED')
        ):
            # Tokenized payment failed due to 3-D Secure authentication request.
            # Reset transaction to draft and switch to redirect flow.
            self.write({
                'state': 'draft',
                'operation': 'online_redirect',
            })
            res['force_flow'] = 'redirect'
        return res

    def _get_specific_rendering_values(self, processing_values):
        """ Override of `payment` to return Worldline-specific processing values.

        Note: self.ensure_one() from `_get_processing_values`.

        :param dict processing_values: The generic processing values of the transaction.
        :return: The dict of provider-specific processing values.
        :rtype: dict
        """
        res = super()._get_specific_rendering_values(processing_values)
        if self.provider_code != 'worldline':
            return res

        checkout_session_data = self._worldline_create_checkout_session()
        return {'api_url': checkout_session_data['redirectUrl']}

    def _worldline_create_checkout_session(self):
        """ Create a hosted checkout session and return the response data.

        :return: The hosted checkout session data.
        :rtype: dict
        """
        self.ensure_one()

        base_url = self.provider_id.get_base_url()
        return_route = WorldlineController._return_url
        return_url_params = urls.url_encode({'provider_id': str(self.provider_id.id)})
        return_url = f'{urls.url_join(base_url, return_route)}?{return_url_params}'
        first_name, last_name = payment_utils.split_partner_name(self.partner_name)
        payload = {
            'hostedCheckoutSpecificInput': {
                'locale': self.partner_lang or '',
                'returnUrl': return_url,
                'showResultPage': False,
            },
            'order': {
                'amountOfMoney': {
                    'amount': payment_utils.to_minor_currency_units(self.amount, self.currency_id),
                    'currencyCode': self.currency_id.name,
                },
                'customer': {  # required to create a token and for some redirected payment methods
                    'billingAddress': {
                        'city': self.partner_city or '',
                        'countryCode': self.partner_country_id.code or '',
                        'state': self.partner_state_id.name or '',
                        'street': self.partner_address or '',
                        'zip': self.partner_zip or '',
                    },
                    'contactDetails': {
                        'emailAddress': self.partner_email or '',
                        'phoneNumber': self.partner_phone or '',
                    },
                    'personalInformation': {
                        'name': {
                            'firstName': first_name or '',
                            'surname': last_name or '',
                        },
                    },
                },
                'references': {
                    'descriptor': self.reference,
                    'merchantReference': self.reference,
                },
            },
        }
        if self.payment_method_id.code in const.REDIRECT_PAYMENT_METHODS:
            payload['redirectPaymentMethodSpecificInput'] = {
                'requiresApproval': False,  # Force the capture.
                'paymentProductId': const.PAYMENT_METHODS_MAPPING[self.payment_method_id.code],
                'redirectionData': {
                    'returnUrl': return_url,
                },
            }
        else:
            payload['cardPaymentMethodSpecificInput'] = {
                'authorizationMode': 'SALE',  # Force the capture.
                'tokenize': self.tokenize,
            }
            if not self.payment_method_id.brand_ids and self.payment_method_id.code != 'card':
                worldline_code = const.PAYMENT_METHODS_MAPPING.get(self.payment_method_id.code, 0)
                payload['cardPaymentMethodSpecificInput']['paymentProductId'] = worldline_code
            else:
                pm_codes = self.env['payment.method'].search([
                    ('active', 'in', [True, False]),
                    ('primary_payment_method_id', '=', self.payment_method_id.id),
                ]).mapped('code')
                worldline_codes = [
                    const.PAYMENT_METHODS_MAPPING[code] for code in pm_codes
                    if code in const.PAYMENT_METHODS_MAPPING
                ]
                payload['hostedCheckoutSpecificInput']['paymentProductFilters'] = {
                    'restrictTo': {
                        'products': worldline_codes,
                    },
                }

        _logger.info(
            "Sending '/hostedcheckouts' request for transaction with reference %s:\n%s",
            self.reference, pprint.pformat(payload)
        )
        checkout_session_data = self.provider_id._worldline_make_request(
            'hostedcheckouts', payload=payload
        )
        _logger.info(
            "Response of '/hostedcheckouts' request for transaction with reference %s:\n%s",
            self.reference, pprint.pformat(checkout_session_data)
        )
        return checkout_session_data

    def _send_payment_request(self):
        """ Override of `payment` to send a payment request to Worldline.

        Note: self.ensure_one()

        :return: None
        :raise UserError: If the transaction is not linked to a token.
        """
        super()._send_payment_request()
        if self.provider_code != 'worldline':
            return

        # Prepare the payment request to Worldline.
        if not self.token_id:
            raise UserError("Worldline: " + _("The transaction is not linked to a token."))

        payload = {
            'cardPaymentMethodSpecificInput': {
                'authorizationMode': 'SALE',  # Force the capture.
                'token': self.token_id.provider_ref,
                'unscheduledCardOnFileRequestor': 'merchantInitiated',
                'unscheduledCardOnFileSequenceIndicator': 'subsequent',
            },
            'order': {
                'amountOfMoney': {
                    'amount': payment_utils.to_minor_currency_units(self.amount, self.currency_id),
                    'currencyCode': self.currency_id.name,
                },
                'references': {
                    'merchantReference': self.reference,
                },
            },
        }

        # Make the payment request to Worldline.
        response_content = self.provider_id._worldline_make_request(
            'payments',
            payload=payload,
            idempotency_key=payment_utils.generate_idempotency_key(
                self, scope='payment_request_token'
            )
        )

        # Handle the payment request response.
        _logger.info(
            "Response of /payment request for transaction with reference %s:\n%s",
            self.reference, pprint.pformat(response_content)
        )
        self._handle_notification_data('worldline', response_content)

    def _get_tx_from_notification_data(self, provider_code, notification_data):
        """ Override of `payment` to find the transaction based on Worldline data.

        :param str provider_code: The code of the provider that handled the transaction.
        :param dict notification_data: The notification data sent by the provider.
        :return: The transaction if found.
        :rtype: payment.transaction
        :raise ValidationError: If inconsistent data are received.
        :raise ValidationError: If the data match no transaction.
        """
        tx = super()._get_tx_from_notification_data(provider_code, notification_data)
        if provider_code != 'worldline' or len(tx) == 1:
            return tx

        # In case of failed payment, paymentResult could be given as a seperate key
        payment_result = notification_data.get('paymentResult', notification_data)
        payment_output = payment_result.get('payment', {}).get('paymentOutput', {})
        reference = payment_output.get('references', {}).get('merchantReference', '')
        if not reference:
            raise ValidationError(
                "Worldline: " + _("Received data with missing reference %(ref)s.", ref=reference)
            )

        tx = self.search([('reference', '=', reference), ('provider_code', '=', 'worldline')])
        if not tx:
            raise ValidationError(
                "Worldline: " + _("No transaction found matching reference %s.", reference)
            )

        return tx

    def _process_notification_data(self, notification_data):
        """ Override of `payment' to process the transaction based on Worldline data.

        Note: self.ensure_one()

        :param dict notification_data: The notification data sent by the provider.
        :return: None
        :raise ValidationError: If inconsistent data are received.
        """
        super()._process_notification_data(notification_data)
        if self.provider_code != 'worldline':
            return

        # In case of failed payment, paymentResult could be given as a seperate key
        payment_result = notification_data.get('paymentResult', notification_data)
        payment_data = payment_result.get('payment', {})

        # Update the provider reference.
        self.provider_reference = payment_data.get('id', '').rsplit('_', 1)[0]

        # Update the payment method.
        payment_output = payment_data.get('paymentOutput', {})
        if 'cardPaymentMethodSpecificOutput' in payment_output:
            payment_method_data = payment_output['cardPaymentMethodSpecificOutput']
        else:
            payment_method_data = payment_output.get('redirectPaymentMethodSpecificOutput', {})
        payment_method_code = payment_method_data.get('paymentProductId', '')
        payment_method = self.env['payment.method']._get_from_code(
            payment_method_code, mapping=const.PAYMENT_METHODS_MAPPING
        )
        self.payment_method_id = payment_method or self.payment_method_id

        # Update the payment state.
        status = payment_data.get('status')
        has_token_data = 'token' in payment_method_data
        if not status:
            raise ValidationError("Worldline: " + _("Received data with missing payment state."))

        if status in const.PAYMENT_STATUS_MAPPING['pending']:
            if status == 'AUTHORIZATION_REQUESTED':
                self._set_error("Worldline: " + status)
            elif self.operation == 'validation' \
                 and status in {'PENDING_CAPTURE', 'CAPTURE_REQUESTED'} \
                 and has_token_data:
                    self._worldline_tokenize_from_notification_data(payment_method_data)
                    self._set_done()
            else:
                self._set_pending()
        elif status in const.PAYMENT_STATUS_MAPPING['done']:
            if self.tokenize and has_token_data:
                self._worldline_tokenize_from_notification_data(payment_method_data)
            self._set_done()
        else:
            error_code = None
            if errors := payment_data.get('statusOutput', {}).get('errors'):
                error_code = errors[0].get('errorCode')
            if status in const.PAYMENT_STATUS_MAPPING['cancel']:
                self._set_canceled("Worldline: " + _(
                    "Transaction cancelled with error code %(error_code)s.",
                    error_code=error_code,
                ))
            elif status in const.PAYMENT_STATUS_MAPPING['declined']:
                self._set_error("Worldline: " + _(
                    "Transaction declined with error code %(error_code)s.",
                    error_code=error_code,
                ))
            else:  # Classify unsupported payment status as the `error` tx state.
                _logger.info(
                    "Received data with invalid payment status (%(status)s) for transaction with "
                    "reference %(ref)s.",
                    {'status': status, 'ref': self.reference},
                )
                self._set_error("Worldline: " + _(
                    "Received invalid transaction status %(status)s with error code "
                    "%(error_code)s.",
                    status=status,
                    error_code=error_code,
                ))

    def _worldline_tokenize_from_notification_data(self, pm_data):
        """ Create a new token based on the notification data.

        Note: self.ensure_one()

        :param dict pm_data: The payment method data sent by the provider
        :return: None
        """
        self.ensure_one()

        token = self.env['payment.token'].create({
            'provider_id': self.provider_id.id,
            'payment_method_id': self.payment_method_id.id,
            'payment_details': pm_data.get('card', {}).get('cardNumber', '')[-4:],  # Padded with *
            'partner_id': self.partner_id.id,
            'provider_ref': pm_data['token'],
        })
        self.write({
            'token_id': token,
            'tokenize': False,
        })
        _logger.info(
            "Created token with id %(token_id)s for partner with id %(partner_id)s from "
            "transaction with reference %(ref)s",
            {'token_id': token.id, 'partner_id': self.partner_id.id, 'ref': self.reference},
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
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M43.105 4h.972V.818h1.108V0H42v.818h1.105V4Zm2.775 0h.858V1.453h.053L47.663 4h.555l.871-2.547h.056V4H50V0h-1.108l-.924 2.714h-.05L46.99 0h-1.11v4Z" fill="#D1D5DB"/><path d="M8.5 15.976a25.287 25.287 0 0 0-2.51 2.576c3.759 3.831 6.471 9.677 7.384 16.446h3.616c-.729-5.964-2.747-11.473-5.862-15.866A25.981 25.981 0 0 0 8.5 15.976ZM4.094 21.11a25.45 25.45 0 0 0-1.843 3.373C4.403 27.205 5.987 30.854 6.709 35h3.63c-.686-4.454-2.268-8.557-4.628-11.878a21.782 21.782 0 0 0-1.617-2.013ZM0 35h3.64c-.524-2.525-1.437-4.874-2.698-6.908A25.664 25.664 0 0 0 0 35Zm39.508-20.525a24.706 24.706 0 0 0-1.452-.986c-.226.24-.45.48-.67.728-2.898 3.264-6.18 8.26-7.92 15.29a51.47 51.47 0 0 0-2.233-8.181c1.886-4.284 4.23-7.595 6.39-10.042a25.325 25.325 0 0 0-1.775-.594 40.475 40.475 0 0 0-5.487 8.422c-1.214-2.861-2.686-5.532-4.4-7.955-.28-.396-.564-.78-.853-1.157a24.419 24.419 0 0 0-3.864.933c2.992 3.512 5.433 7.964 7.105 13.04a45.547 45.547 0 0 0-1.565 5.779c-1.178-5.448-3.296-10.449-6.24-14.607a31.325 31.325 0 0 0-2.376-2.967 24.84 24.84 0 0 0-3.191 1.858c4.729 4.857 8.08 12.34 9.027 20.964h3.745c.28-2.923.794-5.6 1.475-8.043A51.214 51.214 0 0 1 26.616 35h3.618c1.07-9.432 5.028-15.783 8.43-19.615.28-.315.561-.617.843-.91Zm4.697 4.472a25.448 25.448 0 0 0-1.15-1.322c-3.842 3.993-7.055 9.786-8.058 17.373h1.738c1.026-7.365 4.18-12.37 6.905-15.437.187-.21.376-.414.565-.614ZM41.523 35h1.746c.823-4.566 2.675-7.912 4.479-10.22a24.954 24.954 0 0 0-.833-1.728c-2.503 2.967-4.55 6.95-5.392 11.947Zm8.127-4.09A18.584 18.584 0 0 0 48.11 35H50c0-1.393-.122-2.758-.351-4.09Z" fill="#33B6B4"/></svg>

```

## File: static\src\js\payment_form.js

```javascript
import PaymentForm from '@payment/js/payment_form';

PaymentForm.include({
    /**
     * Allow forcing redirect to 3-D Secure authentication for Ogone token flow.
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
        if (providerCode === 'worldline' && processingValues.force_flow === 'redirect') {
            delete processingValues.force_flow;
            this._processRedirectFlow(...arguments);
        } else {
            this._super(...arguments);
        }
    },
});

```

## File: views\payment_provider_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_provider_form" model="ir.ui.view">
        <field name="name">Worldline Provider Form</field>
        <field name="model">payment.provider</field>
        <field name="inherit_id" ref="payment.payment_provider_form"/>
        <field name="arch" type="xml">
            <group name="provider_credentials" position='inside'>
                <group invisible="code != 'worldline'">
                    <field
                        name="worldline_pspid"
                        string="PSPID"
                        required="code == 'worldline' and state != 'disabled'"
                    />
                    <field
                        name="worldline_api_key"
                        string="API Key"
                        required="code == 'worldline' and state != 'disabled'"
                    />
                    <field
                        name="worldline_api_secret"
                        string="API Secret"
                        required="code == 'worldline' and state != 'disabled'"
                        password="True"
                    />
                    <field
                        name="worldline_webhook_key"
                        string="Webhook Key"
                        required="code == 'worldline' and state != 'disabled'"
                    />
                    <field
                        name="worldline_webhook_secret"
                        string="Webhook Secret"
                        required="code == 'worldline' and state != 'disabled'"
                        password="True"
                    />
                </group>
            </group>
        </field>
    </record>

</odoo>

```

## File: views\payment_worldline_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="redirect_form">
        <form t-att-action="api_url" method="post"/>
    </template>

</odoo>

```

