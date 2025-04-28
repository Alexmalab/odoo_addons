# Odoo Module: payment

Category: Hidden

This file contains the source code of the Odoo module.

## File: const.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# According to https://en.wikipedia.org/wiki/ISO_4217#Minor_unit_fractions
CURRENCY_MINOR_UNITS = {
    'ADF': 2,
    'ADP': 0,
    'AED': 2,
    'AFA': 2,
    'AFN': 2,
    'ALL': 2,
    'AMD': 2,
    'ANG': 2,
    'AOA': 2,
    'AOK': 0,
    'AON': 0,
    'AOR': 0,
    'ARA': 2,
    'ARL': 2,
    'ARP': 2,
    'ARS': 2,
    'ATS': 2,
    'AUD': 2,
    'AWG': 2,
    'AYM': 0,
    'AZM': 2,
    'AZN': 2,
    'BAD': 2,
    'BAM': 2,
    'BBD': 2,
    'BDS': 2,
    'BDT': 2,
    'BEF': 2,
    'BGL': 2,
    'BGN': 2,
    'BHD': 3,
    'BIF': 0,
    'BMD': 2,
    'BND': 2,
    'BOB': 2,
    'BOP': 2,
    'BOV': 2,
    'BRB': 2,
    'BRC': 2,
    'BRE': 2,
    'BRL': 2,
    'BRN': 2,
    'BRR': 2,
    'BSD': 2,
    'BTN': 2,
    'BWP': 2,
    'BYB': 2,
    'BYN': 2,
    'BYR': 0,
    'BZD': 2,
    'CAD': 2,
    'CDF': 2,
    'CHC': 2,
    'CHE': 2,
    'CHF': 2,
    'CHW': 2,
    'CLF': 4,
    'CLP': 0,
    'CNH': 2,
    'CNT': 2,
    'CNY': 2,
    'COP': 2,
    'COU': 2,
    'CRC': 2,
    'CSD': 2,
    'CUC': 2,
    'CUP': 2,
    'CVE': 2,
    'CYP': 2,
    'CZK': 2,
    'DEM': 2,
    'DJF': 0,
    'DKK': 2,
    'DOP': 2,
    'DZD': 2,
    'ECS': 0,
    'ECV': 2,
    'EEK': 2,
    'EGP': 2,
    'ERN': 2,
    'ESP': 0,
    'ETB': 2,
    'EUR': 2,
    'FIM': 2,
    'FJD': 2,
    'FKP': 2,
    'FRF': 2,
    'GBP': 2,
    'GEK': 0,
    'GEL': 2,
    'GGP': 2,
    'GHC': 2,
    'GHP': 2,
    'GHS': 2,
    'GIP': 2,
    'GMD': 2,
    'GNF': 0,
    'GTQ': 2,
    'GWP': 2,
    'GYD': 2,
    'HKD': 2,
    'HNL': 2,
    'HRD': 2,
    'HRK': 2,
    'HTG': 2,
    'HUF': 2,
    'IDR': 2,
    'IEP': 2,
    'ILR': 2,
    'ILS': 2,
    'IMP': 2,
    'INR': 2,
    'IQD': 3,
    'IRR': 2,
    'ISJ': 2,
    'ISK': 0,
    'ITL': 0,
    'JEP': 2,
    'JMD': 2,
    'JOD': 3,
    'JPY': 0,
    'KES': 2,
    'KGS': 2,
    'KHR': 2,
    'KID': 2,
    'KMF': 0,
    'KPW': 2,
    'KRW': 0,
    'KWD': 3,
    'KYD': 2,
    'KZT': 2,
    'LAK': 2,
    'LBP': 2,
    'LKR': 2,
    'LRD': 2,
    'LSL': 2,
    'LTL': 2,
    'LTT': 2,
    'LUF': 2,
    'LVL': 2,
    'LVR': 2,
    'LYD': 3,
    'MAD': 2,
    'MAF': 2,
    'MCF': 2,
    'MDL': 2,
    'MGA': 2,
    'MGF': 0,
    'MKD': 2,
    'MMK': 2,
    'MNT': 2,
    'MOP': 2,
    'MRO': 2,
    'MRU': 2,
    'MTL': 2,
    'MUR': 2,
    'MVR': 2,
    'MWK': 2,
    'MXN': 2,
    'MXV': 2,
    'MYR': 2,
    'MZE': 2,
    'MZM': 2,
    'MZN': 2,
    'NAD': 2,
    'NGN': 2,
    'NIC': 2,
    'NIO': 2,
    'NIS': 2,
    'NLG': 2,
    'NOK': 2,
    'NPR': 2,
    'NTD': 2,
    'NZD': 2,
    'OMR': 3,
    'PAB': 2,
    'PEN': 2,
    'PES': 2,
    'PGK': 2,
    'PHP': 2,
    'PKR': 2,
    'PLN': 2,
    'PLZ': 2,
    'PRB': 2,
    'PTE': 0,
    'PYG': 0,
    'QAR': 2,
    'RHD': 2,
    'RMB': 2,
    'ROL': 0,
    'RON': 2,
    'RSD': 2,
    'RUB': 2,
    'RUR': 2,
    'RWF': 0,
    'SAR': 2,
    'SBD': 2,
    'SCR': 2,
    'SDD': 2,
    'SDG': 2,
    'SEK': 2,
    'SGD': 2,
    'SHP': 2,
    'SIT': 2,
    'SKK': 2,
    'SLE': 2,
    'SLL': 2,
    'SLS': 2,
    'SML': 0,
    'SOS': 2,
    'SRD': 2,
    'SRG': 2,
    'SSP': 2,
    'STD': 2,
    'STG': 2,
    'STN': 2,
    'SVC': 2,
    'SYP': 2,
    'SZL': 2,
    'THB': 2,
    'TJR': 0,
    'TJS': 2,
    'TMM': 2,
    'TMT': 2,
    'TND': 3,
    'TOP': 2,
    'TPE': 0,
    'TRL': 0,
    'TRY': 2,
    'TTD': 2,
    'TVD': 2,
    'TWD': 2,
    'TZS': 2,
    'UAH': 2,
    'UAK': 2,
    'UGX': 0,
    'USD': 2,
    'USN': 2,
    'USS': 2,
    'UYI': 0,
    'UYN': 2,
    'UYU': 2,
    'UYW': 4,
    'UZS': 2,
    'VAL': 0,
    'VEB': 2,
    'VED': 2,
    'VEF': 2,
    'VES': 2,
    'VND': 0,
    'VUV': 0,
    'WST': 2,
    'XAF': 0,
    'XCD': 2,
    'XEU': 0,
    'XOF': 0,
    'XPF': 0,
    'YER': 2,
    'YUD': 2,
    'YUG': 2,
    'YUM': 2,
    'YUN': 2,
    'YUO': 2,
    'YUR': 2,
    'ZAL': 2,
    'ZAR': 2,
    'ZMK': 2,
    'ZMW': 2,
    'ZRN': 2,
    'ZRZ': 2,
    'ZWB': 2,
    'ZWC': 2,
    'ZWD': 2,
    'ZWL': 2,
    'ZWN': 2,
    'ZWR': 2
}

```

## File: utils.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from hashlib import sha1

from odoo import fields
from odoo.http import request
from odoo.tools import consteq, float_round, ustr
from odoo.tools.misc import hmac as hmac_tool

from odoo.addons.payment.const import CURRENCY_MINOR_UNITS


# Access token management

def generate_access_token(*values):
    """ Generate an access token based on the provided values.

    The token allows to later verify the validity of a request, based on a given set of values.
    These will generally include the partner id, amount, currency id, transaction id or transaction
    reference.
    All values must be convertible to a string.

    :param list values: The values to use for the generation of the token
    :return: The generated access token
    :rtype: str
    """
    token_str = '|'.join(str(val) for val in values)
    access_token = hmac_tool(request.env(su=True), 'generate_access_token', token_str)
    return access_token


def check_access_token(access_token, *values):
    """ Check the validity of the access token for the provided values.

    The values must be provided in the exact same order as they were to `generate_access_token`.
    All values must be convertible to a string.

    :param str access_token: The access token used to verify the provided values
    :param list values: The values to verify against the token
    :return: True if the check is successful
    :rtype: bool
    """
    authentic_token = generate_access_token(*values)
    return access_token and consteq(ustr(access_token), authentic_token)


# Transaction values formatting

def singularize_reference_prefix(prefix='tx', separator='-', max_length=None):
    """ Make the prefix more unique by suffixing it with the current datetime.

    When the prefix is a placeholder that would be part of a large sequence of references sharing
    the same prefix, such as "tx" or "validation", singularizing it allows to make it part of a
    single-element sequence of transactions. The computation of the full reference will then execute
    faster by failing to find existing references with a matching prefix.

    If the `max_length` argument is passed, the end of the prefix can be stripped before
    singularizing to ensure that the result accounts for no more than `max_length` characters.

    Warning: Generated prefixes are *not* uniques! This function should be used only for making
    transaction reference prefixes more distinguishable and *not* for operations that require the
    generated value to be unique.

    :param str prefix: The custom prefix to singularize
    :param str separator: The custom separator used to separate the prefix from the suffix
    :param int max_length: The maximum length of the singularized prefix
    :return: The singularized prefix
    :rtype: str
    """
    if prefix is None:
        prefix = 'tx'
    if max_length:
        DATETIME_LENGTH = 14
        assert max_length >= 1 + len(separator) + DATETIME_LENGTH  # 1 char + separator + datetime
        prefix = prefix[:max_length-len(separator)-DATETIME_LENGTH]
    return f'{prefix}{separator}{fields.Datetime.now().strftime("%Y%m%d%H%M%S")}'


def to_major_currency_units(minor_amount, currency, arbitrary_decimal_number=None):
    """ Return the amount converted to the major units of its currency.

    The conversion is done by dividing the amount by 10^k where k is the number of decimals of the
    currency as per the ISO 4217 norm.
    To force a different number of decimals, set it as the value of the `arbitrary_decimal_number`
    argument.

    :param float minor_amount: The amount in minor units, to convert in major units
    :param recordset currency: The currency of the amount, as a `res.currency` record
    :param int arbitrary_decimal_number: The number of decimals to use instead of that of ISO 4217
    :return: The amount in major units of its currency
    :rtype: int
    """
    currency.ensure_one()

    if arbitrary_decimal_number is None:
        decimal_number = CURRENCY_MINOR_UNITS.get(currency.name, currency.decimal_places)
    else:
        decimal_number = arbitrary_decimal_number
    return float_round(minor_amount, precision_digits=0) / (10**decimal_number)


def to_minor_currency_units(major_amount, currency, arbitrary_decimal_number=None):
    """ Return the amount converted to the minor units of its currency.

    The conversion is done by multiplying the amount by 10^k where k is the number of decimals of
    the currency as per the ISO 4217 norm.
    To force a different number of decimals, set it as the value of the `arbitrary_decimal_number`
    argument.

    Note: currency.ensure_one() if arbitrary_decimal_number is not provided

    :param float major_amount: The amount in major units, to convert in minor units
    :param recordset currency: The currency of the amount, as a `res.currency` record
    :param int arbitrary_decimal_number: The number of decimals to use instead of that of ISO 4217
    :return: The amount in minor units of its currency
    :rtype: int
    """
    if arbitrary_decimal_number is not None:
        decimal_number = arbitrary_decimal_number
    else:
        currency.ensure_one()
        decimal_number = CURRENCY_MINOR_UNITS.get(currency.name, currency.decimal_places)
    return int(float_round(major_amount * (10**decimal_number), precision_digits=0))


# Token values formatting

def build_token_name(payment_details_short=None, final_length=16):
    """ Pad plain payment details with leading X's to build a token name of the desired length.

    :param str payment_details_short: The plain part of the payment details (usually last 4 digits)
    :param int final_length: The desired final length of the token name (16 for a bank card)
    :return: The padded token name
    :rtype: str
    """
    payment_details_short = payment_details_short or '????'
    return f"{'X' * (final_length - len(payment_details_short))}{payment_details_short}"


# Partner values formatting

def format_partner_address(address1="", address2=""):
    """ Format a two-parts partner address into a one-line address string.

    :param str address1: The first part of the address, usually the `street1` field
    :param str address2: The second part of the address, usually the `street2` field
    :return: The formatted one-line address
    :rtype: str
    """
    address1 = address1 or ""  # Avoid casting as "False"
    address2 = address2 or ""  # Avoid casting as "False"
    return f"{address1} {address2}".strip()


def split_partner_name(partner_name):
    """ Split a single-line partner name in a tuple of first name, last name.

    :param str partner_name: The partner name
    :return: The splitted first name and last name
    :rtype: tuple
    """
    return " ".join(partner_name.split()[:-1]), partner_name.split()[-1]


# Security

def get_customer_ip_address():
    return request and request.httprequest.remote_addr or ''


def check_rights_on_recordset(recordset):
    """ Ensure that the user has the rights to write on the record.

    Call this method to check the access rules and rights before doing any operation that is
    callable by RPC and that requires to be executed in sudo mode.

    :param recordset: The recordset for which the rights should be checked.
    :return: None
    """
    recordset.check_access_rights('write')
    recordset.check_access_rule('write')


# Idempotency

def generate_idempotency_key(tx, scope=None):
    """ Generate an idempotency key for the provided transaction and scope.

    Idempotency keys are used to prevent API requests from going through twice in a short time: the
    API rejects requests made after another one with the same payload and idempotency key if it
    succeeded.

    The idempotency key is generated based on the transaction reference, database UUID, and scope if
    any. This guarantees the key is identical for two API requests with the same transaction
    reference, database, and endpoint. Should one of these parameters differ, the key is unique from
    one request to another (e.g., after dropping the database, for different endpoints, etc.).

    :param recordset tx: The transaction to generate an idempotency key for, as a
                         `payment.transaction` record.
    :param str scope: The scope of the API request to generate an idempotency key for. This should
                      typically be the API endpoint. It is not necessary to provide the scope if the
                      API takes care of comparing idempotency keys per endpoint.
    :return: The generated idempotency key.
    :rtype: str
    """
    database_uuid = tx.env['ir.config_parameter'].sudo().get_param('database.uuid')
    return sha1(f'{database_uuid}{tx.reference}{scope or ""}'.encode()).hexdigest()

```

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models
from . import utils
from . import wizards

from odoo import api, SUPERUSER_ID


def reset_payment_acquirer(cr, registry, provider):
    env = api.Environment(cr, SUPERUSER_ID, {})
    acquirers = env['payment.acquirer'].search([('provider', '=', provider)])
    acquirers.write({
        'provider': 'none',
        'state': 'disabled',
    })

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Payment Acquirer',
    'version': '2.0',
    'category': 'Hidden',
    'summary': 'Base Module for Payment Acquirers',
    'description': """Payment Acquirer Base Module""",
    'depends': ['account'],
    'data': [
        'data/payment_icon_data.xml',
        'data/payment_acquirer_data.xml',
        'data/payment_cron.xml',

        'views/payment_portal_templates.xml',
        'views/payment_templates.xml',

        'views/account_invoice_views.xml',
        'views/account_journal_views.xml',
        'views/account_payment_views.xml',
        'views/payment_acquirer_views.xml',
        'views/payment_icon_views.xml',
        'views/payment_transaction_views.xml',
        'views/payment_token_views.xml',  # Depends on `action_payment_transaction_linked_to_token`
        'views/res_partner_views.xml',

        'security/ir.model.access.csv',
        'security/payment_security.xml',

        'wizards/account_payment_register_views.xml',
        'wizards/payment_acquirer_onboarding_templates.xml',
        'wizards/payment_link_wizard_views.xml',
        'wizards/payment_refund_wizard_views.xml',
    ],
    'auto_install': True,
    'assets': {
        'web.assets_frontend': [
            'payment/static/src/scss/portal_payment.scss',
            'payment/static/src/scss/payment_form.scss',
            'payment/static/lib/jquery.payment/jquery.payment.js',
            'payment/static/src/js/checkout_form.js',
            'payment/static/src/js/manage_form.js',
            'payment/static/src/js/payment_form_mixin.js',
            'payment/static/src/js/post_processing.js',
        ],
        'web.assets_backend': [
            'payment/static/src/scss/payment_acquirer.scss',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\portal.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import urllib.parse
import werkzeug

from odoo import _, http
from odoo.exceptions import AccessError, UserError, ValidationError
from odoo.fields import Command
from odoo.http import request

from odoo.addons.payment import utils as payment_utils
from odoo.addons.payment.controllers.post_processing import PaymentPostProcessing
from odoo.addons.portal.controllers import portal


class PaymentPortal(portal.CustomerPortal):

    """ This controller contains the foundations for online payments through the portal.

    It allows to complete a full payment flow without the need of going though a document-based flow
    made available by another module's controller.

    Such controllers should extend this one to gain access to the _create_transaction static method
    that implements the creation of a transaction before its processing, or to override specific
    routes and change their behavior globally (e.g. make the /pay route handle sale orders).

    The following routes are exposed:
    - `/payment/pay` allows for arbitrary payments.
    - `/my/payment_method` allows the user to create and delete tokens. It's its own `landing_route`
    - `/payment/transaction` is the `transaction_route` for the standard payment flow. It creates a
      draft transaction, and return the processing values necessary for the completion of the
      transaction.
    - `/payment/confirmation` is the `landing_route` for the standard payment flow. It displays the
      payment confirmation page to the user when the transaction is validated.
    """

    @http.route(
        '/payment/pay', type='http', methods=['GET'], auth='public', website=True, sitemap=False,
    )
    def payment_pay(
        self, reference=None, amount=None, currency_id=None, partner_id=None, company_id=None,
        acquirer_id=None, access_token=None, invoice_id=None, **kwargs
    ):
        """ Display the payment form with optional filtering of payment options.

        The filtering takes place on the basis of provided parameters, if any. If a parameter is
        incorrect or malformed, it is skipped to avoid preventing the user from making the payment.

        In addition to the desired filtering, a second one ensures that none of the following
        rules is broken:
            - Public users are not allowed to save their payment method as a token.
            - Payments made by public users should either *not* be made on behalf of a specific
              partner or have an access token validating the partner, amount and currency.
        We let access rights and security rules do their job for logged in users.

        :param str reference: The custom prefix to compute the full reference
        :param str amount: The amount to pay
        :param str currency_id: The desired currency, as a `res.currency` id
        :param str partner_id: The partner making the payment, as a `res.partner` id
        :param str company_id: The related company, as a `res.company` id
        :param str acquirer_id: The desired acquirer, as a `payment.acquirer` id
        :param str access_token: The access token used to authenticate the partner
        :param str invoice_id: The account move for which a payment id made, as a `account.move` id
        :param dict kwargs: Optional data. This parameter is not used here
        :return: The rendered checkout form
        :rtype: str
        :raise: werkzeug.exceptions.NotFound if the access token is invalid
        """
        # Cast numeric parameters as int or float and void them if their str value is malformed
        currency_id, acquirer_id, partner_id, company_id, invoice_id = tuple(map(
            self._cast_as_int, (currency_id, acquirer_id, partner_id, company_id, invoice_id)
        ))
        amount = self._cast_as_float(amount)

        # Raise an HTTP 404 if a partner is provided with an invalid access token
        if partner_id:
            if not payment_utils.check_access_token(access_token, partner_id, amount, currency_id):
                raise werkzeug.exceptions.NotFound  # Don't leak info about the existence of an id

        user_sudo = request.env.user
        logged_in = not user_sudo._is_public()
        # If the user is logged in, take their partner rather than the partner set in the params.
        # This is something that we want, since security rules are based on the partner, and created
        # tokens should not be assigned to the public user. This should have no impact on the
        # transaction itself besides making reconciliation possibly more difficult (e.g. The
        # transaction and invoice partners are different).
        partner_is_different = False
        if logged_in:
            partner_is_different = partner_id and partner_id != user_sudo.partner_id.id
            partner_sudo = user_sudo.partner_id
        else:
            partner_sudo = request.env['res.partner'].sudo().browse(partner_id).exists()
            if not partner_sudo:
                return request.redirect(
                    # Escape special characters to avoid loosing original params when redirected
                    f'/web/login?redirect={urllib.parse.quote(request.httprequest.full_path)}'
                )

        # Instantiate transaction values to their default if not set in parameters
        reference = reference or payment_utils.singularize_reference_prefix(prefix='tx')
        amount = amount or 0.0  # If the amount is invalid, set it to 0 to stop the payment flow
        company_id = company_id or partner_sudo.company_id.id or user_sudo.company_id.id
        company = request.env['res.company'].sudo().browse(company_id)
        currency_id = currency_id or company.currency_id.id

        # Make sure that the currency exists and is active
        currency = request.env['res.currency'].browse(currency_id).exists()
        if not currency or not currency.active:
            raise werkzeug.exceptions.NotFound  # The currency must exist and be active

        # Select all acquirers and tokens that match the constraints
        acquirers_sudo = request.env['payment.acquirer'].sudo()._get_compatible_acquirers(
            company_id, partner_sudo.id, currency_id=currency.id, **kwargs
        )  # In sudo mode to read the fields of acquirers and partner (if not logged in)
        if acquirer_id in acquirers_sudo.ids:  # Only keep the desired acquirer if it's suitable
            acquirers_sudo = acquirers_sudo.browse(acquirer_id)
        payment_tokens = request.env['payment.token'].search(
            [('acquirer_id', 'in', acquirers_sudo.ids), ('partner_id', '=', partner_sudo.id)]
        ) if logged_in else request.env['payment.token']

        # Make sure that the partner's company matches the company passed as parameter.
        if not PaymentPortal._can_partner_pay_in_company(partner_sudo, company):
            acquirers_sudo = request.env['payment.acquirer'].sudo()
            payment_tokens = request.env['payment.token']

        # Compute the fees taken by acquirers supporting the feature
        fees_by_acquirer = {
            acq_sudo: acq_sudo._compute_fees(amount, currency, partner_sudo.country_id)
            for acq_sudo in acquirers_sudo.filtered('fees_active')
        }

        # Generate a new access token in case the partner id or the currency id was updated
        access_token = payment_utils.generate_access_token(partner_sudo.id, amount, currency.id)

        rendering_context = {
            'acquirers': acquirers_sudo,
            'tokens': payment_tokens,
            'fees_by_acquirer': fees_by_acquirer,
            'show_tokenize_input': logged_in,  # Prevent public partner from saving payment methods
            'reference_prefix': reference,
            'amount': amount,
            'currency': currency,
            'partner_id': partner_sudo.id,
            'access_token': access_token,
            'transaction_route': '/payment/transaction',
            'landing_route': '/payment/confirmation',
            'res_company': company,  # Display the correct logo in a multi-company environment
            'partner_is_different': partner_is_different,
            'invoice_id': invoice_id,
            **self._get_custom_rendering_context_values(**kwargs),
        }
        return request.render(self._get_payment_page_template_xmlid(**kwargs), rendering_context)

    def _get_payment_page_template_xmlid(self, **kwargs):
        return 'payment.pay'

    @http.route('/my/payment_method', type='http', methods=['GET'], auth='user', website=True)
    def payment_method(self, **kwargs):
        """ Display the form to manage payment methods.

        :param dict kwargs: Optional data. This parameter is not used here
        :return: The rendered manage form
        :rtype: str
        """
        partner = request.env.user.partner_id
        acquirers_sudo = request.env['payment.acquirer'].sudo()._get_compatible_acquirers(
            request.env.company.id, partner.id, force_tokenization=True, is_validation=True
        )
        tokens = set(partner.payment_token_ids).union(
            partner.commercial_partner_id.sudo().payment_token_ids
        )  # Show all partner's tokens, regardless of which acquirer is available
        access_token = payment_utils.generate_access_token(partner.id, None, None)
        rendering_context = {
            'acquirers': acquirers_sudo,
            'tokens': tokens,
            'reference_prefix': payment_utils.singularize_reference_prefix(prefix='validation'),
            'partner_id': partner.id,
            'access_token': access_token,
            'transaction_route': '/payment/transaction',
            'landing_route': '/my/payment_method',
            **self._get_custom_rendering_context_values(**kwargs),
        }
        return request.render('payment.payment_methods', rendering_context)

    def _get_custom_rendering_context_values(self, **kwargs):
        """ Return a dict of additional rendering context values.

        :param dict kwargs: Optional data. This parameter is not used here
        :return: The dict of additional rendering context values
        :rtype: dict
        """
        return {}

    @http.route('/payment/transaction', type='json', auth='public')
    def payment_transaction(self, amount, currency_id, partner_id, access_token, **kwargs):
        """ Create a draft transaction and return its processing values.

        :param float|None amount: The amount to pay in the given currency.
                                  None if in a payment method validation operation
        :param int|None currency_id: The currency of the transaction, as a `res.currency` id.
                                     None if in a payment method validation operation
        :param int partner_id: The partner making the payment, as a `res.partner` id
        :param str access_token: The access token used to authenticate the partner
        :param dict kwargs: Locally unused data passed to `_create_transaction`
        :return: The mandatory values for the processing of the transaction
        :rtype: dict
        :raise: ValidationError if the access token is invalid
        """
        # Check the access token against the transaction values
        amount = amount and float(amount)  # Cast as float in case the JS stripped the '.0'
        if not payment_utils.check_access_token(access_token, partner_id, amount, currency_id):
            raise ValidationError(_("The access token is invalid."))

        kwargs.pop('custom_create_values', None)  # Don't allow passing arbitrary create values
        tx_sudo = self._create_transaction(
            amount=amount, currency_id=currency_id, partner_id=partner_id, **kwargs
        )
        self._update_landing_route(tx_sudo, access_token)  # Add the required parameters to the route
        return tx_sudo._get_processing_values()

    def _create_transaction(
        self, payment_option_id, reference_prefix, amount, currency_id, partner_id, flow,
        tokenization_requested, landing_route, is_validation=False, invoice_id=None,
        custom_create_values=None, **kwargs
    ):
        """ Create a draft transaction based on the payment context and return it.

        :param int payment_option_id: The payment option handling the transaction, as a
                                      `payment.acquirer` id or a `payment.token` id
        :param str reference_prefix: The custom prefix to compute the full reference
        :param float|None amount: The amount to pay in the given currency.
                                  None if in a payment method validation operation
        :param int|None currency_id: The currency of the transaction, as a `res.currency` id.
                                     None if in a payment method validation operation
        :param int partner_id: The partner making the payment, as a `res.partner` id
        :param str flow: The online payment flow of the transaction: 'redirect', 'direct' or 'token'
        :param bool tokenization_requested: Whether the user requested that a token is created
        :param str landing_route: The route the user is redirected to after the transaction
        :param bool is_validation: Whether the operation is a validation
        :param int invoice_id: The account move for which a payment id made, as an `account.move` id
        :param dict custom_create_values: Additional create values overwriting the default ones
        :param dict kwargs: Locally unused data passed to `_is_tokenization_required` and
                            `_compute_reference`
        :return: The sudoed transaction that was created
        :rtype: recordset of `payment.transaction`
        :raise: UserError if the flow is invalid
        """
        # Prepare create values
        if flow in ['redirect', 'direct']:  # Direct payment or payment with redirection
            acquirer_sudo = request.env['payment.acquirer'].sudo().browse(payment_option_id)
            token_id = None
            tokenization_required_or_requested = acquirer_sudo._is_tokenization_required(
                provider=acquirer_sudo.provider, **kwargs
            ) or tokenization_requested
            tokenize = bool(
                # Don't tokenize if the user tried to force it through the browser's developer tools
                acquirer_sudo.allow_tokenization
                # Token is only created if required by the flow or requested by the user
                and tokenization_required_or_requested
            )
        elif flow == 'token':  # Payment by token
            token_sudo = request.env['payment.token'].sudo().browse(payment_option_id)

            # Prevent from paying with a token that doesn't belong to the current partner (either
            # the current user's partner if logged in, or the partner on behalf of whom the payment
            # is being made).
            partner_sudo = request.env['res.partner'].sudo().browse(partner_id)
            if partner_sudo.commercial_partner_id != token_sudo.partner_id.commercial_partner_id:
                raise AccessError(_("You do not have access to this payment token."))

            acquirer_sudo = token_sudo.acquirer_id
            token_id = payment_option_id
            tokenize = False
        else:
            raise UserError(
                _("The payment should either be direct, with redirection, or made by a token.")
            )

        if invoice_id:
            if custom_create_values is None:
                custom_create_values = {}
            custom_create_values['invoice_ids'] = [Command.set([int(invoice_id)])]

        reference = request.env['payment.transaction']._compute_reference(
            acquirer_sudo.provider,
            prefix=reference_prefix,
            **(custom_create_values or {}),
            **kwargs
        )
        if is_validation:  # Acquirers determine the amount and currency in validation operations
            amount = acquirer_sudo._get_validation_amount()
            currency_id = acquirer_sudo._get_validation_currency().id

        # Create the transaction
        tx_sudo = request.env['payment.transaction'].sudo().create({
            'acquirer_id': acquirer_sudo.id,
            'reference': reference,
            'amount': amount,
            'currency_id': currency_id,
            'partner_id': partner_id,
            'token_id': token_id,
            'operation': f'online_{flow}' if not is_validation else 'validation',
            'tokenize': tokenize,
            'landing_route': landing_route,
            **(custom_create_values or {}),
        })  # In sudo mode to allow writing on callback fields

        if flow == 'token':
            tx_sudo._send_payment_request()  # Payments by token process transactions immediately
        else:
            tx_sudo._log_sent_message()

        # Monitor the transaction to make it available in the portal
        PaymentPostProcessing.monitor_transactions(tx_sudo)

        return tx_sudo

    @staticmethod
    def _update_landing_route(tx_sudo, access_token):
        """ Add the mandatory parameters to the route and recompute the access token if needed.

        The generic landing route requires the tx id and access token to be provided since there is
        no document to rely on. The access token is recomputed in case we are dealing with a
        validation transaction (acquirer-specific amount and currency).

        :param recordset tx_sudo: The transaction whose landing routes to update, as a
                                  `payment.transaction` record.
        :param str access_token: The access token used to authenticate the partner
        :return: None
        """
        if tx_sudo.operation == 'validation':
            access_token = payment_utils.generate_access_token(
                tx_sudo.partner_id.id, tx_sudo.amount, tx_sudo.currency_id.id
            )
        tx_sudo.landing_route = f'{tx_sudo.landing_route}' \
                                f'?tx_id={tx_sudo.id}&access_token={access_token}'

    @http.route('/payment/confirmation', type='http', methods=['GET'], auth='public', website=True)
    def payment_confirm(self, tx_id, access_token, **kwargs):
        """ Display the payment confirmation page with the appropriate status message to the user.

        :param str tx_id: The transaction to confirm, as a `payment.transaction` id
        :param str access_token: The access token used to verify the user
        :param dict kwargs: Optional data. This parameter is not used here
        :raise: werkzeug.exceptions.NotFound if the access token is invalid
        """
        tx_id = self._cast_as_int(tx_id)
        if tx_id:
            tx_sudo = request.env['payment.transaction'].sudo().browse(tx_id)

            # Raise an HTTP 404 if the access token is invalid
            if not payment_utils.check_access_token(
                access_token, tx_sudo.partner_id.id, tx_sudo.amount, tx_sudo.currency_id.id
            ):
                raise werkzeug.exceptions.NotFound  # Don't leak info about existence of an id

            # Fetch the appropriate status message configured on the acquirer
            if tx_sudo.state == 'draft':
                status = 'info'
                message = tx_sudo.state_message \
                          or _("This payment has not been processed yet.")
            elif tx_sudo.state == 'pending':
                status = 'warning'
                message = tx_sudo.acquirer_id.pending_msg
            elif tx_sudo.state in ('authorized', 'done'):
                status = 'success'
                message = tx_sudo.acquirer_id.done_msg
            elif tx_sudo.state == 'cancel':
                status = 'danger'
                message = tx_sudo.acquirer_id.cancel_msg
            else:
                status = 'danger'
                message = tx_sudo.state_message \
                          or _("An error occurred during the processing of this payment.")

            # Display the payment confirmation page to the user
            PaymentPostProcessing.remove_transactions(tx_sudo)
            render_values = {
                'tx': tx_sudo,
                'status': status,
                'message': message
            }
            return request.render('payment.confirm', render_values)
        else:
            # Display the portal homepage to the user
            return request.redirect('/my/home')

    @http.route('/payment/archive_token', type='json', auth='user')
    def archive_token(self, token_id):
        """ Check that a user has write access on a token and archive the token if so.

        :param int token_id: The token to archive, as a `payment.token` id
        :return: None
        """
        partner_sudo = request.env.user.partner_id
        token_sudo = request.env['payment.token'].sudo().search([
            ('id', '=', token_id),
            # Check that the user owns the token before letting them archive anything
            ('partner_id', 'in', [partner_sudo.id, partner_sudo.commercial_partner_id.id])
        ])
        if token_sudo:
            token_sudo.active = False

    @staticmethod
    def _cast_as_int(str_value):
        """ Cast a string as an `int` and return it.

        If the conversion fails, `None` is returned instead.

        :param str str_value: The value to cast as an `int`
        :return: The casted value, possibly replaced by None if incompatible
        :rtype: int|None
        """
        try:
            return int(str_value)
        except (TypeError, ValueError, OverflowError):
            return None

    @staticmethod
    def _cast_as_float(str_value):
        """ Cast a string as a `float` and return it.

        If the conversion fails, `None` is returned instead.

        :param str str_value: The value to cast as a `float`
        :return: The casted value, possibly replaced by None if incompatible
        :rtype: float|None
        """
        try:
            return float(str_value)
        except (TypeError, ValueError, OverflowError):
            return None

    @staticmethod
    def _can_partner_pay_in_company(partner, document_company):
        """ Return whether the provided partner can pay in the provided company.

        The payment is allowed either if the partner's company is not set or if the companies match.

        :param recordset partner: The partner on behalf on which the payment is made, as a
                                  `res.partner` record.
        :param recordset document_company: The company of the document being paid, as a
                                           `res.company` record.
        :return: Whether the payment is allowed.
        :rtype: str
        """
        return not partner.company_id or partner.company_id == document_company

```

## File: controllers\post_processing.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
from datetime import timedelta

import psycopg2

from odoo import fields, http
from odoo.http import request

_logger = logging.getLogger(__name__)


class PaymentPostProcessing(http.Controller):

    """
    This controller is responsible for the monitoring and finalization of the post-processing of
    transactions.

    It exposes the route `/payment/status`: All payment flows must go through this route at some
    point to allow the user checking on the transactions' status, and to trigger the finalization of
    their post-processing.
    """

    MONITORED_TX_IDS_KEY = '__payment_monitored_tx_ids__'

    @http.route('/payment/status', type='http', auth='public', website=True, sitemap=False)
    def display_status(self, **kwargs):
        """ Display the payment status page.

        :param dict kwargs: Optional data. This parameter is not used here
        :return: The rendered status page
        :rtype: str
        """
        return request.render('payment.payment_status')

    @http.route('/payment/status/poll', type='json', auth='public')
    def poll_status(self):
        """ Fetch the transactions to display on the status page and finalize their post-processing.

        :return: The post-processing values of the transactions
        :rtype: dict
        """
        # Retrieve recent user's transactions from the session
        limit_date = fields.Datetime.now() - timedelta(days=1)
        monitored_txs = request.env['payment.transaction'].sudo().search([
            ('id', 'in', self.get_monitored_transaction_ids()),
            ('last_state_change', '>=', limit_date)
        ])
        if not monitored_txs:  # The transaction was not correctly created
            return {
                'success': False,
                'error': 'no_tx_found',
            }

        # Build the list of display values with the display message and post-processing values
        display_values_list = []
        for tx in monitored_txs:
            display_message = None
            if tx.state == 'pending':
                display_message = tx.acquirer_id.pending_msg
            elif tx.state == 'done':
                display_message = tx.acquirer_id.done_msg
            elif tx.state == 'cancel':
                display_message = tx.acquirer_id.cancel_msg
            display_values_list.append({
                'display_message': display_message,
                **tx._get_post_processing_values(),
            })

        # Stop monitoring already post-processed transactions
        post_processed_txs = monitored_txs.filtered('is_post_processed')
        self.remove_transactions(post_processed_txs)

        # Finalize post-processing of transactions before displaying them to the user
        txs_to_post_process = (monitored_txs - post_processed_txs).filtered(
            lambda t: t.state == 'done'
        )
        success, error = True, None
        try:
            txs_to_post_process._finalize_post_processing()
        except psycopg2.OperationalError:  # A collision of accounting sequences occurred
            request.env.cr.rollback()  # Rollback and try later
            success = False
            error = 'tx_process_retry'
        except Exception as e:
            request.env.cr.rollback()
            success = False
            error = str(e)
            _logger.exception(
                "encountered an error while post-processing transactions with ids %s:\n%s",
                ', '.join([str(tx_id) for tx_id in txs_to_post_process.ids]), e
            )

        return {
            'success': success,
            'error': error,
            'display_values_list': display_values_list,
        }

    @classmethod
    def monitor_transactions(cls, transactions):
        """ Add the ids of the provided transactions to the list of monitored transaction ids.

        :param recordset transactions: The transactions to monitor, as a `payment.transaction`
                                       recordset
        :return: None
        """
        if transactions:
            monitored_tx_ids = request.session.get(cls.MONITORED_TX_IDS_KEY, [])
            request.session[cls.MONITORED_TX_IDS_KEY] = list(
                set(monitored_tx_ids).union(transactions.ids)
            )

    @classmethod
    def get_monitored_transaction_ids(cls):
        """ Return the ids of transactions being monitored.

        Only the ids and not the recordset itself is returned to allow the caller browsing the
        recordset with sudo privileges, and using the ids in a custom query.

        :return: The ids of transactions being monitored
        :rtype: list
        """
        return request.session.get(cls.MONITORED_TX_IDS_KEY, [])

    @classmethod
    def remove_transactions(cls, transactions):
        """ Remove the ids of the provided transactions from the list of monitored transaction ids.

        :param recordset transactions: The transactions to remove, as a `payment.transaction`
                                       recordset
        :return: None
        """
        if transactions:
            monitored_tx_ids = request.session.get(cls.MONITORED_TX_IDS_KEY, [])
            request.session[cls.MONITORED_TX_IDS_KEY] = [
                tx_id for tx_id in monitored_tx_ids if tx_id not in transactions.ids
            ]

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import portal

```

## File: data\payment_acquirer_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="payment_acquirer_adyen" model="payment.acquirer">
        <field name="name">Adyen</field>
        <field name="display_as">Credit Card (powered by Adyen)</field>
        <field name="image_128" type="base64" file="payment_adyen/static/src/img/adyen_icon.png"/>
        <field name="module_id" ref="base.module_payment_adyen"/>
        <field name="description" type="html">
            <p>
                A payment gateway to accept online payments via credit cards, debit cards and bank
                transfers.
            </p>
            <ul class="list-inline">
                <li class="list-inline-item"><i class="fa fa-check"/>Online Payment</li>
                <li class="list-inline-item"><i class="fa fa-check"/>Payment Status Tracking</li>
            </ul>
        </field>
        <!-- https://www.adyen.com/payment-methods -->
        <field name="payment_icon_ids"
               eval="[(6, 0, [
                   ref('payment.payment_icon_cc_bancontact'),
                   ref('payment.payment_icon_cc_maestro'),
                   ref('payment.payment_icon_cc_mastercard'),
                   ref('payment.payment_icon_cc_visa'),
                   ref('payment.payment_icon_cc_discover'),
                   ref('payment.payment_icon_cc_diners_club_intl'),
                   ref('payment.payment_icon_cc_jcb'),
                   ref('payment.payment_icon_cc_unionpay'),
               ])]"/>
    </record>

    <record id="payment_acquirer_alipay" model="payment.acquirer">
        <field name="name">Alipay</field>
        <field name="display_as">Credit Card (powered by Alipay)</field>
        <field name="image_128" type="base64" file="payment_alipay/static/description/icon.png"/>
        <field name="module_id" ref="base.module_payment_alipay"/>
        <field name="description" type="html">
            <p>
                Alipay is the most popular online payment platform in China. Chinese consumers can
                buy online using their Alipay eWallet.
            </p>
            <ul class="list-inline">
                <li><i class="fa fa-check"/>Online Payment</li>
                <li><i class="fa fa-check"/>Payment Status Tracking</li>
            </ul>
        </field>
        <!-- https://intl.alipay.com/ihome/home/about/buy.htm?topic=paymentMethods -->
        <field name="payment_icon_ids"
               eval="[(6, 0, [
                   ref('payment.payment_icon_cc_jcb'),
                   ref('payment.payment_icon_cc_mastercard'),
                   ref('payment.payment_icon_cc_western_union'),
                   ref('payment.payment_icon_cc_webmoney'),
                   ref('payment.payment_icon_cc_visa'),
               ])]"/>
    </record>

    <record id="payment_acquirer_authorize" model="payment.acquirer">
        <field name="name">Authorize.net</field>
        <field name="display_as">Credit Card (powered by Authorize)</field>
        <field name="image_128"
               type="base64"
               file="payment_authorize/static/src/img/authorize_icon.png"/>
        <field name="module_id" ref="base.module_payment_authorize"/>
        <field name="description" type="html">
            <p>
                A payment gateway to accept online payments via credit cards and e-checks.
            </p>
            <ul class="list-inline">
                <li class="list-inline-item"><i class="fa fa-check"/>Online Payment</li>
                <li class="list-inline-item"><i class="fa fa-check"/>Payment Status Tracking</li>
                <li class="list-inline-item"><i class="fa fa-check"/>Subscriptions</li>
                <li class="list-inline-item"><i class="fa fa-check"/>Save Cards</li>
                <li class="list-inline-item"><i class="fa fa-check"/>Manual Capture</li>
                <li class="list-inline-item"><i class="fa fa-check"/>Embedded Credit Card Form</li>
            </ul>
        </field>
        <!-- https://www.authorize.net/solutions/merchantsolutions/onlinemerchantaccount/ -->
        <field name="payment_icon_ids"
               eval="[(6, 0, [
                   ref('payment.payment_icon_cc_maestro'),
                   ref('payment.payment_icon_cc_mastercard'),
                   ref('payment.payment_icon_cc_discover'),
                   ref('payment.payment_icon_cc_diners_club_intl'),
                   ref('payment.payment_icon_cc_jcb'),
                   ref('payment.payment_icon_cc_visa'),
               ])]"/>
    </record>

    <record id="payment_acquirer_buckaroo" model="payment.acquirer">
        <field name="name">Buckaroo</field>
        <field name="display_as">Credit Card (powered by Buckaroo)</field>
        <field name="image_128"
               type="base64"
               file="payment_buckaroo/static/src/img/buckaroo_icon.png"/>
        <field name="module_id" ref="base.module_payment_buckaroo"/>
        <field name="description" type="html">
            <p>
                A payment gateway to accept online payments via credit cards.
            </p>
            <ul class="list-inline">
                <li class="list-inline-item"><i class="fa fa-check"/>Online Payment</li>
                <li class="list-inline-item"><i class="fa fa-check"/>Payment Status Tracking</li>
            </ul>
        </field>
        <!-- https://www.buckaroo-payments.com/products/payment-methods/ -->
        <field name="payment_icon_ids"
               eval="[(6, 0, [
                   ref('payment.payment_icon_cc_bancontact'),
                   ref('payment.payment_icon_cc_maestro'),
                   ref('payment.payment_icon_cc_mastercard'),
                   ref('payment.payment_icon_cc_visa'),
                   ref('payment.payment_icon_cc_american_express'),
               ])]"/>
    </record>

    <record id="payment_acquirer_mollie" model="payment.acquirer">
        <field name="name">Mollie</field>
        <field name="image_128" type="base64" file="payment_mollie/static/src/img/mollie_icon.png"/>
        <field name="module_id" ref="base.module_payment_mollie"/>
        <field name="description" type="html">
            <p>
                A payment gateway from Mollie to accept online payments.
            </p>
            <ul class="list-inline">
                <li class="list-inline-item"><i class="fa fa-check"/>Online Payment</li>
                <li class="list-inline-item"><i class="fa fa-check"/>Payment Status Tracking</li>
            </ul>
        </field>

        <!-- https://www.mollie.com/en/payments -->
        <field name="payment_icon_ids" eval="[(6, 0, [
                ref('payment.payment_icon_cc_visa'),
                ref('payment.payment_icon_cc_american_express'),
                ref('payment.payment_icon_cc_maestro'),
                ref('payment.payment_icon_cc_mastercard'),
                ref('payment.payment_icon_cc_bancontact'),
                ref('payment.payment_icon_cc_eps'),
                ref('payment.payment_icon_cc_giropay'),
                ref('payment.payment_icon_cc_p24'),
                ref('payment.payment_icon_cc_ideal'),
                ref('payment.payment_icon_paypal'),
                ref('payment.payment_icon_apple_pay'),
                ref('payment.payment_icon_sepa'),
                ref('payment.payment_icon_kbc')
            ])]"/>

    </record>

    <record id="payment_acquirer_ogone" model="payment.acquirer">
        <field name="name">Ogone</field>
        <field name="display_as">Credit Card (powered by Ogone)</field>
        <field name="image_128"
               type="base64"
               file="payment_ogone/static/src/img/ingenico_icon.png"/>
        <field name="module_id" ref="base.module_payment_ogone"/>
        <field name="description" type="html">
            <p>
                Ogone supports a wide range of payment methods: credit cards, debit cards, bank
                transfers, Bancontact, iDeal, Giropay.
            </p>
            <ul class="list-inline">
                <li class="list-inline-item"><i class="fa fa-check"/>Online Payment</li>
                <li class="list-inline-item"><i class="fa fa-check"/>Payment Status Tracking</li>
                <li class="list-inline-item"><i class="fa fa-check"/>Subscriptions</li>
                <li class="list-inline-item"><i class="fa fa-check"/>Save Cards</li>
                <li class="list-inline-item"><i class="fa fa-check"/>Embedded Credit Card Form</li>
            </ul>
        </field>
        <field name="payment_icon_ids"
               eval="[(6, 0, [
                   ref('payment.payment_icon_cc_ideal'),
                   ref('payment.payment_icon_cc_bancontact'),
                   ref('payment.payment_icon_cc_maestro'),
                   ref('payment.payment_icon_cc_mastercard'),
                   ref('payment.payment_icon_cc_visa'),
               ])]"/>
    </record>

    <record id="payment_acquirer_paypal" model="payment.acquirer">
        <field name="name">PayPal</field>
        <field name="image_128" type="base64" file="payment_paypal/static/src/img/paypal_icon.png"/>
        <field name="module_id" ref="base.module_payment_paypal"/>
        <field name="description" type="html">
            <p>
                PayPal is the easiest way to accept payments via Paypal or credit cards.
            </p>
            <ul class="list-inline">
                <li class="list-inline-item"><i class="fa fa-check"/>Online Payment</li>
                <li class="list-inline-item"><i class="fa fa-check"/>Payment Status Tracking</li>
            </ul>
        </field>
        <!-- https://www.paypal.com/us/selfhelp/article/Which-credit-cards-can-I-accept-with-PayPal-Merchant-Services-FAQ1525#business -->
        <field name="payment_icon_ids"
               eval="[(6, 0, [
                   ref('payment.payment_icon_cc_maestro'),
                   ref('payment.payment_icon_cc_mastercard'),
                   ref('payment.payment_icon_cc_discover'),
                   ref('payment.payment_icon_cc_diners_club_intl'),
                   ref('payment.payment_icon_cc_jcb'),
                   ref('payment.payment_icon_cc_american_express'),
                   ref('payment.payment_icon_cc_unionpay'),
                   ref('payment.payment_icon_cc_visa'),
               ])]"/>
    </record>

    <record id="payment_acquirer_payulatam" model="payment.acquirer">
        <field name="name">PayU Latam</field>
        <field name="display_as">Credit Card (powered by PayU Latam)</field>
        <field name="image_128"
               type="base64"
               file="payment_payulatam/static/src/img/payulatam_icon.png"/>
        <field name="module_id" ref="base.module_payment_payulatam"/>
        <field name="description" type="html">
            <p>
                PayU is a leading financial services provider in Colombia, Argentina, Brazil, Chile,
                Mexico, Panama, and Peru. It allows merchant to accept local payments with just one
                account and integration.
            </p>
            <ul class="list-inline">
                <li class="list-inline-item"><i class="fa fa-check"/>Online Payment</li>
                <li class="list-inline-item"><i class="fa fa-check"/>Payment Status Tracking</li>
            </ul>
        </field>
        <!-- https://www.payulatam.com/medios-de-pago/ -->
        <field name="payment_icon_ids"
               eval="[(6, 0, [
                   ref('payment.payment_icon_cc_diners_club_intl'),
                   ref('payment.payment_icon_cc_mastercard'),
                   ref('payment.payment_icon_cc_american_express'),
                   ref('payment.payment_icon_cc_visa'),
                   ref('payment.payment_icon_cc_codensa_easy_credit'),
               ])]"/>
    </record>

    <record id="payment_acquirer_payumoney" model="payment.acquirer">
        <field name="name">PayUmoney</field>
        <field name="display_as">Credit Card (powered by PayUmoney)</field>
        <field name="image_128"
               type="base64"
               file="payment_payumoney/static/src/img/payumoney_icon.png"/>
        <field name="module_id" ref="base.module_payment_payumoney"/>
        <field name="description" type="html">
            <p>
                PayUmoney is an online payments solutions company serving the Indian market.
            </p>
            <ul class="list-inline">
                <li class="list-inline-item"><i class="fa fa-check"/>Online Payment</li>
                <li class="list-inline-item"><i class="fa fa-check"/>Payment Status Tracking</li>
            </ul>
        </field>
        <!-- See https://www.payumoney.com/selfcare.html?userType=seller
             > Banks & Cards > What options do you have in the Credit Card payment? -->
        <field name="payment_icon_ids"
               eval="[(6, 0, [
                   ref('payment.payment_icon_cc_maestro'),
                   ref('payment.payment_icon_cc_mastercard'),
                   ref('payment.payment_icon_cc_american_express'),
                   ref('payment.payment_icon_cc_visa'),
               ])]"/>
    </record>

    <record id="payment_acquirer_sepa_direct_debit" model="payment.acquirer">
        <field name="name">SEPA Direct Debit</field>
        <field name="sequence">20</field>
        <field name="image_128"
               type="base64"
               file="base/static/img/icons/payment_sepa_direct_debit.png"/>
        <field name="module_id" ref="base.module_payment_sepa_direct_debit"/>
        <field name="description" type="html">
            <p>
                SEPA Direct Debit is a Europe-wide Direct Debit system that allows merchants to
                collect Euro-denominated payments from accounts in the 34 SEPA countries and
                associated territories.
            </p>
        </field>
    </record>

    <record id="payment_acquirer_sips" model="payment.acquirer">
        <field name="name">Sips</field>
        <field name="display_as">Credit Card (powered by Sips)</field>
        <field name="image_128" type="base64" file="payment_sips/static/src/img/sips_icon.png"/>
        <field name="module_id" ref="base.module_payment_sips"/>
        <field name="description" type="html">
            <p>
                A payment gateway from Atos Worldline to accept online payments via credit cards.
            </p>
            <ul class="list-inline">
                <li class="list-inline-item"><i class="fa fa-check"/>Online Payment</li>
                <li class="list-inline-item"><i class="fa fa-check"/>Payment Status Tracking</li>
            </ul>
        </field>
        <!-- See http://sips.worldline.com/en-us/home/features/payment-types-and-acquirers.html -->
        <field name="payment_icon_ids"
               eval="[(6, 0, [
                   ref('payment.payment_icon_cc_maestro'),
                   ref('payment.payment_icon_cc_mastercard'),
                   ref('payment.payment_icon_cc_discover'),
                   ref('payment.payment_icon_cc_diners_club_intl'),
                   ref('payment.payment_icon_cc_jcb'),
                   ref('payment.payment_icon_cc_american_express'),
                   ref('payment.payment_icon_cc_bancontact'),
                   ref('payment.payment_icon_cc_unionpay'),
                   ref('payment.payment_icon_cc_visa'),
               ])]"/>
    </record>

    <record id="payment_acquirer_stripe" model="payment.acquirer">
        <field name="name">Stripe</field>
        <field name="display_as">Credit &amp; Debit Card</field>
        <field name="image_128" type="base64" file="payment_stripe/static/src/img/stripe_icon.png"/>
        <field name="module_id" ref="base.module_payment_stripe"/>
        <field name="description" type="html">
            <p>
                A payment gateway to accept online payments via credit and debit cards.
            </p>
            <ul class="list-inline">
                <li class="list-inline-item"><i class="fa fa-check"/>Online Payment</li>
                <li class="list-inline-item"><i class="fa fa-check"/>Payment Status Tracking</li>
                <li class="list-inline-item"><i class="fa fa-check"/>Subscriptions</li>
                <li class="list-inline-item"><i class="fa fa-check"/>Save Cards</li>
            </ul>
        </field>
        <!--
            See https://stripe.com/payments/payment-methods-guide
            See https://support.goteamup.com/hc/en-us/articles/115002089349-Which-cards-and-payment-types-can-I-accept-with-Stripe-
        -->
        <field name="payment_icon_ids"
               eval="[(6, 0, [
                   ref('payment.payment_icon_cc_maestro'),
                   ref('payment.payment_icon_cc_mastercard'),
                   ref('payment.payment_icon_cc_discover'),
                   ref('payment.payment_icon_cc_diners_club_intl'),
                   ref('payment.payment_icon_cc_jcb'),
                   ref('payment.payment_icon_cc_american_express'),
                   ref('payment.payment_icon_cc_visa'),
               ])]"/>
    </record>

    <record id="payment_acquirer_test" model="payment.acquirer">
        <field name="name">Test</field>
        <field name="sequence">40</field>
        <field name="image_128" type="base64" file="payment_test/static/src/img/test_logo.jpg"/>
        <field name="module_id" ref="base.module_payment_test"/>
        <field name="description" type="html">
            <p>
                A testing payment gateway intended for demonstrating payment flows without the need
                of creating a seller account or providing payment details.
            </p>
        </field>
    </record>

    <record id="payment_acquirer_transfer" model="payment.acquirer">
        <field name="name">Wire Transfer</field>
        <field name="sequence">30</field>
        <field name="image_128"
               type="base64"
               file="payment_transfer/static/src/img/transfer_icon.png"/>
        <field name="module_id" ref="base.module_payment_transfer"/>
        <field name="description" type="html">
            <p>
                Provide instructions to customers so that they can pay their orders manually.
            </p>
        </field>
    </record>

