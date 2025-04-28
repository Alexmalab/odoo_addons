# Odoo Module: payment_ogone

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
            "Consider installing the Payment Provider: Stripe module instead.")


def post_init_hook(cr, registry):
    setup_provider(cr, registry, 'ogone')


def uninstall_hook(cr, registry):
    reset_payment_provider(cr, registry, 'ogone')

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Payment Provider: Ogone',
    'version': '2.0',
    'category': 'Accounting/Payment Providers',
    'sequence': 350,
    'summary': "This module is deprecated.",
    'description': " ",  # Non-empty string to avoid loading the README file.
    'depends': ['payment'],
    'data': [
        'views/payment_ogone_templates.xml',
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
import re

from werkzeug.exceptions import Forbidden

from odoo import http
from odoo.http import request

_logger = logging.getLogger(__name__)


class OgoneController(http.Controller):
    _return_url = '/payment/ogone/return'
    _backward_compatibility_urls = [
        '/payment/ogone/accept', '/payment/ogone/test/accept',
        '/payment/ogone/decline', '/payment/ogone/test/decline',
        '/payment/ogone/exception', '/payment/ogone/test/exception',
        '/payment/ogone/cancel', '/payment/ogone/test/cancel',
        '/payment/ogone/validate/accept',
        '/payment/ogone/validate/decline',
        '/payment/ogone/validate/exception',
    ]  # Facilitates the migration of users who registered the URLs in Ogone's backend prior to 14.3

    @http.route(
        [_return_url] + _backward_compatibility_urls, type='http', auth='public',
        methods=['GET', 'POST'], csrf=False
    )  # Redirect are made with GET requests only. Webhook notifications can be set to GET or POST.
    def ogone_return_from_checkout(self, **raw_data):
        """ Process the notification data sent by Ogone after redirection from checkout.

        This route can also accept S2S notifications from Ogone if it is configured as a webhook in
        Ogone's backend. The user can choose between GET or POST for the webhook notifications.

        :param dict raw_data: The un-formatted notification data
        """
        _logger.info("handling redirection from Ogone with data:\n%s", pprint.pformat(raw_data))
        data = self._normalize_data_keys(raw_data)

        # Check the integrity of the notification
        received_signature = data.get('SHASIGN')
        tx_sudo = request.env['payment.transaction'].sudo()._get_tx_from_notification_data(
            'ogone', data
        )
        self._verify_notification_signature(raw_data, received_signature, tx_sudo)

        # Handle the notification data
        tx_sudo._handle_notification_data('ogone', data)
        return request.redirect('/payment/status')

    @staticmethod
    def _normalize_data_keys(data):
        """ Set all keys of a dictionary to upper-case.

        The keys received from Ogone APIs have inconsistent formatting and must be homogenized to
        allow re-using the same methods. We reformat them to follow a unified nomenclature inspired
        by Ogone Directlink API.

        Formatting steps:
        1) Uppercase key strings: 'Something' -> 'SOMETHING', 'something' -> 'SOMETHING'
        2) Remove the prefix: 'CARD.SOMETHING' -> 'SOMETHING', 'ALIAS.SOMETHING' -> 'SOMETHING'

        :param dict data: The data whose keys to normalize
        :return: The normalized data
        :rtype: dict
        """
        return {re.sub(r'.*\.', '', k.upper()): v for k, v in data.items()}

    @staticmethod
    def _verify_notification_signature(notification_data, received_signature, tx_sudo):
        """ Check that the received signature matches the expected one.

        :param dict notification_data: The notification data
        :param str received_signature: The signature received with the notification data
        :param recordset tx_sudo: The sudoed transaction referenced by the notification data, as a
                                  `payment.transaction` record
        :return: None
        :raise: :class:`werkzeug.exceptions.Forbidden` if the signatures don't match
        """
        # Check for the received signature
        if not received_signature:
            _logger.warning("received notification with missing signature")
            raise Forbidden()

        # Compare the received signature with the expected signature computed from the data
        expected_signature = tx_sudo.provider_id._ogone_generate_signature(notification_data)
        if not hmac.compare_digest(received_signature, expected_signature.upper()):
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
-- disable ogone payment provider
UPDATE payment_provider
   SET ogone_pspid = NULL,
       ogone_userid = NULL,
       ogone_password = NULL,
       ogone_shakey_in = NULL,
       ogone_shakey_out = NULL;

```

## File: data\payment_provider_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="payment_provider_ogone" model="payment.provider">
        <field name="name">Ogone</field>
        <field name="display_as">Credit Card (powered by Ogone)</field>
        <field name="image_128"
               type="base64"
               file="payment_ogone/static/description/icon.png"/>
        <field name="module_id" ref="base.module_payment_ogone"/>
        <field name="payment_icon_ids"
               eval="[(6, 0, [
                   ref('payment.payment_icon_cc_ideal'),
                   ref('payment.payment_icon_cc_bancontact'),
                   ref('payment.payment_icon_cc_maestro'),
                   ref('payment.payment_icon_cc_mastercard'),
                   ref('payment.payment_icon_cc_visa'),
               ])]"/>
        <field name="code">ogone</field>
        <field name="redirect_form_view_id" ref="redirect_form"/>
        <field name="allow_tokenization">True</field>
    </record>

</odoo>

```

## File: models\const.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# See https://epayments-support.ingenico.com/en/integration-solutions/integrations/directlink#directlink_integration_guides_request_a_new_order
# See https://epayments-support.ingenico.com/en/integration-solutions/integrations/directlink#directlink_integration_guides_order_response
VALID_KEYS = [
    'AAVADDRESS',
    'AAVCHECK',
    'AAVMAIL',
    'AAVNAME',
    'AAVPHONE',
    'AAVZIP',
    'ACCEPTANCE',
    'ALIAS',
    'AMOUNT',
    'BIC',
    'BIN',
    'BRAND',
    'CARDNO',
    'CCCTY',
    'CN',
    'COLLECTOR_BIC',
    'COLLECTOR_IBAN',
    'COMPLUS',
    'CREATION_STATUS',
    'CREDITDEBIT',
    'CURRENCY',
    'CVCCHECK',
    'DCC_COMMPERCENTAGE',
    'DCC_CONVAMOUNT',
    'DCC_CONVCCY',
    'DCC_EXCHRATE',
    'DCC_EXCHRATESOURCE',
    'DCC_EXCHRATETS',
    'DCC_INDICATOR',
    'DCC_MARGINPERCENTAGE',
    'DCC_VALIDHOURS',
    'DEVICEID',
    'DIGESTCARDNO',
    'ECI',
    'ED',
    'EMAIL',
    'ENCCARDNO',
    'FXAMOUNT',
    'FXCURRENCY',
    'IP',
    'IPCTY',
    'MANDATEID',
    'MOBILEMODE',
    'NBREMAILUSAGE',
    'NBRIPUSAGE',
    'NBRIPUSAGE_ALLTX',
    'NBRUSAGE',
    'NCERROR',
    'ORDERID',
    'PAYID',
    'PAYIDSUB',
    'PAYMENT_REFERENCE',
    'PM',
    'SCO_CATEGORY',
    'SCORING',
    'SEQUENCETYPE',
    'SIGNDATE',
    'STATUS',
    'SUBBRAND',
    'SUBSCRIPTION_ID',
    'TICKET',
    'TRXDATE',
    'VC',
]


# See https://epayments-support.ingenico.com/en/get-started/transaction-status-full/
PAYMENT_STATUS_MAPPING = {
    'pending': (41, 46, 50, 51, 52, 55, 56, 81, 82, 91, 92, 99),  # 46 = 3DS
    'done': (5, 8, 9),
    'cancel': (1,),
    'declined': (2,),
}

```

## File: models\payment_provider.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
from hashlib import new as hashnew

import requests

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError

from .const import VALID_KEYS

_logger = logging.getLogger(__name__)


class PaymentProvider(models.Model):
    _inherit = 'payment.provider'

    code = fields.Selection(
        selection_add=[('ogone', "Ogone")], ondelete={'ogone': 'set default'})
    ogone_pspid = fields.Char(
        string="PSPID", help="The ID solely used to identify the account with Ogone",
        required_if_provider='ogone')
    ogone_userid = fields.Char(
        string="API User ID", help="The ID solely used to identify the API user with Ogone",
        required_if_provider='ogone')
    ogone_password = fields.Char(
        string="API User Password", required_if_provider='ogone', groups='base.group_system')
    ogone_shakey_in = fields.Char(
        string="SHA Key IN", required_if_provider='ogone', groups='base.group_system')
    ogone_shakey_out = fields.Char(
        string="SHA Key OUT", required_if_provider='ogone', groups='base.group_system')
    ogone_hash_function = fields.Selection(
        [('sha1', 'SHA1'), ('sha256', 'SHA256'), ('sha512', 'SHA512')], default='sha512',
        string="Hash function", required_if_provider='ogone',
    )

    #=== COMPUTE METHODS ===#

    def _compute_feature_support_fields(self):
        """ Override of `payment` to enable additional features. """
        super()._compute_feature_support_fields()
        self.filtered(lambda p: p.code == 'ogone').update({
            'support_tokenization': True,
        })

    #=== BUSINESS METHODS ===#

    @api.model
    def _get_compatible_providers(self, *args, is_validation=False, **kwargs):
        """ Override of payment to unlist Ogone providers for validation operations. """
        providers = super()._get_compatible_providers(*args, is_validation=is_validation, **kwargs)

        if is_validation:
            providers = providers.filtered(lambda p: p.code != 'ogone')

        return providers

    def _ogone_get_api_url(self, api_key):
        """ Return the appropriate URL of the requested API for the provider state.

        Note: self.ensure_one()

        :param str api_key: The API whose URL to get: 'hosted_payment_page' or 'directlink'
        :return: The API URL
        :rtype: str
        """
        self.ensure_one()

        if self.state == 'enabled':
            api_urls = {
                'hosted_payment_page': 'https://secure.ogone.com/ncol/prod/orderstandard_utf8.asp',
                'directlink': 'https://secure.ogone.com/ncol/prod/orderdirect_utf8.asp',
            }
        else:  # 'test'
            api_urls = {
                'hosted_payment_page': 'https://ogone.test.v-psp.com/ncol/test/orderstandard_utf8.asp',
                'directlink': 'https://ogone.test.v-psp.com/ncol/test/orderdirect_utf8.asp',
            }
        return api_urls.get(api_key)

    def _ogone_generate_signature(self, values, incoming=True, format_keys=False):
        """ Generate the signature for incoming or outgoing communications.

        :param dict values: The values used to generate the signature
        :param bool incoming: Whether the signature must be generated for an incoming (Ogone to
                              Odoo) or outgoing (Odoo to Ogone) communication.
        :param bool format_keys: Whether the keys must be formatted as uppercase, dot-separated
                                 strings to comply with Ogone APIs. This must be used when the keys
                                 are formatted as underscore-separated strings to be compliant with
                                 QWeb's `t-att-value`.
        :return: The signature
        :rtype: str
        """

        def _filter_key(_key):
            return not incoming or _key in VALID_KEYS

        key = self.ogone_shakey_out if incoming else self.ogone_shakey_in  # Swapped for Ogone's POV
        if format_keys:
            formatted_items = [(k.upper().replace('_', '.'), v) for k, v in values.items()]
        else:
            formatted_items = [(k.upper(), v) for k, v in values.items()]
        sorted_items = sorted(formatted_items)
        signing_string = ''.join(f'{k}={v}{key}' for k, v in sorted_items if _filter_key(k) and v)
        shasign = hashnew(self.ogone_hash_function)
        shasign.update(signing_string.encode())
        return shasign.hexdigest()

    def _ogone_make_request(self, payload=None, method='POST'):
        """ Make a request to one of Ogone APIs.

        Note: self.ensure_one()

        :param dict payload: The payload of the request
        :param str method: The HTTP method of the request
        :return The content of the response
        :rtype: bytes
        :raise: ValidationError if an HTTP error occurs
        """
        self.ensure_one()

        url = self._ogone_get_api_url('directlink')
        try:
            response = requests.request(method, url, data=payload, timeout=60)
            response.raise_for_status()
        except requests.exceptions.ConnectionError:
            _logger.exception("unable to reach endpoint at %s", url)
            raise ValidationError("Ogone: " + _("Could not establish the connection to the API."))
        except requests.exceptions.HTTPError:
            _logger.exception("invalid API request at %s with data %s", url, payload)
            raise ValidationError("Ogone: " + _("The communication with the API failed."))
        return response.content

```

## File: models\payment_transaction.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import pprint
import uuid

from lxml import etree, objectify
from werkzeug import urls

from odoo import _, api, models
from odoo.exceptions import UserError, ValidationError

from . import const
from odoo.addons.payment import utils as payment_utils
from odoo.addons.payment_ogone.controllers.main import OgoneController

_logger = logging.getLogger(__name__)


class PaymentTransaction(models.Model):
    _inherit = 'payment.transaction'

    @api.model
    def _compute_reference(self, provider_code, prefix=None, separator='-', **kwargs):
        """ Override of payment to ensure that Ogone requirements for references are satisfied.

        Ogone requirements for references are as follows:
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
        if provider_code != 'ogone':
            return super()._compute_reference(provider_code, prefix=prefix, **kwargs)

        if not prefix:
            # If no prefix is provided, it could mean that a module has passed a kwarg intended for
            # the `_compute_reference_prefix` method, as it is only called if the prefix is empty.
            # We call it manually here because singularizing the prefix would generate a default
            # value if it was empty, hence preventing the method from ever being called and the
            # transaction from received a reference named after the related document.
            prefix = self.sudo()._compute_reference_prefix(provider_code, separator, **kwargs) or None
        prefix = payment_utils.singularize_reference_prefix(prefix=prefix, max_length=40)
        return super()._compute_reference(provider_code, prefix=prefix, **kwargs)

    def _get_specific_rendering_values(self, processing_values):
        """ Override of payment to return Ogone-specific rendering values.

        Note: self.ensure_one() from `_get_processing_values`

        :param dict processing_values: The generic and specific processing values of the transaction
        :return: The dict of provider-specific processing values
        :rtype: dict
        """
        res = super()._get_specific_rendering_values(processing_values)
        if self.provider_code != 'ogone':
            return res

        return_url = urls.url_join(self.provider_id.get_base_url(), OgoneController._return_url)
        rendering_values = {
            'PSPID': self.provider_id.ogone_pspid,
            'ORDERID': self.reference,
            'AMOUNT': payment_utils.to_minor_currency_units(self.amount, None, 2),
            'CURRENCY': self.currency_id.name,
            'LANGUAGE': self.partner_lang or 'en_US',
            'EMAIL': self.partner_email or '',
            'OWNERADDRESS': self.partner_address or '',
            'OWNERZIP': self.partner_zip or '',
            'OWNERTOWN': self.partner_city or '',
            'OWNERCTY': self.partner_country_id.code or '',
            'OWNERTELNO': self.partner_phone or '',
            'OPERATION': 'SAL',  # direct sale
            'USERID': self.provider_id.ogone_userid,
            'ACCEPTURL': return_url,
            'DECLINEURL': return_url,
            'EXCEPTIONURL': return_url,
            'CANCELURL': return_url,
        }
        if self.tokenize:
            rendering_values.update({
                'ALIAS': f'ODOO-ALIAS-{uuid.uuid4().hex}',
                'ALIASUSAGE': _("Storing your payment details is necessary for future use."),
            })
        rendering_values.update({
            'SHASIGN': self.provider_id._ogone_generate_signature(
                rendering_values, incoming=False
            ).upper(),
            'api_url': self.provider_id._ogone_get_api_url('hosted_payment_page'),
        })
        return rendering_values

    def _send_payment_request(self):
        """ Override of payment to send a payment request to Ogone.

        Note: self.ensure_one()

        :return: None
        :raise: UserError if the transaction is not linked to a token
        """
        super()._send_payment_request()
        if self.provider_code != 'ogone':
            return

        if not self.token_id:
            raise UserError("Ogone: " + _("The transaction is not linked to a token."))

        # Make the payment request
        data = {
            # DirectLink parameters
            'PSPID': self.provider_id.ogone_pspid,
            'ORDERID': self.reference,
            'USERID': self.provider_id.ogone_userid,
            'PSWD': self.provider_id.ogone_password,
            'AMOUNT': payment_utils.to_minor_currency_units(self.amount, None, 2),
            'CURRENCY': self.currency_id.name,
            'CN': self.partner_name or '',  # Cardholder Name
            'EMAIL': self.partner_email or '',
            'OWNERADDRESS': self.partner_address or '',
            'OWNERZIP': self.partner_zip or '',
            'OWNERTOWN': self.partner_city or '',
            'OWNERCTY': self.partner_country_id.code or '',
            'OWNERTELNO': self.partner_phone or '',
            'OPERATION': 'SAL',  # direct sale
            # Alias Manager parameters
            'ALIAS': self.token_id.provider_ref,
            'ALIASPERSISTEDAFTERUSE': 'Y',
            'ECI': 9,  # Recurring (from eCommerce)
        }
        data['SHASIGN'] = self.provider_id._ogone_generate_signature(data, incoming=False)

        _logger.info(
            "payment request response for transaction with reference %s:\n%s",
            self.reference, pprint.pformat({k: v for k, v in data.items() if k != 'PSWD'})
        )  # Log the payment request data without the password
        response_content = self.provider_id._ogone_make_request(data)
        try:
            tree = objectify.fromstring(response_content)
        except etree.XMLSyntaxError:
            raise ValidationError("Ogone: " + "Received badly structured response from the API.")

        # Handle the feedback data
        _logger.info(
            "payment request response (as an etree) for transaction with reference %s:\n%s",
            self.reference, etree.tostring(tree, pretty_print=True, encoding='utf-8')
        )
        feedback_data = {'ORDERID': tree.get('orderID'), 'tree': tree}
        _logger.info(
            "handling feedback data from Ogone for transaction with reference %s with data:\n%s",
            self.reference, pprint.pformat(feedback_data)
        )
        self._handle_notification_data('ogone', feedback_data)

    def _get_tx_from_notification_data(self, provider_code, notification_data):
        """ Override of payment to find the transaction based on Ogone data.

        :param str provider_code: The code of the provider that handled the transaction
        :param dict notification_data: The notification data sent by the provider
        :return: The transaction if found
        :rtype: recordset of `payment.transaction`
        :raise: ValidationError if the data match no transaction
        """
        tx = super()._get_tx_from_notification_data(provider_code, notification_data)
        if provider_code != 'ogone' or len(tx) == 1:
            return tx

        reference = notification_data.get('ORDERID')
        tx = self.search([('reference', '=', reference), ('provider_code', '=', 'ogone')])
        if not tx:
            raise ValidationError(
                "Ogone: " + _("No transaction found matching reference %s.", reference)
            )
        return tx

    def _process_notification_data(self, notification_data):
        """ Override of payment to process the transaction based on Ogone data.

        Note: self.ensure_one()

        :param dict notification_data: The notification data sent by the provider
        :return: None
        """
        super()._process_notification_data(notification_data)
        if self.provider_code != 'ogone':
            return

        if 'tree' in notification_data:
            notification_data = notification_data['tree']

        self.provider_reference = notification_data.get('PAYID')
        payment_status = int(notification_data.get('STATUS', '0'))
        if payment_status in const.PAYMENT_STATUS_MAPPING['pending']:
            self._set_pending()
        elif payment_status in const.PAYMENT_STATUS_MAPPING['done']:
            has_token_data = 'ALIAS' in notification_data
            if self.tokenize and has_token_data:
                self._ogone_tokenize_from_notification_data(notification_data)
            self._set_done()
        elif payment_status in const.PAYMENT_STATUS_MAPPING['cancel']:
            self._set_canceled()
        elif payment_status in const.PAYMENT_STATUS_MAPPING['declined']:
            if notification_data.get("NCERRORPLUS"):
                reason = notification_data.get("NCERRORPLUS")
            elif notification_data.get("NCERROR"):
                reason = "Error code: %s" % notification_data.get("NCERROR")
            else:
                reason = "Unknown reason"
            _logger.info("the payment has been declined: %s.", reason)
            self._set_error(
                "Ogone: " + _("The payment has been declined: %s", reason)
            )
        else:  # Classify unknown payment statuses as `error` tx state
            _logger.info(
                "received data with invalid payment status (%s) for transaction with reference %s",
                payment_status, self.reference
            )
            self._set_error(
                "Ogone: " + _("Received data with invalid payment status: %s", payment_status)
            )

    def _ogone_tokenize_from_notification_data(self, notification_data):
        """ Create a token from notification data.

        :param dict notification_data: The notification data sent by the provider
        :return: None
        """
        token = self.env['payment.token'].create({
            'provider_id': self.provider_id.id,
            'payment_details': notification_data.get('CARDNO')[-4:],  # Ogone pads details with X's.
            'partner_id': self.partner_id.id,
            'provider_ref': notification_data['ALIAS'],
            'verified': True,  # The payment is authorized, so the payment method is valid
        })
        self.write({
            'token_id': token.id,
            'tokenize': False,
        })
        _logger.info(
            "created token with id %(token_id)s for partner with id %(partner_id)s from "
            "transaction with reference %(ref)s",
            {
                'token_id': token.id,
                'partner_id': self.partner_id.id,
                'ref': self.reference,
            },
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
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#94B6C8"/><stop offset="100%" stop-color="#6A9EBA"/></linearGradient><path id="d" d="M19.25 42h2.714v5.212c0 .188.152.343.339.343h26.394a.342.342 0 0 0 .339-.343V37.5H30.732c.389-3.667-.582-5.833-2.914-6.5h21.218v-3.212a.342.342 0 0 0-.339-.343H29.765c.361-1.01.57-1.924.627-2.742h18.644c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742V42zm-3.79 13.642h-1.037V55h2.833v.642h-1.037v2.832h-.76v-2.832zM17.422 55h1.07l.809 2.389h.01L20.075 55h1.07v3.474h-.711v-2.462h-.01l-.847 2.462h-.586l-.848-2.438h-.01v2.438h-.711V55zm17.403-34.99c.985.007-2.543 2.376-2.271 2.073h-5.692c.113.126.201.23.298.334 1.214 1.315 1.076 3.081.239 4.432-.597.965-1.488 1.586-2.49 2.049-1.103.518-2.277.747-3.475.909a85.771 85.771 0 0 0-3.555.56c-.274.046-.546.186-.782.34-.383.25-.375.645.047.811.595.23 1.216.411 1.843.532 1.768.343 3.567.532 5.295 1.073.842.265 1.651.598 2.341 1.182 1.138.972 1.222 2.65.013 3.726-.948.85-2.094 1.272-3.295 1.557-1.024.243-2.063.36-3.117.396-2.206.076-4.376-.11-6.481-.823-.77-.265-1.497-.614-2.09-1.2-.333-.322-.582-.69-.64-1.174-.08-.656.198-1.127.696-1.496.701-.517 1.51-.775 2.335-.97.797-.192 1.606-.33 2.345-.482-.471-.161-1.025-.314-1.546-.545-.407-.185-.812-.406-1.146-.7-.655-.571-.655-1.42-.049-2.06.442-.468.997-.76 1.586-.974.513-.192 1.041-.34 1.567-.51-.328-.148-.676-.29-1.004-.459-.748-.39-1.42-.886-1.821-1.658-.777-1.496-.475-2.903.467-4.197.908-1.243 2.204-1.916 3.634-2.345a9.62 9.62 0 0 1 2.742-.387c4.671 0 9.339-.013 14.006.007zM18.778 34.016c-1.03.185-2.057.453-3.064.787-.495.16-.972.475-1.405.82-.421.333-.405.865.006 1.224.269.235.58.424.893.553 1.223.505 2.505.489 4.007.601.526-.08 1.283-.165 2.03-.322.546-.114 1.073-.34 1.483-.845.381-.462.363-.962-.063-1.357a2.674 2.674 0 0 0-.689-.487 25.158 25.158 0 0 0-1.952-.725c-.408-.133-.846-.318-1.246-.249zm4.666-7.55c.368-.527.59-1.137.552-1.813-.061-1.064-.609-1.772-1.446-2.208a3.71 3.71 0 0 0-2.357-.396c-1.155.18-2.127.73-2.802 1.824-.64 1.036-.479 2.313.366 3.145.737.729 1.629.933 2.647.983.182-.02.438-.041.69-.08.94-.162 1.748-.606 2.35-1.455z"/><path id="e" d="M19.25 40h2.714v5.212c0 .188.152.343.339.343h26.394a.342.342 0 0 0 .339-.343V35.5H30.732c.389-3.667-.582-5.833-2.914-6.5h21.218v-3.212a.342.342 0 0 0-.339-.343H29.765c.361-1.01.57-1.924.627-2.742h18.644c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742V40zm-3.79 13.642h-1.037V53h2.833v.642h-1.037v2.832h-.76v-2.832zM17.422 53h1.07l.809 2.389h.01L20.075 53h1.07v3.474h-.711v-2.462h-.01l-.847 2.462h-.586l-.848-2.438h-.01v2.438h-.711V53zm17.403-34.99c.985.007-2.543 2.376-2.271 2.073h-5.692c.113.126.201.23.298.334 1.214 1.315 1.076 3.081.239 4.432-.597.965-1.488 1.586-2.49 2.049-1.103.518-2.277.747-3.475.909a85.771 85.771 0 0 0-3.555.56c-.274.046-.546.186-.782.34-.383.25-.375.645.047.811.595.23 1.216.411 1.843.532 1.768.343 3.567.532 5.295 1.073.842.265 1.651.598 2.341 1.182 1.138.972 1.222 2.65.013 3.726-.948.85-2.094 1.272-3.295 1.557-1.024.243-2.063.36-3.117.396-2.206.076-4.376-.11-6.481-.823-.77-.265-1.497-.614-2.09-1.2-.333-.322-.582-.69-.64-1.174-.08-.656.198-1.127.696-1.496.701-.517 1.51-.775 2.335-.97.797-.192 1.606-.33 2.345-.482-.471-.161-1.025-.314-1.546-.545-.407-.185-.812-.406-1.146-.7-.655-.571-.655-1.42-.049-2.06.442-.468.997-.76 1.586-.974.513-.192 1.041-.34 1.567-.51-.328-.148-.676-.29-1.004-.459-.748-.39-1.42-.886-1.821-1.658-.777-1.496-.475-2.903.467-4.197.908-1.243 2.204-1.916 3.634-2.345a9.62 9.62 0 0 1 2.742-.387c4.671 0 9.339-.013 14.006.007zM18.778 32.016c-1.03.185-2.057.453-3.064.787-.495.16-.972.475-1.405.82-.421.333-.405.865.006 1.224.269.235.58.424.893.553 1.223.505 2.505.489 4.007.601.526-.08 1.283-.165 2.03-.322.546-.114 1.073-.34 1.483-.845.381-.462.363-.962-.063-1.357a2.674 2.674 0 0 0-.689-.487 25.158 25.158 0 0 0-1.952-.725c-.408-.133-.846-.318-1.246-.249zm4.666-7.55c.368-.527.59-1.137.552-1.813-.061-1.064-.609-1.772-1.446-2.208a3.71 3.71 0 0 0-2.357-.396c-1.155.18-2.127.73-2.802 1.824-.64 1.036-.479 2.313.366 3.145.737.729 1.629.933 2.647.983.182-.02.438-.041.69-.08.94-.162 1.748-.606 2.35-1.455z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M4 69c-2 0-4-1-4-4V33.916L15.591 19.5 34.912 18l-3.89 4.703h6.228L48 23l3.576 1.968-.236 22.01L39.224 69H4z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: views\payment_ogone_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="redirect_form">
        <form t-att-action="api_url" method="post">
            <input type="hidden" name="PSPID" t-att-value="PSPID"/>
            <input type="hidden" name="ORDERID" t-att-value="ORDERID"/>
            <input type="hidden" name="AMOUNT" t-att-value="AMOUNT"/>
            <input type="hidden" name="CURRENCY" t-att-value="CURRENCY"/>
            <input type="hidden" name="LANGUAGE" t-att-value="LANGUAGE"/>
            <input type="hidden" name="EMAIL" t-att-value="EMAIL"/>
            <input type="hidden" name="OWNERADDRESS" t-att-value="OWNERADDRESS"/>
            <input type="hidden" name="OWNERZIP" t-att-value="OWNERZIP"/>
            <input type="hidden" name="OWNERTOWN" t-att-value="OWNERTOWN"/>
            <input type="hidden" name="OWNERCTY" t-att-value="OWNERCTY"/>
            <input type="hidden" name="OWNERTELNO" t-att-value="OWNERTELNO"/>
            <input type="hidden" name="OPERATION" t-att-value="OPERATION"/>
            <input type="hidden" name="USERID" t-att-value="USERID"/>
            <input type="hidden" name="ACCEPTURL" t-att-value="ACCEPTURL"/>
            <input type="hidden" name="DECLINEURL" t-att-value="DECLINEURL"/>
            <input type="hidden" name="EXCEPTIONURL" t-att-value="EXCEPTIONURL"/>
            <input type="hidden" name="CANCELURL" t-att-value="CANCELURL"/>
            <input type="hidden" name="ALIAS" t-att-value="ALIAS"/>
            <input type="hidden" name="ALIASUSAGE" t-att-value="ALIASUSAGE"/>
            <input type="hidden" name="SHASIGN" t-att-value="SHASIGN"/>
        </form>
    </template>

</odoo>

```

## File: views\payment_provider_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_provider_form" model="ir.ui.view">
        <field name="name">Ogone Provider Form</field>
        <field name="model">payment.provider</field>
        <field name="inherit_id" ref="payment.payment_provider_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='provider_creation_warning']" position="after">
                <div class="alert alert-danger"
                     role="alert"
                     attrs="{'invisible': [('code', '!=', 'ogone')]}">
                    This provider is deprecated.
                    Consider disabling it and moving to <strong>Stripe</strong>.
                </div>
            </xpath>
            <group name="provider_credentials" position="inside">
                <group attrs="{'invisible': [('code', '!=', 'ogone')]}">
                    <field name="ogone_pspid" attrs="{'required':[('code', '=', 'ogone'), ('state', '!=', 'disabled')]}"/>
                    <field name="ogone_userid" attrs="{'required':[('code', '=', 'ogone'), ('state', '!=', 'disabled')]}"/>
                    <field name="ogone_password" attrs="{'required':[('code', '=', 'ogone'), ('state', '!=', 'disabled')]}" password="True"/>
                    <field name="ogone_shakey_in" attrs="{'required':[('code', '=', 'ogone'), ('state', '!=', 'disabled')]}" password="True"/>
                    <field name="ogone_shakey_out" attrs="{'required':[('code', '=', 'ogone'), ('state', '!=', 'disabled')]}" password="True"/>
                    <field name="ogone_hash_function" attrs="{'required':[('code', '=', 'ogone'), ('state', '!=', 'disabled')]}" groups="base.group_no_one"/>
                </group>
            </group>
        </field>
    </record>

</odoo>

```

