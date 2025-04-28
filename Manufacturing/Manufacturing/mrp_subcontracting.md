# Odoo Module: mrp_subcontracting

Category: Manufacturing/Manufacturing

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-

from odoo import SUPERUSER_ID, api

from . import models
from . import wizard


def uninstall_hook(cr, registry):
    env = api.Environment(cr, SUPERUSER_ID, {})
    warehouses = env["stock.warehouse"].search([])
    subcontracting_routes = warehouses.mapped("subcontracting_route_id")
    warehouses.write({"subcontracting_route_id": False})
    # Fail unlink means that the route is used somewhere (e.g. route_id on stock.rule). In this case
    # we don't try to do anything.
    try:
        with env.cr.savepoint():
            subcontracting_routes.unlink()
    except:
        pass

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': "mrp_subcontracting",
    'version': '0.1',
    'summary': "Subcontract Productions",
    'description': "",
    'website': 'https://www.odoo.com/page/manufacturing',
    'category': 'Manufacturing/Manufacturing',
    'depends': ['mrp'],
    'data': [
        'data/mrp_subcontracting_data.xml',
        'views/mrp_bom_views.xml',
        'views/res_partner_views.xml',
        'views/stock_warehouse_views.xml',
        'views/stock_move_views.xml',
        'views/stock_picking_views.xml',
        'views/supplier_info_views.xml',
    ],
    'demo': [
        'data/mrp_subcontracting_demo.xml',
    ],
    'uninstall_hook': 'uninstall_hook',
    'license': 'LGPL-3',
}

```

## File: data\mrp_subcontracting_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="route_resupply_subcontractor_mto" model='stock.location.route'>
            <field name="name">Resupply Subcontractor on Order</field>
            <field name="company_id"></field>
            <field name="sequence">5</field>
        </record>
        <function model="res.company" name="create_missing_subcontracting_location" />
        <function model="stock.warehouse" name="write">
            <value model="stock.warehouse" eval="obj().env['stock.warehouse'].search([]).ids"/>
            <value eval="{'subcontracting_to_resupply': True}"/>
        </function>
    </data>
</odoo>


```

## File: data\mrp_subcontracting_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="mrp_bom_subcontract" model="mrp.bom">
            <field name="product_tmpl_id" ref="product.product_delivery_02"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="type">subcontract</field>
            <field name="subcontractor_ids" eval="[(4, ref('base.res_partner_12'))]"/>
        </record>

        <record id="mrp_bom_line_subcontract" model="mrp.bom.line">
            <field name="product_id" ref="mrp.product_product_computer_desk_screw"/>
            <field name="product_qty">1</field>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="bom_id" ref="mrp_bom_subcontract"/>
        </record>

        <record id="product_supplierinfo_subcontracting" model="product.supplierinfo">
            <field name="product_tmpl_id" ref="product.product_delivery_02_product_template"/>
            <field name="name" ref="base.res_partner_12"/>
            <field name="sequence">1</field>
            <field name="delay">1</field>
            <field name="min_qty">1</field>
            <field name="price">27</field>
        </record>

        <record id="product.product_delivery_02" model="product.product">
            <field name="route_ids" eval="[(4,ref('stock.route_warehouse0_mto'))]"></field>
        </record>

    </data>
</odoo>
```

## File: models\mrp_bom.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models
from odoo.osv.expression import AND

class MrpBom(models.Model):
    _inherit = 'mrp.bom'

    type = fields.Selection(selection_add=[('subcontract', 'Subcontracting')])
    subcontractor_ids = fields.Many2many('res.partner', 'mrp_bom_subcontractor', string='Subcontractors', check_company=True)

    def _bom_subcontract_find(self, product_tmpl=None, product=None, picking_type=None, company_id=False, bom_type='subcontract', subcontractor=False):
        domain = self._bom_find_domain(product_tmpl=product_tmpl, product=product, picking_type=picking_type, company_id=company_id, bom_type=bom_type)
        if subcontractor:
            domain = AND([domain, [('subcontractor_ids', 'parent_of', subcontractor.ids)]])
            return self.search(domain, order='sequence, product_id', limit=1)
        else:
            return self.env['mrp.bom']

```

