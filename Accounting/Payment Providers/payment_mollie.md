# Odoo Module: payment_mollie

Category: Accounting/Payment Providers

This file contains the source code of the Odoo module.

## File: const.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# List of ISO 15897 locale supported by Mollie
# See full details at `locale` parameter at https://docs.mollie.com/reference/v2/payments-api/create-payment
SUPPORTED_LOCALES = [
    'en_US', 'nl_NL', 'nl_BE', 'fr_FR',
    'fr_BE', 'de_DE', 'de_AT', 'de_CH',
    'es_ES', 'ca_ES', 'pt_PT', 'it_IT',
    'nb_NO', 'sv_SE', 'fi_FI', 'da_DK',
    'is_IS', 'hu_HU', 'pl_PL', 'lv_LV',
    'lt_LT'
]

# Currency codes in ISO 4217 format supported by mollie.
# Note: support varies per payment method.
# See https://docs.mollie.com/payments/multicurrency. Last seen online: 22 September 2022.
SUPPORTED_CURRENCIES = [
    'AED',
    'AUD',
    'BGN',
    'BRL',
    'CAD',
    'CHF',
    'CZK',
    'DKK',
    'EUR',
    'GBP',
    'HKD',
    'HRK',
    'HUF',
    'ILS',
    'ISK',
    'JPY',
    'MXN',
    'MYR',
    'NOK',
    'NZD',
    'PHP',
    'PLN',
    'RON',
    'RUB',
    'SEK',
    'SGD',
    'THB',
    'TWD',
    'USD',
    'ZAR'
]

# The codes of the payment methods to activate when Mollie is activated.
DEFAULT_PAYMENT_METHODS_CODES = [
    # Primary payment methods.
    'card',
    'ideal',
    # Brand payment methods.
    'visa',
    'mastercard',
    'amex',
    'discover',
]

# Mapping of payment method codes to Mollie codes.
PAYMENT_METHODS_MAPPING = {
    'apple_pay': 'applepay',
    'card': 'creditcard',
    'bank_transfer': 'banktransfer',
    'kbc_cbc': 'kbc',
    'p24': 'przelewy24',
    'sepa_direct_debit': 'directdebit',
}

```

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

from odoo.addons.payment import setup_provider, reset_payment_provider


def post_init_hook(env):
    setup_provider(env, 'mollie')


def uninstall_hook(env):
    reset_payment_provider(env, 'mollie')

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Payment Provider: Mollie',
    'version': '1.0',
    'category': 'Accounting/Payment Providers',
    'sequence': 350,
    'summary': "A Dutch payment provider covering several European countries.",
    'description': " ",  # Non-empty string to avoid loading the README file.
    'author': 'Odoo S.A., Applix BV, Droggol Infotech Pvt. Ltd.',
    'website': 'https://www.mollie.com',
    'depends': ['payment'],
    'data': [
        'views/payment_mollie_templates.xml',
        'views/payment_provider_views.xml',

        'data/payment_provider_data.xml',
    ],
    'post_init_hook': 'post_init_hook',
    'uninstall_hook': 'uninstall_hook',
    'license': 'LGPL-3'
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


class MollieController(http.Controller):
    _return_url = '/payment/mollie/return'
    _webhook_url = '/payment/mollie/webhook'

    @http.route(
        _return_url, type='http', auth='public', methods=['GET', 'POST'], csrf=False,
        save_session=False
    )
    def mollie_return_from_checkout(self, **data):
        """ Process the notification data sent by Mollie after redirection from checkout.

        The route is flagged with `save_session=False` to prevent Odoo from assigning a new session
        to the user if they are redirected to this route with a POST request. Indeed, as the session
        cookie is created without a `SameSite` attribute, some browsers that don't implement the
        recommended default `SameSite=Lax` behavior will not include the cookie in the redirection
        request from the payment provider to Odoo. As the redirection to the '/payment/status' page
        will satisfy any specification of the `SameSite` attribute, the session of the user will be
        retrieved and with it the transaction which will be immediately post-processed.

        :param dict data: The notification data (only `id`) and the transaction reference (`ref`)
                          embedded in the return URL
        """
        _logger.info("handling redirection from Mollie with data:\n%s", pprint.pformat(data))
        request.env['payment.transaction'].sudo()._handle_notification_data('mollie', data)
        return request.redirect('/payment/status')

    @http.route(_webhook_url, type='http', auth='public', methods=['POST'], csrf=False)
    def mollie_webhook(self, **data):
        """ Process the notification data sent by Mollie to the webhook.

        :param dict data: The notification data (only `id`) and the transaction reference (`ref`)
                          embedded in the return URL
        :return: An empty string to acknowledge the notification
        :rtype: str
        """
        _logger.info("notification received from Mollie with data:\n%s", pprint.pformat(data))
        try:
            request.env['payment.transaction'].sudo()._handle_notification_data('mollie', data)
        except ValidationError:  # Acknowledge the notification to avoid getting spammed
            _logger.exception("unable to handle the notification data; skipping to acknowledge")
        return ''  # Acknowledge the notification

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\neutralize.sql

```sql
-- disable mollie payment provider
UPDATE payment_provider
   SET mollie_api_key = NULL;

