# Odoo Module: payment_authorize

Category: Accounting/Payment Providers

This file contains the source code of the Odoo module.

## File: const.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# The codes of the payment methods to activate when Authorize is activated.
DEFAULT_PAYMENT_METHODS_CODES = [
    # Primary payment methods.
    'ach_direct_debit',
    'card',
    # Brand payment methods.
    'visa',
    'mastercard',
    'amex',
    'discover',
]

# Mapping of payment method codes to Authorize codes.
PAYMENT_METHODS_MAPPING = {
    'amex': 'americanexpress',
    'diners': 'dinersclub',
    'card': 'creditcard'
}

# Mapping of payment status on Authorize side to transaction statuses.
# See https://developer.authorize.net/api/reference/index.html#transaction-reporting-get-transaction-details.
TRANSACTION_STATUS_MAPPING = {
    'authorized': ['authorizedPendingCapture', 'capturedPendingSettlement'],
    'captured': ['settledSuccessfully'],
    'voided': ['voided'],
    'refunded': ['refundPendingSettlement', 'refundSettledSuccessfully'],
}

```

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

from odoo.addons.payment import setup_provider, reset_payment_provider


def post_init_hook(env):
    setup_provider(env, 'authorize')


def uninstall_hook(env):
    reset_payment_provider(env, 'authorize')

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Payment Provider: Authorize.Net',
    'version': '2.0',
    'category': 'Accounting/Payment Providers',
    'sequence': 350,
    'summary': "An payment provider covering the US, Australia, and Canada.",
    'description': " ",  # Non-empty string to avoid loading the README file.
    'depends': ['payment'],
    'data': [
        'views/payment_authorize_templates.xml',
        'views/payment_provider_views.xml',
        'views/payment_token_views.xml',

        'data/payment_provider_data.xml',
    ],
    'post_init_hook': 'post_init_hook',
    'uninstall_hook': 'uninstall_hook',
    'assets': {
        'web.assets_frontend': [
            'payment_authorize/static/src/scss/payment_authorize.scss',
            'payment_authorize/static/src/js/payment_form.js',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import pprint

from odoo import _, http
from odoo.exceptions import ValidationError
from odoo.http import request

from odoo.addons.payment import utils as payment_utils

_logger = logging.getLogger(__name__)


class AuthorizeController(http.Controller):

    @http.route('/payment/authorize/payment', type='json', auth='public')
    def authorize_payment(self, reference, partner_id, access_token, opaque_data):
        """ Make a payment request and handle the response.

        :param str reference: The reference of the transaction
        :param int partner_id: The partner making the transaction, as a `res.partner` id
        :param str access_token: The access token used to verify the provided values
        :param dict opaque_data: The payment details obfuscated by Authorize.Net
        :return: None
        """
        # Check that the transaction details have not been altered
        if not payment_utils.check_access_token(access_token, reference, partner_id):
            raise ValidationError("Authorize.Net: " + _("Received tampered payment request data."))

        # Make the payment request to Authorize.Net
        tx_sudo = request.env['payment.transaction'].sudo().search([('reference', '=', reference)])
        response_content = tx_sudo._authorize_create_transaction_request(opaque_data)

        # Handle the payment request response
        _logger.info(
            "payment request response for transaction with reference %s:\n%s",
            reference, pprint.pformat(response_content)
        )
        tx_sudo._handle_notification_data('authorize', {'response': response_content})

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\neutralize.sql

```sql
-- disable authorize payment provider
UPDATE payment_provider
   SET authorize_login = NULL,
       authorize_transaction_key = NULL,
       authorize_signature_key = NULL,
       authorize_client_key = NULL;

```

## File: data\payment_provider_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="payment.payment_provider_authorize" model="payment.provider">
        <field name="code">authorize</field>
        <field name="inline_form_view_id" ref="inline_form"/>
        <field name="allow_tokenization">True</field>
    </record>

</odoo>

```

## File: models\authorize_request.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json
import logging
import pprint

from uuid import uuid4

import requests

from odoo.addons.payment import utils as payment_utils

_logger = logging.getLogger(__name__)


