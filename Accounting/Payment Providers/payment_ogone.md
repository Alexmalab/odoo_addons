# Odoo Module: payment_ogone

Category: Accounting/Payment Providers

This file contains the source code of the Odoo module.

## File: const.py

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

# The codes of the payment methods to activate when Ogone is activated.
DEFAULT_PAYMENT_METHODS_CODES = [
    # Primary payment methods.
    'card',
    # Brand payment methods.
    'visa',
    'mastercard',
    'amex',
    'discover',
]

# Mapping of payment method codes to Ogone codes.
PAYMENT_METHODS_MAPPING = {
    'card': 'CreditCard',
    'paylib': 'Paylib',
    'p24': 'Przelewy24',
    'bancontact': 'BCMC',
    'paypal': 'PAYPAL',
    'ideal': 'IDEAL',
    'eps': 'EPS',
    'visa': 'VISA',
    'mastercard': 'MasterCard',
    'jcb': 'JCB',
    'klarna_paynow': 'KLARNA_PAYNOW',
    'klarna_pay_over_time': 'KLARNA_PAYLATER',
    'sofort': 'DirectEbanking',
}

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
            "Consider installing the Payment Provider: Stripe module instead.")


def post_init_hook(env):
    setup_provider(env, 'ogone')


def uninstall_hook(env):
    reset_payment_provider(env, 'ogone')

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
        <field name="image_128"
               type="base64"
               file="payment_ogone/static/description/icon.png"/>
        <field name="module_id" ref="base.module_payment_ogone"/>
        <field name="payment_method_ids"
               eval="[(6, 0, [
                   ref('payment.payment_method_card'),
                   ref('payment.payment_method_bancontact'),
                   ref('payment.payment_method_belfius'),
                   ref('payment.payment_method_bizum'),
                   ref('payment.payment_method_klarna_paynow'),
                   ref('payment.payment_method_klarna_pay_over_time'),
                   ref('payment.payment_method_paypal'),
                   ref('payment.payment_method_sofort'),
                   ref('payment.payment_method_twint'),
                   ref('payment.payment_method_axis'),
                   ref('payment.payment_method_eps'),
                   ref('payment.payment_method_paypal'),
                   ref('payment.payment_method_sofort'),
                   ref('payment.payment_method_paylib'),
                   ref('payment.payment_method_p24'),
               ])]"/>
        <field name="code">ogone</field>
        <field name="redirect_form_view_id" ref="redirect_form"/>
        <field name="allow_tokenization">True</field>
    </record>

</odoo>

