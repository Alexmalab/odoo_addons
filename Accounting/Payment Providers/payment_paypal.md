# Odoo Module: payment_paypal

Category: Accounting/Payment Providers

This file contains the source code of the Odoo module.

## File: const.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# ISO 4217 codes of currencies supported by PayPal
# See https://developer.paypal.com/docs/reports/reference/paypal-supported-currencies/.
# Last seen on: 22 September 2022.
SUPPORTED_CURRENCIES = (
    'AUD',
    'BRL',
    'CAD',
    'CNY',
    'CZK',
    'DKK',
    'EUR',
    'HKD',
    'HUF',
    'ILS',
    'JPY',
    'MYR',
    'MXN',
    'TWD',
    'NZD',
    'NOK',
    'PHP',
    'PLN',
    'GBP',
    'RUB',
    'SGD',
    'SEK',
    'CHF',
    'THB',
    'USD',
)

# The codes of the payment methods to activate when Paypal is activated.
DEFAULT_PAYMENT_METHODS_CODES = [
    # Primary payment methods.
    'paypal',
]

# Mapping of transaction states to PayPal payment statuses
# See https://developer.paypal.com/docs/api-basics/notifications/ipn/IPNandPDTVariables/
PAYMENT_STATUS_MAPPING = {
    'pending': ('Pending',),
    'done': ('Processed', 'Completed', 'Cleared'),  # cleared status is required fo echeck
    'cancel': ('Voided', 'Expired'),
}

