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


def setup_provider(cr, registry, code):
    env = api.Environment(cr, SUPERUSER_ID, {})
    env['payment.provider']._setup_provider(code)


def reset_payment_provider(cr, registry, code):
    env = api.Environment(cr, SUPERUSER_ID, {})
    env['payment.provider']._remove_provider(code)

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': "Payment Engine",
    'version': '2.0',
    'category': 'Hidden',
    'summary': "The payment engine used by payment provider modules.",
    'depends': ['portal'],
    'data': [
        'data/payment_icon_data.xml',
        'data/payment_provider_data.xml',
        'data/payment_cron.xml',

        'views/payment_portal_templates.xml',
        'views/payment_templates.xml',

        'views/payment_provider_views.xml',
        'views/payment_icon_views.xml',
        'views/payment_transaction_views.xml',
        'views/payment_token_views.xml',  # Depends on `action_payment_transaction_linked_to_token`
        'views/res_partner_views.xml',

        'security/ir.model.access.csv',
        'security/payment_security.xml',

        'wizards/payment_link_wizard_views.xml',
        'wizards/payment_onboarding_views.xml',
    ],
    'demo': [
        'data/payment_demo.xml',
    ],
    'assets': {
        'web.assets_frontend': [
            'payment/static/src/scss/portal_payment.scss',
            'payment/static/src/scss/payment_templates.scss',
            'payment/static/src/scss/payment_form.scss',
            'payment/static/lib/jquery.payment/jquery.payment.js',
            'payment/static/src/js/checkout_form.js',
            'payment/static/src/js/express_checkout_form.js',
            'payment/static/src/js/manage_form.js',
            'payment/static/src/js/payment_form_mixin.js',
            'payment/static/src/js/post_processing.js',
            'payment/static/src/xml/payment_post_processing.xml',
        ],
        'web.assets_backend': [
            'payment/static/src/scss/payment_provider.scss',
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
        provider_id=None, access_token=None, **kwargs
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
        :param str provider_id: The desired provider, as a `payment.provider` id
        :param str access_token: The access token used to authenticate the partner
        :param dict kwargs: Optional data passed to helper methods.
        :return: The rendered checkout form
        :rtype: str
        :raise: werkzeug.exceptions.NotFound if the access token is invalid
        """
        # Cast numeric parameters as int or float and void them if their str value is malformed
        currency_id, provider_id, partner_id, company_id = tuple(map(
            self._cast_as_int, (currency_id, provider_id, partner_id, company_id)
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

        # Select all providers and tokens that match the constraints
        providers_sudo = request.env['payment.provider'].sudo()._get_compatible_providers(
            company_id, partner_sudo.id, amount, currency_id=currency.id, **kwargs
        )  # In sudo mode to read the fields of providers and partner (if not logged in)
        if provider_id in providers_sudo.ids:  # Only keep the desired provider if it's suitable
            providers_sudo = providers_sudo.browse(provider_id)
        payment_tokens = request.env['payment.token'].search(
            [('provider_id', 'in', providers_sudo.ids), ('partner_id', '=', partner_sudo.id)]
        ) if logged_in else request.env['payment.token']

        # Make sure that the partner's company matches the company passed as parameter.
        if not PaymentPortal._can_partner_pay_in_company(partner_sudo, company):
            providers_sudo = request.env['payment.provider'].sudo()
            payment_tokens = request.env['payment.token']

        # Compute the fees taken by providers supporting the feature
        fees_by_provider = {
            provider_sudo: provider_sudo._compute_fees(amount, currency, partner_sudo.country_id)
            for provider_sudo in providers_sudo.filtered('fees_active')
        }

        # Generate a new access token in case the partner id or the currency id was updated
        access_token = payment_utils.generate_access_token(partner_sudo.id, amount, currency.id)

        rendering_context = {
            'providers': providers_sudo,
            'tokens': payment_tokens,
            'fees_by_provider': fees_by_provider,
            'show_tokenize_input': self._compute_show_tokenize_input_mapping(
                providers_sudo, logged_in=logged_in, **kwargs
            ),
            'reference_prefix': reference,
            'amount': amount,
            'currency': currency,
            'partner_id': partner_sudo.id,
            'access_token': access_token,
            'transaction_route': '/payment/transaction',
            'landing_route': '/payment/confirmation',
            'res_company': company,  # Display the correct logo in a multi-company environment
            'partner_is_different': partner_is_different,
            **self._get_custom_rendering_context_values(**kwargs),
        }
        return request.render(self._get_payment_page_template_xmlid(**kwargs), rendering_context)

    @staticmethod
    def _compute_show_tokenize_input_mapping(providers_sudo, logged_in=False, **kwargs):
        """ Determine for each provider whether the tokenization input should be shown or not.

        :param recordset providers_sudo: The providers for which to determine whether the
                                         tokenization input should be shown or not, as a sudoed
                                         `payment.provider` recordset.
        :param bool logged_in: Whether the user is logged in or not.
        :param dict kwargs: The optional data passed to the helper methods.
        :return: The mapping of the computed value for each provider id.
        :rtype: dict
        """
        show_tokenize_input_mapping = {}
        for provider_sudo in providers_sudo:
            show_tokenize_input = provider_sudo.allow_tokenization \
                                  and not provider_sudo._is_tokenization_required(**kwargs) \
                                  and logged_in
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
        providers_sudo = request.env['payment.provider'].sudo()._get_compatible_providers(
            request.env.company.id,
            partner_sudo.id,
            0.,  # There is no amount to pay with validation transactions.
            force_tokenization=True,
            is_validation=True,
        )

        # Get all partner's tokens for which providers are not disabled.
        tokens_sudo = request.env['payment.token'].sudo().search([
            ('partner_id', 'in', [partner_sudo.id, partner_sudo.commercial_partner_id.id]),
            ('provider_id.state', 'in', ['enabled', 'test']),
        ])

        access_token = payment_utils.generate_access_token(partner_sudo.id, None, None)
        rendering_context = {
            'providers': providers_sudo,
            'tokens': tokens_sudo,
            'reference_prefix': payment_utils.singularize_reference_prefix(prefix='V'),
            'partner_id': partner_sudo.id,
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
        tokenization_requested, landing_route, is_validation=False,
        custom_create_values=None, **kwargs
    ):
        """ Create a draft transaction based on the payment context and return it.

        :param int payment_option_id: The payment option handling the transaction, as a
                                      `payment.provider` id or a `payment.token` id
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
        :param dict custom_create_values: Additional create values overwriting the default ones
        :param dict kwargs: Locally unused data passed to `_is_tokenization_required` and
                            `_compute_reference`
        :return: The sudoed transaction that was created
        :rtype: recordset of `payment.transaction`
        :raise: UserError if the flow is invalid
        """
        # Prepare create values
        if flow in ['redirect', 'direct']:  # Direct payment or payment with redirection
            provider_sudo = request.env['payment.provider'].sudo().browse(payment_option_id)
            token_id = None
            tokenize = bool(
                # Don't tokenize if the user tried to force it through the browser's developer tools
                provider_sudo.allow_tokenization
                # Token is only created if required by the flow or requested by the user
                and (provider_sudo._is_tokenization_required(**kwargs) or tokenization_requested)
            )
        elif flow == 'token':  # Payment by token
            token_sudo = request.env['payment.token'].sudo().browse(payment_option_id)

            # Prevent from paying with a token that doesn't belong to the current partner (either
            # the current user's partner if logged in, or the partner on behalf of whom the payment
            # is being made).
            partner_sudo = request.env['res.partner'].sudo().browse(partner_id)
            if partner_sudo.commercial_partner_id != token_sudo.partner_id.commercial_partner_id:
                raise AccessError(_("You do not have access to this payment token."))

            provider_sudo = token_sudo.provider_id
            token_id = payment_option_id
            tokenize = False
        else:
            raise UserError(
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
            currency_id = provider_sudo._get_validation_currency().id

        # Create the transaction
        tx_sudo = request.env['payment.transaction'].sudo().create({
            'provider_id': provider_sudo.id,
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

            # Stop monitoring the transaction now that it reached a final state.
            PaymentPostProcessing.remove_transactions(tx_sudo)

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
    def poll_status(self, **_kwargs):
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
                display_message = tx.provider_id.pending_msg
            elif tx.state == 'done':
                display_message = tx.provider_id.done_msg
            elif tx.state == 'cancel':
                display_message = tx.provider_id.cancel_msg
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

## File: data\neutralize.sql

```sql
-- disable generic payment provider
UPDATE payment_provider
   SET state = 'disabled'
 WHERE state NOT IN ('test', 'disabled');

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

## File: data\payment_demo.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <!-- Enable the EUR currency since it's the currency of the company. -->
    <function model="res.currency" name="action_unarchive" eval="[[ref('base.EUR')]]"/>

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

    <record id="payment_icon_cc_rupay" model="payment.icon">
        <field name="sequence">65</field>
        <field name="name">Rupay</field>
        <field name="image" type="base64" file="payment/static/img/rupay.png"/>
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

    <record id="payment_icon_mpesa" model="payment.icon">
        <field name="sequence">220</field>
        <field name="name">M-Pesa</field>
        <field name="image" type="base64" file="payment/static/img/m-pesa.png"/>
    </record>

    <record id="payment_icon_airtel_money" model="payment.icon">
        <field name="sequence">230</field>
        <field name="name">Airtel Money</field>
        <field name="image" type="base64" file="payment/static/img/airtel-money.png"/>
    </record>

    <record id="payment_icon_mtn_mobile_money" model="payment.icon">
        <field name="sequence">240</field>
        <field name="name">MTN Mobile Money</field>
        <field name="image" type="base64" file="payment/static/img/mtn-mobile-money.png"/>
    </record>

    <record id="payment_icon_barter_by_flutterwave" model="payment.icon">
        <field name="sequence">250</field>
        <field name="name">Barter by Flutterwave</field>
        <field name="image" type="base64" file="payment/static/img/barter-by-flutterwave.png"/>
    </record>

    <record id="payment_icon_sadad" model="payment.icon">
        <field name="sequence">260</field>
        <field name="name">Sadad</field>
        <field name="image" type="base64" file="payment/static/img/sadad.png"/>
    </record>

    <record id="payment_icon_mada" model="payment.icon">
        <field name="sequence">270</field>
        <field name="name">Mada</field>
        <field name="image" type="base64" file="payment/static/img/mada.png"/>
    </record>

    <record id="payment_icon_bbva_bancomer" model="payment.icon">
        <field name="sequence">280</field>
        <field name="name">BBVA Bancomer</field>
        <field name="image" type="base64" file="payment/static/img/bbva-bancomer.png"/>
    </record>

   <record id="payment_icon_citibanamex" model="payment.icon">
        <field name="sequence">280</field>
        <field name="name">CitiBanamex</field>
        <field name="image" type="base64" file="payment/static/img/citibanamex.png"/>
    </record>

</odoo>

```

## File: data\payment_provider_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="payment_provider_adyen" model="payment.provider">
        <field name="name">Adyen</field>
        <field name="display_as">Credit Card (powered by Adyen)</field>
        <field name="image_128" type="base64" file="payment_adyen/static/description/icon.png"/>
        <field name="module_id" ref="base.module_payment_adyen"/>
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

    <record id="payment_provider_aps" model="payment.provider">
        <field name="name">Amazon Payment Services</field>
        <field name="display_as">Amazon Payment Services</field>
        <field name="image_128" type="base64" file="payment_aps/static/description/icon.png"/>
        <field name="module_id" ref="base.module_payment_aps"/>
        <!-- https://paymentservices.amazon.com/docs/EN/24.html -->
        <field name="payment_icon_ids"
               eval="[(6, 0, [
                        ref('payment.payment_icon_cc_mastercard'),
                        ref('payment.payment_icon_cc_visa'),
                        ref('payment.payment_icon_sadad'),
                        ref('payment.payment_icon_mada'),
                    ])]"/>
    </record>

    <record id="payment_provider_asiapay" model="payment.provider">
        <field name="name">Asiapay</field>
        <field name="display_as">Credit Card (powered by Asiapay)</field>
        <field name="image_128" type="base64" file="payment_asiapay/static/description/icon.png"/>
        <field name="module_id" ref="base.module_payment_asiapay"/>
        <!-- See https://www.asiapay.com/payment.html#option -->
        <field name="payment_icon_ids"
               eval="[(6, 0, [
                    ref('payment.payment_icon_cc_mastercard'),
                    ref('payment.payment_icon_cc_visa'),
                    ref('payment.payment_icon_cc_unionpay'),
               ])]"/>
    </record>

    <record id="payment_provider_authorize" model="payment.provider">
        <field name="name">Authorize.net</field>
        <field name="display_as">Credit Card (powered by Authorize)</field>
        <field name="image_128"
               type="base64"
               file="payment_authorize/static/description/icon.png"/>
        <field name="module_id" ref="base.module_payment_authorize"/>
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

    <record id="payment_provider_buckaroo" model="payment.provider">
        <field name="name">Buckaroo</field>
        <field name="display_as">Credit Card (powered by Buckaroo)</field>
        <field name="image_128"
               type="base64"
               file="payment_buckaroo/static/description/icon.png"/>
        <field name="module_id" ref="base.module_payment_buckaroo"/>
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
        <field name="payment_icon_ids" eval="[(6, 0, [
                ref('payment.payment_icon_cc_visa'),
                ref('payment.payment_icon_cc_mastercard'),
                ref('payment.payment_icon_cc_american_express'),
                ref('payment.payment_icon_mpesa'),
                ref('payment.payment_icon_airtel_money'),
                ref('payment.payment_icon_mtn_mobile_money'),
                ref('payment.payment_icon_barter_by_flutterwave'),
            ])]"/>
    </record>

    <record id="payment_provider_mercado_pago" model="payment.provider">
        <field name="name">Mercado Pago</field>
        <field name="display_as">Credit Card (powered by Mercado Pago)</field>
        <field name="image_128"
               type="base64"
               file="payment_mercado_pago/static/description/icon.png"/>
        <field name="module_id" ref="base.module_payment_mercado_pago"/>

         <!-- Payment methods must be fetched from the API. See
            https://www.mercadopago.com.ar/developers/en/reference/payment_methods/_payment_methods/
        -->
        <field name="payment_icon_ids"
               eval="[(6, 0, [
                    ref('payment.payment_icon_cc_mastercard'),
                    ref('payment.payment_icon_cc_visa'),
                    ref('payment.payment_icon_cc_american_express'),
                    ref('payment.payment_icon_bbva_bancomer'),
                    ref('payment.payment_icon_citibanamex')
               ])]"/>
    </record>

    <record id="payment_provider_mollie" model="payment.provider">
        <field name="name">Mollie</field>
        <field name="image_128" type="base64" file="payment_mollie/static/description/icon.png"/>
        <field name="module_id" ref="base.module_payment_mollie"/>
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

    <record id="payment_provider_paypal" model="payment.provider">
        <field name="name">PayPal</field>
        <field name="image_128" type="base64" file="payment_paypal/static/description/icon.png"/>
        <field name="module_id" ref="base.module_payment_paypal"/>
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

    <record id="payment_provider_razorpay" model="payment.provider">
        <field name="name">Razorpay</field>
        <field name="display_as">Credit &amp; Debit Card, UPI (Powered by Razorpay)</field>
        <field name="image_128" type="base64" file="payment_razorpay/static/description/icon.png"/>
        <field name="module_id" ref="base.module_payment_razorpay"/>
        <!-- https://razorpay.com/docs/payments/payment-methods/#supported-payment-methods -->
        <field name="payment_icon_ids"
               eval="[(6, 0, [
                   ref('payment.payment_icon_cc_maestro'),
                   ref('payment.payment_icon_cc_mastercard'),
                   ref('payment.payment_icon_cc_rupay'),
                   ref('payment.payment_icon_cc_diners_club_intl'),
                   ref('payment.payment_icon_cc_american_express'),
                   ref('payment.payment_icon_cc_visa')
               ])]"/>
    </record>

    <record id="payment_provider_sepa_direct_debit" model="payment.provider">
        <field name="name">SEPA Direct Debit</field>
        <field name="sequence">20</field>
        <field name="image_128"
               type="base64"
               file="base/static/img/icons/payment_sepa_direct_debit.png"/>
        <field name="module_id" ref="base.module_payment_sepa_direct_debit"/>
    </record>

    <record id="payment_provider_sips" model="payment.provider">
        <field name="name">Sips</field>
        <field name="display_as">Credit Card (powered by Sips)</field>
        <field name="image_128" type="base64" file="payment_sips/static/description/icon.png"/>
        <field name="module_id" ref="base.module_payment_sips"/>
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

    <record id="payment_provider_stripe" model="payment.provider">
        <field name="name">Stripe</field>
        <field name="display_as">Credit &amp; Debit Card</field>
        <field name="image_128" type="base64" file="payment_stripe/static/description/icon.png"/>
        <field name="module_id" ref="base.module_payment_stripe"/>
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

    <record id="payment_provider_transfer" model="payment.provider">
        <field name="name">Wire Transfer</field>
        <field name="sequence">30</field>
        <field name="image_128"
               type="base64"
               file="payment_custom/static/description/icon.png"/>
        <field name="module_id" ref="base.module_payment_custom"/>
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

## File: models\payment_icon.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class PaymentIcon(models.Model):
    _name = 'payment.icon'
    _description = 'Payment Icon'
    _order = 'sequence, name'

    name = fields.Char(string="Name")
    provider_ids = fields.Many2many(
        string="Providers", comodel_name='payment.provider',
        help="The list of providers supporting this payment icon")
    image = fields.Image(
        string="Image", max_width=64, max_height=64,
        help="This field holds the image used for this payment icon, limited to 64x64 px")
    image_payment_form = fields.Image(
        string="Image displayed on the payment form", related='image', store=True, max_width=45,
        max_height=30)
    sequence = fields.Integer('Sequence', default=1)

```

## File: models\payment_provider.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from psycopg2 import sql

from odoo import _, api, fields, models
from odoo.exceptions import UserError, ValidationError
from odoo.osv import expression

_logger = logging.getLogger(__name__)


class PaymentProvider(models.Model):
    _name = 'payment.provider'
    _description = 'Payment Provider'
    _order = 'module_state, state desc, sequence, name'

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
    payment_icon_ids = fields.Many2many(
        string="Supported Payment Icons", comodel_name='payment.icon')
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
    maximum_amount = fields.Monetary(
        string="Maximum Amount",
        help="The maximum payment amount that this payment provider is available for. Leave blank "
             "to make it available for any payment amount.",
        currency_field='main_currency_id',
    )

    # Fees fields
    fees_active = fields.Boolean(string="Add Extra Fees")
    fees_dom_fixed = fields.Float(string="Fixed domestic fees")
    fees_dom_var = fields.Float(string="Variable domestic fees (in percents)")
    fees_int_fixed = fields.Float(string="Fixed international fees")
    fees_int_var = fields.Float(string="Variable international fees (in percents)")

    # Message fields
    display_as = fields.Char(
        string="Displayed as", help="Description of the provider for customers",
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
    support_tokenization = fields.Boolean(
        string="Tokenization Supported", compute='_compute_feature_support_fields'
    )
    support_manual_capture = fields.Boolean(
        string="Manual Capture Supported", compute='_compute_feature_support_fields'
    )
    support_express_checkout = fields.Boolean(
        string="Express Checkout Supported", compute='_compute_feature_support_fields'
    )
    support_refund = fields.Selection(
        string="Type of Refund Supported",
        selection=[('full_only', "Full Only"), ('partial', "Partial")],
        compute='_compute_feature_support_fields',
    )
    support_fees = fields.Boolean(
        string="Fees Supported", compute='_compute_feature_support_fields'
    )

    # Kanban view fields
    image_128 = fields.Image(string="Image", max_width=128, max_height=128)
    color = fields.Integer(
        string="Color", help="The color of the card in kanban view", compute='_compute_color',
        store=True)

    # Module-related fields
    module_id = fields.Many2one(string="Corresponding Module", comodel_name='ir.module.module')
    module_state = fields.Selection(
        string="Installation State", related='module_id.state', store=True)  # Stored for sorting.
    module_to_buy = fields.Boolean(string="Odoo Enterprise Module", related='module_id.to_buy')

    # View configuration fields
    show_credentials_page = fields.Boolean(compute='_compute_view_configuration_fields')
    show_allow_tokenization = fields.Boolean(compute='_compute_view_configuration_fields')
    show_allow_express_checkout = fields.Boolean(compute='_compute_view_configuration_fields')
    show_payment_icon_ids = fields.Boolean(compute='_compute_view_configuration_fields')
    show_pre_msg = fields.Boolean(compute='_compute_view_configuration_fields')
    show_pending_msg = fields.Boolean(compute='_compute_view_configuration_fields')
    show_auth_msg = fields.Boolean(compute='_compute_view_configuration_fields')
    show_done_msg = fields.Boolean(compute='_compute_view_configuration_fields')
    show_cancel_msg = fields.Boolean(compute='_compute_view_configuration_fields')

    #=== COMPUTE METHODS ===#

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
    def _compute_view_configuration_fields(self):
        """ Compute the view configuration fields based on the provider.

        View configuration fields are used to hide specific elements (notebook pages, fields, etc.)
        from the form view of payment providers. These fields are set to `True` by default and are
        as follows:

        - `show_credentials_page`: Whether the "Credentials" notebook page should be shown.
        - `show_allow_tokenization`: Whether the `allow_tokenization` field should be shown.
        - `show_allow_express_checkout`: Whether the `allow_express_checkout` field should be shown.
        - `show_payment_icon_ids`: Whether the `payment_icon_ids` field should be shown.
        - `show_pre_msg`: Whether the `pre_msg` field should be shown.
        - `show_pending_msg`: Whether the `pending_msg` field should be shown.
        - `show_auth_msg`: Whether the `auth_msg` field should be shown.
        - `show_done_msg`: Whether the `done_msg` field should be shown.
        - `show_cancel_msg`: Whether the `cancel_msg` field should be shown.

        For a provider to hide specific elements of the form view, it must override this method and
        set the related view configuration fields to `False` on the appropriate `payment.provider`
        records.

        :return: None
        """
        self.update({
            'show_credentials_page': True,
            'show_allow_tokenization': True,
            'show_allow_express_checkout': True,
            'show_payment_icon_ids': True,
            'show_pre_msg': True,
            'show_pending_msg': True,
            'show_auth_msg': True,
            'show_done_msg': True,
            'show_cancel_msg': True,
        })

    def _compute_feature_support_fields(self):
        """ Compute the feature support fields based on the provider.

        Feature support fields are used to specify which additional features are supported by a
        given provider. These fields are as follows:

        - `support_express_checkout`: Whether the "express checkout" feature is supported. `False`
          by default.
        - `support_fees`: Whether the "extra fees" feature is supported. `False` by default.
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
        self.update(dict.fromkeys((
            'support_express_checkout',
            'support_fees',
            'support_manual_capture',
            'support_refund',
            'support_tokenization',
        ), None))

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
                            "provider. Archiving tokens is irreversible.", len(related_tokens)
                        )
                    }
                }

    #=== CONSTRAINT METHODS ===#

    @api.constrains('fees_dom_var', 'fees_int_var')
    def _check_fee_var_within_boundaries(self):
        """ Check that variable fees are within realistic boundaries.

        Variable fee values should always be positive and below 100% to respectively avoid negative
        and infinite (division by zero) fee amounts.

        :return None
        """
        for provider in self:
            if any(not 0 <= fee < 100 for fee in (provider.fees_dom_var, provider.fees_int_var)):
                raise ValidationError(_("Variable fees must always be positive and below 100%."))

    #=== CRUD METHODS ===#

    @api.model_create_multi
    def create(self, values_list):
        providers = super().create(values_list)
        providers._check_required_if_provider()
        return providers

    def write(self, values):
        # Handle provider disabling.
        if 'state' in values:
            state_changed_providers = self.filtered(
                lambda p: p.state not in ('disabled', values['state'])
            )  # Don't handle providers being enabled or whose state is not updated.
            state_changed_providers._handle_state_change()

        result = super().write(values)
        self._check_required_if_provider()

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
                required_for_provider_code == provider.code and not provider[field_name]
                for provider in enabled_providers
            ):
                ir_field = self.env['ir.model.fields']._get(self._name, field_name)
                field_names.append(ir_field.field_description)
        if field_names:
            raise ValidationError(
                _("The following fields must be filled: %s", ", ".join(field_names))
            )

    def _handle_state_change(self):
        """ Archive all the payment tokens linked to the providers.

        :return: None
        """
        self.env['payment.token'].search([('provider_id', 'in', self.ids)]).write({'active': False})

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

    #=== BUSINESS METHODS ===#

    @api.model
    def _get_compatible_providers(
        self, company_id, partner_id, amount, currency_id=None, force_tokenization=False,
        is_express_checkout=False, is_validation=False, **kwargs
    ):
        """ Select and return the providers matching the criteria.

        The criteria are that providers must not be disabled, be in the company that is provided,
        and support the country of the partner if it exists. The criteria can be further refined
        by providing the keyword arguments.

        :param int company_id: The company to which providers must belong, as a `res.company` id.
        :param int partner_id: The partner making the payment, as a `res.partner` id.
        :param float amount: The amount to pay. `0` for validation transactions.
        :param int currency_id: The payment currency, if known beforehand, as a `res.currency` id.
        :param bool force_tokenization: Whether only providers allowing tokenization can be matched.
        :param bool is_express_checkout: Whether the payment is made through express checkout.
        :param bool is_validation: Whether the operation is a validation.
        :param dict kwargs: Optional data. This parameter is not used here.
        :return: The compatible providers.
        :rtype: recordset of `payment.provider`
        """
        # Compute the base domain for compatible providers.
        domain = ['&', ('state', 'in', ['enabled', 'test']), ('company_id', '=', company_id)]

        # Handle the is_published state.
        if not self.env.user._is_internal():
            domain = expression.AND([domain, [('is_published', '=', True)]])

        # Handle partner country.
        partner = self.env['res.partner'].browse(partner_id)
        if partner.country_id:  # The partner country must either not be set or be supported.
            domain = expression.AND([
                domain, [
                    '|',
                    ('available_country_ids', '=', False),
                    ('available_country_ids', 'in', [partner.country_id.id]),
                ]
            ])

        # Handle the maximum amount.
        currency = self.env['res.currency'].browse(currency_id).exists()
        if not is_validation and currency:  # The currency is required to convert the amount.
            company = self.env['res.company'].browse(company_id).exists()
            date = fields.Date.context_today(self)
            converted_amount = currency._convert(amount, company.currency_id, company, date)
            domain = expression.AND([
                domain, [
                    '|', '|',
                    ('maximum_amount', '>=', converted_amount),
                    ('maximum_amount', '=', False),
                    ('maximum_amount', '=', 0.),
                ]
            ])

        # Handle tokenization support requirements.
        if force_tokenization or self._is_tokenization_required(**kwargs):
            domain = expression.AND([domain, [('allow_tokenization', '=', True)]])

        # Handle express checkout.
        if is_express_checkout:
            domain = expression.AND([domain, [('allow_express_checkout', '=', True)]])

        compatible_providers = self.env['payment.provider'].search(domain)
        return compatible_providers

    def _is_tokenization_required(self, **kwargs):
        """ Return whether tokenizing the transaction is required given its context.

        For a module to make the tokenization required based on the transaction context, it must
        override this method and return whether it is required.

        :param dict kwargs: The transaction context. This parameter is not used here.
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

    def _compute_fees(self, amount, currency, country):
        """ Compute the transaction fees.

        The computation is based on the fields `fees_dom_fixed`, `fees_dom_var`, `fees_int_fixed`
        and `fees_int_var`, and is performed with the formula
        :code:`fees = (amount * variable / 100.0 + fixed) / (1 - variable / 100.0)` where the values
        of `fixed` and `variable` are taken from either the domestic (`dom`) or international
        (`int`) fields, depending on whether the country matches the company's country.

        For a provider to base the computation on different variables, or to use a different
        formula, it must override this method and return the resulting fees.

        :param float amount: The amount to pay for the transaction.
        :param recordset currency: The currency of the transaction, as a `res.currency` record.
        :param recordset country: The customer country, as a `res.country` record.
        :return: The computed fees.
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

        For a provider to support tokenization, it must override this method and return the
        validation currency. If the validation amount is `0`, it is not necessary to create the
        override.

        Note: `self.ensure_one()`

        :return: The validation currency.
        :rtype: recordset of `res.currency`
        """
        self.ensure_one()
        return self.company_id.currency_id

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
    def _remove_provider(self, provider_code):
        """ Remove the module-specific data of the given provider.

        :param str provider_code: The code of the provider whose data to remove.
        :return: None
        """
        providers = self.search([('code', '=', provider_code)])
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

```

## File: models\payment_token.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from odoo import _, api, fields, models
from odoo.exceptions import UserError


class PaymentToken(models.Model):
    _name = 'payment.token'
    _order = 'partner_id, id desc'
    _description = 'Payment Token'

    provider_id = fields.Many2one(string="Provider", comodel_name='payment.provider', required=True)
    provider_code = fields.Selection(related='provider_id.code')
    payment_details = fields.Char(
        string="Payment Details", help="The clear part of the payment method's payment details.",
    )
    partner_id = fields.Many2one(string="Partner", comodel_name='res.partner', required=True)
    company_id = fields.Many2one(  # Indexed to speed-up ORM searches (from ir_rule or others)
        related='provider_id.company_id', store=True, index=True)
    provider_ref = fields.Char(
        string="Provider Reference", help="The provider reference of the token of the transaction",
        required=True)  # This is not the same thing as the provider reference of the transaction.
    transaction_ids = fields.One2many(
        string="Payment Transactions", comodel_name='payment.transaction', inverse_name='token_id')
    verified = fields.Boolean(string="Verified")
    active = fields.Boolean(string="Active", default=True)

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
                if any(not token.active for token in self):
                    raise UserError(_("A token cannot be unarchived once it has been archived."))
            else:
                # Call the handlers in sudo mode because this method might have been called by RPC.
                self.filtered('active').sudo()._handle_archiving()

        return super().write(values)

    def _handle_archiving(self):
        """ Handle the archiving of tokens.

        For a module to perform additional operations when a token is archived, it must override
        this method.

        :return: None
        """
        return

    def name_get(self):
        return [(token.id, token._build_display_name()) for token in self]

    #=== BUSINESS METHODS ===#

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

from odoo import _, api, fields, models
from odoo.exceptions import UserError, ValidationError
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

    provider_id = fields.Many2one(
        string="Provider", comodel_name='payment.provider', readonly=True, required=True)
    provider_code = fields.Selection(related='provider_id.code')
    company_id = fields.Many2one(  # Indexed to speed-up ORM searches (from ir_rule or others)
        related='provider_id.company_id', store=True, index=True)
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
    fees = fields.Monetary(
        string="Fees", currency_field='currency_id',
        help="The fees amount; set by the system as it depends on the provider", readonly=True)
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
            ('refund', "Refund")
        ],
        readonly=True,
        index=True,
    )
    source_transaction_id = fields.Many2one(
        string="Source Transaction",
        comodel_name='payment.transaction',
        help="The source transaction of related refund transactions",
        readonly=True,
    )
    child_transaction_ids = fields.One2many(
        string="Child Transactions",
        help="The child transactions of the source transaction.",
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
    callback_model_id = fields.Many2one(
        string="Callback Document Model", comodel_name='ir.model', groups='base.group_system')
    callback_res_id = fields.Integer(string="Callback Record ID", groups='base.group_system')
    callback_method = fields.Char(string="Callback Method", groups='base.group_system')
    # Hash for extra security on top of the callback fields' group in case a bug exposes a sudo.
    callback_hash = fields.Char(string="Callback Hash", groups='base.group_system')
    callback_is_done = fields.Boolean(
        string="Callback Done", help="Whether the callback has already been executed",
        groups="base.group_system", readonly=True)

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
            fields=['source_transaction_id'],
            groupby=['source_transaction_id'],
        )
        data = {x['source_transaction_id'][0]: x['source_transaction_id_count'] for x in rg_data}
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

            # Compute fees. For validation transactions, fees are zero.
            if values.get('operation') == 'validation':
                values['fees'] = 0
            else:
                currency = self.env['res.currency'].browse(values.get('currency_id')).exists()
                values['fees'] = provider._compute_fees(
                    values.get('amount', 0), currency, partner.country_id,
                )

            # Include provider-specific create values
            values.update(self._get_specific_create_values(provider.code, values))

            # Generate the hash for the callback if one has be configured on the tx.
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
        # E.g., tx.create(amount=1111.11) ; tx.invalidate_recordset() -> tx.amount == 1111.11
        txs.invalidate_recordset(['amount', 'fees'])

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
            action['view_mode'] = 'tree,form'
            action['domain'] = [('source_transaction_id', '=', self.id)]
        return action

    def action_capture(self):
        """ Check the state of the transactions and request their capture. """
        if any(tx.state != 'authorized' for tx in self):
            raise ValidationError(_("Only authorized transactions can be captured."))

        payment_utils.check_rights_on_recordset(self)
        for tx in self:
            # In sudo mode because we need to be able to read on provider fields.
            tx.sudo()._send_capture_request()

    def action_void(self):
        """ Check the state of the transaction and request to have them voided. """
        if any(tx.state != 'authorized' for tx in self):
            raise ValidationError(_("Only authorized transactions can be voided."))

        payment_utils.check_rights_on_recordset(self)
        for tx in self:
            # In sudo mode because we need to be able to read on provider fields.
            tx.sudo()._send_void_request()

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

    @api.model
    def _generate_callback_hash(self, callback_model_id, callback_res_id, callback_method):
        """ Return the hash for the callback on the transaction.

        :param int callback_model_id: The model on which the callback method is defined, as a
                                      `res.model` id.
        :param int callback_res_id: The record on which the callback method must be called, as an id
                                    of the callback method's model.
        :param str callback_method: The name of the callback method.
        :return: The callback hash.
        :rtype: str
        """
        if callback_model_id and callback_res_id and callback_method:
            model_name = self.env['ir.model'].sudo().browse(callback_model_id).model
            token = f'{model_name}|{callback_res_id}|{callback_method}'
            callback_hash = hmac_tool(self.env(su=True), 'generate_callback_hash', token)
            return callback_hash
        return None

    def _get_processing_values(self):
        """ Return the values used to process the transaction.

        The values are returned as a dict containing entries with the following keys:

        - `provider_id`: The provider handling the transaction, as a `payment.provider` id.
        - `provider_code`: The code of the provider.
        - `reference`: The reference of the transaction.
        - `amount`: The rounded amount of the transaction.
        - `currency_id`: The currency of the transaction, as a `res.currency` id.
        - `partner_id`: The partner making the transaction, as a `res.partner` id.
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

        refund_tx = self._create_refund_transaction(amount_to_refund=amount_to_refund)
        refund_tx._log_sent_message()
        return refund_tx

    def _create_refund_transaction(self, amount_to_refund=None, **custom_create_values):
        """ Create a new transaction with the operation `refund` and the current transaction as
        source transaction.

        :param float amount_to_refund: The strictly positive amount to refund, in the same currency
                                       as the source transaction.
        :return: The refund transaction.
        :rtype: recordset of `payment.transaction`
        """
        self.ensure_one()

        return self.create({
            'provider_id': self.provider_id.id,
            'reference': self._compute_reference(self.provider_code, prefix=f'R-{self.reference}'),
            'amount': -(amount_to_refund or self.amount),
            'currency_id': self.currency_id.id,
            'token_id': self.token_id.id,
            'operation': 'refund',
            'source_transaction_id': self.id,
            'partner_id': self.partner_id.id,
            **custom_create_values,
        })

    def _send_capture_request(self):
        """ Request the provider handling the transaction to capture the payment.

        For a provider to support authorization, it must override this method and make an API
        request to capture the payment.

        Note: `self.ensure_one()`

        :return: None
        """
        self.ensure_one()
        self._ensure_provider_is_not_disabled()

    def _send_void_request(self):
        """ Request the provider handling the transaction to void the payment.

        For a provider to support authorization, it must override this method and make an API
        request to void the payment.

        Note: `self.ensure_one()`

        :return: None
        """
        self.ensure_one()
        self._ensure_provider_is_not_disabled()

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

    def _handle_notification_data(self, provider_code, notification_data):
        """ Match the transaction with the notification data, update its state and return it.

        :param str provider_code: The code of the provider handling the transaction.
        :param dict notification_data: The notification data sent by the provider.
        :return: The transaction.
        :rtype: recordset of `payment.transaction`
        """
        tx = self._get_tx_from_notification_data(provider_code, notification_data)
        tx._process_notification_data(notification_data)
        tx._execute_callback()
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

    def _set_pending(self, state_message=None):
        """ Update the transactions' state to `pending`.

        :param str state_message: The reason for setting the transactions in the state `pending`.
        :return: The updated transactions.
        :rtype: recordset of `payment.transaction`
        """
        allowed_states = ('draft',)
        target_state = 'pending'
        txs_to_process = self._update_state(allowed_states, target_state, state_message)
        txs_to_process._log_received_message()
        return txs_to_process

    def _set_authorized(self, state_message=None):
        """ Update the transactions' state to `authorized`.

        :param str state_message: The reason for setting the transactions in the state `authorized`.
        :return: The updated transactions.
        :rtype: recordset of `payment.transaction`
        """
        allowed_states = ('draft', 'pending')
        target_state = 'authorized'
        txs_to_process = self._update_state(allowed_states, target_state, state_message)
        txs_to_process._log_received_message()
        return txs_to_process

    def _set_done(self, state_message=None):
        """ Update the transactions' state to `done`.

        :param str state_message: The reason for setting the transactions in the state `done`.
        :return: The updated transactions.
        :rtype: recordset of `payment.transaction`
        """
        allowed_states = ('draft', 'pending', 'authorized', 'error', 'cancel')  # 'cancel' for Payulatam
        target_state = 'done'
        txs_to_process = self._update_state(allowed_states, target_state, state_message)
        txs_to_process._log_received_message()
        return txs_to_process

    def _set_canceled(self, state_message=None):
        """ Update the transactions' state to `cancel`.

        :param str state_message: The reason for setting the transactions in the state `cancel`.
        :return: The updated transactions.
        :rtype: recordset of `payment.transaction`
        """
        allowed_states = ('draft', 'pending', 'authorized', 'done')  # 'done' for Authorize refunds.
        target_state = 'cancel'
        txs_to_process = self._update_state(allowed_states, target_state, state_message)
        # Cancel the existing payments.
        txs_to_process._log_received_message()
        return txs_to_process

    def _set_error(self, state_message):
        """ Update the transactions' state to `error`.

        :param str state_message: The reason for setting the transactions in the state `error`.
        :return: The updated transactions.
        :rtype: recordset of `payment.transaction`
        """
        allowed_states = ('draft', 'pending', 'authorized', 'done')  # 'done' for Stripe refunds.
        target_state = 'error'
        txs_to_process = self._update_state(allowed_states, target_state, state_message)
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
        })
        return txs_to_process

    def _execute_callback(self):
        """ Execute the callbacks defined on the transactions.

        Callbacks that have already been executed are silently ignored. For example, the callback is
        called twice when a transaction is first authorized then confirmed.

        Only successful callbacks are marked as done. This allows callbacks to reschedule
        themselves, should the conditions be unmet in the present call.

        :return: None
        """
        for tx in self.filtered(lambda t: not t.sudo().callback_is_done):
            # Only use sudo to check, not to execute.
            tx_sudo = tx.sudo()
            model_sudo = tx_sudo.callback_model_id
            res_id = tx_sudo.callback_res_id
            method = tx_sudo.callback_method
            callback_hash = tx_sudo.callback_hash
            if not (model_sudo and res_id and method):
                continue  # Skip transactions with unset (or not properly defined) callbacks.

            valid_callback_hash = self._generate_callback_hash(model_sudo.id, res_id, method)
            if not consteq(ustr(valid_callback_hash), callback_hash):
                _logger.warning(
                    "invalid callback signature for transaction with reference %s", tx.reference
                )
                continue  # Ignore tampered callbacks.

            record = self.env[model_sudo.model].browse(res_id).exists()
            if not record:
                _logger.warning(
                    "invalid callback record %(model)s.%(record_id)s for transaction with "
                    "reference %(ref)s",
                    {
                        'model': model_sudo.model,
                        'record_id': res_id,
                        'ref': tx.reference,
                    }
                )
                continue  # Ignore invalidated callbacks.

            success = getattr(record, method)(tx)  # Execute the callback.
            tx_sudo.callback_is_done = success or success is None  # Missing returns are successful.

    #=== BUSINESS METHODS - POST-PROCESSING ===#

    def _get_post_processing_values(self):
        """ Return a dict of values used to display the status of the transaction.

        For a provider to handle transaction status display, it must override this method and
        return a dict of values. Provider-specific values take precedence over those of the dict of
        generic post-processing values.

        The returned dict contains the following entries:

        - `provider_code`: The code of the provider.
        - `reference`: The reference of the transaction.
        - `amount`: The rounded amount of the transaction.
        - `currency_id`: The currency of the transaction, as a `res.currency` id.
        - `state`: The transaction state: `draft`, `pending`, `authorized`, `done`, `cancel`, or
          `error`.
        - `state_message`: The information message about the state.
        - `operation`: The operation of the transaction.
        - `is_post_processed`: Whether the transaction has already been post-processed.
        - `landing_route`: The route the user is redirected to after the transaction.
        - Additional provider-specific entries.

        Note: `self.ensure_one()`

        :return: The dict of processing values.
        :rtype: dict
        """
        self.ensure_one()

        post_processing_values = {
            'provider_code': self.provider_code,
            'reference': self.reference,
            'amount': self.amount,
            'currency_code': self.currency_id.name,
            'state': self.state,
            'state_message': self.state_message,
            'operation': self.operation,
            'is_post_processed': self.is_post_processed,
            'landing_route': self.landing_route,
        }
        _logger.debug(
            "post-processing values of transaction with reference %s for provider with id %s:\n%s",
            self.reference, self.provider_id.id, pprint.pformat(post_processing_values)
        )  # DEBUG level because this can get spammy with transactions in non-final states
        return post_processing_values

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
            except psycopg2.OperationalError:
                self.env.cr.rollback()  # Rollback and try later.
            except Exception as e:
                _logger.exception(
                    "encountered an error while post-processing transaction with reference %s:\n%s",
                    tx.reference, e
                )
                self.env.cr.rollback()

    def _finalize_post_processing(self):
        """ Trigger the final post-processing tasks and mark the transactions as post-processed.

        :return: None
        """
        self.filtered(lambda tx: tx.operation != 'validation')._reconcile_after_done()
        self.is_post_processed = True

    def _reconcile_after_done(self):
        """ Perform compute-intensive operations on related documents.

        For a provider to handle transaction post-processing, it must overwrite this method and
        execute its compute-intensive operations on documents linked to confirmed transactions.

        :return: None
        """
        return

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
                message += "<br />" + _("Error: %s", self.state_message)
        else:
            message = _(
                ("The transaction with reference %(ref)s for %(amount)s is canceled "
                "(%(provider_name)s)."),
                ref=self.reference,
                amount=formatted_amount,
                provider_name=self.provider_id.name
            )
            if self.state_message:
                message += "<br />" + _("Reason: %s", self.state_message)
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

    payment_provider_onboarding_state = fields.Selection(
        string="State of the onboarding payment provider step",
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

    def _run_payment_onboarding_step(self, menu_id):
        """ Install the suggested payment modules and configure the providers.

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

        stripe_provider = new_env['payment.provider'].search(
            [('company_id', '=', self.env.company.id), ('code', '=', 'stripe')], limit=1
        )
        if not stripe_provider:
            base_provider = self.env.ref('payment.payment_provider_stripe')
            # Use sudo to access payment provider record that can be in different company.
            stripe_provider = base_provider.sudo().with_context(
                stripe_connect_onboarding=True,
            ).copy(default={'company_id': self.env.company.id})
        stripe_provider.journal_id = stripe_provider.journal_id or default_journal

        return stripe_provider.action_stripe_connect_account(menu_id=menu_id)

    def _install_modules(self, module_names):
        modules_sudo = self.env['ir.module.module'].sudo().search([('name', 'in', module_names)])
        STATES = ['installed', 'to install', 'to upgrade']
        modules_sudo.filtered(lambda m: m.state not in STATES).button_immediate_install()

    def _mark_payment_onboarding_step_as_done(self):
        """ Mark the payment onboarding step as done.

        :return: None
        """
        self.set_onboarding_step_done('payment_provider_onboarding_state')

    def get_account_invoice_onboarding_steps_states_names(self):
        """ Override of account. """
        steps = super().get_account_invoice_onboarding_steps_states_names()
        return steps + ['payment_provider_onboarding_state']

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

from . import ir_http
from . import payment_provider
from . import payment_icon
from . import payment_token
from . import payment_transaction
from . import res_company
from . import res_partner

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_payment_link_wizard,access_payment_link_wizard,payment.model_payment_link_wizard,base.group_user,0,0,0,0
payment_provider_onboarding_wizard,payment.provider.onboarding.wizard,model_payment_provider_onboarding_wizard,base.group_system,1,1,1,0
payment_provider_system,payment.provider.system,model_payment_provider,base.group_system,1,1,1,1
payment_icon_all,payment.icon.all,model_payment_icon,,1,0,0,0
payment_icon_system,payment.icon.system,model_payment_icon,base.group_system,1,1,1,1
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

    <!-- Providers -->

    <record id="payment_provider_company_rule" model="ir.rule">
        <field name="name">Access providers in own companies only</field>
        <field name="model_id" ref="payment.model_payment_provider"/>
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>

    <!-- Transactions -->

    <record id="payment_transaction_user_rule" model="ir.rule">
        <field name="name">Access own transactions only</field>
        <field name="model_id" ref="payment.model_payment_transaction"/>
        <field name="domain_force">['|', ('partner_id', '=', False), ('partner_id', '=', user.partner_id.id)]</field>
        <field name="groups" eval="[(4, ref('base.group_user')), (4, ref('base.group_portal'))]"/>
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
                    `#o_payment_provider_inline_form_${paymentOptionId}` // Only match provider radios
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

## File: static\src\js\express_checkout_form.js

```javascript
/** @odoo-module */

import core from 'web.core';
import publicWidget from 'web.public.widget';

publicWidget.registry.PaymentExpressCheckoutForm = publicWidget.Widget.extend({
    selector: 'form[name="o_payment_express_checkout_form"]',

    /**
     * @override
     */
    start: async function () {
        await this._super(...arguments);
        this.txContext = {};
        Object.assign(this.txContext, this.$el.data());
        this.txContext.shippingInfoRequired = !!this.txContext.shippingInfoRequired;
        const expressCheckoutForms = this._getExpressCheckoutForms();
        for (const expressCheckoutForm of expressCheckoutForms) {
            await this._prepareExpressCheckoutForm(expressCheckoutForm.dataset);
        }
        // Monitor updates of the amount on eCommerce's cart pages.
        core.bus.on('cart_amount_changed', this, this._updateAmount.bind(this));
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
     * @return {Promise}
     */
    async _prepareExpressCheckoutForm(providerData) {
        return Promise.resolve();
    },

    /**
     * Prepare the params to send to the transaction route.
     *
     * For a provider to overwrite generic params or to add provider-specific ones, it must override
     * this method and return the extended transaction route params.
     *
     * @private
     * @param {number} providerId - The id of the provider handling the transaction.
     * @returns {object} - The transaction route params
     */
    _prepareTransactionRouteParams(providerId) {
        return {
            'payment_option_id': parseInt(providerId),
            'reference_prefix': this.txContext.referencePrefix &&
                                this.txContent.referencePrefix.toString(),
            'currency_id': this.txContext.currencyId &&
                           parseInt(this.txContext.currencyId),
            'partner_id': parseInt(this.txContext.partnerId),
            'flow': 'direct',
            'tokenization_requested': false,
            'landing_route': this.txContext.landingRoute,
            'access_token': this.txContext.accessToken,
            'csrf_token': core.csrf_token,
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
     * @return {undefined}
     */
    _updateAmount(newAmount, newMinorAmount) {
        this.txContext.amount = parseFloat(newAmount);
        this.txContext.minorAmount = parseInt(newMinorAmount);
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
                    'access_token': this.txContext.accessToken,
                    'token_id': tokenId,
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
         * Build the confirmation dialog based on the linked records' information.
         *
         * @private
         * @param {Array} linkedRecordsInfo - The list of information about linked records.
         * @param confirmCallback - The callback method called when the user clicks on the
         *                          confirmation button.
         * @return {object}
         */
        _buildConfirmationDialog: function (linkedRecordsInfo, confirmCallback) {
            const $dialogContentMessage = $(
                '<span>', {text: _t("Are you sure you want to delete this payment method?")}
            );
            if (linkedRecordsInfo.length > 0) { // List the documents linked to the token.
                $dialogContentMessage.append($('<br>'));
                $dialogContentMessage.append($(
                    '<span>', {text: _t("It is currently linked to the following documents:")}
                ));
                const $documentInfoList = $('<ul>');
                linkedRecordsInfo.forEach(documentInfo => {
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
            return new Dialog(this, {
                title: _t("Warning!"),
                size: 'medium',
                $content: $('<div>').append($dialogContentMessage),
                buttons: [
                    {
                        text: _t("Confirm Deletion"), classes: 'btn-primary', close: true,
                        click: confirmCallback,
                    },
                    {
                        text: _t("Cancel"), close: true
                    },
                ],
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
            }).then(linkedRecordsInfo => {
                this._buildConfirmationDialog(linkedRecordsInfo, execute).open();
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
            this.$('[data-bs-toggle="tooltip"]').tooltip();
            const $checkedRadios = this.$('input[name="o_payment_radio"]:checked');
            if ($checkedRadios.length === 1) {
                const checkedRadio = $checkedRadios[0];
                this._displayInlineForm(checkedRadio);
                this._enableButton();
            } else {
                this._setPaymentFlow(); // Initialize the payment flow to let providers overwrite it
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
        _disableButton: function (showLoadingAnimation = true) {
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
            this._setPaymentFlow(); // Reset the payment flow to let providers overwrite it

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
         * As some providers implement both the direct payment and the payment with redirection, the
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
         * @return {number} The provider id or the token id or of the payment option linked to the
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
         * Remove the error in the provider form.
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
         * For a provider to overwrite generic params or to add provider-specific ones, it must
         * override this method and return the extended transaction route params.
         *
         * @private
         * @param {string} code - The code of the selected payment option provider
         * @param {number} paymentOptionId - The id of the selected payment option
         * @param {string} flow - The online payment flow of the selected payment option
         * @return {object} The transaction route params
         */
        _prepareTransactionRouteParams: function (code, paymentOptionId, flow) {
            return {
                'payment_option_id': paymentOptionId,
                'reference_prefix': this.txContext.referencePrefix !== undefined
                    ? this.txContext.referencePrefix.toString() : null,
                'amount': this.txContext.amount !== undefined
                    ? parseFloat(this.txContext.amount) : null,
                'currency_id': this.txContext.currencyId
                    ? parseInt(this.txContext.currencyId) : null,
                'partner_id': parseInt(this.txContext.partnerId),
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
         * Prepare the provider-specific inline form of the selected payment option.
         *
         * For a provider to manage an inline form, it must override this method. When the override
         * is called, it must lookup the parameters to decide whether it is necessary to prepare its
         * inline form. Otherwise, the call must be sent back to the parent method.
         *
         * @private
         * @param {string} code - The code of the selected payment option's provider
         * @param {number} paymentOptionId - The id of the selected payment option
         * @param {string} flow - The online payment flow of the selected payment option
         * @return {Promise}
         */
        _prepareInlineForm: (code, paymentOptionId, flow) => Promise.resolve(),

        /**
         * Process the payment.
         *
         * For a provider to do pre-processing work on the transaction processing flow, or to
         * define its entire own flow that requires re-scheduling the RPC to the transaction route,
         * it must override this method.
         * If only post-processing work is needed, an override of `_processRedirectPayment`,
         * `_processDirectPayment` or `_processTokenPayment` might be more appropriate.
         *
         * @private
         * @param {string} code - The code of the payment option's provider
         * @param {number} paymentOptionId - The id of the payment option handling the transaction
         * @param {string} flow - The online payment flow of the transaction
         * @return {Promise}
         */
        _processPayment: function (code, paymentOptionId, flow) {
            // Call the transaction route to create a tx and retrieve the processing values
            return this._rpc({
                route: this.txContext.transactionRoute,
                params: this._prepareTransactionRouteParams(code, paymentOptionId, flow),
            }).then(processingValues => {
                if (flow === 'redirect') {
                    return this._processRedirectPayment(
                        code, paymentOptionId, processingValues
                    );
                } else if (flow === 'direct') {
                    return this._processDirectPayment(code, paymentOptionId, processingValues);
                } else if (flow === 'token') {
                    return this._processTokenPayment(code, paymentOptionId, processingValues);
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
         * Execute the provider-specific implementation of the direct payment flow.
         *
         * For a provider to redefine the processing of the direct payment flow, it must override
         * this method.
         *
         * @private
         * @param {string} code - The code of the provider
         * @param {number} providerId - The id of the provider handling the transaction
         * @param {object} processingValues - The processing values of the transaction
         * @return {Promise}
         */
        _processDirectPayment: (code, providerId, processingValues) => Promise.resolve(),

        /**
         * Redirect the customer by submitting the redirect form included in the processing values.
         *
         * For a provider to redefine the processing of the payment with redirection flow, it must
         * override this method.
         *
         * @private
         * @param {string} code - The code of the provider
         * @param {number} providerId - The id of the provider handling the transaction
         * @param {object} processingValues - The processing values of the transaction
         * @return {undefined}
         */
        _processRedirectPayment: (code, providerId, processingValues) => {
            // Append the redirect form to the body
            const $redirectForm = $(processingValues.redirect_form_html).attr(
                'id', 'o_payment_redirect_form'
            );
            // Ensures external redirections when in an iframe.
            $redirectForm[0].setAttribute('target', '_top');
            $(document.getElementsByTagName('body')[0]).append($redirectForm);

            // Submit the form
            $redirectForm.submit();
        },

        /**
         * Redirect the customer to the status route.
         *
         * For a provider to redefine the processing of the payment by token flow, it must override
         * this method.
         *
         * @private
         * @param {string} provider_code - The code of the token's provider
         * @param {number} tokenId - The id of the token handling the transaction
         * @param {object} processingValues - The processing values of the transaction
         * @return {undefined}
         */
        _processTokenPayment: (provider_code, tokenId, processingValues) => {
            // The flow is already completed as payments by tokens are immediately processed
            window.location = '/payment/status';
        },

        /**
         * Set the online payment flow for the selected payment option.
         *
         * For a provider to manage direct payments, it must call this method from within its
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
         * Hide all extra payment icons of the provider linked to the clicked button.
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
         * Display all the payment icons of the provider linked to the clicked button.
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

    return publicWidget.registry.PaymentPostProcessing;
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
                <h1>Failed operations</h1>
                <ul class="list-group">
                    <t t-foreach="tx_error" t-as="tx">
                        <a t-att-href="tx['landing_route']" class="list-group-item">
                            <h4 class="list-group-item-heading mb5">
                                <t t-esc="tx['reference']"/>
                                <span class="badge text-bg-light float-end"><t t-esc="tx['amount']"/> <t t-esc="tx['currency_code']"/></span>
                            </h4>
                            <small class="list-group-item-text">
                                An error occurred during the processing of this payment.<br/>
                                <strong>Reason:</strong> <t t-esc="tx['state_message']"/>
                            </small>
                        </a>
                    </t>
                </ul>
            </div>
            <!-- Pending/Authorized/Confirmed transactions -->
            <div t-if="tx_done.length > 0 || tx_authorized.length > 0 || tx_pending.length > 0">
                <h1>Operations in progress</h1>
                <div class="list-group">
                    <!-- Done transactions -->
                    <t t-foreach="tx_done" t-as="tx">
                        <a t-att-href="tx['landing_route']" class="list-group-item">
                            <h4 class="list-group-item-heading mb5">
                                <t t-esc="tx['reference']"/>
                                <span class="badge text-bg-light float-end"><t t-esc="tx['amount']"/> <t t-esc="tx['currency_code']"/></span>
                            </h4>
                            <small class="list-group-item-text">
                                <t t-if="!tx['is_post_processed']">
                                    <t t-if="tx['operation'] != 'validation'">
                                        Your payment is being processed, please wait... <i class="fa fa-cog fa-spin"/>
                                    </t>
                                    <t t-else="">
                                        Saving your payment method, please wait... <i class="fa fa-cog fa-spin"/>
                                    </t>
                                </t>
                                <t t-else="">
                                    <t t-if="tx['operation'] != 'validation'">
                                        Your payment has been processed.<br/>
                                        Click here to be redirected to the confirmation page.
                                    </t>
                                    <t t-else="">
                                        Your payment method has been saved.<br/>
                                        Click here to be redirected to the confirmation page.
                                    </t>
                                </t>
                            </small>
                        </a>
                    </t>
                    <!-- Pending transactions -->
                    <t t-foreach="tx_pending" t-as="tx">
                        <a t-att-href="tx['landing_route']" class="list-group-item">
                            <h4 class="list-group-item-heading mb5">
                                <t t-esc="tx['reference']"/>
                                <span class="badge text-bg-light float-end"><t t-esc="tx['amount']"/> <t t-esc="tx['currency_code']"/></span>
                            </h4>
                            <small class="list-group-item-text">
                                <t t-if="tx['display_message']">
                                    <!-- display_message is the content of the HTML field associated
                                    with the current transaction state, set on the provider. -->
                                    <t t-out="tx['display_message']"/>
                                </t>
                                <t t-else="">
                                    Your payment is in pending state.<br/>
                                    You will be notified when the payment is fully confirmed.<br/>
                                    Click here to be redirected to the confirmation page.
                                </t>
                            </small>
                        </a>
                    </t>
                    <!-- Authorized transactions -->
                    <t t-foreach="tx_authorized" t-as="tx">
                        <a t-att-href="tx['landing_route']" class="list-group-item">
                            <h4 class="list-group-item-heading mb5">
                                <t t-esc="tx['reference']"/>
                                <span class="badge text-bg-light float-end"><t t-esc="tx['amount']"/> <t t-esc="tx['currency_code']"/></span>
                            </h4>
                            <small class="list-group-item-text">
                                <t t-if="tx['display_message']">
                                    <!-- display_message is the content of the HTML field associated
                                    with the current transaction state, set on the provider. -->
                                    <t t-out="tx['display_message']"/>
                                </t>
                                <t t-else="">
                                    Your payment has been received but need to be confirmed manually.<br/>
                                    You will be notified when the payment is confirmed.
                                </t>
                            </small>
                        </a>
                    </t>
                </div>
            </div>
            <!-- Draft transactions -->
            <div t-if="tx_draft.length > 0">
                <h1>Waiting for operations to process</h1>
                <ul class="list-group">
                    <t t-foreach="tx_draft" t-as="tx">
                        <a t-att-href="tx['landing_route']" class="list-group-item">
                            <h4 class="list-group-item-heading mb5">
                                <t t-esc="tx['reference']"/>
                                <span class="badge text-bg-light float-end"><t t-esc="tx['amount']"/> <t t-esc="tx['currency_code']"/></span>
                            </h4>
                            <small class="list-group-item-text">
                                <t t-if="tx['display_message']">
                                    <!-- display_message is the content of the HTML field associated
                                    with the current transaction state, set on the provider. -->
                                    <t t-out="tx['display_message']"/>
                                </t>
                                <t t-else="">
                                    We are waiting for the payment provider to confirm the payment.
                                </t>
                            </small>
                        </a>
                    </t>
                </ul>
            </div>
            <!-- Cancel transactions -->
            <div t-if="tx_cancel.length > 0">
                <h1>Canceled operations</h1>
                <ul class="list-group">
                    <t t-foreach="tx_cancel" t-as="tx">
                        <a t-att-href="tx['landing_route']" class="list-group-item">
                            <h4 class="list-group-item-heading mb5">
                                <t t-esc="tx['reference']"/>
                                <span class="badge text-bg-light float-end"><t t-esc="tx['amount']"/> <t t-esc="tx['currency_code']"/></span>
                            </h4>
                            <small class="list-group-item-text">
                                This payment has been canceled.<br/>
                                No payment has been processed.<br/>
                                <t t-if="tx['state_message']">
                                    <strong>Reason:</strong> <t t-esc="tx['state_message']"/>
                                </t>
                            </small>
                        </a>
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
                        <page string="Providers list" name="providers">
                            <field nolabel="1" name="provider_ids"/>
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
            <t t-set="providers_allowing_tokenization"
               t-value="request.env['payment.provider'].sudo()._get_compatible_providers(request.env.company.id, partner.id, 0., force_tokenization=True, is_validation=True)"/>
            <t t-set="existing_tokens" t-value="partner.payment_token_ids + partner.commercial_partner_id.sudo().payment_token_ids"/>
            <!-- Only show the link if a token can be created or if one already exists -->
            <div t-if="providers_allowing_tokenization or existing_tokens"
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
                            <div t-elif="not providers and not tokens" class="alert alert-warning">
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
                            <t t-if="providers or tokens" t-call="payment.manage"/>
                            <div t-else="" class="alert alert-warning">
                                <p><strong>No suitable payment provider could be found.</strong></p>
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
                             class="col-sm-6 offset-sm-3">
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
                                <div class="mb-3 row">
                                    <label for="form_partner_name" class="col-md-3 col-form-label">
                                        From
                                    </label>
                                    <span name="form_partner_name"
                                          class="col-md-9 col-form-label"
                                          t-esc="tx.partner_name"/>
                                </div>
                                <hr/>
                                <div class="mb-3 row">
                                    <label for="form_reference" class="col-md-3 col-form-label">
                                        Reference
                                    </label>
                                    <span name="form_reference"
                                          class="col-md-9 col-form-label"
                                          t-esc="tx.reference"/>
                                </div>
                                <hr/>
                                <div class="mb-3 row">
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
                                        Processed by <t t-esc="tx.provider_id.sudo().name"/>
                                    </div>
                                    <div class="col-md-4 offset-md-3 mt-2 ps-0">
                                        <a role="button"
                                           t-attf-class="btn btn-primary float-end"
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

## File: views\payment_provider_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_provider_form" model="ir.ui.view">
        <field name="name">payment.provider.form</field>
        <field name="model">payment.provider</field>
        <field name="arch" type="xml">
            <form string="Payment provider">
                <field name="company_id" invisible="1"/>
                <field name="is_published" invisible="1"/>
                <field name="main_currency_id" invisible="1"/>
                <field name="support_fees" invisible="1"/>
                <field name="support_manual_capture" invisible="1"/>
                <field name="support_tokenization" invisible="1"/>
                <field name="support_express_checkout" invisible="1"/>
                <field name="module_id" invisible="1"/>
                <field name="module_state" invisible="1"/>
                <field name="module_to_buy" invisible="1"/>
                <field name="show_credentials_page" invisible="1"/>
                <field name="show_allow_express_checkout" invisible="1"/>
                <field name="show_allow_tokenization" invisible="1"/>
                <field name="show_payment_icon_ids" invisible="1"/>
                <field name="show_pre_msg" invisible="1"/>
                <field name="show_pending_msg" invisible="1"/>
                <field name="show_auth_msg" invisible="1"/>
                <field name="show_done_msg" invisible="1"/>
                <field name="show_cancel_msg" invisible="1"/>
                <field name="code" invisible="1"/>
                <sheet>
                    <!-- === Stat Buttons === -->
                    <div class="oe_button_box" name="button_box"
                         attrs="{'invisible': [('module_state', '!=', 'installed')]}">
                        <button name="action_toggle_is_published"
                                attrs="{'invisible': [('is_published', '=', False)]}"
                                class="oe_stat_button"
                                type="object"
                                icon="fa-globe">
                            <div class="o_stat_info o_field_widget">
                                <span class="text-success">Published</span>
                            </div>
                        </button>
                        <button name="action_toggle_is_published"
                                attrs="{'invisible': [('is_published', '=', True)]}"
                                class="oe_stat_button"
                                type="object"
                                icon="fa-eye-slash">
                            <div class="o_stat_info o_field_widget">
                                <span class="text-danger">Unpublished</span>
                            </div>
                        </button>
                    </div>
                    <field name="image_128" widget="image" class="oe_avatar"
                           attrs="{'readonly': [('module_state', '!=', 'installed')]}"/>
                    <widget name="web_ribbon" title="Disabled" bg_color="bg-danger" attrs="{'invisible': ['|', ('module_state', '!=', 'installed'), ('state', '!=', 'disabled')]}"/>
                    <widget name="web_ribbon" title="Test Mode" bg_color="bg-warning" attrs="{'invisible': ['|', ('module_state', '!=', 'installed'), ('state', '!=', 'test')]}"/>
                    <div class="oe_title">
                        <h1><field name="name" placeholder="Name"/></h1>
                        <div attrs="{'invisible': ['|', ('module_state', '=', 'installed'), ('module_id', '=', False)]}">
                            <a attrs="{'invisible': [('module_to_buy', '=', False)]}" href="https://odoo.com/pricing?utm_source=db&amp;utm_medium=module" target="_blank" class="btn btn-info" role="button">Upgrade</a>
                            <button attrs="{'invisible': [('module_to_buy', '=', True)]}" type="object" class="btn btn-primary" name="button_immediate_install" string="Install"/>
                        </div>
                    </div>
                    <div id="provider_creation_warning" attrs="{'invisible': [('id', '!=', False)]}" class="alert alert-warning" role="alert">
                        <strong>Warning</strong> Creating a payment provider from the <em>CREATE</em> button is not supported.
                        Please use the <em>Duplicate</em> action instead.
                    </div>
                    <group>
                        <group name="payment_state" attrs="{'invisible': [('module_state', 'not in', ('installed', False))]}">
                            <field name="code" groups="base.group_no_one" attrs="{'readonly': [('id', '!=', False)]}"/>
                            <field name="state" widget="radio"/>
                            <field name="company_id" groups="base.group_multi_company" options='{"no_open":True}'/>
                        </group>
                    </group>
                    <notebook attrs="{'invisible': ['&amp;', ('module_id', '!=', False), ('module_state', '!=', 'installed')]}">
                        <page string="Credentials" name="credentials" attrs="{'invisible': ['|', ('code', '=', 'none'), ('show_credentials_page', '=', False)]}">
                            <group name="provider_credentials"/>
                        </page>
                        <page string="Configuration" name="configuration">
                            <group name="provider_config">
                                <group string="Payment Form" name="payment_form">
                                    <field name="display_as" placeholder="If not defined, the provider name will be used."/>
                                    <field name="payment_icon_ids" attrs="{'invisible': [('show_payment_icon_ids', '=', False)]}" widget="many2many_tags"/>
                                    <field name="allow_tokenization" attrs="{'invisible': ['|', ('support_tokenization', '=', False), ('show_allow_tokenization', '=', False)]}"/>
                                    <field name="capture_manually" attrs="{'invisible': [('support_manual_capture', '=', False)]}"/>
                                    <field name="allow_express_checkout" attrs="{'invisible': ['|', ('support_express_checkout', '=', False), ('show_allow_express_checkout', '=', False)]}"/>
                                </group>
                                <group string="Availability" name="availability">
                                    <field name="maximum_amount"/>
                                    <field name="available_country_ids"
                                           widget="many2many_tags"
                                           placeholder="Select countries. Leave empty to make available everywhere."
                                           options="{'no_open': True, 'no_create': True}"/>
                                </group>
                                <group string="Payment Followup" name="payment_followup" invisible="1"/>
                            </group>
                        </page>
                        <page string="Fees" name="fees" attrs="{'invisible': [('support_fees', '=', False)]}">
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
                                <field name="auth_msg" attrs="{'invisible': ['|', ('support_manual_capture', '=', False), ('show_auth_msg', '=', False)]}"/>
                                <field name="done_msg" attrs="{'invisible': [('show_done_msg', '=', False)]}"/>
                                <field name="cancel_msg" attrs="{'invisible': [('show_cancel_msg', '=', False)]}"/>
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
            <tree string="Payment Providers" create="false">
                <field name="sequence" widget="handle"/>
                <field name="name"/>
                <field name="code"/>
                <field name="state"/>
                <field name="available_country_ids" widget="many2many_tags" optional="hide"/>
                <field name="company_id" groups="base.group_multi_company" optional="show"/>
            </tree>
        </field>
    </record>

    <record id="payment_provider_kanban" model="ir.ui.view">
        <field name="name">payment.provider.kanban</field>
        <field name="model">payment.provider</field>
        <field name="arch" type="xml">
            <kanban create="false" quick_create="false" class="o_kanban_dashboard">
                <field name="id"/>
                <field name="name"/>
                <field name="state"/>
                <field name="is_published"/>
                <field name="code"/>
                <field name="module_id"/>
                <field name="module_state"/>
                <field name="module_to_buy"/>
                <field name="color"/>
                <templates>
                    <t t-name="kanban-box">
                        <t t-set="installed" t-value="!record.module_id.value || (record.module_id.value &amp;&amp; record.module_state.raw_value === 'installed')"/>
                        <t t-set="to_buy" t-value="record.module_to_buy.raw_value === true"/>
                        <t t-set="is_disabled" t-value="record.state.raw_value=='disabled'"/>
                        <t t-set="is_published" t-value="record.is_published.raw_value === true"/>
                        <t t-set="to_upgrade" t-value="!installed and to_buy"/>
                        <div t-attf-class="oe_kanban_global_click" class="d-flex p-2">
                            <div class="o_payment_provider_desc d-flex gap-2">
                                <img type="open"
                                     t-att-src="kanban_image('payment.provider', 'image_128', record.id.raw_value)"
                                     class="mb-0 o_image_64_max"
                                     alt="provider"/>
                                <div class="d-flex flex-column justify-content-between w-100">
                                    <div class="o_payment_kanban_info">
                                        <h4 class="mb-0"><t t-esc="record.name.value"/></h4>
                                        <t t-if="installed">
                                            <field name="state"
                                                   widget="label_selection"
                                                   options="{'classes': {'enabled': 'success', 'test': 'warning', 'disabled' : 'light'}}"/>
                                            <t t-if="!is_disabled">
                                                <span t-if="is_published and installed"
                                                      class="badge text-bg-success ms-1">
                                                    Published
                                                </span>
                                                <span t-if="!is_published and installed"
                                                      class="badge text-bg-info ms-1">
                                                    Unpublished
                                                </span>
                                            </t>
                                        </t>
                                        <span t-if="to_upgrade" class="badge text-bg-primary ms-1">Enterprise</span>
                                    </div>
                                    <div class="o_payment_kanban_button text-end">
                                        <button t-if="!installed and !selection_mode and !to_buy" type="object" class="btn btn-sm btn-primary float-end" name="button_immediate_install">Install</button>
                                        <button t-if="installed and is_disabled and !selection_mode" type="edit" class="btn btn-sm btn-secondary float-end">Activate</button>
                                        <button t-if="!installed and to_buy" href="https://odoo.com/pricing?utm_source=db&amp;utm_medium=module" target="_blank" class="btn btn-sm btn-primary float-end">Upgrade</button>
                                    </div>
                                </div>
                            </div>
                        </div>
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
                <field name="code"/>
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
        <field name="view_mode">kanban,tree,form</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a new payment provider
            </p>
        </field>
    </record>

</odoo>

```

## File: views\payment_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- Checkout form -->
    <template id="checkout" name="Payment Checkout">
        <!-- Variables description:
            - 'providers' - The payment providers compatible with the current transaction
            - 'tokens' - The payment tokens of the current partner and payment providers
            - 'default_token_id' - The id of the token that should be pre-selected. Optional
            - 'fees_by_provider' - The dict of transaction fees for each provider. Optional
            - 'show_tokenize_input' - Whether the option to save the payment method is shown
            - 'reference_prefix' - The custom prefix to compute the full transaction reference
            - 'amount' - The amount to pay. Optional (sale_subscription)
            - 'currency' - The currency of the transaction, as a `res.currency` record
            - 'partner_id' - The id of the partner on behalf of whom the payment should be made
            - 'access_token' - The access token used to authenticate the partner.
            - 'transaction_route' - The route used to create a transaction when the user clicks Pay
            - 'landing_route' - The route the user is redirected to after the transaction
            - 'footer_template_id' - The template id for the submit button. Optional
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
              t-att-data-allow-token-selection="True">

            <t t-set="provider_count" t-value="len(providers) if providers else 0"/>
            <t t-set="token_count" t-value="len(tokens) if tokens else 0"/>
            <!-- Check the radio button of the default token, if set, or of the first provider if
                 it is the only payment option -->
            <t t-set="default_payment_option_id"
               t-value="default_token_id if default_token_id and token_count > 0
                        else providers[0].id if provider_count == 1 and token_count == 0
                        else None"/>
            <t t-set="fees_by_provider" t-value="fees_by_provider or dict()"/>
            <t t-set="footer_template_id"
               t-value="footer_template_id or 'payment.footer'"/>

            <div class="card">
                <!-- === Providers === -->
                <t t-foreach="providers" t-as="provider">
                    <div name="o_payment_option_card" class="card-body o_payment_option_card">
                        <label>
                            <!-- === Radio button === -->
                            <!-- Only shown if linked to the only payment option -->
                            <input name="o_payment_radio"
                                   type="radio"
                                   t-att-checked="provider.id == default_payment_option_id"
                                   t-att-class="'' if provider_count + token_count > 1 else 'd-none'"
                                   t-att-data-payment-option-id="provider.id"
                                   t-att-data-provider="provider.code"
                                   data-payment-option-type="provider"/>
                            <!-- === Provider name === -->
                            <span class="payment_option_name">
                                <b t-esc="provider.display_as or provider.name"/>
                            </span>
                            <!-- === "Test Mode" badge === -->
                            <span t-if="provider.state == 'test'"
                                  class="badge rounded-pill text-bg-warning ms-1">
                                Test Mode
                            </span>
                            <!-- === "Unpublished" badge === -->
                            <span t-if="not provider.is_published"
                                  class="badge rounded-pill text-bg-danger ms-1">
                                Unpublished
                            </span>
                            <!-- === Extra fees badge === -->
                            <t t-if="fees_by_provider.get(provider)">
                                <span class="badge rounded-pill text-bg-secondary ms-1">
                                    + <t t-esc="fees_by_provider.get(provider)"
                                         t-options="{'widget': 'monetary', 'display_currency': currency}"/>
                                    Fees
                                </span>
                            </t>
                        </label>
                        <!-- === Payment icon list === -->
                        <t t-call="payment.icon_list"/>
                        <!-- === Help message === -->
                        <div t-if="not is_html_empty(provider.pre_msg)"
                             t-out="provider.pre_msg"
                             class="text-muted ms-3"/>
                    </div>
                    <!-- === Provider inline form === -->
                    <div t-attf-id="o_payment_provider_inline_form_{{provider.id}}"
                         name="o_payment_inline_form"
                         class="card-footer px-3 d-none">
                        <t t-if="provider.sudo()._should_build_inline_form(is_validation=False)">
                            <t t-set="inline_form_xml_id"
                               t-value="provider.sudo().inline_form_view_id.xml_id"/>
                            <!-- === Inline form content (filled by provider) === -->
                            <div t-if="inline_form_xml_id" class="clearfix">
                                <t t-call="{{inline_form_xml_id}}">
                                    <t t-set="provider_id" t-value="provider.id"/>
                                </t>
                            </div>
                        </t>
                        <!-- === "Save my payment details" checkbox === -->
                        <label t-if="show_tokenize_input[provider.id]">
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
                                   t-att-data-provider="token.provider_code"
                                   data-payment-option-type="token"/>
                            <!-- === Token name === -->
                            <span class="payment_option_name" t-esc="token.display_name"/>
                            <!-- === "V" check mark === -->
                            <t t-call="payment.verified_token_checkmark"/>
                            <!-- === "Fees" badge === -->
                            <span t-if="fees_by_provider.get(token.provider_id)"
                                  class="badge rounded-pill text-bg-secondary ms-1">
                                    + <t t-esc="fees_by_provider.get(token.provider_id)"
                                         t-options="{'widget': 'monetary', 'display_currency': currency}"/>
                                    Fees
                            </span>
                            <!-- === "Unpublished" badge === -->
                            <span t-if="not token.provider_id.is_published" class="badge rounded-pill text-bg-danger ms-1">
                                Unpublished
                            </span>
                        </label>
                    </div>
                    <!-- === Token inline form === -->
                    <div t-attf-id="o_payment_token_inline_form_{{token.id}}"
                         name="o_payment_inline_form"
                         class="card-footer d-none">
                        <t t-set="token_inline_form_xml_id"
                           t-value="token.sudo().provider_id.token_inline_form_view_id.xml_id"/>
                        <!-- === Inline form content (filled by provider) === -->
                        <div t-if="token_inline_form_xml_id" class="clearfix">
                            <t t-call="{{token_inline_form_xml_id}}">
                                <t t-set="token" t-value="token"/>
                            </t>
                        </div>
                    </div>
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
            - 'providers' - The payment providers supporting tokenization
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
              t-att-data-transaction-route="transaction_route"
              t-att-data-assign-token-route="assign_token_route"
              t-att-data-landing-route="landing_route"
              t-att-data-allow-token-selection="bool(assign_token_route)">
            <t t-set="provider_count" t-value="len(providers) if providers else 0"/>
            <t t-set="token_count" t-value="len(tokens) if tokens else 0"/>
            <t t-set="no_selectable_token" t-value="token_count == 0 or not assign_token_route"/>
            <t t-set="default_payment_option_id"
               t-value="default_token_id if default_token_id and token_count > 0
                        else providers[0].id if provider_count == 1 and no_selectable_token
                        else None"/>
            <t t-set="footer_template_id"
               t-value="footer_template_id or 'payment.footer'"/>
            <div class="card">
                <!-- === Providers === -->
                <t t-foreach="providers" t-as="provider">
                    <div name="o_payment_option_card" class="card-body o_payment_option_card">
                        <label>
                            <!-- === Radio button === -->
                            <!-- Only shown if linked to the only payment option -->
                            <input name="o_payment_radio"
                                   type="radio"
                                   t-att-checked="provider.id == default_payment_option_id"
                                   t-att-class="'' if provider_count + token_count > 1 else 'd-none'"
                                   t-att-data-payment-option-id="provider.id"
                                   t-att-data-provider="provider.code"
                                   data-payment-option-type="provider"/>
                            <!-- === Provider name === -->
                            <span class="payment_option_name">
                                <b t-esc="provider.display_as or provider.name"/>
                            </span>
                            <!-- === "Test Mode" badge === -->
                            <span t-if="provider.state == 'test'"
                                  class="badge rounded-pill text-bg-warning"
                                  style="margin-left:5px">
                                Test Mode
                            </span>
                            <!-- === "Unpublished" badge === -->
                            <span t-if="not provider.is_published" class="badge rounded-pill text-bg-danger">
                                Unpublished
                            </span>
                        </label>
                        <!-- === Payment icon list === -->
                        <t t-call="payment.icon_list"/>
                        <!-- === Help message === -->
                        <div t-if="not is_html_empty(provider.pre_msg)"
                             t-out="provider.pre_msg"
                             class="text-muted ms-3"/>
                    </div>
                    <!-- === Provider inline form === -->
                    <t t-if="provider.sudo()._should_build_inline_form(is_validation=True)">
                        <div t-attf-id="o_payment_provider_inline_form_{{provider.id}}"
                             name="o_payment_inline_form"
                             class="card-footer d-none">
                            <t t-set="inline_form_xml_id"
                               t-value="provider.sudo().inline_form_view_id.xml_id"/>
                            <!-- === Inline form content (filled by provider) === -->
                            <div t-if="inline_form_xml_id" class="clearfix">
                                <t t-call="{{inline_form_xml_id}}">
                                    <t t-set="provider_id" t-value="provider.id"/>
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
                                   t-att-data-provider="token.provider_code"
                                   data-payment-option-type="token"/>
                            <!-- === Token name === -->
                            <span class="payment_option_name" t-esc="token.display_name"/>
                            <!-- === "V" check mark === -->
                            <t t-call="payment.verified_token_checkmark"/>
                            <!-- === "Unpublished" badge === -->
                            <span t-if="not token.provider_id.is_published and token.env.user._is_internal()"
                                  class="badge rounded-pill text-bg-danger ms-1">
                                Unpublished
                            </span>
                        </label>
                        <!-- === "Delete" token button === -->
                        <button name="o_payment_delete_token"
                                class="btn btn-primary btn-sm float-end">
                            <i class="fa fa-trash"/> Delete
                        </button>
                    </div>
                    <!-- === Token inline form === -->
                    <div t-attf-id="o_payment_token_inline_form_{{token.id}}"
                         name="o_payment_inline_form"
                         class="card-footer d-none">
                        <t t-set="token_inline_form_xml_id"
                           t-value="token.sudo().provider_id.token_inline_form_view_id.xml_id"/>
                        <!-- === Inline form content (filled by provider) === -->
                        <div t-if="token_inline_form_xml_id" class="clearfix">
                            <t t-call="{{token_inline_form_xml_id}}">
                                <t t-set="token" t-value="token"/>
                            </t>
                        </div>
                    </div>
                </t>
            </div>
            <!-- === "Save Payment Method" button === -->
            <t t-call="{{footer_template_id}}">
                <t t-set="label">Save Payment Method</t>
                <t t-set="icon_class" t-value="'fa-plus-circle'"/>
            </t>
        </form>
    </template>

    <!-- Express Checkout form -->
    <template id="express_checkout" name="Payment Express Checkout">
        <!-- Variables description:
            - 'providers' - The payment providers compatible with the current transaction.
            - 'reference_prefix' - The custom prefix to compute the full transaction reference.
            - 'amount' - The amount to pay.
            - 'minor_amount' - The amount to pay in the minor units of its currency.
            - 'currency' - The currency of the transaction, as a `res.currency` record.
            - 'merchant_name' - The merchant name.
            - 'access_token' - The access token used to authenticate the partner.
            - 'shipping_info_required' - Whether the shipping information is required or not.
            - 'transaction_route' - The route used to create a transaction when the user clicks Pay.
            - 'shipping_address_update_route' - The route where available carriers are computed
                                                based on the (partial) shipping information
                                                available. Optional
            - 'express_checkout_route' - The route where the billing and shipping information are
                                         sent.
            - 'landing_route' - The route the user is redirected to after the transaction.
        -->
        <form name="o_payment_express_checkout_form" class="container"
              t-att-data-reference-prefix="reference_prefix"
              t-att-data-amount="amount"
              t-att-data-minor-amount="minor_amount"
              t-att-data-currency-id="currency and currency.id"
              t-att-data-currency-name="currency.name.lower()"
              t-att-data-merchant-name="merchant_name"
              t-att-data-partner-id="partner_id"
              t-att-data-access-token="payment_access_token"
              t-att-data-shipping-info-required="shipping_info_required"
              t-att-data-transaction-route="transaction_route"
              t-att-data-shipping-address-update-route="shipping_address_update_route"
              t-att-data-express-checkout-route="express_checkout_route"
              t-att-data-landing-route="landing_route">
            <t t-foreach="providers_sudo" t-as="provider_sudo">
                <t t-set="express_checkout_form_xml_id"
                   t-value="provider_sudo.express_checkout_form_view_id.xml_id"/>
                <t t-if="express_checkout_form_xml_id">
                    <t t-call="{{express_checkout_form_xml_id}}">
                        <t t-set="provider_sudo" t-value="provider_sudo"/>
                    </t>
                </t>
            </t>
        </form>
    </template>

    <!-- Expandable payment icon list -->
    <template id="icon_list" name="Payment Icon List">
        <ul class="payment_icon_list float-end list-inline" data-max-icons="3">
            <t t-set="icon_index" t-value="0"/>
            <t t-set="MAX_ICONS" t-value="3"/>
            <!-- === Icons === -->
            <!-- Only shown if in the first 3 icons -->
            <t t-foreach="provider.payment_icon_ids.filtered(lambda r: r.image_payment_form)" t-as="icon">
                <li t-attf-class="list-inline-item{{'' if (icon_index &lt; MAX_ICONS) else ' d-none'}}">
                    <span t-field="icon.image_payment_form"
                          t-options="{'widget': 'image', 'alt-field': 'name'}"
                          data-bs-toggle="tooltip"
                          t-att-title="icon.name"/>
                </li>
                <t t-set="icon_index" t-value="icon_index + 1"/>
            </t>
            <t t-if="icon_index >= MAX_ICONS">
                <!-- === "show more" button === -->
                <!-- Only displayed if too many payment icons -->
                <li style="display:block;" class="list-inline-item">
                    <span class="float-end more_option text-info">
                        <a name="o_payment_icon_more"
                           data-bs-toggle="tooltip"
                           t-att-title="', '.join([icon.name for icon in provider.payment_icon_ids[MAX_ICONS:]])">
                            show more
                        </a>
                    </span>
                </li>
                <!-- === "show less" button === -->
                <!-- Only displayed when "show more" is clicked -->
                <li style="display:block;" class="list-inline-item d-none">
                    <span class="float-end more_option text-info">
                        <a name="o_payment_icon_less">show less</a>
                    </span>
                </li>
            </t>
        </ul>
    </template>

    <!-- Verified token checkmark -->
    <template id="verified_token_checkmark" name="Payment Verified Token Checkmark">
        <t t-if="0" name="payment_demo_hook"/>
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
        <div class="float-end mt-2">
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
        <t t-if="tx.state == 'draft'">
            <t t-set="alert_style" t-value="'info'"/>
            <t t-set="status_message">
                <p>Your payment has not been processed yet.</p>
            </t>
        </t>
        <t t-elif="tx.state == 'pending'">
            <t t-set="alert_style" t-value="'warning'"/>
            <t t-set="status_message" t-value="tx.provider_id.sudo().pending_msg"/>
        </t>
        <t t-elif="tx.state == 'authorized'">
            <t t-set="alert_style" t-value="'success'"/>
            <t t-set="status_message" t-value="tx.provider_id.sudo().auth_msg"/>
        </t>
        <t t-elif="tx.state == 'done'">
            <t t-set="alert_style" t-value="'success'"/>
            <t t-set="status_message" t-value="tx.provider_id.sudo().done_msg"/>
        </t>
        <t t-elif="tx.state == 'cancel'">
            <t t-set="alert_style" t-value="'danger'"/>
            <t t-set="status_message" t-value="tx.provider_id.sudo().cancel_msg"/>
        </t>
        <t t-elif="tx.state == 'error'">
            <t t-set="alert_style" t-value="'danger'"/>
            <t t-set="status_message">
                <p>An error occurred during the processing of your payment.</p>
            </t>
        </t>

        <t t-if="is_html_empty(status_message)" t-set="status_message" t-value="''"/>

        <div t-if="status_message or tx.state_message"
             id="o_payment_status_alert"
             t-attf-class="alert alert-{{alert_style}} alert-dismissible">
            <button class="btn-close" data-bs-dismiss="alert" title="Dismiss"/>
            <t t-if="status_message" t-out="status_message"/>
            <t t-if="tx.state_message" t-out="tx.state_message"/>
        </div>
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
                        <group name="general_information">
                            <field name="payment_details"/>
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
            <tree string="Payment Tokens">
                <field name="payment_details"/>
                <field name="partner_id"/>
                <field name="provider_id" readonly="1"/>
                <field name="provider_ref" readonly="1"/>
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
        <field name="view_mode">tree,form</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a new payment token
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
                    <button type="object" name="action_capture" states="authorized" string="Capture Transaction" class="oe_highlight"/>
                    <button type="object" name="action_void" states="authorized" string="Void Transaction"
                            confirm="Are you sure you want to void the authorized transaction? This action can't be undone."/>
                    <field name="state" widget="statusbar"/>
                </header>
                <sheet>
                    <div class="oe_button_box" name="button_box">
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
                            <field name="source_transaction_id"
                                   attrs="{'invisible': [('source_transaction_id', '=', False)]}"/>
                            <field name="amount"/>
                            <field name="fees" attrs="{'invisible': [('fees', '=', 0.0)]}"/>
                            <field name="currency_id" invisible="1"/>
                            <field name="provider_id"/>
                            <field name="company_id" groups="base.group_multi_company"/>
                            <!-- Used by some provider-specific views -->
                            <field name="provider_code" invisible="1"/>
                            <field name="provider_reference"/>
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
                        <field name="state_message" nolabel="1" colspan="2"/>
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
                <field name="provider_id"/>
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
                                    <span class="float-end">
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

## File: wizards\payment_link_wizard.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from werkzeug import urls

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError
from odoo.tools import float_compare

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
    description = fields.Char("Payment Ref")
    link = fields.Char(string="Payment Link", compute='_compute_link')
    company_id = fields.Many2one('res.company', compute='_compute_company_id')
    available_provider_ids = fields.Many2many(
        comodel_name='payment.provider',
        string="Payment Providers Available",
        compute='_compute_available_provider_ids',
        compute_sudo=True,
    )
    has_multiple_providers = fields.Boolean(
        string="Has Multiple Providers",
        compute='_compute_has_multiple_providers',
    )
    payment_provider_selection = fields.Selection(
        string="Allow Payment Provider",
        help="If a specific payment provider is selected, customers will only be allowed to pay "
             "via this one. If 'All' is selected, customers can pay via any available payment "
             "provider.",
        selection='_selection_payment_provider_selection',
        default='all',
        required=True,
    )

    @api.onchange('amount', 'description')
    def _onchange_amount(self):
        if float_compare(self.amount_max, self.amount, precision_rounding=self.currency_id.rounding or 0.01) == -1:
            raise ValidationError(_("Please set an amount smaller than %s.", self.amount_max))
        if self.amount <= 0:
            raise ValidationError(_("The value of the payment amount must be positive."))

    @api.depends('res_model', 'res_id')
    def _compute_company_id(self):
        for link in self:
            record = self.env[link.res_model].browse(link.res_id)
            link.company_id = record.company_id if 'company_id' in record else False

    @api.depends('company_id', 'partner_id', 'currency_id')
    def _compute_available_provider_ids(self):
        for link in self:
            link.available_provider_ids = link._get_payment_provider_available(
                res_model=link.res_model,
                res_id=link.res_id,
                company_id=link.company_id.id,
                partner_id=link.partner_id.id,
                amount=link.amount,
                currency_id=link.currency_id.id,
            )

    def _selection_payment_provider_selection(self):
        """ Specify available providers in the selection field.
        :return: The selection list of available providers.
        :rtype: list[tuple]
        """
        defaults = self.default_get(['res_model', 'res_id'])
        selection = [('all', "All")]
        res_model, res_id = defaults.get('res_model'), defaults.get('res_id')
        if res_id and res_model in ['account.move', "sale.order"]:
            # At module install, the selection method is called
            # but the document context isn't specified.
            related_document = self.env[res_model].browse(res_id)
            company_id = related_document.company_id
            partner_id = related_document.partner_id
            currency_id = related_document.currency_id
            selection.extend(
                self._get_payment_provider_available(
                    res_model=res_model,
                    res_id=res_id,
                    company_id=company_id.id,
                    partner_id=partner_id.id,
                    amount=related_document.amount_total,
                    currency_id=currency_id.id,
                ).name_get()
            )
        return selection

    def _get_payment_provider_available(self, **kwargs):
        """ Select and return the providers matching the criteria.

        :return: The compatible providers
        :rtype: recordset of `payment.provider`
        """
        return self.env['payment.provider'].sudo()._get_compatible_providers(**kwargs)

    @api.depends('available_provider_ids')
    def _compute_has_multiple_providers(self):
        for link in self:
            link.has_multiple_providers = len(link.available_provider_ids) > 1

    def _get_access_token(self):
        self.ensure_one()
        return payment_utils.generate_access_token(
            self.partner_id.id, self.amount, self.currency_id.id
        )

    @api.depends(
        'description', 'amount', 'currency_id', 'partner_id', 'company_id',
        'payment_provider_selection',
    )
    def _compute_link(self):
        for payment_link in self:
            related_document = self.env[payment_link.res_model].browse(payment_link.res_id)
            base_url = related_document.get_base_url()  # Don't generate links for the wrong website
            url_params = {
                'reference': payment_link.description,
                'amount': self.amount,
                'access_token': self._get_access_token(),
                **self._get_additional_link_values(),
            }
            if payment_link.payment_provider_selection != 'all':
                url_params['provider_id'] = str(payment_link.payment_provider_selection)
            payment_link.link = f'{base_url}/payment/pay?{urls.url_encode(url_params)}'

    def _get_additional_link_values(self):
        """ Return the additional values to append to the payment link.

        Note: self.ensure_one()

        :return: The additional payment link values.
        :rtype: dict
        """
        self.ensure_one()
        return {
            'currency_id': self.currency_id.id,
            'partner_id': self.partner_id.id,
            'company_id': self.company_id.id,
        }

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
                <div class="alert alert-warning fw-bold"
                     role="alert"
                     attrs="{'invisible': [('partner_email', '!=', False)]}">
                     This partner has no email, which may cause issues with some payment providers.
                     Setting an email for this partner is advised.
                </div>
                <group>
                    <group>
                        <field name="res_id" invisible="1"/>
                        <field name="res_model" invisible="1"/>
                        <field name="partner_id" invisible="1"/>
                        <field name="partner_email" invisible="1"/>
                        <field name="amount_max" invisible="1"/>
                        <field name="available_provider_ids" invisible="1"/>
                        <field name="has_multiple_providers" invisible="1"/>
                        <field name="description"/>
                        <field name="amount"/>
                        <field name="currency_id" invisible="1"/>
                        <field name="payment_provider_selection"
                            attrs="{'invisible':[('has_multiple_providers', '=', False)]}"/>
                    </group>
                </group>
                <group>
                    <field name="link" readonly="1" widget="CopyClipboardChar"/>
                </group>
                <footer>
                    <button string="Close" class="btn-primary" special="cancel" data-hotkey="z"/>
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
                            <div attrs="{'invisible': [('payment_method', '!=', 'paypal')]}">
                                <group>
                                    <field name="paypal_email_account" attrs="{'required': [('payment_method', '=', 'paypal')]}" string="Email"/>
                                    <field name="paypal_pdt_token" password="True" attrs="{'required': [('payment_method', '=', 'paypal')]}" />
                                </group>
                                <p>
                                    <a href="https://www.odoo.com/documentation/16.0/applications/finance/payment_providers/paypal.html" target="_blank">
                                        <span><i class="fa fa-arrow-right"/> How to configure your PayPal account</span>
                                    </a>
                                </p>
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

    paypal_user_type = fields.Selection([
        ('new_user', "I don't have a Paypal account"),
        ('existing_user', 'I have a Paypal account')], string="Paypal User Type", default='new_user')
    paypal_email_account = fields.Char("Email", default=lambda self: self._get_default_payment_provider_onboarding_value('paypal_email_account'))
    paypal_seller_account = fields.Char("Merchant Account ID")
    paypal_pdt_token = fields.Char("PDT Identity Token", default=lambda self: self._get_default_payment_provider_onboarding_value('paypal_pdt_token'))

    # Account-specific logic. It's kept here rather than moved in `account_payment` as it's not used by `account` module.
    manual_name = fields.Char("Method", default=lambda self: self._get_default_payment_provider_onboarding_value('manual_name'))
    journal_name = fields.Char("Bank Name", default=lambda self: self._get_default_payment_provider_onboarding_value('journal_name'))
    acc_number = fields.Char("Account Number", default=lambda self: self._get_default_payment_provider_onboarding_value('acc_number'))
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

    _payment_provider_onboarding_cache = {}

    def _get_manual_payment_provider(self, env=None):
        if env is None:
            env = self.env
        module_id = env.ref('base.module_payment_custom').id
        return env['payment.provider'].search([('module_id', '=', module_id),
            ('company_id', '=', env.company.id)], limit=1)

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
            provider = self.env['payment.provider'].search(
                [('company_id', '=', self.env.company.id), ('code', '=', 'paypal')], limit=1
            )
            self._payment_provider_onboarding_cache['paypal_email_account'] = provider['paypal_email_account'] or self.env.user.email or ''
            self._payment_provider_onboarding_cache['paypal_pdt_token'] = provider['paypal_pdt_token']

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
                provider = new_env['payment.provider'].search(
                    [('company_id', '=', self.env.company.id), ('code', '=', 'paypal')], limit=1
                )
                if not provider:
                    base_provider = self.env.ref('payment.payment_provider_paypal')
                    # Use sudo to access payment provider record that can be in different company.
                    provider = base_provider.sudo().copy(default={'company_id':self.env.company.id})
                default_journal = new_env['account.journal'].search(
                    [('type', '=', 'bank'), ('company_id', '=', new_env.company.id)], limit=1
                )
                provider.write({
                    'paypal_email_account': self.paypal_email_account,
                    'paypal_pdt_token': self.paypal_pdt_token,
                    'state': 'enabled',
                    'is_published': 'True',
                    'journal_id': provider.journal_id or default_journal
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

        # the user clicked `apply` and not cancel so we can assume this step is done.
        self._set_payment_provider_onboarding_step_done()
        return {'type': 'ir.actions.act_window_close'}

    def _set_payment_provider_onboarding_step_done(self):
        self.env.company.sudo().set_onboarding_step_done('payment_provider_onboarding_state')

    def _start_stripe_onboarding(self):
        """ Start Stripe Connect onboarding. """
        menu = self.env.ref('account_payment.payment_provider_menu', False)
        menu_id = menu and menu.id  # Only set if `account_payment` is installed.
        return self.env.company._run_payment_onboarding_step(menu_id)

```

## File: wizards\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import payment_link_wizard
from . import payment_onboarding_wizard

```

