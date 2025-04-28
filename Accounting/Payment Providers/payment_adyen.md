# Odoo Module: payment_adyen

Category: Accounting/Payment Providers

This file contains the source code of the Odoo module.

## File: const.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Endpoints of the API.
# See https://docs.adyen.com/api-explorer/#/CheckoutService/v70/overview for Checkout API
# See https://docs.adyen.com/api-explorer/#/Recurring/v68/overview for Recurring API
API_ENDPOINT_VERSIONS = {
    '/disable': 68,                 # Recurring API; unused - to remove
    '/paymentMethods': 71,          # Checkout API
    '/payments': 71,                # Checkout API
    '/payments/details': 71,        # Checkout API
    '/payments/{}/cancels': 71,     # Checkout API
    '/payments/{}/captures': 71,    # Checkout API
    '/payments/{}/refunds': 71,     # Checkout API
}

# Adyen-specific mapping of currency codes in ISO 4217 format to the number of decimals.
# Only currencies for which Adyen does not follow the ISO 4217 norm are listed here.
# See https://docs.adyen.com/development-resources/currency-codes
CURRENCY_DECIMALS = {
    'CLP': 2,
    'CVE': 0,
    'IDR': 0,
    'ISK': 2,
}

# The codes of the payment methods to activate when Adyen is activated.
DEFAULT_PAYMENT_METHODS_CODES = [
    # Primary payment methods.
    'card',
    # Brand payment methods.
    'visa',
    'mastercard',
    'amex',
    'discover',
]

# Mapping of payment method codes to Adyen codes.
PAYMENT_METHODS_MAPPING = {
    'ach_direct_debit': 'ach',
    'apple_pay': 'applepay',
    'bacs_direct_debit': 'directdebit_GB',
    'bancontact_card': 'bcmc',
    'bancontact': 'bcmc_mobile',
    'cash_app_pay': 'cashapp',
    'gopay': 'gopay_wallet',
    'becs_direct_debit': 'au_becs_debit',
    'afterpay': 'afterpaytouch',
    'klarna_pay_over_time': 'klarna_account',
    'momo': 'momo_wallet',
    'napas_card': 'momo_atm',
    'paytrail': 'ebanking_FI',
    'online_banking_czech_republic': 'onlineBanking_CZ',
    'online_banking_india': 'onlinebanking_IN',
    'fpx': 'molpay_ebanking_fpx_MY',
    'p24': 'onlineBanking_PL',
    'mastercard': 'mc',
    'online_banking_slovakia': 'onlineBanking_SK',
    'online_banking_thailand': 'molpay_ebanking_TH',
    'open_banking': 'paybybank',
    'samsung_pay': 'samsungpay',
    'sepa_direct_debit': 'sepadirectdebit',
    'sofort': 'directEbanking',
    'unionpay': 'cup',
    'wallets_india': 'wallet_IN',
    'wechat_pay': 'wechatpayQR',
}

# Mapping of transaction states to Adyen result codes.
# See https://docs.adyen.com/checkout/payment-result-codes for the exhaustive list of result codes.
RESULT_CODES_MAPPING = {
    'pending': (
        'ChallengeShopper', 'IdentifyShopper', 'Pending', 'PresentToShopper', 'Received',
        'RedirectShopper'
    ),
    'done': ('Authorised',),
    'cancel': ('Cancelled',),
    'error': ('Error',),
    'refused': ('Refused',),
}

```

## File: utils.py

```python
from odoo import _
from odoo.exceptions import ValidationError

from odoo.addons.payment import utils as payment_utils


def format_partner_name(partner_name):
    """ Format the partner name to comply with the payload structure of the API request.

    :param str partner_name: The name of the partner making the payment.
    :return: The formatted partner name.
    :rtype: dict
    """
    first_name, last_name = payment_utils.split_partner_name(partner_name)
    return {
        'firstName': first_name,
        'lastName': last_name,
    }


def include_partner_addresses(tx_sudo):
    """ Include the billing and delivery addresses of the related sales order to the payload of the
    API request.

    If no related sales order exists, the addresses are not included.

    Note: `self.ensure_one()`

    :param payment.transaction tx_sudo: The sudoed transaction of the payment.
    :return: The subset of the API payload that includes the billing and delivery addresses.
    :rtype: dict
    """
    tx_sudo.ensure_one()

    if 'sale_order_ids' in tx_sudo._fields:  # The module `sale` is installed.
        order = tx_sudo.sale_order_ids[:1]
        if order:
            return {
                'billingAddress': format_partner_address(order.partner_invoice_id),
                'deliveryAddress': format_partner_address(order.partner_shipping_id),
            }
    return {}


def format_partner_address(partner):
    """ Format the partner address to comply with the payload structure of the API request.

    :param res.partner partner: The partner making the payment.
    :return: The formatted partner address.
    :rtype: dict
    """
    street_data = partner._get_street_split()
    # Unlike what is stated in https://docs.adyen.com/risk-management/avs-checks/, not all fields
    # are required at all time. Thus, we fall back to 'Unknown' when a field is not set to avoid
    # blocking the payment (empty string are not accepted) or passing `False` (which may not pass
    # the fraud check).
    return {
        'city': partner.city or 'Unknown',
        'country': partner.country_id.code or 'ZZ',  # 'ZZ' if the country is not known.
        'stateOrProvince': partner.state_id.code or 'Unknown',  # The state is not always required.
        'postalCode': partner.zip or '',
        # Fill in the address fields if the format is supported, or fallback to the raw address.
        'street': street_data.get('street_name', partner.street) or 'Unknown',
        'houseNumberOrName': street_data.get('street_number') or '',
    }

