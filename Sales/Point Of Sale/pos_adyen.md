# Odoo Module: pos_adyen

Category: Sales/Point Of Sale

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
    'category': 'Sales/Point Of Sale',
    'sequence': 6,
    'summary': 'Integrate your POS with an Adyen payment terminal',
    'description': '',
    'data': [
        'views/pos_config_views.xml',
        'views/pos_payment_method_views.xml',
        'views/point_of_sale_assets.xml',
    ],
    'depends': ['point_of_sale'],
    'qweb': ['static/src/xml/pos.xml'],
    'installable': True,
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# coding: utf-8
import logging
import pprint
import json
from odoo import fields, http
from odoo.http import request

_logger = logging.getLogger(__name__)


class PosAdyenController(http.Controller):
    @http.route('/pos_adyen/notification', type='json', methods=['POST'], auth='none', csrf=False)
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
                payment_method.adyen_latest_response = json.dumps(data)
        else:
            _logger.error('received a message for a terminal not registered in Odoo: %s', terminal_identifier)

```

## File: controllers\__init__.py

```python
# coding: utf-8
from . import main

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

    adyen_ask_customer_for_tip = fields.Boolean('Ask Customers For Tip', help='Prompt the customer to tip.')

    @api.constrains('adyen_ask_customer_for_tip', 'iface_tipproduct', 'tip_product_id')
    def _check_adyen_ask_customer_for_tip(self):
        for config in self:
            if config.adyen_ask_customer_for_tip and (not config.tip_product_id or not config.iface_tipproduct):
                raise ValidationError(_("Please configure a tip product for POS %s to support tipping with Adyen.") % config.name)

    @api.onchange('adyen_ask_customer_for_tip')
    def _onchange_adyen_ask_customer_for_tip(self):
        for config in self:
            if config.adyen_ask_customer_for_tip:
                config.iface_tipproduct = True

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

from odoo import fields, models, api, _
from odoo.exceptions import ValidationError

_logger = logging.getLogger(__name__)

class PosPaymentMethod(models.Model):
    _inherit = 'pos.payment.method'

    def _get_payment_terminal_selection(self):
        return super(PosPaymentMethod, self)._get_payment_terminal_selection() + [('adyen', 'Adyen')]

    adyen_api_key = fields.Char(string="Adyen API key", help='Used when connecting to Adyen: https://docs.adyen.com/user-management/how-to-get-the-api-key/#description', copy=False)
    adyen_terminal_identifier = fields.Char(help='[Terminal model]-[Serial number], for example: P400Plus-123456789', copy=False)
    adyen_test_mode = fields.Boolean(help='Run transactions in the test environment.')
    adyen_latest_response = fields.Char(help='Technical field used to buffer the latest asynchronous notification from Adyen.', copy=False, groups='base.group_erp_manager')
    adyen_latest_diagnosis = fields.Char(help='Technical field used to determine if the terminal is still connected.', copy=False, groups='base.group_erp_manager')

    @api.constrains('adyen_terminal_identifier')
    def _check_adyen_terminal_identifier(self):
        for payment_method in self:
            if not payment_method.adyen_terminal_identifier:
                continue
            existing_payment_method = self.search([('id', '!=', payment_method.id),
                                                   ('adyen_terminal_identifier', '=', payment_method.adyen_terminal_identifier)],
                                                  limit=1)
            if existing_payment_method:
                raise ValidationError(_('Terminal %s is already used on payment method %s.')
                                      % (payment_method.adyen_terminal_identifier, existing_payment_method.display_name))

    def _is_write_forbidden(self, fields):
        whitelisted_fields = set(('adyen_latest_response', 'adyen_latest_diagnosis'))
        return super(PosPaymentMethod, self)._is_write_forbidden(fields - whitelisted_fields)

    def _adyen_diagnosis_request_data(self, pos_config_name, terminal_identifier):
        service_id = ''.join(random.choices(string.ascii_letters + string.digits, k=10))
        return {
            "SaleToPOIRequest": {
                "MessageHeader": {
                    "ProtocolVersion": "3.0",
                    "MessageClass": "Service",
                    "MessageCategory": "Diagnosis",
                    "MessageType": "Request",
                    "ServiceID": service_id,
                    "SaleID": pos_config_name,
                    "POIID": terminal_identifier,
                },
                "DiagnosisRequest": {
                    "HostDiagnosisFlag": False
                }
            }
        }

    @api.model
    def get_latest_adyen_status(self, payment_method_id, pos_config_name, terminal_identifier, test_mode, api_key):
        '''See the description of proxy_adyen_request as to why this is an
        @api.model function.
        '''

        # Poll the status of the terminal if there's no new
        # notification we received. This is done so we can quickly
        # notify the user if the terminal is no longer reachable due
        # to connectivity issues.
        self.proxy_adyen_request(self._adyen_diagnosis_request_data(pos_config_name, terminal_identifier),
                                 test_mode,
                                 api_key)

        payment_method = self.sudo().browse(payment_method_id)
        latest_response = payment_method.adyen_latest_response
        latest_response = json.loads(latest_response) if latest_response else False
        payment_method.adyen_latest_response = ''  # avoid handling old responses multiple times

        return {
            'latest_response': latest_response,
            'last_received_diagnosis_id': payment_method.adyen_latest_diagnosis,
        }

    @api.model
    def proxy_adyen_request(self, data, test_mode, api_key):
        '''Necessary because Adyen's endpoints don't have CORS enabled. This is an
        @api.model function to avoid concurrent update errors. Adyen's
        async endpoint can still take well over a second to complete a
        request. By using @api.model and passing in all data we need from
        the POS we avoid locking the pos_payment_method table. This way we
        avoid concurrent update errors when Adyen calls us back on
        /pos_adyen/notification which will need to write on
        pos.payment.method.
        '''
        TIMEOUT = 10
        endpoint = 'https://terminal-api-live.adyen.com/async'
        if test_mode:
            endpoint = 'https://terminal-api-test.adyen.com/async'

        _logger.info('request to adyen\n%s', pprint.pformat(data))
        headers = {
            'x-api-key': api_key,
            'Content-Type': 'application/json'
        }
        req = requests.post(endpoint, data=json.dumps(data), headers=headers, timeout=TIMEOUT)
        _logger.info('response from adyen (HTTP status %s):\n%s', req.status_code, req.text)

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

    @api.onchange('use_payment_terminal')
    def _onchange_use_payment_terminal(self):
        super(PosPaymentMethod, self)._onchange_use_payment_terminal()
        if self.use_payment_terminal != 'adyen':
            self.adyen_api_key = False
            self.adyen_terminal_identifier = False

