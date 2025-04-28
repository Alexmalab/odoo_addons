# Odoo Module: repair

Category: Inventory/Inventory

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import wizard
from . import report

from odoo import api, SUPERUSER_ID

def _create_warehouse_data(env):
    """ This hook is used to add default repair picking types on every warehouse.
    It is necessary if the repair module is installed after some warehouses were already created.
    """
    warehouses = env['stock.warehouse'].search([('repair_type_id', '=', False)])
    for warehouse in warehouses:
        picking_type_vals = warehouse._create_or_update_sequences_and_picking_types()
        if picking_type_vals:
            warehouse.write(picking_type_vals)

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


{
    'name': 'Repairs',
    'version': '1.0',
    'sequence': 230,
    'category': 'Inventory/Inventory',
    'summary': 'Repair damaged products',
    'description': """
The aim is to have a complete module to manage all products repairs.
====================================================================

The following topics are covered by this module:
------------------------------------------------------
    * Add/remove products in the reparation
    * Impact for stocks
    * Warranty concept
    * Repair quotation report
    * Notes for the technician and for the final customer
""",
    'depends': ['stock', 'sale_management'],
    'data': [
        'security/ir.model.access.csv',
        'security/repair_security.xml',
        'wizard/stock_warn_insufficient_qty_views.xml',
        'wizard/repair_warn_uncomplete_move.xml',
        'views/product_views.xml',
        'views/stock_move_views.xml',
        'views/repair_views.xml',
        'views/sale_order_views.xml',
        'views/stock_lot_views.xml',
        'views/stock_picking_views.xml',
        'views/stock_warehouse_views.xml',
        'report/repair_reports.xml',
        'report/repair_templates_repair_order.xml',
        'data/repair_data.xml',
    ],
    'demo': ['data/repair_demo.xml'],
    'post_init_hook': '_create_warehouse_data',
    'installable': True,
    'application': True,
    'license': 'LGPL-3',
}

```

## File: data\repair_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <!--
    Given that data are loaded before update of warehouses through the post_init_hook,
    we manually create the 'Repairs' Operation Type in warehouse0 here such that it's available
    when the initialization of data triggers the default method of the required field 'picking_type_id'.
    -->

    <record id="picking_type_warehouse0_repair" model="stock.picking.type" forcecreate="0">
        <field name="name">Repairs</field>
        <field name="code">repair_operation</field>
        <field name="company_id" ref="base.main_company"/>
        <field name="default_location_src_id" ref="stock.stock_location_stock"/>
        <field name="default_location_dest_id" model="stock.location" search="[('usage', '=', 'production'), ('company_id', '=', obj().env.ref('base.main_company').id)]"/>
        <field name="default_remove_location_dest_id" model="stock.location" search="[('scrap_location', '=', True), ('company_id', '=', obj().env.ref('base.main_company').id)]"/>
        <field name="default_recycle_location_dest_id" ref="stock.stock_location_stock"/>
        <field name="sequence_code">RO</field>
        <field name="warehouse_id" ref="stock.warehouse0"/>
    </record>

    <record id='stock.warehouse0' model='stock.warehouse'>
        <field name='repair_type_id' ref='repair.picking_type_warehouse0_repair'/>
    </record>
</odoo>

```

## File: data\repair_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="product_service_order_repair" model="product.product">
        <field name="name">Repair Services</field>
        <field name="categ_id" ref="product.product_category_3"/>
        <field name="standard_price">20.5</field>
        <field name="list_price">30.75</field>
        <field name="type">service</field>
        <field name="create_repair">True</field>
    </record>

    <record id="repair_r1" model="repair.order">
        <field name="schedule_date" eval="DateTime.today()"/>
        <field name="user_id"/>
        <field name="product_id" ref="product.product_product_3"/>
        <field name="product_uom" ref="uom.product_uom_unit"/>
        <field name="picking_type_id" ref="picking_type_warehouse0_repair"/>
        <field name="move_ids" model="stock.move" eval="[(5, 0, 0), (0, 0, {
            'location_dest_id': obj().env.ref('product.product_product_11').property_stock_production.id,
            'location_id': obj().env.ref('stock.stock_location_stock').id,
            'name': obj().env.ref('product.product_product_11').get_product_multiline_description_sale(),
            'product_id': obj().env.ref('product.product_product_11').id,
            'product_uom': obj().env.ref('uom.product_uom_unit').id,
            'product_uom_qty': '1.0',
            'state': 'draft',
            'repair_line_type': 'add',
            'company_id': obj().env.ref('base.main_company').id,
        })]"/>
        <field name="partner_id" ref="base.res_partner_12"/>
    </record>

    <record id="repair_r0" model="repair.order">
        <field name="schedule_date" eval="DateTime.today()"/>
        <field name="product_id" ref="product.product_product_5"/>
        <field name="product_uom" ref="uom.product_uom_unit"/>
        <field name="user_id"/>
        <field name="picking_type_id" ref="picking_type_warehouse0_repair"/>
        <field name="move_ids" model="stock.move" eval="[(5, 0, 0), (0, 0, {
            'location_dest_id': obj().env.ref('product.product_product_12').property_stock_production.id,
            'location_id': obj().env.ref('stock.stock_location_stock').id,
            'name': obj().env.ref('product.product_product_12').get_product_multiline_description_sale(),
            'product_id': obj().env.ref('product.product_product_12').id,
            'product_uom': obj().env.ref('uom.product_uom_unit').id,
            'product_uom_qty': 1.0,
            'state': 'draft',
            'repair_line_type': 'add',
            'company_id': obj().env.ref('base.main_company').id,
        })]"/>
        <field name="partner_id" ref="base.res_partner_12"/>
    </record>

    <record id="repair_r2" model="repair.order">
        <field name="priority">1</field>
        <field name="schedule_date" eval="DateTime.today() + relativedelta(days=-5)"/>
        <field name="product_id" ref="product.product_product_6"/>
        <field name="product_uom" ref="uom.product_uom_unit"/>
        <field name="user_id"/>
        <field name="picking_type_id" ref="picking_type_warehouse0_repair"/>
        <field name="move_ids" model="stock.move" eval="[(5, 0, 0), (0, 0, {
            'location_dest_id': obj().env.ref('product.product_product_13').property_stock_production.id,
            'location_id': obj().env.ref('stock.stock_location_stock').id,
            'name': obj().env.ref('product.product_product_13').get_product_multiline_description_sale(),
            'product_id': obj().env.ref('product.product_product_13').id,
            'product_uom': obj().env.ref('uom.product_uom_unit').id,
            'product_uom_qty': 1.0,
            'state': 'draft',
            'repair_line_type': 'add',
            'company_id': obj().env.ref('base.main_company').id,
        })]"/>
        <field name="partner_id" ref="base.res_partner_12"/>
    </record>
</odoo>

```

## File: models\mail_compose_message.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class MailComposeMessage(models.TransientModel):
    _inherit = 'mail.compose.message'

    def _action_send_mail(self, auto_commit=False):
        if self.model == 'repair.order':
            self = self.with_context(mail_notify_author=self.env.user.partner_id in self.partner_ids)
        return super(MailComposeMessage, self)._action_send_mail(auto_commit=auto_commit)

```

## File: models\product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class Product(models.Model):
    _inherit = "product.product"

    def _count_returned_sn_products(self, sn_lot):
        remove_count = self.env['stock.move'].search_count([
            ('repair_line_type', 'in', ['remove', 'recycle']),
            ('product_uom_qty', '=', 1),
            ('move_line_ids.lot_id', '=', sn_lot.id),
            ('state', '=', 'done'),
            ('location_dest_usage', '=', 'internal'),
        ])
        add_count = self.env['stock.move'].search_count([
            ('repair_line_type', '=', 'add'),
            ('product_uom_qty', '=', 1),
            ('move_line_ids.lot_id', '=', sn_lot.id),
            ('state', '=', 'done'),
            ('location_dest_usage', '=', 'production'),
        ])
        return super()._count_returned_sn_products(sn_lot) + (remove_count - add_count)


class ProductTemplate(models.Model):
    _inherit = "product.template"

    create_repair = fields.Boolean('Create Repair', help="Create a linked Repair Order on Sale Order confirmation of this product.", groups='stock.group_stock_user')

