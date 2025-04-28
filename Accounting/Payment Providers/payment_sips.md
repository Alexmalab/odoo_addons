# Odoo Module: payment_sips

Category: Accounting/Payment Providers

This file contains the source code of the Odoo module.

## File: const.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# ISO 4217 Data for currencies supported by sips
# NOTE: these are listed on the Atos Wordline SIPS POST documentation page
# at https://documentation.sips.worldline.com/en/WLSIPS.001-GD-Data-dictionary.html#Sips.001_DD_en-Value-currencyCode
# Last seen on: 22 September 2022.
SUPPORTED_CURRENCIES = {
    'ARS': '032',
    'AUD': '036',
    'BHD': '048',
    'KHR': '116',
    'CAD': '124',
    'LKR': '144',
    'CNY': '156',
    'HRK': '191',
    'CZK': '203',
    'DKK': '208',
    'HKD': '344',
    'HUF': '348',
    'ISK': '352',
    'INR': '356',
    'ILS': '376',
    'JPY': '392',
    'KRW': '410',
    'KWD': '414',
    'MYR': '458',
    'MUR': '480',
    'MXN': '484',
    'NPR': '524',
    'NZD': '554',
    'NOK': '578',
    'QAR': '634',
    'RUB': '643',
    'SAR': '682',
    'SGD': '702',
    'ZAR': '710',
    'SEK': '752',
    'CHF': '756',
    'THB': '764',
    'AED': '784',
    'TND': '788',
    'GBP': '826',
    'USD': '840',
    'TWD': '901',
    'RSD': '941',
    'RON': '946',
    'TRY': '949',
    'XOF': '952',
    'XPF': '953',
    'BGN': '975',
    'EUR': '978',
    'UAH': '980',
    'PLN': '985',
    'BRL': '986',
}

# Mapping of transaction states to Sips response codes.
# See https://documentation.sips.worldline.com/en/WLSIPS.001-GD-Data-dictionary.html#Sips.001_DD_en-Value-currencyCode
RESPONSE_CODES_MAPPING = {
    'pending': ('60',),
    'done': ('00',),
    'cancel': (
        '03', '05', '12', '14', '17', '24', '25', '30', '34', '40', '51', '54', '63', '75', '90',
        '94', '97', '99'
    ),
}

# The codes of the payment methods to activate when Sips is activated.
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

from . import models
from . import controllers

from odoo.addons.payment import setup_provider, reset_payment_provider


def post_init_hook(env):
    setup_provider(env, 'sips')


def uninstall_hook(env):
    reset_payment_provider(env, 'sips')

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
# Original Copyright 2015 Eezee-It, modified and maintained by Odoo.

