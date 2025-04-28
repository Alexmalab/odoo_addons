# Odoo Module: payment

Category: Hidden

This file contains the source code of the Odoo module.

## File: const.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.tools.translate import LazyTranslate

_lt = LazyTranslate(__name__, default_lang='en_US')


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

REPORT_REASONS_MAPPING = {
    'exceed_max_amount': _lt("maximum amount exceeded"),
    'express_checkout_not_supported': _lt("express checkout not supported"),
    'incompatible_country': _lt("incompatible country"),
    'incompatible_currency': _lt("incompatible currency"),
    'incompatible_website': _lt("incompatible website"),
    'manual_capture_not_supported': _lt("manual capture not supported"),
    'provider_not_available': _lt("no supported provider available"),
    'tokenization_not_supported': _lt("tokenization not supported"),
    'validation_not_supported': _lt("tokenization without payment no supported"),
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
    return access_token and consteq(access_token, authentic_token)


# Availability report.

def add_to_report(report, records, available=True, reason=''):
    """ Add records to the report with the provided values.

        Structure of the report:
        report = {
            'providers': {
                provider_record : {
                    'available': true|false,
                    'reason': "",
                },
            },
            'payment_methods': {
                pm_record : {
                    'available': true|false,
                    'reason': "",
                    'supported_providers': [(provider_record, report['providers'][p]['available'])],
                },
            },
        }

    :param dict report: The availability report for providers and payment methods.
    :param payment.provider|payment.method records: The records to add to the report.
    :param bool available: Whether the records are available.
    :param str reason: The reason for which records are not available, if any.
    :return: None
    """
    if report is None or not records:  # The report might not be initialized, or no records to add.
        return

    category = 'providers' if records._name == 'payment.provider' else 'payment_methods'
    report.setdefault(category, {})
    for r in records:
        report[category][r] = {
            'available': available,
            'reason': reason,
        }
        if category == 'payment_methods' and 'providers' in report:
            report[category][r]['supported_providers'] = [
                (p, report['providers'][p]['available'])
                for p in r.provider_ids if p in report['providers']
            ]


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
    if arbitrary_decimal_number is None:
        currency.ensure_one()
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
    if arbitrary_decimal_number is None:
        currency.ensure_one()
        decimal_number = CURRENCY_MINOR_UNITS.get(currency.name, currency.decimal_places)
    else:
        decimal_number = arbitrary_decimal_number
    return int(
        float_round(major_amount * (10**decimal_number), precision_digits=0, rounding_method='DOWN')
    )


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
    recordset.check_access('write')


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


def setup_provider(env, code):
    env['payment.provider']._setup_provider(code)


def reset_payment_provider(env, code, **kwargs):
    env['payment.provider']._remove_provider(code, **kwargs)

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': "Payment Engine",
    'version': '2.0',
    'category': 'Hidden',
    'summary': "The payment engine used by payment provider modules.",
    'depends': ['onboarding', 'portal'],
    'data': [
        # Record data.
        'data/ir_actions_server_data.xml',
        'data/onboarding_data.xml',
        'data/payment_method_data.xml',
        'data/payment_provider_data.xml',
        'data/payment_cron.xml',

        # QWeb templates.
        'views/express_checkout_templates.xml',
        'views/payment_form_templates.xml',
        'views/portal_templates.xml',

        # Model views.
        'views/payment_provider_views.xml',
        'views/payment_method_views.xml',  # Depends on `action_payment_provider`.
        'views/payment_transaction_views.xml',
        'views/payment_token_views.xml',  # Depends on `action_payment_transaction_linked_to_token`.
        'views/res_partner_views.xml',

        # Security.
        'security/ir.model.access.csv',
        'security/payment_security.xml',

        # Wizard views.
        'wizards/payment_capture_wizard_views.xml',
        'wizards/payment_link_wizard_views.xml',
        'wizards/payment_onboarding_views.xml',
    ],
    'assets': {
        'web.assets_frontend': [
            'payment/static/lib/jquery.payment/jquery.payment.js',
            'payment/static/src/**/*',
            ('remove', 'payment/static/src/js/payment_wizard_copy_clipboard_field.js'),
        ],
        'web.assets_backend': [
            'payment/static/src/scss/payment_provider.scss',
            'payment/static/src/js/payment_wizard_copy_clipboard_field.js',
        ],
        'web.qunit_suite_tests': [
            'payment/static/tests/payment_wizard_copy_clipboard_field_tests.js',
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
from odoo.exceptions import AccessError, ValidationError
from odoo.http import request

from odoo.addons.payment import utils as payment_utils
from odoo.addons.payment.controllers.post_processing import PaymentPostProcessing
from odoo.addons.portal.controllers import portal


class PaymentPortal(portal.CustomerPortal):

    """ This controller contains the foundations for online payments through the portal.

    It allows to complete a full payment flow without the need of going through a document-based
    flow made available by another module's controller.

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
        access_token=None, **kwargs
    ):
        """ Display the payment form with optional filtering of payment options.

        The filtering takes place on the basis of provided parameters, if any. If a parameter is
        incorrect or malformed, it is skipped to avoid preventing the user from making the payment.

        In addition to the desired filtering, a second one ensures that none of the following
        rules is broken:

        - Public users are not allowed to save their payment method as a token.
        - Payments made by public users should either *not* be made on behalf of a specific partner
          or have an access token validating the partner, amount and currency.

        We let access rights and security rules do their job for logged users.

        :param str reference: The custom prefix to compute the full reference.
        :param str amount: The amount to pay.
        :param str currency_id: The desired currency, as a `res.currency` id.
        :param str partner_id: The partner making the payment, as a `res.partner` id.
        :param str company_id: The related company, as a `res.company` id.
        :param str access_token: The access token used to authenticate the partner.
        :param dict kwargs: Optional data passed to helper methods.
        :return: The rendered payment form.
        :rtype: str
        :raise werkzeug.exceptions.NotFound: If the access token is invalid.
        """
        # Cast numeric parameters as int or float and void them if their str value is malformed
        currency_id, partner_id, company_id = tuple(map(
            self._cast_as_int, (currency_id, partner_id, company_id)
        ))
        amount = self._cast_as_float(amount)

        # Raise an HTTP 404 if a partner is provided with an invalid access token
        if partner_id:
            if not payment_utils.check_access_token(access_token, partner_id, amount, currency_id):
                raise werkzeug.exceptions.NotFound()  # Don't leak information about ids.

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
            raise werkzeug.exceptions.NotFound()  # The currency must exist and be active.

        availability_report = {}
        # Select all the payment methods and tokens that match the payment context.
        providers_sudo = request.env['payment.provider'].sudo()._get_compatible_providers(
            company_id,
            partner_sudo.id,
            amount,
            currency_id=currency.id,
            report=availability_report,
            **kwargs,
        )  # In sudo mode to read the fields of providers and partner (if logged out).
        payment_methods_sudo = request.env['payment.method'].sudo()._get_compatible_payment_methods(
            providers_sudo.ids,
            partner_sudo.id,
            currency_id=currency.id,
            report=availability_report,
            **kwargs,
        )  # In sudo mode to read the fields of providers.
        tokens_sudo = request.env['payment.token'].sudo()._get_available_tokens(
            providers_sudo.ids, partner_sudo.id
        )  # In sudo mode to be able to read tokens of other partners and the fields of providers.

        # Make sure that the partner's company matches the company passed as parameter.
        company_mismatch = not PaymentPortal._can_partner_pay_in_company(partner_sudo, company)

        # Generate a new access token in case the partner id or the currency id was updated
        access_token = payment_utils.generate_access_token(partner_sudo.id, amount, currency.id)

        portal_page_values = {
            'res_company': company,  # Display the correct logo in a multi-company environment.
            'company_mismatch': company_mismatch,
            'expected_company': company,
            'partner_is_different': partner_is_different,
        }
        payment_form_values = {
            'show_tokenize_input_mapping': self._compute_show_tokenize_input_mapping(
                providers_sudo, **kwargs
            ),
        }
        payment_context = {
            'reference_prefix': reference,
            'amount': amount,
            'currency': currency,
            'partner_id': partner_sudo.id,
            'providers_sudo': providers_sudo,
            'payment_methods_sudo': payment_methods_sudo,
            'tokens_sudo': tokens_sudo,
            'availability_report': availability_report,
            'transaction_route': '/payment/transaction',
            'landing_route': '/payment/confirmation',
            'access_token': access_token,
        }
        rendering_context = {
            **portal_page_values,
            **payment_form_values,
            **payment_context,
            **self._get_extra_payment_form_values(
                **payment_context, currency_id=currency.id, **kwargs
            ),  # Pass the payment context to allow overriding modules to check document access.
        }
        return request.render(self._get_payment_page_template_xmlid(**kwargs), rendering_context)

    @staticmethod
    def _compute_show_tokenize_input_mapping(providers_sudo, **kwargs):
        """ Determine for each provider whether the tokenization input should be shown or not.

        :param recordset providers_sudo: The providers for which to determine whether the
                                         tokenization input should be shown or not, as a sudoed
                                         `payment.provider` recordset.
        :param dict kwargs: The optional data passed to the helper methods.
        :return: The mapping of the computed value for each provider id.
        :rtype: dict
        """
        show_tokenize_input_mapping = {}
        for provider_sudo in providers_sudo:
            show_tokenize_input = provider_sudo.allow_tokenization \
                                  and not provider_sudo._is_tokenization_required(**kwargs)
            show_tokenize_input_mapping[provider_sudo.id] = show_tokenize_input
        return show_tokenize_input_mapping

    def _get_payment_page_template_xmlid(self, **kwargs):
        return 'payment.pay'

    @http.route('/my/payment_method', type='http', methods=['GET'], auth='user', website=True)
    def payment_method(self, **kwargs):
        """ Display the form to manage payment methods.

        :param dict kwargs: Optional data. This parameter is not used here
        :return: The rendered manage form
        :rtype: str
        """
        partner_sudo = request.env.user.partner_id  # env.user is always sudoed

        availability_report = {}
        # Select all the payment methods and tokens that match the payment context.
        providers_sudo = request.env['payment.provider'].sudo()._get_compatible_providers(
            request.env.company.id,
            partner_sudo.id,
            0.,  # There is no amount to pay with validation transactions.
            force_tokenization=True,
            is_validation=True,
            report=availability_report,
            **kwargs,
        )  # In sudo mode to read the fields of providers and partner (if logged out).
        payment_methods_sudo = request.env['payment.method'].sudo()._get_compatible_payment_methods(
            providers_sudo.ids,
            partner_sudo.id,
            force_tokenization=True,
            report=availability_report,
        )  # In sudo mode to read the fields of providers.
        tokens_sudo = request.env['payment.token'].sudo()._get_available_tokens(
            None, partner_sudo.id, is_validation=True
        )  # In sudo mode to read the commercial partner's and providers' fields.

        access_token = payment_utils.generate_access_token(partner_sudo.id, None, None)

        payment_form_values = {
            'mode': 'validation',
            'allow_token_selection': False,
            'allow_token_deletion': True,
        }
        payment_context = {
            'reference_prefix': payment_utils.singularize_reference_prefix(prefix='V'),
            'partner_id': partner_sudo.id,
            'providers_sudo': providers_sudo,
            'payment_methods_sudo': payment_methods_sudo,
            'tokens_sudo': tokens_sudo,
            'availability_report': availability_report,
            'transaction_route': '/payment/transaction',
            'landing_route': '/my/payment_method',
            'access_token': access_token,
        }
        rendering_context = {
            **payment_form_values,
            **payment_context,
            **self._get_extra_payment_form_values(**kwargs),
        }
        return request.render('payment.payment_methods', rendering_context)

    def _get_extra_payment_form_values(self, **kwargs):
        """ Return a dict of extra payment form values to include in the rendering context.

        :param dict kwargs: Optional data. This parameter is not used here.
        :return: The dict of extra payment form values.
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

        self._validate_transaction_kwargs(kwargs, additional_allowed_keys=('reference_prefix',))
        tx_sudo = self._create_transaction(
            amount=amount, currency_id=currency_id, partner_id=partner_id, **kwargs
        )
        self._update_landing_route(tx_sudo, access_token)  # Add the required params to the route.
        return tx_sudo._get_processing_values()

    def _create_transaction(
        self, provider_id, payment_method_id, token_id, amount, currency_id, partner_id, flow,
        tokenization_requested, landing_route, reference_prefix=None, is_validation=False,
        custom_create_values=None, **kwargs
    ):
        """ Create a draft transaction based on the payment context and return it.

        :param int provider_id: The provider of the provider payment method or token, as a
                                `payment.provider` id.
        :param int|None payment_method_id: The payment method, if any, as a `payment.method` id.
        :param int|None token_id: The token, if any, as a `payment.token` id.
        :param float|None amount: The amount to pay, or `None` if in a validation operation.
        :param int|None currency_id: The currency of the amount, as a `res.currency` id, or `None`
                                     if in a validation operation.
        :param int partner_id: The partner making the payment, as a `res.partner` id.
        :param str flow: The online payment flow of the transaction: 'redirect', 'direct' or 'token'.
        :param bool tokenization_requested: Whether the user requested that a token is created.
        :param str landing_route: The route the user is redirected to after the transaction.
        :param str reference_prefix: The custom prefix to compute the full reference.
        :param bool is_validation: Whether the operation is a validation.
        :param dict custom_create_values: Additional create values overwriting the default ones.
        :param dict kwargs: Locally unused data passed to `_is_tokenization_required` and
                            `_compute_reference`.
        :return: The sudoed transaction that was created.
        :rtype: payment.transaction
        :raise UserError: If the flow is invalid.
        """
        # Prepare create values
        if flow in ['redirect', 'direct']:  # Direct payment or payment with redirection
            provider_sudo = request.env['payment.provider'].sudo().browse(provider_id)
            payment_method_sudo = request.env['payment.method'].sudo().browse(payment_method_id)
            token_id = None
            tokenize = bool(
                # Don't tokenize if the user tried to force it through the browser's developer tools
                provider_sudo.allow_tokenization
                and payment_method_sudo.support_tokenization
                # Token is only created if required by the flow or requested by the user
                and (provider_sudo._is_tokenization_required(**kwargs) or tokenization_requested)
            )
        elif flow == 'token':  # Payment by token
            token_sudo = request.env['payment.token'].sudo().browse(token_id)

            # Prevent from paying with a token that doesn't belong to the current partner (either
            # the current user's partner if logged in, or the partner on behalf of whom the payment
            # is being made).
            partner_sudo = request.env['res.partner'].sudo().browse(partner_id)
            if partner_sudo.commercial_partner_id != token_sudo.partner_id.commercial_partner_id:
                raise AccessError(_("You do not have access to this payment token."))

            provider_sudo = token_sudo.provider_id
            payment_method_id = token_sudo.payment_method_id.id
            tokenize = False
        else:
            raise ValidationError(
                _("The payment should either be direct, with redirection, or made by a token.")
            )

        reference = request.env['payment.transaction']._compute_reference(
            provider_sudo.code,
            prefix=reference_prefix,
            **(custom_create_values or {}),
            **kwargs
        )
        if is_validation:  # Providers determine the amount and currency in validation operations
            amount = provider_sudo._get_validation_amount()
            payment_method = request.env['payment.method'].browse(payment_method_id)
            currency_id = provider_sudo.with_context(
                validation_pm=payment_method  # Will be converted to a kwarg in master.
            )._get_validation_currency().id

        # Create the transaction
        tx_sudo = request.env['payment.transaction'].sudo().create({
            'provider_id': provider_sudo.id,
            'payment_method_id': payment_method_id,
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

        # Monitor the transaction to make it available in the portal.
        PaymentPostProcessing.monitor_transaction(tx_sudo)

        return tx_sudo

    @staticmethod
    def _update_landing_route(tx_sudo, access_token):
        """ Add the mandatory parameters to the route and recompute the access token if needed.

        The generic landing route requires the tx id and access token to be provided since there is
        no document to rely on. The access token is recomputed in case we are dealing with a
        validation transaction (provider-specific amount and currency).

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
        """ Display the payment confirmation page to the user.

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
                raise werkzeug.exceptions.NotFound()  # Don't leak information about ids.

            # Display the payment confirmation page to the user
            return request.render('payment.confirm', qcontext={'tx': tx_sudo})
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

    @staticmethod
    def _validate_transaction_kwargs(kwargs, additional_allowed_keys=()):
        """ Verify that the keys of a transaction route's kwargs are all whitelisted.

        The whitelist consists of all the keys that are expected to be passed to a transaction
        route, plus optional contextually allowed keys.

        This method must be called in all transaction routes to ensure that no undesired kwarg can
        be passed as param and then injected in the create values of the transaction.

        :param dict kwargs: The transaction route's kwargs to verify.
        :param tuple additional_allowed_keys: The keys of kwargs that are contextually allowed.
        :return: None
        :raise ValidationError: If some kwargs keys are rejected.
        """
        whitelist = {
            'provider_id',
            'payment_method_id',
            'token_id',
            'amount',
            'flow',
            'tokenization_requested',
            'landing_route',
            'is_validation',
            'csrf_token',
        }
        whitelist.update(additional_allowed_keys)
        rejected_keys = set(kwargs.keys()) - whitelist
        if rejected_keys:
            raise ValidationError(
                _("The following kwargs are not whitelisted: %s", ', '.join(rejected_keys))
            )

```

## File: controllers\post_processing.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

import psycopg2

from odoo import http
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

    MONITORED_TX_ID_KEY = '__payment_monitored_tx_id__'

    @http.route('/payment/status', type='http', auth='public', website=True, sitemap=False)
    def display_status(self, **kwargs):
        """ Fetch the transaction and display it on the payment status page.

        :param dict kwargs: Optional data. This parameter is not used here
        :return: The rendered status page
        :rtype: str
        """
        monitored_tx = self._get_monitored_transaction()
        # The session might have expired, or the transaction never existed.
        values = {'tx': monitored_tx} if monitored_tx else {'payment_not_found': True}
        return request.render('payment.payment_status', values)

    @http.route('/payment/status/poll', type='json', auth='public')
    def poll_status(self, **_kwargs):
        """ Fetch the transaction and trigger its post-processing.

        :return: The post-processing values of the transaction.
        :rtype: dict
        """
        # We only poll the payment status if a payment was found, so the transaction should exist.
        monitored_tx = self._get_monitored_transaction()

        # Post-process the transaction before redirecting the user to the landing route and its
        # document.
        if not monitored_tx.is_post_processed:
            try:
                monitored_tx._post_process()
            except (
                psycopg2.OperationalError, psycopg2.IntegrityError
            ):  # The database cursor could not be committed.
                request.env.cr.rollback()  # Rollback and try later.
                raise Exception('retry')
            except Exception as e:
                request.env.cr.rollback()
                _logger.exception(
                    "Encountered an error while post-processing transaction with id %s:\n%s",
                    monitored_tx.id, e
                )
                raise

        return {
            'provider_code': monitored_tx.provider_code,
            'state': monitored_tx.state,
            'landing_route': monitored_tx.landing_route,
        }

    @classmethod
    def monitor_transaction(cls, transaction):
        """ Make the provided transaction id monitored.

        :param payment.transaction transaction: The transaction to monitor.
        :return: None
        """
        request.session[cls.MONITORED_TX_ID_KEY] = transaction.id

    def _get_monitored_transaction(self):
        """ Retrieve the user's last transaction from the session (the transaction being monitored).

        :return: the user's last transaction
        :rtype: payment.transaction
        """
        return request.env['payment.transaction'].sudo().browse(
            request.session.get(self.MONITORED_TX_ID_KEY)
        ).exists()

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import portal

```

## File: data\ir_actions_server_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <record id="action_activate_stripe" model="ir.actions.server">
        <field name="name">Activate Stripe</field>
        <field name="model_id" ref="payment.model_payment_provider"/>
        <field name="state">code</field>
        <field name="code">
action = env.company._run_payment_onboarding_step()
        </field>
    </record>

</odoo>

```

## File: data\neutralize.sql

```sql
-- disable generic payment provider
UPDATE payment_provider
   SET state = 'disabled'
 WHERE state NOT IN ('test', 'disabled');

```

## File: data\onboarding_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <!-- ONBOARDING STEPS (WITHOUT PANEL) -->
    <record id="onboarding_onboarding_step_payment_provider" model="onboarding.onboarding.step">
        <field name="title">Online Payments</field>
        <field name="sequence">99</field>
    </record>

</odoo>

```

## File: data\payment_cron.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record model="ir.cron" id="cron_post_process_payment_tx">
        <field name="name">Payment: Post-process transactions</field>
        <field name="model_id" ref="payment.model_payment_transaction" />
        <field name="state">code</field>
        <field name="code">model._cron_post_process()</field>
        <field name="user_id" ref="base.user_root" />
        <field name="interval_number">10</field>
        <field name="interval_type">minutes</field>
        <field name="active" eval="False"/>
    </record>

    <function model="payment.provider" name="_toggle_post_processing_cron"/>

</odoo>

```

## File: data\payment_method_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <!-- === PRIMARY PAYMENT METHODS === -->

    <record id="payment_method_7eleven" model="payment.method">
        <field name="name">7Eleven</field>
        <field name="code">7eleven</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/7eleven.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.ph'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.PHP'),
                     ])]"
        />
    </record>

    <record id="payment_method_ach_direct_debit" model="payment.method">
        <field name="name">ACH Direct Debit</field>
        <field name="code">ach_direct_debit</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/ach_direct_debit.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.pr'),
                         ref('base.us'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.USD'),
                     ])]"
        />
    </record>

    <record id="payment_method_affirm" model="payment.method">
        <field name="name">Affirm</field>
        <field name="code">affirm</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/affirm.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.us'),
                         ref('base.ca'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.CAD'),
                         ref('base.USD'),
                     ])]"
        />
    </record>

    <record id="payment_method_afterpay" model="payment.method">
        <field name="name">Afterpay</field>
        <field name="code">afterpay</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/afterpay.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.au'),
                         ref('base.nz'),
                         ref('base.us'),
                         ref('base.ca'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.CAD'),
                         ref('base.USD'),
                         ref('base.AUD'),
                         ref('base.NZD'),
                     ])]"
        />
    </record>

    <record id="payment_method_afterpay_riverty" model="payment.method">
        <field name="name">AfterPay</field>
        <field name="code">afterpay_riverty</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/afterpay_riverty.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.nl'),
                         ref('base.be'),
                         ref('base.de'),
                         ref('base.at'),
                         ref('base.fi'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.EUR'),
                     ])]"
        />
    </record>

    <record id="payment_method_akulaku" model="payment.method">
        <field name="name">Akulaku PayLater</field>
        <field name="code">akulaku</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/akulaku.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.id'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.IDR'),
                     ])]"
        />
    </record>

    <record id="payment_method_alipay" model="payment.method">
        <field name="name">Alipay</field>
        <field name="code">alipay</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/alipay.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
    </record>

    <record id="payment_method_alipay_hk" model="payment.method">
        <field name="name">AliPayHK</field>
        <field name="code">alipay_hk</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/alipay_hk.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.hk'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.HKD'),
                     ])]"
        />
    </record>

    <record id="payment_method_alipay_plus" model="payment.method">
        <field name="name">Alipay+</field>
        <field name="code">alipay_plus</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/alipay_plus.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.at'),
                         ref('base.au'),
                         ref('base.be'),
                         ref('base.bg'),
                         ref('base.ch'),
                         ref('base.cr'),
                         ref('base.cy'),
                         ref('base.de'),
                         ref('base.dk'),
                         ref('base.ee'),
                         ref('base.es'),
                         ref('base.fi'),
                         ref('base.fr'),
                         ref('base.gr'),
                         ref('base.hk'),
                         ref('base.hr'),
                         ref('base.hu'),
                         ref('base.ie'),
                         ref('base.is'),
                         ref('base.it'),
                         ref('base.kr'),
                         ref('base.li'),
                         ref('base.lt'),
                         ref('base.lu'),
                         ref('base.lv'),
                         ref('base.mt'),
                         ref('base.my'),
                         ref('base.nl'),
                         ref('base.no'),
                         ref('base.ph'),
                         ref('base.pl'),
                         ref('base.pt'),
                         ref('base.ro'),
                         ref('base.se'),
                         ref('base.si'),
                         ref('base.sk'),
                         ref('base.th'),
                         ref('base.uk'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.AUD'),
                         ref('base.CHF'),
                         ref('base.DKK'),
                         ref('base.EUR'),
                         ref('base.GBP'),
                         ref('base.HKD'),
                         ref('base.KRW'),
                         ref('base.MYR'),
                         ref('base.NOK'),
                         ref('base.PHP'),
                         ref('base.SEK'),
                         ref('base.THB'),
                     ])]"
        />
    </record>

    <record id="payment_method_alma" model="payment.method">
        <field name="name">Alma</field>
        <field name="code">alma</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/alma.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.fr'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.EUR'),
                     ])]"
        />
    </record>

    <record id="payment_method_amazon_pay" model="payment.method">
        <field name="name">Amazon Pay</field>
        <field name="code">amazon_pay</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/amazon_pay.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.be'),
                         ref('base.cy'),
                         ref('base.dk'),
                         ref('base.fr'),
                         ref('base.hu'),
                         ref('base.ie'),
                         ref('base.it'),
                         ref('base.lu'),
                         ref('base.pt'),
                         ref('base.es'),
                         ref('base.uk'),
                         ref('base.us'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.EUR'),
                         ref('base.AUD'),
                         ref('base.GBP'),
                         ref('base.DKK'),
                         ref('base.HKD'),
                         ref('base.JPY'),
                         ref('base.NZD'),
                         ref('base.NOK'),
                         ref('base.ZAR'),
                         ref('base.USD'),
                     ])]"
        />
    </record>

    <record id="payment_method_astropay" model="payment.method">
        <field name="name">Astropay TEF</field>
        <field name="code">astropay</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/bank.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.ec'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.USD'),
                     ])]"
        />
    </record>

    <record id="payment_method_atome" model="payment.method">
        <field name="name">Atome</field>
        <field name="code">atome</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/atome.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.my'),
                         ref('base.sg'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.MYR'),
                         ref('base.SGD'),
                     ])]"
        />
    </record>

    <record id="payment_method_axis" model="payment.method">
        <field name="name">Axis</field>
        <field name="code">axis</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/axis.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.fr'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.EUR'),
                     ])]"
        />
    </record>

    <record id="payment_method_bacs_direct_debit" model="payment.method">
        <field name="name">BACS Direct Debit</field>
        <field name="code">bacs_direct_debit</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/bacs_direct_debit.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.uk'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.GBP'),
                     ])]"
        />
    </record>

    <record id="payment_method_bancnet" model="payment.method">
        <field name="name">BancNet</field>
        <field name="code">bancnet</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/bancnet.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.ph'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.PHP'),
                     ])]"
        />
    </record>

    <record id="payment_method_bancomat_pay" model="payment.method">
        <field name="name">BANCOMAT Pay</field>
        <field name="code">bancomat_pay</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/bancomat_pay.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.it'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.EUR'),
                     ])]"
        />
    </record>

    <record id="payment_method_bancontact" model="payment.method">
        <field name="name">Bancontact</field>
        <field name="code">bancontact</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/bancontact.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.be'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.EUR'),
                     ])]"
        />
    </record>

    <record id="payment_method_bangkok_bank" model="payment.method">
        <field name="name">Bangkok Bank</field>
        <field name="code">bangkok_bank</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/bank.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.th'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.THB'),
                     ])]"
        />
    </record>

    <record id="payment_method_bank_account" model="payment.method">
        <field name="name">Bank Account</field>
        <field name="code">bank_account</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/bank.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
    </record>

    <record id="payment_method_bank_bca" model="payment.method">
        <field name="name">BCA</field>
        <field name="code">bank_bca</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/bank_bca.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.id'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.IDR'),
                     ])]"
        />
    </record>

    <record id="payment_method_bank_of_ayudhya" model="payment.method">
        <field name="name">Bank of Ayudhya</field>
        <field name="code">bank_of_ayudhya</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/bank.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.th'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.THB'),
                     ])]"
        />
    </record>

    <record id="payment_method_bank_permata" model="payment.method">
        <field name="name">Bank Permata</field>
        <field name="code">bank_permata</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/bank_permata.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.id'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.IDR'),
                     ])]"
        />
    </record>

    <record id="payment_method_bank_reference" model="payment.method">
        <field name="name">Bank reference</field>
        <field name="code">bank_reference</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/bank.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
    </record>

    <record id="payment_method_bank_transfer" model="payment.method">
        <field name="name">Bank Transfer</field>
        <field name="code">bank_transfer</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/bank.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.INR'),
                         ref('base.NGN'),
                     ])]"
        />
    </record>

    <record id="payment_method_becs_direct_debit" model="payment.method">
        <field name="name">BECS Direct Debit</field>
        <field name="code">becs_direct_debit</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/becs_direct_debit.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.au'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.AUD'),
                     ])]"
        />
    </record>

    <record id="payment_method_belfius" model="payment.method">
        <field name="name">Belfius</field>
        <field name="code">belfius</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/belfius.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.be'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.EUR'),
                     ])]"
        />
    </record>

    <record id="payment_method_benefit" model="payment.method">
        <field name="name">Benefit</field>
        <field name="code">benefit</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/benefit.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.bh'),
               ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.BHD'),
                     ])]"
        />
    </record>

    <record id="payment_method_bharatqr" model="payment.method">
        <field name="name">BharatQR</field>
        <field name="code">bharatqr</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/bharatqr.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.in'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.INR'),
                     ])]"
        />
    </record>

    <record id="payment_method_billease" model="payment.method">
        <field name="name">BillEase</field>
        <field name="code">billease</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/billease.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
              eval="[Command.set([
                        ref('base.ph'),
                    ])]"
        />
        <field name="supported_currency_ids"
              eval="[Command.set([
                        ref('base.PHP'),
                    ])]"
        />
    </record>

    <record id="payment_method_billink" model="payment.method">
        <field name="name">Billink</field>
        <field name="code">billink</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/billink.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.nl'),
                         ref('base.be'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.EUR'),
                     ])]"
        />
    </record>

    <record id="payment_method_bizum" model="payment.method">
        <field name="name">Bizum</field>
        <field name="code">bizum</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/bizum.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.es'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.EUR'),
                     ])]"
        />
    </record>

    <record id="payment_method_blik" model="payment.method">
        <field name="name">BLIK</field>
        <field name="code">blik</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/blik.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.pl'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.PLN'),
                     ])]"
        />
    </record>

    <record id="payment_method_bni" model="payment.method">
        <field name="name">Bank Negara Indonesia</field>
        <field name="code">bni</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/bni.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.id'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.IDR'),
                     ])]"
        />
    </record>

    <record id="payment_method_boleto" model="payment.method">
        <field name="name">Boleto</field>
        <field name="code">boleto</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/boleto.png"/>
        <field name="support_tokenization">True</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.br'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.BRL'),
                     ])]"
        />
    </record>

    <record id="payment_method_boost" model="payment.method">
        <field name="name">Boost</field>
        <field name="code">boost</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/boost.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.my'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.MYR'),
                     ])]"
        />
    </record>

    <record id="payment_method_bpi" model="payment.method">
       <field name="name">Bank of the Philippine Islands</field>
       <field name="code">bpi</field>
       <field name="sequence">1000</field>
       <field name="active">False</field>
       <field name="image" type="base64" file="payment/static/img/bank.png"/>
       <field name="support_tokenization">False</field>
       <field name="support_express_checkout">False</field>
       <field name="support_refund">none</field>
       <field name="supported_country_ids"
              eval="[Command.set([
                        ref('base.ph'),
                    ])]"
       />
       <field name="supported_currency_ids"
              eval="[Command.set([
                        ref('base.PHP'),
                    ])]"
       />
    </record>

    <record id="payment_method_brankas" model="payment.method">
        <field name="name">BRANKAS</field>
        <field name="code">brankas</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/brankas.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.id'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.IDR'),
                     ])]"
        />
    </record>

    <record id="payment_method_bri" model="payment.method">
        <field name="name">BRI</field>
        <field name="code">bri</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/bri.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.id'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.IDR'),
                     ])]"
        />
    </record>

    <record id="payment_method_bsi" model="payment.method">
        <field name="name">Bank Syariah Indonesia</field>
        <field name="code">bsi</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/bsi.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.id'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.IDR'),
                     ])]"
        />
    </record>

    <record id="payment_method_card" model="payment.method">
        <field name="name">Card</field>
        <field name="code">card</field>
        <field name="sequence">10</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/card.png"/>
        <field name="support_tokenization">True</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
    </record>

    <record id="payment_method_cash_app_pay" model="payment.method">
        <field name="name">Cash App Pay</field>
        <field name="code">cash_app_pay</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/cash_app_pay.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.us'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.USD'),
                     ])]"
        />
    </record>

    <record id="payment_method_cashalo" model="payment.method">
        <field name="name">Cashalo</field>
        <field name="code">cashalo</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/cashalo.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.ph'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.PHP'),
                     ])]"
        />
    </record>

    <record id="payment_method_cebuana" model="payment.method">
        <field name="name">Cebuana</field>
        <field name="code">cebuana</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/cebuana.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.ph'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.PHP'),
                     ])]"
        />
    </record>

    <record id="payment_method_cimb_niaga" model="payment.method">
        <field name="name">CIMB Niaga</field>
        <field name="code">cimb_niaga</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/cimb_niaga.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.id'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.IDR'),
                     ])]"
        />
    </record>

    <record id="payment_method_clearpay" model="payment.method">
        <field name="name">Clearpay</field>
        <field name="code">clearpay</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/clearpay.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.uk'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.GBP'),
                     ])]"
        />
    </record>

    <record id="payment_method_cofidis" model="payment.method">
        <field name="name">cofidis</field>
        <field name="code">cofidis</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/cofidis.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.be'),
                         ref('base.fr'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.EUR'),
                     ])]"
        />
    </record>

    <record id="payment_method_dana" model="payment.method">
        <field name="name">Dana</field>
        <field name="code">dana</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/dana.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.id'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.IDR'),
                     ])]"
        />
    </record>

    <record id="payment_method_dolfin" model="payment.method">
        <field name="name">Dolfin</field>
        <field name="code">dolfin</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/dolfin.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.th'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.THB'),
                     ])]"
        />
    </record>

    <record id="payment_method_duitnow" model="payment.method">
        <field name="name">DuitNow</field>
        <field name="code">duitnow</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/duitnow.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.my'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.MYR'),
                     ])]"
        />
    </record>

    <record id="payment_method_emi_india" model="payment.method">
        <field name="name">EMI</field>
        <field name="code">emi_india</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/card.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.in'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.INR'),
                     ])]"
        />
    </record>

    <record id="payment_method_enets" model="payment.method">
        <field name="name">eNETS</field>
        <field name="code">enets</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/enets.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.sg'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.SGD'),
                     ])]"
        />
    </record>

    <record id="payment_method_eps" model="payment.method">
        <field name="name">EPS</field>
        <field name="code">eps</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/eps.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.at'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.EUR'),
                     ])]"
        />
    </record>

    <record id="payment_method_floa_bank" model="payment.method">
        <field name="name">Floa Bank</field>
        <field name="code">floa_bank</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/floa_bank.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.br'),
                         ref('base.es'),
                         ref('base.fr'),
                         ref('base.it'),
                         ref('base.pt'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.EUR'),
                     ])]"
        />
    </record>

    <record id="payment_method_fps" model="payment.method">
        <field name="name">FPS</field>
        <field name="code">fps</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/card.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.hk'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.HKD'),
                     ])]"
        />
    </record>

    <record id="payment_method_fpx" model="payment.method">
        <field name="name">FPX</field>
        <field name="code">fpx</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/fpx.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.my'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.MYR'),
                     ])]"
        />
    </record>

    <record id="payment_method_frafinance" model="payment.method">
        <field name="name">Frafinance</field>
        <field name="code">frafinance</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/frafinance.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.be'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.EUR'),
                     ])]"
        />
    </record>

    <record id="payment_method_gcash" model="payment.method">
        <field name="name">GCash</field>
        <field name="code">gcash</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/gcash.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.ph'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.PHP'),
                     ])]"
        />
    </record>

    <record id="payment_method_giropay" model="payment.method">
        <field name="name">Giropay</field>
        <field name="code">giropay</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/giropay.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.de'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.EUR'),
                     ])]"
        />
    </record>

    <record id="payment_method_gopay" model="payment.method">
        <field name="name">GoPay</field>
        <field name="code">gopay</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/gopay.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.id'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.IDR'),
                     ])]"
        />
    </record>

    <record id="payment_method_grabpay" model="payment.method">
        <field name="name">GrabPay</field>
        <field name="code">grabpay</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/grabpay.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.my'),
                         ref('base.sg'),
                         ref('base.ph'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.SGD'),
                         ref('base.MYR'),
                         ref('base.PHP'),
                     ])]"
        />
    </record>

    <record id="payment_method_gsb" model="payment.method">
        <field name="name">Government Savings Bank</field>
        <field name="code">gsb</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/bank.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.th'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.THB'),
                     ])]"
        />
    </record>

    <record id="payment_method_hd" model="payment.method">
        <field name="name">HD Bank</field>
        <field name="code">hd</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/bank.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.vn'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.VND'),
                     ])]"
        />
    </record>

    <record id="payment_method_hoolah" model="payment.method">
        <field name="name">Hoolah</field>
        <field name="code">hoolah</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/hoolah.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.sg'),
                         ref('base.my'),
                         ref('base.hk'),
                         ref('base.th'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.SGD'),
                         ref('base.MYR'),
                         ref('base.HKD'),
                         ref('base.THB'),
                     ])]"
        />
    </record>

    <record id="payment_method_humm" model="payment.method">
        <field name="name">Humm</field>
        <field name="code">humm</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/humm.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.au'),
                         ref('base.nz'),
                         ref('base.uk'),
                         ref('base.ie'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.AUD'),
                         ref('base.NZD'),
                         ref('base.GBP'),
                         ref('base.EUR'),
                     ])]"
        />
    </record>

    <record id="payment_method_ideal" model="payment.method">
        <field name="name">iDEAL</field>
        <field name="code">ideal</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/ideal.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.nl'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.EUR'),
                     ])]"
        />
    </record>

    <record id="payment_method_in3" model="payment.method">
        <field name="name">in3</field>
        <field name="code">in3</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/in3.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.nl'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.EUR'),
                     ])]"
        />
    </record>

    <record id="payment_method_jeniuspay" model="payment.method">
        <field name="name">JeniusPay</field>
        <field name="code">jeniuspay</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/jeniuspay.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.id'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.IDR'),
                     ])]"
        />
    </record>

    <record id="payment_method_jkopay" model="payment.method">
        <field name="name">Jkopay</field>
        <field name="code">jkopay</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/jkopay.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.cn'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.CNY'),
                     ])]"
        />
    </record>

    <record id="payment_method_kakaopay" model="payment.method">
        <field name="name">KakaoPay</field>
        <field name="code">kakaopay</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/kakaopay.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.kr'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.KRW'),
                     ])]"
        />
    </record>

    <record id="payment_method_kasikorn_bank" model="payment.method">
        <field name="name">Kasikorn Bank</field>
        <field name="code">kasikorn_bank</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/bank.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.th'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.THB'),
                     ])]"
        />
    </record>

    <record id="payment_method_kbc_cbc" model="payment.method">
        <field name="name">KBC/CBC</field>
        <field name="code">kbc_cbc</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/kbc.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.be'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.EUR'),
                     ])]"
        />
    </record>

    <record id="payment_method_klarna" model="payment.method">
        <field name="name">Klarna</field>
        <field name="code">klarna</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/klarna.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.au'),
                         ref('base.at'),
                         ref('base.be'),
                         ref('base.ca'),
                         ref('base.cz'),
                         ref('base.dk'),
                         ref('base.fi'),
                         ref('base.fr'),
                         ref('base.de'),
                         ref('base.gr'),
                         ref('base.ie'),
                         ref('base.it'),
                         ref('base.nl'),
                         ref('base.nz'),
                         ref('base.no'),
                         ref('base.pl'),
                         ref('base.pt'),
                         ref('base.es'),
                         ref('base.se'),
                         ref('base.ch'),
                         ref('base.uk'),
                         ref('base.us'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.AUD'),
                         ref('base.EUR'),
                         ref('base.CAD'),
                         ref('base.CZK'),
                         ref('base.DKK'),
                         ref('base.NZD'),
                         ref('base.NOK'),
                         ref('base.PLN'),
                         ref('base.SEK'),
                         ref('base.CHF'),
                         ref('base.GBP'),
                         ref('base.USD'),
                     ])]"
        />
    </record>

    <record id="payment_method_klarna_paynow" model="payment.method">
        <field name="name">Klarna - Pay Now</field>
        <field name="code">klarna_paynow</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/klarna.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.de'),
                         ref('base.nl'),
                         ref('base.se'),
                         ref('base.ch'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.EUR'),
                         ref('base.SEK'),
                         ref('base.CHF'),
                     ])]"
        />
    </record>

    <record id="payment_method_klarna_pay_over_time" model="payment.method">
        <field name="name">Klarna - Pay over time</field>
        <field name="code">klarna_pay_over_time</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/klarna.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.au'),
                         ref('base.at'),
                         ref('base.ca'),
                         ref('base.cz'),
                         ref('base.fi'),
                         ref('base.fr'),
                         ref('base.de'),
                         ref('base.gr'),
                         ref('base.ie'),
                         ref('base.it'),
                         ref('base.nl'),
                         ref('base.nz'),
                         ref('base.no'),
                         ref('base.pt'),
                         ref('base.es'),
                         ref('base.se'),
                         ref('base.uk'),
                         ref('base.us'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.AUD'),
                         ref('base.EUR'),
                         ref('base.CAD'),
                         ref('base.CZK'),
                         ref('base.NZD'),
                         ref('base.NOK'),
                         ref('base.SEK'),
                         ref('base.GBP'),
                         ref('base.USD'),
                     ])]"
        />
    </record>

    <record id="payment_method_knet" model="payment.method">
        <field name="name">KNET</field>
        <field name="code">knet</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/knet.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.kw'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.KWD'),
                     ])]"
        />
    </record>

    <record id="payment_method_kredivo" model="payment.method">
        <field name="name">Kredivo</field>
        <field name="code">kredivo</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/kredivo.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.id'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.IDR'),
                     ])]"
        />
    </record>

    <record id="payment_method_krungthai_bank" model="payment.method">
        <field name="name">KrungThai Bank</field>
        <field name="code">krungthai_bank</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/bank.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.th'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.THB'),
                     ])]"
        />
    </record>

    <record id="payment_method_linepay" model="payment.method">
        <field name="name">LINE Pay</field>
        <field name="code">linepay</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/linepay.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.jp'),
                         ref('base.tw'),
                         ref('base.th'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.JPY'),
                         ref('base.TWD'),
                         ref('base.THB'),
                     ])]"
        />
    </record>

    <record id="payment_method_linkaja" model="payment.method">
        <field name="name">LinkAja</field>
        <field name="code">linkaja</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/linkaja.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.id'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.IDR'),
                     ])]"
        />
    </record>

    <record id="payment_method_lydia" model="payment.method">
        <field name="name">Lydia</field>
        <field name="code">lydia</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/lydia.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.fr'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.EUR'),
                         ref('base.GBP'),
                     ])]"
        />
    </record>

    <record id="payment_method_lyfpay" model="payment.method">
        <field name="name">LyfPay</field>
        <field name="code">lyfpay</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/lyfpay.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.fr'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.EUR'),
                     ])]"
        />
    </record>

    <record id="payment_method_mada" model="payment.method">
        <field name="name">Mada</field>
        <field name="code">mada</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/mada.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.sa'),
                         ref('base.ae'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.SAR'),
                     ])]"
        />
    </record>

    <record id="payment_method_mandiri" model="payment.method">
        <field name="name">Mandiri</field>
        <field name="code">mandiri</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/mandiri.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.id'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.IDR'),
                     ])]"
        />
    </record>

    <record id="payment_method_maya" model="payment.method">
        <field name="name">Maya</field>
        <field name="code">maya</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/maya.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.ph'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.PHP'),
                     ])]"
        />
    </record>

    <record id="payment_method_maybank" model="payment.method">
        <field name="name">Maybank</field>
        <field name="code">maybank</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/maybank.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.id'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.IDR'),
                     ])]"
        />
    </record>

    <record id="payment_method_mbway" model="payment.method">
        <field name="name">MB WAY</field>
        <field name="code">mbway</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/mbway.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.pt'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.EUR'),
                     ])]"
        />
    </record>

    <record id="payment_method_mobile_money" model="payment.method">
        <field name="name">Mobile money</field>
        <field name="code">mobile_money</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/mtn-mobile-money.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.gh'),
                         ref('base.cm'),
                         ref('base.ci'),
                         ref('base.ml'),
                         ref('base.sn'),
                         ref('base.ug'),
                         ref('base.rw'),
                         ref('base.zm'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.GHS'),
                         ref('base.XAF'),
                         ref('base.XOF'),
                         ref('base.UGX'),
                         ref('base.RWF'),
                         ref('base.ZMW'),
                     ])]"
        />
    </record>

    <record id="payment_method_mobile_pay" model="payment.method">
        <field name="name">MobilePay</field>
        <field name="code">mobile_pay</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/mobile_pay.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.dk'),
                         ref('base.fi'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.EUR'),
                         ref('base.DKK'),
                         ref('base.SEK'),
                         ref('base.NOK'),
                     ])]"
        />
    </record>

    <record id="payment_method_momo" model="payment.method">
        <field name="name">MoMo</field>
        <field name="code">momo</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/momo.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.vn'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.VND'),
                     ])]"
        />
    </record>

    <record id="payment_method_mpesa" model="payment.method">
        <field name="name">M-Pesa</field>
        <field name="code">mpesa</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/mpesa.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.ke'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.KES'),
                     ])]"
        />
    </record>

    <record id="payment_method_multibanco" model="payment.method">
        <field name="name">Multibanco</field>
        <field name="code">multibanco</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/multibanco.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.pt'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.EUR'),
                     ])]"
        />
    </record>

    <record id="payment_method_mybank" model="payment.method">
        <field name="name">MyBank</field>
        <field name="code">mybank</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/mybank.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.it'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.EUR'),
                     ])]"
        />
    </record>

    <record id="payment_method_nuvei_local" model="payment.method">
        <field name="name">Local Payments</field>
        <field name="code">nuvei_local</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/bank.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.uy'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.UYU'),
                     ])]"
        />
    </record>

    <record id="payment_method_napas_card" model="payment.method">
        <field name="name">Napas Card</field>
        <field name="code">napas_card</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/napas_card.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="support_refund">full_only</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.vn'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.VND'),
                     ])]"
        />
    </record>

    <record id="payment_method_naver_pay" model="payment.method">
        <field name="name">Naver Pay</field>
        <field name="code">naver_pay</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/naver_pay.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.kr'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.KRW'),
                     ])]"
        />
    </record>

    <record id="payment_method_netbanking" model="payment.method">
        <field name="name">Netbanking</field>
        <field name="code">netbanking</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/bank.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.in'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.INR'),
                     ])]"
        />
    </record>

    <record id="payment_method_octopus" model="payment.method">
        <field name="name">Octopus</field>
        <field name="code">octopus</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/octopus.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.hk'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.HKD'),
                     ])]"
        />
    </record>

    <record id="payment_method_online_banking_czech_republic" model="payment.method">
        <field name="name">Online Banking Czech Republic</field>
        <field name="code">online_banking_czech_republic</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/bank.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.cz'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.CZK'),
                     ])]"
        />
    </record>

    <record id="payment_method_online_banking_india" model="payment.method">
        <field name="name">Online Banking India</field>
        <field name="code">online_banking_india</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/bank.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="support_refund">full_only</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.in'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.INR'),
                     ])]"
        />
    </record>

    <record id="payment_method_online_banking_slovakia" model="payment.method">
        <field name="name">Online Banking Slovakia</field>
        <field name="code">online_banking_slovakia</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/bank.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.sk'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.EUR'),
                     ])]"
        />
    </record>

    <record id="payment_method_online_banking_thailand" model="payment.method">
        <field name="name">Online Banking Thailand</field>
        <field name="code">online_banking_thailand</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/bank.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.th'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.THB'),
                     ])]"
        />
    </record>

    <record id="payment_method_open_banking" model="payment.method">
        <field name="name">Open banking</field>
        <field name="code">open_banking</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/bank.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.uk'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.GBP'),
                     ])]"
        />
    </record>

    <record id="payment_method_ovo" model="payment.method">
        <field name="name">OVO</field>
        <field name="code">ovo</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/ovo.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.id'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.IDR'),
                     ])]"
        />
    </record>

    <record id="payment_method_oxxopay" model="payment.method">
        <field name="name">Oxxo Pay</field>
        <field name="code">oxxopay</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/oxxopay.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.mx'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.MXN'),
                     ])]"
        />
    </record>

    <record id="payment_method_paybright" model="payment.method">
        <field name="name">PayBright</field>
        <field name="code">paybright</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/paybright.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.ca'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.CAD'),
                     ])]"
        />
    </record>

    <record id="payment_method_pace" model="payment.method">
        <field name="name">Pace.</field>
        <field name="code">pace</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/pace.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.sg'),
                         ref('base.jp'),
                         ref('base.my'),
                         ref('base.hk'),
                         ref('base.tw'),
                         ref('base.th'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.SGD'),
                         ref('base.MYR'),
                         ref('base.JPY'),
                         ref('base.HKD'),
                         ref('base.THB'),
                         ref('base.TWD'),
                     ])]"
        />
    </record>

    <record id="payment_method_paylater_india" model="payment.method">
        <field name="name">Pay Later</field>
        <field name="code">paylater_india</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/pay_later.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="support_refund">full_only</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.in'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.INR'),
                     ])]"
        />
    </record>

    <record id="payment_method_pay_easy" model="payment.method">
        <field name="name">Pay-easy</field>
        <field name="code">pay_easy</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/pay_easy.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.jp'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.JPY'),
                     ])]"
        />
    </record>

    <record id="payment_method_pay_id" model="payment.method">
        <field name="name">PayID</field>
        <field name="code">pay_id</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/pay_id.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.au'),
                         ref('base.nz'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.AUD'),
                         ref('base.NZD'),
                     ])]"
        />
    </record>

    <record id="payment_method_paylib" model="payment.method">
        <field name="name">Paylib</field>
        <field name="code">paylib</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/paylib.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.fr'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.EUR'),
                     ])]"
        />
    </record>

    <record id="payment_method_payme" model="payment.method">
        <field name="name">PayMe</field>
        <field name="code">payme</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/payme.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.hk'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.HKD'),
                     ])]"
        />
    </record>

    <record id="payment_method_paynow" model="payment.method">
        <field name="name">PayNow</field>
        <field name="code">paynow</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/paynow.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.sg'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.SGD'),
                     ])]"
        />
    </record>

    <record id="payment_method_paypal" model="payment.method">
        <field name="name">Paypal</field>
        <field name="code">paypal</field>
        <field name="sequence">20</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/paypal.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
    </record>

    <record id="payment_method_paypay" model="payment.method">
        <field name="name">PayPay</field>
        <field name="code">paypay</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/paypay.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.jp'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.JPY'),
                     ])]"
        />
    </record>

    <record id="payment_method_paysafecard" model="payment.method">
        <field name="name">PaySafeCard</field>
        <field name="code">paysafecard</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/paysafecard.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="support_refund">full_only</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.at'),
                         ref('base.au'),
                         ref('base.be'),
                         ref('base.br'),
                         ref('base.ca'),
                         ref('base.hr'),
                         ref('base.cy'),
                         ref('base.cz'),
                         ref('base.dk'),
                         ref('base.fi'),
                         ref('base.fr'),
                         ref('base.ge'),
                         ref('base.de'),
                         ref('base.gi'),
                         ref('base.hu'),
                         ref('base.is'),
                         ref('base.ie'),
                         ref('base.it'),
                         ref('base.kw'),
                         ref('base.lv'),
                         ref('base.ie'),
                         ref('base.li'),
                         ref('base.lt'),
                         ref('base.lu'),
                         ref('base.mt'),
                         ref('base.mx'),
                         ref('base.md'),
                         ref('base.me'),
                         ref('base.nl'),
                         ref('base.nz'),
                         ref('base.no'),
                         ref('base.py'),
                         ref('base.pe'),
                         ref('base.pl'),
                         ref('base.pt'),
                         ref('base.ro'),
                         ref('base.sa'),
                         ref('base.rs'),
                         ref('base.sk'),
                         ref('base.si'),
                         ref('base.es'),
                         ref('base.se'),
                         ref('base.ch'),
                         ref('base.tr'),
                         ref('base.ae'),
                         ref('base.uk'),
                         ref('base.us'),
                         ref('base.uy'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.EUR'),
                         ref('base.AUD'),
                         ref('base.BRL'),
                         ref('base.CAD'),
                         ref('base.CZK'),
                         ref('base.DKK'),
                         ref('base.GEL'),
                         ref('base.GIP'),
                         ref('base.HUF'),
                         ref('base.ISK'),
                         ref('base.KWD'),
                         ref('base.CHF'),
                         ref('base.MXN'),
                         ref('base.MDL'),
                         ref('base.NZD'),
                         ref('base.NOK'),
                         ref('base.PYG'),
                         ref('base.PEN'),
                         ref('base.PLN'),
                         ref('base.RON'),
                         ref('base.SAR'),
                         ref('base.RSD'),
                         ref('base.SEK'),
                         ref('base.CHF'),
                         ref('base.TRY'),
                         ref('base.AED'),
                         ref('base.GBP'),
                         ref('base.USD'),
                         ref('base.UYU'),
                     ])]"
        />
    </record>

    <record id="payment_method_paytm" model="payment.method">
        <field name="name">Paytm</field>
        <field name="code">paytm</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/paytm.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.in'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.INR'),
                     ])]"
        />
    </record>

    <record id="payment_method_paytrail" model="payment.method">
        <field name="name">Paytrail</field>
        <field name="code">paytrail</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/paytrail.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.fi'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.EUR'),
                     ])]"
        />
    </record>

    <record id="payment_method_payu" model="payment.method">
        <field name="name">PayU</field>
        <field name="code">payu</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/payu.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.pl'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.PLN'),
                     ])]"
        />
    </record>

    <record id="payment_method_pix" model="payment.method">
        <field name="name">Pix</field>
        <field name="code">pix</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/pix.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.br'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.BRL'),
                     ])]"
        />
    </record>

    <record id="payment_method_poli" model="payment.method">
        <field name="name">POLi</field>
        <field name="code">poli</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/poli.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.au'),
                         ref('base.nz'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.AUD'),
                         ref('base.NZD'),
                     ])]"
        />
    </record>

    <record id="payment_method_post_finance" model="payment.method">
        <field name="name">PostFinance Pay</field>
        <field name="code">post_finance_pay</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/pf_pay.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.at'),
                         ref('base.be'),
                         ref('base.bg'),
                         ref('base.ch'),
                         ref('base.cy'),
                         ref('base.cz'),
                         ref('base.de'),
                         ref('base.dk'),
                         ref('base.ee'),
                         ref('base.es'),
                         ref('base.fi'),
                         ref('base.fr'),
                         ref('base.gr'),
                         ref('base.hr'),
                         ref('base.hu'),
                         ref('base.ie'),
                         ref('base.it'),
                         ref('base.lt'),
                         ref('base.lu'),
                         ref('base.lv'),
                         ref('base.mt'),
                         ref('base.nl'),
                         ref('base.pl'),
                         ref('base.pt'),
                         ref('base.ro'),
                         ref('base.se'),
                         ref('base.si'),
                         ref('base.sk'),
                         ref('base.uk'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.CHF'),
                         ref('base.EUR'),
                     ])]"
        />
    </record>

    <record id="payment_method_poste_pay" model="payment.method">
        <field name="name">PostePay</field>
        <field name="code">poste_pay</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/poste_pay.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.it'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.EUR'),
                     ])]"
        />
    </record>

    <record id="payment_method_pps" model="payment.method">
        <field name="name">PPS</field>
        <field name="code">pps</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/card.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.hk'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.HKD'),
                     ])]"
        />
    </record>

    <record id="payment_method_promptpay" model="payment.method">
        <field name="name">Prompt Pay</field>
        <field name="code">promptpay</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/promptpay.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.th'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.THB'),
                     ])]"
        />
    </record>

    <record id="payment_method_pse" model="payment.method">
        <field name="name">PSE</field>
        <field name="code">pse</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/card.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.co'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.COP'),
                     ])]"
        />
    </record>

    <record id="payment_method_p24" model="payment.method">
        <field name="name">P24</field>
        <field name="code">p24</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/p24.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.pl'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.EUR'),
                         ref('base.PLN'),
                     ])]"
        />
    </record>

    <record id="payment_method_qris" model="payment.method">
        <field name="name">QRIS</field>
        <field name="code">qris</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/qris.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.id'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.IDR'),
                     ])]"
        />
    </record>

     <record id="payment_method_rabbit_line_pay" model="payment.method">
        <field name="name">Rabbit LINE Pay</field>
        <field name="code">rabbit_line_pay</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/rabbit_line_pay.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.th'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.THB'),
                     ])]"
        />
    </record>

    <record id="payment_method_ratepay" model="payment.method">
        <field name="name">Ratepay</field>
        <field name="code">ratepay</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/ratepay.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.at'),
                         ref('base.de'),
                         ref('base.nl'),
                         ref('base.ch'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.EUR'),
                         ref('base.CHF'),
                     ])]"
        />
    </record>

    <record id="payment_method_revolut_pay" model="payment.method">
        <field name="name">Revolut Pay</field>
        <field name="code">revolut_pay</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/revolut_pay.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.uk'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.GBP'),
                     ])]"
        />
    </record>

    <record id="payment_method_samsung_pay" model="payment.method">
        <field name="name">Samsung Pay</field>
        <field name="code">samsung_pay</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/samsung_pay.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
    </record>


    <record id="payment_method_scb" model="payment.method">
        <field name="name">Siam Commerical Bank</field>
        <field name="code">scb</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/bank.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.th'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.THB'),
                     ])]"
        />
    </record>

    <record id="payment_method_sepa_direct_debit" model="payment.method">
        <field name="name">SEPA Direct Debit</field>
        <field name="code">sepa_direct_debit</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/sepa.png"/>
        <field name="support_tokenization">True</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.at'),
                         ref('base.be'),
                         ref('base.cy'),
                         ref('base.ee'),
                         ref('base.fi'),
                         ref('base.fr'),
                         ref('base.de'),
                         ref('base.gr'),
                         ref('base.ie'),
                         ref('base.it'),
                         ref('base.lv'),
                         ref('base.lt'),
                         ref('base.lu'),
                         ref('base.mt'),
                         ref('base.nl'),
                         ref('base.pt'),
                         ref('base.sk'),
                         ref('base.si'),
                         ref('base.es'),
                         ref('base.ch'),
                         ref('base.cz'),
                         ref('base.uk'),
                         ref('base.is'),
                         ref('base.hu'),
                         ref('base.ro'),
                         ref('base.se'),
                         ref('base.hr'),
                         ref('base.no'),
                         ref('base.bg'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.EUR'),
                     ])]"
        />
    </record>

    <record id="payment_method_shopback" model="payment.method">
        <field name="name">ShopBack</field>
        <field name="code">shopback</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/shopback.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.sg'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.SGD'),
                     ])]"
        />
    </record>

    <record id="payment_method_shopeepay" model="payment.method">
        <field name="name">ShopeePay</field>
        <field name="code">shopeepay</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/shopeepay.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.id'),
                         ref('base.ph'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.IDR'),

                     ])]"
        />
    </record>

    <record id="payment_method_sofort" model="payment.method">
        <field name="name">Sofort</field>
        <field name="code">sofort</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/sofort.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.de'),
                         ref('base.at'),
                         ref('base.be'),
                         ref('base.nl'),
                         ref('base.es'),
                         ref('base.ch'),
                         ref('base.pl'),
                         ref('base.it'),
                         ref('base.uk'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.EUR'),
                         ref('base.CHF'),
                     ])]"
        />
    </record>

    <record id="payment_method_spei" model="payment.method">
        <field name="name">SPEI</field>
        <field name="code">spei</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/spei.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.mx'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.MXN'),
                         ref('base.USD'),
                     ])]"
        />
    </record>

    <record id="payment_method_swish" model="payment.method">
        <field name="name">Swish</field>
        <field name="code">swish</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/swish.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.se'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.SEK'),
                     ])]"
        />
    </record>

    <record id="payment_method_techcom" model="payment.method">
        <field name="name">Techcombank</field>
        <field name="code">techcom</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/techcom.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.vn'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.VND'),
                     ])]"
        />
    </record>

    <record id="payment_method_tendopay" model="payment.method">
        <field name="name">TendoPay</field>
        <field name="code">tendopay</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/tendopay.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.ph'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.PHP'),
                     ])]"
        />
    </record>

    <record id="payment_method_tenpay" model="payment.method">
        <field name="name">TENPAY</field>
        <field name="code">tenpay</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/tenpay.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.cn'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.CNY'),
                     ])]"
        />
    </record>

    <record id="payment_method_tienphong" model="payment.method">
        <field name="name">Tienphong</field>
        <field name="code">tienphong</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/bank.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.vn'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.VND'),
                     ])]"
        />
    </record>

    <record id="payment_method_tinka" model="payment.method">
        <field name="name">Tinka</field>
        <field name="code">tinka</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/tinka.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.nl'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.EUR'),
                     ])]"
        />
    </record>

    <record id="payment_method_tmb" model="payment.method">
        <field name="name">Tamilnad Mercantile Bank Limited</field>
        <field name="code">tmb</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/tmb.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.th'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.THB'),
                     ])]"
        />
    </record>

    <record id="payment_method_toss_pay" model="payment.method">
        <field name="name">Toss Pay</field>
        <field name="code">toss_pay</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/toss_pay.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.kr'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.KRW'),
                     ])]"
        />
    </record>

    <record id="payment_method_touch_n_go" model="payment.method">
        <field name="name">Touch'n Go</field>
        <field name="code">touch_n_go</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/touch_n_go.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.my'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.MYR'),
                     ])]"
        />
    </record>

    <record id="payment_method_truemoney" model="payment.method">
        <field name="name">TrueMoney</field>
        <field name="code">truemoney</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/truemoney.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.th'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.THB'),
                     ])]"
        />
    </record>

    <record id="payment_method_trustly" model="payment.method">
        <field name="name">Trustly</field>
        <field name="code">trustly</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/trustly.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.at'),
                         ref('base.de'),
                         ref('base.dk'),
                         ref('base.ee'),
                         ref('base.es'),
                         ref('base.fi'),
                         ref('base.uk'),
                         ref('base.lv'),
                         ref('base.lt'),
                         ref('base.nl'),
                         ref('base.no'),
                         ref('base.se'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.GBP'),
                         ref('base.DKK'),
                         ref('base.SEK'),
                         ref('base.NOK'),
                         ref('base.EUR'),
                         ref('base.CZK'),
                     ])]"
        />
    </record>

    <record id="payment_method_ttb" model="payment.method">
        <field name="name">TTB</field>
        <field name="code">ttb</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/card.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.th'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.THB'),
                     ])]"
        />
    </record>

    <record id="payment_method_twint" model="payment.method">
        <field name="name">Twint</field>
        <field name="code">twint</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/twint.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.ch'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.CHF'),
                     ])]"
        />
    </record>

    <record id="payment_method_uatp" model="payment.method">
        <field name="name">Universal Air Travel Plan</field>
        <field name="code">uatp</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/uatp.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
    </record>

    <record id="payment_method_unknown" model="payment.method">
        <field name="name">Payment method</field>
        <field name="code">unknown</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/unknown.png"/>
        <field name="support_tokenization">True</field>
        <field name="support_express_checkout">True</field>
        <field name="support_refund">partial</field>
    </record>

    <record id="payment_method_uob" model="payment.method">
        <field name="name">United Overseas Bank</field>
        <field name="code">uob</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/bank.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.sg'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.SGD'),
                     ])]"
        />
    </record>

    <record id="payment_method_upi" model="payment.method">
        <field name="name">UPI</field>
        <field name="code">upi</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/upi.png"/>
        <field name="support_tokenization">True</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.in'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.INR'),
                     ])]"
        />
    </record>

    <record id="payment_method_ussd" model="payment.method">
        <field name="name">USSD</field>
        <field name="code">ussd</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/flutterwave.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
    </record>

    <record id="payment_method_venmo" model="payment.method">
        <field name="name">Venmo</field>
        <field name="code">venmo</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/venmo.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.us'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.USD'),
                     ])]"
        />
    </record>

    <record id="payment_method_vietcom" model="payment.method">
        <field name="name">Vietcombank</field>
        <field name="code">vietcom</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/vietcom.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.vn'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.VND'),
                     ])]"
        />
    </record>

    <record id="payment_method_vipps" model="payment.method">
        <field name="name">Vipps</field>
        <field name="code">vipps</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/vipps.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.no'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.NOK'),
                     ])]"
        />
    </record>

    <record id="payment_method_vpay" model="payment.method">
        <field name="name">V PAY</field>
        <field name="code">vpay</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/vpay.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.EUR'),
                         ref('base.GBP'),
                         ref('base.PLN'),
                         ref('base.DKK'),
                         ref('base.NOK'),
                         ref('base.SEK'),
                         ref('base.CHF'),
                         ref('base.USD'),
                     ])]"
        />
    </record>

    <record id="payment_method_wallets_india" model="payment.method">
        <field name="name">Wallets India</field>
        <field name="code">wallets_india</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/wallet.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="support_refund">full_only</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.in'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.INR'),
                     ])]"
        />
    </record>

    <record id="payment_method_walley" model="payment.method">
        <field name="name">Walley</field>
        <field name="code">walley</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/walley.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.dk'),
                         ref('base.fi'),
                         ref('base.no'),
                         ref('base.se'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.DKK'),
                         ref('base.EUR'),
                         ref('base.NOK'),
                         ref('base.SEK'),
                     ])]"
        />
    </record>

    <record id="payment_method_webpay" model="payment.method">
        <field name="name">WebPay</field>
        <field name="code">webpay</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/webpay.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.cl'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.CLP'),
                         ref('base.USD'),
                     ])]"
        />
    </record>

    <record id="payment_method_wechat_pay" model="payment.method">
        <field name="name">WeChat Pay</field>
        <field name="code">wechat_pay</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/wechat_pay.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.AUD'),
                         ref('base.EUR'),
                         ref('base.CAD'),
                         ref('base.CNY'),
                         ref('base.HKD'),
                         ref('base.JPY'),
                         ref('base.NZD'),
                         ref('base.GBP'),
                         ref('base.USD'),
                         ref('base.SGD'),
                     ])]"
        />
    </record>

    <record id="payment_method_welend" model="payment.method">
        <field name="name">WeLend</field>
        <field name="code">welend</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/welend.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.hk'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.HKD'),
                     ])]"
        />
    </record>

    <record id="payment_method_zalopay" model="payment.method">
        <field name="name">Zalopay</field>
        <field name="code">zalopay</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/zalopay.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.vn'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.VND'),
                     ])]"
        />
    </record>

    <record id="payment_method_zip" model="payment.method">
        <field name="name">Zip</field>
        <field name="code">zip</field>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/zip.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
        <field name="supported_country_ids"
               eval="[Command.set([
                         ref('base.au'),
                         ref('base.ca'),
                         ref('base.nz'),
                         ref('base.us'),
                     ])]"
        />
        <field name="supported_currency_ids"
               eval="[Command.set([
                         ref('base.AUD'),
                         ref('base.CAD'),
                         ref('base.NZD'),
                         ref('base.USD'),
                     ])]"
        />
    </record>

    <!-- === PAYMENT METHOD BRANDS === -->

    <record id="payment_method_abitab" model="payment.method">
        <field name="name">Abitab</field>
        <field name="code">abitab</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_nuvei_local')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/abitab.png"/>
    </record>

    <record id="payment_method_amex" model="payment.method">
        <field name="name">American Express</field>
        <field name="code">amex</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_card')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/amex.png"/>
    </record>

    <record id="payment_method_argencard" model="payment.method">
        <field name="name">Argencard</field>
        <field name="code">argencard</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_card')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/argencard.png"/>
    </record>

    <record id="payment_method_banco_de_bogota" model="payment.method">
        <field name="name">Banco de Bogota</field>
        <field name="code">banco_de_bogota</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_bank_reference')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/bank.png"/>
    </record>

    <record id="payment_method_banco_guayaquil" model="payment.method">
        <field name="name">Banco Guayaquil</field>
        <field name="code">banco_guayaquil</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_astropay')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/banco_guayaquil.png"/>
    </record>

    <record id="payment_method_bancolombia" model="payment.method">
        <field name="name">Bancolombia</field>
        <field name="code">bancolombia</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_bank_reference')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/bancolombia.png"/>
    </record>

    <record id="payment_method_banco_pichincha" model="payment.method">
        <field name="name">Banco Pichincha</field>
        <field name="code">banco_pichincha</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_astropay')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/banco_pichincha.png"/>
    </record>

    <record id="payment_method_cabal" model="payment.method">
        <field name="name">Cabal</field>
        <field name="code">cabal</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_card')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/cabal.png"/>
    </record>

    <record id="payment_method_caixa" model="payment.method">
        <field name="name">Caixa</field>
        <field name="code">caixa</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_card')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/caixa.png"/>
    </record>

    <record id="payment_method_carnet" model="payment.method">
        <field name="name">Carnet</field>
        <field name="code">carnet</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_card')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/wallet.png"/>
    </record>

    <record id="payment_method_cartes_bancaires" model="payment.method">
        <field name="name">Cartes Bancaires</field>
        <field name="code">cartes_bancaires</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_card')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/card.png"/>
    </record>

    <record id="payment_method_cencosud" model="payment.method">
        <field name="name">Cencosud</field>
        <field name="code">cencosud</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_card')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/cencosud.png"/>
    </record>

    <record id="payment_method_cirrus" model="payment.method">
        <field name="name">Cirrus</field>
        <field name="code">cirrus</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_card')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/cirrus.png"/>
    </record>

    <record id="payment_method_cmr" model="payment.method">
        <field name="name">CMR</field>
        <field name="code">cmr</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_card')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/card.png"/>
    </record>

    <record id="payment_method_codensa" model="payment.method">
        <field name="name">Codensa</field>
        <field name="code">codensa</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_card')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/codensa.png"/>
    </record>

    <record id="payment_method_cordial" model="payment.method">
        <field name="name">Cordial</field>
        <field name="code">cordial</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_card')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/cordial.png"/>
    </record>

    <record id="payment_method_cordobesa" model="payment.method">
        <field name="name">Cordobesa</field>
        <field name="code">cordobesa</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_card')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/cordobesa.png"/>
    </record>

    <record id="payment_method_credit" model="payment.method">
        <field name="name">Credit Payment</field>
        <field name="code">credit</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_card')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/card.png"/>
    </record>

    <record id="payment_method_dankort" model="payment.method">
        <field name="name">Dankort</field>
        <field name="code">dankort</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_card')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/dankort.png"/>
    </record>

    <record id="payment_method_davivienda" model="payment.method">
        <field name="name">Davivienda</field>
        <field name="code">davivienda</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_bank_reference')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/davivienda.png"/>
    </record>

    <record id="payment_method_diners" model="payment.method">
        <field name="name">Diners Club International</field>
        <field name="code">diners</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_card')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/diners.png"/>
    </record>

    <record id="payment_method_discover" model="payment.method">
        <field name="name">Discover</field>
        <field name="code">discover</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_card')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/discover.png"/>
    </record>

    <record id="payment_method_elo" model="payment.method">
        <field name="name">Elo</field>
        <field name="code">elo</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_card')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/elo.png"/>
    </record>

    <record id="payment_method_facilito" model="payment.method">
        <field name="name">Facilito</field>
        <field name="code">facilito</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_astropay')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/facilito.png"/>
    </record>

    <record id="payment_method_hipercard" model="payment.method">
        <field name="name">Hipercard</field>
        <field name="code">hipercard</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_card')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/hipercard.png"/>
    </record>

    <record id="payment_method_jcb" model="payment.method">
        <field name="name">JCB</field>
        <field name="code">jcb</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_card')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/jcb.png"/>
    </record>

    <record id="payment_method_lider" model="payment.method">
        <field name="name">Lider</field>
        <field name="code">lider</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_card')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/lider.png"/>
    </record>

    <record id="payment_method_mercado_livre" model="payment.method">
        <field name="name">Mercado Livre</field>
        <field name="code">mercado_livre</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_card')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/mercado_livre.png"/>
    </record>


    <record id="payment_method_meeza" model="payment.method">
        <field name="name">Meeza</field>
        <field name="code">meeza</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_card')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/meeza.png"/>
    </record>

    <record id="payment_method_maestro" model="payment.method">
        <field name="name">Maestro</field>
        <field name="code">maestro</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_card')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/maestro.png"/>
    </record>

    <record id="payment_method_magna" model="payment.method">
        <field name="name">Magna</field>
        <field name="code">magna</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_card')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/magna.png"/>
    </record>

    <record id="payment_method_mastercard" model="payment.method">
        <field name="name">MasterCard</field>
        <field name="code">mastercard</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_card')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/mastercard.png"/>
    </record>

    <record id="payment_method_naps" model="payment.method">
        <field name="name">NAPS</field>
        <field name="code">naps</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_card')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/naps.png"/>
    </record>

    <record id="payment_method_naranja" model="payment.method">
        <field name="name">Naranja</field>
        <field name="code">naranja</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_card')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/naranja.png"/>
    </record>

    <record id="payment_method_nativa" model="payment.method">
        <field name="name">Nativa</field>
        <field name="code">nativa</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_card')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/nativa.png"/>
    </record>

    <record id="payment_method_oca" model="payment.method">
        <field name="name">Oca</field>
        <field name="code">oca</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_card')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/card.png"/>
    </record>

    <record id="payment_method_omannet" model="payment.method">
        <field name="name">OmanNet</field>
        <field name="code">omannet</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_card')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/omannet.png"/>
    </record>

    <record id="payment_method_presto" model="payment.method">
        <field name="name">Presto</field>
        <field name="code">presto</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_card')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/presto.png"/>
    </record>

    <record id="payment_method_redpagos" model="payment.method">
        <field name="name">Redpagos</field>
        <field name="code">redpagos</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_nuvei_local')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/redpagos.png"/>
    </record>

    <record id="payment_method_rupay" model="payment.method">
        <field name="name">RuPay</field>
        <field name="code">rupay</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_card')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/rupay.png"/>
    </record>

    <record id="payment_method_shopping" model="payment.method">
        <field name="name">Shopping Card</field>
        <field name="code">shopping</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_card')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/shopping.png"/>
    </record>

    <record id="payment_method_tarjeta_mercadopago" model="payment.method">
        <field name="name">Tarjeta MercadoPago</field>
        <field name="code">tarjeta_mercadopago</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_card')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/tarjeta_mercadopago.png"/>
    </record>

    <record id="payment_method_unionpay" model="payment.method">
        <field name="name">UnionPay</field>
        <field name="code">unionpay</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_card')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/unionpay.png"/>
    </record>

    <record id="payment_method_visa" model="payment.method">
        <field name="name">VISA</field>
        <field name="code">visa</field>
        <field name="primary_payment_method_id" eval="ref('payment.payment_method_card')"/>
        <field name="sequence">1000</field>
        <field name="active">False</field>
        <field name="image" type="base64" file="payment/static/img/visa.png"/>
    </record>

</odoo>

```

## File: data\payment_provider_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="payment_provider_adyen" model="payment.provider">
        <field name="name">Adyen</field>
        <field name="image_128" type="base64" file="payment_adyen/static/description/icon.png"/>
        <field name="module_id" ref="base.module_payment_adyen"/>
        <!-- https://www.adyen.com/payment-methods -->
        <field name="payment_method_ids"
               eval="[Command.set([
                         ref('payment.payment_method_ach_direct_debit'),
                         ref('payment.payment_method_affirm'),
                         ref('payment.payment_method_afterpay'),
                         ref('payment.payment_method_alipay'),
                         ref('payment.payment_method_alipay_hk'),
                         ref('payment.payment_method_alma'),
                         ref('payment.payment_method_bacs_direct_debit'),
                         ref('payment.payment_method_bancontact'),
                         ref('payment.payment_method_benefit'),
                         ref('payment.payment_method_bizum'),
                         ref('payment.payment_method_blik'),
                         ref('payment.payment_method_card'),
                         ref('payment.payment_method_cash_app_pay'),
                         ref('payment.payment_method_clearpay'),
                         ref('payment.payment_method_dana'),
                         ref('payment.payment_method_duitnow'),
                         ref('payment.payment_method_elo'),
                         ref('payment.payment_method_eps'),
                         ref('payment.payment_method_fpx'),
                         ref('payment.payment_method_gcash'),
                         ref('payment.payment_method_giropay'),
                         ref('payment.payment_method_gopay'),
                         ref('payment.payment_method_hipercard'),
                         ref('payment.payment_method_ideal'),
                         ref('payment.payment_method_kakaopay'),
                         ref('payment.payment_method_klarna'),
                         ref('payment.payment_method_klarna_paynow'),
                         ref('payment.payment_method_klarna_pay_over_time'),
                         ref('payment.payment_method_knet'),
                         ref('payment.payment_method_mbway'),
                         ref('payment.payment_method_mobile_pay'),
                         ref('payment.payment_method_momo'),
                         ref('payment.payment_method_multibanco'),
                         ref('payment.payment_method_napas_card'),
                         ref('payment.payment_method_online_banking_czech_republic'),
                         ref('payment.payment_method_online_banking_india'),
                         ref('payment.payment_method_online_banking_slovakia'),
                         ref('payment.payment_method_online_banking_thailand'),
                         ref('payment.payment_method_open_banking'),
                         ref('payment.payment_method_p24'),
                         ref('payment.payment_method_paybright'),
                         ref('payment.payment_method_paysafecard'),
                         ref('payment.payment_method_paynow'),
                         ref('payment.payment_method_paypal'),
                         ref('payment.payment_method_paytm'),
                         ref('payment.payment_method_paytrail'),
                         ref('payment.payment_method_pix'),
                         ref('payment.payment_method_promptpay'),
                         ref('payment.payment_method_ratepay'),
                         ref('payment.payment_method_samsung_pay'),
                         ref('payment.payment_method_sepa_direct_debit'),
                         ref('payment.payment_method_sofort'),
                         ref('payment.payment_method_swish'),
                         ref('payment.payment_method_touch_n_go'),
                         ref('payment.payment_method_trustly'),
                         ref('payment.payment_method_twint'),
                         ref('payment.payment_method_upi'),
                         ref('payment.payment_method_vipps'),
                         ref('payment.payment_method_wallets_india'),
                         ref('payment.payment_method_walley'),
                         ref('payment.payment_method_wechat_pay'),
                         ref('payment.payment_method_zip'),
                     ])]"
        />
    </record>

    <record id="payment_provider_aps" model="payment.provider">
        <field name="name">Amazon Payment Services</field>
        <field name="image_128" type="base64" file="payment_aps/static/description/icon.png"/>
        <field name="module_id" ref="base.module_payment_aps"/>
        <!-- https://paymentservices.amazon.com/docs/EN/24.html -->
        <field name="payment_method_ids"
               eval="[Command.set([
                         ref('payment.payment_method_card'),
                         ref('payment.payment_method_mada'),
                         ref('payment.payment_method_knet'),
                         ref('payment.payment_method_meeza'),
                         ref('payment.payment_method_naps'),
                         ref('payment.payment_method_omannet'),
                         ref('payment.payment_method_benefit'),
                     ])]"
        />
    </record>

    <record id="payment_provider_asiapay" model="payment.provider">
        <field name="name">Asiapay</field>
        <field name="image_128" type="base64" file="payment_asiapay/static/description/icon.png"/>
        <field name="module_id" ref="base.module_payment_asiapay"/>
        <!-- See https://www.asiapay.com/payment.html#option -->
        <field name="payment_method_ids"
               eval="[Command.set([
                         ref('payment.payment_method_card'),
                         ref('payment.payment_method_alipay'),
                         ref('payment.payment_method_wechat_pay'),
                         ref('payment.payment_method_poli'),
                         ref('payment.payment_method_afterpay'),
                         ref('payment.payment_method_clearpay'),
                         ref('payment.payment_method_humm'),
                         ref('payment.payment_method_zip'),
                         ref('payment.payment_method_paypal'),
                         ref('payment.payment_method_atome'),
                         ref('payment.payment_method_pace'),
                         ref('payment.payment_method_shopback'),
                         ref('payment.payment_method_grabpay'),
                         ref('payment.payment_method_samsung_pay'),
                         ref('payment.payment_method_hoolah'),
                         ref('payment.payment_method_boost'),
                         ref('payment.payment_method_duitnow'),
                         ref('payment.payment_method_touch_n_go'),
                         ref('payment.payment_method_bancnet'),
                         ref('payment.payment_method_gcash'),
                         ref('payment.payment_method_paynow'),
                         ref('payment.payment_method_linepay'),
                         ref('payment.payment_method_bangkok_bank'),
                         ref('payment.payment_method_krungthai_bank'),
                         ref('payment.payment_method_uob'),
                         ref('payment.payment_method_scb'),
                         ref('payment.payment_method_bank_of_ayudhya'),
                         ref('payment.payment_method_kasikorn_bank'),
                         ref('payment.payment_method_rabbit_line_pay'),
                         ref('payment.payment_method_truemoney'),
                         ref('payment.payment_method_fpx'),
                         ref('payment.payment_method_fps'),
                         ref('payment.payment_method_hd'),
                         ref('payment.payment_method_maybank'),
                         ref('payment.payment_method_pay_id'),
                         ref('payment.payment_method_promptpay'),
                         ref('payment.payment_method_techcom'),
                         ref('payment.payment_method_tienphong'),
                         ref('payment.payment_method_ttb'),
                         ref('payment.payment_method_upi'),
                         ref('payment.payment_method_vietcom'),
                         ref('payment.payment_method_tendopay'),
                         ref('payment.payment_method_alipay_hk'),
                         ref('payment.payment_method_bharatqr'),
                         ref('payment.payment_method_momo'),
                         ref('payment.payment_method_octopus'),
                         ref('payment.payment_method_maya'),
                         ref('payment.payment_method_uatp'),
                         ref('payment.payment_method_tenpay'),
                         ref('payment.payment_method_enets'),
                         ref('payment.payment_method_jkopay'),
                         ref('payment.payment_method_payme'),
                         ref('payment.payment_method_tmb'),
                     ])]"
        />
    </record>

    <record id="payment_provider_authorize" model="payment.provider">
        <field name="name">Authorize.net</field>
        <field name="image_128"
               type="base64"
               file="payment_authorize/static/description/icon.png"/>
        <field name="module_id" ref="base.module_payment_authorize"/>
        <!-- https://www.authorize.net/solutions/merchantsolutions/onlinemerchantaccount/ -->
        <field name="payment_method_ids"
               eval="[Command.set([
                         ref('payment.payment_method_ach_direct_debit'),
                         ref('payment.payment_method_card'),
                     ])]"
        />
    </record>

    <record id="payment_provider_buckaroo" model="payment.provider">
        <field name="name">Buckaroo</field>
        <field name="image_128"
               type="base64"
               file="payment_buckaroo/static/description/icon.png"/>
        <field name="module_id" ref="base.module_payment_buckaroo"/>
        <!-- https://www.buckaroo-payments.com/products/payment-methods/ -->
        <field name="payment_method_ids"
               eval="[Command.set([
                         ref('payment.payment_method_bancontact'),
                         ref('payment.payment_method_bank_reference'),
                         ref('payment.payment_method_card'),
                         ref('payment.payment_method_paypal'),
                         ref('payment.payment_method_ideal'),
                         ref('payment.payment_method_afterpay_riverty'),
                         ref('payment.payment_method_sepa_direct_debit'),
                         ref('payment.payment_method_alipay'),
                         ref('payment.payment_method_wechat_pay'),
                         ref('payment.payment_method_klarna'),
                         ref('payment.payment_method_trustly'),
                         ref('payment.payment_method_sofort'),
                         ref('payment.payment_method_in3'),
                         ref('payment.payment_method_tinka'),
                         ref('payment.payment_method_billink'),
                         ref('payment.payment_method_kbc_cbc'),
                         ref('payment.payment_method_belfius'),
                         ref('payment.payment_method_giropay'),
                         ref('payment.payment_method_p24'),
                         ref('payment.payment_method_poste_pay'),
                         ref('payment.payment_method_eps'),
                         ref('payment.payment_method_cartes_bancaires'),
                     ])]"
        />
    </record>

    <record id="payment_provider_demo" model="payment.provider">
        <field name="name">Demo</field>
        <field name="sequence">40</field>
        <field name="image_128" type="base64" file="payment_demo/static/description/icon.png"/>
        <field name="module_id" ref="base.module_payment_demo"/>
    </record>

    <record id="payment_provider_flutterwave" model="payment.provider">
        <field name="name">Flutterwave</field>
        <field name="image_128"
               type="base64"
               file="payment_flutterwave/static/description/icon.png"/>
        <field name="module_id" ref="base.module_payment_flutterwave"/>
        <!-- https://developer.flutterwave.com/docs/collecting-payments/payment-methods/ -->
        <field name="payment_method_ids"
               eval="[Command.set([
                         ref('payment.payment_method_card'),
                         ref('payment.payment_method_mpesa'),
                         ref('payment.payment_method_mobile_money'),
                         ref('payment.payment_method_bank_transfer'),
                         ref('payment.payment_method_bank_account'),
                         ref('payment.payment_method_credit'),
                         ref('payment.payment_method_paypal'),
                         ref('payment.payment_method_ussd'),
                     ])]"
        />
    </record>

    <record id="payment_provider_mercado_pago" model="payment.provider">
        <field name="name">Mercado Pago</field>
        <field name="image_128"
               type="base64"
               file="payment_mercado_pago/static/description/icon.png"/>
        <field name="module_id" ref="base.module_payment_mercado_pago"/>

         <!-- Payment methods must be fetched from the API. See
            https://www.mercadopago.com.ar/developers/en/reference/payment_methods/_payment_methods/
        -->
        <field name="payment_method_ids"
               eval="[Command.set([
                         ref('payment.payment_method_card'),
                         ref('payment.payment_method_bank_transfer'),
                         ref('payment.payment_method_paypal'),
                     ])]"
        />
    </record>

    <record id="payment_provider_mollie" model="payment.provider">
        <field name="name">Mollie</field>
        <field name="image_128" type="base64" file="payment_mollie/static/description/icon.png"/>
        <field name="module_id" ref="base.module_payment_mollie"/>
        <!-- https://www.mollie.com/en/payments -->
        <field name="payment_method_ids"
               eval="[Command.set([
                         ref('payment.payment_method_bancontact'),
                         ref('payment.payment_method_bank_transfer'),
                         ref('payment.payment_method_belfius'),
                         ref('payment.payment_method_card'),
                         ref('payment.payment_method_eps'),
                         ref('payment.payment_method_giropay'),
                         ref('payment.payment_method_ideal'),
                         ref('payment.payment_method_kbc_cbc'),
                         ref('payment.payment_method_paypal'),
                         ref('payment.payment_method_paysafecard'),
                         ref('payment.payment_method_p24'),
                         ref('payment.payment_method_sofort'),
                         ref('payment.payment_method_twint'),
                     ])]"
        />

    </record>

    <record id="payment_provider_nuvei" model="payment.provider">
        <field name="name">Nuvei</field>
        <field name="image_128" type="base64" file="payment_nuvei/static/description/icon.png"/>
        <field name="module_id" ref="base.module_payment_nuvei"/>
        <field name="payment_method_ids"
               eval="[Command.set([
                         ref('payment.payment_method_astropay'),
                         ref('payment.payment_method_boleto'),
                         ref('payment.payment_method_card'),
                         ref('payment.payment_method_nuvei_local'),
                         ref('payment.payment_method_oxxopay'),
                         ref('payment.payment_method_pix'),
                         ref('payment.payment_method_pse'),
                         ref('payment.payment_method_spei'),
                         ref('payment.payment_method_webpay'),
                     ])]"
        />
    </record>

    <record id="payment_provider_paypal" model="payment.provider">
        <field name="name">PayPal</field>
        <field name="image_128" type="base64" file="payment_paypal/static/description/icon.png"/>
        <field name="module_id" ref="base.module_payment_paypal"/>
        <!-- https://www.paypal.com/us/selfhelp/article/Which-credit-cards-can-I-accept-with-PayPal-Merchant-Services-FAQ1525#business -->
        <field name="payment_method_ids"
               eval="[Command.set([
                         ref('payment.payment_method_paypal'),
                     ])]"
        />
    </record>

    <record id="payment_provider_razorpay" model="payment.provider">
        <field name="name">Razorpay</field>
        <field name="image_128" type="base64" file="payment_razorpay/static/description/icon.png"/>
        <field name="module_id" ref="base.module_payment_razorpay"/>
        <!-- https://razorpay.com/docs/payments/payment-methods/#supported-payment-methods -->
        <field name="payment_method_ids"
               eval="[Command.set([
                         ref('payment.payment_method_card'),
                         ref('payment.payment_method_netbanking'),
                         ref('payment.payment_method_upi'),
                         ref('payment.payment_method_wallets_india'),
                         ref('payment.payment_method_paylater_india'),
                         ref('payment.payment_method_emi_india'),
                     ])]"
        />
    </record>

    <record id="payment_provider_sepa_direct_debit" model="payment.provider">
        <field name="name">SEPA Direct Debit</field>
        <field name="sequence">20</field>
        <field name="image_128"
               type="base64"
               file="base/static/img/icons/payment_sepa_direct_debit.png"/>
        <field name="module_id" ref="base.module_payment_sepa_direct_debit"/>
        <field name="payment_method_ids"
           eval="[Command.set([
                     ref('payment.payment_method_sepa_direct_debit'),
                 ])]"
        />
    </record>

    <record id="payment_provider_stripe" model="payment.provider">
        <field name="name">Stripe</field>
        <field name="image_128" type="base64" file="payment_stripe/static/description/icon.png"/>
        <field name="module_id" ref="base.module_payment_stripe"/>
        <!--
            See https://stripe.com/payments/payment-methods-guide
            See https://support.goteamup.com/hc/en-us/articles/115002089349-Which-cards-and-payment-types-can-I-accept-with-Stripe-
        -->
        <field name="payment_method_ids"
               eval="[Command.set([
                         ref('payment.payment_method_ach_direct_debit'),
                         ref('payment.payment_method_affirm'),
                         ref('payment.payment_method_afterpay'),
                         ref('payment.payment_method_alipay'),
                         ref('payment.payment_method_bacs_direct_debit'),
                         ref('payment.payment_method_bancontact'),
                         ref('payment.payment_method_becs_direct_debit'),
                         ref('payment.payment_method_boleto'),
                         ref('payment.payment_method_card'),
                         ref('payment.payment_method_cash_app_pay'),
                         ref('payment.payment_method_clearpay'),
                         ref('payment.payment_method_eps'),
                         ref('payment.payment_method_fpx'),
                         ref('payment.payment_method_giropay'),
                         ref('payment.payment_method_grabpay'),
                         ref('payment.payment_method_ideal'),
                         ref('payment.payment_method_klarna'),
                         ref('payment.payment_method_mobile_pay'),
                         ref('payment.payment_method_multibanco'),
                         ref('payment.payment_method_p24'),
                         ref('payment.payment_method_paynow'),
                         ref('payment.payment_method_paypal'),
                         ref('payment.payment_method_pix'),
                         ref('payment.payment_method_promptpay'),
                         ref('payment.payment_method_revolut_pay'),
                         ref('payment.payment_method_sepa_direct_debit'),
                         ref('payment.payment_method_sofort'),
                         ref('payment.payment_method_upi'),
                         ref('payment.payment_method_wechat_pay'),
                         ref('payment.payment_method_zip'),
                     ])]"
        />
    </record>

    <record id="payment_provider_transfer" model="payment.provider">
        <field name="name">Wire Transfer</field>
        <field name="sequence">30</field>
        <field name="image_128" type="base64" file="payment_custom/static/description/icon.png"/>
        <field name="module_id" ref="base.module_payment_custom"/>
    </record>

    <record id="payment_provider_worldline" model="payment.provider">
        <field name="name">Worldline</field>
        <field name="image_128" type="base64" file="payment_worldline/static/description/icon.png"/>
        <field name="module_id" ref="base.module_payment_worldline"/>
        <!-- https://docs.direct.worldline-solutions.com/en/payment-methods-and-features/index -->
        <field name="payment_method_ids"
               eval="[Command.set([
                         ref('payment.payment_method_alipay_plus'),
                         ref('payment.payment_method_bancontact'),
                         ref('payment.payment_method_bizum'),
                         ref('payment.payment_method_card'),
                         ref('payment.payment_method_cofidis'),
                         ref('payment.payment_method_eps'),
                         ref('payment.payment_method_floa_bank'),
                         ref('payment.payment_method_ideal'),
                         ref('payment.payment_method_klarna'),
                         ref('payment.payment_method_mbway'),
                         ref('payment.payment_method_multibanco'),
                         ref('payment.payment_method_p24'),
                         ref('payment.payment_method_paypal'),
                         ref('payment.payment_method_post_finance'),
                         ref('payment.payment_method_twint'),
                         ref('payment.payment_method_wechat_pay'),
                     ])]"
        />
    </record>

    <record id="payment_provider_xendit" model="payment.provider">
        <field name="name">Xendit</field>
        <field name="image_128"
               type="base64"
               file="payment_xendit/static/description/icon.png"
        />
        <field name="module_id" ref="base.module_payment_xendit"/>
        <!-- See https://docs.xendit.co/payment-link/payment-channels for payment methods. -->
        <field name="payment_method_ids"
               eval="[(6, 0, [
                   ref('payment.payment_method_7eleven'),
                   ref('payment.payment_method_akulaku'),
                   ref('payment.payment_method_bank_bca'),
                   ref('payment.payment_method_bank_permata'),
                   ref('payment.payment_method_billease'),
                   ref('payment.payment_method_bni'),
                   ref('payment.payment_method_bri'),
                   ref('payment.payment_method_bsi'),
                   ref('payment.payment_method_card'),
                   ref('payment.payment_method_cashalo'),
                   ref('payment.payment_method_cebuana'),
                   ref('payment.payment_method_cimb_niaga'),
                   ref('payment.payment_method_dana'),
                   ref('payment.payment_method_gcash'),
                   ref('payment.payment_method_grabpay'),
                   ref('payment.payment_method_jeniuspay'),
                   ref('payment.payment_method_kredivo'),
                   ref('payment.payment_method_linkaja'),
                   ref('payment.payment_method_mandiri'),
                   ref('payment.payment_method_maya'),
                   ref('payment.payment_method_ovo'),
                   ref('payment.payment_method_qris'),
                   ref('payment.payment_method_shopeepay'),
               ])]"/>
    </record>

