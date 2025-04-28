# Odoo Module: stock_picking_batch

Category: Inventory/Inventory

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Warehouse Management: Batch Transfer',
    'version': '1.0',
    'category': 'Inventory/Inventory',
    'description': """
This module adds the batch transfer option in warehouse management
==================================================================
    """,
    'depends': ['stock'],
    'data': [
        'security/ir.model.access.csv',
        'views/stock_picking_batch_views.xml',
        'views/stock_picking_type_views.xml',
        'views/stock_move_line_views.xml',
        'views/stock_picking_wave_views.xml',
        'views/stock_picking_views.xml',
        'data/stock_picking_batch_data.xml',
        'wizard/stock_picking_to_batch_views.xml',
        'wizard/stock_add_to_wave_views.xml',
        'report/stock_picking_batch_report_views.xml',
        'report/report_picking_batch.xml',
        'security/stock_picking_batch_security.xml',
    ],
    'demo': [
        'data/stock_picking_batch_demo.xml',
    ],
    'installable': True,
    'license': 'LGPL-3',
    'assets': {
        'web.assets_backend': [
            'stock_picking_batch/static/src/scss/*.scss',
        ],
        'web.assets_tests': [
            'stock_picking_batch/static/tests/tours/**/*',
        ],
    },
}

```

## File: data\stock_picking_batch_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">
    <!-- Batch picking related subtypes for messaging / Chatter -->
    <record id="mt_batch_state" model="mail.message.subtype">
        <field name="name">Stage Changed</field>
        <field name="res_model">stock.picking.batch</field>
        <field name="default" eval="False"/>
        <field name="description">Stage Changed</field>
    </record>

    <record id="seq_picking_batch" model="ir.sequence">
        <field name="name">Batch Transfer</field>
        <field name="code">picking.batch</field>
        <field name="prefix">BATCH/</field>
        <field name="padding">5</field>
        <field name="company_id" eval="False"/>
    </record>

    <record id="seq_picking_wave" model="ir.sequence">
        <field name="name">Wave Transfer</field>
        <field name="code">picking.wave</field>
        <field name="prefix">WAVE/</field>
        <field name="padding">5</field>
        <field name="company_id" eval="False"/>
    </record>

</data></odoo>

```

## File: data\stock_picking_batch_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">
    <!-- Starting inventory -->
    <record id="stock_inventory_batch_1" model="stock.quant">
        <field name="product_id" ref="product.consu_delivery_01"/>
        <field name="inventory_quantity">10.0</field>
        <field name="location_id" ref="stock.stock_location_stock"/>
    </record>
    <record id="stock_inventory_batch_2" model="stock.quant">
        <field name="product_id" ref="product.consu_delivery_02"/>
        <field name="inventory_quantity">18.0</field>
        <field name="location_id" ref="stock.stock_location_stock"/>
    </record>
    <record id="stock_inventory_batch_3" model="stock.quant">
        <field name="product_id" ref="product.consu_delivery_03"/>
        <field name="inventory_quantity">20.0</field>
        <field name="location_id" ref="stock.stock_location_stock"/>
    </record>

        <function model="stock.quant" name="action_apply_inventory">
            <function eval="[[('id', 'in', (ref('stock_inventory_batch_1'),
                                            ref('stock_inventory_batch_2'),
                                            ref('stock_inventory_batch_3'),
                                            ))]]" model="stock.quant" name="search"/>
        </function>

    <!-- Add picking -->
    <record id="Picking_A" model="stock.picking">
        <field name="move_type">one</field>
        <field name="priority">1</field>
        <field name="user_id" eval="False"/>
        <field name="state">draft</field>
        <field name="picking_type_id" ref="stock.picking_type_out"/>
        <field name="location_id" ref="stock.stock_location_stock"/>
        <field name="location_dest_id" ref="stock.stock_location_customers"/>
        <field name="company_id" ref="base.main_company"/>
    </record>
    <record id="Picking_B" model="stock.picking">
        <field name="move_type">one</field>
        <field name="priority">0</field>
        <field name="user_id" eval="False"/>
        <field name="state">draft</field>
        <field name="picking_type_id" ref="stock.picking_type_out"/>
        <field name="location_id" ref="stock.stock_location_stock"/>
        <field name="location_dest_id" ref="stock.stock_location_customers"/>
        <field name="company_id" ref="base.main_company"/>
    </record>
    <record id="Picking_C" model="stock.picking">
        <field name="move_type">one</field>
        <field name="priority">0</field>
        <field name="user_id" eval="False"/>
        <field name="state">draft</field>
        <field name="picking_type_id" ref="stock.picking_type_out"/>
        <field name="location_id" ref="stock.stock_location_stock"/>
        <field name="location_dest_id" ref="stock.stock_location_customers"/>
        <field name="company_id" ref="base.main_company"/>
    </record>
    <record id="Picking_D" model="stock.picking">
        <field name="move_type">one</field>
        <field name="priority">1</field>
        <field name="user_id" eval="False"/>
        <field name="state">draft</field>
        <field name="picking_type_id" ref="stock.picking_type_out"/>
        <field name="location_id" ref="stock.stock_location_stock"/>
        <field name="location_dest_id" ref="stock.stock_location_customers"/>
        <field name="company_id" ref="base.main_company"/>
    </record>

    <!-- Add batch picking -->
    <record id="stock_picking_batch_1" model="stock.picking.batch">
        <field name="picking_type_id" ref="stock.picking_type_out"/>
        <field name="company_id" ref="base.main_company"/>
        <field name="picking_ids" eval="[(6, 0, [ref('stock_picking_batch.Picking_C'), ref('stock_picking_batch.Picking_D')])]"/>
    </record>
    <record id="stock_picking_batch_2" model="stock.picking.batch">
        <field name="picking_type_id" ref="stock.picking_type_out"/>
        <field name="company_id" ref="base.main_company"/>
        <field name="picking_ids" eval="[(6, 0, [ref('stock_picking_batch.Picking_A'), ref('stock_picking_batch.Picking_B')])]"/>
    </record>

    <!-- Add stock move -->
    <record id="stock_move1" model="stock.move">
        <field name="name">A first stock move</field>
        <field name="picking_type_id" ref="stock.picking_type_out"/>
        <field name="picking_id" ref="Picking_A"/>
        <field name="location_id" ref="stock.stock_location_stock"/>
        <field name="location_dest_id" ref="stock.stock_location_customers"/>
        <field name="product_uom_qty">10</field>
        <field name="product_uom" ref="uom.product_uom_unit" />
        <field name="product_id" ref="product.consu_delivery_01"/>
    </record>
    <record id="stock_move2" model="stock.move">
        <field name="name">A second stock move</field>
        <field name="picking_type_id" ref="stock.picking_type_out"/>
        <field name="picking_id" ref="Picking_A"/>
        <field name="location_id" ref="stock.stock_location_stock"/>
        <field name="location_dest_id" ref="stock.stock_location_customers"/>
        <field name="product_uom_qty">10</field>
        <field name="product_uom" ref="uom.product_uom_unit" />
        <field name="product_id" ref="product.consu_delivery_02"/>
    </record>
    <record id="stock_move3" model="stock.move">
        <field name="name">A third stock move</field>
        <field name="picking_type_id" ref="stock.picking_type_out"/>
        <field name="picking_id" ref="Picking_B"/>
        <field name="location_id" ref="stock.stock_location_stock"/>
        <field name="location_dest_id" ref="stock.stock_location_customers"/>
        <field name="product_uom_qty">10</field>
        <field name="product_uom" ref="uom.product_uom_unit" />
        <field name="product_id" ref="product.consu_delivery_03"/>
    </record>
    <record id="stock_move4" model="stock.move">
        <field name="name">A fourth stock move</field>
        <field name="picking_type_id" ref="stock.picking_type_out"/>
        <field name="picking_id" ref="Picking_C"/>
        <field name="location_id" ref="stock.stock_location_stock"/>
        <field name="location_dest_id" ref="stock.stock_location_customers"/>
        <field name="product_uom_qty">4</field>
        <field name="product_uom" ref="uom.product_uom_unit" />
        <field name="product_id" ref="product.consu_delivery_03"/>
    </record>
    <record id="stock_move5" model="stock.move">
        <field name="name">A fifth stock move</field>
        <field name="picking_type_id" ref="stock.picking_type_out"/>
        <field name="picking_id" ref="Picking_D"/>
        <field name="location_id" ref="stock.stock_location_stock"/>
        <field name="location_dest_id" ref="stock.stock_location_customers"/>
        <field name="product_uom_qty">2</field>
        <field name="product_uom" ref="uom.product_uom_unit" />
        <field name="product_id" ref="product.product_product_10"/>
    </record>
    <record id="stock_move6" model="stock.move">
        <field name="name">A sixth stock move</field>
        <field name="picking_type_id" ref="stock.picking_type_out"/>
        <field name="picking_id" ref="Picking_D"/>
        <field name="location_id" ref="stock.stock_location_stock"/>
        <field name="location_dest_id" ref="stock.stock_location_customers"/>
        <field name="product_uom_qty">3</field>
        <field name="product_uom" ref="uom.product_uom_unit" />
        <field name="product_id" ref="product.product_product_25"/>
    </record>

    <!-- Confirm Batch Pickings -->
    <function model="stock.picking.batch" name="action_confirm">
        <value eval="ref('stock_picking_batch_1')"/>
    </function>
    <function model="stock.picking.batch" name="action_confirm">
        <value eval="ref('stock_picking_batch_2')"/>
    </function>
    <!-- Check Availability -->
    <function model="stock.picking.batch" name="action_assign">
        <value eval="ref('stock_picking_batch_1')"/>
    </function>
    <function model="stock.picking.batch" name="action_assign">
        <value eval="ref('stock_picking_batch_2')"/>
    </function>
</data></odoo>

```

## File: models\stock_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.osv import expression


class StockMove(models.Model):
    _inherit = "stock.move"

    def _search_picking_for_assignation_domain(self):
        domain = super()._search_picking_for_assignation_domain()
        domain = expression.AND([domain, ['|', ('batch_id', '=', False), ('batch_id.is_wave', '=', False)]])
        return domain

    def _action_cancel(self):
        res = super()._action_cancel()

        for picking in self.picking_id:
            # Remove the picking from the batch if the whole batch isn't cancelled.
            if picking.state == 'cancel' and picking.batch_id and any(p.state != 'cancel' for p in picking.batch_id.picking_ids):
                picking.batch_id = None
        return res

    def _assign_picking_post_process(self, new=False):
        super()._assign_picking_post_process(new=new)
        for picking in self.picking_id:
            picking._find_auto_batch()

    def write(self, vals):
        res = super().write(vals)
        if 'state' in vals and vals['state'] == 'assigned':
            for picking in self.picking_id:
                if picking.state != 'assigned':
                    continue
                picking._find_auto_batch()

        return res

    def _action_assign(self, force_qty=False):
        super()._action_assign(force_qty=force_qty)
        self.move_line_ids._auto_wave()

```

## File: models\stock_move_line.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict

from odoo import _, Command, fields, models
from odoo.osv import expression
from odoo.tools.float_utils import float_is_zero
from odoo.tools.misc import OrderedSet


class StockMoveLine(models.Model):
    _inherit = "stock.move.line"

    batch_id = fields.Many2one(related='picking_id.batch_id', store=True)

    def action_open_add_to_wave(self):
        # This action can be called from the move line list view or from the 'Add to wave' wizard
        if 'active_wave_id' in self.env.context:
            wave = self.env['stock.picking.batch'].browse(self.env.context.get('active_wave_id'))
            return self._add_to_wave(wave)
        view = self.env.ref('stock_picking_batch.stock_add_to_wave_form')
        return {
            'name': _('Add to Wave'),
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'stock.add.to.wave',
            'views': [(view.id, 'form')],
            'view_id': view.id,
            'target': 'new',
        }

    def _add_to_wave(self, wave=False, description=False):
        """ Detach lines (and corresponding stock move from a picking to another). If wave is
        passed, attach new picking into it. If not attach line to their original picking.

        :param int wave: id of the wave picking on which to put the move lines. """

        if not wave:
            wave = self.env['stock.picking.batch'].create({
                'is_wave': True,
                'picking_type_id': self.picking_type_id and self.picking_type_id[0].id,
                'user_id': self.env.context.get('active_owner_id'),
                'description': description,
            })
        line_by_picking = defaultdict(lambda: self.env['stock.move.line'])
        for line in self:
            line_by_picking[line.picking_id] |= line
        picking_to_wave_vals_list = []
        for picking, lines in line_by_picking.items():
            # Move the entire picking if all the line are taken
            line_by_move = defaultdict(lambda: self.env['stock.move.line'])
            qty_by_move = defaultdict(float)
            for line in lines:
                move = line.move_id
                line_by_move[move] |= line
                qty = line.product_uom_id._compute_quantity(line.quantity, line.product_id.uom_id, rounding_method='HALF-UP')
                qty_by_move[line.move_id] += qty

            # If all moves are to be transferred to the wave, link the picking to the wave
            if lines == picking.move_line_ids and lines.move_id == picking.move_ids:
                add_all_moves = True
                for move, qty in qty_by_move.items():
                    if float_is_zero(qty, precision_rounding=move.product_uom.rounding):
                        add_all_moves = False
                        break
                if add_all_moves:
                    wave.picking_ids = [Command.link(picking.id)]
                    continue

            # Split the picking in two part to extract only line that are taken on the wave
            picking_to_wave_vals = picking.copy_data({
                'move_ids': [],
                'move_line_ids': [],
                'batch_id': wave.id,
                'scheduled_date': picking.scheduled_date,
            })[0]
            for move, move_lines in line_by_move.items():
                picking_to_wave_vals['move_line_ids'] += [Command.link(line.id) for line in lines]
                # if all the line of a stock move are taken we change the picking on the stock move
                if move_lines == move.move_line_ids:
                    picking_to_wave_vals['move_ids'] += [Command.link(move.id)]
                    continue
                # Split the move
                qty = qty_by_move[move]
                new_move = move._split(qty)
                new_move[0]['move_line_ids'] = [Command.set(move_lines.ids)]
                picking_to_wave_vals['move_ids'] += [Command.create(new_move[0])]

            picking_to_wave_vals_list.append(picking_to_wave_vals)

        if picking_to_wave_vals_list:
            self.env['stock.picking'].create(picking_to_wave_vals_list)
        if wave.picking_type_id.batch_auto_confirm:
            wave.action_confirm()

    def _is_auto_waveable(self):
        self.ensure_one()
        if not self.picking_id \
           or (self.picking_id.state != 'assigned' or float_is_zero(self.quantity, precision_rounding=self.product_uom_id.rounding)) and not self.env.context.get('skip_auto_waveable')  \
           or self.batch_id.is_wave \
           or not self.picking_type_id._is_auto_wave_grouped() \
           or (self.picking_type_id.wave_group_by_category and self.product_id.categ_id not in self.picking_type_id.wave_category_ids):  # noqa: SIM103
            return False
        return True

    def _auto_wave(self):
        """ Try to find compatible waves to attach the move lines to, otherwise create new waves when possible/appropriate. """
        wave_locs_by_picking_type = {}
        for picking_type in self.picking_type_id:
            if not picking_type.wave_group_by_location:
                continue
            if picking_type in wave_locs_by_picking_type:
                continue
            wave_locs_by_picking_type[picking_type] = set(picking_type.wave_location_ids.ids)
        lines_nearest_parent_locations = defaultdict(lambda: self.env['stock.location'])
        batchable_line_ids = OrderedSet()
        for line in self:
            if not line._is_auto_waveable():
                continue
            if not line.picking_type_id.wave_group_by_location:
                batchable_line_ids.add(line.id)
                continue
            # We want to find the most descendant location in the wave locations list that is a parent of the line location.
            # Since the wave locations are ordered by complete_name (from the most descendant to the most ancestor), we can iterate in reverse order.
            wave_locs_set = wave_locs_by_picking_type[line.picking_type_id]
            loc = line.location_id
            while (loc):
                if loc.id in wave_locs_set:
                    lines_nearest_parent_locations[line] = loc
                    batchable_line_ids.add(line.id)
                    break
                loc = loc.location_id
        batchable_lines = self.env['stock.move.line'].browse(batchable_line_ids)

        remaining_line_ids = batchable_lines._auto_wave_lines_into_existing_waves(nearest_parent_locations=lines_nearest_parent_locations)
        remaining_lines = self.env['stock.move.line'].browse(remaining_line_ids)
        if remaining_lines:
            remaining_lines._auto_wave_lines_into_new_waves(nearest_parent_locations=lines_nearest_parent_locations)

    def _auto_wave_lines_into_existing_waves(self, nearest_parent_locations=False):
        """ Try to add move lines to existing waves if possible, return move lines of which no appropriate waves were found to link to
         :param nearest_parent_locations (defaultdict): the key is the move line and the value is the nearest parent location in the wave locations list"""
        remaining_lines = OrderedSet()
        for (picking_type, lines) in self.grouped(lambda l: l.picking_type_id).items():
            if lines:
                domain = [
                    ('picking_type_id', '=', picking_type.id),
                    ('company_id', 'in', lines.mapped('company_id').ids),
                    ('is_wave', '=', True)
                ]
                if picking_type.batch_auto_confirm:
                    domain = expression.AND([domain, [('state', 'not in', ['done', 'cancel'])]])
                else:
                    domain = expression.AND([domain, [('state', '=', 'draft')]])
                if picking_type.batch_group_by_partner:
                    domain = expression.AND([domain, [('picking_ids.partner_id', 'in', lines.move_id.partner_id.ids)]])
                if picking_type.batch_group_by_destination:
                    domain = expression.AND([domain, [('picking_ids.partner_id.country_id', 'in', lines.move_id.partner_id.country_id.ids)]])
                if picking_type.batch_group_by_src_loc:
                    domain = expression.AND([domain, [('picking_ids.location_id', 'in', lines.location_id.ids)]])
                if picking_type.batch_group_by_dest_loc:
                    domain = expression.AND([domain, [('picking_ids.location_dest_id', 'in', lines.location_dest_id.ids)]])

                potential_waves = self.env['stock.picking.batch'].search(domain)
                wave_to_new_lines = defaultdict(set)

                # These dictionaries are used to enforce batch max lines/transfers/weight limits
                # Each time a line is matched to a wave, we update the corresponding values
                wave_to_new_moves = defaultdict(set)
                waves_to_new_pickings = defaultdict(set)
                waves_new_extra_weight = defaultdict(float)

                waves_nearest_parent_locations = defaultdict(int)
                if picking_type.wave_group_by_location:
                    valid_wave_ids = set()
                    # We want to find the most descendant location in the wave locations list that is a parent of all the lines in each wave.
                    # We also want to exclude waves that have lines that are not in these locations.
                    for wave in potential_waves:
                        for wave_location in reversed(picking_type.wave_location_ids):
                            if all(loc._child_of(wave_location) for loc in wave.move_line_ids.location_id):
                                waves_nearest_parent_locations[wave] = wave_location.id
                                valid_wave_ids.add(wave.id)
                                break
                    potential_waves = self.env['stock.picking.batch'].browse(valid_wave_ids)

                for line in lines:
                    wave_found = False
                    for wave in potential_waves:
                        if line.company_id != wave.company_id \
                        or (picking_type.batch_group_by_partner and line.move_id.partner_id != wave.picking_ids.partner_id) \
                        or (picking_type.batch_group_by_destination and line.move_id.partner_id.country_id != wave.picking_ids.partner_id.country_id) \
                        or (picking_type.batch_group_by_src_loc and line.location_id != wave.picking_ids.location_id) \
                        or (picking_type.batch_group_by_dest_loc and line.location_dest_id != wave.picking_ids.location_dest_id) \
                        or (picking_type.wave_group_by_product and line.product_id != wave.move_line_ids.product_id) \
                        or (picking_type.wave_group_by_category and line.product_id.categ_id != wave.move_line_ids.product_id.categ_id) \
                        or (picking_type.wave_group_by_location and waves_nearest_parent_locations[wave] != nearest_parent_locations[line].id):
                            continue

                        wave_new_move_ids = wave_to_new_moves[wave]
                        wave_new_picking_ids = waves_to_new_pickings[wave]
                        wave_move_ids = set(wave.move_line_ids.mapped('move_id.id'))
                        wave_picking_ids = set(wave.move_line_ids.mapped('picking_id.id'))
                        # `is_line_auto_mergeable` is a method that checks if the line can be added to the wave without exceeding the limits
                        # It takes as arguments the number of new moves that will be added to the wave, the number of new pickings that will be added to the wave
                        # and the extra weight that will be added to the wave. So we need to check that the move/picking of the line is not already in the wave
                        # so that we don't count them as new moves/pickings.
                        if not wave._is_line_auto_mergeable(
                            line.move_id.id not in wave_move_ids and line.move_id.id not in wave_new_move_ids and len(wave_new_move_ids) + 1,
                            line.picking_id.id not in wave_picking_ids and line.picking_id.id not in wave_new_picking_ids and len(wave_new_picking_ids) + 1,
                            waves_new_extra_weight[wave] + line.product_id.weight * line.quantity_product_uom
                        ):
                            continue

                        if line.move_id.id not in wave_move_ids:
                            wave_to_new_moves[wave].add(line.move_id.id)
                        if line.picking_id.id not in wave_picking_ids:
                            waves_to_new_pickings[wave].add(line.picking_id.id)
                        waves_new_extra_weight[wave] += line.product_id.weight * line.quantity_product_uom
                        wave_to_new_lines[wave].add(line.id)
                        wave_found = True
                        break
                    if not wave_found:
                        remaining_lines.add(line.id)
                for wave, line_ids in wave_to_new_lines.items():
                    lines = self.env['stock.move.line'].browse(line_ids)
                    lines._add_to_wave(wave)
        return list(remaining_lines)

    def _auto_wave_lines_into_new_waves(self, nearest_parent_locations=False):
        """ Create new waves for the move lines that could not be added to existing waves. """
        picking_types = self.picking_type_id
        for picking_type in picking_types:
            lines = self.filtered(lambda l: l.picking_type_id == picking_type)
            domain = [
                ('id', 'in', lines.ids),
                ('company_id', 'in', self.company_id.ids),
                ('picking_id.state', '=', 'assigned'),
                ('picking_type_id', '=', picking_type.id),
                '|',
                ('batch_id', '=', False),
                ('batch_id.is_wave', '=', False)
            ]
            if picking_type.batch_group_by_partner:
                domain = expression.AND([domain, [('move_id.partner_id', 'in', lines.move_id.partner_id.ids)]])
            if picking_type.batch_group_by_destination:
                domain = expression.AND([domain, [('move_id.partner_id.country_id', 'in', lines.move_id.partner_id.country_id.ids)]])
            if picking_type.batch_group_by_src_loc:
                domain = expression.AND([domain, [('location_id', 'in', lines.location_id.ids)]])
            if picking_type.batch_group_by_dest_loc:
                domain = expression.AND([domain, [('location_dest_id', 'in', lines.location_dest_id.ids)]])
            if picking_type.wave_group_by_product:
                domain = expression.AND([domain, [('product_id', 'in', lines.product_id.ids)]])
            if picking_type.wave_group_by_category:
                domain = expression.AND([domain, [('product_id.categ_id', 'in', lines.product_id.categ_id.ids)]])
            if picking_type.wave_group_by_location:
                domain = expression.AND([domain, [('location_id', 'child_of', picking_type.wave_location_ids.ids)]])

            potential_lines = self.env['stock.move.line'].search(domain)
            lines_nearest_parent_locations = defaultdict(int)
            if picking_type.wave_group_by_location:
                for line in potential_lines:
                    for location in reversed(picking_type.wave_location_ids):
                        if line.location_id._child_of(location):
                            lines_nearest_parent_locations[line] = location.id
                            break

            line_to_lines = defaultdict(set)
            matched_lines = set()
            remaining_line_ids = OrderedSet()
            for line in lines:
                lines_found = False
                if line.id in matched_lines:
                    continue
                for potential_line in potential_lines:
                    if line.id == potential_line.id \
                    or line.company_id != potential_line.company_id \
                    or (picking_type.batch_group_by_partner and line.move_id.partner_id != potential_line.move_id.partner_id) \
                    or (picking_type.batch_group_by_destination and line.move_id.partner_id.country_id != potential_line.move_id.partner_id.country_id) \
                    or (picking_type.batch_group_by_src_loc and line.location_id != potential_line.location_id) \
                    or (picking_type.batch_group_by_dest_loc and line.location_dest_id != potential_line.location_dest_id) \
                    or (picking_type.wave_group_by_product and line.product_id != potential_line.product_id) \
                    or (picking_type.wave_group_by_category and line.product_id.categ_id != potential_line.product_id.categ_id) \
                    or (picking_type.wave_group_by_location and lines_nearest_parent_locations[potential_line] != nearest_parent_locations[line].id):
                        continue

                    line_to_lines[line].add(potential_line.id)
                    matched_lines.add(potential_line.id)
                    lines_found = True
                if not lines_found:
                    remaining_line_ids.add(line.id)

            for line, potential_line_ids in line_to_lines.items():
                if line.batch_id.is_wave:
                    continue

                potential_lines = self.env['stock.move.line'].browse(potential_line_ids | {line.id})

                # We want to make sure that batch/wave limits specified in the picking type are respected.
                # We want also to reduce picking splits as much as possible. So we try to group as much as possible by sorting the lines by picking and move.
                potential_lines = potential_lines.sorted(key=lambda l: (l.picking_id.id, l.move_id.id))

                while potential_lines:
                    new_wave = self.env['stock.picking.batch'].create({
                        'is_wave': True,
                        'picking_type_id': picking_type.id,
                        'description': line._get_auto_wave_description(nearest_parent_locations[line]),
                    })
                    wave_move_ids = set()
                    wave_picking_ids = set()
                    wave_weight = 0

                    wave_line_ids = set()

                    for potential_line in potential_lines:
                        if potential_line.batch_id.is_wave:
                            continue
                        wave_move_ids.add(potential_line.move_id.id)
                        wave_picking_ids.add(potential_line.picking_id.id)
                        wave_weight += potential_line.product_id.weight * potential_line.quantity_product_uom
                        if new_wave._is_line_auto_mergeable(
                            len(wave_move_ids),
                            len(wave_picking_ids),
                            wave_weight
                        ):
                            wave_line_ids.add(potential_line.id)
                        else:
                            break
                    wave_lines = self.env['stock.move.line'].browse(wave_line_ids)
                    wave_lines._add_to_wave(new_wave)
                    potential_lines -= wave_lines

            remaining_lines = self.env['stock.move.line'].browse(remaining_line_ids)
            remaining_waves = self.env['stock.picking.batch'].create([{
                'is_wave': True,
                'picking_type_id': picking_type.id,
                'description': remaining_line._get_auto_wave_description(nearest_parent_locations[remaining_line]),
            } for remaining_line in remaining_lines])
            for (line, wave) in zip(remaining_lines, remaining_waves):
                line._add_to_wave(wave)

    def _get_auto_wave_description(self, nearest_parent_location=False):
        self.ensure_one()
        description = self.picking_id._get_auto_batch_description()
        description_items = []
        if description:
            description_items.append(description)

        if self.picking_type_id.wave_group_by_product:
            description_items.append(self.product_id.display_name)
        if self.picking_type_id.wave_group_by_category:
            description_items.append(self.product_id.categ_id.complete_name)
        if self.picking_type_id.wave_group_by_location:
            description_items.append(nearest_parent_location.complete_name)

        description = ', '.join(description_items)
        return description

