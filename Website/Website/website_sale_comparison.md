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
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json

from odoo.http import Controller, request, route


class WebsiteSaleProductComparison(Controller):

    @route('/shop/compare', type='http', auth='public', website=True, sitemap=False)
    def product_compare(self, **post):
        product_ids = [int(i) for i in post.get('products', '').split(',') if i.isdigit()]
        if not product_ids:
            return request.redirect('/shop')

        # use search to check read access on each record/ids
        products = request.env['product.product'].search([('id', 'in', product_ids)])
        return request.render(
            'website_sale_comparison.product_compare',
            {
                'products': products.with_context(display_default_code=False),
            }
        )

    @route('/shop/get_product_data', type='json', auth='public', website=True)
    def get_product_data(self, product_ids, cookies=None):
        ret = {}

        website = request.env['website'].get_current_website()
        products = request.env['product.product'].search([('id', 'in', product_ids)])

        if cookies is not None:
            ret['cookies'] = json.dumps(
                request.env['product.product'].search([
                    ('id', 'in', list(set(product_ids + cookies)))
                ]).ids
            )

        products = products.with_context(display_default_code=False)
        for product in products:
            ret[product.id] = {
                'render': request.env['ir.ui.view']._render_template(
                    'website_sale_comparison.product_product',
                    {'product': product, 'website': website}
                ),
                'product': dict(id=product.id, name=product.name, display_name=product.display_name),
            }
        return ret

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

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
        <field name="value_ids" eval="[(6,0,[ref('website_sale_comparison.product_attribute_value_1')])]"/>
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

## File: models\product_attribute.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ProductAttribute(models.Model):
    _inherit = 'product.attribute'
    _order = 'category_id, sequence, id'

    category_id = fields.Many2one(
        comodel_name='product.attribute.category',
        string="eCommerce Category",
        index=True,
        help="Set a category to regroup similar attributes under the same section in the Comparison"
             " page of eCommerce.",
    )

```

## File: models\product_attribute_category.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ProductAttributeCategory(models.Model):
    _name = "product.attribute.category"
    _description = "Product Attribute Category"
    _order = 'sequence, id'

    name = fields.Char("Category Name", required=True, translate=True)
    sequence = fields.Integer("Sequence", default=10, index=True)

    attribute_ids = fields.One2many('product.attribute', 'category_id', string="Related Attributes", domain="[('category_id', '=', False)]")

```

## File: models\product_product.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import OrderedDict

from odoo import models


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

## File: models\product_template_attribute_line.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import OrderedDict

from odoo import models


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

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import product_attribute
from . import product_attribute_category
from . import product_product
from . import product_template_attribute_line

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_product_attribute_category_public_public,product.attribute.category public,model_product_attribute_category,base.group_public,1,0,0,0
access_product_attribute_category_public_portal,product.attribute.category public,model_product_attribute_category,base.group_portal,1,0,0,0
access_product_attribute_category_public_employee,product.attribute.category public,model_product_attribute_category,base.group_user,1,0,0,0
access_product_attribute_category_public_saleman,product.attribute.category sale manager,model_product_attribute_category,sales_team.group_sale_manager,1,1,1,1
```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path fill-rule="evenodd" clip-rule="evenodd" d="M15.724 6.397C16.377 4.94 17.852 4 19.481 4h11.037c1.63 0 3.104.94 3.757 2.397L37.236 13H41.9c2.46 0 4.367 2.099 4.07 4.481l-3.106 25C42.613 44.49 40.866 46 38.793 46H11.207c-2.074 0-3.82-1.51-4.07-3.519l-3.107-25C3.734 15.1 5.64 13 8.1 13h4.663l2.961-6.603ZM32.917 13H17.082c0-.56.123-1.134.39-1.691l.956-2C19.102 7.9 20.551 7 22.144 7h5.711c1.593 0 3.042.9 3.716 2.308l.957 2c.266.558.39 1.132.39 1.692Z" fill="#F78613"/><path fill-rule="evenodd" clip-rule="evenodd" d="M8.514 45.016a3.963 3.963 0 0 1-1.377-2.535l-3.107-25C3.734 15.1 5.64 13 8.1 13h4.663l2.961-6.603C16.377 4.94 17.852 4 19.481 4h11.037c1.63 0 3.104.94 3.757 2.397l2.59 5.777C35.5 28.256 23.848 41.405 8.515 45.016ZM17.082 13h15.835c0-.56-.123-1.134-.39-1.691l-.956-2C30.897 7.9 29.448 7 27.855 7h-5.711c-1.593 0-3.042.9-3.716 2.308l-.956 2a3.904 3.904 0 0 0-.39 1.692Z" fill="#FBB945"/><path d="m14.691 24.973 11.976-6.914L25.4 22.78l7.942 2.128a4 4 0 0 1 2.829 4.899l-.23.858-21.25-5.694Zm20.619 8.055-11.976 6.914 1.265-4.722-7.942-2.128a4 4 0 0 1-2.828-4.9l.23-.858 21.25 5.694Z" fill="#fff"/></svg>

```

