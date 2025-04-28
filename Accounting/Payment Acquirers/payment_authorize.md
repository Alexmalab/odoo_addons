# Odoo Module: payment_authorize

Category: Accounting/Payment Acquirers

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import controllers
from odoo.addons.payment.models.payment_acquirer import create_missing_journal_for_acquirers
from odoo.addons.payment import reset_payment_provider

def uninstall_hook(cr, registry):
    reset_payment_provider(cr, registry, 'authorize')


```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-

{
    'name': 'Authorize.Net Payment Acquirer',
    'category': 'Accounting/Payment Acquirers',
    'sequence': 350,
    'summary': 'Payment Acquirer: Authorize.net Implementation',
    'version': '1.0',
    'description': """Authorize.Net Payment Acquirer""",
    'depends': ['payment'],
    'data': [
        'views/payment_views.xml',
        'views/payment_authorize_templates.xml',
        'data/payment_acquirer_data.xml',
    ],
    'installable': True,
    'application': True,
    'post_init_hook': 'create_missing_journal_for_acquirers',
    'uninstall_hook': 'uninstall_hook',
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
import pprint
import logging
from werkzeug import urls, utils

from odoo import http, _
from odoo.http import request
from odoo.exceptions import ValidationError, UserError

_logger = logging.getLogger(__name__)


class AuthorizeController(http.Controller):
    _return_url = '/payment/authorize/return/'
    _cancel_url = '/payment/authorize/cancel/'

    @http.route([
        '/payment/authorize/return/',
        '/payment/authorize/cancel/',
    ], type='http', auth='public', csrf=False, save_session=False)
    def authorize_form_feedback(self, **post):
        """ Process the data returned by Authorize after redirection.

        The route is flagged with `save_session=False` to prevent Odoo from assigning a new session
        to the user if they are redirected to this route with a POST request. Indeed, as the session
        cookie is created without a `SameSite` attribute, some browsers that don't implement the
        recommended default `SameSite=Lax` behavior will not include the cookie in the redirection
        request from the payment provider to Odoo. As the redirection to the '/payment/status' page
        will satisfy any specification of the `SameSite` attribute, the session of the user will be
        retrieved and with it the transaction which will be immediately post-processed.
        """
        _logger.info('Authorize: entering form_feedback with post data %s', pprint.pformat(post))
        if post:
            request.env['payment.transaction'].sudo().form_feedback(post, 'authorize')
        base_url = request.env['ir.config_parameter'].sudo().get_param('web.base.url')
        # Authorize.Net is expecting a response to the POST sent by their server.
        # This response is in the form of a URL that Authorize.Net will pass on to the
        # client's browser to redirect them to the desired location need javascript.
        return request.render('payment_authorize.payment_authorize_redirect', {
            'return_url': urls.url_join(base_url, "/payment/process")
        })

    @http.route(['/payment/authorize/s2s/create_json_3ds'], type='json', auth='public', csrf=False)
    def authorize_s2s_create_json_3ds(self, verify_validity=False, **kwargs):
        token = False
        acquirer = request.env['payment.acquirer'].browse(int(kwargs.get('acquirer_id')))

        try:
            if not kwargs.get('partner_id'):
                kwargs = dict(kwargs, partner_id=request.env.user.partner_id.id)
            token = acquirer.s2s_process(kwargs)
        except ValidationError as e:
            message = e.args[0]
            if isinstance(message, dict) and 'missing_fields' in message:
                if request.env.user._is_public():
                    message = _("Please sign in to complete the payment.")
                    # update message if portal mode = b2b
                    if request.env['ir.config_parameter'].sudo().get_param('auth_signup.allow_uninvited', 'False').lower() == 'false':
                        message += _(" If you don't have any account, ask your salesperson to grant you a portal access. ")
                else:
                    msg = _("The transaction cannot be processed because some contact details are missing or invalid: ")
                    message = msg + ', '.join(message['missing_fields']) + '. '
                    message += _("Please complete your profile. ")

            return {
                'error': message
            }

        if not token:
            res = {
                'result': False,
            }
            return res

        res = {
            'result': True,
            'id': token.id,
            'short_name': token.short_name,
            '3d_secure': False,
            'verified': True, #Authorize.net does a transaction type of Authorization Only
                              #As Authorize.net already verify this card, we do not verify this card again.
        }
        #token.validate() don't work with Authorize.net.
        #Payments made via Authorize.net are settled and allowed to be refunded only on the next day.
        #https://account.authorize.net/help/Miscellaneous/FAQ/Frequently_Asked_Questions.htm#Refund
        #<quote>The original transaction that you wish to refund must have a status of Settled Successfully.
        #You cannot issue refunds against unsettled, voided, declined or errored transactions.</quote>
        return res

    @http.route(['/payment/authorize/s2s/create'], type='http', auth='public', save_session=False)
    def authorize_s2s_create(self, **post):
        acquirer_id = int(post.get('acquirer_id'))
        acquirer = request.env['payment.acquirer'].browse(acquirer_id)
        acquirer.s2s_process(post)
        return utils.redirect("/payment/process")

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-

from . import main

```

## File: data\payment_acquirer_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <record id="payment.payment_acquirer_authorize" model="payment.acquirer">
            <field name="name">Authorize.Net</field>
            <field name="image_128" type="base64" file="payment_authorize/static/src/img/authorize_icon.png"/>
            <field name="provider">authorize</field>
            <field name="company_id" ref="base.main_company"/>
            <field name="view_template_id" ref="authorize_form"/>
            <field name="registration_view_template_id" ref="authorize_s2s_form"/>
        </record>

    </data>
</odoo>

```

## File: models\authorize_request.py

```python
# -*- coding: utf-8 -*-
import json
import logging
import requests

from uuid import uuid4

from odoo import _
from odoo.exceptions import UserError

from odoo.addons.payment.models.payment_acquirer import _partner_split_name

_logger = logging.getLogger(__name__)