```

## File: models\__init__.py

```python
# coding: utf-8
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import pos_config
from . import pos_payment_method

```

## File: static\src\js\models.js

```javascript
odoo.define('pos_adyen.models', function (require) {
var models = require('point_of_sale.models');
var PaymentAdyen = require('pos_adyen.payment');

models.register_payment_method('adyen', PaymentAdyen);
models.load_fields('pos.payment.method', ['adyen_terminal_identifier', 'adyen_test_mode', 'adyen_api_key']);
});

```

## File: static\src\js\payment_adyen.js

```javascript
odoo.define('pos_adyen.payment', function (require) {
"use strict";

var core = require('web.core');
var rpc = require('web.rpc');
var PaymentInterface = require('point_of_sale.PaymentInterface');

var _t = core._t;

var PaymentAdyen = PaymentInterface.extend({
    send_payment_request: function (cid) {
        this._super.apply(this, arguments);
        this._reset_state();
        return this._adyen_pay();
    },
    send_payment_cancel: function (order, cid) {
        this._super.apply(this, arguments);
        // set only if we are polling
        this.was_cancelled = !!this.polling;
        return this._adyen_cancel();
    },
    close: function () {
        this._super.apply(this, arguments);
    },

    // private methods
    _reset_state: function () {
        this.was_cancelled = false;
        this.last_diagnosis_service_id = false;
        this.remaining_polls = 2;
        clearTimeout(this.polling);
    },

    _handle_odoo_connection_failure: function (data) {
        // handle timeout
        var line = this.pos.get_order().selected_paymentline;
        if (line) {
            line.set_payment_status('retry');
        }
        this._show_error(_('Could not connect to the Odoo server, please check your internet connection and try again.'));

        return Promise.reject(data); // prevent subsequent onFullFilled's from being called
    },

    _call_adyen: function (data) {
        var self = this;
        return rpc.query({
            model: 'pos.payment.method',
            method: 'proxy_adyen_request',
            args: [data, this.payment_method.adyen_test_mode, this.payment_method.adyen_api_key],
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

    _adyen_pay: function () {
        var self = this;

        if (this.pos.get_order().selected_paymentline.amount < 0) {
            this._show_error(_('Cannot process transactions with negative amount.'));
            return Promise.resolve();
        }

        var data = this._adyen_pay_data();

        return this._call_adyen(data).then(function (data) {
            return self._adyen_handle_response(data);
        });
    },

    _adyen_cancel: function (ignore_error) {
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
                        'SaleID': header.SaleID,
                        'ServiceID': previous_service_id,
                    }
                },
            }
        };

        return this._call_adyen(data).then(function (data) {

            // Only valid response is a 200 OK HTTP response which is
            // represented by true.
            if (! ignore_error && data !== true) {
                self._show_error(_('Cancelling the payment failed. Please cancel it manually on the payment terminal.'));
            }
        });
    },

    _convert_receipt_info: function (output_text) {
        return output_text.reduce(function (acc, entry) {
            var params = new URLSearchParams(entry.Text);

            if (params.get('name') && !params.get('value')) {
                return acc + _.str.sprintf('<br/>%s', params.get('name'));
            } else if (params.get('name') && params.get('value')) {
                return acc + _.str.sprintf('<br/>%s: %s', params.get('name'), params.get('value'));
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
            args: [this.payment_method.id,
                   this._adyen_get_sale_id(),
                   this.payment_method.adyen_terminal_identifier,
                   this.payment_method.adyen_test_mode,
                   this.payment_method.adyen_api_key],
        }, {
            timeout: 5000,
            shadow: true,
        }).catch(function (data) {
            reject();
            return self._handle_odoo_connection_failure(data);
        }).then(function (status) {
            var notification = status.latest_response;
            var last_diagnosis_service_id = status.last_received_diagnosis_id;
            var order = self.pos.get_order();
            var line = order.selected_paymentline;


            if (self.last_diagnosis_service_id != last_diagnosis_service_id) {
                self.last_diagnosis_service_id = last_diagnosis_service_id;
                self.remaining_polls = 2;
            } else {
                self.remaining_polls--;
            }

            if (notification && notification.SaleToPOIResponse.MessageHeader.ServiceID == self.most_recent_service_id) {
                var response = notification.SaleToPOIResponse.PaymentResponse.Response;
                var additional_response = new URLSearchParams(response.AdditionalResponse);

                if (response.Result == 'Success') {
                    var config = self.pos.config;
                    var payment_response = notification.SaleToPOIResponse.PaymentResponse;
                    var payment_result = payment_response.PaymentResult;
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
                    resolve(true);
                } else {
                    var message = additional_response.get('message');
                    self._show_error(_.str.sprintf(_t('Message from Adyen: %s'), message));

                    // this means the transaction was cancelled by pressing the cancel button on the device
                    if (message.startsWith('108 ')) {
                        resolve(false);
                    } else {
                        line.set_payment_status('force_done');
                        reject();
                    }
                }
            } else if (self.remaining_polls <= 0) {
                self._show_error(_t('The connection to your payment terminal failed. Please check if it is still connected to the internet.'));
                self._adyen_cancel();
                resolve(false);
            }
        });
    },

    _adyen_handle_response: function (response) {
        var line = this.pos.get_order().selected_paymentline;

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

            this._show_error(_.str.sprintf(_t('An unexpected error occured. Message from Adyen: %s'), msg));
            if (line) {
                line.set_payment_status('force_done');
            }

            return Promise.resolve();
        } else {
            line.set_payment_status('waitingCard');

            // This is not great, the payment screen should be
            // refactored so it calls render_paymentlines whenever a
            // paymentline changes. This way the call to
            // set_payment_status would re-render it automatically.
            this.pos.chrome.gui.current_screen.render_paymentlines();

            var self = this;
            var res = new Promise(function (resolve, reject) {
                // clear previous intervals just in case, otherwise
                // it'll run forever
                clearTimeout(self.polling);

                self.polling = setInterval(function () {
                    self._poll_for_response(resolve, reject);
                }, 3000);
            });

            // make sure to stop polling when we're done
            res.finally(function () {
                self._reset_state();
            });

            return res;
        }
    },

    _show_error: function (msg, title) {
        if (!title) {
            title =  _t('Adyen Error');
        }
        this.pos.gui.show_popup('error',{
            'title': title,
            'body': msg,
        });
    },
});

return PaymentAdyen;
});

