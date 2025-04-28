# Odoo Module: payment_paypal

Category: Accounting/Payment Acquirers

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

from odoo.addons.payment import reset_payment_acquirer


def uninstall_hook(cr, registry):
    reset_payment_acquirer(cr, registry, 'paypal')

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Paypal Payment Acquirer',
    'version': '2.0',
    'category': 'Accounting/Payment Acquirers',
    'sequence': 365,
    'summary': 'Payment Acquirer: Paypal Implementation',
    'description': " ",  # Non-empty string to avoid loading the README file.
    'depends': ['payment'],
    'data': [
        'views/payment_views.xml',
        'views/payment_paypal_templates.xml',
        'data/payment_acquirer_data.xml',
        'data/payment_paypal_email_data.xml',
    ],
    'application': True,
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
from requests.exceptions import ConnectionError, HTTPError
from werkzeug import urls

from odoo import _, http
from odoo.exceptions import ValidationError
from odoo.http import request
from odoo.tools import html_escape


_logger = logging.getLogger(__name__)


class PaypalController(http.Controller):
    _return_url = '/payment/paypal/dpn/'
    _notify_url = '/payment/paypal/ipn/'

    @http.route(
        _return_url, type='http', auth='public', methods=['GET', 'POST'], csrf=False,
        save_session=False
    )
    def paypal_dpn(self, **data):
        """ Route used by the PDT notification.

        The "PDT notification" is actually POST data sent along the user redirection.
        The route also allows the GET method in case the user clicks on "go back to merchant site".

        The route is flagged with `save_session=False` to prevent Odoo from assigning a new session
        to the user if they are redirected to this route with a POST request. Indeed, as the session
        cookie is created without a `SameSite` attribute, some browsers that don't implement the
        recommended default `SameSite=Lax` behavior will not include the cookie in the redirection
        request from the payment provider to Odoo. As the redirection to the '/payment/status' page
        will satisfy any specification of the `SameSite` attribute, the session of the user will be
        retrieved and with it the transaction which will be immediately post-processed.
        """
        _logger.info("beginning DPN with post data:\n%s", pprint.pformat(data))
        if not data:  # The customer has cancelled the payment
            pass  # Redirect them to the status page to browse the draft transaction
        else:
            try:
                notification_data = self._validate_pdt_data_authenticity(**data)
            except ValidationError:
                _logger.exception("could not verify the origin of the PDT; discarding it")
            else:
                request.env['payment.transaction'].sudo()._handle_feedback_data(
                    'paypal', notification_data
                )

        return request.redirect('/payment/status')

    def _validate_pdt_data_authenticity(self, **data):
        """ Validate the authenticity of PDT data and return the retrieved notification data.

        The validation is done in four steps:

        1. Make a POST request to Paypal with `tx`, the GET param received with the PDT data, and
           with the two other required params `cmd` and `at`.
        2. PayPal sends back a response text starting with either 'SUCCESS' or 'FAIL'. If the
           validation was a success, the notification data are appended to the response text as a
           string formatted as follows: 'SUCCESS\nparam1=value1\nparam2=value2\n...'
        3. Extract the notification data and process these instead of the PDT data.
        4. Return an empty HTTP 200 response (done at the end of the route controller).

        See https://developer.paypal.com/docs/api-basics/notifications/payment-data-transfer/.

        :param dict data: The data whose authenticity must be checked.
        :return: The retrieved notification data
        :raise ValidationError: if the authenticity could not be verified
        """
        tx_sudo = request.env['payment.transaction'].sudo()._get_tx_from_feedback_data(
            'paypal', data
        )
        if 'tx' not in data:  # PDT is not enabled and PayPal directly sent the notification data.
            tx_sudo._log_message_on_linked_documents(_(
                "The status of transaction with reference %(ref)s was not synchronized because the "
                "'Payment data transfer' option is not enabled on the PayPal dashboard.",
                ref=tx_sudo.reference,
            ))
            raise ValidationError("PayPal: PDT are not enabled; cannot verify data origin")
        else:
            acquirer_sudo = tx_sudo.acquirer_id
            if not acquirer_sudo.paypal_pdt_token:  # We received PDT data but can't verify them
                record_link = f'<a href=# data-oe-model=payment.acquirer ' \
                              f'data-oe-id={acquirer_sudo.id}>{html_escape(acquirer_sudo.name)}</a>'
                tx_sudo._log_message_on_linked_documents(_(
                    "The status of transaction with reference %(ref)s was not synchronized because "
                    "the PDT Identify Token is not configured on the acquirer %(acq_link)s.",
                    ref=tx_sudo.reference, acq_link=record_link
                ))
                raise ValidationError("PayPal: The PDT token is not set; cannot verify data origin")
            else:  # The PayPal account is configured to receive PDT data, and the PDT token is set
                # Request a PDT data authenticity check and the notification data to PayPal
                url = acquirer_sudo._paypal_get_api_url()
                payload = {
                    'cmd': '_notify-synch',
                    'tx': data['tx'],
                    'at': acquirer_sudo.paypal_pdt_token,
                }
                try:
                    response = requests.post(url, data=payload, timeout=10)
                    response.raise_for_status()
                except (ConnectionError, HTTPError):
                    raise ValidationError("PayPal: Encountered an error when verifying PDT origin")
                else:
                    notification_data = self._parse_pdt_validation_response(response.text)
                    if notification_data is None:
                        raise ValidationError("PayPal: The PDT origin was not verified by PayPal")

        return notification_data

    @staticmethod
    def _parse_pdt_validation_response(response_content):
        """ Parse the validation response and return the parsed notification data.

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

    @http.route(_notify_url, type='http', auth='public', methods=['GET', 'POST'], csrf=False)
    def paypal_ipn(self, **data):
        """ Route used by the IPN. """
        _logger.info("beginning IPN with post data:\n%s", pprint.pformat(data))
        try:
            self._validate_ipn_data_authenticity(**data)
            request.env['payment.transaction'].sudo()._handle_feedback_data('paypal', data)
        except ValidationError:  # Acknowledge the notification to avoid getting spammed
            _logger.exception("unable to handle the IPN data; skipping to acknowledge the notif")
        return ''

    def _validate_ipn_data_authenticity(self, **data):
        """ Validate the authenticity of IPN data.

        The verification is done in three steps:

        1. POST the complete, unaltered, message back to Paypal (preceded by
           `cmd=_notify-validate`), in the same encoding.
        2. PayPal sends back either 'VERIFIED' or 'INVALID'.
        3. Return an empty HTTP 200 response (done at the end of the route method).

        See https://developer.paypal.com/docs/api-basics/notifications/ipn/IPNIntro/.

        :param dict data: The data whose authenticity must be checked.
        :return: None
        :raise ValidationError: if the authenticity could not be verified
        """
        tx_sudo = request.env['payment.transaction'].sudo()._get_tx_from_feedback_data(
            'paypal', data
        )
        acquirer_sudo = tx_sudo.acquirer_id

        # Request PayPal for an authenticity check
        data['cmd'] = '_notify-validate'
        response = requests.post(acquirer_sudo._paypal_get_api_url(), data, timeout=60)
        response.raise_for_status()

        # Inspect the response code and raise if not 'VERIFIED'.
        response_code = response.text
        if response_code == 'VERIFIED':
            _logger.info("authenticity of notification data verified")
        else:
            if response_code == 'INVALID':
                error_message = "PayPal: " + _("Notification data were not acknowledged.")
            else:
                error_message = "PayPal: " + _(
                    "Received unrecognized authentication check response code: received %s, "
                    "expected VERIFIED or INVALID.",
                    response_code
                )
            tx_sudo._set_error(error_message)
            raise ValidationError(error_message)

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\payment_acquirer_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="payment.payment_acquirer_paypal" model="payment.acquirer">
        <field name="provider">paypal</field>
        <field name="redirect_form_view_id" ref="redirect_form"/>
        <field name="support_authorization">False</field>
        <field name="support_fees_computation">True</field>
        <field name="support_refund"></field>
        <field name="support_tokenization">False</field>
    </record>

    <record id="payment_method_paypal" model="account.payment.method">
        <field name="name">Paypal</field>
        <field name="code">paypal</field>
        <field name="payment_type">inbound</field>
    </record>

</odoo>

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
            <a t-attf-href="/web#id=#{acquirer.id}&amp;model=payment.acquirer&amp;view_type=form"
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

## File: models\account_payment_method.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class AccountPaymentMethod(models.Model):
    _inherit = 'account.payment.method'

    @api.model
    def _get_payment_method_information(self):
        res = super()._get_payment_method_information()
        res['paypal'] = {'mode': 'electronic', 'domain': [('type', '=', 'bank')]}
        return res

```

## File: models\payment_acquirer.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from odoo import _, api, fields, models

from odoo.addons.payment_paypal.const import SUPPORTED_CURRENCIES

_logger = logging.getLogger(__name__)


class PaymentAcquirer(models.Model):
    _inherit = 'payment.acquirer'

    provider = fields.Selection(
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

    @api.model
    def _get_compatible_acquirers(self, *args, currency_id=None, **kwargs):
        """ Override of payment to unlist PayPal acquirers when the currency is not supported. """
        acquirers = super()._get_compatible_acquirers(*args, currency_id=currency_id, **kwargs)

        currency = self.env['res.currency'].browse(currency_id).exists()
        if currency and currency.name not in SUPPORTED_CURRENCIES:
            acquirers = acquirers.filtered(lambda a: a.provider != 'paypal')

        return acquirers

    def _paypal_get_api_url(self):
        """ Return the API URL according to the acquirer state.

        Note: self.ensure_one()

        :return: The API URL
        :rtype: str
        """
        self.ensure_one()

        if self.state == 'enabled':
            return 'https://www.paypal.com/cgi-bin/webscr'
        else:
            return 'https://www.sandbox.paypal.com/cgi-bin/webscr'

    def _get_default_payment_method_id(self):
        self.ensure_one()
        if self.provider != 'paypal':
            return super()._get_default_payment_method_id()
        return self.env.ref('payment_paypal.payment_method_paypal').id

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
    paypal_type = fields.Char(
        string="PayPal Transaction Type", help="This has no use in Odoo except for debugging.")

    def _get_specific_rendering_values(self, processing_values):
        """ Override of payment to return Paypal-specific rendering values.

        Note: self.ensure_one() from `_get_processing_values`

        :param dict processing_values: The generic and specific processing values of the transaction
        :return: The dict of acquirer-specific processing values
        :rtype: dict
        """
        res = super()._get_specific_rendering_values(processing_values)
        if self.provider != 'paypal':
            return res

        base_url = self.acquirer_id.get_base_url()
        partner_first_name, partner_last_name = payment_utils.split_partner_name(self.partner_name)
        notify_url = self.acquirer_id.paypal_use_ipn \
                     and urls.url_join(base_url, PaypalController._notify_url)
        return {
            'address1': self.partner_address,
            'amount': self.amount,
            'business': self.acquirer_id.paypal_email_account,
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
            'no_shipping': '1',  # Do not prompt for a delivery address.
            'notify_url': notify_url,
            'return_url': urls.url_join(base_url, PaypalController._return_url),
            'state': self.partner_state_id.name,
            'zip_code': self.partner_zip,
            'api_url': self.acquirer_id._paypal_get_api_url(),
        }

    @api.model
    def _get_tx_from_feedback_data(self, provider, data):
        """ Override of payment to find the transaction based on Paypal data.

        :param str provider: The provider of the acquirer that handled the transaction
        :param dict data: The feedback data sent by the provider
        :return: The transaction if found
        :rtype: recordset of `payment.transaction`
        :raise: ValidationError if the data match no transaction
        """
        tx = super()._get_tx_from_feedback_data(provider, data)
        if provider != 'paypal':
            return tx

        reference = data.get('item_number')
        tx = self.search([('reference', '=', reference), ('provider', '=', 'paypal')])
        if not tx:
            raise ValidationError(
                "PayPal: " + _("No transaction found matching reference %s.", reference)
            )
        return tx

    def _process_feedback_data(self, data):
        """ Override of payment to process the transaction based on Paypal data.

        Note: self.ensure_one()

        :param dict data: The feedback data sent by the provider
        :return: None
        :raise: ValidationError if inconsistent data were received
        """
        super()._process_feedback_data(data)
        if self.provider != 'paypal':
            return

        amount = data.get('amt') or data.get('mc_gross')
        currency_code = data.get('cc') or data.get('mc_currency')
        assert amount and currency_code, 'PayPal: missing amount or currency'
        assert self.currency_id.compare_amounts(float(amount), self.amount + self.fees) == 0, \
            'PayPal: mismatching amounts'
        assert currency_code == self.currency_id.name, 'PayPal: mismatching currency codes'

        txn_id = data.get('txn_id')
        txn_type = data.get('txn_type')
        if not all((txn_id, txn_type)):
            raise ValidationError(
                "PayPal: " + _(
                    "Missing value for txn_id (%(txn_id)s) or txn_type (%(txn_type)s).",
                    txn_id=txn_id, txn_type=txn_type
                )
            )
        self.acquirer_reference = txn_id
        self.paypal_type = txn_type

        payment_status = data.get('payment_status')

        if payment_status in PAYMENT_STATUS_MAPPING['pending']:
            self._set_pending(state_message=data.get('pending_reason'))
        elif payment_status in PAYMENT_STATUS_MAPPING['done']:
            self._set_done()
        elif payment_status in PAYMENT_STATUS_MAPPING['cancel']:
            self._set_canceled()
        else:
            _logger.info("received data with invalid payment status: %s", payment_status)
            self._set_error(
                "PayPal: " + _("Received data with invalid payment status: %s", payment_status)
            )

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_payment_method
from . import payment_acquirer
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

## File: views\payment_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_acquirer_form" model="ir.ui.view">
        <field name="name">PayPal Acquirer Form</field>
        <field name="model">payment.acquirer</field>
        <field name="inherit_id" ref="payment.payment_acquirer_form"/>
        <field name="arch" type="xml">
            <xpath expr='//group[@name="acquirer"]' position='inside'>
                <group attrs="{'invisible': [('provider', '!=', 'paypal')]}">
                    <field name="paypal_email_account"
                           attrs="{'required':[('provider', '=', 'paypal'), ('state', '!=', 'disabled')]}"/>
                    <!-- This field should no longer be used but is kept in debug mode for the time
                         being, until we are sure that the verification protocol of IPN can be used
                         for DPT notifications -->
                    <field name="paypal_pdt_token"/>
                    <field name="paypal_use_ipn"
                           attrs="{'required':[('provider', '=', 'paypal'), ('state', '!=', 'disabled')]}"/>
                    <a href="https://www.odoo.com/documentation/15.0/applications/general/payment_acquirers/paypal.html"
                       target="_blank"
                       colspan="2">
                        How to configure your paypal account?
                    </a>
                </group>
            </xpath>
        </field>
    </record>

    <record id="payment_transaction_form" model="ir.ui.view">
        <field name="name">PayPal Transaction Form</field>
        <field name="model">payment.transaction</field>
        <field name="inherit_id" ref="payment.payment_transaction_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='acquirer_reference']" position="after">
                <field name="paypal_type"
                       readonly="1"
                       attrs="{'invisible': [('provider', '!=', 'paypal')]}"
                       groups="base.group_no_one"/>
            </xpath>
        </field>
    </record>

</odoo>

```

