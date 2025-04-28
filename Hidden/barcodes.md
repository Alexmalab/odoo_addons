# Odoo Module: barcodes

Category: Hidden

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
    'category': 'Hidden',
    'summary': 'Scan and Parse Barcodes',
    'depends': ['web'],
    'data': [
        'data/barcodes_data.xml',
        'views/barcodes_view.xml',
        'security/ir.model.access.csv',
        ],
    'installable': True,
    'post_init_hook': '_assign_default_nomeclature_id',
    'assets': {
        'web.assets_backend': [
            'barcodes/static/src/**/*',
        ],
        'web.tests_assets': ['barcodes/static/tests/helpers.js'],
        'web.qunit_suite_tests': ['barcodes/static/tests/basic/**/*.js'],
        'web.qunit_mobile_suite_tests': ['barcodes/static/tests/mobile/*.js'],
    },
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

## File: models\barcode_nomenclature.py

```python
import re

from odoo import models, fields, api
from odoo.tools import check_barcode_encoding, get_barcode_check_digit


UPC_EAN_CONVERSIONS = [
    ('none', 'Never'),
    ('ean2upc', 'EAN-13 to UPC-A'),
    ('upc2ean', 'UPC-A to EAN-13'),
    ('always', 'Always'),
]


class BarcodeNomenclature(models.Model):
    _name = 'barcode.nomenclature'
    _description = 'Barcode Nomenclature'

    name = fields.Char(string='Barcode Nomenclature', size=32, required=True, help='An internal identification of the barcode nomenclature')
    rule_ids = fields.One2many('barcode.rule', 'barcode_nomenclature_id', string='Rules', help='The list of barcode rules')
    upc_ean_conv = fields.Selection(
        UPC_EAN_CONVERSIONS, string='UPC/EAN Conversion', required=True, default='always',
        help="UPC Codes can be converted to EAN by prefixing them with a zero. This setting determines if a UPC/EAN barcode should be automatically converted in one way or another when trying to match a rule with the other encoding.")

    @api.model
    def sanitize_ean(self, ean):
        """ Returns a valid zero padded EAN-13 from an EAN prefix.

        :type ean: str
        """
        ean = ean[0:13].zfill(13)
        return ean[0:-1] + str(get_barcode_check_digit(ean))

    @api.model
    def sanitize_upc(self, upc):
        """ Returns a valid zero padded UPC-A from a UPC-A prefix.

        :type upc: str
        """
        return self.sanitize_ean('0' + upc)[1:]

    def match_pattern(self, barcode, pattern):
        """Checks barcode matches the pattern and retrieves the optional numeric value in barcode.

        :param barcode:
        :type barcode: str
        :param pattern:
        :type pattern: str
        :return: an object containing:
            - value: the numerical value encoded in the barcode (0 if no value encoded)
            - base_code: the barcode in which numerical content is replaced by 0's
            - match: boolean
        :rtype: dict
        """
        match = {
            'value': 0,
            'base_code': barcode,
            'match': False,
        }

        barcode = barcode.replace('\\', '\\\\').replace('{', '\\{').replace('}', '\\}').replace('.', '\\.')
        numerical_content = re.search("[{][N]*[D]*[}]", pattern)  # look for numerical content in pattern

        if numerical_content:  # the pattern encodes a numerical content
            num_start = numerical_content.start()  # start index of numerical content
            num_end = numerical_content.end()  # end index of numerical content
            value_string = barcode[num_start:num_end - 2]  # numerical content in barcode

            whole_part_match = re.search("[{][N]*[D}]", numerical_content.group())  # looks for whole part of numerical content
            decimal_part_match = re.search("[{N][D]*[}]", numerical_content.group())  # looks for decimal part
            whole_part = value_string[:whole_part_match.end() - 2]  # retrieve whole part of numerical content in barcode
            decimal_part = "0." + value_string[decimal_part_match.start():decimal_part_match.end() - 1]  # retrieve decimal part
            if whole_part == '':
                whole_part = '0'
            if whole_part.isdigit():
                match['value'] = int(whole_part) + float(decimal_part)

                match['base_code'] = barcode[:num_start] + (num_end - num_start - 2) * "0" + barcode[num_end - 2:]  # replace numerical content by 0's in barcode
                match['base_code'] = match['base_code'].replace("\\\\", "\\").replace("\\{", "{").replace("\\}", "}").replace("\\.", ".")
                pattern = pattern[:num_start] + (num_end - num_start - 2) * "0" + pattern[num_end:]  # replace numerical content by 0's in pattern to match
        match['match'] = re.match(pattern, match['base_code'][:len(pattern)])

        return match

    def parse_barcode(self, barcode):
        """ Attempts to interpret and parse a barcode.

        :param barcode:
        :type barcode: str
        :return: A object containing various information about the barcode, like as:
            - code: the barcode
            - type: the barcode's type
            - value: if the id encodes a numerical value, it will be put there
            - base_code: the barcode code with all the encoding parts set to
              zero; the one put on the product in the backend
        :rtype: dict
        """
        parsed_result = {
            'encoding': '',
            'type': 'error',
            'code': barcode,
            'base_code': barcode,
            'value': 0,
        }

        for rule in self.rule_ids:
            cur_barcode = barcode
            if rule.encoding == 'ean13' and check_barcode_encoding(barcode, 'upca') and self.upc_ean_conv in ['upc2ean', 'always']:
                cur_barcode = '0' + cur_barcode
            elif rule.encoding == 'upca' and check_barcode_encoding(barcode, 'ean13') and barcode[0] == '0' and self.upc_ean_conv in ['ean2upc', 'always']:
                cur_barcode = cur_barcode[1:]

            if not check_barcode_encoding(barcode, rule.encoding):
                continue

            match = self.match_pattern(cur_barcode, rule.pattern)
            if match['match']:
                if rule.type == 'alias':
                    barcode = rule.alias
                    parsed_result['code'] = barcode
                else:
                    parsed_result['encoding'] = rule.encoding
                    parsed_result['type'] = rule.type
                    parsed_result['value'] = match['value']
                    parsed_result['code'] = cur_barcode
                    if rule.encoding == "ean13":
                        parsed_result['base_code'] = self.sanitize_ean(match['base_code'])
                    elif rule.encoding == "upca":
                        parsed_result['base_code'] = self.sanitize_upc(match['base_code'])
                    else:
                        parsed_result['base_code'] = match['base_code']
                    return parsed_result

        return parsed_result

```

