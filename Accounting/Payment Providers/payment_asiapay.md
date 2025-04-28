# Odoo Module: payment_asiapay

Category: Accounting/Payment Providers

This file contains the source code of the Odoo module.

## File: const.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

API_URLS = {
    'production': {
        'pesopay': 'https://www.pesopay.com/b2c2/eng/payment/payForm.jsp',
        'siampay': 'https://www.siampay.com/b2c2/eng/payment/payForm.jsp',
        'bimopay': 'https://www.bimopay.com/b2c2/eng/payment/payForm.jsp',
        'paydollar': 'https://www.paydollar.com/b2c2/eng/payment/payForm.jsp',
    },
    'test': {
        'pesopay': 'https://test.pesopay.com/b2cDemo/eng/payment/payForm.jsp',
        'siampay': 'https://test.siampay.com/b2cDemo/eng/payment/payForm.jsp',
        'paydollar': 'https://test.paydollar.com/b2cDemo/eng/payment/payForm.jsp',
    }
}

# Mapping of currency ISO 4217 codes AsiaPay's currency codes.
# See https://www.paydollar.com/pdf/op/enpdintguide.pdf for the list of currency codes.
CURRENCY_MAPPING = {
    'AED': '784',
    'AUD': '036',
    'BND': '096',
    'CAD': '124',
    'CNY': '156',
    'EUR': '978',
    'GBP': '826',
    'HKD': '344',
    'IDR': '360',
    'INR': '356',
    'JPY': '392',
    'KRW': '410',
    'MOP': '446',
    'MYR': '458',
    'NZD': '554',
    'PHP': '608',
    'SAR': '682',
    'SGD': '702',
    'THB': '764',
    'TWD': '901',
    'USD': '840',
    'VND': '704',
}

# Mapping of both country codes (e.g., 'es') and IETF language tags (e.g.: 'fr-BE') to AsiaPay
# language codes. If a language tag is not listed, the country code prefix can serve as fallback.
LANGUAGE_CODES_MAPPING = {
    'en': 'E',
    'zh_HK': 'C',
    'zh_TW': 'C',
    'zh_CN': 'X',
    'ja_JP': 'J',
    'th_TH': 'T',
    'fr': 'F',
    'de': 'G',
    'ru_RU': 'R',
    'es': 'S',
    'vi_VN': 'S',
}

# The codes of the payment methods to activate when Asiapay is activated.
DEFAULT_PAYMENT_METHOD_CODES = {
    # Primary payment methods.
    'card',
    # Brand payment methods.
    'visa',
    'mastercard',
    'amex',
    'discover',
}

# Mapping of payment method codes to AsiaPay codes.
PAYMENT_METHODS_MAPPING = {
    'alipay': 'ALIPAY',
    'alipay_hk': 'ALIPAYHKONL',
    'amex': 'AMEX',
    'apple_pay': 'APPLEPAY',
    'atome': 'ATOME',
    'bangkok_bank': 'IBANKING',
    'bpi': 'BPI',
    'boa': 'KRUNGSRIONLINE',
    'card': 'CC',
    'cimb': 'CIMBCLICK',
    'dana': 'DANA',
    'ditnow': 'DuitNow',
    'diners': 'Diners',
    'enets': 'ENETS',
    'enetsbanking': 'ENETSBANKING',
    'enetsqr': 'ENETSQR',
    'eximbay': 'Eximbay',
    'fps': 'FPS',
    'gcash': 'GCash',
    'google_pay': 'GOOGLE',
    'hoolah': 'HOOLAH',
    'humm': 'humm',
    'jkopay': 'JKOPAY',
    'jcs': 'JCB',
    'kungthai_bank': 'KTB',
    'linepay': 'LINEPAY',
    'maya': 'PayMaya',
    'mastercard': 'Master',
    'masterpass': 'MP',
    'maybank': 'M2U',
    'momo': 'MOMOPAY',
    'octopus': 'OCTOPUS',
    'ovo': 'OVO',
    'pace': 'Pace',
    'pay_id': 'PAYID',
    'paymaya': 'PayMaya',
    'payme': 'PayMe',
    'payu': 'PAYU',
    'poli': 'POLI',
    'qris': 'QRIS',
    'samsung_pay': 'SAMSUNG',
    'shopeepay': 'SHOPEEPAY',
    'tendopay': 'TendoPay',
    'touch_n_go': 'TouchnGo',
    'truemoney': 'TRUEMONEY',
    'ttb': 'TMB',
    'unionpay': 'CHINAPAY',
    'visa': 'VISA',
    'wechat_pay': 'WECHATONL',
    'zip': 'ZIPPAY',
    'zippay': 'ZIPPAY',
    'tenpay': 'TENPAY',
    'welend': 'WELEND',
    'tmb': 'TMB',
}

