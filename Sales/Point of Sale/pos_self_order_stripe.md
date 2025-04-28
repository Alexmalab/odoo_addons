# Odoo Module: pos_self_order_stripe

Category: Sales/Point Of Sale

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-

from . import models
from . import controllers

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
{
    "name": "POS Self Order Stripe",
    "summary": "Addon for the Self Order App that allows customers to pay by Stripe.",
    "category": "Sales/Point Of Sale",
    "depends": ["pos_stripe", "pos_self_order"],
    "auto_install": True,
    'data': [
        'views/assets_stripe.xml',
    ],
    'assets': {
        'pos_self_order.assets': [
            'pos_self_order_stripe/static/**/*',
        ],
    },
    "license": "LGPL-3",
}

```

## File: controllers\orders.py

```python
# -*- coding: utf-8 -*-
from odoo import http, fields
from odoo.http import request
from odoo.tools import float_is_zero
from odoo.addons.pos_self_order.controllers.orders import PosSelfOrderController
from werkzeug.exceptions import Unauthorized

class PosSelfOrderControllerStripe(PosSelfOrderController):
    @http.route("/pos-self-order/stripe-connection-token/", auth="public", type="json", website=True)
    def get_stripe_creditentials(self, access_token, payment_method_id):
        # stripe_connection_token
        pos_config, _ = self._verify_authorization(access_token, "", False)
        payment_method = pos_config.payment_method_ids.filtered(lambda p: p.id == payment_method_id)
        return payment_method.stripe_connection_token()

    @http.route("/pos-self-order/stripe-capture-payment/", auth="public", type="json", website=True)
    def stripe_capture_payment(self, access_token, order_access_token, payment_intent_id, payment_method_id):
        pos_config, _ = self._verify_authorization(access_token, "", False)
        stripe_confirmation = pos_config.env['pos.payment.method'].stripe_capture_payment(payment_intent_id)
        order = pos_config.env['pos.order'].search([('access_token', '=', order_access_token), ('config_id', '=', pos_config.id)])

        if not order:
            raise Unauthorized()

        payment_method = pos_config.payment_method_ids.filtered(lambda p: p.id == payment_method_id)
        stripe_order_amount = payment_method._stripe_calculate_amount(order.amount_total)

        if float_is_zero(stripe_order_amount - stripe_confirmation['amount'], precision_rounding=pos_config.currency_id.rounding) and stripe_confirmation['status'] == 'succeeded':
            transaction_id = stripe_confirmation['id']
            payment_result = stripe_confirmation['status']

            order.add_payment({
                'amount': order.amount_total,
                'payment_date': fields.Datetime.now(),
                'payment_method_id': payment_method.id,
                'card_type': False,
                'cardholder_name': '',
                'transaction_id': transaction_id,
                'payment_status': payment_result,
                'ticket': '',
                'pos_order_id': order.id
            })

            order.action_pos_order_paid()

            if order.config_id.self_ordering_mode == 'kiosk':
                request.env['bus.bus']._sendone(f'pos_config-{order.config_id.access_token}', 'PAYMENT_STATUS', {
                    'payment_result': 'Success',
                    'order': order._export_for_self_order(),
                })
        else:
            request.env['bus.bus']._sendone(f'pos_config-{order.config_id.access_token}', 'PAYMENT_STATUS', {
                'payment_result': 'fail',
                'order': order._export_for_self_order(),
            })

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
from . import orders

```

## File: models\pos_payment_method.py

```python
from odoo import models


class PosPaymentMethod(models.Model):
    _inherit = "pos.payment.method"

    def _payment_request_from_kiosk(self, order):
        if self.use_payment_terminal != 'stripe':
            return super()._payment_request_from_kiosk(order)
        else:
            return self.stripe_payment_intent(order.amount_total)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import pos_payment_method

```

## File: static\src\app\self_order_service.js

```javascript
/** @odoo-module */
import { patch } from "@web/core/utils/patch";
import { SelfOrder } from "@pos_self_order/app/self_order_service";
import { Stripe, StripeError } from "@pos_self_order_stripe/app/stripe";

patch(SelfOrder.prototype, {
    async setup() {
        await super.setup(...arguments);
        this.stripeState = "not_connected";

        const stripePaymentMethod = this.pos_payment_methods.find(
            (p) => p.use_payment_terminal === "stripe"
        );

        if (stripePaymentMethod) {
            this.stripe = new Stripe(
                this.env,
                stripePaymentMethod,
                this.access_token,
                this.pos_config_id,
                this.handleStripeError.bind(this),
                this.handleReaderConnection.bind(this)
            );
        }
    },
    handleReaderConnection(state) {
        this.stripeState = state.status;
    },
    handleStripeError(error) {
        this.paymentError = true;
        this.handleErrorNotification(error);
    },
    handleErrorNotification(error) {
        let message = "";

        if (error.code) {
            message = `Error: ${error.code}`;
        } else if (error instanceof StripeError) {
            message = `Stripe: ${error.message}`;
        } else {
            super.handleErrorNotification(...arguments);
            return;
        }

        this.notification.add(message, {
            type: "danger",
        });
    },
});