</odoo>

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

## File: models\onboarding_step.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class OnboardingStep(models.Model):
    _inherit = 'onboarding.onboarding.step'

    @api.model
    def action_validate_step_payment_provider(self):
        """ Override of `onboarding` to validate other steps as well. """
        return self.action_validate_step('payment.onboarding_onboarding_step_payment_provider')

```

## File: models\payment_method.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import Command, _, api, fields, models
from odoo.exceptions import UserError
from odoo.osv import expression

from odoo.addons.payment import utils as payment_utils
from odoo.addons.payment.const import REPORT_REASONS_MAPPING


class PaymentMethod(models.Model):
    _name = 'payment.method'
    _description = "Payment Method"
    _order = 'active desc, sequence, name'

    name = fields.Char(string="Name", required=True, translate=True)
    code = fields.Char(
        string="Code", help="The technical code of this payment method.", required=True
    )
    sequence = fields.Integer(string="Sequence", default=1)
    primary_payment_method_id = fields.Many2one(
        string="Primary Payment Method",
        help="The primary payment method of the current payment method, if the latter is a brand."
             "\nFor example, \"Card\" is the primary payment method of the card brand \"VISA\".",
        comodel_name='payment.method',
    )
    brand_ids = fields.One2many(
        string="Brands",
        help="The brands of the payment methods that will be displayed on the payment form.",
        comodel_name='payment.method',
        inverse_name='primary_payment_method_id',
    )
    is_primary = fields.Boolean(
        string="Is Primary Payment Method",
        compute='_compute_is_primary',
        search='_search_is_primary',
    )
    provider_ids = fields.Many2many(
        string="Providers",
        help="The list of providers supporting this payment method.",
        comodel_name='payment.provider',
    )
    active = fields.Boolean(string="Active", default=True)
    image = fields.Image(
        string="Image",
        help="The base image used for this payment method; in a 64x64 px format.",
        max_width=64,
        max_height=64,
        required=True,
    )
    image_payment_form = fields.Image(
        string="The resized image displayed on the payment form.",
        related='image',
        store=True,
        max_width=45,
        max_height=30,
    )

    # Feature support fields.
    support_tokenization = fields.Boolean(
        string="Tokenization",
        help="Tokenization is the process of saving the payment details as a token that can later"
             " be reused without having to enter the payment details again.",
    )
    support_express_checkout = fields.Boolean(
        string="Express Checkout",
        help="Express checkout allows customers to pay faster by using a payment method that"
             " provides all required billing and shipping information, thus allowing to skip the"
             " checkout process.",
    )
    support_refund = fields.Selection(
        string="Refund",
        help="Refund is a feature allowing to refund customers directly from the payment in Odoo.",
        selection=[
            ('none', "Unsupported"),
            ('full_only', "Full Only"),
            ('partial', "Full & Partial"),
        ],
        required=True,
        default='none',
    )
    supported_country_ids = fields.Many2many(
        string="Countries",
        comodel_name='res.country',
        help="The list of countries in which this payment method can be used (if the provider"
             " allows it). In other countries, this payment method is not available to customers."
    )
    supported_currency_ids = fields.Many2many(
        string="Currencies",
        comodel_name='res.currency',
        help="The list of currencies for that are supported by this payment method (if the provider"
             " allows it). When paying with another currency, this payment method is not available "
             "to customers.",
        context={'active_test': False},
    )

    #=== COMPUTE METHODS ===#

    def _compute_is_primary(self):
        for payment_method in self:
            payment_method.is_primary = not payment_method.primary_payment_method_id

    def _search_is_primary(self, operator, value):
        if operator == '=' and value is True:
            return [('primary_payment_method_id', '=', False)]
        elif operator == '=' and value is False:
            return [('primary_payment_method_id', '!=', False)]
        else:
            raise NotImplementedError(_("Operation not supported."))

    #=== ONCHANGE METHODS ===#

    @api.onchange('active', 'provider_ids', 'support_tokenization')
    def _onchange_warn_before_disabling_tokens(self):
        """ Display a warning about the consequences of archiving the payment method, detaching it
        from a provider, or removing its support for tokenization.

        Let the user know that the related tokens will be archived.

        :return: A client action with the warning message, if any.
        :rtype: dict
        """
        disabling = self._origin.active and not self.active
        detached_providers = self._origin.provider_ids.filtered(
            lambda p: p.id not in self.provider_ids.ids
        )  # Cannot use recordset difference operation because self.provider_ids is a set of NewIds.
        blocking_tokenization = self._origin.support_tokenization and not self.support_tokenization
        if disabling or detached_providers or blocking_tokenization:
            related_tokens = self.env['payment.token'].with_context(active_test=True).search(
                expression.AND([
                    [('payment_method_id', 'in', (self._origin + self._origin.brand_ids).ids)],
                    [('provider_id', 'in', detached_providers.ids)] if detached_providers else [],
                ])
            )  # Fix `active_test` in the context forwarded by the view.
            if related_tokens:
                return {
                    'warning': {
                        'title': _("Warning"),
                        'message': _(
                            "This action will also archive %s tokens that are registered with this "
                            "payment method.", len(related_tokens)
                        )
                    }
                }

    @api.onchange('provider_ids')
    def _onchange_provider_ids_warn_before_attaching_payment_method(self):
        """ Display a warning before attaching a payment method to a provider.

        :return: A client action with the warning message, if any.
        :rtype: dict
        """
        attached_providers = self.provider_ids.filtered(
            lambda p: p.id.origin not in self._origin.provider_ids.ids
        )
        if attached_providers:
            return {
                'warning': {
                    'title': _("Warning"),
                    'message': _(
                        "Please make sure that %(payment_method)s is supported by %(provider)s.",
                        payment_method=self.name,
                        provider=', '.join(attached_providers.mapped('name'))
                    )
                }
            }

    #=== CRUD METHODS ===#

    def write(self, values):
        # Handle payment methods being archived, detached from providers, or blocking tokenization.
        archiving = values.get('active') is False
        detached_provider_ids = [
            vals[0] for command, *vals in values['provider_ids'] if command == Command.UNLINK
        ] if 'provider_ids' in values else []
        blocking_tokenization = values.get('support_tokenization') is False
        if archiving or detached_provider_ids or blocking_tokenization:
            linked_tokens = self.env['payment.token'].with_context(active_test=True).search(
                expression.AND([
                    [('payment_method_id', 'in', (self + self.brand_ids).ids)],
                    [('provider_id', 'in', detached_provider_ids)] if detached_provider_ids else [],
                ])
            )  # Fix `active_test` in the context forwarded by the view.
            linked_tokens.active = False

        # Prevent enabling a payment method if it is not linked to an enabled provider.
        if values.get('active'):
            for pm in self:
                primary_pm = pm if pm.is_primary else pm.primary_payment_method_id
                if (
                    not primary_pm.active  # Don't bother for already enabled payment methods.
                    and all(p.state == 'disabled' for p in primary_pm.provider_ids)
                ):
                    raise UserError(_(
                        "This payment method needs a partner in crime; you should enable a payment"
                        " provider supporting this method first."
                    ))

        return super().write(values)

    @api.ondelete(at_uninstall=False)
    def _unlink_if_not_default_payment_method(self):
        payment_method_unknown = self.env.ref('payment.payment_method_unknown')
        if payment_method_unknown in self:
            raise UserError(_("You cannot delete the default payment method."))

    # === BUSINESS METHODS === #

    def _get_compatible_payment_methods(
        self, provider_ids, partner_id, currency_id=None, force_tokenization=False,
        is_express_checkout=False, report=None, **kwargs
    ):
        """ Search and return the payment methods matching the compatibility criteria.

        The compatibility criteria are that payment methods must: be supported by at least one of
        the providers; support the country of the partner if it exists; be primary payment methods
        (not a brand). If provided, the optional keyword arguments further refine the criteria.

        :param list provider_ids: The list of providers by which the payment methods must be at
                                  least partially supported to be considered compatible, as a list
                                  of `payment.provider` ids.
        :param int partner_id: The partner making the payment, as a `res.partner` id.
        :param int currency_id: The payment currency, if known beforehand, as a `res.currency` id.
        :param bool force_tokenization: Whether only payment methods supporting tokenization can be
                                        matched.
        :param bool is_express_checkout: Whether the payment is made through express checkout.
        :param dict report: The report in which each provider's availability status and reason must
                            be logged.
        :param dict kwargs: Optional data. This parameter is not used here.
        :return: The compatible payment methods.
        :rtype: payment.method
        """
        # Search compatible payment methods with the base domain.
        payment_methods = self.env['payment.method'].search([('is_primary', '=', True)])
        payment_utils.add_to_report(report, payment_methods)

        # Filter by compatible providers.
        unfiltered_pms = payment_methods
        payment_methods = payment_methods.filtered(
            lambda pm: any(p in provider_ids for p in pm.provider_ids.ids)
        )
        payment_utils.add_to_report(
            report,
            unfiltered_pms - payment_methods,
            available=False,
            reason=REPORT_REASONS_MAPPING['provider_not_available'],
        )

        # Handle the partner country; allow all countries if the list is empty.
        partner = self.env['res.partner'].browse(partner_id)
        if partner.country_id:  # The partner country must either not be set or be supported.
            unfiltered_pms = payment_methods
            payment_methods = payment_methods.filtered(
                lambda pm: (
                    not pm.supported_country_ids
                    or partner.country_id.id in pm.supported_country_ids.ids
                )
            )
            payment_utils.add_to_report(
                report,
                unfiltered_pms - payment_methods,
                available=False,
                reason=REPORT_REASONS_MAPPING['incompatible_country'],
            )

        # Handle the supported currencies; allow all currencies if the list is empty.
        if currency_id:
            unfiltered_pms = payment_methods
            payment_methods = payment_methods.filtered(
                lambda pm: (
                    not pm.supported_currency_ids
                    or currency_id in pm.supported_currency_ids.ids
                )
            )
            payment_utils.add_to_report(
                report,
                unfiltered_pms - payment_methods,
                available=False,
                reason=REPORT_REASONS_MAPPING['incompatible_currency'],
            )

        # Handle tokenization support requirements.
        if force_tokenization:
            unfiltered_pms = payment_methods
            payment_methods = payment_methods.filtered('support_tokenization')
            payment_utils.add_to_report(
                report,
                unfiltered_pms - payment_methods,
                available=False,
                reason=REPORT_REASONS_MAPPING['tokenization_not_supported'],
            )

        # Handle express checkout.
        if is_express_checkout:
            unfiltered_pms = payment_methods
            payment_methods = payment_methods.filtered('support_express_checkout')
            payment_utils.add_to_report(
                report,
                unfiltered_pms - payment_methods,
                available=False,
                reason=REPORT_REASONS_MAPPING['express_checkout_not_supported'],
            )

        return payment_methods

    def _get_from_code(self, code, mapping=None):
        """ Get the payment method corresponding to the given provider-specific code.

        If a mapping is given, the search uses the generic payment method code that corresponds to
        the given provider-specific code.

        :param str code: The provider-specific code of the payment method to get.
        :param dict mapping: A non-exhaustive mapping of generic payment method codes to
                             provider-specific codes.
        :return: The corresponding payment method, if any.
        :type: payment.method
        """
        generic_to_specific_mapping = mapping or {}
        specific_to_generic_mapping = {v: k for k, v in generic_to_specific_mapping.items()}
        return self.search([('code', '=', specific_to_generic_mapping.get(code, code))], limit=1)

```

