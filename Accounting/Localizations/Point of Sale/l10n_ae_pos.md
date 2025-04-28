# Odoo Module: l10n_ae_pos

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
    'name': 'United Arab Emirates - Point of Sale',
    'author': 'Odoo S.A.',
    'category': 'Accounting/Localizations/Point of Sale',
    'icon': '/l10n_ae/static/description/icon.png',
    'description': """
United Arab Emirates POS Localization
=======================================================
    """,
    'depends': ['l10n_ae', 'point_of_sale'],
    'auto_install': True,
    'license': 'LGPL-3',
    'assets': {
        'point_of_sale.assets': [
            'l10n_ae_pos/static/src/xml/Screens/ReceiptScreen/OrderReceipt.xml',
        ],
    },
}

```

## File: static\src\xml\Screens\ReceiptScreen\OrderReceipt.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="OrderReceiptVAT" t-inherit="point_of_sale.OrderReceipt" t-inherit-mode="extension" owl="1">
        <xpath expr="//div[contains(text(), 'Total Taxes')]" position="replace">
            <div>
                <t t-if="env.pos.company.country.code == 'AE'">VAT</t>
                <t t-else="">Total Taxes</t>
                <span t-esc="env.pos.format_currency(receipt.total_tax)" class="pos-receipt-right-align"/>
            </div>
        </xpath>
    </t>
</templates>

```

