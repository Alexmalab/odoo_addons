# Odoo Module: l10n_in_stock

Category: Accounting/Accounting

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
    'name': 'Indian - Stock Report(GST)',
    'version': '1.0',
    'description': """GST Stock Report""",
    'category': 'Accounting/Accounting',
    'depends': [
        'l10n_in',
        'stock',
    ],
    'data': [
        'views/report_stockpicking_operations.xml',
    ],
    'demo': [
        'data/product_demo.xml',
    ],
    'installable': True,
    'application': False,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\product_demo.xml

```xml
<odoo>
    <data noupdate="1">
        <record id="stock.product_cable_management_box" model="product.product">
            <field name="l10n_in_hsn_code">4819.60.00</field>
            <field name="l10n_in_hsn_description">Box files, letter trays, storage boxes and similar articles, of a kind used in offices, shops or the like</field>
        </record>
    </data>
</odoo>

```

## File: views\report_stockpicking_operations.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="gst_report_picking_inherit" inherit_id="stock.report_picking">
        <xpath expr="//span[@t-field='ml.product_id.description_picking']" position="after">
            <t t-if="ml.product_id and ml.product_id.l10n_in_hsn_code and o.company_id.country_id.code == 'IN'"><h6><strong class="ml16">HSN/SAC Code:</strong> <span t-field="ml.product_id.l10n_in_hsn_code"/></h6></t>
        </xpath>
    </template>

</odoo>

```

