# Odoo Module: payment_adyen

Category: Accounting/Payment Providers

This file contains the source code of the Odoo module.

## File: const.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Endpoints of the API.
# See https://docs.adyen.com/api-explorer/#/CheckoutService/v71/overview for Checkout API
# See https://docs.adyen.com/api-explorer/#/Recurring/v49/overview for Recurring API
API_ENDPOINT_VERSIONS = {
    '/disable': 49,                 # Recurring API
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

from odoo.addons.payment import setup_provider, reset_payment_provider


def post_init_hook(cr, registry):
    setup_provider(cr, registry, 'adyen')


def uninstall_hook(cr, registry):
    reset_payment_provider(cr, registry, 'adyen')

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
        'views/payment_provider_views.xml',
        'views/payment_templates.xml',  # Only load the SDK on pages with a payment form.

        'data/payment_provider_data.xml',  # Depends on views/payment_adyen_templates.xml
    ],
    'application': False,
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
from odoo.addons.payment_adyen.const import CURRENCY_DECIMALS

_logger = logging.getLogger(__name__)


class AdyenController(http.Controller):

    _webhook_url = '/payment/adyen/notification'

    @http.route('/payment/adyen/provider_info', type='json', auth='public')
    def adyen_provider_info(self, provider_id):
        """ Return public information on the provider.

        :param int provider_id: The provider handling the transaction, as a `payment.provider` id
        :return: Public information on the provider, namely: the state and client key
        :rtype: str
        """
        provider_sudo = request.env['payment.provider'].sudo().browse(provider_id).exists()
        return {
            'state': provider_sudo.state,
            'client_key': provider_sudo.adyen_client_key,
        }

    @http.route('/payment/adyen/payment_methods', type='json', auth='public')
    def adyen_payment_methods(self, provider_id, amount=None, currency_id=None, partner_id=None):
        """ Query the available payment methods based on the transaction context.

        :param int provider_id: The provider handling the transaction, as a `payment.provider` id
        :param float amount: The transaction amount
        :param int currency_id: The transaction currency, as a `res.currency` id
        :param int partner_id: The partner making the transaction, as a `res.partner` id
        :return: The JSON-formatted content of the response and formatted amount
        :rtype: dict
        """
        provider_sudo = request.env['payment.provider'].sudo().browse(provider_id)
        currency = request.env['res.currency'].browse(currency_id)
        currency_code = currency_id and currency.name
        converted_amount = amount and currency_code and payment_utils.to_minor_currency_units(
            amount, currency, CURRENCY_DECIMALS.get(currency_code)
        )
        partner_sudo = partner_id and request.env['res.partner'].sudo().browse(partner_id).exists()
        # The lang is taken from the context rather than from the partner because it is not required
        # to be logged in to make a payment, and because the lang is not always set on the partner.
        # Adyen only supports a limited set of languages but, instead of looking for the closest
        # match in https://docs.adyen.com/checkout/components-web/localization-components, we simply
        # provide the lang string as is (after adapting the format) and let Adyen find the best fit.
        lang_code = (request.context.get('lang') or 'en-US').replace('_', '-')
        shopper_reference = partner_sudo and f'ODOO_PARTNER_{partner_sudo.id}'
        amount = {
            'value': converted_amount,
            'currency': request.env['res.currency'].browse(currency_id).name,  # ISO 4217
        }
        partner_country_code = (
            partner_sudo.country_id.code or provider_sudo.company_id.country_id.code or 'NL'
        )
        data = {
            'merchantAccount': provider_sudo.adyen_merchant_account,
            'amount': amount,
            'countryCode': partner_country_code,  # ISO 3166-1 alpha-2 (e.g.: 'BE')
            'shopperLocale': lang_code,  # IETF language tag (e.g.: 'fr-BE')
            'shopperReference': shopper_reference,
            'channel': 'Web',
        }
        payment_methods_data = provider_sudo._adyen_make_request(
            url_field_name='adyen_checkout_api_url',
            endpoint='/paymentMethods',
            payload=data,
            method='POST'
        )
        _logger.info("paymentMethods request response:\n%s", pprint.pformat(payment_methods_data))
        return {
            'payment_methods_data': payment_methods_data,
            'amount_formatted': amount,
            'country_code': partner_country_code
        }

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
            'additionalData': {
                'allow3DS2': True
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
            url_field_name='adyen_checkout_api_url',
            endpoint='/payments',
            payload=data,
            method='POST',
            idempotency_key=idempotency_key,
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

    @http.route('/payment/adyen/payment_details', type='json', auth='public')
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
            url_field_name='adyen_checkout_api_url',
            endpoint='/payments/details',
            payload=payment_details,
            method='POST'
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

    @http.route(_webhook_url, type='json', auth='public')
    def adyen_webhook(self):
        """ Process the data sent by Adyen to the webhook based on the event code.

        See https://docs.adyen.com/development-resources/webhooks/understand-notifications for the
        exhaustive list of event codes.

        :return: The '[accepted]' string to acknowledge the notification
        :rtype: str
        """
        data = request.dispatcher.jsonrequest
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
                else:
                    continue  # Don't handle unsupported event codes and failed events
                try:
                    # Handle the notification data as if they were feedback of a S2S payment request
                    tx_sudo._handle_notification_data('adyen', notification_data)
                except ValidationError:  # Acknowledge the notification to avoid getting spammed
                    _logger.exception(
                        "unable to handle the notification data;skipping to acknowledge"
                    )

        return '[accepted]'  # Acknowledge the notification

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

import logging
import re

import requests

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError

from odoo.addons.payment_adyen.const import API_ENDPOINT_VERSIONS

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
    adyen_checkout_api_url = fields.Char(
        string="Checkout API URL", help="The base URL for the Checkout API endpoints",
        required_if_provider='adyen')
    adyen_recurring_api_url = fields.Char(
        string="Recurring API URL", help="The base URL for the Recurring API endpoints",
        required_if_provider='adyen')

    #=== CRUD METHODS ===#

    @api.model_create_multi
    def create(self, values_list):
        for values in values_list:
            self._adyen_trim_api_urls(values)
        return super().create(values_list)

    def write(self, values):
        self._adyen_trim_api_urls(values)
        return super().write(values)

    @api.model
    def _adyen_trim_api_urls(self, values):
        """ Remove the version and the endpoint from the url of Adyen API fields.

        :param dict values: The create or write values
        :return: None
        """
        for field_name in ('adyen_checkout_api_url', 'adyen_recurring_api_url'):
            if values.get(field_name):  # Test the value in case we're duplicating a provider
                values[field_name] = re.sub(r'[vV]\d+(/.*)?', '', values[field_name])

    #=== COMPUTE METHODS ===#

    def _compute_feature_support_fields(self):
        """ Override of `payment` to enable additional features. """
        super()._compute_feature_support_fields()
        self.filtered(lambda p: p.code == 'adyen').update({
            'support_manual_capture': True,
            'support_refund': 'partial',
            'support_tokenization': True,
        })

    #=== BUSINESS METHODS ===#

    def _adyen_make_request(
        self, url_field_name, endpoint, endpoint_param=None, payload=None, method='POST', idempotency_key=None
    ):
        """ Make a request to Adyen API at the specified endpoint.

        Note: self.ensure_one()

        :param str url_field_name: The name of the field holding the base URL for the request
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

        def _build_url(_base_url, _version, _endpoint):
            """ Build an API URL by appending the version and endpoint to a base URL.

            The final URL follows this pattern: `<_base>/V<_version>/<_endpoint>`.

            :param str _base_url: The base of the url prefixed with `https://`
            :param int _version: The version of the endpoint
            :param str _endpoint: The endpoint of the URL.
            :return: The final URL
            :rtype: str
            """
            _base = _base_url.rstrip('/')  # Remove potential trailing slash
            _endpoint = _endpoint.lstrip('/')  # Remove potential leading slash
            return f'{_base}/V{_version}/{_endpoint}'

        self.ensure_one()

        base_url = self[url_field_name]  # Restrict request URL to the stored API URL fields
        version = API_ENDPOINT_VERSIONS[endpoint]
        endpoint = endpoint if not endpoint_param else endpoint.format(endpoint_param)
        url = _build_url(base_url, version, endpoint)
        headers = {'X-API-Key': self.adyen_api_key}
        if method == 'POST' and idempotency_key:
            headers['idempotency-key'] = idempotency_key
        try:
            response = requests.request(method, url, json=payload, headers=headers, timeout=60)
            response.raise_for_status()
        except requests.exceptions.ConnectionError:
            _logger.exception("unable to reach endpoint at %s", url)
            raise ValidationError("Adyen: " + _("Could not establish the connection to the API."))
        except requests.exceptions.HTTPError as error:
            _logger.exception(
                "invalid API request at %s with data %s: %s", url, payload, error.response.text
            )
            raise ValidationError("Adyen: " + _("The communication with the API failed."))
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

from odoo import _, api, fields, models
from odoo.exceptions import UserError, ValidationError

from odoo.addons.payment import utils as payment_utils
from odoo.addons.payment_adyen import utils as adyen_utils
from odoo.addons.payment_adyen.const import CURRENCY_DECIMALS, RESULT_CODES_MAPPING

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
            self.amount, self.currency_id, CURRENCY_DECIMALS.get(self.currency_id.name)
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
            self.amount, self.currency_id, CURRENCY_DECIMALS.get(self.currency_id.name)
        )
        data = {
            'merchantAccount': self.provider_id.adyen_merchant_account,
            'amount': {
                'value': converted_amount,
                'currency': self.currency_id.name,
            },
            'reference': self.reference,
            'paymentMethod': {
                'recurringDetailReference': self.token_id.provider_ref,
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
        response_content = self.provider_id._adyen_make_request(
            url_field_name='adyen_checkout_api_url',
            endpoint='/payments',
            payload=data,
            method='POST',
            idempotency_key=payment_utils.generate_idempotency_key(
                self, scope='payment_request_token'
            )
        )

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
            arbitrary_decimal_number=CURRENCY_DECIMALS.get(refund_tx.currency_id.name)
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
            url_field_name='adyen_checkout_api_url',
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

    def _send_capture_request(self):
        """ Override of payment to send a capture request to Adyen.

        Note: self.ensure_one()

        :return: None
        """
        super()._send_capture_request()
        if self.provider_code != 'adyen':
            return

        converted_amount = payment_utils.to_minor_currency_units(
            self.amount, self.currency_id, CURRENCY_DECIMALS.get(self.currency_id.name)
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
            url_field_name='adyen_checkout_api_url',
            endpoint='/payments/{}/captures',
            endpoint_param=self.provider_reference,
            payload=data,
            method='POST',
        )
        _logger.info("capture request response:\n%s", pprint.pformat(response_content))

        # Handle the capture request response
        status = response_content.get('status')
        if status == 'received':
            self._log_message_on_linked_documents(_(
                "The capture of the transaction with reference %s has been requested (%s).",
                self.reference, self.provider_id.name
            ))

    def _send_void_request(self):
        """ Override of payment to send a void request to Adyen.

        Note: self.ensure_one()

        :return: None
        """
        super()._send_void_request()
        if self.provider_code != 'adyen':
            return

        data = {
            'merchantAccount': self.provider_id.adyen_merchant_account,
            'reference': self.reference,
        }
        response_content = self.provider_id._adyen_make_request(
            url_field_name='adyen_checkout_api_url',
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
        elif event_code in ['CAPTURE', 'CANCELLATION']:
            # The capture/void may be initiated from Adyen, so we can't trust the reference.
            # We find the transaction based on the original provider reference since Adyen will have
            # two different references: one for the original transaction and one for the capture.
            tx = self.search(
                [('provider_reference', '=', source_reference), ('provider_code', '=', 'adyen')]
            )
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
                    tx = self._adyen_create_refund_tx_from_notification_data(
                        source_tx, notification_data
                    )
                else:  # The refund was initiated for an unknown source transaction
                    pass  # Don't do anything with the refund notification

        if not tx:
            raise ValidationError(
                "Adyen: " + _("No transaction found matching reference %s.", reference)
            )
        return tx

    def _adyen_create_refund_tx_from_notification_data(self, source_tx, notification_data):
        """ Create a refund transaction based on Adyen data.

        :param recordset source_tx: The source transaction for which a refund is initiated, as a
                                    `payment.transaction` recordset
        :param dict notification_data: The notification data sent by the provider
        :return: The newly created refund transaction
        :rtype: recordset of `payment.transaction`
        :raise: ValidationError if inconsistent data were received
        """
        refund_provider_reference = notification_data.get('pspReference')
        amount_to_refund = notification_data.get('amount', {}).get('value')
        if not refund_provider_reference or not amount_to_refund:
            raise ValidationError(
                "Adyen: " + _("Received refund data with missing transaction values")
            )

        converted_amount = payment_utils.to_major_currency_units(
            amount_to_refund, source_tx.currency_id
        )
        return source_tx._create_refund_transaction(
            amount_to_refund=converted_amount, provider_reference=refund_provider_reference
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

        # Handle the provider reference. If the event code is 'CAPTURE' or 'CANCELLATION', we
        # discard the pspReference as it is different from the original pspReference of the tx.
        if 'pspReference' in notification_data and event_code in ['AUTHORISATION', 'REFUND']:
            self.provider_reference = notification_data.get('pspReference')

        # Handle the payment state
        payment_state = notification_data.get('resultCode')
        refusal_reason = notification_data.get('refusalReason') or notification_data.get('reason')
        if not payment_state:
            raise ValidationError("Adyen: " + _("Received data with missing payment state."))

        if payment_state in RESULT_CODES_MAPPING['pending']:
            self._set_pending()
        elif payment_state in RESULT_CODES_MAPPING['done']:
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
        elif payment_state in RESULT_CODES_MAPPING['cancel']:
            self._set_canceled()
        elif payment_state in RESULT_CODES_MAPPING['error']:
            if event_code in ['AUTHORISATION', 'REFUND']:
                _logger.warning(
                    "the transaction with reference %s underwent an error. reason: %s",
                    self.reference, refusal_reason
                )
                self._set_error(
                    _("An error occurred during the processing of your payment. Please try again.")
                )
            elif event_code == 'CANCELLATION':
                _logger.warning(
                    "the void of the transaction with reference %s failed. reason: %s",
                    self.reference, refusal_reason
                )
                self._log_message_on_linked_documents(
                    _("The void of the transaction with reference %s failed.", self.reference)
                )
            else:  # 'CAPTURE'
                _logger.warning(
                    "the capture of the transaction with reference %s failed. reason: %s",
                    self.reference, refusal_reason
                )
                self._log_message_on_linked_documents(
                    _("The capture of the transaction with reference %s failed.", self.reference)
                )
        elif payment_state in RESULT_CODES_MAPPING['refused']:
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
            'payment_details': additional_data.get('cardSummary'),
            'partner_id': self.partner_id.id,
            'provider_ref': additional_data['recurring.recurringDetailReference'],
            'adyen_shopper_reference': additional_data['recurring.shopperReference'],
            'verified': True,  # The payment is authorized, so the payment method is valid
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
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#7CC098"/><stop offset="100%" stop-color="#5F8A71"/></linearGradient><path id="d" d="M19.25 45h2.714v2.212c0 .188.152.343.339.343h26.394a.342.342 0 0 0 .339-.343V37.5H38.5V31h10.536v-3.212a.342.342 0 0 0-.339-.343H38.5v-2.742h10.536c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742V45zm-3.79 10.642h-1.037V55h2.833v.642h-1.037v2.832h-.76v-2.832zM17.422 55h1.07l.809 2.389h.01L20.075 55h1.07v3.474h-.711v-2.462h-.01l-.847 2.462h-.586l-.848-2.438h-.01v2.438h-.711V55zm13.58-34.853a4.263 4.263 0 0 1 4.281 4.28v17.004H18.281A4.263 4.263 0 0 1 14 37.151v-4.757a4.263 4.263 0 0 1 4.28-4.28h4.638v6.777c0 .595.476 1.19 1.19 1.19h2.377v-9.394c0-.595-.475-1.19-1.189-1.19h-10.94v-5.35h16.648z"/><path id="e" d="M19.25 43h2.714v2.212c0 .188.152.343.339.343h26.394a.342.342 0 0 0 .339-.343V35.5H38.5V29h10.536v-3.212a.342.342 0 0 0-.339-.343H38.5v-2.742h10.536c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742V43zm-3.79 10.642h-1.037V53h2.833v.642h-1.037v2.832h-.76v-2.832zM17.422 53h1.07l.809 2.389h.01L20.075 53h1.07v3.474h-.711v-2.462h-.01l-.847 2.462h-.586l-.848-2.438h-.01v2.438h-.711V53zm13.58-34.853a4.263 4.263 0 0 1 4.281 4.28v17.004H18.281A4.263 4.263 0 0 1 14 35.151v-4.757a4.263 4.263 0 0 1 4.28-4.28h4.638v6.777c0 .595.476 1.19 1.19 1.19h2.377v-9.394c0-.595-.475-1.19-1.189-1.19h-10.94v-5.35h16.648z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M4 69c-2 0-4-1-4-4V33.916L14.423 18.5 35 25.5l3.5-2.797L48 23l3.576 1.968-.236 22.01L39.224 69H4z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: static\src\js\payment_form.js

```javascript
odoo.define('payment_adyen.payment_form', require => {
    'use strict';

    const core = require('web.core');
    const { loadJS, loadCSS } = require('@web/core/assets');
    const { pyToJsLocale } = require('@web/core/l10n/utils');
    const checkoutForm = require('payment.checkout_form');
    const manageForm = require('payment.manage_form');

    const _t = core._t;

    const adyenMixin = {

        //--------------------------------------------------------------------------
        // Private
        //--------------------------------------------------------------------------

        /**
         * Handle the additional details event of the Adyen drop-in.
         *
         * @private
         * @param {object} state - The state of the drop-in
         * @param {object} dropin - The drop-in
         * @param {object} actions - The possible action handlers to call
         * @return {Promise}
         */
        _dropinOnAdditionalDetails: function (state, dropin, actions) {
            return this._rpc({
                route: '/payment/adyen/payment_details',
                params: {
                    'provider_id': dropin.providerId,
                    'reference': this.adyenDropin.reference,
                    'payment_details': state.data,
                },
            }).then(paymentResponse => {
                if (!paymentResponse.resultCode) {
                    actions.reject();
                    return;
                }
                actions.resolve(paymentResponse)
            }).guardedCatch((error) => {
                error.event.preventDefault();
                this._displayError(
                    _t("Server Error"),
                    _t("We are not able to process your payment."),
                    error.message.data.message
                );
            });
        },

        /**
        * Called when the payment is completed or failed.
        *
        * @private
        * @param {object} state
        * @param {object} dropin
        * @return {void}
        */
        _dropinOnPaymentResolved: function (state, dropin) {
            window.location = '/payment/status';
        },

        /**
         * Handle the error event of the Adyen drop-in.
         *
         * @private
         * @param {object} error - The error in the drop-in
         * @return {undefined}
         */
        _dropinOnError: function (error='') {
            this._displayError(
                _t("Incorrect Payment Details"),
                _t("Please verify your payment details."),
            );
        },

        /**
         * Handle the submit event of the Adyen drop-in and initiate the payment.
         *
         * @private
         * @param {object} state - The state of the drop-in
         * @param {object} dropin - The drop-in
         * @param {object} actions - The possible action handlers to call
         * @return {Promise}
         */
        _dropinOnSubmit: function (state, dropin, actions) {
            // Create the transaction and retrieve the processing values
            return this._rpc({
                route: this.txContext.transactionRoute,
                params: this._prepareTransactionRouteParams('adyen', dropin.providerId, 'direct'),
            }).then(processingValues => {
                this.adyenDropin.reference = processingValues.reference; // Store final reference
                // Initiate the payment
                return this._rpc({
                    route: '/payment/adyen/payments',
                    params: {
                        'provider_id': dropin.providerId,
                        'reference': processingValues.reference,
                        'converted_amount': processingValues.converted_amount,
                        'currency_id': processingValues.currency_id,
                        'partner_id': processingValues.partner_id,
                        'payment_method': state.data.paymentMethod,
                        'access_token': processingValues.access_token,
                        'browser_info': state.data.browserInfo,
                    },
                });
            }).then(paymentResponse => {
                if (!paymentResponse.resultCode) {
                    actions.reject();
                    return;
                }
                if (paymentResponse.action && paymentResponse.action.type !== 'redirect') {
                    this._hideInputs(); // Only the inputs of the inline form should be used
                    $('body').unblock(); // The page is blocked at this point, unblock it
                }
                actions.resolve(paymentResponse)
            }).guardedCatch((error) => {
                error.event.preventDefault();
                this._displayError(
                    _t("Server Error"),
                    _t("We are not able to process your payment."),
                    error.message.data.message
                );
            });
        },

        /**
         * Prepare the inline form of Adyen for direct payment.
         *
         * @override method from payment.payment_form_mixin
         * @private
         * @param {string} code - The code of the selected payment option's provider
         * @param {number} paymentOptionId - The id of the selected payment option
         * @param {string} flow - The online payment flow of the selected payment option
         * @return {Promise}
         */
        _prepareInlineForm: function (code, paymentOptionId, flow) {
            if (code !== 'adyen') {
                return this._super(...arguments);
            }

            // Check if instantiation of the drop-in is needed
            if (flow === 'token') {
                return Promise.resolve(); // No drop-in for tokens
            } else if (this.adyenDropin && this.adyenDropin.providerId === paymentOptionId) {
                this._setPaymentFlow('direct'); // Overwrite the flow even if no re-instantiation
                return Promise.resolve(); // Don't re-instantiate if already done for this provider
            }

            // Overwrite the flow of the select payment option
            this._setPaymentFlow('direct');

            // Get public information on the provider (state, client_key)
            return this._rpc({
                route: '/payment/adyen/provider_info',
                params: {
                    'provider_id': paymentOptionId,
                },
            }).then(providerInfo => {
                // Get the available payment methods
                return this._rpc({
                    route: '/payment/adyen/payment_methods',
                    params: {
                        'provider_id': paymentOptionId,
                        'partner_id': parseInt(this.txContext.partnerId),
                        'amount': this.txContext.amount
                            ? parseFloat(this.txContext.amount)
                            : undefined,
                        'currency_id': this.txContext.currencyId
                            ? parseInt(this.txContext.currencyId)
                            : undefined,
                    },
                }).then(async (paymentMethodsResult) => {
                    // Instantiate the drop-in
                    const configuration = {
                        paymentMethodsResponse: paymentMethodsResult['payment_methods_data'],
                        amount: paymentMethodsResult['amount_formatted'],
                        clientKey: providerInfo.client_key,
                        locale: pyToJsLocale(this._getContext().lang || 'en-US'),
                        countryCode: paymentMethodsResult['country_code'],
                        environment: providerInfo.state === 'enabled' ? 'live' : 'test',
                        showPayButton: false,
                        onAdditionalDetails: this._dropinOnAdditionalDetails.bind(this),
                        onPaymentCompleted: this._dropinOnPaymentResolved.bind(this),
                        onPaymentFailed: this._dropinOnPaymentResolved.bind(this),
                        onError: this._dropinOnError.bind(this),
                        onSubmit: this._dropinOnSubmit.bind(this),
                    };

                    const script = this.el.parentElement.querySelector(
                        'script[src="https://checkoutshopper-live.adyen.com/checkoutshopper/sdk/4.7.3/adyen.js"]'
                    );
                    if (script) script.remove();
                    const link = this.el.parentElement.querySelector(
                        'link[href="https://checkoutshopper-live.adyen.com/checkoutshopper/sdk/4.7.3/adyen.css"]'
                    );
                    if (link) link.remove();
                    await loadJS('https://checkoutshopper-live.adyen.com/checkoutshopper/sdk/6.9.0/adyen.js');
                    await loadCSS('https://checkoutshopper-live.adyen.com/checkoutshopper/sdk/6.9.0/adyen.css');

                    const { AdyenCheckout, Dropin } = window.AdyenWeb;
                    AdyenCheckout(configuration)
                        .then(checkout => {
                            this.adyenDropin = new Dropin(checkout, {
                                paymentMethodsConfiguration: {
                                    card: {
                                        hasHolderName: true,
                                        holderNameRequired: true,
                                    },
                                    paypal: {
                                        blockPayPalCreditButton: true,
                                        blockPayPalPayLaterButton: true,
                                        showPayButton: true,
                                        style: {
                                            disableMaxWidth: true
                                        },
                                    },
                                },
                                openFirstStoredPaymentMethod: false,
                                showStoredPaymentMethods: false,
                                disableFinalAnimation: true,
                                onSelect: component => {
                                    if (component.props.name === "PayPal") {
                                        this.txContext.tokenizationRequested = false;
                                        // Hide Pay button
                                        this._hideInputs();
                                    }
                                    else {
                                        this._showInputs();
                                    }
                                }
                            });
                            this.adyenDropin.mount(`#o_adyen_dropin_container_${paymentOptionId}`);
                            this.adyenDropin.providerId = paymentOptionId;
                        });
            }).guardedCatch((error) => {
                error.event.preventDefault();
                this._displayError(
                    _t("Server Error"),
                    _t("An error occurred when displayed this payment form."),
                    error.message.data.message
                );
            });
    })},

        /**
         * Trigger the payment processing by submitting the drop-in.
         *
         * @override method from payment.payment_form_mixin
         * @private
         * @param {string} provider - The provider of the payment option's provider
         * @param {number} paymentOptionId - The id of the payment option handling the transaction
         * @param {string} flow - The online payment flow of the transaction
         * @return {Promise}
         */
        async _processPayment(provider, paymentOptionId, flow) {
            if (provider !== 'adyen' || flow === 'token') {
                return this._super(...arguments); // Tokens are handled by the generic flow
            }
            if (this.adyenDropin === undefined) { // The drop-in has not been properly instantiated
                this._displayError(
                    _t("Server Error"), _t("We are not able to process your payment.")
                );
            } else {
                await this.adyenDropin.submit();
                if (!this.adyenDropin.isValid) {
                    this._dropinOnError();
                }
            }
        },

    };

    checkoutForm.include(adyenMixin);
    manageForm.include(adyenMixin);
});

```

## File: views\payment_adyen_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="inline_form">
        <div t-attf-id="o_adyen_dropin_container_{{provider_id}}" class="o_adyen_dropin"/>
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
                <group attrs="{'invisible': [('code', '!=', 'adyen')]}">
                    <field name="adyen_merchant_account" attrs="{'required':[('code', '=', 'adyen'), ('state', '!=', 'disabled')]}"/>
                    <field name="adyen_api_key" attrs="{'required':[('code', '=', 'adyen'), ('state', '!=', 'disabled')]}" password="True"/>
                    <field name="adyen_client_key" attrs="{'required':[('code', '=', 'adyen'), ('state', '!=', 'disabled')]}"/>
                    <field name="adyen_hmac_key" attrs="{'required':[('code', '=', 'adyen'), ('state', '!=', 'disabled')]}" password="True"/>
                    <field name="adyen_checkout_api_url" attrs="{'required':[('code', '=', 'adyen'), ('state', '!=', 'disabled')]}"/>
                    <field name="adyen_recurring_api_url" attrs="{'required':[('code', '=', 'adyen'), ('state', '!=', 'disabled')]}"/>
                </group>
            </group>
            <group name="provider_config" position='before'>
                <div attrs="{'invisible':['|', ('code', '!=', 'adyen'), ('capture_manually', '=', False)]}"
                     class="alert alert-warning"
                     role="alert">
                    <strong>Warning:</strong> To capture the amount manually, you also need to set
                    the Capture Delay to manual on your Adyen account settings.
                    <a href="https://www.odoo.com/documentation/16.0/applications/finance/payment_providers/adyen.html#place-a-hold-on-a-card"
                       title="Learn More"
                       target="_blank">Learn More</a>
                </div>
            </group>
        </field>
    </record>

</odoo>

```

## File: views\payment_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="checkout" inherit_id="payment.checkout">
        <xpath expr="." position="inside">
            <t t-call="payment_adyen.sdk_assets"/>
        </xpath>
    </template>

    <template id="manage" inherit_id="payment.manage">
        <xpath expr="." position="inside">
            <t t-call="payment_adyen.sdk_assets"/>
        </xpath>
    </template>

    <template id="sdk_assets">
        <script src="https://checkoutshopper-live.adyen.com/checkoutshopper/sdk/4.7.3/adyen.js"
                integrity="sha384-YiT4BfPwbplFwUnpDjm2rmWCvpzdi6+l+1E+J9JKTB3CYyKbbwJ8ghkUheQf79X9"
                crossorigin="anonymous">
        </script>
        <link rel="stylesheet"
              href="https://checkoutshopper-live.adyen.com/checkoutshopper/sdk/4.7.3/adyen.css"
              integrity="sha384-X1zQUhO5NGWdMQmDcuv2kyQK65QR7/VJtNthEImZm7jOvOEicQrnVijI0n9DcHkF"
              crossorigin="anonymous">
        </link>
    </template>

</odoo>

```