</odoo>

```

## File: data\payment_cron.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record model="ir.cron" id="cron_post_process_payment_tx">
        <field name="name">payment: post-process transactions</field>
        <field name="model_id" ref="payment.model_payment_transaction" />
        <field name="state">code</field>
        <field name="code">model._cron_finalize_post_processing()</field>
        <field name="user_id" ref="base.user_root" />
        <field name="interval_number">10</field>
        <field name="interval_type">minutes</field>
        <field name="numbercall">-1</field>
    </record>

</odoo>

```

## File: data\payment_icon_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="payment_icon_cc_visa" model="payment.icon">
        <field name="sequence">10</field>
        <field name="name">VISA</field>
        <field name="image" type="base64" file="payment/static/img/visa.png"/>
    </record>

    <record id="payment_icon_cc_mastercard" model="payment.icon">
        <field name="sequence">20</field>
        <field name="name">MasterCard</field>
        <field name="image" type="base64" file="payment/static/img/mastercard.png"/>
    </record>

    <record id="payment_icon_cc_american_express" model="payment.icon">
        <field name="sequence">30</field>
        <field name="name">American Express</field>
        <field name="image" type="base64" file="payment/static/img/american_express.png"/>
    </record>

    <record id="payment_icon_cc_discover" model="payment.icon">
        <field name="sequence">40</field>
        <field name="name">Discover</field>
        <field name="image" type="base64" file="payment/static/img/discover.png"/>
    </record>

    <record id="payment_icon_cc_diners_club_intl" model="payment.icon">
        <field name="sequence">50</field>
        <field name="name">Diners Club International</field>
        <field name="image" type="base64" file="payment/static/img/diners_club_intl.png"/>
    </record>

    <record id="payment_icon_paypal" model="payment.icon">
        <field name="sequence">60</field>
        <field name="name">Paypal</field>
        <field name="image" type="base64" file="payment/static/img/paypal.png"/>
    </record>

    <record id="payment_icon_apple_pay" model="payment.icon">
        <field name="sequence">70</field>
        <field name="name">Apple Pay</field>
        <field name="image" type="base64" file="payment/static/img/applepay.png"/>
    </record>

    <record id="payment_icon_cc_jcb" model="payment.icon">
        <field name="sequence">80</field>
        <field name="name">JCB</field>
        <field name="image" type="base64" file="payment/static/img/jcb.png"/>
    </record>

    <record id="payment_icon_cc_maestro" model="payment.icon">
        <field name="sequence">90</field>
        <field name="name">Maestro</field>
        <field name="image" type="base64" file="payment/static/img/maestro.png"/>
    </record>

    <record id="payment_icon_cc_cirrus" model="payment.icon">
        <field name="sequence">100</field>
        <field name="name">Cirrus</field>
        <field name="image" type="base64" file="payment/static/img/cirrus.png"/>
    </record>

    <record id="payment_icon_cc_unionpay" model="payment.icon">
        <field name="sequence">110</field>
        <field name="name">UnionPay</field>
        <field name="image" type="base64" file="payment/static/img/unionpay.png"/>
    </record>

    <record id="payment_icon_cc_bancontact" model="payment.icon">
        <field name="sequence">120</field>
        <field name="name">Bancontact</field>
        <field name="image" type="base64" file="payment/static/img/bancontact.png"/>
    </record>

    <record id="payment_icon_cc_western_union" model="payment.icon">
        <field name="sequence">130</field>
        <field name="name">Western Union</field>
        <field name="image" type="base64" file="payment/static/img/western_union.png"/>
    </record>

    <record id="payment_icon_sepa" model="payment.icon">
        <field name="sequence">140</field>
        <field name="name">SEPA Direct Debit</field>
        <field name="image" type="base64" file="payment/static/img/sepa.png"/>
    </record>

    <record id="payment_icon_cc_ideal" model="payment.icon">
        <field name="sequence">150</field>
        <field name="name">iDEAL</field>
        <field name="image" type="base64" file="payment/static/img/ideal.png"/>
    </record>

    <record id="payment_icon_cc_webmoney" model="payment.icon">
        <field name="sequence">160</field>
        <field name="name">WebMoney</field>
        <field name="image" type="base64" file="payment/static/img/webmoney.png"/>
    </record>

    <record id="payment_icon_cc_giropay" model="payment.icon">
        <field name="sequence">170</field>
        <field name="name">Giropay</field>
        <field name="image" type="base64" file="payment/static/img/giropay.png"/>
    </record>

    <record id="payment_icon_cc_eps" model="payment.icon">
        <field name="sequence">180</field>
        <field name="name">EPS</field>
        <field name="image" type="base64" file="payment/static/img/eps.png"/>
    </record>

    <record id="payment_icon_cc_p24" model="payment.icon">
        <field name="sequence">190</field>
        <field name="name">P24</field>
        <field name="image" type="base64" file="payment/static/img/p24.png"/>
    </record>

    <record id="payment_icon_cc_codensa_easy_credit" model="payment.icon">
        <field name="sequence">200</field>
        <field name="name">Codensa Easy Credit</field>
        <field name="image" type="base64" file="payment/static/img/codensa_easy_credit.png"/>
    </record>

    <record id="payment_icon_kbc" model="payment.icon">
        <field name="sequence">210</field>
        <field name="name">KBC</field>
        <field name="image" type="base64" file="payment/static/img/kbc.png"/>
    </record>

</odoo>

```

## File: models\account_journal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, Command, models
from odoo.exceptions import UserError


class AccountJournal(models.Model):
    _inherit = "account.journal"

    def _get_available_payment_method_lines(self, payment_type):
        lines = super()._get_available_payment_method_lines(payment_type)

        return lines.filtered(lambda l: l.payment_acquirer_state != 'disabled')

    @api.ondelete(at_uninstall=False)
    def _unlink_except_linked_to_payment_acquirer(self):
        linked_acquirers = self.env['payment.acquirer'].sudo().search([]).filtered(
            lambda acq: acq.journal_id.id in self.ids and acq.state != 'disabled'
        )
        if linked_acquirers:
            raise UserError(_(
                "You must first deactivate a payment acquirer before deleting its journal.\n"
                "Linked acquirer(s): %s", ', '.join(acq.display_name for acq in linked_acquirers)
            ))

```

## File: models\account_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models

from odoo.addons.payment import utils as payment_utils


class AccountMove(models.Model):
    _inherit = 'account.move'

    transaction_ids = fields.Many2many(
        string="Transactions", comodel_name='payment.transaction',
        relation='account_invoice_transaction_rel', column1='invoice_id', column2='transaction_id',
        readonly=True, copy=False)
    authorized_transaction_ids = fields.Many2many(
        string="Authorized Transactions", comodel_name='payment.transaction',
        compute='_compute_authorized_transaction_ids', readonly=True, copy=False)
    amount_paid = fields.Monetary(
        string="Amount paid",
        help="The amount already paid for this invoice.",
        compute='_compute_amount_paid'
    )

    @api.depends('transaction_ids')
    def _compute_authorized_transaction_ids(self):
        for invoice in self:
            invoice.authorized_transaction_ids = invoice.transaction_ids.filtered(
                lambda tx: tx.state == 'authorized'
            )

    @api.depends('transaction_ids')
    def _compute_amount_paid(self):
        """ Sum all the transaction amount for which state is in 'authorized' or 'done'
        """
        for invoice in self:
            invoice.amount_paid = sum(
                invoice.transaction_ids.filtered(
                    lambda tx: tx.state in ('authorized', 'done')
                ).mapped('amount')
            )

    def get_portal_last_transaction(self):
        self.ensure_one()
        return self.with_context(active_test=False).transaction_ids._get_last()

    def payment_action_capture(self):
        """ Capture all transactions linked to this invoice. """
        payment_utils.check_rights_on_recordset(self)
        # In sudo mode because we need to be able to read on acquirer fields.
        self.authorized_transaction_ids.sudo().action_capture()

    def payment_action_void(self):
        """ Void all transactions linked to this invoice. """
        payment_utils.check_rights_on_recordset(self)
        # In sudo mode because we need to be able to read on acquirer fields.
        self.authorized_transaction_ids.sudo().action_void()

    def action_view_payment_transactions(self):
        action = self.env['ir.actions.act_window']._for_xml_id('payment.action_payment_transaction')

        if len(self.transaction_ids) == 1:
            action['view_mode'] = 'form'
            action['res_id'] = self.transaction_ids.id
            action['views'] = []
        else:
            action['domain'] = [('id', 'in', self.transaction_ids.ids)]

        return action

```

## File: models\account_payment.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, Command, fields, models
from odoo.exceptions import ValidationError


class AccountPayment(models.Model):
    _inherit = 'account.payment'

    # == Business fields ==
    payment_transaction_id = fields.Many2one(
        string="Payment Transaction",
        comodel_name='payment.transaction',
        readonly=True,
        auto_join=True,  # No access rule bypass since access to payments means access to txs too
    )
    payment_token_id = fields.Many2one(
        string="Saved Payment Token", comodel_name='payment.token', domain="""[
            ('id', 'in', suitable_payment_token_ids),
        ]""",
        help="Note that only tokens from acquirers allowing to capture the amount are available.")
    amount_available_for_refund = fields.Monetary(compute='_compute_amount_available_for_refund')

    # == Display purpose fields ==
    suitable_payment_token_ids = fields.Many2many(
        comodel_name='payment.token',
        compute='_compute_suitable_payment_token_ids',
        compute_sudo=True,
    )
    use_electronic_payment_method = fields.Boolean(
        compute='_compute_use_electronic_payment_method',
        help='Technical field used to hide or show the payment_token_id if needed.'
    )

    # == Fields used for traceability ==
    source_payment_id = fields.Many2one(
        string="Source Payment",
        comodel_name='account.payment',
        help="The source payment of related refund payments",
        related='payment_transaction_id.source_transaction_id.payment_id',
        readonly=True,
        store=True,  # Stored for the group by in `_compute_refunds_count`
    )
    refunds_count = fields.Integer(string="Refunds Count", compute='_compute_refunds_count')

    def _compute_amount_available_for_refund(self):
        for payment in self:
            tx_sudo = payment.payment_transaction_id.sudo()
            if tx_sudo.acquirer_id.support_refund and tx_sudo.operation != 'refund':
                # Only consider refund transactions that are confirmed by summing the amounts of
                # payments linked to such refund transactions. Indeed, should a refund transaction
                # be stuck forever in a transient state (due to webhook failure, for example), the
                # user would never be allowed to refund the source transaction again.
                refund_payments = self.search([('source_payment_id', '=', payment.id)])
                refunded_amount = abs(sum(refund_payments.mapped('amount')))
                payment.amount_available_for_refund = payment.amount - refunded_amount
            else:
                payment.amount_available_for_refund = 0

    @api.depends('payment_method_line_id')
    def _compute_suitable_payment_token_ids(self):
        for payment in self:
            related_partner_ids = (
                    payment.partner_id
                    | payment.partner_id.commercial_partner_id
                    | payment.partner_id.commercial_partner_id.child_ids
            )._origin

            if payment.use_electronic_payment_method:
                payment.suitable_payment_token_ids = self.env['payment.token'].sudo().search([
                    ('company_id', '=', payment.company_id.id),
                    ('acquirer_id.capture_manually', '=', False),
                    ('partner_id', 'in', related_partner_ids.ids),
                    ('acquirer_id', '=', payment.payment_method_line_id.payment_acquirer_id.id),
                ])
            else:
                payment.suitable_payment_token_ids = [Command.clear()]

    @api.depends('payment_method_line_id')
    def _compute_use_electronic_payment_method(self):
        for payment in self:
            # Get a list of all electronic payment method codes.
            # These codes are comprised of 'electronic' and the providers of each payment acquirer.
            codes = [key for key in dict(self.env['payment.acquirer']._fields['provider']._description_selection(self.env))]
            payment.use_electronic_payment_method = payment.payment_method_code in codes

    def _compute_refunds_count(self):
        rg_data = self.env['account.payment'].read_group(
            domain=[
                ('source_payment_id', 'in', self.ids),
                ('payment_transaction_id.operation', '=', 'refund')
            ],
            fields=['source_payment_id'],
            groupby=['source_payment_id']
        )
        data = {x['source_payment_id'][0]: x['source_payment_id_count'] for x in rg_data}
        for payment in self:
            payment.refunds_count = data.get(payment.id, 0)

    @api.onchange('partner_id', 'payment_method_line_id', 'journal_id')
    def _onchange_set_payment_token_id(self):
        codes = [key for key in dict(self.env['payment.acquirer']._fields['provider']._description_selection(self.env))]
        if not (self.payment_method_code in codes and self.partner_id and self.journal_id):
            self.payment_token_id = False
            return

        related_partner_ids = (
                self.partner_id
                | self.partner_id.commercial_partner_id
                | self.partner_id.commercial_partner_id.child_ids
        )._origin

        self.payment_token_id = self.env['payment.token'].sudo().search([
            ('company_id', '=', self.company_id.id),
            ('partner_id', 'in', related_partner_ids.ids),
            ('acquirer_id.capture_manually', '=', False),
            ('acquirer_id', '=', self.payment_method_line_id.payment_acquirer_id.id),
         ], limit=1)

    def action_post(self):
        # Post the payments "normally" if no transactions are needed.
        # If not, let the acquirer update the state.

        payments_need_tx = self.filtered(
            lambda p: p.payment_token_id and not p.payment_transaction_id
        )
        # creating the transaction require to access data on payment acquirers, not always accessible to users
        # able to create payments
        transactions = payments_need_tx.sudo()._create_payment_transaction()

        res = super(AccountPayment, self - payments_need_tx).action_post()

        for tx in transactions:  # Process the transactions with a payment by token
            tx._send_payment_request()

        # Post payments for issued transactions
        transactions._finalize_post_processing()
        payments_tx_done = payments_need_tx.filtered(
            lambda p: p.payment_transaction_id.state == 'done'
        )
        super(AccountPayment, payments_tx_done).action_post()
        payments_tx_not_done = payments_need_tx.filtered(
            lambda p: p.payment_transaction_id.state != 'done'
        )
        payments_tx_not_done.action_cancel()

        return res

    def action_refund_wizard(self):
        self.ensure_one()
        return {
            'name': _("Refund"),
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'payment.refund.wizard',
            'target': 'new',
        }

    def action_view_refunds(self):
        self.ensure_one()
        action = {
            'name': _("Refund"),
            'res_model': 'account.payment',
            'type': 'ir.actions.act_window',
        }
        if self.refunds_count == 1:
            refund_tx = self.env['account.payment'].search([
                ('source_payment_id', '=', self.id)
            ], limit=1)
            action['res_id'] = refund_tx.id
            action['view_mode'] = 'form'
        else:
            action['view_mode'] = 'tree,form'
            action['domain'] = [('source_payment_id', '=', self.id)]
        return action

    def _get_payment_chatter_link(self):
        self.ensure_one()
        return f'<a href=# data-oe-model=account.payment data-oe-id={self.id}>{self.name}</a>'

    def _create_payment_transaction(self, **extra_create_values):
        for payment in self:
            if payment.payment_transaction_id:
                raise ValidationError(_(
                    "A payment transaction with reference %s already exists.",
                    payment.payment_transaction_id.reference
                ))
            elif not payment.payment_token_id:
                raise ValidationError(_("A token is required to create a new payment transaction."))

        transactions = self.env['payment.transaction']
        for payment in self:
            transaction_vals = payment._prepare_payment_transaction_vals(**extra_create_values)
            transaction = self.env['payment.transaction'].create(transaction_vals)
            transactions += transaction
            payment.payment_transaction_id = transaction  # Link the transaction to the payment
        return transactions

    def _prepare_payment_transaction_vals(self, **extra_create_values):
        self.ensure_one()
        return {
            'acquirer_id': self.payment_token_id.acquirer_id.id,
            'reference': self.ref,
            'amount': self.amount,
            'currency_id': self.currency_id.id,
            'partner_id': self.partner_id.id,
            'token_id': self.payment_token_id.id,
            'operation': 'offline',
            'payment_id': self.id,
            **extra_create_values,
        }

```

## File: models\account_payment_method.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError
from odoo.osv import expression


