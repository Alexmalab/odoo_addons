# Odoo Module: payment_stripe

Category: Accounting/Payment Acquirers

This file contains the source code of the Odoo module.

## File: const.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import namedtuple

API_VERSION = '2019-05-16'  # The API version of Stripe implemented in this module

# Stripe proxy URL
PROXY_URL = 'https://stripe.api.odoo.com/api/stripe/'

# Support payment method types
PMT = namedtuple('PaymentMethodType', ['name', 'countries', 'currencies', 'recurrence'])
PAYMENT_METHOD_TYPES = [
    PMT('card', [], [], 'recurring'),
    PMT('ideal', ['nl'], ['eur'], 'punctual'),
    PMT('bancontact', ['be'], ['eur'], 'punctual'),
    PMT('eps', ['at'], ['eur'], 'punctual'),
    PMT('giropay', ['de'], ['eur'], 'punctual'),
    PMT('p24', ['pl'], ['eur', 'pln'], 'punctual'),
]

# Mapping of transaction states to Stripe {Payment,Setup}Intent statuses.
# See https://stripe.com/docs/payments/intents#intent-statuses for the exhaustive list of status.
INTENT_STATUS_MAPPING = {
    'draft': ('requires_payment_method', 'requires_confirmation', 'requires_action'),
    'pending': ('processing',),
    'done': ('succeeded',),
    'cancel': ('canceled',),
}

# Events which are handled by the webhook
WEBHOOK_HANDLED_EVENTS = [
    'checkout.session.completed',
]

```

## File: utils.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

def get_publishable_key(acquirer_sudo):
    """ Return the publishable key for Stripe.

    Note: This method serves as a hook for modules that would fully implement Stripe Connect.

    :param recordset acquirer_sudo: The acquirer on which the key should be read, as a sudoed
                                    `payment.acquire` record.
    :return: The publishable key
    :rtype: str
    """
    return acquirer_sudo.stripe_publishable_key


def get_secret_key(acquirer_sudo):
    """ Return the secret key for Stripe.

    Note: This method serves as a hook for modules that would fully implement Stripe Connect.

    :param recordset acquirer_sudo: The acquirer on which the key should be read, as a sudoed
                                    `payment.acquire` record.
    :return: The secret key
    :rtype: str
    """
    return acquirer_sudo.stripe_secret_key


def get_webhook_secret(acquirer_sudo):
    """ Return the webhook secret for Stripe.

    Note: This method serves as a hook for modules that would fully implement Stripe Connect.

    :param recordset acquirer_sudo: The acquirer on which the key should be read, as a sudoed
                                    `payment.acquire` record.
    :returns: The webhook secret
    :rtype: str
    """
    return acquirer_sudo.stripe_webhook_secret

```

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

from odoo.addons.payment import reset_payment_acquirer


