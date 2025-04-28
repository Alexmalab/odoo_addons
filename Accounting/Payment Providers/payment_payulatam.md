# Odoo Module: payment_payulatam

Category: Accounting/Payment Providers

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

from odoo.exceptions import UserError
from odoo.tools import config

from odoo.addons.payment import setup_provider, reset_payment_provider


def pre_init_hook(cr):
    if not any(config.get(key) for key in ('init', 'update')):
        raise UserError(
            "This module is deprecated and cannot be installed. "
            "Consider installing the Payment Provider: Mercado Pago module instead.")


def post_init_hook(cr, registry):
    setup_provider(cr, registry, 'payulatam')


def uninstall_hook(cr, registry):
    reset_payment_provider(cr, registry, 'payulatam')

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Payment Provider: PayU Latam',
    'version': '2.0',
    'category': 'Accounting/Payment Providers',
    'sequence': 350,
    'summary': "This module is deprecated.",
    'description': " ",  # Non-empty string to avoid loading the README file.
    'depends': ['payment'],
    'data': [
        'views/payment_payulatam_templates.xml',
        'views/payment_provider_views.xml',

        'data/payment_provider_data.xml',
    ],
    'application': False,
    'pre_init_hook': 'pre_init_hook',
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
from odoo.tools import consteq

_logger = logging.getLogger(__name__)


class PayuLatamController(http.Controller):
    _return_url = '/payment/payulatam/return'
    _webhook_url = '/payment/payulatam/webhook'

    @http.route(_return_url, type='http', auth='public', methods=['GET'])
    def payulatam_return_from_checkout(self, **data):
        """ Process the notification data sent by PayU Latam after redirection from checkout.

        See http://developers.payulatam.com/latam/en/docs/integrations/webcheckout-integration/response-page.html.

        :param dict data: The notification data
        """
        _logger.info("handling redirection from PayU Latam with data:\n%s", pprint.pformat(data))

        # Check the integrity of the notification
        tx_sudo = request.env['payment.transaction'].sudo()._get_tx_from_notification_data(
            'payulatam', data
        )
        self._verify_notification_signature(data, tx_sudo)

        # Handle the notification data
        tx_sudo._handle_notification_data('payulatam', data)
        return request.redirect('/payment/status')

    @http.route(_webhook_url, type='http', auth='public', methods=['POST'], csrf=False)
    def payulatam_webhook(self, **raw_data):
        """ Process the notification data sent by PayU Latam to the webhook.

        See http://developers.payulatam.com/latam/en/docs/integrations/webcheckout-integration/confirmation-page.html.

        :param dict raw_data: The un-formatted notification data
        :return: An empty string to acknowledge the notification
        :rtype: str
        """
        _logger.info(
            "notification received from PayU Latam with data:\n%s", pprint.pformat(raw_data)
        )
        data = self._normalize_data_keys(raw_data)

        try:
            # Check the origin and integrity of the notification
            tx_sudo = request.env['payment.transaction'].sudo().with_context(
                payulatam_is_confirmation_page=True
            )._get_tx_from_notification_data('payulatam', data)
            self._verify_notification_signature(data, tx_sudo)  # Use the normalized data.

            # Handle the notification data
            tx_sudo._handle_notification_data('payulatam', data)
        except ValidationError:  # Acknowledge the notification to avoid getting spammed
            _logger.exception("unable to handle the notification data; skipping to acknowledge")

        return ''

    @staticmethod
    def _normalize_data_keys(webhook_notification_data):
        """ Reshape the webhook notification data to process them as redirect notification data.

        :param dict webhook_notification_data: The webhook notification data
        :return: The normalized notification data
        :rtype: dict
        """
        state_pol = webhook_notification_data.get('state_pol')
        if state_pol == '4':
            lap_transaction_state = 'APPROVED'
        elif state_pol == '6':
            lap_transaction_state = 'DECLINED'
        elif state_pol == '5':
            lap_transaction_state = 'EXPIRED'
        else:
            lap_transaction_state = f'INVALID state_pol {state_pol}'
        return {
            'lapTransactionState': lap_transaction_state,
            'transactionState': webhook_notification_data.get('state_pol'),
            'TX_VALUE': webhook_notification_data.get('value'),
            'currency': webhook_notification_data.get('currency'),
            'referenceCode': webhook_notification_data.get('reference_sale'),
            'transactionId': webhook_notification_data.get('transaction_id'),
            'message': webhook_notification_data.get('response_message_pol'),
            'signature': webhook_notification_data.get('sign'),
        }

    @staticmethod
    def _verify_notification_signature(notification_data, tx_sudo):
        """ Check that the received signature matches the expected one.

        :param dict notification_data: The notification data
        :param recordset tx_sudo: The sudoed transaction referenced by the notification data, as a
                                  `payment.transaction` record
        :return: None
        :raise: :class:`werkzeug.exceptions.Forbidden` if the signatures don't match
        """
        # Retrieve the received signature from the data
        received_signature = notification_data.get('signature')
        if not received_signature:
            _logger.warning("received notification with missing signature")
            raise Forbidden()

        # Compare the received signature with the expected signature computed from the data
        expected_signature = tx_sudo.provider_id._payulatam_generate_sign(notification_data)
        if not consteq(received_signature, expected_signature):
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
-- disable payulatam payment provider
UPDATE payment_provider
   SET payulatam_merchant_id = NULL,
       payulatam_account_id = NULL,
       payulatam_api_key = NULL;

