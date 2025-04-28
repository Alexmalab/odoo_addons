# Odoo Module: l10n_co_pos

Category: Accounting/Localizations/Point of Sale

This file contains the source code of the Odoo module.

## File: __init__.py

```python

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Colombian - Point of Sale',
    'version': '1.0',
    'description': """Colombian - Point of Sale""",
    'category': 'Accounting/Localizations/Point of Sale',
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

## File: static\src\js\PaymentScreen.js

```javascript
odoo.define('l10n_co_pos.PaymentScreen', function(require) {
    'use strict';

    const PaymentScreen = require('point_of_sale.PaymentScreen');
    const Registries = require('point_of_sale.Registries');
    const session = require('web.session');

    const L10nCoPosPaymentScreen = PaymentScreen =>
        class extends PaymentScreen {
            async _postPushOrderResolve(order, order_server_ids) {
                try {
                    if (this.env.pos.is_colombian_country()) {
                        const result = await this.rpc({
                            model: 'pos.order',
                            method: 'search_read',
                            domain: [['id', 'in', order_server_ids]],
                            fields: ['name'],
                            context: session.user_context,
                        });
                        order.set_l10n_co_dian(result[0].name || false);
                    }
                } finally {
                    return super._postPushOrderResolve(...arguments);
                }
            }
        };

    Registries.Component.extend(PaymentScreen, L10nCoPosPaymentScreen);

    return PaymentScreen;
});

```

## File: static\src\js\pos.js

```javascript
odoo.define('l10n_co_pos.pos', function (require) {
"use strict";

var models = require('point_of_sale.models');

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

});

```

## File: static\src\xml\pos.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates id="template" xml:space="preserve">
    <t t-name="OrderReceipt" t-inherit="point_of_sale.OrderReceipt" t-inherit-mode="extension" owl="1">
        <xpath expr="//div[hasclass('pos-receipt-order-data')]" position="inside">
            <t t-if="receipt.l10n_co_dian !== false">
                <div style="word-wrap:break-word;"><t t-esc="receipt.l10n_co_dian"/></div>
            </t>
        </xpath>
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
            <script type="text/javascript" src="/l10n_co_pos/static/src/js/PaymentScreen.js"></script>
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