## File: models\barcode_rule.py

```python
import re

from odoo import models, fields, api, _
from odoo.exceptions import ValidationError


class BarcodeRule(models.Model):
    _name = 'barcode.rule'
    _description = 'Barcode Rule'
    _order = 'sequence asc, id'

    name = fields.Char(string='Rule Name', size=32, required=True, help='An internal identification for this barcode nomenclature rule')
    barcode_nomenclature_id = fields.Many2one('barcode.nomenclature', string='Barcode Nomenclature')
    sequence = fields.Integer(string='Sequence', help='Used to order rules such that rules with a smaller sequence match first')
    encoding = fields.Selection(
        string='Encoding', required=True, default='any', selection=[
            ('any', 'Any'),
            ('ean13', 'EAN-13'),
            ('ean8', 'EAN-8'),
            ('upca', 'UPC-A'),
        ], help='This rule will apply only if the barcode is encoded with the specified encoding')
    type = fields.Selection(
        string='Type', required=True, selection=[
            ('alias', 'Alias'),
            ('product', 'Unit Product'),
        ], default='product')
    pattern = fields.Char(string='Barcode Pattern', help="The barcode matching pattern", required=True, default='.*')
    alias = fields.Char(string='Alias', default='0', help='The matched pattern will alias to this barcode', required=True)

    @api.constrains('pattern')
    def _check_pattern(self):
        for rule in self:
            p = rule.pattern.replace('\\\\', 'X').replace('\\{', 'X').replace('\\}', 'X')
            findall = re.findall("[{]|[}]", p)  # p does not contain escaped { or }
            if len(findall) == 2:
                if not re.search("[{][N]*[D]*[}]", p):
                    raise ValidationError(_("There is a syntax error in the barcode pattern %(pattern)s: braces can only contain N's followed by D's.", pattern=rule.pattern))
                elif re.search("[{][}]", p):
                    raise ValidationError(_("There is a syntax error in the barcode pattern %(pattern)s: empty braces.", pattern=rule.pattern))
            elif len(findall) != 0:
                raise ValidationError(_("There is a syntax error in the barcode pattern %(pattern)s: a rule can only contain one pair of braces.", pattern=rule.pattern))
            elif p == '*':
                raise ValidationError(_(" '*' is not a valid Regex Barcode Pattern. Did you mean '.*' ?"))
            try:
                re.compile(re.sub('{N+D*}', '', p))
            except re.error:
                raise ValidationError(_("The barcode pattern %(pattern)s does not lead to a valid regular expression.", pattern=rule.pattern))

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
        if self.env.user._is_internal():
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
from . import barcode_events_mixin
from . import barcode_nomenclature
from . import barcode_rule
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

## File: static\src\barcode_handlers.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { getVisibleElements } from "@web/core/utils/ui";
import { MacroEngine } from "@web/core/macro";

function clickOnButton(selector) {
    const button = document.body.querySelector(selector);
    if (button) {
        button.click();
    }
}
function updatePager(position) {
    const pager = document.body.querySelector("nav.o_pager");
    if (!pager || pager.innerText.includes("-")) {
        // we don't change pages if we are in a multi record view
        return;
    }
    let next;
    if (position === "first") {
        next = 1;
    } else {
        next = parseInt(pager.querySelector(".o_pager_limit").textContent, 10);
    }
    let current = parseInt(pager.innerText.split('/')[0], 10);
    if (current === next) {
        return;
    }
    const engine = new MacroEngine();
    engine.activate({
        name: "updating pager",
        timeout: 1000,
        interval: 0,
        steps: [
            {
                trigger: "span.o_pager_value",
                action: "click"
            },
            {
                trigger: "input.o_pager_value",
                action: "text",
                value: next
            }
        ]
    });
}

export const COMMANDS = {
    "O-CMD.EDIT": () => clickOnButton(".o_form_button_edit"),
    "O-CMD.DISCARD": () => clickOnButton(".o_form_button_cancel"),
    "O-CMD.SAVE": () => clickOnButton(".o_form_button_save"),
    "O-CMD.PREV": () => clickOnButton(".o_pager_previous"),
    "O-CMD.NEXT": () => clickOnButton(".o_pager_next"),
    "O-CMD.PAGER-FIRST": () => updatePager("first"),
    "O-CMD.PAGER-LAST": () => updatePager("last"),
};

export const barcodeGenericHandlers = {
    dependencies: ["ui", "barcode", "notification"],
    start(env, { ui, barcode, notification }) {

        barcode.bus.addEventListener("barcode_scanned", (ev) => {
            const barcode = ev.detail.barcode;
            if (barcode.startsWith("O-BTN.")) {
                let targets = [];
                try {
                    // the scanned barcode could be anything, and could crash the queryselectorall
                    // function
                    targets = getVisibleElements(ui.activeElement, `[barcode_trigger=${barcode.slice(6)}]`);
                } catch (_e) {
                    console.warn(`Barcode '${barcode}' is not valid`);
                }
                for (let elem of targets) {
                    elem.click();
                }
            }
            if (barcode.startsWith("O-CMD.")) {
                const fn = COMMANDS[barcode];
                if (fn) {
                    fn();
                } else {
                    notification.add(env._t("Barcode: ") + `'${barcode}'`, {
                        title: env._t("Unknown barcode command"),
                        type: "danger"
                    });
                }
            }
        });
    }
};

registry.category("services").add("barcode_handlers", barcodeGenericHandlers);

```

