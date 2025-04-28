# Odoo Module: pos_mercado_pago

Category: Sales/Point of Sale

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models
from . import controllers

```

## File: __manifest__.py

```python
{
    'name': 'POS Mercado Pago',
    'version': '1.0',
    'category': 'Sales/Point of Sale',
    'sequence': 6,
    'summary': 'Integrate your POS with the Mercado Pago Smart Point terminal',
    'data': [
        'views/pos_payment_method_views.xml',
    ],
    'depends': ['point_of_sale'],
    'installable': True,
    'assets': {
        'point_of_sale._assets_pos': [
            'pos_mercado_pago/static/**/*',
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
import re

from odoo import http
from odoo.http import request

_logger = logging.getLogger(__name__)


class PosMercadoPagoWebhook(http.Controller):
    @http.route('/pos_mercado_pago/notification', methods=['POST'], type="http", auth="none", csrf=False)
    def notification(self):
        """ Process the notification sent by Mercado Pago

        Notification format is always json
        """
        # Check for mandatory keys in header
        x_request_id = request.httprequest.headers.get('X-Request-Id')
        if not x_request_id:
            _logger.warning('POST message received with no X-Request-Id in header')
            return http.Response(status=400)

        x_signature = request.httprequest.headers.get('X-Signature')
        if not x_signature:
            _logger.warning('POST message received with no X-Signature in header')
            return http.Response(status=400)

        ts_m = re.search(r"ts=(\d+)", x_signature)
        v1_m = re.search(r"v1=([a-f0-9]+)", x_signature)
        ts = ts_m.group(1) if ts_m else None
        v1 = v1_m.group(1) if v1_m else None
        if not ts or not v1:
            _logger.warning('Webhook bad X-Signature, ts: %s, v1: %s', ts, v1)
            return http.Response(status=400)

        # Check for payload
        data = request.httprequest.get_json(silent=True)
        if not data:
            _logger.warning('POST message received with no data')
            return http.Response(status=400)

        # If and only if this webhook is related with a payment intend (see payment_mercado_pago.js)
        # then the field data['additional_info']['external_reference'] contains a string
        # formated like "XXX_YYY_ZZZ" where "XXX" is the session_id, "YYY" is the payment_method_id,
        # and ZZZ is the pos_reference/uid for customer identification (Format ZZZZ-ZZZZ-ZZZZ)
        external_reference = data.get('additional_info', {}).get('external_reference')

        if not external_reference or not re.fullmatch(r'\d+_\d+[_\d-]*', external_reference):
            _logger.warning('POST message received with no or malformed "external_reference" key: %s', external_reference)
            return http.Response(status=400)

        session_id, payment_method_id, _ = external_reference.split('_')

        pos_session_sudo = request.env['pos.session'].sudo().browse(int(session_id))
        if not pos_session_sudo or pos_session_sudo.state != 'opened':
            _logger.error("Invalid session id: %s", session_id)
            # This error is not related with Mercado Pago, simply acknowledge Mercado Pago message
            return http.Response('OK', status=200)

        payment_method_sudo = pos_session_sudo.config_id.payment_method_ids.filtered(lambda p: p.id == int(payment_method_id))
        if not payment_method_sudo or payment_method_sudo.use_payment_terminal != 'mercado_pago':
            _logger.error("Invalid payment method id: %s", payment_method_id)
            # This error is not related with Mercado Pago, simply acknowledge Mercado Pago message
            return http.Response('OK', status=200)

        # We have to check if this comes from Mercado Pago with the secret key
        secret_key = payment_method_sudo.mp_webhook_secret_key
        signed_template = f"id:{data['id']};request-id:{x_request_id};ts:{ts};"
        cyphed_signature = hmac.new(secret_key.encode(), signed_template.encode(), hashlib.sha256).hexdigest()
        if not hmac.compare_digest(cyphed_signature, v1):
            _logger.error('Webhook authenticating failure, ts: %s, v1: %s', ts, v1)
            return http.Response(status=401)

        _logger.debug('Webhook authenticated, POST message: %s', data)

        # Notify the frontend that we received a message from Mercado Pago
        request.env['bus.bus']._sendone(pos_session_sudo._get_bus_channel_name(), 'MERCADO_PAGO_LATEST_MESSAGE', {})

        # Acknowledge Mercado Pago message
        return http.Response('OK', status=200)

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\neutralize.sql

```sql
UPDATE pos_payment_method
   SET mp_bearer_token = 'dummy_value',
       mp_webhook_secret_key = 'dummy_value'
   WHERE mp_bearer_token IS NOT NULL;

```

## File: models\mercado_pago_pos_request.py

```python
import logging
import requests

_logger = logging.getLogger(__name__)


REQUEST_TIMEOUT = 10
MERCADO_PAGO_API_ENDPOINT = 'https://api.mercadopago.com'


class MercadoPagoPosRequest:
    def __init__(self, mp_bearer_token):
        self.mercado_pago_bearer_token = mp_bearer_token

    def call_mercado_pago(self, method, endpoint, payload):
        """ Make a request to Mercado Pago POS API.

        :param method: "GET", "POST", ...
        :param endpoint: The endpoint to be reached by the request.
        :param payload: The payload of the request.
        :return The JSON-formatted content of the response.

        Note: The platform id below is not secret, and is just used to
        quantify the amount of Odoo users on Mercado's backend.
        """
        endpoint = MERCADO_PAGO_API_ENDPOINT + endpoint
        header = {
            'Authorization': f"Bearer {self.mercado_pago_bearer_token}",
            'X-platform-id': "dev_cdf1cfac242111ef9fdebe8d845d0987"
        }
        try:
            response = requests.request(method, endpoint, headers=header, json=payload, timeout=REQUEST_TIMEOUT)
            return response.json()
        except requests.exceptions.RequestException as error:
            _logger.warning("Cannot connect with Mercado Pago POS. Error: %s", error)
            return {'errorMessage': str(error)}
        except ValueError as error:
            _logger.warning("Cannot decode response json. Error: %s", error)
            return {'errorMessage': f"Cannot decode Mercado Pago POS response. Error: {error}"}

```

## File: models\pos_payment_method.py

```python
import logging

from odoo import fields, models, _
from odoo.exceptions import AccessError, UserError

from .mercado_pago_pos_request import MercadoPagoPosRequest

_logger = logging.getLogger(__name__)


class PosPaymentMethod(models.Model):
    _inherit = 'pos.payment.method'

    mp_bearer_token = fields.Char(
        string="Production user token",
        help='Mercado Pago customer production user token: https://www.mercadopago.com.mx/developers/en/reference',
        groups="point_of_sale.group_pos_manager")
    mp_webhook_secret_key = fields.Char(
        string="Production secret key",
        help='Mercado Pago production secret key from integration application: https://www.mercadopago.com.mx/developers/panel/app',
        groups="point_of_sale.group_pos_manager")
    mp_id_point_smart = fields.Char(
        string="Terminal S/N",
        help="Enter your Point Smart terminal serial number written on the back of your terminal (after the S/N:)")
    mp_id_point_smart_complet = fields.Char()

    def _get_payment_terminal_selection(self):
        return super()._get_payment_terminal_selection() + [('mercado_pago', 'Mercado Pago')]

    def _check_access(self):
        if not self.env.user.has_group('point_of_sale.group_pos_user'):
            raise AccessError(_("Do not have access to fetch token from Mercado Pago"))

    def force_pdv(self):
        """
        Triggered in debug mode when the user wants to force the "PDV" mode.
        It calls the Mercado Pago API to set the terminal mode to "PDV".
        """
        self._check_access()

        mercado_pago = MercadoPagoPosRequest(self.sudo().mp_bearer_token)
        _logger.info('Calling Mercado Pago to force the terminal mode to "PDV"')

        mode = {"operating_mode": "PDV"}
        resp = mercado_pago.call_mercado_pago("patch", f"/point/integration-api/devices/{self.mp_id_point_smart_complet}", mode)
        if resp.get("operating_mode") != "PDV":
            raise UserError(_("Unexpected Mercado Pago response: %s", resp))
        _logger.debug("Successfully set the terminal mode to 'PDV'.")
        return None

    def mp_payment_intent_create(self, infos):
        """
        Called from frontend for creating a payment intent in Mercado Pago
        """
        self._check_access()

        mercado_pago = MercadoPagoPosRequest(self.sudo().mp_bearer_token)
        # Call Mercado Pago for payment intend creation
        resp = mercado_pago.call_mercado_pago("post", f"/point/integration-api/devices/{self.mp_id_point_smart_complet}/payment-intents", infos)
        _logger.debug("mp_payment_intent_create(), response from Mercado Pago: %s", resp)
        return resp

    def mp_payment_intent_get(self, payment_intent_id):
        """
        Called from frontend to get the last payment intend from Mercado Pago
        """
        self._check_access()

        mercado_pago = MercadoPagoPosRequest(self.sudo().mp_bearer_token)
        # Call Mercado Pago for payment intend status
        resp = mercado_pago.call_mercado_pago("get", f"/point/integration-api/payment-intents/{payment_intent_id}", {})
        _logger.debug("mp_payment_intent_get(), response from Mercado Pago: %s", resp)
        return resp

    def mp_get_payment_status(self, payment_id):
        """
        Called from frontend to get the payment status from Mercado Pago
        """
        self._check_access()

        mercado_pago = MercadoPagoPosRequest(self.sudo().mp_bearer_token)

        resp = mercado_pago.call_mercado_pago("get", f"/v1/payments/{payment_id}", {})
        _logger.debug("mp_get_payment_status(), response from Mercado Pago: %s", resp)
        return resp

    def mp_payment_intent_cancel(self, payment_intent_id):
        """
        Called from frontend to cancel a payment intent in Mercado Pago
        """
        self._check_access()

        mercado_pago = MercadoPagoPosRequest(self.sudo().mp_bearer_token)
        # Call Mercado Pago for payment intend cancelation
        resp = mercado_pago.call_mercado_pago("delete", f"/point/integration-api/devices/{self.mp_id_point_smart_complet}/payment-intents/{payment_intent_id}", {})
        _logger.debug("mp_payment_intent_cancel(), response from Mercado Pago: %s", resp)
        return resp

    def _find_terminal(self, token, point_smart):
        mercado_pago = MercadoPagoPosRequest(token)
        data = mercado_pago.call_mercado_pago("get", "/point/integration-api/devices", {})
        if 'devices' in data:
            # Search for a device id that contains the serial number entered by the user
            found_device = next((device for device in data['devices'] if point_smart in device['id']), None)

            if not found_device:
                raise UserError(_("The terminal serial number is not registered on Mercado Pago"))

            return found_device.get('id', '')
        else:
            raise UserError(_("Please verify your production user token as it was rejected"))

    def write(self, vals):
        records = super().write(vals)

        if 'mp_id_point_smart' in vals or 'mp_bearer_token' in vals:
            self.mp_id_point_smart_complet = self._find_terminal(self.mp_bearer_token, self.mp_id_point_smart)

        return records

    def create(self, vals):
        records = super().create(vals)

        for record in records:
            if record.mp_bearer_token:
                record.mp_id_point_smart_complet = record._find_terminal(record.mp_bearer_token, record.mp_id_point_smart)

        return records

```

## File: models\pos_session.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models


class PosSession(models.Model):
    _inherit = 'pos.session'

    def _loader_params_pos_payment_method(self):
        result = super()._loader_params_pos_payment_method()
        result['search_params']['fields'].append('mp_id_point_smart')
        return result

```

## File: models\__init__.py

```python
from . import mercado_pago_pos_request
from . import pos_payment_method
from . import pos_session

```

## File: static\src\app\payment_mercado_pago.js

```javascript
/** @odoo-module */
import { _t } from "@web/core/l10n/translation";
import { PaymentInterface } from "@point_of_sale/app/payment/payment_interface";
import { ErrorPopup } from "@point_of_sale/app/errors/popups/error_popup";

export class PaymentMercadoPago extends PaymentInterface {
    async create_payment_intent() {
        const order = this.pos.get_order();
        const line = order.selected_paymentline;
        // Build informations for creating a payment intend on Mercado Pago.
        // Data in "external_reference" are send back with the webhook notification
        const infos = {
            amount: parseInt(line.amount * 100, 10),
            additional_info: {
                external_reference: `${this.pos.pos_session.id}_${line.payment_method.id}_${order.uid}`,
                print_on_terminal: true,
            },
        };
        // mp_payment_intent_create will call the Mercado Pago api
        return await this.env.services.orm.silent.call(
            "pos.payment.method",
            "mp_payment_intent_create",
            [[line.payment_method.id], infos]
        );
    }
    async get_last_status_payment_intent() {
        const line = this.pos.get_order().selected_paymentline;
        // mp_payment_intent_get will call the Mercado Pago api
        return await this.env.services.orm.silent.call(
            "pos.payment.method",
            "mp_payment_intent_get",
            [[line.payment_method.id], this.payment_intent.id]
        );
    }

    async cancel_payment_intent() {
        const line = this.pos.get_order().selected_paymentline;
        // mp_payment_intent_cancel will call the Mercado Pago api
        return await this.env.services.orm.silent.call(
            "pos.payment.method",
            "mp_payment_intent_cancel",
            [[line.payment_method.id], this.payment_intent.id]
        );
    }

    async get_payment(payment_id) {
        const line = this.pos.get_order().selected_paymentline;
        // mp_get_payment_status will call the Mercado Pago api
        return await this.env.services.orm.silent.call(
            "pos.payment.method",
            "mp_get_payment_status",
            [[line.payment_method.id], payment_id]
        );
    }

    setup() {
        super.setup(...arguments);
        this.webhook_resolver = null;
        this.payment_intent = {};
    }

    async send_payment_request(cid) {
        await super.send_payment_request(...arguments);
        const line = this.pos.get_order().selected_paymentline;
        try {
            // During payment creation, user can't cancel the payment intent
            line.set_payment_status("waitingCapture");
            // Call Mercado Pago to create a payment intent
            const payment_intent = await this.create_payment_intent();
            if (!("id" in payment_intent)) {
                this._showMsg(payment_intent.message, "error");
                return false;
            }
            // Payment intent creation successfull, save it
            this.payment_intent = payment_intent;
            // After payment creation, make the payment intent canceling possible
            line.set_payment_status("waitingCard");
            // Wait for payment intent status change and return status result
            return await new Promise((resolve) => {
                this.webhook_resolver = resolve;
            });
        } catch (error) {
            this._showMsg(error, "System error");
            return false;
        }
    }

    async send_payment_cancel(order, cid) {
        await super.send_payment_cancel(order, cid);
        if (!("id" in this.payment_intent)) {
            return true;
        }
        const canceling_status = await this.cancel_payment_intent();
        if ("error" in canceling_status) {
            const message =
                canceling_status.status === 409
                    ? _t("Payment has to be canceled on terminal")
                    : _t("Payment not found (canceled/finished on terminal)");
            this._showMsg(message, "info");
            return canceling_status.status !== 409;
        }
        return true;
    }

    async handleMercadoPagoWebhook() {
        const line = this.pos.get_order().selected_paymentline;
        const MAX_RETRY = 5; // Maximum number of retries for the "ON_TERMINAL" BUG
        const RETRY_DELAY = 1000; // Delay between retries in milliseconds for the "ON_TERMINAL" BUG

        const showMessageAndResolve = (messageKey, status, resolverValue) => {
            if (!resolverValue) {
                this._showMsg(messageKey, status);
            }
            line.set_payment_status("done");
            this.webhook_resolver?.(resolverValue);
            return resolverValue;
        };

        const handleFinishedPayment = async (paymentIntent) => {
            if (paymentIntent.state === "CANCELED") {
                return showMessageAndResolve(_t("Payment has been canceled"), "info", false);
            }
            if (["FINISHED", "PROCESSED"].includes(paymentIntent.state)) {
                const payment = await this.get_payment(paymentIntent.payment.id);
                if (payment.status === "approved") {
                    return showMessageAndResolve(_t("Payment has been processed"), "info", true);
                }
                return showMessageAndResolve(_t("Payment has been rejected"), "info", false);
            }
        }

        // No payment intent id means either that the user reload the page or
        // it is an old webhook -> trash
        if ("id" in this.payment_intent) {
            // Call Mercado Pago to get the payment intent status
            let last_status_payment_intent = await this.get_last_status_payment_intent();
            // Bad payment intent id, then it's an old webhook not related with the
            // current payment intent -> trash
            if (this.payment_intent.id == last_status_payment_intent.id) {
                if (["FINISHED", "PROCESSED", "CANCELED"].includes(last_status_payment_intent.state)) {
                    return await handleFinishedPayment(last_status_payment_intent);
                }
                // BUG Sometimes the Mercado Pago webhook return ON_TERMINAL
                // instead of CANCELED/FINISHED when we requested a payment status
                // that was actually canceled/finished by the user on the terminal.
                // Then the strategy here is to ask Mercado Pago MAX_RETRY times the
                // payment intent status, hoping going out of this status
                if (["OPEN", "ON_TERMINAL"].includes(last_status_payment_intent.state)) {
                    return await new Promise((resolve) => {
                        let retry_cnt = 0;
                        const s = setInterval(async () => {
                            last_status_payment_intent =
                                await this.get_last_status_payment_intent();
                            if (
                                ["FINISHED", "PROCESSED", "CANCELED"].includes(
                                    last_status_payment_intent.state
                                )
                            ) {
                                clearInterval(s);
                                resolve(await handleFinishedPayment(last_status_payment_intent));
                            }
                            retry_cnt += 1;
                            if (retry_cnt >= MAX_RETRY) {
                                clearInterval(s);
                                resolve(
                                    showMessageAndResolve(
                                        _t("Payment status could not be confirmed"),
                                        "error",
                                        false
                                    )
                                );
                            }
                        }, RETRY_DELAY);
                    });
                }
                // If the state does not match any of the expected values
                return showMessageAndResolve(_t("Unknown payment status"), "error", false);
            }
        }
    }

    // private methods
    _showMsg(msg, title) {
        this.env.services.popup.add(ErrorPopup, {
            title: "Mercado Pago " + title,
            body: msg,
        });
    }
}

```

## File: static\src\app\pos_bus.js

```javascript
/** @odoo-module */
import { patch } from "@web/core/utils/patch";
import { PosBus } from "@point_of_sale/app/bus/pos_bus_service";

patch(PosBus.prototype, {
    // Override
    dispatch(message) {
        super.dispatch(...arguments);
        if (message.type === "MERCADO_PAGO_LATEST_MESSAGE") {
            this.pos
                .getPendingPaymentLine("mercado_pago")
                .payment_method.payment_terminal.handleMercadoPagoWebhook();
        }
    },
});

```

## File: static\src\overrides\models\models.js

```javascript
/** @odoo-module */
import { register_payment_method } from "@point_of_sale/app/store/pos_store";
import { PaymentMercadoPago } from "@pos_mercado_pago/app/payment_mercado_pago";

register_payment_method("mercado_pago", PaymentMercadoPago);

```

## File: views\pos_payment_method_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="pos_payment_method_view_form_inherit_pos_mercado_pago" model="ir.ui.view">
        <field name="name">pos.payment.method.form.inherit.mercado_pago</field>
        <field name="model">pos.payment.method</field>
        <field name="inherit_id" ref="point_of_sale.pos_payment_method_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='use_payment_terminal']" position="after">
                <!-- MercadoPago -->
                <field name="mp_bearer_token" placeholder="APP_USR-..." invisible="use_payment_terminal != 'mercado_pago'" required="use_payment_terminal == 'mercado_pago'"/>
                <field name="mp_webhook_secret_key" placeholder="c2f3662..." invisible="use_payment_terminal != 'mercado_pago'" required="use_payment_terminal == 'mercado_pago'"/>
                <field name="mp_id_point_smart" placeholder="1494126963" invisible="use_payment_terminal != 'mercado_pago'" required="use_payment_terminal == 'mercado_pago'"/>
                <button string="Force PDV" type="object" name="force_pdv" groups="base.group_no_one" class="oe_highlight" invisible="use_payment_terminal != 'mercado_pago'" required="use_payment_terminal == 'mercado_pago'"/>
            </xpath>
        </field>
    </record>
</odoo>

```