# The keys of the values to use in the calculation of the signature.
SIGNATURE_KEYS = {
    'outgoing': [
        'merchant_id',
        'reference',
        'currency_code',
        'amount',
        'payment_type',
    ],
    'incoming': [
        'src',
        'prc',
        'successcode',
        'Ref',
        'PayRef',
        'Cur',
        'Amt',
        'payerAuth',
    ],
}

# Mapping of transaction states to AsiaPay success codes.
SUCCESS_CODE_MAPPING = {
    'done': ('0',),
    'error': ('1',),
}

```

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

from odoo.addons.payment import setup_provider, reset_payment_provider


def post_init_hook(env):
    setup_provider(env, 'asiapay')


def uninstall_hook(env):
    reset_payment_provider(env, 'asiapay')

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': "Payment Provider: AsiaPay",
    'version': '1.0',
    'category': 'Accounting/Payment Providers',
    'sequence': 350,
    'summary': "An payment provider based in Hong Kong covering most Asian countries.",
    'description': " ",  # Non-empty string to avoid loading the README file.
    'depends': ['payment'],
    'data': [
        'views/payment_asiapay_templates.xml',
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


class AsiaPayController(http.Controller):
    _return_url = '/payment/asiapay/return'
    _webhook_url = '/payment/asiapay/webhook'

    @http.route(_return_url, type='http', auth='public', methods=['GET'])
    def asiapay_return_from_checkout(self, **data):
        """ Process the notification data sent by AsiaPay after redirection.

        :param dict data: The notification data.
        """
        # Don't process the notification data as they contain no valuable information except for the
        # reference and AsiaPay doesn't expose an endpoint to fetch the data from the API.
        return request.redirect('/payment/status')

    @http.route(_webhook_url, type='http', auth='public', methods=['POST'], csrf=False)
    def asiapay_webhook(self, **data):
        """ Process the notification data sent by AsiaPay to the webhook.

        :param dict data: The notification data.
        :return: The 'OK' string to acknowledge the notification.
        :rtype: str
        """
        _logger.info("Notification received from AsiaPay with data:\n%s", pprint.pformat(data))
        try:
            # Check the integrity of the notification data.
            tx_sudo = request.env['payment.transaction'].sudo()._get_tx_from_notification_data(
                'asiapay', data
            )
            self._verify_notification_signature(data, tx_sudo)

            # Handle the notification data.
            tx_sudo._handle_notification_data('asiapay', data)
        except ValidationError:  # Acknowledge the notification to avoid getting spammed.
            _logger.exception("Unable to handle the notification data; skipping to acknowledge.")

        return 'OK'  # Acknowledge the notification.

    @staticmethod
    def _verify_notification_signature(notification_data, tx_sudo):
        """ Check that the received signature matches the expected one.

        :param dict notification_data: The notification data
        :param recordset tx_sudo: The sudoed transaction referenced by the notification data, as a
                                  `payment.transaction` record
        :return: None
        :raise: :class:`werkzeug.exceptions.Forbidden` if the signatures don't match
        """
        received_signature = notification_data.get('secureHash')
        if not received_signature:
            _logger.warning("Received notification with missing signature.")
            raise Forbidden()

        # Compare the received signature with the expected signature computed from the data.
        expected_signature = tx_sudo.provider_id._asiapay_calculate_signature(
            notification_data, incoming=True
        )
        if not hmac.compare_digest(received_signature, expected_signature):
            _logger.warning("Received notification with invalid signature.")
            raise Forbidden()

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\neutralize.sql

```sql
-- disable asiapay payment provider
UPDATE payment_provider
   SET asiapay_merchant_id = NULL,
       asiapay_secure_hash_secret = NULL,
       asiapay_secure_hash_function = NULL;