```

## File: models\stock_picking.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, Command, fields, models
from odoo.osv import expression
from odoo.exceptions import ValidationError


class StockPickingType(models.Model):
    _inherit = "stock.picking.type"

    count_picking_batch = fields.Integer(compute='_compute_picking_count')
    count_picking_wave = fields.Integer(compute='_compute_picking_count')
    auto_batch = fields.Boolean('Automatic Batches',
                                help="Automatically put pickings into batches as they are confirmed when possible.")
    batch_group_by_partner = fields.Boolean('Contact', help="Automatically group batches by contacts.")
    batch_group_by_destination = fields.Boolean('Destination Country', help="Automatically group batches by destination country.")
    batch_group_by_src_loc = fields.Boolean('Group by Source Location',
                                            help="Automatically group batches by their source location.")
    batch_group_by_dest_loc = fields.Boolean('Group by Destination Location',
                                             help="Automatically group batches by their destination location.")
    wave_group_by_product = fields.Boolean('Product', help="Split transfers by product then group transfers that have the same product.")
    wave_group_by_category = fields.Boolean('Product Category', help="Split transfers by product category, then group transfers that have the same product category.")
    wave_category_ids = fields.Many2many('product.category', string='Wave Product Categories', help="Categories to consider when grouping waves.")
    wave_group_by_location = fields.Boolean('Location', help="Split transfers by defined locations, then group transfers with the same location.")
    wave_location_ids = fields.Many2many('stock.location', string='Wave Locations', help="Locations to consider when grouping waves.", domain="[('usage', '=', 'internal')]")
    batch_max_lines = fields.Integer("Maximum lines",
                                     help="A transfer will not be automatically added to batches that will exceed this number of lines if the transfer is added to it.\n"
                                          "Leave this value as '0' if no line limit.")
    batch_max_pickings = fields.Integer("Maximum transfers",
                                        help="A transfer will not be automatically added to batches that will exceed this number of transfers.\n"
                                             "Leave this value as '0' if no transfer limit.")
    batch_auto_confirm = fields.Boolean("Auto-confirm", default=True)
    batch_properties_definition = fields.PropertiesDefinition('Batch Properties')

    def _compute_picking_count(self):
        super()._compute_picking_count()
        data = self.env['stock.picking.batch']._read_group(
            [('state', 'not in', ('done', 'cancel')), ('picking_type_id', 'in', self.ids)],
            ['picking_type_id', 'is_wave'], ['__count'])
        count = {(picking_type.id, is_wave): count for picking_type, is_wave, count in data}
        for record in self:
            record.count_picking_wave = count.get((record.id, True), 0)
            record.count_picking_batch = count.get((record.id, False), 0)

    def action_batch(self):
        action = self.env['ir.actions.act_window']._for_xml_id("stock_picking_batch.stock_picking_batch_action")
        if self.env.context.get("view_mode"):
            del action["mobile_view_mode"]
            del action["views"]
            action["view_mode"] = self.env.context["view_mode"]
        return action

    @api.model
    def _is_auto_batch_grouped(self):
        self.ensure_one()
        return self.auto_batch and any(self[key] for key in self._get_batch_group_by_keys())

    @api.model
    def _is_auto_wave_grouped(self):
        self.ensure_one()
        return self.auto_batch and any(self[key] for key in self._get_wave_group_by_keys())

    @api.model
    def _get_batch_group_by_keys(self):
        return ['batch_group_by_partner', 'batch_group_by_destination', 'batch_group_by_src_loc', 'batch_group_by_dest_loc']

    @api.model
    def _get_wave_group_by_keys(self):
        return ['wave_group_by_product', 'wave_group_by_category', 'wave_group_by_location']

    @api.model
    def _get_batch_and_wave_group_by_keys(self):
        return self._get_batch_group_by_keys() + self._get_wave_group_by_keys()

    @api.constrains(lambda self: self._get_batch_group_by_keys() + ['auto_batch'])
    def _validate_auto_batch_group_by(self):
        group_by_keys = self._get_batch_and_wave_group_by_keys()
        for picking_type in self:
            if not picking_type.auto_batch:
                continue
            if not any(picking_type[key] for key in group_by_keys):
                raise ValidationError(_("If the Automatic Batches feature is enabled, at least one 'Group by' option must be selected."))


class StockPicking(models.Model):
    _inherit = "stock.picking"

    batch_id = fields.Many2one(
        'stock.picking.batch', string='Batch Transfer',
        check_company=True,
        help='Batch associated to this transfer', index=True, copy=False)
    batch_sequence = fields.Integer(string='Sequence')

    @api.model_create_multi
    def create(self, vals_list):
        pickings = super().create(vals_list)
        for picking, vals in zip(pickings, vals_list):
            if vals.get('batch_id'):
                if not picking.batch_id.picking_type_id:
                    picking.batch_id.picking_type_id = picking.picking_type_id[0]
                picking.batch_id._sanity_check()
        return pickings

    def write(self, vals):
        old_batches = self.batch_id
        res = super().write(vals)
        if vals.get('batch_id'):
            old_batches.filtered(lambda b: not b.picking_ids).state = 'cancel'
            if not self.batch_id.picking_type_id:
                self.batch_id.picking_type_id = self.picking_type_id[0]
            self.batch_id._sanity_check()
            # assign batch users to batch pickings
            self.batch_id.picking_ids.assign_batch_user(self.batch_id.user_id.id)
        return res

    def action_add_operations(self):
        view = self.env.ref('stock_picking_batch.view_move_line_tree_detailed_wave')
        return {
            'name': _('Add Operations'),
            'type': 'ir.actions.act_window',
            'view_mode': 'list',
            'view': view,
            'views': [(view.id, 'list')],
            'res_model': 'stock.move.line',
            'target': 'new',
            'domain': [
                ('picking_id', 'in', self.ids),
                ('state', '!=', 'done')
            ],
            'context': dict(
                self.env.context,
                picking_to_wave=self.ids,
                active_wave_id=self.env.context.get('active_wave_id').id,
                search_default_by_location=True,
            )}

    def action_confirm(self):
        res = super().action_confirm()
        for picking in self:
            picking._find_auto_batch()
        return res

    def button_validate(self):
        res = super().button_validate()
        to_assign_ids = set()
        # Having non-done pickings after the `super()` call means it stopped early,
        # so we shouldn’t remove the pickings from batches yet.
        if not any(picking.state == 'done' for picking in self):
            return res
        if self and self.env.context.get('pickings_to_detach'):
            pickings_to_detach = self.env['stock.picking'].browse(self.env.context['pickings_to_detach'])
            pickings_to_detach.batch_id = False
            pickings_to_detach.move_ids.filtered(lambda m: not m.quantity).picked = False
            to_assign_ids.update(self.env.context['pickings_to_detach'])

        for picking in self:
            if picking.state != 'done':
                continue
            # Avoid inconsistencies in states of the same batch when validating a single picking in a batch.
            if picking.batch_id and any(p.state != 'done' for p in picking.batch_id.picking_ids):
                picking.batch_id = None
            # If backorder were made, if auto-batch is enabled, seek a batch for each of them with the selected criterias.
            to_assign_ids.update(picking.backorder_ids.ids)

        # To avoid inconsistencies, all incorrect pickings must be removed before assigning backorder pickings
        assignable_pickings = self.env['stock.picking'].browse(to_assign_ids)
        for picking in assignable_pickings:
            picking._find_auto_batch()
        assignable_pickings.move_line_ids.with_context(skip_auto_waveable=True)._auto_wave()

        return res

    def _create_backorder(self, backorder_moves=None):
        pickings_to_detach = self.env['stock.picking'].browse(self.env.context.get('pickings_to_detach'))
        for picking in self:
            # Avoid inconsistencies in states of the same batch when validating a single picking in a batch.
            if picking.batch_id and picking.state != 'done' and any(p not in self for p in picking.batch_id.picking_ids - pickings_to_detach):
                picking.batch_id = None
        return super()._create_backorder(backorder_moves)

    def action_cancel(self):
        res = super().action_cancel()
        for picking in self:
            if picking.batch_id and any(picking.state != 'cancel' for picking in picking.batch_id.picking_ids):
                picking.batch_id = None
        return res

    def _should_show_transfers(self):
        if len(self.batch_id) == 1 and len(self) == (len(self.batch_id.picking_ids) - len(self.env.context.get('pickings_to_detach', []))):
            return False
        return super()._should_show_transfers()

    def _find_auto_batch(self):
        self.ensure_one()
        # Check if auto_batch is enabled for this picking.
        if not self.picking_type_id.auto_batch or not self.picking_type_id._is_auto_batch_grouped() or self.batch_id or not self.move_ids or not self._is_auto_batchable():
            return False

        # Try to find a compatible batch to insert the picking
        possible_batches = self.env['stock.picking.batch'].sudo().search(self._get_possible_batches_domain())
        for batch in possible_batches:
            if batch._is_picking_auto_mergeable(self):
                batch.picking_ids |= self
                return batch

        # If no batch were found, try to find a compatible picking and put them both in a new batch.
        possible_pickings = self.env['stock.picking'].search(self._get_possible_pickings_domain())
        new_batch_data = {
            'picking_ids': [Command.link(self.id)],
            'company_id': self.company_id.id if self.company_id else False,
            'picking_type_id': self.picking_type_id.id,
            'description': self._get_auto_batch_description()
        }
        for picking in possible_pickings:
            if self._is_auto_batchable(picking):
                # Add the picking to the new batch
                new_batch_data['picking_ids'].append(Command.link(picking.id))
                new_batch = self.env['stock.picking.batch'].sudo().create(new_batch_data)
                if picking.picking_type_id.batch_auto_confirm:
                    new_batch.action_confirm()
                return new_batch

        # If nothing was found after those two steps, then create a batch with the current picking alone
        new_batch = self.env['stock.picking.batch'].sudo().create(new_batch_data)
        if self.picking_type_id.batch_auto_confirm:
            new_batch.action_confirm()
        return new_batch

    def _is_auto_batchable(self, picking=None):
        """ Verifies if a picking can be put in a batch with another picking without violating auto_batch constrains.
        """
        if self.state != 'assigned':
            return False
        res = True
        if not picking:
            picking = self.env['stock.picking']
        if self.picking_type_id.batch_max_lines:
            res = res and (len(self.move_ids) + len(picking.move_ids) <= self.picking_type_id.batch_max_lines)
        if self.picking_type_id.batch_max_pickings:
            # Sounds absurd. BUT if we put "batch max picking" to a value <= 1, makes sense ... Or not. Because then there is no point to batch.
            res = res and self.picking_type_id.batch_max_pickings > 1
        return res

    def _get_possible_pickings_domain(self):
        self.ensure_one()
        domain = [
            ('id', '!=', self.id),
            ('company_id', '=', self.company_id.id if self.company_id else False),
            ('state', '=', 'assigned'),
            ('picking_type_id', '=', self.picking_type_id.id),
            ('batch_id', '=', False),
        ]
        if self.picking_type_id.batch_group_by_partner:
            domain = expression.AND([domain, [('partner_id', '=', self.partner_id.id)]])
        if self.picking_type_id.batch_group_by_destination:
            domain = expression.AND([domain, [('partner_id.country_id', '=', self.partner_id.country_id.id)]])
        if self.picking_type_id.batch_group_by_src_loc:
            domain = expression.AND([domain, [('location_id', '=', self.location_id.id)]])
        if self.picking_type_id.batch_group_by_dest_loc:
            domain = expression.AND([domain, [('location_dest_id', '=', self.location_dest_id.id)]])

        return domain

    def _get_possible_batches_domain(self):
        self.ensure_one()
        domain = [
            ('state', 'in', ('draft', 'in_progress') if self.picking_type_id.batch_auto_confirm else ('draft',)),
            ('picking_type_id', '=', self.picking_type_id.id),
            ('company_id', '=', self.company_id.id if self.company_id else False),
            ('is_wave', '=', False)
        ]
        if self.picking_type_id.batch_group_by_partner:
            domain = expression.AND([domain, [('picking_ids.partner_id', '=', self.partner_id.id)]])
        if self.picking_type_id.batch_group_by_destination:
            domain = expression.AND([domain, [('picking_ids.partner_id.country_id', '=', self.partner_id.country_id.id)]])
        if self.picking_type_id.batch_group_by_src_loc:
            domain = expression.AND([domain, [('picking_ids.location_id', '=', self.location_id.id)]])
        if self.picking_type_id.batch_group_by_dest_loc:
            domain = expression.AND([domain, [('picking_ids.location_dest_id', '=', self.location_dest_id.id)]])

        return domain

    def _get_auto_batch_description(self):
        """ Get the description of the automatically created batch based on the grouped pickings and grouping criteria """
        self.ensure_one()
        description_items = []
        if self.picking_type_id.batch_group_by_partner and self.partner_id:
            description_items.append(self.partner_id.name or '')
        if self.picking_type_id.batch_group_by_destination and self.partner_id.country_id:
            description_items.append(self.partner_id.country_id.name)
        if self.picking_type_id.batch_group_by_src_loc and self.location_id:
            description_items.append(self.location_id.display_name)
        if self.picking_type_id.batch_group_by_dest_loc and self.location_dest_id:
            description_items.append(self.location_dest_id.display_name)
        return ', '.join(description_items)

    def _package_move_lines(self, batch_pack=False, move_lines_to_pack=False):
        if batch_pack:
            return super(StockPicking, self.batch_id.picking_ids if self.batch_id else self)._package_move_lines(batch_pack, move_lines_to_pack)
        return super()._package_move_lines(batch_pack, move_lines_to_pack)

    def assign_batch_user(self, user_id):
        pickings = self.filtered(lambda p: p.user_id.id != user_id)
        pickings.write({'user_id': user_id})
        for pick in pickings:
            if user_id:
                log_message = _('Assigned to %s Responsible', pick.batch_id._get_html_link())
            else:
                log_message = _('Unassigned responsible from %s', pick.batch_id._get_html_link())
            pick.message_post(body=log_message)

    def action_view_batch(self):
        self.ensure_one()
        return {
            'type': 'ir.actions.act_window',
            'res_model': 'stock.picking.batch',
            'res_id': self.batch_id.id,
            'view_mode': 'form'
        }

```

## File: models\stock_picking_batch.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from markupsafe import Markup

from odoo import api, fields, models, _
from odoo.exceptions import UserError
from odoo.osv.expression import AND
from odoo.tools import float_is_zero, format_list

