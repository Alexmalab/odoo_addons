# Odoo Module: payment_stripe

Category: Accounting/Payment Providers

This file contains the source code of the Odoo module.

## File: const.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

API_VERSION = '2019-05-16'  # The API version of Stripe implemented in this module

# Stripe proxy URL
PROXY_URL = 'https://stripe.api.odoo.com/api/stripe/'

# The codes of the payment methods to activate when Stripe is activated.
DEFAULT_PAYMENT_METHOD_CODES = {
    # Primary payment methods.
    'card',
    'bancontact',
    'eps',
    'giropay',
    'ideal',
    'p24',
    # Brand payment methods.
    'visa',
    'mastercard',
    'amex',
    'discover',
}

# Mapping of payment method codes to Stripe codes.
PAYMENT_METHODS_MAPPING = {
    'ach_direct_debit': 'us_bank_account',
    'bacs_direct_debit': 'bacs_debit',
    'becs_direct_debit': 'au_becs_debit',
    'sepa_direct_debit': 'sepa_debit',
    'afterpay': 'afterpay_clearpay',
    'clearpay': 'afterpay_clearpay',
    'cash_app_pay': 'cashapp',
    'mobile_pay': 'mobilepay',
    'unknown': 'card',  # For express checkout.
}

INDIAN_MANDATES_SUPPORTED_CURRENCIES = [
    'USD',
    'EUR',
    'GBP',
    'SGD',
    'CAD',
    'CHF',
    'SEK',
    'AED',
    'JPY',
    'NOK',
    'MYR',
    'HKD',
]

# Mapping of transaction states to Stripe objects ({Payment,Setup}Intent, Refund) statuses.
# For each object's exhaustive status list, see:
# https://stripe.com/docs/api/payment_intents/object#payment_intent_object-status
# https://stripe.com/docs/api/setup_intents/object#setup_intent_object-status
# https://stripe.com/docs/api/refunds/object#refund_object-status
STATUS_MAPPING = {
    'draft': ('requires_confirmation', 'requires_action'),
    'pending': ('processing', 'pending'),
    'authorized': ('requires_capture',),
    'done': ('succeeded',),
    'cancel': ('canceled',),
    'error': ('requires_payment_method', 'failed',),
}

# Events which are handled by the webhook
HANDLED_WEBHOOK_EVENTS = [
    'payment_intent.processing',
    'payment_intent.amount_capturable_updated',
    'payment_intent.succeeded',
    'payment_intent.payment_failed',
    'payment_intent.canceled',
    'setup_intent.succeeded',
    'charge.refunded',  # A refund has been issued.
    'charge.refund.updated',  # The refund status has changed, possibly from succeeded to failed.
]

# The countries supported by Stripe. See https://stripe.com/global page.
SUPPORTED_COUNTRIES = {
    'AE',
    'AT',
    'AU',
    'BE',
    'BG',
    'BR',
    'CA',
    'CH',
    'CY',
    'CZ',
    'DE',
    'DK',
    'EE',
    'ES',
    'FI',
    'FR',
    'GB',
    'GI',  # Beta
    'GR',
    'HK',
    'HR',  # Beta
    'HU',
    'ID',  # Beta
    'IE',
    'IT',
    'JP',
    'LI',  # Beta
    'LT',
    'LU',
    'LV',
    'MT',
    'MX',
    'MY',
    'NL',
    'NO',
    'NZ',
    'PH',  # Beta
    'PL',
    'PT',
    'RO',
    'SE',
    'SG',
    'SI',
    'SK',
    'TH',  # Beta
    'US',
}

# Businesses in supported outlying territories should register for a Stripe account with the parent
# territory selected as the Country.
# See https://support.stripe.com/questions/stripe-availability-for-outlying-territories-of-supported-countries.
COUNTRY_MAPPING = {
    'MQ': 'FR',  # Martinique
    'GP': 'FR',  # Guadeloupe
    'GF': 'FR',  # French Guiana
    'RE': 'FR',  # Réunion
    'YT': 'FR',  # Mayotte
    'MF': 'FR',  # Saint-Martin
}

```

## File: utils.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

def get_publishable_key(provider_sudo):
    """ Return the publishable key for Stripe.

    Note: This method serves as a hook for modules that would fully implement Stripe Connect.

    :param recordset provider_sudo: The provider on which the key should be read, as a sudoed
                                    `payment.provider` record.
    :return: The publishable key
    :rtype: str
    """
    return provider_sudo.stripe_publishable_key


def get_secret_key(provider_sudo):
    """ Return the secret key for Stripe.

    Note: This method serves as a hook for modules that would fully implement Stripe Connect.

    :param recordset provider_sudo: The provider on which the key should be read, as a sudoed
                                    `payment.provider` record.
    :return: The secret key
    :rtype: str
    """
    return provider_sudo.stripe_secret_key


def get_webhook_secret(provider_sudo):
    """ Return the webhook secret for Stripe.

    Note: This method serves as a hook for modules that would fully implement Stripe Connect.

    :param recordset provider_sudo: The provider on which the key should be read, as a sudoed
                                    `payment.provider` record.
    :returns: The webhook secret
    :rtype: str
    """
    return provider_sudo.stripe_webhook_secret


def include_shipping_address(tx_sudo):
    """ Include the shipping address of the related sales order or invoice to the payload of the API
    request. If no related sales order or invoice exists, the addres is not included.

    Note: `self.ensure_one()`

    :param payment.transaction tx_sudo: The sudoed transaction of the payment.
    :return: The subset of the API payload that includes the billing and delivery addresses.
    :rtype: dict
    """
    tx_sudo.ensure_one()

    if 'sale_order_ids' in tx_sudo._fields and tx_sudo.sale_order_ids:
        order = tx_sudo.sale_order_ids[:1]
        return format_shipping_address(order.partner_shipping_id)
    elif 'invoice_ids' in tx_sudo._fields and tx_sudo.invoice_ids:
        invoice = tx_sudo.invoice_ids[:1]
        return format_shipping_address(invoice.partner_shipping_id)
    return {}


def format_shipping_address(shipping_partner):
    """ Format the shipping address to comply with the payload structure of the API request.

    :param res.partner shipping_partner: The shipping partner.
    :return: The formatted shipping address.
    :rtype: dict
    """
    return {
        'shipping[address][city]': shipping_partner.city,
        'shipping[address][country]': shipping_partner.country_id.code,
        'shipping[address][line1]': shipping_partner.street,
        'shipping[address][line2]': shipping_partner.street2,
        'shipping[address][postal_code]': shipping_partner.zip,
        'shipping[address][state]': shipping_partner.state_id.name,
        'shipping[name]': shipping_partner.name or shipping_partner.parent_id.name,
    }

```

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

import odoo.addons.payment as payment  # prevent circular import error with payment


def post_init_hook(env):
    payment.setup_provider(env, 'stripe')


