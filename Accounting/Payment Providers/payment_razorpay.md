# Odoo Module: payment_razorpay

Category: Accounting/Payment Providers

This file contains the source code of the Odoo module.

## File: const.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# The currencies supported by Razorpay, in ISO 4217 format. Last updated on May 26, 2021.
# See https://razorpay.com/docs/payments/payments/international-payments/#supported-currencies.
# Last seen online: 16 November 2022.
SUPPORTED_CURRENCIES = [
    'AED',
    'ALL',
    'AMD',
    'ARS',
    'AUD',
    'AWG',
    'BBD',
    'BDT',
    'BMD',
    'BND',
    'BOB',
    'BSD',
    'BWP',
    'BZD',
    'CAD',
    'CHF',
    'CNY',
    'COP',
    'CRC',
    'CUP',
    'CZK',
    'DKK',
    'DOP',
    'DZD',
    'EGP',
    'ETB',
    'EUR',
    'FJD',
    'GBP',
    'GHS',
    'GIP',
    'GMD',
    'GTQ',
    'GYD',
    'HKD',
    'HNL',
    'HRK',
    'HTG',
    'HUF',
    'IDR',
    'ILS',
    'INR',
    'JMD',
    'KES',
    'KGS',
    'KHR',
    'KYD',
    'KZT',
    'LAK',
    'LKR',
    'LRD',
    'LSL',
    'MAD',
    'MDL',
    'MKD',
    'MMK',
    'MNT',
    'MOP',
    'MUR',
    'MVR',
    'MWK',
    'MXN',
    'MYR',
    'NAD',
    'NGN',
    'NIO',
    'NOK',
    'NPR',
    'NZD',
    'PEN',
    'PGK',
    'PHP',
    'PKR',
    'QAR',
    'RUB',
    'SAR',
    'SCR',
    'SEK',
    'SGD',
    'SLL',
    'SOS',
    'SSP',
    'SVC',
    'SZL',
    'THB',
    'TTD',
    'TZS',
    'USD',
    'UYU',
    'UZS',
    'YER',
    'ZAR',
]

# The codes of the payment methods to activate when Razorpay is activated.
DEFAULT_PAYMENT_METHOD_CODES = {
    # Primary payment methods.
    'card',
    'netbanking',
    'upi',
    # Brand payment methods.
    'visa',
    'mastercard',
    'amex',
    'discover',
}

# The codes of payment methods that are not recognized by the orders API.
FALLBACK_PAYMENT_METHOD_CODES = {
    'wallets_india',
    'paylater_india',
    'emi_india',
}

# Mapping of payment method codes to Razorpay codes.
PAYMENT_METHODS_MAPPING = {
    'wallets_india': 'wallet',
    'paylater_india': 'paylater',
    'emi_india': 'emi',
}

# The maximum amount in INR that can be paid through an eMandate.
MANDATE_MAX_AMOUNT = {
    'card': 1000000,
    'upi': 100000,
}

# Mapping of transaction states to Razorpay's payment statuses.
# See https://razorpay.com/docs/payments/payments#payment-life-cycle.
PAYMENT_STATUS_MAPPING = {
    'pending': ('created', 'pending'),
    'authorized': ('authorized',),
    'done': ('captured', 'refunded', 'processed'),  # refunded is included to discard refunded txs.
    'error': ('failed',),
}

# Events that are handled by the webhook.
HANDLED_WEBHOOK_EVENTS = [
    'payment.authorized',
    'payment.captured',
    'payment.failed',
    'refund.failed',
    'refund.processed',
]