```

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models
from . import wizards

from odoo.addons.payment import setup_provider, reset_payment_provider


def post_init_hook(env):
    setup_provider(env, 'adyen')


def uninstall_hook(env):
    reset_payment_provider(env, 'adyen')

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Payment Provider: Adyen',
    'version': '2.0',
    'category': 'Accounting/Payment Providers',
    'sequence': 350,
    'summary': "A Dutch payment provider covering Europe and the US.",
    'description': " ",  # Non-empty string to avoid loading the README file.
    'depends': ['payment'],
    'data': [
        'views/payment_adyen_templates.xml',
        'views/payment_form_templates.xml',
        'views/payment_provider_views.xml',

        'data/payment_provider_data.xml',  # Depends on views/payment_adyen_templates.xml

        'wizards/payment_capture_wizard_views.xml',
    ],
    'post_init_hook': 'post_init_hook',
    'uninstall_hook': 'uninstall_hook',
    'assets': {
        'web.assets_frontend': [
            'payment_adyen/static/src/js/payment_form.js',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
import binascii
import hashlib
import hmac
import logging
import pprint

from werkzeug import urls
from werkzeug.exceptions import Forbidden

from odoo import _, http
from odoo.exceptions import ValidationError
from odoo.http import request

from odoo.addons.payment import utils as payment_utils
from odoo.addons.payment_adyen import utils as adyen_utils

_logger = logging.getLogger(__name__)


class AdyenController(http.Controller):

    _webhook_url = '/payment/adyen/notification'

    @http.route('/payment/adyen/payment_methods', type='json', auth='public')
    def adyen_payment_methods(self, provider_id, formatted_amount=None, partner_id=None):
        """ Query the available payment methods based on the payment context.

        :param int provider_id: The provider handling the transaction, as a `payment.provider` id
        :param dict formatted_amount: The Adyen-formatted amount.
        :param int partner_id: The partner making the transaction, as a `res.partner` id
        :return: The JSON-formatted content of the response
        :rtype: dict
        """
        provider_sudo = request.env['payment.provider'].sudo().browse(provider_id)
        partner_sudo = partner_id and request.env['res.partner'].sudo().browse(partner_id).exists()
        # The lang is taken from the context rather than from the partner because it is not required
        # to be logged in to make a payment, and because the lang is not always set on the partner.
        # Adyen only supports a limited set of languages but, instead of looking for the closest
        # match in https://docs.adyen.com/checkout/components-web/localization-components, we simply
        # provide the lang string as is (after adapting the format) and let Adyen find the best fit.
        lang_code = (request.context.get('lang') or 'en-US').replace('_', '-')
        shopper_reference = partner_sudo and f'ODOO_PARTNER_{partner_sudo.id}'
        partner_country_code = (
            partner_sudo.country_id.code or provider_sudo.company_id.country_id.code or 'NL'
        )
        data = {
            'merchantAccount': provider_sudo.adyen_merchant_account,
            'amount': formatted_amount,
            'countryCode': partner_country_code,  # ISO 3166-1 alpha-2 (e.g.: 'BE')
            'shopperLocale': lang_code,  # IETF language tag (e.g.: 'fr-BE')
            'shopperReference': shopper_reference,
            'channel': 'Web',
        }
        response_content = provider_sudo._adyen_make_request(
            endpoint='/paymentMethods', payload=data, method='POST'
        )
        _logger.info("paymentMethods request response:\n%s", pprint.pformat(response_content))
        response_content['country_code'] = partner_country_code
        return response_content

    @http.route('/payment/adyen/payments', type='json', auth='public')
    def adyen_payments(
        self, provider_id, reference, converted_amount, currency_id, partner_id, payment_method,
        access_token, browser_info=None
    ):
        """ Make a payment request and handle the notification data.

        :param int provider_id: The provider handling the transaction, as a `payment.provider` id
        :param str reference: The reference of the transaction
        :param int converted_amount: The amount of the transaction in minor units of the currency
        :param int currency_id: The currency of the transaction, as a `res.currency` id
        :param int partner_id: The partner making the transaction, as a `res.partner` id
        :param dict payment_method: The details of the payment method used for the transaction
        :param str access_token: The access token used to verify the provided values
        :param dict browser_info: The browser info to pass to Adyen
        :return: The JSON-formatted content of the response
        :rtype: dict
        """
        # Check that the transaction details have not been altered. This allows preventing users
        # from validating transactions by paying less than agreed upon.
        if not payment_utils.check_access_token(
            access_token, reference, converted_amount, currency_id, partner_id
        ):
            raise ValidationError("Adyen: " + _("Received tampered payment request data."))

        # Prepare the payment request to Adyen
        provider_sudo = request.env['payment.provider'].sudo().browse(provider_id).exists()
        tx_sudo = request.env['payment.transaction'].sudo().search([('reference', '=', reference)])
        data = {
            'merchantAccount': provider_sudo.adyen_merchant_account,
            'amount': {
                'value': converted_amount,
                'currency': request.env['res.currency'].browse(currency_id).name,  # ISO 4217
            },
            'reference': reference,
            'paymentMethod': payment_method,
            'shopperReference': provider_sudo._adyen_compute_shopper_reference(partner_id),
            'recurringProcessingModel': 'CardOnFile',  # Most susceptible to trigger a 3DS check
            'shopperIP': payment_utils.get_customer_ip_address(),
            'shopperInteraction': 'Ecommerce',
            'shopperEmail': tx_sudo.partner_email or "",
            'shopperName': adyen_utils.format_partner_name(tx_sudo.partner_name),
            'telephoneNumber': tx_sudo.partner_phone or "",
            'storePaymentMethod': tx_sudo.tokenize,  # True by default on Adyen side
            'authenticationData': {
                'threeDSRequestData': {
                    'nativeThreeDS': 'preferred',
                }
            },
            'channel': 'web',  # Required to support 3DS
            'origin': provider_sudo.get_base_url(),  # Required to support 3DS
            'browserInfo': browser_info,  # Required to support 3DS
            'returnUrl': urls.url_join(
                provider_sudo.get_base_url(),
                # Include the reference in the return url to be able to match it after redirection.
                # The key 'merchantReference' is chosen on purpose to be the same as that returned
                # by the /payments endpoint of Adyen.
                f'/payment/adyen/return?merchantReference={reference}'
            ),
            **adyen_utils.include_partner_addresses(tx_sudo),
        }

        # Force the capture delay on Adyen side if the provider is not configured for capturing
        # payments manually. This is necessary because it's not possible to distinguish
        # 'AUTHORISATION' events sent by Adyen with the merchant account's capture delay set to
        # 'manual' from events with the capture delay set to 'immediate' or a number of hours. If
        # the merchant account is configured to capture payments with a delay but the provider is
        # not, we force the immediate capture to avoid considering authorized transactions as
        # captured on Odoo.
        if not provider_sudo.capture_manually:
            data.update(captureDelayHours=0)

        # Make the payment request to Adyen
        idempotency_key = payment_utils.generate_idempotency_key(
            tx_sudo, scope='payment_request_controller'
        )
        response_content = provider_sudo._adyen_make_request(
            endpoint='/payments', payload=data, method='POST', idempotency_key=idempotency_key
        )

        # Handle the payment request response
        _logger.info(
            "payment request response for transaction with reference %s:\n%s",
            reference, pprint.pformat(response_content)
        )
        tx_sudo._handle_notification_data(
            'adyen', dict(response_content, merchantReference=reference),  # Match the transaction
        )
        return response_content

    @http.route('/payment/adyen/payments/details', type='json', auth='public')
    def adyen_payment_details(self, provider_id, reference, payment_details):
        """ Submit the details of the additional actions and handle the notification data.

         The additional actions can have been performed both from the inline form or during a
         redirection.

        :param int provider_id: The provider handling the transaction, as a `payment.provider` id
        :param str reference: The reference of the transaction
        :param dict payment_details: The details of the additional actions performed for the payment
        :return: The JSON-formatted content of the response
        :rtype: dict
        """
        # Make the payment details request to Adyen
        provider_sudo = request.env['payment.provider'].browse(provider_id).sudo()
        response_content = provider_sudo._adyen_make_request(
            endpoint='/payments/details', payload=payment_details, method='POST'
        )

        # Handle the payment details request response
        _logger.info(
            "payment details request response for transaction with reference %s:\n%s",
            reference, pprint.pformat(response_content)
        )
        request.env['payment.transaction'].sudo()._handle_notification_data(
            'adyen', dict(response_content, merchantReference=reference),  # Match the transaction
        )

        return response_content

    @http.route('/payment/adyen/return', type='http', auth='public', csrf=False, save_session=False)
    def adyen_return_from_3ds_auth(self, **data):
        """ Process the authentication data sent by Adyen after redirection from the 3DS1 page.

        The route is flagged with `save_session=False` to prevent Odoo from assigning a new session
        to the user if they are redirected to this route with a POST request. Indeed, as the session
        cookie is created without a `SameSite` attribute, some browsers that don't implement the
        recommended default `SameSite=Lax` behavior will not include the cookie in the redirection
        request from the payment provider to Odoo. As the redirection to the '/payment/status' page
        will satisfy any specification of the `SameSite` attribute, the session of the user will be
        retrieved and with it the transaction which will be immediately post-processed.

        :param dict data: The authentication result data. May include custom params sent to Adyen in
                          the request to allow matching the transaction when redirected here.
        """
        # Retrieve the transaction based on the reference included in the return url
        tx_sudo = request.env['payment.transaction'].sudo()._get_tx_from_notification_data(
            'adyen', data
        )

        # Overwrite the operation to force the flow to 'redirect'. This is necessary because even
        # thought Adyen is implemented as a direct payment provider, it will redirect the user out
        # of Odoo in some cases. For instance, when a 3DS1 authentication is required, or for
        # special payment methods that are not handled by the drop-in (e.g. Sofort).
        tx_sudo.operation = 'online_redirect'

        # Query and process the result of the additional actions that have been performed
        _logger.info(
            "handling redirection from Adyen for transaction with reference %s with data:\n%s",
            tx_sudo.reference, pprint.pformat(data)
        )
        self.adyen_payment_details(
            tx_sudo.provider_id.id,
            data['merchantReference'],
            {
                'details': {
                    'redirectResult': data['redirectResult'],
                },
            },
        )

        # Redirect the user to the status page
        return request.redirect('/payment/status')

    @http.route(_webhook_url, type='http', methods=['POST'], auth='public', csrf=False)
    def adyen_webhook(self):
        """ Process the data sent by Adyen to the webhook based on the event code.

        See https://docs.adyen.com/development-resources/webhooks/understand-notifications for the
        exhaustive list of event codes.

        :return: The '[accepted]' string to acknowledge the notification
        :rtype: str
        """
        data = request.get_json_data()
        for notification_item in data['notificationItems']:
            notification_data = notification_item['NotificationRequestItem']

            _logger.info(
                "notification received from Adyen with data:\n%s", pprint.pformat(notification_data)
            )
            try:
                # Check the integrity of the notification
                tx_sudo = request.env['payment.transaction'].sudo()._get_tx_from_notification_data(
                    'adyen', notification_data
                )
            except ValidationError:
                # Warn rather than log the traceback to avoid noise when a POS payment notification
                # is received and the corresponding `payment.transaction` record is not found.
                _logger.warning("unable to find the transaction; skipping to acknowledge")
            else:
                self._verify_notification_signature(notification_data, tx_sudo)

                # Check whether the event of the notification succeeded and reshape the notification
                # data for parsing
                success = notification_data['success'] == 'true'
                event_code = notification_data['eventCode']
                if event_code == 'AUTHORISATION' and success:
                    notification_data['resultCode'] = 'Authorised'
                elif event_code == 'CANCELLATION':
                    notification_data['resultCode'] = 'Cancelled' if success else 'Error'
                elif event_code in ['REFUND', 'CAPTURE']:
                    notification_data['resultCode'] = 'Authorised' if success else 'Error'
                elif event_code == 'CAPTURE_FAILED' and success:
                    # The capture failed after a capture notification with success = True was sent
                    notification_data['resultCode'] = 'Error'
                else:
                    continue  # Don't handle unsupported event codes and failed events
                try:
                    # Handle the notification data as if they were feedback of a S2S payment request
                    tx_sudo._handle_notification_data('adyen', notification_data)
                except ValidationError:  # Acknowledge the notification to avoid getting spammed
                    _logger.exception(
                        "unable to handle the notification data;skipping to acknowledge"
                    )

        return request.make_json_response('[accepted]')  # Acknowledge the notification

    @staticmethod
    def _verify_notification_signature(notification_data, tx_sudo):
        """ Check that the received signature matches the expected one.

        :param dict notification_data: The notification payload containing the received signature
        :param recordset tx_sudo: The sudoed transaction referenced by the notification data, as a
                                  `payment.transaction` record
        :return: None
        :raise: :class:`werkzeug.exceptions.Forbidden` if the signatures don't match
        """
        # Retrieve the received signature from the payload
        received_signature = notification_data.get('additionalData', {}).get('hmacSignature')
        if not received_signature:
            _logger.warning("received notification with missing signature")
            raise Forbidden()

        # Compare the received signature with the expected signature computed from the payload
        hmac_key = tx_sudo.provider_id.adyen_hmac_key
        expected_signature = AdyenController._compute_signature(notification_data, hmac_key)
        if not hmac.compare_digest(received_signature, expected_signature):
            _logger.warning("received notification with invalid signature")
            raise Forbidden()

    @staticmethod
    def _compute_signature(payload, hmac_key):
        """ Compute the signature from the payload.

        See https://docs.adyen.com/development-resources/webhooks/verify-hmac-signatures

        :param dict payload: The notification payload
        :param str hmac_key: The HMAC key of the provider handling the transaction
        :return: The computed signature
        :rtype: str
        """
        def _flatten_dict(_value, _path_base='', _separator='.'):
            """ Recursively generate a flat representation of a dict.

            :param Object _value: The value to flatten. A dict or an already flat value
            :param str _path_base: They base path for keys of _value, including preceding separators
            :param str _separator: The string to use as a separator in the key path
            """
            if isinstance(_value, dict):  # The inner value is a dict, flatten it
                _path_base = _path_base if not _path_base else _path_base + _separator
                for _key in _value:
                    yield from _flatten_dict(_value[_key], _path_base + str(_key))
            else:  # The inner value cannot be flattened, yield it
                yield _path_base, _value

        def _to_escaped_string(_value):
            """ Escape payload values that are using illegal symbols and cast them to string.

            String values containing `\\` or `:` are prefixed with `\\`.
            Empty values (`None`) are replaced by an empty string.

            :param Object _value: The value to escape
            :return: The escaped value
            :rtype: string
            """
            if isinstance(_value, str):
                return _value.replace('\\', '\\\\').replace(':', '\\:')
            elif _value is None:
                return ''
            else:
                return str(_value)

        signature_keys = [
            'pspReference', 'originalReference', 'merchantAccountCode', 'merchantReference',
            'amount.value', 'amount.currency', 'eventCode', 'success'
        ]
        # Flatten the payload to allow accessing inner dicts naively
        flattened_payload = {k: v for k, v in _flatten_dict(payload)}
        # Build the list of signature values as per the list of required signature keys
        signature_values = [flattened_payload.get(key) for key in signature_keys]
        # Escape values using forbidden symbols
        escaped_values = [_to_escaped_string(value) for value in signature_values]
        # Concatenate values together with ':' as delimiter
        signing_string = ':'.join(escaped_values)
        # Convert the HMAC key to the binary representation
        binary_hmac_key = binascii.a2b_hex(hmac_key.encode('ascii'))
        # Calculate the HMAC with the binary representation of the signing string with SHA-256
        binary_hmac = hmac.new(binary_hmac_key, signing_string.encode('utf-8'), hashlib.sha256)
        # Calculate the signature by encoding the result with Base64
        return base64.b64encode(binary_hmac.digest()).decode()

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\neutralize.sql

```sql
-- disable adyen payment provider
UPDATE payment_provider
   SET adyen_merchant_account = NULL,
       adyen_api_key = NULL,
       adyen_hmac_key = NULL;

