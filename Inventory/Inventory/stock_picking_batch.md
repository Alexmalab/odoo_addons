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
        'data/stock_picking_batch_data.xml',
        'wizard/stock_picking_to_batch_views.xml',
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
</data></odoo>

```

## File: data\stock_picking_batch_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">
    <!-- Add batch picking -->
    <record id="stock_picking_batch_1" model="stock.picking.batch">
        <field name="picking_type_id" ref="stock.picking_type_out"/>
        <field name="company_id" ref="base.main_company"/>
    </record>
    <record id="stock_picking_batch_2" model="stock.picking.batch">
        <field name="picking_type_id" ref="stock.picking_type_out"/>
        <field name="company_id" ref="base.main_company"/>
    </record>

    <!-- Resource: stock.inventory -->
    <record id="stock_inventory_1" model="stock.inventory">
        <field name="name">Starting Inventory</field>
    </record>

    <!-- Resource: stock.inventory.line -->
    <record id="stock_inventory_line_0" model="stock.inventory.line">
        <field name="product_id" ref="product.consu_delivery_01"/>
        <field name="product_uom_id" ref="uom.product_uom_unit"/>
        <field name="inventory_id" ref="stock_picking_batch.stock_inventory_1"/>
        <field name="product_qty">10.0</field>
        <field name="location_id" ref="stock.stock_location_stock"/>
    </record>

    <record id="stock_inventory_line_1" model="stock.inventory.line">
        <field name="product_id" ref="product.consu_delivery_02"/>
        <field name="product_uom_id" ref="uom.product_uom_unit"/>
        <field name="inventory_id" ref="stock_picking_batch.stock_inventory_1"/>
        <field name="product_qty">10.0</field>
        <field name="location_id" ref="stock.stock_location_stock"/>
    </record>

    <record id="stock_inventory_line_3" model="stock.inventory.line">
        <field name="product_id" ref="product.consu_delivery_03"/>
        <field name="product_uom_id" ref="uom.product_uom_unit"/>
        <field name="inventory_id" ref="stock_picking_batch.stock_inventory_1"/>
        <field name="product_qty">20.0</field>
        <field name="location_id" ref="stock.stock_location_stock"/>
    </record>

    <!-- Inventory start -->
    <function model="stock.inventory" name="_action_start">
        <function eval="[[('state','=','draft'),('id', '=', ref('stock_inventory_1'))]]" model="stock.inventory" name="search"/>
    </function>
    <!-- Inventory validate -->
    <function model="stock.inventory" name="action_validate">
        <function eval="[[('state','=','confirm'),('id', '=', ref('stock_inventory_1'))]]" model="stock.inventory" name="search"/>
    </function>

    <!-- Add picking -->
    <record id="Picking_A" model="stock.picking">
        <field name="move_type">one</field>
        <field name="priority">1</field>
        <field name="user_id" eval="False"/>
        <field name="picking_type_id" ref="stock.picking_type_out"/>
        <field name="batch_id" ref="stock_picking_batch_2"/>
        <field name="location_id" ref="stock.stock_location_stock"/>
        <field name="location_dest_id" ref="stock.stock_location_customers"/>
        <field name="company_id" ref="base.main_company"/>
    </record>
    <record id="Picking_B" model="stock.picking">
        <field name="move_type">one</field>
        <field name="priority">0</field>
        <field name="user_id" eval="False"/>
        <field name="picking_type_id" ref="stock.picking_type_out"/>
        <field name="batch_id" ref="stock_picking_batch_2"/>
        <field name="location_id" ref="stock.stock_location_stock"/>
        <field name="location_dest_id" ref="stock.stock_location_customers"/>
        <field name="company_id" ref="base.main_company"/>
    </record>
    <record id="Picking_C" model="stock.picking">
        <field name="move_type">one</field>
        <field name="priority">0</field>
        <field name="user_id" eval="False"/>
        <field name="picking_type_id" ref="stock.picking_type_out"/>
        <field name="batch_id" ref="stock_picking_batch_1"/>
        <field name="location_id" ref="stock.stock_location_stock"/>
        <field name="location_dest_id" ref="stock.stock_location_customers"/>
        <field name="company_id" ref="base.main_company"/>
    </record>
    <record id="Picking_D" model="stock.picking">
        <field name="move_type">one</field>
        <field name="priority">1</field>
        <field name="user_id" eval="False"/>
        <field name="picking_type_id" ref="stock.picking_type_out"/>
        <field name="batch_id" ref="stock_picking_batch_1"/>
        <field name="location_id" ref="stock.stock_location_stock"/>
        <field name="location_dest_id" ref="stock.stock_location_customers"/>
        <field name="company_id" ref="base.main_company"/>
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

## File: models\stock_picking.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class StockPicking(models.Model):
    _inherit = "stock.picking"

    batch_id = fields.Many2one(
        'stock.picking.batch', string='Batch Transfer',
        check_company=True,
        states={'done': [('readonly', True)], 'cancel': [('readonly', True)]},
        help='Batch associated to this transfer', copy=False)

    @api.model
    def create(self, vals):
        res = super().create(vals)
        if vals.get('batch_id'):
            res.batch_id._sanity_check()
        return res

    def write(self, vals):
        res = super().write(vals)
        if vals.get('batch_id'):
            if not self.batch_id.picking_type_id:
                self.batch_id.picking_type_id = self.picking_type_id[0]
            self.batch_id._sanity_check()
        return res

    def _should_show_transfers(self):
        if len(self.batch_id) == 1 and self == self.batch_id.picking_ids:
            return False
        return super()._should_show_transfers()

```