```

## File: models\repair.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from random import randint

from odoo import api, Command, fields, models, _
from odoo.exceptions import UserError, ValidationError
from odoo.tools import float_compare, float_is_zero, clean_context
from odoo.tools.misc import format_date, groupby

MAP_REPAIR_TO_PICKING_LOCATIONS = {
    'location_id': 'default_location_src_id',
    'location_dest_id': 'default_location_dest_id',
    'parts_location_id': 'default_remove_location_dest_id',
    'recycle_location_id': 'default_recycle_location_dest_id',
}


class Repair(models.Model):
    """ Repair Orders """
    _name = 'repair.order'
    _description = 'Repair Order'
    _inherit = ['mail.thread', 'mail.activity.mixin']
    _order = 'priority desc, create_date desc'
    _check_company_auto = True

    @api.model
    def _default_picking_type_id(self):
        return self._get_picking_type().get((self.env.company, self.env.user))

    # Common Fields
    name = fields.Char(
        'Repair Reference',
        default='New', index='trigram',
        copy=False, required=True,
        readonly=True)
    company_id = fields.Many2one(
        'res.company', 'Company',
        readonly=True, required=True, index=True,
        default=lambda self: self.env.company)
    state = fields.Selection([
        ('draft', 'New'),
        ('confirmed', 'Confirmed'),
        ('under_repair', 'Under Repair'),
        ('done', 'Repaired'),
        ('cancel', 'Cancelled')], string='Status',
        copy=False, default='draft', readonly=True, tracking=True, index=True,
        help="* The \'New\' status is used when a user is encoding a new and unconfirmed repair order.\n"
             "* The \'Confirmed\' status is used when a user confirms the repair order.\n"
             "* The \'Under Repair\' status is used when the repair is ongoing.\n"
             "* The \'Repaired\' status is set when repairing is completed.\n"
             "* The \'Cancelled\' status is used when user cancel repair order.")
    priority = fields.Selection([('0', 'Normal'), ('1', 'Urgent')], default='0', string="Priority")
    partner_id = fields.Many2one(
        'res.partner', 'Customer',
        index=True, check_company=True, change_default=True,
        help='Choose partner for whom the order will be invoiced and delivered. You can find a partner by its Name, TIN, Email or Internal Reference.')
    user_id = fields.Many2one('res.users', string="Responsible", default=lambda self: self.env.user, check_company=True)

    # Specific Fields
    internal_notes = fields.Html('Internal Notes')
    tag_ids = fields.Many2many('repair.tags', string="Tags")
    under_warranty = fields.Boolean(
        'Under Warranty',
        help='If ticked, the sales price will be set to 0 for all products transferred from the repair order.')
    schedule_date = fields.Datetime("Scheduled Date", default=fields.Datetime.now, index=True, required=True, copy=False)

    # Product To Repair
    move_id = fields.Many2one(  # Generated in 'action_repair_done', needed for traceability
        'stock.move', 'Inventory Move',
        copy=False, readonly=True, tracking=True, check_company=True)
    product_id = fields.Many2one(
        'product.product', string='Product to Repair',
        domain="[('type', 'in', ['product', 'consu']), '|', ('company_id', '=', company_id), ('company_id', '=', False), '|', ('id', 'in', picking_product_ids), ('id', '=?', picking_product_id)]",
        check_company=True)
    product_qty = fields.Float(
        'Product Quantity',
        default=1.0, digits='Product Unit of Measure')
    product_uom = fields.Many2one(
        'uom.uom', 'Product Unit of Measure',
        compute='compute_product_uom', store=True, precompute=True,
        domain="[('category_id', '=', product_uom_category_id)]")
    product_uom_category_id = fields.Many2one(related='product_id.uom_id.category_id')
    lot_id = fields.Many2one(
        'stock.lot', 'Lot/Serial',
        compute="compute_lot_id", store=True,
        domain="[('product_id','=', product_id), ('company_id', '=', company_id)]", check_company=True,
        help="Products repaired are all belonging to this lot")
    tracking = fields.Selection(string='Product Tracking', related="product_id.tracking", readonly=False)

    # Picking & Locations
    picking_type_id = fields.Many2one(
        'stock.picking.type', 'Operation Type', copy=True, readonly=False,
        compute='_compute_picking_type_id', store=True,
        default=_default_picking_type_id,
        domain="[('code', '=', 'repair_operation'), ('company_id', '=', company_id)]",
        required=True, precompute=True, check_company=True, index=True)
    procurement_group_id = fields.Many2one(
        'procurement.group', 'Procurement Group',
        copy=False)
    location_id = fields.Many2one(
        'stock.location', 'Location',
        compute="_compute_location_id",
        store=True, readonly=False, required=True, precompute=True,
        index=True, check_company=True,
        help="This is the location where the product to repair is located.")
    location_dest_id = fields.Many2one(
        'stock.location', 'Added Parts Destination Location',
        related="picking_type_id.default_location_dest_id", depends=["picking_type_id"],
        store=True, readonly=True, required=True, precompute=True,
        index=True, check_company=True,
        help="This is the location where the repaired product is located.")
    parts_location_id = fields.Many2one(
        'stock.location', 'Removed Parts Destination Location',
        related="picking_type_id.default_remove_location_dest_id", depends=["picking_type_id"],
        store=True, readonly=True, required=True, precompute=True,
        index=True, check_company=True,
        help="This is the location where the repair parts are located.")
    recycle_location_id = fields.Many2one(
        'stock.location', 'Recycled Parts Destination Location',
        compute="_compute_recycle_location_id",
        store=True, readonly=False, required=True, precompute=True,
        index=True, check_company=True,
        help="This is the location where the repair parts are located.")

    # Parts
    move_ids = fields.One2many(
        'stock.move', 'repair_id', "Parts", check_company=True, copy=True,
        domain=[('repair_line_type', '!=', False)])  # Once RO switch to state done, a binded move is created for the "Product to repair" (move_id), this move appears in 'move_ids' if not filtered
    parts_availability = fields.Char(
        string="Component Status", compute='_compute_parts_availability',
        help="Latest parts availability status for this RO. If green, then the RO's readiness status is ready.")
    parts_availability_state = fields.Selection([
        ('available', 'Available'),
        ('expected', 'Expected'),
        ('late', 'Late')], compute='_compute_parts_availability')
    is_parts_available = fields.Boolean(
        'All Parts are available',
        default=False, store=True, compute='_compute_availability_boolean')
    is_parts_late = fields.Boolean(
        'Any Part is late',
        default=False, store=True, compute='_compute_availability_boolean')

    # Sale Order Binding
    sale_order_id = fields.Many2one(
        'sale.order', 'Sale Order', check_company=True, readonly=True,
        copy=False, help="Sale Order from which the Repair Order comes from.")
    sale_order_line_id = fields.Many2one(
        'sale.order.line', check_company=True, readonly=True,
        copy=False, help="Sale Order Line from which the Repair Order comes from.")
    repair_request = fields.Text(
        related='sale_order_line_id.name',
        string='Repair Request',
        help="Sale Order Line Description.")

    # Return Binding
    picking_id = fields.Many2one(
        'stock.picking', 'Return', check_company=True,
        domain="[('return_id', '!=', False), ('product_id', '=?', product_id)]",
        copy=False, help="Return Order from which the product to be repaired comes from.")
    is_returned = fields.Boolean(
        "Returned", compute='_compute_is_returned',
        help="True if this repair is linked to a Return Order and the order is 'Done'. False otherwise.")
    picking_product_ids = fields.One2many('product.product', compute='compute_picking_product_ids')
    picking_product_id = fields.Many2one(related="picking_id.product_id")
    # UI Fields
    show_set_qty_button = fields.Boolean(compute='_compute_show_qty_button')  # TODO: remove in master.
    show_clear_qty_button = fields.Boolean(compute='_compute_show_qty_button')  # TODO: remove in master.
    unreserve_visible = fields.Boolean(
        'Allowed to Unreserve Production', compute='_compute_unreserve_visible',
        help='Technical field to check when we can unreserve')
    reserve_visible = fields.Boolean(
        'Allowed to Reserve Production', compute='_compute_unreserve_visible',
        help='Technical field to check when we can reserve quantities')

    @api.depends('picking_id')
    def compute_picking_product_ids(self):
        for repair in self:
            repair.picking_product_ids = repair.picking_id.move_ids.product_id

    @api.depends('product_id', 'product_id.uom_id.category_id', 'product_uom.category_id')
    def compute_product_uom(self):
        for repair in self:
            if not repair.product_id:
                repair.product_uom = False
            elif not repair.product_uom or repair.product_uom.category_id != repair.product_id.uom_id.category_id:
                repair.product_uom = repair.product_id.uom_id

    @api.depends('product_id', 'lot_id', 'lot_id.product_id')
    def compute_lot_id(self):
        for repair in self:
            if (repair.product_id and repair.lot_id and repair.lot_id.product_id != repair.product_id) or not repair.product_id:
                repair.lot_id = False

    @api.depends('company_id')
    def _compute_picking_type_id(self):
        picking_type_by_company = self._get_picking_type()
        for ro in self:
            ro.picking_type_id = picking_type_by_company.get((ro.company_id, ro.user_id)) or\
                picking_type_by_company.get((ro.company_id, False))

    @api.depends('picking_type_id')
    def _compute_location_id(self):
        for repair in self:
            repair.location_id = repair.picking_type_id.default_location_src_id

    @api.depends('picking_type_id')
    def _compute_recycle_location_id(self):
        for repair in self:
            repair.recycle_location_id = repair.picking_type_id.default_recycle_location_dest_id

    @api.depends('state', 'schedule_date', 'move_ids', 'move_ids.forecast_availability', 'move_ids.forecast_expected_date')
    def _compute_parts_availability(self):
        repairs = self.filtered(lambda ro: ro.state in ('confirmed', 'under_repair'))
        repairs.parts_availability_state = 'available'
        repairs.parts_availability = _('Available')

        other_repairs = self - repairs
        other_repairs.parts_availability = False
        other_repairs.parts_availability_state = False

        all_moves = repairs.move_ids
        # Force to prefetch more than 1000 by 1000
        all_moves._fields['forecast_availability'].compute_value(all_moves)
        for repair in repairs:
            if any(float_compare(move.forecast_availability, move.product_qty, precision_rounding=move.product_id.uom_id.rounding) < 0 for move in repair.move_ids):
                repair.parts_availability = _('Not Available')
                repair.parts_availability_state = 'late'
                continue
            forecast_date = max(repair.move_ids.filtered('forecast_expected_date').mapped('forecast_expected_date'), default=False)
            if not forecast_date:
                continue
            repair.parts_availability = _('Exp %s', format_date(self.env, forecast_date))
            if repair.schedule_date:
                repair.parts_availability_state = 'late' if forecast_date > repair.schedule_date else 'expected'

    @api.depends('parts_availability_state')
    def _compute_availability_boolean(self):
        self.is_parts_available, self.is_parts_late = False, False
        for repair in self:
            if not repair.parts_availability_state:
                continue
            if repair.parts_availability_state == 'available':
                repair.is_parts_available = True
            elif repair.parts_availability_state == 'late':
                repair.is_parts_late = True

    @api.depends('picking_id', 'picking_id.state')
    def _compute_is_returned(self):
        self.is_returned = False
        returned = self.filtered(lambda r: r.picking_id and r.picking_id.state == 'done')
        returned.is_returned = True

    @api.depends('state', 'move_ids.quantity', 'move_ids.product_uom_qty')
    def _compute_show_qty_button(self):
        self.show_set_qty_button = False
        self.show_clear_qty_button = False

    @api.depends('move_ids', 'state', 'move_ids.product_uom_qty')
    def _compute_unreserve_visible(self):
        for repair in self:
            already_reserved = repair.state not in ('done', 'cancel') and any(repair.mapped('move_ids.move_line_ids.quantity_product_uom'))

            repair.unreserve_visible = already_reserved
            repair.reserve_visible = (
                repair.state in ('confirmed', 'under_repair') and
                any(not move.picked and move.product_uom_qty and move.state in ['confirmed', 'partially_available'] for move in repair.move_ids)
            )

    @api.onchange('product_uom')
    def onchange_product_uom(self):
        res = {}
        if not self.product_id or not self.product_uom:
            return res
        if self.product_uom.category_id != self.product_id.uom_id.category_id:
            res['warning'] = {'title': _('Warning'), 'message': _('The product unit of measure you chose has a different category than the product unit of measure.')}
        return res

    @api.onchange('location_id', 'picking_id')
    def _onchange_location_picking(self):
        location_warehouse = self.location_id.warehouse_id
        picking_warehouse = self.picking_id.location_dest_id.warehouse_id
        if location_warehouse and picking_warehouse and location_warehouse != picking_warehouse:
            return {
                'warning': {'title': _("Warning"), 'message': _("Note that the warehouses of the return and repair locations don't match!")},
            }

    @api.model
    def default_get(self, fields_list):
        # Adds the picking_id if it comes from a return. Avoids having a default_picking_id pollute the context for further move creation.
        res = super().default_get(fields_list)
        if 'picking_id' not in res and 'picking_id' in fields_list and 'default_repair_picking_id' in self.env.context:
            res['picking_id'] = self.env.context.get('default_repair_picking_id')
        if 'lot_id' not in res and 'lot_id' in fields_list and 'default_repair_lot_id' in self.env.context:
            res['lot_id'] = self.env.context.get('default_repair_lot_id')
        return res

    @api.model_create_multi
    def create(self, vals_list):
        # We generate a standard reference
        for vals in vals_list:
            picking_type = self.env['stock.picking.type'].browse(
                vals.get('picking_type_id', self.default_get(['picking_type_id'])['picking_type_id'])
            )
            if 'picking_type_id' not in vals:
                vals['picking_type_id'] = picking_type.id
            if not vals.get('name', False) or vals['name'] == 'New':
                vals['name'] = picking_type.sequence_id.next_by_id()
            if not vals.get('procurement_group_id'):
                vals['procurement_group_id'] = self.env["procurement.group"].create({'name': vals['name']}).id
        return super().create(vals_list)

    def write(self, vals):
        if vals.get('picking_type_id'):
            picking_type = self.env['stock.picking.type'].browse(vals.get('picking_type_id'))
            for repair in self:
                if picking_type != repair.picking_type_id:
                    repair.name = picking_type.sequence_id.next_by_id()
        res = super().write(vals)
        if 'product_id' in vals and self.tracking == 'serial':
            self.write({'product_qty': 1.0})

        for repair in self:
            has_modified_location = any(key in vals for key in MAP_REPAIR_TO_PICKING_LOCATIONS)
            if has_modified_location:
                repair.move_ids._set_repair_locations()
            if 'schedule_date' in vals:
                (repair.move_id + repair.move_ids).filtered(lambda m: m.state not in ('done', 'cancel')).write({'date': repair.schedule_date})
            if 'under_warranty' in vals:
                repair._update_sale_order_line_price()
        return res

    @api.ondelete(at_uninstall=False)
    def _unlink_except_confirmed(self):
        repairs_to_cancel = self.filtered(lambda ro: ro.state not in ('draft', 'cancel'))
        repairs_to_cancel.action_repair_cancel()

    def action_assign(self):
        return self.move_ids._action_assign()

    def action_create_sale_order(self):
        if any(repair.sale_order_id for repair in self):
            concerned_ro = self.filtered('sale_order_id')
            ref_str = "\n".join(ro.name for ro in concerned_ro)
            raise UserError(_("You cannot create a quotation for a repair order that is already linked to an existing sale order.\nConcerned repair order(s) :\n") + ref_str)
        if any(not repair.partner_id for repair in self):
            concerned_ro = self.filtered(lambda ro: not ro.partner_id)
            ref_str = "\n".join(ro.name for ro in concerned_ro)
            raise UserError(_("You need to define a customer for a repair order in order to create an associated quotation.\nConcerned repair order(s) :\n") + ref_str)
        sale_order_values_list = []
        for repair in self:
            sale_order_values_list.append({
                "company_id": self.company_id.id,
                "partner_id": self.partner_id.id,
                "warehouse_id": self.picking_type_id.warehouse_id.id,
                "repair_order_ids": [Command.link(repair.id)],
            })
        self.env['sale.order'].create(sale_order_values_list)
        # Add Sale Order Lines for 'add' move_ids
        self.move_ids._create_repair_sale_order_line()
        return self.action_view_sale_order()

    def action_repair_cancel(self):
        if any(repair.state == 'done' for repair in self):
            raise UserError(_("You cannot cancel a Repair Order that's already been completed"))
        for repair in self:
            if repair.sale_order_id:
                repair.sale_order_line_id.write({'product_uom_qty': 0.0})  # Quantity of the product that generated the RO is set to 0
        self.move_ids._action_cancel()  # Quantity of parts added from the RO to the SO is set to 0
        return self.write({'state': 'cancel'})

    def action_repair_cancel_draft(self):
        if self.filtered(lambda repair: repair.state != 'cancel'):
            self.action_repair_cancel()
        sale_line_to_update = self.move_ids.sale_line_id.filtered(lambda l: l.order_id.state != 'cancel' and float_is_zero(l.product_uom_qty, precision_rounding=l.product_uom.rounding))
        sale_line_to_update.move_ids._update_repair_sale_order_line()
        self.move_ids.state = 'draft'
        self.state = 'draft'
        return True

    def action_repair_done(self):
        """ Creates stock move for final product of repair order.
        Writes move_id and move_ids state to 'done'.
        Writes repair order state to 'Repaired'.
        @return: True
        """

        precision = self.env['decimal.precision'].precision_get('Product Unit of Measure')
        product_move_vals = []

        # Cancel moves with 0 quantity
        self.move_ids.filtered(lambda m: float_is_zero(m.quantity, precision_rounding=m.product_uom.rounding))._action_cancel()

        no_service_policy = 'service_policy' not in self.env['product.template']
        #SOL qty delivered = repair.move_ids.quantity
        for repair in self:
            if all(not move.picked for move in repair.move_ids):
                repair.move_ids.picked = True
            if repair.sale_order_line_id:
                ro_origin_product = repair.sale_order_line_id.product_template_id
                # TODO: As 'service_policy' only appears with 'sale_project' module, isolate conditions related to this field in a 'sale_project_repair' module if it's worth
                if ro_origin_product.detailed_type == 'service' and (no_service_policy or ro_origin_product.service_policy == 'ordered_prepaid'):
                    repair.sale_order_line_id.qty_delivered = repair.sale_order_line_id.product_uom_qty
            if not repair.product_id:
                continue

            if repair.product_id.product_tmpl_id.tracking != 'none' and not repair.lot_id:
                raise ValidationError(_(
                    "Serial number is required for product to repair : %s",
                    repair.product_id.display_name
                ))

            # Try to create move with the appropriate owner
            owner_id = False
            available_qty_owner = self.env['stock.quant']._get_available_quantity(repair.product_id, repair.location_id, repair.lot_id, owner_id=repair.partner_id, strict=True)
            if float_compare(available_qty_owner, repair.product_qty, precision_digits=precision) >= 0:
                owner_id = repair.partner_id.id

            product_move_vals.append({
                'name': repair.name,
                'product_id': repair.product_id.id,
                'product_uom': repair.product_uom.id or repair.product_id.uom_id.id,
                'product_uom_qty': repair.product_qty,
                'partner_id': repair.partner_id.id,
                'location_id': repair.location_id.id,
                'location_dest_id': repair.location_id.id,
                'picked': True,
                'picking_id': False,
                'move_line_ids': [(0, 0, {
                    'product_id': repair.product_id.id,
                    'lot_id': repair.lot_id.id,
                    'product_uom_id': repair.product_uom.id or repair.product_id.uom_id.id,
                    'quantity': repair.product_qty,
                    'package_id': False,
                    'result_package_id': False,
                    'owner_id': owner_id,
                    'location_id': repair.location_id.id,
                    'company_id': repair.company_id.id,
                    'location_dest_id': repair.location_id.id,
                    'consume_line_ids': [(6, 0, repair.move_ids.move_line_ids.ids)]
                })],
                'repair_id': repair.id,
                'origin': repair.name,
                'company_id': repair.company_id.id,
            })

        product_moves = self.env['stock.move'].create(product_move_vals)
        repair_move = {m.repair_id.id: m for m in product_moves}
        for repair in self:
            move_id = repair_move.get(repair.id, False)
            if move_id:
                repair.move_id = move_id
        all_moves = self.move_ids + product_moves
        all_moves._action_done(cancel_backorder=True)

        for sale_line in self.move_ids.sale_line_id:
            price_unit = sale_line.price_unit
            sale_line.write({'product_uom_qty': sale_line.qty_delivered, 'price_unit': price_unit})

        self.state = 'done'
        return True

    def action_repair_end(self):
        """ Checks before action_repair_done.
        @return: True
        """
        if self.filtered(lambda repair: repair.state != 'under_repair'):
            raise UserError(_("Repair must be under repair in order to end reparation."))
        partial_moves = set()
        picked_moves = set()
        for move in self.move_ids:
            if float_compare(move.quantity, move.product_uom_qty, precision_rounding=move.product_uom.rounding) < 0:
                partial_moves.add(move.id)
            if move.picked:
                picked_moves.add(move.id)
        if partial_moves or picked_moves and len(picked_moves) < len(self.move_ids):
            ctx = dict(self.env.context or {})
            ctx['default_repair_ids'] = self.ids
            return {
                'name': _('Uncomplete Move(s)'),
                'type': 'ir.actions.act_window',
                'view_mode': 'form',
                'views': [(False, 'form')],
                'res_model': 'repair.warn.uncomplete.move',
                'target': 'new',
                'context': ctx,
            }

        return self.action_repair_done()

    def action_repair_start(self):
        """ Writes repair order state to 'Under Repair'
        """
        if self.filtered(lambda repair: repair.state != 'confirmed'):
            self._action_repair_confirm()
        return self.write({'state': 'under_repair'})

    def action_unreserve(self):
        return self.move_ids.filtered(lambda m: m.state in ('assigned', 'partially_available'))._do_unreserve()

    def action_validate(self):
        self.ensure_one()
        if self.filtered(lambda repair: any(m.product_uom_qty < 0 for m in repair.move_ids)):
            raise UserError(_("You can not enter negative quantities."))
        if not self.product_id or self.product_id.type == 'consu':
            return self._action_repair_confirm()
        precision = self.env['decimal.precision'].precision_get('Product Unit of Measure')
        available_qty_owner = sum(self.env['stock.quant'].search([
            ('product_id', '=', self.product_id.id),
            ('location_id', '=', self.location_id.id),
            ('lot_id', '=', self.lot_id.id),
            ('owner_id', '=', self.partner_id.id),
        ]).mapped('quantity'))
        available_qty_noown = sum(self.env['stock.quant'].search([
            ('product_id', '=', self.product_id.id),
            ('location_id', '=', self.location_id.id),
            ('lot_id', '=', self.lot_id.id),
            ('owner_id', '=', False),
        ]).mapped('quantity'))
        repair_qty = self.product_uom._compute_quantity(self.product_qty, self.product_id.uom_id)
        for available_qty in [available_qty_owner, available_qty_noown]:
            if float_compare(available_qty, repair_qty, precision_digits=precision) >= 0:
                return self._action_repair_confirm()

        return {
            'name': self.product_id.display_name + _(': Insufficient Quantity To Repair'),
            'view_mode': 'form',
            'res_model': 'stock.warn.insufficient.qty.repair',
            'view_id': self.env.ref('repair.stock_warn_insufficient_qty_repair_form_view').id,
            'type': 'ir.actions.act_window',
            'context': {
                'default_product_id': self.product_id.id,
                'default_location_id': self.location_id.id,
                'default_repair_id': self.id,
                'default_quantity': repair_qty,
                'default_product_uom_name': self.product_id.uom_name
            },
            'target': 'new'
        }

    def action_view_sale_order(self):
        return {
            "type": "ir.actions.act_window",
            "res_model": "sale.order",
            "views": [[False, "form"]],
            "res_id": self.sale_order_id.id,
        }

    def print_repair_order(self):
        return self.env.ref('repair.action_report_repair_order').report_action(self)

    def _action_repair_confirm(self):
        """ Repair order state is set to 'Confirmed'.
        @param *arg: Arguments
        @return: True
        """
        repairs_to_confirm = self.filtered(lambda repair: repair.state == 'draft')
        repairs_to_confirm._check_company()
        repairs_to_confirm.move_ids._check_company()
        repairs_to_confirm.move_ids._adjust_procure_method()
        repairs_to_confirm.move_ids._action_confirm()
        repairs_to_confirm.move_ids._trigger_scheduler()
        repairs_to_confirm.write({'state': 'confirmed'})
        return True

    def _get_location(self, field):
        return self.picking_type_id[MAP_REPAIR_TO_PICKING_LOCATIONS[field]]

    def _get_picking_type(self):
        companies = self.company_id or self.env.company
        if not self:
            # default case
            default_warehouse = self.env.user.with_company(companies.id)._get_default_warehouse_id()
            if default_warehouse and default_warehouse.repair_type_id:
                return {(companies, self.env.user): default_warehouse.repair_type_id}

        picking_type_by_company_user = {}
        without_default_warehouse_companies = set()
        for (company, user), dummy in groupby(self, lambda r: (r.company_id, r.user_id)):
            default_warehouse = user.with_company(company.id)._get_default_warehouse_id()
            if default_warehouse and default_warehouse.repair_type_id:
                picking_type_by_company_user[(company, user)] = default_warehouse.repair_type_id
            else:
                without_default_warehouse_companies.add(company.id)

        if not without_default_warehouse_companies:
            return picking_type_by_company_user

        domain = [
            ('code', '=', 'repair_operation'),
            ('warehouse_id.company_id', 'in', list(without_default_warehouse_companies)),
        ]

        picking_types = self.env['stock.picking.type'].search_read(domain, ['company_id'], load=False)
        for picking_type in picking_types:
            if (picking_type.company_id, False) not in picking_type_by_company_user:
                picking_type_by_company_user[(picking_type.company_id, False)] = picking_type
        return picking_type_by_company_user

    def _update_sale_order_line_price(self):
        for repair in self:
            add_moves = repair.move_ids.filtered(lambda m: m.repair_line_type == 'add' and m.sale_line_id)
            if repair.under_warranty:
                add_moves.sale_line_id.write({'price_unit': 0.0})
            else:
                add_moves.sale_line_id._compute_price_unit()

class RepairTags(models.Model):
    """ Tags of Repair's tasks """
    _name = "repair.tags"
    _description = "Repair Tags"

    def _get_default_color(self):
        return randint(1, 11)

    name = fields.Char('Tag Name', required=True)
    color = fields.Integer(string='Color Index', default=_get_default_color)

    _sql_constraints = [
        ('name_uniq', 'unique (name)', "Tag name already exists!"),
    ]

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.tools import float_compare

class SaleOrder(models.Model):
    _inherit = 'sale.order'

    repair_order_ids = fields.One2many(
        comodel_name='repair.order', inverse_name='sale_order_id',
        string='Repair Order', groups='stock.group_stock_user')
    repair_count = fields.Integer(
        "Repair Order(s)", compute='_compute_repair_count', groups='stock.group_stock_user')

    @api.depends('repair_order_ids')
    def _compute_repair_count(self):
        for order in self:
            order.repair_count = len(order.repair_order_ids)

    def _action_cancel(self):
        res = super()._action_cancel()
        self.order_line._cancel_repair_order()
        return res

    def _action_confirm(self):
        res = super()._action_confirm()
        self.order_line._create_repair_order()
        return res

    def action_show_repair(self):
        self.ensure_one()
        if self.repair_count == 1:
            return {
                "type": "ir.actions.act_window",
                "res_model": "repair.order",
                "views": [[False, "form"]],
                "res_id": self.repair_order_ids.id,
            }
        elif self.repair_count > 1:
            return {
                "name": _("Repair Orders"),
                "type": "ir.actions.act_window",
                "res_model": "repair.order",
                "view_mode": "tree,form",
                "domain": [('sale_order_id', '=', self.id)],
            }


class SaleOrderLine(models.Model):
    _inherit = 'sale.order.line'

    def _compute_qty_delivered(self):
        remaining_so_lines = self
        for so_line in self:
            move = so_line.move_ids.sudo().filtered(lambda m: m.repair_id and m.state == 'done')
            if len(move) != 1:
                continue
            remaining_so_lines -= so_line
            so_line.qty_delivered = move.quantity
        return super(SaleOrderLine, remaining_so_lines)._compute_qty_delivered()

    @api.model_create_multi
    def create(self, vals_list):
        res = super().create(vals_list)
        res.filtered(lambda line: line.state in ('sale', 'done'))._create_repair_order()
        return res

    def write(self, vals_list):
        old_product_uom_qty = {line.id: line.product_uom_qty for line in self}
        res = super().write(vals_list)
        for line in self:
            if line.state in ('sale', 'done') and line.product_id:
                if float_compare(old_product_uom_qty[line.id], 0, precision_rounding=line.product_uom.rounding) <= 0 and float_compare(line.product_uom_qty, 0, precision_rounding=line.product_uom.rounding) > 0:
                    self._create_repair_order()
                if float_compare(old_product_uom_qty[line.id], 0, precision_rounding=line.product_uom.rounding) > 0 and float_compare(line.product_uom_qty, 0, precision_rounding=line.product_uom.rounding) <= 0:
                    self._cancel_repair_order()
        return res

    def _action_launch_stock_rule(self, previous_product_uom_qty=False):
        # Picking must be generated for products created from the SO but not for parts added from the RO, as they're already handled there
        lines_without_repair_move = self.filtered(lambda line: not line.move_ids.sudo().repair_id)
        return super(SaleOrderLine, lines_without_repair_move)._action_launch_stock_rule(previous_product_uom_qty)

    def _create_repair_order(self):
        new_repair_vals = []
        for line in self:
            # One RO for each line with at least a quantity of 1, quantities > 1 don't create multiple ROs
            if any(line.id == ro.sale_order_line_id.id for ro in line.order_id.sudo().repair_order_ids) and float_compare(line.product_uom_qty, 0, precision_rounding=line.product_uom.rounding) > 0:
                binded_ro_ids = line.order_id.sudo().repair_order_ids.filtered(lambda ro: ro.sale_order_line_id.id == line.id and ro.state == 'cancel')
                binded_ro_ids.action_repair_cancel_draft()
                binded_ro_ids._action_repair_confirm()
                continue
            if not line.product_template_id.sudo().create_repair or line.move_ids.sudo().repair_id or float_compare(line.product_uom_qty, 0, precision_rounding=line.product_uom.rounding) <= 0:
                continue

            order = line.order_id
            default_repair_vals = {
                'state': 'confirmed',
                'partner_id': order.partner_id.id,
                'sale_order_id': order.id,
                'sale_order_line_id': line.id,
                'picking_type_id': order.warehouse_id.repair_type_id.id,
            }
            if line.product_id.tracking == 'serial':
                vals = {
                    **default_repair_vals,
                    'product_id': line.product_id.id,
                    'product_qty': 1,
                    'product_uom': line.product_uom.id,
                }
                new_repair_vals.extend([vals] * int(line.product_uom_qty))
            elif line.product_id.type in ('consu', 'product'):
                new_repair_vals.append({
                    **default_repair_vals,
                    'product_id': line.product_id.id,
                    'product_qty': line.product_uom_qty,
                    'product_uom': line.product_uom.id,
                })
            else:
                new_repair_vals.append(default_repair_vals.copy())

        if new_repair_vals:
            self.env['repair.order'].sudo().create(new_repair_vals)

    def _cancel_repair_order(self):
        # Each RO binded to a SO line with Qty set to 0 or cancelled is set to 'Cancelled'
        binded_ro_ids = self.env['repair.order']
        for line in self:
            binded_ro_ids |= line.order_id.sudo().repair_order_ids.filtered(lambda ro: ro.sale_order_line_id.id == line.id and ro.state != 'done')
        binded_ro_ids.action_repair_cancel()

    def has_valued_move_ids(self):
        res = super().has_valued_move_ids()
        return res and not self.move_ids.repair_id

```