## File: models\payment_provider.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from odoo import _, api, fields, models
from odoo.exceptions import UserError, ValidationError

from odoo.addons.payment import utils as payment_utils
from odoo.addons.payment.const import REPORT_REASONS_MAPPING

_logger = logging.getLogger(__name__)


class PaymentProvider(models.Model):
    _name = 'payment.provider'
    _description = 'Payment Provider'
    _order = 'module_state, state desc, sequence, name'
    _check_company_auto = True

    def _valid_field_parameter(self, field, name):
        return name == 'required_if_provider' or super()._valid_field_parameter(field, name)

    # Configuration fields
    name = fields.Char(string="Name", required=True, translate=True)
    sequence = fields.Integer(string="Sequence", help="Define the display order")
    code = fields.Selection(
        string="Code",
        help="The technical code of this payment provider.",
        selection=[('none', "No Provider Set")],
        default='none',
        required=True,
    )
    state = fields.Selection(
        string="State",
        help="In test mode, a fake payment is processed through a test payment interface.\n"
             "This mode is advised when setting up the provider.",
        selection=[('disabled', "Disabled"), ('enabled', "Enabled"), ('test', "Test Mode")],
        default='disabled', required=True, copy=False)
    is_published = fields.Boolean(
        string="Published",
        help="Whether the provider is visible on the website or not. Tokens remain functional but "
             "are only visible on manage forms.",
    )
    company_id = fields.Many2one(  # Indexed to speed-up ORM searches (from ir_rule or others)
        string="Company", comodel_name='res.company', default=lambda self: self.env.company.id,
        required=True, index=True)
    main_currency_id = fields.Many2one(
        related='company_id.currency_id',
        help="The main currency of the company, used to display monetary fields.",
    )
    payment_method_ids = fields.Many2many(
        string="Supported Payment Methods", comodel_name='payment.method'
    )
    allow_tokenization = fields.Boolean(
        string="Allow Saving Payment Methods",
        help="This controls whether customers can save their payment methods as payment tokens.\n"
             "A payment token is an anonymous link to the payment method details saved in the\n"
             "provider's database, allowing the customer to reuse it for a next purchase.")
    capture_manually = fields.Boolean(
        string="Capture Amount Manually",
        help="Capture the amount from Odoo, when the delivery is completed.\n"
             "Use this if you want to charge your customers cards only when\n"
             "you are sure you can ship the goods to them.")
    allow_express_checkout = fields.Boolean(
        string="Allow Express Checkout",
        help="This controls whether customers can use express payment methods. Express checkout "
             "enables customers to pay with Google Pay and Apple Pay from which address "
             "information is collected at payment.",
    )
    redirect_form_view_id = fields.Many2one(
        string="Redirect Form Template", comodel_name='ir.ui.view',
        help="The template rendering a form submitted to redirect the user when making a payment",
        domain=[('type', '=', 'qweb')],
        ondelete='restrict',
    )
    inline_form_view_id = fields.Many2one(
        string="Inline Form Template", comodel_name='ir.ui.view',
        help="The template rendering the inline payment form when making a direct payment",
        domain=[('type', '=', 'qweb')],
        ondelete='restrict',
    )
    token_inline_form_view_id = fields.Many2one(
        string="Token Inline Form Template",
        comodel_name='ir.ui.view',
        help="The template rendering the inline payment form when making a payment by token.",
        domain=[('type', '=', 'qweb')],
        ondelete='restrict',
    )
    express_checkout_form_view_id = fields.Many2one(
        string="Express Checkout Form Template",
        comodel_name='ir.ui.view',
        help="The template rendering the express payment methods' form.",
        domain=[('type', '=', 'qweb')],
        ondelete='restrict',
    )

    # Availability fields
    available_country_ids = fields.Many2many(
        string="Countries",
        comodel_name='res.country',
        help="The countries in which this payment provider is available. Leave blank to make it "
             "available in all countries.",
        relation='payment_country_rel',
        column1='payment_id',
        column2='country_id',
    )
    available_currency_ids = fields.Many2many(
        string="Currencies",
        help="The currencies available with this payment provider. Leave empty not to restrict "
             "any.",
        comodel_name='res.currency',
        relation='payment_currency_rel',
        column1="payment_provider_id",
        column2="currency_id",
        compute='_compute_available_currency_ids',
        store=True,
        readonly=False,
        context={'active_test': False},
    )
    maximum_amount = fields.Monetary(
        string="Maximum Amount",
        help="The maximum payment amount that this payment provider is available for. Leave blank "
             "to make it available for any payment amount.",
        currency_field='main_currency_id',
    )

    # Message fields
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
        default=lambda self: _("Your payment has been successfully processed."),
        translate=True)
    cancel_msg = fields.Html(
        string="Cancelled Message",
        help="The message displayed if the order is cancelled during the payment process",
        default=lambda self: _("Your payment has been cancelled."), translate=True)

    # Feature support fields
    support_tokenization = fields.Boolean(
        string="Tokenization", compute='_compute_feature_support_fields'
    )
    support_manual_capture = fields.Selection(
        string="Manual Capture Supported",
        selection=[('full_only', "Full Only"), ('partial', "Partial")],
        compute='_compute_feature_support_fields',
    )
    support_express_checkout = fields.Boolean(
        string="Express Checkout", compute='_compute_feature_support_fields'
    )
    support_refund = fields.Selection(
        string="Refund",
        help="Refund is a feature allowing to refund customers directly from the payment in Odoo.",
        selection=[
            ('none', "Unsupported"),
            ('full_only', "Full Only"),
            ('partial', "Full & Partial"),
        ],
        compute='_compute_feature_support_fields',
    )

    # Kanban view fields
    image_128 = fields.Image(string="Image", max_width=128, max_height=128)
    color = fields.Integer(
        string="Color", help="The color of the card in kanban view", compute='_compute_color',
        store=True)

    # Module-related fields
    module_id = fields.Many2one(string="Corresponding Module", comodel_name='ir.module.module')
    module_state = fields.Selection(string="Installation State", related='module_id.state')
    module_to_buy = fields.Boolean(string="Odoo Enterprise Module", related='module_id.to_buy')

    #=== COMPUTE METHODS ===#

    @api.depends('code')
    def _compute_available_currency_ids(self):
        """ Compute the available currencies based on their support by the providers.

        If the provider does not filter out any currency, the field is left empty for UX reasons.

        :return: None
        """
        all_currencies = self.env['res.currency'].with_context(active_test=False).search([])
        for provider in self:
            supported_currencies = provider._get_supported_currencies()
            if supported_currencies < all_currencies:  # Some currencies have been filtered out.
                provider.available_currency_ids = supported_currencies
            else:
                provider.available_currency_ids = None

    @api.depends('state', 'module_state')
    def _compute_color(self):
        """ Update the color of the kanban card based on the state of the provider.

        :return: None
        """
        for provider in self:
            if provider.module_id and not provider.module_state == 'installed':
                provider.color = 4  # blue
            elif provider.state == 'disabled':
                provider.color = 3  # yellow
            elif provider.state == 'test':
                provider.color = 2  # orange
            elif provider.state == 'enabled':
                provider.color = 7  # green

    @api.depends('code')
    def _compute_feature_support_fields(self):
        """ Compute the feature support fields based on the provider.

        Feature support fields are used to specify which additional features are supported by a
        given provider. These fields are as follows:

        - `support_express_checkout`: Whether the "express checkout" feature is supported. `False`
          by default.
        - `support_manual_capture`: Whether the "manual capture" feature is supported. `False` by
          default.
        - `support_refund`: Which type of the "refunds" feature is supported: `None`,
          `'full_only'`, or `'partial'`. `None` by default.
        - `support_tokenization`: Whether the "tokenization feature" is supported. `False` by
          default.

        For a provider to specify that it supports additional features, it must override this method
        and set the related feature support fields to the desired value on the appropriate
        `payment.provider` records.

        :return: None
        """
        self.update({
            'support_express_checkout': None,
            'support_manual_capture': None,
            'support_tokenization': None,
            'support_refund': 'none',
        })

    #=== ONCHANGE METHODS ===#

    @api.onchange('state')
    def _onchange_state_switch_is_published(self):
        """ Automatically publish or unpublish the provider depending on its state.

        :return: None
        """
        self.is_published = self.state == 'enabled'

    @api.onchange('state')
    def _onchange_state_warn_before_disabling_tokens(self):
        """ Display a warning about the consequences of disabling a provider.

        Let the user know that tokens related to a provider get archived if it is disabled or if its
        state is changed from 'test' to 'enabled', and vice versa.

        :return: A client action with the warning message, if any.
        :rtype: dict
        """
        if self._origin.state in ('test', 'enabled') and self._origin.state != self.state:
            related_tokens = self.env['payment.token'].search(
                [('provider_id', '=', self._origin.id)]
            )
            if related_tokens:
                return {
                    'warning': {
                        'title': _("Warning"),
                        'message': _(
                            "This action will also archive %s tokens that are registered with this "
                            "provider. ", len(related_tokens)
                        )
                    }
                }

    @api.onchange('company_id')
    def _onchange_company_block_if_existing_transactions(self):
        """ Raise a user error when the company is changed and linked transactions exist.

        :return: None
        :raise UserError: If transactions are linked to the provider.
        """
        if self._origin.company_id != self.company_id and self.env['payment.transaction'].search_count(
            [('provider_id', '=', self._origin.id)], limit=1
        ):
            raise UserError(_(
                "You cannot change the company of a payment provider with existing transactions."
            ))

    #=== CRUD METHODS ===#

    @api.model_create_multi
    def create(self, values_list):
        providers = super().create(values_list)
        providers._check_required_if_provider()
        if any(provider.state != 'disabled' for provider in providers):
            self._toggle_post_processing_cron()
        return providers

    def write(self, values):
        # Handle provider state changes.
        deactivated_providers = self.env['payment.provider']
        activated_providers = self.env['payment.provider']
        if 'state' in values:
            state_changed_providers = self.filtered(
                lambda p: p.state not in ('disabled', values['state'])
            )  # Don't handle providers being enabled or whose state is not updated.
            state_changed_providers._archive_linked_tokens()
            if values['state'] == 'disabled':
                deactivated_providers = state_changed_providers
            else:  # 'enabled' or 'test'
                activated_providers = self.filtered(lambda p: p.state == 'disabled')

        result = super().write(values)
        self._check_required_if_provider()

        deactivated_providers._deactivate_unsupported_payment_methods()
        activated_providers._activate_default_pms()
        if activated_providers or deactivated_providers:
            self._toggle_post_processing_cron()

        return result

    def _check_required_if_provider(self):
        """ Check that provider-specific required fields have been filled.

        The fields that have the `required_if_provider='<provider_code>'` attribute are made
        required for all `payment.provider` records with the `code` field equal to `<provider_code>`
        and with the `state` field equal to `'enabled'` or `'test'`.

        Provider-specific views should make the form fields required under the same conditions.

        :return: None
        :raise ValidationError: If a provider-specific required field is empty.
        """
        field_names = []
        enabled_providers = self.filtered(lambda p: p.state in ['enabled', 'test'])
        for field_name, field in self._fields.items():
            required_for_provider_code = getattr(field, 'required_if_provider', None)
            if required_for_provider_code and any(
                required_for_provider_code == provider._get_code() and not provider[field_name]
                for provider in enabled_providers
            ):
                ir_field = self.env['ir.model.fields']._get(self._name, field_name)
                field_names.append(ir_field.field_description)
        if field_names:
            raise ValidationError(
                _("The following fields must be filled: %s", ", ".join(field_names))
            )

    @api.model
    def _toggle_post_processing_cron(self):
        """ Enable the post-processing cron if some providers are enabled; disable it otherwise.

        This allows for saving resources on the cron's wake-up overhead when it has nothing to do.

        :return: None
        """
        post_processing_cron = self.env.ref(
            'payment.cron_post_process_payment_tx', raise_if_not_found=False
        )
        if post_processing_cron:
            any_active_provider = bool(
                self.sudo().search_count([('state', '!=', 'disabled')], limit=1)
            )
            post_processing_cron.active = any_active_provider

    def _archive_linked_tokens(self):
        """ Archive all the payment tokens linked to the providers.

        :return: None
        """
        self.env['payment.token'].search([('provider_id', 'in', self.ids)]).write({'active': False})

    def _deactivate_unsupported_payment_methods(self):
        """ Deactivate payment methods linked to only disabled providers.

        :return: None
        """
        unsupported_pms = self.payment_method_ids.filtered(
            lambda pm: all(p.state == 'disabled' for p in pm.provider_ids)
        )
        (unsupported_pms + unsupported_pms.brand_ids).active = False

    def _activate_default_pms(self):
        """ Activate the default payment methods of the provider.

        :return: None
        """
        for provider in self:
            pm_codes = provider._get_default_payment_method_codes()
            pms = provider.with_context(active_test=False).payment_method_ids
            (pms + pms.brand_ids).filtered(lambda pm: pm.code in pm_codes).active = True

    @api.ondelete(at_uninstall=False)
    def _unlink_except_master_data(self):
        """ Prevent the deletion of the payment provider if it has an xmlid. """
        external_ids = self.get_external_id()
        for provider in self:
            external_id = external_ids[provider.id]
            if external_id and not external_id.startswith('__export__'):
                raise UserError(_(
                    "You cannot delete the payment provider %s; disable it or uninstall it"
                    " instead.", provider.name
                ))

    #=== ACTION METHODS ===#

    def button_immediate_install(self):
        """ Install the module and reload the page.

        Note: `self.ensure_one()`

        :return: The action to reload the page.
        :rtype: dict
        """
        if self.module_id and self.module_state != 'installed':
            self.module_id.button_immediate_install()
            return {
                'type': 'ir.actions.client',
                'tag': 'reload',
            }

    def action_toggle_is_published(self):
        """ Toggle the field `is_published`.

        :return: None
        :raise UserError: If the provider is disabled.
        """
        if self.state != 'disabled':
            self.is_published = not self.is_published
        else:
            raise UserError(_("You cannot publish a disabled provider."))

    def action_view_payment_methods(self):
        self.ensure_one()
        return {
            'type': 'ir.actions.act_window',
            'name': _("Payment Methods"),
            'res_model': 'payment.method',
            'view_mode': 'list,kanban,form',
            'domain': [('id', 'in', self.with_context(active_test=False).payment_method_ids.ids)],
            'context': {'active_test': False, 'create': False},
        }

    #=== BUSINESS METHODS ===#

    @api.model
    def _get_compatible_providers(
        self, company_id, partner_id, amount, currency_id=None, force_tokenization=False,
        is_express_checkout=False, is_validation=False, report=None, **kwargs
    ):
        """ Search and return the providers matching the compatibility criteria.

        The compatibility criteria are that providers must: not be disabled; be in the company that
        is provided; support the country of the partner if it exists; be compatible with the
        currency if provided. If provided, the optional keyword arguments further refine the
        criteria.

        :param int company_id: The company to which providers must belong, as a `res.company` id.
        :param int partner_id: The partner making the payment, as a `res.partner` id.
        :param float amount: The amount to pay. `0` for validation transactions.
        :param int currency_id: The payment currency, if known beforehand, as a `res.currency` id.
        :param bool force_tokenization: Whether only providers allowing tokenization can be matched.
        :param bool is_express_checkout: Whether the payment is made through express checkout.
        :param bool is_validation: Whether the operation is a validation.
        :param dict report: The report in which each provider's availability status and reason must
                            be logged.
        :param dict kwargs: Optional data. This parameter is not used here.
        :return: The compatible providers.
        :rtype: payment.provider
        """
        # Search compatible providers with the base domain.
        providers = self.env['payment.provider'].search([
            *self.env['payment.provider']._check_company_domain(company_id),
            ('state', 'in', ['enabled', 'test']),
        ])
        payment_utils.add_to_report(report, providers)

        # Filter by `is_published` state.
        if not self.env.user._is_internal():
            providers = providers.filtered('is_published')

        # Handle the partner country; allow all countries if the list is empty.
        partner = self.env['res.partner'].browse(partner_id)
        if partner.country_id:  # The partner country must either not be set or be supported.
            unfiltered_providers = providers
            providers = providers.filtered(
                lambda p: (
                    not p.available_country_ids
                    or partner.country_id.id in p.available_country_ids.ids
                )
            )
            payment_utils.add_to_report(
                report,
                unfiltered_providers - providers,
                available=False,
                reason=REPORT_REASONS_MAPPING['incompatible_country'],
            )

        # Handle the maximum amount.
        currency = self.env['res.currency'].browse(currency_id).exists()
        if not is_validation and currency:  # The currency is required to convert the amount.
            company = self.env['res.company'].browse(company_id).exists()
            date = fields.Date.context_today(self)
            converted_amount = currency._convert(amount, company.currency_id, company, date)
            unfiltered_providers = providers
            providers = providers.filtered(
                lambda p: (
                    not p.maximum_amount
                    or currency.compare_amounts(p.maximum_amount, converted_amount) != -1
                )
            )
            payment_utils.add_to_report(
                report,
                unfiltered_providers - providers,
                available=False,
                reason=REPORT_REASONS_MAPPING['exceed_max_amount'],
            )

        # Handle the available currencies; allow all currencies if the list is empty.
        if currency:
            unfiltered_providers = providers
            providers = providers.filtered(
                lambda p: (
                    not p.available_currency_ids
                    or currency.id in p.available_currency_ids.ids
                )
            )
            payment_utils.add_to_report(
                report,
                unfiltered_providers - providers,
                available=False,
                reason=REPORT_REASONS_MAPPING['incompatible_currency'],
            )

        # Handle tokenization support requirements.
        if force_tokenization or self._is_tokenization_required(**kwargs):
            unfiltered_providers = providers
            providers = providers.filtered('allow_tokenization')
            payment_utils.add_to_report(
                report,
                unfiltered_providers - providers,
                available=False,
                reason=REPORT_REASONS_MAPPING['tokenization_not_supported'],
            )

        # Handle express checkout.
        if is_express_checkout:
            unfiltered_providers = providers
            providers = providers.filtered('allow_express_checkout')
            payment_utils.add_to_report(
                report,
                unfiltered_providers - providers,
                available=False,
                reason=REPORT_REASONS_MAPPING['express_checkout_not_supported'],
            )

        return providers

    def _get_supported_currencies(self):
        """ Return the supported currencies for the payment provider.

        By default, all currencies are considered supported, including the inactive ones. For a
        provider to filter out specific currencies, it must override this method and return the
        subset of supported currencies.

        Note: `self.ensure_one()`

        :return: The supported currencies.
        :rtype: res.currency
        """
        self.ensure_one()
        return self.env['res.currency'].with_context(active_test=False).search([])

    def _is_tokenization_required(self, **kwargs):
        """ Return whether tokenizing the transaction is required given its context.

        For a module to make the tokenization required based on the payment context, it must
        override this method and return whether it is required.

        :param dict kwargs: The payment context. This parameter is not used here.
        :return: Whether tokenizing the transaction is required.
        :rtype: bool
        """
        return False

    def _should_build_inline_form(self, is_validation=False):
        """ Return whether the inline payment form should be instantiated.

        For a provider to handle both direct payments and payments with redirection, it must
        override this method and return whether the inline payment form should be instantiated (i.e.
        if the payment should be direct) based on the operation (online payment or validation).

        :param bool is_validation: Whether the operation is a validation.
        :return: Whether the inline form should be instantiated.
        :rtype: bool
        """
        return True

    def _get_validation_amount(self):
        """ Return the amount to use for validation operations.

        For a provider to support tokenization, it must override this method and return the
        validation amount. If it is `0`, it is not necessary to create the override.

        Note: `self.ensure_one()`

        :return: The validation amount.
        :rtype: float
        """
        self.ensure_one()
        return 0.0

    def _get_validation_currency(self):
        """ Return the currency to use for validation operations.

        The validation currency must be supported by both the provider and the payment method. If
        the payment method is not passed, only the provider's supported currencies are considered.
        If no suitable currency is found, the provider's company's currency is returned instead.

        For a provider to support tokenization and specify a different validation currency, it must
        override this method and return the appropriate validation currency.

        Note: `self.ensure_one()`

        :return: The validation currency.
        :rtype: recordset of `res.currency`
        """
        self.ensure_one()

        # Find the validation currency at the intersection of the provider's and payment method's
        # supported currencies. An empty recordset means that all currencies are supported.
        provider_currencies = self.available_currency_ids
        pm = self.env.context.get('validation_pm')
        pm_currencies = self.env['res.currency'] if not pm else pm.supported_currency_ids
        validation_currency = None
        if provider_currencies and pm_currencies:
            validation_currency = (provider_currencies & pm_currencies)[:1]
        elif provider_currencies and not pm_currencies:
            validation_currency = provider_currencies[:1]
        elif not provider_currencies and pm_currencies:
            validation_currency = pm_currencies[:1]
        if not validation_currency:  # All currencies are supported, or no suitable one was found.
            validation_currency = self.company_id.currency_id
        return validation_currency

    def _get_redirect_form_view(self, is_validation=False):
        """ Return the view of the template used to render the redirect form.

        For a provider to return a different view depending on whether the operation is a
        validation, it must override this method and return the appropriate view.

        Note: `self.ensure_one()`

        :param bool is_validation: Whether the operation is a validation.
        :return: The view of the redirect form template.
        :rtype: record of `ir.ui.view`
        """
        self.ensure_one()
        return self.redirect_form_view_id

    @api.model
    def _setup_provider(self, provider_code):
        """ Perform module-specific setup steps for the provider.

        This method is called after the module of a provider is installed, with its code passed as
        `provider_code`.

        :param str provider_code: The code of the provider to setup.
        :return: None
        """
        return

    @api.model
    def _get_removal_domain(self, provider_code, **kwargs):
        return [('code', '=', provider_code)]

    @api.model
    def _remove_provider(self, provider_code, **kwargs):
        """ Remove the module-specific data of the given provider.

        :param str provider_code: The code of the provider whose data to remove.
        :return: None
        """
        providers = self.search(self._get_removal_domain(provider_code, **kwargs))
        providers.write(self._get_removal_values())

    def _get_removal_values(self):
        """ Return the values to update a provider with when its module is uninstalled.

        For a module to specify additional removal values, it must override this method and complete
        the generic values with its specific values.

        :return: The removal values to update the removed provider with.
        :rtype: dict
        """
        return {
            'code': 'none',
            'state': 'disabled',
            'is_published': False,
            'redirect_form_view_id': None,
            'inline_form_view_id': None,
            'token_inline_form_view_id': None,
            'express_checkout_form_view_id': None,
        }

    def _get_provider_name(self):
        """ Return the translated name of the provider.

        Note: self.ensure_one()

        :return: The translated name of the provider.
        :rtype: str
        """
        self.ensure_one()
        return dict(self._fields['code']._description_selection(self.env))[self.code]

    def _get_code(self):
        """ Return the code of the provider.

        Note: self.ensure_one()

        :return: The code of the provider.
        :rtype: str
        """
        self.ensure_one()
        return self.code

    def _get_default_payment_method_codes(self):
        """ Return the default payment methods for this provider.

        Note: self.ensure_one()

        :return: The default payment method codes.
        :rtype: set
        """
        self.ensure_one()
        return set()

