# Odoo Module: payment_flutterwave

Category: Accounting/Payment Providers

This file contains the source code of the Odoo module.

## File: const.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# The currencies supported by Flutterwave, in ISO 4217 format.
# See https://flutterwave.com/us/support/general/what-are-the-currencies-accepted-on-flutterwave.
# Last website update: June 2022.
# Last seen online: 24 November 2022.
SUPPORTED_CURRENCIES = [
    'GBP',
    'CAD',
    'CLP',
    'COP',
    'EGP',
    'EUR',
    'GHS',
    'GNF',
    'KES',
    'MWK',
    'MAD',
    'NGN',
    'RWF',
    'SLL',
    'STD',
    'ZAR',
    'TZS',
    'UGX',
    'USD',
    'XAF',
    'XOF',
    'ZMW',
]


# Mapping of transaction states to Flutterwave payment statuses.
PAYMENT_STATUS_MAPPING = {
    'pending': ['pending', 'pending auth'],
    'done': ['successful'],
    'cancel': ['cancelled'],
    'error': ['failed'],
}

```

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

from odoo.addons.payment import setup_provider, reset_payment_provider


def post_init_hook(cr, registry):
    setup_provider(cr, registry, 'flutterwave')


def uninstall_hook(cr, registry):
    reset_payment_provider(cr, registry, 'flutterwave')

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': "Payment Provider: Flutterwave",
    'version': '1.0',
    'category': 'Accounting/Payment Providers',
    'sequence': 350,
    'summary': "A Nigerian payment provider covering several African countries.",
    'description': " ",  # Non-empty string to avoid loading the README file.
    'depends': ['payment'],
    'data': [
        'views/payment_flutterwave_templates.xml',
        'views/payment_provider_views.xml',

        'data/payment_provider_data.xml',
    ],
    'assets': {
        'web.assets_frontend': [
            'payment_flutterwave/static/src/js/payment_form.js',
        ],
    },
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
import json
import logging
import pprint

from werkzeug.exceptions import Forbidden

from odoo import http
from odoo.exceptions import ValidationError
from odoo.http import request


_logger = logging.getLogger(__name__)


class FlutterwaveController(http.Controller):
    _return_url = '/payment/flutterwave/return'
    _auth_return_url = '/payment/flutterwave/auth_return'
    _webhook_url = '/payment/flutterwave/webhook'

    @http.route(_return_url, type='http', methods=['GET'], auth='public')
    def flutterwave_return_from_checkout(self, **data):
        """ Process the notification data sent by Flutterwave after redirection from checkout.

        :param dict data: The notification data.
        """
        _logger.info("Handling redirection from Flutterwave with data:\n%s", pprint.pformat(data))

        # Handle the notification data.
        if data.get('status') != 'cancelled':
            request.env['payment.transaction'].sudo()._handle_notification_data('flutterwave', data)
        else:  # The customer cancelled the payment by clicking on the close button.
            pass  # Don't try to process this case because the transaction id was not provided.

        # Redirect the user to the status page.
        return request.redirect('/payment/status')

    @http.route(_auth_return_url, type='http', methods=['GET'], auth='public')
    def flutterwave_return_from_authorization(self, response):
        """ Process the response sent by Flutterwave after authorization.

        :param str response: The stringified JSON response.
        """
        data = json.loads(response)
        return self.flutterwave_return_from_checkout(**data)

    @http.route(_webhook_url, type='http', methods=['POST'], auth='public', csrf=False)
    def flutterwave_webhook(self):
        """ Process the notification data sent by Flutterwave to the webhook.

        :return: An empty string to acknowledge the notification.
        :rtype: str
        """
        data = request.get_json_data()
        _logger.info("Notification received from Flutterwave with data:\n%s", pprint.pformat(data))

        if data['event'] == 'charge.completed':
            try:
                # Check the origin of the notification.
                tx_sudo = request.env['payment.transaction'].sudo()._get_tx_from_notification_data(
                    'flutterwave', data['data']
                )
                signature = request.httprequest.headers.get('verif-hash')
                self._verify_notification_signature(signature, tx_sudo)

                # Handle the notification data.
                notification_data = data['data']
                tx_sudo._handle_notification_data('flutterwave', notification_data)
            except ValidationError:  # Acknowledge the notification to avoid getting spammed.
                _logger.exception("Unable to handle the notification data; skipping to acknowledge")
        return request.make_json_response('')

    @staticmethod
    def _verify_notification_signature(received_signature, tx_sudo):
        """ Check that the received signature matches the expected one.

        :param dict received_signature: The signature received with the notification data.
        :param recordset tx_sudo: The sudoed transaction referenced by the notification data, as a
                                  `payment.transaction` record.
        :return: None
        :raise Forbidden: If the signatures don't match.
        """
        # Check for the received signature.
        if not received_signature:
            _logger.warning("Received notification with missing signature.")
            raise Forbidden()

        # Compare the received signature with the expected signature.
        expected_signature = tx_sudo.provider_id.flutterwave_webhook_secret
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
-- disable flutterwave payment provider
UPDATE payment_provider
   SET flutterwave_public_key = NULL,
       flutterwave_secret_key = NULL,
       flutterwave_webhook_secret = NULL;

```

