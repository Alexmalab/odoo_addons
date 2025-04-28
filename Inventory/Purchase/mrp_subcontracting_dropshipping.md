# Odoo Module: mrp_subcontracting_dropshipping

Category: Inventory/Purchase

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
    'name': 'Dropship and Subcontracting Management',
    'version': '0.1',
    'category': 'Inventory/Purchase',
    'description': """
        This bridge module allows to manage subcontracting with the dropshipping module.
    """,
    'depends': ['mrp_subcontracting', 'stock_dropshipping'],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\stock_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class StockMove(models.Model):
    _inherit = "stock.move"

    def _is_purchase_return(self):
        res = super()._is_purchase_return()
        return res or self._is_dropshipped_returned()

    def _is_dropshipped(self):
        res = super()._is_dropshipped()
        return res or (
                self.partner_id.property_stock_subcontractor.parent_path
                and self.partner_id.property_stock_subcontractor.parent_path in self.location_id.parent_path
                and self.location_dest_id.usage == 'customer'
        )

    def _is_dropshipped_returned(self):
        res = super()._is_dropshipped_returned()
        return res or (
                self.location_id.usage == 'customer'
                and self.partner_id.property_stock_subcontractor.parent_path
                and self.partner_id.property_stock_subcontractor.parent_path in self.location_dest_id.parent_path
        )

```

## File: models\stock_picking.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.tools import float_compare


class StockPicking(models.Model):
    _inherit = 'stock.picking'

    def _action_done(self):
        """ If needed, create a compensation layer, so we add the MO cost
        to the dropship one
        """
        res = super()._action_done()
        for move in self.move_lines:
            if not (move.is_subcontract and move._is_dropshipped() and move.state == 'done'):
                continue

            dropship_svls = move.stock_valuation_layer_ids
            if not dropship_svls:
                continue

            subcontract_svls = move.move_orig_ids.stock_valuation_layer_ids
            subcontract_value = sum(subcontract_svls.mapped('value'))
            dropship_value = abs(sum(dropship_svls.mapped('value')))
            diff = subcontract_value - dropship_value
            if float_compare(diff, 0, precision_rounding=move.company_id.currency_id.rounding) <= 0:
                continue

            svl_vals = move._prepare_common_svl_vals()
            svl_vals.update({
                'remaining_value': 0,
                'remaining_qty': 0,
                'value': -diff,
                'quantity': 0,
                'unit_cost': 0,
                'stock_valuation_layer_id': dropship_svls[0].id,
                'stock_move_id': False,
            })
            svl = self.env['stock.valuation.layer'].create(svl_vals)

            move = move.with_company(move.company_id)
            if move.product_id.valuation != 'real_time':
                continue
            move._account_entry_move(svl.quantity, svl.description, svl.id, svl.value)

        return res

    def _get_warehouse(self, subcontract_move):
        if subcontract_move.sale_line_id:
            return subcontract_move.sale_line_id.order_id.warehouse_id
        return super(StockPicking, self)._get_warehouse(subcontract_move)

    def _prepare_subcontract_mo_vals(self, subcontract_move, bom):
        res = super()._prepare_subcontract_mo_vals(subcontract_move, bom)
        if not res.get('picking_type_id') and (
                subcontract_move.location_dest_id.usage == 'customer'
                or subcontract_move.partner_id.property_stock_subcontractor.parent_path in subcontract_move.location_dest_id.parent_path
        ):
            # If the if-condition is respected, it means that `subcontract_move` is not
            # related to a specific warehouse. This can happen if, for instance, the user
            # confirms a PO with a subcontracted product that should be delivered to a
            # customer (dropshipping). In that case, we can use a default warehouse to
            # get the picking type
            default_warehouse = self.env['stock.warehouse'].search([('company_id', '=', subcontract_move.company_id.id)], limit=1)
            res['picking_type_id'] = default_warehouse.subcontracting_type_id.id,
        return res

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import stock_move
from . import stock_picking

```

