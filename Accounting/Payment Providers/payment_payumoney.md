# Odoo Module: payment_payumoney

Category: Accounting/Payment Providers

This file contains the source code of the Odoo module.

## File: const.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# The codes of the payment methods to activate when PayU Money is activated.
DEFAULT_PAYMENT_METHODS_CODES = [
    # Primary payment methods.
    'card',
    # Brand payment methods.
    'visa',
    'mastercard',
    'amex',
    'discover',
]

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
            "Consider installing the Payment Provider: Razorpay module instead.")


def post_init_hook(env):
    setup_provider(env, 'payumoney')


def uninstall_hook(env):
    reset_payment_provider(env, 'payumoney')

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Payment Provider: PayUmoney',
    'version': '2.0',
    'category': 'Accounting/Payment Providers',
    'sequence': 350,
    'summary': "This module is deprecated.",
    'description': " ",  # Non-empty string to avoid loading the README file.
    'depends': ['payment'],
    'data': [
        'views/payment_payumoney_templates.xml',
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

import hmac
import logging
import pprint

from werkzeug.exceptions import Forbidden

from odoo import http
from odoo.http import request

_logger = logging.getLogger(__name__)


class PayUMoneyController(http.Controller):
    _return_url = '/payment/payumoney/return'

    @http.route(
        _return_url, type='http', auth='public', methods=['GET', 'POST'], csrf=False,
        save_session=False
    )
    def payumoney_return_from_checkout(self, **data):
        """ Process the notification data sent by PayUmoney after redirection from checkout.

        See https://developer.payumoney.com/redirect/.

        The route is flagged with `save_session=False` to prevent Odoo from assigning a new session
        to the user if they are redirected to this route with a POST request. Indeed, as the session
        cookie is created without a `SameSite` attribute, some browsers that don't implement the
        recommended default `SameSite=Lax` behavior will not include the cookie in the redirection
        request from the payment provider to Odoo. As the redirection to the '/payment/status' page
        will satisfy any specification of the `SameSite` attribute, the session of the user will be
        retrieved and with it the transaction which will be immediately post-processed.

        :param dict data: The notification data
        """
        _logger.info("handling redirection from PayU money with data:\n%s", pprint.pformat(data))

        # Check the integrity of the notification
        tx_sudo = request.env['payment.transaction'].sudo()._get_tx_from_notification_data(
            'payumoney', data
        )
        self._verify_notification_signature(data, tx_sudo)

        # Handle the notification data
        tx_sudo._handle_notification_data('payumoney', data)
        return request.redirect('/payment/status')

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
        received_signature = notification_data.get('hash')
        if not received_signature:
            _logger.warning("received notification with missing signature")
            raise Forbidden()

        # Compare the received signature with the expected signature computed from the data
        expected_signature = tx_sudo.provider_id._payumoney_generate_sign(
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
-- disable payumoney payment provider
UPDATE payment_provider
   SET payumoney_merchant_key = NULL,
       payumoney_merchant_salt = NULL;

```

## File: data\payment_provider_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="payment_provider_payumoney" model="payment.provider">
        <field name="name">PayUmoney</field>
        <field name="image_128"
               type="base64"
               file="payment_payumoney/static/description/icon.png"/>
        <field name="module_id" ref="base.module_payment_payumoney"/>
        <!-- See https://www.payumoney.com/selfcare.html?userType=seller
             > Banks & Cards > What options do you have in the Credit Card payment? -->
        <field name="payment_method_ids"
               eval="[(6, 0, [
                   ref('payment.payment_method_card'),
                   ref('payment.payment_method_netbanking'),
                   ref('payment.payment_method_emi'),
                   ref('payment.payment_method_upi'),
               ])]"/>
        <field name="code">payumoney</field>
        <field name="redirect_form_view_id" ref="redirect_form"/>
    </record>

</odoo>

```

## File: models\payment_provider.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import hashlib

from odoo import fields, models

from odoo.addons.payment_payulatam.const import DEFAULT_PAYMENT_METHODS_CODES


class PaymentProvider(models.Model):
    _inherit = 'payment.provider'

    code = fields.Selection(
        selection_add=[('payumoney', "PayUmoney")], ondelete={'payumoney': 'set default'})
    payumoney_merchant_key = fields.Char(
        string="Merchant Key", help="The key solely used to identify the account with PayU money",
        required_if_provider='payumoney')
    payumoney_merchant_salt = fields.Char(
        string="Merchant Salt", required_if_provider='payumoney', groups='base.group_system')

    def _get_supported_currencies(self):
        """ Override of `payment` to return INR as the only supported currency. """
        supported_currencies = super()._get_supported_currencies()
        if self.code == 'payumoney':
            supported_currencies = supported_currencies.filtered(lambda c: c.name == 'INR')
        return supported_currencies

    def _payumoney_generate_sign(self, values, incoming=True):
        """ Generate the shasign for incoming or outgoing communications.

        :param dict values: The values used to generate the signature
        :param bool incoming: Whether the signature must be generated for an incoming (PayUmoney to
                              Odoo) or outgoing (Odoo to PayUMoney) communication.
        :return: The shasign
        :rtype: str
        """
        sign_values = {
            **values,
            'key': self.payumoney_merchant_key,
            'salt': self.payumoney_merchant_salt,
        }
        if incoming:
            keys = 'salt|status||||||udf5|udf4|udf3|udf2|udf1|email|firstname|productinfo|amount|' \
                   'txnid|key'
            sign = '|'.join(f'{sign_values.get(k) or ""}' for k in keys.split('|'))
        else:  # outgoing
            keys = 'key|txnid|amount|productinfo|firstname|email|udf1|udf2|udf3|udf4|udf5||||||salt'
            sign = '|'.join(f'{sign_values.get(k) or ""}' for k in keys.split('|'))
        return hashlib.sha512(sign.encode('utf-8')).hexdigest()

    #=== BUSINESS METHODS ===#

    def _get_default_payment_method_codes(self):
        """ Override of `payment` to return the default payment method codes. """
        default_codes = super()._get_default_payment_method_codes()
        if self.code != 'payumoney':
            return default_codes
        return DEFAULT_PAYMENT_METHODS_CODES

```

## File: models\payment_transaction.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from werkzeug import urls

from odoo import _, api, models
from odoo.exceptions import ValidationError

from odoo.addons.payment import utils as payment_utils
from odoo.addons.payment_payumoney.controllers.main import PayUMoneyController


class PaymentTransaction(models.Model):
    _inherit = 'payment.transaction'

    def _get_specific_rendering_values(self, processing_values):
        """ Override of payment to return Payumoney-specific rendering values.

        Note: self.ensure_one() from `_get_processing_values`

        :param dict processing_values: The generic and specific processing values of the transaction
        :return: The dict of provider-specific processing values
        :rtype: dict
        """
        res = super()._get_specific_rendering_values(processing_values)
        if self.provider_code != 'payumoney':
            return res

        first_name, last_name = payment_utils.split_partner_name(self.partner_id.name)
        api_url = 'https://secure.payu.in/_payment' if self.provider_id.state == 'enabled' \
            else 'https://sandboxsecure.payu.in/_payment'
        payumoney_values = {
            'key': self.provider_id.payumoney_merchant_key,
            'txnid': self.reference,
            'amount': self.amount,
            'productinfo': self.reference,
            'firstname': first_name,
            'lastname': last_name,
            'email': self.partner_email,
            'phone': self.partner_phone,
            'return_url': urls.url_join(self.get_base_url(), PayUMoneyController._return_url),
            'api_url': api_url,
        }
        payumoney_values['hash'] = self.provider_id._payumoney_generate_sign(
            payumoney_values, incoming=False,
        )
        return payumoney_values

    def _get_tx_from_notification_data(self, provider_code, notification_data):
        """ Override of payment to find the transaction based on Payumoney data.

        :param str provider_code: The code of the provider that handled the transaction
        :param dict notification_data: The notification data sent by the provider
        :return: The transaction if found
        :rtype: recordset of `payment.transaction`
        :raise: ValidationError if inconsistent data were received
        :raise: ValidationError if the data match no transaction
        """
        tx = super()._get_tx_from_notification_data(provider_code, notification_data)
        if provider_code != 'payumoney' or len(tx) == 1:
            return tx

        reference = notification_data.get('txnid')
        if not reference:
            raise ValidationError(
                "PayUmoney: " + _("Received data with missing reference (%s)", reference)
            )

        tx = self.search([('reference', '=', reference), ('provider_code', '=', 'payumoney')])
        if not tx:
            raise ValidationError(
                "PayUmoney: " + _("No transaction found matching reference %s.", reference)
            )

        return tx

    def _process_notification_data(self, notification_data):
        """ Override of payment to process the transaction based on Payumoney data.

        Note: self.ensure_one()

        :param dict notification_data: The notification data sent by the provider
        :return: None
        """
        super()._process_notification_data(notification_data)
        if self.provider_code != 'payumoney':
            return

        # Update the provider reference.
        self.provider_reference = notification_data.get('payuMoneyId')

        # Update the payment method
        payment_method_type = notification_data.get('bankcode', '')
        payment_method = self.env['payment.method']._get_from_code(payment_method_type)
        self.payment_method_id = payment_method or self.payment_method_id

        # Update the payment state.
        status = notification_data.get('status')
        if status == 'success':
            self._set_done()
        else:  # 'failure'
            # See https://www.payumoney.com/pdf/PayUMoney-Technical-Integration-Document.pdf
            error_code = notification_data.get('Error')
            self._set_error(
                "PayUmoney: " + _("The payment encountered an error with code %s", error_code)
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
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="m46.325 18.237-2.978-.001a.59.59 0 0 0-.589.59v.416h.207c1.345 0 1.845.222 1.845 1.45v1.744h1.513a.589.589 0 0 0 .589-.588v-3.021a.589.589 0 0 0-.587-.59Zm-16.253 4.69c-.138-.172-.398-.196-.658-.196h-.196c-.649 0-.903.2-1.047.825l-1.804 7.513c-.225.923-.541 1.092-1.083 1.092-.662 0-.928-.158-1.192-1.096l-2.043-7.513c-.17-.63-.419-.821-1.068-.821h-.174c-.262 0-.523.024-.657.2-.134.174-.089.436-.02.694l2.065 7.578c.387 1.45.848 2.651 2.568 2.651.32 0 .618-.045.865-.128-.522 1.644-1.053 2.37-2.618 2.53-.317.027-.524.072-.639.227-.12.16-.092.39-.05.595l.044.194c.093.45.252.728.756.728.053 0 .11-.003.17-.008 2.337-.153 3.59-1.414 4.322-4.352l2.5-10.02c.06-.258.096-.52-.041-.692ZM17.496 28.7v1.517c0 1.236-.458 1.952-2.796 1.952-1.545 0-2.296-.56-2.296-1.713 0-1.263.754-1.756 2.687-1.756h2.405ZM14.7 22.412c-1.275 0-2.074.16-2.377.22-.536.118-.76.265-.76.877v.174c0 .24.035.405.11.522.089.136.232.205.424.205.094 0 .203-.016.333-.048.306-.077 1.286-.236 2.357-.236 1.924 0 2.709.534 2.709 1.844v1.168h-2.427c-3.119 0-4.572 1.054-4.572 3.318 0 2.196 1.5 3.406 4.225 3.406 3.237 0 4.68-1.104 4.68-3.58V25.97c0-2.394-1.538-3.558-4.702-3.558Zm-6.115 1.452c0 1.803-.46 2.78-2.883 2.78h-3.73v-4.653c0-.645.24-.885.883-.885h2.847c1.826 0 2.883.452 2.883 2.758ZM5.702 19.24H2.486C.766 19.24 0 20.007 0 21.73v11.065c0 .665.213.879.877.879h.218c.664 0 .877-.214.877-.88V28.49h3.73c3.312 0 4.855-1.47 4.855-4.625 0-3.155-1.543-4.625-4.855-4.625Zm40.913-4.12h-1.502a.297.297 0 0 1-.297-.298v-1.524c0-.164.134-.297.297-.297h1.502c.164 0 .297.134.297.298v1.524a.297.297 0 0 1-.297.297Zm2.947 3.12H47.35a.437.437 0 0 1-.436-.439v-2.244c0-.241.196-.437.437-.437h2.212c.241 0 .437.197.437.438v2.244a.437.437 0 0 1-.438.438Zm-6.217 4.197a.588.588 0 0 1-.587-.59v-2.604h-.217c-1.344 0-1.844.222-1.844 1.45v2.872l-.001.018v.63l-.002.064v4.013c0 .49-.094.88-.289 1.184-.366.566-1.092.823-2.254.825-1.16-.002-1.886-.259-2.253-.825-.195-.303-.29-.693-.29-1.184v-4.708l-.002-.017v-2.872c0-1.228-.5-1.45-1.845-1.45h-.423c-1.345 0-1.845.222-1.845 1.45v7.597c0 1.222.275 2.257.807 3.091 1.026 1.617 3.014 2.478 5.841 2.478h.02c2.828 0 4.816-.861 5.842-2.478.532-.834.807-1.869.807-3.09v-5.854h-1.465Z" fill="#62CAC3"/><path d="M43.105 11h.972V7.818h1.108V7H42v.818h1.105V11Zm2.775 0h.858V8.453h.053L47.663 11h.555l.871-2.547h.056V11H50V7h-1.108l-.924 2.714h-.05L46.99 7h-1.11v4Z" fill="#D1D5DB"/></svg>

```

## File: views\payment_payumoney_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="redirect_form">
        <form t-att-action="api_url" method="post">
            <input type="hidden" name="key" t-att-value="key"/>
            <input type="hidden" name="txnid" t-att-value="txnid"/>
            <input type="hidden" name="amount" t-att-value="amount"/>
            <input type="hidden" name="productinfo" t-att-value="productinfo"/>
            <input type="hidden" name="firstname" t-att-value="firstname"/>
            <input type="hidden" name="lastname" t-att-value="lastname"/>
            <input type="hidden" name="email" t-att-value="email"/>
            <input type="hidden" name="phone" t-att-value="phone"/>
            <input type="hidden" name="surl" t-att-value="return_url"/>
            <input type="hidden" name="furl" t-att-value="return_url"/>
            <input type="hidden" name="service_provider" value="payu_paisa"/>
            <input type="hidden" name="hash" t-att-value="hash"/>
        </form>
    </template>

</odoo>

```

## File: views\payment_provider_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_provider_form" model="ir.ui.view">
        <field name="name">PayUMoney Provider Form</field>
        <field name="model">payment.provider</field>
        <field name="inherit_id" ref="payment.payment_provider_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='provider_creation_warning']" position="after">
                <div class="alert alert-danger"
                     role="alert"
                     invisible="code != 'payumoney'">
                    This provider is deprecated.
                    Consider disabling it and moving to <strong>Razorpay</strong>.
                </div>
            </xpath>
            <group name="provider_credentials" position="inside">
                <group invisible="code != 'payumoney'">
                    <field name="payumoney_merchant_key" required="code == 'payumoney' and state != 'disabled'"/>
                    <field name="payumoney_merchant_salt" required="code == 'payumoney' and state != 'disabled'" password="True"/>
                </group>
            </group>
        </field>
    </record>

</odoo>

```