```

## File: data\payment_provider_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="payment_provider_payulatam" model="payment.provider">
        <field name="name">PayU Latam</field>
        <field name="display_as">Credit Card (powered by PayU Latam)</field>
        <field name="image_128"
               type="base64"
               file="payment_payulatam/static/description/icon.png"/>
        <field name="module_id" ref="base.module_payment_payulatam"/>
        <!-- https://www.payulatam.com/medios-de-pago/ -->
        <field name="payment_icon_ids"
               eval="[(6, 0, [
                   ref('payment.payment_icon_cc_diners_club_intl'),
                   ref('payment.payment_icon_cc_mastercard'),
                   ref('payment.payment_icon_cc_american_express'),
                   ref('payment.payment_icon_cc_visa'),
                   ref('payment.payment_icon_cc_codensa_easy_credit'),
               ])]"/>
        <field name="code">payulatam</field>
        <field name="redirect_form_view_id" ref="redirect_form"/>
    </record>

</odoo>

```

## File: models\payment_provider.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from hashlib import md5

from odoo import api, fields, models
from odoo.tools.float_utils import float_split, float_repr

SUPPORTED_CURRENCIES = ('ARS', 'BRL', 'CLP', 'COP', 'MXN', 'PEN', 'USD')


class PaymentProvider(models.Model):
    _inherit = 'payment.provider'

    code = fields.Selection(
        selection_add=[('payulatam', 'PayU Latam')], ondelete={'payulatam': 'set default'})
    payulatam_merchant_id = fields.Char(
        string="PayU Latam Merchant ID",
        help="The ID solely used to identify the account with PayULatam",
        required_if_provider='payulatam')
    payulatam_account_id = fields.Char(
        string="PayU Latam Account ID",
        help="The ID solely used to identify the country-dependent shop with PayULatam",
        required_if_provider='payulatam')
    payulatam_api_key = fields.Char(
        string="PayU Latam API Key", required_if_provider='payulatam',
        groups='base.group_system')

    @api.model
    def _get_compatible_providers(self, *args, currency_id=None, **kwargs):
        """ Override of payment to unlist PayU Latam providers for unsupported currencies. """
        providers = super()._get_compatible_providers(*args, currency_id=currency_id, **kwargs)

        currency = self.env['res.currency'].browse(currency_id).exists()
        if currency and currency.name not in SUPPORTED_CURRENCIES:
            providers = providers.filtered(lambda p: p.code != 'payulatam')

        return providers

    def _payulatam_generate_sign(self, values, incoming=True):
        """ Generate the signature for incoming or outgoing communications.

        :param dict values: The values used to generate the signature
        :param bool incoming: Whether the signature must be generated for an incoming (PayU Latam to
                              Odoo) or outgoing (Odoo to PayU Latam) communication.
        :return: The signature
        :rtype: str
        """
        if incoming:
            # "Confirmation" and "Response" pages have a different way to calculate what they call the `new_value`
            if self.env.context.get('payulatam_is_confirmation_page'):
                # https://developers.payulatam.com/latam/en/docs/integrations/webcheckout-integration/confirmation-page.html#signature-validation
                # For confirmation page, PayU Latam round to the first digit if the second one is a zero
                # to generate their signature.
                # e.g:
                #  150.00 -> 150.0
                #  150.26 -> 150.26
                # This happens to be Python 3's default behavior when casting to `float`.
                new_value = "%d.%d" % float_split(float(values.get('TX_VALUE')), 2)
            else:
                # https://developers.payulatam.com/latam/en/docs/integrations/webcheckout-integration/response-page.html#signature-validation
                # PayU Latam use the "Round half to even" rounding method
                # to generate their signature. This happens to be Python 3's
                # default rounding method.
                new_value = float_repr(float(values.get('TX_VALUE')), 1)
            data_string = '~'.join([
                self.payulatam_api_key,
                self.payulatam_merchant_id,
                values['referenceCode'],
                new_value,
                values['currency'],
                values.get('transactionState'),
            ])
        else:
            data_string = '~'.join([
                self.payulatam_api_key,
                self.payulatam_merchant_id,
                values['referenceCode'],
                float_repr(float(values['amount']), 2),
                values['currency'],
            ])
        return md5(data_string.encode('utf-8')).hexdigest()