## File: models\stock_lot.py

```python
# -*- coding: utf-8 -*-

from collections import defaultdict
from odoo import api, fields, models, _


class StockLot(models.Model):
    _inherit = 'stock.lot'

    repair_line_ids = fields.Many2many('repair.order', string="Repair Orders", compute="_compute_repair_line_ids")
    repair_part_count = fields.Integer('Repair part count', compute="_compute_repair_line_ids")
    in_repair_count = fields.Integer('In repair count', compute="_compute_in_repair_count")
    repaired_count = fields.Integer('Repaired count', compute='_compute_repaired_count')

    @api.depends('name')
    def _compute_repair_line_ids(self):
        repair_orders = defaultdict(lambda: self.env['repair.order'])
        repair_moves = self.env['stock.move'].search([
            ('repair_id', '!=', False),
            ('repair_line_type', '!=', False),
            ('move_line_ids.lot_id', 'in', self.ids),
            ('state', '=', 'done')])
        for repair_line in repair_moves:
            for rl_id in repair_line.lot_ids.ids:
                repair_orders[rl_id] |= repair_line.repair_id
        for lot in self:
            lot.repair_line_ids = repair_orders[lot.id]
            lot.repair_part_count = len(lot.repair_line_ids)

    def _compute_in_repair_count(self):
        lot_data = self.env['repair.order']._read_group([('lot_id', 'in', self.ids), ('state', 'not in', ('done', 'cancel'))], ['lot_id'], ['__count'])
        result = {lot.id: count for lot, count in lot_data}
        for lot in self:
            lot.in_repair_count = result.get(lot.id, 0)

    def _compute_repaired_count(self):
        lot_data = self.env['repair.order']._read_group([('lot_id', 'in', self.ids), ('state', '=', 'done')], ['lot_id'], ['__count'])
        result = {lot.id: count for lot, count in lot_data}
        for lot in self:
            lot.repaired_count = result.get(lot.id, 0)

    def action_lot_open_repairs(self):
        action = self.env["ir.actions.actions"]._for_xml_id("repair.action_repair_order_tree")
        action.update({
            'domain': [('lot_id', '=', self.id)],
            'context': {
                'default_product_id': self.product_id.id,
                'default_repair_lot_id': self.id,
                'default_company_id': self.company_id.id,
            },
        })
        return action

    def action_view_ro(self):
        self.ensure_one()

        action = {
            'res_model': 'repair.order',
            'type': 'ir.actions.act_window'
        }
        if len(self.repair_line_ids) == 1:
            action.update({
                'view_mode': 'form',
                'res_id': self.repair_line_ids[0].id
            })
        else:
            action.update({
                'name': _("Repair orders of %s", self.name),
                'domain': [('id', 'in', self.repair_line_ids.ids)],
                'view_mode': 'tree,form'
            })
        return action

```

