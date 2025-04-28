# Odoo Module: pos_mercury

Category: Sales/Point Of Sale

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Vantiv Payment Services',
    'version': '1.0',
    'category': 'Sales/Point Of Sale',
    'sequence': 6,
    'summary': 'Credit card support for Point Of Sale',
    'description': """
Allow credit card POS payments
==============================

This module allows customers to pay for their orders with credit
cards. The transactions are processed by Vantiv (developed by Wells
Fargo Bank). A Vantiv merchant account is necessary. It allows the
following:

* Fast payment by just swiping a credit card while on the payment screen
* Combining of cash payments and credit card payments
* Cashback
* Supported cards: Visa, MasterCard, American Express, Discover
    """,
    'depends': ['web', 'barcodes', 'point_of_sale'],
    'data': [
        'data/pos_mercury_data.xml',
        'security/ir.model.access.csv',
        'views/pos_mercury_templates.xml',
        'views/pos_mercury_views.xml',
        'views/pos_mercury_transaction_templates.xml',
        'views/pos_config_setting_views.xml',
        'views/pos_config_views.xml',
    ],
    'demo': [
        'data/pos_mercury_demo.xml',
    ],
    'qweb': [
        'static/src/xml/pos_mercury.xml',
    ],
    'installable': True,
    'auto_install': False,
    'license': 'LGPL-3',
}

```

## File: data\pos_mercury_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="barcode_rule_credit" model="barcode.rule">
            <field name="name">Magnetic Credit Card</field>
            <field name="barcode_nomenclature_id" ref="barcodes.default_barcode_nomenclature"/>
            <field name="sequence">85</field>
            <field name="type">credit</field>
            <field name="encoding">any</field>
            <field name="pattern">%.*</field>
        </record>
    </data>
</odoo>

```

## File: data\pos_mercury_demo.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data>
      <!-- Vantiv Production Test Account (e2e + token) -->
      <!-- This is a test account for testing with test cards and cannot be used in a live environment -->
      <record id="pos_mercury_configuration" model="pos_mercury.configuration">
        <field name="name">Vantiv Demo</field>
        <field name="merchant_id">755847002</field>
        <field name="merchant_pwd">xyz</field>
      </record>
    </data>
</odoo>

```

## File: models\pos_mercury.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from odoo import models, fields, api, _
from odoo.tools.float_utils import float_compare

_logger = logging.getLogger(__name__)


class BarcodeRule(models.Model):
    _inherit = 'barcode.rule'

    type = fields.Selection(selection_add=[
        ('credit', 'Credit Card')
    ])


class PosMercuryConfiguration(models.Model):
    _name = 'pos_mercury.configuration'
    _description = 'Point of Sale Vantiv Configuration'

    name = fields.Char(required=True, help='Name of this Vantiv configuration')
    merchant_id = fields.Char(string='Merchant ID', required=True, help='ID of the merchant to authenticate him on the payment provider server')
    merchant_pwd = fields.Char(string='Merchant Password', required=True, help='Password of the merchant to authenticate him on the payment provider server')


class PoSPayment(models.Model):
    _inherit = "pos.payment"

    mercury_card_number = fields.Char(string='Card Number', help='The last 4 numbers of the card used to pay')
    mercury_prefixed_card_number = fields.Char(string='Card Number Prefix', compute='_compute_prefixed_card_number', help='The card number used for the payment.')
    mercury_card_brand = fields.Char(string='Card Brand', help='The brand of the payment card (e.g. Visa, AMEX, ...)')
    mercury_card_owner_name = fields.Char(string='Card Owner Name', help='The name of the card owner')
    mercury_ref_no = fields.Char(string='Vantiv reference number', help='Payment reference number from Vantiv Pay')
    mercury_record_no = fields.Char(string='Vantiv record number', help='Payment record number from Vantiv Pay')
    mercury_invoice_no = fields.Char(string='Vantiv invoice number', help='Invoice number from Vantiv Pay')

    def _compute_prefixed_card_number(self):
        for line in self:
            if line.mercury_card_number:
                line.mercury_prefixed_card_number = "********" + line.mercury_card_number
            else:
                line.mercury_prefixed_card_number = ""


class PoSPaymentMethod(models.Model):
    _inherit = 'pos.payment.method'

    pos_mercury_config_id = fields.Many2one('pos_mercury.configuration', string='Vantiv Credentials', help='The configuration of Vantiv used for this journal')

    def _get_payment_terminal_selection(self):
        return super(PoSPaymentMethod, self)._get_payment_terminal_selection() + [('mercury', 'Vantiv')]

    @api.onchange('use_payment_terminal')
    def _onchange_use_payment_terminal(self):
        super(PoSPaymentMethod, self)._onchange_use_payment_terminal()
        if self.use_payment_terminal != 'mercury':
            self.pos_mercury_config_id = False

class PosOrder(models.Model):
    _inherit = "pos.order"

    @api.model
    def _payment_fields(self, order, ui_paymentline):
        fields = super(PosOrder, self)._payment_fields(order, ui_paymentline)

        fields.update({
            'mercury_card_number': ui_paymentline.get('mercury_card_number'),
            'mercury_card_brand': ui_paymentline.get('mercury_card_brand'),
            'mercury_card_owner_name': ui_paymentline.get('mercury_card_owner_name'),
            'mercury_ref_no': ui_paymentline.get('mercury_ref_no'),
            'mercury_record_no': ui_paymentline.get('mercury_record_no'),
            'mercury_invoice_no': ui_paymentline.get('mercury_invoice_no')
        })

        return fields


class AutoVacuum(models.AbstractModel):
    _inherit = 'ir.autovacuum'

    @api.model
    def power_on(self, *args, **kwargs):
        self.env['pos_mercury.mercury_transaction'].cleanup_old_tokens()
        return super(AutoVacuum, self).power_on(*args, **kwargs)

```

