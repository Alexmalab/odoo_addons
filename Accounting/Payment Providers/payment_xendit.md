# Odoo Module: payment_xendit

Category: Accounting/Payment Providers

This file contains the source code of the Odoo module.

## File: const.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# The currencies supported by Xendit, in ISO 4217 format.
SUPPORTED_CURRENCIES = [
    'IDR',
    'PHP',
]

# The codes of the payment methods to activate when Xendit is activated.
DEFAULT_PAYMENT_METHODS_CODES = [
    # Primary payment methods.
    'card',
    'dana',
    'ovo',
    'qris',

    # Brand payment methods.
    'visa',
    'mastercard',
]

# Mapping of payment code to channel code according to Xendit API
PAYMENT_METHODS_MAPPING = {
    'bank_bca': 'BCA',
    'bank_permata': 'PERMATA',
    'bpi': 'DD_BPI',
    'card': 'CREDIT_CARD',
    'maya': 'PAYMAYA',
}

# Mapping of transaction states to Xendit payment statuses.
PAYMENT_STATUS_MAPPING = {
    'draft': (),
    'pending': ('PENDING'),
    'done': ('SUCCEEDED', 'PAID'),
    'cancel': ('CANCELLED', 'EXPIRED'),
    'error': ('FAILED',)
}

```

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

from odoo.addons.payment import setup_provider, reset_payment_provider


def post_init_hook(env):
    setup_provider(env, 'xendit')


def uninstall_hook(env):
    reset_payment_provider(env, 'xendit')

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': "Payment Provider: Xendit",
    'version': '1.0',
    'category': 'Accounting/Payment Providers',
    'sequence': 350,
    'summary': "A payment provider for Indonesian and the Philippines.",
    'description': " ",  # Non-empty string to avoid loading the README file.
    'depends': ['payment'],
    'data': [
        'views/payment_provider_views.xml',
        'views/payment_xendit_templates.xml',

        'data/payment_provider_data.xml',  # Depends on payment_xendit_templates.xml
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

from werkzeug.exceptions import Forbidden

from odoo import http
from odoo.exceptions import ValidationError
from odoo.http import request
from odoo.tools import consteq, str2bool

from odoo.addons.payment import utils as payment_utils


_logger = logging.getLogger(__name__)


class XenditController(http.Controller):

    _webhook_url = '/payment/xendit/webhook'
    _return_url = '/payment/xendit/return'

    @http.route(_webhook_url, type='http', methods=['POST'], auth='public', csrf=False)
    def xendit_webhook(self):
        """ Process the notification data sent by Xendit to the webhook.

        :return: The 'accepted' string to acknowledge the notification.
        """
        data = request.get_json_data()
        _logger.info("Notification received from Xendit with data:\n%s", pprint.pformat(data))

        try:
            # Check the integrity of the notification.
            received_token = request.httprequest.headers.get('x-callback-token')
            tx_sudo = request.env['payment.transaction'].sudo()._get_tx_from_notification_data(
                'xendit', data
            )
            self._verify_notification_token(received_token, tx_sudo)

            # Handle the notification data.
            tx_sudo._handle_notification_data('xendit', data)
        except ValidationError:
            _logger.exception("Unable to handle notification data; skipping to acknowledge.")

        return request.make_json_response(['accepted'], status=200)

    @http.route(_return_url, type='http', methods=['GET'], auth='public')
    def xendit_return(self, tx_ref=None, success=False, access_token=None, **data):
        """Set draft transaction to pending after successfully returning from Xendit."""
        if access_token and str2bool(success, default=False):
            tx_sudo = request.env['payment.transaction'].sudo().search([
                ('provider_code', '=', 'xendit'),
                ('reference', '=', tx_ref),
                ('state', '=', 'draft'),
            ], limit=1)
            if tx_sudo and payment_utils.check_access_token(access_token, tx_ref, tx_sudo.amount):
                tx_sudo._set_pending()
        return request.redirect('/payment/status')

    def _verify_notification_token(self, received_token, tx_sudo):
        """ Check that the received token matches the saved webhook token.

        :param str received_token: The callback token received with the notification data.
        :param payment.transaction tx_sudo: The transaction referenced by the notification data.
        :return: None
        :raise Forbidden: If the tokens don't match.
        """
        # Check for the received token.
        if not received_token:
            _logger.warning("Received notification with missing token.")
            raise Forbidden()

        if not consteq(tx_sudo.provider_id.xendit_webhook_token, received_token):
            _logger.warning("Received notification with invalid callback token %r.", received_token)
            raise Forbidden()

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\neutralize.sql

```sql
UPDATE payment_provider
   SET xendit_secret_key = 'dummysecret',
       xendit_webhook_token = 'dummytoken';

