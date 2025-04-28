# Odoo Module: payment_alipay

Category: Accounting/Payment Providers

This file contains the source code of the Odoo module.

## File: const.py

```python
# The codes of the payment methods to activate when Alipay is activated.
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
            "Consider installing the Payment Provider: AsiaPay module instead.")


def post_init_hook(env):
    setup_provider(env, 'alipay')


def uninstall_hook(env):
    reset_payment_provider(env, 'alipay')

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
        <field name="image_128" type="base64" file="payment_alipay/static/description/icon.png"/>
        <field name="module_id" ref="base.module_payment_alipay"/>
        <!-- https://intl.alipay.com/ihome/home/about/buy.htm?topic=paymentMethods -->
        <field name="payment_method_ids"
               eval="[(6, 0, [
                   ref('payment.payment_method_card'),
                   ref('payment.payment_method_rabbit_line_pay'),
                   ref('payment.payment_method_truemoney'),
                   ref('payment.payment_method_boost'),
                   ref('payment.payment_method_touch_n_go'),
                   ref('payment.payment_method_gcash'),
                   ref('payment.payment_method_billease'),
                   ref('payment.payment_method_bpi'),
                   ref('payment.payment_method_maya'),
                   ref('payment.payment_method_dana'),
                   ref('payment.payment_method_akulaku'),
                   ref('payment.payment_method_kredivo'),
                   ref('payment.payment_method_kakaopay'),
                   ref('payment.payment_method_naver_pay'),
                   ref('payment.payment_method_toss_pay'),
                   ref('payment.payment_method_alipay'),
                   ref('payment.payment_method_alipay_hk'),
                   ref('payment.payment_method_dolfin'),
                   ref('payment.payment_method_grabpay'),
                   ref('payment.payment_method_gopay'),
                   ref('payment.payment_method_linkaja'),
                   ref('payment.payment_method_ovo'),
                   ref('payment.payment_method_paypay'),
                   ref('payment.payment_method_zalopay'),
                   ref('payment.payment_method_bangkok_bank'),
                   ref('payment.payment_method_bank_of_ayudhya'),
                   ref('payment.payment_method_krungthai_bank'),
                   ref('payment.payment_method_scb'),
                   ref('payment.payment_method_blik'),
                   ref('payment.payment_method_gsb'),
                   ref('payment.payment_method_kasikorn_bank'),
                   ref('payment.payment_method_promptpay'),
                   ref('payment.payment_method_paynow'),
                   ref('payment.payment_method_bni'),
                   ref('payment.payment_method_mandiri'),
                   ref('payment.payment_method_maybank'),
                   ref('payment.payment_method_cimb_niaga'),
                   ref('payment.payment_method_bsi'),
                   ref('payment.payment_method_qris'),
                   ref('payment.payment_method_pix'),
                   ref('payment.payment_method_bancontact'),
                   ref('payment.payment_method_giropay'),
                   ref('payment.payment_method_ideal'),
                   ref('payment.payment_method_payu'),
                   ref('payment.payment_method_p24'),
                   ref('payment.payment_method_sofort'),
                   ref('payment.payment_method_eps'),
                   ref('payment.payment_method_bancomat_pay'),
                   ref('payment.payment_method_brankas'),
                   ref('payment.payment_method_pay_easy'),
                   ref('payment.payment_method_fpx'),
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

from odoo.addons.payment_alipay import const

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

    # === BUSINESS METHODS ===#

    @api.model
    def _get_compatible_providers(self, *args, currency_id=None, **kwargs):
        """ Override of payment to unlist Alipay providers when the currency is not CNY in case of
        express checkout. """
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

    def _get_default_payment_method_codes(self):
        """ Override of `payment` to return the default payment method codes. """
        default_codes = super()._get_default_payment_method_codes()
        if self.code != 'alipay':
            return default_codes
        return const.DEFAULT_PAYMENT_METHODS_CODES

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
        rendering_values = {
            '_input_charset': 'utf-8',
            'notify_url': urls.url_join(base_url, AlipayController._webhook_url),
            'out_trade_no': self.reference,
            'partner': self.provider_id.alipay_merchant_partner_id,
            'return_url': urls.url_join(base_url, AlipayController._return_url),
            'subject': self.reference,
            'total_fee': f'{self.amount:.2f}',
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
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M39.274 4H10.728A6.727 6.727 0 0 0 4 10.728v28.544A6.727 6.727 0 0 0 10.728 46h28.546A6.726 6.726 0 0 0 46 39.272V10.728A6.725 6.725 0 0 0 39.274 4Z" fill="#1677FF"/><path d="M15.348 36.298c-6.534 0-8.466-5.146-5.236-7.962 1.077-.952 3.046-1.416 4.096-1.52 3.882-.384 7.474 1.097 11.714 3.166-2.98 3.888-6.776 6.316-10.574 6.316Zm23.232-5.926c-1.681-.563-3.936-1.423-6.448-2.332 1.508-2.622 2.713-5.609 3.505-8.854h-8.279v-2.982H37.5V14.54H27.358V9.569H23.22c-.726 0-.726.716-.726.716v4.255H12.236v1.664h10.257v2.982h-8.468v1.664h16.424a29.28 29.28 0 0 1-2.365 5.781c-5.33-1.758-11.017-3.183-14.59-2.306-2.285.563-3.756 1.567-4.62 2.62-3.969 4.828-1.123 12.16 7.257 12.16 4.955 0 9.728-2.762 13.427-7.314C35.075 34.443 46 38.997 46 38.997v-6.49s-1.372-.11-7.42-2.135Z" fill="#fff"/><path d="M43.105 4h.972V.818h1.108V0H42v.818h1.105V4Zm2.775 0h.858V1.453h.053L47.663 4h.555l.871-2.547h.056V4H50V0h-1.108l-.924 2.714h-.05L46.99 0h-1.11v4Z" fill="#D1D5DB"/></svg>

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
                     invisible="code != 'alipay'">
                    This provider is deprecated.
                    Consider disabling it and moving to <strong>Asiapay</strong>.
                </div>
            </xpath>
            <group name="provider_credentials" position="inside">
                <group invisible="code != 'alipay'">
                    <field name="alipay_payment_method" widget="radio"/>
                    <field name="alipay_seller_email"
                           invisible="alipay_payment_method == 'standard_checkout'"
                           required="code == 'alipay' and alipay_payment_method == 'express_checkout'"/>
                    <field name="alipay_merchant_partner_id"/>
                    <field name="alipay_md5_signature_key" password="True"/>
                </group>
            </group>
        </field>
    </record>

</odoo>

```