```

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

from odoo.addons.payment import setup_provider, reset_payment_provider


def post_init_hook(env):
    setup_provider(env, 'razorpay')


def uninstall_hook(env):
    reset_payment_provider(env, 'razorpay')

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': "Payment Provider: Razorpay",
    'version': '1.0',
    'category': 'Accounting/Payment Providers',
    'sequence': 350,
    'summary': "A payment provider covering India.",
    'description': " ",  # Non-empty string to avoid loading the README file.
    'depends': ['payment'],
    'data': [
        'views/payment_provider_views.xml',
        'views/payment_razorpay_templates.xml',

        'data/payment_provider_data.xml',  # Depends on views/payment_razorpay_templates.xml
    ],
    'assets': {
        'web.assets_frontend': [
            'payment_razorpay/static/src/js/payment_form.js',
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
import logging
import pprint

from werkzeug.exceptions import Forbidden

from odoo import http
from odoo.exceptions import ValidationError
from odoo.http import request

from odoo.addons.payment_razorpay.const import HANDLED_WEBHOOK_EVENTS


_logger = logging.getLogger(__name__)


class RazorpayController(http.Controller):
    _webhook_url = '/payment/razorpay/webhook'

    @http.route(_webhook_url, type='http', methods=['POST'], auth='public', csrf=False)
    def razorpay_webhook(self):
        """ Process the notification data sent by Razorpay to the webhook.

        :return: An empty string to acknowledge the notification.
        :rtype: str
        """
        data = request.get_json_data()
        _logger.info("Notification received from Razorpay with data:\n%s", pprint.pformat(data))

        event_type = data['event']
        if event_type in HANDLED_WEBHOOK_EVENTS:
            entity_type = 'payment' if 'payment' in event_type else 'refund'
            try:
                entity_data = data['payload'].get(entity_type, {}).get('entity', {})
                entity_data.update(entity_type=entity_type)

                # Check the integrity of the event.
                received_signature = request.httprequest.headers.get('X-Razorpay-Signature')
                tx_sudo = request.env['payment.transaction'].sudo()._get_tx_from_notification_data(
                    'razorpay', entity_data
                )
                self._verify_notification_signature(
                    request.httprequest.data, received_signature, tx_sudo
                )

                # Handle the notification data.
                tx_sudo._handle_notification_data('razorpay', entity_data)
            except ValidationError:  # Acknowledge the notification to avoid getting spammed.
                _logger.exception("Unable to handle the notification data; skipping to acknowledge")
        return request.make_json_response('')

    @staticmethod
    def _verify_notification_signature(notification_data, received_signature, tx_sudo):
        """ Check that the received signature matches the expected one.

        :param dict|bytes notification_data: The notification data.
        :param str received_signature: The signature to compare with the expected signature.
        :param recordset tx_sudo: The sudoed transaction referenced by the notification data, as a
                                  `payment.transaction` record
        :return: None
        :raise :class:`werkzeug.exceptions.Forbidden`: If the signatures don't match.
        """
        # Check for the received signature.
        if not received_signature:
            _logger.warning("Received notification with missing signature.")
            raise Forbidden()

        # Compare the received signature with the expected signature.
        expected_signature = tx_sudo.provider_id._razorpay_calculate_signature(notification_data)
        if (
            expected_signature is None
            or not hmac.compare_digest(received_signature, expected_signature)
        ):
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
-- disable razorpay payment provider
UPDATE payment_provider
   SET razorpay_key_id = NULL,
       razorpay_key_secret = NULL,
       razorpay_webhook_secret = NULL;

```

## File: data\payment_provider_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="payment.payment_provider_razorpay" model="payment.provider">
        <field name="code">razorpay</field>
        <field name="token_inline_form_view_id" ref="token_inline_form"/>
    </record>

</odoo>