```

## File: models\payment_token.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from odoo import _, api, fields, models
from odoo.exceptions import UserError, ValidationError


class PaymentToken(models.Model):
    _name = 'payment.token'
    _order = 'partner_id, id desc'
    _description = 'Payment Token'
    _check_company_auto = True

    provider_id = fields.Many2one(string="Provider", comodel_name='payment.provider', required=True)
    provider_code = fields.Selection(string="Provider Code", related='provider_id.code')
    company_id = fields.Many2one(
        related='provider_id.company_id', store=True, index=True
    )  # Indexed to speed-up ORM searches (from ir_rule or others).
    payment_method_id = fields.Many2one(
        string="Payment Method", comodel_name='payment.method', readonly=True, required=True
    )
    payment_method_code = fields.Char(
        string="Payment Method Code", related='payment_method_id.code'
    )
    payment_details = fields.Char(
        string="Payment Details", help="The clear part of the payment method's payment details.",
    )
    partner_id = fields.Many2one(string="Partner", comodel_name='res.partner', required=True)
    provider_ref = fields.Char(
        string="Provider Reference",
        help="The provider reference of the token of the transaction.",
        required=True,
    )  # This is not the same thing as the provider reference of the transaction.
    transaction_ids = fields.One2many(
        string="Payment Transactions", comodel_name='payment.transaction', inverse_name='token_id'
    )
    active = fields.Boolean(string="Active", default=True)

    #=== COMPUTE METHODS ===#

    @api.depends('payment_details', 'create_date')
    def _compute_display_name(self):
        for token in self:
            token.display_name = token._build_display_name()

    #=== CRUD METHODS ===#

    @api.model_create_multi
    def create(self, values_list):
        for values in values_list:
            if 'provider_id' in values:
                provider = self.env['payment.provider'].browse(values['provider_id'])

                # Include provider-specific create values
                values.update(self._get_specific_create_values(provider.code, values))
            else:
                pass  # Let psycopg warn about the missing required field.

        return super().create(values_list)

    @api.model
    def _get_specific_create_values(self, provider_code, values):
        """ Complete the values of the `create` method with provider-specific values.

        For a provider to add its own create values, it must overwrite this method and return a
        dict of values. Provider-specific values take precedence over those of the dict of generic
        create values.

        :param str provider_code: The code of the provider managing the token.
        :param dict values: The original create values.
        :return: The dict of provider-specific create values.
        :rtype: dict
        """
        return dict()

    def write(self, values):
        """ Prevent unarchiving tokens and handle their archiving.

        :return: The result of the call to the parent method.
        :rtype: bool
        :raise UserError: If at least one token is being unarchived.
        """
        if 'active' in values:
            if values['active']:
                if any(
                    not token.payment_method_id.active
                    or token.provider_id.state == 'disabled'
                    for token in self
                ):
                    raise UserError(_(
                        "You can't unarchive tokens linked to inactive payment methods or disabled"
                        " providers."
                    ))
            else:
                # Call the handlers in sudo mode because this method might have been called by RPC.
                self.filtered('active').sudo()._handle_archiving()

        return super().write(values)

    @api.constrains('partner_id')
    def _check_partner_is_never_public(self):
        """ Check that the partner associated with the token is never public. """
        for token in self:
            if token.partner_id.is_public:
                raise ValidationError(_("No token can be assigned to the public partner."))

    def _handle_archiving(self):
        """ Handle the archiving of tokens.

        For a module to perform additional operations when a token is archived, it must override
        this method.

        :return: None
        """
        return

    #=== BUSINESS METHODS ===#

    def _get_available_tokens(self, providers_ids, partner_id, is_validation=False, **kwargs):
        """ Return the available tokens linked to the given providers and partner.

        For a module to retrieve the available tokens, it must override this method and add
        information in the kwargs to define the context of the request.

        :param list providers_ids: The ids of the providers available for the transaction.
        :param int partner_id: The id of the partner.
        :param bool is_validation: Whether the transaction is a validation operation.
        :param dict kwargs: Locally unused keywords arguments.
        :return: The available tokens.
        :rtype: payment.token
        """
        if not is_validation:
            return self.env['payment.token'].search(
                [('provider_id', 'in', providers_ids), ('partner_id', '=', partner_id)]
            )
        else:
            # Get all the tokens of the partner and of their commercial partner, regardless of
            # whether the providers are available.
            partner = self.env['res.partner'].browse(partner_id)
            return self.env['payment.token'].search(
                [('partner_id', 'in', [partner.id, partner.commercial_partner_id.id])]
            )

    def _build_display_name(self, *args, max_length=34, should_pad=True, **kwargs):
        """ Build a token name of the desired maximum length with the format `•••• 1234`.

        The payment details are padded on the left with up to four padding characters. The padding
        is only added if there is enough room for it. If not, it is either reduced or not added at
        all. If there is not enough room for the payment details either, they are trimmed from the
        left.

        For a module to customize the display name of a token, it must override this method and
        return the customized display name.

        Note: `self.ensure_one()`

        :param list args: The arguments passed by QWeb when calling this method.
        :param int max_length: The desired maximum length of the token name. The default is `34` to
                               fit the largest IBANs.
        :param bool should_pad: Whether the token should be padded.
        :param dict kwargs: Optional data used in overrides of this method.
        :return: The padded token name.
        :rtype: str
        """
        self.ensure_one()

        if not self.create_date:
            return ''

        padding_length = max_length - len(self.payment_details or '')
        if not self.payment_details:
            create_date_str = self.create_date.strftime('%Y/%m/%d')
            display_name = _("Payment details saved on %(date)s", date=create_date_str)
        elif padding_length >= 2:  # Enough room for padding.
            padding = '•' * min(padding_length - 1, 4) + ' ' if should_pad else ''
            display_name = ''.join([padding, self.payment_details])
        elif padding_length > 0:  # Not enough room for padding.
            display_name = self.payment_details
        else:  # Not enough room for neither padding nor the payment details.
            display_name = self.payment_details[-max_length:] if max_length > 0 else ''
        return display_name

    def get_linked_records_info(self):
        """ Return a list of information about records linked to the current token.

        For a module to implement payments and link documents to a token, it must override this
        method and add information about linked document records to the returned list.

        The information must be structured as a dict with the following keys:

        - `description`: The description of the record's model (e.g. "Subscription").
        - `id`: The id of the record.
        - `name`: The name of the record.
        - `url`: The url to access the record.

        Note: `self.ensure_one()`

        :return: The list of information about the linked document records.
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
from markupsafe import Markup

from odoo import _, api, fields, models
from odoo.exceptions import UserError, ValidationError
from odoo.tools import email_normalize_all, format_amount

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

    provider_id = fields.Many2one(
        string="Provider", comodel_name='payment.provider', readonly=True, required=True
    )
    provider_code = fields.Selection(string="Provider Code", related='provider_id.code')
    company_id = fields.Many2one(  # Indexed to speed-up ORM searches (from ir_rule or others)
        related='provider_id.company_id', store=True, index=True
    )
    payment_method_id = fields.Many2one(
        string="Payment Method", comodel_name='payment.method', readonly=True, required=True
    )
    payment_method_code = fields.Char(
        string="Payment Method Code", related='payment_method_id.code'
    )
    reference = fields.Char(
        string="Reference", help="The internal reference of the transaction", readonly=True,
        required=True)  # Already has an index from the UNIQUE SQL constraint.
    provider_reference = fields.Char(
        string="Provider Reference", help="The provider reference of the transaction",
        readonly=True)  # This is not the same thing as the provider reference of the token.
    amount = fields.Monetary(
        string="Amount", currency_field='currency_id', readonly=True, required=True)
    currency_id = fields.Many2one(
        string="Currency", comodel_name='res.currency', readonly=True, required=True)
    token_id = fields.Many2one(
        string="Payment Token", comodel_name='payment.token', readonly=True,
        domain='[("provider_id", "=", "provider_id")]', ondelete='restrict')
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

    # Fields used for traceability.
    operation = fields.Selection(  # This should not be trusted if the state is draft or pending.
        string="Operation",
        selection=[
            ('online_redirect', "Online payment with redirection"),
            ('online_direct', "Online direct payment"),
            ('online_token', "Online payment by token"),
            ('validation', "Validation of the payment method"),
            ('offline', "Offline payment by token"),
            ('refund', "Refund"),
        ],
        readonly=True,
        index=True,
    )
    source_transaction_id = fields.Many2one(
        string="Source Transaction",
        comodel_name='payment.transaction',
        help="The source transaction of the related child transactions",
        readonly=True,
    )
    child_transaction_ids = fields.One2many(
        string="Child Transactions",
        help="The child transactions of the transaction.",
        comodel_name='payment.transaction',
        inverse_name='source_transaction_id',
        readonly=True,
    )
    refunds_count = fields.Integer(string="Refunds Count", compute='_compute_refunds_count')

    # Fields used for user redirection & payment post-processing
    is_post_processed = fields.Boolean(
        string="Is Post-processed", help="Has the payment been post-processed")
    tokenize = fields.Boolean(
        string="Create Token",
        help="Whether a payment token should be created when post-processing the transaction")
    landing_route = fields.Char(
        string="Landing Route",
        help="The route the user is redirected to after the transaction")

    # Duplicated partner values allowing to keep a record of them, should they be later updated.
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

    def _compute_refunds_count(self):
        rg_data = self.env['payment.transaction']._read_group(
            domain=[('source_transaction_id', 'in', self.ids), ('operation', '=', 'refund')],
            groupby=['source_transaction_id'],
            aggregates=['__count'],
        )
        data = {source_transaction.id: count for source_transaction, count in rg_data}
        for record in self:
            record.refunds_count = data.get(record.id, 0)

    #=== CONSTRAINT METHODS ===#

    @api.constrains('state')
    def _check_state_authorized_supported(self):
        """ Check that authorization is supported for a transaction in the `authorized` state. """
        illegal_authorize_state_txs = self.filtered(
            lambda tx: tx.state == 'authorized' and not tx.provider_id.support_manual_capture
        )
        if illegal_authorize_state_txs:
            raise ValidationError(_(
                "Transaction authorization is not supported by the following payment providers: %s",
                ', '.join(set(illegal_authorize_state_txs.mapped('provider_id.name')))
            ))

    @api.constrains('token_id')
    def _check_token_is_active(self):
        """ Check that the token used to create the transaction is active. """
        if self.token_id and not self.token_id.active:
            raise ValidationError(_("Creating a transaction from an archived token is forbidden."))

    #=== CRUD METHODS ===#

    @api.model_create_multi
    def create(self, values_list):
        for values in values_list:
            provider = self.env['payment.provider'].browse(values['provider_id'])

            if not values.get('reference'):
                values['reference'] = self._compute_reference(provider.code, **values)

            # Duplicate partner values.
            partner = self.env['res.partner'].browse(values['partner_id'])
            partner_emails = email_normalize_all(partner.email)
            values.update({
                # Use the parent partner as fallback if the invoicing address has no name.
                'partner_name': partner.name or partner.parent_id.name,
                'partner_lang': partner.lang,
                'partner_email': partner_emails[0] if partner_emails else None,
                'partner_address': payment_utils.format_partner_address(
                    partner.street, partner.street2
                ),
                'partner_zip': partner.zip,
                'partner_city': partner.city,
                'partner_state_id': partner.state_id.id,
                'partner_country_id': partner.country_id.id,
                'partner_phone': partner.phone,
            })

            # Include provider-specific create values
            values.update(self._get_specific_create_values(provider.code, values))

        txs = super().create(values_list)

        # Monetary fields are rounded with the currency at creation time by the ORM. Sometimes, this
        # can lead to inconsistent string representation of the amounts sent to the providers.
        # E.g., tx.create(amount=1111.11) -> tx.amount == 1111.1100000000001
        # To ensure a proper string representation, we invalidate this request's cache values of the
        # `amount` field for the created transactions. This forces the ORM to read the values from
        # the DB where there were stored using `float_repr`, which produces a result consistent with
        # the format expected by providers.
        # E.g., tx.create(amount=1111.11) ; tx.invalidate_recordset() -> tx.amount == 1111.11
        txs.invalidate_recordset(['amount'])

        return txs

    @api.model
    def _get_specific_create_values(self, provider_code, values):
        """ Complete the values of the `create` method with provider-specific values.

        For a provider to add its own create values, it must overwrite this method and return a dict
        of values. Provider-specific values take precedence over those of the dict of generic create
        values.

        :param str provider_code: The code of the provider that handled the transaction.
        :param dict values: The original create values.
        :return: The dict of provider-specific create values.
        :rtype: dict
        """
        return dict()

    #=== ACTION METHODS ===#

    def action_view_refunds(self):
        """ Return the windows action to browse the refund transactions linked to the transaction.

        Note: `self.ensure_one()`

        :return: The window action to browse the refund transactions.
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
            action['view_mode'] = 'list,form'
            action['domain'] = [('source_transaction_id', '=', self.id)]
        return action

    def action_capture(self):
        """ Open the partial capture wizard if it is supported by the related providers, otherwise
        capture the transactions immediately.

        :return: The action to open the partial capture wizard, if supported.
        :rtype: action.act_window|None
        """
        payment_utils.check_rights_on_recordset(self)

        if any(tx.provider_id.sudo().support_manual_capture == 'partial' for tx in self):
            return {
                'name': _("Capture"),
                'type': 'ir.actions.act_window',
                'view_mode': 'form',
                'res_model': 'payment.capture.wizard',
                'target': 'new',
                'context': {
                    'active_model': 'payment.transaction',
                    # Consider also confirmed transactions to calculate the total authorized amount.
                    'active_ids': self.filtered(lambda tx: tx.state in ['authorized', 'done']).ids,
                },
            }
        else:
            for tx in self.filtered(lambda tx: tx.state == 'authorized'):
                # In sudo mode because we need to be able to read on provider fields.
                tx.sudo()._send_capture_request()

    def action_void(self):
        """ Check the state of the transaction and request to have them voided. """
        payment_utils.check_rights_on_recordset(self)

        if any(tx.state != 'authorized' for tx in self):
            raise ValidationError(_("Only authorized transactions can be voided."))

        for tx in self:
            # Consider all the confirmed partial capture (same operation as parent) child txs.
            captured_amount = sum(child_tx.amount for child_tx in tx.child_transaction_ids.filtered(
                lambda t: t.state == 'done' and t.operation == tx.operation
            ))
            # In sudo mode because we need to be able to read on provider fields.
            tx.sudo()._send_void_request(amount_to_void=tx.amount - captured_amount)

    def action_refund(self, amount_to_refund=None):
        """ Check the state of the transactions and request their refund.

        :param float amount_to_refund: The amount to be refunded.
        :return: None
        """
        if any(tx.state != 'done' for tx in self):
            raise ValidationError(_("Only confirmed transactions can be refunded."))

        payment_utils.check_rights_on_recordset(self)
        for tx in self:
            # In sudo mode because we need to be able to read on provider fields.
            tx.sudo()._send_refund_request(amount_to_refund=amount_to_refund)

    #=== BUSINESS METHODS - PAYMENT FLOW ===#

    @api.model
    def _compute_reference(self, provider_code, prefix=None, separator='-', **kwargs):
        """ Compute a unique reference for the transaction.

        The reference corresponds to the prefix if no other transaction with that prefix already
        exists. Otherwise, it follows the pattern `{computed_prefix}{separator}{sequence_number}`
        where:

        - `{computed_prefix}` is:

          - The provided custom prefix, if any.
          - The computation result of :meth:`_compute_reference_prefix` if the custom prefix is not
            filled, but the kwargs are.
          - `'tx-{datetime}'` if neither the custom prefix nor the kwargs are filled.

        - `{separator}` is the string that separates the prefix from the sequence number.
        - `{sequence_number}` is the next integer in the sequence of references sharing the same
          prefix. The sequence starts with `1` if there is only one matching reference.

        .. example::

           - Given the custom prefix `'example'` which has no match with an existing reference, the
             full reference will be `'example'`.
           - Given the custom prefix `'example'` which matches the existing reference `'example'`,
             and the custom separator `'-'`, the full reference will be `'example-1'`.
           - Given the kwargs `{'invoice_ids': [1, 2]}`, the custom separator `'-'` and no custom
             prefix, the full reference will be `'INV1-INV2'` (or similar) if no existing reference
             has the same prefix, or `'INV1-INV2-n'` if `n` existing references have the same
             prefix.

        :param str provider_code: The code of the provider handling the transaction.
        :param str prefix: The custom prefix used to compute the full reference.
        :param str separator: The custom separator used to separate the prefix from the suffix.
        :param dict kwargs: Optional values passed to :meth:`_compute_reference_prefix` if no custom
                            prefix is provided.
        :return: The unique reference for the transaction.
        :rtype: str
        """
        # Compute the prefix.
        if prefix:
            # Replace special characters by their ASCII alternative (é -> e ; ä -> a ; ...)
            prefix = unicodedata.normalize('NFKD', prefix).encode('ascii', 'ignore').decode('utf-8')
        if not prefix:  # Prefix not provided or voided above, compute it based on the kwargs.
            prefix = self.sudo()._compute_reference_prefix(provider_code, separator, **kwargs)
        if not prefix:  # Prefix not computed from the kwargs, fallback on time-based value
            prefix = payment_utils.singularize_reference_prefix()

        # Compute the sequence number.
        reference = prefix  # The first reference of a sequence has no sequence number.
        if self.sudo().search_count([('reference', '=', prefix)], limit=1):  # The reference already has a match
            # We now execute a second search on `payment.transaction` to fetch all the references
            # starting with the given prefix. The load of these two searches is mitigated by the
            # index on `reference`. Although not ideal, this solution allows for quickly knowing
            # whether the sequence for a given prefix is already started or not, usually not. An SQL
            # query wouldn't help either as the selector is arbitrary and doing that would be an
            # open-door to SQL injections.
            same_prefix_references = self.sudo().search(
                [('reference', '=like', f'{prefix}{separator}%')]
            ).with_context(prefetch_fields=False).mapped('reference')

            # A final regex search is necessary to figure out the next sequence number. The previous
            # search could not rely on alphabetically sorting the reference to infer the largest
            # sequence number because both the prefix and the separator are arbitrary. A given
            # prefix could happen to be a substring of the reference from a different sequence.
            # For instance, the prefix 'example' is a valid match for the existing references
            # 'example', 'example-1' and 'example-ref', in that order. Trusting the order to infer
            # the sequence number would lead to a collision with 'example-1'.
            search_pattern = re.compile(rf'^{re.escape(prefix)}{separator}(\d+)$')
            max_sequence_number = 0  # If no match is found, start the sequence with this reference.
            for existing_reference in same_prefix_references:
                search_result = re.search(search_pattern, existing_reference)
                if search_result:  # The reference has the same prefix and is from the same sequence
                    # Find the largest sequence number, if any.
                    current_sequence = int(search_result.group(1))
                    if current_sequence > max_sequence_number:
                        max_sequence_number = current_sequence

            # Compute the full reference.
            reference = f'{prefix}{separator}{max_sequence_number + 1}'
        return reference

    @api.model
    def _compute_reference_prefix(self, provider_code, separator, **values):
        """ Compute the reference prefix from the transaction values.

        Note: This method should be called in sudo mode to give access to the documents (invoices,
        sales orders) referenced in the transaction values.

        :param str provider_code: The code of the provider handling the transaction.
        :param str separator: The custom separator used to separate parts of the computed
                              reference prefix.
        :param dict values: The transaction values used to compute the reference prefix.
        :return: The computed reference prefix.
        :rtype: str
        """
        return ''

    def _get_processing_values(self):
        """ Return the values used to process the transaction.

        The values are returned as a dict containing entries with the following keys:

        - `provider_id`: The provider handling the transaction, as a `payment.provider` id.
        - `provider_code`: The code of the provider.
        - `reference`: The reference of the transaction.
        - `amount`: The rounded amount of the transaction.
        - `currency_id`: The currency of the transaction, as a `res.currency` id.
        - `partner_id`: The partner making the transaction, as a `res.partner` id.
        - `should_tokenize`: Whether this transaction should be tokenized.
        - Additional provider-specific entries.

        Note: `self.ensure_one()`

        :return: The processing values.
        :rtype: dict
        """
        self.ensure_one()

        processing_values = {
            'provider_id': self.provider_id.id,
            'provider_code': self.provider_code,
            'reference': self.reference,
            'amount': self.amount,
            'currency_id': self.currency_id.id,
            'partner_id': self.partner_id.id,
            'should_tokenize': self.tokenize,
        }

        # Complete generic processing values with provider-specific values.
        processing_values.update(self._get_specific_processing_values(processing_values))
        _logger.info(
            "generic and provider-specific processing values for transaction with reference "
            "%(ref)s:\n%(values)s",
            {'ref': self.reference, 'values': pprint.pformat(processing_values)},
        )

        # Render the html form for the redirect flow if available.
        if self.operation in ('online_redirect', 'validation'):
            redirect_form_view = self.provider_id._get_redirect_form_view(
                is_validation=self.operation == 'validation'
            )
            if redirect_form_view:  # Some provider don't need a redirect form.
                rendering_values = self._get_specific_rendering_values(processing_values)
                _logger.info(
                    "provider-specific rendering values for transaction with reference "
                    "%(ref)s:\n%(values)s",
                    {'ref': self.reference, 'values': pprint.pformat(rendering_values)},
                )
                redirect_form_html = self.env['ir.qweb']._render(redirect_form_view.id, rendering_values)
                processing_values.update(redirect_form_html=redirect_form_html)

        return processing_values

    def _get_specific_processing_values(self, processing_values):
        """ Return a dict of provider-specific values used to process the transaction.

        For a provider to add its own processing values, it must overwrite this method and return a
        dict of provider-specific values based on the generic values returned by this method.
        Provider-specific values take precedence over those of the dict of generic processing
        values.

        :param dict processing_values: The generic processing values of the transaction.
        :return: The dict of provider-specific processing values.
        :rtype: dict
        """
        return dict()

    def _get_specific_rendering_values(self, processing_values):
        """ Return a dict of provider-specific values used to render the redirect form.

        For a provider to add its own rendering values, it must overwrite this method and return a
        dict of provider-specific values based on the processing values (provider-specific
        processing values included).

        :param dict processing_values: The processing values of the transaction.
        :return: The dict of provider-specific rendering values.
        :rtype: dict
        """
        return dict()

    def _get_mandate_values(self):
        """ Return a dict of module-specific values used to create a mandate.

        For a module to add its own mandate values, it must overwrite this method and return a dict
        of module-specific values.

        Note: `self.ensure_one()`

        :return: The dict of module-specific mandate values.
        :rtype: dict
        """
        self.ensure_one()
        return dict()

    def _send_payment_request(self):
        """ Request the provider handling the transaction to make the payment.

        This method is exclusively used to make payments by token, which correspond to both the
        `online_token` and the `offline` transaction's `operation` field.

        For a provider to support tokenization, it must override this method and make an API request
        to make a payment.

        Note: `self.ensure_one()`

        :return: None
        """
        self.ensure_one()
        self._ensure_provider_is_not_disabled()
        self._log_sent_message()

    def _send_refund_request(self, amount_to_refund=None):
        """ Request the provider handling the transaction to refund it.

        For a provider to support refunds, it must override this method and make an API request to
        make a refund.

        Note: `self.ensure_one()`

        :param float amount_to_refund: The amount to be refunded.
        :return: The refund transaction created to process the refund request.
        :rtype: recordset of `payment.transaction`
        """
        self.ensure_one()
        self._ensure_provider_is_not_disabled()

        refund_tx = self._create_child_transaction(amount_to_refund or self.amount, is_refund=True)
        refund_tx._log_sent_message()
        return refund_tx

    def _send_capture_request(self, amount_to_capture=None):
        """ Request the provider handling the transaction to capture the payment.

        For partial captures, create a child transaction linked to the source transaction.

        For a provider to support authorization, it must override this method and make an API
        request to capture the payment.

        Note: `self.ensure_one()`

        :param float amount_to_capture: The amount to capture.
        :return: The created capture child transaction, if any.
        :rtype: `payment.transaction`
        """
        self.ensure_one()
        self._ensure_provider_is_not_disabled()

        if amount_to_capture and amount_to_capture != self.amount:
            return self._create_child_transaction(amount_to_capture)
        return self.env['payment.transaction']

    def _send_void_request(self, amount_to_void=None):
        """ Request the provider handling the transaction to void the payment.

        For partial voids, create a child transaction linked to the source transaction.

        For a provider to support authorization, it must override this method and make an API
        request to void the payment.

        Note: `self.ensure_one()`

        :param float amount_to_void: The amount to be voided.
        :return: The created void child transaction, if any.
        :rtype: payment.transaction
        """
        self.ensure_one()
        self._ensure_provider_is_not_disabled()

        if amount_to_void and amount_to_void != self.amount:
            return self._create_child_transaction(amount_to_void)

        return self.env['payment.transaction']

    def _ensure_provider_is_not_disabled(self):
        """ Ensure that the provider's state is not `disabled` before sending a request to its
        provider.

        :return: None
        :raise UserError: If the provider's state is `disabled`.
        """
        if self.provider_id.state == 'disabled':
            raise UserError(_(
                "Making a request to the provider is not possible because the provider is disabled."
            ))

    def _create_child_transaction(self, amount, is_refund=False, **custom_create_values):
        """ Create a new transaction with the current transaction as its parent transaction.

        This happens only in case of a refund or a partial capture (where the initial transaction is
        split between smaller transactions, either captured or voided).

        Note: self.ensure_one()

        :param float amount: The strictly positive amount of the child transaction, in the same
                             currency as the source transaction.
        :param bool is_refund: Whether the child transaction is a refund.
        :return: The created child transaction.
        :rtype: payment.transaction
        """
        self.ensure_one()

        if is_refund:
            reference_prefix = f'R-{self.reference}'
            amount = -amount
            operation = 'refund'
        else:  # Partial capture or void.
            reference_prefix = f'P-{self.reference}'
            operation = self.operation

        return self.create({
            'provider_id': self.provider_id.id,
            'payment_method_id': self.payment_method_id.id,
            'reference': self._compute_reference(self.provider_code, prefix=reference_prefix),
            'amount': amount,
            'currency_id': self.currency_id.id,
            'token_id': self.token_id.id,
            'operation': operation,
            'source_transaction_id': self.id,
            'partner_id': self.partner_id.id,
            **custom_create_values,
        })

    def _handle_notification_data(self, provider_code, notification_data):
        """ Match the transaction with the notification data, update its state and return it.

        :param str provider_code: The code of the provider handling the transaction.
        :param dict notification_data: The notification data sent by the provider.
        :return: The transaction.
        :rtype: recordset of `payment.transaction`
        """
        tx = self._get_tx_from_notification_data(provider_code, notification_data)
        tx._process_notification_data(notification_data)
        return tx

    def _get_tx_from_notification_data(self, provider_code, notification_data):
        """ Find the transaction based on the notification data.

        For a provider to handle transaction processing, it must overwrite this method and return
        the transaction matching the notification data.

        :param str provider_code: The code of the provider handling the transaction.
        :param dict notification_data: The notification data sent by the provider.
        :return: The transaction, if found.
        :rtype: recordset of `payment.transaction`
        """
        return self

    def _process_notification_data(self, notification_data):
        """ Update the transaction state and the provider reference based on the notification data.

        This method should usually not be called directly. The correct method to call upon receiving
        notification data is :meth:`_handle_notification_data`.

        For a provider to handle transaction processing, it must overwrite this method and process
        the notification data.

        Note: `self.ensure_one()`

        :param dict notification_data: The notification data sent by the provider.
        :return: None
        """
        self.ensure_one()

    def _set_pending(self, state_message=None, extra_allowed_states=()):
        """ Update the transactions' state to `pending`.

        :param str state_message: The reason for setting the transactions in the state `pending`.
        :param tuple[str] extra_allowed_states: The extra states that should be considered allowed
                                                target states for the source state 'pending'.
        :return: The updated transactions.
        :rtype: recordset of `payment.transaction`
        """
        allowed_states = ('draft',)
        target_state = 'pending'
        txs_to_process = self._update_state(
            allowed_states + extra_allowed_states, target_state, state_message
        )
        txs_to_process._log_received_message()
        return txs_to_process

    def _set_authorized(self, state_message=None, extra_allowed_states=()):
        """ Update the transactions' state to `authorized`.

        :param str state_message: The reason for setting the transactions in the state `authorized`.
        :param tuple[str] extra_allowed_states: The extra states that should be considered allowed
                                                target states for the source state 'authorized'.
        :return: The updated transactions.
        :rtype: recordset of `payment.transaction`
        """
        allowed_states = ('draft', 'pending')
        target_state = 'authorized'
        txs_to_process = self._update_state(
            allowed_states + extra_allowed_states, target_state, state_message
        )
        txs_to_process._log_received_message()
        return txs_to_process

    def _set_done(self, state_message=None, extra_allowed_states=()):
        """ Update the transactions' state to `done`.

        :param str state_message: The reason for setting the transactions in the state `done`.
        :param tuple[str] extra_allowed_states: The extra states that should be considered allowed
                                                target states for the source state 'done'.
        :return: The updated transactions.
        :rtype: recordset of `payment.transaction`
        """
        allowed_states = ('draft', 'pending', 'authorized', 'error')
        target_state = 'done'
        txs_to_process = self._update_state(
            allowed_states + extra_allowed_states, target_state, state_message
        )
        txs_to_process._update_source_transaction_state()
        txs_to_process._log_received_message()
        return txs_to_process

    def _set_canceled(self, state_message=None, extra_allowed_states=()):
        """ Update the transactions' state to `cancel`.

        :param str state_message: The reason for setting the transactions in the state `cancel`.
        :param tuple[str] extra_allowed_states: The extra states that should be considered allowed
                                                target states for the source state 'canceled'.
        :return: The updated transactions.
        :rtype: recordset of `payment.transaction`
        """
        allowed_states = ('draft', 'pending', 'authorized')
        target_state = 'cancel'
        txs_to_process = self._update_state(
            allowed_states + extra_allowed_states, target_state, state_message
        )
        txs_to_process._update_source_transaction_state()
        txs_to_process._log_received_message()
        return txs_to_process

    def _set_error(self, state_message, extra_allowed_states=()):
        """ Update the transactions' state to `error`.

        :param str state_message: The reason for setting the transactions in the state `error`.
        :param tuple[str] extra_allowed_states: The extra states that should be considered allowed
                                                target states for the source state 'error'.
        :return: The updated transactions.
        :rtype: recordset of `payment.transaction`
        """
        allowed_states = ('draft', 'pending', 'authorized')
        target_state = 'error'
        txs_to_process = self._update_state(
            allowed_states + extra_allowed_states, target_state, state_message
        )
        txs_to_process._log_received_message()
        return txs_to_process

    def _update_state(self, allowed_states, target_state, state_message):
        """ Update the transactions' state to the target state if the current state allows it.

        If the current state is the same as the target state, the transaction is skipped and a log
        with level INFO is created.

        :param tuple[str] allowed_states: The allowed source states for the target state.
        :param str target_state: The target state.
        :param str state_message: The message to set as `state_message`.
        :return: The recordset of transactions whose state was updated.
        :rtype: recordset of `payment.transaction`
        """
        def classify_by_state(transactions_):
            """ Classify the transactions according to their current state.

            For each transaction of the current recordset, if:

            - The state is an allowed state: the transaction is flagged as `to process`.
            - The state is equal to the target state: the transaction is flagged as `processed`.
            - The state matches none of above: the transaction is flagged as `in wrong state`.

            :param recordset transactions_: The transactions to classify, as a `payment.transaction`
                                            recordset.
            :return: A 3-items tuple of recordsets of classified transactions, in this order:
                     transactions `to process`, `processed`, and `in wrong state`.
            :rtype: tuple(recordset)
            """
            txs_to_process_ = transactions_.filtered(lambda _tx: _tx.state in allowed_states)
            txs_already_processed_ = transactions_.filtered(lambda _tx: _tx.state == target_state)
            txs_wrong_state_ = transactions_ - txs_to_process_ - txs_already_processed_

            return txs_to_process_, txs_already_processed_, txs_wrong_state_

        txs_to_process, txs_already_processed, txs_wrong_state = classify_by_state(self)
        for tx in txs_already_processed:
            _logger.info(
                "tried to write on transaction with reference %s with the same value for the "
                "state: %s",
                tx.reference, tx.state,
            )
        for tx in txs_wrong_state:
            _logger.warning(
                "tried to write on transaction with reference %(ref)s with illegal value for the "
                "state (previous state: %(tx_state)s, target state: %(target_state)s, expected "
                "previous state to be in: %(allowed_states)s)",
                {
                    'ref': tx.reference,
                    'tx_state': tx.state,
                    'target_state': target_state,
                    'allowed_states': allowed_states,
                },
            )
        txs_to_process.write({
            'state': target_state,
            'state_message': state_message,
            'last_state_change': fields.Datetime.now(),
            'is_post_processed': False,  # Reset to allow post-processing again for other states.
        })
        return txs_to_process

    def _update_source_transaction_state(self):
        """ Update the state of the source transactions for which all child transactions have
        reached a final state.

        :return: None
        """
        for child_tx in self.filtered('source_transaction_id'):
            sibling_txs = child_tx.source_transaction_id.child_transaction_ids.filtered(
                lambda tx: tx.state in ['done', 'cancel'] and tx.operation == child_tx.operation
            )
            processed_amount = round(
                sum(tx.amount for tx in sibling_txs), child_tx.currency_id.decimal_places
            )
            if child_tx.source_transaction_id.amount == processed_amount:
                state_message = _(
                    "This transaction has been confirmed following the processing of its partial "
                    "capture and partial void transactions (%(provider)s).",
                    provider=child_tx.provider_id.name,
                )
                # Call `_update_state` directly instead of `_set_authorized` to avoid looping.
                child_tx.source_transaction_id._update_state(('authorized',), 'done', state_message)

    #=== BUSINESS METHODS - POST-PROCESSING ===#

    def _cron_post_process(self):
        """ Trigger the post-processing of the transactions that were not handled by the client in
        the `poll_status` controller method.

        :return: None
        """
        txs_to_post_process = self
        if not txs_to_post_process:
            # Don't try forever to post-process a transaction that doesn't go through. Set the limit
            # to 4 days because some providers (PayPal) need that much for the payment verification.
            retry_limit_date = datetime.now() - relativedelta.relativedelta(days=4)
            # Retrieve all transactions matching the criteria for post-processing
            txs_to_post_process = self.search(
                [('is_post_processed', '=', False), ('last_state_change', '>=', retry_limit_date)]
            )
        for tx in txs_to_post_process:
            try:
                tx._post_process()
                self.env.cr.commit()
            except psycopg2.OperationalError:
                self.env.cr.rollback()  # Rollback and try later.
            except Exception as e:
                _logger.exception(
                    "encountered an error while post-processing transaction with reference %s:\n%s",
                    tx.reference, e
                )
                self.env.cr.rollback()

    def _post_process(self):
        """ Post-process the transactions.

        The generic post-processing only consists in flagging the transactions as post-processed.
        For a module to add its own logic to the post-processing, it must overwrite this method and
        apply its specific logic to the transactions, optionally after filtering them based on their
        state.

        :return: None
        """
        self.is_post_processed = True

    #=== BUSINESS METHODS - LOGGING ===#

    def _log_sent_message(self):
        """ Log that the transactions have been initiated in the chatter of relevant documents.

        :return: None
        """
        for tx in self:
            message = tx._get_sent_message()
            tx._log_message_on_linked_documents(message)

    def _log_received_message(self):
        """ Log that the transactions have been received in the chatter of relevant documents.

        A transaction is 'received' when a payment status is received from the provider handling the
        transaction.

        :return: None
        """
        for tx in self:
            message = tx._get_received_message()
            tx._log_message_on_linked_documents(message)

    def _log_message_on_linked_documents(self, message):
        """ Log a message on the records linked to the transaction.

        For a module to implement payments and link documents to a transaction, it must override
        this method and call it, then log the message on documents linked to the transaction.

        Note: `self.ensure_one()`

        :param str message: The message to log.
        :return: None
        """
        self.ensure_one()

    #=== BUSINESS METHODS - GETTERS ===#

    def _get_sent_message(self):
        """ Return the message stating that the transaction has been requested.

        Note: `self.ensure_one()`

        :return: The 'transaction sent' message.
        :rtype: str
        """
        self.ensure_one()

        # Choose the message based on the payment flow.
        if self.operation in ('online_redirect', 'online_direct'):
            message = _(
                "A transaction with reference %(ref)s has been initiated (%(provider_name)s).",
                ref=self.reference, provider_name=self.provider_id.name
            )
        elif self.operation == 'refund':
            formatted_amount = format_amount(self.env, -self.amount, self.currency_id)
            message = _(
                "A refund request of %(amount)s has been sent. The payment will be created soon. "
                "Refund transaction reference: %(ref)s (%(provider_name)s).",
                amount=formatted_amount, ref=self.reference, provider_name=self.provider_id.name
            )
        elif self.operation in ('online_token', 'offline'):
            message = _(
                "A transaction with reference %(ref)s has been initiated using the payment method "
                "%(token)s (%(provider_name)s).",
                ref=self.reference,
                token=self.token_id._build_display_name(),
                provider_name=self.provider_id.name
            )
        else:  # 'validation'
            message = _(
                "A transaction with reference %(ref)s has been initiated to save a new payment "
                "method (%(provider_name)s)",
                ref=self.reference,
                provider_name=self.provider_id.name,
            )
        return message

    def _get_received_message(self):
        """ Return the message stating that the transaction has been received by the provider.

        Note: `self.ensure_one()`

        :return: The 'transaction received' message.
        :rtype: str
        """
        self.ensure_one()

        formatted_amount = format_amount(self.env, self.amount, self.currency_id)
        if self.state == 'pending':
            message = _(
                ("The transaction with reference %(ref)s for %(amount)s "
                "is pending (%(provider_name)s)."),
                ref=self.reference,
                amount=formatted_amount,
                provider_name=self.provider_id.name
            )
        elif self.state == 'authorized':
            message = _(
                "The transaction with reference %(ref)s for %(amount)s has been authorized "
                "(%(provider_name)s).", ref=self.reference, amount=formatted_amount,
                provider_name=self.provider_id.name
            )
        elif self.state == 'done':
            message = _(
                "The transaction with reference %(ref)s for %(amount)s has been confirmed "
                "(%(provider_name)s).", ref=self.reference, amount=formatted_amount,
                provider_name=self.provider_id.name
            )
        elif self.state == 'error':
            message = _(
                "The transaction with reference %(ref)s for %(amount)s encountered an error"
                " (%(provider_name)s).",
                ref=self.reference, amount=formatted_amount, provider_name=self.provider_id.name
            )
            if self.state_message:
                message += Markup("<br/>") + _("Error: %s", self.state_message)
        else:
            message = _(
                ("The transaction with reference %(ref)s for %(amount)s is canceled "
                "(%(provider_name)s)."),
                ref=self.reference,
                amount=formatted_amount,
                provider_name=self.provider_id.name
            )
            if self.state_message:
                message += Markup("<br/>") + _("Reason: %s", self.state_message)
        return message

    def _get_last(self):
        """ Return the last transaction of the recordset.

        :return: The last transaction of the recordset, sorted by id.
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

    payment_onboarding_payment_method = fields.Selection(
        string="Selected onboarding payment method",
        selection=[
            ('paypal', "PayPal"),
            ('stripe', "Stripe"),
            ('manual', "Manual"),
            ('other', "Other"),
        ])

    def _run_payment_onboarding_step(self, menu_id=None):
        """ Install the suggested payment modules and configure the providers.

        It's checked that the current company has a Chart of Account.

        :param int menu_id: The menu from which the user started the onboarding step, as an
                            `ir.ui.menu` id
        :return: The action returned by `action_stripe_connect_account`
        :rtype: dict
        """
        self.env.company.get_chart_of_accounts_or_fail()

        self._install_modules(['payment_stripe'])

        # Create a new env including the freshly installed module(s)
        new_env = api.Environment(self.env.cr, self.env.uid, self.env.context)

        # Configure Stripe
        stripe_provider = new_env['payment.provider'].search([
            *self.env['payment.provider']._check_company_domain(self.env.company),
            ('code', '=', 'stripe')
        ], limit=1)
        if not stripe_provider:
            base_provider = self.env.ref('payment.payment_provider_stripe')
            # Use sudo to access payment provider record that can be in different company.
            stripe_provider = base_provider.sudo().with_context(
                stripe_connect_onboarding=True,
            ).copy(default={'company_id': self.env.company.id})

        return stripe_provider.action_stripe_connect_account(menu_id=menu_id)

    def _install_modules(self, module_names):
        modules_sudo = self.env['ir.module.module'].sudo().search([('name', 'in', module_names)])
        STATES = ['installed', 'to install', 'to upgrade']
        modules_sudo.filtered(lambda m: m.state not in STATES).button_immediate_install()

```

