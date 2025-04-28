# Odoo Module: payment_demo

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

from odoo.addons.payment import setup_provider, reset_payment_provider


def post_init_hook(cr, registry):
    setup_provider(cr, registry, 'demo')


def uninstall_hook(cr, registry):
    reset_payment_provider(cr, registry, 'demo')

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Payment Provider: Demo',
    'version': '2.0',
    'category': 'Hidden',
    'sequence': 350,
    'summary': "A payment provider for running fake payment flows for demo purposes.",
    'description': " ",  # Non-empty string to avoid loading the README file.
    'depends': ['payment'],
    'data': [
        'views/payment_demo_templates.xml',
        'views/payment_templates.xml',
        'views/payment_token_views.xml',
        'views/payment_transaction_views.xml',
        'data/payment_provider_data.xml',
    ],
    'post_init_hook': 'post_init_hook',
    'uninstall_hook': 'uninstall_hook',
    'assets': {
        'web.assets_frontend': [
            'payment_demo/static/src/js/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http
from odoo.http import request


class PaymentDemoController(http.Controller):
    _simulation_url = '/payment/demo/simulate_payment'

    @http.route(_simulation_url, type='json', auth='public')
    def demo_simulate_payment(self, **data):
        """ Simulate the response of a payment request.

        :param dict data: The simulated notification data.
        :return: None
        """
        request.env['payment.transaction'].sudo()._handle_notification_data('demo', data)

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\payment_provider_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="payment.payment_provider_demo" model="payment.provider">
        <field name="code">demo</field>
        <field name="inline_form_view_id" ref="inline_form"/>
        <field name="token_inline_form_view_id" ref="token_inline_form"/>
        <field name="allow_tokenization">True</field>
    </record>

</odoo>

```

## File: models\payment_provider.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.exceptions import UserError


class PaymentProvider(models.Model):
    _inherit = 'payment.provider'

    code = fields.Selection(selection_add=[('demo', 'Demo')], ondelete={'demo': 'set default'})

    #=== COMPUTE METHODS ===#

    @api.depends('code')
    def _compute_view_configuration_fields(self):
        """ Override of payment to hide the credentials page.

        :return: None
        """
        super()._compute_view_configuration_fields()
        self.filtered(lambda p: p.code == 'demo').show_credentials_page = False

    def _compute_feature_support_fields(self):
        """ Override of `payment` to enable additional features. """
        super()._compute_feature_support_fields()
        self.filtered(lambda p: p.code == 'demo').update({
            'support_fees': True,
            'support_manual_capture': True,
            'support_refund': 'partial',
            'support_tokenization': True,
        })

    # === CONSTRAINT METHODS ===#

    @api.constrains('state', 'code')
    def _check_provider_state(self):
        if self.filtered(lambda p: p.code == 'demo' and p.state not in ('test', 'disabled')):
            raise UserError(_("Demo providers should never be enabled."))

```

## File: models\payment_token.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details

from odoo import fields, models


class PaymentToken(models.Model):
    _inherit = 'payment.token'

    demo_simulated_state = fields.Selection(
        string="Simulated State",
        help="The state in which transactions created from this token should be set.",
        selection=[
            ('pending', "Pending"),
            ('done', "Confirmed"),
            ('cancel', "Canceled"),
            ('error', "Error"),
        ],
    )

    def _build_display_name(self, *args, should_pad=True, **kwargs):
        """ Override of `payment` to build the display name without padding.

        Note: self.ensure_one()

        :param list args: The arguments passed by QWeb when calling this method.
        :param bool should_pad: Whether the token should be padded or not.
        :param dict kwargs: Optional data.
        :return: The demo token name.
        :rtype: str
        """
        if self.provider_code != 'demo':
            return super()._build_display_name(*args, should_pad=should_pad, **kwargs)
        return super()._build_display_name(*args, should_pad=False, **kwargs)

```

## File: models\payment_transaction.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from odoo import _, fields, models
from odoo.exceptions import UserError, ValidationError

from odoo.addons.payment import utils as payment_utils

_logger = logging.getLogger(__name__)


class PaymentTransaction(models.Model):
    _inherit = 'payment.transaction'

    capture_manually = fields.Boolean(related='provider_id.capture_manually')

    #=== ACTION METHODS ===#

    def action_demo_set_done(self):
        """ Set the state of the demo transaction to 'done'.

        Note: self.ensure_one()

        :return: None
        """
        self.ensure_one()
        if self.provider_code != 'demo':
            return

        notification_data = {'reference': self.reference, 'simulated_state': 'done'}
        self._handle_notification_data('demo', notification_data)

    def action_demo_set_canceled(self):
        """ Set the state of the demo transaction to 'cancel'.

        Note: self.ensure_one()

        :return: None
        """
        self.ensure_one()
        if self.provider_code != 'demo':
            return

        notification_data = {'reference': self.reference, 'simulated_state': 'cancel'}
        self._handle_notification_data('demo', notification_data)

    def action_demo_set_error(self):
        """ Set the state of the demo transaction to 'error'.

        Note: self.ensure_one()

        :return: None
        """
        self.ensure_one()
        if self.provider_code != 'demo':
            return

        notification_data = {'reference': self.reference, 'simulated_state': 'error'}
        self._handle_notification_data('demo', notification_data)

    #=== BUSINESS METHODS ===#

    def _send_payment_request(self):
        """ Override of payment to simulate a payment request.

        Note: self.ensure_one()

        :return: None
        """
        super()._send_payment_request()
        if self.provider_code != 'demo':
            return

        if not self.token_id:
            raise UserError("Demo: " + _("The transaction is not linked to a token."))

        simulated_state = self.token_id.demo_simulated_state
        notification_data = {'reference': self.reference, 'simulated_state': simulated_state}
        self._handle_notification_data('demo', notification_data)

    def _send_refund_request(self, **kwargs):
        """ Override of payment to simulate a refund.

        Note: self.ensure_one()

        :param dict kwargs: The keyword arguments.
        :return: The refund transaction created to process the refund request.
        :rtype: recordset of `payment.transaction`
        """
        refund_tx = super()._send_refund_request(**kwargs)
        if self.provider_code != 'demo':
            return refund_tx

        notification_data = {'reference': refund_tx.reference, 'simulated_state': 'done'}
        refund_tx._handle_notification_data('demo', notification_data)

        return refund_tx

    def _send_capture_request(self):
        """ Override of payment to simulate a capture request.

        Note: self.ensure_one()

        :return: None
        """
        super()._send_capture_request()
        if self.provider_code != 'demo':
            return

        notification_data = {
            'reference': self.reference,
            'simulated_state': 'done',
            'manual_capture': True,  # Distinguish manual captures from regular one-step captures.
        }
        self._handle_notification_data('demo', notification_data)

    def _send_void_request(self):
        """ Override of payment to simulate a void request.

        Note: self.ensure_one()

        :return: None
        """
        super()._send_void_request()
        if self.provider_code != 'demo':
            return

        notification_data = {'reference': self.reference, 'simulated_state': 'cancel'}
        self._handle_notification_data('demo', notification_data)

    def _get_tx_from_notification_data(self, provider_code, notification_data):
        """ Override of payment to find the transaction based on dummy data.

        :param str provider_code: The code of the provider that handled the transaction
        :param dict notification_data: The dummy notification data
        :return: The transaction if found
        :rtype: recordset of `payment.transaction`
        :raise: ValidationError if the data match no transaction
        """
        tx = super()._get_tx_from_notification_data(provider_code, notification_data)
        if provider_code != 'demo' or len(tx) == 1:
            return tx

        reference = notification_data.get('reference')
        tx = self.search([('reference', '=', reference), ('provider_code', '=', 'demo')])
        if not tx:
            raise ValidationError(
                "Demo: " + _("No transaction found matching reference %s.", reference)
            )
        return tx

    def _process_notification_data(self, notification_data):
        """ Override of payment to process the transaction based on dummy data.

        Note: self.ensure_one()

        :param dict notification_data: The dummy notification data
        :return: None
        :raise: ValidationError if inconsistent data were received
        """
        super()._process_notification_data(notification_data)
        if self.provider_code != 'demo':
            return

        self.provider_reference = f'demo-{self.reference}'

        if self.tokenize:
            # The reasons why we immediately tokenize the transaction regardless of the state rather
            # than waiting for the payment method to be validated ('authorized' or 'done') like the
            # other payment providers do are:
            # - To save the simulated state and payment details on the token while we have them.
            # - To allow customers to create tokens whose transactions will always end up in the
            #   said simulated state.
            self._demo_tokenize_from_notification_data(notification_data)

        state = notification_data['simulated_state']
        if state == 'pending':
            self._set_pending()
        elif state == 'done':
            if self.capture_manually and not notification_data.get('manual_capture'):
                self._set_authorized()
            else:
                self._set_done()
                # Immediately post-process the transaction if it is a refund, as the post-processing
                # will not be triggered by a customer browsing the transaction from the portal.
                if self.operation == 'refund':
                    self.env.ref('payment.cron_post_process_payment_tx')._trigger()
        elif state == 'cancel':
            self._set_canceled()
        else:  # Simulate an error state.
            self._set_error(_("You selected the following demo payment status: %s", state))

    def _demo_tokenize_from_notification_data(self, notification_data):
        """ Create a new token based on the notification data.

        Note: self.ensure_one()

        :param dict notification_data: The fake notification data to tokenize from.
        :return: None
        """
        self.ensure_one()

        state = notification_data['simulated_state']
        token = self.env['payment.token'].create({
            'provider_id': self.provider_id.id,
            'payment_details': notification_data['payment_details'],
            'partner_id': self.partner_id.id,
            'provider_ref': 'fake provider reference',
            'verified': True,
            'demo_simulated_state': state,
        })
        self.write({
            'token_id': token,
            'tokenize': False,
        })
        _logger.info(
            "Created token with id %s for partner with id %s.", token.id, self.partner_id.id
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
<svg width="70" height="70" viewBox="0 0 70 70" fill="none" xmlns="http://www.w3.org/2000/svg">
<g clip-path="url(#clip0_160_30)">
<mask id="mask0_160_30" style="mask-type:alpha" maskUnits="userSpaceOnUse" x="0" y="0" width="70" height="70">
<path d="M4 0H65C69 0 70 1 70 5V65C70 69 69 70 65 70H4C1 70 0 69 0 65V5C0 1 1 0 4 0Z" fill="white"/>
</mask>
<g mask="url(#mask0_160_30)">
<path fill-rule="evenodd" clip-rule="evenodd" d="M0 0H70V70H0V0Z" fill="url(#paint0_linear_160_30)"/>
<path fill-rule="evenodd" clip-rule="evenodd" d="M4 1H65C67.667 1 69.333 1.667 70 3V0H0V3C0.667 1.667 2 1 4 1Z" fill="white" fill-opacity="0.383"/>
<path fill-rule="evenodd" clip-rule="evenodd" d="M4 69H65C67.667 69 69.333 68 70 66V70H0V66C0.667 68 2 69 4 69Z" fill="black" fill-opacity="0.383"/>
</g>
<path opacity="0.324" fill-rule="evenodd" clip-rule="evenodd" d="M4 69C2 69 0 68 0 65V39.5C0 39.5 16 23.703 17.5 22.703H31H37.25L48 23L51.576 24.968L51.34 46.978L39.224 69H4Z" fill="#393939"/>
<path fill-rule="evenodd" clip-rule="evenodd" d="M28 27.445H48.697C48.7874 27.4458 48.8737 27.4823 48.9372 27.5466C49.0008 27.6108 49.0363 27.6976 49.036 27.788V31H28.0612L31.7806 37.5H49.036V47.212C49.0363 47.3024 49.0008 47.3892 48.9372 47.4534C48.8737 47.5177 48.7874 47.5542 48.697 47.555H22.303C22.116 47.555 21.964 47.4 21.964 47.212V44H19.2595L19.25 47.555C19.25 49.069 20.464 50.297 21.964 50.297H49.036C50.537 50.297 51.75 49.069 51.75 47.555V27.445C51.75 25.931 50.536 24.703 49.036 24.703H28V27.445Z" fill="#465C68"/>
<path fill-rule="evenodd" clip-rule="evenodd" d="M28 25.445H48.697C48.7874 25.4458 48.8737 25.4823 48.9372 25.5466C49.0008 25.6108 49.0363 25.6976 49.036 25.788V29H28.0612L31.7806 35.5H49.036V45.212C49.0363 45.3024 49.0008 45.3892 48.9372 45.4534C48.8737 45.5177 48.7874 45.5542 48.697 45.555H22.303C22.116 45.555 21.964 45.4 21.964 45.212V42H19.2595L19.25 45.555C19.25 47.069 20.464 48.297 21.964 48.297H49.036C50.537 48.297 51.75 47.069 51.75 45.555V25.445C51.75 23.931 50.536 22.703 49.036 22.703H28V25.445Z" fill="white"/>
<path fill-rule="evenodd" clip-rule="evenodd" d="M19.9412 12.3794C19.9412 12.9967 19.4408 13.4971 18.8235 13.4971C18.2063 13.4971 17.7059 12.9967 17.7059 12.3794C17.7059 11.7622 18.2063 11.2618 18.8235 11.2618C19.4408 11.2618 19.9412 11.7622 19.9412 12.3794ZM22.7353 15.7324C23.6612 15.7324 24.4118 14.9818 24.4118 14.0559C24.4118 13.13 23.6612 12.3794 22.7353 12.3794C21.8094 12.3794 21.0588 13.13 21.0588 14.0559C21.0588 14.9818 21.8094 15.7324 22.7353 15.7324ZM19.3824 19.0853C20.3082 19.0853 21.0588 18.3347 21.0588 17.4088C21.0588 16.4829 20.3082 15.7324 19.3824 15.7324C18.4565 15.7324 17.7059 16.4829 17.7059 17.4088C17.7059 18.3347 18.4565 19.0853 19.3824 19.0853ZM22.3442 20.9457H18.3501C17.239 20.9457 16.7525 21.7025 16.7525 22.1642C16.7525 22.746 17.0797 23.2533 17.5645 23.5207C17.5558 23.5865 17.5513 23.6536 17.5513 23.7217V29.1963L11.2445 39.0567C10.581 40.0941 11.3461 41.4383 12.6002 41.4383H28.3998C29.6539 41.4383 30.419 40.0941 29.7555 39.0567L23.143 28.7184V23.7217C23.143 23.6536 23.1385 23.5865 23.1298 23.5207C23.6146 23.2533 23.9418 22.746 23.9418 22.1642C23.9418 21.7505 23.4553 20.9457 22.3442 20.9457ZM21.5454 23.7217L19.1489 23.7217V29.1963C19.1489 29.4877 19.0651 29.7732 18.907 30.0204L12.6002 39.8808H28.3998L21.7873 29.5425C21.6292 29.2953 21.5454 29.0098 21.5454 28.7184V23.7217ZM26.3383 38.9073H14.356L17.5629 34.0442C17.7835 33.7096 18.2294 33.5929 18.6259 33.6864C19.2966 33.8445 20.2476 33.894 21.146 33.4561C22.4241 32.8331 23.2762 33.7157 23.5424 34.2348L26.3383 38.9073Z" fill="#465C68"/>
<path fill-rule="evenodd" clip-rule="evenodd" d="M19.9412 10.3794C19.9412 10.9967 19.4408 11.4971 18.8235 11.4971C18.2063 11.4971 17.7059 10.9967 17.7059 10.3794C17.7059 9.76217 18.2063 9.26178 18.8235 9.26178C19.4408 9.26178 19.9412 9.76217 19.9412 10.3794ZM22.7353 13.7324C23.6612 13.7324 24.4118 12.9818 24.4118 12.0559C24.4118 11.13 23.6612 10.3794 22.7353 10.3794C21.8094 10.3794 21.0588 11.13 21.0588 12.0559C21.0588 12.9818 21.8094 13.7324 22.7353 13.7324ZM19.3824 17.0853C20.3082 17.0853 21.0588 16.3347 21.0588 15.4088C21.0588 14.4829 20.3082 13.7324 19.3824 13.7324C18.4565 13.7324 17.7059 14.4829 17.7059 15.4088C17.7059 16.3347 18.4565 17.0853 19.3824 17.0853ZM22.3442 18.9457H18.3501C17.239 18.9457 16.7525 19.7025 16.7525 20.1642C16.7525 20.746 17.0797 21.2533 17.5645 21.5207C17.5558 21.5865 17.5513 21.6536 17.5513 21.7217V27.1963L11.2445 37.0567C10.581 38.0941 11.3461 39.4383 12.6002 39.4383H28.3998C29.6539 39.4383 30.419 38.0941 29.7555 37.0567L23.143 26.7184V21.7217C23.143 21.6536 23.1385 21.5865 23.1298 21.5207C23.6146 21.2533 23.9418 20.746 23.9418 20.1642C23.9418 19.7505 23.4553 18.9457 22.3442 18.9457ZM21.5454 21.7217L19.1489 21.7217V27.1963C19.1489 27.4877 19.0651 27.7732 18.907 28.0204L12.6002 37.8808H28.3998L21.7873 27.5425C21.6292 27.2953 21.5454 27.0098 21.5454 26.7184V21.7217ZM26.3383 36.9073H14.356L17.5629 32.0442C17.7835 31.7096 18.2294 31.5929 18.6259 31.6864C19.2966 31.8445 20.2476 31.894 21.146 31.4561C22.4241 30.8331 23.2762 31.7157 23.5424 32.2348L26.3383 36.9073Z" fill="white"/>
</g>
<defs>
<linearGradient id="paint0_linear_160_30" x1="70" y1="0" x2="0" y2="70" gradientUnits="userSpaceOnUse">
<stop stop-color="#CDC484"/>
<stop offset="1" stop-color="#B5AA59"/>
</linearGradient>
<clipPath id="clip0_160_30">
<rect width="70" height="70" fill="white"/>
</clipPath>
</defs>
</svg>

```

## File: static\src\js\payment_form.js

```javascript
odoo.define('payment_demo.payment_form', require => {
    'use strict';

    const checkoutForm = require('payment.checkout_form');
    const manageForm = require('payment.manage_form');

    const paymentDemoMixin = {

        //--------------------------------------------------------------------------
        // Private
        //--------------------------------------------------------------------------

        /**
         * Simulate a feedback from a payment provider and redirect the customer to the status page.
         *
         * @override method from payment.payment_form_mixin
         * @private
         * @param {string} code - The code of the provider
         * @param {number} providerId - The id of the provider handling the transaction
         * @param {object} processingValues - The processing values of the transaction
         * @return {Promise}
         */
        _processDirectPayment: function (code, providerId, processingValues) {
            if (code !== 'demo') {
                return this._super(...arguments);
            }

            const customerInput = document.getElementById('customer_input').value;
            const simulatedPaymentState = document.getElementById('simulated_payment_state').value;
            return this._rpc({
                route: '/payment/demo/simulate_payment',
                params: {
                    'reference': processingValues.reference,
                    'payment_details': customerInput,
                    'simulated_state': simulatedPaymentState,
                },
            }).then(() => {
                window.location = '/payment/status';
            });
        },

        /**
         * Prepare the inline form of Demo for direct payment.
         *
         * @override method from payment.payment_form_mixin
         * @private
         * @param {string} code - The code of the selected payment option's provider
         * @param {integer} paymentOptionId - The id of the selected payment option
         * @param {string} flow - The online payment flow of the selected payment option
         * @return {Promise}
         */
        _prepareInlineForm: function (code, paymentOptionId, flow) {
            if (code !== 'demo') {
                return this._super(...arguments);
            } else if (flow === 'token') {
                return Promise.resolve();
            }
            this._setPaymentFlow('direct');
            return Promise.resolve()
        },
    };
    checkoutForm.include(paymentDemoMixin);
    manageForm.include(paymentDemoMixin);
});

```

## File: views\payment_demo_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="inline_form">
        <div t-attf-id="demo-container-{{provider_id}}">
            <div class="row">
                <div class="mb-3">
                    <input name="provider_id" type="hidden" id="provider_id" t-att-value="id"/>
                    <input name="partner_id" type="hidden" t-att-value="partner_id"/>
                </div>
                <div class="col mt-0 mb-0">
                    <label for="customer_input" class="mt-0">
                        <small><b>Payment Details (test data)</b></small>
                    </label>
                    <input type="text"
                           name="customer_input"
                           id="customer_input"
                           class="form-control"
                           placeholder="XXXX XXXX XXXX XXXX"/>
                </div>
                <div class="col mb-0">
                    <label for="simulated_payment_state" class="mt-0">
                        <small><b>Payment Status</b></small>
                    </label>
                    <select id="simulated_payment_state" class="form-select">
                        <option value="done" title="Successful payment">
                            Successful
                        </option>
                        <option value="pending" title="Payment processing">
                            Pending
                        </option>
                        <option value="cancel" title="Payment canceled by customer">
                            Canceled
                        </option>
                        <option value="error" title="Processing error">
                            Error
                        </option>
                    </select>
                </div>
            </div>
        </div>
    </template>

    <template id="token_inline_form">
        <div t-attf-id="demo-token-container-{{token.id}}">
            <div class="alert alert-warning m-2">
                <span t-if="token.demo_simulated_state=='pending'">
                    Payments made with this payment method will remain <b>pending</b>.
                </span>
                <span t-elif="token.demo_simulated_state=='done'">
                    Payments made with this payment method will be <b>successful</b>.
                </span>
                <span t-elif="token.demo_simulated_state=='cancel'">
                    Payments made with this payment method will be automatically <b>canceled</b>.
                </span>
                <span t-else="">
                    Payments made with this payment method will simulate a processing <b>error</b>.
                </span>
            </div>
        </div>
    </template>

</odoo>

```

## File: views\payment_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="verified_token_checkmark" inherit_id="payment.verified_token_checkmark">
        <xpath expr="//t[@name='payment_demo_hook']" position="replace">
            <t t-if="token.provider_code=='demo'">
                <span class="badge rounded-pill text-bg-warning">Demo Token</span>
            </t>
        </xpath>
    </template>

</odoo>

```

## File: views\payment_token_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_token_form" model="ir.ui.view">
        <field name="name">Demo Token Form</field>
        <field name="model">payment.token</field>
        <field name="inherit_id" ref="payment.payment_token_form"/>
        <field name="arch" type="xml">
            <group name="general_information" position="inside">
                <field name="provider_code" invisible="1"/>
                <field name="demo_simulated_state"
                       attrs="{'invisible': [('provider_code', '!=', 'demo')], 'required': [('provider_code', '=', 'demo')]}"/>
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
        <field name="name">Demo Transaction Form</field>
        <field name="model">payment.transaction</field>
        <field name="inherit_id" ref="payment.payment_transaction_form"/>
        <field name="arch" type="xml">
            <header position="inside">
                <field name="capture_manually" invisible="1"/>
                <button string="Authorize"
                        type="object"
                        name="action_demo_set_done"
                        class="oe_highlight"
                        attrs="{'invisible': ['|', '|', ('provider_code', '!=', 'demo'), ('capture_manually', '=', False), ('state', '!=', 'pending')]}"/>
                <button string="Confirm"
                        type="object"
                        name="action_demo_set_done"
                        class="oe_highlight"
                        attrs="{'invisible': ['|', '|', ('provider_code', '!=', 'demo'), ('capture_manually', '=', True), ('state', '!=', 'pending')]}"/>
                <button string="Cancel"
                        type="object"
                        name="action_demo_set_canceled"
                        attrs="{'invisible': ['|', ('provider_code', '!=', 'demo'), ('state', '!=', 'pending')]}"/>
                <button string="Set to Error"
                        type="object"
                        name="action_demo_set_error"
                        attrs="{'invisible': ['|', ('provider_code', '!=', 'demo'), ('state', '!=', 'pending')]}"/>
            </header>
        </field>
    </record>

</odoo>

```