```

## File: data\payment_provider_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="payment.payment_provider_adyen" model="payment.provider">
        <field name="code">adyen</field>
        <field name="inline_form_view_id" ref="inline_form"/>
        <field name="allow_tokenization">True</field>
    </record>

</odoo>

```

## File: models\payment_provider.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json
import logging
import re
import requests

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError

from odoo.addons.payment import utils as payment_utils
from odoo.addons.payment_adyen import const

_logger = logging.getLogger(__name__)


class PaymentProvider(models.Model):
    _inherit = 'payment.provider'

    code = fields.Selection(
        selection_add=[('adyen', "Adyen")], ondelete={'adyen': 'set default'})
    adyen_merchant_account = fields.Char(
        string="Merchant Account",
        help="The code of the merchant account to use with this provider",
        required_if_provider='adyen', groups='base.group_system')
    adyen_api_key = fields.Char(
        string="API Key", help="The API key of the webservice user", required_if_provider='adyen',
        groups='base.group_system')
    adyen_client_key = fields.Char(
        string="Client Key", help="The client key of the webservice user",
        required_if_provider='adyen')
    adyen_hmac_key = fields.Char(
        string="HMAC Key", help="The HMAC key of the webhook", required_if_provider='adyen',
        groups='base.group_system')
    adyen_api_url_prefix = fields.Char(
        string="API URL Prefix",
        help="The base URL for the API endpoints",
        required_if_provider='adyen',
    )

    #=== CRUD METHODS ===#

    @api.model_create_multi
    def create(self, values_list):
        for values in values_list:
            self._adyen_extract_prefix_from_api_url(values)
        return super().create(values_list)

    def write(self, values):
        self._adyen_extract_prefix_from_api_url(values)
        return super().write(values)

    @api.model
    def _adyen_extract_prefix_from_api_url(self, values):
        """ Update the create or write values with the prefix extracted from the API URL.

        :param dict values: The create or write values.
        :return: None
        """
        if values.get('adyen_api_url_prefix'):  # Test if we're duplicating a provider.
            values['adyen_api_url_prefix'] = re.sub(
                r'(?:https://)?(\w+-\w+).*', r'\1', values['adyen_api_url_prefix']
            )

    #=== COMPUTE METHODS ===#

    def _compute_feature_support_fields(self):
        """ Override of `payment` to enable additional features. """
        super()._compute_feature_support_fields()
        self.filtered(lambda p: p.code == 'adyen').update({
            'support_manual_capture': 'partial',
            'support_refund': 'partial',
            'support_tokenization': True,
        })

    #=== BUSINESS METHODS - PAYMENT FLOW ===#

    def _adyen_make_request(self, endpoint, endpoint_param=None, payload=None, method='POST', idempotency_key=None):
        """ Make a request to Adyen API at the specified endpoint.

        Note: self.ensure_one()

        :param str endpoint: The endpoint to be reached by the request
        :param str endpoint_param: A variable required by some endpoints which are interpolated with
                                   it if provided. For example, the provider reference of the source
                                   transaction for the '/payments/{}/refunds' endpoint.
        :param dict payload: The payload of the request
        :param str method: The HTTP method of the request
        :param str idempotency_key: The idempotency key to pass in the request.
        :return: The JSON-formatted content of the response
        :rtype: dict
        :raise: ValidationError if an HTTP error occurs
        """

        def _build_url(prefix_, version_, endpoint_):
            """ Build an API URL by appending the version and endpoint to a base URL.

            The final URL follows this pattern: `<_base>/V<_version>/<_endpoint>`.

            :param str prefix_: The API URL prefix of the account.
            :param int version_: The version of the endpoint.
            :param str endpoint_: The endpoint of the URL.
            :return: The final URL.
            :rtype: str
            """
            prefix_ = prefix_.rstrip('/')  # Remove potential trailing slash
            endpoint_ = endpoint_.lstrip('/')  # Remove potential leading slash
            test_mode_ = self.state == 'test'
            prefix_ = f'{prefix_}.adyen' if test_mode_ else f'{prefix_}-checkout-live.adyenpayments'
            return f'https://{prefix_}.com/checkout/V{version_}/{endpoint_}'

        self.ensure_one()

        version = const.API_ENDPOINT_VERSIONS[endpoint]
        endpoint = endpoint if not endpoint_param else endpoint.format(endpoint_param)
        url = _build_url(self.adyen_api_url_prefix, version, endpoint)
        headers = {'X-API-Key': self.adyen_api_key}
        if method == 'POST' and idempotency_key:
            headers['idempotency-key'] = idempotency_key
        try:
            response = requests.request(method, url, json=payload, headers=headers, timeout=60)
            try:
                response.raise_for_status()
            except requests.exceptions.HTTPError:
                _logger.exception(
                    "invalid API request at %s with data %s: %s", url, payload, response.text
                )
                msg = response.json().get('message', '')
                raise ValidationError(
                    "Adyen: " + _("The communication with the API failed. Details: %s", msg)
                )
        except requests.exceptions.ConnectionError:
            _logger.exception("unable to reach endpoint at %s", url)
            raise ValidationError("Adyen: " + _("Could not establish the connection to the API."))
        return response.json()

    def _adyen_compute_shopper_reference(self, partner_id):
        """ Compute a unique reference of the partner for Adyen.

        This is used for the `shopperReference` field in communications with Adyen and stored in the
        `adyen_shopper_reference` field on `payment.token` if the payment method is tokenized.

        :param recordset partner_id: The partner making the transaction, as a `res.partner` id
        :return: The unique reference for the partner
        :rtype: str
        """
        return f'ODOO_PARTNER_{partner_id}'

    #=== BUSINESS METHODS - GETTERS ===#

    def _adyen_get_inline_form_values(self, pm_code, amount=None, currency=None):
        """ Return a serialized JSON of the required values to render the inline form.

        Note: `self.ensure_one()`

        :param str pm_code: The code of the payment method whose inline form to render.
        :param float amount: The transaction amount.
        :param res.currency currency: The transaction currency.
        :return: The JSON serial of the required values to render the inline form.
        :rtype: str
        """
        self.ensure_one()

        inline_form_values = {
            'client_key': self.adyen_client_key,
            'adyen_pm_code': const.PAYMENT_METHODS_MAPPING.get(pm_code, pm_code),
            'formatted_amount': self._adyen_get_formatted_amount(amount, currency),
        }
        return json.dumps(inline_form_values)

    def _adyen_get_formatted_amount(self, amount=None, currency=None):
        """ Return the amount in the format required by Adyen.

        The formatted amount is a dict with keys 'value' and 'currency'.

        :param float amount: The transaction amount.
        :param res.currency currency: The transaction currency.
        :return: The Adyen-formatted amount.
        :rtype: dict
        """
        currency_code = currency and currency.name
        converted_amount = amount and currency_code and payment_utils.to_minor_currency_units(
            amount, currency, const.CURRENCY_DECIMALS.get(currency_code)
        )
        return {
            'value': converted_amount,
            'currency': currency_code,
        }

    def _get_default_payment_method_codes(self):
        """ Override of `payment` to return the default payment method codes. """
        default_codes = super()._get_default_payment_method_codes()
        if self.code != 'adyen':
            return default_codes
        return const.DEFAULT_PAYMENT_METHODS_CODES

