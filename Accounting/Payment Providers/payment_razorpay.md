# Odoo Module: payment_razorpay

Category: Accounting/Payment Providers

This file contains the source code of the Odoo module.

## File: const.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# The currencies supported by Razorpay, in ISO 4217 format. Last updated on May 26, 2021.
# See https://razorpay.com/docs/payments/payments/international-payments/#supported-currencies.
SUPPORTED_CURRENCIES = [
    'AED', 'ALL', 'AMD', 'ARS', 'AUD', 'AWG', 'BBD', 'BDT', 'BMD', 'BND', 'BOB', 'BSD', 'BWP',
    'BZD', 'CAD', 'CHF', 'CNY', 'COP', 'CRC', 'CUP', 'CZK', 'DKK', 'DOP', 'DZD', 'EGP', 'ETB',
    'EUR', 'FJD', 'GBP', 'GHS', 'GIP', 'GMD', 'GTQ', 'GYD', 'HKD', 'HNL', 'HRK', 'HTG', 'HUF',
    'IDR', 'ILS', 'INR', 'JMD', 'KES', 'KGS', 'KHR', 'KYD', 'KZT', 'LAK', 'LBP', 'LKR', 'LRD',
    'LSL', 'MAD', 'MDL', 'MKD', 'MMK', 'MNT', 'MOP', 'MUR', 'MVR', 'MWK', 'MXN', 'MYR', 'NAD',
    'NGN', 'NIO', 'NOK', 'NPR', 'NZD', 'PEN', 'PGK', 'PHP', 'PKR', 'QAR', 'RUB', 'SAR', 'SCR',
    'SEK', 'SGD', 'SLL', 'SOS', 'SSP', 'SVC', 'SZL', 'THB', 'TTD', 'TZS', 'USD', 'UYU', 'UZS',
    'YER', 'ZAR',
]

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


def post_init_hook(cr, registry):
    setup_provider(cr, registry, 'razorpay')


def uninstall_hook(cr, registry):
    reset_payment_provider(cr, registry, 'razorpay')

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
    'application': False,
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
    _return_url = '/payment/razorpay/return'
    _webhook_url = '/payment/razorpay/webhook'

    @http.route(
        _return_url, type='http', auth='public', methods=['GET', 'POST'], csrf=False,
        save_session=False
    )
    def razorpay_return_from_checkout(self, reference, **data):
        """ Process the notification data sent by Razorpay after redirection from checkout.

        :param str reference: The transaction reference embedded in the return URL.
        :param dict data: The notification data.
        """
        _logger.info("Handling redirection from Razorpay with data:\n%s", pprint.pformat(data))
        if all(f'razorpay_{key}' in data for key in ('order_id', 'payment_id', 'signature')):
            # Check the integrity of the notification.
            tx_sudo = request.env['payment.transaction'].sudo()._get_tx_from_notification_data(
                'razorpay', {'description': reference}
            )  # Use the same key as for webhook notifications' data.
            self._verify_notification_signature(data, data.get('razorpay_signature'), tx_sudo)

            # Handle the notification data.
            tx_sudo._handle_notification_data('razorpay', data)
        else:  # The customer cancelled the payment or the payment failed.
            pass  # Don't try to process this case because the payment id was not provided.

        # Redirect the user to the status page.
        return request.redirect('/payment/status')

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
                    request.httprequest.data, received_signature, tx_sudo, is_redirect=False
                )

                # Handle the notification data.
                tx_sudo._handle_notification_data('razorpay', entity_data)
            except ValidationError:  # Acknowledge the notification to avoid getting spammed.
                _logger.exception("Unable to handle the notification data; skipping to acknowledge")
        return request.make_json_response('')

    @staticmethod
    def _verify_notification_signature(
        notification_data, received_signature, tx_sudo, is_redirect=True
    ):
        """ Check that the received signature matches the expected one.

        :param dict|bytes notification_data: The notification data.
        :param str received_signature: The signature to compare with the expected signature.
        :param recordset tx_sudo: The sudoed transaction referenced by the notification data, as a
                                  `payment.transaction` record
        :param bool is_redirect: Whether the notification data should be treated as redirect data
                                 or as coming from a webhook notification.
        :return: None
        :raise :class:`werkzeug.exceptions.Forbidden`: If the signatures don't match.
        """
        # Check for the received signature.
        if not received_signature:
            _logger.warning("Received notification with missing signature.")
            raise Forbidden()

        # Compare the received signature with the expected signature.
        expected_signature = tx_sudo.provider_id._razorpay_calculate_signature(
            notification_data, is_redirect=is_redirect
        )
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
        <field name="redirect_form_view_id" ref="redirect_form"/>
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
from werkzeug.urls import url_join

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError

from odoo.addons.payment_razorpay.const import SUPPORTED_CURRENCIES


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
            'support_manual_capture': True,
            'support_refund': 'partial',
        })

    # === BUSINESS METHODS ===#

    @api.model
    def _get_compatible_providers(self, *args, currency_id=None, **kwargs):
        """ Override of `payment` to filter out Razorpay providers for unsupported currencies. """
        providers = super()._get_compatible_providers(*args, currency_id=currency_id, **kwargs)

        currency = self.env['res.currency'].browse(currency_id).exists()
        if currency and currency.name not in SUPPORTED_CURRENCIES:
            providers = providers.filtered(lambda p: p.code != 'razorpay')

        return providers

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

        url = url_join('https://api.razorpay.com/v1/', endpoint)
        auth = (self.razorpay_key_id, self.razorpay_key_secret)
        try:
            if method == 'GET':
                response = requests.get(url, params=payload, auth=auth, timeout=10)
            else:
                response = requests.post(url, json=payload, auth=auth, timeout=10)
            try:
                response.raise_for_status()
            except requests.exceptions.HTTPError:
                _logger.exception(
                    "Invalid API request at %s with data:\n%s", url, pprint.pformat(payload),
                )
                raise ValidationError("Razorpay: " + _(
                    "The communication with the API failed. Razorpay gave us the following "
                    "information: '%s'", response.json().get('error', {}).get('description')
                ))
        except (requests.exceptions.ConnectionError, requests.exceptions.Timeout):
            _logger.exception("Unable to reach endpoint at %s", url)
            raise ValidationError(
                "Razorpay: " + _("Could not establish the connection to the API.")
            )
        return response.json()

    def _razorpay_calculate_signature(self, data, is_redirect=True):
        """ Compute the signature for the request's data according to the Razorpay documentation.

        See https://razorpay.com/docs/webhooks/validate-test#validate-webhooks and
        https://razorpay.com/docs/payments/payment-gateway/web-integration/hosted/build-integration.

        :param dict|bytes data: The data to sign.
        :param bool is_redirect: Whether the data should be treated as redirect data or as coming
                                 from a webhook notification.
        :return: The calculated signature.
        :rtype: str
        """
        if is_redirect:
            secret = self.razorpay_key_secret
            signing_string = f'{data["razorpay_order_id"]}|{data["razorpay_payment_id"]}'
            return hmac.new(
                secret.encode(), msg=signing_string.encode(), digestmod=hashlib.sha256
            ).hexdigest()
        else:  # Notification data.
            secret = self.razorpay_webhook_secret
            return hmac.new(secret.encode(), msg=data, digestmod=hashlib.sha256).hexdigest()

