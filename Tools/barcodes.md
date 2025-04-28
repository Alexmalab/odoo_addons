# Odoo Module: barcodes

Category: Tools

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-

from . import models

from odoo import api, SUPERUSER_ID


def _assign_default_nomeclature_id(cr, registry):
    env = api.Environment(cr, SUPERUSER_ID, {})
    company_ids_without_default_nomenclature_id  = env['res.company'].search([
        ('nomenclature_id', '=', False)
    ])
    default_nomenclature_id = env.ref('barcodes.default_barcode_nomenclature', raise_if_not_found=False)
    if default_nomenclature_id:
        company_ids_without_default_nomenclature_id.write({
            'nomenclature_id': default_nomenclature_id.id,
        })


```

## File: __manifest__.py

```python
{
    'name': 'Barcode',
    'version': '2.0',
    'category': 'Tools',
    'summary': 'Scan and Parse Barcodes',
    'depends': ['web'],
    'data': [
        'data/barcodes_data.xml',
        'views/barcodes_view.xml',
        'security/ir.model.access.csv',
        'views/barcodes_templates.xml',
    ],
    'installable': True,
    'auto_install': False,
    'post_init_hook': '_assign_default_nomeclature_id',
    'license': 'LGPL-3',
}

```

## File: data\barcodes_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    	<record id="default_barcode_nomenclature" model="barcode.nomenclature">
            <field name="name">Default Nomenclature</field>
        </record>

        <record id="barcode_rule_product" model="barcode.rule">
            <field name="name">Product Barcodes</field>
            <field name="barcode_nomenclature_id" ref="default_barcode_nomenclature"/>
            <field name="sequence">90</field>
            <field name="type">product</field>
            <field name="encoding">any</field>
            <field name="pattern">.*</field>
        </record>
</odoo>

```

## File: models\barcodes.py

```python
import logging
import re

from odoo import tools, models, fields, api, _
from odoo.exceptions import ValidationError

_logger = logging.getLogger(__name__)


UPC_EAN_CONVERSIONS = [
    ('none','Never'),
    ('ean2upc','EAN-13 to UPC-A'),
    ('upc2ean','UPC-A to EAN-13'),
    ('always','Always'),
]

class BarcodeNomenclature(models.Model):
    _name = 'barcode.nomenclature'
    _description = 'Barcode Nomenclature'

    name = fields.Char(string='Barcode Nomenclature', size=32, required=True, help='An internal identification of the barcode nomenclature')
    rule_ids = fields.One2many('barcode.rule', 'barcode_nomenclature_id', string='Rules', help='The list of barcode rules')
    upc_ean_conv = fields.Selection(UPC_EAN_CONVERSIONS, string='UPC/EAN Conversion', required=True, default='always',
        help="UPC Codes can be converted to EAN by prefixing them with a zero. This setting determines if a UPC/EAN barcode should be automatically converted in one way or another when trying to match a rule with the other encoding.")

    # returns the checksum of the ean13, or -1 if the ean has not the correct length, ean must be a string
    def ean_checksum(self, ean):
        code = list(ean)
        if len(code) != 13:
            return -1

        oddsum = evensum = total = 0
        code = code[:-1] # Remove checksum
        for i in range(len(code)):
            if i % 2 == 0:
                evensum += int(code[i])
            else:
                oddsum += int(code[i])
        total = oddsum * 3 + evensum
        return int((10 - total % 10) % 10)

    # returns the checksum of the ean8, or -1 if the ean has not the correct length, ean must be a string
    def ean8_checksum(self,ean):
        code = list(ean)
        if len(code) != 8:
            return -1

        sum1  = int(ean[1]) + int(ean[3]) + int(ean[5])
        sum2  = int(ean[0]) + int(ean[2]) + int(ean[4]) + int(ean[6])
        total = sum1 + 3 * sum2
        return int((10 - total % 10) % 10)

    # returns true if the barcode is a valid EAN barcode
    def check_ean(self, ean):
       return re.match("^\d+$", ean) and self.ean_checksum(ean) == int(ean[-1])

    # returns true if the barcode string is encoded with the provided encoding.
    def check_encoding(self, barcode, encoding):
        if encoding == 'ean13':
            return len(barcode) == 13 and re.match("^\d+$", barcode) and self.ean_checksum(barcode) == int(barcode[-1]) 
        elif encoding == 'ean8':
            return len(barcode) == 8 and re.match("^\d+$", barcode) and self.ean8_checksum(barcode) == int(barcode[-1])
        elif encoding == 'upca':
            return len(barcode) == 12 and re.match("^\d+$", barcode) and self.ean_checksum("0"+barcode) == int(barcode[-1])
        elif encoding == 'any':
            return True
        else:
            return False


    # Returns a valid zero padded ean13 from an ean prefix. the ean prefix must be a string.
    def sanitize_ean(self, ean):
        ean = ean[0:13]
        ean = ean + (13-len(ean))*'0'
        return ean[0:12] + str(self.ean_checksum(ean))

    # Returns a valid zero padded UPC-A from a UPC-A prefix. the UPC-A prefix must be a string.
    def sanitize_upc(self, upc):
        return self.sanitize_ean('0'+upc)[1:]

    # Checks if barcode matches the pattern
    # Additionaly retrieves the optional numerical content in barcode
    # Returns an object containing:
    # - value: the numerical value encoded in the barcode (0 if no value encoded)
    # - base_code: the barcode in which numerical content is replaced by 0's
    # - match: boolean
    def match_pattern(self, barcode, pattern):
        match = {
            "value": 0,
            "base_code": barcode,
            "match": False,
        }

        barcode = barcode.replace("\\", "\\\\").replace("{", '\{').replace("}", "\}").replace(".", "\.")
        numerical_content = re.search("[{][N]*[D]*[}]", pattern) # look for numerical content in pattern

        if numerical_content: # the pattern encodes a numerical content
            num_start = numerical_content.start() # start index of numerical content
            num_end = numerical_content.end() # end index of numerical content
            value_string = barcode[num_start:num_end-2] # numerical content in barcode

            whole_part_match = re.search("[{][N]*[D}]", numerical_content.group()) # looks for whole part of numerical content
            decimal_part_match = re.search("[{N][D]*[}]", numerical_content.group()) # looks for decimal part
            whole_part = value_string[:whole_part_match.end()-2] # retrieve whole part of numerical content in barcode
            decimal_part = "0." + value_string[decimal_part_match.start():decimal_part_match.end()-1] # retrieve decimal part
            if whole_part == '':
                whole_part = '0'
            match['value'] = int(whole_part) + float(decimal_part)

            match['base_code'] = barcode[:num_start] + (num_end-num_start-2)*"0" + barcode[num_end-2:] # replace numerical content by 0's in barcode
            match['base_code'] = match['base_code'].replace("\\\\", "\\").replace("\{", "{").replace("\}","}").replace("\.",".")
            pattern = pattern[:num_start] + (num_end-num_start-2)*"0" + pattern[num_end:] # replace numerical content by 0's in pattern to match

        match['match'] = re.match(pattern, match['base_code'][:len(pattern)])

        return match

    # Attempts to interpret an barcode (string encoding a barcode)
    # It will return an object containing various information about the barcode.
    # most importantly : 
    #  - code    : the barcode
    #  - type   : the type of the barcode: 
    #  - value  : if the id encodes a numerical value, it will be put there
    #  - base_code : the barcode code with all the encoding parts set to zero; the one put on
    #                the product in the backend
    def parse_barcode(self, barcode):
        parsed_result = {
            'encoding': '', 
            'type': 'error', 
            'code': barcode, 
            'base_code': barcode, 
            'value': 0,
        }

        rules = []
        for rule in self.rule_ids:
            rules.append({'type': rule.type, 'encoding': rule.encoding, 'sequence': rule.sequence, 'pattern': rule.pattern, 'alias': rule.alias})

        for rule in rules:
            cur_barcode = barcode
            if rule['encoding'] == 'ean13' and self.check_encoding(barcode,'upca') and self.upc_ean_conv in ['upc2ean','always']:
                cur_barcode = '0'+cur_barcode
            elif rule['encoding'] == 'upca' and self.check_encoding(barcode,'ean13') and barcode[0] == '0' and self.upc_ean_conv in ['ean2upc','always']:
                cur_barcode = cur_barcode[1:]

            if not self.check_encoding(barcode,rule['encoding']):
                continue

            match = self.match_pattern(cur_barcode, rule['pattern'])
            if match['match']:
                if rule['type'] == 'alias':
                    barcode = rule['alias']
                    parsed_result['code'] = barcode
                else:
                    parsed_result['encoding'] = rule['encoding']
                    parsed_result['type'] = rule['type']
                    parsed_result['value'] = match['value']
                    parsed_result['code'] = cur_barcode
                    if rule['encoding'] == "ean13":
                        parsed_result['base_code'] = self.sanitize_ean(match['base_code'])
                    elif rule['encoding'] == "upca":
                        parsed_result['base_code'] = self.sanitize_upc(match['base_code'])
                    else:
                        parsed_result['base_code'] = match['base_code']
                    return parsed_result

        return parsed_result

class BarcodeRule(models.Model):
    _name = 'barcode.rule'
    _description = 'Barcode Rule'
    _order = 'sequence asc'


    name = fields.Char(string='Rule Name', size=32, required=True, help='An internal identification for this barcode nomenclature rule')
    barcode_nomenclature_id = fields.Many2one('barcode.nomenclature', string='Barcode Nomenclature')
    sequence = fields.Integer(string='Sequence', help='Used to order rules such that rules with a smaller sequence match first')
    encoding = fields.Selection([
                ('any', 'Any'),
                ('ean13', 'EAN-13'),
                ('ean8', 'EAN-8'),
                ('upca', 'UPC-A'),
        ], string='Encoding', required=True, default='any', help='This rule will apply only if the barcode is encoded with the specified encoding')
    type = fields.Selection([
            ('alias', 'Alias'),
            ('product', 'Unit Product')
        ], string='Type', required=True, default='product')
    pattern = fields.Char(string='Barcode Pattern', size=32, help="The barcode matching pattern", required=True, default='.*')
    alias = fields.Char(string='Alias', size=32, default='0', help='The matched pattern will alias to this barcode', required=True)

    @api.constrains('pattern')
    def _check_pattern(self):
        for rule in self:
            p = rule.pattern.replace("\\\\", "X").replace("\{", "X").replace("\}", "X")
            findall = re.findall("[{]|[}]", p) # p does not contain escaped { or }
            if len(findall) == 2:
                if not re.search("[{][N]*[D]*[}]", p):
                    raise ValidationError(_("There is a syntax error in the barcode pattern ") + rule.pattern + _(": braces can only contain N's followed by D's."))
                elif re.search("[{][}]", p):
                    raise ValidationError(_("There is a syntax error in the barcode pattern ") + rule.pattern + _(": empty braces."))
            elif len(findall) != 0:
                raise ValidationError(_("There is a syntax error in the barcode pattern ") + rule.pattern + _(": a rule can only contain one pair of braces."))
            elif p == '*':
                raise ValidationError(_(" '*' is not a valid Regex Barcode Pattern. Did you mean '.*' ?"))

```

