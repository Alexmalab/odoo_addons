# Odoo Module: website_sale_gift_card

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
    'name': "Website Sale Gift Card",
    'summary': "Use gift card in eCommerce",
    'description': """Integrate gift card mechanism in your ecommerce.""",
    'category': 'Website/Website',
    'version': '1.0',
    'depends': ['website_sale', 'sale_gift_card'],
    'data': [
        'views/template.xml',
        'views/gift_card_views.xml',
        'views/gift_card_menus.xml',
        ],
    'demo': [
        'data/product_demo.xml',
    ],
    'assets': {
        'web.assets_frontend': [
            'website_sale_gift_card/static/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\gift_card_controller.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http
from odoo.http import request
from odoo.addons.website_sale.controllers import main


class GiftCardController(main.WebsiteSale):

    @http.route('/shop/pay_with_gift_card', type='http', methods=['POST'], website=True, auth='public')
    def add_gift_card(self, gift_card_code, **post):
        gift_card = request.env["gift.card"].sudo().search([('code', '=', gift_card_code.strip())], limit=1)
        order = request.env['website'].get_current_website().sale_get_order()
        gift_card_status = order._pay_with_gift_card(gift_card)
        return request.redirect('/shop/payment' + '?keep_carrier=1' + ('&gift_card_error=%s' % gift_card_status if gift_card_status else ''))

    @http.route()
    def shop_payment(self, **post):
        order = request.website.sale_get_order()
        res = super().shop_payment(**post)
        order._recompute_gift_card_lines()
        return res

    @http.route(['/shop/cart'], type='http', auth="public", website=True)
    def cart(self, **post):
        order = request.website.sale_get_order()
        order._recompute_gift_card_lines()
        return super().cart(**post)

    def _get_shop_payment_values(self, order, **kwargs):
        values = super()._get_shop_payment_values(order, **kwargs)
        values['allow_pay_with_gift_card'] = True
        return values

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import gift_card_controller

```

## File: data\product_demo.xml

```xml
<?xml version="1.0"?>
<odoo>
   <record id="gift_card.gift_card_product_50" model="product.product">
        <field name="is_published" eval="True" />
    </record>
</odoo>

```

## File: models\gift_card.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields


class GiftCard(models.Model):
    _name = "gift.card"
    _inherit = ['website.multi.mixin', 'gift.card']

    website_id = fields.Many2one('website', related='buy_line_id.order_id.website_id', store=True, readonly=False)

    def can_be_used(self):
        website = self.env['website'].get_current_website()
        return super(GiftCard, self).can_be_used() and self.website_id.id in [website.id, False]

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class SaleOrder(models.Model):
    _inherit = "sale.order"

    def _compute_website_order_line(self):
        super()._compute_website_order_line()
        for order in self:
            order.website_order_line = order.website_order_line.sorted(lambda ol: ol.gift_card_id.id)

    def _compute_cart_info(self):
        super()._compute_cart_info()
        for order in self:
            gift_card_payment_lines = order.website_order_line.filtered('gift_card_id')
            order.cart_quantity -= int(sum(gift_card_payment_lines.mapped('product_uom_qty')))

    def _cart_update(self, product_id=None, line_id=None, add_qty=0, set_qty=0, **kwargs):
        res = super()._cart_update(product_id=product_id, line_id=line_id, add_qty=add_qty, set_qty=set_qty, **kwargs)
        self._recompute_gift_card_lines()
        return res


class SaleOrderLine(models.Model):
    _inherit = "sale.order.line"

    def _build_gift_card(self):
        gift_card = super()._build_gift_card()
        gift_card['website_id'] = self.order_id.website_id.id
        return gift_card

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import gift_card
from . import sale_order

```

## File: static\src\js\website_sale_gift_card.js

```javascript
odoo.define('website_sale_gift_card.website_sale_gift_card', function (require) {
    'use strict';

    var publicWidget = require('web.public.widget');

    publicWidget.registry.WebsiteSaleGiftCard = publicWidget.Widget.extend({
            selector: '.oe_website_sale_gift_card',
            events: {
                'click .js_show_gift_card': '_onShowGiftCardClick',
            },
            /**
             * @private
             * @param {Event} ev
             */
            _onShowGiftCardClick: function (ev) {
                $(ev.currentTarget).hide();
                this.$('.gift_card_form').removeClass('d-none');
            }
        }
    )
    publicWidget.registry.WebsiteSaleGiftCardCopy = publicWidget.Widget.extend({
        selector: '.o_purchased_gift_card',
        /**
         * @override
         */
        start: function () {
            new ClipboardJS(this.$el.find('.copy-to-clipboard')[0]);
        }
    })
})

```

## File: views\gift_card_menus.xml

```xml
<?xml version="1.0"?>
<odoo>
    <menuitem id="website_product_gift_card_menu"
        name="Gift Cards"
        parent="website_sale.menu_ecommerce_settings"
        groups="sales_team.group_sale_manager"
        action="gift_card.gift_card_action"/>
</odoo>

```

## File: views\gift_card_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="gift_card_view_form" model="ir.ui.view">
        <field name="name">gift.card.form Website</field>
        <field name="model">gift.card</field>
        <field name="inherit_id" ref="gift_card.gift_card_view_form" />
        <field name="arch" type="xml">
            <field name="partner_id">
                <field name="website_id" groups="website.group_multi_website" options="{'no_create': True}"/>
            </field>
        </field>
    </record>

    <!-- Searching -->
    <record id="gift_card_view_search" model="ir.ui.view">
        <field name="name">gift.card.search</field>
        <field name="model">gift.card</field>
        <field name="arch" type="xml">
            <search string="Gift Card">
                <field name="code"/>
                <filter name="valid" string="Valid" domain="[('state', '=', 'valid')]" />
            </search>
        </field>
    </record>
</odoo>

```

## File: views\template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="pay_with_gift_card_form">
        <form action="/shop/pay_with_gift_card" class="mb-2" method="post" name="gift_card_code">
            <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
            <div class="input-group w-100">
                <input name="gift_card_code" class="form-control" type="text" required="required" placeholder="Gift card code..."/>
                <div class="input-group-append">
                    <button href="#" type="submit" role="button" class="btn btn-secondary a-submit">Pay</button>
                </div>
            </div>
        </form>
        <t t-if="request.params.get('gift_card_error')">
            <div class="alert alert-danger text-left mt16" role="alert" t-esc="request.params.get('gift_card_error')"/>
        </t>
    </template>

    <template id="cart_line_product_no_link" inherit_id="website_sale.cart_line_product_link">
        <xpath expr="." position="replace">
            <div>
                <t t-if="not line.gift_card_id">
                    <a t-att-href="line.product_id.website_url">
                        <strong t-field="line.name_short"/>
                    </a>
                </t>
                <t t-else="">
                    <strong t-field="line.name_short"/>
                    <t t-call="sale_gift_card.used_gift_card"/>
                </t>
            </div>
        </xpath>
    </template>

    <template id="add_gift_card" inherit_id="website_sale.total" name="Gift Card">
        <xpath expr="//div[@id='cart_total']//table/tr[last()]" position="after">
            <tr t-if="allow_pay_with_gift_card" class="oe_website_sale_gift_card">
                <td colspan="3" class="text-center text-xl-right border-0">
                    <span class=''>
                        <t t-set='force_gift_card' t-value="request.params.get('gift_card_error')"/>
                        <t t-if="not force_gift_card">
                            <a href="#" class="js_show_gift_card">Pay with a gift card</a>
                        </t>
                        <div t-attf-class="gift_card_form #{not force_gift_card and 'd-none'}">
                            <t t-call="website_sale_gift_card.pay_with_gift_card_form"/>
                        </div>
                    </span>
                </td>
            </tr>
        </xpath>
    </template>

    <template id="cart_summary_inherit_website_gift_card_sale" inherit_id="website_sale.cart_summary">
        <xpath expr="//td[hasclass('td-product_name')]/div/strong" position="after">
            <t t-if="line.gift_card_id" t-call="sale_gift_card.used_gift_card"/>
        </xpath>
    </template>

    <template id="website_sale_purchased_gift_card" inherit_id="website_sale.confirmation" >
        <xpath expr="//div[@id='oe_structure_website_sale_confirmation_2']" position="after">
            <t t-call="sale_gift_card.sale_purchased_gift_card"/>
        </xpath>
    </template>
</odoo>

```