class AuthorizeAPI:
    """ Authorize.net Gateway API integration.

    This class allows contacting the Authorize.net API with simple operation
    requests. It implements a *very limited* subset of the complete API
    (http://developer.authorize.net/api/reference); namely:
        - Customer Profile/Payment Profile creation
        - Transaction authorization/capture/voiding
    """

    AUTH_ERROR_STATUS = '3'

    def __init__(self, provider):
        """Initiate the environment with the provider data.

        :param recordset provider: payment.provider account that will be contacted
        """
        if provider.state == 'enabled':
            self.url = 'https://api.authorize.net/xml/v1/request.api'
        else:
            self.url = 'https://apitest.authorize.net/xml/v1/request.api'

        self.state = provider.state
        self.name = provider.authorize_login
        self.transaction_key = provider.authorize_transaction_key

    def _make_request(self, operation, data=None):
        request = {
            operation: {
                'merchantAuthentication': {
                    'name': self.name,
                    'transactionKey': self.transaction_key,
                },
                **(data or {})
            }
        }
        logged_request = {operation: data or {}}

        _logger.info("sending request to %s:\n%s", self.url, pprint.pformat(logged_request))
        response = requests.post(self.url, json.dumps(request), timeout=60)
        response.raise_for_status()
        response = json.loads(response.content)
        _logger.info("response received:\n%s", pprint.pformat(response))

        messages = response.get('messages')
        if messages and messages.get('resultCode') == 'Error':
            err_msg = messages.get('message')[0]['text']

            tx_errors = response.get('transactionResponse', {}).get('errors')
            if tx_errors:
                if err_msg:
                    err_msg += '\n'
                err_msg += '\n'.join([e.get('errorText', '') for e in tx_errors])

            return {
                'err_code': messages.get('message')[0].get('code'),
                'err_msg': err_msg,
            }

        return response

    def _format_response(self, response, operation):
        if response and response.get('err_code'):
            return {
                'x_response_code': self.AUTH_ERROR_STATUS,
                'x_response_reason_text': response.get('err_msg')
            }
        else:
            tx_response = response.get('transactionResponse', {})
            return {
                'x_response_code': tx_response.get('responseCode'),
                'x_trans_id': tx_response.get('transId'),
                'x_type': operation,
                'payment_method_code': tx_response.get('accountType'),
            }

    # Customer profiles
    def create_customer_profile(self, partner, transaction_id):
        """ Create an Auth.net payment/customer profile from an existing transaction.

        Creates a customer profile for the partner/credit card combination and links
        a corresponding payment profile to it. Note that a single partner in the Odoo
        database can have multiple customer profiles in Authorize.net (i.e. a customer
        profile is created for every res.partner/payment.token couple).

        Note that this function makes 2 calls to the authorize api, since we need to
        obtain a partial card number to generate a meaningful payment.token name.

        :param record partner: the res.partner record of the customer
        :param str transaction_id: id of the authorized transaction in the
                                   Authorize.net backend

        :return: a dict containing the profile_id and payment_profile_id of the
                 newly created customer profile and payment profile as well as the
                 last digits of the card number
        :rtype: dict
        """
        response = self._make_request('createCustomerProfileFromTransactionRequest', {
            'transId': transaction_id,
            'customer': {
                'merchantCustomerId': ('ODOO-%s-%s' % (partner.id, uuid4().hex[:8]))[:20],
                'email': partner.email or ''
            }
        })

        if not response.get('customerProfileId'):
            _logger.warning(
                "unable to create customer payment profile, data missing from transaction with "
                "id %(tx_id)s, partner id: %(partner_id)s",
                {
                    'tx_id': transaction_id,
                    'partner_id': partner,
                },
            )
            return False

        res = {
            'profile_id': response.get('customerProfileId'),
            'payment_profile_id': response.get('customerPaymentProfileIdList')[0]
        }

        response = self._make_request('getCustomerPaymentProfileRequest', {
            'customerProfileId': res['profile_id'],
            'customerPaymentProfileId': res['payment_profile_id'],
        })

        payment = response.get('paymentProfile', {}).get('payment', {})
        if 'creditCard' in payment:
            # Authorize.net pads the card and account numbers with X's.
            res['payment_details'] = payment.get('creditCard', {}).get('cardNumber')[-4:]
        else:
            res['payment_details'] = payment.get('bankAccount', {}).get('accountNumber')[-4:]
        return res

    def delete_customer_profile(self, profile_id):
        """Delete a customer profile

        :param str profile_id: the id of the customer profile in the Authorize.net backend

        :return: a dict containing the response code
        :rtype: dict
        """
        response = self._make_request("deleteCustomerProfileRequest", {'customerProfileId': profile_id})
        return self._format_response(response, 'deleteCustomerProfile')

    #=== Transaction management ===#
    def _prepare_authorization_transaction_request(self, transaction_type, tx_data, tx):
        # The billTo parameter is required for new ACH transactions (transactions without a payment.token),
        # but is not allowed for transactions with a payment.token.
        bill_to = {}
        if 'profile' not in tx_data:
            if tx.partner_id.is_company:
                split_name = '', tx.partner_name
            else:
                split_name = payment_utils.split_partner_name(tx.partner_name)
            # max lengths are defined by the Authorize API
            bill_to = {
                'billTo': {
                    'firstName': split_name[0][:50],
                    'lastName': split_name[1][:50],  # lastName is always required
                    'company': tx.partner_name[:50] if tx.partner_id.is_company else '',
                    'address': tx.partner_address,
                    'city': tx.partner_city,
                    'state': tx.partner_state_id.name or '',
                    'zip': tx.partner_zip,
                    'country': tx.partner_country_id.name or '',
                }
            }

        # These keys have to be in the order defined in
        # https://apitest.authorize.net/xml/v1/schema/AnetApiSchema.xsd
        return {
            'transactionRequest': {
                'transactionType': transaction_type,
                'amount': str(tx.amount),
                **tx_data,
                'order': {
                    'invoiceNumber': tx.reference[:20],
                    'description': tx.reference[:255],
                },
                'customer': {
                    'email': tx.partner_email or '',
                },
                **bill_to,
                'customerIP': payment_utils.get_customer_ip_address(),
            }
        }

    def authorize(self, tx, token=None, opaque_data=None):
        """ Authorize (without capture) a payment for the given amount.

        :param recordset tx: The transaction of the payment, as a `payment.transaction` record
        :param recordset token: The token of the payment method to charge, as a `payment.token`
                                record
        :param dict opaque_data: The payment details obfuscated by Authorize.Net
        :return: a dict containing the response code, transaction id and transaction type
        :rtype: dict
        """
        tx_data = self._prepare_tx_data(token=token, opaque_data=opaque_data)
        response = self._make_request(
            'createTransactionRequest',
            self._prepare_authorization_transaction_request('authOnlyTransaction', tx_data, tx)
        )
        return self._format_response(response, 'auth_only')

    def auth_and_capture(self, tx, token=None, opaque_data=None):
        """Authorize and capture a payment for the given amount.

        Authorize and immediately capture a payment for the given payment.token
        record for the specified amount with reference as communication.

        :param recordset tx: The transaction of the payment, as a `payment.transaction` record
        :param record token: the payment.token record that must be charged
        :param str opaque_data: the transaction opaque_data obtained from Authorize.net

        :return: a dict containing the response code, transaction id and transaction type
        :rtype: dict
        """
        tx_data = self._prepare_tx_data(token=token, opaque_data=opaque_data)
        response = self._make_request(
            'createTransactionRequest',
            self._prepare_authorization_transaction_request('authCaptureTransaction', tx_data, tx)
        )

        result = self._format_response(response, 'auth_capture')
        errors = response.get('transactionResponse', {}).get('errors')
        if errors:
            result['x_response_reason_text'] = '\n'.join([e.get('errorText') for e in errors])
        return result

    def _prepare_tx_data(self, token=None, opaque_data=False):
        """
        :param token: The token of the payment method to charge, as a `payment.token` record
        :param dict opaque_data: The payment details obfuscated by Authorize.Net
        """
        assert (token or opaque_data) and not (token and opaque_data), "Exactly one of token or opaque_data must be specified"
        if token:
            token.ensure_one()
            return {
                'profile': {
                    'customerProfileId': token.authorize_profile,
                    'paymentProfile': {
                        'paymentProfileId': token.provider_ref,
                    }
                },
            }
        else:
            return {
                'payment': {
                    'opaqueData': opaque_data,
                }
            }

    def get_transaction_details(self, transaction_id):
        """ Return detailed information about a specific transaction. Useful to issue refunds.

        :param str transaction_id: transaction id
        :return: a dict containing the transaction details
        :rtype: dict
        """
        return self._make_request('getTransactionDetailsRequest', {'transId': transaction_id})

    def capture(self, transaction_id, amount):
        """Capture a previously authorized payment for the given amount.

        Capture a previously authorized payment. Note that the amount is required
        even though we do not support partial capture.

        :param str transaction_id: id of the authorized transaction in the
                                   Authorize.net backend
        :param str amount: transaction amount (up to 15 digits with decimal point)

        :return: a dict containing the response code, transaction id and transaction type
        :rtype: dict
        """
        response = self._make_request('createTransactionRequest', {
            'transactionRequest': {
                'transactionType': 'priorAuthCaptureTransaction',
                'amount': str(amount),
                'refTransId': transaction_id,
            }
        })
        return self._format_response(response, 'prior_auth_capture')

    def void(self, transaction_id):
        """Void a previously authorized payment.

        :param str transaction_id: the id of the authorized transaction in the
                                   Authorize.net backend
        :return: a dict containing the response code, transaction id and transaction type
        :rtype: dict
        """
        response = self._make_request('createTransactionRequest', {
            'transactionRequest': {
                'transactionType': 'voidTransaction',
                'refTransId': transaction_id
            }
        })
        return self._format_response(response, 'void')

    def refund(self, transaction_id, amount, tx_details):
        """Refund a previously authorized payment. If the transaction is not settled
            yet, it will be voided.

        :param str transaction_id: the id of the authorized transaction in the
                                   Authorize.net backend
        :param float amount: transaction amount to refund
        :param dict tx_details: The transaction details from `get_transaction_details()`.
        :return: a dict containing the response code, transaction id and transaction type
        :rtype: dict
        """
        card = tx_details.get('transaction', {}).get('payment', {}).get('creditCard', {}).get('cardNumber')
        response = self._make_request('createTransactionRequest', {
            'transactionRequest': {
                'transactionType': 'refundTransaction',
                'amount': str(amount),
                'payment': {
                    'creditCard': {
                        'cardNumber': card,
                        'expirationDate': 'XXXX',
                    }
                },
                'refTransId': transaction_id,
            }
        })
        return self._format_response(response, 'refund')

    # Provider configuration: fetch authorize_client_key & currencies
    def merchant_details(self):
        """ Retrieves the merchant details and generate a new public client key if none exists.

        :return: Dictionary containing the merchant details
        :rtype: dict"""
        return self._make_request('getMerchantDetailsRequest')

    # Test
    def test_authenticate(self):
        """ Test Authorize.net communication with a simple credentials check.

        :return: The authentication results
        :rtype: dict
        """
        return self._make_request('authenticateTestRequest')