```

## File: models\payment_transaction.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import pprint

from werkzeug.urls import url_encode, url_join

from odoo import _, models
from odoo.exceptions import UserError, ValidationError

from odoo.addons.payment import utils as payment_utils
from odoo.addons.payment_razorpay.const import PAYMENT_STATUS_MAPPING
from odoo.addons.payment_razorpay.controllers.main import RazorpayController
from odoo.addons.phone_validation.tools.phone_validation import phone_sanitize_numbers


_logger = logging.getLogger(__name__)


class PaymentTransaction(models.Model):
    _inherit = 'payment.transaction'

    def _get_specific_rendering_values(self, processing_values):
        """ Override of `payment` to return razorpay-specific rendering values.

        Note: self.ensure_one() from `_get_processing_values`

        :param dict processing_values: The generic and specific processing values of the
                                       transaction.
        :return: The dict of provider-specific rendering values.
        :rtype: dict
        """
        res = super()._get_specific_rendering_values(processing_values)
        if self.provider_code != 'razorpay':
            return res

        # Initiate the payment and retrieve the related order id.
        payload = self._razorpay_prepare_order_request_payload()
        _logger.info(
            "Payload of '/orders' request for transaction with reference %s:\n%s",
            self.reference, pprint.pformat(payload)
        )
        order_data = self.provider_id._razorpay_make_request(endpoint='orders', payload=payload)
        _logger.info(
            "Response of '/orders' request for transaction with reference %s:\n%s",
            self.reference, pprint.pformat(order_data)
        )

        # Initiate the payment
        converted_amount = payment_utils.to_minor_currency_units(self.amount, self.currency_id)
        base_url = self.provider_id.get_base_url()
        return_url_params = {'reference': self.reference}

        phone = self.partner_phone
        error_message = _("The phone number is missing.")
        if phone:
            # sanitize partner phone
            country_code = self.partner_country_id.code
            country_phone_code = self.partner_country_id.phone_code
            phone_info = phone_sanitize_numbers([phone], country_code, country_phone_code)
            phone = phone_info[self.partner_phone]['sanitized']
            error_message = phone_info[self.partner_phone]['msg']
        if not phone:
            raise ValidationError("Razorpay: " + error_message)

        rendering_values = {
            'key_id': self.provider_id.razorpay_key_id,
            'name': self.company_id.name,
            'description': self.reference,
            'company_logo': url_join(base_url, f'web/image/res.company/{self.company_id.id}/logo'),
            'order_id': order_data['id'],
            'amount': converted_amount,
            'currency': self.currency_id.name,
            'partner_name': self.partner_name,
            'partner_email': self.partner_email,
            'partner_phone': phone,
            'return_url': url_join(
                base_url, f'{RazorpayController._return_url}?{url_encode(return_url_params)}'
            ),
        }
        return rendering_values

    def _razorpay_prepare_order_request_payload(self):
        """ Create the payload for the order request based on the transaction values.

        :return: The request payload.
        :rtype: dict
        """
        converted_amount = payment_utils.to_minor_currency_units(self.amount, self.currency_id)
        payload = {
            'amount': converted_amount,
            'currency': self.currency_id.name,
        }
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

    def _send_capture_request(self):
        """ Override of `payment` to send a capture request to Razorpay.

        Note: self.ensure_one()

        :return: None
        """
        super()._send_capture_request()
        if self.provider_code != 'razorpay':
            return

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

    def _send_void_request(self):
        """ Override of `payment` to explain that it is impossible to void a Razorpay transaction.

        Note: self.ensure_one()

        :return: None
        """
        super()._send_void_request()
        if self.provider_code != 'razorpay':
            return

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
        return source_tx._create_refund_transaction(
            amount_to_refund=converted_amount, provider_reference=refund_provider_reference
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
        else:  # The payment data are not complete (redirect from checkout).
            # Fetch the full payment data.
            entity_data = self.provider_id._razorpay_make_request(
                f'payments/{notification_data["razorpay_payment_id"]}', method='GET'
            )
            _logger.info(
                "Response of '/payments' request for transaction with reference %s:\n%s",
                self.reference, pprint.pformat(entity_data)
            )
        entity_id = entity_data.get('id')
        if not entity_id:
            raise ValidationError("Razorpay: " + _("Received data with missing entity id."))
        self.provider_reference = entity_id

        entity_status = entity_data.get('status')
        if not entity_status:
            raise ValidationError("Razorpay: " + _("Received data with missing status."))

        if entity_status in PAYMENT_STATUS_MAPPING['pending']:
            self._set_pending()
        elif entity_status in PAYMENT_STATUS_MAPPING['authorized']:
            self._set_authorized()
        elif entity_status in PAYMENT_STATUS_MAPPING['done']:
            self._set_done()

            # Immediately post-process the transaction if it is a refund, as the post-processing
            # will not be triggered by a customer browsing the transaction from the portal.
            if self.operation == 'refund':
                self.env.ref('payment.cron_post_process_payment_tx')._trigger()
        elif entity_status in PAYMENT_STATUS_MAPPING['error']:
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

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import payment_provider
from . import payment_transaction

```

## File: static\description\icon.svg