```

## File: models\payment_transaction.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import hmac
import logging

from werkzeug import urls

from odoo import _, api, models
from odoo.exceptions import ValidationError
from odoo.tools.float_utils import float_repr

from odoo.addons.payment import utils as payment_utils
from odoo.addons.payment_payulatam.controllers.main import PayuLatamController

_logger = logging.getLogger(__name__)


class PaymentTransaction(models.Model):
    _inherit = 'payment.transaction'

    @api.model
    def _compute_reference(self, provider_code, prefix=None, separator='-', **kwargs):
        """ Override of payment to ensure that PayU Latam requirements for references are satisfied.

        PayU Latam requirements for transaction are as follows:
        - References must be unique at provider level for a given merchant account.
          This is satisfied by singularizing the prefix with the current datetime. If two
          transactions are created simultaneously, `_compute_reference` ensures the uniqueness of
          references by suffixing a sequence number.

        :param str provider_code: The code of the provider handling the transaction
        :param str prefix: The custom prefix used to compute the full reference
        :param str separator: The custom separator used to separate the prefix from the suffix
        :return: The unique reference for the transaction
        :rtype: str
        """
        if provider_code == 'payulatam':
            if not prefix:
                # If no prefix is provided, it could mean that a module has passed a kwarg intended
                # for the `_compute_reference_prefix` method, as it is only called if the prefix is
                # empty. We call it manually here because singularizing the prefix would generate a
                # default value if it was empty, hence preventing the method from ever being called
                # and the transaction from received a reference named after the related document.
                prefix = self.sudo()._compute_reference_prefix(
                    provider_code, separator, **kwargs
                ) or None
            prefix = payment_utils.singularize_reference_prefix(prefix=prefix, separator=separator)
        return super()._compute_reference(
            provider_code, prefix=prefix, separator=separator, **kwargs
        )

    def _get_specific_rendering_values(self, processing_values):
        """ Override of payment to return Payulatam-specific rendering values.

        Note: self.ensure_one() from `_get_processing_values`

        :param dict processing_values: The generic and specific processing values of the transaction
        :return: The dict of provider-specific processing values
        :rtype: dict
        """
        res = super()._get_specific_rendering_values(processing_values)
        if self.provider_code != 'payulatam':
            return res

        api_url = 'https://checkout.payulatam.com/ppp-web-gateway-payu/' \
            if self.provider_id.state == 'enabled' \
            else 'https://sandbox.checkout.payulatam.com/ppp-web-gateway-payu/'
        base_url = self.provider_id.get_base_url()
        payulatam_values = {
            'merchantId': self.provider_id.payulatam_merchant_id,
            'referenceCode': self.reference,
            'description': self.reference,
            'amount': float_repr(processing_values['amount'], self.currency_id.decimal_places or 2),
            'tax': 0,
            'taxReturnBase': 0,
            'currency': self.currency_id.name,
            'accountId': self.provider_id.payulatam_account_id,
            'buyerFullName': self.partner_name,
            'buyerEmail': self.partner_email,
            'responseUrl': urls.url_join(base_url, PayuLatamController._return_url),
            'confirmationUrl': urls.url_join(base_url, PayuLatamController._webhook_url),
            'api_url': api_url,
        }
        if self.provider_id.state != 'enabled':
            payulatam_values['test'] = 1
        payulatam_values['signature'] = self.provider_id._payulatam_generate_sign(
            payulatam_values, incoming=False
        )
        return payulatam_values

    def _get_tx_from_notification_data(self, provider_code, notification_data):
        """ Override of payment to find the transaction based on Payulatam data.

        :param str provider_code: The code of the provider that handled the transaction
        :param dict notification_data: The notification data sent by the provider
        :return: The transaction if found
        :rtype: recordset of `payment.transaction`
        :raise: ValidationError if the data match no transaction
        """
        tx = super()._get_tx_from_notification_data(provider_code, notification_data)
        if provider_code != 'payulatam' or len(tx) == 1:
            return tx

        reference = notification_data.get('referenceCode')
        tx = self.search([('reference', '=', reference), ('provider_code', '=', 'payulatam')])
        if not tx:
            raise ValidationError(
                "PayU Latam: " + _("No transaction found matching reference %s.", reference)
            )

        return tx

    def _process_notification_data(self, notification_data):
        """ Override of payment to process the transaction based on Payulatam data.

        Note: self.ensure_one()

        :param dict notification_data: The notification data sent by the provider
        :return: None
        """
        super()._process_notification_data(notification_data)
        if self.provider_code != 'payulatam':
            return

        self.provider_reference = notification_data.get('transactionId')

        status = notification_data.get('lapTransactionState')
        state_message = notification_data.get('message')
        if status == 'PENDING':
            self._set_pending(state_message=state_message)
        elif status == 'APPROVED':
            self._set_done(state_message=state_message)
        elif status in ('EXPIRED', 'DECLINED'):
            self._set_canceled(state_message=state_message)
        else:
            _logger.warning(
                "received data with invalid payment status (%s) for transaction with reference %s",
                status, self.reference
            )
            self._set_error("PayU Latam: " + _("Invalid payment status."))

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import payment_provider
from . import payment_transaction

```