```

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

from odoo.addons.payment import setup_provider, reset_payment_provider


def post_init_hook(env):
    setup_provider(env, 'paypal')


def uninstall_hook(env):
    reset_payment_provider(env, 'paypal')

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Payment Provider: Paypal',
    'version': '2.0',
    'category': 'Accounting/Payment Providers',
    'sequence': 350,
    'summary': "An American payment provider for online payments all over the world.",
    'description': " ",  # Non-empty string to avoid loading the README file.
    'depends': ['payment'],
    'data': [
        'views/payment_paypal_templates.xml',
        'views/payment_provider_views.xml',
        'views/payment_transaction_views.xml',

        'data/payment_provider_data.xml',
    ],
    'post_init_hook': 'post_init_hook',
    'uninstall_hook': 'uninstall_hook',
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import pprint

import requests
from werkzeug import urls
from werkzeug.exceptions import Forbidden

from odoo import _, http
from odoo.exceptions import ValidationError
from odoo.http import request
from odoo.tools import html_escape

from odoo.addons.payment import utils as payment_utils


_logger = logging.getLogger(__name__)


class PaypalController(http.Controller):
    _return_url = '/payment/paypal/return/'
    _cancel_url = '/payment/paypal/cancel/'
    _webhook_url = '/payment/paypal/webhook/'

    @http.route(
        _return_url, type='http', auth='public', methods=['GET', 'POST'], csrf=False,
        save_session=False
    )
    def paypal_return_from_checkout(self, **pdt_data):
        """ Process the PDT notification sent by PayPal after redirection from checkout.

        The PDT (Payment Data Transfer) notification contains the parameters necessary to verify the
        origin of the notification and retrieve the actual notification data, if PDT is enabled on
        the account. See https://developer.paypal.com/api/nvp-soap/payment-data-transfer/.

        The route accepts both GET and POST requests because PayPal seems to switch between the two
        depending on whether PDT is enabled, whether the customer pays anonymously (without logging
        in on PayPal), whether they click on "Return to Merchant" after paying, etc.

        The route is flagged with `save_session=False` to prevent Odoo from assigning a new session
        to the user if they are redirected to this route with a POST request. Indeed, as the session
        cookie is created without a `SameSite` attribute, some browsers that don't implement the
        recommended default `SameSite=Lax` behavior will not include the cookie in the redirection
        request from the payment provider to Odoo. As the redirection to the '/payment/status' page
        will satisfy any specification of the `SameSite` attribute, the session of the user will be
        retrieved and with it the transaction which will be immediately post-processed.

        :param dict pdt_data: The PDT notification data send by PayPal.
        """
        _logger.info("Handling redirection from PayPal with data:\n%s", pprint.pformat(pdt_data))

        tx_sudo = request.env['payment.transaction'].sudo()._get_tx_from_notification_data(
            'paypal', pdt_data
        )
        try:
            notification_data = self._verify_pdt_notification_origin(pdt_data, tx_sudo)
        except Forbidden:
            _logger.exception("Could not verify the origin of the PDT; discarding it.")
        else:
            tx_sudo._handle_notification_data('paypal', notification_data)

        return request.redirect('/payment/status')

    @http.route(
        _cancel_url, type='http', auth='public', methods=['GET'], csrf=False, save_session=False
    )
    def paypal_return_from_canceled_checkout(self, tx_ref, return_access_tkn):
        """ Process the transaction after the customer has canceled the payment.

        :param str tx_ref: The reference of the transaction having been canceled.
        :param str return_access_tkn: The access token to verify the authenticity of the request.
                                      PayPal forbids any parameter with the name "token" inside.
        """
        _logger.info(
            "Handling redirection from Paypal for cancellation of transaction with reference %s",
            tx_ref,
        )

        tx_sudo = request.env['payment.transaction'].sudo()._get_tx_from_notification_data(
            'paypal', {'item_number': tx_ref}
        )
        if not payment_utils.check_access_token(return_access_tkn, tx_ref):
            raise Forbidden()
        tx_sudo._handle_notification_data('paypal', {})

        return request.redirect('/payment/status')

    def _verify_pdt_notification_origin(self, pdt_data, tx_sudo):
        """ Validate the authenticity of a PDT and return the retrieved notification data.

        The validation is done in four steps:

        1. Make a POST request to Paypal with `tx`, the GET param received with the PDT data, and
           with the two other required params `cmd` and `at`.
        2. PayPal sends back a response text starting with either 'SUCCESS' or 'FAIL'. If the
           validation was a success, the notification data are appended to the response text as a
           string formatted as follows: 'SUCCESS\nparam1=value1\nparam2=value2\n...'
        3. Extract the notification data and process these instead of the PDT.
        4. Return an empty HTTP 200 response (done at the end of the route controller).

        See https://developer.paypal.com/docs/api-basics/notifications/payment-data-transfer/.

        :param dict pdt_data: The PDT data whose authenticity must be checked.
        :param recordset tx_sudo: The sudoed transaction referenced in the PDT, as a
                                  `payment.transaction` record
        :return: The retrieved notification data
        :raise :class:`werkzeug.exceptions.Forbidden`: if the notification origin can't be verified
        """
        if 'tx' not in pdt_data:  # PDT is not enabled; PayPal directly sent the notification data.
            tx_sudo._log_message_on_linked_documents(_(
                "The status of transaction with reference %(ref)s was not synchronized because the "
                "'Payment data transfer' option is not enabled on the PayPal dashboard.",
                ref=tx_sudo.reference,
            ))
            raise Forbidden("PayPal: PDT are not enabled; cannot verify data origin")
        else:  # The PayPal account is configured to send PDT data.
            # Request a PDT data authenticity check and the notification data to PayPal.
            provider_sudo = tx_sudo.provider_id
            url = provider_sudo._paypal_get_api_url()
            payload = {
                'cmd': '_notify-synch',
                'tx': pdt_data['tx'],
                'at': tx_sudo.provider_id.paypal_pdt_token,
            }
            try:
                response = requests.post(url, data=payload, timeout=10)
                response.raise_for_status()
            except (requests.exceptions.ConnectionError, requests.exceptions.HTTPError):
                raise Forbidden("PayPal: Encountered an error when verifying PDT origin")
            else:
                notification_data = self._parse_pdt_validation_response(response.text)
                if notification_data is None:
                    raise Forbidden("PayPal: The PDT origin was not verified by PayPal")

        return notification_data

    @staticmethod
    def _parse_pdt_validation_response(response_content):
        """ Parse the PDT validation request response and return the parsed notification data.

        :param str response_content: The PDT validation request response
        :return: The parsed notification data
        :rtype: dict
        """
        response_items = response_content.splitlines()
        if response_items[0] == 'SUCCESS':
            notification_data = {}
            for notification_data_param in response_items[1:]:
                key, raw_value = notification_data_param.split('=', 1)
                notification_data[key] = urls.url_unquote_plus(raw_value)
            return notification_data
        return None

    @http.route(_webhook_url, type='http', auth='public', methods=['GET', 'POST'], csrf=False)
    def paypal_webhook(self, **data):
        """ Process the notification data (IPN) sent by PayPal to the webhook.

        The "Instant Payment Notification" is a classical webhook notification.
        See https://developer.paypal.com/api/nvp-soap/ipn/.

        :param dict data: The notification data
        :return: An empty string to acknowledge the notification
        :rtype: str
        """
        _logger.info("notification received from PayPal with data:\n%s", pprint.pformat(data))
        try:
            # Check the origin and integrity of the notification
            tx_sudo = request.env['payment.transaction'].sudo()._get_tx_from_notification_data(
                'paypal', data
            )
            self._verify_webhook_notification_origin(data, tx_sudo)

            # Handle the notification data
            tx_sudo._handle_notification_data('paypal', data)
        except ValidationError:  # Acknowledge the notification to avoid getting spammed
            _logger.warning(
                "unable to handle the notification data; skipping to acknowledge", exc_info=True
            )
        return ''

    @staticmethod
    def _verify_webhook_notification_origin(notification_data, tx_sudo):
        """ Check that the notification was sent by PayPal.

        The verification is done in three steps:

        1. POST the complete message back to Paypal with the additional param
           `cmd=_notify-validate`, in the same encoding.
        2. PayPal sends back either 'VERIFIED' or 'INVALID'.
        3. Return an empty HTTP 200 response if the notification origin is verified by PayPal, raise
           an HTTP 403 otherwise.

        See https://developer.paypal.com/docs/api-basics/notifications/ipn/IPNIntro/.

        :param dict notification_data: The notification data
        :param recordset tx_sudo: The sudoed transaction referenced in the notification data, as a
                                        `payment.transaction` record
        :return: None
        :raise: :class:`werkzeug.exceptions.Forbidden` if the notification origin can't be verified
        """
        # Request PayPal for an authenticity check
        url = tx_sudo.provider_id._paypal_get_api_url()
        payload = dict(notification_data, cmd='_notify-validate')
        try:
            response = requests.post(url, payload, timeout=60)
            response.raise_for_status()
        except (requests.exceptions.ConnectionError, requests.exceptions.HTTPError) as error:
            _logger.exception(
                "could not verify notification origin at %(url)s with data: %(data)s:\n%(error)s",
                {
                    'url': url,
                    'data': pprint.pformat(notification_data),
                    'error': pprint.pformat(error.response.text),
                },
            )
            raise Forbidden()
        else:
            response_content = response.text
            if response_content != 'VERIFIED':
                _logger.warning(
                    "PayPal did not confirm the origin of the notification with data:\n%s",
                    pprint.pformat(notification_data),
                )
                raise Forbidden()

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\neutralize.sql

```sql
-- disable paypal payment provider
UPDATE payment_provider
   SET paypal_email_account = NULL,
       paypal_pdt_token = NULL;

