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
        'stock_delivery',
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
            ('before', 'website_sale/static/src/js/website_sale.js', 'website_sale_stock/static/src/js/variant_mixin.js'),
            'website_sale_stock/static/src/js/combo_configurator_dialog/*',
            'website_sale_stock/static/src/js/models/*',
            'website_sale_stock/static/src/js/product/*',
            'website_sale_stock/static/src/js/product_card/*',
            'website_sale_stock/static/src/js/product_configurator_dialog/*',
            'website_sale_stock/static/src/js/website_sale.js',
            'website_sale_stock/static/src/js/website_sale_reorder.js',
            'website_sale_stock/static/src/xml/**/*',
        ],
        'web.assets_tests': [
            'website_sale_stock/static/tests/tours/*',
            'website_sale_stock/static/src/js/tours/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from werkzeug.exceptions import BadRequest

from odoo import _
from odoo.http import Controller, request, route
from odoo.tools.mail import email_re


class WebsiteSaleStock(Controller):

    @route('/shop/add/stock_notification', type='json', auth='public', website=True)
    def add_stock_email_notification(self, email, product_id):
        if not email_re.match(email):
            raise BadRequest(_("Invalid Email"))

        product = request.env['product.product'].browse(int(product_id))
        partners = request.env['res.partner'].sudo()._mail_find_partner_from_emails([email], force_create=True)
        partner = partners[0]

        if not product._has_stock_notification(partner):
            product.sudo().stock_notification_partner_ids += partner

        if request.website.is_public_user():
            request.session['product_with_stock_notification_enabled'] = list(
                set(request.session.get('product_with_stock_notification_enabled', []))
                | {product_id}
            )
            request.session['stock_notification_email'] = email

```

## File: controllers\reorder.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.addons.website_sale.controllers import reorder
from odoo.http import request, route


class CustomerPortal(reorder.CustomerPortal):

    def _sale_reorder_get_line_context(self):
        return {
            **super()._sale_reorder_get_line_context(),
            'website_sale_stock_get_quantity': True,
        }

    @route()
    def my_orders_reorder_modal_content(self, order_id, access_token):
        result = super().my_orders_reorder_modal_content(order_id, access_token)
        for product in result['products']:
            product['is_storable'] = request.env['product.product'].browse(product['product_id']).is_storable
        return result

```

## File: controllers\variant.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.http import request, route

from odoo.addons.website_sale.controllers.variant import WebsiteSaleVariantController


class WebsiteSaleStockVariantController(WebsiteSaleVariantController):

    @route()
    def get_combination_info_website(self, *args, **kwargs):
        request.update_context(website_sale_stock_get_quantity=True)
        res = super().get_combination_info_website(*args, **kwargs)
        res['is_storable'] = request.env['product.product'].browse(res['product_id']).is_storable
        return res

```

## File: controllers\website_sale.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.http import request

from odoo.addons.website_sale.controllers import main


class WebsiteSale(main.WebsiteSale):

    def _prepare_product_values(self, product, category='', search='', **kwargs):
        values = super()._prepare_product_values(product, category, search, **kwargs)
        # We need the user mail to prefill the back of stock notification, so we put it in the value that will be sent
        values['user_email'] = request.env.user.email or request.session.get('stock_notification_email', '')
        return values

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main
from . import reorder
from . import variant
from . import website_sale

```

## File: data\ir_cron_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="ir_cron_send_availability_email" model="ir.cron">
        <field name="name">Product: send email regarding products availability</field>
        <field name="interval_number">1</field>
        <field name="interval_type">hours</field>
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

## File: models\product_combo.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class ProductCombo(models.Model):
    _inherit = 'product.combo'

    def _get_max_quantity(self, website, **kwargs):
        """ The max quantity of a combo is the max quantity of its combo item with the highest max
        quantity. If one of the combo items has no max quantity, then the combo also has no max
        quantity.

        Note: self.ensure_one()

        :param website website: The website for which to compute the max quantity.
        :return: The max quantity of the combo.
        :rtype: float | None
        """
        self.ensure_one()
        max_quantities = [
            item.product_id._get_max_quantity(website, **kwargs) for item in self.combo_item_ids
        ]
        return max(max_quantities) if (None not in max_quantities) else None

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
                return sum(cart._get_common_product_lines(product=self).mapped('product_uom_qty'))
        return 0

    def _get_max_quantity(self, website, **kwargs):
        """ The max quantity of a product is the difference between the quantity that's free to use
        and the quantity that's already been added to the cart.

        Note: self.ensure_one()

        :param website website: The website for which to compute the max quantity.
        :return: The max quantity of the product.
        :rtype: float | None
        """
        self.ensure_one()
        if self.is_storable and not self.allow_out_of_stock_order:
            free_qty = website._get_product_available_qty(self.sudo(), **kwargs)
            cart_qty = self._get_cart_qty(website)
            return free_qty - cart_qty
        return None

    def _is_sold_out(self):
        self.ensure_one()
        if not self.is_storable:
            return False
        free_qty = self.env['website'].get_current_website()._get_product_available_qty(self.sudo())
        return free_qty <= 0

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
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo.http import request
from odoo.tools.translate import html_translate

from odoo.addons.website.models import ir_http


class ProductTemplate(models.Model):
    _inherit = 'product.template'

    allow_out_of_stock_order = fields.Boolean(string='Continue selling when out-of-stock', default=True)

    available_threshold = fields.Float(string='Show Threshold', default=5.0)
    show_availability = fields.Boolean(string='Show availability Qty', default=False)
    out_of_stock_message = fields.Html(string="Out-of-Stock Message", translate=html_translate)

    def _is_sold_out(self):
        return self.is_storable and self.product_variant_id._is_sold_out()

    def _website_show_quick_add(self):
        return (self.allow_out_of_stock_order or not self._is_sold_out()) and super()._website_show_quick_add()

    def _get_additionnal_combination_info(self, product_or_template, quantity, date, website):
        res = super()._get_additionnal_combination_info(product_or_template, quantity, date, website)

        product_or_template = product_or_template.sudo()
        res.update({
            'product_type': product_or_template.type,
            'allow_out_of_stock_order': product_or_template.allow_out_of_stock_order,
            'available_threshold': product_or_template.available_threshold,
        })
        if product_or_template.is_product_variant:
            product = product_or_template
            free_qty = website._get_product_available_qty(product)
            has_stock_notification = (
                product._has_stock_notification(self.env.user.partner_id)
                or request and product.id in request.session.get(
                    'product_with_stock_notification_enabled', set())
            )
            stock_notification_email = request and request.session.get('stock_notification_email', '')
            res.update({
                'free_qty': free_qty,
                'cart_qty': product._get_cart_qty(website),
                'uom_name': product.uom_id.name,
                'uom_rounding': product.uom_id.rounding,
                'show_availability': product_or_template.show_availability,
                'out_of_stock_message': product_or_template.out_of_stock_message,
                'has_stock_notification': has_stock_notification,
                'stock_notification_email': stock_notification_email,
            })
        else:
            res.update({
                'free_qty': 0,
                'cart_qty': 0,
            })
        if product_or_template.type == 'combo':
            # The max quantity of a combo product is the max quantity of its combo with the lowest
            # max quantity. If none of the combos has a max quantity, then the combo product also
            # has no max quantity.
            max_quantities = [
                max_quantity for combo in product_or_template.combo_ids.sudo()
                if (max_quantity := combo._get_max_quantity(website)) is not None
            ]
            if max_quantities:
                res['max_combo_quantity'] = min(max_quantities)

        return res

    @api.model
    def _get_additional_configurator_data(
        self, product_or_template, date, currency, pricelist, **kwargs
    ):
        """ Override of `website_sale` to append stock data.

        :param product.product|product.template product_or_template: The product for which to get
            additional data.
        :param datetime date: The date to use to compute prices.
        :param res.currency currency: The currency to use to compute prices.
        :param product.pricelist pricelist: The pricelist to use to compute prices.
        :param dict kwargs: Locally unused data passed to `super` and `_get_max_quantity`.
        :rtype: dict
        :return: A dict containing additional data about the specified product.
        """
        data = super()._get_additional_configurator_data(
            product_or_template, date, currency, pricelist, **kwargs
        )

        if (website := ir_http.get_request_website()) and product_or_template.is_product_variant:
            max_quantity = product_or_template._get_max_quantity(website, **kwargs)
            if max_quantity is not None:
                data['free_qty'] = max_quantity
        return data

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
        IrDefaultGet = self.env['ir.default'].sudo()._get
        allow_out_of_stock_order = IrDefaultGet('product.template', 'allow_out_of_stock_order')

        res.update(
            allow_out_of_stock_order=allow_out_of_stock_order if allow_out_of_stock_order is not None else True,
            available_threshold=IrDefaultGet('product.template', 'available_threshold') or 5.0,
            show_availability=IrDefaultGet('product.template', 'show_availability') or False)
        return res

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, models
from odoo.exceptions import ValidationError


class SaleOrder(models.Model):
    _inherit = 'sale.order'

    # NB: dropped in 18.1
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
            if order.website_id.warehouse_id:
                order.warehouse_id = order.website_id.warehouse_id
            else:
                super(SaleOrder, order)._compute_warehouse_id()
            if not order.warehouse_id:
                order.warehouse_id = self.env.user._get_default_warehouse_id()

    def _verify_updated_quantity(self, order_line, product_id, new_qty, **kwargs):
        self.ensure_one()
        product = self.env['product.product'].browse(product_id)
        if product.is_storable and not product.allow_out_of_stock_order:
            product_qty_in_cart, available_qty = self._get_cart_and_free_qty(
                product, line=order_line
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
                    returned_warning = order_line.shop_warning or self.shop_warning
                else:  # 0 or negative allowed_qty
                    # if existing line: it will be deleted
                    # if no existing line: no line will be created
                    if order_line:
                        self.shop_warning = _(
                            "Some products became unavailable and your cart has been updated. We're"
                            " sorry for the inconvenience."
                        )
                        returned_warning = self.shop_warning
                    else:
                        returned_warning = _(
                            "The item has not been added to your cart since it is not available."
                        )
                return allowed_line_qty, returned_warning
        return super()._verify_updated_quantity(order_line, product_id, new_qty, **kwargs)

    def _get_cart_and_free_qty(self, product, line=None):
        """ Get cart quantity and free quantity for given product or line's product.

        Note: self.ensure_one()

        :param ProductProduct product: The product
        :param SaleOrderLine line: The optional line
        """
        self.ensure_one()
        if not line and not product:
            return 0, 0
        cart_qty = sum(self._get_common_product_lines(line, product).mapped('product_uom_qty'))
        free_qty = (product or line.product_id).with_context(
            warehouse_id=self.website_id.warehouse_id.id
        ).free_qty

        return cart_qty, free_qty

    def _get_common_product_lines(self, line=None, product=None):
        """ Get the lines with the same product or line's product

        :param SaleOrderLine line: The optional line
        :param ProductProduct product: The optional product
        """
        if not line and not product:
            return self.env['sale.order.line']
        product = product or line.product_id
        return self.order_line.filtered(lambda l: l.product_id == product)

    def _check_cart_is_ready_to_be_paid(self):
        values = []
        for line in self.order_line:
            if line.product_id.is_storable and not line.product_id.allow_out_of_stock_order:
                cart_qty, avl_qty = self._get_cart_and_free_qty(line.product_id, line=line)
                if cart_qty > avl_qty:
                    line._set_shop_warning_stock(cart_qty, max(avl_qty, 0))
                    values.append(line.shop_warning)
        if values:
            raise ValidationError(' '.join(values))
        return super()._check_cart_is_ready_to_be_paid()

    def _set_shop_warning_stock(self, desired_qty, new_qty):
        self.ensure_one()
        self.shop_warning = _(
            'You ask for %(desired_qty)s products but only %(new_qty)s is available',
            desired_qty=desired_qty, new_qty=new_qty
        )
        return self.shop_warning

    def _filter_can_send_abandoned_cart_mail(self):
        """ Filter sale orders on their product availability. """
        return super()._filter_can_send_abandoned_cart_mail().filtered(
            lambda so: so._all_product_available()
        )

    def _all_product_available(self):
        self.ensure_one()
        for line in self.with_context(website_sale_stock_get_quantity=True).order_line:
            product = line.product_id
            if not product.is_storable or product.allow_out_of_stock_order:
                continue
            free_qty = self.website_id._get_product_available_qty(product)
            if free_qty == 0:
                return False
        return True

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

    def _get_max_line_qty(self):
        max_quantity = self._get_max_available_qty()
        return self.product_uom_qty + max_quantity if (max_quantity is not None) else None

    def _get_max_available_qty(self):
        """ The max quantity of a combo product is the max quantity of its selected combo item with
        the lowest max quantity. If none of the combo items has a max quantity, then the combo
        product also has no max quantity.
        """
        website = self.order_id.website_id
        max_quantities = [
            max_quantity for product in self._get_lines_with_price().product_id
            if (max_quantity := product._get_max_quantity(website)) is not None
        ]
        return min(max_quantities, default=None)

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
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class Website(models.Model):
    _inherit = 'website'

    warehouse_id = fields.Many2one('stock.warehouse', string='Warehouse')

    # NB: unused and dropped in 18.1
    def _get_warehouse_available(self):
        return (
            self.warehouse_id.id or
            self.env['ir.default'].sudo()._get('sale.order', 'warehouse_id', company_id=self.company_id.id) or
            self.env['ir.default'].sudo()._get('sale.order', 'warehouse_id') or
            self.env['stock.warehouse'].sudo().search([('company_id', '=', self.company_id.id)], limit=1).id
        )

    def _get_product_available_qty(self, product, **kwargs):
        return product.with_context(warehouse_id=self.warehouse_id.id).free_qty

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import website
from . import product_combo
from . import product_product
from . import product_template
from . import res_config_settings
from . import sale_order
from . import sale_order_line
from . import stock_picking

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path fill-rule="evenodd" clip-rule="evenodd" d="M15.724 6.397C16.377 4.94 17.852 4 19.481 4h11.037c1.63 0 3.104.94 3.757 2.397L37.236 13H41.9c2.46 0 4.367 2.099 4.07 4.481l-3.106 25C42.613 44.49 40.866 46 38.793 46H11.207c-2.074 0-3.82-1.51-4.07-3.519l-3.107-25C3.734 15.1 5.64 13 8.1 13h4.663l2.961-6.603ZM32.917 13H17.082c0-.56.123-1.134.39-1.691l.956-2C19.102 7.9 20.551 7 22.144 7h5.711c1.593 0 3.042.9 3.716 2.308l.957 2c.266.558.39 1.132.39 1.692Z" fill="#088BF5"/><path fill-rule="evenodd" clip-rule="evenodd" d="M8.514 45.016a3.963 3.963 0 0 1-1.377-2.535l-3.107-25C3.734 15.1 5.64 13 8.1 13h4.663l2.961-6.603C16.377 4.94 17.852 4 19.481 4h11.037c1.63 0 3.104.94 3.757 2.397l2.59 5.777C35.5 28.256 23.848 41.405 8.515 45.016ZM17.082 13h15.835c0-.56-.123-1.134-.39-1.691l-.956-2C30.897 7.9 29.448 7 27.855 7h-5.711c-1.593 0-3.042.9-3.716 2.308l-.956 2a3.904 3.904 0 0 0-.39 1.692Z" fill="#2EBCFA"/><path d="M17.142 20.864a4 4 0 0 1 4.899-2.828l11.59 3.106a4 4 0 0 1 2.83 4.899l-3.107 11.59a4 4 0 0 1-4.899 2.83l-11.59-3.107a4 4 0 0 1-2.83-4.899l3.107-11.59Z" fill="#fff"/></svg>

```

## File: static\src\js\variant_mixin.js

```javascript
/** @odoo-module **/

import VariantMixin from "@website_sale/js/sale_variant_mixin";
import { renderToFragment } from "@web/core/utils/render";
import { formatFloat } from "@web/core/utils/numbers";


import { markup } from "@odoo/owl";

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
        $parent.is('.js_main_product') &&
        combination.product_id === parseInt(product_id);

    if (!this.isWebsite || !isMainProduct) {
        return;
    }

    const $addQtyInput = $parent.find('input[name="add_qty"]');
    let qty = $addQtyInput.val();
    let ctaWrapper = $parent[0].querySelector('#o_wsale_cta_wrapper');
    ctaWrapper.classList.replace('d-none', 'd-flex');
    ctaWrapper.classList.remove('out_of_stock');

    if (combination.is_storable && !combination.allow_out_of_stock_order) {
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

    combination.has_max_combo_quantity = 'max_combo_quantity' in combination
    if (combination.product_type === 'combo' && combination.has_max_combo_quantity) {
        $addQtyInput.data('max', combination.max_combo_quantity || 1);
        if (combination.max_combo_quantity < 1) {
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
            return formatFloat(qty, {digits: [false, decimals]});
        }
    }

    $('.oe_website_sale')
        .find('.availability_message_' + combination.product_template)
        .remove();
    combination.has_out_of_stock_message = $(combination.out_of_stock_message).text() !== '';
    combination.out_of_stock_message = markup(combination.out_of_stock_message);
    $('div.availability_messages').append(renderToFragment(
        'website_sale_stock.product_availability',
        combination
    ));
};

export default VariantMixin;

```

## File: static\src\js\website_sale.js

```javascript
/** @odoo-module **/

import { WebsiteSale } from '@website_sale/js/website_sale';
import { rpc } from "@web/core/network/rpc";
import { isEmail } from '@web/core/utils/strings';
import VariantMixin from "@website_sale/js/sale_variant_mixin";

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

        if (!isEmail(email)) {
            return this._displayEmailIncorrectMessage(stockNotificationEl);
        }

        rpc("/shop/add/stock_notification", {
            product_id: productId,
            email,
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
    },

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

export default WebsiteSale;

```

## File: static\src\js\website_sale_reorder.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { ReorderDialog } from "@website_sale/js/website_sale_reorder";
import { patch } from "@web/core/utils/patch";

patch(ReorderDialog.prototype, {
    /**
     * @override
     */
    async onWillStartHandler() {
        const res = await super.onWillStartHandler(...arguments);
        for (const product of this.content.products) {
            this.stockCheckCombinationInfo(product);
        }
        return res;
    },

    /**
     * @override
     */
    async loadProductCombinationInfo(product) {
        await super.loadProductCombinationInfo(...arguments);
    },

    stockCheckCombinationInfo(product) {
        // Products that should have a max quantity available should be limited by default.
        if (product.combinationInfo.allow_out_of_stock_order || ! product.is_storable) {
            return;
        }
        product.max_quantity_available = product.combinationInfo.free_qty;
        if (!product.max_quantity_available) {
            product.add_to_cart_allowed = false;
        }
        if (product.max_quantity_available < product.qty) {
            product.qty_warning = _t(
                "You ask for %(quantity1)s Units but only %(quantity2)s are available.",
                {
                    quantity1: product.qty.toFixed(1),
                    quantity2: product.max_quantity_available.toFixed(1),
                }
            );
            product.qty = product.max_quantity_available;
            product.stock_warning = true;
        } else if (product.combinationInfo.cart_qty) {
            product.qty_warning = _t(
                "You already have %s Units in your cart.",
                product.combinationInfo.cart_qty.toFixed(1)
            );
        }
    },

    /**
     * @override
     */
    getWarningForProduct(product) {
        if (product.hasOwnProperty("max_quantity_available") && !product.max_quantity_available) {
            return _t("This product is out of stock.");
        }
        return super.getWarningForProduct(...arguments);
    },

    /**
     * @override
     */
    changeProductQty(product, newQty) {
        if (product.max_quantity_available && newQty > product.max_quantity_available) {
            product.qty_warning = _t(
                "You ask for %(quantity1)s Units but only %(quantity2)s are available.",
                {
                    quantity1: newQty.toFixed(1),
                    quantity2: product.max_quantity_available.toFixed(1),
                }
            );
            product.stock_warning = true;
            newQty = product.max_quantity_available;
        } else if (product.stock_warning) {
            product.qty_warning = false;
            product.stock_warning = false;
        }
        super.changeProductQty(product, newQty);
    },
});

```

## File: static\src\js\combo_configurator_dialog\combo_configurator_dialog.js

```javascript
import { patch } from '@web/core/utils/patch';
import {
    ComboConfiguratorDialog
} from '@sale/js/combo_configurator_dialog/combo_configurator_dialog';

patch(ComboConfiguratorDialog.prototype, {
    async selectComboItem(comboId, comboItem) {
        if (!comboItem.product.isQuantityAllowed(this.state.quantity)) {
            return;
        }
        super.selectComboItem(...arguments);
    },

    async setQuantity(quantity) {
        if (!this.isComboQuantityAllowed(quantity)) {
            quantity = Math.min(
                ...this._selectedComboItems
                    .map(comboItem => comboItem.product.free_qty)
                    .filter(freeQty => freeQty !== undefined)
            );
        }
        return super.setQuantity(quantity);
    },

    /**
     * Check whether the provided combo quantity can be added to the cart.
     *
     * @param {Number} quantity The quantity to check.
     * @return {Boolean} Whether the combo quantity can be added to the cart.
     */
    isComboQuantityAllowed(quantity) {
        return this._selectedComboItems.every(
            comboItem => comboItem.product.isQuantityAllowed(quantity)
        );
    },
});

```

## File: static\src\js\combo_configurator_dialog\combo_configurator_dialog.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">
    <t t-inherit="sale.ComboConfiguratorDialog" t-inherit-mode="extension">
        <ProductCard position="attributes">
            <attribute name="quantity">state.quantity</attribute>
        </ProductCard>
        <QuantityButtons position="attributes">
            <attribute name="isPlusButtonDisabled">
                !isComboQuantityAllowed(state.quantity + 1)
            </attribute>
        </QuantityButtons>
        <button name="website_sale_combo_configurator_continue_button" position="attributes">
            <attribute
                name="t-att-disabled"
                add="!isComboQuantityAllowed(state.quantity)"
                separator=" || "
            />
        </button>
        <button name="website_sale_combo_configurator_checkout_button" position="attributes">
            <attribute
                name="t-att-disabled"
                add="!isComboQuantityAllowed(state.quantity)"
                separator=" || "
            />
        </button>
    </t>
</templates>

```

## File: static\src\js\models\product_product.js

```javascript
import { patch } from '@web/core/utils/patch';
import { ProductProduct } from '@sale/js/models/product_product';

patch(ProductProduct.prototype, {
    /**
     * @param {number} free_qty
     * @param args Super's parameter list.
     */
    setup({free_qty, ...args}) {
        super.setup(args);
        this.free_qty = free_qty;
    },

    /**
     * Check whether the provided quantity can be added to the cart.
     *
     * @param {Number} quantity The quantity to check.
     * @return {Boolean} Whether the product quantity can be added to the cart.
     */
    isQuantityAllowed(quantity) {
        return this.free_qty === undefined || this.free_qty >= quantity;
    },
});

```

## File: static\src\js\product\product.js

```javascript
/** @odoo-module **/

import { patch } from '@web/core/utils/patch';
import { Product } from '@sale/js/product/product';

patch(Product, {
    props: {
        ...Product.props,
        free_qty: { type: Number, optional: true },
    },
});

patch(Product.prototype, {
    /**
     * Check whether this product is out of stock.
     *
     * @return {Boolean} - Whether this product is out of stock.
     */
    isOutOfStock() {
        return !this.env.isQuantityAllowed(this.props, 1);
    },
});

```

## File: static\src\js\product\product.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">
    <t t-inherit="sale.Product" t-inherit-mode="extension">
        <QuantityButtons position="attributes">
            <attribute name="t-if" add="!isOutOfStock()" separator=" &amp;&amp; "/>
            <attribute name="isPlusButtonDisabled">
                !env.isQuantityAllowed(props, props.quantity + 1)
            </attribute>
        </QuantityButtons>
        <QuantityButtons position="after">
            <t t-call="website_sale_stock.out_of_stock"/>
        </QuantityButtons>
        <button name="sale_product_configurator_add_button" position="attributes">
            <attribute name="t-if" add="!isOutOfStock()" separator=" &amp;&amp; "/>
        </button>
        <button name="sale_product_configurator_add_button" position="after">
            <t t-call="website_sale_stock.out_of_stock"/>
        </button>
    </t>

    <t t-name="website_sale_stock.out_of_stock">
        <div
            t-if="isOutOfStock()"
            class="text-danger fw-bold"
        >
            <i class="fa fa-times me-1"/>
            Out of stock
        </div>
    </t>
</templates>

```

## File: static\src\js\product_card\product_card.js

```javascript
import { _t } from '@web/core/l10n/translation';
import { patch } from '@web/core/utils/patch';
import { ProductCard } from '@sale/js/product_card/product_card';

patch(ProductCard, {
    props: {
        ...ProductCard.props,
        quantity: { type: Number, optional: true },
    },
});

patch(ProductCard.prototype, {
    setup() {
        super.setup(...arguments);
        this.allQuantitySelectedTooltip = _t("All available quantity selected");
    },
});

```

## File: static\src\js\product_card\product_card.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">
    <t t-inherit="sale.ProductCard" t-inherit-mode="extension">
        <article position="attributes">
            <attribute
                name="t-attf-class"
                remove="cursor-pointer"
                add="{{
                    props.product.isQuantityAllowed(props.quantity)
                    ? 'cursor-pointer' : 'unselectable-card'
                }}"
                separator=" "
            />
        </article>
        <article position="inside">
            <div
                t-if="!props.product.isQuantityAllowed(props.quantity)"
                class="overlay-container justify-content-center text-center text-danger fw-bold p-2"
            >
                <i class="fa fa-times fa-2x mb-2"/>
                <span>Requested quantity not available</span>
            </div>
            <div
                t-elif="
                    props.isSelected &amp;&amp; !props.product.isQuantityAllowed(props.quantity + 1)
                "
                class="overlay-container text-end text-warning fw-bold p-2"
            >
                <i
                    class="fa fa-warning fa-2x"
                    data-toggle="tooltip"
                    data-trigger="click hover focus"
                    t-att-title="allQuantitySelectedTooltip"
                />
            </div>
        </article>
    </t>
</templates>

```

## File: static\src\js\product_configurator_dialog\product_configurator_dialog.js

```javascript
/** @odoo-module **/

import { patch } from '@web/core/utils/patch';
import { useSubEnv } from '@odoo/owl';
import {
    ProductConfiguratorDialog
} from '@sale/js/product_configurator_dialog/product_configurator_dialog';

patch(ProductConfiguratorDialog.prototype, {
    setup() {
        super.setup(...arguments);

        useSubEnv({
            isQuantityAllowed: this._isQuantityAllowed.bind(this),
        });
    },

    async _setQuantity(productTmplId, quantity) {
        const product = this._findProduct(productTmplId);
        if (!this._isQuantityAllowed(product, quantity)) {
            quantity = product.free_qty;
        }
        return super._setQuantity(productTmplId, quantity);
    },

    /**
     * Check whether the provided product quantity can be added to the cart.
     *
     * @param {Object} product - The provided product.
     * @param {Number} quantity - The new quantity of the product.
     * @return {Boolean} - Whether the provided product quantity can be added to the cart.
     */
    _isQuantityAllowed(product, quantity) {
        return !('free_qty' in product) || product.free_qty >= quantity;
    },

    /**
     * Check whether all selected product quantities can be added to the cart.
     *
     * @return {Boolean} - Whether all selected product quantities can be added to the cart.
     */
    areQuantitiesAllowed() {
        return this.state.products.every(p => this._isQuantityAllowed(p, p.quantity));
    },
});

```

## File: static\src\js\product_configurator_dialog\product_configurator_dialog.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">
    <t t-inherit="sale.ProductConfiguratorDialog" t-inherit-mode="extension">
        <button name="website_sale_product_configurator_continue_button" position="attributes">
            <attribute name="t-att-disabled" add="!areQuantitiesAllowed()" separator=" || "/>
        </button>
        <button name="website_sale_product_configurator_checkout_button" position="attributes">
            <attribute name="t-att-disabled" add="!areQuantitiesAllowed()" separator=" || "/>
        </button>
    </t>
</templates>

```

## File: static\src\js\tours\combo_configurator_tour_utils.js

```javascript
import configuratorTourUtils from '@sale/js/tours/combo_configurator_tour_utils';

function assertQuantityNotAvailable(productName) {
    return {
        content: `Assert that the requested quantity isn't available for ${productName}`,
        trigger: `
            ${configuratorTourUtils.comboItemSelector(productName, ['unselectable-card'])}
            span:contains("Requested quantity not available")
        `,
    };
}

