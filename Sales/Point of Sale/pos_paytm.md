# Odoo Module: pos_paytm

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
    'name': 'POS PayTM',
    'version': '1.0',
    'category': 'Sales/Point of Sale',
    'sequence': 6,
    'summary': 'Integrate your POS with a PayTM payment terminal',
    'description': """
Allow Paytm POS payments
==============================

This module allows customers to pay for their orders with debit/credit
cards and UPI. The transactions are processed by Paytm POS. A Paytm merchant account is necessary. It allows the
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
            'pos_paytm/static/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\neutralize.sql

```sql
UPDATE pos_payment_method
SET paytm_test_mode = true,
    paytm_merchant_key = 'dummykey00012345';

```

## File: models\pos_payment_method.py

```python
# coding: utf-8
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import base64
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
import hashlib
import logging
import requests
import secrets
import string

from odoo.exceptions import UserError
from odoo import fields, models, api, _
from datetime import datetime
from dateutil import tz

_logger = logging.getLogger(__name__)
REQUEST_TIMEOUT = 30
iv = b'@@@@&&&&####$$$$'

class PosPaymentMethod(models.Model):
    _inherit = 'pos.payment.method'

    paytm_tid = fields.Char(string='PayTM Terminal ID', help="Terminal model or Activation code \n ex: 70000123")
    channel_id = fields.Char(string='PayTM Channel ID', default='EDC')
    accept_payment = fields.Selection(selection=[('auto', 'Automatically'), ('manual', 'Manually')], default='auto', help="Choose accept payment mode: \n Manually or Automatically")
    allowed_payment_modes = fields.Selection(selection=[('all', 'All'), ('card', 'Card'), ('qr', 'QR')], default='all', help="Choose allow payment mode: \n All/Card or QR")
    paytm_mid = fields.Char(string="PayTM Merchant ID", help="Go to https://business.paytm.com/ and create the merchant account")
    paytm_merchant_key = fields.Char(string="PayTM Merchant API Key", help="Merchant/AES key \n ex: B1o6Ivjy8L1@abc9")
    paytm_test_mode = fields.Boolean(string="PayTM Test Mode", default=False, help="Turn it on when in Test Mode")

    def _get_payment_terminal_selection(self):
        return super()._get_payment_terminal_selection() + [('paytm', 'PayTM')]

    def _paytm_make_request(self, url, payload=None):
        """ Make a request to PayTM API.

        :param str url: The url to be reached by the request.
        :param dict payload: The payload of the request.
        :return The JSON-formatted content of the response.
        :rtype: dict
        """
        try:
            if self.paytm_test_mode:
                api_url = 'https://securegw-stage.paytm.in/ecr/'
            else:
                api_url = 'https://securegw-edc.paytm.in/ecr/'
            response = requests.post(api_url+url, json=payload, timeout=REQUEST_TIMEOUT)
            response.raise_for_status()
        except (requests.exceptions.Timeout, requests.exceptions.RequestException) as error:
            _logger.warning("Cannot connect with PayTM. Error: %s", error)
            return {'error': '%s' % error}
        res_json = response.json()
        if res_json.get('body'):
            return res_json['body']
        default_error_msg = _('Something went wrong with paytm request. Please try later.')
        error = res_json.get('error') or default_error_msg
        return {'error': '%s' % error}

    def paytm_make_payment_request(self, amount, transaction_id, reference_id, timestamp):
        body = self._paytm_get_request_body(transaction_id, reference_id, timestamp)
        body['transactionAmount'] = str(int(amount))
        if self.accept_payment == 'auto':
            body['autoAccept'] = 'True'
        body['paymentMode'] = self.allowed_payment_modes.upper()
        head = self._paytm_get_request_head(body)
        head_error = head.get('error')
        if head_error:
            return {'error': '%s' % head_error}
        merchantExtendedInfo = {'paymentMode': self.allowed_payment_modes.upper()}
        if self.accept_payment == 'auto':
            merchantExtendedInfo['autoAccept'] = 'True'
        body['merchantExtendedInfo'] = merchantExtendedInfo
        payload = {'head': head, 'body': body}
        response = self._paytm_make_request('payment/request', payload=payload)
        result_code = response.get('resultInfo', {}).get('resultCode')
        if result_code == 'A':
            return response['resultInfo']
        elif result_code == 'F':
            return {'error': "%s" % response['resultInfo'].get('resultMsg', _('paytm transaction request declined'))}
        default_error_msg = _('makePaymentRequest expected resultCode not found in the response')
        error = response.get('error') or default_error_msg
        return {'error': '%s' % error}

    def paytm_fetch_payment_status(self, transaction_id, reference_id, timestamp):
        body = self._paytm_get_request_body(transaction_id, reference_id, timestamp)
        head = self._paytm_get_request_head(body)
        head_error = head.get('error') and head
        if head_error:
            return head_error
        payload = {'head': head, 'body': body}
        response = self._paytm_make_request('V2/payment/status', payload=payload)
        result_code = response.get('resultInfo', {}).get('resultCode')
        if result_code == 'S':
            # Since we don't want to send extra data on RPC call
            # Only sending essential data when the transaction is successful
            data = response['resultInfo']
            data.update({
                    'authCode': response.get('authCode'),
                    'issuerMaskCardNo': response.get('issuerMaskCardNo'),
                    'issuingBankName': response.get('issuingBankName'),
                    'payMethod': response.get('payMethod'),
                    'cardType': response.get('cardType'),
                    'cardScheme': response.get('cardScheme'),
                    'merchantReferenceNo': response.get('merchantReferenceNo'),
                    'merchantTransactionId': response.get('merchantTransactionId'),
                    'transactionDateTime': response.get('transactionDateTime'),
            })
            return data
        elif result_code == 'F':
            return {'error': "%s" % response['resultInfo'].get('resultMsg', _('paytm transaction failure'))}
        elif result_code == 'P':
            return response['resultInfo']
        default_error_msg = _('paymentFetchRequest expected resultCode not found in the response')
        error = response.get('error') or default_error_msg
        return {'error': '%s' % error}

    def _paytm_generate_signature(self, params_dict, key):
        params_list = []
        for k in sorted(params_dict.keys()):
            value = params_dict[k]
            if value is None or params_dict[k].lower() == "null":
                value = ""
            params_list.append(str(value))
        salt = ''.join(secrets.choice(string.ascii_letters + string.digits) for _ in range(4))
        params_list.append(salt)
        params_with_salt = '|'.join(params_list)
        hashed_params = hashlib.sha256(params_with_salt.encode())
        hashed_params_with_salt = hashed_params.hexdigest() + salt
        padding = 12 #the padding value is a constant
        padded_hashed_params_with_salt = bytes(hashed_params_with_salt + padding * chr(padding), 'utf-8')
        try:
            cipher = Cipher(algorithms.AES(key.encode()), modes.CBC(iv))
            encryptor = cipher.encryptor()
            encrypted_hashed_params = encryptor.update(padded_hashed_params_with_salt) + encryptor.finalize()
            return base64.b64encode(encrypted_hashed_params).decode("UTF-8")
        except ValueError as error:
            _logger.warning("Cannot generate PayTM signature. Error: %s", error)
            return {'error': '%s' % error}

    def _paytm_get_request_body(self, transaction_id, reference_id, timestamp):

        time = datetime.fromtimestamp(timestamp).astimezone(tz=tz.gettz('Asia/Kolkata')).strftime("%Y-%m-%d %H:%M:%S")
        return {
            'paytmMid': self.paytm_mid,
            'paytmTid': self.paytm_tid,
            'transactionDateTime': time,
            'merchantTransactionId': transaction_id,
            'merchantReferenceNo': reference_id,
        }

    def _paytm_get_request_head(self, body):
        paytm_signature = self._paytm_generate_signature(body, self.paytm_merchant_key)
        error = isinstance(paytm_signature, dict) and paytm_signature.get('error')
        if error:
            return {'error': '%s' % error}
        return {
            'requestTimeStamp' : body["transactionDateTime"],
            'channelId' : self.channel_id,
            'checksum' : paytm_signature,
        }

    @api.constrains('use_payment_terminal')
    def _check_paytm_terminal(self):
        for record in self:
            if record.use_payment_terminal == 'paytm' and record.company_id.currency_id.name != 'INR':
                raise UserError(_('This Payment Terminal is only valid for INR Currency'))

```

## File: models\__init__.py

```python
from . import pos_payment_method

```

## File: static\src\js\model.js

```javascript
import { register_payment_method } from "@point_of_sale/app/store/pos_store";
import { PaymentPaytm } from "@pos_paytm/js/payment_paytm";

register_payment_method("paytm", PaymentPaytm);

```

## File: static\src\js\PaymentScreen.js

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
                    paymentLine.payment_method_id.use_payment_terminal === "paytm" &&
                    !paymentLine.is_done() &&
                    paymentLine.get_payment_status() !== "pending"
            );
            if (pendingPaymentLine) {
                pendingPaymentLine.set_payment_status("force_done");
            }
        });
    },
});

```

## File: static\src\js\payment_paytm.js

```javascript
import { _t } from "@web/core/l10n/translation";
import { PaymentInterface } from "@point_of_sale/app/payment/payment_interface";
import { AlertDialog } from "@web/core/confirmation_dialog/confirmation_dialog";

const REQUEST_TIMEOUT = 5000;

export class PaymentPaytm extends PaymentInterface {
    /**
     * @Override
     * @param { string } uuid
     * @returns Promise
     */
    async send_payment_request(uuid) {
        await super.send_payment_request(...arguments);
        const paymentLine = this.pos.get_order()?.get_selected_paymentline();
        const order = this.pos?.get_order();
        const retry = this._retryCountUtility(order.uuid);
        let transactionId = order.name.replace(" ", "").replaceAll("-", "").toUpperCase();
        if (retry > 0) {
            transactionId = transactionId.concat("retry", retry);
        }
        const transactionAmount = paymentLine.amount * 100;
        const timeStamp = Math.floor(Date.now() / 1000);

        // Preparing Unique Random Reference Id
        const referencePrefix = this.pos.config.name.replace(/\s/g, "").slice(0, 4);
        const referenceId = referencePrefix.concat(Math.floor(Math.random() * 1000000000));
        const response = await this.makePaymentRequest(
            transactionAmount,
            transactionId,
            referenceId,
            timeStamp
        );
        if (!response) {
            paymentLine.set_payment_status("force_done");
            this._incrementRetry(order.uuid);
            return false;
        }
        paymentLine.set_payment_status("waitingCard");
        const pollResponse = await this.pollPayment(transactionId, referenceId, timeStamp);
        if (pollResponse) {
            const retry_remove = true;
            this._retryCountUtility(order.uuid, retry_remove);
            return true;
        } else {
            this._incrementRetry(order.uuid);
            return false;
        }
    }

    /**
     * @Override
     * @param {} order
     * @param { string } uuid
     * @returns Promise
     */
    async send_payment_cancel(order, uuid) {
        await super.send_payment_cancel(...arguments);
        const paymentLine = this.pos.get_order()?.get_selected_paymentline();
        paymentLine.set_payment_status("retry");
        this._incrementRetry(order.uuid);
        clearTimeout(this.pollTimeout);
        return true;
    }

    /**
     * @param { string } transactionId
     * @param { string } referenceId
     * @param { datetime } timestamp
     * @returns Promise
     */
    async pollPayment(transactionId, referenceId, timestamp) {
        const fetchPaymentStatus = async (resolve, reject) => {
            const paymentLine = this.pos.get_order()?.get_selected_paymentline();
            if (!paymentLine || paymentLine.payment_status == "retry") {
                return false;
            }
            try {
                const data = await this.pos.data.silentCall(
                    "pos.payment.method",
                    "paytm_fetch_payment_status",
                    [[this.payment_method_id.id], transactionId, referenceId, timestamp]
                );
                if (data?.error) {
                    throw data?.error;
                }
                const resultCode = data?.resultCode;
                if (resultCode === "S" && data?.merchantReferenceNo != referenceId) {
                    throw _t("Reference number mismatched");
                } else if (resultCode === "S") {
                    paymentLine.payment_method_authcode = data?.authCode;
                    paymentLine.card_no = data?.issuerMaskCardNo;
                    paymentLine.payment_method_issuer_bank = data?.issuingBankName;
                    paymentLine.payment_method_payment_mode = data?.payMethod;
                    paymentLine.card_type = data?.cardType;
                    paymentLine.card_brand = data?.cardScheme;
                    paymentLine.payment_ref_no = data?.merchantReferenceNo;
                    paymentLine.transaction_id = data?.merchantTransactionId;
                    paymentLine.payment_date = data?.transactionDateTime;
                    return resolve(data);
                } else {
                    this.pollTimeout = setTimeout(
                        fetchPaymentStatus,
                        REQUEST_TIMEOUT,
                        resolve,
                        reject
                    );
                }
            } catch (error) {
                const order = this.pos.get_order();
                this._incrementRetry(order.uuid);
                paymentLine.set_payment_status("force_done");
                this._showError(error, "paytmFetchPaymentStatus");
                return resolve(false);
            }
        };
        return new Promise(fetchPaymentStatus);
    }
    /**
     * @param { float } amount
     * @param { string } transactionId
     * @param { string } referenceId
     * @param { datetime } timestamp
     * @returns Promise
     */
    async makePaymentRequest(amount, transactionId, referenceId, timestamp) {
        try {
            const data = await this.pos.data.silentCall(
                "pos.payment.method",
                "paytm_make_payment_request",
                [[this.payment_method_id.id], amount, transactionId, referenceId, timestamp]
            );
            if (data?.error) {
                throw data.error;
            }
            return data;
        } catch (error) {
            this._showError(error, "paytmMakePaymentRequest");
            return false;
        }
    }

    // ---------------------------------------------------------------------------
    // Private methods
    // ---------------------------------------------------------------------------

    _retryCountUtility(uuid, remove = false) {
        if (remove) {
            localStorage.removeItem(uuid);
        } else {
            return localStorage.getItem(uuid) || (localStorage.setItem(uuid, 0) && 0);
        }
    }

    _incrementRetry(uuid) {
        let retry = localStorage.getItem(uuid);
        localStorage.setItem(uuid, ++retry);
    }
    _showError(error_msg, title) {
        this.env.services.dialog.add(AlertDialog, {
            title: title || _t("PayTM Error"),
            body: error_msg,
        });
    }
}

```

## File: views\pos_payment_method_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="pos_payment_method_view_form_inherit_pos_paytm" model="ir.ui.view">
        <field name="name">pos.payment.method.form.inherit.paytm</field>
        <field name="model">pos.payment.method</field>
        <field name="inherit_id" ref="point_of_sale.pos_payment_method_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='use_payment_terminal']" position="after">
                <field name="paytm_mid" invisible="use_payment_terminal != 'paytm'" required="use_payment_terminal == 'paytm'"/>
                <field name="paytm_tid" invisible="use_payment_terminal != 'paytm'" required="use_payment_terminal == 'paytm'"/>
                <field name="paytm_merchant_key" invisible="use_payment_terminal != 'paytm'" required="use_payment_terminal == 'paytm'"/>
                <field name="channel_id" groups="base.group_no_one" invisible="use_payment_terminal != 'paytm'" required="use_payment_terminal == 'paytm'"/>
                <field name="accept_payment" invisible="use_payment_terminal != 'paytm'" required="use_payment_terminal == 'paytm'"/>
                <field name="allowed_payment_modes" invisible="use_payment_terminal != 'paytm'" required="use_payment_terminal == 'paytm'"/>
                <field name="paytm_test_mode" invisible="use_payment_terminal != 'paytm'" required="use_payment_terminal == 'paytm'"/>
            </xpath>
        </field>
    </record>
</odoo>

```

