# Odoo Module: website_sale_comparison

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
    'name': 'Product Comparison',
    'summary': 'Allow shoppers to compare products based on their attributes',
    'description': """
This module adds a comparison tool to your eCommerce shop, so that your shoppers can easily compare products based on their attributes. It will considerably accelerate their purchasing decision.

To configure product attributes, activate *Attributes & Variants* in the Website settings. This will add a dedicated section in the product form. In the configuration, this module adds a category field to product attributes in order to structure the shopper's comparison table.

Finally, the module comes with an option to display an attribute summary table in product web pages (available in Customize menu).
    """,
    'author': 'Odoo SA',
    'category': 'Website/Website',
    'version': '1.0',
    'depends': ['website_sale'],
    'data': [
        'security/ir.model.access.csv',
        'views/website_sale_comparison_template.xml',
        'views/website_sale_comparison_view.xml',
        'views/snippets.xml',
    ],
    'demo': [
        'data/website_sale_comparison_data.xml',
        'data/website_sale_comparison_demo.xml',
    ],
    'installable': True,
    'assets': {
        'web.assets_frontend': [
            'website_sale_comparison/static/src/scss/website_sale_comparison.scss',
            'website_sale_comparison/static/src/js/**/*.js',
            'website_sale_comparison/static/src/xml/comparison.xml',
        ],
        'web.assets_tests': [
            'website_sale_comparison/static/tests/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
from odoo import http
from odoo.http import request
from odoo.addons.website_sale.controllers.main import WebsiteSale
import json


class WebsiteSaleProductComparison(WebsiteSale):

    @http.route('/shop/compare', type='http', auth="public", website=True, sitemap=False)
    def product_compare(self, **post):
        values = {}
        product_ids = [int(i) for i in post.get('products', '').split(',') if i.isdigit()]
        if not product_ids:
            return request.redirect("/shop")
        # use search to check read access on each record/ids
        products = request.env['product.product'].search([('id', 'in', product_ids)])
        values['products'] = products.with_context(display_default_code=False)
        return request.render("website_sale_comparison.product_compare", values)

    @http.route(['/shop/get_product_data'], type='json', auth="public", website=True)
    def get_product_data(self, product_ids, cookies=None):
        ret = {}

        website = request.env['website'].get_current_website()
        pricelist = website.pricelist_id
        products = request.env['product.product'].search([('id', 'in', product_ids)])

        if cookies is not None:
            ret['cookies'] = json.dumps(request.env['product.product'].search([('id', 'in', list(set(product_ids + cookies)))]).ids)

        products = products.with_context(pricelist=pricelist.id, display_default_code=False)
        for product in products:
            ret[product.id] = {
                'render': request.env['ir.ui.view']._render_template(
                    "website_sale_comparison.product_product",
                    {'product': product, 'website': website}
                ),
                'product': dict(id=product.id, name=product.name, display_name=product.display_name),
            }
        return ret

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
from . import main

```

## File: data\website_sale_comparison_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="product_attribute_category_general_features" model="product.attribute.category">
        <field name="name">General Features</field>
        <field name="sequence">1</field>
    </record>
</odoo>
```

## File: data\website_sale_comparison_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="product_attribute_category_duration" model="product.attribute.category">
        <field name="name">Duration</field>
        <field name="sequence">20</field>
    </record>
    <record id="product.product_attribute_3" model="product.attribute">
        <field name="category_id" ref="product_attribute_category_duration"/>
    </record>


    <record id="product_attribute_category_2" model="product.attribute.category">
        <field name="name">Dimensions</field>
        <field name="sequence">7</field>
    </record>

    <record id="product.product_attribute_1" model="product.attribute">
        <field name="category_id" ref="product_attribute_category_general_features"/>
    </record>
    <record id="product.product_attribute_2" model="product.attribute">
        <field name="category_id" ref="product_attribute_category_general_features"/>
    </record>

    <record id="website_sale.product_attribute_brand" model="product.attribute">
        <field name="category_id" ref="product_attribute_category_general_features"/>
    </record>
    <record id="product_attribute_value_1" model="product.attribute.value">
        <field name="name">Apple</field>
        <field name="attribute_id" ref="website_sale.product_attribute_brand"/>
    </record>

    <record id="product_attribute_7" model="product.attribute">
        <field name="name">Weight</field>
        <field name="category_id" ref="product_attribute_category_2"/>
    </record>
    <record id="product_attribute_value_7" model="product.attribute.value">
        <field name="name">308 g</field>
        <field name="attribute_id" ref="product_attribute_7"/>
    </record>

    <record id="product_attribute_8" model="product.attribute">
        <field name="name">Dimensions</field>
        <field name="category_id" ref="product_attribute_category_2"/>
    </record>
    <record id="product_attribute_value_8" model="product.attribute.value">
        <field name="name">134.7 x 200 x 7.2 mm</field>
        <field name="attribute_id" ref="product_attribute_8"/>
    </record>

    <record id="product_6_attribute_1_product_template_attribute_line" model="product.template.attribute.line">
        <field name="product_tmpl_id" ref="product.product_product_6_product_template"/>
        <field name="attribute_id" ref="website_sale.product_attribute_brand"/>
        <field name="value_ids" eval="[(6,0,[ref('product_attribute_value_1')])]"/>
    </record>
    <record id="product_6_attribute_7_product_template_attribute_line" model="product.template.attribute.line">
        <field name="product_tmpl_id" ref="product.product_product_6_product_template"/>
        <field name="attribute_id" ref="product_attribute_7"/>
        <field name="value_ids" eval="[(6,0,[ref('product_attribute_value_7')])]"/>
    </record>
    <record id="product_6_attribute_8_template_attribute_line" model="product.template.attribute.line">
         <field name="product_tmpl_id" ref="product.product_product_6_product_template"/>
        <field name="attribute_id" ref="product_attribute_8"/>
        <field name="value_ids" eval="[(6,0,[ref('product_attribute_value_8')])]"/>
    </record>

</odoo>

```

## File: models\website_sale_comparison.py

```python
# -*- coding: utf-8 -*-

from collections import OrderedDict

from odoo import fields, models


class ProductAttributeCategory(models.Model):
    _name = "product.attribute.category"
    _description = "Product Attribute Category"
    _order = 'sequence, id'

    name = fields.Char("Category Name", required=True, translate=True)
    sequence = fields.Integer("Sequence", default=10, index=True)

    attribute_ids = fields.One2many('product.attribute', 'category_id', string="Related Attributes", domain="[('category_id', '=', False)]")


class ProductAttribute(models.Model):
    _inherit = 'product.attribute'
    _order = 'category_id, sequence, id'

    category_id = fields.Many2one('product.attribute.category', string="Category", index=True,
                                  help="Set a category to regroup similar attributes under "
                                  "the same section in the Comparison page of eCommerce")


class ProductTemplateAttributeLine(models.Model):
    _inherit = 'product.template.attribute.line'

    def _prepare_categories_for_display(self):
        """On the product page group together the attribute lines that concern
        attributes that are in the same category.

        The returned categories are ordered following their default order.

        :return: OrderedDict [{
            product.attribute.category: [product.template.attribute.line]
        }]
        """
        attributes = self.attribute_id
        categories = OrderedDict([(cat, self.env['product.template.attribute.line']) for cat in attributes.category_id.sorted()])
        if any(not pa.category_id for pa in attributes):
            # category_id is not required and the mapped does not return empty
            categories[self.env['product.attribute.category']] = self.env['product.template.attribute.line']
        for ptal in self:
            categories[ptal.attribute_id.category_id] |= ptal
        return categories


class ProductProduct(models.Model):
    _inherit = 'product.product'

    def _prepare_categories_for_display(self):
        """On the comparison page group on the same line the values of each
        product that concern the same attributes, and then group those
        attributes per category.

        The returned categories are ordered following their default order.

        :return: OrderedDict [{
            product.attribute.category: OrderedDict [{
                product.attribute: OrderedDict [{
                    product: [product.template.attribute.value]
                }]
            }]
        }]
        """
        attributes = self.product_tmpl_id.valid_product_template_attribute_line_ids.attribute_id.sorted()
        categories = OrderedDict([(cat, OrderedDict()) for cat in attributes.category_id.sorted()])
        if any(not pa.category_id for pa in attributes):
            # category_id is not required and the mapped does not return empty
            categories[self.env['product.attribute.category']] = OrderedDict()
        for pa in attributes:
            categories[pa.category_id][pa] = OrderedDict([(
                product,
                product.attribute_line_ids.filtered(lambda ptal: ptal.attribute_id == pa).value_ids
            ) for product in self])
        return categories

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
from . import website_sale_comparison
```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_product_attribute_category_public,product.attribute.category public,model_product_attribute_category,,1,0,0,0
access_product_attribute_category_public_saleman,product.attribute.category sale manager,model_product_attribute_category,sales_team.group_sale_manager,1,1,1,1
```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#DA956B"/><stop offset="100%" stop-color="#CC7039"/></linearGradient><path id="d" d="M23 27.281L27 43l1 4h19c1 0 1 2 0 2H26l-6-24h-2v1c0 .667-.333 1-1 1s-1-.333-1-1v-2c.066-.667.4-1 1-1h4c.517 0 .85.333 1 1l1 3.281zM45.5 55a2.5 2.5 0 1 1 0-5 2.5 2.5 0 0 1 0 5zm-19 0a2.5 2.5 0 1 1 0-5 2.5 2.5 0 0 1 0 5zM50 37.299v2.449c0 .11-.038.206-.115.287a.363.363 0 0 1-.272.121H32.977v2.45c0 .11-.038.206-.114.287a.363.363 0 0 1-.272.121.442.442 0 0 1-.29-.128l-3.857-4.082a.393.393 0 0 1-.11-.28.41.41 0 0 1 .11-.294l3.868-4.083a.366.366 0 0 1 .279-.114c.104 0 .195.04.272.12.076.082.114.177.114.288v2.45h16.636c.105 0 .196.04.272.12.077.081.115.177.115.288zm0-7.199a.41.41 0 0 1-.109.294l-3.869 4.082a.366.366 0 0 1-.278.115.363.363 0 0 1-.272-.121.403.403 0 0 1-.115-.287v-2.45H28.722a.363.363 0 0 1-.272-.12.403.403 0 0 1-.115-.288v-2.45c0-.11.038-.206.115-.286a.363.363 0 0 1 .272-.122h16.635v-2.449a.41.41 0 0 1 .11-.293.366.366 0 0 1 .277-.115c.097 0 .194.042.29.127l3.857 4.07A.41.41 0 0 1 50 30.1z"/><path id="e" d="M23 25.281L27 41l1 3h19c1 0 1 3 0 3H26l-6-24h-2v1c0 .667-.333 1-1 1s-1-.333-1-1v-2c.066-.667.4-1 1-1h4c.517 0 .85.333 1 1l1 3.281zM45.5 53a2.5 2.5 0 1 1 0-5 2.5 2.5 0 0 1 0 5zm-19 0a2.5 2.5 0 1 1 0-5 2.5 2.5 0 0 1 0 5zM50 34.689v2.45c0 .11-.038.205-.115.286a.363.363 0 0 1-.272.121H32.977v2.45c0 .11-.038.206-.114.287a.363.363 0 0 1-.272.121.442.442 0 0 1-.29-.127l-3.857-4.083a.393.393 0 0 1-.11-.28.41.41 0 0 1 .11-.294l3.868-4.082a.366.366 0 0 1 .279-.115c.104 0 .195.04.272.121.076.08.114.176.114.287v2.45h16.636c.105 0 .196.04.272.12.077.082.115.177.115.288zm0-7.198a.41.41 0 0 1-.109.293l-3.869 4.083a.366.366 0 0 1-.278.114.363.363 0 0 1-.272-.12.403.403 0 0 1-.115-.288v-2.45H28.722a.363.363 0 0 1-.272-.12.403.403 0 0 1-.115-.288v-2.449c0-.11.038-.206.115-.287a.363.363 0 0 1 .272-.121h16.635v-2.45a.41.41 0 0 1 .11-.293.366.366 0 0 1 .277-.115c.097 0 .194.043.29.128l3.857 4.07a.41.41 0 0 1 .109.293z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M32.073 69H4c-2 0-4-1-4-4V38.16l16.339-16.914L21 21l3.218 10.05 4.433-4.928h14.216l2.738-2.955 4.233 4.435-5.877 6.823c3.956-.137 5.915-.137 5.877 0-.013.046-.013 1.01 0 2.894L43.961 45H47l.709.999-3.337 3.855 2.886 2.366L32.073 69z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
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
        $parent.find('a.a-submit').toggleClass('disabled', !isCombinationPossible);
    },
});