## File: models\barcode_events_mixin.py

```python
# -*- coding: utf-8 -*-

from odoo import models, fields, api

class BarcodeEventsMixin(models.AbstractModel):
    """ Mixin class for objects reacting when a barcode is scanned in their form views
        which contains `<field name="_barcode_scanned" widget="barcode_handler"/>`.
        Models using this mixin must implement the method on_barcode_scanned. It works
        like an onchange and receives the scanned barcode in parameter.
    """

    _name = 'barcodes.barcode_events_mixin'
    _description = 'Barcode Event Mixin'

    _barcode_scanned = fields.Char("Barcode Scanned", help="Value of the last barcode scanned.", store=False)

    @api.onchange('_barcode_scanned')
    def _on_barcode_scanned(self):
        barcode = self._barcode_scanned
        if barcode:
            self._barcode_scanned = ""
            return self.on_barcode_scanned(barcode)

    def on_barcode_scanned(self, barcode):
        raise NotImplementedError("In order to use barcodes.barcode_events_mixin, method on_barcode_scanned must be implemented")

```

## File: models\ir_http.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class IrHttp(models.AbstractModel):
    _inherit = 'ir.http'

    def session_info(self):
        res = super(IrHttp, self).session_info()
        if self.env.user.has_group('base.group_user'):
            res['max_time_between_keys_in_ms'] = int(
                self.env['ir.config_parameter'].sudo().get_param('barcode.max_time_between_keys_in_ms', default='100'))
        return res

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-

from odoo import models, fields


class ResCompany(models.Model):
    _inherit = 'res.company'

    def _get_default_nomenclature(self):
        return self.env.ref('barcodes.default_barcode_nomenclature', raise_if_not_found=False)

    nomenclature_id = fields.Many2one(
        'barcode.nomenclature',
        string="Nomenclature",
        default=_get_default_nomenclature,
    )

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
from . import barcodes
from . import barcode_events_mixin
from . import ir_http
from . import res_company

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_barcode_nomenclature_user,barcode.nomenclature.user,model_barcode_nomenclature,base.group_user,1,0,0,0
access_barcode_nomenclature_manager,barcode.nomenclature.manager,model_barcode_nomenclature,base.group_erp_manager,1,1,1,1
access_barcode_rule_user,barcode.rule.user,model_barcode_rule,base.group_user,1,0,0,0
access_barcode_rule_manager,barcode.rule.manager,model_barcode_rule,base.group_erp_manager,1,1,1,1