{
    'name': 'Payment Provider: Worldline SIPS',
    'version': '2.0',
    'category': 'Accounting/Payment Providers',
    'sequence': 350,
    'summary': "A French payment provider for online payments all over the world.",
    'description': " ",  # Non-empty string to avoid loading the README file.
    'depends': ['payment'],
    'data': [
        'views/payment_provider_views.xml',
        'views/payment_sips_templates.xml',

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
# Original Copyright 2015 Eezee-It, modified and maintained by Odoo.

import hmac
import logging
import pprint

from werkzeug.exceptions import Forbidden

from odoo import http
from odoo.exceptions import ValidationError
from odoo.http import request

_logger = logging.getLogger(__name__)


class SipsController(http.Controller):
    _return_url = '/payment/sips/return/'
    _webhook_url = '/payment/sips/webhook/'

    @http.route(
        _return_url, type='http', auth='public', methods=['POST'], csrf=False, save_session=False
    )
    def sips_return_from_checkout(self, **data):
        """ Process the notification data sent by SIPS after redirection from checkout.

        The route is flagged with `save_session=False` to prevent Odoo from assigning a new session
        to the user if they are redirected to this route with a POST request. Indeed, as the session
        cookie is created without a `SameSite` attribute, some browsers that don't implement the
        recommended default `SameSite=Lax` behavior will not include the cookie in the redirection
        request from the payment provider to Odoo. As the redirection to the '/payment/status' page
        will satisfy any specification of the `SameSite` attribute, the session of the user will be
        retrieved and with it the transaction which will be immediately post-processed.

        :param dict data: The notification data
        """
        _logger.info("handling redirection from SIPS with data:\n%s", pprint.pformat(data))

        # Check the integrity of the notification
        tx_sudo = request.env['payment.transaction'].sudo()._get_tx_from_notification_data(
            'sips', data
        )
        self._verify_notification_signature(data, tx_sudo)

        # Handle the notification data
        tx_sudo._handle_notification_data('sips', data)
        return request.redirect('/payment/status')

    @http.route(_webhook_url, type='http', auth='public', methods=['POST'], csrf=False)
    def sips_webhook(self, **data):
        """ Process the notification data sent by SIPS to the webhook.

        :param dict data: The notification data
        :return: An empty string to acknowledge the notification
        :rtype: str
        """
        _logger.info("notification received from SIPS with data:\n%s", pprint.pformat(data))
        try:
            # Check the integrity of the notification
            tx_sudo = request.env['payment.transaction'].sudo()._get_tx_from_notification_data(
                'sips', data
            )
            self._verify_notification_signature(data, tx_sudo)

            # Handle the notification data
            tx_sudo._handle_notification_data('sips', data)
        except ValidationError:
            _logger.exception("unable to handle the notification data; skipping to acknowledge")
        return ''

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
        received_signature = notification_data.get('Seal')
        if not received_signature:
            _logger.warning("received notification with missing signature")
            raise Forbidden()

        # Compare the received signature with the expected signature computed from the data
        expected_signature = tx_sudo.provider_id._sips_generate_shasign(notification_data['Data'])
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
-- disable sips payment provider
UPDATE payment_provider
   SET sips_merchant_id = NULL,
       sips_secret = NULL;

```

## File: data\payment_provider_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="payment.payment_provider_sips" model="payment.provider">
        <field name="code">sips</field>
        <field name="redirect_form_view_id" ref="redirect_form"/>
    </record>

</odoo>

```

## File: models\payment_provider.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
# Original Copyright 2015 Eezee-It, modified and maintained by Odoo.

from hashlib import sha256

from odoo import fields, models

from odoo.addons.payment_sips import const


class PaymentProvider(models.Model):
    _inherit = 'payment.provider'

    code = fields.Selection(
        selection_add=[('sips', "Sips")], ondelete={'sips': 'set default'})
    sips_merchant_id = fields.Char(
        string="Merchant ID", help="The ID solely used to identify the merchant account with Sips",
        required_if_provider='sips')
    sips_secret = fields.Char(
        string="SIPS Secret Key", size=64, required_if_provider='sips', groups='base.group_system')
    sips_key_version = fields.Integer(
        string="Secret Key Version", required_if_provider='sips', default=2)
    sips_test_url = fields.Char(
        string="Test URL", required_if_provider='sips',
        default="https://payment-webinit.simu.sips-services.com/paymentInit")
    sips_prod_url = fields.Char(
        string="Production URL", required_if_provider='sips',
        default="https://payment-webinit.sips-services.com/paymentInit")
    sips_version = fields.Char(
        string="Interface Version", required_if_provider='sips', default="HP_2.31")

    def _get_supported_currencies(self):
        """ Override of `payment` to return the supported currencies. """
        supported_currencies = super()._get_supported_currencies()
        if self.code == 'sips':
            supported_currencies = supported_currencies.filtered(
                lambda c: c.name in const.SUPPORTED_CURRENCIES.keys()
            )
        return supported_currencies

    def _sips_generate_shasign(self, data):
        """ Generate the shasign for incoming or outgoing communications.

        Note: self.ensure_one()

        :param str data: The data to use to generate the shasign
        :return: shasign
        :rtype: str
        """
        self.ensure_one()

        key = self.sips_secret
        shasign = sha256((data + key).encode('utf-8'))
        return shasign.hexdigest()

    # === BUSINESS METHODS ===#

    def _get_default_payment_method_codes(self):
        """ Override of `payment` to return the default payment method codes. """
        default_codes = super()._get_default_payment_method_codes()
        if self.code != 'sips':
            return default_codes
        return const.DEFAULT_PAYMENT_METHODS_CODES

```

## File: models\payment_transaction.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
# Original Copyright 2015 Eezee-It, modified and maintained by Odoo.

import json
import logging

from werkzeug import urls

from odoo import _, api, models
from odoo.exceptions import ValidationError

from odoo.addons.payment import utils as payment_utils
from odoo.addons.payment_sips.const import RESPONSE_CODES_MAPPING, SUPPORTED_CURRENCIES
from odoo.addons.payment_sips.controllers.main import SipsController

_logger = logging.getLogger(__name__)


class PaymentTransaction(models.Model):
    _inherit = 'payment.transaction'

    @api.model
    def _compute_reference(self, provider_code, prefix=None, separator='-', **kwargs):
        """ Override of payment to ensure that Sips requirements for references are satisfied.

        Sips requirements for transaction are as follows:
        - References can only be made of alphanumeric characters.
          This is satisfied by forcing the custom separator to 'x' to ensure that no '-' character
          will be used to append a suffix. Additionally, the prefix is sanitized if it was provided,
          and generated with 'tx' as default otherwise. This prevents the prefix to be generated
          based on document names that may contain non-alphanum characters (eg: INV/2020/...).
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
        if provider_code == 'sips':
            # We use an empty separator for cosmetic reasons: As the default prefix is 'tx', we want
            # the singularized prefix to look like 'tx2020...' and not 'txx2020...'.
            prefix = payment_utils.singularize_reference_prefix(separator='')
            separator = 'x'  # Still, we need a dedicated separator between the prefix and the seq.
        return super()._compute_reference(provider_code, prefix=prefix, separator=separator, **kwargs)

    def _get_specific_rendering_values(self, processing_values):
        """ Override of payment to return Sips-specific rendering values.

        Note: self.ensure_one() from `_get_processing_values`

        :param dict processing_values: The generic and specific processing values of the transaction
        :return: The dict of provider-specific processing values
        :rtype: dict
        """
        res = super()._get_specific_rendering_values(processing_values)
        if self.provider_code != 'sips':
            return res

        base_url = self.get_base_url()
        data = {
            'amount': payment_utils.to_minor_currency_units(self.amount, self.currency_id),
            'currencyCode': SUPPORTED_CURRENCIES[self.currency_id.name],  # The ISO 4217 code
            'merchantId': self.provider_id.sips_merchant_id,
            'normalReturnUrl': urls.url_join(base_url, SipsController._return_url),
            'automaticResponseUrl': urls.url_join(base_url, SipsController._webhook_url),
            'transactionReference': self.reference,
            'statementReference': self.reference,
            'keyVersion': self.provider_id.sips_key_version,
            'returnContext': json.dumps(dict(reference=self.reference)),
        }
        api_url = self.provider_id.sips_prod_url if self.provider_id.state == 'enabled' \
            else self.provider_id.sips_test_url
        data = '|'.join([f'{k}={v}' for k, v in data.items()])
        return {
            'api_url': api_url,
            'Data': data,
            'InterfaceVersion': self.provider_id.sips_version,
            'Seal': self.provider_id._sips_generate_shasign(data),
        }

    def _get_tx_from_notification_data(self, provider_code, notification_data):
        """ Override of payment to find the transaction based on Sips data.

        :param str provider_code: The code of the provider that handled the transaction
        :param dict notification_data: The notification data sent by the provider
        :return: The transaction if found
        :rtype: recordset of `payment.transaction`
        :raise: ValidationError if the data match no transaction
        """
        tx = super()._get_tx_from_notification_data(provider_code, notification_data)
        if provider_code != 'sips' or len(tx) == 1:
            return tx

        data = self._sips_notification_data_to_object(notification_data['Data'])
        reference = data.get('transactionReference')

        if not reference:
            return_context = json.loads(data.get('returnContext', '{}'))
            reference = return_context.get('reference')

        tx = self.search([('reference', '=', reference), ('provider_code', '=', 'sips')])
        if not tx:
            raise ValidationError(
                "Sips: " + _("No transaction found matching reference %s.", reference)
            )

        return tx

    def _process_notification_data(self, notification_data):
        """ Override of payment to process the transaction based on Sips data.

        Note: self.ensure_one()

        :param dict notification_data: The notification data sent by the provider
        :return: None
        """
        super()._process_notification_data(notification_data)
        if self.provider_code != 'sips':
            return

        data = self._sips_notification_data_to_object(notification_data.get('Data'))

        # Update the provider reference.
        self.provider_reference = data.get('transactionReference')

        # Update the payment method.
        payment_method_type = notification_data.get('paymentMeanBrand', '').lower()
        payment_method = self.env['payment.method']._get_from_code(payment_method_type)
        self.payment_method_id = payment_method or self.payment_method_id

        # Update the payment state.
        response_code = data.get('responseCode')
        if response_code in RESPONSE_CODES_MAPPING['pending']:
            status = "pending"
            self._set_pending()
        elif response_code in RESPONSE_CODES_MAPPING['done']:
            status = "done"
            self._set_done()
        elif response_code in RESPONSE_CODES_MAPPING['cancel']:
            status = "cancel"
            self._set_canceled()
        else:
            status = "error"
            self._set_error(_("Unrecognized response received from the payment provider."))
        _logger.info(
            "received data with response %(response)s for transaction with reference %(ref)s, set "
            "status as '%(status)s'",
            {
                'response': response_code,
                'ref': self.reference,
                'status': status,
            },
        )

    def _sips_notification_data_to_object(self, data):
        res = {}
        for element in data.split('|'):
            key, value = element.split('=', 1)
            res[key] = value
        return res

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import payment_provider
from . import payment_transaction

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M43.105 4h.972V.818h1.108V0H42v.818h1.105V4Zm2.775 0h.858V1.453h.053L47.663 4h.555l.871-2.547h.056V4H50V0h-1.108l-.924 2.714h-.05L46.99 0h-1.11v4Z" fill="#D1D5DB"/><path d="M8.5 15.976a25.287 25.287 0 0 0-2.51 2.576c3.759 3.831 6.471 9.677 7.384 16.446h3.616c-.729-5.964-2.747-11.473-5.862-15.866A25.981 25.981 0 0 0 8.5 15.976ZM4.094 21.11a25.45 25.45 0 0 0-1.843 3.373C4.403 27.205 5.987 30.854 6.709 35h3.63c-.686-4.454-2.268-8.557-4.628-11.878a21.782 21.782 0 0 0-1.617-2.013ZM0 35h3.64c-.524-2.525-1.437-4.874-2.698-6.908A25.664 25.664 0 0 0 0 35Zm39.508-20.525a24.706 24.706 0 0 0-1.452-.986c-.226.24-.45.48-.67.728-2.898 3.264-6.18 8.26-7.92 15.29a51.47 51.47 0 0 0-2.233-8.181c1.886-4.284 4.23-7.595 6.39-10.042a25.325 25.325 0 0 0-1.775-.594 40.475 40.475 0 0 0-5.487 8.422c-1.214-2.861-2.686-5.532-4.4-7.955-.28-.396-.564-.78-.853-1.157a24.419 24.419 0 0 0-3.864.933c2.992 3.512 5.433 7.964 7.105 13.04a45.547 45.547 0 0 0-1.565 5.779c-1.178-5.448-3.296-10.449-6.24-14.607a31.325 31.325 0 0 0-2.376-2.967 24.84 24.84 0 0 0-3.191 1.858c4.729 4.857 8.08 12.34 9.027 20.964h3.745c.28-2.923.794-5.6 1.475-8.043A51.214 51.214 0 0 1 26.616 35h3.618c1.07-9.432 5.028-15.783 8.43-19.615.28-.315.561-.617.843-.91Zm4.697 4.472a25.448 25.448 0 0 0-1.15-1.322c-3.842 3.993-7.055 9.786-8.058 17.373h1.738c1.026-7.365 4.18-12.37 6.905-15.437.187-.21.376-.414.565-.614ZM41.523 35h1.746c.823-4.566 2.675-7.912 4.479-10.22a24.954 24.954 0 0 0-.833-1.728c-2.503 2.967-4.55 6.95-5.392 11.947Zm8.127-4.09A18.584 18.584 0 0 0 48.11 35H50c0-1.393-.122-2.758-.351-4.09Z" fill="#33B6B4"/></svg>

```

## File: views\payment_provider_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_provider_form" model="ir.ui.view">
        <field name="name">Sips Provider Form</field>
        <field name="model">payment.provider</field>
        <field name="inherit_id" ref="payment.payment_provider_form"/>
        <field name="arch" type="xml">
            <group name="provider_credentials" position='inside'>
                <group invisible="code != 'sips'">
                    <field name="sips_merchant_id" required="code == 'sips' and state != 'disabled'"/>
                    <field name="sips_secret" string="Secret Key" required="code == 'sips' and state != 'disabled'" password="True"/>
                    <field name="sips_key_version" required="code == 'sips' and state != 'disabled'" />
                    <field name="sips_test_url" required="code == 'sips' and state != 'disabled'" groups='base.group_no_one'/>
                    <field name="sips_prod_url" required="code == 'sips' and state != 'disabled'" groups='base.group_no_one'/>
                    <field name="sips_version" required="code == 'sips' and state != 'disabled'" />
                </group>
            </group>
        </field>
    </record>

</odoo>

```

## File: views\payment_sips_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="redirect_form">
        <form t-att-action="api_url" method="post">
            <input type="hidden" name="Data" t-att-value="Data"/>
            <input type="hidden" name="InterfaceVersion" t-att-value="InterfaceVersion"/>
            <input type="hidden" name="Seal" t-att-value="Seal"/>
        </form>
    </template>

</odoo>

```