## File: static\src\js\website_sale.js

```javascript
/** @odoo-module **/

import { WebsiteSale } from '@website_sale/js/website_sale';

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
/** @odoo-module **/

import { Mutex } from "@web/core/utils/concurrency";
import publicWidget from "@web/legacy/js/public/public_widget";
import { cookie } from "@web/core/browser/cookie";;
import VariantMixin from "@website_sale/js/sale_variant_mixin";
import website_sale_utils from "@website_sale/js/website_sale_utils";
import { _t } from "@web/core/l10n/translation";
import { rpc } from "@web/core/network/rpc";
import { renderToString } from "@web/core/utils/render";

const cartHandlerMixin = website_sale_utils.cartHandlerMixin;

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
        this.comparelist_product_ids = JSON.parse(cookie.get('comparelist_product_ids') || '[]');
        this.product_compare_limit = 4;
        this.guard = new Mutex();
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
            template: renderToString('popover'),
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
                window.location.href = Object.keys(self.comparelist_product_ids || {}).length === 0 ? '/shop' : newLink;
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
        const cookies = JSON.parse(cookie.get('comparelist_product_ids') || '[]');
        if (product_ids.length == 0 && cookies.length == 0) {
            return Promise.resolve(true);
        }
        return rpc('/shop/get_product_data', {
            product_ids: product_ids,
            cookies: cookies,
        }).then(function (data) {
            self.comparelist_product_ids = JSON.parse(data.cookies);
            delete data.cookies;
            Object.values(data).forEach((product) => {
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
        if (!self.comparelist_product_ids.includes(product_id)) {
            self.comparelist_product_ids.push(product_id);
            if (Object.prototype.hasOwnProperty.call(self.product_data, product_id)) {
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
        this.comparelist_product_ids.forEach((res) => {
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
        this.comparelist_product_ids = this.comparelist_product_ids.filter(
            (comp) => comp !== target.data("product_product_id")
        );
        target.parents('.o_product_row').remove();
        this._updateCookie();
        $('.o_comparelist_limit_warning').hide();
        this._updateContent('show');
    },
    /**
     * @private
     */
    _updateCookie: function () {
        cookie.set('comparelist_product_ids', JSON.stringify(this.comparelist_product_ids), 24 * 60 * 60 * 365, 'required');
        this._updateComparelistView();
    },
    /**
     * @private
     */
    _updateComparelistView: function () {
        this.$('.o_product_circle').text(this.comparelist_product_ids.length);
        this.$('.o_comparelist_button').removeClass('d-md-block');
        if (Object.keys(this.comparelist_product_ids || {}).length === 0) {
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
export default ProductComparison;

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
    <xpath expr="//div[@data-js='WebsiteSaleProductPage']//we-checkbox[@data-name='o_wsale_accordion_item']" position="after">
        <we-select string="Specification" data-no-preview="true" data-reload="/">
            <we-button data-customize-website-views="">None</we-button>
            <we-button data-customize-website-views="website_sale_comparison.product_attributes_body">Bottom of Page</we-button>
            <we-button data-customize-website-views="website_sale_comparison.accordion_specs_item">In accordion</we-button>
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
            <t t-set="attrib_categories" t-value="product.valid_product_template_attribute_line_ids._prepare_categories_for_display()"/>
            <t t-set="product_variant_id" t-value="product._get_first_possible_variant_id()"/>
            <button
                t-if="product_variant_id and attrib_categories"
                type="button"
                role="button"
                class="d-none d-md-inline-block btn btn-light o_add_compare"
                title="Compare"
                aria-label="Compare"
                t-att-data-product-product-id="product_variant_id"
                data-action="o_comparelist"
            >
                <span class="fa fa-exchange"/>
            </button>
        </xpath>
    </template>

    <template id="product_add_to_compare" name='Add to comparison in product page' inherit_id="website_sale.product" priority="8">
        <xpath expr="//div[@id='o_wsale_cta_wrapper']" position="after">
            <t t-set="attrib_categories" t-value="product.valid_product_template_attribute_line_ids._prepare_categories_for_display()"/>
            <t t-set="product_variant_id" t-value="product._get_first_possible_variant_id()"/>
            <button t-if="product_variant_id and attrib_categories"
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
            <t t-set="attrib_categories" t-value="product.valid_product_template_attribute_line_ids._prepare_categories_for_display()"/>
            <t t-if="attrib_categories">
                <section class="pt32 pb32" id="product_full_spec">
                    <div class="container">
                        <div class="d-flex justify-content-between align-items-center mb-4">
                            <h3 class="m-0">Specifications</h3>
                        </div>
                        <div id="product_specifications">
                            <div class="row">
                                <t t-foreach="attrib_categories" t-as="category">
                                    <div class="col-lg-6">
                                        <t t-call="website_sale_comparison.specifications_table"/>
                                    </div>
                                </t>
                                <t t-if="is_view_active('website_sale.product_tags')">
                                    <div class="col-lg-6">
                                        <table class="table">
                                            <t t-if="product.product_variant_ids.all_product_tag_ids">
                                                <tr>
                                                    <th class="text-start" t-att-colspan="2">
                                                        <span>Tags</span>
                                                    </th>
                                                </tr>
                                                <tr class="d-flex">
                                                    <td class="w-25 d-flex align-items-center"><span>Tags</span></td>
                                                    <td class="w-75 text-muted">
                                                        <t t-call="website_sale.product_tags">
                                                            <t t-set="all_product_tags" t-value="product.product_variant_ids.all_product_tag_ids"/>
                                                        </t>
                                                    </td>
                                                </tr>
                                            </t>
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

    <template
        id="accordion_specs_item"
        name="Specifications Accordion Item"
        inherit_id="website_sale.product_accordion"
        active="False"
    >
        <xpath expr="//div[@id='more_information_accordion_item']" position="before">
            <t t-if="product.valid_product_template_attribute_line_ids._prepare_categories_for_display()">
                <t t-foreach="attrib_categories" t-as="category">
                    <div class="accordion-item">
                        <div class="accordion-header my-0 h6">
                            <button
                                t-out="category.name"
                                class="accordion-button collapsed fw-medium"
                                type="button"
                                data-bs-toggle="collapse"
                                t-attf-data-bs-target="#category_accordion_{{category_index}}"
                                aria-expanded="false"
                                aria-controls="specifications"
                            >
                                <t t-if="category_size == 1">Specifications</t>
                                <t t-else="">Others</t>
                            </button>
                        </div>
                        <div
                            t-attf-id="category_accordion_{{category_index}}"
                            class="accordion-collapse collapse"
                            data-bs-parent="#product_accordion"
                        >
                            <div class="accordion-body pt-0">
                                <t t-call="website_sale_comparison.specifications_table">
                                    <t t-set="is_accordion" t-value="True"/>
                                </t>
                            </div>
                        </div>
                    </div>
                </t>
            </t>
        </xpath>
    </template>

    <template id="specifications_table" name="Specifications Table">
        <table t-attf-class="table {{is_accordion and 'table-sm mb-0'}}">
            <t t-if="len(attrib_categories) > 1 and not is_accordion">
                <tr>
                    <th class="text-start" colspan="2">
                        <span t-if="category" t-field="category.name"/>
                        <span t-else="">Others</span>
                    </th>
                </tr>
            </t>
            <tr
                t-foreach="attrib_categories[category].filtered(lambda l: len(l.value_ids) > 1)"
                t-as="ptal"
            >
                <t
                    t-set="hide_border_bottom_classes"
                    t-value="'border-bottom-0' if ptal_last and is_accordion else ''"
                />
                <td t-attf-class="w-25 {{hide_border_bottom_classes}} ps-0">
                    <span t-field="ptal.attribute_id.name"/>
                </td>
                <td t-attf-class="w-75 {{hide_border_bottom_classes}} pe-0 text-muted text-end">
                    <t t-foreach="ptal.value_ids" t-as="pav">
                        <span t-field="pav.name"/><t t-if="not pav_last">, </t>
                    </t>
                </td>
            </tr>
            <t
                t-set="single_value_attributes"
                t-value="attrib_categories[category]._prepare_single_value_for_display()"
            />
            <tr t-foreach="single_value_attributes" t-as="attribute">
                <t
                    t-set="hide_border_bottom_classes"
                    t-value="'border-bottom-0' if attribute_last and is_accordion else ''"
                />
                <td t-attf-class="w-25 {{hide_border_bottom_classes}} ps-0 ">
                    <span t-field="attribute.name"/>
                </td>
                <td t-attf-class="w-75 {{hide_border_bottom_classes}} pe-0 text-muted text-end">
                    <t t-foreach="single_value_attributes[attribute]" t-as="ptal">
                        <span t-field="ptal.product_template_value_ids._only_active().name"/><t t-if="not ptal_last">, </t>
                    </t>
                </td>
            </tr>
        </table>
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
                            <t t-set="attrib_categories" t-value="products._prepare_categories_for_display()"/>
                            <thead>
                                <tr>
                                    <td t-if="len(attrib_categories)" class='o_ws_compare_image td-top-left border-bottom-0'/>
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
                                    <td t-if="len(attrib_categories)" class='td-top-left border-top-0'/>
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
                                                <span t-out="combination_info['price']"
                                                      t-options="{'widget': 'monetary', 'display_currency': website.currency_id}"/>
                                                <del t-if="combination_info['compare_list_price'] and (combination_info['compare_list_price'] &gt; combination_info['price'])"
                                                     t-attf-class="text-muted mr8"
                                                     style="white-space: nowrap;"
                                                     t-esc="combination_info['compare_list_price']"
                                                     t-options="{'widget': 'monetary', 'display_currency': website.currency_id}" />
                                                <del t-else=""
                                                     t-attf-class="text-muted mr8 {{'' if combination_info['has_discounted_price'] else 'd-none'}}"
                                                     style="white-space: nowrap;"
                                                     t-out="combination_info['list_price']"
                                                     t-options="{'widget': 'monetary', 'display_currency': website.currency_id}"/>
                                                <small t-if="combination_info['base_unit_price']"
                                                       class="d-block text-muted"
                                                       groups="website_sale.group_show_uom_price">
                                                    <t t-call='website_sale.base_unit_price'/>
                                                </small>
                                            </span>

                                            <form action="/shop/cart/update" method="post" class="text-center o_add_cart_form_compare">
                                                <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
                                                <input name="product_id"
                                                       type="hidden"
                                                       t-att-value="product.id"
                                                       t-att-data-product-tracking-info="'product_tracking_info' in combination_info and json.dumps(combination_info['product_tracking_info'])"/>
                                                <a t-if="combination_info['prevent_zero_price_sale']"
                                                   t-att-href="website.contact_us_button_url"
                                                   class="btn btn-primary btn_cta">
                                                   Contact Us
                                                </a>
                                                <a t-else="" role="button" class="btn btn-primary a-submit" href="#">
                                                    <i class="fa fa-shopping-cart me-2"/>Add to Cart
                                                </a>
                                            </form>
                                        </div>
                                    </td>
                                </tr>
                            </thead>
                            <tbody>
                                <t t-foreach="attrib_categories" t-as="category">
                                    <t t-if="len(attrib_categories) &gt; 1">
                                        <tr class="clickable" data-bs-toggle="collapse" t-att-data-bs-target="'.o_ws_category_%d' % category.id">
                                            <th class="text-start" t-att-colspan="len(products) + 1"><i class="fa fa-chevron-circle-down o_product_comparison_collpase" role="img" aria-label="Collapse" title="Collapse"></i><span t-if="category" t-field="category.name"/><span t-else="">Uncategorized</span></th>
                                        </tr>
                                    </t>
                                    <tr t-foreach="attrib_categories[category]" t-as="attribute" t-att-class="'collapse show o_ws_category_%d' % category.id">
                                        <td><span t-field="attribute.name"/></td>
                                        <td t-foreach="attrib_categories[category][attribute]" t-as="product">
                                            <t t-foreach="attrib_categories[category][attribute][product]" t-as="ptav">
                                                <span t-field="ptav.name"/><t t-if="not ptav_last">, </t>
                                            </t>
                                        </td>
                                    </tr>
                                </t>
                                <t t-if="is_view_active('website_sale.product_tags') and any([product.all_product_tag_ids for product in products])">
                                    <tr class="clickable" data-bs-toggle="collapse" data-bs-target=".o_ws_tags">
                                        <th class="text-start" t-att-colspan="len(products) + 1">
                                            <i class="fa fa-chevron-circle-down o_product_comparison_collpase" role="img" aria-label="Collapse" title="Collapse"></i><span>Tags</span>
                                        </th>
                                    </tr>
                                    <tr class="collapse show o_ws_tags">
                                        <td><span>Tags</span></td>
                                        <td t-foreach="products" t-as="product">
                                            <div class="d-flex justify-content-center">
                                                <t t-call="website_sale.product_tags">
                                                    <t t-set="all_product_tags" t-value="product.all_product_tag_ids"/>
                                                </t>
                                            </div>

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
                        <span t-out="combination_info['price']"
                              t-options="{'widget': 'monetary', 'display_currency': website.currency_id}"/>
                        <del t-if="combination_info['compare_list_price'] and (combination_info['compare_list_price'] &gt; combination_info['price'])"
                             t-attf-class="text-muted me-1 h6 mb-0 small"
                             style="white-space: nowrap;"
                             t-esc="combination_info['compare_list_price']"
                             t-options="{'widget': 'monetary', 'display_currency': website.currency_id}" />
                        <del t-else=""
                             t-attf-class="text-muted me-1 h6 mb-0 small {{'' if combination_info['has_discounted_price'] else 'd-none'}}"
                             style="white-space: nowrap;"
                             t-esc="combination_info['list_price']"
                             t-options="{'widget': 'monetary', 'display_currency': website.currency_id}"/>
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
        <field name="name">product.attribute.category.list</field>
        <field name="model">product.attribute.category</field>
        <field name="arch" type="xml">
            <list string="Product Attribute Category" editable="bottom">
                <field name="sequence" widget="handle"/>
                <field name="name"/>
                <field name="attribute_ids" widget="many2many_tags" options="{'no_create': True}"/>
            </list>
        </field>
    </record>

    <record id="product_attribute_category_action" model="ir.actions.act_window">
        <field name="name">Attribute Categories</field>
        <field name="res_model">product.attribute.category</field>
        <field name="view_mode">list</field>
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
        <field name="name">product.attribute.list.inherit</field>
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
        <field name="inherit_id" ref="website_sale.product_attribute_view_form"/>
        <field name="priority" eval="8"/>
        <field name="arch" type="xml">
            <group name="ecommerce_main_fields" position="inside">
                <field name="category_id"/>
            </group>
        </field>
    </record>

</odoo>

```