## File: models\stock_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, Command, fields, models
from odoo.tools.misc import groupby

MAP_REPAIR_LINE_TYPE_TO_MOVE_LOCATIONS_FROM_REPAIR = {
    'add': {'location_id': 'location_id', 'location_dest_id': 'location_dest_id'},
    'remove': {'location_id': 'location_dest_id', 'location_dest_id': 'parts_location_id'},
    'recycle': {'location_id': 'location_dest_id', 'location_dest_id': 'recycle_location_id'},
}


class StockMove(models.Model):
    _inherit = 'stock.move'

    repair_id = fields.Many2one('repair.order', check_company=True)
    repair_line_type = fields.Selection([
        ('add', 'Add'),
        ('remove', 'Remove'),
        ('recycle', 'Recycle')
    ], 'Type', store=True, index=True)

    @api.depends('repair_line_type')
    def _compute_forecast_information(self):
        moves_to_compute = self.filtered(lambda move: not move.repair_line_type or move.repair_line_type == 'add')
        for move in (self - moves_to_compute):
            move.forecast_availability = move.product_qty
            move.forecast_expected_date = False
        return super(StockMove, moves_to_compute)._compute_forecast_information()

    @api.depends('repair_id.picking_type_id')
    def _compute_picking_type_id(self):
        remaining_moves = self
        for move in self:
            if move.repair_id:
                move.picking_type_id = move.repair_id.picking_type_id
                remaining_moves -= move
        return super(StockMove, remaining_moves)._compute_picking_type_id()

    def copy_data(self, default=None):
        default = dict(default or {})
        if 'repair_id' in default or self.repair_id:
            default['sale_line_id'] = False
        return super().copy_data(default)

    @api.ondelete(at_uninstall=False)
    def _unlink_if_draft_or_cancel(self):
        self.filtered('repair_id')._action_cancel()
        return super()._unlink_if_draft_or_cancel()

    def unlink(self):
        self._clean_repair_sale_order_line()
        return super().unlink()

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            if not vals.get('repair_id') or 'repair_line_type' not in vals:
                continue
            repair_id = self.env['repair.order'].browse([vals['repair_id']])
            vals['name'] = repair_id.name
            src_location, dest_location = self._get_repair_locations(vals['repair_line_type'], repair_id)
            if not vals.get('location_id'):
                vals['location_id'] = src_location.id
            if not vals.get('location_dest_id'):
                vals['location_dest_id'] = dest_location.id
        moves = super().create(vals_list)
        repair_moves = self.env['stock.move']
        for move in moves:
            if not move.repair_id:
                continue
            move.group_id = move.repair_id.procurement_group_id.id
            move.origin = move.name
            move.picking_type_id = move.repair_id.picking_type_id.id
            repair_moves |= move
        no_repair_moves = moves - repair_moves
        draft_repair_moves = repair_moves.filtered(lambda m: m.state == 'draft' and m.repair_id.state in ('confirmed', 'under_repair'))
        other_repair_moves = repair_moves - draft_repair_moves
        draft_repair_moves._check_company()
        draft_repair_moves._adjust_procure_method()
        res = draft_repair_moves._action_confirm()
        res._trigger_scheduler()
        confirmed_repair_moves = (res | other_repair_moves)
        confirmed_repair_moves._create_repair_sale_order_line()
        return (confirmed_repair_moves | no_repair_moves)

    def write(self, vals):
        res = super().write(vals)
        repair_moves = self.env['stock.move']
        moves_to_create_so_line = self.env['stock.move']
        for move in self:
            if not move.repair_id:
                continue
            # checks vals update
            if 'repair_line_type' in vals or 'picking_type_id' in vals and move.repair_line_type:
                move.location_id, move.location_dest_id = move._get_repair_locations(move.repair_line_type)
            if not move.sale_line_id and 'sale_line_id' not in vals and move.repair_line_type == 'add':
                moves_to_create_so_line |= move
            if move.sale_line_id and ('repair_line_type' in vals or 'product_uom_qty' in vals):
                repair_moves |= move

        repair_moves._update_repair_sale_order_line()
        moves_to_create_so_line._create_repair_sale_order_line()
        return res

    # Needed to also cancel the lastly added part
    def _action_cancel(self):
        self._clean_repair_sale_order_line()
        return super()._action_cancel()

    def _create_repair_sale_order_line(self):
        if not self:
            return
        so_line_vals = []
        for move in self:
            if move.sale_line_id or move.repair_line_type != 'add' or not move.repair_id.sale_order_id:
                continue
            product_qty = move.product_uom_qty if move.repair_id.state != 'done' else move.quantity
            so_line_vals.append({
                'order_id': move.repair_id.sale_order_id.id,
                'product_id': move.product_id.id,
                'product_uom_qty': product_qty, # When relying only on so_line compute method, the sol quantity is only updated on next sol creation
                'move_ids': [Command.link(move.id)],
            })
            if move.repair_id.under_warranty:
                so_line_vals[-1]['price_unit'] = 0.0
            elif move.price_unit:
                so_line_vals[-1]['price_unit'] = move.price_unit

        self.env['sale.order.line'].create(so_line_vals)

    def _clean_repair_sale_order_line(self):
        self.filtered(
            lambda m: m.repair_id and m.sale_line_id
        ).mapped('sale_line_id').write({'product_uom_qty': 0.0})

    def _update_repair_sale_order_line(self):
        if not self:
            return
        moves_to_clean = self.env['stock.move']
        moves_to_update = self.env['stock.move']
        for move in self:
            if not move.repair_id:
                continue
            if move.sale_line_id and move.repair_line_type != 'add':
                moves_to_clean |= move
            if move.sale_line_id and move.repair_line_type == 'add':
                moves_to_update |= move
        moves_to_clean._clean_repair_sale_order_line()
        for sale_line, _ in groupby(moves_to_update, lambda m: m.sale_line_id):
            sale_line.product_uom_qty = sum(sale_line.move_ids.mapped('product_uom_qty'))

    def _is_consuming(self):
        return super()._is_consuming() or (self.repair_id and self.repair_line_type == 'add')

    def _get_repair_locations(self, repair_line_type, repair_id=False):
        location_map = MAP_REPAIR_LINE_TYPE_TO_MOVE_LOCATIONS_FROM_REPAIR.get(repair_line_type)
        if location_map:
            if not repair_id:
                self.repair_id.ensure_one()
                repair_id = self.repair_id
            location_id, location_dest_id = [repair_id[field] for field in location_map.values()]
        else:
            location_id, location_dest_id = False, False
        return location_id, location_dest_id

    def _get_source_document(self):
        return self.repair_id or super()._get_source_document()

    def _set_repair_locations(self):
        moves_per_repair = self.filtered(lambda m: (m.repair_id and m.repair_line_type) is not False).grouped('repair_id')
        if not moves_per_repair:
            return
        for moves in moves_per_repair.values():
            grouped_moves = moves.grouped('repair_line_type')
            for line_type, m in grouped_moves.items():
                m.location_id, m.location_dest_id = m._get_repair_locations(line_type)

    def _should_be_assigned(self):
        if self.repair_id:
            return False
        return super()._should_be_assigned()

    def _create_extra_move(self):
        if self.repair_id:
            return self
        return super(StockMove, self)._create_extra_move()

    def _split(self, qty, restrict_partner_id=False):
        # When setting the Repair Order as done with partially done moves, do not split these moves
        if self.repair_id:
            return []
        return super(StockMove, self)._split(qty, restrict_partner_id)

    def action_show_details(self):
        action = super().action_show_details()
        if self.repair_line_type == 'recycle':
            action['context'].update({'show_quant': False, 'show_destination_location': True})
        return action