```

## File: data\payment_provider_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="payment.payment_provider_xendit" model="payment.provider">
        <field name="code">xendit</field>
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

from odoo import _, fields, models
from odoo.exceptions import ValidationError

from odoo.addons.payment_xendit import const


_logger = logging.getLogger(__name__)


class PaymentProvider(models.Model):
    _inherit = 'payment.provider'

    code = fields.Selection(
        selection_add=[('xendit', "Xendit")], ondelete={'xendit': 'set default'}
    )
    xendit_secret_key = fields.Char(
        string="Xendit Secret Key", groups='base.group_system', required_if_provider='xendit'
    )
    xendit_webhook_token = fields.Char(
        string="Xendit Webhook Token", groups='base.group_system', required_if_provider='xendit'
    )

    # === BUSINESS METHODS ===#

    def _get_supported_currencies(self):
        """ Override of `payment` to return the supported currencies. """
        supported_currencies = super()._get_supported_currencies()
        if self.code == 'xendit':
            supported_currencies = supported_currencies.filtered(
                lambda c: c.name in const.SUPPORTED_CURRENCIES
            )
        return supported_currencies

    def _get_default_payment_method_codes(self):
        """ Override of `payment` to return the default payment method codes. """
        default_codes = super()._get_default_payment_method_codes()
        if self.code != 'xendit':
            return default_codes
        return const.DEFAULT_PAYMENT_METHODS_CODES

    def _xendit_make_request(self, payload=None):
        """ Make a request to Xendit API and return the JSON-formatted content of the response.

        Note: self.ensure_one()

        :param dict payload: The payload of the request.
        :return The JSON-formatted content of the response.
        :rtype: dict
        :raise ValidationError: If an HTTP error occurs.
        """
        self.ensure_one()

        auth = (self.xendit_secret_key, '')
        url = "https://api.xendit.co/v2/invoices"
        try:
            response = requests.post(url, json=payload, auth=auth, timeout=10)
            response.raise_for_status()
        except (requests.exceptions.ConnectionError, requests.exceptions.Timeout):
            _logger.exception("Unable to reach endpoint at %s", url)
            raise ValidationError("Xendit: " + _("Could not establish the connection to the API."))
        except requests.exceptions.HTTPError as err:
            error_message = err.response.json().get('message')
            _logger.exception(
                "Invalid API request at %s with data:\n%s", url, pprint.pformat(payload)
            )
            raise ValidationError(
                "Xendit: " + _(
                    "The communication with the API failed. Xendit gave us the following"
                    " information: '%s'", error_message
                )
            )
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

from odoo.addons.payment import utils as payment_utils
from odoo.addons.payment_xendit import const
from odoo.addons.payment_xendit.controllers.main import XenditController


_logger = logging.getLogger(__name__)


class PaymentTransaction(models.Model):
    _inherit = 'payment.transaction'

    def _get_specific_rendering_values(self, processing_values):
        """ Override of `payment` to return Xendit-specific rendering values.

        Note: self.ensure_one() from `_get_processing_values`

        :param dict processing_values: The generic and specific processing values of the transaction
        :return: The dict of provider-specific processing values.
        :rtype: dict
        """
        res = super()._get_specific_rendering_values(processing_values)
        if self.provider_code != 'xendit':
            return res

        # Initiate the payment and retrieve the invoice data.
        payload = self._xendit_prepare_invoice_request_payload()
        _logger.info("Sending invoice request for link creation:\n%s", pprint.pformat(payload))
        invoice_data = self.provider_id._xendit_make_request(payload)
        _logger.info("Received invoice request response:\n%s", pprint.pformat(invoice_data))

        # Extract the payment link URL and embed it in the redirect form.
        rendering_values = {
            'api_url': invoice_data.get('invoice_url')
        }
        return rendering_values

    def _xendit_prepare_invoice_request_payload(self):
        """ Create the payload for the invoice request based on the transaction values.

        :return: The request payload.
        :rtype: dict
        """
        base_url = self.provider_id.get_base_url()
        redirect_url = urls.url_join(base_url, XenditController._return_url)
        access_token = payment_utils.generate_access_token(self.reference, self.amount)
        success_url_params = urls.url_encode({
            'tx_ref': self.reference,
            'access_token': access_token,
            'success': 'true',
        })
        payload = {
            'external_id': self.reference,
            'amount': self.amount,
            'description': self.reference,
            'customer': {
                'given_names': self.partner_name,
            },
            'success_redirect_url': f'{redirect_url}?{success_url_params}',
            'failure_redirect_url': redirect_url,
            'payment_methods': [const.PAYMENT_METHODS_MAPPING.get(
                self.payment_method_code, self.payment_method_code.upper())
            ],
            'currency': self.currency_id.name,
        }
        # Extra payload values that must not be included if empty.
        if self.partner_email:
            payload['customer']['email'] = self.partner_email
        if phone := self.partner_id.mobile or self.partner_id.phone:
            payload['customer']['mobile_number'] = phone
        address_details = {}
        if self.partner_city:
            address_details['city'] = self.partner_city
        if self.partner_country_id.name:
            address_details['country'] = self.partner_country_id.name
        if self.partner_zip:
            address_details['postal_code'] = self.partner_zip
        if self.partner_state_id.name:
            address_details['state'] = self.partner_state_id.name
        if self.partner_address:
            address_details['street_line1'] = self.partner_address
        if address_details:
            payload['customer']['addresses'] = [address_details]

        return payload

    def _get_tx_from_notification_data(self, provider_code, notification_data):
        """ Override of `payment` to find the transaction based on the notification data.

        :param str provider_code: The code of the provider that handled the transaction.
        :param dict notification_data: The notification data sent by the provider.
        :return: The transaction if found.
        :rtype: payment.transaction
        :raise ValidationError: If inconsistent data were received.
        :raise ValidationError: If the data match no transaction.
        """
        tx = super()._get_tx_from_notification_data(provider_code, notification_data)
        if provider_code != 'xendit' or len(tx) == 1:
            return tx

        reference = notification_data.get('external_id')
        if not reference:
            raise ValidationError("Xendit: " + _("Received data with missing reference."))

        tx = self.search([('reference', '=', reference), ('provider_code', '=', 'xendit')])
        if not tx:
            raise ValidationError(
                "Xendit: " + _("No transaction found matching reference %s.", reference)
            )
        return tx

    def _process_notification_data(self, notification_data):
        """ Override of `payment` to process the transaction based on Xendit data.

        Note: self.ensure_one()

        :param dict notification_data: The notification data sent by the provider.
        :return: None
        :raise ValidationError: If inconsistent data were received.
        """
        self.ensure_one()

        super()._process_notification_data(notification_data)
        if self.provider_code != 'xendit':
            return

        # Update the provider reference.
        self.provider_reference = notification_data.get('id')

        # Update payment method.
        payment_method_code = notification_data.get('payment_method', '')
        payment_method = self.env['payment.method']._get_from_code(
            payment_method_code, mapping=const.PAYMENT_METHODS_MAPPING
        )
        self.payment_method_id = payment_method or self.payment_method_id

        # Update the payment state.
        payment_status = notification_data.get('status')
        if payment_status in const.PAYMENT_STATUS_MAPPING['pending']:
            self._set_pending()
        elif payment_status in const.PAYMENT_STATUS_MAPPING['done']:
            self._set_done()
        elif payment_status in const.PAYMENT_STATUS_MAPPING['cancel']:
            self._set_canceled()
        elif payment_status in const.PAYMENT_STATUS_MAPPING['error']:
            self._set_error(_(
                "An error occurred during the processing of your payment (status %s). Please try "
                "again."
            ))

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import payment_provider
from . import payment_transaction

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M26.936 5H15.642L4 25.012 15.642 45l2.296-3.929-9.346-16.06 9.346-16.06h6.68L26.936 5Z" fill="#446CB3"/><path d="m18.982 18.579 2.296-3.929-3.34-5.698-2.296 3.952 3.34 5.675Zm13.08-9.627 2.296 3.952-16.42 28.168-2.296-3.952 16.42-28.168Z" fill="#D24D57"/><path d="m23.087 45 2.273-3.952h6.702l9.346-16.037-9.346-16.06L34.358 5 46 25.012 34.358 45H23.087Z" fill="#446CB3"/><path d="m30.972 31.352-2.296 3.93 3.386 5.766 2.296-3.883-3.386-5.813Z" fill="#D24D57"/></svg>

```

## File: views\payment_provider_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_provider_form_xendit" model="ir.ui.view">
        <field name="name">Xendit Provider Form</field>
        <field name="model">payment.provider</field>
        <field name="inherit_id" ref="payment.payment_provider_form"/>
        <field name="arch" type="xml">
            <group name="provider_credentials" position="inside">
                <group invisible="code != 'xendit'">
                    <field name="xendit_secret_key"
                           string="Secret Key"
                           required="code == 'xendit' and state != 'disabled'"
                           password="True"
                    />
                    <field name="xendit_webhook_token"
                           string="Webhook Token"
                           required="code == 'xendit' and state != 'disabled'"
                           password="True"
                    />
                </group>
            </group>
        </field>
    </record>

</odoo>

```

## File: views\payment_xendit_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="redirect_form">
        <form t-att-action="api_url" method="get"/>
    </template>

</odoo>

```