class AuthorizeAPI():
    """Authorize.net Gateway API integration.

    This class allows contacting the Authorize.net API with simple operation
    requests. It implements a *very limited* subset of the complete API
    (http://developer.authorize.net/api/reference); namely:
        - Customer Profile/Payment Profile creation
        - Transaction authorization/capture/voiding
    """

    AUTH_ERROR_STATUS = 3

    def __init__(self, acquirer):
        """Initiate the environment with the acquirer data.

        :param record acquirer: payment.acquirer account that will be contacted
        """
        if acquirer.state == 'enabled':
            self.url = 'https://api.authorize.net/xml/v1/request.api'
        else:
            self.url = 'https://apitest.authorize.net/xml/v1/request.api'

        self.state = acquirer.state
        self.name = acquirer.authorize_login
        self.transaction_key = acquirer.authorize_transaction_key

    def _authorize_request(self, data):
        _logger.info('_authorize_request: Sending values to URL %s, values:\n%s', self.url, data)
        resp = requests.post(self.url, json.dumps(data))
        resp.raise_for_status()
        resp = json.loads(resp.content)
        _logger.info("_authorize_request: Received response:\n%s", resp)
        messages = resp.get('messages')
        if messages and messages.get('resultCode') == 'Error':
            return {
                'err_code': messages.get('message')[0].get('code'),
                'err_msg': messages.get('message')[0].get('text')
            }

        return resp

    # Customer profiles
    def create_customer_profile(self, partner, opaqueData):
        """Create a payment and customer profile in the Authorize.net backend.

        Creates a customer profile for the partner/credit card combination and links
        a corresponding payment profile to it. Note that a single partner in the Odoo
        database can have multiple customer profiles in Authorize.net (i.e. a customer
        profile is created for every res.partner/payment.token couple).

        :param record partner: the res.partner record of the customer
        :param str cardnumber: cardnumber in string format (numbers only, no separator)
        :param str expiration_date: expiration date in 'YYYY-MM' string format
        :param str card_code: three- or four-digit verification number

        :return: a dict containing the profile_id and payment_profile_id of the
                 newly created customer profile and payment profile
        :rtype: dict
        """
        values = {
            'createCustomerProfileRequest': {
                'merchantAuthentication': {
                    'name': self.name,
                    'transactionKey': self.transaction_key
                },
                'profile': {
                    'description': ('ODOO-%s-%s' % (partner.id, uuid4().hex[:8]))[:20],
                    'email': partner.email or '',
                    'paymentProfiles': {
                        'customerType': 'business' if partner.is_company else 'individual',
                        'billTo': {
                            'firstName': '' if partner.is_company else _partner_split_name(partner.name)[0][:50],
                            'lastName':  partner.name[:50] if partner.is_company else _partner_split_name(partner.name)[1][:50],
                            'address': (partner.street or '' + (partner.street2 if partner.street2 else '')) or None,
                            'city': partner.city,
                            'state': partner.state_id.name or None,
                            'zip': partner.zip or '',
                            'country': partner.country_id.name or None,
                            'phoneNumber': partner.phone or '',
                        },
                        'payment': {
                            'opaqueData': {
                                'dataDescriptor': opaqueData.get('dataDescriptor'),
                                'dataValue': opaqueData.get('dataValue')
                            }
                        }
                    }
                },
                'validationMode': 'liveMode' if self.state == 'enabled' else 'testMode'
            }
        }

        response = self._authorize_request(values)

        if response and response.get('err_code'):
            raise UserError(_(
                "Authorize.net Error:\nCode: %s\nMessage: %s",
                response.get('err_code'), response.get('err_msg'),
            ))

        return {
            'profile_id': response.get('customerProfileId'),
            'payment_profile_id': response.get('customerPaymentProfileIdList')[0]
        }

    def create_customer_profile_from_tx(self, partner, transaction_id):
        """Create an Auth.net payment/customer profile from an existing transaction.

        Creates a customer profile for the partner/credit card combination and links
        a corresponding payment profile to it. Note that a single partner in the Odoo
        database can have multiple customer profiles in Authorize.net (i.e. a customer
        profile is created for every res.partner/payment.token couple).

        Note that this function makes 2 calls to the authorize api, since we need to
        obtain a partial cardnumber to generate a meaningful payment.token name.

        :param record partner: the res.partner record of the customer
        :param str transaction_id: id of the authorized transaction in the
                                   Authorize.net backend

        :return: a dict containing the profile_id and payment_profile_id of the
                 newly created customer profile and payment profile as well as the
                 last digits of the card number
        :rtype: dict
        """
        values = {
            'createCustomerProfileFromTransactionRequest': {
                "merchantAuthentication": {
                    "name": self.name,
                    "transactionKey": self.transaction_key
                },
                'transId': transaction_id,
                'customer': {
                    'merchantCustomerId': ('ODOO-%s-%s' % (partner.id, uuid4().hex[:8]))[:20],
                    'email': partner.email or ''
                }
            }
        }

        response = self._authorize_request(values)

        if not response.get('customerProfileId'):
            _logger.warning(
                'Unable to create customer payment profile, data missing from transaction. Transaction_id: %s - Partner_id: %s'
                % (transaction_id, partner)
            )
            return False

        res = {
            'profile_id': response.get('customerProfileId'),
            'payment_profile_id': response.get('customerPaymentProfileIdList')[0]
        }

        values = {
            'getCustomerPaymentProfileRequest': {
                "merchantAuthentication": {
                    "name": self.name,
                    "transactionKey": self.transaction_key
                },
                'customerProfileId': res['profile_id'],
                'customerPaymentProfileId': res['payment_profile_id'],
            }
        }

        response = self._authorize_request(values)

        res['name'] = response.get('paymentProfile', {}).get('payment', {}).get('creditCard', {}).get('cardNumber')
        return res

    # Transaction management
    def auth_and_capture(self, token, amount, reference):
        """Authorize and capture a payment for the given amount.

        Authorize and immediately capture a payment for the given payment.token
        record for the specified amount with reference as communication.

        :param record token: the payment.token record that must be charged
        :param str amount: transaction amount (up to 15 digits with decimal point)
        :param str reference: used as "invoiceNumber" in the Authorize.net backend

        :return: a dict containing the response code, transaction id and transaction type
        :rtype: dict
        """
        values = {
            'createTransactionRequest': {
                "merchantAuthentication": {
                    "name": self.name,
                    "transactionKey": self.transaction_key
                },
                'transactionRequest': {
                    'transactionType': 'authCaptureTransaction',
                    'amount': str(amount),
                    'profile': {
                        'customerProfileId': token.authorize_profile,
                        'paymentProfile': {
                            'paymentProfileId': token.acquirer_ref,
                        }
                    },
                    'order': {
                        'invoiceNumber': reference[:20],
                        'description': reference[:255],
                    }
                }

            }
        }
        response = self._authorize_request(values)

        if response and response.get('err_code'):
            return {
                'x_response_code': self.AUTH_ERROR_STATUS,
                'x_response_reason_text': response.get('err_msg')
            }

        result = {
            'x_response_code': response.get('transactionResponse', {}).get('responseCode'),
            'x_trans_id': response.get('transactionResponse', {}).get('transId'),
            'x_type': 'auth_capture'
        }
        errors = response.get('transactionResponse', {}).get('errors')
        if errors:
            result['x_response_reason_text'] = '\n'.join([e.get('errorText') for e in errors])
        return result

    def authorize(self, token, amount, reference):
        """Authorize a payment for the given amount.

        Authorize (without capture) a payment for the given payment.token
        record for the specified amount with reference as communication.

        :param record token: the payment.token record that must be charged
        :param str amount: transaction amount (up to 15 digits with decimal point)
        :param str reference: used as "invoiceNumber" in the Authorize.net backend

        :return: a dict containing the response code, transaction id and transaction type
        :rtype: dict
        """
        values = {
            'createTransactionRequest': {
                "merchantAuthentication": {
                    "name": self.name,
                    "transactionKey": self.transaction_key
                },
                'transactionRequest': {
                    'transactionType': 'authOnlyTransaction',
                    'amount': str(amount),
                    'profile': {
                        'customerProfileId': token.authorize_profile,
                        'paymentProfile': {
                            'paymentProfileId': token.acquirer_ref,
                        }
                    },
                    'order': {
                        'invoiceNumber': reference[:20],
                        'description': reference[:255],
                    }
                }

            }
        }
        response = self._authorize_request(values)

        if response and response.get('err_code'):
            return {
                'x_response_code': self.AUTH_ERROR_STATUS,
                'x_response_reason_text': response.get('err_msg')
            }

        return {
            'x_response_code': response.get('transactionResponse', {}).get('responseCode'),
            'x_trans_id': response.get('transactionResponse', {}).get('transId'),
            'x_type': 'auth_only'
        }

    def capture(self, transaction_id, amount):
        """Capture a previously authorized payment for the given amount.

        Capture a previsouly authorized payment. Note that the amount is required
        even though we do not support partial capture.

        :param str transaction_id: id of the authorized transaction in the
                                   Authorize.net backend
        :param str amount: transaction amount (up to 15 digits with decimal point)

        :return: a dict containing the response code, transaction id and transaction type
        :rtype: dict
        """
        values = {
            'createTransactionRequest': {
                "merchantAuthentication": {
                    "name": self.name,
                    "transactionKey": self.transaction_key
                },
                'transactionRequest': {
                    'transactionType': 'priorAuthCaptureTransaction',
                    'amount': str(amount),
                    'refTransId': transaction_id,
                }
            }
        }

        response = self._authorize_request(values)

        if response and response.get('err_code'):
            return {
                'x_response_code': self.AUTH_ERROR_STATUS,
                'x_response_reason_text': response.get('err_msg')
            }

        return {
            'x_response_code': response.get('transactionResponse', {}).get('responseCode'),
            'x_trans_id': response.get('transactionResponse', {}).get('transId'),
            'x_type': 'prior_auth_capture'
        }

    def void(self, transaction_id):
        """Void a previously authorized payment.

        :param str transaction_id: the id of the authorized transaction in the
                                   Authorize.net backend

        :return: a dict containing the response code, transaction id and transaction type
        :rtype: dict
        """
        values = {
            'createTransactionRequest': {
                "merchantAuthentication": {
                    "name": self.name,
                    "transactionKey": self.transaction_key
                },
                'transactionRequest': {
                    'transactionType': 'voidTransaction',
                    'refTransId': transaction_id
                }
            }
        }

        response = self._authorize_request(values)

        if response and response.get('err_code'):
            return {
                'x_response_code': self.AUTH_ERROR_STATUS,
                'x_response_reason_text': response.get('err_msg')
            }

        return {
            'x_response_code': response.get('transactionResponse', {}).get('responseCode'),
            'x_trans_id': response.get('transactionResponse', {}).get('transId'),
            'x_type': 'void'
        }

    # Test
    def test_authenticate(self):
        """Test Authorize.net communication with a simple credentials check.

        :return: True if authentication was successful, else False (or throws an error)
        :rtype: bool
        """
        values = {
            'authenticateTestRequest': {
                "merchantAuthentication": {
                    "name": self.name,
                    "transactionKey": self.transaction_key
                },
            }
        }

        response = self._authorize_request(values)
        if response and response.get('err_code'):
            return False
        return True

    # Client Key
    def get_client_secret(self):
        """ Create a client secret that will be needed for the AcceptJS integration. """
        values = {
            "getMerchantDetailsRequest": {
                "merchantAuthentication": {
                    "name": self.name,
                    "transactionKey": self.transaction_key,
                }
            }
        }
        response = self._authorize_request(values)
        client_secret = response.get('publicClientKey')
        return client_secret