class AccountPaymentMethodLine(models.Model):
    _inherit = "account.payment.method.line"

    payment_acquirer_id = fields.Many2one(
        comodel_name='payment.acquirer',
        compute='_compute_payment_acquirer_id',
        store=True,
        readonly=False,
    )
    payment_acquirer_state = fields.Selection(
        related='payment_acquirer_id.state'
    )

    @api.depends('payment_method_id')
    def _compute_payment_acquirer_id(self):
        results = self.journal_id._get_journals_payment_method_information()
        manage_acquirers = results['manage_acquirers']
        method_information_mapping = results['method_information_mapping']
        acquirers_per_code = results['acquirers_per_code']

        for line in self:
            journal = line.journal_id
            company = journal.company_id
            if (
                company
                and line.payment_method_id
                and manage_acquirers
                and line.payment_method_id.id in method_information_mapping
                and method_information_mapping[line.payment_method_id.id]['mode'] == 'electronic'
            ):
                acquirer_ids = acquirers_per_code.get(company.id, {}).get(line.code, set())

                # Exclude the 'unique' / 'electronic' values that are already set on the journal.
                protected_acquirer_ids = set()
                for payment_type in ('inbound', 'outbound'):
                    lines = journal[f'{payment_type}_payment_method_line_ids']
                    for journal_line in lines:
                        if journal_line.payment_method_id and journal_line.payment_method_id.id in method_information_mapping:
                            if manage_acquirers and method_information_mapping[journal_line.payment_method_id.id]['mode'] == 'electronic':
                                protected_acquirer_ids.add(journal_line.payment_acquirer_id.id)

                candidates_acquirer_ids = acquirer_ids - protected_acquirer_ids
                if candidates_acquirer_ids:
                    line.payment_acquirer_id = list(candidates_acquirer_ids)[0]

    def _get_payment_method_domain(self):
        # OVERRIDE
        domain = super()._get_payment_method_domain()
        information = self._get_payment_method_information().get(self.code)

        unique = information.get('mode') == 'unique'
        if unique:
            company_ids = self.env['payment.acquirer'].sudo().search([('provider', '=', self.code)]).mapped('company_id')
            if company_ids:
                domain = expression.AND([domain, [('company_id', 'in', company_ids.ids)]])

        return domain

    @api.ondelete(at_uninstall=False)
    def _unlink_except_active_acquirer(self):
        """ Ensure we don't remove an account.payment.method.line that is linked to an acquirer
        in the test or enabled state.
        """
        active_acquirer = self.payment_acquirer_id.filtered(lambda acquirer: acquirer.state in ['enabled', 'test'])
        if active_acquirer:
            raise UserError(_(
                "You can't delete a payment method that is linked to a provider in the enabled "
                "or test state.\n""Linked providers(s): %s",
                ', '.join(a.display_name for a in active_acquirer),
            ))

    def action_open_acquirer_form(self):
        self.ensure_one()
        return {
            'type': 'ir.actions.act_window',
            'name': _('Acquirer'),
            'view_mode': 'form',
            'res_model': 'payment.acquirer',
            'target': 'current',
            'res_id': self.payment_acquirer_id.id
        }

```

## File: models\ir_http.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class IrHttp(models.AbstractModel):
    _inherit = 'ir.http'

    @classmethod
    def _get_translation_frontend_modules_name(cls):
        mods = super(IrHttp, cls)._get_translation_frontend_modules_name()
        return mods + ['payment']

```

## File: models\ir_ui_view.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, models
from odoo.exceptions import UserError


class IrUiView(models.Model):
    _inherit = 'ir.ui.view'

    @api.ondelete(at_uninstall=False)
    def _unlink_if_not_referenced_by_acquirer(self):
        referencing_acquirers_sudo = self.env['payment.acquirer'].sudo().search([
            '|', ('redirect_form_view_id', 'in', self.ids), ('inline_form_view_id', 'in', self.ids)
        ])  # In sudo mode to allow non-admin users (e.g., Website designers) to read the view ids.
        if referencing_acquirers_sudo:
            raise UserError(_("You cannot delete a view that is used by a payment acquirer."))

```

## File: models\payment_acquirer.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from odoo import _, api, fields, models, SUPERUSER_ID
from odoo.exceptions import UserError, ValidationError
from odoo.osv import expression

_logger = logging.getLogger(__name__)


class PaymentAcquirer(models.Model):
    _name = 'payment.acquirer'
    _description = 'Payment Acquirer'
    _order = 'module_state, state desc, sequence, name'

    def _valid_field_parameter(self, field, name):
        return name == 'required_if_provider' or super()._valid_field_parameter(field, name)

    # Configuration fields
    name = fields.Char(string="Name", required=True, translate=True)
    sequence = fields.Integer(string="Sequence", help="Define the display order")
    provider = fields.Selection(
        string="Provider", help="The Payment Service Provider to use with this acquirer",
        selection=[('none', "No Provider Set")], default='none', required=True)
    state = fields.Selection(
        string="State",
        help="In test mode, a fake payment is processed through a test payment interface.\n"
             "This mode is advised when setting up the acquirer.",
        selection=[('disabled', "Disabled"), ('enabled', "Enabled"), ('test', "Test Mode")],
        default='disabled', required=True, copy=False)
    company_id = fields.Many2one(  # Indexed to speed-up ORM searches (from ir_rule or others)
        string="Company", comodel_name='res.company', default=lambda self: self.env.company.id,
        required=True, index=True)
    payment_icon_ids = fields.Many2many(
        string="Supported Payment Icons", comodel_name='payment.icon')
    allow_tokenization = fields.Boolean(
        string="Allow Saving Payment Methods",
        help="This controls whether customers can save their payment methods as payment tokens.\n"
             "A payment token is an anonymous link to the payment method details saved in the\n"
             "acquirer's database, allowing the customer to reuse it for a next purchase.")
    capture_manually = fields.Boolean(
        string="Capture Amount Manually",
        help="Capture the amount from Odoo, when the delivery is completed.\n"
             "Use this if you want to charge your customers cards only when\n"
             "you are sure you can ship the goods to them.")
    redirect_form_view_id = fields.Many2one(
        string="Redirect Form Template", comodel_name='ir.ui.view',
        help="The template rendering a form submitted to redirect the user when making a payment",
        domain=[('type', '=', 'qweb')])
    inline_form_view_id = fields.Many2one(
        string="Inline Form Template", comodel_name='ir.ui.view',
        help="The template rendering the inline payment form when making a direct payment",
        domain=[('type', '=', 'qweb')])
    country_ids = fields.Many2many(
        string="Countries", comodel_name='res.country', relation='payment_country_rel',
        column1='payment_id', column2='country_id',
        help="The countries for which this payment acquirer is available.\n"
             "If none is set, it is available for all countries.")
    journal_id = fields.Many2one(
        string="Payment Journal", comodel_name='account.journal',
        compute='_compute_journal_id', inverse='_inverse_journal_id',
        help="The journal in which the successful transactions are posted",
        domain="[('type', '=', 'bank'), ('company_id', '=', company_id)]")

    # Fees fields
    fees_active = fields.Boolean(string="Add Extra Fees")
    fees_dom_fixed = fields.Float(string="Fixed domestic fees")
    fees_dom_var = fields.Float(string="Variable domestic fees (in percents)")
    fees_int_fixed = fields.Float(string="Fixed international fees")
    fees_int_var = fields.Float(string="Variable international fees (in percents)")

    # Message fields
    display_as = fields.Char(
        string="Displayed as", help="Description of the acquirer for customers",
        translate=True)
    pre_msg = fields.Html(
        string="Help Message", help="The message displayed to explain and help the payment process",
        translate=True)
    pending_msg = fields.Html(
        string="Pending Message",
        help="The message displayed if the order pending after the payment process",
        default=lambda self: _(
            "Your payment has been successfully processed but is waiting for approval."
        ), translate=True)
    auth_msg = fields.Html(
        string="Authorize Message", help="The message displayed if payment is authorized",
        default=lambda self: _("Your payment has been authorized."), translate=True)
    done_msg = fields.Html(
        string="Done Message",
        help="The message displayed if the order is successfully done after the payment process",
        default=lambda self: _("Your payment has been successfully processed. Thank you!"),
        translate=True)
    cancel_msg = fields.Html(
        string="Canceled Message",
        help="The message displayed if the order is canceled during the payment process",
        default=lambda self: _("Your payment has been cancelled."), translate=True)

    # Feature support fields
    support_authorization = fields.Boolean(string="Authorize Mechanism Supported")
    support_fees_computation = fields.Boolean(string="Fees Computation Supported")
    support_tokenization = fields.Boolean(string="Tokenization Supported")
    support_refund = fields.Selection(
        string="Type of Refund Supported",
        selection=[('full_only', "Full Only"), ('partial', "Partial")],
    )

    # Kanban view fields
    description = fields.Html(
        string="Description", help="The description shown in the card in kanban view ")
    image_128 = fields.Image(string="Image", max_width=128, max_height=128)
    color = fields.Integer(
        string="Color", help="The color of the card in kanban view", compute='_compute_color',
        store=True)

    # Module-related fields
    module_id = fields.Many2one(string="Corresponding Module", comodel_name='ir.module.module')
    module_state = fields.Selection(
        string="Installation State", related='module_id.state', store=True)  # Stored for sorting
    module_to_buy = fields.Boolean(string="Odoo Enterprise Module", related='module_id.to_buy')

    # View configuration fields
    show_credentials_page = fields.Boolean(compute='_compute_view_configuration_fields')
    show_allow_tokenization = fields.Boolean(compute='_compute_view_configuration_fields')
    show_payment_icon_ids = fields.Boolean(compute='_compute_view_configuration_fields')
    show_pre_msg = fields.Boolean(compute='_compute_view_configuration_fields')
    show_pending_msg = fields.Boolean(compute='_compute_view_configuration_fields')
    show_auth_msg = fields.Boolean(compute='_compute_view_configuration_fields')
    show_done_msg = fields.Boolean(compute='_compute_view_configuration_fields')
    show_cancel_msg = fields.Boolean(compute='_compute_view_configuration_fields')

    #=== COMPUTE METHODS ===#

    @api.depends('state', 'module_state')
    def _compute_color(self):
        """ Update the color of the kanban card based on the state of the acquirer.

        :return: None
        """
        for acquirer in self:
            if acquirer.module_id and not acquirer.module_state == 'installed':
                acquirer.color = 4  # blue
            elif acquirer.state == 'disabled':
                acquirer.color = 3  # yellow
            elif acquirer.state == 'test':
                acquirer.color = 2  # orange
            elif acquirer.state == 'enabled':
                acquirer.color = 7  # green

    @api.depends('provider')
    def _compute_view_configuration_fields(self):
        """ Compute view configuration fields based on the provider.

        By default, all fields are set to `True`.
        For an acquirer to hide generic elements (pages, fields) in a view, it must override this
        method and set their corresponding view configuration field to `False`.

        :return: None
        """
        self.update({
            'show_credentials_page': True,
            'show_allow_tokenization': True,
            'show_payment_icon_ids': True,
            'show_pre_msg': True,
            'show_pending_msg': True,
            'show_auth_msg': True,
            'show_done_msg': True,
            'show_cancel_msg': True,
        })

    def _ensure_payment_method_line(self, allow_create=True):
        self.ensure_one()
        if not self.id:
            return

        pay_method_line = self.env['account.payment.method.line'].search(
            [('code', '=', self.provider), ('payment_acquirer_id', '=', self.id)],
            limit=1,
        )

        if not self.journal_id:
            if pay_method_line:
                pay_method_line.unlink()
                return

        if not pay_method_line:
            pay_method_line = self.env['account.payment.method.line'].search(
                [
                    ('company_id', '=', self.company_id.id),
                    ('code', '=', self.provider),
                    ('payment_acquirer_id', '=', False),
                ],
                limit=1,
            )
        if pay_method_line:
            pay_method_line.name = self.name
            pay_method_line.payment_acquirer_id = self
            pay_method_line.journal_id = self.journal_id
        elif allow_create:
            default_payment_method_id = self._get_default_payment_method_id()
            if not default_payment_method_id:
                return

            create_values = {
                'name': self.name,
                'payment_method_id': default_payment_method_id,
                'journal_id': self.journal_id.id,
                'payment_acquirer_id': self.id,
            }
            pay_method_line_same_code = self.env['account.payment.method.line'].search(
                [
                    ('company_id', '=', self.company_id.id),
                    ('code', '=', self.provider),
                ],
                limit=1,
            )
            if pay_method_line_same_code:
                create_values['payment_account_id'] = pay_method_line_same_code.payment_account_id.id
            self.env['account.payment.method.line'].create(create_values)

    @api.depends('provider', 'state', 'company_id')
    def _compute_journal_id(self):
        for acquirer in self:
            pay_method_line = self.env['account.payment.method.line'].search(
                [('code', '=', acquirer.provider), ('payment_acquirer_id', '=', acquirer._origin.id)],
                limit=1,
            )

            if pay_method_line:
                acquirer.journal_id = pay_method_line.journal_id
            elif acquirer.state in ('enabled', 'test'):
                acquirer.journal_id = self.env['account.journal'].search(
                    [
                        ('company_id', '=', acquirer.company_id.id),
                        ('type', '=', 'bank'),
                    ],
                    limit=1,
                )
                if acquirer.id:
                    acquirer._ensure_payment_method_line()

    def _inverse_journal_id(self):
        for acquirer in self:
            acquirer._ensure_payment_method_line()

    def _get_default_payment_method_id(self):
        self.ensure_one()
        return None

    #=== CONSTRAINT METHODS ===#

    @api.constrains('fees_dom_var', 'fees_int_var')
    def _check_fee_var_within_boundaries(self):
        """ Check that variable fees are within realistic boundaries.

        Variable fees values should always be positive and below 100% to respectively avoid negative
        and infinite (division by zero) fees amount.

        :return None
        """
        for acquirer in self:
            if any(not 0 <= fee < 100 for fee in (acquirer.fees_dom_var, acquirer.fees_int_var)):
                raise ValidationError(_("Variable fees must always be positive and below 100%."))

    #=== CRUD METHODS ===#

    @api.model_create_multi
    def create(self, values_list):
        acquirers = super().create(values_list)
        acquirers._check_required_if_provider()
        return acquirers

    def write(self, values):
        result = super().write(values)
        self._check_required_if_provider()
        return result

    def _check_required_if_provider(self):
        """ Check that acquirer-specific required fields have been filled.

        The fields that have the `required_if_provider="<provider>"` attribute are made required
        for all payment.acquirer records with the `provider` field equal to <provider> and with the
        `state` field equal to 'enabled' or 'test'.
        Acquirer-specific views should make the form fields required under the same conditions.

        :return: None
        :raise ValidationError: if an acquirer-specific required field is empty
        """
        field_names = []
        enabled_acquirers = self.filtered(lambda acq: acq.state in ['enabled', 'test'])
        for name, field in self._fields.items():
            required_provider = getattr(field, 'required_if_provider', None)
            if required_provider and any(
                required_provider == acquirer.provider and not acquirer[name]
                for acquirer in enabled_acquirers
            ):
                ir_field = self.env['ir.model.fields']._get(self._name, name)
                field_names.append(ir_field.field_description)
        if field_names:
            raise ValidationError(
                _("The following fields must be filled: %s", ", ".join(field_names))
            )

    @api.ondelete(at_uninstall=False)
    def _unlink_except_master_data(self):
        """ Prevent the deletion of the payment acquirer if it has an xmlid. """
        external_ids = self.get_external_id()
        for acquirer in self:
            external_id = external_ids[acquirer.id]
            if external_id and not external_id.startswith('__export__'):
                raise UserError(_(
                    "You cannot delete the payment acquirer %s; disable it or uninstall it instead.",
                    acquirer.name,
                ))

    #=== ACTION METHODS ===#

    def button_immediate_install(self):
        """ Install the acquirer's module and reload the page.

        Note: self.ensure_one()

        :return: The action to reload the page
        :rtype: dict
        """
        if self.module_id and self.module_state != 'installed':
            self.module_id.button_immediate_install()
            return {
                'type': 'ir.actions.client',
                'tag': 'reload',
            }

    #=== BUSINESS METHODS ===#

    @api.model
    def _get_compatible_acquirers(
        self, company_id, partner_id, currency_id=None, force_tokenization=False,
        is_validation=False, **kwargs
    ):
        """ Select and return the acquirers matching the criteria.

        The base criteria are that acquirers must not be disabled, be in the company that is
        provided, and support the country of the partner if it exists.

        :param int company_id: The company to which acquirers must belong, as a `res.company` id
        :param int partner_id: The partner making the payment, as a `res.partner` id
        :param int currency_id: The payment currency if known beforehand, as a `res.currency` id
        :param bool force_tokenization: Whether only acquirers allowing tokenization can be matched
        :param bool is_validation: Whether the operation is a validation
        :param dict kwargs: Optional data. This parameter is not used here
        :return: The compatible acquirers
        :rtype: recordset of `payment.acquirer`
        """
        # Compute the base domain for compatible acquirers
        domain = ['&', ('state', 'in', ['enabled', 'test']), ('company_id', '=', company_id)]

        # Handle partner country
        partner = self.env['res.partner'].browse(partner_id)
        if partner.country_id:  # The partner country must either not be set or be supported
            domain = expression.AND([
                domain,
                ['|', ('country_ids', '=', False), ('country_ids', 'in', [partner.country_id.id])]
            ])

        # Handle tokenization support requirements
        if force_tokenization or self._is_tokenization_required(**kwargs):
            domain = expression.AND([domain, [('allow_tokenization', '=', True)]])

        compatible_acquirers = self.env['payment.acquirer'].search(domain)
        return compatible_acquirers

    @api.model
    def _is_tokenization_required(self, provider=None, **kwargs):
        """ Return whether tokenizing the transaction is required given its context.

        For a module to make the tokenization required based on the transaction context, it must
        override this method and return whether it is required.

        :param str provider: The provider of the acquirer handling the transaction
        :param dict kwargs: The transaction context. This parameter is not used here
        :return: Whether tokenizing the transaction is required
        :rtype: bool
        """
        return False

    def _should_build_inline_form(self, is_validation=False):
        """ Return whether the inline form should be instantiated if it exists.

        For an acquirer to handle both direct payments and payment with redirection, it should
        override this method and return whether the inline form should be instantiated (i.e. if the
        payment should be direct) based on the operation (online payment or validation).

        :param bool is_validation: Whether the operation is a validation
        :return: Whether the inline form should be instantiated
        :rtype: bool
        """
        return True

    def _compute_fees(self, amount, currency, country):
        """ Compute the transaction fees.

        The computation is based on the generic fields `fees_dom_fixed`, `fees_dom_var`,
        `fees_int_fixed` and `fees_int_var` and is done according to the following formula:

        `fees = (amount * variable / 100.0 + fixed) / (1 - variable / 100.0)` where the value
        of `fixed` and `variable` is taken either from the domestic (dom) or international (int)
        field depending on whether the country matches the company's country.

        For an acquirer to base the computation on different variables, or to use a different
        formula, it must override this method and return the resulting fees as a float.

        :param float amount: The amount to pay for the transaction
        :param recordset currency: The currency of the transaction, as a `res.currency` record
        :param recordset country: The customer country, as a `res.country` record
        :return: The computed fees
        :rtype: float
        """
        self.ensure_one()

        fees = 0.0
        if self.fees_active:
            if country == self.company_id.country_id:
                fixed = self.fees_dom_fixed
                variable = self.fees_dom_var
            else:
                fixed = self.fees_int_fixed
                variable = self.fees_int_var
            fees = (amount * variable / 100.0 + fixed) / (1 - variable / 100.0)
        return fees

    def _get_validation_amount(self):
        """ Get the amount to transfer in a payment method validation operation.

        For an acquirer to support tokenization, it must override this method and return the amount
        to be transferred in a payment method validation operation *if the validation amount is not
        null*.

        Note: self.ensure_one()

        :return: The validation amount
        :rtype: float
        """
        self.ensure_one()
        return 0.0

    def _get_validation_currency(self):
        """ Get the currency of the transfer in a payment method validation operation.

        For an acquirer to support tokenization, it must override this method and return the
        currency to be used in a payment method validation operation *if the validation amount is
        not null*.

        Note: self.ensure_one()

        :return: The validation currency
        :rtype: recordset of `res.currency`
        """
        self.ensure_one()
        return self.journal_id.currency_id or self.company_id.currency_id

    def _get_redirect_form_view(self, is_validation=False):
        """ Return the view of the template used to render the redirect form.

        For an acquirer to return a different view depending on whether the operation is a
        validation, it must override this method and return the appropriate view.

        Note: self.ensure_one()

        :param bool is_validation: Whether the operation is a validation
        :return: The redirect form template
        :rtype: record of `ir.ui.view`
        """
        self.ensure_one()
        return self.redirect_form_view_id

```

## File: models\payment_icon.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class PaymentIcon(models.Model):
    _name = 'payment.icon'
    _description = 'Payment Icon'
    _order = 'sequence, name'

    name = fields.Char(string="Name")
    acquirer_ids = fields.Many2many(
        string="Acquirers", comodel_name='payment.acquirer',
        help="The list of acquirers supporting this payment icon")
    image = fields.Image(
        string="Image", max_width=64, max_height=64,
        help="This field holds the image used for this payment icon, limited to 64x64 px")
    image_payment_form = fields.Image(
        string="Image displayed on the payment form", related='image', store=True, max_width=45,
        max_height=30)
    sequence = fields.Integer('Sequence', default=1)

```

## File: models\payment_token.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from odoo import api, fields, models

_logger = logging.getLogger(__name__)


class PaymentToken(models.Model):
    _name = 'payment.token'
    _order = 'partner_id, id desc'
    _description = 'Payment Token'

    acquirer_id = fields.Many2one(
        string="Acquirer Account", comodel_name='payment.acquirer', required=True)
    provider = fields.Selection(related='acquirer_id.provider')
    name = fields.Char(
        string="Name", help="The anonymized acquirer reference of the payment method",
        required=True)
    partner_id = fields.Many2one(string="Partner", comodel_name='res.partner', required=True)
    company_id = fields.Many2one(  # Indexed to speed-up ORM searches (from ir_rule or others)
        related='acquirer_id.company_id', store=True, index=True)
    acquirer_ref = fields.Char(
        string="Acquirer Reference", help="The acquirer reference of the token of the transaction",
        required=True)  # This is not the same thing as the acquirer reference of the transaction
    transaction_ids = fields.One2many(
        string="Payment Transactions", comodel_name='payment.transaction', inverse_name='token_id')
    verified = fields.Boolean(string="Verified")
    active = fields.Boolean(string="Active", default=True)

    #=== CRUD METHODS ===#

    @api.model_create_multi
    def create(self, values_list):
        for values in values_list:
            if 'acquirer_id' in values:
                acquirer = self.env['payment.acquirer'].browse(values['acquirer_id'])

                # Include acquirer-specific create values
                values.update(self._get_specific_create_values(acquirer.provider, values))
            else:
                pass  # Let psycopg warn about the missing required field

        return super().create(values_list)

    @api.model
    def _get_specific_create_values(self, provider, values):
        """ Complete the values of the `create` method with acquirer-specific values.

        For an acquirer to add its own create values, it must overwrite this method and return a
        dict of values. Acquirer-specific values take precedence over those of the dict of generic
        create values.

        :param str provider: The provider of the acquirer managing the token
        :param dict values: The original create values
        :return: The dict of acquirer-specific create values
        :rtype: dict
        """
        return dict()

    def write(self, values):
        """ Delegate the handling of active state switch to dedicated methods.

        Unless an exception is raised in the handling methods, the toggling proceeds no matter what.
        This is because allowing users to hide their saved payment methods comes before making sure
        that the recorded payment details effectively get deleted.

        :return: The result of the write
        :rtype: bool
        """
        # Let acquirers handle activation/deactivation requests
        if 'active' in values:
            for token in self:
                # Call handlers in sudo mode because this method might have been called by RPC
                if values['active'] and not token.active:
                    token.sudo()._handle_reactivation_request()
                elif not values['active'] and token.active:
                    token.sudo()._handle_deactivation_request()

        # Proceed with the toggling of the active state
        return super().write(values)

    #=== BUSINESS METHODS ===#

    def _handle_deactivation_request(self):
        """ Handle the request for deactivation of the token.

        For an acquirer to support deactivation of tokens, or perform additional operations when a
        token is deactivated, it must overwrite this method and raise an UserError if the token
        cannot be deactivated.

        Note: self.ensure_one()

        :return: None
        """
        self.ensure_one()

    def _handle_reactivation_request(self):
        """ Handle the request for reactivation of the token.

        For an acquirer to support reactivation of tokens, or perform additional operations when a
        token is reactivated, it must overwrite this method and raise an UserError if the token
        cannot be reactivated.

        Note: self.ensure_one()

        :return: None
        """
        self.ensure_one()

    def get_linked_records_info(self):
        """ Return a list of information about records linked to the current token.

        For a module to implement payments and link documents to a token, it must override this
        method and add information about linked records to the returned list.

        The information must be structured as a dict with the following keys:
          - description: The description of the record's model (e.g. "Subscription")
          - id: The id of the record
          - name: The name of the record
          - url: The url to access the record.

        Note: self.ensure_one()

        :return: The list of information about linked documents
        :rtype: list
        """
        self.ensure_one()
        return []

```

## File: models\payment_transaction.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import pprint
import re
import unicodedata
from datetime import datetime

import psycopg2
from dateutil import relativedelta

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError
from odoo.tools import consteq, format_amount, ustr
from odoo.tools.misc import hmac as hmac_tool

from odoo.addons.payment import utils as payment_utils

_logger = logging.getLogger(__name__)