```

## File: models\payment_provider.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json
import logging
import pprint

from odoo import _, api, fields, models
from odoo.exceptions import UserError, ValidationError
from odoo.fields import Command

from odoo.addons.payment_authorize import const
from odoo.addons.payment_authorize.models.authorize_request import AuthorizeAPI

_logger = logging.getLogger(__name__)


class PaymentProvider(models.Model):
    _inherit = 'payment.provider'

    code = fields.Selection(
        selection_add=[('authorize', 'Authorize.Net')], ondelete={'authorize': 'set default'})
    authorize_login = fields.Char(
        string="API Login ID", help="The ID solely used to identify the account with Authorize.Net",
        required_if_provider='authorize')
    authorize_transaction_key = fields.Char(
        string="API Transaction Key", required_if_provider='authorize', groups='base.group_system')
    authorize_signature_key = fields.Char(
        string="API Signature Key", required_if_provider='authorize', groups='base.group_system')
    authorize_client_key = fields.Char(
        string="API Client Key",
        help="The public client key. To generate directly from Odoo or from Authorize.Net backend.")

    # === CONSTRAINT METHODS ===#

    # Authorize.Net supports only one currency: "One gateway account is required for each currency"
    # See https://community.developer.authorize.net/t5/The-Authorize-Net-Developer-Blog/Authorize-Net-UK-Europe-Update/ba-p/35957
    @api.constrains('available_currency_ids', 'state')
    def _limit_available_currency_ids(self):
        for provider in self.filtered(lambda p: p.code == 'authorize'):
            if len(provider.available_currency_ids) > 1 and provider.state != 'disabled':
                raise ValidationError(
                    _("Only one currency can be selected by Authorize.Net account.")
                )

    #=== COMPUTE METHODS ===#

    def _compute_feature_support_fields(self):
        """ Override of `payment` to enable additional features. """
        super()._compute_feature_support_fields()
        self.filtered(lambda p: p.code == 'authorize').update({
            'support_manual_capture': 'full_only',
            'support_refund': 'full_only',
            'support_tokenization': True,
        })

    # === ACTION METHODS ===#

    def action_update_merchant_details(self):
        """ Fetch the merchant details to update the client key and the account currency. """
        self.ensure_one()

        if self.state == 'disabled':
            raise UserError(_("This action cannot be performed while the provider is disabled."))

        authorize_API = AuthorizeAPI(self)

        # Validate the API Login ID and Transaction Key
        res_content = authorize_API.test_authenticate()
        _logger.info("test_authenticate request response:\n%s", pprint.pformat(res_content))
        if res_content.get('err_msg'):
            raise UserError(_("Failed to authenticate.\n%s", res_content['err_msg']))

        # Update the merchant details
        res_content = authorize_API.merchant_details()
        _logger.info("merchant_details request response:\n%s", pprint.pformat(res_content))
        if res_content.get('err_msg'):
            raise UserError(_("Could not fetch merchant details:\n%s", res_content['err_msg']))

        currency = self.env['res.currency'].search([('name', 'in', res_content.get('currencies'))])
        self.available_currency_ids = [Command.set(currency.ids)]
        self.authorize_client_key = res_content.get('publicClientKey')

    # === BUSINESS METHODS ===#

    def _get_validation_amount(self):
        """ Override of payment to return the amount for Authorize.Net validation operations.

        :return: The validation amount
        :rtype: float
        """
        res = super()._get_validation_amount()
        if self.code != 'authorize':
            return res

        return 0.01

    def _authorize_get_inline_form_values(self):
        """ Return a serialized JSON of the required values to render the inline form.

        Note: `self.ensure_one()`

        :return: The JSON serial of the required values to render the inline form.
        :rtype: str
        """
        self.ensure_one()

        inline_form_values = {
            'state': self.state,
            'login_id': self.authorize_login,
            'client_key': self.authorize_client_key,
        }
        return json.dumps(inline_form_values)

    def _get_default_payment_method_codes(self):
        """ Override of `payment` to return the default payment method codes. """
        default_codes = super()._get_default_payment_method_codes()
        if self.code != 'authorize':
            return default_codes
        return const.DEFAULT_PAYMENT_METHODS_CODES

```