```

## File: models\payment.py

```python
# coding: utf-8
from werkzeug import urls

from .authorize_request import AuthorizeAPI
import hashlib
import hmac
import logging
import time

from odoo import _, api, fields, models
from odoo.addons.payment.models.payment_acquirer import ValidationError
from odoo.addons.payment_authorize.controllers.main import AuthorizeController
from odoo.tools.float_utils import float_compare, float_repr
from odoo.exceptions import UserError

_logger = logging.getLogger(__name__)


class PaymentAcquirerAuthorize(models.Model):
    _inherit = 'payment.acquirer'

    provider = fields.Selection(selection_add=[
        ('authorize', 'Authorize.Net')
    ], ondelete={'authorize': 'set default'})
    authorize_login = fields.Char(string='API Login Id', required_if_provider='authorize', groups='base.group_user')
    authorize_transaction_key = fields.Char(string='API Transaction Key', required_if_provider='authorize', groups='base.group_user')
    authorize_signature_key = fields.Char(string='API Signature Key', required_if_provider='authorize', groups='base.group_user')
    authorize_client_key = fields.Char(string='API Client Key', groups='base.group_user')

    @api.onchange('provider', 'check_validity')
    def onchange_check_validity(self):
        if self.provider == 'authorize' and self.check_validity:
            self.check_validity = False
            return {'warning': {
                'title': _("Warning"),
                'message': ('This option is not supported for Authorize.net')}}

    def action_client_secret(self):
        api = AuthorizeAPI(self)
        if not api.test_authenticate():
            raise UserError(_('Unable to fetch Client Key, make sure the API Login and Transaction Key are correct.'))
        self.authorize_client_key = api.get_client_secret()
        return True

    def _get_feature_support(self):
        """Get advanced feature support by provider.

        Each provider should add its technical in the corresponding
        key for the following features:
            * fees: support payment fees computations
            * authorize: support authorizing payment (separates
                         authorization and capture)
            * tokenize: support saving payment data in a payment.tokenize
                        object
        """
        res = super(PaymentAcquirerAuthorize, self)._get_feature_support()
        res['authorize'].append('authorize')
        res['tokenize'].append('authorize')
        return res

    def _get_authorize_urls(self, environment):
        """ Authorize URLs """
        if environment == 'prod':
            return {'authorize_form_url': 'https://secure2.authorize.net/gateway/transact.dll'}
        else:
            return {'authorize_form_url': 'https://test.authorize.net/gateway/transact.dll'}

    def _authorize_generate_hashing(self, values):
        data = '^'.join([
            values['x_login'],
            values['x_fp_sequence'],
            values['x_fp_timestamp'],
            values['x_amount'],
            values['x_currency_code']]).encode('utf-8')

        return hmac.new(bytes.fromhex(self.authorize_signature_key), data, hashlib.sha512).hexdigest().upper()

    def authorize_form_generate_values(self, values):
        self.ensure_one()
        # State code is only supported in US, use state name by default
        # See https://developer.authorize.net/api/reference/
        state = values['partner_state'].name if values.get('partner_state') else ''
        if values.get('partner_country') and values.get('partner_country') == self.env.ref('base.us', False):
            state = values['partner_state'].code if values.get('partner_state') else ''
        billing_state = values['billing_partner_state'].name if values.get('billing_partner_state') else ''
        if values.get('billing_partner_country') and values.get('billing_partner_country') == self.env.ref('base.us', False):
            billing_state = values['billing_partner_state'].code if values.get('billing_partner_state') else ''

        base_url = self.get_base_url()
        authorize_tx_values = dict(values)
        temp_authorize_tx_values = {
            'x_login': self.authorize_login,
            'x_amount': float_repr(values['amount'], values['currency'].decimal_places if values['currency'] else 2),
            'x_show_form': 'PAYMENT_FORM',
            'x_type': 'AUTH_CAPTURE' if not self.capture_manually else 'AUTH_ONLY',
            'x_method': 'CC',
            'x_fp_sequence': '%s%s' % (self.id, int(time.time())),
            'x_version': '3.1',
            'x_relay_response': 'TRUE',
            'x_fp_timestamp': str(int(time.time())),
            'x_relay_url': urls.url_join(base_url, AuthorizeController._return_url),
            'x_cancel_url': urls.url_join(base_url, AuthorizeController._cancel_url),
            'x_currency_code': values['currency'] and values['currency'].name or '',
            'address': values.get('partner_address'),
            'city': values.get('partner_city'),
            'country': values.get('partner_country') and values.get('partner_country').name or '',
            'email': values.get('partner_email'),
            'zip_code': values.get('partner_zip'),
            'first_name': values.get('partner_first_name'),
            'last_name': values.get('partner_last_name'),
            'phone': values.get('partner_phone'),
            'state': state,
            'billing_address': values.get('billing_partner_address'),
            'billing_city': values.get('billing_partner_city'),
            'billing_country': values.get('billing_partner_country') and values.get('billing_partner_country').name or '',
            'billing_email': values.get('billing_partner_email'),
            'billing_zip_code': values.get('billing_partner_zip'),
            'billing_first_name': values.get('billing_partner_first_name'),
            'billing_last_name': values.get('billing_partner_last_name'),
            'billing_phone': values.get('billing_partner_phone'),
            'billing_state': billing_state,
        }
        temp_authorize_tx_values['returndata'] = authorize_tx_values.pop('return_url', '')
        temp_authorize_tx_values['x_fp_hash'] = self._authorize_generate_hashing(temp_authorize_tx_values)
        authorize_tx_values.update(temp_authorize_tx_values)
        return authorize_tx_values

    def authorize_get_form_action_url(self):
        self.ensure_one()
        environment = 'prod' if self.state == 'enabled' else 'test'
        return self._get_authorize_urls(environment)['authorize_form_url']

    @api.model
    def authorize_s2s_form_process(self, data):
        values = {
            'opaqueData': data.get('opaqueData'),
            'encryptedCardData': data.get('encryptedCardData'),
            'acquirer_id': int(data.get('acquirer_id')),
            'partner_id': int(data.get('partner_id'))
        }
        PaymentMethod = self.env['payment.token'].sudo().create(values)
        return PaymentMethod

    def authorize_s2s_form_validate(self, data):
        error = dict()
        mandatory_fields = ["opaqueData", "encryptedCardData"]
        # Validation
        for field_name in mandatory_fields:
            if not data.get(field_name):
                error[field_name] = 'missing'
        return False if error else True

    def authorize_test_credentials(self):
        self.ensure_one()
        transaction = AuthorizeAPI(self.acquirer_id)
        return transaction.test_authenticate()