class PaymentTransaction(models.Model):
    _name = 'payment.transaction'
    _description = 'Payment Transaction'
    _order = 'id desc'
    _rec_name = 'reference'

    @api.model
    def _lang_get(self):
        return self.env['res.lang'].get_installed()

    acquirer_id = fields.Many2one(
        string="Acquirer", comodel_name='payment.acquirer', readonly=True, required=True)
    provider = fields.Selection(related='acquirer_id.provider')
    company_id = fields.Many2one(  # Indexed to speed-up ORM searches (from ir_rule or others)
        related='acquirer_id.company_id', store=True, index=True)
    reference = fields.Char(
        string="Reference", help="The internal reference of the transaction", readonly=True,
        required=True)  # Already has an index from the UNIQUE SQL constraint
    acquirer_reference = fields.Char(
        string="Acquirer Reference", help="The acquirer reference of the transaction",
        readonly=True)  # This is not the same thing as the acquirer reference of the token
    amount = fields.Monetary(
        string="Amount", currency_field='currency_id', readonly=True, required=True)
    currency_id = fields.Many2one(
        string="Currency", comodel_name='res.currency', readonly=True, required=True)
    fees = fields.Monetary(
        string="Fees", currency_field='currency_id',
        help="The fees amount; set by the system as it depends on the acquirer", readonly=True)
    token_id = fields.Many2one(
        string="Payment Token", comodel_name='payment.token', readonly=True,
        domain='[("acquirer_id", "=", "acquirer_id")]', ondelete='restrict')
    state = fields.Selection(
        string="Status",
        selection=[('draft', "Draft"), ('pending', "Pending"), ('authorized', "Authorized"),
                   ('done', "Confirmed"), ('cancel', "Canceled"), ('error', "Error")],
        default='draft', readonly=True, required=True, copy=False, index=True)
    state_message = fields.Text(
        string="Message", help="The complementary information message about the state",
        readonly=True)
    last_state_change = fields.Datetime(
        string="Last State Change Date", readonly=True, default=fields.Datetime.now)

    # Fields used for traceability
    operation = fields.Selection(  # This should not be trusted if the state is 'draft' or 'pending'
        string="Operation",
        selection=[
            ('online_redirect', "Online payment with redirection"),
            ('online_direct', "Online direct payment"),
            ('online_token', "Online payment by token"),
            ('validation', "Validation of the payment method"),
            ('offline', "Offline payment by token"),
            ('refund', "Refund")
        ],
        readonly=True,
        index=True,
    )
    payment_id = fields.Many2one(string="Payment", comodel_name='account.payment', readonly=True)
    source_transaction_id = fields.Many2one(
        string="Source Transaction",
        comodel_name='payment.transaction',
        help="The source transaction of related refund transactions",
        readonly=True
    )
    refunds_count = fields.Integer(string="Refunds Count", compute='_compute_refunds_count')
    invoice_ids = fields.Many2many(
        string="Invoices", comodel_name='account.move', relation='account_invoice_transaction_rel',
        column1='transaction_id', column2='invoice_id', readonly=True, copy=False,
        domain=[('move_type', 'in', ('out_invoice', 'out_refund', 'in_invoice', 'in_refund'))])
    invoices_count = fields.Integer(string="Invoices Count", compute='_compute_invoices_count')

    # Fields used for user redirection & payment post-processing
    is_post_processed = fields.Boolean(
        string="Is Post-processed", help="Has the payment been post-processed")
    tokenize = fields.Boolean(
        string="Create Token",
        help="Whether a payment token should be created when post-processing the transaction")
    landing_route = fields.Char(
        string="Landing Route",
        help="The route the user is redirected to after the transaction")
    callback_model_id = fields.Many2one(
        string="Callback Document Model", comodel_name='ir.model', groups='base.group_system')
    callback_res_id = fields.Integer(string="Callback Record ID", groups='base.group_system')
    callback_method = fields.Char(string="Callback Method", groups='base.group_system')
    # Hash for additional security on top of the callback fields' group in case a bug exposes a sudo
    callback_hash = fields.Char(string="Callback Hash", groups='base.group_system')
    callback_is_done = fields.Boolean(
        string="Callback Done", help="Whether the callback has already been executed",
        groups="base.group_system", readonly=True)

    # Duplicated partner values allowing to keep a record of them, should they be later updated
    partner_id = fields.Many2one(
        string="Customer", comodel_name='res.partner', readonly=True, required=True,
        ondelete='restrict')
    partner_name = fields.Char(string="Partner Name")
    partner_lang = fields.Selection(string="Language", selection=_lang_get)
    partner_email = fields.Char(string="Email")
    partner_address = fields.Char(string="Address")
    partner_zip = fields.Char(string="Zip")
    partner_city = fields.Char(string="City")
    partner_state_id = fields.Many2one(string="State", comodel_name='res.country.state')
    partner_country_id = fields.Many2one(string="Country", comodel_name='res.country')
    partner_phone = fields.Char(string="Phone")

    _sql_constraints = [
        ('reference_uniq', 'unique(reference)', "Reference must be unique!"),
    ]

    #=== COMPUTE METHODS ===#

    @api.depends('invoice_ids')
    def _compute_invoices_count(self):
        tx_data = {}
        if self.ids:
            self.env.cr.execute(
                '''
                SELECT transaction_id, count(invoice_id)
                FROM account_invoice_transaction_rel
                WHERE transaction_id IN %s
                GROUP BY transaction_id
                ''',
                [tuple(self.ids)]
            )
            tx_data = dict(self.env.cr.fetchall())  # {id: count}
        for tx in self:
            tx.invoices_count = tx_data.get(tx.id, 0)

    def _compute_refunds_count(self):
        rg_data = self.env['payment.transaction'].read_group(
            domain=[('source_transaction_id', 'in', self.ids), ('operation', '=', 'refund')],
            fields=['source_transaction_id'],
            groupby=['source_transaction_id'],
        )
        data = {x['source_transaction_id'][0]: x['source_transaction_id_count'] for x in rg_data}
        for record in self:
            record.refunds_count = data.get(record.id, 0)

    #=== CONSTRAINT METHODS ===#

    @api.constrains('state')
    def _check_state_authorized_supported(self):
        """ Check that authorization is supported for a transaction in the 'authorized' state. """
        illegal_authorize_state_txs = self.filtered(
            lambda tx: tx.state == 'authorized' and not tx.acquirer_id.support_authorization
        )
        if illegal_authorize_state_txs:
            raise ValidationError(_(
                "Transaction authorization is not supported by the following payment acquirers: %s",
                ', '.join(set(illegal_authorize_state_txs.mapped('acquirer_id.name')))
            ))

    #=== CRUD METHODS ===#

    @api.model_create_multi
    def create(self, values_list):
        for values in values_list:
            acquirer = self.env['payment.acquirer'].browse(values['acquirer_id'])

            if not values.get('reference'):
                values['reference'] = self._compute_reference(acquirer.provider, **values)

            # Duplicate partner values
            partner = self.env['res.partner'].browse(values['partner_id'])
            values.update({
                # Use the parent partner as fallback if the invoicing address has no name.
                'partner_name': partner.name or partner.parent_id.name,
                'partner_lang': partner.lang,
                'partner_email': partner.email,
                'partner_address': payment_utils.format_partner_address(
                    partner.street, partner.street2
                ),
                'partner_zip': partner.zip,
                'partner_city': partner.city,
                'partner_state_id': partner.state_id.id,
                'partner_country_id': partner.country_id.id,
                'partner_phone': partner.phone,
            })

            # Compute fees, for validation transactions fees are zero
            if values.get('operation') == 'validation':
                values['fees'] = 0
            else:
                currency = self.env['res.currency'].browse(values.get('currency_id')).exists()
                values['fees'] = acquirer._compute_fees(
                    values.get('amount', 0), currency, partner.country_id,
                )

            # Include acquirer-specific create values
            values.update(self._get_specific_create_values(acquirer.provider, values))

            # Generate the hash for the callback if one has be configured on the tx
            values['callback_hash'] = self._generate_callback_hash(
                values.get('callback_model_id'),
                values.get('callback_res_id'),
                values.get('callback_method'),
            )

        txs = super().create(values_list)

        # Monetary fields are rounded with the currency at creation time by the ORM. Sometimes, this
        # can lead to inconsistent string representation of the amounts sent to the providers.
        # E.g., tx.create(amount=1111.11) -> tx.amount == 1111.1100000000001
        # To ensure a proper string representation, we invalidate this request's cache values of the
        # `amount` and `fees` fields for the created transactions. This forces the ORM to read the
        # values from the DB where there were stored using `float_repr`, which produces a result
        # consistent with the format expected by providers.
        # E.g., tx.create(amount=1111.11) ; tx.invalidate_cache() -> tx.amount == 1111.11
        txs.invalidate_cache(['amount', 'fees'])

        return txs

    @api.model
    def _get_specific_create_values(self, provider, values):
        """ Complete the values of the `create` method with acquirer-specific values.

        For an acquirer to add its own create values, it must overwrite this method and return a
        dict of values. Acquirer-specific values take precedence over those of the dict of generic
        create values.

        :param str provider: The provider of the acquirer that handled the transaction
        :param dict values: The original create values
        :return: The dict of acquirer-specific create values
        :rtype: dict
        """
        return dict()

    #=== ACTION METHODS ===#

    def action_view_invoices(self):
        """ Return the action for the views of the invoices linked to the transaction.

        Note: self.ensure_one()

        :return: The action
        :rtype: dict
        """
        self.ensure_one()

        action = {
            'name': _("Invoices"),
            'type': 'ir.actions.act_window',
            'res_model': 'account.move',
            'target': 'current',
        }
        invoice_ids = self.invoice_ids.ids
        if len(invoice_ids) == 1:
            invoice = invoice_ids[0]
            action['res_id'] = invoice
            action['view_mode'] = 'form'
            action['views'] = [(self.env.ref('account.view_move_form').id, 'form')]
        else:
            action['view_mode'] = 'tree,form'
            action['domain'] = [('id', 'in', invoice_ids)]
        return action

    def action_view_refunds(self):
        """ Return the action for the views of the refund transactions linked to the transaction.

        Note: self.ensure_one()

        :return: The action
        :rtype: dict
        """
        self.ensure_one()

        action = {
            'name': _("Refund"),
            'res_model': 'payment.transaction',
            'type': 'ir.actions.act_window',
        }
        if self.refunds_count == 1:
            refund_tx = self.env['payment.transaction'].search([
                ('source_transaction_id', '=', self.id),
            ])[0]
            action['res_id'] = refund_tx.id
            action['view_mode'] = 'form'
        else:
            action['view_mode'] = 'tree,form'
            action['domain'] = [('source_transaction_id', '=', self.id)]
        return action

    def action_capture(self):
        """ Check the state of the transactions and request their capture. """
        if any(tx.state != 'authorized' for tx in self):
            raise ValidationError(_("Only authorized transactions can be captured."))

        payment_utils.check_rights_on_recordset(self)
        for tx in self:
            # In sudo mode because we need to be able to read on acquirer fields.
            tx.sudo()._send_capture_request()

    def action_void(self):
        """ Check the state of the transaction and request to have them voided. """
        if any(tx.state != 'authorized' for tx in self):
            raise ValidationError(_("Only authorized transactions can be voided."))

        payment_utils.check_rights_on_recordset(self)
        for tx in self:
            # In sudo mode because we need to be able to read on acquirer fields.
            tx.sudo()._send_void_request()

    def action_refund(self, amount_to_refund=None):
        """ Check the state of the transactions and request their refund.

        :param float amount_to_refund: The amount to be refunded
        :return: None
        """
        if any(tx.state != 'done' for tx in self):
            raise ValidationError(_("Only confirmed transactions can be refunded."))

        payment_utils.check_rights_on_recordset(self)
        for tx in self:
            # In sudo mode because we need to be able to read on acquirer fields.
            tx.sudo()._send_refund_request(amount_to_refund)

    #=== BUSINESS METHODS - PAYMENT FLOW ===#

    @api.model
    def _compute_reference(self, provider, prefix=None, separator='-', **kwargs):
        """ Compute a unique reference for the transaction.

        The reference either corresponds to the prefix if no other transaction with that prefix
        already exists, or follows the pattern `{computed_prefix}{separator}{sequence_number}` where
          - {computed_prefix} is:
            - The provided custom prefix, if any.
            - The computation result of `_compute_reference_prefix` if the custom prefix is not
              filled but the kwargs are.
            - 'tx-{datetime}', if neither the custom prefix nor the kwargs are filled.
          - {separator} is a custom string also used in `_compute_reference_prefix`.
          - {sequence_number} is the next integer in the sequence of references sharing the exact
            same prefix, '1' if there is only one matching reference (hence without sequence number)

        Examples:
          - Given the custom prefix 'example' which has no match with an existing reference, the
            full reference will be 'example'.
          - Given the custom prefix 'example' which matches the existing reference 'example', and
            the custom separator '-', the full reference will be 'example-1'.
          - Given the kwargs {'invoice_ids': [1, 2]}, the custom separator '-' and no custom prefix,
            the full reference will be 'INV1-INV2' (or similar) if no existing reference has the
            same prefix, or 'INV1-INV2-n' if n existing references have the same prefix.

        :param str provider: The provider of the acquirer handling the transaction
        :param str prefix: The custom prefix used to compute the full reference
        :param str separator: The custom separator used to separate the prefix from the suffix, and
                              passed to `_compute_reference_prefix` if it is called
        :param dict kwargs: Optional values passed to `_compute_reference_prefix` if no custom
                            prefix is provided
        :return: The unique reference for the transaction
        :rtype: str
        """
        # Compute the prefix
        if prefix:
            # Replace special characters by their ASCII alternative (é -> e ; ä -> a ; ...)
            prefix = unicodedata.normalize('NFKD', prefix).encode('ascii', 'ignore').decode('utf-8')
        if not prefix:  # Prefix not provided or voided above, compute it based on the kwargs
            prefix = self.sudo()._compute_reference_prefix(provider, separator, **kwargs)
        if not prefix:  # Prefix not computed from the kwargs, fallback on time-based value
            prefix = payment_utils.singularize_reference_prefix()

        # Compute the sequence number
        reference = prefix  # The first reference of a sequence has no sequence number
        if self.sudo().search([('reference', '=', prefix)]):  # The reference already has a match
            # We now execute a second search on `payment.transaction` to fetch all the references
            # starting with the given prefix. The load of these two searches is mitigated by the
            # index on `reference`. Although not ideal, this solution allows for quickly knowing
            # whether the sequence for a given prefix is already started or not, usually not. An SQL
            # query wouldn't help either as the selector is arbitrary and doing that would be an
            # open-door to SQL injections.
            same_prefix_references = self.sudo().search(
                [('reference', 'like', f'{prefix}{separator}%')]
            ).with_context(prefetch_fields=False).mapped('reference')

            # A final regex search is necessary to figure out the next sequence number. The previous
            # search could not rely on alphabetically sorting the reference to infer the largest
            # sequence number because both the prefix and the separator are arbitrary. A given
            # prefix could happen to be a substring of the reference from a different sequence.
            # For instance, the prefix 'example' is a valid match for the existing references
            # 'example', 'example-1' and 'example-ref', in that order. Trusting the order to infer
            # the sequence number would lead to a collision with 'example-1'.
            search_pattern = re.compile(rf'^{re.escape(prefix)}{separator}(\d+)$')
            max_sequence_number = 0  # If no match is found, start the sequence with this reference
            for existing_reference in same_prefix_references:
                search_result = re.search(search_pattern, existing_reference)
                if search_result:  # The reference has the same prefix and is from the same sequence
                    # Find the largest sequence number, if any
                    current_sequence = int(search_result.group(1))
                    if current_sequence > max_sequence_number:
                        max_sequence_number = current_sequence

            # Compute the full reference
            reference = f'{prefix}{separator}{max_sequence_number + 1}'
        return reference

    @api.model
    def _compute_reference_prefix(self, provider, separator, **values):
        """ Compute the reference prefix from the transaction values.

        If the `values` parameter has an entry with 'invoice_ids' as key and a list of (4, id, O) or
        (6, 0, ids) X2M command as value, the prefix is computed based on the invoice name(s).
        Otherwise, an empty string is returned.

        Note: This method should be called in sudo mode to give access to documents (INV, SO, ...).

        :param str provider: The provider of the acquirer handling the transaction
        :param str separator: The custom separator used to separate data references
        :param dict values: The transaction values used to compute the reference prefix. It should
                            have the structure {'invoice_ids': [(X2M command), ...], ...}.
        :return: The computed reference prefix if invoice ids are found, an empty string otherwise
        :rtype: str
        """
        command_list = values.get('invoice_ids')
        if command_list:
            # Extract invoice id(s) from the X2M commands
            invoice_ids = self._fields['invoice_ids'].convert_to_cache(command_list, self)
            invoices = self.env['account.move'].browse(invoice_ids).exists()
            if len(invoices) == len(invoice_ids):  # All ids are valid
                return separator.join(invoices.mapped('name'))
        return ''

    @api.model
    def _generate_callback_hash(self, callback_model_id, callback_res_id, callback_method):
        """ Return the hash for the callback on the transaction.

        :param int callback_model_id: The model on which the callback method is defined, as a
                                      `res.model` id
        :param int callback_res_id: The record on which the callback method must be called, as an id
                                    of the callback model
        :param str callback_method: The name of the callback method
        :return: The callback hash
        :rtype: str
        """
        if callback_model_id and callback_res_id and callback_method:
            model_name = self.env['ir.model'].sudo().browse(callback_model_id).model
            token = f'{model_name}|{callback_res_id}|{callback_method}'
            callback_hash = hmac_tool(self.env(su=True), 'generate_callback_hash', token)
            return callback_hash
        return None

    def _get_processing_values(self):
        """ Return a dict of values used to process the transaction.

        The returned dict contains the following entries:
            - tx_id: The transaction, as a `payment.transaction` id
            - acquirer_id: The acquirer handling the transaction, as a `payment.acquirer` id
            - provider: The provider of the acquirer
            - reference: The reference of the transaction
            - amount: The rounded amount of the transaction
            - currency_id: The currency of the transaction, as a res.currency id
            - partner_id: The partner making the transaction, as a res.partner id
            - Additional acquirer-specific entries

        Note: self.ensure_one()

        :return: The dict of processing values
        :rtype: dict
        """
        self.ensure_one()

        processing_values = {
            'acquirer_id': self.acquirer_id.id,
            'provider': self.provider,
            'reference': self.reference,
            'amount': self.amount,
            'currency_id': self.currency_id.id,
            'partner_id': self.partner_id.id,
        }

        # Complete generic processing values with acquirer-specific values
        processing_values.update(self._get_specific_processing_values(processing_values))
        _logger.info(
            "generic and acquirer-specific processing values for transaction with id %s:\n%s",
            self.id, pprint.pformat(processing_values)
        )

        # Render the html form for the redirect flow if available
        if self.operation in ('online_redirect', 'validation'):
            redirect_form_view = self.acquirer_id._get_redirect_form_view(
                is_validation=self.operation == 'validation'
            )
            if redirect_form_view:  # Some acquirer don't need a redirect form
                rendering_values = self._get_specific_rendering_values(processing_values)
                _logger.info(
                    "acquirer-specific rendering values for transaction with id %s:\n%s",
                    self.id, pprint.pformat(rendering_values)
                )
                redirect_form_html = redirect_form_view._render(rendering_values, engine='ir.qweb')
                processing_values.update(redirect_form_html=redirect_form_html)

        return processing_values

    def _get_specific_processing_values(self, processing_values):
        """ Return a dict of acquirer-specific values used to process the transaction.

        For an acquirer to add its own processing values, it must overwrite this method and return a
        dict of acquirer-specific values based on the generic values returned by this method.
        Acquirer-specific values take precedence over those of the dict of generic processing
        values.

        :param dict processing_values: The generic processing values of the transaction
        :return: The dict of acquirer-specific processing values
        :rtype: dict
        """
        return dict()

    def _get_specific_rendering_values(self, processing_values):
        """ Return a dict of acquirer-specific values used to render the redirect form.

        For an acquirer to add its own rendering values, it must overwrite this method and return a
        dict of acquirer-specific values based on the processing values (acquirer-specific
        processing values included).

        :param dict processing_values: The processing values of the transaction
        :return: The dict of acquirer-specific rendering values
        :rtype: dict
        """
        return dict()

    def _send_payment_request(self):
        """ Request the provider of the acquirer handling the transaction to execute the payment.

        For an acquirer to support tokenization, it must override this method and call it to log the
        'sent' message, then request a money transfer to its provider.

        Note: self.ensure_one()

        :return: None
        """
        self.ensure_one()
        self._log_sent_message()

    def _send_refund_request(self, amount_to_refund=None, create_refund_transaction=True):
        """ Request the provider of the acquirer handling the transaction to refund it.

        For an acquirer to support refunds, it must override this method and request a refund
        to its provider.

        Note: self.ensure_one()

        :param float amount_to_refund: The amount to be refunded
        :param bool create_refund_transaction: Whether a refund transaction should be created
        :return: The refund transaction if any
        :rtype: recordset of `payment.transaction`
        """
        self.ensure_one()

        if create_refund_transaction:
            refund_tx = self._create_refund_transaction(amount_to_refund=amount_to_refund)
            refund_tx._log_sent_message()
            return refund_tx
        else:
            return self.env['payment.transaction']

    def _send_capture_request(self):
        """ Request the provider of the acquirer handling the transaction to capture it.

        For an acquirer to support authorization, it must override this method and request a capture
        to its provider.

        Note: self.ensure_one()

        :return: None
        """
        self.ensure_one()

    def _send_void_request(self):
        """ Request the provider of the acquirer handling the transaction to void it.

        For an acquirer to support authorization, it must override this method and request the
        transaction to be voided to its provider.

        Note: self.ensure_one()

        :return: None
        """
        self.ensure_one()

    def _create_refund_transaction(self, amount_to_refund=None, **custom_create_values):
        """ Create a new transaction with operation 'refund' and link it to the current transaction.

        :param float amount_to_refund: The strictly positive amount to refund, in the same currency
                                       as the source transaction
        :return: The refund transaction
        :rtype: recordset of `payment.transaction`
        """
        self.ensure_one()

        return self.create({
            'acquirer_id': self.acquirer_id.id,
            'reference': self._compute_reference(self.provider, prefix=f'R-{self.reference}'),
            'amount': -(amount_to_refund or self.amount),
            'currency_id': self.currency_id.id,
            'token_id': self.token_id.id,
            'operation': 'refund',
            'source_transaction_id': self.id,
            'partner_id': self.partner_id.id,
            **custom_create_values,
        })

    @api.model
    def _handle_feedback_data(self, provider, data):
        """ Match the transaction with the feedback data, update its state and return it.

        :param str provider: The provider of the acquirer that handled the transaction
        :param dict data: The feedback data sent by the provider
        :return: The transaction
        :rtype: recordset of `payment.transaction`
        """
        tx = self._get_tx_from_feedback_data(provider, data)
        tx._process_feedback_data(data)
        tx._execute_callback()
        return tx

    @api.model
    def _get_tx_from_feedback_data(self, provider, data):
        """ Find the transaction based on the feedback data.

        For an acquirer to handle transaction post-processing, it must overwrite this method and
        return the transaction matching the data.

        :param str provider: The provider of the acquirer that handled the transaction
        :param dict data: The feedback data sent by the acquirer
        :return: The transaction if found
        :rtype: recordset of `payment.transaction`
        """
        return self

    def _process_feedback_data(self, data):
        """ Update the transaction state and the acquirer reference based on the feedback data.

        For an acquirer to handle transaction post-processing, it must overwrite this method and
        process the feedback data.

        Note: self.ensure_one()

        :param dict data: The feedback data sent by the acquirer
        :return: None
        """
        self.ensure_one()

    def _set_pending(self, state_message=None):
        """ Update the transactions' state to 'pending'.

        :param str state_message: The reason for which the transaction is set in 'pending' state
        :return: None
        """
        allowed_states = ('draft',)
        target_state = 'pending'
        txs_to_process = self._update_state(allowed_states, target_state, state_message)
        txs_to_process._log_received_message()

    def _set_authorized(self, state_message=None):
        """ Update the transactions' state to 'authorized'.

        :param str state_message: The reason for which the transaction is set in 'authorized' state
        :return: None
        """
        allowed_states = ('draft', 'pending')
        target_state = 'authorized'
        txs_to_process = self._update_state(allowed_states, target_state, state_message)
        txs_to_process._log_received_message()

    def _set_done(self, state_message=None):
        """ Update the transactions' state to 'done'.

        :return: None
        """
        allowed_states = ('draft', 'pending', 'authorized', 'error', 'cancel')  # 'cancel' for Payulatam
        target_state = 'done'
        txs_to_process = self._update_state(allowed_states, target_state, state_message)
        txs_to_process._log_received_message()

    def _set_canceled(self, state_message=None):
        """ Update the transactions' state to 'cancel'.

        :param str state_message: The reason for which the transaction is set in 'cancel' state
        :return: None
        """
        allowed_states = ('draft', 'pending', 'authorized')
        target_state = 'cancel'
        txs_to_process = self._update_state(allowed_states, target_state, state_message)
        # Cancel the existing payments
        txs_to_process.mapped('payment_id').action_cancel()
        txs_to_process._log_received_message()

    def _set_error(self, state_message):
        """ Update the transactions' state to 'error'.

        :param str state_message: The reason for which the transaction is set in 'error' state
        :return: None
        """
        allowed_states = ('draft', 'pending', 'authorized')
        target_state = 'error'
        txs_to_process = self._update_state(allowed_states, target_state, state_message)
        txs_to_process._log_received_message()

    def _update_state(self, allowed_states, target_state, state_message):
        """ Update the transactions' state to the target state if the current state allows it.

        If the current state is the same as the target state, the transaction is skipped.

        :param tuple[str] allowed_states: The allowed source states for the target state
        :param str target_state: The target state
        :param str state_message: The message to set as `state_message`
        :return: The recordset of transactions whose state was correctly updated
        :rtype: recordset of `payment.transaction`
        """

        def _classify_by_state(_transactions):
            """Classify the transactions according to their current state.

            For each transaction of the current recordset, if:
                - The state is an allowed state: the transaction is flagged as 'to process'.
                - The state is equal to the target state: the transaction is flagged as 'processed'.
                - The state matches none of above: the transaction is flagged as 'in wrong state'.

            :param recordset _transactions: The transactions to classify, as a `payment.transaction`
                                            recordset
            :return: A 3-items tuple of recordsets of classified transactions, in this order:
                     transactions 'to process', 'processed', and 'in wrong state'
            :rtype: tuple(recordset)
            """
            _txs_to_process = _transactions.filtered(lambda _tx: _tx.state in allowed_states)
            _txs_already_processed = _transactions.filtered(lambda _tx: _tx.state == target_state)
            _txs_wrong_state = _transactions - _txs_to_process - _txs_already_processed

            return _txs_to_process, _txs_already_processed, _txs_wrong_state

        txs_to_process, txs_already_processed, txs_wrong_state = _classify_by_state(self)
        for tx in txs_already_processed:
            _logger.info(
                "tried to write tx state with same value (ref: %s, state: %s)",
                tx.reference, tx.state
            )
        for tx in txs_wrong_state:
            logging_values = {
                'reference': tx.reference,
                'tx_state': tx.state,
                'target_state': target_state,
                'allowed_states': allowed_states,
            }
            _logger.warning(
                "tried to write tx state with illegal value (ref: %(reference)s, previous state "
                "%(tx_state)s, target state: %(target_state)s, expected previous state to be in: "
                "%(allowed_states)s)", logging_values
            )
        txs_to_process.write({
            'state': target_state,
            'state_message': state_message,
            'last_state_change': fields.Datetime.now(),
        })
        return txs_to_process

    def _execute_callback(self):
        """ Execute the callbacks defined on the transactions.

        Callbacks that have already been executed are silently ignored. This case can happen when a
        transaction is first authorized before being confirmed, for instance. In this case, both
        status updates try to execute the callback.

        Only successful callbacks are marked as done. This allows callbacks to reschedule themselves
        should the conditions not be met in the present call.

        :return: None
        """
        for tx in self.filtered(lambda t: not t.sudo().callback_is_done):
            # Only use sudo to check, not to execute
            tx_sudo = tx.sudo()
            model_sudo = tx_sudo.callback_model_id
            res_id = tx_sudo.callback_res_id
            method = tx_sudo.callback_method
            callback_hash = tx_sudo.callback_hash
            if not (model_sudo and res_id and method):
                continue  # Skip transactions with unset (or not properly defined) callbacks

            valid_callback_hash = self._generate_callback_hash(model_sudo.id, res_id, method)
            if not consteq(ustr(valid_callback_hash), callback_hash):
                _logger.warning("invalid callback signature for transaction with id %s", tx.id)
                continue  # Ignore tampered callbacks

            record = self.env[model_sudo.model].browse(res_id).exists()
            if not record:
                logging_values = {
                    'model': model_sudo.model,
                    'record_id': res_id,
                    'tx_id': tx.id,
                }
                _logger.warning(
                    "invalid callback record %(model)s.%(record_id)s for transaction with id "
                    "%(tx_id)s", logging_values
                )
                continue  # Ignore invalidated callbacks

            success = getattr(record, method)(tx)  # Execute the callback
            tx_sudo.callback_is_done = success or success is None  # Missing returns are successful

    #=== BUSINESS METHODS - POST-PROCESSING ===#

    def _get_post_processing_values(self):
        """ Return a dict of values used to display the status of the transaction.

        For an acquirer to handle transaction status display, it must override this method and
        return a dict of values. Acquirer-specific values take precedence over those of the dict of
        generic post-processing values.

        The returned dict contains the following entries:
            - provider: The provider of the acquirer
            - reference: The reference of the transaction
            - amount: The rounded amount of the transaction
            - currency_id: The currency of the transaction, as a res.currency id
            - state: The transaction state: draft, pending, authorized, done, cancel or error
            - state_message: The information message about the state
            - is_post_processed: Whether the transaction has already been post-processed
            - landing_route: The route the user is redirected to after the transaction
            - Additional acquirer-specific entries

        Note: self.ensure_one()

        :return: The dict of processing values
        :rtype: dict
        """
        self.ensure_one()

        post_processing_values = {
            'provider': self.provider,
            'reference': self.reference,
            'amount': self.amount,
            'currency_code': self.currency_id.name,
            'state': self.state,
            'state_message': self.state_message,
            'is_post_processed': self.is_post_processed,
            'landing_route': self.landing_route,
        }
        _logger.debug(
            "post-processing values for acquirer with id %s:\n%s",
            self.acquirer_id.id, pprint.pformat(post_processing_values)
        )  # DEBUG level because this can get spammy with transactions in non-final states
        return post_processing_values

    def _finalize_post_processing(self):
        """ Trigger the final post-processing tasks and mark the transactions as post-processed.

        :return: None
        """
        self._reconcile_after_done()
        self._log_received_message()  # 2nd call to link the created account.payment in the chatter
        self.is_post_processed = True

    def _cron_finalize_post_processing(self):
        """ Finalize the post-processing of recently done transactions not handled by the client.

        :return: None
        """
        txs_to_post_process = self
        if not txs_to_post_process:
            # Let the client post-process transactions so that they remain available in the portal
            client_handling_limit_date = datetime.now() - relativedelta.relativedelta(minutes=10)
            # Don't try forever to post-process a transaction that doesn't go through. Set the limit
            # to 4 days because some providers (PayPal) need that much for the payment verification.
            retry_limit_date = datetime.now() - relativedelta.relativedelta(days=4)
            # Retrieve all transactions matching the criteria for post-processing
            txs_to_post_process = self.search([
                ('state', '=', 'done'),
                ('is_post_processed', '=', False),
                '|', ('last_state_change', '<=', client_handling_limit_date),
                     ('operation', '=', 'refund'),
                ('last_state_change', '>=', retry_limit_date),
            ])
        for tx in txs_to_post_process:
            try:
                tx._finalize_post_processing()
                self.env.cr.commit()
            except psycopg2.OperationalError:  # A collision of accounting sequences occurred
                self.env.cr.rollback()  # Rollback and try later
            except Exception as e:
                _logger.exception(
                    "encountered an error while post-processing transaction with id %s:\n%s",
                    tx.id, e
                )
                self.env.cr.rollback()

    def _reconcile_after_done(self):
        """ Post relevant fiscal documents and create missing payments.

        As there is nothing to reconcile for validation transactions, no payment is created for
        them. This is also true for validations with a validity check (transfer of a small amount
        with immediate refund) because validation amounts are not included in payouts.

        :return: None
        """
        # Validate invoices automatically once the transaction is confirmed
        self.invoice_ids.filtered(lambda inv: inv.state == 'draft').action_post()

        # Create and post missing payments for transactions requiring reconciliation
        for tx in self.filtered(lambda t: t.operation != 'validation' and not t.payment_id):
            tx._create_payment()

    def _create_payment(self, **extra_create_values):
        """Create an `account.payment` record for the current transaction.

        If the transaction is linked to some invoices, their reconciliation is done automatically.

        Note: self.ensure_one()

        :param dict extra_create_values: Optional extra create values
        :return: The created payment
        :rtype: recordset of `account.payment`
        """
        self.ensure_one()

        payment_method_line = self.acquirer_id.journal_id.inbound_payment_method_line_ids\
            .filtered(lambda l: l.payment_acquirer_id == self.acquirer_id)
        payment_values = {
            'amount': abs(self.amount),  # A tx may have a negative amount, but a payment must >= 0
            'payment_type': 'inbound' if self.amount > 0 else 'outbound',
            'currency_id': self.currency_id.id,
            'partner_id': self.partner_id.commercial_partner_id.id,
            'partner_type': 'customer',
            'journal_id': self.acquirer_id.journal_id.id,
            'company_id': self.acquirer_id.company_id.id,
            'payment_method_line_id': payment_method_line.id,
            'payment_token_id': self.token_id.id,
            'payment_transaction_id': self.id,
            'ref': self.reference,
            **extra_create_values,
        }
        payment = self.env['account.payment'].create(payment_values)
        payment.action_post()

        # Track the payment to make a one2one.
        self.payment_id = payment

        if self.invoice_ids:
            self.invoice_ids.filtered(lambda inv: inv.state == 'draft').action_post()

            (payment.line_ids + self.invoice_ids.line_ids).filtered(
                lambda line: line.account_id == payment.destination_account_id
                and not line.reconciled
            ).reconcile()

        return payment

    #=== BUSINESS METHODS - LOGGING ===#

    def _log_sent_message(self):
        """ Log in the chatter of relevant documents that the transactions have been initiated.

        :return: None
        """
        for tx in self:
            message = tx._get_sent_message()
            tx._log_message_on_linked_documents(message)

    def _log_received_message(self):
        """ Log in the chatter of relevant documents that the transactions have been received.

        A transaction is 'received' when a response is received from the provider of the acquirer
        handling the transaction.

        :return: None
        """
        for tx in self:
            message = tx._get_received_message()
            tx._log_message_on_linked_documents(message)

    def _log_message_on_linked_documents(self, message):
        """ Log a message on the payment and the invoices linked to the transaction.

        For a module to implement payments and link documents to a transaction, it must override
        this method and call super, then log the message on documents linked to the transaction.

        Note: self.ensure_one()

        :param str message: The message to be logged
        :return: None
        """
        self.ensure_one()
        if self.source_transaction_id.payment_id:
            self.source_transaction_id.payment_id.message_post(body=message)
            for invoice in self.source_transaction_id.invoice_ids:
                invoice.message_post(body=message)
        for invoice in self.invoice_ids:
            invoice.message_post(body=message)

    #=== BUSINESS METHODS - GETTERS ===#

    def _get_sent_message(self):
        """ Return the message stating that the transaction has been requested.

        Note: self.ensure_one()

        :return: The 'transaction sent' message
        :rtype: str
        """
        self.ensure_one()

        # Choose the message based on the payment flow
        if self.operation in ('online_redirect', 'online_direct'):
            message = _(
                "A transaction with reference %(ref)s has been initiated (%(acq_name)s).",
                ref=self.reference, acq_name=self.acquirer_id.name
            )
        elif self.operation == 'refund':
            formatted_amount = format_amount(self.env, -self.amount, self.currency_id)
            message = _(
                "A refund request of %(amount)s has been sent. The payment will be created soon. "
                "Refund transaction reference: %(ref)s (%(acq_name)s).",
                amount=formatted_amount, ref=self.reference, acq_name=self.acquirer_id.name
            )
        else:  # 'online_token'
            message = _(
                "A transaction with reference %(ref)s has been initiated using the payment method "
                "%(token_name)s (%(acq_name)s).",
                ref=self.reference, token_name=self.token_id.name, acq_name=self.acquirer_id.name
            )
        return message

    def _get_received_message(self):
        """ Return the message stating that the transaction has been received by the provider.

        Note: self.ensure_one()
        """
        self.ensure_one()

        formatted_amount = format_amount(self.env, self.amount, self.currency_id)
        if self.state == 'pending':
            message = _(
                "The transaction with reference %(ref)s for %(amount)s is pending (%(acq_name)s).",
                ref=self.reference, amount=formatted_amount, acq_name=self.acquirer_id.name
            )
        elif self.state == 'authorized':
            message = _(
                "The transaction with reference %(ref)s for %(amount)s has been authorized "
                "(%(acq_name)s).", ref=self.reference, amount=formatted_amount,
                acq_name=self.acquirer_id.name
            )
        elif self.state == 'done':
            message = _(
                "The transaction with reference %(ref)s for %(amount)s has been confirmed "
                "(%(acq_name)s).", ref=self.reference, amount=formatted_amount,
                acq_name=self.acquirer_id.name
            )
            if self.payment_id:
                message += "<br />" + _(
                    "The related payment is posted: %s",
                    self.payment_id._get_payment_chatter_link()
                )
        elif self.state == 'error':
            message = _(
                "The transaction with reference %(ref)s for %(amount)s encountered an error"
                " (%(acq_name)s).",
                ref=self.reference, amount=formatted_amount, acq_name=self.acquirer_id.name
            )
            if self.state_message:
                message += "<br />" + _("Error: %s", self.state_message)
        else:
            message = _(
                "The transaction with reference %(ref)s for %(amount)s is canceled (%(acq_name)s).",
                ref=self.reference, amount=formatted_amount, acq_name=self.acquirer_id.name
            )
            if self.state_message:
                message += "<br />" + _("Reason: %s", self.state_message)
        return message

    def _get_last(self):
        """ Return the last transaction of the recordset.

        :return: The last transaction of the recordset, sorted by id
        :rtype: recordset of `payment.transaction`
        """
        return self.filtered(lambda t: t.state != 'draft').sorted()[:1]

```

## File: models\res_company.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ResCompany(models.Model):
    _inherit = 'res.company'

    payment_acquirer_onboarding_state = fields.Selection(
        string="State of the onboarding payment acquirer step",
        selection=[('not_done', "Not done"), ('just_done', "Just done"), ('done', "Done")],
        default='not_done')
    payment_onboarding_payment_method = fields.Selection(
        string="Selected onboarding payment method",
        selection=[
            ('paypal', "PayPal"),
            ('stripe', "Stripe"),
            ('manual', "Manual"),
            ('other', "Other"),
        ])

    @api.model
    def action_open_payment_onboarding_payment_acquirer(self):
        """ Called by onboarding panel above the customer invoice list. """
        # TODO remove me in master.
        #  This action is never used anywhere because the onboarding step's method is overridden in
        #  website_sale to call action_open_website_sale_onboarding_payment_acquirer instead.
        # Fail if there are no existing accounts
        self.env.company.get_chart_of_accounts_or_fail()

        action = self.env['ir.actions.actions']._for_xml_id(
            'payment.action_open_payment_onboarding_payment_acquirer_wizard'
        )
        return action

    def _run_payment_onboarding_step(self, menu_id):
        """ Install the suggested payment modules and configure the acquirers.

        It's checked that the current company has a Chart of Account.

        :param int menu_id: The menu from which the user started the onboarding step, as an
                            `ir.ui.menu` id
        :return: The action returned by `action_stripe_connect_account`
        :rtype: dict
        """
        self.env.company.get_chart_of_accounts_or_fail()

        self._install_modules(['payment_stripe', 'account_payment'])

        # Create a new env including the freshly installed module(s)
        new_env = api.Environment(self.env.cr, self.env.uid, self.env.context)

        # Configure Stripe
        default_journal = new_env['account.journal'].search(
            [('type', '=', 'bank'), ('company_id', '=', new_env.company.id)], limit=1
        )

        stripe_acquirer = new_env['payment.acquirer'].search(
            [('company_id', '=', self.env.company.id), ('name', '=', 'Stripe')], limit=1
        )
        if not stripe_acquirer:
            base_acquirer = self.env.ref('payment.payment_acquirer_stripe')
            # Use sudo to access payment acquirer record that can be in different company.
            stripe_acquirer = base_acquirer.sudo().copy(default={'company_id': self.env.company.id})
            stripe_acquirer.company_id = self.env.company.id
        stripe_acquirer.journal_id = stripe_acquirer.journal_id or default_journal

        return stripe_acquirer.action_stripe_connect_account(menu_id=menu_id)

    def _install_modules(self, module_names):
        modules_sudo = self.env['ir.module.module'].sudo().search([('name', 'in', module_names)])
        STATES = ['installed', 'to install', 'to upgrade']
        modules_sudo.filtered(lambda m: m.state not in STATES).button_immediate_install()

    def _mark_payment_onboarding_step_as_done(self):
        """ Mark the payment onboarding step as done.

        :return: None
        """
        self.set_onboarding_step_done('payment_acquirer_onboarding_state')

    def get_account_invoice_onboarding_steps_states_names(self):
        """ Override of account. """
        steps = super().get_account_invoice_onboarding_steps_states_names()
        return steps + ['payment_acquirer_onboarding_state']

```

## File: models\res_partner.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ResPartner(models.Model):
    _inherit = 'res.partner'

    payment_token_ids = fields.One2many(
        string="Payment Tokens", comodel_name='payment.token', inverse_name='partner_id')
    payment_token_count = fields.Integer(
        string="Payment Token Count", compute='_compute_payment_token_count')

    @api.depends('payment_token_ids')
    def _compute_payment_token_count(self):
        payments_data = self.env['payment.token'].read_group(
            [('partner_id', 'in', self.ids)], ['partner_id'], ['partner_id']
        )
        partners_data = {payment_data['partner_id'][0]: payment_data['partner_id_count']
                         for payment_data in payments_data}
        for partner in self:
            partner.payment_token_count = partners_data.get(partner.id, 0)

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_payment
from . import account_payment_method
from . import account_move
from . import ir_http
from . import ir_ui_view
from . import payment_acquirer
from . import payment_icon
from . import payment_token
from . import payment_transaction
from . import res_company
from . import res_partner
from . import account_journal

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
payment_acquirer_onboarding_wizard,payment.acquirer.onboarding.wizard,model_payment_acquirer_onboarding_wizard,base.group_system,1,1,1,0
payment_acquirer_system,payment.acquirer.system,model_payment_acquirer,base.group_system,1,1,1,1
payment_icon_all,payment.icon.all,model_payment_icon,,1,0,0,0
payment_icon_system,payment.icon.system,model_payment_icon,base.group_system,1,1,1,1
payment_link_wizard,payment.link.wizard,model_payment_link_wizard,account.group_account_invoice,1,1,1,0
payment_refund_wizard,payment.refund.wizard,model_payment_refund_wizard,account.group_account_invoice,1,1,1,0
payment_token_all,payment.token.all,model_payment_token,,1,0,0,0
payment_token_portal,payment.token.portal,model_payment_token,base.group_portal,1,1,1,1
payment_token_system,payment.token.system,model_payment_token,base.group_system,1,1,1,1
payment_token_user,payment.token.user,model_payment_token,base.group_user,1,1,1,1
payment_transaction_all,payment.transaction.all,model_payment_transaction,,1,0,0,0
payment_transaction_system,payment.transaction.system,model_payment_transaction,base.group_system,1,1,1,1
payment_transaction_user,payment.transaction.user,model_payment_transaction,base.group_user,1,1,1,0

