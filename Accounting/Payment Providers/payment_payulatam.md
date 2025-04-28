# Odoo Module: payment_payulatam

Category: Accounting/Payment Providers

This file contains the source code of the Odoo module.

## File: const.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Supported currencies of PayuLatam, in ISO 4217 currency codes.
# https://developers.payulatam.com/latam/en/docs/getting-started/response-codes-and-variables.html#accepted-currencies.
# Last seen online: 22 September 2022.
SUPPORTED_CURRENCIES = [
    'ARS',
    'BRL',
    'CLP',
    'COP',
    'MXN',
    'PEN',
    'USD'
]

# The codes of the payment methods to activate when PayULatam is activated.
DEFAULT_PAYMENT_METHODS_CODES = [
    # Primary payment methods.
    'card',
    # Brand payment methods.
    'visa',
    'mastercard',
    'amex',
    'discover',
]

# Mapping of payment method codes to PayU Latam codes.
PAYMENT_METHODS_MAPPING = {
    'bank_reference': 'BANK_REFERENCED',
    'pix': 'PIX',
    'card': 'VISA,VISA_DEBIT,MASTERCARD,MASTERCARD_DEBIT,AMEX,ARGENCARD,CABAL,CENCOSUD,DINERS,ELO,NARANJA,SHOPPING,HIPERCARD,TRANSBANK_DEBIT,CODENSA',
    'bank_transfer': 'ITAU,PSE,SPEI',
}