```

## File: models\payment_token.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, fields, models


class PaymentToken(models.Model):
    _inherit = 'payment.token'

    adyen_shopper_reference = fields.Char(
        string="Shopper Reference", help="The unique reference of the partner owning this token",
        readonly=True)

```

## File: models\payment_transaction.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import pprint

from odoo import _, models
from odoo.exceptions import UserError, ValidationError
from odoo.tools import format_amount

from odoo.addons.payment import utils as payment_utils
from odoo.addons.payment_adyen import utils as adyen_utils
from odoo.addons.payment_adyen import const

_logger = logging.getLogger(__name__)


class PaymentTransaction(models.Model):
    _inherit = 'payment.transaction'

    #=== BUSINESS METHODS ===#

    def _get_specific_processing_values(self, processing_values):
        """ Override of payment to return Adyen-specific processing values.

        Note: self.ensure_one() from `_get_processing_values`

        :param dict processing_values: The generic processing values of the transaction
        :return: The dict of provider-specific processing values
        :rtype: dict
        """
        res = super()._get_specific_processing_values(processing_values)
        if self.provider_code != 'adyen':
            return res

        converted_amount = payment_utils.to_minor_currency_units(
            self.amount, self.currency_id, const.CURRENCY_DECIMALS.get(self.currency_id.name)
        )
        return {
            'converted_amount': converted_amount,
            'access_token': payment_utils.generate_access_token(
                processing_values['reference'],
                converted_amount,
                self.currency_id.id,
                processing_values['partner_id']
            )
        }

    def _send_payment_request(self):
        """ Override of payment to send a payment request to Adyen.

        Note: self.ensure_one()

        :return: None
        :raise: UserError if the transaction is not linked to a token
        """
        super()._send_payment_request()
        if self.provider_code != 'adyen':
            return

        # Prepare the payment request to Adyen
        if not self.token_id:
            raise UserError("Adyen: " + _("The transaction is not linked to a token."))

        converted_amount = payment_utils.to_minor_currency_units(
            self.amount, self.currency_id, const.CURRENCY_DECIMALS.get(self.currency_id.name)
        )
        data = {
            'merchantAccount': self.provider_id.adyen_merchant_account,
            'amount': {
                'value': converted_amount,
                'currency': self.currency_id.name,
            },
            'reference': self.reference,
            'paymentMethod': {
                'storedPaymentMethodId': self.token_id.provider_ref,
            },
            'shopperReference': self.token_id.adyen_shopper_reference,
            'recurringProcessingModel': 'Subscription',
            'shopperIP': payment_utils.get_customer_ip_address(),
            'shopperInteraction': 'ContAuth',
            'shopperEmail': self.partner_email,
            'shopperName': adyen_utils.format_partner_name(self.partner_name),
            'telephoneNumber': self.partner_phone,
            **adyen_utils.include_partner_addresses(self),
        }

        # Force the capture delay on Adyen side if the provider is not configured for capturing
        # payments manually. This is necessary because it's not possible to distinguish
        # 'AUTHORISATION' events sent by Adyen with the merchant account's capture delay set to
        # 'manual' from events with the capture delay set to 'immediate' or a number of hours. If
        # the merchant account is configured to capture payments with a delay but the provider is
        # not, we force the immediate capture to avoid considering authorized transactions as
        # captured on Odoo.
        if not self.provider_id.capture_manually:
            data.update(captureDelayHours=0)

        # Make the payment request to Adyen
        try:
            response_content = self.provider_id._adyen_make_request(
                endpoint='/payments',
                payload=data,
                method='POST',
                idempotency_key=payment_utils.generate_idempotency_key(
                    self, scope='payment_request_token'
                )
            )
        except ValidationError as e:
            if self.operation == 'offline':
                self._set_error(str(e))  # Log the error message on linked documents' chatter.
                return  # There is nothing to process.
            else:
                raise e

        # Handle the payment request response
        _logger.info(
            "payment request response for transaction with reference %s:\n%s",
            self.reference, pprint.pformat(response_content)
        )
        self._handle_notification_data('adyen', response_content)

    def _send_refund_request(self, amount_to_refund=None):
        """ Override of payment to send a refund request to Adyen.

        Note: self.ensure_one()

        :param float amount_to_refund: The amount to refund
        :return: The refund transaction created to process the refund request.
        :rtype: recordset of `payment.transaction`
        """
        refund_tx = super()._send_refund_request(amount_to_refund=amount_to_refund)
        if self.provider_code != 'adyen':
            return refund_tx

        # Make the refund request to Adyen
        converted_amount = payment_utils.to_minor_currency_units(
            -refund_tx.amount,  # The amount is negative for refund transactions
            refund_tx.currency_id,
            arbitrary_decimal_number=const.CURRENCY_DECIMALS.get(refund_tx.currency_id.name)
        )
        data = {
            'merchantAccount': self.provider_id.adyen_merchant_account,
            'amount': {
                'value': converted_amount,
                'currency': refund_tx.currency_id.name,
            },
            'reference': refund_tx.reference,
        }
        response_content = refund_tx.provider_id._adyen_make_request(
            endpoint='/payments/{}/refunds',
            endpoint_param=self.provider_reference,
            payload=data,
            method='POST'
        )
        _logger.info(
            "refund request response for transaction with reference %s:\n%s",
            self.reference, pprint.pformat(response_content)
        )

        # Handle the refund request response
        psp_reference = response_content.get('pspReference')
        status = response_content.get('status')
        if psp_reference and status == 'received':
            # The PSP reference associated with this /refunds request is different from the psp
            # reference associated with the original payment request.
            refund_tx.provider_reference = psp_reference

        return refund_tx

    def _send_capture_request(self, amount_to_capture=None):
        """ Override of `payment` to send a capture request to Adyen. """
        capture_child_tx = super()._send_capture_request(amount_to_capture=amount_to_capture)
        if self.provider_code != 'adyen':
            return capture_child_tx

        amount_to_capture = amount_to_capture or self.amount
        converted_amount = payment_utils.to_minor_currency_units(
            amount_to_capture, self.currency_id, const.CURRENCY_DECIMALS.get(self.currency_id.name)
        )
        data = {
            'merchantAccount': self.provider_id.adyen_merchant_account,
            'amount': {
                'value': converted_amount,
                'currency': self.currency_id.name,
            },
            'reference': self.reference,
        }
        response_content = self.provider_id._adyen_make_request(
            endpoint='/payments/{}/captures',
            endpoint_param=self.provider_reference,
            payload=data,
            method='POST',
        )
        _logger.info("capture request response:\n%s", pprint.pformat(response_content))

        # Handle the capture request response
        status = response_content.get('status')
        formatted_amount = format_amount(self.env, amount_to_capture, self.currency_id)
        if status == 'received':
            self._log_message_on_linked_documents(_(
                "The capture request of %(amount)s for the transaction with reference %(ref)s has "
                "been requested (%(provider_name)s).",
                amount=formatted_amount, ref=self.reference, provider_name=self.provider_id.name
            ))

        if capture_child_tx:
            # The PSP reference associated with this capture request is different from the PSP
            # reference associated with the original payment request.
            capture_child_tx.provider_reference = response_content.get('pspReference')

        return capture_child_tx

    def _send_void_request(self, amount_to_void=None):
        """ Override of `payment` to send a void request to Adyen. """
        child_void_tx = super()._send_void_request(amount_to_void=amount_to_void)
        if self.provider_code != 'adyen':
            return child_void_tx

        data = {
            'merchantAccount': self.provider_id.adyen_merchant_account,
            'reference': self.reference,
        }
        response_content = self.provider_id._adyen_make_request(
            endpoint='/payments/{}/cancels',
            endpoint_param=self.provider_reference,
            payload=data,
            method='POST',
        )
        _logger.info("void request response:\n%s", pprint.pformat(response_content))

        # Handle the void request response
        status = response_content.get('status')
        if status == 'received':
            self._log_message_on_linked_documents(_(
                "A request was sent to void the transaction with reference %s (%s).",
                self.reference, self.provider_id.name
            ))

        if child_void_tx:
            # The PSP reference associated with this void request is different from the PSP
            # reference associated with the original payment request.
            child_void_tx.provider_reference = response_content.get('pspReference')

        return child_void_tx

    def _get_tx_from_notification_data(self, provider_code, notification_data):
        """ Override of payment to find the transaction based on Adyen data.

        :param str provider_code: The code of the provider that handled the transaction
        :param dict notification_data: The notification data sent by the provider
        :return: The transaction if found
        :rtype: recordset of `payment.transaction`
        :raise: ValidationError if inconsistent data were received
        :raise: ValidationError if the data match no transaction
        """
        tx = super()._get_tx_from_notification_data(provider_code, notification_data)
        if provider_code != 'adyen' or len(tx) == 1:
            return tx

        reference = notification_data.get('merchantReference')
        if not reference:
            raise ValidationError("Adyen: " + _("Received data with missing merchant reference"))

        event_code = notification_data.get('eventCode', 'AUTHORISATION')  # Fallback on auth if S2S.
        provider_reference = notification_data.get('pspReference')
        source_reference = notification_data.get('originalReference')
        if event_code == 'AUTHORISATION':
            tx = self.search([('reference', '=', reference), ('provider_code', '=', 'adyen')])
        elif event_code in ['CANCELLATION', 'CAPTURE', 'CAPTURE_FAILED']:
            # The capture/void may be initiated from Adyen, so we can't trust the reference.
            # We find the transaction based on the original provider reference since Adyen will have
            # two different references: one for the original transaction and one for the capture or
            # void. We keep the second one only for child transactions. For full capture/void, no
            # child transaction are created. Thus, we first look for the source transaction before
            # checking if we need to find/create a child transaction.
            source_tx = self.search(
                [('provider_reference', '=', source_reference), ('provider_code', '=', 'adyen')]
            )
            if source_tx:
                notification_data_amount = notification_data.get('amount', {}).get('value')
                converted_notification_amount = payment_utils.to_major_currency_units(
                    notification_data_amount, source_tx.currency_id
                )
                if source_tx.amount == converted_notification_amount:  # Full capture/void.
                    tx = source_tx
                else:  # Partial capture/void; we search for the child transaction instead.
                    tx = self.search([
                        ('provider_reference', '=', provider_reference),
                        ('provider_code', '=', 'adyen'),
                    ])
                    if tx and tx.amount != converted_notification_amount:
                        # If the void was requested expecting a certain amount but, in the meantime,
                        # others captures that Odoo was unaware of were done, the amount voided will
                        # be different from the amount of the existing transaction.
                        tx._set_error(_(
                            "The amount processed by Adyen for the transaction %s is different than"
                            " the one requested. Another transaction is created with the correct"
                            " amount.", tx.reference
                        ))
                        tx = self.env['payment.transaction']
                    if not tx:  # Partial capture/void initiated from Adyen or with a wrong amount.
                        # Manually create a child transaction with a new reference. The reference of
                        # the child transaction was personalized from Adyen and could be identical
                        # to that of an existing transaction.
                        tx = self._adyen_create_child_tx_from_notification_data(
                            source_tx, notification_data
                        )
            else:  # The capture/void was initiated for an unknown source transaction
                pass  # Don't do anything with the capture/void notification
        else:  # 'REFUND'
            # The refund may be initiated from Adyen, so we can't trust the reference, which could
            # be identical to another existing transaction. We find the transaction based on the
            # provider reference.
            tx = self.search(
                [('provider_reference', '=', provider_reference), ('provider_code', '=', 'adyen')]
            )
            if not tx:  # The refund was initiated from Adyen
                # Find the source transaction based on the original reference
                source_tx = self.search(
                    [('provider_reference', '=', source_reference), ('provider_code', '=', 'adyen')]
                )
                if source_tx:
                    # Manually create a refund transaction with a new reference. The reference of
                    # the refund transaction was personalized from Adyen and could be identical to
                    # that of an existing transaction.
                    tx = self._adyen_create_child_tx_from_notification_data(
                        source_tx, notification_data, is_refund=True
                    )
                else:  # The refund was initiated for an unknown source transaction
                    pass  # Don't do anything with the refund notification

        if not tx:
            raise ValidationError(
                "Adyen: " + _("No transaction found matching reference %s.", reference)
            )
        return tx

    def _adyen_create_child_tx_from_notification_data(
        self, source_tx, notification_data, is_refund=False
    ):
        """ Create a child transaction based on Adyen data.

        :param payment.transaction source_tx: The source transaction for which a new operation is
                                              initiated.
        :param dict notification_data: The notification data sent by the provider
        :return: The newly created child transaction.
        :rtype: payment.transaction
        :raise ValidationError: If inconsistent data were received.
        """
        provider_reference = notification_data.get('pspReference')
        amount = notification_data.get('amount', {}).get('value')
        if not provider_reference or amount is None:  # amount == 0 if success == False
            raise ValidationError(
                "Adyen: " + _("Received data for child transaction with missing transaction values")
            )

        converted_amount = payment_utils.to_major_currency_units(amount, source_tx.currency_id)
        return source_tx._create_child_transaction(
            converted_amount, is_refund=is_refund, provider_reference=provider_reference
        )

    def _process_notification_data(self, notification_data):
        """ Override of payment to process the transaction based on Adyen data.

        Note: self.ensure_one()

        :param dict notification_data: The notification data sent by the provider
        :return: None
        :raise: ValidationError if inconsistent data were received
        """
        super()._process_notification_data(notification_data)
        if self.provider_code != 'adyen':
            return

        # Extract or assume the event code. If none is provided, the feedback data originate from a
        # direct payment request whose feedback data share the same payload as an 'AUTHORISATION'
        # webhook notification.
        event_code = notification_data.get('eventCode', 'AUTHORISATION')

        # Update the provider reference. If the event code is 'CAPTURE' or 'CANCELLATION', we
        # discard the pspReference as it is different from the original pspReference of the tx.
        if 'pspReference' in notification_data and event_code in ['AUTHORISATION', 'REFUND']:
            self.provider_reference = notification_data.get('pspReference')

        # Update the payment method.
        payment_method_data = notification_data.get('paymentMethod', '')
        if isinstance(payment_method_data, dict):  # Not from webhook: the data contain the PM code.
            payment_method_type = payment_method_data['type']
            if payment_method_type == 'scheme':  # card
                payment_method_code = payment_method_data['brand']
            else:
                payment_method_code = payment_method_type
        else:  # Sent from the webhook: the PM code is directly received as a string.
            payment_method_code = payment_method_data

        payment_method = self.env['payment.method']._get_from_code(
            payment_method_code, mapping=const.PAYMENT_METHODS_MAPPING
        )
        self.payment_method_id = payment_method or self.payment_method_id

        # Update the payment state.
        payment_state = notification_data.get('resultCode')
        refusal_reason = notification_data.get('refusalReason') or notification_data.get('reason')
        if not payment_state:
            raise ValidationError("Adyen: " + _("Received data with missing payment state."))
        if payment_state in const.RESULT_CODES_MAPPING['pending']:
            self._set_pending()
        elif payment_state in const.RESULT_CODES_MAPPING['done']:
            additional_data = notification_data.get('additionalData', {})
            has_token_data = 'recurring.recurringDetailReference' in additional_data
            if self.tokenize and has_token_data:
                self._adyen_tokenize_from_notification_data(notification_data)

            if not self.provider_id.capture_manually:
                self._set_done()
            else:  # The payment was configured for manual capture.
                # Differentiate the state based on the event code.
                if event_code == 'AUTHORISATION':
                    self._set_authorized()
                else:  # 'CAPTURE'
                    self._set_done()

            # Immediately post-process the transaction if it is a refund, as the post-processing
            # will not be triggered by a customer browsing the transaction from the portal.
            if self.operation == 'refund':
                self.env.ref('payment.cron_post_process_payment_tx')._trigger()
        elif payment_state in const.RESULT_CODES_MAPPING['cancel']:
            self._set_canceled()
        elif payment_state in const.RESULT_CODES_MAPPING['error']:
            if event_code in ['AUTHORISATION', 'REFUND']:
                _logger.warning(
                    "the transaction with reference %s underwent an error. reason: %s",
                    self.reference, refusal_reason,
                )
                self._set_error(
                    _("An error occurred during the processing of your payment. Please try again.")
                )
            elif event_code == 'CANCELLATION':
                _logger.warning(
                    "The void of the transaction with reference %s failed. reason: %s",
                    self.reference, refusal_reason,
                )
                if self.source_transaction_id:  # child tx => The event can't be retried.
                    self._set_error(
                        _("The void of the transaction with reference %s failed.", self.reference)
                    )
                else:  # source tx with failed void stays in its state, could be voided again
                    self._log_message_on_linked_documents(
                        _("The void of the transaction with reference %s failed.", self.reference)
                    )
            else:  # 'CAPTURE', 'CAPTURE_FAILED'
                _logger.warning(
                    "The capture of the transaction with reference %s failed. reason: %s",
                    self.reference, refusal_reason,
                )
                if self.source_transaction_id:  # child_tx => The event can't be retried.
                    self._set_error(_(
                        "The capture of the transaction with reference %s failed.", self.reference
                    ))
                else:  # source tx with failed capture stays in its state, could be captured again
                    self._log_message_on_linked_documents(_(
                        "The capture of the transaction with reference %s failed.", self.reference
                    ))
        elif payment_state in const.RESULT_CODES_MAPPING['refused']:
            _logger.warning(
                "the transaction with reference %s was refused. reason: %s",
                self.reference, refusal_reason
            )
            self._set_error(_("Your payment was refused. Please try again."))
        else:  # Classify unsupported payment state as `error` tx state
            _logger.warning(
                "received data for transaction with reference %s with invalid payment state: %s",
                self.reference, payment_state
            )
            self._set_error(
                "Adyen: " + _("Received data with invalid payment state: %s", payment_state)
            )

    def _adyen_tokenize_from_notification_data(self, notification_data):
        """ Create a new token based on the notification data.

        Note: self.ensure_one()

        :param dict notification_data: The notification data sent by the provider
        :return: None
        """
        self.ensure_one()

        additional_data = notification_data['additionalData']
        token = self.env['payment.token'].create({
            'provider_id': self.provider_id.id,
            'payment_method_id': self.payment_method_id.id,
            'payment_details': additional_data.get('cardSummary'),
            'partner_id': self.partner_id.id,
            'provider_ref': additional_data['recurring.recurringDetailReference'],
            'adyen_shopper_reference': additional_data['recurring.shopperReference'],
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
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M7.054 20.637c.977 0 1.77.783 1.77 1.75v6.977H1.77A1.76 1.76 0 0 1 0 27.614V25.66c0-.967.792-1.75 1.77-1.75h1.907v2.772c0 .276.226.5.505.5h.965V23.32c0-.277-.226-.5-.506-.5H.126v-2.182h6.928Zm8.387 6.545V17h3.677v12.364h-7.054a1.76 1.76 0 0 1-1.77-1.75v-5.227c0-.967.792-1.75 1.77-1.75h1.906v6.045c0 .276.227.5.506.5h.965Zm10.294 0v-6.545h3.677V31.25a1.76 1.76 0 0 1-1.77 1.75h-6.927v-2.545h5.02v-1.091h-3.377a1.76 1.76 0 0 1-1.77-1.75v-6.977h3.677v6.045c0 .276.226.5.505.5h.965Zm12.201-6.545c.977 0 1.77.783 1.77 1.75v1.954a1.76 1.76 0 0 1-1.77 1.75h-1.907V23.32c0-.277-.226-.5-.505-.5h-.965v3.863c0 .276.226.5.505.5h4.515v2.182h-6.927a1.76 1.76 0 0 1-1.77-1.75v-6.977h7.054Zm10.294 0c.978 0 1.77.783 1.77 1.75v6.977h-3.676v-6.046c0-.275-.228-.5-.506-.5h-.965v6.546h-3.677v-8.727h7.054Z" fill="#32AA52"/><path d="M43.105 4h.972V.818h1.108V0H42v.818h1.105V4Zm2.775 0h.858V1.453h.053L47.663 4h.555l.871-2.547h.056V4H50V0h-1.108l-.924 2.714h-.05L46.99 0h-1.11v4Z" fill="#D1D5DB"/></svg>

```

## File: static\src\js\payment_form.js

```javascript
/** @odoo-module **/

import { loadJS, loadCSS } from '@web/core/assets';
import { _t } from '@web/core/l10n/translation';
import { pyToJsLocale } from '@web/core/l10n/utils';
import paymentForm from '@payment/js/payment_form';
import { RPCError } from '@web/core/network/rpc_service';

paymentForm.include({

    adyenCheckout: undefined,
    adyenComponents: undefined,

    // #=== DOM MANIPULATION ===#

    /**
     * Prepare the inline form of Adyen for direct payment.
     *
     * @override method from payment.payment_form
     * @private
     * @param {number} providerId - The id of the selected payment option's provider.
     * @param {string} providerCode - The code of the selected payment option's provider.
     * @param {number} paymentOptionId - The id of the selected payment option
     * @param {string} paymentMethodCode - The code of the selected payment method, if any.
     * @param {string} flow - The online payment flow of the selected payment option
     * @return {void}
     */
    async _prepareInlineForm(providerId, providerCode, paymentOptionId, paymentMethodCode, flow) {
        if (providerCode !== 'adyen') {
            this._super(...arguments);
            return;
        }

        // Check if instantiation of the component is needed.
        this.adyenComponents ??= {}; // Store the component of each instantiated payment method.
        if (flow === 'token') {
            return; // No component for tokens.
        } else if (this.adyenComponents[paymentOptionId]) {
            this._setPaymentFlow('direct'); // Overwrite the flow even if no re-instantiation.
            if (paymentMethodCode === 'paypal') { // PayPal uses its own button to submit.
                this._hideInputs();
            }
            return; // Don't re-instantiate if already done for this payment method.
        }

        // Overwrite the flow of the selected payment method.
        this._setPaymentFlow('direct');

        // Extract and deserialize the inline form values.
        const radio = document.querySelector('input[name="o_payment_radio"]:checked');
        const inlineFormValues = JSON.parse(radio.dataset['adyenInlineFormValues']);
        const formattedAmount = inlineFormValues['formatted_amount'];

        this.el.parentElement.querySelector(
            'script[src="https://checkoutshopper-live.adyen.com/checkoutshopper/sdk/5.39.0/adyen.js"]'
        )?.remove();
        this.el.parentElement.querySelector(
            'link[href="https://checkoutshopper-live.adyen.com/checkoutshopper/sdk/5.39.0/adyen.css"]'
        )?.remove();
        await loadJS('https://checkoutshopper-live.adyen.com/checkoutshopper/sdk/6.9.0/adyen.js')
        await loadCSS('https://checkoutshopper-live.adyen.com/checkoutshopper/sdk/6.9.0/adyen.css')
        const { AdyenCheckout, createComponent } = window.AdyenWeb;

        // Create the checkout object if not already done for another payment method.
        if (!this.adyenCheckout) {
            try {
                // Await the RPC to let it create AdyenCheckout before using it.
                const response = await this.rpc(
                    '/payment/adyen/payment_methods',
                    {
                        'provider_id': providerId,
                        'partner_id': parseInt(this.paymentContext['partnerId']),
                        'formatted_amount': formattedAmount,
                    },
                );
                // Create the Adyen Checkout SDK.
                const providerState = this._getProviderState(radio);
                const configuration = {
                    paymentMethodsResponse: response,
                    clientKey: inlineFormValues['client_key'],
                    amount: formattedAmount,
                    locale: pyToJsLocale(this._getContext().lang || 'en-US'),
                    countryCode: response['country_code'],
                    environment: providerState === 'enabled' ? 'live' : 'test',
                    onAdditionalDetails: this._adyenOnSubmitAdditionalDetails.bind(this),
                    onPaymentCompleted: this._adyenOnPaymentResolved.bind(this),
                    onPaymentFailed: this._adyenOnPaymentResolved.bind(this),
                    onError: this._adyenOnError.bind(this),
                    onSubmit: this._adyenOnSubmit.bind(this),
                };
                this.adyenCheckout = await AdyenCheckout(configuration);
            } catch (error) {
                if (error instanceof RPCError) {
                    this._displayErrorDialog(
                        _t("Cannot display the payment form"), error.data.message
                    );
                    this._enableButton();
                    return;
                }
                else {
                    return Promise.reject(error);
                }
            }
        }

        // Instantiate and mount the component.
        const componentConfiguration = {
            showPayButton: false,
        };
        if (paymentMethodCode === 'card') {
            componentConfiguration['hasHolderName'] = true;
            componentConfiguration['holderNameRequired'] = true;
            // Forbid Bancontact cards in the card component.
            componentConfiguration['brands'] = ['mc', 'visa', 'amex', 'discover'];
        }
        else if (paymentMethodCode === 'paypal') {
            // PayPal requires the form to be submitted with its own button.
            Object.assign(componentConfiguration, {
                style: {
                    disableMaxWidth: true
                },
                showPayButton: true,
                blockPayPalCreditButton: true,
                blockPayPalPayLaterButton: true
            });
            this._hideInputs();
            // Define necessary fields as the step _submitForm is missed.
            Object.assign(this.paymentContext, {
                tokenizationRequested: false,
                providerId: providerId,
                paymentMethodId: paymentOptionId,
            });
        }
        const inlineForm = this._getInlineForm(radio);
        const adyenContainer = inlineForm.querySelector('[name="o_adyen_component_container"]');
        this.adyenComponents[paymentOptionId] = createComponent(
            inlineFormValues['adyen_pm_code'],
            this.adyenCheckout,
            componentConfiguration
        ).mount(adyenContainer);
    },

    // #=== PAYMENT FLOW ===#

    /**
     * Trigger the payment processing by submitting the component.
     *
     * The component is submitted here instead of in `_processDirectFlow` because we use the native
     * submit button for PayPal, which does not follow the standard flow. The transaction is created
     * in `_adyenOnSubmit`.
     *
     * @override method from payment.payment_form
     * @private
     * @param {string} providerCode - The code of the selected payment option's provider.
     * @param {number} paymentOptionId - The id of the payment option handling the transaction.
     * @param {string} paymentMethodCode - The code of the selected payment method, if any.
     * @param {string} flow - The online payment flow of the transaction.
     * @return {void}
     */
    async _initiatePaymentFlow(providerCode, paymentOptionId, paymentMethodCode, flow) {
        if (providerCode !== 'adyen' || flow === 'token') {
            await this._super(...arguments); // Tokens are handled by the generic flow
            return;
        }

        if (!this.adyenComponents[paymentOptionId]) {  // Component creation failed.
            this._enableButton();
            return;
        }

        this.adyenComponents[paymentOptionId].submit();

        // The `onError` event handler is not used to validate inputs anymore since v5.0.0.
        if (!this.adyenComponents[paymentOptionId].isValid) {
            this._displayErrorDialog(_t("Incorrect payment details"));
            this._enableButton();
        }
    },

    /**
     * Handle the submit event of the component and initiate the payment.
     *
     * @private
     * @param {object} state - The state of the component.
     * @param {object} component - The component.
     * @param {object} actions - The possible action handlers to call.
     * @return {void}
     */
    _adyenOnSubmit(state, component, actions) {
        // Create the transaction and retrieve the processing values.
        this.rpc(
            this.paymentContext['transactionRoute'],
            this._prepareTransactionRouteParams(),
        ).then(processingValues => {
            component.reference = processingValues.reference; // Store final reference.
            // Initiate the payment.
            return this.rpc('/payment/adyen/payments', {
                'provider_id': processingValues.provider_id,
                'reference': processingValues.reference,
                'converted_amount': processingValues.converted_amount,
                'currency_id': processingValues.currency_id,
                'partner_id': processingValues.partner_id,
                'payment_method': state.data.paymentMethod,
                'access_token': processingValues.access_token,
                'browser_info': state.data.browserInfo,
            });
        }).then(paymentResponse => {
            if (!paymentResponse.resultCode) {
                actions.reject();
                return;
            }
            if (paymentResponse.action && paymentResponse.action.type !== 'redirect') {
                this._hideInputs(); // Only the inputs of the inline form should be used.
                this.call('ui', 'unblock'); // The page is blocked at this point; unblock it.
            }
            actions.resolve(paymentResponse);
        }).catch(error => {
            const error_message = error instanceof RPCError ? error.data.message : error.message;
            this._displayErrorDialog(_t("Payment processing failed"), error_message);
            this._enableButton();
        });
    },

    /**
     * Handle the additional details event of the component.
     *
     * @private
     * @param {object} state - The state of the component.
     * @param {object} component - The component.
     * @param {object} actions - The possible action handlers to call.
     * @return {void}
     */
    _adyenOnSubmitAdditionalDetails(state, component, actions) {
        this.rpc('/payment/adyen/payments/details', {
            'provider_id': this.paymentContext['providerId'],
            'reference': component.reference,
            'payment_details': state.data,
        }).then(paymentDetails => {
            if (!paymentDetails.resultCode) {
                actions.reject();
                return;
            }
            actions.resolve(paymentDetails);
        }).catch(error => {
            const error_message = error instanceof RPCError ? error.data.message : error.message;
            this._displayErrorDialog(_t("Payment processing failed"), error_message);
            this._enableButton();
        });
    },

    /**
    * Called when the payment is completed or failed.
    *
    * @private
    * @param {object} result
    * @param {object} component
    * @return {void}
    */
    _adyenOnPaymentResolved(result, component) {
        window.location = '/payment/status';
    },

    /**
     * Handle the error event of the component.
     *
     * @private
     * @param {object} error - The error in the component.
     * @return {void}
     */
    _adyenOnError(error) {
        this._displayErrorDialog(_t("Payment processing failed"), error.message);
        this._enableButton();
    },

});

```

## File: views\payment_adyen_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="inline_form">
        <div name="o_adyen_component_container"/>
    </template>

</odoo>

```

## File: views\payment_form_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- Only load the SDK on pages with the payment form. -->
    <template id="payment_adyen.form" inherit_id="payment.form">
        <xpath expr="." position="inside">
            <script src="https://checkoutshopper-live.adyen.com/checkoutshopper/sdk/5.39.0/adyen.js"
                    integrity="sha384-xEeOeJS9noqG6GBdLeyfdybymyC6T4EG+kKvvii2xJkChmwENEgCTpP+XvET9NDG"
                    crossorigin="anonymous"
            />
            <link rel="stylesheet"
                  href="https://checkoutshopper-live.adyen.com/checkoutshopper/sdk/5.39.0/adyen.css"
                  integrity="sha384-wfFaUCatOjF81fz/vsj2zdQuPWekgx9HbIMfEUjEHzTKX7v/juxeM+zIJA0QKJtO"
                  crossorigin="anonymous"
            />
        </xpath>
    </template>

    <template id="payment_adyen.method_form" inherit_id="payment.method_form">
        <xpath expr="//input[@name='o_payment_radio']" position="attributes">
            <attribute name="t-att-data-adyen-inline-form-values">
                provider_sudo._adyen_get_inline_form_values(pm_sudo.code, amount, currency)
            </attribute>
        </xpath>
    </template>

</odoo>

```

## File: views\payment_provider_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_provider_form" model="ir.ui.view">
        <field name="name">Adyen Provider Form</field>
        <field name="model">payment.provider</field>
        <field name="inherit_id" ref="payment.payment_provider_form"/>
        <field name="arch" type="xml">
            <group name="provider_credentials" position='inside'>
                <group invisible="code != 'adyen'">
                    <field name="adyen_merchant_account" required="code == 'adyen' and state != 'disabled'"/>
                    <field name="adyen_api_key" required="code == 'adyen' and state != 'disabled'" password="True"/>
                    <field name="adyen_client_key" required="code == 'adyen' and state != 'disabled'"/>
                    <field name="adyen_hmac_key" required="code == 'adyen' and state != 'disabled'" password="True"/>
                    <field name="adyen_api_url_prefix" required="code == 'adyen' and state != 'disabled'"/>
                </group>
            </group>
            <group name="provider_config" position='before'>
                <div invisible="code != 'adyen' or not capture_manually"
                     class="alert alert-warning"
                     role="alert">
                    <strong>Warning:</strong> To capture the amount manually, you also need to set
                    the Capture Delay to manual on your Adyen account settings.
                    <a href="https://www.odoo.com/documentation/17.0/applications/finance/payment_providers/adyen.html#place-a-hold-on-a-card"
                       title="Learn More"
                       target="_blank">Learn More</a>
                </div>
            </group>
        </field>
    </record>

</odoo>

```

## File: wizards\payment_capture_wizard.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class PaymentCaptureWizard(models.TransientModel):
    _inherit = 'payment.capture.wizard'

    has_adyen_tx = fields.Boolean(compute='_compute_has_adyen_tx')

    @api.depends('transaction_ids')
    def _compute_has_adyen_tx(self):
        for wizard in self:
            wizard.has_adyen_tx = any(tx.provider_code == 'adyen' for tx in wizard.transaction_ids)

```

## File: wizards\payment_capture_wizard_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_capture_wizard_view_form" model="ir.ui.view">
        <field name="name">payment.adyen.capture.wizard.form</field>
        <field name="model">payment.capture.wizard</field>
        <field name="inherit_id" ref="payment.payment_capture_wizard_view_form"/>
        <field name="arch" type="xml">
            <footer position="before">
                <field name="has_adyen_tx" invisible="1"/>
                <div id="alert_delayed_capture_tx"
                     role="alert"
                     class="alert alert-info mb-2"
                     invisible="not has_adyen_tx">
                    The capture or void of the transaction might take a few minutes to be
                    processed by Adyen and reflected in Odoo.
                </div>
            </footer>
        </field>
    </record>

</odoo>

```

## File: wizards\__init__.py

```python

from . import payment_capture_wizard

```

