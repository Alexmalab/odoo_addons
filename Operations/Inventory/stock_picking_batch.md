# Odoo Module: stock_picking_batch

Category: Operations/Inventory

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
    'category': 'Operations/Inventory',
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
<odoo>

    <!-- Add batch picking -->
    <record id="stock_picking_batch_dry_1" model="stock.picking.batch">
        <field name="state">in_progress</field>
        <field name="company_id" ref="base.main_company"/>
    </record>
    <record id="stock_picking_batch_freeze_1" model="stock.picking.batch">
        <field name="state">in_progress</field>
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
        <field name="priority">2</field>
        <field name="picking_type_id" ref="stock.picking_type_internal"/>
        <field name="batch_id" ref="stock_picking_batch_freeze_1"/>
        <field name="location_id" ref="stock.stock_location_stock"/>
        <field name="location_dest_id" ref="stock.stock_location_output"/>
        <field name="company_id" ref="base.main_company"/>
    </record>
    <record id="Picking_B" model="stock.picking">
        <field name="move_type">one</field>
        <field name="state">assigned</field>
        <field name="priority">1</field>
        <field name="picking_type_id" ref="stock.picking_type_internal"/>
        <field name="batch_id" ref="stock_picking_batch_freeze_1"/>
        <field name="location_id" ref="stock.stock_location_stock"/>
        <field name="location_dest_id" ref="stock.stock_location_output"/>
        <field name="company_id" ref="base.main_company"/>
    </record>
    <record id="Picking_C" model="stock.picking">
        <field name="move_type">one</field>
        <field name="state">assigned</field>
        <field name="priority">1</field>
        <field name="picking_type_id" ref="stock.picking_type_internal"/>
        <field name="batch_id" ref="stock_picking_batch_dry_1"/>
        <field name="location_id" ref="stock.stock_location_stock"/>
        <field name="location_dest_id" ref="stock.stock_location_output"/>
        <field name="company_id" ref="base.main_company"/>
    </record>

    <!-- Add stock move -->
    <record id="stock_move1" model="stock.move">
        <field name="name">A first stock move</field>
        <field name="picking_type_id" ref="stock.picking_type_internal"/>
        <field name="picking_id" ref="Picking_A"/>
        <field name="location_id" ref="stock.stock_location_stock"/>
        <field name="location_dest_id" ref="stock.stock_location_output"/>
        <field name="product_uom_qty">10</field>
        <field name="product_uom" ref="uom.product_uom_unit" />
        <field name="product_id" ref="product.consu_delivery_01"/>
    </record>
    <record id="stock_move2" model="stock.move">
        <field name="name">A second stock move</field>
        <field name="picking_type_id" ref="stock.picking_type_internal"/>
        <field name="picking_id" ref="Picking_A"/>
        <field name="location_id" ref="stock.stock_location_stock"/>
        <field name="location_dest_id" ref="stock.stock_location_output"/>
        <field name="product_uom_qty">10</field>
        <field name="product_uom" ref="uom.product_uom_unit" />
        <field name="product_id" ref="product.consu_delivery_02"/>
    </record>
    <record id="stock_move3" model="stock.move">
        <field name="name">A third stock move</field>
        <field name="picking_type_id" ref="stock.picking_type_internal"/>
        <field name="picking_id" ref="Picking_B"/>
        <field name="location_id" ref="stock.stock_location_stock"/>
        <field name="location_dest_id" ref="stock.stock_location_output"/>
        <field name="product_uom_qty">10</field>
        <field name="product_uom" ref="uom.product_uom_unit" />
        <field name="product_id" ref="product.consu_delivery_03"/>
    </record>
    <record id="stock_move4" model="stock.move">
        <field name="name">A fourth stock move</field>
        <field name="picking_type_id" ref="stock.picking_type_internal"/>
        <field name="picking_id" ref="Picking_C"/>
        <field name="location_id" ref="stock.stock_location_stock"/>
        <field name="location_dest_id" ref="stock.stock_location_output"/>
        <field name="product_uom_qty">10</field>
        <field name="product_uom" ref="uom.product_uom_unit" />
        <field name="product_id" ref="product.consu_delivery_03"/>
    </record>

    <!-- Confirm Pickings -->
    <function model="stock.picking" name="action_confirm">
        <value eval="[
            ref('stock_picking_batch.Picking_A'),
            ref('stock_picking_batch.Picking_B'),
            ref('stock_picking_batch.Picking_C')]"/>
    </function>

    <!-- Check Availability Pickings -->
    <function model="stock.picking" name="action_assign">
        <value eval="[
            ref('stock_picking_batch.Picking_A'),
            ref('stock_picking_batch.Picking_B'),
            ref('stock_picking_batch.Picking_C')]"/>
    </function>