class StockPickingBatch(models.Model):
    _inherit = ['mail.thread', 'mail.activity.mixin']
    _name = "stock.picking.batch"
    _description = "Batch Transfer"
    _order = "name desc"

    name = fields.Char(
        string='Batch Transfer', default='New',
        copy=False, required=True, readonly=True)
    description = fields.Char('Description')
    user_id = fields.Many2one(
        'res.users', string='Responsible', tracking=True, check_company=True)
    company_id = fields.Many2one(
        'res.company', string="Company", required=True, readonly=True,
        index=True, default=lambda self: self.env.company)
    picking_ids = fields.One2many(
        'stock.picking', 'batch_id', string='Transfers',
        domain="[('id', 'in', allowed_picking_ids)]", check_company=True,
        help='List of transfers associated to this batch')
    show_check_availability = fields.Boolean(
        compute='_compute_move_ids',
        string='Show Check Availability')
    show_allocation = fields.Boolean(
        compute='_compute_show_allocation',
        string='Show Allocation Button')
    allowed_picking_ids = fields.One2many('stock.picking', compute='_compute_allowed_picking_ids')
    move_ids = fields.One2many(
        'stock.move', string="Stock moves", compute='_compute_move_ids')
    move_line_ids = fields.One2many(
        'stock.move.line', string='Stock move lines',
        compute='_compute_move_line_ids', inverse='_set_move_line_ids', search='_search_move_line_ids')
    state = fields.Selection([
        ('draft', 'Draft'),
        ('in_progress', 'In progress'),
        ('done', 'Done'),
        ('cancel', 'Cancelled')], default='draft',
        store=True, compute='_compute_state',
        copy=False, tracking=True, required=True, readonly=True, index=True)
    picking_type_id = fields.Many2one(
        'stock.picking.type', 'Operation Type', check_company=True, copy=False,
        index=True)
    warehouse_id = fields.Many2one(
        'stock.warehouse', related='picking_type_id.warehouse_id')
    picking_type_code = fields.Selection(
        related='picking_type_id.code')
    scheduled_date = fields.Datetime(
        'Scheduled Date', copy=False, store=True, readonly=False, compute="_compute_scheduled_date",
        help="""Scheduled date for the transfers to be processed.
              - If manually set then scheduled date for all transfers in batch will automatically update to this date.
              - If not manually changed and transfers are added/removed/updated then this will be their earliest scheduled date
                but this scheduled date will not be set for all transfers in batch.""")
    is_wave = fields.Boolean('This batch is a wave')
    show_lots_text = fields.Boolean(compute='_compute_show_lots_text')
    estimated_shipping_weight = fields.Float(
        "shipping_weight", compute='_compute_estimated_shipping_capacity', digits='Product Unit of Measure')
    estimated_shipping_volume = fields.Float(
        "shipping_volume", compute='_compute_estimated_shipping_capacity', digits='Product Unit of Measure')
    properties = fields.Properties('Properties', definition='picking_type_id.batch_properties_definition', copy=True)

    @api.depends('description')
    @api.depends_context('add_to_existing_batch')
    def _compute_display_name(self):
        if not self.env.context.get('add_to_existing_batch'):
            return super()._compute_display_name()
        for batch in self:
            batch.display_name = f"{batch.name}: {batch.description}" if batch.description else batch.name

    @api.depends('picking_type_id')
    def _compute_show_lots_text(self):
        for batch in self:
            batch.show_lots_text = batch.picking_ids and batch.picking_ids[0].show_lots_text

    def _compute_estimated_shipping_capacity(self):
        for batch in self:
            estimated_shipping_weight = 0
            estimated_shipping_volume = 0
            # packs
            for pack in self.move_line_ids.result_package_id:
                p_type = pack.package_type_id
                estimated_shipping_weight += pack.shipping_weight
                if p_type:
                    estimated_shipping_weight += p_type.base_weight or 0
                    estimated_shipping_volume += (p_type.packaging_length * p_type.width * p_type.height) / 1000.0**3
            # move without packs
            for move in self.picking_ids.move_ids_without_package:
                estimated_shipping_weight += move.product_id.weight * move.product_qty
                estimated_shipping_volume += move.product_id.volume * move.product_qty
            batch.estimated_shipping_weight = estimated_shipping_weight
            batch.estimated_shipping_volume = estimated_shipping_volume

    @api.depends('company_id', 'picking_type_id', 'state')
    def _compute_allowed_picking_ids(self):
        allowed_picking_states = ['waiting', 'confirmed', 'assigned']

        for batch in self:
            domain_states = list(allowed_picking_states)
            # Allows to add draft pickings only if batch is in draft as well.
            if batch.state == 'draft':
                domain_states.append('draft')
            domain = [
                ('company_id', '=', batch.company_id.id),
                ('state', 'in', domain_states),
            ]
            if batch.picking_type_id:
                domain += [('picking_type_id', '=', batch.picking_type_id.id)]
            batch.allowed_picking_ids = self.env['stock.picking'].search(domain)

    @api.depends('picking_ids', 'picking_ids.move_line_ids', 'picking_ids.move_ids', 'picking_ids.move_ids.state')
    def _compute_move_ids(self):
        for batch in self:
            batch.move_ids = batch.picking_ids.move_ids
            batch.show_check_availability = any(m.state not in ['assigned', 'cancel', 'done'] for m in batch.move_ids)

    @api.depends('picking_ids', 'picking_ids.move_line_ids')
    def _compute_move_line_ids(self):
        for batch in self:
            batch.move_line_ids = batch.picking_ids.move_line_ids

    def _search_move_line_ids(self, operator, value):
        return [('picking_ids.move_line_ids',operator,value)]

    @api.depends('state', 'move_ids', 'picking_type_id')
    def _compute_show_allocation(self):
        self.show_allocation = False
        if not self.env.user.has_group('stock.group_reception_report'):
            return
        for batch in self:
            batch.show_allocation = batch.picking_ids._get_show_allocation(batch.picking_type_id)

    @api.depends('picking_ids', 'picking_ids.state')
    def _compute_state(self):
        batchs = self.filtered(lambda batch: batch.state not in ['cancel', 'done'])
        for batch in batchs:
            if not batch.picking_ids:
                continue
            # Cancels automatically the batch picking if all its transfers are cancelled.
            if all(picking.state == 'cancel' for picking in batch.picking_ids):
                batch.state = 'cancel'
            # Batch picking is marked as done if all its not canceled transfers are done.
            elif all(picking.state in ['cancel', 'done'] for picking in batch.picking_ids):
                batch.state = 'done'

    @api.depends('picking_ids', 'picking_ids.scheduled_date')
    def _compute_scheduled_date(self):
        for rec in self:
            rec.scheduled_date = min(rec.picking_ids.filtered('scheduled_date').mapped('scheduled_date'), default=False)

    @api.onchange('scheduled_date')
    def onchange_scheduled_date(self):
        if self.scheduled_date:
            self.picking_ids.scheduled_date = self.scheduled_date

    def _set_move_line_ids(self):
        new_move_lines = self[0].move_line_ids
        for picking in self.picking_ids:
            old_move_lines = picking.move_line_ids
            picking.move_line_ids = new_move_lines.filtered(lambda ml: ml.picking_id.id == picking.id)
            move_lines_to_unlink = old_move_lines - new_move_lines
            if move_lines_to_unlink:
                move_lines_to_unlink.unlink()

    # -------------------------------------------------------------------------
    # CRUD
    # -------------------------------------------------------------------------
    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            if vals.get('name', '/') == '/':
                company_id = vals.get('company_id', self.env.company.id)
                if vals.get('is_wave'):
                    vals['name'] = self.env['ir.sequence'].with_company(company_id).next_by_code('picking.wave') or '/'
                else:
                    vals['name'] = self.env['ir.sequence'].with_company(company_id).next_by_code('picking.batch') or '/'
        return super().create(vals_list)

    def write(self, vals):
        res = super().write(vals)
        if not self.picking_ids:
            self.filtered(lambda b: b.state == 'in_progress').action_cancel()
        if vals.get('picking_type_id'):
            self._sanity_check()
        if vals.get('picking_ids'):
            batch_without_picking_type = self.filtered(lambda batch: not batch.picking_type_id)
            if batch_without_picking_type:
                picking = self.picking_ids and self.picking_ids[0]
                batch_without_picking_type.picking_type_id = picking.picking_type_id.id
        if 'user_id' in vals:
            self.picking_ids.assign_batch_user(vals['user_id'])
        return res

    @api.ondelete(at_uninstall=False)
    def _unlink_if_not_done(self):
        if any(batch.state == 'done' for batch in self):
            raise UserError(_("You cannot delete Done batch transfers."))

    # -------------------------------------------------------------------------
    # Action methods
    # -------------------------------------------------------------------------
    def action_confirm(self):
        """Sanity checks, confirm the pickings and mark the batch as confirmed."""
        self.ensure_one()
        if not self.picking_ids:
            raise UserError(_("You have to set some pickings to batch."))
        self.picking_ids.action_confirm()
        self._check_company()
        self.state = 'in_progress'
        return True

    def action_cancel(self):
        self.state = 'cancel'
        self.picking_ids = False
        return True

    def action_print(self):
        self.ensure_one()
        return self.env.ref('stock_picking_batch.action_report_picking_batch').report_action(self)

    def action_done(self):
        def has_no_quantity(picking):
            return all(not m.picked or float_is_zero(m.quantity, precision_rounding=m.product_uom.rounding) for m in picking.move_ids if m.state not in ('done', 'cancel'))

        def is_empty(picking):
            return all(float_is_zero(m.quantity, precision_rounding=m.product_uom.rounding) for m in picking.move_ids if m.state not in ('done', 'cancel'))

        self.ensure_one()
        self._check_company()
        # Empty 'assigned' or 'waiting for another operation' pickings will be removed from the batch when it is validated.
        pickings = self.mapped('picking_ids').filtered(lambda picking: picking.state not in ('cancel', 'done'))
        empty_waiting_pickings = self.mapped('picking_ids').filtered(lambda p: (p.state in ('waiting', 'confirmed') and has_no_quantity(p)) or (p.state == 'assigned' and is_empty(p)))
        pickings = pickings - empty_waiting_pickings

        empty_pickings = pickings.filtered(has_no_quantity)

        # Run sanity_check as a batch and ignore the one in button_validate() since it is done here.
        pickings._sanity_check(separate_pickings=False)
        # Skip sanity_check in pickings button_validate() & remove 'waiting' pickings from the batch
        context = {'skip_sanity_check': True, 'pickings_to_detach': empty_waiting_pickings.ids}
        if len(empty_pickings) != len(pickings):
            # If some pickings are at least partially done, other pickings (empty & waiting) will be removed from batch without being cancelled in case of no backorder
            pickings = pickings - empty_pickings
            context['pickings_to_detach'] = context['pickings_to_detach'] + empty_pickings.ids

        for picking in pickings:
            picking.message_post(
                body=Markup("<b>%s:</b> %s <a href=#id=%s&view_type=form&model=stock.picking.batch>%s</a>") % (
                    _("Transferred by"),
                    _("Batch Transfer"),
                    picking.batch_id.id,
                    picking.batch_id.name))

        return pickings.with_context(**context).button_validate()

    def action_assign(self):
        self.ensure_one()
        self.picking_ids.action_assign()

    def action_put_in_pack(self):
        """ Action to put move lines with 'Done' quantities into a new pack
        This method follows same logic to stock.picking.
        """
        self.ensure_one()
        if self.state not in ('done', 'cancel'):
            move_line_ids = self.picking_ids[0]._package_move_lines(batch_pack=True)
            if move_line_ids:
                res = move_line_ids.picking_id[0]._pre_put_in_pack_hook(move_line_ids)
                if res:
                    return res
                package = move_line_ids.picking_id._put_in_pack(move_line_ids)
                return move_line_ids.picking_id[0]._post_put_in_pack_hook(package)
            raise UserError(_("Please add 'Done' quantities to the batch picking to create a new pack."))

    def action_view_reception_report(self):
        action = self.picking_ids[0].action_view_reception_report()
        action['context'] = {'default_picking_ids': self.picking_ids.ids}
        return action

    def action_open_label_layout(self):
        if self.env.user.has_group('stock.group_production_lot') and self.move_line_ids.lot_id:
            view = self.env.ref('stock.picking_label_type_form')
            return {
                'name': _('Choose Type of Labels To Print'),
                'type': 'ir.actions.act_window',
                'res_model': 'picking.label.type',
                'views': [(view.id, 'form')],
                'target': 'new',
                'context': {'default_picking_ids': self.picking_ids.ids},
            }
        view = self.env.ref('stock.product_label_layout_form_picking')
        return {
            'name': _('Choose Labels Layout'),
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'product.label.layout',
            'views': [(view.id, 'form')],
            'view_id': view.id,
            'target': 'new',
            'context': {
                'default_product_ids': self.move_line_ids.product_id.ids,
                'default_move_ids': self.move_ids.ids,
                'default_move_quantity': 'move'},
        }

    # -------------------------------------------------------------------------
    # Miscellaneous
    # -------------------------------------------------------------------------
    def _sanity_check(self):
        for batch in self:
            if not batch.picking_ids <= batch.allowed_picking_ids:
                erroneous_pickings = batch.picking_ids - batch.allowed_picking_ids
                raise UserError(_(
                    "The following transfers cannot be added to batch transfer %(batch)s. "
                    "Please check their states and operation types.\n\n"
                    "Incompatibilities: %(incompatible_transfers)s",
                    batch=batch.name,
                    incompatible_transfers=format_list(self.env, erroneous_pickings.mapped('name'))))

    def _track_subtype(self, init_values):
        if 'state' in init_values:
            return self.env.ref('stock_picking_batch.mt_batch_state')
        return super()._track_subtype(init_values)

    def _is_picking_auto_mergeable(self, picking):
        """ Verifies if a picking can be safely inserted into the batch without violating auto_batch_constrains.
        """
        res = True
        if self.picking_type_id.batch_max_lines:
            res = res and (len(self.move_ids) + len(picking.move_ids) <= self.picking_type_id.batch_max_lines)
        if self.picking_type_id.batch_max_pickings:
            res = res and (len(self.picking_ids) + 1 <= self.picking_type_id.batch_max_pickings)
        return res

    def _is_line_auto_mergeable(self, num_of_moves=False, num_of_pickings=False, weight=False):
        """ Verifies if a line can be safely inserted into the wave without violating auto_batch_constrains.
        """
        self.ensure_one()
        res = True
        if num_of_moves:
            res = res and self._are_moves_auto_mergeable(num_of_moves)
        if num_of_pickings:
            res = res and self._are_pickings_auto_mergeable(num_of_pickings)
        return res

    def _are_moves_auto_mergeable(self, num_of_moves):
        self.ensure_one()
        res = True
        if self.picking_type_id.batch_max_lines:
            res = res and (len(self.move_ids) + num_of_moves <= self.picking_type_id.batch_max_lines)
        return res

    def _are_pickings_auto_mergeable(self, num_of_pickings):
        self.ensure_one()
        res = True
        if self.picking_type_id.batch_max_pickings:
            res = res and (len(self.picking_ids) + num_of_pickings <= self.picking_type_id.batch_max_pickings)
        return res

```

## File: models\stock_warehouse.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class StockWarehouse(models.Model):
    _inherit = 'stock.warehouse'

    def _get_picking_type_create_values(self, max_sequence):
        data, next_sequence = super()._get_picking_type_create_values(max_sequence)
        updatable_types = {k: v for (k, v) in data.items() if v.get('code') in ('incoming', 'outgoing')}
        for picking_type in updatable_types.values():
            picking_type.update({
                'auto_batch': True,
                'batch_group_by_partner': True,
            })
        return data, next_sequence

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import stock_move
from . import stock_move_line
from . import stock_picking
from . import stock_picking_batch
from . import stock_warehouse

```

## File: report\report_picking_batch.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <template id="stock_picking_batch.report_picking_batch">
            <t t-call="web.html_container">
                <div class="oe_structure"></div>
                <t t-foreach="docs" t-as="o">
                    <t t-set="move_line_ids" t-value="o.picking_ids.mapped('move_line_ids')"/>
                    <t t-set="has_package" t-value="move_line_ids.filtered('result_package_id')" groups="stock.group_tracking_lot"/>
                    <t t-set="has_serial_number" t-value="move_line_ids.filtered('lot_id')" groups="stock.group_production_lot"/>
                    <t t-set="has_barcode" t-value="move_line_ids.mapped('product_id').filtered('barcode')"/>
                    <t t-set="locations" t-value="move_line_ids.mapped('location_id').sorted(lambda location: location.complete_name)"/>
                    <t t-call="web.external_layout">
                        <div class="page">
                            <div class="d-flex">
                                <div><h3>Summary: <span t-field="o.name">Batch Name</span></h3></div>
                                <div class="me-auto">
                                    <span t-field="o.name" t-options="{'widget': 'barcode', 'width': 600, 'height': 150, 'img_style': 'width:300px;height:50px;'}">Batch</span>
                                </div>
                            </div>
                            <div t-if="o.user_id">
                                <strong>Responsible:</strong>
                                <span t-field="o.user_id">John Doe</span>
                            </div><br/>
                            <div class="oe_structure"></div>
                            <table class="table table-borderless">
                                <thead>
                                    <tr>
                                        <th>Transfer</th>
                                        <th>Barcode</th>
                                        <th>Status</th>
                                        <th>Scheduled Date</th>
                                    </tr>
                                </thead>
                                <tbody>
                                    <tr t-foreach="o.picking_ids.sorted(lambda p: p.batch_sequence)" t-as="pick">
                                        <td>
                                            <span t-field="pick.name">Transfer Name</span>
                                        </td>
                                        <td>
                                            <span t-field="pick.name" t-options="{'widget': 'barcode', 'quiet': 0, 'width': 400, 'height': 100, 'img_style': 'width:200px;height:50px;'}">Transfer</span>
                                        </td>
                                        <td>
                                            <span t-field="pick.state">Gujarat</span>
                                        </td>
                                        <td >
                                            <span t-field="pick.scheduled_date">2023-08-20</span>
                                        </td>
                                    </tr>
                                </tbody>
                            </table>
                            <p style="page-break-after: always;"/>
                            <div class="oe_structure"></div>
                            <h3><span t-field="o.name">Batch Name</span></h3>
                            <div t-if="o.user_id">
                                <strong>Responsible:</strong>
                                <span t-field="o.user_id">John Doe</span>
                            </div><br/>
                            <div class="oe_structure"></div>
                            <table class="table table-borderless">
                                <thead>
                                    <tr>
                                        <th>Product</th>
                                        <th>Quantity</th>
                                        <th width="20%">Transfer</th>
                                        <th t-if="has_serial_number" width="15%">
                                            <strong>Lot/Serial Number</strong>
                                        </th>
                                        <th t-if="has_barcode" width="15%" class="text-center">
                                            <strong>Product Barcode</strong>
                                        </th>
                                        <th t-if="has_package" width="15%">
                                            <strong>Package</strong>
                                        </th>
                                    </tr>
                                </thead>
                                <t t-foreach="locations" t-as="location_from">
                                    <t t-set="loc_move_line" t-value="move_line_ids.filtered(lambda x: x.location_id==location_from)"/>
                                    <t t-foreach="loc_move_line.location_dest_id" t-as="location_dest">
                                        <t t-set="move_lines" t-value="loc_move_line.filtered(lambda x: x.location_dest_id==location_dest).sorted(lambda ml: ml.picking_id.batch_sequence)"/>
                                        <t t-set="products" t-value="move_lines.product_id"/>
                                        <tbody>
                                            <tr>
                                                <td style="background-color:lightgrey" colspan="6" >
                                                    <p style="margin:0px"><strong>FROM</strong>
                                                    <span t-out="move_lines.location_id.display_name">Location From Name</span></p>
                                                    <p style="margin:0px"><strong>TO</strong>
                                                    <span t-out="move_lines.location_dest_id.display_name">Location To Name</span></p>
                                                </td>
                                            </tr>
                                            <tr t-foreach="move_lines" t-as="move_operation">
                                                <td>
                                                    <span t-field="move_operation.display_name">Product Name</span>
                                                </td>
                                                <td>
                                                    <span t-esc="sum(move_operation.mapped('quantity'))">Quantity Done</span>
                                                    <span t-field="move_operation.product_uom_id" groups="uom.group_uom"/>
                                                </td>
                                                <td>
                                                    <span t-out="move_operation.picking_id.display_name">Transfer Display Name</span>
                                                </td>
                                                <td t-if="has_serial_number" class="text-center h6" width="15%">
                                                    <span t-if="move_operation.lot_id or move_operation.lot_name" t-field="move_operation.lot_id.name" t-options="{'widget': 'barcode', 'humanreadable': 1, 'width': 600, 'height': 100, 'img_style': 'width:100%;height:35px;'}">Lot Number</span>
                                                </td>
                                                <td width="15%" class="text-center" t-if="has_barcode">
                                                    <span t-field="move_operation.product_id.barcode" t-options="{'widget': 'barcode', 'symbology': 'auto', 'width': 600, 'height': 100, 'img_style': 'width:100%;height:35px;'}">Product Barcode</span>
                                                </td>
                                                <td t-if="has_package" width="15%">
                                                    <span t-field="move_operation.package_id">Package ID</span>
                                                    <t t-if="move_operation.result_package_id">
                                                        <strong>→</strong> <span t-field="move_operation.result_package_id">Result Package ID</span>
                                                    </t>
                                                </td>
                                            </tr>
                                        </tbody>
                                    </t>
                                </t>
                            </table>
                        </div>
                    </t>
                </t>
            </t>
        </template>
    </data>
</odoo>

```

## File: report\stock_picking_batch_report_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="action_report_picking_batch" model="ir.actions.report">
            <field name="name">Batch Transfer</field>
            <field name="model">stock.picking.batch</field>
            <field name="report_type">qweb-pdf</field>
            <field name="report_name">stock_picking_batch.report_picking_batch</field>
            <field name="report_file">stock_picking_batch.report_picking_batch</field>
            <field name="binding_model_id" ref="model_stock_picking_batch"/>
            <field name="binding_type">report</field>
        </record>
    </data>
</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_stock_picking_batch,stock.picking.batch stock users,model_stock_picking_batch,stock.group_stock_user,1,1,1,1
access_stock_picking_to_batch,access.stock.picking.to.batch,model_stock_picking_to_batch,stock.group_stock_user,1,1,1,0
access_stock_add_to_wave,access.stock.picking.to.wave,model_stock_add_to_wave,stock.group_stock_user,1,1,1,0

```

## File: security\stock_picking_batch_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data noupdate="0">
    <record model="ir.rule" id="stock_picking_batch_multicompany_rule">
        <field name="name">stock.picking.batch multi-company</field>
        <field name="model_id" ref="model_stock_picking_batch"/>
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>
</data>
</odoo>

