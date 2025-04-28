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
# See https://docs.mollie.com/payments/multicurrency
SUPPORTED_CURRENCIES = [
    'AED', 'AUD', 'BGN', 'BRL', 'CAD', 'CHF',
    'CZK', 'DKK', 'EUR', 'GBP', 'HKD', 'HRK',
    'HUF', 'ILS', 'ISK', 'JPY', 'MXN', 'MYR',
    'NOK', 'NZD', 'PHP', 'PLN', 'RON', 'RUB',
    'SEK', 'SGD', 'THB', 'TWD', 'USD', 'ZAR'
]

```

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

from odoo.addons.payment import setup_provider, reset_payment_provider


def post_init_hook(cr, registry):
    setup_provider(cr, registry, 'mollie')


def uninstall_hook(cr, registry):
    reset_payment_provider(cr, registry, 'mollie')

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
    'author': 'Odoo S.A, Applix BV, Droggol Infotech Pvt. Ltd.',
    'website': 'https://www.mollie.com',
    'depends': ['payment'],
    'data': [
        'views/payment_mollie_templates.xml',
        'views/payment_provider_views.xml',

        'data/payment_provider_data.xml',
    ],
    'application': False,
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

import requests
from werkzeug import urls

from odoo import _, api, fields, models, service
from odoo.exceptions import ValidationError

from odoo.addons.payment_mollie.const import SUPPORTED_CURRENCIES

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

    @api.model
    def _get_compatible_providers(self, *args, currency_id=None, **kwargs):
        """ Override of payment to unlist Mollie providers for unsupported currencies. """
        providers = super()._get_compatible_providers(*args, currency_id=currency_id, **kwargs)

        currency = self.env['res.currency'].browse(currency_id).exists()
        if currency and currency.name not in SUPPORTED_CURRENCIES:
            providers = providers.filtered(lambda p: p.code != 'mollie')

        return providers

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
            response.raise_for_status()
        except requests.exceptions.RequestException:
            _logger.exception("unable to communicate with Mollie: %s", url)
            raise ValidationError("Mollie: " + _("Could not establish the connection to the API."))
        return response.json()

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
from odoo.addons.payment_mollie.const import SUPPORTED_LOCALES
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
            'locale': user_lang if user_lang in SUPPORTED_LOCALES else 'en_US',

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
<svg width="70" height="70" viewBox="0 0 70 70" fill="none" xmlns="http://www.w3.org/2000/svg">
<mask id="mask0_10_298" style="mask-type:alpha" maskUnits="userSpaceOnUse" x="0" y="0" width="70" height="70">
<path d="M4 0H65C69 0 70 1 70 5V65C70 69 69 70 65 70H4C1 70 0 69 0 65V5C0 1 1 0 4 0Z" fill="white"/>
</mask>
<g mask="url(#mask0_10_298)">
<path fill-rule="evenodd" clip-rule="evenodd" d="M0 0H70V70H0V0Z" fill="url(#paint0_linear_10_298)"/>
<path fill-rule="evenodd" clip-rule="evenodd" d="M4 1H65C67.667 1 69.333 1.667 70 3V0H0V3C0.667 1.667 2 1 4 1Z" fill="white" fill-opacity="0.383"/>
<path fill-rule="evenodd" clip-rule="evenodd" d="M4 69H65C67.667 69 69.333 68 70 66V70H0V66C0.667 68 2 69 4 69Z" fill="black" fill-opacity="0.383"/>
</g>
<path opacity="0.324" fill-rule="evenodd" clip-rule="evenodd" d="M4 69C2 69 0 68 0 65V33.916C0 33.916 11 24 12.5 23C14 22 15 22 16 22C17 22 21 25 21 25L26 22.5L37.25 22.703L48 23L51.576 24.968L51.34 46.978L39.224 69H4Z" fill="#393939"/>
<path d="M8.57797 31.1558H8.58035C8.58035 27.3204 11.4955 24.1137 15.1952 24.0029C17.2714 23.9409 19.2616 24.8628 20.6017 26.5074C21.848 24.9821 23.7079 24.0066 25.7887 24.0066C29.5394 24.0066 32.578 27.1495 32.578 31.0365V40H29.4183V31.1558C29.4183 29.0881 27.7863 27.3463 25.791 27.366C23.8362 27.394 22.261 29.0351 22.2444 31.0611V39.9988H18.997V31.1373C18.997 29.0758 17.3685 27.3364 15.378 27.3537C13.4245 27.3769 11.8462 29.012 11.8231 31.0365V40H8.57797V31.1558Z" fill="#465C68"/>
<path d="M8.57797 29.1558H8.58035C8.58035 25.3204 11.4955 22.1137 15.1952 22.0029C17.2714 21.9409 19.2616 22.8628 20.6017 24.5074C21.848 22.9821 23.7079 22.0066 25.7887 22.0066C29.5394 22.0066 32.578 25.1495 32.578 29.0365V38H29.4183V29.1558C29.4183 27.0881 27.7863 25.3463 25.791 25.366C23.8362 25.394 22.261 27.0351 22.2444 29.0611V37.9988H18.997V29.1373C18.997 27.0758 17.3685 25.3364 15.378 25.3537C13.4245 25.3769 11.8462 27.012 11.8231 29.0365V38H8.57797V29.1558Z" fill="white"/>
<path d="M35 31V37.5H49.036V47.212C49.0363 47.3024 49.0008 47.3892 48.9372 47.4534C48.8737 47.5177 48.7873 47.5542 48.697 47.555H22.303C22.116 47.555 21.964 47.4 21.964 47.212V43H19.2622L19.25 47.555C19.25 49.069 20.464 50.297 21.964 50.297H49.036C50.537 50.297 51.75 49.069 51.75 47.555V27.445C51.75 25.931 50.536 24.703 49.036 24.703H33.1081C33.7772 25.4932 34.2966 26.423 34.6205 27.445H48.697C48.7873 27.4458 48.8737 27.4823 48.9372 27.5466C49.0008 27.6108 49.0363 27.6976 49.036 27.788V31H35Z" fill="#465C68"/>
<path d="M14.423 55.642H15.459V58.474H16.219V55.642H17.256V55H14.423V55.642Z" fill="#465C68"/>
<path d="M17.422 55H18.492L19.301 57.389H19.311L20.075 55H21.145V58.474H20.434V56.012H20.424L19.577 58.474H18.991L18.143 56.036H18.133V58.474H17.422V55Z" fill="#465C68"/>
<path d="M35 29V35.5H49.036V45.212C49.0363 45.3024 49.0008 45.3892 48.9372 45.4534C48.8737 45.5177 48.7873 45.5542 48.697 45.555H22.303C22.116 45.555 21.964 45.4 21.964 45.212V41H19.2622L19.25 45.555C19.25 47.069 20.464 48.297 21.964 48.297H49.036C50.537 48.297 51.75 47.069 51.75 45.555V25.445C51.75 23.931 50.536 22.703 49.036 22.703H33.1081C33.7772 23.4932 34.2966 24.423 34.6205 25.445H48.697C48.7873 25.4458 48.8737 25.4823 48.9372 25.5466C49.0008 25.6108 49.0363 25.6976 49.036 25.788V29H35Z" fill="white"/>
<path d="M14.423 53.642H15.459V56.474H16.219V53.642H17.256V53H14.423V53.642Z" fill="white"/>
<path d="M17.422 53H18.492L19.301 55.389H19.311L20.075 53H21.145V56.474H20.434V54.012H20.424L19.577 56.474H18.991L18.143 54.036H18.133V56.474H17.422V53Z" fill="white"/>
<defs>
<linearGradient id="paint0_linear_10_298" x1="70" y1="0" x2="0" y2="70" gradientUnits="userSpaceOnUse">
<stop stop-color="#797C79"/>
<stop offset="1" stop-color="#545554"/>
</linearGradient>
</defs>
</svg>

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
                <group attrs="{'invisible': [('code', '!=', 'mollie')]}">
                    <field name="mollie_api_key" string="API Key" attrs="{'required': [('code', '=', 'mollie'), ('state', '!=', 'disabled')]}" password="True"/>
                </group>
            </group>
        </field>
    </record>

</odoo>

```