## File: static\src\barcode_handler_field.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { standardFieldProps } from "@web/views/fields/standard_field_props";
import { useBus, useService } from "@web/core/utils/hooks";

const { Component, xml } = owl;

export class BarcodeHandlerField extends Component {
    setup() {
        const barcode = useService("barcode");
        useBus(barcode.bus, "barcode_scanned", this.onBarcodeScanned);
    }
    onBarcodeScanned(event) {
        const { barcode } = event.detail;
        this.props.update(barcode);
    }
}

BarcodeHandlerField.template = xml``;
BarcodeHandlerField.props = { ...standardFieldProps };

registry.category("fields").add("barcode_handler", BarcodeHandlerField);

```

## File: static\src\barcode_service.js

```javascript
/** @odoo-module **/

import { isBrowserChrome, isMobileOS } from "@web/core/browser/feature_detection";
import { registry } from "@web/core/registry";
import { session } from "@web/session";

const { EventBus, whenReady } = owl;

function isEditable(element) {
    return element.matches('input,textarea,[contenteditable="true"]');
}

function makeBarcodeInput() {
    const inputEl = document.createElement('input');
    inputEl.setAttribute("style", "position:fixed;top:50%;transform:translateY(-50%);z-index:-1;opacity:0");
    inputEl.setAttribute("autocomplete", "off");
    inputEl.setAttribute("inputmode", "none"); // magic! prevent native keyboard from popping
    inputEl.classList.add("o-barcode-input");
    inputEl.setAttribute('name', 'barcode');
    return inputEl;
}

