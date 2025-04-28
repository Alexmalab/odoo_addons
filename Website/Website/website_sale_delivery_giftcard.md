# Odoo Module: website_sale_delivery_giftcard

Category: Website/Website

This file contains the source code of the Odoo module.

## File: __init__.py

```python

```

## File: __manifest__.py

```python
{
    'name': 'Website Sale Delivery Giftcard',
    'category': 'Website/Website',
    'version': '1.0',
    'depends': ['website_sale_delivery', 'website_sale_gift_card'],
    'installable': True,
    'auto_install': True,
    'assets': {
        'web.assets_tests': [
            'website_sale_delivery_giftcard/static/tests/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

