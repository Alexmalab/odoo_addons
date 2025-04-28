# Odoo Module: payment_demo

Category: Hidden

This file contains the source code of the Odoo module.

## File: const.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# The codes of the payment methods to activate when Demo is activated.
DEFAULT_PAYMENT_METHOD_CODES = [
    'demo',
]

```

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

from odoo.addons.payment import setup_provider, reset_payment_provider


def post_init_hook(env):
    setup_provider(env, 'demo')


def uninstall_hook(env):
    reset_payment_provider(env, 'demo')

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
        'views/payment_token_views.xml',
        'views/payment_transaction_views.xml',

        'data/payment_method_data.xml',
        'data/payment_provider_data.xml',  # Depends on `payment_method_demo`.
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

## File: data\payment_method_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="payment_method_demo" model="payment.method">
        <field name="name">Demo</field>
        <field name="code">demo</field>
        <field name="sequence">1</field>
        <field name="image" type="base64" file="payment_demo/static/img/demo.png"/>
        <field name="support_tokenization">True</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">partial</field>
    </record>

</odoo>

```

## File: data\payment_provider_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="payment.payment_provider_demo" model="payment.provider">
        <field name="code">demo</field>
        <field name="state">test</field>
        <field name="is_published">True</field>
        <field name="inline_form_view_id" ref="inline_form"/>
        <field name="token_inline_form_view_id" ref="token_inline_form"/>
        <field name="express_checkout_form_view_id" ref="express_checkout_form"/>
        <field name="allow_tokenization">True</field>
        <field name="allow_express_checkout">True</field>
        <field name="payment_method_ids"
               eval="[Command.set([
                         ref('payment_demo.payment_method_demo'),
                     ])]"
        />
    </record>

</odoo>

```

## File: models\payment_provider.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.exceptions import UserError

from odoo.addons.payment_demo import const


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
            'support_express_checkout': True,
            'support_manual_capture': 'partial',
            'support_refund': 'partial',
            'support_tokenization': True,
        })

    # === CONSTRAINT METHODS ===#

    @api.constrains('state', 'code')
    def _check_provider_state(self):
        if self.filtered(lambda p: p.code == 'demo' and p.state not in ('test', 'disabled')):
            raise UserError(_("Demo providers should never be enabled."))

    def _get_default_payment_method_codes(self):
        """ Override of `payment` to return the default payment method codes. """
        default_codes = super()._get_default_payment_method_codes()
        if self.code != 'demo':
            return default_codes
        return const.DEFAULT_PAYMENT_METHOD_CODES

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

    def _send_capture_request(self, amount_to_capture=None):
        """ Override of `payment` to simulate a capture request. """
        child_capture_tx = super()._send_capture_request(amount_to_capture=amount_to_capture)
        if self.provider_code != 'demo':
            return child_capture_tx

        tx = child_capture_tx or self
        notification_data = {
            'reference': tx.reference,
            'simulated_state': 'done',
            'manual_capture': True,  # Distinguish manual captures from regular one-step captures.
        }
        tx._handle_notification_data('demo', notification_data)

        return child_capture_tx

    def _send_void_request(self, amount_to_void=None):
        """ Override of `payment` to simulate a void request. """
        child_void_tx = super()._send_void_request(amount_to_void=amount_to_void)
        if self.provider_code != 'demo':
            return child_void_tx

        tx = child_void_tx or self
        notification_data = {'reference': tx.reference, 'simulated_state': 'cancel'}
        tx._handle_notification_data('demo', notification_data)

        return child_void_tx

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

        # Update the provider reference.
        self.provider_reference = f'demo-{self.reference}'

        # Create the token.
        if self.tokenize:
            # The reasons why we immediately tokenize the transaction regardless of the state rather
            # than waiting for the payment method to be validated ('authorized' or 'done') like the
            # other payment providers do are:
            # - To save the simulated state and payment details on the token while we have them.
            # - To allow customers to create tokens whose transactions will always end up in the
            #   said simulated state.
            self._demo_tokenize_from_notification_data(notification_data)

        # Update the payment state.
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
            'payment_method_id': self.payment_method_id.id,
            'payment_details': notification_data['payment_details'],
            'partner_id': self.partner_id.id,
            'provider_ref': 'fake provider reference',
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
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M0 12h50v26a4 4 0 0 1-4 4H0V12Z" fill="#FBB945"/><path d="M4 21a4 4 0 0 1-4-4v-5a4 4 0 0 1 4-4h46v9a4 4 0 0 1-4 4H4Z" fill="#FC868B"/><path d="M0 16h50v4a4 4 0 0 1-4 4H4a4 4 0 0 1-4-4v-4Z" fill="#953B24"/></svg>

```

