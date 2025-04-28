# Odoo Module: payment_paypal

Category: Accounting/Payment Providers

This file contains the source code of the Odoo module.

## File: const.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# ISO 4217 codes of currencies supported by PayPal
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

# Mapping of transaction states to PayPal payment statuses
# See https://developer.paypal.com/docs/api-basics/notifications/ipn/IPNandPDTVariables/
PAYMENT_STATUS_MAPPING = {
    'pending': ('Pending',),
    'done': ('Processed', 'Completed'),
    'cancel': ('Voided', 'Expired'),
}

```

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

from odoo.addons.payment import setup_provider, reset_payment_provider


def post_init_hook(cr, registry):
    setup_provider(cr, registry, 'paypal')


def uninstall_hook(cr, registry):
    reset_payment_provider(cr, registry, 'paypal')

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
        'data/payment_paypal_email_data.xml',
    ],
    'application': False,
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


_logger = logging.getLogger(__name__)


class PaypalController(http.Controller):
    _return_url = '/payment/paypal/return/'
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
        in on PayPal), whether the customer cancels the payment, whether they click on "Return to
        Merchant" after paying, etc.

        The route is flagged with `save_session=False` to prevent Odoo from assigning a new session
        to the user if they are redirected to this route with a POST request. Indeed, as the session
        cookie is created without a `SameSite` attribute, some browsers that don't implement the
        recommended default `SameSite=Lax` behavior will not include the cookie in the redirection
        request from the payment provider to Odoo. As the redirection to the '/payment/status' page
        will satisfy any specification of the `SameSite` attribute, the session of the user will be
        retrieved and with it the transaction which will be immediately post-processed.
        """
        _logger.info("handling redirection from PayPal with data:\n%s", pprint.pformat(pdt_data))
        if not pdt_data:  # The customer has canceled or paid then clicked on "Return to Merchant"
            pass  # Redirect them to the status page to browse the (currently) draft transaction
        else:
            # Check the origin of the notification
            tx_sudo = request.env['payment.transaction'].sudo()._get_tx_from_notification_data(
                'paypal', pdt_data
            )
            try:
                notification_data = self._verify_pdt_notification_origin(pdt_data, tx_sudo)
            except Forbidden:
                _logger.exception("could not verify the origin of the PDT; discarding it")
            else:
                # Handle the notification data
                tx_sudo._handle_notification_data('paypal', notification_data)

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

        :param dict pdt_data: The PDT whose authenticity must be checked.
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
        else:
            provider_sudo = tx_sudo.provider_id
            if not provider_sudo.paypal_pdt_token:  # We received PDT data but can't verify them
                record_link = f'<a href=# data-oe-model=payment.provider ' \
                              f'data-oe-id={provider_sudo.id}>{html_escape(provider_sudo.name)}</a>'
                tx_sudo._log_message_on_linked_documents(_(
                    "The status of transaction with reference %(ref)s was not synchronized because "
                    "the PDT Identify Token is not configured on the provider %(record_link)s.",
                    ref=tx_sudo.reference, record_link=record_link
                ))
                raise Forbidden("PayPal: The PDT token is not set; cannot verify data origin")
            else:  # The PayPal account is configured to receive PDT data, and the PDT token is set
                # Request a PDT data authenticity check and the notification data to PayPal
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
            _logger.exception("unable to handle the notification data; skipping to acknowledge")
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
       paypal_seller_account = NULL,
       paypal_pdt_token = NULL;

```

## File: data\payment_paypal_email_data.xml

