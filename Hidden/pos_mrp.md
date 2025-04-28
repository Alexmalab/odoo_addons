# Odoo Module: pos_mrp

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
    'name': 'pos_mrp',
    'version': '1.0',
    'category': 'Hidden',
    'sequence': 6,
    'summary': 'Link module between Point of Sale and Mrp',
    'description': """
    This is a link module between Point of Sale and Mrp.
""",
    'depends': ['point_of_sale', 'mrp'],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\pos_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models

class PosOrderLine(models.Model):
    _inherit = "pos.order.line"

    def _get_stock_moves_to_consider(self, stock_moves, product):
        self.ensure_one()
        bom = product.env['mrp.bom']._bom_find(product, company_id=stock_moves.company_id.id, bom_type='phantom').get(product)
        if not bom:
            return super()._get_stock_moves_to_consider(stock_moves, product)
        boms, components = bom.explode(product, self.qty)
        ml_product_to_consider = (product.bom_ids and [comp[0].product_id.id for comp in components]) or [product.id]
        if len(boms) > 1:
            return stock_moves.filtered(lambda ml: ml.product_id.id in ml_product_to_consider and ml.bom_line_id)
        return stock_moves.filtered(lambda ml: ml.product_id.id in ml_product_to_consider and (ml.bom_line_id in bom.bom_line_ids))

class PosOrder(models.Model):
    _inherit = "pos.order"

    def _get_pos_anglo_saxon_price_unit(self, product, partner_id, quantity):
        bom = product.env['mrp.bom']._bom_find(product, company_id=self.mapped('picking_ids.move_line_ids').company_id.id, bom_type='phantom')[product]
        if not bom:
            return super()._get_pos_anglo_saxon_price_unit(product, partner_id, quantity)
        _dummy, components = bom.explode(product, quantity)
        total_price_unit = 0
        for comp in components:
            price_unit = super()._get_pos_anglo_saxon_price_unit(comp[0].product_id, partner_id, comp[1]['qty'])
            price_unit = comp[0].product_id.uom_id._compute_price(price_unit, comp[0].product_uom_id)
            qty_per_kit = comp[1]['qty'] / bom.product_qty / quantity
            total_price_unit += price_unit * qty_per_kit
        return total_price_unit

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import pos_order

```

