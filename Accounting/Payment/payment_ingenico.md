# Odoo Module: payment_ingenico

Category: Accounting/Payment

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
    reset_payment_provider(cr, registry, 'ogone')

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-

{
    'name': 'Ingenico Payment Acquirer',
    'category': 'Accounting/Payment',
    'summary': 'Payment Acquirer: Ingenico Implementation',
    'version': '1.0',
    'description': """Ingenico Payment Acquirer""",
    'depends': ['payment'],
    'data': [
        'views/payment_views.xml',
        'views/payment_ingenico_templates.xml',
        'data/payment_acquirer_data.xml',
    ],
    'installable': True,
    'post_init_hook': 'create_missing_journal_for_acquirers',
    'uninstall_hook': 'uninstall_hook',
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
import logging
import pprint
import werkzeug
from werkzeug.urls import url_unquote_plus

from odoo import http
from odoo.http import request
from odoo.addons.payment.models.payment_acquirer import ValidationError
from odoo.addons.payment.controllers.portal import PaymentProcessing

_logger = logging.getLogger(__name__)


class OgoneController(http.Controller):
    _accept_url = '/payment/ogone/test/accept'
    _decline_url = '/payment/ogone/test/decline'
    _exception_url = '/payment/ogone/test/exception'
    _cancel_url = '/payment/ogone/test/cancel'

    @http.route([
        '/payment/ogone/accept', '/payment/ogone/test/accept',
        '/payment/ogone/decline', '/payment/ogone/test/decline',
        '/payment/ogone/exception', '/payment/ogone/test/exception',
        '/payment/ogone/cancel', '/payment/ogone/test/cancel',
    ], type='http', auth='public', csrf=False)
    def ogone_form_feedback(self, **post):
        """ Handle both redirection from Ingenico (GET) and s2s notification (POST/GET) """
        _logger.info('Ogone: entering form_feedback with post data %s', pprint.pformat(post))  # debug
        request.env['payment.transaction'].sudo().form_feedback(post, 'ogone')
        return werkzeug.utils.redirect("/payment/process")

    @http.route(['/payment/ogone/s2s/create_json'], type='json', auth='public', csrf=False)
    def ogone_s2s_create_json(self, **kwargs):
        if not kwargs.get('partner_id'):
            kwargs = dict(kwargs, partner_id=request.env.user.partner_id.id)
        new_id = request.env['payment.acquirer'].browse(int(kwargs.get('acquirer_id'))).s2s_process(kwargs)
        return new_id.id

    @http.route(['/payment/ogone/s2s/create_json_3ds'], type='json', auth='public', csrf=False)
    def ogone_s2s_create_json_3ds(self, verify_validity=False, **kwargs):
        if not kwargs.get('partner_id'):
            kwargs = dict(kwargs, partner_id=request.env.user.partner_id.id)
        token = False
        error = None

        try:
            token = request.env['payment.acquirer'].browse(int(kwargs.get('acquirer_id'))).s2s_process(kwargs)
        except Exception as e:
            error = str(e)

        if not token:
            res = {
                'result': False,
                'error': error,
            }
            return res

        res = {
            'result': True,
            'id': token.id,
            'short_name': token.short_name,
            '3d_secure': False,
            'verified': False,
        }

        if verify_validity != False:
            baseurl = request.env['ir.config_parameter'].sudo().get_param('web.base.url')
            params = {
                'accept_url': baseurl + '/payment/ogone/validate/accept',
                'decline_url': baseurl + '/payment/ogone/validate/decline',
                'exception_url': baseurl + '/payment/ogone/validate/exception',
                'return_url': kwargs.get('return_url', baseurl)
                }
            tx = token.validate(**params)
            res['verified'] = token.verified

            if tx and tx.html_3ds:
                res['3d_secure'] = tx.html_3ds

        return res

    @http.route(['/payment/ogone/s2s/create'], type='http', auth='public', methods=["POST"], csrf=False)
    def ogone_s2s_create(self, **post):
        error = ''
        acq = request.env['payment.acquirer'].browse(int(post.get('acquirer_id')))
        try:
            token = acq.s2s_process(post)
        except Exception as e:
            # synthax error: 'CHECK ERROR: |Not a valid date\n\n50001111: None'
            token = False
            error = str(e).splitlines()[0].split('|')[-1] or ''

        if token and post.get('verify_validity'):
            baseurl = request.env['ir.config_parameter'].sudo().get_param('web.base.url')
            params = {
                'accept_url': baseurl + '/payment/ogone/validate/accept',
                'decline_url': baseurl + '/payment/ogone/validate/decline',
                'exception_url': baseurl + '/payment/ogone/validate/exception',
                'return_url': post.get('return_url', baseurl)
                }
            tx = token.validate(**params)
            if tx and tx.html_3ds:
                return tx.html_3ds
            # add the payment transaction into the session to let the page /payment/process to handle it
            PaymentProcessing.add_payment_transaction(tx)
        return werkzeug.utils.redirect("/payment/process")

    @http.route([
        '/payment/ogone/validate/accept',
        '/payment/ogone/validate/decline',
        '/payment/ogone/validate/exception',
    ], type='http', auth='public')
    def ogone_validation_form_feedback(self, **post):
        """ Feedback from 3d secure for a bank card validation """
        request.env['payment.transaction'].sudo().form_feedback(post, 'ogone')
        return werkzeug.utils.redirect("/payment/process")

    @http.route(['/payment/ogone/s2s/feedback'], auth='public', csrf=False)
    def feedback(self, **kwargs):
        try:
            tx = request.env['payment.transaction'].sudo()._ogone_form_get_tx_from_data(kwargs)
            tx._ogone_s2s_validate_tree(kwargs)
        except ValidationError:
            return 'ko'
        return 'ok'

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-

from . import main

```

## File: data\ogone.py

```python
# -*- coding: utf-8 -*-

OGONE_ERROR_MAP = {
    '0020001001': "Authorization failed, please retry",
    '0020001002': "Authorization failed, please retry",
    '0020001003': "Authorization failed, please retry",
    '0020001004': "Authorization failed, please retry",
    '0020001005': "Authorization failed, please retry",
    '0020001006': "Authorization failed, please retry",
    '0020001007': "Authorization failed, please retry",
    '0020001008': "Authorization failed, please retry",
    '0020001009': "Authorization failed, please retry",
    '0020001010': "Authorization failed, please retry",
    '0030001999': "Our payment system is currently under maintenance, please try later",
    '0050001005': "Expiration Date error",
    '0050001007': "Requested Operation code not allowed",
    '0050001008': "Invalid delay value",
    '0050001010': "Input date in invalid format",
    '0050001013': "Unable to parse socket input stream",
    '0050001014': "Error in parsing stream content",
    '0050001015': "Currency error",
    '0050001016': "Transaction still posted at end of wait",
    '0050001017': "Sync value not compatible with delay value",
    '0050001019': "Transaction duplicate of a pre-existing transaction",
    '0050001020': "Acceptation code empty while required for the transaction",
    '0050001024': "Maintenance acquirer differs from original transaction acquirer",
    '0050001025': "Maintenance merchant differs from original transaction merchant",
    '0050001028': "Maintenance operation not accurate for the original transaction",
    '0050001031': "Host application unknown for the transaction",
    '0050001032': "Unable to perform requested operation with requested currency",
    '0050001033': "Maintenance card number differs from original transaction card number",
    '0050001034': "Operation code not allowed",
    '0050001035': "Exception occurred in socket input stream treatment",
    '0050001036': "Card length does not correspond to an acceptable value for the brand",
    '0050001036': "Card length does not correspond to an acceptable value for the brand",
    '0050001068': "A technical problem occurred, please contact helpdesk",
    '0050001069': "Invalid check for CardID and Brand",
    '0050001070': "A technical problem occurred, please contact helpdesk",
    '0050001116': "Unknown origin IP",
    '0050001117': "No origin IP detected",
    '0050001118': "Merchant configuration problem, please contact support",
    '10001001': "Communication failure",
    '10001002': "Communication failure",
    '10001003': "Communication failure",
    '10001004': "Communication failure",
    '10001005': "Communication failure",
    '20001001': "We received an unknown status for the transaction. We will contact your acquirer and update the status of the transaction within one working day. Please check the status later.",
    '20001002': "We received an unknown status for the transaction. We will contact your acquirer and update the status of the transaction within one working day. Please check the status later.",
    '20001003': "We received an unknown status for the transaction. We will contact your acquirer and update the status of the transaction within one working day. Please check the status later.",
    '20001004': "We received an unknown status for the transaction. We will contact your acquirer and update the status of the transaction within one working day. Please check the status later.",
    '20001005': "We received an unknown status for the transaction. We will contact your acquirer and update the status of the transaction within one working day. Please check the status later.",
    '20001006': "We received an unknown status for the transaction. We will contact your acquirer and update the status of the transaction within one working day. Please check the status later.",
    '20001007': "We received an unknown status for the transaction. We will contact your acquirer and update the status of the transaction within one working day. Please check the status later.",
    '20001008': "We received an unknown status for the transaction. We will contact your acquirer and update the status of the transaction within one working day. Please check the status later.",
    '20001009': "We received an unknown status for the transaction. We will contact your acquirer and update the status of the transaction within one working day. Please check the status later.",
    '20001010': "We received an unknown status for the transaction. We will contact your acquirer and update the status of the transaction within one working day. Please check the status later.",
    '20001101': "A technical problem occurred, please contact helpdesk",
    '20001105': "We received an unknown status for the transaction. We will contact your acquirer and update the status of the transaction within one working day. Please check the status later.",
    '20001111': "A technical problem occurred, please contact helpdesk",
    '20002001': "Origin for the response of the bank can not be checked",
    '20002002': "Beneficiary account number has been modified during processing",
    '20002003': "Amount has been modified during processing",
    '20002004': "Currency has been modified during processing",
    '20002005': "No feedback from the bank server has been detected",
    '30001001': "Payment refused by the acquirer",
    '30001002': "Duplicate request",
    '30001010': "A technical problem occurred, please contact helpdesk",
    '30001011': "A technical problem occurred, please contact helpdesk",
    '30001012': "Card black listed - Contact acquirer",
    '30001015': "Your merchant's acquirer is temporarily unavailable, please try later or choose another payment method.",
    '30001051': "A technical problem occurred, please contact helpdesk",
    '30001054': "A technical problem occurred, please contact helpdesk",
    '30001057': "Your merchant's acquirer is temporarily unavailable, please try later or choose another payment method.",
    '30001058': "Your merchant's acquirer is temporarily unavailable, please try later or choose another payment method.",
    '30001060': "Aquirer indicates that a failure occured during payment processing",
    '30001070': "RATEPAY Invalid Response Type (Failure)",
    '30001071': "RATEPAY Missing Mandatory status code field (failure)",
    '30001072': "RATEPAY Missing Mandatory Result code field (failure)",
    '30001073': "RATEPAY Response parsing Failed",
    '30001090': "CVC check required by front end and returned invalid by acquirer",
    '30001091': "ZIP check required by front end and returned invalid by acquirer",
    '30001092': "Address check required by front end and returned as invalid by acquirer.",
    '30001100': "Unauthorized buyer's country",
    '30001101': "IP country <> card country",
    '30001102': "Number of different countries too high",
    '30001103': "unauthorized card country",
    '30001104': "unauthorized ip address country",
    '30001105': "Anonymous proxy",
    '30001110': "If the problem persists, please contact Support, or go to paysafecard's card balance page (https://customer.cc.at.paysafecard.com/psccustomer/GetWelcomePanelServlet?language=en) to see when the amount reserved on your card will be available again.",
    '30001120': "IP address in merchant's black list",
    '30001130': "BIN in merchant's black list",
    '30001131': "Wrong BIN for 3xCB",
    '30001140': "Card in merchant's card blacklist",
    '30001141': "Email in blacklist",
    '30001142': "Passenger name in blacklist",
    '30001143': "Card holder name in blacklist",
    '30001144': "Passenger name different from owner name",
    '30001145': "Time to departure too short",
    '30001149': "Card Configured in Card Supplier Limit for another relation (CSL)",
    '30001150': "Card not configured in the system for this customer (CSL)",
    '30001151': "REF1 not allowed for this relationship (Contract number",
    '30001152': "Card/Supplier Amount limit reached (CSL)",
    '30001153': "Card not allowed for this supplier (Date out of contract bounds)",
    '30001154': "You have reached the usage limit allowed",
    '30001155': "You have reached the usage limit allowed",
    '30001156': "You have reached the usage limit allowed",
    '30001157': "Unauthorized IP country for itinerary",
    '30001158': "email usage limit reached",
    '30001159': "Unauthorized card country/IP country combination",
    '30001160': "Postcode in highrisk group",
    '30001161': "generic blacklist match",
    '30001162': "Billing Address is a PO Box",
    '30001180': "maximum scoring reached",
    '30001997': "Authorization canceled by simulation",
    '30001998': "A technical problem occurred, please try again.",
    '30001999': "Your merchant's acquirer is temporarily unavailable, please try later or choose another payment method.",
    '30002001': "Payment refused by the financial institution",
    '30002001': "Payment refused by the financial institution",
    '30021001': "Call acquirer support call number.",
    '30022001': "Payment must be approved by the acquirer before execution.",
    '30031001': "Invalid merchant number.",
    '30041001': "Retain card.",
    '30051001': "Authorization declined",
    '30071001': "Retain card - special conditions.",
    '30121001': "Invalid transaction",
    '30131001': "Invalid amount",
    '30131002': "You have reached the total amount allowed",
    '30141001': "Invalid card number",
    '30151001': "Unknown acquiring institution.",
    '30171001': "Payment method cancelled by the buyer",
    '30171002': "The maximum time allowed is elapsed.",
    '30191001': "Try again later.",
    '30201001': "A technical problem occurred, please contact helpdesk",
    '30301001': "Invalid format",
    '30311001': "Unknown acquirer ID.",
    '30331001': "Card expired.",
    '30341001': "Suspicion of fraud.",
    '30341002': "Suspicion of fraud (3rdMan)",
    '30341003': "Suspicion of fraud (Perseuss)",
    '30341004': "Suspicion of fraud (ETHOCA)",
    '30381001': "A technical problem occurred, please contact helpdesk",
    '30401001': "Invalid function.",
    '30411001': "Lost card.",
    '30431001': "Stolen card, pick up",
    '30511001': "Insufficient funds.",
    '30521001': "No Authorization. Contact the issuer of your card.",
    '30541001': "Card expired.",
    '30551001': "Invalid PIN.",
    '30561001': "Card not in authorizer's database.",
    '30571001': "Transaction not permitted on card.",
    '30581001': "Transaction not allowed on this terminal",
    '30591001': "Suspicion of fraud.",
    '30601001': "The merchant must contact the acquirer.",
    '30611001': "Amount exceeds card ceiling.",
    '30621001': "Restricted card.",
    '30631001': "Security policy not respected.",
    '30641001': "Amount changed from ref. trn.",
    '30681001': "Tardy response.",
    '30751001': "PIN entered incorrectly too often",
    '30761001': "Card holder already contesting.",
    '30771001': "PIN entry required.",
    '30811001': "Message flow error.",
    '30821001': "Authorization center unavailable",
    '30831001': "Authorization center unavailable",
    '30901001': "Temporary system shutdown.",
    '30911001': "Acquirer unavailable.",
    '30921001': "Invalid card type for acquirer.",
    '30941001': "Duplicate transaction",
    '30961001': "Processing temporarily not possible",
    '30971001': "A technical problem occurred, please contact helpdesk",
    '30981001': "A technical problem occurred, please contact helpdesk",
    '31011001': "Unknown acceptance code",
    '31021001': "Invalid currency",
    '31031001': "Acceptance code missing",
    '31041001': "Inactive card",
    '31051001': "Merchant not active",
    '31061001': "Invalid expiration date",
    '31071001': "Interrupted host communication",
    '31081001': "Card refused",
    '31091001': "Invalid password",
    '31101001': "Plafond transaction (majoré du bonus) dépassé",
    '31111001': "Plafond mensuel (majoré du bonus) dépassé",
    '31121001': "Plafond centre de facturation dépassé",
    '31131001': "Plafond entreprise dépassé",
    '31141001': "Code MCC du fournisseur non autorisé pour la carte",
    '31151001': "Numéro SIRET du fournisseur non autorisé pour la carte",
    '31161001': "This is not a valid online banking account",
    '32001004': "A technical problem occurred, please try again.",
    '34011001': "Bezahlung mit RatePAY nicht möglich.",
    '39991001': "A technical problem occurred, please contact the helpdesk of your acquirer",
    '40001001': "A technical problem occurred, please try again.",
    '40001002': "A technical problem occurred, please try again.",
    '40001003': "A technical problem occurred, please try again.",
    '40001004': "A technical problem occurred, please try again.",
    '40001005': "A technical problem occurred, please try again.",
    '40001006': "A technical problem occurred, please try again.",
    '40001007': "A technical problem occurred, please try again.",
    '40001008': "A technical problem occurred, please try again.",
    '40001009': "A technical problem occurred, please try again.",
    '40001010': "A technical problem occurred, please try again.",
    '40001011': "A technical problem occurred, please contact helpdesk",
    '40001012': "Your merchant's acquirer is temporarily unavailable, please try later or choose another payment method.",
    '40001013': "A technical problem occurred, please contact helpdesk",
    '40001016': "A technical problem occurred, please contact helpdesk",
    '40001018': "A technical problem occurred, please try again.",
    '40001019': "Sorry, an error occurred during processing. Please retry the operation (use back button of the browser). If problem persists, contact your merchant's helpdesk.",
    '40001020': "Sorry, an error occurred during processing. Please retry the operation (use back button of the browser). If problem persists, contact your merchant's helpdesk.",
    '40001050': "A technical problem occurred, please contact helpdesk",
    '40001133': "Authentication failed, the signature of your bank access control server is incorrect",
    '40001134': "Authentication failed, please retry or cancel.",
    '40001135': "Authentication temporary unavailable, please retry or cancel.",
    '40001136': "Technical problem with your browser, please retry or cancel",
    '40001137': "Your bank access control server is temporary unavailable, please retry or cancel",
    '40001998': "Temporary technical problem. Please retry a little bit later.",
    '50001001': "Unknown card type",
    '50001002': "Card number format check failed for given card number.",
    '50001003': "Merchant data error",
    '50001004': "Merchant identification missing",
    '50001005': "Expiration Date error",
    '50001006': "Amount is not a number",
    '50001007': "A technical problem occurred, please contact helpdesk",
    '50001008': "A technical problem occurred, please contact helpdesk",
    '50001009': "A technical problem occurred, please contact helpdesk",
    '50001010': "A technical problem occurred, please contact helpdesk",
    '50001011': "Brand not supported for that merchant",
    '50001012': "A technical problem occurred, please contact helpdesk",
    '50001013': "A technical problem occurred, please contact helpdesk",
    '50001014': "A technical problem occurred, please contact helpdesk",
    '50001015': "Invalid currency code",
    '50001016': "A technical problem occurred, please contact helpdesk",
    '50001017': "A technical problem occurred, please contact helpdesk",
    '50001018': "A technical problem occurred, please contact helpdesk",
    '50001019': "A technical problem occurred, please contact helpdesk",
    '50001020': "A technical problem occurred, please contact helpdesk",
    '50001021': "A technical problem occurred, please contact helpdesk",
    '50001022': "A technical problem occurred, please contact helpdesk",
    '50001023': "A technical problem occurred, please contact helpdesk",
    '50001024': "A technical problem occurred, please contact helpdesk",
    '50001025': "A technical problem occurred, please contact helpdesk",
    '50001026': "A technical problem occurred, please contact helpdesk",
    '50001027': "A technical problem occurred, please contact helpdesk",
    '50001028': "A technical problem occurred, please contact helpdesk",
    '50001029': "A technical problem occurred, please contact helpdesk",
    '50001030': "A technical problem occurred, please contact helpdesk",
    '50001031': "A technical problem occurred, please contact helpdesk",
    '50001032': "A technical problem occurred, please contact helpdesk",
    '50001033': "A technical problem occurred, please contact helpdesk",
    '50001034': "A technical problem occurred, please contact helpdesk",
    '50001035': "A technical problem occurred, please contact helpdesk",
    '50001036': "Card length does not correspond to an acceptable value for the brand",
    '50001037': "Purchasing card number for a regular merchant",
    '50001038': "Non Purchasing card for a Purchasing card merchant",
    '50001039': "Details sent for a non-Purchasing card merchant, please contact helpdesk",
    '50001040': "Details not sent for a Purchasing card transaction, please contact helpdesk",
    '50001041': "Payment detail validation failed",
    '50001042': "Given transactions amounts (tax,discount,shipping,net,etc…) do not compute correctly together",
    '50001043': "A technical problem occurred, please contact helpdesk",
    '50001044': "No acquirer configured for this operation",
    '50001045': "No UID configured for this operation",
    '50001046': "Operation not allowed for the merchant",
    '50001047': "A technical problem occurred, please contact helpdesk",
    '50001048': "A technical problem occurred, please contact helpdesk",
    '50001049': "A technical problem occurred, please contact helpdesk",
    '50001050': "A technical problem occurred, please contact helpdesk",
    '50001051': "A technical problem occurred, please contact helpdesk",
    '50001052': "A technical problem occurred, please contact helpdesk",
    '50001053': "A technical problem occurred, please contact helpdesk",
    '50001054': "Card number incorrect or incompatible",
    '50001055': "A technical problem occurred, please contact helpdesk",
    '50001056': "A technical problem occurred, please contact helpdesk",
    '50001057': "A technical problem occurred, please contact helpdesk",
    '50001058': "A technical problem occurred, please contact helpdesk",
    '50001059': "A technical problem occurred, please contact helpdesk",
    '50001060': "A technical problem occurred, please contact helpdesk",
    '50001061': "A technical problem occurred, please contact helpdesk",
    '50001062': "A technical problem occurred, please contact helpdesk",
    '50001063': "Card Issue Number does not correspond to range or not present",
    '50001064': "Start Date not valid or not present",
    '50001066': "Format of CVC code invalid",
    '50001067': "The merchant is not enrolled for 3D-Secure",
    '50001068': "The card number or account number (PAN) is invalid",
    '50001069': "Invalid check for CardID and Brand",
    '50001070': "The ECI value given is either not supported, or in conflict with other data in the transaction",
    '50001071': "Incomplete TRN demat",
    '50001072': "Incomplete PAY demat",
    '50001073': "No demat APP",
    '50001074': "Authorisation too old",
    '50001075': "VERRes was an error message",
    '50001076': "DCP amount greater than authorisation amount",
    '50001077': "Details negative amount",
    '50001078': "Details negative quantity",
    '50001079': "Could not decode/decompress received PARes (3D-Secure)",
    '50001080': "Received PARes was an erereor message from ACS (3D-Secure)",
    '50001081': "Received PARes format was invalid according to the 3DS specifications (3D-Secure)",
    '50001082': "PAReq/PARes reconciliation failure (3D-Secure)",
    '50001084': "Maximum amount reached",
    '50001087': "The transaction type requires authentication, please check with your bank.",
    '50001090': "CVC missing at input, but CVC check asked",
    '50001091': "ZIP missing at input, but ZIP check asked",
    '50001092': "Address missing at input, but Address check asked",
    '50001095': "Invalid date of birth",
    '50001096': "Invalid commodity code",
    '50001097': "The requested currency and brand are incompatible.",
    '50001111': "Data validation error",
    '50001113': "This order has already been processed",
    '50001114': "Error pre-payment check page access",
    '50001115': "Request not received in secure mode",
    '50001116': "Unknown IP address origin",
    '50001117': "NO IP address origin",
    '50001118': "Pspid not found or not correct",
    '50001119': "Password incorrect or disabled due to numbers of errors",
    '50001120': "Invalid currency",
    '50001121': "Invalid number of decimals for the currency",
    '50001122': "Currency not accepted by the merchant",
    '50001123': "Card type not active",
    '50001124': "Number of lines don't match with number of payments",
    '50001125': "Format validation error",
    '50001126': "Overflow in data capture requests for the original order",
    '50001127': "The original order is not in a correct status",
    '50001128': "missing authorization code for unauthorized order",
    '50001129': "Overflow in refunds requests",
    '50001130': "Error access to original order",
    '50001131': "Error access to original history item",
    '50001132': "The Selected Catalog is empty",
    '50001133': "Duplicate request",
    '50001134': "Authentication failed, please retry or cancel.",
    '50001135': "Authentication temporary unavailable, please retry or cancel.",
    '50001136': "Technical problem with your browser, please retry or cancel",
    '50001137': "Your bank access control server is temporary unavailable, please retry or cancel",
    '50001150': "Fraud Detection, Technical error (IP not valid)",
    '50001151': "Fraud detection : technical error (IPCTY unknown or error)",
    '50001152': "Fraud detection : technical error (CCCTY unknown or error)",
    '50001153': "Overflow in redo-authorisation requests",
    '50001170': "Dynamic BIN check failed",
    '50001171': "Dynamic country check failed",
    '50001172': "Error in Amadeus signature",
    '50001174': "Card Holder Name is too long",
    '50001175': "Name contains invalid characters",
    '50001176': "Card number is too long",
    '50001177': "Card number contains non-numeric info",
    '50001178': "Card Number Empty",
    '50001179': "CVC too long",
    '50001180': "CVC contains non-numeric info",
    '50001181': "Expiration date contains non-numeric info",
    '50001182': "Invalid expiration month",
    '50001183': "Expiration date must be in the future",
    '50001184': "SHA Mismatch",
    '50001205': "Missing mandatory fields for billing address.",
    '50001206': "Missing mandatory field date of birth.",
    '50001207': "Missing required shopping basket details.",
    '50001208': "Missing social security number",
    '50001209': "Invalid country code",
    '50001210': "Missing yearly salary",
    '50001211': "Missing gender",
    '50001212': "Missing email",
    '50001213': "Missing IP address",
    '50001214': "Missing part payment campaign ID",
    '50001215': "Missing invoice number",
    '50001216': "The alias must be different than the card number",
    '60000001': "account number unknown",
    '60000003': "not credited dd-mm-yy",
    '60000005': "name/number do not correspond",
    '60000007': "account number blocked",
    '60000008': "specific direct debit block",
    '60000009': "account number WKA",
    '60000010': "administrative reason",
    '60000011': "account number expired",
    '60000012': "no direct debit authorisation given",
    '60000013': "debit not approved",
    '60000014': "double payment",
    '60000018': "name/address/city not entered",
    '60001001': "no original direct debit for revocation",
    '60001002': "payer’s account number format error",
    '60001004': "payer’s account at different bank",
    '60001005': "payee’s account at different bank",
    '60001006': "payee’s account number format error",
    '60001007': "payer’s account number blocked",
    '60001008': "payer’s account number expired",
    '60001009': "payee’s account number expired",
    '60001010': "direct debit not possible",
    '60001011': "creditor payment not possible",
    '60001012': "payer’s account number unknown WKA-number",
    '60001013': "payee’s account number unknown WKA-number",
    '60001014': "impermissible WKA transaction",
    '60001015': "period for revocation expired",
    '60001017': "reason for revocation not correct",
    '60001018': "original run number not numeric",
    '60001019': "payment ID incorrect",
    '60001020': "amount not numeric",
    '60001021': "amount zero not permitted",
    '60001022': "negative amount not permitted",
    '60001023': "payer and payee giro account number",
    '60001025': "processing code (verwerkingscode) incorrect",
    '60001028': "revocation not permitted",
    '60001029': "guaranteed direct debit on giro account number",
    '60001030': "NBC transaction type incorrect",
    '60001031': "description too large",
    '60001032': "book account number not issued",
    '60001034': "book account number incorrect",
    '60001035': "payer’s account number not numeric",
    '60001036': "payer’s account number not eleven-proof",
    '60001037': "payer’s account number not issued",
    '60001039': "payer’s account number of DNB/BGC/BLA",
    '60001040': "payee’s account number not numeric",
    '60001041': "payee’s account number not eleven-proof",
    '60001042': "payee’s account number not issued",
    '60001044': "payee’s account number unknown",
    '60001050': "payee’s name missing",
    '60001051': "indicate payee’s bank account number instead of 3102",
    '60001052': "no direct debit contract",
    '60001053': "amount beyond bounds",
    '60001054': "selective direct debit block",
    '60001055': "original run number unknown",
    '60001057': "payer’s name missing",
    '60001058': "payee’s account number missing",
    '60001059': "restore not permitted",
    '60001060': "bank’s reference (navraaggegeven) missing",
    '60001061': "BEC/GBK number incorrect",
    '60001062': "BEC/GBK code incorrect",
    '60001087': "book account number not numeric",
    '60001090': "cancelled on request",
    '60001091': "cancellation order executed",
    '60001092': "cancelled instead of bended",
    '60001093': "book account number is a shortened account number",
    '60001094': "instructing party account number not identical with payer",
    '60001095': "payee unknown GBK acceptor",
    '60001097': "instructing party account number not identical with payee",
    '60001099': "clearing not permitted",
    '60001101': "payer’s account number not spaces",
    '60001102': "PAN length not numeric",
    '60001103': "PAN length outside limits",
    '60001104': "track number not numeric",
    '60001105': "track number not valid",
    '60001106': "PAN sequence number not numeric",
    '60001107': "domestic PAN not numeric",
    '60001108': "domestic PAN not eleven-proof",
    '60001109': "domestic PAN not issued",
    '60001110': "foreign PAN not numeric",
    '60001111': "card valid date not numeric",
    '60001112': "book period number (boekperiodenr) not numeric",
    '60001113': "transaction number not numeric",
    '60001114': "transaction time not numeric",
    '60001115': "transaction no valid time",
    '60001116': "transaction date not numeric",
    '60001117': "transaction no valid date",
    '60001118': "STAN not numeric",
    '60001119': "instructing party’s name missing",
    '60001120': "foreign amount (bedrag-vv) not numeric",
    '60001122': "rate (verrekenkoers) not numeric",
    '60001125': "number of decimals (aantaldecimalen) incorrect",
    '60001126': "tariff (tarifering) not B/O/S",
    '60001127': "domestic costs (kostenbinnenland) not numeric",
    '60001128': "domestic costs (kostenbinnenland) not higher than zero",
    '60001129': "foreign costs (kostenbuitenland) not numeric",
    '60001130': "foreign costs (kostenbuitenland) not higher than zero",
    '60001131': "domestic costs (kostenbinnenland) not zero",
    '60001132': "foreign costs (kostenbuitenland) not zero",
    '60001134': "Euro record not fully filled in",
    '60001135': "Client currency incorrect",
    '60001136': "Amount NLG not numeric",
    '60001137': "Amount NLG not higher than zero",
    '60001138': "Amount NLG not equal to Amount",
    '60001139': "Amount NLG incorrectly converted",
    '60001140': "Amount EUR not numeric",
    '60001141': "Amount EUR not greater than zero",
    '60001142': "Amount EUR not equal to Amount",
    '60001143': "Amount EUR incorrectly converted",
    '60001144': "Client currency not NLG",
    '60001145': "rate euro-vv (Koerseuro-vv) not numeric",
    '60001146': "comma rate euro-vv (Kommakoerseuro-vv) incorrect",
    '60001147': "acceptgiro distributor not valid",
    '60001148': "Original run number and/or BRN are missing",
    '60001149': "Amount/Account number/ BRN different",
    '60001150': "Direct debit already revoked/restored",
    '60001151': "Direct debit already reversed/revoked/restored",
    '60001153': "Payer’s account number not known",
}

DATA_VALIDATION_ERROR = '50001111'


def retryable(error):
    return error in [
        '0020001001', '0020001002', '0020001003', '0020001004', '0020001005',
        '0020001006', '0020001007', '0020001008', '0020001009', '0020001010',
        '30001010', '30001011', '30001015',
        '30001057', '30001058',
        '30001998', '30001999',
        #'30611001',     # amount exceeds card limit
        '30961001',
        '40001001', '40001002', '40001003', '40001004', '40001005',
        '40001006', '40001007', '40001008', '40001009', '40001010',
        '40001012',
        '40001018', '40001019', '40001020',
        '40001134', '40001135', '40001136', '40001137',
        #'50001174',      # cardholder name too long
    ]

```

## File: data\payment_acquirer_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <record id="payment.payment_acquirer_ingenico" model="payment.acquirer">
            <field name="name">Ingenico</field>
            <field name="image_128" type="base64" file="payment_ingenico/static/src/img/ingenico_icon.png"/>
            <field name="provider">ogone</field>
            <field name="company_id" ref="base.main_company"/>
            <field name="view_template_id" ref="ogone_form"/>
            <field name="registration_view_template_id" ref="ogone_s2s_form"/>
        </record>

        <record model="ir.config_parameter" id="payment_ogone_hash_function" forcecreate="False">
            <field name="key">payment_ogone.hash_function</field>
            <field name="value">sha1</field>
        </record>

    </data>
</odoo>

```

## File: data\__init__.py

```python
# -*- coding: utf-8 -*-

from . import ogone

```

## File: models\payment.py

```python
# coding: utf-8
import base64
import datetime
import logging
import time
from hashlib import new as hashnew
from pprint import pformat
from unicodedata import normalize

import requests
from lxml import etree, objectify
from werkzeug import urls, url_encode

from odoo import api, fields, models, _
from odoo.addons.payment.models.payment_acquirer import ValidationError
from odoo.addons.payment_ingenico.controllers.main import OgoneController
from odoo.addons.payment_ingenico.data import ogone
from odoo.http import request
from odoo.tools import DEFAULT_SERVER_DATE_FORMAT, ustr
from odoo.tools.float_utils import float_compare, float_repr, float_round

_logger = logging.getLogger(__name__)


class PaymentAcquirerOgone(models.Model):
    _inherit = 'payment.acquirer'

    provider = fields.Selection(selection_add=[('ogone', 'Ingenico')])
    ogone_pspid = fields.Char('PSPID', required_if_provider='ogone', groups='base.group_user')
    ogone_userid = fields.Char('API User ID', required_if_provider='ogone', groups='base.group_user')
    ogone_password = fields.Char('API User Password', required_if_provider='ogone', groups='base.group_user')
    ogone_shakey_in = fields.Char('SHA Key IN', required_if_provider='ogone', groups='base.group_user')
    ogone_shakey_out = fields.Char('SHA Key OUT', required_if_provider='ogone', groups='base.group_user')
    ogone_alias_usage = fields.Char('Alias Usage', default="Allow saving my payment data",
                                    help="If you want to use Ogone Aliases, this default "
                                    "Alias Usage will be presented to the customer as the "
                                    "reason you want to keep his payment data")

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
        res = super(PaymentAcquirerOgone, self)._get_feature_support()
        res['tokenize'].append('ogone')
        return res

    def _get_ogone_urls(self, environment):
        """ Ogone URLS:
         - standard order: POST address for form-based """
        return {
            'ogone_standard_order_url': 'https://secure.ogone.com/ncol/%s/orderstandard_utf8.asp' % (environment,),
            'ogone_direct_order_url': 'https://secure.ogone.com/ncol/%s/orderdirect_utf8.asp' % (environment,),
            'ogone_direct_query_url': 'https://secure.ogone.com/ncol/%s/querydirect_utf8.asp' % (environment,),
            'ogone_afu_agree_url': 'https://secure.ogone.com/ncol/%s/AFU_agree.asp' % (environment,),
            'ogone_maintenance_direct_url': 'https://secure.ogone.com/ncol/%s/maintenancedirect.asp' % (environment,),
        }

    def _ogone_generate_shasign(self, inout, values):
        """ Generate the shasign for incoming or outgoing communications.

        :param string inout: 'in' (odoo contacting ogone) or 'out' (ogone
                             contacting odoo). In this last case only some
                             fields should be contained (see e-Commerce basic)
        :param dict values: transaction values

        :return string: shasign
        """
        assert inout in ('in', 'out')
        assert self.provider == 'ogone'
        key = getattr(self, 'ogone_shakey_' + inout)

        def filter_key(key):
            if inout == 'in':
                return True
            else:
                # SHA-OUT keys
                # source https://payment-services.ingenico.com/int/en/ogone/support/guides/integration guides/e-commerce/transaction-feedback
                keys = [
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
                return key.upper() in keys

        items = sorted((k.upper(), v) for k, v in values.items())
        sign = ''.join('%s=%s%s' % (k, v, key) for k, v in items if v and filter_key(k))
        sign = sign.encode("utf-8")

        hash_function = self.env['ir.config_parameter'].sudo().get_param('payment_ogone.hash_function')
        if not hash_function or hash_function.lower() not in ['sha1', 'sha256', 'sha512']:
            hash_function = 'sha1'

        shasign = hashnew(hash_function)
        shasign.update(sign)
        return shasign.hexdigest()

    def ogone_form_generate_values(self, values):
        base_url = self.get_base_url()
        ogone_tx_values = dict(values)
        param_plus = {
            'return_url': ogone_tx_values.pop('return_url', False)
        }
        temp_ogone_tx_values = {
            'PSPID': self.ogone_pspid,
            'ORDERID': values['reference'],
            'AMOUNT': float_repr(float_round(values['amount'], 2) * 100, 0),
            'CURRENCY': values['currency'] and values['currency'].name or '',
            'LANGUAGE': values.get('partner_lang'),
            'CN': values.get('partner_name'),
            'EMAIL': values.get('partner_email'),
            'OWNERZIP': values.get('partner_zip'),
            'OWNERADDRESS': values.get('partner_address'),
            'OWNERTOWN': values.get('partner_city'),
            'OWNERCTY': values.get('partner_country') and values.get('partner_country').code or '',
            'OWNERTELNO': values.get('partner_phone'),
            'ACCEPTURL': urls.url_join(base_url, OgoneController._accept_url),
            'DECLINEURL': urls.url_join(base_url, OgoneController._decline_url),
            'EXCEPTIONURL': urls.url_join(base_url, OgoneController._exception_url),
            'CANCELURL': urls.url_join(base_url, OgoneController._cancel_url),
            'PARAMPLUS': url_encode(param_plus),
        }
        if self.save_token in ['ask', 'always']:
            temp_ogone_tx_values.update({
                'ALIAS': 'ODOO-NEW-ALIAS-%s' % time.time(),    # something unique,
                'ALIASUSAGE': values.get('alias_usage') or self.ogone_alias_usage,
            })
        shasign = self._ogone_generate_shasign('in', temp_ogone_tx_values)
        temp_ogone_tx_values['SHASIGN'] = shasign
        ogone_tx_values.update(temp_ogone_tx_values)
        return ogone_tx_values

    def ogone_get_form_action_url(self):
        self.ensure_one()
        environment = 'prod' if self.state == 'enabled' else 'test'
        return self._get_ogone_urls(environment)['ogone_standard_order_url']

    def ogone_s2s_form_validate(self, data):
        error = dict()

        mandatory_fields = ["cc_number", "cc_cvc", "cc_holder_name", "cc_expiry", "cc_brand"]
        # Validation
        for field_name in mandatory_fields:
            if not data.get(field_name):
                error[field_name] = 'missing'

        return False if error else True

    def ogone_s2s_form_process(self, data):
        values = {
            'cc_number': data.get('cc_number'),
            'cc_cvc': int(data.get('cc_cvc')),
            'cc_holder_name': data.get('cc_holder_name'),
            'cc_expiry': data.get('cc_expiry'),
            'cc_brand': data.get('cc_brand'),
            'acquirer_id': int(data.get('acquirer_id')),
            'partner_id': int(data.get('partner_id'))
        }
        pm_id = self.env['payment.token'].sudo().create(values)
        return pm_id


class PaymentTxOgone(models.Model):
    _inherit = 'payment.transaction'
    # ogone status
    _ogone_valid_tx_status = [5, 9, 8]
    _ogone_wait_tx_status = [41, 50, 51, 52, 55, 56, 91, 92, 99]
    _ogone_pending_tx_status = [46, 81, 82]   # 46 = 3DS HTML response
    _ogone_cancel_tx_status = [1]

    # --------------------------------------------------
    # FORM RELATED METHODS
    # --------------------------------------------------

    @api.model
    def _ogone_form_get_tx_from_data(self, data):
        """ Given a data dict coming from ogone, verify it and find the related
        transaction record. Create a payment token if an alias is returned."""
        reference, pay_id, shasign, alias = data.get('orderID'), data.get('PAYID'), data.get('SHASIGN'), data.get('ALIAS')
        if not reference or not pay_id or not shasign:
            error_msg = _('Ogone: received data with missing reference (%s) or pay_id (%s) or shasign (%s)') % (reference, pay_id, shasign)
            _logger.info(error_msg)
            raise ValidationError(error_msg)

        # find tx -> @TDENOTE use paytid ?
        tx = self.search([('reference', '=', reference)])
        if not tx or len(tx) > 1:
            error_msg = _('Ogone: received data for reference %s') % (reference)
            if not tx:
                error_msg += _('; no order found')
            else:
                error_msg += _('; multiple order found')
            _logger.info(error_msg)
            raise ValidationError(error_msg)

        # verify shasign
        shasign_check = tx.acquirer_id._ogone_generate_shasign('out', data)
        if shasign_check.upper() != shasign.upper():
            error_msg = _('Ogone: invalid shasign, received %s, computed %s, for data %s') % (shasign, shasign_check, data)
            _logger.info(error_msg)
            raise ValidationError(error_msg)

        if not tx.acquirer_reference:
            tx.acquirer_reference = pay_id

        # alias was created on ogone server, store it
        if alias and tx.type == 'form_save':
            Token = self.env['payment.token']
            domain = [('acquirer_ref', '=', alias)]
            cardholder = data.get('CN')
            if not Token.search_count(domain):
                _logger.info('Ogone: saving alias %s for partner %s' % (data.get('CARDNO'), tx.partner_id))
                ref = Token.create({'name': data.get('CARDNO') + (' - ' + cardholder if cardholder else ''),
                                    'partner_id': tx.partner_id.id,
                                    'acquirer_id': tx.acquirer_id.id,
                                    'acquirer_ref': alias})
                tx.write({'payment_token_id': ref.id})

        return tx

    def _ogone_form_get_invalid_parameters(self, data):
        invalid_parameters = []

        # TODO: txn_id: should be false at draft, set afterwards, and verified with txn details
        if self.acquirer_reference and data.get('PAYID') != self.acquirer_reference:
            invalid_parameters.append(('PAYID', data.get('PAYID'), self.acquirer_reference))
        # check what is bought
        if float_compare(float(data.get('amount', '0.0')), self.amount, 2) != 0:
            invalid_parameters.append(('amount', data.get('amount'), '%.2f' % self.amount))
        if data.get('currency') != self.currency_id.name:
            invalid_parameters.append(('currency', data.get('currency'), self.currency_id.name))

        return invalid_parameters

    def _ogone_form_validate(self, data):
        if self.state not in ['draft', 'pending']:
            _logger.info('Ogone: trying to validate an already validated tx (ref %s)', self.reference)
            return True

        status = int(data.get('STATUS', '0'))
        if status in self._ogone_valid_tx_status:
            vals = {
                'date': datetime.datetime.strptime(data['TRXDATE'], '%m/%d/%y').strftime(DEFAULT_SERVER_DATE_FORMAT),
                'acquirer_reference': data['PAYID'],
            }
            if data.get('ALIAS') and self.partner_id and \
               (self.type == 'form_save' or self.acquirer_id.save_token == 'always')\
               and not self.payment_token_id:
                pm = self.env['payment.token'].create({
                    'partner_id': self.partner_id.id,
                    'acquirer_id': self.acquirer_id.id,
                    'acquirer_ref': data.get('ALIAS'),
                    'name': '%s - %s' % (data.get('CARDNO'), data.get('CN'))
                })
                vals.update(payment_token_id=pm.id)
            self.write(vals)
            if self.payment_token_id:
                self.payment_token_id.verified = True
            self._set_transaction_done()
            self.execute_callback()
            # if this transaction is a validation one, then we refund the money we just withdrawn
            if self.type == 'validation':
                self.s2s_do_refund()

            return True
        elif status in self._ogone_cancel_tx_status:
            self.write({'acquirer_reference': data.get('PAYID')})
            self._set_transaction_cancel()
        elif status in self._ogone_pending_tx_status or status in self._ogone_wait_tx_status:
            self.write({'acquirer_reference': data.get('PAYID')})
            self._set_transaction_pending()
        else:
            error = 'Ogone: feedback error: %(error_str)s\n\n%(error_code)s: %(error_msg)s' % {
                'error_str': data.get('NCERRORPLUS'),
                'error_code': data.get('NCERROR'),
                'error_msg': ogone.OGONE_ERROR_MAP.get(data.get('NCERROR')),
            }
            _logger.info(error)
            self.write({
                'state_message': error,
                'acquirer_reference': data.get('PAYID'),
            })
            self._set_transaction_cancel()
            return False

    # --------------------------------------------------
    # S2S RELATED METHODS
    # --------------------------------------------------
    def ogone_s2s_do_transaction(self, **kwargs):
        # TODO: create tx with s2s type
        account = self.acquirer_id
        reference = self.reference or "ODOO-%s-%s" % (datetime.datetime.now().strftime('%y%m%d_%H%M%S'), self.partner_id.id)

        param_plus = {
            'return_url': kwargs.get('return_url', False)
        }

        data = {
            'PSPID': account.ogone_pspid,
            'USERID': account.ogone_userid,
            'PSWD': account.ogone_password,
            'ORDERID': reference,
            'AMOUNT': int(self.amount * 100),
            'CURRENCY': self.currency_id.name,
            'OPERATION': 'SAL',
            'ECI': 9,   # Recurring (from eCommerce)
            'ALIAS': self.payment_token_id.acquirer_ref,
            'RTIMEOUT': 30,
            'PARAMPLUS': url_encode(param_plus),
            'EMAIL': self.partner_id.email or '',
            'CN': self.partner_id.name or '',
        }

        if request:
            data['REMOTE_ADDR'] = request.httprequest.remote_addr

        if kwargs.get('3d_secure'):
            data.update({
                'FLAG3D': 'Y',
                'LANGUAGE': self.partner_id.lang or 'en_US',
            })

            for url in 'accept decline exception'.split():
                key = '{0}_url'.format(url)
                val = kwargs.pop(key, None)
                if val:
                    key = '{0}URL'.format(url).upper()
                    data[key] = val

        data['SHASIGN'] = self.acquirer_id._ogone_generate_shasign('in', data)

        direct_order_url = self.acquirer_id._get_ogone_urls('prod' if self.acquirer_id.state == 'enabled' else 'test')['ogone_direct_order_url']

        logged_data = data.copy()
        logged_data.pop('PSWD')
        _logger.info("ogone_s2s_do_transaction: Sending values to URL %s, values:\n%s", direct_order_url, pformat(logged_data))
        result = requests.post(direct_order_url, data=data).content

        try:
            tree = objectify.fromstring(result)
            _logger.info('ogone_s2s_do_transaction: Values received:\n%s', etree.tostring(tree, pretty_print=True, encoding='utf-8'))
        except etree.XMLSyntaxError:
            # invalid response from ogone
            _logger.exception('Invalid xml response from ogone')
            _logger.info('ogone_s2s_do_transaction: Values received:\n%s', result)
            raise

        return self._ogone_s2s_validate_tree(tree)

    def ogone_s2s_do_refund(self, **kwargs):
        account = self.acquirer_id
        reference = self.reference or "ODOO-%s-%s" % (datetime.datetime.now().strftime('%y%m%d_%H%M%S'), self.partner_id.id)

        data = {
            'PSPID': account.ogone_pspid,
            'USERID': account.ogone_userid,
            'PSWD': account.ogone_password,
            'ORDERID': reference,
            'AMOUNT': int(self.amount * 100),
            'CURRENCY': self.currency_id.name,
            'OPERATION': 'RFS',
            'PAYID': self.acquirer_reference,
        }
        data['SHASIGN'] = self.acquirer_id._ogone_generate_shasign('in', data)

        direct_order_url = self.acquirer_id._get_ogone_urls('prod' if self.acquirer_id.state == 'enabled' else 'test')['ogone_maintenance_direct_url']

        logged_data = data.copy()
        logged_data.pop('PSWD')
        _logger.info("ogone_s2s_do_refund: Sending values to URL %s, values:\n%s", direct_order_url, pformat(logged_data))
        result = requests.post(direct_order_url, data=data).content

        try:
            tree = objectify.fromstring(result)
            _logger.info('ogone_s2s_do_refund: Values received:\n%s', etree.tostring(tree, pretty_print=True, encoding='utf-8'))
        except etree.XMLSyntaxError:
            # invalid response from ogone
            _logger.exception('Invalid xml response from ogone')
            _logger.info('ogone_s2s_do_refund: Values received:\n%s', result)
            raise

        return self._ogone_s2s_validate_tree(tree)

    def _ogone_s2s_validate(self):
        tree = self._ogone_s2s_get_tx_status()
        return self._ogone_s2s_validate_tree(tree)

    def _ogone_s2s_validate_tree(self, tree, tries=2):
        if self.state not in ['draft', 'pending']:
            _logger.info('Ogone: trying to validate an already validated tx (ref %s)', self.reference)
            return True

        status = int(tree.get('STATUS') or 0)
        if status in self._ogone_valid_tx_status:
            self.write({
                'date': datetime.date.today().strftime(DEFAULT_SERVER_DATE_FORMAT),
                'acquirer_reference': tree.get('PAYID'),
            })
            if tree.get('ALIAS') and self.partner_id and \
               (self.type == 'form_save' or self.acquirer_id.save_token == 'always')\
               and not self.payment_token_id:
                pm = self.env['payment.token'].create({
                    'partner_id': self.partner_id.id,
                    'acquirer_id': self.acquirer_id.id,
                    'acquirer_ref': tree.get('ALIAS'),
                    'name': tree.get('CARDNO'),
                })
                self.write({'payment_token_id': pm.id})
            if self.payment_token_id:
                self.payment_token_id.verified = True
            self._set_transaction_done()
            self.execute_callback()
            # if this transaction is a validation one, then we refund the money we just withdrawn
            if self.type == 'validation':
                self.s2s_do_refund()
            return True
        elif status in self._ogone_cancel_tx_status:
            self.write({'acquirer_reference': tree.get('PAYID')})
            self._set_transaction_cancel()
        elif status in self._ogone_pending_tx_status:
            vals = {
                'acquirer_reference': tree.get('PAYID'),
            }
            if status == 46: # HTML 3DS
                vals['html_3ds'] = ustr(base64.b64decode(tree.HTML_ANSWER.text))
            self.write(vals)
            self._set_transaction_pending()
        elif status in self._ogone_wait_tx_status and tries > 0:
            time.sleep(0.5)
            self.write({'acquirer_reference': tree.get('PAYID')})
            tree = self._ogone_s2s_get_tx_status()
            return self._ogone_s2s_validate_tree(tree, tries - 1)
        else:
            error = 'Ogone: feedback error: %(error_str)s\n\n%(error_code)s: %(error_msg)s' % {
                'error_str': tree.get('NCERRORPLUS'),
                'error_code': tree.get('NCERROR'),
                'error_msg': ogone.OGONE_ERROR_MAP.get(tree.get('NCERROR')),
            }
            _logger.info(error)
            self.write({
                'state_message': error,
                'acquirer_reference': tree.get('PAYID'),
            })
            self._set_transaction_cancel()
            return False

    def _ogone_s2s_get_tx_status(self):
        account = self.acquirer_id
        #reference = tx.reference or "ODOO-%s-%s" % (datetime.datetime.now().strftime('%Y%m%d_%H%M%S'), tx.partner_id.id)

        data = {
            'PAYID': self.acquirer_reference,
            'PSPID': account.ogone_pspid,
            'USERID': account.ogone_userid,
            'PSWD': account.ogone_password,
        }

        query_direct_url = self.acquirer_id._get_ogone_urls('prod' if self.acquirer_id.state == 'enabled' else 'test')['ogone_direct_query_url']

        logged_data = data.copy()
        logged_data.pop('PSWD')

        _logger.info("_ogone_s2s_get_tx_status: Sending values to URL %s, values:\n%s", query_direct_url, pformat(logged_data))
        result = requests.post(query_direct_url, data=data).content

        try:
            tree = objectify.fromstring(result)
            _logger.info('_ogone_s2s_get_tx_status: Values received:\n%s', etree.tostring(tree, pretty_print=True, encoding='utf-8'))
        except etree.XMLSyntaxError:
            # invalid response from ogone
            _logger.exception('Invalid xml response from ogone')
            _logger.info('_ogone_s2s_get_tx_status: Values received:\n%s', result)
            raise

        return tree


class PaymentToken(models.Model):
    _inherit = 'payment.token'

    def ogone_create(self, values):
        if values.get('cc_number'):
            # create a alias via batch
            values['cc_number'] = values['cc_number'].replace(' ', '')
            acquirer = self.env['payment.acquirer'].browse(values['acquirer_id'])
            alias = 'ODOO-NEW-ALIAS-%s' % time.time()

            expiry = str(values['cc_expiry'][:2]) + str(values['cc_expiry'][-2:])
            line = 'ADDALIAS;%(alias)s;%(cc_holder_name)s;%(cc_number)s;%(expiry)s;%(cc_brand)s;%(pspid)s'
            line = line % dict(values, alias=alias, expiry=expiry, pspid=acquirer.ogone_pspid)

            data = {
                'FILE_REFERENCE': alias,
                'TRANSACTION_CODE': 'MTR',
                'OPERATION': 'SAL',
                'NB_PAYMENTS': 1,   # even if we do not actually have any payment, ogone want it to not be 0
                'FILE': normalize('NFKD', line).encode('ascii','ignore'),  # Ogone Batch must be ASCII only
                'REPLY_TYPE': 'XML',
                'PSPID': acquirer.ogone_pspid,
                'USERID': acquirer.ogone_userid,
                'PSWD': acquirer.ogone_password,
                'PROCESS_MODE': 'CHECKANDPROCESS',
            }

            url = acquirer._get_ogone_urls('prod' if acquirer.state == 'enabled' else 'test')['ogone_afu_agree_url']
            _logger.info("ogone_create: Creating new alias %s via url %s", alias, url)
            result = requests.post(url, data=data).content

            try:
                tree = objectify.fromstring(result)
            except etree.XMLSyntaxError:
                _logger.exception('Invalid xml response from ogone')
                return None

            error_code = error_str = None
            if hasattr(tree, 'PARAMS_ERROR'):
                error_code = tree.NCERROR.text
                error_str = 'PARAMS ERROR: %s' % (tree.PARAMS_ERROR.text or '',)
            else:
                node = tree.FORMAT_CHECK
                error_node = getattr(node, 'FORMAT_CHECK_ERROR', None)
                if error_node is not None:
                    error_code = error_node.NCERROR.text
                    error_str = 'CHECK ERROR: %s' % (error_node.ERROR.text or '',)

            if error_code:
                error_msg = tree.get(error_code)
                error = '%s\n\n%s: %s' % (error_str, error_code, error_msg)
                _logger.error(error)
                raise Exception(error)

            return {
                'acquirer_ref': alias,
                'name': 'XXXXXXXXXXXX%s - %s' % (values['cc_number'][-4:], values['cc_holder_name'])
            }
        return {}

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import payment

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#94B6C8"/><stop offset="100%" stop-color="#6A9EBA"/></linearGradient><path id="d" d="M19.25 42h2.714v5.212c0 .188.152.343.339.343h26.394a.342.342 0 0 0 .339-.343V37.5H30.732c.389-3.667-.582-5.833-2.914-6.5h21.218v-3.212a.342.342 0 0 0-.339-.343H29.765c.361-1.01.57-1.924.627-2.742h18.644c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742V42zm-3.79 13.642h-1.037V55h2.833v.642h-1.037v2.832h-.76v-2.832zM17.422 55h1.07l.809 2.389h.01L20.075 55h1.07v3.474h-.711v-2.462h-.01l-.847 2.462h-.586l-.848-2.438h-.01v2.438h-.711V55zm17.403-34.99c.985.007-2.543 2.376-2.271 2.073h-5.692c.113.126.201.23.298.334 1.214 1.315 1.076 3.081.239 4.432-.597.965-1.488 1.586-2.49 2.049-1.103.518-2.277.747-3.475.909a85.771 85.771 0 0 0-3.555.56c-.274.046-.546.186-.782.34-.383.25-.375.645.047.811.595.23 1.216.411 1.843.532 1.768.343 3.567.532 5.295 1.073.842.265 1.651.598 2.341 1.182 1.138.972 1.222 2.65.013 3.726-.948.85-2.094 1.272-3.295 1.557-1.024.243-2.063.36-3.117.396-2.206.076-4.376-.11-6.481-.823-.77-.265-1.497-.614-2.09-1.2-.333-.322-.582-.69-.64-1.174-.08-.656.198-1.127.696-1.496.701-.517 1.51-.775 2.335-.97.797-.192 1.606-.33 2.345-.482-.471-.161-1.025-.314-1.546-.545-.407-.185-.812-.406-1.146-.7-.655-.571-.655-1.42-.049-2.06.442-.468.997-.76 1.586-.974.513-.192 1.041-.34 1.567-.51-.328-.148-.676-.29-1.004-.459-.748-.39-1.42-.886-1.821-1.658-.777-1.496-.475-2.903.467-4.197.908-1.243 2.204-1.916 3.634-2.345a9.62 9.62 0 0 1 2.742-.387c4.671 0 9.339-.013 14.006.007zM18.778 34.016c-1.03.185-2.057.453-3.064.787-.495.16-.972.475-1.405.82-.421.333-.405.865.006 1.224.269.235.58.424.893.553 1.223.505 2.505.489 4.007.601.526-.08 1.283-.165 2.03-.322.546-.114 1.073-.34 1.483-.845.381-.462.363-.962-.063-1.357a2.674 2.674 0 0 0-.689-.487 25.158 25.158 0 0 0-1.952-.725c-.408-.133-.846-.318-1.246-.249zm4.666-7.55c.368-.527.59-1.137.552-1.813-.061-1.064-.609-1.772-1.446-2.208a3.71 3.71 0 0 0-2.357-.396c-1.155.18-2.127.73-2.802 1.824-.64 1.036-.479 2.313.366 3.145.737.729 1.629.933 2.647.983.182-.02.438-.041.69-.08.94-.162 1.748-.606 2.35-1.455z"/><path id="e" d="M19.25 40h2.714v5.212c0 .188.152.343.339.343h26.394a.342.342 0 0 0 .339-.343V35.5H30.732c.389-3.667-.582-5.833-2.914-6.5h21.218v-3.212a.342.342 0 0 0-.339-.343H29.765c.361-1.01.57-1.924.627-2.742h18.644c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742V40zm-3.79 13.642h-1.037V53h2.833v.642h-1.037v2.832h-.76v-2.832zM17.422 53h1.07l.809 2.389h.01L20.075 53h1.07v3.474h-.711v-2.462h-.01l-.847 2.462h-.586l-.848-2.438h-.01v2.438h-.711V53zm17.403-34.99c.985.007-2.543 2.376-2.271 2.073h-5.692c.113.126.201.23.298.334 1.214 1.315 1.076 3.081.239 4.432-.597.965-1.488 1.586-2.49 2.049-1.103.518-2.277.747-3.475.909a85.771 85.771 0 0 0-3.555.56c-.274.046-.546.186-.782.34-.383.25-.375.645.047.811.595.23 1.216.411 1.843.532 1.768.343 3.567.532 5.295 1.073.842.265 1.651.598 2.341 1.182 1.138.972 1.222 2.65.013 3.726-.948.85-2.094 1.272-3.295 1.557-1.024.243-2.063.36-3.117.396-2.206.076-4.376-.11-6.481-.823-.77-.265-1.497-.614-2.09-1.2-.333-.322-.582-.69-.64-1.174-.08-.656.198-1.127.696-1.496.701-.517 1.51-.775 2.335-.97.797-.192 1.606-.33 2.345-.482-.471-.161-1.025-.314-1.546-.545-.407-.185-.812-.406-1.146-.7-.655-.571-.655-1.42-.049-2.06.442-.468.997-.76 1.586-.974.513-.192 1.041-.34 1.567-.51-.328-.148-.676-.29-1.004-.459-.748-.39-1.42-.886-1.821-1.658-.777-1.496-.475-2.903.467-4.197.908-1.243 2.204-1.916 3.634-2.345a9.62 9.62 0 0 1 2.742-.387c4.671 0 9.339-.013 14.006.007zM18.778 32.016c-1.03.185-2.057.453-3.064.787-.495.16-.972.475-1.405.82-.421.333-.405.865.006 1.224.269.235.58.424.893.553 1.223.505 2.505.489 4.007.601.526-.08 1.283-.165 2.03-.322.546-.114 1.073-.34 1.483-.845.381-.462.363-.962-.063-1.357a2.674 2.674 0 0 0-.689-.487 25.158 25.158 0 0 0-1.952-.725c-.408-.133-.846-.318-1.246-.249zm4.666-7.55c.368-.527.59-1.137.552-1.813-.061-1.064-.609-1.772-1.446-2.208a3.71 3.71 0 0 0-2.357-.396c-1.155.18-2.127.73-2.802 1.824-.64 1.036-.479 2.313.366 3.145.737.729 1.629.933 2.647.983.182-.02.438-.041.69-.08.94-.162 1.748-.606 2.35-1.455z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M4 69c-2 0-4-1-4-4V33.916L15.591 19.5 34.912 18l-3.89 4.703h6.228L48 23l3.576 1.968-.236 22.01L39.224 69H4z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: views\payment_ingenico_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <template id="ogone_form">
            <input type="hidden" name="data_set" t-att-data-action-url="tx_url" data-remove-me=""/>
            <!-- seller -->
            <input type="hidden" name='PSPID' t-att-value='PSPID'/>
            <input type="hidden" name='ORDERID' t-att-value='ORDERID'/>
            <!-- cart -->
            <input type="hidden" name='AMOUNT' t-att-value='AMOUNT or "0.0"'/>
            <input type="hidden" name='CURRENCY' t-att-value='CURRENCY'/>
            <!-- buyer -->
            <input type="hidden" name='LANGUAGE' t-att-value='LANGUAGE'/>
            <input type="hidden" name='CN' t-att-value='CN'/>
            <input type="hidden" name='EMAIL' t-att-value='EMAIL'/>
            <input type="hidden" name='OWNERZIP' t-att-value='OWNERZIP'/>
            <input type="hidden" name='OWNERADDRESS' t-att-value='OWNERADDRESS'/>
            <input type="hidden" name='OWNERCTY' t-att-value='OWNERCTY'/>
            <input type="hidden" name='OWNERTOWN' t-att-value='OWNERTOWN'/>
            <input type="hidden" name='OWNERTELNO' t-att-value='OWNERTELNO'/>
            <t t-if="acquirer.save_token in ['ask', 'always']">
                <input type="hidden" name='ALIAS' t-att-value='ALIAS'/>
                <input type="hidden" name='ALIASUSAGE' t-att-value='ALIASUSAGE'/>
            </t>
            <!-- before payment verification -->
            <input type="hidden" name='SHASIGN' t-att-value='SHASIGN'/>
            <!-- after payment parameters -->
            <t t-if='PARAMPLUS'>
                <input type="hidden" name="PARAMPLUS" t-att-value='PARAMPLUS'/>
            </t>
            <!-- redirection -->
            <input type="hidden" name='ACCEPTURL' t-att-value='ACCEPTURL'/>
            <input type="hidden" name='DECLINEURL' t-att-value='DECLINEURL'/>
            <input type="hidden" name='EXCEPTIONURL' t-att-value='EXCEPTIONURL'/>
            <input type="hidden" name='CANCELURL' t-att-value='CANCELURL'/>
        </template>

        <template id="ogone_s2s_form">
            <input type="hidden" name="data_set" data-create-route="/payment/ogone/s2s/create_json_3ds"/>
            <input type="hidden" name="acquirer_id" t-att-value="id"/>
            <input t-if="return_url" type="hidden" name="return_url" t-att-value="return_url"/>
            <input t-if="partner_id" type="hidden" name="partner_id" t-att-value="partner_id"/>
            <div t-attf-class="row mt8 #{'' if bootstrap_formatting else 'o_card_brand_detail'}">
                <div t-att-class="'form-group col-lg-12' if bootstrap_formatting else 'form-group'">
                    <input type="tel" name="cc_number" id="cc_number" class="form-control" placeholder="Card number" data-is-required="true"/>
                    <div class="card_placeholder"></div>
                    <div class="visa"></div>
                    <input type="hidden" name="cc_brand" value=""/>
                </div>
                <div t-att-class="'form-group col-lg-5' if bootstrap_formatting else 'form-group'">
                    <input type="text" name="cc_holder_name" id="cc_holder_name" class="form-control" placeholder="Cardholder name" data-is-required="true"/>
                </div>
                <div t-att-class="'form-group col-lg-3' if bootstrap_formatting else 'form-group'">
                    <input type="text" name="cc_expiry" id="cc_expiry" class="form-control" maxlength="7" placeholder="Expires (MM / YY)" data-is-required="true"/>
                </div>
                <div t-att-class="'form-group col-lg-4' if bootstrap_formatting else 'form-group'">
                    <input type="text" name="cc_cvc" id="cc_cvc" class="form-control" maxlength="4" placeholder="CVC" data-is-required="true"/>
                </div>
            </div>
        </template>
    </data>
</odoo>

```

## File: views\payment_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record id="acquirer_form_ogone" model="ir.ui.view">
            <field name="name">acquirer.form.ogone</field>
            <field name="model">payment.acquirer</field>
            <field name="inherit_id" ref="payment.acquirer_form"/>
            <field name="arch" type="xml">
                <xpath expr='//group[@name="acquirer"]' position='inside'>
                    <group attrs="{'invisible': [('provider', '!=', 'ogone')]}">
                        <field name="ogone_pspid" attrs="{'required':[ ('provider', '=', 'ogone'), ('state', '!=', 'disabled')]}"/>
                        <field name="ogone_userid" attrs="{'required':[ ('provider', '=', 'ogone'), ('state', '!=', 'disabled')]}"/>
                        <field name="ogone_password" attrs="{'required':[ ('provider', '=', 'ogone'), ('state', '!=', 'disabled')]}"/>
                        <field name="ogone_shakey_in" attrs="{'required':[ ('provider', '=', 'ogone'), ('state', '!=', 'disabled')]}"/>
                        <field name="ogone_shakey_out" attrs="{'required':[ ('provider', '=', 'ogone'), ('state', '!=', 'disabled')]}"/>
                        <field name="ogone_alias_usage"/>
                    </group>
                </xpath>
            </field>
        </record>

    </data>
</odoo>

```