## File: models\res_country.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
import odoo.addons.payment_stripe as stripe  # prevent circular import error with payment_stripe


class ResCountry(models.Model):
    _inherit = 'res.country'

    is_stripe_supported_country = fields.Boolean(compute='_compute_is_stripe_supported_country')

    @api.depends('code')
    def _compute_is_stripe_supported_country(self):
        for country in self:
            country.is_stripe_supported_country = stripe.const.COUNTRY_MAPPING.get(
                country.code, country.code
            ) in stripe.const.SUPPORTED_COUNTRIES

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
        payments_data = self.env['payment.token']._read_group(
            [('partner_id', 'in', self.ids)], ['partner_id'], ['__count'],
        )
        partners_data = {partner.id: count for partner, count in payments_data}
        for partner in self:
            partner.payment_token_count = partners_data.get(partner.id, 0)

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import ir_http
from . import onboarding_step
from . import payment_method
from . import payment_provider
from . import payment_token
from . import payment_transaction
from . import res_company
from . import res_country
from . import res_partner

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_payment_link_wizard,access_payment_link_wizard,payment.model_payment_link_wizard,base.group_user,0,0,0,0
payment_capture_wizard_user,payment.capture.wizard,model_payment_capture_wizard,base.group_user,1,1,1,0
payment_provider_onboarding_wizard,payment.provider.onboarding.wizard,model_payment_provider_onboarding_wizard,base.group_system,1,1,1,0
payment_provider_system,payment.provider.system,model_payment_provider,base.group_system,1,1,1,1
payment_method_public,payment.method.all,model_payment_method,base.group_public,1,0,0,0
payment_method_portal,payment.method.all,model_payment_method,base.group_portal,1,0,0,0
payment_method_employee,payment.method.all,model_payment_method,base.group_user,1,0,0,0
payment_method_system,payment.method.system,model_payment_method,base.group_system,1,1,1,1
payment_token_public,payment.token.all,model_payment_token,base.group_public,1,0,0,0
payment_token_portal,payment.token.all,model_payment_token,base.group_portal,1,0,0,0
payment_token_employee,payment.token.all,model_payment_token,base.group_user,1,0,0,0
payment_token_system,payment.token.system,model_payment_token,base.group_system,1,1,1,1
payment_transaction_system,payment.transaction.system,model_payment_transaction,base.group_system,1,1,1,1

```

## File: security\payment_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <!-- Providers -->

    <record id="payment_provider_company_rule" model="ir.rule">
        <field name="name">Access providers in own companies only</field>
        <field name="model_id" ref="payment.model_payment_provider"/>
        <field name="domain_force">[('company_id', 'parent_of', company_ids)]</field>
    </record>

    <!-- Transactions -->

    <record id="transaction_company_rule" model="ir.rule">
        <field name="name">Access transactions in own companies only</field>
        <field name="model_id" ref="payment.model_payment_transaction"/>
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>

    <!-- Tokens -->

    <record id="payment_token_user_rule" model="ir.rule">
        <field name="name">Users can access only their own tokens</field>
        <field name="model_id" ref="payment.model_payment_token"/>
        <field name="domain_force">[('partner_id', '=', user.partner_id.id)]</field>
        <field name="groups" eval="[(4, ref('base.group_user')),
                                    (4, ref('base.group_portal')),
                                    (4, ref('base.group_public'))]"/>
    </record>

    <record id="payment_token_company_rule" model="ir.rule">
        <field name="name">Access tokens in own companies only</field>
        <field name="model_id" ref="payment.model_payment_token"/>
        <field name="domain_force">[('company_id', 'parent_of', company_ids)]</field>
    </record>

    <!-- Wizards -->

    <record id="payment_capture_wizard_rule" model="ir.rule">
        <field name="name">Payment Capture Wizard</field>
        <field name="model_id" ref="model_payment_capture_wizard"/>
        <field name="domain_force">[('create_uid', '=', user.id)]</field>
     </record>

</odoo>

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M0 12h50v26a4 4 0 0 1-4 4H0V12Z" fill="#985184"/><path d="M4 21a4 4 0 0 1-4-4v-5a4 4 0 0 1 4-4h46v9a4 4 0 0 1-4 4H4Z" fill="#FBB945"/><path d="M0 16h50v4a4 4 0 0 1-4 4H4a4 4 0 0 1-4-4v-4Z" fill="#F78613"/></svg>

```

## File: static\img\payment-methods.svg

```svg
<svg width="64" height="64" viewBox="0 0 64 64" fill="none" xmlns="http://www.w3.org/2000/svg">
<path d="M51.8288 2.55638C52.0805 2.55638 52.3323 2.62152 52.5578 2.7518L54.158 3.67631C54.6084 3.93658 54.886 4.41727 54.8862 4.93752L54.897 36.072C54.8971 36.592 54.6203 37.0726 54.1704 37.3333L12.9021 61.2475C12.6762 61.3784 12.4238 61.4438 12.1715 61.4438C11.9198 61.4438 11.668 61.3787 11.4425 61.2484L9.84226 60.3238C9.39179 60.0635 9.11424 59.5828 9.11405 59.0626L9.10311 27.9344C9.10292 27.4145 9.37975 26.9339 9.82955 26.6732L51.0981 2.75283C51.3241 2.62186 51.5764 2.55638 51.8288 2.55638ZM51.8288 1.64209C51.4117 1.64209 51.0005 1.75263 50.6396 1.96179L9.37106 25.8822C8.64155 26.3051 8.18855 27.0916 8.18884 27.9348L8.19979 59.063C8.20009 59.9068 8.65419 60.6933 9.38489 61.1154L10.9851 62.04C11.3453 62.2481 11.7555 62.3581 12.1715 62.3581C12.5885 62.3581 12.9997 62.2476 13.3605 62.0385L54.6288 38.1244C55.3585 37.7016 55.8116 36.915 55.8113 36.0717L55.8005 4.93722C55.8002 4.09334 55.3461 3.30684 54.6154 2.88465L53.0152 1.96015C52.655 1.75206 52.2448 1.64209 51.8288 1.64209Z" fill="#374874"/>
<path d="M54.8858 4.09686L51.8276 2.32996L9.10278 27.0945L12.161 28.8614L54.8858 4.09686Z" fill="white"/>
<path d="M12.1725 61.6702L9.11433 59.9033L9.10278 27.0945L12.161 28.8614L12.1725 61.6702Z" fill="#FBDBD0"/>
<path d="M12.1726 61.6702L54.8972 36.9121L54.8857 4.10339L12.161 28.8614L12.1726 61.6702Z" fill="white"/>
<path d="M54.891 20.5496L12.1666 45.2097V34.0217L54.891 9.36157V20.5496Z" fill="#374874"/>
<path d="M49.0564 35.9273L31.7461 45.8722C31.213 46.1777 30.5331 45.9932 30.2276 45.4601C30.1309 45.2913 30.0801 45.1002 30.0803 44.9057V42.6122C30.0816 41.8655 30.4807 41.1762 31.1276 40.8034L49.1186 30.4856C49.5803 30.2213 50.169 30.3814 50.4333 30.8431C50.5167 30.9888 50.5605 31.1537 50.5606 31.3216V33.3323C50.5595 34.4041 49.9859 35.3936 49.0564 35.9273Z" fill="#C1DBF6"/>
<path d="M18.2645 50.7359C18.1862 49.5732 18.4754 48.4154 19.0912 47.4261C18.4364 47.0959 17.6638 47.0959 17.009 47.4261C15.309 48.1906 14.0815 50.478 14.2648 52.5322C14.4482 54.5864 15.9772 55.64 17.6834 54.8661C18.4872 54.4703 19.1548 53.8438 19.6009 53.0667C18.8768 52.6969 18.3671 51.8764 18.2645 50.7359Z" fill="#FBDBD0"/>
<path d="M24.4211 47.9699C24.2377 45.9157 22.7087 44.8621 21.0025 45.6391C20.1986 46.0345 19.5302 46.6597 19.0819 47.4354C19.7998 47.8207 20.325 48.6443 20.412 49.7755C20.4899 50.9372 20.2007 52.0939 19.5854 53.0822C20.24 53.4129 21.0129 53.4129 21.6676 53.0822C23.3769 52.3115 24.6044 50.0273 24.4211 47.9699Z" fill="#C1DBF6"/>
<path d="M20.4214 49.7756C20.3188 48.6444 19.7998 47.8208 19.0912 47.4354C18.4774 48.4221 18.1882 49.5764 18.2646 50.7359C18.3671 51.8671 18.8861 52.6876 19.5947 53.0729C20.2081 52.0872 20.4972 50.9341 20.4214 49.7756Z" fill="#374874"/>
</svg>

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

## File: static\src\js\express_checkout_form.js

```javascript
/** @odoo-module */