```

## File: models\stock_move_line.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class StockMoveLine(models.Model):
    _inherit = 'stock.move.line'

    def _should_show_lot_in_invoice(self):
        return super()._should_show_lot_in_invoice() or self.move_id.repair_line_type

```

## File: models\stock_picking.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import time

from odoo import _, api, fields, models
from odoo.tools import DEFAULT_SERVER_DATETIME_FORMAT
from odoo.tools.misc import clean_context


class PickingType(models.Model):
    _inherit = 'stock.picking.type'

    code = fields.Selection(selection_add=[
        ('repair_operation', 'Repair')
    ], ondelete={'repair_operation': 'cascade'})

    count_repair_confirmed = fields.Integer(
        string="Number of Repair Orders Confirmed", compute='_compute_count_repair')
    count_repair_under_repair = fields.Integer(
        string="Number of Repair Orders Under Repair", compute='_compute_count_repair')
    count_repair_ready = fields.Integer(
        string="Number of Repair Orders to Process", compute='_compute_count_repair')

    default_remove_location_dest_id = fields.Many2one(
        'stock.location', 'Default Remove Destination Location', compute='_compute_default_remove_location_dest_id',
        check_company=True, store=True, readonly=False, precompute=True,
        help="This is the default remove destination location when you create a repair order with this operation type.")

    default_recycle_location_dest_id = fields.Many2one(
        'stock.location', 'Default Recycle Destination Location', compute='_compute_default_recycle_location_dest_id',
        check_company=True, store=True, readonly=False, precompute=True,
        help="This is the default recycle destination location when you create a repair order with this operation type.")

    is_repairable = fields.Boolean(
        'Create Repair Orders from Returns',
        compute='_compute_is_repairable', store=True, readonly=False, default=False,
        help="If ticked, you will be able to directly create repair orders from a return.")
    return_type_of_ids = fields.One2many('stock.picking.type', 'return_picking_type_id')

    def _compute_count_repair(self):
        repair_picking_types = self.filtered(lambda picking: picking.code == 'repair_operation')

        # By default, set count_repair_xxx to False
        self.count_repair_ready = False
        self.count_repair_confirmed = False
        self.count_repair_under_repair = False

        # shortcut
        if not repair_picking_types:
            return

        picking_types = self.env['repair.order']._read_group(
            [
                ('picking_type_id', 'in', repair_picking_types.ids),
                ('state', 'in', ('confirmed', 'under_repair')),
            ],
            groupby=['picking_type_id', 'is_parts_available', 'state'],
            aggregates=['id:count']
        )

        counts = {}
        for pt in picking_types:
            pt_count = counts.setdefault(pt[0].id, {})
            if pt[1]:
                pt_count.setdefault('ready', 0)
                pt_count['ready'] += pt[3]
            pt_count.setdefault(pt[2], 0)
            pt_count[pt[2]] += pt[3]

        for pt in repair_picking_types:
            if pt.id not in counts:
                continue
            pt.count_repair_ready = counts[pt.id].get('ready')
            pt.count_repair_confirmed = counts[pt.id].get('confirmed')
            pt.count_repair_under_repair = counts[pt.id].get('under_repair')

    @api.depends('return_type_of_ids', 'code')
    def _compute_is_repairable(self):
        for picking_type in self:
            if not picking_type.return_type_of_ids:
                picking_type.is_repairable = False  # Reset the user choice as it's no more available.

    def _compute_default_location_src_id(self):
        remaining_picking_type = self.env['stock.picking.type']
        for picking_type in self:
            if picking_type.code != 'repair_operation':
                remaining_picking_type |= picking_type
                continue
            stock_location = picking_type.warehouse_id.lot_stock_id
            picking_type.default_location_src_id = stock_location.id
        super(PickingType, remaining_picking_type)._compute_default_location_src_id()

    def _compute_default_location_dest_id(self):
        repair_picking_type = self.filtered(lambda pt: pt.code == 'repair_operation')
        prod_locations = self.env['stock.location']._read_group(
            [('usage', '=', 'production'), ('company_id', 'in', repair_picking_type.company_id.ids)],
            ['company_id'],
            ['id:min'],
        )
        prod_locations = {l[0].id: l[1] for l in prod_locations}
        for picking_type in repair_picking_type:
            picking_type.default_location_dest_id = prod_locations.get(picking_type.company_id.id)
        super(PickingType, (self - repair_picking_type))._compute_default_location_dest_id()

    @api.depends('code')
    def _compute_default_remove_location_dest_id(self):
        repair_picking_type = self.filtered(lambda pt: pt.code == 'repair_operation')
        company_ids = repair_picking_type.company_id.ids
        company_ids.append(False)
        scrap_locations = self.env['stock.location']._read_group(
            [('scrap_location', '=', True), ('company_id', 'in', company_ids)],
            ['company_id'],
            ['id:min'],
        )
        scrap_locations = {l[0].id: l[1] for l in scrap_locations}
        for picking_type in repair_picking_type:
            picking_type.default_remove_location_dest_id = scrap_locations.get(picking_type.company_id.id)

    @api.depends('code')
    def _compute_default_recycle_location_dest_id(self):
        for picking_type in self:
            if picking_type.code == 'repair_operation':
                stock_location = picking_type.warehouse_id.lot_stock_id
                picking_type.default_recycle_location_dest_id = stock_location.id

    def get_repair_stock_picking_action_picking_type(self):
        action = self.env["ir.actions.actions"]._for_xml_id('repair.action_picking_repair')
        if self:
            action['display_name'] = self.display_name
        return action


class Picking(models.Model):
    _inherit = 'stock.picking'

    is_repairable = fields.Boolean(compute='_compute_is_repairable')
    repair_ids = fields.One2many('repair.order', 'picking_id')
    nbr_repairs = fields.Integer('Number of repairs linked to this picking', compute='_compute_nbr_repairs')

    @api.depends('picking_type_id.is_repairable', 'return_id')
    def _compute_is_repairable(self):
        for picking in self:
            picking.is_repairable = picking.picking_type_id.is_repairable and picking.return_id

    @api.depends('repair_ids')
    def _compute_nbr_repairs(self):
        for picking in self:
            picking.nbr_repairs = len(picking.repair_ids)

    def action_repair_return(self):
        self.ensure_one()
        ctx = clean_context(self.env.context.copy())
        ctx.update({
            'default_location_id': self.location_dest_id.id,
            'default_repair_picking_id': self.id,
            'default_picking_type_id': self.picking_type_id.warehouse_id.repair_type_id.id,
            'default_partner_id': self.partner_id and self.partner_id.id or False,
        })
        return {
            'name': _('Create Repair'),
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'repair.order',
            'view_id': self.env.ref('repair.view_repair_order_form').id,
            'context': ctx,
        }

    def action_view_repairs(self):
        if self.repair_ids:
            action = {
                'res_model': 'repair.order',
                'type': 'ir.actions.act_window',
            }
            if len(self.repair_ids) == 1:
                action.update({
                    'view_mode': 'form',
                    'res_id': self.repair_ids[0].id,
                })
            else:
                action.update({
                    'name': _('Repair Orders'),
                    'view_mode': 'tree,form',
                    'domain': [('id', 'in', self.repair_ids.ids)],
                })
            return action

```

## File: models\stock_traceability.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, api


class MrpStockReport(models.TransientModel):
    _inherit = 'stock.traceability.report'

    @api.model
    def _get_reference(self, move_line):
        res_model, res_id, ref = super(MrpStockReport, self)._get_reference(move_line)
        if move_line.move_id.repair_id:
            res_model = 'repair.order'
            res_id = move_line.move_id.repair_id.id
            ref = move_line.move_id.repair_id.name
        return res_model, res_id, ref

    @api.model
    def _get_linked_move_lines(self, move_line):
        move_lines, is_used = super(MrpStockReport, self)._get_linked_move_lines(move_line)
        if not move_lines:
            move_lines = move_line.move_id.repair_id and move_line.consume_line_ids
        if not is_used:
            is_used = move_line.move_id.repair_id and move_line.produce_line_ids
        return move_lines, is_used

```

## File: models\stock_warehouse.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _


class StockWarehouse(models.Model):
    _inherit = 'stock.warehouse'

    repair_type_id = fields.Many2one('stock.picking.type', 'Repair Operation Type', check_company=True, copy=False)

    def _get_sequence_values(self, name=False, code=False):
        values = super(StockWarehouse, self)._get_sequence_values(name=name, code=code)
        values.update({
            'repair_type_id': {
                'name': self.name + ' ' + _('Sequence repair'),
                'prefix': self.code + '/' + (self.repair_type_id.sequence_code or 'RO') + '/',
                'padding': 5,
                'company_id': self.company_id.id
                },
        })
        return values

    def _get_picking_type_create_values(self, max_sequence):
        data, next_sequence = super(StockWarehouse, self)._get_picking_type_create_values(max_sequence)
        prod_location = self.env['stock.location'].search([('usage', '=', 'production'), ('company_id', '=', self.company_id.id)], limit=1)
        scrap_location = self.env['stock.location'].search([('scrap_location', '=', True), ('company_id', 'in', [self.company_id.id, False])], limit=1)
        data.update({
            'repair_type_id': {
                'name': _('Repairs'),
                'code': 'repair_operation',
                'default_location_src_id': self.lot_stock_id.id,
                'default_location_dest_id': prod_location.id,
                'default_remove_location_dest_id':scrap_location.id,
                'default_recycle_location_dest_id': self.lot_stock_id.id,
                'sequence': next_sequence + 1,
                'sequence_code': 'RO',
                'company_id': self.company_id.id,
            },
        })
        return data, max_sequence + 2

    def _get_picking_type_update_values(self):
        data = super(StockWarehouse, self)._get_picking_type_update_values()
        data.update({
            'repair_type_id': {
                'active': self.active,
                'barcode': self.code.replace(" ", "").upper() + "-RO",
            },
        })
        return data

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import repair
from . import stock_move
from . import stock_move_line
from . import stock_picking
from . import stock_traceability
from . import stock_lot
from . import product
from . import sale_order
from . import stock_warehouse

```

## File: report\repair_reports.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="action_report_repair_order" model="ir.actions.report">
            <field name="name">Repair Order</field>
            <field name="model">repair.order</field>
            <field name="report_type">qweb-pdf</field>
            <field name="report_name">repair.report_repairorder2</field>
            <field name="report_file">repair.report_repairorder</field>
            <field name="print_report_name">('Repair Order - %s' % (object.name))</field>
            <field name="binding_model_id" ref="model_repair_order"/>
            <field name="binding_type">report</field>
        </record>
    </data>
</odoo>

```

