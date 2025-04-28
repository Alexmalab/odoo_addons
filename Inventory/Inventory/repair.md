# Odoo Module: repair

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
    * Invoicing (products and/or services)
    * Warranty concept
    * Repair quotation report
    * Notes for the technician and for the final customer
""",
    'depends': ['stock', 'sale_management', 'account'],
    'data': [
        'security/ir.model.access.csv',
        'security/repair_security.xml',
        'wizard/repair_make_invoice_views.xml',
        'wizard/stock_warn_insufficient_qty_views.xml',
        'views/stock_move_views.xml',
        'views/repair_views.xml',
        'views/stock_lot_views.xml',
        'views/stock_picking_views.xml',
        'report/repair_reports.xml',
        'report/repair_templates_repair_order.xml',
        'data/ir_sequence_data.xml',
        'data/mail_template_data.xml',
    ],
    'demo': ['data/repair_demo.xml'],
    'installable': True,
    'application': True,
    'license': 'LGPL-3',
}

```

## File: data\ir_sequence_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="seq_repair" model="ir.sequence">
            <field name="name">Repair Order</field>
            <field name="code">repair.order</field>
            <field name="prefix">RO/</field>
            <field name="padding">5</field>
            <field name="company_id" eval="False"/>
        </record>
    </data>
</odoo>

```

## File: data\mail_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="mail_template_repair_quotation" model="mail.template">
            <field name="name">Repair: Quotation</field>
            <field name="model_id" ref="repair.model_repair_order"/>
            <field name="subject">{{ object.partner_id.name }} Repair Orders (Ref {{ object.name or 'n/a' }})</field>
            <field name="email_from">{{ (object.create_uid.email_formatted or user.email_formatted) }}</field>
            <field name="partner_to">{{ object.partner_id.id }}</field>
            <field name="description">Sent manually when clicking on "Send Quotation" on a repair order</field>
            <field name="body_html" type="html">
<div style="margin: 0px; padding: 0px;">
    <p style="margin: 0px; padding: 0px;font-size: 13px;">
        Hello <t t-out="object.partner_id.name or ''">Brandon Freeman</t>,<br/>
        Here is your repair order <strong t-out="object.name or ''">RO/00004</strong>
        <t t-if="object.invoice_method != 'none'">
            amounting in <strong><t t-out="format_amount(object.amount_total, object.pricelist_id.currency_id) or ''">$ 100.00</t>.</strong><br/>
        </t>
        <t t-else="">
            .<br/>
        </t>
        You can reply to this email if you have any questions.
        <br/><br/>
        Thank you,
        <t t-if="user.signature">
            <br />
            <t t-out="user.signature or ''">--<br/>Mitchell Admin</t>
        </t>
    </p>
</div></field>
            <field name="report_template" ref="action_report_repair_order"/>
            <field name="report_name">{{ (object.name or '').replace('/','_') }}</field>
            <field name="lang">{{ object.partner_id.lang }}</field>
            <field name="auto_delete" eval="True"/>
        </record>
    </data>
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
    </record>

    <record id="repair_r1" model="repair.order">
        <field name="schedule_date" eval="DateTime.today()"/>
        <field name="address_id" ref="base.res_partner_address_1"/>
        <field name="guarantee_limit" eval="datetime.today().strftime('%Y-%m-%d')"/>
        <field name="invoice_method">none</field>
        <field name="user_id"/>
        <field name="product_id" ref="product.product_product_3"/>
        <field name="product_uom" ref="uom.product_uom_unit"/>
        <field name="partner_invoice_id" ref="base.res_partner_address_1"/>
        <field name="location_id" ref="stock.stock_location_stock"/>
        <field name="operations" model="repair.line" eval="[(5, 0, 0), (0, 0, {
            'location_dest_id': obj().env.ref('product.product_product_11').property_stock_production.id,
            'location_id': obj().env.ref('stock.stock_location_stock').id,
            'name': obj().env.ref('product.product_product_11').get_product_multiline_description_sale(),
            'product_id': obj().env.ref('product.product_product_11').id,
            'product_uom': obj().env.ref('uom.product_uom_unit').id,
            'product_uom_qty': '1.0',
            'price_unit': 50.0,
            'state': 'draft',
            'type': 'add',
            'company_id': obj().env.ref('base.main_company').id,
        })]"/>
        <field name="fees_lines" model="repair.fee" eval="[(5, 0, 0), (0, 0, {
            'name': obj().env.ref('repair.product_service_order_repair').get_product_multiline_description_sale(),
            'product_id': obj().env.ref('repair.product_service_order_repair').id,
            'product_uom_qty': 1.0,
            'product_uom': obj().env.ref('uom.product_uom_unit').id,
            'company_id': obj().env.ref('base.main_company').id,
            'price_unit': 50.0,
        })]"/>
        <field name="partner_id" ref="base.res_partner_12"/>
    </record>

    <record id="repair_r0" model="repair.order">
        <field name="schedule_date" eval="DateTime.today()"/>
        <field name="product_id" ref="product.product_product_5"/>
        <field name="product_uom" ref="uom.product_uom_unit"/>
        <field name="address_id" ref="base.res_partner_address_1"/>
        <field name="guarantee_limit" eval="datetime.today().strftime('%Y-%m-%d')"/>
        <field name="invoice_method">after_repair</field>
        <field name="user_id"/>
        <field name="partner_invoice_id" ref="base.res_partner_address_1"/>
        <field name="location_id" ref="stock.stock_location_stock"/>
        <field name="operations" model="repair.line" eval="[(5, 0, 0), (0, 0, {
            'location_dest_id': obj().env.ref('product.product_product_12').property_stock_production.id,
            'location_id': obj().env.ref('stock.stock_location_stock').id,
            'name': obj().env.ref('product.product_product_12').get_product_multiline_description_sale(),
            'price_unit': 50.0,
            'product_id': obj().env.ref('product.product_product_12').id,
            'product_uom': obj().env.ref('uom.product_uom_unit').id,
            'product_uom_qty': 1.0,
            'state': 'draft',
            'type': 'add',
            'company_id': obj().env.ref('base.main_company').id,
        })]"/>
        <field name="fees_lines" model="repair.fee" eval="[(5, 0, 0), (0, 0, {
            'name': obj().env.ref('repair.product_service_order_repair').get_product_multiline_description_sale(),
            'product_id': obj().env.ref('repair.product_service_order_repair').id,
            'product_uom_qty': 1.0,
            'product_uom': obj().env.ref('uom.product_uom_unit').id,
            'price_unit': 50.0,
            'company_id': obj().env.ref('base.main_company').id,
        })]"/>
        <field name="partner_id" ref="base.res_partner_12"/>
    </record>

    <record id="repair_r2" model="repair.order">
        <field name="priority">1</field>
        <field name="schedule_date" eval="DateTime.today() + relativedelta(days=-5)"/>
        <field name="product_id" ref="product.product_product_6"/>
        <field name="product_uom" ref="uom.product_uom_unit"/>
        <field name="address_id" ref="base.res_partner_address_1"/>
        <field name="guarantee_limit" eval="datetime.today().strftime('%Y-%m-%d')"/>
        <field name="invoice_method">b4repair</field>
        <field name="user_id"/>
        <field name="partner_invoice_id" ref="base.res_partner_address_1"/>
        <field name="location_id" ref="stock.stock_location_14"/>
        <field name="operations" model="repair.line" eval="[(5, 0, 0), (0, 0, {
            'location_dest_id': obj().env.ref('product.product_product_13').property_stock_production.id,
            'location_id': obj().env.ref('stock.stock_location_stock').id,
            'name': obj().env.ref('product.product_product_13').get_product_multiline_description_sale(),
            'price_unit': 50.0,
            'product_id': obj().env.ref('product.product_product_13').id,
            'product_uom': obj().env.ref('uom.product_uom_unit').id,
            'product_uom_qty': 1.0,
            'state': 'draft',
            'type': 'add',
            'company_id': obj().env.ref('base.main_company').id,
        })]"/>
        <field name="fees_lines" model="repair.fee" eval="[(5, 0, 0), (0, 0, {
            'name': obj().env.ref('repair.product_service_order_repair').get_product_multiline_description_sale(),
            'product_id': obj().env.ref('repair.product_service_order_repair').id,
            'product_uom_qty': 1.0,
            'product_uom': obj().env.ref('uom.product_uom_unit').id,
            'price_unit': 50.0,
            'company_id': obj().env.ref('base.main_company').id,
        })]"/>
        <field name="partner_id" ref="base.res_partner_12"/>
    </record>
</odoo>

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-

from odoo import models, fields


class AccountMove(models.Model):
    _inherit = 'account.move'

    repair_ids = fields.One2many('repair.order', 'invoice_id', readonly=True, copy=False)

    def unlink(self):
        repairs = self.sudo().repair_ids.filtered(lambda repair: repair.state != 'cancel')
        if repairs:
            repairs.sudo(False).state = '2binvoiced'
        return super().unlink()


class AccountMoveLine(models.Model):
    _inherit = 'account.move.line'

    repair_line_ids = fields.One2many('repair.line', 'invoice_line_id', readonly=True, copy=False)
    repair_fee_ids = fields.One2many('repair.fee', 'invoice_line_id', readonly=True, copy=False)

    def _stock_account_get_anglo_saxon_price_unit(self):
        price_unit = super()._stock_account_get_anglo_saxon_price_unit()
        ro_line = self.sudo().repair_line_ids
        if ro_line:
            am = ro_line.invoice_line_id.move_id.sudo(False)
            sm = ro_line.move_id.sudo(False)
            price_unit = self._deduce_anglo_saxon_unit_price(am, sm)
        return price_unit

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

from odoo import models


class Product(models.Model):
    _inherit = "product.product"

    def _count_returned_sn_products(self, sn_lot):
        remove_count = self.env['repair.line'].search_count([
            ('type', '=', 'remove'),
            ('product_uom_qty', '=', 1),
            ('lot_id', '=', sn_lot.id),
            ('state', '=', 'done'),
            ('location_dest_id.usage', '=', 'internal'),
        ])
        add_count = self.env['repair.line'].search_count([
            ('type', '=', 'add'),
            ('product_uom_qty', '=', 1),
            ('lot_id', '=', sn_lot.id),
            ('state', '=', 'done'),
            ('location_dest_id.usage', '=', 'production'),
        ])
        return super()._count_returned_sn_products(sn_lot) + (remove_count - add_count)

```

## File: models\repair.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict
from random import randint

from markupsafe import Markup

from odoo import api, fields, models, _
from odoo.exceptions import UserError, ValidationError
from odoo.tools import float_compare, is_html_empty, clean_context


class StockMove(models.Model):
    _inherit = 'stock.move'

    repair_id = fields.Many2one('repair.order', check_company=True)


class Repair(models.Model):
    _name = 'repair.order'
    _description = 'Repair Order'
    _inherit = ['mail.thread', 'mail.activity.mixin']
    _order = 'priority desc, create_date desc'

    name = fields.Char(
        'Repair Reference',
        default='New', index='trigram',
        copy=False, required=True,
        readonly=True)
    description = fields.Char('Repair Description')
    product_id = fields.Many2one(
        'product.product', string='Product to Repair',
        domain="[('type', 'in', ['product', 'consu']), '|', ('company_id', '=', company_id), ('company_id', '=', False)]",
        readonly=True, required=True, states={'draft': [('readonly', False)]}, check_company=True)
    product_qty = fields.Float(
        'Product Quantity',
        default=1.0, digits='Product Unit of Measure',
        readonly=True, required=True, states={'draft': [('readonly', False)]})
    product_uom = fields.Many2one(
        'uom.uom', 'Product Unit of Measure',
        compute='_compute_product_uom', store=True, precompute=True,
        readonly=True, required=True, states={'draft': [('readonly', False)]}, domain="[('category_id', '=', product_uom_category_id)]")
    product_uom_category_id = fields.Many2one(related='product_id.uom_id.category_id')
    partner_id = fields.Many2one(
        'res.partner', 'Customer',
        index=True, states={'confirmed': [('readonly', True)]}, check_company=True, change_default=True,
        help='Choose partner for whom the order will be invoiced and delivered. You can find a partner by its Name, TIN, Email or Internal Reference.')
    address_id = fields.Many2one(
        'res.partner', 'Delivery Address',
        domain="[('parent_id','=',partner_id)]", check_company=True,
        states={'confirmed': [('readonly', True)]})
    default_address_id = fields.Many2one('res.partner', compute='_compute_default_address_id')
    state = fields.Selection([
        ('draft', 'Quotation'),
        ('confirmed', 'Confirmed'),
        ('ready', 'Ready to Repair'),
        ('under_repair', 'Under Repair'),
        ('2binvoiced', 'To be Invoiced'),
        ('done', 'Repaired'),
        ('cancel', 'Cancelled')], string='Status',
        copy=False, default='draft', readonly=True, tracking=True,
        help="* The \'Draft\' status is used when a user is encoding a new and unconfirmed repair order.\n"
             "* The \'Confirmed\' status is used when a user confirms the repair order.\n"
             "* The \'Ready to Repair\' status is used to start to repairing, user can start repairing only after repair order is confirmed.\n"
             "* The \'Under Repair\' status is used when the repair is ongoing.\n"
             "* The \'To be Invoiced\' status is used to generate the invoice before or after repairing done.\n"
             "* The \'Done\' status is set when repairing is completed.\n"
             "* The \'Cancelled\' status is used when user cancel repair order.")
    schedule_date = fields.Date("Scheduled Date")
    location_id = fields.Many2one(
        'stock.location', 'Location',
        compute="_compute_location_id", store=True, precompute=True,
        index=True, readonly=True, required=True, check_company=True,
        help="This is the location where the product to repair is located.",
        states={'draft': [('readonly', False)], 'confirmed': [('readonly', True)]})
    lot_id = fields.Many2one(
        'stock.lot', 'Lot/Serial',
        domain="[('product_id','=', product_id), ('company_id', '=', company_id)]", check_company=True,
        help="Products repaired are all belonging to this lot")
    guarantee_limit = fields.Date('Warranty Expiration', states={'confirmed': [('readonly', True)]})
    operations = fields.One2many(
        'repair.line', 'repair_id', 'Parts',
        copy=True)
    pricelist_id = fields.Many2one(
        'product.pricelist', 'Pricelist',
        default=lambda self: self.env['product.pricelist'].search([('company_id', 'in', [self.env.company.id, False])], limit=1).id,
        help='Pricelist of the selected partner.', check_company=True)
    currency_id = fields.Many2one(related='pricelist_id.currency_id')
    partner_invoice_id = fields.Many2one('res.partner', 'Invoicing Address', check_company=True)
    invoice_method = fields.Selection([
        ("none", "No Invoice"),
        ("b4repair", "Before Repair"),
        ("after_repair", "After Repair")], string="Invoice Method",
        default='none', index=True, readonly=True, required=True,
        states={'draft': [('readonly', False)]},
        help='Selecting \'Before Repair\' or \'After Repair\' will allow you to generate invoice before or after the repair is done respectively. \'No invoice\' means you don\'t want to generate invoice for this repair order.')
    invoice_id = fields.Many2one(
        'account.move', 'Invoice',
        copy=False, readonly=True, tracking=True,
        domain=[('move_type', '=', 'out_invoice')])
    move_id = fields.Many2one(
        'stock.move', 'Inventory Move',
        copy=False, readonly=True, tracking=True, check_company=True)
    fees_lines = fields.One2many(
        'repair.fee', 'repair_id', 'Operations',
        copy=True, readonly=False)
    internal_notes = fields.Html('Internal Notes')
    quotation_notes = fields.Html('Quotation Notes')
    user_id = fields.Many2one('res.users', string="Responsible", default=lambda self: self.env.user, check_company=True)
    company_id = fields.Many2one(
        'res.company', 'Company',
        readonly=True, required=True, index=True,
        default=lambda self: self.env.company)
    sale_order_id = fields.Many2one(
        'sale.order', 'Sale Order', check_company=True,
        copy=False, help="Sale Order from which the product to be repaired comes from.")
    picking_id = fields.Many2one(
        'stock.picking', 'Return', check_company=True,
        copy=False, help="Return Order from which the product to be repaired comes from.")
    allowed_picking_type_ids = fields.Many2many('stock.picking.type', compute='_compute_allowed_picking_type_ids')
    is_returned = fields.Boolean(
        "Returned", compute='_compute_is_returned',
        help="True if this repair is linked to a Return Order and the order is 'Done'. False otherwise.")
    tag_ids = fields.Many2many('repair.tags', string="Tags")
    invoiced = fields.Boolean('Invoiced', copy=False, readonly=True)
    repaired = fields.Boolean('Repaired', copy=False, readonly=True)
    amount_untaxed = fields.Float('Untaxed Amount', compute='_amount_untaxed', store=True)
    amount_tax = fields.Float('Taxes', compute='_amount_tax', store=True)
    amount_total = fields.Float('Total', compute='_amount_total', store=True)
    tracking = fields.Selection(string='Product Tracking', related="product_id.tracking", readonly=False)
    invoice_state = fields.Selection(string='Invoice State', related='invoice_id.state')
    priority = fields.Selection([('0', 'Normal'), ('1', 'Urgent')], default='0', string="Priority")

    @api.depends('product_id')
    def _compute_allowed_picking_type_ids(self):
        '''
            computes the ids of return picking types
        '''
        out_picking_types = self.env['stock.picking.type'].search_read(domain=[('code', '=', 'outgoing')],
                                                                          fields=['return_picking_type_id'], load='')
        self.allowed_picking_type_ids = [
            pt['return_picking_type_id'] for pt in out_picking_types if pt['return_picking_type_id']]

    @api.depends('partner_id')
    def _compute_default_address_id(self):
        for order in self:
            if order.partner_id:
                order.default_address_id = order.partner_id.address_get(['contact'])['contact']

    @api.depends('picking_id', 'picking_id.state')
    def _compute_is_returned(self):
        self.is_returned = False
        returned = self.filtered(lambda r: r.picking_id and r.picking_id.state == 'done')
        returned.is_returned = True

    @api.depends('operations.price_subtotal', 'invoice_method', 'fees_lines.price_subtotal', 'pricelist_id.currency_id')
    def _amount_untaxed(self):
        for order in self:
            total = sum(operation.price_subtotal for operation in order.operations)
            total += sum(fee.price_subtotal for fee in order.fees_lines)
            currency = order.pricelist_id.currency_id or self.env.company.currency_id
            order.amount_untaxed = currency.round(total)

    @api.depends('operations.price_unit', 'operations.product_uom_qty', 'operations.product_id',
                 'fees_lines.price_unit', 'fees_lines.product_uom_qty', 'fees_lines.product_id',
                 'pricelist_id.currency_id', 'partner_id')
    def _amount_tax(self):
        for order in self:
            val = 0.0
            currency = order.pricelist_id.currency_id or self.env.company.currency_id
            for operation in order.operations:
                if operation.tax_id:
                    tax_calculate = operation.tax_id.compute_all(operation.price_unit, currency, operation.product_uom_qty, operation.product_id, order.partner_id)
                    for c in tax_calculate['taxes']:
                        val += c['amount']
            for fee in order.fees_lines:
                if fee.tax_id:
                    tax_calculate = fee.tax_id.compute_all(fee.price_unit, currency, fee.product_uom_qty, fee.product_id, order.partner_id)
                    for c in tax_calculate['taxes']:
                        val += c['amount']
            order.amount_tax = val

    @api.depends('amount_untaxed', 'amount_tax')
    def _amount_total(self):
        for order in self:
            currency = order.pricelist_id.currency_id or self.env.company.currency_id
            order.amount_total = currency.round(order.amount_untaxed + order.amount_tax)

    _sql_constraints = [
        ('name', 'unique (name)', 'The name of the Repair Order must be unique!'),
    ]

    @api.onchange('location_id', 'picking_id')
    def _onchange_location_picking(self):
        location_warehouse = self.location_id.warehouse_id
        picking_warehouse = self.picking_id.location_dest_id.warehouse_id
        if location_warehouse and picking_warehouse and location_warehouse != picking_warehouse:
            return {
                'warning': {'title': _("Warning"), 'message': _("Note that the warehouses of the return and repair locations don't match!")},
            }

    @api.depends('product_id')
    def _compute_product_uom(self):
        for repair in self:
            if repair.product_id:
                repair.product_uom = repair.product_id.uom_id

    @api.onchange('product_id')
    def onchange_product_id(self):
        self.guarantee_limit = False
        if (self.product_id and self.lot_id and self.lot_id.product_id != self.product_id) or not self.product_id:
            self.lot_id = False

    @api.onchange('product_uom')
    def onchange_product_uom(self):
        res = {}
        if not self.product_id or not self.product_uom:
            return res
        if self.product_uom.category_id != self.product_id.uom_id.category_id:
            res['warning'] = {'title': _('Warning'), 'message': _('The product unit of measure you chose has a different category than the product unit of measure.')}
            self.product_uom = self.product_id.uom_id.id
        return res

    @api.onchange('partner_id')
    def onchange_partner_id(self):
        self = self.with_company(self.company_id)
        if not self.partner_id:
            self.address_id = False
            self.partner_invoice_id = False
            self.pricelist_id = self.env['product.pricelist'].search([
                ('company_id', 'in', [self.env.company.id, False]),
            ], limit=1)
        else:
            addresses = self.partner_id.address_get(['delivery', 'invoice', 'contact'])
            self.address_id = addresses['delivery'] or addresses['contact']
            self.partner_invoice_id = addresses['invoice']
            self.pricelist_id = self.partner_id.property_product_pricelist.id

    @api.depends('company_id')
    def _compute_location_id(self):
        for order in self:
            if order.company_id:
                if order.location_id.company_id != order.company_id:
                    warehouse = self.env['stock.warehouse'].search([('company_id', '=', order.company_id.id)], limit=1)
                    order.location_id = warehouse.lot_stock_id
            else:
                order.location_id = False

    @api.ondelete(at_uninstall=False)
    def _unlink_except_confirmed(self):
        for order in self:
            if order.invoice_id and order.invoice_id.posted_before:
                raise UserError(_('You can not delete a repair order which is linked to an invoice which has been posted once.'))
            if order.state == 'done':
                raise UserError(_('You cannot delete a completed repair order.'))
            if order.state not in ('draft', 'cancel'):
                raise UserError(_('You can not delete a repair order once it has been confirmed. You must first cancel it.'))

    @api.model_create_multi
    def create(self, vals_list):
        # We generate a standard reference
        for vals in vals_list:
            vals['name'] = self.env['ir.sequence'].next_by_code('repair.order') or '/'
        return super().create(vals_list)

    def button_dummy(self):
        # TDE FIXME: this button is very interesting
        return True

    def action_repair_cancel_draft(self):
        if self.filtered(lambda repair: repair.state != 'cancel'):
            raise UserError(_("Repair must be canceled in order to reset it to draft."))
        self.mapped('operations').write({'state': 'draft'})
        return self.write({'state': 'draft', 'invoice_id': False})

    def action_validate(self):
        self.ensure_one()
        if self.filtered(lambda repair: any(op.product_uom_qty < 0 for op in repair.operations)):
            raise UserError(_("You can not enter negative quantities."))
        if self.product_id.type == 'consu':
            return self.action_repair_confirm()
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
                return self.action_repair_confirm()
        else:
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

    def action_repair_confirm(self):
        """ Repair order state is set to 'To be invoiced' when invoice method
        is 'Before repair' else state becomes 'Confirmed'.
        @param *arg: Arguments
        @return: True
        """
        if self.filtered(lambda repair: repair.state != 'draft'):
            raise UserError(_("Only draft repairs can be confirmed."))
        self._check_company()
        self.operations._check_company()
        self.fees_lines._check_company()
        before_repair = self.filtered(lambda repair: repair.invoice_method == 'b4repair')
        before_repair.write({'state': '2binvoiced'})
        to_confirm = self - before_repair
        to_confirm_operations = to_confirm.mapped('operations')
        to_confirm_operations.write({'state': 'confirmed'})
        to_confirm.write({'state': 'confirmed'})
        return True

    def action_repair_cancel(self):
        if any(repair.state == 'done' for repair in self):
            raise UserError(_("You cannot cancel a completed repair order."))
        invoice_to_cancel = self.filtered(lambda repair: repair.invoice_id.state == 'draft').invoice_id
        if invoice_to_cancel:
            invoice_to_cancel.button_cancel()
        self.mapped('operations').write({'state': 'cancel'})
        return self.write({'state': 'cancel'})

    def action_send_mail(self):
        self.ensure_one()
        template_id = self.env.ref('repair.mail_template_repair_quotation').id
        ctx = {
            'default_model': 'repair.order',
            'default_res_id': self.id,
            'default_use_template': bool(template_id),
            'default_template_id': template_id,
            'default_composition_mode': 'comment',
            'default_email_layout_xmlid': 'mail.mail_notification_light',
        }
        return {
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'mail.compose.message',
            'target': 'new',
            'context': ctx,
        }

    def print_repair_order(self):
        return self.env.ref('repair.action_report_repair_order').report_action(self)

    def action_repair_invoice_create(self):
        for repair in self:
            repair._create_invoices()
            if repair.invoice_method == 'b4repair':
                repair.action_repair_ready()
            elif repair.invoice_method == 'after_repair':
                repair.write({'state': 'done'})
        return True

    def _create_invoices(self, group=False):
        """ Creates invoice(s) for repair order.
        @param group: It is set to true when group invoice is to be generated.
        @return: Invoice Ids.
        """
        grouped_invoices_vals = {}
        repairs = self.filtered(lambda repair: repair.state not in ('draft', 'cancel')
                                               and not repair.invoice_id
                                               and repair.invoice_method != 'none')
        for repair in repairs:
            repair = repair.with_company(repair.company_id)
            partner_invoice = repair.partner_invoice_id or repair.partner_id
            if not partner_invoice:
                raise UserError(_('You have to select an invoice address in the repair form.'))

            narration = repair.quotation_notes
            currency = repair.pricelist_id.currency_id
            company = repair.env.company

            if (partner_invoice.id, currency.id, company.id) not in grouped_invoices_vals:
                grouped_invoices_vals[(partner_invoice.id, currency.id, company.id)] = []
            current_invoices_list = grouped_invoices_vals[(partner_invoice.id, currency.id, company.id)]

            if not group or len(current_invoices_list) == 0:
                fpos = self.env['account.fiscal.position']._get_fiscal_position(partner_invoice, delivery=repair.address_id)
                invoice_vals = {
                    'move_type': 'out_invoice',
                    'partner_id': partner_invoice.id,
                    'partner_shipping_id': repair.address_id.id,
                    'currency_id': currency.id,
                    'narration': narration if not is_html_empty(narration) else '',
                    'invoice_origin': repair.name,
                    'repair_ids': [(4, repair.id)],
                    'invoice_line_ids': [],
                    'fiscal_position_id': fpos.id
                }
                if partner_invoice.property_payment_term_id:
                    invoice_vals['invoice_payment_term_id'] = partner_invoice.property_payment_term_id.id
                current_invoices_list.append(invoice_vals)
            else:
                # if group == True: concatenate invoices by partner and currency
                invoice_vals = current_invoices_list[0]
                invoice_vals['invoice_origin'] += ', ' + repair.name
                invoice_vals['repair_ids'].append((4, repair.id))
                if not is_html_empty(narration):
                    if  is_html_empty(invoice_vals['narration']):
                        invoice_vals['narration'] = narration
                    else:
                        invoice_vals['narration'] += Markup('<br/>') + narration

            # Create invoice lines from operations.
            for operation in repair.operations.filtered(lambda op: op.type == 'add'):
                if group:
                    name = repair.name + '-' + operation.name
                else:
                    name = operation.name

                account = operation.product_id.product_tmpl_id.get_product_accounts(fiscal_pos=fpos)['income']
                if not account:
                    raise UserError(_('No account defined for product "%s".', operation.product_id.name))

                invoice_line_vals = {
                    'name': name,
                    'account_id': account.id,
                    'quantity': operation.product_uom_qty,
                    'tax_ids': [(6, 0, operation.tax_id.ids)],
                    'product_uom_id': operation.product_uom.id,
                    'price_unit': operation.price_unit,
                    'product_id': operation.product_id.id,
                    'repair_line_ids': [(4, operation.id)],
                }

                if currency == company.currency_id:
                    balance = -(operation.product_uom_qty * operation.price_unit)
                    invoice_line_vals.update({
                        'debit': balance > 0.0 and balance or 0.0,
                        'credit': balance < 0.0 and -balance or 0.0,
                    })
                else:
                    amount_currency = -(operation.product_uom_qty * operation.price_unit)
                    balance = currency._convert(amount_currency, company.currency_id, company, fields.Date.today())
                    invoice_line_vals.update({
                        'amount_currency': amount_currency,
                        'debit': balance > 0.0 and balance or 0.0,
                        'credit': balance < 0.0 and -balance or 0.0,
                        'currency_id': currency.id,
                    })
                invoice_vals['invoice_line_ids'].append((0, 0, invoice_line_vals))

            # Create invoice lines from fees.
            for fee in repair.fees_lines:
                if group:
                    name = repair.name + '-' + fee.name
                else:
                    name = fee.name

                if not fee.product_id:
                    raise UserError(_('No product defined on fees.'))

                account = fee.product_id.product_tmpl_id.get_product_accounts(fiscal_pos=fpos)['income']
                if not account:
                    raise UserError(_('No account defined for product "%s".', fee.product_id.name))

                invoice_line_vals = {
                    'name': name,
                    'account_id': account.id,
                    'quantity': fee.product_uom_qty,
                    'tax_ids': [(6, 0, fee.tax_id.ids)],
                    'product_uom_id': fee.product_uom.id,
                    'price_unit': fee.price_unit,
                    'product_id': fee.product_id.id,
                    'repair_fee_ids': [(4, fee.id)],
                }

                if currency == company.currency_id:
                    balance = -(fee.product_uom_qty * fee.price_unit)
                    invoice_line_vals.update({
                        'debit': balance > 0.0 and balance or 0.0,
                        'credit': balance < 0.0 and -balance or 0.0,
                    })
                else:
                    amount_currency = -(fee.product_uom_qty * fee.price_unit)
                    balance = currency._convert(amount_currency, company.currency_id, company,
                                                fields.Date.today())
                    invoice_line_vals.update({
                        'amount_currency': amount_currency,
                        'debit': balance > 0.0 and balance or 0.0,
                        'credit': balance < 0.0 and -balance or 0.0,
                        'currency_id': currency.id,
                    })
                invoice_vals['invoice_line_ids'].append((0, 0, invoice_line_vals))

        # Create invoices.
        invoices_vals_list_per_company = defaultdict(list)
        for (partner_invoice_id, currency_id, company_id), invoices in grouped_invoices_vals.items():
            for invoice in invoices:
                invoices_vals_list_per_company[company_id].append(invoice)

        for company_id, invoices_vals_list in invoices_vals_list_per_company.items():
            # VFE TODO remove the default_company_id ctxt key ?
            # Account fallbacks on self.env.company, which is correct with with_company
            self.env['account.move'].with_company(company_id).with_context(default_company_id=company_id, default_move_type='out_invoice').create(invoices_vals_list)

        repairs.write({'invoiced': True})
        repairs.mapped('operations').filtered(lambda op: op.type == 'add').write({'invoiced': True})
        repairs.mapped('fees_lines').write({'invoiced': True})

        return dict((repair.id, repair.invoice_id.id) for repair in repairs)

    def action_created_invoice(self):
        self.ensure_one()
        return {
            'name': _('Invoice created'),
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'account.move',
            'view_id': self.env.ref('account.view_move_form').id,
            'target': 'current',
            'res_id': self.invoice_id.id,
            }

    def action_repair_ready(self):
        self.mapped('operations').write({'state': 'confirmed'})
        return self.write({'state': 'ready'})

    def action_repair_start(self):
        """ Writes repair order state to 'Under Repair'
        @return: True
        """
        if self.filtered(lambda repair: repair.state not in ['confirmed', 'ready']):
            raise UserError(_("Repair must be confirmed before starting reparation."))
        self.mapped('operations').write({'state': 'confirmed'})
        return self.write({'state': 'under_repair'})

    def action_repair_end(self):
        """ Writes repair order state to 'To be invoiced' if invoice method is
        After repair else state is set to 'Ready'.
        @return: True
        """
        if self.filtered(lambda repair: repair.state != 'under_repair'):
            raise UserError(_("Repair must be under repair in order to end reparation."))
        self._check_product_tracking()
        for repair in self:
            repair.write({'repaired': True})
            vals = {'state': 'done'}
            vals['move_id'] = repair.action_repair_done().get(repair.id)
            if not repair.invoice_id and repair.invoice_method == 'after_repair':
                vals['state'] = '2binvoiced'
            repair.write(vals)
        return True

    def action_repair_done(self):
        """ Creates stock move for operation and stock move for final product of repair order.
        @return: Move ids of final products

        """
        if self.filtered(lambda repair: not repair.repaired):
            raise UserError(_("Repair must be repaired in order to make the product moves."))
        self._check_company()
        self.operations._check_company()
        self.fees_lines._check_company()
        # Clean the context to get rid of residual default_* keys that could cause issues
        # during the creation of stock.move.
        self = self.with_context(clean_context(self._context))
        res = {}
        precision = self.env['decimal.precision'].precision_get('Product Unit of Measure')
        Move = self.env['stock.move']
        for repair in self:
            # Try to create move with the appropriate owner
            owner_id = False
            available_qty_owner = self.env['stock.quant']._get_available_quantity(repair.product_id, repair.location_id, repair.lot_id, owner_id=repair.partner_id, strict=True)
            if float_compare(available_qty_owner, repair.product_qty, precision_digits=precision) >= 0:
                owner_id = repair.partner_id.id

            moves = self.env['stock.move']
            for operation in repair.operations:
                move = Move.create({
                    'name': repair.name,
                    'product_id': operation.product_id.id,
                    'product_uom_qty': operation.product_uom_qty,
                    'product_uom': operation.product_uom.id,
                    'partner_id': repair.address_id.id,
                    'location_id': operation.location_id.id,
                    'location_dest_id': operation.location_dest_id.id,
                    'repair_id': repair.id,
                    'origin': repair.name,
                    'company_id': repair.company_id.id,
                })

                # Best effort to reserve the product in a (sub)-location where it is available
                product_qty = move.product_uom._compute_quantity(
                    operation.product_uom_qty, move.product_id.uom_id, rounding_method='HALF-UP')
                available_quantity = self.env['stock.quant']._get_available_quantity(
                    move.product_id,
                    move.location_id,
                    lot_id=operation.lot_id,
                    strict=False,
                )
                move._update_reserved_quantity(
                    product_qty,
                    available_quantity,
                    move.location_id,
                    lot_id=operation.lot_id,
                    strict=False,
                )
                # Then, set the quantity done. If the required quantity was not reserved, negative
                # quant is created in operation.location_id.
                move._set_quantity_done(operation.product_uom_qty)

                if operation.lot_id:
                    move.move_line_ids.lot_id = operation.lot_id

                moves |= move
                operation.write({'move_id': move.id, 'state': 'done'})
            move = Move.create({
                'name': repair.name,
                'product_id': repair.product_id.id,
                'product_uom': repair.product_uom.id or repair.product_id.uom_id.id,
                'product_uom_qty': repair.product_qty,
                'partner_id': repair.address_id.id,
                'location_id': repair.location_id.id,
                'location_dest_id': repair.location_id.id,
                'move_line_ids': [(0, 0, {'product_id': repair.product_id.id,
                                           'lot_id': repair.lot_id.id,
                                           'reserved_uom_qty': 0,  # bypass reservation here
                                           'product_uom_id': repair.product_uom.id or repair.product_id.uom_id.id,
                                           'qty_done': repair.product_qty,
                                           'package_id': False,
                                           'result_package_id': False,
                                           'owner_id': owner_id,
                                           'location_id': repair.location_id.id, #TODO: owner stuff
                                           'company_id': repair.company_id.id,
                                           'location_dest_id': repair.location_id.id,})],
                'repair_id': repair.id,
                'origin': repair.name,
                'company_id': repair.company_id.id,
            })
            consumed_lines = moves.mapped('move_line_ids')
            produced_lines = move.move_line_ids
            moves |= move
            moves._action_done()
            produced_lines.write({'consume_line_ids': [(6, 0, consumed_lines.ids)]})
            res[repair.id] = move.id
        return res

    def _check_product_tracking(self):
        invalid_lines = self.operations.filtered(lambda x: x.tracking != 'none' and not x.lot_id)
        if invalid_lines:
            products = invalid_lines.product_id
            raise ValidationError(_(
                "Serial number is required for operation lines with products: %s",
                ", ".join(products.mapped('display_name')),
            ))


class RepairLine(models.Model):
    _name = 'repair.line'
    _description = 'Repair Line (parts)'

    name = fields.Text('Description', required=True)
    repair_id = fields.Many2one(
        'repair.order', 'Repair Order Reference', required=True,
        index=True, ondelete='cascade', check_company=True)
    company_id = fields.Many2one(
        related='repair_id.company_id', store=True, index=True)
    currency_id = fields.Many2one(
        related='repair_id.currency_id')
    type = fields.Selection([
        ('add', 'Add'),
        ('remove', 'Remove')], 'Type', default='add', required=True)
    product_id = fields.Many2one(
        'product.product', 'Product', required=True, check_company=True,
        domain="[('type', 'in', ['product', 'consu']), '|', ('company_id', '=', company_id), ('company_id', '=', False)]")
    invoiced = fields.Boolean('Invoiced', copy=False, readonly=True)
    price_unit = fields.Float('Unit Price', required=True, digits='Product Price')
    price_subtotal = fields.Float('Subtotal', compute='_compute_price_total_and_subtotal', store=True, digits=0)
    price_total = fields.Float('Total', compute='_compute_price_total_and_subtotal', store=True, digits=0)
    tax_id = fields.Many2many(
        'account.tax', 'repair_operation_line_tax', 'repair_operation_line_id', 'tax_id', 'Taxes',
        domain="[('type_tax_use','=','sale'), ('company_id', '=', company_id)]", check_company=True)
    product_uom_qty = fields.Float(
        'Quantity', default=1.0,
        digits='Product Unit of Measure', required=True)
    product_uom = fields.Many2one(
        'uom.uom', 'Product Unit of Measure',
        compute='_compute_product_uom', store=True, readonly=False, precompute=True,
        required=True, domain="[('category_id', '=', product_uom_category_id)]")
    product_uom_category_id = fields.Many2one(related='product_id.uom_id.category_id')
    invoice_line_id = fields.Many2one(
        'account.move.line', 'Invoice Line',
        copy=False, readonly=True, check_company=True)
    location_id = fields.Many2one(
        'stock.location', 'Source Location',
        compute='_compute_location_id', store=True, readonly=False, precompute=True,
        index=True, required=True, check_company=True)
    location_dest_id = fields.Many2one(
        'stock.location', 'Dest. Location',
        compute='_compute_location_id', store=True, readonly=False, precompute=True,
        index=True, required=True, check_company=True)
    move_id = fields.Many2one(
        'stock.move', 'Inventory Move',
        copy=False, readonly=True)
    lot_id = fields.Many2one(
        'stock.lot', 'Lot/Serial',
        domain="[('product_id','=', product_id), ('company_id', '=', company_id)]", check_company=True)
    state = fields.Selection([
        ('draft', 'Draft'),
        ('confirmed', 'Confirmed'),
        ('done', 'Done'),
        ('cancel', 'Cancelled')], 'Status', default='draft',
        copy=False, readonly=True, required=True,
        help='The status of a repair line is set automatically to the one of the linked repair order.')
    tracking = fields.Selection(string='Product Tracking', related="product_id.tracking")

    @api.depends('price_unit', 'repair_id', 'product_uom_qty', 'product_id', 'tax_id', 'repair_id.invoice_method')
    def _compute_price_total_and_subtotal(self):
        for line in self:
            taxes = line.tax_id.compute_all(line.price_unit, line.repair_id.pricelist_id.currency_id, line.product_uom_qty, line.product_id, line.repair_id.partner_id)
            line.price_subtotal = taxes['total_excluded']
            line.price_total = taxes['total_included']

    @api.depends('product_id')
    def _compute_product_uom(self):
        for line in self:
            line.product_uom = line.product_id.uom_id.id

    @api.depends('type')
    def _compute_location_id(self):
        for line in self:
            if not line.type:
                line.location_id = False
                line.location_dest_id = False
            elif line.type == 'add':
                args = line.repair_id.company_id and [('company_id', '=', line.repair_id.company_id.id)] or []
                warehouse = line.env['stock.warehouse'].search(args, limit=1)
                line.location_id = warehouse.lot_stock_id
                line.location_dest_id = line.env['stock.location'].search([('usage', '=', 'production'), ('company_id', '=', line.repair_id.company_id.id)], limit=1)
            else:
                line.location_id = line.env['stock.location'].search([('usage', '=', 'production'), ('company_id', '=', line.repair_id.company_id.id)], limit=1).id
                line.location_dest_id = line.env['stock.location'].search([('scrap_location', '=', True), ('company_id', 'in', [line.repair_id.company_id.id, False])], limit=1).id

    @api.onchange('type')
    def onchange_operation_type(self):
        """ On change of operation type it sets source location, destination location
        and to invoice field.
        @param product: Changed operation type.
        @param guarantee_limit: Guarantee limit of current record.
        @return: Dictionary of values.
        """
        if not self.type:
            pass
        elif self.type == 'add':
            self.onchange_product_id()
        else:
            self.price_unit = 0.0
            self.tax_id = False

    @api.onchange('repair_id', 'product_id', 'product_uom_qty')
    def onchange_product_id(self):
        """ On change of product it sets product quantity, tax account, name,
        uom of product, unit price and price subtotal. """
        if not self.product_id or not self.product_uom_qty:
            return
        self = self.with_company(self.company_id)
        partner = self.repair_id.partner_id
        partner_invoice = self.repair_id.partner_invoice_id or partner
        if partner:
            self = self.with_context(lang=partner.lang)
        product = self.product_id
        self.name = product.display_name
        if product.description_sale:
            if partner:
                self.name += '\n' + self.product_id.with_context(lang=partner.lang).description_sale
            else:
                self.name += '\n' + self.product_id.description_sale
        if self.type != 'remove':
            if partner:
                fpos = self.env['account.fiscal.position']._get_fiscal_position(partner_invoice, delivery=self.repair_id.address_id)
                taxes = self.product_id.taxes_id.filtered(lambda x: x.company_id == self.repair_id.company_id)
                self.tax_id = fpos.map_tax(taxes)
            warning = False
            pricelist = self.repair_id.pricelist_id
            if not pricelist:
                warning = {
                    'title': _('No pricelist found.'),
                    'message':
                        _('You have to select a pricelist in the Repair form !\n Please set one before choosing a product.')}
                return {'warning': warning}
            else:
                self._onchange_product_uom()

    @api.onchange('product_uom')
    def _onchange_product_uom(self):
        pricelist = self.repair_id.pricelist_id
        if pricelist and self.product_id and self.type != 'remove':
            price = pricelist._get_product_price(self.product_id, self.product_uom_qty, uom=self.product_uom)
            if price is False:
                warning = {
                    'title': _('No valid pricelist line found.'),
                    'message':
                        _("Couldn't find a pricelist line matching this product and quantity.\nYou have to change either the product, the quantity or the pricelist.")}
                return {'warning': warning}
            else:
                self.price_unit = price


class RepairFee(models.Model):
    _name = 'repair.fee'
    _description = 'Repair Fees'

    repair_id = fields.Many2one(
        'repair.order', 'Repair Order Reference',
        index=True, ondelete='cascade', required=True)
    company_id = fields.Many2one(
        related="repair_id.company_id", index=True, store=True)
    currency_id = fields.Many2one(
        related="repair_id.currency_id")
    name = fields.Text('Description', index=True, required=True)
    product_id = fields.Many2one(
        'product.product', 'Product', check_company=True,
        domain="[('type', '=', 'service'), '|', ('company_id', '=', company_id), ('company_id', '=', False)]")
    product_uom_qty = fields.Float('Quantity', digits='Product Unit of Measure', required=True, default=1.0)
    price_unit = fields.Float('Unit Price', required=True, digits='Product Price')
    product_uom = fields.Many2one('uom.uom', 'Product Unit of Measure', required=True, domain="[('category_id', '=', product_uom_category_id)]")
    product_uom_category_id = fields.Many2one(related='product_id.uom_id.category_id')
    price_subtotal = fields.Float('Subtotal', compute='_compute_price_total_and_subtotal', store=True, digits=0)
    price_total = fields.Float('Total', compute='_compute_price_total_and_subtotal', store=True, digits=0)
    tax_id = fields.Many2many(
        'account.tax', 'repair_fee_line_tax', 'repair_fee_line_id', 'tax_id', 'Taxes',
        domain="[('type_tax_use','=','sale'), ('company_id', '=', company_id)]", check_company=True)
    invoice_line_id = fields.Many2one('account.move.line', 'Invoice Line', copy=False, readonly=True, check_company=True)
    invoiced = fields.Boolean('Invoiced', copy=False, readonly=True)

    @api.depends('price_unit', 'repair_id', 'product_uom_qty', 'product_id', 'tax_id')
    def _compute_price_total_and_subtotal(self):
        for fee in self:
            taxes = fee.tax_id.compute_all(fee.price_unit, fee.repair_id.pricelist_id.currency_id, fee.product_uom_qty, fee.product_id, fee.repair_id.partner_id)
            fee.price_subtotal = taxes['total_excluded']
            fee.price_total = taxes['total_included']

    @api.onchange('repair_id', 'product_id', 'product_uom_qty')
    def onchange_product_id(self):
        """ On change of product it sets product quantity, tax account, name,
        uom of product, unit price and price subtotal. """
        if not self.product_id:
            return

        self = self.with_company(self.company_id)

        partner = self.repair_id.partner_id
        partner_invoice = self.repair_id.partner_invoice_id or partner
        pricelist = self.repair_id.pricelist_id

        if partner and self.product_id:
            fpos = self.env['account.fiscal.position']._get_fiscal_position(partner_invoice, delivery=self.repair_id.address_id)
            taxes = self.product_id.taxes_id.filtered(lambda x: x.company_id == self.repair_id.company_id)
            self.tax_id = fpos.map_tax(taxes)
        if partner:
            self.name = self.product_id.with_context(lang=partner.lang).display_name
        else:
            self.name = self.product_id.display_name
        self.product_uom = self.product_id.uom_id.id
        if self.product_id.description_sale:
            if partner:
                self.name += '\n' + self.product_id.with_context(lang=partner.lang).description_sale
            else:
                self.name += '\n' + self.product_id.description_sale

        warning = False
        if not pricelist:
            warning = {
                'title': _('No pricelist found.'),
                'message':
                    _('You have to select a pricelist in the Repair form !\n Please set one before choosing a product.')}
            return {'warning': warning}
        else:
            self._onchange_product_uom()

    @api.onchange('product_uom')
    def _onchange_product_uom(self):
        pricelist = self.repair_id.pricelist_id
        if pricelist and self.product_id:
            price = pricelist._get_product_price(self.product_id, self.product_uom_qty, uom=self.product_uom)
            if price is False:
                warning = {
                    'title': _('No valid pricelist line found.'),
                    'message':
                        _("Couldn't find a pricelist line matching this product and quantity.\nYou have to change either the product, the quantity or the pricelist.")}
                return {'warning': warning}
            else:
                self.price_unit = price

    # TODO: replace with computes in master
    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            if not vals.get('product_uom'):
                vals['product_uom'] = self.env["product.product"].browse(vals.get('product_id')).uom_id.id
        return super().create(vals_list)

    # TODO: replace with computes in master
    def write(self, vals):
        if vals.get('product_id') and not vals.get('product_uom'):
            vals['product_uom'] = self.env["product.product"].browse(vals.get('product_id')).uom_id.id
        return super().write(vals)


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

## File: models\stock_lot.py

```python
# -*- coding: utf-8 -*-

from collections import defaultdict
from odoo import api, fields, models, _

class StockLot(models.Model):
    _inherit = 'stock.lot'

    repair_order_ids = fields.Many2many('repair.order', string="Repair Orders", compute="_compute_repair_order_ids")
    repair_order_count = fields.Integer('Repair order count', compute="_compute_repair_order_ids")

    @api.depends('name')
    def _compute_repair_order_ids(self):
        repair_orders = defaultdict(lambda: self.env['repair.order'])
        for repair_line in self.env['repair.line'].search([('lot_id', 'in', self.ids), ('state', '=', 'done')]):
            repair_orders[repair_line.lot_id.id] |= repair_line.repair_id
        for lot in self:
            lot.repair_order_ids = repair_orders[lot.id]
            lot.repair_order_count = len(lot.repair_order_ids)

    def action_view_ro(self):
        self.ensure_one()

        action = {
            'res_model': 'repair.order',
            'type': 'ir.actions.act_window'
        }
        if len(self.repair_order_ids) == 1:
            action.update({
                'view_mode': 'form',
                'res_id': self.repair_order_ids[0].id
            })
        else:
            action.update({
                'name': _("Repair orders of %s", self.name),
                'domain': [('id', 'in', self.repair_order_ids.ids)],
                'view_mode': 'tree,form'
            })
        return action

```

## File: models\stock_picking.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models


class PickingType(models.Model):
    _inherit = 'stock.picking.type'

    is_repairable = fields.Boolean(
        'Create Repair Orders from Returns',
        compute='_compute_is_repairable', store=True, readonly=False,
        help="If ticked, you will be able to directly create repair orders from a return.")
    return_type_of_ids = fields.One2many('stock.picking.type', 'return_picking_type_id')

    @api.depends('return_type_of_ids', 'code')
    def _compute_is_repairable(self):
        for picking_type in self:
            if not(picking_type.code == 'incoming' and picking_type.return_type_of_ids):
                picking_type.is_repairable = False


class Picking(models.Model):
    _inherit = 'stock.picking'

    is_repairable = fields.Boolean(related='picking_type_id.is_repairable')
    repair_ids = fields.One2many('repair.order', 'picking_id')
    nbr_repairs = fields.Integer('Number of repairs linked to this picking', compute='_compute_nbr_repairs')

    @api.depends('repair_ids')
    def _compute_nbr_repairs(self):
        for picking in self:
            picking.nbr_repairs = len(picking.repair_ids)

    def action_repair_return(self):
        self.ensure_one()
        ctx = self.env.context.copy()
        ctx.update({
            'default_location_id': self.location_dest_id.id,
            'default_picking_id': self.id,
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

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import repair
from . import stock_picking
from . import stock_traceability
from . import stock_lot
from . import account_move
from . import product
from . import mail_compose_message

```

## File: report\repair_reports.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="action_report_repair_order" model="ir.actions.report">
            <field name="name">Quotation / Order</field>
            <field name="model">repair.order</field>
            <field name="report_type">qweb-pdf</field>
            <field name="report_name">repair.report_repairorder2</field>
            <field name="report_file">repair.report_repairorder</field>
            <field name="print_report_name">(
                object.state == 'draft' and 'Repair Quotation - %s' % (object.name) or
                'Repair Order - %s' % (object.name))</field>
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
                <t t-set="information_block">
                    <strong t-if="o.address_id == o.partner_invoice_id">Invoice and shipping address:</strong>
                    <div t-if="o.partner_invoice_id">
                        <strong t-if="o.address_id != o.partner_invoice_id">Invoice address: </strong>
                        <div t-field="o.partner_invoice_id" 
                             t-options='{"widget": "contact", "fields": ["address", "name", "phone", "vat"], "no_marker": True, "phone_icons": True}'/>
                    </div>
                    <t t-if="o.address_id != o.partner_invoice_id">
                        <strong>Shipping address :</strong>
                        <div t-field="o.address_id" 
                             t-options='{"widget": "contact", "fields": ["address", "name", "phone", "vat"], "no_marker": True, "phone_icons": True}'/>
                    </t>
                </t>
                <t t-set="address">
                    <div t-field="o.partner_id"
                        t-options='{"widget": "contact", "fields": ["address", "name"], "no_marker": True}' />
                </t>
                <div class="page">
                    <div class="oe_structure"/>

                    <h2>
                        <span t-if="o.state != 'draft'">Repair Order #:</span>
                        <span t-if="o.state == 'draft'">Repair Quotation #:</span>
                        <span t-field="o.name"/>
                    </h2>

                    <div id="informations" class="row mt32 mb32">
                        <div t-if="o.product_id.name" class="col-3 bm-2">
                            <strong>Product to Repair:</strong>
                            <p t-field="o.product_id.name" class="m-0"/>
                        </div>
                        <div class="col-3 bm-2" groups="stock.group_production_lot">
                            <strong>Lot/Serial Number:</strong>
                            <t t-if="o.lot_id">
                                <span t-field="o.lot_id.name"/>
                            </t>
                        </div>
                        <div t-if="o.guarantee_limit" class="col-3 bm-2">
                            <strong>Warranty:</strong>
                            <p t-field="o.guarantee_limit" class="m-0"/>
                        </div>
                        <div class="col-3 bm-2">
                            <strong>Printing Date:</strong>
                            <p t-esc="datetime.datetime.now().strftime('%Y-%m-%d')" t-options="{'widget': 'date'}" class="m-0"/>
                        </div>
                    </div>

                    <table class="table table-sm o_main_table">
                        <thead>
                            <tr>
                                <th>Description</th>
                                <th class="text-end">Quantity</th>
                                <t t-if="o.invoice_method != 'none'">
                                    <th class="text-end">Unit Price</th>
                                    <th class="text-center">Tax</th>
                                    <th class="text-end">Price</th>
                                </t>
                            </tr>
                        </thead>
                        <tbody>
                            <t t-if="o.operations">
                                <tr class="bg-200 o_line_section"><td colspan="5"><strong>Parts</strong></td></tr>
                                <tr t-foreach="o.operations" t-as="line">
                                    <td>
                                        <p t-if="line.type == 'add'"><i>(Add)</i> <span t-field="line.name" /></p>
                                        <p t-if="line.type == 'remove'">(<i>Remove</i>) <span t-field="line.name"/></p>
                                    </td>
                                    <td class="text-end">
                                        <span t-field="line.product_uom_qty"/>
                                        <span groups="uom.group_uom" t-field="line.product_uom.name"/>
                                    </td>
                                    <t t-if="(line.repair_id.invoice_method != 'none')">
                                        <td class="text-end">
                                            <span t-field="line.price_unit"/>
                                        </td>
                                        <td class="text-center">
                                            <span t-esc="','.join(map( lambda x: x.description or x.name, line.tax_id))"/>
                                        </td>
                                        <td class="text-end o_price_total">
                                            <span t-field="line.price_subtotal" t-options='{"widget": "monetary", "display_currency": o.pricelist_id.currency_id}'/>
                                       </td>
                                    </t>
                                </tr>
                            </t>
                            <t t-if="o.fees_lines">
                                <tr class="bg-200 o_line_section"><td colspan="5"><strong>Operations</strong></td></tr>
                                <tr t-foreach="o.fees_lines" t-as="fees">
                                    <td>
                                        <span t-field="fees.name"/>
                                    </td>
                                    <td class="text-end">
                                        <span t-field="fees.product_uom_qty"/>
                                        <span groups="uom.group_uom" t-field="fees.product_uom.name"/>
                                    </td>
                                    <t t-if="(fees.repair_id.invoice_method != 'none')">
                                        <td class="text-end">
                                            <span t-field="fees.price_unit"/>
                                        </td>
                                        <td class="text-center">
                                            <span t-esc="','.join(map( lambda x: x.description or x.name, fees.tax_id))"/>
                                        </td>
                                        <td class="text-end o_price_total">
                                            <span t-field="fees.price_subtotal"
                                                t-options='{"widget": "monetary", "display_currency": o.pricelist_id.currency_id}'/>
                                       </td>
                                    </t>
                                </tr>
                            </t>
                        </tbody>
                    </table>

                    <div id="total" class="row justify-content-end">
                        <div class="col-4">
                            <table class="table table-sm">
                                <t t-if="o.invoice_method !='none'">
                                    <tr class="border-black o_subtotal">
                                        <td><strong>Total Without Taxes</strong></td>
                                        <td class="text-end">
                                            <span t-field="o.amount_untaxed"
                                                t-options='{"widget": "monetary", "display_currency": o.pricelist_id.currency_id}'/>
                                        </td>
                                    </tr>
                                    <tr>
                                        <td>Taxes</td>
                                        <td class="text-end o_price_total">
                                            <span t-field="o.amount_tax"
                                                t-options='{"widget": "monetary", "display_currency": o.pricelist_id.currency_id}'/>
                                        </td>
                                    </tr>
                                    <tr class="border-black o_total">
                                        <td><strong>Total</strong></td>
                                        <td class="text-end o_price_total">
                                            <span t-field="o.amount_total"
                                                t-options='{"widget": "monetary", "display_currency": o.pricelist_id.currency_id}'/>
                                        </td>
                                    </tr>
                                </t>
                            </table>
                        </div>
                    </div>

                    <p t-field="o.quotation_notes"/>
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

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_repair_fee_user,Repair Fee user,model_repair_fee,stock.group_stock_user,1,1,1,1
access_repair_user,Repair user,model_repair_order,stock.group_stock_user,1,1,1,1
access_repair_tag_user,Repair Tags user,model_repair_tags,stock.group_stock_user,1,1,1,1
access_repair_line_user,repair.line user,model_repair_line,stock.group_stock_user,1,1,1,1
access_account_tax_user,account.tax,account.model_account_tax,stock.group_stock_user,1,0,0,0
access_repair_order_make_invoice,access.repair.order.make_invoice,model_repair_order_make_invoice,stock.group_stock_user,1,1,1,0
access_stock_warn_insufficient_qty_repair,access.stock.warn.insufficient.qty.repair,model_stock_warn_insufficient_qty_repair,stock.group_stock_user,1,1,1,0

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
    <record model="ir.rule" id="repair_line_rule">
        <field name="name">repair line multi-company</field>
        <field name="model_id" search="[('model','=','repair.line')]" model="ir.model"/>
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>
    <record model="ir.rule" id="repair_fee_rule">
        <field name="name">repair fee multi-company</field>
        <field name="model_id" search="[('model','=','repair.fee')]" model="ir.model"/>
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>

</odoo>

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#7CC098"/><stop offset="100%" stop-color="#5F8A71"/></linearGradient><path id="d" d="M18.952 29.454h8.316l3.163-4.327-3.163-4.327h-8.316c-.74 0-1.192-.791-.81-1.411C20.135 16.159 23.76 14 27.9 14c6.283 0 11.426 5.059 11.387 11.198-.04 6.112-5.122 11.055-11.387 11.055-4.145 0-7.773-2.164-9.764-5.399-.38-.616.08-1.4.816-1.4zm14.994 8.574l-1.42-1.388c3.258-1.25 5.88-3.81 7.157-6.993l15.835 15.472a4.866 4.866 0 0 1 0 6.994c-1.977 1.931-5.181 1.931-7.157 0l-2.487-2.43c-.735-6.3-5.734-11.243-11.928-11.655zm14.83 9.352c0 1.024.85 1.854 1.898 1.854s1.898-.83 1.898-1.854-.85-1.854-1.898-1.854-1.898.83-1.898 1.854zm-21.772 3.937a7.262 7.262 0 0 1 0-2.634l-1.6-.924a.451.451 0 0 1-.205-.525 9.294 9.294 0 0 1 2.053-3.55.451.451 0 0 1 .556-.084l1.599.923a7.19 7.19 0 0 1 2.28-1.319v-1.846c0-.21.147-.393.353-.44a9.382 9.382 0 0 1 4.1 0 .45.45 0 0 1 .351.44v1.846a7.19 7.19 0 0 1 2.28 1.319l1.6-.923a.451.451 0 0 1 .556.084 9.294 9.294 0 0 1 2.053 3.55.451.451 0 0 1-.205.525l-1.6.924c.16.87.16 1.763 0 2.634l1.6.924a.451.451 0 0 1 .205.525 9.294 9.294 0 0 1-2.053 3.55.451.451 0 0 1-.557.084l-1.598-.923a7.19 7.19 0 0 1-2.28 1.319v1.846a.451.451 0 0 1-.353.44 9.382 9.382 0 0 1-4.1 0 .45.45 0 0 1-.352-.44v-1.846a7.19 7.19 0 0 1-2.28-1.319l-1.598.923a.451.451 0 0 1-.557-.084 9.294 9.294 0 0 1-2.053-3.55.451.451 0 0 1 .205-.525l1.6-.924zM31 50a3.129 3.129 0 0 0 3.125 3.125A3.129 3.129 0 0 0 37.25 50a3.129 3.129 0 0 0-3.125-3.125A3.129 3.129 0 0 0 31 50z"/><path id="e" d="M18.952 27.454h8.316l3.163-4.327-3.163-4.327h-8.316c-.74 0-1.192-.791-.81-1.411C20.135 14.159 23.76 12 27.9 12c6.283 0 11.426 5.059 11.387 11.198-.04 6.112-5.122 11.055-11.387 11.055-4.145 0-7.773-2.164-9.764-5.399-.38-.616.08-1.4.816-1.4zm14.994 8.574l-1.42-1.388c3.258-1.25 5.88-3.81 7.157-6.993l15.835 15.472a4.866 4.866 0 0 1 0 6.994c-1.977 1.931-5.181 1.931-7.157 0l-2.487-2.43c-.735-6.3-5.734-11.243-11.928-11.655zm14.83 9.352c0 1.024.85 1.854 1.898 1.854s1.898-.83 1.898-1.854-.85-1.854-1.898-1.854-1.898.83-1.898 1.854zm-21.772 3.937a7.262 7.262 0 0 1 0-2.634l-1.6-.924a.451.451 0 0 1-.205-.525 9.294 9.294 0 0 1 2.053-3.55.451.451 0 0 1 .556-.084l1.599.923a7.19 7.19 0 0 1 2.28-1.319v-1.846c0-.21.147-.393.353-.44a9.382 9.382 0 0 1 4.1 0 .45.45 0 0 1 .351.44v1.846a7.19 7.19 0 0 1 2.28 1.319l1.6-.923a.451.451 0 0 1 .556.084 9.294 9.294 0 0 1 2.053 3.55.451.451 0 0 1-.205.525l-1.6.924c.16.87.16 1.763 0 2.634l1.6.924a.451.451 0 0 1 .205.525 9.294 9.294 0 0 1-2.053 3.55.451.451 0 0 1-.557.084l-1.598-.923a7.19 7.19 0 0 1-2.28 1.319v1.846a.451.451 0 0 1-.353.44 9.382 9.382 0 0 1-4.1 0 .45.45 0 0 1-.352-.44v-1.846a7.19 7.19 0 0 1-2.28-1.319l-1.598.923a.451.451 0 0 1-.557-.084 9.294 9.294 0 0 1-2.053-3.55.451.451 0 0 1 .205-.525l1.6-.924zM31 48a3.129 3.129 0 0 0 3.125 3.125A3.129 3.129 0 0 0 37.25 48a3.129 3.129 0 0 0-3.125-3.125A3.129 3.129 0 0 0 31 48z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M4 69c-2 0-4-1-4-4V38.984l18.987-22.726L24 13l31 30 1.03 6.473L42.838 69H4z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: views\repair_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

    <record id="view_repair_order_tree" model="ir.ui.view">
        <field name="name">repair.tree</field>
        <field name="model">repair.order</field>
        <field name="arch" type="xml">
            <tree string="Repairs order" multi_edit="1" sample="1" decoration-info="state == 'draft'">
                <field name="company_id" invisible="1"/>
                <field name="priority" optional="show" widget="priority" nolabel="1"/>
                <field name="name"/>
                <field name="schedule_date" optional="show" widget="remaining_days"/>
                <field name="description" optional="hide"/>
                <field name="product_id" readonly="1" optional="show"/>
                <field name="product_qty" optional="hide" string="Quantity"/>
                <field name="product_uom" string="Unit of Measure" readonly="1" groups="uom.group_uom" optional="hide"/>
                <field name="user_id" optional="hide" widget='many2one_avatar_user'/>
                <field name="partner_id" readonly="1" optional="show"/>
                <field name="address_id" optional="show"/>
                <field name="guarantee_limit" optional="show"/>
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
               <header>
                   <button name="action_validate" states="draft" type="object" string="Confirm Repair" class="oe_highlight" data-hotkey="v"/>
                   <button name="action_repair_start" attrs="{'invisible': ['&amp;', ('state','!=','confirmed'), '!', '&amp;', ('state','=','ready'), ('invoice_method','=','b4repair')]}"
                    type="object" string="Start Repair" class="oe_highlight" data-hotkey="q"/>
                   <button name="action_repair_end" states="under_repair" type="object" string="End Repair" class="oe_highlight" data-hotkey="x"/>
                   <button name="action_repair_invoice_create" type="object" string="Create Invoice" class="oe_highlight" groups="account.group_account_invoice" attrs="{'invisible': ['|', ('state', '!=', '2binvoiced'), ('invoice_id', '!=', False)]}" data-hotkey="w"/>
                   <button name="action_send_mail" states="draft" string="Send Quotation" type="object" data-hotkey="g"/>
                   <button name="print_repair_order" states="draft" string="Print Quotation" type="object" data-hotkey="y"/>
                   <button name="action_repair_cancel_draft" states="cancel" string="Set to Draft" type="object" data-hotkey="z"/>
                   <button name="action_repair_cancel" string="Cancel Repair" type="object" confirm="Draft invoices for this order will be cancelled. Do you confirm the action?" attrs="{'invisible':['|', ('state', 'in', ('cancel', 'done')), ('invoice_state', '!=', 'draft')]}" data-hotkey="l"/>
                   <button name="action_repair_cancel" string="Cancel Repair" type="object" attrs="{'invisible': ['|', ('state','in', ('cancel','done')), ('invoice_state', '=', 'draft')]}" data-hotkey="l"/>
                   <field name="state" widget="statusbar" statusbar_visible="draft,confirmed,done"/>
               </header>
               <sheet string="Repairs order">
                    <div class="oe_button_box" name="button_box">
                        <field name="invoice_id" invisible="1"/>
                        <button name="%(action_repair_move_lines)d" type="action" string="Product Moves" class="oe_stat_button" icon="fa-exchange" attrs="{'invisible': [('state', 'not in', ['done', 'cancel'])]}"/>
                        <button name="action_created_invoice"
                            type="object"
                            class="oe_stat_button"
                            icon="fa-pencil-square-o"
                            attrs="{'invisible': [('invoice_id', '=', False)]}"
                            groups="account.group_account_invoice">
                            <div class="o_field_widget o_stat_info">
                                <span class="o_stat_text">1</span>
                                <span class="o_stat_text">Invoices</span>
                            </div>
                        </button>
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
                            <field name="description"/>
                            <field name="invoice_state" invisible="1"/>
                            <field name="tracking" invisible="1" attrs="{'readonly': 1}"/>
                            <field name="company_id" invisible="1"/>
                            <field name="product_id"/>
                            <field name="lot_id" context="{'default_product_id': product_id, 'default_company_id': company_id}" groups="stock.group_production_lot" attrs="{'required':[('tracking', 'in', ['serial', 'lot'])], 'invisible': [('tracking', 'not in', ['serial', 'lot'])], 'readonly': [('state', '=', 'done')]}"/>
                            <field name="product_uom_category_id" invisible="1"/>
                            <label for="product_qty"/>
                            <div class="o_row">
                                <field name="product_qty" attrs="{'readonly':[('tracking', '=', 'serial')]}"/>
                                <field name="product_uom" groups="uom.group_uom"/>
                            </div>
                            <field name="partner_id" widget="res_partner_many2one" attrs="{'required':[('invoice_method','!=','none')]}" context="{'res_partner_search_mode': 'customer', 'show_vat': True}"/>
                            <field name="address_id" groups="account.group_delivery_invoice_address"/>
                            <field name="sale_order_id"/>
                            <field name="allowed_picking_type_ids" invisible="1"/>
                            <field name="picking_id" domain="[('picking_type_id','in', allowed_picking_type_ids), ('product_id','=',product_id)]" options="{'no_create': True}"/>
                        </group>
                        <group>
                            <field name="schedule_date"/>
                            <field name="user_id" domain="[('share', '=', False)]"/>
                            <field name="location_id" options="{'no_create': True}"/>
                            <field name="company_id" groups="base.group_multi_company" options="{'no_create': True}"/>
                            <field name="guarantee_limit"/>
                            <field name="invoice_method"/>
                            <field name="partner_invoice_id" attrs="{'invisible':[('invoice_method','=', 'none')],'required':[('invoice_method','!=','none')]}" groups="account.group_delivery_invoice_address"/>
                            <field name="pricelist_id" groups="product.group_product_pricelist" context="{'product_id':product_id}" attrs="{'invisible':[('invoice_method','=', 'none')]}"/>
                            <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color', 'no_create_edit': True}"/>
                            <field name="currency_id" invisible="1"/>
                        </group>
                    </group>
                <notebook>
                    <page string="Parts" name="parts">
                        <field name="operations" context="{'default_product_uom_qty': product_qty, 'default_company_id': company_id}" attrs="{'readonly':[('state', 'in', ['done', 'cancel'])]}">
                            <form string="Operations">
                                <group>
                                    <group>
                                        <field name="company_id" invisible="1" force_save="1"/>
                                        <field name="type"/>
                                        <field name="product_id"/>
                                        <field name="name"/>
                                        <field name="product_uom_category_id" invisible="1"/>
                                        <label for="product_uom_qty"/>
                                        <div class="o_row">
                                            <field name="product_uom_qty"/>
                                            <field name="product_uom" groups="uom.group_uom"/>
                                        </div>
                                        <field name="price_unit"/>
                                        <field name="tax_id" widget="many2many_tags"/>
                                        <field name="invoiced" invisible="1"/>
                                        <field name="price_subtotal" widget="monetary" invisible="1"/>
                                        <field name="currency_id" invisible="1"/>
                                    </group>
                                    <group>
                                        <field name="lot_id" context="{'default_product_id': product_id, 'default_company_id': company_id}" groups="stock.group_production_lot"/>
                                        <field name="location_id" options="{'no_create': True}" groups="stock.group_stock_multi_locations"/>
                                        <field name="location_dest_id" options="{'no_create': True}" groups="stock.group_stock_multi_locations"/>
                                    </group>
                                </group>
                                <group name="History" string="History" attrs="{'invisible':[('move_id','=', False)]}">
                                    <field name="move_id"/>
                                    <field name="invoice_line_id" invisible="1"/>
                                </group>
                            </form>
                            <tree string="Operations" editable="bottom">
                                <field name="company_id" invisible="1" force_save="1"/>
                                <field name="type"/>
                                <field name="product_id"/>
                                <field name='name' optional="show"/>
                                <field name="product_uom_category_id" invisible="1"/>
                                <field name="tracking" invisible="1"/>
                                <field name="lot_id" context="{'default_product_id': product_id, 'default_company_id': company_id}" groups="stock.group_production_lot" attrs="{'readonly':[('tracking', 'not in', ['serial', 'lot'])]}"/>
                                <field name="location_id" options="{'no_create': True}" groups="stock.group_stock_multi_locations" optional="show"/>
                                <field name="location_dest_id" options="{'no_create': True}" groups="stock.group_stock_multi_locations" optional="show"/>
                                <field name="product_uom_qty" string="Quantity"/>
                                <field name="product_uom" string="UoM" groups="uom.group_uom" optional="show"/>
                                <field name="price_unit"/>
                                <field name="tax_id" widget="many2many_tags" optional="show"/>
                                <field name="price_subtotal" widget="monetary" groups="account.group_show_line_subtotals_tax_excluded"/>
                                <field name="price_total" widget="monetary" groups="account.group_show_line_subtotals_tax_included"/>
                                <field name="currency_id" invisible="1"/>
                            </tree>
                        </field>
                        <group class="oe_subtotal_footer oe_right">
                            <field name="amount_untaxed" sum="Untaxed amount"/>
                            <field name="amount_tax"/>
                            <div class="oe_subtotal_footer_separator oe_inline o_td_label">
                                <label for="amount_total" />
                                <button name="button_dummy"
                                    states="draft" string="(update)" type="object" class="oe_edit_only oe_link"/>
                            </div>
                            <field name="amount_total" nolabel="1" sum="Total amount" class="oe_subtotal_footer_separator"/>
                        </group>
                        <div class="clearfix"/>
                    </page>
                    <page string="Operations" name="operations">
                        <field name="fees_lines" context="{'default_company_id': company_id}" attrs="{'readonly':[('state', 'in', ['done', 'cancel'])]}">
                            <form string="Fees">
                                <group>
                                    <field name="company_id" invisible="1" force_save="1"/>
                                    <field name="product_id" required="True"/>
                                    <field name="name"/>
                                    <field name="product_uom_category_id" invisible="1"/>
                                    <label for="product_uom_qty"/>
                                    <div class="o_row">
                                        <field name="product_uom_qty" string="Quantity"/>
                                        <field name="product_uom" groups="uom.group_uom"/>
                                    </div>
                                    <field name="price_unit"/>
                                    <field widget="many2many_tags" name="tax_id"/>
                                    <field name="price_subtotal" widget="monetary" invisible="1"/>
                                    <field name="currency_id" invisible="1"/>
                                </group>
                            </form>
                            <tree string="Fees" editable="bottom">
                                <field name="company_id" invisible="1" force_save="1"/>
                                <field name="product_id" required="True" context="{'default_type': 'service'}"/>
                                <field name='name' optional="show"/>
                                <field name="product_uom_qty" string="Quantity"/>
                                <field name="product_uom_category_id" invisible="1"/>
                                <field name="product_uom" string="Unit of Measure" groups="uom.group_uom" optional="show"/>
                                <field name="price_unit"/>
                                <field name="tax_id" widget="many2many_tags" optional="show"/>
                                <field name="price_subtotal" widget="monetary" groups="account.group_show_line_subtotals_tax_excluded"/>
                                <field name="price_total" widget="monetary" groups="account.group_show_line_subtotals_tax_included"/>
                                <field name="currency_id" invisible="1"/>
                            </tree>
                        </field>
                    </page>
                    <page string="Extra Info" name="extra_info" groups="base.group_no_one">
                        <group>
                            <group>
                                <field name="move_id"/>
                            </group>
                            <group>
                                <field name="repaired"/>
                                <field name="invoiced"/>
                            </group>
                        </group>
                    </page>
                    <page string="Repair Notes">
                        <field name="internal_notes" placeholder="Add internal notes."/>
                    </page>
                    <page string="Quotation Notes">
                        <field name="quotation_notes" placeholder="Add quotation notes."/>
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
            <kanban class="o_kanban_mobile" sample="1">
                <field name="company_id" invisible="1"/>
                <field name="name" />
                <field name="product_id" />
                <field name="partner_id"/>
                <field name="address_id"/>
                <field name="guarantee_limit"/>
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
                <filter string="Quotations" name="quotations" domain="[('state', '=', 'draft')]"/>
                <filter string="Confirmed" domain="[('state', '=', 'confirmed')]" name="current" />
                <filter string="Ready To Repair" name="ready_to_repair" domain="[('state', '=', 'ready')]"/>
                <filter string="Returned" name="is_returned" domain="[('picking_id', '!=', False), ('picking_id.state', '=', 'done'), ('state', 'not in', ['cancel', 'done'])]"/>
                <separator/>
                <filter string="Invoiced" name="invoiced" domain="[('invoiced', '=', True)]"/>
                <separator/>
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
                    <filter string="Warranty Expiration" name="guarantee_limit" domain="[]" context="{'group_by': 'guarantee_limit'}"/>
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

         <record id="action_repair_order_tree" model="ir.actions.act_window">
            <field name="name">Repair Orders</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">repair.order</field>
            <field name="view_mode">tree,kanban,graph,pivot,form</field>
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
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">repair.order</field>
            <field name="view_mode">tree,kanban,graph,pivot,form</field>
            <field name="view_id" ref="view_repair_graph"/>
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

        <menuitem action="action_repair_order_tree" id="menu_repair_order" groups="stock.group_stock_user" name="Repairs" sequence="165"
            web_icon="repair,static/description/icon.svg"/>

        <menuitem id="repair_menu_reporting" name="Reporting" parent="menu_repair_order" groups="stock.group_stock_manager"/>

        <menuitem id="repair_menu" name="Repairs" parent="repair_menu_reporting" action="action_repair_order_graph"/>

        <menuitem id="repair_menu_config" name="Configuration" parent="menu_repair_order" groups="stock.group_stock_manager"/>

        <menuitem id="repair_menu_tag" name="Repair Orders Tags" parent="repair_menu_config" action="action_repair_order_tag"/>
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
                        attrs="{'invisible': ['|', ('repair_order_count', '=', 0), ('display_complete', '=', False)]}">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_value">
                            <field name="repair_order_count" widget="statinfo" nolabel="1" class="mr4"/>
                        </span>
                        <span class="o_stat_text">Repairs</span>
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
        <field name="type">ir.actions.act_window</field>
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
            <xpath expr="//group[@name='locations']" position="after">
                <field name="return_type_of_ids" invisible="1"/>
                <group string="Repairs" attrs="{'invisible': ['|', ('code', '!=', 'incoming'), ('return_type_of_ids', '=', [])]}">
                    <field name="is_repairable"/>
                </group>
            </xpath>
        </field>
    </record>

    <record id="repair_view_picking_form" model="ir.ui.view">
        <field name="name">stock.picking.form.inherit.repair</field>
        <field name="model">stock.picking</field>
        <field name="inherit_id" ref="stock.view_picking_form"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@name='%(stock.act_stock_return_picking)d']" position="after">
                <field name="is_repairable" invisible="1"/>
                <button string="Repair" name="action_repair_return" type="object" attrs="{'invisible': [('is_repairable', '=', False)]}" data-hotkey="shift+k"/>
            </xpath>
            <xpath expr="//button[@name='action_see_packages']" position="after">
                <field name="repair_ids" invisible="1"/>
                <button name="action_view_repairs" type="object"
                        class="oe_stat_button" icon="fa-wrench"
                        attrs="{'invisible': [('nbr_repairs', '=', 0)]}">
                        <field name="nbr_repairs" string="Repair Orders" widget="statinfo"/>
                </button>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: wizard\repair_make_invoice.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class MakeInvoice(models.TransientModel):
    _name = 'repair.order.make_invoice'
    _description = 'Create Mass Invoice (repair)'

    group = fields.Boolean('Group by partner invoice address')

    def make_invoices(self):
        if not self._context.get('active_ids'):
            return {'type': 'ir.actions.act_window_close'}
        new_invoice = {}
        for wizard in self:
            repairs = self.env['repair.order'].browse(self._context['active_ids'])
            new_invoice = repairs._create_invoices(group=wizard.group)

            # We have to udpate the state of the given repairs, otherwise they remain 'to be invoiced'.
            # Note that this will trigger another call to the method '_create_invoices',
            # but that second call will not do anything, since the repairs are already invoiced.
            repairs.action_repair_invoice_create()
        return {
            'domain': [('id', 'in', list(new_invoice.values()))],
            'name': 'Invoices',
            'view_mode': 'tree,form',
            'res_model': 'account.move',
            'view_id': False,
            'views': [(self.env.ref('account.view_move_tree').id, 'tree'), (self.env.ref('account.view_move_form').id, 'form')],
            'context': "{'move_type':'out_invoice'}",
            'type': 'ir.actions.act_window'
        }

```

## File: wizard\repair_make_invoice_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        
        <!--  Make Invoice -->
        
        <record id="view_make_invoice" model="ir.ui.view">
            <field name="name">Make Invoice</field>
            <field name="model">repair.order.make_invoice</field>
            <field name="arch" type="xml">
            <form string="Create invoices">
                <group string="Do you really want to create the invoice(s)?">
                    <field name="group"/>
                </group>
                <footer>
                    <button name="make_invoices" string="Create Invoice" type="object" class="btn-primary" data-hotkey="q"/>
                    <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="z" />
                </footer>
            </form>
            </field>
        </record>

        <record id="act_repair_invoice" model="ir.actions.act_window">
            <field name="name">Create invoices</field>
            <field name="res_model">repair.order.make_invoice</field>
            <field name="view_mode">form</field>
            <field name="target">new</field>
            <field name="binding_model_id" ref="model_repair_order"/>
            <field name="binding_view_types">list</field>
        </record>

    </data>
</odoo>

```

## File: wizard\stock_warn_insufficient_qty.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models

class StockWarnInsufficientQtyRepair(models.TransientModel):
    _name = 'stock.warn.insufficient.qty.repair'
    _inherit = 'stock.warn.insufficient.qty'
    _description = 'Warn Insufficient Repair Quantity'

    repair_id = fields.Many2one('repair.order', string='Repair')

    def _get_reference_document_company_id(self):
        return self.repair_id.company_id

    def action_done(self):
        self.ensure_one()
        return self.repair_id.action_repair_confirm()

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

from . import repair_make_invoice
from . import stock_warn_insufficient_qty

```