class TxAuthorize(models.Model):
    _inherit = 'payment.transaction'

    _authorize_valid_tx_status = 1
    _authorize_pending_tx_status = 4
    _authorize_cancel_tx_status = 2
    _authorize_error_tx_status = 3

    # --------------------------------------------------
    # FORM RELATED METHODS
    # --------------------------------------------------

    @api.model
    def _authorize_form_get_tx_from_data(self, data):
        """ Given a data dict coming from authorize, verify it and find the related
        transaction record. """
        reference, description, trans_id, fingerprint = data.get('x_invoice_num'), data.get('x_description'), data.get('x_trans_id'), data.get('x_SHA2_Hash') or data.get('x_MD5_Hash')
        if not reference or not trans_id or not fingerprint:
            error_msg = _('Authorize: received data with missing reference (%s) or trans_id (%s) or fingerprint (%s)') % (reference, trans_id, fingerprint)
            _logger.info(error_msg)
            raise ValidationError(error_msg)
        tx = self.search(['|', ('reference', '=', reference), ('reference', '=', description)])
        if not tx or len(tx) > 1:
            error_msg = 'Authorize: received data for x_invoice_num %s and x_description %s' % (reference, description)
            if not tx:
                error_msg += '; no order found'
            else:
                error_msg += '; multiple order found'
            _logger.info(error_msg)
            raise ValidationError(error_msg)
        return tx[0]

    def _authorize_form_get_invalid_parameters(self, data):
        invalid_parameters = []

        if self.acquirer_reference and data.get('x_trans_id') != self.acquirer_reference:
            invalid_parameters.append(('Transaction Id', data.get('x_trans_id'), self.acquirer_reference))
        # check what is buyed
        if float_compare(float(data.get('x_amount', '0.0')), self.amount, 2) != 0:
            invalid_parameters.append(('Amount', data.get('x_amount'), '%.2f' % self.amount))
        return invalid_parameters

    def _authorize_form_validate(self, data):
        if self.state == 'done':
            _logger.warning('Authorize: trying to validate an already validated tx (ref %s)' % self.reference)
            return True
        status_code = int(data.get('x_response_code', '0'))
        if status_code == self._authorize_valid_tx_status:
            if data.get('x_type').lower() in ['auth_capture', 'prior_auth_capture']:
                self.write({
                    'acquirer_reference': data.get('x_trans_id'),
                    'date': fields.Datetime.now(),
                })
                self._set_transaction_done()
            elif data.get('x_type').lower() in ['auth_only']:
                self.write({'acquirer_reference': data.get('x_trans_id')})
                self._set_transaction_authorized()
            if self.partner_id and not self.payment_token_id and \
               (self.type == 'form_save' or self.acquirer_id.save_token == 'always'):
                transaction = AuthorizeAPI(self.acquirer_id)
                res = transaction.create_customer_profile_from_tx(self.partner_id, self.acquirer_reference)
                if res:
                    token_id = self.env['payment.token'].create({
                        'authorize_profile': res.get('profile_id'),
                        'name': res.get('name'),
                        'acquirer_ref': res.get('payment_profile_id'),
                        'acquirer_id': self.acquirer_id.id,
                        'partner_id': self.partner_id.id,
                    })
                    self.payment_token_id = token_id
            return True
        elif status_code == self._authorize_pending_tx_status:
            self.write({'acquirer_reference': data.get('x_trans_id')})
            self._set_transaction_pending()
            return True
        else:
            error = data.get('x_response_reason_text')
            _logger.info(error)
            self.write({
                'state_message': error,
                'acquirer_reference': data.get('x_trans_id'),
            })
            self._set_transaction_cancel()
            return False

    def authorize_s2s_do_transaction(self, **data):
        self.ensure_one()
        transaction = AuthorizeAPI(self.acquirer_id)

        if not self.payment_token_id.authorize_profile:
            raise UserError(_('Invalid token found: the Authorize profile is missing.'
                              'Please make sure the token has a valid acquirer reference.'))

        if not self.acquirer_id.capture_manually:
            res = transaction.auth_and_capture(self.payment_token_id, round(self.amount, self.currency_id.decimal_places), self.reference)
        else:
            res = transaction.authorize(self.payment_token_id, round(self.amount, self.currency_id.decimal_places), self.reference)
        return self._authorize_s2s_validate_tree(res)

    def authorize_s2s_capture_transaction(self):
        self.ensure_one()
        transaction = AuthorizeAPI(self.acquirer_id)
        tree = transaction.capture(self.acquirer_reference or '', round(self.amount, self.currency_id.decimal_places))
        return self._authorize_s2s_validate_tree(tree)

    def authorize_s2s_void_transaction(self):
        self.ensure_one()
        transaction = AuthorizeAPI(self.acquirer_id)
        tree = transaction.void(self.acquirer_reference or '')
        return self._authorize_s2s_validate_tree(tree)

    def _authorize_s2s_validate_tree(self, tree):
        return self._authorize_s2s_validate(tree)

    def _authorize_s2s_validate(self, tree):
        if self.state == 'done':
            _logger.warning('Authorize: trying to validate an already validated tx (ref %s)' % self.reference)
            return True
        status_code = int(tree.get('x_response_code', '0'))
        if status_code == self._authorize_valid_tx_status:
            if tree.get('x_type').lower() in ['auth_capture', 'prior_auth_capture']:
                init_state = self.state
                self.write({
                    'acquirer_reference': tree.get('x_trans_id'),
                    'date': fields.Datetime.now(),
                })

                self._set_transaction_done()

                if init_state != 'authorized':
                    self.execute_callback()
            if tree.get('x_type').lower() == 'auth_only':
                self.write({'acquirer_reference': tree.get('x_trans_id')})
                self._set_transaction_authorized()
                self.execute_callback()
            if tree.get('x_type').lower() == 'void':
                self._set_transaction_cancel()
            return True
        elif status_code == self._authorize_pending_tx_status:
            self.write({'acquirer_reference': tree.get('x_trans_id')})
            self._set_transaction_pending()
            return True
        else:
            error = tree.get('x_response_reason_text')
            _logger.info(error)
            self.write({
                'acquirer_reference': tree.get('x_trans_id'),
            })
            self._set_transaction_error(msg=error)
            return False