```xml
<?xml version="1.0" ?>
<odoo>
    <!-- No longer used, deleted in following version-->
    <template id="mail_template_paypal_invite_user_to_configure">
        <div>
            <p>
                Hello,
                <br/><br/>
                You have received a payment through PayPal.<br/>
                Kindly follow the instructions given by PayPal to create your account.<br/>
                Then, help us complete your Paypal credentials in Odoo.<br/><br/>
            </p>
            <a t-attf-href="/web#id=#{provider.id}&amp;model=payment.provider&amp;view_type=form"
               style="background-color: #875A7B; padding: 10px; text-decoration: none; color: #fff; border-radius: 5px; font-size: 12px;">
                Set Paypal credentials
            </a>
            <p>
                <br/><br/>
                Thanks,<br/>
                <b>The Odoo Team</b>
            </p>
        </div>
    </template>

</odoo>

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

from odoo import _, api, fields, models

from odoo.addons.payment_paypal.const import SUPPORTED_CURRENCIES

_logger = logging.getLogger(__name__)


class PaymentProvider(models.Model):
    _inherit = 'payment.provider'

    code = fields.Selection(
        selection_add=[('paypal', "Paypal")], ondelete={'paypal': 'set default'})
    paypal_email_account = fields.Char(
        string="Email",
        help="The public business email solely used to identify the account with PayPal",
        required_if_provider='paypal')
    paypal_seller_account = fields.Char(
        string="Merchant Account ID", groups='base.group_system')
    paypal_pdt_token = fields.Char(string="PDT Identity Token", groups='base.group_system')
    paypal_use_ipn = fields.Boolean(
        string="Use IPN", help="Paypal Instant Payment Notification", default=True)

    #=== COMPUTE METHODS ===#

    def _compute_feature_support_fields(self):
        """ Override of `payment` to enable additional features. """
        super()._compute_feature_support_fields()
        self.filtered(lambda p: p.code == 'paypal').update({
            'support_fees': True,
        })

    #=== BUSINESS METHODS ===#

    @api.model
    def _get_compatible_providers(self, *args, currency_id=None, **kwargs):
        """ Override of payment to unlist PayPal providers when the currency is not supported. """
        providers = super()._get_compatible_providers(*args, currency_id=currency_id, **kwargs)

        currency = self.env['res.currency'].browse(currency_id).exists()
        if currency and currency.name not in SUPPORTED_CURRENCIES:
            providers = providers.filtered(lambda p: p.code != 'paypal')

        return providers

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
        partner_first_name, partner_last_name = payment_utils.split_partner_name(self.partner_name)
        webhook_url = urls.url_join(base_url, PaypalController._webhook_url)
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
            'city': self.partner_city,
            'country': self.partner_country_id.code,
            'currency_code': self.currency_id.name,
            'email': self.partner_email,
            'first_name': partner_first_name,
            'handling': self.fees,
            'item_name': f"{self.company_id.name}: {self.reference}",
            'item_number': self.reference,
            'last_name': partner_last_name,
            'lc': self.partner_lang,
            'no_shipping': no_shipping,
            'address_override': address_override,
            'notify_url': webhook_url if self.provider_id.paypal_use_ipn else None,
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

        amount = notification_data.get('amt') or notification_data.get('mc_gross')
        currency_code = notification_data.get('cc') or notification_data.get('mc_currency')
        assert amount and currency_code, 'PayPal: missing amount or currency'
        assert self.currency_id.compare_amounts(float(amount), self.amount + self.fees) == 0, \
            'PayPal: mismatching amounts'
        assert currency_code == self.currency_id.name, 'PayPal: mismatching currency codes'

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
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#94B6C8"/><stop offset="100%" stop-color="#6A9EBA"/></linearGradient><path id="d" d="M35.363 37.5c1.129-2.152 1.841-4.318 2.137-6.5h11.536v-3.212a.342.342 0 0 0-.339-.343H37.5c-.109-.965-.34-1.879-.697-2.742h12.233c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742 2.258.007 3.907.022 5.137 0h24.31a.342.342 0 0 0 .339-.343V37.5H35.363zM15.46 55.642h-1.036V55h2.833v.642h-1.037v2.832h-.76v-2.832zM17.423 55h1.07l.809 2.389h.01L20.075 55h1.07v3.474h-.711v-2.462h-.01l-.847 2.462h-.586l-.848-2.438h-.01v2.438h-.711V55zm4.765-36h4.547c2.803 0 6.062 2.167 5.09 6.933-.858 4.21-4.06 6.686-7.892 6.686h-2.496c-.549 0-.913.478-.999 1.047L19.334 41h-4.326c-.548 0-1.085-.479-.999-1.048l2.997-19.904c.086-.57.45-1.048 1-1.048h4.182zm11.656 10.987c-.76 3.915-3.604 6.218-7.006 6.218l-.81-.003-.488.003c-.488.003-.727.442-.803.97l-.053.363-.774 5.388c-.081.56-.416 1.036-.92 1.072L19.896 44c-.487 0-.963-.445-.887-.974l.015-.103h1.985l1.043-7.26.029.004.176-1.36h1.589c4.727 0 8.518-3.272 9.606-8.307.512 1.003.717 2.325.393 3.987z"/><path id="e" d="M35.363 35.5c1.129-2.152 1.841-4.318 2.137-6.5h11.536v-3.212a.342.342 0 0 0-.339-.343H37.5c-.109-.965-.34-1.879-.697-2.742h12.233c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742 2.258.007 3.907.022 5.137 0h24.31a.342.342 0 0 0 .339-.343V35.5H35.363zM15.46 53.642h-1.036V53h2.833v.642h-1.037v2.832h-.76v-2.832zM17.423 53h1.07l.809 2.389h.01L20.075 53h1.07v3.474h-.711v-2.462h-.01l-.847 2.462h-.586l-.848-2.438h-.01v2.438h-.711V53zm4.765-36h4.547c2.803 0 6.062 2.167 5.09 6.933-.858 4.21-4.06 6.686-7.892 6.686h-2.496c-.549 0-.913.478-.999 1.047L19.334 39h-4.326c-.548 0-1.085-.479-.999-1.048l2.997-19.904c.086-.57.45-1.048 1-1.048h4.182zm11.656 10.987c-.76 3.915-3.604 6.218-7.006 6.218l-.81-.003-.488.003c-.488.003-.727.442-.803.97l-.053.363-.774 5.388c-.081.56-.416 1.036-.92 1.072L19.896 42c-.487 0-.963-.445-.887-.974l.015-.103h1.985l1.043-7.26.029.004.176-1.36h1.589c4.727 0 8.518-3.272 9.606-8.307.512 1.003.717 2.325.393 3.987z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M4 69c-2 0-4-1-4-4V33.916L17.423 17.25H28l3.391 5.453h5.859L48 23l3.576 1.968-.236 22.01L39.224 69H4z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
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
            <input type="hidden" name="cancel_return" t-att-value="return_url"/>
            <input type="hidden" name="city" t-att-value="city"/>
            <input type="hidden" name="cmd" value="_xclick"/>
            <input type="hidden" name="country" t-att-value="country"/>
            <input type="hidden" name="currency_code" t-att-value="currency_code"/>
            <input type="hidden" name="email" t-att-value="email"/>
            <input type="hidden" name="first_name" t-att-value="first_name"/>
            <input t-if="handling"
                   type="hidden" name="handling" t-att-value="handling"/>
            <input type="hidden" name="item_name" t-att-value="item_name"/>
            <input type="hidden" name="item_number" t-att-value="item_number"/>
            <input type="hidden" name="last_name" t-att-value="last_name"/>
            <input type="hidden" name="lc" t-att-value="lc"/>
            <input type="hidden" name="no_shipping" t-att-value="no_shipping"/>
            <input type="hidden" name="address_override" t-att-value="address_override"/>
            <input t-if="notify_url"
                   type="hidden" name="notify_url" t-att-value="notify_url"/>
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
                <group attrs="{'invisible': [('code', '!=', 'paypal')]}">
                    <field name="paypal_email_account"
                           attrs="{'required':[('code', '=', 'paypal'), ('state', '!=', 'disabled')]}"/>
                    <!-- This field should no longer be used but is kept in debug mode for the time
                         being, until we are sure that the verification protocol of IPN can be used
                         for DPT notifications -->
                    <field name="paypal_pdt_token" password="True"/>
                    <field name="paypal_use_ipn"
                           attrs="{'required':[('code', '=', 'paypal'), ('state', '!=', 'disabled')]}"/>
                    <a href="https://www.odoo.com/documentation/16.0/applications/finance/payment_providers/paypal.html"
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
                       attrs="{'invisible': [('provider_code', '!=', 'paypal')]}"
                       groups="base.group_no_one"/>
            </field>
        </field>
    </record>

</odoo>

```

