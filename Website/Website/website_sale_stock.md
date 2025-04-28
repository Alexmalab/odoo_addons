# Odoo Module: website_sale_stock

Category: Website/Website

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Product Availability',
    'category': 'Website/Website',
    'summary': 'Manage product inventory & availability',
    'description': """
Manage the inventory of your products and display their availability status in your eCommerce store.
In case of stockout, you can decide to block further sales or to keep selling.
A default behavior can be selected in the Website settings.
Then it can be made specific at the product level.
    """,
    'depends': [
        'website_sale',
        'sale_stock',
    ],
    'data': [
        'views/product_template_views.xml',
        'views/res_config_settings_views.xml',
        'views/website_sale_stock_templates.xml',
        'views/stock_picking_views.xml',
        'views/website_pages_views.xml',
        'data/template_email.xml',
        'data/ir_cron_data.xml',
    ],
    'demo': [
        'data/website_sale_stock_demo.xml',
    ],
    'auto_install': True,
    'assets': {
        'web.assets_frontend': [
            'website_sale_stock/static/src/js/**/*',
            'website_sale_stock/static/src/xml/**/*',
        ],
        'web.assets_tests': [
            'website_sale_stock/static/tests/tours/website_sale_stock_multilang.js',
            'website_sale_stock/static/tests/tours/website_sale_stock_stock_notification.js',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.addons.website_sale.controllers import main as website_sale_controller
from odoo.tools import email_re
from odoo import http, _
from odoo.http import request
from odoo.exceptions import ValidationError
from werkzeug.exceptions import BadRequest


class PaymentPortal(website_sale_controller.PaymentPortal):

    @http.route()
    def shop_payment_transaction(self, *args, **kwargs):
        """ Payment transaction override to double check cart quantities before
        placing the order
        """
        order = request.website.sale_get_order()
        values = []
        for line in order.order_line:
            if line.product_id.type == 'product' and not line.product_id.allow_out_of_stock_order:
                cart_qty, avl_qty = order._get_cart_and_free_qty(line=line)
                if cart_qty > avl_qty:
                    line._set_shop_warning_stock(cart_qty, max(avl_qty, 0))
                    values.append(line.shop_warning)
        if values:
            raise ValidationError(' '.join(values))
        return super().shop_payment_transaction(*args, **kwargs)


class WebsiteSale(website_sale_controller.WebsiteSale):
    @http.route(['/shop/add/stock_notification'], type="json", auth="public", website=True)
    def add_stock_email_notification(self, email, product_id):
        if not email_re.match(email):
            raise BadRequest(_("Invalid Email"))

        product = request.env['product.product'].browse(int(product_id))
        partners = request.env['res.partner'].sudo()._mail_find_partner_from_emails([email], force_create=True)
        partner = partners[0]

        if not product._has_stock_notification(partner):
            product.sudo().stock_notification_partner_ids += partner

        if request.website.is_public_user():
            request.session['product_with_stock_notification_enabled'] = request.session.get(
                'product_with_stock_notification_enabled',
                set()
            ) | {product_id}
            request.session['stock_notification_email'] = email

    def _prepare_product_values(self, product, category='', search='', **kwargs):
        values = super()._prepare_product_values(product, category, search, **kwargs)
        # We need the user mail to prefill the back of stock notification, so we put it in the value that will be sent
        values['user_email'] = request.env.user.email or request.session.get('stock_notification_email', '')
        return values

class CustomerPortal(website_sale_controller.CustomerPortal):
    def _sale_reorder_get_line_context(self):
        return {
            **super()._sale_reorder_get_line_context(),
            'website_sale_stock_get_quantity': True,
        }

```

## File: controllers\variant.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http
from odoo.addons.website_sale.controllers.variant import WebsiteSaleVariantController


class WebsiteSaleStockVariantController(WebsiteSaleVariantController):
    @http.route()
    def get_combination_info_website(self, product_template_id, product_id, combination, add_qty, **kw):
        kw['context'] = kw.get('context', {})
        kw['context'].update(website_sale_stock_get_quantity=True)
        return super(WebsiteSaleStockVariantController, self).get_combination_info_website(product_template_id, product_id, combination, add_qty, **kw)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import variant
from . import main

```

## File: data\ir_cron_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="ir_cron_send_availability_email" model="ir.cron">
        <field name="name">Product: send email regarding products availability</field>
        <field name="interval_number">1</field>
        <field name="interval_type">hours</field>
        <field name="numbercall">-1</field>
        <field name="doall" eval="False"/>
        <field name="model_id" ref="model_product_product"/>
        <field name="code">model._send_availability_email()</field>
        <field name="state">code</field>
    </record>
</odoo>

```

## File: data\template_email.xml

```xml
<?xml version="1.0" ?>
<odoo>
    <template id="availability_email_body">
        <div id="body">
            <p>Dear Customer,</p>
            <p>The following product is now available.</p>
            <div style="display: flex; justify-content: center; width: 100%;">
                <a t-attf-href="#{product.website_url}">
                    <img t-attf-src="/web/image/product.product/#{product.id}/image_1920"/>
                </a>
            </div>
            <div style="display: flex; flex-direction: row; align-items: center; justify-content: center; width: 100%;">
                <p t-esc="product.name"/>
                <p t-if="product.product_template_attribute_value_ids"
                   style="margin-left: 0.5em;">
                    (<t
                    t-out="', '.join(product.product_template_attribute_value_ids.mapped('name'))"
                    />)
                </p>
                <p style="margin-left: 0.5em; margin-right: 0.5em">-</p>
                <p t-esc="product.list_price" t-options="{'widget': 'monetary', 'display_currency': product.currency_id}"/>
            </div>
            <p t-esc="product.description_sale"/>
            <div style="display: flex; justify-content: center; width: 100%;">
                <a t-attf-href="#{product.website_url}" style="background-color: #875A7B; padding: 8px 16px 8px 16px; text-decoration: none; color: #fff; border-radius: 5px; font-size:13px;">
                    Order Now
                </a>
            </div>
            <p>Regards,</p>
        </div>
    </template>
</odoo>

```

## File: data\website_sale_stock_demo.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="stock.product_cable_management_box" model="product.product">
        <field name="is_published" eval="True"/>
    </record>
    <record id="stock.product_cable_management_box_product_template" model="product.template">
        <field name="public_categ_ids" eval="[(6,0,[ref('website_sale.public_category_boxes')])]"/>
    </record>
</odoo>

```

## File: models\product_product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields, _
from odoo.http import request


class ProductProduct(models.Model):
    _inherit = 'product.product'

    stock_notification_partner_ids = fields.Many2many('res.partner', relation='stock_notification_product_partner_rel', string='Back in stock Notifications')

    def _has_stock_notification(self, partner):
        self.ensure_one()
        return partner in self.stock_notification_partner_ids

    def _get_cart_qty(self, website=None):
        if not self.allow_out_of_stock_order:
            website = website or self.env['website'].get_current_website()
            # When the cron is run manually, request has no attribute website, and that would cause a crash
            # so we check for it
            cart = website and request and hasattr(request, 'website') and website.sale_get_order() or None
            if cart:
                return sum(
                    cart._get_common_product_lines(product=self).mapped('product_uom_qty')
                )
        return 0

    def _is_sold_out(self):
        combination_info = self.with_context(website_sale_stock_get_quantity=True).product_tmpl_id._get_combination_info(product_id=self.id)
        return combination_info['product_type'] == 'product' and combination_info['free_qty'] <= 0

    def _website_show_quick_add(self):
        return (self.allow_out_of_stock_order or not self._is_sold_out()) and super()._website_show_quick_add()

    def _send_availability_email(self):
        for product in self.search([('stock_notification_partner_ids', '!=', False)]):
            if product._is_sold_out():
                continue
            for partner in product.stock_notification_partner_ids:
                self_ctxt = self.with_context(lang=partner.lang)
                product_ctxt = product.with_context(lang=partner.lang)
                body_html = self_ctxt.env['ir.qweb']._render(
                    'website_sale_stock.availability_email_body', {'product': product_ctxt})
                msg = self_ctxt.env['mail.message'].sudo().new(dict(body=body_html, record_name=product_ctxt.name))
                full_mail = self_ctxt.env['mail.render.mixin']._render_encapsulate(
                    "mail.mail_notification_light",
                    body_html,
                    add_context=dict(message=msg, model_description=_("Product")),
                )
                context = {'lang': partner.lang}  # Use partner lang to translate mail subject below
                mail_values = {
                    "subject": _("The product '%(product_name)s' is now available", product_name=product_ctxt.name),
                    "email_from": (product.company_id.partner_id or self.env.user).email_formatted,
                    "email_to": partner.email_formatted,
                    "body_html": full_mail,
                }
                del context

                mail = self_ctxt.env['mail.mail'].sudo().create(mail_values)
                mail.send(raise_exception=False)
                product.stock_notification_partner_ids -= partner

```

## File: models\product_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models
from odoo.tools.translate import html_translate
from odoo.http import request


class ProductTemplate(models.Model):
    _inherit = 'product.template'

    allow_out_of_stock_order = fields.Boolean(string='Continue selling when out-of-stock', default=True)

    available_threshold = fields.Float(string='Show Threshold', default=5.0)
    show_availability = fields.Boolean(string='Show availability Qty', default=False)
    out_of_stock_message = fields.Html(string="Out-of-Stock Message", translate=html_translate)

    def _get_combination_info(self, combination=False, product_id=False, add_qty=1, pricelist=False, parent_combination=False, only_template=False):
        combination_info = super(ProductTemplate, self)._get_combination_info(
            combination=combination, product_id=product_id, add_qty=add_qty, pricelist=pricelist,
            parent_combination=parent_combination, only_template=only_template)

        if not self.env.context.get('website_sale_stock_get_quantity'):
            return combination_info

        if combination_info['product_id']:
            product = self.env['product.product'].sudo().browse(combination_info['product_id'])
            website = self.env['website'].get_current_website()
            free_qty = product.with_context(warehouse=website._get_warehouse_available()).free_qty
            has_stock_notification = product._has_stock_notification(self.env.user.partner_id) \
                                     or request \
                                     and product.id in request.session.get('product_with_stock_notification_enabled',
                                                                           set())
            stock_notification_email = request and request.session.get('stock_notification_email', '')
            combination_info.update({
                'free_qty': free_qty,
                'product_type': product.type,
                'product_template': self.id,
                'available_threshold': self.available_threshold,
                'cart_qty': product._get_cart_qty(website),
                'uom_name': product.uom_id.name,
                'uom_rounding': product.uom_id.rounding,
                'allow_out_of_stock_order': self.allow_out_of_stock_order,
                'show_availability': self.show_availability,
                'out_of_stock_message': self.out_of_stock_message,
                'has_stock_notification': has_stock_notification,
                'stock_notification_email': stock_notification_email,
            })
        else:
            product_template = self.sudo()
            combination_info.update({
                'free_qty': 0,
                'product_type': product_template.type,
                'allow_out_of_stock_order': product_template.allow_out_of_stock_order,
                'available_threshold': product_template.available_threshold,
                'product_template': product_template.id,
                'cart_qty': 0,
            })

        return combination_info

    def _is_sold_out(self):
        return self.product_variant_id._is_sold_out()

    def _website_show_quick_add(self):
        return (self.allow_out_of_stock_order or not self._is_sold_out()) and super()._website_show_quick_add()

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    allow_out_of_stock_order = fields.Boolean(
        string='Continue selling when out-of-stock',
        default=True)
    available_threshold = fields.Float(
        string='Show Threshold',
        default=5.0)
    show_availability = fields.Boolean(
        string='Show availability Qty',
        default=False)
    website_warehouse_id = fields.Many2one(
        'stock.warehouse',
        related='website_id.warehouse_id',
        domain="[('company_id', '=', website_company_id)]",
        readonly=False)

    def set_values(self):
        super(ResConfigSettings, self).set_values()
        IrDefault = self.env['ir.default'].sudo()

        IrDefault.set('product.template', 'allow_out_of_stock_order', self.allow_out_of_stock_order)
        IrDefault.set('product.template', 'available_threshold', self.available_threshold)
        IrDefault.set('product.template', 'show_availability', self.show_availability)

    @api.model
    def get_values(self):
        res = super(ResConfigSettings, self).get_values()
        IrDefault = self.env['ir.default'].sudo()
        allow_out_of_stock_order = IrDefault.get('product.template', 'allow_out_of_stock_order')

        res.update(
            allow_out_of_stock_order=allow_out_of_stock_order if allow_out_of_stock_order is not None else True,
            available_threshold=IrDefault.get('product.template', 'available_threshold') or 5.0,
            show_availability=IrDefault.get('product.template', 'show_availability') or False)
        return res

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, _


class SaleOrder(models.Model):
    _inherit = 'sale.order'

    def _get_warehouse_available(self):
        self.ensure_one()
        warehouse = self.website_id._get_warehouse_available()
        if not warehouse and self.user_id and self.company_id:
            warehouse = self.user_id.with_company(self.company_id.id)._get_default_warehouse_id()
        if not warehouse:
            warehouse = self.env.user._get_default_warehouse_id()
        return warehouse

    def _compute_warehouse_id(self):
        website_orders = self.filtered('website_id')
        super(SaleOrder, self - website_orders)._compute_warehouse_id()
        for order in website_orders:
            order.warehouse_id = order._get_warehouse_available()

    def _verify_updated_quantity(self, order_line, product_id, new_qty, **kwargs):
        self.ensure_one()
        product = self.env['product.product'].browse(product_id)
        if product.type == 'product' and not product.allow_out_of_stock_order:
            product_qty_in_cart, available_qty = self._get_cart_and_free_qty(
                line=order_line, product=product, **kwargs
            )

            old_qty = order_line.product_uom_qty if order_line else 0
            added_qty = new_qty - old_qty
            total_cart_qty = product_qty_in_cart + added_qty
            if available_qty < total_cart_qty:
                allowed_line_qty = available_qty - (product_qty_in_cart - old_qty)
                if allowed_line_qty > 0:
                    if order_line:
                        order_line._set_shop_warning_stock(total_cart_qty, available_qty)
                    else:
                        self._set_shop_warning_stock(total_cart_qty, available_qty)
                else:  # 0 or negative allowed_qty
                    # if existing line: it will be deleted
                    # if no existing line: no line will be created
                    self.shop_warning = _(
                        "Some products became unavailable and your cart has been updated. We're sorry for the inconvenience.")
                return allowed_line_qty, order_line.shop_warning or self.shop_warning
        return super()._verify_updated_quantity(order_line, product_id, new_qty, **kwargs)

    def _get_cart_and_free_qty(self, line=None, product=None, **kwargs):
        """ Get cart quantity and free quantity for given product or line's product.

        Note: self.ensure_one()

        :param SaleOrderLine line: The optional line
        :param ProductProduct product: The optional product
        """
        self.ensure_one()
        if not line and not product:
            return 0, 0
        cart_qty = sum(
            self._get_common_product_lines(line, product, **kwargs).mapped('product_uom_qty')
        )
        free_qty = (product or line.product_id).with_context(warehouse=self.warehouse_id.id).free_qty
        return cart_qty, free_qty

    def _get_common_product_lines(self, line=None, product=None, **kwargs):
        """ Get the lines with the same product or line's product

        :param SaleOrderLine line: The optional line
        :param ProductProduct product: The optional product
        """
        if not line and not product:
            return self.env['sale.order.line']
        product = product or line.product_id
        return self.order_line.filtered(lambda l: l.product_id == product)

    def _set_shop_warning_stock(self, desired_qty, new_qty):
        self.ensure_one()
        self.shop_warning = _(
            'You ask for %(desired_qty)s products but only %(new_qty)s is available',
            desired_qty=desired_qty, new_qty=new_qty
        )
        return self.shop_warning

    def _get_cache_key_for_line(self, line):
        return line.product_id

    def _get_context_for_line(self, line):
        return {
            'website_sale_stock_get_quantity': True,
        }

    def _filter_can_send_abandoned_cart_mail(self):
        """ Filter sale orders on their product availability. """
        self = super()._filter_can_send_abandoned_cart_mail()
        combination_info_cache = {}

        def _are_all_product_available_for_purchase(sale_order):
            for line in sale_order.order_line:
                product = line.product_id
                if product.type != 'product':
                    continue
                cache_key = self._get_cache_key_for_line(line)
                combination_info = combination_info_cache.get(cache_key)
                if not combination_info:
                    combination_info = product.with_context(**self._get_context_for_line(line), website_id=sale_order.website_id.id)._get_combination_info_variant(add_qty=line.product_uom_qty)
                    combination_info_cache[cache_key] = combination_info
                if not product.allow_out_of_stock_order and combination_info['free_qty'] == 0:
                    return False
            return True

        # If none of the products in the checkout are available for purchase (empty inventory, for example),
        # then the email won't be sent.
        return self.filtered(_are_all_product_available_for_purchase)

```

## File: models\sale_order_line.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, models


class SaleOrderLine(models.Model):
    _inherit = 'sale.order.line'

    def _set_shop_warning_stock(self, desired_qty, new_qty):
        self.ensure_one()
        self.shop_warning = _(
            'You ask for %(desired_qty)s %(product_name)s but only %(new_qty)s is available',
            desired_qty=desired_qty, product_name=self.product_id.name, new_qty=new_qty
        )
        return self.shop_warning

    def _get_max_available_qty(self):
        return self.product_id.free_qty - self.product_id._get_cart_qty()

```

## File: models\stock_picking.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class StockPicking(models.Model):
    _inherit = 'stock.picking'

    website_id = fields.Many2one('website', related='sale_id.website_id', string='Website',
                                 help='Website where this order has been placed, for eCommerce orders.',
                                 store=True, readonly=True)

```

## File: models\website.py

```python
# -*- coding: utf-8 -*-
from odoo import api, fields, models


class Website(models.Model):
    _inherit = 'website'

    warehouse_id = fields.Many2one('stock.warehouse', string='Warehouse')

    def _prepare_sale_order_values(self, partner_sudo):
        values = super()._prepare_sale_order_values(partner_sudo)

        warehouse_id = self._get_warehouse_available()
        if warehouse_id:
            values['warehouse_id'] = warehouse_id
        return values

    def _get_warehouse_available(self):
        return (
            self.warehouse_id.id or
            self.env['ir.default'].get('sale.order', 'warehouse_id', company_id=self.company_id.id) or
            self.env['ir.default'].get('sale.order', 'warehouse_id') or
            self.env['stock.warehouse'].sudo().search([('company_id', '=', self.company_id.id)], limit=1).id
        )

    # FIXME VFE check if still needed
    def sale_get_order(self, *args, **kwargs):
        so = super().sale_get_order(*args, **kwargs)
        return so.with_context(warehouse=so.warehouse_id.id) if so else so

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import website
from . import product_product
from . import product_template
from . import res_config_settings
from . import sale_order
from . import sale_order_line
from . import stock_picking

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#B06161"/><stop offset="45.785%" stop-color="#984E4E"/><stop offset="100%" stop-color="#7C3838"/></linearGradient><path id="d" d="M24.592 27.281l3.283 14.7L31 41.796V39h19v2.042h1.908c1.441 0 1.441 1.834.088 1.998L28.36 44l.942 3h21.801c.942 0 .942 2 0 2H27.417l-5.65-24h-1.884v1c0 .667-.313 1-.941 1-.628 0-.942-.333-.942-1v-2c.062-.667.376-1 .942-1h3.767c.486 0 .8.333.941 1l.942 3.281zM49.423 55a2.497 2.497 0 0 1-2.493-2.5c0-1.38 1.116-2.5 2.493-2.5a2.497 2.497 0 0 1 2.494 2.5c0 1.38-1.117 2.5-2.494 2.5zm-19.95 0a2.497 2.497 0 0 1-2.494-2.5c0-1.38 1.116-2.5 2.494-2.5a2.497 2.497 0 0 1 2.493 2.5c0 1.38-1.116 2.5-2.493 2.5zM31 35h19v3H31v-3zm0-5h19v4H31v-4zm0-5h19v4H31v-4zm1 1v2h17v-2H32z"/><path id="e" d="M24.592 25.281l3.283 14.7L31 39.796V37h19v2.042h1.908c1.441 0 1.441 1.834.088 1.998L28.36 42l.942 3h21.801c.942 0 .942 2 0 2H27.417l-5.65-24h-1.884v1c0 .667-.313 1-.941 1-.628 0-.942-.333-.942-1v-2c.062-.667.376-1 .942-1h3.767c.486 0 .8.333.941 1l.942 3.281zM49.423 53a2.497 2.497 0 0 1-2.493-2.5c0-1.38 1.116-2.5 2.493-2.5a2.497 2.497 0 0 1 2.494 2.5c0 1.38-1.117 2.5-2.494 2.5zm-19.95 0a2.497 2.497 0 0 1-2.494-2.5c0-1.38 1.116-2.5 2.494-2.5a2.497 2.497 0 0 1 2.493 2.5c0 1.38-1.116 2.5-2.493 2.5zM31 33h19v3H31v-3zm0-5h19v4H31v-4zm0-5h19v4H31v-4zm1 1v2h17v-2H32z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M35.869 69H4c-2 0-4-1-4-4V38.32l18.528-17.185h4.459l2.385 7.467L31 23h19v16l2.5 1.5-9.773 4.8H51V47l-.637 1.385 1.485 2.626-.388.927L35.869 69z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: static\src\js\variant_mixin.js

```javascript
odoo.define('website_sale_stock.VariantMixin', function (require) {
'use strict';

const {Markup} = require('web.utils');
const field_utils = require('web.field_utils');
var VariantMixin = require('sale.VariantMixin');
var publicWidget = require('web.public.widget');
var core = require('web.core');
var QWeb = core.qweb;

require('website_sale.website_sale');

/**
 * Addition to the variant_mixin._onChangeCombination
 *
 * This will prevent the user from selecting a quantity that is not available in the
 * stock for that product.
 *
 * It will also display various info/warning messages regarding the select product's stock.
 *
 * This behavior is only applied for the web shop (and not on the SO form)
 * and only for the main product.
 *
 * @param {MouseEvent} ev
 * @param {$.Element} $parent
 * @param {Array} combination
 */
VariantMixin._onChangeCombinationStock = function (ev, $parent, combination) {
    let product_id = 0;
    // needed for list view of variants
    if ($parent.find('input.product_id:checked').length) {
        product_id = $parent.find('input.product_id:checked').val();
    } else {
        product_id = $parent.find('.product_id').val();
    }
    const isMainProduct = combination.product_id &&
        ($parent.is('.js_main_product') || $parent.is('.main_product')) &&
        combination.product_id === parseInt(product_id);

    if (!this.isWebsite || !isMainProduct) {
        return;
    }

    const $addQtyInput = $parent.find('input[name="add_qty"]');
    let qty = $addQtyInput.val();
    let ctaWrapper = $parent[0].querySelector('#o_wsale_cta_wrapper');
    ctaWrapper.classList.replace('d-none', 'd-flex');
    ctaWrapper.classList.remove('out_of_stock');

    if (combination.product_type === 'product' && !combination.allow_out_of_stock_order) {
        combination.free_qty -= parseInt(combination.cart_qty);
        $addQtyInput.data('max', combination.free_qty || 1);
        if (combination.free_qty < 0) {
            combination.free_qty = 0;
        }
        if (qty > combination.free_qty) {
            qty = combination.free_qty || 1;
            $addQtyInput.val(qty);
        }
        if (combination.free_qty < 1) {
            ctaWrapper.classList.replace('d-flex', 'd-none');
            ctaWrapper.classList.add('out_of_stock');
        }
    }

    // needed xml-side for formatting of remaining qty
    combination.formatQuantity = (qty) => {
        if (Number.isInteger(qty)) {
            return qty;
        } else {
            const decimals = Math.max(
                0,
                Math.ceil(-Math.log10(combination.uom_rounding))
            );
            return field_utils.format.float(qty, {digits: [false, decimals]});
        }
    }

    $('.oe_website_sale')
        .find('.availability_message_' + combination.product_template)
        .remove();
    combination.has_out_of_stock_message = $(combination.out_of_stock_message).text() !== '';
    combination.out_of_stock_message = Markup(combination.out_of_stock_message);
    const $message = $(QWeb.render(
        'website_sale_stock.product_availability',
        combination
    ));
    $('div.availability_messages').html($message);
};

publicWidget.registry.WebsiteSale.include({
    /**
     * Adds the stock checking to the regular _onChangeCombination method
     * @override
     */
    _onChangeCombination: function () {
        this._super.apply(this, arguments);
        VariantMixin._onChangeCombinationStock.apply(this, arguments);
    },
    /**
     * Recomputes the combination after adding a product to the cart
     * @override
     */
    _onClickAdd(ev) {
        return this._super.apply(this, arguments).then(() => {
            if ($('div.availability_messages').length) {
                this._getCombinationInfo(ev);
            }
        });
    }
});

return VariantMixin;

});

```

## File: static\src\js\website_sale.js

```javascript
/** @odoo-module alias=website_sale_stock.website_sale**/

import { WebsiteSale } from 'website_sale.website_sale';
import { is_email } from 'web.utils';

WebsiteSale.include({
    events: Object.assign({}, WebsiteSale.prototype.events, {
        'click #product_stock_notification_message': '_onClickProductStockNotificationMessage',
        'click #product_stock_notification_form_submit_button': '_onClickSubmitProductStockNotificationForm',
    }),

    _onClickProductStockNotificationMessage: function (ev) {
        const partnerEmail = document.querySelector('#wsale_user_email').value;
        const emailInputEl = document.querySelector('#stock_notification_input');

        emailInputEl.value = partnerEmail;
        this._handleClickStockNotificationMessage(ev);
    },

    _onClickSubmitProductStockNotificationForm: function (ev) {
        const formEl = ev.currentTarget.closest('#stock_notification_form');
        const productId = parseInt(formEl.querySelector('input[name="product_id"]').value);
        this._handleClickSubmitStockNotificationForm(ev, productId);
    },


    _handleClickStockNotificationMessage(ev) {
        ev.currentTarget.classList.add('d-none');
        ev.currentTarget.parentElement.querySelector('#stock_notification_form').classList.remove('d-none');
    },

    _handleClickSubmitStockNotificationForm(ev, productId) {
        const stockNotificationEl = ev.currentTarget.closest('#stock_notification_div');
        const formEl = stockNotificationEl.querySelector('#stock_notification_form');
        const email = stockNotificationEl.querySelector('#stock_notification_input').value.trim();

        if (!is_email(email)) {
            return this._displayEmailIncorrectMessage(stockNotificationEl);
        }

        this._rpc({
            route: "/shop/add/stock_notification",
            params: {
                product_id: productId,
                email,
            },
        }).then((data) => {
            const message = stockNotificationEl.querySelector('#stock_notification_success_message');

            message.classList.remove('d-none');
            formEl.classList.add('d-none');
        }).catch((error) => {
            this._displayEmailIncorrectMessage(stockNotificationEl);
        });
    },

    _displayEmailIncorrectMessage(stockNotificationEl) {
        const incorrectIconEl = stockNotificationEl.querySelector('#stock_notification_input_incorrect');
        incorrectIconEl.classList.remove('d-none');
    }
});

export default WebsiteSale;

```

## File: static\src\js\website_sale_reorder.js

```javascript
/** @odoo-module **/

import { ReorderDialog } from "@website_sale/js/website_sale_reorder";
import { patch } from "@web/core/utils/patch";
import { sprintf } from "@web/core/utils/strings";

patch(ReorderDialog.prototype, "website_sale_stock_reorder", {
    /**
     * @override
     */
    async onWillStartHandler() {
        const res = await this._super(...arguments);
        for (const product of this.content.products) {
            this.stockCheckCombinationInfo(product);
        }
        return res;
    },

    /**
     * @override
     */
    async loadProductCombinationInfo(product) {
        await this._super(...arguments);
    },

    stockCheckCombinationInfo(product) {
        // Products that should have a max quantity available should be limited by default.
        if (product.combinationInfo.allow_out_of_stock_order || product.type !== "product") {
            return;
        }
        product.max_quantity_available = product.combinationInfo.free_qty;
        if (!product.max_quantity_available) {
            product.add_to_cart_allowed = false;
        }
        if (product.max_quantity_available < product.qty) {
            product.qty_warning = sprintf(
                this.env._t("You ask for %s Units but only %s are available."),
                product.qty.toFixed(1),
                product.max_quantity_available.toFixed(1)
            );
            product.qty = product.max_quantity_available;
            product.stock_warning = true;
        } else if (product.combinationInfo.cart_qty) {
            product.qty_warning = sprintf(
                this.env._t("You already have %s Units in your cart."),
                product.combinationInfo.cart_qty.toFixed(1)
            );
        }
    },

    /**
     * @override
     */
    getWarningForProduct(product) {
        if (product.hasOwnProperty("max_quantity_available") && !product.max_quantity_available) {
            return this.env._t("This product is out of stock.");
        }
        return this._super(...arguments);
    },

    /**
     * @override
     */
    changeProductQty(product, newQty) {
        if (product.max_quantity_available && newQty > product.max_quantity_available) {
            product.qty_warning = sprintf(
                this.env._t("You ask for %s Units but only %s are available."),
                newQty.toFixed(1),
                product.max_quantity_available.toFixed(1)
            );
            product.stock_warning = true;
            newQty = product.max_quantity_available;
        } else if (product.stock_warning) {
            product.qty_warning = false;
            product.stock_warning = false;
        }
        this._super(product, newQty);
    },
});

```

## File: static\src\js\tours\website_sale_stock_reorder_from_portal.js

```javascript
/** @odoo-module **/

import tour from 'web_tour.tour';
import wTourUtils from 'website.tour_utils';

tour.register('website_sale_stock_reorder_from_portal', {
        test: true,
        url: '/my/orders',
    },
    [
        {
            content: 'Select first order',
            trigger: '.o_portal_my_doc_table a:first',
        },
        wTourUtils.clickOnElement('Reorder Again', '.o_wsale_reorder_button'),
        {
            content: "Check that there is one out of stock product",
            trigger: "#o_wsale_reorder_body div.text-warning span:contains('This product is out of stock.')",
        },
        {
            content: "Check that there is one product that does not have enough stock",
            trigger: "#o_wsale_reorder_body div.text-warning:contains('You ask for 2.0 Units but only 1.0 are available.')",
        },
    ]
);


```

## File: static\src\xml\website_sale_stock_product_availability.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates>
    <t t-name="website_sale_stock.product_availability">
        <t t-if="product_type == 'product' and !prevent_zero_price_sale">
            <div t-if="free_qty lte 0 and !cart_qty" t-attf-class="availability_message_#{product_template} mb-1">
                <div id="out_of_stock_message">
                    <t t-if='has_out_of_stock_message' t-out='out_of_stock_message'/>
                    <t t-elif="!allow_out_of_stock_order">
                        <div class="text-danger fw-bold">
                            <i class="fa fa-times me-1"/>
                            Out of Stock
                        </div>
                    </t>
                </div>
                <div id="stock_notification_div" t-if="!allow_out_of_stock_order">
                    <div class="btn btn-link px-0" t-if="!has_stock_notification"
                         id="product_stock_notification_message">
                        <i class="fa fa-envelope-o me-1"/>
                        Get notified when back in stock
                    </div>
                    <div id="stock_notification_form" class="d-none">
                        <div class="input-group">
                            <input class="form-control"
                                   id="stock_notification_input" name="email"
                                   type="text" placeholder="youremail@gmail.com" t-value="stock_notification_email if stock_notification_email else None"/>
                            <input name="product_id" type="hidden" t-att-value="product_id"/>

                            <div id="product_stock_notification_form_submit_button" class="btn btn-secondary">
                                <i class="fa fa-paper-plane"/>
                            </div>
                            <div id="stock_notification_input_incorrect" class="btn d-none">
                                <i class="fa fa-times text-danger"/>
                                Invalid email
                            </div>
                        </div>

                    </div>
                    <div id="stock_notification_success_message"
                         t-att-class="has_stock_notification ? '' : 'd-none'">
                        <div class="text-muted">
                            <i class="fa fa-bell"/>
                            We'll notify you once the product is back in stock.
                        </div>
                    </div>
                </div>
            </div>
            <div id="threshold_message" t-elif="show_availability and free_qty lte available_threshold" t-attf-class="availability_message_#{product_template} text-warning fw-bold">
                Only <t t-esc="formatQuantity(free_qty)"/> <t t-esc="uom_name" /> left in stock.
            </div>

            <div id="already_in_cart_message" t-if="!allow_out_of_stock_order and show_availability and cart_qty" t-attf-class="availability_message_#{product_template} text-warning mt8">
                <t t-if='!free_qty'>
                    You already added all the available product in your cart.
                </t>
                <t t-else=''>
                    You already added <t t-esc="cart_qty" /> <t t-esc="uom_name" /> in your cart.
                </t>
            </div>
        </t>
    </t>
</templates>

```

## File: views\product_template_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="product_template_form_view_inherit_website_sale_stock" model="ir.ui.view">
        <field name="name">product.template.form.inherit.website.sale.stock</field>
        <field name="model">product.template</field>
        <field name="inherit_id" ref="website_sale.product_template_form_view" />
        <field name="arch" type="xml">
            <xpath expr="//field[@name='public_categ_ids']" position="after">
                <label for="allow_out_of_stock_order" attrs="{'invisible': [('type', 'in', ['service', 'consu'])]}" string="Out-of-Stock"/>
                <div attrs="{'invisible': [('type', 'in', ['service', 'consu'])]}">
                    <field name="allow_out_of_stock_order" class="oe_inline" /> Continue Selling
                </div>

                <label for="show_availability" attrs="{'invisible': [('type', 'in', ['service', 'consu'])]}" string="Show Available Qty"/>
                <div attrs="{'invisible': [('type', 'in', ['service', 'consu'])]}">
                    <field name="show_availability" class="oe_inline" />
                    <span attrs="{'invisible': [('show_availability', '=', False)]}">
                        <label for="available_threshold" string="only if below" class="o_light_label"/>
                        <field name="available_threshold" class="oe_inline col-1" widget="integer"/>
                        Units
                    </span>
                </div>
                <field name="out_of_stock_message" attrs="{'invisible': [('type', 'in', ['service', 'consu'])]}"/>
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
        <field name="name">res.config.settings.view.form.inherit.website.sale.stock</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="website_sale.res_config_settings_view_form" />
        <field name="arch" type="xml">
            <xpath expr="//div[@id='comparator_option_setting']" position="after">
                <div class="col-12 col-lg-6 o_setting_box" id="product_availability_setting">
                    <div class="o_setting_left_pane">
                    </div>
                    <div class="o_setting_right_pane" name="stock_inventory_availability">
                        <div class="o_form_label">
                            Inventory Defaults
                        </div>
                        <div class="text-muted">
                            How to display products having low quantities (on hand - reserved)
                        </div>
                        <div class="content-group">
                            <div class="row mt16"
                                id="website_warehouse_setting"
                                groups="stock.group_stock_multi_warehouses">
                                <field name="website_company_id" invisible="1"/>
                                <label for="website_warehouse_id" string="Warehouse" class="col-lg-3 o_light_label" />
                                <field name="website_warehouse_id"/>
                            </div>
                            <div class="content-group">
                                <div class="row mt16"
                                    id="allow_out_of_stock_order_setting"
                                    title="Default availability mode set on newly created storable products. This can be changed at the product level.">
                                    <div class="col-12">
                                        <label for="allow_out_of_stock_order" string="Out-of-Stock" class="p-0 col-4 o_light_label"/>
                                        <field name="allow_out_of_stock_order" class=" w-auto"/>
                                        <label for="allow_out_of_stock_order" class="o_light_label" string="Continue Selling"/>
                                    </div>
                                </div>
                            </div>
                            <div class="content-group">
                                <div class="row"
                                    id="show_availability_setting"
                                    title="Default visibility for custom messages.">
                                    <div class="col-12">
                                        <label for="show_availability" string="Show Available Qty" class="p-0 col-4 o_light_label mb-3 mt-2"/>
                                        <field name="show_availability" class="w-auto"/>
                                        <label for="available_threshold" string="only if below" class="o_light_label" attrs="{'invisible': [('show_availability', '=', False)]}"/>
                                        <field name="available_threshold" class="oe_inline col-1" widget="integer" attrs="{'invisible': [('show_availability', '=', False)]}"/>
                                        <span attrs="{'invisible': [('show_availability', '=', False)]}">Units</span>
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

## File: views\stock_picking_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_picking_form_inherit_website_sale_stock" model="ir.ui.view">
        <field name="name">stock.picking.form.inherit.website.sale.stock</field>
        <field name="model">stock.picking</field>
        <field name="inherit_id" ref="stock.view_picking_form"/>
        <field name="arch" type="xml">
            <xpath expr="//page[@name='extra']/group/group/field[@name='company_id']" position="before">
                <field name="website_id" groups="website.group_multi_website" invisible="1"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\website_pages_views.xml

```xml
<?xml version="1.0"?>
<odoo>

<record id="product_pages_tree_view" model="ir.ui.view">
    <field name="name">Product Pages Tree (stock inherit)</field>
    <field name="model">product.template</field>
    <field name="inherit_id" ref="website_sale.product_pages_tree_view"/>
    <field name="arch" type="xml">
        <field name="responsible_id" position="attributes">
            <attribute name="optional">hide</attribute>
        </field>
        <field name="virtual_available" position="attributes">
            <attribute name="optional">hide</attribute>
        </field>
    </field>
</record>

</odoo>

```

## File: views\website_sale_stock_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Shopping Cart Lines -->
    <template id="website_sale_stock_cart_lines" inherit_id="website_sale.cart_lines" name="Shopping Cart Lines">
        <xpath expr="//input[@type='text'][hasclass('quantity')]" position="attributes">
          <attribute name='t-att-data-max'>(line.product_uom_qty + line._get_max_available_qty()) if line.product_id.type == 'product' and not line.product_id.allow_out_of_stock_order else None</attribute>
        </xpath>
        <xpath expr="//div[hasclass('css_quantity')]" position="after">
            <div class='availability_messages'/>
        </xpath>
    </template>

  <template id="website_sale_stock_product" inherit_id="website_sale.product" priority="4">
    <xpath expr="//div[@id='o_wsale_cta_wrapper']" position="after">
      <div class="availability_messages o_not_editable"/>
    </xpath>
      <xpath expr="//div[@id='product_details']" position="inside">
          <input id="wsale_user_email" type="hidden" t-att-value="user_email" t-nocache="user_email changes between users"/>
      </xpath>
  </template>

</odoo>

```

