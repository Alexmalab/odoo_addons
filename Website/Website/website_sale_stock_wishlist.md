# Odoo Module: website_sale_stock_wishlist

Category: Website/Website

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import controllers

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Product Availability Notifications',
    'category': 'Website/Website',
    'summary': 'Notify the user when a product is back in stock',
    'description': """
Allow the user to select if he wants to receive email notifications when a product of his wishlist gets back in stock.
    """,
    'depends': [
        'website_sale_stock',
        'website_sale_wishlist',
    ],
    'data': [
        'views/website_sale_stock_wishlist_templates.xml',
    ],
    'assets': {
        'web.assets_frontend': [
            'website_sale_stock_wishlist/static/src/**/*',
        ],
        'web.assets_tests': [
            'website_sale_stock_wishlist/static/tests/tours/website_sale_stock_wishlist_stock_notification.js',
        ],
    },
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: controllers\variant.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.http import request, route

from odoo.addons.website_sale.controllers.variant import WebsiteSaleVariantController


class WebsiteSaleStockWishlistVariantController(WebsiteSaleVariantController):

    @route()
    def get_combination_info_website(self, *args, **kwargs):
        request.update_context(website_sale_stock_wishlist_get_wish=True)
        return super().get_combination_info_website(*args, **kwargs)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import variant

```

## File: models\product_template.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class ProductTemplate(models.Model):
    _inherit = 'product.template'

    def _get_additionnal_combination_info(self, product_or_template, quantity, date, website):
        res = super()._get_additionnal_combination_info(product_or_template, quantity, date, website)

        if not self.env.context.get('website_sale_stock_wishlist_get_wish'):
            return res

        if product_or_template.is_product_variant:
            product_sudo = product_or_template.sudo()
            res['is_in_wishlist'] = product_sudo._is_in_wishlist()

        return res

```

## File: models\product_wishlist.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models, api


class ProductWishlist(models.Model):
    _inherit = "product.wishlist"

    stock_notification = fields.Boolean(compute='_compute_stock_notification', default=False, required=True)

    @api.depends("product_id", "partner_id")
    def _compute_stock_notification(self):
        for record in self:
            record.stock_notification = record.product_id._has_stock_notification(record.partner_id)

    def _inverse_stock_notification(self):
        for record in self:
            if record.stock_notification:
                record.product_id.stock_notification_partner_ids += record.partner_id

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import product_template
from . import product_wishlist

```

## File: static\src\js\variant.js

```javascript
/** @odoo-module **/

import VariantMixin from "@website_sale_stock/js/variant_mixin";
import "@website_sale/js/website_sale";
import { renderToElement } from "@web/core/utils/render";

const oldChangeCombinationStock = VariantMixin._onChangeCombinationStock;
/**
 * Displays additional info messages regarding the product's
 * stock and the wishlist.
 *
 * @override
 */
VariantMixin._onChangeCombinationStock = function (ev, $parent, combination) {
    oldChangeCombinationStock.apply(this, arguments);
    if (this.el.querySelector('.o_add_wishlist_dyn')) {
        const messageEl = this.el.querySelector('div.availability_messages');
        if (messageEl && !this.el.querySelector('#stock_wishlist_message')) {
            messageEl.append(
                renderToElement('website_sale_stock_wishlist.product_availability', combination) || ''
            );
        }
    }
};

```

## File: static\src\js\website_sale.js

```javascript
/** @odoo-module **/

import WebsiteSale from '@website_sale_stock/js/website_sale';

WebsiteSale.include({

    events: Object.assign({}, WebsiteSale.prototype.events, {
        'click #wishlist_stock_notification_message': '_onClickWishlistStockNotificationMessage',
        'click #wishlist_stock_notification_form_submit_button': '_onClickSubmitWishlistStockNotificationForm',
    }),

    _onClickWishlistStockNotificationMessage(ev) {
        this._handleClickStockNotificationMessage(ev);
    },

    _onClickSubmitWishlistStockNotificationForm(ev) {
        const productId = ev.currentTarget.closest('tr').dataset.productId;
        this._handleClickSubmitStockNotificationForm(ev, productId);
    },
});

```

## File: static\src\js\wishlist.js

```javascript
/** @odoo-module **/

import publicWidget from '@web/legacy/js/public/public_widget';
import '@website_sale_wishlist/js/website_sale_wishlist';