## File: static\src\js\express_checkout_form.js

```javascript
/** @odoo-module */

import {_t} from '@web/core/l10n/translation';
import publicWidget from '@web/legacy/js/public/public_widget';
import { ConfirmationDialog } from '@web/core/confirmation_dialog/confirmation_dialog';
import { jsonrpc } from '@web/core/network/rpc_service';
import { debounce } from '@web/core/utils/timing';

import { paymentExpressCheckoutForm } from '@payment/js/express_checkout_form';
import paymentDemoMixin from '@payment_demo/js/payment_demo_mixin';

paymentExpressCheckoutForm.include({
    events: Object.assign({}, publicWidget.Widget.prototype.events, {
        'click button[name="o_payment_submit_button"]': '_initiateExpressPayment',
    }),

    // #=== WIDGET LIFECYCLE ===#

    /**
     * @override
     */
    start: async function () {
        await this._super(...arguments);
        document.querySelector('[name="o_payment_submit_button"]')?.removeAttribute('disabled');
        this.rpc = this.bindService('rpc');
        this._initiateExpressPayment = debounce(this._initiateExpressPayment, 500, true);
    },

    // #=== EVENT HANDLERS ===#

    /**
     * Process the payment.
     *
     * @private
     * @param {Event} ev
     * @return {void}
     */
    async _initiateExpressPayment(ev) {
        ev.stopPropagation();
        ev.preventDefault();

        const shippingInformationRequired = document.querySelector(
            '[name="o_payment_express_checkout_form"]'
        ).dataset.shippingInfoRequired;
        const providerId = ev.target.parentElement.dataset.providerId;
        let expressShippingAddress = {};
        if (shippingInformationRequired){
            const shippingInfo = document.querySelector(
                `#o_payment_demo_shipping_info_${providerId}`
            );
            expressShippingAddress =  {
                'name': shippingInfo.querySelector('#o_payment_demo_shipping_name').value,
                'email': shippingInfo.querySelector('#o_payment_demo_shipping_email').value,
                'street': shippingInfo.querySelector('#o_payment_demo_shipping_address').value,
                'street2': shippingInfo.querySelector('#o_payment_demo_shipping_address2').value,
                'zip': shippingInfo.querySelector('#o_payment_demo_shipping_zip').value,
                'city': shippingInfo.querySelector('#o_payment_demo_shipping_city').value,
                'country': shippingInfo.querySelector('#o_payment_demo_shipping_country').value,
            };
            // Call the shipping address update route to fetch the shipping options.
            const availableCarriers = await this.rpc(
                this.paymentContext['shippingAddressUpdateRoute'],
                {partial_shipping_address: expressShippingAddress},
            );
            if (availableCarriers.length > 0) {
                const id = parseInt(availableCarriers[0].id);
                await this.rpc('/shop/update_carrier', {carrier_id: id});
            } else {
                this.call('dialog', 'add', ConfirmationDialog, {
                    title: _t("Validation Error"),
                    body: _t("No delivery method is available."),
                });
                return;
            }
        }
        await jsonrpc(
            document.querySelector(
                '[name="o_payment_express_checkout_form"]'
            ).dataset['expressCheckoutRoute'],
            {
                'shipping_address': expressShippingAddress,
                'billing_address': {
                    'name': 'Demo User',
                    'email': 'demo@test.com',
                    'street': 'Rue des Bourlottes 9',
                    'street2': '23',
                    'country': 'BE',
                    'city':'Ramillies',
                    'zip':'1367'
                },
            }
        );
        const processingValues = await jsonrpc(
            this.paymentContext['transactionRoute'],
            this._prepareTransactionRouteParams(providerId),
        )
        paymentDemoMixin.processDemoPayment(processingValues);
    },
});