```

## File: views\point_of_sale_assets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="assets" inherit_id="point_of_sale.assets">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/pos_adyen/static/src/js/payment_adyen.js"></script>
            <script type="text/javascript" src="/pos_adyen/static/src/js/models.js"></script>
        </xpath>
    </template>
</odoo>

```

## File: views\pos_config_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="pos_config_view_form" model="ir.ui.view">
        <field name="name">pos.config.form.view.inherit.pos_adyen</field>
        <field name="model">pos.config</field>
        <field name="inherit_id" ref="point_of_sale.pos_config_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='payment_methods_new']/.." position="after">
                <h2>Adyen</h2>
                <div class="row mt16 o_settings_container" id="adyen">
                    <div class="col-12 col-lg-6 o_setting_box">
                        <div class="o_setting_left_pane">
                          <field name="adyen_ask_customer_for_tip"/>
                        </div>
                        <div class="o_setting_right_pane">
                            <label for="adyen_ask_customer_for_tip"/>
                            <div class="text-muted">
                                Ask customers to tip before paying.
                            </div>
                        </div>
                    </div>
                </div>
            </xpath>
        </field>
    </record>
</odoo>

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
                <field name="adyen_api_key"
                        attrs="{'invisible': [('use_payment_terminal', '!=', 'adyen')], 'required': [('use_payment_terminal', '=', 'adyen')]}"/>
                <field name="adyen_terminal_identifier" attrs="{'invisible': [('use_payment_terminal', '!=', 'adyen')], 'required': [('use_payment_terminal', '=', 'adyen')]}"/>
                <field name="adyen_test_mode" attrs="{'invisible': [('use_payment_terminal', '!=', 'adyen')]}"/>
          </xpath>
      </field>
    </record>
</odoo>

```