## File: static\description\icon.svg

```svg
<svg id="Layer_1" data-name="Layer 1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 70 70">
  <defs>
    <mask id="mask" x="0" y="0" width="70" height="70" maskUnits="userSpaceOnUse">
      <g id="b">
        <path id="a" d="M4,0H65c4,0,5,1,5,5V65c0,4-1,5-5,5H4c-3,0-4-1-4-5V5C0,1,1,0,4,0Z" fill="#fff" fill-rule="evenodd"/>
      </g>
    </mask>
    <linearGradient id="linear-gradient" x1="-1438.5" y1="-4.39" x2="-1439.5" y2="-5.39" gradientTransform="matrix(70, 0, 0, -70, 100764.99, -307.17)" gradientUnits="userSpaceOnUse">
      <stop offset="0" stop-color="#7cc098"/>
      <stop offset="1" stop-color="#5f8a71"/>
    </linearGradient>
  </defs>
  <g mask="url(#mask)">
    <g>
      <path d="M0,0H70V70H0Z" fill-rule="evenodd" fill="url(#linear-gradient)"/>
      <path d="M4,1H65c2.67,0,4.33.67,5,2V0H0V3C.67,1.67,2,1,4,1Z" fill="#fff" fill-opacity="0.38" fill-rule="evenodd"/>
      <path d="M4,69a3.66,3.66,0,0,1-4-4V37.35L16.62,20.8l3.49,3.38L34.74,9.61l3.68,2v1.7L39.69,12l4.41,4.32L33.82,26.52l.24,2.67,5.53-5.54L50.39,25l1.36,12.54-3.84,3.86c.66,5.52.62,13.53.44,22.08L42.8,69Z" fill="#393939" fill-rule="evenodd" opacity="0.32" style="isolation: isolate"/>
      <path d="M4,69H65c2.67,0,4.33-1,5-3v4H0V66A3.92,3.92,0,0,0,4,69Z" fill-opacity="0.38" fill-rule="evenodd"/>
      <g>
        <g opacity="0.4">
          <rect x="32.43" y="10.58" width="4" height="4" rx="1"/>
          <path d="M47,39.5h2.71V27.44A2.73,2.73,0,0,0,47,24.7H37.5v2.74h9.2a.34.34,0,0,1,.34.34V31H36.5v6.5H47Z"/>
          <rect x="37.43" y="13.58" width="5" height="5" rx="1"/>
          <polygon points="17.31 57.39 17.3 57.39 16.5 55 15.43 55 15.43 58.47 16.14 58.47 16.14 56.03 16.15 56.03 16.99 58.47 17.58 58.47 18.43 56.01 18.44 56.01 18.44 58.47 19.15 58.47 19.15 55 18.08 55 17.31 57.39"/>
          <path d="M40.22,47.53l-19.91,0a.34.34,0,0,1-.34-.34V46H17.25v1.55A2.73,2.73,0,0,0,20,50.29H40.18Z"/>
          <path d="M23.1,43.48a9.9,9.9,0,0,0,6.72-2.12q2.43-2.11,2.43-6.46V23.58h3.18a1,1,0,0,0,1-1v-4a1,1,0,0,0-1-1h-4a1,1,0,0,0-1,1v3H27.54V34.9a5.34,5.34,0,0,1-1,3.52c-.64.75-1.8,1.13-3.48,1.13a5.39,5.39,0,0,1-2.29-.41,3.49,3.49,0,0,1-1.34-1,3.45,3.45,0,0,1-.63-1.48,9.71,9.71,0,0,1-.15-1.71V21.58H14V34.9q0,4.38,2.39,6.48A9.85,9.85,0,0,0,23.1,43.48Z"/>
          <polygon points="12.43 55.64 13.46 55.64 13.46 58.47 14.22 58.47 14.22 55.64 15.26 55.64 15.26 55 12.43 55 12.43 55.64"/>
          <g id="layer101">
            <path d="M57.44,48.81c-.38-.79-2.25-1.85-3.79-2.13l-.46-.08L53,46.16a2.22,2.22,0,0,0-1.86-1.47c-.4,0-.42,0-.86-.47-.25-.25-.57-.55-.71-.68l-.25-.24-.55.06a2.81,2.81,0,0,1-1.56-.14,3.66,3.66,0,0,0-2-.16,1.2,1.2,0,0,0-.56.63s-.13,0-.27,0a.81.81,0,0,0-.55,0c-.29.11-.3.1-.41-.07a1.57,1.57,0,0,1-.14-.7,6,6,0,0,0-.06-.71c0-.2-.42-.43-.7-.43a.62.62,0,0,1-.26,0,.81.81,0,0,1,.07-.36,1.39,1.39,0,0,0,.07-.55,1.51,1.51,0,0,0-.34-.52,2.14,2.14,0,0,0-.94-.59A7.46,7.46,0,0,0,39,41.3c0,.29,0,.52,0,.58h0v0a.56.56,0,0,1,0,.12.85.85,0,0,1,0,.17,4,4,0,0,1,.56.76.37.37,0,0,1,.12.07,3.6,3.6,0,0,0,.61.31,2.49,2.49,0,0,1,1.27.88,3.48,3.48,0,0,0,2.11,1.24c.19,0,.28,0,.26.09a7.76,7.76,0,0,1-.45.73,4.64,4.64,0,0,0-.47.89,5.73,5.73,0,0,0,0,.89,2,2,0,0,0,.35,1.24c1.1,2,1.3,2.33,2,2.9a7.7,7.7,0,0,1,.62.53,5.13,5.13,0,0,1,0,1.84c0,.25-.24,1.07-.44,1.82-.61,2.21-.73,3.17-.47,3.67s.19.41-.11.69A1,1,0,0,0,44.72,62a6.27,6.27,0,0,1,.21.91,4.67,4.67,0,0,0,.21.86l.06.16,0,.07a10.3,10.3,0,0,0,.63,1.62l.34,0a7.83,7.83,0,0,0,1.15-1.21s-.09,0-.09,0a2.38,2.38,0,0,1,.21-.38,1.33,1.33,0,0,0,.21-.52.42.42,0,0,1,.11-.28,2.58,2.58,0,0,0,.59-1.35,2,2,0,0,1,.52-1.16,2.77,2.77,0,0,0,.67-1.2c0-.1,0-.14.17-.14a1.92,1.92,0,0,0,.89-.33,1.44,1.44,0,0,0,.56-.82c0-.05.14-.19.31-.28A4.49,4.49,0,0,0,53,56a7.8,7.8,0,0,1,.77-1.29,1.29,1.29,0,0,1,.79-.3c.55-.1.6-.12.88-.41a4.86,4.86,0,0,0,.86-2.29,2.11,2.11,0,0,1,.73-1.47,1.75,1.75,0,0,0,.54-1A1.28,1.28,0,0,0,57.44,48.81Z"/>
          </g>
        </g>
        <g>
          <rect x="34.42" y="8.58" width="4" height="4" rx="1" fill="#fff"/>
          <path d="M49,37.5l2.71,0V25.45A2.73,2.73,0,0,0,49,22.7H39.5v2.75h9.2a.34.34,0,0,1,.34.34V29H38.5v6.5H49Z" fill="#fff"/>
          <rect x="39.42" y="11.58" width="5" height="5" rx="1" fill="#fff"/>
          <polygon points="19.31 55.39 19.3 55.39 18.49 53 17.42 53 17.42 56.47 18.13 56.47 18.13 54.04 18.14 54.04 18.99 56.47 19.58 56.47 20.42 54.01 20.43 54.01 20.43 56.47 21.14 56.47 21.14 53 20.07 53 19.31 55.39" fill="#fff"/>
          <path d="M42.22,45.53l-19.92,0a.34.34,0,0,1-.34-.34V44H19.25v1.55A2.73,2.73,0,0,0,22,48.3H42.18Z" fill="#fff"/>
          <path d="M25.1,41.48a9.85,9.85,0,0,0,6.72-2.12c1.62-1.4,2.43-3.56,2.43-6.46V21.58h3.17a1,1,0,0,0,1-1v-4a1,1,0,0,0-1-1h-4a1,1,0,0,0-1,1v3H29.54V32.9a5.34,5.34,0,0,1-1,3.52c-.64.75-1.8,1.13-3.48,1.13a5.65,5.65,0,0,1-2.3-.4,3.39,3.39,0,0,1-1.33-1.06,3.45,3.45,0,0,1-.63-1.48,9.71,9.71,0,0,1-.15-1.71V19.58H16V32.9c0,2.92.8,5.08,2.38,6.48A9.89,9.89,0,0,0,25.1,41.48Z" fill="#fff"/>
          <polygon points="14.42 53.64 15.46 53.64 15.46 56.47 16.22 56.47 16.22 53.64 17.26 53.64 17.26 53 14.42 53 14.42 53.64" fill="#fff"/>
          <g id="layer101-2" data-name="layer101">
            <path d="M59.44,46.82c-.39-.8-2.25-1.85-3.79-2.13l-.46-.09L55,44.16a2.25,2.25,0,0,0-1.86-1.47c-.4,0-.43,0-.86-.47l-.71-.68-.25-.24-.55.06a2.86,2.86,0,0,1-1.57-.13,3.44,3.44,0,0,0-2-.16,1.18,1.18,0,0,0-.56.62s-.14,0-.27,0a.76.76,0,0,0-.55,0c-.29.1-.3.1-.41-.07a1.63,1.63,0,0,1-.14-.71,5.09,5.09,0,0,0-.07-.7.79.79,0,0,0-.7-.44.56.56,0,0,1-.25,0s0-.18.06-.36a1.21,1.21,0,0,0,.07-.55,1.42,1.42,0,0,0-.33-.52,2.17,2.17,0,0,0-.95-.59A7.28,7.28,0,0,0,41,39.31c0,.28,0,.52,0,.57h0v0s0,.07,0,.11,0,.11,0,.17a4.74,4.74,0,0,1,.56.77.24.24,0,0,1,.11.06,3.82,3.82,0,0,0,.62.31,2.52,2.52,0,0,1,1.26.88,3.48,3.48,0,0,0,2.11,1.24c.2,0,.29,0,.27.1s-.22.37-.46.73a4.12,4.12,0,0,0-.47.89,6,6,0,0,0,0,.89,2,2,0,0,0,.35,1.24c1.09,2,1.3,2.32,2,2.89a6.78,6.78,0,0,1,.63.53,5.38,5.38,0,0,1,0,1.84c0,.25-.24,1.07-.45,1.82-.61,2.21-.73,3.17-.46,3.67s.18.41-.11.69A1,1,0,0,0,46.72,60a5.3,5.3,0,0,1,.2.91,4.53,4.53,0,0,0,.22.86l.06.17,0,.06a10.93,10.93,0,0,0,.62,1.62l.35,0a7.83,7.83,0,0,0,1.14-1.22l-.09,0a2,2,0,0,1,.22-.38,1.37,1.37,0,0,0,.21-.52.35.35,0,0,1,.11-.27,2.71,2.71,0,0,0,.59-1.36,1.87,1.87,0,0,1,.52-1.15,2.85,2.85,0,0,0,.67-1.21c0-.1,0-.14.17-.14a1.89,1.89,0,0,0,.89-.33,1.48,1.48,0,0,0,.55-.81c0-.06.14-.19.31-.29A4.37,4.37,0,0,0,55,54a8.29,8.29,0,0,1,.76-1.3,1.43,1.43,0,0,1,.8-.3c.55-.09.6-.12.87-.41a4.76,4.76,0,0,0,.87-2.29,2,2,0,0,1,.73-1.46,1.81,1.81,0,0,0,.54-1A1.65,1.65,0,0,0,59.44,46.82Z" fill="#fff"/>
          </g>
        </g>
      </g>
    </g>
  </g>
</svg>

```

