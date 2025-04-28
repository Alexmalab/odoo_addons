# Odoo Module: pos_stripe

Category: Sales/Point of Sale

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# coding: utf-8
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'POS Stripe',
    'version': '1.0',
    'category': 'Sales/Point of Sale',
    'sequence': 6,
    'summary': 'Integrate your POS with a Stripe payment terminal',
    'description': '',
    'data': [
        'views/pos_payment_method_views.xml',
        'views/assets_stripe.xml',
    ],
    'depends': ['point_of_sale', 'payment_stripe'],
    'installable': True,
    'assets': {
        'point_of_sale.assets': [
            'pos_stripe/static/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: models\pos_payment_method.py

```python
# coding: utf-8
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import logging
import requests
import werkzeug

from odoo import fields, models, api, _
from odoo.exceptions import ValidationError, UserError, AccessError

_logger = logging.getLogger(__name__)
TIMEOUT = 10

class PosPaymentMethod(models.Model):
    _inherit = 'pos.payment.method'

    def _get_payment_terminal_selection(self):
        return super()._get_payment_terminal_selection() + [('stripe', 'Stripe')]

    # Stripe
    stripe_serial_number = fields.Char(help='[Serial number of the stripe terminal], for example: WSC513105011295', copy=False)

    @api.constrains('stripe_serial_number')
    def _check_stripe_serial_number(self):
        for payment_method in self:
            if not payment_method.stripe_serial_number:
                continue
            existing_payment_method = self.search([('id', '!=', payment_method.id),
                                                   ('stripe_serial_number', '=', payment_method.stripe_serial_number)],
                                                  limit=1)
            if existing_payment_method:
                raise ValidationError(_('Terminal %s is already used on payment method %s.',\
                     payment_method.stripe_serial_number, existing_payment_method.display_name))

    def _get_stripe_payment_provider(self):
        stripe_payment_provider = self.env['payment.provider'].search([
            ('code', '=', 'stripe'),
            ('company_id', '=', self.env.company.id)
        ], limit=1)

        if not stripe_payment_provider:
            raise UserError(_("Stripe payment provider for company %s is missing", self.env.company.name))

        return stripe_payment_provider

    @api.model
    def _get_stripe_secret_key(self):
        stripe_secret_key = self._get_stripe_payment_provider().stripe_secret_key

        if not stripe_secret_key:
            raise ValidationError(_('Complete the Stripe onboarding for company %s.', self.env.company.name))

        return stripe_secret_key

    @api.model
    def stripe_connection_token(self):
        if not self.env.user.has_group('point_of_sale.group_pos_user'):
            raise AccessError(_("Do not have access to fetch token from Stripe"))

        endpoint = 'https://api.stripe.com/v1/terminal/connection_tokens'

        try:
            resp = requests.post(endpoint, auth=(self.sudo()._get_stripe_secret_key(), ''), timeout=TIMEOUT)
        except requests.exceptions.RequestException:
            _logger.exception("Failed to call stripe_connection_token endpoint")
            raise UserError(_("There are some issues between us and Stripe, try again later."))

        return resp.json()

    def _stripe_calculate_amount(self, amount):
        currency = self.journal_id.currency_id or self.company_id.currency_id
        return round(amount/currency.rounding)

    def stripe_payment_intent(self, amount):
        if not self.env.user.has_group('point_of_sale.group_pos_user'):
            raise AccessError(_("Do not have access to fetch token from Stripe"))

        # For Terminal payments, the 'payment_method_types' parameter must include
        # at least 'card_present' and the 'capture_method' must be set to 'manual'.
        endpoint = 'https://api.stripe.com/v1/payment_intents'
        currency = self.journal_id.currency_id or self.company_id.currency_id

        params = [
            ("currency", currency.name),
            ("amount", self._stripe_calculate_amount(amount)),
            ("payment_method_types[]", "card_present"),
            ("capture_method", "manual"),
        ]

        if currency.name == 'AUD' and self.company_id.country_code == 'AU':
            # See https://stripe.com/docs/terminal/payments/regional?integration-country=AU
            # This parameter overrides "capture_method": "manual" above.
            params.append(("payment_method_options[card_present][capture_method]", "manual_preferred"))
        elif currency.name == 'CAD' and self.company_id.country_code == 'CA':
            params.append(("payment_method_types[]", "interac_present"))

        try:
            data = werkzeug.urls.url_encode(params)
            resp = requests.post(endpoint, data=data, auth=(self.sudo()._get_stripe_secret_key(), ''), timeout=TIMEOUT)
            resp = resp.json()
            redacted_resp = {k: '<redacted in odoo logs>' if k == 'client_secret' else v for k, v in resp.items()}
            _logger.info("Stripe payment intent response: %s", redacted_resp)
        except requests.exceptions.RequestException:
            _logger.exception("Failed to call stripe_payment_intent endpoint")
            raise UserError(_("There are some issues between us and Stripe, try again later."))

        return resp

    @api.model
    def stripe_capture_payment(self, paymentIntentId, amount=None):
        """Captures the payment identified by paymentIntentId.

        :param paymentIntentId: the id of the payment to capture
        :param amount: without this parameter the entire authorized
                       amount is captured. Specifying a larger amount allows
                       overcapturing to support tips.
        """
        if not self.env.user.has_group('point_of_sale.group_pos_user'):
            raise AccessError(_("Do not have access to fetch token from Stripe"))

        endpoint = ('payment_intents/%s/capture') % (werkzeug.urls.url_quote(paymentIntentId))

        data = None
        if amount is not None:
            data = {
                "amount_to_capture": self._stripe_calculate_amount(amount),
            }

        return self.sudo()._get_stripe_payment_provider()._stripe_make_request(endpoint, data)

    def action_stripe_key(self):
        res_id = self._get_stripe_payment_provider().id
        # Redirect
        return {
            'name': _('Stripe'),
            'res_model': 'payment.provider',
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_id': res_id,
        }

```

## File: models\pos_session.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class PosSession(models.Model):
    _inherit = 'pos.session'

    def _loader_params_pos_payment_method(self):
        result = super()._loader_params_pos_payment_method()
        result['search_params']['fields'].append('stripe_serial_number')
        return result

```

## File: models\__init__.py

```python
# coding: utf-8
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import pos_payment_method
from . import pos_session

```

## File: static\src\js\models.js

```javascript
odoo.define('pos_stripe.models', function (require) {
var models = require('point_of_sale.models');
var PaymentStripe = require('pos_stripe.payment');

models.register_payment_method('stripe', PaymentStripe);
});

```

## File: static\src\js\payment_stripe.js

```javascript
/* global StripeTerminal */
odoo.define('pos_stripe.payment', function (require) {
"use strict";

const core = require('web.core');
const rpc = require('web.rpc');
const PaymentInterface = require('point_of_sale.PaymentInterface');
const { Gui } = require('point_of_sale.Gui');

const _t = core._t;

let PaymentStripe = PaymentInterface.extend({
    init: function (pos, payment_method) {
        this._super(...arguments);
        this.createStripeTerminal();
    },

    stripeUnexpectedDisconnect: function () {
      // Include a way to attempt to reconnect to a reader ?
        this._showError(_t('Reader disconnected'));
    },

    stripeFetchConnectionToken: async function () {
        // Do not cache or hardcode the ConnectionToken.
        try {
            let data = await rpc.query({
                model: 'pos.payment.method',
                method: 'stripe_connection_token',
                kwargs: { context: this.pos.env.session.user_context },
            }, {
                silent: true,
            });
            if (data.error) {
                throw data.error;
            }
            return data.secret;
        } catch (error) {
            const message = error.message.code === 200 ? error.message.data.message : error.message.message;
            this._showError(message, 'Fetch Token');
            this.terminal = false;
        };
    },

    discoverReaders: async function () {
        let discoverResult = await this.terminal.discoverReaders({});
        if (discoverResult.error) {
            this._showError(_.str.sprintf(_t('Failed to discover: %s'), discoverResult.error));
        } else if (discoverResult.discoveredReaders.length === 0) {
            this._showError(_t('No available Stripe readers.'));
        } else {
            // Need to stringify all Readers to avoid to put the array into a proxy Object not interpretable
            // for the Stripe SDK
            this.pos.discoveredReaders = JSON.stringify(discoverResult.discoveredReaders);
        }
    },

    checkReader: async function () {
        try {
            if ( !this.terminal ) {
                let createStripeTerminal = this.createStripeTerminal();
                if ( !createStripeTerminal ) {
                    throw _t('Failed to load resource: net::ERR_INTERNET_DISCONNECTED.');
                }
            }
        } catch (error) {
            this._showError(error);
            return false;
        }
        let line = this.pos.get_order().selected_paymentline;
        // Because the reader can only connect to one instance of the SDK at a time.
        // We need the disconnect this reader if we want to use another one
        if (
            this.pos.connectedReader != this.payment_method.stripe_serial_number &&
            this.terminal.getConnectionStatus() == 'connected'
            ) {
            let disconnectResult = await this.terminal.disconnectReader();
            if (disconnectResult.error) {
                this._showError(disconnectResult.error.message, disconnectResult.error.code);
                line.set_payment_status('retry');
                return false;
            } else {
                return await this.connectReader();
            }
        } else if (this.terminal.getConnectionStatus() == 'not_connected') {
            return await this.connectReader();
        } else {
            return true;
        }
    },

    connectReader: async function () {
        let line = this.pos.get_order().selected_paymentline;
        let discoveredReaders = JSON.parse(this.pos.discoveredReaders);
        for (const selectedReader of discoveredReaders) {
            if (selectedReader.serial_number == this.payment_method.stripe_serial_number) {
                try {
                    let connectResult = await this.terminal.connectReader(selectedReader, {fail_if_in_use: true});
                    if (connectResult.error) {
                        throw connectResult;
                    }
                    this.pos.connectedReader = this.payment_method.stripe_serial_number;
                    return true;
                } catch (error) {
                    if (error.error) {
                        this._showError(error.error.message, error.code);
                    } else {
                        this._showError(error);
                    }
                    line.set_payment_status('retry');
                    return false;
                }
            }
        }
        this._showError(_.str.sprintf(
            _t('Stripe readers %s not listed in your account'), 
            this.payment_method.stripe_serial_number
        ));
    },

    _getCapturedCardAndTransactionId: function (processPayment) {
        const charges = processPayment.paymentIntent.charges;
        if (!charges) {
            return [false, false];
        }

        const intentCharge = charges.data[0];
        const processPaymentDetails = intentCharge.payment_method_details;

        if (processPaymentDetails.type === 'interac_present') {
            // Canadian interac payments should not be captured:
            // https://stripe.com/docs/terminal/payments/regional?integration-country=CA#create-a-paymentintent
            return ['interac', intentCharge.id];
        }
        const cardPresentBrand = this.getCardBrandFromPaymentMethodDetails(processPaymentDetails);
        if (cardPresentBrand.includes('eftpos')) {
            // Australian eftpos should not be captured:
            // https://stripe.com/docs/terminal/payments/regional?integration-country=AU
            return [cardPresentBrand, intentCharge.id];
        }

        return [false, false];
    },

    collectPayment: async function (amount) {
        let line = this.pos.get_order().selected_paymentline;
        let clientSecret = await this.fetchPaymentIntentClientSecret(line.payment_method, amount);
        if (!clientSecret) {
            line.set_payment_status('retry');
            return false;
        }
        line.set_payment_status('waitingCard');
        let collectPaymentMethod = await this.terminal.collectPaymentMethod(clientSecret);
        if (collectPaymentMethod.error) {
            this._showError(collectPaymentMethod.error.message, collectPaymentMethod.error.code);
            line.set_payment_status('retry');
            return false;
        } else {
            line.set_payment_status('waitingCapture');
            let processPayment = await this.terminal.processPayment(collectPaymentMethod.paymentIntent);
            line.transaction_id = collectPaymentMethod.paymentIntent.id;
            if (processPayment.error) {
                this._showError(processPayment.error.message, processPayment.error.code);
                line.set_payment_status('retry');
                return false;
            } else if (processPayment.paymentIntent) {
                line.set_payment_status('waitingCapture');

                const [captured_card_type, captured_transaction_id] = this._getCapturedCardAndTransactionId(processPayment);
                if (captured_card_type && captured_transaction_id) {
                    line.card_type = captured_card_type;
                    line.transaction_id = captured_transaction_id;
                } else {
                    await this.captureAfterPayment(processPayment, line);
                }

                line.set_payment_status('done');
                return true;
            }
        }
    },

    createStripeTerminal: function () {
        try {
            this.terminal = StripeTerminal.create({
                onFetchConnectionToken: this.stripeFetchConnectionToken.bind(this),
                onUnexpectedReaderDisconnect: this.stripeUnexpectedDisconnect.bind(this),
            });
            this.discoverReaders();
            return true;
        } catch (error) {
            this._showError(_t('Failed to load resource: net::ERR_INTERNET_DISCONNECTED.'), error);
            this.terminal = false;
            return false;
        }
    },

    getCardBrandFromPaymentMethodDetails(paymentMethodDetails) {
        // Both `card_present` and `interac_present` are "nullable" so we need to check for their existence, see:
        // https://docs.stripe.com/api/charges/object#charge_object-payment_method_details-card_present
        // https://docs.stripe.com/api/charges/object#charge_object-payment_method_details-interac_present
        // In Canada `card_present` might not be present, but `interac_present` will be 
        if (paymentMethodDetails.card_present) {
            return paymentMethodDetails.card_present.brand;
        } 
        if (paymentMethodDetails.interac_present) {
            return paymentMethodDetails.interac_present.brand;
        }
        return "";
    },

    captureAfterPayment: async function (processPayment, line) {
        let capturePayment = await this.capturePayment(processPayment.paymentIntent.id);
        if (capturePayment.charges)
            line.card_type = this.getCardBrandFromPaymentMethodDetails(capturePayment.charges.data[0].payment_method_details);
        line.transaction_id = capturePayment.id;
    },

    capturePayment: async function (paymentIntentId) {
        try {
            let data = await rpc.query({
                model: 'pos.payment.method',
                method: 'stripe_capture_payment',
                args: [paymentIntentId],
                kwargs: { context: this.pos.env.session.user_context },
            }, {
                silent: true,
            });
            if (data.error) {
                throw data.error;
            }
            return data;
        } catch (error) {
            const message = error.message.code === 200 ? error.message.data.message : error.message.message;
            this._showError(message, 'Capture Payment');
            return false;
        };
    },

    fetchPaymentIntentClientSecret: async function (payment_method, amount) {
        try {
            let data = await rpc.query({
                model: 'pos.payment.method',
                method: 'stripe_payment_intent',
                args: [[payment_method.id], amount],
                kwargs: { context: this.pos.env.session.user_context },
            }, {
                silent: true,
            });
            if (data.error) {
                throw data.error;
            }
            return data.client_secret;
        } catch (error) {
            const message = error.message.code === 200 ? error.message.data.message : error.message.message || error.message;
            this._showError(message, 'Fetch Secret');
            return false;
        };
    },

    send_payment_request: async function (cid) {
        /**
         * Override
         */
        await this._super.apply(this, arguments);
        let line = this.pos.get_order().selected_paymentline;
        line.set_payment_status('waiting');
        try {
            if (await this.checkReader()) {
                return await this.collectPayment(line.amount);
            }
        } catch (error) {
            this._showError(error);
            return false;
        }
    },

    send_payment_cancel: async function (order, cid) {
        /**
         * Override
         */
        this._super.apply(this, arguments);
        let line = this.pos.get_order().selected_paymentline;
        let stripeCancel = await this.stripeCancel();
        if (stripeCancel) {
            line.set_payment_status('retry');
            return true;
        }
    },

    stripeCancel: async function () {
        if (!this.terminal) {
            return true;
        } else if (this.terminal.getConnectionStatus() != 'connected') {
            this._showError(_t('Payment canceled because not reader connected'));
            return true;
        } else {
            let cancelCollectPaymentMethod = await this.terminal.cancelCollectPaymentMethod();
            if (cancelCollectPaymentMethod.error) {
                this._showError(cancelCollectPaymentMethod.error.message, cancelCollectPaymentMethod.error.code);
            }
            return true;
        }
    },

    // private methods

    _showError: function (msg, title) {
        if (!title) {
            title =  _t('Stripe Error');
        }
        Gui.showPopup('ErrorPopup',{
            'title': title,
            'body': msg,
        });
    },
});

return PaymentStripe;
});

```

## File: views\assets_stripe.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="stripe" inherit_id="point_of_sale.assets_common">
        <xpath expr="." position="inside">
            <!-- As the following link does not end with '.js', it's not loaded when
                 placed in __manifest__.py. The following declaration fix this problem -->
            <script src="https://js.stripe.com/terminal/v1/"></script>
        </xpath>
    </template>

</odoo>

```

## File: views\pos_payment_method_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="pos_payment_method_view_form_inherit_pos_stripe" model="ir.ui.view">
        <field name="name">pos.payment.method.form.inherit.stripe</field>
        <field name="model">pos.payment.method</field>
        <field name="inherit_id" ref="point_of_sale.pos_payment_method_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='use_payment_terminal']" position="after">
                <!-- Stripe -->
                <field name="stripe_serial_number" attrs="{'invisible': [('use_payment_terminal', '!=', 'stripe')], 'required': [('use_payment_terminal', '=', 'stripe')]}"/>
                <div colspan="2" class="mt16" attrs="{'invisible': [('use_payment_terminal', '!=', 'stripe')], 'required': [('use_payment_terminal', '=', 'stripe')]}">
                    <button name="action_stripe_key" type="object" icon="fa-arrow-right" string="Don't forget to complete Stripe connect before using this payment method." class="btn-link"/>
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