```

## File: models\payment_provider.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import hashlib
import hmac
import logging
import pprint

import requests

from odoo import _, fields, models
from odoo.exceptions import ValidationError

from odoo.addons.payment_razorpay import const


_logger = logging.getLogger(__name__)


class PaymentProvider(models.Model):
    _inherit = 'payment.provider'

    code = fields.Selection(
        selection_add=[('razorpay', "Razorpay")], ondelete={'razorpay': 'set default'}
    )
    razorpay_key_id = fields.Char(
        string="Razorpay Key Id",
        help="The key solely used to identify the account with Razorpay.",
        required_if_provider='razorpay',
    )
    razorpay_key_secret = fields.Char(
        string="Razorpay Key Secret",
        required_if_provider='razorpay',
        groups='base.group_system',
    )
    razorpay_webhook_secret = fields.Char(
        string="Razorpay Webhook Secret",
        required_if_provider='razorpay',
        groups='base.group_system',
    )

    #=== COMPUTE METHODS ===#

    def _compute_feature_support_fields(self):
        """ Override of `payment` to enable additional features. """
        super()._compute_feature_support_fields()
        self.filtered(lambda p: p.code == 'razorpay').update({
            'support_manual_capture': 'full_only',
            'support_refund': 'partial',
            'support_tokenization': True,
        })

    # === BUSINESS METHODS - PAYMENT FLOW === #

    def _get_supported_currencies(self):
        """ Override of `payment` to return the supported currencies. """
        supported_currencies = super()._get_supported_currencies()
        if self.code == 'razorpay':
            supported_currencies = supported_currencies.filtered(
                lambda c: c.name in const.SUPPORTED_CURRENCIES
            )
        return supported_currencies

    def _razorpay_make_request(self, endpoint, payload=None, method='POST'):
        """ Make a request to Razorpay API at the specified endpoint.

        Note: self.ensure_one()

        :param str endpoint: The endpoint to be reached by the request.
        :param dict payload: The payload of the request.
        :param str method: The HTTP method of the request.
        :return The JSON-formatted content of the response.
        :rtype: dict
        :raise ValidationError: If an HTTP error occurs.
        """
        self.ensure_one()

        # TODO: Make api_version a kwarg in master.
        api_version = self.env.context.get('razorpay_api_version', 'v1')
        url = f'https://api.razorpay.com/{api_version}/{endpoint}'
        headers = None
        if access_token := self._razorpay_get_access_token():
            headers = {'Authorization': f'Bearer {access_token}'}
        auth = (self.razorpay_key_id, self.razorpay_key_secret) if self.razorpay_key_id else None
        try:
            if method == 'GET':
                response = requests.get(
                    url,
                    params=payload,
                    headers=headers,
                    auth=auth,
                    timeout=10,
                )
            else:
                response = requests.post(
                    url,
                    json=payload,
                    headers=headers,
                    auth=auth,
                    timeout=10,
                )
            try:
                response.raise_for_status()
            except requests.exceptions.HTTPError:
                _logger.exception(
                    "Invalid API request at %s with data:\n%s", url, pprint.pformat(payload),
                )
                raise ValidationError("Razorpay: " + _(
                    "Razorpay gave us the following information: '%s'",
                    response.json().get('error', {}).get('description')
                ))
        except (requests.exceptions.ConnectionError, requests.exceptions.Timeout):
            _logger.exception("Unable to reach endpoint at %s", url)
            raise ValidationError(
                "Razorpay: " + _("Could not establish the connection to the API.")
            )
        return response.json()

    def _razorpay_calculate_signature(self, data):
        """ Compute the signature for the request's data according to the Razorpay documentation.

        See https://razorpay.com/docs/webhooks/validate-test#validate-webhooks.

        :param bytes data: The data to sign.
        :return: The calculated signature.
        :rtype: str
        """
        secret = self.razorpay_webhook_secret
        if not secret:
            _logger.warning("Missing webhook secret; aborting signature calculation.")
            return None
        return hmac.new(secret.encode(), msg=data, digestmod=hashlib.sha256).hexdigest()

    def _get_default_payment_method_codes(self):
        """ Override of `payment` to return the default payment method codes. """
        default_codes = super()._get_default_payment_method_codes()
        if self.code != 'razorpay':
            return default_codes
        return const.DEFAULT_PAYMENT_METHOD_CODES

    def _get_validation_amount(self):
        """ Override of `payment` to return the amount for Razorpay validation operations.

        :return: The validation amount.
        :rtype: float
        """
        res = super()._get_validation_amount()
        if self.code != 'razorpay':
            return res

        return 1.0

    # === BUSINESS METHODS - OAUTH === #

    def _razorpay_get_public_token(self):  # TODO: remove in master
        self.ensure_one()
        return None

    def _razorpay_get_access_token(self):  # TODO: remove in master
        self.ensure_one()
        return None

```

## File: models\payment_token.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, models
from odoo.tools import float_round

from odoo.addons.payment_razorpay import const


class PaymentToken(models.Model):
    _inherit = 'payment.token'

    def _razorpay_get_limit_exceed_warning(self, amount, currency_id):
        """ Return a warning message when the maximum payment amount is exceeded.

        :param float amount: The amount to be paid.
        :param currency_id: The currency of the amount.
        :return: A warning message when the maximum payment amount is exceeded.
        :rtype: str
        """
        self.ensure_one()

        if not amount or self.provider_code != 'razorpay':
            return ""

        # Try to get the maximum amount based on the transaction from which this token was created.
        Transaction = self.env['payment.transaction']
        primary_tx = Transaction.search(
            [('token_id', '=', self.id), ('operation', 'not in', ['offline', 'online_token'])],
            limit=1,
        )
        if primary_tx:
            mandate_max_amount = primary_tx._razorpay_get_mandate_max_amount()
        else:  # Get the maximum amount based on the token's payment method code.
            pm = self.payment_method_id.primary_payment_method_id or self.payment_method_id
            mandate_max_amount_INR = const.MANDATE_MAX_AMOUNT.get(
                pm.code, const.MANDATE_MAX_AMOUNT['card']
            )
            mandate_max_amount = Transaction._razorpay_convert_inr_to_currency(
                mandate_max_amount_INR, currency_id
            )

        # Return the warning message if the amount exceeds the maximum amount; else an empty string.
        if amount > mandate_max_amount:
            return _(
                "You can not pay amounts greater than %(currency_symbol)s %(max_amount)s with this"
                " payment method",
                currency_symbol=currency_id.symbol,
                max_amount=float_round(mandate_max_amount, precision_digits=0),
            )
        return ""