export const barcodeService = {
    // Keys from a barcode scanner are usually processed as quick as possible,
    // but some scanners can use an intercharacter delay (we support <= 50 ms)
    maxTimeBetweenKeysInMs: session.max_time_between_keys_in_ms || 100,

    // this is done here to make it easily mockable in mobile tests
    isMobileChrome: isMobileOS() && isBrowserChrome(),

    cleanBarcode: function(barcode) {
        return barcode.replace(/Alt|Shift|Control/g, '');
    },

    start() {
        const bus = new EventBus();
        let timeout = null;

        let bufferedBarcode = "";
        let currentTarget = null;
        let barcodeInput = null;

        function handleBarcode(barcode, target) {
            bus.trigger('barcode_scanned', {barcode,target});
            if (target.getAttribute('barcode_events') === "true") {
                const barcodeScannedEvent = new CustomEvent("barcode_scanned", { detail: { barcode, target } });
                target.dispatchEvent(barcodeScannedEvent);
            }
        }

        /**
         * check if we have a barcode, and trigger appropriate events
         */
        function checkBarcode(ev) {
            let str = barcodeInput ? barcodeInput.value : bufferedBarcode;
            str = barcodeService.cleanBarcode(str);
            if (str.length >= 3) {
                if (ev) {
                    ev.preventDefault();
                }
                handleBarcode(str, currentTarget);
            }
            if (barcodeInput) {
                barcodeInput.value = "";
            }
            bufferedBarcode = "";
            currentTarget = null;
        }

        function keydownHandler(ev) {
            if (!ev.key) {
                // Chrome may trigger incomplete keydown events under certain circumstances.
                // E.g. when using browser built-in autocomplete on an input.
                // See https://stackoverflow.com/questions/59534586/google-chrome-fires-keydown-event-when-form-autocomplete
                return;
            }
            // Ignore 'Shift', 'Escape', 'Backspace', 'Insert', 'Delete', 'Home', 'End', Arrow*, F*, Page*, ...
            // meta is often used for UX purpose (like shortcuts)
            // Notes:
            // - shiftKey is not ignored because it can be used by some barcode scanner for digits.
            // - altKey/ctrlKey are not ignored because it can be used in some barcodes (e.g. GS1 separator)
            const isSpecialKey = !['Control', 'Alt'].includes(ev.key) && (ev.key.length > 1 || ev.metaKey);
            const isEndCharacter = ev.key.match(/(Enter|Tab)/);

            // Don't catch non-printable keys except 'enter' and 'tab'
            if (isSpecialKey && !isEndCharacter) {
                return;
            }

            currentTarget = ev.target;
            // Don't catch events targeting elements that are editable because we
            // have no way of redispatching 'genuine' key events. Resent events
            // don't trigger native event handlers of elements. So this means that
            // our fake events will not appear in eg. an <input> element.
            if (currentTarget !== barcodeInput && isEditable(currentTarget) &&
                !currentTarget.dataset.enableBarcode &&
                currentTarget.getAttribute("barcode_events") !== "true") {
                return;
            }

            clearTimeout(timeout);
            if (isEndCharacter) {
                checkBarcode(ev);
            } else {
                bufferedBarcode += ev.key;
                timeout = setTimeout(checkBarcode, barcodeService.maxTimeBetweenKeysInMs);
            }
        }

        function mobileChromeHandler(ev) {
            if (ev.key === "Unidentified") {
                return;
            }
            if ($(document.activeElement).not('input:text, textarea, [contenteditable], ' +
                '[type="email"], [type="number"], [type="password"], [type="tel"], [type="search"]').length) {
                barcodeInput.focus();
            }
            keydownHandler(ev);
        }

        whenReady(() => {
            const isMobileChrome = barcodeService.isMobileChrome;
            if (isMobileChrome) {
                barcodeInput = makeBarcodeInput();
                document.body.appendChild(barcodeInput);
            }
            const handler = isMobileChrome ? mobileChromeHandler : keydownHandler;
            document.body.addEventListener('keydown', handler);
        });

        return {
            bus,
        };
    },
};

registry.category("services").add("barcode", barcodeService);

