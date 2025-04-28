# Odoo Module: payment_xendit

Category: Accounting/Payment Providers

This file contains the source code of the Odoo module.

## File: const.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# The currencies supported by Xendit, in ISO 4217 format.
SUPPORTED_CURRENCIES = [
    'IDR',
    'PHP',
]

# To correctly allow lowest decimal place rounding
# https://docs.xendit.co/payment-link/payment-channels
CURRENCY_DECIMALS = {
    'IDR': 0,
    'PHP': 0,
}

# The codes of the payment methods to activate when Xendit is activated.
DEFAULT_PAYMENT_METHOD_CODES = {
    # Primary payment methods.
    'card',
    'dana',
    'ovo',
    'qris',

    # Brand payment methods.
    'visa',
    'mastercard',
}

# Mapping of payment code to channel code according to Xendit API
PAYMENT_METHODS_MAPPING = {
    'bank_bca': 'BCA',
    'bank_permata': 'PERMATA',
    'bpi': 'DD_BPI',
    'card': 'CREDIT_CARD',
    'maya': 'PAYMAYA',
}

# Mapping of transaction states to Xendit payment statuses.
PAYMENT_STATUS_MAPPING = {
    'draft': (),
    'pending': ('PENDING'),
    'done': ('SUCCEEDED', 'PAID', 'CAPTURED'),
    'cancel': ('CANCELLED', 'EXPIRED'),
    'error': ('FAILED',)
}

```

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

from odoo.addons.payment import setup_provider, reset_payment_provider


def post_init_hook(env):
    setup_provider(env, 'xendit')


def uninstall_hook(env):
    reset_payment_provider(env, 'xendit')

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': "Payment Provider: Xendit",
    'version': '1.0',
    'category': 'Accounting/Payment Providers',
    'sequence': 350,
    'summary': "A payment provider for Indonesian and the Philippines.",
    'description': " ",  # Non-empty string to avoid loading the README file.
    'depends': ['payment'],
    'data': [
        'views/payment_provider_views.xml',
        'views/payment_xendit_templates.xml',

        'data/payment_provider_data.xml',  # Depends on payment_xendit_templates.xml
    ],
    'post_init_hook': 'post_init_hook',
    'uninstall_hook': 'uninstall_hook',
    'assets': {
        'web.assets_frontend': [
            'payment_xendit/static/src/**/*',
        ]
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import pprint

from werkzeug.exceptions import Forbidden

from odoo import http
from odoo.exceptions import ValidationError
from odoo.http import request
from odoo.tools import consteq, str2bool

from odoo.addons.payment import utils as payment_utils


_logger = logging.getLogger(__name__)


class XenditController(http.Controller):

    _webhook_url = '/payment/xendit/webhook'
    _return_url = '/payment/xendit/return'

    @http.route('/payment/xendit/payment', type='json', auth='public')
    def xendit_payment(self, reference, token_ref):
        """ Make a payment by token request and handle the response.

        :param str reference: The reference of the transaction.
        :param str token_ref: The reference of the Xendit token to use to make the payment.
        :return: None
        """
        tx_sudo = request.env['payment.transaction'].sudo().search([('reference', '=', reference)])
        tx_sudo._xendit_create_charge(token_ref)

    @http.route(_webhook_url, type='http', methods=['POST'], auth='public', csrf=False)
    def xendit_webhook(self):
        """ Process the notification data sent by Xendit to the webhook.

        :return: The 'accepted' string to acknowledge the notification.
        """
        data = request.get_json_data()
        _logger.info("Notification received from Xendit with data:\n%s", pprint.pformat(data))

        try:
            # Check the integrity of the notification.
            received_token = request.httprequest.headers.get('x-callback-token')
            tx_sudo = request.env['payment.transaction'].sudo()._get_tx_from_notification_data(
                'xendit', data
            )
            self._verify_notification_token(received_token, tx_sudo)

            # Handle the notification data.
            tx_sudo._handle_notification_data('xendit', data)
        except ValidationError:
            _logger.exception("Unable to handle notification data; skipping to acknowledge.")

        return request.make_json_response(['accepted'], status=200)

    @http.route(_return_url, type='http', methods=['GET'], auth='public')
    def xendit_return(self, tx_ref=None, success=False, access_token=None, **data):
        """Set draft transaction to pending after successfully returning from Xendit."""
        if access_token and str2bool(success, default=False):
            tx_sudo = request.env['payment.transaction'].sudo().search([
                ('provider_code', '=', 'xendit'),
                ('reference', '=', tx_ref),
                ('state', '=', 'draft'),
            ], limit=1)
            if tx_sudo and payment_utils.check_access_token(access_token, tx_ref, tx_sudo.amount):
                tx_sudo._set_pending()
        return request.redirect('/payment/status')

    def _verify_notification_token(self, received_token, tx_sudo):
        """ Check that the received token matches the saved webhook token.

        :param str received_token: The callback token received with the notification data.
        :param payment.transaction tx_sudo: The transaction referenced by the notification data.
        :return: None
        :raise Forbidden: If the tokens don't match.
        """
        # Check for the received token.
        if not received_token:
            _logger.warning("Received notification with missing token.")
            raise Forbidden()

        if not consteq(tx_sudo.provider_id.xendit_webhook_token, received_token):
            _logger.warning("Received notification with invalid callback token %r.", received_token)
            raise Forbidden()

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\neutralize.sql

```sql
UPDATE payment_provider
   SET xendit_secret_key = 'dummysecret',
       xendit_webhook_token = 'dummytoken';