## File: models\payment_token.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import pprint

from odoo import _, fields, models
from odoo.exceptions import UserError

from .authorize_request import AuthorizeAPI

_logger = logging.getLogger(__name__)


class PaymentToken(models.Model):
    _inherit = 'payment.token'

    authorize_profile = fields.Char(
        string="Authorize.Net Profile ID",
        help="The unique reference for the partner/token combination in the Authorize.net backend.")

```

## File: models\payment_transaction.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import pprint

from odoo import _, models
from odoo.exceptions import UserError, ValidationError

from odoo.addons.payment import utils as payment_utils
from odoo.addons.payment_authorize.models.authorize_request import AuthorizeAPI
from odoo.addons.payment_authorize.const import PAYMENT_METHODS_MAPPING, TRANSACTION_STATUS_MAPPING


_logger = logging.getLogger(__name__)


class PaymentTransaction(models.Model):
    _inherit = 'payment.transaction'

    def _get_specific_processing_values(self, processing_values):
        """ Override of payment to return an access token as provider-specific processing values.

        Note: self.ensure_one() from `_get_processing_values`

        :param dict processing_values: The generic processing values of the transaction
        :return: The dict of provider-specific processing values
        :rtype: dict
        """
        res = super()._get_specific_processing_values(processing_values)
        if self.provider_code != 'authorize':
            return res

        return {
            'access_token': payment_utils.generate_access_token(
                processing_values['reference'], processing_values['partner_id']
            )
        }

    def _authorize_create_transaction_request(self, opaque_data):
        """ Create an Authorize.Net payment transaction request.

        Note: self.ensure_one()

        :param dict opaque_data: The payment details obfuscated by Authorize.Net
        :return:
        """
        self.ensure_one()

        authorize_API = AuthorizeAPI(self.provider_id)
        if self.provider_id.capture_manually or self.operation == 'validation':
            return authorize_API.authorize(self, opaque_data=opaque_data)
        else:
            return authorize_API.auth_and_capture(self, opaque_data=opaque_data)

    def _send_payment_request(self):
        """ Override of payment to send a payment request to Authorize.

        Note: self.ensure_one()

        :return: None
        :raise: UserError if the transaction is not linked to a token
        """
        super()._send_payment_request()
        if self.provider_code != 'authorize':
            return

        if not self.token_id.authorize_profile:
            raise UserError("Authorize.Net: " + _("The transaction is not linked to a token."))

        authorize_API = AuthorizeAPI(self.provider_id)
        if self.provider_id.capture_manually:
            res_content = authorize_API.authorize(self, token=self.token_id)
            _logger.info(
                "authorize request response for transaction with reference %s:\n%s",
                self.reference, pprint.pformat(res_content)
            )
        else:
            res_content = authorize_API.auth_and_capture(self, token=self.token_id)
            _logger.info(
                "auth_and_capture request response for transaction with reference %s:\n%s",
                self.reference, pprint.pformat(res_content)
            )
        self._handle_notification_data('authorize', {'response': res_content})

    def _send_refund_request(self, amount_to_refund=None):
        """ Override of payment to send a refund request to Authorize.

        Note: self.ensure_one()

        :param float amount_to_refund: The amount to refund
        :return: The refund transaction created to process the refund request.
        :rtype: recordset of `payment.transaction`
        """
        self.ensure_one()

        if self.provider_code != 'authorize':
            return super()._send_refund_request(amount_to_refund=amount_to_refund)

        authorize_api = AuthorizeAPI(self.provider_id)
        tx_details = authorize_api.get_transaction_details(self.provider_reference)
        if 'err_code' in tx_details:  # Could not retrieve the transaction details.
            raise ValidationError("Authorize.Net: " + _(
                "Could not retrieve the transaction details. (error code: %s; error_details: %s)",
                tx_details['err_code'], tx_details.get('err_msg')
            ))

        refund_tx = self.env['payment.transaction']
        tx_status = tx_details.get('transaction', {}).get('transactionStatus')
        if tx_status in TRANSACTION_STATUS_MAPPING['voided']:
            # The payment has been voided from Authorize.net side before we could refund it.
            self._set_canceled(extra_allowed_states=('done',))
        elif tx_status in TRANSACTION_STATUS_MAPPING['refunded']:
            # The payment has been refunded from Authorize.net side before we could refund it. We
            # create a refund tx on Odoo to reflect the move of the funds.
            refund_tx = super()._send_refund_request(amount_to_refund=amount_to_refund)
            refund_tx._set_done()
            # Immediately post-process the transaction as the post-processing will not be
            # triggered by a customer browsing the transaction from the portal.
            self.env.ref('payment.cron_post_process_payment_tx')._trigger()
        elif any(tx_status in TRANSACTION_STATUS_MAPPING[k] for k in ('authorized', 'captured')):
            if tx_status in TRANSACTION_STATUS_MAPPING['authorized']:
                # The payment has not been settled on Authorize.net yet. It must be voided rather
                # than refunded. Since the funds have not moved yet, we don't create a refund tx.
                res_content = authorize_api.void(self.provider_reference)
                tx_to_process = self
            else:
                # The payment has been settled on Authorize.net side. We can refund it.
                refund_tx = super()._send_refund_request(amount_to_refund=amount_to_refund)
                rounded_amount = round(amount_to_refund, self.currency_id.decimal_places)
                res_content = authorize_api.refund(
                    self.provider_reference, rounded_amount, tx_details
                )
                tx_to_process = refund_tx
            _logger.info(
                "refund request response for transaction with reference %s:\n%s",
                self.reference, pprint.pformat(res_content)
            )
            data = {'reference': tx_to_process.reference, 'response': res_content}
            tx_to_process._handle_notification_data('authorize', data)
        else:
            raise ValidationError("Authorize.net: " + _(
                "The transaction is not in a status to be refunded. (status: %s, details: %s)",
                tx_status, tx_details.get('messages', {}).get('message')
            ))
        return refund_tx

    def _send_capture_request(self, amount_to_capture=None):
        """ Override of `payment` to send a capture request to Authorize. """
        child_capture_tx = super()._send_capture_request(amount_to_capture=amount_to_capture)
        if self.provider_code != 'authorize':
            return child_capture_tx

        authorize_API = AuthorizeAPI(self.provider_id)
        rounded_amount = round(self.amount, self.currency_id.decimal_places)
        res_content = authorize_API.capture(self.provider_reference, rounded_amount)
        _logger.info(
            "capture request response for transaction with reference %s:\n%s",
            self.reference, pprint.pformat(res_content)
        )
        self._handle_notification_data('authorize', {'response': res_content})

        return child_capture_tx

    def _send_void_request(self, amount_to_void=None):
        """ Override of payment to send a void request to Authorize. """
        child_void_tx = super()._send_void_request(amount_to_void=amount_to_void)
        if self.provider_code != 'authorize':
            return child_void_tx

        authorize_API = AuthorizeAPI(self.provider_id)
        res_content = authorize_API.void(self.provider_reference)
        _logger.info(
            "void request response for transaction with reference %s:\n%s",
            self.reference, pprint.pformat(res_content)
        )
        self._handle_notification_data('authorize', {'response': res_content})

        return child_void_tx

    def _get_tx_from_notification_data(self, provider_code, notification_data):
        """ Find the transaction based on Authorize.net data.

        :param str provider_code: The code of the provider that handled the transaction
        :param dict notification_data: The notification data sent by the provider
        :return: The transaction if found
        :rtype: recordset of `payment.transaction`
        """
        tx = super()._get_tx_from_notification_data(provider_code, notification_data)
        if provider_code != 'authorize' or len(tx) == 1:
            return tx

        reference = notification_data.get('reference')
        tx = self.search([('reference', '=', reference), ('provider_code', '=', 'authorize')])
        if not tx:
            raise ValidationError(
                "Authorize.Net: " + _("No transaction found matching reference %s.", reference)
            )
        return tx

    def _process_notification_data(self, notification_data):
        """ Override of payment to process the transaction based on Authorize data.

        Note: self.ensure_one()

        :param dict notification_data: The notification data sent by the provider
        :return: None
        """
        super()._process_notification_data(notification_data)
        if self.provider_code != 'authorize':
            return

        response_content = notification_data.get('response')

        # Update the provider reference.
        self.provider_reference = response_content.get('x_trans_id')

        # Update the payment method.
        payment_method_code = response_content.get('payment_method_code', '').lower()
        payment_method = self.env['payment.method']._get_from_code(
            payment_method_code, mapping=PAYMENT_METHODS_MAPPING
        )
        self.payment_method_id = payment_method or self.payment_method_id

        # Update the payment state.
        status_code = response_content.get('x_response_code', '3')
        if status_code == '1':  # Approved
            status_type = response_content.get('x_type').lower()
            if status_type in ('auth_capture', 'prior_auth_capture'):
                self._set_done()
                if self.tokenize and not self.token_id:
                    self._authorize_tokenize()
            elif status_type == 'auth_only':
                self._set_authorized()
                if self.tokenize and not self.token_id:
                    self._authorize_tokenize()
                if self.operation == 'validation':
                    self._send_void_request()  # In last step because it processes the response.
            elif status_type == 'void':
                if self.operation == 'validation':  # Validation txs are authorized and then voided
                    self._set_done()  # If the refund went through, the validation tx is confirmed
                else:
                    self._set_canceled(extra_allowed_states=('done',))
            elif status_type == 'refund' and self.operation == 'refund':
                self._set_done()
                # Immediately post-process the transaction as the post-processing will not be
                # triggered by a customer browsing the transaction from the portal.
                self.env.ref('payment.cron_post_process_payment_tx')._trigger()
        elif status_code == '2':  # Declined
            self._set_canceled(state_message=response_content.get('x_response_reason_text'))
        elif status_code == '4':  # Held for Review
            self._set_pending()
        else:  # Error / Unknown code
            error_code = response_content.get('x_response_reason_text')
            _logger.info(
                "received data with invalid status (%(status)s) and error code (%(err)s) for "
                "transaction with reference %(ref)s",
                {
                    'status': status_code,
                    'err': error_code,
                    'ref': self.reference,
                },
            )
            self._set_error(
                "Authorize.Net: " + _(
                    "Received data with status code \"%(status)s\" and error code \"%(error)s\"",
                    status=status_code, error=error_code
                )
            )

    def _authorize_tokenize(self):
        """ Create a token for the current transaction.

        Note: self.ensure_one()

        :return: None
        """
        self.ensure_one()

        authorize_API = AuthorizeAPI(self.provider_id)
        cust_profile = authorize_API.create_customer_profile(
            self.partner_id, self.provider_reference
        )
        _logger.info(
            "create_customer_profile request response for transaction with reference %s:\n%s",
            self.reference, pprint.pformat(cust_profile)
        )
        if cust_profile:
            token = self.env['payment.token'].create({
                'provider_id': self.provider_id.id,
                'payment_method_id': self.payment_method_id.id,
                'payment_details': cust_profile.get('payment_details'),
                'partner_id': self.partner_id.id,
                'provider_ref': cust_profile.get('payment_profile_id'),
                'authorize_profile': cust_profile.get('profile_id'),
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
from . import payment_token
from . import payment_transaction

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><circle cx="18.5" cy="18.5" r="14.5" fill="#3477F6"/><circle cx="37.5" cy="37.5" r="8.5" fill="#F5C142"/><path d="M43.105 4h.972V.818h1.108V0H42v.818h1.105V4Zm2.775 0h.858V1.453h.053L47.663 4h.555l.871-2.547h.056V4H50V0h-1.108l-.924 2.714h-.05L46.99 0h-1.11v4Z" fill="#D1D5DB"/></svg>

```

