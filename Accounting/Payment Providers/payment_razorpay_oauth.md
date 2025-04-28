# Odoo Module: payment_razorpay_oauth

Category: Accounting/Payment Providers

This file contains the source code of the Odoo module.

## File: const.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

OAUTH_URL = 'https://razorpay.api.odoo.com/api/razorpay/1'

```

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': "Razorpay OAuth Integration",
    'category': 'Accounting/Payment Providers',
    'sequence': 350,
    'summary': "Easy Razorpay Onboarding With Oauth.",
    'icon': '/payment_razorpay/static/description/icon.png',
    'depends': ['payment_razorpay'],
    'data': [
        'views/payment_provider_views.xml',
        'views/razorpay_templates.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: controllers\onboarding.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import pprint
from datetime import timedelta
from urllib.parse import urlencode

from werkzeug.exceptions import Forbidden

from odoo import _, fields
from odoo.exceptions import ValidationError
from odoo.http import Controller, request, route


_logger = logging.getLogger(__name__)


class RazorpayController(Controller):

    OAUTH_RETURN_URL = '/payment/razorpay/oauth/return'

    @route(OAUTH_RETURN_URL, type='http', auth='user', methods=['GET'], website=True)
    def razorpay_return_from_authorization(self, **data):
        """ Exchange the authorization code for an access token and redirect to the provider form.

        :param dict data: The authorization code received from Razorpay, in addition to the provided
                          provider id and CSRF token that were sent back by the proxy.
        :raise Forbidden: If the received CSRF token cannot be verified.
        :raise ValidationError: If the provider id does not match any Razorpay provider.
        :return: Redirect to the payment provider form.
        """
        _logger.info("Returning from authorization with data:\n%s", pprint.pformat(data))

        # Retrieve the Razorpay data and Odoo metadata from the redirect data.
        provider_id = int(data['provider_id'])
        authorization_code = data.get('authorization_code')
        csrf_token = data['csrf_token']
        provider_sudo = request.env['payment.provider'].sudo().browse(provider_id).exists()
        if not provider_sudo or provider_sudo.code != 'razorpay':
            raise ValidationError(_("Could not find Razorpay provider with id %s", provider_sudo))

        # Verify the CSRF token.
        if not request.validate_csrf(csrf_token):
            _logger.warning("CSRF token verification failed.")
            raise Forbidden()

        # Request and set the OAuth tokens on the provider.
        action = request.env.ref('payment.action_payment_provider')
        redirect_url = f'/odoo/action-{action.id}/{int(provider_sudo.id)}'
        if not authorization_code: # The user cancelled the authorization.
            return request.redirect(redirect_url)
        try:
            response_content = provider_sudo._razorpay_make_proxy_request(
                '/get_access_token', payload={'authorization_code': authorization_code}
            )
        except ValidationError as e:
            return request.render(
                'payment_razorpay_oauth.authorization_error',
                qcontext={'error_message': str(e), 'provider_url': redirect_url},
            )
        expires_in = fields.Datetime.now() + timedelta(seconds=int(response_content['expires_in']))
        provider_sudo.write({
            # Reset the classical API key fields.
            'razorpay_key_id': None,
            'razorpay_key_secret': None,
            'razorpay_webhook_secret': None,
            # Set the new OAuth fields.
            'razorpay_account_id': response_content['razorpay_account_id'],
            'razorpay_public_token': response_content['public_token'],
            'razorpay_refresh_token': response_content['refresh_token'],
            'razorpay_access_token': response_content['access_token'],
            'razorpay_access_token_expiry': expires_in,
            # Enable the provider.
            'state': 'enabled',
            'is_published': True,
        })
        try:
            provider_sudo.action_razorpay_create_webhook()
        except ValidationError as error:
            _logger.warning(error)
        return request.redirect(redirect_url)

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import onboarding

```

## File: data\neutralize.sql

```sql
-- disable razorpay payment provider
UPDATE payment_provider
   SET razorpay_public_token = NULL,
       razorpay_refresh_token = NULL,
       razorpay_access_token = NULL,
       razorpay_access_token_expiry = NULL,
       razorpay_account_id = NULL;

```

## File: models\payment_provider.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import pprint
import uuid
from datetime import timedelta
from urllib.parse import urlencode

import requests

from odoo import _, fields, models
from odoo.exceptions import RedirectWarning, ValidationError
from odoo.http import request

from odoo.addons.payment_razorpay import const
from odoo.addons.payment_razorpay_oauth import const as oauth_const
from odoo.addons.payment_razorpay_oauth.controllers.onboarding import RazorpayController


_logger = logging.getLogger(__name__)