```

## File: static\src\js\payment_demo_mixin.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { jsonrpc, RPCError } from "@web/core/network/rpc_service";

export default {

    /**
     * Simulate a feedback from a payment provider and redirect the customer to the status page.
     *
     * @private
     * @param {object} processingValues - The processing values of the transaction.
     * @return {void}
     */
    async processDemoPayment(processingValues) {
        const customerInput = document.getElementById('customer_input').value;
        const simulatedPaymentState = document.getElementById('simulated_payment_state').value;

        jsonrpc('/payment/demo/simulate_payment', {
            'reference': processingValues.reference,
            'payment_details': customerInput,
            'simulated_state': simulatedPaymentState,
        }).then(() => {
            window.location = '/payment/status';
        }).catch(error => {
            if (error instanceof RPCError) {
                this._displayErrorDialog(_t("Payment processing failed"), error.data.message);
                this._enableButton?.(); // This method doesn't exists in Express Checkout form.
            } else {
                return Promise.reject(error);
            }
        });
    },

};

```

## File: static\src\js\payment_form.js

```javascript
/** @odoo-module **/

import paymentForm from '@payment/js/payment_form';
import paymentDemoMixin from '@payment_demo/js/payment_demo_mixin';

paymentForm.include({

    // #=== DOM MANIPULATION ===#

    /**
     * Prepare the inline form of Demo for direct payment.
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
        if (providerCode !== 'demo') {
            this._super(...arguments);
            return;
        } else if (flow === 'token') {
            return;
        }
        this._setPaymentFlow('direct');
    },

    // #=== PAYMENT FLOW ===#

    /**
     * Simulate a feedback from a payment provider and redirect the customer to the status page.
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
        if (providerCode !== 'demo') {
            this._super(...arguments);
            return;
        }
        paymentDemoMixin.processDemoPayment(processingValues);
    },

});

```

