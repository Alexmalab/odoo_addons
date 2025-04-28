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
    'description': '',
    'data': [
        'security/ir.model.access.csv',
        'views/adyen_account_views.xml',
        'views/pos_config_views.xml',
        'views/pos_payment_method_views.xml',
        'views/point_of_sale_assets.xml',
        'views/res_config_settings_views.xml',
    ],
    'depends': ['adyen_platforms', 'point_of_sale'],
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
            _logger.info('notification writed from adyen\n%s', data)
        else:
            _logger.error('received a message for a terminal not registered in Odoo: %s', terminal_identifier)

```

## File: controllers\__init__.py

```python
# coding: utf-8
from . import main

```

## File: models\adyen_account.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import uuid
from werkzeug.urls import url_join

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


class AdyenAccount(models.Model):
    _inherit = 'adyen.account'

    store_ids = fields.One2many('adyen.store', 'adyen_account_id')
    terminal_ids = fields.One2many('adyen.terminal', 'adyen_account_id')

    @api.model
    def _sync_adyen_cron(self):
        self.env['adyen.terminal']._sync_adyen_terminals()
        super(AdyenAccount, self)._sync_adyen_cron()

    def action_order_terminal(self):
        if not self.store_ids:
            raise ValidationError(_('Please create a store first.'))

        store_uuids = ','.join(self.store_ids.mapped('store_uuid'))
        onboarding_url = self.env['ir.config_parameter'].sudo().get_param('adyen_platforms.onboarding_url')
        return {
            'type': 'ir.actions.act_url',
            'target': 'new',
            'url': url_join(onboarding_url, 'order_terminals?store_uuids=%s' % store_uuids),
        }


class AdyenStore(models.Model):
    _name = 'adyen.store'
    _inherit = ['adyen.address.mixin']
    _description = 'Adyen for Platforms Store'

    adyen_account_id = fields.Many2one('adyen.account', ondelete='cascade')
    store_reference = fields.Char('Reference', default=lambda self: uuid.uuid4().hex)
    store_uuid = fields.Char('UUID', readonly=True) # Given by Adyen
    name = fields.Char('Name', required=True)
    phone_number = fields.Char('Phone Number', required=True)
    terminal_ids = fields.One2many('adyen.terminal', 'store_id', string='Payment Terminals', readonly=True)

    @api.model
    def create(self, values):
        adyen_store_id = super(AdyenStore, self).create(values)
        response = adyen_store_id.adyen_account_id._adyen_rpc('create_store', adyen_store_id._format_data())
        stores = response['accountHolderDetails']['storeDetails']
        created_store = next(store for store in stores if store['storeReference'] == adyen_store_id.store_reference)
        adyen_store_id.with_context(update_from_adyen=True).sudo().write({
            'store_uuid': created_store['store'],
        })
        return adyen_store_id

    def unlink(self):
        for store_id in self:
            store_id.adyen_account_id._adyen_rpc('close_stores', {
                'accountHolderCode': store_id.adyen_account_id.account_holder_code,
                'stores': [store_id.store_uuid],
            })
        return super(AdyenStore, self).unlink()

    def _format_data(self):
        return {
            'accountHolderCode': self.adyen_account_id.account_holder_code,
            'accountHolderDetails': {
                'storeDetails': [{
                    'storeReference': self.store_reference,
                    'storeName': self.name,
                    'merchantCategoryCode': '7999',
                    'address': {
                        'city': self.city,
                        'country': self.country_id.code,
                        'houseNumberOrName': self.house_number_or_name,
                        'postalCode': self.zip,
                        'stateOrProvince': self.state_id.code or None,
                        'street': self.street,
                    },
                    'fullPhoneNumber': self.phone_number,
                }],
            }
        }


class AdyenTerminal(models.Model):
    _name = 'adyen.terminal'
    _description = 'Adyen for Platforms Terminal'
    _rec_name = 'terminal_uuid'

    adyen_account_id = fields.Many2one('adyen.account', ondelete='cascade')
    store_id = fields.Many2one('adyen.store')
    terminal_uuid = fields.Char('Terminal ID')

    @api.model
    def _sync_adyen_terminals(self):
        for adyen_store_id in self.env['adyen.store'].search([]):
            response = adyen_store_id.adyen_account_id._adyen_rpc('connected_terminals', {
                'store': adyen_store_id.store_uuid,
            })
            terminals_in_db = set(self.search([('store_id', '=', adyen_store_id.id)]).mapped('terminal_uuid'))

            # Added terminals
            for terminal in set(response.get('uniqueTerminalIds')) - terminals_in_db:
                self.sudo().create({
                    'adyen_account_id': adyen_store_id.adyen_account_id.id,
                    'store_id': adyen_store_id.id,
                    'terminal_uuid': terminal,
                })

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

    @api.onchange('iface_tipproduct')
    def _onchange_iface_tipproduct_adyen(self):
        if not self.iface_tipproduct:
            self.adyen_ask_customer_for_tip = False

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
from werkzeug.exceptions import Forbidden

from odoo import fields, models, api, _
from odoo.exceptions import ValidationError

_logger = logging.getLogger(__name__)

class PosPaymentMethod(models.Model):
    _inherit = 'pos.payment.method'

    def _get_payment_terminal_selection(self):
        return super(PosPaymentMethod, self)._get_payment_terminal_selection() + [('odoo_adyen', 'Odoo Payments by Adyen'), ('adyen', 'Adyen')]

    # Adyen
    adyen_api_key = fields.Char(string="Adyen API key", help='Used when connecting to Adyen: https://docs.adyen.com/user-management/how-to-get-the-api-key/#description', copy=False)
    adyen_terminal_identifier = fields.Char(help='[Terminal model]-[Serial number], for example: P400Plus-123456789', copy=False)
    adyen_test_mode = fields.Boolean(help='Run transactions in the test environment.')

    # Odoo Payments by Adyen
    adyen_account_id = fields.Many2one('adyen.account', related='company_id.adyen_account_id')
    adyen_payout_id = fields.Many2one('adyen.payout', string='Adyen Payout', domain="[('adyen_account_id', '=', adyen_account_id)]")
    adyen_terminal_id = fields.Many2one('adyen.terminal', string='Adyen Terminal', domain="[('adyen_account_id', '=', adyen_account_id)]")

    adyen_latest_response = fields.Char(help='Technical field used to buffer the latest asynchronous notification from Adyen.', copy=False, groups='base.group_erp_manager')
    adyen_latest_diagnosis = fields.Char(help='Technical field used to determine if the terminal is still connected.', copy=False, groups='base.group_erp_manager')

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

    @api.onchange('adyen_terminal_id')
    def onchange_use_payment_terminal(self):
        for payment_method in self:
            if payment_method.use_payment_terminal == 'odoo_adyen' and payment_method.adyen_terminal_id:
                payment_method.adyen_terminal_identifier = payment_method.adyen_terminal_id.terminal_uuid

    def _is_write_forbidden(self, fields):
        whitelisted_fields = set(('adyen_latest_response', 'adyen_latest_diagnosis'))
        return super(PosPaymentMethod, self)._is_write_forbidden(fields - whitelisted_fields)

    def _adyen_diagnosis_request_data(self, pos_config_name):
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
                    "POIID": self.adyen_terminal_identifier,
                },
                "DiagnosisRequest": {
                    "HostDiagnosisFlag": False
                }
            }
        }

    def get_latest_adyen_status(self, pos_config_name):
        _logger.info('get_latest_adyen_status\n%s', pos_config_name)
        self.ensure_one()

        latest_response = self.sudo().adyen_latest_response
        latest_response = json.loads(latest_response) if latest_response else False

        return {
            'latest_response': latest_response,
        }

    def proxy_adyen_request(self, data, operation=False):
        ''' Necessary because Adyen's endpoints don't have CORS enabled '''
        if data['SaleToPOIRequest']['MessageHeader']['MessageCategory'] == 'Payment': # Clear only if it is a payment request
            self.sudo().adyen_latest_response = ''  # avoid handling old responses multiple times

        if not operation:
            operation = 'terminal_request'

        if self.use_payment_terminal == 'odoo_adyen':
            return self._proxy_adyen_request_odoo_proxy(data, operation)
        else:
            return self._proxy_adyen_request_direct(data, operation)

    def _proxy_adyen_request_direct(self, data, operation):
        self.ensure_one()
        TIMEOUT = 10

        _logger.info('request to adyen\n%s', pprint.pformat(data))

        environment = 'test' if self.adyen_test_mode else 'live'
        endpoint = self._get_adyen_endpoints()[operation] % environment
        headers = {
            'x-api-key': self.adyen_api_key,
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

    def _proxy_adyen_request_odoo_proxy(self, data, operation):
        try:
            return self.env.company.sudo().adyen_account_id._adyen_rpc(operation, {
                'request_data': data,
                'account_code': self.sudo().adyen_payout_id.code,
                'notification_url': self.env['ir.config_parameter'].sudo().get_param('web.base.url'),
            })
        except Forbidden:
            return {
                'error': {
                    'status_code': 401,
                }
            }

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    adyen_account_id = fields.Many2one(string='Adyen Account', related='company_id.adyen_account_id')

    def create_adyen_account(self):
        return self.env['adyen.account'].action_create_redirect()

```

