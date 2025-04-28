# Odoo Module: pos_self_order_razorpay

Category: Sales/Point Of Sale

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models
from . import controllers

```

## File: __manifest__.py

```python
{
    "name": "POS Self Order Razorpay",
    "version": "1.0",
    "summary": "Addon for the Self Order App that allows customers to pay by Razorpay POS Terminal.",
    "category": "Sales/Point Of Sale",
    "depends": ["pos_razorpay", "pos_self_order"],
    "auto_install": True,
    'assets': {
        'pos_self_order.assets': [
            'pos_self_order_razorpay/static/**/*',
        ],
    },
    "license": "LGPL-3",
}

```

## File: controllers\orders.py

```python
from odoo import http
from odoo.addons.pos_self_order.controllers.orders import PosSelfOrderController
from werkzeug.exceptions import Unauthorized


class PosSelfOrderControllerRazorpay(PosSelfOrderController):
    @http.route("/pos-self-order/razorpay-fetch-payment-status/", auth="public", type="json", website=True)
    def razorpay_payment_status(self, access_token, order_id, payment_data, payment_method_id):
        pos_config = self._verify_pos_config(access_token)
        order = pos_config.env['pos.order'].search([
            ('id', '=', order_id), ('config_id', '=', pos_config.id)
        ], limit=1)

        if not order:
            raise Unauthorized()

        payment_method = pos_config.env['pos.payment.method'].browse(payment_method_id)
        razorpay_status_response = payment_method.razorpay_fetch_payment_status(payment_data)
        payment_status = razorpay_status_response.get('status')
        if payment_status == "AUTHORIZED":
            order.add_payment({
                'amount': order.amount_total,
                'payment_method_id': payment_method.id,
                'card_type': razorpay_status_response.get('paymentCardType'),
                'cardholder_name': razorpay_status_response.get('nameOnCard'),
                'transaction_id': razorpay_status_response.get('txnId'),
                'payment_status': razorpay_status_response.get('status'),
                'pos_order_id': order.id,
                'payment_method_authcode': razorpay_status_response.get('authCode'),
                'card_brand': razorpay_status_response.get('paymentCardBrand'),
                'payment_method_issuer_bank': razorpay_status_response.get('acquirerCode'),
                'card_no': razorpay_status_response.get('cardLastFourDigit'),
                'payment_method_payment_mode': razorpay_status_response.get('paymentMode'),
                'payment_ref_no': razorpay_status_response.get('externalRefNumber'),
                'razorpay_reverse_ref_no': razorpay_status_response.get('reverseReferenceNumber'),
            })

            order.action_pos_order_paid()

            if order.config_id.self_ordering_mode == 'kiosk':
                self.call_bus_service(order, payment_result='Success')
        elif payment_status == "FAILED" or not payment_status:
            self.call_bus_service(order, payment_result='fail')
        return razorpay_status_response

    @http.route("/pos-self-order/razorpay-cancel-transaction/", auth="public", type="json", website=True)
    def razorpay_cancel_status(self, access_token, order_id, payment_data, payment_method_id):
        pos_config = self._verify_pos_config(access_token)
        order = pos_config.env['pos.order'].search([
            ('id', '=', order_id), ('config_id', '=', pos_config.id)
        ], limit=1)

        if not order:
            raise Unauthorized()

        payment_method = pos_config.env['pos.payment.method'].browse(payment_method_id)
        razorpay_cancel_response = payment_method.razorpay_cancel_payment_request(payment_data)
        cancel_status = razorpay_cancel_response.get('status')
        if cancel_status:
            self.call_bus_service(order, payment_result='fail')
        return razorpay_cancel_response

    def call_bus_service(self, order, payment_result):
        order.config_id._notify('PAYMENT_STATUS', {
            'payment_result': payment_result,
            'data': {
                'pos.order': order.read(order._load_pos_self_data_fields(order.config_id.id), load=False),
                'pos.order.line': order.lines.read(order._load_pos_self_data_fields(order.config_id.id), load=False),
            }
        })