```

## File: security\payment_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <!-- Acquirers -->

    <record id="payment_acquirer_company_rule" model="ir.rule">
        <field name="name">Access acquirers in own companies only</field>
        <field name="model_id" ref="payment.model_payment_acquirer"/>
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>

    <!-- Transactions -->

    <record id="payment_transaction_user_rule" model="ir.rule">
        <field name="name">Access own transactions only</field>
        <field name="model_id" ref="payment.model_payment_transaction"/>
        <field name="domain_force">['|', ('partner_id', '=', False), ('partner_id', '=', user.partner_id.id)]</field>
        <field name="groups" eval="[(4, ref('base.group_user')), (4, ref('base.group_portal')), (4, ref('base.group_public'))]"/>
    </record>

    <record id="payment_transaction_billing_rule" model="ir.rule">
        <field name="name">Access every transaction</field>
        <field name="model_id" ref="payment.model_payment_transaction"/>
        <!-- Reset the domain defined by payment.transaction_user_rule -->
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4, ref('account.group_account_invoice'))]"/>
    </record>

    <record id="transaction_company_rule" model="ir.rule">
        <field name="name">Access transactions in own companies only</field>
        <field name="model_id" ref="payment.model_payment_transaction"/>
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>

    <!-- Tokens -->

    <record id="payment_token_user_rule" model="ir.rule">
        <field name="name">Access only tokens belonging to commercial partner</field>
        <field name="model_id" ref="payment.model_payment_token"/>
        <field name="domain_force">[('partner_id', 'child_of', user.partner_id.commercial_partner_id.id)]</field>
        <field name="groups" eval="[(4, ref('base.group_user')), (4, ref('base.group_portal')), (4, ref('base.group_public'))]"/>
    </record>

    <record id="payment_token_billing_rule" model="ir.rule">
        <field name="name">Access every token</field>
        <field name="model_id" ref="payment.model_payment_token"/>
        <!-- Reset the domain defined by payment.token_user_rule -->
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4, ref('account.group_account_invoice'))]"/>
    </record>

    <record id="payment_token_company_rule" model="ir.rule">
        <field name="name">Access tokens in own companies only</field>
        <field name="model_id" ref="payment.model_payment_token"/>
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>

</odoo>

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="98.616%"><stop offset="0%" stop-color="#797C79"/><stop offset="100%" stop-color="#545554"/></linearGradient><path id="d" d="M19.25 34.621C20.983 35.541 22.9 36 25 36c4 0 7-1.667 9-5h15.036v-3.212a.342.342 0 0 0-.339-.343H35c.347-.502.423-1.416.228-2.742h13.808c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742V34.62zm29.447 12.934a.342.342 0 0 0 .339-.343V37.5H21.964v9.712c0 .188.152.343.339.343h26.394zm-18.614-5.713v2.285a.683.683 0 0 1-.677.685h-4.062a.683.683 0 0 1-.677-.685v-2.285c0-.377.304-.686.677-.686h4.062c.373 0 .677.309.677.686zm10.834 0v2.285a.683.683 0 0 1-.677.685h-7.674a.683.683 0 0 1-.677-.685v-2.285c0-.377.305-.686.677-.686h7.674c.372 0 .677.309.677.686zM22.874 19.77c0 2.517 8.146 1.879 8.146 6.663 0 2.292-1.783 4.25-4.681 4.657v1.897c0 .265-.237.48-.53.48h-1.763c-.292 0-.53-.215-.53-.48v-1.928c-1.74-.268-3.293-.997-4.353-1.917a.447.447 0 0 1-.058-.634l1.34-1.625a.566.566 0 0 1 .757-.086c1.098.8 2.517 1.435 3.873 1.435 1.58 0 2.299-.853 2.299-1.646 0-2.342-8.147-1.834-8.147-6.772 0-2.107 1.705-3.811 4.29-4.342V13.48c0-.265.237-.48.53-.48h1.763c.292 0 .529.215.529.48v1.888c1.418.149 2.954.653 4.025 1.511.184.148.23.391.113.587l-1.038 1.74c-.152.255-.516.33-.775.161-.985-.642-2.193-1.132-3.371-1.132-1.47 0-2.42.603-2.42 1.535z"/><path id="e" d="M19.25 32.621C20.983 33.541 22.9 34 25 34c4 0 7-1.667 9-5h15.036v-3.212a.342.342 0 0 0-.339-.343H35c.347-.502.423-1.416.228-2.742h13.808c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742V32.62zm29.447 12.934a.342.342 0 0 0 .339-.343V35.5H21.964v9.712c0 .188.152.343.339.343h26.394zm-18.614-5.713v2.285a.683.683 0 0 1-.677.685h-4.062a.683.683 0 0 1-.677-.685v-2.285c0-.377.304-.686.677-.686h4.062c.373 0 .677.309.677.686zm10.834 0v2.285a.683.683 0 0 1-.677.685h-7.674a.683.683 0 0 1-.677-.685v-2.285c0-.377.305-.686.677-.686h7.674c.372 0 .677.309.677.686zM22.874 17.77c0 2.517 8.146 1.879 8.146 6.663 0 2.292-1.783 4.25-4.681 4.657v1.897c0 .265-.237.48-.53.48h-1.763c-.292 0-.53-.215-.53-.48v-1.928c-1.74-.268-3.293-.997-4.353-1.917a.447.447 0 0 1-.058-.634l1.34-1.625a.566.566 0 0 1 .757-.086c1.098.8 2.517 1.435 3.873 1.435 1.58 0 2.299-.853 2.299-1.646 0-2.342-8.147-1.834-8.147-6.772 0-2.107 1.705-3.811 4.29-4.342V11.48c0-.265.237-.48.53-.48h1.763c.292 0 .529.215.529.48v1.888c1.418.149 2.954.653 4.025 1.511.184.148.23.391.113.587l-1.038 1.74c-.152.255-.516.33-.775.161-.985-.642-2.193-1.132-3.371-1.132-1.47 0-2.42.603-2.42 1.535z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M4 69c-2 0-4-1-4-4V36.095l23.6-24.837 5.552 6.125-3.147 3.581 4.02 6.176 5.257-4.413L48 23l3.377 2.017v21.868L32.142 69H4z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: static\lib\jquery.payment\jquery.payment.js

```javascript
// Generated by CoffeeScript 1.7.1
(function() {
  var $, cardFromNumber, cardFromType, cards, defaultFormat, formatBackCardNumber, formatBackExpiry, formatCardNumber, formatExpiry, formatForwardExpiry, formatForwardSlashAndSpace, hasTextSelected, luhnCheck, reFormatCVC, reFormatCardNumber, reFormatExpiry, reFormatNumeric, replaceFullWidthChars, restrictCVC, restrictCardNumber, restrictExpiry, restrictNumeric, safeVal, setCardType,
    __slice = [].slice,
    __indexOf = [].indexOf || function(item) { for (var i = 0, l = this.length; i < l; i++) { if (i in this && this[i] === item) return i; } return -1; };

  $ = window.jQuery || window.Zepto || window.$;

  $.payment = {};

  $.payment.fn = {};

  $.fn.payment = function() {
    var args, method;
    method = arguments[0], args = 2 <= arguments.length ? __slice.call(arguments, 1) : [];
    return $.payment.fn[method].apply(this, args);
  };

  defaultFormat = /(\d{1,4})/g;

  $.payment.cards = cards = [
    {
      type: 'maestro',
      patterns: [5018, 502, 503, 506, 56, 58, 639, 6220, 67],
      format: defaultFormat,
      length: [12, 13, 14, 15, 16, 17, 18, 19],
      cvcLength: [3],
      luhn: true
    }, {
      type: 'forbrugsforeningen',
      patterns: [600],
      format: defaultFormat,
      length: [16],
      cvcLength: [3],
      luhn: true
    }, {
      type: 'dankort',
      patterns: [5019],
      format: defaultFormat,
      length: [16],
      cvcLength: [3],
      luhn: true
    }, {
      type: 'visa',
      patterns: [4],
      format: defaultFormat,
      length: [13, 16],
      cvcLength: [3],
      luhn: true
    }, {
      type: 'mastercard',
      patterns: [51, 52, 53, 54, 55, 22, 23, 24, 25, 26, 27],
      format: defaultFormat,
      length: [16],
      cvcLength: [3],
      luhn: true
    }, {
      type: 'amex',
      patterns: [34, 37],
      format: /(\d{1,4})(\d{1,6})?(\d{1,5})?/,
      length: [15],
      cvcLength: [3, 4],
      luhn: true
    }, {
      type: 'dinersclub',
      patterns: [30, 36, 38, 39],
      format: /(\d{1,4})(\d{1,6})?(\d{1,4})?/,
      length: [14],
      cvcLength: [3],
      luhn: true
    }, {
      type: 'discover',
      patterns: [60, 64, 65, 622],
      format: defaultFormat,
      length: [16],
      cvcLength: [3],
      luhn: true
    }, {
      type: 'unionpay',
      patterns: [62, 88],
      format: defaultFormat,
      length: [16, 17, 18, 19],
      cvcLength: [3],
      luhn: false
    }, {
      type: 'jcb',
      patterns: [35],
      format: defaultFormat,
      length: [16],
      cvcLength: [3],
      luhn: true
    }
  ];

  cardFromNumber = function(num) {
    var card, p, pattern, _i, _j, _len, _len1, _ref;
    num = (num + '').replace(/\D/g, '');
    for (_i = 0, _len = cards.length; _i < _len; _i++) {
      card = cards[_i];
      _ref = card.patterns;
      for (_j = 0, _len1 = _ref.length; _j < _len1; _j++) {
        pattern = _ref[_j];
        p = pattern + '';
        if (num.substr(0, p.length) === p) {
          return card;
        }
      }
    }
  };

  cardFromType = function(type) {
    var card, _i, _len;
    for (_i = 0, _len = cards.length; _i < _len; _i++) {
      card = cards[_i];
      if (card.type === type) {
        return card;
      }
    }
  };

  luhnCheck = function(num) {
    var digit, digits, odd, sum, _i, _len;
    odd = true;
    sum = 0;
    digits = (num + '').split('').reverse();
    for (_i = 0, _len = digits.length; _i < _len; _i++) {
      digit = digits[_i];
      digit = parseInt(digit, 10);
      if ((odd = !odd)) {
        digit *= 2;
      }
      if (digit > 9) {
        digit -= 9;
      }
      sum += digit;
    }
    return sum % 10 === 0;
  };

  hasTextSelected = function($target) {
    var _ref;
    if (($target.prop('selectionStart') != null) && $target.prop('selectionStart') !== $target.prop('selectionEnd')) {
      return true;
    }
    if ((typeof document !== "undefined" && document !== null ? (_ref = document.selection) != null ? _ref.createRange : void 0 : void 0) != null) {
      if (document.selection.createRange().text) {
        return true;
      }
    }
    return false;
  };

  safeVal = function(value, $target) {
    var currPair, cursor, digit, error, last, prevPair;
    try {
      cursor = $target.prop('selectionStart');
    } catch (_error) {
      error = _error;
      cursor = null;
    }
    last = $target.val();
    $target.val(value);
    if (cursor !== null && $target.is(":focus")) {
      if (cursor === last.length) {
        cursor = value.length;
      }
      if (last !== value) {
        prevPair = last.slice(cursor - 1, +cursor + 1 || 9e9);
        currPair = value.slice(cursor - 1, +cursor + 1 || 9e9);
        digit = value[cursor];
        if (/\d/.test(digit) && prevPair === ("" + digit + " ") && currPair === (" " + digit)) {
          cursor = cursor + 1;
        }
      }
      $target.prop('selectionStart', cursor);
      return $target.prop('selectionEnd', cursor);
    }
  };

  replaceFullWidthChars = function(str) {
    var chars, chr, fullWidth, halfWidth, idx, value, _i, _len;
    if (str == null) {
      str = '';
    }
    fullWidth = '\uff10\uff11\uff12\uff13\uff14\uff15\uff16\uff17\uff18\uff19';
    halfWidth = '0123456789';
    value = '';
    chars = str.split('');
    for (_i = 0, _len = chars.length; _i < _len; _i++) {
      chr = chars[_i];
      idx = fullWidth.indexOf(chr);
      if (idx > -1) {
        chr = halfWidth[idx];
      }
      value += chr;
    }
    return value;
  };

  reFormatNumeric = function(e) {
    var $target;
    $target = $(e.currentTarget);
    return setTimeout(function() {
      var value;
      value = $target.val();
      value = replaceFullWidthChars(value);
      value = value.replace(/\D/g, '');
      return safeVal(value, $target);
    });
  };

  reFormatCardNumber = function(e) {
    var $target;
    $target = $(e.currentTarget);
    return setTimeout(function() {
      var value;
      value = $target.val();
      value = replaceFullWidthChars(value);
      value = $.payment.formatCardNumber(value);
      return safeVal(value, $target);
    });
  };

  formatCardNumber = function(e) {
    var $target, card, digit, length, re, upperLength, value;
    digit = String.fromCharCode(e.which);
    if (!/^\d+$/.test(digit)) {
      return;
    }
    $target = $(e.currentTarget);
    value = $target.val();
    card = cardFromNumber(value + digit);
    length = (value.replace(/\D/g, '') + digit).length;
    upperLength = 16;
    if (card) {
      upperLength = card.length[card.length.length - 1];
    }
    if (length >= upperLength) {
      return;
    }
    if (($target.prop('selectionStart') != null) && $target.prop('selectionStart') !== value.length) {
      return;
    }
    if (card && card.type === 'amex') {
      re = /^(\d{4}|\d{4}\s\d{6})$/;
    } else {
      re = /(?:^|\s)(\d{4})$/;
    }
    if (re.test(value)) {
      e.preventDefault();
      return setTimeout(function() {
        return $target.val(value + ' ' + digit);
      });
    } else if (re.test(value + digit)) {
      e.preventDefault();
      return setTimeout(function() {
        return $target.val(value + digit + ' ');
      });
    }
  };

  formatBackCardNumber = function(e) {
    var $target, value;
    $target = $(e.currentTarget);
    value = $target.val();
    if (e.which !== 8) {
      return;
    }
    if (($target.prop('selectionStart') != null) && $target.prop('selectionStart') !== value.length) {
      return;
    }
    if (/\d\s$/.test(value)) {
      e.preventDefault();
      return setTimeout(function() {
        return $target.val(value.replace(/\d\s$/, ''));
      });
    } else if (/\s\d?$/.test(value)) {
      e.preventDefault();
      return setTimeout(function() {
        return $target.val(value.replace(/\d$/, ''));
      });
    }
  };

  reFormatExpiry = function(e) {
    var $target;
    $target = $(e.currentTarget);
    return setTimeout(function() {
      var value;
      value = $target.val();
      value = replaceFullWidthChars(value);
      value = $.payment.formatExpiry(value);
      return safeVal(value, $target);
    });
  };

  formatExpiry = function(e) {
    var $target, digit, val;
    digit = String.fromCharCode(e.which);
    if (!/^\d+$/.test(digit)) {
      return;
    }
    $target = $(e.currentTarget);
    val = $target.val() + digit;
    if (/^\d$/.test(val) && (val !== '0' && val !== '1')) {
      e.preventDefault();
      return setTimeout(function() {
        return $target.val("0" + val + " / ");
      });
    } else if (/^\d\d$/.test(val)) {
      e.preventDefault();
      return setTimeout(function() {
        var m1, m2;
        m1 = parseInt(val[0], 10);
        m2 = parseInt(val[1], 10);
        if (m2 > 2 && m1 !== 0) {
          return $target.val("0" + m1 + " / " + m2);
        } else {
          return $target.val("" + val + " / ");
        }
      });
    }
  };

  formatForwardExpiry = function(e) {
    var $target, digit, val;
    digit = String.fromCharCode(e.which);
    if (!/^\d+$/.test(digit)) {
      return;
    }
    $target = $(e.currentTarget);
    val = $target.val();
    if (/^\d\d$/.test(val)) {
      return $target.val("" + val + " / ");
    }
  };

  formatForwardSlashAndSpace = function(e) {
    var $target, val, which;
    which = String.fromCharCode(e.which);
    if (!(which === '/' || which === ' ')) {
      return;
    }
    $target = $(e.currentTarget);
    val = $target.val();
    if (/^\d$/.test(val) && val !== '0') {
      return $target.val("0" + val + " / ");
    }
  };

  formatBackExpiry = function(e) {
    var $target, value;
    $target = $(e.currentTarget);
    value = $target.val();
    if (e.which !== 8) {
      return;
    }
    if (($target.prop('selectionStart') != null) && $target.prop('selectionStart') !== value.length) {
      return;
    }
    if (/\d\s\/\s$/.test(value)) {
      e.preventDefault();
      return setTimeout(function() {
        return $target.val(value.replace(/\d\s\/\s$/, ''));
      });
    }
  };

  reFormatCVC = function(e) {
    var $target;
    $target = $(e.currentTarget);
    return setTimeout(function() {
      var value;
      value = $target.val();
      value = replaceFullWidthChars(value);
      value = value.replace(/\D/g, '').slice(0, 4);
      return safeVal(value, $target);
    });
  };

  restrictNumeric = function(e) {
    var input;
    if (e.metaKey || e.ctrlKey) {
      return true;
    }
    if (e.which === 32) {
      return false;
    }
    if (e.which === 0) {
      return true;
    }
    if (e.which < 33) {
      return true;
    }
    input = String.fromCharCode(e.which);
    return !!/[\d\s]/.test(input);
  };

  restrictCardNumber = function(e) {
    var $target, card, digit, value;
    $target = $(e.currentTarget);
    digit = String.fromCharCode(e.which);
    if (!/^\d+$/.test(digit)) {
      return;
    }
    if (hasTextSelected($target)) {
      return;
    }
    value = ($target.val() + digit).replace(/\D/g, '');
    card = cardFromNumber(value);
    if (card) {
      return value.length <= card.length[card.length.length - 1];
    } else {
      return value.length <= 16;
    }
  };

  restrictExpiry = function(e) {
    var $target, digit, value;
    $target = $(e.currentTarget);
    digit = String.fromCharCode(e.which);
    if (!/^\d+$/.test(digit)) {
      return;
    }
    if (hasTextSelected($target)) {
      return;
    }
    value = $target.val() + digit;
    value = value.replace(/\D/g, '');
    if (value.length > 6) {
      return false;
    }
  };

  restrictCVC = function(e) {
    var $target, digit, val;
    $target = $(e.currentTarget);
    digit = String.fromCharCode(e.which);
    if (!/^\d+$/.test(digit)) {
      return;
    }
    if (hasTextSelected($target)) {
      return;
    }
    val = $target.val() + digit;
    return val.length <= 4;
  };

  setCardType = function(e) {
    var $target, allTypes, card, cardType, val;
    $target = $(e.currentTarget);
    val = $target.val();
    cardType = $.payment.cardType(val) || 'unknown';
    if (!$target.hasClass(cardType)) {
      allTypes = (function() {
        var _i, _len, _results;
        _results = [];
        for (_i = 0, _len = cards.length; _i < _len; _i++) {
          card = cards[_i];
          _results.push(card.type);
        }
        return _results;
      })();
      $target.removeClass('unknown');
      $target.removeClass(allTypes.join(' '));
      $target.addClass(cardType);
      $target.toggleClass('identified', cardType !== 'unknown');
      return $target.trigger('payment.cardType', cardType);
    }
  };

  $.payment.fn.formatCardCVC = function() {
    this.on('keypress', restrictNumeric);
    this.on('keypress', restrictCVC);
    this.on('paste', reFormatCVC);
    this.on('change', reFormatCVC);
    this.on('input', reFormatCVC);
    return this;
  };

  $.payment.fn.formatCardExpiry = function() {
    this.on('keypress', restrictNumeric);
    this.on('keypress', restrictExpiry);
    this.on('keypress', formatExpiry);
    this.on('keypress', formatForwardSlashAndSpace);
    this.on('keypress', formatForwardExpiry);
    this.on('keydown', formatBackExpiry);
    this.on('change', reFormatExpiry);
    this.on('input', reFormatExpiry);
    return this;
  };

  $.payment.fn.formatCardNumber = function() {
    this.on('keypress', restrictNumeric);
    this.on('keypress', restrictCardNumber);
    this.on('keypress', formatCardNumber);
    this.on('keydown', formatBackCardNumber);
    this.on('keyup', setCardType);
    this.on('paste', reFormatCardNumber);
    this.on('change', reFormatCardNumber);
    this.on('input', reFormatCardNumber);
    this.on('input', setCardType);
    return this;
  };

  $.payment.fn.restrictNumeric = function() {
    this.on('keypress', restrictNumeric);
    this.on('paste', reFormatNumeric);
    this.on('change', reFormatNumeric);
    this.on('input', reFormatNumeric);
    return this;
  };

  $.payment.fn.cardExpiryVal = function() {
    return $.payment.cardExpiryVal($(this).val());
  };

  $.payment.cardExpiryVal = function(value) {
    var month, prefix, year, _ref;
    _ref = value.split(/[\s\/]+/, 2), month = _ref[0], year = _ref[1];
    if ((year != null ? year.length : void 0) === 2 && /^\d+$/.test(year)) {
      prefix = (new Date).getFullYear();
      prefix = prefix.toString().slice(0, 2);
      year = prefix + year;
    }
    month = parseInt(month, 10);
    year = parseInt(year, 10);
    return {
      month: month,
      year: year
    };
  };

  $.payment.validateCardNumber = function(num) {
    var card, _ref;
    num = (num + '').replace(/\s+|-/g, '');
    if (!/^\d+$/.test(num)) {
      return false;
    }
    card = cardFromNumber(num);
    if (!card) {
      return false;
    }
    return (_ref = num.length, __indexOf.call(card.length, _ref) >= 0) && (card.luhn === false || luhnCheck(num));
  };

  $.payment.validateCardExpiry = function(month, year) {
    var currentTime, expiry, _ref;
    if (typeof month === 'object' && 'month' in month) {
      _ref = month, month = _ref.month, year = _ref.year;
    }
    if (!(month && year)) {
      return false;
    }
    month = $.trim(month);
    year = $.trim(year);
    if (!/^\d+$/.test(month)) {
      return false;
    }
    if (!/^\d+$/.test(year)) {
      return false;
    }
    if (!((1 <= month && month <= 12))) {
      return false;
    }
    if (year.length === 2) {
      if (year < 70) {
        year = "20" + year;
      } else {
        year = "19" + year;
      }
    }
    if (year.length !== 4) {
      return false;
    }
    expiry = new Date(year, month);
    currentTime = new Date;
    expiry.setMonth(expiry.getMonth() - 1);
    expiry.setMonth(expiry.getMonth() + 1, 1);
    return expiry > currentTime;
  };

  $.payment.validateCardCVC = function(cvc, type) {
    var card, _ref;
    cvc = $.trim(cvc);
    if (!/^\d+$/.test(cvc)) {
      return false;
    }
    card = cardFromType(type);
    if (card != null) {
      return _ref = cvc.length, __indexOf.call(card.cvcLength, _ref) >= 0;
    } else {
      return cvc.length >= 3 && cvc.length <= 4;
    }
  };

  $.payment.cardType = function(num) {
    var _ref;
    if (!num) {
      return null;
    }
    return ((_ref = cardFromNumber(num)) != null ? _ref.type : void 0) || null;
  };

  $.payment.formatCardNumber = function(num) {
    var card, groups, upperLength, _ref;
    num = num.replace(/\D/g, '');
    card = cardFromNumber(num);
    if (!card) {
      return num;
    }
    upperLength = card.length[card.length.length - 1];
    num = num.slice(0, upperLength);
    if (card.format.global) {
      return (_ref = num.match(card.format)) != null ? _ref.join(' ') : void 0;
    } else {
      groups = card.format.exec(num);
      if (groups == null) {
        return;
      }
      groups.shift();
      groups = $.grep(groups, function(n) {
        return n;
      });
      return groups.join(' ');
    }
  };

  $.payment.formatExpiry = function(expiry) {
    var mon, parts, sep, year;
    parts = expiry.match(/^\D*(\d{1,2})(\D+)?(\d{1,4})?/);
    if (!parts) {
      return '';
    }
    mon = parts[1] || '';
    sep = parts[2] || '';
    year = parts[3] || '';
    if (year.length > 0) {
      sep = ' / ';
    } else if (sep === ' /') {
      mon = mon.substring(0, 1);
      sep = '';
    } else if (mon.length === 2 || sep.length > 0) {
      sep = ' / ';
    } else if (mon.length === 1 && (mon !== '0' && mon !== '1')) {
      mon = "0" + mon;
      sep = ' / ';
    }
    return mon + sep + year;
  };

}).call(this);

```

## File: static\src\js\checkout_form.js

```javascript
odoo.define('payment.checkout_form', require => {
    'use strict';

    const publicWidget = require('web.public.widget');

    const paymentFormMixin = require('payment.payment_form_mixin');

    publicWidget.registry.PaymentCheckoutForm = publicWidget.Widget.extend(paymentFormMixin, {
        selector: 'form[name="o_payment_checkout"]',
        events: Object.assign({}, publicWidget.Widget.prototype.events, {
                'click div[name="o_payment_option_card"]': '_onClickPaymentOption',
                'click a[name="o_payment_icon_more"]': '_onClickMorePaymentIcons',
                'click a[name="o_payment_icon_less"]': '_onClickLessPaymentIcons',
                'click button[name="o_payment_submit_button"]': '_onClickPay',
                'submit': '_onSubmit',
            }),

        /**
         * @constructor
         */
        init: function () {
            const preventDoubleClick = handlerMethod => {
                return _.debounce(handlerMethod, 500, true);
            };
            this._super(...arguments);
            // Prevent double-clicks and browser glitches on all inputs
            this._onClickLessPaymentIcons = preventDoubleClick(this._onClickLessPaymentIcons);
            this._onClickMorePaymentIcons = preventDoubleClick(this._onClickMorePaymentIcons);
            this._onClickPay = preventDoubleClick(this._onClickPay);
            this._onClickPaymentOption = preventDoubleClick(this._onClickPaymentOption);
            this._onSubmit = preventDoubleClick(this._onSubmit);
        },

        //--------------------------------------------------------------------------
        // Handlers
        //--------------------------------------------------------------------------

        /**
         * Handle a direct payment, a payment with redirection, or a payment by token.
         *
         * Called when clicking on the 'Pay' button or when submitting the form.
         *
         * @private
         * @param {Event} ev
         * @return {undefined}
         */
        _onClickPay: async function (ev) {
            ev.stopPropagation();
            ev.preventDefault();

            // Check that the user has selected a payment option
            const $checkedRadios = this.$('input[name="o_payment_radio"]:checked');
            if (!this._ensureRadioIsChecked($checkedRadios)) {
                return;
            }
            const checkedRadio = $checkedRadios[0];

            // Extract contextual values from the radio button
            const provider = this._getProviderFromRadio(checkedRadio);
            const paymentOptionId = this._getPaymentOptionIdFromRadio(checkedRadio);
            const flow = this._getPaymentFlowFromRadio(checkedRadio);

            // Update the tx context with the value of the "Save my payment details" checkbox
            if (flow !== 'token') {
                const $tokenizeCheckbox = this.$(
                    `#o_payment_acquirer_inline_form_${paymentOptionId}` // Only match acq. radios
                ).find('input[name="o_payment_save_as_token"]');
                this.txContext.tokenizationRequested = $tokenizeCheckbox.length === 1
                    && $tokenizeCheckbox[0].checked;
            } else {
                this.txContext.tokenizationRequested = false;
            }

            // Make the payment
            this._hideError(); // Don't keep the error displayed if the user is going through 3DS2
            this._disableButton(true); // Disable until it is needed again
            $('body').block({
                message: false,
                overlayCSS: {backgroundColor: "#000", opacity: 0, zIndex: 1050},
            });
            this._processPayment(provider, paymentOptionId, flow);
        },

        /**
         * Delegate the handling of the payment request to `_onClickPay`.
         *
         * Called when submitting the form (e.g. through the Return key).
         *
         * @private
         * @param {Event} ev
         * @return {undefined}
         */
        _onSubmit: function (ev) {
            ev.stopPropagation();
            ev.preventDefault();

            this._onClickPay(ev);
        },

    });
    return publicWidget.registry.PaymentCheckoutForm;
});

```

## File: static\src\js\manage_form.js

```javascript
odoo.define('payment.manage_form', require => {
    'use strict';

    const core = require('web.core');
    const publicWidget = require('web.public.widget');
    const Dialog = require('web.Dialog');

    const paymentFormMixin = require('payment.payment_form_mixin');

    const _t = core._t;

    publicWidget.registry.PaymentManageForm = publicWidget.Widget.extend(paymentFormMixin, {
        selector: 'form[name="o_payment_manage"]',
        events: Object.assign({}, publicWidget.Widget.prototype.events, {
            'click div[name="o_payment_option_card"]': '_onClickPaymentOption',
            'click a[name="o_payment_icon_more"]': '_onClickMorePaymentIcons',
            'click a[name="o_payment_icon_less"]': '_onClickLessPaymentIcons',
            'click button[name="o_payment_submit_button"]': '_onClickSaveToken',
            'click button[name="o_payment_delete_token"]': '_onClickDeleteToken',
            'submit': '_onSubmit',
        }),

        /**
         * @constructor
         */
        init: function () {
            const preventDoubleClick = handlerMethod => {
                return _.debounce(handlerMethod, 500, true);
            };
            this._super(...arguments);
            // Prevent double-clicks and browser glitches on all inputs
            this._onClickDeleteToken = preventDoubleClick(this._onClickDeleteToken);
            this._onClickLessPaymentIcons = preventDoubleClick(this._onClickLessPaymentIcons);
            this._onClickMorePaymentIcons = preventDoubleClick(this._onClickMorePaymentIcons);
            this._onClickPaymentOption = preventDoubleClick(this._onClickPaymentOption);
            this._onClickSaveToken = preventDoubleClick(this._onClickSaveToken);
            this._onSubmit = preventDoubleClick(this._onSubmit);
        },

        //--------------------------------------------------------------------------
        // Private
        //--------------------------------------------------------------------------

        /**
         * Assign the token to a record.
         *
         * @private
         * @param {number} tokenId - The id of the token to assign
         * @return {undefined}
         */
        _assignToken: function (tokenId) {
            // Call the assign route to assign the token to a record
            this._rpc({
                route: this.txContext.assignTokenRoute,
                params: {
                    'token_id': tokenId,
                    'csrf_token': core.csrf_token,
                }
            }).then(() => {
                window.location = this.txContext.landingRoute;
            }).guardedCatch(error => {
                error.event.preventDefault();
                this._displayError(
                    _t("Server Error"),
                    _t("We are not able to save your payment method."),
                    error.message.data.message
                );
            });
        },

        /**
         * Search for documents linked to the token and ask the user for confirmation.
         *
         * If any such document is found, a confirmation dialog is shown.
         *
         * @private
         * @param {number} tokenId - The id of the token to delete
         * @return {undefined}
         */
        _deleteToken: function (tokenId) {
            const execute = () => {
                this._rpc({
                    route: '/payment/archive_token',
                    params: {
                        'token_id': tokenId,
                    },
                }).then(() => {
                    const $tokenCard = this.$(
                        `input[name="o_payment_radio"][data-payment-option-id="${tokenId}"]` +
                        `[data-payment-option-type="token"]`
                    ).closest('div[name="o_payment_option_card"]');
                    $tokenCard.siblings(`#o_payment_token_inline_form_${tokenId}`).remove();
                    $tokenCard.remove();
                    this._disableButton(false);
                }).guardedCatch(error => {
                    error.event.preventDefault();
                    this._displayError(
                        _t("Server Error"),
                        _t("We are not able to delete your payment method."),
                        error.message.data.message
                    );
                });
            };

            // Fetch documents linked to the token
            this._rpc({
                model: 'payment.token',
                method: 'get_linked_records_info',
                args: [tokenId],
            }).then(result => {
                const $dialogContentMessage = $(
                    '<span>', {text: _t("Are you sure you want to delete this payment method?")}
                );
                if (result.length > 0) { // There are documents linked to the token, list them
                    $dialogContentMessage.append($('<br>'));
                    $dialogContentMessage.append($(
                        '<span>', {text: _t("It is currently linked to the following documents:")}
                    ));
                    const $documentInfoList = $('<ul>');
                    result.forEach(documentInfo => {
                        $documentInfoList.append($('<li>').append($(
                            '<a>', {
                                href: documentInfo.url,
                                target: '_blank',
                                title: documentInfo.description,
                                text: documentInfo.name
                            }
                        )));
                    });
                    $dialogContentMessage.append($documentInfoList);
                }
                new Dialog(this, {
                    title: _t("Warning!"),
                    size: 'medium',
                    $content: $('<div>').append($dialogContentMessage),
                    buttons: [
                        {
                            text: _t("Confirm Deletion"), classes: 'btn-primary', close: true,
                            click: execute
                        },
                        {
                            text: _t("Cancel"), close: true
                        },
                    ],
                }).open();
            }).guardedCatch(error => {
                this._displayError(
                    _t("Server Error"),
                    _t("We are not able to delete your payment method."),
                    error.message.data.message
                );
            });
        },

        //--------------------------------------------------------------------------
        // Handlers
        //--------------------------------------------------------------------------

        /**
         * Find the radio button linked to the click 'Delete' button and trigger the token deletion.
         *
         * Let `_onClickPaymentOption` select the radio button and display the inline form.
         *
         * Called when clicking on the 'Delete' button of a token.
         *
         * @private
         * @param {Event} ev
         * @return {undefined}
         */
        _onClickDeleteToken: function (ev) {
            ev.preventDefault();

            // Extract contextual values from the delete button
            const linkedRadio = $(ev.currentTarget).siblings().find('input[name="o_payment_radio"]')[0];
            const tokenId = this._getPaymentOptionIdFromRadio(linkedRadio);

            // Delete the token
            this._deleteToken(tokenId);
        },

        /**
         * Handle the creation of a new token or the assignation of a token to a record.
         *
         * Called when clicking on the 'Save Payment Method' button of when submitting the form.
         *
         * @private
         * @param {Event} ev
         * @return {undefined}
         */
        _onClickSaveToken: async function (ev) {
            ev.stopPropagation();
            ev.preventDefault();

            // Check that the user has selected a payment option
            const $checkedRadios = this.$('input[name="o_payment_radio"]:checked');
            if (!this._ensureRadioIsChecked($checkedRadios)) {
                return;
            }
            const checkedRadio = $checkedRadios[0];

            // Extract contextual values from the radio button
            const provider = this._getProviderFromRadio(checkedRadio);
            const paymentOptionId = this._getPaymentOptionIdFromRadio(checkedRadio);
            const flow = this._getPaymentFlowFromRadio(checkedRadio);

            // Save the payment method
            this._hideError(); // Don't keep the error displayed if the user is going through 3DS2
            this._disableButton(true); // Disable until it is needed again
            $('body').block({
                message: false,
                overlayCSS: {backgroundColor: "#000", opacity: 0, zIndex: 1050},
            });
            if (flow !== 'token') { // Creation of a new token
                this.txContext.tokenizationRequested = true;
                this.txContext.isValidation = true;
                this._processPayment(provider, paymentOptionId, flow);
            } else if (this.txContext.allowTokenSelection) { // Assignation of a token to a record
                this._assignToken(paymentOptionId);
            }
        },

        /**
         * Delegate the handling of the token to `_onClickSaveToken`.
         *
         * Called when submitting the form (e.g. through the Return key).
         *
         * @private
         * @param {Event} ev
         * @return {undefined}
         */
        _onSubmit: function (ev) {
            ev.stopPropagation();
            ev.preventDefault();

            this._onClickSaveToken(ev);
        },

    });
    return publicWidget.registry.PaymentManageForm;
});

