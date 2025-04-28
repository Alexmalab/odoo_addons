# Odoo Module: purchase_requisition

Category: Inventory/Purchase

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import wizard

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
        'report/purchase_requisition_report.xml',
        'report/report_purchaserequisition.xml',
        'wizard/purchase_requisition_alternative_warning.xml',
        'wizard/purchase_requisition_create_alternative.xml',
    ],
    'assets': {
        'web.assets_backend': [
            'purchase_requisition/static/src/*/**.js',
            'purchase_requisition/static/src/views/*/**.js',
            'purchase_requisition/static/src/*/**.scss',
            'purchase_requisition/static/src/*/**.xml',
        ],
    },
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
        <!--Blanket Order Agreement-->
        <record id="bo_requisition" model="purchase.requisition">
            <field name="user_id" ref="base.user_admin"/>
            <field name="vendor_id" ref="base.res_partner_1"/>
            <field name="type_id" ref="type_single"/>
        </record>

        <record id="bo_requisition_line" model="purchase.requisition.line">
            <field name="requisition_id" ref="bo_requisition"/>
            <field name="product_id" ref="product.product_product_13"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="product_qty">100</field>
            <field name="price_unit">60</field>
        </record>
        <function model="purchase.requisition" name="action_in_progress" eval="[[ref('bo_requisition')]]"/>

        <!--Resource: purchase.order-->
        <record id="rfq1" model="purchase.order">
            <field name="partner_id" ref="base.res_partner_1"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="requisition_id" ref="bo_requisition"/>
        </record>

        <record id="rfq1_line" model="purchase.order.line">
            <field name="order_id" ref="rfq1"/>
            <field name="name" model="purchase.order.line" eval="obj().env.ref('product.product_product_13').partner_ref"/>
            <field name="date_planned" eval="time.strftime('%Y-%m-10')"/>
            <field name="product_id" ref="product.product_product_13"/>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">60</field>
            <field name="product_qty">25</field>
        </record>

        <function model="purchase.order" name="button_confirm" eval="[[ref('rfq1')]]"/>

        <record id="rfq2" model="purchase.order">
            <field name="partner_id" ref="base.res_partner_1"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="requisition_id" ref="bo_requisition"/>
        </record>

        <record id="rfq2_line" model="purchase.order.line">
            <field name="order_id" ref="rfq2"/>
            <field name="name" model="purchase.order.line" eval="obj().env.ref('product.product_product_13').partner_ref"/>
            <field name="date_planned" eval="time.strftime('%Y-%m-15')"/>
            <field name="product_id" ref="product.product_product_13"/>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">60</field>
            <field name="product_qty">10</field>
        </record>

        <!-- Call to Tender, i.e. linked RFQs -->
        <!-- Note that forcecreate="0" is added to this demo data due to product type being updated to 'product'
             by stock, but upgrade issue will occur since 'product' type won't exist yet in purchase when trying
             to add the new PO lines (i.e. stock product type update won't exist yet, but it will be in the db) -->
        <record id="rfq3" model="purchase.order" forcecreate="0">
            <field name="partner_id" ref="base.res_partner_1"/>
            <field name="user_id" ref="base.user_admin"/>
        </record>
        <record id="rfq3_line" model="purchase.order.line" forcecreate="0">
            <field name="order_id" ref="rfq3"/>
            <field name="name" model="purchase.order.line" eval="obj().env.ref('product.product_product_13').partner_ref"/>
            <field name="date_planned" eval="time.strftime('%Y-%m-10')"/>
            <field name="product_id" ref="product.product_product_13"/>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">100</field>
            <field name="product_qty">5</field>
        </record>

        <record id="rfq4" model="purchase.order" forcecreate="0">
            <field name="partner_id" ref="base.res_partner_2"/>
            <field name="user_id" ref="base.user_admin"/>
        </record>
        <record id="rfq4_line" model="purchase.order.line" forcecreate="0">
            <field name="order_id" ref="rfq4"/>
            <field name="name" model="purchase.order.line" eval="obj().env.ref('product.product_product_13').partner_ref"/>
            <field name="date_planned" eval="time.strftime('%Y-%m-15')"/>
            <field name="product_id" ref="product.product_product_13"/>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">120</field>
            <field name="product_qty">4</field>
        </record>
        <record id="rfq_group" model="purchase.order.group" forcecreate="0">
            <field name="order_ids" eval="[Command.set([ref('purchase_requisition.rfq3'), ref('purchase_requisition.rfq4')])]"/>
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

    purchase_requisition_id = fields.Many2one('purchase.requisition', related='purchase_requisition_line_id.requisition_id', string='Agreement')
    purchase_requisition_line_id = fields.Many2one('purchase.requisition.line')


class ProductProduct(models.Model):
    _inherit = 'product.product'

    def _prepare_sellers(self, params=False):
        sellers = super(ProductProduct, self)._prepare_sellers(params=params)
        if params and params.get('order_id'):
            return sellers.filtered(lambda s: not s.purchase_requisition_id or s.purchase_requisition_id == params['order_id'].requisition_id)
        else:
            return sellers

```

## File: models\purchase.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict

from odoo import api, fields, models, _, Command
from odoo.tools import DEFAULT_SERVER_DATETIME_FORMAT, get_lang


class PurchaseOrderGroup(models.Model):
    _name = 'purchase.order.group'
    _description = "Technical model to group PO for call to tenders"

    order_ids = fields.One2many('purchase.order', 'purchase_group_id')

    def write(self, vals):
        res = super().write(vals)
        # when len(POs) == 1, only linking PO to itself at this point => self implode (delete) group
        self.filtered(lambda g: len(g.order_ids) <= 1).unlink()
        return res


class PurchaseOrder(models.Model):
    _inherit = 'purchase.order'

    requisition_id = fields.Many2one('purchase.requisition', string='Blanket Order', copy=False)
    is_quantity_copy = fields.Selection(related='requisition_id.is_quantity_copy', readonly=False)

    purchase_group_id = fields.Many2one('purchase.order.group')
    alternative_po_ids = fields.One2many(
        'purchase.order', related='purchase_group_id.order_ids', readonly=False,
        domain="[('id', '!=', id), ('state', 'in', ['draft', 'sent', 'to approve'])]",
        string="Alternative POs", check_company=True,
        help="Other potential purchase orders for purchasing products")
    has_alternatives = fields.Boolean(
        "Has Alternatives", compute='_compute_has_alternatives',
        help="Whether or not this purchase order is linked to another purchase order as an alternative.")

    @api.depends('purchase_group_id')
    def _compute_has_alternatives(self):
        self.has_alternatives = False
        self.filtered(lambda po: po.purchase_group_id).has_alternatives = True

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
        fpos = FiscalPosition.with_company(self.company_id)._get_fiscal_position(partner)

        self.partner_id = partner.id
        self.fiscal_position_id = fpos.id
        self.payment_term_id = payment_term.id
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
        if self.alternative_po_ids and not self.env.context.get('skip_alternative_check', False):
            alternative_po_ids = self.alternative_po_ids.filtered(lambda po: po.state in ['draft', 'sent', 'to approve'] and po.id not in self.ids)
            if alternative_po_ids:
                view = self.env.ref('purchase_requisition.purchase_requisition_alternative_warning_form')
                return {
                    'name': _("What about the alternative Requests for Quotations?"),
                    'type': 'ir.actions.act_window',
                    'view_mode': 'form',
                    'res_model': 'purchase.requisition.alternative.warning',
                    'views': [(view.id, 'form')],
                    'target': 'new',
                    'context': dict(self.env.context, default_alternative_po_ids=alternative_po_ids.ids, default_po_ids=self.ids),
                }
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

    @api.model_create_multi
    def create(self, vals_list):
        orders = super().create(vals_list)
        if self.env.context.get('origin_po_id'):
            # po created as an alt to another PO:
            origin_po_id = self.env['purchase.order'].browse(self.env.context.get('origin_po_id'))
            if origin_po_id.purchase_group_id:
                origin_po_id.purchase_group_id.order_ids |= orders
            else:
                self.env['purchase.order.group'].create({'order_ids': [Command.set(origin_po_id.ids + orders.ids)]})
        for order in orders:
            if order.requisition_id:
                order.message_post_with_source(
                    'mail.message_origin_link',
                    render_values={'self': order, 'origin': order.requisition_id},
                    subtype_xmlid='mail.mt_note',
                )
        return orders

    def write(self, vals):
        if vals.get('purchase_group_id', False):
            # store in case linking to a PO with existing linkages
            orig_purchase_group = self.purchase_group_id
        result = super(PurchaseOrder, self).write(vals)
        if vals.get('requisition_id'):
            for order in self:
                order.message_post_with_source(
                    'mail.message_origin_link',
                    render_values={'self': order, 'origin': order.requisition_id, 'edit': True},
                    subtype_xmlid='mail.mt_note',
                )
        if vals.get('alternative_po_ids', False):
            if not self.purchase_group_id and len(self.alternative_po_ids + self) > len(self):
                # this can create a new group + delete an existing one (or more) when linking to already linked PO(s), but this is
                # simplier than additional logic checking if exactly 1 exists or merging multiple groups if > 1
                self.env['purchase.order.group'].create({'order_ids': [Command.set(self.ids + self.alternative_po_ids.ids)]})
            elif self.purchase_group_id and len(self.alternative_po_ids + self) <= 1:
                # write in purchase group isn't called so we have to manually unlink obsolete groups here
                self.purchase_group_id.unlink()
        if vals.get('purchase_group_id', False):
            # the write is for multiple POs => don't double count the POs of the final group
            additional_groups = orig_purchase_group - self.purchase_group_id
            if additional_groups:
                additional_pos = (additional_groups.order_ids - self.purchase_group_id.order_ids)
                additional_groups.unlink()
                if additional_pos:
                    self.purchase_group_id.order_ids |= additional_pos

        return result

    def action_create_alternative(self):
        ctx = dict(**self.env.context, default_origin_po_id=self.id)
        return {
            'name': _('Create alternative'),
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'purchase.requisition.create.alternative',
            'view_id': self.env.ref('purchase_requisition.purchase_requisition_create_alternative_form').id,
            'target': 'new',
            'context': ctx,
        }

    def action_compare_alternative_lines(self):
        ctx = dict(
            self.env.context,
            search_default_groupby_product=True,
            purchase_order_id=self.id,
        )
        view_id = self.env.ref('purchase_requisition.purchase_order_line_compare_tree').id
        return {
            'name': _('Compare Order Lines'),
            'type': 'ir.actions.act_window',
            'view_mode': 'list',
            'res_model': 'purchase.order.line',
            'views': [(view_id, "list")],
            'domain': [('order_id', 'in', (self | self.alternative_po_ids).ids), ('display_type', '=', False)],
            'context': ctx,
        }

    def get_tender_best_lines(self):
        product_to_best_price_line = defaultdict(lambda: self.env['purchase.order.line'])
        product_to_best_date_line = defaultdict(lambda: self.env['purchase.order.line'])
        product_to_best_price_unit = defaultdict(lambda: self.env['purchase.order.line'])
        po_alternatives = self | self.alternative_po_ids

        multiple_currencies = False
        if len(po_alternatives.currency_id) > 1:
            multiple_currencies = True

        for line in po_alternatives.order_line:
            if not line.product_qty or not line.price_subtotal or line.state in ['cancel', 'purchase', 'done']:
                continue

            # if no best price line => no best price unit line either
            if not product_to_best_price_line[line.product_id]:
                product_to_best_price_line[line.product_id] = line
                product_to_best_price_unit[line.product_id] = line
            else:
                price_subtotal = line.price_subtotal
                price_unit = line.price_unit
                current_price_subtotal = product_to_best_price_line[line.product_id][0].price_subtotal
                current_price_unit = product_to_best_price_unit[line.product_id][0].price_unit
                if multiple_currencies:
                    price_subtotal /= line.order_id.currency_rate
                    price_unit /= line.order_id.currency_rate
                    current_price_subtotal /= product_to_best_price_line[line.product_id][0].order_id.currency_rate
                    current_price_unit /= product_to_best_price_unit[line.product_id][0].order_id.currency_rate

                if current_price_subtotal > price_subtotal:
                    product_to_best_price_line[line.product_id] = line
                elif current_price_subtotal == price_subtotal:
                    product_to_best_price_line[line.product_id] |= line
                if current_price_unit > price_unit:
                    product_to_best_price_unit[line.product_id] = line
                elif current_price_unit == price_unit:
                    product_to_best_price_unit[line.product_id] |= line

            if not product_to_best_date_line[line.product_id] or product_to_best_date_line[line.product_id][0].date_planned > line.date_planned:
                product_to_best_date_line[line.product_id] = line
            elif product_to_best_date_line[line.product_id][0].date_planned == line.date_planned:
                product_to_best_date_line[line.product_id] |= line

        best_price_ids = set()
        best_date_ids = set()
        best_price_unit_ids = set()
        for lines in product_to_best_price_line.values():
            best_price_ids.update(lines.ids)
        for lines in product_to_best_date_line.values():
            best_date_ids.update(lines.ids)
        for lines in product_to_best_price_unit.values():
            best_price_unit_ids.update(lines.ids)
        return list(best_price_ids), list(best_date_ids), list(best_price_unit_ids)


class PurchaseOrderLine(models.Model):
    _inherit = 'purchase.order.line'

    def _compute_price_unit_and_date_planned_and_name(self):
        po_lines_without_requisition = self.env['purchase.order.line']
        for pol in self:
            if pol.product_id.id not in pol.order_id.requisition_id.line_ids.product_id.ids:
                po_lines_without_requisition |= pol
                continue
            for line in pol.order_id.requisition_id.line_ids:
                if line.product_id == pol.product_id:
                    pol.price_unit = line.product_uom_id._compute_price(line.price_unit, pol.product_uom)
                    partner = pol.order_id.partner_id or pol.order_id.requisition_id.vendor_id
                    params = {'order_id': pol.order_id}
                    seller = pol.product_id._select_seller(
                        partner_id=partner,
                        quantity=pol.product_qty,
                        date=pol.order_id.date_order and pol.order_id.date_order.date(),
                        uom_id=line.product_uom_id,
                        params=params)

                    if not pol.date_planned:
                        pol.date_planned = pol._get_date_planned(seller).strftime(DEFAULT_SERVER_DATETIME_FORMAT)

                    product_ctx = {'seller_id': seller.id, 'lang': get_lang(pol.env, partner.lang).code}
                    name = pol._get_product_purchase_description(pol.product_id.with_context(product_ctx))
                    if line.product_description_variants:
                        name += '\n' + line.product_description_variants
                    pol.name = name
                    break
        super(PurchaseOrderLine, po_lines_without_requisition)._compute_price_unit_and_date_planned_and_name()

    def action_clear_quantities(self):
        zeroed_lines = self.filtered(lambda l: l.state not in ['cancel', 'purchase', 'done'])
        zeroed_lines.write({'product_qty': 0})
        if len(self) > len(zeroed_lines):
            return {
                'type': 'ir.actions.client',
                'tag': 'display_notification',
                'params': {
                    'title': _("Some not cleared"),
                    'message': _("Some quantities were not cleared because their status is not a RFQ status."),
                    'sticky': False,
                }
            }
        return False

    def action_choose(self):
        order_lines = (self.order_id | self.order_id.alternative_po_ids).mapped('order_line')
        order_lines = order_lines.filtered(lambda l: l.product_qty and l.product_id.id in self.product_id.ids and l.id not in self.ids)
        if order_lines:
            return order_lines.action_clear_quantities()
        return {
            'type': 'ir.actions.client',
            'tag': 'display_notification',
            'params': {
                'title': _("Nothing to clear"),
                'message': _("There are no quantities to clear."),
                'sticky': False,
            }
        }

```

## File: models\purchase_requisition.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from datetime import datetime, time

from odoo import api, fields, models, _
from odoo.exceptions import UserError
from collections import defaultdict


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
    active = fields.Boolean(default=True, help="Set active to false to hide the Purchase Agreement Types without removing it.")


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
    vendor_id = fields.Many2one('res.partner', string="Vendor", check_company=True)
    type_id = fields.Many2one('purchase.requisition.type', string="Agreement Type", required=True, default=_get_type_id)
    ordering_date = fields.Date(string="Ordering Date", tracking=True)
    date_end = fields.Datetime(string='Agreement Deadline', tracking=True)
    schedule_date = fields.Date(string='Delivery Date', index=True, help="The expected and scheduled delivery date where all the products are received", tracking=True)
    user_id = fields.Many2one(
        'res.users', string='Purchase Representative',
        default=lambda self: self.env.user, check_company=True)
    description = fields.Html()
    company_id = fields.Many2one('res.company', string='Company', required=True, default=lambda self: self.env.company)
    purchase_ids = fields.One2many('purchase.order', 'requisition_id', string='Purchase Orders')
    line_ids = fields.One2many('purchase.requisition.line', 'requisition_id', string='Products to Purchase', copy=True)
    product_id = fields.Many2one('product.product', related='line_ids.product_id', string='Product')
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
                requisition_line.supplier_info_ids.sudo().unlink()
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
            self.name = self.env['ir.sequence'].with_company(self.company_id).next_by_code('purchase.requisition.blanket.order')

    def action_open(self):
        self.write({'state': 'open'})

    def action_draft(self):
        self.ensure_one()
        self.write({'state': 'draft'})

    def action_done(self):
        """
        Generate all purchase order based on selected lines, should only be called on one agreement at a time
        """
        if any(purchase_order.state in ['draft', 'sent', 'to approve'] for purchase_order in self.mapped('purchase_ids')):
            raise UserError(_("To close this purchase requisition, cancel related Requests for Quotation.\n\n"
                "Imagine the mess if someone confirms these duplicates: double the order, double the trouble :)"))
        for requisition in self:
            for requisition_line in requisition.line_ids:
                requisition_line.supplier_info_ids.sudo().unlink()
        self.write({'state': 'done'})

    @api.ondelete(at_uninstall=False)
    def _unlink_if_draft_or_cancel(self):
        if any(requisition.state not in ('draft', 'cancel') for requisition in self):
            raise UserError(_('You can only delete draft or cancelled requisitions.'))

    def unlink(self):
        # Draft requisitions could have some requisition lines.
        self.mapped('line_ids').unlink()
        return super(PurchaseRequisition, self).unlink()


class PurchaseRequisitionLine(models.Model):
    _name = "purchase.requisition.line"
    _inherit = 'analytic.mixin'
    _description = "Purchase Requisition Line"
    _rec_name = 'product_id'

    product_id = fields.Many2one('product.product', string='Product', domain=[('purchase_ok', '=', True)], required=True)
    product_uom_id = fields.Many2one(
        'uom.uom', 'Product Unit of Measure',
        compute='_compute_product_uom_id', store=True, readonly=False, precompute=True,
        domain="[('category_id', '=', product_uom_category_id)]")
    product_uom_category_id = fields.Many2one(related='product_id.uom_id.category_id')
    product_qty = fields.Float(string='Quantity', digits='Product Unit of Measure')
    product_description_variants = fields.Char('Custom Description')
    price_unit = fields.Float(string='Unit Price', digits='Product Price')
    qty_ordered = fields.Float(compute='_compute_ordered_qty', string='Ordered Quantities')
    requisition_id = fields.Many2one('purchase.requisition', required=True, string='Purchase Agreement', ondelete='cascade')
    company_id = fields.Many2one('res.company', related='requisition_id.company_id', string='Company', store=True, readonly=True)
    schedule_date = fields.Date(string='Scheduled Date')
    supplier_info_ids = fields.One2many('product.supplierinfo', 'purchase_requisition_line_id')

    @api.model_create_multi
    def create(self, vals_list):
        lines = super().create(vals_list)
        for line, vals in zip(lines, vals_list):
            if line.requisition_id.state not in ['draft', 'cancel', 'done'] and line.requisition_id.is_quantity_copy == 'none':
                supplier_infos = self.env['product.supplierinfo'].search([
                    ('product_id', '=', vals.get('product_id')),
                    ('partner_id', '=', line.requisition_id.vendor_id.id),
                ])
                if not any(s.purchase_requisition_id for s in supplier_infos):
                    line.create_supplier_info()
                if vals['price_unit'] <= 0.0:
                    raise UserError(_('You cannot confirm the blanket order without price.'))
        return lines

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
            self.env['product.supplierinfo'].sudo().create({
                'partner_id': purchase_requisition.vendor_id.id,
                'product_id': self.product_id.id,
                'product_tmpl_id': self.product_id.product_tmpl_id.id,
                'price': self.price_unit,
                'currency_id': self.requisition_id.currency_id.id,
                'purchase_requisition_line_id': self.id,
            })

    @api.depends('requisition_id.purchase_ids.state')
    def _compute_ordered_qty(self):
        line_found = defaultdict(set)
        for line in self:
            total = 0.0
            for po in line.requisition_id.purchase_ids.filtered(lambda purchase_order: purchase_order.state in ['purchase', 'done']):
                for po_line in po.order_line.filtered(lambda order_line: order_line.product_id == line.product_id):
                    if po_line.product_uom != line.product_uom_id:
                        total += po_line.product_uom._compute_quantity(po_line.product_qty, line.product_uom_id)
                    else:
                        total += po_line.product_qty
            if line.product_id not in line_found[line.requisition_id]:
                line.qty_ordered = total
                line_found[line.requisition_id].add(line.product_id)
            else:
                line.qty_ordered = 0

    @api.depends('product_id')
    def _compute_product_uom_id(self):
        for line in self:
            line.product_uom_id = line.product_id.uom_id

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
            'analytic_distribution': self.analytic_distribution,
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
        <field name="name">Purchase Agreements</field>
        <field name="model">purchase.requisition</field>
        <field name="report_type">qweb-pdf</field>
        <field name="report_name">purchase_requisition.report_purchaserequisitions</field>
        <field name="report_file">purchase_requisition.report.report_purchaserequisitions</field>
        <field name="print_report_name">'Purchase Agreement - %s' % (object.name)</field>
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
<template id="report_purchaserequisition_document">
    <t t-set="o" t-value="o.with_context(lang=o.vendor_id.lang)"/>
    <t t-call="web.external_layout">
        <t t-set="address">
            <span t-field="o.vendor_id"
                t-options='{"widget": "contact", "fields": ["address", "name", "phone"], "no_marker": True, "phone_icons": True}'/>
            <p t-if="o.vendor_id.vat"><t t-out="o.company_id.account_fiscal_country_id.vat_label or 'Tax ID'"/>: <span t-field="o.vendor_id.vat"/></p>
        </t>
        <div class="page">
            <div class="oe_structure"/>

            <h2><span t-out="o.type_id.name">Agreement</span> <span t-field="o.name">Agreement00004</span></h2>

            <div class="row my-2">
                <div class="col-3">
                    <strong><span t-out="o.type_id.name">Agreement:</span></strong><br/>
                    <span t-field="o.name">Agreement00004</span>
                </div>
                <div t-if="o.ordering_date" class="col-3">
                    <strong>Scheduled Ordering Date:</strong><br/>
                    <span t-field="o.ordering_date">2023-08-20</span>
                </div>
                <div t-if="o.date_end" class="col-3">
                    <strong>Agreement Deadline:</strong><br/>
                    <span t-field="o.date_end">2023-09-15</span>
                </div>
                <div t-if="o.origin" class="col-3">
                    <strong>Source:</strong><br/>
                    <span t-field="o.origin">Origin</span>
                </div>
            </div>

            <t t-if="o.line_ids">
                <h3>Products</h3>
                <div class="oe_structure"></div>
                <table class="table table-sm">
                    <thead>
                        <tr>
                            <th><strong>Product</strong></th>
                            <th><strong>Description</strong></th>
                            <th class="text-end"><strong>Qty</strong></th>
                            <th class="text-center" groups="uom.group_uom">
                                <strong>Product UoM</strong>
                            </th>
                            <th t-if="o.type_id.quantity_copy == 'none'">Price Unit</th>
                            <th class="text-end"><strong>Scheduled Date</strong></th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr t-foreach="o.line_ids" t-as="line_ids">
                            <td>
                                <span t-if="line_ids.product_id.code"><!--internal reference exists-->
                                    [ <span t-field="line_ids.product_id.code">Code</span> ]
                                </span>
                                <span t-field="line_ids.product_id.name">Product</span>
                            </td>
                            <td>
                                <span t-if="line_ids.product_description_variants" t-field="line_ids.product_description_variants">Product description</span>
                            </td>
                            <td class="text-end">
                                <span t-field="line_ids.product_qty">5</span>
                            </td>
                            <td class="text-center" groups="uom.group_uom">
                                <span t-field="line_ids.product_uom_id.name">Unit</span>
                            </td>
                            <td t-if="o.type_id.quantity_copy == 'none'">
                                <span t-field="line_ids.price_unit" t-options='{"widget": "monetary", "display_currency": line_ids.requisition_id.currency_id}'>$50</span>
                            </td>
                            <td class="text-end">
                                <span t-if="line_ids.schedule_date" t-field="line_ids.schedule_date">2023-08-11</span>
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
                            <th class="text-end"><strong>Date</strong></th>
                            <th class="text-end"><strong>Reference </strong></th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr t-foreach="o.purchase_ids" t-as="purchase_ids">
                            <td>
                                <span t-field="purchase_ids.partner_id.name">Vendor Name</span>
                            </td>
                            <td class="text-end">
                                <span t-field="purchase_ids.date_order">2023-08-15</span>
                            </td>
                            <td class="text-end">
                                <span t-field="purchase_ids.name">PO000042</span>
                            </td>
                        </tr>
                    </tbody>
                </table>
            </t>

            <div t-if="o.description" t-out="o.description"/>

            <div class="oe_structure"/>
        </div>
    </t>
</template>
<template id="report_purchaserequisitions">
    <t t-call="web.html_container">
        <t t-foreach="docs" t-as="o">
            <t t-call="purchase_requisition.report_purchaserequisition_document" t-lang="o.vendor_id.lang"/>
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
access_purchase_requisition_alternative_warning, purchase.requisition.alternative.warning,model_purchase_requisition_alternative_warning,purchase.group_purchase_user,1,1,1,1
access_purchase_requisition_create_alternative, purchase.requisition.create.alternative,model_purchase_requisition_create_alternative,purchase.group_purchase_user,1,1,1,1
access_purchase_requisition_purchase_order_group, purchase.order.group,model_purchase_order_group,purchase.group_purchase_user,1,1,1,1

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

## File: static\src\views\list\purchase_order_line_compare_list_renderer.js

```javascript
/** @odoo-module */

import { ListRenderer } from "@web/views/list/list_renderer";
import { onWillStart, useState, useSubEnv } from "@odoo/owl";

export class PurchaseOrderLineCompareListRenderer extends ListRenderer {
    setup() {
        super.setup();
        this.bestFields = useState({
                best_price_ids: [],
                best_date_ids: [],
                best_price_unit_ids: [],
        });
        onWillStart(async () => {
            await this.updateBestFields();
        });
        const defaultOnClickViewButton = this.env.onClickViewButton;
        useSubEnv({
            onClickViewButton: async (params) => {
                await defaultOnClickViewButton(params);
                await this.updateBestFields();
            }
        });
    }

    async updateBestFields() {
        [this.bestFields.best_price_ids,
         this.bestFields.best_date_ids,
         this.bestFields.best_price_unit_ids] = await this.props.list.model.orm.call(
            "purchase.order",
            "get_tender_best_lines",
            [this.props.list.context.purchase_order_id || this.props.list.context.active_id],
            { context: this.props.list.context }
        );
    }

    getCellClass(column, record) {
        let classNames = super.getCellClass(...arguments);
        const customClassNames = [];
        if (column.name === "price_subtotal" && this.bestFields.best_price_ids.includes(record.resId)) {
        customClassNames.push("text-success");
        }
        if (column.name === "date_planned" && this.bestFields.best_date_ids.includes(record.resId)) {
        customClassNames.push("text-success");
        }
        if (column.name === "price_unit" && this.bestFields.best_price_unit_ids.includes(record.resId)) {
        customClassNames.push("text-success");
        }
        return classNames.concat(" ", customClassNames.join(" "));
    }
}

```

## File: static\src\views\list\purchase_order_line_compare_list_view.js

```javascript
/** @odoo-module **/

import { listView } from '@web/views/list/list_view';
import { registry } from "@web/core/registry";
import { PurchaseOrderLineCompareListRenderer } from "./purchase_order_line_compare_list_renderer";


export const PurchaseOrderLineCompareListView = {
    ...listView,
    Renderer: PurchaseOrderLineCompareListRenderer,
};

registry.category("views").add("purchase_order_line_compare", PurchaseOrderLineCompareListView);

```

## File: static\src\widgets\purchase_order_alternatives_widget.js

```javascript
/** @odoo-module */

import { registry } from "@web/core/registry";
import { useService } from "@web/core/utils/hooks";
import { X2ManyField, x2ManyField } from "@web/views/fields/x2many/x2many_field";
import { ListRenderer } from "@web/views/list/list_renderer";


export class FieldMany2ManyAltPOsRenderer extends ListRenderer {
   isCurrentRecord(record) {
      return record.resId === this.props.list.model.root.resId;
  }
}

FieldMany2ManyAltPOsRenderer.recordRowTemplate = "purchase_requisition.AltPOsListRenderer.RecordRow";

export class FieldMany2ManyAltPOs extends X2ManyField {
   setup() {
      super.setup();
      this.orm = useService("orm");
      this.action = useService("action");
   }

   get isMany2Many() {
      return true;
   }

   /**
    * Override to: avoid reopening currently open record
    *              open record in same window w/breadcrumb extended
    * @override
    */
   async openRecord(record) {
      if (record.resId !== this.props.record.resId) {
         const action = await this.orm.call(record.resModel, "get_formview_action", [[record.resId]], {
               context: this.props.context,
         });
         await this.action.doAction(action);
      }
   }
}

FieldMany2ManyAltPOs.components = {
   ...X2ManyField.components,
   ListRenderer: FieldMany2ManyAltPOsRenderer,
};

export const fieldMany2ManyAltPOs = {
    ...x2ManyField,
    component: FieldMany2ManyAltPOs,
};

registry.category("fields").add("many2many_alt_pos", fieldMany2ManyAltPOs);

```

## File: static\src\widgets\purchase_order_alternatives_widget.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <!-- Ensure we can't unlink a PO from itself (i.e. confusing behavior) -->
    <t t-name="purchase_requisition.AltPOsListRenderer.RecordRow" t-inherit="web.ListRenderer.RecordRow" t-inherit-mode="primary">
        <xpath expr="//t[@t-if='displayOptionalFields or hasX2ManyAction']" position="attributes">
            <attribute name="t-if">(displayOptionalFields or hasX2ManyAction) and !isCurrentRecord(record)</attribute>
        </xpath>
    </t>

</templates>

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
              <field name="purchase_requisition_id" optional="hide"/>
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
                    invisible="not purchase_requisition_id"/>
            </field>
        </field>
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
                <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                <group>
                    <group string="Agreement Type">
                        <field name="name"/>
                        <field name="exclusive" widget="radio"/>
                        <field name="active" invisible="1"/>
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
    <record model="ir.ui.view" id="view_purchase_requisition_type_search">
        <field name="name">purchase.requisition.type.search</field>
        <field name="model">purchase.requisition.type</field>
        <field name="arch" type="xml">
            <search>
                <field name="name"/>
                <separator/>
                <filter name="archived" string="Archived" domain="[('active', '=', False)]"/>
            </search>
        </field>
    </record>

    <!-- Purchase Orders -->

    <record model="ir.actions.act_window" id="action_purchase_requisition_to_so">
        <field name="name">Request for Quotation</field>
        <field name="res_model">purchase.order</field>
        <field name="view_mode">form,tree</field>
        <field name="domain">[('requisition_id','=',active_id)]</field>
        <field name="context">{
            "default_requisition_id":active_id,
            }
        </field>
    </record>
    <record model="ir.actions.act_window" id="action_purchase_requisition_list">
        <field name="name">Request for Quotations</field>
        <field name="res_model">purchase.order</field>
        <field name="view_mode">tree,form</field>
        <field name="domain">[('requisition_id','=',active_id)]</field>
        <field name="context">{
            "default_requisition_id":active_id,
            }
        </field>
    </record>

    <record model="ir.ui.view" id="view_purchase_requisition_form">
        <field name="name">purchase.requisition.form</field>
        <field name="model">purchase.requisition</field>
        <field name="arch" type="xml">
            <form string="Purchase Agreements">
            <field name="company_id" invisible="1"/>
            <field name="currency_id" invisible="1"/>
            <header>
                <button name="%(action_purchase_requisition_to_so)d" type="action"
                    string="New Quotation"
                    context="{'default_currency_id': currency_id, 'default_user_id': user_id}"
                    invisible="state != 'open'"/>
                <button name="%(action_purchase_requisition_to_so)d" type="action"
                    string="New Quotation" class="btn-primary"
                    context="{'default_currency_id': currency_id, 'default_user_id': user_id}"
                    invisible="state not in ('in_progress', 'ongoing')"/>
                <button name="action_in_progress" invisible="state != 'draft'" string="Confirm" type="object" class="btn-primary"/>
                <button name="action_open" invisible="state != 'in_progress'" string="Validate" type="object" class="btn-primary"/>
                <button name="action_done" invisible="state not in ('open', 'ongoing')" string="Close" type="object" class="btn-primary"/>
                <button name="action_draft" invisible="state != 'cancel'" string="Reset to Draft" type="object"/>
                <button name="action_cancel" invisible="state not in ('draft', 'in_progress', 'ongoing')" string="Cancel" type="object"/>
                <field name="state" widget="statusbar" statusbar_visible="draft,in_progress,open,done" invisible="is_quantity_copy == 'none'"/>
                <field name="state_blanket_order" widget="statusbar" statusbar_visible="draft,ongoing,done" invisible="is_quantity_copy != 'none'"/>
            </header>
            <sheet>
                <div class="oe_button_box" name="button_box">
                    <button name="%(action_purchase_requisition_list)d" type="action" class="oe_stat_button" icon="fa-list-alt"
                        invisible="state == 'draft'" context="{'default_currency_id': currency_id}">
                        <field name="order_count" widget="statinfo" string="RFQs/Orders"/>
                    </button>
                </div>
                <div class="oe_title">
                    <label for="name" class="oe_inline"/>
                    <h1>
                        <field name="name"/>
                    </h1>
                </div>
                <group>
                    <group>
                        <field name="is_quantity_copy" invisible='1'/>
                        <field name="user_id" readonly="state not in ('draft', 'in_progress', 'open')" domain="[('share', '=', False)]"/>
                        <field name="type_id" readonly="state != 'draft'"/>
                        <field name="vendor_id" context="{'res_partner_search_mode': 'supplier'}" readonly="state in ['ongoing', 'done']" required="is_quantity_copy == 'none'"/>
                        <field name="currency_id" groups="base.group_multi_currency"/>
                    </group>
                    <group>
                        <field name="date_end" readonly="state not in ('draft', 'in_progress', 'open', 'ongoing')"/>
                        <field name="ordering_date" readonly="state not in ('draft', 'in_progress', 'open', 'ongoing')"/>
                        <field name="schedule_date" readonly="state not in ('draft', 'in_progress', 'open', 'ongoing')"/>
                        <field name="origin" placeholder="e.g. PO0025" readonly="state != 'draft'"/>
                        <field name="company_id" groups="base.group_multi_company" options="{'no_create': True}" readonly="state != 'draft'"/>
                    </group>
                </group>
                <notebook>
                    <page string="Products" name="products">
                        <field name="line_ids" readonly="state == 'done'">
                            <tree string="Products" editable="bottom">
                                <field name="product_id"
                                       domain="[('purchase_ok', '=', True), '|', ('company_id', '=', False), ('company_id', '=', parent.company_id)]"/>
                                <field name="product_description_variants" invisible="product_description_variants == ''"/>
                                <field name="product_qty"/>
                                <field name="qty_ordered" optional="show"/>
                                <field name="product_uom_category_id" column_invisible="True"/>
                                <field name="product_uom_id" string="UoM" groups="uom.group_uom" optional="show" required="product_id"/>
                                <field name="schedule_date" optional="hide"/>
                                <field name="analytic_distribution" widget="analytic_distribution"
                                       optional="hide"
                                       groups="analytic.group_analytic_accounting"
                                       options="{'product_field': 'product_id', 'business_domain': 'purchase_order'}"/>
                                <field name="price_unit"/>
                            </tree>
                            <form string="Products">
                                <group>
                                    <field name="product_id"
                                           domain="[('purchase_ok', '=', True), '|', ('company_id', '=', False), ('company_id', '=', parent.company_id)]" />
                                    <field name="product_qty"/>
                                    <field name="qty_ordered"/>
                                    <field name="product_uom_category_id" invisible="1"/>
                                    <field name="product_uom_id" groups="uom.group_uom"/>
                                    <field name="schedule_date"/>
                                    <field name="analytic_distribution" widget="analytic_distribution"
                                           groups="analytic.group_analytic_accounting"
                                           options="{'product_field': 'product_id', 'business_domain': 'purchase_order'}"/>
                                    <field name="company_id" groups="base.group_multi_company" options="{'no_create': True}"/>
                                </group>
                            </form>
                        </field>
                        <separator string="Terms and Conditions"/>
                        <field name="description" class="oe-bordered-editor" readonly="state not in ('draft', 'in_progress', 'open')"/>
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
                <field name="message_needaction" column_invisible="True"/>
                <field name="name" decoration-bf="1"/>
                <field name="vendor_id"/>
                <field name="user_id" optional="show" widget='many2one_avatar_user'/>
                <field name="company_id" groups="base.group_multi_company" options="{'no_create': True}" optional="show"/>
                <field name="ordering_date" optional="show"/>
                <field name="schedule_date" optional="hide"/>
                <field name="date_end" optional="show" widget='remaining_days' decoration-danger="date_end and date_end&lt;current_date" invisible="state in ('done', 'cancel')"/>
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
                    <field name="vendor_id"/>
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
        <field name="name">Blanket Orders</field>
        <field name="res_model">purchase.requisition</field>
        <field name="view_mode">tree,kanban,form</field>
        <field name="context">{}</field>
        <field name="search_view_id" ref="view_purchase_requisition_filter"/>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Start a new purchase agreement
          </p><p>
            An example of a purchase agreement is a blanket order.
          </p><p>
            For a blanket order, you can record an agreement for a specific period
            (e.g. a year) and you order products within this agreement to benefit
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
                <field name="partner_id" widget="res_partner_many2one" context="{'res_partner_search_mode': 'supplier', 'show_vat': True}" readonly="is_quantity_copy == 'none' or state in ['purchase', 'done', 'cancel']" placeholder="Name, TIN, Email, or Reference" force_save="1"/>
            </field>
            <field name="partner_ref" position="after">
                <field name="requisition_id" domain="[('state', 'in', ('in_progress', 'open', 'ongoing')), ('vendor_id', 'in', (partner_id, False)), ('company_id', '=', company_id)]"
                options="{'no_create': True}"/>
            </field>
            <xpath expr="//page[@name='purchase_delivery_invoice']" position="after">
                <page string="Alternatives" name="alternative_pos">
                    <group>
                        <group>
                            <p colspan="2">Create a call for tender by adding alternative requests for quotation to different vendors.
                            Make your choice by selecting the best combination of lead time, OTD and/or total amount.
                            By comparing product lines you can also decide to order some products from one vendor and others from another vendor.</p>
                        </group>
                        <group>
                            <p colspan="2">
                                <button name="action_create_alternative" type="object" class="btn-link d-block" string="Create Alternative" icon="fa-copy"/>
                                <button name="action_compare_alternative_lines" type="object" class="btn-link d-block" string="Compare Product Lines" icon="fa-bar-chart" invisible="not alternative_po_ids"/>
                            </p>
                        </group>
                    </group>
                    <field name="alternative_po_ids" readonly="not id" widget="many2many_alt_pos" context="{'quotation_only': True}">
                        <tree string="Alternative Purchase Order" decoration-muted="state in ['cancel', 'purchase', 'done']" decoration-bf="id == parent.id">
                            <control>
                                <create string="Link to Existing RfQ"/>
                            </control>
                            <field name="currency_id" column_invisible="1"/>
                            <field name="partner_id" readonly="state in ['cancel', 'done', 'purchase']"/>
                            <field name="name" string="Reference"/>
                            <field name="date_planned"/>
                            <field name="amount_total"/>
                            <field name="state"/>
                        </tree>
                    </field>
                </page>
            </xpath>
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

    <record id="purchase_order_kpis_tree_inherit_purchase_requisition" model="ir.ui.view">
        <field name="name">purchase.order.kpis.tree.inherit.purchase.requisition</field>
        <field name="model">purchase.order</field>
        <field name="inherit_id" ref="purchase.purchase_order_kpis_tree"/>
        <field name="arch" type="xml">
            <field name="name" position="before">
                <field name="has_alternatives" column_invisible="True"/>
                <button class="fa fa-copy"
                    title="Has Alternatives"
                    disabled="1"
                    invisible="not has_alternatives"/>
            </field>
        </field>
    </record>

    <record id="purchase_order_tree_inherit_purchase_requisition" model="ir.ui.view">
        <field name="name">purchase.order.tree.inherit.purchase.requisition</field>
        <field name="model">purchase.order</field>
        <field name="inherit_id" ref="purchase.purchase_order_tree"/>
        <field name="arch" type="xml">
            <field name="name" position="before">
                <field name="has_alternatives" column_invisible="True"/>
                <button class="fa fa-copy"
                    title="Has Alternatives"
                    disabled="1"
                    invisible="not has_alternatives"/>
            </field>
        </field>
    </record>

    <record id="purchase_order_line_compare_tree" model="ir.ui.view">
        <field name="name">purchase.order.line.compare.tree</field>
        <field name="model">purchase.order.line</field>
        <field name="priority">1000</field>
        <field name="arch" type="xml">
            <tree string="Purchase Order Lines"
            decoration-muted="state in ['cancel', 'purchase', 'done']"
                create="0" delete="0" edit="0" expand="1"
                js_class="purchase_order_line_compare">
                <header>
                    <button name="action_clear_quantities" string="Clear Selected" type="object" class="o_clear_qty_buttons"/>
                </header>
                <field name="product_id" readonly="1"/>
                <field name="partner_id" string="Vendor"/>
                <field name="order_id" string="Reference" readonly="1"/>
                <field name="state"/>
                <field name="name" readonly="1"/>
                <field name="date_planned" readonly="1"/>
                <field name="product_qty"/>
                <field name="product_uom" groups="uom.group_uom"/>
                <field name="price_unit"/>
                <field name="price_subtotal" string="Total"/>
                <field name="currency_id"/>
                <button name="action_choose" string="Choose" type="object" class="o_clear_qty_buttons" icon="fa-bullseye"
                    invisible="product_qty &lt;= 0.0"/>
                <button name="action_clear_quantities" string="Clear" type="object" class="o_clear_qty_buttons" icon="fa-times"
                    invisible="product_qty &lt;= 0.0 or state in ['cancel', 'purchase', 'done']"/>
            </tree>
        </field>
    </record>

</odoo>

```

## File: wizard\purchase_requisition_alternative_warning.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class PurchaseRequisitionAlternativeWarning(models.TransientModel):
    _name = 'purchase.requisition.alternative.warning'
    _description = 'Wizard in case PO still has open alternative requests for quotation'

    po_ids = fields.Many2many('purchase.order', 'warning_purchase_order_rel', string="POs to Confirm")
    alternative_po_ids = fields.Many2many('purchase.order', 'warning_purchase_order_alternative_rel', string="Alternative POs")

    def action_keep_alternatives(self):
        return self._action_done()

    def action_cancel_alternatives(self):
        # in theory alternative_po_ids shouldn't have any po_ids in it, but it's possible by accident/forcing it, so avoid cancelling them to be safe
        self.alternative_po_ids.filtered(lambda po: po.state in ['draft', 'sent', 'to approve'] and po.id not in self.po_ids.ids).button_cancel()
        return self._action_done()

    def _action_done(self):
        return self.po_ids.with_context({'skip_alternative_check': True}).button_confirm()

```

## File: wizard\purchase_requisition_alternative_warning.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="purchase_requisition_alternative_warning_form" model="ir.ui.view">
            <field name="name">Alternative Warning</field>
            <field name="model">purchase.requisition.alternative.warning</field>
            <field name="arch" type="xml">
                <form string="Alternative Warning">
                    <field name="alternative_po_ids" nolabel="1" readonly="1">
                        <tree create="0" delete="0" edit="0">
                            <field name="currency_id" column_invisible="1"/>
                            <field name="partner_id" readonly="state in ['cancel', 'done', 'purchase']"/>
                            <field name="name" string="Reference"/>
                            <field name="date_planned"/>
                            <field name="amount_total"/>
                            <field name="state"/>
                        </tree>
                    </field>
                    <footer>
                        <button name="action_cancel_alternatives" string="Cancel Alternatives" data-hotkey="q" type="object" colspan="1" class="btn-primary"/>
                        <button name="action_keep_alternatives" string="Keep Alternatives" data-hotkey="w" type="object" colspan="1" class="btn-primary"/>
                        <button string="Discard" data-hotkey="x" special="cancel" colspan="1" class="btn-secondary"/>
                    </footer>
                </form>
            </field>
        </record>
    </data>
</odoo>

```

## File: wizard\purchase_requisition_create_alternative.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models, Command
from odoo.exceptions import UserError


class PurchaseRequisitionCreateAlternative(models.TransientModel):
    _name = 'purchase.requisition.create.alternative'
    _description = 'Wizard to preset values for alternative PO'

    origin_po_id = fields.Many2one(
        'purchase.order', help="The original PO that this alternative PO is being created for."
    )
    partner_id = fields.Many2one(
        'res.partner', string='Vendor', required=True,
        help="Choose a vendor for alternative PO")
    creation_blocked = fields.Boolean(
        help="If the chosen vendor or if any of the products in the original PO have a blocking warning then we prevent creation of alternative PO. "
             "This is because normally these fields are cleared w/warning message within form view, but we cannot recreate that in this case.",
        compute="_compute_purchase_warn",
        groups="purchase.group_warning_purchase")
    purchase_warn_msg = fields.Text(
        'Warning Messages',
        compute="_compute_purchase_warn",
        groups="purchase.group_warning_purchase")
    copy_products = fields.Boolean(
        "Copy Products", default=True,
        help="If this is checked, the product quantities of the original PO will be copied")

    @api.depends('partner_id', 'copy_products')
    def _compute_purchase_warn(self):
        self.creation_blocked = False
        self.purchase_warn_msg = ''
        # follows partner warning logic from PurchaseOrder
        if not self.env.user.has_group('purchase.group_warning_purchase'):
            return
        partner = self.partner_id
        # If partner has no warning, check its company
        if partner and partner.purchase_warn == 'no-message':
            partner = partner.parent_id
        if partner and partner.purchase_warn != 'no-message':
            self.purchase_warn_msg = _("Warning for %(partner)s:\n%(warning_message)s\n", partner=partner.name, warning_message=partner.purchase_warn_msg)
            if partner.purchase_warn == 'block':
                self.creation_blocked = True
                self.purchase_warn_msg += _("This is a blocking warning!\n")
        if self.copy_products and self.origin_po_id.order_line:
            for line in self.origin_po_id.order_line:
                if line.product_id.purchase_line_warn != 'no-message':
                    self.purchase_warn_msg += _("Warning for %(product)s:\n%(warning_message)s\n", product=line.product_id.name, warning_message=line.product_id.purchase_line_warn_msg)
                    if line.product_id.purchase_line_warn == 'block':
                        self.creation_blocked = True
                        self.purchase_warn_msg += _("This is a blocking warning!\n")

    def action_create_alternative(self):
        if self.env.user.has_group('purchase.group_warning_purchase') and self.creation_blocked:
            raise UserError(
                _('The vendor you have selected or at least one of the products you are copying from the original '
                  'order has a blocking warning on it and cannot be selected to create an alternative.')
            )
        vals = self._get_alternative_values()
        alt_po = self.env['purchase.order'].with_context(origin_po_id=self.origin_po_id.id, default_requisition_id=False).create(vals)
        alt_po.order_line._compute_tax_id()
        return {
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'purchase.order',
            'res_id': alt_po.id,
            'context': {
                'active_id': alt_po.id,
            },
        }

    def _get_alternative_values(self):
        vals = {
            'date_order': self.origin_po_id.date_order,
            'partner_id': self.partner_id.id,
            'user_id': self.origin_po_id.user_id.id,
            'dest_address_id': self.origin_po_id.dest_address_id.id,
            'origin': self.origin_po_id.origin,
        }
        if self.copy_products and self.origin_po_id:
            vals['order_line'] = [Command.create(self._get_alternative_line_value(line)) for line in self.origin_po_id.order_line]
        return vals

    @api.model
    def _get_alternative_line_value(self, order_line):
        return {
            'product_id': order_line.product_id.id,
            'product_qty': order_line.product_qty,
            'product_uom': order_line.product_uom.id,
            'display_type': order_line.display_type,
            **({'name': order_line.name} if order_line.display_type in ('line_section', 'line_note') else {}),
        }

```

## File: wizard\purchase_requisition_create_alternative.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="purchase_requisition_create_alternative_form" model="ir.ui.view">
            <field name="name">Create Alternative</field>
            <field name="model">purchase.requisition.create.alternative</field>
            <field name="arch" type="xml">
                <form string="Create Alternative">
                    <group>
                        <field name="origin_po_id" invisible="1"/>
                        <group>
                            <field name="partner_id"/>
                        </group>
                        <group>
                            <field name="copy_products"/>
                        </group>
                        <group groups="purchase.group_warning_purchase" invisible="purchase_warn_msg == ''">
                            <field name="purchase_warn_msg"/>
                        </group>
                    </group>
                    <footer>
                        <button name="action_create_alternative" string="Create Alternative" data-hotkey="q" type="object" colspan="1" class="btn-primary"/>
                        <button string="Cancel"  data-hotkey="x" special="cancel" colspan="1" class="btn-secondary"/>
                    </footer>
                </form>
            </field>
        </record>
    </data>
</odoo>

```

## File: wizard\__init__.py

```python
from . import purchase_requisition_alternative_warning
from . import purchase_requisition_create_alternative

```