```

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

from odoo.exceptions import UserError
from odoo.tools import config

from odoo.addons.payment import setup_provider, reset_payment_provider


def pre_init_hook(env):
    if not any(config.get(key) for key in ('init', 'update')):
        raise UserError(
            "This module is deprecated and cannot be installed. "
            "Consider installing the Payment Provider: Mercado Pago module instead.")


def post_init_hook(env):
    setup_provider(env, 'payulatam')


def uninstall_hook(env):
    reset_payment_provider(env, 'payulatam')

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
        <field name="image_128"
               type="base64"
               file="payment_payulatam/static/description/icon.png"/>
        <field name="module_id" ref="base.module_payment_payulatam"/>
        <!-- https://www.payulatam.com/medios-de-pago/ -->
        <field name="payment_method_ids"
               eval="[(6, 0, [
                   ref('payment.payment_method_card'),
                   ref('payment.payment_method_pix'),
                   ref('payment.payment_method_bank_reference'),
                   ref('payment.payment_method_bank_transfer'),
                   ref('payment.payment_method_pse'),
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

from odoo import fields, models
from odoo.tools.float_utils import float_repr, float_split

from odoo.addons.payment_payulatam import const

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

    def _get_supported_currencies(self):
        """ Override of `payment` to return the supported currencies. """
        supported_currencies = super()._get_supported_currencies()
        if self.code == 'payulatam':
            supported_currencies = supported_currencies.filtered(
                lambda c: c.name in const.SUPPORTED_CURRENCIES
            )
        return supported_currencies

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
                values['paymentMethods'],
            ])
        return md5(data_string.encode('utf-8')).hexdigest()

    #=== BUSINESS METHODS ===#

    def _get_default_payment_method_codes(self):
        """ Override of `payment` to return the default payment method codes. """
        default_codes = super()._get_default_payment_method_codes()
        if self.code != 'payulatam':
            return default_codes
        return const.DEFAULT_PAYMENT_METHODS_CODES

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
from odoo.addons.payment_payulatam import const
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
            'paymentMethods': const.PAYMENT_METHODS_MAPPING.get(
                self.payment_method_code, self.payment_method_code
            ),
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

        # Update the provider reference.
        self.provider_reference = notification_data.get('transactionId')

        # Update the payment method.
        payment_method_type = notification_data.get('lapPaymentMethod', '').lower()
        payment_method = self.env['payment.method']._get_from_code(payment_method_type)
        self.payment_method_id = payment_method or self.payment_method_id

        # Update the payment state.
        status = notification_data.get('lapTransactionState')
        state_message = notification_data.get('message')
        if status == 'PENDING':
            self._set_pending()
        elif status == 'APPROVED':
            self._set_done(extra_allowed_states=('cancel',))
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
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M43.105 4h.972V.818h1.108V0H42v.818h1.105V4Zm2.775 0h.858V1.453h.053L47.663 4h.555l.871-2.547h.056V4H50V0h-1.108l-.924 2.714h-.05L46.99 0h-1.11v4Z" fill="#D1D5DB"/><path d="m46.325 18.237-2.978-.001a.59.59 0 0 0-.589.59v.416h.207c1.345 0 1.845.222 1.845 1.45v1.744h1.513a.589.589 0 0 0 .589-.588v-3.021a.589.589 0 0 0-.587-.59Zm-16.253 4.69c-.138-.172-.398-.196-.658-.196h-.196c-.649 0-.903.2-1.047.825l-1.804 7.513c-.225.923-.541 1.092-1.083 1.092-.662 0-.928-.158-1.192-1.096l-2.043-7.513c-.17-.63-.419-.821-1.068-.821h-.174c-.262 0-.523.024-.657.2-.134.174-.089.436-.02.694l2.065 7.578c.387 1.45.848 2.651 2.568 2.651.32 0 .618-.045.865-.128-.522 1.644-1.053 2.37-2.618 2.53-.317.027-.524.072-.639.227-.12.16-.092.39-.05.595l.044.194c.093.45.252.728.756.728.053 0 .11-.003.17-.008 2.337-.153 3.59-1.414 4.322-4.352l2.5-10.02c.06-.258.096-.52-.041-.692ZM17.496 28.7v1.517c0 1.236-.458 1.952-2.796 1.952-1.545 0-2.296-.56-2.296-1.713 0-1.263.754-1.756 2.687-1.756h2.405ZM14.7 22.412c-1.275 0-2.074.16-2.377.22-.536.118-.76.265-.76.877v.174c0 .24.035.405.11.522.089.136.232.205.424.205.094 0 .203-.016.333-.048.306-.077 1.286-.236 2.357-.236 1.924 0 2.709.534 2.709 1.844v1.168h-2.427c-3.119 0-4.572 1.054-4.572 3.318 0 2.196 1.5 3.406 4.225 3.406 3.237 0 4.68-1.104 4.68-3.58V25.97c0-2.394-1.538-3.558-4.702-3.558Zm-6.115 1.452c0 1.803-.46 2.78-2.883 2.78h-3.73v-4.653c0-.645.24-.885.883-.885h2.847c1.826 0 2.883.452 2.883 2.758ZM5.702 19.24H2.486C.766 19.24 0 20.007 0 21.73v11.065c0 .665.213.879.877.879h.218c.664 0 .877-.214.877-.88V28.49h3.73c3.312 0 4.855-1.47 4.855-4.625 0-3.155-1.543-4.625-4.855-4.625Zm40.913-4.12h-1.502a.297.297 0 0 1-.297-.298v-1.524c0-.164.134-.297.297-.297h1.502c.164 0 .297.134.297.298v1.524a.297.297 0 0 1-.297.297Zm2.947 3.12H47.35a.437.437 0 0 1-.436-.439v-2.244c0-.241.196-.437.437-.437h2.212c.241 0 .437.197.437.438v2.244a.437.437 0 0 1-.438.438Zm-6.217 4.197a.588.588 0 0 1-.587-.59v-2.604h-.217c-1.344 0-1.844.222-1.844 1.45v2.872l-.001.018v.63l-.002.064v4.013c0 .49-.094.88-.289 1.184-.366.566-1.092.823-2.254.825-1.16-.002-1.886-.259-2.253-.825-.195-.303-.29-.693-.29-1.184v-4.708l-.002-.017v-2.872c0-1.228-.5-1.45-1.845-1.45h-.423c-1.345 0-1.845.222-1.845 1.45v7.597c0 1.222.275 2.257.807 3.091 1.026 1.617 3.014 2.478 5.841 2.478h.02c2.828 0 4.816-.861 5.842-2.478.532-.834.807-1.869.807-3.09v-5.854h-1.465Z" fill="#A6C307"/></svg>

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
            <input type="hidden" name="paymentMethods" t-att-value="paymentMethods"/>
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
                     invisible="code != 'payulatam'">
                    This provider is deprecated.
                    Consider disabling it and moving to <strong>Mercado Pago</strong>.
                </div>
            </xpath>
            <group name="provider_credentials" position="inside">
                <group invisible="code != 'payulatam'">
                    <field name="payulatam_merchant_id"
                           required="code == 'payulatam' and state != 'disabled'"/>
                    <field name="payulatam_account_id"
                           required="code == 'payulatam' and state != 'disabled'"/>
                    <field name="payulatam_api_key"
                           required="code == 'payulatam' and state != 'disabled'"
                           password="True"/>
                </group>
            </group>
        </field>
    </record>

</odoo>

```