```

## File: static\src\js\payment_form_mixin.js

```javascript
odoo.define('payment.payment_form_mixin', require => {
    'use strict';

    const core = require('web.core');
    const Dialog = require('web.Dialog');

    const _t = core._t;

    return {

        /**
         * @override
         */
        start: async function () {
            this.txContext = {}; // Synchronously initialize txContext before any await.
            Object.assign(this.txContext, this.$el.data());
            await this._super(...arguments);
            window.addEventListener('pageshow', function (event) {
                if (event.persisted) {
                    window.location.reload();
                }
            });
            this.$('[data-toggle="tooltip"]').tooltip();
            const $checkedRadios = this.$('input[name="o_payment_radio"]:checked');
            if ($checkedRadios.length === 1) {
                const checkedRadio = $checkedRadios[0];
                this._displayInlineForm(checkedRadio);
                this._enableButton();
            } else {
                this._setPaymentFlow(); // Initialize the payment flow to let acquirers overwrite it
            }
        },

        //--------------------------------------------------------------------------
        // Private
        //--------------------------------------------------------------------------

        /**
         * Disable the submit button.
         *
         * The icons are updated to either show that an action is processing or that the button is
         * not ready, depending on the value of `showLoadingAnimation`.
         *
         * @private
         * @param {boolean} showLoadingAnimation - Whether a spinning loader should be shown
         * @return {undefined}
         */
        _disableButton: (showLoadingAnimation = true) => {
            const $submitButton = this.$('button[name="o_payment_submit_button"]');
            const iconClass = $submitButton.data('icon-class');
            $submitButton.attr('disabled', true);
            if (showLoadingAnimation) {
                $submitButton.find('i').removeClass(iconClass);
                $submitButton.prepend(
                    '<span class="o_loader"><i class="fa fa-refresh fa-spin"></i>&nbsp;</span>'
                );
            }
        },

        /**
         * Display an error in the payment form.
         *
         * If no payment option is selected, the error is displayed in a dialog. If exactly one
         * payment option is selected, the error is displayed in the inline form of that payment
         * option and the view is focused on the error.
         *
         * @private
         * @param {string} title - The title of the error
         * @param {string} description - The description of the error
         * @param {string} error - The raw error message
         * @return {(Dialog|undefined)} A dialog showing the error if no payment option is selected,
         *                              undefined otherwise.
         */
        _displayError: function (title, description = '', error = '') {
            const $checkedRadios = this.$('input[name="o_payment_radio"]:checked');
            if ($checkedRadios.length !== 1) { // Cannot find selected payment option, show dialog
                return new Dialog(null, {
                    title: _.str.sprintf(_t("Error: %s"), title),
                    size: 'medium',
                    $content: `<p>${_.str.escapeHTML(description) || ''}</p>`,
                    buttons: [{text: _t("Ok"), close: true}]
                }).open();
            } else { // Show error in inline form
                this._hideError(); // Remove any previous error

                // Build the html for the error
                let errorHtml = `<div class="alert alert-danger mb4" name="o_payment_error">
                                 <b>${_.str.escapeHTML(title)}</b>`;
                if (description !== '') {
                    errorHtml += `</br>${_.str.escapeHTML(description)}`;
                }
                if (error !== '') {
                    errorHtml += `</br>${_.str.escapeHTML(error)}`;
                }
                errorHtml += '</div>';

                // Append error to inline form and center the page on the error
                const checkedRadio = $checkedRadios[0];
                const paymentOptionId = this._getPaymentOptionIdFromRadio(checkedRadio);
                const formType = $(checkedRadio).data('payment-option-type');
                const $inlineForm = this.$(`#o_payment_${formType}_inline_form_${paymentOptionId}`);
                $inlineForm.removeClass('d-none'); // Show the inline form even if it was empty
                $inlineForm.append(errorHtml).find('div[name="o_payment_error"]')[0]
                    .scrollIntoView({behavior: 'smooth', block: 'center'});
            }
            this._enableButton(); // Enable button back after it was disabled before processing
            $('body').unblock(); // The page is blocked at this point, unblock it
        },

        /**
         * Display the inline form of the selected payment option and hide others.
         *
         * @private
         * @param {HTMLInputElement} radio - The radio button linked to the payment option
         * @return {undefined}
         */
        _displayInlineForm: function (radio) {
            this._hideInlineForms(); // Collapse previously opened inline forms
            this._hideError(); // The error is only relevant until it is hidden with its inline form
            this._setPaymentFlow(); // Reset the payment flow to let acquirers overwrite it

            // Extract contextual values from the radio button
            const provider = this._getProviderFromRadio(radio);
            const paymentOptionId = this._getPaymentOptionIdFromRadio(radio);
            const flow = this._getPaymentFlowFromRadio(radio);

            // Prepare the inline form of the selected payment option and display it if not empty
            this._prepareInlineForm(provider, paymentOptionId, flow);
            const formType = $(radio).data('payment-option-type');
            const $inlineForm = this.$(`#o_payment_${formType}_inline_form_${paymentOptionId}`);
            if ($inlineForm.children().length > 0) {
                $inlineForm.removeClass('d-none');
            }
        },

        /**
         * Check if the submit button can be enabled and do it if so.
         *
         * The icons are updated to show that the button is ready.
         *
         * @private
         * @return {boolean} Whether the button was enabled.
         */
        _enableButton: function () {
            if (this._isButtonReady()) {
                const $submitButton = this.$('button[name="o_payment_submit_button"]');
                const iconClass = $submitButton.data('icon-class');
                $submitButton.attr('disabled', false);
                $submitButton.find('i').addClass(iconClass);
                $submitButton.find('span.o_loader').remove();
                return true;
            }
            return false;
        },

        /**
         * Verify that exactly one radio button is checked and display an error otherwise.
         *
         * @private
         * @param {jQuery} $checkedRadios - The currently check radio buttons
         *
         * @return {boolean} Whether exactly one radio button among the provided radios is checked
         */
        _ensureRadioIsChecked: function ($checkedRadios) {
            if ($checkedRadios.length === 0) {
                this._displayError(
                    _t("No payment option selected"),
                    _t("Please select a payment option.")
                );
                return false;
            } else if ($checkedRadios.length > 1) {
                this._displayError(
                    _t("Multiple payment options selected"),
                    _t("Please select only one payment option.")
                );
                return false;
            }
            return true;
        },

        /**
         * Determine and return the online payment flow of the selected payment option.
         *
         * As some acquirers implement both the direct payment and the payment with redirection, the
         * flow cannot be inferred from the radio button only. The radio button only indicates
         * whether the payment option is a token. If not, the transaction context is looked up to
         * determine whether the flow is 'direct' or 'redirect'.
         *
         * @private
         * @param {HTMLInputElement} radio - The radio button linked to the payment option
         * @return {string} The flow of the selected payment option. redirect, direct or token.
         */
        _getPaymentFlowFromRadio: function (radio) {
            if (
                $(radio).data('payment-option-type') === 'token'
                || this.txContext.flow === 'token'
            ) {
                return 'token';
            } else if (this.txContext.flow === 'redirect') {
                return 'redirect';
            } else {
                return 'direct';
            }
        },

        /**
         * Determine and return the id of the selected payment option.
         *
         * @private
         * @param {HTMLInputElement} radio - The radio button linked to the payment option
         * @return {number} The acquirer id or the token id or of the payment option linked to the
         *                  radio button.
         */
        _getPaymentOptionIdFromRadio: radio => $(radio).data('payment-option-id'),

        /**
         * Determine and return the provider of the selected payment option.
         *
         * @private
         * @param {HTMLInputElement} radio - The radio button linked to the payment option
         * @return {number} The provider of the payment option linked to the radio button.
         */
        _getProviderFromRadio: radio => $(radio).data('provider'),

        /**
         * Remove the error in the acquirer form.
         *
         * @private
         * @return {jQuery} The removed error
         */
        _hideError: () => this.$('div[name="o_payment_error"]').remove(),

        /**
         * Collapse all inline forms.
         *
         * @private
         * @return {undefined}.
         */
        _hideInlineForms: () => this.$('[name="o_payment_inline_form"]').addClass('d-none'),

        /**
         * Hide the "Save my payment details" label and checkbox, and the submit button.
         *
         * The inputs should typically be hidden when the customer has to perform additional actions
         * in the inline form. All inputs are automatically shown again when the customer clicks on
         * another inline form.
         *
         * @private
         * @return {undefined}
         */
        _hideInputs: function () {
            const $submitButton = this.$('button[name="o_payment_submit_button"]');
            const $tokenizeCheckboxes = this.$('input[name="o_payment_save_as_token"]');
            $submitButton.addClass('d-none');
            $tokenizeCheckboxes.closest('label').addClass('d-none');
        },

        /**
         * Verify that the submit button is ready to be enabled.
         *
         * For a module to support a custom behavior for the submit button, it must override this
         * method and only return true if the result of this method is true and if nothing prevents
         * enabling the submit button for that custom behavior.
         *
         * @private
         *
         * @return {boolean} Whether the submit button can be enabled
         */
        _isButtonReady: function () {
            const $checkedRadios = this.$('input[name="o_payment_radio"]:checked');
            if ($checkedRadios.length === 1) {
                const checkedRadio = $checkedRadios[0];
                const flow = this._getPaymentFlowFromRadio(checkedRadio);
                return flow !== 'token' || this.txContext.allowTokenSelection;
            } else {
                return false;
            }
        },

        /**
         * Prepare the params to send to the transaction route.
         *
         * For an acquirer to overwrite generic params or to add acquirer-specific ones, it must
         * override this method and return the extended transaction route params.
         *
         * @private
         * @param {string} provider - The provider of the selected payment option's acquirer
         * @param {number} paymentOptionId - The id of the selected payment option
         * @param {string} flow - The online payment flow of the selected payment option
         * @return {object} The transaction route params
         */
        _prepareTransactionRouteParams: function (provider, paymentOptionId, flow) {
            return {
                'payment_option_id': paymentOptionId,
                'reference_prefix': this.txContext.referencePrefix !== undefined
                    ? this.txContext.referencePrefix.toString() : null,
                'amount': this.txContext.amount !== undefined
                    ? parseFloat(this.txContext.amount) : null,
                'currency_id': this.txContext.currencyId
                    ? parseInt(this.txContext.currencyId) : null,
                'partner_id': parseInt(this.txContext.partnerId),
                'invoice_id': this.txContext.invoiceId
                    ? parseInt(this.txContext.invoiceId) : null,
                'flow': flow,
                'tokenization_requested': this.txContext.tokenizationRequested,
                'landing_route': this.txContext.landingRoute,
                'is_validation': this.txContext.isValidation,
                'access_token': this.txContext.accessToken
                    ? this.txContext.accessToken : undefined,
                'csrf_token': core.csrf_token,
            };
        },

        /**
         * Prepare the acquirer-specific inline form of the selected payment option.
         *
         * For an acquirer to manage an inline form, it must override this method. When the override
         * is called, it must lookup the parameters to decide whether it is necessary to prepare its
         * inline form. Otherwise, the call must be sent back to the parent method.
         *
         * @private
         * @param {string} provider - The provider of the selected payment option's acquirer
         * @param {number} paymentOptionId - The id of the selected payment option
         * @param {string} flow - The online payment flow of the selected payment option
         * @return {Promise}
         */
        _prepareInlineForm: (provider, paymentOptionId, flow) => Promise.resolve(),

        /**
         * Process the payment.
         *
         * For an acquirer to do pre-processing work on the transaction processing flow, or to
         * define its entire own flow that requires re-scheduling the RPC to the transaction route,
         * it must override this method.
         * If only post-processing work is needed, an override of `_processRedirectPayment`,
         * `_processDirectPayment` or `_processTokenPayment` might be more appropriate.
         *
         * @private
         * @param {string} provider - The provider of the payment option's acquirer
         * @param {number} paymentOptionId - The id of the payment option handling the transaction
         * @param {string} flow - The online payment flow of the transaction
         * @return {Promise}
         */
        _processPayment: function (provider, paymentOptionId, flow) {
            // Call the transaction route to create a tx and retrieve the processing values
            return this._rpc({
                route: this.txContext.transactionRoute,
                params: this._prepareTransactionRouteParams(provider, paymentOptionId, flow),
            }).then(processingValues => {
                if (flow === 'redirect') {
                    return this._processRedirectPayment(
                        provider, paymentOptionId, processingValues
                    );
                } else if (flow === 'direct') {
                    return this._processDirectPayment(provider, paymentOptionId, processingValues);
                } else if (flow === 'token') {
                    return this._processTokenPayment(provider, paymentOptionId, processingValues);
                }
            }).guardedCatch(error => {
                error.event.preventDefault();
                this._displayError(
                    _t("Server Error"),
                    _t("We are not able to process your payment."),
                    error.message.data.message
                );
            });
        },

        /**
         * Execute the acquirer-specific implementation of the direct payment flow.
         *
         * For an acquirer to redefine the processing of the direct payment flow, it must override
         * this method.
         *
         * @private
         * @param {string} provider - The provider of the acquirer
         * @param {number} acquirerId - The id of the acquirer handling the transaction
         * @param {object} processingValues - The processing values of the transaction
         * @return {Promise}
         */
        _processDirectPayment: (provider, acquirerId, processingValues) => Promise.resolve(),

        /**
         * Redirect the customer by submitting the redirect form included in the processing values.
         *
         * For an acquirer to redefine the processing of the payment with redirection flow, it must
         * override this method.
         *
         * @private
         * @param {string} provider - The provider of the acquirer
         * @param {number} acquirerId - The id of the acquirer handling the transaction
         * @param {object} processingValues - The processing values of the transaction
         * @return {undefined}
         */
        _processRedirectPayment: (provider, acquirerId, processingValues) => {
            // Append the redirect form to the body
            const $redirectForm = $(processingValues.redirect_form_html).attr(
                'id', 'o_payment_redirect_form'
            );
            $(document.getElementsByTagName('body')[0]).append($redirectForm);

            // Submit the form
            $redirectForm.submit();
        },

        /**
         * Redirect the customer to the status route.
         *
         * For an acquirer to redefine the processing of the payment by token flow, it must override
         * this method.
         *
         * @private
         * @param {string} provider - The provider of the token's acquirer
         * @param {number} tokenId - The id of the token handling the transaction
         * @param {object} processingValues - The processing values of the transaction
         * @return {undefined}
         */
        _processTokenPayment: (provider, tokenId, processingValues) => {
            // The flow is already completed as payments by tokens are immediately processed
            window.location = '/payment/status';
        },

        /**
         * Set the online payment flow for the selected payment option.
         *
         * For an acquirer to manage direct payments, it must call this method from within its
         * override of `_prepareInlineForm` to declare its payment flow for the selected payment
         * option.
         *
         * @private
         * @param {string} flow - The flow for the selected payment option. Either 'redirect',
         *                        'direct' or 'token'
         * @return {undefined}
         */
        _setPaymentFlow: function (flow = 'redirect') {
            if (flow !== 'redirect' && flow !== 'direct' && flow !== 'token') {
                console.warn(
                    `payment_form_mixin: method '_setPaymentFlow' was called with invalid flow: 
                    ${flow}. Falling back to 'redirect'.`
                );
                this.txContext.flow = 'redirect';
            } else {
                this.txContext.flow = flow;
            }
        },

        /**
         * Show the "Save my payment details" label and checkbox, and the submit button.
         *
         * @private
         * @return {undefined}.
         */
        _showInputs: function () {
            const $submitButton = this.$('button[name="o_payment_submit_button"]');
            const $tokenizeCheckboxes = this.$('input[name="o_payment_save_as_token"]');
            $submitButton.removeClass('d-none');
            $tokenizeCheckboxes.closest('label').removeClass('d-none');
        },

        //--------------------------------------------------------------------------
        // Handlers
        //--------------------------------------------------------------------------

        /**
         * Hide all extra payment icons of the acquirer linked to the clicked button.
         *
         * Called when clicking on the "show less" button.
         *
         * @private
         * @param {Event} ev
         * @return {undefined}
         */
        _onClickLessPaymentIcons: ev => {
            ev.preventDefault();
            ev.stopPropagation();
            // Hide the extra payment icons, and the "show less" button
            const $itemList = $(ev.currentTarget).parents('ul');
            const maxIconNumber = $itemList.data('max-icons');
            $itemList.children('li').slice(maxIconNumber).addClass('d-none');
            // Show the "show more" button
            $itemList.find('a[name="o_payment_icon_more"]').parents('li').removeClass('d-none');
        },

        /**
         * Display all the payment icons of the acquirer linked to the clicked button.
         *
         * Called when clicking on the "show more" button.
         *
         * @private
         * @param {Event} ev
         * @return {undefined}
         */
        _onClickMorePaymentIcons: ev => {
            ev.preventDefault();
            ev.stopPropagation();
            // Display all the payment icons, and the "show less" button
            $(ev.currentTarget).parents('ul').children('li').removeClass('d-none');
            // Hide the "show more" button
            $(ev.currentTarget).parents('li').addClass('d-none');
        },

        /**
         * Mark the clicked card radio button as checked and open the inline form, if any.
         *
         * Called when clicking on the card of a payment option.
         *
         * @private
         * @param {Event} ev
         * @return {undefined}
         */
        _onClickPaymentOption: function (ev) {
            // Uncheck all radio buttons
            this.$('input[name="o_payment_radio"]').prop('checked', false);
            // Check radio button linked to selected payment option
            const checkedRadio = $(ev.currentTarget).find('input[name="o_payment_radio"]')[0];
            $(checkedRadio).prop('checked', true);

            // Show the inputs in case they had been hidden
            this._showInputs();

            // Disable the submit button while building the content
            this._disableButton(false);

            // Unfold and prepare the inline form of selected payment option
            this._displayInlineForm(checkedRadio);

            // Re-enable the submit button
            this._enableButton();
        },

    };

});

```

## File: static\src\js\post_processing.js

```javascript
odoo.define('payment.post_processing', function (require) {
    'use strict';

    var publicWidget = require('web.public.widget');
    var core = require('web.core');
    const {Markup} = require('web.utils');

    var _t = core._t;

    $.blockUI.defaults.css.border = '0';
    $.blockUI.defaults.css["background-color"] = '';
    $.blockUI.defaults.overlayCSS["opacity"] = '0.9';

    publicWidget.registry.PaymentPostProcessing = publicWidget.Widget.extend({
        selector: 'div[name="o_payment_status"]',
        xmlDependencies: ['/payment/static/src/xml/payment_post_processing.xml'],

        _pollCount: 0,

        start: function() {
            this.displayLoading();
            this.poll();
            return this._super.apply(this, arguments);
        },
        /* Methods */
        startPolling: function () {
            var timeout = 3000;
            //
            if(this._pollCount >= 10 && this._pollCount < 20) {
                timeout = 10000;
            }
            else if(this._pollCount >= 20) {
                timeout = 30000;
            }
            //
            setTimeout(this.poll.bind(this), timeout);
            this._pollCount ++;
        },
        poll: function () {
            var self = this;
            this._rpc({
                route: '/payment/status/poll',
                params: {
                    'csrf_token': core.csrf_token,
                }
            }).then(function(data) {
                if(data.success === true) {
                    self.processPolledData(data.display_values_list);
                }
                else {
                    switch(data.error) {
                    case "tx_process_retry":
                        break;
                    case "no_tx_found":
                        self.displayContent("payment.no_tx_found", {});
                        break;
                    default: // if an exception is raised
                        self.displayContent("payment.exception", {exception_msg: data.error});
                        break;
                    }
                }
                self.startPolling();

            }).guardedCatch(function() {
                self.displayContent("payment.rpc_error", {});
                self.startPolling();
            });
        },
        processPolledData: function (display_values_list) {
            var render_values = {
                'tx_draft': [],
                'tx_pending': [],
                'tx_authorized': [],
                'tx_done': [],
                'tx_cancel': [],
                'tx_error': [],
            };

            if (display_values_list.length > 0) {
                // In almost every cases there will be a single transaction to display. If there are
                // more than one transaction, the last one will most likely be the one that was
                // confirmed. We use this one to redirect the user to the final page.
                window.location = display_values_list[0].landing_route;
                return;
            }

            // group the transaction according to their state
            display_values_list.forEach(function (display_values) {
                var key = 'tx_' + display_values.state;
                if(key in render_values) {
                    if (display_values["display_message"]) {
                        display_values.display_message = Markup(display_values.display_message)
                    }
                    render_values[key].push(display_values);
                }
            });

            function countTxInState(states) {
                var nbTx = 0;
                for (var prop in render_values) {
                    if (states.indexOf(prop) > -1 && render_values.hasOwnProperty(prop)) {
                        nbTx += render_values[prop].length;
                    }
                }
                return nbTx;
            }
                       
            /*
            * When the server sends the list of monitored transactions, it tries to post-process 
            * all the successful ones. If it succeeds or if the post-process has already been made, 
            * the transaction is removed from the list of monitored transactions and won't be 
            * included in the next response. We assume that successful and post-process 
            * transactions should always prevail on others, regardless of their number or state.
            */
            if (render_values['tx_done'].length === 1 &&
                render_values['tx_done'][0].is_post_processed) {
                    window.location = render_values['tx_done'][0].landing_route;
                    return;
            }
            // If there are multiple transactions monitored, display them all to the customer. If
            // there is only one transaction monitored, redirect directly the customer to the
            // landing route.
            if(countTxInState(['tx_done', 'tx_error', 'tx_pending', 'tx_authorized']) === 1) {
                // We don't want to redirect customers to the landing page when they have a pending
                // transaction. The successful transactions are dealt with before.
                var tx = render_values['tx_authorized'][0] || render_values['tx_error'][0];
                if (tx) {
                    window.location = tx.landing_route;
                    return;
                }
            }

            this.displayContent("payment.display_tx_list", render_values);
        },
        displayContent: function (xmlid, render_values) {
            var html = core.qweb.render(xmlid, render_values);
            $.unblockUI();
            this.$el.find('div[name="o_payment_status_content"]').html(html);
        },
        displayLoading: function () {
            var msg = _t("We are processing your payment, please wait ...");
            $.blockUI({
                'message': '<h2 class="text-white"><img src="/web/static/img/spin.png" class="fa-pulse"/>' +
                    '    <br />' + msg +
                    '</h2>'
            });
        },
    });
});

```

## File: static\src\xml\payment_post_processing.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="payment" xml:space="preserve">
    <!-- The templates here as rendered by 'post_processing.js', you can also take
            a look at payment_portal_templates.xml (xmlid: payment_status) for more infos-->
    <t t-name="payment.display_tx_list">
        <div>
            <!-- Error transactions -->
            <div t-if="tx_error.length > 0">
                <h1>Payments failed</h1>
                <ul class="list-group">
                    <t t-foreach="tx_error" t-as="tx">
                        <li class="list-group-item">
                            <h4 class="list-group-item-heading mb5">
                                <t t-esc="tx['reference']"/>
                                <span class="badge pull-right"><t t-esc="tx['amount']"/> <t t-esc="tx['currency_code']"/></span>
                            </h4>
                            <small class="list-group-item-text">
                                An error occurred during the processing of this payment.<br/>
                                <strong>Reason:</strong> <t t-esc="tx['state_message']"/>
                            </small>
                        </li>
                    </t>
                </ul>
            </div>
            <div t-if="tx_done.length > 0 || tx_authorized.length > 0 || tx_pending.length > 0">
                <h1>Payments received</h1>
                <div class="list-group">
                    <!-- Done transactions -->
                    <t t-foreach="tx_done" t-as="tx">
                        <a t-att-href="tx['landing_route']" class="list-group-item">
                            <h4 class="list-group-item-heading mb5">
                                <t t-esc="tx['reference']"/>
                                <span class="badge pull-right"><t t-esc="tx['amount']"/> <t t-esc="tx['currency_code']"/></span>
                            </h4>
                            <small class="list-group-item-text">
                                <t t-if="!tx['is_post_processed']">
                                    Your order is being processed, please wait ... <i class="fa fa-cog fa-spin"/>
                                </t>
                                <t t-else="">
                                    Your order has been processed.<br/>
                                    Click here to be redirected to the confirmation page.
                                </t>
                            </small>
                        </a>
                    </t>
                    <!-- Pending transactions -->
                    <t t-foreach="tx_pending" t-as="tx">
                        <a t-att-href="tx['landing_route']" class="list-group-item">
                            <h4 class="list-group-item-heading mb5">
                                <t t-esc="tx['reference']"/>
                                <span class="badge pull-right"><t t-esc="tx['amount']"/> <t t-esc="tx['currency_code']"/></span>
                            </h4>
                            <small class="list-group-item-text">
                                <t t-if="tx['display_message']">
                                    <!-- display_message is the content of the HTML field associated
                                    with the current transaction state, set on the acquirer. -->
                                    <t t-out="tx['display_message']"/>
                                </t>
                                <t t-else="">
                                    Your payment is in pending state.<br/>
                                    You will be notified when the payment is fully confirmed.<br/>
                                    You can click here to be redirected to the confirmation page.
                                </t>
                            </small>
                        </a>
                    </t>
                    <!-- Authorized transactions -->
                    <t t-foreach="tx_authorized" t-as="tx">
                        <li class="list-group-item">
                            <h4 class="list-group-item-heading mb5">
                                <t t-esc="tx['reference']"/>
                                <span class="badge pull-right"><t t-esc="tx['amount']"/> <t t-esc="tx['currency_code']"/></span>
                            </h4>
                            <small class="list-group-item-text">
                                <t t-if="tx['display_message']">
                                    <!-- display_message is the content of the HTML field associated
                                    with the current transaction state, set on the acquirer. -->
                                    <t t-out="tx['display_message']"/>
                                </t>
                                <t t-else="">
                                    Your payment has been received but need to be confirmed manually.<br/>
                                    You will be notified when the payment is confirmed.
                                </t>
                            </small>
                        </li>
                    </t>
                </div>
            </div>
            <!-- Draft transactions -->
            <div t-if="tx_draft.length > 0">
                <h1>Waiting for payment</h1>
                <ul class="list-group">
                    <t t-foreach="tx_draft" t-as="tx">
                        <li class="list-group-item">
                            <h4 class="list-group-item-heading mb5">
                                <t t-esc="tx['reference']"/>
                                <span class="badge pull-right"><t t-esc="tx['amount']"/> <t t-esc="tx['currency_code']"/></span>
                            </h4>
                            <small class="list-group-item-text">
                                <t t-if="tx['display_message']">
                                    <!-- display_message is the content of the HTML field associated
                                    with the current transaction state, set on the acquirer. -->
                                    <t t-out="tx['display_message']"/>
                                </t>
                                <t t-else="">
                                    We are waiting for the payment acquirer to confirm the payment.
                                </t>
                            </small>
                        </li>
                    </t>
                </ul>
            </div>
            <!-- Cancel transactions -->
            <div t-if="tx_cancel.length > 0">
                <h1>Cancelled payments</h1>
                <ul class="list-group">
                    <t t-foreach="tx_cancel" t-as="tx">
                        <li class="list-group-item">
                            <h4 class="list-group-item-heading mb5">
                                <t t-esc="tx['reference']"/>
                                <span class="badge pull-right"><t t-esc="tx['amount']"/> <t t-esc="tx['currency_code']"/></span>
                            </h4>
                            <small class="list-group-item-text">
                                This transaction has been cancelled.<br/>
                                No payment has been processed.
                            </small>
                        </li>
                    </t>
                </ul>
            </div>
        </div>
    </t>

    <t t-name="payment.no_tx_found">
        <div class="text-center">
            <p>We are not able to find your payment, but don't worry.</p>
            <p>You should receive an email confirming your payment in a few minutes.</p>
            <p>If the payment hasn't been confirmed you can contact us.</p>
        </div>
    </t>

    <t t-name="payment.rpc_error">
        <div class="text-center">
            <p><strong>Server error:</strong> Unable to contact the Odoo server.</p>
            <p>Please wait ... <i class="fa fa-refresh fa-spin"></i></p>
        </div>
    </t>

    <t t-name="payment.exception">
        <div class="text-center">
            <h2>Internal server error</h2>
            <pre><t t-esc="exception_msg"/></pre>
        </div>
    </t>

</templates>

```

## File: views\account_invoice_views.xml

```xml
<?xml version="1.0"?>
<odoo>

    <record id="account_invoice_view_form_inherit_payment" model="ir.ui.view">
        <field name="name">account.move.view.form.inherit.payment</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_move_form"/>
        <field name="arch" type="xml">
            <!--
            The user must capture/void the authorized transactions before registering a new payment.
            -->
            <xpath expr="//button[@id='account_invoice_payment_btn']" position="attributes">
                <attribute name="attrs">{'invisible': ['|', '|', '|', ('state', '!=', 'posted'), ('payment_state', 'not in', ('partial', 'not_paid')), ('move_type', 'not in', ('out_invoice', 'out_refund', 'in_invoice', 'in_refund', 'out_receipt', 'in_receipt')), ('authorized_transaction_ids', '!=', [])]}</attribute>
            </xpath>
            <xpath expr="//button[@id='account_invoice_payment_btn']" position="after">
                <field name="authorized_transaction_ids" invisible="1"/>
                <button name="payment_action_capture" type="object"
                        groups="account.group_account_invoice"
                        string="Capture Transaction" class="oe_highlight" data-hotkey="shift+g"
                        attrs="{'invisible': ['|', '|', ('move_type', 'not in', ('out_invoice', 'out_refund', 'in_invoice', 'in_refund')), ('state', '!=', 'posted'), ('authorized_transaction_ids', '=', [])]}"/>
                <button name="payment_action_void" type="object"
                        groups="account.group_account_invoice"
                        string="Void Transaction" data-hotkey="shift+v"
                        confirm="Are you sure you want to void the authorized transaction? This action can't be undone."
                        attrs="{'invisible': ['|', '|', ('move_type', 'not in', ('out_invoice', 'out_refund', 'in_invoice', 'in_refund')), ('state', '!=', 'posted'), ('authorized_transaction_ids', '=', [])]}"/>
            </xpath>
            <xpath expr="//div[@name='button_box']" position="inside">
                <field name="transaction_ids" invisible="1" />
                <button name="action_view_payment_transactions" type="object"
                        class="oe_stat_button" icon="fa-money"
                        string="Payment Transaction"
                        attrs="{'invisible': [('transaction_ids', '=', [])]}" />
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\account_journal_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_account_journal_form" model="ir.ui.view">
        <field name="name">account.journal.form.inherit.payment</field>
        <field name="model">account.journal</field>
        <field name="inherit_id" ref="account.view_account_journal_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='inbound_payment_method_line_ids']//field[@name='payment_account_id']" position="after">
                <field name="payment_acquirer_id" options="{'no_open': True, 'no_create': True}" optional="hide" domain="[('provider', '=', code)]"/>
                <field name="payment_acquirer_state" invisible="1"/>
                <field name="code" invisible="1"/>
                <button name="action_open_acquirer_form"
                        type="object"
                        string="SETUP"
                        class="float-right btn-secondary"
                        attrs="{'invisible': [('payment_acquirer_id', '=', False)]}"
                        groups="base.group_system"/>
            </xpath>
            <xpath expr="//field[@name='inbound_payment_method_line_ids']/tree" position="attributes">
                <attribute name="decoration-muted">payment_acquirer_state == 'disabled'</attribute>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\account_payment_views.xml

```xml
<?xml version="1.0"?>
<odoo>

    <record id="view_account_payment_form_inherit_payment" model="ir.ui.view">
        <field name="name">view.account.payment.form.inherit.payment</field>
        <field name="model">account.payment</field>
        <field name="inherit_id" ref="account.view_account_payment_form"/>
        <field name="arch" type="xml">
            <xpath expr="//header/button[@name='action_draft']" position="after">
                <field name="amount_available_for_refund" invisible="1"/>
                <button type="object"
                    name="action_refund_wizard"
                    string="Refund"
                    groups="account.group_account_invoice"
                    attrs="{'invisible': [('amount_available_for_refund', '&lt;=', 0)]}"
                    class="btn-secondary"/>
            </xpath>
            <xpath expr="//div[@name='button_box']" position="inside">
                <button name="action_view_refunds"
                    type="object"
                    class="oe_stat_button"
                    icon="fa-money"
                    attrs="{'invisible': [('refunds_count', '=', 0)]}">
                    <field name="refunds_count" widget="statinfo" string="Refunds"/>
                </button>
            </xpath>
            <xpath expr='//group[2]' position="inside">
                <field name="source_payment_id" attrs="{'invisible': [('source_payment_id', '=', False)]}"/>
                <field name="payment_transaction_id" groups="base.group_no_one" attrs="{'invisible': [('use_electronic_payment_method', '!=', True)]}"/>
            </xpath>
            <field name="payment_method_line_id" position="after">
                <field name="payment_method_code" invisible="1"/>
                <field name="suitable_payment_token_ids" invisible="1"/>
                <field name="use_electronic_payment_method" invisible="1"/>
                <field name="payment_token_id" options="{'no_create': True}"
                    attrs="{'invisible': [('use_electronic_payment_method', '!=', True)], 'readonly': [('state', '!=', 'draft')]}"/>
            </field>
        </field>
    </record>

</odoo>