class PaymentToken(models.Model):
    _inherit = 'payment.token'

    authorize_profile = fields.Char(string='Authorize.net Profile ID', help='This contains the unique reference '
                                    'for this partner/payment token combination in the Authorize.net backend')
    provider = fields.Selection(string='Provider', related='acquirer_id.provider', readonly=False)
    save_token = fields.Selection(string='Save Cards', related='acquirer_id.save_token', readonly=False)

    @api.model
    def authorize_create(self, values):
        if values.get('opaqueData') and values.get('encryptedCardData'):
            acquirer = self.env['payment.acquirer'].browse(values['acquirer_id'])
            partner = self.env['res.partner'].browse(values['partner_id'])
            transaction = AuthorizeAPI(acquirer)
            res = transaction.create_customer_profile(partner, values['opaqueData'])
            if res.get('profile_id') and res.get('payment_profile_id'):
                return {
                    'authorize_profile': res.get('profile_id'),
                    'name': values['encryptedCardData'].get('cardNumber'),
                    'acquirer_ref': res.get('payment_profile_id'),
                    'verified': True
                }
            else:
                raise ValidationError(_('The Customer Profile creation in Authorize.NET failed.'))
        else:
            return values

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
from . import payment

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#94B6C8"/><stop offset="100%" stop-color="#6A9EBA"/></linearGradient><path id="d" d="M19.25 46h2.714v1.212c0 .188.152.343.339.343h26.394a.342.342 0 0 0 .339-.343V37.5H33.625l-2.62-6.5h18.031v-3.212a.342.342 0 0 0-.339-.343H29.765l-1.018-2.742h20.289c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742V46zm-3.79 9.642h-1.037V55h2.833v.642h-1.037v2.832h-.76v-2.832zM17.422 55h1.07l.809 2.389h.01L20.075 55h1.07v3.474h-.711v-2.462h-.01l-.847 2.462h-.586l-.848-2.438h-.01v2.438h-.711V55zm17.163-12.266c0 .228-.19.342-.57.342-.253 0-.481-.013-.684-.038l-4.826-.076-4.826.076a4.036 4.036 0 0 1-.57.038c-.507 0-.76-.127-.76-.38 0-.279.342-.418 1.026-.418 2.23-.025 3.344-.253 3.344-.684 0-.025-.025-.127-.076-.304l-.304-1.026-.19-.532c-.228-.785-.785-2.52-1.672-5.206h-8.132c-1.52 4.053-2.28 6.384-2.28 6.992 0 .355.152.57.456.646.076.025.71.076 1.9.152.735.025 1.102.152 1.102.38 0 .253-.33.38-.988.38-1.165 0-2.52-.05-4.066-.152-.38-.025-.773-.038-1.178-.038-.405 0-.798.025-1.178.076A4.157 4.157 0 0 1 9.62 43c-.355 0-.532-.114-.532-.342 0-.228.399-.355 1.197-.38.798-.025 1.374-.146 1.729-.361s.633-.627.836-1.235c3.268-9.323 5.928-16.809 7.98-22.458a5.04 5.04 0 0 1 .304-1.026l.19-.684c.076-.279.177-.418.304-.418.076 0 .14.076.19.228.33.988 1.077 3.053 2.242 6.194.912 2.457 3.18 8.626 6.802 18.506.203.557.462.912.779 1.064.317.152.969.24 1.957.266.659.025.988.152.988.38zM24.25 33.652l-2.014-5.662-1.824-5.206c-1.064 2.787-2.343 6.41-3.838 10.868h7.676z"/><path id="e" d="M19.25 44h2.714v1.212c0 .188.152.343.339.343h26.394a.342.342 0 0 0 .339-.343V35.5H33.625l-2.62-6.5h18.031v-3.212a.342.342 0 0 0-.339-.343H29.765l-1.018-2.742h20.289c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742V44zm-3.79 9.642h-1.037V53h2.833v.642h-1.037v2.832h-.76v-2.832zM17.422 53h1.07l.809 2.389h.01L20.075 53h1.07v3.474h-.711v-2.462h-.01l-.847 2.462h-.586l-.848-2.438h-.01v2.438h-.711V53zm17.163-12.266c0 .228-.19.342-.57.342-.253 0-.481-.013-.684-.038l-4.826-.076-4.826.076a4.036 4.036 0 0 1-.57.038c-.507 0-.76-.127-.76-.38 0-.279.342-.418 1.026-.418 2.23-.025 3.344-.253 3.344-.684 0-.025-.025-.127-.076-.304l-.304-1.026-.19-.532c-.228-.785-.785-2.52-1.672-5.206h-8.132c-1.52 4.053-2.28 6.384-2.28 6.992 0 .355.152.57.456.646.076.025.71.076 1.9.152.735.025 1.102.152 1.102.38 0 .253-.33.38-.988.38-1.165 0-2.52-.05-4.066-.152-.38-.025-.773-.038-1.178-.038-.405 0-.798.025-1.178.076A4.157 4.157 0 0 1 9.62 41c-.355 0-.532-.114-.532-.342 0-.228.399-.355 1.197-.38.798-.025 1.374-.146 1.729-.361s.633-.627.836-1.235c3.268-9.323 5.928-16.809 7.98-22.458a5.04 5.04 0 0 1 .304-1.026l.19-.684c.076-.279.177-.418.304-.418.076 0 .14.076.19.228.33.988 1.077 3.053 2.242 6.194.912 2.457 3.18 8.626 6.802 18.506.203.557.462.912.779 1.064.317.152.969.24 1.957.266.659.025.988.152.988.38zM24.25 31.652l-2.014-5.662-1.824-5.206c-1.064 2.787-2.343 6.41-3.838 10.868h7.676z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M4 69c-2 0-4-1-4-4V33.916l21.6-19.82 3.4 8.607h12.25L48 23l3.576 1.968-.236 22.01L39.224 69H4z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: static\src\js\payment_form.js