## File: report\repair_templates_repair_order.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data>
<template id="report_repairorder">
        <t t-set="o" t-value="doc"/>
            <t t-call="web.external_layout">
                <t t-set="o" t-value="o.with_context(lang=o.partner_id.lang)" />
                <div class="page">
                    <div class="oe_structure"/>
                    <h2>
                        <span>Repair Order #:</span>
                        <span t-field="o.name">RO123456</span>
                    </h2>
                    <div class="oe_structure"/>
                    <div id="informations" class="row mb-3">
                        <div class="col-6">
                            <p t-if="o.partner_id" class="m-0">
                                <strong>Customer:</strong>
                                <span t-field="o.partner_id">John Doe</span>
                            </p>
                            <p t-if="o.product_id" class="m-0">
                                <strong>Product:</strong>
                                <span t-field="o.product_id">Laptop</span>
                            </p>
                            <p t-if="o.lot_id" class="m-0" groups="stock.group_production_lot">
                                <strong>Lot/Serial:</strong>
                                <span t-field="o.lot_id">L12345</span>
                            </p>
                        </div>
                        <div class="col-6">
                            <p class="m-0">
                                <strong>Status:</strong>
                                <span t-field="o.state">Pending</span>
                            </p>
                            <p t-if="o.user_id" class="m-0">
                                <strong>Responsible:</strong>
                                <span t-field="o.user_id">Jane Smith</span>
                            </p>
                        </div>
                    </div>
                    <div class="oe_structure"/>
                    <h2 class="mb-3 border-bottom border-2 border-dark">Parts</h2>
                    <table class="table table-sm o_main_table">
                        <thead>
                            <tr>
                                <th>Description</th>
                                <th class="text-end">Quantity</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr t-foreach="o.move_ids" t-as="line">
                                <td>
                                    <p t-if="line.repair_line_type == 'add'"><i>(Add)</i> <span t-field="line.product_id">Product A</span></p>
                                    <p t-if="line.repair_line_type == 'remove'">(<i>Remove</i>) <span t-field="line.product_id">Product B</span></p>
                                    <p t-if="line.repair_line_type == 'recycle'">(<i>Recycle</i>) <span t-field="line.product_id" data-oe-demo="Product C"/></p>
                                </td>
                                <td class="text-end">
                                    <span t-field="line.product_uom_qty">5</span>
                                    <span groups="uom.group_uom" t-field="line.product_uom.name">Units</span>
                                </td>
                            </tr>
                        </tbody>
                    </table>

                    <div class="oe_structure"/>
                    <div t-if="o.internal_notes">
                        <h2 class="mb-3 border-bottom border-2 border-dark">Repair Notes</h2>
                        <span t-field="o.internal_notes">This is a repair note.</span>
                    </div>
                    <div class="oe_structure"/>
                </div>
            </t>
</template>

<template id="report_repairorder2">
    <t t-call="web.html_container">
        <t t-foreach="docs" t-as="doc">
            <t t-call="repair.report_repairorder" t-lang="doc.partner_id.lang"/>
        </t>
    </t>
</template>
</data>
</odoo>

```

## File: report\stock_forecasted.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class StockForecasted(models.AbstractModel):
    _inherit = 'stock.forecasted_product_product'

    def _get_reservation_data(self, move):
        if move.repair_id and move.repair_line_type:
            return False
        return super()._get_reservation_data(move)

    def _product_sale_domain(self, product_template_ids, product_ids):
        """
        When a product's move is bind at the same time to a Repair Order
        and to a Sale Order, only take the data into account once, as a RO
        """
        sol_domain = super(StockForecasted, self)._product_sale_domain(product_template_ids, product_ids)
        move_domain = self._product_domain(product_template_ids, product_ids)
        move_domain += [
            ('repair_id', '!=', False),
            ('sale_line_id', '!=', False),
            ('repair_line_type', '=', 'add')
        ]
        sol_ids = self.env['stock.move']._read_group(move_domain, aggregates=['sale_line_id:array_agg'])[0][0]

        sol_domain += [('id', 'not in', sol_ids)]
        return sol_domain

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import stock_forecasted

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_repair_user,Repair user,model_repair_order,stock.group_stock_user,1,1,1,1
access_repair_tag_user,Repair Tags user,model_repair_tags,stock.group_stock_user,1,1,1,1
access_stock_warn_insufficient_qty_repair,access.stock.warn.insufficient.qty.repair,model_stock_warn_insufficient_qty_repair,stock.group_stock_user,1,1,1,0
access_repair_warn_uncomplete_move,access.repair.warn.uncomplete.move,model_repair_warn_uncomplete_move,stock.group_stock_user,1,1,1,0

```

## File: security\repair_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <!-- Multicompay rules-->
    <record model="ir.rule" id="repair_order_rule">
        <field name="name">repair order multi-company</field>
        <field name="model_id" search="[('model','=','repair.order')]" model="ir.model"/>
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>

</odoo>

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M29.319 6.862c3.816-3.816 10.003-3.816 13.819 0 3.816 3.816 3.816 10.003 0 13.82L20.68 43.137c-3.816 3.816-10.003 3.816-13.819 0-3.816-3.816-3.816-10.003 0-13.82L29.32 6.863Z" fill="#FC868B"/><path d="M43.138 29.319c3.816 3.816 3.816 10.003 0 13.819-3.816 3.816-10.003 3.816-13.82 0L6.863 20.68c-3.816-3.816-3.816-10.003 0-13.819 3.816-3.816 10.003-3.816 13.82 0L43.138 29.32Z" fill="#1AD3BB"/><path d="M24.999 11.18 11.179 25 25 38.82 38.82 25 25 11.18Z" fill="#1A6F66"/></svg>