```

## File: views\payment_acquirer_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_acquirer_form" model="ir.ui.view">
        <field name="name">payment.acquirer.form</field>
        <field name="model">payment.acquirer</field>
        <field name="arch" type="xml">
            <form string="Payment Acquirer">
                <field name="support_authorization" invisible="1"/>
                <field name="support_fees_computation" invisible="1"/>
                <field name="support_tokenization" invisible="1"/>
                <field name="module_id" invisible="1"/>
                <field name="module_state" invisible="1"/>
                <field name="module_to_buy" invisible="1"/>
                <field name="show_credentials_page" invisible="1"/>
                <field name="show_allow_tokenization" invisible="1"/>
                <field name="show_payment_icon_ids" invisible="1"/>
                <field name="show_pre_msg" invisible="1"/>
                <field name="show_pending_msg" invisible="1"/>
                <field name="show_auth_msg" invisible="1"/>
                <field name="show_done_msg" invisible="1"/>
                <field name="show_cancel_msg" invisible="1"/>
                <sheet>
                    <field name="image_128" widget="image" class="oe_avatar"/>
                    <widget name="web_ribbon" title="Disabled" bg_color="bg-danger" attrs="{'invisible': [('state', '!=', 'disabled')]}"/>
                    <widget name="web_ribbon" title="Test Mode" bg_color="bg-warning" attrs="{'invisible': [('state', '!=', 'test')]}"/>
                    <div class="oe_title">
                        <h1><field name="name" placeholder="Name"/></h1>
                        <div attrs="{'invisible': ['|', ('module_state', '=', 'installed'), ('module_id', '=', False)]}">
                            <a attrs="{'invisible': [('module_to_buy', '=', False)]}" href="https://odoo.com/pricing?utm_source=db&amp;utm_medium=module" target="_blank" class="btn btn-info" role="button">Upgrade</a>
                            <button attrs="{'invisible': [('module_to_buy', '=', True)]}" type="object" class="btn btn-primary" name="button_immediate_install" string="Install"/>
                        </div>
                    </div>
                    <div attrs="{'invisible': ['|', ('module_state', '=', 'installed'), ('module_id', '=', False)]}">
                        <div class="o_payment_acquirer_desc">
                            <field name="description"/>
                        </div>
                    </div>
                    <div attrs="{'invisible': [('id', '!=', False)]}" class="alert alert-warning" role="alert">
                        <strong>Warning</strong> Creating a payment acquirer from the <em>CREATE</em> button is not supported.
                        Please use the <em>Duplicate</em> action instead.
                    </div>
                    <group>
                        <group name="payment_state">
                            <field name="provider" groups="base.group_no_one" attrs="{'readonly': [('id', '!=', False)], 'invisible': [('module_id', '!=', False), ('module_state', '!=', 'installed')]}"/>
                            <field name="state" widget="radio" attrs="{'invisible': [('module_state', '=', 'uninstalled')]}"/>
                            <field name="company_id" groups="base.group_multi_company" options='{"no_open":True}'/>
                        </group>
                    </group>
                    <notebook attrs="{'invisible': ['&amp;', ('module_id', '!=', False), ('module_state', '!=', 'installed')]}">
                        <page string="Credentials" name="acquirer_credentials" attrs="{'invisible': ['|', ('provider', '=', 'none'), ('show_credentials_page', '=', False)]}">
                            <group name="acquirer"/>
                        </page>
                        <page string="Configuration" name="configuration">
                            <group name="acquirer_config">
                                <group string="Payment Form" name="payment_form">
                                    <field name="display_as" placeholder="If not defined, the acquirer name will be used."/>
                                    <field name="payment_icon_ids" attrs="{'invisible': [('show_payment_icon_ids', '=', False)]}" widget="many2many_tags"/>
                                    <field name="allow_tokenization" attrs="{'invisible': ['|', ('support_tokenization', '=', False), ('show_allow_tokenization', '=', False)]}"/>
                                    <field name="capture_manually" attrs="{'invisible': [('support_authorization', '=', False)]}"/>
                                </group>
                                <group string="Availability" name="availability">
                                    <field name="country_ids" widget="many2many_tags" placeholder="Select countries. Leave empty to use everywhere." options="{'no_open': True, 'no_create': True}"/>
                                </group>
                                <group string="Payment Followup" name="payment_followup">
                                    <field name="journal_id" context="{'default_type': 'bank'}"
                                           attrs="{'required': [('state', '!=', 'disabled'), ('provider', 'not in', ['none', 'transfer'])]}"/>
                                </group>
                            </group>
                        </page>
                        <page string="Fees" name="fees" attrs="{'invisible': [('support_fees_computation', '=', False)]}">
                            <group name="payment_fees">
                                <field name="fees_active"/>
                                <field name="fees_dom_fixed" attrs="{'invisible': [('fees_active', '=', False)]}"/>
                                <field name="fees_dom_var" attrs="{'invisible': [('fees_active', '=', False)]}"/>
                                <field name="fees_int_fixed" attrs="{'invisible': [('fees_active', '=', False)]}"/>
                                <field name="fees_int_var" attrs="{'invisible': [('fees_active', '=', False)]}"/>
                            </group>
                        </page>
                        <page string="Messages"
                            name="messages"
                            attrs="{'invisible': [('module_id', '=', True), ('module_state', '!=', 'installed')]}">
                            <group>
                                <field name="pre_msg" attrs="{'invisible': [('show_pre_msg', '=', False)]}"/>
                                <field name="pending_msg" attrs="{'invisible': [('show_pending_msg', '=', False)]}"/>
                                <field name="auth_msg" attrs="{'invisible': ['|', ('support_authorization', '=', False), ('show_auth_msg', '=', False)]}"/>
                                <field name="done_msg" attrs="{'invisible': [('show_done_msg', '=', False)]}"/>
                                <field name="cancel_msg" attrs="{'invisible': [('show_cancel_msg', '=', False)]}"/>
                            </group>
                        </page>
                    </notebook>
                </sheet>
            </form>
        </field>
    </record>

    <record id="payment_acquirer_list" model="ir.ui.view">
        <field name="name">payment.acquirer.list</field>
        <field name="model">payment.acquirer</field>
        <field name="arch" type="xml">
            <tree string="Payment Acquirers" create="false">
                <field name="sequence" widget="handle"/>
                <field name="name"/>
                <field name="provider"/>
                <field name="state"/>
                <field name="country_ids" widget="many2many_tags" optional="hide"/>
                <field name="company_id" groups="base.group_multi_company" optional="show"/>
            </tree>
        </field>
    </record>

    <record id="payment_acquirer_kanban" model="ir.ui.view">
        <field name="name">payment.acquirer.kanban</field>
        <field name="model">payment.acquirer</field>
        <field name="arch" type="xml">
            <kanban create="false" quick_create="false" class="o_kanban_payment_acquirer o_kanban_dashboard">
                <field name="id"/>
                <field name="name"/>
                <field name="description"/>
                <field name="provider"/>
                <field name="module_id"/>
                <field name="module_state"/>
                <field name="module_to_buy"/>
                <field name="color"/>
                <templates>
                    <t t-name="kanban-box">
                        <t t-set="installed" t-value="!record.module_id.value || (record.module_id.value &amp;&amp; record.module_state.raw_value === 'installed')"/>
                        <t t-set="to_buy" t-value="record.module_to_buy.raw_value === true"/>
                        <div t-attf-class="oe_kanban_global_click #{kanban_color(record.color.raw_value)}">
                            <div class="o_payment_acquirer_desc">
                                <div class="o_kanban_image">
                                    <img type="open" t-att-src="kanban_image('payment.acquirer', 'image_128', record.id.raw_value)" alt="Acquirer"/>
                                </div>
                                <h3 class="mt4"><t t-esc="record.name.value"/></h3>
                                <t t-if="record.description.raw_value" t-out="record.description.value"/>
                            </div>
                            <div class="o_payment_acquirer_bottom">
                                <t t-if="installed">
                                    <field name="state" widget="label_selection" options="{'classes': {'enabled': 'success', 'test': 'warning', 'disabled' : 'danger'}}"/>
                                </t>
                                <button t-if="!installed and !selection_mode and !to_buy" type="object" class="btn btn-secondary float-right" name="button_immediate_install">Install</button>
                                <t t-if="!installed and to_buy">
                                    <button href="https://odoo.com/pricing?utm_source=db&amp;utm_medium=module" target="_blank" class="btn btn-info btn-sm float_right">Upgrade</button>
                                    <span class="badge badge-primary oe_inline o_enterprise_label">Enterprise</span>
                                </t>
                                <button t-if="installed and record.state.raw_value == 'disabled' and !selection_mode" type="edit" class="btn btn-primary float-right">Activate</button>
                                <button t-if="installed and record.state.raw_value in ['enabled', 'test'] and !selection_mode" type="edit" class="btn btn-primary float-right">Configure</button>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="payment_acquirer_search" model="ir.ui.view">
        <field name="name">payment.acquirer.search</field>
        <field name="model">payment.acquirer</field>
        <field name="arch" type="xml">
            <search>
                <field name="name" string="Acquirer" filter_domain="['|', ('name', 'ilike', self), ('description', 'ilike', self)]"/>
                <field name="provider"/>
                <filter name="acquirer_installed" string="Installed" domain="[('module_state', '=', 'installed')]"/>
                <group expand="0" string="Group By">
                    <filter string="Provider" name="provider" context="{'group_by': 'provider'}"/>
                    <filter string="State" name="state" context="{'group_by': 'state'}"/>
                    <filter string="Company" name="company" context="{'group_by': 'company_id'}" groups="base.group_multi_company"/>
                </group>
            </search>
        </field>
    </record>

    <record id="action_payment_acquirer" model="ir.actions.act_window">
        <field name="name">Payment Acquirers</field>
        <field name="res_model">payment.acquirer</field>
        <field name="view_mode">kanban,tree,form</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a new payment acquirer
            </p>
        </field>
    </record>

    <menuitem action="action_payment_acquirer"
              id="payment_acquirer_menu"
              parent="account.root_payment_menu"
              sequence="10"/>

</odoo>

```

## File: views\payment_icon_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_icon_form" model="ir.ui.view">
        <field name="name">payment.icon.form</field>
        <field name="model">payment.icon</field>
        <field name="arch" type="xml">
            <form string="Payment Icon">
                <sheet>
                    <field name="image" widget="image" class="oe_avatar"/>
                    <div class="oe_title">
                        <h1><field name="name" placeholder="Name"/></h1>
                    </div>
                    <notebook>
                        <page string="Acquirers list" name="acquirers">
                            <field nolabel="1" name="acquirer_ids"/>
                        </page>
                    </notebook>
                </sheet>
            </form>
        </field>
    </record>

    <record id="payment_icon_tree" model="ir.ui.view">
        <field name="name">payment.icon.tree</field>
        <field name="model">payment.icon</field>
        <field name="arch" type="xml">
            <tree>
                <field name="sequence" widget="handle"/>
                <field name="name"/>
            </tree>
        </field>
    </record>

    <record id="action_payment_icon" model="ir.actions.act_window">
        <field name="name">Payment Icons</field>
        <field name="res_model">payment.icon</field>
        <field name="view_mode">tree,form</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a payment icon
            </p>
        </field>
    </record>

    <menuitem action="action_payment_icon"
              id="payment_icon_menu"
              parent="account.root_payment_menu"
              groups="base.group_no_one"/>

</odoo>

```

## File: views\payment_portal_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- Integration of a conditional "Manage payment methods" link in /my -->
    <template id="pay_meth_link" inherit_id="portal.portal_layout">
        <xpath expr="//div[hasclass('o_portal_my_details')]" position="inside">
            <t t-set="partner" t-value="request.env.user.partner_id"/>
            <t t-set="acquirers_allowing_tokenization"
               t-value="request.env['payment.acquirer'].sudo()._get_compatible_acquirers(request.env.company.id, partner.id, force_tokenization=True, is_validation=True)"/>
            <t t-set="existing_tokens" t-value="partner.payment_token_ids + partner.commercial_partner_id.sudo().payment_token_ids"/>
            <!-- Only show the link if a token can be created or if one already exists -->
            <div t-if="acquirers_allowing_tokenization or existing_tokens"
                 class='manage_payment_method mt16'>
                <a href="/my/payment_method">Manage payment methods</a>
            </div>
        </xpath>
    </template>

    <!-- Display of /payment/pay -->
    <template id="pay">
        <!-- Variables description:
            - 'partner_is_different' - Whether the partner logged in is the one making the payment
        -->
        <t t-call="portal.frontend_layout">
            <t t-set="page_title" t-value="'Payment'"/>
            <t t-set="additional_title"><t t-esc="page_title"/></t>
            <div class="wrap">
                <div class="container">
                    <!-- Portal breadcrumb -->
                    <t t-call="payment.portal_breadcrumb"/>
                    <!-- Payment page -->
                    <div class="row">
                        <div class="col-lg-7">
                            <div t-if="not amount" class="alert alert-info">
                                There is nothing to pay.
                            </div>
                            <div t-elif="not currency" class="alert alert-warning">
                                <strong>Warning</strong> The currency is missing or incorrect.
                            </div>
                            <div t-elif="not partner_id" class="alert alert-warning">
                                <strong>Warning</strong> You must be logged in to pay.
                            </div>
                            <div t-elif="not acquirers and not tokens" class="alert alert-warning">
                                <strong>No suitable payment option could be found.</strong><br/>
                                If you believe that it is an error, please contact the website administrator.
                            </div>
                            <t t-else="">
                                <div t-if="partner_is_different" class="alert alert-warning">
                                    <strong>Warning</strong> Make sure your are logged in as the right partner before making this payment.
                                </div>
                                <t t-if="reference_prefix">
                                    <b>Reference:</b>
                                    <t t-esc="reference_prefix"/><br/>
                                </t>
                                <b>Amount:</b>
                                <t t-esc="amount"
                                   t-options="{'widget': 'monetary', 'display_currency': currency}"/>
                                <t t-call="payment.checkout"/>
                            </t>
                        </div>
                    </div>
                </div>
            </div>
        </t>
    </template>

    <!-- Display of /my/payment_methods -->
    <template id="payment_methods" name="Payment Methods">
        <t t-call="portal.frontend_layout">
            <t t-set="page_title" t-value="'Payment Methods'"/>
            <t t-set="additional_title"><t t-esc="page_title"/></t>
            <div class="wrap">
                <div class="container">
                    <!-- Portal breadcrumb -->
                    <t t-call="payment.portal_breadcrumb"/>
                    <!-- Manage page -->
                    <div class="row">
                        <div class="col-lg-7">
                            <t t-if="acquirers or tokens" t-call="payment.manage"/>
                            <div t-else="" class="alert alert-warning">
                                <p><strong>No suitable payment acquirer could be found.</strong></p>
                                <p>If you believe that it is an error, please contact the website administrator.</p>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </t>
    </template>

    <!-- Display of /payment/status -->
    <template id="payment_status" name="Payment Status">
        <t t-call="portal.frontend_layout">
            <t t-set="page_title" t-value="'Payment Status'"/>
            <t t-set="additional_title"><t t-esc="page_title"/></t>
            <div class="wrap">
                <div class="container">
                    <!-- Portal breadcrumb -->
                    <t t-call="payment.portal_breadcrumb"/>
                    <!-- Status page -->
                    <div name="o_payment_status">
                        <div name="o_payment_status_content"
                             class="col-sm-6 col-sm-offset-3">
                            <!-- The content is generated in JavaScript -->
                        </div>
                    </div>
                </div>
            </div>
        </t>
    </template>

    <!-- Display of /payment/confirmation -->
    <template id="confirm">
        <!-- Variables description:
            - 'tx' - The transaction to display
            - 'message' - The acquirer message configured for the given transaction state
            - 'status' - The alert class to use for the message
        -->
        <t t-call="portal.frontend_layout">
            <t t-set="page_title" t-value="'Payment Confirmation'"/>
            <t t-set="additional_title"><t t-esc="page_title"/></t>
            <div class="wrap">
                <div class="container">
                    <!-- Portal breadcrumb -->
                    <t t-call="payment.portal_breadcrumb"/>
                    <!-- Confirmation page -->
                    <div class="row">
                        <div class="col-lg-6">
                            <div>
                                <t t-call="payment.transaction_status"/>
                                <div class="form-group row">
                                    <label for="form_partner_name" class="col-md-3 col-form-label">
                                        From
                                    </label>
                                    <span name="form_partner_name"
                                          class="col-md-9 col-form-label"
                                          t-esc="tx.partner_name"/>
                                </div>
                                <hr/>
                                <div class="form-group row">
                                    <label for="form_reference" class="col-md-3 col-form-label">
                                        Reference
                                    </label>
                                    <span name="form_reference"
                                          class="col-md-9 col-form-label"
                                          t-esc="tx.reference"/>
                                </div>
                                <hr/>
                                <div class="form-group row">
                                    <label for="form_amount" class="col-md-3 col-form-label">
                                        Amount
                                    </label>
                                    <span name="form_amount"
                                          class="col-md-9 col-form-label"
                                          t-esc="tx.amount"
                                          t-options="{'widget': 'monetary', 'display_currency': tx.currency_id}"/>
                                </div>
                                <hr/>
                                <div class="row">
                                    <div class="col-md-5 text-muted">
                                        Processed by <t t-esc="tx.acquirer_id.sudo().name"/>
                                    </div>
                                    <div class="col-md-4 offset-md-3 mt-2 pl-0">
                                        <a role="button"
                                           t-attf-class="btn btn-#{status} float-right"
                                           href="/my/home">
                                            <i class="fa fa-arrow-circle-right"/> Back to My Account
                                        </a>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </t>
    </template>

    <!-- Breadcrumb for the portal -->
    <template id="portal_breadcrumb">
        <!-- Variables description:
            - 'page_title' - The title of the breadcrumb item
        -->
        <div class="row">
            <div class="col-md-6">
                <ol class="breadcrumb mt8">
                    <li class="breadcrumb-item">
                        <a href="/my/home">
                            <i class="fa fa-home"
                               role="img"
                               title="Home"
                               aria-label="Home"/>
                        </a>
                    </li>
                    <li class="breadcrumb-item"><t t-esc="page_title"/></li>
                </ol>
            </div>
        </div>
    </template>

</odoo>

```

## File: views\payment_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- Checkout form -->
    <template id="checkout" name="Payment Checkout">
        <!-- Variables description:
            - 'acquirers' - The payment acquirers compatible with the current transaction
            - 'tokens' - The payment tokens of the current partner and payment acquirers
            - 'default_token_id' - The id of the token that should be pre-selected. Optional
            - 'fees_by_acquirer' - The dict of transaction fees for each acquirer. Optional
            - 'show_tokenize_input' - Whether the option to save the payment method is shown
            - 'reference_prefix' - The custom prefix to compute the full transaction reference
            - 'amount' - The amount to pay. Optional (sale_subscription)
            - 'currency' - The currency of the transaction, as a `res.currency` record
            - 'partner_id' - The id of the partner on behalf of whom the payment should be made
            - 'access_token' - The access token used to authenticate the partner.
            - 'transaction_route' - The route used to create a transaction when the user clicks Pay
            - 'landing_route' - The route the user is redirected to after the transaction
            - 'footer_template_id' - The template id for the submit button. Optional
            - 'invoice_id' - The id of the account move being paid. Optional
        -->
        <form name="o_payment_checkout"
              class="o_payment_form mt-3 clearfix"
              t-att-data-reference-prefix="reference_prefix"
              t-att-data-amount="amount"
              t-att-data-currency-id="currency and currency.id"
              t-att-data-partner-id="partner_id"
              t-att-data-access-token="access_token"
              t-att-data-transaction-route="transaction_route"
              t-att-data-landing-route="landing_route"
              t-att-data-allow-token-selection="True"
              t-att-data-invoice-id="invoice_id">

            <t t-set="acquirer_count" t-value="len(acquirers) if acquirers else 0"/>
            <t t-set="token_count" t-value="len(tokens) if tokens else 0"/>
            <!-- Check the radio button of the default token, if set, or of the first acquirer if
                 it is the only payment option -->
            <t t-set="default_payment_option_id"
               t-value="default_token_id if default_token_id and token_count > 0
                        else acquirers[0].id if acquirer_count == 1 and token_count == 0
                        else None"/>
            <t t-set="fees_by_acquirer" t-value="fees_by_acquirer or dict()"/>
            <t t-set="footer_template_id"
               t-value="footer_template_id or 'payment.footer'"/>

            <div class="card">
                <!-- === Acquirers === -->
                <t t-foreach="acquirers" t-as="acquirer">
                    <div name="o_payment_option_card" class="card-body o_payment_option_card">
                        <label>
                            <!-- === Radio button === -->
                            <!-- Only shown if linked to the only payment option -->
                            <input name="o_payment_radio"
                                   type="radio"
                                   t-att-checked="acquirer.id == default_payment_option_id"
                                   t-att-class="'' if acquirer_count + token_count > 1 else 'd-none'"
                                   t-att-data-payment-option-id="acquirer.id"
                                   t-att-data-provider="acquirer.provider"
                                   data-payment-option-type="acquirer"/>
                            <!-- === Acquirer name === -->
                            <span class="payment_option_name">
                                <b t-esc="acquirer.display_as or acquirer.name"/>
                            </span>
                            <!-- === "Test Mode" badge === -->
                            <span t-if="acquirer.state == 'test'"
                                  class="badge-pill badge-warning ml-1">
                                Test Mode
                            </span>
                            <!-- === Extra fees badge === -->
                            <t t-if="fees_by_acquirer.get(acquirer)">
                                <span class="badge-pill badge-secondary ml-1">
                                    + <t t-esc="fees_by_acquirer.get(acquirer)"
                                         t-options="{'widget': 'monetary', 'display_currency': currency}"/>
                                    Fees
                                </span>
                            </t>
                        </label>
                        <!-- === Payment icon list === -->
                        <t t-call="payment.icon_list"/>
                        <!-- === Help message === -->
                        <div t-if="not is_html_empty(acquirer.pre_msg)"
                             t-out="acquirer.pre_msg"
                             class="text-muted ml-3"/>
                    </div>
                    <!-- === Acquirer inline form === -->
                    <div t-attf-id="o_payment_acquirer_inline_form_{{acquirer.id}}"
                         name="o_payment_inline_form"
                         class="card-footer d-none">
                        <!-- === Inline form content (filled by acquirer) === -->
                        <t t-if="acquirer.sudo()._should_build_inline_form(is_validation=False)">
                            <t t-set="inline_form_xml_id"
                               t-value="acquirer.sudo().inline_form_view_id.xml_id"/>
                            <div t-if="inline_form_xml_id" class="clearfix">
                                <t t-call="{{inline_form_xml_id}}">
                                    <t t-set="acquirer_id" t-value="acquirer.id"/>
                                </t>
                            </div>
                        </t>
                        <!-- === "Save my payment details" checkbox === -->
                        <!-- Only included if partner is known and if the choice is given -->
                        <t t-set="tokenization_required"
                           t-value="acquirer._is_tokenization_required(provider=acquirer.provider)"/>
                        <label t-if="show_tokenize_input and acquirer.allow_tokenization and not tokenization_required">
                            <input name="o_payment_save_as_token" type="checkbox"/>
                            Save my payment details
                        </label>
                    </div>
                </t>
                <!-- === Tokens === -->
                <t t-foreach="tokens" t-as="token">
                    <div name="o_payment_option_card" class="card-body o_payment_option_card">
                        <label>
                            <!-- === Radio button === -->
                            <input name="o_payment_radio"
                                   type="radio"
                                   t-att-checked="token.id == default_payment_option_id"
                                   t-att-data-payment-option-id="token.id"
                                   t-att-data-provider="token.provider"
                                   data-payment-option-type="token"/>
                            <!-- === Token name === -->
                            <span class="payment_option_name" t-esc="token.name"/>
                            <!-- === "V" check mark === -->
                            <t t-call="payment.verified_token_checkmark"/>
                        </label>
                    </div>
                    <!-- === Token inline form === -->
                    <div t-attf-id="o_payment_token_inline_form_{{token.id}}"
                         name="o_payment_inline_form"
                         class="card-footer d-none"/>
                </t>
            </div>
            <!-- === "Pay" button === -->
            <t t-call="{{footer_template_id}}">
                <t t-set="label">Pay</t>
                <t t-set="icon_class" t-value="'fa-lock'"/>
            </t>
        </form>
    </template>

    <!-- Manage (token create and deletion) form -->
    <template id="manage" name="Payment Manage">
        <!-- Variables description:
            - 'acquirers' - The payment acquirers supporting tokenization
            - 'tokens' - The set of payment tokens of the current partner
            - 'default_token_id' - The id of the token that should be pre-selected. Optional
            - 'reference_prefix' - The custom prefix to compute the full transaction reference
            - 'partner_id' - The id of the partner managing the tokens
            - 'access_token' - The access token used to authenticate the partner.
            - 'transaction_route' - The route used to create a validation transaction
            - 'assign_token_route' - The route to call to assign a token to a record. If set, it
                                     enables the token assignation mechanisms: creation of a new
                                     token through a refunded transaction and assignation of an
                                     existing token
            - 'landing_route' - The route the user is redirected to at then end of the flow
            - 'footer_template_id' - The template id for the submit button. Optional
        -->
        <form name="o_payment_manage"
              class="o_payment_form mt-3 clearfix"
              t-att-data-reference-prefix="reference_prefix"
              t-att-data-partner-id="partner_id"
              t-att-data-access-token="access_token"
              t-att-data-invoice-id="invoice_id"
              t-att-data-transaction-route="transaction_route"
              t-att-data-assign-token-route="assign_token_route"
              t-att-data-landing-route="landing_route"
              t-att-data-allow-token-selection="bool(assign_token_route)">
            <t t-set="acquirer_count" t-value="len(acquirers) if acquirers else 0"/>
            <t t-set="token_count" t-value="len(tokens) if tokens else 0"/>
            <t t-set="no_selectable_token" t-value="token_count == 0 or not assign_token_route"/>
            <t t-set="default_payment_option_id"
               t-value="default_token_id if default_token_id and token_count > 0
                        else acquirers[0].id if acquirer_count == 1 and no_selectable_token
                        else None"/>
            <t t-set="footer_template_id"
               t-value="footer_template_id or 'payment.footer'"/>
            <div class="card">
                <!-- === Acquirers === -->
                <t t-foreach="acquirers" t-as="acquirer">
                    <div name="o_payment_option_card" class="card-body o_payment_option_card">
                        <label>
                            <!-- === Radio button === -->
                            <!-- Only shown if linked to the only payment option -->
                            <input name="o_payment_radio"
                                   type="radio"
                                   t-att-checked="acquirer.id == default_payment_option_id"
                                   t-att-class="'' if acquirer_count + token_count > 1 else 'd-none'"
                                   t-att-data-payment-option-id="acquirer.id"
                                   t-att-data-provider="acquirer.provider"
                                   data-payment-option-type="acquirer"/>
                            <!-- === Acquirer name === -->
                            <span class="payment_option_name">
                                <b><t t-esc="acquirer.display_as or acquirer.name"/></b>
                            </span>
                            <!-- === "Test Mode" badge === -->
                            <span t-if="acquirer.state == 'test'"
                                  class="badge-pill badge-warning"
                                  style="margin-left:5px">
                                Test Mode
                            </span>
                        </label>
                        <!-- === Payment icon list === -->
                        <t t-call="payment.icon_list"/>
                        <!-- === Help message === -->
                        <div t-if="not is_html_empty(acquirer.pre_msg)"
                             t-out="acquirer.pre_msg"
                             class="text-muted ml-3"/>
                    </div>
                    <!-- === Acquirer inline form === -->
                    <t t-if="acquirer.sudo()._should_build_inline_form(is_validation=True)">
                        <div t-attf-id="o_payment_acquirer_inline_form_{{acquirer.id}}"
                             name="o_payment_inline_form"
                             class="card-footer d-none">
                            <!-- === Inline form content (filled by acquirer) === -->
                            <t t-set="inline_form_xml_id"
                               t-value="acquirer.sudo().inline_form_view_id.xml_id"/>
                            <div t-if="inline_form_xml_id" class="clearfix">
                                <t t-call="{{inline_form_xml_id}}">
                                    <t t-set="acquirer_id" t-value="acquirer.id"/>
                                </t>
                            </div>
                        </div>
                    </t>
                </t>
                <!-- === Tokens === -->
                <t t-foreach="tokens" t-as="token">
                    <div name="o_payment_option_card" class="card-body o_payment_option_card">
                        <label>
                            <!-- === Radio button === -->
                            <!-- Only shown if 'assign_token_route' is set -->
                            <input name="o_payment_radio"
                                   type="radio"
                                   t-att-checked="token.id == default_payment_option_id"
                                   t-att-class="'' if bool(assign_token_route) else 'd-none'"
                                   t-att-data-payment-option-id="token.id"
                                   t-att-data-provider="token.provider"
                                   data-payment-option-type="token"/>
                            <!-- === Token name === -->
                            <span class="payment_option_name" t-esc="token.name"/>
                            <!-- === "V" check mark === -->
                            <t t-call="payment.verified_token_checkmark"/>
                        </label>
                        <!-- === "Delete" token button === -->
                        <button name="o_payment_delete_token"
                                class="btn btn-primary btn-sm float-right">
                            <i class="fa fa-trash"/> Delete
                        </button>
                    </div>
                    <!-- === Token inline form === -->
                    <div t-attf-id="o_payment_token_inline_form_{{token.id}}"
                         name="o_payment_inline_form"
                         class="card-footer d-none"/>
                </t>
            </div>
            <!-- === "Save Payment Method" button === -->
            <t t-call="{{footer_template_id}}">
                <t t-set="label">Save Payment Method</t>
                <t t-set="icon_class" t-value="'fa-plus-circle'"/>
            </t>
        </form>
    </template>

    <!-- Expandable payment icon list -->
    <template id="icon_list" name="Payment Icon List">
        <ul class="payment_icon_list float-right list-inline" data-max-icons="3">
            <t t-set="icon_index" t-value="0"/>
            <t t-set="MAX_ICONS" t-value="3"/>
            <!-- === Icons === -->
            <!-- Only shown if in the first 3 icons -->
            <t t-foreach="acquirer.payment_icon_ids.filtered(lambda r: r.image_payment_form)" t-as="icon">
                <li t-attf-class="list-inline-item{{'' if (icon_index &lt; MAX_ICONS) else ' d-none'}}">
                    <span t-esc="icon.image_payment_form"
                          t-options="{'widget': 'image', 'alt-field': 'name'}"
                          data-toggle="tooltip"
                          t-att-title="icon.name"/>
                </li>
                <t t-set="icon_index" t-value="icon_index + 1"/>
            </t>
            <t t-if="icon_index >= MAX_ICONS">
                <!-- === "show more" button === -->
                <!-- Only displayed if too many payment icons -->
                <li style="display:block;" class="list-inline-item">
                    <span class="float-right more_option text-info">
                        <a name="o_payment_icon_more"
                           data-toggle="tooltip"
                           t-att-title="', '.join([icon.name for icon in acquirer.payment_icon_ids[MAX_ICONS:]])">
                            show more
                        </a>
                    </span>
                </li>
                <!-- === "show less" button === -->
                <!-- Only displayed when "show more" is clicked -->
                <li style="display:block;" class="list-inline-item d-none">
                    <span class="float-right more_option text-info">
                        <a name="o_payment_icon_less">show less</a>
                    </span>
                </li>
            </t>
        </ul>
    </template>

    <!-- Verified token checkmark -->
    <template id="verified_token_checkmark" name="Payment Verified Token Checkmark">
        <t t-if="0" name="payment_test_hook"/>
        <t t-else="">
            <i t-if="token.verified" class="fa fa-check text-success"
                title="This payment method has been verified by our system."
                role="img"
                aria-label="Ok"/>
            <i t-else="" class="fa fa-check text-muted"
                title="This payment method has not been verified by our system."
                role="img"
                aria-label="Not verified"/>
        </t>
    </template>

    <!-- Generic footer for payment forms -->
    <template id="footer" name="Payment Footer">
        <!-- Variables description:
            - 'label' - The label for the submit button
            - 'icon_class' - The Font Awesome icon class (e.g. 'fa-lock') for the submit button
        -->
        <div class="float-right mt-2">
            <button name="o_payment_submit_button"
                    type="submit"
                    class="btn btn-primary btn-lg mb8 mt8"
                    disabled="true"
                    t-att-data-icon-class="icon_class">
                <i t-attf-class="fa {{icon_class}}"/> <t t-esc="label"/>
            </button>
        </div>
    </template>

    <!-- Transaction status in portal -->
    <template id="transaction_status">
        <!-- Variables description:
            - 'tx' - The transaction whose status must be displayed
        -->
        <div t-if="tx.state == 'pending' and not is_html_empty(tx.acquirer_id.sudo().pending_msg)"
             class="alert alert-warning alert-dismissible">
            <span t-out="tx.acquirer_id.sudo().pending_msg"/>
            <button class="close" data-dismiss="alert" title="Dismiss">×</button>
        </div>
        <div t-elif="tx.state == 'authorized' and not is_html_empty(tx.acquirer_id.sudo().auth_msg)"
             class="alert alert-success alert-dismissible">
            <span t-out="tx.acquirer_id.sudo().auth_msg"/>
            <button class="close" data-dismiss="alert" title="Dismiss">×</button>
        </div>
        <div t-elif="tx.state == 'done' and not is_html_empty(tx.acquirer_id.sudo().done_msg)"
             class="alert alert-success alert-dismissible">
            <span t-out="tx.acquirer_id.sudo().done_msg"/>
            <button class="close" data-dismiss="alert" title="Dismiss">×</button>
        </div>
        <div t-elif="tx.state == 'cancel' and not is_html_empty(tx.acquirer_id.sudo().cancel_msg)"
             class="alert alert-danger alert-dismissible">
            <span t-out="tx.acquirer_id.sudo().cancel_msg"/>
            <button class="close" data-dismiss="alert" title="Dismiss">×</button>
        </div>
        <span t-if="tx.state_message" t-esc="tx.state_message"/>
    </template>

</odoo>

```

## File: views\payment_token_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_token_form" model="ir.ui.view">
        <field name="name">payment.token.form</field>
        <field name="model">payment.token</field>
        <field name="arch" type="xml">
            <form string="Payment Tokens" create="false" editable="bottom">
                <sheet>
                    <field name="active" invisible="1"/>
                    <div class="oe_button_box" name="button_box">
                        <button class="oe_stat_button"
                                name="%(action_payment_transaction_linked_to_token)d"
                                type="action" icon="fa-money" string="Payments">
                        </button>
                    </div>
                    <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                    <group>
                        <field name="name"/>
                        <field name="partner_id" />
                    </group>
                    <group>
                        <field name="acquirer_id"/>
                        <field name="acquirer_ref"/>
                        <field name="company_id" groups="base.group_multi_company"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="payment_token_list" model="ir.ui.view">
        <field name="name">payment.token.list</field>
        <field name="model">payment.token</field>
        <field name="arch" type="xml">
            <tree string="Payment Tokens">
                <field name="name"/>
                <field name="partner_id"/>
                <field name="acquirer_id" readonly="1"/>
                <field name="acquirer_ref" readonly="1"/>
                <field name="company_id" groups="base.group_multi_company" optional="show"/>
            </tree>
        </field>
    </record>

    <record id="payment_token_search" model="ir.ui.view">
        <field name="name">payment.token.search</field>
        <field name="model">payment.token</field>
        <field name="arch" type="xml">
            <search string="Payment Tokens">
                <field name="partner_id"/>
                <separator/>
                <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
                <group expand="1" string="Group By">
                    <filter string="Acquirer" name="acquirer_id" context="{'group_by': 'acquirer_id'}"/>
                    <filter string="Partner" name="partner_id" context="{'group_by': 'partner_id'}"/>
                    <filter string="Company" name="company" context="{'group_by': 'company_id'}" groups="base.group_multi_company"/>
                </group>
            </search>
        </field>
    </record>

    <record id="action_payment_token" model="ir.actions.act_window">
        <field name="name">Payment Tokens</field>
        <field name="res_model">payment.token</field>
        <field name="view_mode">tree,form</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a new payment token
            </p>
        </field>
    </record>

    <menuitem action="action_payment_token"
              id="payment_token_menu"
              parent="account.root_payment_menu"
              groups="base.group_no_one"/>

</odoo>

```

