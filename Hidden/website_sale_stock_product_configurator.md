# Odoo Module: website_sale_stock_product_configurator

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
{
    'name': "Website Sale Stock Product Configurator",
    'summary': """
        Bridge module for website_sale_stock / sale_product_configurator""",
    'description': """
        Bridge module to make the website e-commerce stock management compatible with the product configurator
    """,
    'category': 'Hidden',
    'version': '1.0',
    'depends': ['website_sale_stock', 'sale_product_configurator'],
    'auto_install': True,
    'data': [
        'views/product_configurator_templates.xml',
    ],
    'assets' : {
        'web.assets_tests': [
            'website_sale_stock_product_configurator/static/tests/**/*',
        ]
    },
    'license': 'LGPL-3',
}

```

## File: views\product_configurator_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="website_sale_stock_modal" inherit_id="sale_product_configurator.product_quantity_config" name="Stocks Modal">
        <xpath expr="//input[@type='text'][hasclass('quantity')]" position="attributes">
          <attribute name='t-att-data-max'>max(product.sudo().free_qty - product.cart_qty, 1) if handle_stock and product.type == "product" and not product.allow_out_of_stock_order else None</attribute>
        </xpath>
        <xpath expr="//div[hasclass('css_quantity')]" position="after">
          <t t-if="handle_stock">
            <div class='availability_messages'/>
          </t>
        </xpath>
    </template>
</odoo>
```

