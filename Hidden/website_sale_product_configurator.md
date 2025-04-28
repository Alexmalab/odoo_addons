# Odoo Module: website_sale_product_configurator

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-

from . import controllers
from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
{
    'name': "Website Sale Product Configurator",
    'summary': """
        Bridge module for website_sale / sale_product_configurator""",
    'description': """
        Bridge module to make the website e-commerce compatible with the product configurator
    """,
    'category': 'Hidden',
    'version': '0.1',
    'depends': ['website_sale', 'sale_product_configurator'],
    'auto_install': True,
    'data': [
        'views/website_sale_product_configurator_templates.xml',
    ],
    'demo': [
        'data/demo.xml',
    ],
    'assets': {
        'web.assets_frontend': [
            ('before', 'website_sale/static/src/js/website_sale.js', 'sale_product_configurator/static/src/js/product_configurator_modal.js'),
            ('before', 'website_sale/static/src/js/website_sale.js', 'website_sale_product_configurator/static/src/js/product_configurator_modal.js'),
            'sale/static/src/scss/product_configurator.scss',
            'website_sale_product_configurator/static/src/scss/website_sale_options.scss',
            'website_sale_product_configurator/static/src/js/website_sale_options.js',
        ],
        'web.assets_tests': [
            'website_sale_product_configurator/static/tests/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import json
from odoo import http
from odoo.http import request

from odoo.addons.sale_product_configurator.controllers.main import ProductConfiguratorController
from odoo.addons.website_sale.controllers import main


class WebsiteSaleProductConfiguratorController(ProductConfiguratorController):
    @http.route(['/sale_product_configurator/show_advanced_configurator_website'], type='json', auth="public", methods=['POST'], website=True)
    def show_advanced_configurator_website(self, product_id, variant_values, **kw):
        """Special route to use website logic in get_combination_info override.
        This route is called in JS by appending _website to the base route.
        """
        kw.pop('pricelist_id')
        product = request.env['product.product'].browse(int(product_id))
        combination = request.env['product.template.attribute.value'].browse(variant_values)
        has_optional_products = product.optional_product_ids.filtered(lambda p: p._is_add_to_cart_possible(combination))

        if not has_optional_products and (product.product_variant_count <= 1 or variant_values):
            # The modal is not shown if there are no optional products and
            # the main product either has no variants or is already configured
            return False
        if variant_values:
            kw["already_configured"] = True
        return self.show_advanced_configurator(product_id, variant_values, request.website.get_current_pricelist(), **kw)

    @http.route(['/sale_product_configurator/optional_product_items_website'], type='json', auth="public", methods=['POST'], website=True)
    def optional_product_items_website(self, product_id, **kw):
        """Special route to use website logic in get_combination_info override.
        This route is called in JS by appending _website to the base route.
        """
        kw.pop('pricelist_id')
        return self.optional_product_items(product_id, request.website.get_current_pricelist(), **kw)


class WebsiteSale(main.WebsiteSale):
    def _prepare_product_values(self, product, category, search, **kwargs):
        values = super(WebsiteSale, self)._prepare_product_values(product, category, search, **kwargs)

        values['optional_product_ids'] = [p.with_context(active_id=p.id) for p in product.optional_product_ids]
        return values

    @http.route(['/shop/cart/update_option'], type='http', auth="public", methods=['POST'], website=True, multilang=False)
    def cart_options_update_json(self, product_and_options, goto_shop=None, lang=None, **kwargs):
        """This route is called when submitting the optional product modal.
            The product without parent is the main product, the other are options.
            Options need to be linked to their parents with a unique ID.
            The main product is the first product in the list and the options
            need to be right after their parent.
            product_and_options {
                'product_id',
                'product_template_id',
                'quantity',
                'parent_unique_id',
                'unique_id',
                'product_custom_attribute_values',
                'no_variant_attribute_values'
            }
        """
        if lang:
            request.website = request.website.with_context(lang=lang)

        order = request.website.sale_get_order(force_create=True)
        if order.state != 'draft':
            request.session['sale_order_id'] = None
            order = request.website.sale_get_order(force_create=True)

        product_and_options = json.loads(product_and_options)
        if product_and_options:
            # The main product is the first, optional products are the rest
            main_product = product_and_options[0]
            value = order._cart_update(
                product_id=main_product['product_id'],
                add_qty=main_product['quantity'],
                product_custom_attribute_values=main_product['product_custom_attribute_values'],
                no_variant_attribute_values=main_product['no_variant_attribute_values'],
            )

            # Link option with its parent.
            option_parent = {main_product['unique_id']: value['line_id']}
            for option in product_and_options[1:]:
                parent_unique_id = option['parent_unique_id']
                option_value = order._cart_update(
                    product_id=option['product_id'],
                    set_qty=option['quantity'],
                    linked_line_id=option_parent[parent_unique_id],
                    product_custom_attribute_values=option['product_custom_attribute_values'],
                    no_variant_attribute_values=option['no_variant_attribute_values'],
                )
                option_parent[option['unique_id']] = option_value['line_id']

        return str(order.cart_quantity)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-

from . import main

```

## File: data\demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <record id="sale_product_configurator.product_product_1_product_template" model="product.template">
            <field name="website_sequence">9980</field>
            <field name="is_published" eval="True"/>
        </record>
    <record id="product.product_product_4_product_template" model="product.template">
        <field name="optional_product_ids" eval="[(6,0,[ref('product.product_product_11_product_template'), ref('website_sale.product_product_1_product_template')])]"/>
    </record>
</odoo>

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class SaleOrder(models.Model):
    _inherit = "sale.order"

    def _cart_find_product_line(self, product_id=None, line_id=None, **kwargs):
        lines = super(SaleOrder, self)._cart_find_product_line(product_id, line_id, **kwargs)
        if line_id:  # in this case we get the exact line we want, so filtering below would be wrong
            return lines

        linked_line_id = kwargs.get('linked_line_id', False)
        optional_product_ids = set(kwargs.get('optional_product_ids', []))

        lines = lines.filtered(lambda line: line.linked_line_id.id == linked_line_id)
        if optional_product_ids:
            # only match the lines with the same chosen optional products on the existing lines
            lines = lines.filtered(lambda line: optional_product_ids == set(line.mapped('option_line_ids.product_id.id')))
        else:
            lines = lines.filtered(lambda line: not line.option_line_ids)

        return lines

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import sale_order

```

## File: static\src\js\product_configurator_modal.js

```javascript
odoo.define('website_sale_product_configurator.OptionalProductsModal', function (require) {
    "use strict";

var OptionalProductsModal = require('sale_product_configurator.OptionalProductsModal');

OptionalProductsModal.include({
    /**
     * If the "isWebsite" param is true, will also disable the following events:
     * - change [data-attribute_exclusions]
     * - click button.js_add_cart_json
     *
     * This has to be done because those events are already registered at the "website_sale"
     * component level.
     * This modal is part of the form that has these events registered and we
     * want to avoid duplicates.
     *
     * @override
     * @param {$.Element} parent The parent container
     * @param {Object} params
     * @param {boolean} params.isWebsite If we're on a web shop page, we need some
     *   custom behavior
     */
    init: function (parent, params) {
        this._super.apply(this, arguments);
        this.isWebsite = params.isWebsite;

        this.dialogClass = 'oe_advanced_configurator_modal' + (params.isWebsite ? ' oe_website_sale' : '');
    },
    /**
     * @override
     * @private
     */
    _triggerPriceUpdateOnChangeQuantity: function () {
        return !this.isWebsite;
    }
});

});

```

## File: static\src\js\website_sale_options.js

```javascript
odoo.define('website_sale_options.website_sale', function (require) {
'use strict';

var ajax = require('web.ajax');
var core = require('web.core');
var publicWidget = require('web.public.widget');
var OptionalProductsModal = require('sale_product_configurator.OptionalProductsModal');
require('website_sale.website_sale');

var _t = core._t;

publicWidget.registry.WebsiteSale.include({

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    _onProductReady: function () {
        if (this.isBuyNow) {
            return this._submitForm();
        }
        this.optionalProductsModal = new OptionalProductsModal(this.$form, {
            rootProduct: this.rootProduct,
            isWebsite: true,
            okButtonText: _t('Proceed to Checkout'),
            cancelButtonText: _t('Continue Shopping'),
            title: _t('Add to cart'),
            context: this._getContext(),
        }).open();

        this.optionalProductsModal.on('options_empty', null, this._submitForm.bind(this));
        this.optionalProductsModal.on('update_quantity', null, this._onOptionsUpdateQuantity.bind(this));
        this.optionalProductsModal.on('confirm', null, this._onModalSubmit.bind(this, true));
        this.optionalProductsModal.on('back', null, this._onModalSubmit.bind(this, false));

        return this.optionalProductsModal.opened();
    },
    /**
     * Overridden to resolve _opened promise on modal
     * when stayOnPageOption is activated.
     *
     * @override
     */
    _submitForm() {
        const ret = Promise.resolve(this._super(...arguments));
        if (this.optionalProductsModal && this.stayOnPageOption) {
            ret.then(()=>{
                this.optionalProductsModal._openedResolver()
            });
        }
        return ret;
    },
    /**
     * Update web shop base form quantity
     * when quantity is updated in the optional products window
     *
     * @private
     * @param {integer} quantity
     */
    _onOptionsUpdateQuantity: function (quantity) {
        var $qtyInput = this.$form
            .find('.js_main_product input[name="add_qty"]')
            .first();

        if ($qtyInput.length) {
            $qtyInput.val(quantity).trigger('change');
        } else {
            // This handles the case when the "Select Quantity" customize show
            // is disabled, and therefore the above selector does not find an
            // element.
            // To avoid duplicating all RPC, only trigger the variant change if
            // it is not already done from the above trigger.
            this.optionalProductsModal.triggerVariantChange(this.optionalProductsModal.$el);
        }
    },

    /**
     * Submits the form with additional parameters
     * - lang
     * - product_custom_attribute_values: The products custom variant values
     *
     * @private
     * @param {Boolean} goToShop Triggers a page refresh to the url "shop/cart"
     */
    _onModalSubmit: function (goToShop) {
        const $product = $('#product_detail');
        let currency;
        if ($product.length) {
            currency = $product.data('product-tracking-info')['currency'];
        } else {
            // Add to cart from /shop page
            currency = this.$('[itemprop="priceCurrency"]').first().text();
        }
        const productsTrackingInfo = [];
        this.$('.js_product.in_cart').each((i, el) => {
            productsTrackingInfo.push({
                'item_id': parseInt(el.getElementsByClassName('product_id')[0].value),
                'item_name': el.getElementsByClassName('product_display_name')[0].textContent,
                'quantity': parseFloat(el.getElementsByClassName('js_quantity')[0].value),
                'currency': currency,
                'price': parseFloat(el.getElementsByClassName('js_raw_price')[0].textContent),
            });
        });
        if (productsTrackingInfo.length) {
            this.$el.trigger('add_to_cart_event', productsTrackingInfo);
        }

        this.optionalProductsModal.getAndCreateSelectedProducts()
            .then((products) => {
                const productAndOptions = JSON.stringify(products);
                ajax.post('/shop/cart/update_option', {product_and_options: productAndOptions})
                    .then(function (quantity) {
                        if (goToShop) {
                            window.location.pathname = "/shop/cart";
                        }
                        const $quantity = $(".my_cart_quantity");
                        $quantity.parent().parent().removeClass('d-none');
                        $quantity.text(quantity).hide().fadeIn(600);
                    }).then(()=>{
                        this._getCombinationInfo($.Event('click', {target: $("#add_to_cart")}));
                    });
            });
    },
});

return publicWidget.registry.WebsiteSaleOptions;

});

```

## File: views\website_sale_product_configurator_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="configure_optional_products_website" inherit_id="sale_product_configurator.configure_optional_products">
        <xpath expr="//th[hasclass('td-qty')]/span[hasclass('label')]" position="attributes">
            <attribute name='t-if'>not request.is_frontend or (request.is_frontend and is_view_active('website_sale.product_quantity'))</attribute>
        </xpath>
    </template>
    <template id="product_quantity_config_website" inherit_id="sale_product_configurator.product_quantity_config" priority="18">
        <xpath expr="//div[hasclass('css_quantity')]" position="attributes">
            <attribute name='t-if'>not request.is_frontend or (request.is_frontend and is_view_active('website_sale.product_quantity'))</attribute>
        </xpath>
        <xpath expr="//div[hasclass('css_quantity')]" position="after">
            <input t-else="" type="hidden" class="d-none js_quantity form-control quantity" name="add_qty" t-att-value="add_qty or 1"/>
        </xpath>
    </template>
</odoo>

```