function assertAllQuantitySelected(productName) {
    return {
        content: `Assert that all available quantity has been selected for ${productName}`,
        trigger: `
            ${configuratorTourUtils.comboItemSelector(productName)}
            i[title="All available quantity selected"]
        `,
    };
}

export default {
    assertQuantityNotAvailable,
    assertAllQuantitySelected,
};

```

## File: static\src\js\tours\product_configurator_tour_utils.js

```javascript
/** @odoo-module **/

import configuratorTourUtils from '@sale/js/tours/product_configurator_tour_utils';

function assertProductOutOfStock(productName) {
    return [
        {
            content: `Assert that ${productName} is out of stock`,
            trigger: `
                ${configuratorTourUtils.productSelector(productName)}
                td.o_sale_product_configurator_qty:contains("Out of stock")
            `,
        },
        {
            content: `Assert that ${productName} has no quantity`,
            trigger: `
                ${configuratorTourUtils.productSelector(productName)}
                td.o_sale_product_configurator_qty:not(:has(input[name="sale_quantity"]))
            `,
        },
    ];
}

function assertOptionalProductOutOfStock(productName) {
    return [
        {
            content: `Assert that ${productName} is out of stock`,
            trigger: `
                ${configuratorTourUtils.optionalProductSelector(productName)}
                td.o_sale_product_configurator_price:contains("Out of stock")
            `,
        },
        {
            content: `Assert that ${productName} has no "Add" button`,
            trigger: `
                ${configuratorTourUtils.optionalProductSelector(productName)}
                td.o_sale_product_configurator_price:not(:has(button:contains("Add")))
            `,
        },
    ];
}

