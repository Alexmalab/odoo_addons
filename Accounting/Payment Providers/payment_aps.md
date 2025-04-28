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

# The codes of the payment methods to activate when APS is activated.
DEFAULT_PAYMENT_METHOD_CODES = {
    # Primary payment methods.
    'card',
    # Brand payment methods.
    'visa',
    'mastercard',
    'amex',
    'discover',
}

```

## File: utils.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

def get_payment_option(payment_method_code):
    """ Map the payment method code to one of the payment options expected by APS.

    As APS expects the specific card brand (e.g, VISA) rather than the generic 'card' option, we
    skip the mapping and return an empty string when the provided payment method code is 'card'.
    This allows the user to select the desired brand on APS' checkout page.

    :param str payment_method_code: The code of the payment method.
    :return: The corresponding APS' payment option.
    :rtype: str
    """
    return payment_method_code.upper() if payment_method_code != 'card' else ''

```

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

from odoo.addons.payment import setup_provider, reset_payment_provider


def post_init_hook(env):
    setup_provider(env, 'aps')


def uninstall_hook(env):
    reset_payment_provider(env, 'aps')

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

from odoo.addons.payment_aps import const


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

    def _get_default_payment_method_codes(self):
        """ Override of `payment` to return the default payment method codes. """
        default_codes = super()._get_default_payment_method_codes()
        if self.code != 'aps':
            return default_codes
        return const.DEFAULT_PAYMENT_METHOD_CODES

```

## File: models\payment_transaction.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from werkzeug import urls

from odoo import _, api, models
from odoo.exceptions import ValidationError

from odoo.addons.payment import utils as payment_utils
from odoo.addons.payment_aps import utils as aps_utils
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
        payment_option = aps_utils.get_payment_option(self.payment_method_id.code)
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
        if payment_option:  # Not included if the payment method is 'card'.
            rendering_values['payment_option'] = payment_option
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

        # Update the provider reference.
        self.provider_reference = notification_data.get('fort_id')

        # Update the payment method.
        payment_option = notification_data.get('payment_option', '')
        payment_method = self.env['payment.method']._get_from_code(payment_option.lower())
        self.payment_method_id = payment_method or self.payment_method_id

        # Update the payment state.
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
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M40.84 38.587c-17.865 8.236-28.952 1.345-36.049-2.84-.439-.264-1.185.062-.538.782C6.619 39.306 14.366 46 24.482 46c10.12 0 16.142-5.35 16.895-6.283.748-.925.22-1.436-.536-1.13Zm5.017-2.684c-.48-.605-2.917-.718-4.45-.535-1.537.177-3.843 1.086-3.643 1.632.103.205.313.113 1.369.021 1.058-.102 4.022-.464 4.64.318.62.788-.946 4.54-1.231 5.145-.277.605.105.761.624.358.512-.403 1.44-1.446 2.061-2.923.618-1.484.995-3.555.63-4.016Z" fill="#F90"/><path d="M36.319 25.513V11.721C36.319 9.352 33.953 4 25.45 4c-8.5 0-13.023 5.146-13.023 9.782l7.105.616s1.582-4.633 5.255-4.633c3.674 0 3.422 2.882 3.422 3.505v2.997c-4.706.153-16.39 1.455-16.39 11 0 10.264 13.374 10.694 17.76 4.06.169.27.361.533.602.78 1.614 1.644 3.767 3.602 3.767 3.602l5.485-5.25c.002-.002-3.115-2.372-3.115-4.946Zm-16.252.484c0-4.408 4.876-5.302 8.144-5.407v3.794c-.001 7.517-8.144 6.38-8.144 1.613Z"/><path d="M43.105 4h.972V.818h1.108V0H42v.818h1.105V4Zm2.775 0h.858V1.453h.053L47.663 4h.555l.871-2.547h.056V4H50V0h-1.108l-.924 2.714h-.05L46.99 0h-1.11v4Z" fill="#D1D5DB"/></svg>

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
            <input type="hidden" name="payment_option" t-att-value="payment_option" t-if="payment_option"/>
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
                <group invisible="code != 'aps'">
                    <field name="aps_merchant_identifier"
                           string="Merchant Identifier"
                           required="code == 'aps' and state != 'disabled'"/>
                    <field name="aps_access_code"
                           string="Access Code"
                           required="code == 'aps' and state != 'disabled'"
                           password="True"/>
                    <field name="aps_sha_request"
                           string="SHA Request Phrase"
                           required="code == 'aps' and state != 'disabled'"
                           password="True"/>
                    <field name="aps_sha_response"
                           string="SHA Response Phrase"
                           required="code == 'aps' and state != 'disabled'"
                           password="True"/>
                </group>
            </group>
        </field>
    </record>

</odoo>

```