## File: models\pos_mercury_transaction.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import date, timedelta

import requests
import werkzeug

from odoo import models, api, service
from odoo.tools.translate import _
from odoo.exceptions import UserError
from odoo.tools import DEFAULT_SERVER_DATETIME_FORMAT, misc


class MercuryTransaction(models.Model):
    _name = 'pos_mercury.mercury_transaction'
    _description = 'Point of Sale Vantiv Transaction'

    def _get_pos_session(self):
        pos_session = self.env['pos.session'].search([('state', '=', 'opened'), ('user_id', '=', self.env.uid)], limit=1)
        if not pos_session:
            raise UserError(_("No opened point of sale session for user %s found.") % self.env.user.name)

        pos_session.login()

        return pos_session

    def _get_pos_mercury_config_id(self, config, payment_method_id):
        payment_method = config.current_session_id.payment_method_ids.filtered(lambda pm: pm.id == payment_method_id)

        if payment_method and payment_method.pos_mercury_config_id:
            return payment_method.pos_mercury_config_id
        else:
            raise UserError(_("No Vantiv configuration associated with the payment method."))

    def _setup_request(self, data):
        # todo: in master make the client include the pos.session id and use that
        pos_session = self._get_pos_session()

        config = pos_session.config_id
        pos_mercury_config = self._get_pos_mercury_config_id(config, data['payment_method_id'])

        data['operator_id'] = pos_session.user_id.login
        data['merchant_id'] = pos_mercury_config.sudo().merchant_id
        data['merchant_pwd'] = pos_mercury_config.sudo().merchant_pwd
        data['memo'] = "Odoo " + service.common.exp_version()['server_version']

    def _do_request(self, template, data):
        xml_transaction = self.env.ref(template).render(data).decode()

        if not data['merchant_id'] or not data['merchant_pwd']:
            return "not setup"

        soap_header = '<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:mer="http://www.mercurypay.com"><soapenv:Header/><soapenv:Body><mer:CreditTransaction><mer:tran>'
        soap_footer = '</mer:tran><mer:pw>' + data['merchant_pwd'] + '</mer:pw></mer:CreditTransaction></soapenv:Body></soapenv:Envelope>'
        xml_transaction = soap_header + misc.html_escape(xml_transaction) + soap_footer

        response = ''

        headers = {
            'Content-Type': 'text/xml',
            'SOAPAction': 'http://www.mercurypay.com/CreditTransaction',
        }

        url = 'https://w1.mercurypay.com/ws/ws.asmx'
        if self.env['ir.config_parameter'].sudo().get_param('pos_mercury.enable_test_env'):
            url = 'https://w1.mercurycert.net/ws/ws.asmx'

        try:
            r = requests.post(url, data=xml_transaction, headers=headers, timeout=65)
            r.raise_for_status()
            response = werkzeug.utils.unescape(r.content.decode())
        except Exception:
            response = "timeout"

        return response

    def _do_reversal_or_voidsale(self, data, is_voidsale):
        try:
            self._setup_request(data)
        except UserError:
            return "internal error"

        data['is_voidsale'] = is_voidsale
        response = self._do_request('pos_mercury.mercury_voidsale', data)
        return response

    @api.model
    def do_payment(self, data):
        try:
            self._setup_request(data)
        except UserError:
            return "internal error"

        response = self._do_request('pos_mercury.mercury_transaction', data)
        return response

    @api.model
    def do_reversal(self, data):
        return self._do_reversal_or_voidsale(data, False)

    @api.model
    def do_voidsale(self, data):
        return self._do_reversal_or_voidsale(data, True)

    def do_return(self, data):
        try:
            self._setup_request(data)
        except UserError:
            return "internal error"

        response = self._do_request('pos_mercury.mercury_return', data)
        return response

    # One time (the ones we use) Vantiv tokens are required to be
    # deleted after 6 months
    @api.model
    def cleanup_old_tokens(self):
        expired_creation_date = (date.today() - timedelta(days=6 * 30)).strftime(DEFAULT_SERVER_DATETIME_FORMAT)

        for order in self.env['pos.order'].search([('create_date', '<', expired_creation_date)]):
            order.ref_no = ""
            order.record_no = ""

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import pos_mercury
from . import pos_mercury_transaction

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_pos_mercury_configuration,mercury.configuration,model_pos_mercury_configuration,point_of_sale.group_pos_manager,1,1,1,1
access_pos_mercury_mercury_transaction,mercury.transaction,model_pos_mercury_mercury_transaction,,1,0,0,0
```

## File: static\src\js\pos_mercury.js

```javascript
odoo.define('pos_mercury.pos_mercury', function (require) {
"use strict";

var core    = require('web.core');
var rpc    = require('web.rpc');
var screens = require('point_of_sale.screens');
var gui     = require('point_of_sale.gui');
var pos_model = require('point_of_sale.models');

var _t      = core._t;

var PopupWidget = require('point_of_sale.popups');
var ScreenWidget = screens.ScreenWidget;
var PaymentScreenWidget = screens.PaymentScreenWidget;

pos_model.load_fields('pos.payment.method', 'pos_mercury_config_id');

pos_model.PosModel = pos_model.PosModel.extend({
    getOnlinePaymentMethods: function () {
        var online_payment_methods = [];

        $.each(this.payment_methods, function (i, payment_method) {
            if (payment_method.pos_mercury_config_id) {
                online_payment_methods.push({label: payment_method.name, item: payment_method.id});
            }
        });

        return online_payment_methods;
    },
    decodeMagtek: function (magtekInput) {
        // Regular expression to identify and extract data from the track 1 & 2 of the magnetic code
        var _track1_regex = /%B?([0-9]*)\^([A-Z\/ -_]*)\^([0-9]{4})(.{3})([^?]+)\?/;

        var track1 = magtekInput.match(_track1_regex);
        var magtek_generated = magtekInput.split('|');

        var to_return = {};
        try {
            track1.shift(); // get rid of complete match
            to_return['number'] = track1.shift().substr(-4);
            to_return['name'] = track1.shift();
            track1.shift(); // expiration date
            track1.shift(); // service code
            track1.shift(); // discretionary data
            track1.shift(); // zero pad

            magtek_generated.shift(); // track1 and track2
            magtek_generated.shift(); // clear text crc
            magtek_generated.shift(); // encryption counter
            to_return['encrypted_block'] = magtek_generated.shift();
            magtek_generated.shift(); // enc session id
            magtek_generated.shift(); // device serial
            magtek_generated.shift(); // magneprint data
            magtek_generated.shift(); // magneprint status
            magtek_generated.shift(); // enc track3
            to_return['encrypted_key'] = magtek_generated.shift();
            magtek_generated.shift(); // enc track1
            magtek_generated.shift(); // reader enc status

            return to_return;
        } catch (e) {
            return 0;
        }
    },
    decodeMercuryResponse: function (data) {
        // get rid of xml version declaration and just keep the RStream
        // from the response because the xml contains two version
        // declarations. One for the SOAP, and one for the content. Maybe
        // we should unpack the SOAP layer in python?
        data = data.replace(/.*<\?xml version="1.0"\?>/, "");
        data = data.replace(/<\/CreditTransactionResult>.*/, "");

        var xml = $($.parseXML(data));
        var cmd_response = xml.find("CmdResponse");
        var tran_response = xml.find("TranResponse");

        return {
            status: cmd_response.find("CmdStatus").text(),
            message: cmd_response.find("TextResponse").text(),
            error: cmd_response.find("DSIXReturnCode").text(),
            card_type: tran_response.find("CardType").text(),
            auth_code: tran_response.find("AuthCode").text(),
            acq_ref_data: tran_response.find("AcqRefData").text(),
            process_data: tran_response.find("ProcessData").text(),
            invoice_no: tran_response.find("InvoiceNo").text(),
            ref_no: tran_response.find("RefNo").text(),
            record_no: tran_response.find("RecordNo").text(),
            purchase: parseFloat(tran_response.find("Purchase").text()),
            authorize: parseFloat(tran_response.find("Authorize").text()),
        };
    }
});

var _paylineproto = pos_model.Paymentline.prototype;
pos_model.Paymentline = pos_model.Paymentline.extend({
    init_from_JSON: function (json) {
        _paylineproto.init_from_JSON.apply(this, arguments);

        this.paid = json.paid;
        this.mercury_card_number = json.mercury_card_number;
        this.mercury_card_brand = json.mercury_card_brand;
        this.mercury_card_owner_name = json.mercury_card_owner_name;
        this.mercury_ref_no = json.mercury_ref_no;
        this.mercury_record_no = json.mercury_record_no;
        this.mercury_invoice_no = json.mercury_invoice_no;
        this.mercury_auth_code = json.mercury_auth_code;
        this.mercury_data = json.mercury_data;
        this.mercury_swipe_pending = json.mercury_swipe_pending;

        this.set_credit_card_name();
    },
    export_as_JSON: function () {
        return _.extend(_paylineproto.export_as_JSON.apply(this, arguments), {paid: this.paid,
                                                                              mercury_card_number: this.mercury_card_number,
                                                                              mercury_card_brand: this.mercury_card_brand,
                                                                              mercury_card_owner_name: this.mercury_card_owner_name,
                                                                              mercury_ref_no: this.mercury_ref_no,
                                                                              mercury_record_no: this.mercury_record_no,
                                                                              mercury_invoice_no: this.mercury_invoice_no,
                                                                              mercury_auth_code: this.mercury_auth_code,
                                                                              mercury_data: this.mercury_data,
                                                                              mercury_swipe_pending: this.mercury_swipe_pending});
    },
    set_credit_card_name: function () {
        if (this.mercury_card_number) {
            this.name = this.mercury_card_brand + " (****" + this.mercury_card_number + ")";
        }
    }
});

// Lookup table to store status and error messages
var lookUpCodeTransaction = {
    'Approved': {
        '000000': _t('Transaction approved'),
    },
    'TimeoutError': {
        '001006': 'Global API Not Initialized',
        '001007': 'Timeout on Response',
        '003003': 'Socket Error sending request',
        '003004': 'Socket already open or in use',
        '003005': 'Socket Creation Failed',
        '003006': 'Socket Connection Failed',
        '003007': 'Connection Lost',
        '003008': 'TCP/IP Failed to Initialize',
        '003010': 'Time Out waiting for server response',
        '003011': 'Connect Canceled',
        '003053': 'Initialize Failed',
        '009999': 'Unknown Error',
    },
    'FatalError': {
        '-1':     'Timeout error',
        '001001': 'General Failure',
        '001003': 'Invalid Command Format',
        '001004': 'Insufficient Fields',
        '001011': 'Empty Command String',
        '002000': 'Password Verified',
        '002001': 'Queue Full',
        '002002': 'Password Failed – Disconnecting',
        '002003': 'System Going Offline',
        '002004': 'Disconnecting Socket',
        '002006': 'Refused ‘Max Connections’',
        '002008': 'Duplicate Serial Number Detected',
        '002009': 'Password Failed (Client / Server)',
        '002010': 'Password failed (Challenge / Response)',
        '002011': 'Internal Server Error – Call Provider',
        '003002': 'In Process with server',
        '003009': 'Control failed to find branded serial (password lookup failed)',
        '003012': '128 bit CryptoAPI failed',
        '003014': 'Threaded Auth Started Expect Response',
        '003017': 'Failed to start Event Thread.',
        '003050': 'XML Parse Error',
        '003051': 'All Connections Failed',
        '003052': 'Server Login Failed',
        '004001': 'Global Response Length Error (Too Short)',
        '004002': 'Unable to Parse Response from Global (Indistinguishable)',
        '004003': 'Global String Error',
        '004004': 'Weak Encryption Request Not Supported',
        '004005': 'Clear Text Request Not Supported',
        '004010': 'Unrecognized Request Format',
        '004011': 'Error Occurred While Decrypting Request',
        '004017': 'Invalid Check Digit',
        '004018': 'Merchant ID Missing',
        '004019': 'TStream Type Missing',
        '004020': 'Could Not Encrypt Response- Call Provider',
        '100201': 'Invalid Transaction Type',
        '100202': 'Invalid Operator ID',
        '100203': 'Invalid Memo',
        '100204': 'Invalid Account Number',
        '100205': 'Invalid Expiration Date',
        '100206': 'Invalid Authorization Code',
        '100207': 'Invalid Authorization Code',
        '100208': 'Invalid Authorization Amount',
        '100209': 'Invalid Cash Back Amount',
        '100210': 'Invalid Gratuity Amount',
        '100211': 'Invalid Purchase Amount',
        '100212': 'Invalid Magnetic Stripe Data',
        '100213': 'Invalid PIN Block Data',
        '100214': 'Invalid Derived Key Data',
        '100215': 'Invalid State Code',
        '100216': 'Invalid Date of Birth',
        '100217': 'Invalid Check Type',
        '100218': 'Invalid Routing Number',
        '100219': 'Invalid TranCode',
        '100220': 'Invalid Merchant ID',
        '100221': 'Invalid TStream Type',
        '100222': 'Invalid Batch Number',
        '100223': 'Invalid Batch Item Count',
        '100224': 'Invalid MICR Input Type',
        '100225': 'Invalid Driver’s License',
        '100226': 'Invalid Sequence Number',
        '100227': 'Invalid Pass Data',
        '100228': 'Invalid Card Type',
    },
};
// Popup to show all transaction state for the payment.

var PaymentTransactionPopupWidget = PopupWidget.extend({
    template: 'PaymentTransactionPopupWidget',
    show: function (options) {
        var self = this;
        this._super(options);
        options.transaction.then(function (data) {
            if (data.auto_close) {
                setTimeout(function () {
                    self.gui.close_popup();
                }, 2000);
            } else {
                self.close();
                self.$el.find('.popup').append('<div class="footer"><div class="button cancel">Ok</div></div>');
            }

            self.$el.find('p.body').html(data.message);
        }).progress(function (data) {
            self.$el.find('p.body').html(data.message);
        });
    }
});

gui.define_popup({name:'payment-transaction', widget: PaymentTransactionPopupWidget});

// On all screens, if a card is swipped, return a popup error.
ScreenWidget.include({
    credit_error_action: function () {
        this.gui.show_popup('error-barcode',_t('Go to payment screen to use cards'));
    },

    show: function () {
        this._super();
        if(this.pos.getOnlinePaymentMethods().length !== 0) {
            this.pos.barcode_reader.set_action_callback('credit', _.bind(this.credit_error_action, this));
        }
    }
});

// On Payment screen, allow online payments
PaymentScreenWidget.include({
    // How long we wait for the odoo server to deliver the response of
    // a Vantiv transaction
    server_timeout_in_ms: 95000,

    // How many Vantiv transactions we send without receiving a
    // response
    server_retries: 3,

    _get_swipe_pending_line: function () {
        var i = 0;
        var lines = this.pos.get_order().get_paymentlines();

        for (i = 0; i < lines.length; i++) {
            if (lines[i].mercury_swipe_pending) {
                return lines[i];
            }
        }

        return 0;
    },

    _does_credit_payment_line_exist: function (amount, card_number, card_brand, card_owner_name) {
        var i = 0;
        var lines = this.pos.get_order().get_paymentlines();

        for (i = 0; i < lines.length; i++) {
            if (lines[i].mercury_amount === amount &&
                lines[i].mercury_card_number === card_number &&
                lines[i].mercury_card_brand === card_brand &&
                lines[i].mercury_card_owner_name === card_owner_name) {
                return true;
            }
        }

        return false;
    },

    retry_mercury_transaction: function (def, response, retry_nr, can_connect_to_server, callback, args) {
        var self = this;
        var message = "";

        if (retry_nr < self.server_retries) {
            if (response) {
                message = "Retry #" + (retry_nr + 1) + "...<br/><br/>" + response.message;
            } else {
                message = "Retry #" + (retry_nr + 1) + "...";
            }
            def.notify({
                message: message
            });

            setTimeout(function () {
                callback.apply(self, args);
            }, 1000);
        } else {
            if (response) {
                message = "Error " + response.error + ": " + lookUpCodeTransaction["TimeoutError"][response.error] + "<br/>" + response.message;
            } else {
                if (can_connect_to_server) {
                    message = _t("No response from Vantiv (Vantiv down?)");
                } else {
                    message = _t("No response from server (connected to network?)");
                }
            }
            def.resolve({
                message: message,
                auto_close: false
            });
        }
    },

    // Handler to manage the card reader string
    credit_code_transaction: function (parsed_result, old_deferred, retry_nr) {
        var order = this.pos.get_order();
        if (order.get_due(order.selected_paymentline) < 0) {
            this.gui.show_popup('error',{
                'title': _t('Refunds not supported'),
                'body':  _t('Credit card refunds are not supported. Instead select your credit card payment method, click \'Validate\' and refund the original charge manually through the Vantiv backend.'),
            });
            return;
        }

        if(this.pos.getOnlinePaymentMethods().length === 0) {
            return;
        }

        var self = this;
        var decodedMagtek = self.pos.decodeMagtek(parsed_result.code);

        if (! decodedMagtek) {
            this.gui.show_popup('error',{
                'title': _t('Could not read card'),
                'body':  _t('This can be caused by a badly executed swipe or by not having your keyboard layout set to US QWERTY (not US International).'),
            });
            return;
        }

        var swipe_pending_line = self._get_swipe_pending_line();
        var purchase_amount = 0;

        if (swipe_pending_line) {
            purchase_amount = swipe_pending_line.get_amount();
        } else {
            purchase_amount = self.pos.get_order().get_due();
        }

        var transaction = {
            'encrypted_key'     : decodedMagtek['encrypted_key'],
            'encrypted_block'   : decodedMagtek['encrypted_block'],
            'transaction_type'  : 'Credit',
            'transaction_code'  : 'Sale',
            'invoice_no'        : self.pos.get_order().uid.replace(/-/g,''),
            'purchase'          : purchase_amount,
            'payment_method_id' : parsed_result.payment_method_id,
        };

        var def = old_deferred || new $.Deferred();
        retry_nr = retry_nr || 0;

        // show the transaction popup.
        // the transaction deferred is used to update transaction status
        // if we have a previous deferred it indicates that this is a retry
        if (! old_deferred) {
            self.gui.show_popup('payment-transaction', {
                transaction: def
            });
            def.notify({
                message: _t('Handling transaction...'),
            });
        }

        rpc.query({
                model: 'pos_mercury.mercury_transaction',
                method: 'do_payment',
                args: [transaction],
            }, {
                timeout: self.server_timeout_in_ms,
            })
            .then(function (data) {
                // if not receiving a response from Vantiv, we should retry
                if (data === "timeout") {
                    self.retry_mercury_transaction(def, null, retry_nr, true, self.credit_code_transaction, [parsed_result, def, retry_nr + 1]);
                    return;
                }

                if (data === "not setup") {
                    def.resolve({
                        message: _t("Please setup your Vantiv merchant account.")
                    });
                    return;
                }

                if (data === "internal error") {
                    def.resolve({
                        message: _t("Odoo error while processing transaction.")
                    });
                    return;
                }

                var response = self.pos.decodeMercuryResponse(data);
                response.payment_method_id = parsed_result.payment_method_id;

                if (response.status === 'Approved') {
                    // AP* indicates a duplicate request, so don't add anything for those
                    if (response.message === "AP*" && self._does_credit_payment_line_exist(response.authorize, decodedMagtek['number'],
                                                                                        response.card_type, decodedMagtek['name'])) {
                        def.resolve({
                            message: lookUpCodeTransaction["Approved"][response.error],
                            auto_close: true,
                        });
                    } else {
                        // If the payment is approved, add a payment line
                        var order = self.pos.get_order();

                        if (swipe_pending_line) {
                            order.select_paymentline(swipe_pending_line);
                        } else {
                            order.add_paymentline(self.pos.payment_methods_by_id[parsed_result.payment_method_id]);
                        }

                        order.selected_paymentline.paid = true;
                        order.selected_paymentline.mercury_swipe_pending = false;
                        order.selected_paymentline.mercury_amount = response.authorize;
                        order.selected_paymentline.set_amount(response.authorize);
                        order.selected_paymentline.mercury_card_number = decodedMagtek['number'];
                        order.selected_paymentline.mercury_card_brand = response.card_type;
                        order.selected_paymentline.mercury_card_owner_name = decodedMagtek['name'];
                        order.selected_paymentline.mercury_ref_no = response.ref_no;
                        order.selected_paymentline.mercury_record_no = response.record_no;
                        order.selected_paymentline.mercury_invoice_no = response.invoice_no;
                        order.selected_paymentline.mercury_auth_code = response.auth_code;
                        order.selected_paymentline.mercury_data = response; // used to reverse transactions
                        order.selected_paymentline.set_credit_card_name();

                        self.order_changes();
                        self.reset_input();
                        self.render_paymentlines();
                        order.trigger('change', order); // needed so that export_to_JSON gets triggered

                        if (response.message === "PARTIAL AP") {
                            def.resolve({
                                message: _t("Partially approved"),
                                auto_close: false,
                            });
                        } else {
                            def.resolve({
                                message: lookUpCodeTransaction["Approved"][response.error],
                                auto_close: true,
                            });
                        }
                    }
                }

                // if an error related to timeout or connectivity issues arised, then retry the same transaction
                else {
                    if (lookUpCodeTransaction["TimeoutError"][response.error]) { // recoverable error
                        self.retry_mercury_transaction(def, response, retry_nr, true, self.credit_code_transaction, [parsed_result, def, retry_nr + 1]);
                    } else { // not recoverable
                        def.resolve({
                            message: "Error " + response.error + ":<br/>" + response.message,
                            auto_close: false
                        });
                    }
                }

            }).catch(function () {
                self.retry_mercury_transaction(def, null, retry_nr, false, self.credit_code_transaction, [parsed_result, def, retry_nr + 1]);
            });
    },

    credit_code_cancel: function () {
        return;
    },

    credit_code_action: function (parsed_result) {
        var self = this;
        var online_payment_methods = this.pos.getOnlinePaymentMethods();

        if (online_payment_methods.length === 1) {
            parsed_result.payment_method_id = online_payment_methods[0].item;
            self.credit_code_transaction(parsed_result);
        } else { // this is for supporting another payment system like mercury
            this.gui.show_popup('selection',{
                title:   _t('Pay with: '),
                list:    online_payment_methods,
                confirm: function (item) {
                    parsed_result.payment_method_id = item;
                    self.credit_code_transaction(parsed_result);
                },
                cancel:  self.credit_code_cancel,
            });
        }
    },

    remove_paymentline_by_ref: function (line) {
        this.pos.get_order().remove_paymentline(line);
        this.reset_input();
        this.render_paymentlines();
    },

    do_reversal: function (line, is_voidsale, old_deferred, retry_nr) {
        var def = old_deferred || new $.Deferred();
        var self = this;
        retry_nr = retry_nr || 0;

        // show the transaction popup.
        // the transaction deferred is used to update transaction status
        this.gui.show_popup('payment-transaction', {
            transaction: def
        });

        var request_data = _.extend({
            'transaction_type': 'Credit',
            'transaction_code': 'VoidSaleByRecordNo',
        }, line.mercury_data);

        var message = "";
        var rpc_method = "";

        if (is_voidsale) {
            message = _t("Reversal failed, sending VoidSale...");
            rpc_method = "do_voidsale";
        } else {
            message = _t("Sending reversal...");
            rpc_method = "do_reversal";
        }

        if (! old_deferred) {
            def.notify({
                message: message,
            });
        }

        rpc.query({
                model: 'pos_mercury.mercury_transaction',
                method: rpc_method,
                args: [request_data],
            }, {
                timeout: self.server_timeout_in_ms
            })
            .then(function (data) {
                if (data === "timeout") {
                    self.retry_mercury_transaction(def, null, retry_nr, true, self.do_reversal, [line, is_voidsale, def, retry_nr + 1]);
                    return;
                }

                if (data === "internal error") {
                    def.resolve({
                        message: _t("Odoo error while processing transaction.")
                    });
                    return;
                }

                var response = self.pos.decodeMercuryResponse(data);

                if (! is_voidsale) {
                    if (response.status != 'Approved' || response.message != 'REVERSED') {
                        // reversal was not successful, send voidsale
                        self.do_reversal(line, true);
                    } else {
                        // reversal was successful
                        def.resolve({
                            message: _t("Reversal succeeded"),
                        });

                        self.remove_paymentline_by_ref(line);
                    }
                } else { // voidsale ended, nothing more we can do
                    if (response.status === 'Approved') {
                        def.resolve({
                            message: _t("VoidSale succeeded"),
                        });

                        self.remove_paymentline_by_ref(line);
                    } else {
                        def.resolve({
                            message: "Error " + response.error + ":<br/>" + response.message,
                        });
                    }
                }
            }).catch(function () {
                self.retry_mercury_transaction(def, null, retry_nr, false, self.do_reversal, [line, is_voidsale, def, retry_nr + 1]);
            });
    },

    click_delete_paymentline: function (cid) {
        var lines = this.pos.get_order().get_paymentlines();

        for (var i = 0; i < lines.length; i++) {
            if (lines[i].cid === cid && lines[i].mercury_data) {
                this.do_reversal(lines[i], false);
                return;
            }
        }

        this._super(cid);
    },

    // make sure there is only one paymentline waiting for a swipe
    click_paymentmethods: function (id) {
        var i;
        var order = this.pos.get_order();
        var payment_method = this.pos.payment_methods_by_id[id]
        // this.pos.get_order().add_paymentline(payment_method);

        if (payment_method.pos_mercury_config_id) {
            var already_swipe_pending = false;
            var lines = order.get_paymentlines();

            for (i = 0; i < lines.length; i++) {
                if (lines[i].payment_method.pos_mercury_config_id && lines[i].mercury_swipe_pending) {
                    already_swipe_pending = true;
                }
            }

            if (already_swipe_pending) {
                this.gui.show_popup('error',{
                    'title': _t('Error'),
                    'body':  _t('One credit card swipe already pending.'),
                });
            } else {
                this._super(id);
                if (order.get_due(order.selected_paymentline) > 0) {
                    order.selected_paymentline.mercury_swipe_pending = true;
                    this.render_paymentlines();
                    order.trigger('change', order); // needed so that export_to_JSON gets triggered
                }
            }
        } else {
            this._super(id);
        }
    },

    show: function () {
        this._super();
        if (this.pos.getOnlinePaymentMethods().length !== 0) {
            this.pos.barcode_reader.set_action_callback('credit', _.bind(this.credit_code_action, this));
        }
    },

    // before validating, get rid of any paymentlines that are waiting
    // on a swipe.
    validate_order: function(force_validation) {
        if (this.pos.get_order().is_paid() && ! this.invoicing) {
            var lines = this.pos.get_order().get_paymentlines();

            for (var i = 0; i < lines.length; i++) {
                if (lines[i].mercury_swipe_pending) {
                    this.pos.get_order().remove_paymentline(lines[i]);
                    this.render_paymentlines();
                }
            }
        }

        this._super(force_validation);
    }
});

});

