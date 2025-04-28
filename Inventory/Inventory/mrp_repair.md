# Odoo Module: mrp_repair

Category: Inventory/Inventory

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
    'name': 'Mrp Repairs',
    'version': '1.0',
    'category': 'Inventory/Inventory',
    'depends': ['repair', 'mrp'],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\repair.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class Repair(models.Model):
    _inherit = 'repair.order'

    @api.model_create_multi
    def create(self, vals_list):
        orders = super().create(vals_list)
        orders.action_explode()
        return orders

    def write(self, vals):
        res = super().write(vals)
        self.action_explode()
        return res

    def action_explode(self):
        lines_to_unlink_ids = set()
        line_vals_list = []
        for op in self.move_ids:
            bom = self.env['mrp.bom'].sudo()._bom_find(op.product_id, company_id=op.company_id.id, bom_type='phantom')[op.product_id]
            if not bom:
                continue
            factor = op.product_uom._compute_quantity(op.product_uom_qty, bom.product_uom_id) / bom.product_qty
            _boms, lines = bom.sudo().explode(op.product_id, factor, picking_type=bom.picking_type_id)
            for bom_line, line_data in lines:
                if bom_line.product_id.type != 'service':
                    line_vals_list.append(op._prepare_phantom_line_vals(bom_line, line_data['qty']))
            lines_to_unlink_ids.add(op.id)

        self.env['stock.move'].browse(lines_to_unlink_ids).sudo().unlink()
        if line_vals_list:
            self.env['stock.move'].create(line_vals_list)


class StockMove(models.Model):
    _inherit = 'stock.move'

    def _prepare_phantom_line_vals(self, bom_line, qty):
        self.ensure_one()
        product = bom_line.product_id
        return {
            'name': self.name,
            'repair_id': self.repair_id.id,
            'repair_line_type': self.repair_line_type,
            'product_id': product.id,
            'price_unit': self.price_unit,
            'product_uom_qty': qty,
            'location_id': self.location_id.id,
            'location_dest_id': self.location_dest_id.id,
            'state': 'draft',
        }

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import repair

```

