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
        'point_of_sale.assets': [
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
from odoo import fields, http
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
        terminal_identifier = data['SaleToPOIResponse']['MessageHeader']['POIID']
        payment_method = request.env['pos.payment.method'].sudo().search([('adyen_terminal_identifier', '=', terminal_identifier)], limit=1)

        if payment_method:
            # These are only used to see if the terminal is reachable,
            # store the most recent ID we received.
            if data['SaleToPOIResponse'].get('DiagnosisResponse'):
                payment_method.adyen_latest_diagnosis = data['SaleToPOIResponse']['MessageHeader']['ServiceID']
            else:
                try:
                    adyen_additional_response = data['SaleToPOIResponse']['PaymentResponse']['Response']['AdditionalResponse']
                    parsed_adyen_additional_response = parse_qs(adyen_additional_response)
                    pos_hmac_metadata = parsed_adyen_additional_response.get('metadata.pos_hmac')
                    pos_hmac = pos_hmac_metadata[0] if pos_hmac_metadata and len(pos_hmac_metadata) == 1 else None

                    msg_header = data['SaleToPOIResponse']['MessageHeader']
                    if not pos_hmac or not consteq(pos_hmac, payment_method._get_hmac(msg_header['SaleID'], msg_header['ServiceID'], msg_header['POIID'], data['SaleToPOIResponse']['PaymentResponse']['SaleData']['SaleTransactionID']['TransactionID'])):
                        _logger.warning('Received an invalid Adyen event notification (invalid hmac): \n%s', pprint.pformat(data))
                        return

                    # The HMAC is removed to prevent anyone from using it in place of Adyen.
                    pos_hmac_metadata_raw = 'metadata.pos_hmac='+pos_hmac
                    safe_additional_response = adyen_additional_response.replace('&'+pos_hmac_metadata_raw, '').replace(pos_hmac_metadata_raw, '')
                    data['SaleToPOIResponse']['PaymentResponse']['Response']['AdditionalResponse'] = safe_additional_response
                except (KeyError, AttributeError):
                    _logger.warning('Received an invalid Adyen event notification: \n%s', pprint.pformat(data))
                    return
                payment_method.adyen_latest_response = json.dumps(data)
        else:
            _logger.error('received a message for a terminal not registered in Odoo: %s', terminal_identifier)

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
import random
import requests
import string
from urllib.parse import parse_qs
from werkzeug.exceptions import Forbidden

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
    adyen_latest_diagnosis = fields.Char(copy=False, groups='base.group_erp_manager') # used to determine if the terminal is still connected.

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
                    raise ValidationError(_('Terminal %s is already used on payment method %s.')
                                      % (payment_method.adyen_terminal_identifier, existing_payment_method.display_name))
                else:
                    raise ValidationError(_('Terminal %s is already used in company %s on payment method %s.')
                                          % (payment_method.adyen_terminal_identifier,
                                             existing_payment_method.company_id.name,
                                             existing_payment_method.display_name))

    def _get_adyen_endpoints(self):
        return {
            'terminal_request': 'https://terminal-api-%s.adyen.com/async',
        }

    def _is_write_forbidden(self, fields):
        whitelisted_fields = set(('adyen_latest_response', 'adyen_latest_diagnosis'))
        return super(PosPaymentMethod, self)._is_write_forbidden(fields - whitelisted_fields)

    def get_latest_adyen_status(self):
        self.ensure_one()
        if not self.env.su and not self.user_has_groups('point_of_sale.group_pos_user'):
            raise AccessDenied()

        latest_response = self.sudo().adyen_latest_response
        latest_response = json.loads(latest_response) if latest_response else False

        return {
            'latest_response': latest_response,
        }

    def proxy_adyen_request(self, data, operation=False):
        ''' Necessary because Adyen's endpoints don't have CORS enabled '''
        self.ensure_one()
        if not self.env.su and not self.user_has_groups('point_of_sale.group_pos_user'):
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
            is_payment_request_with_acquirer_data = len(parsed_sale_to_acquirer_data) <= 2
            if is_payment_request_with_acquirer_data:
                for key, values in parsed_sale_to_acquirer_data.items():
                    if len(values) != 1:
                        is_payment_request_with_acquirer_data = False
                        break
                    value = values[0]
                    if not ((key == 'tenderOption' and value == 'AskGratuity') or (key == 'authorisationType' and value == 'PreAuth')):
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
    def _get_hmac(self, sale_id, service_id, poiid, sale_transaction_id):
        return hmac(
            env=self.env(su=True),
            scope='pos_adyen_payment',
            message=(sale_id, service_id, poiid, sale_transaction_id),
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

## File: models\pos_session.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class PosSession(models.Model):
    _inherit = 'pos.session'

    def _loader_params_pos_payment_method(self):
        result = super()._loader_params_pos_payment_method()
        result['search_params']['fields'].append('adyen_terminal_identifier')
        return result

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
from . import pos_session
from . import res_config_settings

```

## File: static\src\js\models.js

```javascript
odoo.define('pos_adyen.models', function (require) {
const { register_payment_method, Payment } = require('point_of_sale.models');
const PaymentAdyen = require('pos_adyen.payment');
const Registries = require('point_of_sale.Registries');

register_payment_method('adyen', PaymentAdyen);

const PosAdyenPayment = (Payment) => class PosAdyenPayment extends Payment {
    constructor(obj, options) {
        super(...arguments);
        this.terminalServiceId = this.terminalServiceId || null;
    }
    //@override
    export_as_JSON() {
        const json = super.export_as_JSON(...arguments);
        json.terminal_service_id = this.terminalServiceId;
        return json;
    }
    //@override
    init_from_JSON(json) {
        super.init_from_JSON(...arguments);
        this.terminalServiceId = json.terminal_service_id;
    }
    setTerminalServiceId(id) {
        this.terminalServiceId = id;
    }
}
Registries.Model.extend(Payment, PosAdyenPayment);
});

```

## File: static\src\js\PaymentScreen.js

```javascript
odoo.define('pos_adyen.PaymentScreen', function(require) {
    "use strict";

    const PaymentScreen = require('point_of_sale.PaymentScreen');
    const Registries = require('point_of_sale.Registries');
    const { onMounted } = owl;

    const PosAdyenPaymentScreen = PaymentScreen => class extends PaymentScreen {
        setup() {
        super.setup();
            onMounted(() => {
                const pendingPaymentLine = this.currentOrder.paymentlines.find(
                    paymentLine => paymentLine.payment_method.use_payment_terminal === 'adyen' &&
                        (!paymentLine.is_done() && paymentLine.get_payment_status() !== 'pending')
                );
                if (pendingPaymentLine) {
                    const paymentTerminal = pendingPaymentLine.payment_method.payment_terminal;
                    paymentTerminal.set_most_recent_service_id(pendingPaymentLine.terminalServiceId);
                    pendingPaymentLine.set_payment_status('waiting');
                    paymentTerminal.start_get_status_polling().then(isPaymentSuccessful => {
                        if (isPaymentSuccessful) {
                            pendingPaymentLine.set_payment_status('done');
                            pendingPaymentLine.can_be_reversed = paymentTerminal.supports_reversals;
                        } else {
                            pendingPaymentLine.set_payment_status('retry');
                        }
                    });
                }
            });
        }
    };

    Registries.Component.extend(PaymentScreen, PosAdyenPaymentScreen);

    return PaymentScreen;
});

```

## File: static\src\js\payment_adyen.js

```javascript
odoo.define('pos_adyen.payment', function (require) {
"use strict";

var core = require('web.core');
var rpc = require('web.rpc');
var PaymentInterface = require('point_of_sale.PaymentInterface');
const { Gui } = require('point_of_sale.Gui');

var _t = core._t;

var PaymentAdyen = PaymentInterface.extend({
    send_payment_request: function (cid) {
        this._super.apply(this, arguments);
        this._reset_state();
        return this._adyen_pay(cid);
    },
    send_payment_cancel: function (order, cid) {
        this._super.apply(this, arguments);
        return this._adyen_cancel();
    },
    close: function () {
        this._super.apply(this, arguments);
    },

    set_most_recent_service_id(id) {
        this.most_recent_service_id = id;
    },

    pending_adyen_line() {
      return this.pos.get_order().paymentlines.find(
        paymentLine => paymentLine.payment_method.use_payment_terminal === 'adyen' && (!paymentLine.is_done()));
    },

    // private methods
    _reset_state: function () {
        this.was_cancelled = false;
        this.remaining_polls = 4;
        clearTimeout(this.polling);
    },

    _handle_odoo_connection_failure: function (data) {
        // handle timeout
        var line = this.pending_adyen_line();
        if (line) {
            line.set_payment_status('retry');
        }
        this._show_error(_t('Could not connect to the Odoo server, please check your internet connection and try again.'));

        return Promise.reject(data); // prevent subsequent onFullFilled's from being called
    },

    _call_adyen: function (data, operation) {
        return rpc.query({
            model: 'pos.payment.method',
            method: 'proxy_adyen_request',
            args: [[this.payment_method.id], data, operation],
        }, {
            // When a payment terminal is disconnected it takes Adyen
            // a while to return an error (~6s). So wait 10 seconds
            // before concluding Odoo is unreachable.
            timeout: 10000,
            shadow: true,
        }).catch(this._handle_odoo_connection_failure.bind(this));
    },

    _adyen_get_sale_id: function () {
        var config = this.pos.config;
        return _.str.sprintf('%s (ID: %s)', config.display_name, config.id);
    },

    _adyen_common_message_header: function () {
        var config = this.pos.config;
        this.most_recent_service_id = Math.floor(Math.random() * Math.pow(2, 64)).toString(); // random ID to identify request/response pairs
        this.most_recent_service_id = this.most_recent_service_id.substring(0, 10); // max length is 10

        return {
            'ProtocolVersion': '3.0',
            'MessageClass': 'Service',
            'MessageType': 'Request',
            'SaleID': this._adyen_get_sale_id(config),
            'ServiceID': this.most_recent_service_id,
            'POIID': this.payment_method.adyen_terminal_identifier
        };
    },

    _adyen_pay_data: function () {
        var order = this.pos.get_order();
        var config = this.pos.config;
        var line = order.selected_paymentline;
        var data = {
            'SaleToPOIRequest': {
                'MessageHeader': _.extend(this._adyen_common_message_header(), {
                    'MessageCategory': 'Payment',
                }),
                'PaymentRequest': {
                    'SaleData': {
                        'SaleTransactionID': {
                            'TransactionID': order.uid,
                            'TimeStamp': moment().format(), // iso format: '2018-01-10T11:30:15+00:00'
                        }
                    },
                    'PaymentTransaction': {
                        'AmountsReq': {
                            'Currency': this.pos.currency.name,
                            'RequestedAmount': line.amount,
                        }
                    }
                }
            }
        };

        if (config.adyen_ask_customer_for_tip) {
            data.SaleToPOIRequest.PaymentRequest.SaleData.SaleToAcquirerData = "tenderOption=AskGratuity";
        }

        return data;
    },

    _adyen_pay: function (cid) {
        var self = this;
        var order = this.pos.get_order();

        if (order.selected_paymentline.amount < 0) {
            this._show_error(_t('Cannot process transactions with negative amount.'));
            return Promise.resolve();
        }

        if (order === this.poll_error_order) {
            delete this.poll_error_order;
            return self._adyen_handle_response({});
        }

        var data = this._adyen_pay_data();
        var line = order.paymentlines.find(paymentLine => paymentLine.cid === cid);
        line.setTerminalServiceId(this.most_recent_service_id);
        return this._call_adyen(data).then(function (data) {
            return self._adyen_handle_response(data);
        });
    },

    _adyen_cancel: function (ignore_error) {
        var self = this;
        var config = this.pos.config;
        var previous_service_id = this.most_recent_service_id;
        var header = _.extend(this._adyen_common_message_header(), {
            'MessageCategory': 'Abort',
        });

        var data = {
            'SaleToPOIRequest': {
                'MessageHeader': header,
                'AbortRequest': {
                    'AbortReason': 'MerchantAbort',
                    'MessageReference': {
                        'MessageCategory': 'Payment',
                        'SaleID': this._adyen_get_sale_id(config),
                        'ServiceID': previous_service_id,
                    }
                },
            }
        };

        return this._call_adyen(data).then(function (data) {
            // Only valid response is a 200 OK HTTP response which is
            // represented by true.
            if (! ignore_error && data !== true) {
                self._show_error(_t('Cancelling the payment failed. Please cancel it manually on the payment terminal.'));
                self.was_cancelled = !!self.polling;
            }
        });
    },

    _convert_receipt_info: function (output_text) {
        return output_text.reduce(function (acc, entry) {
            var params = new URLSearchParams(entry.Text);

            if (params.get('name') && !params.get('value')) {
                return acc + _.str.sprintf('\n%s', params.get('name'));
            } else if (params.get('name') && params.get('value')) {
                return acc + _.str.sprintf('\n%s: %s', params.get('name'), params.get('value'));
            }

            return acc;
        }, '');
    },

    _poll_for_response: function (resolve, reject) {
        var self = this;
        if (this.was_cancelled) {
            resolve(false);
            return Promise.resolve();
        }

        return rpc.query({
            model: 'pos.payment.method',
            method: 'get_latest_adyen_status',
            args: [[this.payment_method.id]],
        }, {
            timeout: 5000,
            shadow: true,
        }).catch(function (data) {
            if (self.remaining_polls != 0) {
                self.remaining_polls--;
            } else {
                reject();
                self.poll_error_order = self.pos.get_order();
                return self._handle_odoo_connection_failure(data);
            }
            // This is to make sure that if 'data' is not an instance of Error (i.e. timeout error),
            // this promise don't resolve -- that is, it doesn't go to the 'then' clause.
            return Promise.reject(data);
        }).then(function (status) {
            var notification = status.latest_response;
            var order = self.pos.get_order();
            var line = self.pending_adyen_line() || resolve(false);

            if (notification && notification.SaleToPOIResponse.MessageHeader.ServiceID == line.terminalServiceId) {
                var response = notification.SaleToPOIResponse.PaymentResponse.Response;
                var additional_response = new URLSearchParams(response.AdditionalResponse);

                if (response.Result == 'Success') {
                    var config = self.pos.config;
                    var payment_response = notification.SaleToPOIResponse.PaymentResponse;
                    var payment_result = payment_response.PaymentResult;

                    var cashier_receipt = payment_response.PaymentReceipt.find(function (receipt) {
                        return receipt.DocumentQualifier == 'CashierReceipt';
                    });

                    if (cashier_receipt) {
                        line.set_cashier_receipt(self._convert_receipt_info(cashier_receipt.OutputContent.OutputText));
                    }

                    var customer_receipt = payment_response.PaymentReceipt.find(function (receipt) {
                        return receipt.DocumentQualifier == 'CustomerReceipt';
                    });

                    if (customer_receipt) {
                        line.set_receipt_info(self._convert_receipt_info(customer_receipt.OutputContent.OutputText));
                    }

                    var tip_amount = payment_result.AmountsResp.TipAmount;
                    if (config.adyen_ask_customer_for_tip && tip_amount > 0) {
                        order.set_tip(tip_amount);
                        line.set_amount(payment_result.AmountsResp.AuthorizedAmount);
                    }

                    line.transaction_id = additional_response.get('pspReference');
                    line.card_type = additional_response.get('cardType');
                    line.cardholder_name = additional_response.get('cardHolderName') || '';
                    resolve(true);
                } else {
                    var message = additional_response.get('message');
                    self._show_error(_.str.sprintf(_t('Message from Adyen: %s'), message));

                    // this means the transaction was cancelled by pressing the cancel button on the device
                    if (message.startsWith('108 ')) {
                        resolve(false);
                    } else {
                        line.set_payment_status('retry');
                        reject();
                    }
                }
            } else {
                line.set_payment_status('waitingCard')
            }
        });
    },

    _adyen_handle_response: function (response) {
        var line = this.pending_adyen_line();

        if (response.error && response.error.status_code == 401) {
            this._show_error(_t('Authentication failed. Please check your Adyen credentials.'));
            line.set_payment_status('force_done');
            return Promise.resolve();
        }

        response = response.SaleToPOIRequest;
        if (response && response.EventNotification && response.EventNotification.EventToNotify == 'Reject') {
            console.error('error from Adyen', response);

            var msg = '';
            if (response.EventNotification) {
                var params = new URLSearchParams(response.EventNotification.EventDetails);
                msg = params.get('message');
            }

            this._show_error(_.str.sprintf(_t('An unexpected error occurred. Message from Adyen: %s'), msg));
            if (line) {
                line.set_payment_status('force_done');
            }

            return Promise.resolve();
        } else {
            line.set_payment_status('waitingCard');
            return this.start_get_status_polling()
        }
    },

    start_get_status_polling() {
        var self = this;
        var res = new Promise(function (resolve, reject) {
            // clear previous intervals just in case, otherwise
            // it'll run forever
            clearTimeout(self.polling);
            self._poll_for_response(resolve, reject);
            self.polling = setInterval(function () {
                self._poll_for_response(resolve, reject);
            }, 5500);
        });

        // make sure to stop polling when we're done
        res.finally(function () {
            self._reset_state();
        });

        return res;
    },

    _show_error: function (msg, title) {
        if (!title) {
            title =  _t('Adyen Error');
        }
        Gui.showPopup('ErrorPopup',{
            'title': title,
            'body': msg,
        });
    },
});

return PaymentAdyen;
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
                <field name="adyen_api_key" attrs="{'invisible': [('use_payment_terminal', '!=', 'adyen')], 'required': [('use_payment_terminal', '=', 'adyen')]}" password="True"/>
                <field name="adyen_terminal_identifier" attrs="{'invisible': [('use_payment_terminal', '!=', 'adyen')], 'required': [('use_payment_terminal', '=', 'adyen')]}"/>
                <field name="adyen_test_mode" attrs="{'invisible': [('use_payment_terminal', '!=', 'adyen')], 'required': [('use_payment_terminal', '=', 'adyen')]}"/>
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
                <div attrs="{'invisible': [('pos_iface_tipproduct', '=', False)]}">
                    <field name="pos_adyen_ask_customer_for_tip" class="oe_inline"/>
                    <label class="fw-normal" for="pos_adyen_ask_customer_for_tip" string="Add tip through payment terminal (Adyen)"/>
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

