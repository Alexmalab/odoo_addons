# Odoo Module: sale_gelato_stock

Category: Sales/Sales

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': "Gelato/Stock bridge",
    'category': 'Sales/Sales',
    'depends': ['sale_gelato', 'sale_stock'],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\sale_order_line.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class SaleOrderLine(models.Model):
    _inherit = 'sale.order.line'

    # === ACTION METHODS === #

    def _action_launch_stock_rule(self, **kwargs):
        """ Override of sale_stock to prevent creating pickings for Gelato products. """
        gelato_lines = self.filtered(lambda l: l.product_id.gelato_product_uid)
        super(SaleOrderLine, self - gelato_lines)._action_launch_stock_rule(**kwargs)

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import sale_order_line

```