def uninstall_hook(cr, registry):
    reset_payment_acquirer(cr, registry, 'stripe')

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Stripe Payment Acquirer',
    'version': '2.0',
    'category': 'Accounting/Payment Acquirers',
    'sequence': 380,
    'summary': 'Payment Acquirer: Stripe Implementation',
    'description': " ",  # Non-empty string to avoid loading the README file.
    'depends': ['payment'],
    'data': [
        'views/payment_views.xml',
        'views/payment_templates.xml',
        'data/payment_acquirer_data.xml',
    ],
    'application': True,
    'uninstall_hook': 'uninstall_hook',
    'assets': {
        'web.assets_frontend': [
            'payment_stripe/static/src/js/payment_form.js',
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
import json
import logging
import pprint
from datetime import datetime

from odoo import http
from odoo.exceptions import ValidationError
from odoo.http import request
from odoo.tools import consteq

from odoo.addons.payment_stripe import utils as stripe_utils


_logger = logging.getLogger(__name__)


class StripeController(http.Controller):
    _checkout_return_url = '/payment/stripe/checkout_return'
    _validation_return_url = '/payment/stripe/validation_return'
    _webhook_url = '/payment/stripe/webhook'
    WEBHOOK_AGE_TOLERANCE = 10*60  # seconds

    @http.route(_checkout_return_url, type='http', auth='public', csrf=False)
    def stripe_return_from_checkout(self, **data):
        """ Process the data returned by Stripe after redirection for checkout.

        :param dict data: The GET params appended to the URL in `_stripe_create_checkout_session`
        """
        # Retrieve the tx and acquirer based on the tx reference included in the return url
        tx_sudo = request.env['payment.transaction'].sudo()._get_tx_from_feedback_data(
            'stripe', data
        )
        acquirer_sudo = tx_sudo.acquirer_id

        # Fetch the PaymentIntent, Charge and PaymentMethod objects from Stripe
        payment_intent = acquirer_sudo._stripe_make_request(
            f'payment_intents/{tx_sudo.stripe_payment_intent}', method='GET'
        )
        _logger.info("received payment_intents response:\n%s", pprint.pformat(payment_intent))
        self._include_payment_intent_in_feedback_data(payment_intent, data)

        # Handle the feedback data crafted with Stripe API objects
        request.env['payment.transaction'].sudo()._handle_feedback_data('stripe', data)

        # Redirect the user to the status page
        return request.redirect('/payment/status')

    @http.route(_validation_return_url, type='http', auth='public', csrf=False)
    def stripe_return_from_validation(self, **data):
        """ Process the data returned by Stripe after redirection for validation.

        :param dict data: The GET params appended to the URL in `_stripe_create_checkout_session`
        """
        # Retrieve the acquirer based on the tx reference included in the return url
        acquirer_sudo = request.env['payment.transaction'].sudo()._get_tx_from_feedback_data(
            'stripe', data
        ).acquirer_id

        # Fetch the Session, SetupIntent and PaymentMethod objects from Stripe
        checkout_session = acquirer_sudo._stripe_make_request(
            f'checkout/sessions/{data.get("checkout_session_id")}',
            payload={'expand[]': 'setup_intent.payment_method'},  # Expand all required objects
            method='GET'
        )
        _logger.info("received checkout/session response:\n%s", pprint.pformat(checkout_session))
        self._include_setup_intent_in_feedback_data(checkout_session.get('setup_intent', {}), data)

        # Handle the feedback data crafted with Stripe API objects
        request.env['payment.transaction'].sudo()._handle_feedback_data('stripe', data)

        # Redirect the user to the status page
        return request.redirect('/payment/status')

    @http.route(_webhook_url, type='json', auth='public')
    def stripe_webhook(self):
        """ Process the `checkout.session.completed` event sent by Stripe to the webhook.

        :return: An empty string to acknowledge the notification with an HTTP 200 response
        :rtype: str
        """
        event = json.loads(request.httprequest.data)
        _logger.info("event received:\n%s", pprint.pformat(event))
        try:
            if event['type'] == 'checkout.session.completed':
                checkout_session = event['data']['object']

                # Check the source and integrity of the event
                data = {'reference': checkout_session['client_reference_id']}
                tx_sudo = request.env['payment.transaction'].sudo()._get_tx_from_feedback_data(
                    'stripe', data
                )
                if self._verify_webhook_signature(
                    stripe_utils.get_webhook_secret(tx_sudo.acquirer_id)
                ):
                    # Fetch the PaymentIntent, Charge and PaymentMethod objects from Stripe
                    if checkout_session.get('payment_intent'):  # Can be None
                        payment_intent = tx_sudo.acquirer_id._stripe_make_request(
                            f'payment_intents/{tx_sudo.stripe_payment_intent}', method='GET'
                        )
                        _logger.info(
                            "received payment_intents response:\n%s", pprint.pformat(payment_intent)
                        )
                        self._include_payment_intent_in_feedback_data(payment_intent, data)
                    # Fetch the SetupIntent and PaymentMethod objects from Stripe
                    if checkout_session.get('setup_intent'):  # Can be None
                        setup_intent = tx_sudo.acquirer_id._stripe_make_request(
                            f'setup_intents/{checkout_session.get("setup_intent")}',
                            payload={'expand[]': 'payment_method'},
                            method='GET'
                        )
                        _logger.info(
                            "received setup_intents response:\n%s", pprint.pformat(setup_intent)
                        )
                        self._include_setup_intent_in_feedback_data(setup_intent, data)
                    # Handle the feedback data crafted with Stripe API objects as a regular feedback
                    request.env['payment.transaction'].sudo()._handle_feedback_data('stripe', data)
        except ValidationError:  # Acknowledge the notification to avoid getting spammed
            _logger.exception("unable to handle the event data; skipping to acknowledge")
        return ''

    @staticmethod
    def _include_payment_intent_in_feedback_data(payment_intent, data):
        data.update({'payment_intent': payment_intent})
        if payment_intent.get('charges', {}).get('total_count', 0) > 0:
            charge = payment_intent['charges']['data'][0]  # Use the latest charge object
            data.update({
                'charge': charge,
                'payment_method': charge.get('payment_method_details'),
            })

    @staticmethod
    def _include_setup_intent_in_feedback_data(setup_intent, data):
        data.update({
            'setup_intent': setup_intent,
            'payment_method': setup_intent.get('payment_method')
        })

    def _verify_webhook_signature(self, webhook_secret):
        """ Check that the signature computed from the feedback matches the received one.

        See https://stripe.com/docs/webhooks/signatures#verify-manually.

        :param str webhook_secret: The secret webhook key of the acquirer handling the transaction
        :return: Whether the signatures match
        :rtype: str
        """
        if not webhook_secret:
            _logger.warning("ignored webhook event due to undefined webhook secret")
            return False

        notification_payload = request.httprequest.data.decode('utf-8')
        signature_entries = request.httprequest.headers.get('Stripe-Signature').split(',')
        signature_data = {k: v for k, v in [entry.split('=') for entry in signature_entries]}

        # Check the timestamp of the event
        event_timestamp = int(signature_data['t'])
        if datetime.utcnow().timestamp() - event_timestamp > self.WEBHOOK_AGE_TOLERANCE:
            _logger.warning("ignored webhook event due to age tolerance: %s", event_timestamp)
            return False

        # Compare signatures
        received_signature = signature_data['v1']
        signed_payload = f'{event_timestamp}.{notification_payload}'
        expected_signature = hmac.new(
            webhook_secret.encode('utf-8'),
            signed_payload.encode('utf-8'),
            hashlib.sha256
        ).hexdigest()
        if not consteq(received_signature, expected_signature):
            _logger.warning("ignored event with invalid signature")
            return False

        return True

```

## File: controllers\onboarding.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from werkzeug.urls import url_encode

from odoo import http
from odoo.http import request


class OnboardingController(http.Controller):
    _onboarding_return_url = '/payment/stripe/onboarding/return'
    _onboarding_refresh_url = '/payment/stripe/onboarding/refresh'

    @http.route(_onboarding_return_url, type='http', methods=['GET'], auth='user')
    def stripe_return_from_onboarding(self, acquirer_id, menu_id):
        """ Redirect the user to the acquirer form of the onboarded Stripe account.

        The user is redirected to this route by Stripe after or during (if the user clicks on a
        dedicated button) the onboarding.

        :param str acquirer_id: The acquirer linked to the Stripe account being onboarded, as a
                                `payment.acquirer` id
        :param str menu_id: The menu from which the user started the onboarding step, as an
                            `ir.ui.menu` id
        """
        stripe_acquirer = request.env['payment.acquirer'].browse(int(acquirer_id))
        stripe_acquirer.company_id._mark_payment_onboarding_step_as_done()
        action = request.env.ref(
            'payment_stripe.action_payment_acquirer_onboarding', raise_if_not_found=False
        ) or request.env.ref('payment.action_payment_acquirer')
        get_params_string = url_encode({'action': action.id, 'id': acquirer_id, 'menu_id': menu_id})
        return request.redirect(f'/web?#{get_params_string}')

    @http.route(_onboarding_refresh_url, type='http', methods=['GET'], auth='user')
    def stripe_refresh_onboarding(self, acquirer_id, account_id, menu_id):
        """ Redirect the user to a new Stripe Connect onboarding link.

        The user is redirected to this route by Stripe if the onboarding link they used was expired.

        :param str acquirer_id: The acquirer linked to the Stripe account being onboarded, as a
                                `payment.acquirer` id
        :param str account_id: The id of the connected account
        :param str menu_id: The menu from which the user started the onboarding step, as an
                            `ir.ui.menu` id
        """
        stripe_acquirer = request.env['payment.acquirer'].browse(int(acquirer_id))
        account_link = stripe_acquirer._stripe_create_account_link(account_id, int(menu_id))
        return request.redirect(account_link, local=False)

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main
from . import onboarding

```

## File: data\payment_acquirer_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="payment.payment_acquirer_stripe" model="payment.acquirer">
        <field name="provider">stripe</field>
        <field name="support_authorization">False</field>
        <field name="support_fees_computation">False</field>
        <field name="support_refund"></field>
        <field name="support_tokenization">True</field>
        <field name="allow_tokenization">True</field>
    </record>

    <record id="payment_method_stripe" model="account.payment.method">
        <field name="name">Stripe</field>
        <field name="code">stripe</field>
        <field name="payment_type">inbound</field>
    </record>

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
        res['stripe'] = {'mode': 'electronic', 'domain': [('type', '=', 'bank')]}
        return res

```

## File: models\payment_acquirer.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import uuid

import requests
from werkzeug.urls import url_encode, url_join

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError

from odoo.addons.payment_stripe import utils as stripe_utils
from odoo.addons.payment_stripe import const
from odoo.addons.payment_stripe.controllers.main import StripeController
from odoo.addons.payment_stripe.controllers.onboarding import OnboardingController


_logger = logging.getLogger(__name__)


class PaymentAcquirer(models.Model):
    _inherit = 'payment.acquirer'

    provider = fields.Selection(
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

    #=== CONSTRAINT METHODS ===#

    @api.constrains('state', 'stripe_publishable_key', 'stripe_secret_key')
    def _check_state_of_connected_account_is_never_test(self):
        """ Check that the acquirer of a connected account can never been set to 'test'.

        This constraint is defined in the present module to allow the export of the translation
        string of the `ValidationError` should it be raised by modules that would fully implement
        Stripe Connect.

        Additionally, the field `state` is used as a trigger for this constraint to allow those
        modules to indirectly trigger it when writing on custom fields. Indeed, by always writing on
        `state` together with writing on those custom fields, the constraint would be triggered.

        :return: None
        :raise ValidationError: If the acquirer of a connected account is set in state 'test'.
        """
        for acquirer in self:
            if acquirer.state == 'test' and acquirer._stripe_has_connected_account():
                raise ValidationError(_(
                    "You cannot set the acquirer to Test Mode while it is linked with your Stripe "
                    "account."
                ))

    def _stripe_has_connected_account(self):
        """ Return whether the acquirer is linked to a connected Stripe account.

        Note: This method serves as a hook for modules that would fully implement Stripe Connect.
        Note: self.ensure_one()

        :return: Whether the acquirer is linked to a connected Stripe account
        :rtype: bool
        """
        self.ensure_one()
        return False

    @api.constrains('state')
    def _check_onboarding_of_enabled_provider_is_completed(self):
        """ Check that the acquirer cannot be set to 'enabled' if the onboarding is ongoing.

        This constraint is defined in the present module to allow the export of the translation
        string of the `ValidationError` should it be raised by modules that would fully implement
        Stripe Connect.

        :return: None
        :raise ValidationError: If the acquirer of a connected account is set in state 'enabled'
                                while the onboarding is not finished.
        """
        for acquirer in self:
            if acquirer.state == 'enabled' and acquirer._stripe_onboarding_is_ongoing():
                raise ValidationError(_(
                    "You cannot set the acquirer state to Enabled until your onboarding to Stripe "
                    "is completed."
                ))

    def _stripe_onboarding_is_ongoing(self):
        """ Return whether the acquirer is linked to an ongoing onboarding to Stripe Connect.

        Note: This method serves as a hook for modules that would fully implement Stripe Connect.
        Note: self.ensure_one()

        :return: Whether the acquirer is linked to an ongoing onboarding to Stripe Connect
        :rtype: bool
        """
        self.ensure_one()
        return False

    # === ACTION METHODS === #

    def action_stripe_connect_account(self, menu_id=None):
        """ Create a Stripe Connect account and redirect the user to the next onboarding step.

        If the acquirer is already enabled, close the current window. Otherwise, generate a Stripe
        Connect onboarding link and redirect the user to it. If provided, the menu id is included in
        the URL the user is redirected to when coming back on Odoo after the onboarding. If the link
        generation failed, redirect the user to the acquirer form.

        Note: This method serves as a hook for modules that would fully implement Stripe Connect.
        Note: self.ensure_one()

        :param int menu_id: The menu from which the user started the onboarding step, as an
                            `ir.ui.menu` id.
        :return: The next step action
        :rtype: dict
        """
        self.ensure_one()

        if self.state == 'enabled':
            self.company_id._mark_payment_onboarding_step_as_done()
            action = {'type': 'ir.actions.act_window_close'}
        else:
            # Account creation
            connected_account = self._stripe_fetch_or_create_connected_account()

            # Link generation
            menu_id = menu_id or self.env.ref('payment.payment_acquirer_menu').id
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
                    'model': 'payment.acquirer',
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
                    'enabled_events[]': const.WEBHOOK_HANDLED_EVENTS,
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
            # If the request originates from an offline operation, don't raise and return the resp.
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

    def _get_default_payment_method_id(self):
        self.ensure_one()
        if self.provider != 'stripe':
            return super()._get_default_payment_method_id()
        return self.env.ref('payment_stripe.payment_method_stripe').id

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
            'country': self.company_id.country_id.code,
            'email': self.company_id.email,
            'business_type': 'individual',
            'company[address][city]': self.company_id.city or '',
            'company[address][country]': self.company_id.country_id.code or '',
            'company[address][line1]': self.company_id.street or '',
            'company[address][line2]': self.company_id.street2 or '',
            'company[address][postal_code]': self.company_id.zip or '',
            'company[address][state]': self.company_id.state_id.name or '',
            'company[name]': self.company_id.name,
            'individual[address][city]': self.company_id.city or '',
            'individual[address][country]': self.company_id.country_id.code or '',
            'individual[address][line1]': self.company_id.street or '',
            'individual[address][line2]': self.company_id.street2 or '',
            'individual[address][postal_code]': self.company_id.zip or '',
            'individual[address][state]': self.company_id.state_id.name or '',
            'individual[email]': self.company_id.email or '',
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
        return_params = dict(acquirer_id=self.id, menu_id=menu_id)
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
            _logger.error("request forwarded with error: %s", error_data['message'])
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
        response_content = self.acquirer_id._stripe_make_request(
            'payment_methods',
            payload={
                'customer': self.acquirer_ref,
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

from werkzeug import urls

from odoo import _, api, fields, models
from odoo.exceptions import UserError, ValidationError

from odoo.addons.payment import utils as payment_utils
from odoo.addons.payment_stripe import utils as stripe_utils
from odoo.addons.payment_stripe.const import INTENT_STATUS_MAPPING, PAYMENT_METHOD_TYPES
from odoo.addons.payment_stripe.controllers.main import StripeController


_logger = logging.getLogger(__name__)


class PaymentTransaction(models.Model):
    _inherit = 'payment.transaction'

    stripe_payment_intent = fields.Char(string="Stripe Payment Intent ID", readonly=True)

    def _get_specific_processing_values(self, processing_values):
        """ Override of payment to return Stripe-specific processing values.

        Note: self.ensure_one() from `_get_processing_values`

        :param dict processing_values: The generic processing values of the transaction
        :return: The dict of acquirer-specific processing values
        :rtype: dict
        """
        res = super()._get_specific_processing_values(processing_values)
        if self.provider != 'stripe' or self.operation == 'online_token':
            return res

        checkout_session = self._stripe_create_checkout_session()
        return {
            'publishable_key': stripe_utils.get_publishable_key(self.acquirer_id),
            'session_id': checkout_session['id'],
        }

    def _stripe_create_checkout_session(self):
        """ Create and return a Checkout Session.

        :return: The Checkout Session
        :rtype: dict
        """
        # Filter payment method types by available payment method
        existing_pms = [pm.name.lower() for pm in self.env['payment.icon'].search([])]
        linked_pms = [pm.name.lower() for pm in self.acquirer_id.payment_icon_ids]
        pm_filtered_pmts = filter(
            lambda pmt: pmt.name == 'card'
            # If the PM (payment.icon) record related to a PMT doesn't exist, don't filter out the
            # PMT because the user couldn't even have linked it to the acquirer in the first place.
            or (pmt.name in linked_pms or pmt.name not in existing_pms),
            PAYMENT_METHOD_TYPES
        )
        # Filter payment method types by country code
        country_code = self.partner_country_id and self.partner_country_id.code.lower()
        country_filtered_pmts = filter(
            lambda pmt: not pmt.countries or country_code in pmt.countries, pm_filtered_pmts
        )
        # Filter payment method types by currency name
        currency_name = self.currency_id.name.lower()
        currency_filtered_pmts = filter(
            lambda pmt: not pmt.currencies or currency_name in pmt.currencies, country_filtered_pmts
        )
        # Filter payment method types by recurrence if the transaction must be tokenized
        if self.tokenize:
            recurrence_filtered_pmts = filter(
                lambda pmt: pmt.recurrence == 'recurring', currency_filtered_pmts
            )
        else:
            recurrence_filtered_pmts = currency_filtered_pmts
        # Build the session values related to payment method types
        pmt_values = {}
        for pmt_id, pmt_name in enumerate(map(lambda pmt: pmt.name, recurrence_filtered_pmts)):
            pmt_values[f'payment_method_types[{pmt_id}]'] = pmt_name

        # Create the session according to the operation and return it
        customer = self._stripe_create_customer()
        common_session_values = self._get_common_stripe_session_values(pmt_values, customer)
        base_url = self.acquirer_id.get_base_url()
        if self.operation == 'online_redirect':
            return_url = f'{urls.url_join(base_url, StripeController._checkout_return_url)}' \
                         f'?reference={urls.url_quote_plus(self.reference)}'
            # Specify a future usage for the payment intent to:
            # 1. attach the payment method to the created customer
            # 2. trigger a 3DS check if one if required, while the customer is still present
            future_usage = 'off_session' if self.tokenize else None
            checkout_session = self.acquirer_id._stripe_make_request(
                'checkout/sessions', payload={
                    **common_session_values,
                    'mode': 'payment',
                    'success_url': return_url,
                    'cancel_url': return_url,
                    'line_items[0][price_data][currency]': self.currency_id.name,
                    'line_items[0][price_data][product_data][name]': self.reference,
                    'line_items[0][price_data][unit_amount]': payment_utils.to_minor_currency_units(
                        self.amount, self.currency_id
                    ),
                    'line_items[0][quantity]': 1,
                    'payment_intent_data[description]': self.reference,
                    'payment_intent_data[setup_future_usage]': future_usage,
                }
            )
            self.stripe_payment_intent = checkout_session['payment_intent']
        else:  # 'validation'
            # {CHECKOUT_SESSION_ID} is a template filled by Stripe when the Session is created
            return_url = f'{urls.url_join(base_url, StripeController._validation_return_url)}' \
                         f'?reference={urls.url_quote_plus(self.reference)}' \
                         f'&checkout_session_id={{CHECKOUT_SESSION_ID}}'
            checkout_session = self.acquirer_id._stripe_make_request(
                'checkout/sessions', payload={
                    **common_session_values,
                    'mode': 'setup',
                    'success_url': return_url,
                    'cancel_url': return_url,
                }
            )
        return checkout_session

    def _stripe_create_customer(self):
        """ Create and return a Customer.

        :return: The Customer
        :rtype: dict
        """
        customer = self.acquirer_id._stripe_make_request(
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

    def _get_common_stripe_session_values(self, pmt_values, customer):
        """ Return the Stripe Session values that are common to redirection and validation.

        Note: This method serves as a hook for modules that would fully implement Stripe Connect.

        :param dict pmt_values: The payment method types values
        :param dict customer: The Stripe customer to assign to the session
        :return: The common Stripe Session values
        :rtype: dict
        """
        return {
            **pmt_values,
            'client_reference_id': self.reference,
            # Assign a customer to the session so that Stripe automatically attaches the payment
            # method to it in a validation flow. In checkout flow, a customer is automatically
            # created if not provided but we still do it here to avoid requiring the customer to
            # enter his email on the checkout page.
            'customer': customer['id'],
        }

    def _send_payment_request(self):
        """ Override of payment to send a payment request to Stripe with a confirmed PaymentIntent.

        Note: self.ensure_one()

        :return: None
        :raise: UserError if the transaction is not linked to a token
        """
        super()._send_payment_request()
        if self.provider != 'stripe':
            return

        # Make the payment request to Stripe
        if not self.token_id:
            raise UserError("Stripe: " + _("The transaction is not linked to a token."))

        payment_intent = self._stripe_create_payment_intent()
        feedback_data = {'reference': self.reference}
        StripeController._include_payment_intent_in_feedback_data(payment_intent, feedback_data)
        _logger.info("entering _handle_feedback_data with data:\n%s", pprint.pformat(feedback_data))
        self._handle_feedback_data('stripe', feedback_data)

    def _stripe_create_payment_intent(self):
        """ Create and return a PaymentIntent.

        Note: self.ensure_one()

        :return: The Payment Intent
        :rtype: dict
        """
        if not self.token_id.stripe_payment_method:  # Pre-SCA token -> migrate it
            self.token_id._stripe_sca_migrate_customer()

        response = self.acquirer_id._stripe_make_request(
            'payment_intents',
            payload=self._stripe_prepare_payment_intent_payload(),
            offline=self.operation == 'offline',
            idempotency_key=payment_utils.generate_idempotency_key(
                self, scope='payment_intents_token'
            ) if self.operation == 'offline' else None,
        )  # Make the request idempotent to prevent multiple payments (e.g., rollback mechanism).
        if 'error' not in response:
            payment_intent = response
        else:  # A processing error was returned in place of the payment intent
            error_msg = response['error'].get('message')
            self._set_error("Stripe: " + _(
                "The communication with the API failed.\n"
                "Stripe gave us the following info about the problem:\n'%s'", error_msg
            ))  # Flag transaction as in error now as the intent status might have a valid value
            payment_intent = response['error'].get('payment_intent')  # Get the PI from the error

        return payment_intent

    def _stripe_prepare_payment_intent_payload(self):
        """ Prepare the payload for the creation of a payment intent in Stripe format.

        Note: This method serves as a hook for modules that would fully implement Stripe Connect.
        Note: self.ensure_one()

        :return: The Stripe-formatted payload for the payment intent request
        :rtype: dict
        """
        return {
            'amount': payment_utils.to_minor_currency_units(self.amount, self.currency_id),
            'currency': self.currency_id.name.lower(),
            'confirm': True,
            'customer': self.token_id.acquirer_ref,
            'off_session': True,
            'payment_method': self.token_id.stripe_payment_method,
            'description': self.reference,
        }

    @api.model
    def _get_tx_from_feedback_data(self, provider, data):
        """ Override of payment to find the transaction based on Stripe data.

        :param str provider: The provider of the acquirer that handled the transaction
        :param dict data: The feedback data sent by the provider
        :return: The transaction if found
        :rtype: recordset of `payment.transaction`
        :raise: ValidationError if inconsistent data were received
        :raise: ValidationError if the data match no transaction
        """
        tx = super()._get_tx_from_feedback_data(provider, data)
        if provider != 'stripe':
            return tx

        reference = data.get('reference')
        if not reference:
            raise ValidationError("Stripe: " + _("Received data with missing merchant reference"))

        tx = self.search([('reference', '=', reference), ('provider', '=', 'stripe')])
        if not tx:
            raise ValidationError(
                "Stripe: " + _("No transaction found matching reference %s.", reference)
            )
        return tx

    def _process_feedback_data(self, data):
        """ Override of payment to process the transaction based on Adyen data.

        Note: self.ensure_one()

        :param dict data: The feedback data build from information passed to the return route.
                          Depending on the operation of the transaction, the entries with the keys
                          'payment_intent', 'charge', 'setup_intent' and 'payment_method' can be
                          populated with their corresponding Stripe API objects.
        :return: None
        :raise: ValidationError if inconsistent data were received
        """
        super()._process_feedback_data(data)
        if self.provider != 'stripe':
            return

        if 'charge' in data:
            self.acquirer_reference = data['charge']['id']

        # Handle the intent status
        if self.operation == 'validation':
            intent_status = data.get('setup_intent', {}).get('status')
        else:  # 'online_redirect', 'online_token', 'offline'
            intent_status = data.get('payment_intent', {}).get('status')
        if not intent_status:
            raise ValidationError(
                "Stripe: " + _("Received data with missing intent status.")
            )

        if intent_status in INTENT_STATUS_MAPPING['draft']:
            pass
        elif intent_status in INTENT_STATUS_MAPPING['pending']:
            self._set_pending()
        elif intent_status in INTENT_STATUS_MAPPING['done']:
            if self.tokenize:
                self._stripe_tokenize_from_feedback_data(data)
            self._set_done()
        elif intent_status in INTENT_STATUS_MAPPING['cancel']:
            self._set_canceled()
        else:  # Classify unknown intent statuses as `error` tx state
            _logger.warning("received data with invalid intent status: %s", intent_status)
            self._set_error(
                "Stripe: " + _("Received data with invalid intent status: %s", intent_status)
            )

    def _stripe_tokenize_from_feedback_data(self, data):
        """ Create a new token based on the feedback data.

        :param dict data: The feedback data built with Stripe objects. See `_process_feedback_data`.
        :return: None
        """
        if self.operation == 'online_redirect':
            payment_method_id = data.get('charge', {}).get('payment_method')
            customer_id = data.get('charge', {}).get('customer')
        else:  # 'validation'
            payment_method_id = data.get('setup_intent', {}).get('payment_method', {}).get('id')
            customer_id = data.get('setup_intent', {}).get('customer')
        payment_method = data.get('payment_method')
        if not payment_method_id or not payment_method:
            _logger.warning("requested tokenization with payment method missing from feedback data")
            return

        if payment_method.get('type') != 'card':
            # Only 'card' payment methods can be tokenized. This case should normally not happen as
            # non-recurring payment methods are not shown to the customer if the "Save my payment
            # details checkbox" is shown. Still, better be on the safe side..
            _logger.warning("requested tokenization of non-recurring payment method")
            return

        token = self.env['payment.token'].create({
            'acquirer_id': self.acquirer_id.id,
            'name': payment_utils.build_token_name(payment_method['card'].get('last4')),
            'partner_id': self.partner_id.id,
            'acquirer_ref': customer_id,
            'verified': True,
            'stripe_payment_method': payment_method_id,
        })
        self.write({
            'token_id': token,
            'tokenize': False,
        })
        _logger.info(
            "created token with id %s for partner with id %s", token.id, self.partner_id.id
        )

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_payment_method
from . import payment_acquirer
from . import payment_token
from . import payment_transaction

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#94B6C8"/><stop offset="100%" stop-color="#6A9EBA"/></linearGradient><path id="d" d="M19.25 46h2.714v1.212c0 .188.152.343.339.343h26.394a.342.342 0 0 0 .339-.343V37.5H32.5c.333-3.333-.165-5.5-1.494-6.5h18.03v-3.212a.342.342 0 0 0-.339-.343H29.765c.361-1.01.57-1.924.627-2.742h18.644c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742V46zm-3.79 9.642h-1.037V55h2.833v.642h-1.037v2.832h-.76v-2.832zM17.422 55h1.07l.809 2.389h.01L20.075 55h1.07v3.474h-.711v-2.462h-.01l-.847 2.462h-.586l-.848-2.438h-.01v2.438h-.711V55zm5.66-27.992c4.256 1.523 6.896 3.327 6.896 7.649 0 2.612-.901 4.633-2.64 6.001-1.553 1.244-3.852 1.897-6.616 1.897-3.48 0-6.834-1.057-8.635-2.084l.932-5.814c2.112 1.244 5.342 2.207 7.299 2.207 1.584 0 2.454-.59 2.454-1.616 0-1.058-.901-1.742-3.603-2.706-4.193-1.523-6.771-3.327-6.771-7.556 0-2.332.838-4.26 2.453-5.597 1.553-1.275 3.728-1.959 6.337-1.959 3.696 0 6.367 1.027 7.671 1.649l-.931 5.752c-1.647-.808-4.038-1.71-6.368-1.71-1.273 0-1.988.497-1.988 1.368 0 1.026 1.243 1.68 3.51 2.519z"/><path id="e" d="M19.25 44h2.714v1.212c0 .188.152.343.339.343h26.394a.342.342 0 0 0 .339-.343V35.5H32.5c.333-3.333-.165-5.5-1.494-6.5h18.03v-3.212a.342.342 0 0 0-.339-.343H29.765c.361-1.01.57-1.924.627-2.742h18.644c1.5 0 2.714 1.228 2.714 2.742v20.11c0 1.514-1.213 2.742-2.714 2.742H21.964c-1.5 0-2.714-1.228-2.714-2.742V44zm-3.79 9.642h-1.037V53h2.833v.642h-1.037v2.832h-.76v-2.832zM17.422 53h1.07l.809 2.389h.01L20.075 53h1.07v3.474h-.711v-2.462h-.01l-.847 2.462h-.586l-.848-2.438h-.01v2.438h-.711V53zm5.66-27.992c4.256 1.523 6.896 3.327 6.896 7.649 0 2.612-.901 4.633-2.64 6.001-1.553 1.244-3.852 1.897-6.616 1.897-3.48 0-6.834-1.057-8.635-2.084l.932-5.814c2.112 1.244 5.342 2.207 7.299 2.207 1.584 0 2.454-.59 2.454-1.616 0-1.058-.901-1.742-3.603-2.706-4.193-1.523-6.771-3.327-6.771-7.556 0-2.332.838-4.26 2.453-5.597 1.553-1.275 3.728-1.959 6.337-1.959 3.696 0 6.367 1.027 7.671 1.649l-.931 5.752c-1.647-.808-4.038-1.71-6.368-1.71-1.273 0-1.988.497-1.988 1.368 0 1.026 1.243 1.68 3.51 2.519z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M4 69c-2 0-4-1-4-4V33.916L14 17.5l13.818 5.203h9.432L48 23l3.576 1.968-.236 22.01L39.224 69H4z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: static\src\js\payment_form.js

```javascript
/* global Stripe */
odoo.define('payment_stripe.payment_form', require => {
    'use strict';

    const checkoutForm = require('payment.checkout_form');
    const manageForm = require('payment.manage_form');

    const stripeMixin = {

        /**
         * Redirect the customer to Stripe hosted payment page.
         *
         * @override method from payment.payment_form_mixin
         * @private
         * @param {string} provider - The provider of the payment option's acquirer
         * @param {number} paymentOptionId - The id of the payment option handling the transaction
         * @param {object} processingValues - The processing values of the transaction
         * @return {undefined}
         */
        _processRedirectPayment: function (provider, paymentOptionId, processingValues) {
            if (provider !== 'stripe') {
                return this._super(...arguments);
            }

            const stripeJS = Stripe(processingValues['publishable_key'],
                this._prepareStripeOptions(processingValues));
            stripeJS.redirectToCheckout({
                sessionId: processingValues['session_id']
            });
        },

        /**
         * Prepare the options to init the Stripe JS Object
         *
         * Function overriden in internal module
         *
         * @param {object} processingValues
         * @return {object}
         */
        _prepareStripeOptions: function (processingValues) {
            return {
                'apiVersion': '2019-05-16',  // The API version of Stripe implemented in this module
            };
        },
    };

    checkoutForm.include(stripeMixin);
    manageForm.include(stripeMixin);

});

```

## File: views\payment_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="checkout" inherit_id="payment.checkout">
        <xpath expr="." position="inside">
            <!-- As the following link does not end with '.js', it's not loaded when
                 placed in __manifest__.py. The following declaration fix this problem -->
            <script type="text/javascript" src="https://js.stripe.com/v3/"></script>
        </xpath>
    </template>

    <template id="manage" inherit_id="payment.manage">
        <xpath expr="." position="inside">
            <!-- As the following link does not end with '.js', it's not loaded when
                 placed in __manifest__.py. The following declaration fix this problem -->
            <script type="text/javascript" src="https://js.stripe.com/v3/"></script>
        </xpath>
    </template>

</odoo>

```

## File: views\payment_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_acquirer_form" model="ir.ui.view">
        <field name="name">Stripe Acquirer Form</field>
        <field name="model">payment.acquirer</field>
        <field name="inherit_id" ref="payment.payment_acquirer_form"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='acquirer']" position="before">
                <group invisible="context.get('stripe_onboarding', False)"
                       name="stripe_onboarding_group"
                       attrs="{'invisible': ['|', '|', ('provider', '!=', 'stripe'), ('stripe_secret_key', '!=', False), ('stripe_publishable_key', '!=', False)]}">
                    <button string="Connect Stripe"
                            type="object"
                            name="action_stripe_connect_account"
                            class="btn-primary"
                            attrs="{'invisible': [('state', '=', 'enabled')]}"/>
                </group>
            </xpath>
            <xpath expr="//group[@name='acquirer']" position="inside">
                <group attrs="{'invisible': [('provider', '!=', 'stripe')]}" name="stripe_credentials">
                    <field name="stripe_publishable_key" attrs="{'required':[('provider', '=', 'stripe'), ('state', '!=', 'disabled')]}" password="True"/>
                    <field name="stripe_secret_key" attrs="{'required':[('provider', '=', 'stripe'), ('state', '!=', 'disabled')]}" password="True"/>
                    <label for="stripe_webhook_secret"/>
                    <div class="o_row" col="2">
                        <field name="stripe_webhook_secret" password="True"/>
                        <button string="Generate your webhook"
                                type="object"
                                name="action_stripe_create_webhook"
                                class="btn-primary"
                                attrs="{'invisible': ['|', ('stripe_webhook_secret', '!=', False), ('stripe_secret_key', '=', False)]}"/>
                    </div>
                </group>
                <div name="stripe_keys_link"
                     invisible="not context.get('stripe_onboarding', False)"
                     attrs="{'invisible': ['|', ('provider', '!=', 'stripe'), '&amp;', ('stripe_secret_key', '!=', False), ('stripe_publishable_key', '!=', False)]}">
                    <a class="btn btn-link" role="button" href="https://dashboard.stripe.com/account/apikeys" target="_blank">
                        Get your Secret and Publishable keys
                    </a>
                </div>
            </xpath>
        </field>
    </record>

    <record id="action_payment_acquirer_onboarding" model="ir.actions.act_window">
        <field name="name">Payment Acquirers</field>
        <field name="res_model">payment.acquirer</field>
        <field name="view_mode">form</field>
        <field name="context">{'stripe_onboarding': True, 'form_view_initial_mode': 'edit'}</field>
    </record>

</odoo>

```