```

## File: static\src\xml\pos_mercury.xml

```xml
<?xml version="1.0" encoding="utf-8"?>

<templates id="template" inherit_id="point_of_sale.template">
  <t t-name="PaymentTransactionPopupWidget">
    <div role="dialog" class="modal-dialog">
      <div class="popup">
        <p class="title">Online Payment</p>
        <p class="body"></p>
      </div>
    </div>
  </t>

  <t t-extend="PaymentScreen-Paymentlines">
    <t t-jquery=".col-name" t-operation="inner">
      <t t-if="! line.payment_method.is_cash_count">
        <t t-if="line.mercury_swipe_pending">
          <div>WAITING FOR SWIPE</div>
        </t>
        <t t-if="! line.mercury_swipe_pending">
          <t t-esc='line.name' />
        </t>
      </t>
      <t t-if="line.payment_method.is_cash_count">
        <t t-esc='line.name' />
      </t>
    </t>
    <t t-jquery="tbody tr.paymentline.selected">
      this.removeAttr('class');
      this.attr('t-attf-class', 'paymentline selected #{line.mercury_swipe_pending ? \'o_pos_mercury_swipe_pending\' : \'\'}');
    </t>
    <t t-jquery="tbody tr.paymentline[t-att-data-cid*='line.cid']">
      this.removeAttr('class');
      this.attr('t-attf-class', 'paymentline #{line.mercury_swipe_pending ? \'o_pos_mercury_swipe_pending\' : \'\'}');
    </t>
    <t t-jquery="tbody tr td.col-tendered.edit">
      this.removeAttr('class');
      this.attr('t-attf-class', 'col-tendered edit #{line.mercury_swipe_pending ? \'o_pos_mercury_swipe_pending\' : \'\'}');
    </t>
  </t>

  <t t-name="PosMercurySignature">
    <t t-foreach="paymentlines" t-as="paymentline">
      <t t-if="paymentline.mercury_data &amp;&amp; ! printed_signature">
        <br />
        <div>CARDHOLDER WILL PAY CARD ISSUER</div>
        <div>ABOVE AMOUNT PURSUANT</div>
        <div>TO CARDHOLDER AGREEMENT</div>
        <br />
        <br />
        <div>X______________________________</div>
        <t t-set="printed_signature" t-value="true"/>
      </t>
    </t>
  </t>

  <t t-extend="OrderReceipt">
    <t t-jquery="t[t-foreach*='paymentlines'][t-as*='line']" t-operation="append">
      <t t-if="line.mercury_data">
        <div class="pos-receipt-left-padding">
          APPROVAL CODE: <t t-esc="line.mercury_auth_code"/>
        </div>
      </t>
    </t>
    <t t-jquery="receipt" t-operation="append">
      <div>
        <t t-call="PosMercurySignature"/>
      </div>
    </t>
  </t>

