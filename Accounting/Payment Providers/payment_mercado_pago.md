# Odoo Module: payment_mercado_pago

Category: Accounting/Payment Providers

This file contains the source code of the Odoo module.

## File: const.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _


# Currency codes of the currencies supported by Mercado Pago in ISO 4217 format.
# See https://api.mercadopago.com/currencies. Last seen online: 2024-10-29.
SUPPORTED_CURRENCIES = [
    'ARS',  # Argentinian Peso
    'BOB',  # Boliviano
    'BRL',  # Real
    'CLF',  # Fomento Unity
    'CLP',  # Chilean Peso
    'COP',  # Colombian Peso
    'CRC',  # Colon
    'CUC',  # Cuban Convertible Peso
    'CUP',  # Cuban Peso
    'DOP',  # Dominican Peso
    'EUR',  # Euro
    'GTQ',  # Guatemalan Quetzal
    'HNL',  # Lempira
    'MXN',  # Mexican Peso
    'NIO',  # Cordoba
    'PAB',  # Balboa
    'PEN',  # Sol
    'PYG',  # Guarani
    'USD',  # US Dollars
    'UYU',  # Uruguayan Peso
    'VEF',  # Strong Bolivar
    'VES',  # Sovereign Bolivar
]

# Set of currencies where Mercado Pago's minor units deviates from the ISO 4217 standard.
# See https://www.six-group.com/dam/download/financial-information/data-center/iso-currrency/lists/list-one.xls
# vs. https://api.mercadopago.com/currencies. Last seen online: 2024-10-29.
CURRENCY_DECIMALS = {
    'COP': 0,
    'HNL': 0,
    'NIO': 0,
}

# The codes of the payment methods to activate when Mercado Pago is activated.
DEFAULT_PAYMENT_METHODS_CODES = [
    # Primary payment methods.
    'card',
    # Brand payment methods.
    'visa',
    'mastercard',
    'argencard',
    'ceconsud',
    'cordobesa',
    'codensa',
    'lider',
    'magna',
    'naranja',
    'nativa',
    'oca',
    'presto',
    'tarjeta_mercadopago',
    'shopping',
    'elo',
    'hipercard',
]

# Mapping of payment method codes to Mercado Pago codes.
PAYMENT_METHODS_MAPPING = {
    'card': 'debit_card,credit_card,prepaid_card',
    'paypal': 'digital_wallet',
}

# Mapping of transaction states to Mercado Pago payment statuses.
# See https://www.mercadopago.com.mx/developers/en/reference/payments/_payments_id/get.
TRANSACTION_STATUS_MAPPING = {
    'pending': ('pending', 'in_process', 'in_mediation', 'authorized'),
    'done': ('approved', 'refunded'),
    'canceled': ('cancelled', 'null'),
    'error': ('rejected',),
}

# Mapping of error states to Mercado Pago error messages.
# See https://www.mercadopago.com.ar/developers/en/docs/checkout-api/response-handling/collection-results
ERROR_MESSAGE_MAPPING = {
    'accredited': _(
        "Your payment has been credited. In your summary you will see the charge as a statement "
        "descriptor."
    ),
    'pending_contingency': _(
        "We are processing your payment. Don't worry, in less than 2 business days, we will notify "
        "you by e-mail if your payment has been credited."
    ),
    'pending_review_manual': _(
        "We are processing your payment. Don't worry, less than 2 business days we will notify you "
        "by e-mail if your payment has been credited or if we need more information."
    ),
    'cc_rejected_bad_filled_card_number': _("Check the card number."),
    'cc_rejected_bad_filled_date': _("Check expiration date."),
    'cc_rejected_bad_filled_other': _("Check the data."),
    'cc_rejected_bad_filled_security_code': _("Check the card security code."),
    'cc_rejected_blacklist': _("We were unable to process your payment, please use another card."),
    'cc_rejected_call_for_authorize': _("You must authorize the payment with this card."),
    'cc_rejected_card_disabled': _(
        "Call your card issuer to activate your card or use another payment method. The phone "
        "number is on the back of your card."
    ),
    'cc_rejected_card_error': _(
        "We were unable to process your payment, please check your card information."
    ),
    'cc_rejected_duplicated_payment': _(
        "You have already made a payment for that value. If you need to pay again, use another card"
        " or another payment method."
    ),
    'cc_rejected_high_risk': _(
        "We were unable to process your payment, please use another card."
    ),
    'cc_rejected_insufficient_amount': _("Your card has not enough funds."),
    'cc_rejected_invalid_installments': _(
        "This payment method does not process payments in installments."
    ),
    'cc_rejected_max_attempts': _(
        "You have reached the limit of allowed attempts. Choose another card or other means of "
        "payment."
    ),
    'cc_rejected_other_reason': _("Payment was not processed, use another card or contact issuer.")
}

