# Odoo Module: sale_purchase

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Sale Purchase',
    'summary': 'Sale based on service outsourcing.',
    'description': """
Allows the outsourcing of services. This module allows one to sell services provided
by external providers and will automatically generate purchase orders directed to the service seller.
    """,
    'version': '1.0',
    'website': 'https://www.odoo.com/',
    'category': 'Hidden',
    'depends': [
        'sale',
        'purchase',
    ],
    'data': [
        'data/mail_data.xml',
        'views/product_views.xml',
        'views/sale_order_views.xml',
        'views/purchase_order_views.xml',
    ],
    'demo': [
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\mail_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="exception_purchase_on_sale_quantity_decreased" name="Message: Alert on purchase orders when ordered quantity decreased on Sales Orders">
    <div>
        <p>
            Exception(s) occurred on the sale order(s):
            <t t-foreach="sale_orders" t-as="sale_order">
                <a href="#" data-oe-model="sale.order" t-att-data-oe-id="sale_order.id"><t t-esc="sale_order.name"/></a>.
            </t>
            Manual actions may be needed.
        </p>
        <div class="mt16">
            <p>Exception(s):</p>
            <ul t-foreach="sale_lines" t-as="sale_line">
                <li>
                    <t t-set="new_qty" t-value="sale_line.product_uom_qty"/>
                    <t t-set="old_qty" t-value="origin_values.get(sale_line.id, 0.0)"/>

                    <a href="#" data-oe-model="sale.order" t-att-data-oe-id="sale_line.order_id.id"><t t-esc="sale_line.order_id.name"/></a>:
                    <t t-esc="new_qty"/> <t t-esc="sale_line.product_uom.name"/> of <t t-esc="sale_line.product_id.name"/> ordered instead of <t t-esc="old_qty"/> <t t-esc="sale_line.product_uom.name"/>
                  </li>
            </ul>
        </div>
    </div>
</template>

<template id="exception_purchase_on_sale_cancellation" name="Message: Alert on purchase orders when sales orders are cancelled">
    <div>
        <p>
            Exception(s) occurred on the sale order(s):
            <t t-foreach="sale_orders" t-as="sale_order">
                <a href="#" data-oe-model="sale.order" t-att-data-oe-id="sale_order.id"><t t-esc="sale_order.name"/></a>
            </t>.
            Manual actions may be needed.
        </p>
        <div class="mt16">
            <p>Exception(s):</p>
            <ul t-foreach="sale_order_lines" t-as="sale_line">
                <li>
                    <a href="#" data-oe-model="sale.order" t-att-data-oe-id="sale_line.order_id.id"><t t-esc="sale_line.order_id.name"/></a>: <t t-esc="sale_line.product_uom_qty"/> <t t-esc="sale_line.product_uom.name"/> of <t t-esc="sale_line.product_id.name"/> cancelled
                </li>
            </ul>
        </div>
    </div>
</template>

<template id="exception_sale_on_purchase_cancellation" name="Message: Alert on sale orders when purchase orders are cancelled">
    <div>
        <p>
            Exception(s) occurred on the purchase order(s):
            <t t-foreach="purchase_orders" t-as="purchase_order">
                <a href="#" data-oe-model="purchase.order" t-att-data-oe-id="purchase_order.id"><t t-esc="purchase_order.name"/></a>
            </t>.
            Manual actions may be needed.
        </p>
        <div class="mt16">
            <p>Exception(s):</p>
            <ul t-foreach="purchase_order_lines" t-as="purchase_line">
                <li>
                    <a href="#" data-oe-model="purchase.order" t-att-data-oe-id="purchase_line.order_id.id"><t t-esc="purchase_line.order_id.name"/></a>: <t t-esc="purchase_line.product_qty"/> <t t-esc="purchase_line.product_uom.name"/> of <t t-esc="purchase_line.product_id.name"/> cancelled
                </li>
            </ul>
        </div>
    </div>
</template>

</odoo>

```

## File: models\product_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ProductTemplate(models.Model):
    _inherit = 'product.template'

    service_to_purchase = fields.Boolean("Subcontract Service", help="If ticked, each time you sell this product through a SO, a RfQ is automatically created to buy the product. Tip: don't forget to set a vendor on the product.")

    _sql_constraints = [
        ('service_to_purchase', "CHECK((type != 'service' AND service_to_purchase != true) or (type = 'service'))", 'Product that is not a service can not create RFQ.'),
    ]

    @api.onchange('type')
    def _onchange_type(self):
        res = super(ProductTemplate, self)._onchange_type()
        if self.type != 'service':
            self.service_to_purchase = False
        return res

    @api.onchange('expense_policy')
    def _onchange_expense_policy(self):
        if self.expense_policy != 'no':
            self.service_to_purchase = False

```

## File: models\purchase_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _


class PurchaseOrder(models.Model):
    _inherit = "purchase.order"

    sale_order_count = fields.Integer(
        "Number of Source Sale",
        compute='_compute_sale_order_count',
        groups='sales_team.group_sale_salesman')

    @api.depends('order_line.sale_order_id')
    def _compute_sale_order_count(self):
        for purchase in self:
            purchase.sale_order_count = len(purchase._get_sale_orders())

    def action_view_sale_orders(self):
        self.ensure_one()
        sale_order_ids = self._get_sale_orders().ids
        action = {
            'res_model': 'sale.order',
            'type': 'ir.actions.act_window',
        }
        if len(sale_order_ids) == 1:
            action.update({
                'view_mode': 'form',
                'res_id': sale_order_ids[0],
            })
        else:
            action.update({
                'name': _('Sources Sale Orders %s', self.name),
                'domain': [('id', 'in', sale_order_ids)],
                'view_mode': 'tree,form',
            })
        return action

    def button_cancel(self):
        result = super(PurchaseOrder, self).button_cancel()
        self.sudo()._activity_cancel_on_sale()
        return result

    def _get_sale_orders(self):
        return self.order_line.sale_order_id

    def _activity_cancel_on_sale(self):
        """ If some PO are cancelled, we need to put an activity on their origin SO (only the open ones). Since a PO can have
            been modified by several SO, when cancelling one PO, many next activities can be schedulded on different SO.
        """
        sale_to_notify_map = {}  # map SO -> recordset of PO as {sale.order: set(purchase.order.line)}
        for order in self:
            for purchase_line in order.order_line:
                if purchase_line.sale_line_id:
                    sale_order = purchase_line.sale_line_id.order_id
                    sale_to_notify_map.setdefault(sale_order, self.env['purchase.order.line'])
                    sale_to_notify_map[sale_order] |= purchase_line

        for sale_order, purchase_order_lines in sale_to_notify_map.items():
            sale_order._activity_schedule_with_view('mail.mail_activity_data_warning',
                user_id=sale_order.user_id.id or self.env.uid,
                views_or_xmlid='sale_purchase.exception_sale_on_purchase_cancellation',
                render_context={
                    'purchase_orders': purchase_order_lines.mapped('order_id'),
                    'purchase_order_lines': purchase_order_lines,
            })


class PurchaseOrderLine(models.Model):
    _inherit = 'purchase.order.line'

    sale_order_id = fields.Many2one(related='sale_line_id.order_id', string="Sale Order", store=True, readonly=True)
    sale_line_id = fields.Many2one('sale.order.line', string="Origin Sale Item", index=True, copy=False)

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from dateutil.relativedelta import relativedelta

from odoo import api, fields, models, _
from odoo.exceptions import UserError
from odoo.tools import float_compare
from odoo.tools.misc import get_lang


class SaleOrder(models.Model):
    _inherit = 'sale.order'

    purchase_order_count = fields.Integer(
        "Number of Purchase Order Generated",
        compute='_compute_purchase_order_count',
        groups='purchase.group_purchase_user')

    @api.depends('order_line.purchase_line_ids.order_id')
    def _compute_purchase_order_count(self):
        for order in self:
            order.purchase_order_count = len(order._get_purchase_orders())

    def _action_confirm(self):
        result = super(SaleOrder, self)._action_confirm()
        for order in self:
            order.order_line.sudo()._purchase_service_generation()
        return result

    def _action_cancel(self):
        result = super()._action_cancel()
        # When a sale person cancel a SO, he might not have the rights to write
        # on PO. But we need the system to create an activity on the PO (so 'write'
        # access), hence the `sudo`.
        self.sudo()._activity_cancel_on_purchase()
        return result

    def action_view_purchase_orders(self):
        self.ensure_one()
        purchase_order_ids = self._get_purchase_orders().ids
        action = {
            'res_model': 'purchase.order',
            'type': 'ir.actions.act_window',
        }
        if len(purchase_order_ids) == 1:
            action.update({
                'view_mode': 'form',
                'res_id': purchase_order_ids[0],
            })
        else:
            action.update({
                'name': _("Purchase Order generated from %s", self.name),
                'domain': [('id', 'in', purchase_order_ids)],
                'view_mode': 'tree,form',
            })
        return action

    def _get_purchase_orders(self):
        return self.order_line.purchase_line_ids.order_id

    def _activity_cancel_on_purchase(self):
        """ If some SO are cancelled, we need to put an activity on their generated purchase. If sale lines of
            different sale orders impact different purchase, we only want one activity to be attached.
        """
        purchase_to_notify_map = {}  # map PO -> recordset of SOL as {purchase.order: set(sale.orde.liner)}

        purchase_order_lines = self.env['purchase.order.line'].search([('sale_line_id', 'in', self.mapped('order_line').ids), ('state', '!=', 'cancel')])
        for purchase_line in purchase_order_lines:
            purchase_to_notify_map.setdefault(purchase_line.order_id, self.env['sale.order.line'])
            purchase_to_notify_map[purchase_line.order_id] |= purchase_line.sale_line_id

        for purchase_order, sale_order_lines in purchase_to_notify_map.items():
            purchase_order._activity_schedule_with_view('mail.mail_activity_data_warning',
                user_id=purchase_order.user_id.id or self.env.uid,
                views_or_xmlid='sale_purchase.exception_purchase_on_sale_cancellation',
                render_context={
                    'sale_orders': sale_order_lines.mapped('order_id'),
                    'sale_order_lines': sale_order_lines,
            })


class SaleOrderLine(models.Model):
    _inherit = 'sale.order.line'

    purchase_line_ids = fields.One2many('purchase.order.line', 'sale_line_id', string="Generated Purchase Lines", readonly=True, help="Purchase line generated by this Sales item on order confirmation, or when the quantity was increased.")
    purchase_line_count = fields.Integer("Number of generated purchase items", compute='_compute_purchase_count')

    @api.depends('purchase_line_ids')
    def _compute_purchase_count(self):
        database_data = self.env['purchase.order.line'].sudo().read_group([('sale_line_id', 'in', self.ids)], ['sale_line_id'], ['sale_line_id'])
        mapped_data = dict([(db['sale_line_id'][0], db['sale_line_id_count']) for db in database_data])
        for line in self:
            line.purchase_line_count = mapped_data.get(line.id, 0)

    @api.onchange('product_uom_qty')
    def _onchange_service_product_uom_qty(self):
        if self.state == 'sale' and self.product_id.type == 'service' and self.product_id.service_to_purchase:
            if self.product_uom_qty < self._origin.product_uom_qty:
                if self.product_uom_qty < self.qty_delivered:
                    return {}
                warning_mess = {
                    'title': _('Ordered quantity decreased!'),
                    'message': _('You are decreasing the ordered quantity! Do not forget to manually update the purchase order if needed.'),
                }
                return {'warning': warning_mess}
        return {}

    # --------------------------
    # CRUD
    # --------------------------

    @api.model_create_multi
    def create(self, values):
        lines = super(SaleOrderLine, self).create(values)
        # Do not generate purchase when expense SO line since the product is already delivered
        lines.filtered(
            lambda line: line.state == 'sale' and not line.is_expense
        )._purchase_service_generation()
        return lines

    def write(self, values):
        increased_lines = None
        decreased_lines = None
        increased_values = {}
        decreased_values = {}
        if 'product_uom_qty' in values:
            precision = self.env['decimal.precision'].precision_get('Product Unit of Measure')
            increased_lines = self.sudo().filtered(lambda r: r.product_id.service_to_purchase and r.purchase_line_count and float_compare(r.product_uom_qty, values['product_uom_qty'], precision_digits=precision) == -1)
            decreased_lines = self.sudo().filtered(lambda r: r.product_id.service_to_purchase and r.purchase_line_count and float_compare(r.product_uom_qty, values['product_uom_qty'], precision_digits=precision) == 1)
            increased_values = {line.id: line.product_uom_qty for line in increased_lines}
            decreased_values = {line.id: line.product_uom_qty for line in decreased_lines}

        result = super(SaleOrderLine, self).write(values)

        if increased_lines:
            increased_lines._purchase_increase_ordered_qty(values['product_uom_qty'], increased_values)
        if decreased_lines:
            decreased_lines._purchase_decrease_ordered_qty(values['product_uom_qty'], decreased_values)
        return result

    # --------------------------
    # Business Methods
    # --------------------------

    def _purchase_decrease_ordered_qty(self, new_qty, origin_values):
        """ Decrease the quantity from SO line will add a next acitivities on the related purchase order
            :param new_qty: new quantity (lower than the current one on SO line), expressed
                in UoM of SO line.
            :param origin_values: map from sale line id to old value for the ordered quantity (dict)
        """
        purchase_to_notify_map = {}  # map PO -> set(SOL)
        last_purchase_lines = self.env['purchase.order.line'].search([('sale_line_id', 'in', self.ids)])
        for purchase_line in last_purchase_lines:
            purchase_to_notify_map.setdefault(purchase_line.order_id, self.env['sale.order.line'])
            purchase_to_notify_map[purchase_line.order_id] |= purchase_line.sale_line_id

        # create next activity
        for purchase_order, sale_lines in purchase_to_notify_map.items():
            render_context = {
                'sale_lines': sale_lines,
                'sale_orders': sale_lines.mapped('order_id'),
                'origin_values': origin_values,
            }
            purchase_order._activity_schedule_with_view('mail.mail_activity_data_warning',
                user_id=purchase_order.user_id.id or self.env.uid,
                views_or_xmlid='sale_purchase.exception_purchase_on_sale_quantity_decreased',
                render_context=render_context)

    def _purchase_increase_ordered_qty(self, new_qty, origin_values):
        """ Increase the quantity on the related purchase lines
            :param new_qty: new quantity (higher than the current one on SO line), expressed
                in UoM of SO line.
            :param origin_values: map from sale line id to old value for the ordered quantity (dict)
        """
        for line in self:
            last_purchase_line = self.env['purchase.order.line'].search([('sale_line_id', '=', line.id)], order='create_date DESC', limit=1)
            if last_purchase_line.state in ['draft', 'sent', 'to approve']:  # update qty for draft PO lines
                quantity = line.product_uom._compute_quantity(new_qty, last_purchase_line.product_uom)
                last_purchase_line.write({'product_qty': quantity})
            elif last_purchase_line.state in ['purchase', 'done', 'cancel']:  # create new PO, by forcing the quantity as the difference from SO line
                quantity = line.product_uom._compute_quantity(new_qty - origin_values.get(line.id, 0.0), last_purchase_line.product_uom)
                line._purchase_service_create(quantity=quantity)

    def _purchase_get_date_order(self, supplierinfo):
        """ return the ordered date for the purchase order, computed as : SO commitment date - supplier delay """
        commitment_date = fields.Datetime.from_string(self.order_id.commitment_date or fields.Datetime.now())
        return commitment_date - relativedelta(days=int(supplierinfo.delay))

    def _purchase_service_prepare_order_values(self, supplierinfo):
        """ Returns the values to create the purchase order from the current SO line.
            :param supplierinfo: record of product.supplierinfo
            :rtype: dict
        """
        self.ensure_one()
        partner_supplier = supplierinfo.name
        fpos = self.env['account.fiscal.position'].sudo().get_fiscal_position(partner_supplier.id)
        date_order = self._purchase_get_date_order(supplierinfo)
        return {
            'partner_id': partner_supplier.id,
            'partner_ref': partner_supplier.ref,
            'company_id': self.company_id.id,
            'currency_id': partner_supplier.property_purchase_currency_id.id or self.env.company.currency_id.id,
            'dest_address_id': False, # False since only supported in stock
            'origin': self.order_id.name,
            'payment_term_id': partner_supplier.property_supplier_payment_term_id.id,
            'date_order': date_order,
            'fiscal_position_id': fpos.id,
        }

    def _purchase_service_prepare_line_values(self, purchase_order, quantity=False):
        """ Returns the values to create the purchase order line from the current SO line.
            :param purchase_order: record of purchase.order
            :rtype: dict
            :param quantity: the quantity to force on the PO line, expressed in SO line UoM
        """
        self.ensure_one()
        # compute quantity from SO line UoM
        product_quantity = self.product_uom_qty
        if quantity:
            product_quantity = quantity

        purchase_qty_uom = self.product_uom._compute_quantity(product_quantity, self.product_id.uom_po_id)

        # determine vendor (real supplier, sharing the same partner as the one from the PO, but with more accurate informations like validity, quantity, ...)
        # Note: one partner can have multiple supplier info for the same product
        supplierinfo = self.product_id._select_seller(
            partner_id=purchase_order.partner_id,
            quantity=purchase_qty_uom,
            date=purchase_order.date_order and purchase_order.date_order.date(), # and purchase_order.date_order[:10],
            uom_id=self.product_id.uom_po_id
        )
        supplier_taxes = self.product_id.supplier_taxes_id.filtered(lambda t: t.company_id.id == self.company_id.id)
        taxes = purchase_order.fiscal_position_id.map_tax(supplier_taxes)

        # compute unit price
        price_unit = 0.0
        product_ctx = {
            'lang': get_lang(self.env, purchase_order.partner_id.lang).code,
            'company_id': purchase_order.company_id,
        }
        if supplierinfo:
            price_unit = self.env['account.tax'].sudo()._fix_tax_included_price_company(
                supplierinfo.price, supplier_taxes, taxes, self.company_id)
            if purchase_order.currency_id and supplierinfo.currency_id != purchase_order.currency_id:
                price_unit = supplierinfo.currency_id._convert(price_unit, purchase_order.currency_id, purchase_order.company_id, fields.Date.context_today(self))
            product_ctx.update({'seller_id': supplierinfo.id})
        else:
            product_ctx.update({'partner_id': purchase_order.partner_id.id})

        product = self.product_id.with_context(**product_ctx)
        name = product.display_name
        if product.description_purchase:
            name += '\n' + product.description_purchase

        line_description = self.with_context(lang=self.order_id.partner_id.lang)._get_sale_order_line_multiline_description_variants()
        if line_description:
            name += line_description

        return {
            'name': name,
            'product_qty': purchase_qty_uom,
            'product_id': self.product_id.id,
            'product_uom': self.product_id.uom_po_id.id,
            'price_unit': price_unit,
            'date_planned': fields.Date.from_string(purchase_order.date_order) + relativedelta(days=int(supplierinfo.delay)),
            'taxes_id': [(6, 0, taxes.ids)],
            'order_id': purchase_order.id,
            'sale_line_id': self.id,
        }

    def _retrieve_purchase_partner(self):
        """ In case we want to explicitely name a partner from whom we want to buy or receive products
        """
        self.ensure_one()
        return False

    def _purchase_service_create(self, quantity=False):
        """ On Sales Order confirmation, some lines (services ones) can create a purchase order line and maybe a purchase order.
            If a line should create a RFQ, it will check for existing PO. If no one is find, the SO line will create one, then adds
            a new PO line. The created purchase order line will be linked to the SO line.
            :param quantity: the quantity to force on the PO line, expressed in SO line UoM
        """
        PurchaseOrder = self.env['purchase.order']
        supplier_po_map = {}
        sale_line_purchase_map = {}
        for line in self:
            line = line.with_company(line.company_id)
            # determine vendor of the order (take the first matching company and product)
            suppliers = line.product_id._select_seller(partner_id=line._retrieve_purchase_partner(), quantity=line.product_uom_qty, uom_id=line.product_uom)
            if not suppliers:
                raise UserError(_("There is no vendor associated to the product %s. Please define a vendor for this product.") % (line.product_id.display_name,))
            supplierinfo = suppliers[0]
            partner_supplier = supplierinfo.name  # yes, this field is not explicit .... it is a res.partner !

            # determine (or create) PO
            purchase_order = supplier_po_map.get(partner_supplier.id)
            if not purchase_order:
                purchase_order = PurchaseOrder.search([
                    ('partner_id', '=', partner_supplier.id),
                    ('state', '=', 'draft'),
                    ('company_id', '=', line.company_id.id),
                ], limit=1)
            if not purchase_order:
                values = line._purchase_service_prepare_order_values(supplierinfo)
                purchase_order = PurchaseOrder.with_context(mail_create_nosubscribe=True).create(values)
            else:  # update origin of existing PO
                so_name = line.order_id.name
                origins = []
                if purchase_order.origin:
                    origins = purchase_order.origin.split(', ') + origins
                if so_name not in origins:
                    origins += [so_name]
                    purchase_order.write({
                        'origin': ', '.join(origins)
                    })
            supplier_po_map[partner_supplier.id] = purchase_order

            # add a PO line to the PO
            values = line._purchase_service_prepare_line_values(purchase_order, quantity=quantity)
            purchase_line = line.env['purchase.order.line'].create(values)

            # link the generated purchase to the SO line
            sale_line_purchase_map.setdefault(line, line.env['purchase.order.line'])
            sale_line_purchase_map[line] |= purchase_line
        return sale_line_purchase_map

    def _purchase_service_generation(self):
        """ Create a Purchase for the first time from the sale line. If the SO line already created a PO, it
            will not create a second one.
        """
        sale_line_purchase_map = {}
        for line in self:
            # Do not regenerate PO line if the SO line has already created one in the past (SO cancel/reconfirmation case)
            if line.product_id.service_to_purchase and not line.purchase_line_count:
                result = line._purchase_service_create()
                sale_line_purchase_map.update(result)
        return sale_line_purchase_map

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import product_template
from . import sale_order
from . import purchase_order

```

## File: views\product_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="product_template_form_view_inherit" model="ir.ui.view">
        <field name="name">product.template.form.inherit</field>
        <field name="model">product.template</field>
        <field name="inherit_id" ref="purchase.view_product_supplier_inherit"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='bill']" position="before">
                <group string="Reordering" attrs="{'invisible': [('type','!=','service')]}">
                    <field name="service_to_purchase"/>
                </group>
            </xpath>
        </field>
    </record>

</odoo>
```

## File: views\purchase_order_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>
    <record id="purchase_order_inherited_form_sale" model="ir.ui.view">
        <field name="name">purchase.order.inherited.form.sale</field>
        <field name="model">purchase.order</field>
        <field name="inherit_id" ref="purchase.purchase_order_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='button_box']" position="inside">
                <button class="oe_stat_button" name="action_view_sale_orders" type="object" icon="fa-dollar" groups='sales_team.group_sale_salesman' attrs="{'invisible': [('sale_order_count', '=', 0)]}">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_value"><field name="sale_order_count"/></span>
                        <span class="o_stat_text">Sale</span>
                    </div>
                </button>
            </xpath>
        </field>
    </record>
</data></odoo>

```

## File: views\sale_order_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>
    <record id="sale_order_inherited_form_purchase" model="ir.ui.view">
        <field name="name">sale.order.inherited.form.purchase</field>
        <field name="model">sale.order</field>
        <field name="inherit_id" ref="sale.view_order_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='button_box']" position="inside">
                <button class="oe_stat_button" name="action_view_purchase_orders" type="object" icon="fa-credit-card" groups='purchase.group_purchase_user' attrs="{'invisible': [('purchase_order_count', '=', 0)]}">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_value"><field name="purchase_order_count"/></span>
                        <span class="o_stat_text">Purchase</span>
                    </div>
                </button>
            </xpath>
        </field>
    </record>
</data></odoo>

```