</templates>

```

## File: views\pos_config_setting_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form_inherit_pos_mercury" model="ir.ui.view">
        <field name="name">res.config.form.inherit.mercury</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="point_of_sale.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <div id="btn_use_pos_mercury" position="replace">
                <div class="mt16">
                    <button name="%(pos_mercury.action_configuration_form)d" icon="fa-arrow-right" type="action" string="Vantiv Accounts" class="btn-link"/>
                </div>
                <div>
                    <a href="https://www.odoo.com/page/point-of-sale-hardware#part_8" target="_blank"><i class="fa fa-fw fa-arrow-right"/>Buy a card reader</a>
                </div>
            </div>
        </field>
    </record>
</odoo>

```

## File: views\pos_config_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="pos_config_view_form_inherit_pos_mercury" model="ir.ui.view">
        <field name="name">pos.config.form.view.inherit.mercury</field>
        <field name="model">pos.config</field>
        <field name="inherit_id" ref="point_of_sale.pos_config_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[hasclass('iot_barcode_scanner')]//label" position="attributes">
                <attribute name="string">Barcode Scanner/Card Reader</attribute>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\pos_mercury_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <template id="assets" inherit_id="point_of_sale.assets">
            <xpath expr="." position="inside">
                <script type="text/javascript" src="/pos_mercury/static/src/js/pos_mercury.js"></script>
                <link rel="stylesheet" href="/pos_mercury/static/src/css/pos_mercury.css"/>
            </xpath>
        </template>

    </data>
