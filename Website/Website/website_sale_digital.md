# Odoo Module: website_sale_digital

Category: Website/Website

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- encoding: utf-8 -*-
from . import controllers
from . import models

```

## File: __manifest__.py

```python
# -*- encoding: utf-8 -*-
{
    'name': 'Digital Products',
    'version': '0.1',
    'summary': 'Sell digital products in your eCommerce store',
    'category': 'Website/Website',
    'description': """
Sell e-goods in your eCommerce store (e.g. webinars, articles, e-books, video tutorials).
To do so, create the product and attach the file to share via the *Files* button of the product form.
Once the order is paid, the file is made available in the order confirmation page and in the customer portal.
    """,
    'depends': [
        'attachment_indexation',
        'website_sale',
    ],
    'installable': True,
    'data': [
        'views/website_sale_digital.xml',
        'views/website_sale_digital_view.xml',
    ],
    'demo': [
        'data/product_demo.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
import io
import os
import mimetypes
from werkzeug.utils import redirect

from odoo import http
from odoo.exceptions import AccessError
from odoo.http import request
from odoo.addons.sale.controllers.portal import CustomerPortal
from odoo.addons.website_sale.controllers.main import WebsiteSale


class WebsiteSaleDigitalConfirmation(WebsiteSale):
    @http.route([
        '/shop/confirmation',
    ], type='http', auth="public", website=True)
    def payment_confirmation(self, **post):
        response = super(WebsiteSaleDigitalConfirmation, self).payment_confirmation(**post)
        order_lines = response.qcontext['order'].order_line
        digital_content = any(x.product_id.type == 'digital' for x in order_lines)
        response.qcontext.update(digital=digital_content)
        return response


class WebsiteSaleDigital(CustomerPortal):
    orders_page = '/my/orders'

    @http.route([
        '/my/orders/<int:order_id>',
    ], type='http', auth='public', website=True)
    def portal_order_page(self, order_id=None, **post):
        response = super(WebsiteSaleDigital, self).portal_order_page(order_id=order_id, **post)
        if not 'sale_order' in response.qcontext:
            return response
        order = response.qcontext['sale_order']
        invoiced_lines = request.env['account.move.line'].sudo().search([('move_id', 'in', order.invoice_ids.ids), ('move_id.payment_state', 'in', ['paid', 'in_payment'])])
        products = invoiced_lines.mapped('product_id') | order.order_line.filtered(lambda r: not r.price_subtotal).mapped('product_id')
        if not order.amount_total:
            # in that case, we should add all download links to the products
            # since there is nothing to pay, so we shouldn't wait for an invoice
            products = order.order_line.mapped('product_id')

        Attachment = request.env['ir.attachment'].sudo()
        purchased_products_attachments = {}
        for product in products.filtered(lambda p: p.attachment_count):
            # Search for product attachments
            product_id = product.id
            template = product.product_tmpl_id
            att = Attachment.sudo().search_read(
                domain=['|', '&', ('res_model', '=', product._name), ('res_id', '=', product_id), '&', ('res_model', '=', template._name), ('res_id', '=', template.id), ('product_downloadable', '=', True)],
                fields=['name', 'write_date'],
                order='write_date desc',
            )

            # Ignore products with no attachments
            if not att:
                continue

            purchased_products_attachments[product_id] = att

        response.qcontext.update({
            'digital_attachments': purchased_products_attachments,
        })
        return response

    @http.route([
        '/my/download',
    ], type='http', auth='public')
    def download_attachment(self, attachment_id):
        # Check if this is a valid attachment id
        attachment = request.env['ir.attachment'].sudo().search_read(
            [('id', '=', int(attachment_id))],
            ["name", "datas", "mimetype", "res_model", "res_id", "type", "url"]
        )

        if attachment:
            attachment = attachment[0]
        else:
            return redirect(self.orders_page)

        try:
            request.env['ir.attachment'].browse(attachment_id).check('read')
        except AccessError:  # The user does not have read access on the attachment.
            # Check if access can be granted through their purchases.
            res_model = attachment['res_model']
            res_id = attachment['res_id']
            digital_purchases = request.env['account.move.line'].get_digital_purchases()
            if res_model == 'product.product':
                purchased_product_ids = digital_purchases
            elif res_model == 'product.template':
                purchased_product_ids = request.env['product.product'].sudo().browse(
                    digital_purchases
                ).mapped('product_tmpl_id').ids
            else:
                purchased_product_ids = []  # The purchases must be related to products.
            if res_id not in purchased_product_ids:  # No related purchase was found.
                return redirect(self.orders_page)  # Prevent the user from downloading.

        # The user has bought the product, or has the rights to the attachment
        if attachment["type"] == "url":
            if attachment["url"]:
                return redirect(attachment["url"])
            else:
                return request.not_found()
        elif attachment["datas"]:
            data = io.BytesIO(base64.standard_b64decode(attachment["datas"]))
            # we follow what is done in ir_http's binary_content for the extension management
            extension = os.path.splitext(attachment["name"] or '')[1]
            extension = extension if extension else mimetypes.guess_extension(attachment["mimetype"] or '')
            filename = attachment['name']
            filename = filename if os.path.splitext(filename)[1] else filename + extension
            return http.send_file(data, filename=filename, as_attachment=True)
        else:
            return request.not_found()

```

## File: controllers\__init__.py

```python
# -*- encoding: utf-8 -*-
from . import main

```

## File: data\product_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="website_sale_digital.product_1" model="product.template">
        <field name="name">eBook: Office Renovation for Dummies</field>
        <field name="standard_price">2</field>
        <field name="list_price">4.50</field>
        <field name="type">service</field>
        <field name="is_published" eval="True"/>
        <field name="image_1920" type="base64" file="website_sale_digital/static/digital_product_1.jpg"/>
        <field name="categ_id" ref="product.product_category_6"/>
    </record>

    <record id="website_sale_digital.attach1" model="ir.attachment">
        <field name="name">ebook.pdf</field>
        <field name="type">binary</field>
        <field name="datas" type="base64" file="website_sale_digital/static/ebook.pdf"/>
        <field name="res_id" ref="website_sale_digital.product_1"/>
        <field name="res_model">product.template</field>
        <field name="product_downloadable">True</field>
    </record>

</odoo>

```

## File: models\account_invoice.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class AccountInvoiceLine(models.Model):

    _inherit = ['account.move.line']

    def get_digital_purchases(self):
        partner = self.env.user.partner_id

        # Get paid invoices
        purchases = self.sudo().search_read(
            domain=[
                ('move_id.payment_state', 'in', ['paid', 'in_payment']),
                ('move_id.partner_id', '=', partner.id),
                ('product_id', '!=', False),
            ],
            fields=['product_id'],
        )

        # Get free products
        purchases += self.env['sale.order.line'].sudo().search_read(
            domain=[('display_type', '=', False), ('order_id.partner_id', '=', partner.id), '|', ('price_subtotal', '=', 0.0), ('order_id.amount_total', '=', 0.0)],
            fields=['product_id'],
        )

        # I only want product_ids, but search_read insists in giving me a list of
        # (product_id: <id>, name: <product code> <template_name> <attributes>)
        return [line['product_id'][0] for line in purchases]

```

## File: models\ir_attachment.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class Attachment(models.Model):

    _inherit = ['ir.attachment']

    product_downloadable = fields.Boolean("Downloadable from product portal", default=False)

```

## File: models\product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _


class ProductTemplate(models.Model):
    _inherit = ['product.template']

    attachment_count = fields.Integer(compute='_compute_attachment_count', string="File")

    def _compute_attachment_count(self):
        attachment_data = self.env['ir.attachment'].read_group([('res_model', '=', self._name), ('res_id', 'in', self.ids), ('product_downloadable', '=', True)], ['res_id'], ['res_id'])
        mapped_data = dict([(data['res_id'], data['res_id_count']) for data in attachment_data])
        for product_template in self:
            product_template.attachment_count = mapped_data.get(product_template.id, 0)

    def action_open_attachments(self):
        self.ensure_one()
        return {
            'name': _('Digital Attachments'),
            'domain': [('res_model', '=', self._name), ('res_id', '=', self.id), ('product_downloadable', '=', True)],
            'res_model': 'ir.attachment',
            'type': 'ir.actions.act_window',
            'view_mode': 'kanban,form',
            'context': "{'default_res_model': '%s','default_res_id': %d, 'default_product_downloadable': True}" % (self._name, self.id),
            'help': """
                <p class="o_view_nocontent_smiling_face">%s</p>
                <p>%s</p>
                """ % (_("Add attachments for this digital product"),
                       _("The attached files are the ones that will be purchased and sent to the customer.")),
        }


class Product(models.Model):
    _inherit = 'product.product'

    attachment_count = fields.Integer(compute='_compute_attachment_count', string="File")

    def _compute_attachment_count(self):
        for product in self:
            product.attachment_count = self.env['ir.attachment'].search_count([
                '|',
                '&', '&', ('res_model', '=', 'product.template'), ('res_id', '=', product.product_tmpl_id.id), ('product_downloadable', '=', True),
                '&', '&', ('res_model', '=', 'product.product'), ('res_id', '=', product.id), ('product_downloadable', '=', True)])

    def action_open_attachments(self):
        self.ensure_one()
        return {
            'name': _('Digital Attachments'),
            'domain': [('product_downloadable', '=', True), '|',
                       '&', ('res_model', '=', 'product.template'), ('res_id', '=', self.product_tmpl_id.id),
                       '&', ('res_model', '=', self._name), ('res_id', '=', self.id)],
            'res_model': 'ir.attachment',
            'type': 'ir.actions.act_window',
            'view_mode': 'kanban,form',
            'context': "{'default_res_model': '%s','default_res_id': %d, 'default_product_downloadable': True}" % (self._name, self.id),
            'help': """
                <p class="o_view_nocontent_smiling_face">%s</p>
                <p>%s</p>
                """ % (_("Add attachments for this digital product"),
                       _("The attached files are the ones that will be purchased and sent to the customer.")),
        }

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import ir_attachment
from . import account_invoice
from . import product

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#CDC484"/><stop offset="100%" stop-color="#B5AA59"/></linearGradient><path id="d" d="M24.594 27.281l3.284 14.7 24.038-.939c1.441 0 1.441 1.834.088 1.998L28.361 44l.942 3H51.11c.942 0 .942 2 0 2H27.42l-5.652-24h-1.884v1c0 .667-.314 1-.942 1-.628 0-.942-.333-.942-1v-2c.062-.667.376-1 .942-1h3.768c.487 0 .8.333.942 1l.942 3.281zM49.43 55a2.497 2.497 0 0 1-2.494-2.5c0-1.38 1.116-2.5 2.494-2.5a2.497 2.497 0 0 1 2.494 2.5c0 1.38-1.117 2.5-2.494 2.5zm-19.956 0a2.497 2.497 0 0 1-2.494-2.5c0-1.38 1.117-2.5 2.494-2.5a2.497 2.497 0 0 1 2.495 2.5c0 1.38-1.117 2.5-2.495 2.5zm13.851-31.422l-.195-2.227c-.108-1.236-.928-1.158-1.95-1.069l-3.686.323c-1.023.089-1.844.155-1.736 1.391l.195 2.227-5.529.484 1.18 13.477c.107 1.235 1.014 2.155 2.037 2.065l14.744-1.29c1.023-.09 1.756-1.152 1.648-2.388l-1.18-13.477-5.528.484zm-6.448-1.632l5.427-.495.195 2.227-5.454.47-.168-2.202zm.317 13.81l-.712-8.145 8.666 2.89-7.954 5.255z"/><path id="e" d="M24.594 25.281l3.284 14.7 24.038-.939c1.441 0 1.441 1.834.088 1.998L28.361 42l.942 3H51.11c.942 0 .942 2 0 2H27.42l-5.652-24h-1.884v1c0 .667-.314 1-.942 1-.628 0-.942-.333-.942-1v-2c.062-.667.376-1 .942-1h3.768c.487 0 .8.333.942 1l.942 3.281zM49.43 53a2.497 2.497 0 0 1-2.494-2.5c0-1.38 1.116-2.5 2.494-2.5a2.497 2.497 0 0 1 2.494 2.5c0 1.38-1.117 2.5-2.494 2.5zm-19.956 0a2.497 2.497 0 0 1-2.494-2.5c0-1.38 1.117-2.5 2.494-2.5a2.497 2.497 0 0 1 2.495 2.5c0 1.38-1.117 2.5-2.495 2.5zm13.851-31.422l-.195-2.227c-.108-1.236-.928-1.158-1.95-1.069l-3.686.323c-1.023.089-1.844.155-1.736 1.391l.195 2.227-5.529.484 1.18 13.477c.107 1.235 1.014 2.155 2.037 2.065l14.744-1.29c1.023-.09 1.756-1.152 1.648-2.388l-1.18-13.477-5.528.484zm-6.448-1.632l5.427-.495.195 2.227-5.454.47-.168-2.202zm.317 13.81l-.712-8.145 8.666 2.89-7.954 5.255z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M36.154 69H4c-2 0-4-1-4-4V38.622l18.355-17.41h4.96l1.23 7.474 5.835-5.712 2.4-.255 3.374-3.621 6.194-.611.832 3.94 5.577-1.215.993 14.768-5.361 4.836 8.365-.134-4.902 4.372 3.676 1.768-1.935 2.04 1.266 3.581L36.154 69z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: views\website_sale_digital.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="sale_order_portal_content_inherit_website_sale_digital" name="Orders Downloads Followup" inherit_id="sale.sale_order_portal_content">
        <xpath expr="//section[@id='details']//td[@id='product_name']" position="inside">
            <t t-if="digital_attachments" t-set="attachments" t-value="digital_attachments.get(line.product_id.id)"/>
            <t t-if="attachments">
                <span class="dropdown">
                    <button class="btn btn-sm btn-secondary dropdown-toggle" type="button" id="dropdownMenu1" data-toggle="dropdown">
                        Downloads
                    </button>
                    <div class="dropdown-menu" role="menu" aria-labelledby="dropdownMenu1">
                        <t t-foreach="attachments" t-as="a">
                            <a role="menuitem" tabindex="-1" t-att-href="'/my/download?attachment_id=%i' % a['id']" class="dropdown-item"><t t-esc="a['name']"/></a>
                        </t>
                    </div>
                </span>
            </t>
        </xpath>
    </template>
</odoo>

```

## File: views\website_sale_digital_view.xml

```xml
<?xml version="1.0" ?>
<odoo>
    <record id="product_template_view_form_inherit_digital" model="ir.ui.view">
        <field name="name">product.template.view.form.inherit.digital</field>
        <field name="model">product.template</field>
        <field name="inherit_id" ref="product.product_template_only_form_view" />
        <field name="arch" type="xml">
            <div name="button_box" position="inside">
                <button class="oe_stat_button" name="action_open_attachments" type="object" icon="fa-file-text-o">
                    <field string="Digital Files" name="attachment_count" widget="statinfo" />
                </button>
            </div>
        </field>
    </record>

    <record id="product_product_view_form_inherit_digital" model="ir.ui.view">
        <field name="name">product.product.view.form.inherit.digital</field>
        <field name="model">product.product</field>
        <field name="inherit_id" ref="product.product_normal_form_view"/>
        <field name="arch" type="xml">
             <div name="button_box" position="inside">
                <button class="oe_stat_button" name="action_open_attachments" type="object" icon="fa-file-text-o">
                    <field string="Digital Files" name="attachment_count" widget="statinfo" />
                </button>
            </div>
        </field>
    </record>
</odoo>

```