```

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

from odoo.addons.payment import setup_provider, reset_payment_provider


def post_init_hook(env):
    setup_provider(env, 'mercado_pago')


def uninstall_hook(env):
    reset_payment_provider(env, 'mercado_pago')

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': "Payment Provider: Mercado Pago",
    'version': '1.0',
    'category': 'Accounting/Payment Providers',
    'sequence': 350,
    'summary': "A payment provider covering several countries in Latin America.",
    'description': " ",  # Non-empty string to avoid loading the README file.
    'depends': ['payment'],
    'data': [
        'views/payment_mercado_pago_templates.xml',
        'views/payment_provider_views.xml',

        'data/payment_provider_data.xml',  # Depends on views/payment_mercado_pago_templates.xml
    ],
    'post_init_hook': 'post_init_hook',
    'uninstall_hook': 'uninstall_hook',
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import pprint

from odoo import http
from odoo.exceptions import ValidationError
from odoo.http import request


_logger = logging.getLogger(__name__)


class MercadoPagoController(http.Controller):
    _return_url = '/payment/mercado_pago/return'
    _webhook_url = '/payment/mercado_pago/webhook'

    @http.route(_return_url, type='http', methods=['GET'], auth='public')
    def mercado_pago_return_from_checkout(self, **data):
        """ Process the notification data sent by Mercado Pago after redirection from checkout.

        :param dict data: The notification data.
        """
        # Handle the notification data.
        _logger.info("Handling redirection from Mercado Pago with data:\n%s", pprint.pformat(data))
        if data.get('payment_id') != 'null':
            request.env['payment.transaction'].sudo()._handle_notification_data(
                'mercado_pago', data
            )
        else:  # The customer cancelled the payment by clicking on the return button.
            pass  # Don't try to process this case because the payment id was not provided.

        # Redirect the user to the status page.
        return request.redirect('/payment/status')

    @http.route(
        f'{_webhook_url}/<reference>', type='http', auth='public', methods=['POST'], csrf=False
    )
    def mercado_pago_webhook(self, reference, **_kwargs):
        """ Process the notification data sent by Mercado Pago to the webhook.

        :param str reference: The transaction reference embedded in the webhook URL.
        :param dict _kwargs: The extra query parameters.
        :return: An empty string to acknowledge the notification.
        :rtype: str
        """
        data = request.get_json_data()
        _logger.info("Notification received from Mercado Pago with data:\n%s", pprint.pformat(data))

        # Mercado Pago sends two types of asynchronous notifications: webhook notifications and
        # IPNs which are very similar to webhook notifications but are sent later and contain less
        # information. Therefore, we filter the notifications we receive based on the 'action'
        # (type of event) key as it is not populated for IPNs, and we don't want to process the
        # other types of events.
        if data.get('action') in ('payment.created', 'payment.updated'):
            # Handle the notification data.
            try:
                payment_id = data.get('data', {}).get('id')
                request.env['payment.transaction'].sudo()._handle_notification_data(
                    'mercado_pago', {'external_reference': reference, 'payment_id': payment_id}
                )  # Use 'external_reference' as the reference key like in the redirect data.
            except ValidationError:  # Acknowledge the notification to avoid getting spammed.
                _logger.exception("Unable to handle the notification data; skipping to acknowledge")
        return ''  # Acknowledge the notification.

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\neutralize.sql

```sql
-- disable mercado_pago payment provider
UPDATE payment_provider
   SET mercado_pago_access_token = NULL;

```

## File: data\payment_provider_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="payment.payment_provider_mercado_pago" model="payment.provider">
        <field name="code">mercado_pago</field>
        <field name="redirect_form_view_id" ref="redirect_form"/>
    </record>

</odoo>

```

## File: models\payment_provider.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import pprint

import requests
from werkzeug import urls

from odoo import _, fields, models
from odoo.exceptions import ValidationError

from odoo.addons.payment_mercado_pago import const


_logger = logging.getLogger(__name__)


