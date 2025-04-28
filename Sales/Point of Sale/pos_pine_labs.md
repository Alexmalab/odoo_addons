# Odoo Module: pos_pine_labs

Category: Sales/Point of Sale

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
{
    'name': 'POS Pine Labs',
    'version': '1.0',
    'category': 'Sales/Point of Sale',
    'sequence': 6,
    'summary': 'Integrate your POS with Pine Labs payment terminals',
    'description': """
Allow Pine Labs POS payments
==============================

This module is available only for companies that use INR currency.
It enables customers to pay for their orders using debit/credit cards and UPI through Pine Labs POS terminals.
A Pine Labs merchant account is required to process transactions.
Features include:

* Quick payments by swiping, scanning, or tapping your credit/debit card or UPI QR code at the payment terminal.
* Supported cards: Visa, MasterCard, RuPay.
    """,
    'data': [
        'views/pos_order_views.xml',
        'views/pos_payment_method_views.xml',
    ],
    'depends': ['point_of_sale'],
    'installable': True,
    'assets': {
        'point_of_sale._assets_pos': [
            'pos_pine_labs/static/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\neutralize.sql

```sql
UPDATE pos_payment_method
   SET pine_labs_test_mode = true,
       pine_labs_merchant = 'dummymerchant',
       pine_labs_security_token = 'dummy123456';

```

## File: models\pine_labs_pos_request.py

```python
import json
import logging
import requests

from odoo import _

REQUEST_TIMEOUT = 10
PINE_LABS_AUTO_CANCEL_DURATION_MIN = 10
ALLOWED_ENDPOINTS = ['UploadBilledTransaction', 'GetCloudBasedTxnStatus', 'CancelTransactionForced']
ALLOWED_PAYMENT_MODES_MAPPING = {
    'all': 0,
    'card': 1,
    'upi': 10,
}

_logger = logging.getLogger(__name__)


def call_pine_labs(payment_method: object, endpoint: str, payload: dict) -> dict:
    """
    Make a request to Pine Labs POS API.

    :return The content of the response parsed from json
    """
    if endpoint not in ALLOWED_ENDPOINTS:
        raise ValueError(f"Invalid endpoint: '{endpoint}'. Allowed endpoints are: {ALLOWED_ENDPOINTS}")
    payload |= pine_labs_request_body(endpoint == 'UploadBilledTransaction', payment_method=payment_method)
    pine_labs_url = _get_pine_labs_url(payment_method=payment_method)
    url = pine_labs_url + endpoint
    try:
        response = requests.post(url, json=payload, timeout=REQUEST_TIMEOUT)
        response.raise_for_status()
        return response.json()
    except requests.exceptions.ConnectionError as error:
        _logger.warning('Connection Error: %r with the given URL %r', error, url)
        return {'errorMessage': error}
    except requests.exceptions.HTTPError as error:
        _logger.warning('HTTPError: %r', error)
        return {'errorMessage': error}
    except requests.exceptions.Timeout as error:
        _logger.warning('Timeout: %r', error)
        return {'errorMessage': error}
    except json.decoder.JSONDecodeError as error:
        _logger.warning('JSONDecodeError: %r', error)
        return {'errorMessage': error}

def pine_labs_request_body(payment_mode: bool, payment_method: object) -> dict:
    """
    :param payment_mode: If set, includes allowed payment mode and auto-cancel duration in
                        the request body for `UploadBilledTransaction` endpoint.

    rtype: dict
    """
    request_parameters = {
        'MerchantID': payment_method.pine_labs_merchant,
        'StoreID': payment_method.pine_labs_store,
        'ClientID': payment_method.pine_labs_client,
        'SecurityToken': payment_method.pine_labs_security_token
    }
    if payment_mode:
        # Added AutoCancelDurationInMinutes set to 10 minutes for automatically cancelling transactions on the Pine Labs side in case of lost transaction request data from the PoS system.
        pine_labs_auto_cancel_duration = payment_method.env['ir.config_parameter'].sudo().get_param('pos_pine_labs.payment_auto_cancel_duration', PINE_LABS_AUTO_CANCEL_DURATION_MIN)
        request_parameters.update({
            'AllowedPaymentMode': ALLOWED_PAYMENT_MODES_MAPPING.get(payment_method.pine_labs_allowed_payment_mode),
            'AutoCancelDurationInMinutes': pine_labs_auto_cancel_duration
        })
    return request_parameters

def _get_pine_labs_url(payment_method) -> str:
    if payment_method.env['ir.config_parameter'].sudo().get_param('pos_pine_labs.pine_labs_proxy_endpoint', ''):
        return payment_method.env['ir.config_parameter'].sudo().get_param('pos_pine_labs.pine_labs_proxy_endpoint')
    _logger.warning('To use Pine Labs outside India and Malesia, the Pine Labs Proxy Endpoint must be set because the Pine Labs URL is only accessible in India and Malesia.')
    if payment_method.pine_labs_test_mode:
        return 'https://www.plutuscloudserviceuat.in:8201/API/CloudBasedIntegration/V1/'
    return 'https://www.plutuscloudservice.in:8201/API/CloudBasedIntegration/V1/'

```

## File: models\pos_payment.py

```python
from odoo import models, fields

class PosPayment(models.Model):
    _inherit = 'pos.payment'

    pine_labs_plutus_transaction_ref = fields.Char(
        string='Pine Labs PlutusTransactionReferenceID',
        help='Required during the refund order process: https://developer.pinelabs.com/in/instore/cloud-integration#Example-JSON-request-for-Void-ICB-on-UPI-transaction')

```

## File: models\pos_payment_method.py

```python
from odoo import api, fields, models,  _
from odoo.exceptions import UserError

from .pine_labs_pos_request import call_pine_labs


class PosPaymentMethod(models.Model):
    _inherit = 'pos.payment.method'

    pine_labs_merchant = fields.Char(string='Pine Labs Merchant ID', help='A merchant id issued directly to the merchant by Pine Labs.', copy=False)
    pine_labs_store = fields.Char(string='Pine Labs Store ID', help='A store id issued directly to the merchant by Pine Labs.', copy=False)
    pine_labs_client = fields.Char(string='Pine Labs Client ID', help='A client id issued directly to the merchant by Pine Labs.', copy=False)
    pine_labs_security_token = fields.Char(string='Pine Labs Security Token', help='A security token issued directly to the merchant by Pine Labs.')
    pine_labs_allowed_payment_mode = fields.Selection(
        selection=[('all', "All"), ('card', "Card"), ('upi', "Upi")],
        string='Pine Labs Allowed Payment Modes',
        help='Accepted payment modes by Pine Labs for transactions.')
    pine_labs_test_mode = fields.Boolean(string='Pine Labs Test Mode', help='Test Pine Labs transaction process.')

    def _get_payment_terminal_selection(self):
        return super()._get_payment_terminal_selection() + [('pine_labs', 'Pine Labs')]

    def pine_labs_make_payment_request(self, data):
        """
        Sends a payment request to the Pine Labs POS API.

        :param dict data: Contains `amount`, `transactionNumber`, and `sequenceNumber`.
        :return: On success, returns `responseCode`, `status`, and `plutusTransactionReferenceID`. 
                On failure, returns an error message.
        :rtype: dict
        """
        body = {
            'Amount': data['amount'],
            'TransactionNumber': data['transactionNumber'],
            'SequenceNumber': data['sequenceNumber']
        }
        response = call_pine_labs(payment_method=self, endpoint='UploadBilledTransaction', payload=body)
        if response.get('ResponseCode') == 0 and response.get('ResponseMessage') == "APPROVED":
            return {
                'responseCode': response['ResponseCode'],
                'status': response['ResponseMessage'],
                'plutusTransactionReferenceID': response['PlutusTransactionReferenceID'],
            }
        default_error = _('The expected error code for the Pine Labs POS status request was not included in the response.')
        error = response.get('ResponseMessage') or response.get('errorMessage') or default_error
        return {"error": error}

    def pine_labs_fetch_payment_status(self, data):
        """
        Fetches payment status from the Pine Labs POS API.

        :param dict data: Contains `plutusTransactionReferenceID` for the status request.
        :return: On success, returns `responseCode`, `status`, `plutusTransactionReferenceID`, and `data` (formatted transaction details). 
                On failure, returns an error message.
        :rtype: dict
        """
        body = { 'PlutusTransactionReferenceID': data['plutusTransactionReferenceID'] }
        response = call_pine_labs(payment_method=self, endpoint='GetCloudBasedTxnStatus', payload=body)
        if response.get('ResponseCode') in [0, 1001]:
            formatted_transaction_data = { d['Tag']: d['Value'] for d in response['TransactionData'] } if response.get('ResponseCode') == 0 else {}
            return {
                'responseCode': response['ResponseCode'],
                'status': response['ResponseMessage'],
                'plutusTransactionReferenceID': response['PlutusTransactionReferenceID'],
                'data': formatted_transaction_data,
            }
        default_error = _('The expected error code for the Pine Labs POS status request was not included in the response.')
        error = response.get('ResponseMessage') or response.get('errorMessage') or default_error
        return {'error': error}

    def pine_labs_cancel_payment_request(self, data):
        """
        Cancels a payment request via Pine Labs POS API.

        :param dict data: Contains `amount` and `plutusTransactionReferenceID`.
        :return: Success response with `responseCode` and `notification` or error with `errorMessage`.
        :rtype: dict
        """
        body = {
            'Amount': data['amount'],
            'PlutusTransactionReferenceID': data['plutusTransactionReferenceID'],
            'TakeToHomeScreen': True,
            'ConfirmationRequired': True
        }
        response = call_pine_labs(payment_method=self, endpoint='CancelTransactionForced', payload=body)
        if response.get('ResponseCode') == 0 and response.get('ResponseMessage') == "APPROVED":
            return {
                'responseCode': response['ResponseCode'],
                'notification': _('Pine Labs POS transaction cancelled. Retry again for collecting payment.')
            }
        default_error = _('The expected error code for the Pine Labs POS status request was not included in the response.')
        error = response.get('ResponseMessage') or response.get('errorMessage') or default_error
        return { 'error': error }

    @api.constrains('use_payment_terminal')
    def _check_pine_labs_terminal(self):
        if any(record.use_payment_terminal == 'pine_labs' and record.company_id.currency_id.name != 'INR' for record in self):
            raise UserError(_('This Payment Terminal is only valid for INR Currency'))

```

## File: models\__init__.py

```python
from . import pos_payment_method
from . import pos_payment

```

## File: static\src\app\utils\payment\payment_pine_labs.js

```javascript
import { _t } from "@web/core/l10n/translation";
import { PaymentInterface } from "@point_of_sale/app/payment/payment_interface";
import { AlertDialog } from "@web/core/confirmation_dialog/confirmation_dialog";
import { serializeDateTime } from "@web/core/l10n/dates";
import { offlineErrorHandler, handleRPCError } from "@point_of_sale/app/errors/error_handlers";
import { register_payment_method } from "@point_of_sale/app/store/pos_store";
import { ConnectionLostError, RPCError } from "@web/core/network/rpc";

const REQUEST_TIMEOUT_MS = 5000;
const CANCEL_REQUEST_TIME_LIMIT_MS = 125000;
const { DateTime } = luxon;

export class PaymentPineLabs extends PaymentInterface {
    setup() {
        super.setup(...arguments);
        this.pollingTimeout = null;
        this.inactivityTimeout = null;
        this.payment_stopped = false;
    }

    send_payment_request(cid) {
        super.send_payment_request(cid);
        return this._processPineLabs();
    }

    pendingPineLabsPaymentLine() {
        return this.pos.getPendingPaymentLine("pine_labs");
    }

    send_payment_cancel(order, cid) {
        super.send_payment_cancel(order, cid);
        return this._pineLabsCancel();
    }

    _callPineLabs(data, action) {
        return this.pos.data
            .call("pos.payment.method", action, [[this.payment_method_id.id], data])
            .catch((error) => {
                const line = this.pendingPineLabsPaymentLine();
                this.pos.paymentTerminalInProgress = false;
                if (line) {
                    line.set_payment_status("force_done");
                }
                if (error instanceof ConnectionLostError) {
                    offlineErrorHandler(this.env, error, error);
                } else if (error instanceof RPCError) {
                    handleRPCError(error, this.env.services.dialog);
                } else {
                    throw error;
                }
            });
    }

    /**
     * Handles the response from Pine Labs for a make payment request.
     * @param {Object} response - The response received from Pine Labs.
     * @returns {Promise<Object|boolean>} - Resolves when the payment confirmation process completes.
     */
    async _makePaymentRequestHandler(response) {
        const line = this.pendingPineLabsPaymentLine();
        if (!response || response?.error) {
            line.set_payment_status("retry");
            this._showError(response?.error || _t("Pine Labs make payment request failed"));
            return false;
        }

        line.set_payment_status("waitingCard");
        line.update({ pine_labs_plutus_transaction_ref: response.plutusTransactionReferenceID });
        return await this._waitForPaymentToConfirm();
    }
    /**
     * Handles the response from Pine Labs for a payment status request.
     * @param {Object} response - The response received from Pine Labs.
     * @param {Function} callBack - The function to call for retrying the status request.
     * @param {Function} resolve - The function to resolve the promise.
     * @param {Function} reject - The function to reject the promise.
     * @returns {Promise<Object|boolean>} - Resolves with the response object on success, otherwise `false`.
     */
    _paymentStatusRequestHandler(response, callBack, resolve, reject) {
        const line = this.pendingPineLabsPaymentLine();

        if (!response || response?.error) {
            const status = response ? "retry" : "force_done";
            line.set_payment_status(status);
            this._showError(response?.error || _t("Pine Labs get payment status request failed"));
            if (response) {
                return resolve(false);
            }
        }
        const resultStatus = response?.status;
        if (resultStatus === "TXN UPLOADED") {
            this.pollingTimeout = setTimeout(callBack, REQUEST_TIMEOUT_MS, resolve, reject);
            return;
        } else if (
            resultStatus === "TXN APPROVED" &&
            response?.plutusTransactionReferenceID !==
                parseInt(line.pine_labs_plutus_transaction_ref)
        ) {
            response.error = _t("Reference number mismatched");
        } else if (resultStatus === "TXN APPROVED") {
            const data = response.data;
            line.update({
                payment_method_issuer_bank: data["Acquirer Name"],
                payment_method_authcode: data["ApprovalCode"],
                cardholder_name: data["Card Holder Name"],
                card_no: data["Card Number"]?.slice(-4) || "",
                card_brand: data["Card Type"],
                payment_method_payment_mode: data["PaymentMode"],
                transaction_id: data["TransactionLogId"],
                payment_date: this._getPaymentDate(
                    data["Transaction Date"],
                    data["Transaction Time"]
                ),
            });
            this._removePaymentHandler();
            return resolve(response);
        }
    }
    /**
     * Handles the response from Pine Labs for a payment cancellation request.
     * @param {Object} response - The response received from Pine Labs.
     * @returns {boolean} - Returns `true` if a notification is processed, otherwise `false`.
     */
    _paymentCancelRequestHandler(response) {
        const line = this.pendingPineLabsPaymentLine();
        if (!response || response?.error) {
            this._showError(response?.error || _t("Pine Labs payment cancellation request failed"));
            return false;
        } else if (response.notification) {
            line.set_payment_status("retry");
            if (this.payment_stopped) {
                this._showError(_t("Transaction failed due to inactivity"));
            } else {
                this.pos.notification.add(response.notification, {
                    type: "warning",
                    sticky: false,
                });
            }
            this._removePaymentHandler();
            return true;
        } else {
            return false;
        }
    }

    async _pineLabsCancel() {
        const paymentLine = this.pendingPineLabsPaymentLine();
        const data = {
            plutusTransactionReferenceID: paymentLine.pine_labs_plutus_transaction_ref,
            amount: paymentLine.amount * 100, // We need to provide the amount in paisa since Pine Labs processes amounts in paisa.
        };

        const response = await this._callPineLabs(data, "pine_labs_cancel_payment_request");

        return this._paymentCancelRequestHandler(response);
    }

    /**
     * This method processes order data and sends payment requests from POS to Pine Labs.
     */
    async _processPineLabs() {
        const order = this.pos.get_order();
        const paymentLine = order.get_selected_paymentline();
        const sequenceNumber = order.payment_ids.filter(
            (pi) => pi.payment_method_id.use_payment_terminal === "pine_labs"
        ).length;
        if (paymentLine.amount < 0) {
            this._showError(_t("Cannot process transactions with negative amount."));
            return false;
        }

        const orderId = order?.pos_reference?.replace(" ", "").replaceAll("-", "").toUpperCase();
        const referencePrefix = this.pos.config.name.replace(/\s/g, "").slice(0, 4);
        paymentLine.update({
            payment_ref_no:
                referencePrefix + "/" + orderId + "/" + crypto.randomUUID().replaceAll("-", ""),
        });

        // Assume that the Pine Labs terminal payment method is configured with INR (Indian Rupees) as the currency_id in the POS config.
        // The conversion rate between INR and paisa is set as 1 INR = 100 paisa.
        const data = {
            amount: paymentLine.amount * 100, // We need to provide the amount in paisa since Pine Labs processes amounts in paisa.
            transactionNumber: paymentLine.payment_ref_no,
            sequenceNumber: sequenceNumber, // In the case of multiple transactions for the same order, it is important to follow the correct sequence of transactions.
        };
        const response = await this._callPineLabs(data, "pine_labs_make_payment_request");
        return await this._makePaymentRequestHandler(response);
    }

    /**
     * This method waits for the payment to be confirmed by Pine Labs.
     * Also, this method uses polling to check the payment status..
     */
    async _waitForPaymentToConfirm() {
        const paymentLine = this.pos.get_order().get_selected_paymentline();
        if (!paymentLine || paymentLine.payment_status == "retry") {
            return false;
        }
        const data = {
            plutusTransactionReferenceID: paymentLine.pine_labs_plutus_transaction_ref,
        };
        this._stopPendingPayment().then(() => (this.payment_stopped = true));
        const pineLabsFetchPaymentStatus = async (resolve, reject) => {
            //Clear the previous timeout before setting a new one
            clearTimeout(this.pollingTimeout);

            // If the user navigates to another screen, stop the polling
            if (this.pos.mainScreen.component.name !== "PaymentScreen") {
                this._removePaymentHandler();
                return;
            }

            if (this.payment_stopped) {
                this._pineLabsCancel().then(() => {
                    paymentLine.set_payment_status("retry");
                    this.payment_stopped = false;
                });
                return resolve(false);
            }

            if (paymentLine.payment_status == "retry") {
                return resolve(false);
            }
            const response = await this._callPineLabs(data, "pine_labs_fetch_payment_status");
            return this._paymentStatusRequestHandler(
                response,
                pineLabsFetchPaymentStatus,
                resolve,
                reject
            );
        };
        return new Promise(pineLabsFetchPaymentStatus);
    }

    _getPaymentDate(dateString, timeString) {
        // The dateString value appears as `03122024`, while the timeString value appears as `063515`.
        const localDate = DateTime.fromFormat(`${dateString} ${timeString}`, "ddMMyyyy HHmmss");
        return serializeDateTime(localDate);
    }

    _stopPendingPayment() {
        return new Promise(
            (resolve) =>
                (this.inactivityTimeout = setTimeout(resolve, CANCEL_REQUEST_TIME_LIMIT_MS))
        );
    }

    _removePaymentHandler() {
        clearTimeout(this.pollingTimeout);
        clearTimeout(this.inactivityTimeout);
        this.payment_stopped = false;
    }

    _showError(error_msg) {
        this.env.services.dialog.add(AlertDialog, {
            title: _t("Pine Labs Error"),
            body: error_msg,
        });
    }
}

register_payment_method("pine_labs", PaymentPineLabs);

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
            const waitingPaymentLine = this.currentOrder.payment_ids.find(
                (paymentLine) =>
                    paymentLine.payment_method_id.use_payment_terminal === "pine_labs" &&
                    !paymentLine.is_done() &&
                    paymentLine.get_payment_status() !== "pending"
            );
            if (waitingPaymentLine) {
                const payment_status =
                    await waitingPaymentLine.payment_method_id.payment_terminal._waitForPaymentToConfirm();
                if (payment_status?.status === "TXN APPROVED") {
                    waitingPaymentLine.set_payment_status("done");
                    this.pos.paymentTerminalInProgress = false;
                } else if (payment_status?.status === "TXN UPLOADED") {
                    waitingPaymentLine.set_payment_status("waitingCard");
                } else {
                    waitingPaymentLine.set_payment_status("retry");
                    this.pos.paymentTerminalInProgress = false;
                }
            }
        });
    },
});

```

## File: views\pos_order_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_pos_pos_form_inherit_pos_pine_labs" model="ir.ui.view">
        <field name="name">pos.order.form.inherit.pos.pine.labs</field>
        <field name="model">pos.order</field>
        <field name="inherit_id" ref="point_of_sale.view_pos_pos_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='payment_ids']//list//field[@name='cardholder_name']" position="after">
                <field name="pine_labs_plutus_transaction_ref" optional="hide"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\pos_payment_method_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="pos_payment_method_view_form_inherit_pos_pine_labs" model="ir.ui.view">
        <field name="name">pos.payment.method.form.inherit.pos.pine.labs</field>
        <field name="model">pos.payment.method</field>
        <field name="inherit_id" ref="point_of_sale.pos_payment_method_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='use_payment_terminal']" position="after">
                <field name="pine_labs_merchant" invisible="use_payment_terminal != 'pine_labs'" required="use_payment_terminal == 'pine_labs'"/>
                <field name="pine_labs_store" invisible="use_payment_terminal != 'pine_labs'" required="use_payment_terminal == 'pine_labs'"/>
                <field name="pine_labs_client" invisible="use_payment_terminal != 'pine_labs'" required="use_payment_terminal == 'pine_labs'"/>
                <field name="pine_labs_security_token" invisible="use_payment_terminal != 'pine_labs'" required="use_payment_terminal == 'pine_labs'" password="True"/>
                <field name="pine_labs_allowed_payment_mode" invisible="use_payment_terminal != 'pine_labs'" required="use_payment_terminal == 'pine_labs'"/>
                <field name="pine_labs_test_mode" invisible="use_payment_terminal != 'pine_labs'" required="use_payment_terminal == 'pine_labs'"/>
            </xpath>
        </field>
    </record>
</odoo>

```