## File: models\product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class SupplierInfo(models.Model):
    _inherit = 'product.supplierinfo'

    is_subcontractor = fields.Boolean('Subcontracted', compute='_compute_is_subcontractor', help="Choose a vendor of type subcontractor if you want to subcontract the product")

    @api.depends('name', 'product_id', 'product_tmpl_id')
    def _compute_is_subcontractor(self):
        for supplier in self:
            boms = supplier.product_id.variant_bom_ids
            boms |= supplier.product_tmpl_id.bom_ids.filtered(lambda b: not b.product_id)
            supplier.is_subcontractor = supplier.name in boms.subcontractor_ids

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _


class ResCompany(models.Model):
    _inherit = 'res.company'

    subcontracting_location_id = fields.Many2one('stock.location')

    @api.model
    def create_missing_subcontracting_location(self):
        company_without_subcontracting_loc = self.env['res.company'].search(
            [('subcontracting_location_id', '=', False)])
        company_without_subcontracting_loc._create_subcontracting_location()

    def _create_per_company_locations(self):
        super(ResCompany, self)._create_per_company_locations()
        self._create_subcontracting_location()

    def _create_subcontracting_location(self):
        parent_location = self.env.ref('stock.stock_location_locations_partner', raise_if_not_found=False)
        property_stock_subcontractor_res_partner_field = self.env['ir.model.fields'].search([
            ('model', '=', 'res.partner'),
            ('name', '=', 'property_stock_subcontractor')
        ])
        for company in self:
            subcontracting_location = self.env['stock.location'].create({
                'name': _('%s: Subcontracting Location') % company.name,
                'usage': 'internal',
                'location_id': parent_location.id,
                'company_id': company.id,
            })
            self.env['ir.property'].create({
                'name': 'property_stock_subcontractor_%s' % company.name,
                'fields_id': property_stock_subcontractor_res_partner_field.id,
                'company_id': company.id,
                'value': 'stock.location,%d' % subcontracting_location.id,
            })
            company.subcontracting_location_id = subcontracting_location

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResPartner(models.Model):
    _inherit = 'res.partner'

    property_stock_subcontractor = fields.Many2one(
        'stock.location', string="Subcontractor Location", company_dependent=True,
        help="The stock location used as source and destination when sending\
        goods to this contact during a subcontracting process.")