## File: data\payment_provider_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="payment.payment_provider_flutterwave" model="payment.provider">
        <field name="code">flutterwave</field>
        <field name="redirect_form_view_id" ref="redirect_form"/>
        <field name="allow_tokenization">True</field>
    </record>

</odoo>

```

## File: models\payment_provider.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import pprint

import requests
from werkzeug.urls import url_join

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError

from odoo.addons.payment_flutterwave.const import SUPPORTED_CURRENCIES


_logger = logging.getLogger(__name__)


class PaymentProvider(models.Model):
    _inherit = 'payment.provider'

    code = fields.Selection(
        selection_add=[('flutterwave', "Flutterwave")], ondelete={'flutterwave': 'set default'}
    )
    flutterwave_public_key = fields.Char(
        string="Flutterwave Public Key",
        help="The key solely used to identify the account with Flutterwave.",
        required_if_provider='flutterwave',
    )
    flutterwave_secret_key = fields.Char(
        string="Flutterwave Secret Key",
        required_if_provider='flutterwave',
        groups='base.group_system',
    )
    flutterwave_webhook_secret = fields.Char(
        string="Flutterwave Webhook Secret",
        required_if_provider='flutterwave',
        groups='base.group_system',
    )

    #=== COMPUTE METHODS ===#

    def _compute_feature_support_fields(self):
        """ Override of `payment` to enable additional features. """
        super()._compute_feature_support_fields()
        self.filtered(lambda p: p.code == 'flutterwave').update({
            'support_tokenization': True,
        })

    # === BUSINESS METHODS ===#

    @api.model
    def _get_compatible_providers(self, *args, currency_id=None, is_validation=False, **kwargs):
        """ Override of payment to filter out Flutterwave providers for unsupported currencies or
        for validation operations. """
        providers = super()._get_compatible_providers(
            *args, currency_id=currency_id, is_validation=is_validation, **kwargs
        )

        currency = self.env['res.currency'].browse(currency_id).exists()
        if (currency and currency.name not in SUPPORTED_CURRENCIES) or is_validation:
            providers = providers.filtered(lambda p: p.code != 'flutterwave')

        return providers

    def _flutterwave_make_request(self, endpoint, payload=None, method='POST'):
        """ Make a request to Flutterwave API at the specified endpoint.

        Note: self.ensure_one()

        :param str endpoint: The endpoint to be reached by the request.
        :param dict payload: The payload of the request.
        :param str method: The HTTP method of the request.
        :return The JSON-formatted content of the response.
        :rtype: dict
        :raise ValidationError: If an HTTP error occurs.
        """
        self.ensure_one()

        url = url_join('https://api.flutterwave.com/v3/', endpoint)
        headers = {'Authorization': f'Bearer {self.flutterwave_secret_key}'}
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
                raise ValidationError("Flutterwave: " + _(
                    "The communication with the API failed. Flutterwave gave us the following "
                    "information: '%s'", response.json().get('message', '')
                ))
        except (requests.exceptions.ConnectionError, requests.exceptions.Timeout):
            _logger.exception("Unable to reach endpoint at %s", url)
            raise ValidationError(
                "Flutterwave: " + _("Could not establish the connection to the API.")
            )
        return response.json()

```

## File: models\payment_token.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class PaymentToken(models.Model):
    _inherit = 'payment.token'

    flutterwave_customer_email = fields.Char(
        help="The email of the customer at the time the token was created.", readonly=True
    )