## File: models\__init__.py

```python
# coding: utf-8
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import adyen_account
from . import pos_config
from . import pos_payment_method
from . import res_config_settings

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_adyen_store_group_erp_manager,adyen.store,model_adyen_store,base.group_erp_manager,1,0,1,1
access_adyen_payout_group_pos_manager,adyen.payout,adyen_platforms.model_adyen_payout,point_of_sale.group_pos_manager,1,0,0,0
access_adyen_terminal_group_pos_manager,adyen.terminal,model_adyen_terminal,point_of_sale.group_pos_manager,1,0,0,0

```

## File: static\src\js\models.js

```javascript
odoo.define('pos_adyen.models', function (require) {
var models = require('point_of_sale.models');
var PaymentAdyen = require('pos_adyen.payment');

models.register_payment_method('adyen', PaymentAdyen);
models.register_payment_method('odoo_adyen', PaymentAdyen);
models.load_fields('pos.payment.method', 'adyen_terminal_identifier');

const superPaymentline = models.Paymentline.prototype;
models.Paymentline = models.Paymentline.extend({
    initialize: function(attr, options) {
        superPaymentline.initialize.call(this,attr,options);
        this.terminalServiceId = this.terminalServiceId  || null;
    },
    export_as_JSON: function(){
        const json = superPaymentline.export_as_JSON.call(this);
        json.terminal_service_id = this.terminalServiceId;
        return json;
    },
    init_from_JSON: function(json){
        superPaymentline.init_from_JSON.apply(this,arguments);
        this.terminalServiceId = json.terminal_service_id;
    },
    setTerminalServiceId: function(id) {
        this.terminalServiceId = id;
    }
});

});

```