```javascript
odoo.define('payment_authorize.payment_form', function (require) {
"use strict";

var ajax = require('web.ajax');
var core = require('web.core');
var PaymentForm = require('payment.payment_form');

var _t = core._t;

PaymentForm.include({

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Returns the parameters for the AcceptUI button that AcceptJS will use.
     *
     * @private
     * @param {Object} formData data obtained by getFormData
     * @returns {Object} params for the AcceptJS button
     */
    _acceptJsParams: function (formData) {
        return {
            'class': 'AcceptUI d-none',
            'data-apiLoginID': formData.login_id,
            'data-clientKey': formData.client_key,
            'data-billingAddressOptions': '{"show": false, "required": false}',
            'data-responseHandler': 'responseHandler'
        };
    },

    /**
     * called when clicking on pay now or add payment event to create token for credit card/debit card.
     *
     * @private
     * @param {Event} ev
     * @param {DOMElement} checkedRadio
     * @param {Boolean} addPmEvent
     */
    _createAuthorizeToken: function (ev, $checkedRadio, addPmEvent) {
        var self = this;
        if (ev.type === 'submit') {
            var button = $(ev.target).find('*[type="submit"]')[0]
        } else {
            var button = ev.target;
        }
        var acquirerID = this.getAcquirerIdFromRadio($checkedRadio);
        var acquirerForm = this.$('#o_payment_add_token_acq_' + acquirerID);
        var inputsForm = $('input', acquirerForm);
        var formData = self.getFormData(inputsForm);
        if (this.options.partnerId === undefined) {
            console.warn('payment_form: unset partner_id when adding new token; things could go wrong');
        }
        var AcceptJs = false;
        if (formData.acquirer_state === 'enabled') {
            AcceptJs = 'https://js.authorize.net/v3/AcceptUI.js';
        } else {
            AcceptJs = 'https://jstest.authorize.net/v3/AcceptUI.js';
        }

        window.responseHandler = function (response) {
            _.extend(formData, response);

            if (response.messages.resultCode === "Error") {
                var errorMessage = "";
                _.each(response.messages.message, function (message) {
                    errorMessage += message.code + ": " + message.text;
                })
                acquirerForm.removeClass('d-none');
                return self.displayError(_t('Server Error'), errorMessage);
            }

            self._rpc({
                route: formData.data_set,
                params: formData
            }).then (function (data) {
                if (addPmEvent) {
                    if (formData.return_url) {
                        window.location = formData.return_url;
                    } else {
                        window.location.reload();
                    }
                } else {
                    $checkedRadio.val(data.id);
                    self.el.submit();
                }
            }).guardedCatch(function (error) {
                // if the rpc fails, pretty obvious
                error.event.preventDefault();
                acquirerForm.removeClass('d-none');
                self.displayError(
                    _t('Server Error'),
                    _t("We are not able to add your payment method at the moment.") +
                        self._parseError(error)
                );
            });
        };

        if (this.$button === undefined) {
            this.$button = $('<button>', this._acceptJsParams(formData));
            this.$button.appendTo('body');
        }
        ajax.loadJS(AcceptJs).then(function () {
            self.$button.trigger('click');
        });
    },
    /**
     * @override
     */
    updateNewPaymentDisplayStatus: function () {
        var $checkedRadio = this.$('input[type="radio"]:checked');

        if ($checkedRadio.length !== 1) {
            return;
        }

        //  hide add token form for authorize
        if ($checkedRadio.data('provider') === 'authorize' && this.isNewPaymentRadio($checkedRadio)) {
            this.$('[id*="o_payment_add_token_acq_"]').addClass('d-none');
        } else {
            this._super.apply(this, arguments);
        }
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    payEvent: function (ev) {
        ev.preventDefault();
        var $checkedRadio = this.$('input[type="radio"]:checked');

        // first we check that the user has selected a authorize as s2s payment method
        if ($checkedRadio.length === 1 && this.isNewPaymentRadio($checkedRadio) && $checkedRadio.data('provider') === 'authorize') {
            this._createAuthorizeToken(ev, $checkedRadio);
        } else {
            this._super.apply(this, arguments);
        }
    },
    /**
     * @override
     */
    addPmEvent: function (ev) {
        ev.stopPropagation();
        ev.preventDefault();
        var $checkedRadio = this.$('input[type="radio"]:checked');

        // first we check that the user has selected a authorize as add payment method
        if ($checkedRadio.length === 1 && this.isNewPaymentRadio($checkedRadio) && $checkedRadio.data('provider') === 'authorize') {
            this._createAuthorizeToken(ev, $checkedRadio, true);
        } else {
            this._super.apply(this, arguments);
        }
    },
});
});

```