class PaymentProvider(models.Model):
    _inherit = 'payment.provider'

    razorpay_key_id = fields.Char(required_if_provider=False)
    razorpay_key_secret = fields.Char(required_if_provider=False)
    razorpay_webhook_secret = fields.Char(required_if_provider=False)

    # OAuth fields
    razorpay_account_id = fields.Char(string="Razorpay Account ID", groups='base.group_system')
    razorpay_refresh_token = fields.Char(
        string="Razorpay Refresh Token", groups='base.group_system'
    )
    razorpay_public_token = fields.Char(string="Razorpay Public Token", groups='base.group_system')
    razorpay_access_token = fields.Char(string="Razorpay Access Token", groups='base.group_system')
    razorpay_access_token_expiry = fields.Datetime(
        string="Razorpay Access Token Expiry", groups='base.group_system'
    )

    # === ACTION METHODS ===#

    def action_razorpay_redirect_to_oauth_url(self):
        """ Redirect to the Razorpay OAuth URL.

        Note: `self.ensure_one()`

        :return: An URL action to redirect to the Razorpay OAuth URL.
        :rtype: dict
        """
        self.ensure_one()

        if self.company_id.currency_id.name not in const.SUPPORTED_CURRENCIES:
            raise RedirectWarning(
                _(
                    "Razorpay is not available in your country; please use another payment"
                    " provider."
                ),
                self.env.ref('payment.action_payment_provider').id,
                _("Other Payment Providers"),
            )

        params = {
            'return_url': f'{self.get_base_url()}{RazorpayController.OAUTH_RETURN_URL}',
            'provider_id': self.id,
            'csrf_token': request.csrf_token(),
        }
        authorization_url = f'{oauth_const.OAUTH_URL}/authorize?{urlencode(params)}'
        return {
            'type': 'ir.actions.act_url',
            'url': authorization_url,
            'target': 'self',
        }

    def action_razorpay_reset_oauth_account(self):
        """ Reset the Razorpay OAuth account.

        Note: self.ensure_one()

        :return: None
        """
        self.ensure_one()

        return self.write({
            'razorpay_account_id': None,
            'razorpay_public_token': None,
            'razorpay_refresh_token': None,
            'razorpay_access_token': None,
            'razorpay_access_token_expiry': None,
            'state': 'disabled',
            'is_published': False,
        })

    def action_razorpay_create_webhook(self):
        """ Create a webhook and display a toast notification.

        Note: `self.ensure_one()`

        :return: The feedback notification.
        :rtype: dict
        """
        self.ensure_one()

        webhook_secret = uuid.uuid4().hex  # Generate a random webhook secret.
        payload = {
            'url': f'{self.get_base_url()}/payment/razorpay/webhook',
            'alert_email': self.env.user.partner_id.email,
            'secret': webhook_secret,
            'events': const.HANDLED_WEBHOOK_EVENTS,
        }
        _logger.info(
            "Sending '/accounts/%(account_id)s/webhooks' request:\n%(payload)s",
            {'account_id': self.razorpay_account_id, 'payload': pprint.pformat(payload)},
        )
        webhook_data = self.with_context(razorpay_api_version='v2')._razorpay_make_request(
            f'accounts/{self.razorpay_account_id}/webhooks', payload=payload
        )
        _logger.info(
            "Response of '/accounts/%(account_id)s/webhooks' request:\n%(response)s",
            {'account_id': self.razorpay_account_id, 'response': pprint.pformat(webhook_data)},
        )
        self.razorpay_webhook_secret = webhook_secret

        return {
            'type': 'ir.actions.client',
            'tag': 'display_notification',
            'params': {
                'type': 'success',
                'message': _("Your Razorpay webhook was successfully set up!"),
                'next': {'type': 'ir.actions.client', 'tag': 'soft_reload'},
            },
        }

    # === BUSINESS METHODS - OAUTH === #

    def _razorpay_make_proxy_request(self, endpoint, payload=None):
        """ Make a request to the Razorpay proxy at the specified endpoint.

        :param str endpoint: The proxy endpoint to be reached by the request; prefixed with '/'.
        :param dict payload: The payload of the request.
        :return The JSON-formatted content of the response.
        :rtype: dict
        :raise ValidationError: If an HTTP error occurs.
        """
        proxy_payload = {
            'jsonrpc': '2.0',
            'id': uuid.uuid4().hex,
            'method': 'call',
            'params': payload,
        }
        url = f'{oauth_const.OAUTH_URL}{endpoint}'
        try:
            response = requests.post(url, json=proxy_payload, timeout=10)
            response.raise_for_status()
        except (requests.exceptions.ConnectionError, requests.exceptions.Timeout):
            _logger.exception("Unable to reach endpoint at %s", url)
            raise ValidationError("Razorpay Proxy: " + _("Could not establish the connection."))
        except requests.exceptions.HTTPError:
            _logger.exception(
                "Invalid API request at %s with data %s", url, pprint.pformat(payload)
            )
            raise ValidationError(
                "Razorpay Proxy: " + _("An error occurred when communicating with the proxy.")
            )

        # Razorpay proxy endpoints always respond with HTTP 200 as they implement JSON-RPC 2.0.
        response_content = response.json()
        if response_content.get('error'):  # An exception was raised on the proxy side.
            error_message = response_content['error']['data']['message']
            _logger.exception("Request forwarded with error: %s", error_message)
            raise ValidationError(f"Razorpay Proxy: {error_message}")

        return response_content['result']

    def _razorpay_get_public_token(self):
        self.ensure_one()

        return self.razorpay_public_token

    def _razorpay_get_access_token(self):
        self.ensure_one()

        if self.razorpay_access_token and self.razorpay_access_token_expiry < fields.Datetime.now():
            self._razorpay_refresh_access_token()
        return self.razorpay_access_token

    def _razorpay_refresh_access_token(self):
        """ Refresh the access token.

        Note: `self.ensure_one()`

        :return: dict
        """
        self.ensure_one()

        response_content = self._razorpay_make_proxy_request(
            '/refresh_access_token', payload={'refresh_token': self.razorpay_refresh_token}
        )
        if response_content.get('access_token'):
            expiry = fields.Datetime.now() + timedelta(seconds=int(response_content['expires_in']))
            self.write({
                'razorpay_public_token': response_content['public_token'],
                'razorpay_refresh_token': response_content['refresh_token'],
                'razorpay_access_token': response_content['access_token'],
                'razorpay_access_token_expiry': expiry,
            })

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import payment_provider