## File: static\src\js\payment_form.js

```javascript
/** @odoo-module **/
/* global Accept */

import { _t } from '@web/core/l10n/translation';
import { loadJS } from '@web/core/assets';

import paymentForm from '@payment/js/payment_form';
import { RPCError } from '@web/core/network/rpc_service';

paymentForm.include({

    authorizeData: undefined,

    // #=== DOM MANIPULATION ===#

    /**
     * Prepare the inline form of Authorize.net for direct payment.
     *
     * @private
     * @param {number} providerId - The id of the selected payment option's provider.
     * @param {string} providerCode - The code of the selected payment option's provider.
     * @param {number} paymentOptionId - The id of the selected payment option.
     * @param {string} paymentMethodCode - The code of the selected payment method, if any.
     * @param {string} flow - The online payment flow of the selected payment option.
     * @return {void}
     */
    async _prepareInlineForm(providerId, providerCode, paymentOptionId, paymentMethodCode, flow) {
        if (providerCode !== 'authorize') {
            await this._super(...arguments);
            return;
        }

        // Check if the inline form values were already extracted.
        this.authorizeData ??= {}; // Store the form data of each instantiated payment method.
        if (flow === 'token') {
            return; // Don't show the form for tokens.
        } else if (this.authorizeData[paymentOptionId]) {
            this._setPaymentFlow('direct'); // Overwrite the flow even if no re-instantiation.
            await loadJS(this.authorizeData[paymentOptionId]['acceptJSUrl']); // Reload the SDK.
            return; // Don't re-extract the data if already done for this payment method.
        }

        // Overwrite the flow of the selected payment method.
        this._setPaymentFlow('direct');

        // Extract and deserialize the inline form values.
        const radio = document.querySelector('input[name="o_payment_radio"]:checked');
        const inlineForm = this._getInlineForm(radio);
        const authorizeForm = inlineForm.querySelector('[name="o_authorize_form"]');
        this.authorizeData[paymentOptionId] = JSON.parse(
            authorizeForm.dataset['authorizeInlineFormValues']
        );
        let acceptJSUrl = 'https://js.authorize.net/v1/Accept.js';
        if (this.authorizeData[paymentOptionId].state !== 'enabled') {
            acceptJSUrl = 'https://jstest.authorize.net/v1/Accept.js';
        }
        this.authorizeData[paymentOptionId].form = authorizeForm;
        this.authorizeData[paymentOptionId].acceptJSUrl = acceptJSUrl;

        // Load the SDK.
        await loadJS(acceptJSUrl);
    },

    // #=== PAYMENT FLOW ===#

    /**
     * Trigger the payment processing by submitting the data.
     *
     * @override method from payment.payment_form
     * @private
     * @param {string} providerCode - The code of the selected payment option's provider.
     * @param {number} paymentOptionId - The id of the selected payment option.
     * @param {string} paymentMethodCode - The code of the selected payment method, if any.
     * @param {string} flow - The payment flow of the selected payment option.
     * @return {void}
     */
    async _initiatePaymentFlow(providerCode, paymentOptionId, paymentMethodCode, flow) {
        if (providerCode !== 'authorize' || flow === 'token') {
            await this._super(...arguments); // Tokens are handled by the generic flow
            return;
        }

        const inputs = Object.values(
            this._authorizeGetInlineFormInputs(paymentOptionId, paymentMethodCode)
        );
        if (!inputs.every(element => element.reportValidity())) {
            this._enableButton(); // The submit button is disabled at this point, enable it
            return;
        }

        await this._super(...arguments);
    },

    /**
     * Process the direct payment flow.
     *
     * @override method from payment.payment_form
     * @private
     * @param {string} providerCode - The code of the selected payment option's provider.
     * @param {number} paymentOptionId - The id of the selected payment option.
     * @param {string} paymentMethodCode - The code of the selected payment method, if any.
     * @param {object} processingValues - The processing values of the transaction.
     * @return {void}
     */
    async _processDirectFlow(providerCode, paymentOptionId, paymentMethodCode, processingValues) {
        if (providerCode !== 'authorize') {
            this._super(...arguments);
            return;
        }

        // Build the authentication and card data objects to be dispatched to Authorized.Net
        const secureData = {
            authData: {
                apiLoginID: this.authorizeData[paymentOptionId]['login_id'],
                clientKey: this.authorizeData[paymentOptionId]['client_key'],
            },
            ...this._authorizeGetPaymentDetails(paymentOptionId, paymentMethodCode),
        };

        // Dispatch secure data to Authorize.Net to get a payment nonce in return
        Accept.dispatchData(
            secureData, response => this._authorizeHandleResponse(response, processingValues)
        );
    },

    /**
     * Handle the response from Authorize.Net and initiate the payment.
     *
     * @private
     * @param {object} response - The payment nonce returned by Authorized.Net
     * @param {object} processingValues - The processing values of the transaction.
     * @return {void}
     */
    _authorizeHandleResponse(response, processingValues) {
        if (response.messages.resultCode === 'Error') {
            let error = '';
            response.messages.message.forEach(msg => error += `${msg.code}: ${msg.text}\n`);
            this._displayErrorDialog(_t("Payment processing failed"), error);
            this._enableButton();
            return;
        }

        // Initiate the payment
        this.rpc('/payment/authorize/payment', {
            'reference': processingValues.reference,
            'partner_id': processingValues.partner_id,
            'opaque_data': response.opaqueData,
            'access_token': processingValues.access_token,
        }).then(() => {
            window.location = '/payment/status';
        }).catch((error) => {
            if (error instanceof RPCError) {
                this._displayErrorDialog(_t("Payment processing failed"), error.data.message);
                this._enableButton();
            } else {
                return Promise.reject(error);
            }
        });
    },

    // #=== GETTERS ===#

    /**
     * Return all relevant inline form inputs based on the payment method type of the provider.
     *
     * @private
     * @param {number} paymentOptionId - The id of the selected payment option.
     * @param {string} paymentMethodCode - The code of the selected payment method, if any.
     * @return {Object} - An object mapping the name of inline form inputs to their DOM element
     */
    _authorizeGetInlineFormInputs(paymentOptionId, paymentMethodCode) {
        const form = this.authorizeData[paymentOptionId]['form'];
        if (paymentMethodCode === 'card') {
            return {
                card: form.querySelector('#o_authorize_card'),
                month: form.querySelector('#o_authorize_month'),
                year: form.querySelector('#o_authorize_year'),
                code: form.querySelector('#o_authorize_code'),
            };
        } else {
            return {
                accountName: form.querySelector('#o_authorize_account_name'),
                accountNumber: form.querySelector('#o_authorize_account_number'),
                abaNumber: form.querySelector('#o_authorize_aba_number'),
                accountType: form.querySelector('#o_authorize_account_type'),
            };
        }
    },

    /**
     * Return the credit card or bank data to pass to the Accept.dispatch request.
     *
     * @private
     * @param {number} paymentOptionId - The id of the selected payment option.
     * @param {string} paymentMethodCode - The code of the selected payment method, if any.
     * @return {Object} - Data to pass to the Accept.dispatch request
     */
    _authorizeGetPaymentDetails(paymentOptionId, paymentMethodCode) {
        const inputs = this._authorizeGetInlineFormInputs(paymentOptionId, paymentMethodCode);
        if (paymentMethodCode === 'card') {
            return {
                cardData: {
                    cardNumber: inputs.card.value.replace(/ /g, ''), // Remove all spaces
                    month: inputs.month.value,
                    year: inputs.year.value,
                    cardCode: inputs.code.value,
                },
            };
        } else {
            return {
                bankData: {
                    nameOnAccount: inputs.accountName.value.substring(0, 22), // Max allowed by acceptjs
                    accountNumber: inputs.accountNumber.value,
                    routingNumber: inputs.abaNumber.value,
                    accountType: inputs.accountType.value,
                },
            };
        }
    },

});

```

