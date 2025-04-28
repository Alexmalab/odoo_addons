# Odoo Module: payment_payumoney

Category: Accounting/Payment Acquirers

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import controllers
from odoo.addons.payment.models.payment_acquirer import create_missing_journal_for_acquirers
from odoo.addons.payment import reset_payment_provider

def uninstall_hook(cr, registry):
    reset_payment_provider(cr, registry, 'payumoney')

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'PayuMoney Payment Acquirer',
    'category': 'Accounting/Payment Acquirers',
    'sequence': 375,
    'summary': 'Payment Acquirer: PayuMoney Implementation',
    'description': """
    PayuMoney Payment Acquirer for India.

    PayUmoney payment gateway supports only INR currency.
    """,
    'depends': ['payment'],
    'data': [
        'views/payment_views.xml',
        'views/payment_payumoney_templates.xml',
        'data/payment_acquirer_data.xml',
    ],
    'application': True,
    'post_init_hook': 'create_missing_journal_for_acquirers',
    'uninstall_hook': 'uninstall_hook',
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import pprint
import werkzeug

from odoo import http
from odoo.http import request

_logger = logging.getLogger(__name__)


class PayuMoneyController(http.Controller):
    @http.route(['/payment/payumoney/return', '/payment/payumoney/cancel', '/payment/payumoney/error'], type='http', auth='public', csrf=False, save_session=False)
    def payu_return(self, **post):
        """ PayUmoney.
        The session cookie created by Odoo has not the attribute SameSite. Most of browsers will force this attribute
        with the value 'Lax'. After the payment, PayUMoney will perform a POST request on this route. For all these reasons,
        the cookie won't be added to the request. As a result, if we want to save the session, the server will create
        a new session cookie. Therefore, the previous session and all related information will be lost, so it will lead
        to undesirable behaviors. This is the reason why `save_session=False` is needed.
        """
        _logger.info(
            'PayUmoney: entering form_feedback with post data %s', pprint.pformat(post))
        if post:
            request.env['payment.transaction'].sudo().form_feedback(post, 'payumoney')
        return werkzeug.utils.redirect('/payment/process')

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
<odoo>
    <data noupdate="1">
        <record id="payment.payment_acquirer_payu" model="payment.acquirer">
            <field name="name">PayUmoney</field>
            <field name="image_128" type="base64" file="payment_payumoney/static/src/img/payumoney_icon.png"/>
            <field name="provider">payumoney</field>
            <field name="company_id" ref="base.main_company"/>
            <field name="view_template_id" ref="payumoney_form"/>
        </record>
    </data>
</odoo>