```

## File: static\shapes\batch-picking.svg

```svg
<svg width="260" height="260" viewBox="0 0 260 260" fill="none" xmlns="http://www.w3.org/2000/svg">
<path d="M0 0H260V260H0V0Z" fill="white"/>
<path d="M100.487 171.928C107.867 176.925 115.722 179.864 124.703 178.885C129.152 178.389 133.322 177.06 137.333 174.625C136.842 173.474 136.306 172.403 135.91 171.3C135.353 169.751 136.072 168.795 137.732 168.913C139.995 169.077 142.277 169.227 144.537 169.544C145.835 169.716 146.567 170.576 146.006 172.01C145.1 174.382 144.257 176.797 143.225 179.102C142.865 179.902 141.993 180.481 141.362 181.16C140.688 180.495 139.954 179.906 139.374 179.168C138.968 178.656 138.785 177.988 138.438 177.28C129.01 182.033 119.411 182.693 109.653 178.653C106.073 177.179 102.816 175.187 100.47 171.926L100.487 171.928Z" fill="#714B67"/>
<path d="M50.2382 162.195C49.8109 163.14 49.5203 164.188 48.8919 164.986C48.5041 165.485 47.6257 165.609 46.9724 165.917C46.7788 165.275 46.2508 164.469 46.4564 163.984C47.5998 161.31 48.8802 158.685 50.2331 156.117C50.844 154.946 52.4775 154.99 53.0757 156.169C54.4633 158.925 55.7942 161.702 57.0397 164.535C57.2348 164.965 56.8376 165.664 56.7081 166.238C56.0944 166.1 55.2609 166.159 54.9199 165.772C54.206 164.949 53.7484 163.904 52.941 162.566C52.8001 166.451 52.7711 169.947 52.5158 173.418C52.2567 176.856 51.7793 180.281 51.3869 183.702C51.1808 183.68 50.9625 183.666 50.7644 183.657C50.7626 176.546 50.773 169.427 50.7632 162.304C50.5895 162.278 50.4279 162.245 50.2542 162.219L50.2382 162.195Z" fill="#714B67"/>
<path d="M145.068 95.2087C144.641 96.1539 144.35 97.2014 143.722 97.9994C143.334 98.4991 142.456 98.6228 141.802 98.9304C141.609 98.2889 141.081 97.483 141.287 96.9982C142.43 94.3236 143.71 91.6987 145.063 89.1309C145.674 87.96 147.308 88.0036 147.906 89.1827C149.293 91.9393 150.624 94.7158 151.87 97.5484C152.065 97.9792 151.668 98.6775 151.538 99.2519C150.924 99.114 150.091 99.1732 149.75 98.786C149.036 97.9626 148.578 96.9182 147.771 95.5796C147.63 99.4644 147.601 102.961 147.346 106.432C147.087 109.87 146.609 113.294 146.217 116.716C146.011 116.694 145.793 116.68 145.595 116.671C145.593 109.56 145.603 102.441 145.593 95.3176C145.42 95.2921 145.258 95.2586 145.084 95.2331L145.068 95.2087Z" fill="#714B67"/>
<path d="M96.8759 110.623C96.4486 109.678 96.158 108.63 95.5296 107.832C95.1418 107.333 94.2634 107.209 93.6101 106.901C93.4165 107.543 92.8885 108.349 93.0941 108.833C94.2375 111.508 95.5179 114.133 96.8708 116.701C97.4817 117.872 99.1152 117.828 99.7134 116.649C101.101 113.892 102.432 111.116 103.677 108.283C103.873 107.852 103.475 107.154 103.346 106.58C102.732 106.718 101.899 106.658 101.558 107.046C100.844 107.869 100.386 108.913 99.5787 110.252C99.4378 106.367 99.4088 102.871 99.1535 99.4001C98.8944 95.9618 98.417 92.5372 98.0246 89.1161C97.8185 89.1377 97.6002 89.1514 97.4021 89.1608C97.4003 96.2718 97.4107 103.391 97.4009 110.514C97.2272 110.54 97.0656 110.573 96.8919 110.599L96.8759 110.623Z" fill="#714B67"/>
<path d="M190.15 110.623C189.723 109.678 189.432 108.63 188.804 107.832C188.416 107.333 187.538 107.209 186.885 106.901C186.691 107.543 186.163 108.349 186.369 108.833C187.512 111.508 188.792 114.133 190.145 116.701C190.756 117.872 192.39 117.828 192.988 116.649C194.375 113.892 195.706 111.116 196.952 108.283C197.147 107.852 196.75 107.154 196.62 106.58C196.006 106.718 195.173 106.658 194.832 107.046C194.118 107.869 193.66 108.913 192.853 110.252C192.712 106.367 192.683 102.871 192.428 99.4001C192.169 95.9618 191.691 92.5372 191.299 89.1161C191.093 89.1377 190.875 89.1514 190.677 89.1608C190.675 96.2718 190.685 103.391 190.675 110.514C190.502 110.54 190.34 110.573 190.166 110.599L190.15 110.623Z" fill="#714B67"/>
<rect x="27.25" y="34.9449" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="199.809" y="34.9449" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="27.25" y="59.8702" width="13.4912" height="15.0782" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="199.809" y="59.8702" width="13.4912" height="15.0782" fill="#1BB6F9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="27.25" y="47.4076" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="199.809" y="47.4076" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="27.25" y="73.8905" width="13.4912" height="13.5204" fill="#00CEB3" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="199.809" y="73.8905" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="27.25" y="86.3533" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="199.809" y="86.3533" width="13.4912" height="13.5204" fill="#00CEB3" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="27.25" y="111.278" width="13.4912" height="13.5204" fill="#FBB130" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="199.809" y="111.278" width="13.4912" height="13.5204" fill="#FBB130" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="27.25" y="137.761" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="199.809" y="137.761" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="27.25" y="98.8157" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="199.809" y="98.8157" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="27.25" y="123.741" width="13.4912" height="15.0782" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="199.809" y="123.741" width="13.4912" height="15.0782" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="27.25" y="150.224" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="199.809" y="150.224" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="61.4512" y="34.9449" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="108.088" y="34.9449" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="154.726" y="34.9449" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="73.8877" y="34.9449" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="120.525" y="34.9449" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="167.162" y="34.9449" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="61.4512" y="59.8702" width="13.4912" height="15.0782" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="108.088" y="59.8702" width="13.4912" height="15.0782" fill="#017E84" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="154.726" y="59.8702" width="13.4912" height="15.0782" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="73.8877" y="59.8702" width="13.4912" height="15.0782" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="120.525" y="59.8702" width="13.4912" height="15.0782" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="167.162" y="59.8702" width="13.4912" height="15.0782" fill="#FBB130" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="61.4512" y="47.4076" width="13.4912" height="13.5204" fill="#FC787D" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="108.088" y="47.4076" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="154.726" y="47.4076" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="73.8877" y="47.4076" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="120.525" y="47.4076" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="167.162" y="47.4076" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="61.4512" y="73.8905" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="108.088" y="73.8905" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="154.726" y="73.8905" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="73.8877" y="73.8905" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="120.525" y="73.8905" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="167.162" y="73.8905" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="61.4512" y="86.3533" width="13.4912" height="13.5204" fill="#FBB130" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="108.088" y="86.3533" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="154.726" y="86.3533" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="73.8877" y="86.3533" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="120.525" y="86.3533" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="167.162" y="86.3533" width="13.4912" height="13.5204" fill="#00CEB3" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="61.4512" y="111.278" width="13.4912" height="13.5204" fill="#D8DADD" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="108.088" y="111.278" width="13.4912" height="13.5204" fill="#D8DADD" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="154.726" y="111.278" width="13.4912" height="13.5204" fill="#D8DADD" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="73.8877" y="111.278" width="13.4912" height="13.5204" fill="#D8DADD" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="120.525" y="111.278" width="13.4912" height="13.5204" fill="#D8DADD" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="167.162" y="111.278" width="13.4912" height="13.5204" fill="#D8DADD" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="61.4512" y="137.761" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="108.088" y="137.761" width="13.4912" height="13.5204" fill="#FBB130" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="154.726" y="137.761" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="73.8877" y="137.761" width="13.4912" height="13.5204" fill="#FC787D" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="120.525" y="137.761" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="167.162" y="137.761" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="61.4512" y="98.8157" width="13.4912" height="13.5204" fill="#1BB6F9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="108.088" y="98.8157" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="154.726" y="98.8157" width="13.4912" height="13.5204" fill="#FC787D" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="73.8877" y="98.8157" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="120.525" y="98.8157" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="167.162" y="98.8157" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="61.4512" y="123.741" width="13.4912" height="15.0782" fill="#00CEB3" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="108.088" y="123.741" width="13.4912" height="15.0782" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="154.726" y="123.741" width="13.4912" height="15.0782" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="73.8877" y="123.741" width="13.4912" height="15.0782" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="120.525" y="123.741" width="13.4912" height="15.0782" fill="#00CEB3" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="167.162" y="123.741" width="13.4912" height="15.0782" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="61.4512" y="150.224" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="108.088" y="150.224" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="154.726" y="150.224" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="73.8877" y="150.224" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="120.525" y="150.224" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="167.162" y="150.224" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<path d="M52.7633 28.5619C60.1435 23.5656 67.9982 20.6257 76.9795 21.6055C81.4286 22.1009 85.5984 23.4302 89.6096 25.8656C89.1183 27.0158 88.5826 28.0871 88.1867 29.1907C87.6295 30.7396 88.348 31.6955 90.008 31.5772C92.2713 31.4129 94.5534 31.263 96.8135 30.9465C98.1109 30.7746 98.8437 29.9145 98.282 28.48C97.3765 26.1077 96.5332 23.6933 95.5012 21.3887C95.1417 20.5886 94.2697 20.0091 93.6385 19.3298C92.9647 19.9955 92.23 20.5846 91.6504 21.3226C91.244 21.8342 91.0614 22.5027 90.7147 23.2099C81.2862 18.4577 71.687 17.7972 61.9295 21.837C58.3497 23.3117 55.0921 25.3033 52.7467 28.5641L52.7633 28.5619Z" fill="#714B67"/>
<path d="M146.039 28.5613C153.419 23.565 161.274 20.6251 170.255 21.6049C174.704 22.1003 178.874 23.4296 182.885 25.865C182.394 27.0152 181.858 28.0864 181.462 29.1901C180.905 30.739 181.623 31.6949 183.283 31.5766C185.547 31.4123 187.829 31.2624 190.089 30.9459C191.386 30.7739 192.119 29.9139 191.557 28.4794C190.652 26.1071 189.809 23.6927 188.777 21.3881C188.417 20.588 187.545 20.0085 186.914 19.3292C186.24 19.9949 185.505 20.584 184.926 21.322C184.519 21.8336 184.337 22.5021 183.99 23.2093C174.562 18.4571 164.962 17.7965 155.205 21.8364C151.625 23.3111 148.367 25.3026 146.022 28.5635L146.039 28.5613Z" fill="#714B67"/>
<rect x="218.6" y="199.093" width="13.4912" height="13.5204" fill="#00CEB3" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="218.6" y="185.093" width="13.4912" height="13.5204" fill="#017E84" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="204.628" y="199.093" width="13.4912" height="13.5204" fill="#FBB130" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="204.628" y="185.093" width="13.4912" height="13.5204" fill="#1BB6F9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="190.657" y="199.093" width="13.4912" height="13.5204" fill="#FC787D" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="148.745" y="199.093" width="13.4912" height="13.5204" fill="#FC787D" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="129.78" y="199.093" width="13.4912" height="13.5204" fill="#FBB130" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="89.8594" y="199.093" width="13.4912" height="13.5204" fill="#017E84" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="115.81" y="199.093" width="13.4912" height="13.5204" fill="#FC787D" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="218.6" y="213.093" width="13.4912" height="13.5204" fill="#00CEB3" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="204.628" y="213.093" width="13.4912" height="13.5204" fill="#FBB130" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="190.657" y="213.093" width="13.4912" height="13.5204" fill="#FC787D" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="148.745" y="213.093" width="13.4912" height="13.5204" fill="#00CEB3" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="96.8457" y="213.093" width="13.4912" height="13.5204" fill="#FBB130" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="129.78" y="213.093" width="13.4912" height="13.5204" fill="#1BB6F9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="82.874" y="213.093" width="13.4912" height="13.5204" fill="#FC787D" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="115.81" y="213.093" width="13.4912" height="13.5204" fill="#00CEB3" stroke="#F3F4F6" stroke-width="0.5"/>
<path d="M181.432 210.92C180.731 210.621 179.958 210.424 179.361 209.974C178.988 209.697 178.883 209.052 178.646 208.575C179.114 208.422 179.696 208.019 180.056 208.163C182.037 208.961 183.985 209.861 185.892 210.815C186.762 211.246 186.757 212.45 185.902 212.911C183.902 213.979 181.886 215.006 179.828 215.972C179.515 216.123 178.996 215.841 178.572 215.756C178.663 215.301 178.606 214.688 178.885 214.43C179.477 213.89 180.236 213.535 181.205 212.918C178.351 212.879 175.785 212.916 173.233 212.786C170.706 212.652 168.184 212.357 165.667 212.125C165.68 211.973 165.686 211.812 165.69 211.665C170.908 211.546 176.133 211.435 181.361 211.309C181.377 211.18 181.398 211.061 181.414 210.932L181.432 210.92Z" fill="#714B67"/>
<path d="M200.649 243.755C200.521 243.755 200.414 243.686 200.329 243.547C200.265 243.483 200.195 243.382 200.121 243.243C200.046 243.105 200.019 242.881 200.041 242.571C200.073 242.251 200.19 241.787 200.393 241.179C200.499 240.774 200.585 240.481 200.649 240.299C200.713 240.107 200.745 239.969 200.745 239.883C200.745 239.766 200.755 239.67 200.777 239.595C200.809 239.51 200.825 239.43 200.825 239.355C200.889 239.355 200.953 239.297 201.017 239.179C201.081 239.062 201.134 238.902 201.177 238.699C201.219 238.518 201.257 238.374 201.289 238.267C201.331 238.15 201.353 238.07 201.353 238.027C201.406 237.942 201.459 237.809 201.513 237.627C201.577 237.446 201.673 237.227 201.801 236.971C201.886 236.651 201.982 236.321 202.089 235.979C202.195 235.627 202.302 235.302 202.409 235.003C202.526 234.694 202.622 234.454 202.697 234.283C202.814 233.953 202.905 233.723 202.969 233.595C203.033 233.457 203.033 233.366 202.969 233.323C202.894 233.27 202.857 233.137 202.857 232.923C202.867 232.699 202.969 232.507 203.161 232.347C203.278 232.209 203.427 232.113 203.609 232.059C203.79 231.995 203.998 231.963 204.233 231.963C204.467 231.963 204.713 231.974 204.969 231.995C205.161 232.006 205.363 232.011 205.577 232.011C205.801 232.011 205.939 232.011 205.993 232.011C206.089 232.054 206.206 232.102 206.345 232.155C206.483 232.209 206.606 232.262 206.713 232.315C206.83 232.358 206.889 232.395 206.889 232.427C206.889 232.491 206.91 232.534 206.953 232.555C207.006 232.566 207.059 232.577 207.113 232.587C207.187 232.587 207.31 232.657 207.481 232.795C207.651 232.923 207.822 233.083 207.993 233.275C208.174 233.457 208.302 233.638 208.377 233.819C208.462 234.001 208.531 234.171 208.585 234.331C208.649 234.491 208.654 234.721 208.601 235.019C208.59 235.179 208.563 235.339 208.521 235.499C208.489 235.659 208.451 235.766 208.409 235.819C208.345 235.862 208.286 235.915 208.233 235.979C208.179 236.043 208.153 236.102 208.153 236.155C208.153 236.198 208.11 236.305 208.025 236.475C207.939 236.635 207.833 236.822 207.705 237.035C207.577 237.238 207.438 237.441 207.289 237.643C207.139 237.846 207.001 238.006 206.873 238.123C206.734 238.294 206.569 238.454 206.377 238.603C206.195 238.753 206.03 238.875 205.881 238.971C205.603 239.099 205.283 239.222 204.921 239.339C204.569 239.457 204.227 239.553 203.897 239.627C203.566 239.691 203.299 239.718 203.097 239.707C202.787 239.75 202.585 239.793 202.489 239.835C202.393 239.867 202.323 239.958 202.281 240.107C202.227 240.246 202.174 240.39 202.121 240.539C202.078 240.689 202.057 240.795 202.057 240.859C202.035 240.902 201.987 241.019 201.913 241.211C201.849 241.403 201.769 241.627 201.673 241.883C201.587 242.139 201.507 242.395 201.433 242.651C201.358 242.907 201.299 243.126 201.257 243.307C201.214 243.457 201.123 243.569 200.985 243.643C200.857 243.718 200.745 243.755 200.649 243.755ZM202.921 238.411C202.963 238.454 203.102 238.459 203.337 238.427C203.571 238.395 203.854 238.315 204.185 238.187C204.398 238.113 204.611 238.022 204.825 237.915C205.049 237.809 205.262 237.691 205.465 237.563C205.571 237.489 205.683 237.398 205.801 237.291C205.918 237.174 206.083 236.982 206.297 236.715C206.329 236.662 206.382 236.561 206.457 236.411C206.542 236.251 206.617 236.102 206.681 235.963C206.777 235.814 206.873 235.595 206.969 235.307C207.065 235.009 207.086 234.763 207.033 234.571C206.979 234.294 206.873 234.065 206.713 233.883C206.553 233.691 206.35 233.553 206.105 233.467C205.87 233.425 205.673 233.398 205.513 233.387C205.363 233.377 205.198 233.387 205.017 233.419C204.846 233.473 204.723 233.547 204.649 233.643C204.585 233.739 204.489 233.942 204.361 234.251C204.286 234.401 204.179 234.609 204.041 234.875C203.902 235.131 203.785 235.393 203.689 235.659C203.497 236.363 203.326 236.902 203.177 237.275C203.038 237.649 202.947 237.894 202.905 238.011C202.915 238.129 202.91 238.214 202.889 238.267C202.867 238.31 202.878 238.358 202.921 238.411ZM207.636 242.507C207.594 242.422 207.498 242.337 207.348 242.251C207.21 242.166 207.14 242.102 207.14 242.059C207.14 242.017 207.114 241.953 207.06 241.867C207.018 241.771 206.975 241.702 206.932 241.659C206.879 241.595 206.895 241.441 206.98 241.195C207.066 240.939 207.194 240.63 207.364 240.267C207.535 239.905 207.722 239.537 207.924 239.163C208.01 239.078 208.095 238.966 208.18 238.827C208.266 238.689 208.33 238.598 208.372 238.555C208.372 238.513 208.388 238.475 208.42 238.443C208.452 238.401 208.49 238.379 208.532 238.379L208.66 238.123C208.682 238.081 208.746 237.995 208.852 237.867C208.97 237.739 209.108 237.601 209.268 237.451C209.428 237.291 209.588 237.142 209.748 237.003C209.919 236.865 210.058 236.763 210.164 236.699C210.388 236.55 210.618 236.481 210.852 236.491C211.098 236.502 211.306 236.577 211.476 236.715C211.508 236.747 211.578 236.811 211.684 236.907C211.802 237.003 211.93 237.11 212.068 237.227C212.218 237.345 212.346 237.451 212.452 237.547L213.044 238.075L212.772 238.875C212.634 239.302 212.596 239.675 212.66 239.995C212.724 240.315 212.826 240.593 212.964 240.827C213.018 240.945 213.087 241.035 213.172 241.099C213.268 241.163 213.391 241.206 213.54 241.227C213.626 241.238 213.706 241.27 213.78 241.323C213.855 241.377 213.892 241.462 213.892 241.579C213.892 241.835 213.844 242.022 213.748 242.139C213.663 242.246 213.567 242.31 213.46 242.331C213.108 242.374 212.772 242.337 212.452 242.219C212.132 242.102 211.844 241.798 211.588 241.307C211.546 241.243 211.487 241.126 211.412 240.955C211.348 240.774 211.311 240.657 211.3 240.603C211.258 240.657 211.199 240.726 211.124 240.811C211.06 240.886 210.996 240.955 210.932 241.019C210.175 241.787 209.524 242.289 208.98 242.523C208.447 242.747 207.999 242.742 207.636 242.507ZM208.516 241.179C208.591 241.19 208.708 241.153 208.868 241.067C209.028 240.971 209.242 240.817 209.508 240.603C209.871 240.294 210.17 240.038 210.404 239.835C210.639 239.633 210.868 239.371 211.092 239.051L211.444 238.475C211.338 238.23 211.242 238.07 211.156 237.995C211.071 237.921 210.986 237.883 210.9 237.883C210.762 237.883 210.596 237.974 210.404 238.155C210.212 238.326 209.994 238.582 209.748 238.923C209.514 239.254 209.247 239.67 208.948 240.171C208.81 240.406 208.703 240.635 208.628 240.859C208.554 241.073 208.516 241.179 208.516 241.179ZM216.329 242.907C216.02 242.961 215.71 242.934 215.401 242.827C215.102 242.721 214.852 242.55 214.649 242.315C214.457 242.07 214.361 241.777 214.361 241.435C214.329 241.115 214.356 240.737 214.441 240.299C214.537 239.851 214.676 239.403 214.857 238.955C215.049 238.507 215.268 238.129 215.513 237.819C215.726 237.563 215.94 237.329 216.153 237.115C216.366 236.902 216.558 236.758 216.729 236.683C216.878 236.619 217.06 236.571 217.273 236.539C217.497 236.497 217.652 236.475 217.737 236.475C217.908 236.518 218.094 236.587 218.297 236.683C218.51 236.779 218.617 236.891 218.617 237.019C218.617 237.019 218.628 237.035 218.649 237.067C218.681 237.089 218.697 237.099 218.697 237.099C218.761 237.099 218.83 237.179 218.905 237.339C218.99 237.489 219.054 237.665 219.097 237.867C219.14 238.059 219.129 238.214 219.065 238.331C219.022 238.534 218.937 238.731 218.809 238.923C218.692 239.115 218.59 239.211 218.505 239.211C218.462 239.211 218.42 239.217 218.377 239.227C218.345 239.227 218.329 239.254 218.329 239.307C218.329 239.361 218.27 239.387 218.153 239.387C218.046 239.387 217.934 239.366 217.817 239.323C217.7 239.281 217.625 239.217 217.593 239.131C217.593 239.078 217.604 239.009 217.625 238.923C217.646 238.838 217.678 238.694 217.721 238.491C217.785 238.193 217.79 237.995 217.737 237.899C217.684 237.793 217.55 237.782 217.337 237.867C217.07 238.017 216.852 238.193 216.681 238.395C216.51 238.587 216.356 238.859 216.217 239.211C216.089 239.563 215.945 240.054 215.785 240.683C215.721 240.929 215.71 241.121 215.753 241.259C215.806 241.398 215.849 241.473 215.881 241.483C216.03 241.526 216.19 241.537 216.361 241.515C216.542 241.483 216.692 241.435 216.809 241.371C216.98 241.286 217.129 241.195 217.257 241.099C217.396 240.993 217.545 240.881 217.705 240.763C217.758 240.731 217.806 240.673 217.849 240.587C217.892 240.491 217.956 240.422 218.041 240.379C218.126 240.251 218.233 240.187 218.361 240.187C218.5 240.177 218.633 240.171 218.761 240.171C218.846 240.193 218.937 240.246 219.033 240.331C219.129 240.417 219.198 240.502 219.241 240.587C219.294 240.662 219.289 240.715 219.225 240.747C219.182 240.747 219.156 240.763 219.145 240.795C219.145 240.817 219.145 240.827 219.145 240.827C219.166 240.87 219.118 240.966 219.001 241.115C218.894 241.265 218.756 241.425 218.585 241.595C218.414 241.766 218.249 241.915 218.089 242.043C217.94 242.161 217.833 242.219 217.769 242.219C217.662 242.219 217.609 242.251 217.609 242.315C217.609 242.337 217.529 242.39 217.369 242.475C217.22 242.561 217.044 242.646 216.841 242.731C216.649 242.817 216.478 242.875 216.329 242.907ZM220.637 242.731C220.477 242.678 220.333 242.593 220.205 242.475C220.088 242.358 220.04 242.166 220.061 241.899C220.082 241.739 220.114 241.574 220.157 241.403C220.2 241.222 220.253 241.046 220.317 240.875C220.349 240.683 220.386 240.491 220.429 240.299C220.482 240.107 220.568 239.926 220.685 239.755C220.824 239.361 220.968 238.971 221.117 238.587C221.266 238.193 221.394 237.782 221.501 237.355C221.661 236.971 221.805 236.63 221.933 236.331C222.072 236.033 222.237 235.649 222.429 235.179C222.536 234.934 222.626 234.662 222.701 234.363C222.786 234.054 222.914 233.782 223.085 233.547C223.16 233.451 223.213 233.371 223.245 233.307C223.288 233.233 223.33 233.163 223.373 233.099C223.448 232.95 223.56 232.881 223.709 232.891C223.869 232.902 223.976 232.945 224.029 233.019C224.21 233.094 224.338 233.233 224.413 233.435C224.488 233.627 224.514 233.766 224.493 233.851C224.493 233.969 224.45 234.097 224.365 234.235C224.322 234.278 224.28 234.342 224.237 234.427C224.194 234.502 224.13 234.603 224.045 234.731C224.002 234.87 223.96 235.009 223.917 235.147C223.874 235.286 223.826 235.414 223.773 235.531C223.72 235.649 223.661 235.766 223.597 235.883C223.533 236.001 223.49 236.097 223.469 236.171C223.373 236.417 223.266 236.657 223.149 236.891C223.042 237.115 222.946 237.355 222.861 237.611C222.776 237.857 222.685 238.118 222.589 238.395C222.493 238.662 222.37 238.918 222.221 239.163C222.146 239.27 222.104 239.414 222.093 239.595C222.093 239.777 222.072 239.953 222.029 240.123C221.954 240.315 221.885 240.491 221.821 240.651C221.768 240.811 221.741 240.913 221.741 240.955C221.666 241.19 221.608 241.425 221.565 241.659C221.522 241.883 221.453 242.081 221.357 242.251C221.357 242.507 221.288 242.667 221.149 242.731C221.01 242.795 220.84 242.795 220.637 242.731ZM224.301 242.875C224.13 242.854 224.002 242.822 223.917 242.779C223.832 242.726 223.672 242.614 223.437 242.443C223.309 242.315 223.149 242.118 222.957 241.851C222.765 241.585 222.573 241.302 222.381 241.003C222.189 240.694 222.018 240.433 221.869 240.219L222.749 239.099C222.888 239.27 223 239.43 223.085 239.579C223.17 239.729 223.24 239.83 223.293 239.883C223.538 240.246 223.73 240.539 223.869 240.763C224.008 240.987 224.125 241.169 224.221 241.307C224.328 241.446 224.445 241.569 224.573 241.675C224.701 241.782 224.866 241.894 225.069 242.011C225.101 242.033 225.133 242.086 225.165 242.171C225.197 242.246 225.208 242.326 225.197 242.411C225.176 242.539 225.096 242.646 224.957 242.731C224.829 242.806 224.61 242.854 224.301 242.875ZM221.981 239.915C221.917 239.734 221.901 239.531 221.933 239.307C221.976 239.073 222.066 238.918 222.205 238.843C222.248 238.769 222.29 238.721 222.333 238.699C222.386 238.667 222.418 238.63 222.429 238.587C222.45 238.545 222.504 238.486 222.589 238.411C222.685 238.337 222.77 238.267 222.845 238.203C223.016 238.097 223.202 237.969 223.405 237.819C223.618 237.659 223.821 237.499 224.013 237.339C224.216 237.179 224.381 237.041 224.509 236.923C224.648 236.806 224.733 236.731 224.765 236.699C224.797 236.657 224.861 236.63 224.957 236.619C225.064 236.609 225.165 236.625 225.261 236.667C225.389 236.699 225.49 236.769 225.565 236.875C225.65 236.982 225.693 237.131 225.693 237.323C225.725 237.441 225.656 237.585 225.485 237.755C225.325 237.915 225.122 238.075 224.877 238.235C224.792 238.289 224.674 238.369 224.525 238.475C224.386 238.582 224.264 238.683 224.157 238.779C224.082 238.822 223.97 238.897 223.821 239.003C223.682 239.099 223.56 239.195 223.453 239.291C223.368 239.345 223.266 239.441 223.149 239.579C223.042 239.707 222.92 239.803 222.781 239.867C222.44 240.187 222.232 240.342 222.157 240.331C222.093 240.321 222.034 240.182 221.981 239.915Z" fill="#714B67"/>
</svg>

