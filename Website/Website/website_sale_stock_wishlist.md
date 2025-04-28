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
        'views/templates.xml',
        'data/template_email.xml',
        'data/ir_cron_data.xml',
    ],
    'assets': {
        'web.assets_frontend': [
            'website_sale_stock_wishlist/static/src/**/*',
        ],
    },
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import http
from odoo.http import request
from odoo.addons.website_sale.controllers.main import WebsiteSale


class WebsiteSaleStockWishlist(WebsiteSale):
    @http.route(['/shop/wishlist/notify/<model("product.wishlist"):wish>'], type='json', auth="public", website=True)
    def notify_stock(self, wish, notify=True, **kw):
        if not request.website.is_public_user():
            wish['stock_notification'] = notify
        return wish['stock_notification']

```

## File: controllers\variant.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http
from odoo.addons.website_sale.controllers.variant import WebsiteSaleVariantController


class WebsiteSaleStockWishlistVariantController(WebsiteSaleVariantController):
    @http.route()
    def get_combination_info_website(self, product_template_id, product_id, combination, add_qty, **kw):
        kw['context'] = kw.get('context', {})
        kw['context'].update(website_sale_stock_wishlist_get_wish=True)
        return super().get_combination_info_website(product_template_id, product_id, combination, add_qty, **kw)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main
from . import variant

```

## File: data\ir_cron_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="ir_cron_send_availability_email" model="ir.cron">
        <field name="name">Wishlist: send email regarding products availability</field>
        <field name="interval_number">1</field>
        <field name="interval_type">hours</field>
        <field name="numbercall">-1</field>
        <field name="doall" eval="False"/>
        <field name="model_id" ref="model_product_wishlist"/>
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
                <a t-attf-href="#{wishlist.product_id.website_url}">
                    <img t-attf-src="/web/image/product.product/#{wishlist.product_id.id}/image_1920"/>
                </a>
            </div>
            <div style="display: flex; flex-direction: row; align-items: center; justify-content: center; width: 100%;">
                <p t-esc="wishlist.product_id.name"/>
                <p style="margin-left: 0.5em; margin-right: 0.5em">-</p>
                <p t-esc="wishlist.price" t-options="{'widget': 'monetary', 'display_currency': wishlist.currency_id}"/>
            </div>
            <p t-esc="wishlist.product_id.description_sale"/>
            <div style="display: flex; justify-content: center; width: 100%;">
                <a t-attf-href="#{wishlist.product_id.website_url}" style="background-color: #875A7B; padding: 8px 16px 8px 16px; text-decoration: none; color: #fff; border-radius: 5px; font-size:13px;">
                    Order Now
                </a>
            </div>
            <p>Regards,</p>
        </div>
    </template>
</odoo>

```

## File: models\product_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models

class ProductTemplate(models.Model):
    _inherit = "product.template"

    def _get_combination_info(self, combination=False, product_id=False, add_qty=1, pricelist=False, parent_combination=False, only_template=False):
        combination_info = super(ProductTemplate, self)._get_combination_info(
            combination=combination,
            product_id=product_id,
            add_qty=add_qty,
            pricelist=pricelist,
            parent_combination=parent_combination,
            only_template=only_template,
        )

        if not self.env.context.get('website_sale_stock_wishlist_get_wish'):
            return combination_info

        if combination_info['product_id']:
            product = self.env['product.product'].sudo().browse(combination_info["product_id"])
            combination_info['wish'] = product._is_in_wishlist()

        return combination_info

```

## File: models\product_wishlist.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models, _


class ProductWishlist(models.Model):
    _inherit = "product.wishlist"

    stock_notification = fields.Boolean(default=False, required=True)

    def _add_to_wishlist(self, pricelist_id, currency_id, website_id, price, product_id, partner_id=False):
        wish = super()._add_to_wishlist(
            pricelist_id=pricelist_id,
            currency_id=currency_id,
            website_id=website_id,
            price=price,
            product_id=product_id,
            partner_id=partner_id,
        )
        wish['stock_notification'] = wish.product_id._is_sold_out()

        return wish

    def _send_availability_email(self):
        to_notify = self.env['product.wishlist'].search([('stock_notification', '=', True)])

        if not to_notify:
            return

        notified = self.env['product.wishlist']

        # cannot group by product_id because it depend of website_id -> warehouse_id
        tmpl = self.env.ref("website_sale_stock_wishlist.availability_email_body")
        for wishlist in to_notify:
            product = wishlist.with_context(website_id=wishlist.website_id.id).product_id
            if not product._is_sold_out():
                body_html = tmpl._render({"wishlist": wishlist})
                msg = self.env["mail.message"].sudo().new(dict(body=body_html, record_name=product.name))
                full_mail = self.env["mail.render.mixin"]._render_encapsulate(
                    "mail.mail_notification_light",
                    body_html,
                    add_context=dict(message=msg, model_description=_("Wishlist")),
                )
                mail_values = {
                    "subject": _("The product '%(product_name)s' is now available") % {'product_name': product.name},
                    "email_from": (product.company_id.partner_id or self.env.user).email_formatted,
                    "email_to": wishlist.partner_id.email_formatted,
                    "body_html": full_mail,
                }

                mail = self.env["mail.mail"].sudo().create(mail_values)
                mail.send(raise_exception=False)
                notified += wishlist
        notified.stock_notification = False

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import product_wishlist
from . import product_template