</odoo>

```

## File: views\pos_mercury_transaction_templates.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<odoo>
  <data>
    <template id='mercury_common'>
      <MerchantID><t t-esc='merchant_id'/></MerchantID>
      <TranType><t t-esc='transaction_type'/></TranType>
      <TranCode><t t-esc='transaction_code'/></TranCode>
      <InvoiceNo><t t-esc='invoice_no'/></InvoiceNo>
      <Frequency t-translation="off">OneTime</Frequency>
    </template>

    <template id='mercury_transaction'>
      <TStream t-translation="off">
        <Transaction>
          <t t-call='pos_mercury.mercury_common'/>
          <OperatorID><t t-esc='operator_id'/></OperatorID>
          <RecordNo>RecordNumberRequested</RecordNo>
          <PartialAuth>Allow</PartialAuth>
          <Memo><t t-esc='memo'/></Memo>
          <Account>
            <AccountSource>Swiped</AccountSource>
            <EncryptedBlock><t t-esc='encrypted_block'/></EncryptedBlock>
            <EncryptedKey><t t-esc='encrypted_key'/></EncryptedKey>
            <EncryptedFormat>MagneSafe</EncryptedFormat>
          </Account>
          <Amount>
            <Purchase><t t-esc='"%.2f" % purchase'/></Purchase> <!-- format required by Vantiv -->
          </Amount>
        </Transaction>
      </TStream>
    </template>

    <template id='mercury_voidsale'>
      <TStream t-translation="off">
        <Transaction>
          <t t-call='pos_mercury.mercury_common'/>
          <RefNo><t t-esc='ref_no'/></RefNo>
          <RecordNo><t t-esc='record_no'/></RecordNo>
          <Memo><t t-esc='memo'/></Memo>
          <Amount>
            <Purchase><t t-esc='"%.2f" % authorize'/></Purchase> <!-- format required by Vantiv -->
          </Amount>
          <TranInfo>
            <AuthCode><t t-esc='auth_code'/></AuthCode>
            <t t-if='not is_voidsale'>
              <AcqRefData><t t-esc='acq_ref_data'/></AcqRefData>
              <ProcessData><t t-esc='process_data'/></ProcessData>
            </t>
          </TranInfo>
        </Transaction>
      </TStream>
    </template>

    <template id='mercury_return'>
      <TStream t-translation="off">
        <Transaction>
          <t t-call='pos_mercury.mercury_common'/>
          <OperatorID><t t-esc='operator_id'/></OperatorID>
          <PartialAuth>Allow</PartialAuth>
          <RefNo><t t-esc='ref_no'/></RefNo>
          <RecordNo><t t-esc='record_no'/></RecordNo>
          <Amount>
            <Purchase><t t-esc='"%.2f" % purchase'/></Purchase>
          </Amount>
        </Transaction>
      </TStream>
    </template>
  </data>
</odoo>

```