```

## File: views\payment_provider_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_provider_form_razorpay_oauth" model="ir.ui.view">
        <field name="name">Razorpay Provider Oauth Form</field>
        <field name="model">payment.provider</field>
        <field name="inherit_id" ref="payment_razorpay.payment_provider_form_razorpay"/>
        <field name="arch" type="xml">
            <group name="provider_credentials" position="before">
                <div class="text-muted mb-2" invisible="not razorpay_account_id">
                    This provider is linked with your Razorpay account.
                </div>
            </group>
            <field name="razorpay_key_id" position="before">
                <label string="Account ID" for="razorpay_account_id" groups="base.group_no_one"/>
                <div class="o_row" groups="base.group_no_one">
                    <field name="razorpay_account_id" readonly="True"/>
                    <button
                        string="Reset Your Razorpay Account"
                        type="object"
                        name="action_razorpay_reset_oauth_account"
                        class="btn-secondary ms-2"
                        confirm="Are you sure you want to disconnect?"
                        invisible="not razorpay_account_id"
                        readonly="True"
                    />
                </div>
            </field>
            <field name="razorpay_key_id" position="attributes">
                <attribute name="required">False</attribute>
                <attribute name="decoration-muted">razorpay_account_id</attribute>
                <attribute name="groups">base.group_no_one</attribute>
            </field>
            <field name="razorpay_key_secret" position="attributes">
                <attribute name="required">razorpay_key_id</attribute>
                <attribute name="decoration-muted">razorpay_account_id</attribute>
                <attribute name="groups">base.group_no_one</attribute>
            </field>
            <field name="razorpay_webhook_secret" position="replace">
                <label for="razorpay_webhook_secret" groups="base.group_no_one"/>
                <div class="o_row" groups="base.group_no_one">
                    <field
                        string="Webhook Secret"
                        name="razorpay_webhook_secret"
                        password="True"
                    />
                    <button
                        string="Generate your webhook"
                        type="object"
                        name="action_razorpay_create_webhook"
                        class="btn-primary ms-2"
                        invisible="not razorpay_account_id or razorpay_webhook_secret"
                    />
                </div>
            </field>
            <group name="provider_credentials" position="after">
                <div
                    class="alert alert-warning"
                    role="alert"
                    invisible="code != 'razorpay' or not razorpay_key_id or razorpay_account_id"
                >
                    You are currently connected to Razorpay through the credentials method, which is
                    deprecated. Click the "Connect" button below to use the recommended OAuth
                    method.
                </div>
                <div invisible="code != 'razorpay' or razorpay_account_id">
                    <button
                        string="Connect"
                        type="object"
                        name="action_razorpay_redirect_to_oauth_url"
                        class="btn-primary"
                    />
                </div>
            </group>
        </field>
    </record>

</odoo>

```

## File: views\razorpay_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="authorization_error" name="Authorization Error">
        <!-- Variables description:
            - 'error_message' - The reason of the error.
            - 'provider_url' - The URL to the Razorpay provider.
        -->
        <t t-call="portal.frontend_layout">
            <div class="wrap">
                <div class="container">
                    <h1>An error occurred</h1>
                    <p>An error occurred while linking your Razorpay account with Odoo.</p>
                    <p><t t-out="error_message"/></p>
                    <a t-att-href="provider_url" class="btn btn-primary mt-2">
                        Back to the Razorpay provider
                    </a>
                </div>
            </div>
        </t>
    </template>

</odoo>

```