```

## File: models\stock_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict

from odoo import fields, models, _
from odoo.exceptions import UserError
from odoo.tools.float_utils import float_compare, float_is_zero


class StockMove(models.Model):
    _inherit = 'stock.move'

    is_subcontract = fields.Boolean('The move is a subcontract receipt')
    show_subcontracting_details_visible = fields.Boolean(
        compute='_compute_show_subcontracting_details_visible'
    )

    def _compute_show_subcontracting_details_visible(self):
        """ Compute if the action button in order to see moves raw is visible """
        for move in self:
            if move.is_subcontract and move._has_tracked_subcontract_components() and\
                    not float_is_zero(move.quantity_done, precision_rounding=move.product_uom.rounding):
                move.show_subcontracting_details_visible = True
            else:
                move.show_subcontracting_details_visible = False

    def _compute_show_details_visible(self):
        """ If the move is subcontract and the components are tracked. Then the
        show details button is visible.
        """
        res = super(StockMove, self)._compute_show_details_visible()
        for move in self:
            if not move.is_subcontract:
                continue
            if not move._has_tracked_subcontract_components():
                continue
            move.show_details_visible = True
        return res

    def copy(self, default=None):
        self.ensure_one()
        if not self.is_subcontract or 'location_id' in default:
            return super(StockMove, self).copy(default=default)
        if not default:
            default = {}
        default['location_id'] = self.picking_id.location_id.id
        return super(StockMove, self).copy(default=default)

    def write(self, values):
        """ If the initial demand is updated then also update the linked
        subcontract order to the new quantity.
        """
        if 'product_uom_qty' in values:
            if self.env.context.get('cancel_backorder') is False:
                return super(StockMove, self).write(values)
            self.filtered(lambda m: m.is_subcontract and
            m.state not in ['draft', 'cancel', 'done'])._update_subcontract_order_qty(values['product_uom_qty'])
        return super(StockMove, self).write(values)

    def action_show_details(self):
        """ Open the produce wizard in order to register tracked components for
        subcontracted product. Otherwise use standard behavior.
        """
        self.ensure_one()
        if self.is_subcontract:
            rounding = self.product_uom.rounding
            production = self.move_orig_ids.production_id
            if self._has_tracked_subcontract_components() and\
                    float_compare(production.qty_produced, production.product_uom_qty, precision_rounding=rounding) < 0 and\
                    float_compare(self.quantity_done, self.product_uom_qty, precision_rounding=rounding) < 0:
                return self._action_record_components()
        action = super(StockMove, self).action_show_details()
        if self.is_subcontract and self._has_tracked_subcontract_components():
            action['views'] = [(self.env.ref('stock.view_stock_move_operations').id, 'form')]
            action['context'].update({
                'show_lots_m2o': self.has_tracking != 'none',
                'show_lots_text': False,
            })
        return action

    def action_show_subcontract_details(self):
        """ Display moves raw for subcontracted product self. """
        moves = self.move_orig_ids.production_id.move_raw_ids
        tree_view = self.env.ref('mrp_subcontracting.mrp_subcontracting_move_tree_view')
        form_view = self.env.ref('mrp_subcontracting.mrp_subcontracting_move_form_view')
        return {
            'name': _('Raw Materials for %s') % (self.product_id.display_name),
            'type': 'ir.actions.act_window',
            'res_model': 'stock.move',
            'views': [(tree_view.id, 'tree'), (form_view.id, 'form')],
            'target': 'current',
            'domain': [('id', 'in', moves.ids)],
        }

    def _action_cancel(self):
        for move in self:
            if move.is_subcontract:
                production = move.move_orig_ids.production_id
                moves = self.env.context.get('moves_todo')
                if not moves or production not in moves.move_orig_ids.production_id:
                    production._action_cancel()
        return super()._action_cancel()

    def _action_confirm(self, merge=True, merge_into=False):
        subcontract_details_per_picking = defaultdict(list)
        move_to_not_merge = self.env['stock.move']
        for move in self:
            if move.location_id.usage != 'supplier' or move.location_dest_id.usage == 'supplier':
                continue
            if move.move_orig_ids.production_id:
                continue
            bom = move._get_subcontract_bom()
            if not bom:
                continue
            if float_is_zero(move.product_qty, precision_rounding=move.product_uom.rounding) and\
                    move.picking_id.immediate_transfer is True:
                raise UserError(_("To subcontract, use a planned transfer."))
            subcontract_details_per_picking[move.picking_id].append((move, bom))
            move.write({
                'is_subcontract': True,
                'location_id': move.picking_id.partner_id.with_context(force_company=move.company_id.id).property_stock_subcontractor.id
            })
            move_to_not_merge |= move
        for picking, subcontract_details in subcontract_details_per_picking.items():
            picking._subcontracted_produce(subcontract_details)

        # We avoid merging move due to complication with stock.rule.
        res = super(StockMove, move_to_not_merge)._action_confirm(merge=False)
        res |= super(StockMove, self - move_to_not_merge)._action_confirm(merge=merge, merge_into=merge_into)
        if subcontract_details_per_picking:
            self.env['stock.picking'].concat(*list(subcontract_details_per_picking.keys())).action_assign()
        return res

    def _action_record_components(self):
        action = self.env.ref('mrp.act_mrp_product_produce').read()[0]
        action['context'] = dict(
            default_production_id=self.move_orig_ids.production_id.id,
            default_subcontract_move_id=self.id
        )
        return action

    def _check_overprocessed_subcontract_qty(self):
        """ If a subcontracted move use tracked components. Do not allow to add
        quantity without the produce wizard. Instead update the initial demand
        and use the register component button. Split or correct a lot/sn is
        possible.
        """
        overprocessed_moves = self.env['stock.move']
        for move in self:
            if not move.is_subcontract:
                continue
            # Extra quantity is allowed when components do not need to be register
            if not move._has_tracked_subcontract_components():
                continue
            rounding = move.product_uom.rounding
            if float_compare(move.quantity_done, move.move_orig_ids.production_id.qty_produced, precision_rounding=rounding) > 0:
                overprocessed_moves |= move
        if overprocessed_moves:
            raise UserError(_("""
You have to use 'Records Components' button in order to register quantity for a
subcontracted product(s) with tracked component(s):
 %s.
If you want to process more than initially planned, you
can use the edit + unlock buttons in order to adapt the initial demand on the
operations.""") % ('\n'.join(overprocessed_moves.mapped('product_id.display_name'))))

    def _get_subcontract_bom(self):
        self.ensure_one()
        bom = self.env['mrp.bom'].sudo()._bom_subcontract_find(
            product=self.product_id,
            picking_type=self.picking_type_id,
            company_id=self.company_id.id,
            bom_type='subcontract',
            subcontractor=self.picking_id.partner_id,
        )
        return bom

    def _has_tracked_subcontract_components(self):
        self.ensure_one()
        return any(m.has_tracking != 'none' for m in self.move_orig_ids.production_id.move_raw_ids)

    def _prepare_extra_move_vals(self, qty):
        vals = super(StockMove, self)._prepare_extra_move_vals(qty)
        vals['location_id'] = self.location_id.id
        return vals

    def _prepare_move_split_vals(self, qty):
        vals = super(StockMove, self)._prepare_move_split_vals(qty)
        vals['location_id'] = self.location_id.id
        return vals

    def _should_bypass_reservation(self):
        """ If the move is subcontracted then ignore the reservation. """
        should_bypass_reservation = super(StockMove, self)._should_bypass_reservation()
        if not should_bypass_reservation and self.is_subcontract:
            return True
        return should_bypass_reservation

    def _update_subcontract_order_qty(self, quantity):
        for move in self:
            quantity_change = quantity - move.product_uom_qty
            production = move.move_orig_ids.production_id.filtered(lambda p: p.state != 'cancel')
            if production:
                self.env['change.production.qty'].with_context(skip_activity=True).create({
                    'mo_id': production.id,
                    'product_qty': production.product_uom_qty + quantity_change
                }).change_prod_qty()


```

