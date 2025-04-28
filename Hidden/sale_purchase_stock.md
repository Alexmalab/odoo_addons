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
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\purchase_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import Command, api, models


class PurchaseOrder(models.Model):
    _inherit = 'purchase.order'

    @api.depends('order_line.move_dest_ids.group_id.sale_id', 'order_line.move_ids.move_dest_ids.group_id.sale_id')
    def _compute_sale_order_count(self):
        super()._compute_sale_order_count()

    def _get_sale_orders(self):
        linked_so = self.order_line.move_dest_ids.group_id.sale_id \
                  | self.env['stock.move'].browse(self.order_line.move_ids._rollup_move_dests()).group_id.sale_id
        group_so = self.order_line.group_id.sale_id

        return super()._get_sale_orders() | linked_so | group_so


class PurchaseOrderLine(models.Model):
    _inherit = 'purchase.order.line'

    def _prepare_stock_moves(self, picking):
        res = super()._prepare_stock_moves(picking)
        for re in res:
            re['sale_line_id'] = self.sale_line_id.id
            if self.sale_line_id.route_id:
               re['route_ids'] = [Command.link(self.sale_line_id.route_id.id)]
            if self.order_id.dest_address_id:
                # In a dropshipping context we do not need the description of the purchase order or it will be displayed
                # in Delivery slip report and it may be confusing for the customer to see several times the same text (product name + description_picking).
                product = self.product_id.with_context(lang=self.order_id.dest_address_id.lang or self.env.user.lang)
                re['description_picking'] = product._get_description(
                    self.env['stock.picking.type'].browse(re['picking_type_id'])
                )
        return res

    def _find_candidate(self, product_id, product_qty, product_uom, location_id, name, origin, company_id, values):
        # if this is defined, this is a dropshipping line, so no
        # this is to correctly map delivered quantities to the so lines
        lines = self.filtered(lambda po_line: po_line.sale_line_id.id == values['sale_line_id']) if values.get('sale_line_id') else self
        return super(PurchaseOrderLine, lines)._find_candidate(product_id, product_qty, product_uom, location_id, name, origin, company_id, values)

    @api.model
    def _prepare_purchase_order_line_from_procurement(self, product_id, product_qty, product_uom, location_dest_id, name, origin, company_id, values, po):
        res = super()._prepare_purchase_order_line_from_procurement(product_id, product_qty, product_uom, location_dest_id, name, origin, company_id, values, po)
        res['sale_line_id'] = values.get('sale_line_id', False)
        return res

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class SaleOrder(models.Model):
    _inherit = 'sale.order'

    @api.depends('procurement_group_id.stock_move_ids.created_purchase_line_ids.order_id', 'procurement_group_id.stock_move_ids.move_orig_ids.purchase_line_id.order_id')
    def _compute_purchase_order_count(self):
        super()._compute_purchase_order_count()

    def _get_purchase_orders(self):
        linked_po = self.procurement_group_id.stock_move_ids.created_purchase_line_ids.order_id \
                  | self.env['stock.move'].browse(self.procurement_group_id.stock_move_ids._rollup_move_origs()).purchase_line_id.order_id
        group_po = self.procurement_group_id.purchase_line_ids.order_id

        return super()._get_purchase_orders() | linked_po | group_po

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import purchase_order
from . import sale_order

```

