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
        'views/product_views.xml',
        'views/mrp_production_views.xml',
        'wizard/stock_picking_return_views.xml',
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
            <field name="currency_id" ref="base.USD"/>
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

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError
from odoo.osv.expression import AND

class MrpBom(models.Model):
    _inherit = 'mrp.bom'

    type = fields.Selection(selection_add=[
        ('subcontract', 'Subcontracting')
    ], ondelete={'subcontract': lambda recs: recs.write({'type': 'normal', 'active': False})})
    subcontractor_ids = fields.Many2many('res.partner', 'mrp_bom_subcontractor', string='Subcontractors', check_company=True)

    def _bom_subcontract_find(self, product_tmpl=None, product=None, picking_type=None, company_id=False, bom_type='subcontract', subcontractor=False):
        domain = self._bom_find_domain(product_tmpl=product_tmpl, product=product, picking_type=picking_type, company_id=company_id, bom_type=bom_type)
        if subcontractor:
            domain = AND([domain, [('subcontractor_ids', 'parent_of', subcontractor.ids)]])
            return self.search(domain, order='sequence, product_id', limit=1)
        else:
            return self.env['mrp.bom']

    @api.constrains('operation_ids', 'type')
    def _check_subcontracting_no_operation(self):
        if self.filtered_domain([('type', '=', 'subcontract'), ('operation_ids', '!=', False)]):
            raise ValidationError(_('You can not set a Bill of Material with operations as subcontracting.'))

