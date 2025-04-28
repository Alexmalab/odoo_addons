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
        'views/stock_picking_wave_views.xml',
        'views/stock_move_line_views.xml',
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
        <field name="picking_type_id" ref="stock.picking_type_out"/>
        <field name="location_id" ref="stock.stock_location_stock"/>
        <field name="location_dest_id" ref="stock.stock_location_customers"/>
        <field name="company_id" ref="base.main_company"/>
    </record>
    <record id="Picking_B" model="stock.picking">
        <field name="move_type">one</field>
        <field name="priority">0</field>
        <field name="user_id" eval="False"/>
        <field name="picking_type_id" ref="stock.picking_type_out"/>
        <field name="location_id" ref="stock.stock_location_stock"/>
        <field name="location_dest_id" ref="stock.stock_location_customers"/>
        <field name="company_id" ref="base.main_company"/>
    </record>
    <record id="Picking_C" model="stock.picking">
        <field name="move_type">one</field>
        <field name="priority">0</field>
        <field name="user_id" eval="False"/>
        <field name="picking_type_id" ref="stock.picking_type_out"/>
        <field name="location_id" ref="stock.stock_location_stock"/>
        <field name="location_dest_id" ref="stock.stock_location_customers"/>
        <field name="company_id" ref="base.main_company"/>
    </record>
    <record id="Picking_D" model="stock.picking">
        <field name="move_type">one</field>
        <field name="priority">1</field>
        <field name="user_id" eval="False"/>
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

