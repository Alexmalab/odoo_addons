# Odoo Module: payment_sips

Category: Accounting/Payment Acquirers

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import controllers

from odoo.addons.payment import reset_payment_acquirer


def uninstall_hook(cr, registry):
    reset_payment_acquirer(cr, registry, 'sips')

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
# Original Copyright 2015 Eezee-It, modified and maintained by Odoo.

{
    'name': 'Worldline SIPS',
    'version': '2.0',
    'category': 'Accounting/Payment Acquirers',
    'sequence': 385,
    'description': " ",  # Non-empty string to avoid loading the README file.
    'depends': ['payment'],
    'data': [
        'views/payment_views.xml',
        'views/payment_sips_templates.xml',
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
# Original Copyright 2015 Eezee-It, modified and maintained by Odoo.

import logging
import pprint

from odoo import http
from odoo.exceptions import ValidationError
from odoo.http import request

_logger = logging.getLogger(__name__)


class SipsController(http.Controller):
    _return_url = '/payment/sips/dpn/'
    _notify_url = '/payment/sips/ipn/'

    @http.route(
        _return_url, type='http', auth='public', methods=['POST'], csrf=False, save_session=False
    )
    def sips_dpn(self, **post):
        """ Process the data returned by SIPS after redirection.

        The route is flagged with `save_session=False` to prevent Odoo from assigning a new session
        to the user if they are redirected to this route with a POST request. Indeed, as the session
        cookie is created without a `SameSite` attribute, some browsers that don't implement the
        recommended default `SameSite=Lax` behavior will not include the cookie in the redirection
        request from the payment provider to Odoo. As the redirection to the '/payment/status' page
        will satisfy any specification of the `SameSite` attribute, the session of the user will be
        retrieved and with it the transaction which will be immediately post-processed.

        :param dict post: The feedback data to process
        """
        _logger.info("beginning Sips DPN _handle_feedback_data with data %s", pprint.pformat(post))
        try:
            if self._sips_validate_data(post):
                request.env['payment.transaction'].sudo()._handle_feedback_data('sips', post)
        except ValidationError:
            pass
        return request.redirect('/payment/status')

    @http.route(_notify_url, type='http', auth='public', methods=['POST'], csrf=False)
    def sips_ipn(self, **post):
        """ Sips IPN. """
        _logger.info("beginning Sips IPN _handle_feedback_data with data %s", pprint.pformat(post))
        if not post:
            # SIPS sometimes sends empty notifications, the reason why is unclear but they tend to
            # pollute logs and do not provide any meaningful information; log as a warning instead
            # of a traceback.
            _logger.warning("received empty notification; skip.")
        else:
            try:
                if self._sips_validate_data(post):
                    request.env['payment.transaction'].sudo()._handle_feedback_data('sips', post)
            except ValidationError:
                pass  # Acknowledge the notification to avoid getting spammed
        return ''

    def _sips_validate_data(self, post):
        tx_sudo = request.env['payment.transaction'].sudo()._get_tx_from_feedback_data('sips', post)
        acquirer_sudo = tx_sudo.acquirer_id
        security = acquirer_sudo._sips_generate_shasign(post['Data'])
        if security == post['Seal']:
            _logger.debug('validated data')
            return True
        else:
            _logger.warning('data are tampered')
            return False

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

    <record id="payment.payment_acquirer_sips" model="payment.acquirer">
        <field name="provider">sips</field>
        <field name="redirect_form_view_id" ref="redirect_form"/>
        <field name="support_authorization">False</field>
        <field name="support_fees_computation">False</field>
        <field name="support_refund"></field>
        <field name="support_tokenization">False</field>
    </record>

    <record id="payment_method_sips" model="account.payment.method">
        <field name="name">Sips</field>
        <field name="code">sips</field>
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
        res['sips'] = {'mode': 'electronic', 'domain': [('type', '=', 'bank')]}
        return res

```

## File: models\const.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# ISO 4217 Data for currencies supported by sips
# NOTE: these are listed on the Atos Wordline SIPS POST documentation page
# at https://documentation.sips.worldline.com/en/WLSIPS.001-GD-Data-dictionary.html#Sips.001_DD_en-Value-currencyCode
# Yet with the simu environment, some of these currencies are *not* working
# I have no way to know if this is caused by the SIMU environment, or if it's
# the doc of SIPS that lists currencies that don't work, but since this list is
# restrictive, I'm gonna assume they are supported when using the right flow
# and payment methods, which may not work in SIMU...
# Since SIPS advises to use 'in production', well...
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

```

## File: models\payment_acquirer.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
# Original Copyright 2015 Eezee-It, modified and maintained by Odoo.

from hashlib import sha256

from odoo import api, fields, models

from .const import SUPPORTED_CURRENCIES


class PaymentAcquirer(models.Model):
    _inherit = 'payment.acquirer'

    provider = fields.Selection(
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

    @api.model
    def _get_compatible_acquirers(self, *args, currency_id=None, **kwargs):
        """ Override of payment to unlist Sips acquirers when the currency is not supported. """
        acquirers = super()._get_compatible_acquirers(*args, currency_id=currency_id, **kwargs)

        currency = self.env['res.currency'].browse(currency_id).exists()
        if currency and currency.name not in SUPPORTED_CURRENCIES:
            acquirers = acquirers.filtered(lambda a: a.provider != 'sips')

        return acquirers

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

    def _get_default_payment_method_id(self):
        self.ensure_one()
        if self.provider != 'sips':
            return super()._get_default_payment_method_id()
        return self.env.ref('payment_sips.payment_method_sips').id

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
from odoo.addons.payment_sips.controllers.main import SipsController
from .const import RESPONSE_CODES_MAPPING, SUPPORTED_CURRENCIES

_logger = logging.getLogger(__name__)


class PaymentTransaction(models.Model):
    _inherit = 'payment.transaction'

    @api.model
    def _compute_reference(self, provider, prefix=None, separator='-', **kwargs):
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

        :param str provider: The provider of the acquirer handling the transaction
        :param str prefix: The custom prefix used to compute the full reference
        :param str separator: The custom separator used to separate the prefix from the suffix
        :return: The unique reference for the transaction
        :rtype: str
        """
        if provider == 'sips':
            # We use an empty separator for cosmetic reasons: As the default prefix is 'tx', we want
            # the singularized prefix to look like 'tx2020...' and not 'txx2020...'.
            prefix = payment_utils.singularize_reference_prefix(separator='')
            separator = 'x'  # Still, we need a dedicated separator between the prefix and the seq.
        return super()._compute_reference(provider, prefix=prefix, separator=separator, **kwargs)

    def _get_specific_rendering_values(self, processing_values):
        """ Override of payment to return Sips-specific rendering values.

        Note: self.ensure_one() from `_get_processing_values`

        :param dict processing_values: The generic and specific processing values of the transaction
        :return: The dict of acquirer-specific processing values
        :rtype: dict
        """
        res = super()._get_specific_rendering_values(processing_values)
        if self.provider != 'sips':
            return res

        base_url = self.get_base_url()
        data = {
            'amount': payment_utils.to_minor_currency_units(self.amount, self.currency_id),
            'currencyCode': SUPPORTED_CURRENCIES[self.currency_id.name],  # The ISO 4217 code
            'merchantId': self.acquirer_id.sips_merchant_id,
            'normalReturnUrl': urls.url_join(base_url, SipsController._return_url),
            'automaticResponseUrl': urls.url_join(base_url, SipsController._notify_url),
            'transactionReference': self.reference,
            'statementReference': self.reference,
            'keyVersion': self.acquirer_id.sips_key_version,
            'returnContext': json.dumps(dict(reference=self.reference)),
        }
        api_url = self.acquirer_id.sips_prod_url if self.acquirer_id.state == 'enabled' \
            else self.acquirer_id.sips_test_url
        data = '|'.join([f'{k}={v}' for k, v in data.items()])
        return {
            'api_url': api_url,
            'Data': data,
            'InterfaceVersion': self.acquirer_id.sips_version,
            'Seal': self.acquirer_id._sips_generate_shasign(data),
        }

    @api.model
    def _get_tx_from_feedback_data(self, provider, data):
        """ Override of payment to find the transaction based on Sips data.

        :param str provider: The provider of the acquirer that handled the transaction
        :param dict data: The feedback data sent by the provider
        :return: The transaction if found
        :rtype: recordset of `payment.transaction`
        :raise: ValidationError if the data match no transaction
        :raise: ValidationError if the currency is not supported
        :raise: ValidationError if the amount mismatch
        """
        tx = super()._get_tx_from_feedback_data(provider, data)
        if provider != 'sips':
            return tx

        data = self._sips_data_to_object(data['Data'])
        reference = data.get('transactionReference')

        if not reference:
            return_context = json.loads(data.get('returnContext', '{}'))
            reference = return_context.get('reference')

        tx = self.search([('reference', '=', reference), ('provider', '=', 'sips')])
        if not tx:
            raise ValidationError(
                "Sips: " + _("No transaction found matching reference %s.", reference)
            )

        sips_currency = SUPPORTED_CURRENCIES.get(tx.currency_id.name)
        if not sips_currency:
            raise ValidationError(
                "Sips: " + _("This currency is not supported: %s.", tx.currency_id.name)
            )

        amount_converted = payment_utils.to_major_currency_units(
            float(data.get('amount', '0.0')), tx.currency_id
        )
        if tx.currency_id.compare_amounts(amount_converted, tx.amount) != 0:
            raise ValidationError(
                "Sips: " + _(
                    "Incorrect amount: received %(received).2f, expected %(expected).2f",
                    received=amount_converted, expected=tx.amount
                )
            )
        return tx

    def _process_feedback_data(self, data):
        """ Override of payment to process the transaction based on Sips data.

        Note: self.ensure_one()

        :param dict data: The feedback data sent by the provider
        :return: None
        """
        super()._process_feedback_data(data)
        if self.provider != 'sips':
            return

        data = self._sips_data_to_object(data.get('Data'))
        self.acquirer_reference = data.get('transactionReference')
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
            "ref: %s, got response [%s], set as '%s'.", self.reference, response_code, status
        )

    def _sips_data_to_object(self, data):
        res = {}
        for element in data.split('|'):
            key, value = element.split('=', 1)
            res[key] = value
        return res

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
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#94B6C8"/><stop offset="100%" stop-color="#6A9EBA"/></linearGradient><path id="d" d="M19.25 43h2.714v4.212c0 .188.152.343.339.343h26.394a.342.342 0 0 0 .339-.343V37.5H32.732c1.623-2.333 2.212-4.5 1.768-6.5h14.536v-3.212a.342.342 0 0 0-.339-.343h-14.79a15.663 15.663 0 0 0-1.215-2.742h16.344c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742V43zm-3.79 12.642h-1.037V55h2.833v.642h-1.037v2.832h-.76v-2.832zM17.422 55h1.07l.809 2.389h.01L20.075 55h1.07v3.474h-.711v-2.462h-.01l-.847 2.462h-.586l-.848-2.438h-.01v2.438h-.711V55zm-1.394-16.606C15 37.5 14.5 37.025 13.733 35.448c-.556-1.144-.908-2.315-1.026-3.6-.035-.377-.04-1.232-.01-1.572.127-1.427.479-2.634 1.124-3.861a9.868 9.868 0 0 1 4.994-4.507c.24-.096.63-.236.637-.229a.859.859 0 0 1-.071.132c-.633 1.046-1.176 3.203-1.275 5.059l-.018.353-.115.162a6.324 6.324 0 0 0-1.094 2.865 9.018 9.018 0 0 0 0 1.521 6.257 6.257 0 0 0 1.208 3.02c.836 1.112 2.034 1.88 3.332 2.14.426.086.53.095 1.082.094.445 0 .547-.005.751-.038a5.349 5.349 0 0 0 2.003-.723c.116-.07.23-.14.252-.158l.041-.031-.092-.039a15.055 15.055 0 0 1-1.08-.556 9.864 9.864 0 0 1-3.215-3.084 9.796 9.796 0 0 1-.613-9.633c.095-.196.236-.466.313-.6.152-.265.535-.852.586-.896.078-.07 1.26-.095 1.865-.041a9.844 9.844 0 0 1 8.65 7.201c.254.942.352 1.744.334 2.764a8.847 8.847 0 0 1-.301 2.261c-.715 2.785-2.654 5.138-5.278 6.405a9.795 9.795 0 0 1-3.547.946 16.36 16.36 0 0 1-1.405-.002 9.878 9.878 0 0 1-5.736-2.407zM27.92 32.69c.022-.079.065-.272.097-.43a6.412 6.412 0 0 0-.299-3.548c-.64-1.645-1.91-2.891-3.508-3.446a1.81 1.81 0 0 0-.196-.06c-.044 0-.192.682-.25 1.148-.029.237-.029 1.131 0 1.37.155 1.251.585 2.3 1.327 3.232.16.201.553.607.747.772.553.47 1.12.796 1.808 1.04.111.04.21.071.219.069a.487.487 0 0 0 .055-.147z"/><path id="e" d="M19.25 42h2.714v3.212c0 .188.152.343.339.343h26.394a.342.342 0 0 0 .339-.343V35.5H32.732c1.623-2.333 2.212-4.5 1.768-6.5h14.536v-3.212a.342.342 0 0 0-.339-.343h-14.79a15.663 15.663 0 0 0-1.215-2.742h16.344c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742V42zm-3.79 11.642h-1.037V53h2.833v.642h-1.037v2.832h-.76v-2.832zM17.422 53h1.07l.809 2.389h.01L20.075 53h1.07v3.474h-.711v-2.462h-.01l-.847 2.462h-.586l-.848-2.438h-.01v2.438h-.711V53zm-1.394-16.606C15 35.5 14.5 35.025 13.733 33.448c-.556-1.144-.908-2.315-1.026-3.6-.035-.377-.04-1.232-.01-1.572.127-1.427.479-2.634 1.124-3.861a9.868 9.868 0 0 1 4.994-4.507c.24-.096.63-.236.637-.229a.859.859 0 0 1-.071.132c-.633 1.046-1.176 3.203-1.275 5.059l-.018.353-.115.162a6.324 6.324 0 0 0-1.094 2.865 9.018 9.018 0 0 0 0 1.521 6.257 6.257 0 0 0 1.208 3.02c.836 1.112 2.034 1.88 3.332 2.14.426.086.53.095 1.082.094.445 0 .547-.005.751-.038a5.349 5.349 0 0 0 2.003-.723c.116-.07.23-.14.252-.158l.041-.031-.092-.039a15.055 15.055 0 0 1-1.08-.556 9.864 9.864 0 0 1-3.215-3.084 9.796 9.796 0 0 1-.613-9.633c.095-.196.236-.466.313-.6.152-.265.535-.852.586-.896.078-.07 1.26-.095 1.865-.041a9.844 9.844 0 0 1 8.65 7.201c.254.942.352 1.744.334 2.764a8.847 8.847 0 0 1-.301 2.261c-.715 2.785-2.654 5.138-5.278 6.405a9.795 9.795 0 0 1-3.547.946 16.36 16.36 0 0 1-1.405-.002 9.878 9.878 0 0 1-5.736-2.407zM27.92 30.69c.022-.079.065-.272.097-.43a6.412 6.412 0 0 0-.299-3.548c-.64-1.645-1.91-2.891-3.508-3.446a1.81 1.81 0 0 0-.196-.06c-.044 0-.192.682-.25 1.148-.029.237-.029 1.131 0 1.37.155 1.251.585 2.3 1.327 3.232.16.201.553.607.747.772.553.47 1.12.796 1.808 1.04.111.04.21.071.219.069a.487.487 0 0 0 .055-.147z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M4 69c-2 0-4-1-4-4V33.916L16.034 21.5 18 21.197l3.6-1.697 7.4 3.203h8.25L48 23l3.576 1.968-.236 22.01L39.224 69H4z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
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

## File: views\payment_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_acquirer_form" model="ir.ui.view">
        <field name="name">Sips Acquirer Form</field>
        <field name="model">payment.acquirer</field>
        <field name="inherit_id" ref="payment.payment_acquirer_form"/>
        <field name="arch" type="xml">
            <xpath expr='//group[@name="acquirer"]' position='inside'>
                <group attrs="{'invisible': [('provider', '!=', 'sips')]}">
                    <field name="sips_merchant_id" attrs="{'required':[ ('provider', '=', 'sips'), ('state', '!=', 'disabled')]}"/>
                    <field name="sips_secret" string="Secret Key" attrs="{'required':[ ('provider', '=', 'sips'), ('state', '!=', 'disabled')]}"/>
                    <field name="sips_key_version" attrs="{'required':[ ('provider', '=', 'sips'), ('state', '!=', 'disabled')]}" />
                    <field name="sips_test_url" attrs="{'required':[ ('provider', '=', 'sips'), ('state', '!=', 'disabled')]}" groups='base.group_no_one'/>
                    <field name="sips_prod_url" attrs="{'required':[ ('provider', '=', 'sips'), ('state', '!=', 'disabled')]}" groups='base.group_no_one'/>
                    <field name="sips_version" attrs="{'required':[ ('provider', '=', 'sips'), ('state', '!=', 'disabled')]}" />
                </group>
            </xpath>
        </field>
    </record>

</odoo>

```