</odoo>

```

## File: models\stock_picking_batch.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError


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
        help='Person responsible for this batch transfer')
    company_id = fields.Many2one(
        'res.company', string="Company", required=True, readonly=True,
        index=True, default=lambda self: self.env.company)
    picking_ids = fields.One2many(
        'stock.picking', 'batch_id', string='Transfers',
        domain="[('company_id', '=', company_id), ('state', 'not in', ('done', 'cancel'))]",
        help='List of transfers associated to this batch')
    state = fields.Selection([
        ('draft', 'Draft'),
        ('in_progress', 'In progress'),
        ('done', 'Done'),
        ('cancel', 'Cancelled')], default='draft',
        copy=False, tracking=True, required=True, readonly=True)

    @api.model
    def create(self, vals):
        if vals.get('name', '/') == '/':
            vals['name'] = self.env['ir.sequence'].next_by_code('picking.batch') or '/'
        return super(StockPickingBatch, self).create(vals)

    def confirm_picking(self):
        self._check_company()
        pickings_todo = self.mapped('picking_ids')
        self.write({'state': 'in_progress'})
        return pickings_todo.action_assign()

    def cancel_picking(self):
        self.mapped('picking_ids').action_cancel()
        return self.write({'state': 'cancel'})

    def print_picking(self):
        pickings = self.mapped('picking_ids')
        if not pickings:
            raise UserError(_('Nothing to print.'))
        return self.env.ref('stock_picking_batch.action_report_picking_batch').report_action(self)

    def done(self):
        self._check_company()
        pickings = self.mapped('picking_ids').filtered(lambda picking: picking.state not in ('cancel', 'done'))
        if any(picking.state not in ('assigned') for picking in pickings):
            raise UserError(_('Some transfers are still waiting for goods. Please check or force their availability before setting this batch to done.'))
        for picking in pickings:
            picking.message_post(
                body="<b>%s:</b> %s <a href=#id=%s&view_type=form&model=stock.picking.batch>%s</a>" % (
                    _("Transferred by"),
                    _("Batch Transfer"),
                    picking.batch_id.id,
                    picking.batch_id.name))

        picking_to_backorder = self.env['stock.picking']
        picking_without_qty_done = self.env['stock.picking']
        for picking in pickings:
            if all([x.qty_done == 0.0 for x in picking.move_line_ids]):
                # If no lots when needed, raise error
                picking_type = picking.picking_type_id
                if (picking_type.use_create_lots or picking_type.use_existing_lots):
                    for ml in picking.move_line_ids:
                        if ml.product_id.tracking != 'none' and not ml.lot_id and not ml.lot_name:
                            raise UserError(_('Some products require lots/serial numbers.'))
                # Check if we need to set some qty done.
                picking_without_qty_done |= picking
            elif picking._check_backorder():
                picking_to_backorder |= picking
            else:
                picking.action_done()
        if pickings and len(picking_without_qty_done) == len(pickings):
            view = self.env.ref('stock.view_immediate_transfer')
            wiz = self.env['stock.immediate.transfer'].create({
                'pick_ids': [(4, p.id) for p in picking_without_qty_done],
                'pick_to_backorder_ids': [(4, p.id) for p in picking_to_backorder],
            })
            return {
                'name': _('Immediate Transfer?'),
                'type': 'ir.actions.act_window',
                'view_mode': 'form',
                'res_model': 'stock.immediate.transfer',
                'views': [(view.id, 'form')],
                'view_id': view.id,
                'target': 'new',
                'res_id': wiz.id,
                'context': self.env.context,
            }
        if picking_to_backorder or picking_without_qty_done:
            res = picking_to_backorder.action_generate_backorder_wizard()
            if picking_without_qty_done and 'context' in res:
                res['context']['pickings_to_detach'] = picking_without_qty_done.ids
            return res

        # Change the state only if there is no other action (= wizard) waiting.
        self.write({'state': 'done'})
        return True

    def _track_subtype(self, init_values):
        if 'state' in init_values:
            return self.env.ref('stock_picking_batch.mt_batch_state')
        return super(StockPickingBatch, self)._track_subtype(init_values)


class StockPicking(models.Model):
    _inherit = "stock.picking"

    batch_id = fields.Many2one(
        'stock.picking.batch', string='Batch Transfer',
        check_company=True,
        states={'done': [('readonly', True)], 'cancel': [('readonly', True)]},
        help='Batch associated to this transfer', copy=False)


```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

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
                    <t t-set="locations" t-value="move_line_ids.mapped('location_id')"/>
                    <t t-call="web.external_layout">
                        <div class="page">
                            <h3>Summary: <span t-field="o.name"/></h3>
                            <div t-if="o.user_id">
                                <strong>Responsible:</strong>
                                <span t-field="o.user_id"/>
                            </div><br/>
                            <table class="table table-condensed">
                                <thead>
                                    <tr>
                                        <th>Picking Reference</th>
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
                                            <th width="23%">Picking</th>
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
                                                    <t t-if="any(move_operation.filtered(lambda l: l.state == 'done'))">
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
                                            <td t-if="has_serial_number and (move_operation.lot_id or move_operation.lot_name)" class="text-center h6" width="15%">
                                                <img t-att-src="'/report/barcode/?type=%s&amp;value=%s&amp;width=%s&amp;height=%s&amp;humanreadable=1' % ('Code128', move_operation.lot_id.name, 600, 100)" style="width:100%;height:35px;" alt="Barcode"/>
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
        <report
            string="Batch Transfer"
            id="action_report_picking_batch"
            model="stock.picking.batch"
            report_type="qweb-pdf"
            menu="False"
            name="stock_picking_batch.report_picking_batch"
            file="stock_picking_batch.report_picking_batch"
        />
    </data>