```

## File: views\product_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_product_template_form_inherit_repair" model="ir.ui.view">
        <field name="name">product.template.form.inherit.repair</field>
        <field name="model">product.template</field>
        <field name="inherit_id" ref="product.product_template_form_view"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='group_general']/field[@name='uom_po_id']" position="after">
                <field name="create_repair" invisible="detailed_type not in ('consu', 'product', 'service')"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\repair_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <data>

    <record id="repair_order_view_activity" model="ir.ui.view">
        <field name="name">repair.order.view.activity</field>
        <field name="model">repair.order</field>
        <field name="arch" type="xml">
            <activity string="Activity view">
                <templates>
                    <div t-name="activity-box">
                        <field name="user_id" widget="many2one_avatar_user" domain="[('share', '=', False)]"/>
                        <div>
                            <field name="name" display="full" class="o_text_block o_text_bold"/>
                            <field name="product_id" class="o_text_block"/>
                            <field name="schedule_date" widget="date" class="d-block"/>
                        </div>
                    </div>
                </templates>
            </activity>
        </field>
    </record>

    <record id="view_repair_order_tree" model="ir.ui.view">
        <field name="name">repair.tree</field>
        <field name="model">repair.order</field>
        <field name="arch" type="xml">
            <tree string="Repairs order" multi_edit="1" sample="1" decoration-info="state == 'draft'">
                <field name="company_id" column_invisible="True"/>
                <field name="priority" optional="show" widget="priority" nolabel="1"/>
                <field name="name"/>
                <field name="schedule_date" optional="show" widget="remaining_days"/>
                <field name="product_id" readonly="1" optional="show"/>
                <field name="parts_availability_state" column_invisible="True"/>
                <field name="parts_availability"
                    invisible="state not in ['confirmed', 'under_repair']"
                    optional="show"
                    decoration-success="parts_availability_state == 'available'"
                    decoration-warning="parts_availability_state == 'expected'"
                    decoration-danger="parts_availability_state == 'late'"/>
                <field name="product_qty" optional="hide" string="Quantity" readonly="state != 'draft'"/>
                <field name="product_uom" string="Unit of Measure" readonly="1" groups="uom.group_uom" optional="hide"/>
                <field name="user_id" optional="hide" widget='many2one_avatar_user'/>
                <field name="partner_id" readonly="1" optional="show"/>
                <field name="picking_id" optional="hide"/>
                <field name="is_returned" optional="hide"/>
                <field name="sale_order_id" optional="show"/>
                <field name="location_id" optional="hide"/>
                <field name="company_id" groups="base.group_multi_company" readonly="1" optional="show"/>
                <field name="state" optional="show" widget='badge' decoration-success="state == 'done'" decoration-info="state not in ('done', 'cancel')"/>
                <field name="activity_exception_decoration" widget="activity_exception"/>
            </tree>
        </field>
    </record>

    <record id="view_repair_order_form" model="ir.ui.view">
        <field name="name">repair.form</field>
        <field name="model">repair.order</field>
        <field name="arch" type="xml">
            <form string="Repair Order">
                <field name="unreserve_visible" invisible="1"/>
                <field name="reserve_visible" invisible="1"/>
               <header>
                   <button name="action_validate" invisible="state != 'draft'" type="object" string="Confirm Repair" class="oe_highlight" data-hotkey="v"/>
                   <button name="action_repair_start" invisible="state != 'confirmed'" type="object" string="Start Repair" class="oe_highlight" data-hotkey="q"/>
                   <button name="action_repair_end" invisible="state != 'under_repair'" type="object" string="End Repair" class="oe_highlight" data-hotkey="x"/>
                   <button name="action_assign" invisible="state in ('draft', 'done', 'cancel') or not reserve_visible" string="Check availability" type="object"/>
                   <button name="action_unreserve" type="object" string="Unreserve" invisible="not unreserve_visible" data-hotkey="w"/>
                   <button name="action_create_sale_order" type="object" string="Create Quotation" invisible="state == 'cancel' or sale_order_id"/>
                   <button name="action_repair_cancel" string="Cancel Repair" type="object" invisible="state in ('done', 'cancel')" data-hotkey="l"/>
                   <button name="action_repair_cancel_draft" invisible="state != 'cancel'" string="Set to Draft" type="object" data-hotkey="z"/>
                   <field name="state" widget="statusbar" statusbar_visible="draft,confirmed,under_repair,done"/>
               </header>
               <sheet string="Repairs order">
                    <div class="oe_button_box" name="button_box">
                        <!-- No groups attribute on the next button as "stock.group_stock_user" is needed for Repair, and as this group is granted 'sale.order' read/write accesses in sale_stock module (forcefully loaded as transitive dependency) -->
                        <button name="action_view_sale_order" type="object" string="Sale Order" icon="fa-dollar" class="oe_stat_button" invisible="not sale_order_id">
                            <div class="o_field_widget o_stat_info">
                                <span class="o_stat_value">
                                    <field name="sale_order_id" widget="statinfo" nolabel="1" class="mr4"/>
                                </span>
                                <span class="o_stat_text">Sale Order</span>
                            </div>
                        </button>
                        <button name="%(action_repair_move_lines)d" type="action" string="Product Moves" class="oe_stat_button" icon="fa-exchange" invisible="state not in ['done', 'cancel']"/>
                    </div>
                    <div class="oe_title">
                        <label class="o_form_label" for="name"/>
                        <h1 class="d-flex">
                            <field name="priority" widget="priority" class="me-3"/>
                            <field name="name"/>
                        </h1>
                    </div>
                    <group>
                        <group>
                            <field name="picking_product_ids" invisible="1"/>
                            <field name="picking_product_id" invisible="1"/>
                            <field name="tracking" invisible="1" readonly="True"/>
                            <field name="company_id" invisible="1"/>
                            <field name="sale_order_id" invisible="1"/>
                            <field name="sale_order_line_id" invisible="1"/>
                            <field name="repair_request" invisible="not sale_order_line_id"/>
                            <field name="partner_id" widget="res_partner_many2one" context="{'res_partner_search_mode': 'customer', 'show_vat': True}" readonly="sale_order_id"/>
                            <field name="product_id" readonly="state in ['cancel', 'done']"/>
                            <field name="lot_id" context="{'default_product_id': product_id, 'default_company_id': company_id}" groups="stock.group_production_lot" options="{'no_create': True, 'no_create_edit': True}" invisible="tracking not in ['serial', 'lot']" readonly="state == 'done'" required="tracking in ['serial', 'lot']"/>
                            <field name="product_uom_category_id" invisible="1"/>
                            <label for="product_qty" invisible="not product_id"/>
                            <div class="o_row" invisible="not product_id">
                                <field name="product_qty" readonly="tracking == 'serial' or state in ('done', 'cancel')"/>
                                <field name="product_uom" groups="uom.group_uom" readonly="state != 'draft'"/>
                            </div>
                            <field name="picking_id" options="{'no_create': True}"/>
                            <field name="under_warranty" readonly="state in ['cancel', 'done']"/>
                        </group>
                        <group>
                            <field name="schedule_date" readonly="state in ['done', 'cancel']"/>
                            <field name="user_id" domain="[('share', '=', False)]"/>
                            <field name="company_id" groups="base.group_multi_company" options="{'no_create': True}"/>
                            <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color', 'no_create_edit': True}"/>
                            <field name="parts_availability_state" invisible="True"/>
                            <field name="parts_availability"
                                invisible="state not in ['confirmed', 'under_repair']"
                                decoration-success="parts_availability_state == 'available'"
                                decoration-warning="parts_availability_state == 'expected'"
                                decoration-danger="parts_availability_state == 'late'"/>
                        </group>
                    </group>
                <notebook>
                    <page string="Parts" name="parts">
                        <field name="move_ids" readonly="state == 'cancel' or state == 'done'"
                        context="{'default_repair_id': id, 'default_product_uom_qty': 1, 'default_company_id': company_id, 'default_date': schedule_date, 'default_repair_line_type': 'add'}">
                            <tree string="Operations" editable="bottom">
                                <field name="company_id" column_invisible="True"/>
                                <field name="name" column_invisible="True"/>
                                <field name="state" column_invisible="True"/>
                                <field name="repair_line_type" required="1"/>
                                <field name="picking_type_id" column_invisible="True"/>
                                <field name="location_id" column_invisible="True"/>
                                <field name="location_dest_id" column_invisible="True"/>
                                <field name="partner_id" column_invisible="True" readonly="state == 'done'"/>
                                <field name="scrapped" column_invisible="True"/>
                                <field name="picking_code" column_invisible="True"/>
                                <field name="product_type" column_invisible="True"/>
                                <field name="show_details_visible" column_invisible="True"/>
                                <field name="additional" column_invisible="True"/>
                                <field name="move_lines_count" column_invisible="True"/>
                                <field name="is_locked" column_invisible="True"/>
                                <field name="product_uom_category_id" column_invisible="True"/>
                                <field name="has_tracking" column_invisible="True"/>
                                <field name="display_assign_serial" column_invisible="True"/>
                                <field name="product_id" context="{'default_detailed_type': 'product'}" required="1" readonly="(state != 'draft' and not additional) or move_lines_count &gt; 0"/>
                                <field name="description_picking" string="Description" optional="hide"/>
                                <field name="date" optional="hide"/>
                                <field name="date_deadline" optional="hide"/>
                                <field name="product_packaging_id" groups="product.group_stock_packaging"/>
                                <field name="product_uom_qty" string="Demand" readonly="state in ('done', 'cancel')"/>
                                <field name="forecast_expected_date" column_invisible="True"/>
                                <field name="forecast_availability" string="Forecasted" column_invisible="parent.state in ('draft', 'done')" widget="forecast_widget"/>
                                <field name="product_qty" readonly="1" column_invisible="True"/>
                                <field name="quantity" string="Done" readonly="not product_id"/>
                                <field name="product_uom" readonly="state != 'draft' and not additional" options="{'no_open': True, 'no_create': True}" string="Unit of Measure" groups="uom.group_uom"/>
                                <field name="picked" string="Used"/>
                                <field name="lot_ids" widget="many2many_tags"
                                    groups="stock.group_production_lot"
                                    invisible="not show_details_visible or has_tracking != 'serial'"
                                    optional="hide"
                                    context="{'default_company_id': company_id, 'default_product_id': product_id}"
                                    domain="[('product_id','=',product_id)]"/>
                                <button type="object" name="action_product_forecast_report" title="Forecast Report" icon="fa-area-chart" column_invisible="parent.state != 'draft'" invisible="forecast_availability &lt; 0 and repair_line_type == 'add'"/>
                                <button type="object" name="action_product_forecast_report" title="Forecast Report" icon="fa-area-chart text-danger" column_invisible="parent.state != 'draft'" invisible="forecast_availability &gt;= 0 or repair_line_type != 'add'"/>
                                <button name="action_show_details" type="object" icon="fa-list" width="0.1" title="Details"
                                        invisible="not show_details_visible" options='{"warn": true}'
                                        context="{'default_location_dest_id': location_dest_id}"
                                    />
                            </tree>
                        </field>
                        <div class="clearfix"/>
                    </page>
                    <page string="Repair Notes" name="repair_notes">
                        <field name="internal_notes" placeholder="Add internal notes."/>
                    </page>
                    <page string="Miscellaneous" name="page_miscellaneous">
                        <group>
                            <field name="picking_type_id" options="{'no_create': True}" readonly="state != 'draft'"/>
                        </group>
                        <group string="Locations" groups="stock.group_stock_multi_locations" name="locations">
                            <field name="location_id" readonly="state != 'draft'" options="{'no_create': True}"/>
                            <field name="recycle_location_id" readonly="state != 'draft'" options="{'no_create': True}"/>
                        </group>
                    </page>
                </notebook>
                </sheet>
                <div class="oe_chatter">
                    <field name="message_follower_ids"/>
                    <field name="activity_ids"/>
                    <field name="message_ids"/>
                </div>
            </form>
        </field>
    </record>


    <record id="view_repair_kanban" model="ir.ui.view">
        <field name="name">repair.kanban</field>
        <field name="model">repair.order</field>
        <field name="arch" type="xml">
            <kanban class="o_kanban_mobile" sample="1" quick_create="false">
                <field name="company_id" invisible="1"/>
                <field name="name"/>
                <field name="product_id"/>
                <field name="partner_id"/>
                <field name="state"/>
                <field name="activity_state"/>
                <progressbar field="activity_state" colors='{"planned": "success", "today": "warning", "overdue": "danger"}'/>
                <templates>
                    <t t-name="kanban-box">
                        <div t-attf-class="oe_kanban_card oe_kanban_global_click">
                            <div class="row mb4">
                                <div class="col-6">
                                    <strong><span><t t-esc="record.name.value"/></span></strong>
                                </div>
                                <div class="col-6 text-end">
                                    <field name="state" widget="label_selection" options="{'classes': {'draft': 'info', 'cancel': 'danger', 'done': 'success', 'under_repair': 'secondary'}}"/>
                                </div>
                            </div>
                            <div class="row">
                                <div class="col-6 text-muted">
                                    <span><t t-esc="record.product_id.value"/></span>
                                    <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>
                                </div>
                                <div class="col-6">
                                    <span class="float-end">
                                        <field name="partner_id"/>
                                    </span>
                                </div>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="view_repair_order_form_filter" model="ir.ui.view">
        <field name="name">repair.select</field>
        <field name="model">repair.order</field>
        <field name="arch" type="xml">
            <search string="Search Repair Orders">
                <field name="name" string="Repair Order" filter_domain="['|', ('name', 'ilike', self), ('product_id', 'ilike', self)]"/>
                <field name="product_id"/>
                <field name="partner_id" filter_domain="[('partner_id', 'child_of', self)]"/>
                <field name="sale_order_id"/>
                <filter string="New" domain="[('state', '=', 'draft')]" name="filter_draft" />
                <filter string="Confirmed" domain="[('state', '=', 'confirmed')]" name="filter_confirmed" />
                <filter string="Under Repair" name="filter_under_repair" domain="[('state', '=', 'under_repair')]"/>
                <filter string="Repaired" name="filter_done" domain="[('state', '=', 'done')]"/>
                <filter string="Cancelled" name="filter_cancel" domain="[('state', '=', 'cancel')]"/>
                <filter string="Returned" name="returned" domain="[('picking_id', '!=', False), ('picking_id.state', '=', 'done')]"/>
                <separator/>
                <filter string="Ready" name="ready" domain="[('state', 'in', ('confirmed', 'under_repair')),('is_parts_available', '=', True)]" invisible="True"/>
                <filter string="Late" name="filter_late" domain="[('state', 'in', ('confirmed', 'under_repair')), '|', ('schedule_date', '&lt;', context_today().strftime('%Y-%m-%d')), ('is_parts_late', '=', True)]"/>
                <filter name="filter_create_date" date="create_date"/>
                <separator/>
                <filter invisible="1" string="Late Activities" name="activities_overdue"
                    domain="[('my_activity_date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                    help="Show all records which has next action date is before today"/>
                <filter invisible="1" string="Today Activities" name="activities_today"
                    domain="[('my_activity_date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                    domain="[('my_activity_date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
                <group expand="0" string="Group By">
                    <filter string="Customer" name="partner" domain="[]" context="{'group_by': 'partner_id'}"/>
                    <filter string="Product" name="product" domain="[]" context="{'group_by': 'product_id'}"/>
                    <filter string="Status" name="status" domain="[]" context="{'group_by': 'state'}"/>
                    <filter string="Company" name="company" domain="[]" context="{'group_by': 'company_id'}" groups="base.group_multi_company"/>
                </group>
            </search>
        </field>
    </record>

    <record id="view_repair_graph" model="ir.ui.view">
        <field name="name">repair.graph</field>
        <field name="model">repair.order</field>
        <field name="arch" type="xml">
            <graph string="Repair Orders" sample="1">
                <field name="create_date"/>
                <field name="product_id"/>
            </graph>
        </field>
    </record>

    <record id="view_repair_pivot" model="ir.ui.view">
        <field name="name">repair.pivot</field>
        <field name="model">repair.order</field>
        <field name="arch" type="xml">
            <pivot string="Repair Orders" sample="1">
                <field name="create_date" type="row"/>
                <field name="product_id" type="col"/>
            </pivot>
        </field>
    </record>

    <record id="action_repair_order_form" model="ir.actions.act_window">
        <field name="name">Repair Orders</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">repair.order</field>
        <field name="view_mode">form</field>
    </record>

         <record id="action_repair_order_tree" model="ir.actions.act_window">
            <field name="name">Repair Orders</field>
            <field name="res_model">repair.order</field>
            <field name="view_mode">tree,kanban,graph,pivot,form,activity</field>
            <field name="search_view_id" ref="view_repair_order_form_filter"/>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                No repair order found. Let's create one!
              </p><p>
                In a repair order, you can detail the components you remove,
                add or replace and record the time you spent on the different
                operations.
              </p>
            </field>
        </record>

        <record id="action_repair_order_graph" model="ir.actions.act_window">
            <field name="name">Repair Orders</field>
            <field name="context">{
                'search_default_product': 1,
                'search_default_createDate': 1,
            }
            </field>
            <field name="res_model">repair.order</field>
            <field name="view_mode">tree,kanban,graph,pivot,form</field>
            <field name="view_id" ref="view_repair_graph"/>
        </record>

        <record id="action_picking_repair" model="ir.actions.act_window">
            <field name="name">Repair Orders</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">repair.order</field>
            <field name="view_mode">tree,kanban,form</field>
            <field name="view_id" eval="False"/>
            <field name="search_view_id" ref="view_repair_order_form_filter"/>
            <field name="domain">[('picking_type_id', '=', active_id)]</field>
            <field name="context">{'default_picking_type_id': active_id}</field>
        </record>

        <record id="view_repair_tag_form" model="ir.ui.view">
            <field name="name">repair.tag.form</field>
            <field name="model">repair.tags</field>
            <field name="arch" type="xml">
                <form string="Repair Tags">
                    <sheet>
                        <group>
                            <field name="name"/>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="view_repair_tag_tree" model="ir.ui.view">
            <field name="name">repair.tag.tree</field>
            <field name="model">repair.tags</field>
            <field name="arch" type="xml">
                <tree string="Tags" editable="bottom">
                    <field name="name"/>
                </tree>
            </field>
        </record>

        <record id="view_repair_tag_search" model="ir.ui.view">
            <field name="name">repair.tag.search</field>
            <field name="model">repair.tags</field>
            <field name="arch" type="xml">
                <search string="Tags">
                    <field name="name"/>
                </search>
            </field>
        </record>

        <record id="action_repair_order_tag" model="ir.actions.act_window">
            <field name="name">Tags</field>
            <field name="res_model">repair.tags</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                Create a new tag
              </p>
            </field>
        </record>

        <menuitem id="menu_repair_order" groups="stock.group_stock_user" name="Repairs" sequence="165"
                  web_icon="repair,static/description/icon.png"/>

        <menuitem id="repair_order_menu" name="Orders" action="action_repair_order_tree" groups="stock.group_stock_user"
                  parent="menu_repair_order"/>

        <menuitem id="repair_menu_reporting" name="Reporting" parent="menu_repair_order" groups="stock.group_stock_manager"/>

        <menuitem id="repair_menu" name="Repairs" parent="repair_menu_reporting" action="action_repair_order_graph"/>

        <menuitem id="repair_menu_config" name="Configuration" parent="menu_repair_order" groups="stock.group_stock_manager"/>

        <menuitem id="repair_menu_tag" name="Repair Orders Tags" parent="repair_menu_config" action="action_repair_order_tag"/>
    </data>
</odoo>

```

