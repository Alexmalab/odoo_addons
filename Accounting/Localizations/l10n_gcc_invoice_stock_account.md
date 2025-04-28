# Odoo Module: l10n_gcc_invoice_stock_account

Category: Accounting/Localizations

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
    'name': "Gulf Cooperation Council WMS Accounting",
    'version': '1.0',
    'description': """
        Arabic/English for GCC + lot/SN numbers
    """,
    'author': "Odoo S.A.",
    'website': "https://www.odoo.com",
    'category': 'Accounting/Localizations',

    'depends': ['l10n_gcc_invoice', 'stock_account'],

    'data': [
        'views/report_invoice.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: views\report_invoice.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="arabic_english_invoice_with_snln" inherit_id="l10n_gcc_invoice.arabic_english_invoice">
        <xpath expr="//div[@id='total']" position="after">
          <t groups="stock_account.group_lot_on_invoice">
            <t t-set="lot_values" t-value="o._get_invoiced_lot_values()"/>
            <t t-if="lot_values">
                <br/>
                <table class="table table-sm" style="width: 50%;" name="invoice_snln_table">
                    <thead>
                        <tr>
                            <th><span>
                                Product
                                </span>
                                <br/>
                                <span>
                                 المنتج
                                </span>
                            </th>
                            <th class="text-right">
                                <span>
                                    Quantity
                                </span>
                                <br/>
                                <span>
                                    الكمية
                                </span>
                            </th>
                            <th class="text-right">
                                <span>
                                    SN/LN
                                </span>
                                <br/>
                                <span>
                                    رقم مسلسل\لوط
                                </span>
                            </th>
                        </tr>
                    </thead>
                    <tbody>
                        <t t-foreach="lot_values" t-as="snln_line">
                            <tr>
                                <td><t t-esc="snln_line['product_name']"/></td>
                                <td class="text-right">
                                    <t t-esc="snln_line['quantity']"/>
                                    <t t-esc="snln_line['uom_name']" groups="uom.group_uom"/>
                                </td>
                                <td class="text-right"><t t-esc="snln_line['lot_name']"/></td>
                            </tr>
                        </t>
                    </tbody>
                </table>
            </t>
          </t>
        </xpath>
    </template>
</odoo>

```