```

## File: models\payment_transaction.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import pprint
import time
from datetime import datetime

from dateutil.relativedelta import relativedelta

from odoo import _, api, models
from odoo.exceptions import UserError, ValidationError

from odoo.addons.payment import utils as payment_utils
from odoo.addons.payment_razorpay import const


_logger = logging.getLogger(__name__)


class PaymentTransaction(models.Model):
    _inherit = 'payment.transaction'

    def _get_specific_processing_values(self, processing_values):
        """ Override of `payment` to return razorpay-specific processing values.

        Note: self.ensure_one() from `_get_processing_values`

        :param dict processing_values: The generic and specific processing values of the
                                       transaction.
        :return: The provider-specific processing values.
        :rtype: dict
        """
        res = super()._get_specific_processing_values(processing_values)
        if self.provider_code != 'razorpay':
            return res

        if self.operation in ('online_token', 'offline'):
            return {}

        customer_id = self._razorpay_create_customer()['id']
        order_id = self._razorpay_create_order(customer_id)['id']
        return {
            'razorpay_key_id': self.provider_id.razorpay_key_id,
            'razorpay_public_token': self.provider_id._razorpay_get_public_token(),
            'razorpay_customer_id': customer_id,
            'is_tokenize_request': self.tokenize,
            'razorpay_order_id': order_id,
        }

    def _razorpay_create_customer(self):
        """ Create and return a Customer object.

        :return: The created Customer.
        :rtype: dict
        """
        payload = {
            'name': self.partner_name,
            'email': self.partner_email or '',
            'contact': self.partner_phone and self._validate_phone_number(self.partner_phone) or '',
            'fail_existing': '0',  # Don't throw an error if the customer already exists.
        }
        _logger.info(
            "Sending '/customers' request for transaction with reference %s:\n%s",
            self.reference, pprint.pformat(payload)
        )
        customer_data = self.provider_id._razorpay_make_request('customers', payload=payload)
        _logger.info(
            "Response of '/customers' request for transaction with reference %s:\n%s",
            self.reference, pprint.pformat(customer_data)
        )
        return customer_data

    @api.model
    def _validate_phone_number(self, phone):
        """ Validate and format the phone number.

        :param str phone: The phone number to validate.
        :return str: The formatted phone number.
        :raise ValidationError: If the phone number is missing or incorrect.
        """
        if not phone and self.tokenize:
            raise ValidationError("Razorpay: " + _("The phone number is missing."))

        try:
            phone = self._phone_format(
                number=phone, country=self.partner_country_id, raise_exception=self.tokenize
            )
        except Exception:
            raise ValidationError("Razorpay: " + _("The phone number is invalid."))
        return phone

    def _razorpay_create_order(self, customer_id=None):
        """ Create and return an Order object to initiate the payment.

        :param str customer_id: The ID of the Customer object to assign to the Order for
                                non-subsequent payments.
        :return: The created Order.
        :rtype: dict
        """
        payload = self._razorpay_prepare_order_payload(customer_id=customer_id)
        _logger.info(
            "Sending '/orders' request for transaction with reference %s:\n%s",
            self.reference, pprint.pformat(payload)
        )
        order_data = self.provider_id._razorpay_make_request('orders', payload=payload)
        _logger.info(
            "Response of '/orders' request for transaction with reference %s:\n%s",
            self.reference, pprint.pformat(order_data)
        )
        return order_data

    def _razorpay_prepare_order_payload(self, customer_id=None):
        """ Prepare the payload for the order request based on the transaction values.

        :param str customer_id: The ID of the Customer object to assign to the Order for
                                non-subsequent payments.
        :return: The request payload.
        :rtype: dict
        """
        converted_amount = payment_utils.to_minor_currency_units(self.amount, self.currency_id)
        pm_code = (self.payment_method_id.primary_payment_method_id or self.payment_method_id).code
        payload = {
            'amount': converted_amount,
            'currency': self.currency_id.name,
            **({'method': pm_code} if pm_code not in const.FALLBACK_PAYMENT_METHOD_CODES else {}),
        }
        if self.operation in ['online_direct', 'validation']:
            payload['customer_id'] = customer_id  # Required for only non-subsequent payments.
            if self.tokenize:
                payload['token'] = {
                    'max_amount': payment_utils.to_minor_currency_units(
                        self._razorpay_get_mandate_max_amount(), self.currency_id
                    ),
                    'expire_at': time.mktime(
                        (datetime.today() + relativedelta(years=10)).timetuple()
                    ),  # Don't expire the token before at least 10 years.
                    'frequency': 'as_presented',
                }
        else:  # 'online_token', 'offline'
            # Required for only subsequent payments.
            payload['payment_capture'] = not self.provider_id.capture_manually
        if self.provider_id.capture_manually:  # The related payment must be only authorized.
            payload.update({
                'payment': {
                    'capture': 'manual',
                    'capture_options': {
                        'manual_expiry_period': 7200,  # The default value for this required option.
                        'refund_speed': 'normal',  # The default value for this required option.
                    }
                },
            })
        return payload

    def _razorpay_get_mandate_max_amount(self):
        """ Return the eMandate's maximum amount to define.

        :return: The eMandate's maximum amount.
        :rtype: float
        """
        pm_code = (
            self.payment_method_id.primary_payment_method_id or self.payment_method_id
        ).code
        pm_max_amount_INR = const.MANDATE_MAX_AMOUNT.get(pm_code, 100000)
        pm_max_amount = self._razorpay_convert_inr_to_currency(pm_max_amount_INR, self.currency_id)
        mandate_values = self._get_mandate_values()  # The linked document's values.
        if 'amount' in mandate_values and 'MRR' in mandate_values:
            max_amount = min(
                pm_max_amount, max(mandate_values['amount'] * 1.5, mandate_values['MRR'] * 5)
            )
        else:
            max_amount = pm_max_amount
        return max_amount

    @api.model
    def _razorpay_convert_inr_to_currency(self, amount, currency_id):
        """ Convert the amount from INR to the given currency.

        :param float amount: The amount to converted, in INR.
        :param currency_id: The currency to which the amount should be converted.
        :return: The converted amount in the given currency.
        :rtype: float
        """
        inr_currency = self.env['res.currency'].with_context(active_test=False).search([
            ('name', '=', 'INR'),
        ], limit=1)
        return inr_currency._convert(amount, currency_id)

    def _send_payment_request(self):
        """ Override of `payment` to send a payment request to Razorpay.

        Note: self.ensure_one()

        :return: None
        :raise UserError: If the transaction is not linked to a token.
        """
        super()._send_payment_request()
        if self.provider_code != 'razorpay':
            return

        if not self.token_id:
            raise UserError("Razorpay: " + _("The transaction is not linked to a token."))

        try:
            order_data = self._razorpay_create_order()
            phone = self._validate_phone_number(self.partner_phone)
            customer_id, token_id = self.token_id.provider_ref.split(',')
            payload = {
                'email': self.partner_email,
                'contact': phone,
                'amount': order_data['amount'],
                'currency': self.currency_id.name,
                'order_id': order_data['id'],
                'customer_id': customer_id,
                'token': token_id,
                'description': self.reference,
                'recurring': '1',
            }
            _logger.info(
                "Sending '/payments/create/recurring' request for transaction with reference %s:\n%s",
                self.reference, pprint.pformat(payload)
            )
            recurring_payment_data = self.provider_id._razorpay_make_request(
                'payments/create/recurring', payload=payload
            )
            _logger.info(
                "Response of '/payments/create/recurring' request for transaction with reference "
                "%s:\n%s", self.reference, pprint.pformat(recurring_payment_data)
            )
            self._handle_notification_data('razorpay', recurring_payment_data)
        except ValidationError as e:
            if self.operation == 'offline':
                self._set_error(str(e))
            else:
                raise

    def _send_refund_request(self, amount_to_refund=None):
        """ Override of `payment` to send a refund request to Razorpay.

        Note: self.ensure_one()

        :param float amount_to_refund: The amount to refund.
        :return: The refund transaction created to process the refund request.
        :rtype: recordset of `payment.transaction`
        """
        refund_tx = super()._send_refund_request(amount_to_refund=amount_to_refund)
        if self.provider_code != 'razorpay':
            return refund_tx

        # Make the refund request to Razorpay.
        converted_amount = payment_utils.to_minor_currency_units(
            -refund_tx.amount, refund_tx.currency_id
        )  # The amount is negative for refund transactions.
        payload = {
            'amount': converted_amount,
            'notes': {
                'reference': refund_tx.reference,  # Allow retrieving the ref. from webhook data.
            },
        }
        _logger.info(
            "Payload of '/payments/<id>/refund' request for transaction with reference %s:\n%s",
            self.reference, pprint.pformat(payload)
        )
        response_content = refund_tx.provider_id._razorpay_make_request(
            f'payments/{self.provider_reference}/refund', payload=payload
        )
        _logger.info(
            "Response of '/payments/<id>/refund' request for transaction with reference %s:\n%s",
            self.reference, pprint.pformat(response_content)
        )
        response_content.update(entity_type='refund')
        refund_tx._handle_notification_data('razorpay', response_content)

        return refund_tx

    def _send_capture_request(self, amount_to_capture=None):
        """ Override of `payment` to send a capture request to Razorpay. """
        child_capture_tx = super()._send_capture_request(amount_to_capture=amount_to_capture)
        if self.provider_code != 'razorpay':
            return child_capture_tx

        converted_amount = payment_utils.to_minor_currency_units(self.amount, self.currency_id)
        payload = {'amount': converted_amount, 'currency': self.currency_id.name}
        _logger.info(
            "Payload of '/payments/<id>/capture' request for transaction with reference %s:\n%s",
            self.reference, pprint.pformat(payload)
        )
        response_content = self.provider_id._razorpay_make_request(
            f'payments/{self.provider_reference}/capture', payload=payload
        )
        _logger.info(
            "Response of '/payments/<id>/capture' request for transaction with reference %s:\n%s",
            self.reference, pprint.pformat(response_content)
        )

        # Handle the capture request response.
        self._handle_notification_data('razorpay', response_content)

        return child_capture_tx

    def _send_void_request(self, amount_to_void=None):
        """ Override of `payment` to explain that it is impossible to void a Razorpay transaction.
        """
        child_void_tx = super()._send_void_request(amount_to_void=amount_to_void)
        if self.provider_code != 'razorpay':
            return child_void_tx

        raise UserError(_("Transactions processed by Razorpay can't be manually voided from Odoo."))

    def _get_tx_from_notification_data(self, provider_code, notification_data):
        """ Override of `payment` to find the transaction based on razorpay data.

        :param str provider_code: The code of the provider that handled the transaction
        :param dict notification_data: The normalized notification data sent by the provider
        :return: The transaction if found
        :rtype: recordset of `payment.transaction`
        :raise: ValidationError if the data match no transaction
        """
        tx = super()._get_tx_from_notification_data(provider_code, notification_data)
        if provider_code != 'razorpay' or len(tx) == 1:
            return tx

        entity_type = notification_data.get('entity_type', 'payment')
        if entity_type == 'payment':
            reference = notification_data.get('description')
            if not reference:
                raise ValidationError("Razorpay: " + _("Received data with missing reference."))
            tx = self.search([('reference', '=', reference), ('provider_code', '=', 'razorpay')])
        else:  # 'refund'
            notes = notification_data.get('notes')
            reference = isinstance(notes, dict) and notes.get('reference')
            if reference:  # The refund was initiated from Odoo.
                tx = self.search([('reference', '=', reference), ('provider_code', '=', 'razorpay')])
            else:  # The refund was initiated from Razorpay.
                # Find the source transaction based on its provider reference.
                source_tx = self.search([
                    ('provider_reference', '=', notification_data['payment_id']),
                    ('provider_code', '=', 'razorpay'),
                ])
                if source_tx:
                    # Manually create a refund transaction with a new reference.
                    tx = self._razorpay_create_refund_tx_from_notification_data(
                        source_tx, notification_data
                    )
                else:  # The refund was initiated for an unknown source transaction.
                    pass  # Don't do anything with the refund notification.
        if not tx:
            raise ValidationError(
                "Razorpay: " + _("No transaction found matching reference %s.", reference)
            )

        return tx

    def _razorpay_create_refund_tx_from_notification_data(self, source_tx, notification_data):
        """ Create a refund transaction based on Razorpay data.

        :param recordset source_tx: The source transaction for which a refund is initiated, as a
                                    `payment.transaction` recordset.
        :param dict notification_data: The notification data sent by the provider.
        :return: The newly created refund transaction.
        :rtype: recordset of `payment.transaction`
        :raise ValidationError: If inconsistent data were received.
        """
        refund_provider_reference = notification_data.get('id')
        amount_to_refund = notification_data.get('amount')
        if not refund_provider_reference or not amount_to_refund:
            raise ValidationError("Razorpay: " + _("Received incomplete refund data."))

        converted_amount = payment_utils.to_major_currency_units(
            amount_to_refund, source_tx.currency_id
        )
        return source_tx._create_child_transaction(
            converted_amount, is_refund=True, provider_reference=refund_provider_reference
        )

    def _process_notification_data(self, notification_data):
        """ Override of `payment` to process the transaction based on Razorpay data.

        Note: self.ensure_one()

        :param dict notification_data: The notification data sent by the provider
        :return: None
        """
        super()._process_notification_data(notification_data)
        if self.provider_code != 'razorpay':
            return

        if 'id' in notification_data:  # We have the full entity data (S2S request or webhook).
            entity_data = notification_data
        else:  # The payment data are not complete (Payments made by a token).
            # Fetch the full payment data.
            entity_data = self.provider_id._razorpay_make_request(
                f'payments/{notification_data["razorpay_payment_id"]}', method='GET'
            )
            _logger.info(
                "Response of '/payments' request for transaction with reference %s:\n%s",
                self.reference, pprint.pformat(entity_data)
            )

        # Update the provider reference.
        entity_id = entity_data.get('id')
        if not entity_id:
            raise ValidationError("Razorpay: " + _("Received data with missing entity id."))
        self.provider_reference = entity_id

        # Update the payment method.
        payment_method_type = entity_data.get('method', '')
        if payment_method_type == 'card':
            payment_method_type = entity_data.get('card', {}).get('network', '').lower()
        payment_method = self.env['payment.method']._get_from_code(
            payment_method_type, mapping=const.PAYMENT_METHODS_MAPPING
        )
        self.payment_method_id = payment_method or self.payment_method_id

        # Update the payment state.
        entity_status = entity_data.get('status')
        if not entity_status:
            raise ValidationError("Razorpay: " + _("Received data with missing status."))

        if entity_status in const.PAYMENT_STATUS_MAPPING['pending']:
            self._set_pending()
        elif entity_status in const.PAYMENT_STATUS_MAPPING['authorized']:
            if self.provider_id.capture_manually:
                self._set_authorized()
        elif entity_status in const.PAYMENT_STATUS_MAPPING['done']:
            if (
                not self.token_id
                and entity_data.get('token_id')
                and self.provider_id.allow_tokenization
            ):
                self._razorpay_tokenize_from_notification_data(entity_data)
            self._set_done()

            # Immediately post-process the transaction if it is a refund, as the post-processing
            # will not be triggered by a customer browsing the transaction from the portal.
            if self.operation == 'refund':
                self.env.ref('payment.cron_post_process_payment_tx')._trigger()
        elif entity_status in const.PAYMENT_STATUS_MAPPING['error']:
            _logger.warning(
                "The transaction with reference %s underwent an error. Reason: %s",
                self.reference, entity_data.get('error_description')
            )
            self._set_error(
                _("An error occurred during the processing of your payment. Please try again.")
            )
        else:  # Classify unsupported payment status as the `error` tx state.
            _logger.warning(
                "Received data for transaction with reference %s with invalid payment status: %s",
                self.reference, entity_status
            )
            self._set_error(
                "Razorpay: " + _("Received data with invalid status: %s", entity_status)
            )

    def _razorpay_tokenize_from_notification_data(self, notification_data):
        """ Create a new token based on the notification data.

        :param dict notification_data: The notification data built with Razorpay objects.
                                       See `_process_notification_data`.
        :return: None
        """
        pm_code = (self.payment_method_id.primary_payment_method_id or self.payment_method_id).code
        if pm_code == 'card':
            details = notification_data.get('card', {}).get('last4')
        elif pm_code == 'upi':
            temp_vpa = notification_data.get('vpa')
            details = temp_vpa[temp_vpa.find('@') - 1:]
        else:
            details = pm_code

        token = self.env['payment.token'].create({
            'provider_id': self.provider_id.id,
            'payment_method_id': self.payment_method_id.id,
            'payment_details': details,
            'partner_id': self.partner_id.id,
            # Razorpay requires both the customer ID and the token ID which are extracted from here.
            'provider_ref': f'{notification_data["customer_id"]},{notification_data["token_id"]}',
        })
        self.write({
            'token_id': token,
            'tokenize': False,
        })
        _logger.info(
            "Created token with id %(token_id)s for partner with id %(partner_id)s from "
            "transaction with reference %(ref)s",
            {
                'token_id': token.id,
                'partner_id': self.partner_id.id,
                'ref': self.reference,
            },
        )

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
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path fill-rule="evenodd" clip-rule="evenodd" d="m22.43 17.6-1.995 7.494 11.418-7.537-7.467 28.436 7.583.007L43 4 22.43 17.6Z" fill="#3395FF"/><path fill-rule="evenodd" clip-rule="evenodd" d="M10.14 34.046 7 46h15.544l6.36-24.32-18.765 12.366Z" fill="#072654"/><path d="M43.105 4h.972V.818h1.108V0H42v.818h1.105V4Zm2.775 0h.858V1.453h.053L47.663 4h.555l.871-2.547h.056V4H50V0h-1.108l-.924 2.714h-.05L46.99 0h-1.11v4Z" fill="#D1D5DB"/></svg>

```