```

## File: data\payment_provider_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="payment.payment_provider_mollie" model="payment.provider">
        <field name="code">mollie</field>
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

from odoo import _, fields, models, service
from odoo.exceptions import ValidationError

from odoo.addons.payment_mollie import const

_logger = logging.getLogger(__name__)


class PaymentProvider(models.Model):
    _inherit = 'payment.provider'

    code = fields.Selection(
        selection_add=[('mollie', 'Mollie')], ondelete={'mollie': 'set default'}
    )
    mollie_api_key = fields.Char(
        string="Mollie API Key",
        help="The Test or Live API Key depending on the configuration of the provider",
        required_if_provider="mollie", groups="base.group_system"
    )

    #=== BUSINESS METHODS ===#

    def _get_supported_currencies(self):
        """ Override of `payment` to return the supported currencies. """
        supported_currencies = super()._get_supported_currencies()
        if self.code == 'mollie':
            supported_currencies = supported_currencies.filtered(
                lambda c: c.name in const.SUPPORTED_CURRENCIES
            )
        return supported_currencies

    def _mollie_make_request(self, endpoint, data=None, method='POST'):
        """ Make a request at mollie endpoint.

        Note: self.ensure_one()

        :param str endpoint: The endpoint to be reached by the request
        :param dict data: The payload of the request
        :param str method: The HTTP method of the request
        :return The JSON-formatted content of the response
        :rtype: dict
        :raise: ValidationError if an HTTP error occurs
        """
        self.ensure_one()
        endpoint = f'/v2/{endpoint.strip("/")}'
        url = urls.url_join('https://api.mollie.com/', endpoint)

        odoo_version = service.common.exp_version()['server_version']
        module_version = self.env.ref('base.module_payment_mollie').installed_version
        headers = {
            "Accept": "application/json",
            "Authorization": f'Bearer {self.mollie_api_key}',
            "Content-Type": "application/json",
            # See https://docs.mollie.com/integration-partners/user-agent-strings
            "User-Agent": f'Odoo/{odoo_version} MollieNativeOdoo/{module_version}',
        }

        try:
            response = requests.request(method, url, json=data, headers=headers, timeout=60)
            try:
                response.raise_for_status()
            except requests.exceptions.HTTPError:
                _logger.exception(
                    "Invalid API request at %s with data:\n%s", url, pprint.pformat(data)
                )
                raise ValidationError(
                    "Mollie: " + _(
                        "The communication with the API failed. Mollie gave us the following "
                        "information: %s", response.json().get('detail', '')
                    ))
        except (requests.exceptions.ConnectionError, requests.exceptions.Timeout):
            _logger.exception("Unable to reach endpoint at %s", url)
            raise ValidationError(
                "Mollie: " + _("Could not establish the connection to the API.")
            )
        return response.json()

    def _get_default_payment_method_codes(self):
        """ Override of `payment` to return the default payment method codes. """
        default_codes = super()._get_default_payment_method_codes()
        if self.code != 'mollie':
            return default_codes
        return const.DEFAULT_PAYMENT_METHODS_CODES

```

## File: models\payment_transaction.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import pprint

from werkzeug import urls