publicWidget.registry.ProductWishlist.include({

    /**
     * Removes wishlist indication when adding a product to the wishlist.
     *
     * @override
     */
    _addNewProducts: function () {
        this._super(...arguments);
        const saveForLaterButtonEl = document.querySelector('#wsale_save_for_later_button');
        const addedToYourWishListAlertEl = document.querySelector('#wsale_added_to_your_wishlist_alert');
        if (saveForLaterButtonEl) {
            saveForLaterButtonEl.classList.add('d-none');
            addedToYourWishListAlertEl.classList.remove('d-none');
        }
    },
});

```

## File: static\src\xml\product_availability.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates>
    <t t-name="website_sale_stock_wishlist.product_availability" inherit_id="website_sale_stock.product_availability">
        <div id="stock_wishlist_message"
             t-if="product_type == 'product' and !free_qty and !allow_out_of_stock_order and !prevent_zero_price_sale"
             t-attf-class="availability_message_#{product_template} my-2 d-flex align-items-center flex-column flex-md-row">
            <button
                    id="wsale_save_for_later_button"
                    t-if="!is_in_wishlist"
                    type="button"
                    role="button"
                    class="btn btn-secondary text-nowrap o_add_wishlist_dyn"
                    t-att-data-product-template-id="product_id"
                    t-att-data-product-product-id="product_id"
                    data-action="o_wishlist"
                    title="Add to wishlist">
                <i class="fa fa-clock-o me-2"/>
                Save for later
            </button>
            <div id="wsale_added_to_your_wishlist_alert" t-att-class="is_in_wishlist ? '' : 'd-none'">
                <div class="alert alert-success">
                    <i class="fa fa-heart me-1"/>
                    Added to your wishlist
                </div>
            </div>

        </div>
    </t>
</templates>

```

## File: views\website_sale_stock_wishlist_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="product_wishlist" inherit_id="website_sale_wishlist.product_wishlist">
        <xpath expr="//button[hasclass('o_wish_rm')]" position="before">
            <small class="text-danger d-md-block" t-if="wish.product_id._is_sold_out() and not wish.product_id.allow_out_of_stock_order">Temporarily out of stock</small>
        </xpath>
        <xpath expr="//button[hasclass('o_wish_rm')]" position="after">
            <t t-if="not wish.product_id.allow_out_of_stock_order and wish.product_id._is_sold_out()">
                <t t-set="has_stock_notification"
                   t-value="wish.product_id._has_stock_notification(wish.partner_id) or request and wish.product_id.id in request.session.get('product_with_stock_notification_enabled', set())"/>
                <div id="stock_notification_div" t-if="not wish.product_id.allow_out_of_stock_order"
                     class="small d-inline-block">
                    <div class="btn btn-link" t-if="not has_stock_notification"
                         id="wishlist_stock_notification_message">
                        <small>
                            <i class="fa fa-envelope-o"/>
                            Get notified when back in stock
                        </small>
                    </div>
                    <div id="stock_notification_form" class="d-none">
                        <div class="input-group">
                            <input id="stock_notification_input"
                                   class="form-control"
                                   name="email"
                                   type="text"
                                   placeholder="youremail@gmail.com"
                                   t-att-value="request.env.user.partner_id.email or request.session.get('stock_notification_email', '')"/>
                            <div id="wishlist_stock_notification_form_submit_button"
                                    class="btn btn-secondary">
                                <i class="fa fa-paper-plane"/>
                            </div>
                            <div id="stock_notification_input_incorrect" class="btn d-none">
                                <i class="fa fa-times text-danger"/>
                                Invalid email
                            </div>
                        </div>
                    </div>
                    <div id="stock_notification_success_message" class="text-muted"
                         t-att-class="'text-muted' if has_stock_notification else 'd-none'">
                        <i class="fa fa-bell"/>
                        We'll notify you once the product is back in stock.
                    </div>
                </div>
            </t>
        </xpath>
        <xpath expr="//button[@id='add_to_cart_button']" position="replace">
            <t t-set="is_sold_out"
               t-value="not wish.product_id.allow_out_of_stock_order and wish.product_id._is_sold_out()"/>
            <button t-if="not combination_info['prevent_zero_price_sale']" id="add_to_cart_button"
                    class="btn btn-secondary btn-block o_wish_add mb4"
                    t-att-disabled="is_sold_out">
                <span class="fa fa-fw fa-shopping-cart"/>
                <span class="d-none d-md-inline">Add</span>
            </button>
        </xpath>
    </template>
</odoo>

```