import publicWidget from '@web/legacy/js/public/public_widget';
import { Component } from '@odoo/owl';

publicWidget.registry.PaymentExpressCheckoutForm = publicWidget.Widget.extend({
    selector: 'form[name="o_payment_express_checkout_form"]',

    /**
     * @override
     */
    start: async function () {
        await this._super(...arguments);
        this.paymentContext = {};
        Object.assign(this.paymentContext, this.el.dataset);
        this.paymentContext.shippingInfoRequired = !!this.paymentContext['shippingInfoRequired'];
        const expressCheckoutForms = this._getExpressCheckoutForms();
        for (const expressCheckoutForm of expressCheckoutForms) {
            await this._prepareExpressCheckoutForm(expressCheckoutForm.dataset);
        }
        // Monitor updates of the amount on eCommerce's cart pages.
        Component.env.bus.addEventListener('cart_amount_changed', (ev) => this._updateAmount(...ev.detail));
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Return all express checkout forms found on the page.
     *
     * @private
     * @return {NodeList} - All express checkout forms found on the page.
     */
    _getExpressCheckoutForms() {
        return document.querySelectorAll(
            'form[name="o_payment_express_checkout_form"] div[name="o_express_checkout_container"]'
        );
    },

    /**
     * Prepare the provider-specific express checkout form based on the provided data.
     *
     * For a provider to manage an express checkout form, it must override this method.
     *
     * @private
     * @param {Object} providerData - The provider-specific data.
     * @return {void}
     */
    async _prepareExpressCheckoutForm(providerData) {},

    /**
     * Prepare the params for the RPC to the transaction route.
     *
     * @private
     * @param {number} providerId - The id of the provider handling the transaction.
     * @returns {object} - The transaction route params.
     */
    _prepareTransactionRouteParams(providerId) {
        return {
            'provider_id': parseInt(providerId),
            'payment_method_id': parseInt(this.paymentContext['paymentMethodUnknownId']),
            'token_id': null,
            'flow': 'direct',
            'tokenization_requested': false,
            'landing_route': this.paymentContext['landingRoute'],
            'access_token': this.paymentContext['accessToken'],
            'csrf_token': odoo.csrf_token,
        };
    },

    /**
     * Update the amount of the express checkout form.
     *
     * For a provider to manage an express form, it must override this method.
     *
     * @private
     * @param {number} newAmount - The new amount.
     * @param {number} newMinorAmount - The new minor amount.
     * @return {void}
     */
    _updateAmount(newAmount, newMinorAmount) {
        this.paymentContext.amount = parseFloat(newAmount);
        this.paymentContext.minorAmount = parseInt(newMinorAmount);
        this._getExpressCheckoutForms().forEach(form => {
            if (newAmount == 0) {
                form.classList.add('d-none')}
            else {
                form.classList.remove('d-none')
            }
        })
    },

});

export const paymentExpressCheckoutForm = publicWidget.registry.PaymentExpressCheckoutForm;

```

## File: static\src\js\payment_button.js

```javascript
/** @odoo-module **/

import publicWidget from '@web/legacy/js/public/public_widget';
import { Component } from "@odoo/owl";

publicWidget.registry.PaymentButton = publicWidget.Widget.extend({
    selector: '[name="o_payment_submit_button"]',

    async start() {
        await this._super(...arguments);
        this.paymentButton = this.el;
        this.iconClass = this.paymentButton.dataset.iconClass;
        this._enable();
        Component.env.bus.addEventListener('enablePaymentButton', this._enable.bind(this));
        Component.env.bus.addEventListener('disablePaymentButton',this._disable.bind(this));
        Component.env.bus.addEventListener('hidePaymentButton', this._hide.bind(this));
        Component.env.bus.addEventListener('showPaymentButton', this._show.bind(this));
    },

    /**
     * Check if the payment button can be enabled and do it if so.
     *
     * @private
     * @return {void}
     */
    _enable() {
        if (this._canSubmit()) {
            this._setEnabled();
        }
    },

    /**
     * Check whether the payment form can be submitted, i.e. whether exactly one payment option is
     * selected.
     *
     * For a module to add a condition on the submission of the form, it must override this method
     * and return whether both this method's condition and the override method's condition are met.
     *
     * @private
     * @return {boolean} Whether the form can be submitted.
     */
    _canSubmit() {
        const paymentForm = document.querySelector('#o_payment_form');
        if (!paymentForm) {  // Payment form is not present.
            return true; // Ignore the check.
        }
        return document.querySelectorAll('input[name="o_payment_radio"]:checked').length === 1;
    },

    /**
     * Enable the payment button.
     *
     * @private
     * @return {void}
     */
    _setEnabled() {
        this.paymentButton.disabled = false;
    },

    /**
     * Disable the payment button.
     *
     * @private
     * @return {void}
     */
    _disable() {
        this.paymentButton.disabled = true;
    },

    /**
     * Hide the payment button.
     *
     * @private
     * @return {void}
     */
    _hide() {
        this.paymentButton.classList.add('d-none');
    },

    /**
     * Show the payment button.
     *
     * @private
     * @return {void}
     */
    _show() {
        this.paymentButton.classList.remove('d-none');
    },

});
export default publicWidget.registry.PaymentButton;

```

## File: static\src\js\payment_form.js

```javascript
/** @odoo-module **/

import { Component } from '@odoo/owl';
import publicWidget from '@web/legacy/js/public/public_widget';
import { browser } from '@web/core/browser/browser';
import { ConfirmationDialog } from '@web/core/confirmation_dialog/confirmation_dialog';
import { _t } from '@web/core/l10n/translation';
import { renderToMarkup } from '@web/core/utils/render';
import { rpc, RPCError } from '@web/core/network/rpc';

publicWidget.registry.PaymentForm = publicWidget.Widget.extend({
    selector: '#o_payment_form',
    events: Object.assign({}, publicWidget.Widget.prototype.events, {
        'click [name="o_payment_radio"]': '_selectPaymentOption',
        'click [name="o_payment_delete_token"]': '_fetchTokenData',
        'click [name="o_payment_expand_button"]': '_hideExpandButton',
        'click [name="o_payment_submit_button"]': '_submitForm',
    }),

    // #=== WIDGET LIFECYCLE ===#

    /**
     * @override
     */
    init() {
        this._super(...arguments);
        this.orm = this.bindService("orm");
    },

    /**
     * @override
     */
    async start() {
        // Synchronously initialize paymentContext before any await.
        this.paymentContext = {};
        Object.assign(this.paymentContext, this.el.dataset);

        await this._super(...arguments);

        // Expand the payment form of the selected payment option if there is only one.
        const checkedRadio = document.querySelector('input[name="o_payment_radio"]:checked');
        if (checkedRadio) {
            await this._expandInlineForm(checkedRadio);
            this._enableButton(false);
        } else {
            this._setPaymentFlow(); // Initialize the payment flow to let providers overwrite it.
        }

        this.$('[data-bs-toggle="tooltip"]').tooltip();
    },

    // #=== EVENT HANDLERS ===#

    /**
     * Open the inline form of the selected payment option, if any.
     *
     * @private
     * @param {Event} ev
     * @return {void}
     */
    async _selectPaymentOption(ev) {
        // Show the inputs in case they have been hidden.
        this._showInputs();

        // Disable the submit button while preparing the inline form.
        this._disableButton();

        // Unfold and prepare the inline form of the selected payment option.
        const checkedRadio = ev.target;
        await this._expandInlineForm(checkedRadio);

        // Re-enable the submit button after the inline form has been prepared.
        this._enableButton(false);
    },

    /**
     * Fetch data relative to the documents linked to the token and delegate them to the token
     * deletion confirmation dialog.
     *
     * @private
     * @param {Event} ev
     * @return {void}
     */
    _fetchTokenData(ev) {
        ev.preventDefault();

        const linkedRadio = document.getElementById(ev.currentTarget.dataset['linkedRadio']);
        const tokenId = this._getPaymentOptionId(linkedRadio);
        this.orm.call(
            'payment.token',
            'get_linked_records_info',
            [tokenId],
        ).then(linkedRecordsInfo => {
            this._challengeTokenDeletion(tokenId, linkedRecordsInfo);
        }).catch(error => {
            if (error instanceof RPCError) {
                this._displayErrorDialog(
                    _t("Cannot delete payment method"), error.data.message
                );
            } else {
                return Promise.reject(error);
            }
        });
    },

    /**
     * Hide the button to expand the payment methods section once it has been clicked.
     *
     * @private
     * @param {Event} ev
     * @return {void}
     */
    _hideExpandButton(ev) {
        ev.target.classList.add('d-none');
    },

    /**
     * Update the payment context with the selected payment option and initiate its payment flow.
     *
     * @private
     * @param {Event} ev
     * @return {void}
     */
    async _submitForm(ev) {
        ev.stopPropagation();
        ev.preventDefault();

        const checkedRadio = this.el.querySelector('input[name="o_payment_radio"]:checked');

        // Block the entire UI to prevent fiddling with other widgets.
        this._disableButton(true);

        // Initiate the payment flow of the selected payment option.
        const flow = this.paymentContext.flow = this._getPaymentFlow(checkedRadio);
        const paymentOptionId = this.paymentContext.paymentOptionId = this._getPaymentOptionId(
            checkedRadio
        );
        if (flow === 'token' && this.paymentContext['assignTokenRoute']) { // Assign token flow.
            await this._assignToken(paymentOptionId);
        } else { // Both tokens and payment methods must process a payment operation.
            const providerCode = this.paymentContext.providerCode = this._getProviderCode(
                checkedRadio
            );
            const pmCode = this.paymentContext.paymentMethodCode = this._getPaymentMethodCode(
                checkedRadio
            );
            this.paymentContext.providerId = this._getProviderId(checkedRadio);
            if (this._getPaymentOptionType(checkedRadio) === 'token') {
                this.paymentContext.tokenId = paymentOptionId;
            } else { // 'payment_method'
                this.paymentContext.paymentMethodId = paymentOptionId;
            }
            const inlineForm = this._getInlineForm(checkedRadio);
            this.paymentContext.tokenizationRequested = inlineForm?.querySelector(
                '[name="o_payment_tokenize_checkbox"]'
            )?.checked ?? this.paymentContext['mode'] === 'validation';
            await this._initiatePaymentFlow(providerCode, paymentOptionId, pmCode, flow);
        }
    },

    // #=== DOM MANIPULATION ===#

    /**
     * Check if the submit button can be enabled and do it if so.
     *
     * @private
     * @param {boolean} unblockUI - Whether the UI should also be unblocked.
     * @return {void}
     */
    _enableButton(unblockUI = true) {
        Component.env.bus.trigger('enablePaymentButton');
        if (unblockUI) {
            this.call('ui', 'unblock');
        }
    },

    /**
     * Disable the submit button.
     *
     * @private
     * @param {boolean} blockUI - Whether the UI should also be blocked.
     * @return {void}
     */
    _disableButton(blockUI = false) {
        Component.env.bus.trigger('disablePaymentButton');
        if (blockUI) {
            this.call('ui', 'block');
        }
    },

    /**
     * Show the tokenization checkbox, its label, and the submit button.
     *
     * @private
     * @return {void}
     */
    _showInputs() {
        // Show the tokenization checkbox and its label.
        const tokenizeContainer = this.el.querySelector('[name="o_payment_tokenize_container"]');
        tokenizeContainer?.classList.remove('d-none');

        // Show the submit button.
        Component.env.bus.trigger('showPaymentButton');
    },

    /**
     * Hide the tokenization checkbox, its label, and the submit button.
     *
     * The inputs should typically be hidden when the customer has to perform additional actions in
     * the inline form. All inputs are automatically shown again when the customer selects another
     * payment option.
     *
     * @private
     * @return {void}
     */
    _hideInputs() {
        // Hide the tokenization checkbox and its label.
        const tokenizeContainer = this.el.querySelector('[name="o_payment_tokenize_container"]');
        tokenizeContainer?.classList.add('d-none');

        // Hide the submit button.
        Component.env.bus.trigger('hidePaymentButton');
    },

    /**
     * Open the inline form of the selected payment option and collapse the others.
     *
     * @private
     * @param {HTMLInputElement} radio - The radio button linked to the payment option.
     * @return {void}
     */
    async _expandInlineForm(radio) {
        this._collapseInlineForms(); // Collapse previously opened inline forms.
        this._setPaymentFlow(); // Reset the payment flow to let providers overwrite it.

        // Prepare the inline form of the selected payment option.
        const providerId = this._getProviderId(radio);
        const providerCode = this._getProviderCode(radio);
        const paymentOptionId = this._getPaymentOptionId(radio);
        const paymentMethodCode = this._getPaymentMethodCode(radio);
        const flow = this._getPaymentFlow(radio);
        await this._prepareInlineForm(
            providerId, providerCode, paymentOptionId, paymentMethodCode, flow
        );

        // Display the prepared inline form if it is not empty.
        const inlineForm = this._getInlineForm(radio);
        if (inlineForm && inlineForm.children.length > 0) {
            inlineForm.classList.remove('d-none');
        }
    },

    /**
     * Prepare the provider-specific inline form of the selected payment option.
     *
     * For a provider to manage an inline form, it must override this method and render the content
     * of the form.
     *
     * @private
     * @param {number} providerId - The id of the selected payment option's provider.
     * @param {string} providerCode - The code of the selected payment option's provider.
     * @param {number} paymentOptionId - The id of the selected payment option.
     * @param {string} paymentMethodCode - The code of the selected payment method, if any.
     * @param {string} flow - The online payment flow of the selected payment option.
     * @return {void}
     */
    async _prepareInlineForm(providerId, providerCode, paymentOptionId, paymentMethodCode, flow) {},

    /**
     * Collapse all inline forms of the current widget.
     *
     * @private
     * @return {void}
     */
    _collapseInlineForms() {
        this.el.querySelectorAll('[name="o_payment_inline_form"]').forEach(inlineForm => {
            inlineForm.classList.add('d-none');
        });
    },

    /**
     * Display an error dialog.
     *
     * @private
     * @param {string} title - The title of the dialog.
     * @param {string} errorMessage - The error message.
     * @return {void}
     */
    _displayErrorDialog(title, errorMessage = '') {
        this.call('dialog', 'add', ConfirmationDialog, { title: title, body: errorMessage || "" });
    },

    /**
     * Display the token deletion confirmation dialog.
     *
     * @private
     * @param {number} tokenId - The id of the token whose deletion was requested.
     * @param {object} linkedRecordsInfo - The data relative to the documents linked to the token.
     * @return {void}
     */
    _challengeTokenDeletion(tokenId, linkedRecordsInfo) {
        const body = renderToMarkup('payment.deleteTokenDialog', { linkedRecordsInfo });
        this.call('dialog', 'add', ConfirmationDialog, {
            title: _t("Warning!"),
            body,
            confirmLabel: _t("Confirm Deletion"),
            confirm: () => this._archiveToken(tokenId),
            cancel: () => {},
        });
    },

    // #=== PAYMENT FLOW ===#

    /**
     * Set the payment flow for the selected payment option.
     *
     * For a provider to manage direct payments, it must call this method and set the payment flow
     * when its payment option is selected.
     *
     * @private
     * @param {string} flow - The flow for the selected payment option. Either 'redirect', 'direct',
     *                        or 'token'
     * @return {void}
     */
    _setPaymentFlow(flow = 'redirect') {
        if (['redirect', 'direct', 'token'].includes(flow)) {
            this.paymentContext.flow = flow;
        } else {
            console.warn(`The value ${flow} is not a supported flow. Falling back to redirect.`);
            this.paymentContext.flow = 'redirect';
        }
    },

    /**
     * Assign the selected token to a document through the `assignTokenRoute`.
     *
     * @private
     * @param {number} tokenId - The id of the token to assign.
     * @return {void}
     */
    async _assignToken(tokenId) {
        rpc(this.paymentContext['assignTokenRoute'], {
            'token_id': tokenId,
            'access_token': this.paymentContext['accessToken'],
        }).then(() => {
            window.location = this.paymentContext['landingRoute'];
        }).catch(error => {
            if (error instanceof RPCError) {
                this._displayErrorDialog(_t("Cannot save payment method"), error.data.message);
                this._enableButton(); // The button has been disabled before initiating the flow.
            } else {
                return Promise.reject(error);
            }
        });
    },

    /**
     * Make an RPC to initiate the payment flow by creating a new transaction.
     *
     * For a provider to do pre-processing work (e.g., perform checks on the form inputs), or to
     * process the payment flow in its own terms (e.g., re-schedule the RPC to the transaction
     * route), it must override this method.
     *
     * To alter the flow-specific processing, it is advised to override `_processRedirectFlow`,
     * `_processDirectFlow`, or `_processTokenFlow` instead.
     *
     * @private
     * @param {string} providerCode - The code of the selected payment option's provider.
     * @param {number} paymentOptionId - The id of the selected payment option.
     * @param {string} paymentMethodCode - The code of the selected payment method, if any.
     * @param {string} flow - The payment flow of the selected payment option.
     * @return {void}
     */
    async _initiatePaymentFlow(providerCode, paymentOptionId, paymentMethodCode, flow) {
        // Create a transaction and retrieve its processing values.
        await rpc(
            this.paymentContext['transactionRoute'],
            this._prepareTransactionRouteParams(),
        ).then(processingValues => {
            if (flow === 'redirect') {
                this._processRedirectFlow(
                    providerCode, paymentOptionId, paymentMethodCode, processingValues
                );
            } else if (flow === 'direct') {
                this._processDirectFlow(
                    providerCode, paymentOptionId, paymentMethodCode, processingValues
                );
            } else if (flow === 'token') {
                this._processTokenFlow(
                    providerCode, paymentOptionId, paymentMethodCode, processingValues
                );
            }
        }).catch(error => {
            if (error instanceof RPCError) {
                this._displayErrorDialog(_t("Payment processing failed"), error.data.message);
                this._enableButton(); // The button has been disabled before initiating the flow.
            }
            return Promise.reject(error);
        });
    },

    /**
     * Prepare the params for the RPC to the transaction route.
     *
     * @private
     * @return {object} The transaction route params.
     */
    _prepareTransactionRouteParams() {
        let transactionRouteParams = {
            'provider_id': this.paymentContext.providerId,
            'payment_method_id': this.paymentContext.paymentMethodId ?? null,
            'token_id': this.paymentContext.tokenId ?? null,
            'amount': this.paymentContext['amount'] !== undefined
                ? parseFloat(this.paymentContext['amount']) : null,
            'flow': this.paymentContext['flow'],
            'tokenization_requested': this.paymentContext['tokenizationRequested'],
            'landing_route': this.paymentContext['landingRoute'],
            'is_validation': this.paymentContext['mode'] === 'validation',
            'access_token': this.paymentContext['accessToken'],
            'csrf_token': odoo.csrf_token,
        };
        // Generic payment flows (i.e., that are not attached to a document) require extra params.
        if (this.paymentContext['transactionRoute'] === '/payment/transaction') {
            Object.assign(transactionRouteParams, {
                'currency_id': this.paymentContext['currencyId']
                    ? parseInt(this.paymentContext['currencyId']) : null,
                'partner_id': parseInt(this.paymentContext['partnerId']),
                'reference_prefix': this.paymentContext['referencePrefix']?.toString(),
            });
        }
        return transactionRouteParams;
    },

    /**
     * Redirect the customer by submitting the redirect form included in the processing values.
     *
     * @private
     * @param {string} providerCode - The code of the selected payment option's provider.
     * @param {number} paymentOptionId - The id of the selected payment option.
     * @param {string} paymentMethodCode - The code of the selected payment method, if any.
     * @param {object} processingValues - The processing values of the transaction.
     * @return {void}
     */
    _processRedirectFlow(providerCode, paymentOptionId, paymentMethodCode, processingValues) {
        // Create and configure the form element with the content rendered by the server.
        const div = document.createElement('div');
        div.innerHTML = processingValues['redirect_form_html'];
        const redirectForm = div.querySelector('form');
        redirectForm.setAttribute('id', 'o_payment_redirect_form');
        redirectForm.setAttribute('target', '_top');  // Ensures redirections when in an iframe.

        // Submit the form.
        document.body.appendChild(redirectForm);
        redirectForm.submit();
    },

   /**
     * Process the provider-specific implementation of the direct payment flow.
     *
     * @private
     * @param {string} providerCode - The code of the selected payment option's provider.
     * @param {number} paymentOptionId - The id of the selected payment option.
     * @param {string} paymentMethodCode - The code of the selected payment method, if any.
     * @param {object} processingValues - The processing values of the transaction.
     * @return {void}
     */
    _processDirectFlow(providerCode, paymentOptionId, paymentMethodCode, processingValues) {},

    /**
     * Redirect the customer to the status route.
     *
     * @private
     * @param {string} providerCode - The code of the selected payment option's provider.
     * @param {number} paymentOptionId - The id of the selected payment option.
     * @param {string} paymentMethodCode - The code of the selected payment method, if any.
     * @param {object} processingValues - The processing values of the transaction.
     * @return {void}
     */
    _processTokenFlow(providerCode, paymentOptionId, paymentMethodCode, processingValues) {
        // The flow is already completed as payments by tokens are immediately processed.
        window.location = '/payment/status';
    },

    /**
     * Archive the provided token.
     *
     * @private
     * @param {number} tokenId - The id of the token whose deletion was requested.
     * @return {void}
     */
    _archiveToken(tokenId) {
        rpc('/payment/archive_token', {
            'token_id': tokenId,
        }).then(() => {
            browser.location.reload();
        }).catch(error => {
            if (error instanceof RPCError) {
                this._displayErrorDialog(
                    _t("Cannot delete payment method"), error.data.message
                );
            } else {
                return Promise.reject(error);
            }
        });
    },

    // #=== GETTERS ===#

    /**
     * Determine and return the inline form of the selected payment option.
     *
     * @private
     * @param {HTMLInputElement} radio - The radio button linked to the payment option.
     * @return {Element | null} The inline form of the selected payment option, if any.
     */
    _getInlineForm(radio) {
        const inlineFormContainer = radio.closest('[name="o_payment_option"]');
        return inlineFormContainer?.querySelector('[name="o_payment_inline_form"]');
    },

    /**
     * Determine and return the payment flow of the selected payment option.
     *
     * As some providers implement both direct payments and the payment with redirection flow, we
     * cannot infer it from the radio button only. The radio button indicates only whether the
     * payment option is a token. If not, the payment context is looked up to determine whether the
     * flow is 'direct' or 'redirect'.
     *
     * @private
     * @param {HTMLInputElement} radio - The radio button linked to the payment option.
     * @return {string} The flow of the selected payment option: 'redirect', 'direct' or 'token'.
     */
    _getPaymentFlow(radio) {
        // The flow is read from the payment context too in case it was forced in a custom implem.
        if (this._getPaymentOptionType(radio) === 'token' || this.paymentContext.flow === 'token') {
            return 'token';
        } else if (this.paymentContext.flow === 'redirect') {
            return 'redirect';
        } else {
            return 'direct';
        }
    },

    /**
     * Determine and return the code of the selected payment method.
     *
     * @private
     * @param {HTMLElement} radio - The radio button linked to the payment method.
     * @return {string} The code of the selected payment method.
     */
    _getPaymentMethodCode(radio) {
        return radio.dataset['paymentMethodCode'];
    },

    /**
     * Determine and return the id of the selected payment option.
     *
     * @private
     * @param {HTMLElement} radio - The radio button linked to the payment option.
     * @return {number} The id of the selected payment option.
     */
    _getPaymentOptionId(radio) {
        return Number(radio.dataset['paymentOptionId']);
    },

    /**
     * Determine and return the type of the selected payment option.
     *
     * @private
     * @param {HTMLElement} radio - The radio button linked to the payment option.
     * @return {string} The type of the selected payment option: 'token' or 'payment_method'.
     */
    _getPaymentOptionType(radio) {
        return radio.dataset['paymentOptionType'];
    },

    /**
     * Determine and return the id of the provider of the selected payment option.
     *
     * @private
     * @param {HTMLElement} radio - The radio button linked to the payment option.
     * @return {number} The id of the provider of the selected payment option.
     */
    _getProviderId(radio) {
        return Number(radio.dataset['providerId']);
    },

    /**
     * Determine and return the code of the provider of the selected payment option.
     *
     * @private
     * @param {HTMLElement} radio - The radio button linked to the payment option.
     * @return {string} The code of the provider of the selected payment option.
     */
    _getProviderCode(radio) {
        return radio.dataset['providerCode'];
    },

    /**
     * Determine and return the state of the provider of the selected payment option.
     *
     * @private
     * @param {HTMLElement} radio - The radio button linked to the payment option.
     * @return {string} The state of the provider of the selected payment option.
     */
    _getProviderState(radio) {
        return radio.dataset['providerState'];
    },

});

export default publicWidget.registry.PaymentForm;

```

## File: static\src\js\payment_wizard_copy_clipboard_field.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import {
    copyClipboardButtonField,
    CopyClipboardButtonField,
} from "@web/views/fields/copy_clipboard/copy_clipboard_field";

import { CopyButton } from "@web/core/copy_button/copy_button";

class PaymentWizardCopyButton extends CopyButton {
    async onClick() {
        await this.env.model.mutex.getUnlockedDef();
        return super.onClick();
    }
}

class PaymentWizardCopyClipboardButtonField extends CopyClipboardButtonField {
    static components = { CopyButton: PaymentWizardCopyButton };
}

const paymentWizardCopyClipboardButtonField = {
    ...copyClipboardButtonField,
    component: PaymentWizardCopyClipboardButtonField,
};

registry
    .category("fields")
    .add("PaymentWizardCopyClipboardButtonField", paymentWizardCopyClipboardButtonField);

```

## File: static\src\js\post_processing.js

```javascript
/** @odoo-module **/

import publicWidget from '@web/legacy/js/public/public_widget';
import { ConnectionLostError, rpc, RPCError } from '@web/core/network/rpc';

publicWidget.registry.PaymentPostProcessing = publicWidget.Widget.extend({
    selector: 'div[name="o_payment_status"]',

    timeout: 0,
    pollCount: 0,

    async start() {
        this._poll();
        return this._super.apply(this, arguments);
    },

    _poll() {
        this._updateTimeout();
        setTimeout(() => {
            // Fetch the post-processing values from the server.
            const self = this;
            rpc('/payment/status/poll', {
                'csrf_token': odoo.csrf_token,
            }).then(postProcessingValues => {
                let {provider_code, state, landing_route} = postProcessingValues;

                // Redirect the user to the landing route if the transaction reached a final state.
                if (self._getFinalStates(provider_code).has(state)) {
                    window.location = landing_route;
                } else {
                    self._poll();
                }
            }).catch(error => {
                const isRetryError = error instanceof RPCError && error.data.message === 'retry';
                const isConnectionLostError = error instanceof ConnectionLostError;
                if (isRetryError || isConnectionLostError) {
                    self._poll();
                }
                if (!isRetryError) {
                    throw error;
                }
            });
        }, this.timeout);
    },

    _getFinalStates(providerCode) {
        return new Set(['authorized', 'done', 'cancel', 'error']);
    },

    _updateTimeout() {
        if (this.pollCount >= 1 && this.pollCount < 10) {
            this.timeout = 3000;
        }
        if (this.pollCount >= 10 && this.pollCount < 20) {
            this.timeout = 10000;
        }
        else if (this.pollCount >= 20) {
            this.timeout = 30000;
        }
        this.pollCount++;
    },
});

export default publicWidget.registry.PaymentPostProcessing;

```

## File: static\src\xml\payment_form_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates xml:space="preserve">

    <t t-name="payment.deleteTokenDialog">
        <div>
            <p>Are you sure you want to delete this payment method?</p>
            <t t-if="linkedRecordsInfo.length > 0">
                <p>It is currently linked to the following documents:</p>
                <ul>
                    <li t-foreach="linkedRecordsInfo" t-as="documentInfo" t-key="documentInfoIndex">
                        <a t-att-title="documentInfo.description"
                           t-att-href="documentInfo.url"
                           t-esc="documentInfo.name"
                        />
                    </li>
                </ul>
            </t>
        </div>
    </t>

</templates>

```

## File: views\express_checkout_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="payment.express_checkout" name="Payment Express Checkout">
        <!-- Parameters description:
            - reference_prefix: The custom prefix to compute the full transaction reference.
            - amount: The amount to pay.
            - minor_amount: The amount to pay in the minor units of its currency.
            - currency: The currency of the payment, as a `res.currency` record.
            - providers_sudo: The compatible providers, as a sudoed `payment.provider` recordset.
            - merchant_name: The merchant name.
            - payment_method_unknown_id: The ID of the "Unknown" payment method record, to use as
                                         the generic express checkout method.
            - payment_access_token: The access token used to authenticate the partner. Since this
                                    template is loaded in the shopping cart, this parameter is
                                    called `payment_access_token` to prevent mixing up with the
                                    `access_token` used for abandoned carts.
            - shipping_info_required: Whether the shipping information is required or not.
            - transaction_route: The route used to create a transaction when the user clicks Pay.
            - shipping_address_update_route: The route where available carriers are computed based
                                             on the (partial) shipping information available.
                                             Optional.
            - express_checkout_route: The route where the billing and shipping information are sent.
            - landing_route: The route the user is redirected to after the transaction.
            - payment_access_token: The access token used to authenticate the partner. Since this
                                    template is loaded in the shopping cart, this parameter is
                                    called `payment_access_token` to prevent mixing up with the
                                    `access_token` used for abandoned carts.
        -->
        <form name="o_payment_express_checkout_form" class="container"
              t-att-data-reference-prefix="reference_prefix"
              t-att-data-amount="amount"
              t-att-data-minor-amount="minor_amount"
              t-att-data-currency-id="currency and currency.id"
              t-att-data-currency-name="currency.name.lower()"
              t-att-data-merchant-name="merchant_name"
              t-att-data-partner-id="partner_id"
              t-att-data-payment-method-unknown-id="payment_method_unknown_id"
              t-att-data-access-token="payment_access_token"
              t-att-data-shipping-info-required="shipping_info_required"
              t-att-data-delivery-amount="delivery_amount"
              t-att-data-transaction-route="transaction_route"
              t-att-data-shipping-address-update-route="shipping_address_update_route"
              t-att-data-express-checkout-route="express_checkout_route"
              t-att-data-landing-route="landing_route"
        >
            <t t-set="provider_sudo" t-value="providers_sudo[:1]"/>
            <t t-set="express_checkout_form_xml_id"
               t-value="provider_sudo.express_checkout_form_view_id.xml_id"
            />
            <t t-if="express_checkout_form_xml_id">
                <t t-call="{{express_checkout_form_xml_id}}"/>
            </t>
        </form>
    </template>

</odoo>

```

## File: views\payment_form_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="payment.form" name="Payment Form">
        <!-- Form customization parameters:
            - mode: The operation mode of the form: `payment` or `validation`; default: `payment`.
            - allow_token_selection: Whether tokens can be selected for payment or assignation
                                     (through the `assign_token_route` parameter); default: `True`.
            - allow_token_deletion: Whether tokens can be deleted (archived); default: `False`.
            - default_token_id: The id of the token that should be pre-selected; default: `None`.
            - show_tokenize_input_mapping: For each provider, whether the tokenization checkbox is
                                           shown; used only in `payment` mode.
            - display_submit_button: Whether the submit button is displayed; default: `True`.
            - submit_button_label: The label of the submit button; default: 'Pay'/'Save'.
        -->
        <!-- Payment context:
            - reference_prefix: The custom prefix to compute the full transaction reference.
            - amount: The amount to pay.
            - currency: The currency of the payment, as a `res.currency` record.
            - partner_id: The id of the partner on behalf of whom the payment should be made.
            - providers_sudo: The compatible providers, as a sudoed `payment.provider` recordset.
            - payment_methods_sudo: The compatible payment methods, as a sudoed `payment.method`
                                    recordset.
            - tokens_sudo: The available payment tokens, as a sudoed `payment.token` recordset.
            - transaction_route: The route to call to create the transaction.
            - assign_token_route: The route to call to assign a new or existing token to a record.
            - landing_route: The route the user is redirected to after payment.
            - access_token: The access token used to authenticate the partner.
        -->
        <t t-set="mode" t-value="mode or 'payment'"/>
        <t t-set="allow_token_selection"
           t-value="True if allow_token_selection is None else allow_token_selection"
        />
        <t t-set="allow_token_deletion" t-value="allow_token_deletion or False"/>
        <t t-set="selected_token_id"
           t-value="allow_token_selection and (default_token_id or tokens_sudo[:1].id)"
        />
        <t t-set="selected_method_id"
           t-value="not selected_token_id
                    and len(payment_methods_sudo) == 1
                    and payment_methods_sudo[:1].id"
        />
        <t t-set="collapse_payment_methods"
           t-value="tokens_sudo and allow_token_selection and payment_methods_sudo"
        />
        <t t-set="display_submit_button"
           t-value="True if display_submit_button is None else display_submit_button"
        />
        <t t-set="pay_label">Pay</t> <!-- Allow translating the label. -->
        <t t-set="save_label">Save</t> <!-- Allow translating the label. -->
        <t t-set="submit_button_label"
           t-value="submit_button_label or (pay_label if mode == 'payment' else save_label)"
        />
        <form t-if="payment_methods_sudo or tokens_sudo" id="o_payment_form"
              class="o_payment_form"
              t-att-data-mode="mode"
              t-att-data-reference-prefix="reference_prefix"
              t-att-data-amount="amount"
              t-att-data-currency-id="currency and currency.id"
              t-att-data-partner-id="partner_id"
              t-att-data-transaction-route="transaction_route"
              t-att-data-assign-token-route="assign_token_route"
              t-att-data-landing-route="landing_route"
              t-att-data-access-token="access_token"
        >
            <div id="o_payment_form_options" class="d-flex flex-column gap-3">
                <!-- === Payment tokens === -->
                <div t-if="tokens_sudo">
                    <!-- === Header === -->
                    <h4 id="o_payment_tokens_heading" class="fs-6 small text-uppercase fw-bolder">
                        Your payment methods
                    </h4>
                    <!-- === Body === -->
                    <ul class="list-group">
                        <t t-foreach="tokens_sudo" t-as="token_sudo">
                            <li name="o_payment_option"
                                t-att-class="'list-group-item d-flex flex-column gap-2 py-3'
                                             + (' o_outline' if allow_token_selection else '')"
                            >
                                <t t-call="payment.token_form">
                                    <t t-set="is_selected"
                                       t-value="token_sudo.id == selected_token_id"
                                    />
                                </t>
                            </li>
                        </t>
                    </ul>
                </div>
                <!-- === Payment methods === -->
                <div t-if="payment_methods_sudo"
                     id="o_payment_methods"
                     t-att-class="'collapse' if collapse_payment_methods else ''"
                >
                    <!-- === Header === -->
                    <h4 class="fs-6 small text-uppercase fw-bolder">
                        <t t-if="not collapse_payment_methods">Choose a payment method</t>
                        <t t-else="">Other payment methods</t>
                        <t t-call="payment.availability_report_button"/>
                    </h4>
                    <!-- === Body === -->
                    <ul class="list-group">
                        <t t-foreach="payment_methods_sudo" t-as="pm_sudo">
                            <li name="o_payment_option"
                                class="list-group-item d-flex flex-column gap-2 py-3 o_outline"
                            >
                                <t t-call="payment.method_form">
                                    <t t-set="is_selected"
                                       t-value="pm_sudo.id == selected_method_id"
                                    />
                                </t>
                            </li>
                        </t>
                    </ul>
                </div>
            </div>
            <div class="d-flex justify-content-end flex-column flex-md-row gap-2 my-2">
                <!-- === Expand payment methods button === -->
                <button t-if="collapse_payment_methods"
                        name="o_payment_expand_button"
                        type="button"
                        href="#o_payment_methods"
                        class="btn btn-link"
                        data-bs-toggle="collapse"
                >
                    Choose another method <i class="oi oi-arrow-down"/>
                </button>
                <!-- === Submit button === -->
                <t t-if="display_submit_button" t-call="payment.submit_button"/>
            </div>
            <!-- === Availability report === -->
            <t t-call="payment.availability_report"/>
        </form>
        <t t-else="" t-call="payment.no_pms_available_warning"/>
    </template>

    <template id="payment.token_form" name="Payment Token Form">
        <!-- Parameters description:
            - token_sudo: The token to display, as a sudoed `payment.token` recordset.
            - allow_token_selection: Whether tokens can be selected for payment or assignation (if
                                     the `assign_route` parameter is provided); default: `True`.
            - is_selected: Whether the radio button of the token should be checked.
        -->
        <t t-set="provider_sudo" t-value="token_sudo.provider_id"/>
        <t t-set="is_test" t-value="provider_sudo.state == 'test'"/>
        <t t-set="is_unpublished" t-value="not provider_sudo.is_published"/>
        <t t-set="inline_form_xml_id" t-value="provider_sudo.token_inline_form_view_id.xml_id"/>
        <div class="d-flex gap-3 align-items-start align-items-md-center">
            <!-- === Delete button === -->
            <button t-if="allow_token_deletion"
                    name="o_payment_delete_token"
                    t-att-class="'btn btn-link px-2 py-0 lh-lg z-1'
                                 + (' d-none' if mode != 'validation' else '')"
                    t-attf-data-linked-radio="o_payment_token_{{token_sudo.id}}"
            >
                <i class="fa fa-trash"
                   title="Delete payment method"
                   data-bs-toggle="tooltip"
                   data-bs-placement="top"
                   data-bs-delay="0"
                />
            </button>
            <div class="row flex-column flex-md-row flex-grow-1 gap-lg-3 align-items-start
                        align-items-md-center"
            >
                <div class="col col-lg-5">
                    <div t-att-class="'form-check mb-0'
                                      + (' ps-0' if not allow_token_selection else '')">
                        <!-- === Radio button === -->
                        <input t-attf-id="o_payment_token_{{token_sudo.id}}"
                               name="o_payment_radio"
                               type="radio"
                               t-att-checked="is_selected"
                               t-att-disabled="not allow_token_selection"
                               t-att-class="'form-check-input'
                                            + (' d-none' if not allow_token_selection else '')"
                               data-payment-option-type="token"
                               t-att-data-payment-option-id="token_sudo.id"
                               t-att-data-provider-code="token_sudo.provider_id._get_code()"
                               t-att-data-provider-id="token_sudo.provider_id.id"
                        />
                        <div class="d-flex align-items-center flex-wrap gap-2">
                            <!-- === Token label === -->
                            <label t-out="token_sudo.payment_method_id.name"
                                   class="o_payment_option_label text-break"
                                   t-attf-for="o_payment_token_{{token_sudo.id}}"
                            />
                            <div class="d-flex flex-nowrap gap-2">
                                <!-- === "Unpublished" icon === -->
                                <t t-if="is_unpublished" t-call="payment.form_icon">
                                    <t t-set="icon_name" t-value="'eye-slash'"/>
                                    <t t-set="color_name" t-value="'danger'"/>
                                    <t t-set="title" t-value="'Unpublished'"/>
                                </t>
                                <!-- === "Test mode" icon === -->
                                <t t-if="is_test" t-call="payment.form_icon">
                                    <t t-set="icon_name" t-value="'exclamation-triangle'"/>
                                    <t t-set="color_name" t-value="'warning'"/>
                                    <t t-set="title" t-value="'Test mode'"/>
                                </t>
                            </div>
                        </div>
                    </div>
                </div>
                <!-- === Token name (payment details) === -->
                <div class="col">
                    <p t-out="token_sudo.display_name"
                       t-att-class="'mb-0 small fw-bold text-break'
                                    + (' ms-4 ms-md-0' if allow_token_selection else '')"
                    />
                </div>
                <!-- === Provider name (only for desktop and tablet) === -->
                <t t-set="hide_secured_by" t-value="False"/>
                <div class="col d-none d-md-block">
                    <p name="o_payment_secured_by_desktop" t-att-class="'mb-0 small text-600'
                                    + (' ms-4 ms-md-0' if allow_token_selection else '')
                                    + (' d-none' if hide_secured_by else '')"
                    >
                        <span><i class="fa fa-lock"/> Secured by</span>
                        <span t-out="dict(provider_sudo._fields['code']._description_selection(
                                         provider_sudo.env
                                     ))[provider_sudo.code]"
                              class="text-break"
                        />
                    </p>
                </div>
            </div>
            <!-- === Payment method logo === -->
            <div t-call="payment.form_logo">
                <t t-set="logo_pm_sudo" t-value="token_sudo.payment_method_id"/>
            </div>
        </div>
        <!-- === Inline form === -->
        <div t-if="inline_form_xml_id"
             name="o_payment_inline_form"
             class="position-relative d-none"
        >
            <t t-call="{{inline_form_xml_id}}"/>
        </div>
        <!-- === Provider name (only for mobile) === -->
        <p name="o_payment_secured_by_mobile"
           t-att-class="'align-self-end d-block d-md-none mb-0 small text-600'
                        + (' d-none' if hide_secured_by else '')"
        >
            <span><i class="fa fa-lock"/> Secured by</span>
            <span t-out="dict(provider_sudo._fields['code']._description_selection(
                             provider_sudo.env
                         ))[provider_sudo.code]"
                  class="text-break"
            />
        </p>
    </template>

    <template id="payment.method_form" name="Payment Method Form">
        <!-- Parameters description:
            - pm_sudo: The payment method to display, as a sudoed `payment.method` recordset.
            - is_selected: Whether the radio button of the payment method should be checked.
        -->
        <t t-set="provider_sudo"
           t-value="pm_sudo.provider_ids.filtered(lambda p: p in providers_sudo)[:1]"
        />
        <t t-set="is_test" t-value="provider_sudo.state == 'test'"/>
        <t t-set="is_unpublished" t-value="not provider_sudo.is_published"/>
        <t t-set="pms_to_display_sudo" t-value="pm_sudo.brand_ids or pm_sudo"/>
        <t t-set="inline_form_xml_id" t-value="provider_sudo.inline_form_view_id.xml_id"/>
        <div class="d-flex flex-wrap justify-content-between align-items-center gap-2 mb-0 p-0"
             for="o_payment_radio"
        >
            <div class="form-check d-flex flex-grow-1 flex-wrap mb-0">
                <div class="d-flex justify-content-between align-items-center gap-2 flex-wrap w-100">
                    <!-- === Radio button === -->
                    <input t-attf-id="o_payment_method_{{pm_sudo.id}}"
                           name="o_payment_radio"
                           type="radio"
                           t-att-checked="is_selected"
                           class="form-check-input position-absolute mt-0"
                           data-payment-option-type="payment_method"
                           t-att-data-payment-option-id="pm_sudo.id"
                           t-att-data-payment-method-code="pm_sudo.code"
                           t-att-data-provider-id="provider_sudo.id"
                           t-att-data-provider-code="provider_sudo._get_code()"
                           t-att-data-provider-state="provider_sudo.state"
                    />
                    <div class="d-flex gap-2 flex-grow-1 me-auto">
                        <!-- === Method label === -->
                        <label t-out="pm_sudo.name"
                               class="o_payment_option_label mb-0 text-break"
                               t-attf-for="o_payment_method_{{pm_sudo.id}}"
                        />
                        <div class="d-flex flex-nowrap gap-2 mt-1">
                            <!-- === "Unpublished" icon === -->
                            <t t-if="is_unpublished" t-call="payment.form_icon">
                                <t t-set="icon_name" t-value="'eye-slash'"/>
                                <t t-set="color_name" t-value="'danger'"/>
                                <t t-set="title" t-value="'Unpublished'"/>
                            </t>
                            <!-- === "Test mode" icon === -->
                            <t t-if="is_test" t-call="payment.form_icon">
                                <t t-set="icon_name" t-value="'exclamation-triangle'"/>
                                <t t-set="color_name" t-value="'warning'"/>
                                <t t-set="title" t-value="'Test mode'"/>
                            </t>
                        </div>
                    </div>
                    <div class="gap-1 flex-wrap d-flex">
                        <!-- === Payment method logos === -->
                        <t t-set="pm_index" t-value="0"/>
                        <t t-foreach="pms_to_display_sudo" t-as="pm_to_display_sudo">
                            <t t-if="pm_index &lt; 4" t-call="payment.form_logo">
                                <t t-set="logo_pm_sudo" t-value="pm_to_display_sudo"/>
                            </t>
                            <t t-set="pm_index" t-value="pm_index + 1"/>
                        </t>
                    </div>
                </div>
            </div>
            <!-- === Help message === -->
            <div t-if="not is_html_empty(provider_sudo.pre_msg)"
                 class="w-100 mb-0 ms-4 small text-600"
            >
                <t t-out="provider_sudo.pre_msg"/>
            </div>
        </div>
        <!-- === Inline form === -->
        <div name="o_payment_inline_form" class="position-relative d-none">
            <t t-if="inline_form_xml_id and provider_sudo._should_build_inline_form(
                         is_validation=mode == 'validation'
                     )"
               t-call="{{inline_form_xml_id}}"
            >
                <t t-set="provider_id" t-value="provider_sudo.id"/>
            </t>
            <div class="d-flex flex-column flex-md-row align-md-items-center justify-content-between
                        gap-2 mt-2"
            >
                <!-- === Tokenization checkbox === -->
                <div t-if="mode == 'payment'
                           and pm_sudo.support_tokenization
                           and show_tokenize_input_mapping[provider_sudo.id]"
                     name="o_payment_tokenize_container"
                     class="o-checkbox form-check m-0"
                >
                    <label>
                        <input name="o_payment_tokenize_checkbox"
                               type="checkbox"
                               class="form-check-input"
                        />
                        <small class="text-600">Save my payment details</small>
                    </label>
                </div>
                <!-- === Provider name === -->
                <t t-set="hide_secured_by" t-value="False"/>
                <p name="o_payment_secured_by"
                   t-att-class="'align-self-end mb-0 ms-auto small text-600'
                                + (' d-none' if hide_secured_by else '')"
                >
                    <span><i class="fa fa-lock"/> Secured by</span>
                    <span t-out="dict(provider_sudo._fields['code']._description_selection(
                                     provider_sudo.env
                                 ))[provider_sudo.code]"
                          class="text-break"
                    />
                </p>
            </div>
        </div>
    </template>

    <template id="payment.form_icon" name="Form Icon">
        <!-- Parameters description:
            - icon_name: The name of the FontAwesome icon.
            - color_name: The class name of the color (`warning`, `danger`...).
            - title: The title to display on hover.
        -->
        <i t-attf-class="fa fa-{{icon_name}} text-{{color_name}} position-relative z-1"
           t-att-title="title"
           data-bs-toggle="tooltip"
           data-bs-placement="top"
           data-bs-delay="0"
        />
    </template>

    <template id="payment.form_logo" name="Form Logo">
        <!-- Parameters description:
            - logo_pm_sudo: The payment method whose logo to display, as a sudoed `payment.method`
                            record.
        -->
        <span t-field="logo_pm_sudo.image_payment_form"
              t-options="{'widget': 'image', 'alt-field': 'name'}"
              class="position-relative d-block rounded overflow-hidden z-1 shadow-sm"
              t-att-title="logo_pm_sudo.name"
              data-bs-toggle="tooltip"
              data-bs-placement="top"
              data-bs-delay="0"
        />
    </template>

    <template id="payment.submit_button" name="Submit Button">
        <!-- Parameters description:
            - label: The label of the submit button.
        -->
        <button name="o_payment_submit_button"
                type="submit"
                t-out="submit_button_label"
                class="btn btn-primary w-100 w-md-auto ms-auto px-5"
                disabled="true"
        />
    </template>

    <template id="payment.no_pms_available_warning">
        <div class="alert alert-warning mt-2">
            <div>
                <strong>No payment method available</strong>
            </div>
            <div t-if="request.env.is_system()" class="mt-2">
                <p t-if="providers_sudo">
                    None is configured for:
                    <t t-out="request.env.company.country_id.name"/>,
                    <t t-out="request.env.company.currency_id.name"/>.
                </p>
                <p t-else="">
                    No payment providers are configured.
                </p>
                <a
                    t-if="request.env.company.country_id.is_stripe_supported_country
                          and not providers_sudo"
                    name="activate_stripe"
                    href="/odoo/action-payment.action_activate_stripe"
                    role="button"
                    class="btn btn-primary me-2"
                > ACTIVATE STRIPE </a>
                <a
                    t-if="availability_report"
                    role="button"
                    class="btn-link alert-warning me-2"
                    data-bs-toggle="collapse"
                    href="#payment_availability_report"
                >
                    <strong><i class="fa fa-file-text"/> Show availability report</strong>
                </a>
                <a
                    t-if="not providers_sudo"
                    role="button"
                    type="action"
                    class="btn-link alert-warning me-2"
                    href="/odoo/action-payment.action_payment_provider"
                >
                    <strong><i class="oi oi-arrow-right"/> Payment Providers</strong>
                </a>
                <a
                    t-else=""
                    role="button"
                    type="action"
                    class="btn-link alert-warning"
                    href="/odoo/action-payment.action_payment_method"
                >
                    <strong><i class="oi oi-arrow-right"/> Payment Methods</strong>
                </a>
            </div>
            <div t-else="" class="mt-2">
                If you believe that it is an error, please contact the website administrator.
            </div>
        </div>
        <!-- === Availability report === -->
        <t t-call="payment.availability_report"/>
    </template>

    <template id="payment.availability_report_button">
        <a
            t-if="request.env.user._is_system() and availability_report"
            role="button"
            data-bs-toggle="collapse"
            href="#payment_availability_report"
            aria-expanded="false"
            aria-controls="payment_availability_report"
        >
            <i class="fa fa-bug"/>
        </a>
    </template>

    <template id="payment.availability_report">
        <div
            t-if="request.env.user._is_system() and availability_report"
            id="payment_availability_report"
            class="collapse"
        >
            <h4 class="fs-6 text-uppercase fw-bolder"> Availability report </h4>
                <h6 class="mt-3 text-uppercase fw-normal fs-6"> Payment providers </h6>
                <t t-call="payment.availability_report_records">
                    <t t-set="records" t-value="availability_report.get('providers')"/>
                </t>
                <h6 class="mt-3 text-uppercase fw-normal fs-6"> Payment methods </h6>
                <t t-call="payment.availability_report_records">
                    <t t-set="records" t-value="availability_report.get('payment_methods')"/>
                </t>
        </div>
    </template>

    <template id="availability_report_records">
        <!-- Parameters description:
            - records: The records to list in the availability report, as a dict with the structure
                       {record: {'available': bool, 'reason': str, supported_providers: list}}.
        -->
        <ul class="list-group">
            <t t-foreach="records" t-as="r">
                <t t-set="available" t-value="records[r]['available']"/>
                <li class="list-group-item ps-0">
                    <div class="d-flex gap-2">
                        <div class="ms-2">
                            <i
                                t-attf-class="fa fa-fw fa-{{'check-circle' if available else 'times-circle'}}"
                                t-attf-style="color: {{'green' if available else 'red'}};"
                            />
                        </div>
                        <div>
                            <p class="lead mb-0">
                                <span class="fw-normal"><t t-out="r.name"/></span>
                                <span class="text-muted">(ID: <t t-out="r.id"/>)</span>
                            </p>
                            <t t-if="not available">
                                <p class="mb-0 fw-light">
                                    Reason: <t t-out="records[r]['reason']"/>
                                </p>
                            </t>
                            <t t-if="r._name == 'payment.method' and 'supported_providers' in records[r]">
                                <p class="mb-0 fw-light">
                                    Supported providers:
                                     <t t-foreach="records[r]['supported_providers']" t-as="p">
                                        <span t-attf-class="text-{{'success' if p[1] else 'danger'}}">
                                            <t t-out="p[0].name"/>
                                        </span>
                                        <t t-if="records[r]['supported_providers'][-1] != p">,</t>
                                    </t>
                                </p>
                            </t>
                        </div>
                    </div>
                </li>
            </t>
        </ul>
    </template>