## File: models\stock_picking_batch.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError
from odoo.tools.float_utils import float_compare, float_is_zero, float_round

class StockPickingBatch(models.Model):
    _inherit = ['mail.thread', 'mail.activity.mixin']
    _name = "stock.picking.batch"
    _description = "Batch Transfer"
    _order = "name desc"

    name = fields.Char(
        string='Batch Transfer', default='New',
        copy=False, required=True, readonly=True,
        help='Name of the batch transfer')
    user_id = fields.Many2one(
        'res.users', string='Responsible', tracking=True, check_company=True,
        readonly=True, states={'draft': [('readonly', False)], 'in_progress': [('readonly', False)]},
        help='Person responsible for this batch transfer')
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
        help='Technical field used to compute whether the check availability button should be shown.')
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
        copy=False, tracking=True, required=True, readonly=True)
    picking_type_id = fields.Many2one(
        'stock.picking.type', 'Operation Type', check_company=True, copy=False,
        readonly=True, states={'draft': [('readonly', False)]})
    scheduled_date = fields.Datetime(
        'Scheduled Date', copy=False, store=True, readonly=False, compute="_compute_scheduled_date",
        states={'done': [('readonly', True)], 'cancel': [('readonly', True)]},
        help="""Scheduled date for the transfers to be processed.
              - If manually set then scheduled date for all transfers in batch will automatically update to this date.
              - If not manually changed and transfers are added/removed/updated then this will be their earliest scheduled date
                but this scheduled date will not be set for all transfers in batch.""")

    @api.depends('company_id', 'picking_type_id', 'state')
    def _compute_allowed_picking_ids(self):
        allowed_picking_states = ['waiting', 'confirmed', 'assigned']
        cancelled_batchs = self.env['stock.picking.batch'].search_read(
            [('state', '=', 'cancel')], ['id']
        )
        cancelled_batch_ids = [batch['id'] for batch in cancelled_batchs]

        for batch in self:
            domain_states = list(allowed_picking_states)
            # Allows to add draft pickings only if batch is in draft as well.
            if batch.state == 'draft':
                domain_states.append('draft')
            domain = [
                ('company_id', '=', batch.company_id.id),
                ('immediate_transfer', '=', False),
                ('state', 'in', domain_states),
                '|',
                '|',
                ('batch_id', '=', False),
                ('batch_id', '=', batch.id),
                ('batch_id', 'in', cancelled_batch_ids),
            ]
            if batch.picking_type_id:
                domain += [('picking_type_id', '=', batch.picking_type_id.id)]
            batch.allowed_picking_ids = self.env['stock.picking'].search(domain)

    @api.depends('picking_ids', 'picking_ids.move_line_ids', 'picking_ids.move_lines', 'picking_ids.move_lines.state')
    def _compute_move_ids(self):
        for batch in self:
            batch.move_ids = batch.picking_ids.move_lines
            batch.move_line_ids = batch.picking_ids.move_line_ids
            batch.show_check_availability = any(m.state not in ['assigned', 'done'] for m in batch.move_ids)

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
    @api.model
    def create(self, vals):
        if vals.get('name', '/') == '/':
            vals['name'] = self.env['ir.sequence'].next_by_code('picking.batch') or '/'
        return super().create(vals)

    def write(self, vals):
        res = super().write(vals)
        if vals.get('picking_type_id'):
            self._sanity_check()
        if vals.get('picking_ids'):
            batch_without_picking_type = self.filtered(lambda batch: not batch.picking_type_id)
            if batch_without_picking_type:
                picking = self.picking_ids and self.picking_ids[0]
                batch_without_picking_type.picking_type_id = picking.picking_type_id.id
        return res

    def unlink(self):
        if any(batch.state != 'draft' for batch in self):
            raise UserError(_("You can only delete draft batch transfers."))
        return super().unlink()

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
        self.ensure_one()
        self.state = 'cancel'
        return True

    def action_print(self):
        self.ensure_one()
        return self.env.ref('stock_picking_batch.action_report_picking_batch').report_action(self)

    def action_done(self):
        self.ensure_one()
        self._check_company()
        pickings = self.mapped('picking_ids').filtered(lambda picking: picking.state not in ('cancel', 'done'))
        if any(picking.state not in ('assigned', 'confirmed') for picking in pickings):
            raise UserError(_('Some transfers are still waiting for goods. Please check or force their availability before setting this batch to done.'))

        empty_pickings = set()
        for picking in pickings:
            if all(float_is_zero(line.qty_done, precision_rounding=line.product_uom_id.rounding) for line in picking.move_line_ids if line.state not in ('done', 'cancel')):
                empty_pickings.add(picking.id)
            picking.message_post(
                body="<b>%s:</b> %s <a href=#id=%s&view_type=form&model=stock.picking.batch>%s</a>" % (
                    _("Transferred by"),
                    _("Batch Transfer"),
                    picking.batch_id.id,
                    picking.batch_id.name))

        if len(empty_pickings) == len(pickings):
            return pickings.button_validate()
        else:
            res = pickings.with_context(skip_immediate=True).button_validate()
            if empty_pickings and res.get('context'):
                res['context']['pickings_to_detach'] = list(empty_pickings)
            return res

    def action_assign(self):
        self.ensure_one()
        self.picking_ids.action_assign()

    def action_put_in_pack(self):
        """ Action to put move lines with 'Done' quantities into a new pack
        This method follows same logic to stock.picking.
        """
        self.ensure_one()
        if self.state not in ('done', 'cancel'):
            picking_move_lines = self.move_line_ids

            move_line_ids = picking_move_lines.filtered(lambda ml:
                float_compare(ml.qty_done, 0.0, precision_rounding=ml.product_uom_id.rounding) > 0
                and not ml.result_package_id
            )
            if not move_line_ids:
                move_line_ids = picking_move_lines.filtered(lambda ml: float_compare(ml.product_uom_qty, 0.0,
                                     precision_rounding=ml.product_uom_id.rounding) > 0 and float_compare(ml.qty_done, 0.0,
                                     precision_rounding=ml.product_uom_id.rounding) == 0)
            if move_line_ids:
                res = move_line_ids.picking_id[0]._pre_put_in_pack_hook(move_line_ids)
                if not res:
                    res = move_line_ids.picking_id[0]._put_in_pack(move_line_ids, False)
                return res
            else:
                raise UserError(_("Please add 'Done' quantities to the batch picking to create a new pack."))

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
                    "transfers or if they're not already part of another batch transfer.\n\n"
                    "Incompatibilities: %s", batch.name, ', '.join(erroneous_pickings.mapped('name'))))

    def _track_subtype(self, init_values):
        if 'state' in init_values:
            return self.env.ref('stock_picking_batch.mt_batch_state')
        return super()._track_subtype(init_values)


