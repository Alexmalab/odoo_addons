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
    'summary': "Bridge module for website_sale / sale_product_configurator",
    'description': """
Bridge module to make the website e-commerce compatible with the product configurator
    """,
    'category': 'Hidden',
    'depends': ['website_sale', 'sale_product_configurator'],
    'auto_install': True,
    'data': [
        'views/templates.xml',
    ],
    'demo': [
        'data/demo.xml',
    ],
    'assets': {
        'web.assets_frontend': [
            ('before', 'website_sale/static/src/js/website_sale.js', 'website_sale_product_configurator/static/src/js/sale_product_configurator_modal.js'),
            'website_sale/static/src/scss/product_configurator.scss',
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
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.http import Controller, request, route


class WebsiteSaleProductConfiguratorController(Controller):

    @route(
        '/sale_product_configurator/show_advanced_configurator',
        type='json', auth='public', methods=['POST'], website=True,
    )
    def show_advanced_configurator(
        self, product_id, variant_values, add_qty=1, force_dialog=False, **kw,
    ):
        product = request.env['product.product'].browse(int(product_id))
        product_template = product.product_tmpl_id
        combination = request.env['product.template.attribute.value'].browse(variant_values)
        has_optional_products = product.optional_product_ids.filtered(
            lambda p: p._is_add_to_cart_possible(combination)
                      and (not request.website.prevent_zero_price_sale or p._get_contextual_price())
        )

        already_configured = bool(combination)
        if not force_dialog and not has_optional_products and (
            product.product_variant_count <= 1 or already_configured
        ):
            # The modal is not shown if there are no optional products and
            # the main product either has no variants or is already configured
            return False

        add_qty = float(add_qty)
        combination_info = product_template._get_combination_info(
            combination=combination,
            product_id=product.id,
            add_qty=add_qty,
        )

        return request.env['ir.ui.view']._render_template(
            'website_sale_product_configurator.optional_products_modal',
            {
                'product': product,
                'product_template': product_template,
                'combination': combination,
                'combination_info': combination_info,
                'add_qty': add_qty,
                'parent_name': product.name,
                'variant_values': variant_values,
                'already_configured': already_configured,
                'mode': kw.get('mode', 'add'),
                'product_custom_attribute_values': kw.get('product_custom_attribute_values', None),
                'no_attribute': kw.get('no_attribute', False),
                'custom_attribute': kw.get('custom_attribute', False),
            }
        )

    @route(
        '/sale_product_configurator/optional_product_items',
        type='json', auth='public', methods=['POST'], website=True,
    )
    def optional_product_items(self, product_id, add_qty=1, **kw):
        product = request.env['product.product'].browse(int(product_id))

        exclude_product_tmpl_ids = kw.get('exclude_product_tmpl_ids')
        if exclude_product_tmpl_ids:
            # Temporarily exclude products from being in `optional_product_ids`
            # to avoid issues with mutually recursive/cyclic optional products
            optional_products = product.optional_product_ids
            exclude_products = request.env['product.template'].browse(exclude_product_tmpl_ids)
            request.env.cache.update(
                product,
                product._fields['optional_product_ids'],
                [(optional_products - exclude_products).ids],
            )
        res = request.env['ir.ui.view']._render_template(
            'website_sale_product_configurator.optional_product_items',
            {
                'product': product,
                'parent_name': product.name,
                'parent_combination': product.product_template_attribute_value_ids,
                'add_qty': float(add_qty) or 1.0,
            }
        )
        if exclude_product_tmpl_ids:
            # Re-add the excluded products after rendering the configurator template
            request.env.cache.update(
                product,
                product._fields['optional_product_ids'],
                [optional_products.ids],
            )
        return res

```

## File: controllers\website_sale.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json

from odoo.http import request, route

from odoo.addons.website_sale.controllers import main


class WebsiteSale(main.WebsiteSale):

    def _prepare_product_values(self, product, category, search, **kwargs):
        values = super()._prepare_product_values(product, category, search, **kwargs)

        values['optional_product_ids'] = [p.with_context(active_id=p.id) for p in product.optional_product_ids]
        return values

    @route(
        '/shop/cart/update_option',
        type='json',
        auth='public',
        methods=['POST'],
        website=True,
        multilang=False,
    )
    def cart_options_update_json(self, product_and_options, lang=None, **kwargs):
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
            values = order._cart_update(
                product_id=main_product['product_id'],
                add_qty=main_product['quantity'],
                product_custom_attribute_values=main_product['product_custom_attribute_values'],
                no_variant_attribute_values=main_product['no_variant_attribute_values'],
                **kwargs
            )

            line_ids = [values['line_id']]

            if values['line_id']:
                # Link option with its parent iff line has been created.
                option_parent = {main_product['unique_id']: values['line_id']}
                for option in product_and_options[1:]:
                    parent_unique_id = option['parent_unique_id']
                    option_values = order._cart_update(
                        product_id=option['product_id'],
                        set_qty=option['quantity'],
                        linked_line_id=option_parent[parent_unique_id],
                        product_custom_attribute_values=option['product_custom_attribute_values'],
                        no_variant_attribute_values=option['no_variant_attribute_values'],
                        **kwargs
                    )
                    option_parent[option['unique_id']] = option_values['line_id']
                    line_ids.append(option_values['line_id'])

            values['notification_info'] = self._get_cart_notification_information(order, line_ids)

        values['cart_quantity'] = order.cart_quantity
        request.session['website_sale_cart_quantity'] = order.cart_quantity

        return values

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-

from . import main
from . import website_sale

```

## File: data\demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <record id="sale_product_configurator.product_product_1_product_template" model="product.template">
            <field name="website_sequence">9985</field>
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

from odoo import models


class SaleOrder(models.Model):
    _inherit = "sale.order"

    def _cart_find_product_line(
        self, product_id=None, line_id=None,
        linked_line_id=False, optional_product_ids=None, **kwargs
    ):
        lines = super()._cart_find_product_line(product_id, line_id, **kwargs)
        if line_id:  # in this case we get the exact line we want, so filtering below would be wrong
            return lines

        lines = lines.filtered(lambda line: line.linked_line_id.id == linked_line_id)
        if optional_product_ids:
            # only match the lines with the same chosen optional products on the existing lines
            lines = lines.filtered(lambda line: optional_product_ids == set(line.option_line_ids.product_id.id))
        else:
            lines = lines.filtered(lambda line: not line.option_line_ids)

        return lines

```

## File: models\website.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class Website(models.Model):
    _inherit = 'website'

    add_to_cart_action = fields.Selection(
        selection_add=[('force_dialog', "Let the user decide (dialog)")],
        ondelete={'force_dialog': 'set default'})

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import sale_order
from . import website

```

## File: static\src\js\sale_product_configurator_modal.js

```javascript
/** @odoo-module */

import Dialog from '@web/legacy/js/core/dialog';
import VariantMixin from '@website_sale/js/sale_variant_mixin';
import { uniqueId } from '@web/core/utils/functions';
import { jsonrpc } from '@web/core/network/rpc_service';

export const OptionalProductsModal = Dialog.extend(VariantMixin, {
    events:  Object.assign({}, Dialog.prototype.events, VariantMixin.events, {
        'click a.js_add, a.js_remove': '_onAddOrRemoveOption',
        'click button.js_add_cart_json': 'onClickAddCartJSON',
        'change .in_cart input.js_quantity': '_onChangeQuantity',
        'change .js_raw_price': '_computePriceTotal'
    }),
    /**
     * Initializes the optional products modal
     *
     * @override
     * @param {$.Element} parent The parent container
     * @param {Object} params
     * @param {integer} params.pricelistId
     * @param {boolean} params.isWebsite If we're on a web shop page, we need some
     *   custom behavior
     * @param {string} params.okButtonText The text to apply on the "ok" button, typically
     *   "Add" for the sale order and "Proceed to checkout" on the web shop
     * @param {string} params.cancelButtonText same as "params.okButtonText" but
     *   for the cancel button
     * @param {integer} params.previousModalHeight used to configure a min height on the modal-content.
     *   This parameter is provided by the product configurator to "cover" its modal by making
     *   this one big enough. This way the user can't see multiple buttons (which can be confusing).
     * @param {Object} params.rootProduct The root product of the optional products window
     * @param {integer} params.rootProduct.product_id
     * @param {integer} params.rootProduct.quantity
     * @param {Array} params.rootProduct.variant_values
     * @param {Array} params.rootProduct.product_custom_attribute_values
     * @param {Array} params.rootProduct.no_variant_attribute_values
     */
    init: function (parent, params) {
        var self = this;

        var options = Object.assign({
            size: 'large',
            buttons: [{
                text: params.okButtonText,
                click: this._onConfirmButtonClick,
                // the o_sale_product_configurator_edit class is used for tours.
                classes: 'btn-primary o_sale_product_configurator_edit'
            }, {
                text: params.cancelButtonText,
                click: this._onCancelButtonClick
            }],
            technical: !params.isWebsite,
        }, params || {});

        this._super(parent, options);

        this.isWebsite = params.isWebsite;
        this.forceDialog = params.forceDialog;

        this.dialogClass = 'oe_advanced_configurator_modal' + (params.isWebsite ? ' oe_website_sale' : '');
        this.context = params.context;
        this.rootProduct = params.rootProduct;
        this.container = parent;
        this.pricelistId = params.pricelistId;
        this.previousModalHeight = params.previousModalHeight;
        this.mode = params.mode;
        this.dialogClass = 'oe_advanced_configurator_modal';
        this._productImageField = 'image_128';

        this._opened.then(function () {
            if (self.previousModalHeight) {
                self.$el.closest('.modal-content').css('min-height', self.previousModalHeight + 'px');
            }
        });

        this.rpc = this.bindService("rpc");
    },
     /**
     * @override
     */
    willStart: function () {
        var self = this;

        var getModalContent = jsonrpc("/sale_product_configurator/show_advanced_configurator", {
            mode: self.mode,
            product_id: self.rootProduct.product_id,
            variant_values: self.rootProduct.variant_values,
            product_custom_attribute_values: self.rootProduct.product_custom_attribute_values,
            pricelist_id: self.pricelistId || false,
            add_qty: self.rootProduct.quantity,
            force_dialog: self.forceDialog,
            no_attribute: self.rootProduct.no_variant_attribute_values,
            custom_attribute: self.rootProduct.product_custom_attribute_values,
            context: Object.assign({'quantity': self.rootProduct.quantity}, this.context),
        })
        .then(function (modalContent) {
            if (modalContent) {
                var $modalContent = $(modalContent);
                $modalContent = self._postProcessContent($modalContent);
                self.$content = $modalContent;
            } else {
                self.trigger('options_empty');
                self.preventOpening = true;
            }
        });

        var parentInit = self._super.apply(self, arguments);
        return Promise.all([getModalContent, parentInit]);
    },

    /**
     * This is overridden to append the modal to the provided container (see init("parent")).
     * We need this to have the modal contained in the web shop product form.
     * The additional products data will then be contained in the form and sent on submit.
     *
     * @override
     */
    open: function (options) {
        $('.tooltip').remove(); // remove open tooltip if any to prevent them staying when modal is opened

        var self = this;
        this.appendTo($('<div/>')).then(function () {
            if (!self.preventOpening) {
                self.$modal.find(".modal-body").replaceWith(self.$el);
                self.$modal.attr('open', true);
                self.$modal.appendTo(self.container);
                const modal = new Modal(self.$modal[0], {
                    focus: true,
                });
                modal.show();
                self._openedResolver();
            }
        });
        if (options && options.shouldFocusButtons) {
            self._onFocusControlButton();
        }

        return self;
    },
    /**
     * Will update quantity input to synchronize with previous window
     *
     * @override
     */
    start: function () {
        var def = this._super.apply(this, arguments);
        var self = this;

        this.$el.find('input[name="add_qty"]').val(this.rootProduct.quantity);

        // set a unique id to each row for options hierarchy
        var $products = this.$el.find('tr.js_product').toArray();
        $products.forEach((el) => {
            var $el = $(el);
            var uniqueId = self._getUniqueId(el);

            var productId = parseInt($el.find('input.product_id').val(), 10);
            if (productId === self.rootProduct.product_id) {
                self.rootProduct.unique_id = uniqueId;
            } else {
                el.dataset.parentUniqueId = self.rootProduct.unique_id;
            }
        });

        return def.then(function () {
            // This has to be triggered to compute the "out of stock" feature
            self._opened.then(function () {
                self.triggerVariantChange(self.$el);
            });
        });
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * Returns the list of selected products.
     * The root product is added on top of the list.
     *
     * @returns {Array} products
     *   {integer} product_id
     *   {integer} quantity
     *   {Array} product_custom_variant_values
     *   {Array} no_variant_attribute_values
     * @public
     */
    getAndCreateSelectedProducts: async function () {
        var self = this;
        const products = [];
        let productCustomVariantValues;
        let noVariantAttributeValues;
        for (const product of self.$modal.find('.js_product.in_cart')) {
            var $item = $(product);
            var quantity = parseFloat($item.find('input[name="add_qty"]').val().replace(',', '.') || 1);
            var parentUniqueId = product.dataset.parentUniqueId;
            var uniqueId = product.dataset.uniqueId;
            productCustomVariantValues = $item.find('.custom-attribute-info').data("attribute-value") || self.getCustomVariantValues($item);
            noVariantAttributeValues = $item.find('.no-attribute-info').data("attribute-value") || self.getNoVariantAttributeValues($item);

            const productID = await self.selectOrCreateProduct(
                $item,
                parseInt($item.find('input.product_id').val(), 10),
                parseInt($item.find('input.product_template_id').val(), 10),
                true
            );
            products.push({
                'product_id': productID,
                'product_template_id': parseInt($item.find('input.product_template_id').val(), 10),
                'quantity': quantity,
                'parent_unique_id': parentUniqueId,
                'unique_id': uniqueId,
                'product_custom_attribute_values': productCustomVariantValues,
                'no_variant_attribute_values': noVariantAttributeValues
            });
        }
        return products;
    },

    // ------------------------------------------
    // Private
    // ------------------------------------------

    /**
     * Adds the product image and updates the product description
     * based on attribute values that are either "no variant" or "custom".
     *
     * @private
     */
    _postProcessContent: function ($modalContent) {
        var productId = this.rootProduct.product_id;
        $modalContent
            .find('img:first')
            .attr("src", "/web/image/product.product/" + productId + "/image_128");

        if (this.rootProduct &&
                (this.rootProduct.product_custom_attribute_values ||
                 this.rootProduct.no_variant_attribute_values)) {
            var $productDescription = $modalContent
                .find('.main_product')
                .find('td.td-product_name div.text-muted.small > div:first');
            var $updatedDescription = $('<div/>');
            $updatedDescription.append($('<p>', {
                text: $productDescription.text()
            }));
            $.each(this.rootProduct.product_custom_attribute_values, function () {
                if (this.custom_value) {
                    const $customInput = $modalContent
                        .find(".main_product [data-is_custom='True']")
                        .closest(`[data-value_id='${this.custom_product_template_attribute_value_id.res_id}']`);
                    $customInput.attr('previous_custom_value', this.custom_value);
                    VariantMixin.handleCustomValues($customInput);
                }
            });

            $.each(this.rootProduct.no_variant_attribute_values, function () {
                if (this.is_custom !== 'True') {
                    var $currentDescription = $updatedDescription.find(`div[name=ptal-${this.id}]`);
                    if ($currentDescription?.length > 0) { // one row per multicheckbox
                        $currentDescription.text($currentDescription.text() + ', ' + this.attribute_value_name);
                    } else {
                        $updatedDescription.append($('<div>', {
                            text: this.attribute_name + ': ' + this.attribute_value_name,
                            name: `ptal-${this.id}`,
                        }));
                    }
                }
            });

            $productDescription.replaceWith($updatedDescription);
        }

        return $modalContent;
    },

    /**
     * @private
     */
    _onConfirmButtonClick: function () {
        this.trigger('confirm');
        this.close();
    },

    /**
     * @private
     */
    _onCancelButtonClick: function () {
        this.trigger('back');
        this.close();
    },

    /**
     * Will add/remove the option, that includes:
     * - Moving it to the correct DOM section
     *   and possibly under its parent product
     * - Hiding attribute values selection and showing the quantity
     * - Creating the product if it's in "dynamic" mode (see product_attribute.create_variant)
     * - Updating the description based on custom/no_create attribute values
     * - Removing optional products if parent product is removed
     * - Computing the total price
     *
     * @private
     * @param {MouseEvent} ev
     */
    _onAddOrRemoveOption: function (ev) {
        ev.preventDefault();
        var self = this;
        var $target = $(ev.currentTarget);
        var $modal = $target.parents('.oe_advanced_configurator_modal');
        var $parent = $target.parents('.js_product:first');
        $parent.find("a.js_add, span.js_remove").toggleClass('d-none');
        $parent.find(".js_remove");

        var productTemplateId = $parent.find(".product_template_id").val();
        if ($target.hasClass('js_add')) {
            self._onAddOption($modal, $parent, productTemplateId);
        } else {
            self._onRemoveOption($modal, $parent);
        }

        self._computePriceTotal();
    },

    /**
     * @private
     * @see _onAddOrRemoveOption
     * @param {$.Element} $modal
     * @param {$.Element} $parent
     * @param {integer} productTemplateId
     */
    _onAddOption: function ($modal, $parent, productTemplateId) {
        var self = this;
        var $selectOptionsText = $modal.find('.o_select_options');

        var parentUniqueId = $parent[0].dataset.parentUniqueId;
        var $optionParent = $modal.find('tr.js_product[data-unique-id="' + parentUniqueId + '"]');

        // remove attribute values selection and update + show quantity input
        $parent.find('.td-product_name').removeAttr("colspan");
        $parent.find('.td-qty').removeClass('d-none');

        var productCustomVariantValues = self.getCustomVariantValues($parent);
        var noVariantAttributeValues = self.getNoVariantAttributeValues($parent);
        if (productCustomVariantValues || noVariantAttributeValues) {
            var $productDescription = $parent
                .find('td.td-product_name div.float-start');

            var $customAttributeValuesDescription = $('<div>', {
                class: 'custom_attribute_values_description text-muted small'
            });
            if (productCustomVariantValues.length !== 0 || noVariantAttributeValues.length !== 0) {
                $customAttributeValuesDescription.append($('<br/>'));
            }

            $.each(productCustomVariantValues, function (){
                $customAttributeValuesDescription.append($('<div>', {
                    text: this.attribute_value_name + ': ' + this.custom_value
                }));
            });

            $.each(noVariantAttributeValues, function (){
                if (this.is_custom !== 'True'){
                    var $currentDescription = $customAttributeValuesDescription.find(`div[name=ptal-${this.id}]`);
                    if ($currentDescription?.length > 0) { // one row per multicheckbox
                        $currentDescription.text($currentDescription.text() + ', ' + this.attribute_value_name);
                    } else {
                        $customAttributeValuesDescription.append($('<div>', {
                            text: this.attribute_name + ': ' + this.attribute_value_name,
                            name: `ptal-${this.id}`,
                        }));
                    }
                }
            });

            $productDescription.append($customAttributeValuesDescription);
        }

        // place it after its parent and its parent options
        var $tmpOptionParent = $optionParent;
        while ($tmpOptionParent.length) {
            $optionParent = $tmpOptionParent;
            $tmpOptionParent = $modal.find('tr.js_product.in_cart[data-parent-unique-id="' + $optionParent[0].dataset.uniqueId + '"]').last();
        }
        $optionParent.after($parent);
        $parent.addClass('in_cart');

        this.selectOrCreateProduct(
            $parent,
            $parent.find('.product_id').val(),
            productTemplateId,
            true
        ).then(function (productId) {
            $parent.find('.product_id').val(productId);

            // Get currently displayed items to exclude them from being added again as options
            const product_tmpl_ids = new Array(...$modal.find('input.product_template_id')).map(
                (el) => parseInt(el.value)
            );
            jsonrpc("/sale_product_configurator/optional_product_items", {
                'product_id': productId,
                'pricelist_id': self.pricelistId || false,
                'exclude_product_tmpl_ids': product_tmpl_ids,
            }).then(function (addedItem) {
                var $addedItem = $(addedItem);
                $modal.find('tr:last').after($addedItem);

                self.$el.find('input[name="add_qty"]').trigger('change');
                self.triggerVariantChange($addedItem);

                // add a unique id to the new products
                var parentUniqueId = $parent[0].dataset.uniqueId;
                var parentQty = $parent.find('input[name="add_qty"]').val();
                $addedItem.filter('.js_product').each(function () {
                    var $el = $(this);
                    var uniqueId = self._getUniqueId(this);
                    this.dataset.uniqueId = uniqueId;
                    this.dataset.parentUniqueId = parentUniqueId;
                    $el.find('input[name="add_qty"]').val(parentQty);
                });

                if ($selectOptionsText.nextAll('.js_product').length === 0) {
                    // no more optional products to select -> hide the header
                    $selectOptionsText.hide();
                }
            });
        });
    },

    /**
     * @private
     * @see _onAddOrRemoveOption
     * @param {$.Element} $modal
     * @param {$.Element} $parent
     */
    _onRemoveOption: function ($modal, $parent) {
        // restore attribute values selection
        var uniqueId = $parent[0].dataset.parentUniqueId;
        var qty = $modal.find('tr.js_product.in_cart[data-unique-id="' + uniqueId + '"]').find('input[name="add_qty"]').val();
        $parent.removeClass('in_cart');
        $parent.find('.td-product_name').attr("colspan", 2);
        $parent.find('.td-qty').addClass('d-none');
        $parent.find('input[name="add_qty"]').val(qty);
        $parent.find('.custom_attribute_values_description').remove();

        $modal.find('.o_select_options').show();

        var productUniqueId = $parent[0].dataset.uniqueId;
        this._removeOptionOption($modal, productUniqueId);

        $modal.find('tr:last').after($parent);
    },

    /**
     * If the removed product had optional products, remove them as well
     *
     * @private
     * @param {$.Element} $modal
     * @param {integer} optionUniqueId The removed optional product id
     */
    _removeOptionOption: function ($modal, optionUniqueId) {
        var self = this;
        $modal.find('tr.js_product[data-parent-unique-id="' + optionUniqueId + '"]').each(function () {
            var uniqueId = this.dataset.uniqueId;
            $(this).remove();
            self._removeOptionOption($modal, uniqueId);
        });
    },
    /**
     * @override
     */
    _onChangeCombination: function (ev, $parent, combination) {
        $parent
            .find('.td-product_name .product-name')
            .first()
            .text(combination.display_name);

        VariantMixin._onChangeCombination.apply(this, arguments);

        this._computePriceTotal();
    },
    /**
     * Update price total when the quantity of a product is changed
     *
     * @private
     * @param {MouseEvent} ev
     */
    _onChangeQuantity: function (ev) {
        var $product = $(ev.target.closest('tr.js_product'));
        var qty = parseFloat($(ev.currentTarget).val());

        var uniqueId = $product[0].dataset.uniqueId;
        this.$el.find('tr.js_product:not(.in_cart)[data-parent-unique-id="' + uniqueId + '"] input[name="add_qty"]').each(function () {
            $(this).val(qty);
        });

        if (this._triggerPriceUpdateOnChangeQuantity()) {
            this.onChangeAddQuantity(ev);
        }
        if ($product.hasClass('main_product')) {
            this.rootProduct.quantity = qty;
        }
        this.trigger('update_quantity', this.rootProduct.quantity);
        this._computePriceTotal();
    },

    /**
     * When a product is added or when the quantity is changed,
     * we need to refresh the total price row
     */
    _computePriceTotal: function () {
        if (this.$modal.find('.js_price_total').length) {
            var price = 0;
            this.$modal.find('.js_product.in_cart').each(function () {
                var quantity = parseFloat($(this).find('input[name="add_qty"]').first().val().replace(',', '.') || 1);
                price += parseFloat($(this).find('.js_raw_price').html()) * quantity;
            });

            this.$modal.find('.js_price_total .oe_currency_value').text(
                this._priceToStr(parseFloat(price))
            );
        }
    },

    /**
     * Extension point for website_sale
     *
     * @private
     */
    _triggerPriceUpdateOnChangeQuantity: function () {
        return !this.isWebsite;
    },
    /**
     * Returns a unique id for `$el`.
     *
     * @private
     * @param {Element} el
     * @returns {integer}
     */
    _getUniqueId: function (el) {
        if (!el.dataset.uniqueId) {
            el.dataset.uniqueId = parseInt(uniqueId(), 10);
        }
        return el.dataset.uniqueId;
    },
});

```

## File: static\src\js\website_sale_options.js

```javascript
/** @odoo-module **/

import publicWidget from "@web/legacy/js/public/public_widget";
import wSaleUtils from "@website_sale/js/website_sale_utils";
import { OptionalProductsModal } from "@website_sale_product_configurator/js/sale_product_configurator_modal";
import "@website_sale/js/website_sale";
import { _t } from "@web/core/l10n/translation";

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
            forceDialog: this.forceDialog,
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
        const mainProduct = this.$('.js_product.in_cart.main_product').children('.product_id');
        const productTrackingInfo = mainProduct.data('product-tracking-info');
        if (productTrackingInfo) {
            const currency = productTrackingInfo['currency'];
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
        }

        const callService = this.call.bind(this)
        this.optionalProductsModal.getAndCreateSelectedProducts()
            .then((products) => {
                const productAndOptions = JSON.stringify(products);
                this.rpc('/shop/cart/update_option', {
                    product_and_options: productAndOptions,
                    ...this._getOptionalCombinationInfoParam(),
                }).then(function (values) {
                    if (goToShop) {
                        window.location.pathname = "/shop/cart";
                    } else {
                        wSaleUtils.updateCartNavBar(values);
                        wSaleUtils.showCartNotification(callService, values.notification_info);
                    }
                }).then(() => {
                    this._getCombinationInfo($.Event('click', {target: $("#add_to_cart")}));
                });
            });
    },
});

export default publicWidget.registry.WebsiteSaleOptions;

```

## File: views\templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="optional_products_modal" name="Optional Products">
        <main class="modal-body">
            <t t-call="website_sale_product_configurator.configure_optional_products" />
        </main>
    </template>

    <template id="product_quantity_config">
        <div t-if="is_view_active('website_sale.product_quantity')"
            class="css_quantity input-group">
            <button t-attf-href="#" class="btn btn-primary float_left js_add_cart_json d-none d-md-inline-block" aria-label="Remove one" title="Remove one">
                <i class="fa fa-minus"></i>
            </button>
            <input type="text"
                class="js_quantity form-control quantity text-center"
                style="max-width: 4rem"
                data-min="1"
                name="add_qty"
                t-att-value="add_qty or 1"/>
            <button t-attf-href="#" class="btn btn-primary float_left js_add_cart_json d-none d-md-inline-block" aria-label="Add one" title="Add one">
                <i class="fa fa-plus"></i>
            </button>
        </div>
        <input t-else="" type="hidden" class="d-none js_quantity form-control quantity" name="add_qty" t-att-value="add_qty or 1"/>
    </template>

    <!-- modal: full table, currenclty selected products at top -->
    <template id="configure_optional_products">
        <table class="table table-striped table-sm">
        <thead>
            <tr>
                <th class="td-img">
                    <span class="label">Product</span>
                </th>
                <th>
                    <span class="label"></span>
                </th>
                <th class="text-center td-qty">
                    <span t-if="is_view_active('website_sale.product_quantity')"
                          class="label">
                        Quantity
                    </span>
                </th>
                <th class="text-center td-price">
                    <span class="label">Price</span>
                </th>
            </tr>
        </thead>
        <tbody>
            <tr class="js_product in_cart main_product">
                <input type="hidden" class="product_template_id" t-att-value="product_template.id"/>
                <input type="hidden"
                       class="product_id"
                       t-att-value="product.id"
                       t-att-data-product-tracking-info="'product_tracking_info' in combination_info and json.dumps(combination_info['product_tracking_info'])"/>
                <td class="td-img">
                    <img class="product_detail_img" t-if="product" t-att-src="'/web/image/product.product/%s/image_128' % product_id" alt="Product Image"/>
                    <img class="product_detail_img" t-else="" t-att-src="'/web/image/product.template/%s/image_128' % product_template_id" alt="Product Image"/>
                </td>
                <td class="td-product_name">
                    <strong class="product-name product_display_name" t-out="combination_info['display_name']"/>
                    <div class="text-muted small">
                        <div t-field="product.description_sale"/>
                        <div class="js_attributes"/>
                        <div t-if="product_custom_attribute_values">
                            <t t-foreach="product_custom_attribute_values" t-as="custom_value">
                                <span t-esc="custom_value.get('attribute_value_name', None)"/>: <span t-esc="custom_value['custom_value']"/>
                                <input type="hidden" class="variant_custom_value"
                                       t-att-data-custom_product_template_attribute_value_id="custom_value['custom_product_template_attribute_value_id']"
                                       t-att-data-attribute_value_name="custom_value.get('attribute_value_name', None)"
                                       t-att-value="custom_value['custom_value']"/>
                            </t>
                        </div>
                    </div>
                    <div>
                        <t t-if="product and not combination">
                            <t t-set="combination" t-value="product_template._get_first_possible_combination()"/>
                        </t>
                        <t t-if="combination and not already_configured" t-call="website_sale.variants">
                            <t t-set="ul_class" t-valuef="flex-column" />
                            <t t-set="product" t-value="product_template"/>
                        </t>
                        <t t-else="">
                            <ul class="d-none js_add_cart_variants mb-0" t-att-data-attribute_exclusions="{'exclusions: []'}"/>
                            <div class="d-none oe_unchanged_value_ids" t-att-data-unchanged_value_ids="variant_values" ></div>
                            <!-- Keep the information to use it later (when leaving the modal window) -->
                            <div class="d-none no-attribute-info" t-att-data-attribute-value="json.dumps(no_attribute)"></div>
                            <div class="d-none custom-attribute-info" t-att-data-attribute-value="json.dumps(custom_attribute)"></div>
                        </t>
                    </div>
                </td>
                <td class="text-center td-qty">
                    <t t-call="website_sale_product_configurator.product_quantity_config"/>
                </td>
                <td class="text-center td-price" name="price">
                    <div
                        t-if="not combination_info.get('compare_list_price')"
                        t-attf-class="text-danger oe_default_price oe_striked_price
                            {{'' if combination_info['has_discounted_price'] else 'd-none'}}"
                        t-out="combination_info['list_price']"
                        t-options="{'widget': 'monetary', 'display_currency': website.currency_id}"/>
                    <div
                        t-if="combination_info.get('compare_list_price')
                            and combination_info['compare_list_price']
                            &gt; combination_info['price']"
                        class="text-danger oe_striked_price"
                        t-out="combination_info['compare_list_price']"
                        groups="website_sale.group_product_price_comparison"
                        t-options='{
                            "widget": "monetary",
                            "display_currency": (pricelist or product).currency_id,
                        }'
                    />
                    <span class="oe_price product_id" style="white-space: nowrap;"
                        t-att-data-product-id="product.id"
                        t-out="combination_info['price']"
                        t-options="{'widget': 'monetary', 'display_currency': website.currency_id}"/>
                    <span class="js_raw_price d-none" t-out="combination_info['price']"/>
                    <p class="css_not_available_msg alert alert-warning">Option not available</p>
                </td>
            </tr>
            <tr class="o_total_row">
                <td colspan="4" class="text-end">
                    <strong>Total:</strong>
                    <span class="js_price_total fw-bold" style="white-space: nowrap;"
                        t-att-data-product-id="product.id"
                        t-out="combination_info['price'] * (add_qty or 1)"
                        t-options="{'widget': 'monetary', 'display_currency': website.currency_id}"/>
                </td>
            </tr>
            <t t-if="product.optional_product_ids and mode != 'edit'">
                <tr class="o_select_options"><td colspan="4"><h4>Available Options:</h4></td></tr>
                <t t-call="website_sale_product_configurator.optional_product_items">
                    <t t-set="parent_combination" t-value="combination"/>
                </t>
            </t>
        </tbody>
        </table>
    </template>

    <!-- modal: optional products -->
    <template id="optional_product_items">
        <t t-foreach="product.optional_product_ids" t-as="product">
            <t t-if="product._is_add_to_cart_possible(parent_combination)">

                <t t-set="combination" t-value="product._get_first_possible_combination(parent_combination)"/>
                <t t-set="combination_info" t-value="product._get_combination_info(combination, add_qty=add_qty)"/>
                <t t-set="product_variant" t-value="product.env['product.product'].browse(combination_info['product_id'])"/>

                <tr class="js_product" t-if="not combination_info.get('prevent_zero_price_sale', False)">
                    <td class="td-img">
                        <input type="hidden" class="product_template_id" t-att-value="product.id"/>
                        <input type="hidden" class="product_id" t-attf-name="optional-product-#{product.id}" t-att-value="product_variant.id"/>
                        <img t-if="product_variant" t-att-src="'/web/image/product.product/%s/image_128' % product_variant.id"  class="variant_image" alt="Product Image"/>
                        <img t-else="" t-att-src="'/web/image/product.template/%s/image_128' % product.id"  class="variant_image" alt="Product Image"/>
                    </td>
                    <td class='td-product_name' colspan="2">
                        <div class="mb-3">
                            <strong class="product-name product_display_name" t-out="combination_info['display_name']"/>
                            <div class="text-muted small" t-field="product.description_sale"/>
                        </div>
                        <t t-call="website_sale.variants"/>
                    </td>
                    <td class="text-center td-qty d-none">
                        <t t-call='website_sale_product_configurator.product_quantity_config' />
                    </td>
                    <td class="text-center td-price">
                        <div
                            t-if="not combination_info.get('compare_list_price')"
                            t-attf-class="text-danger oe_default_price oe_optional oe_striked_price
                                {{'' if combination_info['has_discounted_price'] else 'd-none'}}"
                            t-out="combination_info['list_price']"
                            t-options="{'widget': 'monetary', 'display_currency': website.currency_id}"/>
                        <div
                            t-if="combination_info.get('compare_list_price')
                                and combination_info['compare_list_price']
                                &gt; combination_info['price']"
                            class="text-danger oe_striked_price"
                            t-out="combination_info['compare_list_price']"
                            groups="website_sale.group_product_price_comparison"
                            t-options='{
                                "widget": "monetary",
                                "display_currency": (pricelist or product).currency_id
                            }'
                        />
                        <div class="oe_price" style="white-space: nowrap;"
                            t-out="combination_info['price']"
                            t-options="{'widget': 'monetary', 'display_currency': website.currency_id}"/>
                        <span class="js_raw_price d-none" t-out="combination_info['price']" />
                        <p class="css_not_available_msg alert alert-warning">Option not available</p>

                        <a role="button" href="#" class="js_add btn btn-primary">
                            <i class="fa fa-shopping-cart add-optionnal-item"/>
                        </a>
                        <span class="js_remove d-none">
                            <a role="button" href="#" class="js_remove"><i class="fa fa-trash-o remove-optionnal-item"></i></a>
                        </span>
                    </td>
                </tr>
            </t>
        </t>
    </template>
</odoo>

```

