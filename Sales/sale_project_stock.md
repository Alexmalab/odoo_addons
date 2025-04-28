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
    'depends': ['sale_project', 'sale_stock', 'project_stock_account'],
    'data': [
        'views/stock_move_views.xml',
    ],
    'auto_install': True,
}

```

## File: models\project_project.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class ProjectProject(models.Model):
    _inherit = 'project.project'

    def _get_picking_action(self, action_name, picking_type=None):
        result = super()._get_picking_action(action_name, picking_type)

        if picking_type and (property_warehouse := self.env.user.property_warehouse_id):
            if picking_type == 'outgoing':
                result['context']['default_picking_type_id'] = property_warehouse.out_type_id.id
            elif picking_type == 'incoming':
                result['context']['default_picking_type_id'] = property_warehouse.in_type_id.id

        return result

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
        if self.env.user.has_group('stock.group_stock_user'):
            stock_move_read_group = self.env['stock.move']._read_group([('sale_line_id', 'in', self.ids)], ['sale_line_id'], ['id:array_agg'])
            stock_move_ids_per_sol = {sale_line.id: ids for sale_line, ids in stock_move_read_group}
        for sol in self:
            stock_move_ids = stock_move_ids_per_sol.get(sol.id, [])
            if not sol.is_service and stock_move_ids:
                action_per_sol[sol.id] = stock_move_action, stock_move_ids[0] if len(stock_move_ids) == 1 else False
        return action_per_sol

```

## File: models\stock_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models
from odoo.tools import float_is_zero


class StockMove(models.Model):
    _inherit = 'stock.move'

    def _sale_get_invoice_price(self, order):
        """ Based on the current stock move, compute the price to reinvoice the analytic line that is going to be created (so the
            price of the sale line).
        """
        self.ensure_one()

        if self.product_id.expense_policy == 'sales_price':
            return order.pricelist_id._get_product_price(
                self.product_id,
                1.0,
                uom=self.product_uom,
                date=order.date_order,
            )

        uom_precision_digits = self.env['decimal.precision'].precision_get('Product Unit of Measure')
        if float_is_zero(self.quantity, precision_digits=uom_precision_digits):
            return 0.0

        price_unit = self.product_id.standard_price
        # Prevent unnecessary currency conversion that could be impacted by exchange rate
        # fluctuations
        if self.company_id.currency_id and price_unit and self.company_id.currency_id == order.currency_id:
            return self.company_id.currency_id.round(price_unit)

        currency_id = self.company_id.currency_id
        if currency_id and currency_id != order.currency_id:
            price_unit = currency_id._convert(price_unit, order.currency_id, order.company_id, order.date_order or fields.Date.today())
        return price_unit

    def _sale_prepare_sale_line_values(self, order, price, last_sequence):
        """ Generate the sale.line creation value from the current stock move """
        self.ensure_one()

        fpos = order.fiscal_position_id or order.fiscal_position_id._get_fiscal_position(order.partner_id)
        product_taxes = self.product_id.taxes_id._filter_taxes_by_company(order.company_id)
        taxes = fpos.map_tax(product_taxes)

        return {
            'order_id': order.id,
            'name': self.name,
            'sequence': last_sequence,
            'price_unit': price,
            'tax_id': [x.id for x in taxes],
            'discount': 0.0,
            'product_id': self.product_id.id,
            'product_uom_qty': self.product_uom_qty,
            'qty_delivered': self.quantity,
        }

    def _get_new_picking_values(self):
        return {
            **super()._get_new_picking_values(),
            'project_id': self.sale_line_id.order_id.project_id.id,
        }

    def _assign_picking_values(self, picking):
        return {
            **super()._assign_picking_values(picking),
            'project_id': self[:1].sale_line_id.order_id.project_id.id,
        }

```

## File: models\stock_picking.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, models
from odoo.exceptions import UserError


class StockPicking(models.Model):
    _inherit = 'stock.picking'

    def button_validate(self):
        res = super().button_validate()
        if res is not True:
            return res

        for picking in self:
            project = picking.project_id
            sale_order = project.sudo().reinvoiced_sale_order_id
            if not (sale_order and picking.picking_type_id.analytic_costs):
                continue
            reinvoicable_stock_moves = picking.move_ids.filtered(lambda m: m.product_id.expense_policy in {'sales_price', 'cost'})
            if not reinvoicable_stock_moves:
                continue
            # raise if the sale order is not currently open
            if sale_order.state in ('draft', 'sent'):
                raise UserError(_(
                    "The Sales Order %(order)s linked to the Project %(project)s must be"
                    " validated before validating the stock picking.",
                    order=sale_order.name,
                    project=project.name,
                ))
            elif sale_order.state == 'cancel':
                raise UserError(_(
                    "The Sales Order %(order)s linked to the Project %(project)s is cancelled."
                    " You cannot validate a stock picking on a cancelled Sales Order.",
                    order=sale_order.name,
                    project=project.name,
                ))
            elif sale_order.locked:
                raise UserError(_(
                    "The Sales Order %(order)s linked to the Project %(project)s is currently locked."
                    " You cannot validate a stock picking on a locked Sales Order."
                    " Please create a new SO linked to this Project.",
                    order=sale_order.name,
                    project=project.name,
                ))
            # Create SOLs in reinvoiced_sale_order_id with reinvoicable stock moves
            sale_line_values_to_create = []
            # Get last sequence SOL
            last_so_line = self.env['sale.order.line'].search_read(
                [('order_id', '=', sale_order.id)],
                ['sequence'], order='sequence desc', limit=1,
            )
            last_sequence = next((sol['sequence'] for sol in last_so_line), 100)

            for stock_move in reinvoicable_stock_moves:
                # Get price
                price = stock_move._sale_get_invoice_price(sale_order)
                # Create the sale lines in batch
                sale_line_values_to_create.append(stock_move._sale_prepare_sale_line_values(sale_order, price, last_sequence))
                last_sequence += 1
            self.env['sale.order.line'].with_context(skip_procurement=True).sudo().create(sale_line_values_to_create)
        return res

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import project_project
from . import sale_order_line
from . import stock_move
from . import stock_picking

```

## File: views\stock_move_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="stock_move_per_sale_order_line_action" model="ir.actions.act_window">
        <field name="name">Transfers</field>
        <field name="res_model">stock.move</field>
        <field name="view_mode">list,kanban,pivot,graph,form</field>
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