def uninstall_hook(env):
    payment.reset_payment_provider(env, 'stripe')

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Payment Provider: Stripe',
    'version': '2.0',
    'category': 'Accounting/Payment Providers',
    'sequence': 350,
    'summary': "An Irish-American payment provider covering the US and many others.",
    'description': " ",  # Non-empty string to avoid loading the README file.
    'depends': ['payment'],
    'data': [
        'views/payment_provider_views.xml',
        'views/payment_stripe_templates.xml',
        'views/payment_templates.xml',  # Only load the SDK on pages with a payment form.

        'data/payment_provider_data.xml',  # Depends on views/payment_stripe_templates.xml
    ],
    'post_init_hook': 'post_init_hook',
    'uninstall_hook': 'uninstall_hook',
    'assets': {
        'web.assets_frontend': [
            'payment_stripe/static/src/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import hashlib
import hmac
import logging
import pprint
from datetime import datetime

from werkzeug.exceptions import Forbidden

from odoo import http
from odoo.exceptions import ValidationError
from odoo.http import request
from odoo.tools.misc import file_open

from odoo.addons.payment import utils as payment_utils
from odoo.addons.payment_stripe import utils as stripe_utils
from odoo.addons.payment_stripe.const import HANDLED_WEBHOOK_EVENTS


_logger = logging.getLogger(__name__)


class StripeController(http.Controller):
    _return_url = '/payment/stripe/return'
    _webhook_url = '/payment/stripe/webhook'
    _apple_pay_domain_association_url = '/.well-known/apple-developer-merchantid-domain-association'
    WEBHOOK_AGE_TOLERANCE = 10*60  # seconds

    @http.route(_return_url, type='http', methods=['GET'], auth='public')
    def stripe_return(self, **data):
        """ Process the notification data sent by Stripe after redirection from payment.

        Customers go through this route regardless of whether the payment was direct or with
        redirection to Stripe or to an external service (e.g., for strong authentication).

        :param dict data: The notification data, including the reference appended to the URL in
                          `_get_specific_processing_values`.
        """
        # Retrieve the transaction based on the reference included in the return url.
        tx_sudo = request.env['payment.transaction'].sudo()._get_tx_from_notification_data(
            'stripe', data
        )

        if tx_sudo.operation != 'validation':
            # Fetch the PaymentIntent and PaymentMethod objects from Stripe.
            payment_intent = tx_sudo.provider_id._stripe_make_request(
                f'payment_intents/{data.get("payment_intent")}',
                payload={'expand[]': 'payment_method'},  # Expand all required objects.
                method='GET',
            )
            _logger.info("Received payment_intents response:\n%s", pprint.pformat(payment_intent))
            self._include_payment_intent_in_notification_data(payment_intent, data)
        else:
            # Fetch the SetupIntent and PaymentMethod objects from Stripe.
            setup_intent = tx_sudo.provider_id._stripe_make_request(
                f'setup_intents/{data.get("setup_intent")}',
                payload={'expand[]': 'payment_method'},  # Expand all required objects.
                method='GET',
            )
            _logger.info("Received setup_intents response:\n%s", pprint.pformat(setup_intent))
            self._include_setup_intent_in_notification_data(setup_intent, data)

        # Handle the notification data crafted with Stripe API's objects.
        tx_sudo._handle_notification_data('stripe', data)

        # Redirect the user to the status page.
        return request.redirect('/payment/status')

    @http.route(_webhook_url, type='http', methods=['POST'], auth='public', csrf=False)
    def stripe_webhook(self):
        """ Process the notification data sent by Stripe to the webhook.

        :return: An empty string to acknowledge the notification.
        :rtype: str
        """
        event = request.get_json_data()
        _logger.info("Notification received from Stripe with data:\n%s", pprint.pformat(event))
        try:
            if event['type'] in HANDLED_WEBHOOK_EVENTS:
                stripe_object = event['data']['object']  # {Payment,Setup}Intent, Charge, or Refund.

                # Check the integrity of the event.
                data = {
                    'reference': stripe_object.get('description'),
                    'event_type': event['type'],
                    'object_id': stripe_object['id'],
                }
                tx_sudo = request.env['payment.transaction'].sudo()._get_tx_from_notification_data(
                    'stripe', data
                )
                self._verify_notification_signature(tx_sudo)

                # Handle the notification data.
                if event['type'].startswith('payment_intent'):  # Payment operation.
                    if tx_sudo.tokenize:
                        payment_method = tx_sudo.provider_id._stripe_make_request(
                            f'payment_methods/{stripe_object["payment_method"]}', method='GET'
                        )
                        _logger.info(
                            "Received payment_methods response:\n%s", pprint.pformat(payment_method)
                        )
                        stripe_object['payment_method'] = payment_method
                    self._include_payment_intent_in_notification_data(stripe_object, data)
                elif event['type'].startswith('setup_intent'):  # Validation operation.
                    # Fetch the missing PaymentMethod object.
                    payment_method = tx_sudo.provider_id._stripe_make_request(
                        f'payment_methods/{stripe_object["payment_method"]}', method='GET'
                    )
                    _logger.info(
                        "Received payment_methods response:\n%s", pprint.pformat(payment_method)
                    )
                    stripe_object['payment_method'] = payment_method
                    self._include_setup_intent_in_notification_data(stripe_object, data)
                elif event['type'] == 'charge.refunded':  # Refund operation (refund creation).
                    refunds = stripe_object['refunds']['data']

                    # The refunds linked to this charge are paginated, fetch the remaining refunds.
                    has_more = stripe_object['refunds']['has_more']
                    while has_more:
                        payload = {
                            'charge': stripe_object['id'],
                            'starting_after': refunds[-1]['id'],
                            'limit': 100,
                        }
                        additional_refunds = tx_sudo.provider_id._stripe_make_request(
                            'refunds', payload=payload, method='GET'
                        )
                        refunds += additional_refunds['data']
                        has_more = additional_refunds['has_more']

                    # Process the refunds for which a refund transaction has not been created yet.
                    processed_refund_ids = tx_sudo.child_transaction_ids.filtered(
                        lambda tx: tx.operation == 'refund'
                    ).mapped('provider_reference')
                    for refund in filter(lambda r: r['id'] not in processed_refund_ids, refunds):
                        refund_tx_sudo = self._create_refund_tx_from_refund(tx_sudo, refund)
                        self._include_refund_in_notification_data(refund, data)
                        refund_tx_sudo._handle_notification_data('stripe', data)
                    # Don't handle the notification data for the source transaction.
                    return request.make_json_response('')
                elif event['type'] == 'charge.refund.updated':  # Refund operation (with update).
                    # A refund was updated by Stripe after it was already processed (possibly to
                    # cancel it). This can happen when the customer's payment method can no longer
                    # be topped up (card expired, account closed...). The `tx_sudo` record is the
                    # refund transaction to update.
                    self._include_refund_in_notification_data(stripe_object, data)

                # Handle the notification data crafted with Stripe API objects
                tx_sudo._handle_notification_data('stripe', data)
        except ValidationError:  # Acknowledge the notification to avoid getting spammed
            _logger.exception("unable to handle the notification data; skipping to acknowledge")
        return request.make_json_response('')

    @staticmethod
    def _include_payment_intent_in_notification_data(payment_intent, notification_data):
        notification_data.update({
            'payment_intent': payment_intent,
            'payment_method': payment_intent.get('payment_method'),
        })

    @staticmethod
    def _include_setup_intent_in_notification_data(setup_intent, notification_data):
        notification_data.update({
            'setup_intent': setup_intent,
            'payment_method': setup_intent.get('payment_method'),
        })

    @staticmethod
    def _include_refund_in_notification_data(refund, notification_data):
        notification_data.update(refund=refund)

    @staticmethod
    def _create_refund_tx_from_refund(source_tx_sudo, refund_object):
        """ Create a refund transaction based on Stripe data.

        :param recordset source_tx_sudo: The source transaction for which a refund is initiated, as
                                         a sudoed `payment.transaction` record.
        :param dict refund_object: The Stripe refund object to create the refund from.
        :return: The created refund transaction.
        :rtype: recordset of `payment.transaction`
        """
        amount_to_refund = refund_object['amount']
        converted_amount = payment_utils.to_major_currency_units(
            amount_to_refund, source_tx_sudo.currency_id
        )
        return source_tx_sudo._create_child_transaction(converted_amount, is_refund=True)

    def _verify_notification_signature(self, tx_sudo):
        """ Check that the received signature matches the expected one.

        See https://stripe.com/docs/webhooks/signatures#verify-manually.

        :param recordset tx_sudo: The sudoed transaction referenced by the notification data, as a
                                  `payment.transaction` record
        :return: None
        :raise: :class:`werkzeug.exceptions.Forbidden` if the timestamp is too old or if the
                signatures don't match
        """
        webhook_secret = stripe_utils.get_webhook_secret(tx_sudo.provider_id)
        if not webhook_secret:
            _logger.warning("ignored webhook event due to undefined webhook secret")
            return

        notification_payload = request.httprequest.data.decode('utf-8')
        signature_entries = request.httprequest.headers['Stripe-Signature'].split(',')
        signature_data = {k: v for k, v in [entry.split('=') for entry in signature_entries]}

        # Retrieve the timestamp from the data
        event_timestamp = int(signature_data.get('t', '0'))
        if not event_timestamp:
            _logger.warning("received notification with missing timestamp")
            raise Forbidden()

        # Check if the timestamp is not too old
        if datetime.utcnow().timestamp() - event_timestamp > self.WEBHOOK_AGE_TOLERANCE:
            _logger.warning("received notification with outdated timestamp: %s", event_timestamp)
            raise Forbidden()

        # Retrieve the received signature from the data
        received_signature = signature_data.get('v1')
        if not received_signature:
            _logger.warning("received notification with missing signature")
            raise Forbidden()

        # Compare the received signature with the expected signature computed from the data
        signed_payload = f'{event_timestamp}.{notification_payload}'
        expected_signature = hmac.new(
            webhook_secret.encode('utf-8'), signed_payload.encode('utf-8'), hashlib.sha256
        ).hexdigest()
        if not hmac.compare_digest(received_signature, expected_signature):
            _logger.warning("received notification with invalid signature")
            raise Forbidden()

    @http.route(_apple_pay_domain_association_url, type='http', auth='public', csrf=False)
    def stripe_apple_pay_get_domain_association_file(self):
        """ Get the domain association file for Stripe's Apple Pay.

        Stripe handles the process of "merchant validation" described in Apple's documentation for
        Apple Pay on the Web. Stripe and Apple will access this route to check the content of the
        file and verify that the web domain is registered.

        See https://stripe.com/docs/stripe-js/elements/payment-request-button#verifying-your-domain-with-apple-pay.

        :return: The content of the domain association file.
        :rtype: str
        """
        return file_open(
            'payment_stripe/static/files/apple-developer-merchantid-domain-association'
        ).read()

```

## File: controllers\onboarding.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http
from odoo.http import request


class OnboardingController(http.Controller):
    _onboarding_return_url = '/payment/stripe/onboarding/return'
    _onboarding_refresh_url = '/payment/stripe/onboarding/refresh'

    @http.route(_onboarding_return_url, type='http', methods=['GET'], auth='user')
    def stripe_return_from_onboarding(self, provider_id, menu_id):
        """ Redirect the user to the provider form of the onboarded Stripe account.

        The user is redirected to this route by Stripe after or during (if the user clicks on a
        dedicated button) the onboarding.

        :param str provider_id: The provider linked to the Stripe account being onboarded, as a
                                `payment.provider` id
        :param str menu_id: The menu from which the user started the onboarding step, as an
                            `ir.ui.menu` id
        """
        stripe_provider = request.env['payment.provider'].browse(int(provider_id))
        request.env['onboarding.onboarding.step'].with_company(
            stripe_provider.company_id
        ).action_validate_step_payment_provider()
        url = f"/odoo/action-payment_stripe.action_payment_provider_onboarding/{provider_id}?menu_id={menu_id}"
        return request.redirect(url)

    @http.route(_onboarding_refresh_url, type='http', methods=['GET'], auth='user')
    def stripe_refresh_onboarding(self, provider_id, account_id, menu_id):
        """ Redirect the user to a new Stripe Connect onboarding link.

        The user is redirected to this route by Stripe if the onboarding link they used was expired.

        :param str provider_id: The provider linked to the Stripe account being onboarded, as a
                                `payment.provider` id
        :param str account_id: The id of the connected account
        :param str menu_id: The menu from which the user started the onboarding step, as an
                            `ir.ui.menu` id
        """
        stripe_provider = request.env['payment.provider'].browse(int(provider_id))
        account_link = stripe_provider._stripe_create_account_link(account_id, int(menu_id))
        return request.redirect(account_link, local=False)

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main
from . import onboarding

```

## File: data\neutralize.sql

```sql
-- disable stripe payment provider
UPDATE payment_provider
   SET stripe_secret_key = NULL,
       stripe_publishable_key = NULL,
       stripe_webhook_secret = NULL;

```

## File: data\payment_provider_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="payment.payment_provider_stripe" model="payment.provider">
        <field name="code">stripe</field>
        <field name="inline_form_view_id" ref="inline_form"/>
        <field name="express_checkout_form_view_id" ref="express_checkout_form"/>
        <field name="allow_tokenization">True</field>
        <field name="allow_express_checkout">True</field>
    </record>

</odoo>

```

## File: models\payment_provider.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json
import logging
import uuid

import requests
from werkzeug.urls import url_encode, url_join, url_parse

from odoo import _, api, fields, models
from odoo.exceptions import RedirectWarning, UserError, ValidationError

from odoo.addons.payment import utils as payment_utils
from odoo.addons.payment_stripe import const, utils as stripe_utils
from odoo.addons.payment_stripe.controllers.main import StripeController
from odoo.addons.payment_stripe.controllers.onboarding import OnboardingController


_logger = logging.getLogger(__name__)


class PaymentProvider(models.Model):
    _inherit = 'payment.provider'

    code = fields.Selection(
        selection_add=[('stripe', "Stripe")], ondelete={'stripe': 'set default'})
    stripe_publishable_key = fields.Char(
        string="Publishable Key", help="The key solely used to identify the account with Stripe",
        required_if_provider='stripe')
    stripe_secret_key = fields.Char(
        string="Secret Key", required_if_provider='stripe', groups='base.group_system')
    stripe_webhook_secret = fields.Char(
        string="Webhook Signing Secret",
        help="If a webhook is enabled on your Stripe account, this signing secret must be set to "
             "authenticate the messages sent from Stripe to Odoo.",
        groups='base.group_system')

    #=== COMPUTE METHODS ===#

    def _compute_feature_support_fields(self):
        """ Override of `payment` to enable additional features. """
        super()._compute_feature_support_fields()
        self.filtered(lambda p: p.code == 'stripe').update({
            'support_express_checkout': True,
            'support_manual_capture': 'full_only',
            'support_refund': 'partial',
            'support_tokenization': True,
        })

    #=== CONSTRAINT METHODS ===#

    @api.constrains('state', 'stripe_publishable_key', 'stripe_secret_key')
    def _check_state_of_connected_account_is_never_test(self):
        """ Check that the provider of a connected account can never been set to 'test'.

        This constraint is defined in the present module to allow the export of the translation
        string of the `ValidationError` should it be raised by modules that would fully implement
        Stripe Connect.

        Additionally, the field `state` is used as a trigger for this constraint to allow those
        modules to indirectly trigger it when writing on custom fields. Indeed, by always writing on
        `state` together with writing on those custom fields, the constraint would be triggered.

        :return: None
        :raise ValidationError: If the provider of a connected account is set in state 'test'.
        """
        for provider in self:
            if provider.state == 'test' and provider._stripe_has_connected_account():
                raise ValidationError(_(
                    "You cannot set the provider to Test Mode while it is linked with your Stripe "
                    "account."
                ))

    def _stripe_has_connected_account(self):
        """ Return whether the provider is linked to a connected Stripe account.

        Note: This method serves as a hook for modules that would fully implement Stripe Connect.
        Note: self.ensure_one()

        :return: Whether the provider is linked to a connected Stripe account
        :rtype: bool
        """
        self.ensure_one()
        return False

    @api.constrains('state')
    def _check_onboarding_of_enabled_provider_is_completed(self):
        """ Check that the provider cannot be set to 'enabled' if the onboarding is ongoing.

        This constraint is defined in the present module to allow the export of the translation
        string of the `ValidationError` should it be raised by modules that would fully implement
        Stripe Connect.

        :return: None
        :raise ValidationError: If the provider of a connected account is set in state 'enabled'
                                while the onboarding is not finished.
        """
        for provider in self:
            if provider.state == 'enabled' and provider._stripe_onboarding_is_ongoing():
                raise ValidationError(_(
                    "You cannot set the provider state to Enabled until your onboarding to Stripe "
                    "is completed."
                ))

    def _stripe_onboarding_is_ongoing(self):
        """ Return whether the provider is linked to an ongoing onboarding to Stripe Connect.

        Note: This method serves as a hook for modules that would fully implement Stripe Connect.
        Note: self.ensure_one()

        :return: Whether the provider is linked to an ongoing onboarding to Stripe Connect
        :rtype: bool
        """
        self.ensure_one()
        return False

    # === ACTION METHODS === #

    def action_stripe_connect_account(self, menu_id=None):
        """ Create a Stripe Connect account and redirect the user to the next onboarding step.

        If the provider is already enabled, close the current window. Otherwise, generate a Stripe
        Connect onboarding link and redirect the user to it. If provided, the menu id is included in
        the URL the user is redirected to when coming back on Odoo after the onboarding. If the link
        generation failed, redirect the user to the provider form.

        Note: This method serves as a hook for modules that would fully implement Stripe Connect.
        Note: self.ensure_one()

        :param int menu_id: The menu from which the user started the onboarding step, as an
                            `ir.ui.menu` id.
        :return: The next step action
        :rtype: dict
        """
        self.ensure_one()

        if self.env.company.country_id.code not in const.SUPPORTED_COUNTRIES:
            raise RedirectWarning(
                _(
                    "Stripe Connect is not available in your country, please use another payment"
                    " provider."
                ),
                self.env.ref('payment.action_payment_provider').id,
                _("Other Payment Providers"),
            )

        if self.state == 'enabled':
            self.env['onboarding.onboarding.step'].action_validate_step_payment_provider()
            action = {'type': 'ir.actions.act_window_close'}
        else:
            # Account creation
            connected_account = self._stripe_fetch_or_create_connected_account()

            # Link generation
            if not menu_id:
                # Fall back on `account_payment`'s menu if it is installed. If not, the user is
                # redirected to the provider's form view but without any menu in the breadcrumb.
                menu = self.env.ref('account_payment.payment_provider_menu', False)
                menu_id = menu and menu.id  # Only set if `account_payment` is installed.

            account_link_url = self._stripe_create_account_link(connected_account['id'], menu_id)
            if account_link_url:
                action = {
                    'type': 'ir.actions.act_url',
                    'url': account_link_url,
                    'target': 'self',
                }
            else:
                action = {
                    'type': 'ir.actions.act_window',
                    'model': 'payment.provider',
                    'views': [[False, 'form']],
                    'res_id': self.id,
                }

        return action

    def action_stripe_create_webhook(self):
        """ Create a webhook and return a feedback notification.

        Note: This action only works for instances using a public URL

        :return: The feedback notification
        :rtype: dict
        """
        self.ensure_one()

        if self.stripe_webhook_secret:
            message = _("Your Stripe Webhook is already set up.")
            notification_type = 'warning'
        elif not self.stripe_secret_key:
            message = _("You cannot create a Stripe Webhook if your Stripe Secret Key is not set.")
            notification_type = 'danger'
        else:
            webhook = self._stripe_make_request(
                'webhook_endpoints', payload={
                    'url': self._get_stripe_webhook_url(),
                    'enabled_events[]': const.HANDLED_WEBHOOK_EVENTS,
                    'api_version': const.API_VERSION,
                }
            )
            self.stripe_webhook_secret = webhook.get('secret')
            message = _("You Stripe Webhook was successfully set up!")
            notification_type = 'info'

        return {
            'type': 'ir.actions.client',
            'tag': 'display_notification',
            'params': {
                'message': message,
                'sticky': False,
                'type': notification_type,
                'next': {'type': 'ir.actions.act_window_close'},  # Refresh the form to show the key
            }
        }

    def action_stripe_verify_apple_pay_domain(self):
        """ Verify the web domain with Stripe to enable Apple Pay.

        The domain is sent to Stripe API for them to verify that it is valid by making a request to
        the `/.well-known/apple-developer-merchantid-domain-association` route. If the domain is
        valid, it is registered to use with Apple Pay.
        See https://stripe.com/docs/stripe-js/elements/payment-request-button#verifying-your-domain-with-apple-pay.

        :return dict: A client action with a success message.
        :raise UserError: If test keys are used to make the request.
        """
        self.ensure_one()

        web_domain = url_parse(self.get_base_url()).netloc
        response_content = self._stripe_make_request('apple_pay/domains', payload={
            'domain_name': web_domain
        })
        if not response_content['livemode']:
            # If test keys are used to make the request, Stripe will respond with an HTTP 200 but
            # will not register the domain. Ask the user to use live credentials.
            raise UserError(_("Please use live credentials to enable Apple Pay."))

        return {
            'type': 'ir.actions.client',
            'tag': 'display_notification',
            'params': {
                'message': _("Your web domain was successfully verified."),
                'type': 'success',
            },
        }

    def _get_stripe_webhook_url(self):
        return url_join(self.get_base_url(), StripeController._webhook_url)

    # === BUSINESS METHODS - PAYMENT FLOW === #

    def _stripe_make_request(
        self, endpoint, payload=None, method='POST', offline=False, idempotency_key=None
    ):
        """ Make a request to Stripe API at the specified endpoint.

        Note: self.ensure_one()

        :param str endpoint: The endpoint to be reached by the request
        :param dict payload: The payload of the request
        :param str method: The HTTP method of the request
        :param bool offline: Whether the operation of the transaction being processed is 'offline'
        :param str idempotency_key: The idempotency key to pass in the request.
        :return The JSON-formatted content of the response
        :rtype: dict
        :raise: ValidationError if an HTTP error occurs
        """
        self.ensure_one()

        url = url_join('https://api.stripe.com/v1/', endpoint)
        headers = {
            'AUTHORIZATION': f'Bearer {stripe_utils.get_secret_key(self)}',
            'Stripe-Version': const.API_VERSION,  # SetupIntent requires a specific version.
            **self._get_stripe_extra_request_headers(),
        }
        if method == 'POST' and idempotency_key:
            headers['Idempotency-Key'] = idempotency_key
        try:
            response = requests.request(method, url, data=payload, headers=headers, timeout=60)
            # Stripe can send 4XX errors for payment failures (not only for badly-formed requests).
            # Check if an error code is present in the response content and raise only if not.
            # See https://stripe.com/docs/error-codes.
            # If the request originates from an offline operation, don't raise to avoid a cursor
            # rollback and return the response as-is for flow-specific handling.
            if not response.ok \
                    and not offline \
                    and 400 <= response.status_code < 500 \
                    and response.json().get('error'):  # The 'code' entry is sometimes missing
                try:
                    response.raise_for_status()
                except requests.exceptions.HTTPError:
                    _logger.exception("invalid API request at %s with data %s", url, payload)
                    error_msg = response.json().get('error', {}).get('message', '')
                    raise ValidationError(
                        "Stripe: " + _(
                            "The communication with the API failed.\n"
                            "Stripe gave us the following info about the problem:\n'%s'", error_msg
                        )
                    )
        except requests.exceptions.ConnectionError:
            _logger.exception("unable to reach endpoint at %s", url)
            raise ValidationError("Stripe: " + _("Could not establish the connection to the API."))
        return response.json()

    def _get_stripe_extra_request_headers(self):
        """ Return the extra headers for the Stripe API request.

        Note: This method serves as a hook for modules that would fully implement Stripe Connect.

        :return: The extra request headers.
        :rtype: dict
        """
        return {}

    # === BUSINESS METHODS - STRIPE CONNECT ONBOARDING === #

    def _stripe_fetch_or_create_connected_account(self):
        """ Fetch the connected Stripe account and create one if not already done.

        Note: This method serves as a hook for modules that would fully implement Stripe Connect.

        :return: The connected account
        :rtype: dict
        """
        return self._stripe_make_proxy_request(
            'accounts', payload=self._stripe_prepare_connect_account_payload()
        )

    def _stripe_prepare_connect_account_payload(self):
        """ Prepare the payload for the creation of a connected account in Stripe format.

        Note: This method serves as a hook for modules that would fully implement Stripe Connect.
        Note: self.ensure_one()

        :return: The Stripe-formatted payload for the creation request
        :rtype: dict
        """
        self.ensure_one()

        return {
            'type': 'standard',
            'country': self._stripe_get_country(self.company_id.country_id.code),
            'email': self.company_id.email,
            'business_type': 'company',
            'company[address][city]': self.company_id.city or '',
            'company[address][country]': self._stripe_get_country(self.company_id.country_id.code),
            'company[address][line1]': self.company_id.street or '',
            'company[address][line2]': self.company_id.street2 or '',
            'company[address][postal_code]': self.company_id.zip or '',
            'company[address][state]': self.company_id.state_id.name or '',
            'company[name]': self.company_id.name,
            'business_profile[name]': self.company_id.name,
        }

    def _stripe_create_account_link(self, connected_account_id, menu_id):
        """ Create an account link and return its URL.

        An account link url is the beginning URL of Stripe Onboarding.
        This URL is only valid once, and can only be used once.

        Note: self.ensure_one()

        :param str connected_account_id: The id of the connected account.
        :param int menu_id: The menu from which the user started the onboarding step, as an
                            `ir.ui.menu` id
        :return: The account link URL
        :rtype: str
        """
        self.ensure_one()

        base_url = self.company_id.get_base_url()
        return_url = OnboardingController._onboarding_return_url
        refresh_url = OnboardingController._onboarding_refresh_url
        return_params = dict(provider_id=self.id, menu_id=menu_id)
        refresh_params = dict(**return_params, account_id=connected_account_id)

        account_link = self._stripe_make_proxy_request('account_links', payload={
            'account': connected_account_id,
            'return_url': f'{url_join(base_url, return_url)}?{url_encode(return_params)}',
            'refresh_url': f'{url_join(base_url, refresh_url)}?{url_encode(refresh_params)}',
            'type': 'account_onboarding',
        })
        return account_link['url']

    def _stripe_make_proxy_request(self, endpoint, payload=None, version=1):
        """ Make a request to the Stripe proxy at the specified endpoint.

        :param str endpoint: The proxy endpoint to be reached by the request
        :param dict payload: The payload of the request
        :param int version: The proxy version used
        :return The JSON-formatted content of the response
        :rtype: dict
        :raise: ValidationError if an HTTP error occurs
        """
        proxy_payload = {
            'jsonrpc': '2.0',
            'id': uuid.uuid4().hex,
            'method': 'call',
            'params': {
                'payload': payload,  # Stripe data.
                'proxy_data': self._stripe_prepare_proxy_data(stripe_payload=payload),
            },
        }
        url = url_join(const.PROXY_URL, f'{version}/{endpoint}')
        try:
            response = requests.post(url=url, json=proxy_payload, timeout=60)
            response.raise_for_status()
        except requests.exceptions.ConnectionError:
            _logger.exception("unable to reach endpoint at %s", url)
            raise ValidationError(_("Stripe Proxy: Could not establish the connection."))
        except requests.exceptions.HTTPError:
            _logger.exception("invalid API request at %s with data %s", url, payload)
            raise ValidationError(
                _("Stripe Proxy: An error occurred when communicating with the proxy.")
            )

        # Stripe proxy endpoints always respond with HTTP 200 as they implement JSON-RPC 2.0
        response_content = response.json()
        if response_content.get('error'):  # An exception was raised on the proxy
            error_data = response_content['error']['data']
            _logger.warning("request forwarded with error: %s", error_data['message'])
            raise ValidationError(_("Stripe Proxy error: %(error)s", error=error_data['message']))

        return response_content.get('result', {})

    def _stripe_prepare_proxy_data(self, stripe_payload=None):
        """ Prepare the contextual data passed to the proxy when making a request.

        Note: This method serves as a hook for modules that would fully implement Stripe Connect.
        Note: self.ensure_one()

        :param dict stripe_payload: The part of the request payload to be forwarded to Stripe.
        :return: The proxy data.
        :rtype: dict
        """
        self.ensure_one()

        return {}

    #=== BUSINESS METHODS - GETTERS ===#

    def _stripe_get_publishable_key(self):
        """ Return the publishable key of the provider.

        This getter allows fetching the publishable key from a QWeb template and through Stripe's
        utils.

        Note: `self.ensure_one()

        :return: The publishable key.
        :rtype: str
        """
        self.ensure_one()

        return stripe_utils.get_publishable_key(self.sudo())

    def _stripe_get_inline_form_values(
        self, amount, currency, partner_id, is_validation, payment_method_sudo=None, **kwargs
    ):
        """ Return a serialized JSON of the required values to render the inline form.

        Note: `self.ensure_one()`

        :param float amount: The amount in major units, to convert in minor units.
        :param res.currency currency: The currency of the transaction.
        :param int partner_id: The partner of the transaction, as a `res.partner` id.
        :param bool is_validation: Whether the operation is a validation.
        :param payment.method payment_method_sudo: The sudoed payment method record to which the
                                                   inline form belongs.
        :return: The JSON serial of the required values to render the inline form.
        :rtype: str
        """
        self.ensure_one()

        if not is_validation:
            currency_name = currency and currency.name.lower()
        else:
            currency_name = self.with_context(
                validation_pm=payment_method_sudo  # Will be converted to a kwarg in master.
            )._get_validation_currency().name.lower()
        partner = self.env['res.partner'].with_context(show_address=1).browse(partner_id).exists()
        inline_form_values = {
            'publishable_key': self._stripe_get_publishable_key(),
            'currency_name': currency_name,
            'minor_amount': amount and payment_utils.to_minor_currency_units(amount, currency),
            'capture_method': 'manual' if self.capture_manually else 'automatic',
            'billing_details': {
                'name': partner.name or '',
                'email': partner.email or '',
                'phone': partner.phone or '',
                'address': {
                    'line1': partner.street or '',
                    'line2': partner.street2 or '',
                    'city': partner.city or '',
                    'state': partner.state_id.code or '',
                    'country': partner.country_id.code or '',
                    'postal_code': partner.zip or '',
                },
            },
            'is_tokenization_required': self._is_tokenization_required(**kwargs),
            'payment_methods_mapping': const.PAYMENT_METHODS_MAPPING,
        }
        return json.dumps(inline_form_values)

    def _stripe_get_country(self, country_code):
        """ Return the mapped country code of the company.

        Businesses in supported outlying territories should register for a Stripe account with the
        parent territory selected as the Country.

        :param str country_code: The country code of the company.
        :return: The mapped country code.
        :rtype: str
        """
        return const.COUNTRY_MAPPING.get(country_code, country_code)

    def _get_default_payment_method_codes(self):
        """ Override of `payment` to return the default payment method codes. """
        default_codes = super()._get_default_payment_method_codes()
        if self.code != 'stripe':
            return default_codes
        return const.DEFAULT_PAYMENT_METHOD_CODES

```

## File: models\payment_token.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import pprint

from odoo import _, fields, models
from odoo.exceptions import ValidationError

_logger = logging.getLogger(__name__)


class PaymentToken(models.Model):
    _inherit = 'payment.token'

    stripe_payment_method = fields.Char(string="Stripe Payment Method ID", readonly=True)
    stripe_mandate = fields.Char(string="Stripe Mandate", readonly=True)

    def _stripe_sca_migrate_customer(self):
        """ Migrate a token from the old implementation of Stripe to the SCA-compliant one.

        In the old implementation, it was possible to create a Charge by giving only the customer id
        and let Stripe use the default source (= default payment method). Stripe now requires to
        specify the payment method for each new PaymentIntent. To do so, we fetch the payment method
        associated to a customer and save its id on the token.
        This migration happens once per token created with the old implementation.

        Note: self.ensure_one()

        :return: None
        """
        self.ensure_one()

        # Fetch the available payment method of type 'card' for the given customer
        response_content = self.provider_id._stripe_make_request(
            'payment_methods',
            payload={
                'customer': self.provider_ref,
                'type': 'card',
                'limit': 1,  # A new customer is created for each new token. Never > 1 card.
            },
            method='GET'
        )
        _logger.info("received payment_methods response:\n%s", pprint.pformat(response_content))

        # Store the payment method ID on the token
        payment_methods = response_content.get('data', [])
        payment_method_id = payment_methods and payment_methods[0].get('id')
        if not payment_method_id:
            raise ValidationError("Stripe: " + _("Unable to convert payment token to new API."))
        self.stripe_payment_method = payment_method_id
        _logger.info("converted token with id %s to new API", self.id)

```

## File: models\payment_transaction.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import pprint

from werkzeug.urls import url_encode, url_join

from odoo import _, fields, models
from odoo.exceptions import UserError, ValidationError

from odoo.addons.payment import utils as payment_utils
from odoo.addons.payment_stripe import const
from odoo.addons.payment_stripe import utils as stripe_utils
from odoo.addons.payment_stripe.controllers.main import StripeController


_logger = logging.getLogger(__name__)


class PaymentTransaction(models.Model):
    _inherit = 'payment.transaction'

    def _get_specific_processing_values(self, processing_values):
        """ Override of payment to return Stripe-specific processing values.

        Note: self.ensure_one() from `_get_processing_values`

        :param dict processing_values: The generic processing values of the transaction
        :return: The dict of provider-specific processing values
        :rtype: dict
        """
        res = super()._get_specific_processing_values(processing_values)
        if self.provider_code != 'stripe' or self.operation == 'online_token':
            return res

        intent = self._stripe_create_intent()
        base_url = self.provider_id.get_base_url()
        return {
            'client_secret': intent['client_secret'],
            'return_url': url_join(
                base_url,
                f'{StripeController._return_url}?{url_encode({"reference": self.reference})}',
            ),
        }

    def _send_payment_request(self):
        """ Override of payment to send a payment request to Stripe with a confirmed PaymentIntent.

        Note: self.ensure_one()

        :return: None
        :raise: UserError if the transaction is not linked to a token
        """
        super()._send_payment_request()
        if self.provider_code != 'stripe':
            return

        if not self.token_id:
            raise UserError("Stripe: " + _("The transaction is not linked to a token."))

        # Make the payment request to Stripe
        payment_intent = self._stripe_create_intent()
        _logger.info(
            "payment request response for transaction with reference %s:\n%s",
            self.reference, pprint.pformat(payment_intent)
        )
        if not payment_intent:  # The PI might be missing if Stripe failed to create it.
            return  # There is nothing to process; the transaction is in error at this point.

        # Handle the payment request response
        notification_data = {'reference': self.reference}
        StripeController._include_payment_intent_in_notification_data(
            payment_intent, notification_data
        )
        self._handle_notification_data('stripe', notification_data)

    def _stripe_create_intent(self):
        """ Create and return a PaymentIntent or a SetupIntent object, depending on the operation.

        :return: The created PaymentIntent or SetupIntent object.
        :rtype: dict
        """
        if self.operation == 'validation':
            response = self.provider_id._stripe_make_request(
                'setup_intents', payload=self._stripe_prepare_setup_intent_payload()
            )
        else:  # 'online_direct', 'online_token', 'offline'.
            response = self.provider_id._stripe_make_request(
                'payment_intents',
                payload=self._stripe_prepare_payment_intent_payload(),
                offline=self.operation == 'offline',
                # Prevent multiple offline payments by token (e.g., due to a cursor rollback).
                idempotency_key=payment_utils.generate_idempotency_key(
                    self, scope='payment_intents_token'
                ) if self.operation == 'offline' else None,
            )

        if 'error' not in response:
            intent = response
        else:  # A processing error was returned in place of the intent.
            # The request failed and no error was raised because we are in an offline payment flow.
            # Extract the error from the response, log it, and set the transaction in error to let
            # the calling module handle the issue without rolling back the cursor.
            error_msg = response['error'].get('message')
            _logger.warning(
                "The creation of the intent failed.\n"
                "Stripe gave us the following info about the problem:\n'%s'", error_msg
            )
            self._set_error("Stripe: " + _(
                "The communication with the API failed.\n"
                "Stripe gave us the following info about the problem:\n'%s'", error_msg
            ))  # Flag transaction as in error now, as the intent status might have a valid value.
            intent = response['error'].get('payment_intent') \
                     or response['error'].get('setup_intent')  # Get the intent from the error.

        return intent

    def _stripe_prepare_setup_intent_payload(self):
        """ Prepare the payload for the creation of a SetupIntent object in Stripe format.

        Note: This method serves as a hook for modules that would fully implement Stripe Connect.

        :return: The Stripe-formatted payload for the SetupIntent request.
        :rtype: dict
        """
        customer = self._stripe_create_customer()
        setup_intent_payload = {
            'customer': customer['id'],
            'description': self.reference,
            'payment_method_types[]': const.PAYMENT_METHODS_MAPPING.get(
                self.payment_method_code, self.payment_method_code
            ),
        }
        if self.currency_id.name in const.INDIAN_MANDATES_SUPPORTED_CURRENCIES:
            setup_intent_payload.update(**self._stripe_prepare_mandate_options())
        return setup_intent_payload

    def _stripe_prepare_payment_intent_payload(self):
        """ Prepare the payload for the creation of a PaymentIntent object in Stripe format.

        Note: This method serves as a hook for modules that would fully implement Stripe Connect.

        :return: The Stripe-formatted payload for the PaymentIntent request.
        :rtype: dict
        """
        ppm_code = self.payment_method_id.primary_payment_method_id.code
        payment_method_type = ppm_code or self.payment_method_code
        payment_intent_payload = {
            'amount': payment_utils.to_minor_currency_units(self.amount, self.currency_id),
            'currency': self.currency_id.name.lower(),
            'description': self.reference,
            'capture_method': 'manual' if self.provider_id.capture_manually else 'automatic',
            'payment_method_types[]': const.PAYMENT_METHODS_MAPPING.get(
                payment_method_type, payment_method_type
            ),
            'expand[]': 'payment_method',
            **stripe_utils.include_shipping_address(self),
        }
        if self.operation in ['online_token', 'offline']:
            if not self.token_id.stripe_payment_method:  # Pre-SCA token, migrate it.
                self.token_id._stripe_sca_migrate_customer()

            payment_intent_payload.update({
                'confirm': True,
                'customer': self.token_id.provider_ref,
                'off_session': True,
                'payment_method': self.token_id.stripe_payment_method,
                'mandate': self.token_id.stripe_mandate or None,
            })
        else:
            customer = self._stripe_create_customer()
            payment_intent_payload['customer'] = customer['id']
            if self.tokenize:
                payment_intent_payload['setup_future_usage'] = 'off_session'
                if self.currency_id.name in const.INDIAN_MANDATES_SUPPORTED_CURRENCIES:
                    payment_intent_payload.update(**self._stripe_prepare_mandate_options())
        return payment_intent_payload

    def _stripe_create_customer(self):
        """ Create and return a Customer.

        :return: The Customer
        :rtype: dict
        """
        customer = self.provider_id._stripe_make_request(
            'customers', payload={
                'address[city]': self.partner_city or None,
                'address[country]': self.partner_country_id.code or None,
                'address[line1]': self.partner_address or None,
                'address[postal_code]': self.partner_zip or None,
                'address[state]': self.partner_state_id.name or None,
                'description': f'Odoo Partner: {self.partner_id.name} (id: {self.partner_id.id})',
                'email': self.partner_email or None,
                'name': self.partner_name,
                'phone': self.partner_phone and self.partner_phone[:20] or None,
            }
        )
        return customer

    def _stripe_prepare_mandate_options(self):
        """ Prepare the configuration options for setting up an eMandate along with an intent.

        :return: The Stripe-formatted payload for the mandate options.
        :rtype: dict
        """
        mandate_values = self._get_mandate_values()

        OPTION_PATH_PREFIX = 'payment_method_options[card][mandate_options]'
        mandate_options = {
            f'{OPTION_PATH_PREFIX}[reference]': self.reference,
            f'{OPTION_PATH_PREFIX}[amount_type]': 'maximum',
            f'{OPTION_PATH_PREFIX}[amount]': payment_utils.to_minor_currency_units(
                mandate_values.get('amount', 15000), self.currency_id
            ),  # Use the specified amount, if any, or define the maximum amount of 15.000 INR.
            f'{OPTION_PATH_PREFIX}[start_date]': int(round(
                (mandate_values.get('start_datetime') or fields.Datetime.now()).timestamp()
            )),
            f'{OPTION_PATH_PREFIX}[interval]': 'sporadic',
            f'{OPTION_PATH_PREFIX}[supported_types][]': 'india',
        }
        if mandate_values.get('end_datetime'):
            mandate_options[f'{OPTION_PATH_PREFIX}[end_date]'] = int(round(
                mandate_values['end_datetime'].timestamp()
            ))
        if mandate_values.get('recurrence_unit') and mandate_values.get('recurrence_duration'):
            mandate_options.update({
                f'{OPTION_PATH_PREFIX}[interval]': mandate_values['recurrence_unit'],
                f'{OPTION_PATH_PREFIX}[interval_count]': mandate_values['recurrence_duration'],
            })
        if self.operation == 'validation':
            currency_name = self.provider_id.with_context(
                validation_pm=self.payment_method_id  # Will be converted to a kwarg in master.
            )._get_validation_currency().name.lower()
            mandate_options[f'{OPTION_PATH_PREFIX}[currency]'] = currency_name

        return mandate_options

    def _send_refund_request(self, amount_to_refund=None):
        """ Override of payment to send a refund request to Stripe.

        Note: self.ensure_one()

        :param float amount_to_refund: The amount to refund.
        :return: The refund transaction created to process the refund request.
        :rtype: recordset of `payment.transaction`
        """
        refund_tx = super()._send_refund_request(amount_to_refund=amount_to_refund)
        if self.provider_code != 'stripe':
            return refund_tx

        # Make the refund request to stripe.
        data = self.provider_id._stripe_make_request(
            'refunds', payload={
                'payment_intent': self.provider_reference,
                'amount': payment_utils.to_minor_currency_units(
                    -refund_tx.amount,  # Refund transactions' amount is negative, inverse it.
                    refund_tx.currency_id,
                ),
            }
        )
        _logger.info(
            "Refund request response for transaction wih reference %s:\n%s",
            self.reference, pprint.pformat(data)
        )
        # Handle the refund request response.
        notification_data = {}
        StripeController._include_refund_in_notification_data(data, notification_data)
        refund_tx._handle_notification_data('stripe', notification_data)

        return refund_tx

    def _send_capture_request(self, amount_to_capture=None):
        """ Override of `payment` to send a capture request to Stripe. """
        child_capture_tx = super()._send_capture_request(amount_to_capture=amount_to_capture)
        if self.provider_code != 'stripe':
            return child_capture_tx

        # Make the capture request to Stripe
        payment_intent = self.provider_id._stripe_make_request(
            f'payment_intents/{self.provider_reference}/capture'
        )
        _logger.info(
            "capture request response for transaction with reference %s:\n%s",
            self.reference, pprint.pformat(payment_intent)
        )

        # Handle the capture request response
        notification_data = {'reference': self.reference}
        StripeController._include_payment_intent_in_notification_data(
            payment_intent, notification_data
        )
        self._handle_notification_data('stripe', notification_data)

        return child_capture_tx

    def _send_void_request(self, amount_to_void=None):
        """ Override of `payment` to send a void request to Stripe. """
        child_void_tx = super()._send_void_request(amount_to_void=amount_to_void)
        if self.provider_code != 'stripe':
            return child_void_tx

        # Make the void request to Stripe
        payment_intent = self.provider_id._stripe_make_request(
            f'payment_intents/{self.provider_reference}/cancel'
        )
        _logger.info(
            "void request response for transaction with reference %s:\n%s",
            self.reference, pprint.pformat(payment_intent)
        )

        # Handle the void request response
        notification_data = {'reference': self.reference}
        StripeController._include_payment_intent_in_notification_data(
            payment_intent, notification_data
        )
        self._handle_notification_data('stripe', notification_data)

        return child_void_tx

    def _get_tx_from_notification_data(self, provider_code, notification_data):
        """ Override of payment to find the transaction based on Stripe data.

        :param str provider_code: The code of the provider that handled the transaction
        :param dict notification_data: The notification data sent by the provider
        :return: The transaction if found
        :rtype: recordset of `payment.transaction`
        :raise: ValidationError if inconsistent data were received
        :raise: ValidationError if the data match no transaction
        """
        tx = super()._get_tx_from_notification_data(provider_code, notification_data)
        if provider_code != 'stripe' or len(tx) == 1:
            return tx

        reference = notification_data.get('reference')
        if reference:
            tx = self.search([('reference', '=', reference), ('provider_code', '=', 'stripe')])
        elif notification_data.get('event_type') == 'charge.refund.updated':
            # The webhook notifications sent for `charge.refund.updated` events only contain a
            # refund object that has no 'description' (the merchant reference) field. We thus search
            # the transaction by its provider reference which is the refund id for refund txs.
            refund_id = notification_data['object_id']  # The object is a refund.
            tx = self.search(
                [('provider_reference', '=', refund_id), ('provider_code', '=', 'stripe')]
            )
        else:
            raise ValidationError("Stripe: " + _("Received data with missing merchant reference"))

        if not tx:
            raise ValidationError(
                "Stripe: " + _("No transaction found matching reference %s.", reference)
            )
        return tx

    def _process_notification_data(self, notification_data):
        """ Override of `payment` to process the transaction based on Stripe data.

        Note: self.ensure_one()

        :param dict notification_data: The notification data build from information passed to the
                                       return route. Depending on the operation of the transaction,
                                       the entries with the keys 'payment_intent', 'setup_intent'
                                       and 'payment_method' can be populated with their
                                       corresponding Stripe API objects.
        :return: None
        :raise: ValidationError if inconsistent data were received
        """
        super()._process_notification_data(notification_data)
        if self.provider_code != 'stripe':
            return

        # Update the payment method.
        payment_method = notification_data.get('payment_method')
        if isinstance(payment_method, dict):  # capture/void/refund requests receive a string.
            payment_method_type = payment_method.get('type')
            if self.payment_method_id.code == payment_method_type == 'card':
                payment_method_type = notification_data['payment_method']['card']['brand']
            payment_method = self.env['payment.method']._get_from_code(
                payment_method_type, mapping=const.PAYMENT_METHODS_MAPPING
            )
            self.payment_method_id = payment_method or self.payment_method_id

        # Update the provider reference and the payment state.
        if self.operation == 'validation':
            self.provider_reference = notification_data['setup_intent']['id']
            status = notification_data['setup_intent']['status']
        elif self.operation == 'refund':
            self.provider_reference = notification_data['refund']['id']
            status = notification_data['refund']['status']
        else:  # 'online_direct', 'online_token', 'offline'
            self.provider_reference = notification_data['payment_intent']['id']
            status = notification_data['payment_intent']['status']
        if not status:
            raise ValidationError(
                "Stripe: " + _("Received data with missing intent status.")
            )
        if status in const.STATUS_MAPPING['draft']:
            pass
        elif status in const.STATUS_MAPPING['pending']:
            self._set_pending()
        elif status in const.STATUS_MAPPING['authorized']:
            if self.tokenize:
                self._stripe_tokenize_from_notification_data(notification_data)
            self._set_authorized()
        elif status in const.STATUS_MAPPING['done']:
            if self.tokenize:
                self._stripe_tokenize_from_notification_data(notification_data)

            self._set_done()

            # Immediately post-process the transaction if it is a refund, as the post-processing
            # will not be triggered by a customer browsing the transaction from the portal.
            if self.operation == 'refund':
                self.env.ref('payment.cron_post_process_payment_tx')._trigger()
        elif status in const.STATUS_MAPPING['cancel']:
            self._set_canceled()
        elif status in const.STATUS_MAPPING['error']:
            if self.operation != 'refund':
                last_payment_error = notification_data.get('payment_intent', {}).get(
                    'last_payment_error'
                )
                if last_payment_error:
                    message = last_payment_error.get('message', {})
                else:
                    message = _("The customer left the payment page.")
                self._set_error(message)
            else:
                self._set_error(_(
                    "The refund did not go through. Please log into your Stripe Dashboard to get "
                    "more information on that matter, and address any accounting discrepancies."
                ), extra_allowed_states=('done',))
        else:  # Classify unknown intent statuses as `error` tx state
            _logger.warning(
                "received invalid payment status (%s) for transaction with reference %s",
                status, self.reference
            )
            self._set_error(_("Received data with invalid intent status: %s", status))

    def _stripe_tokenize_from_notification_data(self, notification_data):
        """ Create a new token based on the notification data.

        :param dict notification_data: The notification data built with Stripe objects.
                                       See `_process_notification_data`.
        :return: None
        """
        payment_method = notification_data.get('payment_method')
        if not payment_method:
            _logger.warning(
                "requested tokenization from notification data with missing payment method"
            )
            return

        mandate = None
        # Extract the Stripe objects from the notification data.
        if self.operation == 'online_direct':
            customer_id = notification_data['payment_intent']['customer']
            charges_data = notification_data['payment_intent']['charges']
            payment_method_details = charges_data['data'][0].get('payment_method_details')
            if payment_method_details:
                mandate = payment_method_details[payment_method_details['type']].get("mandate")
        else:  # 'validation'
            customer_id = notification_data['setup_intent']['customer']
        # Another payment method (e.g., SEPA) might have been generated.
        if not payment_method[payment_method['type']]:
            payment_methods = self.provider_id._stripe_make_request(
                f'customers/{customer_id}/payment_methods', method='GET'
            )
            _logger.info("Received payment_methods response:\n%s", pprint.pformat(payment_methods))
            payment_method = payment_methods['data'][0]

        # Create the token.
        token = self.env['payment.token'].create({
            'provider_id': self.provider_id.id,
            'payment_method_id': self.payment_method_id.id,
            'payment_details': payment_method[payment_method['type']].get('last4'),
            'partner_id': self.partner_id.id,
            'provider_ref': customer_id,
            'stripe_payment_method': payment_method['id'],
            'stripe_mandate': mandate,
        })
        self.write({
            'token_id': token,
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
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path fill-rule="evenodd" clip-rule="evenodd" d="M50 25.367c0-3.76-1.722-6.726-5.014-6.726-3.305 0-5.305 2.966-5.305 6.697 0 4.42 2.36 6.652 5.75 6.652 1.652 0 2.902-.396 3.847-.954v-2.937c-.945.499-2.028.807-3.403.807-1.347 0-2.542-.499-2.694-2.232h6.791c0-.19.028-.955.028-1.307Zm-6.861-1.395c0-1.66.958-2.35 1.833-2.35.847 0 1.75.69 1.75 2.35H43.14ZM34.32 18.64c-1.361 0-2.236.676-2.723 1.146l-.18-.91H28.36V36l3.473-.779.013-4.156c.5.382 1.236.925 2.459.925 2.486 0 4.75-2.114 4.75-6.77-.014-4.259-2.306-6.58-4.736-6.58Zm-.834 10.12c-.82 0-1.305-.309-1.639-.69l-.013-5.45c.36-.425.86-.719 1.652-.719 1.264 0 2.14 1.498 2.14 3.422 0 1.968-.862 3.436-2.14 3.436Zm-9.903-10.986 3.486-.793V14l-3.486.778v2.996Z" fill="#635BFF"/><path d="M27.07 18.89h-3.487v12.851h3.486v-12.85Z" fill="#635BFF"/><path fill-rule="evenodd" clip-rule="evenodd" d="m19.847 19.978-.222-1.087h-3v12.85h3.472v-8.708c.82-1.131 2.209-.926 2.64-.764V18.89c-.445-.176-2.07-.5-2.89 1.087Zm-6.944-4.274-3.39.764L9.5 28.232c0 2.173 1.542 3.774 3.597 3.774 1.14 0 1.972-.22 2.43-.485V28.54c-.444.19-2.638.866-2.638-1.307v-5.214h2.639v-3.128h-2.64l.015-3.187ZM3.514 22.62c0-.572.444-.793 1.18-.793 1.056 0 2.39.338 3.445.94v-3.45a8.72 8.72 0 0 0-3.445-.676C1.875 18.64 0 20.197 0 22.797c0 4.053 5.278 3.407 5.278 5.155 0 .675-.556.896-1.334.896-1.152 0-2.625-.5-3.791-1.175v3.495a9.182 9.182 0 0 0 3.791.837c2.89 0 4.875-1.513 4.875-4.141-.013-4.377-5.305-3.599-5.305-5.243Z" fill="#635BFF"/><path d="M43.105 4h.972V.818h1.108V0H42v.818h1.105V4Zm2.775 0h.858V1.453h.053L47.663 4h.555l.871-2.547h.056V4H50V0h-1.108l-.924 2.714h-.05L46.99 0h-1.11v4Z" fill="#D1D5DB"/></svg>

```

## File: static\files\apple-developer-merchantid-domain-association

```text
7B227073704964223A2239373943394538343346343131343044463144313834343232393232313734313034353044314339464446394437384337313531303944334643463542433731222C2276657273696F6E223A312C22637265617465644F6E223A313536363233343735303036312C227369676E6174757265223A22333038303036303932613836343838366637306430313037303261303830333038303032303130313331306633303064303630393630383634383031363530333034303230313035303033303830303630393261383634383836663730643031303730313030303061303830333038323033653333303832303338386130303330323031303230323038346333303431343935313964353433363330306130363038326138363438636533643034303330323330376133313265333032633036303335353034303330633235343137303730366336353230343137303730366336393633363137343639366636653230343936653734363536373732363137343639366636653230343334313230326432303437333333313236333032343036303335353034306230633164343137303730366336353230343336353732373436393636363936333631373436393666366532303431373537343638366637323639373437393331313333303131303630333535303430613063306134313730373036633635323034393665363332653331306233303039303630333535303430363133303235353533333031653137306433313339333033353331333833303331333333323335333735613137306433323334333033353331333633303331333333323335333735613330356633313235333032333036303335353034303330633163363536333633326437333664373032643632373236663662363537323264373336393637366535663535343333343264353035323466343433313134333031323036303335353034306230633062363934663533323035333739373337343635366437333331313333303131303630333535303430613063306134313730373036633635323034393665363332653331306233303039303630333535303430363133303235353533333035393330313330363037326138363438636533643032303130363038326138363438636533643033303130373033343230303034633231353737656465626436633762323231386636386464373039306131323138646337623062643666326332383364383436303935643934616634613534313162383334323065643831316633343037653833333331663163353463336637656233323230643662616435643465666634393238393839336537633066313361333832303231313330383230323064333030633036303335353164313330313031666630343032333030303330316630363033353531643233303431383330313638303134323366323439633434663933653465663237653663346636323836633366613262626664326534623330343530363038326230363031303530353037303130313034333933303337333033353036303832623036303130353035303733303031383632393638373437343730336132663266366636333733373032653631373037303663363532653633366636643266366636333733373033303334326436313730373036633635363136393633363133333330333233303832303131643036303335353164323030343832303131343330383230313130333038323031306330363039326138363438383666373633363430353031333038316665333038316333303630383262303630313035303530373032303233303831623630633831623335323635366336393631366536333635323036663665323037343638363937333230363336353732373436393636363936333631373436353230363237393230363136653739323037303631373237343739323036313733373337353664363537333230363136333633363537303734363136653633363532303666363632303734363836353230373436383635366532303631373037303663363936333631363236633635323037333734363136653634363137323634323037343635373236643733323036313665363432303633366636653634363937343639366636653733323036663636323037353733363532633230363336353732373436393636363936333631373436353230373036663663363936333739323036313665363432303633363537323734363936363639363336313734363936663665323037303732363136333734363936333635323037333734363137343635366436353665373437333265333033363036303832623036303130353035303730323031313632613638373437343730336132663266373737373737326536313730373036633635326536333666366432663633363537323734363936363639363336313734363536313735373436383666373236393734373932663330333430363033353531643166303432643330326233303239613032376130323538363233363837343734373033613266326636333732366332653631373037303663363532653633366636643266363137303730366336353631363936333631333332653633373236633330316430363033353531643065303431363034313439343537646236666435373438313836383938393736326637653537383530376537396235383234333030653036303335353164306630313031666630343034303330323037383033303066303630393261383634383836663736333634303631643034303230353030333030613036303832613836343863653364303430333032303334393030333034363032323130306265303935373166653731653165373335623535653561666163623463373266656234343566333031383532323263373235313030326236316562643666353530323231303064313862333530613564643664643665623137343630333562313165623263653837636661336536616636636264383338303839306463383263646461613633333038323032656533303832303237356130303330323031303230323038343936643266626633613938646139373330306130363038326138363438636533643034303330323330363733313162333031393036303335353034303330633132343137303730366336353230353236663666373432303433343132303264323034373333333132363330323430363033353530343062306331643431373037303663363532303433363537323734363936363639363336313734363936663665323034313735373436383666373236393734373933313133333031313036303335353034306130633061343137303730366336353230343936653633326533313062333030393036303335353034303631333032353535333330316531373064333133343330333533303336333233333334333633333330356131373064333233393330333533303336333233333334333633333330356133303761333132653330326330363033353530343033306332353431373037303663363532303431373037303663363936333631373436393666366532303439366537343635363737323631373436393666366532303433343132303264323034373333333132363330323430363033353530343062306331643431373037303663363532303433363537323734363936363639363336313734363936663665323034313735373436383666373236393734373933313133333031313036303335353034306130633061343137303730366336353230343936653633326533313062333030393036303335353034303631333032353535333330353933303133303630373261383634386365336430323031303630383261383634386365336430333031303730333432303030346630313731313834313964373634383564353161356532353831303737366538383061326566646537626165346465303864666334623933653133333536643536363562333561653232643039373736306432323465376262613038666437363137636538386362373662623636373062656338653832393834666635343435613338316637333038316634333034363036303832623036303130353035303730313031303433613330333833303336303630383262303630313035303530373330303138363261363837343734373033613266326636663633373337303265363137303730366336353265363336663664326636663633373337303330333432643631373037303663363537323666366637343633363136373333333031643036303335353164306530343136303431343233663234396334346639336534656632376536633466363238366333666132626266643265346233303066303630333535316431333031303166663034303533303033303130316666333031663036303335353164323330343138333031363830313462626230646561313538333338383961613438613939646562656264656261666461636232346162333033373036303335353164316630343330333032653330326361303261613032383836323636383734373437303361326632663633373236633265363137303730366336353265363336663664326636313730373036633635373236663666373436333631363733333265363337323663333030653036303335353164306630313031666630343034303330323031303633303130303630613261383634383836663736333634303630323065303430323035303033303061303630383261383634386365336430343033303230333637303033303634303233303361636637323833353131363939623138366662333563333536636136326266663431376564643930663735346461323865626566313963383135653432623738396638393866373962353939663938643534313064386639646539633266653032333033323264643534343231623061333035373736633564663333383362393036376664313737633263323136643936346663363732363938323132366635346638376137643162393963623962303938393231363130363939306630393932316430303030333138323031386233303832303138373032303130313330383138363330376133313265333032633036303335353034303330633235343137303730366336353230343137303730366336393633363137343639366636653230343936653734363536373732363137343639366636653230343334313230326432303437333333313236333032343036303335353034306230633164343137303730366336353230343336353732373436393636363936333631373436393666366532303431373537343638366637323639373437393331313333303131303630333535303430613063306134313730373036633635323034393665363332653331306233303039303630333535303430363133303235353533303230383463333034313439353139643534333633303064303630393630383634383031363530333034303230313035303061303831393533303138303630393261383634383836663730643031303930333331306230363039326138363438383666373064303130373031333031633036303932613836343838366637306430313039303533313066313730643331333933303338333133393331333733313332333333303561333032613036303932613836343838366637306430313039333433313164333031623330306430363039363038363438303136353033303430323031303530306131306130363038326138363438636533643034303330323330326630363039326138363438383666373064303130393034333132323034323062303731303365313430613462386231376262613230316130336163643036396234653431366232613263383066383661383338313435633239373566633131333030613036303832613836343863653364303430333032303434363330343430323230343639306264636637626461663833636466343934396534633035313039656463663334373665303564373261313264376335666538633033303033343464663032323032363764353863393365626233353031333836363062353730373938613064643731313734316262353864626436613138363633353038353431656565393035303030303030303030303030227D
```

## File: static\src\js\express_checkout_form.js

```javascript
/** @odoo-module **/
/* global Stripe */

import { _t } from '@web/core/l10n/translation';
import { rpc } from "@web/core/network/rpc";
import { paymentExpressCheckoutForm } from '@payment/js/express_checkout_form';
import { StripeOptions } from '@payment_stripe/js/stripe_options';

paymentExpressCheckoutForm.include({
    /**
     * Get the order details to display on the payment form.
     *
     * @private
     * @param {number} deliveryAmount - The delivery costs.
     * @param {number} amountFreeShipping - The free shipping discount amount, <= 0.
     * @returns {Object} The information to be displayed on the payment form.
     */
    _getOrderDetails(deliveryAmount, amountFreeShipping) {
        const pending = this.paymentContext['shippingInfoRequired'] && deliveryAmount === undefined;
        const orderDeliveryAmount = parseInt(this.paymentContext['deliveryAmount'])
        let minorAmount = parseInt(this.paymentContext['minorAmount'])
        if (orderDeliveryAmount) { // The delivery method is set on the order.
            // Subtract the delivery amount from the total amount to display the right total.
            minorAmount -= orderDeliveryAmount;
        }
        const displayItems = [
            {
                label: _t("Your order"),
                amount: minorAmount,
            },
        ];
        if (this.paymentContext['shippingInfoRequired'] && deliveryAmount !== undefined) {
            displayItems.push({
                label: _t("Delivery"),
                amount: deliveryAmount,
            });
        }
        if (amountFreeShipping) {
            displayItems.push({
                label: _t("Free Shipping"),
                amount: amountFreeShipping,
            });
        }
        return {
            total: {
                label: this.paymentContext['merchantName'],
                amount: minorAmount + (deliveryAmount ?? 0) + (amountFreeShipping ?? 0),
                // Delay the display of the amount until the shipping price is retrieved.
                pending: pending,
            },
            displayItems: displayItems,
        };
    },

    /**
     * Prepare the express checkout form of Stripe for direct payment.
     *
     * @override method from payment.express_form
     * @private
     * @param {Object} providerData - The provider-specific data.
     * @return {void}
     */
    async _prepareExpressCheckoutForm(providerData) {
        /*
         * When applying a coupon, the amount can be totally covered, with nothing left to pay. In
         * that case, the check is whether the variable is defined because the server doesn't send
         * the value when it equals '0'.
         */
        if (providerData.providerCode !== 'stripe' || !this.paymentContext['amount']) {
            this._super(...arguments);
            return;
        }

        const stripeJS = Stripe(
            providerData.stripePublishableKey,
            new StripeOptions()._prepareStripeOptions(providerData),
        );
        const paymentRequest = stripeJS.paymentRequest({
            country: providerData.countryCode,
            currency: this.paymentContext['currencyName'],
            requestPayerName: true, // Force fetching the billing address for Apple Pay.
            requestPayerEmail: true,
            requestPayerPhone: true,
            requestShipping: this.paymentContext['shippingInfoRequired'],
            ...this._getOrderDetails(),
        });
        if (this.stripePaymentRequests === undefined) {
            this.stripePaymentRequests = [];
        }
        this.stripePaymentRequests.push(paymentRequest);
        const paymentRequestButton = stripeJS.elements().create('paymentRequestButton', {
            paymentRequest: paymentRequest,
            style: {paymentRequestButton: {type: 'buy'}},
        });

        // Check the availability of the Payment Request API first.
        const canMakePayment = await paymentRequest.canMakePayment();
        if (canMakePayment) {
            paymentRequestButton.mount(
                `#o_stripe_express_checkout_container_${providerData.providerId}`
            );
        } else {
            document.querySelector(
                `#o_stripe_express_checkout_container_${providerData.providerId}`
            ).style.display = 'none';
        }

        paymentRequest.on('paymentmethod', async (ev) => {
            const addresses = {
                'billing_address': {
                    name: ev.payerName,
                    email: ev.payerEmail,
                    phone: ev.payerPhone,
                    street: ev.paymentMethod.billing_details.address.line1,
                    street2: ev.paymentMethod.billing_details.address.line2,
                    zip: ev.paymentMethod.billing_details.address.postal_code,
                    city: ev.paymentMethod.billing_details.address.city,
                    country: ev.paymentMethod.billing_details.address.country,
                    state: ev.paymentMethod.billing_details.address.state,
                }
            };
            if (this.paymentContext['shippingInfoRequired']) {
                addresses.shipping_address = {
                    name: ev.shippingAddress.recipient,
                    email: ev.payerEmail,
                    phone: ev.shippingAddress.phone || ev.payerPhone,
                    street: ev.shippingAddress.addressLine[0],
                    street2: ev.shippingAddress.addressLine[1],
                    zip: ev.shippingAddress.postalCode,
                    city: ev.shippingAddress.city,
                    country: ev.shippingAddress.country,
                    state: ev.shippingAddress.region,
                };
                addresses.shipping_option = ev.shippingOption;
            }
            // Update the customer addresses on the related document.
            this.paymentContext.partnerId = parseInt(await rpc(
                this.paymentContext['expressCheckoutRoute'],
                addresses,
            ));
            // Call the transaction route to create the transaction and retrieve the client secret.
            const { client_secret } = await rpc(
                this.paymentContext['transactionRoute'],
                this._prepareTransactionRouteParams(providerData.providerId),
            );
            // Confirm the PaymentIntent without handling eventual next actions (e.g. 3DS).
            const { paymentIntent, error: confirmError } = await stripeJS.confirmCardPayment(
                client_secret, {payment_method: ev.paymentMethod.id}, {handleActions: false}
            );
            if (confirmError) {
                // Report to the browser that the payment failed, prompting it to re-show the
                // payment interface, or show an error message and close the payment interface.
                ev.complete('fail');
            } else {
                // Report to the browser that the confirmation was successful, prompting it to close
                // the browser payment method collection interface.
                ev.complete('success');
                if (paymentIntent.status === 'requires_action') { // A next step is required.
                    await stripeJS.confirmCardPayment(client_secret); // Trigger the step.
                }
                window.location = '/payment/status';
            }
        });

        if (this.paymentContext['shippingInfoRequired']) {
            // Wait until the express checkout form is loaded for Apple Pay and Google Pay to select
            // a default shipping address and trigger the `shippingaddresschange` event, so we can
            // fetch the available shipping options. When the customer manually selects a different
            // shipping address, the shipping options need to be fetched again.
            paymentRequest.on('shippingaddresschange', async (ev) => {
                // Call the shipping address update route to fetch the shipping options.
                const availableCarriers = await rpc(
                    this.paymentContext['shippingAddressUpdateRoute'],
                    {
                        partial_delivery_address: {
                            zip: ev.shippingAddress.postalCode,
                            city: ev.shippingAddress.city,
                            country: ev.shippingAddress.country,
                            state: ev.shippingAddress.region,
                        },
                    },
                );
                if (availableCarriers.length === 0) {
                    ev.updateWith({status: 'invalid_shipping_address'});
                } else {
                    ev.updateWith({
                        status: 'success',
                        shippingOptions: availableCarriers.map(carrier => ({
                            id: String(carrier.id),
                            label: carrier.name,
                            detail: carrier.description ? carrier.description:'',
                            amount: carrier.minorAmount,
                        })),
                        ...this._getOrderDetails(availableCarriers[0].minorAmount),
                    });
                }
            });

            // When the customer selects a different shipping option, update the displayed total.
            paymentRequest.on('shippingoptionchange', async (ev) => {
                const result = await rpc('/shop/set_delivery_method', {
                    dm_id: parseInt(ev.shippingOption.id),
                });
                ev.updateWith({
                    status: 'success',
                    ...this._getOrderDetails(
                        ev.shippingOption.amount,
                        parseInt(result.delivery_discount_minor_amount) || 0,
                    ),
                });
            });
        }
    },

    /**
     * Update the amount of the express checkout form.
     *
     * @override method from payment.express_form
     * @private
     * @param {number} newAmount - The new amount.
     * @param {number} newMinorAmount - The new minor amount.
     * @return {void}
     */
    _updateAmount(newAmount, newMinorAmount) {
        this._super(...arguments);
        this.stripePaymentRequests && this.stripePaymentRequests.map(
            paymentRequest => paymentRequest.update(this._getOrderDetails())
        );
    },
});

```

## File: static\src\js\payment_form.js

```javascript
/** @odoo-module */
/* global Stripe */

import { _t } from '@web/core/l10n/translation';
import { StripeOptions } from '@payment_stripe/js/stripe_options';
import paymentForm from '@payment/js/payment_form';

paymentForm.include({

    // #=== DOM MANIPULATION ===#

    /**
     * Prepare the inline form of Stripe for direct payment.
     *
     * @override method from @payment/js/payment_form
     * @private
     * @param {number} providerId - The id of the selected payment option's provider.
     * @param {string} providerCode - The code of the selected payment option's provider.
     * @param {number} paymentOptionId - The id of the selected payment option
     * @param {string} paymentMethodCode - The code of the selected payment method, if any.
     * @param {string} flow - The online payment flow of the selected payment option.
     * @return {void}
     */
    async _prepareInlineForm(providerId, providerCode, paymentOptionId, paymentMethodCode, flow) {
        if (providerCode !== 'stripe') {
            this._super(...arguments);
            return;
        }

        // Check if instantiation of the element is needed.
        this.stripeElements ??= {}; // Store the element of each instantiated payment method.
        // Check if instantiation of the element is needed.
        if (flow === 'token') {
            return; // No elements for tokens.
        } else if (this.stripeElements[paymentOptionId]) {
            this._setPaymentFlow('direct'); // Overwrite the flow even if no re-instantiation.
            return; // Don't re-instantiate if already done for this provider.
        }

        // Overwrite the flow of the select payment option.
        this._setPaymentFlow('direct');

        // Extract and deserialize the inline form values.
        const radio = document.querySelector('input[name="o_payment_radio"]:checked');
        const inlineForm = this._getInlineForm(radio);
        const stripeInlineForm = inlineForm.querySelector('[name="o_stripe_element_container"]');
        this.stripeInlineFormValues = JSON.parse(
            stripeInlineForm.dataset['stripeInlineFormValues']
        );

        // Instantiate Stripe object if needed.
        this.stripeJS ??= Stripe(
            this.stripeInlineFormValues['publishable_key'],
            // The values required by Stripe Connect are inserted into the dataset.
            new StripeOptions()._prepareStripeOptions(stripeInlineForm.dataset),
        );

        // Instantiate the elements.
        let elementsOptions =  {
            appearance: { theme: 'stripe' },
            currency: this.stripeInlineFormValues['currency_name'],
            captureMethod: this.stripeInlineFormValues['capture_method'],
            paymentMethodTypes: [
                this.stripeInlineFormValues['payment_methods_mapping'][paymentMethodCode]
                ?? paymentMethodCode
            ],
        };
        if (this.paymentContext['mode'] === 'payment') {
            elementsOptions.mode = 'payment';
            elementsOptions.amount = parseInt(this.stripeInlineFormValues['minor_amount']);
            if (this.stripeInlineFormValues['is_tokenization_required']) {
                elementsOptions.setupFutureUsage = 'off_session';
            }
        }
        else {
            elementsOptions.mode = 'setup';
            elementsOptions.setupFutureUsage = 'off_session';
        }
        this.stripeElements[paymentOptionId] = this.stripeJS.elements(elementsOptions);

        // Instantiate the payment element.
        const paymentElementOptions = {
            defaultValues: {
                billingDetails: this.stripeInlineFormValues['billing_details'],
            },
        };
        const paymentElement = this.stripeElements[paymentOptionId].create(
            'payment', paymentElementOptions
        );
        paymentElement.on('loaderror', response => {
            this._displayErrorDialog(_t("Cannot display the payment form"), response.error.message);
        });
        paymentElement.mount(stripeInlineForm);

        const tokenizationCheckbox = inlineForm.querySelector(
            'input[name="o_payment_tokenize_checkbox"]'
        );
        if (tokenizationCheckbox) {
            // Display tokenization-specific inputs when the tokenization checkbox is checked.
            this.stripeElements[paymentOptionId].update({
                setupFutureUsage: tokenizationCheckbox.checked ? 'off_session' : null,
            }); // Force sync the states of the API and the checkbox in case they were inconsistent.
            tokenizationCheckbox.addEventListener('input', () => {
                this.stripeElements[paymentOptionId].update({
                    setupFutureUsage: tokenizationCheckbox.checked ? 'off_session' : null,
                });
            });
        }
    },

    // #=== PAYMENT FLOW ===#

    /**
     * Trigger the payment processing by submitting the elements.
     *
     * @override method from @payment/js/payment_form
     * @private
     * @param {string} providerCode - The code of the selected payment option's provider.
     * @param {number} paymentOptionId - The id of the selected payment option.
     * @param {string} paymentMethodCode - The code of the selected payment method, if any.
     * @param {string} flow - The payment flow of the selected payment option.
     * @return {void}
     */
    async _initiatePaymentFlow(providerCode, paymentOptionId, paymentMethodCode, flow) {
        if (providerCode !== 'stripe' || flow === 'token') {
            await this._super(...arguments); // Tokens are handled by the generic flow.
            return;
        }

        // Trigger form validation and wallet collection.
        const _super = this._super.bind(this);
        try {
            await this.stripeElements[paymentOptionId].submit();
        } catch (error) {
            this._displayErrorDialog(_t("Incorrect payment details"), error.message);
            this._enableButton();
            return
        }
        return await _super(...arguments);
    },

    /**
     * Process Stripe implementation of the direct payment flow.
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
        if (providerCode !== 'stripe') {
            await this._super(...arguments);
            return;
        }

        const { error } = await this._stripeConfirmIntent(processingValues, paymentOptionId);
        if (error) {
            this._displayErrorDialog(_t("Payment processing failed"), error.message);
            this._enableButton();
        }
    },

    /**
     * Confirm the intent on Stripe's side and handle any next action.
     *
     * @private
     * @param {object} processingValues - The processing values of the transaction.
     * @param {number} paymentOptionId - The id of the payment option handling the transaction.
     * @return {object} The processing error, if any.
     */
    async _stripeConfirmIntent(processingValues, paymentOptionId) {
        const confirmOptions = {
            elements: this.stripeElements[paymentOptionId],
            clientSecret: processingValues['client_secret'],
            confirmParams: {
                return_url: processingValues['return_url'],
            },
        };
        if (this.paymentContext['mode'] === 'payment'){
             return await this.stripeJS.confirmPayment(confirmOptions);
        }
        else {
            return await this.stripeJS.confirmSetup(confirmOptions);
        }
    },

});

```

## File: static\src\js\stripe_options.js

```javascript
/** @odoo-module */

export class StripeOptions {
    /**
     * Prepare the options to init the Stripe JS Object.
     *
     * This method serves as a hook for modules that would fully implement Stripe Connect.
     *
     * @param {object} processingValues
     * @return {object}
     */
    _prepareStripeOptions(processingValues) {
        return {
            'apiVersion': '2019-05-16',  // The API version of Stripe implemented in this module.
        };
    };
}

```

## File: views\payment_provider_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_provider_form" model="ir.ui.view">
        <field name="name">Stripe Provider Form</field>
        <field name="model">payment.provider</field>
        <field name="inherit_id" ref="payment.payment_provider_form"/>
        <field name="arch" type="xml">
            <group name="provider_credentials" position="before">
                <div
                     name="stripe_onboarding_group"
                     invisible="context.get('stripe_onboarding', False) or (code != 'stripe' or stripe_secret_key or stripe_publishable_key)">
                    <button string="Connect Stripe"
                            type="object"
                            name="action_stripe_connect_account"
                            class="btn-primary"
                            colspan="2"
                            invisible="state == 'enabled'"/>
                </div>
            </group>
            <group name="provider_credentials" position="inside">
                <group invisible="code != 'stripe'" name="stripe_credentials">
                    <field name="stripe_publishable_key" required="code == 'stripe' and state != 'disabled'"/>
                    <field name="stripe_secret_key" required="code == 'stripe' and state != 'disabled'" password="True"/>
                    <label for="stripe_webhook_secret"/>
                    <div class="o_row" col="2">
                        <field name="stripe_webhook_secret" password="True"/>
                        <button string="Generate your webhook"
                                type="object"
                                name="action_stripe_create_webhook"
                                class="btn-primary"
                                invisible="stripe_webhook_secret or not stripe_secret_key"/>
                    </div>
                </group>
                <div name="stripe_keys_link"
                     invisible="not context.get('stripe_onboarding', False) or (code != 'stripe' or stripe_secret_key and stripe_publishable_key)">
                    <a class="btn btn-link" role="button" href="https://dashboard.stripe.com/account/apikeys" target="_blank">
                        Get your Secret and Publishable keys
                    </a>
                </div>
            </group>
            <field name="allow_express_checkout" position="replace">
                <label for="allow_express_checkout" invisible="not support_express_checkout"/>
                <div class="o_row" col="2" invisible="not support_express_checkout">
                    <field name="allow_express_checkout"/>
                    <button string="Enable Apple Pay"
                            type="object"
                            name="action_stripe_verify_apple_pay_domain"
                            class="btn btn-primary"
                            invisible="not allow_express_checkout or code != 'stripe'"/>
                </div>
            </field>
        </field>
    </record>

    <record id="action_payment_provider_onboarding" model="ir.actions.act_window">
        <field name="name">Payment Providers</field>
        <field name="res_model">payment.provider</field>
        <field name="view_mode">form</field>
        <field name="context">{'stripe_onboarding': True}</field>
    </record>

</odoo>

```

## File: views\payment_stripe_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="inline_form">
        <t t-set="inline_form_values"
           t-value="provider_sudo._stripe_get_inline_form_values(
                        amount,
                        currency,
                        partner_id,
                        mode == 'validation',
                        payment_method_sudo=pm_sudo,
                        sale_order_id=sale_order_id,
                    )"
        />
        <div name="o_stripe_element_container"
             t-att-data-stripe-inline-form-values="inline_form_values"
        />
    </template>

    <template id="express_checkout_form">
        <div name="o_express_checkout_container"
             t-attf-id="o_stripe_express_checkout_container_{{provider_sudo.id}}"
             t-att-data-provider-id="provider_sudo.id"
             t-att-data-provider-code="provider_sudo.code"
             t-att-data-stripe-publishable-key="provider_sudo._stripe_get_publishable_key()"
             t-att-data-country-code="provider_sudo._stripe_get_country(provider_sudo.company_id.country_id.code)"
             class="w-100 mt-2"
        />
    </template>

</odoo>

```

## File: views\payment_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="payment_form" inherit_id="payment.form">
        <xpath expr="." position="inside">
            <t t-call="payment_stripe.sdk_assets"/>
        </xpath>
    </template>

    <template id="no_pms_available_warning" inherit_id="payment.no_pms_available_warning">
        <a name="activate_stripe" position="attributes">
            <!-- Hide 'Activate Stripe' button when the `payment_stripe` module is installed. -->
            <attribute name="class" separator=" " add="d-none"/>
        </a>
    </template>

    <template id="express_checkout" inherit_id="payment.express_checkout">
        <xpath expr="." position="inside">
            <t t-call="payment_stripe.sdk_assets"/>
        </xpath>
    </template>

    <template id="sdk_assets">
        <!-- As the following link does not end with '.js', it's not loaded when
             placed in __manifest__.py. The following declaration fix this problem -->
        <script type="text/javascript" src="https://js.stripe.com/v3/"></script>
    </template>
</odoo>

```

