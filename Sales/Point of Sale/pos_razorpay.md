# Odoo Module: pos_razorpay

Category: Sales/Point of Sale

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'POS Razorpay',
    'version': '1.0',
    'category': 'Sales/Point of Sale',
    'sequence': 6,
    'summary': 'Integrate your POS with a Razorpay payment terminal',
    'description': """
Allow Razorpay POS payments
==============================

This module allows customers to pay for their orders with debit/credit
cards and UPI. The transactions are processed by Razorpay POS. A Razorpay merchant account is necessary. It allows the
following:

* Fast payment by just swiping/scanning a credit/debit card or a QR code while on the payment screen
* Supported cards: Visa, MasterCard, Rupay, UPI
    """,
    'data': [
        'views/pos_payment_method_views.xml',
    ],
    'depends': ['point_of_sale'],
    'installable': True,
    'assets': {
        'point_of_sale._assets_pos': [
            'pos_razorpay/static/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\neutralize.sql

```sql
UPDATE pos_payment_method
SET razorpay_test_mode = true,
    razorpay_api_key = 'dummykey00012345';

```

## File: models\pos_payment.py

```python
from odoo import models, fields

class PosPayment(models.Model):

    _inherit = "pos.payment"

    razorpay_reverse_ref_no = fields.Char('Razorpay Reverse Reference No.')

```

## File: models\pos_payment_method.py

```python
from odoo.exceptions import UserError
from odoo import fields, models, api, _

from .razorpay_pos_request import RazorpayPosRequest

class PosPaymentMethod(models.Model):
    _inherit = 'pos.payment.method'

    razorpay_tid = fields.Char(string='Razorpay Device Serial No', help='Device Serial No \n ex: 7000012300')
    razorpay_allowed_payment_modes = fields.Selection(selection=[('all', 'All'), ('card', 'Card'), ('upi', 'UPI'), ('bharatqr', 'BHARATQR')], default='all', help='Choose allow payment mode: \n All/Card/UPI or QR')
    razorpay_username = fields.Char(string='Razorpay Username', help='Username(Device Login) \n ex: 1234500121')
    razorpay_api_key = fields.Char(string='Razorpay API Key', help='Used when connecting to Razorpay: https://razorpay.com/docs/payments/dashboard/account-settings/api-keys/')
    razorpay_test_mode = fields.Boolean(string='Razorpay Test Mode', default=False, help='Turn it on when in Test Mode')

    def _get_payment_terminal_selection(self):
        return super()._get_payment_terminal_selection() + [('razorpay', 'Razorpay')]

    def razorpay_make_payment_request(self, data):
        razorpay = RazorpayPosRequest(self)
        body = razorpay._razorpay_get_payment_request_body(payment_mode=True)
        body.update({
            'amount': data.get('amount'),
            'externalRefNumber': data.get('referenceId')
        })
        response = razorpay._call_razorpay(endpoint='pay', payload=body)
        if response.get('success') and not response.get('errorCode'):
            return {
                'success': True,
                'p2pRequestId': str(response.get('p2pRequestId'))
            }
        default_error_msg = _('Razorpay POS payment request expected errorCode not found in the response')
        error = response.get('errorMessage') or default_error_msg
        return {'error': str(error)}

    def razorpay_fetch_payment_status(self, data):
        razorpay = RazorpayPosRequest(self)
        body = razorpay._razorpay_get_payment_status_request_body()
        body.update({'origP2pRequestId': data.get('p2pRequestId')})
        response = razorpay._call_razorpay(endpoint='status', payload=body)
        if response.get('success') and not response.get('errorCode'):
            payment_status = response.get('status')
            payment_messageCode = response.get('messageCode')
            if payment_status == 'AUTHORIZED' and payment_messageCode == 'P2P_DEVICE_TXN_DONE':
                return {
                    'status': response.get('status'),
                    'authCode': response.get('authCode'),
                    'cardLastFourDigit': response.get('cardLastFourDigit'),
                    'externalRefNumber': response.get('externalRefNumber'),
                    'reverseReferenceNumber': response.get('reverseReferenceNumber'),
                    'txnId': response.get('txnId'),
                    'paymentMode': response.get('paymentMode'),
                    'paymentCardType': response.get('paymentCardType'),
                    'paymentCardBrand': response.get('paymentCardBrand'),
                    'nameOnCard': response.get('nameOnCard'),
                    'acquirerCode': response.get('acquirerCode'),
                    'createdTime': response.get('createdTime'),
                }
            elif payment_status == 'FAILED' or payment_messageCode == 'P2P_DEVICE_CANCELED':
                return {'error': str(response.get('message', _('Razorpay POS transaction failed'))),
                        'payment_messageCode': payment_messageCode}
            elif payment_messageCode in ['P2P_DEVICE_RECEIVED', 'P2P_DEVICE_SENT', 'P2P_STATUS_QUEUED']:
                return {'status': payment_messageCode.split('_')[-1]}
        default_error_msg = _('Razorpay POS payment status request expected errorCode not found in the response')
        error = response.get('errorMessage') or default_error_msg
        return {'error': str(error)}

    def razorpay_cancel_payment_request(self, data):
        razorpay = RazorpayPosRequest(self)
        body = razorpay._razorpay_get_payment_request_body(payment_mode=False)
        body.update({'origP2pRequestId': data.get('p2pRequestId')})
        response = razorpay._call_razorpay(endpoint='cancel', payload=body)
        if response.get('success') and not response.get('errorCode'):
            return {'error': _('Razorpay POS transaction canceled successfully')}
        default_error_msg = _('Razorpay POS payment cancel request expected errorCode not found in the response')
        errorMessage = response.get('errorMessage') or default_error_msg
        return {'errorMessage': str(errorMessage)}

    @api.constrains('use_payment_terminal')
    def _check_razorpay_terminal(self):
        if any(record.use_payment_terminal == 'razorpay' and record.company_id.currency_id.name != 'INR' for record in self):
            raise UserError(_('This Payment Terminal is only valid for INR Currency'))

```

## File: models\razorpay_pos_request.py

```python
import requests
import logging

from odoo import _
REQUEST_TIMEOUT = 10

_logger = logging.getLogger(__name__)

class RazorpayPosRequest:
    def __init__(self, payment_method):
        self.razorpay_test_mode = payment_method.razorpay_test_mode
        self.razorpay_api_key = payment_method.razorpay_api_key
        self.razorpay_username = payment_method.razorpay_username
        self.razorpay_tid = payment_method.razorpay_tid
        self.razorpay_allowed_payment_modes = payment_method.razorpay_allowed_payment_modes
        self.payment_method = payment_method
        self.session = requests.Session()

    def _razorpay_get_endpoint(self):
        if self.razorpay_test_mode:
            return 'https://demo.ezetap.com/api/3.0/p2padapter/'
        return 'https://www.ezetap.com/api/3.0/p2padapter/'

    def _call_razorpay(self, endpoint, payload):
        """ Make a request to Razorpay POS API.

        :param str endpoint: The endpoint to be reached by the request.
        :param dict payload: The payload of the request.
        :return The JSON-formatted content of the response.
        :rtype: dict
        """
        endpoint = f'{self._razorpay_get_endpoint()}{endpoint}'
        request_timeout = self.payment_method.env['ir.config_parameter'].sudo().get_param('pos_razorpay.timeout', REQUEST_TIMEOUT)
        try:
            response = self.session.post(endpoint, json=payload, timeout=request_timeout)
            response.raise_for_status()
            res_json = response.json()
        except requests.exceptions.RequestException as error:
            _logger.warning('Cannot connect with Razorpay POS. Error: %s', error)
            return {'errorMessage': str(error)}
        except ValueError as error:
            _logger.warning('Cannot decode response json. Error: %s', error)
            return {'errorMessage': _('Cannot decode Razorpay POS response')}
        return res_json

    def _razorpay_get_payment_request_body(self, payment_mode=True):
        request_parameters = {
            'pushTo': {
                'deviceId': f'{self.razorpay_tid}|ezetap_android',
            },
        }
        if payment_mode:
            request_parameters.update({'mode': self.razorpay_allowed_payment_modes.upper()})
        request_parameters.update(self._razorpay_get_payment_status_request_body())
        return request_parameters

    def _razorpay_get_payment_status_request_body(self):
        return {
            'username': self.razorpay_username,
            'appKey': self.razorpay_api_key,
        }

```

## File: models\__init__.py

```python
from . import pos_payment_method
from . import pos_payment

```

## File: static\src\app\payment_razorpay.js

```javascript
import { _t } from "@web/core/l10n/translation";
import { PaymentInterface } from "@point_of_sale/app/payment/payment_interface";
import { AlertDialog } from "@web/core/confirmation_dialog/confirmation_dialog";
import { serializeDateTime } from "@web/core/l10n/dates";

const REQUEST_TIMEOUT = 10000;
const { DateTime } = luxon;

export class PaymentRazorpay extends PaymentInterface {
    setup() {
        super.setup(...arguments);
        this.pollingTimeout = null;
        this.inactivityTimeout = null;
        this.queued = false;
        this.payment_stopped = false;
    }

    send_payment_request(cid) {
        super.send_payment_request(cid);
        return this._process_razorpay(cid);
    }

    pending_razorpay_line() {
        return this.pos.getPendingPaymentLine("razorpay");
    }

    send_payment_cancel(order, cid) {
        super.send_payment_cancel(order, cid);
        return this._razorpay_cancel();
    }

    _call_razorpay(data, action) {
        return this.env.services.orm.silent
            .call("pos.payment.method", action, [[this.payment_method_id.id], data])
            .catch(this._handle_odoo_connection_failure.bind(this));
    }

    _handle_odoo_connection_failure(data = {}) {
        // handle timeout
        const line = this.pending_razorpay_line();
        if (line) {
            line.set_payment_status("retry");
        }
        this._showError(
            _t(
                "Could not connect to the Odoo server, please check your internet connection and try again."
            )
        );

        return Promise.reject(data); // prevent subsequent onFullFilled's from being called
    }

    /**
     * This method handles the response that comes from Razorpay
     * when we make a request for payment/cancel.
     */
    _razorpay_handle_response(response) {
        const line = this.pending_razorpay_line();
        if (response.error) {
            line.set_payment_status("force_done");
            this.payment_stopped
                ? this._showError(_t("Transaction failed due to inactivity"))
                : this._showError(response.error);
            if (response.payment_messageCode === "P2P_DEVICE_CANCELED") {
                line.set_payment_status("retry");
            }
            this._removePaymentHandler(["p2pRequestId", "referenceId"]);
            return Promise.resolve(false);
        }
        line.set_payment_status("waitingCard");
        localStorage.setItem("p2pRequestId", response.p2pRequestId);
        return this._waitForPaymentConfirmation();
    }

    _razorpay_cancel() {
        const data = { p2pRequestId: localStorage.getItem("p2pRequestId") };
        return this._call_razorpay(data, "razorpay_cancel_payment_request").then((data) => {
            // This proficiently tackles scenarios where payment initiation is in progress and close to the completion phase
            if (data.errorMessage) {
                this._showError(data.errorMessage);
                return Promise.resolve(false);
            }
            this._razorpay_handle_response(data);
            return Promise.resolve(true);
        });
    }

    _process_razorpay(cid) {
        const order = this.pos.get_order();
        const line = order.get_selected_paymentline();

        if (line.amount < 0) {
            this._showError(_t("Cannot process transactions with negative amount."));
            return Promise.resolve();
        }

        const orderId = order.name.replace(" ", "").replaceAll("-", "").toUpperCase();
        const referencePrefix = this.pos.config.name.replace(/\s/g, "").slice(0, 4);
        localStorage.setItem(
            "referenceId",
            referencePrefix + "/" + orderId + "/" + crypto.randomUUID().replaceAll("-", "")
        );
        const data = {
            amount: line.amount,
            referenceId: localStorage.getItem("referenceId"),
        };
        return this._call_razorpay(data, "razorpay_make_payment_request").then((data) => {
            return this._razorpay_handle_response(data);
        });
    }

    /**
     * Polling
     * This method calls and handles the razorpay status response
     * calls every 10 sec until payment is not resolved.
     */

    async _waitForPaymentConfirmation() {
        const paymentLine = this.pos.get_order().get_selected_paymentline();
        if (!paymentLine || paymentLine.payment_status == "retry") {
            return false;
        }
        const data = { p2pRequestId: localStorage.getItem("p2pRequestId") };
        this._stop_pending_payment().then(() => (this.payment_stopped = true));
        const razorpayFetchPaymentStatus = async (resolve, reject) => {
            //Clear previous timeout before setting a new one
            clearTimeout(this.pollingTimeout);

            // If the user navigates to another screen, stop the polling
            if (this.pos.mainScreen.component.name !== "PaymentScreen") {
                return;
            }

            //Within 90 seconds, inactivity will result in transaction cancellation and payment termination.
            if (this.payment_stopped) {
                this._razorpay_cancel().then(() => {
                    paymentLine.set_payment_status("retry");
                    this.payment_stopped = false;
                });
                return resolve(false);
            }

            const response = await this._call_razorpay(data, "razorpay_fetch_payment_status");
            if (response.error) {
                return this._razorpay_handle_response(response);
            }

            const resultCode = response?.status;

            if (resultCode === "QUEUED" && this.queued === false) {
                this._showError(
                    _t(
                        "Payment has been queued. You may choose to wait for the payment to initiate on terminal or proceed to cancel this transaction"
                    )
                );
                this.queued = true;
            }
            if (
                resultCode === "AUTHORIZED" &&
                response?.externalRefNumber !== localStorage.getItem("referenceId")
            ) {
                return this._razorpay_handle_response({ error: _t("Reference number mismatched") });
            } else if (resultCode === "AUTHORIZED") {
                paymentLine.payment_method_authcode = response?.authCode;
                paymentLine.card_no = response?.cardLastFourDigit || "";
                paymentLine.payment_method_issuer_bank = response?.acquirerCode;
                paymentLine.payment_method_payment_mode = response?.paymentMode;
                paymentLine.card_type = response?.paymentCardType;
                paymentLine.card_brand = response?.paymentCardBrand || "";
                paymentLine.cardholder_name = response?.nameOnCard;
                paymentLine.payment_ref_no = response?.externalRefNumber;
                paymentLine.razorpay_reverse_ref_no = response?.reverseReferenceNumber;
                paymentLine.transaction_id = response?.txnId;
                // `createdTime` is provided in milliseconds in local GMT+5.5 timezone.
                // Thus, we need to subtract 19800000 to get the correct time in milliseconds.
                paymentLine.payment_date = this._getPaymentDate(response?.createdTime - 19800000);
                this._removePaymentHandler(["p2pRequestId", "referenceId"]);
                return resolve(response);
            } else {
                this.pollingTimeout = setTimeout(
                    razorpayFetchPaymentStatus,
                    REQUEST_TIMEOUT,
                    resolve,
                    reject
                );
            }
        };
        return new Promise(razorpayFetchPaymentStatus);
    }

    _getPaymentDate(timeMillis) {
        const utcDate = timeMillis
            ? DateTime.fromMillis(timeMillis, { zone: "utc" })
            : DateTime.now();
        return serializeDateTime(utcDate);
    }

    _stop_pending_payment() {
        return new Promise((resolve) => (this.inactivityTimeout = setTimeout(resolve, 90000)));
    }

    _removePaymentHandler(payment_data) {
        payment_data.forEach((data) => {
            localStorage.removeItem(data);
        });
        clearTimeout(this.pollingTimeout);
        clearTimeout(this.inactivityTimeout);
        this.queued = this.payment_stopped = false;
    }

    _showError(error_msg, title) {
        this.env.services.dialog.add(AlertDialog, {
            title: title || _t("Razorpay Error"),
            body: error_msg,
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
        onMounted(async () => {
            const pendingPaymentLine = this.currentOrder.payment_ids.find(
                (paymentLine) =>
                    paymentLine.payment_method_id.use_payment_terminal === "razorpay" &&
                    !paymentLine.is_done() &&
                    paymentLine.get_payment_status() !== "pending"
            );
            if (pendingPaymentLine) {
                const payment_status =
                    await pendingPaymentLine.payment_method_id.payment_terminal._waitForPaymentConfirmation();
                if (payment_status?.status === "AUTHORIZED") {
                    pendingPaymentLine.set_payment_status("done");
                } else {
                    pendingPaymentLine.set_payment_status("force_done");
                }
            }
        });
    },
});

```

## File: static\src\overrides\models.js

```javascript
import { register_payment_method } from "@point_of_sale/app/store/pos_store";
import { PaymentRazorpay } from "@pos_razorpay/app/payment_razorpay";

register_payment_method("razorpay", PaymentRazorpay);

```

## File: views\pos_payment_method_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="pos_payment_method_view_form_inherit_pos_razorpay" model="ir.ui.view">
        <field name="name">pos.payment.method.form.inherit.razorpay</field>
        <field name="model">pos.payment.method</field>
        <field name="inherit_id" ref="point_of_sale.pos_payment_method_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='use_payment_terminal']" position="after">
                <field name="razorpay_username" invisible="use_payment_terminal != 'razorpay'" required="use_payment_terminal == 'razorpay'"/>
                <field name="razorpay_tid" invisible="use_payment_terminal != 'razorpay'" required="use_payment_terminal == 'razorpay'"/>
                <field name="razorpay_api_key" invisible="use_payment_terminal != 'razorpay'" required="use_payment_terminal == 'razorpay'"/>
                <field name="razorpay_test_mode" invisible="use_payment_terminal != 'razorpay'" required="use_payment_terminal == 'razorpay'"/>
                <field name="razorpay_allowed_payment_modes" invisible="use_payment_terminal != 'razorpay'" required="use_payment_terminal == 'razorpay'"/>
            </xpath>
        </field>
    </record>
</odoo>

```