## File: views\payment_authorize_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="inline_form">
        <div name="o_authorize_form"
             class="o_authorize_form"
             t-att-data-authorize-inline-form-values="provider_sudo._authorize_get_inline_form_values()"
        >
            <t t-if="pm_sudo.code == 'card'">
                <div class="mb-3">
                    <label for="o_authorize_card" class="col-form-label">Card Number</label>
                    <input type="text" id="o_authorize_card" required="" maxlength="19" class="form-control"/>
                </div>
                <div class="row">
                    <div class="col-sm-8 mb-3">
                        <label for="o_authorize_month">Expiration</label>
                        <div class="input-group">
                            <input type="number" id="o_authorize_month" placeholder="MM" min="1" max="12" required="" class="form-control"/>
                            <input type="number" id="o_authorize_year" placeholder="YY" min="00" max="99" required="" class="form-control"/>
                        </div>
                    </div>
                    <div class="col-sm-4 mb-3">
                        <label for="o_authorize_code">Card Code</label>
                        <input type="number" id="o_authorize_code" max="9999" class="form-control"/>
                    </div>
                </div>
            </t>
            <t t-else="" >
                <div class="mb-3">
                    <label for="o_authorize_bank_name" class="col-form-label">Bank Name</label>
                    <input type="text" id="o_authorize_bank_name" required="" class="form-control"/>
                </div>
                <div class="mb-3">
                    <label for="o_authorize_account_name" class="col-form-label">Name On Account</label>
                    <input type="text" id="o_authorize_account_name" required="" class="form-control"/>
                </div>
                <div class="mb-3">
                    <label for="o_authorize_account_number" class="col-form-label">Account Number</label>
                    <input type="text" id="o_authorize_account_number" required="" class="form-control"/>
                </div>
                <div class="mb-3">
                    <label for="o_authorize_aba_number" class="col-form-label">ABA Routing Number</label>
                    <input type="text" id="o_authorize_aba_number" required="" class="form-control"/>
                </div>
                <div class="mb-3">
                    <label for="o_authorize_account_type" class="col-form-label">Bank Account Type</label>
                    <select id="o_authorize_account_type" required="" class="form-select">
                        <option value="checking">Personal Checking</option>
                        <option value="savings">Personal Savings</option>
                        <option value="businessChecking">Business Checking</option>
                    </select>
                </div>
            </t>
        </div>
    </template>

