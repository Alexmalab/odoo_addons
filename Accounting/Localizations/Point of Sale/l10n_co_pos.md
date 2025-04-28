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

## File: static\src\overrides\models\pos.js

```javascript
import { PosStore } from "@point_of_sale/app/store/pos_store";
import { patch } from "@web/core/utils/patch";

patch(PosStore.prototype, {
    is_colombian_country() {
        return this.company.country_id?.code === "CO";
    },
});

```

## File: static\src\overrides\models\pos_order.js

```javascript
import { PosOrder } from "@point_of_sale/app/models/pos_order";
import { patch } from "@web/core/utils/patch";

patch(PosOrder.prototype, {
    is_colombian_country() {
        return this.company.country_id?.code === "CO";
    },
    export_for_printing(baseUrl, headerData) {
        const result = super.export_for_printing(...arguments);
        result.l10n_co_dian = this.name;
        return result;
    },
    wait_for_push_order() {
        var result = super.wait_for_push_order(...arguments);
        result = Boolean(result || this.is_colombian_country());
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

