# Odoo Module: website_sale_wishlist

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
    'name': "Shopper's Wishlist",
    'summary': 'Allow shoppers to enlist products',
    'description': """
Allow shoppers of your eCommerce store to create personalized collections of products they want to buy and save them for future reference.
    """,
    'author': 'Odoo SA',
    'category': 'Website/Website',
    'version': '1.0',
    'depends': ['website_sale'],
    'data': [
        'security/website_sale_wishlist_security.xml',
        'security/ir.model.access.csv',
        'views/website_sale_wishlist_template.xml',
        'views/snippets.xml',
    ],
    'installable': True,
    'assets': {
        'web.assets_frontend': [
            'website_sale_wishlist/static/src/**/*',
        ],
        'web.assets_tests': [
            'website_sale_wishlist/static/tests/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-

import json

from odoo.http import request, route
from odoo.addons.website_sale.controllers.main import WebsiteSale


class WebsiteSaleWishlist(WebsiteSale):

    @route(['/shop/wishlist/add'], type='json', auth="public", website=True)
    def add_to_wishlist(self, product_id, **kw):
        website = request.website
        pricelist = website.pricelist_id
        product = request.env['product.product'].browse(product_id)

        price = product._get_combination_info_variant(
            pricelist=website.pricelist_id,
        )['price']

        Wishlist = request.env['product.wishlist']
        if request.website.is_public_user():
            Wishlist = Wishlist.sudo()
            partner_id = False
        else:
            partner_id = request.env.user.partner_id.id

        wish = Wishlist._add_to_wishlist(
            pricelist.id,
            pricelist.currency_id.id,
            request.website.id,
            price,
            product_id,
            partner_id
        )

        if not partner_id:
            request.session['wishlist_ids'] = request.session.get('wishlist_ids', []) + [wish.id]

        return wish

    @route(['/shop/wishlist'], type='http', auth="public", website=True, sitemap=False)
    def get_wishlist(self, count=False, **kw):
        values = request.env['product.wishlist'].with_context(display_default_code=False).current()
        if count:
            return request.make_response(json.dumps(values.mapped('product_id').ids))

        if not len(values):
            return request.redirect("/shop")

        return request.render("website_sale_wishlist.product_wishlist", dict(wishes=values))

    @route(['/shop/wishlist/remove/<model("product.wishlist"):wish>'], type='json', auth="public", website=True)
    def rm_from_wishlist(self, wish, **kw):
        if request.website.is_public_user():
            wish_ids = request.session.get('wishlist_ids') or []
            if wish.id in wish_ids:
                request.session['wishlist_ids'].remove(wish.id)
                request.session.touch()
                wish.sudo().unlink()
        else:
            wish.unlink()
        return True

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
from . import main

```

## File: models\product_wishlist.py

```python
# -*- coding: utf-8 -*-
from datetime import datetime, timedelta
from odoo import api, fields, models
from odoo.http import request


class ProductWishlist(models.Model):
    _name = 'product.wishlist'
    _description = 'Product Wishlist'
    _sql_constraints = [
        ("product_unique_partner_id",
         "UNIQUE(product_id, partner_id)",
         "Duplicated wishlisted product for this partner."),
    ]

    partner_id = fields.Many2one('res.partner', string='Owner')
    product_id = fields.Many2one('product.product', string='Product', required=True)
    currency_id = fields.Many2one('res.currency', related='pricelist_id.currency_id', readonly=True)
    pricelist_id = fields.Many2one('product.pricelist', string='Pricelist', help='Pricelist when added')
    price = fields.Monetary(currency_field='currency_id', string='Price', help='Price of the product when it has been added in the wishlist')
    website_id = fields.Many2one('website', ondelete='cascade', required=True)
    active = fields.Boolean(default=True, required=True)

    @api.model
    def current(self):
        """Get all wishlist items that belong to current user or session,
        filter products that are unpublished."""
        if not request:
            return self

        if request.website.is_public_user():
            wish = self.sudo().search([('id', 'in', request.session.get('wishlist_ids', []))])
        else:
            wish = self.search([("partner_id", "=", self.env.user.partner_id.id), ('website_id', '=', request.website.id)])

        return wish.filtered(
            lambda wish:
                wish.sudo().product_id.product_tmpl_id.website_published
                and wish.sudo().product_id.product_tmpl_id._can_be_added_to_cart()
        )

    @api.model
    def _add_to_wishlist(self, pricelist_id, currency_id, website_id, price, product_id, partner_id=False):
        wish = self.env['product.wishlist'].create({
            'partner_id': partner_id,
            'product_id': product_id,
            'currency_id': currency_id,
            'pricelist_id': pricelist_id,
            'price': price,
            'website_id': website_id,
        })
        return wish

    @api.model
    def _check_wishlist_from_session(self):
        """Assign all wishlist withtout partner from this the current session"""
        session_wishes = self.sudo().search([('id', 'in', request.session.get('wishlist_ids', []))])
        partner_wishes = self.sudo().search([("partner_id", "=", self.env.user.partner_id.id)])
        partner_products = partner_wishes.mapped("product_id")
        # Remove session products already present for the user
        duplicated_wishes = session_wishes.filtered(lambda wish: wish.product_id <= partner_products)
        session_wishes -= duplicated_wishes
        duplicated_wishes.unlink()
        # Assign the rest to the user
        session_wishes.write({"partner_id": self.env.user.partner_id.id})
        request.session.pop('wishlist_ids')

    @api.autovacuum
    def _gc_sessions(self, *args, **kwargs):
        """Remove wishlists for unexisting sessions."""
        self.with_context(active_test=False).search([
            ("create_date", "<", fields.Datetime.to_string(datetime.now() - timedelta(weeks=kwargs.get('wishlist_week', 5)))),
            ("partner_id", "=", False),
        ]).unlink()


class ResPartner(models.Model):
    _inherit = 'res.partner'

    wishlist_ids = fields.One2many('product.wishlist', 'partner_id', string='Wishlist', domain=[('active', '=', True)])


class ProductTemplate(models.Model):
    _inherit = 'product.template'

    def _is_in_wishlist(self):
        self.ensure_one()
        return self in self.env['product.wishlist'].current().mapped('product_id.product_tmpl_id')


class ProductProduct(models.Model):
    _inherit = 'product.product'

    def _is_in_wishlist(self):
        self.ensure_one()
        return self in self.env['product.wishlist'].current().mapped('product_id')

```

## File: models\res_users.py

```python
from odoo import api, fields, models
from odoo.http import request


class ResUsers(models.Model):
    _inherit = "res.users"

    def _check_credentials(self, password, env):
        """Make all wishlists from session belong to its owner user."""
        result = super(ResUsers, self)._check_credentials(password, env)
        if request and request.session.get('wishlist_ids'):
            self.env["product.wishlist"]._check_wishlist_from_session()
        return result

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
from . import product_wishlist
from . import res_users

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_product_wishlist_default,access_product_wishlist_default,model_product_wishlist,,0,0,0,0
access_product_wishlist_public,access_product_wishlist_public,model_product_wishlist,base.group_public,0,0,0,0
access_product_wishlist_portal,access_product_wishlist_portal,model_product_wishlist,base.group_portal,1,1,1,1
access_product_wishlist_user,access_product_wishlist_user,model_product_wishlist,base.group_user,1,1,1,1

```

## File: security\website_sale_wishlist_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

        <record id="product_wishlist_rule" model="ir.rule">
            <field name="name">See own Wishlist</field>
            <field name="model_id" ref="model_product_wishlist"/>
            <field name="domain_force">[('partner_id','=', user.partner_id.id)]</field>
            <field name="groups" eval="[(4, ref('base.group_portal')), (4, ref('base.group_user'))]"/>
        </record>

        <record id="all_product_wishlist_rule" model="ir.rule">
            <field name="name">See all wishlist</field>
            <field name="model_id" ref="model_product_wishlist"/>
            <field name="domain_force">[(1, '=', 1)]</field>
            <field name="groups" eval="[(4, ref('sales_team.group_sale_manager'))]"/>
        </record>

</odoo>

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#269396"/><stop offset="100%" stop-color="#218689"/></linearGradient><path id="d" d="M50.363 41a1.636 1.636 0 0 1-1.59 1.286L27 43l1 4h19c1 0 1 2 0 2H26l-6-24h-2v1c0 .667-.333 1-1 1s-1-.333-1-1v-2c.066-.667.4-1 1-1h4c.517 0 .85.333 1 1l1 3.281h28.45c1.048 0 1.824.985 1.592 2.019L50.363 41zM45.5 55a2.5 2.5 0 1 1 0-5 2.5 2.5 0 0 1 0 5zm-19 0a2.5 2.5 0 1 1 0-5 2.5 2.5 0 0 1 0 5zm11.375-15c.114 0 .21-.04.289-.118l4.084-3.934c1.001-1.001 1.502-1.985 1.502-2.95 0-.962-.278-1.714-.833-2.256-.555-.542-1.322-.813-2.301-.813-.271 0-.548.047-.83.14-.282.095-.544.221-.786.38a8.39 8.39 0 0 0-.627.45c-.175.14-.34.288-.498.446a6.041 6.041 0 0 0-.498-.446 8.39 8.39 0 0 0-.627-.45 3.357 3.357 0 0 0-.786-.38 2.614 2.614 0 0 0-.83-.14c-.979 0-1.746.27-2.301.813-.555.542-.833 1.294-.833 2.255 0 .293.051.595.154.905.103.31.22.575.35.793.132.219.28.432.447.64.166.207.287.35.364.43.076.078.136.135.18.17l4.091 3.947a.392.392 0 0 0 .289.118z"/><path id="e" d="M50.363 39a1.636 1.636 0 0 1-1.59 1.286L27 41l1 4h19c1 0 1 2 0 2H26l-6-24h-2v1c0 .667-.333 1-1 1s-1-.333-1-1v-2c.066-.667.4-1 1-1h4c.517 0 .85.333 1 1l1 3.281h28.45c1.048 0 1.824.985 1.592 2.019L50.363 39zM45.5 53a2.5 2.5 0 1 1 0-5 2.5 2.5 0 0 1 0 5zm-19 0a2.5 2.5 0 1 1 0-5 2.5 2.5 0 0 1 0 5zm11.375-15c.114 0 .21-.04.289-.118l4.084-3.934c1.001-1.001 1.502-1.985 1.502-2.95 0-.962-.278-1.714-.833-2.256-.555-.542-1.322-.813-2.301-.813-.271 0-.548.047-.83.14-.282.095-.544.221-.786.38a8.39 8.39 0 0 0-.627.45c-.175.14-.34.288-.498.446a6.041 6.041 0 0 0-.498-.446 8.39 8.39 0 0 0-.627-.45 3.357 3.357 0 0 0-.786-.38 2.614 2.614 0 0 0-.83-.14c-.979 0-1.746.27-2.301.813-.555.542-.833 1.294-.833 2.255 0 .293.051.595.154.905.103.31.22.575.35.793.132.219.28.432.447.64.166.207.287.35.364.43.076.078.136.135.18.17l4.091 3.947a.392.392 0 0 0 .289.118z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M31.59 69H4c-2 0-4-1-4-4V38.29l16.235-16.918L21 21l2 5h29.636L50.23 39.393 45.537 45H47l.511 1.724-2.21 2.392 2.3 2.671L31.59 69z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: static\src\js\website_sale.js

```javascript
/** @odoo-module **/

import { WebsiteSale } from 'website_sale.website_sale';

WebsiteSale.include({
    /**
     * Toggles the add to cart button depending on the possibility of the
     * current combination.
     *
     * @override
     */
    _toggleDisable: function ($parent, isCombinationPossible) {
        this._super(...arguments);
        $parent.find('button.o_wish_add').toggleClass('disabled', !isCombinationPossible);
    },
});

```

## File: static\src\js\website_sale_wishlist.js

```javascript
odoo.define('website_sale_wishlist.wishlist', function (require) {
"use strict";

var publicWidget = require('web.public.widget');
var wSaleUtils = require('website_sale.utils');
var VariantMixin = require('sale.VariantMixin');

// VariantMixin events are overridden on purpose here
// to avoid registering them more than once since they are already registered
// in website_sale.js
publicWidget.registry.ProductWishlist = publicWidget.Widget.extend(VariantMixin, {
    selector: '.oe_website_sale',
    events: {
        'click .o_wsale_my_wish': '_onClickMyWish',
        'click .o_add_wishlist, .o_add_wishlist_dyn': '_onClickAddWish',
        'change input.product_id': '_onChangeVariant',
        'change input.js_product_change': '_onChangeProduct',
        'click .wishlist-section .o_wish_rm': '_onClickWishRemove',
        'click .wishlist-section .o_wish_add': '_onClickWishAdd',
    },

    /**
     * @constructor
     */
    init: function (parent) {
        this._super.apply(this, arguments);
        this.wishlistProductIDs = JSON.parse(sessionStorage.getItem('website_sale_wishlist_product_ids') || '[]');
    },
    /**
     * Gets the current wishlist items.
     * In editable mode, do nothing instead.
     *
     * @override
     */
    willStart: function () {
        var self = this;
        var def = this._super.apply(this, arguments);
        var wishDef;
        if (this.wishlistProductIDs.length != +$('#top_menu .my_wish_quantity').text()) {
            wishDef = $.get('/shop/wishlist', {
                count: 1,
            }).then(function (res) {
                self.wishlistProductIDs = JSON.parse(res);
                sessionStorage.setItem('website_sale_wishlist_product_ids', res);
            });

        }
        return Promise.all([def, wishDef]);
    },
    /**
     * Updates the wishlist view (navbar) & the wishlist button (product page).
     * In editable mode, do nothing instead.
     *
     * @override
     */
    start: function () {
        var def = this._super.apply(this, arguments);

        this._updateWishlistView();
        // trigger change on only one input
        if (this.$('input.js_product_change').length) { // manage "List View of variants"
            this.$('input.js_product_change:checked').first().trigger('change');
        } else {
            this.$('input.product_id').first().trigger('change');
        }

        return def;
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _addNewProducts: function ($el) {
        var self = this;
        var productID = $el.data('product-product-id');
        if ($el.hasClass('o_add_wishlist_dyn')) {
            productID = parseInt($el.closest('.js_product').find('.product_id:checked').val());;
        }
        var $form = $el.closest('form');
        var templateId = $form.find('.product_template_id').val();
        // when adding from /shop instead of the product page, need another selector
        if (!templateId) {
            templateId = $el.data('product-template-id');
        }
        $el.prop("disabled", true).addClass('disabled');
        var productReady = this.selectOrCreateProduct(
            $el.closest('form'),
            productID,
            templateId,
            false
        );

        productReady.then(function (productId) {
            productId = parseInt(productId, 10);

            if (productId && !_.contains(self.wishlistProductIDs, productId)) {
                return self._rpc({
                    route: '/shop/wishlist/add',
                    params: {
                        product_id: productId,
                    },
                }).then(function () {
                    var $navButton = $('header .o_wsale_my_wish').first();
                    self.wishlistProductIDs.push(productId);
                    sessionStorage.setItem('website_sale_wishlist_product_ids', JSON.stringify(self.wishlistProductIDs));
                    self._updateWishlistView();
                    wSaleUtils.animateClone($navButton, $el.closest('form'), 25, 40);
                    // It might happen that `onChangeVariant` is called at the same time as this function.
                    // In this case we need to set the button to disabled again.
                    // Do this only if the productID is still the same.
                    let currentProductId = $el.data('product-product-id');
                    if ($el.hasClass('o_add_wishlist_dyn')) {
                        currentProductId = parseInt($el.closest('.js_product').find('.product_id:checked').val());
                    }
                    if (productId === currentProductId) {
                        $el.prop("disabled", true).addClass('disabled');
                    }
                }).guardedCatch(function () {
                    $el.prop("disabled", false).removeClass('disabled');
                });
            }
        }).guardedCatch(function () {
            $el.prop("disabled", false).removeClass('disabled');
        });
    },
    /**
     * @private
     */
    _updateWishlistView: function () {
        const $wishButton = $('.o_wsale_my_wish');
        if ($wishButton.hasClass('o_wsale_my_wish_hide_empty')) {
            $wishButton.toggleClass('d-none', !this.wishlistProductIDs.length);
        }
        $wishButton.find('.my_wish_quantity').text(this.wishlistProductIDs.length);
    },
    /**
     * @private
     */
    _removeWish: function (e, deferred_redirect) {
        var tr = $(e.currentTarget).parents('tr');
        var wish = tr.data('wish-id');
        var product = tr.data('product-id');
        var self = this;

        this._rpc({
            route: '/shop/wishlist/remove/' + wish,
        }).then(function () {
            $(tr).hide();
        });

        this.wishlistProductIDs = _.without(this.wishlistProductIDs, product);
        sessionStorage.setItem('website_sale_wishlist_product_ids', JSON.stringify(this.wishlistProductIDs));
        if (this.wishlistProductIDs.length === 0) {
            if (deferred_redirect) {
                deferred_redirect.then(function () {
                    self._redirectNoWish();
                });
            }
        }
        this._updateWishlistView();
    },
    /**
     * @private
     */
    _addOrMoveWish: function (e) {
        var $navButton = $('header .o_wsale_my_cart').first();
        var tr = $(e.currentTarget).parents('tr');
        var product = tr.data('product-id');
        $('.o_wsale_my_cart').removeClass('d-none');
        wSaleUtils.animateClone($navButton, tr, 25, 40);

        if ($('#b2b_wish').is(':checked')) {
            return this._addToCart(product, tr.find('add_qty').val() || 1);
        } else {
            var adding_deffered = this._addToCart(product, tr.find('add_qty').val() || 1);
            this._removeWish(e, adding_deffered);
            return adding_deffered;
        }
    },
    /**
     * @private
     */
    _addToCart: function (productID, qty) {
        const $tr = this.$(`tr[data-product-id="${productID}"]`);
        const productTrackingInfo = $tr.data('product-tracking-info');
        if (productTrackingInfo) {
            productTrackingInfo.quantity = parseFloat(qty);
            $tr.trigger('add_to_cart_event', [productTrackingInfo]);
        }
        return this._rpc({
            route: "/shop/cart/update_json",
            params: this._getCartUpdateJsonParams(productID, qty),
        }).then(function (data) {
            sessionStorage.setItem('website_sale_cart_quantity', data.cart_quantity);
            wSaleUtils.updateCartNavBar(data);
            wSaleUtils.showWarning(data.warning);
        });
    },
    /**
     * Get the cart update params.
     *
     * @param {string} productId
     * @param {string} qty
     */
    _getCartUpdateJsonParams(productId, qty) {
        return {
            product_id: parseInt(productId, 10),
            add_qty: parseInt(qty, 10),
            display: false,
        };
    },
    /**
     * @private
     */
    _redirectNoWish: function () {
        window.location.href = '/shop/cart';
    },


    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onClickMyWish: function () {
        if (this.wishlistProductIDs.length === 0) {
            this._updateWishlistView();
            this._redirectNoWish();
            return;
        }
        window.location = '/shop/wishlist';
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onClickAddWish: function (ev) {
        this._addNewProducts($(ev.currentTarget));
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onChangeVariant: function (ev) {
        var $input = $(ev.target);
        var $parent = $input.closest('.js_product');
        var $el = $parent.find("[data-action='o_wishlist']");
        if (!_.contains(this.wishlistProductIDs, parseInt($input.val(), 10))) {
            $el.prop("disabled", false).removeClass('disabled').removeAttr('disabled');
        } else {
            $el.prop("disabled", true).addClass('disabled').attr('disabled', 'disabled');
        }
        $el.data('product-product-id', parseInt($input.val(), 10));
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onChangeProduct: function (ev) {
        var productID = ev.currentTarget.value;
        var $el = $(ev.target).closest('.js_add_cart_variants').find("[data-action='o_wishlist']");

        if (!_.contains(this.wishlistProductIDs, parseInt(productID, 10))) {
            $el.prop("disabled", false).removeClass('disabled').removeAttr('disabled');
        } else {
            $el.prop("disabled", true).addClass('disabled').attr('disabled', 'disabled');
        }
        $el.data('product-product-id', productID);
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onClickWishRemove: function (ev) {
        this._removeWish(ev, false);
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onClickWishAdd: function (ev) {
        if (ev.currentTarget.classList.contains('disabled')) {
            ev.preventDefault();
            return;
        }
        var self = this;
        this.$('.wishlist-section .o_wish_add').addClass('disabled');
        this._addOrMoveWish(ev).then(function () {
            self.$('.wishlist-section .o_wish_add').removeClass('disabled');
        });
    },
});
});

```

## File: views\snippets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="snippet_options" inherit_id="website.snippet_options" name="Wishlist Snippet Options">
    <xpath expr="." position="inside">
        <div data-selector="#wrapwrap > header"
            data-no-check="true"
            groups="website.group_website_designer">
            <we-checkbox string="Show Empty Wishlist"
                         data-customize-website-views="website_sale_wishlist.header_hide_empty_wishlist_link|"
                         data-no-preview="true"
                         data-reload="/"/>
        </div>
    </xpath>
    <xpath expr="//we-button[hasclass('o_we_add_to_cart_btn')]" position="after">
        <we-button title="Wishlist" class="fa fa-fw fa-heart"
                   data-customize-website-views="website_sale_wishlist.add_to_wishlist"
                   data-no-preview="true"
                   data-reload="/"/>
    </xpath>
    <xpath expr="//we-button[hasclass('o_we_buy_now_btn')]" position="after">
        <we-button title="Wishlist"
                   data-customize-website-views="website_sale_wishlist.product_add_to_wishlist"
                   data-no-preview="true"
                   data-reload="/">
            <i class="fa fa-fw fa-heart"/>
            Wishlist
        </we-button>
    </xpath>
</template>

</odoo>

```

## File: views\website_sale_wishlist_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="add_to_wishlist" inherit_id="website_sale.products_item" name="Wishlist Button" priority="20">
        <xpath expr="//div[hasclass('o_wsale_product_btn')]" position="inside">
            <t t-nocache="The wishlist depends on the user and must not be shared with other users. As this rendering is in the loop, the product value is set in the template, we must retain the id of the product on which we are."
               t-nocache-product_template_id="product.id">
                <t t-set="product" t-value="products.filtered(lambda p: p.id == product_template_id)"/>
                <t t-set="in_wish" t-value="product._is_in_wishlist()"/>
                <t t-set="product_variant_id" t-value="product._get_first_possible_variant_id()"/>
                <button t-if="product_variant_id" type="button" role="button" class="btn btn-outline-primary bg-white o_add_wishlist" t-att-disabled='in_wish or None' title="Add to Wishlist" t-att-data-product-template-id="product.id" t-att-data-product-product-id="product_variant_id" data-action="o_wishlist"><span class="fa fa-heart" role="img" aria-label="Add to wishlist"></span></button>
            </t>
        </xpath>
    </template>

    <template id="product_add_to_wishlist" inherit_id="website_sale.product" name="Wishlist Button" priority="20">
        <xpath expr="//div[@id='product_option_block']" position="inside">
            <t t-nocache="The wishlist depends on the user and must not be shared with other users. The product come from the controller.">
                <t t-set="product_variant" t-value="product_variant or product._create_first_product_variant()"/>
                <t t-set="in_wish" t-value="product_variant and product_variant._is_in_wishlist()"/>
                <button t-if="product_variant" type="button" role="button" class="btn btn-link px-0 pe-3 o_add_wishlist_dyn" t-att-disabled='in_wish or None' t-att-data-product-template-id="product.id" t-att-data-product-product-id="product_variant.id" data-action="o_wishlist" title="Add to wishlist"><i class="fa fa-heart-o me-2" role="img" aria-label="Add to wishlist"/>Add to wishlist</button>
            </t>
        </xpath>
    </template>

    <template id="header_wishlist_link" name="Header Wishlist Link">
        <t t-nocache="The wishlist may vary and depends on the user."
           t-nocache-_icon="_icon"
           t-nocache-_item_class="_item_class"
           t-nocache-_link_class="_link_class"
           t-nocache-_text="_text">
            <t t-set="wishcount" t-value="len(request.env['product.wishlist'].current())"/>
            <t t-set="show_wishes" t-value="True"/>
            <li t-attf-class="o_wsale_my_wish #{not show_wishes and 'd-none'} #{_item_class}">
                <a href="/shop/wishlist" t-attf-class="#{_link_class}">
                    <i t-if="_icon" class="fa fa-1x fa-heart"/>
                    <span t-if="_text">Wishlist</span>
                    <sup t-esc="wishcount" t-attf-class="my_wish_quantity o_animate_blink badge text-bg-primary"/>
                </a>
            </li>
        </t>
    </template>

    <template id="header_hide_empty_wishlist_link" inherit_id="website_sale_wishlist.header_wishlist_link">
        <xpath expr="//t[@t-set='show_wishes']" position="after">
            <t t-set="show_wishes" t-value="show_wishes and wishcount > 0"/>
        </xpath>
        <xpath expr="//li[contains(@t-attf-class, 'o_wsale_my_wish')]" position="attributes">
            <attribute name="t-attf-class" add="o_wsale_my_wish_hide_empty" separator=" "/>
        </xpath>
    </template>

    <template id="template_header_default" inherit_id="website_sale.template_header_default">
        <xpath expr="//t[@t-call='website_sale.header_cart_link']" position="after">
            <t t-call="website_sale_wishlist.header_wishlist_link">
                <t t-set="_icon" t-value="True"/>
                <t t-set="_item_class" t-value="'nav-item me-lg-3'"/>
                <t t-set="_link_class" t-value="'nav-link'"/>
            </t>
        </xpath>
    </template>

    <template id="template_header_hamburger" inherit_id="website_sale.template_header_hamburger">
        <xpath expr="//t[@t-call='website_sale.header_cart_link']" position="after">
            <t t-call="website_sale_wishlist.header_wishlist_link">
                <t t-set="_text" t-value="True"/>
                <t t-set="_item_class" t-value="'nav-item'"/>
                <t t-set="_link_class" t-value="'nav-link'"/>
            </t>
        </xpath>
    </template>

    <template id="template_header_vertical" inherit_id="website_sale.template_header_vertical">
        <xpath expr="//t[@t-call='website_sale.header_cart_link']" position="after">
            <t t-call="website_sale_wishlist.header_wishlist_link">
                <t t-set="_icon" t-value="True"/>
                <t t-set="_item_class" t-value="'nav-item ms-lg-3'"/>
                <t t-set="_link_class" t-value="'nav-link'"/>
            </t>
        </xpath>
    </template>

    <template id="template_header_sidebar" inherit_id="website_sale.template_header_sidebar">
        <xpath expr="//t[@t-call='website_sale.header_cart_link']" position="after">
            <t t-call="website_sale_wishlist.header_wishlist_link">
                <t t-set="_icon" t-value="True"/>
                <t t-set="_text" t-value="True"/>
                <t t-set="_item_class" t-value="'nav-item'"/>
                <t t-set="_link_class" t-value="'nav-link'"/>
            </t>
        </xpath>
    </template>

    <template id="template_header_slogan" inherit_id="website_sale.template_header_slogan">
        <xpath expr="//t[@t-call='website_sale.header_cart_link']" position="after">
            <t t-call="website_sale_wishlist.header_wishlist_link">
                <t t-set="_icon" t-value="True"/>
                <t t-set="_item_class" t-value="'nav-item ms-lg-3'"/>
                <t t-set="_link_class" t-value="'nav-link'"/>
            </t>
        </xpath>
    </template>

    <template id="template_header_contact" inherit_id="website_sale.template_header_contact">
        <xpath expr="//t[@t-call='website_sale.header_cart_link']" position="after">
            <t t-call="website_sale_wishlist.header_wishlist_link">
                <t t-set="_icon" t-value="True"/>
                <t t-set="_item_class" t-value="'nav-item ms-lg-3'"/>
                <t t-set="_link_class" t-value="'nav-link'"/>
            </t>
        </xpath>
    </template>

    <template id="template_header_boxed" inherit_id="website_sale.template_header_boxed">
        <xpath expr="//t[@t-call='website_sale.header_cart_link']" position="after">
            <t t-call="website_sale_wishlist.header_wishlist_link">
                <t t-set="_text" t-value="True"/>
                <t t-set="_item_class" t-value="'nav-item ms-lg-3'"/>
                <t t-set="_link_class" t-value="'nav-link'"/>
            </t>
        </xpath>
    </template>

    <template id="template_header_centered_logo" inherit_id="website_sale.template_header_centered_logo">
        <xpath expr="//t[@t-call='website_sale.header_cart_link']" position="after">
            <t t-call="website_sale_wishlist.header_wishlist_link">
                <t t-set="_icon" t-value="True"/>
                <t t-set="_item_class" t-value="'nav-item ms-1'"/>
                <t t-set="_link_class" t-value="'nav-link'"/>
            </t>
        </xpath>
    </template>

    <template id="template_header_image" inherit_id="website_sale.template_header_image">
        <xpath expr="//t[@t-call='website_sale.header_cart_link']" position="after">
            <t t-call="website_sale_wishlist.header_wishlist_link">
                <t t-set="_icon" t-value="True"/>
                <t t-set="_item_class" t-value="'nav-item ms-lg-2'"/>
                <t t-set="_link_class" t-value="'nav-link'"/>
            </t>
        </xpath>
    </template>

    <template id="template_header_hamburger_full" inherit_id="website_sale.template_header_hamburger_full">
        <xpath expr="//t[@t-call='website_sale.header_cart_link']" position="after">
            <t t-call="website_sale_wishlist.header_wishlist_link">
                <t t-set="_icon" t-value="True"/>
                <t t-set="_item_class" t-value="'nav-item ms-lg-2'"/>
                <t t-set="_link_class" t-value="'nav-link'"/>
            </t>
        </xpath>
    </template>

    <template id="template_header_magazine" inherit_id="website_sale.template_header_magazine">
        <xpath expr="//t[@t-call='website_sale.header_cart_link']" position="after">
            <t t-call="website_sale_wishlist.header_wishlist_link">
                <t t-set="_text" t-value="True"/>
                <t t-set="_item_class" t-value="'nav-item'"/>
                <t t-set="_link_class" t-value="'nav-link'"/>
            </t>
        </xpath>
    </template>

    <template id="product_wishlist" name="Wishlist Page">
        <t t-call="website.layout">
            <t t-set="additional_title">Shop Wishlist</t>
            <div id="wrap" class="js_sale">
                <div class="oe_structure" id="oe_structure_website_sale_wishlist_product_wishlist_1"/>
                <div class="container oe_website_sale pt-3">
                    <section class="container wishlist-section">
                        <h3>My Wishlist</h3>
                        <div class="checkbox">
                            <label class='text-muted'><input type="checkbox" id='b2b_wish' value="1" class="mr8"/>Add product to my cart but keep it in my wishlist</label>
                        </div>
                        <table class="table table-bordered table-striped table-hover text-center mt16 table-comparator " style="table-layout:auto" id="o_comparelist_table">
                            <body>
                                <t t-foreach="wishes" t-as="wish">
                                    <t t-set="combination_info" t-value="wish.product_id._get_combination_info_variant()"/>
                                    <tr t-att-data-wish-id='wish.id' t-att-data-product-id='wish.product_id.id'
                                        t-att-data-product-tracking-info="json.dumps(request.env['product.template'].get_google_analytics_data(combination_info))">
                                        <td class='td-img align-middle'>
                                            <a t-att-href="wish.product_id.website_url">
                                                <img t-attf-src="/web/image/product.product/#{wish.product_id.id}/image_128" class="img img-fluid" style="margin:auto;" alt="Product image"/>
                                            </a>
                                        </td>
                                        <td class='text-start align-middle'>
                                            <strong><a t-att-href="wish.product_id.website_url"><t t-esc="wish.product_id.display_name" /></a></strong>
                                            <small class='d-none d-md-block'><p t-field="wish.product_id.description_sale" class="text-muted"/></small>
                                            <button type="button" class="btn btn-link o_wish_rm no-decoration"><small><i class='fa fa-trash-o'></i> Remove</small></button>
                                        </td>
                                        <td class="align-middle" t-if="combination_info['prevent_zero_price_sale']">
                                            <span t-field="website.prevent_zero_price_sale_text"/>
                                        </td>
                                        <td class="align-middle o_wish_price" t-else="">
                                            <t t-esc="combination_info['price']" t-options="{'widget': 'monetary', 'display_currency': website.pricelist_id.currency_id}"/>
                                            <small t-if="combination_info['base_unit_price']" class="cart_product_base_unit_price d-block text-muted" groups="website_sale.group_show_uom_price">
                                                <t t-call='website_sale.base_unit_price'><t t-set='product' t-value='wish.product_id' /></t>
                                            </small>
                                        </td>
                                        <td class='text-center td-wish-btn align-middle'>
                                            <input name="product_id" t-att-value="wish.product_id.id" type="hidden"/>
                                            <a t-if="combination_info['prevent_zero_price_sale']" t-att-href="website.contact_us_button_url" class="btn btn-primary btn_cta">Contact Us</a>
                                            <button id="add_to_cart_button" t-else="" type="button" role="button" class="btn btn-secondary btn-block o_wish_add mb4" >
                                                Add <span class='d-none d-md-inline'>to Cart</span>
                                            </button>
                                        </td>
                                    </tr>
                                </t>
                            </body>
                        </table>
                    </section>
                </div>
            </div>
        </t>
    </template>

</odoo>

```