```

## File: static\shapes\cluster-picking.svg

```svg
<svg width="260" height="260" viewBox="0 0 260 260" fill="none" xmlns="http://www.w3.org/2000/svg">
<path d="M0 0H260V260H0V0Z" fill="white"/>
<path d="M55.082 152.392C54.6547 153.338 54.3641 154.385 53.7356 155.183C53.3479 155.683 52.4694 155.806 51.8162 156.114C51.6226 155.472 51.0945 154.667 51.3002 154.182C52.4435 151.507 53.724 148.882 55.0768 146.315C55.6878 145.144 57.3212 145.187 57.9194 146.366C59.307 149.123 60.6379 151.899 61.8835 154.732C62.0786 155.163 61.6813 155.861 61.5518 156.436C60.9381 156.298 60.1046 156.357 59.7637 155.97C59.0497 155.146 58.5921 154.102 57.7848 152.763C57.6438 156.648 57.6148 160.145 57.3595 163.615C57.1004 167.053 56.6231 170.478 56.2306 173.899C56.0245 173.878 55.8063 173.864 55.6082 173.854C55.6064 166.743 55.6167 159.624 55.6069 152.501C55.4332 152.476 55.2717 152.442 55.098 152.417L55.082 152.392Z" fill="#714B67"/>
<path d="M150.068 95.2087C149.641 96.1539 149.35 97.2014 148.722 97.9994C148.334 98.4991 147.456 98.6228 146.802 98.9304C146.609 98.2889 146.081 97.483 146.287 96.9982C147.43 94.3236 148.71 91.6987 150.063 89.1309C150.674 87.96 152.308 88.0036 152.906 89.1827C154.293 91.9393 155.624 94.7158 156.87 97.5484C157.065 97.9792 156.668 98.6775 156.538 99.2519C155.924 99.114 155.091 99.1732 154.75 98.786C154.036 97.9626 153.578 96.9182 152.771 95.5796C152.63 99.4644 152.601 102.961 152.346 106.432C152.087 109.87 151.609 113.294 151.217 116.716C151.011 116.694 150.793 116.68 150.595 116.671C150.593 109.56 150.603 102.441 150.593 95.3176C150.42 95.2921 150.258 95.2586 150.084 95.2331L150.068 95.2087Z" fill="#714B67"/>
<path d="M101.876 109.439C101.449 108.494 101.158 107.447 100.53 106.649C100.142 106.149 99.2634 106.025 98.6101 105.718C98.4165 106.359 97.8885 107.165 98.0941 107.65C99.2375 110.324 100.518 112.949 101.871 115.517C102.482 116.688 104.115 116.644 104.713 115.465C106.101 112.709 107.432 109.932 108.677 107.1C108.873 106.669 108.475 105.971 108.346 105.396C107.732 105.534 106.899 105.475 106.558 105.862C105.844 106.685 105.386 107.73 104.579 109.068C104.438 105.184 104.409 101.687 104.153 98.2165C103.894 94.7782 103.417 91.3536 103.025 87.9325C102.818 87.9542 102.6 87.9678 102.402 87.9772C102.4 95.0882 102.411 102.207 102.401 109.33C102.227 109.356 102.066 109.389 101.892 109.415L101.876 109.439Z" fill="#714B67"/>
<path d="M194.791 165.623C194.364 164.678 194.073 163.63 193.445 162.832C193.057 162.333 192.178 162.209 191.525 161.901C191.332 162.543 190.804 163.349 191.009 163.833C192.153 166.508 193.433 169.133 194.786 171.701C195.397 172.872 197.03 172.828 197.628 171.649C199.016 168.892 200.347 166.116 201.592 163.283C201.788 162.852 201.39 162.154 201.261 161.58C200.647 161.718 199.814 161.658 199.473 162.046C198.759 162.869 198.301 163.913 197.494 165.252C197.353 161.367 197.324 157.871 197.069 154.4C196.809 150.962 196.332 147.537 195.94 144.116C195.734 144.138 195.515 144.151 195.317 144.161C195.315 151.272 195.326 158.391 195.316 165.514C195.142 165.54 194.981 165.573 194.807 165.599L194.791 165.623Z" fill="#714B67"/>
<rect x="32.25" y="34.9449" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="204.808" y="34.9449" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="32.25" y="59.8702" width="13.4912" height="15.0782" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="204.808" y="59.8702" width="13.4912" height="15.0782" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="32.25" y="47.4076" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="204.808" y="47.4076" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="32.25" y="73.8905" width="13.4912" height="13.5204" fill="#00CEB3" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="204.808" y="73.8905" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="32.25" y="86.3533" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="204.808" y="86.3533" width="13.4912" height="13.5204" fill="#00CEB3" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="32.25" y="111.278" width="13.4912" height="13.5204" fill="#FBB130" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="204.808" y="111.278" width="13.4912" height="13.5204" fill="#FBB130" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="32.25" y="137.761" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="204.808" y="137.761" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="32.25" y="98.8157" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="204.808" y="98.8157" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="32.25" y="123.741" width="13.4912" height="15.0782" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="204.808" y="123.741" width="13.4912" height="15.0782" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="32.25" y="150.224" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="204.808" y="150.224" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="66.4502" y="34.9449" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="113.088" y="34.9449" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="159.726" y="34.9449" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="78.8867" y="34.9449" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="125.524" y="34.9449" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="172.161" y="34.9449" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="66.4502" y="59.8702" width="13.4912" height="15.0782" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="113.088" y="59.8702" width="13.4912" height="15.0782" fill="#00CEB3" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="159.726" y="59.8702" width="13.4912" height="15.0782" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="78.8867" y="59.8702" width="13.4912" height="15.0782" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="125.524" y="59.8702" width="13.4912" height="15.0782" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="172.161" y="59.8702" width="13.4912" height="15.0782" fill="#FBB130" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="66.4502" y="47.4076" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="113.088" y="47.4076" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="159.726" y="47.4076" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="78.8867" y="47.4076" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="125.524" y="47.4076" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="172.161" y="47.4076" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="66.4502" y="73.8905" width="13.4912" height="13.5204" fill="#FC787D" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="113.088" y="73.8905" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="159.726" y="73.8905" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="78.8867" y="73.8905" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="125.524" y="73.8905" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="172.161" y="73.8905" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="66.4502" y="86.3533" width="13.4912" height="13.5204" fill="#FBB130" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="113.088" y="86.3533" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="159.726" y="86.3533" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="78.8867" y="86.3533" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="125.524" y="86.3533" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="172.161" y="86.3533" width="13.4912" height="13.5204" fill="#00CEB3" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="66.4502" y="111.278" width="13.4912" height="13.5204" fill="#D8DADD" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="113.088" y="111.278" width="13.4912" height="13.5204" fill="#D8DADD" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="159.726" y="111.278" width="13.4912" height="13.5204" fill="#D8DADD" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="78.8867" y="111.278" width="13.4912" height="13.5204" fill="#D8DADD" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="125.524" y="111.278" width="13.4912" height="13.5204" fill="#D8DADD" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="172.161" y="111.278" width="13.4912" height="13.5204" fill="#D8DADD" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="66.4502" y="137.761" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="113.088" y="137.761" width="13.4912" height="13.5204" fill="#FBB130" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="159.726" y="137.761" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="78.8867" y="137.761" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="125.524" y="137.761" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="172.161" y="137.761" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="66.4502" y="98.8157" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="113.088" y="98.8157" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="159.726" y="98.8157" width="13.4912" height="13.5204" fill="#FC787D" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="78.8867" y="98.8157" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="125.524" y="98.8157" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="172.161" y="98.8157" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="66.4502" y="123.741" width="13.4912" height="15.0782" fill="#00CEB3" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="113.088" y="123.741" width="13.4912" height="15.0782" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="159.726" y="123.741" width="13.4912" height="15.0782" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="78.8867" y="123.741" width="13.4912" height="15.0782" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="125.524" y="123.741" width="13.4912" height="15.0782" fill="#00CEB3" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="172.161" y="123.741" width="13.4912" height="15.0782" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="66.4502" y="150.224" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="113.088" y="150.224" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="159.726" y="150.224" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="78.8867" y="150.224" width="13.4912" height="13.5204" fill="#FC787D" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="125.524" y="150.224" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="172.161" y="150.224" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<path d="M57.7633 28.5619C65.1435 23.5656 72.9982 20.6257 81.9795 21.6055C86.4286 22.1009 90.5984 23.4302 94.6096 25.8656C94.1183 27.0158 93.5826 28.0871 93.1867 29.1907C92.6295 30.7396 93.348 31.6955 95.008 31.5772C97.2713 31.4129 99.5534 31.263 101.813 30.9465C103.111 30.7746 103.844 29.9145 103.282 28.48C102.377 26.1077 101.533 23.6933 100.501 21.3887C100.142 20.5886 99.2697 20.0091 98.6385 19.3298C97.9647 19.9955 97.23 20.5846 96.6504 21.3226C96.244 21.8342 96.0614 22.5027 95.7147 23.2099C86.2862 18.4577 76.687 17.7972 66.9295 21.837C63.3497 23.3117 60.0921 25.3033 57.7467 28.5641L57.7633 28.5619Z" fill="#714B67"/>
<path d="M151.038 28.5613C158.418 23.565 166.273 20.6251 175.254 21.6049C179.703 22.1003 183.873 23.4296 187.884 25.865C187.393 27.0152 186.857 28.0864 186.461 29.1901C185.904 30.739 186.622 31.6949 188.282 31.5766C190.546 31.4123 192.828 31.2624 195.088 30.9459C196.385 30.7739 197.118 29.9139 196.556 28.4794C195.651 26.1071 194.808 23.6927 193.776 21.3881C193.416 20.588 192.544 20.0085 191.913 19.3292C191.239 19.9949 190.504 20.584 189.925 21.322C189.518 21.8336 189.336 22.5021 188.989 23.2093C179.561 18.4571 169.961 17.7965 160.204 21.8364C156.624 23.3111 153.366 25.3026 151.021 28.5635L151.038 28.5613Z" fill="#714B67"/>
<path d="M104.4 172.078C111.78 177.074 119.635 180.014 128.616 179.034C133.065 178.539 137.235 177.21 141.246 174.774C140.755 173.624 140.219 172.553 139.823 171.449C139.266 169.9 139.985 168.944 141.645 169.063C143.908 169.227 146.19 169.377 148.45 169.693C149.748 169.865 150.48 170.725 149.919 172.16C149.013 174.532 148.17 176.947 147.138 179.251C146.778 180.051 145.906 180.631 145.275 181.31C144.601 180.644 143.867 180.055 143.287 179.317C142.881 178.806 142.698 178.137 142.351 177.43C132.923 182.182 123.324 182.843 113.566 178.803C109.986 177.328 106.729 175.337 104.383 172.076L104.4 172.078Z" fill="#714B67"/>
<rect x="205.636" y="195.093" width="13.4912" height="13.5204" fill="#00CEB3" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="191.665" y="195.093" width="13.4912" height="13.5204" fill="#FBB130" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="177.694" y="195.093" width="13.4912" height="13.5204" fill="#FC787D" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="205.636" y="209.093" width="13.4912" height="13.5204" fill="#00CEB3" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="191.665" y="209.093" width="13.4912" height="13.5204" fill="#FBB130" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="177.694" y="209.093" width="13.4912" height="13.5204" fill="#FC787D" stroke="#F3F4F6" stroke-width="0.5"/>
<path d="M180.952 242.755C180.824 242.755 180.718 242.686 180.632 242.547C180.568 242.483 180.499 242.382 180.424 242.243C180.35 242.105 180.323 241.881 180.344 241.571C180.376 241.251 180.494 240.787 180.696 240.179C180.803 239.774 180.888 239.481 180.952 239.299C181.016 239.107 181.048 238.969 181.048 238.883C181.048 238.766 181.059 238.67 181.08 238.595C181.112 238.51 181.128 238.43 181.128 238.355C181.192 238.355 181.256 238.297 181.32 238.179C181.384 238.062 181.438 237.902 181.48 237.699C181.523 237.518 181.56 237.374 181.592 237.267C181.635 237.15 181.656 237.07 181.656 237.027C181.71 236.942 181.763 236.809 181.816 236.627C181.88 236.446 181.976 236.227 182.104 235.971C182.19 235.651 182.286 235.321 182.392 234.979C182.499 234.627 182.606 234.302 182.712 234.003C182.83 233.694 182.926 233.454 183 233.283C183.118 232.953 183.208 232.723 183.272 232.595C183.336 232.457 183.336 232.366 183.272 232.323C183.198 232.27 183.16 232.137 183.16 231.923C183.171 231.699 183.272 231.507 183.464 231.347C183.582 231.209 183.731 231.113 183.912 231.059C184.094 230.995 184.302 230.963 184.536 230.963C184.771 230.963 185.016 230.974 185.272 230.995C185.464 231.006 185.667 231.011 185.88 231.011C186.104 231.011 186.243 231.011 186.296 231.011C186.392 231.054 186.51 231.102 186.648 231.155C186.787 231.209 186.91 231.262 187.016 231.315C187.134 231.358 187.192 231.395 187.192 231.427C187.192 231.491 187.214 231.534 187.256 231.555C187.31 231.566 187.363 231.577 187.416 231.587C187.491 231.587 187.614 231.657 187.784 231.795C187.955 231.923 188.126 232.083 188.296 232.275C188.478 232.457 188.606 232.638 188.68 232.819C188.766 233.001 188.835 233.171 188.888 233.331C188.952 233.491 188.958 233.721 188.904 234.019C188.894 234.179 188.867 234.339 188.824 234.499C188.792 234.659 188.755 234.766 188.712 234.819C188.648 234.862 188.59 234.915 188.536 234.979C188.483 235.043 188.456 235.102 188.456 235.155C188.456 235.198 188.414 235.305 188.328 235.475C188.243 235.635 188.136 235.822 188.008 236.035C187.88 236.238 187.742 236.441 187.592 236.643C187.443 236.846 187.304 237.006 187.176 237.123C187.038 237.294 186.872 237.454 186.68 237.603C186.499 237.753 186.334 237.875 186.184 237.971C185.907 238.099 185.587 238.222 185.224 238.339C184.872 238.457 184.531 238.553 184.2 238.627C183.87 238.691 183.603 238.718 183.4 238.707C183.091 238.75 182.888 238.793 182.792 238.835C182.696 238.867 182.627 238.958 182.584 239.107C182.531 239.246 182.478 239.39 182.424 239.539C182.382 239.689 182.36 239.795 182.36 239.859C182.339 239.902 182.291 240.019 182.216 240.211C182.152 240.403 182.072 240.627 181.976 240.883C181.891 241.139 181.811 241.395 181.736 241.651C181.662 241.907 181.603 242.126 181.56 242.307C181.518 242.457 181.427 242.569 181.288 242.643C181.16 242.718 181.048 242.755 180.952 242.755ZM183.224 237.411C183.267 237.454 183.406 237.459 183.64 237.427C183.875 237.395 184.158 237.315 184.488 237.187C184.702 237.113 184.915 237.022 185.128 236.915C185.352 236.809 185.566 236.691 185.768 236.563C185.875 236.489 185.987 236.398 186.104 236.291C186.222 236.174 186.387 235.982 186.6 235.715C186.632 235.662 186.686 235.561 186.76 235.411C186.846 235.251 186.92 235.102 186.984 234.963C187.08 234.814 187.176 234.595 187.272 234.307C187.368 234.009 187.39 233.763 187.336 233.571C187.283 233.294 187.176 233.065 187.016 232.883C186.856 232.691 186.654 232.553 186.408 232.467C186.174 232.425 185.976 232.398 185.816 232.387C185.667 232.377 185.502 232.387 185.32 232.419C185.15 232.473 185.027 232.547 184.952 232.643C184.888 232.739 184.792 232.942 184.664 233.251C184.59 233.401 184.483 233.609 184.344 233.875C184.206 234.131 184.088 234.393 183.992 234.659C183.8 235.363 183.63 235.902 183.48 236.275C183.342 236.649 183.251 236.894 183.208 237.011C183.219 237.129 183.214 237.214 183.192 237.267C183.171 237.31 183.182 237.358 183.224 237.411ZM187.94 241.507C187.897 241.422 187.801 241.337 187.652 241.251C187.513 241.166 187.444 241.102 187.444 241.059C187.444 241.017 187.417 240.953 187.364 240.867C187.321 240.771 187.279 240.702 187.236 240.659C187.183 240.595 187.199 240.441 187.284 240.195C187.369 239.939 187.497 239.63 187.668 239.267C187.839 238.905 188.025 238.537 188.228 238.163C188.313 238.078 188.399 237.966 188.484 237.827C188.569 237.689 188.633 237.598 188.676 237.555C188.676 237.513 188.692 237.475 188.724 237.443C188.756 237.401 188.793 237.379 188.836 237.379L188.964 237.123C188.985 237.081 189.049 236.995 189.156 236.867C189.273 236.739 189.412 236.601 189.572 236.451C189.732 236.291 189.892 236.142 190.052 236.003C190.223 235.865 190.361 235.763 190.468 235.699C190.692 235.55 190.921 235.481 191.156 235.491C191.401 235.502 191.609 235.577 191.78 235.715C191.812 235.747 191.881 235.811 191.988 235.907C192.105 236.003 192.233 236.11 192.372 236.227C192.521 236.345 192.649 236.451 192.756 236.547L193.348 237.075L193.076 237.875C192.937 238.302 192.9 238.675 192.964 238.995C193.028 239.315 193.129 239.593 193.268 239.827C193.321 239.945 193.391 240.035 193.476 240.099C193.572 240.163 193.695 240.206 193.844 240.227C193.929 240.238 194.009 240.27 194.084 240.323C194.159 240.377 194.196 240.462 194.196 240.579C194.196 240.835 194.148 241.022 194.052 241.139C193.967 241.246 193.871 241.31 193.764 241.331C193.412 241.374 193.076 241.337 192.756 241.219C192.436 241.102 192.148 240.798 191.892 240.307C191.849 240.243 191.791 240.126 191.716 239.955C191.652 239.774 191.615 239.657 191.604 239.603C191.561 239.657 191.503 239.726 191.428 239.811C191.364 239.886 191.3 239.955 191.236 240.019C190.479 240.787 189.828 241.289 189.284 241.523C188.751 241.747 188.303 241.742 187.94 241.507ZM188.82 240.179C188.895 240.19 189.012 240.153 189.172 240.067C189.332 239.971 189.545 239.817 189.812 239.603C190.175 239.294 190.473 239.038 190.708 238.835C190.943 238.633 191.172 238.371 191.396 238.051L191.748 237.475C191.641 237.23 191.545 237.07 191.46 236.995C191.375 236.921 191.289 236.883 191.204 236.883C191.065 236.883 190.9 236.974 190.708 237.155C190.516 237.326 190.297 237.582 190.052 237.923C189.817 238.254 189.551 238.67 189.252 239.171C189.113 239.406 189.007 239.635 188.932 239.859C188.857 240.073 188.82 240.179 188.82 240.179ZM196.633 241.907C196.323 241.961 196.014 241.934 195.705 241.827C195.406 241.721 195.155 241.55 194.953 241.315C194.761 241.07 194.665 240.777 194.665 240.435C194.633 240.115 194.659 239.737 194.745 239.299C194.841 238.851 194.979 238.403 195.161 237.955C195.353 237.507 195.571 237.129 195.817 236.819C196.03 236.563 196.243 236.329 196.457 236.115C196.67 235.902 196.862 235.758 197.033 235.683C197.182 235.619 197.363 235.571 197.577 235.539C197.801 235.497 197.955 235.475 198.041 235.475C198.211 235.518 198.398 235.587 198.601 235.683C198.814 235.779 198.921 235.891 198.921 236.019C198.921 236.019 198.931 236.035 198.953 236.067C198.985 236.089 199.001 236.099 199.001 236.099C199.065 236.099 199.134 236.179 199.209 236.339C199.294 236.489 199.358 236.665 199.401 236.867C199.443 237.059 199.433 237.214 199.369 237.331C199.326 237.534 199.241 237.731 199.113 237.923C198.995 238.115 198.894 238.211 198.809 238.211C198.766 238.211 198.723 238.217 198.681 238.227C198.649 238.227 198.633 238.254 198.633 238.307C198.633 238.361 198.574 238.387 198.457 238.387C198.35 238.387 198.238 238.366 198.121 238.323C198.003 238.281 197.929 238.217 197.897 238.131C197.897 238.078 197.907 238.009 197.929 237.923C197.95 237.838 197.982 237.694 198.025 237.491C198.089 237.193 198.094 236.995 198.041 236.899C197.987 236.793 197.854 236.782 197.641 236.867C197.374 237.017 197.155 237.193 196.985 237.395C196.814 237.587 196.659 237.859 196.521 238.211C196.393 238.563 196.249 239.054 196.089 239.683C196.025 239.929 196.014 240.121 196.057 240.259C196.11 240.398 196.153 240.473 196.185 240.483C196.334 240.526 196.494 240.537 196.665 240.515C196.846 240.483 196.995 240.435 197.113 240.371C197.283 240.286 197.433 240.195 197.561 240.099C197.699 239.993 197.849 239.881 198.009 239.763C198.062 239.731 198.11 239.673 198.153 239.587C198.195 239.491 198.259 239.422 198.345 239.379C198.43 239.251 198.537 239.187 198.665 239.187C198.803 239.177 198.937 239.171 199.065 239.171C199.15 239.193 199.241 239.246 199.337 239.331C199.433 239.417 199.502 239.502 199.545 239.587C199.598 239.662 199.593 239.715 199.529 239.747C199.486 239.747 199.459 239.763 199.449 239.795C199.449 239.817 199.449 239.827 199.449 239.827C199.47 239.87 199.422 239.966 199.305 240.115C199.198 240.265 199.059 240.425 198.889 240.595C198.718 240.766 198.553 240.915 198.393 241.043C198.243 241.161 198.137 241.219 198.073 241.219C197.966 241.219 197.913 241.251 197.913 241.315C197.913 241.337 197.833 241.39 197.673 241.475C197.523 241.561 197.347 241.646 197.145 241.731C196.953 241.817 196.782 241.875 196.633 241.907ZM200.941 241.731C200.781 241.678 200.637 241.593 200.509 241.475C200.391 241.358 200.343 241.166 200.365 240.899C200.386 240.739 200.418 240.574 200.461 240.403C200.503 240.222 200.557 240.046 200.621 239.875C200.653 239.683 200.69 239.491 200.733 239.299C200.786 239.107 200.871 238.926 200.989 238.755C201.127 238.361 201.271 237.971 201.421 237.587C201.57 237.193 201.698 236.782 201.805 236.355C201.965 235.971 202.109 235.63 202.237 235.331C202.375 235.033 202.541 234.649 202.733 234.179C202.839 233.934 202.93 233.662 203.005 233.363C203.09 233.054 203.218 232.782 203.389 232.547C203.463 232.451 203.517 232.371 203.549 232.307C203.591 232.233 203.634 232.163 203.677 232.099C203.751 231.95 203.863 231.881 204.013 231.891C204.173 231.902 204.279 231.945 204.333 232.019C204.514 232.094 204.642 232.233 204.717 232.435C204.791 232.627 204.818 232.766 204.797 232.851C204.797 232.969 204.754 233.097 204.669 233.235C204.626 233.278 204.583 233.342 204.541 233.427C204.498 233.502 204.434 233.603 204.349 233.731C204.306 233.87 204.263 234.009 204.221 234.147C204.178 234.286 204.13 234.414 204.077 234.531C204.023 234.649 203.965 234.766 203.901 234.883C203.837 235.001 203.794 235.097 203.773 235.171C203.677 235.417 203.57 235.657 203.453 235.891C203.346 236.115 203.25 236.355 203.165 236.611C203.079 236.857 202.989 237.118 202.893 237.395C202.797 237.662 202.674 237.918 202.525 238.163C202.45 238.27 202.407 238.414 202.397 238.595C202.397 238.777 202.375 238.953 202.333 239.123C202.258 239.315 202.189 239.491 202.125 239.651C202.071 239.811 202.045 239.913 202.045 239.955C201.97 240.19 201.911 240.425 201.869 240.659C201.826 240.883 201.757 241.081 201.661 241.251C201.661 241.507 201.591 241.667 201.453 241.731C201.314 241.795 201.143 241.795 200.941 241.731ZM204.605 241.875C204.434 241.854 204.306 241.822 204.221 241.779C204.135 241.726 203.975 241.614 203.741 241.443C203.613 241.315 203.453 241.118 203.261 240.851C203.069 240.585 202.877 240.302 202.685 240.003C202.493 239.694 202.322 239.433 202.173 239.219L203.053 238.099C203.191 238.27 203.303 238.43 203.389 238.579C203.474 238.729 203.543 238.83 203.597 238.883C203.842 239.246 204.034 239.539 204.173 239.763C204.311 239.987 204.429 240.169 204.525 240.307C204.631 240.446 204.749 240.569 204.877 240.675C205.005 240.782 205.17 240.894 205.373 241.011C205.405 241.033 205.437 241.086 205.469 241.171C205.501 241.246 205.511 241.326 205.501 241.411C205.479 241.539 205.399 241.646 205.261 241.731C205.133 241.806 204.914 241.854 204.605 241.875ZM202.285 238.915C202.221 238.734 202.205 238.531 202.237 238.307C202.279 238.073 202.37 237.918 202.509 237.843C202.551 237.769 202.594 237.721 202.637 237.699C202.69 237.667 202.722 237.63 202.733 237.587C202.754 237.545 202.807 237.486 202.893 237.411C202.989 237.337 203.074 237.267 203.149 237.203C203.319 237.097 203.506 236.969 203.709 236.819C203.922 236.659 204.125 236.499 204.317 236.339C204.519 236.179 204.685 236.041 204.813 235.923C204.951 235.806 205.037 235.731 205.069 235.699C205.101 235.657 205.165 235.63 205.261 235.619C205.367 235.609 205.469 235.625 205.565 235.667C205.693 235.699 205.794 235.769 205.869 235.875C205.954 235.982 205.997 236.131 205.997 236.323C206.029 236.441 205.959 236.585 205.789 236.755C205.629 236.915 205.426 237.075 205.181 237.235C205.095 237.289 204.978 237.369 204.829 237.475C204.69 237.582 204.567 237.683 204.461 237.779C204.386 237.822 204.274 237.897 204.125 238.003C203.986 238.099 203.863 238.195 203.757 238.291C203.671 238.345 203.57 238.441 203.453 238.579C203.346 238.707 203.223 238.803 203.085 238.867C202.743 239.187 202.535 239.342 202.461 239.331C202.397 239.321 202.338 239.182 202.285 238.915ZM208.125 242.131C207.538 242.131 207.085 241.961 206.765 241.619C206.456 241.278 206.301 240.755 206.301 240.051C206.301 239.667 206.37 239.246 206.509 238.787C206.658 238.329 206.85 237.881 207.085 237.443C207.33 237.006 207.597 236.627 207.885 236.307C208.173 235.977 208.461 235.747 208.749 235.619C208.856 235.566 208.968 235.529 209.085 235.507C209.213 235.475 209.336 235.459 209.453 235.459C209.581 235.459 209.72 235.47 209.869 235.491C210.029 235.502 210.178 235.534 210.317 235.587C210.562 235.662 210.776 235.806 210.957 236.019C211.149 236.233 211.245 236.542 211.245 236.947C211.245 237.054 211.234 237.171 211.213 237.299C211.192 237.417 211.154 237.534 211.101 237.651C210.962 238.003 210.749 238.339 210.461 238.659C210.184 238.979 209.89 239.203 209.581 239.331C209.496 239.374 209.41 239.406 209.325 239.427C209.25 239.438 209.17 239.443 209.085 239.443C208.861 239.443 208.658 239.411 208.477 239.347C208.306 239.283 208.146 239.198 207.997 239.091L207.869 238.979L207.789 239.139C207.714 239.342 207.661 239.513 207.629 239.651C207.608 239.79 207.597 239.891 207.597 239.955C207.597 240.083 207.602 240.217 207.613 240.355C207.624 240.483 207.656 240.59 207.709 240.675C207.752 240.729 207.805 240.771 207.869 240.803C207.944 240.835 208.034 240.851 208.141 240.851C208.205 240.851 208.264 240.851 208.317 240.851C208.381 240.841 208.434 240.83 208.477 240.819C208.669 240.777 208.824 240.707 208.941 240.611C209.069 240.505 209.202 240.414 209.341 240.339L209.405 240.307L209.389 240.259C209.57 240.163 209.693 240.105 209.757 240.083C209.821 240.051 209.869 240.025 209.901 240.003C209.944 239.961 209.992 239.939 210.045 239.939C210.194 239.918 210.338 239.961 210.477 240.067C210.626 240.163 210.701 240.286 210.701 240.435C210.701 240.499 210.653 240.601 210.557 240.739C210.472 240.878 210.36 241.017 210.221 241.155C210.093 241.283 209.96 241.385 209.821 241.459C209.789 241.47 209.757 241.486 209.725 241.507C209.693 241.518 209.661 241.534 209.629 241.555H209.645C209.549 241.683 209.453 241.747 209.357 241.747H209.229C209.186 241.822 209.048 241.902 208.813 241.987C208.578 242.083 208.349 242.131 208.125 242.131ZM208.957 238.275C209.053 238.275 209.138 238.259 209.213 238.227C209.298 238.195 209.378 238.158 209.453 238.115C209.709 237.934 209.874 237.699 209.949 237.411C210.024 237.123 209.997 236.905 209.869 236.755C209.848 236.723 209.821 236.702 209.789 236.691C209.757 236.67 209.72 236.659 209.677 236.659C209.56 236.659 209.426 236.697 209.277 236.771C209.138 236.846 209 236.953 208.861 237.091C208.722 237.23 208.594 237.395 208.477 237.587L208.269 237.923C208.29 237.913 208.306 237.907 208.317 237.907C208.338 237.897 208.354 237.891 208.365 237.891C208.461 237.891 208.525 237.923 208.557 237.987C208.589 238.051 208.626 238.115 208.669 238.179C208.722 238.243 208.818 238.275 208.957 238.275ZM215.965 242.035C215.848 242.078 215.714 242.099 215.565 242.099C215.426 242.11 215.266 242.046 215.085 241.907C214.978 241.79 214.904 241.667 214.861 241.539C214.829 241.411 214.813 241.214 214.813 240.947C214.834 240.787 214.877 240.558 214.941 240.259C215.005 239.95 215.074 239.63 215.149 239.299C215.234 238.958 215.309 238.654 215.373 238.387C215.448 238.11 215.506 237.923 215.549 237.827C215.624 237.571 215.64 237.411 215.597 237.347C215.554 237.283 215.426 237.246 215.213 237.235C214.968 237.225 214.712 237.342 214.445 237.587C214.178 237.833 213.874 238.222 213.533 238.755C213.277 239.15 213.122 239.475 213.069 239.731C213.016 239.977 213.01 240.147 213.053 240.243C213.074 240.286 213.112 240.313 213.165 240.323C213.229 240.323 213.304 240.307 213.389 240.275C213.485 240.243 213.586 240.201 213.693 240.147C213.768 240.105 213.858 240.062 213.965 240.019C214.082 239.966 214.194 239.918 214.301 239.875C214.365 239.875 214.408 239.913 214.429 239.987C214.461 240.062 214.477 240.153 214.477 240.259C214.477 240.302 214.488 240.382 214.509 240.499C214.541 240.617 214.594 240.697 214.669 240.739C214.712 240.782 214.621 240.873 214.397 241.011C214.184 241.139 213.906 241.299 213.565 241.491C213.245 241.566 213.005 241.603 212.845 241.603C212.696 241.603 212.53 241.534 212.349 241.395C212.05 241.214 211.858 240.878 211.773 240.387C211.698 239.897 211.762 239.353 211.965 238.755C212.168 238.275 212.392 237.865 212.637 237.523C212.893 237.171 213.16 236.851 213.437 236.563C213.48 236.521 213.613 236.435 213.837 236.307C214.061 236.179 214.29 236.078 214.525 236.003C214.792 235.918 215.016 235.886 215.197 235.907C215.389 235.918 215.57 235.982 215.741 236.099C215.933 236.142 216.061 236.137 216.125 236.083C216.2 236.03 216.285 235.881 216.381 235.635C216.573 235.102 216.776 234.563 216.989 234.019C217.202 233.465 217.426 232.905 217.661 232.339C217.768 232.137 217.832 231.982 217.853 231.875C217.885 231.758 217.922 231.678 217.965 231.635L218.413 231.491C218.53 231.459 218.664 231.513 218.813 231.651C218.973 231.79 219.074 231.923 219.117 232.051C219.117 232.147 219.074 232.361 218.989 232.691C218.904 233.011 218.765 233.395 218.573 233.843C218.402 234.238 218.232 234.638 218.061 235.043C217.901 235.438 217.746 235.859 217.597 236.307C217.48 236.638 217.373 236.905 217.277 237.107C217.181 237.31 217.117 237.486 217.085 237.635C217.021 237.913 216.941 238.163 216.845 238.387C216.76 238.601 216.701 238.75 216.669 238.835C216.53 239.241 216.424 239.577 216.349 239.843C216.274 240.099 216.226 240.323 216.205 240.515C216.184 240.697 216.178 240.878 216.189 241.059C216.2 241.23 216.221 241.427 216.253 241.651C216.253 241.694 216.221 241.758 216.157 241.843C216.104 241.918 216.04 241.982 215.965 242.035Z" fill="#714B67"/>
</svg>

