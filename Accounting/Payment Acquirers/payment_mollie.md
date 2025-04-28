# Odoo Module: payment_mollie

Category: Accounting/Payment Acquirers

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
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

from odoo.addons.payment import reset_payment_acquirer


def uninstall_hook(cr, registry):
    reset_payment_acquirer(cr, registry, 'mollie')

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Mollie Payment Acquirer',
    'version': '1.0',
    'category': 'Accounting/Payment Acquirers',
    'sequence': 356,
    'summary': 'Payment Acquirer: Mollie Implementation',
    'description': " ",  # Non-empty string to avoid loading the README file.

    'author': 'Odoo S.A, Applix BV, Droggol Infotech Pvt. Ltd.',
    'website': 'https://www.mollie.com',

    'depends': ['payment'],
    'data': [
        'views/payment_mollie_templates.xml',
        'views/payment_views.xml',
        'data/payment_acquirer_data.xml',
    ],
    'application': True,
    'uninstall_hook': 'uninstall_hook',
    'license': 'LGPL-3'
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import pprint

from odoo import http
from odoo.exceptions import ValidationError
from odoo.http import request

_logger = logging.getLogger(__name__)


class MollieController(http.Controller):
    _return_url = "/payment/mollie/return"
    _notify_url = "/payment/mollie/notify"

    @http.route(
        _return_url, type='http', auth='public', methods=['GET', 'POST'], csrf=False,
        save_session=False
    )
    def mollie_return(self, **data):
        """ Process the data returned by Mollie after redirection.

        The route is flagged with `save_session=False` to prevent Odoo from assigning a new session
        to the user if they are redirected to this route with a POST request. Indeed, as the session
        cookie is created without a `SameSite` attribute, some browsers that don't implement the
        recommended default `SameSite=Lax` behavior will not include the cookie in the redirection
        request from the payment provider to Odoo. As the redirection to the '/payment/status' page
        will satisfy any specification of the `SameSite` attribute, the session of the user will be
        retrieved and with it the transaction which will be immediately post-processed.

        :param dict data: The feedback data (only `id`) and the transaction reference (`ref`)
                          embedded in the return URL
        """
        _logger.info("Received Mollie return data:\n%s", pprint.pformat(data))
        request.env['payment.transaction'].sudo()._handle_feedback_data('mollie', data)
        return request.redirect('/payment/status')

    @http.route(_notify_url, type='http', auth='public', methods=['POST'], csrf=False)
    def mollie_notify(self, **data):
        """ Process the data sent by Mollie to the webhook.

        :param dict data: The feedback data (only `id`) and the transaction reference (`ref`)
                          embedded in the return URL
        :return: An empty string to acknowledge the notification
        :rtype: str
        """
        _logger.info("Received Mollie notify data:\n%s", pprint.pformat(data))
        try:
            request.env['payment.transaction'].sudo()._handle_feedback_data('mollie', data)
        except ValidationError:  # Acknowledge the notification to avoid getting spammed
            _logger.exception("unable to handle the notification data; skipping to acknowledge")
        return ''  # Acknowledge the notification with an HTTP 200 response

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\payment_acquirer_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="payment.payment_acquirer_mollie" model="payment.acquirer">
        <field name="provider">mollie</field>
        <field name="redirect_form_view_id" ref="redirect_form"/>
        <field name="support_authorization">False</field>
        <field name="support_fees_computation">False</field>
        <field name="support_refund"></field>
        <field name="support_tokenization">False</field>
    </record>

    <record id="payment_method_mollie" model="account.payment.method">
        <field name="name">Mollie</field>
        <field name="code">mollie</field>
        <field name="payment_type">inbound</field>
    </record>

</odoo>

```

## File: models\account_payment_method.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class AccountPaymentMethod(models.Model):
    _inherit = 'account.payment.method'

    @api.model
    def _get_payment_method_information(self):
        res = super()._get_payment_method_information()
        res['mollie'] = {'mode': 'electronic', 'domain': [('type', '=', 'bank')]}
        return res

```

## File: models\payment_acquirer.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

import requests
from werkzeug import urls

from odoo import _, api, fields, models, service
from odoo.exceptions import ValidationError

from odoo.addons.payment_mollie.const import SUPPORTED_CURRENCIES

_logger = logging.getLogger(__name__)