## File: views\pos_mercury_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data>
        <record id="view_pos_mercury_configuration_form" model="ir.ui.view" >
            <field name="name">Vantiv Configurations</field>
            <field name="model">pos_mercury.configuration</field>
            <field name="arch" type="xml">
                <form string="Card Reader">
                    <sheet>
                        <div class="oe_title">
                           <label for="name" class="oe_edit_only"/>
                           <h1><field name="name"/></h1>
                        </div>
                        <div>
                            <p>
                                <i>Vantiv Configurations</i> define what Vantiv account will be used when
                                processing credit card transactions in the Point Of Sale. Setting up a Vantiv
                                configuration will enable you to allow payments with various credit cards
                                (eg. Visa, MasterCard, Discovery, American Express, ...). After setting up this
                                configuration you should associate it with a Point Of Sale payment method.
                            </p><p>
                                We currently support the MagTek Dynamag card reader device. It can be connected
                                directly to the Point Of Sale device or it can be connected to the IoTBox.
                            </p><p>
                                Using the Vantiv integration in the Point Of Sale is easy: just press the
                                associated payment method. After that the amount can be adjusted (eg. for cashback)
                                just like on any other payment line. Whenever the payment line is set up, a card
                                can be swiped through the card reader device.
                            </p><p>
                                For quickly handling orders: just swiping a credit card when on the payment screen
                                (without having pressed anything else) will charge the full amount of the order to
                                the card.
                            </p><p>
                                If you don't already have a Vantiv account, contact Vantiv at +1 (800) 846-4472
                                to create one.
                            </p>
                        </div>
                        <group col="2">
                            <field name="merchant_id"/>
                            <field name="merchant_pwd"/>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="view_pos_mercury_configuration_tree" model="ir.ui.view">
            <field name="name">Vantiv Configurations</field>
            <field name="model">pos_mercury.configuration</field>
            <field name="arch" type="xml">
                <tree string="Card Reader">
                    <field name="name"/>
                    <field name="merchant_id"/>
                </tree>
            </field>
        </record>

        <record id="action_configuration_form" model="ir.actions.act_window">
            <field name="name">Vantiv Configurations</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">pos_mercury.configuration</field>
            <field name="view_mode">tree,kanban,form</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Configure your card reader
              </p>
            </field>
        </record>

        <record id="pos_payment_method_view_form" model="ir.ui.view">
            <field name="name">Payment Method</field>
            <field name="model">pos.payment.method</field>
            <field name="inherit_id" ref="point_of_sale.pos_payment_method_view_form"></field>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='use_payment_terminal']" position="after">
                    <field name="pos_mercury_config_id" attrs="{'invisible': [('use_payment_terminal', '!=', 'mercury')], 'required': [('use_payment_terminal', '=', 'mercury')]}"/>
                </xpath>
            </field>
        </record>

        <record id="view_pos_order" model="ir.ui.view">
          <field name="name">POS orders</field>
          <field name="model">pos.order</field>
          <field name="inherit_id" ref="point_of_sale.view_pos_pos_form"/>
          <field name="arch" type="xml">
            <xpath expr="//field[@name='amount']" position="before">
              <field name="mercury_prefixed_card_number" string="Card Number"/>
              <field name="mercury_card_brand"/>
              <field name="mercury_card_owner_name"/>
            </xpath>
          </field>
        </record>

        <menuitem parent="point_of_sale.menu_point_config_product" action="pos_mercury.action_configuration_form" id="menu_pos_pos_mercury_config" groups="base.group_no_one" sequence="35"/>
    </data>
</odoo>

```