```

## File: static\shapes\wave-picking.svg

```svg
<svg width="260" height="260" viewBox="0 0 260 260" fill="none" xmlns="http://www.w3.org/2000/svg">
<path d="M0 0H260V260H0V0Z" fill="white"/>
<path d="M128.876 109.745C128.449 108.799 128.158 107.752 127.53 106.954C127.142 106.454 126.263 106.33 125.61 106.023C125.417 106.664 124.888 107.47 125.094 107.955C126.237 110.63 127.518 113.255 128.871 115.822C129.482 116.993 131.115 116.95 131.713 115.771C133.101 113.014 134.432 110.237 135.677 107.405C135.873 106.974 135.475 106.276 135.346 105.701C134.732 105.839 133.899 105.78 133.558 106.167C132.844 106.991 132.386 108.035 131.579 109.374C131.438 105.489 131.409 101.992 131.153 98.5216C130.894 95.0834 130.417 91.6588 130.025 88.2377C129.818 88.2593 129.6 88.273 129.402 88.2824C129.4 95.3934 129.411 102.512 129.401 109.636C129.227 109.661 129.066 109.695 128.892 109.72L128.876 109.745Z" fill="#714B67"/>
<path d="M82.082 108.928C81.6547 107.983 81.3641 106.935 80.7356 106.137C80.3479 105.638 79.4694 105.514 78.8162 105.206C78.6226 105.848 78.0945 106.654 78.3002 107.139C79.4435 109.813 80.724 112.438 82.0768 115.006C82.6878 116.177 84.3212 116.133 84.9194 114.954C86.307 112.198 87.6379 109.421 88.8835 106.588C89.0786 106.158 88.6813 105.459 88.5518 104.885C87.9381 105.023 87.1046 104.964 86.7637 105.351C86.0497 106.174 85.5921 107.219 84.7848 108.557C84.6438 104.672 84.6148 101.176 84.3595 97.7052C84.1004 94.267 83.6231 90.8424 83.2306 87.4213C83.0245 87.4429 82.8063 87.4566 82.6082 87.466C82.6064 94.577 82.6167 101.696 82.6069 108.819C82.4332 108.845 82.2717 108.878 82.098 108.904L82.082 108.928Z" fill="#714B67"/>
<path d="M175.887 110.928C175.459 109.983 175.169 108.935 174.54 108.137C174.153 107.638 173.274 107.514 172.621 107.206C172.427 107.848 171.899 108.654 172.105 109.139C173.248 111.813 174.529 114.438 175.882 117.006C176.492 118.177 178.126 118.133 178.724 116.954C180.112 114.198 181.443 111.421 182.688 108.588C182.883 108.158 182.486 107.459 182.357 106.885C181.743 107.023 180.909 106.964 180.568 107.351C179.854 108.174 179.397 109.219 178.589 110.557C178.449 106.672 178.419 103.176 178.164 99.7052C177.905 96.267 177.428 92.8424 177.035 89.4213C176.829 89.4429 176.611 89.4566 176.413 89.466C176.411 96.577 176.421 103.696 176.412 110.819C176.238 110.845 176.076 110.878 175.903 110.904L175.887 110.928Z" fill="#714B67"/>
<rect x="59.25" y="35.25" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="59.25" y="60.1754" width="13.4912" height="15.0782" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="59.25" y="47.7126" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="59.25" y="74.1957" width="13.4912" height="13.5204" fill="#00CEB3" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="59.25" y="86.6583" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="59.25" y="111.583" width="13.4912" height="13.5204" fill="#FBB130" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="59.25" y="138.066" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="59.25" y="99.1207" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="59.25" y="124.046" width="13.4912" height="15.0782" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="59.25" y="150.529" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="93.4502" y="35.25" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="140.088" y="35.25" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="186.726" y="35.25" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="105.888" y="35.25" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="152.524" y="35.25" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="93.4502" y="60.1754" width="13.4912" height="15.0782" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="140.088" y="60.1754" width="13.4912" height="15.0782" fill="#017E84" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="186.726" y="60.1754" width="13.4912" height="15.0782" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="105.888" y="60.1754" width="13.4912" height="15.0782" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="152.524" y="60.1754" width="13.4912" height="15.0782" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="93.4502" y="47.7126" width="13.4912" height="13.5204" fill="#FC787D" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="140.088" y="47.7126" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="186.726" y="47.7126" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="105.888" y="47.7126" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="152.524" y="47.7126" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="93.4502" y="74.1957" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="140.088" y="74.1957" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="186.726" y="74.1957" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="105.888" y="74.1957" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="152.524" y="74.1957" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="93.4502" y="86.6583" width="13.4912" height="13.5204" fill="#FBB130" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="140.088" y="86.6583" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="186.726" y="86.6583" width="13.4912" height="13.5204" fill="#FC787D" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="105.888" y="86.6583" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="152.524" y="86.6583" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="93.4502" y="111.583" width="13.4912" height="13.5204" fill="#D8DADD" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="140.088" y="111.583" width="13.4912" height="13.5204" fill="#D8DADD" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="186.726" y="111.583" width="13.4912" height="13.5204" fill="#D8DADD" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="105.888" y="111.583" width="13.4912" height="13.5204" fill="#D8DADD" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="152.524" y="111.583" width="13.4912" height="13.5204" fill="#D8DADD" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="93.4502" y="138.066" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="140.088" y="138.066" width="13.4912" height="13.5204" fill="#FBB130" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="186.726" y="138.066" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="105.888" y="138.066" width="13.4912" height="13.5204" fill="#FC787D" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="152.524" y="138.066" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="93.4502" y="99.1207" width="13.4912" height="13.5204" fill="#1BB6F9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="140.088" y="99.1207" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="186.726" y="99.1207" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="105.888" y="99.1207" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="152.524" y="99.1207" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="93.4502" y="124.046" width="13.4912" height="15.0782" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="140.088" y="124.046" width="13.4912" height="15.0782" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="186.726" y="124.046" width="13.4912" height="15.0782" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="105.888" y="124.046" width="13.4912" height="15.0782" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="152.524" y="124.046" width="13.4912" height="15.0782" fill="#00CEB3" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="93.4502" y="150.529" width="13.4912" height="13.5204" fill="#00CEB3" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="140.088" y="150.529" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="186.726" y="150.529" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="105.888" y="150.529" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="152.524" y="150.529" width="13.4912" height="13.5204" fill="#D9D9D9" stroke="#F3F4F6" stroke-width="0.5"/>
<path d="M173.031 208.437C172.543 208.211 172.001 208.057 171.59 207.727C171.332 207.523 171.271 207.066 171.114 206.725C171.447 206.626 171.866 206.354 172.117 206.463C173.499 207.067 174.855 207.742 176.181 208.454C176.786 208.776 176.758 209.626 176.145 209.934C174.712 210.647 173.268 211.33 171.796 211.969C171.572 212.069 171.212 211.86 170.914 211.79C170.988 211.471 170.96 211.038 171.162 210.861C171.591 210.493 172.134 210.258 172.83 209.842C170.817 209.756 169.005 209.729 167.208 209.585C165.427 209.438 163.653 209.179 161.882 208.963C161.894 208.856 161.901 208.742 161.907 208.639C165.592 208.662 169.282 208.691 172.973 208.71C172.987 208.619 173.005 208.535 173.019 208.445L173.031 208.437Z" fill="#714B67"/>
<path d="M132.116 208.437C131.628 208.211 131.086 208.057 130.675 207.727C130.417 207.523 130.356 207.066 130.199 206.725C130.532 206.626 130.951 206.354 131.202 206.463C132.584 207.067 133.94 207.742 135.266 208.454C135.871 208.776 135.843 209.626 135.23 209.934C133.797 210.647 132.353 211.33 130.881 211.969C130.657 212.069 130.297 211.86 129.999 211.79C130.073 211.471 130.045 211.038 130.247 210.861C130.676 210.493 131.219 210.258 131.915 209.842C129.902 209.756 128.09 209.729 126.293 209.585C124.512 209.438 122.738 209.179 120.967 208.963C120.979 208.856 120.986 208.742 120.992 208.639C124.677 208.662 128.367 208.691 132.058 208.71C132.072 208.619 132.09 208.535 132.104 208.445L132.116 208.437Z" fill="#714B67"/>
<path d="M79.2267 208.437C78.7383 208.211 78.1964 208.057 77.785 207.727C77.5273 207.523 77.4661 207.066 77.3089 206.725C77.642 206.626 78.0614 206.354 78.312 206.463C79.6942 207.067 81.0504 207.742 82.3766 208.454C82.9814 208.776 82.9534 209.626 82.3403 209.934C80.907 210.647 79.4637 211.33 77.9915 211.969C77.7676 212.069 77.4071 211.86 77.1098 211.79C77.1833 211.471 77.1554 211.038 77.3572 210.861C77.7863 210.493 78.3291 210.258 79.0255 209.842C77.0127 209.756 75.2007 209.729 73.4029 209.585C71.6219 209.438 69.8487 209.179 68.077 208.963C68.0889 208.856 68.0967 208.742 68.1022 208.639C71.7875 208.662 75.4769 208.691 79.1686 208.71C79.1823 208.619 79.2002 208.535 79.214 208.445L79.2267 208.437Z" fill="#714B67"/>
<rect x="207.896" y="195.25" width="13.4912" height="13.5204" fill="#00CEB3" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="207.896" y="181.25" width="13.4912" height="13.5204" fill="#017E84" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="193.925" y="195.25" width="13.4912" height="13.5204" fill="#FBB130" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="193.925" y="181.25" width="13.4912" height="13.5204" fill="#1BB6F9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="179.954" y="195.25" width="13.4912" height="13.5204" fill="#FC787D" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="143.031" y="195.25" width="13.4912" height="13.5204" fill="#FC787D" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="52.2207" y="195.25" width="13.4912" height="13.5204" fill="#FBB130" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="94.1328" y="195.25" width="13.4912" height="13.5204" fill="#017E84" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="38.25" y="195.25" width="13.4912" height="13.5204" fill="#FC787D" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="207.896" y="209.25" width="13.4912" height="13.5204" fill="#00CEB3" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="193.925" y="209.25" width="13.4912" height="13.5204" fill="#FBB130" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="179.954" y="209.25" width="13.4912" height="13.5204" fill="#FC787D" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="143.031" y="209.25" width="13.4912" height="13.5204" fill="#00CEB3" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="101.119" y="209.25" width="13.4912" height="13.5204" fill="#FBB130" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="52.2207" y="209.25" width="13.4912" height="13.5204" fill="#1BB6F9" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="87.1475" y="209.25" width="13.4912" height="13.5204" fill="#FC787D" stroke="#F3F4F6" stroke-width="0.5"/>
<rect x="38.25" y="209.25" width="13.4912" height="13.5204" fill="#00CEB3" stroke="#F3F4F6" stroke-width="0.5"/>
<path d="M188.942 241.912C188.814 241.912 188.707 241.843 188.622 241.704C188.558 241.64 188.488 241.539 188.414 241.4C188.339 241.261 188.312 241.037 188.334 240.728C188.366 240.408 188.483 239.944 188.686 239.336C188.792 238.931 188.878 238.637 188.942 238.456C189.006 238.264 189.038 238.125 189.038 238.04C189.038 237.923 189.048 237.827 189.07 237.752C189.102 237.667 189.118 237.587 189.118 237.512C189.182 237.512 189.246 237.453 189.31 237.336C189.374 237.219 189.427 237.059 189.47 236.856C189.512 236.675 189.55 236.531 189.582 236.424C189.624 236.307 189.646 236.227 189.646 236.184C189.699 236.099 189.752 235.965 189.806 235.784C189.87 235.603 189.966 235.384 190.094 235.128C190.179 234.808 190.275 234.477 190.382 234.136C190.488 233.784 190.595 233.459 190.702 233.16C190.819 232.851 190.915 232.611 190.99 232.44C191.107 232.109 191.198 231.88 191.262 231.752C191.326 231.613 191.326 231.523 191.262 231.48C191.187 231.427 191.15 231.293 191.15 231.08C191.16 230.856 191.262 230.664 191.454 230.504C191.571 230.365 191.72 230.269 191.902 230.216C192.083 230.152 192.291 230.12 192.526 230.12C192.76 230.12 193.006 230.131 193.262 230.152C193.454 230.163 193.656 230.168 193.87 230.168C194.094 230.168 194.232 230.168 194.286 230.168C194.382 230.211 194.499 230.259 194.638 230.312C194.776 230.365 194.899 230.419 195.006 230.472C195.123 230.515 195.182 230.552 195.182 230.584C195.182 230.648 195.203 230.691 195.246 230.712C195.299 230.723 195.352 230.733 195.406 230.744C195.48 230.744 195.603 230.813 195.774 230.952C195.944 231.08 196.115 231.24 196.286 231.432C196.467 231.613 196.595 231.795 196.67 231.976C196.755 232.157 196.824 232.328 196.878 232.488C196.942 232.648 196.947 232.877 196.894 233.176C196.883 233.336 196.856 233.496 196.814 233.656C196.782 233.816 196.744 233.923 196.702 233.976C196.638 234.019 196.579 234.072 196.526 234.136C196.472 234.2 196.446 234.259 196.446 234.312C196.446 234.355 196.403 234.461 196.318 234.632C196.232 234.792 196.126 234.979 195.998 235.192C195.87 235.395 195.731 235.597 195.582 235.8C195.432 236.003 195.294 236.163 195.166 236.28C195.027 236.451 194.862 236.611 194.67 236.76C194.488 236.909 194.323 237.032 194.174 237.128C193.896 237.256 193.576 237.379 193.214 237.496C192.862 237.613 192.52 237.709 192.19 237.784C191.859 237.848 191.592 237.875 191.39 237.864C191.08 237.907 190.878 237.949 190.782 237.992C190.686 238.024 190.616 238.115 190.574 238.264C190.52 238.403 190.467 238.547 190.414 238.696C190.371 238.845 190.35 238.952 190.35 239.016C190.328 239.059 190.28 239.176 190.206 239.368C190.142 239.56 190.062 239.784 189.966 240.04C189.88 240.296 189.8 240.552 189.726 240.808C189.651 241.064 189.592 241.283 189.55 241.464C189.507 241.613 189.416 241.725 189.278 241.8C189.15 241.875 189.038 241.912 188.942 241.912ZM191.214 236.568C191.256 236.611 191.395 236.616 191.63 236.584C191.864 236.552 192.147 236.472 192.478 236.344C192.691 236.269 192.904 236.179 193.118 236.072C193.342 235.965 193.555 235.848 193.758 235.72C193.864 235.645 193.976 235.555 194.094 235.448C194.211 235.331 194.376 235.139 194.59 234.872C194.622 234.819 194.675 234.717 194.75 234.568C194.835 234.408 194.91 234.259 194.974 234.12C195.07 233.971 195.166 233.752 195.262 233.464C195.358 233.165 195.379 232.92 195.326 232.728C195.272 232.451 195.166 232.221 195.006 232.04C194.846 231.848 194.643 231.709 194.398 231.624C194.163 231.581 193.966 231.555 193.806 231.544C193.656 231.533 193.491 231.544 193.31 231.576C193.139 231.629 193.016 231.704 192.942 231.8C192.878 231.896 192.782 232.099 192.654 232.408C192.579 232.557 192.472 232.765 192.334 233.032C192.195 233.288 192.078 233.549 191.982 233.816C191.79 234.52 191.619 235.059 191.47 235.432C191.331 235.805 191.24 236.051 191.198 236.168C191.208 236.285 191.203 236.371 191.182 236.424C191.16 236.467 191.171 236.515 191.214 236.568ZM195.929 240.664C195.887 240.579 195.791 240.493 195.641 240.408C195.503 240.323 195.433 240.259 195.433 240.216C195.433 240.173 195.407 240.109 195.353 240.024C195.311 239.928 195.268 239.859 195.225 239.816C195.172 239.752 195.188 239.597 195.273 239.352C195.359 239.096 195.487 238.787 195.657 238.424C195.828 238.061 196.015 237.693 196.217 237.32C196.303 237.235 196.388 237.123 196.473 236.984C196.559 236.845 196.623 236.755 196.665 236.712C196.665 236.669 196.681 236.632 196.713 236.6C196.745 236.557 196.783 236.536 196.825 236.536L196.953 236.28C196.975 236.237 197.039 236.152 197.145 236.024C197.263 235.896 197.401 235.757 197.561 235.608C197.721 235.448 197.881 235.299 198.041 235.16C198.212 235.021 198.351 234.92 198.457 234.856C198.681 234.707 198.911 234.637 199.145 234.648C199.391 234.659 199.599 234.733 199.769 234.872C199.801 234.904 199.871 234.968 199.977 235.064C200.095 235.16 200.223 235.267 200.361 235.384C200.511 235.501 200.639 235.608 200.745 235.704L201.337 236.232L201.065 237.032C200.927 237.459 200.889 237.832 200.953 238.152C201.017 238.472 201.119 238.749 201.257 238.984C201.311 239.101 201.38 239.192 201.465 239.256C201.561 239.32 201.684 239.363 201.833 239.384C201.919 239.395 201.999 239.427 202.073 239.48C202.148 239.533 202.185 239.619 202.185 239.736C202.185 239.992 202.137 240.179 202.041 240.296C201.956 240.403 201.86 240.467 201.753 240.488C201.401 240.531 201.065 240.493 200.745 240.376C200.425 240.259 200.137 239.955 199.881 239.464C199.839 239.4 199.78 239.283 199.705 239.112C199.641 238.931 199.604 238.813 199.593 238.76C199.551 238.813 199.492 238.883 199.417 238.968C199.353 239.043 199.289 239.112 199.225 239.176C198.468 239.944 197.817 240.445 197.273 240.68C196.74 240.904 196.292 240.899 195.929 240.664ZM196.809 239.336C196.884 239.347 197.001 239.309 197.161 239.224C197.321 239.128 197.535 238.973 197.801 238.76C198.164 238.451 198.463 238.195 198.697 237.992C198.932 237.789 199.161 237.528 199.385 237.208L199.737 236.632C199.631 236.387 199.535 236.227 199.449 236.152C199.364 236.077 199.279 236.04 199.193 236.04C199.055 236.04 198.889 236.131 198.697 236.312C198.505 236.483 198.287 236.739 198.041 237.08C197.807 237.411 197.54 237.827 197.241 238.328C197.103 238.563 196.996 238.792 196.921 239.016C196.847 239.229 196.809 239.336 196.809 239.336ZM204.622 241.064C204.313 241.117 204.003 241.091 203.694 240.984C203.395 240.877 203.145 240.707 202.942 240.472C202.75 240.227 202.654 239.933 202.654 239.592C202.622 239.272 202.649 238.893 202.734 238.456C202.83 238.008 202.969 237.56 203.15 237.112C203.342 236.664 203.561 236.285 203.806 235.976C204.019 235.72 204.233 235.485 204.446 235.272C204.659 235.059 204.851 234.915 205.022 234.84C205.171 234.776 205.353 234.728 205.566 234.696C205.79 234.653 205.945 234.632 206.03 234.632C206.201 234.675 206.387 234.744 206.59 234.84C206.803 234.936 206.91 235.048 206.91 235.176C206.91 235.176 206.921 235.192 206.942 235.224C206.974 235.245 206.99 235.256 206.99 235.256C207.054 235.256 207.123 235.336 207.198 235.496C207.283 235.645 207.347 235.821 207.39 236.024C207.433 236.216 207.422 236.371 207.358 236.488C207.315 236.691 207.23 236.888 207.102 237.08C206.985 237.272 206.883 237.368 206.798 237.368C206.755 237.368 206.713 237.373 206.67 237.384C206.638 237.384 206.622 237.411 206.622 237.464C206.622 237.517 206.563 237.544 206.446 237.544C206.339 237.544 206.227 237.523 206.11 237.48C205.993 237.437 205.918 237.373 205.886 237.288C205.886 237.235 205.897 237.165 205.918 237.08C205.939 236.995 205.971 236.851 206.014 236.648C206.078 236.349 206.083 236.152 206.03 236.056C205.977 235.949 205.843 235.939 205.63 236.024C205.363 236.173 205.145 236.349 204.974 236.552C204.803 236.744 204.649 237.016 204.51 237.368C204.382 237.72 204.238 238.211 204.078 238.84C204.014 239.085 204.003 239.277 204.046 239.416C204.099 239.555 204.142 239.629 204.174 239.64C204.323 239.683 204.483 239.693 204.654 239.672C204.835 239.64 204.985 239.592 205.102 239.528C205.273 239.443 205.422 239.352 205.55 239.256C205.689 239.149 205.838 239.037 205.998 238.92C206.051 238.888 206.099 238.829 206.142 238.744C206.185 238.648 206.249 238.579 206.334 238.536C206.419 238.408 206.526 238.344 206.654 238.344C206.793 238.333 206.926 238.328 207.054 238.328C207.139 238.349 207.23 238.403 207.326 238.488C207.422 238.573 207.491 238.659 207.534 238.744C207.587 238.819 207.582 238.872 207.518 238.904C207.475 238.904 207.449 238.92 207.438 238.952C207.438 238.973 207.438 238.984 207.438 238.984C207.459 239.027 207.411 239.123 207.294 239.272C207.187 239.421 207.049 239.581 206.878 239.752C206.707 239.923 206.542 240.072 206.382 240.2C206.233 240.317 206.126 240.376 206.062 240.376C205.955 240.376 205.902 240.408 205.902 240.472C205.902 240.493 205.822 240.547 205.662 240.632C205.513 240.717 205.337 240.803 205.134 240.888C204.942 240.973 204.771 241.032 204.622 241.064ZM208.93 240.888C208.77 240.835 208.626 240.749 208.498 240.632C208.381 240.515 208.333 240.323 208.354 240.056C208.375 239.896 208.407 239.731 208.45 239.56C208.493 239.379 208.546 239.203 208.61 239.032C208.642 238.84 208.679 238.648 208.722 238.456C208.775 238.264 208.861 238.083 208.978 237.912C209.117 237.517 209.261 237.128 209.41 236.744C209.559 236.349 209.687 235.939 209.794 235.512C209.954 235.128 210.098 234.787 210.226 234.488C210.365 234.189 210.53 233.805 210.722 233.336C210.829 233.091 210.919 232.819 210.994 232.52C211.079 232.211 211.207 231.939 211.378 231.704C211.453 231.608 211.506 231.528 211.538 231.464C211.581 231.389 211.623 231.32 211.666 231.256C211.741 231.107 211.853 231.037 212.002 231.048C212.162 231.059 212.269 231.101 212.322 231.176C212.503 231.251 212.631 231.389 212.706 231.592C212.781 231.784 212.807 231.923 212.786 232.008C212.786 232.125 212.743 232.253 212.658 232.392C212.615 232.435 212.573 232.499 212.53 232.584C212.487 232.659 212.423 232.76 212.338 232.888C212.295 233.027 212.253 233.165 212.21 233.304C212.167 233.443 212.119 233.571 212.066 233.688C212.013 233.805 211.954 233.923 211.89 234.04C211.826 234.157 211.783 234.253 211.762 234.328C211.666 234.573 211.559 234.813 211.442 235.048C211.335 235.272 211.239 235.512 211.154 235.768C211.069 236.013 210.978 236.275 210.882 236.552C210.786 236.819 210.663 237.075 210.514 237.32C210.439 237.427 210.397 237.571 210.386 237.752C210.386 237.933 210.365 238.109 210.322 238.28C210.247 238.472 210.178 238.648 210.114 238.808C210.061 238.968 210.034 239.069 210.034 239.112C209.959 239.347 209.901 239.581 209.858 239.816C209.815 240.04 209.746 240.237 209.65 240.408C209.65 240.664 209.581 240.824 209.442 240.888C209.303 240.952 209.133 240.952 208.93 240.888ZM212.594 241.032C212.423 241.011 212.295 240.979 212.21 240.936C212.125 240.883 211.965 240.771 211.73 240.6C211.602 240.472 211.442 240.275 211.25 240.008C211.058 239.741 210.866 239.459 210.674 239.16C210.482 238.851 210.311 238.589 210.162 238.376L211.042 237.256C211.181 237.427 211.293 237.587 211.378 237.736C211.463 237.885 211.533 237.987 211.586 238.04C211.831 238.403 212.023 238.696 212.162 238.92C212.301 239.144 212.418 239.325 212.514 239.464C212.621 239.603 212.738 239.725 212.866 239.832C212.994 239.939 213.159 240.051 213.362 240.168C213.394 240.189 213.426 240.243 213.458 240.328C213.49 240.403 213.501 240.483 213.49 240.568C213.469 240.696 213.389 240.803 213.25 240.888C213.122 240.963 212.903 241.011 212.594 241.032ZM210.274 238.072C210.21 237.891 210.194 237.688 210.226 237.464C210.269 237.229 210.359 237.075 210.498 237C210.541 236.925 210.583 236.877 210.626 236.856C210.679 236.824 210.711 236.787 210.722 236.744C210.743 236.701 210.797 236.643 210.882 236.568C210.978 236.493 211.063 236.424 211.138 236.36C211.309 236.253 211.495 236.125 211.698 235.976C211.911 235.816 212.114 235.656 212.306 235.496C212.509 235.336 212.674 235.197 212.802 235.08C212.941 234.963 213.026 234.888 213.058 234.856C213.09 234.813 213.154 234.787 213.25 234.776C213.357 234.765 213.458 234.781 213.554 234.824C213.682 234.856 213.783 234.925 213.858 235.032C213.943 235.139 213.986 235.288 213.986 235.48C214.018 235.597 213.949 235.741 213.778 235.912C213.618 236.072 213.415 236.232 213.17 236.392C213.085 236.445 212.967 236.525 212.818 236.632C212.679 236.739 212.557 236.84 212.45 236.936C212.375 236.979 212.263 237.053 212.114 237.16C211.975 237.256 211.853 237.352 211.746 237.448C211.661 237.501 211.559 237.597 211.442 237.736C211.335 237.864 211.213 237.96 211.074 238.024C210.733 238.344 210.525 238.499 210.45 238.488C210.386 238.477 210.327 238.339 210.274 238.072Z" fill="#714B67"/>
</svg>