```

## File: static\src\float_scannable_field.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { FloatField } from "@web/views/fields/float/float_field";

export class FloatScannableField extends FloatField {
    onBarcodeScanned() {
        this.inputRef.el.dispatchEvent(new InputEvent("input"));
    }
}
FloatScannableField.template = "barcodes.FloatScannableField";

registry.category("fields").add("field_float_scannable", FloatScannableField);

```

## File: static\src\float_scannable_field.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="barcodes.FloatScannableField" t-inherit="web.FloatField" t-inherit-mode="primary" owl="1">
        <xpath expr="//input" position="attributes">
            <attribute name="t-on-barcode_scanned">onBarcodeScanned</attribute>
            <attribute name="t-att-data-enable-barcode">true</attribute>
            <attribute name="barcode_events">true</attribute>
        </xpath>
    </t>

</templates>

```

## File: static\src\js\barcode_events.js

```javascript
/** @odoo-module **/

/**
 * BarcodeEvents has been removed and replaced by the barcode service.
 * 
 * This file is a temporary service to remap barcode events from new barcode
 * service to core.bus (which was the purpose of BarcodeEvents).
 * 
 * @TODO: remove this as soon as all barcode code is using new barcode service
 */

import { registry } from "@web/core/registry";
import core from "web.core";

export const barcodeRemapperService = {
    dependencies: ["barcode"],
    start(env, { barcode }) {
        barcode.bus.addEventListener("barcode_scanned", ev => {
            const { barcode, target } = ev.detail;
            core.bus.trigger('barcode_scanned', barcode, target);
        });
    },
};
registry.category("services").add("barcode_remapper", barcodeRemapperService);