</odoo>

```

## File: views\payment_provider_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_provider_form" model="ir.ui.view">
        <field name="name">Authorize.Net Provider Form</field>
        <field name="model">payment.provider</field>
        <field name="inherit_id" ref="payment.payment_provider_form"/>
        <field name="arch" type="xml">
            <group name="provider_credentials" position='inside'>
                <group invisible="code != 'authorize'">
                    <field name="authorize_login" required="code == 'authorize' and state != 'disabled'"/>
                    <field name="authorize_transaction_key" password="True" required="code == 'authorize' and state != 'disabled'"/>
                    <field name="authorize_signature_key" password="True" required="code == 'authorize' and state != 'disabled'"/>
                    <label for="authorize_client_key"/>
                    <div class="o_row" col="2">
                        <field name="authorize_client_key"/>
                        <button class="oe_link" icon="fa-refresh" type="object"
                                name="action_update_merchant_details"
                                string="Generate Client Key"/>
                    </div>
                    <a colspan="2" href="https://www.odoo.com/documentation/17.0/applications/finance/payment_providers/authorize.html" target="_blank">
                        How to get paid with Authorize.Net
                    </a>
                </group>
            </group>
            <div name="available_currencies" position="inside">
                <button string="Set Account Currency"
                        type="object"
                        name="action_update_merchant_details"
                        invisible="code != 'authorize'"
                        icon="fa-refresh"
                        class="oe_link"/>
            </div>
        </field>
    </record>

</odoo>

```

## File: views\payment_token_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_token_form" model="ir.ui.view">
        <field name="name">Authorize.Net Token Form</field>
        <field name="model">payment.token</field>
        <field name="inherit_id" ref="payment.payment_token_form"/>
        <field name="arch" type="xml">
            <field name="provider_ref" position="after">
                <field name="provider_code" invisible="1"/>
                <field name="authorize_profile" invisible="provider_code != 'authorize'"/>
            </field>
        </field>
    </record>

</odoo>

```