class PaymentProvider(models.Model):
    _inherit = 'payment.provider'

    code = fields.Selection(
        selection_add=[('mercado_pago', "Mercado Pago")], ondelete={'mercado_pago': 'set default'}
    )
    mercado_pago_access_token = fields.Char(
        string="Mercado Pago Access Token",
        required_if_provider='mercado_pago',
        groups='base.group_system',
    )

    # === BUSINESS METHODS === #

    def _get_supported_currencies(self):
        """ Override of `payment` to return the supported currencies. """
        supported_currencies = super()._get_supported_currencies()
        if self.code == 'mercado_pago':
            supported_currencies = supported_currencies.filtered(
                lambda c: c.name in const.SUPPORTED_CURRENCIES
            )
        return supported_currencies

    def _mercado_pago_make_request(self, endpoint, payload=None, method='POST'):
        """ Make a request to Mercado Pago API at the specified endpoint.

        Note: self.ensure_one()

        :param str endpoint: The endpoint to be reached by the request.
        :param dict payload: The payload of the request.
        :param str method: The HTTP method of the request.
        :return The JSON-formatted content of the response.
        :rtype: dict
        :raise ValidationError: If an HTTP error occurs.
        """
        self.ensure_one()

        url = urls.url_join('https://api.mercadopago.com', endpoint)
        headers = {
            'Authorization': f'Bearer {self.mercado_pago_access_token}',
            'X-Platform-Id': 'dev_cdf1cfac242111ef9fdebe8d845d0987',
        }
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
                    try:
                        response_content = response.json()
                        error_code = response_content.get('error')
                        error_message = response_content.get('message')
                        raise ValidationError("Mercado Pago: " + _(
                            "The communication with the API failed. Mercado Pago gave us the"
                            " following information: '%s' (code %s)", error_message, error_code
                        ))
                    except ValueError:  # The response can be empty when the access token is wrong.
                        raise ValidationError("Mercado Pago: " + _(
                            "The communication with the API failed. The response is empty. Please"
                            " verify your access token."
                        ))
        except (requests.exceptions.ConnectionError, requests.exceptions.Timeout):
            _logger.exception("Unable to reach endpoint at %s", url)
            raise ValidationError(
                "Mercado Pago: " + _("Could not establish the connection to the API.")
            )
        return response.json()

    def _get_default_payment_method_codes(self):
        """ Override of `payment` to return the default payment method codes. """
        default_codes = super()._get_default_payment_method_codes()
        if self.code != 'mercado_pago':
            return default_codes
        return const.DEFAULT_PAYMENT_METHODS_CODES