```

## File: data\payment_provider_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="payment.payment_provider_paypal" model="payment.provider">
        <field name="code">paypal</field>
        <field name="redirect_form_view_id" ref="redirect_form"/>
    </record>

</odoo>

```

## File: models\payment_provider.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from odoo import _, fields, models

from odoo.addons.payment_paypal import const


_logger = logging.getLogger(__name__)


class PaymentProvider(models.Model):
    _inherit = 'payment.provider'

    code = fields.Selection(
        selection_add=[('paypal', "Paypal")], ondelete={'paypal': 'set default'}
    )
    paypal_email_account = fields.Char(
        string="Email",
        help="The public business email solely used to identify the account with PayPal",
        required_if_provider='paypal',
        default=lambda self: self.env.company.email,
    )
    paypal_pdt_token = fields.Char(string="PDT Identity Token", groups='base.group_system')

    #=== BUSINESS METHODS ===#

    def _get_supported_currencies(self):
        """ Override of `payment` to return the supported currencies. """
        supported_currencies = super()._get_supported_currencies()
        if self.code == 'paypal':
            supported_currencies = supported_currencies.filtered(
                lambda c: c.name in const.SUPPORTED_CURRENCIES
            )
        return supported_currencies

    def _paypal_get_api_url(self):
        """ Return the API URL according to the provider state.

        Note: self.ensure_one()

        :return: The API URL
        :rtype: str
        """
        self.ensure_one()

        if self.state == 'enabled':
            return 'https://www.paypal.com/cgi-bin/webscr'
        else:
            return 'https://www.sandbox.paypal.com/cgi-bin/webscr'

    def _get_default_payment_method_codes(self):
        """ Override of `payment` to return the default payment method codes. """
        default_codes = super()._get_default_payment_method_codes()
        if self.code != 'paypal':
            return default_codes
        return const.DEFAULT_PAYMENT_METHODS_CODES