```

## File: models\payment.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import hashlib

from werkzeug import urls

from odoo import api, fields, models, _
from odoo.addons.payment.models.payment_acquirer import ValidationError
from odoo.tools.float_utils import float_compare

import logging

_logger = logging.getLogger(__name__)


class PaymentAcquirerPayumoney(models.Model):
    _inherit = 'payment.acquirer'

    provider = fields.Selection(selection_add=[
        ('payumoney', 'PayUmoney')
    ], ondelete={'payumoney': 'set default'})
    payumoney_merchant_key = fields.Char(string='Merchant Key', required_if_provider='payumoney', groups='base.group_user')
    payumoney_merchant_salt = fields.Char(string='Merchant Salt', required_if_provider='payumoney', groups='base.group_user')

    def _get_payumoney_urls(self, environment):
        """ PayUmoney URLs"""
        if environment == 'prod':
            return {'payumoney_form_url': 'https://secure.payu.in/_payment'}
        else:
            return {'payumoney_form_url': 'https://sandboxsecure.payu.in/_payment'}

    def _payumoney_generate_sign(self, inout, values):
        """ Generate the shasign for incoming or outgoing communications.
        :param self: the self browse record. It should have a shakey in shakey out
        :param string inout: 'in' (odoo contacting payumoney) or 'out' (payumoney
                             contacting odoo).
        :param dict values: transaction values

        :return string: shasign
        """
        if inout not in ('in', 'out'):
            raise Exception("Type must be 'in' or 'out'")

        if inout == 'in':
            keys = "key|txnid|amount|productinfo|firstname|email|udf1|||||||||".split('|')
            sign = ''.join('%s|' % (values.get(k) or '') for k in keys)
            sign += self.payumoney_merchant_salt or ''
        else:
            keys = "|status||||||||||udf1|email|firstname|productinfo|amount|txnid".split('|')
            sign = ''.join('%s|' % (values.get(k) or '') for k in keys)
            sign = self.payumoney_merchant_salt + sign + self.payumoney_merchant_key

        shasign = hashlib.sha512(sign.encode('utf-8')).hexdigest()
        return shasign

    def payumoney_form_generate_values(self, values):
        self.ensure_one()
        base_url = self.get_base_url()
        payumoney_values = dict(values,
                                key=self.payumoney_merchant_key,
                                txnid=values['reference'],
                                amount=values['amount'],
                                productinfo=values['reference'],
                                firstname=values.get('partner_name'),
                                email=values.get('partner_email'),
                                phone=values.get('partner_phone'),
                                service_provider='payu_paisa',
                                surl=urls.url_join(base_url, '/payment/payumoney/return'),
                                furl=urls.url_join(base_url, '/payment/payumoney/error'),
                                curl=urls.url_join(base_url, '/payment/payumoney/cancel')
                                )

        payumoney_values['udf1'] = payumoney_values.pop('return_url', '/')
        payumoney_values['hash'] = self._payumoney_generate_sign('in', payumoney_values)
        return payumoney_values

    def payumoney_get_form_action_url(self):
        self.ensure_one()
        environment = 'prod' if self.state == 'enabled' else 'test'
        return self._get_payumoney_urls(environment)['payumoney_form_url']


class PaymentTransactionPayumoney(models.Model):
    _inherit = 'payment.transaction'

    @api.model
    def _payumoney_form_get_tx_from_data(self, data):
        """ Given a data dict coming from payumoney, verify it and find the related
        transaction record. """
        reference = data.get('txnid')
        pay_id = data.get('mihpayid')
        shasign = data.get('hash')
        if not reference or not pay_id or not shasign:
            raise ValidationError(_('PayUmoney: received data with missing reference (%s) or pay_id (%s) or shasign (%s)') % (reference, pay_id, shasign))

        transaction = self.search([('reference', '=', reference)])

        if not transaction:
            error_msg = (_('PayUmoney: received data for reference %s; no order found') % (reference))
            raise ValidationError(error_msg)
        elif len(transaction) > 1:
            error_msg = (_('PayUmoney: received data for reference %s; multiple orders found') % (reference))
            raise ValidationError(error_msg)

        #verify shasign
        shasign_check = transaction.acquirer_id._payumoney_generate_sign('out', data)
        if shasign_check.upper() != shasign.upper():
            raise ValidationError(_('PayUmoney: invalid shasign, received %s, computed %s, for data %s') % (shasign, shasign_check, data))
        return transaction

    def _payumoney_form_get_invalid_parameters(self, data):
        invalid_parameters = []

        if self.acquirer_reference and data.get('mihpayid') != self.acquirer_reference:
            invalid_parameters.append(
                ('Transaction Id', data.get('mihpayid'), self.acquirer_reference))
        #check what is buyed
        if float_compare(float(data.get('amount', '0.0')), self.amount, 2) != 0:
            invalid_parameters.append(
                ('Amount', data.get('amount'), '%.2f' % self.amount))

        return invalid_parameters

    def _payumoney_form_validate(self, data):
        status = data.get('status')
        result = self.write({
            'acquirer_reference': data.get('payuMoneyId'),
            'date': fields.Datetime.now(),
        })
        if status == 'success':
            self._set_transaction_done()
        elif status != 'pending':
            self._set_transaction_cancel()
        else:
            self._set_transaction_pending()
        return result

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import payment

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#7CC098"/><stop offset="100%" stop-color="#5F8A71"/></linearGradient><path id="d" d="M32.423 21.58v-3a1 1 0 0 1 1-1h4a1 1 0 0 1 1 1v4a1 1 0 0 1-1 1H34.25V34.9c0 2.9-.81 5.055-2.43 6.465-1.62 1.41-3.86 2.115-6.72 2.115-2.9 0-5.145-.7-6.735-2.1-1.59-1.4-2.385-3.56-2.385-6.48V21.58h4.71V34.9c0 .58.05 1.15.15 1.71.1.56.31 1.055.63 1.485.32.43.765.78 1.335 1.05s1.335.405 2.295.405c1.68 0 2.84-.375 3.48-1.125.64-.75.96-1.925.96-3.525V21.58h2.883zM19.25 46h2.714v1.212c0 .188.152.343.339.343h26.394a.342.342 0 0 0 .339-.343V37.5H38.5V31h10.536v-3.212a.342.342 0 0 0-.339-.343H39.5v-2.742h9.536c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742V46zm-3.79 9.642h-1.037V55h2.833v.642h-1.037v2.832h-.76v-2.832zM17.422 55h1.07l.809 2.389h.01L20.075 55h1.07v3.474h-.711v-2.462h-.01l-.847 2.462h-.586l-.848-2.438h-.01v2.438h-.711V55zm23-41.42h3a1 1 0 0 1 1 1v3a1 1 0 0 1-1 1h-3a1 1 0 0 1-1-1v-3a1 1 0 0 1 1-1zm-5-3h2a1 1 0 0 1 1 1v2a1 1 0 0 1-1 1h-2a1 1 0 0 1-1-1v-2a1 1 0 0 1 1-1z"/><path id="e" d="M32.423 19.58v-3a1 1 0 0 1 1-1h4a1 1 0 0 1 1 1v4a1 1 0 0 1-1 1H34.25V32.9c0 2.9-.81 5.055-2.43 6.465-1.62 1.41-3.86 2.115-6.72 2.115-2.9 0-5.145-.7-6.735-2.1-1.59-1.4-2.385-3.56-2.385-6.48V19.58h4.71V32.9c0 .58.05 1.15.15 1.71.1.56.31 1.055.63 1.485.32.43.765.78 1.335 1.05s1.335.405 2.295.405c1.68 0 2.84-.375 3.48-1.125.64-.75.96-1.925.96-3.525V19.58h2.883zM19.25 44h2.714v1.212c0 .188.152.343.339.343h26.394a.342.342 0 0 0 .339-.343V35.5H38.5V29h10.536v-3.212a.342.342 0 0 0-.339-.343H39.5v-2.742h9.536c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742V44zm-3.79 9.642h-1.037V53h2.833v.642h-1.037v2.832h-.76v-2.832zM17.422 53h1.07l.809 2.389h.01L20.075 53h1.07v3.474h-.711v-2.462h-.01l-.847 2.462h-.586l-.848-2.438h-.01v2.438h-.711V53zm23-41.42h3a1 1 0 0 1 1 1v3a1 1 0 0 1-1 1h-3a1 1 0 0 1-1-1v-3a1 1 0 0 1 1-1zm-5-3h2a1 1 0 0 1 1 1v2a1 1 0 0 1-1 1h-2a1 1 0 0 1-1-1v-2a1 1 0 0 1 1-1z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M4 69c-2 0-4-1-4-4V33.916L16 19.58l3.25.42L35 8.58 38 12l-.89 2 2.584-2 4.729 4-5.923 6.703L48 23l3.576 1.968-.236 22.01L39.224 69H4z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: views\payment_payumoney_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="payumoney_form">
        <input type="hidden" name="data_set" t-att-data-action-url="tx_url" data-remove-me=""/>
        <input type="hidden" name="key" t-att-value='key' />
        <input type="hidden" name="txnid" t-att-value='txnid' />
        <input type="hidden" name="amount" t-att-value='amount' />
        <input type="hidden" name="productinfo" t-att-value='productinfo' />
        <input type="hidden" name="firstname" t-att-value='firstname' />
        <input type="hidden" name="email" t-att-value='email' />
        <input type="hidden" name="phone" t-att-value='phone'/>
        <input type="hidden" name="service_provider" t-att-value='service_provider' />
        <input type="hidden" name="hash" t-att-value='hash' />
        <input type="hidden" name="surl" t-att-value='surl' />
        <input type="hidden" name="furl" t-att-value='furl' />
        <input type="hidden" name="curl" t-att-value='curl' />
        <input type="hidden" name="udf1" t-att-value='udf1'/>
    </template>
</odoo>

```

## File: views\payment_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="payment_acquirer_form_payumoney" model="ir.ui.view">
        <field name="name">payment.acquirer.form.inherit</field>
        <field name="model">payment.acquirer</field>
        <field name="inherit_id" ref="payment.acquirer_form"/>
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