class PaymentAcquirer(models.Model):
    _inherit = 'payment.acquirer'

    provider = fields.Selection(
        selection_add=[('mollie', 'Mollie')], ondelete={'mollie': 'set default'}
    )
    mollie_api_key = fields.Char(
        string="Mollie API Key",
        help="The Test or Live API Key depending on the configuration of the acquirer",
        required_if_provider="mollie", groups="base.group_system"
    )

    #=== BUSINESS METHODS ===#

    @api.model
    def _get_compatible_acquirers(self, *args, currency_id=None, **kwargs):
        """ Override of payment to unlist Mollie acquirers for unsupported currencies. """
        acquirers = super()._get_compatible_acquirers(*args, currency_id=currency_id, **kwargs)

        currency = self.env['res.currency'].browse(currency_id).exists()
        if currency and currency.name not in SUPPORTED_CURRENCIES:
            acquirers = acquirers.filtered(lambda a: a.provider != 'mollie')

        return acquirers

    def _get_default_payment_method_id(self):
        self.ensure_one()
        if self.provider != 'mollie':
            return super()._get_default_payment_method_id()
        return self.env.ref('payment_mollie.payment_method_mollie').id

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
            _logger.exception("Unable to communicate with Mollie: %s", url)
            raise ValidationError("Mollie: " + _("Could not establish the connection to the API."))
        return response.json()

```

## File: models\payment_transaction.py

```python
# -*- coding: utf-8 -*-
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
        :return: The dict of acquirer-specific rendering values
        :rtype: dict
        """
        res = super()._get_specific_rendering_values(processing_values)
        if self.provider != 'mollie':
            return res

        payload = self._mollie_prepare_payment_request_payload()
        _logger.info("sending '/payments' request for link creation:\n%s", pprint.pformat(payload))
        payment_data = self.acquirer_id._mollie_make_request('/payments', data=payload)

        # The acquirer reference is set now to allow fetching the payment status after redirection
        self.acquirer_reference = payment_data.get('id')

        # Extract the checkout URL from the payment data and add it with its query parameters to the
        # rendering values. Passing the query parameters separately is necessary to prevent them
        # from being stripped off when redirecting the user to the checkout URL, which can happen
        # when only one payment method is enabled on Mollie and query parameters are provided.
        checkout_url = payment_data["_links"]["checkout"]["href"]
        parsed_url = urls.url_parse(checkout_url)
        url_params = urls.url_decode(parsed_url.query)
        return {'api_url': checkout_url, 'url_params': url_params}

    def _mollie_prepare_payment_request_payload(self):
        """ Create the payload for the payment request based on the transaction values.

        :return: The request payload
        :rtype: dict
        """
        user_lang = self.env.context.get('lang')
        base_url = self.acquirer_id.get_base_url()
        redirect_url = urls.url_join(base_url, MollieController._return_url)
        webhook_url = urls.url_join(base_url, MollieController._notify_url)
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

    def _get_tx_from_feedback_data(self, provider, data):
        """ Override of payment to find the transaction based on Mollie data.

        :param str provider: The provider of the acquirer that handled the transaction
        :param dict data: The feedback data sent by the provider
        :return: The transaction if found
        :rtype: recordset of `payment.transaction`
        :raise: ValidationError if the data match no transaction
        """
        tx = super()._get_tx_from_feedback_data(provider, data)
        if provider != 'mollie':
            return tx

        tx = self.search([('reference', '=', data.get('ref')), ('provider', '=', 'mollie')])
        if not tx:
            raise ValidationError(
                "Mollie: " + _("No transaction found matching reference %s.", data.get('ref'))
            )
        return tx

    def _process_feedback_data(self, data):
        """ Override of payment to process the transaction based on Mollie data.

        Note: self.ensure_one()

        :param dict data: The feedback data sent by the provider
        :return: None
        """
        super()._process_feedback_data(data)
        if self.provider != 'mollie':
            return

        payment_data = self.acquirer_id._mollie_make_request(
            f'/payments/{self.acquirer_reference}', method="GET"
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
            _logger.info("Received data with invalid payment status: %s", payment_status)
            self._set_error(
                "Mollie: " + _("Received data with invalid payment status: %s", payment_status)
            )

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_payment_method
from . import payment_acquirer
from . import payment_transaction

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

## File: views\payment_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="payment_acquirer_form" model="ir.ui.view">
        <field name="name">Mollie Acquirer Form</field>
        <field name="model">payment.acquirer</field>
        <field name="inherit_id" ref="payment.payment_acquirer_form"/>
        <field name="arch" type="xml">
            <xpath expr='//group[@name="acquirer"]' position='inside'>
                <group attrs="{'invisible': [('provider', '!=', 'mollie')]}">
                    <field name="mollie_api_key" string="API Key" attrs="{'required': [('provider', '=', 'mollie'), ('state', '!=', 'disabled')]}" password="True"/>
                </group>
            </xpath>
        </field>
    </record>

</odoo>

```