```

## File: models\payment_transaction.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from werkzeug import urls

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError

from odoo.addons.payment import utils as payment_utils
from odoo.addons.payment_paypal.const import PAYMENT_STATUS_MAPPING
from odoo.addons.payment_paypal.controllers.main import PaypalController

_logger = logging.getLogger(__name__)


class PaymentTransaction(models.Model):
    _inherit = 'payment.transaction'

    # See https://developer.paypal.com/docs/api-basics/notifications/ipn/IPNandPDTVariables/
    # this field has no use in Odoo except for debugging
    paypal_type = fields.Char(string="PayPal Transaction Type")

    def _get_specific_rendering_values(self, processing_values):
        """ Override of payment to return Paypal-specific rendering values.

        Note: self.ensure_one() from `_get_processing_values`

        :param dict processing_values: The generic and specific processing values of the transaction
        :return: The dict of provider-specific processing values
        :rtype: dict
        """
        res = super()._get_specific_rendering_values(processing_values)
        if self.provider_code != 'paypal':
            return res

        base_url = self.provider_id.get_base_url()
        cancel_url = urls.url_join(base_url, PaypalController._cancel_url)
        cancel_url_params = {
            'tx_ref': self.reference,
            'return_access_tkn': payment_utils.generate_access_token(self.reference),
        }
        partner_first_name, partner_last_name = payment_utils.split_partner_name(self.partner_name)
        if (
            self.partner_address
            and self.partner_city
            and self.partner_country_id
            and (self.partner_zip or not self.partner_country_id.zip_required)
            and (self.partner_state_id or not self.partner_country_id.state_required)
        ):
            # Ensure given address cannot be altered.
            no_shipping = '0'
            address_override = '1'
        else:
            # Do not prompt for a delivery address.
            no_shipping = '1'
            address_override = '0'
        return {
            'address1': self.partner_address,
            'amount': self.amount,
            'business': self.provider_id.paypal_email_account,
            'cancel_url': f'{cancel_url}?{urls.url_encode(cancel_url_params)}',
            'city': self.partner_city,
            'country': self.partner_country_id.code,
            'currency_code': self.currency_id.name,
            'email': self.partner_email,
            'first_name': partner_first_name,
            'item_name': f"{self.company_id.name}: {self.reference}",
            'item_number': self.reference,
            'last_name': partner_last_name,
            'lc': self.partner_lang,
            'no_shipping': no_shipping,
            'address_override': address_override,
            'notify_url': urls.url_join(base_url, PaypalController._webhook_url),
            'return_url': urls.url_join(base_url, PaypalController._return_url),
            'state': self.partner_state_id.name,
            'zip_code': self.partner_zip,
            'api_url': self.provider_id._paypal_get_api_url(),
        }

    def _get_tx_from_notification_data(self, provider_code, notification_data):
        """ Override of payment to find the transaction based on Paypal data.

        :param str provider_code: The code of the provider that handled the transaction
        :param dict notification_data: The notification data sent by the provider
        :return: The transaction if found
        :rtype: recordset of `payment.transaction`
        :raise: ValidationError if the data match no transaction
        """
        tx = super()._get_tx_from_notification_data(provider_code, notification_data)
        if provider_code != 'paypal' or len(tx) == 1:
            return tx

        reference = notification_data.get('item_number')
        tx = self.search([('reference', '=', reference), ('provider_code', '=', 'paypal')])
        if not tx:
            raise ValidationError(
                "PayPal: " + _("No transaction found matching reference %s.", reference)
            )
        return tx

    def _process_notification_data(self, notification_data):
        """ Override of payment to process the transaction based on Paypal data.

        Note: self.ensure_one()

        :param dict notification_data: The notification data sent by the provider
        :return: None
        :raise: ValidationError if inconsistent data were received
        """
        super()._process_notification_data(notification_data)
        if self.provider_code != 'paypal':
            return

        if not notification_data:
            self._set_canceled(state_message=_("The customer left the payment page."))
            return

        amount = notification_data.get('amt') or notification_data.get('mc_gross')
        currency_code = notification_data.get('cc') or notification_data.get('mc_currency')
        assert amount and currency_code, 'PayPal: missing amount or currency'
        assert self.currency_id.compare_amounts(float(amount), self.amount) == 0, \
            'PayPal: mismatching amounts'
        assert currency_code == self.currency_id.name, 'PayPal: mismatching currency codes'

        # Update the provider reference.
        txn_id = notification_data.get('txn_id')
        txn_type = notification_data.get('txn_type')
        if not all((txn_id, txn_type)):
            raise ValidationError(
                "PayPal: " + _(
                    "Missing value for txn_id (%(txn_id)s) or txn_type (%(txn_type)s).",
                    txn_id=txn_id, txn_type=txn_type
                )
            )
        self.provider_reference = txn_id
        self.paypal_type = txn_type

        # Force PayPal as the payment method if it exists.
        self.payment_method_id = self.env['payment.method'].search(
            [('code', '=', 'paypal')], limit=1
        ) or self.payment_method_id

        # Update the payment state.
        payment_status = notification_data.get('payment_status')

        if payment_status in PAYMENT_STATUS_MAPPING['pending']:
            self._set_pending(state_message=notification_data.get('pending_reason'))
        elif payment_status in PAYMENT_STATUS_MAPPING['done']:
            self._set_done()
        elif payment_status in PAYMENT_STATUS_MAPPING['cancel']:
            self._set_canceled()
        else:
            _logger.info(
                "received data with invalid payment status (%s) for transaction with reference %s",
                payment_status, self.reference
            )
            self._set_error(
                "PayPal: " + _("Received data with invalid payment status: %s", payment_status)
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
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="m17.287 44.574.74-4.623-1.649-.038H8.5l5.475-34.116a.449.449 0 0 1 .445-.373h13.282c4.41 0 7.453.901 9.042 2.682.745.835 1.219 1.707 1.448 2.668.241 1.007.245 2.211.01 3.68l-.017.107v.94l.745.415c.627.327 1.126.702 1.508 1.13.637.714 1.05 1.622 1.224 2.698.18 1.106.12 2.423-.174 3.913-.34 1.715-.89 3.208-1.632 4.43a9.172 9.172 0 0 1-2.584 2.784c-.986.687-2.157 1.21-3.48 1.543-1.284.329-2.747.494-4.35.494h-1.035a3.17 3.17 0 0 0-2.02.73 3.063 3.063 0 0 0-1.054 1.85l-.078.415-1.308 8.15-.06.298c-.015.095-.042.142-.082.174a.221.221 0 0 1-.136.05h-6.382Z" fill="#253B80"/><path d="M39.638 14.671a23.1 23.1 0 0 1-.136.766c-1.752 8.839-7.744 11.892-15.398 11.892h-3.897c-.936 0-1.725.668-1.87 1.576L16.34 41.342l-.565 3.525A.986.986 0 0 0 16.76 46h6.912c.818 0 1.514-.585 1.642-1.378l.069-.345 1.3-8.117.084-.445a1.654 1.654 0 0 1 1.643-1.38h1.034c6.696 0 11.939-2.673 13.47-10.406.64-3.23.31-5.927-1.384-7.824-.513-.572-1.149-1.047-1.892-1.434Z" fill="#179BD7"/><path d="M37.804 13.953a13.964 13.964 0 0 0-1.704-.372 22.002 22.002 0 0 0-3.435-.246h-10.41c-.257 0-.5.057-.719.16-.48.227-.837.673-.923 1.22l-2.215 13.787-.064.402a1.883 1.883 0 0 1 1.871-1.575h3.897c7.654 0 13.647-3.055 15.398-11.893.053-.261.097-.516.136-.765a9.436 9.436 0 0 0-1.44-.597 12.1 12.1 0 0 0-.392-.121Z" fill="#222D65"/><path d="M20.614 14.715a1.63 1.63 0 0 1 .923-1.219c.22-.103.462-.16.718-.16h10.411c1.233 0 2.385.08 3.435.246a14.023 14.023 0 0 1 2.097.491c.517.169.998.368 1.44.598.522-3.267-.003-5.49-1.8-7.505C35.855 4.95 32.28 4 27.706 4H14.423c-.935 0-1.732.668-1.876 1.577L7.014 40.044a1.128 1.128 0 0 0 1.126 1.297h8.2l2.06-12.839 2.214-13.787Z" fill="#253B80"/></svg>

```