## File: models\stock_move_line.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class StockMoveLine(models.Model):
    _inherit = 'stock.move.line'

    @api.model_create_multi
    def create(self, vals_list):
        records = super(StockMoveLine, self).create(vals_list)
        records.filtered(lambda ml: ml.move_id.is_subcontract).move_id._check_overprocessed_subcontract_qty()
        return records

    def write(self, values):
        res = super(StockMoveLine, self).write(values)
        self.filtered(lambda ml: ml.move_id.is_subcontract).move_id._check_overprocessed_subcontract_qty()
        return res

    def _should_bypass_reservation(self, location):
        """ If the move line is subcontracted then ignore the reservation. """
        should_bypass_reservation = super(StockMoveLine, self)._should_bypass_reservation(location)
        if not should_bypass_reservation and self.move_id.is_subcontract:
            return True
        return should_bypass_reservation

```

## File: models\stock_picking.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import timedelta

from odoo import api, fields, models


class StockPicking(models.Model):
    _inherit = 'stock.picking'

    display_action_record_components = fields.Boolean(compute='_compute_display_action_record_components')

    @api.depends('state')
    def _compute_display_action_record_components(self):
        for picking in self:
            # Hide if not encoding state
            if picking.state in ('draft', 'cancel', 'done'):
                picking.display_action_record_components = False
                continue
            if not picking._is_subcontract():
                picking.display_action_record_components = False
                continue
            # Hide if no components are track
            subcontracted_productions = picking._get_subcontracted_productions()
            subcontracted_moves = subcontracted_productions.mapped('move_raw_ids')
            if all(subcontracted_move.has_tracking == 'none' for subcontracted_move in subcontracted_moves):
                picking.display_action_record_components = False
                continue
            # Hide if the production is to close
            if not subcontracted_productions.filtered(lambda mo: mo.state not in ('to_close', 'done')):
                picking.display_action_record_components = False
                continue
            picking.display_action_record_components = True

    # -------------------------------------------------------------------------
    # Action methods
    # -------------------------------------------------------------------------

    def action_done(self):
        res = super(StockPicking, self).action_done()
        productions = self.env['mrp.production']
        for picking in self:
            for move in picking.move_lines:
                if not move.is_subcontract:
                    continue
                production = move.move_orig_ids.production_id
                if move._has_tracked_subcontract_components():
                    move.move_orig_ids.filtered(lambda m: m.state not in ('done', 'cancel')).move_line_ids.unlink()
                    move_finished_ids = move.move_orig_ids.filtered(lambda m: m.state not in ('done', 'cancel'))
                    for ml in move.move_line_ids:
                        ml.copy({
                            'picking_id': False,
                            'production_id': move_finished_ids.production_id.id,
                            'move_id': move_finished_ids.id,
                            'qty_done': ml.qty_done,
                            'result_package_id': False,
                            'location_id': move_finished_ids.location_id.id,
                            'location_dest_id': move_finished_ids.location_dest_id.id,
                        })
                else:
                    wizards_vals = []
                    for move_line in move.move_line_ids:
                        wizards_vals.append({
                            'production_id': production.id,
                            'qty_producing': move_line.qty_done,
                            'product_uom_id': move_line.product_uom_id.id,
                            'finished_lot_id': move_line.lot_id.id,
                            'consumption': 'strict',
                        })
                    wizards = self.env['mrp.product.produce'].with_context(default_production_id=production.id).create(wizards_vals)
                    wizards._generate_produce_lines()
                    wizards._record_production()
                productions |= production
            for subcontracted_production in productions:
                if subcontracted_production.state == 'progress':
                    subcontracted_production.post_inventory()
                else:
                    subcontracted_production.button_mark_done()
                # For concistency, set the date on production move before the date
                # on picking. (Tracability report + Product Moves menu item)
                minimum_date = min(picking.move_line_ids.mapped('date'))
                production_moves = subcontracted_production.move_raw_ids | subcontracted_production.move_finished_ids
                production_moves.write({'date': minimum_date - timedelta(seconds=1)})
                production_moves.move_line_ids.write({'date': minimum_date - timedelta(seconds=1)})
        return res

    def action_record_components(self):
        self.ensure_one()
        for move in self.move_lines:
            if not move._has_tracked_subcontract_components():
                continue
            production = move.move_orig_ids.production_id
            if not production or production.state in ('done', 'to_close'):
                continue
            return move._action_record_components()

    # -------------------------------------------------------------------------
    # Subcontract helpers
    # -------------------------------------------------------------------------
    def _is_subcontract(self):
        self.ensure_one()
        return self.picking_type_id.code == 'incoming' and any(m.is_subcontract for m in self.move_lines)

    def _get_subcontracted_productions(self):
        self.ensure_one()
        return self.move_lines.mapped('move_orig_ids.production_id')

    def _get_warehouse(self, subcontract_move):
        return subcontract_move.warehouse_id or self.picking_type_id.warehouse_id

    def _prepare_subcontract_mo_vals(self, subcontract_move, bom):
        subcontract_move.ensure_one()
        group = self.env['procurement.group'].create({
            'name': self.name,
            'partner_id': self.partner_id.id,
        })
        product = subcontract_move.product_id
        warehouse = self._get_warehouse(subcontract_move)
        vals = {
            'company_id': subcontract_move.company_id.id,
            'procurement_group_id': group.id,
            'product_id': product.id,
            'product_uom_id': subcontract_move.product_uom.id,
            'bom_id': bom.id,
            'location_src_id': subcontract_move.picking_id.partner_id.with_context(force_company=subcontract_move.company_id.id).property_stock_subcontractor.id,
            'location_dest_id': subcontract_move.picking_id.partner_id.with_context(force_company=subcontract_move.company_id.id).property_stock_subcontractor.id,
            'product_qty': subcontract_move.product_uom_qty,
            'picking_type_id': warehouse.subcontracting_type_id.id
        }
        return vals

    def _subcontracted_produce(self, subcontract_details):
        self.ensure_one()
        for move, bom in subcontract_details:
            mo = self.env['mrp.production'].with_context(force_company=move.company_id.id).create(self._prepare_subcontract_mo_vals(move, bom))
            self.env['stock.move'].create(mo._get_moves_raw_values())
            mo.action_confirm()

            # Link the finished to the receipt move.
            finished_move = mo.move_finished_ids.filtered(lambda m: m.product_id == move.product_id)
            finished_move.write({'move_dest_ids': [(4, move.id, False)]})
            mo.action_assign()

```