export default {
    assertProductOutOfStock,
    assertOptionalProductOutOfStock,
};

```

## File: static\src\js\tours\website_sale_stock_reorder_from_portal.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { clickOnElement } from '@website/js/tours/tour_utils';

registry.category("web_tour.tours").add('website_sale_stock_reorder_from_portal', {
        url: '/my/orders',
    steps: () => [
        {
            content: 'Select first order',
            trigger: '.o_portal_my_doc_table a:first',
            run: "click",
        },
        clickOnElement('Reorder Again', '.o_wsale_reorder_button'),
        {
            content: "Check that there is one out of stock product",
            trigger: "#o_wsale_reorder_body div.text-warning span:contains('This product is out of stock.')",
            run: "click",
        },
        {
            content: "Check that there is one product that does not have enough stock",
            trigger: "#o_wsale_reorder_body div.text-warning:contains('You ask for 2.0 Units but only 1.0 are available.')",
        },
    ]
});

```

## File: static\src\xml\website_sale_stock_product_availability.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates>
    <t t-name="website_sale_stock.product_availability">
        <div t-if="is_storable and !prevent_zero_price_sale" id="product_stock_availability">
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
                                   type="text" placeholder="youremail@gmail.com" t-att-value="stock_notification_email? stock_notification_email: ''"/>
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
        </div>
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
            <field name="website_ribbon_id" position="before">
                <label for="allow_out_of_stock_order" invisible="not is_storable" string="Out-of-Stock"/>
                <div invisible="not is_storable">
                    <field name="allow_out_of_stock_order" class="oe_inline" /> Continue Selling
                </div>
            </field>
            <field name="website_ribbon_id" position="after">
                <label for="show_availability" invisible="not is_storable" string="Show Available Qty"/>
                <div invisible="not is_storable">
                    <field name="show_availability" class="oe_inline" />
                    <span invisible="not show_availability">
                        <label for="available_threshold" string="only if below" class="o_light_label"/>
                        <field name="available_threshold" class="oe_inline col-1" widget="integer"/>
                        Units
                    </span>
                </div>
                <field name="out_of_stock_message" invisible="not is_storable"/>
            </field>
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
            <xpath expr="//setting[@id='comparator_option_setting']" position="after">
                <setting id="product_availability_setting" string="Inventory Defaults" help="How to display products having low quantities (on hand - reserved)">
                    <div class="content-group">
                        <div class="row mt16"
                            id="website_warehouse_setting"
                            groups="stock.group_stock_multi_warehouses">
                            <field name="website_company_id" invisible="1"/>
                            <label for="website_warehouse_id" string="Warehouse" class="col-lg-3 o_light_label" />
                            <field name="website_warehouse_id" placeholder="All warehouses"/>
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
                                    <label for="available_threshold" string="only if below" class="o_light_label" invisible="not show_availability"/>
                                    <field name="available_threshold" class="oe_inline col-1" widget="integer" invisible="not show_availability"/>
                                    <span invisible="not show_availability">Units</span>
                                </div>
                            </div>
                        </div>
                    </div>
                </setting>
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
    <field name="name">Product Pages List (stock inherit)</field>
    <field name="model">product.template</field>
    <field name="inherit_id" ref="website_sale.product_pages_tree_view"/>
    <field name="arch" type="xml">
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

    <template id="website_sale_stock_cart_lines" inherit_id="website_sale.cart_lines" name="Shopping Cart Lines">
        <xpath expr="//input[@type='text'][hasclass('quantity')]" position="attributes">
            <attribute name="t-att-data-max">line._get_max_line_qty()</attribute>
        </xpath>
        <xpath expr="//div[@name='website_sale_cart_line_quantity']" position="after">
            <div class="availability_messages"/>
        </xpath>
    </template>

    <template id="website_sale_stock_product" inherit_id="website_sale.product" priority="4">
        <xpath expr="//div[@id='o_wsale_cta_wrapper']" position="after">
            <div class="availability_messages o_not_editable"/>
        </xpath>
        <xpath expr="//div[@id='product_details']" position="inside">
            <input id="wsale_user_email" type="hidden" t-att-value="user_email"/>
        </xpath>
    </template>

</odoo>

```