## File: views\payment_authorize_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <template id="authorize_form">
            <div>
                <input type="hidden" name="data_set" t-att-data-action-url="tx_url" data-remove-me=""/>
                <input type="hidden" name='x_login' t-att-value='x_login'/>
                <input type="hidden" name='x_fp_hash' t-att-value='x_fp_hash'/>
                <input type="hidden" name='x_amount' t-att-value='x_amount'/>
                <input type="hidden" name='x_show_form' t-att-value="x_show_form"/>
                <input type="hidden" name='x_type' t-att-value="x_type"/>
                <input type="hidden" name='x_method' t-att-value="x_method"/>
                <input type="hidden" name='x_fp_sequence' t-att-value='x_fp_sequence'/>
                <input type="hidden" name='x_version' t-att-value="x_version"/>
                <input type="hidden" name="x_relay_response" t-att-value="x_relay_response"/>
                <input type="hidden" name="x_relay_url" t-att-value="x_relay_url"/>
                <input type="hidden" name="x_fp_timestamp" t-att-value="x_fp_timestamp"/>
                <input type="hidden" name='return_url' t-att-value="returndata"/>
                <input type="hidden" name='x_cancel_url' t-att-value="x_cancel_url"/>
                <!--Order Information -->
                <input type="hidden" name='x_invoice_num' t-att-value='reference'/>
                <input type="hidden" name='x_description' t-att-value='reference'/>
                <input type="hidden" name='x_currency_code' t-att-value='x_currency_code'/>
                <!-- Billing Information-->
                <input type="hidden" name='x_first_name' t-att-value="billing_first_name"/>
                <input type="hidden" name='x_last_name' t-att-value="billing_last_name"/>
                <input type="hidden" name="x_company" t-att-value="billing_partner_commercial_company_name"/>
                <input type="hidden" name='x_address' t-att-value="billing_address"/>
                <input type="hidden" name='x_city' t-att-value="billing_city"/>
                <input type="hidden" name='x_zip' t-att-value="billing_zip_code"/>
                <input type="hidden" name='x_country' t-att-value="billing_country"/>
                <input type="hidden" name='x_phone' t-att-value='billing_phone'/>
                <input type="hidden" name='x_email' t-att-value="billing_email"/>
                <input type="hidden" name='x_state' t-att-value="billing_state"/>
                <!-- Shipping Information-->
                <input type="hidden" name='x_ship_to_first_name' t-att-value="first_name"/>
                <input type="hidden" name='x_ship_to_last_name' t-att-value="last_name"/>
                <input type="hidden" name='x_ship_to_address' t-att-value="address"/>
                <input type="hidden" name='x_ship_to_city' t-att-value="city"/>
                <input type="hidden" name='x_ship_to_zip' t-att-value="zip_code"/>
                <input type="hidden" name='x_ship_to_country' t-att-value="country"/>
                <input type="hidden" name='x_ship_to_phone' t-att-value='phone'/>
                <input type="hidden" name='x_ship_to_email' t-att-value="email"/>
                <input type="hidden" name='x_ship_to_state' t-att-value="state"/>
            </div>
        </template>

        <template id="payment_authorize_redirect" name="Payment Authorize">
            <script type="text/javascript">
                window.location.href = '<t t-esc="return_url"/>';
            </script>
        </template>

        <template id="authorize_s2s_form">
            <input type="hidden" name="data_set" value="/payment/authorize/s2s/create_json_3ds"/>
            <input type="hidden" name="acquirer_id" t-att-value="id"/>
            <input type="hidden" name="acquirer_state" t-att-value="acq.state"/>
            <input type="hidden" name="login_id" t-att-value="acq.sudo().authorize_login"/>
            <input type="hidden" name="client_key" t-att-value="acq.sudo().authorize_client_key"/>
            <input class="d-none" name="csrf_token" t-att-value="request.csrf_token()"/>
            <input t-if="return_url" type="hidden" name="return_url" t-att-value="return_url"/>
            <input t-if="partner_id" type="hidden" name="partner_id" t-att-value="partner_id"/>
         </template>

         <template id="assets_frontend" inherit_id="web.assets_frontend">
            <xpath expr="script[last()]" position="after">
                <script type="text/javascript" src="/payment_authorize/static/src/js/payment_form.js"></script>
            </xpath>
        </template>
    </data>
