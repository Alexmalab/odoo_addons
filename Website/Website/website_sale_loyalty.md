# Odoo Module: website_sale_loyalty

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
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': "Coupons, Promotions, Gift Card and Loyalty for eCommerce",
    'summary': """Use coupon, promotion, gift cards and loyalty programs in your eCommerce store""",
    'description': """
Create coupon, promotion codes, gift cards and loyalty programs to boost your sales (free products, discounts, etc.). Shoppers can use them in the eCommerce checkout.

Coupon & promotion programs can be edited in the Catalog menu of the Website app.
    """,
    'category': 'Website/Website',
    'version': '1.0',
    'depends': ['website_sale', 'website_links', 'sale_loyalty'],
    'data': [
        'security/ir.model.access.csv',

        'views/loyalty_card_views.xml',
        'views/loyalty_program_views.xml',
        'views/snippets.xml',
        'views/website_sale_templates.xml',
        'views/website_sale_loyalty_menus.xml',

        'wizard/coupon_share_views.xml',
        'wizard/res_config_settings_views.xml',
    ],
    'demo': [
        'data/product_demo.xml',
    ],
    'auto_install': ['website_sale', 'sale_loyalty'],
    'assets': {
        'web.assets_frontend': [
            'website_sale_loyalty/static/src/**/*',
        ],
        'web.assets_tests': [
            'website_sale_loyalty/static/tests/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\delivery.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from functools import partial

from odoo.http import request

from odoo.addons.payment import utils as payment_utils
from odoo.addons.website_sale.controllers.delivery import Delivery


class WebsiteSaleLoyaltyDelivery(Delivery):

    def _order_summary_values(self, order, **post):
        to_html = partial(
            request.env['ir.qweb.field.monetary'].value_to_html,
            options={'display_currency': order.currency_id},
        )
        res = super()._order_summary_values(order, **post)
        free_shipping_lines = order._get_free_shipping_lines()
        if free_shipping_lines:
            shipping_discount = sum(free_shipping_lines.mapped('price_subtotal'))
            res['amount_delivery_discounted'] = to_html(shipping_discount)
            res['delivery_discount_minor_amount'] = payment_utils.to_minor_currency_units(
                shipping_discount, order.currency_id
            )
        res['discount_reward_amounts'] = [
            to_html(sum(lines.mapped('price_subtotal')))
            for reward, lines in order.order_line.grouped('reward_id').items()
            if reward.reward_type == 'discount'
        ]
        return res

```

## File: controllers\main.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from werkzeug.urls import url_encode, url_parse

from odoo import _
from odoo.exceptions import UserError
from odoo.http import request, route

from odoo.addons.website_sale.controllers import main


class WebsiteSale(main.WebsiteSale):

    @route()
    def pricelist(self, promo, **post):
        order = request.website.sale_get_order()
        if not order:
            return request.redirect('/shop')
        coupon_status = order._try_apply_code(promo)
        if coupon_status.get('not_found'):
            return super().pricelist(promo, **post)
        elif coupon_status.get('error'):
            request.session['error_promo_code'] = coupon_status['error']
        elif 'error' not in coupon_status:
            reward_successfully_applied = True
            if len(coupon_status) == 1:
                coupon, rewards = next(iter(coupon_status.items()))
                if request.env.context.get('product_id') or (len(rewards) == 1 and not rewards.multi_product):
                    reward_successfully_applied = self._apply_reward(order, rewards, coupon)

            if reward_successfully_applied:
                request.session['successful_code'] = promo
        return request.redirect(post.get('r', '/shop/cart'))

    @route()
    def shop_payment(self, **post):
        order = request.website.sale_get_order()
        if order:
            order._update_programs_and_rewards()
            order._auto_apply_rewards()
        return super().shop_payment(**post)

    @route()
    def cart(self, **post):
        order = request.website.sale_get_order()
        if order and order.state != 'draft':
            request.session['sale_order_id'] = None
            order = request.website.sale_get_order()
        if order:
            order._update_programs_and_rewards()
            order._auto_apply_rewards()
        return super().cart(**post)

    @route(['/coupon/<string:code>'], type='http', auth='public', website=True, sitemap=False)
    def activate_coupon(self, code, r='/shop', **kw):
        url_parts = url_parse(r)
        url_query = url_parts.decode_query()
        url_query.pop('coupon_error', False)  # trust only Odoo error message
        url_query.pop('coupon_error_type', False)
        code = code.strip()

        request.session['pending_coupon_code'] = code
        order = request.website.sale_get_order()
        if order:
            result = order._try_pending_coupon()
            if isinstance(result, dict) and 'error' in result:
                url_query['coupon_error'] = result['error']
            else:
                url_query['notify_coupon'] = code
        else:
            url_query['coupon_error'] = _("The coupon will be automatically applied when you add something in your cart.")
            url_query['coupon_error_type'] = 'warning'
        redirect = url_parts.replace(query=url_encode(url_query))
        return request.redirect(redirect.to_url())

    @route('/shop/claimreward', type='http', auth='public', website=True, sitemap=False)
    def claim_reward(self, reward_id, code=None, **post):
        order_sudo = request.website.sale_get_order()
        redirect = post.get('r', '/shop/cart')
        if not order_sudo:
            return request.redirect(redirect)

        try:
            reward_id = int(reward_id)
        except ValueError:
            reward_id = None

        reward_sudo = request.env['loyalty.reward'].sudo().browse(reward_id).exists()
        if not reward_sudo:
            return request.redirect(redirect)

        if reward_sudo.multi_product and 'product_id' in post:
            request.update_context(product_id=int(post['product_id']))
        else:
            request.redirect(redirect)

        program_sudo = reward_sudo.program_id
        claimable_rewards = order_sudo._get_claimable_and_showable_rewards()
        coupon = request.env['loyalty.card']
        for coupon_, rewards in claimable_rewards.items():
            if reward_sudo in rewards:
                coupon = coupon_
                if code == coupon.code and (
                    (program_sudo.trigger == 'with_code' and program_sudo.program_type != 'promo_code')
                    or (program_sudo.trigger == 'auto'
                        and program_sudo.applies_on == 'future'
                        and program_sudo.program_type not in ('ewallet', 'loyalty'))
                ):
                    return self.pricelist(code)
        if coupon:
            self._apply_reward(order_sudo, reward_sudo, coupon)
        return request.redirect(redirect)

    def _apply_reward(self, order, reward, coupon):
        """Try to apply the given program reward

        :returns: whether the reward was successfully applied
        :rtype: bool
        """
        product_id = request.env.context.get('product_id')
        product = product_id and request.env['product.product'].sudo().browse(product_id)
        try:
            reward_status = order._apply_program_reward(reward, coupon, product=product)
        except UserError as e:
            request.session['error_promo_code'] = str(e)
            return False
        if 'error' in reward_status:
            request.session['error_promo_code'] = reward_status['error']
            return False
        order._update_programs_and_rewards()
        if order.carrier_id.free_over and not reward.program_id.is_payment_program:
            # update shiping cost if it's `free_over` and reward isn't eWallet or gift card
            # will call `_update_programs_and_rewards` again, updating applied eWallet/gift cards
            res = order.carrier_id.rate_shipment(order)
            if res.get('success'):
                order.set_delivery_line(order.carrier_id, res['price'])
            else:
                order._remove_delivery_line()
        return True

    @route()
    def cart_update_json(self, *args, set_qty=None, **kwargs):
        # When a reward line is deleted we remove it from the auto claimable rewards
        if set_qty == 0:
            request.update_context(website_sale_loyalty_delete=True)
            # We need to update the website since `get_sale_order` is called on the website
            # and does not follow the request's context
            request.website = request.website.with_context(website_sale_loyalty_delete=True)
        return super().cart_update_json(*args, set_qty=set_qty, **kwargs)

```

## File: controllers\payment.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _
from odoo.exceptions import ValidationError

from odoo.addons.website_sale.controllers import payment


class PaymentPortal(payment.PaymentPortal):

    def _validate_transaction_for_order(self, transaction, sale_order):
        """Update programs & rewards before finalizing transaction.

        :param payment.transaction transaction: The payment transaction
        :param int order_id: The id of the sale order to pay
        :raise: ValidationError if the order amount changed after updating rewards
        """
        super()._validate_transaction_for_order(transaction, sale_order)
        if sale_order.exists():
            initial_amount = sale_order.amount_total
            sale_order._update_programs_and_rewards()
            if sale_order.currency_id.compare_amounts(sale_order.amount_total, initial_amount):
                raise ValidationError(
                    _(
                        "Cannot process payment: applied reward was changed or has expired.\n"
                        "Please refresh the page and try again."
                    )
                )

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import delivery
from . import main
from . import payment

```

## File: data\product_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="loyalty.gift_card_product_50" model="product.product">
        <field name="is_published" eval="True"/>
    </record>
</odoo>

```

## File: models\loyalty_card.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models

class LoyaltyCard(models.Model):
    _inherit = 'loyalty.card'

    def action_coupon_share(self):
        self.ensure_one()
        return self.env['coupon.share'].create_share_action(coupon=self)

```

## File: models\loyalty_program.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models

class LoyaltyProgram(models.Model):
    _name = 'loyalty.program'
    _inherit = ['loyalty.program', 'website.multi.mixin']

    ecommerce_ok = fields.Boolean("Available on Website", default=True)

    def action_program_share(self):
        self.ensure_one()
        return self.env['coupon.share'].create_share_action(program=self)

```

## File: models\loyalty_rule.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError

class LoyaltyRule(models.Model):
    _inherit = 'loyalty.rule'

    website_id = fields.Many2one(related='program_id.website_id', store=True)

    # NOTE: is this sufficient?
    @api.constrains('code', 'website_id')
    def _constrains_code(self):
        #Programs with the same code are allowed to coexist as long
        # as they are not both accessible from a website.
        with_code = self.filtered(lambda r: r.mode == 'with_code')
        mapped_codes = with_code.mapped('code')
        read_result = self.env['loyalty.rule'].search_read(
            [('website_id', 'in', [False] + [w.id for w in self.website_id]),
            ('mode', '=', 'with_code'), ('code', 'in', mapped_codes),
            ('id', 'not in', with_code.ids)],
            fields=['code', 'website_id']) + [{'code': p.code, 'website_id': p.website_id} for p in with_code]
        existing_codes = set()
        for res in read_result:
            website_checks = (res['website_id'], False) if res['website_id'] else (False,)
            for website in website_checks:
                val = (res['code'], website)
                if val in existing_codes:
                    raise ValidationError(_('The promo code must be unique.'))
                existing_codes.add(val)
        # Prevent coupons and programs from sharing a code
        if self.env['loyalty.card'].search_count([('code', 'in', mapped_codes)]):
            raise ValidationError(_('A coupon with the same code was found.'))

```

## File: models\sale_order.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict
from datetime import timedelta

from odoo import api, fields, models
from odoo.exceptions import UserError
from odoo.http import request
from odoo.osv import expression


class SaleOrder(models.Model):
    _inherit = 'sale.order'

    # List of disabled rewards for automatic claim
    disabled_auto_rewards = fields.Many2many("loyalty.reward", relation="sale_order_disabled_auto_rewards_rel")

    def _get_program_domain(self):
        res = super()._get_program_domain()
        # Replace `sale_ok` leaf with `ecommerce_ok` if order is linked to a website
        if self.website_id:
            for idx, leaf in enumerate(res):
                if leaf[0] != 'sale_ok':
                    continue
                res[idx] = ('ecommerce_ok', '=', True)
                return expression.AND([res, [('website_id', 'in', (self.website_id.id, False))]])
        return res

    def _get_trigger_domain(self):
        res = super()._get_trigger_domain()
        # Replace `sale_ok` leaf with `ecommerce_ok` if order is linked to a website
        if self.website_id:
            for idx, leaf in enumerate(res):
                if leaf[0] != 'program_id.sale_ok':
                    continue
                res[idx] = ('program_id.ecommerce_ok', '=', True)
                return expression.AND([res, [('program_id.website_id', 'in', (self.website_id.id, False))]])
        return res

    def _try_pending_coupon(self):
        if not request:
            return False

        pending_coupon_code = request.session.get('pending_coupon_code')
        if pending_coupon_code:
            status = self._try_apply_code(pending_coupon_code)
            if 'error' not in status: # Returns an array if everything went right
                request.session.pop('pending_coupon_code')
                if len(status) == 1:
                    coupon, rewards = next(iter(status.items()))
                    if len(rewards) == 1 and not rewards.multi_product:
                        self._apply_program_reward(rewards, coupon)
            return status
        return True

    def _update_programs_and_rewards(self):
        for order in self:
            order._try_pending_coupon()
        return super()._update_programs_and_rewards()

    def _auto_apply_rewards(self):
        """
        Tries to auto apply claimable rewards.

        It must answer to the following rules:
         - Must not be from a nominative program
         - The reward must be the only reward of the program
         - The reward may not be a multi product reward

        Returns True if any reward was claimed else False
        """
        self.ensure_one()

        claimed_reward_count = 0
        claimable_rewards = self._get_claimable_rewards()
        for coupon, rewards in claimable_rewards.items():
            if (
                len(coupon.program_id.reward_ids) != 1
                or coupon.program_id.is_nominative
                or (rewards.reward_type == 'product' and rewards.multi_product)
                or rewards in self.disabled_auto_rewards
                or rewards in self.order_line.reward_id
            ):
                continue

            try:
                res = self._apply_program_reward(rewards, coupon)
                if 'error' not in res:
                    claimed_reward_count += 1
            except UserError:
                pass

        return bool(claimed_reward_count)

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
            grouped_order_lines = defaultdict(lambda: self.env['sale.order.line'])
            for line in order.order_line:
                if line.reward_id and line.coupon_id:
                    grouped_order_lines[(line.reward_id, line.coupon_id, line.reward_identifier_code)] |= line
            new_lines = self.env['sale.order.line']
            for lines in grouped_order_lines.values():
                if lines.reward_id.reward_type != 'discount':
                    continue
                new_lines += self.env['sale.order.line'].new({
                    'product_id': lines[0].product_id.id,
                    'tax_id': False,
                    'price_unit': sum(lines.mapped('price_unit')),
                    'price_subtotal': sum(lines.mapped('price_subtotal')),
                    'price_total': sum(lines.mapped('price_total')),
                    'discount': 0.0,
                    'name': lines[0].name_short if lines.reward_id.reward_type != 'product' else lines[0].name,
                    'product_uom_qty': 1,
                    'product_uom': lines[0].product_uom.id,
                    'order_id': order.id,
                    'is_reward_line': True,
                    'coupon_id': lines.coupon_id,
                    'reward_id': lines.reward_id,
                })
            if new_lines:
                order.website_order_line += new_lines

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

    def get_promo_code_success_message(self, delete=True):
        if not request.session.get('successful_code'):
            return False
        code = request.session.get('successful_code')
        if delete:
            request.session.pop('successful_code')
        return code

    def _set_delivery_method(self, *args, **kwargs):
        super()._set_delivery_method(*args, **kwargs)
        self._update_programs_and_rewards()

    def _remove_delivery_line(self):
        super()._remove_delivery_line()
        self._update_programs_and_rewards()

    def _cart_update(self, product_id, line_id=None, add_qty=0, set_qty=0, **kwargs):
        line = self.order_line.filtered(lambda sol: sol.product_id.id == product_id)[:1]
        reward_id = line.reward_id
        if set_qty == 0 and line.coupon_id and reward_id and reward_id.reward_type == 'discount':
            # Force the deletion of the line even if it's a temporary record created by new()
            line_id = line.id
        res = super()._cart_update(
            product_id, line_id=line_id, add_qty=add_qty, set_qty=set_qty, **kwargs
        )
        self._update_programs_and_rewards()
        self._auto_apply_rewards()
        return res

    def _get_free_shipping_lines(self):
        self.ensure_one()
        return self.order_line.filtered(lambda l: l.reward_id.reward_type == 'shipping')

    def _allow_nominative_programs(self):
        if not request or not hasattr(request, 'website'):
            return super()._allow_nominative_programs()
        return not request.website.is_public_user() and super()._allow_nominative_programs()

    @api.autovacuum
    def _gc_abandoned_coupons(self, *args, **kwargs):
        """Remove coupons from abandonned ecommerce order."""
        ICP = self.env['ir.config_parameter']
        validity = ICP.get_param('website_sale_coupon.abandonned_coupon_validity', 4)
        validity = fields.Datetime.to_string(fields.datetime.now() - timedelta(days=int(validity)))
        so_to_reset = self.env['sale.order'].search([
            ('state', '=', 'draft'),
            ('write_date', '<', validity),
            ('website_id', '!=', False),
            ('applied_coupon_ids', '!=', False),
        ])
        so_to_reset.applied_coupon_ids = False
        for so in so_to_reset:
            so._update_programs_and_rewards()

    def _get_claimable_and_showable_rewards(self):
        self.ensure_one()
        res = self._get_claimable_rewards()
        loyality_cards = self.env['loyalty.card'].search([
            ('partner_id', '=', self.partner_id.id),
            ('program_id', 'any', self._get_program_domain()),
            '|',
                ('program_id.trigger', '=', 'with_code'),
                '&', ('program_id.trigger', '=', 'auto'), ('program_id.applies_on', '=', 'future'),
        ])
        total_is_zero = self.currency_id.is_zero(self.amount_total)
        global_discount_reward = self._get_applied_global_discount()
        for coupon in loyality_cards:
            points = self._get_real_points_for_coupon(coupon)
            for reward in coupon.program_id.reward_ids:
                if (
                    reward.is_global_discount
                    and global_discount_reward
                    and self._best_global_discount_already_applied(global_discount_reward, reward)
                ):
                    continue
                if reward.reward_type == 'discount' and total_is_zero:
                    continue
                if coupon.expiration_date and coupon.expiration_date < fields.Date.today():
                    continue
                if points >= reward.required_points:
                    if coupon in res:
                        res[coupon] |= reward
                    else:
                        res[coupon] = reward
        return res

    def _cart_find_product_line(self, product_id, line_id=None, **kwargs):
        """ Override to filter out reward lines from the cart lines.

        These are handled by the _update_programs_and_rewards and _auto_apply_rewards methods.
        """
        lines = super()._cart_find_product_line(product_id, line_id, **kwargs)
        lines = lines.filtered(lambda l: not l.is_reward_line) if not line_id else lines
        return lines

```

## File: models\sale_order_line.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict
from odoo import models

class SaleOrderLine(models.Model):
    _inherit = 'sale.order.line'

    def _show_in_cart(self):
        # Hide discount lines from website_order_line, see `order._compute_website_order_line`
        return self.reward_id.reward_type != 'discount' and super()._show_in_cart()

    def unlink(self):
        if self.env.context.get('website_sale_loyalty_delete', False):
            disabled_rewards_per_order = defaultdict(lambda: self.env['loyalty.reward'])
            for line in self:
                if line.reward_id:
                    disabled_rewards_per_order[line.order_id] |= line.reward_id
            for order, rewards in disabled_rewards_per_order.items():
                order.disabled_auto_rewards += rewards
        return super().unlink()

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import loyalty_card
from . import loyalty_program
from . import loyalty_rule
from . import sale_order_line
from . import sale_order

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_coupon_share_manager,access_coupon_share_manager,model_coupon_share,sales_team.group_sale_manager,1,1,1,0

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path fill-rule="evenodd" clip-rule="evenodd" d="M15.724 6.397C16.377 4.94 17.852 4 19.481 4h11.037c1.63 0 3.104.94 3.757 2.397L37.236 13H41.9c2.46 0 4.367 2.099 4.07 4.481l-3.106 25C42.613 44.49 40.866 46 38.793 46H11.207c-2.074 0-3.82-1.51-4.07-3.519l-3.107-25C3.734 15.1 5.64 13 8.1 13h4.663l2.961-6.603ZM32.917 13H17.082c0-.56.123-1.134.39-1.691l.956-2C19.102 7.9 20.551 7 22.144 7h5.711c1.593 0 3.042.9 3.716 2.308l.957 2c.266.558.39 1.132.39 1.692Z" fill="#F9464C"/><path fill-rule="evenodd" clip-rule="evenodd" d="M8.514 45.016a3.963 3.963 0 0 1-1.377-2.535l-3.107-25C3.734 15.1 5.64 13 8.1 13h4.663l2.961-6.603C16.377 4.94 17.852 4 19.481 4h11.037c1.63 0 3.104.94 3.757 2.397l2.59 5.777C35.5 28.256 23.848 41.405 8.515 45.016ZM17.082 13h15.835c0-.56-.123-1.134-.39-1.691l-.956-2C30.897 7.9 29.448 7 27.855 7h-5.711c-1.593 0-3.042.9-3.716 2.308l-.956 2a3.904 3.904 0 0 0-.39 1.692Z" fill="#FC868B"/><path d="M17.142 20.864a4 4 0 0 1 4.899-2.828l11.59 3.106a4 4 0 0 1 2.83 4.899l-3.107 11.59a4 4 0 0 1-4.899 2.83l-11.59-3.107a4 4 0 0 1-2.83-4.899l3.107-11.59Z" fill="#fff"/><path d="M23.135 28.615A2.828 2.828 0 1 1 26.28 26.8l3.3-1.906c.492-.284 1.056-.23 1.258.122l.919 1.59-5.188 2.996a2.828 2.828 0 1 1-2.726 1.574l-3.666 2.116c-.492.284-1.055.23-1.258-.122L18 31.58l5.135-2.965Z" fill="#F9464C"/></svg>

```

## File: static\src\js\checkout.js

```javascript
import WebsiteSaleCheckout from '@website_sale/js/checkout';

WebsiteSaleCheckout.include({
    /**
     * @override
     */
    _updateCartSummary(result) {
        this._super.apply(this, arguments);
        if (result.amount_delivery_discounted) {
            // Update discount of the order
            const cart_summary_shipping_reward = document.querySelector(
                '[data-reward-type="shipping"]'
            );
            if (cart_summary_shipping_reward) {
                cart_summary_shipping_reward.innerHTML = result.amount_delivery_discounted;
            }
        }
        if (result.discount_reward_amounts) {
            const cart_summary_discount_rewards = document.querySelectorAll(
                '[data-reward-type=discount]'
            );
            if (cart_summary_discount_rewards.length !== result.discount_reward_amounts.length) {
                // refresh cart summary to sync number of discount items
                location.reload();
            } else {
                cart_summary_discount_rewards.forEach(
                    (el, i) => (el.innerHTML = result.discount_reward_amounts[i])
                );
            }
        }
    },
});

```

## File: static\src\js\coupon_toaster_widget.js

```javascript
import publicWidget from '@web/legacy/js/public/public_widget';

const CouponToasterWidget = publicWidget.Widget.extend({
    selector: '.coupon-message',

    init() {
        this._super(...arguments);
        this.notification = this.bindService("notification");
    },

    start() {
        let options = {};
        const contentEl = this.el.querySelector('.coupon-message-content');
        const titleEl = this.el.querySelector('.coupon-message-title');
        let message = null;

        if (contentEl) {
            message = contentEl.innerHTML;
            if (titleEl) {
                Object.assign(options, { title: titleEl.innerHTML });
            }
        } else if (titleEl) {
            message = titleEl.innerHTML;
        }

        if (this.el.classList.contains('coupon-info-message')) {
            this.notification.add(message, Object.assign({type: 'success'}, options));
        } else if (this.el.classList.contains('coupon-error-message')) {
            this.notification.add(message, Object.assign({type: 'danger'}, options));
        } else if (this.el.classList.contains('coupon-warning-message')) {
            this.notification.add(message, Object.assign({type: 'warning'}, options));
        }

        return this._super(...arguments);
    },
});

publicWidget.registry.CouponToasterWidget = CouponToasterWidget;

export default CouponToasterWidget;

```

## File: static\src\js\portal_loyalty_card.xml

```xml
<templates>

    <t t-inherit="loyalty.portal_loyalty_card_dialog" t-inherit-mode="extension">
        <div name="history_lines" position="after">
            <div
                t-if="props.program.program_type == 'loyalty'"
                t-foreach="props.rewards"
                t-as="reward"
                t-key="reward_index"
                class="mb-2 mt-2"
            >
                <div class="d-flex border rounded align-items-center justify-content-center p-2">
                    <div class="flex-fill fs-6">
                        <t t-out="reward.description"/>
                    </div>
                    <span class="text-primary" t-out="reward.points"/>
                </div>
            </div>
            <div t-if="props.rewards.length > 0" class="d-flex justify-content-center">
                <a
                    t-if="props.program.program_type == 'loyalty'"
                    type="button"
                    href="/shop"
                    class="btn btn-primary"
                >
                    Claim
                </a>
            </div>
        </div>
    </t>

</templates>

```

## File: static\src\js\website_sale_gift_card.js

```javascript
/** @odoo-module **/

import publicWidget from '@web/legacy/js/public/public_widget';
import { browser } from '@web/core/browser/browser';

publicWidget.registry.WebsiteSaleGiftCardCopy = publicWidget.Widget.extend({
    selector: '.o_purchased_gift_card',
    events: {
        "click .copy-to-clipboard": "_onClickCopyToClipboard",
    },
    async _onClickCopyToClipboard(ev) {
        const textValue = ev.target.dataset.clipboardText;
        await browser.navigator.clipboard.writeText(textValue);
    },
});

```

## File: views\loyalty_card_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="loyalty_card_view_tree_inherit_website_sale_loyalty" model="ir.ui.view">
        <field name="name">loyalty.card.view.list.inherit.website.sale.loyalty</field>
        <field name="model">loyalty.card</field>
        <field name="inherit_id" ref="loyalty.loyalty_card_view_tree"/>
        <field name="arch" type="xml">
            <button name="action_coupon_send" position="after">
                <button name="action_coupon_share" string="Share" type="object" icon="fa-share-alt"/>
            </button>
        </field>
    </record>
</odoo>

```

## File: views\loyalty_program_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="loyalty_program_view_form_inherit_website_sale_loyalty" model="ir.ui.view">
        <field name="name">loyalty.program.view.form.inherit.website.sale.loyalty</field>
        <field name="model">loyalty.program</field>
        <field name="inherit_id" ref="sale_loyalty.loyalty_program_view_form_inherit_sale_loyalty"/>
        <field name="arch" type="xml">
            <xpath expr="//label[@for='available_on']" position="attributes">
                <attribute name="invisible">0</attribute>
            </xpath>
            <xpath expr="//div[@id='o_loyalty_program_availabilities']" position="attributes">
                <attribute name="invisible">0</attribute>
            </xpath>
            <xpath expr="//div[@id='o_loyalty_program_availabilities']" position="inside">
                <span class="d-inline-flex text-break">
                    <field name="ecommerce_ok"/>
                    <label for="ecommerce_ok" string="Website"/>
                </span>
            </xpath>
            <xpath expr="//div[@id='o_loyalty_program_availabilities']" position="after">
                <field name="website_id" invisible="not ecommerce_ok" options="{'no_create': True}" groups="website.group_multi_website" placeholder="All websites"/>
            </xpath>
        </field>
    </record>

    <record id="loyalty_program_view_tree_inherit_website_sale_loyalty" model="ir.ui.view">
        <field name="name">loyalty.program.view.list.inherit.website.sale.loyalty</field>
        <field name="model">loyalty.program</field>
        <field name="inherit_id" ref="loyalty.loyalty_program_view_tree"/>
        <field name="arch" type="xml">
            <field name="coupon_count_display" position="after">
                <field name="website_id" options="{'no_create': True}" groups="website.group_multi_website"/>
            </field>
            <field name="company_id" position="after">
                <button name="action_program_share" string="Share" type="object" icon="fa-share-alt" invisible="program_type != 'promo_code'"/>
            </field>
        </field>
    </record>
</odoo>

```

## File: views\snippets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="snippet_options" inherit_id="website.snippet_options" name="Coupon Snippet Options">
    <xpath expr="." position="inside">
        <div data-selector="main:has(.oe_website_sale .wizard)" data-page-options="true" groups="website.group_website_designer" data-no-check="true">
            <we-checkbox string="Show Discount in Subtotal"
                         data-customize-website-views="website_sale_loyalty.cart_discount"
                         data-no-preview="true"
                         data-reload="/"/>
        </div>
    </xpath>
</template>

</odoo>

```

## File: views\website_sale_loyalty_menus.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <menuitem id="menu_loyalty"
        name="Loyalty"
        parent="website_sale.menu_ecommerce"
        groups="sales_team.group_sale_manager"
        sequence="4">

        <menuitem id="menu_discount_loyalty_type_config"
            action="loyalty.loyalty_program_discount_loyalty_action"/>
        <menuitem id="menu_gift_ewallet_type_config"
            action="loyalty.loyalty_program_gift_ewallet_action"/>

    </menuitem>

</odoo>

```

## File: views\website_sale_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="sale_coupon_result" inherit_id="website_sale.coupon_form">
        <xpath expr="//form[@name='coupon_code']//input[@name='promo']" position="attributes">
            <attribute name="placeholder">Gift card or discount code...</attribute>
        </xpath>
        <xpath expr="//t[@name='code_not_available']" position="replace"/>
    </template>

    <template id="modify_code_form" inherit_id="website_sale.total" name="Loyalty, coupon, gift card">
        <xpath expr="//div[@id='cart_total']//table/tr[last()]" position="after">
            <tr t-if="not hide_promotions" class="oe_website_sale_gift_card">
                <td colspan="3" class="text-center text-xl-end border-0 p-0">
                    <span class=''>
                        <t t-if="request.params.get('code_not_available')">
                            <div class="alert alert-danger text-start mt16" role="alert">
                                Invalid or expired promo code.
                            </div>
                        </t>
                        <t t-if="website_sale_order.get_promo_code_error(delete=False)">
                            <div class="alert alert-danger text-start mt16" role="alert">
                                <t t-esc="website_sale_order.get_promo_code_error()"/>
                            </div>
                        </t>
                        <t t-if="website_sale_order">
                            <t t-if="website_sale_order.get_promo_code_success_message(delete=False)">
                                <div class="alert alert-success text-start mt16" role="alert">
                                    You have successfully applied the following code: <strong t-esc="website_sale_order.get_promo_code_success_message()"/>
                                </div>
                            </t>
                            <t t-foreach="website_sale_order._get_claimable_and_showable_rewards().items()" t-as="coupon_reward">
                                <t t-set="coupon" t-value="coupon_reward[0]"/>
                                <t t-set="rewards" t-value="coupon_reward[1]"/>
                                <t t-foreach="rewards" t-as="reward">
                                    <form t-att-action="'/shop/claimreward%s' % (redirect and '?r=' + redirect or '')"
                                        method="post" name="claim_reward">
                                        <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
                                        <input type="hidden" name="reward_id" t-att-value="reward.id"/>
                                        <input type="hidden" name="code" t-att-value="coupon.code"/>
                                        <div class="alert alert-success text-start mt16" role="alert">
                                            <div class="d-flex flex-row">
                                                <div class="flex-grow-1 text-break">
                                                    <t t-set="program" t-value="reward.program_id"/>
                                                    <t t-set="points" t-value="coupon._format_points(website_sale_order._get_real_points_for_coupon(coupon))"/>
                                                    <t t-if="program.program_type not in ['ewallet', 'promo_code', 'loyalty'] and (program.trigger == 'with_code' or (program.trigger == 'auto' and program.applies_on == 'future'))">
                                                        <t t-if="program.program_type == 'gift_card'">
                                                            <strong t-esc="reward.description"/>
                                                            <strong> - </strong>
                                                            <strong t-esc="points"/>
                                                        </t>
                                                        <t t-elif="program.program_type == 'coupons'">
                                                            <strong>Coupons - </strong><strong t-esc="reward.description"/>
                                                        </t>
                                                        <t t-elif="program.trigger == 'auto' and program.applies_on == 'future'">
                                                            <strong t-esc="reward.description"/>
                                                        </t>
                                                        <div class="text-muted d-md-block small">
                                                            <span t-if="coupon and not coupon.program_id.is_nominative">
                                                                Code:
                                                                <t t-esc="coupon.code[-4:].rjust(14, '&#8902;')"/>
                                                            </span>
                                                            <t t-if="coupon.expiration_date">
                                                                <br/>
                                                                <span>
                                                                    Expired Date:
                                                                    <t t-esc="coupon.expiration_date"/>
                                                                </span>
                                                            </t>
                                                        </div>
                                                    </t>
                                                    <t t-elif="program.program_type == 'promo_code'">
                                                        <strong t-esc="reward.description"/>
                                                    </t>
                                                    <t t-else="">
                                                        <strong t-esc="reward.description"/>
                                                        <div t-if="program.portal_visible">
                                                            <t t-if="not program.is_nominative">
                                                                <span t-out="points"/>
                                                            </t>
                                                            <t t-else="">
                                                                <span>You have <t t-out="points"/></span>
                                                                <span t-if="program.program_type == 'ewallet'"> in your ewallet</span>
                                                                <t t-else="">
                                                                    <br/>
                                                                    <span>Costs <t t-out="coupon._format_points(reward.required_points)"/></span>
                                                                </t>
                                                            </t>
                                                        </div>
                                                    </t>
                                                    <select
                                                        t-if="reward.multi_product"
                                                        class="o_select w-100 form-select form-select-sm css_attribute_select"
                                                        name="product_id"
                                                    >
                                                        <option
                                                            t-foreach="reward.reward_product_ids"
                                                            t-as="product"
                                                            t-att-value="product.id"
                                                        >
                                                            <t t-out="product.display_name"/>
                                                        </option>
                                                    </select>
                                                </div>
                                                <div class="justify-content-end">
                                                    <a class="btn btn-primary a-submit" href="#" role="button">
                                                        <t t-if="reward.program_id.program_type == 'ewallet'">
                                                            Pay with eWallet
                                                        </t>
                                                        <t t-elif="(reward.program_id.trigger == 'with_code' and reward.program_id.program_type != 'promo_code') or (reward.program_id.trigger == 'auto' and reward.program_id.applies_on == 'future')">
                                                            Use
                                                        </t>
                                                        <t t-else="">Claim</t>
                                                    </a>
                                                </div>
                                            </div>
                                        </div>
                                    </form>
                                </t>
                            </t>
                        </t>
                    </span>
                </td>
            </tr>
        </xpath>
    </template>

    <template id="website_sale_coupon_cart_hide_qty" inherit_id="website_sale.cart_lines">
        <xpath expr="//del" position="attributes">
            <attribute name="t-if">not line.is_reward_line</attribute>
        </xpath>
    </template>

    <template id="layout" inherit_id="website.layout">
        <xpath expr="//*[@id='wrapwrap']" position="inside">
            <t t-set="coupon_error" t-value="request.params.get('coupon_error')"/>
            <t t-set="pending_coupon_code" t-value="request.session.get('pending_coupon_code')"/>
            <t t-if="coupon_error and pending_coupon_code">
                <div t-attf-class="d-none coupon-message coupon-{{ request.params.get('coupon_error_type', 'error') }}-message">
                    <span class="coupon-message-title">Could not apply the promo code: <t t-out="pending_coupon_code"/></span>
                    <span class="coupon-message-content" t-out="coupon_error"/>
                </div>
            </t>
            <t t-set="notify_coupon" t-value="request.params.get('notify_coupon')"/>
            <div t-if="notify_coupon" class="d-none coupon-message coupon-info-message">
                <span class="coupon-message-content">The following promo code was applied on your order: <t t-out="notify_coupon"/></span>
            </div>
        </xpath>
    </template>

    <template id="cart_discount" name="Show Discount in Subtotal" active="False" inherit_id="website_sale.total">
        <xpath expr="//tr[@id='order_total_untaxed']" position="before">
            <tr t-if="website_sale_order and website_sale_order.reward_amount">
            <td class="text-end border-0 text-muted" title="Discounted amount">Discount:</td>
            <td class="text-xl-end border-0 text-muted">
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
        <xpath expr="//t[@t-set='force_coupon']" position="after">
            <t t-set="_placeholder" t-value="'Discount code or gift card'"/>
        </xpath>
    </template>

    <template id="cart_summary" name="Payment" inherit_id="website_sale.cart_product_price">
        <xpath expr="//span" position="attributes">
            <attribute name="t-att-data-reward-type">line.reward_id.reward_type</attribute>
        </xpath>
    </template>

    <template id="cart_line_product_no_link" inherit_id="website_sale.cart_line_product_link">
        <xpath expr="//a" position="replace">
            <t t-if="line.is_reward_line">
                <strong t-field="line.name"/>
                <t t-call="sale_loyalty.used_gift_card"/>
            </t>
            <t t-else="">$0</t>
        </xpath>
    </template>

    <template id="cart_summary_inherit_website_gift_card_sale" inherit_id="website_sale.checkout_layout">
        <xpath expr="//td[@name='website_sale_cart_summary_product_name']/h6" position="after">
            <t t-call="sale_loyalty.used_gift_card"/>
        </xpath>
    </template>

    <template id="website_sale_purchased_gift_card" inherit_id="website_sale.confirmation" >
        <xpath expr="//div[@id='oe_structure_website_sale_confirmation_2']" position="after">
            <t t-call="sale_loyalty.sale_purchased_gift_card"/>
        </xpath>
    </template>
</odoo>

```

## File: wizard\coupon_share.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from werkzeug.urls import url_encode

from odoo import fields, models, api, _
from odoo.exceptions import UserError, ValidationError


class CouponShare(models.TransientModel):
    _name = 'coupon.share'
    _description = 'Create links that apply a coupon and redirect to a specific page'

    def _get_default_website_id(self):
        program_website_id = self.env['loyalty.program'].browse(self.env.context.get('default_program_id')).website_id
        if program_website_id:
            return program_website_id
        else:
            Website = self.env['website']
            websites = Website.search([])
            return len(websites) == 1 and websites or Website

    website_id = fields.Many2one('website', required=True, default=_get_default_website_id)
    coupon_id = fields.Many2one('loyalty.card', domain="[('program_id', '=', program_id)]")
    program_id = fields.Many2one('loyalty.program', required=True, domain=[
        '|', ('program_type', '=', 'coupons'), # All coupons programs
        '|', ('trigger', '=', 'with_code'), # All programs that require a code
             ('rule_ids.code', '!=', False), # All programs that can not trigger without a code
    ])
    program_website_id = fields.Many2one('website', string='Program Website', related='program_id.website_id')

    promo_code = fields.Char(compute='_compute_promo_code')
    share_link = fields.Char(compute='_compute_share_link')
    redirect = fields.Char(required=True, default='/shop')

    @api.constrains('coupon_id', 'program_id')
    def _check_program(self):
        if self.filtered(lambda record: not record.coupon_id and record.program_id.program_type == 'coupons'):
            raise ValidationError(_("A coupon is needed for coupon programs."))

    @api.constrains('website_id', 'program_id')
    def _check_website(self):
        if self.filtered(lambda record: record.program_website_id and record.program_website_id != record.website_id):
            raise ValidationError(_("The shared website should correspond to the website of the program."))

    @api.depends('coupon_id.code', 'program_id.rule_ids.code')
    def _compute_promo_code(self):
        for record in self:
            record.promo_code = record.coupon_id.code or record.program_id.rule_ids.filtered('code')[:1].code

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
            'name': _('Share'),
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
            'name': _('Share %s', self.env["loyalty.program"]._program_items_name().get((program or coupon).program_type, "")),
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'coupon.share',
            'target': 'new',
            'context': {
                'default_program_id': program and program.id or coupon.program_id.id,
                'default_coupon_id': coupon and coupon.id or None,
            }
        }

```

## File: wizard\coupon_share_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="coupon_share_view_form" model="ir.ui.view">
        <field name="name">coupon.share.form</field>
        <field name="model">coupon.share</field>
        <field name="arch" type="xml">
            <form string="Share Loyalty Card" edit="1" create="0">
                <field name="website_id" invisible="1"/>
                <field name="program_website_id" invisible="1"/>
                <group>
                    <div colspan="2">
                        <p>
                            You can share this promotion with your customers.
                            It will be applied at checkout when the customer uses this link.
                        </p>
                    </div>
                    <field name="share_link" widget="CopyClipboardURL" no_label="1"/>
                </group>
                <group invisible="id">
                    <group>
                        <field name="website_id" groups="website.group_multi_website" widget="selection"
                            invisible="program_website_id"/>
                        <field name="redirect"/>
                    </group>
                </group>
                <footer>
                    <button string="Done" class="btn-primary" special="cancel" data-hotkey="x"/>
                    <button string="Generate Short Link" class="btn-secondary" type="object" name="action_generate_short_link"
                            invisible="not website_id or id" data-hotkey="g"/>
                </footer>
            </form>
        </field>
    </record>
</odoo>

```

## File: wizard\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="res_config_settings_view_form_inherit_website_sale_loyalty" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.website.sale.loyalty</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="website_sale.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <setting id="promotion_coupon_programs" position="inside">
                <div class="content-group">
                    <div class="mt8" invisible="not module_loyalty">
                        <button name="%(loyalty.loyalty_program_discount_loyalty_action)d"
                            type="action"
                            string="Loyalty Programs"
                            icon="oi-arrow-right"
                            class="btn-link"/>
                    </div>
                </div>
            </setting>
        </field>
    </record>

</odoo>

```

## File: wizard\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import coupon_share

```