```

## File: models\payment_provider.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
from hashlib import new as hashnew

import requests

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError

from odoo.addons.payment_ogone import const


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
            return not incoming or _key in const.VALID_KEYS

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

    def _get_default_payment_method_codes(self):
        """ Override of `payment` to return the default payment method codes. """
        default_codes = super()._get_default_payment_method_codes()
        if self.code != 'ogone':
            return default_codes
        return const.DEFAULT_PAYMENT_METHODS_CODES

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

from odoo.addons.payment import utils as payment_utils
from odoo.addons.payment_ogone import const
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
            'CN': self.partner_name or '',  # Cardholder Name
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
            'PM': const.PAYMENT_METHODS_MAPPING.get(
                self.payment_method_code, self.payment_method_code
            ),
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

        # Update the provider reference.
        self.provider_reference = notification_data.get('PAYID')

        # Update the payment method.
        payment_method_code = notification_data.get('BRAND', '')
        payment_method = self.env['payment.method']._get_from_code(
            payment_method_code, mapping=const.PAYMENT_METHODS_MAPPING
        )
        self.payment_method_id = payment_method or self.payment_method_id

        # Update the payment state.
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
            'payment_method_id': self.payment_method_id.id,
            'payment_details': notification_data.get('CARDNO')[-4:],  # Ogone pads details with X's.
            'partner_id': self.partner_id.id,
            'provider_ref': notification_data['ALIAS'],
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
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M9.241 22.46c-.204-1.014-.885-1.658-1.82-2.057-.789-.334-1.618-.422-2.473-.388-.95.04-1.853.246-2.674.718-1.498.861-2.34 2.112-2.27 3.866.044 1.094.562 1.906 1.534 2.436.79.427 1.651.55 2.546.594.472-.059.953-.09 1.418-.188 1.203-.253 2.236-.808 2.98-1.793.718-.951.993-2.023.76-3.189Zm-4.657 3.874c-.719.14-1.39.029-1.927-.512-.417-.424-.54-.952-.497-1.519.056-.699.273-1.349.699-1.918.586-.786 1.359-1.24 2.11-1.263 1.285 0 1.991.536 2.144 1.517.228 1.486-.6 3.323-2.529 3.695Zm35.046-3.666c-.28 1.38-.626 2.746-.942 4.118-.037.159-.07.317-.118.471-.017.047-.08.109-.121.109-.64.006-1.278.002-1.918 0-.016 0-.03-.017-.066-.034.055-.256.106-.518.166-.777.276-1.261.567-2.52.83-3.784a1.85 1.85 0 0 0-.002-.728c-.082-.424-.503-.72-.976-.756-.694-.05-1.323.172-1.9.507-.492.281-.945.621-1.411.948a.477.477 0 0 0-.156.262 487.61 487.61 0 0 0-.92 4.158c-.035.163-.104.217-.274.215-.547-.011-1.094-.005-1.642-.005-.065 0-.13-.011-.226-.018.072-.343.138-.664.207-.986.42-1.933.833-3.866 1.254-5.8.086-.396.009-.344.425-.346.526-.002 1.05.003 1.575 0 .139-.002.197.021.158.18-.087.345-.147.699-.229 1.082.196-.134.355-.251.524-.362.718-.487 1.497-.834 2.365-.955.79-.109 1.571-.096 2.303.272.876.445 1.278 1.32 1.094 2.229Zm10.354-.036c-.13-.968-.699-1.647-1.555-2.098-.797-.423-1.656-.555-2.558-.522a6.339 6.339 0 0 0-2.098.416c-1.084.428-1.975 1.093-2.48 2.152-.543 1.139-.54 2.282.19 3.354.487.715 1.21 1.123 2.033 1.377.902.278 1.83.316 2.772.243a8.802 8.802 0 0 0 2.515-.585c.274-.104.44-.243.476-.55.036-.307.129-.61.208-.962-.114.048-.175.075-.24.096-.569.212-1.128.453-1.71.628-.912.276-1.845.348-2.778.094-1.138-.31-1.898-1.099-1.679-2.368.03-.161.08-.234.27-.234 2.112.008 4.223.007 6.337.007.045 0 .095.014.133-.004.054-.025.13-.07.132-.11.026-.31.072-.626.032-.934Zm-2.254.078h-4.254c.485-.897 1.176-1.433 2.195-1.527.423-.039.83-.017 1.232.107.576.18.929.567 1.045 1.154.046.232.02.266-.218.266Zm-19.854-1.972c-1.151-.75-2.442-.838-3.76-.659-.542.075-1.078.151-1.628.148-2.608-.01-5.216-.003-7.826-.003-.52 0-1.034.066-1.532.208-.799.231-1.523.594-2.03 1.264-.527.697-.696 1.455-.262 2.262.224.416.6.683 1.018.893.183.092.378.168.56.248-.293.091-.588.171-.875.274-.329.116-.639.273-.886.525-.338.345-.338.803.028 1.111.186.158.413.277.64.377.29.124.6.207.864.294-.413.081-.865.156-1.31.26-.461.104-.913.243-1.305.522-.279.2-.434.453-.39.806.034.26.172.46.358.633.332.316.739.504 1.168.647 1.176.384 2.389.484 3.622.444a9.155 9.155 0 0 0 1.741-.214c.671-.154 1.312-.381 1.841-.84.676-.58.63-1.484-.007-2.007-.385-.315-.837-.495-1.308-.637-.965-.292-1.97-.394-2.959-.579a6.182 6.182 0 0 1-1.03-.286c-.235-.09-.24-.302-.026-.437.132-.084.284-.16.437-.183.66-.113 1.322-.218 1.986-.303.67-.087 1.326-.21 1.943-.49.56-.249 1.057-.583 1.39-1.104.468-.727.545-1.68-.133-2.388-.054-.056-.103-.112-.166-.18h3.18c-.152.163-.296.307-.426.46-.79.938-1.106 2.016-.908 3.212.18 1.077.844 1.801 1.86 2.194 1.319.511 2.673.511 4.024.132 1.914-.541 3.489-2.116 3.38-4.393-.047-.953-.477-1.695-1.274-2.211Zm-13.93 7.37c.23-.032.483.055.718.116.38.098.754.212 1.126.336.141.052.276.136.397.226.246.183.256.415.036.63-.236.234-.54.339-.855.392-.43.073-.867.112-1.17.15-.867-.053-1.606-.045-2.312-.28-.18-.06-.36-.147-.514-.256-.237-.167-.247-.413-.004-.568.25-.16.524-.306.81-.38a15.548 15.548 0 0 1 1.767-.365Zm2.407-4.483c-.347.429-.814.653-1.358.735-.146.02-.294.03-.399.04-.589-.025-1.105-.128-1.53-.496-.489-.42-.582-1.066-.212-1.59.39-.553.952-.831 1.62-.921.475-.066.93.001 1.363.2.483.22.8.578.835 1.116.023.34-.106.65-.319.916Zm9.616 1.924c-.566.551-1.267.819-1.845.831-1.179.002-1.91-.558-2.086-1.454-.12-.598-.004-1.182.224-1.739.39-.967 1.053-1.67 2.095-1.972.45-.128.909-.142 1.363-.006.714.21 1.13.704 1.23 1.407.156 1.125-.158 2.122-.981 2.933Z" fill="#2873B9"/></svg>

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
            <input type="hidden" name="CN" t-att-value="CN"/>
            <input type="hidden" name="OWNERADDRESS" t-att-value="OWNERADDRESS"/>
            <input type="hidden" name="OWNERZIP" t-att-value="OWNERZIP"/>
            <input type="hidden" name="OWNERTOWN" t-att-value="OWNERTOWN"/>
            <input type="hidden" name="OWNERCTY" t-att-value="OWNERCTY"/>
            <input type="hidden" name="OWNERTELNO" t-att-value="OWNERTELNO"/>
            <input type="hidden" name="OPERATION" t-att-value="OPERATION"/>
            <input type="hidden" name="USERID" t-att-value="USERID"/>
            <input type="hidden" name="PM" t-att-value="PM"/>
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
                     invisible="code != 'ogone'">
                    This provider is deprecated.
                    Consider disabling it and moving to <strong>Stripe</strong>.
                </div>
            </xpath>
            <group name="provider_credentials" position="inside">
                <group invisible="code != 'ogone'">
                    <field name="ogone_pspid" required="code == 'ogone' and state != 'disabled'"/>
                    <field name="ogone_userid" required="code == 'ogone' and state != 'disabled'"/>
                    <field name="ogone_password" required="code == 'ogone' and state != 'disabled'" password="True"/>
                    <field name="ogone_shakey_in" required="code == 'ogone' and state != 'disabled'" password="True"/>
                    <field name="ogone_shakey_out" required="code == 'ogone' and state != 'disabled'" password="True"/>
                    <field name="ogone_hash_function" required="code == 'ogone' and state != 'disabled'" groups="base.group_no_one"/>
                </group>
            </group>
        </field>
    </record>

</odoo>

```

