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
            'website_sale_stock/static/src/js/**/*',
            'website_sale_stock/static/src/xml/**/*',
        ],
        'web.assets_tests': [
            'website_sale_stock/static/tests/tours/website_sale_stock_multilang.js',
            'website_sale_stock/static/tests/tours/website_sale_stock_stock_notification.js',
            'website_sale_stock/static/tests/tours/website_sale_stock_message_after_close_configurator_modal.js'
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
from werkzeug.exceptions import BadRequest


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
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.http import request, route

from odoo.addons.website_sale.controllers.variant import WebsiteSaleVariantController


class WebsiteSaleStockVariantController(WebsiteSaleVariantController):

    @route()
    def get_combination_info_website(self, *args, **kwargs):
        request.update_context(website_sale_stock_get_quantity=True)
        return super().get_combination_info_website(*args, **kwargs)

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
                return sum(cart._get_common_product_lines(product=self).mapped('product_uom_qty'))
        return 0

    def _is_sold_out(self):
        self.ensure_one()
        if not self.type == 'product':
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

from odoo import fields, models
from odoo.http import request
from odoo.tools.translate import html_translate


class ProductTemplate(models.Model):
    _inherit = 'product.template'

    allow_out_of_stock_order = fields.Boolean(string='Continue selling when out-of-stock', default=True)

    available_threshold = fields.Float(string='Show Threshold', default=5.0)
    show_availability = fields.Boolean(string='Show availability Qty', default=False)
    out_of_stock_message = fields.Html(string="Out-of-Stock Message", translate=html_translate)

    def _is_sold_out(self):
        return self.type == 'product' and self.product_variant_id._is_sold_out()

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

        return res

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
        free_qty = (product or line.product_id).with_context(warehouse=self.warehouse_id.id).free_qty
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
            if line.product_id.type == 'product' and not line.product_id.allow_out_of_stock_order:
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
            if product.type != 'product' or product.allow_out_of_stock_order:
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
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


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
            self.env['ir.default'].sudo()._get('sale.order', 'warehouse_id', company_id=self.company_id.id) or
            self.env['ir.default'].sudo()._get('sale.order', 'warehouse_id') or
            self.env['stock.warehouse'].sudo().search([('company_id', '=', self.company_id.id)], limit=1).id
        )

    # FIXME VFE check if still needed
    def sale_get_order(self, *args, **kwargs):
        so = super().sale_get_order(*args, **kwargs)
        return so.with_context(warehouse=so.warehouse_id.id) if so else so

    def _get_product_available_qty(self, product):
        return product.with_context(warehouse=self._get_warehouse_available()).free_qty

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
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path fill-rule="evenodd" clip-rule="evenodd" d="M15.724 6.397C16.377 4.94 17.852 4 19.481 4h11.037c1.63 0 3.104.94 3.757 2.397L37.236 13H41.9c2.46 0 4.367 2.099 4.07 4.481l-3.106 25C42.613 44.49 40.866 46 38.793 46H11.207c-2.074 0-3.82-1.51-4.07-3.519l-3.107-25C3.734 15.1 5.64 13 8.1 13h4.663l2.961-6.603ZM32.917 13H17.082c0-.56.123-1.134.39-1.691l.956-2C19.102 7.9 20.551 7 22.144 7h5.711c1.593 0 3.042.9 3.716 2.308l.957 2c.266.558.39 1.132.39 1.692Z" fill="#088BF5"/><path fill-rule="evenodd" clip-rule="evenodd" d="M8.514 45.016a3.963 3.963 0 0 1-1.377-2.535l-3.107-25C3.734 15.1 5.64 13 8.1 13h4.663l2.961-6.603C16.377 4.94 17.852 4 19.481 4h11.037c1.63 0 3.104.94 3.757 2.397l2.59 5.777C35.5 28.256 23.848 41.405 8.515 45.016ZM17.082 13h15.835c0-.56-.123-1.134-.39-1.691l-.956-2C30.897 7.9 29.448 7 27.855 7h-5.711c-1.593 0-3.042.9-3.716 2.308l-.956 2a3.904 3.904 0 0 0-.39 1.692Z" fill="#2EBCFA"/><path d="M17.142 20.864a4 4 0 0 1 4.899-2.828l11.59 3.106a4 4 0 0 1 2.83 4.899l-3.107 11.59a4 4 0 0 1-4.899 2.83l-11.59-3.107a4 4 0 0 1-2.83-4.899l3.107-11.59Z" fill="#fff"/></svg>

```

## File: static\src\js\variant_mixin.js

```javascript
/** @odoo-module **/

import VariantMixin from "@website_sale/js/sale_variant_mixin";
import publicWidget from "@web/legacy/js/public/public_widget";
import { renderToFragment } from "@web/core/utils/render";
import { formatFloat } from "@web/core/utils/numbers";

import "@website_sale/js/website_sale";

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

export default VariantMixin;

```

## File: static\src\js\website_sale.js

```javascript
/** @odoo-module **/

import { WebsiteSale } from '@website_sale/js/website_sale';
import { isEmail } from '@web/core/utils/strings';

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

        this.rpc("/shop/add/stock_notification", {
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
        if (product.combinationInfo.allow_out_of_stock_order || product.type !== "product") {
            return;
        }
        product.max_quantity_available = product.combinationInfo.free_qty;
        if (!product.max_quantity_available) {
            product.add_to_cart_allowed = false;
        }
        if (product.max_quantity_available < product.qty) {
            product.qty_warning = _t(
                "You ask for %s Units but only %s are available.",
                product.qty.toFixed(1),
                product.max_quantity_available.toFixed(1)
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
                "You ask for %s Units but only %s are available.",
                newQty.toFixed(1),
                product.max_quantity_available.toFixed(1)
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

## File: static\src\js\tours\website_sale_stock_reorder_from_portal.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import wTourUtils from '@website/js/tours/tour_utils';

registry.category("web_tour.tours").add('website_sale_stock_reorder_from_portal', {
        test: true,
        url: '/my/orders',
    steps: () => [
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
            isCheck: true,
        },
    ]
});


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
                <label for="allow_out_of_stock_order" invisible="type in ['service', 'consu']" string="Out-of-Stock"/>
                <div invisible="type in ['service', 'consu']">
                    <field name="allow_out_of_stock_order" class="oe_inline" /> Continue Selling
                </div>

                <label for="show_availability" invisible="type in ['service', 'consu']" string="Show Available Qty"/>
                <div invisible="type in ['service', 'consu']">
                    <field name="show_availability" class="oe_inline" />
                    <span invisible="not show_availability">
                        <label for="available_threshold" string="only if below" class="o_light_label"/>
                        <field name="available_threshold" class="oe_inline col-1" widget="integer"/>
                        Units
                    </span>
                </div>
                <field name="out_of_stock_message" invisible="type in ['service', 'consu']"/>
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
            <xpath expr="//setting[@id='comparator_option_setting']" position="after">
                <setting id="product_availability_setting" string="Inventory Defaults" help="How to display products having low quantities (on hand - reserved)">
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
        <xpath expr="//div[@name='website_sale_cart_line_quantity']" position="after">
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