```

## File: models\payment_transaction.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import pprint

from werkzeug import urls

from odoo import _, models
from odoo.exceptions import UserError, ValidationError

from odoo.addons.payment import utils as payment_utils
from odoo.addons.payment_flutterwave.const import PAYMENT_STATUS_MAPPING
from odoo.addons.payment_flutterwave.controllers.main import FlutterwaveController


_logger = logging.getLogger(__name__)


class PaymentTransaction(models.Model):
    _inherit = 'payment.transaction'

    def _get_specific_processing_values(self, processing_values):
        """ Override of payment to redirect pending token-flow transactions.

        If the financial institution insists on 3-D Secure authentication, this
        override will redirect the user to the provided authorization page.

        Note: `self.ensure_one()`
        """
        res = super()._get_specific_processing_values(processing_values)
        if self._flutterwave_is_authorization_pending():
            res['auth_redirect_form_html'] = self.env['ir.qweb']._render(
                self.provider_id.redirect_form_view_id.id,
                {'api_url': self.provider_reference},
            )
        return res

    def _get_specific_rendering_values(self, processing_values):
        """ Override of payment to return Flutterwave-specific rendering values.

        Note: self.ensure_one() from `_get_processing_values`

        :param dict processing_values: The generic and specific processing values of the transaction
        :return: The dict of provider-specific processing values.
        :rtype: dict
        """
        res = super()._get_specific_rendering_values(processing_values)
        if self.provider_code != 'flutterwave':
            return res

        # Initiate the payment and retrieve the payment link data.
        base_url = self.provider_id.get_base_url()
        payload = {
            'tx_ref': self.reference,
            'amount': self.amount,
            'currency': self.currency_id.name,
            'redirect_url': urls.url_join(base_url, FlutterwaveController._return_url),
            'customer': {
                'email': self.partner_email,
                'name': self.partner_name,
                'phonenumber': self.partner_phone,
            },
            'customizations': {
                'title': self.company_id.name,
                'logo': urls.url_join(base_url, f'web/image/res.company/{self.company_id.id}/logo'),
            },
        }
        payment_link_data = self.provider_id._flutterwave_make_request('payments', payload=payload)

        # Extract the payment link URL and embed it in the redirect form.
        rendering_values = {
            'api_url': payment_link_data['data']['link'],
        }
        return rendering_values

    def _send_payment_request(self):
        """ Override of payment to send a payment request to Flutterwave.

        Note: self.ensure_one()

        :return: None
        :raise UserError: If the transaction is not linked to a token.
        """
        super()._send_payment_request()
        if self.provider_code != 'flutterwave':
            return

        # Prepare the payment request to Flutterwave.
        if not self.token_id:
            raise UserError("Flutterwave: " + _("The transaction is not linked to a token."))

        first_name, last_name = payment_utils.split_partner_name(self.partner_name)
        base_url = self.provider_id.get_base_url()
        data = {
            'token': self.token_id.provider_ref,
            'email': self.token_id.flutterwave_customer_email,
            'amount': self.amount,
            'currency': self.currency_id.name,
            'country': self.company_id.country_id.code,
            'tx_ref': self.reference,
            'first_name': first_name,
            'last_name': last_name,
            'ip': payment_utils.get_customer_ip_address(),
            'redirect_url': urls.url_join(base_url, FlutterwaveController._auth_return_url),
        }

        # Make the payment request to Flutterwave.
        response_content = self.provider_id._flutterwave_make_request(
            'tokenized-charges', payload=data
        )

        # Handle the payment request response.
        _logger.info(
            "payment request response for transaction with reference %s:\n%s",
            self.reference, pprint.pformat(response_content)
        )
        self._handle_notification_data('flutterwave', response_content['data'])

    def _get_tx_from_notification_data(self, provider_code, notification_data):
        """ Override of payment to find the transaction based on Flutterwave data.

        :param str provider_code: The code of the provider that handled the transaction.
        :param dict notification_data: The notification data sent by the provider.
        :return: The transaction if found.
        :rtype: recordset of `payment.transaction`
        :raise ValidationError: If inconsistent data were received.
        :raise ValidationError: If the data match no transaction.
        """
        tx = super()._get_tx_from_notification_data(provider_code, notification_data)
        if provider_code != 'flutterwave' or len(tx) == 1:
            return tx

        reference = notification_data.get('tx_ref') or notification_data.get('txRef')
        if not reference:
            raise ValidationError("Flutterwave: " + _("Received data with missing reference."))

        tx = self.search([('reference', '=', reference), ('provider_code', '=', 'flutterwave')])
        if not tx:
            raise ValidationError(
                "Flutterwave: " + _("No transaction found matching reference %s.", reference)
            )
        return tx

    def _process_notification_data(self, notification_data):
        """ Override of payment to process the transaction based on Flutterwave data.

        Note: self.ensure_one()

        :param dict notification_data: The notification data sent by the provider.
        :return: None
        :raise ValidationError: If inconsistent data were received.
        """
        super()._process_notification_data(notification_data)
        if self.provider_code != 'flutterwave':
            return

        # Verify the notification data.
        verification_response_content = self.provider_id._flutterwave_make_request(
            'transactions/verify_by_reference', payload={'tx_ref': self.reference}, method='GET'
        )
        verified_data = verification_response_content['data']

        # Process the verified notification data.
        self.provider_reference = verified_data['id']
        payment_status = verified_data['status'].lower()
        if payment_status in PAYMENT_STATUS_MAPPING['pending']:
            auth_url = notification_data.get('meta', {}).get('authorization', {}).get('redirect')
            if auth_url:
                # will be set back to the actual value after moving away from pending
                self.provider_reference = auth_url
            self._set_pending()
        elif payment_status in PAYMENT_STATUS_MAPPING['done']:
            self._set_done()
            has_token_data = 'token' in verified_data.get('card', {})
            if self.tokenize and has_token_data:
                self._flutterwave_tokenize_from_notification_data(verified_data)
        elif payment_status in PAYMENT_STATUS_MAPPING['cancel']:
            self._set_canceled()
        elif payment_status in PAYMENT_STATUS_MAPPING['error']:
            self._set_error(_(
                "An error occurred during the processing of your payment (status %s). Please try "
                "again.", payment_status
            ))
        else:
            _logger.warning(
                "Received data with invalid payment status (%s) for transaction with reference %s.",
                payment_status, self.reference
            )
            self._set_error("Flutterwave: " + _("Unknown payment status: %s", payment_status))

    def _flutterwave_tokenize_from_notification_data(self, notification_data):
        """ Create a new token based on the notification data.

        Note: self.ensure_one()

        :param dict notification_data: The notification data sent by the provider.
        :return: None
        """
        self.ensure_one()

        token = self.env['payment.token'].create({
            'provider_id': self.provider_id.id,
            'payment_details': notification_data['card']['last_4digits'],
            'partner_id': self.partner_id.id,
            'provider_ref': notification_data['card']['token'],
            'flutterwave_customer_email': notification_data['customer']['email'],
            'verified': True,  # The payment is confirmed, so the payment method is valid.
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

    def _flutterwave_is_authorization_pending(self):
        return self.filtered_domain([
            ('provider_code', '=', 'flutterwave'),
            ('operation', '=', 'online_token'),
            ('state', '=', 'pending'),
            ('provider_reference', 'ilike', 'https'),
        ])

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
<?xml version="1.0" encoding="utf-8"?>
<!-- Generator: Adobe Illustrator 26.2.1, SVG Export Plug-In . SVG Version: 6.00 Build 0)  -->
<svg version="1.1" id="Layer_1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" x="0px" y="0px"
	 viewBox="0 0 70 70" style="enable-background:new 0 0 70 70;" xml:space="preserve">
<style type="text/css">
	.st0{filter:url(#Adobe_OpacityMaskFilter);}
	.st1{fill-rule:evenodd;clip-rule:evenodd;fill:#FFFFFF;}
	.st2{mask:url(#b_00000056428700427525433750000007737325548001924286_);}
	.st3{fill-rule:evenodd;clip-rule:evenodd;fill:url(#SVGID_1_);}
	.st4{fill-rule:evenodd;clip-rule:evenodd;fill:#FFFFFF;fill-opacity:0.383;}
	.st5{opacity:0.324;fill-rule:evenodd;clip-rule:evenodd;fill:#393939;enable-background:new    ;}
	.st6{fill-rule:evenodd;clip-rule:evenodd;fill-opacity:0.383;}
	.st7{opacity:0.4;}
	.st8{fill-rule:evenodd;clip-rule:evenodd;}
	.st9{fill:#FFFFFF;}
</style>
<defs>
	<filter id="Adobe_OpacityMaskFilter" filterUnits="userSpaceOnUse" x="0" y="0" width="70" height="70">
		<feColorMatrix  type="matrix" values="1 0 0 0 0  0 1 0 0 0  0 0 1 0 0  0 0 0 1 0"/>
	</filter>
</defs>
<mask maskUnits="userSpaceOnUse" x="0" y="0" width="70" height="70" id="b_00000056428700427525433750000007737325548001924286_">
	<g class="st0">
		<path id="a_00000138559125790419082090000007347613071121435547_" class="st1" d="M4,0h61c4,0,5,1,5,5v60c0,4-1,5-5,5H4
			c-3,0-4-1-4-5V5C0,1,1,0,4,0z"/>
	</g>
</mask>
<g class="st2">
	
		<linearGradient id="SVGID_1_" gradientUnits="userSpaceOnUse" x1="-2234.6001" y1="1879.5037" x2="-2235.6001" y2="1878.5037" gradientTransform="matrix(70 0 0 -70 156492 131565.2656)">
		<stop  offset="0" style="stop-color:#94B6C8"/>
		<stop  offset="1" style="stop-color:#6A9EBA"/>
	</linearGradient>
	<path class="st3" d="M0,0h70v70H0V0z"/>
	<path class="st4" d="M4,1h61c2.7,0,4.3,0.7,5,2V0H0v3C0.7,1.7,2,1,4,1z"/>
	<path class="st5" d="M4,69c-2,0-4-1-4-4V31.9l12.6-12.5c5.9-2.4,10.9-1.3,15.3,3.3c10-7.1,15.8-7.6,11.3,8l7.5-7.4L56.7,24
		l-0.5,23.9L35.2,69H4z"/>
	<path class="st6" d="M4,69h61c2.7,0,4.3-1,5-3v4H0v-4C0.7,68,2,69,4,69z"/>
	<g class="st7">
		<polygon points="19.7,55.6 18.7,55.6 18.7,55 21.5,55 21.5,55.6 20.5,55.6 20.5,58.5 19.7,58.5 19.7,55.6 		"/>
		<polygon points="21.7,55 22.7,55 23.6,57.4 23.6,57.4 24.3,55 25.4,55 25.4,58.5 24.7,58.5 24.7,56 24.7,56 23.8,58.5 23.2,58.5 
			22.4,56 22.4,56 22.4,58.5 21.7,58.5 		"/>
		<path d="M53.3,24.7h-9.9c0,0.9-0.2,1.8-0.4,2.7h10c0.2,0,0.3,0.2,0.3,0.3V31H41.6c-0.9,1.9-2.2,3.9-3.8,5.8c0,0.2,0,0.5,0.1,0.7
			h15.4v9.7c0,0.1,0,0.2-0.1,0.2c-0.1,0.1-0.1,0.1-0.2,0.1H26.6c-0.1,0-0.2,0-0.2-0.1c-0.1-0.1-0.1-0.1-0.1-0.2v-1
			c-0.3-0.1-0.7-0.2-1-0.3c-0.6,0.2-1.2,0.3-1.7,0.4v1.2c0,1.5,1.2,2.7,2.7,2.7h27.1c1.5,0,2.7-1.2,2.7-2.7V27.4
			C56,25.9,54.8,24.7,53.3,24.7z"/>
		<path id="path0_00000051376682181676208090000007751481062063305623_" class="st8" d="M34.3,18.7c-2.7,0.2-5.8,1.6-8.7,3.8
			c-0.4,0.3-0.3,0.3-0.8-0.1c-6.2-4.5-12.3-4.9-14.2-1c-1.6,3.4,0.3,8.9,4.8,14c0.3,0.4,0.4,0.4,0.3,0.5c0,0-0.1,0.3-0.1,0.6
			c-1,5.7,3.1,8.5,9.2,6.4c0.3-0.1,0.5-0.2,0.5-0.1c0,0,0.3,0.1,0.6,0.2c6.1,2.1,10.1-1,8.9-6.8l-0.1-0.3l0.3-0.3
			c4.5-5,6.5-10.8,4.9-14.2C38.9,19.5,36.8,18.4,34.3,18.7 M16.5,21.2c1.9,0.3,4.5,1.5,6.6,3l0.2,0.1L23,24.5
			c-2.7,2.4-5,5.6-6.3,8.4c-0.1,0.3-0.1,0.3-0.3,0c-3.5-4.4-4.9-8.9-3.3-10.9C13.7,21.3,14.9,21,16.5,21.2 M35.6,21.2
			c1.1,0.2,1.8,0.7,2.1,1.6c0.8,2-0.4,5.6-2.9,9.1c-0.5,0.6-1,1.4-1.1,1.3c0,0-0.1-0.2-0.2-0.4c-1.2-2.6-3.4-5.6-5.8-7.9
			c-0.3-0.3-0.5-0.5-0.5-0.5s0.2-0.2,0.4-0.3C30.6,21.9,33.6,20.9,35.6,21.2 M25.5,26.2c2.7,2.4,5,5.6,6.2,8.4
			c0.2,0.6,0.3,0.5-0.2,1c-1.7,1.7-3.8,3.2-5.7,4.2c-0.6,0.3-0.5,0.3-1.2-0.1c-1.7-0.9-3.3-2-4.9-3.4c-0.6-0.5-1.2-1.1-1.2-1.2
			c0,0,0-0.2,0.1-0.3c1-2.5,2.7-5.1,5-7.5c0.5-0.6,1.5-1.5,1.6-1.5C25.2,25.9,25.3,26,25.5,26.2 M18.4,38.5c1,0.9,2.2,1.8,3.3,2.4
			c0.3,0.2,0.3,0.2-0.3,0.2c-2.2,0.2-3.4-0.8-3.5-2.8l0-0.3l0.1,0.1C18,38.1,18.2,38.3,18.4,38.5 M32.5,38.4c-0.1,2.1-1.4,3-3.8,2.7
			c-0.2,0-0.2,0,0.2-0.2c1-0.6,2.3-1.6,3.3-2.5c0.2-0.2,0.3-0.3,0.3-0.3C32.5,38.1,32.5,38.2,32.5,38.4"/>
	</g>
	<g>
		<polygon class="st9" points="21.7,53.6 20.7,53.6 20.7,53 23.5,53 23.5,53.6 22.5,53.6 22.5,56.5 21.7,56.5 21.7,53.6 		"/>
		<polygon class="st9" points="23.7,53 24.7,53 25.6,55.4 25.6,55.4 26.3,53 27.4,53 27.4,56.5 26.7,56.5 26.7,54 26.7,54 
			25.8,56.5 25.2,56.5 24.4,54 24.4,54 24.4,56.5 23.7,56.5 		"/>
		<path class="st9" d="M55.3,22.7h-9.9c0,0.9-0.2,1.8-0.4,2.7h10c0.2,0,0.3,0.2,0.3,0.3V29H43.6c-0.9,1.9-2.2,3.9-3.8,5.8
			c0,0.2,0,0.5,0.1,0.7h15.4v9.7c0,0.1,0,0.2-0.1,0.2c-0.1,0.1-0.1,0.1-0.2,0.1H28.6c-0.1,0-0.2,0-0.2-0.1c-0.1-0.1-0.1-0.1-0.1-0.2
			v-1c-0.3-0.1-0.7-0.2-1-0.3c-0.6,0.2-1.2,0.3-1.7,0.4v1.2c0,1.5,1.2,2.7,2.7,2.7h27.1c1.5,0,2.7-1.2,2.7-2.7V25.4
			C58,23.9,56.8,22.7,55.3,22.7z"/>
		<path id="path0_00000123403326104996206480000014033077542406494338_" class="st1" d="M36.3,16.7c-2.7,0.2-5.8,1.6-8.7,3.8
			c-0.4,0.3-0.3,0.3-0.8-0.1c-6.2-4.5-12.3-4.9-14.2-1c-1.6,3.4,0.3,8.9,4.8,14c0.3,0.4,0.4,0.4,0.3,0.5c0,0-0.1,0.3-0.1,0.6
			c-1,5.7,3.1,8.5,9.2,6.4c0.3-0.1,0.5-0.2,0.5-0.1c0,0,0.3,0.1,0.6,0.2c6.1,2.1,10.1-1,8.9-6.8l-0.1-0.3l0.3-0.3
			c4.5-5,6.5-10.8,4.9-14.2C40.9,17.5,38.8,16.4,36.3,16.7 M18.5,19.2c1.9,0.3,4.5,1.5,6.6,3l0.2,0.1L25,22.5
			c-2.7,2.4-5,5.6-6.3,8.4c-0.1,0.3-0.1,0.3-0.3,0c-3.5-4.4-4.9-8.9-3.3-10.9C15.7,19.3,16.9,19,18.5,19.2 M37.6,19.2
			c1.1,0.2,1.8,0.7,2.1,1.6c0.8,2-0.4,5.6-2.9,9.1c-0.5,0.6-1,1.4-1.1,1.3c0,0-0.1-0.2-0.2-0.4c-1.2-2.6-3.4-5.6-5.8-7.9
			c-0.3-0.3-0.5-0.5-0.5-0.5c0,0,0.2-0.2,0.4-0.3C32.6,19.9,35.6,18.9,37.6,19.2 M27.5,24.2c2.7,2.4,5,5.6,6.2,8.4
			c0.2,0.6,0.3,0.5-0.2,1c-1.7,1.7-3.8,3.2-5.7,4.2c-0.6,0.3-0.5,0.3-1.2-0.1c-1.7-0.9-3.3-2-4.9-3.4c-0.6-0.5-1.2-1.1-1.2-1.2
			c0,0,0-0.2,0.1-0.3c1-2.5,2.7-5.1,5-7.5c0.5-0.6,1.5-1.5,1.6-1.5C27.2,23.9,27.3,24,27.5,24.2 M20.4,36.5c1,0.9,2.2,1.8,3.3,2.4
			c0.3,0.2,0.3,0.2-0.3,0.2c-2.2,0.2-3.4-0.8-3.5-2.8l0-0.3l0.1,0.1C20,36.1,20.2,36.3,20.4,36.5 M34.5,36.4c-0.1,2.1-1.4,3-3.8,2.7
			c-0.2,0-0.2,0,0.2-0.2c1-0.6,2.3-1.6,3.3-2.5c0.2-0.2,0.3-0.3,0.3-0.3C34.5,36.1,34.5,36.2,34.5,36.4"/>
	</g>
</g>
</svg>

```

