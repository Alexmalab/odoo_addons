# Odoo Module: purchase_requisition

Category: Inventory/Purchase

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Purchase Agreements',
    'version': '0.1',
    'category': 'Inventory/Purchase',
    'description': """
This module allows you to manage your Purchase Agreements.
===========================================================

Manage calls for tenders and blanket orders. Calls for tenders are used to get
competing offers from different vendors and select the best ones. Blanket orders
are agreements you have with vendors to benefit from a predetermined pricing.
""",
    'depends': ['purchase'],
    'demo': ['data/purchase_requisition_demo.xml'],
    'data': [
        'security/purchase_requisition_security.xml',
        'security/ir.model.access.csv',
        'data/purchase_requisition_data.xml',
        'views/product_views.xml',
        'views/purchase_views.xml',
        'views/purchase_requisition_views.xml',
        'views/res_config_settings_views.xml',
        'report/purchase_requisition_report.xml',
        'report/report_purchaserequisition.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\purchase_requisition_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="type_single" model="purchase.requisition.type">
            <field name="name">Blanket Order</field>
            <field name="sequence">3</field>
            <field name="quantity_copy">none</field>
        </record>
        <record id="type_multi" model="purchase.requisition.type">
            <field name="name">Call for Tender</field>
            <field name="sequence">1</field>
            <field name="quantity_copy">copy</field>
        </record>

        <record id="seq_purchase_tender" model="ir.sequence">
            <field name="name">Call for Tender</field>
            <field name="code">purchase.requisition.purchase.tender</field>
            <field name="prefix">TE</field>
            <field name="padding">5</field>
            <field name="company_id" eval="False"></field>
        </record>

        <record id="seq_blanket_order" model="ir.sequence">
            <field name="name">Blanket Order</field>
            <field name="code">purchase.requisition.blanket.order</field>
            <field name="prefix">BO</field>
            <field name="padding">5</field>
            <field name="company_id" eval="False"></field>
        </record>
    </data>
</odoo>

```

## File: data\purchase_requisition_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <record id="base.user_demo" model="res.users">
            <field eval="[(4, ref('purchase.group_purchase_user'))]" name="groups_id"/>
        </record>

        <record id="product.product_product_13" model="product.product">
            <field name="purchase_requisition">tenders</field>
        </record>

    <!--Resource: purchase.requisition-->

        <record id="requisition1" model="purchase.requisition">
            <field name="user_id" ref="base.user_admin"/>
        </record>

        <record id="requisition_line1" model="purchase.requisition.line">
            <field name="requisition_id" ref="requisition1"/>
            <field name="product_id" ref="product.product_product_13"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="product_qty">5</field>
        </record>
        <function model="purchase.requisition" name="action_in_progress" eval="[[ref('requisition1')]]"/>

        <!--Resource: purchase.order-->

        <record id="rfq1" model="purchase.order">
            <field name="partner_id" ref="base.res_partner_4"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="requisition_id" ref="requisition1"/>
        </record>

        <record id="rfq1_line" model="purchase.order.line">
            <field name="order_id" ref="rfq1"/>
            <field name="name" model="purchase.order.line" eval="obj().env.ref('product.product_product_13').partner_ref"/>
            <field name="date_planned" eval="time.strftime('%Y-%m-10')"/>
            <field name="product_id" ref="product.product_product_13"/>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">60</field>
            <field name="product_qty">5</field>
        </record>
               
        <record id="rfq2" model="purchase.order">
            <field name="partner_id" ref="base.res_partner_1"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="requisition_id" ref="requisition1"/>
        </record>

        <record id="rfq2_line" model="purchase.order.line">
            <field name="order_id" ref="rfq2"/>
            <field name="name" model="purchase.order.line" eval="obj().env.ref('product.product_product_13').partner_ref"/>
            <field name="date_planned" eval="time.strftime('%Y-%m-15')"/>
            <field name="product_id" ref="product.product_product_13"/>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">50</field>
            <field name="product_qty">3</field>
        </record>        


    </data>
</odoo>

```

## File: models\product.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class SupplierInfo(models.Model):
    _inherit = 'product.supplierinfo'

    purchase_requisition_id = fields.Many2one('purchase.requisition', related='purchase_requisition_line_id.requisition_id', string='Agreement', readonly=False)
    purchase_requisition_line_id = fields.Many2one('purchase.requisition.line')


class ProductProduct(models.Model):
    _inherit = 'product.product'

    def _prepare_sellers(self, params=False):
        sellers = super(ProductProduct, self)._prepare_sellers(params=params)
        if params and params.get('order_id'):
            return sellers.filtered(lambda s: not s.purchase_requisition_id or s.purchase_requisition_id == params['order_id'].requisition_id)
        else:
            return sellers


class ProductTemplate(models.Model):
    _inherit = 'product.template'

    purchase_requisition = fields.Selection(
        [('rfq', 'Create a draft purchase order'),
         ('tenders', 'Propose a call for tenders')],
        string='Procurement', default='rfq',
        help="Create a draft purchase order: Based on your product configuration, the system will create a draft "
             "purchase order.Propose a call for tender : If the 'purchase_requisition' module is installed and this option "
             "is selected, the system will create a draft call for tender.")

```

## File: models\purchase.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class PurchaseOrder(models.Model):
    _inherit = 'purchase.order'

    requisition_id = fields.Many2one('purchase.requisition', string='Purchase Agreement', copy=False)
    is_quantity_copy = fields.Selection(related='requisition_id.is_quantity_copy', readonly=False)

    @api.onchange('requisition_id')
    def _onchange_requisition_id(self):
        if not self.requisition_id:
            return

        self = self.with_company(self.company_id)
        requisition = self.requisition_id
        if self.partner_id:
            partner = self.partner_id
        else:
            partner = requisition.vendor_id
        payment_term = partner.property_supplier_payment_term_id

        FiscalPosition = self.env['account.fiscal.position']
        fpos = FiscalPosition.with_company(self.company_id).get_fiscal_position(partner.id)

        self.partner_id = partner.id
        self.fiscal_position_id = fpos.id
        self.payment_term_id = payment_term.id,
        self.company_id = requisition.company_id.id
        self.currency_id = requisition.currency_id.id
        if not self.origin or requisition.name not in self.origin.split(', '):
            if self.origin:
                if requisition.name:
                    self.origin = self.origin + ', ' + requisition.name
            else:
                self.origin = requisition.name
        self.notes = requisition.description
        self.date_order = fields.Datetime.now()

        if requisition.type_id.line_copy != 'copy':
            return

        # Create PO lines if necessary
        order_lines = []
        for line in requisition.line_ids:
            # Compute name
            product_lang = line.product_id.with_context(
                lang=partner.lang or self.env.user.lang,
                partner_id=partner.id
            )
            name = product_lang.display_name
            if product_lang.description_purchase:
                name += '\n' + product_lang.description_purchase

            # Compute taxes
            taxes_ids = fpos.map_tax(line.product_id.supplier_taxes_id.filtered(lambda tax: tax.company_id == requisition.company_id)).ids

            # Compute quantity and price_unit
            if line.product_uom_id != line.product_id.uom_po_id:
                product_qty = line.product_uom_id._compute_quantity(line.product_qty, line.product_id.uom_po_id)
                price_unit = line.product_uom_id._compute_price(line.price_unit, line.product_id.uom_po_id)
            else:
                product_qty = line.product_qty
                price_unit = line.price_unit

            if requisition.type_id.quantity_copy != 'copy':
                product_qty = 0

            # Create PO line
            order_line_values = line._prepare_purchase_order_line(
                name=name, product_qty=product_qty, price_unit=price_unit,
                taxes_ids=taxes_ids)
            order_lines.append((0, 0, order_line_values))
        self.order_line = order_lines

    def button_confirm(self):
        res = super(PurchaseOrder, self).button_confirm()
        for po in self:
            if not po.requisition_id:
                continue
            if po.requisition_id.type_id.exclusive == 'exclusive':
                others_po = po.requisition_id.mapped('purchase_ids').filtered(lambda r: r.id != po.id)
                others_po.button_cancel()
                if po.state not in ['draft', 'sent', 'to approve']:
                    po.requisition_id.action_done()
        return res

    @api.model
    def create(self, vals):
        purchase = super(PurchaseOrder, self).create(vals)
        if purchase.requisition_id:
            purchase.message_post_with_view('mail.message_origin_link',
                    values={'self': purchase, 'origin': purchase.requisition_id},
                    subtype_id=self.env['ir.model.data'].xmlid_to_res_id('mail.mt_note'))
        return purchase

    def write(self, vals):
        result = super(PurchaseOrder, self).write(vals)
        if vals.get('requisition_id'):
            self.message_post_with_view('mail.message_origin_link',
                    values={'self': self, 'origin': self.requisition_id, 'edit': True},
                    subtype_id=self.env['ir.model.data'].xmlid_to_res_id('mail.mt_note'))
        return result


class PurchaseOrderLine(models.Model):
    _inherit = 'purchase.order.line'

    def _compute_account_analytic_id(self):
        super(PurchaseOrderLine, self.filtered(lambda pol: not pol.order_id.requisition_id))._compute_account_analytic_id()

    def _compute_analytic_tag_ids(self):
        super(PurchaseOrderLine, self.filtered(lambda pol: not pol.order_id.requisition_id))._compute_analytic_tag_ids()

    @api.onchange('product_qty', 'product_uom')
    def _onchange_quantity(self):
        res = super(PurchaseOrderLine, self)._onchange_quantity()
        if self.order_id.requisition_id:
            for line in self.order_id.requisition_id.line_ids.filtered(lambda l: l.product_id == self.product_id):
                if line.product_uom_id != self.product_uom:
                    self.price_unit = line.product_uom_id._compute_price(
                        line.price_unit, self.product_uom)
                else:
                    self.price_unit = line.price_unit
                break
        return res

```

## File: models\purchase_requisition.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from datetime import datetime, time

from odoo import api, fields, models, _
from odoo.exceptions import UserError


PURCHASE_REQUISITION_STATES = [
    ('draft', 'Draft'),
    ('ongoing', 'Ongoing'),
    ('in_progress', 'Confirmed'),
    ('open', 'Bid Selection'),
    ('done', 'Closed'),
    ('cancel', 'Cancelled')
]


class PurchaseRequisitionType(models.Model):
    _name = "purchase.requisition.type"
    _description = "Purchase Requisition Type"
    _order = "sequence"

    name = fields.Char(string='Agreement Type', required=True, translate=True)
    sequence = fields.Integer(default=1)
    exclusive = fields.Selection([
        ('exclusive', 'Select only one RFQ (exclusive)'), ('multiple', 'Select multiple RFQ (non-exclusive)')],
        string='Agreement Selection Type', required=True, default='multiple',
            help="""Select only one RFQ (exclusive):  when a purchase order is confirmed, cancel the remaining purchase order.\n
                    Select multiple RFQ (non-exclusive): allows multiple purchase orders. On confirmation of a purchase order it does not cancel the remaining orders""")
    quantity_copy = fields.Selection([
        ('copy', 'Use quantities of agreement'), ('none', 'Set quantities manually')],
        string='Quantities', required=True, default='none')
    line_copy = fields.Selection([
        ('copy', 'Use lines of agreement'), ('none', 'Do not create RfQ lines automatically')],
        string='Lines', required=True, default='copy')


class PurchaseRequisition(models.Model):
    _name = "purchase.requisition"
    _description = "Purchase Requisition"
    _inherit = ['mail.thread', 'mail.activity.mixin']
    _order = "id desc"

    def _get_type_id(self):
        return self.env['purchase.requisition.type'].search([], limit=1)

    name = fields.Char(string='Reference', required=True, copy=False, default='New', readonly=True)
    origin = fields.Char(string='Source Document')
    order_count = fields.Integer(compute='_compute_orders_number', string='Number of Orders')
    vendor_id = fields.Many2one('res.partner', string="Vendor", domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    type_id = fields.Many2one('purchase.requisition.type', string="Agreement Type", required=True, default=_get_type_id)
    ordering_date = fields.Date(string="Ordering Date", tracking=True)
    date_end = fields.Datetime(string='Agreement Deadline', tracking=True)
    schedule_date = fields.Date(string='Delivery Date', index=True, help="The expected and scheduled delivery date where all the products are received", tracking=True)
    user_id = fields.Many2one(
        'res.users', string='Purchase Representative',
        default=lambda self: self.env.user, check_company=True)
    description = fields.Text()
    company_id = fields.Many2one('res.company', string='Company', required=True, default=lambda self: self.env.company)
    purchase_ids = fields.One2many('purchase.order', 'requisition_id', string='Purchase Orders', states={'done': [('readonly', True)]})
    line_ids = fields.One2many('purchase.requisition.line', 'requisition_id', string='Products to Purchase', states={'done': [('readonly', True)]}, copy=True)
    product_id = fields.Many2one('product.product', related='line_ids.product_id', string='Product', readonly=False)
    state = fields.Selection(PURCHASE_REQUISITION_STATES,
                              'Status', tracking=True, required=True,
                              copy=False, default='draft')
    state_blanket_order = fields.Selection(PURCHASE_REQUISITION_STATES, compute='_set_state')
    is_quantity_copy = fields.Selection(related='type_id.quantity_copy', readonly=True)
    currency_id = fields.Many2one('res.currency', 'Currency', required=True,
        default=lambda self: self.env.company.currency_id.id)

    @api.depends('state')
    def _set_state(self):
        for requisition in self:
            requisition.state_blanket_order = requisition.state

    @api.onchange('vendor_id')
    def _onchange_vendor(self):
        self = self.with_company(self.company_id)
        if not self.vendor_id:
            self.currency_id = self.env.company.currency_id.id
        else:
            self.currency_id = self.vendor_id.property_purchase_currency_id.id or self.env.company.currency_id.id

        requisitions = self.env['purchase.requisition'].search([
            ('vendor_id', '=', self.vendor_id.id),
            ('state', '=', 'ongoing'),
            ('type_id.quantity_copy', '=', 'none'),
            ('company_id', '=', self.company_id.id),
        ])
        if any(requisitions):
            title = _("Warning for %s", self.vendor_id.name)
            message = _("There is already an open blanket order for this supplier. We suggest you complete this open blanket order, instead of creating a new one.")
            warning = {
                'title': title,
                'message': message
            }
            return {'warning': warning}

    @api.depends('purchase_ids')
    def _compute_orders_number(self):
        for requisition in self:
            requisition.order_count = len(requisition.purchase_ids)

    def action_cancel(self):
        # try to set all associated quotations to cancel state
        for requisition in self:
            for requisition_line in requisition.line_ids:
                requisition_line.supplier_info_ids.unlink()
            requisition.purchase_ids.button_cancel()
            for po in requisition.purchase_ids:
                po.message_post(body=_('Cancelled by the agreement associated to this quotation.'))
        self.write({'state': 'cancel'})

    def action_in_progress(self):
        self.ensure_one()
        if not self.line_ids:
            raise UserError(_("You cannot confirm agreement '%s' because there is no product line.", self.name))
        if self.type_id.quantity_copy == 'none' and self.vendor_id:
            for requisition_line in self.line_ids:
                if requisition_line.price_unit <= 0.0:
                    raise UserError(_('You cannot confirm the blanket order without price.'))
                if requisition_line.product_qty <= 0.0:
                    raise UserError(_('You cannot confirm the blanket order without quantity.'))
                requisition_line.create_supplier_info()
            self.write({'state': 'ongoing'})
        else:
            self.write({'state': 'in_progress'})
        # Set the sequence number regarding the requisition type
        if self.name == 'New':
            if self.is_quantity_copy != 'none':
                self.name = self.env['ir.sequence'].next_by_code('purchase.requisition.purchase.tender')
            else:
                self.name = self.env['ir.sequence'].next_by_code('purchase.requisition.blanket.order')

    def action_open(self):
        self.write({'state': 'open'})

    def action_draft(self):
        self.ensure_one()
        self.name = 'New'
        self.write({'state': 'draft'})

    def action_done(self):
        """
        Generate all purchase order based on selected lines, should only be called on one agreement at a time
        """
        if any(purchase_order.state in ['draft', 'sent', 'to approve'] for purchase_order in self.mapped('purchase_ids')):
            raise UserError(_('You have to cancel or validate every RfQ before closing the purchase requisition.'))
        for requisition in self:
            for requisition_line in requisition.line_ids:
                requisition_line.supplier_info_ids.unlink()
        self.write({'state': 'done'})

    def unlink(self):
        if any(requisition.state not in ('draft', 'cancel') for requisition in self):
            raise UserError(_('You can only delete draft requisitions.'))
        # Draft requisitions could have some requisition lines.
        self.mapped('line_ids').unlink()
        return super(PurchaseRequisition, self).unlink()


class PurchaseRequisitionLine(models.Model):
    _name = "purchase.requisition.line"
    _description = "Purchase Requisition Line"
    _rec_name = 'product_id'

    product_id = fields.Many2one('product.product', string='Product', domain=[('purchase_ok', '=', True)], required=True)
    product_uom_id = fields.Many2one('uom.uom', string='Product Unit of Measure', domain="[('category_id', '=', product_uom_category_id)]")
    product_uom_category_id = fields.Many2one(related='product_id.uom_id.category_id')
    product_qty = fields.Float(string='Quantity', digits='Product Unit of Measure')
    product_description_variants = fields.Char('Custom Description')
    price_unit = fields.Float(string='Unit Price', digits='Product Price')
    qty_ordered = fields.Float(compute='_compute_ordered_qty', string='Ordered Quantities')
    requisition_id = fields.Many2one('purchase.requisition', required=True, string='Purchase Agreement', ondelete='cascade')
    company_id = fields.Many2one('res.company', related='requisition_id.company_id', string='Company', store=True, readonly=True, default= lambda self: self.env.company)
    account_analytic_id = fields.Many2one('account.analytic.account', string='Analytic Account', store=True, compute='_compute_account_analytic_id', readonly=False)
    analytic_tag_ids = fields.Many2many('account.analytic.tag', string='Analytic Tags', store=True, compute='_compute_analytic_tag_ids', readonly=False)
    schedule_date = fields.Date(string='Scheduled Date')
    supplier_info_ids = fields.One2many('product.supplierinfo', 'purchase_requisition_line_id')

    @api.model
    def create(self,vals):
        res = super(PurchaseRequisitionLine, self).create(vals)
        if res.requisition_id.state not in ['draft', 'cancel', 'done'] and res.requisition_id.is_quantity_copy == 'none':
            supplier_infos = self.env['product.supplierinfo'].search([
                ('product_id', '=', vals.get('product_id')),
                ('name', '=', res.requisition_id.vendor_id.id),
            ])
            if not any(s.purchase_requisition_id for s in supplier_infos):
                res.create_supplier_info()
            if vals['price_unit'] <= 0.0:
                raise UserError(_('You cannot confirm the blanket order without price.'))
        return res

    def write(self, vals):
        res = super(PurchaseRequisitionLine, self).write(vals)
        if 'price_unit' in vals:
            if vals['price_unit'] <= 0.0 and any(
                    requisition.state not in ['draft', 'cancel', 'done'] and
                    requisition.is_quantity_copy == 'none' for requisition in self.mapped('requisition_id')):
                raise UserError(_('You cannot confirm the blanket order without price.'))
            # If the price is updated, we have to update the related SupplierInfo
            self.supplier_info_ids.write({'price': vals['price_unit']})
        return res

    def unlink(self):
        to_unlink = self.filtered(lambda r: r.requisition_id.state not in ['draft', 'cancel', 'done'])
        to_unlink.mapped('supplier_info_ids').unlink()
        return super(PurchaseRequisitionLine, self).unlink()

    def create_supplier_info(self):
        purchase_requisition = self.requisition_id
        if purchase_requisition.type_id.quantity_copy == 'none' and purchase_requisition.vendor_id:
            # create a supplier_info only in case of blanket order
            self.env['product.supplierinfo'].create({
                'name': purchase_requisition.vendor_id.id,
                'product_id': self.product_id.id,
                'product_tmpl_id': self.product_id.product_tmpl_id.id,
                'price': self.price_unit,
                'currency_id': self.requisition_id.currency_id.id,
                'purchase_requisition_line_id': self.id,
            })

    @api.depends('requisition_id.purchase_ids.state')
    def _compute_ordered_qty(self):
        line_found = set()
        for line in self:
            total = 0.0
            for po in line.requisition_id.purchase_ids.filtered(lambda purchase_order: purchase_order.state in ['purchase', 'done']):
                for po_line in po.order_line.filtered(lambda order_line: order_line.product_id == line.product_id):
                    if po_line.product_uom != line.product_uom_id:
                        total += po_line.product_uom._compute_quantity(po_line.product_qty, line.product_uom_id)
                    else:
                        total += po_line.product_qty
            if line.product_id not in line_found :
                line.qty_ordered = total
                line_found.add(line.product_id)
            else:
                line.qty_ordered = 0

    @api.depends('product_id', 'schedule_date')
    def _compute_account_analytic_id(self):
        for line in self:
            default_analytic_account = line.env['account.analytic.default'].sudo().account_get(
                product_id=line.product_id.id,
                partner_id=line.requisition_id.vendor_id.id,
                user_id=line.env.uid,
                date=line.schedule_date,
                company_id=line.company_id.id,
            )
            line.account_analytic_id = default_analytic_account.analytic_id

    @api.depends('product_id', 'schedule_date')
    def _compute_analytic_tag_ids(self):
        for line in self:
            default_analytic_account = line.env['account.analytic.default'].sudo().account_get(
                product_id=line.product_id.id,
                partner_id=line.requisition_id.vendor_id.id,
                user_id=line.env.uid,
                date=line.schedule_date,
                company_id=line.company_id.id,
            )
            line.analytic_tag_ids = default_analytic_account.analytic_tag_ids

    @api.onchange('product_id')
    def _onchange_product_id(self):
        if self.product_id:
            self.product_uom_id = self.product_id.uom_po_id
            self.product_qty = 1.0
        if not self.schedule_date:
            self.schedule_date = self.requisition_id.schedule_date

    def _prepare_purchase_order_line(self, name, product_qty=0.0, price_unit=0.0, taxes_ids=False):
        self.ensure_one()
        requisition = self.requisition_id
        if self.product_description_variants:
            name += '\n' + self.product_description_variants
        if requisition.schedule_date:
            date_planned = datetime.combine(requisition.schedule_date, time.min)
        else:
            date_planned = datetime.now()
        return {
            'name': name,
            'product_id': self.product_id.id,
            'product_uom': self.product_id.uom_po_id.id,
            'product_qty': product_qty,
            'price_unit': price_unit,
            'taxes_id': [(6, 0, taxes_ids)],
            'date_planned': date_planned,
            'account_analytic_id': self.account_analytic_id.id,
            'analytic_tag_ids': self.analytic_tag_ids.ids,
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import purchase
from . import product
from . import purchase_requisition

```

## File: report\purchase_requisition_report.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="action_report_purchase_requisitions" model="ir.actions.report">
        <field name="name">Call for Tenders</field>
        <field name="model">purchase.requisition</field>
        <field name="report_type">qweb-pdf</field>
        <field name="report_name">purchase_requisition.report_purchaserequisitions</field>
        <field name="report_file">purchase_requisition.report.report_purchaserequisitions</field>
        <field name="print_report_name">'Tender - %s' % (object.name)</field>
        <field name="binding_model_id" ref="model_purchase_requisition"/>
        <field name="binding_type">report</field>
    </record>
</odoo>

```

## File: report\report_purchaserequisition.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data>
<template id="report_purchaserequisitions">
    <t t-call="web.html_container">
        <t t-foreach="docs" t-as="o">
            <t t-call="web.external_layout">
                <div class="page">
                    <div class="oe_structure"/>

                    <h2>Call for Tenders <span t-field="o.name"/></h2>

                    <div class="row mt32 mb32">
                        <div class="col-3">
                            <strong>Call for Tender Reference:</strong><br/>
                            <span t-field="o.name"/>
                        </div>
                        <div class="col-3">
                            <strong>Scheduled Ordering Date:</strong><br/>
                            <span t-field="o.ordering_date"/>
                        </div>
                        <div class="col-3">
                            <strong>Selection Type:</strong><br/>
                            <span t-esc="o.type_id.name">Multiple Requisitions</span>
                        </div>
                        <div class="col-3">
                            <strong>Source:</strong><br/>
                            <span t-field="o.origin"/>
                        </div>
                    </div>

                    <t t-if="o.line_ids">
                        <h3>Products</h3>
                        <table class="table table-sm">
                            <thead>
                                <tr>
                                    <th><strong>Description</strong></th>
                                    <th class="text-right"><strong>Qty</strong></th>
                                    <th class="text-center" groups="uom.group_uom">
                                        <strong>Product UoM</strong>
                                    </th>
                                    <th class="text-right"><strong>Scheduled Date</strong></th>
                                </tr>
                            </thead>
                            <tbody>
                                <tr t-foreach="o.line_ids" t-as="line_ids">
                                    <td>
                                        <t t-if="line_ids.product_id.code"><!--internal reference exists-->
                                            [ <span t-field="line_ids.product_id.code"/> ]
                                        </t>
                                        <span t-field="line_ids.product_id.name"/>
                                    </td>
                                    <td class="text-right">
                                        <span t-field="line_ids.product_qty"/>
                                    </td>
                                    <t>
                                        <td class="text-center" groups="uom.group_uom">
                                            <span t-field="line_ids.product_uom_id.category_id.name"/>
                                        </td>
                                    </t>
                                    <td class="text-right">
                                        <span t-field="line_ids.schedule_date"/>
                                    </td>
                                </tr>
                            </tbody>
                        </table>
                    </t>
                    
                    <t t-if="o.purchase_ids">
                        <h3>Requests for Quotation Details</h3>
                        <table class="table table-sm">
                            <thead>
                                <tr>
                                    <th><strong>Vendor </strong></th>
                                    <th><strong>Date</strong></th>
                                    <th class="text-right"><strong>Reference </strong></th>
                                </tr>
                            </thead>
                            <tbody>
                                <tr t-foreach="o.purchase_ids" t-as="purchase_ids">
                                    <td>
                                        <span t-field="purchase_ids.partner_id.name"/>
                                    </td>
                                    <td class="text-center">
                                        <span t-field="purchase_ids.date_order"/>
                                    </td>
                                    <td class="text-right">
                                        <span t-field="purchase_ids.name"/>
                                    </td>
                                </tr>
                            </tbody>
                        </table>
                    </t>

                    <div class="oe_structure"/>
                </div>
            </t>
        </t>
    </t>
</template>
</data>
</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_purchase_requisition_type,purchase.requisition.type,model_purchase_requisition_type,purchase.group_purchase_user,1,0,0,0
access_purchase_requisition_type_manager,purchase.requisition.type,model_purchase_requisition_type,purchase.group_purchase_manager,1,1,1,1
access_purchase_requisition,purchase.requisition,model_purchase_requisition,purchase.group_purchase_user,1,1,1,1
access_purchase_requisition_line_purchase_user,purchase.requisition.line,model_purchase_requisition_line,purchase.group_purchase_user,1,1,1,1
access_purchase_requisition_manager,purchase.requisition manager,model_purchase_requisition,purchase.group_purchase_manager,1,0,0,0
access_purchase_requisition_line_manager,purchase.requisition.line manager,model_purchase_requisition_line,purchase.group_purchase_manager,1,0,0,0

```

## File: security\purchase_requisition_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record model="ir.rule" id="purchase_requisition_comp_rule">
        <field name="name">Purchase Requisition multi-company</field>
        <field name="model_id" ref="model_purchase_requisition"/>
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>

    <record model="ir.rule" id="purchase_requisition_line_comp_rule">
        <field name="name">Purchase requisition Line multi-company</field>
        <field name="model_id" ref="model_purchase_requisition_line"/>
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>

</odoo>

```

## File: views\product_views.xml

```xml
<?xml version="1.0"?>
<odoo>

    <record id="product_supplierinfo_tree_view_inherit" model="ir.ui.view">
        <field name="name">product.template.product.form.inherit</field>
        <field name="model">product.supplierinfo</field>
        <field name="inherit_id" ref="product.product_supplierinfo_tree_view"/>
        <field name="arch" type="xml">
          <xpath expr="//field[@name='product_id']" position="after">
              <field name="purchase_requisition_id" readonly="1" optional="hide"/>
          </xpath>
        </field>
    </record>

    <record id="supplier_info_form_inherit" model="ir.ui.view">
        <field name="name">product.supplierinfo.requisition.view</field>
        <field name="model">product.supplierinfo</field>
        <field name="priority">20</field>
        <field name="inherit_id" ref="product.product_supplierinfo_form_view"/>
        <field name="arch" type="xml">
            <field name="product_code" position="after">
                <field name="purchase_requisition_id"
                    attrs="{'invisible': [('purchase_requisition_id', '=', False)]}"/>
            </field>
        </field>
    </record>

    <record id="product_template_form_view_inherit" model="ir.ui.view">
        <field name="name">product.template.form.inherit</field>
        <field name="model">product.template</field>
        <field name="inherit_id" ref="purchase.view_product_supplier_inherit"/>
        <field name="arch" type="xml">
            <group name="bill" position="before">
                <group string="Reordering" attrs="{'invisible': [('type', '=', 'service')]}">
                    <field name="purchase_requisition" widget="radio"/>
                </group>
            </group>
        </field>
    </record>

    <record id="act_res_partner_2_purchase_order" model="ir.actions.act_window">
        <field name="name">Purchase orders</field>
        <field name="res_model">purchase.order</field>
        <field name="domain">[('requisition_id', '=', active_id)]</field>
        <field name="context">{'default_requisition_id': active_id}</field>
        <field name="binding_model_id" ref="model_purchase_requisition"/>
        <field name="binding_view_types">form</field>
    </record>

</odoo>

```

## File: views\purchase_requisition_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data>

    <!-- Purchase Requisition Type -->

    <record model="ir.ui.view" id="view_purchase_requisition_type_tree">
        <field name="name">purchase.requisition.type.tree</field>
        <field name="model">purchase.requisition.type</field>
        <field name="arch" type="xml">
            <tree string="Purchase Agreement Types">
                <field name="sequence" widget="handle"/>
                <field name="name"/>
                <field name="exclusive"/>
            </tree>
        </field>
    </record>

    <record id="view_purchase_requisition_type_kanban" model="ir.ui.view">
        <field name="name">purchase.requisition.type.kanban</field>
        <field name="model">purchase.requisition.type</field>
        <field name="arch" type="xml">
            <kanban class="o_kanban_mobile">
                <field name="name"/>
                <field name="exclusive"/>
                <templates>
                    <t t-name="kanban-box">
                        <div t-attf-class="oe_kanban_global_click">
                            <div class="o_kanban_record_top ">
                                <div class="o_kanban_record_headings mt4">
                                    <strong class="o_kanban_record_title"><field name="name"/></strong>
                                </div>
                                <field name="exclusive" widget="label_selection"/>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>
    <record model="ir.ui.view" id="view_purchase_requisition_type_form">
        <field name="name">purchase.requisition.type.form</field>
        <field name="model">purchase.requisition.type</field>
        <field name="arch" type="xml">
            <form string="Purchase Agreement Types">
            <sheet>
                <group>
                    <group string="Agreement Type">
                        <field name="name"/>
                        <field name="exclusive" widget="radio"/>
                    </group>
                    <group string="Data for new quotations">
                        <field name="line_copy" widget="radio"/>
                        <field name="quantity_copy" widget="radio"/>
                    </group>
                </group>
            </sheet>
            </form>
        </field>
    </record>
    <record id="tender_type_action" model="ir.actions.act_window">
        <field name="name">Purchase Agreement Types</field>
        <field name="res_model">purchase.requisition.type</field>
        <field name="context">{}</field>
        <field name="view_mode">tree,kanban,form</field>
    </record>
    <menuitem
        id="menu_purchase_requisition_type"
        sequence="2"
        parent="purchase.menu_purchase_config"
        action="tender_type_action"/>

    <!-- Purchase Orders -->

    <record model="ir.actions.act_window" id="action_purchase_requisition_to_so">
        <field name="name">Request for Quotation</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">purchase.order</field>
        <field name="view_mode">form,tree</field>
        <field name="domain">[('requisition_id','=',active_id)]</field>
        <field name="context">{
            "default_requisition_id":active_id,
            "default_user_id": False,
            }
        </field>
    </record>
    <record model="ir.actions.act_window" id="action_purchase_requisition_list">
        <field name="name">Request for Quotations</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">purchase.order</field>
        <field name="view_mode">tree,form</field>
        <field name="domain">[('requisition_id','=',active_id)]</field>
        <field name="context">{
            "default_requisition_id":active_id,
            "default_user_id": False,
            }
        </field>
    </record>

    <record model="ir.ui.view" id="view_purchase_requisition_form">
        <field name="name">purchase.requisition.form</field>
        <field name="model">purchase.requisition</field>
        <field name="arch" type="xml">
            <form string="Purchase Agreements">
            <header>
                <button name="%(action_purchase_requisition_to_so)d" type="action"
                    string="New Quotation"
                    attrs="{'invisible': [('state', '!=', 'open')]}"/>
                <button name="%(action_purchase_requisition_to_so)d" type="action"
                    string="New Quotation" class="btn-primary"
                    attrs="{'invisible': [('state', 'not in', ('in_progress', 'ongoing'))]}"/>
                <button name="action_in_progress" states="draft" string="Confirm" type="object" class="btn-primary"/>
                <button name="action_open" states="in_progress" string="Validate" type="object" class="btn-primary"/>
                <button name="action_done" states="open,ongoing" string="Close" type="object" class="btn-primary"/>
                <button name="action_draft" states="cancel" string="Reset to Draft" type="object"/>
                <button name="action_cancel" states="draft,in_progress,ongoing" string="Cancel" type="object"/>
                <field name="state" widget="statusbar" statusbar_visible="draft,in_progress,open,done" attrs="{'invisible': [('is_quantity_copy', '=', 'none')]}"/>
                <field name="state_blanket_order" widget="statusbar" statusbar_visible="draft,ongoing,done" attrs="{'invisible': [('is_quantity_copy', '!=', 'none')]}"/>
            </header>
            <sheet>
                <div class="oe_button_box" name="button_box">
                    <button name="%(action_purchase_requisition_list)d" type="action" class="oe_stat_button" icon="fa-list-alt"
                        attrs="{'invisible': [('state', '=', 'draft')]}">
                        <field name="order_count" widget="statinfo" string="RFQs/Orders"/>
                    </button>
                </div>
                <div class="float-left">
                    <label for="name" class="oe_edit_only oe_inline"/>
                    <h1>
                        <field name="name"/>
                    </h1>
                </div>
                <group>
                    <group>
                        <field name="is_quantity_copy" invisible='1'/>
                        <field name="user_id" attrs="{'readonly': [('state','not in',('draft','in_progress','open'))]}" domain="[('share', '=', False)]"/>
                        <field name="type_id" attrs="{'readonly': [('state','!=','draft')]}"/>
                        <field name="vendor_id" context="{'res_partner_search_mode': 'supplier'}" attrs="{'required': [('is_quantity_copy', '=', 'none')], 'readonly': [('state', 'in', ['ongoing','done'])]}"/>
                        <field name="currency_id" groups="base.group_multi_currency"/>
                    </group>
                    <group>
                        <field name="date_end" attrs="{'readonly': [('state','not in',('draft','in_progress','open','ongoing'))]}"/>
                        <field name="ordering_date" attrs="{'readonly': [('state','not in',('draft','in_progress','open','ongoing'))]}"/>
                        <field name="schedule_date" attrs="{'readonly': [('state','not in',('draft','in_progress','open','ongoing'))]}"/>
                        <field name="origin" placeholder="e.g. PO0025" attrs="{'readonly': [('state', '!=', 'draft')]}"/>
                        <field name="company_id" groups="base.group_multi_company" options="{'no_create': True}" attrs="{'readonly': [('state','not in',('draft'))]}"/>
                    </group>
                </group>
                <notebook>
                    <page string="Products" name="products">
                        <field name="line_ids">
                            <tree string="Products" editable="bottom">
                                <field name="product_id" context="{'default_purchase_requisition': 'tenders'}"
                                       domain="[('purchase_ok', '=', True), '|', ('company_id', '=', False), ('company_id', '=', parent.company_id)]"/>
                                <field name="product_description_variants" attrs="{'invisible': [('product_description_variants', '=', '')], 'readonly': [('parent.state', '!=', 'draft')]}"/>
                                <field name="product_qty"/>
                                <field name="qty_ordered" optional="show"/>
                                <field name="product_uom_category_id" invisible="1"/>
                                <field name="product_uom_id" string="UoM" groups="uom.group_uom" optional="show" attrs="{'required': [('product_id', '!=', False)]}"/>
                                <field name="schedule_date" groups="base.group_no_one"/>
                                <field name="account_analytic_id" optional="hide" domain="['|', ('company_id', '=', False), ('company_id', '=', parent.company_id)]" groups="analytic.group_analytic_accounting"/>
                                <field name="analytic_tag_ids" optional="hide" domain="['|', ('company_id', '=', False), ('company_id', '=', parent.company_id)]" groups="analytic.group_analytic_tags" widget="many2many_tags"/>
                                <field name="price_unit"/>
                            </tree>
                            <form string="Products">
                                <group>
                                    <field name="product_id" context="{'default_purchase_requisition': 'tenders'}"
                                           domain="[('purchase_ok', '=', True), '|', ('company_id', '=', False), ('company_id', '=', parent.company_id)]" />
                                    <field name="product_qty"/>
                                    <field name="qty_ordered"/>
                                    <field name="product_uom_category_id" invisible="1"/>
                                    <field name="product_uom_id" />
                                    <field name="schedule_date"/>
                                    <field name="account_analytic_id"  domain="['|', ('company_id', '=', False), ('company_id', '=', parent.company_id)]" groups="analytic.group_analytic_accounting"/>
                                    <field name="company_id" groups="base.group_multi_company" options="{'no_create': True}"/>
                                </group>
                            </form>
                        </field>
                        <separator string="Terms and Conditions"/>
                        <field name="description" attrs="{'readonly': [('state','not in',('draft','in_progress','open'))]}"/>
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
    <record model="ir.ui.view" id="view_purchase_requisition_tree">
        <field name="name">purchase.requisition.tree</field>
        <field name="model">purchase.requisition</field>
        <field name="arch" type="xml">
            <tree string="Purchase Agreements" sample="1">
                <field name="message_needaction" invisible="1"/>
                <field name="name" decoration-bf="1"/>
                <field name="user_id" optional="show" widget='many2one_avatar_user'/>
                <field name="company_id" groups="base.group_multi_company" options="{'no_create': True}" optional="show"/>
                <field name="ordering_date" optional="show"/>
                <field name="schedule_date" optional="hide"/>
                <field name="date_end" optional="show" widget='remaining_days' decoration-danger="date_end and date_end&lt;current_date" attrs="{'invisible': [('state','in', ('done', 'cancel'))]}"/>
                <field name="origin" optional="show"/>
                <field name="state" optional="show" widget='badge' decoration-success="state == 'done'" decoration-info="state not in ('done', 'cancel')"/>
                <field name="activity_exception_decoration" widget="activity_exception"/>
            </tree>
      </field>
    </record>

    <record id="view_purchase_requisition_kanban" model="ir.ui.view">
        <field name="name">purchase.requisition.kanban</field>
        <field name="model">purchase.requisition</field>
        <field name="arch" type="xml">
            <kanban class="o_kanban_mobile" sample="1">
                <field name="name"/>
                <field name="state"/>
                <field name="user_id"/>
                <field name="type_id"/>
                <templates>
                    <t t-name="kanban-box">
                        <div t-attf-class="oe_kanban_card oe_kanban_global_click">
                            <div class="o_kanban_record_top">
                                <div class="o_kanban_record_headings mt4">
                                    <strong class="o_kanban_record_title"><span><field name="name"/></span></strong>
                                </div>
                                <field name="state" widget="label_selection" options="{'classes': {'draft': 'default', 'in_progress': 'default', 'open': 'success', 'done': 'success', 'close': 'danger'}}" readonly="1"/>
                            </div>
                            <div class="o_kanban_record_body">
                                <span class="text-muted"><field name="type_id"/></span>
                            </div>
                            <div class="o_kanban_record_bottom">
                                <div class="oe_kanban_bottom_left">
                                    <field name="vendor_id"/>
                                </div>
                                <div class="oe_kanban_bottom_right">
                                    <field name="user_id" widget="many2one_avatar_user"/>
                                </div>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="view_purchase_requisition_filter" model="ir.ui.view">
            <field name="name">purchase.requisition.list.select</field>
            <field name="model">purchase.requisition</field>
            <field name="arch" type="xml">
                <search string="Search Purchase Agreements">
                    <field name="name" string="Reference" filter_domain="['|', ('name', 'ilike', self), ('origin', 'ilike', self)]"/>
                    <field name="user_id"/>
                    <field name="product_id"/>
                    <filter string="My Agreements" name="my_agreements" domain="[('user_id', '=', uid)]"/>
                    <separator/>
                    <filter string="Draft" name="draft" domain="[('state', '=', 'draft')]" help="New Agreements"/>
                    <filter string="Confirmed" name="confirmed" domain="[('state', 'in', ('in_progress', 'open'))]" help="In negotiation"/>
                    <filter string="Done" name="done" domain="[('state', '=', 'done')]"/>
                    <separator/>
                    <filter invisible="1" string="Late Activities" name="activities_overdue"
                        domain="[('my_activity_date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                        help="Show all records which has next action date is before today"/>
                    <filter invisible="1" string="Today Activities" name="activities_today"
                        domain="[('my_activity_date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                    <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                        domain="[('my_activity_date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
                    <group expand="0" string="Group By">
                        <filter string="Purchase Representative" name="representative" domain="[]" context="{'group_by': 'user_id'}"/>
                        <filter string="Status" name="status" domain="[]" context="{'group_by': 'state'}"/>
                        <filter string="Ordering Date" name="ordering_date" domain="[]" context="{'group_by': 'ordering_date'}"/>
                    </group>
                </search>
            </field>
        </record>


    <record model="ir.actions.act_window" id="action_purchase_requisition">
        <field name="name">Purchase Agreements</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">purchase.requisition</field>
        <field name="view_mode">tree,kanban,form</field>
        <field name="context">{}</field>
        <field name="search_view_id" ref="view_purchase_requisition_filter"/>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Start a new purchase agreement
          </p><p>
            Example of purchase agreements include call for tenders and blanket orders.
          </p><p>
            In a call for tenders, you can record the products you need to buy
            and generate the creation of RfQs to vendors. Once the tenders have
            been registered, you can review and compare them and you can
            validate some and cancel others.
          </p><p>
            For a blanket order, you can record an agreement for a specifc period
            (e.g. a year) and you order products within this agreement, benefiting
            from the negotiated prices.
          </p>
        </field>
    </record>

    <menuitem
        id="menu_purchase_requisition_pro_mgt"
        sequence="10"
        parent="purchase.menu_procurement_management"
        action="action_purchase_requisition"/>

    </data>
</odoo>

```

## File: views\purchase_views.xml

```xml
<?xml version="1.0"?>
<odoo>

    <record id="purchase_order_form_inherit" model="ir.ui.view">
        <field name="name">purchase.order.form.inherit</field>
        <field name="model">purchase.order</field>
        <field name="inherit_id" ref="purchase.purchase_order_form"/>
        <field name="arch" type="xml">
            <field name="partner_id" position="replace">
                <field name="is_quantity_copy" invisible="1"/>
                <field name="partner_id" widget="res_partner_many2one" context="{'res_partner_search_mode': 'supplier', 'show_vat': True}" attrs="{'readonly': ['|', ('is_quantity_copy', '=', 'none'), ('state', 'in', ['purchase', 'done', 'cancel'])]}" placeholder="Name, TIN, Email, or Reference" force_save="1"/>
                </field>
                <field name="partner_ref" position="after">
                <field name="requisition_id" domain="[('state', 'in', ('in_progress', 'open', 'ongoing')), ('vendor_id', 'in', (partner_id, False)), ('company_id', '=', company_id)]"/>
            </field>
        </field>
    </record>

    <record id="purchase_order_search_inherit" model="ir.ui.view">
        <field name="name">purchase.order.list.select.inherit</field>
        <field name="model">purchase.order</field>
        <field name="inherit_id" ref="purchase.view_purchase_order_filter"/>
        <field name="arch" type="xml">
            <xpath expr="//filter[@name='approved']" position="after">
                <filter string="Requisition" name="requisition" domain="[('requisition_id', '!=', False)]"  help="Purchase Orders with requisition"/>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.purchase.requisition</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="purchase.res_config_settings_view_form_purchase"/>
        <field name="arch" type="xml">
            <div id="use_purchase_requisition" position="replace">
                <div class="mt8">
                    <button name="%(purchase_requisition.tender_type_action)d" icon="fa-arrow-right" type="action" string="Agreement Types" class="btn-link"/>
                </div>
            </div>
        </field>
    </record>
</odoo>

```