```

## File: data\payment_provider_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="payment.payment_provider_xendit" model="payment.provider">
        <field name="code">xendit</field>
        <field name="redirect_form_view_id" ref="redirect_form"/>
        <field name="inline_form_view_id" ref="inline_form"/>
        <field name="allow_tokenization">True</field>
    </record>

</odoo>

```

## File: models\payment_provider.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import pprint

import requests

from odoo import _, fields, models
from odoo.exceptions import ValidationError

from odoo.addons.payment_xendit import const


_logger = logging.getLogger(__name__)


class PaymentProvider(models.Model):
    _inherit = 'payment.provider'

    code = fields.Selection(
        selection_add=[('xendit', "Xendit")], ondelete={'xendit': 'set default'}
    )
    xendit_public_key = fields.Char(
        string="Xendit Public Key", groups='base.group_system', required_if_provider='xendit'
    )
    xendit_secret_key = fields.Char(
        string="Xendit Secret Key", groups='base.group_system', required_if_provider='xendit'
    )
    xendit_webhook_token = fields.Char(
        string="Xendit Webhook Token", groups='base.group_system', required_if_provider='xendit'
    )

    # === COMPUTE METHODS === #

    def _compute_feature_support_fields(self):
        """ Override of `payment` to enable additional features. """
        super()._compute_feature_support_fields()
        self.filtered(lambda p: p.code == 'xendit').support_tokenization = True

    # === BUSINESS METHODS - PAYMENT FLOW ===#

    def _get_supported_currencies(self):
        """ Override of `payment` to return the supported currencies. """
        supported_currencies = super()._get_supported_currencies()
        if self.code == 'xendit':
            supported_currencies = supported_currencies.filtered(
                lambda c: c.name in const.SUPPORTED_CURRENCIES
            )
        return supported_currencies

    def _get_default_payment_method_codes(self):
        """ Override of `payment` to return the default payment method codes. """
        default_codes = super()._get_default_payment_method_codes()
        if self.code != 'xendit':
            return default_codes
        return const.DEFAULT_PAYMENT_METHOD_CODES

    def _xendit_make_request(self, endpoint, payload=None):
        """ Make a request to Xendit API and return the JSON-formatted content of the response.

        Note: self.ensure_one()

        :param str endpoint: The endpoint to be reached by the request.
        :param dict payload: The payload of the request.
        :return The JSON-formatted content of the response.
        :rtype: dict
        :raise ValidationError: If an HTTP error occurs.
        """
        self.ensure_one()

        url = f'https://api.xendit.co/{endpoint}'
        auth = (self.xendit_secret_key, '')
        try:
            response = requests.post(url, json=payload, auth=auth, timeout=10)
            response.raise_for_status()
        except (requests.exceptions.ConnectionError, requests.exceptions.Timeout):
            _logger.exception("Unable to reach endpoint at %s", url)
            raise ValidationError("Xendit: " + _("Could not establish the connection to the API."))
        except requests.exceptions.HTTPError as err:
            error_message = err.response.json().get('message')
            _logger.exception(
                "Invalid API request at %s with data:\n%s", url, pprint.pformat(payload)
            )
            raise ValidationError(
                "Xendit: " + _(
                    "The communication with the API failed. Xendit gave us the following"
                    " information: '%s'", error_message
                )
            )
        return response.json()

    # === BUSINESS METHODS - GETTERS === #

    def _get_redirect_form_view(self, is_validation=False):
        """ Override of `payment` to avoid rendering the form view for validation operations.

        Unlike other compatible payment methods in Xendit, `Card` is implemented using a direct
        flow. To avoid rendering a useless template, and also to avoid computing wrong values, this
        method returns `None` for Xendit's validation operations (Card is and will always be the
        sole tokenizable payment method for Xendit).

        Note: `self.ensure_one()`

        :param bool is_validation: Whether the operation is a validation.
        :return: The view of the redirect form template or None.
        :rtype: ir.ui.view | None
        """
        self.ensure_one()

        if self.code == 'xendit' and is_validation:
            return None
        return super()._get_redirect_form_view(is_validation)

