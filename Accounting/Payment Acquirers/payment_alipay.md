# Odoo Module: payment_alipay

Category: Accounting/Payment Acquirers

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

from odoo.addons.payment import reset_payment_acquirer


def uninstall_hook(cr, registry):
    reset_payment_acquirer(cr, registry, 'alipay')

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Alipay Payment Acquirer',
    'category': 'Accounting/Payment Acquirers',
    'version': '2.0',
    'sequence': 345,
    'summary': 'Payment Acquirer: Alipay Implementation',
    'description': " ",  # Non-empty string to avoid loading the README file.
    'depends': ['payment'],
    'data': [
        'views/payment_alipay_templates.xml',
        'views/payment_views.xml',
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

import requests

from odoo import _, http
from odoo.exceptions import ValidationError
from odoo.http import request

_logger = logging.getLogger(__name__)


class AlipayController(http.Controller):
    _return_url = '/payment/alipay/return'
    _notify_url = '/payment/alipay/notify'

    @http.route(_return_url, type='http', auth="public", methods=['GET'])
    def alipay_return_from_redirect(self, **data):
        """ Alipay return """
        _logger.info("received Alipay return data:\n%s", pprint.pformat(data))
        request.env['payment.transaction'].sudo()._handle_feedback_data('alipay', data)
        return request.redirect('/payment/status')

    @http.route(_notify_url, type='http', auth='public', methods=['POST'], csrf=False)
    def alipay_notify(self, **post):
        """ Alipay Notify """
        _logger.info("received Alipay notification data:\n%s", pprint.pformat(post))
        self._alipay_validate_notification(**post)
        request.env['payment.transaction'].sudo()._handle_feedback_data('alipay', post)
        return 'success'  # Return 'success' to stop receiving notifications for this tx

    def _alipay_validate_notification(self, **post):
        tx_sudo = request.env['payment.transaction'].sudo()._get_tx_from_feedback_data(
            'alipay', post
        )
        if not tx_sudo:
            raise ValidationError(
                "Alipay: " + _(
                    "Received notification data with unknown reference:\n%s", pprint.pformat(post)
                )
            )

        # Ensure that the notification was sent by Alipay
        # See https://global.alipay.com/docs/ac/wap/async
        acquirer_sudo = tx_sudo.acquirer_id
        val = {
            'service': 'notify_verify',
            'partner': acquirer_sudo.alipay_merchant_partner_id,
            'notify_id': post['notify_id']
        }
        response = requests.post(acquirer_sudo._alipay_get_api_url(), val, timeout=60)
        response.raise_for_status()
        if response.text != 'true':
            raise ValidationError(
                "Alipay: " + _(
                    "Received notification data not acknowledged by Alipay:\n%s",
                    pprint.pformat(post)
                )
            )

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

    <record id="payment.payment_acquirer_alipay" model="payment.acquirer">
        <field name="provider">alipay</field>
        <field name="redirect_form_view_id" ref="redirect_form"/>
        <field name="support_authorization">False</field>
        <field name="support_fees_computation">True</field>
        <field name="support_refund"></field>
        <field name="support_tokenization">False</field>
    </record>

    <record id="payment_method_alipay" model="account.payment.method">
        <field name="name">Alipay</field>
        <field name="code">alipay</field>
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
        res['alipay'] = {'mode': 'electronic', 'domain': [('type', '=', 'bank')]}
        return res

```

## File: models\payment_acquirer.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
from hashlib import md5

from odoo import api, fields, models

_logger = logging.getLogger(__name__)


class PaymentAcquirer(models.Model):
    _inherit = 'payment.acquirer'

    provider = fields.Selection(
        selection_add=[('alipay', "Alipay")], ondelete={'alipay': 'set default'})
    alipay_payment_method = fields.Selection(
        string="Account",
        help="* Cross-border: For the overseas seller \n* Express Checkout: For the Chinese Seller",
        selection=[
            ('express_checkout', 'Express Checkout (only for Chinese merchants)'),
            ('standard_checkout', 'Cross-border')
        ], default='express_checkout', required_if_provider='alipay')
    alipay_merchant_partner_id = fields.Char(
        string="Merchant Partner ID",
        help="The public partner ID solely used to identify the account with Alipay",
        required_if_provider='alipay')
    alipay_md5_signature_key = fields.Char(
        string="MD5 Signature Key", required_if_provider='alipay', groups='base.group_system')
    alipay_seller_email = fields.Char(
        string="Alipay Seller Email", help="The public Alipay partner email")

    @api.model
    def _get_compatible_acquirers(self, *args, currency_id=None, **kwargs):
        """ Override of payment to unlist Alipay acquirers for unsupported currencies. """
        acquirers = super()._get_compatible_acquirers(*args, currency_id=currency_id, **kwargs)

        currency = self.env['res.currency'].browse(currency_id).exists()
        if currency and currency.name != 'CNY':
            acquirers = acquirers.filtered(
                lambda a: a.provider != 'alipay' or a.alipay_payment_method != 'express_checkout'
            )

        return acquirers

    def _alipay_build_sign(self, val):
        # Rearrange parameters in the data set alphabetically
        data_to_sign = sorted(val.items())
        # Format key-value pairs of parameters that should be signed
        data_to_sign = [f"{k}={v}" for k, v in data_to_sign
                        if k not in ['sign', 'sign_type', 'reference']]
        # Build the data string of &-separated key-value pairs
        data_string = '&'.join(data_to_sign)
        data_string += self.alipay_md5_signature_key
        return md5(data_string.encode('utf-8')).hexdigest()

    def _alipay_get_api_url(self):
        if self.state == 'enabled':
            return 'https://mapi.alipay.com/gateway.do'
        else:  # test environment
            return 'https://openapi.alipaydev.com/gateway.do'

    def _get_default_payment_method_id(self):
        self.ensure_one()
        if self.provider != 'alipay':
            return super()._get_default_payment_method_id()
        return self.env.ref('payment_alipay.payment_method_alipay').id

```

## File: models\payment_transaction.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from werkzeug import urls

from odoo import _, api, models
from odoo.exceptions import ValidationError
from odoo.tools.float_utils import float_compare

from odoo.addons.payment_alipay.controllers.main import AlipayController

_logger = logging.getLogger(__name__)


class PaymentTransaction(models.Model):
    _inherit = 'payment.transaction'

    def _get_specific_rendering_values(self, processing_values):
        """ Override of payment to return Alipay-specific rendering values.

        Note: self.ensure_one() from `_get_processing_values`

        :param dict processing_values: The generic and specific processing values of the transaction
        :return: The dict of acquirer-specific processing values
        :rtype: dict
        """
        res = super()._get_specific_rendering_values(processing_values)
        if self.provider != 'alipay':
            return res

        base_url = self.acquirer_id.get_base_url()
        if self.fees:
            # Similarly to what is done in `payment::payment.transaction.create`, we need to round
            # the sum of the amount and of the fees to avoid inconsistent string representations.
            # E.g., str(1111.11 + 7.09) == '1118.1999999999998'
            total_fee = self.currency_id.round(self.amount + self.fees)
        else:
            total_fee = self.amount
        rendering_values = {
            '_input_charset': 'utf-8',
            'notify_url': urls.url_join(base_url, AlipayController._notify_url),
            'out_trade_no': self.reference,
            'partner': self.acquirer_id.alipay_merchant_partner_id,
            'return_url': urls.url_join(base_url, AlipayController._return_url),
            'subject': self.reference,
            'total_fee': f'{total_fee:.2f}',
        }
        if self.acquirer_id.alipay_payment_method == 'standard_checkout':
            # https://global.alipay.com/docs/ac/global/create_forex_trade
            rendering_values.update({
                'service': 'create_forex_trade',
                'product_code': 'NEW_OVERSEAS_SELLER',
                'currency': self.currency_id.name,
            })
        else:
            rendering_values.update({
                'service': 'create_direct_pay_by_user',
                'payment_type': 1,
                'seller_email': self.acquirer_id.alipay_seller_email,
            })

        sign = self.acquirer_id._alipay_build_sign(rendering_values)
        rendering_values.update({
            'sign_type': 'MD5',
            'sign': sign,
            'api_url': self.acquirer_id._alipay_get_api_url(),
        })
        return rendering_values

    @api.model
    def _get_tx_from_feedback_data(self, provider, data):
        """ Override of payment to find the transaction based on Alipay data.

        :param str provider: The provider of the acquirer that handled the transaction
        :param dict data: The feedback data sent by the provider
        :return: The transaction if found
        :rtype: recordset of `payment.transaction`
        :raise: ValidationError if inconsistent data were received
        :raise: ValidationError if the data match no transaction
        """
        tx = super()._get_tx_from_feedback_data(provider, data)
        if provider != 'alipay':
            return tx

        reference = data.get('reference') or data.get('out_trade_no')
        txn_id = data.get('trade_no')
        if not reference or not txn_id:
            raise ValidationError(
                "Alipay: " + _(
                    "Received data with missing reference %(r)s or txn_id %(t)s.",
                    r=reference, t=txn_id
                )
            )

        tx = self.search([('reference', '=', reference), ('provider', '=', 'alipay')])
        if not tx:
            raise ValidationError(
                "Alipay: " + _("No transaction found matching reference %s.", reference)
            )

        # Verify signature (done here because we need the reference to get the acquirer)
        sign_check = tx.acquirer_id._alipay_build_sign(data)
        sign = data.get('sign')
        if sign != sign_check:
            raise ValidationError(
                "Alipay: " + _(
                    "Expected signature %(sc) but received %(sign)s.", sc=sign_check, sign=sign
                )
            )

        return tx

    def _process_feedback_data(self, data):
        """ Override of payment to process the transaction based on Alipay data.

        Note: self.ensure_one()

        :param dict data: The feedback data sent by the provider
        :return: None
        :raise: ValidationError if inconsistent data were received
        """
        super()._process_feedback_data(data)
        if self.provider != 'alipay':
            return

        if float_compare(float(data.get('total_fee', '0.0')), (self.amount + self.fees), 2) != 0:
            # mc_gross is amount + fees
            logging_values = {
                'amount': data.get('total_fee', '0.0'),
                'total': self.amount,
                'fees': self.fees,
                'reference': self.reference,
            }
            _logger.error(
                "the paid amount (%(amount)s) does not match the total + fees (%(total)s + "
                "%(fees)s) for the transaction with reference %(reference)s", logging_values
            )
            raise ValidationError("Alipay: " + _("The amount does not match the total + fees."))
        if self.acquirer_id.alipay_payment_method == 'standard_checkout':
            if data.get('currency') != self.currency_id.name:
                raise ValidationError(
                    "Alipay: " + _(
                        "The currency returned by Alipay %(rc)s does not match the transaction "
                        "currency %(tc)s.", rc=data.get('currency'), tc=self.currency_id.name
                    )
                )
        elif data.get('seller_email') != self.acquirer_id.alipay_seller_email:
            _logger.error(
                "the seller email (%s) does not match the configured Alipay account (%s).",
                data.get('seller_email'), self.acquirer_id.alipay_seller_email
            )
            raise ValidationError(
                "Alipay: " + _("The seller email does not match the configured Alipay account.")
            )

        self.acquirer_reference = data.get('trade_no')
        status = data.get('trade_status')
        if status in ['TRADE_FINISHED', 'TRADE_SUCCESS']:
            self._set_done()
        elif status == 'TRADE_CLOSED':
            self._set_canceled()
        else:
            _logger.info(
                "received invalid transaction status for transaction with reference %s: %s",
                self.reference, status
            )
            self._set_error("Alipay: " + _("received invalid transaction status: %s", status))

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
<svg id="Layer_1" data-name="Layer 1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 70 70">
  <defs>
    <mask id="mask" x="0" y="0" width="70" height="70" maskUnits="userSpaceOnUse">
      <g id="b">
        <path id="a" d="M4,0H65c4,0,5,1,5,5V65c0,4-1,5-5,5H4c-3,0-4-1-4-5V5C0,1,1,0,4,0Z" fill="#fff" fill-rule="evenodd"/>
      </g>
    </mask>
    <linearGradient id="linear-gradient" x1="-1349.79" y1="-4.39" x2="-1350.79" y2="-5.39" gradientTransform="matrix(70, 0, 0, -70, 94554.99, -307.17)" gradientUnits="userSpaceOnUse">
      <stop offset="0" stop-color="#94b6c8"/>
      <stop offset="1" stop-color="#6a9eba"/>
    </linearGradient>
  </defs>
  <g mask="url(#mask)">
    <g>
      <path d="M0,0H70V70H0Z" fill-rule="evenodd" fill="url(#linear-gradient)"/>
      <path d="M4,1H65c2.67,0,4.33.67,5,2V0H0V3C.67,1.67,2,1,4,1Z" fill="#fff" fill-opacity="0.38" fill-rule="evenodd"/>
      <path d="M4,69a3.66,3.66,0,0,1-4-4V33.92L14.68,19.3,34,18.45,35.6,27.6l3.27-3.27,10.33-.18.45,24.05L28.85,69Z" fill="#393939" fill-rule="evenodd" opacity="0.32" style="isolation: isolate"/>
      <path d="M4,69H65c2.67,0,4.33-1,5-3v4H0V66A3.92,3.92,0,0,0,4,69Z" fill-opacity="0.38" fill-rule="evenodd"/>
      <g>
        <g opacity="0.4">
          <path d="M47,24.7H36.5v2.74H46.7a.34.34,0,0,1,.34.34V31H36.5v6.5H47v9.72a.34.34,0,0,1-.34.34H20.3a.35.35,0,0,1-.34-.34V46H17.25v1.56A2.73,2.73,0,0,0,20,50.29H47a2.73,2.73,0,0,0,2.71-2.74V27.44A2.73,2.73,0,0,0,47,24.7Z"/>
          <polygon points="12.42 55.64 13.46 55.64 13.46 58.47 14.22 58.47 14.22 55.64 15.26 55.64 15.26 54.99 12.42 54.99 12.42 55.64"/>
          <polygon points="17.31 57.38 17.3 57.38 16.49 54.99 15.42 54.99 15.42 58.47 16.13 58.47 16.13 56.03 16.14 56.03 16.99 58.47 17.58 58.47 18.42 56.01 18.43 56.01 18.43 58.47 19.14 58.47 19.14 54.99 18.07 54.99 17.31 57.38"/>
          <g id="_13-alipay" data-name="13-alipay">
            <path d="M30.64,18.77H14.85a3.6,3.6,0,0,0-3.6,3.61V38.17a3.61,3.61,0,0,0,3.6,3.61H30.64a3.62,3.62,0,0,0,3.61-3.58c-2.36-1.31-5.68-3.1-8.81-4.54a9.57,9.57,0,0,1-7.63,4.16c-3.63,0-4.82-2.33-5-3.92-.2-2,.77-4.19,5.11-4.19A23.63,23.63,0,0,1,24.46,31a20.84,20.84,0,0,0,1.36-3.1H16.67V27H21.4v-1.6H15.78v-1H21.4V21.87H24v2.59h5.62v1H24V27h4.56a34.92,34.92,0,0,1-2,4.67c2.51.86,5.14,1.85,7.63,2.71v-12A3.59,3.59,0,0,0,30.64,18.77Zm-17,14.94c.05,1,.53,2.76,3.59,2.76,2.68,0,4.76-2,6.06-3.74a17.49,17.49,0,0,0-5.62-1.62C14.23,31.11,13.62,32.81,13.67,33.71Z"/>
          </g>
        </g>
        <g>
          <path d="M49,22.7H38.5v2.74H48.7a.34.34,0,0,1,.34.34V29H38.5v6.5H49v9.72a.34.34,0,0,1-.34.34H22.3a.35.35,0,0,1-.34-.34V44H19.25v1.56A2.73,2.73,0,0,0,22,48.29H49a2.73,2.73,0,0,0,2.71-2.74V25.44A2.73,2.73,0,0,0,49,22.7Z" fill="#fff"/>
          <polygon points="14.42 53.64 15.46 53.64 15.46 56.47 16.22 56.47 16.22 53.64 17.26 53.64 17.26 52.99 14.42 52.99 14.42 53.64" fill="#fff"/>
          <polygon points="19.31 55.38 19.3 55.38 18.49 52.99 17.42 52.99 17.42 56.47 18.13 56.47 18.13 54.03 18.14 54.03 18.99 56.47 19.58 56.47 20.42 54.01 20.43 54.01 20.43 56.47 21.14 56.47 21.14 52.99 20.07 52.99 19.31 55.38" fill="#fff"/>
          <g id="_13-alipay-2" data-name="13-alipay">
            <path d="M32.64,16.77H16.85a3.6,3.6,0,0,0-3.6,3.61V36.17a3.61,3.61,0,0,0,3.6,3.61H32.64a3.62,3.62,0,0,0,3.61-3.58c-2.36-1.31-5.68-3.1-8.81-4.54a9.57,9.57,0,0,1-7.63,4.16c-3.63,0-4.82-2.33-5-3.92-.2-2,.77-4.19,5.11-4.19A23.63,23.63,0,0,1,26.46,29a20.84,20.84,0,0,0,1.36-3.1H18.67V25H23.4v-1.6H17.78v-1H23.4V19.87H26v2.59h5.62v1H26V25h4.56a34.92,34.92,0,0,1-2,4.67c2.51.86,5.14,1.85,7.63,2.71v-12A3.59,3.59,0,0,0,32.64,16.77Zm-17,14.94c.05,1,.53,2.76,3.59,2.76,2.68,0,4.76-2,6.06-3.74a17.49,17.49,0,0,0-5.62-1.62C16.23,29.11,15.62,30.81,15.67,31.71Z" fill="#fff"/>
          </g>
        </g>
      </g>
    </g>
  </g>
</svg>

```

## File: views\payment_alipay_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="redirect_form">
        <form t-att-action="api_url" method="post">
            <input type="hidden" name="_input_charset" t-att-value="_input_charset"/>
            <input t-if="service == 'create_forex_trade'" type="hidden" name="currency" t-att-value="currency"/>
            <input type="hidden" name="notify_url" t-att-value="notify_url"/>
            <input type="hidden" name="out_trade_no" t-att-value="out_trade_no"/>
            <input type="hidden" name="partner" t-att-value="partner"/>
            <input t-if="product_code" type="hidden" name="product_code" t-att-value="product_code"/>
            <input type="hidden" name="return_url" t-att-value="return_url"/>
            <input type="hidden" name="service" t-att-value="service"/>
            <input type="hidden" name="sign" t-att-value="sign"/>
            <input type="hidden" name="subject" t-att-value="subject"/>
            <input type="hidden" name="total_fee" t-att-value="total_fee"/>
            <input type="hidden" name="sign_type" t-att-value="sign_type"/>
            <input t-if="payment_type" type="hidden" name="payment_type" t-att-value="payment_type"/>
            <input t-if="seller_email" type="hidden" name="seller_email" t-att-value="seller_email"/>
        </form>
    </template>

</odoo>

```

## File: views\payment_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_acquirer_form" model="ir.ui.view">
        <field name="name">Alipay Acquirer Form</field>
        <field name="model">payment.acquirer</field>
        <field name="inherit_id" ref="payment.payment_acquirer_form"/>
        <field name="arch" type="xml">
            <xpath expr='//group[@name="acquirer"]' position='inside'>
                <group attrs="{'invisible': [('provider', '!=', 'alipay')]}">
                    <field name="alipay_payment_method" widget="radio"/>
                    <field name="alipay_seller_email"
                           attrs="{'invisible': [('alipay_payment_method', '=', 'standard_checkout')], 'required': [('provider', '=', 'alipay'), ('alipay_payment_method', '=', 'express_checkout')]}"/>
                    <field name="alipay_merchant_partner_id" password="True"/>
                    <field name="alipay_md5_signature_key" password="True"/>
                </group>
            </xpath>
        </field>
    </record>

</odoo>

```