## File: models\stock_rule.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class StockRule(models.Model):
    _inherit = "stock.rule"

    def _push_prepare_move_copy_values(self, move_to_copy, new_date):
        new_move_vals = super(StockRule, self)._push_prepare_move_copy_values(move_to_copy, new_date)
        new_move_vals["is_subcontract"] = False
        return new_move_vals

```

## File: models\stock_warehouse.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _


class StockWarehouse(models.Model):
    _inherit = 'stock.warehouse'

    subcontracting_to_resupply = fields.Boolean(
        'Resupply Subcontractors', default=True,
        help="Resupply subcontractors with components")

    subcontracting_mto_pull_id = fields.Many2one(
        'stock.rule', 'Subcontracting MTO Rule')
    subcontracting_pull_id = fields.Many2one(
        'stock.rule', 'Subcontracting MTS Rule'
    )

    subcontracting_route_id = fields.Many2one('stock.location.route', 'Resupply Subcontractor', ondelete='restrict')

    subcontracting_type_id = fields.Many2one(
        'stock.picking.type', 'Subcontracting Operation Type',
        domain=[('code', '=', 'mrp_operation')])

    def get_rules_dict(self):
        result = super(StockWarehouse, self).get_rules_dict()
        subcontract_location_id = self._get_subcontracting_location()
        for warehouse in self:
            result[warehouse.id].update({
                'subcontract': [
                    self.Routing(warehouse.lot_stock_id, subcontract_location_id, warehouse.out_type_id, 'pull'),
                ]
            })
        return result

    def _get_routes_values(self):
        routes = super(StockWarehouse, self)._get_routes_values()
        routes.update({
            'subcontracting_route_id': {
                'routing_key': 'subcontract',
                'depends': ['subcontracting_to_resupply'],
                'route_create_values': {
                    'product_categ_selectable': False,
                    'warehouse_selectable': True,
                    'product_selectable': False,
                    'company_id': self.company_id.id,
                    'sequence': 10,
                    'name': self._format_routename(name=_('Resupply Subcontractor'))
                },
                'route_update_values': {
                    'active': self.subcontracting_to_resupply,
                },
                'rules_values': {
                    'active': self.subcontracting_to_resupply,
                }
            }
        })
        return routes

    def _get_global_route_rules_values(self):
        rules = super(StockWarehouse, self)._get_global_route_rules_values()
        subcontract_location_id = self._get_subcontracting_location()
        production_location_id = self._get_production_location()
        rules.update({
            'subcontracting_mto_pull_id': {
                'depends': ['subcontracting_to_resupply'],
                'create_values': {
                    'procure_method': 'make_to_order',
                    'company_id': self.company_id.id,
                    'action': 'pull',
                    'auto': 'manual',
                    'route_id': self._find_global_route('stock.route_warehouse0_mto', _('Make To Order')).id,
                    'name': self._format_rulename(self.lot_stock_id, subcontract_location_id, 'MTO'),
                    'location_id': subcontract_location_id.id,
                    'location_src_id': self.lot_stock_id.id,
                    'picking_type_id': self.out_type_id.id
                },
                'update_values': {
                    'active': self.subcontracting_to_resupply
                }
            },
            'subcontracting_pull_id': {
                'depends': ['subcontracting_to_resupply'],
                'create_values': {
                    'procure_method': 'make_to_order',
                    'company_id': self.company_id.id,
                    'action': 'pull',
                    'auto': 'manual',
                    'route_id': self._find_global_route('mrp_subcontracting.route_resupply_subcontractor_mto',
                                                        _('Resupply Subcontractor on Order')).id,
                    'name': self._format_rulename(self.lot_stock_id, subcontract_location_id, False),
                    'location_id': production_location_id.id,
                    'location_src_id': subcontract_location_id.id,
                    'picking_type_id': self.out_type_id.id
                },
                'update_values': {
                    'active': self.subcontracting_to_resupply
                }
            },
        })
        return rules

    def _get_picking_type_create_values(self, max_sequence):
        data, next_sequence = super(StockWarehouse, self)._get_picking_type_create_values(max_sequence)
        data.update({
            'subcontracting_type_id': {
                'name': _('Subcontracting'),
                'code': 'mrp_operation',
                'use_create_components_lots': True,
                'sequence': next_sequence + 2,
                'sequence_code': 'SBC',
                'company_id': self.company_id.id,
            },
        })
        return data, max_sequence + 4

    def _get_sequence_values(self):
        values = super(StockWarehouse, self)._get_sequence_values()
        values.update({
            'subcontracting_type_id': {'name': self.name + ' ' + _('Sequence subcontracting'), 'prefix': self.code + '/SBC/', 'padding': 5, 'company_id': self.company_id.id},
        })
        return values

    def _get_picking_type_update_values(self):
        data = super(StockWarehouse, self)._get_picking_type_update_values()
        subcontract_location_id = self._get_subcontracting_location()
        production_location_id = self._get_production_location()
        data.update({
            'subcontracting_type_id': {
                'active': False,
                'default_location_src_id': subcontract_location_id.id,
                'default_location_dest_id': production_location_id.id,
            },
        })
        return data

    def _get_subcontracting_location(self):
        return self.company_id.subcontracting_location_id

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import mrp_bom
from . import product
from . import res_company
from . import res_partner
from . import stock_move
from . import stock_move_line
from . import stock_picking
from . import stock_rule
from . import stock_warehouse


```

