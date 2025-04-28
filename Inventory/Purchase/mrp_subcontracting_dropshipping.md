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
        'views/purchase_order_views.xml',
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

        <record id="route_subcontracting_dropshipping" model='stock.route'>
            <field name="name">Dropship Subcontractor on Order</field>
            <field name="sequence">5</field>
            <field name="company_id"></field>
            <field name="product_selectable" eval="True"/>
        </record>

        <function model="res.company" name="_create_missing_subcontracting_dropshipping_sequence"/>
        <function model="res.company" name="_create_missing_subcontracting_dropshipping_picking_type"/>
        <function model="res.company" name="_create_missing_subcontracting_dropshipping_rules"/>

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

from odoo import api, fields, models, _
from odoo.exceptions import UserError


class PurchaseOrder(models.Model):
    _inherit = 'purchase.order'

    default_location_dest_id_is_subcontracting_loc = fields.Boolean(related='picking_type_id.default_location_dest_id.is_subcontracting_location')

    @api.depends('default_location_dest_id_is_subcontracting_loc')
    def _compute_dest_address_id(self):
        dropship_subcontract_pos = self.filtered(lambda po: po.default_location_dest_id_is_subcontracting_loc)
        for order in dropship_subcontract_pos:
            subcontractor_ids = order.picking_type_id.default_location_dest_id.subcontractor_ids
            order.dest_address_id = subcontractor_ids if len(subcontractor_ids) == 1 else False
        super(PurchaseOrder, self - dropship_subcontract_pos)._compute_dest_address_id()

    @api.onchange('picking_type_id')
    def onchange_picking_type_id(self):
        if self.default_location_dest_id_is_subcontracting_loc:
            return {
                'warning': {'title': _('Warning'), 'message': _('Please note this purchase order is for subcontracting purposes.')}
            }

    def _get_destination_location(self):
        self.ensure_one()
        if self.default_location_dest_id_is_subcontracting_loc:
            return self.dest_address_id.property_stock_subcontractor.id
        return super()._get_destination_location()

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ResCompany(models.Model):
    _inherit = 'res.company'

    dropship_subcontractor_pick_type_id = fields.Many2one('stock.picking.type')

    def _create_subcontracting_dropshipping_sequence(self):
        seq_vals = [{
            'name': 'Dropship Subcontractor (%s)' % company.name,
            'code': 'mrp.subcontracting.dropshipping',
            'company_id': company.id,
            'prefix': 'DSC/',
            'padding': 5,
        } for company in self]

        if seq_vals:
            self.env['ir.sequence'].create(seq_vals)

    def _create_subcontracting_dropshipping_picking_type(self):
        pick_type_vals = []
        for company in self:
            sequence = self.env['ir.sequence'].search([
                ('code', '=', 'mrp.subcontracting.dropshipping'),
                ('company_id', '=', company.id),
            ])
            pick_type_vals.append({
                'name': 'Dropship Subcontractor',
                'company_id': company.id,
                'warehouse_id': False,
                'sequence_id': sequence.id,
                'code': 'incoming',
                'default_location_src_id': self.env.ref('stock.stock_location_suppliers').id,
                'default_location_dest_id': company.subcontracting_location_id.id,
                'sequence_code': 'DSC',
                'use_existing_lots': False,
            })
        if pick_type_vals:
            pick_type_ids = self.env['stock.picking.type'].create(pick_type_vals)
            for pick_type in pick_type_ids:
                pick_type.company_id.dropship_subcontractor_pick_type_id = pick_type.id

    def _create_subcontracting_dropshipping_rules(self):
        route = self.env.ref('mrp_subcontracting_dropshipping.route_subcontracting_dropshipping')
        supplier_location = self.env.ref('stock.stock_location_suppliers')
        vals = []
        for company in self:
            subcontracting_location = company.subcontracting_location_id
            dropship_picking_type = self.env['stock.picking.type'].search([
                ('company_id', '=', company.id),
                ('default_location_src_id.usage', '=', 'supplier'),
                ('default_location_dest_id', '=', subcontracting_location.id),
            ], limit=1, order='sequence')
            if dropship_picking_type:
                vals.append({
                    'name': '%s → %s' % (supplier_location.name, subcontracting_location.name),
                    'action': 'buy',
                    'location_dest_id': subcontracting_location.id,
                    'location_src_id': supplier_location.id,
                    'procure_method': 'make_to_stock',
                    'route_id': route.id,
                    'picking_type_id': dropship_picking_type.id,
                    'company_id': company.id,
                })
        if vals:
            self.env['stock.rule'].create(vals)

    @api.model
    def _create_missing_subcontracting_dropshipping_rules(self):
        route = self.env.ref('mrp_subcontracting_dropshipping.route_subcontracting_dropshipping')
        company_ids = self.env['res.company'].search([])
        company_has_rules = self.env['stock.rule'].search([('route_id', '=', route.id)]).mapped('company_id')
        company_todo_rules = company_ids - company_has_rules
        company_todo_rules._create_subcontracting_dropshipping_rules()

    @api.model
    def _create_missing_subcontracting_dropshipping_sequence(self):
        company_ids = self.env['res.company'].search([])
        company_has_seq = self.env['ir.sequence'].search([('code', '=', 'mrp.subcontracting.dropshipping')]).mapped('company_id')
        company_todo_sequence = company_ids - company_has_seq
        company_todo_sequence._create_subcontracting_dropshipping_sequence()

    @api.model
    def _create_missing_subcontracting_dropshipping_picking_type(self):
        company_ids = self.env['res.company'].search([])
        company_has_dropship_subcontractor_picking_type = self.env['stock.picking.type'].search([
            ('default_location_src_id.usage', '=', 'supplier'),
            ('default_location_dest_id', 'in', company_ids.subcontracting_location_id.ids),
        ]).mapped('company_id')
        company_todo_picking_type = company_ids - company_has_dropship_subcontractor_picking_type
        company_todo_picking_type._create_subcontracting_dropshipping_picking_type()

    def _create_per_company_sequences(self):
        super()._create_per_company_sequences()
        self._create_subcontracting_dropshipping_sequence()

    def _create_per_company_rules(self):
        res = super()._create_per_company_rules()
        self._create_subcontracting_dropshipping_rules()
        return res

    def _create_per_company_picking_types(self):
        super()._create_per_company_picking_types()
        self._create_subcontracting_dropshipping_picking_type()