</odoo>

```

## File: views\payment_method_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_method_form" model="ir.ui.view">
        <field name="name">payment.method.form</field>
        <field name="model">payment.method</field>
        <field name="arch" type="xml">
            <form string="Payment Method">
                <sheet>
                    <field name="is_primary" invisible="True"/>
                    <field name="image" widget="image" class="oe_avatar"/>
                    <div class="oe_title">
                        <h1><field name="name" placeholder="Name"/></h1>
                    </div>
                    <group>
                        <field name="code" readonly="id" groups="base.group_no_one"/>
                        <field name="primary_payment_method_id" invisible="is_primary"/>
                        <field name="active"/>
                        <label for="supported_country_ids"/>
                        <div>
                            <field name="supported_country_ids"
                                   class="oe_inline"
                                   widget="many2many_tags"
                                   readonly="1"
                            />
                            <span class="oe_inline text-muted" invisible="supported_country_ids">
                                All countries are supported.
                            </span>
                        </div>
                        <label for="supported_currency_ids"/>
                        <div>
                            <field name="supported_currency_ids"
                                   class="oe_inline"
                                   widget="many2many_tags"
                                   readonly="1"
                            />
                            <span class="oe_inline text-muted" invisible="supported_currency_ids">
                                All currencies are supported.
                            </span>
                        </div>
                    </group>
                    <notebook>
                        <page string="Providers" name="providers">
                            <field name="provider_ids" readonly="1">
                                <list decoration-muted="state == 'disabled'" editable="bottom">
                                    <field name="name"/>
                                    <field name="state"/>
                                </list>
                            </field>
                        </page>
                        <page string="Brands" name="brands" invisible="not is_primary">
                            <field name="brand_ids"/>
                        </page>
                        <page string="Configuration"
                              name="configuration"
                              groups="base.group_no_one"
                        >
                            <div class="alert alert-warning" role="alert">
                                <i class="fa fa-exclamation-triangle"/> These properties are set to
                                match the behavior of providers and that of their integration with
                                Odoo regarding this payment method. Any change may result in errors
                                and should be tested on a test database first.
                            </div>
                            <group>
                                <field name="support_tokenization"/>
                                <field name="support_express_checkout"/>
                                <field name="support_refund" />
                                <field name="supported_country_ids"
                                       widget="many2many_tags"
                                       placeholder="Select countries. Leave empty to allow any."
                                />
                                <field name="supported_currency_ids"
                                       widget="many2many_tags"
                                       placeholder="Select currencies. Leave empty to allow any."
                                />
                                <field name="provider_ids"
                                       string="Supported by"
                                       widget="many2many_tags"
                                />
                            </group>
                        </page>
                    </notebook>
                </sheet>
            </form>
        </field>
    </record>

    <record id="payment_method_tree" model="ir.ui.view">
        <field name="name">payment.method.list</field>
        <field name="model">payment.method</field>
        <field name="arch" type="xml">
            <list multi_edit="True" decoration-muted="not active">
                <field name="sequence" widget="handle"/>
                <field name="name"/>
                <field name="active" widget="boolean_toggle"/>
            </list>
        </field>
    </record>

    <record id="payment_method_kanban" model="ir.ui.view">
        <field name="name">payment.method.kanban</field>
        <field name="model">payment.method</field>
        <field name="priority">1</field>
        <field name="arch" type="xml">
            <kanban>
                <templates>
                    <t t-name="card" class="flex-row">
                        <field name="name" class="fw-bolder"/>
                        <field name="image" widget="image" class="ms-auto"/>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>


     <record id="payment_method_search" model="ir.ui.view">
        <field name="name">payment.method.search</field>
        <field name="model">payment.method</field>
        <field name="arch" type="xml">
            <search>
                <field name="name" string="Name"/>
                <filter name="available_pms"
                        string="Available methods"
                        domain="[('provider_ids.state', '!=', 'disabled')]"
                />
            </search>
        </field>
    </record>

    <record id="action_payment_method" model="ir.actions.act_window">
        <field name="name">Payment Methods</field>
        <field name="res_model">payment.method</field>
        <field name="view_mode">list,kanban,form</field>
        <field name="domain">[('is_primary', '=', True)]</field>
        <field name="context">{'active_test': False, 'search_default_available_pms': 1}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                No payment methods found for your payment providers.
            </p>
            <p>
                <a type="action" class="text-primary" name="%(payment.action_payment_provider)d">
                    <i class="oi oi-arrow-right me-1"/> Configure a payment provider
                </a>
            </p>
        </field>
    </record>

</odoo>

```

## File: views\payment_provider_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_provider_form" model="ir.ui.view">
        <field name="name">payment.provider.form</field>
        <field name="model">payment.provider</field>
        <field name="arch" type="xml">
            <form string="Payment provider">
                <!-- Prevent considering the field readonly and thus allow writing on it. -->
                <field name="is_published" invisible="1"/>
                <sheet>
                    <!-- === Stat Buttons === -->
                    <div class="oe_button_box" name="button_box"
                         invisible="module_state != 'installed'">
                        <button name="action_toggle_is_published"
                                invisible="not is_published"
                                class="oe_stat_button"
                                type="object"
                                icon="fa-globe">
                            <div class="o_stat_info o_field_widget">
                                <span class="o_stat_text text-success">Published</span>
                            </div>
                        </button>
                        <button name="action_toggle_is_published"
                                invisible="is_published"
                                class="oe_stat_button"
                                type="object"
                                icon="fa-eye-slash">
                            <div class="o_stat_info o_field_widget">
                                <span class="o_stat_text text-danger">Unpublished</span>
                            </div>
                        </button>
                    </div>
                    <field name="image_128" widget="image" class="oe_avatar"
                           readonly="module_state != 'installed'"/>
                    <widget name="web_ribbon" title="Disabled" bg_color="text-bg-danger" invisible="module_state != 'installed' or state != 'disabled'"/>
                    <widget name="web_ribbon" title="Test Mode" bg_color="text-bg-warning" invisible="module_state != 'installed' or state != 'test'"/>
                    <div class="oe_title">
                        <h1><field name="name" placeholder="Name"/></h1>
                        <div invisible="module_state == 'installed' or not module_id">
                            <a invisible="not module_to_buy" href="https://odoo.com/pricing?utm_source=db&amp;utm_medium=module" target="_blank" class="btn btn-info" role="button">Upgrade</a>
                            <button invisible="module_to_buy" type="object" class="btn btn-primary" name="button_immediate_install" string="Install"/>
                        </div>
                    </div>
                    <div id="provider_creation_warning" invisible="id" class="alert alert-warning" role="alert">
                        <strong>Warning</strong> Creating a payment provider from the <em>CREATE</em> button is not supported.
                        Please use the <em>Duplicate</em> action instead.
                    </div>
                    <group>
                        <group name="payment_state" invisible="module_state not in ('installed', False)">
                            <field name="code" groups="base.group_no_one" readonly="id"/>
                            <field name="state" widget="radio"/>
                            <field name="company_id" groups="base.group_multi_company" options='{"no_open":True}'/>
                        </group>
                    </group>
                    <notebook invisible="module_id and module_state != 'installed'">
                        <page string="Credentials" name="credentials" invisible="code == 'none'">
                            <group name="provider_credentials"/>
                        </page>
                        <page string="Configuration" name="configuration">
                            <group name="provider_config">
                                <group string="Payment Form" name="payment_form">
                                    <field name="payment_method_ids"
                                           string="Payment Methods"
                                           domain="[('is_primary', '=', True)]"
                                           readonly="True"
                                           invisible="state == 'disabled'"
                                           widget="many2many_tags"
                                    />
                                    <div colspan="2">
                                        <a type="object"
                                           name="action_view_payment_methods"
                                           class="btn btn-link"
                                           role="button"
                                           invisible="state == 'disabled'"
                                        >
                                            <i class="oi oi-fw o_button_icon oi-arrow-right"/>
                                            Enable Payment Methods
                                        </a>
                                    </div>
                                    <field name="allow_tokenization" invisible="not support_tokenization"/>
                                    <field name="capture_manually" invisible="not support_manual_capture"/>
                                    <field name="allow_express_checkout" invisible="not support_express_checkout"/>
                                </group>
                                <group string="Availability" name="availability">
                                    <field name="maximum_amount"/>
                                    <label for="available_currency_ids"/>
                                    <!-- Use `o_row` to allow placing a button next to the field in overrides. -->
                                    <div name="available_currencies" class="o_row">
                                        <field name="available_currency_ids"
                                               widget="many2many_tags"
                                               placeholder="Select currencies. Leave empty not to restrict any."
                                               options="{'no_create': True}"/>
                                    </div>
                                    <field name="available_country_ids"
                                           widget="many2many_tags"
                                           placeholder="Select countries. Leave empty to make available everywhere."
                                           options="{'no_create': True}"/>
                                </group>
                                <group string="Payment Followup" name="payment_followup" invisible="1"/>
                            </group>
                        </page>
                        <page string="Messages"
                            name="messages"
                            invisible="module_id and module_state != 'installed'">
                            <group>
                                <field name="pre_msg"/>
                                <field name="pending_msg"/>
                                <field name="auth_msg" invisible="not support_manual_capture"/>
                                <field name="done_msg"/>
                                <field name="cancel_msg"/>
                            </group>
                        </page>
                    </notebook>
                </sheet>
            </form>
        </field>
    </record>

    <record id="payment_provider_list" model="ir.ui.view">
        <field name="name">payment.provider.list</field>
        <field name="model">payment.provider</field>
        <field name="arch" type="xml">
            <list string="Payment Providers" create="false">
                <field name="sequence" widget="handle"/>
                <field name="name"/>
                <field name="code" groups="base.group_no_one"/>
                <field name="state"/>
                <field name="available_country_ids" widget="many2many_tags" optional="hide"/>
                <field name="company_id" groups="base.group_multi_company" optional="show"/>
            </list>
        </field>
    </record>

    <record id="payment_provider_kanban" model="ir.ui.view">
        <field name="name">payment.provider.kanban</field>
        <field name="model">payment.provider</field>
        <field name="arch" type="xml">
            <kanban create="false" quick_create="false">
                <field name="is_published"/>
                <field name="module_id"/>
                <field name="module_state"/>
                <field name="module_to_buy"/>
                <templates>
                    <t t-name="card" class="flex-row">
                        <t t-set="installed" t-value="!record.module_id.value || (record.module_id.value &amp;&amp; record.module_state.raw_value === 'installed')"/>
                        <t t-set="to_buy" t-value="record.module_to_buy.raw_value === true"/>
                        <t t-set="is_disabled" t-value="record.state.raw_value=='disabled'"/>
                        <t t-set="is_published" t-value="record.is_published.raw_value === true"/>
                        <t t-set="to_upgrade" t-value="!installed and to_buy"/>
                        <aside>
                            <field type="open"
                                 name="image_128" widget="image"
                                 class="mb-0 o_image_64_max"
                                 alt="provider"/>
                        </aside>
                        <main class="ms-2">
                            <field name="name" class="mb-0 fw-bold fs-4"/>
                            <div class="d-flex">
                                <t t-if="installed">
                                    <field name="state"
                                        widget="label_selection"
                                        options="{'classes': {'enabled': 'success', 'test': 'warning', 'disabled' : 'light'}}"/>
                                    <t t-if="!is_disabled">
                                        <div>
                                            <span t-if="is_published"
                                                class="badge text-bg-success ms-1">
                                                Published
                                            </span>
                                            <span t-else=""
                                                class="badge text-bg-info ms-1">
                                                Unpublished
                                            </span>
                                        </div>
                                    </t>
                                </t>
                                <span t-if="to_upgrade" class="badge text-bg-primary">Enterprise</span>
                            </div>
                            <footer>
                                <button t-if="!installed and !selection_mode and !to_buy" type="object" class="btn btn-sm btn-primary ms-auto" name="button_immediate_install">Install</button>
                                <button t-if="installed and is_disabled and !selection_mode" type="edit" class="btn btn-sm btn-secondary ms-auto">Activate</button>
                                <button t-if="!installed and to_buy" href="https://odoo.com/pricing?utm_source=db&amp;utm_medium=module" target="_blank" class="btn btn-sm btn-primary ms-auto">Upgrade</button>
                            </footer>
                        </main>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="payment_provider_search" model="ir.ui.view">
        <field name="name">payment.provider.search</field>
        <field name="model">payment.provider</field>
        <field name="arch" type="xml">
            <search>
                <field name="name" string="provider" filter_domain="[('name', 'ilike', self)]"/>
                <field name="payment_method_ids"
                       string="payment method"
                       context="{'active_test': False}"
                       filter_domain="[
                            '|',
                            ('payment_method_ids.name', 'ilike', self),
                            ('payment_method_ids.code', 'ilike', self),
                       ]"
                />
                <filter name="provider_installed" string="Installed" domain="[('module_state', '=', 'installed')]"/>
                <group expand="0" string="Group By">
                    <filter string="Provider" name="code" context="{'group_by': 'code'}"/>
                    <filter string="State" name="state" context="{'group_by': 'state'}"/>
                    <filter string="Company" name="company" context="{'group_by': 'company_id'}" groups="base.group_multi_company"/>
                </group>
            </search>
        </field>
    </record>

    <record id="action_payment_provider" model="ir.actions.act_window">
        <field name="name">Payment Providers</field>
        <field name="res_model">payment.provider</field>
        <field name="view_mode">kanban,list,form</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a new payment provider
            </p>
        </field>
    </record>

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
            <form string="Payment Tokens" create="false" edit="false">
                <sheet>
                    <field name="active" invisible="1"/>
                    <div class="oe_button_box" name="button_box">
                        <button class="oe_stat_button"
                                name="%(action_payment_transaction_linked_to_token)d"
                                type="action" icon="fa-money" string="Payments">
                        </button>
                    </div>
                    <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                    <group>
                        <group name="general_information">
                            <field name="payment_details"/>
                            <field name="payment_method_id"/>
                            <field name="partner_id" />
                        </group>
                        <group name="technical_information">
                            <field name="provider_id"/>
                            <field name="provider_ref"/>
                            <field name="company_id" groups="base.group_multi_company"/>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="payment_token_list" model="ir.ui.view">
        <field name="name">payment.token.list</field>
        <field name="model">payment.token</field>
        <field name="arch" type="xml">
            <list string="Payment Tokens" create="false">
                <field name="payment_details"/>
                <field name="partner_id"/>
                <field name="payment_method_id"/>
                <field name="provider_id"/>
                <field name="provider_ref"/>
                <field name="company_id" groups="base.group_multi_company" optional="show"/>
            </list>
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
                    <filter string="Provider" name="provider_id" context="{'group_by': 'provider_id'}"/>
                    <filter string="Partner" name="partner_id" context="{'group_by': 'partner_id'}"/>
                    <filter string="Company" name="company" context="{'group_by': 'company_id'}" groups="base.group_multi_company"/>
                </group>
            </search>
        </field>
    </record>

    <record id="action_payment_token" model="ir.actions.act_window">
        <field name="name">Payment Tokens</field>
        <field name="res_model">payment.token</field>
        <field name="view_mode">list,form</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                There is no token created yet.
            </p>
        </field>
    </record>

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
                    <button type="object" name="action_capture" invisible="state != 'authorized'" string="Capture Transaction" class="oe_highlight"/>
                    <button type="object" name="action_void" invisible="state != 'authorized'" string="Void Transaction"
                            confirm="Are you sure you want to void the authorized transaction? This action can't be undone."/>
                    <field name="state" widget="statusbar"/>
                </header>
                <sheet>
                    <div class="oe_button_box" name="button_box">
                        <button name="action_view_refunds"
                                type="object"
                                class="oe_stat_button"
                                icon="fa-money"
                                invisible="refunds_count == 0">
                            <field name="refunds_count" widget="statinfo" string="Refunds"/>
                        </button>
                    </div>
                    <group>
                        <group name="transaction_details">
                            <field name="reference"/>
                            <field name="source_transaction_id"
                                   invisible="not source_transaction_id"/>
                            <field name="amount"/>
                            <field name="currency_id" invisible="1"/>
                            <field name="payment_method_id"/>
                            <field name="provider_id"/>
                            <field name="company_id" groups="base.group_multi_company"/>
                            <!-- Used by some provider-specific views -->
                            <field name="provider_code" invisible="1"/>
                            <field name="provider_reference"/>
                            <field name="token_id" invisible="not token_id"/>
                            <field name="create_date"/>
                            <field name="last_state_change"/>
                            <field name="is_post_processed" groups="base.group_no_one"/>
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
                    <separator string="Child transactions" invisible="not child_transaction_ids"/>
                    <field name="child_transaction_ids" invisible="not child_transaction_ids"/>
                    <group string="Message" invisible="not state_message">
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
            <list string="Payment Transactions" create="false">
                <field name="reference"/>
                <field name="create_date"/>
                <field name="payment_method_id"/>
                <field name="provider_id"/>
                <field name="partner_id"/>
                <field name="partner_name"/>
                <!-- Needed to display the currency of the amounts -->
                <field name="currency_id" column_invisible="True"/>
                <field name="amount"/>
                <field name="state"/>
                <field name="company_id" groups="base.group_multi_company" optional="show"/>
            </list>
        </field>
    </record>

    <record id="payment_transaction_kanban" model="ir.ui.view">
        <field name="name">payment.transaction.kanban</field>
        <field name="model">payment.transaction</field>
        <field name="arch" type="xml">
            <kanban class="o_kanban_mobile" create="false">
                <field name="currency_id"/>
                <templates>
                    <t t-name="card">
                        <div class="d-flex">
                            <field name="reference" class="fw-bolder"/>
                            <field name="amount" class="ms-auto"/>
                        </div>
                        <field name="partner_name"/>
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
                <field name="provider_id"/>
                <field name="partner_id"/>
                <field name="partner_name"/>
                <group expand="1" string="Group By">
                    <filter string="Provider" name="provider_id" context="{'group_by': 'provider_id'}"/>
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
        <field name="view_mode">list,kanban,form</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_neutral_face">
                There are no transactions to show
            </p>
        </field>
    </record>

    <record id="action_payment_transaction_linked_to_token" model="ir.actions.act_window">
        <field name="name">Payment Transactions Linked To Token</field>
        <field name="res_model">payment.transaction</field>
        <field name="view_mode">list,form</field>
        <field name="domain">[('token_id','=', active_id)]</field>
        <field name="context">{'create': False}</field>
    </record>