```

## File: static\src\js\website_sale_comparison.js

```javascript
odoo.define('website_sale_comparison.comparison', function (require) {
'use strict';

var concurrency = require('web.concurrency');
var core = require('web.core');
var publicWidget = require('web.public.widget');
const {getCookie, setCookie} = require('web.utils.cookies');
var VariantMixin = require('sale.VariantMixin');
var website_sale_utils = require('website_sale.utils');
const cartHandlerMixin = website_sale_utils.cartHandlerMixin;

var qweb = core.qweb;
var _t = core._t;

// VariantMixin events are overridden on purpose here
// to avoid registering them more than once since they are already registered
// in website_sale.js
var ProductComparison = publicWidget.Widget.extend(VariantMixin, {
    template: 'product_comparison_template',
    events: {
        'click .o_product_panel_header': '_onClickPanelHeader',
    },

    /**
     * @constructor
     */
    init: function () {
        this._super.apply(this, arguments);

        this.product_data = {};
        this.comparelist_product_ids = JSON.parse(getCookie('comparelist_product_ids') || '[]');
        this.product_compare_limit = 4;
        this.guard = new concurrency.Mutex();
    },
    /**
     * @override
     */
    start: function () {
        var self = this;

        self._loadProducts(this.comparelist_product_ids).then(function () {
            self._updateContent('hide');
        });
        self._updateComparelistView();

        $('#comparelist .o_product_panel_header').popover({
            trigger: 'manual',
            animation: true,
            html: true,
            title: function () {
                return _t("Compare Products");
            },
            container: '.o_product_feature_panel',
            placement: 'top',
            template: qweb.render('popover'),
            content: function () {
                return $('#comparelist .o_product_panel_content').html();
            }
        });
        // We trigger a resize to launch the event that checks if this element hides
        // a button when the page is loaded.
        $(window).trigger('resize');

        $(document.body).on('click.product_comparaison_widget', '.comparator-popover .o_comparelist_products .o_remove', function (ev) {
            ev.preventDefault();
            self._removeFromComparelist(ev);
        });
        $(document.body).on('click.product_comparaison_widget', '.o_comparelist_remove', function (ev) {
            self._removeFromComparelist(ev);
            self.guard.exec(function() {
                const newLink = '/shop/compare?products=' + encodeURIComponent(self.comparelist_product_ids);
                window.location.href = _.isEmpty(self.comparelist_product_ids) ? '/shop' : newLink;
            });
        });

        return this._super.apply(this, arguments);
    },
    /**
     * @override
     */
    destroy: function () {
        this._super.apply(this, arguments);
        $(document.body).off('.product_comparaison_widget');
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * @param {jQuery} $elem
     */
    handleCompareAddition: function ($elem) {
        var self = this;
        if (this.comparelist_product_ids.length < this.product_compare_limit) {
            var productId = $elem.data('product-product-id');
            if ($elem.hasClass('o_add_compare_dyn')) {
                productId = $elem.parent().find('.product_id').val();
                if (!productId) { // case List View Variants
                    productId = $elem.parent().find('input:checked').first().val();
                }
            }

            let $form = $elem.closest('form');
            $form = $form.length ? $form : $('#product_details > form');
            this.selectOrCreateProduct(
                $form,
                productId,
                $form.find('.product_template_id').val(),
                false
            ).then(function (productId) {
                productId = parseInt(productId, 10) || parseInt($elem.data('product-product-id'), 10);
                if (!productId) {
                    return;
                }
                self._addNewProducts(productId).then(function () {
                    website_sale_utils.animateClone(
                        $('#comparelist .o_product_panel_header'),
                        $elem.closest('form'),
                        -50,
                        10
                    );
                });
            });
        } else {
            this.$('.o_comparelist_limit_warning').show();
            $('#comparelist .o_product_panel_header').popover('show');
        }
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _loadProducts: function (product_ids) {
        var self = this;
        return this._rpc({
            route: '/shop/get_product_data',
            params: {
                product_ids: product_ids,
                cookies: JSON.parse(getCookie('comparelist_product_ids') || '[]'),
            },
        }).then(function (data) {
            self.comparelist_product_ids = JSON.parse(data.cookies);
            delete data.cookies;
            _.each(data, function (product) {
                self.product_data[product.product.id] = product;
            });
            if (product_ids.length > Object.keys(data).length) {
                /* If some products have been archived
                they are not displayed but the count & cookie
                need to be updated.
                */
                self._updateCookie();
            }
        });
    },
    /**
     * @private
     */
    _togglePanel: function () {
        $('#comparelist .o_product_panel_header').popover('toggle');
    },
    /**
     * @private
     */
    _addNewProducts: function (product_id) {
        return this.guard.exec(this._addNewProductsImpl.bind(this, product_id));
    },
    _addNewProductsImpl: function (product_id) {
        var self = this;
        $('.o_product_feature_panel').addClass('d-md-block');
        if (!_.contains(self.comparelist_product_ids, product_id)) {
            self.comparelist_product_ids.push(product_id);
            if (_.has(self.product_data, product_id)){
                self._updateContent();
            } else {
                return self._loadProducts([product_id]).then(function () {
                    self._updateContent();
                    self._updateCookie();
                });
            }
        }
        self._updateCookie();
    },
    /**
     * @private
     */
    _updateContent: function (force) {
        var self = this;
        this.$('.o_comparelist_products .o_product_row').remove();
        _.each(this.comparelist_product_ids, function (res) {
            if (self.product_data.hasOwnProperty(res)) {
                // It is possible that we do not have the required product_data for all IDs in
                // comparelist_product_ids
                var $template = self.product_data[res].render;
                self.$('.o_comparelist_products').append($template);
            }
        });
        if (force !== 'hide' && (this.comparelist_product_ids.length > 1 || force === 'show')) {
            $('#comparelist .o_product_panel_header').popover('show');
        }
        else {
            $('#comparelist .o_product_panel_header').popover('hide');
        }
    },
    /**
     * @private
     */
    _removeFromComparelist: function (e) {
        this.guard.exec(this._removeFromComparelistImpl.bind(this, e));
    },
    _removeFromComparelistImpl: function (e) {
        var target = $(e.target.closest('.o_comparelist_remove, .o_remove'));
        this.comparelist_product_ids = _.without(this.comparelist_product_ids, target.data('product_product_id'));
        target.parents('.o_product_row').remove();
        this._updateCookie();
        $('.o_comparelist_limit_warning').hide();
        this._updateContent('show');
    },
    /**
     * @private
     */
    _updateCookie: function () {
        setCookie('comparelist_product_ids', JSON.stringify(this.comparelist_product_ids), 24 * 60 * 60 * 365, 'required');
        this._updateComparelistView();
    },
    /**
     * @private
     */
    _updateComparelistView: function () {
        this.$('.o_product_circle').text(this.comparelist_product_ids.length);
        this.$('.o_comparelist_button').removeClass('d-md-block');
        if (_.isEmpty(this.comparelist_product_ids)) {
            $('.o_product_feature_panel').removeClass('d-md-block');
        } else {
            $('.o_product_feature_panel').addClass('d-md-block');
            this.$('.o_comparelist_products').addClass('d-md-block');
            if (this.comparelist_product_ids.length >=2) {
                this.$('.o_comparelist_button').addClass('d-md-block');
                this.$('.o_comparelist_button a').attr('href',
                    '/shop/compare?products=' + encodeURIComponent(this.comparelist_product_ids));
            }
        }
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onClickPanelHeader: function () {
        this._togglePanel();
    },
});

publicWidget.registry.ProductComparison = publicWidget.Widget.extend(cartHandlerMixin, {
    selector: '.js_sale',
    events: {
        'click .o_add_compare, .o_add_compare_dyn': '_onClickAddCompare',
        'click #o_comparelist_table tr': '_onClickComparelistTr',
        'submit .o_add_cart_form_compare': '_onFormSubmit',
    },

    /**
     * @override
     */
    start: function () {
        var def = this._super.apply(this, arguments);
        this.productComparison = new ProductComparison(this);
        this.getRedirectOption();
        return Promise.all([def, this.productComparison.appendTo(this.$el)]);
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {Event} ev
     */
    _onClickAddCompare: function (ev) {
        this.productComparison.handleCompareAddition($(ev.currentTarget));
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onClickComparelistTr: function (ev) {
        var $target = $(ev.currentTarget);
        $($target.data('target')).children().slideToggle(100);
        $target.find('.fa-chevron-circle-down, .fa-chevron-circle-right').toggleClass('fa-chevron-circle-down fa-chevron-circle-right');
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onFormSubmit(ev) {
        ev.preventDefault();
        const $form = $(ev.currentTarget);
        const cellIndex = $(ev.currentTarget).closest('td')[0].cellIndex;
        this.getCartHandlerOptions(ev);
        // Override product image container for animation. 
        this.$itemImgContainer = this.$('#o_comparelist_table tr').first().find('td').eq(cellIndex);
        const $inputProduct = $form.find('input[type="hidden"][name="product_id"]').first();
        const productId = parseInt($inputProduct.val());
        if (productId) {
            const productTrackingInfo = $inputProduct.data('product-tracking-info');
            if (productTrackingInfo) {
                productTrackingInfo.quantity = 1;
                $inputProduct.trigger('add_to_cart_event', [productTrackingInfo]);
            }
            return this.addToCart(this._getAddToCartParams(productId, $form));
        }
    },
    /**
     * Get the addToCart Params
     *
     * @param {number} productId
     * @param {JQuery} $form
     * @override
     */
    _getAddToCartParams(productId, $form) {
        return {
            product_id: productId,
            add_qty: 1,
        };
    }
});
return ProductComparison;
});

```

## File: static\src\xml\comparison.xml

```xml
<templates id="compare_products" xml:space="preserve">

    <t t-name="product_comparison_template">
        <div class="o_product_feature_panel d-none css_editable_mode_hidden o_bottom_fixed_element bg-white rounded-top border-primary border-bottom-0 px-3 py-2">
            <span class="o_product_panel" id="comparelist">
                <span class="o_product_panel_header text-center">
                    <span class="o_product_icon"><i class="fa fa-exchange" role="img" aria-label="Product" title="Product"></i></span>
                    <span class="o_product_text">Compare</span>
                    <span class="o_product_circle o_animate_blink badge text-bg-primary">0</span>
                </span>
                <span class="o_product_panel_content">
                    <div class="o_comparelist_products">
                        <div class="o_comparelist_limit_warning" style="display:none">
                            <div class="o_shortlog alert alert-warning" role="alert">
                                <span><i class="fa fa-warning text-danger" role="img" aria-label="Warning" title="Warning"></i> You can compare max 4 products.</span>
                            </div>
                        </div>
                    </div>
                    <div class="o_comparelist_button" style='display:none'>
                        <a role="button" class="btn btn-primary d-block" href="#"><i class="fa fa-exchange me-2"/>Compare</a>
                    </div>
                </span>
            </span>
        </div>
    </t>

    <t t-name="popover">
        <div style="width:600px;" class="popover comparator-popover" role="tooltip">
            <div class="arrow"/>
            <h3 class="popover-header"/>
            <div class="popover-body"/>
        </div>
    </t>

</templates>

```

## File: views\snippets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="snippet_options" inherit_id="website.snippet_options" name="Comparison Snippet Options">
    <xpath expr="//div[@data-js='WebsiteSaleProductPage']//we-row[@data-name='o_wsale_buy_now_opt']" position="after">
        <we-select string="Specification" data-no-preview="true" data-reload="/">
            <we-button data-customize-website-views="">None</we-button>
            <we-button data-customize-website-views="website_sale_comparison.product_attributes_body">Bottom of Page</we-button>
        </we-select>
    </xpath>
    <xpath expr="//we-button[hasclass('o_we_add_to_cart_btn')]" position="after">
        <we-button title="Compare" class="fa fa-fw fa-exchange"
                   data-customize-website-views="website_sale_comparison.add_to_compare"
                   data-no-preview="true"
                   data-reload="/"/>
    </xpath>
    <xpath expr="//we-button[hasclass('o_we_buy_now_btn')]" position="after">
        <we-button title="Compare" class="fa fa-fw fa-exchange"
                   data-customize-website-views="website_sale_comparison.product_add_to_compare"
                   data-no-preview="true"
                   data-reload="/"/>
    </xpath>
</template>

</odoo>

```

## File: views\website_sale_comparison_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="add_to_compare" inherit_id="website_sale.products_item" name="Comparison List" priority="22">
        <xpath expr="//div[hasclass('o_wsale_product_btn')]" position="inside">
            <t t-set="categories" t-value="product.valid_product_template_attribute_line_ids._prepare_categories_for_display()"/>
            <t t-set="product_variant_id" t-value="product._get_first_possible_variant_id()"/>
            <button t-if="product_variant_id and categories" type="button" role="button" class="d-none d-md-inline-block btn btn-outline-primary bg-white o_add_compare" title="Compare" aria-label="Compare" t-att-data-product-product-id="product_variant_id" data-action="o_comparelist"><span class="fa fa-exchange"></span></button>
        </xpath>
    </template>

    <template id="product_add_to_compare" name='Add to comparison in product page' inherit_id="website_sale.product" priority="8">
        <xpath expr="//div[@id='o_wsale_cta_wrapper']" position="after">
            <t t-set="categories" t-value="product.valid_product_template_attribute_line_ids._prepare_categories_for_display()"/>
            <t t-set="product_variant_id" t-value="product._get_first_possible_variant_id()"/>
            <button t-if="product_variant_id and categories"
                type="button"
                role="button"
                class="d-none d-md-block btn btn-link px-0 o_add_compare_dyn"
                aria-label="Compare"
                t-att-data-product-product-id="product_variant_id"
                data-action="o_comparelist">
                    <span class="fa fa-exchange me-2"/>Compare
            </button>
        </xpath>
    </template>

    <template id="product_attributes_body" inherit_id="website_sale.product" name="Product attributes table">
        <xpath expr="//div[@id='product_attributes_simple']" position="replace"/>
        <xpath expr="//div[@id='product_full_description']" position="after">
            <t t-set="categories" t-value="product.valid_product_template_attribute_line_ids._prepare_categories_for_display()"/>
            <t t-if="categories">
                <section class="pt32 pb32" id="product_full_spec">
                    <div class="container">
                        <div class="d-flex justify-content-between align-items-center mb-4">
                            <h3 class="m-0">Specifications</h3>
                        </div>
                        <div id="product_specifications">
                            <div class="row">
                                <t t-foreach="categories" t-as="category">
                                    <div class="col-lg-6">
                                        <table class="table">
                                            <t t-if="len(categories) > 1">
                                                <tr>
                                                    <th class="text-start" t-att-colspan="2">
                                                        <span t-if="category" t-field="category.name"/>
                                                        <span t-else="">Uncategorized</span>
                                                    </th>
                                                </tr>
                                            </t>
                                            <tr t-foreach="categories[category].filtered(lambda l: len(l.value_ids) > 1)" t-as="ptal">
                                                <td class="w-25"><span t-field="ptal.attribute_id.name"/></td>
                                                <td class="w-75 text-muted">
                                                    <t t-foreach="ptal.value_ids" t-as="pav">
                                                        <span t-field="pav.name"/><t t-if="not pav_last"> or</t>
                                                    </t>
                                                </td>
                                            </tr>
                                            <t t-set="single_value_attributes" t-value="categories[category]._prepare_single_value_for_display()"/>
                                            <tr t-foreach="single_value_attributes" t-as="attribute">
                                                <td class="w-25"><span t-field="attribute.name"/></td>
                                                <td class="w-75 text-muted">
                                                    <t t-foreach="single_value_attributes[attribute]" t-as="ptal">
                                                        <span t-field="ptal.product_template_value_ids._only_active().name"/><t t-if="not ptal_last">, </t>
                                                    </t>
                                                </td>
                                            </tr>
                                        </table>
                                    </div>
                                </t>
                            </div>
                        </div>
                    </div>
                </section>
            </t>
        </xpath>
    </template>

    <template id="product_compare" name="Comparator Page">
        <t t-call="website.layout">
            <t t-set="additional_title">Shop Comparator</t>
            <div id="wrap" class="js_sale">
                <div class="oe_structure oe_empty" id="oe_structure_website_sale_comparison_product_compare_1"/>
                <div class="container oe_website_sale pt-3">
                    <section class="container">
                        <h3>Compare Products</h3>
                        <table class="table table-bordered table-hover text-center mt16 table-comparator" id="o_comparelist_table">
                            <t t-set="categories" t-value="products._prepare_categories_for_display()"/>
                            <thead>
                                <tr>
                                    <td t-if="len(categories)" class='o_ws_compare_image td-top-left border-bottom-0'/>
                                    <td t-foreach="products" t-as="product" class="o_ws_compare_image position-relative border-bottom-0">
                                        <a href="#" t-att-data-product_product_id="product.id" class="o_comparelist_remove" t-if="len(products) &gt; 2">
                                            <strong>x</strong>
                                        </a>
                                        <a t-att-href="product.website_url">
                                            <img t-attf-src="/web/image/product.product/#{product.id}/image_256" class="img img-fluid" style="margin:auto;" alt="Product image"/>
                                        </a>
                                    </td>
                                </tr>
                                <tr>
                                    <td t-if="len(categories)" class='td-top-left border-top-0'/>
                                    <td t-foreach="products" t-as="product" class="border-top-0">
                                        <t t-set="combination_info" t-value="product._get_combination_info_variant()"/>
                                        <div class='product_summary'>
                                            <a class="o_product_comparison_table" t-att-href="product.website_url">
                                                <span t-esc="combination_info['display_name']"></span><br/>
                                            </a>

                                            <span class="o_comparison_price" t-if="combination_info['prevent_zero_price_sale']">
                                                <strong t-field="website.prevent_zero_price_sale_text"/>
                                            </span>
                                            <span class="o_comparison_price" t-else="">
                                                <strong>Price:</strong>
                                                <del t-attf-class="text-danger mr8 {{'' if combination_info['has_discounted_price'] else 'd-none'}}"
                                                     style="white-space: nowrap;" t-esc="combination_info['list_price']"
                                                     t-options="{'widget': 'monetary', 'display_currency': website.currency_id}"/>
                                                <span t-esc="combination_info['price']"
                                                      t-options="{'widget': 'monetary', 'display_currency': website.currency_id}"/>
                                                <small class="d-block text-muted" groups="website_sale.group_show_uom_price" t-if="combination_info['base_unit_price']">
                                                    <t t-call='website_sale.base_unit_price'/>
                                                </small>
                                            </span>

                                            <form action="/shop/cart/update" method="post" class="text-center o_add_cart_form_compare">
                                                <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
                                                <input name="product_id" t-att-value="product.id" type="hidden"
                                                       t-att-data-product-tracking-info="json.dumps(request.env['product.template'].get_google_analytics_data(combination_info))"/>
                                                <a t-if="combination_info['prevent_zero_price_sale']" t-att-href="website.contact_us_button_url" class="btn btn-primary btn_cta">Contact Us</a>
                                                <a t-else="" role="button" class="btn btn-primary a-submit" href="#">
                                                    <i class="fa fa-shopping-cart me-2"/>Add to Cart
                                                </a>
                                            </form>
                                        </div>
                                    </td>
                                </tr>
                            </thead>
                            <tbody>
                                <t t-foreach="categories" t-as="category">
                                    <t t-if="len(categories) &gt; 1">
                                        <tr class="clickable" data-bs-toggle="collapse" t-att-data-bs-target="'.o_ws_category_%d' % category.id">
                                            <th class="text-start" t-att-colspan="len(products) + 1"><i class="fa fa-chevron-circle-down o_product_comparison_collpase" role="img" aria-label="Collapse" title="Collapse"></i><span t-if="category" t-field="category.name"/><span t-else="">Uncategorized</span></th>
                                        </tr>
                                    </t>
                                    <tr t-foreach="categories[category]" t-as="attribute" t-att-class="'collapse show o_ws_category_%d' % category.id">
                                        <td><span t-field="attribute.name"/></td>
                                        <td t-foreach="categories[category][attribute]" t-as="product">
                                            <t t-foreach="categories[category][attribute][product]" t-as="ptav">
                                                <span t-field="ptav.name"/><t t-if="not ptav_last">, </t>
                                            </t>
                                        </td>
                                    </tr>
                                </t>
                            </tbody>
                        </table>
                    </section>
                </div>
                <div class="oe_structure" id="oe_structure_website_sale_comparison_product_compare_2"/>
            </div>
        </t>
    </template>

    <template id="product_product" name="Comparator - Product row in comparator popover">
        <t t-set="combination_info" t-value="product._get_combination_info_variant()"/>
        <div class="row g-0 align-items-center my-1 o_product_row" t-att-data-category_ids="product.public_categ_ids.ids">
            <div class="col-3 text-center">
                <img class="img o_image_64_max" t-att-src="website.image_url(product, 'image_128')" alt="Product image"/>
            </div>
            <div class="col-8 ps-2">
                <h6>
                    <a t-att-href="product.website_url"><t t-esc="combination_info['display_name']" /></a><br/>
                    <div t-attf-class="{{'d-none' if combination_info['prevent_zero_price_sale'] else ''}}">
                        <del t-attf-class="text-danger mr8 {{'' if combination_info['has_discounted_price'] else 'd-none'}}" style="white-space: nowrap;" t-esc="combination_info['list_price']" t-options="{'widget': 'monetary', 'display_currency': website.currency_id}" />
                        <span t-esc="combination_info['price']" t-options="{'widget': 'monetary', 'display_currency': website.currency_id}"/>
                    </div>
                    <div t-attf-class="{{'' if combination_info['prevent_zero_price_sale'] else 'd-none'}}" t-field="website.prevent_zero_price_sale_text"/>
                </h6>
            </div>
            <div class="col-1 text-end">
                <a href='#' class="o_remove" title="Remove" t-att-data-product_product_id="product.id">
                    <i class="fa fa-trash" role="img" aria-label="Remove"></i>
                </a>
            </div>
        </div>
    </template>

</odoo>

```

## File: views\website_sale_comparison_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="product_attribute_category_tree_view" model="ir.ui.view">
        <field name="name">product.attribute.category.tree</field>
        <field name="model">product.attribute.category</field>
        <field name="arch" type="xml">
            <tree string="Product Attribute Category" editable="bottom">
                <field name="sequence" widget="handle"/>
                <field name="name"/>
                <field name="attribute_ids" widget="many2many_tags" options="{'no_create': True}"/>
            </tree>
        </field>
    </record>

    <record id="product_attribute_category_action" model="ir.actions.act_window">
        <field name="name">Attribute Categories</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">product.attribute.category</field>
        <field name="view_mode">tree</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a new attribute category
            </p><p>
                Group attributes by category that will appear in the specification
                part of a product page.
            </p>
        </field>
    </record>

    <menuitem action="product_attribute_category_action"
        id="menu_attribute_category_action"
        parent="website_sale.menu_catalog" groups="base.group_no_one" sequence="11"/>

    <record id="product_attribute_tree_view_inherit" model="ir.ui.view">
        <field name="name">product.attribute.tree.inherit</field>
        <field name="model">product.attribute</field>
        <field name="inherit_id" ref="product.attribute_tree_view"/>
        <field name="arch" type="xml">
            <field name="name" position="after">
               <field name="category_id"/>
            </field>
        </field>
    </record>

    <record id="product_attribute_view_form" model="ir.ui.view">
        <field name="name">product.attribute.form.inherit</field>
        <field name="model">product.attribute</field>
        <field name="inherit_id" ref="product.product_attribute_view_form"/>
        <field name="priority" eval="8"/>
        <field name="arch" type="xml">
            <field name='name' position='after'>
                <field name="category_id"/>
            </field>
        </field>
    </record>

</odoo>

```