</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_stock_picking_batch,stock.picking.batch stock users,model_stock_picking_batch,stock.group_stock_user,1,1,1,1

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

    <record id="stock_picking_batch_form" model="ir.ui.view">
        <field name="name">stock.picking.batch.form</field>
        <field name="model">stock.picking.batch</field>
        <field name="arch" type="xml">
            <form string="Stock Batch Transfer">
                <header>
                    <button name="confirm_picking" states="draft" string="Confirm" type="object" class="oe_highlight"/>
                    <button name="print_picking" string="Print" type="object" class="oe_highlight"/>
                    <button name="done" states="in_progress" string="Done" type="object" class="oe_highlight"/>
                    <button name="cancel_picking" string="Cancel" type="object" states="in_progress"/>
                    <field name="state" widget="statusbar" statusbar_visible="in_progress,done"/>
                </header>
                <sheet>
                    <div class="oe_title">
                        <h1><field name="name" class="oe_inline"/></h1>
                    </div>
                    <group>
                        <group>
                            <field name="user_id"/>
                            <field name="company_id" groups="base.group_multi_company" attrs="{'readonly': [('picking_ids', '!=', [])]}" force_save="1"/>
                        </group>
                    </group>
                    <separator string="Transfers"/>
                    <field name="picking_ids" widget="many2many" options="{'not_delete': True}" mode="tree,kanban">
                        <tree>
                            <field name="name"/>
                            <field name="scheduled_date"/>
                            <field name="location_id"/>
                            <field name="backorder_id"/>
                            <field name="origin"/>
                            <field name="state"/>
                            <button name="action_assign" string="Confirm picking" type="object" icon="fa-check text-success" attrs="{'invisible': [('state', 'in', ('done', 'cancel', 'confirmed', 'assigned'))]}"/>
                            <button name="action_cancel" string="Cancel picking" type="object" icon="fa-times-circle text-danger" attrs="{'invisible': [('state', 'in', ('done', 'cancel'))]}"/>
                        </tree>
                    </field>
                </sheet>
                <div class="oe_chatter">
                    <field name="message_follower_ids" widget="mail_followers" groups="base.group_user"/>
                    <field name="activity_ids" widget="mail_activity"/>
                    <field name="message_ids" widget="mail_thread"/>
                </div>
            </form>
        </field>
    </record>
    <record id="stock_picking_batch_tree" model="ir.ui.view">
        <field name="name">stock.picking.batch.tree</field>
        <field name="model">stock.picking.batch</field>
        <field name="arch" type="xml">
            <tree string="Stock Batch Transfer" decoration-muted="state=='cancel'" multi_edit="1">
                <field name="name"/>
                <field name="user_id"/>
                <field name="state"/>
                <field name="company_id" groups="base.group_multi_company"/>
                <field name="activity_exception_decoration" widget="activity_exception"/>
            </tree>
        </field>
    </record>
    <record id="stock_picking_batch_kanban" model="ir.ui.view">
        <field name="name">stock.picking.batch.kanban</field>
        <field name="model">stock.picking.batch</field>
        <field name="arch" type="xml">
            <kanban class="o_kanban_mobile">
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
                                <div class="oe_kanban_bottom_left"/>
                                <div class="oe_kanban_bottom_right">
                                    <img t-att-src="kanban_image('res.users', 'image_128', record.user_id.raw_value)" t-att-title="record.user_id.value" t-att-alt="record.user_id.value" class="oe_kanban_avatar"/>
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
                <filter name="in_progress" string="Running" domain="[('state', '=', 'in_progress')]" help="Batch Transfers not finished"/>
                <filter name="done" string="Done" domain="[('state', '=', 'done')]"/>
                <separator/>
                <filter invisible="1" string="Late Activities" name="activities_overdue"
                    domain="[('activity_ids.date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                    help="Show all records which has next action date is before today"/>
                <filter invisible="1" string="Today Activities" name="activities_today"
                    domain="[('activity_ids.date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                    domain="[('activity_ids.date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
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
        <field name="context">{"search_default_in_progress" : True, 'default_company_id': allowed_company_ids[0]}</field>
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
            <xpath expr="//field[@name='state']" position="before">
                <field name="batch_id" optional="show"/>
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

## File: wizard\stock_immediate_transfer.py

```python
from odoo import models, fields, api


class StockImmediateTransfer(models.TransientModel):
    _inherit = 'stock.immediate.transfer'
    _description = 'Immediate Transfer'

    pick_to_backorder_ids = fields.Many2many('stock.picking', help='Picking to backorder')

    def process(self):
        backorder_wizard_dict = super(StockImmediateTransfer, self).process()
        # If the immediate transfer wizard process all our picking but with some back order maybe needed we want to add the backorder already passed to the wizard.
        if backorder_wizard_dict:
            backorder_wizard = self.env['stock.backorder.confirmation'].browse(backorder_wizard_dict.get('res_id', False))
            backorder_wizard.write({'pick_ids': [(4, p.id) for p in self.pick_to_backorder_ids]})
            return backorder_wizard_dict
        # If there is no backorder returned by the immediate transfer basic function we still wanted to process those manually given
        elif self.pick_to_backorder_ids:
            return self.pick_to_backorder_ids.action_generate_backorder_wizard()
        return False

```

## File: wizard\stock_picking_to_batch.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class StockPickingToBatch(models.TransientModel):
    _name = 'stock.picking.to.batch'
    _description = 'Batch Transfer Lines'

    batch_id = fields.Many2one('stock.picking.batch', string='Batch Transfer')

    def attach_pickings(self):
        # use active_ids to add picking line to the selected batch
        self.ensure_one()
        picking_ids = self.env.context.get('active_ids')
        return self.env['stock.picking'].browse(picking_ids).write({'batch_id': self.batch_id.id})

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
            <form string="Add pickings to batch">
                <separator string="Select a batch"/>
                <field name="batch_id" class="oe_inline" required="1" options="{'no_create_edit': True, 'no_open': True}"/>
                <footer>
                    <button name="attach_pickings" type="object" string="Add to Batch" class="btn-primary"/>
                    <button string="Cancel" class="btn-secondary" special="cancel"/>
                </footer>
            </form>
        </field>
    </record>

    <!--  add picking to batch action -->
    <record id="stock_picking_to_batch_action" model="ir.actions.act_window">
        <field name="name">Add to Batch</field>
        <field name="res_model">stock.picking.to.batch</field>
        <field name="view_mode">form</field>
        <field name="view_id" ref="stock_picking_to_batch_form"/>
        <field name="target">new</field>
    </record>

    <act_window id="stock_picking_to_batch_action_stock_picking"
        name="Add to batch"
        res_model="stock.picking.to.batch"
        binding_model="stock.picking"
        binding_views="list"
        view_mode="form" target="new"
    />

</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import stock_picking_to_batch
from . import stock_immediate_transfer
from . import stock_backorder_confirmation

```