```

## File: data\payment_provider_data.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<odoo noupdate="1">

    <record id="payment.payment_provider_asiapay" model="payment.provider">
        <field name="code">asiapay</field>
        <field name="redirect_form_view_id" ref="redirect_form"/>
    </record>

</odoo>

```

## File: models\payment_provider.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from hashlib import new as hashnew

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError

from odoo.addons.payment_asiapay import const


class PaymentProvider(models.Model):
    _inherit = 'payment.provider'

    code = fields.Selection(
        selection_add=[('asiapay', "AsiaPay")], ondelete={'asiapay': 'set default'}
    )
    asiapay_brand = fields.Selection(
        string="Asiapay Brand",
        help="The brand associated to your AsiaPay account.",
        selection=[("paydollar", "PayDollar"), ("pesopay", "PesoPay"),
                    ("siampay", "SiamPay"), ("bimopay", "BimoPay")],
        default='paydollar',
        required_if_provider='asiapay',
    )
    asiapay_merchant_id = fields.Char(
        string="AsiaPay Merchant ID",
        help="The Merchant ID solely used to identify your AsiaPay account.",
        required_if_provider='asiapay',
    )
    asiapay_secure_hash_secret = fields.Char(
        string="AsiaPay Secure Hash Secret",
        required_if_provider='asiapay',
        groups='base.group_system',
    )
    asiapay_secure_hash_function = fields.Selection(
        string="AsiaPay Secure Hash Function",
        help="The secure hash function associated to your AsiaPay account.",
        selection=[('sha1', "SHA1"), ('sha256', "SHA256"), ('sha512', 'SHA512')],
        default='sha1',
        required_if_provider='asiapay',
    )

    # ==== CONSTRAINT METHODS ===#

    @api.constrains('available_currency_ids', 'state')
    def _limit_available_currency_ids(self):
        for provider in self.filtered(lambda p: p.code == 'asiapay'):
            if len(provider.available_currency_ids) > 1 and provider.state != 'disabled':
                raise ValidationError(_("Only one currency can be selected by AsiaPay account."))

    # === BUSINESS METHODS ===#

    def _asiapay_get_api_url(self):
        """ Return the URL of the API corresponding to the provider's state.

        :return: The API URL.
        :rtype: str
        """
        self.ensure_one()

        environment = 'production' if self.state == 'enabled' else 'test'
        api_urls = const.API_URLS[environment]
        return api_urls.get(self.asiapay_brand, api_urls['paydollar'])

    def _asiapay_calculate_signature(self, data, incoming=True):
        """ Compute the signature for the provided data according to the AsiaPay documentation.

        :param dict data: The data to sign.
        :param bool incoming: Whether the signature must be generated for an incoming (AsiaPay to
                              Odoo) or outgoing (Odoo to AsiaPay) communication.
        :return: The calculated signature.
        :rtype: str
        """
        signature_keys = const.SIGNATURE_KEYS['incoming' if incoming else 'outgoing']
        data_to_sign = [str(data[k]) for k in signature_keys] + [self.asiapay_secure_hash_secret]
        signing_string = '|'.join(data_to_sign)
        shasign = hashnew(self.asiapay_secure_hash_function)
        shasign.update(signing_string.encode())
        return shasign.hexdigest()

    def _get_default_payment_method_codes(self):
        """ Override of `payment` to return the default payment method codes. """
        default_codes = super()._get_default_payment_method_codes()
        if self.code != 'asiapay':
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
from odoo.addons.payment_asiapay import const
from odoo.addons.payment_asiapay.controllers.main import AsiaPayController