```

## File: models\payment_transaction.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import pprint

from werkzeug import urls

from odoo import _, models
from odoo.exceptions import ValidationError
from odoo.tools import float_round

from odoo.addons.payment import utils as payment_utils
from odoo.addons.payment_xendit import const
from odoo.addons.payment_xendit.controllers.main import XenditController


_logger = logging.getLogger(__name__)


class PaymentTransaction(models.Model):
    _inherit = 'payment.transaction'

    def _get_specific_processing_values(self, processing_values):
        """ Override of payment to return Xendit-specific processing values.

        Note: self.ensure_one() from `_get_processing_values`

        :param dict processing_values: The generic processing values of the transaction
        :return: The dict of provider-specific processing values
        :rtype: dict
        """
        res = super()._get_specific_processing_values(processing_values)
        if self.provider_code != 'xendit':
            return res

        if self.currency_id.name in const.CURRENCY_DECIMALS:
            rounding = const.CURRENCY_DECIMALS.get(self.currency_id.name)
        else:
            rounding = self.currency_id.decimal_places
        rounded_amount = float_round(self.amount, rounding, rounding_method='DOWN')
        return {
            'rounded_amount': rounded_amount
        }

    def _get_specific_rendering_values(self, processing_values):
        """ Override of `payment` to return Xendit-specific rendering values.

        Note: self.ensure_one() from `_get_processing_values`

        :param dict processing_values: The generic and specific processing values of the transaction
        :return: The dict of provider-specific processing values.
        :rtype: dict
        """
        res = super()._get_specific_rendering_values(processing_values)
        if self.provider_code != 'xendit' or self.payment_method_code == 'card':
            return res

        # Initiate the payment and retrieve the invoice data.
        payload = self._xendit_prepare_invoice_request_payload()
        _logger.info("Sending invoice request for link creation:\n%s", pprint.pformat(payload))
        invoice_data = self.provider_id._xendit_make_request('v2/invoices', payload=payload)
        _logger.info("Received invoice request response:\n%s", pprint.pformat(invoice_data))

        # Extract the payment link URL and embed it in the redirect form.
        rendering_values = {
            'api_url': invoice_data.get('invoice_url')
        }
        return rendering_values

    def _xendit_prepare_invoice_request_payload(self):
        """ Create the payload for the invoice request based on the transaction values.

        :return: The request payload.
        :rtype: dict
        """
        base_url = self.provider_id.get_base_url()
        redirect_url = urls.url_join(base_url, XenditController._return_url)
        access_token = payment_utils.generate_access_token(self.reference, self.amount)
        success_url_params = urls.url_encode({
            'tx_ref': self.reference,
            'access_token': access_token,
            'success': 'true',
        })
        payload = {
            'external_id': self.reference,
            'amount': self.amount,
            'description': self.reference,
            'customer': {
                'given_names': self.partner_name,
            },
            'success_redirect_url': f'{redirect_url}?{success_url_params}',
            'failure_redirect_url': redirect_url,
            'payment_methods': [const.PAYMENT_METHODS_MAPPING.get(
                self.payment_method_code, self.payment_method_code.upper())
            ],
            'currency': self.currency_id.name,
        }
        # Extra payload values that must not be included if empty.
        if self.partner_email:
            payload['customer']['email'] = self.partner_email
        if phone := self.partner_id.mobile or self.partner_id.phone:
            payload['customer']['mobile_number'] = phone
        address_details = {}
        if self.partner_city:
            address_details['city'] = self.partner_city
        if self.partner_country_id.name:
            address_details['country'] = self.partner_country_id.name
        if self.partner_zip:
            address_details['postal_code'] = self.partner_zip
        if self.partner_state_id.name:
            address_details['state'] = self.partner_state_id.name
        if self.partner_address:
            address_details['street_line1'] = self.partner_address
        if address_details:
            payload['customer']['addresses'] = [address_details]

        return payload

    def _send_payment_request(self):
        """ Override of `payment` to send a payment request to Xendit.

        Note: self.ensure_one()

        :return: None
        :raise UserError: If the transaction is not linked to a token.
        """
        super()._send_payment_request()
        if self.provider_code != 'xendit':
            return

        if not self.token_id:
            raise ValidationError("Xendit: " + _("The transaction is not linked to a token."))

        self._xendit_create_charge(self.token_id.provider_ref)

    def _xendit_create_charge(self, token_ref):
        """ Create a charge on Xendit using the `credit_card_charges` endpoint.

        :param str token_ref: The reference of the Xendit token to use to make the payment.
        :return: None
        """
        if self.currency_id.name in const.CURRENCY_DECIMALS:
            rounding = const.CURRENCY_DECIMALS.get(self.currency_id.name)
        else:
            rounding = self.currency_id.decimal_places
        rounded_amount = float_round(self.amount, rounding, rounding_method='DOWN')
        payload = {
            'token_id': token_ref,
            'external_id': self.reference,
            'amount': rounded_amount,
            'currency': self.currency_id.name,
        }
        charge_notification_data = self.provider_id._xendit_make_request(
            'credit_card_charges', payload=payload
        )
        self._handle_notification_data('xendit', charge_notification_data)

    def _get_tx_from_notification_data(self, provider_code, notification_data):
        """ Override of `payment` to find the transaction based on the notification data.

        :param str provider_code: The code of the provider that handled the transaction.
        :param dict notification_data: The notification data sent by the provider.
        :return: The transaction if found.
        :rtype: payment.transaction
        :raise ValidationError: If inconsistent data were received.
        :raise ValidationError: If the data match no transaction.
        """
        tx = super()._get_tx_from_notification_data(provider_code, notification_data)
        if provider_code != 'xendit' or len(tx) == 1:
            return tx

        reference = notification_data.get('external_id')
        if not reference:
            raise ValidationError("Xendit: " + _("Received data with missing reference."))

        tx = self.search([('reference', '=', reference), ('provider_code', '=', 'xendit')])
        if not tx:
            raise ValidationError(
                "Xendit: " + _("No transaction found matching reference %s.", reference)
            )
        return tx

    def _process_notification_data(self, notification_data):
        """ Override of `payment` to process the transaction based on Xendit data.

        Note: self.ensure_one()

        :param dict notification_data: The notification data sent by the provider.
        :return: None
        :raise ValidationError: If inconsistent data were received.
        """
        self.ensure_one()

        super()._process_notification_data(notification_data)
        if self.provider_code != 'xendit':
            return

        # Update the provider reference.
        self.provider_reference = notification_data.get('id')

        # Update payment method.
        payment_method_code = notification_data.get('payment_method', '')
        payment_method = self.env['payment.method']._get_from_code(
            payment_method_code, mapping=const.PAYMENT_METHODS_MAPPING
        )
        self.payment_method_id = payment_method or self.payment_method_id

        # Update the payment state.
        payment_status = notification_data.get('status')
        if payment_status in const.PAYMENT_STATUS_MAPPING['pending']:
            self._set_pending()
        elif payment_status in const.PAYMENT_STATUS_MAPPING['done']:
            if self.tokenize:
                self._xendit_tokenize_from_notification_data(notification_data)
            self._set_done()
        elif payment_status in const.PAYMENT_STATUS_MAPPING['cancel']:
            self._set_canceled()
        elif payment_status in const.PAYMENT_STATUS_MAPPING['error']:
            failure_reason = notification_data.get('failure_reason')
            self._set_error(_(
                "An error occurred during the processing of your payment (%s). Please try again.",
                failure_reason,
            ))

    def _xendit_tokenize_from_notification_data(self, notification_data):
        """ Create a new token based on the notification data.

        :param dict notification_data: Xendit's response to a charge API request.
        :return: None
        """
        card_info = notification_data['masked_card_number'][-4:]  # Xendit pads details with X's.
        token_id = notification_data['credit_card_token_id']
        token = self.env['payment.token'].create({
            "provider_id": self.provider_id.id,
            "payment_method_id": self.payment_method_id.id,
            "payment_details": card_info,
            "partner_id": self.partner_id.id,
            "provider_ref": token_id,
        })
        self.write({
            'token_id': token.id,
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
from . import payment_transaction

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M26.936 5H15.642L4 25.012 15.642 45l2.296-3.929-9.346-16.06 9.346-16.06h6.68L26.936 5Z" fill="#446CB3"/><path d="m18.982 18.579 2.296-3.929-3.34-5.698-2.296 3.952 3.34 5.675Zm13.08-9.627 2.296 3.952-16.42 28.168-2.296-3.952 16.42-28.168Z" fill="#D24D57"/><path d="m23.087 45 2.273-3.952h6.702l9.346-16.037-9.346-16.06L34.358 5 46 25.012 34.358 45H23.087Z" fill="#446CB3"/><path d="m30.972 31.352-2.296 3.93 3.386 5.766 2.296-3.883-3.386-5.813Z" fill="#D24D57"/></svg>

```