```

## File: models\stock_move_line.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict

from odoo import _, fields, models
from odoo import Command
from odoo.tools.float_utils import float_compare


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

    def _add_to_wave(self, wave=False):
        """ Detach lines (and corresponding stock move from a picking to another). If wave is
        passed, attach new picking into it. If not attach line to their original picking.

        :param int wave: id of the wave picking on which to put the move lines. """

        if not wave:
            wave = self.env['stock.picking.batch'].create({
                'is_wave': True,
                'picking_type_id': self.picking_type_id and self.picking_type_id[0].id,
                'user_id': self.env.context.get('active_owner_id'),
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
                if move.from_immediate_transfer:
                    qty = line.product_uom_id._compute_quantity(line.qty_done, line.product_id.uom_id, rounding_method='HALF-UP')
                else:
                    qty = line.reserved_qty
                qty_by_move[line.move_id] += qty

            if lines == picking.move_line_ids and lines.move_id == picking.move_ids:
                move_complete = True
                for move, qty in qty_by_move.items():
                    if float_compare(move.product_qty, qty, precision_rounding=move.product_uom.rounding) != 0:
                        move_complete = False
                        break
                if move_complete:
                    wave.picking_ids = [Command.link(picking.id)]
                    continue

            # Split the picking in two part to extract only line that are taken on the wave
            picking_to_wave_vals = picking.copy_data({
                'move_ids': [],
                'move_line_ids': [],
                'batch_id': wave.id,
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
        wave.action_confirm()

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
    batch_group_by_src_loc = fields.Boolean('Source Location',
                                            help="Automatically group batches by their source location.")
    batch_group_by_dest_loc = fields.Boolean('Destination Location',
                                             help="Automatically group batches by their destination location.")
    batch_max_lines = fields.Integer("Maximum lines per batch",
                                     help="A transfer will not be automatically added to batches that will exceed this number of lines if the transfer is added to it.\n"
                                          "Leave this value as '0' if no line limit.")
    batch_max_pickings = fields.Integer("Maximum transfers per batch",
                                        help="A transfer will not be automatically added to batches that will exceed this number of transfers.\n"
                                             "Leave this value as '0' if no transfer limit.")
    batch_auto_confirm = fields.Boolean("Auto-confirm", default=True)

    def _compute_picking_count(self):
        super()._compute_picking_count()
        domains = {
            'count_picking_batch': [('is_wave', '=', False)],
            'count_picking_wave': [('is_wave', '=', True)],
        }
        for field in domains:
            data = self.env['stock.picking.batch']._read_group(domains[field] +
                [('state', 'not in', ('done', 'cancel')), ('picking_type_id', 'in', self.ids)],
                ['picking_type_id'], ['picking_type_id'])
            count = {
                x['picking_type_id'][0]: x['picking_type_id_count']
                for x in data if x['picking_type_id']
            }
            for record in self:
                record[field] = count.get(record.id, 0)

    @api.model
    def _get_batch_group_by_keys(self):
        return ['batch_group_by_partner', 'batch_group_by_destination', 'batch_group_by_src_loc', 'batch_group_by_dest_loc']

    @api.constrains(lambda self: self._get_batch_group_by_keys() + ['auto_batch'])
    def _validate_auto_batch_group_by(self):
        group_by_keys = self._get_batch_group_by_keys()
        for picking_type in self:
            if not picking_type.auto_batch:
                continue
            if not any(picking_type[key] for key in group_by_keys):
                raise ValidationError(_("If the Automatic Batches feature is enabled, at least one 'Group by' option must be selected."))

    def get_action_picking_tree_batch(self):
        return self._get_action('stock_picking_batch.stock_picking_batch_action')

    def get_action_picking_tree_wave(self):
        return self._get_action('stock_picking_batch.action_picking_tree_wave')


class StockPicking(models.Model):
    _inherit = "stock.picking"

    batch_id = fields.Many2one(
        'stock.picking.batch', string='Batch Transfer',
        check_company=True,
        states={'done': [('readonly', True)], 'cancel': [('readonly', True)]},
        help='Batch associated to this transfer', index=True, copy=False)

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
            'views': [(view.id, 'tree')],
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
        if self and self.env.context.get('pickings_to_detach'):
            self.env['stock.picking'].browse(self.env.context['pickings_to_detach']).batch_id = False
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

        return res

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
        if not self.picking_type_id.auto_batch or self.immediate_transfer or self.batch_id or not self.move_ids or not self._is_auto_batchable():
            return False

        # Try to find a compatible batch to insert the picking
        possible_batches = self.env['stock.picking.batch'].sudo().search(self._get_possible_batches_domain())
        for batch in possible_batches:
            if batch._is_picking_auto_mergeable(self):
                batch.picking_ids |= self
                return batch

        # If no batch were found, try to find a compatible picking and put them both in a new batch.
        possible_pickings = self.env['stock.picking'].search(self._get_possible_pickings_domain())
        for picking in possible_pickings:
            if self._is_auto_batchable(picking):
                # Create new batch with both pickings
                new_batch = self.env['stock.picking.batch'].sudo().create({
                    'picking_ids': [Command.link(self.id), Command.link(picking.id)],
                    'company_id': self.company_id.id if self.company_id else False,
                    'picking_type_id': self.picking_type_id.id,
                })
                if picking.picking_type_id.batch_auto_confirm:
                    new_batch.action_confirm()
                return new_batch

        # If nothing was found after those two steps, then no batch is doable given the conditions
        return False

    def _is_auto_batchable(self, picking=None):
        """ Verifies if a picking can be put in a batch with another picking without violating auto_batch constrains.
        """
        if self.state not in ('waiting', 'confirmed', 'assigned'):
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
            ('immediate_transfer', '=', False),
            ('state', 'in', ('waiting', 'confirmed', 'assigned')),
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

    def _package_move_lines(self, batch_pack=False):
        if batch_pack:
            return super(StockPicking, self.batch_id.picking_ids if self.batch_id else self)._package_move_lines(batch_pack)
        return super()._package_move_lines(batch_pack)

    def assign_batch_user(self, user_id):
        pickings = self.filtered(lambda p: p.user_id.id != user_id)
        pickings.write({'user_id': user_id})
        for pick in pickings:
            if user_id:
                log_message = _('Assigned to %s Responsible', (pick.batch_id._get_html_link()))
            else:
                log_message = _('Unassigned responsible from %s', (pick.batch_id._get_html_link()))
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

from odoo import api, fields, models, _
from odoo.exceptions import UserError
from odoo.osv.expression import AND
from odoo.tools.float_utils import float_is_zero

class StockPickingBatch(models.Model):
    _inherit = ['mail.thread', 'mail.activity.mixin']
    _name = "stock.picking.batch"
    _description = "Batch Transfer"
    _order = "name desc"

    name = fields.Char(
        string='Batch Transfer', default='New',
        copy=False, required=True, readonly=True)
    user_id = fields.Many2one(
        'res.users', string='Responsible', tracking=True, check_company=True,
        readonly=True, states={'draft': [('readonly', False)], 'in_progress': [('readonly', False)]})
    company_id = fields.Many2one(
        'res.company', string="Company", required=True, readonly=True,
        index=True, default=lambda self: self.env.company)
    picking_ids = fields.One2many(
        'stock.picking', 'batch_id', string='Transfers', readonly=True,
        domain="[('id', 'in', allowed_picking_ids)]", check_company=True,
        states={'draft': [('readonly', False)], 'in_progress': [('readonly', False)]},
        help='List of transfers associated to this batch')
    show_check_availability = fields.Boolean(
        compute='_compute_move_ids',
        string='Show Check Availability')
    show_validate = fields.Boolean(
        compute='_compute_show_validate',
        string='Show Validate Button')
    show_allocation = fields.Boolean(
        compute='_compute_show_allocation',
        string='Show Allocation Button')
    allowed_picking_ids = fields.One2many('stock.picking', compute='_compute_allowed_picking_ids')
    move_ids = fields.One2many(
        'stock.move', string="Stock moves", compute='_compute_move_ids')
    move_line_ids = fields.One2many(
        'stock.move.line', string='Stock move lines',
        compute='_compute_move_ids', inverse='_set_move_line_ids', readonly=True,
        states={'draft': [('readonly', False)], 'in_progress': [('readonly', False)]})
    state = fields.Selection([
        ('draft', 'Draft'),
        ('in_progress', 'In progress'),
        ('done', 'Done'),
        ('cancel', 'Cancelled')], default='draft',
        store=True, compute='_compute_state',
        copy=False, tracking=True, required=True, readonly=True, index=True)
    picking_type_id = fields.Many2one(
        'stock.picking.type', 'Operation Type', check_company=True, copy=False,
        readonly=True, index=True, states={'draft': [('readonly', False)]})
    picking_type_code = fields.Selection(
        related='picking_type_id.code')
    scheduled_date = fields.Datetime(
        'Scheduled Date', copy=False, store=True, readonly=False, compute="_compute_scheduled_date",
        states={'done': [('readonly', True)], 'cancel': [('readonly', True)]},
        help="""Scheduled date for the transfers to be processed.
              - If manually set then scheduled date for all transfers in batch will automatically update to this date.
              - If not manually changed and transfers are added/removed/updated then this will be their earliest scheduled date
                but this scheduled date will not be set for all transfers in batch.""")
    is_wave = fields.Boolean('This batch is a wave')
    show_set_qty_button = fields.Boolean(compute='_compute_show_qty_button')
    show_clear_qty_button = fields.Boolean(compute='_compute_show_qty_button')
    show_lots_text = fields.Boolean(compute='_compute_show_lots_text')

    @api.depends('state', 'show_validate',
                 'picking_ids.show_set_qty_button',
                 'picking_ids.show_clear_qty_button')
    def _compute_show_qty_button(self):
        self.show_set_qty_button = False
        self.show_clear_qty_button = False
        for batch in self:
            if not batch.show_validate or batch.state != 'in_progress':
                continue
            if any(p.show_set_qty_button for p in self.picking_ids):
                batch.show_set_qty_button = True
            elif any(p.show_clear_qty_button for p in self.picking_ids):
                batch.show_clear_qty_button = True

    @api.depends('picking_type_id')
    def _compute_show_lots_text(self):
        for batch in self:
            batch.show_lots_text = batch.picking_ids and batch.picking_ids[0].show_lots_text

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
            if not batch.is_wave:
                domain = AND([domain, [('immediate_transfer', '=', False)]])
            if batch.picking_type_id:
                domain += [('picking_type_id', '=', batch.picking_type_id.id)]
            batch.allowed_picking_ids = self.env['stock.picking'].search(domain)

    @api.depends('picking_ids', 'picking_ids.move_line_ids', 'picking_ids.move_ids', 'picking_ids.move_ids.state')
    def _compute_move_ids(self):
        for batch in self:
            batch.move_ids = batch.picking_ids.move_ids
            batch.move_line_ids = batch.picking_ids.move_line_ids
            batch.show_check_availability = any(m.state not in ['assigned', 'cancel', 'done'] for m in batch.move_ids)

    @api.depends('picking_ids', 'picking_ids.show_validate')
    def _compute_show_validate(self):
        for batch in self:
            batch.show_validate = any(picking.show_validate for picking in batch.picking_ids)

    @api.depends('state', 'move_ids', 'picking_type_id')
    def _compute_show_allocation(self):
        self.show_allocation = False
        if not self.user_has_groups('stock.group_reception_report'):
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

    def onchange(self, values, field_name, field_onchange):
        """Override onchange to NOT to update all scheduled_date on pickings when
        scheduled_date on batch is updated by the change of scheduled_date on pickings.
        """
        result = super().onchange(values, field_name, field_onchange)
        if field_name == 'picking_ids' and 'value' in result:
            for line in result['value'].get('picking_ids', []):
                if line[0] < 2 and 'scheduled_date' in line[2]:
                    del line[2]['scheduled_date']
        return result

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

    def action_set_quantities_to_reservation(self):
        self.picking_ids.filtered("show_set_qty_button").action_set_quantities_to_reservation()

    def action_clear_quantities_to_zero(self):
        self.picking_ids.filtered("show_clear_qty_button").action_clear_quantities_to_zero()

    def action_done(self):
        def has_no_qty_done(picking):
            return all(float_is_zero(line.qty_done, precision_rounding=line.product_uom_id.rounding) for line in picking.move_line_ids if line.state not in ('done', 'cancel'))

        self.ensure_one()
        self._check_company()
        # Empty 'waiting for another operation' pickings will be removed from the batch when it is validated.
        pickings = self.mapped('picking_ids').filtered(lambda picking: picking.state not in ('cancel', 'done'))
        empty_waiting_pickings = self.mapped('picking_ids').filtered(lambda p: p.state == 'waiting' and has_no_qty_done(p))
        pickings = pickings - empty_waiting_pickings

        empty_pickings = set()
        for picking in pickings:
            if has_no_qty_done(picking):
                empty_pickings.add(picking.id)
            picking.message_post(
                body="<b>%s:</b> %s <a href=#id=%s&view_type=form&model=stock.picking.batch>%s</a>" % (
                    _("Transferred by"),
                    _("Batch Transfer"),
                    picking.batch_id.id,
                    picking.batch_id.name))

        # Run sanity_check as a batch and ignore the one in button_validate() since it is done here.
        pickings._sanity_check(separate_pickings=False)
        # Skip sanity_check in pickings button_validate() & remove 'waiting' pickings from the batch
        context = {'skip_sanity_check': True, 'pickings_to_detach': empty_waiting_pickings.ids}
        if len(empty_pickings) == len(pickings):
            return pickings.with_context(**context).button_validate()
        else:
            # If some pickings are at least partially done, other pickings (empty & waiting) will be removed from batch without being cancelled in case of no backorder
            pickings = pickings - self.env['stock.picking'].browse(empty_pickings)
            context['pickings_to_detach'] = context['pickings_to_detach'] + list(empty_pickings)
            return pickings.with_context(skip_immediate=True, **context).button_validate()

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
                if not res:
                    res = move_line_ids.picking_id[0]._put_in_pack(move_line_ids, False)
                return res
            else:
                raise UserError(_("Please add 'Done' quantities to the batch picking to create a new pack."))

    def action_view_reception_report(self):
        action = self.picking_ids[0].action_view_reception_report()
        action['context'] = {'default_picking_ids': self.picking_ids.ids}
        return action

    def action_open_label_layout(self):
        if self.user_has_groups('stock.group_production_lot') and self.move_line_ids.lot_id:
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
                'default_move_line_ids': self.move_line_ids.ids,
                'default_picking_quantity': 'picking'},
        }

    # -------------------------------------------------------------------------
    # Miscellaneous
    # -------------------------------------------------------------------------
    def _sanity_check(self):
        for batch in self:
            if not batch.picking_ids <= batch.allowed_picking_ids:
                erroneous_pickings = batch.picking_ids - batch.allowed_picking_ids
                raise UserError(_(
                    "The following transfers cannot be added to batch transfer %s. "
                    "Please check their states and operation types, if they aren't immediate "
                    "transfers.\n\n"
                    "Incompatibilities: %s", batch.name, ', '.join(erroneous_pickings.mapped('name'))))

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
                <t t-foreach="docs" t-as="o">
                    <t t-set="move_line_ids" t-value="o.picking_ids.mapped('move_line_ids')"/>
                    <t t-set="has_package" t-value="move_line_ids.filtered('result_package_id')" groups="stock.group_tracking_lot"/>
                    <t t-set="has_serial_number" t-value="move_line_ids.filtered('lot_id')" groups="stock.group_production_lot"/>
                    <t t-set="has_barcode" t-value="move_line_ids.mapped('product_id').filtered('barcode')"/>
                    <t t-set="locations" t-value="move_line_ids.mapped('location_id').sorted(lambda location: location.complete_name)"/>
                    <t t-call="web.external_layout">
                        <div class="page">
                            <div class="d-flex">
                                <div><h3>Summary: <span t-field="o.name"/></h3></div>
                                <div class="me-auto">
                                    <div t-field="o.name" t-options="{'widget': 'barcode', 'width': 600, 'height': 150, 'img_style': 'width:300px;height:50px;'}"/>
                                </div>
                            </div>
                            <div t-if="o.user_id">
                                <strong>Responsible:</strong>
                                <span t-field="o.user_id"/>
                            </div><br/>
                            <table class="table table-condensed">
                                <thead>
                                    <tr>
                                        <th>Transfer</th>
                                        <th>Barcode</th>
                                        <th>Status</th>
                                        <th>Scheduled Date</th>
                                    </tr>
                                </thead>
                                <tbody>
                                    <tr t-foreach="o.picking_ids" t-as="pick">
                                        <td>
                                            <span t-field="pick.name"/>
                                        </td>
                                        <td>
                                            <div t-field="pick.name" t-options="{'widget': 'barcode', 'quiet': 0, 'width': 400, 'height': 100, 'img_style': 'width:200px;height:50px;'}"/>
                                        </td>
                                        <td>
                                            <span t-field="pick.state"/>
                                        </td>
                                        <td >
                                            <span t-field="pick.scheduled_date"/>
                                        </td>
                                    </tr>
                                </tbody>
                            </table>
                            <p style="page-break-after: always;"/>
                            <h3><span t-field="o.name"/></h3>
                            <div t-if="o.user_id">
                                <strong>Responsible:</strong>
                                <span t-field="o.user_id"/>
                            </div><br/>
                            <table class="table table-condensed">
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
                                        <t t-set="move_lines" t-value="loc_move_line.filtered(lambda x: x.location_dest_id==location_dest)"/>
                                        <t t-set="products" t-value="move_lines.product_id"/>
                                        <tbody>
                                            <tr>
                                                <td style="background-color:lightgrey" colspan="6" >
                                                    <p style="margin:0px"><strong>FROM</strong>
                                                    <span t-esc="move_lines.location_id.display_name"/></p>
                                                    <p style="margin:0px"><strong>TO</strong>
                                                    <span t-esc="move_lines.location_dest_id.display_name"/></p>
                                                </td>
                                            </tr>
                                            <tr t-foreach="move_lines" t-as="move_operation">
                                                <td>
                                                    <span t-field="move_operation.display_name"/>
                                                </td>
                                                <td>
                                                    <span t-if="has_package or move_operation.state == 'done'"
                                                          t-esc="sum(move_operation.mapped('qty_done'))"/>
                                                    <span t-else="" t-esc="sum(move_operation.mapped('reserved_uom_qty'))"/>
                                                    <span t-field="move_operation.product_uom_id" groups="uom.group_uom"/>
                                                </td>
                                                <td>
                                                    <span t-esc="move_operation.picking_id.display_name"/>
                                                </td>
                                                <td t-if="has_serial_number" class="text-center h6" width="15%">
                                                    <div t-if="move_operation.lot_id or move_operation.lot_name" t-field="move_operation.lot_id.name" t-options="{'widget': 'barcode', 'humanreadable': 1, 'width': 600, 'height': 100, 'img_style': 'width:100%;height:35px;'}"/>
                                                </td>
                                                <td width="15%" class="text-center" t-if="has_barcode">
                                                    <span t-if="move_operation.product_id and move_operation.product_id.barcode">
                                                        <div t-field="move_operation.product_id.barcode" t-options="{'widget': 'barcode', 'symbology': 'auto', 'width': 600, 'height': 100, 'img_style': 'width:100%;height:35px;'}"/>
                                                    </span>
                                                </td>
                                                <td t-if="has_package" width="15%">
                                                    <span t-field="move_operation.package_id"/>
                                                    <t t-if="move_operation.result_package_id">
                                                        <strong>→</strong> <span t-field="move_operation.result_package_id"/>
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

## File: views\stock_move_line_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_move_line_tree_detailed_wave" model="ir.ui.view">
        <field name="name">stock_picking_wave.move.line.tree.wave</field>
        <field name="model">stock.move.line</field>
        <field name="inherit_id" ref="stock.view_move_line_tree_detailed"/>
        <field name="arch" type="xml">
            <xpath expr="//tree" position="inside">
                <header>
                    <button name="action_open_add_to_wave" type="object" string="Add to Wave" groups="stock.group_stock_picking_wave"/>
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
    <record id="view_picking_tree_batch" model="ir.ui.view">
        <field name="name">stock_picking_batch.picking.tree.batch</field>
        <field name="model">stock.picking</field>
        <field name="mode">primary</field>
        <field name="priority" eval="1000"/>
        <field name="arch" type="xml">
            <tree>
                <field name="name"/>
                <field name="scheduled_date"/>
                <field name="location_id"/>
                <field name="backorder_id"/>
                <field name="origin"/>
                <field name="state"/>
            </tree>
        </field>
    </record>

    <record id="view_picking_move_tree_inherited" model="ir.ui.view">
        <field name="name">stock_picking_batch.picking.move.tree</field>
        <field name="model">stock.move</field>
        <field name="inherit_id" ref="stock.view_picking_move_tree"/>
        <field name="mode">primary</field>
        <field name="priority" eval="1000"/>
        <field name="arch" type="xml">
            <xpath expr="//tree" position="attributes">
                <attribute name="delete">0</attribute>
            </xpath>
            <xpath expr="//field[@name='product_id']" position="after">
                <field name="picking_id" required="1"
                    attrs="{'readonly': [('id', '!=', False)]}"
                    domain="[('id', 'in', parent.picking_ids)]"
                    options="{'no_create_edit': True}"/>
            </xpath>
            <xpath expr="//field[@name='quantity_done']" position="attributes">
                <attribute name="readonly">1</attribute>
            </xpath>
        </field>
    </record>

    <record id="view_move_line_tree" model="ir.ui.view">
        <field name="name">stock_picking_batch.move.line.tree</field>
        <field name="model">stock.move.line</field>
        <field name="arch" type="xml">
            <tree editable="top" decoration-muted="state == 'cancel'" string="Move Lines" default_order="location_id">
                <field name="tracking" invisible="1"/>
                <field name="state" invisible="1"/>
                <field name="company_id" invisible="1"/>
                <field name="product_id" context="{'default_detailed_type': 'product'}" required="1" attrs="{'readonly': [('id', '!=', False)]}"/>
                <field name="picking_id" required="1" attrs="{'readonly': [('id', '!=', False)]}"
                    options="{'no_create_edit': True}" domain="[('id', 'in', parent.picking_ids)]"/>
                <field name="lot_id" groups="stock.group_production_lot" attrs="{'column_invisible': [('parent.show_lots_text', '=', True)], 'readonly': [('tracking', 'not in', ['lot', 'serial'])]}"/>
                <field name="lot_name" groups="stock.group_production_lot" attrs="{'column_invisible': [('parent.show_lots_text', '=', False)], 'readonly': [('tracking', 'not in', ['lot', 'serial'])]}"/>
                <field name="location_id"/>
                <field name="location_dest_id"/>
                <field name="package_id" groups="stock.group_tracking_lot"/>
                <field name="result_package_id" groups="stock.group_tracking_lot"/>
                <field name="reserved_uom_qty" readonly="1"/>
                <field name="product_uom_id" options="{'no_create': True}"
                    groups="uom.group_uom" readonly="1" force_save="1"/>
                <field name="qty_done"/>
                <field name="company_id" groups="base.group_multi_company" force_save="1"/>
            </tree>
        </field>
    </record>

    <record id="stock_picking_batch_form" model="ir.ui.view">
        <field name="name">stock.picking.batch.form</field>
        <field name="model">stock.picking.batch</field>
        <field name="arch" type="xml">
            <form string="Stock Batch Transfer">
                <field name="company_id" invisible="1"/>
                <field name="show_check_availability" invisible="1"/>
                <field name="show_validate" invisible="1"/>
                <field name="show_allocation" invisible="1"/>
                <field name="picking_type_code" invisible="1"/>
                <field name="is_wave" invisible="1"/>
                <field name="show_set_qty_button" invisible="1"/>
                <field name="show_clear_qty_button" invisible="1"/>
                <field name="show_lots_text" invisible="1"/>
                <header>
                    <button name="action_confirm" states="draft" string="Confirm" type="object" class="oe_highlight"/>
                    <button name="action_done" string="Validate" type="object" class="oe_highlight"
                        attrs="{'invisible': [
                            '|',
                                ('state', '!=', 'in_progress'),
                                '|',
                                    ('show_check_availability', '=', True),
                                    ('show_validate', '=', False)]}"/>
                    <button name="action_assign" string="Check Availability" type="object" class="oe_highlight"
                        attrs="{'invisible': [
                            '|',
                                ('state', '!=', 'in_progress'),
                                ('show_check_availability', '=', False)]}"/>
                    <button name="action_done" string="Validate" type="object"
                        attrs="{'invisible': [
                            '|',
                                ('state', '!=', 'in_progress'),
                                '|',
                                    ('show_check_availability', '=', False),
                                    ('show_validate', '=', False)]}"/>
                    <button name="action_set_quantities_to_reservation" string="Set quantities" type="object"
                        attrs="{'invisible': [('show_set_qty_button', '=', False)]}"/>
                    <button name="action_clear_quantities_to_zero" string="Clear quantities" type="object"
                        attrs="{'invisible': [('show_clear_qty_button', '=', False)]}"/>
                    <button name="action_assign" string="Check Availability" type="object"
                        attrs="{'invisible': [
                            '|',
                                ('state', '!=', 'draft'),
                                ('show_check_availability', '=', False)]}"/>
                    <button name="action_print" states="in_progress,done" string="Print" type="object"/>
                    <button string="Print Labels" type="object" name="action_open_label_layout"/>
                    <button name="action_cancel" string="Cancel" type="object" states="in_progress"/>
                    <field name="state" widget="statusbar" statusbar_visible="draft,in_progress,done"/>
                </header>
                <sheet>
                    <div class="oe_button_box" name="button_box">
                        <button name="action_view_reception_report" string="Allocation" type="object"
                            class="oe_stat_button" icon="fa-list"
                            attrs="{'invisible': [('show_allocation', '=', False)]}"
                            groups="stock.group_reception_report"/>
                    </div>
                    <div class="oe_title">
                        <h1><field name="name" class="oe_inline"/></h1>
                    </div>
                    <group>
                        <group>
                            <field name="user_id"/>
                            <field name="picking_type_id"/>
                            <field name="company_id" groups="base.group_multi_company" attrs="{'readonly': [('picking_ids', '!=', [])]}" force_save="1"/>
                        </group>
                        <group>
                            <field name="scheduled_date"/>
                        </group>
                    </group>
                    <notebook>
                        <page string="Detailed Operations" attrs="{'invisible': [('state', '=', 'draft')]}">
                            <field name="move_line_ids" context="{'tree_view_ref': 'stock_picking_batch.view_move_line_tree'}"/>
                            <button class="oe_highlight" name="action_put_in_pack" type="object" string="Put in Pack" attrs="{'invisible': [('state', 'in', ('draft', 'done', 'cancel'))]}" groups="stock.group_tracking_lot"/>
                        </page>
                        <page string="Operations" attrs="{'invisible': [('state', '=', 'draft')]}">
                            <field name="move_ids" context="{'tree_view_ref': 'stock_picking_batch.view_picking_move_tree_inherited'}"/>
                        </page>
                        <page string="Transfers">
                            <field name="allowed_picking_ids" invisible="1"/>
                            <field name="picking_ids" widget="many2many" mode="tree,kanban"
                                context="{'form_view_ref': 'stock_picking_batch.view_picking_form_inherited', 'tree_view_ref': 'stock_picking_batch.view_picking_tree_batch'}"/>
                        </page>
                    </notebook>
                </sheet>
                <div class="oe_chatter">
                    <field name="message_follower_ids" groups="base.group_user"/>
                    <field name="activity_ids"/>
                    <field name="message_ids"/>
                </div>
            </form>
        </field>
    </record>
    <record id="stock_picking_batch_tree" model="ir.ui.view">
        <field name="name">stock.picking.batch.tree</field>
        <field name="model">stock.picking.batch</field>
        <field name="arch" type="xml">
            <tree string="Stock Batch Transfer" multi_edit="1" sample="1">
                <field name="company_id" invisible="1"/>
                <field name="name" decoration-bf="1"/>
                <field name="scheduled_date"/>
                <field name="user_id" widget="many2one_avatar_user"/>
                <field name="picking_type_id"/>
                <field name="company_id" groups="base.group_multi_company"/>
                <field name="state" widget="badge" decoration-success="state == 'done'" decoration-info="state in ('draft', 'in_progress')"/>
                <field name="activity_exception_decoration" widget="activity_exception"/>
            </tree>
        </field>
    </record>
    <record id="stock_picking_batch_kanban" model="ir.ui.view">
        <field name="name">stock.picking.batch.kanban</field>
        <field name="model">stock.picking.batch</field>
        <field name="arch" type="xml">
            <kanban class="o_kanban_mobile" sample="1">
                <field name="name"/>
                <field name="user_id"/>
                <field name="state"/>
                <templates>
                    <t t-name="kanban-box">
                        <div t-attf-class="oe_kanban_global_click">
                            <div class="o_kanban_record_top mb16">
                                <div class="o_kanban_record_headings">
                                    <strong class="o_kanban_record_title"><field name="name"/></strong>
                                </div>
                                <field name="state" widget="label_selection"/>
                            </div>
                            <div class="o_kanban_record_bottom">
                                <div class="oe_kanban_bottom_left">
                                    <field name="picking_type_id"/>
                                </div>
                                <div class="oe_kanban_bottom_right">
                                    <field name="scheduled_date" widget="date"/>
                                    <field name="user_id" widget="many2one_avatar_user"/>
                                </div>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
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
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">stock.picking.batch</field>
        <field name="view_mode">tree,kanban,form</field>
        <field name="domain">[('is_wave', '=', False)]</field>
        <field name="context">{'search_default_draft': True, 'search_default_in_progress': True}</field>
        <field name="search_view_id" ref="stock_picking_batch_filter"/>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Create a new batch transfer
          </p><p>
            The goal of the batch transfer is to group operations that may
            (needs to) be done together in order to increase their efficiency.
            It may also be useful to assign jobs (one person = one batch) or
            help the timing management of operations (tasks to be done at 1pm).
          </p>
        </field>
    </record>
    <menuitem action="stock_picking_batch_action" id="stock_picking_batch_menu" parent="stock.menu_stock_warehouse_mgmt" sequence="21"/>

    <record id="vpicktree_inherit_stock_picking_batch" model="ir.ui.view">
        <field name="name">stock.picking.tree</field>
        <field name="model">stock.picking</field>
        <field name="inherit_id" ref="stock.vpicktree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='picking_type_id']" position="after">
                <field name="batch_id" optional="show"
                    domain="[
                        ('state', 'in', ['draft', 'in_progress']),
                        '|',
                            ('picking_type_id', '=', picking_type_id),
                            ('picking_type_id', '=', False),
                    ]"
                    widget="many2one"
                    context="{'default_picking_type_id': picking_type_id}"/>
            </xpath>
        </field>
    </record>
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
        <field name="name">stock.move.line.tree.stock_picking_batch</field>
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
                <group name="batch" attrs="{'invisible': [('code', 'not in', ('incoming', 'outgoing', 'internal'))]}">
                    <group string="Batch Transfers">
                        <field name="auto_batch"/>
                        <span class="o_form_label fw-bold" attrs="{'invisible':[('auto_batch', '=', False)]}">Group by</span>
                        <div name="batch_contact" class="o_row" attrs="{'invisible':[('auto_batch', '=', False)]}">
                            <field name="batch_group_by_partner"/>
                            <label for="batch_group_by_partner" string="Contact"/>
                        </div>
                        <span attrs="{'invisible':[('auto_batch', '=', False)]}"/>
                        <div name="batch_destination" class="o_row" attrs="{'invisible':[('auto_batch', '=', False)]}">
                            <field name="batch_group_by_destination"/>
                            <label for="batch_group_by_destination"/>
                        </div>
                        <span attrs="{'invisible':['|', ('auto_batch', '=', False), ('default_location_src_id', '=', False)]}" groups="stock.group_stock_multi_locations"/>
                        <div name="batch_source_location" class="o_row" attrs="{'invisible':['|', ('auto_batch', '=', False), ('default_location_src_id', '=', False)]}" groups="stock.group_stock_multi_locations">
                            <field name="batch_group_by_src_loc"/>
                            <label for="batch_group_by_src_loc"/>
                        </div>
                        <span attrs="{'invisible':['|', ('auto_batch', '=', False), ('default_location_dest_id', '=', False)]}" groups="stock.group_stock_multi_locations"/>
                        <div name="batch_dest_subloc" class="o_row" attrs="{'invisible':['|', ('auto_batch', '=', False), ('default_location_dest_id', '=', False)]}" groups="stock.group_stock_multi_locations">
                            <field name="batch_group_by_dest_loc"/>
                            <label for="batch_group_by_dest_loc"/>
                        </div>
                        <field name="batch_max_lines" attrs="{'invisible': [('auto_batch', '=', False)]}"/>
                        <field name="batch_max_pickings" attrs="{'invisible': [('auto_batch', '=', False)]}"/>
                        <field name="batch_auto_confirm" attrs="{'invisible': [('auto_batch', '=', False)]}"/>
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
                <field name="batch_id" invisible="1"/>
                <button type="object"
                        name="action_view_batch"
                        class="oe_stat_button"
                        icon="fa-truck"
                        string="Batch"
                        attrs="{'invisible': [('batch_id', '=', False)]}"/>
            </div>
        </field>
    </record>
</odoo>

```

