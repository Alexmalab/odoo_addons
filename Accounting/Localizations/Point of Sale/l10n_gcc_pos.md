# Odoo Module: l10n_gcc_pos

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
    'name': 'Gulf Cooperation Council - Point of Sale',
    'category': 'Accounting/Localizations/Point of Sale',
    'description': """
GCC POS Localization
=======================================================
    """,
    'license': 'LGPL-3',
    'depends': ['point_of_sale', 'l10n_gcc_invoice'],
    'assets': {
        'point_of_sale._assets_pos': [
            'l10n_gcc_pos/static/src/**/*',
        ]
    },
    'auto_install': True,
}

```

## File: static\src\overrides\pos_store.js

```javascript
import { patch } from "@web/core/utils/patch";
import { PosStore } from "@point_of_sale/app/store/pos_store";

patch(PosStore.prototype, {
    getReceiptHeaderData(order) {
        return {
            ...super.getReceiptHeaderData(...arguments),
            is_gcc_country: ["SA", "AE", "BH", "OM", "QA", "KW"].includes(
                this.company.country_id?.code
            ),
        };
    },
});

```

## File: static\src\overrides\app\models\pos_order.js

```javascript
import { PosOrder } from "@point_of_sale/app/models/pos_order";
import { patch } from "@web/core/utils/patch";
import { _t } from "@web/core/l10n/translation";

patch(PosOrder.prototype, {
    get isGccCountry() {
        return ["SA", "AE", "BH", "OM", "QA", "KW"].includes(this.company.country_id?.code);
    },

    export_for_printing(baseUrl, headerData) {
        const results = super.export_for_printing(...arguments);
        results.is_gcc_country = this.isGccCountry;
        if (results.is_gcc_country) {
            results.label_total = _t("TOTAL / اﻹجمالي");
            results.label_rounding = _t("Rounding / التقريب");
            results.label_change = _t("CHANGE / الباقي");
            results.label_discounts = _t("Discounts / الخصومات");
        }
        return results;
    },
});

```

## File: static\src\overrides\app\screens\receipt_screen\receipt\order_receipt.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-inherit="point_of_sale.ReceiptHeader" t-inherit-mode="extension">
        <xpath expr="//div[hasclass('pos-receipt-contact')]" position="after">
            <t t-if="props.data.is_gcc_country">
                <br/>
                <br/>
                <div class="pos-receipt-header">
                    <span id="title_english" t-translation="off">Tax Invoice</span>
                </div>
                <div class="pos-receipt-header">
                    <span id="title_arabic" t-translation="off">الفاتورة الضريبية</span>
                </div>
            </t>
        </xpath>

        <xpath expr="//div[@t-esc='props.data.cashier']/.." position="attributes">
            <attribute name="t-if">!props.data.is_gcc_country</attribute>
        </xpath>
        <xpath expr="//div[@t-esc='props.data.cashier']/.." position="after">
            <div t-if="props.data.is_gcc_country" t-translation="off">
                <div>Served by / خدم بواسطة <t t-esc="props.data.cashier"/></div>
            </div>
        </xpath>
    </t>
</templates>

```