## File: static\src\js\auth_service.js

```javascript
import { registry } from '@web/core/registry';
import { AuthUI } from './auth_ui';
import { EventBus } from '@odoo/owl';

const bus = new EventBus();

export const authService = {
    start(env) {
        registry.category('main_components').add('AuthUI', { Component: AuthUI, props: { bus } });
    }
}
registry.category('services').add('auth_ui', authService);

```

## File: static\src\js\auth_ui.js

```javascript
import { EventBus, Component, xml } from '@odoo/owl';

export class AuthUI extends Component {
    static props = {
        bus: EventBus,
    }

    // Has to be in front of the block UI layer (which is z-index: 1070).
    static template = xml`
        <div
            id="three-ds-container"
            style="width: 500px;
            height: 600px;
            line-height: 200px;
            position: fixed;
            top: 25%;
            left: 40%;
            display: none;
            margin-top: -100px;
            margin-left: -150px;
            background-color: #ffffff;
            border-radius: 5px;mon
            text-align: center;
            z-index: 1072 !important;"
        >
            <iframe height="600" width="450" id="authorization-form" name="authorization-form"/>
        </div>
    `;
}

```

## File: static\src\js\payment_form.js

```javascript
/* global Xendit */

import { loadJS } from '@web/core/assets'
import { _t } from '@web/core/l10n/translation';
import { rpc, RPCError } from '@web/core/network/rpc';

import paymentForm from '@payment/js/payment_form';

paymentForm.include({
    xenditData: undefined,

    // #=== DOM MANIPULATION ===#

    /**
     * Prepare the inline form of Xendit for direct payment.
     *
     * @override method from @payment/js/payment_form
     * @private
     * @param {number} providerId - The id of the selected payment option's provider.
     * @param {string} providerCode - The code of the selected payment option's provider.
     * @param {number} paymentOptionId - The id of the selected payment option.
     * @param {string} paymentMethodCode - The code of the selected payment method, if any.
     * @param {string} flow - The online payment flow of the selected payment option.
     * @return {void}
     */
    async _prepareInlineForm(providerId, providerCode, paymentOptionId, paymentMethodCode, flow) {
        if (providerCode !== 'xendit' || paymentMethodCode !== 'card') {
            this._super(...arguments);
            return;
        }

        // Check if instantiation of the inline form is needed.
        this.xenditData ??= {}; // Store the form and public key of each instantiated Card method.
        if (flow === 'token') {
            return; // No inline form for tokens.
        } else if (this.xenditData[paymentOptionId]) {
            this._setPaymentFlow('direct'); // Overwrite the flow even if no re-instantiation.
            return; // Don't re-extract the data if already done for this payment method.
        }

        // Overwrite the flow of the selected payment method.
        this._setPaymentFlow('direct');

        // Extract and store the public key.
        const radio = document.querySelector('input[name="o_payment_radio"]:checked');
        const inlineForm = this._getInlineForm(radio);
        const xenditInlineForm = inlineForm.querySelector('[name="o_xendit_form"]');
        this.xenditData[paymentOptionId] = {
            publicKey: xenditInlineForm.dataset['xenditPublicKey'],
            inlineForm: xenditInlineForm,
        };

        // Load the SDK.
        await loadJS('https://js.xendit.co/v1/xendit.min.js');
        Xendit.setPublishableKey(this.xenditData[paymentOptionId].publicKey)
    },

    // #=== PAYMENT FLOW ===#

    /**
     * Validate the form inputs before initiating the payment flow.
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
        if (providerCode !== 'xendit' || flow === 'token' || paymentMethodCode !== 'card') {
            // Tokens are handled by the generic flow and other payment methods have no inline form.
            await this._super(...arguments);
            return;
        }

        const formInputs = this._xenditGetInlineFormInputs(paymentOptionId)
        const details = this._xenditGetPaymentDetails(paymentOptionId)

        // Set custom validity messages on inputs based on Xendit's feedback.
        Object.keys(formInputs).forEach(el => formInputs[el].setCustomValidity(""))
        if (!Xendit.card.validateCardNumber(details.card_number)){
            formInputs['card'].setCustomValidity(_t("Invalid Card Number"))
        }
        if (!Xendit.card.validateExpiry(details.card_exp_month, details.card_exp_year)) {
            formInputs['month'].setCustomValidity(_t("Invalid Date"))
            formInputs['year'].setCustomValidity(_t("Invalid Date"))
        }
        if (!Xendit.card.validateCvn(details.card_cvn)){
            formInputs['cvn'].setCustomValidity(_t("Invalid CVN"))
        }

        // Ensure that all inputs are valid.
        if (!Object.values(formInputs).every(element => element.reportValidity())) {
            this._enableButton(); // The submit button is disabled at this point, enable it.
            return;
        }
        await this._super(...arguments);
    },

    /**
     * Process the direct payment flow by creating a token and proceeding with a token payment flow.
     *
     * @override method from @payment/js/payment_form
     * @private
     * @param {string} providerCode - The code of the selected payment option's provider.
     * @param {number} paymentOptionId - The id of the selected payment option.
     * @param {string} paymentMethodCode - The code of the selected payment method, if any.
     * @param {object} processingValues - The processing values of the transaction.
     * @return {void}
     */
    async _processDirectFlow(providerCode, paymentOptionId, paymentMethodCode, processingValues) {
        if (providerCode !== 'xendit') {
            this._super(...arguments);
            return;
        }

        Xendit.card.createToken(
            {
                ...this._xenditGetPaymentDetails(paymentOptionId),
                // Allow reusing tokens when the transaction should be tokenized.
                is_multiple_use: processingValues['should_tokenize'],
                amount: processingValues['rounded_amount'],
            },
            (err, token) => this._xenditHandleResponse(err, token, processingValues),
        );
    },

    /**
     * Handle the token creation response and initiate the token payment.
     *
     * @private
     * @param {object} err - The error with the cause.
     * @param {object} token - The created token's data.
     * @param {object} processingValues - The processing values of the transaction.
     * @return {void}
     */
    _xenditHandleResponse(err, token, processingValues) {
        if (err) {
            let errMessage = err.message;

            if (err.error_code === 'API_VALIDATION_ERROR') {  // Invalid user input
                errMessage = err.errors[0].message // Wrong field format
            }
            this._displayErrorDialog(_t("Payment processing failed"), errMessage);
            this._enableButton();
            return;
        }
        if (token.status === 'VERIFIED') {
            rpc('/payment/xendit/payment', {
                'reference': processingValues.reference,
                'partner_id': processingValues.partner_id,
                'token_ref': token.id,
            }).then(() => {
                window.location = '/payment/status'
            }).catch(error => {
                if (error instanceof RPCError) {
                    this._displayErrorDialog(_t("Payment processing failed"), error.data.message);
                    this._enableButton();
                } else {
                    return Promise.reject(error);
                }
            })
        } else if (token.status === 'FAILED') {
            this._displayErrorDialog(_t("Payment processing failed"), token.failure_reason);
            document.querySelector('#three-ds-container').style.display = 'none';
            this._enableButton();
        } else if (token.status === 'IN_REVIEW') {
            document.querySelector('#three-ds-container').style.display = 'block';
            window.open(token.payer_authentication_url, 'authorization-form');
        }
    },

    // #=== GETTERS ===#

    /**
     * Return all relevant inline form inputs of the provided payment option.
     *
     * @private
     * @param {number} paymentOptionId - The id of the selected payment option.
     * @return {Object} - An object mapping the name of inline form inputs to their DOM element.
     */
    _xenditGetInlineFormInputs(paymentOptionId) {
        const form = this.xenditData[paymentOptionId]['inlineForm'];
        return {
            card: form.querySelector('#o_xendit_card'),
            month: form.querySelector('#o_xendit_month'),
            year: form.querySelector('#o_xendit_year'),
            cvn: form.querySelector('#o_xendit_cvn'),
            first_name: form.querySelector('#o_xendit_first_name'),
            last_name: form.querySelector('#o_xendit_last_name'),
            phone: form.querySelector('#o_xendit_phone'),
            email: form.querySelector('#o_xendit_email'),
        };
    },

    /**
     * Return the credit card data to prepare the payload for the create token request.
     *
     * @private
     * @param {number} paymentOptionId - The id of the selected payment option.
     * @return {Object} - Data to pass to the Xendit createToken request.
     */
    _xenditGetPaymentDetails(paymentOptionId) {
        const inputs = this._xenditGetInlineFormInputs(paymentOptionId);
        return {
            card_number: inputs.card.value.replace(/ /g, ''),
            card_exp_month: inputs.month.value,
            card_exp_year: inputs.year.value,
            card_cvn: inputs.cvn.value,
            card_holder_email: inputs.email.value,
            card_holder_first_name: inputs.first_name.value,
            card_holder_last_name: inputs.last_name.value,
            card_holder_phone_number: inputs.phone.value 
        };
    },

})

```

