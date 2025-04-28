# Odoo Module: payment_mercado_pago

Category: Accounting/Payment Providers

This file contains the source code of the Odoo module.

## File: const.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _


# Currency codes of the currencies supported by Mercado Pago in ISO 4217 format.
# See https://api.mercadopago.com/currencies. Last seen online: 24 November 2022.
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


def post_init_hook(cr, registry):
    setup_provider(cr, registry, 'mercado_pago')


def uninstall_hook(cr, registry):
    reset_payment_provider(cr, registry, 'mercado_pago')

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
    'application': False,
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

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError

from odoo.addons.payment_mercado_pago.const import SUPPORTED_CURRENCIES


_logger = logging.getLogger(__name__)


class Paymentprovider(models.Model):
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

    @api.model
    def _get_compatible_providers(self, *args, currency_id=None, **kwargs):
        """ Override of `payment` to unlist Mercado Pago providers for unsupported currencies. """
        providers = super()._get_compatible_providers(*args, currency_id=currency_id, **kwargs)

        currency = self.env['res.currency'].browse(currency_id).exists()
        if currency and currency.name not in SUPPORTED_CURRENCIES:
            providers = providers.filtered(lambda p: p.code != 'mercado_pago')

        return providers

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

```

## File: models\payment_transaction.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import pprint
from urllib.parse import quote as url_quote

from werkzeug import urls

from odoo import _, api, models
from odoo.exceptions import UserError, ValidationError

from odoo.addons.payment_mercado_pago.const import ERROR_MESSAGE_MAPPING, TRANSACTION_STATUS_MAPPING
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

        # Extract the payment link URL and embed it in the redirect form.
        rendering_values = {
            'api_url': api_url,
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

        # In the case where we are issuing a preference request in CLP or COP, we must ensure that
        # the price unit is an integer because these currencies do not have a minor unit.
        unit_price = self.amount
        if self.currency_id.name in ('CLP', 'COP'):
            rounded_unit_price = int(self.amount)
            if rounded_unit_price != self.amount:
                raise UserError(_(
                    "Prices in the currency %s must be expressed in integer values.",
                    self.currency_id.name,
                ))
            unit_price = rounded_unit_price

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

        payment_id = notification_data.get('payment_id')
        if not payment_id:
            raise ValidationError("Mercado Pago: " + _("Received data with missing payment id."))
        self.provider_reference = payment_id

        # Verify the notification data.
        verified_payment_data = self.provider_id._mercado_pago_make_request(
            f'/v1/payments/{self.provider_reference}', method='GET'
        )

        payment_status = verified_payment_data.get('status')
        if not payment_status:
            raise ValidationError("Mercado Pago: " + _("Received data with missing status."))

        if payment_status in TRANSACTION_STATUS_MAPPING['pending']:
            self._set_pending()
        elif payment_status in TRANSACTION_STATUS_MAPPING['done']:
            self._set_done()
        elif payment_status in TRANSACTION_STATUS_MAPPING['canceled']:
            self._set_canceled()
        elif payment_status in TRANSACTION_STATUS_MAPPING['error']:
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
        return "Mercado Pago: " + ERROR_MESSAGE_MAPPING.get(
            status_detail, ERROR_MESSAGE_MAPPING['cc_rejected_other_reason']
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
    <mask id="mask0_3_265" style="mask-type:alpha" maskUnits="userSpaceOnUse" x="0" y="0" width="70" height="70">
    <path d="M4 0H65C69 0 70 1 70 5V65C70 69 69 70 65 70H4C1 70 0 69 0 65V5C0 1 1 0 4 0Z" fill="white"/>
    </mask>
    <g mask="url(#mask0_3_265)">
    <path fill-rule="evenodd" clip-rule="evenodd" d="M0 0H70V70H0V0Z" fill="url(#paint0_linear_3_265)"/>
    <path fill-rule="evenodd" clip-rule="evenodd" d="M4 1H65C67.667 1 69.333 1.667 70 3V0H0V3C0.667 1.667 2 1 4 1Z" fill="white" fill-opacity="0.383"/>
    <path fill-rule="evenodd" clip-rule="evenodd" d="M4 69H65C67.667 69 69.333 68 70 66V70H0V66C0.667 68 2 69 4 69Z" fill="black" fill-opacity="0.383"/>
    </g>
    <path opacity="0.324" fill-rule="evenodd" clip-rule="evenodd" d="M4 69C2 69 0 68 0 65V33.916L8.01445 22.7066H13.6388L16.8306 19.9728L27.1533 19.9728L30.3274 22.7132H34.666H48.107L51.576 24.968L51.34 46.978L39.224 69H4Z" fill="#393939"/>
    <path d="M27.0181 21.75H23.26C22.91 21.75 22.5731 21.8789 22.315 22.1109L18.0144 25.9781C18.01 25.9824 18.0056 25.991 18.0013 25.9953C17.275 26.6656 17.2881 27.7355 17.9094 28.4016C18.465 28.9988 19.6331 29.1578 20.3638 28.5176C20.3681 28.5133 20.3769 28.5133 20.3812 28.509L23.8769 25.3637C24.1612 25.1102 24.6075 25.1273 24.8656 25.4066C25.1281 25.6859 25.1062 26.1199 24.8219 26.3777L23.68 27.4047L30.05 32.4836C30.1769 32.5867 30.2906 32.6984 30.3956 32.8145V24.5L28.0069 22.1539C27.7487 21.8961 27.39 21.75 27.0181 21.75ZM31.8 24.5086V34.1293C31.8 34.8898 32.4256 35.5043 33.2 35.5043H36V24.5086H31.8ZM8 35.5H10.8C11.5744 35.5 12.2 34.8855 12.2 34.125V24.5086H8V35.5ZM29.1706 33.5535L22.6388 28.3457L21.3263 29.5273C20.0269 30.6918 18.0362 30.5801 16.8769 29.3383C15.7 28.075 15.7919 26.1199 17.0694 24.9684L20.6481 21.75H16.9819C16.61 21.75 16.2556 21.8961 15.9931 22.1539L13.6 24.5V34.1207H14.4006L18.36 37.6398C19.5587 38.598 21.3219 38.4176 22.2975 37.2402L22.3062 37.2316L23.0894 37.8977C23.785 38.4562 24.8131 38.3488 25.3775 37.6656L26.7512 36.007L26.9875 36.1961C27.5869 36.673 28.4706 36.5871 28.9562 35.9941L29.3719 35.4914C29.8619 34.8984 29.77 34.0348 29.1706 33.5535Z" fill="#41535E"/>
    <path d="M27.0181 19.9497H23.26C22.91 19.9497 22.5731 20.0786 22.315 20.3106L18.0144 24.1778C18.01 24.1821 18.0056 24.1907 18.0013 24.195C17.275 24.8653 17.2881 25.9352 17.9094 26.6012C18.465 27.1985 19.6331 27.3575 20.3638 26.7172C20.3681 26.7129 20.3769 26.7129 20.3812 26.7086L23.8769 23.5633C24.1612 23.3098 24.6075 23.327 24.8656 23.6063C25.1281 23.8856 25.1062 24.3196 24.8219 24.5774L23.68 25.6043L30.05 30.6832C30.1769 30.7864 30.2906 30.8981 30.3956 31.0141V22.6997L28.0069 20.3536C27.7487 20.0957 27.39 19.9497 27.0181 19.9497ZM31.8 22.7082V32.329C31.8 33.0895 32.4256 33.704 33.2 33.704H36V22.7082H31.8ZM8 33.6997H10.8C11.5744 33.6997 12.2 33.0852 12.2 32.3247V22.7082H8V33.6997ZM29.1706 31.7532L22.6388 26.5454L21.3263 27.727C20.0269 28.8915 18.0362 28.7797 16.8769 27.5379C15.7 26.2747 15.7919 24.3196 17.0694 23.168L20.6481 19.9497H16.9819C16.61 19.9497 16.2556 20.0957 15.9931 20.3536L13.6 22.6997V32.3204H14.4006L18.36 35.8395C19.5587 36.7977 21.3219 36.6172 22.2975 35.4399L22.3062 35.4313L23.0894 36.0973C23.785 36.6559 24.8131 36.5485 25.3775 35.8653L26.7512 34.2067L26.9875 34.3957C27.5869 34.8727 28.4706 34.7868 28.9562 34.1938L29.3719 33.6911C29.8619 33.0981 29.77 32.2344 29.1706 31.7532Z" fill="white"/>
    <path d="M19.2612 43.3711L19.25 47.555C19.25 49.069 20.464 50.297 21.964 50.297H49.036C50.537 50.297 51.75 49.069 51.75 47.555V27.445C51.75 25.931 50.536 24.703 49.036 24.703H39.0229V27.445H48.697C48.7874 27.4458 48.8737 27.4823 48.9372 27.5466C49.0008 27.6109 49.0363 27.6976 49.036 27.788V31H39.0229V37.5H49.036V47.212C49.0363 47.3024 49.0008 47.3892 48.9372 47.4534C48.8737 47.5177 48.7874 47.5542 48.697 47.555H22.303C22.116 47.555 21.964 47.4 21.964 47.212V42.4767C20.9467 43.2767 20.0327 43.4848 19.2612 43.3711Z" fill="#41535E"/>
    <path d="M14.423 55.642H15.459V58.474H16.219V55.642H17.256V55H14.423V55.642Z" fill="#41535E"/>
    <path d="M18.492 55H17.422V58.474H18.133V56.036H18.143L18.991 58.474H19.577L20.424 56.012H20.434V58.474H21.145V55H20.075L19.311 57.389H19.301L18.492 55Z" fill="#41535E"/>
    <path d="M19.2612 41.3711L19.25 45.555C19.25 47.069 20.464 48.297 21.964 48.297H49.036C50.537 48.297 51.75 47.069 51.75 45.555V25.445C51.75 23.931 50.536 22.703 49.036 22.703H39.0229V25.445H48.697C48.7874 25.4458 48.8737 25.4823 48.9372 25.5466C49.0008 25.6109 49.0363 25.6976 49.036 25.788V29H39.0229V35.5H49.036V45.212C49.0363 45.3024 49.0008 45.3892 48.9372 45.4534C48.8737 45.5177 48.7874 45.5542 48.697 45.555H22.303C22.116 45.555 21.964 45.4 21.964 45.212V40.4767C20.9467 41.2767 20.0327 41.4848 19.2612 41.3711Z" fill="white"/>
    <path d="M14.423 53.642H15.459V56.474H16.219V53.642H17.256V53H14.423V53.642Z" fill="white"/>
    <path d="M18.492 53H17.422V56.474H18.133V54.036H18.143L18.991 56.474H19.577L20.424 54.012H20.434V56.474H21.145V53H20.075L19.311 55.389H19.301L18.492 53Z" fill="white"/>
    <defs>
    <linearGradient id="paint0_linear_3_265" x1="70" y1="0" x2="0" y2="70" gradientUnits="userSpaceOnUse">
    <stop stop-color="#94B6C8"/>
    <stop offset="1" stop-color="#6A9EBA"/>
    <stop offset="1" stop-color="#6A9EBA"/>
    </linearGradient>
    </defs>
    </svg>
```

## File: views\payment_mercado_pago_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>

<odoo>

    <template id="redirect_form">
        <!-- Mercado Pago generates a unique URL for each payment request. -->
        <form t-att-action="api_url" method="post"/>
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
                <group attrs="{'invisible': [('code', '!=', 'mercado_pago')]}">
                    <field name="mercado_pago_access_token"
                           string="Access Token"
                           attrs="{'required': [('code', '=', 'mercado_pago'), ('state', '!=', 'disabled')]}"
                           password="True"/>
                </group>
            </group>
        </field>
    </record>

</odoo>

```

