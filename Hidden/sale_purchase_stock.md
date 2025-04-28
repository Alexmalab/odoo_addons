# Odoo Module: sale_purchase_stock

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'MTO Sale <-> Purchase',
    'version': '1.0',
    'category': 'Hidden',
    'summary': 'SO/PO relation in case of MTO',
    'description': """
Add relation information between Sale Orders and Purchase Orders if Make to Order (MTO) is activated on one sold product.
""",
    'depends': ['sale_stock', 'purchase_stock', 'sale_purchase'],
    'data': [],
    'demo': [],
    'qweb': [],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\purchase_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class PurchaseOrder(models.Model):
    _inherit = 'purchase.order'

    @api.depends('order_line.move_dest_ids.group_id.sale_id', 'order_line.move_ids.move_dest_ids.group_id.sale_id')
    def _compute_sale_order_count(self):
        super(PurchaseOrder, self)._compute_sale_order_count()

    def _get_sale_orders(self):
        return super(PurchaseOrder, self)._get_sale_orders() | self.order_line.move_dest_ids.group_id.sale_id | self.order_line.move_ids.move_dest_ids.group_id.sale_id

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class SaleOrder(models.Model):
    _inherit = 'sale.order'

    @api.depends('procurement_group_id.stock_move_ids.created_purchase_line_id.order_id', 'procurement_group_id.stock_move_ids.move_orig_ids.purchase_line_id.order_id')
    def _compute_purchase_order_count(self):
        super(SaleOrder, self)._compute_purchase_order_count()

    def _get_purchase_orders(self):
        return super(SaleOrder, self)._get_purchase_orders() | self.procurement_group_id.stock_move_ids.created_purchase_line_id.order_id | self.procurement_group_id.stock_move_ids.move_orig_ids.purchase_line_id.order_id

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import purchase_order
from . import sale_order

```

