# Odoo Module: sale_mrp_margin

Category: Sales/Sales

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
    'name': "Sale Mrp Margin",
    'category': 'Sales/Sales',
    'version': '0.1',
    'description': 'Handle BoM prices to compute sale margin.',
    'depends': ['sale_mrp', 'sale_stock_margin'],
    'license': 'LGPL-3',
}

```