## File: views\payment_payulatam_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="redirect_form">
        <form t-att-action="api_url" method="post">
            <input type="hidden" name="merchantId" t-att-value="merchantId"/>
            <input type="hidden" name="referenceCode" t-att-value="referenceCode"/>
            <input type="hidden" name="description" t-att-value="description"/>
            <input type="hidden" name="amount" t-att-value="amount"/>
            <!-- Use t-attf to set O, otherwise the value attribute is not included in the input -->
            <input type="hidden" name="tax" t-attf-value="{{tax}}"/>
            <!-- Use t-attf to set O, otherwise the value attribute is not included in the input -->
            <input type="hidden" name="taxReturnBase" t-attf-value="{{taxReturnBase}}"/>
            <input type="hidden" name="signature" t-att-value="signature"/>
            <input type="hidden" name="currency" t-att-value="currency"/>
            <input type="hidden" name="test" t-att-value="test"/>
            <input type="hidden" name="accountId" t-att-value="accountId"/>
            <input type="hidden" name="buyerFullName" t-att-value="buyerFullName"/>
            <input type="hidden" name="buyerEmail" t-att-value="buyerEmail"/>
            <input type="hidden" name="responseUrl" t-att-value="responseUrl"/>
            <input type="hidden" name="confirmationUrl" t-att-value="confirmationUrl"/>
        </form>
    </template>

</odoo>

```

## File: views\payment_provider_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_provider_form" model="ir.ui.view">
        <field name="name">PayU latam Provider Form</field>
        <field name="model">payment.provider</field>
        <field name="inherit_id" ref="payment.payment_provider_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='provider_creation_warning']" position="after">
                <div class="alert alert-danger"
                     role="alert"
                     attrs="{'invisible': [('code', '!=', 'payulatam')]}">
                    This provider is deprecated.
                    Consider disabling it and moving to <strong>Mercado Pago</strong>.
                </div>
            </xpath>
            <group name="provider_credentials" position="inside">
                <group attrs="{'invisible': [('code', '!=', 'payulatam')]}">
                    <field name="payulatam_merchant_id"
                           attrs="{'required':[('code', '=', 'payulatam'), ('state', '!=', 'disabled')]}"/>
                    <field name="payulatam_account_id"
                           attrs="{'required':[('code', '=', 'payulatam'), ('state', '!=', 'disabled')]}"/>
                    <field name="payulatam_api_key"
                           attrs="{'required':[('code', '=', 'payulatam'), ('state', '!=', 'disabled')]}"
                           password="True"/>
                </group>
            </group>
        </field>
    </record>

</odoo>

```