```

## File: models\payment_transaction.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import pprint
from urllib.parse import quote as url_quote

from werkzeug import urls

from odoo import _, api, models
from odoo.exceptions import ValidationError
from odoo.tools import float_round

from odoo.addons.payment_mercado_pago import const
from odoo.addons.payment_mercado_pago.controllers.main import MercadoPagoController


_logger = logging.getLogger(__name__)


class PaymentTransaction(models.Model):
    _inherit = 'payment.transaction'

    def _get_specific_rendering_values(self, processing_values):
        """ Override of `payment` to return Mercado Pago-specific rendering values.

        Note: self.ensure_one() from `_get_rendering_values`.

        :param dict processing_values: The generic and specific processing values of the transaction
        :return: The dict of provider-specific processing values.
        :rtype: dict
        """
        res = super()._get_specific_rendering_values(processing_values)
        if self.provider_code != 'mercado_pago':
            return res

        # Initiate the payment and retrieve the payment link data.
        payload = self._mercado_pago_prepare_preference_request_payload()
        _logger.info(
            "Sending '/checkout/preferences' request for link creation:\n%s",
            pprint.pformat(payload),
        )
        api_url = self.provider_id._mercado_pago_make_request(
            '/checkout/preferences', payload=payload
        )['init_point' if self.provider_id.state == 'enabled' else 'sandbox_init_point']

        # Extract the payment link URL and params and embed them in the redirect form.
        parsed_url = urls.url_parse(api_url)
        url_params = urls.url_decode(parsed_url.query)
        rendering_values = {
            'api_url': api_url,
            'url_params': url_params,  # Encore the params as inputs to preserve them.
        }
        return rendering_values

    def _mercado_pago_prepare_preference_request_payload(self):
        """ Create the payload for the preference request based on the transaction values.

        :return: The request payload.
        :rtype: dict
        """
        base_url = self.provider_id.get_base_url()
        return_url = urls.url_join(base_url, MercadoPagoController._return_url)
        sanitized_reference = url_quote(self.reference)
        webhook_url = urls.url_join(
            base_url, f'{MercadoPagoController._webhook_url}/{sanitized_reference}'
        )  # Append the reference to identify the transaction from the webhook notification data.

        unit_price = self.amount
        decimal_places = const.CURRENCY_DECIMALS.get(self.currency_id.name)
        if decimal_places is not None:
            unit_price = float_round(unit_price, decimal_places, rounding_method='DOWN')

        return {
            'auto_return': 'all',
            'back_urls': {
                'success': return_url,
                'pending': return_url,
                'failure': return_url,
            },
            'external_reference': self.reference,
            'items': [{
                'title': self.reference,
                'quantity': 1,
                'currency_id': self.currency_id.name,
                'unit_price': unit_price,
            }],
            'notification_url': webhook_url,
            'payer': {
                'name': self.partner_name,
                'email': self.partner_email,
                'phone': {
                    'number': self.partner_phone,
                },
                'address': {
                    'zip_code': self.partner_zip,
                    'street_name': self.partner_address,
                },
            },
            'payment_methods': {
                'installments': 1,  # Prevent MP from proposing several installments for a payment.
            },
        }

    def _get_tx_from_notification_data(self, provider_code, notification_data):
        """ Override of `payment` to find the transaction based on Mercado Pago data.

        :param str provider_code: The code of the provider that handled the transaction.
        :param dict notification_data: The notification data sent by the provider.
        :return: The transaction if found.
        :rtype: recordset of `payment.transaction`
        :raise ValidationError: If inconsistent data were received.
        :raise ValidationError: If the data match no transaction.
        """
        tx = super()._get_tx_from_notification_data(provider_code, notification_data)
        if provider_code != 'mercado_pago' or len(tx) == 1:
            return tx

        reference = notification_data.get('external_reference')
        if not reference:
            raise ValidationError("Mercado Pago: " + _("Received data with missing reference."))

        tx = self.search([('reference', '=', reference), ('provider_code', '=', 'mercado_pago')])
        if not tx:
            raise ValidationError(
                "Mercado Pago: " + _("No transaction found matching reference %s.", reference)
            )
        return tx

    def _process_notification_data(self, notification_data):
        """ Override of `payment` to process the transaction based on Mercado Pago data.

        Note: self.ensure_one() from `_process_notification_data`

        :param dict notification_data: The notification data sent by the provider.
        :return: None
        :raise ValidationError: If inconsistent data were received.
        """
        super()._process_notification_data(notification_data)
        if self.provider_code != 'mercado_pago':
            return

        # Update the provider reference.
        payment_id = notification_data.get('payment_id')
        if not payment_id:
            raise ValidationError("Mercado Pago: " + _("Received data with missing payment id."))
        self.provider_reference = payment_id

        # Verify the notification data.
        verified_payment_data = self.provider_id._mercado_pago_make_request(
            f'/v1/payments/{self.provider_reference}', method='GET'
        )

        # Update the payment method.
        payment_method_type = verified_payment_data.get('payment_type_id', '')
        for odoo_code, mp_codes in const.PAYMENT_METHODS_MAPPING.items():
            if any(payment_method_type == mp_code for mp_code in mp_codes.split(',')):
                payment_method_type = odoo_code
                break
        payment_method = self.env['payment.method']._get_from_code(
            payment_method_type, mapping=const.PAYMENT_METHODS_MAPPING
        )
        # Fall back to "unknown" if the payment method is not found (and if "unknown" is found), as
        # the user might have picked a different payment method than on Odoo's payment form.
        if not payment_method:
            payment_method = self.env['payment.method'].search([('code', '=', 'unknown')], limit=1)
        self.payment_method_id = payment_method or self.payment_method_id

        # Update the payment state.
        payment_status = verified_payment_data.get('status')
        if not payment_status:
            raise ValidationError("Mercado Pago: " + _("Received data with missing status."))

        if payment_status in const.TRANSACTION_STATUS_MAPPING['pending']:
            self._set_pending()
        elif payment_status in const.TRANSACTION_STATUS_MAPPING['done']:
            self._set_done()
        elif payment_status in const.TRANSACTION_STATUS_MAPPING['canceled']:
            self._set_canceled()
        elif payment_status in const.TRANSACTION_STATUS_MAPPING['error']:
            status_detail = verified_payment_data.get('status_detail')
            _logger.warning(
                "Received data for transaction with reference %s with status %s and error code: %s",
                self.reference, payment_status, status_detail
            )
            error_message = self._mercado_pago_get_error_msg(status_detail)
            self._set_error(error_message)
        else:  # Classify unsupported payment status as the `error` tx state.
            _logger.warning(
                "Received data for transaction with reference %s with invalid payment status: %s",
                self.reference, payment_status
            )
            self._set_error(
                "Mercado Pago: " + _("Received data with invalid status: %s", payment_status)
            )

    @api.model
    def _mercado_pago_get_error_msg(self, status_detail):
        """ Return the error message corresponding to the payment status.

        :param str status_detail: The status details sent by the provider.
        :return: The error message.
        :rtype: str
        """
        return "Mercado Pago: " + const.ERROR_MESSAGE_MAPPING.get(
            status_detail, const.ERROR_MESSAGE_MAPPING['cc_rejected_other_reason']
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
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M43.105 4h.972V.818h1.108V0H42v.818h1.105V4Zm2.775 0h.858V1.453h.053L47.663 4h.555l.871-2.547h.056V4H50V0h-1.108l-.924 2.714h-.05L46.99 0h-1.11v4Z" fill="#D1D5DB"/><path d="M49 24.07C49 15.234 38.26 8 25 8S1 15.234 1 24.07V25c0 9.404 9.378 17 24 17 14.672 0 24-7.596 24-17v-.93Z" fill="#2D3277"/><path d="M48.143 24.07c0 8.32-10.337 15.089-23.042 15.089-12.706 0-23.042-6.77-23.042-15.088 0-8.32 10.336-15.089 23.042-15.089 12.706 0 23.042 6.77 23.042 15.089Z" fill="#52BAF9"/><path d="M17.437 19.317s-.252.258-.1.465c.352.465 1.462.723 2.621.465.656-.155 1.563-.878 2.37-1.55.908-.724 1.815-1.499 2.723-1.757.958-.31 1.563-.155 1.966-.052.454.155.958.465 1.815 1.085 1.563 1.189 7.916 6.873 9.026 7.855.857-.414 4.79-2.119 10.134-3.36-.454-2.893-2.168-5.58-4.79-7.75-3.63 1.55-8.118 2.377-12.504.207 0 0-2.37-1.137-4.74-1.085-3.479.103-4.94 1.601-6.554 3.255l-1.967 2.222Z" fill="#fff"/><path d="M37.605 26.447c-.05-.051-7.462-6.665-9.126-7.957-.958-.724-1.512-.93-2.067-1.034-.303-.051-.706 0-1.008.104-.757.206-1.816.93-2.723 1.653-.958.775-1.815 1.499-2.622 1.654-1.059.258-2.32-.052-2.874-.465a1.328 1.328 0 0 1-.504-.569c-.202-.516.201-.93.252-.981l2.017-2.274.706-.723c-.656.103-1.26.258-1.866.413-.756.207-1.462.413-2.168.413-.303 0-1.916-.258-2.219-.361-1.865-.517-3.479-1.034-5.899-2.17-2.874 2.221-4.84 4.96-5.395 8.008.404.104 1.11.31 1.362.362 6.554 1.499 8.571 3.049 8.974 3.359a2.074 2.074 0 0 1 1.614-.724 2.18 2.18 0 0 1 1.765.93c.353-.31.907-.568 1.563-.568.302 0 .605.052.958.155.756.258 1.109.775 1.31 1.24.253-.103.555-.206.908-.206s.706.103 1.11.258c1.21.517 1.411 1.757 1.31 2.687h.252c1.463 0 2.622 1.188 2.622 2.687 0 .465-.1.878-.302 1.24.403.207 1.361.723 2.269.62.706-.103.958-.31 1.058-.465.05-.103.152-.207.05-.31l-1.865-2.119s-.302-.31-.201-.413c.1-.103.302.052.454.155.957.827 2.067 2.015 2.067 2.015s.1.155.504.259c.353.051 1.008 0 1.462-.362.1-.103.252-.207.303-.31.453-.62-.05-1.24-.05-1.24l-2.169-2.48s-.302-.31-.202-.414c.101-.103.303.052.454.155.706.569 1.664 1.602 2.572 2.532.201.155 1.008.672 2.067-.052.655-.465.807-.981.756-1.395-.05-.516-.454-.93-.454-.93l-2.924-3.048s-.302-.259-.202-.414c.101-.103.303.052.454.155.958.827 3.48 3.204 3.48 3.204.05 0 .907.672 2.016-.052.403-.258.655-.62.655-1.085.05-.672-.504-1.137-.504-1.137Z" fill="#fff"/><path d="M23.336 30.323c-.454 0-.958.258-1.009.258-.05 0 0-.207.05-.31.051-.103.656-1.963-.806-2.635-1.11-.517-1.815.051-2.017.31-.05.051-.1.051-.1 0 0-.31-.152-1.24-1.16-1.55-1.412-.465-2.27.568-2.521.93-.101-.827-.756-1.447-1.614-1.447-.907 0-1.613.723-1.613 1.653s.706 1.654 1.613 1.654c.454 0 .807-.155 1.11-.465v.052c-.05.413-.202 1.911 1.31 2.48.606.258 1.11.052 1.564-.259.15-.103.15-.051.15.052-.05.362 0 1.188 1.16 1.654.858.361 1.362 0 1.664-.31.152-.156.202-.104.202.103.05 1.085.958 1.963 2.017 1.963 1.11 0 2.017-.93 2.017-2.067 0-1.136-.908-2.066-2.017-2.066Z" fill="#fff"/><path d="M37.907 25.672c-2.269-2.015-7.512-6.717-8.975-7.802-.806-.62-1.36-.982-1.865-1.085-.202-.052-.504-.155-.908-.155-.353 0-.756.052-1.16.207-.907.31-1.814 1.033-2.722 1.756l-.05.052c-.807.672-1.664 1.343-2.32 1.498-.302.052-.554.104-.857.104-.706 0-1.361-.207-1.613-.517-.05-.052 0-.155.1-.258l2.017-2.222c1.563-1.602 3.026-3.1 6.454-3.204h.151c2.118 0 4.236.982 4.488 1.085 2.017.982 4.033 1.499 6.1 1.499 2.169 0 4.387-.569 6.706-1.654-.252-.206-.554-.465-.806-.671-2.067.93-3.983 1.343-5.9 1.343-1.915 0-3.831-.465-5.697-1.395-.1-.052-2.42-1.188-4.89-1.188h-.202c-2.874.051-4.488 1.085-5.546 2.015-1.06 0-1.967.31-2.774.516-.706.207-1.361.362-1.966.362h-.756c-.706 0-4.236-.878-7.009-2.015-.302.207-.554.413-.857.62 2.924 1.24 6.504 2.17 7.614 2.274.302 0 .655.051 1.008.051.756 0 1.462-.207 2.218-.413.454-.104.908-.259 1.362-.362l-.404.413-2.016 2.274c-.152.155-.505.62-.303 1.137.1.206.303.413.555.62.504.31 1.361.568 2.168.568.302 0 .605-.051.857-.103.857-.207 1.765-.93 2.672-1.705.757-.62 1.815-1.395 2.622-1.654.252-.051.504-.103.756-.103h.202c.555.052 1.059.258 2.017.982 1.664 1.291 9.075 7.905 9.126 7.957 0 0 .454.414.454 1.137 0 .362-.252.723-.605.982a1.76 1.76 0 0 1-.958.31c-.505 0-.858-.259-.858-.259s-2.57-2.377-3.478-3.203c-.152-.155-.303-.259-.454-.259-.101 0-.152.052-.202.104-.151.206 0 .465.202.62l2.975 3.048s.352.362.403.827c0 .517-.202.93-.706 1.24a1.748 1.748 0 0 1-1.059.362c-.454 0-.756-.207-.857-.258l-.454-.414c-.756-.775-1.563-1.602-2.168-2.067-.151-.103-.302-.258-.454-.258-.05 0-.15 0-.201.103-.05.052-.101.207.05.465.05.104.151.155.151.155l2.168 2.48s.454.569.05 1.034l-.1.103-.202.207c-.353.31-.857.362-1.059.362h-.302c-.202-.052-.353-.103-.454-.207-.1-.155-1.21-1.24-2.117-2.015-.101-.103-.252-.207-.404-.207a.379.379 0 0 0-.201.104c-.152.206.1.516.201.62l1.866 2.067s0 .051-.05.155c-.051.103-.303.31-.959.413h-.252c-.706 0-1.412-.362-1.815-.569.151-.361.252-.774.252-1.188 0-1.55-1.21-2.79-2.722-2.79h-.101c.05-.724-.05-2.067-1.412-2.635-.403-.155-.756-.259-1.16-.259-.302 0-.554.052-.857.155-.302-.568-.756-.981-1.361-1.188a3.414 3.414 0 0 0-1.009-.155c-.554 0-1.058.155-1.512.516a2.298 2.298 0 0 0-3.429-.206c-.554-.465-2.823-1.912-8.924-3.359-.303-.052-.958-.258-1.361-.413-.05.31-.101.672-.152 1.033 0 0 1.11.259 1.362.31 6.201 1.395 8.268 2.894 8.621 3.152-.1.31-.15.62-.15.93 0 1.344 1.058 2.377 2.319 2.377.15 0 .302 0 .453-.051.202.981.807 1.705 1.765 2.066.302.104.555.155.807.155.151 0 .353 0 .554-.051.152.465.555 1.033 1.462 1.395.303.155.606.206.908.206.252 0 .504-.051.706-.154.403 1.033 1.412 1.756 2.521 1.756.756 0 1.462-.31 1.966-.878.454.258 1.362.723 2.32.723h.353c.958-.103 1.361-.516 1.563-.775.05-.051.05-.103.1-.155.202.052.454.104.757.104.504 0 1.008-.155 1.512-.569.505-.361.857-.878.857-1.343.152.051.354.051.505.051.504 0 1.059-.155 1.563-.516a2.32 2.32 0 0 0 1.109-2.015c.151.051.353.051.504.051a2.58 2.58 0 0 0 1.462-.465c.605-.413.958-.982 1.009-1.653.05-.465-.101-.93-.303-1.344 1.614-.723 5.244-2.067 9.58-3.1 0-.362-.05-.672-.151-1.034-5.193 1.137-9.076 2.842-10.034 3.307Zm-14.571 8.63c-1.009 0-1.815-.827-1.866-1.86 0-.104 0-.31-.201-.31-.101 0-.152.051-.252.103-.202.206-.505.413-.908.413a1.32 1.32 0 0 1-.605-.155c-1.059-.465-1.11-1.188-1.059-1.498 0-.104 0-.155-.05-.207l-.05-.052h-.051c-.05 0-.101 0-.202.104-.302.206-.605.31-.907.31-.152 0-.353-.052-.505-.104-1.411-.568-1.31-1.912-1.21-2.325 0-.103 0-.155-.05-.207l-.101-.103-.1.103c-.303.259-.656.414-1.01.414-.806 0-1.461-.672-1.461-1.499 0-.826.655-1.498 1.462-1.498.706 0 1.361.568 1.462 1.291l.05.414.202-.362c0-.052.605-.982 1.714-.93.202 0 .404.052.656.104.857.258 1.008 1.085 1.008 1.395 0 .206.151.206.151.206.05 0 .152-.051.152-.103a1.47 1.47 0 0 1 1.059-.465c.252 0 .504.052.806.207 1.362.62.757 2.377.757 2.428-.101.31-.152.413 0 .517h.1c.05 0 .152 0 .252-.052.202-.052.454-.155.706-.155 1.009 0 1.866.879 1.866 1.912.05 1.137-.807 1.964-1.815 1.964Z" fill="#2D3277"/></svg>

```

## File: views\payment_mercado_pago_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>

<odoo>

    <template id="redirect_form">
        <!-- Mercado Pago generates a unique URL for each payment request. -->
        <form t-att-action="api_url" method="get">
            <t t-foreach="url_params" t-as="param">
                <input type="hidden" t-att-name="param" t-att-value="url_params[param]" />
            </t>
        </form>
    </template>

</odoo>

```

## File: views\payment_provider_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_provider_form" model="ir.ui.view">
        <field name="name">Mercado Pago Provider Form</field>
        <field name="model">payment.provider</field>
        <field name="inherit_id" ref="payment.payment_provider_form"/>
        <field name="arch" type="xml">
            <group name="provider_credentials" position="inside">
                <group invisible="code != 'mercado_pago'">
                    <field name="mercado_pago_access_token"
                           string="Access Token"
                           required="code == 'mercado_pago' and state != 'disabled'"
                           password="True"/>
                </group>
            </group>
        </field>
    </record>

</odoo>

```