```

## File: models\mrp_production.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict
from odoo import fields, models, _, api
from odoo.exceptions import UserError
from odoo.osv import expression
from odoo.tools.float_utils import float_compare, float_is_zero


class MrpProduction(models.Model):
    _inherit = 'mrp.production'

    move_line_raw_ids = fields.One2many(
        'stock.move.line', string="Detail Component", readonly=False,
        inverse='_inverse_move_line_raw_ids', compute='_compute_move_line_raw_ids'
    )
    incoming_picking = fields.Many2one(related='move_finished_ids.move_dest_ids.picking_id')

    @api.depends('name')
    def name_get(self):
        return [
            (record.id, "%s (%s)" % (record.incoming_picking.name, record.name)) if record.bom_id.type == 'subcontract'
            else (record.id, record.name) for record in self
        ]

    @api.model
    def _name_search(self, name='', args=None, operator='ilike', limit=100, name_get_uid=None):
        args = list(args or [])

        if name == '' and operator == 'ilike':
            return self._search(args, limit=limit, access_rights_uid=name_get_uid)

        # search through MO
        domain = [(self._rec_name, operator, name)]

        # search through transfers
        picking_rec_name = self.env['stock.picking']._rec_name
        picking_domain = [('bom_id.type', '=', 'subcontract'), ('incoming_picking.%s' % picking_rec_name, operator, name)]
        domain = expression.OR([domain, picking_domain])

        args = expression.AND([args, domain])
        return self._search(args, limit=limit, access_rights_uid=name_get_uid)

    @api.depends('move_raw_ids.move_line_ids')
    def _compute_move_line_raw_ids(self):
        for production in self:
            production.move_line_raw_ids = production.move_raw_ids.move_line_ids

    def _inverse_move_line_raw_ids(self):
        for production in self:
            line_by_product = defaultdict(lambda: self.env['stock.move.line'])
            for line in production.move_line_raw_ids:
                line_by_product[line.product_id] |= line
            for move in production.move_raw_ids:
                move.move_line_ids = line_by_product.pop(move.product_id, self.env['stock.move.line'])
            for product_id, lines in line_by_product.items():
                qty = sum(line.product_uom_id._compute_quantity(line.qty_done, product_id.uom_id) for line in lines)
                move = production._get_move_raw_values(product_id, qty, product_id.uom_id)
                move['additional'] = True
                production.move_raw_ids = [(0, 0, move)]
                production.move_raw_ids.filtered(lambda m: m.product_id == product_id)[:1].move_line_ids = lines

    def subcontracting_record_component(self):
        self.ensure_one()
        assert self.env.context.get('subcontract_move_id')
        if float_is_zero(self.qty_producing, precision_rounding=self.product_uom_id.rounding):
            return {'type': 'ir.actions.act_window_close'}
        if self.product_tracking != 'none' and not self.lot_producing_id:
            raise UserError(_('You must enter a serial number for %s') % self.product_id.name)
        for sml in self.move_raw_ids.move_line_ids:
            if sml.tracking != 'none' and not sml.lot_id:
                raise UserError(_('You must enter a serial number for each line of %s') % sml.product_id.display_name)
        self._update_finished_move()
        quantity_issues = self._get_quantity_produced_issues()
        if quantity_issues:
            backorder = self._generate_backorder_productions(close_mo=False)
            # No qty to consume to avoid propagate additional move
            # TODO avoid : stock move created in backorder with 0 as qty
            backorder.move_raw_ids.filtered(lambda m: m.additional).product_uom_qty = 0.0

            backorder.qty_producing = backorder.product_qty
            backorder._set_qty_producing()

            self.product_qty = self.qty_producing
            subcontract_move_id = self.env['stock.move'].browse(self.env.context.get('subcontract_move_id'))
            action = subcontract_move_id._action_record_components()
            action.update({'res_id': backorder.id})
            return action
        return {'type': 'ir.actions.act_window_close'}

    def action_subcontracting_discard_remaining_components(self):
        self.ensure_one()
        self.qty_producing = 0
        return {'type': 'ir.actions.act_window_close'}

    def _pre_button_mark_done(self):
        if self.env.context.get('subcontract_move_id'):
            return True
        return super()._pre_button_mark_done()

    def _update_finished_move(self):
        """ After producing, set the move line on the subcontract picking. """
        self.ensure_one()
        subcontract_move_id = self.env.context.get('subcontract_move_id')
        if subcontract_move_id:
            subcontract_move_id = self.env['stock.move'].browse(subcontract_move_id)
            quantity = self.qty_producing
            if self.lot_producing_id:
                move_lines = subcontract_move_id.move_line_ids.filtered(lambda ml: ml.lot_id == self.lot_producing_id or not ml.lot_id)
            else:
                move_lines = subcontract_move_id.move_line_ids.filtered(lambda ml: not ml.lot_id)
            # Update reservation and quantity done
            for ml in move_lines:
                rounding = ml.product_uom_id.rounding
                if float_compare(quantity, 0, precision_rounding=rounding) <= 0:
                    break
                quantity_to_process = min(quantity, ml.product_uom_qty - ml.qty_done)
                quantity -= quantity_to_process

                new_quantity_done = (ml.qty_done + quantity_to_process)

                # on which lot of finished product
                if float_compare(new_quantity_done, ml.product_uom_qty, precision_rounding=rounding) >= 0:
                    ml.write({
                        'qty_done': new_quantity_done,
                        'lot_id': self.lot_producing_id and self.lot_producing_id.id,
                    })
                else:
                    new_qty_reserved = ml.product_uom_qty - new_quantity_done
                    default = {
                        'product_uom_qty': new_quantity_done,
                        'qty_done': new_quantity_done,
                        'lot_id': self.lot_producing_id and self.lot_producing_id.id,
                    }
                    ml.copy(default=default)
                    ml.with_context(bypass_reservation_update=True).write({
                        'product_uom_qty': new_qty_reserved,
                        'qty_done': 0
                    })

            if float_compare(quantity, 0, precision_rounding=self.product_uom_id.rounding) > 0:
                self.env['stock.move.line'].create({
                    'move_id': subcontract_move_id.id,
                    'picking_id': subcontract_move_id.picking_id.id,
                    'product_id': self.product_id.id,
                    'location_id': subcontract_move_id.location_id.id,
                    'location_dest_id': subcontract_move_id.location_dest_id.id,
                    'product_uom_qty': 0,
                    'product_uom_id': self.product_uom_id.id,
                    'qty_done': quantity,
                    'lot_id': self.lot_producing_id and self.lot_producing_id.id,
                })
            if not self._get_quantity_to_backorder():
                ml_reserved = subcontract_move_id.move_line_ids.filtered(lambda ml:
                    float_is_zero(ml.qty_done, precision_rounding=ml.product_uom_id.rounding) and
                    not float_is_zero(ml.product_uom_qty, precision_rounding=ml.product_uom_id.rounding))
                ml_reserved.unlink()
                for ml in subcontract_move_id.move_line_ids:
                    ml.product_uom_qty = ml.qty_done
                subcontract_move_id._recompute_state()

    def _subcontracting_filter_to_done(self):
        """ Filter subcontracting production where composant is already recorded and should be consider to be validate """
        def filter_in(mo):
            if mo.state in ('done', 'cancel'):
                return False
            if float_is_zero(mo.qty_producing, precision_rounding=mo.product_uom_id.rounding):
                return False
            if not all(line.lot_id for line in mo.move_raw_ids.filtered(lambda sm: sm.has_tracking != 'none').move_line_ids):
                return False
            if mo.product_tracking != 'none' and not mo.lot_producing_id:
                return False
            return True

        return self.filtered(filter_in)

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
        parent_location = self.env.ref('stock.stock_location_locations', raise_if_not_found=False)
        for company in self:
            subcontracting_location = self.env['stock.location'].create({
                'name': _('Subcontracting Location'),
                'usage': 'internal',
                'location_id': parent_location.id,
                'company_id': company.id,
            })
            self.env['ir.property']._set_default(
                "property_stock_subcontractor",
                "res.partner",
                subcontracting_location,
                company,
            )
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
    is_subcontractor = fields.Boolean(
        string="Subcontractor", store=False, search="_search_is_subcontractor")

    def _search_is_subcontractor(self, operator, value):
        assert operator in ('=', '!=', '<>') and value in (True, False), 'Operation not supported'
        subcontractor_ids = self.env['mrp.bom'].search(
            [('type', '=', 'subcontract')]).subcontractor_ids.ids
        if (operator == '=' and value is True) or (operator in ('<>', '!=') and value is False):
            search_operator = 'in'
        else:
            search_operator = 'not in'
        return [('id', search_operator, subcontractor_ids)]

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
        if 'product_uom_qty' in values and self.env.context.get('cancel_backorder') is not False:
            self.filtered(lambda m: m.is_subcontract and m.state not in ['draft', 'cancel', 'done'])._update_subcontract_order_qty(values['product_uom_qty'])
        res = super().write(values)
        if 'date' in values:
            for move in self:
                if move.state in ('done', 'cancel') or not move.is_subcontract:
                    continue
                move.move_orig_ids.production_id.filtered(lambda p: p.state not in ('done', 'cancel')).write({
                    'date_planned_finished': move.date,
                    'date_planned_start': move.date,
                })
        return res

    def action_show_details(self):
        """ Open the produce wizard in order to register tracked components for
        subcontracted product. Otherwise use standard behavior.
        """
        self.ensure_one()
        if self._has_components_to_record():
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
        ctx = dict(self._context, search_default_by_product=True, subcontract_move_id=self.id)
        return {
            'name': _('Raw Materials for %s') % (self.product_id.display_name),
            'type': 'ir.actions.act_window',
            'res_model': 'stock.move',
            'views': [(tree_view.id, 'list'), (form_view.id, 'form')],
            'target': 'current',
            'domain': [('id', 'in', moves.ids)],
            'context': ctx
        }

    def _action_cancel(self):
        for move in self:
            if move.is_subcontract:
                active_production = move.move_orig_ids.production_id.filtered(lambda p: p.state not in ('done', 'cancel'))
                moves = self.env.context.get('moves_todo')
                if not moves or active_production not in moves.move_orig_ids.production_id:
                    active_production.with_context(skip_activity=True).action_cancel()
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
                'location_id': move.picking_id.partner_id.with_company(move.company_id).property_stock_subcontractor.id
            })
            move_to_not_merge |= move
        for picking, subcontract_details in subcontract_details_per_picking.items():
            picking._subcontracted_produce(subcontract_details)

        # We avoid merging move due to complication with stock.rule.
        res = super(StockMove, move_to_not_merge)._action_confirm(merge=False)
        res |= super(StockMove, self - move_to_not_merge)._action_confirm(merge=merge, merge_into=merge_into)
        if subcontract_details_per_picking:
            self.env['stock.picking'].concat(*list(subcontract_details_per_picking.keys())).action_assign()
        return res.exists()

    def _action_record_components(self):
        self.ensure_one()
        production = self.move_orig_ids.production_id[-1:]
        view = self.env.ref('mrp_subcontracting.mrp_production_subcontracting_form_view')
        return {
            'name': _('Subcontract'),
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'mrp.production',
            'views': [(view.id, 'form')],
            'view_id': view.id,
            'target': 'new',
            'res_id': production.id,
            'context': dict(self.env.context, subcontract_move_id=self.id),
        }

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

    def _has_components_to_record(self):
        """ Returns true if the move has still some tracked components to record. """
        self.ensure_one()
        if not self.is_subcontract:
            return False
        rounding = self.product_uom.rounding
        production = self.move_orig_ids.production_id[-1:]
        return self._has_tracked_subcontract_components() and\
            float_compare(production.qty_produced, production.product_uom_qty, precision_rounding=rounding) < 0 and\
            float_compare(self.quantity_done, self.product_uom_qty, precision_rounding=rounding) < 0

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

    def _should_bypass_set_qty_producing(self):
        if self.env.context.get('subcontract_move_id'):
            return False
        return super()._should_bypass_set_qty_producing()

    def _should_bypass_reservation(self):
        """ If the move is subcontracted then ignore the reservation. """
        should_bypass_reservation = super(StockMove, self)._should_bypass_reservation()
        if not should_bypass_reservation and self.is_subcontract:
            return True
        return should_bypass_reservation

    def _update_subcontract_order_qty(self, new_quantity):
        for move in self:
            quantity_to_remove = move.product_uom_qty - new_quantity
            productions = move.move_orig_ids.production_id.filtered(lambda p: p.state not in ('done', 'cancel'))[::-1]
            # Cancel productions until reach new_quantity
            for production in productions:
                if quantity_to_remove <= 0.0:
                    break
                if quantity_to_remove >= production.product_qty:
                    quantity_to_remove -= production.product_qty
                    production.with_context(skip_activity=True).action_cancel()
                else:
                    self.env['change.production.qty'].with_context(skip_activity=True).create({
                        'mo_id': production.id,
                        'product_qty': production.product_uom_qty - quantity_to_remove
                    }).change_prod_qty()

```