## File: static\src\js\PaymentScreen.js

```javascript
odoo.define('pos_adyen.PaymentScreen', function(require) {
    "use strict";

    const PaymentScreen = require('point_of_sale.PaymentScreen');
    const Registries = require('point_of_sale.Registries');
    const { onMounted } = owl.hooks;

    const PosAdyenPaymentScreen = PaymentScreen => class extends PaymentScreen {
        constructor() {
            super(...arguments);
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
            args: [[this.payment_method.id], this._adyen_get_sale_id()],
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

            this._show_error(_.str.sprintf(_t('An unexpected error occured. Message from Adyen: %s'), msg));
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

## File: views\adyen_account_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="adyen_account_view_form" model="ir.ui.view">
        <field name="name">adyen.account.view.form.inherit.pos.adyen</field>
        <field name="model">adyen.account</field>
        <field name="inherit_id" ref="adyen_platforms.adyen_account_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//header" position="inside">
                <button name="action_order_terminal" string="Order a Terminal" class="oe_highlight" type="object" attrs="{'invisible': [('adyen_uuid', '=', False)]}"/>
            </xpath>
            <xpath expr="//notebook" position="inside">
                <page string="Stores">
                    <field name="store_ids">
                        <tree>
                            <field name="name"/>
                            <field name="store_uuid"/>
                        </tree>
                    </field>
                </page>
            </xpath>
        </field>
    </record>


    <record id="adyen_store_view_form" model="ir.ui.view">
        <field name="name">adyen.store.view.form</field>
        <field name="model">adyen.store</field>
        <field name="arch" type="xml">
            <form string="Adyen Store">
                <sheet>
                    <group>
                        <field name="store_uuid" attrs="{'invisible': [('id', '=', False)]}"/>
                        <field name="name" attrs="{'readonly': [('id', '!=', False)]}"/>
                        <field name="phone_number" attrs="{'readonly': [('id', '!=', False)]}"/>
                        <label for="street" string="Address"/>
                        <div class="o_address_format">
                            <field name="street" placeholder="Street" class="o_address_street" attrs="{'readonly': [('id', '!=', False)]}"/>
                            <field name="house_number_or_name" placeholder="House number or name" class="o_address_street" attrs="{'readonly': [('id', '!=', False)]}"/>
                            <field name="city" placeholder="City" class="o_address_city" attrs="{'readonly': [('id', '!=', False)]}"/>
                            <field name="state_id" class="o_address_state" placeholder="State" options="{'no_open': True, 'no_quick_create': True}"
                                attrs="{'required': [('country_code', 'in', ['AU', 'CA', 'IT', 'US'])], 'readonly': [('id', '!=', False)]}"/>
                            <field name="zip" placeholder="ZIP" class="o_address_zip" attrs="{'readonly': [('id', '!=', False)]}"/>
                            <field name="country_id" placeholder="Country" class="o_address_country" options="{'no_open': True, 'no_create': True}" attrs="{'readonly': [('id', '!=', False)]}"/>
                            <field name="country_code" invisible="1"/>
                            <field name="terminal_ids" attrs="{'invisible': [('terminal_ids', '=', [])]}"/>
                        </div>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="action_pos_adyen_account" model="ir.actions.server">
        <field name="name">Adyen Account</field>
        <field name="model_id" ref="model_adyen_account"/>
        <field name="state">code</field>
        <field name="code">
            action = model.action_create_redirect()
        </field>
    </record>

    <menuitem id="menu_pos_adyen_account"
        parent="point_of_sale.menu_point_config_product"
        action="action_pos_adyen_account"
        sequence="5"
        groups="point_of_sale.group_pos_manager,point_of_sale.group_pos_user"/>

</odoo>

```