```

## File: controllers\__init__.py

```python
from . import orders

```

## File: models\pos_payment_method.py

```python
import uuid
from odoo import models, api
from odoo.osv import expression


class PosPaymentMethod(models.Model):
    _inherit = 'pos.payment.method'

    def _payment_request_from_kiosk(self, order):
        if self.use_payment_terminal != 'razorpay':
            return super()._payment_request_from_kiosk(order)
        reference_prefix = order.config_id.name.replace(' ', '')
        data = {
            'amount': order.amount_total,
            'referenceId': f'{reference_prefix}/Order/{order.id}/{uuid.uuid4().hex}',
        }
        return self.razorpay_make_payment_request(data)

    @api.model
    def _load_pos_self_data_domain(self, data):
        domain = super()._load_pos_self_data_domain(data)
        if data['pos.config']['data'][0]['self_ordering_mode'] == 'kiosk':
            domain = expression.OR([[('use_payment_terminal', '=', 'razorpay')], domain])
        return domain

```

## File: models\__init__.py

```python
from . import pos_payment_method

```

## File: static\src\app\razorpay.js

```javascript
import { rpc } from "@web/core/network/rpc";

const REQUEST_TIMEOUT = 10000;

export class RazorpayError extends Error {}

export class Razorpay {
    constructor(...args) {
        this.setup(...args);
    }

    setup(env, razorpayPaymentMethod, access_token, pos_config, errorCallback) {
        this.env = env;
        this.access_token = access_token;
        this.razorpayPaymentMethod = razorpayPaymentMethod;
        this.pos_config = pos_config;
        this.errorCallback = errorCallback;
        this.savedOrder = false;
        this.pollTimeout = null;
        this.inactivityTimeout = null;
        this.queued = false;
        this.payment_stopped = false;
    }

    handleRazorpayResponse(response) {
        if (response?.error) {
            this.payment_stopped
                ? this.errorCallback(new RazorpayError("Transaction canceled due to inactivity"))
                : this.errorCallback(new RazorpayError(response.error));
            this.removePaymentHandler(["p2pRequestId"]);
            return false;
        }
        localStorage.setItem("p2pRequestId", response?.p2pRequestId);
        return true;
    }

    async cancelPayment(order) {
        const data = { p2pRequestId: localStorage.getItem("p2pRequestId") };
        try {
            const cancel_response = await rpc("/pos-self-order/razorpay-cancel-transaction/", {
                access_token: this.access_token,
                order_id: order.id,
                payment_data: data,
                payment_method_id: this.razorpayPaymentMethod.id,
            });
            if (cancel_response) {
                if (cancel_response?.errorMessage) {
                    this.errorCallback(cancel_response.errorMessage, "warning");
                    return true;
                }
                return this.handleRazorpayResponse(cancel_response);
            }
        } catch (error) {
            this.errorCallback(error);
            return false;
        }
    }

    async startPayment(order) {
        const call_razorpay = await this.processPayment(order);
        if (call_razorpay) {
            await this.paymentPolling(this.savedOrder);
        }
    }

    async processPayment(order) {
        try {
            const initial_response = await rpc(`/kiosk/payment/${this.pos_config.id}/kiosk`, {
                order: order.serialize({ orm: true }),
                access_token: this.access_token,
                payment_method_id: this.razorpayPaymentMethod.id,
            });
            if (initial_response) {
                this.savedOrder = initial_response.order[0];
                return this.handleRazorpayResponse(initial_response.payment_status);
            }
        } catch (error) {
            this.errorCallback(error);
            return false;
        }
    }