```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import stock_picking
from . import stock_picking_batch

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
                                <div class="mr-auto">
                                    <img alt="Barcode" t-att-src="'/report/barcode/?type=%s&amp;value=%s&amp;width=%s&amp;height=%s' % ('Code128', quote_plus(o.name or ''), 600, 150)" style="width:300px;height:50px"/>
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
                                            <img t-att-src="'/report/barcode/?type=%s&amp;value=%s&amp;width=%s&amp;height=%s&amp;quiet=%s' % ('Code128', pick.name, 400, 100, 0)" style="width:200px;height:50px" alt="Barcode"/>
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
                            <t t-foreach="locations" t-as="location">
                                <t t-set="loc_move_line" t-value="move_line_ids.filtered(lambda x: x.location_id==location)"/>
                                <t t-set="products" t-value="loc_move_line.mapped('product_id')"/>
                                <h3><span t-field="o.name"/></h3>
                                <div t-if="o.user_id">
                                    <strong>Responsible:</strong>
                                    <span t-field="o.user_id"/>
                                </div><br/>
                                <h4><strong>To take from: <span t-field="location.display_name"/></strong></h4>
                                <table class="table table-condensed">
                                    <thead>
                                        <tr>
                                            <th>Product</th>
                                            <th>Quantity</th>
                                            <th width="27%">To</th>
                                            <th width="23%">Transfer</th>
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
                                    <tbody>
                                        <tr t-foreach="loc_move_line" t-as="move_operation">
                                            <td>
                                                <span t-field="move_operation.display_name"/>
                                            </td>
                                            <td>
                                                <t t-if="not has_package">
                                                    <t t-if="move_operation.state == 'done'">
                                                        <span t-esc="sum(move_operation.mapped('qty_done'))"/>
                                                    </t>
                                                    <t t-else="">
                                                        <span t-esc="sum(move_operation.mapped('product_uom_qty'))"/>
                                                    </t>
                                                </t>
                                                <t t-if="has_package">
                                                    <span t-esc="sum(move_operation.mapped('qty_done'))"/>
                                                </t>
                                                <span t-field="move_operation.uom_id" groups="move_operation.group_uom"/>
                                            </td>
                                            <td>
                                                <span t-esc="move_operation.mapped('location_dest_id').display_name"/>
                                            </td>
                                            <td>
                                                <span t-esc="move_operation.mapped('picking_id').display_name"/>
                                            </td>
                                            <td t-if="has_serial_number" class="text-center h6" width="15%">
                                                <img t-if="move_operation.lot_id or move_operation.lot_name" t-att-src="'/report/barcode/?type=%s&amp;value=%s&amp;width=%s&amp;height=%s&amp;humanreadable=1' % ('Code128', move_operation.lot_id.name, 600, 100)" style="width:100%;height:35px;" alt="Barcode"/>
                                            </td>
                                            <td width="15%" class="text-center" t-if="has_barcode">
                                                <span t-if="move_operation.product_id and move_operation.product_id.barcode">
                                                    <img t-if="len(move_operation.product_id.barcode) == 13" t-att-src="'/report/barcode/?type=%s&amp;value=%s&amp;width=%s&amp;height=%s' % ('EAN13', move_operation.product_id.barcode, 600, 100)" style="width:100%;height:35px" alt="Barcode"/>
                                                    <img t-elif="len(move_operation.product_id.barcode) == 8" t-att-src="'/report/barcode/?type=%s&amp;value=%s&amp;width=%s&amp;height=%s' % ('EAN8', move_operation.product_id.barcode, 600, 100)" style="width:100%;height:35px" alt="Barcode"/>
                                                    <img t-else="" t-att-src="'/report/barcode/?type=%s&amp;value=%s&amp;width=%s&amp;height=%s' % ('Code128', move_operation.product_id.barcode, 600, 100)" style="width:100%;height:35px" alt="Barcode"/>

                                                </span>
                                            </td>
                                            <td t-if="has_package" width="15%">
                                                <span t-field="move_operation.package_id"/>
                                                <t t-if="move_operation.result_package_id">
                                                     → <span t-field="move_operation.result_package_id"/>
                                                </t>
                                            </td>
                                        </tr>
                                    </tbody>
                                </table>
                                <p style="page-break-after: always;"/>
                            </t>
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
                <field name="product_id" context="{'default_type': 'product'}" required="1" attrs="{'readonly': [('id', '!=', False)]}"/>
                <field name="picking_id" required="1" attrs="{'readonly': [('id', '!=', False)]}"
                    options="{'no_create_edit': True}" domain="[('id', 'in', parent.picking_ids)]"/>
                <field name="lot_id" groups="stock.group_production_lot" attrs="{'readonly': [('tracking', 'not in', ['lot', 'serial'])]}"/>
                <field name="location_id"/>
                <field name="location_dest_id"/>
                <field name="package_id" groups="stock.group_tracking_lot"/>
                <field name="result_package_id" groups="stock.group_tracking_lot"/>
                <field name="product_uom_qty" readonly="1"/>
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
                <field name="show_check_availability" invisible="1"/>
                <header>
                    <button name="action_confirm" states="draft" string="Confirm" type="object" class="oe_highlight"/>
                    <button name="action_done" string="Validate" type="object" class="oe_highlight"
                        attrs="{'invisible': [
                            '|',
                                ('state', '!=', 'in_progress'),
                                ('show_check_availability', '=', True)]}"/>
                    <button name="action_assign" string="Check Availability" type="object" class="oe_highlight"
                        attrs="{'invisible': [
                            '|',
                                ('state', '!=', 'in_progress'),
                                ('show_check_availability', '=', False)]}"/>
                    <button name="action_done" string="Validate" type="object"
                        attrs="{'invisible': [
                            '|',
                                ('state', '!=', 'in_progress'),
                                ('show_check_availability', '=', False)]}"/>
                    <button name="action_assign" string="Check Availability" type="object"
                        attrs="{'invisible': [
                            '|',
                                ('state', '!=', 'draft'),
                                ('show_check_availability', '=', False)]}"/>
                    <button name="action_print" states="in_progress,done" string="Print" type="object"/>
                    <button name="action_cancel" string="Cancel" type="object" states="in_progress"/>
                    <field name="state" widget="statusbar" statusbar_visible="draft,in_progress,done"/>
                </header>
                <sheet>
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
                                context="{'form_view_ref': 'stock_picking_batch.view_picking_form_inherited'}">
                                <tree>
                                    <field name="name"/>
                                    <field name="scheduled_date"/>
                                    <field name="location_id"/>
                                    <field name="backorder_id"/>
                                    <field name="origin"/>
                                    <field name="state"/>
                                </tree>
                            </field>
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
                <field name="name" decoration-bf="1"/>
                <field name="scheduled_date"/>
                <field name="user_id" widget="many2one_avatar_user"/>
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
                <field name="user_id"/>
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
        <field name="context">{
            "search_default_draft" : True,
            "search_default_in_progress" : True,
            'default_company_id': allowed_company_ids[0]
        }</field>
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
    <menuitem action="stock_picking_batch_action" id="stock_picking_batch_menu" parent="stock.menu_stock_warehouse_mgmt" sequence="10"/>

    <record id="vpicktree_inherit_stock_picking_batch" model="ir.ui.view">
        <field name="name">stock.picking.tree</field>
        <field name="model">stock.picking</field>
        <field name="inherit_id" ref="stock.vpicktree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='company_id']" position="before">
                <field name="batch_id" optional="show"
                    domain="[
                        ('state', 'in', ['draft', 'in_progress']),
                        '|',
                            ('picking_type_id', '=', picking_type_id),
                            ('picking_type_id', '=', False),
                    ]"
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

</odoo>

```