## File: views\payment_paypal_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- https://developer.paypal.com/docs/paypal-payments-standard/integration-guide/formbasics -->
    <template id="redirect_form">
        <form t-att-action="api_url" method="post">
            <input type="hidden" name="address1" t-att-value="address1"/>
            <input type="hidden" name="amount" t-att-value="amount"/>
            <input type="hidden" name="business" t-att-value="business"/>
            <input type="hidden" name="cancel_return" t-att-value="cancel_url"/>
            <input type="hidden" name="city" t-att-value="city"/>
            <input type="hidden" name="cmd" value="_xclick"/>
            <input type="hidden" name="country" t-att-value="country"/>
            <input type="hidden" name="currency_code" t-att-value="currency_code"/>
            <input type="hidden" name="email" t-att-value="email"/>
            <input type="hidden" name="first_name" t-att-value="first_name"/>
            <input type="hidden" name="item_name" t-att-value="item_name"/>
            <input type="hidden" name="item_number" t-att-value="item_number"/>
            <input type="hidden" name="last_name" t-att-value="last_name"/>
            <input type="hidden" name="lc" t-att-value="lc"/>
            <input type="hidden" name="no_shipping" t-att-value="no_shipping"/>
            <input type="hidden" name="address_override" t-att-value="address_override"/>
            <input type="hidden" name="notify_url" t-att-value="notify_url"/>
            <input type="hidden" name="return" t-att-value="return_url"/>
            <input type="hidden" name="rm" value="2"/>
            <input t-if="state"
                   type="hidden" name="state" t-att-value="state"/>
            <input type="hidden" name="zip" t-att-value="zip_code"/>
        </form>
    </template>

