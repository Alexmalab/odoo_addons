# Odoo Module: payment_authorize

Category: Accounting/Payment Providers

This file contains the source code of the Odoo module.

## File: const.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

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


def post_init_hook(cr, registry):
    setup_provider(cr, registry, 'authorize')


def uninstall_hook(cr, registry):
    reset_payment_provider(cr, registry, 'authorize')

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
    'application': False,
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

    @http.route('/payment/authorize/get_provider_info', type='json', auth='public')
    def authorize_get_provider_info(self, provider_id):
        """ Return public information on the provider.

        :param int provider_id: The provider handling the transaction, as a `payment.provider` id
        :return: Information on the provider, namely: the state, payment method type, login ID, and
                 public client key
        :rtype: dict
        """
        provider_sudo = request.env['payment.provider'].sudo().browse(provider_id).exists()
        return {
            'state': provider_sudo.state,
            'payment_method_type': provider_sudo.authorize_payment_method_type,
            # The public API key solely used to identify the seller account with Authorize.Net
            'login_id': provider_sudo.authorize_login,
            # The public client key solely used to identify requests from the Accept.js suite
            'client_key': provider_sudo.authorize_client_key,
        }

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
        self.payment_method_type = provider.authorize_payment_method_type

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
            return {
                'x_response_code': response.get('transactionResponse', {}).get('responseCode'),
                'x_trans_id': response.get('transactionResponse', {}).get('transId'),
                'x_type': operation,
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
        if self.payment_method_type == 'credit_card':
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

import logging
import pprint

from odoo import _, api, fields, models
from odoo.fields import Command
from odoo.exceptions import UserError, ValidationError

from .authorize_request import AuthorizeAPI

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
    # Authorize.Net supports only one currency: "One gateway account is required for each currency"
    # See https://community.developer.authorize.net/t5/The-Authorize-Net-Developer-Blog/Authorize-Net-UK-Europe-Update/ba-p/35957
    authorize_currency_id = fields.Many2one(
        string="Authorize Currency", comodel_name='res.currency')
    authorize_payment_method_type = fields.Selection(
        string="Allow Payments From",
        help="Determines with what payment method the customer can pay.",
        selection=[('credit_card', "Credit Card"), ('bank_account', "Bank Account (USA Only)")],
        default='credit_card',
        required_if_provider='authorize',
    )

    # === CONSTRAINT METHODS ===#

    @api.constrains('authorize_payment_method_type')
    def _check_payment_method_type(self):
        for provider in self.filtered(lambda p: p.code == "authorize"):
            if self.env['payment.token'].search([('provider_id', '=', provider.id)], limit=1):
                raise ValidationError(_(
                    "There are active tokens linked to this provider. To change the payment method "
                    "type, please disable the provider and duplicate it. Then, change the payment "
                    "method type on the duplicated provider."
                ))

    #=== COMPUTE METHODS ===#

    def _compute_feature_support_fields(self):
        """ Override of `payment` to enable additional features. """
        super()._compute_feature_support_fields()
        self.filtered(lambda p: p.code == 'authorize').update({
            'support_manual_capture': True,
            'support_refund': 'full_only',
            'support_tokenization': True,
        })

    # === ONCHANGE METHODS ===#

    @api.onchange('authorize_payment_method_type')
    def _onchange_authorize_payment_method_type(self):
        if self.authorize_payment_method_type == 'bank_account':
            self.display_as = _("Bank (powered by Authorize)")
            self.payment_icon_ids = [Command.clear()]
        else:
            self.display_as = _("Credit Card (powered by Authorize)")
            self.payment_icon_ids = [Command.set([self.env.ref(icon_xml_id).id for icon_xml_id in (
                'payment.payment_icon_cc_maestro',
                'payment.payment_icon_cc_mastercard',
                'payment.payment_icon_cc_discover',
                'payment.payment_icon_cc_diners_club_intl',
                'payment.payment_icon_cc_jcb',
                'payment.payment_icon_cc_visa',
            )])]

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
        self.authorize_currency_id = currency
        self.authorize_client_key = res_content.get('publicClientKey')

    # === BUSINESS METHODS ===#

    @api.model
    def _get_compatible_providers(self, *args, currency_id=None, **kwargs):
        """ Override of payment to unlist Authorize providers for unsupported currencies. """
        providers = super()._get_compatible_providers(*args, currency_id=currency_id, **kwargs)

        currency = self.env['res.currency'].browse(currency_id).exists()
        if currency:
            providers = providers.filtered(
                lambda p: p.code != 'authorize' or currency == p.authorize_currency_id
            )

        return providers

    def _get_validation_amount(self):
        """ Override of payment to return the amount for Authorize.Net validation operations.

        :return: The validation amount
        :rtype: float
        """
        res = super()._get_validation_amount()
        if self.code != 'authorize':
            return res

        return 0.01

    def _get_validation_currency(self):
        """ Override of payment to return the currency for Authorize.Net validation operations.

        :return: The validation currency
        :rtype: recordset of `res.currency`
        """
        res = super()._get_validation_currency()
        if self.code != 'authorize':
            return res

        return self.authorize_currency_id

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
    authorize_payment_method_type = fields.Selection(
        string="Authorize.Net Payment Type",
        help="The type of payment method this token is linked to.",
        selection=[("credit_card", "Credit Card"), ("bank_account", "Bank Account (USA Only)")],
    )

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
from odoo.addons.payment_authorize.const import TRANSACTION_STATUS_MAPPING


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
            self._set_canceled()
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
                # The payment has not been settle on Authorize.net yet. It must be voided rather
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

    def _send_capture_request(self):
        """ Override of payment to send a capture request to Authorize.

        Note: self.ensure_one()

        :return: None
        """
        super()._send_capture_request()
        if self.provider_code != 'authorize':
            return

        authorize_API = AuthorizeAPI(self.provider_id)
        rounded_amount = round(self.amount, self.currency_id.decimal_places)
        res_content = authorize_API.capture(self.provider_reference, rounded_amount)
        _logger.info(
            "capture request response for transaction with reference %s:\n%s",
            self.reference, pprint.pformat(res_content)
        )
        self._handle_notification_data('authorize', {'response': res_content})

    def _send_void_request(self):
        """ Override of payment to send a void request to Authorize.

        Note: self.ensure_one()

        :return: None
        """
        super()._send_void_request()
        if self.provider_code != 'authorize':
            return

        authorize_API = AuthorizeAPI(self.provider_id)
        res_content = authorize_API.void(self.provider_reference)
        _logger.info(
            "void request response for transaction with reference %s:\n%s",
            self.reference, pprint.pformat(res_content)
        )
        self._handle_notification_data('authorize', {'response': res_content})

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

        self.provider_reference = response_content.get('x_trans_id')
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
                    self._set_canceled()
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
                'payment_details': cust_profile.get('payment_details'),
                'partner_id': self.partner_id.id,
                'provider_ref': cust_profile.get('payment_profile_id'),
                'authorize_profile': cust_profile.get('profile_id'),
                'authorize_payment_method_type': self.provider_id.authorize_payment_method_type,
                'verified': True,
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
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#94B6C8"/><stop offset="100%" stop-color="#6A9EBA"/></linearGradient><path id="d" d="M19.25 46h2.714v1.212c0 .188.152.343.339.343h26.394a.342.342 0 0 0 .339-.343V37.5H33.625l-2.62-6.5h18.031v-3.212a.342.342 0 0 0-.339-.343H29.765l-1.018-2.742h20.289c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742V46zm-3.79 9.642h-1.037V55h2.833v.642h-1.037v2.832h-.76v-2.832zM17.422 55h1.07l.809 2.389h.01L20.075 55h1.07v3.474h-.711v-2.462h-.01l-.847 2.462h-.586l-.848-2.438h-.01v2.438h-.711V55zm17.163-12.266c0 .228-.19.342-.57.342-.253 0-.481-.013-.684-.038l-4.826-.076-4.826.076a4.036 4.036 0 0 1-.57.038c-.507 0-.76-.127-.76-.38 0-.279.342-.418 1.026-.418 2.23-.025 3.344-.253 3.344-.684 0-.025-.025-.127-.076-.304l-.304-1.026-.19-.532c-.228-.785-.785-2.52-1.672-5.206h-8.132c-1.52 4.053-2.28 6.384-2.28 6.992 0 .355.152.57.456.646.076.025.71.076 1.9.152.735.025 1.102.152 1.102.38 0 .253-.33.38-.988.38-1.165 0-2.52-.05-4.066-.152-.38-.025-.773-.038-1.178-.038-.405 0-.798.025-1.178.076A4.157 4.157 0 0 1 9.62 43c-.355 0-.532-.114-.532-.342 0-.228.399-.355 1.197-.38.798-.025 1.374-.146 1.729-.361s.633-.627.836-1.235c3.268-9.323 5.928-16.809 7.98-22.458a5.04 5.04 0 0 1 .304-1.026l.19-.684c.076-.279.177-.418.304-.418.076 0 .14.076.19.228.33.988 1.077 3.053 2.242 6.194.912 2.457 3.18 8.626 6.802 18.506.203.557.462.912.779 1.064.317.152.969.24 1.957.266.659.025.988.152.988.38zM24.25 33.652l-2.014-5.662-1.824-5.206c-1.064 2.787-2.343 6.41-3.838 10.868h7.676z"/><path id="e" d="M19.25 44h2.714v1.212c0 .188.152.343.339.343h26.394a.342.342 0 0 0 .339-.343V35.5H33.625l-2.62-6.5h18.031v-3.212a.342.342 0 0 0-.339-.343H29.765l-1.018-2.742h20.289c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742V44zm-3.79 9.642h-1.037V53h2.833v.642h-1.037v2.832h-.76v-2.832zM17.422 53h1.07l.809 2.389h.01L20.075 53h1.07v3.474h-.711v-2.462h-.01l-.847 2.462h-.586l-.848-2.438h-.01v2.438h-.711V53zm17.163-12.266c0 .228-.19.342-.57.342-.253 0-.481-.013-.684-.038l-4.826-.076-4.826.076a4.036 4.036 0 0 1-.57.038c-.507 0-.76-.127-.76-.38 0-.279.342-.418 1.026-.418 2.23-.025 3.344-.253 3.344-.684 0-.025-.025-.127-.076-.304l-.304-1.026-.19-.532c-.228-.785-.785-2.52-1.672-5.206h-8.132c-1.52 4.053-2.28 6.384-2.28 6.992 0 .355.152.57.456.646.076.025.71.076 1.9.152.735.025 1.102.152 1.102.38 0 .253-.33.38-.988.38-1.165 0-2.52-.05-4.066-.152-.38-.025-.773-.038-1.178-.038-.405 0-.798.025-1.178.076A4.157 4.157 0 0 1 9.62 41c-.355 0-.532-.114-.532-.342 0-.228.399-.355 1.197-.38.798-.025 1.374-.146 1.729-.361s.633-.627.836-1.235c3.268-9.323 5.928-16.809 7.98-22.458a5.04 5.04 0 0 1 .304-1.026l.19-.684c.076-.279.177-.418.304-.418.076 0 .14.076.19.228.33.988 1.077 3.053 2.242 6.194.912 2.457 3.18 8.626 6.802 18.506.203.557.462.912.779 1.064.317.152.969.24 1.957.266.659.025.988.152.988.38zM24.25 31.652l-2.014-5.662-1.824-5.206c-1.064 2.787-2.343 6.41-3.838 10.868h7.676z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M4 69c-2 0-4-1-4-4V33.916l21.6-19.82 3.4 8.607h12.25L48 23l3.576 1.968-.236 22.01L39.224 69H4z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: static\src\js\payment_form.js

```javascript
/* global Accept */
odoo.define('payment_authorize.payment_form', require => {
    'use strict';

    const core = require('web.core');
    const { loadJS } = require('@web/core/assets');

    const checkoutForm = require('payment.checkout_form');
    const manageForm = require('payment.manage_form');

    const _t = core._t;

    const authorizeMixin = {

        /**
         * Return all relevant inline form inputs based on the payment method type of the provider.
         *
         * @private
         * @param {number} providerId - The id of the selected provider
         * @return {Object} - An object mapping the name of inline form inputs to their DOM element
         */
        _getInlineFormInputs: function (providerId) {
            if (this.authorizeInfo.payment_method_type === "credit_card") {
                return {
                    card: document.getElementById(`o_authorize_card_${providerId}`),
                    month: document.getElementById(`o_authorize_month_${providerId}`),
                    year: document.getElementById(`o_authorize_year_${providerId}`),
                    code: document.getElementById(`o_authorize_code_${providerId}`),
                };
            } else {
                return {
                    accountName: document.getElementById(`o_authorize_account_name_${providerId}`),
                    accountNumber: document.getElementById(
                        `o_authorize_account_number_${providerId}`
                    ),
                    abaNumber: document.getElementById(`o_authorize_aba_number_${providerId}`),
                    accountType: document.getElementById(`o_authorize_account_type_${providerId}`),
                };
            }
        },

        /**
         * Return the credit card or bank data to pass to the Accept.dispatch request.
         *
         * @private
         * @param {number} providerId - The id of the selected provider
         * @return {Object} - Data to pass to the Accept.dispatch request
         */
        _getPaymentDetails: function (providerId) {
            const inputs = this._getInlineFormInputs(providerId);
            if (this.authorizeInfo.payment_method_type === 'credit_card') {
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

        /**
         * Prepare the inline form of Authorize.Net for direct payment.
         *
         * @override method from payment.payment_form_mixin
         * @private
         * @param {string} provider - The provider of the selected payment option's provider
         * @param {number} paymentOptionId - The id of the selected payment option
         * @param {string} flow - The online payment flow of the selected payment option
         * @return {Promise}
         */
        _prepareInlineForm: function (code, paymentOptionId, flow) {
            if (code !== 'authorize') {
                return this._super(...arguments);
            }

            if (flow === 'token') {
                return Promise.resolve(); // Don't show the form for tokens
            }

            this._setPaymentFlow('direct');

            let acceptJSUrl = 'https://js.authorize.net/v1/Accept.js';
            return this._rpc({
                route: '/payment/authorize/get_provider_info',
                params: {
                    'provider_id': paymentOptionId,
                },
            }).then(providerInfo => {
                if (providerInfo.state !== 'enabled') {
                    acceptJSUrl = 'https://jstest.authorize.net/v1/Accept.js';
                }
                this.authorizeInfo = providerInfo;
            }).then(() => {
                loadJS(acceptJSUrl);
            }).guardedCatch((error) => {
                error.event.preventDefault();
                this._displayError(
                    _t("Server Error"),
                    _t("An error occurred when displayed this payment form."),
                    error.message.data.message
                );
            });
        },

        /**
         * Dispatch the secure data to Authorize.Net.
         *
         * @override method from payment.payment_form_mixin
         * @private
         * @param {string} code - The code of the payment option's provider
         * @param {number} paymentOptionId - The id of the payment option handling the transaction
         * @param {string} flow - The online payment flow of the transaction
         * @return {Promise}
         */
        _processPayment: function (code, paymentOptionId, flow) {
            if (code !== 'authorize' || flow === 'token') {
                return this._super(...arguments); // Tokens are handled by the generic flow
            }

            if (!this._validateFormInputs(paymentOptionId)) {
                this._enableButton(); // The submit button is disabled at this point, enable it
                $('body').unblock(); // The page is blocked at this point, unblock it
                return Promise.resolve();
            }

            // Build the authentication and card data objects to be dispatched to Authorized.Net
            const secureData = {
                authData: {
                    apiLoginID: this.authorizeInfo.login_id,
                    clientKey: this.authorizeInfo.client_key,
                },
                ...this._getPaymentDetails(paymentOptionId),
            };

            // Dispatch secure data to Authorize.Net to get a payment nonce in return
            return Accept.dispatchData(
                secureData, response => this._responseHandler(paymentOptionId, response)
            );
        },

        /**
         * Handle the response from Authorize.Net and initiate the payment.
         *
         * @private
         * @param {number} providerId - The id of the selected provider
         * @param {object} response - The payment nonce returned by Authorized.Net
         * @return {Promise}
         */
        _responseHandler: function (providerId, response) {
            if (response.messages.resultCode === 'Error') {
                let error = "";
                response.messages.message.forEach(msg => error += `${msg.code}: ${msg.text}\n`);
                this._displayError(
                    _t("Server Error"),
                    _t("We are not able to process your payment."),
                    error
                );
                return Promise.resolve();
            }

            // Create the transaction and retrieve the processing values
            return this._rpc({
                route: this.txContext.transactionRoute,
                params: this._prepareTransactionRouteParams('authorize', providerId, 'direct'),
            }).then(processingValues => {
                // Initiate the payment
                return this._rpc({
                    route: '/payment/authorize/payment',
                    params: {
                        'reference': processingValues.reference,
                        'partner_id': processingValues.partner_id,
                        'opaque_data': response.opaqueData,
                        'access_token': processingValues.access_token,
                    }
                }).then(() => window.location = '/payment/status');
            }).guardedCatch((error) => {
                error.event.preventDefault();
                this._displayError(
                    _t("Server Error"),
                    _t("We are not able to process your payment."),
                    error.message.data.message
                );
            });
        },

        /**
         * Checks that all payment inputs adhere to the DOM validation constraints.
         *
         * @private
         * @param {number} providerId - The id of the selected provider
         * @return {boolean} - Whether all elements pass the validation constraints
         */
        _validateFormInputs: function (providerId) {
            const inputs = Object.values(this._getInlineFormInputs(providerId));
            return inputs.every(element => element.reportValidity());
        },

    };

    checkoutForm.include(authorizeMixin);
    manageForm.include(authorizeMixin);
});

```

## File: views\payment_authorize_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="inline_form">
        <div t-if="provider.authorize_payment_method_type == 'credit_card'" t-attf-id="o_authorize_form_{{provider_id}}" class="o_authorize_form">
            <div class="mb-3">
                <label t-attf-for="o_authorize_card_{{provider_id}}" class="col-form-label">Card Number</label>
                <input type="text" t-attf-id="o_authorize_card_{{provider_id}}" required="" maxlength="19" class="form-control"/>
            </div>
            <div class="row">
                <div class="col-sm-8 mb-3">
                    <label t-attf-for="o_authorize_month_{{provider_id}}">Expiration</label>
                    <div class="input-group">
                        <input type="number" t-attf-id="o_authorize_month_{{provider_id}}" placeholder="MM" min="1" max="12" required="" class="form-control"/>
                        <input type="number" t-attf-id="o_authorize_year_{{provider_id}}" placeholder="YY" min="00" max="99" required="" class="form-control"/>
                    </div>
                </div>
                <div class="col-sm-4 mb-3">
                    <label t-attf-for="o_authorize_code_{{provider_id}}">Card Code</label>
                    <input type="number" t-attf-id="o_authorize_code_{{provider_id}}" max="9999" class="form-control"/>
                </div>
            </div>
        </div>
        <div t-else="" t-attf-id="o_authorize_form_{{provider_id}}" class="o_authorize_form">
            <div class="mb-3">
                <label t-attf-for="o_authorize_bank_name_{{provider_id}}" class="col-form-label">Bank Name</label>
                <input type="text" t-attf-id="o_authorize_bank_name_{{provider_id}}" required="" class="form-control"/>
            </div>
            <div class="mb-3">
                <label t-attf-for="o_authorize_account_name_{{provider_id}}" class="col-form-label">Name On Account</label>
                <input type="text" t-attf-id="o_authorize_account_name_{{provider_id}}" required="" class="form-control"/>
            </div>
            <div class="mb-3">
                <label t-attf-for="o_authorize_account_number_{{provider_id}}" class="col-form-label">Account Number</label>
                <input type="text" t-attf-id="o_authorize_account_number_{{provider_id}}" required="" class="form-control"/>
            </div>
            <div class="mb-3">
                <label t-attf-for="o_authorize_aba_number_{{provider_id}}" class="col-form-label">ABA Routing Number</label>
                <input type="text" t-attf-id="o_authorize_aba_number_{{provider_id}}" required="" class="form-control"/>
            </div>
            <div class="mb-3">
                <label t-attf-for="o_authorize_account_type_{{provider_id}}" class="col-form-label">Bank Account Type</label>
                <select t-attf-id="o_authorize_account_type_{{provider_id}}" required="" class="form-select">
                    <option value="checking">Personal Checking</option>
                    <option value="savings">Personal Savings</option>
                    <option value="businessChecking">Business Checking</option>
                </select>
            </div>
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
                <group attrs="{'invisible': [('code', '!=', 'authorize')]}">
                    <field name="authorize_login" attrs="{'required':[('code', '=', 'authorize'), ('state', '!=', 'disabled')]}"/>
                    <field name="authorize_transaction_key" password="True" attrs="{'required':[ ('code', '=', 'authorize'), ('state', '!=', 'disabled')]}"/>
                    <field name="authorize_signature_key" password="True" attrs="{'required':[ ('code', '=', 'authorize'), ('state', '!=', 'disabled')]}"/>
                    <label for="authorize_client_key"/>
                    <div class="o_row" col="2">
                        <field name="authorize_client_key"/>
                        <button class="oe_link" icon="fa-refresh" type="object"
                                name="action_update_merchant_details"
                                string="Generate Client Key"/>
                    </div>
                    <a colspan="2" href="https://www.odoo.com/documentation/16.0/applications/finance/payment_providers/authorize.html" target="_blank">
                        How to get paid with Authorize.Net
                    </a>
                </group>
            </group>
            <field name="display_as" position="before">
                <field name="authorize_payment_method_type"
                       attrs="{'invisible': [('code', '!=', 'authorize')], 'required':[('code', '=', 'authorize'), ('state', '!=', 'disabled')]}"/>
            </field>
            <field name="available_country_ids" position="after">
                <label for="authorize_currency_id" string="Currency" attrs="{'invisible': [('code', '!=', 'authorize')]}"/>
                <div  class="o_row" col="2" attrs="{'invisible': [('code', '!=', 'authorize')]}">
                    <field name="authorize_currency_id"/>
                    <button class="oe_link" icon="fa-refresh" type="object"
                            name="action_update_merchant_details"
                            string="Set Account Currency"/>
                </div>
            </field>
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
                <field name="authorize_profile" attrs="{'invisible':[('provider_code', '!=', 'authorize')]}"/>
                <field name="authorize_payment_method_type" attrs="{'invisible': [('provider_code', '!=', 'authorize')]}"/>
            </field>
        </field>
    </record>

</odoo>

```