## File: views\payment_transaction_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_transaction_form" model="ir.ui.view">
        <field name="name">payment.transaction.form</field>
        <field name="model">payment.transaction</field>
        <field name="arch" type="xml">
            <form string="Payment Transactions" create="false" edit="false">
                <header>
                    <button type="object" name="action_capture" states="authorized" string="Capture Transaction" class="oe_highlight"/>
                    <button type="object" name="action_void" states="authorized" string="Void Transaction"
                            confirm="Are you sure you want to void the authorized transaction? This action can't be undone."/>
                    <field name="state" widget="statusbar"/>
                </header>
                <sheet>
                    <div class="oe_button_box" name="button_box">
                        <button name="action_view_invoices" type="object"
                                class="oe_stat_button" icon="fa-money"
                                attrs="{'invisible': [('invoices_count', '=', 0)]}">
                            <field name="invoices_count" widget="statinfo" string="Invoice(s)"/>
                        </button>
                        <button name="action_view_refunds"
                                type="object"
                                class="oe_stat_button"
                                icon="fa-money"
                                attrs="{'invisible': [('refunds_count', '=', 0)]}">
                            <field name="refunds_count" widget="statinfo" string="Refunds"/>
                        </button>
                    </div>
                    <group>
                        <group name="transaction_details">
                            <field name="reference"/>
                            <field name="payment_id"/>
                            <field name="source_transaction_id"
                                   attrs="{'invisible': [('source_transaction_id', '=', False)]}"/>
                            <field name="amount"/>
                            <field name="fees" attrs="{'invisible': [('fees', '=', 0.0)]}"/>
                            <field name="currency_id" invisible="1"/>
                            <field name="acquirer_id"/>
                            <field name="company_id" groups="base.group_multi_company"/>
                            <!-- Used by some acquirer-specific views -->
                            <field name="provider" invisible="1"/>
                            <field name="acquirer_reference"/>
                            <field name="token_id" attrs="{'invisible': [('token_id', '=', False)]}"/>
                            <field name="create_date"/>
                            <field name="last_state_change"/>
                        </group>
                        <group name="transaction_partner">
                            <field name="partner_id" widget="res_partner_many2one"/>
                            <label for="partner_address" string="Address"/>
                            <div class="o_address_format">
                                <field name="partner_address" placeholder="Address" class="o_address_street"/>
                                <field name="partner_city" placeholder="City" class="o_address_city"/>
                                <field name="partner_state_id" placeholder="State" class="o_address_state" options="{'no_open': True}"/>
                                <field name="partner_zip" placeholder="ZIP" class="o_address_zip"/>
                                <field name="partner_country_id" placeholder="Country" class="o_address_country" options="{'no_open': True}"/>
                            </div>
                            <field name="partner_email" widget="email"/>
                            <field name="partner_phone" widget="phone"/>
                            <field name="partner_lang"/>
                        </group>
                    </group>
                    <group string="Message" attrs="{'invisible': [('state_message', '=', False)]}">
                        <field name="state_message" nolabel="1"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="payment_transaction_list" model="ir.ui.view">
        <field name="name">payment.transaction.list</field>
        <field name="model">payment.transaction</field>
        <field name="arch" type="xml">
            <tree string="Payment Transactions" create="false">
                <field name="reference"/>
                <field name="create_date"/>
                <field name="acquirer_id"/>
                <field name="partner_id"/>
                <field name="partner_name"/>
                <!-- Needed to display the currency of the amounts -->
                <field name="currency_id" invisible="1"/>
                <field name="amount"/>
                <field name="fees"/>
                <field name="state"/>
                <field name="company_id" groups="base.group_multi_company" optional="show"/>
            </tree>
        </field>
    </record>

    <record id="payment_transaction_kanban" model="ir.ui.view">
        <field name="name">payment.transaction.kanban</field>
        <field name="model">payment.transaction</field>
        <field name="arch" type="xml">
            <kanban class="o_kanban_mobile" create="false">
                <templates>
                    <t t-name="kanban-box">
                        <div t-attf-class="oe_kanban_content oe_kanban_global_click">
                            <div class="row">
                                <div class="col-6">
                                    <strong><field name="reference"/></strong>
                                </div>
                                <div class="col-6">
                                    <span><field name="partner_name"/></span>
                                </div>
                                <div class="col-6">
                                    <span class="float-right">
                                        <field name="amount"/>
                                        <field name="currency_id" invisible="1"/>
                                    </span>
                                </div>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="payment_transaction_search" model="ir.ui.view">
        <field name="name">payment.transaction.search</field>
        <field name="model">payment.transaction</field>
        <field name="arch" type="xml">
            <search>
                <field name="reference"/>
                <field name="acquirer_id"/>
                <field name="partner_id"/>
                <field name="partner_name"/>
                <group expand="1" string="Group By">
                    <filter string="Acquirer" name="acquirer_id" context="{'group_by': 'acquirer_id'}"/>
                    <filter string="Partner" name="partner_id" context="{'group_by': 'partner_id'}"/>
                    <filter string="Status" name="state" context="{'group_by': 'state'}"/>
                    <filter string="Company" name="company" context="{'group_by': 'company_id'}" groups="base.group_multi_company"/>
                </group>
            </search>
        </field>
    </record>

    <record id="action_payment_transaction" model="ir.actions.act_window">
        <field name="name">Payment Transactions</field>
        <field name="res_model">payment.transaction</field>
        <field name="view_mode">tree,kanban,form</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_neutral_face">
                There are no transactions to show
            </p>
        </field>
    </record>

    <record id="action_payment_transaction_linked_to_token" model="ir.actions.act_window">
        <field name="name">Payment Transactions Linked To Token</field>
        <field name="res_model">payment.transaction</field>
        <field name="view_mode">tree,form</field>
        <field name="domain">[('token_id','=', active_id)]</field>
        <field name="context">{'create': False}</field>
    </record>

    <menuitem action="action_payment_transaction"
              id="payment_transaction_menu"
              parent="account.root_payment_menu"
              groups="base.group_no_one"
              sequence="20"/>

</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0"?>
<odoo>

    <!-- Add credit card to res.partner -->
    <record id="view_partners_form_payment_defaultcreditcard" model="ir.ui.view">
        <field name="name">view.res.partner.form.payment.defaultcreditcard</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="base.view_partner_form"/>
        <field name="priority" eval="15"/>
        <field name="arch" type="xml">
            <div name="button_box" position="inside">
                <button type="action" class="oe_stat_button"
                        icon="fa-credit-card-alt"
                        name="%(payment.action_payment_token)d"
                        context="{'search_default_partner_id': active_id, 'create': False, 'edit': False}"
                        attrs="{'invisible': [('payment_token_count', '=', 0)]}">
                    <div class="o_form_field o_stat_info">
                        <span class="o_stat_value">
                            <field name="payment_token_count" widget="statinfo" nolabel="1"/>
                        </span>
                        <span class="o_stat_text">Saved Payment Methods</span>
                    </div>
                </button>
            </div>
        </field>
    </record>

</odoo>

```

## File: wizards\account_payment_register.py

```python
from odoo import api, Command, fields, models


class AccountPaymentRegister(models.TransientModel):
    _inherit = 'account.payment.register'

    # == Business fields ==
    payment_token_id = fields.Many2one(
        comodel_name='payment.token',
        string="Saved payment token",
        store=True, readonly=False,
        compute='_compute_payment_token_id',
        domain='''[
            ('id', 'in', suitable_payment_token_ids),
        ]''',
        help="Note that tokens from acquirers set to only authorize transactions (instead of capturing the amount) are "
             "not available.")

    # == Display purpose fields ==
    suitable_payment_token_ids = fields.Many2many(
        comodel_name='payment.token',
        compute='_compute_suitable_payment_token_ids'
    )
    use_electronic_payment_method = fields.Boolean(
        compute='_compute_use_electronic_payment_method',
        help='Technical field used to hide or show the payment_token_id if needed.'
    )
    payment_method_code = fields.Char(
        related='payment_method_line_id.code')

    # -------------------------------------------------------------------------
    # COMPUTE METHODS
    # -------------------------------------------------------------------------

    @api.depends('payment_method_line_id')
    def _compute_suitable_payment_token_ids(self):
        for wizard in self:
            if wizard.can_edit_wizard and wizard.use_electronic_payment_method:
                related_partner_ids = (
                        wizard.partner_id
                        | wizard.partner_id.commercial_partner_id
                        | wizard.partner_id.commercial_partner_id.child_ids
                )._origin

                wizard.suitable_payment_token_ids = self.env['payment.token'].sudo().search([
                    ('company_id', '=', wizard.company_id.id),
                    ('acquirer_id.capture_manually', '=', False),
                    ('partner_id', 'in', related_partner_ids.ids),
                    ('acquirer_id', '=', wizard.payment_method_line_id.payment_acquirer_id.id),
                ])
            else:
                wizard.suitable_payment_token_ids = [Command.clear()]

    @api.depends('payment_method_line_id')
    def _compute_use_electronic_payment_method(self):
        for wizard in self:
            # Get a list of all electronic payment method codes.
            # These codes are comprised of the providers of each payment acquirer.
            codes = [key for key in dict(self.env['payment.acquirer']._fields['provider']._description_selection(self.env))]
            wizard.use_electronic_payment_method = wizard.payment_method_code in codes

    @api.onchange('can_edit_wizard', 'payment_method_line_id', 'journal_id')
    def _compute_payment_token_id(self):
        codes = [key for key in dict(self.env['payment.acquirer']._fields['provider']._description_selection(self.env))]
        for wizard in self:
            related_partner_ids = (
                    wizard.partner_id
                    | wizard.partner_id.commercial_partner_id
                    | wizard.partner_id.commercial_partner_id.child_ids
            )._origin
            if wizard.can_edit_wizard \
                    and wizard.payment_method_line_id.code in codes \
                    and wizard.journal_id \
                    and related_partner_ids:

                wizard.payment_token_id = self.env['payment.token'].sudo().search([
                    ('company_id', '=', wizard.company_id.id),
                    ('partner_id', 'in', related_partner_ids.ids),
                    ('acquirer_id.capture_manually', '=', False),
                    ('acquirer_id', '=', wizard.payment_method_line_id.payment_acquirer_id.id),
                 ], limit=1)
            else:
                wizard.payment_token_id = False

    # -------------------------------------------------------------------------
    # BUSINESS METHODS
    # -------------------------------------------------------------------------

    def _create_payment_vals_from_wizard(self):
        # OVERRIDE
        payment_vals = super()._create_payment_vals_from_wizard()
        payment_vals['payment_token_id'] = self.payment_token_id.id
        return payment_vals

```

## File: wizards\account_payment_register_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_account_payment_register_form_inherit_payment" model="ir.ui.view">
        <field name="name">account.payment.register.form.inherit.payment</field>
        <field name="model">account.payment.register</field>
        <field name="inherit_id" ref="account.view_account_payment_register_form"/>
        <field name="arch" type="xml">
            <field name="payment_method_line_id" position="after">
                <field name="payment_method_code" invisible="1"/>
                <field name="suitable_payment_token_ids" invisible="1"/>
                <field name="use_electronic_payment_method" invisible="1"/>
                <field name="payment_token_id"
                       options="{'no_create': True}"
                       attrs="{'invisible': ['|', ('use_electronic_payment_method', '!=', True), '|', ('can_edit_wizard', '=', False), '&amp;', ('can_group_payments', '=', True), ('group_payment', '=', False)]}"/>
            </field>
        </field>
    </record>

</odoo>

```

## File: wizards\payment_acquirer_onboarding_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- onboarding step -->
    <template id="onboarding_payment_acquirer_step">
        <t t-call="base.onboarding_step">
            <t t-set="title">Online Payments</t>
            <t t-set="description">Enable credit &amp; debit card payments supported by Stripe</t>
            <t t-set="btn_text">Activate Stripe</t>
            <t t-set="done_text">Online payments enabled</t>
            <t t-set="method" t-value="'action_open_payment_onboarding_payment_acquirer'" />
            <t t-set="model" t-value="'res.company'" />
            <t t-set="state" t-value="state.get('payment_acquirer_onboarding_state')" />
        </t>
    </template>

    <record model="ir.ui.view" id="payment_acquirer_onboarding_wizard_form">
        <field name="name">payment.acquirer.onboarding.wizard.form</field>
        <field name="model">payment.acquirer.onboarding.wizard</field>
        <field name="arch" type="xml">
            <form string="Choose a payment method" class="o_onboarding_payment_acquirer_wizard">
                <div class="container">
                    <div class="row align-items-start">
                        <div class="col col-4" name="left-column">
                            <group>
                                <field name="payment_method" widget="radio" nolabel="1"/>
                            </group>
                        </div>
                        <div class="col" name="right-column">
                            <div attrs="{'invisible': [('payment_method', '!=', 'paypal')]}">
                                <group>
                                    <field name="paypal_email_account" attrs="{'required': [('payment_method', '=', 'paypal')]}" string="Email"/>
                                    <field name="paypal_pdt_token" password="True" attrs="{'required': [('payment_method', '=', 'paypal')]}" />
                                </group>
                                <p>
                                    <a href="https://www.odoo.com/documentation/15.0/applications/general/payment_acquirers/paypal.html" target="_blank">
                                        <span><i class="fa fa-arrow-right"/> How to configure your PayPal account</span>
                                    </a>
                                </p>
                            </div>
                            <div invisible="1">
                                <group>
                                    <field name="stripe_secret_key" password="True"/>
                                    <field name="stripe_publishable_key" password="True"/>
                                </group>
                                <p>
                                    <a href="https://dashboard.stripe.com/account/apikeys" target="_blank">
                                        <span><i class="fa fa-arrow-right"/> Get my Stripe keys</span>
                                    </a>
                                </p>
                            </div>

                            <div attrs="{'invisible': [('payment_method', '!=', 'other')]}">
                                <button type="object" name="action_onboarding_other_payment_acquirer">
                                    Check here
                                </button>
                                to choose another payment method.
                            </div>
                            <div attrs="{'invisible': [('payment_method', '!=', 'manual')]}">
                                <group>
                                    <field name="manual_name" attrs="{'required': [('payment_method', '=', 'manual')]}"/>
                                    <field name="journal_name" attrs="{'required': [('payment_method', '=', 'manual')]}"/>
                                    <field name="acc_number" attrs="{'required': [('payment_method', '=', 'manual')]}"/>
                                    <field name="manual_post_msg" attrs="{'required': [('payment_method', '=', 'manual')]}"/>
                                </group>
                            </div>
                        </div>
                    </div>
                </div>
                <footer>
                    <button name="add_payment_methods" string="Apply" class="oe_highlight"
                            type="object" data-hotkey="q" />
                    <button special="cancel" data-hotkey="z" string="Cancel" />
                </footer>
            </form>
        </field>
    </record>

    <record id="action_open_payment_onboarding_payment_acquirer_wizard" model="ir.actions.act_window">
        <field name="name">Choose a payment method</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">payment.acquirer.onboarding.wizard</field>
        <field name="view_mode">form</field>
        <field name="view_id" ref="payment.payment_acquirer_onboarding_wizard_form" />
        <field name="target">new</field>
    </record>
</odoo>

```

## File: wizards\payment_acquirer_onboarding_wizard.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.exceptions import UserError


class PaymentWizard(models.TransientModel):
    _name = 'payment.acquirer.onboarding.wizard'
    _description = 'Payment acquire onboarding wizard'

    payment_method = fields.Selection([
        ('paypal', "PayPal"),
        ('stripe', "Credit card (via Stripe)"),
        ('other', "Other payment acquirer"),
        ('manual', "Custom payment instructions"),
    ], string="Payment Method", default=lambda self: self._get_default_payment_acquirer_onboarding_value('payment_method'))

    paypal_user_type = fields.Selection([
        ('new_user', "I don't have a Paypal account"),
        ('existing_user', 'I have a Paypal account')], string="Paypal User Type", default='new_user')
    paypal_email_account = fields.Char("Email", default=lambda self: self._get_default_payment_acquirer_onboarding_value('paypal_email_account'))
    paypal_seller_account = fields.Char("Merchant Account ID")
    paypal_pdt_token = fields.Char("PDT Identity Token", default=lambda self: self._get_default_payment_acquirer_onboarding_value('paypal_pdt_token'))

    stripe_secret_key = fields.Char(default=lambda self: self._get_default_payment_acquirer_onboarding_value('stripe_secret_key'))
    stripe_publishable_key = fields.Char(default=lambda self: self._get_default_payment_acquirer_onboarding_value('stripe_publishable_key'))

    manual_name = fields.Char("Method",  default=lambda self: self._get_default_payment_acquirer_onboarding_value('manual_name'))
    journal_name = fields.Char("Bank Name", default=lambda self: self._get_default_payment_acquirer_onboarding_value('journal_name'))
    acc_number = fields.Char("Account Number",  default=lambda self: self._get_default_payment_acquirer_onboarding_value('acc_number'))
    manual_post_msg = fields.Html("Payment Instructions")

    _data_fetched = fields.Boolean(store=False)

    @api.onchange('journal_name', 'acc_number')
    def _set_manual_post_msg_value(self):
        self.manual_post_msg = _(
            '<h3>Please make a payment to: </h3><ul><li>Bank: %s</li><li>Account Number: %s</li><li>Account Holder: %s</li></ul>',
            self.journal_name or _("Bank"),
            self.acc_number or _("Account"),
            self.env.company.name
        )

    _payment_acquirer_onboarding_cache = {}

    def _get_manual_payment_acquirer(self, env=None):
        if env is None:
            env = self.env
        module_id = env.ref('base.module_payment_transfer').id
        return env['payment.acquirer'].search([('module_id', '=', module_id),
            ('company_id', '=', env.company.id)], limit=1)

    def _get_default_payment_acquirer_onboarding_value(self, key):
        if not self.env.is_admin():
            raise UserError(_("Only administrators can access this data."))

        if self._data_fetched:
            return self._payment_acquirer_onboarding_cache.get(key, '')

        self._data_fetched = True

        self._payment_acquirer_onboarding_cache['payment_method'] = self.env.company.payment_onboarding_payment_method

        installed_modules = self.env['ir.module.module'].sudo().search([
            ('name', 'in', ('payment_paypal', 'payment_stripe')),
            ('state', '=', 'installed'),
        ]).mapped('name')

        if 'payment_paypal' in installed_modules:
            acquirer = self.env['payment.acquirer'].search(
                [('company_id', '=', self.env.company.id), ('name', '=', 'PayPal')], limit=1
            )
            self._payment_acquirer_onboarding_cache['paypal_email_account'] = acquirer['paypal_email_account'] or self.env.user.email or ''
            self._payment_acquirer_onboarding_cache['paypal_pdt_token'] = acquirer['paypal_pdt_token']

        if 'payment_stripe' in installed_modules:
            acquirer = self.env['payment.acquirer'].search(
                [('company_id', '=', self.env.company.id), ('name', '=', 'Stripe')], limit=1
            )
            self._payment_acquirer_onboarding_cache['stripe_secret_key'] = acquirer['stripe_secret_key']
            self._payment_acquirer_onboarding_cache['stripe_publishable_key'] = acquirer['stripe_publishable_key']

        manual_payment = self._get_manual_payment_acquirer()
        journal = manual_payment.journal_id

        self._payment_acquirer_onboarding_cache['manual_name'] = manual_payment['name']
        self._payment_acquirer_onboarding_cache['manual_post_msg'] = manual_payment['pending_msg']
        self._payment_acquirer_onboarding_cache['journal_name'] = journal.name if journal.name != "Bank" else ""
        self._payment_acquirer_onboarding_cache['acc_number'] = journal.bank_acc_number

        return self._payment_acquirer_onboarding_cache.get(key, '')

    def _install_module(self, module_name):
        module = self.env['ir.module.module'].sudo().search([('name', '=', module_name)])
        if module.state not in ('installed', 'to install', 'to upgrade'):
            module.button_immediate_install()

    def _on_save_payment_acquirer(self):
        self._install_module('account_payment')

    def add_payment_methods(self):
        """ Install required payment acquiers, configure them and mark the
            onboarding step as done."""
        if self.payment_method == 'stripe' and not self.stripe_publishable_key:
            self.env.company.payment_onboarding_payment_method = self.payment_method
            return self._start_stripe_onboarding()

        if self.payment_method == 'paypal':
            self._install_module('payment_paypal')

        if self.payment_method == 'stripe':
            self._install_module('payment_stripe')

        if self.payment_method  in ('paypal', 'stripe', 'manual', 'other'):

            self._on_save_payment_acquirer()

            self.env.company.payment_onboarding_payment_method = self.payment_method

            # create a new env including the freshly installed module(s)

            new_env = api.Environment(self.env.cr, self.env.uid, self.env.context)
            if self.payment_method == 'paypal':
                acquirer = new_env['payment.acquirer'].search(
                    [('name', '=', 'PayPal'), ('company_id', '=', self.env.company.id)], limit=1
                )
                if not acquirer:
                    base_acquirer = self.env.ref('payment.payment_acquirer_paypal')
                    # Use sudo to access payment acquirer record that can be in different company.
                    acquirer = base_acquirer.sudo().copy(
                        default={'company_id': self.env.company.id}
                    )
                    acquirer.company_id = self.env.company.id
                default_journal = new_env['account.journal'].search(
                    [('type', '=', 'bank'), ('company_id', '=', new_env.company.id)], limit=1
                )
                acquirer.write({
                    'paypal_email_account': self.paypal_email_account,
                    'paypal_seller_account': self.paypal_seller_account,
                    'paypal_pdt_token': self.paypal_pdt_token,
                    'state': 'enabled',
                    'journal_id': acquirer.journal_id or default_journal
                })
            if self.payment_method == 'stripe':
                new_env['payment.acquirer'].search(
                    [('name', '=', 'Stripe'), ('company_id', '=', self.env.company.id)], limit=1
                ).write({
                    'stripe_secret_key': self.stripe_secret_key,
                    'stripe_publishable_key': self.stripe_publishable_key,
                    'state': 'enabled',
                })
            if self.payment_method == 'manual':
                manual_acquirer = self._get_manual_payment_acquirer(new_env)
                if not manual_acquirer:
                    raise UserError(_(
                        'No manual payment method could be found for this company. '
                        'Please create one from the Payment Acquirer menu.'
                    ))
                manual_acquirer.name = self.manual_name
                manual_acquirer.pending_msg = self.manual_post_msg
                manual_acquirer.state = 'enabled'

                journal = manual_acquirer.journal_id
                if journal:
                    journal.name = self.journal_name
                    journal.bank_acc_number = self.acc_number

            # delete wizard data immediately to get rid of residual credentials
            self.sudo().unlink()
        # the user clicked `apply` and not cancel so we can assume this step is done.
        self._set_payment_acquirer_onboarding_step_done()
        return {'type': 'ir.actions.act_window_close'}

    def _set_payment_acquirer_onboarding_step_done(self):
        self.env.company.sudo().set_onboarding_step_done('payment_acquirer_onboarding_state')

    def action_onboarding_other_payment_acquirer(self):
        self._set_payment_acquirer_onboarding_step_done()
        action = self.env["ir.actions.actions"]._for_xml_id("payment.action_payment_acquirer")
        return action

    def _start_stripe_onboarding(self):
        """ Start Stripe Connect onboarding. """
        menu_id = self.env.ref('payment.payment_acquirer_menu').id
        return self.env.company._run_payment_onboarding_step(menu_id)

```

## File: wizards\payment_link_wizard.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from lxml import etree
from werkzeug import urls

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError
from odoo.tools import float_compare

from odoo.addons.payment import utils as payment_utils


class PaymentLinkWizard(models.TransientModel):
    _name = "payment.link.wizard"
    _description = "Generate Payment Link"

    @api.model
    def default_get(self, fields):
        res = super(PaymentLinkWizard, self).default_get(fields)
        res_id = self._context.get('active_id')
        res_model = self._context.get('active_model')
        res.update({'res_id': res_id, 'res_model': res_model})
        amount_field = 'amount_residual' if res_model == 'account.move' else 'amount_total'
        if res_id and res_model == 'account.move':
            record = self.env[res_model].browse(res_id)
            res.update({
                'description': record.payment_reference,
                'amount': record[amount_field],
                'currency_id': record.currency_id.id,
                'partner_id': record.partner_id.id,
                'amount_max': record[amount_field],
            })
        return res

    @api.model
    def fields_view_get(self, *args, **kwargs):
        """ Overrides orm fields_view_get

        Using a Many2One field, when a user opens this wizard and tries to select a preferred
        payment acquirer, he will get an AccessError telling that he is not allowed to access
        'payment.acquirer' records. This error is thrown because the Many2One field is filled
        by the name_get() function and users don't have clearance to read 'payment.acquirer' records.

        This override allows replacing the Many2One with a selection field, that is prefilled in the
        backend with the name of available acquirers. Therefore, Users will be able to select their
        preferred acquirer.

        :return: composition of the requested view (including inherited views and extensions)
        :rtype: dict
        """
        res = super().fields_view_get(*args, **kwargs)
        if res['type'] == 'form':
            doc = etree.XML(res['arch'])

            # Replace acquirer_id with payment_acquirer_selection in the view
            acq = doc.xpath("//field[@name='acquirer_id']")[0]
            acq.attrib['name'] = 'payment_acquirer_selection'
            acq.attrib['widget'] = 'selection'
            acq.attrib['string'] = _('Force Payment Acquirer')
            del acq.attrib['options']
            del acq.attrib['placeholder']

            # Replace acquirer_id with payment_acquirer_selection in the fields list
            xarch, xfields = self.env['ir.ui.view'].postprocess_and_fields(doc, model=self._name)

            res['arch'] = xarch
            res['fields'] = xfields
        return res

    res_model = fields.Char('Related Document Model', required=True)
    res_id = fields.Integer('Related Document ID', required=True)
    amount = fields.Monetary(currency_field='currency_id', required=True)
    amount_max = fields.Monetary(currency_field='currency_id')
    currency_id = fields.Many2one('res.currency')
    partner_id = fields.Many2one('res.partner')
    partner_email = fields.Char(related='partner_id.email')
    link = fields.Char(string='Payment Link', compute='_compute_values')
    description = fields.Char('Payment Ref')
    access_token = fields.Char(compute='_compute_values')
    company_id = fields.Many2one('res.company', compute='_compute_company')
    available_acquirer_ids = fields.Many2many(
        comodel_name='payment.acquirer',
        string="Payment Acquirers Available",
        compute='_compute_available_acquirer_ids',
        compute_sudo=True,
    )
    acquirer_id = fields.Many2one(
        comodel_name='payment.acquirer',
        string="Force Payment Acquirer",
        domain="[('id', 'in', available_acquirer_ids)]",
        help="Force the customer to pay via the specified payment acquirer. Leave empty to allow the customer to choose among all acquirers."
    )
    has_multiple_acquirers = fields.Boolean(
        string="Has Multiple Acquirers",
        compute='_compute_has_multiple_acquirers',
    )
    payment_acquirer_selection = fields.Selection(
        string="Payment acquirer selected",
        selection='_selection_payment_acquirer_selection',
        default='all',
        compute='_compute_payment_acquirer_selection',
        inverse='_inverse_payment_acquirer_selection',
        required=True,
    )

    @api.onchange('amount', 'description')
    def _onchange_amount(self):
        if float_compare(self.amount_max, self.amount, precision_rounding=self.currency_id.rounding or 0.01) == -1:
            raise ValidationError(_("Please set an amount smaller than %s.") % (self.amount_max))
        if self.amount <= 0:
            raise ValidationError(_("The value of the payment amount must be positive."))

    @api.depends('amount', 'description', 'partner_id', 'currency_id', 'payment_acquirer_selection')
    def _compute_values(self):
        for payment_link in self:
            payment_link.access_token = payment_utils.generate_access_token(
                payment_link.partner_id.id, payment_link.amount, payment_link.currency_id.id
            )
        # must be called after token generation, obvsly - the link needs an up-to-date token
        self._generate_link()

    @api.depends('res_model', 'res_id')
    def _compute_company(self):
        for link in self:
            record = self.env[link.res_model].browse(link.res_id)
            link.company_id = record.company_id if 'company_id' in record else False

    @api.depends('company_id', 'partner_id', 'currency_id')
    def _compute_available_acquirer_ids(self):
        for link in self:
            link.available_acquirer_ids = link._get_payment_acquirer_available()

    @api.depends('acquirer_id')
    def _compute_payment_acquirer_selection(self):
        for link in self:
            link.payment_acquirer_selection = link.acquirer_id.id if link.acquirer_id else 'all'

    def _inverse_payment_acquirer_selection(self):
        for link in self:
            link.acquirer_id = link.payment_acquirer_selection if link.payment_acquirer_selection != 'all' else False

    def _selection_payment_acquirer_selection(self):
        """ Specify available acquirers in the selection field.
        :return: The selection list of available acquirers.
        :rtype: list[tuple]
        """
        defaults = self.default_get(['res_model', 'res_id'])
        selection = [('all', "All")]
        res_model, res_id = defaults['res_model'], defaults['res_id']
        if res_id and res_model in ['account.move', "sale.order"]:
            # At module install, the selection method is called
            # but the document context isn't specified.
            related_document = self.env[res_model].browse(res_id)
            company_id = related_document.company_id
            partner_id = related_document.partner_id
            currency_id = related_document.currency_id
            if res_model == "sale.order":
                # If the Order contains a recurring product but is not already linked to a
                # subscription, the payment acquirer must support tokenization. The res_id allows
                # the overrides of sale_subscription to check this condition.
                selection.extend(
                    self._get_payment_acquirer_available(
                        company_id.id, partner_id.id, currency_id.id, res_id,
                    ).name_get()
                )
            else:
                selection.extend(
                    self._get_payment_acquirer_available(
                        company_id.id, partner_id.id, currency_id.id,
                    ).name_get()
                )
        return selection

    def _get_payment_acquirer_available(self, company_id=None, partner_id=None, currency_id=None):
        """ Select and return the acquirers matching the criteria.

        :param int company_id: The company to which acquirers must belong, as a `res.company` id
        :param int partner_id: The partner making the payment, as a `res.partner` id
        :param int currency_id: The payment currency if known beforehand, as a `res.currency` id
        :return: The compatible acquirers
        :rtype: recordset of `payment.acquirer`
        """
        return self.env['payment.acquirer'].sudo()._get_compatible_acquirers(
            company_id=company_id or self.company_id.id,
            partner_id=partner_id or self.partner_id.id,
            currency_id=currency_id or self.currency_id.id
        )

    @api.depends('available_acquirer_ids')
    def _compute_has_multiple_acquirers(self):
        for link in self:
            link.has_multiple_acquirers = len(link.available_acquirer_ids) > 1

    def _generate_link(self):
        for payment_link in self:
            related_document = self.env[payment_link.res_model].browse(payment_link.res_id)
            base_url = related_document.get_base_url()  # Don't generate links for the wrong website
            payment_link.link = f'{base_url}/payment/pay' \
                   f'?reference={urls.url_quote(payment_link.description)}' \
                   f'&amount={payment_link.amount}' \
                   f'&currency_id={payment_link.currency_id.id}' \
                   f'&partner_id={payment_link.partner_id.id}' \
                   f'&company_id={payment_link.company_id.id}' \
                   f'&invoice_id={payment_link.res_id}' \
                   f'{"&acquirer_id=" + str(payment_link.payment_acquirer_selection) if payment_link.payment_acquirer_selection != "all" else "" }' \
                   f'&access_token={payment_link.access_token}'

```

## File: wizards\payment_link_wizard_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_link_wizard_view_form" model="ir.ui.view">
        <field name="name">payment.link.wizard.form</field>
        <field name="model">payment.link.wizard</field>
        <field name="arch" type="xml">
            <form string="Generate Payment Link">
                <group>
                    <group>
                        <field name="res_id" invisible="1"/>
                        <field name="res_model" invisible="1"/>
                        <field name="partner_id" invisible="1"/>
                        <field name="partner_email" invisible="1"/>
                        <field name="amount_max" invisible="1"/>
                        <field name="available_acquirer_ids" invisible="1"/>
                        <field name="has_multiple_acquirers" invisible="1"/>
                        <field name="description"/>
                        <field name="amount"/>
                        <field name="acquirer_id"
                            placeholder="Leave empty to allow all acquirers"
                            options="{'no_open': True, 'no_create': True}"
                            attrs="{'invisible':[('has_multiple_acquirers', '=', False)]}"/>
                        <field name="currency_id" invisible="1"/>
                        <field name="access_token" invisible="1"/>
                    </group>
                </group>
                <group>
                    <field name="link" readonly="1" widget="CopyClipboardChar"/>
                </group>
                <group attrs="{'invisible':[('partner_email', '!=', False)]}">
                    <p class="alert alert-warning font-weight-bold" role="alert">This partner has no email, which may cause issues with some payment acquirers. Setting an email for this partner is advised.</p>
                </group>
                <footer>
                    <button string="Close" class="btn-primary" special="cancel" data-hotkey="z"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="action_invoice_order_generate_link" model="ir.actions.act_window">
        <field name="name">Generate a Payment Link</field>
        <field name="res_model">payment.link.wizard</field>
        <field name="view_mode">form</field>
        <field name="view_id" ref="payment_link_wizard_view_form"/>
        <field name="target">new</field>
        <field name="binding_model_id" ref="model_account_move"/>
        <field name="binding_view_types">form</field>
    </record>

</odoo>

```

## File: wizards\payment_refund_wizard.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError


class PaymentRefundWizard(models.TransientModel):
    _name = 'payment.refund.wizard'
    _description = "Payment Refund Wizard"

    payment_id = fields.Many2one(
        string="Payment",
        comodel_name='account.payment',
        readonly=True,
        default=lambda self: self.env.context.get('active_id'),
    )
    transaction_id = fields.Many2one(
        string="Payment Transaction", related='payment_id.payment_transaction_id'
    )
    payment_amount = fields.Monetary(string="Payment Amount", related='payment_id.amount')
    refunded_amount = fields.Monetary(string="Refunded Amount", compute='_compute_refunded_amount')
    amount_available_for_refund = fields.Monetary(
        string="Maximum Refund Allowed", related='payment_id.amount_available_for_refund'
    )
    amount_to_refund = fields.Monetary(
        string="Refund Amount", compute='_compute_amount_to_refund', store=True, readonly=False
    )
    currency_id = fields.Many2one(string="Currency", related='transaction_id.currency_id')
    support_refund = fields.Selection(related='transaction_id.acquirer_id.support_refund')
    has_pending_refund = fields.Boolean(
        string="Has a pending refund", compute='_compute_has_pending_refund'
    )

    @api.constrains('amount_to_refund')
    def _check_amount_to_refund_within_boundaries(self):
        for wizard in self:
            if not 0 < wizard.amount_to_refund <= wizard.amount_available_for_refund:
                raise ValidationError(_(
                    "The amount to be refunded must be positive and cannot be superior to %s.",
                    wizard.amount_available_for_refund
                ))

    @api.depends('amount_available_for_refund')
    def _compute_refunded_amount(self):
        for wizard in self:
            wizard.refunded_amount = wizard.payment_amount - wizard.amount_available_for_refund

    @api.depends('amount_available_for_refund')
    def _compute_amount_to_refund(self):
        """ Set the default amount to refund to the amount available for refund. """
        for wizard in self:
            wizard.amount_to_refund = wizard.amount_available_for_refund

    @api.depends('payment_id')  # To always trigger the compute
    def _compute_has_pending_refund(self):
        for wizard in self:
            pending_refunds_count = self.env['payment.transaction'].search_count([
                ('source_transaction_id', '=', wizard.payment_id.payment_transaction_id.id),
                ('operation', '=', 'refund'),
                ('state', 'in', ['draft', 'pending', 'authorized']),
            ])
            wizard.has_pending_refund = pending_refunds_count > 0

    def action_refund(self):
        for wizard in self:
            wizard.transaction_id.action_refund(amount_to_refund=wizard.amount_to_refund)

```

## File: wizards\payment_refund_wizard_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_refund_wizard_view_form" model="ir.ui.view">
        <field name="name">payment.refund.wizard.form</field>
        <field name="model">payment.refund.wizard</field>
        <field name="arch" type="xml">
            <form string="Refund">
                <field name="has_pending_refund" invisible="1"/>
                <div class="alert alert-warning"
                     id="alert_draft_refund_tx"
                     role="alert"
                     attrs="{'invisible': [('has_pending_refund', '=', False)]}">
                    <p>
                        <strong>Warning!</strong> There is a refund pending for this payment.
                        Wait a moment for it to be processed. If the refund is still pending in a
                        few minutes, please check your payment acquirer configuration.
                    </p>
                </div>
                <group>
                    <group>
                        <field name="payment_id" invisible="1"/>
                        <field name="transaction_id" invisible="1"/>
                        <field name="currency_id" invisible="1"/>
                        <field name="support_refund" invisible="1"/>
                        <field name="payment_amount"/>
                        <field name="refunded_amount"
                               attrs="{'invisible': [('refunded_amount', '&lt;=', 0)]}"/>
                        <field name="amount_available_for_refund"/>
                        <field name="amount_to_refund"
                               attrs="{'readonly': [('support_refund', '=', 'full_only')]}"/>
                    </group>
                </group>
                <footer>
                    <button string="Refund" type="object" name="action_refund" class="btn-primary"/>
                    <button string="Close" special="cancel" class="btn-secondary"/>
                </footer>
            </form>
        </field>
    </record>

</odoo>

```

## File: wizards\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_payment_register
from . import payment_acquirer_onboarding_wizard
from . import payment_link_wizard
from . import payment_refund_wizard

```

