# Odoo Module: pos_reprint

Category: Sales/Point Of Sale

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Point of Sale Receipt Reprinting',
    'version': '1.0',
    'category': 'Sales/Point Of Sale',
    'sequence': 6,
    'summary': 'Allow cashier to reprint receipts',
    'description': """

Allow cashier to reprint receipts

""",
    'depends': ['point_of_sale'],
    'data': [
        'views/pos_reprint_templates.xml',
        'views/pos_config_views.xml'
    ],
    'qweb': [
        'static/src/xml/reprint.xml',
    ],
    'installable': True,
    'license': 'LGPL-3',
}

```

## File: static\src\js\reprint.js

```javascript
odoo.define('pos_reprint.pos_reprint', function (require) {
"use strict";

var screens = require('point_of_sale.screens');
var gui = require('point_of_sale.gui');
var core = require('web.core');

var _t = core._t;

screens.ReceiptScreenWidget.include({
    get_receipt_render_env: function() {
        this.pos.last_receipt_render_env = this._super();
        return this.pos.last_receipt_render_env;
    }
});

var ReprintReceiptScreenWidget = screens.ReceiptScreenWidget.extend({
    template: 'ReprintReceiptScreenWidget',
    render_change: function() {},
    click_next: function() {},
    click_back: function() {
        this._super();
        this.gui.show_screen('products');
        // old order may be reprinted but
        // the current is still open
        this.pos.get_order()._printed = false;
    },
    get_receipt_render_env: function() {
        this.pos.last_receipt_render_env.receipt.reprint = true;
        return this.pos.last_receipt_render_env;
    },
});
gui.define_screen({name:'reprint_receipt', widget: ReprintReceiptScreenWidget});

var ReprintButton = screens.ActionButtonWidget.extend({
    template: 'ReprintButton',
    button_click: function() {
        if (this.pos.last_receipt_render_env) {
            this.gui.show_screen('reprint_receipt');
        } else {
            this.gui.show_popup('error', {
                'title': _t('Nothing to Print'),
                'body':  _t('There is no previous receipt to print.'),
            });
        }
    },
});

screens.define_action_button({
    'name': 'reprint',
    'widget': ReprintButton,
    'condition': function(){
        return this.pos.config.module_pos_reprint;
    },
});

return {
    ReprintReceiptScreenWidget: ReprintReceiptScreenWidget,
};

});

```

## File: static\src\xml\reprint.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="ReprintButton">
        <div class='control-button js_reprint'>
            <i class="fa fa-retweet"></i> Reprint Receipt
        </div>
    </t>

    <t t-name="ReprintReceiptScreenWidget" t-extend="ReceiptScreenWidget">
        <t t-jquery="div.top-content" t-operation="inner">
            <span class='button back'>
                <i class='fa fa-angle-double-left'></i>
                Back
            </span>
        </t>
    </t>

    <t t-extend="OrderReceipt">
        <t t-jquery='.pos-receipt-order-data' t-operation='append'>
            <t t-if='receipt.reprint === true'>
                <div>DUPLICATA</div>
            </t>
        </t>
    </t>

</templates>

```

## File: views\pos_config_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="pos_config_view_form_inherit_pos_reprint" model="ir.ui.view">
        <field name="name">pos.config.form.inherit.reprint</field>
        <field name="model">pos.config</field>
        <field name="inherit_id" ref="point_of_sale.pos_config_view_form"/>
        <field name="arch" type="xml">
            <div id="btn_use_pos_reprint" position="replace"/>
        </field>
    </record>
</odoo>

```

## File: views\pos_reprint_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <template id="assets" inherit_id="point_of_sale.assets">
          <xpath expr="." position="inside">
              <script type="text/javascript" src="/pos_reprint/static/src/js/reprint.js"></script>
          </xpath>
        </template>
</odoo>

```

