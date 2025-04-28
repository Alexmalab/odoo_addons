# Odoo Module: payment_aps

Category: Accounting/Payment Providers

This file contains the source code of the Odoo module.

## File: const.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Mapping of transaction states to APS payment statuses.
# See https://paymentservices-reference.payfort.com/docs/api/build/index.html#transactions-response-codes.
PAYMENT_STATUS_MAPPING = {
    'pending': ('19',),
    'done': ('14',),
}

```

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

from odoo.addons.payment import setup_provider, reset_payment_provider


def post_init_hook(cr, registry):
    setup_provider(cr, registry, 'aps')


def uninstall_hook(cr, registry):
    reset_payment_provider(cr, registry, 'aps')

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': "Payment Provider: Amazon Payment Services",
    'version': '1.0',
    'category': 'Accounting/Payment Providers',
    'sequence': 350,
    'summary': "An Amazon payment provider covering the MENA region.",
    'description': " ",  # Non-empty string to avoid loading the README file.
    'depends': ['payment'],
    'data': [
        'views/payment_aps_templates.xml',
        'views/payment_provider_views.xml',

        'data/payment_provider_data.xml',
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


_logger = logging.getLogger(__name__)


class APSController(http.Controller):
    _return_url = '/payment/aps/return'
    _webhook_url = '/payment/aps/webhook'

    @http.route(
        _return_url, type='http', auth='public', methods=['POST'], csrf=False, save_session=False
    )
    def aps_return_from_checkout(self, **data):
        """ Process the notification data sent by APS after redirection.

        The route is flagged with `save_session=False` to prevent Odoo from assigning a new session
        to the user if they are redirected to this route with a POST request. Indeed, as the session
        cookie is created without a `SameSite` attribute, some browsers that don't implement the
        recommended default `SameSite=Lax` behavior will not include the cookie in the redirection
        request from the payment provider to Odoo. As the redirection to the '/payment/status' page
        will satisfy any specification of the `SameSite` attribute, the session of the user will be
        retrieved and with it the transaction which will be immediately post-processed.

        :param dict data: The notification data.
        """
        _logger.info("Handling redirection from APS with data:\n%s", pprint.pformat(data))

        # Check the integrity of the notification.
        tx_sudo = request.env['payment.transaction'].sudo()._get_tx_from_notification_data(
            'aps', data
        )
        self._verify_notification_signature(data, tx_sudo)

        # Handle the notification data.
        tx_sudo._handle_notification_data('aps', data)
        return request.redirect('/payment/status')

    @http.route(_webhook_url, type='http', auth='public', methods=['POST'], csrf=False)
    def aps_webhook(self, **data):
        """ Process the notification data sent by APS to the webhook.

        See https://paymentservices-reference.payfort.com/docs/api/build/index.html#transaction-feedback.

        :param dict data: The notification data.
        :return: The 'SUCCESS' string to acknowledge the notification
        :rtype: str
        """
        _logger.info("Notification received from APS with data:\n%s", pprint.pformat(data))
        try:
            # Check the integrity of the notification.
            tx_sudo = request.env['payment.transaction'].sudo()._get_tx_from_notification_data(
                'aps', data
            )
            self._verify_notification_signature(data, tx_sudo)

            # Handle the notification data.
            tx_sudo._handle_notification_data('aps', data)
        except ValidationError:  # Acknowledge the notification to avoid getting spammed.
            _logger.exception("Unable to handle the notification data; skipping to acknowledge.")

        return ''  # Acknowledge the notification.

    @staticmethod
    def _verify_notification_signature(notification_data, tx_sudo):
        """ Check that the received signature matches the expected one.

        :param dict notification_data: The notification data
        :param recordset tx_sudo: The sudoed transaction referenced by the notification data, as a
                                  `payment.transaction` record
        :return: None
        :raise: :class:`werkzeug.exceptions.Forbidden` if the signatures don't match
        """
        received_signature = notification_data.get('signature')
        if not received_signature:
            _logger.warning("received notification with missing signature")
            raise Forbidden()

        # Compare the received signature with the expected signature computed from the data.
        expected_signature = tx_sudo.provider_id._aps_calculate_signature(
            notification_data, incoming=True
        )
        if not hmac.compare_digest(received_signature, expected_signature):
            _logger.warning("received notification with invalid signature")
            raise Forbidden()

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\neutralize.sql

```sql
-- disable aps payment provider
UPDATE payment_provider
   SET aps_merchant_identifier = NULL,
       aps_access_code = NULL,
       aps_sha_request = NULL,
       aps_sha_response = NULL;