## File: models\stock_move_line.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields


class StockMoveLine(models.Model):
    _inherit = 'stock.move.line'

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
from odoo.tools.float_utils import float_compare
from dateutil.relativedelta import relativedelta


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
            # Hide if all tracked product move lines are already recorded.
            picking.display_action_record_components = any(
                move._has_components_to_record() for move in picking.move_lines)

    # -------------------------------------------------------------------------
    # Action methods
    # -------------------------------------------------------------------------
    def _action_done(self):
        res = super(StockPicking, self)._action_done()

        for move in self.move_lines.filtered(lambda move: move.is_subcontract):
            # Auto set qty_producing/lot_producing_id of MO if there isn't tracked component
            # If there is tracked component, the flow use subcontracting_record_component instead
            if move._has_tracked_subcontract_components():
                continue
            production = move.move_orig_ids.production_id.filtered(lambda p: p.state not in ('done', 'cancel'))[-1:]
            if not production:
                continue
            # Manage additional quantities
            quantity_done_move = move.product_uom._compute_quantity(move.quantity_done, production.product_uom_id)
            if float_compare(production.product_qty, quantity_done_move, precision_rounding=production.product_uom_id.rounding) == -1:
                change_qty = self.env['change.production.qty'].create({
                    'mo_id': production.id,
                    'product_qty': quantity_done_move
                })
                change_qty.with_context(skip_activity=True).change_prod_qty()
            # Create backorder MO for each move lines
            for move_line in move.move_line_ids:
                if move_line.lot_id:
                    production.lot_producing_id = move_line.lot_id
                production.qty_producing = move_line.product_uom_id._compute_quantity(move_line.qty_done, production.product_uom_id)
                production._set_qty_producing()
                if move_line != move.move_line_ids[-1]:
                    backorder = production._generate_backorder_productions(close_mo=False)
                    # The move_dest_ids won't be set because the _split filter out done move
                    backorder.move_finished_ids.filtered(lambda mo: mo.product_id == move.product_id).move_dest_ids = production.move_finished_ids.filtered(lambda mo: mo.product_id == move.product_id).move_dest_ids
                    production.product_qty = production.qty_producing
                    production = backorder

        for picking in self:
            productions_to_done = picking._get_subcontracted_productions()._subcontracting_filter_to_done()
            if not productions_to_done:
                continue
            production_ids_backorder = []
            if not self.env.context.get('cancel_backorder'):
                production_ids_backorder = productions_to_done.filtered(lambda mo: mo.state == "progress").ids
            productions_to_done.with_context(subcontract_move_id=True, mo_ids_to_backorder=production_ids_backorder).button_mark_done()
            # For concistency, set the date on production move before the date
            # on picking. (Traceability report + Product Moves menu item)
            minimum_date = min(picking.move_line_ids.mapped('date'))
            production_moves = productions_to_done.move_raw_ids | productions_to_done.move_finished_ids
            production_moves.write({'date': minimum_date - timedelta(seconds=1)})
            production_moves.move_line_ids.write({'date': minimum_date - timedelta(seconds=1)})
        return res

    def action_record_components(self):
        self.ensure_one()
        for move in self.move_lines:
            if move._has_components_to_record():
                return move._action_record_components()

    # -------------------------------------------------------------------------
    # Subcontract helpers
    # -------------------------------------------------------------------------
    def _is_subcontract(self):
        self.ensure_one()
        return self.picking_type_id.code == 'incoming' and any(m.is_subcontract for m in self.move_lines)

    def _get_subcontracted_productions(self):
        return self.move_lines.filtered(lambda move: move.is_subcontract).move_orig_ids.production_id

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
            'location_src_id': subcontract_move.picking_id.partner_id.with_company(subcontract_move.company_id).property_stock_subcontractor.id,
            'location_dest_id': subcontract_move.picking_id.partner_id.with_company(subcontract_move.company_id).property_stock_subcontractor.id,
            'product_qty': subcontract_move.product_uom_qty,
            'picking_type_id': warehouse.subcontracting_type_id.id,
            'date_planned_start': subcontract_move.date - relativedelta(days=product.produce_delay)
        }
        return vals

    def _subcontracted_produce(self, subcontract_details):
        self.ensure_one()
        for move, bom in subcontract_details:
            mo = self.env['mrp.production'].with_company(move.company_id).create(self._prepare_subcontract_mo_vals(move, bom))
            self.env['stock.move'].create(mo._get_moves_raw_values())
            self.env['stock.move'].create(mo._get_moves_finished_values())
            mo.date_planned_finished = move.date  # Avoid to have the picking late depending of the MO
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
from . import mrp_production

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