## File: views\payment_provider_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_provider_form_xendit" model="ir.ui.view">
        <field name="name">Xendit Provider Form</field>
        <field name="model">payment.provider</field>
        <field name="inherit_id" ref="payment.payment_provider_form"/>
        <field name="arch" type="xml">
            <group name="provider_credentials" position="inside">
                <group invisible="code != 'xendit'">
                    <field
                        name="xendit_public_key"
                        string="Public Key"
                        required="code == 'xendit' and state != 'disabled'"
                    />
                    <field
                        name="xendit_secret_key"
                        string="Secret Key"
                        required="code == 'xendit' and state != 'disabled'"
                        password="True"
                    />
                    <field
                        name="xendit_webhook_token"
                        string="Webhook Token"
                        required="code == 'xendit' and state != 'disabled'"
                        password="True"
                    />
                </group>
            </group>
        </field>
    </record>

</odoo>

```

## File: views\payment_xendit_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="redirect_form">
        <form t-att-action="api_url" method="get"/>
    </template>

    <template id="inline_form">
        <div name="o_xendit_form" t-att-data-xendit-public-key="provider_sudo.xendit_public_key">
            <t t-if="pm_sudo.code == 'card'">
                <!-- Name -->
                <div class="row">
                    <div class="col-sm-6 mb-3">
                        <label for="o_xendit_first_name">Card Holder First Name</label>
                            <input
                                id="o_xendit_first_name"
                                type="text"
                                placeholder="John"
                                required=""
                                class="form-control"
                            />
                    </div>
                    <div class="col-sm-6 mb-3">
                        <label for="o_xendit_last_name">Card Holder Last Name</label>
                            <input
                                id="o_xendit_last_name"
                                type="text"
                                placeholder="Smith"
                                required=""
                                class="form-control"
                            />
                    </div>
                </div>
                <!-- Phone and email -->
                <div class="row">
                    <div class="col-sm-6 mb-3">
                        <label for="o_xendit_phone">Phone Number</label>
                            <input
                                id="o_xendit_phone"
                                type="text"
                                required=""
                                class="form-control"
                            />
                    </div>
                    <div class="col-sm-6 mb-3">
                        <label for="o_xendit_email">Email</label>
                            <input
                                id="o_xendit_email"
                                type="text"
                                required=""
                                placeholder="john.smith@example.com"
                                class="form-control"
                            />
                    </div>
                </div>
                <!-- Card Number -->
                <div class="mb-3">
                    <label for="o_xendit_card" class="col-form-label">Card Number</label>
                    <input
                        id="o_xendit_card"
                        type="text"
                        required=""
                        maxlength="19"
                        autocomplete="cc-number"
                        class="form-control"
                    />
                </div>
                <!-- Expiry date and security code -->
                <div class="row">
                    <div class="col-sm-8 mb-3">
                        <label for="o_xendit_month">Expiration</label>
                        <div class="input-group">
                            <input
                                id="o_xendit_month"
                                type="number"
                                placeholder="MM"
                                min="1"
                                max="12"
                                autocomplete="cc-exp-month"
                                required=""
                                class="form-control"
                            />
                            <input
                                id="o_xendit_year"
                                type="number"
                                placeholder="YYYY"
                                min="1000"
                                max="9999"
                                autocomplete="cc-exp-year"
                                required=""
                                class="form-control"
                            />
                        </div>
                    </div>
                    <div class="col-sm-4 mb-3">
                        <label for="o_xendit_cvn">Card Code</label>
                        <input
                            id="o_xendit_cvn"
                            type="number"
                            max="9999"
                            autocomplete="cc-csc"
                            required=""
                            class="form-control"
                        />
                    </div>
                </div>
            </t>
        </div>
    </template>

</odoo>

```

