# Odoo Module: sale_stock_margin

Category: Sales/Sales

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
{
    'name': "Sale Stock Margin",
    'category': 'Sales/Sales',
    'summary': '',
    'description': 'Once the delivery is validated, update the cost on the SO to have an exact margin computation.',
    'version': '0.1',
    'depends': ['stock_account', 'sale_margin'],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\ir_module.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, models


class IrModule(models.Model):
    _inherit = 'ir.module.module'


    @api.returns('self')
    def downstream_dependencies(
            self,
            known_deps=None,
            exclude_states=('uninstalled', 'uninstallable', 'to remove'),
            ):
        # sale_stock_margin implicitly depends on sale_stock, but sale_stock is not marked as one of
        # its dependencies, thus uninstalling sale_stock without uninstalling sale_stock_margin
        # will make the registry crash, the install works purely because sale_stock is auto-installed
        # when the dependencies of sale_stock_margin are installed
        if 'sale_stock' in self.mapped('name'):
            # we force sale_stock_margin as a dependant of sale_stock
            known_deps = (known_deps or self.browse()) | self.search([
                ('name', '=', 'sale_stock_margin'),
                ('state', '=', 'installed'),
            ], limit=1)
        return super().downstream_dependencies(known_deps, exclude_states)

```

## File: models\sale_order_line.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class SaleOrderLine(models.Model):
    _inherit = 'sale.order.line'

    @api.depends('move_ids', 'move_ids.stock_valuation_layer_ids', 'move_ids.picking_id.state')
    def _compute_purchase_price(self):
        lines_without_moves = self.browse()
        for line in self:
            product = line.product_id.with_company(line.company_id)
            if not line.move_ids:
                lines_without_moves |= line
            elif product.categ_id.property_cost_method != 'standard':
                purch_price = product._compute_average_price(0, line.product_uom_qty, line.move_ids)
                if line.product_uom and line.product_uom != product.uom_id:
                    purch_price = product.uom_id._compute_price(purch_price, line.product_uom)
                to_cur = line.currency_id or line.order_id.currency_id
                line.purchase_price = product.cost_currency_id._convert(
                    from_amount=purch_price,
                    to_currency=to_cur,
                    company=line.company_id or self.env.company,
                    date=line.order_id.date_order or fields.Date.today(),
                    round=False,
                ) if to_cur and purch_price else purch_price
        return super(SaleOrderLine, lines_without_moves)._compute_purchase_price()

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import ir_module
from . import sale_order_line

```