## File: static\src\js\payment_form.js

```javascript
/** @odoo-module **/
/* global Razorpay */

import { _t } from '@web/core/l10n/translation';
import { loadJS } from '@web/core/assets';
import paymentForm from '@payment/js/payment_form';

paymentForm.include({

    // #=== DOM MANIPULATION ===#

    /**
     * Update the payment context to set the flow to 'direct'.
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
        if (providerCode !== 'razorpay') {
            this._super(...arguments);
            return;
        }

        if (flow === 'token') {
            return; // No need to update the flow for tokens.
        }

        // Overwrite the flow of the select payment method.
        this._setPaymentFlow('direct');
    },

    // #=== PAYMENT FLOW ===#

    async _processDirectFlow(providerCode, paymentOptionId, paymentMethodCode, processingValues) {
        if (providerCode !== 'razorpay') {
            this._super(...arguments);
            return;
        }
        const razorpayOptions = this._prepareRazorpayOptions(processingValues);
        await loadJS('https://checkout.razorpay.com/v1/checkout.js');
        const RazorpayJS = Razorpay(razorpayOptions);
        RazorpayJS.open();
        RazorpayJS.on('payment.failed', response => {
            this._displayErrorDialog(_t("Payment processing failed"), response.error.description);
        });
    },

    /**
     * Prepare the options to init the RazorPay SDK Object.
     *
     * @param {object} processingValues - The processing values.
     * @return {object}
     */
    _prepareRazorpayOptions(processingValues) {
        return Object.assign({}, processingValues, {
            'key': processingValues['razorpay_public_token'] || processingValues['razorpay_key_id'],
            'customer_id': processingValues['razorpay_customer_id'],
            'order_id': processingValues['razorpay_order_id'],
            'description': processingValues['reference'],
            'recurring': processingValues['is_tokenize_request'] ? '1': '0',
            'handler': response => {
                if (
                    response['razorpay_payment_id']
                    && response['razorpay_order_id']
                    && response['razorpay_signature']
                ) { // The payment reached a final state; redirect to the status page.
                    window.location = '/payment/status';
                }
            },
            'modal': {
                'ondismiss': () => {
                    window.location.reload();
                }
            },
        });
    },

});

```