## File: static\src\js\payment_form.js

```javascript
odoo.define('payment_flutterwave.payment_form', require => {
    'use strict';

    const checkoutForm = require('payment.checkout_form');
    const manageForm = require('payment.manage_form');

    const flutterwaveMixin = {
        /**
         * Allow forcing redirect to authorization url for Flutterwave token flow.
         *
         * @override method from payment.payment_form_mixin
         * @private
         * @param {string} provider_code - The code of the token's provider
         * @param {number} tokenId - The id of the token handling the transaction
         * @param {object} processingValues - The processing values of the transaction
         * @return {undefined}
         */
        _processTokenPayment: (provider_code, tokenId, processingValues) => {
            if (provider_code === 'flutterwave' && processingValues.auth_redirect_form_html) {
                // Append the redirect form to the body
                const $redirectForm = $(processingValues.auth_redirect_form_html).attr(
                    'id', 'o_payment_redirect_form'
                );

                // Authorization happens via POST instead of GET
                $redirectForm[0].setAttribute('method', 'post');

                // Ensures external redirections when in an iframe.
                $redirectForm[0].setAttribute('target', '_top');
                $(document.getElementsByTagName('body')[0]).append($redirectForm);

                // Submit the form
                $redirectForm.submit();
            } else {
                this._super(provider_code, tokenId, processingValues);
            }
        }
    };

    checkoutForm.include(flutterwaveMixin);
    manageForm.include(flutterwaveMixin);
});

```

## File: views\payment_flutterwave_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="redirect_form">
        <form t-att-action="api_url" method="get"/>
    </template>

</odoo>

```

## File: views\payment_provider_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_provider_form" model="ir.ui.view">
        <field name="name">Flutterwave Provider Form</field>
        <field name="model">payment.provider</field>
        <field name="inherit_id" ref="payment.payment_provider_form"/>
        <field name="arch" type="xml">
            <group name="provider_credentials" position="inside">
                <group attrs="{'invisible': [('code', '!=', 'flutterwave')]}"
                       name="flutterwave_credentials">
                    <field name="flutterwave_public_key"
                           string="Public Key"
                           attrs="{'required':[('code', '=', 'flutterwave'), ('state', '!=', 'disabled')]}"/>
                    <field name="flutterwave_secret_key"
                           string="Secret Key"
                           attrs="{'required':[('code', '=', 'flutterwave'), ('state', '!=', 'disabled')]}"
                           password="True"/>
                    <field name="flutterwave_webhook_secret"
                           string="Webhook Secret"
                           attrs="{'required':[('code', '=', 'flutterwave'), ('state', '!=', 'disabled')]}"
                           password="True"/>
                </group>
            </group>
        </field>
    </record>

</odoo>

```