```

## File: static\src\js\barcode_events.js

```javascript
odoo.define('barcodes.BarcodeEvents', function(require) {
"use strict";

var core = require('web.core');
var mixins = require('web.mixins');
var session = require('web.session');


// For IE >= 9, use this, new CustomEvent(), instead of new Event()
function CustomEvent ( event, params ) {
    params = params || { bubbles: false, cancelable: false, detail: undefined };
    var evt = document.createEvent( 'CustomEvent' );
    evt.initCustomEvent( event, params.bubbles, params.cancelable, params.detail );
    return evt;
   }
CustomEvent.prototype = window.Event.prototype;

var BarcodeEvents = core.Class.extend(mixins.PropertiesMixin, {
    timeout: null,
    key_pressed: {},
    buffered_key_events: [],
    // Regexp to match a barcode input and extract its payload
    // Note: to build in init() if prefix/suffix can be configured
    regexp: /(.{3,})[\n\r\t]*/,
    // By knowing the terminal character we can interpret buffered keys
    // as a barcode as soon as it's encountered (instead of waiting x ms)
    suffix: /[\n\r\t]+/,
    // Keys from a barcode scanner are usually processed as quick as possible,
    // but some scanners can use an intercharacter delay (we support <= 50 ms)
    max_time_between_keys_in_ms: session.max_time_between_keys_in_ms || 100,
    // To be able to receive the barcode value, an input must be focused.
    // On mobile devices, this causes the virtual keyboard to open.
    // Unfortunately it is not possible to avoid this behavior...
    // To avoid keyboard flickering at each detection of a barcode value,
    // we want to keep it open for a while (800 ms).
    inputTimeOut: 800,

    init: function() {
        mixins.PropertiesMixin.init.call(this);
        // Keep a reference of the handler functions to use when adding and removing event listeners
        this.__keydown_handler = _.bind(this.keydown_handler, this);
        this.__keyup_handler = _.bind(this.keyup_handler, this);
        this.__handler = _.bind(this.handler, this);
        // Bind event handler once the DOM is loaded
        // TODO: find a way to be active only when there are listeners on the bus
        $(_.bind(this.start, this, false));

        // Mobile device detection
        var isMobile = navigator.userAgent.match(/Android/i) ||
                       navigator.userAgent.match(/webOS/i) ||
                       navigator.userAgent.match(/iPhone/i) ||
                       navigator.userAgent.match(/iPad/i) ||
                       navigator.userAgent.match(/iPod/i) ||
                       navigator.userAgent.match(/BlackBerry/i) ||
                       navigator.userAgent.match(/Windows Phone/i);
        this.isChromeMobile = isMobile && navigator.userAgent.match(/Chrome/i);

        // Creates an input who will receive the barcode scanner value.
        this.$barcodeInput = $('<input/>', {
            name: 'barcode',
            type: 'text',
            css: {
                'position': 'fixed',
                'top': '50%',
                'transform': 'translateY(-50%)',
                'z-index': '-1',
                'opacity': '0',
            },
        });
        // Avoid to show autocomplete for a non appearing input
        this.$barcodeInput.attr('autocomplete', 'off');

        this.__blurBarcodeInput = _.debounce(this._blurBarcodeInput, this.inputTimeOut);
    },

    handle_buffered_keys: function() {
        var str = this.buffered_key_events.reduce(function(memo, e) { return memo + String.fromCharCode(e.which) }, '');
        var match = str.match(this.regexp);

        if (match) {
            var barcode = match[1];

            // Send the target in case there are several barcode widgets on the same page (e.g.
            // registering the lot numbers in a stock picking)
            core.bus.trigger('barcode_scanned', barcode, this.buffered_key_events[0].target);

            // Dispatch a barcode_scanned DOM event to elements that have barcode_events="true" set.
            if (this.buffered_key_events[0].target.getAttribute("barcode_events") === "true")
                $(this.buffered_key_events[0].target).trigger('barcode_scanned', barcode);
        } else {
            this.resend_buffered_keys();
        }

        this.buffered_key_events = [];
    },

    resend_buffered_keys: function() {
        var old_event, new_event;
        for(var i = 0; i < this.buffered_key_events.length; i++) {
            old_event = this.buffered_key_events[i];

            if(old_event.which !== 13) { // ignore returns
                // We do not create a 'real' keypress event through
                // eg. KeyboardEvent because there are several issues
                // with them that make them very different from
                // genuine keypresses. Chrome per example has had a
                // bug for the longest time that causes keyCode and
                // charCode to not be set for events created this way:
                // https://bugs.webkit.org/show_bug.cgi?id=16735
                var params = {
                    'bubbles': old_event.bubbles,
                    'cancelable': old_event.cancelable,
                };
                new_event = $.Event('keypress', params);
                new_event.viewArg = old_event.viewArg;
                new_event.ctrl = old_event.ctrl;
                new_event.alt = old_event.alt;
                new_event.shift = old_event.shift;
                new_event.meta = old_event.meta;
                new_event.char = old_event.char;
                new_event.key = old_event.key;
                new_event.charCode = old_event.charCode;
                new_event.keyCode = old_event.keyCode || old_event.which; // Firefox doesn't set keyCode for keypresses, only keyup/down
                new_event.which = old_event.which;
                new_event.dispatched_by_barcode_reader = true;

                $(old_event.target).trigger(new_event);
            }
        }
    },

    element_is_editable: function(element) {
        return $(element).is('input,textarea,[contenteditable="true"]');
    },

    // This checks that a keypress event is either ESC, TAB, an arrow
    // key or a function key. This is Firefox specific, in Chrom{e,ium}
    // keypress events are not fired for these types of keys, only
    // keyup/keydown.
    is_special_key: function(e) {
        if (e.key === "ArrowLeft" || e.key === "ArrowRight" ||
            e.key === "ArrowUp" || e.key === "ArrowDown" ||
            e.key === "Escape" || e.key === "Tab" ||
            e.key === "Backspace" || e.key === "Delete" ||
            e.key === "Home" || e.key === "End" ||
            e.key === "PageUp" || e.key === "PageDown" ||
            e.key === "Unidentified" || /F\d\d?/.test(e.key)) {
            return true;
        } else {
            return false;
        }
    },

    // The keydown and keyup handlers are here to disallow key
    // repeat. When preventDefault() is called on a keydown event
    // the keypress that normally follows is cancelled.
    keydown_handler: function(e){
        if (this.key_pressed[e.which]) {
            e.preventDefault();
        } else {
            this.key_pressed[e.which] = true;
        }
    },

    keyup_handler: function(e){
        this.key_pressed[e.which] = false;
    },

    handler: function(e){
        // Don't catch events we resent
        if (e.dispatched_by_barcode_reader)
            return;
        // Don't catch non-printable keys for which Firefox triggers a keypress
        if (this.is_special_key(e))
            return;
        // Don't catch keypresses which could have a UX purpose (like shortcuts)
        if (e.ctrlKey || e.metaKey || e.altKey)
            return;
        // Don't catch Return when nothing is buffered. This way users
        // can still use Return to 'click' on focused buttons or links.
        if (e.which === 13 && this.buffered_key_events.length === 0)
            return;
        // Don't catch events targeting elements that are editable because we
        // have no way of redispatching 'genuine' key events. Resent events
        // don't trigger native event handlers of elements. So this means that
        // our fake events will not appear in eg. an <input> element.
        if ((this.element_is_editable(e.target) && !$(e.target).data('enableBarcode')) && e.target.getAttribute("barcode_events") !== "true")
            return;

        // Catch and buffer the event
        this.buffered_key_events.push(e);
        e.preventDefault();
        e.stopImmediatePropagation();

        // Handle buffered keys immediately if the the keypress marks the end
        // of a barcode or after x milliseconds without a new keypress
        clearTimeout(this.timeout);
        if (String.fromCharCode(e.which).match(this.suffix)) {
            this.handle_buffered_keys();
        } else {
            this.timeout = setTimeout(_.bind(this.handle_buffered_keys, this), this.max_time_between_keys_in_ms);
        }
    },

    /**
     * Try to detect the barcode value by listening all keydown events:
     * Checks if a dom element who may contains text value has the focus.
     * If not, it's probably because these events are triggered by a barcode scanner.
     * To be able to handle this value, a focused input will be created.
     *
     * This function also has the responsibility to detect the end of the barcode value.
     * (1) In most of cases, an optional key (tab or enter) is sent to mark the end of the value.
     * So, we direclty handle the value.
     * (2) If no end key is configured, we have to calculate the delay between each keydowns.
     * 'max_time_between_keys_in_ms' depends of the device and may be configured.
     * Exceeded this timeout, we consider that the barcode value is entirely sent.
     *
     * @private
     * @param  {jQuery.Event} e keydown event
     */
    _listenBarcodeScanner: function (e) {
        if ($(document.activeElement).not('input:text, textarea, [contenteditable], ' +
            '[type="email"], [type="number"], [type="password"], [type="tel"], [type="search"]').length) {
            $('body').append(this.$barcodeInput);
            this.$barcodeInput.focus();
        }
        if (this.$barcodeInput.is(":focus")) {
            // Handle buffered keys immediately if the keypress marks the end
            // of a barcode or after x milliseconds without a new keypress.
            clearTimeout(this.timeout);
            // On chrome mobile, e.which only works for some special characters like ENTER or TAB.
            if (String.fromCharCode(e.which).match(this.suffix)) {
                this._handleBarcodeValue(e);
            } else {
                this.timeout = setTimeout(this._handleBarcodeValue.bind(this, e),
                    this.max_time_between_keys_in_ms);
            }
            // if the barcode input doesn't receive keydown for a while, remove it.
            this.__blurBarcodeInput();
        }
    },

    /**
     * Retrieves the barcode value from the temporary input element.
     * This checks this value and trigger it on the bus.
     *
     * @private
     * @param  {jQuery.Event} keydown event
     */
    _handleBarcodeValue: function (e) {
        var barcodeValue = this.$barcodeInput.val();
        if (barcodeValue.match(this.regexp)) {
            core.bus.trigger('barcode_scanned', barcodeValue, $(e.target).parent()[0]);
            this._blurBarcodeInput();
        }
    },

    /**
     * Removes the value and focus from the barcode input.
     * If nothing happens, the focus will be lost and
     * the virtual keyboard on mobile devices will be closed.
     *
     * @private
     */
    _blurBarcodeInput: function () {
        // Close the virtual keyboard on mobile browsers
        // FIXME: actually we can't prevent keyboard from opening
        this.$barcodeInput.val('').blur();
    },

    start: function(prevent_key_repeat){
        // Chrome Mobile isn't triggering keypress event.
        // This is marked as Legacy in the DOM-Level-3 Standard.
        // See: https://www.w3.org/TR/uievents/#legacy-keyboardevent-event-types
        // This fix is only applied for Google Chrome Mobile but it should work for
        // all other cases.
        // In master, we could remove the behavior with keypress and only use keydown.
        if (this.isChromeMobile) {
            $('body').on("keydown", this._listenBarcodeScanner.bind(this));
        } else {
            $('body').bind("keypress", this.__handler);
        }
        if (prevent_key_repeat === true) {
            $('body').bind("keydown", this.__keydown_handler);
            $('body').bind('keyup', this.__keyup_handler);
        }
    },

    stop: function(){
        $('body').off("keypress", this.__handler);
        $('body').off("keydown", this.__keydown_handler);
        $('body').off('keyup', this.__keyup_handler);
    },
});

return {
    /** Singleton that emits barcode_scanned events on core.bus */
    BarcodeEvents: new BarcodeEvents(),
    /**
     * List of barcode prefixes that are reserved for internal purposes
     * @type Array
     */
    ReservedBarcodePrefixes: ['O-CMD'],
};

});

