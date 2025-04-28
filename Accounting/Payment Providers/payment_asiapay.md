# Odoo Module: payment_asiapay

Category: Accounting/Payment Providers

This file contains the source code of the Odoo module.

## File: const.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

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


def post_init_hook(cr, registry):
    setup_provider(cr, registry, 'asiapay')


def uninstall_hook(cr, registry):
    reset_payment_provider(cr, registry, 'asiapay')

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
    'application': False,
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
       asiapay_currency_id = NULL,
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

from odoo import api, fields, models

from odoo.addons.payment_asiapay import const


class PaymentProvider(models.Model):
    _inherit = 'payment.provider'

    def _domain_asiapay_currency_id(self):
        currency_xmlids = [f'base.{key}' for key in const.CURRENCY_MAPPING]
        return [('id', 'in', [self.env.ref(xmlid).id for xmlid in currency_xmlids])]

    code = fields.Selection(
        selection_add=[('asiapay', "AsiaPay")], ondelete={'asiapay': 'set default'}
    )
    asiapay_merchant_id = fields.Char(
        string="AsiaPay Merchant ID",
        help="The Merchant ID solely used to identify your AsiaPay account.",
        required_if_provider='asiapay',
    )
    asiapay_currency_id = fields.Many2one(
        string="AsiaPay Currency",
        help="The currency associated to your AsiaPay account.",
        comodel_name='res.currency',
        domain=_domain_asiapay_currency_id,
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

    # === BUSINESS METHODS ===#

    @api.model
    def _get_compatible_providers(self, *args, currency_id=None, **kwargs):
        """ Override of `payment` to filter out AsiaPay providers for unsupported currencies. """
        providers = super()._get_compatible_providers(*args, currency_id=currency_id, **kwargs)

        currency = self.env['res.currency'].browse(currency_id).exists()
        if currency:
            providers = providers.filtered(
                lambda p: p.code != 'asiapay' or currency == p.asiapay_currency_id
            )

        return providers

    def _asiapay_get_api_url(self):
        """ Return the URL of the API corresponding to the provider's state.

        :return: The API URL.
        :rtype: str
        """
        self.ensure_one()

        if self.state == 'enabled':
            return 'https://www.paydollar.com/b2c2/eng/payment/payForm.jsp'
        else:  # 'test'
            return 'https://test.paydollar.com/b2cDemo/eng/payment/payForm.jsp'

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
            'currency_code': const.CURRENCY_MAPPING[self.provider_id.asiapay_currency_id.name],
            'mps_mode': 'SCP',
            'return_url': urls.url_join(base_url, AsiaPayController._return_url),
            'payment_type': 'N',
            'language': get_language_code(lang),
            'payment_method': 'ALL',
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

        self.provider_reference = notification_data.get('PayRef')

        success_code = notification_data.get('successcode')
        primary_response_code = notification_data.get('prc')
        if not success_code:
            raise ValidationError("AsiaPay: " + _("Received data with missing success code."))

        if success_code in const.SUCCESS_CODE_MAPPING['done']:
            self._set_done()
        elif success_code in const.SUCCESS_CODE_MAPPING['error']:
            self._set_error(_(
                "An error occurred during the processing of your payment (success code %s; primary "
                "response code %s). Please try again.", success_code, primary_response_code
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
<svg width="70" height="70" viewBox="0 0 70 70" fill="none" xmlns="http://www.w3.org/2000/svg">
<mask id="mask0_3_167" style="mask-type:alpha" maskUnits="userSpaceOnUse" x="0" y="0" width="70" height="70">
<path d="M4 0H65C69 0 70 1 70 5V65C70 69 69 70 65 70H4C1 70 0 69 0 65V5C0 1 1 0 4 0Z" fill="white"/>
</mask>
<g mask="url(#mask0_3_167)">
<path fill-rule="evenodd" clip-rule="evenodd" d="M0 0H70V70H0V0Z" fill="url(#paint0_linear_3_167)"/>
<path fill-rule="evenodd" clip-rule="evenodd" d="M4 1H65C67.667 1 69.333 1.667 70 3V0H0V3C0.667 1.667 2 1 4 1Z" fill="white" fill-opacity="0.383"/>
<path fill-rule="evenodd" clip-rule="evenodd" d="M4 69H65C67.667 69 69.333 68 70 66V70H0V66C0.667 68 2 69 4 69Z" fill="black" fill-opacity="0.383"/>
</g>
<path opacity="0.324" fill-rule="evenodd" clip-rule="evenodd" d="M4 69C2 69 0 68 0 65V33.916C0 33.916 17 21.5 18.5 21C20 20.5 23 20.5 23 20.5L28.7087 22.703H37.25L48 23L51.576 24.968L51.34 46.978L39.224 69H4Z" fill="#393939"/>
<path d="M26.3156 37.6268C26.2605 38.8771 23.8758 39.3985 21.8287 39.3985C19.8212 39.3985 18.954 37.7879 18.954 36.2127C18.954 32.0789 24.5867 32.0789 26.3156 32.0415V37.6268ZM29.3105 27.5129C29.3105 24.2042 27.7743 22 23.2458 22C20.7247 22 17.1041 23.1389 17.1041 23.8109C17.1041 24.0066 17.1438 24.1234 17.2242 24.3204L17.7343 25.6986C17.7755 25.8197 17.8913 25.8962 18.0119 25.8962C18.3631 25.8962 20.1774 24.6392 22.8117 24.6392C25.6093 24.6392 26.3157 25.8962 26.3157 28.1833V29.5979C23.8759 29.6769 16 29.7955 16 36.4483C16 39.9537 18.0119 42 21.118 42C25.7194 42 29.3105 41.2544 29.3105 35.901V27.5129Z" fill="#465C68"/>
<path d="M26.3156 35.6268C26.2605 36.8771 23.8758 37.3985 21.8287 37.3985C19.8212 37.3985 18.954 35.7879 18.954 34.2127C18.954 30.0789 24.5867 30.0789 26.3156 30.0415V35.6268ZM29.3105 25.5129C29.3105 22.2042 27.7743 20 23.2458 20C20.7247 20 17.1041 21.1389 17.1041 21.8109C17.1041 22.0066 17.1438 22.1234 17.2242 22.3204L17.7343 23.6986C17.7755 23.8197 17.8913 23.8962 18.0119 23.8962C18.3631 23.8962 20.1774 22.6392 22.8117 22.6392C25.6093 22.6392 26.3157 23.8962 26.3157 26.1833V27.5979C23.8759 27.6769 16 27.7955 16 34.4483C16 37.9537 18.0119 40 21.118 40C25.7194 40 29.3105 39.2544 29.3105 33.901V25.5129Z" fill="white"/>
<path d="M19.2568 45L19.25 47.555C19.25 49.069 20.464 50.297 21.964 50.297H49.036C50.537 50.297 51.75 49.069 51.75 47.555V27.445C51.75 25.931 50.536 24.703 49.036 24.703H32V27.445H48.697C48.7874 27.4458 48.8737 27.4823 48.9372 27.5466C49.0008 27.6108 49.0363 27.6976 49.036 27.788V31H32V37.5H49.036V47.212C49.0363 47.3024 49.0008 47.3892 48.9372 47.4534C48.8737 47.5177 48.7874 47.5542 48.697 47.555H22.303C22.116 47.555 21.964 47.4 21.964 47.212V45H19.2568Z" fill="#465C68"/>
<path d="M14.423 55.642H15.459V58.474H16.219V55.642H17.256V55H14.423V55.642Z" fill="#465C68"/>
<path d="M18.492 55H17.422V58.474H18.133V56.036H18.143L18.991 58.474H19.577L20.424 56.012H20.434V58.474H21.145V55H20.075L19.311 57.389H19.301L18.492 55Z" fill="#465C68"/>
<path d="M19.2568 43L19.25 45.555C19.25 47.069 20.464 48.297 21.964 48.297H49.036C50.537 48.297 51.75 47.069 51.75 45.555V25.445C51.75 23.931 50.536 22.703 49.036 22.703H32V25.445H48.697C48.7874 25.4458 48.8737 25.4823 48.9372 25.5466C49.0008 25.6108 49.0363 25.6976 49.036 25.788V29H32V35.5H49.036V45.212C49.0363 45.3024 49.0008 45.3892 48.9372 45.4534C48.8737 45.5177 48.7874 45.5542 48.697 45.555H22.303C22.116 45.555 21.964 45.4 21.964 45.212V43H19.2568Z" fill="white"/>
<path d="M14.423 53.642H15.459V56.474H16.219V53.642H17.256V53H14.423V53.642Z" fill="white"/>
<path d="M18.492 53H17.422V56.474H18.133V54.036H18.143L18.991 56.474H19.577L20.424 54.012H20.434V56.474H21.145V53H20.075L19.311 55.389H19.301L18.492 53Z" fill="white"/>
<defs>
<linearGradient id="paint0_linear_3_167" x1="70" y1="0" x2="0" y2="70" gradientUnits="userSpaceOnUse">
<stop stop-color="#CDC484"/>
<stop offset="1" stop-color="#B5AA59"/>
</linearGradient>
</defs>
</svg>

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
                <group attrs="{'invisible': [('code', '!=', 'asiapay')]}">
                    <field name="asiapay_merchant_id"
                           string="Merchant ID"
                           attrs="{'required': [('code', '=', 'asiapay'), ('state', '!=', 'disabled')]}"/>
                    <field name="asiapay_currency_id"
                           string="Currency"
                           attrs="{'required': [('code', '=', 'asiapay'), ('state', '!=', 'disabled')]}"/>
                    <field name="asiapay_secure_hash_secret"
                           string="Secure Hash Secret"
                           attrs="{'required': [('code', '=', 'asiapay'), ('state', '!=', 'disabled')]}"
                           password="True"/>
                    <field name="asiapay_secure_hash_function"
                           string="Secure Hash Function"
                           attrs="{'required': [('code', '=', 'asiapay'), ('state', '!=', 'disabled')]}"
                           groups="base.group_no_one"/>
                </group>
            </group>
        </field>
    </record>

</odoo>

```