## File: views\mrp_production_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="mrp_production_subcontracting_form_view" model="ir.ui.view">
        <field name="name">mrp.production.subcontracting.form.view</field>
        <field name="model">mrp.production</field>
        <field name="mode">primary</field>
        <field name="priority">1000</field>
        <field name="inherit_id" ref="mrp.mrp_production_form_view" />
        <field name="arch" type="xml">
            <xpath expr="//header" position="replace">
                <field name="state" invisible="1"/>
            </xpath>
            <xpath expr="//div[@name='button_box']" position="replace"/>
            <xpath expr="//group[@name='group_extra_info']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//page[@name='operations']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//field[@name='lot_producing_id']" position="attributes">
                <attribute name="attrs">{'invisible': [('product_tracking', 'in', ('none', False))]}</attribute>
            </xpath>
            <xpath expr="//page[@name='miscellaneous']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//field[@name='name']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//field[@name='priority']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//field[@name='bom_id']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//button[@name='action_generate_serial']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath> 
            <xpath expr="//field[@name='move_raw_ids']" position="attributes">
                <attribute name="invisible">1</attribute>
                <attribute name="readonly">1</attribute>
            </xpath>
            <xpath expr="//field[@name='move_raw_ids']" position="after">
                <field name="move_line_raw_ids" force_save="1"
                    context="{'tree_view_ref': 'mrp_subcontracting.mrp_subcontracting_stock_move_line_tree_view', 'default_company_id': company_id, 'default_location_id': location_src_id, 'default_location_dest_id': production_location_id}"
                    />
            </xpath>
            <xpath expr="//sheet" position="inside">
                <footer>
                    <button name="subcontracting_record_component" attrs="{'invisible': ['|', ('state', '=', 'to_close'), ('qty_producing', '&lt;', 0.001)]}" string="Continue" type="object" class="oe_highlight"/>
                    <button name="subcontracting_record_component" attrs="{'invisible': [('state', '!=', 'to_close')]}" string="Record Production" type="object" class="oe_highlight"/>
                    <button name="action_subcontracting_discard_remaining_components" string="Discard" type="object"/>
                </footer>
            </xpath>
        </field>
    </record>
