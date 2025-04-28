# Odoo Module: l10n_co_pos

Category: Localization

This file contains the source code of the Odoo module.

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Colombian - Point of Sale',
    'version': '1.0',
    'description': """Colombian - Point of Sale""",
    'category': 'Localization',
    'auto_install': True,
    'depends': [
        'l10n_co',
        'point_of_sale'
    ],
    'data': [
        'views/templates.xml',
        'views/views.xml'
    ],
    'qweb': [
        'static/src/xml/pos.xml'
    ],
    'license': 'LGPL-3',
}

```

## File: static\src\js\pos.js

```javascript
odoo.define('l10n_co_pos.pos', function (require) {
"use strict";

var models = require('point_of_sale.models');
var screens = require('point_of_sale.screens');
var session = require('web.session');
var rpc = require('web.rpc');

models.PosModel = models.PosModel.extend({
    is_colombian_country: function () {
        return this.company.country.code === 'CO';
    },
});

var _super_order = models.Order.prototype;
models.Order = models.Order.extend({
    export_for_printing: function () {
        var result = _super_order.export_for_printing.apply(this, arguments);
        result.l10n_co_dian = this.get_l10n_co_dian();
        return result;
    },
    set_l10n_co_dian: function (l10n_co_dian) {
        this.l10n_co_dian = l10n_co_dian;
    },
    get_l10n_co_dian: function () {
        return this.l10n_co_dian;
    },
    wait_for_push_order: function () {
        var result = _super_order.wait_for_push_order.apply(this, arguments);
        result = Boolean(result || this.pos.is_colombian_country());
        return result;
    }
});

screens.PaymentScreenWidget.include({
    post_push_order_resolve: function (order, server_ids) {
        if (this.pos.is_colombian_country()) {
            var _super = this._super;
            var args = arguments;
            var self = this;
            return new Promise (function (resolve, reject) {
                rpc.query({
                    model: 'pos.order',
                    method: 'search_read',
                    domain: [['id', 'in', server_ids]],
                    fields: ['name'],
                    context: session.user_context,
                }).then(function (result) {
                    order.set_l10n_co_dian(result[0].name || false);
                }).finally(function () {
                    _super.apply(self, args).then(function () {
                        resolve();
                    }).catch(function (error) {
                        reject(error);
                    });
                });
            });
        } else {
            return this._super(order, server_ids);
        }
    },
});

});

```

## File: static\src\xml\pos.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates id="template" xml:space="preserve">
    <t t-extend="OrderReceipt">
        <t t-jquery=".pos-receipt-order-data" t-operation="append">
            <t t-if="receipt.l10n_co_dian !== false">
                <div style="word-wrap:break-word;"><t t-esc="receipt.l10n_co_dian"/></div>
            </t>
        </t>
    </t>
</templates>

```

## File: views\templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="assets" inherit_id="point_of_sale.assets">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/l10n_co_pos/static/src/js/pos.js"></script>
        </xpath>
    </template>

</odoo>

```

## File: views\views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="pos_config_view_form" model="ir.ui.view">
        <field name="name">pos.config.form.view.inherit.l10n_co_pos</field>
        <field name="model">pos.config</field>
        <field name="inherit_id" ref="point_of_sale.pos_config_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='order_reference']" position="attributes">
                <attribute name="groups"></attribute>
            </xpath>
            <xpath expr="//field[@name='sequence_id']" position="attributes">
                <attribute name="readonly">0</attribute>
            </xpath>
        </field>
    </record>

</odoo>

```

