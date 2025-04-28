# Odoo Module: sale_management

Category: Sales/Sales

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import controllers
from odoo.api import Environment, SUPERUSER_ID


def uninstall_hook(cr, registry):
    env = Environment(cr, SUPERUSER_ID, {})
    res_ids = env['ir.model.data'].search([
        ('model', '=', 'ir.ui.menu'),
        ('module', '=', 'sale')
    ]).mapped('res_id')
    env['ir.ui.menu'].browse(res_ids).update({'active': False})


def post_init_hook(cr, registry):
    env = Environment(cr, SUPERUSER_ID, {})
    res_ids = env['ir.model.data'].search([
        ('model', '=', 'ir.ui.menu'),
        ('module', '=', 'sale'),
    ]).mapped('res_id')
    env['ir.ui.menu'].browse(res_ids).update({'active': True})

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Sales',
    'version': '1.0',
    'category': 'Sales/Sales',
    'sequence': 17,
    'summary': 'From quotations to invoices',
    'description': """
Manage sales quotations and orders
==================================

This application allows you to manage your sales goals in an effective and efficient manner by keeping track of all sales orders and history.

It handles the full sales workflow:

* **Quotation** -> **Sales order** -> **Invoice**

Preferences (only with Warehouse Management installed)
------------------------------------------------------

If you also installed the Warehouse Management, you can deal with the following preferences:

* Shipping: Choice of delivery at once or partial delivery
* Invoicing: choose how invoices will be paid
* Incoterms: International Commercial terms


With this module you can personnalize the sales order and invoice report with
categories, subtotals or page-breaks.

The Dashboard for the Sales Manager will include
------------------------------------------------
* My Quotations
* Monthly Turnover (Graph)
    """,
    'website': 'https://www.odoo.com/page/sales',
    'depends': ['sale', 'digest'],
    'data': [
        'security/sale_management_security.xml',
        'views/sale_portal_templates.xml',
        'views/sale_order_template_views.xml',
        'security/ir.model.access.csv',
        'views/res_config_settings_views.xml',
        'data/digest_data.xml',
        'views/sale_management_views.xml',
        'views/digest_views.xml',
        'views/sale_order_views.xml',
        'report/sale_report_templates.xml',
    ],
    'demo': [
        'data/sale_order_template_demo.xml',
    ],
    'application': True,
    'uninstall_hook': 'uninstall_hook',
    'post_init_hook': 'post_init_hook',
    'license': 'LGPL-3',
}

```

## File: controllers\portal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from functools import partial

from odoo import http
from odoo.tools import formatLang
from odoo.exceptions import AccessError, MissingError
from odoo.http import request
from odoo.addons.sale.controllers.portal import CustomerPortal


class CustomerPortal(CustomerPortal):

    @http.route(['/my/orders/<int:order_id>/update_line'], type='json', auth="public", website=True)
    def update(self, line_id, remove=False, unlink=False, order_id=None, access_token=None, **post):
        values = self.update_line_dict(line_id, remove, unlink, order_id, access_token, **post)
        if values:
            return [values['order_line_product_uom_qty'], values['order_amount_total']]
        return values

    @http.route(['/my/orders/<int:order_id>/update_line_dict'], type='json', auth="public", website=True)
    def update_line_dict(self, line_id, remove=False, unlink=False, order_id=None, access_token=None, input_quantity=False, **kwargs):
        try:
            order_sudo = self._document_check_access('sale.order', order_id, access_token=access_token)
        except (AccessError, MissingError):
            return request.redirect('/my')

        if order_sudo.state not in ('draft', 'sent'):
            return False
        order_line = request.env['sale.order.line'].sudo().browse(int(line_id))
        if order_line.order_id != order_sudo:
            return False
        if unlink:
            order_line.unlink()
            return False  # return False to reload the page, the line must move back to options and the JS doesn't handle it

        if input_quantity is not False:
            quantity = input_quantity
        else:
            number = -1 if remove else 1
            quantity = order_line.product_uom_qty + number

        if quantity < 0:
            quantity = 0.0
        order_line.write({'product_uom_qty': quantity})
        currency = order_sudo.currency_id
        format_price = partial(formatLang, request.env, digits=currency.decimal_places)

        results = {
            'order_line_product_uom_qty': str(quantity),
            'order_line_price_total': format_price(order_line.price_total),
            'order_line_price_subtotal': format_price(order_line.price_subtotal),
            'order_amount_total': format_price(order_sudo.amount_total),
            'order_amount_untaxed': format_price(order_sudo.amount_untaxed),
            'order_amount_tax': format_price(order_sudo.amount_tax),
            'order_amount_undiscounted': format_price(order_sudo.amount_undiscounted),
        }
        try:
            results['order_totals_table'] = request.env['ir.ui.view'].render_template('sale.sale_order_portal_content_totals_table', {'sale_order': order_sudo})
        except ValueError:
            pass

        return results

    @http.route(["/my/orders/<int:order_id>/add_option/<int:option_id>"], type='http', auth="public", website=True)
    def add(self, order_id, option_id, access_token=None, **post):
        try:
            order_sudo = self._document_check_access('sale.order', order_id, access_token=access_token)
        except (AccessError, MissingError):
            return request.redirect('/my')

        option_sudo = request.env['sale.order.option'].sudo().browse(option_id)

        if order_sudo != option_sudo.order_id:
            return request.redirect(order_sudo.get_portal_url())

        option_sudo.add_option_to_order()

        return request.redirect(option_sudo.order_id.get_portal_url(anchor='details'))

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import portal

```

## File: data\digest_data.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<odoo>
    <data noupdate="1">
        <record id="digest.digest_digest_default" model="digest.digest">
            <field name="kpi_all_sale_total">True</field>
        </record>
    </data>
</odoo>

```

## File: data\sale_order_template_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <!-- We want to activate SO template by default for easier demoing. -->
        <record id="base.group_user" model="res.groups">
            <field name="implied_ids" eval="[(4, ref('sale_management.group_sale_order_template'))]"/>
        </record>

        <record id="sale_order_template_1" model="sale.order.template">
            <field name="name">4 Person Desk</field>
            <field name="number_of_days">45</field>
        </record>

        <record id="sale_order_template_line_1" model="sale.order.template.line">
            <field name="sale_order_template_id" ref="sale_order_template_1"/>
            <field name="name">4 Person Desk</field>
            <field name="product_id" ref="product.consu_delivery_03"/>
            <field name="product_uom_qty">1</field>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="price_unit">150.0</field>
            <field name="discount">10.00</field>
        </record>

        <record id="sale_order_template_option_1" model="sale.order.template.option">
            <field name="sale_order_template_id" ref="sale_order_template_1"/>
            <field name="name">Office Chair</field>
            <field name="product_id" ref="product.product_delivery_01"/>
            <field name="quantity">4</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="price_unit">45.0</field>
            <field name="discount">10.0</field>
        </record>
</odoo>

```

## File: models\digest.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _
from odoo.exceptions import AccessError