    /**
     * Polling
     * This method calls and handles the razorpay payment status
     * calls every 10 sec until payment status not found.
     */
    async paymentPolling(order) {
        const data = { p2pRequestId: localStorage.getItem("p2pRequestId") };
        this.stopInactivePayment().then(() => (this.payment_stopped = true));
        const fetchPaymentStatus = async () => {
            try {
                // Within 90 seconds, inactivity will result in transaction cancellation and payment termination.
                if (this.payment_stopped) {
                    await this.cancelPayment(order);
                    return false;
                }

                const polling_response = await rpc(
                    "/pos-self-order/razorpay-fetch-payment-status/",
                    {
                        access_token: this.access_token,
                        order_id: order.id,
                        payment_data: data,
                        payment_method_id: this.razorpayPaymentMethod.id,
                    }
                );
                if (polling_response?.error) {
                    this.handleRazorpayResponse(polling_response);
                    return false;
                }

                const result_code = polling_response?.status;

                if (result_code === "QUEUED" && this.queued === false) {
                    await this.cancelPayment(order);
                    await this.startPayment(order);
                }
                if (result_code === "AUTHORIZED") {
                    this.removePaymentHandler(["p2pRequestId"]);
                    return true;
                } else {
                    // clearing previous timeout before setting a new one
                    clearTimeout(this.pollTimeout);
                    this.pollTimeout = setTimeout(fetchPaymentStatus, REQUEST_TIMEOUT);
                }
            } catch (error) {
                this.errorCallback(error);
            }
        };
        await fetchPaymentStatus();
    }

    stopInactivePayment() {
        return new Promise((resolve) => (this.inactivityTimeout = setTimeout(resolve, 90000)));
    }

    removePaymentHandler(payment_data) {
        payment_data.forEach((data) => {
            localStorage.removeItem(data);
        });
        clearTimeout(this.pollTimeout);
        clearTimeout(this.inactivityTimeout);
        this.queued = this.payment_stopped = false;
    }
}

```

## File: static\src\app\self_order_service.js

```javascript
import { patch } from "@web/core/utils/patch";
import { SelfOrder } from "@pos_self_order/app/self_order_service";
import { Razorpay, RazorpayError } from "@pos_self_order_razorpay/app/razorpay";

patch(SelfOrder.prototype, {
    async setup() {
        await super.setup(...arguments);

        const razorpayPaymentMethod = this.models["pos.payment.method"].find(
            (p) => p.use_payment_terminal === "razorpay"
        );

        if (razorpayPaymentMethod) {
            this.razorpay = new Razorpay(
                this.env,
                razorpayPaymentMethod,
                this.access_token,
                this.config,
                this.handleRazorpayError.bind(this)
            );
        }
    },

    filterPaymentMethods(pms) {
        const pm = super.filterPaymentMethods(...arguments);
        const razorpay_pm = pms.filter((rec) => rec.use_payment_terminal === "razorpay");
        return [...new Set([...pm, ...razorpay_pm])];
    },

    handleRazorpayError(error, type) {
        this.paymentError = true;
        this.handleErrorNotification(error, type);
    },

    handleErrorNotification(error, type = "danger") {
        let errorMessage = "";
        if (error instanceof RazorpayError) {
            errorMessage = `Razorpay POS: ${error.message}`;
            this.notification.add(errorMessage, {
                type: type,
            });
        } else {
            super.handleErrorNotification(...arguments);
        }
    },
});

```

## File: static\src\app\pages\payment_page\payment_page.js

```javascript
import { patch } from "@web/core/utils/patch";
import { PaymentPage } from "@pos_self_order/app/pages/payment_page/payment_page";

patch(PaymentPage.prototype, {
    async startPayment() {
        this.selfOrder.paymentError = false;
        const paymentMethod = this.selfOrder.models["pos.payment.method"].find(
            (p) => p.id === this.state.paymentMethodId
        );

        if (paymentMethod.use_payment_terminal === "razorpay") {
            await this.selfOrder.razorpay.startPayment(this.selfOrder.currentOrder);
        } else {
            await super.startPayment(...arguments);
        }
    },
});

```