## File: views\stock_picking_wave_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="stock_picking_wave_tree" model="ir.ui.view">
        <field name="name">stock.picking.wave.tree</field>
        <field name="model">stock.picking.batch</field>
        <field name="priority">25</field>
        <field name="mode">primary</field>
        <field name="inherit_id" ref="stock_picking_batch.stock_picking_batch_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//tree" position="attributes">
                <attribute name="create">0</attribute>
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
        </field>
    </record>
    <record id="action_picking_tree_wave" model="ir.actions.act_window">
        <field name="name">Wave Transfers</field>
        <field name="res_model">stock.picking.batch</field>
        <field name="type">ir.actions.act_window</field>
        <field name="view_mode">tree,kanban,form</field>
        <field name="context">{'search_default_draft': True, 'search_default_in_progress': True}</field>
        <field name="view_ids" eval="[(5, 0, 0),
            (0, 0, {'view_mode': 'tree', 'view_id': ref('stock_picking_batch.stock_picking_wave_tree')}),
            (0, 0, {'view_mode': 'kanban', 'view_id': ref('stock_picking_batch.stock_picking_wave_kanban')})]"/>
        <field name="domain">[('is_wave', '=', True)]</field>
        <field name="search_view_id" ref="stock_picking_batch_filter"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a new wave transfer
                </p><p>
                    The goal of the wave transfer is to group operations from different transfer
                    together in order to increase their efficiency.
                    It may also be useful to assign jobs (one person = one batch) or
                    help the timing management of operations (tasks to be done at 1pm).
            </p>
        </field>
    </record>

    <record id="stock_picking_type_kanban_batch" model="ir.ui.view">
        <field name="name">picking.type.kanban.batch</field>
        <field name="model">stock.picking.type</field>
        <field name="inherit_id" ref="stock.stock_picking_type_kanban"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='picking_type_backorder_count']" position="after">
                <div t-if="record.count_picking_batch.raw_value > 0" class="row">
                    <div class="col-12">
                        <a class="oe_kanban_stock_picking_type_list" name="get_action_picking_tree_batch" type="object">
                            <field name="count_picking_batch"/>
                            Batches
                        </a>
                    </div>
                </div>
                <div t-if="record.count_picking_wave.raw_value > 0" class="row">
                    <div class="col-12">
                        <a class="oe_kanban_stock_picking_type_list" name="get_action_picking_tree_wave" type="object">
                            <field name="count_picking_wave"/>
                            Waves
                        </a>
                    </div>
                </div>
            </xpath>
        </field>
    </record>
    <menuitem
        action="action_picking_tree_wave"
        id="stock_picking_wave_menu"
        parent="stock.menu_stock_warehouse_mgmt"
        groups="stock.group_stock_picking_wave"
        sequence="23"/>

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
            'views': [(view.id, 'tree')],
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
                        <field name="wave_id" options="{'no_create': True, 'no_open': True}" attrs="{'invisible': [('mode', '=', 'new')], 'required': [('mode', '=', 'existing')]}"/>
                        <field name="user_id" options="{'no_create': True}" attrs="{'invisible': [('mode', '=', 'existing')]}"/>
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
        <field name="groups_id" eval="[(4, ref('stock.group_stock_picking_wave'))]"/>
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
            destination.move_line_ids = destination.picking_id.batch_id.move_line_ids.filtered(lambda l: l.qty_done > 0 and not l.result_package_id)
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
            })
        else:
            batch = self.batch_id

        pickings.write({'batch_id': batch.id})
        # you have to set some pickings to batch before confirm it.
        if self.mode == 'new' and not self.is_create_draft:
            batch.action_confirm()

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
                        <field name="batch_id" options="{'no_create': True, 'no_open': True}" attrs="{'invisible': [('mode', '=', 'new')], 'required': [('mode', '=', 'existing')]}"/>
                        <field name="user_id" options="{'no_create': True}" attrs="{'invisible': [('mode', '=', 'existing')]}"/>
                        <field name="is_create_draft"  attrs="{'invisible': [('mode', '!=', 'new')]}"/>
                    </group>
                </group>

                <footer>
                    <button name="attach_pickings" type="object" string="Confirm" class="btn-primary" data-hotkey="q"/>
                    <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="z"/>
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

