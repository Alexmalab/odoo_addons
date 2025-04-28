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
    'data': [
        'data/mrp_subcontracting_dropshipping_data.xml',
        'views/stock_warehouse_views.xml',
    ],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\mrp_subcontracting_dropshipping_data.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data noupdate="1">

        <record id="route_subcontracting_dropshipping" model='stock.location.route'>
            <field name="name">Dropship Subcontractor on Order</field>
            <field name="sequence">5</field>
            <field name="company_id"></field>
            <field name="product_selectable" eval="True"/>
        </record>

        <function model="res.company" name="create_missing_subcontracting_dropshipping_rules"/>

        <function model="stock.warehouse" name="write">
            <value model="stock.warehouse" eval="obj().env['stock.warehouse'].search([]).ids"/>
            <value eval="{'subcontracting_dropshipping_to_resupply': True}"/>
        </function>

    </data>
</odoo>

```

## File: models\purchase.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, _
from odoo.exceptions import UserError


class PurchaseOrder(models.Model):
    _inherit = 'purchase.order'

    def _get_destination_location(self):
        self.ensure_one()
        if not self.dest_address_id:
            return super()._get_destination_location()

        mrp_production_ids = self._get_mrp_productions(remove_archived_picking_types=False)
        if mrp_production_ids:
            if self.dest_address_id in mrp_production_ids.bom_id.subcontractor_ids:
                return self.dest_address_id.property_stock_subcontractor.id
        elif self.sale_order_count:
            return super()._get_destination_location()

        in_bom_products = False
        not_in_bom_products = self.env['purchase.order.line']
        for order_line in self.order_line:
            if any(bom_line.bom_id.type == 'subcontract' and self.dest_address_id in bom_line.bom_id.subcontractor_ids for bom_line in order_line.product_id.bom_line_ids.filtered(lambda line: line.company_id == self.company_id)):
                in_bom_products = True
            elif not order_line.display_type:
                not_in_bom_products |= order_line
        if in_bom_products and not_in_bom_products:
            raise UserError(
                _("It appears some components in this RFQ are not meant for subcontracting. Please create a separate order for these.") +
                '\n\n' + '\n'.join(not_in_bom_products.mapped('name'))
            )
        elif in_bom_products:
            return self.dest_address_id.property_stock_subcontractor.id
        return super()._get_destination_location()

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class ResCompany(models.Model):
    _inherit = 'res.company'

    def _create_subcontracting_dropshipping_rules(self):
        route = self.env.ref('mrp_subcontracting_dropshipping.route_subcontracting_dropshipping')
        supplier_location = self.env.ref('stock.stock_location_suppliers')
        vals = []
        for company in self:
            subcontracting_location = company.subcontracting_location_id
            dropship_picking_type = self.env['stock.picking.type'].search([
                ('company_id', '=', company.id),
                ('default_location_src_id.usage', '=', 'supplier'),
                ('default_location_dest_id.usage', '=', 'customer'),
            ], limit=1, order='sequence')
            if dropship_picking_type:
                vals.append({
                    'name': '%s → %s' % (supplier_location.name, subcontracting_location.name),
                    'action': 'buy',
                    'location_id': subcontracting_location.id,
                    'location_src_id': supplier_location.id,
                    'procure_method': 'make_to_stock',
                    'route_id': route.id,
                    'picking_type_id': dropship_picking_type.id,
                    'company_id': company.id,
                })
        if vals:
            self.env['stock.rule'].create(vals)

    @api.model
    def create_missing_subcontracting_dropshipping_rules(self):
        route = self.env.ref('mrp_subcontracting_dropshipping.route_subcontracting_dropshipping')
        company_ids = self.env['res.company'].search([])
        company_has_rules = self.env['stock.rule'].search([('route_id', '=', route.id)]).mapped('company_id')
        company_todo_rules = company_ids - company_has_rules
        company_todo_rules._create_subcontracting_dropshipping_rules()

    def _create_per_company_rules(self):
        res = super()._create_per_company_rules()
        self.create_missing_subcontracting_dropshipping_rules()
        return res

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

    def _compute_is_dropship(self):
        dropship_subcontract_pickings = self.filtered(lambda p:
            p.location_id.usage == 'supplier'
            and any(m.location_dest_id == m.partner_id.property_stock_subcontractor
                    or (m.partner_id.property_stock_subcontractor.parent_path
                        and m.location_dest_id.parent_path
                        and m.partner_id.property_stock_subcontractor.parent_path in m.location_dest_id.parent_path)
                    for m in p.move_lines)
        )
        dropship_subcontract_pickings.is_dropship = True
        super(StockPicking, self - dropship_subcontract_pickings)._compute_is_dropship()

    def _get_warehouse(self, subcontract_move):
        if subcontract_move.sale_line_id:
            return subcontract_move.sale_line_id.order_id.warehouse_id
        return super(StockPicking, self)._get_warehouse(subcontract_move)

    def _action_done(self):
        res = super()._action_done()
        # If needed, create a compensation layer, so we add the MO cost to the dropship one
        svls = self.env['stock.valuation.layer']
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
                'stock_move_id': move.id,
            })
            svls |= self.env['stock.valuation.layer'].create(svl_vals)
        svls._validate_accounting_entries()

        return res

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

## File: models\stock_rule.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class StockRule(models.Model):
    _inherit = 'stock.rule'

    def _prepare_purchase_order(self, company_id, origins, values):
        if 'partner_id' not in values[0] and company_id.subcontracting_location_id.parent_path in self.location_id.parent_path:
            values[0]['partner_id'] = values[0]['group_id'].partner_id.id
        return super()._prepare_purchase_order(company_id, origins, values)

```