</odoo>

```

## File: views\portal_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- Display of /payment/pay -->
    <template id="payment.pay">
        <!-- Parameters description:
            - reference_prefix: The custom prefix to compute the full transaction reference.
            - amount: The amount to pay.
            - currency: The currency of the payment, as a `res.currency` record.
            - partner_id: The id of the partner on behalf of whom the payment should be made.
            - payment_methods_sudo: The compatible payment methods, as a sudoed `payment.method`
                                    recordset.
            - tokens_sudo: The available payment tokens, as a sudoed `payment.token` recordset.
            - availability_report: The availability report of providers and payment methods.
            - res_company: The company in which the payment if made (for the company logo).
            - company_mismatch: Whether the user should make the payment in another company.
            - expected_company: The record of the company that the user should switch to.
            - partner_is_different: Whether the partner logged in is the one making the payment.
        -->
        <t t-call="portal.frontend_layout">
            <t t-set="page_title" t-value="'Payment'"/>
            <t t-set="additional_title"><t t-esc="page_title"/></t>
            <div class="wrap">
                <div class="container">
                    <!-- Portal breadcrumb -->
                    <t t-call="payment.portal_breadcrumb"/>
                    <!-- Payment page -->
                    <div class="row justify-content-center my-3">
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
                            <div t-elif="company_mismatch">
                                <t t-call="payment.company_mismatch_warning"/>
                            </div>
                            <t t-else="">
                                <div t-if="partner_is_different" class="alert alert-warning">
                                    <strong>Warning</strong> Make sure you are logged in as the
                                    correct partner before making this payment.
                                </div>
                                <div class="text-bg-light row row-cols-1 row-cols-md-2 mx-0 py-2
                                            rounded"
                                >
                                    <t t-call="payment.summary_item">
                                        <t t-set="name" t-value="'amount'"/>
                                        <t t-set="label">Amount</t>
                                        <t t-set="value" t-value="amount"/>
                                        <t t-set="options"
                                           t-value="{'widget': 'monetary', 'display_currency': currency}"
                                        />
                                    </t>
                                    <t t-call="payment.summary_item">
                                        <t t-set="name" t-value="'reference'"/>
                                        <t t-set="label">Reference</t>
                                        <t t-set="value" t-value="reference_prefix"/>
                                        <t t-set="include_separator" t-value="True"/>
                                    </t>
                                </div>
                                <div class="mt-4">
                                    <t t-call="payment.form"/>
                                </div>
                            </t>
                        </div>
                    </div>
                </div>
            </div>
        </t>
    </template>

    <template id="payment.company_mismatch_warning" name="Company Mismatch Warning">
        <!-- Parameters description:
            - expected_company: The record of the company that the user should switch to.
        -->
        <div class="row mr16">
            <div class="alert alert-warning col-lg-12 ms-3 me-3" role="alert">
                <p>
                    Please switch to company <t t-esc="expected_company.name"/> to make this
                    payment.
                </p>
            </div>
        </div>
    </template>

    <!-- Display of /my/payment_methods -->
    <template id="payment.payment_methods" name="Payment Methods">
        <t t-call="portal.frontend_layout">
            <t t-set="page_title" t-value="'Payment Methods'"/>
            <t t-set="additional_title"><t t-esc="page_title"/></t>
            <div class="wrap">
                <div class="container">
                    <!-- Portal breadcrumb -->
                    <t t-call="payment.portal_breadcrumb"/>
                    <!-- Payment methods page -->
                    <div class="row justify-content-center">
                        <div class="col-lg-7">
                            <t t-call="payment.form"/>
                        </div>
                    </div>
                </div>
            </div>
        </t>
    </template>

    <!-- Display of /payment/status -->
    <template id="payment.payment_status" name="Payment Status">
        <t t-call="portal.frontend_layout">
            <t t-set="page_title" t-value="'Payment Status'"/>
            <t t-set="additional_title"><t t-esc="page_title"/></t>
            <div class="wrap">
                <div class="container">
                    <!-- Portal breadcrumb -->
                    <t t-call="payment.portal_breadcrumb"/>
                    <!-- Payment status page -->
                    <div class="row justify-content-center my-3">
                        <div class="col-12 col-lg-8">
                            <div t-if="payment_not_found" class="text-center">
                                <p>Your payment is on its way!</p>
                                <p>
                                    You should receive an email confirming your payment within a few
                                    minutes.
                                </p>
                                <p>Don't hesitate to contact us if you don't receive it.</p>
                            </div>
                            <div t-else="" name="o_payment_status">
                                <t t-call="payment.state_header">
                                    <t t-set="is_processing" t-value="True"/>
                                </t>
                                <div class="text-bg-light row row-cols-1 row-cols-md-2 mx-0 mb-3
                                            py-2 rounded"
                                >
                                    <t t-call="payment.summary_item">
                                        <t t-set="name" t-value="'amount'"/>
                                        <t t-set="label">Amount</t>
                                        <t t-set="value" t-value="tx.amount"/>
                                        <t t-set="options"
                                           t-value="{
                                                        'widget': 'monetary',
                                                        'display_currency': tx.currency_id,
                                                    }"
                                        />
                                    </t>
                                    <t t-call="payment.summary_item">
                                        <t t-set="name" t-value="'reference'"/>
                                        <t t-set="label">Reference</t>
                                        <t t-set="value" t-value="tx.reference"/>
                                        <t t-set="include_separator" t-value="True"/>
                                    </t>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </t>
    </template>

    <!-- Display of /payment/confirmation -->
    <template id="payment.confirm">
        <!-- Parameters description:
            - tx: The transaction to display.
        -->
        <t t-call="portal.frontend_layout">
            <t t-set="page_title" t-value="'Payment Confirmation'"/>
            <t t-set="additional_title"><t t-esc="page_title"/></t>
            <t t-set="show_pm" t-value="tx.payment_method_code != 'unknown'"/>
            <div class="wrap">
                <div class="container">
                    <!-- Portal breadcrumb -->
                    <t t-call="payment.portal_breadcrumb"/>
                    <div class="row justify-content-center my-3">
                    <div class="col-12 col-lg-7">
                        <!-- Confirmation page -->
                        <div class="row">
                            <div class="col">
                                <t t-call="payment.state_header"/>
                            </div>
                        </div>
                        <div t-att-class="'text-bg-light row row-cols-1 mx-0 mb-3 py-2 rounded'
                                          + (' row-cols-md-4' if show_pm else ' row-cols-md-3')"
                        >
                            <t t-call="payment.summary_item">
                                <t t-set="name" t-value="'amount'"/>
                                <t t-set="label">Amount</t>
                                <t t-set="value" t-value="tx.amount"/>
                                <t t-set="options"
                                   t-value="{'widget': 'monetary', 'display_currency': tx.currency_id}"
                                />
                            </t>
                            <t t-call="payment.summary_item">
                                <t t-set="name" t-value="'reference'"/>
                                <t t-set="label">Reference</t>
                                <t t-set="value" t-value="tx.reference"/>
                                <t t-set="include_separator" t-value="True"/>
                            </t>
                            <t t-if="tx.payment_method_code != 'unknown'">
                               <t t-call="payment.summary_item">
                                    <t t-set="name" t-value="'method'"/>
                                    <t t-set="label">Payment Method</t>
                                    <t t-set="value" t-value="tx.payment_method_id.name"/>
                                    <t t-set="include_separator" t-value="True"/>
                               </t>
                            </t>
                            <t t-call="payment.summary_item">
                                <t t-set="name" t-value="'provider'"/>
                                <t t-set="label">Processed by</t>
                                <t t-set="value" t-value="tx.provider_id.sudo().name"/>
                                <t t-set="include_separator" t-value="True"/>
                            </t>
                        </div>

                        <div class="row">
                            <div class="col offset-md-3 ps-0">
                                <a role="button" class="btn btn-primary float-end" href="/my/home">
                                    Go to my Account <i class="oi oi-arrow-right ms-2"/>
                                </a>
                            </div>
                        </div>
                    </div>
                    </div>
                </div>
            </div>
        </t>
    </template>

    <!-- Breadcrumb for the portal -->
    <template id="payment.portal_breadcrumb">
        <!-- Parameters description:
            - page_title: The title of the breadcrumb item.
        -->
        <div class="row">
            <div class="col-md-6">
                <ol class="breadcrumb px-0 mt16">
                    <li id="o_payment_portal_home" class="breadcrumb-item">
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

    <template id="payment.summary_item">
        <!-- Parameters description:
            - name: The summary item name that is suffixed to `o_payment_summary_` to create the id.
            - label: The label that is displayed.
            - value: The value of the summary item.
            - options: The widget options to set.
            - include_separator: Whether the summary item should be preceded by a separator.
        -->
        <t t-set="options" t-value="options or {'widget': 'string'}"/>
        <hr t-if="include_separator" class="d-md-none m-0 text-300 opacity-100"/>
        <div t-att-class="'col my-3 text-break'
                          + (' o_payment_summary_separator' if include_separator else '')"
        >
            <label t-attf-for="o_payment_summary_{{name}}"
                   t-out="label"
                   class="d-block small opacity-75"
            />
            <span t-attf-id="o_payment_summary_{{name}}"
                  t-out="value"
                  t-options="options"
                  class="fs-5 fw-bold"
            />
        </div>
    </template>

    <template id="payment.state_header">
        <!-- Parameters description:
            - tx: The transaction whose status must be displayed.
            - is_processing: Whether the transaction is being processed.
        -->
        <t t-set="waiting_heading">
            <p>Please wait...</p>
        </t>
        <t t-if="tx.state == 'draft'">
            <t t-set="alert_style" t-value="'warning'"/>
            <t t-if="is_processing" t-set="status_heading" t-value="waiting_heading"/>
            <t t-set="status_message">
                <p>Your payment has not been processed yet.</p>
            </t>
        </t>
        <t t-elif="tx.state == 'pending'">
            <t t-set="alert_style" t-value="'info'"/>
            <t t-if="is_processing" t-set="status_heading" t-value="waiting_heading"/>
            <t t-if="tx.operation == 'validation'" t-set="status_message">
                <p>Saving your payment method.</p>
            </t>
            <t t-else="" t-set="status_message" t-value="tx.provider_id.sudo().pending_msg"/>
        </t>
        <t t-elif="tx.state == 'authorized'">
            <t t-set="alert_style" t-value="'success'"/>
            <t t-if="is_processing" t-set="status_heading" t-value="waiting_heading"/>
            <t t-set="status_message" t-value="tx.provider_id.sudo().auth_msg"/>
        </t>
        <t t-elif="tx.state == 'done'">
            <t t-set="alert_style" t-value="'success'"/>
            <t t-if="not is_processing" t-set="status_heading">
                <p>Thank you!</p>
            </t>
            <t t-if="tx.operation == 'validation'" t-set="status_message">
                <p>Your payment method has been saved.</p>
            </t>
            <t t-else="" t-set="status_message" t-value="tx.provider_id.sudo().done_msg"/>
        </t>
        <t t-elif="tx.state == 'cancel'">
            <t t-set="alert_style" t-value="'danger'"/>
            <t t-if="tx.operation == 'validation'" t-set="status_message">
                <p>The saving of your payment method has been canceled.</p>
            </t>
            <t t-else="" t-set="status_message" t-value="tx.provider_id.sudo().cancel_msg"/>
        </t>
        <t t-elif="tx.state == 'error'">
            <t t-set="alert_style" t-value="'danger'"/>
            <t t-if="tx.operation == 'validation'" t-set="status_message">
                <p class="mb-0">An error occurred while saving your payment method.</p>
            </t>
            <t t-else="" t-set="status_message">
                <p class="mb-0">An error occurred during the processing of your payment.</p>
            </t>
        </t>

        <t t-if="is_html_empty(status_message)" t-set="status_message" t-value="''"/>
        <t t-set="o_payment_status_alert_class"
           t-value="'alert alert-'+ alert_style +' d-flex gap-3'"
        />

        <div t-if="status_heading or status_message or tx.state_message"
             name="o_payment_status_alert"
             t-attf-class="{{o_payment_status_alert_class}}"
        >
            <t t-set="alert_icon"
               t-value="'fa-cog fa-spin' if is_processing and alert_style != 'danger'
                        else 'fa-check' if alert_style == 'success'
                        else 'fa-info-circle' if alert_style == 'info'
                        else 'fa-exclamation-triangle'"
            />
            <div id="o_payment_status_icon">
                <i t-attf-class="fa {{alert_icon}}"/>
            </div>
            <div id="o_payment_status_message" class="w-100">
                <h5 t-if="status_heading" t-out="status_heading" class="alert-heading mb-0"/>
                <t t-if="status_message" t-out="status_message" class="mb-0"/>
                <t t-if="tx.state_message" t-out="tx.state_message" class="mb-0"/>
            </div>
            <a t-if="is_processing"
               t-att-href="tx.landing_route"
               class="alert-link ms-auto text-nowrap"
            >
                Skip <i class="oi oi-arrow-right ms-1 small"/>
            </a>
        </div>
    </template>

    <!-- "Manage payment methods" card on /my -->
    <template id="portal_my_home_payment" name="Payment Methods" customize_show="True" inherit_id="portal.portal_my_home" priority="60">
        <xpath expr="//div[hasclass('o_portal_docs')]" position="before">
            <t t-set="portal_client_category_enable" t-value="True"/>
        </xpath>
        <div id="portal_client_category" position="inside">
            <t t-set="partner_sudo" t-value="request.env.user.partner_id"/>
            <t t-set="providers_sudo"
               t-value="request.env['payment.provider'].sudo()._get_compatible_providers(request.env.company.id, partner_sudo.id, 0., force_tokenization=True, is_validation=True)"/>
            <t t-set="methods_allowing_tokenization"
               t-value="request.env['payment.method'].sudo()._get_compatible_payment_methods(
                            providers_sudo.ids,
                            partner_sudo.id,
                            force_tokenization=True,
                        )"
            />
            <t t-set="existing_tokens" t-value="partner_sudo.payment_token_ids + partner_sudo.commercial_partner_id.payment_token_ids"/>
            <t t-if="methods_allowing_tokenization or existing_tokens" t-call="portal.portal_docs_entry">
                <t t-set="icon" t-value="'/payment/static/img/payment-methods.svg'"/>
                <t t-set="title">Payment methods</t>
                <t t-set="text">Manage your payment methods</t>
                <t t-set="url" t-value="'/my/payment_method'"/>
                <t t-set="config_card" t-value="True"/>
            </t>
        </div>
    </template>

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
                        context="{'search_default_partner_id': id, 'create': False, 'edit': False}"
                        invisible="payment_token_count == 0">
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

## File: wizards\payment_capture_wizard.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError
from odoo.tools import format_amount


class PaymentCaptureWizard(models.TransientModel):
    _name = 'payment.capture.wizard'
    _description = "Payment Capture Wizard"

    transaction_ids = fields.Many2many(  # All the source txs related to the capture request
        comodel_name='payment.transaction',
        default=lambda self: self.env.context.get('active_ids'),
        readonly=True,
    )
    authorized_amount = fields.Monetary(
        string="Authorized Amount", compute='_compute_authorized_amount'
    )
    captured_amount = fields.Monetary(string="Already Captured", compute='_compute_captured_amount')
    voided_amount = fields.Monetary(string="Already Voided", compute='_compute_voided_amount')
    available_amount = fields.Monetary(
        string="Maximum Capture Allowed", compute='_compute_available_amount'
    )
    amount_to_capture = fields.Monetary(
        compute='_compute_amount_to_capture', store=True, readonly=False
    )
    is_amount_to_capture_valid = fields.Boolean(compute='_compute_is_amount_to_capture_valid')
    void_remaining_amount = fields.Boolean()
    currency_id = fields.Many2one(related='transaction_ids.currency_id')
    support_partial_capture = fields.Boolean(
        help="Whether each of the transactions' provider supports the partial capture.",
        compute='_compute_support_partial_capture',
        compute_sudo=True,
    )
    has_draft_children = fields.Boolean(compute='_compute_has_draft_children')
    has_remaining_amount = fields.Boolean(compute='_compute_has_remaining_amount')

    #=== COMPUTE METHODS ===#

    @api.depends('transaction_ids')
    def _compute_authorized_amount(self):
        for wizard in self:
            wizard.authorized_amount = sum(wizard.transaction_ids.mapped('amount'))

    @api.depends('transaction_ids')
    def _compute_captured_amount(self):
        for wizard in self:
            full_capture_txs = wizard.transaction_ids.filtered(
                lambda tx: tx.state == 'done' and not tx.child_transaction_ids
            )  # Transactions that have been fully captured in a single capture operation.
            partial_capture_child_txs = wizard.transaction_ids.child_transaction_ids.filtered(
                lambda tx: tx.state == 'done'
            )  # Transactions that represent a partial capture of their source transaction.
            wizard.captured_amount = sum(
                (full_capture_txs | partial_capture_child_txs).mapped('amount')
            )

    @api.depends('transaction_ids')
    def _compute_voided_amount(self):
        for wizard in self:
            void_child_txs = wizard.transaction_ids.child_transaction_ids.filtered(
                lambda tx: tx.state == 'cancel'
            )
            wizard.voided_amount = sum(void_child_txs.mapped('amount'))

    @api.depends('authorized_amount', 'captured_amount', 'voided_amount')
    def _compute_available_amount(self):
        for wizard in self:
            wizard.available_amount = wizard.authorized_amount \
                                      - wizard.captured_amount \
                                      - wizard.voided_amount

    @api.depends('available_amount')
    def _compute_amount_to_capture(self):
        """ Set the default amount to capture to the amount available for capture. """
        for wizard in self:
            wizard.amount_to_capture = wizard.available_amount

    @api.depends('amount_to_capture', 'available_amount')
    def _compute_is_amount_to_capture_valid(self):
        for wizard in self:
            is_valid = 0 < wizard.amount_to_capture <= wizard.available_amount
            wizard.is_amount_to_capture_valid = is_valid

    @api.depends('transaction_ids')
    def _compute_support_partial_capture(self):
        for wizard in self:
            wizard.support_partial_capture = all(
                tx.provider_id.support_manual_capture == 'partial' for tx in wizard.transaction_ids
            )

    @api.depends('transaction_ids')
    def _compute_has_draft_children(self):
        for wizard in self:
            wizard.has_draft_children = bool(wizard.transaction_ids.child_transaction_ids.filtered(
                lambda tx: tx.state == 'draft'
            ))

    @api.depends('available_amount', 'amount_to_capture')
    def _compute_has_remaining_amount(self):
        for wizard in self:
            wizard.has_remaining_amount = wizard.amount_to_capture < wizard.available_amount
            if not wizard.has_remaining_amount:
                wizard.void_remaining_amount = False

    #=== CONSTRAINT METHODS ===#

    @api.constrains('amount_to_capture')
    def _check_amount_to_capture_within_boundaries(self):
        for wizard in self:
            if not wizard.is_amount_to_capture_valid:
                formatted_amount = format_amount(
                    self.env, wizard.available_amount, wizard.currency_id
                )
                raise ValidationError(_(
                    "The amount to capture must be positive and cannot be superior to %s.",
                    formatted_amount
                ))
            if not wizard.support_partial_capture \
               and wizard.amount_to_capture != wizard.available_amount:
                raise ValidationError(_(
                    "Some of the transactions you intend to capture can only be captured in full. "
                    "Handle the transactions individually to capture a partial amount."
                ))

    #=== ACTION METHODS ===#

    def action_capture(self):
        for wizard in self:
            remaining_amount_to_capture = wizard.amount_to_capture
            for source_tx in wizard.transaction_ids.filtered(lambda tx: tx.state == 'authorized'):
                partial_capture_child_txs = wizard.transaction_ids.child_transaction_ids.filtered(
                    lambda tx: tx.source_transaction_id == source_tx and tx.state == 'done'
                )  # We can void all the remaining amount only at once => don't check cancel state.
                source_tx_remaining_amount = source_tx.currency_id.round(
                    source_tx.amount - sum(partial_capture_child_txs.mapped('amount'))
                )
                if remaining_amount_to_capture:
                    amount_to_capture = min(source_tx_remaining_amount, remaining_amount_to_capture)
                    # In sudo mode because we need to be able to read on provider fields.
                    source_tx.sudo()._send_capture_request(amount_to_capture=amount_to_capture)
                    remaining_amount_to_capture -= amount_to_capture
                    source_tx_remaining_amount -= amount_to_capture

                if source_tx_remaining_amount and wizard.void_remaining_amount:
                    # The source tx isn't fully captured and the user wants to void the remaining.
                    # In sudo mode because we need to be able to read on provider fields.
                    source_tx.sudo()._send_void_request(amount_to_void=source_tx_remaining_amount)
                elif not remaining_amount_to_capture and not wizard.void_remaining_amount:
                    # The amount to capture has been completely captured.
                    break  # Skip the remaining transactions.

```

## File: wizards\payment_capture_wizard_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_capture_wizard_view_form" model="ir.ui.view">
        <field name="name">payment.capture.wizard.form</field>
        <field name="model">payment.capture.wizard</field>
        <field name="arch" type="xml">
            <form string="Capture">
                <field name="transaction_ids" invisible="1"/>
                <field name="is_amount_to_capture_valid" invisible="1"/>
                <field name="currency_id" invisible="1"/>
                <field name="support_partial_capture" invisible="1"/>
                <field name="has_draft_children" invisible="1"/>
                <field name="has_remaining_amount" invisible="1"/>
                <div id="alert_draft_capture_tx"
                     role="alert"
                     class="alert alert-warning"
                     invisible="not has_draft_children">
                    <strong>Warning!</strong> There is a partial capture pending. Please wait a
                    moment for it to be processed. Check your payment provider configuration if
                    the capture is still pending after a few minutes.
                </div>
                <group name="readonly_fields">
                    <field name="authorized_amount"/>
                    <field name="captured_amount"
                           invisible="captured_amount &lt;= 0"/>
                    <field name="voided_amount"
                           invisible="voided_amount &lt;= 0"/>
                </group>
                <hr/>
                <group name="input_fields">
                    <label for="amount_to_capture" class="oe_inline"/>
                    <div class="o_row">
                        <field name="amount_to_capture"
                               class="oe_inline"
                               readonly="support_partial_capture == 'full_only'"/>
                        <i class="fa fa-info-circle oe_inline"
                           invisible="support_partial_capture != 'full_only'"
                           title="Some of the transactions you intend to capture can only be captured in full. Handle the transactions individually to capture a partial amount."/>
                    </div>
                    <field name="void_remaining_amount" readonly="not has_remaining_amount"/>
                </group>
                <div id="alert_amount_to_capture_above_authorized_amount"
                     role="alert"
                     class="alert alert-warning mb-2"
                     invisible="is_amount_to_capture_valid">
                    <strong>Warning!</strong> You can not capture a negative amount nor more
                    than <field name='available_amount' class='oe_inline' widget='monetary'/>.
                </div>
                <footer>
                    <button string="Capture" type="object" name="action_capture" class="btn-primary"/>
                    <button string="Close" special="cancel" class="btn-secondary"/>
                </footer>
            </form>
        </field>
    </record>

</odoo>

```

## File: wizards\payment_link_wizard.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from werkzeug import urls

from odoo import _, api, fields, models

from odoo.addons.payment import utils as payment_utils


class PaymentLinkWizard(models.TransientModel):
    _name = 'payment.link.wizard'
    _description = "Generate Payment Link"

    @api.model
    def default_get(self, fields_list):
        res = super().default_get(fields_list)
        res_id = self.env.context.get('active_id')
        res_model = self.env.context.get('active_model')
        if res_id and res_model:
            res.update({'res_model': res_model, 'res_id': res_id})
            res.update(
                self.env[res_model].browse(res_id)._get_default_payment_link_values()
            )
        return res

    res_model = fields.Char("Related Document Model", required=True)
    res_id = fields.Integer("Related Document ID", required=True)
    amount = fields.Monetary(currency_field='currency_id', required=True)
    amount_max = fields.Monetary(currency_field='currency_id')
    currency_id = fields.Many2one('res.currency')
    partner_id = fields.Many2one('res.partner')
    partner_email = fields.Char(related='partner_id.email')
    link = fields.Char(string="Payment Link", compute='_compute_link')
    company_id = fields.Many2one('res.company', compute='_compute_company_id')
    warning_message = fields.Char(compute='_compute_warning_message')

    @api.depends('amount', 'amount_max')
    def _compute_warning_message(self):
        self.warning_message = ''
        for wizard in self:
            if wizard.amount_max <= 0:
                wizard.warning_message = _("There is nothing to be paid.")
            elif wizard.amount <= 0:
                wizard.warning_message = _("Please set a positive amount.")
            elif wizard.amount > wizard.amount_max:
                wizard.warning_message = _("Please set an amount lower than %s.", wizard.currency_id.format(wizard.amount_max))

    @api.depends('res_model', 'res_id')
    def _compute_company_id(self):
        for link in self:
            record = self.env[link.res_model].browse(link.res_id)
            link.company_id = record.company_id if 'company_id' in record else False

    @api.depends('amount', 'currency_id', 'partner_id', 'company_id')
    def _compute_link(self):
        for payment_link in self:
            related_document = self.env[payment_link.res_model].browse(payment_link.res_id)
            base_url = related_document.get_base_url()  # Generate links for the right website.
            url = self._prepare_url(base_url, related_document)
            query_params = self._prepare_query_params(related_document)
            anchor = self._prepare_anchor()
            if '?' in url:
                payment_link.link = f'{url}&{urls.url_encode(query_params)}{anchor}'
            else:
                payment_link.link = f'{url}?{urls.url_encode(query_params)}{anchor}'

    def _prepare_url(self, base_url, related_document):
        """ Build the URL of the payment link with the website's base URL and return it.
        :param str base_url: The website's base URL.
        :param recordset related_document: The record for which the payment link is generated.
        :return: The URL of the payment link.
        :rtype: str
        """
        return f'{base_url}/payment/pay'

    def _prepare_query_params(self, related_document):
        """ Prepare the query string params to append to the payment link URL.

        Note: self.ensure_one()

        :param recordset related_document: The record for which the payment link is generated.
        :return: The query params of the payment link.
        :rtype: dict
        """
        self.ensure_one()
        return {
            'amount': self.amount,
            'access_token': self._prepare_access_token(),
            'currency_id': self.currency_id.id,
            'partner_id': self.partner_id.id,
            'company_id': self.company_id.id,
        }

    def _prepare_access_token(self):
        self.ensure_one()
        return payment_utils.generate_access_token(
            self.partner_id.id, self.amount, self.currency_id.id
        )

    def _prepare_anchor(self):
        """ Prepare the anchor to append to the payment link.

        Note: self.ensure_one()

        :return: The anchor of the payment link.
        :rtype: str
        """
        self.ensure_one()
        return ''

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
                <div name="no_partner_email"
                     class="alert alert-warning fw-bold"
                     role="alert"
                     invisible="partner_email">
                     This partner has no email, which may cause issues with some payment providers.
                     Setting an email for this partner is advised.
                </div>
                <div name="payment_link_warning_information"
                     class="alert alert-warning fw-bold"
                     role="alert"
                     invisible="warning_message == ''">
                    <field name="warning_message"/>
                </div>
                <group>
                    <group name="payment_info" string="Payment Info" class="mt-n4">
                        <field name="res_id" invisible="1"/>
                        <field name="res_model" invisible="1"/>
                        <field name="partner_id" invisible="1"/>
                        <field name="partner_email" invisible="1"/>
                        <field name="amount"/>
                        <field name="amount_max" invisible="1"/>
                        <field name="warning_message" invisible="1"/>
                        <field name="currency_id" invisible="1"/>
                    </group>
                </group>
                <footer>
                    <field name="link"
                           string="Generate and Copy Payment Link"
                           readonly="1"
                           disabled="bool(warning_message)"
                           widget="PaymentWizardCopyClipboardButtonField"
                           data-hotkey="q"/>
                    <button string="Close"
                            class="btn btn-secondary rounded-2"
                            special="cancel"
                            data-hotkey="x"/>
                </footer>
            </form>
        </field>
    </record>

</odoo>

```

## File: wizards\payment_onboarding_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_provider_onboarding_wizard_form" model="ir.ui.view">
        <field name="name">payment.provider.onboarding.wizard.form</field>
        <field name="model">payment.provider.onboarding.wizard</field>
        <field name="arch" type="xml">
            <form string="Choose a payment method" class="o_onboarding_payment_provider_wizard">
                <div class="container">
                    <div class="row align-items-start">
                        <div class="col col-4" name="left-column">
                                <field name="payment_method" widget="radio"/>
                        </div>
                        <div class="col" name="right-column">
                            <div invisible="payment_method != 'paypal'">
                                <group>
                                    <field name="paypal_email_account" required="payment_method == 'paypal'" string="Email"/>
                                </group>
                                <widget name="documentation_link" path="/applications/finance/payment_providers/paypal.html" label=" How to configure your PayPal account" icon="oi oi-arrow-right"/>
                            </div>

                            <div invisible="payment_method != 'manual'">
                                <group>
                                    <field name="manual_name" required="payment_method == 'manual'"/>
                                    <field name="journal_name" required="payment_method == 'manual'"/>
                                    <field name="acc_number" required="payment_method == 'manual'"/>
                                    <field name="manual_post_msg" required="payment_method == 'manual'"/>
                                </group>
                            </div>
                        </div>
                    </div>
                </div>
                <footer>
                    <button name="add_payment_methods" string="Apply" class="oe_highlight"
                            type="object" data-hotkey="q" />
                    <button special="cancel" data-hotkey="x" string="Cancel" />
                </footer>
            </form>
        </field>
    </record>

</odoo>

```

## File: wizards\payment_onboarding_wizard.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.exceptions import UserError


class PaymentWizard(models.TransientModel):
    _name = 'payment.provider.onboarding.wizard'
    _description = 'Payment provider onboarding wizard'

    payment_method = fields.Selection([
        ('stripe', "Credit & Debit card (via Stripe)"),
        ('paypal', "PayPal"),
        ('manual', "Custom payment instructions"),
    ], string="Payment Method", default=lambda self: self._get_default_payment_provider_onboarding_value('payment_method'))
    paypal_email_account = fields.Char("Email", default=lambda self: self._get_default_payment_provider_onboarding_value('paypal_email_account'))

    # Account-specific logic. It's kept here rather than moved in `account_payment` as it's not used by `account` module.
    manual_name = fields.Char("Method", default=lambda self: self._get_default_payment_provider_onboarding_value('manual_name'))
    journal_name = fields.Char("Bank Name", default=lambda self: self._get_default_payment_provider_onboarding_value('journal_name'))
    acc_number = fields.Char("Account Number", default=lambda self: self._get_default_payment_provider_onboarding_value('acc_number'))
    manual_post_msg = fields.Html("Payment Instructions")

    _data_fetched = fields.Boolean(store=False)

    @api.onchange('journal_name', 'acc_number')
    def _set_manual_post_msg_value(self):
        self.manual_post_msg = _(
            '<h3>Please make a payment to: </h3><ul><li>Bank: %(bank)s</li><li>Account Number: %(account_number)s</li><li>Account Holder: %(account_holder)s</li></ul>',
            bank=self.journal_name or _("Bank"),
            account_number=self.acc_number or _("Account"),
            account_holder=self.env.company.name,
        )

    _payment_provider_onboarding_cache = {}

    def _get_manual_payment_provider(self, env=None):
        if env is None:
            env = self.env
        module_id = env.ref('base.module_payment_custom').id
        return env['payment.provider'].search([
            *env['payment.provider']._check_company_domain(self.env.company),
            ('module_id', '=', module_id),
        ], limit=1)

    def _get_default_payment_provider_onboarding_value(self, key):
        if not self.env.is_admin():
            raise UserError(_("Only administrators can access this data."))

        if self._data_fetched:
            return self._payment_provider_onboarding_cache.get(key, '')

        self._data_fetched = True

        self._payment_provider_onboarding_cache['payment_method'] = self.env.company.payment_onboarding_payment_method

        installed_modules = self.env['ir.module.module'].sudo().search([
            ('name', 'in', ('payment_paypal', 'payment_stripe')),
            ('state', '=', 'installed'),
        ]).mapped('name')

        if 'payment_paypal' in installed_modules:
            provider = self.env['payment.provider'].search([
                *self.env['payment.provider']._check_company_domain(self.env.company),
                ('code', '=', 'paypal'),

            ], limit=1)
            self._payment_provider_onboarding_cache['paypal_email_account'] = provider['paypal_email_account'] or self.env.company.email
        else:
            self._payment_provider_onboarding_cache['paypal_email_account'] = self.env.company.email

        manual_payment = self._get_manual_payment_provider()
        journal = manual_payment.journal_id

        self._payment_provider_onboarding_cache['manual_name'] = manual_payment['name']
        self._payment_provider_onboarding_cache['manual_post_msg'] = manual_payment['pending_msg']
        self._payment_provider_onboarding_cache['journal_name'] = journal.name if journal.name != "Bank" else ""
        self._payment_provider_onboarding_cache['acc_number'] = journal.bank_acc_number

        return self._payment_provider_onboarding_cache.get(key, '')

    def add_payment_methods(self):
        """ Install required payment providers, configure them and mark the
            onboarding step as done."""
        payment_method = self.payment_method

        if self.payment_method == 'paypal':
            self.env.company._install_modules(['payment_paypal', 'account_payment'])
        elif self.payment_method == 'manual':
            self.env.company._install_modules(['account_payment'])

        if self.payment_method in ('paypal', 'manual'):
            # create a new env including the freshly installed module(s)
            new_env = api.Environment(self.env.cr, self.env.uid, self.env.context)

            if self.payment_method == 'paypal':
                provider = new_env['payment.provider'].search([
                    *self.env['payment.provider']._check_company_domain(self.env.company),
                    ('code', '=', 'paypal')
                ], limit=1)
                if not provider:
                    base_provider = self.env.ref('payment.payment_provider_paypal')
                    # Use sudo to access payment provider record that can be in different company.
                    provider = base_provider.sudo().copy(default={'company_id':self.env.company.id})
                provider.write({
                    'paypal_email_account': self.paypal_email_account,
                    'state': 'enabled',
                    'is_published': 'True',
                })
            elif self.payment_method == 'manual':
                manual_provider = self._get_manual_payment_provider(new_env)
                if not manual_provider:
                    raise UserError(_(
                        'No manual payment method could be found for this company. '
                        'Please create one from the Payment Provider menu.'
                    ))
                manual_provider.name = self.manual_name
                manual_provider.pending_msg = self.manual_post_msg
                manual_provider.state = 'enabled'

                journal = manual_provider.journal_id
                if journal:
                    journal.name = self.journal_name
                    journal.bank_acc_number = self.acc_number

        if self.payment_method in ('paypal', 'manual', 'stripe'):
            self.env.company.payment_onboarding_payment_method = self.payment_method

        # delete wizard data immediately to get rid of residual credentials
        self.sudo().unlink()

        if payment_method == 'stripe':
            return self._start_stripe_onboarding()

        # the user clicked `apply` and not cancel, so we can assume this step is done.
        self.env['onboarding.onboarding.step'].sudo().action_validate_step_payment_provider()
        return {'type': 'ir.actions.act_window_close'}

    def _start_stripe_onboarding(self):
        """ Start Stripe Connect onboarding. """
        menu = self.env.ref('account_payment.payment_provider_menu', False)
        menu_id = menu and menu.id  # Only set if `account_payment` is installed.
        return self.env.company._run_payment_onboarding_step(menu_id)

```

## File: wizards\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import payment_capture_wizard
from . import payment_link_wizard
from . import payment_onboarding_wizard

```