_logger = logging.getLogger(__name__)


class PaymentTransaction(models.Model):
    _inherit = 'payment.transaction'

    @api.model
    def _compute_reference(self, provider_code, prefix=None, separator='-', **kwargs):
        """ Override of `payment` to ensure that AsiaPay requirements for references are satisfied.

        AsiaPay requirements for references are as follows:
        - References must be unique at provider level for a given merchant account.
          This is satisfied by singularizing the prefix with the current datetime. If two
          transactions are created simultaneously, `_compute_reference` ensures the uniqueness of
          references by suffixing a sequence number.
        - References must be at most 35 characters long.

        :param str provider_code: The code of the provider handling the transaction.
        :param str prefix: The custom prefix used to compute the full reference.
        :param str separator: The custom separator used to separate the prefix from the suffix.
        :return: The unique reference for the transaction.
        :rtype: str
        """
        if provider_code != 'asiapay':
            return super()._compute_reference(provider_code, prefix=prefix, **kwargs)

        if not prefix:
            # If no prefix is provided, it could mean that a module has passed a kwarg intended for
            # the `_compute_reference_prefix` method, as it is only called if the prefix is empty.
            # We call it manually here because singularizing the prefix would generate a default
            # value if it was empty, hence preventing the method from ever being called and the
            # transaction from received a reference named after the related document.
            prefix = self.sudo()._compute_reference_prefix(provider_code, separator, **kwargs) or None
        prefix = payment_utils.singularize_reference_prefix(prefix=prefix, max_length=35)
        return super()._compute_reference(provider_code, prefix=prefix, **kwargs)

    def _get_specific_rendering_values(self, processing_values):
        """ Override of `payment` to return AsiaPay-specific rendering values.

        Note: self.ensure_one() from `_get_processing_values`.

        :param dict processing_values: The generic and specific processing values of the
                                       transaction.
        :return: The dict of provider-specific processing values.
        :rtype: dict
        """
        def get_language_code(lang_):
            """ Return the language code corresponding to the provided lang.

            If the lang is not mapped to any language code, the country code is used instead. In
            case the country code has no match either, we fall back to English.

            :param str lang_: The lang, in IETF language tag format.
            :return: The corresponding language code.
            :rtype: str
            """
            language_code_ = const.LANGUAGE_CODES_MAPPING.get(lang_)
            if not language_code_:
                country_code_ = lang_.split('_')[0]
                language_code_ = const.LANGUAGE_CODES_MAPPING.get(country_code_)
            if not language_code_:
                language_code_ = const.LANGUAGE_CODES_MAPPING['en']
            return language_code_

        res = super()._get_specific_rendering_values(processing_values)
        if self.provider_code != 'asiapay':
            return res

        base_url = self.provider_id.get_base_url()
        # The lang is taken from the context rather than from the partner because it is not required
        # to be logged in to make a payment, and because the lang is not always set on the partner.
        lang = self._context.get('lang') or 'en_US'
        rendering_values = {
            'merchant_id': self.provider_id.asiapay_merchant_id,
            'amount': self.amount,
            'reference': self.reference,
            'currency_code': const.CURRENCY_MAPPING[self.provider_id.available_currency_ids[0].name],
            'mps_mode': 'SCP',
            'return_url': urls.url_join(base_url, AsiaPayController._return_url),
            'payment_type': 'N',
            'language': get_language_code(lang),
            'payment_method': const.PAYMENT_METHODS_MAPPING.get(self.payment_method_id.code, 'ALL'),
        }
        rendering_values.update({
            'secure_hash': self.provider_id._asiapay_calculate_signature(
                rendering_values, incoming=False
            ),
            'api_url': self.provider_id._asiapay_get_api_url()
        })
        return rendering_values

    def _get_tx_from_notification_data(self, provider_code, notification_data):
        """ Override of `payment` to find the transaction based on AsiaPay data.

        :param str provider_code: The code of the provider that handled the transaction.
        :param dict notification_data: The notification data sent by the provider.
        :return: The transaction if found.
        :rtype: recordset of `payment.transaction`
        :raise ValidationError: If inconsistent data are received.
        :raise ValidationError: If the data match no transaction.
        """
        tx = super()._get_tx_from_notification_data(provider_code, notification_data)
        if provider_code != 'asiapay' or len(tx) == 1:
            return tx

        reference = notification_data.get('Ref')
        if not reference:
            raise ValidationError(
                "AsiaPay: " + _("Received data with missing reference %(ref)s.", ref=reference)
            )

        tx = self.search([('reference', '=', reference), ('provider_code', '=', 'asiapay')])
        if not tx:
            raise ValidationError(
                "AsiaPay: " + _("No transaction found matching reference %s.", reference)
            )

        return tx

    def _process_notification_data(self, notification_data):
        """ Override of `payment' to process the transaction based on AsiaPay data.

        Note: self.ensure_one()

        :param dict notification_data: The notification data sent by the provider.
        :return: None
        :raise ValidationError: If inconsistent data are received.
        """
        super()._process_notification_data(notification_data)
        if self.provider_code != 'asiapay':
            return

        # Update the provider reference.
        self.provider_reference = notification_data.get('PayRef')

        # Update the payment method.
        payment_method_code = notification_data.get('payMethod')
        payment_method = self.env['payment.method']._get_from_code(
            payment_method_code, mapping=const.PAYMENT_METHODS_MAPPING
        )
        self.payment_method_id = payment_method or self.payment_method_id

        # Update the payment state.
        success_code = notification_data.get('successcode')
        primary_response_code = notification_data.get('prc')
        if not success_code:
            raise ValidationError("AsiaPay: " + _("Received data with missing success code."))
        if success_code in const.SUCCESS_CODE_MAPPING['done']:
            self._set_done()
        elif success_code in const.SUCCESS_CODE_MAPPING['error']:
            self._set_error(_(
                "An error occurred during the processing of your payment (success code %(success_code)s; primary "
                "response code %(response_code)s). Please try again.", success_code=success_code, response_code=primary_response_code,
            ))
        else:
            _logger.warning(
                "Received data with invalid success code (%s) for transaction with primary response "
                "code %s and reference %s.", success_code, primary_response_code, self.reference
            )
            self._set_error("AsiaPay: " + _("Unknown success code: %s", success_code))

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import payment_provider
from . import payment_transaction

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M18.012 25.826h-.69a6.387 6.387 0 0 0-2.83.95c-1.323.693-1.323 2.407-1.323 3.043v12.223c0 .127 0 .313.126.313.184-.058.127 0 .311-.058l1.127-.313c.127 0 .184-.127.184-.255v-4.815c.564.382 1.254.764 2.393.822h.437c2.704 0 3.9-2.848 3.9-6.204.011-2.916-1.127-5.706-3.635-5.706Zm-.449 10.324h-.253c-1-.057-1.702-.382-2.393-.764v-6.4c0-.695.943-1.551 2.462-1.62.138-.024.195-.024.31-.024 1.887 0 2.014 2.28 2.014 4.248 0 2.28-.184 4.56-2.14 4.56Z" fill="#3C4187"/><path d="M29.343 11.254h-.38c-1.45.058-3.279.694-3.279 1.076 0 .127 0 .185.057.313l.311.764c.058.127.058.127.127.127.253 0 1.277-.764 2.795-.764h.104c1.633 0 2.082.764 2.082 2.153v.822c-.506 0-1.323.057-2.21.185-1.829.312-3.9 1.273-3.9 3.935 0 2.083 1.14 3.299 3.026 3.299h.886c1.45-.128 2.393-.44 2.646-.637 1.196-.764 1.323-2.222 1.323-3.414v-4.56c0-2.026-.943-3.3-3.588-3.3Zm1.817 9.445c0 .382-1.138.822-2.209.891h-.38c-1.196 0-1.829-.949-1.829-1.898 0-1.389 1.07-1.967 2.21-2.222.885-.255 1.76-.255 2.208-.255v3.484Z" fill="#F89C33"/><path d="M40.674 26.335c0-.184.069-.44-.127-.44l-1.254-.069c-.184 0-.31 0-.31.127l-.07 9.502c-.632.568-1.38.765-2.519.765-1.45.058-2.324-1.459-2.393-3.3-.126-2.349 0-4.432 0-6.782 0-.058-.253-.255-.437-.255h-1.139s-.253.313-.253.382c.057 1.262.506 9.121.817 9.688.126.185.38.567.632.891.254.255.564.44.944.695.817.509 3.393.312 4.406-.567l.057 1.076-.057 1.203c-.07 1.204-.07 3.171-1.887 3.23-1.196 0-1.76-.764-1.956-.127 0 .057-.184.949-.126 1.076.184.255.76.51 1.76.567 1.324.058 2.83-.636 3.28-1.586.759-1.516.689-5.069.689-6.4l-.057-9.676Z" fill="#3C4187"/><path d="M19.829 12.458c0-.127.057-.185.057-.255 0-.127-.057-.255-.253-.313-.633-.312-1.956-.694-2.773-.694-2.082 0-3.531 1.332-3.531 3.3 0 3.552 5.165 2.974 5.165 5.45 0 1.135-.817 1.713-1.83 1.713-1.633 0-2.83-.822-3.025-.822-.057 0-.195 0-.253.197l-.311.764c-.057.058-.057.127-.057.255 0 .44 2.392 1.145 3.52 1.145 2.013 0 3.773-1.145 3.773-3.356 0-3.669-5.223-2.917-5.223-5.51 0-1.133.817-1.585 1.76-1.585 1.197 0 2.21.636 2.324.636.127 0 .184-.069.311-.255l.346-.67Zm3.463-.567c0-.185-.058-.313-.311-.313h-1.196c-.184 0-.31.07-.31.256v10.764c0 .184.125.255.31.255h1.196c.184 0 .31-.07.31-.255V11.89Zm.379-4.502c0-.764-.633-1.389-1.38-1.389-.818 0-1.45.636-1.45 1.39 0 .821.62 1.457 1.45 1.457.747 0 1.38-.637 1.38-1.458Z" fill="#F89C33"/><path d="M26.996 25.919h-.38c-1.45.07-3.279.694-3.279 1.076 0 .127 0 .185.058.313l.31.764c.058.127.058.127.127.127.253 0 1.277-.764 2.796-.764h.103c1.633 0 2.082.764 2.082 2.153v.822c-.506 0-1.323.057-2.21.184-1.828.313-3.899 1.274-3.899 3.936 0 2.083 1.139 3.299 3.025 3.299h.886c1.45-.127 2.393-.452 2.646-.637 1.196-.764 1.323-2.222 1.323-3.426v-4.56c.012-2.014-.942-3.287-3.588-3.287Zm1.83 9.445c0 .382-1.14.822-2.21.892h-.38c-1.196 0-1.829-.95-1.829-1.899 0-1.389 1.07-1.967 2.21-2.222.885-.255 1.76-.255 2.208-.255v3.484Z" fill="#3C4187"/><path d="M8.291 11.254h-.38c-1.45.058-3.279.694-3.279 1.076 0 .127 0 .185.058.313l.31.764c.058.127.058.127.127.127.254 0 1.277-.764 2.796-.764h.103c1.633 0 2.07.764 2.07 2.153v.822c-.506 0-1.323.057-2.196.185-1.83.312-3.9 1.273-3.9 3.935 0 2.083 1.139 3.299 3.025 3.299h.886c1.45-.128 2.393-.44 2.646-.637 1.196-.764 1.323-2.222 1.323-3.414v-4.56c0-2.026-.943-3.3-3.589-3.3Zm1.83 9.445c0 .382-1.14.822-2.197.891h-.38c-1.196 0-1.83-.949-1.83-1.898 0-1.389 1.07-1.967 2.21-2.222.885-.255 1.76-.255 2.196-.255v3.484Z" fill="#F89C33"/><path d="M46 27.423c0 .51-.184.95-.54 1.309a1.776 1.776 0 0 1-1.3.543c-.507 0-.944-.184-1.301-.543a1.798 1.798 0 0 1-.54-1.309c0-.509.183-.949.54-1.308a1.776 1.776 0 0 1 1.3-.544c.506 0 .944.185 1.3.544.357.36.54.788.54 1.308Zm-.241 0c0-.451-.161-.822-.472-1.145a1.541 1.541 0 0 0-1.127-.475 1.54 1.54 0 0 0-1.127.475 1.567 1.567 0 0 0-.472 1.145c0 .452.161.822.472 1.146.31.313.69.475 1.127.475.437 0 .817-.15 1.127-.475a1.61 1.61 0 0 0 .472-1.146Zm-.542.938h-.483l-.598-.764h-.265v.764h-.357v-1.956h.598c.138 0 .242 0 .323.011a.55.55 0 0 1 .23.081c.08.047.149.104.183.174a.595.595 0 0 1 .058.266.508.508 0 0 1-.115.348.883.883 0 0 1-.311.22l.737.856Zm-.701-1.4c0-.058-.011-.093-.023-.14a.252.252 0 0 0-.092-.103c-.034-.023-.08-.035-.115-.046-.046-.012-.103-.012-.172-.012h-.241v.66h.207c.069 0 .126-.012.195-.024a.355.355 0 0 0 .15-.07c.034-.034.068-.068.08-.115.011-.023.011-.08.011-.15Z" fill="#3C4187"/></svg>