```

## File: data\payment_provider_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="payment.payment_provider_aps" model="payment.provider">
        <field name="code">aps</field>
        <field name="redirect_form_view_id" ref="redirect_form"/>
    </record>

</odoo>

```

## File: models\payment_provider.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import hashlib
import logging

from odoo import fields, models


_logger = logging.getLogger(__name__)


class PaymentProvider(models.Model):
    _inherit = 'payment.provider'

    code = fields.Selection(
        selection_add=[('aps', "Amazon Payment Services")], ondelete={'aps': 'set default'}
    )
    aps_merchant_identifier = fields.Char(
        string="APS Merchant Identifier",
        help="The code of the merchant account to use with this provider.",
        required_if_provider='aps',
    )
    aps_access_code = fields.Char(
        string="APS Access Code",
        help="The access code associated with the merchant account.",
        required_if_provider='aps',
        groups='base.group_system',
    )
    aps_sha_request = fields.Char(
        string="APS SHA Request Phrase",
        required_if_provider='aps',
        groups='base.group_system',
    )
    aps_sha_response = fields.Char(
        string="APS SHA Response Phrase",
        required_if_provider='aps',
        groups='base.group_system',
    )

    #=== BUSINESS METHODS ===#

    def _aps_get_api_url(self):
        if self.state == 'enabled':
            return 'https://checkout.payfort.com/FortAPI/paymentPage'
        else:  # 'test'
            return 'https://sbcheckout.payfort.com/FortAPI/paymentPage'

    def _aps_calculate_signature(self, data, incoming=True):
        """ Compute the signature for the provided data according to the APS documentation.

        :param dict data: The data to sign.
        :param bool incoming: Whether the signature must be generated for an incoming (APS to Odoo)
                              or outgoing (Odoo to APS) communication.
        :return: The calculated signature.
        :rtype: str
        """
        sign_data = ''.join([f'{k}={v}' for k, v in sorted(data.items()) if k != 'signature'])
        key = self.aps_sha_response if incoming else self.aps_sha_request
        signing_string = ''.join([key, sign_data, key])
        return hashlib.sha256(signing_string.encode()).hexdigest()