## File: views\payment_provider_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_provider_form_razorpay" model="ir.ui.view">
        <field name="name">Razorpay Provider Form</field>
        <field name="model">payment.provider</field>
        <field name="inherit_id" ref="payment.payment_provider_form"/>
        <field name="arch" type="xml">
            <group name="provider_credentials" position='inside'>
                <group name="razorpay_credentials"
                       invisible="code != 'razorpay'">
                    <field name="razorpay_key_id"
                           string="Key Id"
                           required="code == 'razorpay' and state != 'disabled'"/>
                    <field name="razorpay_key_secret"
                           string="Key Secret"
                           required="code == 'razorpay' and state != 'disabled'"
                           password="True"/>
                    <field name="razorpay_webhook_secret"
                           string="Webhook Secret"
                           required="code == 'razorpay' and state != 'disabled'"
                           password="True"/>
                </group>
            </group>
            <field name="allow_tokenization" position="after">
                <div invisible="code != 'razorpay' or not allow_tokenization" colspan="2">
                    <widget class="mx-2" colspan="2" name="documentation_link" path="/applications/finance/payment_providers/razorpay.html#payment-providers-razorpay-recurring-payments" label="Enable recurring payments on Razorpay" icon="oi oi-fw o_button_icon oi-arrow-right"/>
                </div>
            </field>
        </field>
    </record>

</odoo>

```

## File: views\payment_razorpay_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="token_inline_form">
        <t t-set="warning" t-value="token_sudo._razorpay_get_limit_exceed_warning(amount, currency)"/>
        <div t-if="warning" t-out="warning" class="alert alert-danger mb-0"/>
    </template>

</odoo>

```