```

## File: models\stock_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class StockMove(models.Model):
    _inherit = "stock.move"

    def _prepare_procurement_values(self):
        vals = super()._prepare_procurement_values()
        partner = self.group_id.partner_id
        if not vals.get('partner_id') and partner and self.location_id.is_subcontracting_location:
            vals['partner_id'] = partner.id
        return vals

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

## File: models\stock_orderpoint.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class StockWarehouseOrderpoint(models.Model):
    _inherit = 'stock.warehouse.orderpoint'

    def _prepare_procurement_values(self, date=False, group=False):
        vals = super()._prepare_procurement_values(date, group)
        if not vals.get('partner_id') and self.location_id.is_subcontracting_location and len(self.location_id.subcontractor_ids) == 1:
            vals['partner_id'] = self.location_id.subcontractor_ids.id
        return vals

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
        dropship_subcontract_pickings = self.filtered(lambda p: p.location_dest_id.is_subcontracting_location and p.location_id.usage == 'supplier')
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
        for move in self.move_ids:
            if not (move.is_subcontract and move._is_dropshipped() and move.state == 'done'):
                continue

            dropship_svls = move.stock_valuation_layer_ids
            if not dropship_svls:
                continue

            # In a backorder chain, only generate SVLs for the latest backorder, or else their
            # value will be cumulative.
            if any(move.move_orig_ids.production_id.mapped('backorder_sequence')):
                moves_with_svls = move.move_orig_ids.filtered('stock_valuation_layer_ids')
                subcontract_svls = max(
                    moves_with_svls,
                    key=lambda sm: sm.production_id.backorder_sequence
                ).stock_valuation_layer_ids
            else:
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
                or subcontract_move.location_dest_id.is_subcontracting_location
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
        if 'partner_id' not in values[0] \
            and (company_id.subcontracting_location_id.parent_path in self.location_dest_id.parent_path
                 or self.location_dest_id.is_subcontracting_location):
            values[0]['partner_id'] = values[0]['group_id'].partner_id.id
        return super()._prepare_purchase_order(company_id, origins, values)

    def _make_po_get_domain(self, company_id, values, partner):
        domain = super()._make_po_get_domain(company_id, values, partner)
        if values.get('partner_id', False):
            domain += (('dest_address_id', '=', values.get('partner_id')),)
        return domain

```

## File: models\stock_warehouse.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _


class StockWarehouse(models.Model):
    _inherit = 'stock.warehouse'

    subcontracting_dropshipping_to_resupply = fields.Boolean(
        'Dropship Subcontractors', default=True,
        help="Dropship subcontractors with components")

    subcontracting_dropshipping_pull_id = fields.Many2one(
        'stock.rule', 'Subcontracting-Dropshipping MTS Rule'
    )

    @api.model_create_multi
    def create(self, vals_list):
        res = super().create(vals_list)
        # if new warehouse has resupply enabled, enable global route
        if any([vals.get('subcontracting_dropshipping_to_resupply', False) for vals in vals_list]):
            res.update_global_route_dropship_subcontractor()
        return res

    def write(self, vals):
        res = super().write(vals)
        # if all warehouses have resupply disabled, disable global route, until its enabled on a warehouse
        if 'subcontracting_dropshipping_to_resupply' in vals or 'active' in vals:
            if 'subcontracting_dropshipping_to_resupply' in vals:
                # ignore when warehouse archived since it will auto-archive all of its rules
                self._update_dropship_subcontract_rules()
            self.update_global_route_dropship_subcontractor()
        return res

    def _update_dropship_subcontract_rules(self):
        '''update (archive/unarchive) any warehouse subcontracting location dropship rules'''
        subcontracting_locations = self._get_subcontracting_locations()
        route_id = self._find_or_create_global_route('mrp_subcontracting_dropshipping.route_subcontracting_dropshipping',
                                           _('Dropship Subcontractor on Order'))
        warehouses_dropship = self.filtered(lambda w: w.subcontracting_dropshipping_to_resupply and w.active)
        if warehouses_dropship:
            self.env['stock.rule'].with_context(active_test=False).search([
                ('route_id', '=', route_id.id),
                ('action', '=', 'pull'),
                ('warehouse_id', 'in', warehouses_dropship.ids),
                ('location_src_id', 'in', subcontracting_locations.ids)]).action_unarchive()

        warehouses_no_dropship = self - warehouses_dropship
        if warehouses_no_dropship:
            self.env['stock.rule'].search([
                ('route_id', '=', route_id.id),
                ('action', '=', 'pull'),
                ('warehouse_id', 'in', warehouses_no_dropship.ids),
                ('location_src_id', 'in', subcontracting_locations.ids)]).action_archive()

    def update_global_route_dropship_subcontractor(self):
        route_id = self._find_or_create_global_route('mrp_subcontracting_dropshipping.route_subcontracting_dropshipping',
                                           _('Dropship Subcontractor on Order'))
        # if route has no pull rules, it means all warehouses have Dropship Subcontractor disabled
        # Pick type is per company so we need to check rules per company to archive it, however
        # the route is global so we need to check all rules regardless of company
        all_rules = route_id.sudo().rule_ids.filtered(lambda r: r.active)
        for company in self.company_id:
            company_rules = all_rules.filtered(lambda r: r.company_id == company)
            company.dropship_subcontractor_pick_type_id.active = bool(company_rules.filtered(lambda r: r.action == 'pull'))

        route_id.active = bool(all_rules.filtered(lambda r: r.action == 'pull'))

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
                    'route_id': self._find_or_create_global_route('mrp_subcontracting_dropshipping.route_subcontracting_dropshipping',
                                                        _('Dropship Subcontractor on Order')).id,
                    'name': self._format_rulename(subcontract_location_id, production_location_id, False),
                    'location_dest_id': production_location_id.id,
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
from . import stock_orderpoint

```

## File: views\purchase_order_views.xml

```xml
<odoo>
    <record id="view_purchase_order_inherit" model="ir.ui.view">
        <field name="name">Purchase Order Inherit Dropship Subcontractor</field>
        <field name="model">purchase.order</field>
        <field name="inherit_id" ref="purchase.purchase_order_form"/>
        <field name="arch" type="xml">
            <field name="dest_address_id" position="before">
                <field name="default_location_dest_id_is_subcontracting_loc" invisible="1"/>
            </field>
            <field name="dest_address_id" position="attributes">
                <attribute name="attrs">{'invisible': [('default_location_dest_id_usage', '!=', 'customer'), ('default_location_dest_id_is_subcontracting_loc', '=', False)],
                    'required': ['|', ('default_location_dest_id_usage', '=', 'customer'), ('default_location_dest_id_is_subcontracting_loc', '=', True)]}</attribute>
            </field>
        </field>
    </record>
</odoo>

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