```

## File: models\payment_transaction.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from werkzeug import urls

from odoo import _, api, models
from odoo.exceptions import ValidationError

from odoo.addons.payment import utils as payment_utils
from odoo.addons.payment_aps.const import PAYMENT_STATUS_MAPPING
from odoo.addons.payment_aps.controllers.main import APSController


_logger = logging.getLogger(__name__)


class PaymentTransaction(models.Model):
    _inherit = 'payment.transaction'

    @api.model
    def _compute_reference(self, provider_code, prefix=None, separator='-', **kwargs):
        """ Override of `payment` to ensure that APS' requirements for references are satisfied.

        APS' requirements for transaction are as follows:
        - References can only be made of alphanumeric characters and/or '-' and '_'.
          The prefix is generated with 'tx' as default. This prevents the prefix from being
          generated based on document names that may contain non-allowed characters
          (eg: INV/2020/...).

        :param str provider_code: The code of the provider handling the transaction.
        :param str prefix: The custom prefix used to compute the full reference.
        :param str separator: The custom separator used to separate the prefix from the suffix.
        :return: The unique reference for the transaction.
        :rtype: str
        """
        if provider_code == 'aps':
            prefix = payment_utils.singularize_reference_prefix()

        return super()._compute_reference(provider_code, prefix=prefix, separator=separator, **kwargs)

    def _get_specific_rendering_values(self, processing_values):
        """ Override of `payment` to return APS-specific processing values.

        Note: self.ensure_one() from `_get_processing_values`

        :param dict processing_values: The generic processing values of the transaction.
        :return: The dict of provider-specific processing values.
        :rtype: dict
        """
        res = super()._get_specific_rendering_values(processing_values)
        if self.provider_code != 'aps':
            return res

        converted_amount = payment_utils.to_minor_currency_units(self.amount, self.currency_id)
        base_url = self.provider_id.get_base_url()
        rendering_values = {
            'command': 'PURCHASE',
            'access_code': self.provider_id.aps_access_code,
            'merchant_identifier': self.provider_id.aps_merchant_identifier,
            'merchant_reference': self.reference,
            'amount': str(converted_amount),
            'currency': self.currency_id.name,
            'language': self.partner_lang[:2],
            'customer_email': self.partner_id.email_normalized,
            'return_url': urls.url_join(base_url, APSController._return_url),
        }
        rendering_values.update({
            'signature': self.provider_id._aps_calculate_signature(
                rendering_values, incoming=False
            ),
            'api_url': self.provider_id._aps_get_api_url(),
        })
        return rendering_values

    def _get_tx_from_notification_data(self, provider_code, notification_data):
        """ Override of `payment` to find the transaction based on APS data.

        :param str provider_code: The code of the provider that handled the transaction.
        :param dict notification_data: The notification data sent by the provider.
        :return: The transaction if found.
        :rtype: recordset of `payment.transaction`
        :raise ValidationError: If inconsistent data are received.
        :raise ValidationError: If the data match no transaction.
        """
        tx = super()._get_tx_from_notification_data(provider_code, notification_data)
        if provider_code != 'aps' or len(tx) == 1:
            return tx

        reference = notification_data.get('merchant_reference')
        if not reference:
            raise ValidationError(
                "APS: " + _("Received data with missing reference %(ref)s.", ref=reference)
            )

        tx = self.search([('reference', '=', reference), ('provider_code', '=', 'aps')])
        if not tx:
            raise ValidationError(
                "APS: " + _("No transaction found matching reference %s.", reference)
            )

        return tx

    def _process_notification_data(self, notification_data):
        """ Override of `payment' to process the transaction based on APS data.

        Note: self.ensure_one()

        :param dict notification_data: The notification data sent by the provider.
        :return: None
        :raise ValidationError: If inconsistent data are received.
        """
        super()._process_notification_data(notification_data)
        if self.provider_code != 'aps':
            return

        self.provider_reference = notification_data.get('fort_id')

        status = notification_data.get('status')
        if not status:
            raise ValidationError("APS: " + _("Received data with missing payment state."))

        if status in PAYMENT_STATUS_MAPPING['pending']:
            self._set_pending()
        elif status in PAYMENT_STATUS_MAPPING['done']:
            self._set_done()
        else:  # Classify unsupported payment state as `error` tx state.
            status_description = notification_data.get('response_message')
            _logger.info(
                "Received data with invalid payment status (%(status)s) and reason '%(reason)s' "
                "for transaction with reference %(ref)s",
                {'status': status, 'reason': status_description, 'ref': self.reference},
            )
            self._set_error("APS: " + _(
                "Received invalid transaction status %(status)s and reason '%(reason)s'.",
                status=status, reason=status_description
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
<svg width="70" height="70" viewBox="0 0 70 70" fill="none" xmlns="http://www.w3.org/2000/svg">
    <mask id="mask0_2_36" style="mask-type:alpha" maskUnits="userSpaceOnUse" x="0" y="0" width="70" height="70">
    <path d="M4 0H65C69 0 70 1 70 5V65C70 69 69 70 65 70H4C1 70 0 69 0 65V5C0 1 1 0 4 0Z" fill="white"/>
    </mask>
    <g mask="url(#mask0_2_36)">
    <path fill-rule="evenodd" clip-rule="evenodd" d="M0 0H70V70H0V0Z" fill="url(#paint0_linear_2_36)"/>
    <path fill-rule="evenodd" clip-rule="evenodd" d="M4 1H65C67.667 1 69.333 1.667 70 3V0H0V3C0.667 1.667 2 1 4 1Z" fill="white" fill-opacity="0.383"/>
    <path fill-rule="evenodd" clip-rule="evenodd" d="M4 69H65C67.667 69 69.333 68 70 66V70H0V66C0.667 68 2 69 4 69Z" fill="black" fill-opacity="0.383"/>
    </g>
    <path opacity="0.324" fill-rule="evenodd" clip-rule="evenodd" d="M4 69C2 69 0 68 0 65V33.916C0 33.916 18 23 19.5 22.5C21 22 23 22 23 22L28.7087 22.703H37.25L48 23L51.576 24.968L51.34 46.978L39.224 69H4Z" fill="#393939"/>
    <path d="M30.3879 41.7839C21.4712 46.0275 15.9374 42.477 12.395 40.3205C12.1758 40.1846 11.8033 40.3523 12.1265 40.7236C13.3067 42.1545 17.1742 45.6035 22.2225 45.6035C27.2743 45.6035 30.2796 42.847 30.6556 42.3662C31.029 41.8894 30.7653 41.6264 30.3879 41.7839ZM32.8921 40.4009C32.6527 40.0891 31.4361 40.031 30.6705 40.125C29.9036 40.2164 28.7527 40.685 28.8527 40.9664C28.9041 41.0718 29.0089 41.0246 29.5357 40.9772C30.0639 40.9245 31.5436 40.7377 31.852 41.1408C32.1617 41.5467 31.38 43.4801 31.2372 43.7919C31.0993 44.1037 31.2899 44.1841 31.549 43.9764C31.8046 43.7688 32.2672 43.2312 32.5777 42.4704C32.886 41.7055 33.0741 40.6383 32.8921 40.4009Z" fill="#465C68"/>
    <path fill-rule="evenodd" clip-rule="evenodd" d="M24.3924 33.1249C24.3924 34.2383 24.4205 35.1669 23.8577 36.1557C23.4035 36.9597 22.6839 37.4541 21.88 37.4541C20.7826 37.4541 20.1435 36.618 20.1435 35.384C20.1435 32.948 22.3261 32.5058 24.3924 32.5058V33.1249ZM27.2745 40.0912C27.0856 40.2599 26.8122 40.2721 26.5992 40.1594C25.6506 39.3716 25.4817 39.0058 24.9592 38.2541C23.3915 39.854 22.282 40.3323 20.248 40.3323C17.8442 40.3323 15.9709 38.849 15.9709 35.8784C15.9709 33.559 17.2292 31.9792 19.0179 31.2075C20.5696 30.5241 22.7362 30.4035 24.3924 30.2146V29.8448C24.3924 29.1654 24.4446 28.3615 24.0467 27.7746C23.6969 27.248 23.0297 27.0309 22.4427 27.0309C21.3534 27.0309 20.3806 27.5896 20.1435 28.7473C20.0952 29.0047 19.9063 29.2579 19.649 29.2699L16.8754 28.9725C16.6423 28.9202 16.385 28.7313 16.4493 28.3735C17.0885 25.013 20.1233 24 22.8406 24C24.2315 24 26.0484 24.3699 27.1458 25.4231C28.5367 26.7214 28.404 28.4539 28.404 30.3392V34.7931C28.404 36.1317 28.9587 36.7185 29.4813 37.4421C29.6661 37.6994 29.7064 38.009 29.4732 38.2018C28.8903 38.6882 27.8532 39.5927 27.2825 40.0992L27.2745 40.0912Z" fill="#465C68"/>
    <path d="M30.3879 39.7839C21.4712 44.0275 15.9374 40.477 12.395 38.3205C12.1758 38.1846 11.8033 38.3523 12.1265 38.7236C13.3067 40.1545 17.1742 43.6035 22.2225 43.6035C27.2743 43.6035 30.2796 40.847 30.6556 40.3662C31.029 39.8894 30.7653 39.6264 30.3879 39.7839ZM32.8921 38.4009C32.6527 38.0891 31.4361 38.031 30.6705 38.125C29.9036 38.2164 28.7527 38.685 28.8527 38.9664C28.9041 39.0718 29.0089 39.0246 29.5357 38.9772C30.0639 38.9245 31.5436 38.7377 31.852 39.1408C32.1617 39.5467 31.38 41.4801 31.2372 41.7919C31.0993 42.1037 31.2899 42.1841 31.549 41.9764C31.8046 41.7688 32.2672 41.2312 32.5777 40.4704C32.886 39.7055 33.0741 38.6383 32.8921 38.4009Z" fill="white"/>
    <path fill-rule="evenodd" clip-rule="evenodd" d="M24.3924 31.1249C24.3924 32.2383 24.4205 33.1669 23.8577 34.1557C23.4035 34.9597 22.6839 35.4541 21.88 35.4541C20.7826 35.4541 20.1435 34.618 20.1435 33.384C20.1435 30.948 22.3261 30.5058 24.3924 30.5058V31.1249ZM27.2745 38.0912C27.0856 38.2599 26.8122 38.2721 26.5992 38.1594C25.6506 37.3716 25.4817 37.0058 24.9592 36.2541C23.3915 37.854 22.282 38.3323 20.248 38.3323C17.8442 38.3323 15.9709 36.849 15.9709 33.8784C15.9709 31.559 17.2292 29.9792 19.0179 29.2075C20.5696 28.5241 22.7362 28.4035 24.3924 28.2146V27.8448C24.3924 27.1654 24.4446 26.3615 24.0467 25.7746C23.6969 25.248 23.0297 25.0309 22.4427 25.0309C21.3534 25.0309 20.3806 25.5896 20.1435 26.7473C20.0952 27.0047 19.9063 27.2579 19.649 27.2699L16.8754 26.9725C16.6423 26.9202 16.385 26.7313 16.4493 26.3735C17.0885 23.013 20.1233 22 22.8406 22C24.2315 22 26.0484 22.3699 27.1458 23.4231C28.5367 24.7214 28.404 26.4539 28.404 28.3392V32.7931C28.404 34.1317 28.9587 34.7185 29.4813 35.4421C29.6661 35.6994 29.7064 36.009 29.4732 36.2018C28.8903 36.6882 27.8532 37.5927 27.2825 38.0992L27.2745 38.0912Z" fill="white"/>
    <path d="M32 37.5C31.5 37.5 30.9327 36.408 30.9327 35.4908L31.0317 31H49.036V27.788C49.0363 27.6976 49.0008 27.6108 48.9372 27.5466C48.8737 27.4823 48.7874 27.4458 48.697 27.445H31.0317C30.8494 26.7368 30.5144 26.0864 29.9023 25.5151C29.5571 25.1837 29.1449 24.9159 28.6974 24.703H49.036C50.536 24.703 51.75 25.931 51.75 27.445V47.555C51.75 49.069 50.537 50.297 49.036 50.297H21.964C20.464 50.297 19.25 49.069 19.25 47.555H21.964C21.964 47.555 22.116 47.555 22.303 47.555H48.697C48.7874 47.5542 48.8737 47.5177 48.9372 47.4534C49.0008 47.3892 49.0363 47.3024 49.036 47.212V37.5C49.036 37.5 32.5 37.5 32 37.5Z" fill="#465C68"/>
    <path d="M15.459 55.642H14.423V55H17.256V55.642H16.219V58.474H15.459V55.642Z" fill="#465C68"/>
    <path d="M17.422 55H18.492L19.301 57.389H19.311L20.075 55H21.145V58.474H20.434V56.012H20.424L19.577 58.474H18.991L18.143 56.036H18.133V58.474H17.422V55Z" fill="#465C68"/>
    <path d="M32 35.5C31.5 35.5 30.9327 34.408 30.9327 33.4908L31.0317 29H49.036V25.788C49.0363 25.6976 49.0008 25.6108 48.9372 25.5466C48.8737 25.4823 48.7874 25.4458 48.697 25.445H31.0317C30.8494 24.7368 30.5144 24.0864 29.9023 23.5151C29.5571 23.1837 29.1449 22.9159 28.6974 22.703H49.036C50.536 22.703 51.75 23.931 51.75 25.445V45.555C51.75 47.069 50.537 48.297 49.036 48.297H21.964C20.464 48.297 19.25 47.069 19.25 45.555H21.964C21.964 45.555 22.116 45.555 22.303 45.555H48.697C48.7874 45.5542 48.8737 45.5177 48.9372 45.4534C49.0008 45.3892 49.0363 45.3024 49.036 45.212V35.5C49.036 35.5 32.5 35.5 32 35.5Z" fill="white"/>
    <path d="M15.459 53.642H14.423V53H17.256V53.642H16.219V56.474H15.459V53.642Z" fill="white"/>
    <path d="M17.422 53H18.492L19.301 55.389H19.311L20.075 53H21.145V56.474H20.434V54.012H20.424L19.577 56.474H18.991L18.143 54.036H18.133V56.474H17.422V53Z" fill="white"/>
    <defs>
    <linearGradient id="paint0_linear_2_36" x1="70" y1="0" x2="0" y2="70" gradientUnits="userSpaceOnUse">
    <stop stop-color="#DA956B"/>
    <stop offset="1" stop-color="#CC7039"/>
    </linearGradient>
    </defs>
</svg>
```

## File: views\payment_aps_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="redirect_form">
        <form t-att-action="api_url" method="post">
            <input type="hidden" name="command" t-att-value="command"/>
            <input type="hidden" name="access_code" t-att-value="access_code"/>
            <input type="hidden" name="merchant_identifier" t-att-value="merchant_identifier"/>
            <input type="hidden" name="merchant_reference" t-att-value="merchant_reference"/>
            <input type="hidden" name="amount" t-att-value="amount"/>
            <input type="hidden" name="currency" t-att-value="currency"/>
            <input type="hidden" name="language" t-att-value="language"/>
            <input type="hidden" name="customer_email" t-att-value="customer_email"/>
            <input type="hidden" name="signature" t-att-value="signature"/>
            <input type="hidden" name="return_url" t-att-value="return_url"/>
        </form>
    </template>

</odoo>

```

## File: views\payment_provider_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_provider_form" model="ir.ui.view">
        <field name="name">APS Provider Form</field>
        <field name="model">payment.provider</field>
        <field name="inherit_id" ref="payment.payment_provider_form"/>
        <field name="arch" type="xml">
            <group name="provider_credentials" position='inside'>
                <group attrs="{'invisible': [('code', '!=', 'aps')]}">
                    <field name="aps_merchant_identifier"
                           string="Merchant Identifier"
                           attrs="{'required': [('code', '=', 'aps'), ('state', '!=', 'disabled')]}"/>
                    <field name="aps_access_code"
                           string="Access Code"
                           attrs="{'required': [('code', '=', 'aps'), ('state', '!=', 'disabled')]}"
                           password="True"/>
                    <field name="aps_sha_request"
                           string="SHA Request Phrase"
                           attrs="{'required': [('code', '=', 'aps'), ('state', '!=', 'disabled')]}"
                           password="True"/>
                    <field name="aps_sha_response"
                           string="SHA Response Phrase"
                           attrs="{'required': [('code', '=', 'aps'), ('state', '!=', 'disabled')]}"
                           password="True"/>
                </group>
            </group>
        </field>
    </record>

</odoo>

```

