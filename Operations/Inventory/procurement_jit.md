# Odoo Module: procurement_jit

Category: Operations/Inventory

This file contains the source code of the Odoo module.

## File: sale.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class SaleOrderLine(models.Model):
    _inherit = "sale.order.line"

    def _action_launch_stock_rule(self, previous_product_uom_qty=False):
        res = super(SaleOrderLine, self)._action_launch_stock_rule(previous_product_uom_qty=previous_product_uom_qty)
        orders = list(set(x.order_id for x in self))
        for order in orders:
            reassign = order.picking_ids.filtered(lambda x: x.state=='confirmed' or (x.state in ['waiting', 'assigned'] and not x.printed))
            if reassign:
                reassign.action_assign()
        return res

```

## File: stock_picking.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class StockPicking(models.Model):
    _inherit = 'stock.picking'

    def _needs_automatic_assign(self):
        self.ensure_one()
        if self.sale_id:
            return True
        return super()._needs_automatic_assign()

```

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import sale
from . import stock_picking

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


{
    'name': 'Just In Time Scheduling',
    'version': '1.0',
    'category': 'Operations/Inventory',
    'description': """
This module will automatically reserve the picking from stock when a sales order is confirmed
=============================================================================================
Upon confirmation of a sales order or when quantities are added,
the picking that reserves from stock will be reserved if the
necessary quantities are available.

In the simplest configurations, this is an easy way of working:
first come, first served.  However, when not installed, you can
use manual reservation or run the schedulers where the system
will take into account the expected date and the priority.

If this automatic reservation would reserve too much, you can
still unreserve a picking.
    """,
    'depends': ['sale_stock'],
    'data': [],
    'demo': [],
    'test': [],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