```

## File: static\src\js\barcode_field.js

```javascript
odoo.define('barcodes.field', function(require) {
"use strict";

var AbstractField = require('web.AbstractField');
var basicFields = require('web.basic_fields');
var fieldRegistry = require('web.field_registry');
var BarcodeEvents = require('barcodes.BarcodeEvents').BarcodeEvents;

// Field in which the user can both type normally and scan barcodes

var FieldFloatScannable = basicFields.FieldFloat.extend({
    events: _.extend({}, basicFields.FieldFloat.prototype.events, {
        // The barcode_events component intercepts keypresses and releases them when it
        // appears they are not part of a barcode. But since released keypresses don't
        // trigger native behaviour (like characters input), we must simulate it.
        keypress: '_onKeypress',
    }),

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     * @private
     */
    _renderEdit: function () {
        var self = this;
        return Promise.resolve(this._super()).then(function () {
            self.$input.data('enableBarcode', true);
        });
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {KeyboardEvent} e
     */
    _onKeypress: function (e) {
        /* only simulate a keypress if it has been previously prevented */
        if (e.dispatched_by_barcode_reader !== true) {
            if (!BarcodeEvents.is_special_key(e)) {
                e.preventDefault();
            }
            return;
        }
        var character = String.fromCharCode(e.which);
        var current_str = e.target.value;
        var str_before_carret = current_str.substring(0, e.target.selectionStart);
        var str_after_carret = current_str.substring(e.target.selectionEnd);
        e.target.value = str_before_carret + character + str_after_carret;
        var new_carret_index = str_before_carret.length + character.length;
        e.target.setSelectionRange(new_carret_index, new_carret_index);
        // trigger an 'input' event to notify the widget that it's value changed
        $(e.target).trigger('input');
    },
});

// Field to use scan barcodes
var FormViewBarcodeHandler = AbstractField.extend({
    /**
     * @override
     */
    init: function() {
        this._super.apply(this, arguments);

        this.trigger_up('activeBarcode', {
            name: this.name,
            commands: {
                barcode: '_barcodeAddX2MQuantity',
            }
        });
    },
});

fieldRegistry.add('field_float_scannable', FieldFloatScannable);
fieldRegistry.add('barcode_handler', FormViewBarcodeHandler);

return {
    FieldFloatScannable: FieldFloatScannable,
    FormViewBarcodeHandler: FormViewBarcodeHandler,
};

});

```

## File: static\src\js\barcode_form_view.js

```javascript
odoo.define('barcodes.FormView', function (require) {
"use strict";

var BarcodeEvents = require('barcodes.BarcodeEvents'); // handle to trigger barcode on bus
var concurrency = require('web.concurrency');
var core = require('web.core');
var Dialog = require('web.Dialog');
var FormController = require('web.FormController');
var FormRenderer = require('web.FormRenderer');

var _t = core._t;


FormController.include({
    custom_events: _.extend({}, FormController.prototype.custom_events, {
        activeBarcode: '_barcodeActivated',
    }),

    /**
     * add default barcode commands for from view
     *
     * @override
     */
    init: function () {
        this._super.apply(this, arguments);
        this.activeBarcode = {
            form_view: {
                commands: {
                    'O-CMD.EDIT': this._barcodeEdit.bind(this),
                    'O-CMD.DISCARD': this._barcodeDiscard.bind(this),
                    'O-CMD.SAVE': this._barcodeSave.bind(this),
                    'O-CMD.PREV': this._barcodePagerPrevious.bind(this),
                    'O-CMD.NEXT': this._barcodePagerNext.bind(this),
                    'O-CMD.PAGER-FIRST': this._barcodePagerFirst.bind(this),
                    'O-CMD.PAGER-LAST': this._barcodePagerLast.bind(this),
                },
            },
        };

        this.barcodeMutex = new concurrency.Mutex();
        this._barcodeStartListening();
    },
    /**
     * @override
     */
    destroy: function () {
        this._barcodeStopListening();
        this._super();
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {string} barcode sent by the scanner (string generate from keypress series)
     * @param {Object} activeBarcode: options sent by the field who use barcode features
     * @returns {Promise}
     */
    _barcodeAddX2MQuantity: function (barcode, activeBarcode) {
        if (this.mode === 'readonly') {
            this.do_warn(_t('Error: Document not editable'),
                _t('To modify this document, please first start edition.'));
            return Promise.reject();
        }

        var record = this.model.get(this.handle);
        var candidate = this._getBarCodeRecord(record, barcode, activeBarcode);
        if (candidate) {
            return this._barcodeSelectedCandidate(candidate, record, barcode, activeBarcode);
        } else {
            return this._barcodeWithoutCandidate(record, barcode, activeBarcode);
        }
    },
    /**
     * @private
     */
    _barcodeDiscard: function () {
        return this.discardChanges();
    },
    /**
     * @private
     */
    _barcodeEdit: function () {
        return this._setMode('edit');
    },
    /**
     * @private
     */
    _barcodePagerFirst: function () {
        var self = this;
        return this.mutex.exec(function () {}).then(function () {
            if (!self.pager) {
                self.do_warn(_t('Error: Pager not available'));
                return;
            }
            self.pager.updateState({
                current_min: 1,
            }, {notifyChange: true});
        });
    },
    /**
     * @private
     */
    _barcodePagerLast: function () {
        var self = this;
        return this.mutex.exec(function () {}).then(function () {
            if (!self.pager) {
                self.do_warn(_t('Error: Pager not available'));
                return;
            }
            var state = self.model.get(self.handle, {raw: true});
            self.pager.updateState({
                current_min: state.count,
            }, {notifyChange: true});
        });
    },
    /**
     * @private
     */
    _barcodePagerNext: function () {
        var self = this;
        return this.mutex.exec(function () {}).then(function () {
            if (!self.pager) {
                self.do_warn(_t('Error: Pager not available'));
                return;
            }
            self.pager.next();
        });
    },
    /**
     * @private
     */
    _barcodePagerPrevious: function () {
        var self = this;
        return this.mutex.exec(function () {}).then(function () {
            if (!self.pager) {
                self.do_warn(_t('Error: Pager not available'));
                return;
            }
            self.pager.previous();
        });
    },
    /**
     * Returns true iff the given barcode matches the given record (candidate).
     *
     * @private
     * @param {Object} candidate: record in the x2m
     * @param {string} barcode sent by the scanner (string generate from keypress series)
     * @param {Object} activeBarcode: options sent by the field who use barcode features
     * @returns {boolean}
     */
    _barcodeRecordFilter: function (candidate, barcode, activeBarcode) {
        return candidate.data.product_barcode === barcode;
    },
    /**
     * @private
     */
    _barcodeSave: function () {
        return this.saveRecord();
    },
    /**
     * @private
     * @param {Object} candidate: record in the x2m
     * @param {Object} current record
     * @param {string} barcode sent by the scanner (string generate from keypress series)
     * @param {Object} activeBarcode: options sent by the field who use barcode features
     * @returns {Promise}
     */
    _barcodeSelectedCandidate: function (candidate, record, barcode, activeBarcode, quantity) {
        var changes = {};
        var candidateChanges = {};
        candidateChanges[activeBarcode.quantity] = quantity ? quantity : candidate.data[activeBarcode.quantity] + 1;
        changes[activeBarcode.fieldName] = {
            operation: 'UPDATE',
            id: candidate.id,
            data: candidateChanges,
        };
        return this.model.notifyChanges(this.handle, changes, {notifyChange: activeBarcode.notifyChange});
    },
    /**
     * @private
     */
    _barcodeStartListening: function () {
        core.bus.on('barcode_scanned', this, this._barcodeScanned);
        core.bus.on('keypress', this, this._quantityListener);
    },
    /**
     * @private
     */
    _barcodeStopListening: function () {
        core.bus.off('barcode_scanned', this, this._barcodeScanned);
        core.bus.off('keypress', this, this._quantityListener);
    },
    /**
     * @private
     * @param {Object} current record
     * @param {string} barcode sent by the scanner (string generate from keypress series)
     * @param {Object} activeBarcode: options sent by the field who use barcode features
     * @returns {Promise}
     */
    _barcodeWithoutCandidate: function (record, barcode, activeBarcode) {
        var changes = {};
        changes[activeBarcode.name] = barcode;
        return this.model.notifyChanges(record.id, changes);
    },
    /**
     * @private
     * @param {Object} current record
     * @param {string} barcode sent by the scanner (string generate from keypress series)
     * @param {Object} activeBarcode: options sent by the field who use barcode features
     * @returns {Object|undefined}
     */
    _getBarCodeRecord: function (record, barcode, activeBarcode) {
        var self = this;
        if (!activeBarcode.fieldName || !record.data[activeBarcode.fieldName]) {
            return;
        }
        return _.find(record.data[activeBarcode.fieldName].data, function (record) {
            return self._barcodeRecordFilter(record, barcode, activeBarcode);
        });
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * The barcode is activate when at least one widget trigger_up 'activeBarcode' event
     * with the widget option
     *
     * @param {OdooEvent} event
     * @param {string} event.data.name: the current field name
     * @param {string} [event.data.fieldName] optional for x2many sub field
     * @param {boolean} [event.data.notifyChange] optional for x2many sub field
     *     do not trigger on change server side if a candidate has been found
     * @param {string} [event.data.quantity] optional field to increase quantity
     * @param {Object} [event.data.commands] optional added methods
     *     can use comand with specific barcode (with ReservedBarcodePrefixes)
     *     or change 'barcode' for all other received barcodes
     *     (e.g.: 'O-CMD.MAIN-MENU': function ..., barcode: function () {...})
     */
    _barcodeActivated: function (event) {
        event.stopPropagation();
        var name = event.data.name;
        this.activeBarcode[name] = {
            name: name,
            handle: this.handle,
            target: event.target,
            widget: event.target.attrs && event.target.attrs.widget,
            setQuantityWithKeypress: !! event.data.setQuantityWithKeypress,
            fieldName: event.data.fieldName,
            notifyChange: (event.data.notifyChange !== undefined) ? event.data.notifyChange : true,
            quantity: event.data.quantity,
            commands: event.data.commands || {},
            candidate: this.activeBarcode[name] && this.activeBarcode[name].handle === this.handle ?
                this.activeBarcode[name].candidate : null,
        };

        // we want to disable autofocus when activating the barcode to avoid
        // putting the scanned value in the focused field
        this.disableAutofocus = true;
    },
    /**
     * @private
     * @param {string|function} method defined by the commands options
     * @param {string} barcode sent by the scanner (string generate from keypress series)
     * @param {Object} activeBarcode: options sent by the field who use barcode features
     * @returns {Promise}
     */
    _barcodeActiveScanned: function (method, barcode, activeBarcode) {
        var self = this;
        var methodDef;
        var def = new Promise(function (resolve, reject) {
            if (typeof method === 'string') {
                methodDef = self[method](barcode, activeBarcode);
            } else {
                methodDef = method.call(self, barcode, activeBarcode);
            }
            methodDef
                .then(function () {
                    var record = self.model.get(self.handle);
                    var candidate = self._getBarCodeRecord(record, barcode, activeBarcode);
                    activeBarcode.candidate = candidate;
                })
                .then(resolve, resolve);
        });
        return def;
    },
    /**
     * Method called when a user scan a barcode, call each method in function of the
     * widget options then update the renderer
     *
     * @private
     * @param {string} barcode sent by the scanner (string generate from keypress series)
     * @param {DOM Object} target
     * @returns {Promise}
     */
    _barcodeScanned: function (barcode, target) {
        var self = this;
        return this.barcodeMutex.exec(function () {
            var prefixed = _.any(BarcodeEvents.ReservedBarcodePrefixes,
                    function (reserved) {return barcode.indexOf(reserved) === 0;});
            var hasCommand = false;
            var defs = [];
            if (! $.contains(target, self.el)) {
                return;
            }
            for (var k in self.activeBarcode) {
                var activeBarcode = self.activeBarcode[k];
                // Handle the case where there are several barcode widgets on the same page. Since the
                // event is global on the page, all barcode widgets will be triggered. However, we only
                // want to keep the event on the target widget.
                var methods = self.activeBarcode[k].commands;
                var method = prefixed ? methods[barcode] : methods.barcode;
                if (method) {
                    if (prefixed) {
                        hasCommand = true;
                    }
                    defs.push(self._barcodeActiveScanned(method, barcode, activeBarcode));
                }
            }
            if (prefixed && !hasCommand) {
                self.do_warn(_t('Error: Barcode command is undefined'), barcode);
            }
            return self.alive(Promise.all(defs)).then(function () {
                if (!prefixed) {
                    // remember the barcode scanned for the quantity listener
                    self.current_barcode = barcode;
                    // redraw the view if we scanned a real barcode (required if
                    // we manually apply the change in JS, e.g. incrementing the
                    // quantity)
                    self.update({}, {reload: false});
                }
            });
        });
    },
    /**
     * @private
     * @param {KeyEvent} event
     */
    _quantityListener: function (event) {
        var character = String.fromCharCode(event.which);

        if (! $.contains(event.target, this.el)) {
            return;
        }
        // only catch the event if we're not focused in
        // another field and it's a number
        if (!$(event.target).is('body, .modal') || !/[0-9]/.test(character)) {
            return;
        }

        var barcodeInfos = _.filter(this.activeBarcode, 'setQuantityWithKeypress');
        if (!barcodeInfos.length) {
            return;
        }

        if (!_.compact(_.pluck(barcodeInfos, 'candidate')).length) {
            return this.do_warn(_t('Error: No last scanned barcode'),
                _t('To set the quantity please scan a barcode first.'));
        }

        for (var k in this.activeBarcode) {
            if (this.activeBarcode[k].candidate) {
                this._quantityOpenDialog(character, this.activeBarcode[k]);
            }
        }
    },
    /**
     * @private
     * @param {string} character
     * @param {Object} activeBarcode: options sent by the field who use barcode features
     */
    _quantityOpenDialog: function (character, activeBarcode) {
        var self = this;
        var $content = $('<div>').append($('<input>', {type: 'text', class: 'o_set_qty_input'}));
        this.dialog = new Dialog(this, {
            title: _t('Set quantity'),
            buttons: [{text: _t('Select'), classes: 'btn-primary', close: true, click: function () {
                var new_qty = this.$content.find('.o_set_qty_input').val();
                var record = self.model.get(self.handle);
                return self._barcodeSelectedCandidate(activeBarcode.candidate, record,
                        self.current_barcode, activeBarcode, parseFloat(new_qty))
                .then(function () {
                    self.update({}, {reload: false});
                });
            }}, {text: _t('Discard'), close: true}],
            $content: $content,
        });
        this.dialog.opened().then(function () {
            // This line set the value of the key which triggered the _set_quantity in the input
            var $input = self.dialog.$('.o_set_qty_input').focus().val(character);
            var $selectBtn = self.dialog.$footer.find('.btn-primary');
            $input.on('keypress', function (event){
                if (event.which === 13) {
                    event.preventDefault();
                    $input.off();
                    $selectBtn.click();
                }
            });
        });
        this.dialog.open();
    },
});


FormRenderer.include({

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------
    /**
     * trigger_up 'activeBarcode' to Add barcode event handler
     *
     * @private
     * @param {jQueryElement} $button
     * @param {Object} node
     */
    _barcodeButtonHandler: function ($button, node) {
        var commands = {};
        commands.barcode = function () {return Promise.resolve();};
        commands['O-BTN.' + node.attrs.barcode_trigger] = function () {
            if (!$button.hasClass('o_invisible_modifier')) {
                $button.click();
            }
            return Promise.resolve();
        };
        var name = node.attrs.name;
        if (node.attrs.string) {
            name = name + '_' + node.attrs.string;
        }

        this.trigger_up('activeBarcode', {
            name: name,
            commands: commands
        });
    },
    /**
     * Add barcode event handler
     *
     * @override
     * @private
     * @param {Object} node
     * @returns {jQueryElement}
     */
    _renderHeaderButton: function (node) {
        var $button = this._super.apply(this, arguments);
        if (node.attrs.barcode_trigger) {
            this._barcodeButtonHandler($button, node);
        }
        return $button;
    },
    /**
     * Add barcode event handler
     *
     * @override
     * @private
     * @param {Object} node
     * @returns {jQueryElement}
     */
    _renderStatButton: function (node) {
        var $button = this._super.apply(this, arguments);
        if (node.attrs.barcode_trigger) {
            this._barcodeButtonHandler($button, node);
        }
        return $button;
    },
    /**
     * Add barcode event handler
     *
     * @override
     * @private
     * @param {Object} node
     * @returns {jQueryElement}
     */
    _renderTagButton: function (node) {
        var $button = this._super.apply(this, arguments);
        if (node.attrs.barcode_trigger) {
            this._barcodeButtonHandler($button, node);
        }
        return $button;
    }
});

BarcodeEvents.ReservedBarcodePrefixes.push('O-BTN');

});

```

## File: static\src\js\barcode_parser.js

```javascript
odoo.define('barcodes.BarcodeParser', function (require) {
"use strict";

var Class = require('web.Class');
var rpc = require('web.rpc');

// The BarcodeParser is used to detect what is the category
// of a barcode (product, partner, ...) and extract an encoded value
// (like weight, price, etc.)
var BarcodeParser = Class.extend({
    init: function(attributes) {
        this.nomenclature_id = attributes.nomenclature_id;
        this.loaded = this.load();
    },

    // This loads the barcode nomenclature and barcode rules which are
    // necessary to parse the barcodes. The BarcodeParser is operational
    // only when those data have been loaded
    load: function(){
        var self = this;
        if (!this.nomenclature_id) {
            return;
        }
        var id = this.nomenclature_id[0];
        return rpc.query({
                model: 'barcode.nomenclature',
                method: 'read',
                args: [[id], ['name','rule_ids','upc_ean_conv']],
            })
            .then(function (nomenclatures){
                self.nomenclature = nomenclatures[0];

                var args = [
                    [['barcode_nomenclature_id', '=', self.nomenclature.id]],
                    ['name', 'sequence', 'type', 'encoding', 'pattern', 'alias'],
                ];
                return rpc.query({
                    model: 'barcode.rule',
                    method: 'search_read',
                    args: args,
                });
            }).then(function(rules){
                rules = rules.sort(function(a, b){ return a.sequence - b.sequence; });
                self.nomenclature.rules = rules;
            });
    },

    // resolves when the barcode parser is operational.
    is_loaded: function() {
        return this.loaded;
    },

    // returns the checksum of the ean13, or -1 if the ean has not the correct length, ean must be a string
    ean_checksum: function(ean){
        var code = ean.split('');
        if(code.length !== 13){
            return -1;
        }
        var oddsum = 0, evensum = 0, total = 0;
        code = code.reverse().splice(1);
        for(var i = 0; i < code.length; i++){
            if(i % 2 === 0){
                oddsum += Number(code[i]);
            }else{
                evensum += Number(code[i]);
            }
        }
        total = oddsum * 3 + evensum;
        return Number((10 - total % 10) % 10);
    },

    // returns the checksum of the ean8, or -1 if the ean has not the correct length, ean must be a string
    ean8_checksum: function(ean){
        var code = ean.split('');
        if (code.length !== 8) {
            return -1;
        }
        var sum1  = Number(code[1]) + Number(code[3]) + Number(code[5]);
        var sum2  = Number(code[0]) + Number(code[2]) + Number(code[4]) + Number(code[6]);
        var total = sum1 + 3 * sum2;
        return Number((10 - total % 10) % 10);
    },


    // returns true if the ean is a valid EAN barcode number by checking the control digit.
    // ean must be a string
    check_ean: function(ean){
        return /^\d+$/.test(ean) && this.ean_checksum(ean) === Number(ean[ean.length-1]);
    },

    // returns true if the barcode string is encoded with the provided encoding.
    check_encoding: function(barcode, encoding) {
        var len    = barcode.length;
        var allnum = /^\d+$/.test(barcode);
        var check  = Number(barcode[len-1]);

        if (encoding === 'ean13') {
            return len === 13 && allnum && this.ean_checksum(barcode) === check;
        } else if (encoding === 'ean8') {
            return len === 8  && allnum && this.ean8_checksum(barcode) === check;
        } else if (encoding === 'upca') {
            return len === 12 && allnum && this.ean_checksum('0'+barcode) === check;
        } else if (encoding === 'any') {
            return true;
        } else {
            return false;
        }
    },

    // returns a valid zero padded ean13 from an ean prefix. the ean prefix must be a string.
    sanitize_ean: function(ean){
        ean = ean.substr(0,13);

        for(var n = 0, count = (13 - ean.length); n < count; n++){
            ean = '0' + ean;
        }
        return ean.substr(0,12) + this.ean_checksum(ean);
    },

    // Returns a valid zero padded UPC-A from a UPC-A prefix. the UPC-A prefix must be a string.
    sanitize_upc: function(upc) {
        return this.sanitize_ean('0'+upc).substr(1,12);
    },

    // Checks if barcode matches the pattern
    // Additionnaly retrieves the optional numerical content in barcode
    // Returns an object containing:
    // - value: the numerical value encoded in the barcode (0 if no value encoded)
    // - base_code: the barcode in which numerical content is replaced by 0's
    // - match: boolean
    match_pattern: function (barcode, pattern, encoding){
        var match = {
            value: 0,
            base_code: barcode,
            match: false,
        };
        barcode = barcode.replace("\\", "\\\\").replace("{", '\{').replace("}", "\}").replace(".", "\.");

        var numerical_content = pattern.match(/[{][N]*[D]*[}]/); // look for numerical content in pattern
        var base_pattern = pattern;
        if(numerical_content){ // the pattern encodes a numerical content
            var num_start = numerical_content.index; // start index of numerical content
            var num_length = numerical_content[0].length; // length of numerical content
            var value_string = barcode.substr(num_start, num_length-2); // numerical content in barcode
            var whole_part_match = numerical_content[0].match("[{][N]*[D}]"); // looks for whole part of numerical content
            var decimal_part_match = numerical_content[0].match("[{N][D]*[}]"); // looks for decimal part
            var whole_part = value_string.substr(0, whole_part_match.index+whole_part_match[0].length-2); // retrieve whole part of numerical content in barcode
            var decimal_part = "0." + value_string.substr(decimal_part_match.index, decimal_part_match[0].length-1); // retrieve decimal part
            if (whole_part === ''){
                whole_part = '0';
            }
            match['value'] = parseInt(whole_part) + parseFloat(decimal_part);

            // replace numerical content by 0's in barcode and pattern
            match['base_code'] = barcode.substr(0,num_start);
            var base_pattern = pattern.substr(0,num_start);
            for(var i=0;i<(num_length-2);i++) {
                match['base_code'] += "0";
                base_pattern += "0";
            }
            match['base_code'] += barcode.substr(num_start+num_length-2,barcode.length-1);
            base_pattern += pattern.substr(num_start+num_length,pattern.length-1);

            match['base_code'] = match['base_code']
                .replace("\\\\", "\\")
                .replace("\{", "{")
                .replace("\}","}")
                .replace("\.",".");

            var base_code = match.base_code.split('')
            if (encoding === 'ean13') {
                base_code[12] = '' + this.ean_checksum(match.base_code);
            } else if (encoding === 'ean8') {
                base_code[7]  = '' + this.ean8_checksum(match.base_code);
            } else if (encoding === 'upca') {
                base_code[11] = '' + this.ean_checksum('0' + match.base_code);
            }
            match.base_code = base_code.join('')
        }

        if (base_pattern[0] !== '^') {
            base_pattern = "^" + base_pattern;
        }
        match.match = match.base_code.match(base_pattern);

        return match;
    },

    // attempts to interpret a barcode (string encoding a barcode Code-128)
    // it will return an object containing various information about the barcode.
    // most importantly :
    // - code    : the barcode
    // - type   : the type of the barcode (e.g. alias, unit product, weighted product...)
    //
    // - value  : if the barcode encodes a numerical value, it will be put there
    // - base_code : the barcode with all the encoding parts set to zero; the one put on
    //               the product in the backend
    parse_barcode: function(barcode){
        var parsed_result = {
            encoding: '',
            type:'error',
            code:barcode,
            base_code: barcode,
            value: 0,
        };

        if (!this.nomenclature) {
            return parsed_result;
        }

        var rules = this.nomenclature.rules;
        for (var i = 0; i < rules.length; i++) {
            var rule = rules[i];
            var cur_barcode = barcode;

            if (    rule.encoding === 'ean13' &&
                    this.check_encoding(barcode,'upca') &&
                    this.nomenclature.upc_ean_conv in {'upc2ean':'','always':''} ){
                cur_barcode = '0' + cur_barcode;
            } else if (rule.encoding === 'upca' &&
                    this.check_encoding(barcode,'ean13') &&
                    barcode[0] === '0' &&
                    this.upc_ean_conv in {'ean2upc':'','always':''} ){
                cur_barcode = cur_barcode.substr(1,12);
            }

            if (!this.check_encoding(cur_barcode,rule.encoding)) {
                continue;
            }

            var match = this.match_pattern(cur_barcode, rules[i].pattern, rule.encoding);
            if (match.match) {
                if(rules[i].type === 'alias') {
                    barcode = rules[i].alias;
                    parsed_result.code = barcode;
                    parsed_result.type = 'alias';
                }
                else {
                    parsed_result.encoding  = rules[i].encoding;
                    parsed_result.type      = rules[i].type;
                    parsed_result.value     = match.value;
                    parsed_result.code      = cur_barcode;
                    if (rules[i].encoding === "ean13"){
                        parsed_result.base_code = this.sanitize_ean(match.base_code);
                    }
                    else{
                        parsed_result.base_code = match.base_code;
                    }
                    return parsed_result;
                }
            }
        }
        return parsed_result;
    },
});

return BarcodeParser;
});

```

## File: views\barcodes_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

       <template id="assets_backend" name="barcodes assets" inherit_id="web.assets_backend">
            <xpath expr="." position="inside">
                <script type="text/javascript" src="/barcodes/static/src/js/barcode_parser.js"></script>
                <script type="text/javascript" src="/barcodes/static/src/js/barcode_events.js"></script>
                <script type="text/javascript" src="/barcodes/static/src/js/barcode_form_view.js"></script>
                <script type="text/javascript" src="/barcodes/static/src/js/barcode_field.js"></script>
            </xpath>
        </template>

        <template id="qunit_suite" name="barcode_tests" inherit_id="web.qunit_suite">
            <xpath expr="//t[@t-set='head']" position="inside">
                <script type="text/javascript" src="/barcodes/static/tests/barcode_tests.js"/>
            </xpath>
        </template>

        <template id="qunit_mobile_suite" name="barcode_mobile_tests" inherit_id="web.qunit_mobile_suite">
            <xpath expr="//t[@t-set='head']" position="inside">
                <script type="text/javascript" src="/barcodes/static/tests/barcode_mobile_tests.js"></script>
            </xpath>
        </template>

</odoo>

```

## File: views\barcodes_view.xml

```xml
<?xml version="1.0"?>
<odoo>
        <!--     BARCODE NOMENCLATURES     -->
        <record model="ir.ui.view" id="view_barcode_nomenclature_form">
            <field name="name">Barcode Nomenclatures</field>
            <field name="model">barcode.nomenclature</field>
            <field name="arch" type="xml">
                <form string="Barcode Nomenclature">
                    <sheet>
                        <group col="4">
                            <field name="name" />
                            <field name="upc_ean_conv"/>
                        </group>
                        <div>
                            <p>
                                <i>Barcodes Nomenclatures</i> define how barcodes are recognized and categorized.
                                When a barcode is scanned it is associated to the <i>first</i> rule with a matching
                                pattern. The pattern syntax is that of regular expression, and a barcode is matched
                                if the regular expression matches a prefix of the barcode. 
                            </p><p>
                                Patterns can also define how numerical values, such as weight or price, can be
                                encoded into the barcode. They are indicated by <code>{NNN}</code> where the N's
                                define where the number's digits are encoded. Floats are also supported with the 
                                decimals indicated with D's, such as <code>{NNNDD}</code>. In these cases, 
                                the barcode field on the associated records <i>must</i> show these digits as 
                                zeroes. 
                            </p>
                        </div>
                        <field name="rule_ids">
                            <tree string='Tables'>
                                <field name="sequence" widget="handle"/>
                                <field name="name" />
                                <field name="type" />
                                <field name="encoding" />
                                <field name="pattern" />
                            </tree>
                        </field>
                    </sheet>
                </form>
            </field>
        </record>

        <record model="ir.ui.view" id="view_barcode_nomenclature_tree">
            <field name="name">Barcode Nomenclatures</field>
            <field name="model">barcode.nomenclature</field>
            <field name="arch" type="xml">
                <tree string="Barcode Nomenclatures">
                    <field name="name" />
                </tree>
            </field>
        </record>

        <record model="ir.actions.act_window" id="action_barcode_nomenclature_form">
            <field name="name">Barcode Nomenclatures</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">barcode.nomenclature</field>
            <field name="view_mode">tree,kanban,form</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Add a new barcode nomenclature
              </p><p>
                A barcode nomenclature defines how the point of sale identify and interprets barcodes
              </p>
            </field>
        </record>

        <record model="ir.ui.view" id="view_barcode_rule_form">
            <field name="name">Barcode Rule</field>
            <field name="model">barcode.rule</field>
            <field name="arch" type="xml">
                <form string="Barcode Rule">
                    <group col="4">
                        <field name="name" />
                        <field name="sequence" />
                        <field name="type"/>  
                        <field name="encoding" attrs="{'invisible': [('type','=', 'alias')]}"/> 
                        <field name="pattern" />
                        <field name="alias" attrs="{'invisible': [('type','!=', 'alias')]}"/>   
                    </group>
                </form>
            </field>
        </record>
</odoo>

```