</odoo>

```

## File: views\payment_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="acquirer_form_authorize" model="ir.ui.view">
            <field name="name">acquirer.form.authorize</field>
            <field name="model">payment.acquirer</field>
            <field name="inherit_id" ref="payment.acquirer_form"/>
            <field name="arch" type="xml">
                <xpath expr='//group[@name="acquirer"]' position='inside'>
                    <group attrs="{'invisible': [('provider', '!=', 'authorize')]}">
                        <field name="authorize_login" attrs="{'required':[ ('provider', '=', 'authorize'), ('state', '!=', 'disabled')]}"/>
                        <field name="authorize_transaction_key" password="True" attrs="{'required':[ ('provider', '=', 'authorize'), ('state', '!=', 'disabled')]}"/>
                        <field name="authorize_signature_key" password="True" attrs="{'required':[ ('provider', '=', 'authorize'), ('state', '!=', 'disabled')]}"/>
                        <label for="authorize_client_key"/>
                        <div>
                            <field name="authorize_client_key" password="True"/>
                            <button class="oe_link" icon="fa-refresh" type="object" name="action_client_secret" string="Generate Client Key" />
                        </div>
                        <a colspan="2" href="https://www.odoo.com/documentation/14.0/applications/general/payment_acquirers/authorize.html" target="_blank">How to get paid with Authorize.Net</a>
                    </group>
                </xpath>
            </field>
        </record>

        <record id="token_form_authorize_net" model="ir.ui.view">
            <field name='name'>payment.token.form</field>
            <field name='model'>payment.token</field>
            <field name="inherit_id" ref="payment.payment_token_form_view"/>
            <field name="arch" type="xml">
                <xpath expr='//field[@name="acquirer_ref"]' position='after'>
                    <field name="authorize_profile" attrs="{'invisible':['|', ('provider', '!=', 'authorize'), ('save_token', '=', 'none')]}"/>
                    <field name="provider" invisible='1'/>
                    <field name="save_token" invisible='1'/>
                </xpath>
            </field>
        </record>
</odoo>

```

