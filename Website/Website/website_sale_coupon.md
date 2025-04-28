# Odoo Module: website_sale_coupon

Category: Website/Website

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models
from . import wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': "Coupons & Promotions for eCommerce",
    'summary': """Use coupon & promotion programs in your eCommerce store""",
    'description': """
Create coupon and promotion codes to share in order to boost your sales (free products, discounts, etc.). Shoppers can use them in the eCommerce checkout.

Coupon & promotion programs can be edited in the Catalog menu of the Website app.
    """,
    'category': 'Website/Website',
    'version': '1.0',
    'depends': ['website_sale', 'website_links', 'sale_coupon'],
    'data': [
        'security/ir.model.access.csv',
        'views/coupon_share_views.xml',
        'views/website_sale_templates.xml',
        'views/res_config_settings_views.xml',
        'views/sale_coupon_coupon_views.xml',
        'views/sale_coupon_program_views.xml',
    ],
    'auto_install': ['website_sale', 'sale_coupon'],
    'assets': {
        'web.assets_frontend': [
            'website_sale_coupon/static/src/js/coupon_toaster_widget.js',
        ],
        'web.assets_tests': [
            'website_sale_coupon/static/tests/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
from odoo import http, _
from odoo.addons.website_sale.controllers import main
from odoo.exceptions import UserError
from odoo.http import request

from werkzeug.urls import url_encode, url_parse


class WebsiteSale(main.WebsiteSale):

    @http.route(['/shop/pricelist'])
    def pricelist(self, promo, **post):
        order = request.website.sale_get_order()
        coupon_status = request.env['sale.coupon.apply.code'].sudo().apply_coupon(order, promo)
        if coupon_status.get('not_found'):
            return super(WebsiteSale, self).pricelist(promo, **post)
        elif coupon_status.get('error'):
            request.session['error_promo_code'] = coupon_status['error']
        return request.redirect(post.get('r', '/shop/cart'))

    @http.route()
    def shop_payment(self, **post):
        order = request.website.sale_get_order()
        order.recompute_coupon_lines()
        return super(WebsiteSale, self).shop_payment(**post)

    @http.route(['/shop/cart'], type='http', auth="public", website=True)
    def cart(self, **post):
        order = request.website.sale_get_order()
        order.recompute_coupon_lines()
        return super(WebsiteSale, self).cart(**post)

    @http.route(['/coupon/<string:code>'], type='http', auth='public', website=True, sitemap=False)
    def activate_coupon(self, code, r='/shop', **kw):
        url_parts = url_parse(r)
        url_query = url_parts.decode_query()
        url_query.pop('coupon_error', False)  # trust only Odoo error message

        request.session['pending_coupon_code'] = code
        order = request.website.sale_get_order()
        if order:
            result = order._try_pending_coupon()
            order.recompute_coupon_lines()
            if isinstance(result, UserError):
                url_query['coupon_error'] = result
            else:
                url_query['notify_coupon'] = code
        else:
            url_query['coupon_error'] = _("The coupon will be automatically applied when you add something in your cart.")
        redirect = url_parts.replace(query=url_encode(url_query))
        return request.redirect(redirect.to_url())

    # Override
    # Add in the rendering the free_shipping_line
    def _get_shop_payment_values(self, order, **kwargs):
        values = super(WebsiteSale, self)._get_shop_payment_values(order, **kwargs)
        values['free_shipping_lines'] = order._get_free_shipping_lines()
        return values

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: models\sale_coupon.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class SaleCoupon(models.Model):
    _inherit = 'coupon.coupon'

    def _check_coupon_code(self, order_date, partner_id, **kwargs):
        order = kwargs.get('order', False)
        if order and self.program_id.website_id and self.program_id.website_id != order.website_id:
            return {'error': 'This coupon is not valid on this website.'}
        return super()._check_coupon_code(order_date, partner_id, **kwargs)

    def action_coupon_share(self):
        """ Open a window to copy the coupon link """
        self.ensure_one()
        return self.env['coupon.share'].create_share_action(coupon=self)

```

## File: models\sale_coupon_program.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, _
from odoo.exceptions import ValidationError


class CouponProgram(models.Model):
    _name = 'coupon.program'
    _inherit = ['coupon.program', 'website.multi.mixin']

    @api.constrains('promo_code', 'website_id')
    def _check_promo_code_constraint(self):
        """ Only case where multiple same code could coexists is if they all belong to their own website.
            If the program is website generic, we should ensure there is no generic and no specific (even for other website) already
            If the program is website specific, we should ensure there is no existing code for this website or False
        """
        for program in self.filtered(lambda p: p.promo_code):
            domain = [('id', '!=', program.id), ('promo_code', '=', program.promo_code)]
            if program.website_id:
                domain += program.website_id.website_domain()
            if self.search(domain):
                raise ValidationError(_('The program code must be unique by website!'))

    def _filter_programs_on_website(self, order):
        return self.filtered(lambda program: not program.website_id or program.website_id.id == order.website_id.id)

    @api.model
    def _filter_programs_from_common_rules(self, order, next_order=False):
        programs = self._filter_programs_on_website(order)
        return super(CouponProgram, programs)._filter_programs_from_common_rules(order, next_order)

    def _check_promo_code(self, order, coupon_code):
        if self.website_id and self.website_id != order.website_id:
            return {'error': 'This promo code is not valid on this website.'}
        return super()._check_promo_code(order, coupon_code)

    def action_program_share(self):
        """ Open a window to copy the program link """
        self.ensure_one()
        return self.env['coupon.share'].create_share_action(program=self)

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
from datetime import timedelta

from odoo import api, fields, models
from odoo.exceptions import UserError
from odoo.http import request


class SaleOrder(models.Model):
    _inherit = "sale.order"

    def _try_pending_coupon(self):
        if not request:
            return

        pending_coupon_code = request.session.get('pending_coupon_code')
        if pending_coupon_code:
            try:
                self.env['sale.coupon.apply.code'].with_context(active_id=self.id).create({
                    'coupon_code': pending_coupon_code
                }).process_coupon()
                request.session.pop('pending_coupon_code')
            except UserError as e:
                return e
        return True

    def recompute_coupon_lines(self):
        for order in self:
            order._try_pending_coupon()
        return super().recompute_coupon_lines()

    def _compute_website_order_line(self):
        """ This method will merge multiple discount lines generated by a same program
            into a single one (temporary line with `new()`).
            This case will only occur when the program is a discount applied on multiple
            products with different taxes.
            In this case, each taxes will have their own discount line. This is required
            to have correct amount of taxes according to the discount.
            But we wan't these lines to be `visually` merged into a single one in the
            e-commerce since the end user should only see one discount line.
            This is only possible since we don't show taxes in cart.
            eg:
                line 1: 10% discount on product with tax `A` - $15
                line 2: 10% discount on product with tax `B` - $11.5
                line 3: 10% discount on product with tax `C` - $10
            would be `hidden` and `replaced` by
                line 1: 10% discount - $36.5

            Note: The line will be created without tax(es) and the amount will be computed
                  depending if B2B or B2C is enabled.
        """
        super()._compute_website_order_line()
        for order in self:
            # TODO: potential performance bottleneck downstream
            programs = order._get_applied_programs_with_rewards_on_current_order()
            for program in programs:
                program_lines = order.order_line.filtered(lambda line:
                    line.product_id == program.discount_line_product_id)
                if len(program_lines) > 1:
                    if self.env.user.has_group('sale.group_show_price_subtotal'):
                        price_unit = sum(program_lines.mapped('price_subtotal'))
                    else:
                        price_unit = sum(program_lines.mapped('price_total'))
                    # TODO: batch then flush
                    order.website_order_line += self.env['sale.order.line'].new({
                        'product_id': program_lines[0].product_id.id,
                        'price_unit': price_unit,
                        'name': program_lines[0].name,
                        'product_uom_qty': 1,
                        'product_uom': program_lines[0].product_uom.id,
                        'order_id': order.id,
                        'is_reward_line': True,
                    })
                    order.website_order_line -= program_lines

    def _compute_cart_info(self):
        super(SaleOrder, self)._compute_cart_info()
        for order in self:
            reward_lines = order.website_order_line.filtered(lambda line: line.is_reward_line)
            order.cart_quantity -= int(sum(reward_lines.mapped('product_uom_qty')))

    def get_promo_code_error(self, delete=True):
        error = request.session.get('error_promo_code')
        if error and delete:
            request.session.pop('error_promo_code')
        return error

    def _get_coupon_program_domain(self):
        return [('website_id', 'in', [False, self.website_id.id])]

    def _cart_update(self, product_id=None, line_id=None, add_qty=0, set_qty=0, **kwargs):
        res = super(SaleOrder, self)._cart_update(product_id=product_id, line_id=line_id, add_qty=add_qty, set_qty=set_qty, **kwargs)
        self.recompute_coupon_lines()
        return res

    def _get_free_shipping_lines(self):
        self.ensure_one()
        free_shipping_prgs_ids = self._get_applied_programs_with_rewards_on_current_order().filtered(lambda p: p.reward_type == 'free_shipping')
        if not free_shipping_prgs_ids:
            return self.env['sale.order.line']
        free_shipping_product_ids = free_shipping_prgs_ids.mapped('discount_line_product_id')
        return self.order_line.filtered(lambda l: l.product_id in free_shipping_product_ids)

    @api.autovacuum
    def _gc_abandoned_coupons(self, *args, **kwargs):
        """Remove/free coupon from abandonned ecommerce order."""
        ICP = self.env['ir.config_parameter']
        validity = ICP.get_param('website_sale_coupon.abandonned_coupon_validity', 4)
        validity = fields.Datetime.to_string(fields.datetime.now() - timedelta(days=int(validity)))
        coupon_to_reset = self.env['coupon.coupon'].search([
            ('state', '=', 'used'),
            ('sales_order_id.state', '=', 'draft'),
            ('sales_order_id.write_date', '<', validity),
            ('sales_order_id.website_id', '!=', False),
        ])
        for coupon in coupon_to_reset:
            coupon.sales_order_id.applied_coupon_ids -= coupon
        coupon_to_reset.write({'state': 'new'})
        coupon_to_reset.mapped('sales_order_id').recompute_coupon_lines()

```

## File: models\website.py

```python
# -*- coding: utf-8 -*-
from odoo import models
from odoo.http import request


class Website(models.Model):
    _inherit = 'website'

    def sale_reset(self):
        request.session.pop('pending_coupon_code')
        return super().sale_reset()

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import sale_coupon
from . import sale_coupon_program
from . import sale_order

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_coupon_share,access_coupon_share,model_coupon_share,sales_team.group_sale_manager,1,1,1,0

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="98.162%" x2="0%" y1="1.838%" y2="100%"><stop offset="0%" stop-color="#797DA5"/><stop offset="50.799%" stop-color="#6D7194"/><stop offset="100%" stop-color="#626584"/></linearGradient><path id="d" d="M50 31.035a3.5 3.5 0 1 0 0 6.93v3.06a1.633 1.633 0 0 1-1.588 1.286L26.974 43l.998 4h18.955c.998 0 .998 2 0 2h-20.95L19.99 25h-1.996v1c0 .667-.332 1-.997 1S16 26.667 16 26v-2c.066-.667.398-1 .998-1h3.99c.516 0 .848.333.998 1l.998 3.281 25.428.037c1.045 0 1.588.761 1.588 2.018v1.7zM45.43 55a2.497 2.497 0 0 1-2.493-2.5c0-1.38 1.116-2.5 2.494-2.5a2.497 2.497 0 0 1 2.494 2.5c0 1.38-1.117 2.5-2.494 2.5zm-18.955 0a2.497 2.497 0 0 1-2.494-2.5c0-1.38 1.117-2.5 2.494-2.5a2.497 2.497 0 0 1 2.495 2.5c0 1.38-1.117 2.5-2.495 2.5zm12.7-24.996a.51.51 0 0 0-.409.2l-7.668 9.056c-.233.31.004.738.41.738l.696-.002a.51.51 0 0 0 .409-.2l7.63-9.056c.234-.312-.005-.74-.41-.738l-.658.002zm-5.841 4.371c1.29 0 2.334-.979 2.334-2.188 0-1.208-1.044-2.187-2.334-2.187S31 30.979 31 32.188c0 1.208 1.044 2.187 2.334 2.187zm0-3.125c.552 0 1 .42 1 .938 0 .517-.448.937-1 .937s-1-.42-1-.938c0-.517.448-.937 1-.937zm4.668 4.375c-1.29 0-2.334.979-2.334 2.188 0 1.208 1.044 2.187 2.334 2.187s2.334-.979 2.334-2.188c0-1.208-1.044-2.187-2.334-2.187zm0 3.125c-.553 0-1-.42-1-.938 0-.517.447-.937 1-.937.552 0 1 .42 1 .938 0 .517-.448.937-1 .937z"/><path id="e" d="M50 30a3 3 0 0 0 0 6v3.025a1.633 1.633 0 0 1-1.588 1.286L26.974 41l.998 4h18.955c.998 0 .998 2 0 2h-20.95L19.99 23h-1.996v1c0 .667-.332 1-.997 1S16 24.667 16 24v-2c.066-.667.398-1 .998-1h3.99c.516 0 .848.333.998 1l.998 3.281 25.428.037c1.045 0 1.588.761 1.588 2.018V30zm-4.57 23a2.497 2.497 0 0 1-2.493-2.5c0-1.38 1.116-2.5 2.494-2.5a2.497 2.497 0 0 1 2.494 2.5c0 1.38-1.117 2.5-2.494 2.5zm-18.955 0a2.497 2.497 0 0 1-2.494-2.5c0-1.38 1.117-2.5 2.494-2.5a2.497 2.497 0 0 1 2.495 2.5c0 1.38-1.117 2.5-2.495 2.5zM48 26a1 1 0 1 0 0 2 1 1 0 0 0 0-2zm0 11a1 1 0 1 0 0 2 1 1 0 0 0 0-2zm-8.825-8.996a.51.51 0 0 0-.409.2l-7.668 9.056c-.233.31.004.738.41.738l.696-.002a.51.51 0 0 0 .409-.2l7.63-9.056c.234-.312-.005-.74-.41-.738l-.658.002zm-5.841 4.371c1.29 0 2.334-.979 2.334-2.188 0-1.208-1.044-2.187-2.334-2.187S31 28.979 31 30.188c0 1.208 1.044 2.187 2.334 2.187zm0-3.125c.552 0 1 .42 1 .938 0 .517-.448.937-1 .937s-1-.42-1-.938c0-.517.448-.937 1-.937zm4.668 4.375c-1.29 0-2.334.979-2.334 2.188 0 1.208 1.044 2.187 2.334 2.187s2.334-.979 2.334-2.188c0-1.208-1.044-2.187-2.334-2.187zm0 3.125c-.553 0-1-.42-1-.938 0-.517.447-.937 1-.937.552 0 1 .42 1 .938 0 .517-.448.937-1 .937z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M33.317 69H4c-2 0-4-1-4-4V38.29l16.258-16.95L21 21l2 5h26.583l.371 3.987-3.862 4.883 3.764 1.445-.11 3.27L45.076 45H47l.507 1.61-1.993 1.943 1.815 3.434L33.317 69z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: static\src\js\coupon_toaster_widget.js

```javascript
/** @odoo-module **/

import publicWidget from 'web.public.widget';
import {registry} from "@web/core/registry";

const CouponToasterWidget = publicWidget.Widget.extend({
    start() {
        let options = {};
        const $content = this.$('.coupon-message-content');
        const $title = this.$('.coupon-message-title');

        if ($content.length) {
            Object.assign(options, {message: $content[0].innerHTML});
        }
        if ($title.length) {
            Object.assign(options, {title: $title[0].innerHTML});
        }

        if (this.$el.hasClass('coupon-info-message')) {
            this.displayNotification(Object.assign({type: 'success'}, options));
        } else if (this.$el.hasClass('coupon-error-message')) {
            this.displayNotification(Object.assign({type: 'danger'}, options));
        }

        return this._super(...arguments);
    },
});

registry.category("public_root_widgets").add("CouponToasterWidget", {
    Widget: CouponToasterWidget,
    selector: '.coupon-message',
});

export default CouponToasterWidget;

```

## File: views\coupon_share_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="coupon_share_view_form" model="ir.ui.view">
        <field name="name">coupon.share.form</field>
        <field name="model">coupon.share</field>
        <field name="arch" type="xml">
            <form string="Share Coupon" edit="1" create="0">
                <field name="program_website_id" invisible="1"/>
                <group attrs="{'invisible': [('website_id', '=', False)]}">
                    <div colspan="2">
                        <p>
                            You can share this promotion with your customers.
                            It will be applied at checkout when the customer uses this link.
                        </p>
                    </div>
                    <field name="share_link" widget="CopyClipboardURL" nolabel="1"/>
                </group>
                <hr attrs="{'invisible': ['|', ('website_id', '=', False), ('id', '!=', False)]}"/>
                <group attrs="{'invisible': [('id', '!=', False)]}">
                    <field name="website_id" groups="website.group_multi_website" widget="selection"
                           attrs="{'invisible': [('program_website_id', '!=', False)]}"/>
                    <field name="redirect"/>
                </group>
                <footer>
                    <button string="Done" class="btn-primary" special="cancel" data-hotkey="z"/>
                    <button string="Generate Short Link" class="btn-secondary" type="object" name="action_generate_short_link"
                            attrs="{'invisible': ['|', ('website_id', '=', False), ('id', '!=', False)]}" data-hotkey="g"/>
                </footer>
            </form>
        </field>
    </record>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.website.sale.coupon</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="website.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='website_sale_coupon']" position="after">
                <div class="content-group">
                    <div class="mt8" attrs="{'invisible': [('module_sale_coupon', '=', False)]}">
                        <button name="%(coupon.coupon_program_action_promo_program)d" icon="fa-arrow-right" type="action" string="Promotion Programs" class="btn-link"/>
                    </div>
                    <div class="mt8" attrs="{'invisible': [('module_sale_coupon', '=', False)]}">
                        <button name="%(coupon.coupon_program_action_coupon_program)d" icon="fa-arrow-right" type="action" string="Coupon Programs" class="btn-link"/>
                    </div>
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\sale_coupon_coupon_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="coupon_view_tree" model="ir.ui.view">
        <field name="name">coupon.coupon.tree</field>
        <field name="model">coupon.coupon</field>
        <field name="inherit_id" ref="coupon.coupon_view_tree"/>
        <field name="arch" type="xml">
            <button name="action_coupon_send" position="after">
                <button name="action_coupon_share" string="Share" type="object" icon="fa-share-alt" attrs="{'invisible': [('state', 'not in', ['new', 'sent'])]}"/>
            </button>
        </field>
    </record>
</odoo>

```

## File: views\sale_coupon_program_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <menuitem action="coupon.coupon_program_action_promo_program" id="menu_promotion_type_config" name="Promotion Programs" parent="website_sale.menu_catalog" groups="sales_team.group_sale_manager" sequence="50"/>
    <menuitem id="menu_coupon_type_config" action="coupon.coupon_program_action_coupon_program" name="Coupon Programs" parent="website_sale.menu_catalog" groups="sales_team.group_sale_manager" sequence="51"/>

    <record model="ir.ui.view" id="sale_coupon_program_view_form_common_website">
        <field name="name">coupon.program.common.form</field>
        <field name="model">coupon.program</field>
        <field name="inherit_id" ref="coupon.coupon_program_view_form_common"/>
        <field name="arch" type="xml">
            <group name="validity" position="inside">
                <label for="website_id" groups="website.group_multi_website"/>
                <div>
                    <field name="website_id" options="{'no_create': True}" groups="website.group_multi_website"/>
                </div>
            </group>
        </field>
    </record>

    <record id="sale_coupon_program_view_tree_website" model="ir.ui.view">
        <field name="name">coupon.program.tree</field>
        <field name="model">coupon.program</field>
        <field name="inherit_id" ref="coupon.coupon_program_view_tree"/>
        <field name="arch" type="xml">
            <field name="name" position="after">
                <field name="promo_code" invisible="1"/>
                <field name="website_id" groups="website.group_multi_website"/>
                <button name="action_program_share" string="Share" type="object" icon="fa-share-alt" attrs="{'invisible': [('promo_code', '=', False)]}"/>
            </field>
        </field>
    </record>

</odoo>
```

## File: views\website_sale_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="website_sale_coupon_cart_hide_qty" inherit_id="website_sale.cart_lines">
    <xpath expr="//del" position="attributes">
        <attribute name="t-if">not line.is_reward_line</attribute>
    </xpath>
</template>
<template id="layout" inherit_id="website.layout">
    <body position="inside">
        <t t-set="coupon_error" t-value="request.params.get('coupon_error')"/>
        <t t-set="pending_coupon_code" t-value="request.session.get('pending_coupon_code')"/>
        <t t-if="coupon_error and pending_coupon_code">
            <div class="d-none coupon-message coupon-error-message">
                <span class="coupon-message-title">Could not apply the promo code: <t t-out="pending_coupon_code"/></span>
                <span class="coupon-message-content" t-out="coupon_error"/>
            </div>
        </t>
        <t t-set="notify_coupon" t-value="request.params.get('notify_coupon')"/>
        <div t-if="notify_coupon" class="d-none coupon-message coupon-info-message">
            <span class="coupon-message-content">The following promo code was applied on your order: <t t-out="notify_coupon"/></span>
        </div>
    </body>
</template>
<template id="sale_coupon_result" inherit_id="website_sale.coupon_form">
    <xpath expr="//form[@name='coupon_code']" position="after">
        <t t-if="website_sale_order and website_sale_order.applied_coupon_ids">
            <t t-foreach="website_sale_order.applied_coupon_ids" t-as="coupon">
                <div class="alert alert-success text-left mt16" role="alert">
                    You have successfully applied following promo code: <strong t-esc="coupon.code"/>
                </div>
            </t>
        </t>
        <t t-if="website_sale_order and website_sale_order.promo_code">
            <div class="alert alert-success text-left mt16" role="alert">
                You have successfully applied following promo code: <strong t-esc="website_sale_order.promo_code"/>
            </div>
        </t>
        <t t-if="website_sale_order and website_sale_order.generated_coupon_ids">
            <t t-foreach="website_sale_order.generated_coupon_ids.filtered(lambda c: c.state != 'expired')" t-as="coupon">
                <div class="alert alert-success text-left mt16" role="alert">
                    Your reward <strong t-esc="coupon.discount_line_product_id.name"/> is available on a next order with this promo code: <strong t-esc="coupon.code"/>
                </div>
            </t>
        </t>
        <t t-if="request.params.get('code_not_available')">
            <div class="alert alert-danger text-left mt16" role="alert">
                Invalid or expired promo code.
            </div>
        </t>
        <t t-if="website_sale_order.get_promo_code_error(delete=False)">
            <div class="alert alert-danger text-left mt16" role="alert">
                <t t-esc="website_sale_order.get_promo_code_error()"/>
            </div>
        </t>
    </xpath>
    <xpath expr="//t[@name='code_not_available']" position="replace"/>
</template>

<template id="cart_discount" name="Show Discount in Subtotal" customize_show="True" active="False" inherit_id="website_sale.total">
    <xpath expr="//tr[@id='order_total_untaxed']" position="before">
        <tr t-if="website_sale_order and website_sale_order.reward_amount">
          <td class="text-right border-0 text-muted" title="Discounted amount">Discount:</td>
          <td class="text-xl-right border-0 text-muted">
               <span t-field="website_sale_order.reward_amount" style="white-space: nowrap;"
                 class="monetary_field"
                 t-options='{
                          "widget": "monetary",
                          "display_currency": website_sale_order.currency_id,
                 }'/>
          </td>
        </tr>
    </xpath>
</template>

<template id="reduction_coupon_code" inherit_id="website_sale.reduction_code">
    <xpath expr="//t[@t-set='force_coupon']" position="replace">
        <t t-set='force_coupon' t-value="website_sale_order.pricelist_id.code or request.params.get('code_not_available') or website_sale_order.promo_code or website_sale_order.generated_coupon_ids or website_sale_order.applied_coupon_ids or website_sale_order.get_promo_code_error(delete=False)"/>
    </xpath>
</template>

<template id="cart_summary" name="Payment" inherit_id="website_sale.cart_summary">
    <xpath expr="//table[@id='cart_products']/tbody/tr/td[hasclass('td-price')]/child::*" position="attributes">
        <attribute name="t-att-style">'display: None;' if free_shipping_lines and line in free_shipping_lines else ''</attribute>
    </xpath>
</template>
</odoo>

```

## File: wizard\sale_coupon_share.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from werkzeug.urls import url_encode

from odoo import fields, models, api, _
from odoo.exceptions import UserError, ValidationError


class SaleCouponShare(models.TransientModel):
    _name = 'coupon.share'
    _description = 'Create links that apply a coupon and redirect to a specific page'

    def _get_default_website_id(self):
        # program_website_id = self.program_website_id
        program_website_id = self.env['coupon.program'].browse(self.env.context.get('default_program_id')).website_id
        if program_website_id:
            return program_website_id
        else:
            Website = self.env['website']
            websites = Website.search([])
            return len(websites) == 1 and websites or Website

    website_id = fields.Many2one('website', required=True, default=_get_default_website_id)
    coupon_id = fields.Many2one('coupon.coupon', domain="[('program_id', '=', program_id)]")
    program_id = fields.Many2one('coupon.program', required=True, domain=[
        '|', ('program_type', '=', 'coupon_program'),
        ('promo_code_usage', '=', 'code_needed'),
    ])
    program_website_id = fields.Many2one('website', string='Program Website', related='program_id.website_id')

    promo_code = fields.Char(compute='_compute_promo_code')
    share_link = fields.Char(compute='_compute_share_link')
    redirect = fields.Char(required=True, default='/shop')

    @api.constrains('coupon_id', 'program_id')
    def _check_program(self):
        if self.filtered(lambda record: not record.coupon_id and record.program_id.program_type == 'coupon_program'):
            raise ValidationError(_("A coupon is needed for coupon programs."))

    @api.constrains('website_id', 'program_id')
    def _check_website(self):
        if self.filtered(lambda record: record.program_website_id and record.program_website_id != record.website_id):
            raise ValidationError(_("The shared website should correspond to the website of the program."))

    @api.depends('coupon_id.code', 'program_id.promo_code')
    def _compute_promo_code(self):
        for record in self:
            record.promo_code = record.coupon_id.code or record.program_id.promo_code

    @api.depends('website_id', 'redirect')
    @api.depends_context('use_short_link')
    def _compute_share_link(self):
        for record in self:
            target_url = '{base}/coupon/{code}?{query}'.format(
                base=record.website_id.get_base_url(),
                code=record.promo_code,
                query=url_encode({'r': record.redirect}),
            )

            if record.env.context.get('use_short_link'):
                tracker = self.env['link.tracker'].search([('url', '=', target_url)], limit=1)
                if not tracker:
                    tracker = self.env['link.tracker'].create({'url': target_url})
                record.share_link = tracker.short_url
            else:
                record.share_link = target_url

    def action_generate_short_link(self):
        return {
            'name': _('Share Coupon'),
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'coupon.share',
            'target': 'new',
            'res_id': self.id,
            'context': {
                'use_short_link': True,
            }
        }

    @api.model
    def create_share_action(self, coupon=None, program=None):
        if bool(program) == bool(coupon):
            raise UserError(_("Provide either a coupon or a program."))

        return {
            'name': _('Share Coupon'),
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'coupon.share',
            'target': 'new',
            'flags': {'form': {'action_buttons': True}},
            'context': {
                'form_view_initial_mode': 'edit',
                'default_program_id': program and program.id or coupon.program_id.id,
                'default_coupon_id': coupon and coupon.id or None,
            }
        }

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import sale_coupon_share

```

