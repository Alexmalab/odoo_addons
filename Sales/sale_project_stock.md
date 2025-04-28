# Odoo Module: sale_project_stock

Category: Sales

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
    'name': 'Sale Project - Sale Stock',
    'version': '1.0',
    'description': 'Adds a full traceability of inventory operations on the profitability report.',
    'summary': 'Adds a full traceability of inventory operations on the profitability report.',
    'license': 'LGPL-3',
    'category': 'Sales',
    'depends': ['sale_project', 'sale_stock'],
    'data': [
        'views/stock_move_views.xml',
    ],
    'auto_install': True,
}

```

## File: models\sale_order_line.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models

class SaleOrderLine(models.Model):
    _inherit = 'sale.order.line'

    def _get_action_per_item(self):
        """ Get action per Sales Order Item to display the stock moves linked

            :returns: Dict containing id of SOL as key and the action as value
        """
        action_per_sol = super()._get_action_per_item()
        stock_move_action = self.env.ref('sale_project_stock.stock_move_per_sale_order_line_action').id
        stock_move_ids_per_sol = {}
        if self.user_has_groups('stock.group_stock_user'):
            stock_move_read_group = self.env['stock.move']._read_group([('sale_line_id', 'in', self.ids)], ['sale_line_id', 'ids:array_agg(id)'], ['sale_line_id'])
            stock_move_ids_per_sol = {res['sale_line_id'][0]: res['ids'] for res in stock_move_read_group}
        for sol in self:
            stock_move_ids = stock_move_ids_per_sol.get(sol.id, [])
            if not sol.is_service and stock_move_ids:
                action_per_sol[sol.id] = stock_move_action, stock_move_ids[0] if len(stock_move_ids) == 1 else False
        return action_per_sol

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import sale_order_line

```

## File: views\stock_move_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="stock_move_per_sale_order_line_action" model="ir.actions.act_window">
        <field name="name">Transfers</field>
        <field name="res_model">stock.move</field>
        <field name="view_mode">tree,kanban,pivot,graph,form</field>
        <field name="domain">[('sale_line_id', '=', active_id)]</field>
        <field name="context">{}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                No stock move found
            </p>
            <p>
                This menu gives you the full traceability of inventory
                operations on a specific product. You can filter on the product
                to see all the past or future movements for the product.
            </p>
        </field>
    </record>

</odoo>

```