```

## File: static\src\js\barcode_field.js

```javascript
odoo.define('barcodes.field', function(require) {
"use strict";

var AbstractField = require('web.AbstractField');
var basicFields = require('web.basic_fields');
var fieldRegistry = require('web.field_registry');
var core = require('web.core');

// Field in which the user can both type normally and scan barcodes

var FieldFloatScannable = basicFields.FieldFloat.extend({
    events: _.extend({}, basicFields.FieldFloat.prototype.events, {
        barcode_scanned: '_onBarcodeScan',
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
            self.$input[0].dataset.enableBarcode = true;
        });
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    _onBarcodeScan() {
        // trigger an 'input' event to make sure that the widget is call
        // notifyChanges
        this.$input.trigger('input');
    }
});

var FormViewBarcodeHandler = AbstractField.extend({
    /**
     * @override
     */
    init: function() {
        this._super.apply(this, arguments);
        core.bus.on('barcode_scanned', this, this._barcodeScanned);
    },
    destroy: function () {
        core.bus.off('barcode_scanned', this, this._barcodeScanned);
        this._super();
    },
    _barcodeScanned(barcode) {
        this._setValue(barcode);
    },
});

fieldRegistry.add('field_float_scannable', FieldFloatScannable);
fieldRegistry.add('barcode_handler', FormViewBarcodeHandler);

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
        this.nomenclature = attributes.nomenclature;
        this.loaded = this.load();
    },

    // This loads the barcode nomenclature and barcode rules which are
    // necessary to parse the barcodes. The BarcodeParser is operational
    // only when those data have been loaded
    load: function(){
        if (!this.nomenclature_id) {
            return this.nomenclature ? Promise.resolve() : Promise.reject();
        }
        var id = this.nomenclature_id[0];
        return rpc.query({
                model: 'barcode.nomenclature',
                method: 'read',
                args: [[id], this._barcodeNomenclatureFields()],
            }).then(nomenclatures => {
                this.nomenclature = nomenclatures[0];
                var args = [
                    [['barcode_nomenclature_id', '=', this.nomenclature.id]],
                    this._barcodeRuleFields(),
                ];
                return rpc.query({
                    model: 'barcode.rule',
                    method: 'search_read',
                    args: args,
                });
            }).then(rules => {
                rules = rules.sort(function(a, b){ return a.sequence - b.sequence; });
                this.nomenclature.rules = rules;
            });
    },

    // resolves when the barcode parser is operational.
    is_loaded: function() {
        return this.loaded;
    },

    /**
     * This algorithm is identical for all fixed length numeric GS1 data structures.
     *
     * It is also valid for EAN-8, EAN-12 (UPC-A), EAN-13 check digit after sanitizing.
     * https://www.gs1.org/sites/default/files/docs/barcodes/GS1_General_Specifications.pdf
     *
     * @param {String} numericBarcode Need to have a length of 18
     * @returns {number} Check Digit
     */
    get_barcode_check_digit(numericBarcode) {
        let oddsum = 0, evensum = 0, total = 0;
        // Reverses the barcode to be sure each digit will be in the right place
        // regardless the barcode length.
        const code = numericBarcode.split('').reverse();
        // Removes the last barcode digit (should not be took in account for its own computing).
        code.shift();

        // Multiply value of each position by
        // N1  N2  N3  N4  N5  N6  N7  N8  N9  N10 N11 N12 N13 N14 N15 N16 N17 N18
        // x3  X1  x3  x1  x3  x1  x3  x1  x3  x1  x3  x1  x3  x1  x3  x1  x3  CHECK_DIGIT
        for (let i = 0; i < code.length; i++) {
            if (i % 2 === 0) {
                evensum += parseInt(code[i]);
            } else {
                oddsum += parseInt(code[i]);
            }
        }
        total = evensum * 3 + oddsum;
        return (10 - total % 10) % 10;
    },

    /**
     * Checks if the barcode string is encoded with the provided encoding.
     *
     * @param {String} barcode
     * @param {String} encoding could be 'any' (no encoding rules), 'ean8', 'upca' or 'ean13'
     * @returns {boolean}
     */
    check_encoding: function(barcode, encoding) {
        if (encoding === 'any') {
            return true;
        }
        const barcodeSizes = {
            ean8: 8,
            ean13: 13,
            upca: 12,
        };
        return barcode.length === barcodeSizes[encoding] && /^\d+$/.test(barcode) &&
            this.get_barcode_check_digit(barcode) === parseInt(barcode[barcode.length - 1]);
    },

    /**
     * Sanitizes a EAN-13 prefix by padding it with chars zero.
     *
     * @param {String} ean
     * @returns {String}
     */
    sanitize_ean: function(ean){
        ean = ean.substr(0, 13);
        ean = "0".repeat(13 - ean.length) + ean;
        return ean.substr(0, 12) + this.get_barcode_check_digit(ean);
    },

    /**
     * Sanitizes a UPC-A prefix by padding it with chars zero.
     *
     * @param {String} upc
     * @returns {String}
     */
    sanitize_upc: function(upc) {
        return this.sanitize_ean(upc).substr(1, 12);
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
            match.value = parseInt(whole_part) + parseFloat(decimal_part);

            // replace numerical content by 0's in barcode and pattern
            match.base_code = barcode.substr(0, num_start);
            base_pattern = pattern.substr(0, num_start);
            for(var i=0;i<(num_length-2);i++) {
                match.base_code += "0";
                base_pattern += "0";
            }
            match.base_code += barcode.substr(num_start + num_length - 2, barcode.length - 1);
            base_pattern += pattern.substr(num_start + num_length, pattern.length - 1);

            match.base_code = match.base_code
                .replace("\\\\", "\\")
                .replace("\{", "{")
                .replace("\}","}")
                .replace("\.",".");

            var base_code = match.base_code.split('');
            if (encoding === 'ean13') {
                base_code[12] = '' + this.get_barcode_check_digit(match.base_code);
            } else if (encoding === 'ean8') {
                base_code[7]  = '' + this.get_barcode_check_digit(match.base_code);
            } else if (encoding === 'upca') {
                base_code[11] = '' + this.get_barcode_check_digit(match.base_code);
            }
            match.base_code = base_code.join('');
        }

        base_pattern = base_pattern.split('|')
            .map(part => part.startsWith('^') ? part : '^' + part)
            .join('|');
        match.match = match.base_code.match(base_pattern);

        return match;
    },

    /**
     * Attempts to interpret a barcode (string encoding a barcode Code-128)
     *
     * @param {string} barcode
     * @returns {Object} the returned object containing informations about the barcode:
     *      - code: the barcode
     *      - type: the type of the barcode (e.g. alias, unit product, weighted product...)
     *      - value: if the barcode encodes a numerical value, it will be put there
     *      - base_code: the barcode with all the encoding parts set to zero; the one put on the product in the backend
     */
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
                    this.nomenclature.upc_ean_conv in {'ean2upc':'','always':''} ){
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

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    _barcodeNomenclatureFields: function () {
        return [
            'name',
            'rule_ids',
            'upc_ean_conv',
        ];
    },

    _barcodeRuleFields: function () {
        return [
            'name',
            'sequence',
            'type',
            'encoding',
            'pattern',
            'alias',
        ];
    },
});

return BarcodeParser;
});

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
                        <group name="general_attributes" col="4">
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
                    <group>
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

