# Odoo Module: payment_alipay

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
            "Consider installing the Payment Provider: AsiaPay module instead.")


def post_init_hook(cr, registry):
    setup_provider(cr, registry, 'alipay')


def uninstall_hook(cr, registry):
    reset_payment_provider(cr, registry, 'alipay')

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Payment Provider: Alipay',
    'category': 'Accounting/Payment Providers',
    'version': '2.0',
    'sequence': 350,
    'summary': "This module is deprecated.",
    'description': " ",  # Non-empty string to avoid loading the README file.
    'depends': ['payment'],
    'data': [
        'views/payment_alipay_templates.xml',
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

import requests
from werkzeug.exceptions import Forbidden

from odoo import http
from odoo.exceptions import ValidationError
from odoo.http import request

_logger = logging.getLogger(__name__)


class AlipayController(http.Controller):
    _return_url = '/payment/alipay/return'
    _webhook_url = '/payment/alipay/webhook'

    @http.route(_return_url, type='http', auth='public', methods=['GET'])
    def alipay_return_from_checkout(self, **data):
        """ Process the notification data sent by Alipay after redirection from checkout.

        See https://global.alipay.com/docs/ac/web/sync.

        :param dict data: The notification data
        """
        _logger.info("handling redirection from Alipay with data:\n%s", pprint.pformat(data))

        # Check the integrity of the notification
        tx_sudo = request.env['payment.transaction'].sudo()._get_tx_from_notification_data(
            'alipay', data
        )
        self._verify_notification_signature(data, tx_sudo)

        # Handle the notification data
        tx_sudo._handle_notification_data('alipay', data)
        return request.redirect('/payment/status')

    @http.route(_webhook_url, type='http', auth='public', methods=['POST'], csrf=False)
    def alipay_webhook(self, **data):
        """ Process the notification data sent by Alipay to the webhook.

        See https://global.alipay.com/docs/ac/web/async.

        :param dict data: The notification data
        :return: The 'SUCCESS' string to acknowledge the notification
        :rtype: str
        """
        _logger.info("notification received from Alipay with data:\n%s", pprint.pformat(data))
        try:
            # Check the origin and integrity of the notification
            tx_sudo = request.env['payment.transaction'].sudo()._get_tx_from_notification_data(
                'alipay', data
            )
            self._verify_notification_origin(data, tx_sudo)
            self._verify_notification_signature(data, tx_sudo)

            # Handle the notification data
            tx_sudo._handle_notification_data('alipay', data)
        except ValidationError:  # Acknowledge the notification to avoid getting spammed
            _logger.exception("unable to handle the notification data; skipping to acknowledge")

        return 'SUCCESS'  # Acknowledge the notification

    @staticmethod
    def _verify_notification_origin(notification_data, tx_sudo):
        """ Check that the notification was sent by Alipay.

        See https://global.alipay.com/docs/ac/web/async#9727f6bd.

        :param dict notification_data: The notification data
        :param recordset tx_sudo: The sudoed transaction referenced in the notification data, as a
                                        `payment.transaction` record
        :return: None
        :raise: :class:`werkzeug.exceptions.Forbidden` if the notification origin can't be verified
        """
        url = tx_sudo.provider_id._alipay_get_api_url()
        payload = {
            'service': 'notify_verify',
            'partner': tx_sudo.provider_id.alipay_merchant_partner_id,
            'notify_id': notification_data['notify_id'],
        }
        try:
            response = requests.post(url, data=payload, timeout=60)
            response.raise_for_status()
        except (requests.exceptions.ConnectionError, requests.exceptions.HTTPError) as error:
            _logger.exception(
                "could not verify notification origin at %(url)s with data: %(data)s:\n%(error)s",
                {'url': url, 'data': payload, 'error': pprint.pformat(error.response.text)},
            )
            raise Forbidden()
        else:
            response_content = response.text
            if response_content != 'true':
                _logger.warning(
                    "Alipay did not confirm the origin of the notification with data:\n%s", payload
                )
                raise Forbidden()

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
        received_signature = notification_data.get('sign')
        if not received_signature:
            _logger.warning("received notification with missing signature")
            raise Forbidden()

        # Compare the received signature with the expected signature computed from the data
        expected_signature = tx_sudo.provider_id._alipay_compute_signature(notification_data)
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
-- disable adyen payment provider
UPDATE payment_provider
   SET alipay_merchant_partner_id = NULL,
       alipay_md5_signature_key = NULL,
       alipay_seller_email = NULL;

```

## File: data\payment_provider_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="payment_provider_alipay" model="payment.provider">
        <field name="name">Alipay</field>
        <field name="display_as">Credit Card (powered by Alipay)</field>
        <field name="image_128" type="base64" file="payment_alipay/static/description/icon.png"/>
        <field name="module_id" ref="base.module_payment_alipay"/>
        <!-- https://intl.alipay.com/ihome/home/about/buy.htm?topic=paymentMethods -->
        <field name="payment_icon_ids"
               eval="[(6, 0, [
                   ref('payment.payment_icon_cc_jcb'),
                   ref('payment.payment_icon_cc_mastercard'),
                   ref('payment.payment_icon_cc_western_union'),
                   ref('payment.payment_icon_cc_webmoney'),
                   ref('payment.payment_icon_cc_visa'),
               ])]"/>
        <field name="code">alipay</field>
        <field name="redirect_form_view_id" ref="redirect_form"/>
    </record>

</odoo>

```

## File: models\payment_provider.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
from hashlib import md5

from odoo import api, fields, models

_logger = logging.getLogger(__name__)


class PaymentProvider(models.Model):
    _inherit = 'payment.provider'

    code = fields.Selection(
        selection_add=[('alipay', "Alipay")], ondelete={'alipay': 'set default'})
    alipay_payment_method = fields.Selection(
        string="Account",
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

    #=== COMPUTE METHODS ===#

    def _compute_feature_support_fields(self):
        """ Override of `payment` to enable additional features. """
        super()._compute_feature_support_fields()
        self.filtered(lambda p: p.code == 'alipay').update({
            'support_fees': True,
        })

    # === BUSINESS METHODS ===#

    @api.model
    def _get_compatible_providers(self, *args, currency_id=None, **kwargs):
        """ Override of payment to unlist Alipay providers for unsupported currencies. """
        providers = super()._get_compatible_providers(*args, currency_id=currency_id, **kwargs)

        currency = self.env['res.currency'].browse(currency_id).exists()
        if currency and currency.name != 'CNY':
            providers = providers.filtered(
                lambda p: p.code != 'alipay' or p.alipay_payment_method != 'express_checkout'
            )

        return providers

    def _alipay_compute_signature(self, data):
        # Rearrange parameters in the data set alphabetically
        data_to_sign = sorted(data.items())
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
        :return: The dict of provider-specific processing values
        :rtype: dict
        """
        res = super()._get_specific_rendering_values(processing_values)
        if self.provider_code != 'alipay':
            return res

        base_url = self.provider_id.get_base_url()
        if self.fees:
            # Similarly to what is done in `payment::payment.transaction.create`, we need to round
            # the sum of the amount and of the fees to avoid inconsistent string representations.
            # E.g., str(1111.11 + 7.09) == '1118.1999999999998'
            total_fee = self.currency_id.round(self.amount + self.fees)
        else:
            total_fee = self.amount
        rendering_values = {
            '_input_charset': 'utf-8',
            'notify_url': urls.url_join(base_url, AlipayController._webhook_url),
            'out_trade_no': self.reference,
            'partner': self.provider_id.alipay_merchant_partner_id,
            'return_url': urls.url_join(base_url, AlipayController._return_url),
            'subject': self.reference,
            'total_fee': f'{total_fee:.2f}',
        }
        if self.provider_id.alipay_payment_method == 'standard_checkout':
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
                'seller_email': self.provider_id.alipay_seller_email,
            })

        sign = self.provider_id._alipay_compute_signature(rendering_values)
        rendering_values.update({
            'sign_type': 'MD5',
            'sign': sign,
            'api_url': self.provider_id._alipay_get_api_url(),
        })
        return rendering_values

    def _get_tx_from_notification_data(self, provider_code, notification_data):
        """ Override of payment to find the transaction based on Alipay data.

        :param str provider_code: The code of the provider that handled the transaction
        :param dict notification_data: The notification data sent by the provider
        :return: The transaction if found
        :rtype: recordset of `payment.transaction`
        :raise: ValidationError if inconsistent data were received
        :raise: ValidationError if the data match no transaction
        """
        tx = super()._get_tx_from_notification_data(provider_code, notification_data)
        if provider_code != 'alipay' or len(tx) == 1:
            return tx

        reference = notification_data.get('reference') or notification_data.get('out_trade_no')
        txn_id = notification_data.get('trade_no')
        if not reference or not txn_id:
            raise ValidationError(
                "Alipay: " + _(
                    "Received data with missing reference %(r)s or txn_id %(t)s.",
                    r=reference, t=txn_id
                )
            )

        tx = self.search([('reference', '=', reference), ('provider_code', '=', 'alipay')])
        if not tx:
            raise ValidationError(
                "Alipay: " + _("No transaction found matching reference %s.", reference)
            )

        return tx

    def _process_notification_data(self, notification_data):
        """ Override of payment to process the transaction based on Alipay data.

        Note: self.ensure_one()

        :param dict notification_data: The notification data sent by the provider
        :return: None
        :raise: ValidationError if inconsistent data were received
        """
        super()._process_notification_data(notification_data)
        if self.provider_code != 'alipay':
            return

        self.provider_reference = notification_data.get('trade_no')
        status = notification_data.get('trade_status')
        if status in ['TRADE_FINISHED', 'TRADE_SUCCESS']:
            self._set_done()
        elif status == 'TRADE_CLOSED':
            self._set_canceled()
        else:
            _logger.info(
                "received data with invalid payment status (%s) for transaction with reference %s",
                status, self.reference,
            )
            self._set_error("Alipay: " + _("received invalid transaction status: %s", status))

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

## File: views\payment_provider_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_provider_form" model="ir.ui.view">
        <field name="name">Alipay Provider Form</field>
        <field name="model">payment.provider</field>
        <field name="inherit_id" ref="payment.payment_provider_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='provider_creation_warning']" position="after">
                <div class="alert alert-danger"
                     role="alert"
                     attrs="{'invisible': [('code', '!=', 'alipay')]}">
                    This provider is deprecated.
                    Consider disabling it and moving to <strong>Asiapay</strong>.
                </div>
            </xpath>
            <group name="provider_credentials" position="inside">
                <group attrs="{'invisible': [('code', '!=', 'alipay')]}">
                    <field name="alipay_payment_method" widget="radio"/>
                    <field name="alipay_seller_email"
                           attrs="{'invisible': [('alipay_payment_method', '=', 'standard_checkout')], 'required': [('code', '=', 'alipay'), ('alipay_payment_method', '=', 'express_checkout')]}"/>
                    <field name="alipay_merchant_partner_id"/>
                    <field name="alipay_md5_signature_key" password="True"/>
                </group>
            </group>
        </field>
    </record>

</odoo>

```