## File: views\point_of_sale_assets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="assets" inherit_id="point_of_sale.assets">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/pos_adyen/static/src/js/payment_adyen.js"></script>
            <script type="text/javascript" src="/pos_adyen/static/src/js/PaymentScreen.js"></script>
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
            <xpath expr="//div[@id='tip_product']" position="after">
                <div attrs="{'invisible': [('iface_tipproduct', '=', False)]}">
                    <field name="adyen_ask_customer_for_tip" class="oe_inline"/>
                    <label class="font-weight-normal" for="adyen_ask_customer_for_tip" string="Add tip through payment terminal (Adyen)"/>
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
                <!-- Adyen -->
                <field name="adyen_api_key" attrs="{'invisible': [('use_payment_terminal', '!=', 'adyen')], 'required': [('use_payment_terminal', '=', 'adyen')]}"/>
                <field name="adyen_terminal_identifier" attrs="{'invisible': [('use_payment_terminal', '!=', 'adyen')], 'required': [('use_payment_terminal', '=', 'adyen')]}"/>
                <field name="adyen_test_mode" attrs="{'invisible': [('use_payment_terminal', '!=', 'adyen')], 'required': [('use_payment_terminal', '=', 'adyen')]}"/>
                
                <!-- Odoo Payments by Adyen -->
                <field name="adyen_account_id" invisible="1"/>
                <field name="adyen_terminal_id" attrs="{'invisible': [('use_payment_terminal', '!=', 'odoo_adyen')], 'required': [('use_payment_terminal', '=', 'odoo_adyen')]}"/>
                <field name="adyen_payout_id" attrs="{'invisible': [('use_payment_terminal', '!=', 'odoo_adyen')], 'required': [('use_payment_terminal', '=', 'odoo_adyen')]}" options="{'no_create': True}"/>
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
        <field name="name">res.config.settings.view.form.inherit.point_of_sale</field>
        <field name="model">res.config.settings</field>
        <field name="priority" eval="95"/>
        <field name="inherit_id" ref="base.res_config_settings_view_form" />
        <field name="arch" type="xml">
            <xpath expr="//div[@id='adyen_payment_terminal_setting']/div[hasclass('o_setting_right_pane')]" position="inside">
                <span attrs="{'invisible': [('adyen_account_id', '=', False)]}">
                    <label for="adyen_account_id" string="Account"/>
                    <span class="fa fa-lg fa-building-o mr8" title="Values set here are company-specific." aria-label="Values set here are company-specific." groups="base.group_multi_company" role="img"/>
                    <field name="adyen_account_id"/>
                </span>
                <button name="create_adyen_account" icon="fa-arrow-right" type="object" class="btn-link" string="Create an account in 1 minute"
                    attrs="{'invisible': [('adyen_account_id', '!=', False)]}"/>
            </xpath>
        </field>
    </record>
</odoo>

```

