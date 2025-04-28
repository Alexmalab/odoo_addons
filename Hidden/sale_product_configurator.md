# Odoo Module: sale_product_configurator

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-

from . import models
from . import controllers
from . import wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': "Sale Product Configurator",
    'version': '1.0',
    'category': 'Hidden',
    'summary': "Configure your products",

    'description': """
Technical module installed when the user checks the "module_sale_product_configurator" setting.
The main purpose is to override the sale_order view to allow configuring products in the SO form.

It also enables the "optional products" feature.
    """,

    'depends': ['sale'],
    'data': [
        'views/assets.xml',
        'views/templates.xml',
        'views/sale_views.xml',
        'wizard/sale_product_configurator_views.xml',
    ],
    'demo': [
        'data/sale_demo.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http
from odoo.http import request


class ProductConfiguratorController(http.Controller):
    @http.route(['/sale_product_configurator/configure'], type='json', auth="user", methods=['POST'])
    def configure(self, product_template_id, pricelist_id, **kw):
        add_qty = float(kw.get('add_qty', 1))
        product_template = request.env['product.template'].browse(int(product_template_id))
        pricelist = self._get_pricelist(pricelist_id)

        product_combination = False
        attribute_value_ids = set(kw.get('product_template_attribute_value_ids', []))
        attribute_value_ids |= set(kw.get('product_no_variant_attribute_value_ids', []))
        if attribute_value_ids:
            product_combination = request.env['product.template.attribute.value'].browse(attribute_value_ids)

        if pricelist:
            product_template = product_template.with_context(pricelist=pricelist.id, partner=request.env.user.partner_id)

        return request.env['ir.ui.view'].render_template("sale_product_configurator.configure", {
            'product': product_template,
            'pricelist': pricelist,
            'add_qty': add_qty,
            'product_combination': product_combination
        })

    @http.route(['/sale_product_configurator/show_optional_products'], type='json', auth="user", methods=['POST'])
    def show_optional_products(self, product_id, variant_values, pricelist_id, **kw):
        pricelist = self._get_pricelist(pricelist_id)
        return self._show_optional_products(product_id, variant_values, pricelist, False, **kw)

    @http.route(['/sale_product_configurator/optional_product_items'], type='json', auth="user", methods=['POST'])
    def optional_product_items(self, product_id, pricelist_id, **kw):
        pricelist = self._get_pricelist(pricelist_id)
        return self._optional_product_items(product_id, pricelist, **kw)

    def _optional_product_items(self, product_id, pricelist, **kw):
        add_qty = float(kw.get('add_qty', 1))
        product = request.env['product.product'].browse(int(product_id))

        parent_combination = product.product_template_attribute_value_ids
        if product.env.context.get('no_variant_attribute_values'):
            # Add "no_variant" attribute values' exclusions
            # They are kept in the context since they are not linked to this product variant
            parent_combination |= product.env.context.get('no_variant_attribute_values')

        return request.env['ir.ui.view'].render_template("sale_product_configurator.optional_product_items", {
            'product': product,
            'parent_name': product.name,
            'parent_combination': parent_combination,
            'pricelist': pricelist,
            'add_qty': add_qty,
        })

    def _show_optional_products(self, product_id, variant_values, pricelist, handle_stock, **kw):
        product = request.env['product.product'].browse(int(product_id))
        combination = request.env['product.template.attribute.value'].browse(variant_values)
        has_optional_products = product.optional_product_ids.filtered(lambda p: p._is_add_to_cart_possible(combination))

        if not has_optional_products:
            return False

        add_qty = float(kw.get('add_qty', 1))

        no_variant_attribute_values = combination.filtered(
            lambda product_template_attribute_value: product_template_attribute_value.attribute_id.create_variant == 'no_variant'
        )
        if no_variant_attribute_values:
            product = product.with_context(no_variant_attribute_values=no_variant_attribute_values)

        return request.env['ir.ui.view'].render_template("sale_product_configurator.optional_products_modal", {
            'product': product,
            'combination': combination,
            'add_qty': add_qty,
            'parent_name': product.name,
            'variant_values': variant_values,
            'pricelist': pricelist,
            'handle_stock': handle_stock,
        })

    def _get_pricelist(self, pricelist_id, pricelist_fallback=False):
        return request.env['product.pricelist'].browse(int(pricelist_id or 0))

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\sale_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="product_product_1_product_template" model="product.template">
            <field name="name">Chair floor protection</field>
            <field name="categ_id" ref="product.product_category_5"/>
            <field name="list_price">12.0</field>
            <field name="weight">0.01</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="description_sale">Office chairs can harm your floor: protect it.</field>
            <field name="image_1920" type="base64" file="sale/static/img/floor_protection-image.png"/>
        </record>

        <record id="product.product_product_4_product_template" model="product.template">
            <field name="optional_product_ids" eval="[(6,0,[ref('product.product_product_11_product_template')])]"/>
        </record>
        <record id="product.product_product_11_product_template" model="product.template">
            <field name="optional_product_ids" eval="[(6,0,[ref('product_product_1_product_template')])]"/>
        </record>
        <record id="product.product_product_13_product_template" model="product.template">
            <field name="optional_product_ids" eval="[(6,0,[ref('product.product_product_11_product_template')])]"/>
        </record>
    </data>
</odoo>

```

## File: models\product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api


class ProductTemplate(models.Model):
    _inherit = 'product.template'
    _check_company_auto = True

    optional_product_ids = fields.Many2many(
        'product.template', 'product_optional_rel', 'src_id', 'dest_id',
        string='Optional Products', help="Optional Products are suggested "
        "whenever the customer hits *Add to Cart* (cross-sell strategy, "
        "e.g. for computers: warranty, software, etc.).", check_company=True)

    @api.depends('attribute_line_ids.value_ids.is_custom', 'attribute_line_ids.attribute_id.create_variant')
    def _compute_has_configurable_attributes(self):
        """ A product is considered configurable if:
        - It has dynamic attributes
        - It has any attribute line with at least 2 attribute values configured
        - It has at least one custom attribute value """
        for product in self:
            product.has_configurable_attributes = any(attribute.create_variant == 'dynamic' for attribute in product.mapped('attribute_line_ids.attribute_id')) \
                or any(len(attribute_line_id.value_ids) >= 2 for attribute_line_id in product.attribute_line_ids) \
                or any(attribute_value.is_custom for attribute_value in product.mapped('attribute_line_ids.value_ids'))

    def get_single_product_variant(self):
        """ Method used by the product configurator to check if the product is configurable or not.

        We need to open the product configurator if the product:
        - is configurable (see has_configurable_attributes)
        - has optional products """
        self.ensure_one()
        res = super(ProductTemplate, self).get_single_product_variant()
        if res.get('product_id', False):
            has_optional_products = False
            for optional_product in self.product_variant_id.optional_product_ids:
                if optional_product.has_dynamic_attributes() or optional_product._get_possible_variants(self.product_variant_id.product_template_attribute_value_ids):
                    has_optional_products = True
                    break
            res.update({'has_optional_products': has_optional_products})
        return res

```

## File: models\sale_order_line.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class SaleOrderLine(models.Model):
    _inherit = "sale.order.line"

    is_configurable_product = fields.Boolean('Is the product configurable?', related="product_template_id.has_configurable_attributes")
    product_template_attribute_value_ids = fields.Many2many(related='product_id.product_template_attribute_value_ids', readonly=True)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import sale_order_line
from . import product

```

## File: static\src\js\product_configurator_controller.js

```javascript
odoo.define('sale_product_configurator.ProductConfiguratorFormController', function (require) {
"use strict";

var core = require('web.core');
var _t = core._t;
var FormController = require('web.FormController');
var OptionalProductsModal = require('sale_product_configurator.OptionalProductsModal');

var ProductConfiguratorFormController = FormController.extend({
    custom_events: _.extend({}, FormController.prototype.custom_events, {
        field_changed: '_onFieldChanged',
        handle_add: '_handleAdd'
    }),
    /**
     * @override
     */
    start: function () {
        var self = this;
        return this._super.apply(this, arguments).then(function () {
            self.$el.addClass('o_product_configurator');
        });
    },
    /**
     * We need to first load the template of the selected product and then render the content
     * to avoid a flicker when the modal is opened.
     *
     * @override
     */
    willStart: function () {
        var def = this._super.apply(this, arguments);
        if (this.initialState.data.product_template_id) {
            return this._configureProduct(
                this.initialState.data.product_template_id.data.id
            ).then(function () {
                return def;
            });
        }

        return def;
    },
    /**
     * Showing this window is useless for configuratorMode 'options' as this form view
     * is used as a bridge between SO lines and optional products.
     *
     * Placed here because it's the only method that is called after the modal is rendered.
     *
     * @override
     */
    renderButtons: function () {
        this._super.apply(this, arguments);

        if (this.renderer.state.context.configuratorMode === 'options') {
            this.$el.closest('.modal').addClass('d-none');
        }
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------


    /**
    * We need to override the default click behavior for our "Add" button
    * because there is a possibility that this product has optional products.
    * If so, we need to display an extra modal to choose the options.
    *
    * @override
    */
    _onButtonClicked: function (event) {
        if (event.stopPropagation) {
            event.stopPropagation();
        }
        var attrs = event.data.attrs;
        if (attrs.special === 'cancel') {
            this._super.apply(this, arguments);
        } else {
            if (!this.$el
                    .parents('.modal')
                    .find('.o_sale_product_configurator_add')
                    .hasClass('disabled')) {
                this._handleAdd();
            }
        }
    },
    /**
     * This is overridden to allow catching the "select" event on our product template select field.
     *
     * @override
     * @private
     */
    _onFieldChanged: function (event) {
        this._super.apply(this, arguments);

        var self = this;
        var productId = event.data.changes.product_template_id.id;

        // check to prevent traceback when emptying the field
        if (!productId) {
            return;
        }

        this._configureProduct(event.data.changes.product_template_id.id)
            .then(function () {
                self.renderer.renderConfigurator(self.renderer.configuratorHtml);
            });
    },

    /**
     * Renders the "variants" part of the wizard
     *
     * @param {integer} productTemplateId
     */
    _configureProduct: function (productTemplateId) {
        var self = this;
        var initialProduct = this.initialState.data.product_template_id;
        var changed = initialProduct && initialProduct.data.id !== productTemplateId;
        var data = this.renderer.state.data;
        var quantity = initialProduct.context && initialProduct.context.default_quantity ? initialProduct.context.default_quantity : data.quantity;
        return this._rpc({
            route: '/sale_product_configurator/configure',
            params: {
                product_template_id: productTemplateId,
                pricelist_id: this.renderer.pricelistId,
                add_qty: quantity,
                product_template_attribute_value_ids: changed ? [] : this._getAttributeValueIds(
                    data.product_template_attribute_value_ids
                ),
                product_no_variant_attribute_value_ids: changed ? [] : this._getAttributeValueIds(
                    data.product_no_variant_attribute_value_ids
                )
            }
        }).then(function (configurator) {
            self.renderer.configuratorHtml = configurator;
        });
    },
    /**
    * When the user adds a product that has optional products, we need to display
    * a window to allow the user to choose these extra options.
    *
    * This will also create the product if it's in "dynamic" mode
    * (see product_attribute.create_variant)
    *
    * If "self.renderer.state.context.configuratorMode" is 'edit', this will only send
    * the main product with its changes.
    *
    * As opposed to the 'add' mode that will add the main product AND all the configured optional products.
    *
    * A third mode, 'options', is available for products that don't have a configuration but have
    * optional products to select. This will bypass the configuration step and open the
    * options modal directly.
    *
    * @private
    */
    _handleAdd: function () {
        var self = this;
        var $modal = this.$el;
        var productSelector = [
            'input[type="hidden"][name="product_id"]',
            'input[type="radio"][name="product_id"]:checked'
        ];

        var productId = parseInt($modal.find(productSelector.join(', ')).first().val(), 10);
        var productTemplateId = $modal.find('.product_template_id').val();
        this.renderer.selectOrCreateProduct(
            $modal,
            productId,
            productTemplateId,
            false
        ).then(function (productId) {
            $modal.find(productSelector.join(', ')).val(productId);

            var variantValues = self
                .renderer
                .getSelectedVariantValues($modal.find('.js_product'));

            var productCustomVariantValues = self
                .renderer
                .getCustomVariantValues($modal.find('.js_product'));

            var noVariantAttributeValues = self
                .renderer
                .getNoVariantAttributeValues($modal.find('.js_product'));

            self.rootProduct = {
                product_id: productId,
                product_template_id: parseInt(productTemplateId),
                quantity: parseFloat($modal.find('input[name="add_qty"]').val() || 1),
                variant_values: variantValues,
                product_custom_attribute_values: productCustomVariantValues,
                no_variant_attribute_values: noVariantAttributeValues
            };

            if (self.renderer.state.context.configuratorMode === 'edit') {
                // edit mode only takes care of main product
                self._onAddRootProductOnly();
                return;
            }

            self.optionalProductsModal = new OptionalProductsModal($('body'), {
                rootProduct: self.rootProduct,
                pricelistId: self.renderer.pricelistId,
                okButtonText: _t('Confirm'),
                cancelButtonText: _t('Back'),
                title: _t('Configure'),
                context: self.initialState.context,
                previousModalHeight: self.$el.closest('.modal-content').height()
            }).open();

            self.optionalProductsModal.on('options_empty', null,
                // no optional products found for this product, only add the root product
                self._onAddRootProductOnly.bind(self));

            self.optionalProductsModal.on('update_quantity', null,
                self._onOptionsUpdateQuantity.bind(self));

            self.optionalProductsModal.on('confirm', null,
                self._onModalConfirm.bind(self));

            self.optionalProductsModal.on('closed', null,
                self._onModalClose.bind(self));
        });
    },

    /**
     * Add root product only and forget optional products.
     * Used when product has no optional products and in 'edit' mode.
     *
     * @private
     */
    _onAddRootProductOnly: function () {
        this._addProducts([this.rootProduct]);
    },

    /**
     * Add all selected products
     *
     * @private
     */
    _onModalConfirm: function () {
        this._wasConfirmed = true;
        this._addProducts(this.optionalProductsModal.getSelectedProducts());
    },

    /**
     * When the optional products modal is closed (and not confirmed) on 'options' mode,
     * this window should also be closed immediately.
     *
     * @private
     */
    _onModalClose: function () {
        if (this.renderer.state.context.configuratorMode === 'options'
            && this._wasConfirmed !== true) {
              this.do_action({type: 'ir.actions.act_window_close'});
        }
    },

    /**
     * Update product configurator form
     * when quantity is updated in the optional products window
     *
     * @private
     * @param {integer} quantity
     */
    _onOptionsUpdateQuantity: function (quantity) {
        this.$el
            .find('input[name="add_qty"]')
            .val(quantity)
            .trigger('change');
    },

    /**
    * This triggers the close action for the window and
    * adds the product as the "infos" parameter.
    * It will allow the caller (typically the product_configurator widget) of this window
    * to handle the added products.
    *
    * @private
    * @param {Array} products the list of added products
    *   {integer} products.product_id: the id of the product
    *   {integer} products.quantity: the added quantity for this product
    *   {Array} products.product_custom_attribute_values:
    *     see variant_mixin.getCustomVariantValues
    *   {Array} products.no_variant_attribute_values:
    *     see variant_mixin.getNoVariantAttributeValues
    */
    _addProducts: function (products) {
        this.do_action({type: 'ir.actions.act_window_close', infos: {
            mainProduct: products[0],
            options: products.slice(1)
        }});
    },
    /**
     * Extracts the ids from the passed attributeValueIds and returns them
     * as a plain array.
     *
     * @param {Array} attributeValueIds
     */
    _getAttributeValueIds: function (attributeValueIds) {
        if (!attributeValueIds || attributeValueIds.length === 0) {
            return false;
        }

        var result = [];
        _.each(attributeValueIds.data, function (attributeValue) {
            result.push(attributeValue.data.id);
        });

        return result;
    }
});

return ProductConfiguratorFormController;

});

```

## File: static\src\js\product_configurator_modal.js

```javascript
odoo.define('sale_product_configurator.OptionalProductsModal', function (require) {
    "use strict";

var ajax = require('web.ajax');
var Dialog = require('web.Dialog');
var ServicesMixin = require('web.ServicesMixin');
var VariantMixin = require('sale.VariantMixin');

var OptionalProductsModal = Dialog.extend(ServicesMixin, VariantMixin, {
    events:  _.extend({}, Dialog.prototype.events, VariantMixin.events, {
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

        var options = _.extend({
            size: 'large',
            buttons: [{
                text: params.okButtonText,
                click: this._onConfirmButtonClick,
                classes: 'btn-primary'
            }, {
                text: params.cancelButtonText,
                click: this._onCancelButtonClick
            }],
            technical: !params.isWebsite,
        }, params || {});

        this._super(parent, options);

        this.context = params.context;
        this.rootProduct = params.rootProduct;
        this.container = parent;
        this.pricelistId = params.pricelistId;
        this.previousModalHeight = params.previousModalHeight;
        this.dialogClass = 'oe_optional_products_modal';
        this._productImageField = 'image_128';

        this._opened.then(function () {
            if (self.previousModalHeight) {
                self.$el.closest('.modal-content').css('min-height', self.previousModalHeight + 'px');
            }
        });
    },
     /**
     * @override
     */
    willStart: function () {
        var self = this;

        var uri = this._getUri("/sale_product_configurator/show_optional_products");
        var getModalContent = ajax.jsonRpc(uri, 'call', {
            product_id: self.rootProduct.product_id,
            variant_values: self.rootProduct.variant_values,
            pricelist_id: self.pricelistId || false,
            add_qty: self.rootProduct.quantity,
            kwargs: {
                context: _.extend({
                    'quantity': self.rootProduct.quantity
                }, this.context),
            }
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
                self.$modal.removeAttr("aria-hidden");
                self.$modal.modal().appendTo(self.container);
                self.$modal.focus();
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
        var $products = this.$el.find('tr.js_product');
        _.each($products, function (el) {
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
    getSelectedProducts: function () {
        var self = this;
        var products = [this.rootProduct];
        this.$modal.find('.js_product.in_cart:not(.main_product)').each(function () {
            var $item = $(this);
            var quantity = parseFloat($item.find('input[name="add_qty"]').val().replace(',', '.') || 1);
            var parentUniqueId = this.dataset.parentUniqueId;
            var uniqueId = this.dataset.uniqueId;
            var productCustomVariantValues = self.getCustomVariantValues($(this));
            var noVariantAttributeValues = self.getNoVariantAttributeValues($(this));
            products.push({
                'product_id': parseInt($item.find('input.product_id').val(), 10),
                'product_template_id': parseInt($item.find('input.product_template_id').val(), 10),
                'quantity': quantity,
                'parent_unique_id': parentUniqueId,
                'unique_id': uniqueId,
                'product_custom_attribute_values': productCustomVariantValues,
                'no_variant_attribute_values': noVariantAttributeValues
            });
        });

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

            $.each(this.rootProduct.product_custom_attribute_values, function (){
                $updatedDescription.append($('<div>', {
                    text: this.attribute_value_name + ': ' + this.custom_value
                }));
            });

            $.each(this.rootProduct.no_variant_attribute_values, function (){
                if (this.is_custom !== 'True'){
                    $updatedDescription.append($('<div>', {
                        text: this.attribute_name + ': ' + this.attribute_value_name
                    }));
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
        var $modal = $target.parents('.oe_optional_products_modal');
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
                .find('td.td-product_name div.float-left');

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
                    $customAttributeValuesDescription.append($('<div>', {
                        text: this.attribute_name + ': ' + this.attribute_value_name
                    }));
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

            ajax.jsonRpc(self._getUri("/sale_product_configurator/optional_product_items"), 'call', {
                'product_id': productId,
                'pricelist_id': self.pricelistId || false,
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
        return true;
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
            el.dataset.uniqueId = parseInt(_.uniqueId(), 10);
        }
        return el.dataset.uniqueId;
    },
});

return OptionalProductsModal;

});

```

## File: static\src\js\product_configurator_renderer.js

```javascript
odoo.define('sale_product_configurator.ProductConfiguratorFormRenderer', function (require) {
"use strict";

var FormRenderer = require('web.FormRenderer');
var VariantMixin = require('sale.VariantMixin');

var ProductConfiguratorFormRenderer = FormRenderer.extend(VariantMixin, {

    events: _.extend({}, FormRenderer.prototype.events, VariantMixin.events, {
        'click button.js_add_cart_json': 'onClickAddCartJSON',
    }),

    /**
     * @override
     */
    init: function () {
        this._super.apply(this, arguments);
        this.pricelistId = this.state.context.default_pricelist_id || 0;
    },
    /**
     * @override
     */
    start: function () {
        var self = this;
        return this._super.apply(this, arguments).then(function () {
            self.$el.append($('<div>', {class: 'configurator_container'}));
            self.renderConfigurator(self.configuratorHtml);
            self._checkMode();
        });
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * Renders the product configurator within the form
     *
     * Will also:
     *
     * - add events handling for variant changes
     * - trigger variant change to compute the price and other
     *   variant specific changes
     *
     * @param {string} configuratorHtml the evaluated template of
     *   the product configurator
     */
    renderConfigurator: function (configuratorHtml) {
        var $configuratorContainer = this.$('.configurator_container');
        $configuratorContainer.empty();

        var $configuratorHtml = $(configuratorHtml);
        $configuratorHtml.appendTo($configuratorContainer);

        this.triggerVariantChange($configuratorContainer);
        this._applyCustomValues();
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * If the configuratorMode in the given context is 'edit', we need to
     * hide the regular 'Add' button to replace it with an 'EDIT' button.
     *
     * If the configuratorMode is set to 'options', we will directly open the
     * options modal.
     *
     * @private
     */
    _checkMode: function () {
        if (this.state.context.configuratorMode === 'edit') {
            this.$('.o_sale_product_configurator_add').hide();
            this.$('.o_sale_product_configurator_edit').css('display', 'inline-block');
        } else if (this.state.context.configuratorMode === 'options') {
            this.trigger_up('handle_add');
        }
    },

    /**
     * Toggles the add button depending on the possibility of the current
     * combination.
     *
     * @override
     */
    _toggleDisable: function ($parent, isCombinationPossible) {
        VariantMixin._toggleDisable.apply(this, arguments);
        $parent.parents('.modal').find('.o_sale_product_configurator_add').toggleClass('disabled', !isCombinationPossible);
    },

    /**
     * Will fill the custom values input based on the provided initial configuration.
     *
     * @private
     */
    _applyCustomValues: function () {
        var self = this;
        var customValueIds = this.state.data.product_custom_attribute_value_ids;
        if (customValueIds) {
            _.each(customValueIds.data, function (customValue) {
                if (customValue.data.custom_value) {
                    var attributeValueId = customValue.data.custom_product_template_attribute_value_id.data.id;
                    var $input = self._findRelatedAttributeValueInput(attributeValueId);
                    $input
                        .closest('li[data-attribute_id]')
                        .find('.variant_custom_value')
                        .val(customValue.data.custom_value);
                }
            });
        }
    },

    /**
     * Find the $.Element input/select related to that product.attribute.value
     *
     * @param {integer} attributeValueId
     *
     * @private
     */
    _findRelatedAttributeValueInput: function (attributeValueId) {
        var selectors = [
            'ul.js_add_cart_variants input[data-value_id="' + attributeValueId + '"]',
            'ul.js_add_cart_variants option[data-value_id="' + attributeValueId + '"]'
        ];

        return this.$(selectors.join(', '));
    }
});

return ProductConfiguratorFormRenderer;

});

```

## File: static\src\js\product_configurator_view.js

```javascript
odoo.define('sale_product_configurator.ProductConfiguratorFormView', function (require) {
"use strict";

var ProductConfiguratorFormController = require('sale_product_configurator.ProductConfiguratorFormController');
var ProductConfiguratorFormRenderer = require('sale_product_configurator.ProductConfiguratorFormRenderer');
var FormView = require('web.FormView');
var viewRegistry = require('web.view_registry');

var ProductConfiguratorFormView = FormView.extend({
    config: _.extend({}, FormView.prototype.config, {
        Controller: ProductConfiguratorFormController,
        Renderer: ProductConfiguratorFormRenderer,
    }),
});

viewRegistry.add('product_configurator_form', ProductConfiguratorFormView);

return ProductConfiguratorFormView;

});

```

## File: static\src\js\product_configurator_widget.js

```javascript
odoo.define('sale_product_configurator.product_configurator', function (require) {
var ProductConfiguratorWidget = require('sale.product_configurator');

/**
 * Extension of the ProductConfiguratorWidget to support product configuration.
 * It opens when a configurable product_template is set.
 * (multiple variants, or custom attributes)
 *
 * The product customization information includes :
 * - is_configurable_product
 * - product_template_attribute_value_ids
 *
 */
ProductConfiguratorWidget.include({
    /**
     * Override of sale.product_configurator Hook
     *
     * @override
    */
    _isConfigurableProduct: function () {
        return this.recordData.is_configurable_product || this._super.apply(this, arguments);
    },

    /**
     * Set restoreProductTemplateId for further backtrack.
     * Saves the optional products in the widget for future application
     * post-line configuration.
     *
     * {OdooEvent ev}
     *    {Array} ev.data.optionalProducts the various selected optional products
     *     with their configuration
     *
     * @override
     * @private
     */
    _onFieldChanged: function (ev) {
        this.restoreProductTemplateId = this.recordData.product_template_id;
        this.optionalProducts = (ev.data && ev.data.optionalProducts) || this.optionalProducts;

        this._super.apply(this, arguments);
    },

    /**
    * This method is overridden to check if the product_template_id
    * needs configuration or not:
    *
    * - The product_template has only one "product.product" and is not dynamic
    *   -> Set the product_id on the SO line
    *   -> If the product has optional products, open the configurator in 'options' mode
    *
    * - The product_template is configurable
    *   -> Open the product configurator wizard and initialize it with
    *      the provided product_template_id and its current attribute values
    *
    * @override
    * @private
    */
    _onTemplateChange: function (productTemplateId, dataPointId) {
        var self = this;

        return this._rpc({
            model: 'product.template',
            method: 'get_single_product_variant',
            args: [
                productTemplateId
            ]
        }).then(function (result) {
            if (result.product_id && !result.has_optional_products) {
                self.trigger_up('field_changed', {
                    dataPointID: dataPointId,
                    changes: {
                        product_id: {id: result.product_id},
                        product_custom_attribute_value_ids: {
                            operation: 'DELETE_ALL'
                        }
                    },
                });
            } else {
                return self._openConfigurator(result, productTemplateId, dataPointId);
            }
            // always returns true for the moment because no other configurator exists.
        });
    },

    /**
     *  When line is configured, apply the options defined earlier.
     *  @override
     *  @private
     */
    _onLineConfigured: function () {
        var self = this;
        this._super.apply(this, arguments);
        var parentList = self.getParent();
        var unselectRow = (parentList.unselectRow || function() {}).bind(parentList); // form view on mobile
        if (self.optionalProducts && self.optionalProducts.length !== 0) {
            self.trigger_up('add_record', {
                context: self._productsToRecords(self.optionalProducts),
                forceEditable: 'bottom',
                allowWarning: true,
                onSuccess: function () {
                    // Leave edit mode of one2many list.
                    unselectRow();
                }
            });
        } else if (!self._isConfigurableLine() && self._isConfigurableProduct()) {
            // Leave edit mode of current line if line was configured
            // only through the product configurator.
            unselectRow();
        }
    },

    _openConfigurator: function (result, productTemplateId, dataPointId) {
        if (!result.mode || result.mode === 'configurator') {
            this._openProductConfigurator({
                    configuratorMode: result && result.has_optional_products ? 'options' : 'add',
                    default_pricelist_id: this._getPricelistId(),
                    default_product_template_id: productTemplateId
                },
                dataPointId
            );
            return Promise.resolve(true);
        }
        return Promise.resolve(false);
    },

    /**
     * Opens the product configurator to allow configuring the product template
     * and its various options.
     *
     * The configuratorMode param controls how to open the configurator.
     * - The "add" mode will allow configuring the product template & options.
     * - The "edit" mode will only allow editing the product template's configuration.
     * - The "options" mode is a special case where the product configurator is used as a bridge
     *   between the SO line and the optional products modal. It will hide its window and handle
     *   the communication between those two.
     *
     * When the configuration is canceled (i.e when the product configurator is closed using the
     * "CANCEL" button or the cross on the top right corner of the window),
     * the product_template is reset to its previous value if any.
     *
     * @param {Object} data various "default_" values
     *  {string} data.configuratorMode 'add' or 'edit' or 'options'.
     * @param {string} dataPointId
     *
     * @private
     */
    _openProductConfigurator: function (data, dataPointId) {
        this.optionalProducts = undefined;
        var self = this;
        this.do_action('sale_product_configurator.sale_product_configurator_action', {
            additional_context: data,
            on_close: function (result) {
                if (result && !result.special) {
                    self._addProducts(result, dataPointId);
                } else {
                    if (self.restoreProductTemplateId) {
                        // if configurator opened in edit mode.
                        self.trigger_up('field_changed', {
                            dataPointID: dataPointId,
                            preventProductIdCheck: true,
                            changes: {
                                product_template_id: self.restoreProductTemplateId.data
                            }
                        });
                    } else {
                        // if configurator opened to create line:
                        // destroy line if configurator closed during configuration process.
                        self.trigger_up('field_changed', {
                            dataPointID: dataPointId,
                            changes: {
                                product_template_id: false,
                                product_id: false,
                            },
                        });
                    }
                }
            }
        });
    },

    /**
     * Opens the product configurator in "edit" mode.
     * (see '_openProductConfigurator' for more info on the "edit" mode).
     * The requires to retrieve all the needed data from the SO line
     * that are kept in the "recordData" object.
     *
     * @private
     */
    _onEditProductConfiguration: function () {
        if (!this.recordData.is_configurable_product) {
            // if line should be edited by another configurator
            // or simply inline.
            this._super.apply(this, arguments);
            return;
        }
        this.restoreProductTemplateId = this.recordData.product_template_id;
        // If line has been set up through the product_configurator:
        this._openProductConfigurator({
                configuratorMode: 'edit',
                default_product_template_id: this.recordData.product_template_id.data.id,
                default_pricelist_id: this._getPricelistId(),
                default_product_template_attribute_value_ids: this._convertFromMany2Many(
                    this.recordData.product_template_attribute_value_ids
                ),
                default_product_no_variant_attribute_value_ids: this._convertFromMany2Many(
                    this.recordData.product_no_variant_attribute_value_ids
                ),
                default_product_custom_attribute_value_ids: this._convertFromOne2Many(
                    this.recordData.product_custom_attribute_value_ids
                ),
                default_quantity: this.recordData.product_uom_qty
            },
            this.dataPointID
        );
    },

    /**
     * This will first modify the SO line to update all the information coming from
     * the product configurator using the 'field_changed' event.
     *
     * onSuccess from that first method, it will add the optional products to the SO
     * using the 'add_record' event.
     *
     * Doing both at the same time could lead to unordered product_template/options.
     *
     * @param {Object} products the products to add to the SO line.
     *   {Object} products.mainProduct the product_template configured
     *     with various attribute/custom values
     *   {Array} products.options the various selected optional products
     *     with their configuration
     * @param {string} dataPointId
     *
     * @private
     */
    _addProducts: function (result, dataPointId) {
        this.trigger_up('field_changed', {
            dataPointID: dataPointId,
            preventProductIdCheck: true,
            optionalProducts: result.options,
            changes: this._getMainProductChanges(result.mainProduct)
        });
    },

    /**
     * This will convert the result of the product configurator into
     * "changes" that are understood by the basic_model.js
     *
     * For the product_custom_attribute_value_ids, we need to do a DELETE_ALL
     * command to clean the currently selected values and then a CREATE for every
     * custom value specified in the configurator.
     *
     * For the product_no_variant_attribute_value_ids, we also need to do a DELETE_ALL
     * command to clean the currently selected values and issue a single ADD_M2M containing
     * all the ids of the product_attribute_values.
     *
     * @param {Object} mainProduct
     *
     * @private
     */
    _getMainProductChanges: function (mainProduct) {
        var result = {
            product_id: {id: mainProduct.product_id},
            product_template_id: {id: mainProduct.product_template_id},
            product_uom_qty: mainProduct.quantity
        };

        var customAttributeValues = mainProduct.product_custom_attribute_values;
        var customValuesCommands = [{operation: 'DELETE_ALL'}];
        if (customAttributeValues && customAttributeValues.length !== 0) {
            _.each(customAttributeValues, function (customValue) {
                // FIXME awa: This could be optimized by adding a "disableDefaultGet" to avoid
                // having multiple default_get calls that are useless since we already
                // have all the default values locally.
                // However, this would mean a lot of changes in basic_model.js to handle
                // those "default_" values and set them on the various fields (text,o2m,m2m,...).
                // -> This is not considered as worth it right now.
                customValuesCommands.push({
                    operation: 'CREATE',
                    context: [{
                        default_custom_product_template_attribute_value_id: customValue.custom_product_template_attribute_value_id,
                        default_custom_value: customValue.custom_value
                    }]
                });
            });
        }

        result['product_custom_attribute_value_ids'] = {
            operation: 'MULTI',
            commands: customValuesCommands
        };

        var noVariantAttributeValues = mainProduct.no_variant_attribute_values;
        var noVariantCommands = [{operation: 'DELETE_ALL'}];
        if (noVariantAttributeValues && noVariantAttributeValues.length !== 0) {
            var resIds = _.map(noVariantAttributeValues, function (noVariantValue) {
                return {id: parseInt(noVariantValue.value)};
            });

            noVariantCommands.push({
                operation: 'ADD_M2M',
                ids: resIds
            });
        }

        result['product_no_variant_attribute_value_ids'] = {
            operation: 'MULTI',
            commands: noVariantCommands
        };

        return result;
    },

    /**
     * Returns the pricelist_id set on the sale_order form
     *
     * @private
     * @returns {integer} pricelist_id's id
     */
    _getPricelistId: function () {
        return this.record.evalContext.parent.pricelist_id;
    },

    /**
     * Will map the products to appropriate record objects that are
     * ready for the default_get.
     *
     * @param {Array} products The products to transform into records
     *
     * @private
     */
    _productsToRecords: function (products) {
        var records = [];
        _.each(products, function (product) {
            var record = {
                default_product_id: product.product_id,
                default_product_template_id: product.product_template_id,
                default_product_uom_qty: product.quantity
            };

            if (product.no_variant_attribute_values) {
                var defaultProductNoVariantAttributeValues = [];
                _.each(product.no_variant_attribute_values, function (attributeValue) {
                        defaultProductNoVariantAttributeValues.push(
                            [4, parseInt(attributeValue.value)]
                        );
                });
                record['default_product_no_variant_attribute_value_ids']
                    = defaultProductNoVariantAttributeValues;
            }

            if (product.product_custom_attribute_values) {
                var defaultCustomAttributeValues = [];
                _.each(product.product_custom_attribute_values, function (attributeValue) {
                    defaultCustomAttributeValues.push(
                            [0, 0, {
                                custom_product_template_attribute_value_id: attributeValue.custom_product_template_attribute_value_id,
                                custom_value: attributeValue.custom_value
                            }]
                        );
                });
                record['default_product_custom_attribute_value_ids']
                    = defaultCustomAttributeValues;
            }

            records.push(record);
        });

        return records;
    }
});

return ProductConfiguratorWidget;

});

```

## File: views\assets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="assets_backend" inherit_id="web.assets_backend" name="sale_product_configurator assets backend">
        <xpath expr="script[last()]" position="after">
            <script type="text/javascript" src="/sale/static/src/js/variant_mixin.js"></script>
            <script type="text/javascript" src="/sale_product_configurator/static/src/js/product_configurator_renderer.js"></script>
            <script type="text/javascript" src="/sale_product_configurator/static/src/js/product_configurator_controller.js"></script>
            <script type="text/javascript" src="/sale_product_configurator/static/src/js/product_configurator_view.js"></script>
            <script type="text/javascript" src="/sale_product_configurator/static/src/js/product_configurator_widget.js"/>
            <script type="text/javascript" src="/sale_product_configurator/static/src/js/product_configurator_modal.js"></script>
        </xpath>
    </template>
    <template id="assets_tests" name="Sale Product Configurator Assets Tests" inherit_id="web.assets_tests">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/sale_product_configurator/static/tests/tours/product_configurator_ui.js"/>
            <script type="text/javascript" src="/sale_product_configurator/static/tests/tours/product_configurator_advanced_ui.js"/>
            <script type="text/javascript" src="/sale_product_configurator/static/tests/tours/product_configurator_edition_ui.js"/>
            <script type="text/javascript" src="/sale_product_configurator/static/tests/tours/product_configurator_single_custom_attribute_ui.js"/>
            <script type="text/javascript" src="/sale_product_configurator/static/tests/tours/product_configurator_pricelist_ui.js"/>
            <script type="text/javascript" src="/sale_product_configurator/static/tests/tours/product_configurator_optional_products_ui.js"/>
        </xpath>
    </template>
    <template id="qunit_suite" inherit_id="web.qunit_suite" name="sale_product_configurator_tests">
        <xpath expr="//t[@t-set='head']" position="inside">
            <script type="text/javascript" src="/sale_product_configurator/static/tests/product_configurator.test.js"></script>
        </xpath>
    </template>
</odoo>

```

## File: views\sale_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="product_template_view_form" model="ir.ui.view">
        <field name="name">product.template.form.inherit.sale.product.configurator</field>
        <field name="model">product.template</field>
        <field name="inherit_id" ref="product.product_template_form_view"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='sale']" position="after">
                <group name="options" groups="product.group_product_variant">
                    <group string="Options">
                        <field name="optional_product_ids" widget="many2many_tags" options="{'color_field': 'color'}" domain="[('id', '!=', active_id), '|', ('company_id', '=', company_id), ('company_id', '=', False)]" />
                    </group>
                </group>
            </xpath>
        </field>
    </record>

    <record id="sale_order_view_form" model="ir.ui.view">
        <field name="name">sale.order.form.inherit.sale.product.configurator</field>
        <field name="model">sale.order</field>
        <field name="inherit_id" ref="sale.view_order_form"/>
        <field name="arch" type="xml">
            <xpath expr="//tree/field[@name='product_template_id']" position="attributes">
                <attribute name="invisible">0</attribute>
            </xpath>
            <xpath expr="//tree/field[@name='product_template_id']" position="after">
                <field name="product_template_attribute_value_ids" invisible="1" />
                <field name="product_custom_attribute_value_ids" invisible="1" >
                    <tree>
                        <field name="custom_product_template_attribute_value_id" />
                        <field name="custom_value" />
                    </tree>
                </field>
                <field name="product_no_variant_attribute_value_ids" invisible="1" />
                <field name="is_configurable_product" invisible="1" />
            </xpath>
            <xpath expr="//tree/field[@name='product_id']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="optional_products_modal" name="Optional Products">
        <main class="modal-body">
            <t t-call="sale_product_configurator.configure_optional_products" />
        </main>
    </template>

    <!-- backend -->
    <template id="configure" name="Configure">
        <div class="js_product main_product">

            <t t-set="combination" t-value="product_combination if product_combination else product._get_first_possible_combination()"/>
            <t t-set="combination_info" t-value="product._get_combination_info(combination, add_qty=add_qty or 1, pricelist=pricelist)"/>
            <t t-set="product_variant" t-value="product.env['product.product'].browse(combination_info['product_id'])"/>

            <input type="hidden" class="product_template_id" t-att-value="product.id"/>
            <input type="hidden" class="product_id" t-attf-name="product_id" t-att-value="product_variant.id"/>
            <div class="col-lg-12 text-center mt-5">
                <t t-if="product._is_add_to_cart_possible()">
                    <div class="col-lg-5 d-inline-block text-left">
                        <t t-if="combination" t-call="sale.variants">
                            <t t-set="parent_combination" t-value="None"/>
                        </t>
                        <h2>
                            <span t-attf-class="text-danger oe_default_price oe_striked_price {{'' if combination_info['has_discounted_price'] else 'd-none'}}"
                                t-esc="combination_info['list_price']"
                                t-options='{
                                    "widget": "monetary",
                                    "display_currency": (pricelist or product).currency_id
                                }'/>
                            <span class="oe_price product_id mt-3" style="white-space: nowrap;"
                                t-att-data-product-id="product.id"
                                t-esc="combination_info['price']"
                                t-options='{
                                    "widget": "monetary",
                                    "display_currency": (pricelist or product).currency_id
                                }'/>
                        </h2>
                        <div class="css_quantity input-group" t-if="product.visible_qty_configurator">
                            <div class="input-group-prepend">
                                <button t-attf-href="#" class="btn btn-primary js_add_cart_json d-none d-md-inline-block" aria-label="Remove one" title="Remove one">
                                    <i class="fa fa-minus"></i>
                                </button>
                            </div>
                            <input type="text" class="js_quantity form-control quantity" data-min="1" name="add_qty" t-att-value="add_qty or 1"/>
                            <div class="input-group-append">
                                <button t-attf-href="#" class="btn btn-primary float_left js_add_cart_json d-none d-md-inline-block" aria-label="Add one" title="Add one">
                                    <i class="fa fa-plus"></i>
                                </button>
                            </div>
                        </div>
                        <p class="css_not_available_msg alert alert-warning">This combination does not exist.</p>
                    </div>
                    <div class="col-lg-1 d-inline-block"></div>
                    <div class="col-lg-5 d-inline-block align-top text-left">
                        <img t-if="product_variant" t-att-src="'/web/image/product.product/%s/image_1024' % product_variant.id" class="d-block product_detail_img" alt="Product Image"/>
                        <img t-else="" t-att-src="'/web/image/product.template/%s/image_1024' % product.id" class="d-block product_detail_img" alt="Product Image"/>
                    </div>
                </t>
                <t t-else="">
                    <div class="col-lg-5 d-inline-block text-left">
                        <p class="alert alert-warning">This product has no valid combination.</p>
                    </div>
                </t>
            </div>
        </div>
    </template>

    <!-- modal: full table, currenclty selected products at top -->
    <template id="configure_optional_products">
        <table class="table table-striped table-sm">
        <thead>
            <tr>
                <th class="td-img">Product</th>
                <th></th>
                <th class="text-center td-qty">Quantity</th>
                <th class="text-center td-price">Price</th>
            </tr>
        </thead>
        <tbody>
            <tr class="js_product in_cart main_product">

                <t t-set="combination_info" t-value="product.product_tmpl_id._get_combination_info(combination, product.id, add_qty or 1, pricelist)"/>
                <t t-set="product_variant" t-value="product.env['product.product'].browse(combination_info['product_id'])"/>

                <input type="hidden" class="product_template_id" t-att-value="product.product_tmpl_id.id"/>
                <input type="hidden" class="product_id" t-att-value="product_variant.id"/>
                <td class='td-img'>
                    <img t-if="product_variant" t-att-src="'/web/image/product.product/%s/image_128' % product_variant.id" alt="Product Image"/>
                    <img t-else="" t-att-src="'/web/image/product.template/%s/image_128' % product.id" alt="Product Image"/>
                </td>
                <td class='td-product_name'>
                    <strong t-esc="combination_info['display_name']"/>
                    <div class="text-muted small">
                        <div t-field="product.description_sale"/>
                        <div class="js_attributes"/>
                    </div>
                </td>
                <td class="text-center td-qty">
                    <div class="css_quantity input-group">
                        <div class="input-group-prepend">
                            <button t-attf-href="#" class="btn btn-primary js_add_cart_json d-none d-md-inline-block" aria-label="Remove one" title="Remove one">
                                <i class="fa fa-minus"></i>
                            </button>
                        </div>
                        <input type="text" class="js_quantity form-control quantity" data-min="1" name="add_qty" t-att-value="add_qty or 1"/>
                        <div class="input-group-append">
                            <button t-attf-href="#" class="btn btn-primary float_left js_add_cart_json d-none d-md-inline-block" aria-label="Add one" title="Add one">
                                <i class="fa fa-plus"></i>
                            </button>
                        </div>
                    </div>
                </td>
                <td class="text-center td-price" name="price">
                    <ul class="d-none js_add_cart_variants" t-att-data-attribute_exclusions="{'exclusions: []'}"></ul>
                    <div class="d-none oe_unchanged_value_ids" t-att-data-unchanged_value_ids="variant_values" ></div>
                    <div t-attf-class="text-danger oe_default_price oe_striked_price {{'' if combination_info['has_discounted_price'] else 'd-none'}}"
                        t-esc="combination_info['list_price']"
                        t-options='{
                            "widget": "monetary",
                            "display_currency": (pricelist or product).currency_id
                        }'
                    />
                    <span class="oe_price product_id" style="white-space: nowrap;"
                        t-att-data-product-id="product.id"
                        t-esc="combination_info['price']"
                        t-options='{
                            "widget": "monetary",
                            "display_currency": (pricelist or product).currency_id
                        }'/>
                    <span class="js_raw_price d-none" t-esc="product.price"/>
                </td>
            </tr>
            <tr class="o_total_row">
                <td colspan="4" class="text-right">
                    <strong>Total:</strong>
                    <span class="js_price_total font-weight-bold" style="white-space: nowrap;"
                        t-att-data-product-id="product.id"
                        t-esc="combination_info['price'] * (add_qty or 1)"
                        t-options='{
                            "widget": "monetary",
                            "display_currency": (pricelist or product).currency_id
                        }'/>
                </td>
            </tr>
            <tr t-if="product.optional_product_ids" class="o_select_options"><td colspan="4"><h4>Available Options:</h4></td></tr>
            <t t-call="sale_product_configurator.optional_product_items">
                <t t-set="parent_combination" t-value="combination"/>
            </t>
        </tbody>
        </table>
    </template>

    <!-- modal: optional products -->
    <template id="optional_product_items">
        <t t-foreach="product.optional_product_ids" t-as="product">
            <t t-set="combination" t-value="product._get_first_possible_combination(parent_combination)"/>
            <t t-if="product._is_add_to_cart_possible(parent_combination)">

                <t t-set="combination_info" t-value="product._get_combination_info(combination, add_qty=add_qty or 1, pricelist=pricelist)"/>
                <t t-set="product_variant" t-value="product.env['product.product'].browse(combination_info['product_id'])"/>

                <tr class="js_product">
                    <td class="td-img">
                        <input type="hidden" class="product_template_id" t-att-value="product.id"/>
                        <input type="hidden" class="product_id" t-attf-name="optional-product-#{product.id}" t-att-value="product_variant.id"/>
                        <img t-if="product_variant" t-att-src="'/web/image/product.product/%s/image_128' % product_variant.id"  class="variant_image" alt="Product Image"/>
                        <img t-else="" t-att-src="'/web/image/product.template/%s/image_128' % product.id"  class="variant_image" alt="Product Image"/>
                    </td>
                    <td class='td-product_name' colspan="2">
                        <div class="float-left">
                            <strong class="product-name product_display_name" t-esc="combination_info['display_name']"/>
                            <div class="text-muted small" t-field="product.description_sale"/>
                        </div>
                        <div class="float-right">
                            <t t-call="sale.variants"/>
                        </div>
                    </td>
                    <td class="text-center td-qty d-none">
                        <div class="css_quantity input-group">
                            <div class="input-group-prepend">
                                <button t-attf-href="#" class="btn btn-primary float_left js_add_cart_json d-none d-md-inline-block" aria-label="Remove one" title="Remove one">
                                    <i class="fa fa-minus"></i>
                                </button>
                            </div>
                            <input type="text" class="js_quantity form-control quantity" data-min="1" name="add_qty" t-att-value="add_qty or 1"/>
                            <div class="input-group-append">
                                <button t-attf-href="#" class="btn btn-primary float_left js_add_cart_json d-none d-md-inline-block" aria-label="Add one" title="Add one">
                                    <i class="fa fa-plus"></i>
                                </button>
                            </div>
                        </div>
                    </td>
                    <td class="text-center td-price">
                        <div t-attf-class="text-danger oe_default_price oe_optional oe_striked_price {{'' if combination_info['has_discounted_price'] else 'd-none'}}"
                            t-esc="combination_info['list_price']"
                            t-options='{
                                "widget": "monetary",
                                "display_currency": (pricelist or product).currency_id
                            }'/>
                        <div class="oe_price" style="white-space: nowrap;"
                            t-esc="combination_info['price']"
                            t-options='{
                                "widget": "monetary",
                                "display_currency": (pricelist or product).currency_id
                            }'/>
                        <span class="js_raw_price d-none" t-esc="combination_info['price']" />
                        <p class="css_not_available_msg alert alert-warning">Option not available</p>

                        <a role="button" href="#" class="js_add btn btn-primary btn-sm"><i class="fa fa-shopping-cart add-optionnal-item"></i> Add to cart</a>
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

## File: wizard\sale_product_configurator.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields


class SaleProductConfigurator(models.TransientModel):
    _name = 'sale.product.configurator'
    _description = 'Sale Product Configurator'

    product_template_id = fields.Many2one(
        'product.template', string="Product",
        required=True, domain=[('sale_ok', '=', True), ('attribute_line_ids.value_ids', '!=', False)])
    quantity = fields.Integer('Quantity')
    pricelist_id = fields.Many2one('product.pricelist', 'Pricelist', readonly=True)
    product_template_attribute_value_ids = fields.Many2many(
        'product.template.attribute.value', 'product_configurator_template_attribute_value_rel', string='Attribute Values', readonly=True)
    product_custom_attribute_value_ids = fields.Many2many(
        'product.attribute.custom.value', 'product_configurator_custom_attribute_value_rel', string="Custom Values")
    product_no_variant_attribute_value_ids = fields.Many2many(
        'product.template.attribute.value', 'product_configurator_no_variant_attribute_value_rel', string="Extra Values")

```

## File: wizard\sale_product_configurator_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="sale_product_configurator_view_form" model="ir.ui.view">
        <field name="name">sale_product_configurator.product.configurator.view.form</field>
        <field name="model">sale.product.configurator</field>
        <field name="arch" type="xml">
            <form js_class="product_configurator_form">
                <group>
                    <field name="product_template_id" class="oe_product_configurator_product_template_id" />
                    <field name="product_template_attribute_value_ids" invisible="1">
                        <tree limit="10000"/>
                    </field>
                    <field name="product_custom_attribute_value_ids" invisible="1" widget="one2many" >
                        <tree limit="10000">
                            <field name="custom_product_template_attribute_value_id"/>
                            <field name="custom_value"/>
                        </tree>
                    </field>
                    <field name="product_no_variant_attribute_value_ids" invisible="1">
                        <tree limit="10000"/>
                    </field>
                    <field name="quantity" invisible="1" />
                </group>
                <footer>
                    <button string="Add" class="btn-primary o_sale_product_configurator_add" special="add"/>
                    <button string="Save" class="btn-primary o_sale_product_configurator_edit" style="display: none;" special="save"/>
                    <button string="Cancel" class="btn-secondary" special="cancel"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="sale_product_configurator_action" model="ir.actions.act_window">
        <field name="name">Configure a product</field>
        <field name="res_model">sale.product.configurator</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
        <field name="view_id" ref="sale_product_configurator_view_form"/>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import sale_product_configurator

```