```

## File: views\stock_move_line_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_move_line_tree_detailed_wave" model="ir.ui.view">
        <field name="name">stock_picking_wave.move.line.list.wave</field>
        <field name="model">stock.move.line</field>
        <field name="inherit_id" ref="stock.view_move_line_tree_detailed"/>
        <field name="arch" type="xml">
            <xpath expr="//list" position="inside">
                <header>
                    <button name="action_open_add_to_wave" type="object" string="Add to Wave"/>
                </header>
            </xpath>
            <xpath expr="//field[@name='picking_id']" position="after">
                <field name="batch_id" optional="show"/>
            </xpath>
        </field>
    </record>

    <record id="view_picking_internal_search_inherit" model="ir.ui.view">
        <field name="name">stock.picking.internal.search.inherit</field>
        <field name="model">stock.picking</field>
        <field name="inherit_id" ref="stock.view_picking_internal_search"/>
        <field name="arch" type="xml">
            <filter name="picking_type" position="after">
                <filter string="Batch Transfer" name="batch" domain="[]" context="{'group_by': 'batch_id'}"/>
            </filter>
        </field>
    </record>
</odoo>

```

## File: views\stock_picking_batch_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_picking_form_inherited" model="ir.ui.view">
        <field name="name">stock_picking_batch.picking.form</field>
        <field name="model">stock.picking</field>
        <field name="inherit_id" ref="stock.view_picking_form"/>
        <field name="mode">primary</field>
        <field name="priority" eval="1000"/>
        <field name="arch" type="xml">
            <xpath expr="//header" position="replace">
                <header>
                    <field name="state" widget="statusbar" statusbar_visible="draft,confirmed,assigned,done"/>
                </header>
            </xpath>
        </field>
    </record>

    <record id="view_picking_move_tree_inherited" model="ir.ui.view">
        <field name="name">stock_picking_batch.picking.move.list</field>
        <field name="model">stock.move</field>
        <field name="inherit_id" ref="stock.view_picking_move_tree"/>
        <field name="mode">primary</field>
        <field name="priority" eval="1000"/>
        <field name="arch" type="xml">
            <xpath expr="//list" position="attributes">
                <attribute name="delete">0</attribute>
            </xpath>
            <xpath expr="//field[@name='product_id']" position="after">
                <field name="picking_id"
                    required="1"
                    readonly="id"
                    domain="[('id', 'in', parent.picking_ids)]"
                    options="{'no_create_edit': True}"/>
            </xpath>
            <xpath expr="//field[@name='quantity']" position="attributes">
                <attribute name="readonly">1</attribute>
            </xpath>
        </field>
    </record>

    <record id="view_move_line_tree" model="ir.ui.view">
        <field name="name">stock_picking_batch.move.line.list</field>
        <field name="model">stock.move.line</field>
        <field name="arch" type="xml">
            <list editable="top" decoration-muted="state == 'cancel'" string="Move Lines" default_order="location_id">
                <field name="tracking" column_invisible="True"/>
                <field name="state" column_invisible="True"/>
                <field name="company_id" column_invisible="True"/>
                <field name="product_id" context="{'default_is_storable': True}" required="1" readonly="id"/>
                <field name="picking_id" required="1" readonly="id"
                    options="{'no_create_edit': True}" domain="[('id', 'in', parent.picking_ids)]"/>
                <field name="lot_id"   groups="stock.group_production_lot" readonly="tracking not in ['lot', 'serial']" column_invisible="parent.show_lots_text" />
                <field name="lot_name" groups="stock.group_production_lot" readonly="tracking not in ['lot', 'serial']" column_invisible="not parent.show_lots_text"/>
                <field name="location_id"/>
                <field name="location_dest_id"/>
                <field name="package_id" groups="stock.group_tracking_lot"/>
                <field name="result_package_id" groups="stock.group_tracking_lot"/>
                <field name="product_uom_id" options="{'no_create': True}"
                    groups="uom.group_uom" readonly="1" force_save="1"/>
                <field name="quantity"/>
                <field name="company_id" groups="base.group_multi_company" force_save="1"/>
            </list>
        </field>
    </record>

    <record id="stock_picking_batch_form" model="ir.ui.view">
        <field name="name">stock.picking.batch.form</field>
        <field name="model">stock.picking.batch</field>
        <field name="arch" type="xml">
            <form string="Stock Batch Transfer">
                <field name="company_id" invisible="1"/>
                <field name="show_check_availability" invisible="1"/>
                <field name="show_allocation" invisible="1"/>
                <field name="picking_type_code" invisible="1"/>
                <field name="is_wave" invisible="1"/>
                <field name="show_lots_text" invisible="1"/>
                <header>
                    <button name="action_confirm" invisible="state != 'draft'" string="Confirm" type="object" class="oe_highlight"/>
                    <button name="action_done" string="Validate" type="object" class="oe_highlight"
                        invisible="state != 'in_progress' or show_check_availability"/>
                    <button name="action_assign" string="Check Availability" type="object" class="oe_highlight"
                        invisible="state != 'in_progress' or not show_check_availability"/>
                    <button name="action_done" string="Validate" type="object"
                        invisible="state != 'in_progress' or not show_check_availability"/>
                    <button name="action_assign" string="Check Availability" type="object"
                        invisible="state != 'draft' or not show_check_availability"/>
                    <button name="action_print" invisible="state not in ('in_progress', 'done')" string="Print" type="object"/>
                    <button string="Print Labels" type="object" name="action_open_label_layout"/>
                    <button name="action_cancel" string="Cancel" type="object" invisible="state != 'in_progress'"/>
                    <field name="state" widget="statusbar" statusbar_visible="draft,in_progress,done"/>
                </header>
                <sheet>
                    <div class="oe_button_box" name="button_box">
                        <button name="action_view_reception_report" string="Allocation" type="object"
                            class="oe_stat_button" icon="fa-list"
                            invisible="not show_allocation"
                            groups="stock.group_reception_report"/>
                    </div>
                    <div class="oe_title">
                        <h1><field name="name" class="oe_inline"/></h1>
                    </div>
                    <group>
                        <group id="batch_delivery_data">
                            <field name="user_id" readonly="state not in ['draft', 'in_progress']"/>
                            <field name="picking_type_id" readonly="state != 'draft'"/>
                            <field name="company_id" groups="base.group_multi_company" readonly="picking_ids" force_save="1"/>
                            <field name="scheduled_date" readonly="state in ['cancel', 'done']"/>
                            <field name="description"/>
                        </group>
                        <field name="properties" nolabel="1" columns="2" hideAddButton="1"/>
                    </group>
                    <notebook>
                        <page string="Detailed Operations" name="page_detailed_operations" invisible="state == 'draft'">
                            <field name="move_line_ids" context="{'list_view_ref': 'stock_picking_batch.view_move_line_tree'}" readonly="state not in ['draft', 'in_progress']"/>
                            <button class="oe_highlight" name="action_put_in_pack" type="object" string="Put in Pack" invisible="state in ('draft', 'done', 'cancel')" groups="stock.group_tracking_lot"/>
                        </page>
                        <page string="Operations" name="page_operations" invisible="state == 'draft'">
                            <field name="move_ids" context="{'list_view_ref': 'stock_picking_batch.view_picking_move_tree_inherited'}"/>
                        </page>
                        <page string="Transfers" name="page_transfers">
                            <field name="allowed_picking_ids" invisible="1"/>
                            <field name="picking_ids" widget="many2many" mode="list,kanban"
                                context="{'form_view_ref': 'stock_picking_batch.view_picking_form_inherited', 'list_view_ref': 'stock_picking_batch.stock_picking_view_batch_tree_ref'}" readonly="state not in ['draft', 'in_progress']"/>
                        </page>
                    </notebook>
                </sheet>
                <chatter/>
            </form>
        </field>
    </record>

    <record id="stock_picking_batch_tree" model="ir.ui.view">
        <field name="name">stock.picking.batch.list</field>
        <field name="model">stock.picking.batch</field>
        <field name="arch" type="xml">
            <list string="Stock Batch Transfer" multi_edit="1" sample="1" class="oe_stock_picking_batch">
                <field name="company_id" column_invisible="True"/>
                <field name="name" decoration-bf="1"/>
                <field name="description"/>
                <field name="scheduled_date" readonly="state in ['cancel', 'done']"/>
                <field name="user_id" widget="many2one_avatar_user" readonly="state not in ['draft', 'in_progress']"/>
                <field name="picking_type_id" readonly="state != 'draft'"/>
                <field name="company_id" optional="hide" groups="base.group_multi_company"/>
                <field name="state" widget="badge" decoration-success="state == 'done'" decoration-info="state in ('draft', 'in_progress')" decoration-danger="state == 'cancel'"/>
                <field name="activity_exception_decoration" widget="activity_exception"/>
            </list>
        </field>
    </record>

    <record id="stock_picking_batch_kanban" model="ir.ui.view">
        <field name="name">stock.picking.batch.kanban</field>
        <field name="model">stock.picking.batch</field>
        <field name="arch" type="xml">
            <kanban class="o_kanban_mobile oe_stock_picking_batch" sample="1">
                <field name="company_id"/>
                <templates>
                    <t t-name="card">
                        <div class="d-flex">
                            <field name="name" class="fw-bolder fs-5"/>
                            <field name="state" widget="label_selection" class="ms-auto"/>
                        </div>
                        <field name="description" class="fw-bold mb-2"/>
                        <footer class="pt-0">
                            <field name="picking_type_id" readonly="state != 'draft'"/>
                            <div class="d-flex">
                                <field name="scheduled_date" readonly="state in ['cancel', 'done']"/>
                                <field name="user_id" widget="many2one_avatar_user" readonly="state not in ['draft', 'in_progress']"/>
                            </div>
                        </footer>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record model="ir.ui.view" id="stock_picking_batch_calendar">
        <field name="name">stock.picking.batch.calendar</field>
        <field name="model">stock.picking.batch</field>
        <field name="priority" eval="2"/>
        <field name="arch" type="xml">
            <calendar string="Calendar View" date_start="scheduled_date" event_limit="5" quick_create="0">
                <field name="scheduled_date"/>
            </calendar>
        </field>
    </record>

    <record id="stock_picking_batch_filter" model="ir.ui.view">
        <field name="name">stock.picking.batch.filter</field>
        <field name="model">stock.picking.batch</field>
        <field name="arch" type="xml">
            <search string="Search Batch Transfer">
                <field name="name" string="Batch Transfer"/>
                <field name="picking_type_id" invisible="1"/>
                <field name="user_id"/>
                <filter name="to_do_transfers" string="To Do" domain="['&amp;',('user_id', 'in', [uid, False]),('state','not in',['done','cancel'])]"/>
                <filter name="my_transfers" string="My Transfers" domain="[('user_id', '=', uid)]"/>
                <separator/>
                <filter name="draft" string="Draft" domain="[('state', '=', 'draft')]"/>
                <filter name="in_progress" string="In Progress" domain="[('state', '=', 'in_progress')]" help="Batch Transfers not finished"/>
                <filter name="done" string="Done" domain="[('state', '=', 'done')]"/>
                <separator/>
                <filter invisible="1" string="Late Activities" name="activities_overdue"
                    domain="[('my_activity_date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                    help="Show all records which has next action date is before today"/>
                <filter invisible="1" string="Today Activities" name="activities_today"
                    domain="[('my_activity_date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                    domain="[('my_activity_date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
                <group expand="0" string="Group By">
                    <filter string="Responsible" name="user" domain="[]" context="{'group_by': 'user_id'}"/>
                    <filter string="State" name="state" domain="[]" context="{'group_by': 'state'}"/>
                </group>
           </search>
        </field>
    </record>

    <record id="stock_picking_batch_action" model="ir.actions.act_window">
        <field name="name">Batch Transfers</field>
        <field name="res_model">stock.picking.batch</field>
        <field name="view_mode">list,kanban,form</field>
        <field name="domain">[('is_wave', '=', False)]</field>
        <field name="context">{'search_default_draft': True, 'search_default_in_progress': True}</field>
        <field name="search_view_id" ref="stock_picking_batch_filter"/>
        <field name="help" type="html">
            <div class="container mt-5">
                <div class="row g-5">
                    <div class="col-lg-4" style="opacity: 0.5;">
                        <img src="/stock_picking_batch/static/shapes/wave-picking.svg" class="shadow rounded-3 w-100 mb-4"/>
                        <h5>Wave transfers</h5>
                        <p class="small">Launch picking orders by aisle or area and regroup at packing zone. Ideal for large warehouses.</p>
                    </div>
                    <div class="col-lg-4">
                        <img src="/stock_picking_batch/static/shapes/batch-picking.svg" class="shadow rounded-3 w-100 mb-4"/>
                        <h5>Batch transfers</h5>
                        <p class="small">Regroup multiple orders into one picking and consolidate at the packing zone.</p>
                    </div>
                    <div class="col-lg-4">
                        <img src="/stock_picking_batch/static/shapes/cluster-picking.svg" class="shadow rounded-3 w-100 mb-4"/>
                        <h5>Cluster transfers</h5>
                        <p class="small">Pick multiple orders in one trip and prepare orders as you pick. This reduces packing time and is ideal for small products.</p>
                    </div>
                </div>
            </div>
        </field>
    </record>

    <menuitem id="menu_stock_jobs" name="Jobs" parent="stock.menu_stock_warehouse_mgmt" sequence="2"/>
    <menuitem action="stock_picking_batch_action" id="stock_picking_batch_menu" parent="menu_stock_jobs" sequence="30"/>

    <record id="view_picking_internal_search_inherit_stock_picking_batch" model="ir.ui.view">
        <field name="name">stock.picking.search</field>
        <field name="model">stock.picking</field>
        <field name="inherit_id" ref="stock.view_picking_internal_search"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='product_id']" position="after">
                <field name="batch_id"/>
            </xpath>
        </field>
    </record>
    <record id="view_move_line_tree_inherit_stock_picking_batch" model="ir.ui.view">
        <field name="name">stock.move.line.list.stock_picking_batch</field>
        <field name="model">stock.move.line</field>
        <field name="inherit_id" ref="stock.view_move_line_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='state']" position="after">
                <field name="batch_id" optional="hide"/>
            </xpath>
        </field>
    </record>
    <record id="stock_move_line_view_search_inherit_stock_picking_batch" model="ir.ui.view">
        <field name="name">stock.move.line.search.stock_picking_batch</field>
        <field name="model">stock.move.line</field>
        <field name="inherit_id" ref="stock.stock_move_line_view_search"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='groupby']" position="inside">
                <filter string="Batch Transfer" name="by_batch_id" context="{'group_by': 'batch_id'}"/>
            </xpath>
        </field>
    </record>

    <record id="action_unreserve_batch_picking" model="ir.actions.server">
        <field name="name">Unreserve</field>
        <field name="model_id" ref="stock_picking_batch.model_stock_picking_batch"/>
        <field name="binding_model_id" ref="stock_picking_batch.model_stock_picking_batch"/>
        <field name="binding_view_types">list,form</field>
        <field name="state">code</field>
        <field name="code">
        if records:
            records.picking_ids.do_unreserve()
        </field>
    </record>