## File: views\payment_demo_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="payment_demo.token_form" inherit_id="payment.token_form">
    </template>

    <template id="payment_demo.payment_method_form" inherit_id="payment.method_form">
    </template>

    <template id="inline_form">
        <div t-attf-id="demo-container-{{provider_id}}">
            <t t-call="payment_demo.payment_details"/>
        </div>
    </template>

    <template id="token_inline_form">
        <div t-attf-id="demo-token-container-{{token_sudo.id}}">
            <div class="alert alert-warning mb-0">
                <span t-if="token_sudo.demo_simulated_state=='pending'">
                    Payments made with this payment method will remain <b>pending</b>.
                </span>
                <span t-elif="token_sudo.demo_simulated_state=='done'">
                    Payments made with this payment method will be <b>successful</b>.
                </span>
                <span t-elif="token_sudo.demo_simulated_state=='cancel'">
                    Payments made with this payment method will be automatically <b>canceled</b>.
                </span>
                <span t-else="">
                    Payments made with this payment method will simulate a processing <b>error</b>.
                </span>
            </div>
        </div>
    </template>

    <template id="express_checkout_form">
        <div name="o_express_checkout_container"
             t-attf-id="o_demo_express_checkout_container_{{provider_sudo.id}}"
             t-att-data-provider-id="provider_sudo.id"
             t-att-data-provider-code="provider_sudo.code"
        >
            <button type="button"
                    class="btn btn-primary mt-2 w-100"
                    data-bs-toggle="modal"
                    t-attf-data-bs-target="#o_payment_demo_modal_{{provider_sudo.id}}"
            >
                    Pay with Demo
                    <span t-if="not provider_sudo.is_published"
                          class="badge rounded-pill text-bg-danger ms-1"
                    >
                          Unpublished
                    </span>
            </button>
            <div t-attf-id="o_payment_demo_modal_{{provider_sudo.id}}"
                 class="modal fade mt-5"
                 tabindex="-1"
                 aria-labelledby="o_payment_demo_modal_label"
                 aria-hidden="true"
            >
                <div class="modal-dialog modal-dialog-centered">
                    <div class="modal-content">
                        <div class="modal-header">
                            <h5 class="modal-title" id="o_payment_demo_modal_label">
                                Demo Express Checkout
                            </h5>
                            <span t-if="provider_sudo.state == 'test'"
                                  class="badge rounded-pill text-bg-warning ms-1"
                            >
                                Test Mode
                            </span>
                            <button type="button"
                                    class="btn-close"
                                    data-bs-dismiss="modal"
                                    aria-label="Close"
                            />
                        </div>
                        <div class="modal-body">
                            <t t-call="payment_demo.express_inline_form">
                                <t t-set="provider_id" t-value="provider_sudo.id"/>
                            </t>
                            <div class="float-end mt-2" t-att-data-provider-id="provider_sudo.id">
                                <t t-call="payment.submit_button">
                                    <t t-set="submit_button_label">Pay</t>
                                </t>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </template>

    <template id="express_inline_form">
        <div>
            <t t-call="payment_demo.payment_details"/>
            <div t-if="shipping_info_required"
                 t-attf-id="o_payment_demo_shipping_info_{{provider_id}}"
            >
                <t t-set="customer" t-value="request.env.user.partner_id"/>
                <div class="row mt-0">
                    <div class="col mb-0">
                        <label for="o_payment_demo_shipping_name" class=" mt-0">
                            <small><b>Name</b></small>
                        </label>
                        <input type="text"
                               id="o_payment_demo_shipping_name"
                               class="form-control"
                               t-att-value="customer.name"
                               readonly="1"
                               required=""
                        />
                    </div>
                    <div class="col mb-0">
                        <label for="o_payment_demo_shipping_email" class=" mt-0">
                            <small>Email</small>
                        </label>
                        <input type="text"
                               id="o_payment_demo_shipping_email"
                               class="form-control"
                               t-att-value="customer.email or 'example@example.com'"
                               readonly="1"
                        />
                    </div>
                </div>
                <div class="row">
                    <div class="col mb-0">
                        <label for="o_payment_demo_shipping_address" class="mt-1">
                            <small><b>Street and Number</b></small>
                        </label>
                        <input type="text"
                               id="o_payment_demo_shipping_address"
                               class="form-control"
                               required=""
                               t-att-value="customer.street or 'Rue des Bourlottes 9'"
                               readonly="1"
                        />
                    </div>
                    <div class="col mb-0">
                        <label for="o_payment_demo_shipping_address2" class="mt-1">
                            <small>Street 2</small>
                        </label>
                        <input type="text"
                               id="o_payment_demo_shipping_address2"
                               class="form-control"
                               t-att-value="customer.street2"
                               readonly="1"
                        />
                    </div>
                </div>
                <div class="row">
                    <div class="col mb-0">
                        <label for="o_payment_demo_shipping_zip" class="mt-1">
                            <small><b>Zip Code</b></small>
                        </label>
                        <input type="text"
                               id="o_payment_demo_shipping_zip"
                               class="form-control"
                               required=""
                               t-att-value="customer.zip or '1367'"
                               readonly="1"
                        />
                    </div>
                    <div class="col mb-0">
                        <label for="o_payment_demo_shipping_city" class="mt-1">
                            <small><b>City</b></small>
                        </label>
                        <input type="text"
                               id="o_payment_demo_shipping_city"
                               class="form-control"
                               t-att-value="customer.city or 'Ramillies'"
                               readonly="1"
                               required=""
                        />
                    </div>
                    <div class="col mb-0">
                        <label for="o_payment_demo_shipping_country" class="mt-1">
                            <small><b>Country</b></small>
                        </label>
                        <select id="o_payment_demo_shipping_country"
                                class="form-select"
                                disabled="true"
                        >
                            <option t-att-value="customer.country_id.code or 'BE'"
                                    t-out="customer.country_id.name or 'Belgium'"
                            />
                        </select>
                    </div>
                </div>
            </div>
        </div>
    </template>

    <template id="payment_details">
        <div class="row gap2 gap-md-0">
            <input name="provider_id" type="hidden" id="provider_id" t-att-value="id"/>
            <input name="partner_id" type="hidden" t-att-value="partner_id"/>
            <div class="col-12 col-md mt-0 mb-0">
                <label for="customer_input" class="mt-0">
                    <small>Payment Details (test data)</small>
                </label>
                <input type="text"
                       name="customer_input"
                       id="customer_input"
                       class="form-control"
                       placeholder="XXXX XXXX XXXX XXXX"/>
            </div>
            <div class="col-12 col-md mb-0">
                <label for="simulated_payment_state" class="mt-0 text-muted">
                    <small>Payment Status</small>
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
                       invisible="provider_code != 'demo'"
                       required="provider_code == 'demo'"/>
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
                        invisible="provider_code != 'demo' or not capture_manually or state != 'pending'"/>
                <button string="Confirm"
                        type="object"
                        name="action_demo_set_done"
                        class="oe_highlight"
                        invisible="provider_code != 'demo' or capture_manually or state != 'pending'"/>
                <button string="Cancel"
                        type="object"
                        name="action_demo_set_canceled"
                        invisible="provider_code != 'demo' or state != 'pending'"/>
                <button string="Set to Error"
                        type="object"
                        name="action_demo_set_error"
                        invisible="provider_code != 'demo' or state != 'pending'"/>
            </header>
        </field>
    </record>

</odoo>

```