class Digest(models.Model):
    _inherit = 'digest.digest'

    kpi_all_sale_total = fields.Boolean('All Sales')
    kpi_all_sale_total_value = fields.Monetary(compute='_compute_kpi_sale_total_value')

    def _compute_kpi_sale_total_value(self):
        if not self.env.user.has_group('sales_team.group_sale_salesman_all_leads'):
            raise AccessError(_("Do not have access, skip this data for user's digest email"))
        for record in self:
            start, end, company = record._get_kpi_compute_parameters()
            all_channels_sales = self.env['sale.report'].read_group([
                ('date', '>=', start),
                ('date', '<', end),
                ('state', 'not in', ['draft', 'cancel', 'sent']),
                ('company_id', '=', company.id)], ['price_total'], ['price_total'])
            record.kpi_all_sale_total_value = sum([channel_sale['price_total'] for channel_sale in all_channels_sales])

    def compute_kpis_actions(self, company, user):
        res = super(Digest, self).compute_kpis_actions(company, user)
        res['kpi_all_sale_total'] = 'sale.report_all_channels_sales_action&menu_id=%s' % self.env.ref('sale.sale_menu_root').id
        return res

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    group_sale_order_template = fields.Boolean("Quotation Templates", implied_group='sale_management.group_sale_order_template')
    default_sale_order_template_id = fields.Many2one('sale.order.template', default_model='sale.order', string='Default Template')
    module_sale_quotation_builder = fields.Boolean("Quotation Builder")

    @api.onchange('group_sale_order_template')
    def _onchange_group_sale_order_template(self):
        if not self.group_sale_order_template:
            self.module_sale_quotation_builder = False
            self.default_sale_order_template_id = False

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import timedelta

from odoo import api, fields, models, _
from odoo.exceptions import UserError, ValidationError