```

## File: static\src\js\website_sale.js

```javascript
/** @odoo-module **/

import publicWidget from "web.public.widget";
import "website_sale.website_sale";
import ajax from "web.ajax";
import { qweb as QWeb } from "web.core";

const loadXml = async () => {
    return ajax.loadXML('/website_sale_stock_wishlist/static/src/xml/product_availability.xml', QWeb);
};

publicWidget.registry.WebsiteSale.include({

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Displays additional info messages regarding the product's
     * stock and the wishlist.
     *
     * @override
     */
    _onChangeCombination: async function (ev, $parent, combination) {
        this._super(...arguments);
        loadXml().then(() => {
            if (this.el.querySelector('.o_add_wishlist_dyn')) {
                const messageEl = this.el.querySelector('div.availability_messages');
                if (messageEl)
                    messageEl.insertAdjacentHTML('beforeend', QWeb.render('website_sale_stock_wishlist.product_availability', combination));
            }
        });
    },
});

```

## File: static\src\js\wishlist.js

```javascript
/** @odoo-module **/

import publicWidget from 'web.public.widget';
import 'website_sale_wishlist.wishlist';

publicWidget.registry.ProductWishlist.include({
    events: _.extend({}, publicWidget.registry.ProductWishlist.prototype.events, {
        'click .wishlist-section .o_notify_stock': '_onClickNotifyStock',
    }),

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Removes wishlist indication when adding a product to the wishlist.
     *
     * @override
     */
    _addNewProducts: function () {
        this._super(...arguments);
        const wishlistMessageEl = this.el.querySelector('#stock_wishlist_message');
        if (wishlistMessageEl) {
            wishlistMessageEl.classList.add('d-none');
        }
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onClickNotifyStock: function (ev) {
        const targetEl = ev.currentTarget;
        const wishID = targetEl.closest('tr').dataset.wishId;
        const iconEl = targetEl.querySelector('i');
        const currentNotify = targetEl.dataset.notify === 'True';
        this._rpc({
            route: `/shop/wishlist/notify/${encodeURIComponent(wishID)}`,
            params: {
                notify: !currentNotify,
            }
        }).then((notify) => {
            targetEl.dataset.notify = notify ? 'True' : 'False';
            iconEl.classList.toggle('fa-check-square-o', notify);
            iconEl.classList.toggle('fa-square-o', !notify);
        });
    },
});

```

## File: static\src\xml\product_availability.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates>
    <t t-name="website_sale_stock_wishlist.product_availability" inherit_id="website_sale_stock.product_availability">
        <div id="stock_wishlist_message" t-if="product_type == 'product' and !free_qty and !allow_out_of_stock_order and !wish" t-attf-class="availability_message_#{product_template} text-success mt8">
            Add the item to your wishlist to be notified when the product is back in stock.
        </div>
    </t>
</templates>

```

## File: views\templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="product_wishlist" inherit_id="website_sale_wishlist.product_wishlist">
        <xpath expr="//button[hasclass('o_wish_rm')]" position="before">
            <small class="text-danger d-md-block" t-if="wish.product_id._is_sold_out()">Temporarily out of stock</small>
        </xpath>
        <xpath expr="//button[hasclass('o_wish_rm')]" position="after">
            <t t-set="notify" t-value="wish.stock_notification"/>
            <button groups="base.group_user,base.group_portal" t-att-data-notify="notify" t-if="notify or wish.product_id._is_sold_out()" type="button" class="btn btn-link o_notify_stock no-decoration">
                <small><i t-attf-class="fa #{'fa-check-square-o' if notify else 'fa-square-o'}"></i> Be notified when back in stock</small>
            </button>
        </xpath>
        <xpath expr="//button[hasclass('o_wish_add')]" position="attributes">
            <attribute name="t-att-disabled">not wish.product_id.allow_out_of_stock_order and wish.product_id._is_sold_out()</attribute>
        </xpath>
    </template>
</odoo>

```