</odoo>

```

## File: views\stock_picking_type_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_picking_type_form_inherit" model="ir.ui.view">
        <field name="name">stock.picking.type.form.inherit</field>
        <field name="model">stock.picking.type</field>
        <field name="inherit_id" ref="stock.view_picking_type_form"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='second']" position="after">
                <group name="batch" invisible="code not in ('incoming', 'outgoing', 'internal')">
                    <group string="Batch &amp; Wave Transfers" colspan="12">
                        <group>
                            <field name="auto_batch"/>
                            <field name="batch_max_lines" invisible="not auto_batch"/>
                            <field name="batch_max_pickings" invisible="not auto_batch"/>
                            <field name="batch_auto_confirm" invisible="not auto_batch"/>
                        </group>
                        <group>
                            <span class="o_form_label fw-bold" invisible="not auto_batch">Batch Grouping</span>
                            <div name="batch_contact" class="o_row" invisible="not auto_batch">
                                <field name="batch_group_by_partner"/>
                                <label for="batch_group_by_partner" string="Contact"/>
                            </div>
                            <span invisible="not auto_batch"/>
                            <div name="batch_destination" class="o_row" invisible="not auto_batch">
                                <field name="batch_group_by_destination"/>
                                <label for="batch_group_by_destination"/>
                            </div>
                            <span invisible="not auto_batch or not default_location_src_id" groups="stock.group_stock_multi_locations"/>
                            <div name="batch_source_location" class="o_row" invisible="not auto_batch or not default_location_src_id" groups="stock.group_stock_multi_locations">
                                <field name="batch_group_by_src_loc"/>
                                <label for="batch_group_by_src_loc" string="Source Location"/>
                            </div>
                            <span invisible="not auto_batch or not default_location_dest_id" groups="stock.group_stock_multi_locations"/>
                            <div name="batch_dest_subloc" class="o_row" invisible="not auto_batch or not default_location_dest_id" groups="stock.group_stock_multi_locations">
                                <field name="batch_group_by_dest_loc"/>
                                <label for="batch_group_by_dest_loc" string="Destination Location"/>
                            </div>
                            <span class="o_form_label fw-bold" invisible="not auto_batch" style="white-space: nowrap;">Wave Grouping</span>
                            <div name="wave_product" class="o_row" invisible="not auto_batch">
                                <field name="wave_group_by_product"/>
                                <label for="wave_group_by_product"/>
                            </div>
                            <span invisible="not auto_batch"/>
                            <div name="wave_category" class="o_row" invisible="not auto_batch">
                                <field name="wave_group_by_category"/>
                                <label for="wave_group_by_category"/>
                                <field name="wave_category_ids" widget="many2many_tags" placeholder="Select product categories you want to group"
                                    invisible="not wave_group_by_category" required="wave_group_by_category"/>
                            </div>
                            <span invisible="not auto_batch" groups="stock.group_stock_multi_locations"/>
                            <div name="wave_location" class="o_row" invisible="not auto_batch" groups="stock.group_stock_multi_locations">
                                <field name="wave_group_by_location"/>
                                <label for="wave_group_by_location"/>
                                <field name="wave_location_ids" widget="many2many_tags" placeholder="Select locations you want to group"
                                    invisible="not wave_group_by_location" required="wave_group_by_location"/>
                            </div>
                        </group>
                    </group>
                </group>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\stock_picking_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="stock_picking_form_inherit" model="ir.ui.view">
        <field name="name">stock.picking.form.inherit</field>
        <field name="model">stock.picking</field>
        <field name="inherit_id" ref="stock.view_picking_form"/>
        <field name="arch" type="xml">
            <div name="button_box" position="inside">
                <button type="object"
                        name="action_view_batch"
                        class="oe_stat_button"
                        icon="fa-truck"
                        string="Batch"
                        invisible="not batch_id"/>
            </div>
            <xpath expr="//page[@name='extra']//field[@name='company_id']" position="after">
                <field name="batch_id"
                    readonly="1"
                    options="{'no_create': True}"/>
            </xpath>
        </field>
    </record>

    <record id="vpicktree" model="ir.ui.view">
        <field name="name">stock.picking.list.inherit.stock.picking.batch</field>
        <field name="model">stock.picking</field>
        <field name="inherit_id" ref="stock.vpicktree"/>
        <field name="arch" type="xml">
            <field name="picking_type_id" position="after">
                <field name="batch_id" optional="show"
                    domain="[
                        ('state', 'in', ['draft', 'in_progress']),
                        '|',
                            ('picking_type_id', '=', picking_type_id),
                            ('picking_type_id', '=', False),
                    ]"
                    context="{'default_picking_type_id': picking_type_id}" readonly="state in ['cancel', 'done']"/>
            </field>
        </field>
    </record>

    <record id="stock_picking_view_batch_tree_ref" model="ir.ui.view">
        <field name="name">stock.picking.view.list.inherit.stock.picking.batch</field>
        <field name="model">stock.picking</field>
        <field name="inherit_id" ref="stock.vpicktree"/>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <xpath expr="//list" position="attributes">
                <attribute name="default_order">batch_sequence</attribute>
            </xpath>
            <field name="company_id" position="before">
                <field name="batch_sequence" widget="handle"/>
            </field>
            <field name="company_id" position="replace"/>
            <field name="batch_id" position="replace"/>
            <field name="scheduled_date" position="attributes">
                <attribute name="optional">hide</attribute>
            </field>
        </field>
    </record>
</odoo>

```

## File: views\stock_picking_wave_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="action_prepare_wave_for_picking_type" model="ir.actions.act_window">
        <field name="name">Prepare Wave</field>
        <field name="res_model">stock.move.line</field>
        <field name="view_mode">list</field>
        <field name="view_id" ref="view_move_line_tree_detailed_wave"/>
        <field name="target">new</field>
        <field name="domain">[('state', '!=', 'done'), ('picking_type_id', '=', active_id), ('picking_id', '!=', False)]</field>
        <field name="context">{'active_wave_id': False, 'search_default_by_location': True}</field>
    </record>
    <record id="action_prepare_wave" model="ir.actions.act_window">
        <field name="name">Prepare Wave</field>
        <field name="res_model">stock.move.line</field>
        <field name="view_mode">list</field>
        <field name="view_id" ref="view_move_line_tree_detailed_wave"/>
        <field name="target">new</field>
        <field name="domain">[('state', '!=', 'done'), ('picking_id', '!=', False)]</field>
        <field name="context">{'active_wave_id': False, 'search_default_by_location': True}</field>
    </record>
    <record id="stock_picking_wave_tree" model="ir.ui.view">
        <field name="name">stock.picking.wave.list</field>
        <field name="model">stock.picking.batch</field>
        <field name="priority">25</field>
        <field name="mode">primary</field>
        <field name="inherit_id" ref="stock_picking_batch.stock_picking_batch_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//list" position="attributes">
                <attribute name="create">0</attribute>
            </xpath>
            <xpath expr="//list" position="inside">
                <header>
                    <button string="Prepare Wave" class="btn btn-primary" type="action" display="always"
                     name="stock_picking_batch.action_prepare_wave"/>
                </header>
            </xpath>
        </field>
    </record>
    <record id="stock_picking_wave_kanban" model="ir.ui.view">
        <field name="name">stock.picking.wave.kanban</field>
        <field name="model">stock.picking.batch</field>
        <field name="priority">25</field>
        <field name="mode">primary</field>
        <field name="inherit_id" ref="stock_picking_batch.stock_picking_batch_kanban"/>
        <field name="arch" type="xml">
            <xpath expr="//kanban" position="attributes">
                <attribute name="create">0</attribute>
            </xpath>
            <xpath expr="//kanban" position="inside">
                <header>
                    <button string="Prepare Wave" class="btn btn-primary" type="action" display="always"
                     name="stock_picking_batch.action_prepare_wave"/>
                </header>
            </xpath>
        </field>
    </record>
    <record id="action_picking_tree_wave" model="ir.actions.act_window">
        <field name="name">Wave Transfers</field>
        <field name="res_model">stock.picking.batch</field>
        <field name="view_mode">list,kanban,form</field>
        <field name="context">{'search_default_draft': True, 'search_default_in_progress': True}</field>
        <field name="view_ids" eval="[(5, 0, 0),
            (0, 0, {'view_mode': 'list', 'view_id': ref('stock_picking_batch.stock_picking_wave_tree')}),
            (0, 0, {'view_mode': 'kanban', 'view_id': ref('stock_picking_batch.stock_picking_wave_kanban')})]"/>
        <field name="domain">[('is_wave', '=', True)]</field>
        <field name="search_view_id" ref="stock_picking_batch_filter"/>
        <field name="help" type="html">
            <div class="container mt-5">
                <div class="row g-5">
                    <div class="col-lg-4">
                        <img src="/stock_picking_batch/static/shapes/wave-picking.svg" class="shadow rounded-3 w-100 mb-4"/>
                        <h5>Wave transfers</h5>
                        <p class="small">Launch picking orders by aisle or area and regroup at packing zone. Ideal for large warehouses.</p>
                    </div>
                    <div class="col-lg-4" style="opacity: 0.5;">
                        <img src="/stock_picking_batch/static/shapes/batch-picking.svg" class="shadow rounded-3 w-100 mb-4"/>
                        <h5>Batch transfers</h5>
                        <p class="small">Regroup multiple orders into one picking and consolidate at the packing zone.</p>
                    </div>
                    <div class="col-lg-4">
                        <img src="/stock_picking_batch/static/shapes/cluster-picking.svg" class="shadow rounded-3 w-100 mb-4"/>
                        <h5>Cluster transfers</h5>
                        <p class="small">Pick multiple orders in one trip and prepare orders as you pick. This reduces packing time and is ideal for small products.</p>
                    </div>
                </div>
            </div>
        </field>
    </record>

    <record id="stock_picking_type_kanban_batch" model="ir.ui.view">
        <field name="name">picking.type.kanban.batch</field>
        <field name="model">stock.picking.type</field>
        <field name="inherit_id" ref="stock.stock_picking_type_kanban"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='kanban_menu_section']" position="inside">
                <div role="menuitem">
                    <a name="action_batch" type="object" context="{'view_mode':'form'}">
                        Prepare Batch
                    </a>
                </div>
                <div role="menuitem">
                    <a name="stock_picking_batch.action_prepare_wave_for_picking_type" type="action">
                        Prepare Wave
                    </a>
                </div>
            </xpath>
            <xpath expr="//button[@name='get_action_picking_tree_ready']" position="before">
                <button t-if="record.count_picking_batch.raw_value > 0" class="btn btn-primary me-2" name="action_batch" type="object">
                    <field name="count_picking_batch"/> Batches
                </button>
            </xpath>
            <xpath expr="//button[@name='get_action_picking_tree_ready']" position="attributes">
                <attribute
                    name="t-attf-class"
                    remove="btn-primary"
                    add="{{ record.count_picking_batch.raw_value > 0 ? 'btn-secondary' : 'btn-primary'}}"
                    separator=" "
                />
            </xpath>
            <xpath expr="//div[@name='picking_type_backorder_count']" class="row" position="after">
                <div class="row">
                    <a class="col-8 offset-4 text-truncate" name="%(stock_picking_batch.action_picking_tree_wave)d" type="action">
                        <div t-if="record.count_picking_wave.raw_value > 0"  class="row">
                            <span class="col-6">Waves</span>
                            <field name="count_picking_wave" class="col-2 text-end"/>
                        </div>
                    </a>
                </div>
            </xpath>
        </field>
    </record>

    <menuitem
        action="action_picking_tree_wave"
        id="stock_picking_wave_menu"
        parent="menu_stock_jobs"
        sequence="31"/>

</odoo>

```

## File: wizard\stock_add_to_wave.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api, _
from odoo.exceptions import UserError


class StockPickingToWave(models.TransientModel):
    _name = 'stock.add.to.wave'
    _description = 'Wave Transfer Lines'

    @api.model
    def default_get(self, fields_list):
        res = super().default_get(fields_list)
        if self.env.context.get('active_model') == 'stock.move.line':
            lines = self.env['stock.move.line'].browse(self.env.context.get('active_ids'))
            res['line_ids'] = self.env.context.get('active_ids')
            picking_types = lines.picking_type_id
        elif self.env.context.get('active_model') == 'stock.picking':
            pickings = self.env['stock.picking'].browse(self.env.context.get('active_ids'))
            res['picking_ids'] = self.env.context.get('active_ids')
            picking_types = pickings.picking_type_id
        else:
            return res

        if len(picking_types) > 1:
            raise UserError(_("The selected transfers should belong to the same operation type"))
        return res

    wave_id = fields.Many2one('stock.picking.batch', string='Wave Transfer', domain="[('is_wave', '=', True), ('state', 'in', ('draft', 'in_progress'))]")
    picking_ids = fields.Many2many('stock.picking')
    line_ids = fields.Many2many('stock.move.line')
    mode = fields.Selection([('existing', 'an existing wave transfer'), ('new', 'a new wave transfer')], default='existing')
    user_id = fields.Many2one('res.users', string='Responsible')


    def attach_pickings(self):
        self.ensure_one()

        self = self.with_context(active_owner_id=self.user_id.id)
        if self.line_ids:
            company = self.line_ids.company_id
            if len(company) > 1:
                raise UserError(_("The selected operations should belong to a unique company."))
            return self.line_ids._add_to_wave(self.wave_id)
        if self.picking_ids:
            company = self.picking_ids.company_id
            if len(company) > 1:
                raise UserError(_("The selected transfers should belong to a unique company."))
        else:
            raise UserError(_('Cannot create wave transfers'))

        view = self.env.ref('stock_picking_batch.view_move_line_tree_detailed_wave')
        return {
            'name': _('Add Operations'),
            'type': 'ir.actions.act_window',
            'view_mode': 'list',
            'views': [(view.id, 'list')],
            'res_model': 'stock.move.line',
            'target': 'new',
            'domain': [
                ('picking_id', 'in', self.picking_ids.ids),
                ('state', '!=', 'done')
            ],
            'context': dict(
                self.env.context,
                picking_to_wave=self.picking_ids.ids,
                active_wave_id=self.wave_id.id,
            )}

```

## File: wizard\stock_add_to_wave_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- add picking to wave  -->
    <record id="stock_add_to_wave_form" model="ir.ui.view">
        <field name="name">stock.add.to.wave.form</field>
        <field name="model">stock.add.to.wave</field>
        <field name="arch" type="xml">
            <form string="Add to Wave">
                <group>
                    <group>
                        <label for="mode" string="Add to"/>
                        <field name="mode" widget="radio" nolabel="1"/>
                        <field name="wave_id" options="{'no_create': True, 'no_open': True}" invisible="mode == 'new'" context="{'add_to_existing_batch': True}" required="mode == 'existing'"/>
                        <field name="user_id" options="{'no_create': True}" invisible="mode == 'existing'"/>
                    </group>
                </group>

                <footer>
                    <button name="attach_pickings" type="object" string="Confirm" class="btn-primary"/>
                    <button string="Cancel" class="btn-secondary" special="cancel"/>
                </footer>
            </form>
        </field>
    </record>
    <record id="stock_add_to_wave_action_stock_picking" model="ir.actions.act_window">
        <field name="name">Add to wave</field>
        <field name="res_model">stock.add.to.wave</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
        <field name="binding_model_id" ref="stock.model_stock_picking"/>
        <field name="binding_view_types">list</field>
    </record>
</odoo>

```

## File: wizard\stock_package_destination.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class ChooseDestinationLocation(models.TransientModel):
    _inherit = "stock.package.destination"

    def _compute_move_line_ids(self):
        destination_without_batch = self.env['stock.package.destination']
        for destination in self:
            if not destination.picking_id.batch_id:
                destination_without_batch |= destination
                continue
            destination.move_line_ids = destination.picking_id.batch_id.move_line_ids.filtered(lambda l: l.quantity > 0 and not l.result_package_id)
        super(ChooseDestinationLocation, destination_without_batch)._compute_move_line_ids()

    def action_done(self):
        if self.picking_id.batch_id:
            # set the same location on each move line and pass again in action_put_in_pack
            self.move_line_ids.location_dest_id = self.location_dest_id
            return self.picking_id.batch_id.action_put_in_pack()
        else:
            return super().action_done()

```

## File: wizard\stock_picking_to_batch.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _
from odoo.exceptions import UserError


class StockPickingToBatch(models.TransientModel):
    _name = 'stock.picking.to.batch'
    _description = 'Batch Transfer Lines'

    batch_id = fields.Many2one('stock.picking.batch', string='Batch Transfer', domain="[('is_wave', '=', False), ('state', 'in', ('draft', 'in_progress'))]")
    mode = fields.Selection([('existing', 'an existing batch transfer'), ('new', 'a new batch transfer')], default='new')
    user_id = fields.Many2one('res.users', string='Responsible')
    is_create_draft = fields.Boolean(string="Draft", help='When checked, create the batch in draft status')
    description = fields.Char('Description')

    def attach_pickings(self):
        self.ensure_one()
        pickings = self.env['stock.picking'].browse(self.env.context.get('active_ids'))
        if self.mode == 'new':
            company = pickings.company_id
            if len(company) > 1:
                raise UserError(_("The selected pickings should belong to an unique company."))
            batch = self.env['stock.picking.batch'].create({
                'user_id': self.user_id.id,
                'company_id': company.id,
                'picking_type_id': pickings[0].picking_type_id.id,
                'description': self.description,
            })
        else:
            batch = self.batch_id

        pickings.write({'batch_id': batch.id})
        # you have to set some pickings to batch before confirm it.
        if self.mode == 'new' and not self.is_create_draft:
            batch.action_confirm()
        return {
            'name': _('Batch Transfer'),
            'view_mode': 'form',
            'res_model': 'stock.picking.batch',
            'type': 'ir.actions.act_window',
            'res_id': batch.id,
        }

```

## File: wizard\stock_picking_to_batch_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- add picking to batch  -->
    <record id="stock_picking_to_batch_form" model="ir.ui.view">
        <field name="name">stock.picking.to.batch.form</field>
        <field name="model">stock.picking.to.batch</field>
        <field name="arch" type="xml">
            <form string="Add pickings to">
                <group>
                    <group>
                        <label for="mode" string="Add to"/>
                        <field name="mode" widget="radio" nolabel="1"/>
                        <field name="description" invisible="mode == 'existing'"/>
                        <field name="batch_id" context="{'add_to_existing_batch': True}" options="{'no_create': True, 'no_open': True}" invisible="mode == 'new'" required="mode == 'existing'"/>
                        <field name="user_id" options="{'no_create': True}" invisible="mode == 'existing'"/>
                        <field name="is_create_draft"  invisible="mode != 'new'"/>
                    </group>
                </group>

                <footer>
                    <button name="attach_pickings" type="object" string="Confirm" class="btn-primary" data-hotkey="q"/>
                    <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="x"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="stock_picking_to_batch_action_stock_picking" model="ir.actions.act_window">
        <field name="name">Add to batch</field>
        <field name="res_model">stock.picking.to.batch</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
        <field name="binding_model_id" ref="model_stock_picking"/>
        <field name="binding_view_types">list</field>
    </record>

</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import stock_picking_to_batch
from . import stock_add_to_wave
from . import stock_package_destination

```

