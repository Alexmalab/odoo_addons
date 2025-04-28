# Odoo Module: pos_viva_wallet

Category: Sales/Point of Sale

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# coding: utf-8
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import controllers

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
{
    'name': 'POS Viva Wallet',
    'version': '1.0',
    'category': 'Sales/Point of Sale',
    'sequence': 7,
    'summary': 'Integrate your POS with a Viva Wallet payment terminal',
    'data': [
        'views/pos_payment_method_views.xml',
    ],
    'depends': ['point_of_sale'],
    'installable': True,
    'assets': {
        'point_of_sale._assets_pos': [
            'pos_viva_wallet/static/src/**/*',
        ],
        'web.assets_tests': [
            'pos_viva_wallet/static/tests/tours/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# coding: utf-8
import logging
import json
from odoo import http, _
from odoo.http import request
from odoo.tools import consteq

_logger = logging.getLogger(__name__)


class PosVivaWalletController(http.Controller):
    @http.route('/pos_viva_wallet/notification', type='http', auth='none', csrf=False, readonly=False)
    def notification(self, company_id, token):
        _logger.info('notification received from Viva Wallet')

        viva_payment_methods = request.env['pos.payment.method'].sudo().search([('use_payment_terminal', '=', 'viva_wallet'), ('company_id.id', '=', company_id)])
        payment_method_sudo = next(
            (pm for pm in viva_payment_methods if consteq(pm.viva_wallet_webhook_verification_key, token)),
            None
        )

        if payment_method_sudo:
            if request.httprequest.data:
                data = request.get_json_data()
                terminal_id = data.get('EventData', {}).get('TerminalId', '')
                data_webhook = data.get('EventData', {})
                if terminal_id:
                    payment_method_sudo = request.env['pos.payment.method'].sudo().search([('viva_wallet_terminal_id', '=', terminal_id)], limit=1)
                    payment_method_sudo._retrieve_session_id(data_webhook)
                else:
                    _logger.error(_('received a message for a terminal not registered in Odoo: %s', terminal_id))
            return json.dumps({'Key': payment_method_sudo.viva_wallet_webhook_verification_key})
        else:
            _logger.error(_('received a message for a pos payment provider not registered.'))

```

## File: controllers\__init__.py

```python
# coding: utf-8
from . import main

```

## File: data\neutralize.sql

```sql
UPDATE pos_payment_method
   SET viva_wallet_test_mode = true;

```

## File: models\pos_payment_method.py

```python
# coding: utf-8
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import logging
import requests

from odoo import fields, models, api, tools, _
from odoo.exceptions import UserError, AccessError

_logger = logging.getLogger(__name__)
TIMEOUT = 10


class PosPaymentMethod(models.Model):
    _inherit = 'pos.payment.method'

    # Viva Wallet
    viva_wallet_merchant_id = fields.Char(string="Merchant ID", help='Used when connecting to Viva Wallet: https://developer.vivawallet.com/getting-started/find-your-account-credentials/merchant-id-and-api-key/')
    viva_wallet_api_key = fields.Char(string="API Key", help='Used when connecting to Viva Wallet: https://developer.vivawallet.com/getting-started/find-your-account-credentials/merchant-id-and-api-key/')
    viva_wallet_client_id = fields.Char(string="Client ID", help='Used when connecting to Viva Wallet: https://developer.vivawallet.com/getting-started/find-your-account-credentials/pos-apis-credentials/#find-your-pos-apis-credentials')
    viva_wallet_client_secret = fields.Char(string="Client secret")
    viva_wallet_terminal_id = fields.Char(string="Terminal ID", help='[Terminal ID of the Viva Wallet terminal], for example: 16002169')
    viva_wallet_bearer_token = fields.Char(default='Bearer Token')
    viva_wallet_webhook_verification_key = fields.Char()
    viva_wallet_latest_response = fields.Json() # used to buffer the latest asynchronous notification from Adyen.
    viva_wallet_test_mode = fields.Boolean(string="Test mode", help="Run transactions in the test environment.")
    viva_wallet_webhook_endpoint = fields.Char(compute='_compute_viva_wallet_webhook_endpoint', readonly=True)


    def _viva_wallet_account_get_endpoint(self):
        if self.viva_wallet_test_mode:
            return 'https://demo-accounts.vivapayments.com'
        return 'https://accounts.vivapayments.com'

    def _viva_wallet_api_get_endpoint(self):
        if self.viva_wallet_test_mode:
            return 'https://demo-api.vivapayments.com'
        return 'https://api.vivapayments.com'

    def _viva_wallet_webhook_get_endpoint(self):
        if self.viva_wallet_test_mode:
            return 'https://demo.vivapayments.com'
        return 'https://www.vivapayments.com'

    def _compute_viva_wallet_webhook_endpoint(self):
        web_base_url = self.get_base_url()
        self.viva_wallet_webhook_endpoint = f"{web_base_url}/pos_viva_wallet/notification?company_id={self.company_id.id}&token={self.viva_wallet_webhook_verification_key}"

    def _is_write_forbidden(self, fields):
        # Allow the modification of these fields even if a pos_session is open
        whitelisted_fields = {'viva_wallet_bearer_token', 'viva_wallet_webhook_verification_key', 'viva_wallet_latest_response'}
        return super(PosPaymentMethod, self)._is_write_forbidden(fields - whitelisted_fields)

    def _get_payment_terminal_selection(self):
        return super()._get_payment_terminal_selection() + [('viva_wallet', 'Viva Wallet')]

    def _bearer_token(self, session):
        self.ensure_one()

        data = {'grant_type': 'client_credentials'}
        auth = requests.auth.HTTPBasicAuth(self.viva_wallet_client_id, self.viva_wallet_client_secret)
        try:
            resp = session.post(f"{self._viva_wallet_account_get_endpoint()}/connect/token", auth=auth, data=data, timeout=TIMEOUT)
        except requests.exceptions.RequestException:
            _logger.exception("Failed to call viva_wallet_bearer_token endpoint")

        access_token = resp.json().get('access_token')
        if access_token:
            self.viva_wallet_bearer_token = access_token
            return {'Authorization': f"Bearer {access_token}"}
        else:
            raise UserError(_('Unable to retrieve Viva Wallet Bearer Token: Please verify that the Client ID and Client Secret are correct'))

    def _get_verification_key(self, endpoint, viva_wallet_merchant_id, viva_wallet_api_key):
        # Get a key to configure the webhook.
        # this key need to be the response when we receive a notifiaction
        # do not execute this query in test mode
        if tools.config['test_enable']:
            return 'viva_wallet_test'

        auth = requests.auth.HTTPBasicAuth(viva_wallet_merchant_id, viva_wallet_api_key)
        try:
            resp = requests.get(f"{endpoint}/api/messages/config/token", auth=auth, timeout=TIMEOUT)
        except requests.exceptions.RequestException:
            _logger.exception('Failed to call https://%s/api/messages/config/token endpoint', endpoint)
        return resp.json().get('Key')

    def _call_viva_wallet(self, endpoint, action, data=None, should_retry=True):
        session = get_viva_wallet_session(should_retry)
        session.headers.update({'Authorization': f"Bearer {self.viva_wallet_bearer_token}"})
        endpoint = f"{self._viva_wallet_api_get_endpoint()}/ecr/v1/{endpoint}"
        try:
            resp = session.request(action, endpoint, json=data, timeout=TIMEOUT)
        except requests.exceptions.RequestException as e:
            return {'error': _("There are some issues between us and Viva Wallet, try again later.%s)", e)}
        if resp.text and resp.json().get('detail') == 'Could not validate credentials':
            session.headers.update(self._bearer_token(session))
            resp = session.request(action, endpoint, json=data, timeout=TIMEOUT)

        if resp.status_code == 200:
            if resp.text:
                return resp.json()
            return {'success': resp.status_code}
        else:
            return {'error': _("There are some issues between us and Viva Wallet, try again later. %s", resp.json().get('detail'))}

    def _retrieve_session_id(self, data_webhook):
        # Send a request to confirm the status of the sesions_id
        # Need wait to the status of sesions_id is updated setted in session headers; code 202

        MerchantTrns = data_webhook.get('MerchantTrns')
        if not MerchantTrns:
            return self._send_notification(
                {'error': _(
                    "Your transaction with Viva Wallet failed. Please try again later."
                    )}
                )
        session_id, pos_session_id = MerchantTrns.split("/")  # Split to retrieve pos_sessions_id
        endpoint = f"sessions/{session_id}"
        data = self._call_viva_wallet(endpoint, 'get')

        if data.get('success'):
            data.update({'pos_session_id': pos_session_id, 'data_webhook': data_webhook})
            self.viva_wallet_latest_response = data
            self._send_notification(data)
        else:
            self._send_notification(
                {'error': _(
                    "There are some issues between us and Viva Wallet, try again later. %s",
                    data.get('detail')
                    )}
                )

    def _send_notification(self, data):
        # Send a notification to the point of sale channel to indicate that the transaction are finish
        pos_session_sudo = self.env["pos.session"].browse(int(data.get('pos_session_id', False)))
        if pos_session_sudo:
            pos_session_sudo.config_id._notify('VIVA_WALLET_LATEST_RESPONSE', {
                'config_id': pos_session_sudo.config_id.id
            })

    def _load_pos_data_fields(self, config_id):
        data = super()._load_pos_data_fields(config_id)
        data += ['viva_wallet_terminal_id']
        return data

    def viva_wallet_send_payment_request(self, data):
        if not self.env.user.has_group('point_of_sale.group_pos_user'):
            raise AccessError(_("Only 'group_pos_user' are allowed to send a Viva Wallet payment request"))

        endpoint = "transactions:sale"
        return self._call_viva_wallet(endpoint, 'post', data)

    def viva_wallet_send_payment_cancel(self, data):
        if not self.env.user.has_group('point_of_sale.group_pos_user'):
            raise AccessError(_("Only 'group_pos_user' are allowed to cancel a Viva Wallet payment"))

        session_id = data.get('sessionId')
        cash_register_id = data.get('cashRegisterId')
        endpoint = f"sessions/{session_id}?cashRegisterId={cash_register_id}"
        return self._call_viva_wallet(endpoint, 'delete')

    def viva_wallet_get_payment_status(self, session_id):
        if not self.env.user.has_group('point_of_sale.group_pos_user'):
            raise AccessError(_("Only 'group_pos_user' are allowed to get the payment status from Viva Wallet"))

        endpoint = f"sessions/{session_id}"
        return self._call_viva_wallet(endpoint, 'get', should_retry=False)

    def write(self, vals):
        record = super().write(vals)

        if vals.get('viva_wallet_merchant_id') and vals.get('viva_wallet_api_key'):
            self.viva_wallet_webhook_verification_key = self._get_verification_key(
                self._viva_wallet_webhook_get_endpoint(),
                self.viva_wallet_merchant_id,
                self.viva_wallet_api_key
                )
            if not self.viva_wallet_webhook_verification_key:
                raise UserError(_("Can't update payment method. Please check the data and update it."))

        return record


    def create(self, vals):
        records = super().create(vals)

        for record in records:
            if record.viva_wallet_merchant_id and record.viva_wallet_api_key:
                record.viva_wallet_webhook_verification_key = record._get_verification_key(
                    record._viva_wallet_webhook_get_endpoint(),
                    record.viva_wallet_merchant_id,
                    record.viva_wallet_api_key,
                )
                if not record.viva_wallet_webhook_verification_key:
                    raise UserError(_("Can't create payment method. Please check the data and update it."))

        return records

    def get_latest_viva_wallet_status(self):
        if not self.env.user.has_group('point_of_sale.group_pos_user'):
            raise AccessError(_("Only 'group_pos_user' are allowed to get latest transaction status"))

        self.ensure_one()
        latest_response = self.sudo().viva_wallet_latest_response
        return latest_response

    @api.constrains('use_payment_terminal')
    def _check_viva_wallet_credentials(self):
        for record in self:
            if (record.use_payment_terminal == 'viva_wallet'
                and not all(record[f] for f in [
                    'viva_wallet_merchant_id',
                    'viva_wallet_api_key',
                    'viva_wallet_client_id',
                    'viva_wallet_client_secret',
                    'viva_wallet_terminal_id']
                )
            ):
                raise UserError(_('It is essential to provide API key for the use of viva wallet'))


def get_viva_wallet_session(should_retry=True):
    session = requests.Session()
    if should_retry:
        session.mount('https://', requests.adapters.HTTPAdapter(max_retries=requests.adapters.Retry(
            total=5,
            backoff_factor=2,
            status_forcelist=[202, 500, 502, 503, 504],
            )))
    return session

```

## File: models\__init__.py

```python
# coding: utf-8
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import pos_payment_method

```

## File: static\src\app\payment_viva_wallet.js

```javascript
import { _t } from "@web/core/l10n/translation";
import { PaymentInterface } from "@point_of_sale/app/payment/payment_interface";
import { AlertDialog } from "@web/core/confirmation_dialog/confirmation_dialog";
import { sprintf } from "@web/core/utils/strings";
import { roundPrecision } from "@web/core/utils/numbers";
import { uuidv4 } from "@point_of_sale/utils";

// Due to consistency issues with the webhook, we also poll
// the status of the payment periodically as a fallback.
const POLLING_INTERVAL_MS = 5000;

export class PaymentVivaWallet extends PaymentInterface {
    /*
     Developer documentation:
    https://developer.vivawallet.com/apis-for-point-of-sale/card-terminals-devices/rest-api/eft-pos-api-documentation/
    */

    setup() {
        super.setup(...arguments);
        this.paymentLineResolvers = {};
    }
    send_payment_request(uuid) {
        super.send_payment_request(uuid);
        return this._viva_wallet_pay(uuid);
    }
    send_payment_cancel(order, uuid) {
        super.send_payment_cancel(order, uuid);
        return this._viva_wallet_cancel();
    }
    pending_viva_wallet_line() {
        return this.pos.getPendingPaymentLine("viva_wallet");
    }

    _call_viva_wallet(data, action) {
        return this.env.services.orm.silent
            .call("pos.payment.method", action, [[this.payment_method_id.id], data])
            .catch(this._handle_odoo_connection_failure.bind(this));
    }

    _handle_odoo_connection_failure(data = {}) {
        // handle timeout
        var line = this.pending_viva_wallet_line();
        if (line) {
            line.set_payment_status("retry");
        }
        this._show_error(
            _t(
                "Could not connect to the Odoo server, please check your internet connection and try again."
            )
        );

        return Promise.reject(data); // prevent subsequent onFullFilled's from being called
    }

    _viva_wallet_handle_response(response) {
        var line = this.pending_viva_wallet_line();
        line.set_payment_status("waitingCard");
        if (response.error) {
            this._show_error(response.error);
        }
        return this.waitForPaymentConfirmation();
    }

    _viva_wallet_pay() {
        /**
         * Override
         */
        super.send_payment_request(...arguments);
        var order = this.pos.get_order();
        var line = order.get_selected_paymentline();
        let customerTrns = " ";
        line.set_payment_status("waitingCard");

        if (line.amount < 0) {
            this._show_error(_t("Cannot process transactions with negative amount."));
            return false;
        }

        if (order.partner) {
            customerTrns = order.partner.name + " - " + order.partner.email;
        }

        line.sessionId = order.uuid + " - " + uuidv4();
        var data = {
            sessionId: line.sessionId,
            terminalId: line.payment_method_id.viva_wallet_terminal_id,
            cashRegisterId: this.pos.get_cashier().name,
            amount: roundPrecision(line.amount * 100),
            currencyCode: this.pos.currency.iso_numeric.toString(),
            merchantReference: line.sessionId + "/" + this.pos.session.id,
            customerTrns: customerTrns,
            preauth: false,
            maxInstalments: 0,
            tipAmount: 0,
        };
        return this._call_viva_wallet(data, "viva_wallet_send_payment_request").then((data) => {
            return this._viva_wallet_handle_response(data);
        });
    }

    async _viva_wallet_cancel(order, uuid) {
        /**
         * Override
         */
        super.send_payment_cancel(...arguments);
        const line = this.pos.get_order().get_selected_paymentline();

        var data = {
            sessionId: line.sessionId,
            cashRegisterId: this.pos.get_cashier().name,
        };
        return this._call_viva_wallet(data, "viva_wallet_send_payment_cancel").then((data) => {
            if (data.error) {
                this._show_error(data.error);
            }
            return true;
        });
    }

    /**
     * This method is called from pos_bus when the payment
     * confirmation from Viva Wallet is received via the webhook and confirmed in the retrieve_session_id.
     */
    async handleVivaWalletStatusResponse() {
        var line = this.pending_viva_wallet_line();
        const notification = await this.env.services.orm.silent.call(
            "pos.payment.method",
            "get_latest_viva_wallet_status",
            [[this.payment_method_id.id]]
        );

        if (!notification) {
            this._handle_odoo_connection_failure();
            return;
        }

        const isPaymentSuccessful = this.isPaymentSuccessful(notification);
        if (isPaymentSuccessful) {
            this.handleSuccessResponse(line, notification);
        } else {
            this._show_error(sprintf(_t("Message from Viva Wallet: %s"), notification.error));
        }

        // when starting to wait for the payment response we create a promise
        // that will be resolved when the payment response is received.
        // In case this resolver is lost ( for example on a refresh ) we
        // we use the handle_payment_response method on the payment line
        const resolver = this.paymentLineResolvers?.[line.uuid];
        if (resolver) {
            this.paymentLineResolvers[line.uuid] = null;
            resolver(isPaymentSuccessful);
        } else {
            line.handle_payment_response(isPaymentSuccessful);
        }
    }

    isPaymentSuccessful(notification) {
        return (
            notification &&
            notification.sessionId == this.pending_viva_wallet_line().sessionId &&
            notification.success
        );
    }

    waitForPaymentConfirmation() {
        return new Promise((resolve) => {
            const paymentLine = this.pending_viva_wallet_line();
            const sessionId = paymentLine.sessionId;
            this.paymentLineResolvers[paymentLine.uuid] = resolve;
            const intervalId = setInterval(async () => {
                const isPaymentStillValid = () =>
                    this.paymentLineResolvers[paymentLine.uuid] &&
                    this.pending_viva_wallet_line()?.sessionId === sessionId &&
                    paymentLine.payment_status === "waitingCard";
                if (!isPaymentStillValid()) {
                    clearInterval(intervalId);
                    return;
                }

                const result = await this._call_viva_wallet(
                    sessionId,
                    "viva_wallet_get_payment_status"
                );
                if ("success" in result && isPaymentStillValid()) {
                    clearInterval(intervalId);
                    if (this.isPaymentSuccessful(result)) {
                        this.handleSuccessResponse(paymentLine, result);
                        resolve(true);
                    } else {
                        resolve(false);
                    }
                    this.paymentLineResolvers[paymentLine.uuid] = null;
                }
            }, POLLING_INTERVAL_MS);
        });
    }

    handleSuccessResponse(line, notification) {
        line.transaction_id = notification.transactionId;
        line.card_type = notification.applicationLabel;
        line.cardholder_name = notification.FullName || "";
    }

    _show_error(msg, title) {
        if (!title) {
            title = _t("Viva Wallet Error");
        }
        this.env.services.dialog.add(AlertDialog, {
            title: title,
            body: msg,
        });
    }
}

```

## File: static\src\components\payment_screen\payment_screen.js

```javascript
import { PaymentScreen } from "@point_of_sale/app/screens/payment_screen/payment_screen";
import { patch } from "@web/core/utils/patch";
import { onMounted } from "@odoo/owl";

patch(PaymentScreen.prototype, {
    setup() {
        super.setup(...arguments);
        onMounted(() => {
            const pendingPaymentLine = this.currentOrder.payment_ids.find(
                (paymentLine) =>
                    paymentLine.payment_method_id.use_payment_terminal === "viva_wallet" &&
                    !paymentLine.is_done() &&
                    paymentLine.get_payment_status() !== "pending"
            );
            if (!pendingPaymentLine) {
                return;
            }
        });
    },
});

```

## File: static\src\overrides\models.js

```javascript
import { register_payment_method } from "@point_of_sale/app/store/pos_store";
import { PaymentVivaWallet } from "@pos_viva_wallet/app/payment_viva_wallet";

register_payment_method("viva_wallet", PaymentVivaWallet);

```

## File: static\src\overrides\pos_store.js

```javascript
import { patch } from "@web/core/utils/patch";
import { PosStore } from "@point_of_sale/app/store/pos_store";

patch(PosStore.prototype, {
    async setup() {
        await super.setup(...arguments);
        this.data.connectWebSocket("VIVA_WALLET_LATEST_RESPONSE", () => {
            const pendingLine = this.getPendingPaymentLine("viva_wallet");

            if (pendingLine) {
                pendingLine.payment_method_id.payment_terminal.handleVivaWalletStatusResponse();
            }
        });
    },
});

```

## File: views\pos_payment_method_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="pos_payment_method_view_form_inherit_pos_viva_wallet" model="ir.ui.view">
        <field name="name">pos.payment.method.form.inherit.viva.wallet</field>
        <field name="model">pos.payment.method</field>
        <field name="inherit_id" ref="point_of_sale.pos_payment_method_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='use_payment_terminal']" position="after">
                <!-- Viva Wallet -->
                <field name="viva_wallet_merchant_id" invisible="use_payment_terminal != 'viva_wallet'" required="use_payment_terminal == 'viva_wallet'"/>
                <field name="viva_wallet_api_key" invisible="use_payment_terminal != 'viva_wallet'" required="use_payment_terminal == 'viva_wallet'"/>
                <field name="viva_wallet_client_id" invisible="use_payment_terminal != 'viva_wallet'" required="use_payment_terminal == 'viva_wallet'"/>
                <field name="viva_wallet_client_secret" invisible="use_payment_terminal != 'viva_wallet'" required="use_payment_terminal == 'viva_wallet'"/>
                <field name="viva_wallet_test_mode" invisible="use_payment_terminal != 'viva_wallet'" required="use_payment_terminal == 'viva_wallet'"/>
                <field name="viva_wallet_terminal_id" invisible="use_payment_terminal != 'viva_wallet'" required="use_payment_terminal == 'viva_wallet'"/>
                <field name="viva_wallet_webhook_endpoint" invisible="use_payment_terminal != 'viva_wallet' or not id" required="use_payment_terminal == 'viva_wallet'" widget="CopyClipboardChar"/>
            </xpath>
        </field>
    </record>
</odoo>

```

