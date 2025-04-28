# Odoo Module: payment_payumoney

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
            "Consider installing the Payment Provider: Razorpay module instead.")


def post_init_hook(cr, registry):
    setup_provider(cr, registry, 'payumoney')


def uninstall_hook(cr, registry):
    reset_payment_provider(cr, registry, 'payumoney')

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
        <field name="display_as">Credit Card (powered by PayUmoney)</field>
        <field name="image_128"
               type="base64"
               file="payment_payumoney/static/description/icon.png"/>
        <field name="module_id" ref="base.module_payment_payumoney"/>
        <!-- See https://www.payumoney.com/selfcare.html?userType=seller
             > Banks & Cards > What options do you have in the Credit Card payment? -->
        <field name="payment_icon_ids"
               eval="[(6, 0, [
                   ref('payment.payment_icon_cc_maestro'),
                   ref('payment.payment_icon_cc_mastercard'),
                   ref('payment.payment_icon_cc_american_express'),
                   ref('payment.payment_icon_cc_visa'),
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

from odoo import api, fields, models


class PaymentProvider(models.Model):
    _inherit = 'payment.provider'

    code = fields.Selection(
        selection_add=[('payumoney', "PayUmoney")], ondelete={'payumoney': 'set default'})
    payumoney_merchant_key = fields.Char(
        string="Merchant Key", help="The key solely used to identify the account with PayU money",
        required_if_provider='payumoney')
    payumoney_merchant_salt = fields.Char(
        string="Merchant Salt", required_if_provider='payumoney', groups='base.group_system')

    @api.model
    def _get_compatible_providers(self, *args, currency_id=None, **kwargs):
        """ Override of payment to unlist PayUmoney providers when the currency is not INR. """
        providers = super()._get_compatible_providers(*args, currency_id=currency_id, **kwargs)

        currency = self.env['res.currency'].browse(currency_id).exists()
        if currency and currency.name != 'INR':
            providers = providers.filtered(lambda p: p.code != 'payumoney')

        return providers

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

        status = notification_data.get('status')
        self.provider_reference = notification_data.get('payuMoneyId')

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
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#7CC098"/><stop offset="100%" stop-color="#5F8A71"/></linearGradient><path id="d" d="M32.423 21.58v-3a1 1 0 0 1 1-1h4a1 1 0 0 1 1 1v4a1 1 0 0 1-1 1H34.25V34.9c0 2.9-.81 5.055-2.43 6.465-1.62 1.41-3.86 2.115-6.72 2.115-2.9 0-5.145-.7-6.735-2.1-1.59-1.4-2.385-3.56-2.385-6.48V21.58h4.71V34.9c0 .58.05 1.15.15 1.71.1.56.31 1.055.63 1.485.32.43.765.78 1.335 1.05s1.335.405 2.295.405c1.68 0 2.84-.375 3.48-1.125.64-.75.96-1.925.96-3.525V21.58h2.883zM19.25 46h2.714v1.212c0 .188.152.343.339.343h26.394a.342.342 0 0 0 .339-.343V37.5H38.5V31h10.536v-3.212a.342.342 0 0 0-.339-.343H39.5v-2.742h9.536c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742V46zm-3.79 9.642h-1.037V55h2.833v.642h-1.037v2.832h-.76v-2.832zM17.422 55h1.07l.809 2.389h.01L20.075 55h1.07v3.474h-.711v-2.462h-.01l-.847 2.462h-.586l-.848-2.438h-.01v2.438h-.711V55zm23-41.42h3a1 1 0 0 1 1 1v3a1 1 0 0 1-1 1h-3a1 1 0 0 1-1-1v-3a1 1 0 0 1 1-1zm-5-3h2a1 1 0 0 1 1 1v2a1 1 0 0 1-1 1h-2a1 1 0 0 1-1-1v-2a1 1 0 0 1 1-1z"/><path id="e" d="M32.423 19.58v-3a1 1 0 0 1 1-1h4a1 1 0 0 1 1 1v4a1 1 0 0 1-1 1H34.25V32.9c0 2.9-.81 5.055-2.43 6.465-1.62 1.41-3.86 2.115-6.72 2.115-2.9 0-5.145-.7-6.735-2.1-1.59-1.4-2.385-3.56-2.385-6.48V19.58h4.71V32.9c0 .58.05 1.15.15 1.71.1.56.31 1.055.63 1.485.32.43.765.78 1.335 1.05s1.335.405 2.295.405c1.68 0 2.84-.375 3.48-1.125.64-.75.96-1.925.96-3.525V19.58h2.883zM19.25 44h2.714v1.212c0 .188.152.343.339.343h26.394a.342.342 0 0 0 .339-.343V35.5H38.5V29h10.536v-3.212a.342.342 0 0 0-.339-.343H39.5v-2.742h9.536c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742V44zm-3.79 9.642h-1.037V53h2.833v.642h-1.037v2.832h-.76v-2.832zM17.422 53h1.07l.809 2.389h.01L20.075 53h1.07v3.474h-.711v-2.462h-.01l-.847 2.462h-.586l-.848-2.438h-.01v2.438h-.711V53zm23-41.42h3a1 1 0 0 1 1 1v3a1 1 0 0 1-1 1h-3a1 1 0 0 1-1-1v-3a1 1 0 0 1 1-1zm-5-3h2a1 1 0 0 1 1 1v2a1 1 0 0 1-1 1h-2a1 1 0 0 1-1-1v-2a1 1 0 0 1 1-1z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M4 69c-2 0-4-1-4-4V33.916L16 19.58l3.25.42L35 8.58 38 12l-.89 2 2.584-2 4.729 4-5.923 6.703L48 23l3.576 1.968-.236 22.01L39.224 69H4z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
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
                     attrs="{'invisible': [('code', '!=', 'payumoney')]}">
                    This provider is deprecated.
                    Consider disabling it and moving to <strong>Razorpay</strong>.
                </div>
            </xpath>
            <group name="provider_credentials" position="inside">
                <group attrs="{'invisible': [('code', '!=', 'payumoney')]}">
                    <field name="payumoney_merchant_key" attrs="{'required':[ ('code', '=', 'payumoney'), ('state', '!=', 'disabled')]}"/>
                    <field name="payumoney_merchant_salt" attrs="{'required':[ ('code', '=', 'payumoney'), ('state', '!=', 'disabled')]}" password="True"/>
                </group>
            </group>
        </field>
    </record>

</odoo>

```

