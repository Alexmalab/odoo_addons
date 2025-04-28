# Odoo Module: l10n_co_pos

Category: Accounting/Localizations/Point of Sale

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Colombian - Point of Sale',
    'countries': ['co'],
    'version': '1.0',
    'description': """Colombian - Point of Sale""",
    'category': 'Accounting/Localizations/Point of Sale',
    'auto_install': True,
    'depends': [
        'l10n_co',
        'point_of_sale'
    ],
    'data': [
        'views/res_config_settings_views.xml'
    ],
    'assets': {
        'point_of_sale._assets_pos': [
            'l10n_co_pos/static/src/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: models\pos_order.py

```python
from odoo import models


class PosOrder(models.Model):
    _inherit = "pos.order"

    def _prepare_invoice_vals(self):
        move_vals = super()._prepare_invoice_vals()
        if "l10n_co_edi_description_code_credit" in self.env["account.move"] and move_vals.get("move_type") == "out_refund" and move_vals.get("reversed_entry_id"):
            move_vals["l10n_co_edi_description_code_credit"] = move_vals.get("l10n_co_edi_description_code_credit", "1")
        return move_vals

```

## File: models\__init__.py

```python
from . import pos_order

```

## File: static\src\overrides\components\order_receipt\order_receipt.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates id="template" xml:space="preserve">
    <t t-name="l10n_co_pos.OrderReceipt" t-inherit="point_of_sale.OrderReceipt" t-inherit-mode="extension">
        <xpath expr="//div[hasclass('pos-receipt-order-data')]" position="inside">
            <t t-if="props.data.l10n_co_dian !== false">
                <div style="word-wrap:break-word;"><t t-esc="props.data.l10n_co_dian"/></div>
            </t>
        </xpath>
    </t>
</templates>

```

## File: static\src\overrides\components\payment_screen\payment_screen.js

```javascript
/** @odoo-module */

import { PaymentScreen } from "@point_of_sale/app/screens/payment_screen/payment_screen";
import { patch } from "@web/core/utils/patch";

patch(PaymentScreen.prototype, {
    async _postPushOrderResolve(order, order_server_ids) {
        try {
            if (this.pos.is_colombian_country()) {
                const result = await this.orm.searchRead(
                    "pos.order",
                    [["id", "in", order_server_ids]],
                    ["name"]
                );
                order.set_l10n_co_dian(result[0].name || false);
            }
        } catch {
            // FIXME this doesn't seem correct but is equivalent to return in finally which we had before.
        }
        return super._postPushOrderResolve(...arguments);
    },
});

```

## File: static\src\overrides\models\pos.js

```javascript
/** @odoo-module */

import { PosStore } from "@point_of_sale/app/store/pos_store";
import { Order } from "@point_of_sale/app/store/models";
import { patch } from "@web/core/utils/patch";

patch(PosStore.prototype, {
    is_colombian_country() {
        return this.company.country?.code === "CO";
    },
});

patch(Order.prototype, {
    export_for_printing() {
        const result = super.export_for_printing(...arguments);
        result.l10n_co_dian = this.get_l10n_co_dian();
        return result;
    },
    set_l10n_co_dian(l10n_co_dian) {
        this.l10n_co_dian = l10n_co_dian;
    },
    get_l10n_co_dian() {
        return this.l10n_co_dian;
    },
    wait_for_push_order() {
        var result = super.wait_for_push_order(...arguments);
        result = Boolean(result || this.pos.is_colombian_country());
        return result;
    },
});

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.l10n_co_pos</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="point_of_sale.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//setting[@id='order_reference']" position="attributes">
                <attribute name="groups"></attribute>
            </xpath>
            <xpath expr="//field[@name='pos_sequence_id']" position="attributes">
                <attribute name="readonly">0</attribute>
            </xpath>
        </field>
    </record>

</odoo>

```