## File: views\mrp_bom_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="mrp_bom_form_view" model="ir.ui.view">
        <field name="name">mrp.bom.form.view</field>
        <field name="model">mrp.bom</field>
        <field name="inherit_id" ref="mrp.mrp_bom_form_view" />
        <field name="arch" type="xml">
            <xpath expr="//field[@name='type']" position="after">
                <field name="subcontractor_ids" widget="many2many_tags" attrs="{'invisible': [('type', '!=', 'subcontract')], 'required': [('type', '=', 'subcontract')]}"/>
            </xpath>
        </field>
    </record>
</odoo>


```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_partner_mrp_subcontracting_form" model="ir.ui.view">
        <field name="name">res.partner.mrp_subcontracting.property.form.inherit</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="stock.view_partner_stock_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='property_stock_supplier']" position="after">
                <field name="property_stock_subcontractor"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\stock_move_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="mrp_subcontracting_move_form_view" model="ir.ui.view">
        <field name="name">mrp.subcontracting.move.form.view</field>
        <field name="model">stock.move</field>
        <field name="priority">1000</field>
        <field name="arch" type="xml">
            <form create="0" delete="0">
                <sheet>
                    <field name="company_id" invisible="1"/>
                    <field name="product_id" invisible="1"/>
                    <field name="finished_lots_exist" invisible="1"/>
                    <field name="state" invisible="1"/>
                    <field name="move_line_ids" context="{'default_product_id': product_id}" attrs="{'readonly': [('state', 'in', ['done', 'cancel'])]}">
                        <tree editable="bottom" decoration-muted="state in ('done', 'cancel')">
                            <field name="company_id" invisible="1"/>
                            <field name="state" invisible="1"/>
                            <field name="tracking" invisible="1"/>
                            <field name="product_id" readonly="1"/>
                            <field name="lot_produced_ids"
                                widget="many2many_tags"
                                context="{'default_product_id': parent.product_id}"
                                attrs="{'column_invisible': [('parent.finished_lots_exist', '!=', True)]}"
                            />
                            <field name="qty_done"/>
                            <field name="lot_id" context="{'default_product_id': product_id}"/>
                        </tree>
                    </field>
                </sheet>
            </form>
        </field>
    </record>
    <record id="mrp_subcontracting_move_tree_view" model="ir.ui.view">
        <field name="name">mrp.subcontracting.move.tree.view</field>
        <field name="model">stock.move</field>
        <field name="priority">1000</field>
        <field name="mode">primary</field>
        <field name="inherit_id" ref="mrp.view_stock_move_raw_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//tree" position="attributes">
                <attribute name="create">0</attribute>
                <attribute name="delete">0</attribute>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\stock_picking_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="stock_picking_form_view" model="ir.ui.view">
        <field name="name">stock.picking.form.view</field>
        <field name="model">stock.picking</field>
        <field name="inherit_id" ref="stock.view_picking_form" />
        <field name="arch" type="xml">
            <xpath expr="//button[@name='button_validate'][hasclass('o_btn_validate')]" position="before">
                <field name="display_action_record_components" invisible="1" />
                <button name="action_record_components" class="oe_highlight" attrs="{'invisible': [('display_action_record_components', '=', False)]}" string="Record components" type="object"/>
            </xpath>
            <xpath expr="//field[@name='move_ids_without_package']//tree//button[@name='action_show_details']" position="after">
                <field name="show_subcontracting_details_visible" invisible="1"/>
                <button name="action_show_subcontract_details" string="Register components for subcontracted product" type="object" icon="fa-sitemap"
                    width="0.1" attrs="{'invisible': [('show_subcontracting_details_visible', '=', False)]}"/>
            </xpath>
        </field>
    </record>
</odoo>


```