## File: views\sale_order_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="view_sale_order_form_inherit_repair" model="ir.ui.view">
            <field name="name">sale.order.form.inherit.repair</field>
            <field name="model">sale.order</field>
            <field name="inherit_id" ref="sale.view_order_form"/>
            <field name="arch" type="xml">
                <xpath expr="//button[@name='action_view_invoice']" position="after">
                    <button
                        name="action_show_repair"
                        type="object"
                        class="oe_stat_button"
                        icon="fa-wrench"
                        invisible="repair_count == 0"
                        groups="stock.group_stock_user">
                        <field name="repair_count" widget="statinfo" string="Repairs"/>
                    </button>
                </xpath>
           </field>
        </record>
    </data>
</odoo>

```

## File: views\stock_lot_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
     <record id="stock_production_lot_view_form" model="ir.ui.view">
        <field name="name">stock.production.lot.view.form</field>
        <field name="model">stock.lot</field>
        <field name="inherit_id" ref="stock.view_production_lot_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[hasclass('oe_button_box')]/button" position="before">
                <button class="oe_stat_button" name="action_view_ro" type="object" icon="fa-wrench" help="Repair Orders"
                        invisible="repair_part_count == 0 or not display_complete">
                    <div class="o_field_widget o_stat_info">
                        <div class="o_field_widget o_stat_info align-items-baseline flex-row gap-1 me-1">
                            <span class="o_stat_text">Repair Parts:</span>
                            <span class="o_stat_value">
                                <field name="repair_part_count" widget="statinfo" nolabel="1" class="mr4"/>
                            </span>
                        </div>
                    </div>
                </button>
                <button name="action_lot_open_repairs" icon="fa-wrench" class="oe_stat_button" type="object">
                    <div class="o_field_widget o_stat_info">
                        <div class="o_field_widget o_stat_info align-items-baseline flex-row gap-1 me-1">
                          <span class="o_stat_text">To Do:</span>
                          <span class="o_stat_value">
                              <field name="in_repair_count" widget="statinfo" nolabel="1" class="mr4"/>
                          </span>
                        </div>
                        <div class="o_field_widget o_stat_info align-items-baseline flex-row gap-1 me-1">
                          <span class="o_stat_text">Done:</span>
                          <span class="o_stat_value">
                              <field name="repaired_count" widget="statinfo" nolabel="1" class="mr4"/>
                          </span>
                        </div>
                    </div>
                </button>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\stock_move_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record model="ir.actions.act_window" id="action_repair_move_lines">
        <field name="name">Inventory Moves</field>
        <field name="res_model">stock.move.line</field>
        <field name="view_mode">tree,form</field>
        <field name="domain">[('move_id.repair_id', '=', active_id)]</field>
    </record>

</odoo>

```

## File: views\stock_picking_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="repair_view_picking_type_form" model="ir.ui.view">
        <field name="name">stock.picking.type.inherit.repair</field>
        <field name="model">stock.picking.type</field>
        <field name="inherit_id" ref="stock.view_picking_type_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='default_location_dest_id']" position="after">
                <field name="default_remove_location_dest_id" options="{'no_create': True}" invisible="code != 'repair_operation'" required="code == 'repair_operation'"/>
                <field name="default_recycle_location_dest_id" options="{'no_create': True}" invisible="code != 'repair_operation'" required="code == 'repair_operation'"/>
            </xpath>
            <xpath expr="//group[@name='locations']" position="after">
                <field name="return_type_of_ids" invisible="1"/>
                <group string="Repairs" invisible="not return_type_of_ids">
                    <field name="is_repairable"/>
                </group>
            </xpath>
            <field name="create_backorder" position="attributes">
                <attribute name="invisible">code == 'repair_operation'</attribute>
            </field>
        </field>
    </record>

    <record id="repair_view_picking_form" model="ir.ui.view">
        <field name="name">stock.picking.form.inherit.repair</field>
        <field name="model">stock.picking</field>
        <field name="inherit_id" ref="stock.view_picking_form"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@name='%(stock.act_stock_return_picking)d']" position="after">
                <field name="is_repairable" invisible="1"/>
                <button string="Repair" name="action_repair_return" type="object" invisible="not is_repairable" data-hotkey="shift+k"/>
            </xpath>
            <xpath expr="//button[@name='action_see_packages']" position="after">
                <field name="repair_ids" invisible="1"/>
                <button name="action_view_repairs" type="object"
                        class="oe_stat_button" icon="fa-wrench"
                        invisible="nbr_repairs == 0">
                        <field name="nbr_repairs" string="Repair Orders" widget="statinfo"/>
                </button>
            </xpath>
        </field>
    </record>

    <record id="stock_repair_type_kanban" model="ir.ui.view">
        <field name="name">stock.picking.type.kanban</field>
        <field name="model">stock.picking.type</field>
        <field name="inherit_id" ref="stock.stock_picking_type_kanban"/>
        <field name="arch" type="xml">
            <field name="code" position="after">
                <field name="count_repair_ready"/>
            </field>
            <xpath expr="//t[@t-name='kanban-menu']" position="inside">
                <div class="container" t-if="record.code.raw_value == 'repair_operation'">
                        <div class="row">
                            <div class="col-6 o_kanban_card_manage_section o_kanban_manage_view" name="picking_left_manage_pane">
                                <h5 role="menuitem" class="o_kanban_card_manage_title">
                                    <span>Orders</span>
                                </h5>
                                <div role="menuitem">
                                    <a name="get_repair_stock_picking_action_picking_type" type="object">All</a>
                                </div>
                                <div role="menuitem">
                                    <a name="get_repair_stock_picking_action_picking_type" context="{'search_default_ready': 1}" type="object">Ready</a>
                                </div>
                            </div>
                            <div class="col-6 o_kanban_card_manage_section o_kanban_manage_new">
                                <h5 role="menuitem" class="o_kanban_card_manage_title">
                                    <a name="%(action_repair_order_form)d" type="action" context="{'default_picking_type_id': id}">New</a>
                                </h5>
                            </div>
                        </div>

                        <div t-if="widget.editable" class="o_kanban_card_manage_settings row">
                            <div role="menuitem" aria-haspopup="true" class="col-8">
                                <ul role="menu" class="oe_kanban_colorpicker" data-field="color"/>
                            </div>
                            <div class="col-4">
                                <a class="dropdown-item" role="menuitem" type="edit">Configuration</a>
                            </div>
                        </div>
                    </div>
            </xpath>
            <xpath expr='//div[@name="stock_picking"]' position="after">
                <div t-if="record.code.raw_value == 'repair_operation'" t-attf-class="#{!selection_mode ? kanban_color(record.color.raw_value) : ''}">
                    <div>
                        <div t-attf-class="o_kanban_card_header">
                            <div class="o_kanban_card_header_title">
                                <div class="o_primary" t-if="!selection_mode">
                                    <a type="object" name="get_repair_stock_picking_action_picking_type">
                                        <field name="name"/>
                                    </a>
                                </div>
                                <span class="o_primary" t-if="selection_mode">
                                    <field name="name"/>
                                </span>
                                <div class="o_secondary">
                                    <field class="o_secondary"  name="warehouse_id" readonly="1" groups="stock.group_stock_multi_warehouses"/>
                                </div>
                            </div>
                        </div>
                        <div class="container o_kanban_card_content" t-if="!selection_mode">
                            <div class="row">
                                <div class="col-6 o_kanban_primary_left">
                                    <button class="btn btn-primary" name="get_repair_stock_picking_action_picking_type" context="{'search_default_ready': 1}" type="object">
                                        <span>
                                            <t t-esc="record.count_repair_ready.value"/> To Process
                                        </span>
                                    </button>
                                </div>
                                <div class="col-6 o_kanban_primary_right">
                                    <div t-if="record.count_repair_confirmed.raw_value > 0" class="row">
                                        <div class="col-12">
                                            <a name="get_repair_stock_picking_action_picking_type" context="{'search_default_filter_confirmed': 1}" type="object">
                                                <field name="count_repair_confirmed"/>
                                                Confirmed
                                            </a>
                                        </div>
                                    </div>
                                    <div t-if="record.count_repair_under_repair.raw_value > 0" class="row">
                                        <div class="col-12">
                                            <a name="get_repair_stock_picking_action_picking_type" context="{'search_default_filter_under_repair': 1}" type="object">
                                                <field name="count_repair_under_repair"/>
                                                Under Repair
                                            </a>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\stock_warehouse_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="view_warehouse_inherit_repair" model="ir.ui.view">
            <field name="name">Stock Warehouse Inherit Repair</field>
            <field name="model">stock.warehouse</field>
            <field name="inherit_id" ref="stock.view_warehouse"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='out_type_id']" position="after">
                    <field name="repair_type_id" readonly="True"/>
                </xpath>
            </field>
        </record>
</odoo>

```

## File: wizard\repair_warn_uncomplete_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models

class RepairkWarnUncompleteMove(models.TransientModel):
    _name = 'repair.warn.uncomplete.move'
    _description = 'Warn Uncomplete Move(s)'

    repair_ids = fields.Many2many('repair.order', string='Repair Orders')

    def action_validate(self):
        self.ensure_one()
        return self.repair_ids.action_repair_done()

```

## File: wizard\repair_warn_uncomplete_move.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_repair_warn_uncomplete_move" model="ir.ui.view">
        <field name="name">repair.warn.uncomplete.move.view.form</field>
        <field name="model">repair.warn.uncomplete.move</field>
        <field name="arch" type="xml">
            <form string="Uncomplete Move(s)">
                <p>
                For some of the parts, there is a difference between the initial demand and the actual quantity that was used.<br/>
                Are you sure you want to confirm ?
                </p>
                <footer>
                    <button name="action_validate" string="Validate" type="object" class="btn-primary" />
                    <button string="Discard" special="cancel" data-hotkey="z" class="btn-secondary"/>
                </footer>
            </form>
        </field>
    </record>
</odoo>

```

## File: wizard\stock_warn_insufficient_qty.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models
from odoo.tools.misc import clean_context


class StockWarnInsufficientQtyRepair(models.TransientModel):
    _name = 'stock.warn.insufficient.qty.repair'
    _inherit = 'stock.warn.insufficient.qty'
    _description = 'Warn Insufficient Repair Quantity'

    repair_id = fields.Many2one('repair.order', string='Repair')

    def _get_reference_document_company_id(self):
        return self.repair_id.company_id

    def action_done(self):
        self.ensure_one()
        self = self.with_context(clean_context(self.env.context))
        return self.repair_id._action_repair_confirm()

```

## File: wizard\stock_warn_insufficient_qty_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="stock_warn_insufficient_qty_repair_form_view" model="ir.ui.view">
        <field name="name">stock.warn.insufficient.qty.repair</field>
        <field name="model">stock.warn.insufficient.qty.repair</field>
        <field name="inherit_id" ref="stock.stock_warn_insufficient_qty_form_view"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='description']" position="after">
                Do you confirm you want to repair <strong><field name="quantity" readonly="True"/></strong><field name="product_uom_name" readonly="True" class="mx-1"/>from location <strong><field name="location_id" readonly="True"/></strong>? This may lead to inconsistencies in your inventory.
            </xpath>
        </field>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import stock_warn_insufficient_qty
from . import repair_warn_uncomplete_move

```