## File: models\stock_warehouse.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _


class StockWarehouse(models.Model):
    _inherit = 'stock.warehouse'

    subcontracting_dropshipping_to_resupply = fields.Boolean(
        'Dropship Subcontractors', default=True,
        help="Dropship subcontractors with components")

    subcontracting_dropshipping_pull_id = fields.Many2one(
        'stock.rule', 'Subcontracting-Dropshipping MTS Rule'
    )

    def _get_global_route_rules_values(self):
        rules = super()._get_global_route_rules_values()
        subcontract_location_id = self._get_subcontracting_location()
        production_location_id = self._get_production_location()
        rules.update({
            'subcontracting_dropshipping_pull_id': {
                'depends': ['subcontracting_dropshipping_to_resupply'],
                'create_values': {
                    'procure_method': 'make_to_order',
                    'company_id': self.company_id.id,
                    'action': 'pull',
                    'auto': 'manual',
                    'route_id': self._find_global_route('mrp_subcontracting_dropshipping.route_subcontracting_dropshipping',
                                                        _('Dropship Subcontractor on Order')).id,
                    'name': self._format_rulename(subcontract_location_id, production_location_id, False),
                    'location_id': production_location_id.id,
                    'location_src_id': subcontract_location_id.id,
                    'picking_type_id': self.subcontracting_type_id.id
                },
                'update_values': {
                    'active': self.subcontracting_dropshipping_to_resupply
                }
            },
        })
        return rules

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import stock_move
from . import stock_picking
from . import res_company
from . import stock_warehouse
from . import purchase
from . import stock_rule

```

## File: views\stock_warehouse_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_warehouse_inherit_mrp_subcontracting_dropshipping" model="ir.ui.view">
        <field name="name">Stock Warehouse Inherit Subcontracting Dropshipping</field>
        <field name="model">stock.warehouse</field>
        <field name="inherit_id" ref="mrp_subcontracting.view_warehouse_inherit_mrp_subcontracting"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='subcontracting_to_resupply']" position="before">
                <field name="subcontracting_dropshipping_to_resupply" />
            </xpath>
        </field>
    </record>
</odoo>

```