```svg
<svg width="70" height="70" viewBox="0 0 70 70" fill="none" xmlns="http://www.w3.org/2000/svg">
<mask id="mask0_10_362" style="mask-type:alpha" maskUnits="userSpaceOnUse" x="0" y="0" width="70" height="70">
<path d="M4 0H65C69 0 70 1 70 5V65C70 69 69 70 65 70H4C1 70 0 69 0 65V5C0 1 1 0 4 0Z" fill="white"/>
</mask>
<g mask="url(#mask0_10_362)">
<path fill-rule="evenodd" clip-rule="evenodd" d="M0 0H70V70H0V0Z" fill="url(#paint0_linear_10_362)"/>
<path fill-rule="evenodd" clip-rule="evenodd" d="M4 1H65C67.667 1 69.333 1.667 70 3V0H0V3C0.667 1.667 2 1 4 1Z" fill="white" fill-opacity="0.383"/>
<path fill-rule="evenodd" clip-rule="evenodd" d="M4 69H65C67.667 69 69.333 68 70 66V70H0V66C0.667 68 2 69 4 69Z" fill="black" fill-opacity="0.383"/>
</g>
<path opacity="0.324" fill-rule="evenodd" clip-rule="evenodd" d="M4 69C2 69 0 68 0 65V39.5C0 39.5 20 25.5 21.5 24.5L31 22.703H37.25L48 23L51.576 24.968L51.34 46.978L39.224 69H4Z" fill="#393939"/>
<path fill-rule="evenodd" clip-rule="evenodd" d="M21.5744 26.4129L20.4692 30.496L26.805 26.3918L22.6515 41.8793H26.8754L33 19" fill="#465C68"/>
<path fill-rule="evenodd" clip-rule="evenodd" d="M14.7459 35.3745L13 41.8793H21.6378L25.1718 28.6445L14.7529 35.3675" fill="#465C68"/>
<path d="M32.8154 31L31.0904 37.5H49.036V47.212C49.0363 47.3024 49.0008 47.3892 48.9373 47.4534C48.8737 47.5177 48.7874 47.5542 48.697 47.555H22.303C22.116 47.555 21.964 47.4 21.964 47.212V45H19.2568L19.25 47.555C19.25 49.069 20.464 50.297 21.964 50.297H49.036C50.537 50.297 51.75 49.069 51.75 47.555V27.445C51.75 25.931 50.536 24.703 49.036 24.703H34.4865L33.7588 27.445H48.697C48.7874 27.4458 48.8737 27.4823 48.9373 27.5466C49.0008 27.6108 49.0363 27.6976 49.036 27.788V31H32.8154Z" fill="#465C68"/>
<path d="M14.423 55.642H15.459V58.474H16.219V55.642H17.256V55H14.423V55.642Z" fill="#465C68"/>
<path d="M18.492 55H17.422V58.474H18.133V56.036H18.143L18.991 58.474H19.577L20.424 56.012H20.434V58.474H21.145V55H20.075L19.311 57.389H19.301L18.492 55Z" fill="#465C68"/>
<path fill-rule="evenodd" clip-rule="evenodd" d="M21.5744 24.4129L20.4692 28.496L26.805 24.3918L22.6515 39.8793H26.8754L33 17" fill="white"/>
<path fill-rule="evenodd" clip-rule="evenodd" d="M14.7459 33.3745L13 39.8793H21.6378L25.1718 26.6445L14.7529 33.3675" fill="white"/>
<path d="M32.8154 29L31.0904 35.5H49.036V45.212C49.0363 45.3024 49.0008 45.3892 48.9373 45.4534C48.8737 45.5177 48.7874 45.5542 48.697 45.555H22.303C22.116 45.555 21.964 45.4 21.964 45.212V43H19.2568L19.25 45.555C19.25 47.069 20.464 48.297 21.964 48.297H49.036C50.537 48.297 51.75 47.069 51.75 45.555V25.445C51.75 23.931 50.536 22.703 49.036 22.703H34.4865L33.7588 25.445H48.697C48.7874 25.4458 48.8737 25.4823 48.9373 25.5466C49.0008 25.6108 49.0363 25.6976 49.036 25.788V29H32.8154Z" fill="white"/>
<path d="M14.423 53.642H15.459V56.474H16.219V53.642H17.256V53H14.423V53.642Z" fill="white"/>
<path d="M18.492 53H17.422V56.474H18.133V54.036H18.143L18.991 56.474H19.577L20.424 54.012H20.434V56.474H21.145V53H20.075L19.311 55.389H19.301L18.492 53Z" fill="white"/>
<defs>
<linearGradient id="paint0_linear_10_362" x1="70" y1="0" x2="0" y2="70" gradientUnits="userSpaceOnUse">
<stop stop-color="#94B6C8"/>
<stop offset="1" stop-color="#6A9EBA"/>
</linearGradient>
</defs>
</svg>

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
                       attrs="{'invisible': [('code', '!=', 'razorpay')]}">
                    <field name="razorpay_key_id"
                           string="Key Id"
                           attrs="{'required': [('code', '=', 'razorpay'), ('state', '!=', 'disabled')]}"/>
                    <field name="razorpay_key_secret"
                           string="Key Secret"
                           attrs="{'required': [('code', '=', 'razorpay'), ('state', '!=', 'disabled')]}"
                           password="True"/>
                    <field name="razorpay_webhook_secret"
                           string="Webhook Secret"
                           attrs="{'required': [('code', '=', 'razorpay'), ('state', '!=', 'disabled')]}"
                           password="True"/>
                </group>
            </group>
        </field>
    </record>

</odoo>

```

## File: views\payment_razorpay_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="redirect_form">
        <form action="https://api.razorpay.com/v1/checkout/embedded" method="post">
            <input type="hidden" name="key_id" t-att-value="key_id"/>
            <input type="hidden" name="name" t-att-value="name"/>
            <input type="hidden" name="description" t-att-value="description"/>
            <input type="hidden" name="image" t-att-value="company_logo"/>
            <input type="hidden" name="order_id" t-att-value="order_id"/>
            <input type="hidden" name="amount" t-att-value="amount"/>
            <input type="hidden" name="currency" t-att-value="currency"/>
            <input type="hidden" name="prefill[name]" t-att-value="partner_name"/>
            <input type="hidden" name="prefill[email]" t-att-value="partner_email"/>
            <input type="hidden" name="prefill[contact]" t-att-value="partner_phone"/>
            <input type="hidden" name="callback_url" t-att-value="return_url"/>
            <input type="hidden" name="cancel_url" t-att-value="return_url"/>
        </form>
    </template>

</odoo>

```