class SaleOrder(models.Model):
    _inherit = 'sale.order'

    sale_order_template_id = fields.Many2one(
        'sale.order.template', 'Quotation Template',
        readonly=True, check_company=True,
        states={'draft': [('readonly', False)], 'sent': [('readonly', False)]},
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    sale_order_option_ids = fields.One2many(
        'sale.order.option', 'order_id', 'Optional Products Lines',
        copy=True, readonly=True,
        states={'draft': [('readonly', False)], 'sent': [('readonly', False)]})

    @api.constrains('company_id', 'sale_order_option_ids')
    def _check_optional_product_company_id(self):
        for order in self:
            companies = order.sale_order_option_ids.product_id.company_id
            if companies and companies != order.company_id:
                bad_products = order.sale_order_option_ids.product_id.filtered(lambda p: p.company_id and p.company_id != order.company_id)
                raise ValidationError((_("Your quotation contains products from company %s whereas your quotation belongs to company %s. \n Please change the company of your quotation or remove the products from other companies (%s).") % (', '.join(companies.mapped('display_name')), order.company_id.display_name, ', '.join(bad_products.mapped('display_name')))))

    @api.returns('self', lambda value: value.id)
    def copy(self, default=None):
        if self.sale_order_template_id and self.sale_order_template_id.number_of_days > 0:
            default = dict(default or {})
            default['validity_date'] = fields.Date.context_today(self) + timedelta(self.sale_order_template_id.number_of_days)
        return super(SaleOrder, self).copy(default=default)

    @api.onchange('partner_id')
    def onchange_partner_id(self):
        super(SaleOrder, self).onchange_partner_id()
        template = self.sale_order_template_id.with_context(lang=self.partner_id.lang)
        self.note = template.note or self.note

    def _compute_line_data_for_template_change(self, line):
        return {
            'display_type': line.display_type,
            'name': line.name,
            'state': 'draft',
        }

    def _compute_option_data_for_template_change(self, option):
        if self.pricelist_id:
            price = self.pricelist_id.with_context(uom=option.uom_id.id).get_product_price(option.product_id, 1, False)
        else:
            price = option.price_unit
        return {
            'product_id': option.product_id.id,
            'name': option.name,
            'quantity': option.quantity,
            'uom_id': option.uom_id.id,
            'price_unit': price,
            'discount': option.discount,
        }

    @api.onchange('sale_order_template_id')
    def onchange_sale_order_template_id(self):
        if not self.sale_order_template_id:
            self.require_signature = self._get_default_require_signature()
            self.require_payment = self._get_default_require_payment()
            return
        template = self.sale_order_template_id.with_context(lang=self.partner_id.lang)

        order_lines = [(5, 0, 0)]
        for line in template.sale_order_template_line_ids:
            data = self._compute_line_data_for_template_change(line)
            if line.product_id:
                discount = 0
                if self.pricelist_id:
                    price = self.pricelist_id.with_context(uom=line.product_uom_id.id).get_product_price(line.product_id, 1, False)
                    if self.pricelist_id.discount_policy == 'without_discount' and line.price_unit:
                        discount = (line.price_unit - price) / line.price_unit * 100
                        # negative discounts (= surcharge) are included in the display price
                        if discount < 0:
                            discount = 0
                        else:
                            price = line.price_unit
                    elif line.price_unit:
                        price = line.price_unit

                else:
                    price = line.price_unit

                data.update({
                    'price_unit': price,
                    'discount': 100 - ((100 - discount) * (100 - line.discount) / 100),
                    'product_uom_qty': line.product_uom_qty,
                    'product_id': line.product_id.id,
                    'product_uom': line.product_uom_id.id,
                    'customer_lead': self._get_customer_lead(line.product_id.product_tmpl_id),
                })
                if self.pricelist_id:
                    data.update(self.env['sale.order.line']._get_purchase_price(self.pricelist_id, line.product_id, line.product_uom_id, fields.Date.context_today(self)))
            order_lines.append((0, 0, data))

        self.order_line = order_lines
        self.order_line._compute_tax_id()

        option_lines = [(5, 0, 0)]
        for option in template.sale_order_template_option_ids:
            data = self._compute_option_data_for_template_change(option)
            option_lines.append((0, 0, data))
        self.sale_order_option_ids = option_lines

        if template.number_of_days > 0:
            self.validity_date = fields.Date.context_today(self) + timedelta(template.number_of_days)

        self.require_signature = template.require_signature
        self.require_payment = template.require_payment

        if template.note:
            self.note = template.note

    def action_confirm(self):
        res = super(SaleOrder, self).action_confirm()
        for order in self:
            if order.sale_order_template_id and order.sale_order_template_id.mail_template_id:
                self.sale_order_template_id.mail_template_id.send_mail(order.id)
        return res

    def get_access_action(self, access_uid=None):
        """ Instead of the classic form view, redirect to the online quote if it exists. """
        self.ensure_one()
        user = access_uid and self.env['res.users'].sudo().browse(access_uid) or self.env.user

        if not self.sale_order_template_id or (not user.share and not self.env.context.get('force_website')):
            return super(SaleOrder, self).get_access_action(access_uid)
        return {
            'type': 'ir.actions.act_url',
            'url': self.get_portal_url(),
            'target': 'self',
            'res_id': self.id,
        }


class SaleOrderLine(models.Model):
    _inherit = "sale.order.line"
    _description = "Sales Order Line"

    sale_order_option_ids = fields.One2many('sale.order.option', 'line_id', 'Optional Products Lines')

    # Take the description on the order template if the product is present in it
    @api.onchange('product_id')
    def product_id_change(self):
        domain = super(SaleOrderLine, self).product_id_change()
        if self.product_id and self.order_id.sale_order_template_id:
            for line in self.order_id.sale_order_template_id.sale_order_template_line_ids:
                if line.product_id == self.product_id:
                    self.name = line.name + self._get_sale_order_line_multiline_description_variants()
                    break
        return domain


class SaleOrderOption(models.Model):
    _name = "sale.order.option"
    _description = "Sale Options"
    _order = 'sequence, id'

    is_present = fields.Boolean(string="Present on Quotation",
                           help="This field will be checked if the option line's product is "
                                "already present in the quotation.",
                           compute="_compute_is_present", search="_search_is_present")
    order_id = fields.Many2one('sale.order', 'Sales Order Reference', ondelete='cascade', index=True)
    line_id = fields.Many2one('sale.order.line', ondelete="set null", copy=False)
    name = fields.Text('Description', required=True)
    product_id = fields.Many2one('product.product', 'Product', required=True, domain=[('sale_ok', '=', True)])
    price_unit = fields.Float('Unit Price', required=True, digits='Product Price')
    discount = fields.Float('Discount (%)', digits='Discount')
    uom_id = fields.Many2one('uom.uom', 'Unit of Measure ', required=True, domain="[('category_id', '=', product_uom_category_id)]")
    product_uom_category_id = fields.Many2one(related='product_id.uom_id.category_id', readonly=True)
    quantity = fields.Float('Quantity', required=True, digits='Product UoS', default=1)
    sequence = fields.Integer('Sequence', help="Gives the sequence order when displaying a list of optional products.")

    @api.depends('line_id', 'order_id.order_line', 'product_id')
    def _compute_is_present(self):
        # NOTE: this field cannot be stored as the line_id is usually removed
        # through cascade deletion, which means the compute would be false
        for option in self:
            option.is_present = bool(option.order_id.order_line.filtered(lambda l: l.product_id == option.product_id))

    def _search_is_present(self, operator, value):
        if (operator, value) in [('=', True), ('!=', False)]:
            return [('line_id', '=', False)]
        return [('line_id', '!=', False)]

    @api.onchange('product_id', 'uom_id', 'quantity')
    def _onchange_product_id(self):
        if not self.product_id:
            return
        product = self.product_id.with_context(
            lang=self.order_id.partner_id.lang,
            partner=self.order_id.partner_id,
            quantity=self.quantity,
            date=self.order_id.date_order,
            pricelist=self.order_id.pricelist_id.id,
            uom=self.uom_id.id,
            fiscal_position=self.env.context.get('fiscal_position')
        )
        self.name = product.get_product_multiline_description_sale()
        self.uom_id = self.uom_id or product.uom_id
        domain = {'uom_id': [('category_id', '=', self.product_id.uom_id.category_id.id)]}
        # To compute the dicount a so line is created in cache
        values = self._get_values_to_add_to_order()
        new_sol = self.env['sale.order.line'].new(values)
        new_sol._onchange_discount()
        self.discount = new_sol.discount
        if self.order_id.pricelist_id and self.order_id.partner_id:
            self.price_unit = new_sol._get_display_price(product)
        return {'domain': domain}

    def button_add_to_order(self):
        self.add_option_to_order()

    def add_option_to_order(self):
        self.ensure_one()

        sale_order = self.order_id

        if sale_order.state not in ['draft', 'sent']:
            raise UserError(_('You cannot add options to a confirmed order.'))

        values = self._get_values_to_add_to_order()
        order_line = self.env['sale.order.line'].create(values)
        order_line._compute_tax_id()

        self.write({'line_id': order_line.id})
        if sale_order:
            sale_order.add_option_to_order_with_taxcloud()


    def _get_values_to_add_to_order(self):
        self.ensure_one()
        return {
            'order_id': self.order_id.id,
            'price_unit': self.price_unit,
            'name': self.name,
            'product_id': self.product_id.id,
            'product_uom_qty': self.quantity,
            'product_uom': self.uom_id.id,
            'discount': self.discount,
            'company_id': self.order_id.company_id.id,
        }

```

## File: models\sale_order_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError, ValidationError


class SaleOrderTemplate(models.Model):
    _name = "sale.order.template"
    _description = "Quotation Template"

    def _get_default_require_signature(self):
        return self.env.company.portal_confirmation_sign

    def _get_default_require_payment(self):
        return self.env.company.portal_confirmation_pay

    name = fields.Char('Quotation Template', required=True)
    sale_order_template_line_ids = fields.One2many('sale.order.template.line', 'sale_order_template_id', 'Lines', copy=True)
    note = fields.Text('Terms and conditions', translate=True)
    sale_order_template_option_ids = fields.One2many('sale.order.template.option', 'sale_order_template_id', 'Optional Products', copy=True)
    number_of_days = fields.Integer('Quotation Duration',
        help='Number of days for the validity date computation of the quotation')
    require_signature = fields.Boolean('Online Signature', default=_get_default_require_signature, help='Request a online signature to the customer in order to confirm orders automatically.')
    require_payment = fields.Boolean('Online Payment', default=_get_default_require_payment, help='Request an online payment to the customer in order to confirm orders automatically.')
    mail_template_id = fields.Many2one(
        'mail.template', 'Confirmation Mail',
        domain=[('model', '=', 'sale.order')],
        help="This e-mail template will be sent on confirmation. Leave empty to send nothing.")
    active = fields.Boolean(default=True, help="If unchecked, it will allow you to hide the quotation template without removing it.")
    company_id = fields.Many2one('res.company', string='Company')

    @api.constrains('company_id', 'sale_order_template_line_ids', 'sale_order_template_option_ids')
    def _check_company_id(self):
        for template in self:
            companies = template.mapped('sale_order_template_line_ids.product_id.company_id') | template.mapped('sale_order_template_option_ids.product_id.company_id')
            if len(companies) > 1:
                raise ValidationError(_("Your template cannot contain products from multiple companies."))
            elif companies and companies != template.company_id:
                raise ValidationError((_("Your template contains products from company %s whereas your template belongs to company %s. \n Please change the company of your template or remove the products from other companies.") % (companies.mapped('display_name'), template.company_id.display_name)))

    @api.onchange('sale_order_template_line_ids', 'sale_order_template_option_ids')
    def _onchange_template_line_ids(self):
        companies = self.mapped('sale_order_template_option_ids.product_id.company_id') | self.mapped('sale_order_template_line_ids.product_id.company_id')
        if companies and self.company_id not in companies:
            self.company_id = companies[0]

    @api.model_create_multi
    def create(self, vals_list):
        records = super(SaleOrderTemplate, self).create(vals_list)
        records._update_product_translations()
        return records

    def write(self, vals):
        if 'active' in vals and not vals.get('active'):
            template_id = self.env['ir.default'].get('sale.order', 'sale_order_template_id')
            for template in self:
                if template_id and template_id == template.id:
                    raise UserError(_('Before archiving "%s" please select another default template in the settings.') % template.name)
        result = super(SaleOrderTemplate, self).write(vals)
        self._update_product_translations()
        return result

    def _update_product_translations(self):
        languages = self.env['res.lang'].search([('active', '=', 'true')])
        for lang in languages:
            for line in self.sale_order_template_line_ids:
                if line.name == line.product_id.get_product_multiline_description_sale():
                    self.create_or_update_translations(model_name='sale.order.template.line,name', lang_code=lang.code,
                                                       res_id=line.id,src=line.name,
                                                       value=line.product_id.with_context(lang=lang.code).get_product_multiline_description_sale())
            for option in self.sale_order_template_option_ids:
                if option.name == option.product_id.get_product_multiline_description_sale():
                    self.create_or_update_translations(model_name='sale.order.template.option,name', lang_code=lang.code,
                                                       res_id=option.id,src=option.name,
                                                       value=option.product_id.with_context(lang=lang.code).get_product_multiline_description_sale())

    def create_or_update_translations(self, model_name, lang_code, res_id, src, value):
        data = {
            'type': 'model',
            'name': model_name,
            'lang': lang_code,
            'res_id': res_id,
            'src': src,
            'value': value,
            'state': 'inprogress',
        }
        existing_trans = self.env['ir.translation'].search([('name', '=', model_name),
                                                            ('res_id', '=', res_id),
                                                            ('lang', '=', lang_code)])
        if not existing_trans:
            self.env['ir.translation'].create(data)
        else:
            existing_trans.write(data)


class SaleOrderTemplateLine(models.Model):
    _name = "sale.order.template.line"
    _description = "Quotation Template Line"
    _order = 'sale_order_template_id, sequence, id'

    sequence = fields.Integer('Sequence', help="Gives the sequence order when displaying a list of sale quote lines.",
        default=10)
    sale_order_template_id = fields.Many2one(
        'sale.order.template', 'Quotation Template Reference',
        required=True, ondelete='cascade', index=True)
    company_id = fields.Many2one('res.company', related='sale_order_template_id.company_id', store=True, index=True)
    name = fields.Text('Description', required=True, translate=True)
    product_id = fields.Many2one(
        'product.product', 'Product', check_company=True,
        domain=[('sale_ok', '=', True)])
    price_unit = fields.Float('Unit Price', required=True, digits='Product Price')
    discount = fields.Float('Discount (%)', digits='Discount', default=0.0)
    product_uom_qty = fields.Float('Quantity', required=True, digits='Product UoS', default=1)
    product_uom_id = fields.Many2one('uom.uom', 'Unit of Measure', domain="[('category_id', '=', product_uom_category_id)]")
    product_uom_category_id = fields.Many2one(related='product_id.uom_id.category_id', readonly=True)

    display_type = fields.Selection([
        ('line_section', "Section"),
        ('line_note', "Note")], default=False, help="Technical field for UX purpose.")

    @api.onchange('product_id')
    def _onchange_product_id(self):
        self.ensure_one()
        if self.product_id:
            name = self.product_id.display_name
            if self.product_id.description_sale:
                name += '\n' + self.product_id.description_sale
            self.name = name
            self.price_unit = self.product_id.lst_price
            self.product_uom_id = self.product_id.uom_id.id

    @api.onchange('product_uom_id')
    def _onchange_product_uom(self):
        if self.product_id and self.product_uom_id:
            self.price_unit = self.product_id.uom_id._compute_price(self.product_id.lst_price, self.product_uom_id)

    @api.model
    def create(self, values):
        if values.get('display_type', self.default_get(['display_type'])['display_type']):
            values.update(product_id=False, price_unit=0, product_uom_qty=0, product_uom_id=False)
        return super(SaleOrderTemplateLine, self).create(values)

    def write(self, values):
        if 'display_type' in values and self.filtered(lambda line: line.display_type != values.get('display_type')):
            raise UserError(_("You cannot change the type of a sale quote line. Instead you should delete the current line and create a new line of the proper type."))
        return super(SaleOrderTemplateLine, self).write(values)

    _sql_constraints = [
        ('accountable_product_id_required',
            "CHECK(display_type IS NOT NULL OR (product_id IS NOT NULL AND product_uom_id IS NOT NULL))",
            "Missing required product and UoM on accountable sale quote line."),

        ('non_accountable_fields_null',
            "CHECK(display_type IS NULL OR (product_id IS NULL AND price_unit = 0 AND product_uom_qty = 0 AND product_uom_id IS NULL))",
            "Forbidden product, unit price, quantity, and UoM on non-accountable sale quote line"),
    ]


class SaleOrderTemplateOption(models.Model):
    _name = "sale.order.template.option"
    _description = "Quotation Template Option"
    _check_company_auto = True

    sale_order_template_id = fields.Many2one('sale.order.template', 'Quotation Template Reference', ondelete='cascade',
        index=True, required=True)
    company_id = fields.Many2one('res.company', related='sale_order_template_id.company_id', store=True, index=True)
    name = fields.Text('Description', required=True, translate=True)
    product_id = fields.Many2one(
        'product.product', 'Product', domain=[('sale_ok', '=', True)],
        required=True, check_company=True)
    price_unit = fields.Float('Unit Price', required=True, digits='Product Price')
    discount = fields.Float('Discount (%)', digits='Discount')
    uom_id = fields.Many2one('uom.uom', 'Unit of Measure ', required=True, domain="[('category_id', '=', product_uom_category_id)]")
    product_uom_category_id = fields.Many2one(related='product_id.uom_id.category_id', readonly=True)
    quantity = fields.Float('Quantity', required=True, digits='Product UoS', default=1)

    @api.onchange('product_id')
    def _onchange_product_id(self):
        if not self.product_id:
            return
        product = self.product_id
        self.price_unit = product.lst_price
        name = product.name
        if self.product_id.description_sale:
            name += '\n' + self.product_id.description_sale
        self.name = name
        self.uom_id = product.uom_id
        domain = {'uom_id': [('category_id', '=', self.product_id.uom_id.category_id.id)]}
        return {'domain': domain}

    @api.onchange('uom_id')
    def _onchange_product_uom(self):
        if not self.product_id:
            return
        if not self.uom_id:
            self.price_unit = 0.0
            return
        if self.uom_id.id != self.product_id.uom_id.id:
            self.price_unit = self.product_id.uom_id._compute_price(self.product_id.lst_price, self.uom_id)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import digest
from . import res_config_settings
from . import sale_order
from . import sale_order_template

```

## File: report\sale_report_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<template id="report_saleorder_document_inherit_sale_management" inherit_id="sale.report_saleorder_document">
    <xpath expr="//div[@name='signature']" position="after">
        <div t-if="doc.sale_order_option_ids and doc.state in ['draft', 'sent']">
            <t t-set="has_option_discount" t-value="any(doc.sale_order_option_ids.filtered(lambda o: o.discount != 0.0))" />
            <h4 name="h_optional_products">
                <span>Options</span>
            </h4>
            <table name="table_optional_products" class="table table-sm">
                <thead>
                    <tr>
                        <th name="th_option_name" class="text-left">Description</th>
                        <th t-if="has_option_discount" name="th_option_discount" groups="product.group_discount_per_so_line" class="text-left">Disc.%</th>
                        <th name="th_option_price_unit" class="text-right">Unit Price</th>
                    </tr>
                </thead>
                <tbody class="sale_tbody">
                    <tr t-foreach="doc.sale_order_option_ids" t-as="option">
                        <td name="td_option_name">
                            <span t-field="option.name"/>
                        </td>
                        <td t-if="has_option_discount" name="td_option_discount" groups="product.group_discount_per_so_line">
                            <strong t-if="option.discount != 0.0" class="text-info">
                                <t t-esc="((option.discount % 1) and '%s' or '%d') % option.discount"/>%
                            </strong>
                        </td>
                        <td name="td_option_price_unit">
                            <strong class="text-right">
                                <div t-field="option.price_unit"
                                    t-options='{"widget": "monetary", "display_currency": doc.pricelist_id.currency_id}'
                                    t-att-style="option.discount and 'text-decoration: line-through' or None"
                                    t-att-class="option.discount and 'text-danger' or None"/>
                                <div t-if="option.discount">
                                    <t t-esc="'%.2f' % ((1-option.discount / 100.0) * option.price_unit)"/>
                                </div>
                            </strong>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </xpath>
</template>
</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_sale_order_template,sale.order.template,model_sale_order_template,sales_team.group_sale_salesman,1,0,0,0
access_sale_order_template_manager,sale.order.template,model_sale_order_template,sales_team.group_sale_manager,1,1,1,1
access_sale_order_template_system,sale.order.template,model_sale_order_template,base.group_system,1,0,0,0
access_sale_order_template_line,sale.order.template.line,model_sale_order_template_line,sales_team.group_sale_salesman,1,0,0,0
access_sale_order_template_line_manager,sale.order.template.line,model_sale_order_template_line,sales_team.group_sale_manager,1,1,1,1
access_sale_order_template_option,sale.order.template.option,model_sale_order_template_option,sales_team.group_sale_salesman,1,0,0,0
access_sale_order_template_option_manager,sale.order.template.option,model_sale_order_template_option,sales_team.group_sale_manager,1,1,1,1
access_sale_order_option,sale.order.option,model_sale_order_option,sales_team.group_sale_salesman,1,1,1,1
access_sale_order_option_all,sale.order.option,model_sale_order_option,,1,0,0,0

```

## File: security\sale_management_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="group_sale_order_template" model="res.groups">
        <field name="name">Quotation Templates</field>
        <field name="category_id" ref="base.module_category_hidden"/>
    </record>

    <record id="sale_order_template_rule_company" model="ir.rule">
        <field name="name">Quotation Template multi-company</field>
        <field name="model_id" ref="model_sale_order_template"/>
        <field name="global" eval="True"/>
        <field name="domain_force">['|', ('company_id', '=', False), ('company_id', 'in', company_ids)]</field>
    </record>

</odoo>

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#DA956B"/><stop offset="100%" stop-color="#CC7039"/></linearGradient><path id="d" d="M56.243 52.279c.578 0 1.05.537 1.05 1.193v.978c0 .656-.472 1.194-1.05 1.194H13.532c-.578 0-1.05-.538-1.05-1.194v-35.8c0-.657.472-1.194 1.05-1.194h1.5c.578 0 1.05.537 1.05 1.194v33.629h40.161zM39 23.025l4.981 4.963-6.302 7.25-4.866-5.53c-.411-.467-1.068-.467-1.48 0L20.92 41.423a1.31 1.31 0 0 0-.018 1.68l2.494 2.924c.412.477 1.086.487 1.497.01l7.186-8.165 4.857 5.52a.965.965 0 0 0 1.488 0l9.558-10.86L53 37.664 55 20l-16 3.025z"/><path id="e" d="M56.243 50.279c.578 0 1.05.537 1.05 1.193v.978c0 .656-.472 1.194-1.05 1.194H13.532c-.578 0-1.05-.538-1.05-1.194v-35.8c0-.657.472-1.194 1.05-1.194h1.5c.578 0 1.05.537 1.05 1.194v33.629h40.161zM39 21.025l4.981 4.963-6.302 7.25-4.866-5.53c-.411-.467-1.068-.467-1.48 0L20.92 39.423a1.31 1.31 0 0 0-.018 1.68l2.494 2.924c.412.477 1.086.487 1.497.01l7.186-8.165 4.857 5.52a.965.965 0 0 0 1.488 0l9.558-10.86L53 35.664 55 18l-16 3.025z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M45.243 69H4c-2 0-4-.146-4-4.077V35.315L13 16h3v26.5l15-14.27.974.024L40 21.096l13 14.27-18 18.346h22L45.243 69z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: static\src\js\sale_management.js

```javascript
odoo.define('sale_management.sale_management', function (require) {
'use strict';

var publicWidget = require('web.public.widget');

publicWidget.registry.SaleUpdateLineButton = publicWidget.Widget.extend({
    selector: '.o_portal_sale_sidebar a.js_update_line_json',
    events: {
        'click': '_onClick',
    },
    /**
     * @override
     */
    start: function () {
        var self = this;
            return this._super.apply(this, arguments).then(function () {
                self.elems = self._getUpdatableElements();
                 self.elems.$lineQuantity.change(function (ev) {
                    var quantity = parseInt(this.value);
                    self._onChangeQuantity(quantity);
                });
            });
        },
    /**
     * Process the change in line quantity
     *
     * @private
     * @param {Int} quantity, the new quantity of the line
     *    If not present it will increment/decrement the existing quantity
     */
    _onChangeQuantity: function (quantity) {
        var href = this.$el.attr("href");
        var orderID = href.match(/my\/orders\/([0-9]+)/);
        var lineID = href.match(/update_line\/([0-9]+)/);
        var params = {
                'line_id': parseInt(lineID[1]),
                'remove': this.$el.is('[href*="remove"]'),
                'unlink': this.$el.is('[href*="unlink"]'),
                'input_quantity': quantity >= 0 ? quantity : false,
        };
            var token = href.match(/token=([\w\d-]*)/)[1];
        if (token) {
            params['access_token'] = token;
        }

        orderID = parseInt(orderID[1]);
        this._callUpdateLineRoute(orderID, params).then(this._updateOrderValues.bind(this));
    },
    /**
     * Reacts to the click on the -/+ buttons
     *
     * @param {Event} ev
     */
    _onClick: function (ev) {
        ev.preventDefault();
        return this._onChangeQuantity();
    },
    /**
     * Calls the route to get updated values of the line and order
     * when the quantity of a product has changed
     *
     * @private
     * @param {integer} order_id
     * @param {Object} params
     * @return {Deferred}
     */
    _callUpdateLineRoute: function (order_id, params) {
        var url = "/my/orders/" + order_id + "/update_line_dict";
        return this._rpc({
            route: url,
            params: params,
        });
    },
    /**
     * Processes data from the server to update the UI
     *
     * @private
     * @param {Object} data: contains order and line updated values
     */
    _updateOrderValues: function (data) {
        if (!data) {
            window.location.reload();
        }

        var orderAmountTotal = data.order_amount_total;
        var orderAmountUntaxed = data.order_amount_untaxed;
        var orderAmountTax = data.order_amount_tax;
        var orderAmountUndiscounted = data.order_amount_undiscounted;
        var orderTotalsTable = $(data.order_totals_table);

        var lineProductUomQty = data.order_line_product_uom_qty;
        var linePriceTotal = data.order_line_price_total;
        var linePriceSubTotal = data.order_line_price_subtotal;

        this.elems.$lineQuantity.val(lineProductUomQty);

        if (this.elems.$linePriceTotal.length && linePriceTotal !== undefined) {
            this.elems.$linePriceTotal.text(linePriceTotal);
        }
        if (this.elems.$linePriceSubTotal.length && linePriceSubTotal !== undefined) {
            this.elems.$linePriceSubTotal.text(linePriceSubTotal);
        }

        if (orderAmountUntaxed !== undefined) {
            this.elems.$orderAmountUntaxed.text(orderAmountUntaxed);
        }

        if (orderAmountTotal !== undefined) {
            this.elems.$orderAmountTotal.text(orderAmountTotal);
        }

        if (orderAmountUndiscounted !== undefined) {
            this.elems.$orderAmountUndiscounted.text(orderAmountUndiscounted);
        }
        if (orderTotalsTable) {
            this.elems.$orderTotalsTable.find('table').replaceWith(orderTotalsTable);
        }
    },
    /**
     * Locate in the DOM the elements to update
     * Mostly for compatibility, when the module has not been upgraded
     * In that case, we need to fall back to some other elements
     *
     * @private
     * @return {Object}: Jquery elements to update
     */
    _getUpdatableElements: function () {
        var $parentTr = this.$el.parents('tr:first');
        var $linePriceTotal = $parentTr.find('.oe_order_line_price_total .oe_currency_value');
        var $linePriceSubTotal = $parentTr.find('.oe_order_line_price_subtotal .oe_currency_value');

        if (!$linePriceTotal.length && !$linePriceSubTotal.length) {
            $linePriceTotal = $linePriceSubTotal = $parentTr.find('.oe_currency_value').last();
        }

        var $orderAmountUntaxed = $('[data-id="total_untaxed"]').find('span, b');
        var $orderAmountTotal = $('[data-id="total_amount"]').find('span, b');
        var $orderAmountUndiscounted = $('[data-id="amount_undiscounted"]').find('span, b');

        if (!$orderAmountUntaxed.length) {
            $orderAmountUntaxed = $orderAmountTotal.eq(1);
            $orderAmountTotal = $orderAmountTotal.eq(0).add($orderAmountTotal.eq(2));
        }

        return {
            $lineQuantity: this.$el.closest('.input-group').find('.js_quantity'),
            $linePriceSubTotal: $linePriceSubTotal,
            $linePriceTotal: $linePriceTotal,
            $orderAmountUntaxed: $orderAmountUntaxed,
            $orderAmountTotal: $orderAmountTotal,
            $orderTotalsTable: $('#total'),
            $orderAmountUndiscounted: $orderAmountUndiscounted,
        };
    }
});
});

```

## File: views\digest_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="digest_digest_view_form" model="ir.ui.view">
        <field name="name">digest.digest.view.form.inherit.sale.order</field>
        <field name="model">digest.digest</field>
        <field name="inherit_id" ref="digest.digest_digest_view_form" />
        <field name="arch" type="xml">
            <xpath expr="//group[@name='kpi_sales']" position="attributes">
                <attribute name="string">Sales</attribute>
                <attribute name="groups">sales_team.group_sale_salesman_all_leads</attribute>
            </xpath>
            <xpath expr="//group[@name='kpi_sales']" position="inside">
                <field name="kpi_all_sale_total"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.sale.management</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="sale.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='confirmation_email_setting']" position="after">
                <div class="col-12 col-lg-6 o_setting_box">
                    <div class="o_setting_left_pane">
                        <field name="group_sale_order_template"/>
                        <field name="module_sale_quotation_builder" invisible="1"/>
                    </div>
                    <div class="o_setting_right_pane">
                        <label for="group_sale_order_template"/>
                        <div class="text-muted">
                            Create standardized offers with default products
                        </div>
                        <div class="content-group" attrs="{'invisible': [('group_sale_order_template', '=', False)]}">
                            <div class="mt16">
                                <label for="default_sale_order_template_id" class="o_light_label"/>
                                <field name="default_sale_order_template_id" class="oe_inline"/>
                            </div>
                            <div class="mt8">
                                <button name="%(sale_management.sale_order_template_action)d" icon="fa-arrow-right" type="action" string="Quotation Templates" class="btn-link"/>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="col-12 col-lg-6 o_setting_box" attrs="{'invisible': [('group_sale_order_template','=',False)]}">
                    <div class="o_setting_left_pane">
                        <field name="module_sale_quotation_builder"/>
                    </div>
                    <div class="o_setting_right_pane">
                        <label for="module_sale_quotation_builder"/>
                        <div class="text-muted">
                            Design your quotation templates using building blocks<br/>
                            <em attrs="{'invisible': [('module_sale_quotation_builder','=',False)]}">Warning: this option will install the Website app.</em>
                        </div>
                    </div>
                </div>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\sale_management_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="sale.sale_menu_root" model="ir.ui.menu">
        <field name="active" eval="True"/>
    </record>
    <record id="sale.menu_report_product_all" model="ir.ui.menu">
        <field name="active" eval="True"/>
    </record>
    <record id="sale.menu_sales_config" model="ir.ui.menu">
        <field name="active" eval="True"/>
    </record>
    <record id="sale.product_menu_catalog" model="ir.ui.menu">
        <field name="active" eval="True"/>
    </record>
    <record id="sale.menu_product" model="ir.ui.menu">
        <field name="active" eval="True"/>
    </record>
    <record id="sale.menu_product_template_action" model="ir.ui.menu">
        <field name="active" eval="True"/>
    </record>
    <record id="sale.prod_config_main" model="ir.ui.menu">
        <field name="active" eval="True"/>
    </record>
    <record id="sale.menu_products" model="ir.ui.menu">
        <field name="active" eval="True"/>
    </record>
    <record id="sale.next_id_16" model="ir.ui.menu">
        <field name="active" eval="True"/>
    </record>
    <record id="sale.menu_product_uom_form_action" model="ir.ui.menu">
        <field name="active" eval="True"/>
    </record>
    <record id="sale.menu_product_uom_categ_form_action" model="ir.ui.menu">
        <field name="active" eval="True"/>
    </record>
    <record id="sale.menu_product_pricelist_main" model="ir.ui.menu">
        <field name="active" eval="True"/>
    </record>
    <record id="sale.menu_sale_order" model="ir.ui.menu">
        <field name="active" eval="True"/>
    </record>
    <record id="sale.menu_sale_invoicing" model="ir.ui.menu">
        <field name="active" eval="True"/>
    </record>
    <record id="sale.menu_sale_order_invoice" model="ir.ui.menu">
        <field name="active" eval="True"/>
    </record>
    <record id="sale.menu_sale_order_upselling" model="ir.ui.menu">
        <field name="active" eval="True"/>
    </record>
    <record id="sale.menu_sale_quotations" model="ir.ui.menu">
        <field name="active" eval="True"/>
    </record>

    <record id="res_config_settings_view_form_inherit_sale_management" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.sale.management</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="sale.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@data-key='sale_management']" position="attributes">
                <attribute name="class">app_settings_block</attribute>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\sale_order_template_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="sale_order_template_view_search" model="ir.ui.view">
        <field name="name">sale.order.template.search</field>
        <field name="model">sale.order.template</field>
        <field name="arch" type="xml">
            <search string="Search Quotation Template">
                <field name="name"/>
                <filter string="Archived" name="inactive" domain="[('active','=', False)]"/>
            </search>
        </field>
    </record>

    <record id="sale_order_template_view_form" model="ir.ui.view">
        <field name="name">sale.order.template.form</field>
        <field name="model">sale.order.template</field>
        <field name="type">form</field>
        <field name="arch" type="xml">
            <form string="Quotation Template">
                <sheet>
                    <div name="button_box" class="oe_button_box"/>
                    <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                    <div class="oe_title">
                        <label for="name" class="oe_edit_only"/>
                        <h1>
                            <field name="name" placeholder="Quotation Template"/>
                        </h1>
                    </div>
                    <group>
                        <group>
                            <field name="active" invisible="1"/>
                           <label for="number_of_days" string="Quotation expires after"/>
                           <div id="number_of_days">
                               <field name="number_of_days" class="oe_inline"/> days
                           </div>
                        </group>
                        <group>
                            <field name="company_id" options="{'no_create': True}" groups="base.group_multi_company"/>
                        </group>
                    </group>
                    <notebook name="main_book">
                        <page string="Lines">
                        <field name="sale_order_template_line_ids" widget="section_and_note_one2many">
                            <form string="Quotation Template Lines">
                                <!--
                                    We need the sequence field to be here for new lines to be added at the correct position.
                                    TODO: at some point we want to fix this in the framework so that an invisible field is not required.
                                -->
                                <field name="sequence" invisible="1"/>
                                <field name="display_type" invisible="1"/>
                                <field name="product_uom_category_id" invisible="1"/>
                                <group attrs="{'invisible': [('display_type', '!=', False)]}">
                                    <group>
                                        <field name="product_id" attrs="{'required': [('display_type', '=', False)]}"/>
                                        <label for="product_uom_qty"/>
                                        <div>
                                            <field name="product_uom_qty" class="oe_inline"/>
                                        </div>
                                        <field name="price_unit"/>
                                        <label for="discount" groups="product.group_discount_per_so_line"/>
                                        <div groups="product.group_discount_per_so_line">
                                            <field name="discount" class="oe_inline"/> %%
                                        </div>
                                    </group>
                                </group>
                                <notebook colspan="4" name="description">
                                    <page string="Description" attrs="{'invisible': [('display_type', '!=', False)]}">
                                        <field name="name" />
                                    </page>
                                    <page string="Section" attrs="{'invisible': [('display_type', '!=', 'line_section')]}">
                                        <field name="name" />
                                    </page>
                                    <page string="Note" attrs="{'invisible': [('display_type', '!=', 'line_note')]}">
                                        <field name="name" />
                                    </page>
                                </notebook>
                            </form>
                            <tree string="Quotation Template Lines" editable="bottom">
                                <control>
                                    <create name="add_product_control" string="Add a product"/>
                                    <create name="add_section_control" string="Add a section" context="{'default_display_type': 'line_section'}"/>
                                    <create name="add_note_control" string="Add a note" context="{'default_display_type': 'line_note'}"/>
                                </control>

                                <field name="display_type" invisible="1"/>
                                <field name="sequence" widget="handle"/>
                                <field name="product_id"/>
                                <field name="name" widget="section_and_note_text"/>
                                <field name="product_uom_qty"/>
                                <field name="product_uom_category_id" invisible="1"/>
                                <field
                                    name="product_uom_id"
                                    groups="uom.group_uom"
                                    attrs="{'required': [('display_type', '=', False)]}"
                                />
                                <field name="discount" groups="product.group_discount_per_so_line"/>
                                <field name="price_unit"/>
                            </tree>
                        </field>
                    </page>
                    <page string="Optional Products">
                        <field name="sale_order_template_option_ids">
                          <tree string="Quotation Template Lines" editable="bottom">
                            <field name="product_id"/>
                            <field name="name"/>
                            <field name="quantity"/>
                            <field name="product_uom_category_id" invisible="1"/>
                            <field name="uom_id" groups="uom.group_uom"/>
                            <field name="price_unit"/>
                            <field name="discount" groups="product.group_discount_per_so_line"/>
                          </tree>
                        </field>
                    </page>
                    <page string="Confirmation">
                        <group>
                            <field name="require_signature"/>
                            <field name="require_payment"/>
                            <field name="mail_template_id" context="{'default_model':'sale.order'}"/>
                        </group>
                    </page>
                    </notebook>
                    <field name="note" nolabel="1"
                        placeholder="The Administrator can set default Terms &amp; Conditions in Sales Settings. Terms set here will show up instead if you select this quotation template."/>
                </sheet>
            </form>
        </field>
    </record>

    <record id="sale_order_template_view_tree" model="ir.ui.view">
        <field name="name">sale.order.template.tree</field>
        <field name="model">sale.order.template</field>
        <field name="type">tree</field>
        <field name="arch" type="xml">
            <tree string="Quotation Template">
                <field name="name"/>
            </tree>
        </field>
    </record>

    <record id="sale_order_template_action" model="ir.actions.act_window">
        <field name="name">Quotation Templates</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">sale.order.template</field>
        <field name="view_mode">tree,form</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create your quotation template
            </p><p>
                Use templates to create polished, professional quotes in minutes.
                Send these quotes by email and let your customers sign online.
                Use cross-selling and discounts to push and boost your sales.
            </p>
        </field>
    </record>

    <menuitem id="sale_order_template_menu" action="sale_order_template_action" parent="sale.menu_sales_config" sequence="1" name="Quotation Templates" groups="sale_management.group_sale_order_template"/>

</odoo>

```

## File: views\sale_order_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="sale_order_form_quote" model="ir.ui.view">
        <field name="name">sale.order.form.payment</field>
        <field name="model">sale.order</field>
        <field name="inherit_id" ref="sale.view_order_form"/>
        <field name="arch" type="xml">

            <xpath expr="//page/field[@name='order_line']/.." position="after">
                <page string="Optional Products" name="optional_products" attrs="{'invisible': [('state', 'not in', ['draft', 'sent'])]}">
                    <field name="sale_order_option_ids" mode="tree,form,kanban">
                        <form string="Optional Products">
                            <group>
                                <field name="product_id" domain="[('sale_ok', '=', True), '|', ('company_id', '=', False), ('company_id', '=', parent.company_id)]"/>
                                <field name="name"/>
                                <field name="quantity"/>
                                <field name="product_uom_category_id" invisible="1"/>
                                <field name="uom_id" groups="uom.group_uom"/>
                                <field name="price_unit"/>
                                <field name="discount" groups="product.group_discount_per_so_line"/>
                                <field name="is_present" />
                            </group>
                        </form>
                        <kanban class="o_kanban_mobile">
                            <field name="product_id"/>
                            <field name="quantity"/>
                            <field name="uom_id" groups="uom.group_uom"/>
                            <field name="price_unit"/>
                            <field name="is_present" />
                            <templates>
                                <t t-name="kanban-box">
                                    <div class="oe_kanban_card oe_kanban_global_click">
                                        <div class="row">
                                            <div class="col-10">
                                                <strong>
                                                    <span>
                                                        <t t-esc="record.product_id.value"/>
                                                    </span>
                                                </strong>
                                            </div>
                                            <div class="col-2">
                                                <button name="button_add_to_order" class="btn btn-link oe_link fa fa-shopping-cart" title="Add to order lines" type="object" attrs="{'invisible': [('is_present', '=', True)]}"/>
                                            </div>
                                        </div>
                                        <div class="row">
                                            <div class="col-12 text-muted">
                                                <span>
                                                    Quantity:
                                                    <t t-esc="record.quantity.value"/>
                                                    <t t-esc="record.uom_id.value"/>
                                                </span>
                                            </div>
                                        </div>
                                        <div class="row">
                                            <div class="col-12 text-muted">
                                                <span>
                                                    Unit Price:
                                                    <t t-esc="record.price_unit.value"/>
                                                </span>
                                            </div>
                                        </div>
                                    </div>
                                </t>
                            </templates>
                        </kanban>
                        <tree string="Sales Quotation Template Lines" editable="bottom" decoration-success="is_present == True">
                            <control>
                                <create name="add_product_control" string="Add a product"/>
                            </control>
                            <field name="sequence" widget="handle"/>
                            <field name="product_id" domain="[('sale_ok', '=', True), '|', ('company_id', '=', False), ('company_id', '=', parent.company_id)]"/>
                            <field name="name" optional="show"/>
                            <field name="quantity"/>
                            <field name="uom_id" string="UoM" groups="uom.group_uom" optional="show"/>
                            <field name="product_uom_category_id" invisible="1"/>
                            <field name="price_unit"/>
                            <field name="discount" string="Disc.%" groups="product.group_discount_per_so_line" optional="show"/>
                            <field name="is_present" invisible="1" />
                            <button name="button_add_to_order" class="oe_link" icon="fa-shopping-cart" title="Add to order lines" type="object" attrs="{'invisible': [('is_present', '=', True)]}"/>
                        </tree>
                    </field>
                </page>
            </xpath>

            <xpath expr="//field[@name='partner_shipping_id']" position="after">
                <field name="sale_order_template_id" context="{'company_id': company_id}"
                    options="{'no_create': True, 'no_open': True}"
                    groups="sale_management.group_sale_order_template"
                />
            </xpath>

        </field>
    </record>

    <record id="sale.action_quotations" model="ir.actions.act_window">
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a new quotation, the first step of a new sale!
            </p>
            <p>
                Once the quotation is confirmed by the customer, it becomes a sales order.<br/> You will be able to create an invoice and collect the payment.
            </p>
        </field>
    </record>

    <menuitem id="menu_product_attribute_action"
        action="product.attribute_action"
        parent="sale.prod_config_main"  groups="product.group_product_variant" sequence="1"/>
</odoo>

```

## File: views\sale_portal_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="assets_frontend" inherit_id="web.assets_frontend" name="Website Quote frontend assets">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/sale_management/static/src/js/sale_management.js"></script>
        </xpath>
    </template>

    <template id="sale_order_portal_content_inherit_sale_management" name="Order Options" inherit_id="sale.sale_order_portal_content">

        <xpath expr="//section[@id='details']//t[@t-set='display_discount']" position="after">
            <t t-set="display_remove" t-value="sale_order.state in ('draft', 'sent') and any([line.sale_order_option_ids for line in sale_order.order_line])"/>
        </xpath>

        <xpath expr="//section[@id='details']//table[@id='sales_order_table']/thead/tr" position="inside">
            <!-- add blank Tr in thead for layout conciseness -->
            <th t-if="display_remove">
            </th>
        </xpath>

        <xpath expr="//section[@id='details']//t[@t-if='not line.display_type']" position="inside">
            <td class="text-center" t-if="display_remove">
                <a t-att-href="sale_order.get_portal_url(suffix='/update_line/%s' % line.id, query_string='&amp;unlink=True')" class="mb8 js_update_line_json d-print-none" t-if="sale_order.state in ('draft', 'sent') and line.sale_order_option_ids" aria-label="Remove" title="Remove">
                    <span class="fa fa-trash-o"></span>
                </a>
            </td>
        </xpath>

        <xpath expr="//section[@id='signature']" position="after">
            <t t-if="any([(not option.is_present) for option in sale_order.sale_order_option_ids])">
                <section>
                    <h3>Options</h3>
                    <t t-set="display_discount" t-value="True in [option.discount > 0 for option in sale_order.sale_order_option_ids]"/>
                    <table class="table table-sm">
                        <thead>
                            <tr>
                                <th class="text-left">Product</th>
                                <th t-if="display_discount" class="text-right">Disc.%
                                </th>
                                <th class="text-right">Unit Price</th>
                                <th t-if="sale_order.state in ['draft', 'sent'] and report_type == 'html'"></th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr t-foreach="sale_order.sale_order_option_ids" t-as="option">
                                <t t-if="not option.is_present">
                                    <td>
                                        <div t-field="option.name"/>
                                    </td>
                                    <td t-if="display_discount" class="text-right">
                                        <strong t-if="option.discount" class="text-info">
                                            <t t-esc="((option.discount % 1) and '%s' or '%d') % option.discount"/>%
                                        </strong>
                                    </td>
                                    <td>
                                        <strong class="text-right">
                                            <div t-field="option.price_unit"
                                                t-options='{"widget": "monetary", "display_currency": sale_order.pricelist_id.currency_id}'
                                                t-att-style="option.discount and 'text-decoration: line-through' or None"
                                                t-att-class="option.discount and 'text-danger' or None"/>
                                            <div t-if="option.discount">
                                                <t t-esc="(1-option.discount / 100.0) * option.price_unit" t-options='{"widget": "monetary", "display_currency": sale_order.pricelist_id.currency_id}'/>
                                            </div>
                                        </strong>
                                    </td>
                                    <td class="text-center" t-if="sale_order.state in ['draft', 'sent'] and report_type == 'html'">
                                        <a t-att-href="sale_order.get_portal_url(suffix='/add_option/%s' % option.id)" class="mb8 d-print-none" aria-label="Add to cart" title="Add to cart">
                                            <span class="fa fa-lg fa-shopping-cart"/>
                                        </a>
                                    </td>
                                </t>
                            </tr>
                        </tbody>
                    </table>
                </section>
            </t>
        </xpath>

        <xpath expr="//section[@id='details']//div[@id='quote_qty']" position="replace">
            <t t-if="sale_order.state in ['draft', 'sent'] and line.sale_order_option_ids">
                <div class="input-group js_quantity_container pull-right">

                    <div class="input-group-prepend d-print-none">
                        <span class="input-group-text d-none d-md-inline-block">
                            <a t-att-href="sale_order.get_portal_url(suffix='/update_line/%s' % line.id, query_string='&amp;remove=True')" class="js_update_line_json" aria-label="Remove one" title="Remove one">
                                <span class="fa fa-minus"/>
                            </a>
                        </span>
                    </div>
                    <!-- TODO add uom in this case too -->
                    <input type="text" class="js_quantity form-control" t-att-data-id="line.id" t-att-value="line.product_uom_qty"/>
                    <div class="input-group-append d-print-none">
                        <span class="input-group-text d-none d-md-inline-block">
                            <a t-att-href="sale_order.get_portal_url(suffix='/update_line/%s' % line.id)" class="js_update_line_json" aria-label="Add one" title="Add one">
                                <span class="fa fa-plus"/>
                            </a>
                        </span>
                    </div>
                </div>
            </t>
            <t t-else="">
                <span t-field="line.product_uom_qty"/>
                <span t-field="line.product_uom" groups="uom.group_uom"/>
            </t>
        </xpath>

    </template>

</odoo>

```