from odoo import _, models
from odoo.exceptions import ValidationError

from odoo.addons.payment.const import CURRENCY_MINOR_UNITS
from odoo.addons.payment_mollie import const
from odoo.addons.payment_mollie.controllers.main import MollieController

_logger = logging.getLogger(__name__)


class PaymentTransaction(models.Model):
    _inherit = 'payment.transaction'

    def _get_specific_rendering_values(self, processing_values):
        """ Override of payment to return Mollie-specific rendering values.

        Note: self.ensure_one() from `_get_processing_values`

        :param dict processing_values: The generic and specific processing values of the transaction
        :return: The dict of provider-specific rendering values
        :rtype: dict
        """
        res = super()._get_specific_rendering_values(processing_values)
        if self.provider_code != 'mollie':
            return res

        payload = self._mollie_prepare_payment_request_payload()
        _logger.info("sending '/payments' request for link creation:\n%s", pprint.pformat(payload))
        payment_data = self.provider_id._mollie_make_request('/payments', data=payload)

        # The provider reference is set now to allow fetching the payment status after redirection
        self.provider_reference = payment_data.get('id')

        # Extract the checkout URL from the payment data and add it with its query parameters to the
        # rendering values. Passing the query parameters separately is necessary to prevent them
        # from being stripped off when redirecting the user to the checkout URL, which can happen
        # when only one payment method is enabled on Mollie and query parameters are provided.
        checkout_url = payment_data['_links']['checkout']['href']
        parsed_url = urls.url_parse(checkout_url)
        url_params = urls.url_decode(parsed_url.query)
        return {'api_url': checkout_url, 'url_params': url_params}

    def _mollie_prepare_payment_request_payload(self):
        """ Create the payload for the payment request based on the transaction values.

        :return: The request payload
        :rtype: dict
        """
        user_lang = self.env.context.get('lang')
        base_url = self.provider_id.get_base_url()
        redirect_url = urls.url_join(base_url, MollieController._return_url)
        webhook_url = urls.url_join(base_url, MollieController._webhook_url)
        decimal_places = CURRENCY_MINOR_UNITS.get(
            self.currency_id.name, self.currency_id.decimal_places
        )

        return {
            'description': self.reference,
            'amount': {
                'currency': self.currency_id.name,
                'value': f"{self.amount:.{decimal_places}f}",
            },
            'locale': user_lang if user_lang in const.SUPPORTED_LOCALES else 'en_US',
            'method': [const.PAYMENT_METHODS_MAPPING.get(
                self.payment_method_code, self.payment_method_code
            )],
            # Since Mollie does not provide the transaction reference when returning from
            # redirection, we include it in the redirect URL to be able to match the transaction.
            'redirectUrl': f'{redirect_url}?ref={self.reference}',
            'webhookUrl': f'{webhook_url}?ref={self.reference}',
        }

    def _get_tx_from_notification_data(self, provider_code, notification_data):
        """ Override of payment to find the transaction based on Mollie data.

        :param str provider_code: The code of the provider that handled the transaction
        :param dict notification_data: The notification data sent by the provider
        :return: The transaction if found
        :rtype: recordset of `payment.transaction`
        :raise: ValidationError if the data match no transaction
        """
        tx = super()._get_tx_from_notification_data(provider_code, notification_data)
        if provider_code != 'mollie' or len(tx) == 1:
            return tx

        tx = self.search(
            [('reference', '=', notification_data.get('ref')), ('provider_code', '=', 'mollie')]
        )
        if not tx:
            raise ValidationError("Mollie: " + _(
                "No transaction found matching reference %s.", notification_data.get('ref')
            ))
        return tx

    def _process_notification_data(self, notification_data):
        """ Override of payment to process the transaction based on Mollie data.

        Note: self.ensure_one()

        :param dict notification_data: The notification data sent by the provider
        :return: None
        """
        super()._process_notification_data(notification_data)
        if self.provider_code != 'mollie':
            return

        payment_data = self.provider_id._mollie_make_request(
            f'/payments/{self.provider_reference}', method="GET"
        )

        # Update the payment method.
        payment_method_type = payment_data.get('method', '')
        if payment_method_type == 'creditcard':
            payment_method_type = payment_data.get('details', {}).get('cardLabel', '').lower()
        payment_method = self.env['payment.method']._get_from_code(
            payment_method_type, mapping=const.PAYMENT_METHODS_MAPPING
        )
        self.payment_method_id = payment_method or self.payment_method_id

        # Update the payment state.
        payment_status = payment_data.get('status')
        if payment_status == 'pending':
            self._set_pending()
        elif payment_status == 'authorized':
            self._set_authorized()
        elif payment_status == 'paid':
            self._set_done()
        elif payment_status in ['expired', 'canceled', 'failed']:
            self._set_canceled("Mollie: " + _("Canceled payment with status: %s", payment_status))
        else:
            _logger.info(
                "received data with invalid payment status (%s) for transaction with reference %s",
                payment_status, self.reference
            )
            self._set_error(
                "Mollie: " + _("Received data with invalid payment status: %s", payment_status)
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
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path fill-rule="evenodd" clip-rule="evenodd" d="M37.694 20.15c.857 0 1.551-.705 1.551-1.575S38.551 17 37.694 17s-1.551.705-1.551 1.575.694 1.575 1.551 1.575Zm-16.546 1.748c-2.746 0-4.974 2.269-4.974 5.051 0 2.783 2.233 5.051 4.974 5.051 2.74 0 4.974-2.268 4.974-5.05 0-2.783-2.229-5.052-4.974-5.052Zm0 7.713c-1.443 0-2.616-1.192-2.616-2.657 0-1.464 1.173-2.656 2.616-2.656 1.442 0 2.616 1.192 2.616 2.657 0 1.464-1.174 2.656-2.616 2.656ZM10.61 21.888a4.22 4.22 0 0 1 3.196 1.45 4.523 4.523 0 0 1 1.132 2.997v5.46h-2.379v-5.528c-.005-1.087-.889-1.974-1.964-1.974-.062 0-.13.005-.197.01-.962.1-1.768 1.024-1.768 2.021v5.471H6.25v-5.513c-.005-1.092-.884-1.984-1.96-1.984-.061 0-.129.005-.196.01-.956.1-1.768 1.024-1.768 2.027v5.46H0v-5.528c0-2.415 1.939-4.379 4.317-4.379 1.19 0 2.322.504 3.14 1.38a4.29 4.29 0 0 1 3.153-1.38Zm19.173-4.652h-2.379v14.57h2.379v-14.57Zm2.172 0h2.378v14.57h-2.378v-14.57Zm6.928 4.904h-2.378v9.66h2.378v-9.66Zm9.736 1.176A4.872 4.872 0 0 1 50 26.724v1.107h-7.234a2.678 2.678 0 0 0 2.575 1.985c.947 0 1.8-.488 2.29-1.313l.099-.162 1.965.981-.114.195c-.894 1.527-2.534 2.478-4.281 2.478h-.005a4.896 4.896 0 0 1-3.532-1.507 5.054 5.054 0 0 1-1.442-3.607 5.075 5.075 0 0 1 1.453-3.512 4.895 4.895 0 0 1 3.453-1.481h.063c1.256 0 2.435.51 3.33 1.428Zm-1.023 2.52c-.342-1.029-1.314-1.753-2.4-1.753-1.085 0-2.057.724-2.393 1.753h4.793Z"/><path d="M43.105 4h.972V.818h1.108V0H42v.818h1.105V4Zm2.775 0h.858V1.453h.053L47.663 4h.555l.871-2.547h.056V4H50V0h-1.108l-.924 2.714h-.05L46.99 0h-1.11v4Z" fill="#D1D5DB"/></svg>

```

## File: views\payment_mollie_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="redirect_form">
        <!-- Mollie generates a unique URL for each payment request -->
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
        <field name="name">Mollie Provider Form</field>
        <field name="model">payment.provider</field>
        <field name="inherit_id" ref="payment.payment_provider_form"/>
        <field name="arch" type="xml">
            <group name="provider_credentials" position="inside">
                <group invisible="code != 'mollie'">
                    <field name="mollie_api_key" string="API Key" required="code == 'mollie' and state != 'disabled'" password="True"/>
                </group>
            </group>
        </field>
    </record>

</odoo>

```