## File: views\stock_warehouse_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_warehouse_inherit_mrp_subcontracting" model="ir.ui.view">
        <field name="name">Stock Warehouse Inherit Subcontracting</field>
        <field name="model">stock.warehouse</field>
        <field name="inherit_id" ref="mrp.view_warehouse_inherit_mrp"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='manufacture_to_resupply']" position="before">
                <field name="subcontracting_to_resupply" />
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\supplier_info_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="product_supplierinfo_subcontractor_tree_view" model="ir.ui.view">
        <field name="name">product.supplierinfo.subcontractor.tree.view</field>
        <field name="model">product.supplierinfo</field>
        <field name="inherit_id" ref="product.product_supplierinfo_tree_view"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='name']" position="after">
                <field name="is_subcontractor" readonly="1"/>
            </xpath>
        </field>
    </record>
</odoo>


```

## File: wizard\mrp_product_produce.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models
from odoo.tools.float_utils import float_is_zero

class MrpProductProduce(models.TransientModel):
    _inherit = 'mrp.product.produce'

    subcontract_move_id = fields.Many2one('stock.move', 'stock move from the subcontract picking', check_company=True)

    def continue_production(self):
        action = super(MrpProductProduce, self).continue_production()
        action['context'] = dict(action['context'], default_subcontract_move_id=self.subcontract_move_id.id)
        return action

    def _generate_produce_lines(self):
        """ When the wizard is called in backend, the onchange that create the
        produce lines is not trigger. This method generate them and is used with
        _record_production to appropriately set the lot_produced_id and
        appropriately create raw stock move lines.
        """
        line_values = []
        for wizard in self:
            moves = (wizard.move_raw_ids | wizard.move_finished_ids).filtered(
                lambda move: move.state not in ('done', 'cancel')
            )
            for move in moves:
                qty_producing = wizard.product_uom_id._compute_quantity(wizard.qty_producing, wizard.production_id.product_uom_id)
                qty_to_consume = wizard._prepare_component_quantity(move, qty_producing)
                vals = wizard._generate_lines_values(move, qty_to_consume)
                line_values += vals
        self.env['mrp.product.produce.line'].create(line_values)

    def _update_finished_move(self):
        """ After producing, set the move line on the subcontract picking. """
        res = super(MrpProductProduce, self)._update_finished_move()
        move_line_vals = []
        for wizard in self:
            if wizard.subcontract_move_id:
                move_line_vals.append({
                    'move_id': wizard.subcontract_move_id.id,
                    'picking_id': wizard.subcontract_move_id.picking_id.id,
                    'product_id': wizard.product_id.id,
                    'location_id': wizard.subcontract_move_id.location_id.id,
                    'location_dest_id': wizard.subcontract_move_id.location_dest_id.id,
                    'product_uom_qty': 0,
                    'product_uom_id': wizard.product_uom_id.id,
                    'qty_done': wizard.qty_producing,
                    'lot_id': wizard.finished_lot_id and wizard.finished_lot_id.id,
                })
                if not wizard._get_todo(wizard.production_id):
                    ml_reserved = wizard.subcontract_move_id.move_line_ids.filtered(lambda ml:
                        float_is_zero(ml.qty_done, precision_rounding=ml.product_uom_id.rounding) and
                        not float_is_zero(ml.product_uom_qty, precision_rounding=ml.product_uom_id.rounding))
                    ml_reserved.unlink()
                    for ml in wizard.subcontract_move_id.move_line_ids:
                        ml.product_uom_qty = ml.qty_done
                    wizard.subcontract_move_id._recompute_state()
        self.env['stock.move.line'].create(move_line_vals)
        return res


```

## File: wizard\stock_picking_return.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models
from odoo.osv.expression import OR

class ReturnPicking(models.TransientModel):
    _inherit = 'stock.return.picking'

    @api.onchange('picking_id')
    def _onchange_picking_id(self):
        res = super(ReturnPicking, self)._onchange_picking_id()
        if not any(self.product_return_moves.filtered(lambda r: r.quantity > 0).move_id.mapped('is_subcontract')):
            return res
        subcontract_location = self.picking_id.partner_id.with_context(force_company=self.picking_id.company_id.id).property_stock_subcontractor
        self.location_id = subcontract_location.id
        domain_location = OR([
            ['|', ('id', '=', self.original_location_id.id), ('return_location', '=', True)],
            [('id', '=', subcontract_location.id)]
        ])
        if not res:
            res = {'domain': {'location_id': domain_location}}
        else:
            res['domain'] = {'location_id': domain_location}
        return res

    def _prepare_move_default_values(self, return_line, new_picking):
        vals = super(ReturnPicking, self)._prepare_move_default_values(return_line, new_picking)
        vals['is_subcontract'] = False
        return vals

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import mrp_product_produce
from . import stock_picking_return

```