</odoo>


```

## File: views\product_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="mrp_subcontracting_product_template_search_view" model="ir.ui.view">
            <field name="name">mrp.subcontracting.product.template.search</field>
            <field name="model">product.template</field>
            <field name="inherit_id" ref="product.product_template_search_view" />
            <field name="groups_id" eval="[(4, ref('mrp.group_mrp_user'))]"/>
            <field name="arch" type="xml">
                <xpath expr="//filter[@name='filter_to_purchase']" position="after">
                    <filter string="Can be Subcontracted" name="filter_can_be_subcontracted" domain="[('bom_ids.type', '=', 'subcontract')]" />
                </xpath>
            </field>
        </record>
    </data>
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
                <separator/>
            </xpath>
        </field>
    </record>
    <record id="view_partner_mrp_subcontracting_filter" model="ir.ui.view">
        <field name="name">res.partner.select.inherit</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="base.view_res_partner_filter" />
        <field name="groups_id" eval="[(4, ref('mrp.group_mrp_user'))]"/>
        <field name="arch" type="xml">
            <xpath expr="//filter[@name='inactive']" position="before">
                <filter string="Subcontractors" name="type_subcontractors" domain="[('is_subcontractor', '=', True)]" />
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\stock_move_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="mrp_subcontracting_stock_move_line_tree_view" model="ir.ui.view">
        <field name="name">mrp.subcontracting.stock.move.line.tree.view</field>
        <field name="model">stock.move.line</field>
        <field name="priority">1000</field>
        <field name="arch" type="xml">
            <tree editable="bottom">
                <field name="company_id" invisible="1"/>
                <field name="product_uom_category_id" invisible="1"/>
                <field name="owner_id" invisible="1"/>
                <field name="tracking" invisible="1"/>
                <field name="package_id" invisible="1"/>
                <field name="result_package_id" invisible="1"/>
                <field name="location_id" invisible="1"/>
                <field name="location_dest_id" invisible="1"/>
                <field name="state" invisible="1"/>
                <!-- Don't put move_id here to avoid that the framework send falsy move_id -->
                <field name="id" invisible="1"/>
                <field name="product_id" required="1"/>
                <field name="lot_id" groups="stock.group_production_lot"
                    attrs="{'invisible': [('tracking', 'not in', ('serial', 'lot'))], 'required': [('tracking', 'in', ('serial', 'lot'))]}"
                    context="{'default_product_id': product_id, 'default_company_id': company_id}"/>
                <field name="product_uom_qty" readonly="1" force_save="1"/>
                <field name="qty_done"/>
                <field name="product_uom_id" groups="uom.group_uom"/>
            </tree>
        </field>
    </record>
    <record id="mrp_subcontracting_move_form_view" model="ir.ui.view">
        <field name="name">mrp.subcontracting.move.form.view</field>
        <field name="model">stock.move</field>
        <field name="priority">1000</field>
        <field name="arch" type="xml">
            <form create="0" delete="0">
                <header>
                    <field name="state" widget="statusbar"/>
                </header>
                <sheet>
                    <field name="product_uom_category_id" invisible="1"/>
                    <field name="company_id" invisible="1"/>
                    <field name="product_id" invisible="1"/>
                    <field name="sequence" invisible="1"/>
                    <field name="location_id" invisible="1"/>
                    <field name="picking_id" invisible="1"/>
                    <field name="location_dest_id" invisible="1"/>
                    <field name="has_tracking" invisible="1"/>
                    <field name="product_uom_qty" invisible="1"/>
                    <group>
                        <field name="order_finished_lot_ids" widget="many2many_tags"/>
                        <field name="product_uom" groups="uom.group_uom"/>
                        <field name="quantity_done" string="Total Consumed" readonly="1"/>
                    </group>
                    <field name="move_line_ids"
                        attrs="{'readonly': [('state', 'in', ['done', 'cancel'])]}"
                        context="{'default_product_uom_id': product_uom, 'default_picking_id': picking_id, 'default_move_id': id, 'default_product_id': product_id, 'default_location_id': location_id, 'default_location_dest_id': location_dest_id, 'default_company_id': company_id}">
                        <tree editable="bottom" decoration-muted="state in ('done', 'cancel')">
                            <field name="company_id" invisible="1"/>
                            <field name="state" invisible="1"/>
                            <field name="tracking" invisible="1"/>
                            <field name="product_uom_id" invisible="1"/>
                            <field name="product_uom_category_id" invisible="1"/>
                            <field name="picking_id" invisible="1"/>
                            <field name="move_id" invisible="1"/>
                            <field name="location_id" invisible="1"/>
                            <field name="location_dest_id" invisible="1"/>
                            <field name="product_id" readonly="1" force_save="1"/>
                            <field name="qty_done"/>
                            <field name="lot_id" attrs="{'column_invisible':[('parent.has_tracking', 'not in', ('serial', 'lot'))], 'required': [('tracking', 'in', ('serial', 'lot'))]}" context="{'default_product_id': product_id, 'default_company_id': company_id}" groups="stock.group_production_lot"/>
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
        <field name="arch" type="xml">
            <tree delete="0" create="0" decoration-muted="is_done" decoration-warning="quantity_done - product_uom_qty &gt; 0.0001" decoration-success="not is_done and quantity_done - product_uom_qty &lt; 0.0001">
                <field name="company_id" invisible="1"/>
                <field name="sequence" invisible="1"/>
                <field name="product_uom_category_id" invisible="1"/>
                <field name="name" invisible="1"/>
                <field name="unit_factor" invisible="1"/>
                <field name="date" invisible="1"/>
                <field name="picking_type_id" invisible="1"/>
                <field name="has_tracking" invisible="1"/>
                <field name="operation_id" invisible="1"/>
                <field name="is_done" invisible="1"/>
                <field name="bom_line_id" invisible="1"/>
                <field name="location_id" invisible="1"/>
                <field name="warehouse_id" invisible="1"/>
                <field name="product_uom_qty" invisible="1"/>
                <field name="location_dest_id" invisible="1"/>
                <field name="state" invisible="1" force_save="1"/>
                <field name="raw_material_production_id" invisible="1"/>
                <field name="product_id" required="1"/>
                <field name="order_finished_lot_ids" widget="many2many_tags"/>
                <field name="reserved_availability" attrs="{'invisible': [('is_done', '=', True)]}" string="Reserved"/>
                <field name="quantity_done" string="Consumed" readonly="1"/>
                <field name="product_uom" groups="uom.group_uom"/>
            </tree>
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

## File: wizard\stock_picking_return.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, fields


class ReturnPicking(models.TransientModel):
    _inherit = 'stock.return.picking'

    subcontract_location_id = fields.Many2one('stock.location', compute='_compute_subcontract_location_id')

    @api.depends('picking_id')
    def _compute_subcontract_location_id(self):
        for record in self:
            record.subcontract_location_id = record.picking_id.partner_id.with_company(
                record.picking_id.company_id
            ).property_stock_subcontractor

    @api.onchange('picking_id')
    def _onchange_picking_id(self):
        res = super(ReturnPicking, self)._onchange_picking_id()
        if any(return_line.quantity > 0 and return_line.move_id.is_subcontract for return_line in self.product_return_moves):
            self.location_id = self.picking_id.partner_id.with_company(self.picking_id.company_id).property_stock_subcontractor
        return res

    def _prepare_move_default_values(self, return_line, new_picking):
        vals = super(ReturnPicking, self)._prepare_move_default_values(return_line, new_picking)
        vals['is_subcontract'] = False
        return vals

```

## File: wizard\stock_picking_return_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_stock_return_picking_form_subcontracting" model="ir.ui.view">
        <field name="name">Return lines</field>
        <field name="model">stock.return.picking</field>
        <field name="inherit_id" ref="stock.view_stock_return_picking_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='location_id']" position="before">
                <field name="subcontract_location_id" invisible="1"/>
                <field name="original_location_id" invisible="1"/>
            </xpath>
            <xpath expr="//field[@name='location_id']" position="attributes">
                <attribute name="domain">
                    ['|', '|', ('id', '=', original_location_id), ('return_location', '=', True), ('id', '=', subcontract_location_id)]
                </attribute>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import stock_picking_return

```

