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

## File: static\src\overrides\models.js

```javascript
/** @odoo-module */

import { patch } from "@web/core/utils/patch";
import { PosStore } from "@point_of_sale/app/store/pos_store";

patch(PosStore.prototype, {
    getReceiptHeaderData(order) {
        return {
            ...super.getReceiptHeaderData(...arguments),
            is_gcc_country: ["SA", "AE", "BH", "OM", "QA", "KW"].includes(
                this.company.country?.code
            ),
        };
    },
});

```

## File: static\src\overrides\app\screens\receipt_screen\receipt_screen.js

```javascript
/** @odoo-module */

import { Order, Orderline } from "@point_of_sale/app/store/models";
import { patch } from "@web/core/utils/patch";

patch(Order.prototype, {
    export_for_printing() {
        return {
            ...super.export_for_printing(),
            is_gcc_country: ["SA", "AE", "BH", "OM", "QA", "KW"].includes(
                this.pos.company.country?.code
            ),
        };
    },
});

patch(Orderline.prototype, {
    getDisplayData() {
        return {
            ...super.getDisplayData(),
            tax: this.env.utils.formatCurrency(this.get_tax(), false),
        };
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

        <xpath expr="//t[@t-esc='props.data.cashier']/.." position="attributes">
            <attribute name="t-if">!props.data.is_gcc_country</attribute>
        </xpath>
        <xpath expr="//t[@t-esc='props.data.cashier']/.." position="after">
            <div t-if="props.data.is_gcc_country" t-translation="off">
                <div>Served by / خدم بواسطة <t t-esc="props.data.cashier"/></div>
            </div>
        </xpath>
    </t>

    <t t-inherit="point_of_sale.OrderReceipt" t-inherit-mode="extension">
        <xpath expr="//span[@t-esc='props.formatCurrency(props.data.amount_total)']/.." position="attributes">
            <attribute name="t-if">!props.data.is_gcc_country</attribute>
        </xpath>
        <xpath expr="//span[@t-esc='props.formatCurrency(props.data.amount_total)']/.." position="after">
            <div t-if="props.data.is_gcc_country" class="pos-receipt-amount pos-receipt-amount-arabic" t-translation="off">
                TOTAL / الإجمالي
                <span t-esc="props.formatCurrency(props.data.amount_total)" class="pos-receipt-right-align"/>
            </div>
        </xpath>

        <xpath expr="//t[@t-if='props.data.rounding_applied']" position="attributes">
            <attribute name="t-if" add=" and !props.data.is_gcc_country" separator=" "/>
        </xpath>
        <xpath expr="//t[@t-if='props.data.rounding_applied and !props.data.is_gcc_country']" position="after">
            <t t-elif="props.data.rounding_applied and props.data.is_gcc_country">
                <div class="pos-receipt-amount pos-receipt-amount-arabic" t-translation="off">
                    Rounding / التقريب
                    <span t-esc="props.formatCurrency(props.data.rounding_applied)" class="pos-receipt-right-align"/>
                </div>
                <div class="pos-receipt-amount">
                    To Pay
                    <span t-esc="props.formatCurrency(props.data.amount_total + props.data.rounding_applied)" class="pos-receipt-right-align"/>
                </div>
            </t>
        </xpath>

        <xpath expr="//span[@t-esc='props.formatCurrency(props.data.change)']/.." position="attributes">
            <attribute name="t-if">!props.data.is_gcc_country</attribute>
        </xpath>
        <xpath expr="//span[@t-esc='props.formatCurrency(props.data.change)']/.." position="after">
            <div t-if="props.data.is_gcc_country" class="pos-receipt-amount receipt-change pos-receipt-amount-arabic" t-translation="off">
                CHANGE / الباقي
                <span t-esc="props.formatCurrency(props.data.change)" class="pos-receipt-right-align"/>
            </div>
        </xpath>

        <xpath expr="//span[@t-esc='props.formatCurrency(props.data.total_discount)']/.." position="attributes">
            <attribute name="t-if">!props.data.is_gcc_country</attribute>
        </xpath>
        <xpath expr="//span[@t-esc='props.formatCurrency(props.data.total_discount)']/.." position="after">
            <div t-if="props.data.is_gcc_country" t-translation="off">
                Discounts / الخصومات
                <span t-esc="props.formatCurrency(props.data.total_discount)" class="pos-receipt-right-align"/>
            </div>
        </xpath>

        <xpath expr="//div[hasclass('pos-receipt-taxes')]/span[text()='Total']" position="after">
            <t t-if="props.data.is_gcc_country">
                <span/>
                <span t-translation="off">النسبة</span>
                <span t-translation="off">مبلغ الضريبة</span>
                <span t-translation="off">قبل الضريبة</span>
                <span t-translation="off">الإجمالي</span>
            </t>
        </xpath>
        <xpath expr="//Orderline">
            <t t-if="props.data.is_gcc_country">
                <div style="display: inline-flex;" t-translation="off">Taxes / الضرائب</div>:<span t-esc="line.tax " style="margin-left: 5px"/>
            </t>
        </xpath>
    </t>
</templates>

```