```

## File: views\payment_asiapay_templates.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<odoo>

    <template id="redirect_form">
        <form t-att-action="api_url" method="post">
            <input type="hidden" name="merchantId" t-att-value="merchant_id"/>
            <input type="hidden" name="amount" t-att-value="amount"/>
            <input type="hidden" name="orderRef" t-att-value="reference"/>
            <input type="hidden" name="currCode" t-att-value="currency_code"/>
            <input type="hidden" name="mpsMode" t-att-value="mps_mode"/>
            <input type="hidden" name="successUrl" t-att-value="return_url"/>
            <input type="hidden" name="failUrl" t-att-value="return_url"/>
            <input type="hidden" name="cancelUrl" t-att-value="return_url"/>
            <input type="hidden" name="payType" t-att-value="payment_type"/>
            <input type="hidden" name="lang" t-att-value="language"/>
            <input type="hidden" name="payMethod" t-att-value="payment_method"/>
            <input type="hidden" name="secureHash" t-att-value="secure_hash"/>
        </form>
    </template>

</odoo>

```

## File: views\payment_provider_views.xml

```xml
<?xml version='1.0' encoding='utf-8' ?>
<odoo>

    <record id="payment_provider_form" model="ir.ui.view">
        <field name="name">AsiaPay Provider Form</field>
        <field name="model">payment.provider</field>
        <field name="inherit_id" ref="payment.payment_provider_form"/>
        <field name="arch" type="xml">
            <group name="provider_credentials" position='inside'>
                <group invisible="code != 'asiapay'">
                    <field name="asiapay_brand"
                           string="Brand"
                           required="code == 'asiapay' and state != 'disabled'"/>
                    <field name="asiapay_merchant_id"
                           string="Merchant ID"
                           required="code == 'asiapay' and state != 'disabled'"/>
                    <field name="asiapay_secure_hash_secret"
                           string="Secure Hash Secret"
                           required="code == 'asiapay' and state != 'disabled'"
                           password="True"/>
                    <field name="asiapay_secure_hash_function"
                           string="Secure Hash Function"
                           required="code == 'asiapay' and state != 'disabled'"
                           groups="base.group_no_one"/>
                </group>
            </group>
            <field name="available_currency_ids" position="attributes">
                <attribute
                    name="required"
                    separator="or"
                    add="(code == 'asiapay' and state != 'disabled')"
                />
            </field>
        </field>
    </record>

</odoo>

```

