# Odoo Module: payment_payumoney

Category: Accounting/Payment Acquirers

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

from odoo.addons.payment import reset_payment_acquirer


def uninstall_hook(cr, registry):
    reset_payment_acquirer(cr, registry, 'payumoney')

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'PayuMoney Payment Acquirer',
    'version': '2.0',
    'category': 'Accounting/Payment Acquirers',
    'sequence': 375,
    'summary': 'Payment Acquirer: PayuMoney Implementation',
    'description': " ",  # Non-empty string to avoid loading the README file.
    'depends': ['payment'],
    'data': [
        'views/payment_views.xml',
        'views/payment_payumoney_templates.xml',
        'data/payment_acquirer_data.xml',
    ],
    'application': True,
    'uninstall_hook': 'uninstall_hook',
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import pprint

from odoo import http
from odoo.http import request

_logger = logging.getLogger(__name__)


class PayUMoneyController(http.Controller):
    _return_url = '/payment/payumoney/return'

    @http.route(
        _return_url, type='http', auth='public', methods=['GET', 'POST'], csrf=False,
        save_session=False
    )
    def payumoney_return(self, **data):
        """ Process the data returned by PayUmoney after redirection.

        The route is flagged with `save_session=False` to prevent Odoo from assigning a new session
        to the user if they are redirected to this route with a POST request. Indeed, as the session
        cookie is created without a `SameSite` attribute, some browsers that don't implement the
        recommended default `SameSite=Lax` behavior will not include the cookie in the redirection
        request from the payment provider to Odoo. As the redirection to the '/payment/status' page
        will satisfy any specification of the `SameSite` attribute, the session of the user will be
        retrieved and with it the transaction which will be immediately post-processed.

        :param dict data: The feedback data to process
        """
        _logger.info("entering handle_feedback_data with data:\n%s", pprint.pformat(data))
        request.env['payment.transaction'].sudo()._handle_feedback_data('payumoney', data)
        return request.redirect('/payment/status')

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\payment_acquirer_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="payment.payment_acquirer_payumoney" model="payment.acquirer">
        <field name="provider">payumoney</field>
        <field name="redirect_form_view_id" ref="redirect_form"/>
        <field name="support_authorization">False</field>
        <field name="support_fees_computation">False</field>
        <field name="support_refund"></field>
        <field name="support_tokenization">False</field>
    </record>

    <record id="payment_method_payumoney" model="account.payment.method">
        <field name="name">PayUmoney</field>
        <field name="code">payumoney</field>
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
        res['payumoney'] = {'mode': 'electronic', 'domain': [('type', '=', 'bank')]}
        return res

```

## File: models\payment_acquirer.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import hashlib

from odoo import api, fields, models


class PaymentAcquirer(models.Model):
    _inherit = 'payment.acquirer'

    provider = fields.Selection(
        selection_add=[('payumoney', "PayUmoney")], ondelete={'payumoney': 'set default'})
    payumoney_merchant_key = fields.Char(
        string="Merchant Key", help="The key solely used to identify the account with PayU money",
        required_if_provider='payumoney')
    payumoney_merchant_salt = fields.Char(
        string="Merchant Salt", required_if_provider='payumoney', groups='base.group_system')

    @api.model
    def _get_compatible_acquirers(self, *args, currency_id=None, **kwargs):
        """ Override of payment to unlist PayUmoney acquirers when the currency is not INR. """
        acquirers = super()._get_compatible_acquirers(*args, currency_id=currency_id, **kwargs)

        currency = self.env['res.currency'].browse(currency_id).exists()
        if currency and currency.name != 'INR':
            acquirers = acquirers.filtered(lambda a: a.provider != 'payumoney')

        return acquirers

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

    def _get_default_payment_method_id(self):
        self.ensure_one()
        if self.provider != 'payumoney':
            return super()._get_default_payment_method_id()
        return self.env.ref('payment_payumoney.payment_method_payumoney').id

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
        :return: The dict of acquirer-specific processing values
        :rtype: dict
        """
        res = super()._get_specific_rendering_values(processing_values)
        if self.provider != 'payumoney':
            return res

        first_name, last_name = payment_utils.split_partner_name(self.partner_id.name)
        api_url = 'https://secure.payu.in/_payment' if self.acquirer_id.state == 'enabled' \
            else 'https://sandboxsecure.payu.in/_payment'
        payumoney_values = {
            'key': self.acquirer_id.payumoney_merchant_key,
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
        payumoney_values['hash'] = self.acquirer_id._payumoney_generate_sign(
            payumoney_values, incoming=False,
        )
        return payumoney_values

    @api.model
    def _get_tx_from_feedback_data(self, provider, data):
        """ Override of payment to find the transaction based on Payumoney data.

        :param str provider: The provider of the acquirer that handled the transaction
        :param dict data: The feedback data sent by the provider
        :return: The transaction if found
        :rtype: recordset of `payment.transaction`
        :raise: ValidationError if inconsistent data were received
        :raise: ValidationError if the data match no transaction
        :raise: ValidationError if the signature can not be verified
        """
        tx = super()._get_tx_from_feedback_data(provider, data)
        if provider != 'payumoney':
            return tx

        reference = data.get('txnid')
        shasign = data.get('hash')
        if not reference or not shasign:
            raise ValidationError(
                "PayUmoney: " + _(
                    "Received data with missing reference (%(ref)s) or shasign (%(sign)s)",
                    ref=reference, sign=shasign,
                )
            )

        tx = self.search([('reference', '=', reference), ('provider', '=', 'payumoney')])
        if not tx:
            raise ValidationError(
                "PayUmoney: " + _("No transaction found matching reference %s.", reference)
            )

        # Verify shasign
        shasign_check = tx.acquirer_id._payumoney_generate_sign(data, incoming=True)
        if shasign_check != shasign:
            raise ValidationError(
                "PayUmoney: " + _(
                    "Invalid shasign: received %(sign)s, computed %(computed)s.",
                    sign=shasign, computed=shasign_check
                )
            )

        return tx

    def _process_feedback_data(self, data):
        """ Override of payment to process the transaction based on Payumoney data.

        Note: self.ensure_one()

        :param dict data: The feedback data sent by the provider
        :return: None
        """
        super()._process_feedback_data(data)
        if self.provider != 'payumoney':
            return

        status = data.get('status')
        self.acquirer_reference = data.get('payuMoneyId')

        if status == 'success':
            self._set_done()
        else:  # 'failure'
            # See https://www.payumoney.com/pdf/PayUMoney-Technical-Integration-Document.pdf
            error_code = data.get('Error')
            self._set_error(
                "PayUmoney: " + _("The payment encountered an error with code %s", error_code)
            )

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_payment_method
from . import payment_acquirer
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

## File: views\payment_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_acquirer_form" model="ir.ui.view">
        <field name="name">PayUMoney Acquirer Form</field>
        <field name="model">payment.acquirer</field>
        <field name="inherit_id" ref="payment.payment_acquirer_form"/>
        <field name="arch" type="xml">
            <xpath expr='//group[@name="acquirer"]' position='inside'>
                <group attrs="{'invisible': [('provider', '!=', 'payumoney')]}">
                    <field name="payumoney_merchant_key" attrs="{'required':[ ('provider', '=', 'payumoney'), ('state', '!=', 'disabled')]}"/>
                    <field name="payumoney_merchant_salt" attrs="{'required':[ ('provider', '=', 'payumoney'), ('state', '!=', 'disabled')]}" password="True"/>
                </group>
            </xpath>
        </field>
    </record>

</odoo>

```