## File: wizard\stock_backorder_confirmation.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class StockBackorderConfirmation(models.TransientModel):
    _inherit = 'stock.backorder.confirmation'

    def process(self):
        res = super().process()
        if self.env.context.get('pickings_to_detach'):
            self.env['stock.picking'].browse(self.env.context['pickings_to_detach']).batch_id = False
        return res

    def process_cancel_backorder(self):
        res = super().process_cancel_backorder()
        if self.env.context.get('pickings_to_detach'):
            self.env['stock.picking'].browse(self.env.context['pickings_to_detach']).batch_id = False
        return res

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

    batch_id = fields.Many2one('stock.picking.batch', string='Batch Transfer', domain="[('state', '=', 'draft')]")
    mode = fields.Selection([('existing', 'an existing batch transfer'), ('new', 'a new batch transfer')], default='existing')
    user_id = fields.Many2one('res.users', string='Responsible', help='Person responsible for this batch transfer')

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
                        <field name="batch_id" options="{'no_create_edit': True, 'no_open': True}" attrs="{'invisible': [('mode', '=', 'new')], 'required': [('mode', '=', 'existing')]}"/>
                        <field name="user_id" options="{'no_create_edit': True}" attrs="{'invisible': [('mode', '=', 'existing')]}"/>
                    </group>
                </group>

                <footer>
                    <button name="attach_pickings" type="object" string="Confirm" class="btn-primary"/>
                    <button string="Cancel" class="btn-secondary" special="cancel"/>
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
from . import stock_package_destination
from . import stock_backorder_confirmation

```