```

## File: static\src\app\stripe.js

```javascript
/** @odoo-module **/
/* global StripeTerminal */

export class StripeError extends Error {}

export class Stripe {
    constructor(...args) {
        this.setup(...args);
    }

    setup(
        env,
        stripePaymentMethod,
        access_token,
        pos_config_id,
        errorCallback,
        handleReaderConnection
    ) {
        this.env = env;
        this.terminal = null;
        this.access_token = access_token;
        this.stripePaymentMethod = stripePaymentMethod;
        this.pos_config_id = pos_config_id;
        this.errorCallback = errorCallback;
        this.handleReaderConnection = handleReaderConnection;

        this.createTerminal();
    }

    get connectionStatus() {
        return this.terminal.getConnectionStatus();
    }

    createTerminal() {
        this.terminal = StripeTerminal.create({
            onFetchConnectionToken: this.getBackendConnectionToken.bind(this),
            onConnectionStatusChange: this.handleReaderConnection.bind(this),
            onUnexpectedReaderDisconnect: () => {
                this.handleReaderConnection({
                    status: "not_connected",
                });
            },
        });
    }

    async startPayment(order) {
        try {
            const result = await this.env.services.rpc(
                `/kiosk/payment/${this.pos_config_id}/kiosk`,
                {
                    order: order,
                    access_token: this.access_token,
                    payment_method_id: this.stripePaymentMethod.id,
                }
            );
            const paymentStatus = result.payment_status;
            const savedOrder = result.order;
            await this.connectReader();
            const clientSecret = paymentStatus.client_secret;
            const paymentMethod = await this.collectPaymentMethod(clientSecret);
            const processPayment = await this.processPayment(paymentMethod.paymentIntent);
            await this.capturePayment(processPayment.paymentIntent.id, savedOrder);
        } catch (error) {
            this.errorCallback(error);
        }
    }

    async processPayment(paymentIntent) {
        const result = await this.terminal.processPayment(paymentIntent);

        if (result.error) {
            throw new StripeError(result.error.code);
        }

        return result;
    }

    async getBackendConnectionToken() {
        const data = await this.env.services.rpc("/pos-self-order/stripe-connection-token", {
            access_token: this.access_token,
            payment_method_id: this.stripePaymentMethod.id,
        });

        return data.secret;
    }

    async capturePayment(paymentIntentId, order) {
        return await this.env.services.rpc("/pos-self-order/stripe-capture-payment", {
            access_token: this.access_token,
            order_access_token: order.access_token,
            payment_intent_id: paymentIntentId,
            payment_method_id: this.stripePaymentMethod.id,
        });
    }

    async discoverReaders() {
        const result = await this.terminal.discoverReaders({
            allowCustomerCancel: true,
        });

        if (result.error) {
            throw new StripeError(result.error.code);
        }

        return result;
    }

    async connectReader() {
        if (this.connectionStatus !== "not_connected") {
            return;
        }

        const discoverReaders = await this.discoverReaders();
        const discoveredReaders = discoverReaders.discoveredReaders;
        const findLinkedReader = discoveredReaders.find(
            (reader) => reader.serial_number == this.stripePaymentMethod.stripe_serial_number
        );

        const result = await this.terminal.connectReader(findLinkedReader, {
            fail_if_in_use: true,
        });

        if (result.error) {
            throw new StripeError(result.error.code);
        }

        return result;
    }

    async collectPaymentMethod(clientSecret) {
        const result = await this.terminal.collectPaymentMethod(clientSecret);

        if (result.error) {
            throw new StripeError(result.error.code);
        }

        return result;
    }
}

```

## File: static\src\app\pages\payment_page\payment_page.js

```javascript
/** @odoo-module */

import { patch } from "@web/core/utils/patch";
import { PaymentPage } from "@pos_self_order/app/pages/payment_page/payment_page";

patch(PaymentPage.prototype, {
    async startPayment() {
        this.selfOrder.paymentError = false;
        const paymentMethod = this.selfOrder.pos_payment_methods.find(
            (p) => p.id === this.state.paymentMethodId
        );

        if (paymentMethod.use_payment_terminal === "stripe") {
            await this.selfOrder.stripe.startPayment(this.selfOrder.currentOrder);
        } else {
            await super.startPayment(...arguments);
        }
    },
});

```

## File: views\assets_stripe.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="stripe" inherit_id="pos_self_order.index">
        <xpath expr="//head" position="inside">
            <!-- As the following link does not end with '.js', it's not loaded when
                 placed in __manifest__.py. The following declaration fix this problem -->
            <script src="https://js.stripe.com/terminal/v1/"></script>
        </xpath>
    </template>
</odoo>

```

