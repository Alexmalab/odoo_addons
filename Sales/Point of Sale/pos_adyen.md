# Odoo Module: pos_adyen

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
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'POS Adyen',
    'version': '1.0',
    'category': 'Sales/Point of Sale',
    'sequence': 6,
    'summary': 'Integrate your POS with an Adyen payment terminal',
    'data': [
        'views/res_config_settings_views.xml',
        'views/pos_payment_method_views.xml',
    ],
    'depends': ['point_of_sale'],
    'installable': True,
    'assets': {
        'point_of_sale._assets_pos': [
            'pos_adyen/static/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# coding: utf-8
import logging
import pprint
import json
from urllib.parse import parse_qs
from odoo import http
from odoo.http import request
from odoo.tools import consteq

_logger = logging.getLogger(__name__)


class PosAdyenController(http.Controller):

    @http.route('/pos_adyen/notification', type='json', methods=['POST'], auth='public', csrf=False, save_session=False)
    def notification(self):
        data = json.loads(request.httprequest.data)

        # ignore if it's not a response to a sales request
        if not data.get('SaleToPOIResponse'):
            return

        _logger.info('notification received from adyen:\n%s', pprint.pformat(data))

        msg_header = data['SaleToPOIResponse'].get('MessageHeader')
        if not msg_header \
            or msg_header.get('ProtocolVersion') != '3.0' \
            or msg_header.get('MessageClass') != 'Service' \
            or msg_header.get('MessageType') != 'Response' \
            or msg_header.get('MessageCategory') != 'Payment' \
            or not msg_header.get('POIID'):
            _logger.warning('Received an unexpected Adyen notification')
            return

        terminal_identifier = msg_header['POIID']
        adyen_pm_sudo = request.env['pos.payment.method'].sudo().search([('adyen_terminal_identifier', '=', terminal_identifier)], limit=1)
        if not adyen_pm_sudo:
            _logger.warning('Received an Adyen event notification for a terminal not registered in Odoo: %s', terminal_identifier)
            return

        try:
            adyen_additional_response = data['SaleToPOIResponse']['PaymentResponse']['Response']['AdditionalResponse']
            pos_hmac = PosAdyenController._get_additional_data_from_unparsed(adyen_additional_response, 'metadata.pos_hmac')

            if not pos_hmac or not consteq(pos_hmac, adyen_pm_sudo._get_hmac(msg_header['SaleID'], msg_header['ServiceID'], msg_header['POIID'], data['SaleToPOIResponse']['PaymentResponse']['SaleData']['SaleTransactionID']['TransactionID'])):
                _logger.warning('Received an invalid Adyen event notification (invalid hmac): \n%s', pprint.pformat(data))
                return

            # The HMAC is removed to prevent anyone from using it in place of Adyen.
            pos_hmac_metadata_raw = 'metadata.pos_hmac='+pos_hmac
            safe_additional_response = adyen_additional_response.replace('&'+pos_hmac_metadata_raw, '').replace(pos_hmac_metadata_raw, '')
            data['SaleToPOIResponse']['PaymentResponse']['Response']['AdditionalResponse'] = safe_additional_response
        except (KeyError, AttributeError):
            _logger.warning('Received an invalid Adyen event notification: \n%s', pprint.pformat(data))
            return

        return self._process_payment_response(data, adyen_pm_sudo)

    @staticmethod
    def _get_additional_data_from_unparsed(adyen_additional_response, data_key):
        parsed_adyen_additional_response = parse_qs(adyen_additional_response)
        return PosAdyenController._get_additional_data_from_parsed(parsed_adyen_additional_response, data_key)

    @staticmethod
    def _get_additional_data_from_parsed(parsed_adyen_additional_response, data_key):
        data_value = parsed_adyen_additional_response.get(data_key)
        return data_value[0] if data_value and len(data_value) == 1 else None

    def _process_payment_response(self, data, adyen_pm_sudo):
        transaction_id = data['SaleToPOIResponse']['PaymentResponse']['SaleData']['SaleTransactionID']['TransactionID']
        if not transaction_id:
            return
        transaction_id_parts = transaction_id.split("--")
        if len(transaction_id_parts) != 2:
            return
        pos_session_id = int(transaction_id_parts[1])
        pos_session_sudo = request.env["pos.session"].sudo().browse(pos_session_id)
        adyen_pm_sudo.adyen_latest_response = json.dumps(data)
        pos_session_sudo.config_id._notify("ADYEN_LATEST_RESPONSE", pos_session_sudo.config_id.id)
        return request.make_json_response('[accepted]') # https://docs.adyen.com/point-of-sale/design-your-integration/choose-your-architecture/cloud/#guarantee

```

## File: controllers\__init__.py

```python
# coding: utf-8
from . import main

```

## File: data\neutralize.sql

```sql
-- disable Adyen Payement POS integration
UPDATE pos_payment_method
   SET adyen_test_mode = true;

```

## File: models\pos_config.py

```python
# coding: utf-8
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import logging

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError

_logger = logging.getLogger(__name__)


class PosConfig(models.Model):
    _inherit = 'pos.config'

    adyen_ask_customer_for_tip = fields.Boolean('Ask Customers For Tip')

    @api.constrains('adyen_ask_customer_for_tip', 'iface_tipproduct', 'tip_product_id')
    def _check_adyen_ask_customer_for_tip(self):
        for config in self:
            if config.adyen_ask_customer_for_tip and (not config.tip_product_id or not config.iface_tipproduct):
                raise ValidationError(_("Please configure a tip product for POS %s to support tipping with Adyen.", config.name))

```

## File: models\pos_payment_method.py

```python
# coding: utf-8
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import json
import logging
import pprint
import requests
from urllib.parse import parse_qs

from odoo import fields, models, api, _
from odoo.exceptions import ValidationError, UserError, AccessDenied
from odoo.tools import hmac

_logger = logging.getLogger(__name__)

UNPREDICTABLE_ADYEN_DATA = object() # sentinel

class PosPaymentMethod(models.Model):
    _inherit = 'pos.payment.method'

    def _get_payment_terminal_selection(self):
        return super(PosPaymentMethod, self)._get_payment_terminal_selection() + [('adyen', 'Adyen')]

    # Adyen
    adyen_api_key = fields.Char(string="Adyen API key", help='Used when connecting to Adyen: https://docs.adyen.com/user-management/how-to-get-the-api-key/#description', copy=False, groups='base.group_erp_manager')
    adyen_terminal_identifier = fields.Char(help='[Terminal model]-[Serial number], for example: P400Plus-123456789', copy=False)
    adyen_test_mode = fields.Boolean(help='Run transactions in the test environment.', groups='base.group_erp_manager')

    adyen_latest_response = fields.Char(copy=False, groups='base.group_erp_manager') # used to buffer the latest asynchronous notification from Adyen.
    adyen_event_url = fields.Char(
        string="Event URL",
        help="This URL needs to be pasted on Adyen's portal terminal settings.",
        readonly=True,
        store=False,
        default=lambda self: f"{self.get_base_url()}/pos_adyen/notification",
    )

    @api.model
    def _load_pos_data_fields(self, config_id):
        params = super()._load_pos_data_fields(config_id)
        params += ['adyen_terminal_identifier']
        return params

    @api.constrains('adyen_terminal_identifier')
    def _check_adyen_terminal_identifier(self):
        for payment_method in self:
            if not payment_method.adyen_terminal_identifier:
                continue
            # sudo() to search all companies
            existing_payment_method = self.sudo().search([('id', '!=', payment_method.id),
                                                   ('adyen_terminal_identifier', '=', payment_method.adyen_terminal_identifier)],
                                                  limit=1)
            if existing_payment_method:
                if existing_payment_method.company_id == payment_method.company_id:
                    raise ValidationError(_('Terminal %(terminal)s is already used on payment method %(payment_method)s.',
                                      terminal=payment_method.adyen_terminal_identifier, payment_method=existing_payment_method.display_name))
                else:
                    raise ValidationError(_('Terminal %(terminal)s is already used in company %(company)s on payment method %(payment_method)s.',
                                             terminal=payment_method.adyen_terminal_identifier,
                                             company=existing_payment_method.company_id.name,
                                             payment_method=existing_payment_method.display_name))

    def _get_adyen_endpoints(self):
        return {
            'terminal_request': 'https://terminal-api-%s.adyen.com/async',
        }

    def _is_write_forbidden(self, fields):
        return super(PosPaymentMethod, self)._is_write_forbidden(fields - {'adyen_latest_response'})

    def get_latest_adyen_status(self):
        self.ensure_one()
        if not self.env.su and not self.env.user.has_group('point_of_sale.group_pos_user'):
            raise AccessDenied()

        latest_response = self.sudo().adyen_latest_response
        latest_response = json.loads(latest_response) if latest_response else False
        return latest_response

    def proxy_adyen_request(self, data, operation=False):
        ''' Necessary because Adyen's endpoints don't have CORS enabled '''
        self.ensure_one()
        if not self.env.su and not self.env.user.has_group('point_of_sale.group_pos_user'):
            raise AccessDenied()
        if not data:
            raise UserError(_('Invalid Adyen request'))

        if 'SaleToPOIRequest' in data and data['SaleToPOIRequest']['MessageHeader']['MessageCategory'] == 'Payment' and 'PaymentRequest' in data['SaleToPOIRequest']:  # Clear only if it is a payment request
            self.sudo().adyen_latest_response = ''  # avoid handling old responses multiple times

        if not operation:
            operation = 'terminal_request'

        # These checks are not optimal. This RPC method should be changed.

        is_capture_data = operation == 'capture' and hasattr(self, 'adyen_merchant_account') and self._is_valid_adyen_request_data(data, {
            'originalReference': UNPREDICTABLE_ADYEN_DATA,
            'modificationAmount': {
                'value': UNPREDICTABLE_ADYEN_DATA,
                'currency': UNPREDICTABLE_ADYEN_DATA,
            },
            'merchantAccount': self.adyen_merchant_account,
        })

        is_adjust_data = operation == 'adjust' and hasattr(self, 'adyen_merchant_account') and self._is_valid_adyen_request_data(data, {
            'originalReference': UNPREDICTABLE_ADYEN_DATA,
            'modificationAmount': {
                'value': UNPREDICTABLE_ADYEN_DATA,
                'currency': UNPREDICTABLE_ADYEN_DATA,
            },
            'merchantAccount': self.adyen_merchant_account,
            'additionalData': {
                'industryUsage': 'DelayedCharge',
            },
        })

        is_cancel_data = operation == 'terminal_request' and self._is_valid_adyen_request_data(data, {
            'SaleToPOIRequest': {
                'MessageHeader': self._get_expected_message_header('Abort'),
                'AbortRequest': {
                    'AbortReason': 'MerchantAbort',
                    'MessageReference': {
                        'MessageCategory': 'Payment',
                        'SaleID': UNPREDICTABLE_ADYEN_DATA,
                        'ServiceID': UNPREDICTABLE_ADYEN_DATA,
                    },
                },
            },
        })

        is_payment_request_with_acquirer_data = operation == 'terminal_request' and self._is_valid_adyen_request_data(data, self._get_expected_payment_request(True))

        if is_payment_request_with_acquirer_data:
            parsed_sale_to_acquirer_data = parse_qs(data['SaleToPOIRequest']['PaymentRequest']['SaleData']['SaleToAcquirerData'])
            valid_acquirer_data = self._get_valid_acquirer_data()
            is_payment_request_with_acquirer_data = len(parsed_sale_to_acquirer_data.keys()) <= len(valid_acquirer_data.keys())
            if is_payment_request_with_acquirer_data:
                for key, values in parsed_sale_to_acquirer_data.items():
                    if len(values) != 1:
                        is_payment_request_with_acquirer_data = False
                        break
                    value = values[0]
                    valid_value = valid_acquirer_data.get(key)
                    if valid_value == UNPREDICTABLE_ADYEN_DATA:
                        continue
                    if value != valid_value:
                        is_payment_request_with_acquirer_data = False
                        break

        is_payment_request_without_acquirer_data = operation == 'terminal_request' and self._is_valid_adyen_request_data(data, self._get_expected_payment_request(False))

        if not is_payment_request_without_acquirer_data and not is_payment_request_with_acquirer_data and not is_adjust_data and not is_cancel_data and not is_capture_data:
            raise UserError(_('Invalid Adyen request'))

        if is_payment_request_with_acquirer_data or is_payment_request_without_acquirer_data:
            acquirer_data = data['SaleToPOIRequest']['PaymentRequest']['SaleData'].get('SaleToAcquirerData')
            msg_header = data['SaleToPOIRequest']['MessageHeader']
            metadata = 'metadata.pos_hmac=' + self._get_hmac(msg_header['SaleID'], msg_header['ServiceID'], msg_header['POIID'], data['SaleToPOIRequest']['PaymentRequest']['SaleData']['SaleTransactionID']['TransactionID'])

            data['SaleToPOIRequest']['PaymentRequest']['SaleData']['SaleToAcquirerData'] = acquirer_data + '&' + metadata if acquirer_data else metadata

        return self._proxy_adyen_request_direct(data, operation)

    @api.model
    def _is_valid_adyen_request_data(self, provided_data, expected_data):
        if not isinstance(provided_data, dict) or set(provided_data.keys()) != set(expected_data.keys()):
            return False

        for provided_key, provided_value in provided_data.items():
            expected_value = expected_data[provided_key]
            if expected_value == UNPREDICTABLE_ADYEN_DATA:
                continue
            if isinstance(expected_value, dict):
                if not self._is_valid_adyen_request_data(provided_value, expected_value):
                    return False
            else:
                if provided_value != expected_value:
                    return False
        return True

    def _get_expected_message_header(self, expected_message_category):
        return {
            'ProtocolVersion': '3.0',
            'MessageClass': 'Service',
            'MessageType': 'Request',
            'MessageCategory': expected_message_category,
            'SaleID': UNPREDICTABLE_ADYEN_DATA,
            'ServiceID': UNPREDICTABLE_ADYEN_DATA,
            'POIID': self.adyen_terminal_identifier,
        }

    def _get_expected_payment_request(self, with_acquirer_data):
        res = {
            'SaleToPOIRequest': {
                'MessageHeader': self._get_expected_message_header('Payment'),
                'PaymentRequest': {
                    'SaleData': {
                        'SaleTransactionID': {
                            'TransactionID': UNPREDICTABLE_ADYEN_DATA,
                            'TimeStamp': UNPREDICTABLE_ADYEN_DATA,
                        },
                    },
                    'PaymentTransaction': {
                        'AmountsReq': {
                            'Currency': UNPREDICTABLE_ADYEN_DATA,
                            'RequestedAmount': UNPREDICTABLE_ADYEN_DATA,
                        },
                    },
                },
            },
        }

        if with_acquirer_data:
            res['SaleToPOIRequest']['PaymentRequest']['SaleData']['SaleToAcquirerData'] = UNPREDICTABLE_ADYEN_DATA
        return res

    @api.model
    def _get_valid_acquirer_data(self):
        return {
            'tenderOption': 'AskGratuity',
            'authorisationType': 'PreAuth'
        }

    @api.model
    def _get_hmac(self, sale_id, service_id, poi_id, sale_transaction_id):
        return hmac(
            env=self.env(su=True),
            scope='pos_adyen_payment',
            message=(sale_id, service_id, poi_id, sale_transaction_id)
        )

    def _proxy_adyen_request_direct(self, data, operation):
        self.ensure_one()
        TIMEOUT = 10

        _logger.info('Request to Adyen by user #%d:\n%s', self.env.uid, pprint.pformat(data))

        environment = 'test' if self.sudo().adyen_test_mode else 'live'
        endpoint = self._get_adyen_endpoints()[operation] % environment
        headers = {
            'x-api-key': self.sudo().adyen_api_key,
        }
        req = requests.post(endpoint, json=data, headers=headers, timeout=TIMEOUT)

        # Authentication error doesn't return JSON
        if req.status_code == 401:
            return {
                'error': {
                    'status_code': req.status_code,
                    'message': req.text
                }
            }

        if req.text == 'ok':
            return True

        return req.json()

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    # pos.config fields
    pos_adyen_ask_customer_for_tip = fields.Boolean(compute='_compute_pos_adyen_ask_customer_for_tip', store=True, readonly=False)

    @api.depends('pos_iface_tipproduct', 'pos_config_id')
    def _compute_pos_adyen_ask_customer_for_tip(self):
        for res_config in self:
            if res_config.pos_iface_tipproduct:
                res_config.pos_adyen_ask_customer_for_tip = res_config.pos_config_id.adyen_ask_customer_for_tip
            else:
                res_config.pos_adyen_ask_customer_for_tip = False

```

## File: models\__init__.py

```python
# coding: utf-8
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import pos_config
from . import pos_payment_method
from . import res_config_settings

```

## File: static\src\app\payment_adyen.js

```javascript
import { _t } from "@web/core/l10n/translation";
import { PaymentInterface } from "@point_of_sale/app/payment/payment_interface";
import { AlertDialog } from "@web/core/confirmation_dialog/confirmation_dialog";
import { register_payment_method } from "@point_of_sale/app/store/pos_store";
import { sprintf } from "@web/core/utils/strings";
const { DateTime } = luxon;

export class PaymentAdyen extends PaymentInterface {
    setup() {
        super.setup(...arguments);
        this.paymentLineResolvers = {};
    }

    send_payment_request(uuid) {
        super.send_payment_request(uuid);
        return this._adyen_pay(uuid);
    }
    send_payment_cancel(order, uuid) {
        super.send_payment_cancel(order, uuid);
        return this._adyen_cancel();
    }

    set_most_recent_service_id(id) {
        this.most_recent_service_id = id;
    }

    pending_adyen_line() {
        return this.pos.getPendingPaymentLine("adyen");
    }

    _handle_odoo_connection_failure(data = {}) {
        // handle timeout
        var line = this.pending_adyen_line();
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

    _call_adyen(data, operation = false) {
        return this.pos.data
            .silentCall("pos.payment.method", "proxy_adyen_request", [
                [this.payment_method_id.id],
                data,
                operation,
            ])
            .catch(this._handle_odoo_connection_failure.bind(this));
    }

    _adyen_get_sale_id() {
        var config = this.pos.config;
        return sprintf("%s (ID: %s)", config.display_name, config.id);
    }

    _adyen_common_message_header() {
        var config = this.pos.config;
        this.most_recent_service_id = Math.floor(Math.random() * Math.pow(2, 64)).toString(); // random ID to identify request/response pairs
        this.most_recent_service_id = this.most_recent_service_id.substring(0, 10); // max length is 10

        return {
            ProtocolVersion: "3.0",
            MessageClass: "Service",
            MessageType: "Request",
            SaleID: this._adyen_get_sale_id(config),
            ServiceID: this.most_recent_service_id,
            POIID: this.payment_method_id.adyen_terminal_identifier,
        };
    }

    _adyen_pay_data() {
        var order = this.pos.get_order();
        var config = this.pos.config;
        var line = order.get_selected_paymentline();
        var data = {
            SaleToPOIRequest: {
                MessageHeader: Object.assign(this._adyen_common_message_header(), {
                    MessageCategory: "Payment",
                }),
                PaymentRequest: {
                    SaleData: {
                        SaleTransactionID: {
                            TransactionID: `${order.uuid}--${order.session_id.id}`,
                            TimeStamp: DateTime.now().toFormat("yyyy-MM-dd'T'HH:mm:ssZZ"), // iso format: '2018-01-10T11:30:15+00:00'
                        },
                    },
                    PaymentTransaction: {
                        AmountsReq: {
                            Currency: this.pos.currency.name,
                            RequestedAmount: line.amount,
                        },
                    },
                },
            },
        };

        if (config.adyen_ask_customer_for_tip) {
            data.SaleToPOIRequest.PaymentRequest.SaleData.SaleToAcquirerData =
                "tenderOption=AskGratuity";
        }

        return data;
    }

    _adyen_pay(uuid) {
        var order = this.pos.get_order();

        if (order.get_selected_paymentline().amount < 0) {
            this._show_error(_t("Cannot process transactions with negative amount."));
            return Promise.resolve();
        }

        var data = this._adyen_pay_data();
        var line = order.payment_ids.find((paymentLine) => paymentLine.uuid === uuid);
        line.setTerminalServiceId(this.most_recent_service_id);
        return this._call_adyen(data).then((data) => {
            return this._adyen_handle_response(data);
        });
    }

    _adyen_cancel(ignore_error) {
        var config = this.pos.config;
        var previous_service_id = this.most_recent_service_id;
        var header = Object.assign(this._adyen_common_message_header(), {
            MessageCategory: "Abort",
        });

        var data = {
            SaleToPOIRequest: {
                MessageHeader: header,
                AbortRequest: {
                    AbortReason: "MerchantAbort",
                    MessageReference: {
                        MessageCategory: "Payment",
                        SaleID: this._adyen_get_sale_id(config),
                        ServiceID: previous_service_id,
                    },
                },
            },
        };

        return this._call_adyen(data).then((data) => {
            // Only valid response is a 200 OK HTTP response which is
            // represented by true.
            if (!ignore_error && data !== true) {
                this._show_error(
                    _t(
                        "Cancelling the payment failed. Please cancel it manually on the payment terminal."
                    )
                );
            }
        });
    }

    _convert_receipt_info(output_text) {
        return output_text.reduce((acc, entry) => {
            var params = new URLSearchParams(entry.Text);
            if (params.get("name") && !params.get("value")) {
                return acc + sprintf("\n%s", params.get("name"));
            } else if (params.get("name") && params.get("value")) {
                return acc + sprintf("\n%s: %s", params.get("name"), params.get("value"));
            }

            return acc;
        }, "");
    }

    /**
     * This method handles the response that comes from Adyen
     * when we first make a request to pay.
     */
    _adyen_handle_response(response) {
        var line = this.pending_adyen_line();

        if (response.error && response.error.status_code == 401) {
            this._show_error(_t("Authentication failed. Please check your Adyen credentials."));
            line.set_payment_status("force_done");
            return false;
        }

        response = response.SaleToPOIRequest;
        if (response?.EventNotification?.EventToNotify === "Reject") {
            console.error("error from Adyen", response);

            var msg = "";
            if (response.EventNotification) {
                var params = new URLSearchParams(response.EventNotification.EventDetails);
                msg = params.get("message");
            }

            this._show_error(_t("An unexpected error occurred. Message from Adyen: %s", msg));
            if (line) {
                line.set_payment_status("force_done");
            }
            return false;
        } else {
            line.set_payment_status("waitingCard");
            return this.waitForPaymentConfirmation();
        }
    }

    waitForPaymentConfirmation() {
        return new Promise((resolve) => {
            this.paymentLineResolvers[this.pending_adyen_line().uuid] = resolve;
        });
    }

    /**
     * This method is called from pos_bus when the payment
     * confirmation from Adyen is received via the webhook.
     */
    async handleAdyenStatusResponse() {
        const notification = await this.pos.data.silentCall(
            "pos.payment.method",
            "get_latest_adyen_status",
            [[this.payment_method_id.id]]
        );

        if (!notification) {
            this._handle_odoo_connection_failure();
            return;
        }
        const line = this.pending_adyen_line();
        const response = notification.SaleToPOIResponse.PaymentResponse.Response;
        const additional_response = new URLSearchParams(response.AdditionalResponse);
        const isPaymentSuccessful = this.isPaymentSuccessful(notification, response);
        if (isPaymentSuccessful) {
            this.handleSuccessResponse(line, notification, additional_response);
        } else {
            this._show_error(
                sprintf(_t("Message from Adyen: %s"), additional_response.get("message"))
            );
        }
        // when starting to wait for the payment response we create a promise
        // that will be resolved when the payment response is received.
        // In case this resolver is lost ( for example on a refresh ) we
        // we use the handle_payment_response method on the payment line
        const resolver = this.paymentLineResolvers?.[line.uuid];
        if (resolver) {
            resolver(isPaymentSuccessful);
        } else {
            line.handle_payment_response(isPaymentSuccessful);
        }
    }
    isPaymentSuccessful(notification, response) {
        return (
            notification &&
            notification.SaleToPOIResponse.MessageHeader.ServiceID ==
                this.pending_adyen_line().terminalServiceId &&
            response.Result === "Success"
        );
    }
    handleSuccessResponse(line, notification, additional_response) {
        const config = this.pos.config;
        const payment_response = notification.SaleToPOIResponse.PaymentResponse;
        const payment_result = payment_response.PaymentResult;

        const cashier_receipt = payment_response.PaymentReceipt.find((receipt) => {
            return receipt.DocumentQualifier == "CashierReceipt";
        });

        if (cashier_receipt) {
            line.set_cashier_receipt(
                this._convert_receipt_info(cashier_receipt.OutputContent.OutputText)
            );
        }

        const customer_receipt = payment_response.PaymentReceipt.find((receipt) => {
            return receipt.DocumentQualifier == "CustomerReceipt";
        });

        if (customer_receipt) {
            line.set_receipt_info(
                this._convert_receipt_info(customer_receipt.OutputContent.OutputText)
            );
        }

        const tip_amount = payment_result.AmountsResp.TipAmount;
        if (config.adyen_ask_customer_for_tip && tip_amount > 0) {
            this.pos.set_tip(tip_amount);
            line.set_amount(payment_result.AmountsResp.AuthorizedAmount);
        }

        line.transaction_id = additional_response.get("pspReference");
        line.card_type = additional_response.get("cardType");
        line.cardholder_name = additional_response.get("cardHolderName") || "";
    }

    _show_error(msg, title) {
        if (!title) {
            title = _t("Adyen Error");
        }
        this.env.services.dialog.add(AlertDialog, {
            title: title,
            body: msg,
        });
    }
}

register_payment_method("adyen", PaymentAdyen);

```

## File: static\src\app\pos_store.js

```javascript
import { patch } from "@web/core/utils/patch";
import { PosStore } from "@point_of_sale/app/store/pos_store";

patch(PosStore.prototype, {
    async setup() {
        await super.setup(...arguments);
        this.data.connectWebSocket("ADYEN_LATEST_RESPONSE", () => {
            const pendingLine = this.getPendingPaymentLine("adyen");

            if (pendingLine) {
                pendingLine.payment_method_id.payment_terminal.handleAdyenStatusResponse();
            }
        });
    },
});

```

## File: static\src\overrides\components\payment_screen\payment_screen.js

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
                    paymentLine.payment_method_id.use_payment_terminal === "adyen" &&
                    !paymentLine.is_done() &&
                    paymentLine.get_payment_status() !== "pending"
            );
            if (!pendingPaymentLine) {
                return;
            }
            pendingPaymentLine.payment_method_id.payment_terminal.set_most_recent_service_id(
                pendingPaymentLine.terminalServiceId
            );
        });
    },
});

```

## File: static\src\overrides\models\pos_payment.js

```javascript
import { PosPayment } from "@point_of_sale/app/models/pos_payment";
import { patch } from "@web/core/utils/patch";

patch(PosPayment.prototype, {
    setTerminalServiceId(id) {
        this.terminalServiceId = id;
    },
});

```

## File: views\pos_payment_method_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="pos_payment_method_view_form_inherit_pos_adyen" model="ir.ui.view">
        <field name="name">pos.payment.method.form.inherit.adyen</field>
        <field name="model">pos.payment.method</field>
        <field name="inherit_id" ref="point_of_sale.pos_payment_method_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='use_payment_terminal']" position="after">
                <!-- Adyen -->
                <field name="adyen_api_key" invisible="use_payment_terminal != 'adyen'" required="use_payment_terminal == 'adyen'" password="True"/>
                <field name="adyen_terminal_identifier" invisible="use_payment_terminal != 'adyen'" required="use_payment_terminal == 'adyen'"/>
                <field name="adyen_event_url" invisible="use_payment_terminal != 'adyen'" widget="CopyClipboardChar"/>
                <field name="adyen_test_mode" invisible="use_payment_terminal != 'adyen'" required="use_payment_terminal == 'adyen'"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.pos_adyen</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="point_of_sale.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='tip_product']" position="after">
                <div invisible="not pos_iface_tipproduct">
                    <field name="pos_adyen_ask_customer_for_tip" class="oe_inline"/>
                    <label class="fw-normal" for="pos_adyen_ask_customer_for_tip" string="Add tip through payment terminal (Adyen)"/>
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