</odoo>

```

## File: views\payment_provider_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_provider_form" model="ir.ui.view">
        <field name="name">PayPal Provider Form</field>
        <field name="model">payment.provider</field>
        <field name="inherit_id" ref="payment.payment_provider_form"/>
        <field name="arch" type="xml">
            <group name="provider_credentials" position='inside'>
                <group invisible="code != 'paypal'">
                    <field name="paypal_email_account"
                           required="code == 'paypal' and state != 'disabled'"/>
                    <field name="paypal_pdt_token"  password="True"
                           required="code == 'paypal' and state != 'disabled'"/>
                    <a href="https://www.odoo.com/documentation/17.0/applications/finance/payment_providers/paypal.html"
                       target="_blank"
                       colspan="2">
                        How to configure your paypal account?
                    </a>
                </group>
            </group>
        </field>
    </record>

</odoo>

```

## File: views\payment_transaction_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_transaction_form" model="ir.ui.view">
        <field name="name">PayPal Transaction Form</field>
        <field name="model">payment.transaction</field>
        <field name="inherit_id" ref="payment.payment_transaction_form"/>
        <field name="arch" type="xml">
            <field name="provider_reference" position="after">
                <field name="paypal_type"
                       readonly="1"
                       invisible="provider_code != 'paypal'"
                       groups="base.group_no_one"/>
            </field>
        </field>
    </record>

</odoo>

```

