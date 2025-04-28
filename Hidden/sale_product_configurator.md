# Odoo Module: sale_product_configurator

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
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': "Sale Product Configurator",
    'version': '1.0',
    'category': 'Hidden',
    'summary': "Configure your products",

    'description': """
Technical module:
The main purpose is to override the sale_order view to allow configuring products in the SO form.

It also enables the "optional products" feature.
    """,

    'depends': ['sale'],
    'data': [
        'views/product_template_views.xml',
        'views/sale_order_views.xml',
        'views/templates.xml',
    ],
    'demo': [
        'data/sale_demo.xml',
    ],
    'assets': {
        'web.assets_backend': [
            'sale/static/src/js/variant_mixin.js',
            'sale_product_configurator/static/src/js/product_configurator_modal.js',
            'sale_product_configurator/static/src/js/sale_product_field.js',
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


class ProductConfiguratorController(http.Controller):
    @http.route(['/sale_product_configurator/configure'], type='json', auth="user", methods=['POST'])
    def configure(self, product_template_id, pricelist_id, **kw):
        add_qty = float(kw.get('quantity', 1))
        product_template = request.env['product.template'].browse(int(product_template_id))
        pricelist = self._get_pricelist(pricelist_id)

        product_combination = False
        attribute_value_ids = set(kw.get('product_template_attribute_value_ids', []))
        attribute_value_ids |= set(kw.get('product_no_variant_attribute_value_ids', []))
        if attribute_value_ids:
            product_combination = request.env['product.template.attribute.value'].browse(
                attribute_value_ids
            ).filtered(
                lambda ptav: ptav.product_tmpl_id == product_template
            )  # Filter out ptavs not belonging to the given template
            # It happens when you change the template on an already configured line
            # receiving the configured attributes data from the previous template configuration.

        if pricelist:
            product_template = product_template.with_context(pricelist=pricelist.id, partner=request.env.user.partner_id)

        return request.env['ir.ui.view']._render_template(
            "sale_product_configurator.configure",
            {
                'product': product_template,
                'pricelist': pricelist,
                'add_qty': add_qty,
                'product_combination': product_combination
            },
        )

    @http.route(['/sale_product_configurator/show_advanced_configurator'], type='json', auth="user", methods=['POST'])
    def show_advanced_configurator(self, product_id, variant_values, pricelist_id, **kw):
        pricelist = self._get_pricelist(pricelist_id)
        return self._show_advanced_configurator(product_id, variant_values, pricelist, False, **kw)

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
        res = request.env['ir.ui.view']._render_template("sale_product_configurator.optional_product_items", {
            'product': product,
            'parent_name': product.name,
            'parent_combination': parent_combination,
            'pricelist': pricelist,
            'add_qty': add_qty,
        })
        if exclude_product_tmpl_ids:
            # Re-add the excluded products after rendering the configurator template
            request.env.cache.update(
                product,
                product._fields['optional_product_ids'],
                [optional_products.ids],
            )
        return res

    def _show_advanced_configurator(self, product_id, variant_values, pricelist, handle_stock, **kw):
        product = request.env['product.product'].browse(int(product_id))
        combination = request.env['product.template.attribute.value'].browse(variant_values)
        add_qty = float(kw.get('add_qty', 1))

        no_variant_attribute_values = combination.filtered(
            lambda product_template_attribute_value: product_template_attribute_value.attribute_id.create_variant == 'no_variant'
        )
        if no_variant_attribute_values:
            product = product.with_context(no_variant_attribute_values=no_variant_attribute_values)

        return request.env['ir.ui.view']._render_template("sale_product_configurator.optional_products_modal", {
            'product': product,
            'combination': combination,
            'add_qty': add_qty,
            'parent_name': product.name,
            'variant_values': variant_values,
            'pricelist': pricelist,
            'handle_stock': handle_stock,
            'already_configured': kw.get("already_configured", False),
            'mode': kw.get('mode', 'add'),
            'product_custom_attribute_values': kw.get('product_custom_attribute_values', None),
            'no_attribute': kw.get('no_attribute', False),
            'custom_attribute': kw.get('custom_attribute', False)
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
<odoo noupdate="1">

    <record id="product_product_1_product_template" model="product.template">
        <field name="name">Chair floor protection</field>
        <field name="categ_id" ref="product.product_category_5"/>
        <field name="list_price">12.0</field>
        <field name="weight">0.01</field>
        <field name="uom_id" ref="uom.product_uom_unit"/>
        <field name="uom_po_id" ref="uom.product_uom_unit"/>
        <field name="description_sale">Office chairs can harm your floor: protect it.</field>
        <field name="image_1920" type="base64" file="sale/static/img/floor_protection-image.jpg"/>
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

</odoo>

```

## File: models\product_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ProductTemplate(models.Model):
    _inherit = 'product.template'
    _check_company_auto = True

    optional_product_ids = fields.Many2many(
        comodel_name='product.template',
        relation='product_optional_rel',
        column1='src_id',
        column2='dest_id',
        string="Optional Products",
        help="Optional Products are suggested "
             "whenever the customer hits *Add to Cart* (cross-sell strategy, "
             "e.g. for computers: warranty, software, etc.).",
        check_company=True)

    @api.depends('attribute_line_ids.value_ids.is_custom', 'attribute_line_ids.attribute_id.create_variant')
    def _compute_has_configurable_attributes(self):
        """ A product is considered configurable if:
        - It has dynamic attributes
        - It has any attribute line with at least 2 attribute values configured
        - It has at least one custom attribute value """
        for product in self:
            product.has_configurable_attributes = (
                any(attribute.create_variant == 'dynamic' for attribute in product.attribute_line_ids.attribute_id)
                or any(len(attribute_line_id.value_ids) >= 2 for attribute_line_id in product.attribute_line_ids)
                or any(attribute_value.is_custom for attribute_value in product.attribute_line_ids.value_ids)
            )

    def get_single_product_variant(self):
        """ Method used by the product configurator to check if the product is configurable or not.

        We need to open the product configurator if the product:
        - is configurable (see has_configurable_attributes)
        - has optional products """
        res = super().get_single_product_variant()
        if res.get('product_id', False):
            has_optional_products = False
            for optional_product in self.product_variant_id.optional_product_ids:
                if optional_product.has_dynamic_attributes() or optional_product._get_possible_variants(
                    self.product_variant_id.product_template_attribute_value_ids
                ):
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
    _inherit = 'sale.order.line'

    is_configurable_product = fields.Boolean(
        string="Is the product configurable?",
        related='product_template_id.has_configurable_attributes',
        depends=['product_id'])
    product_template_attribute_value_ids = fields.Many2many(
        related='product_id.product_template_attribute_value_ids',
        depends=['product_id'])

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import product_template
from . import sale_order_line

```

## File: static\src\js\product_configurator_modal.js

```javascript
/** @odoo-module */

import ajax from 'web.ajax';
import Dialog from 'web.Dialog';
import OwlDialog from 'web.OwlDialog';
import ServicesMixin from 'web.ServicesMixin';
import VariantMixin from 'sale.VariantMixin';

export const OptionalProductsModal = Dialog.extend(ServicesMixin, VariantMixin, {
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
                // the o_sale_product_configurator_edit class is used for tours.
                classes: 'btn-primary o_sale_product_configurator_edit'
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
        this.mode = params.mode;
        this.dialogClass = 'oe_advanced_configurator_modal';
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

        var uri = this._getUri("/sale_product_configurator/show_advanced_configurator");
        var getModalContent = ajax.jsonRpc(uri, 'call', {
            mode: self.mode,
            product_id: self.rootProduct.product_id,
            variant_values: self.rootProduct.variant_values,
            product_custom_attribute_values: self.rootProduct.product_custom_attribute_values,
            pricelist_id: self.pricelistId || false,
            add_qty: self.rootProduct.quantity,
            force_dialog: self.forceDialog,
            no_attribute: self.rootProduct.no_variant_attribute_values,
            custom_attribute: self.rootProduct.product_custom_attribute_values,
            context: _.extend({'quantity': self.rootProduct.quantity}, this.context),
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

                // Notifies OwlDialog to adjust focus/active properties on owl dialogs
                OwlDialog.display(self);
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

            // Get currently displayed items to exclude them from being added again as options
            const product_tmpl_ids = new Array(...$modal.find('input.product_template_id')).map(
                (el) => parseInt(el.value)
            );
            ajax.jsonRpc(self._getUri("/sale_product_configurator/optional_product_items"), 'call', {
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

```

## File: static\src\js\sale_product_field.js

```javascript
/** @odoo-module **/

import { patch } from "@web/core/utils/patch";
import { useService } from "@web/core/utils/hooks";
import { SaleOrderLineProductField } from '@sale/js/sale_product_field';
import { OptionalProductsModal } from "@sale_product_configurator/js/product_configurator_modal";
import {
    selectOrCreateProduct,
    getSelectedVariantValues,
    getNoVariantAttributeValues,
} from "sale.VariantMixin";


patch(SaleOrderLineProductField.prototype, 'sale_product_configurator', {

    setup() {
        this._super(...arguments);

        this.rpc = useService("rpc");
        this.ui = useService("ui");
        this.orm = useService("orm");
    },

    async _onProductTemplateUpdate() {
        this._super(...arguments);
        const result = await this.orm.call(
            'product.template',
            'get_single_product_variant',
            [this.props.record.data.product_template_id[0]],
            {
                context: this.context,
            }
        );
        if(result && result.product_id) {
            if (this.props.record.data.product_id != result.product_id.id) {
                if (result.has_optional_products) {
                    this._openProductConfigurator('options');
                } else {
                    await this.props.record.update({
                        product_id: [result.product_id, result.product_name],
                    });
                    this._onProductUpdate();
                }
            }
        } else {
            if (!result.mode || result.mode === 'configurator') {
                this._openProductConfigurator('add');
            } else {
                // only triggered when sale_product_matrix is installed.
                this._openGridConfigurator(result.mode);
            }
        }
    },

    _editProductConfiguration() {
        this._super(...arguments);
        if (this.props.record.data.is_configurable_product) {
            this._openProductConfigurator('edit');
        }
    },

    get isConfigurableTemplate() {
        return this._super(...arguments) || this.props.record.data.is_configurable_product;
    },

    async _openProductConfigurator(mode) {
        const saleOrderRecord = this.props.record.model.root;
        const pricelistId = saleOrderRecord.data.pricelist_id ? saleOrderRecord.data.pricelist_id[0] : false;
        const productTemplateId = this.props.record.data.product_template_id[0];
        const $modal = $(
            await this.rpc(
                "/sale_product_configurator/configure",
                {
                    product_template_id: productTemplateId,
                    quantity: this.props.record.data.product_uom_qty || 1,
                    pricelist_id: pricelistId,
                    product_template_attribute_value_ids: this.props.record.data.product_template_attribute_value_ids.records.map(
                        record => record.data.id
                    ),
                    product_no_variant_attribute_value_ids: this.props.record.data.product_no_variant_attribute_value_ids.records.map(
                        record => record.data.id
                    ),
                    context: this.context,
                },
            )
        );
        const productSelector = `input[type="hidden"][name="product_id"], input[type="radio"][name="product_id"]:checked`;
        // TODO VFE drop this selectOrCreate and make it so that
        // get_single_product_variant returns first variant as well.
        // and use specified product on edition mode.
        const productId = await selectOrCreateProduct.call(
            this,
            $modal,
            parseInt($modal.find(productSelector).first().val(), 10),
            productTemplateId,
            false
        );
        $modal.find(productSelector).val(productId);
        const variantValues = getSelectedVariantValues($modal);
        const noVariantAttributeValues = getNoVariantAttributeValues($modal);
        /**
         *  `product_custom_attribute_value_ids` records are not loaded in the view bc sub templates
         *  are not loaded in list views. Therefore, we fetch them from the server if the record is
         *  saved. Else we use the value stored on the line.
         */
        const customAttributeValueRecords = this.props.record.data.product_custom_attribute_value_ids.records;
        let customAttributeValues = [];
        if (customAttributeValueRecords.length > 0) {
            if (customAttributeValueRecords[0].isNew) {
                customAttributeValues = customAttributeValueRecords.map(
                    record => record.data
                );
            } else {
                customAttributeValues = await this.orm.read(
                    'product.attribute.custom.value',
                    this.props.record.data.product_custom_attribute_value_ids.currentIds,
                    ["custom_product_template_attribute_value_id", "custom_value"]
                );
            }
        }
        const formattedCustomAttributeValues = customAttributeValues.map(
            data => {
                // NOTE: this dumb formatting is necessary to avoid
                // modifying the shared code between frontend & backend for now.
                return {
                    custom_value: data.custom_value,
                    custom_product_template_attribute_value_id: {
                        res_id: data.custom_product_template_attribute_value_id[0],
                    },
                };
            }
        );
        this.rootProduct = {
            product_id: productId,
            product_template_id: productTemplateId,
            quantity: parseFloat($modal.find('input[name="add_qty"]').val() || 1),
            variant_values: variantValues,
            product_custom_attribute_values: formattedCustomAttributeValues,
            no_variant_attribute_values: noVariantAttributeValues,
        };
        const optionalProductsModal = new OptionalProductsModal(null, {
            rootProduct: this.rootProduct,
            pricelistId: pricelistId,
            okButtonText: this.env._t("Confirm"),
            cancelButtonText: this.env._t("Back"),
            title: this.env._t("Configure"),
            context: this.context,
            mode: mode,
        });
        let modalEl;
        optionalProductsModal.opened(() => {
            modalEl = optionalProductsModal.el;
            this.ui.activateElement(modalEl);
        });
        optionalProductsModal.on("closed", null, async () => {
            // Wait for the event that caused the close to bubble
            await new Promise(resolve => setTimeout(resolve, 0));
            this.ui.deactivateElement(modalEl);
        });
        optionalProductsModal.open();

        let confirmed = false;
        optionalProductsModal.on("confirm", null, async () => {
            confirmed = true;
            const [
                mainProduct,
                ...optionalProducts
            ] = await optionalProductsModal.getAndCreateSelectedProducts();

            await this.props.record.update(await this._convertConfiguratorDataToUpdateData(mainProduct));
            this._onProductUpdate();
            const optionalProductLinesCreationContext = this._convertConfiguratorDataToLinesCreationContext(optionalProducts);
            for (let optionalProductLineCreationContext of optionalProductLinesCreationContext) {
                const line = await saleOrderRecord.data.order_line.addNew({
                    position: 'bottom',
                    context: optionalProductLineCreationContext,
                    mode: 'readonly',  // whatever but not edit !
                    allowWarning: true,
                });
                // FIXME: update sets the field dirty otherwise on the next edit and click out it gets deleted
                line.update({ sequence: line.data.sequence });
            }
            saleOrderRecord.data.order_line.unselectRecord();
        });
        optionalProductsModal.on("closed", null, () => {
            if (confirmed) {
                return;
            }
            if (mode != 'edit') {
                this.props.record.update({
                    product_template_id: false,
                    product_id: false,
                    product_uom_qty: 1.0,
                    // TODO reset custom/novariant values (and remove onchange logic?)
                });
            }
        });
    },

    async _convertConfiguratorDataToUpdateData(mainProduct) {
        const nameGet = await this.orm.nameGet(
            'product.product',
            [mainProduct.product_id],
            { context: this.context }
        );
        let result = {
            product_id: nameGet[0],
            product_uom_qty: mainProduct.quantity,
        };
        var customAttributeValues = mainProduct.product_custom_attribute_values;
        var customValuesCommands = [{ operation: "DELETE_ALL" }];
        if (customAttributeValues && customAttributeValues.length !== 0) {
            _.each(customAttributeValues, function (customValue) {
                customValuesCommands.push({
                    operation: "CREATE",
                    context: [
                        {
                            default_custom_product_template_attribute_value_id:
                                customValue.custom_product_template_attribute_value_id,
                            default_custom_value: customValue.custom_value,
                        },
                    ],
                });
            });
        }

        result.product_custom_attribute_value_ids = {
            operation: "MULTI",
            commands: customValuesCommands,
        };

        var noVariantAttributeValues = mainProduct.no_variant_attribute_values;
        var noVariantCommands = [{ operation: "DELETE_ALL" }];
        if (noVariantAttributeValues && noVariantAttributeValues.length !== 0) {
            var resIds = _.map(noVariantAttributeValues, function (noVariantValue) {
                return { id: parseInt(noVariantValue.value) };
            });

            noVariantCommands.push({
                operation: "ADD_M2M",
                ids: resIds,
            });
        }

        result.product_no_variant_attribute_value_ids = {
            operation: "MULTI",
            commands: noVariantCommands,
        };

        return result;
    },

    /**
     * Will map the optional producs data to sale.order.line
     * creation contexts.
     *
     * @param {Array} optionalProductsData The optional products data given by the configurator
     *
     * @private
     */
    _convertConfiguratorDataToLinesCreationContext: function (optionalProductsData) {
        return optionalProductsData.map(productData => {
            return {
                default_product_id: productData.product_id,
                default_product_template_id: productData.product_template_id,
                default_product_uom_qty: productData.quantity,
                default_product_no_variant_attribute_value_ids: productData.no_variant_attribute_values.map(
                    noVariantAttributeData => {
                        return [4, parseInt(noVariantAttributeData.value)];
                    }
                ),
                default_product_custom_attribute_value_ids: productData.product_custom_attribute_values.map(
                    customAttributeData => {
                        return [
                            0,
                            0,
                            {
                                custom_product_template_attribute_value_id:
                                    customAttributeData.custom_product_template_attribute_value_id,
                                custom_value: customAttributeData.custom_value,
                            },
                        ];
                    }
                )
            };
        });
    },
});

```

## File: views\product_template_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="product_template_view_form" model="ir.ui.view">
        <field name="name">product.template.form.inherit.sale.product.configurator</field>
        <field name="model">product.template</field>
        <field name="inherit_id" ref="product.product_template_form_view"/>
        <field name="arch" type="xml">
            <group name="upsell" position="attributes">
                <attribute name="invisible">0</attribute>
            </group>
            <group name="upsell" position="inside">
                <field name="optional_product_ids"
                    widget="many2many_tags"
                    options="{'color_field': 'color'}"
                    domain="[('id', '!=', active_id), '|', ('company_id', '=', company_id), ('company_id', '=', False)]"
                    placeholder="Recommend when 'Adding to Cart' or quotation"/>
            </group>
        </field>
    </record>

</odoo>

```

## File: views\sale_order_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

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
                <attribute name="optional">hide</attribute>
                <attribute name="string">Product Variant</attribute>
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

    <template id="product_quantity_config">
        <div class="css_quantity input-group">
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
    </template>

    <!-- backend -->
    <template id="configure" name="Configure">
        <div class="js_product main_product">

            <t t-set="combination" t-value="product_combination if product_combination else product._get_first_possible_combination()"/>
            <t t-set="combination_info" t-value="product._get_combination_info(combination, add_qty=add_qty or 1, pricelist=pricelist)"/>
            <t t-set="product_variant" t-value="product.env['product.product'].browse(combination_info['product_id'])"/>

            <input type="hidden" class="product_template_id" t-att-value="product.id"/>
            <input type="hidden" class="product_id" t-attf-name="product_id" t-att-value="product_variant.id"/>
            <input type="hidden" class="has_optional_products" t-attf-name="has_optional_products"
                   t-att-value="product_variant.optional_product_ids.filtered(lambda p: p._is_add_to_cart_possible(combination))"/>
            <div class="col-lg-12 text-center mt-5">
                <t t-if="product._is_add_to_cart_possible()">
                    <div class="col-lg-5 d-inline-block text-start">
                        <t t-if="combination" t-call="sale.variants">
                            <t t-set="parent_combination" t-value="None"/>
                        </t>
                        <h2>
                            <span t-attf-class="text-danger oe_default_price oe_striked_price {{'' if combination_info['has_discounted_price'] else 'd-none'}}"
                                t-out="combination_info['list_price']"
                                t-options='{
                                    "widget": "monetary",
                                    "display_currency": (pricelist or product).currency_id
                                }'/>
                            <span class="oe_price product_id mt-3" style="white-space: nowrap;"
                                t-att-data-product-id="product.id"
                                t-out="combination_info['price']"
                                t-options='{
                                    "widget": "monetary",
                                    "display_currency": (pricelist or product).currency_id
                                }'/>
                        </h2>
                        <t t-if="product.visible_qty_configurator">
                            <t t-call="sale_product_configurator.product_quantity_config" />
                        </t>
                        <p class="css_not_available_msg alert alert-warning">This combination does not exist.</p>
                    </div>
                    <div class="col-lg-1 d-inline-block"></div>
                    <div class="col-lg-5 d-inline-block align-top text-start">
                        <img t-if="product_variant" t-att-src="'/web/image/product.product/%s/image_1024' % product_variant.id" class="d-block product_detail_img" alt="Product Image"/>
                        <img t-else="" t-att-src="'/web/image/product.template/%s/image_1024' % product.id" class="d-block product_detail_img" alt="Product Image"/>
                    </div>
                </t>
                <t t-else="">
                    <div class="col-lg-5 d-inline-block text-start">
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
                <th class="td-img"><span class='label'>Product</span></th>
                <th><span class='label'></span></th>
                <th class="text-center td-qty"><span class='label'>Quantity</span></th>
                <th class="text-center td-price"><span class='label'>Price</span></th>
            </tr>
        </thead>
        <tbody>
            <tr class="js_product in_cart main_product">

                <t t-set="combination_info" t-value="product.product_tmpl_id._get_combination_info(combination, product.id, add_qty or 1, pricelist)"/>
                <t t-set="product_variant" t-value="product.env['product.product'].browse(combination_info['product_id'])"/>

                <input type="hidden" class="product_template_id" t-att-value="product.product_tmpl_id.id"/>
                <input type="hidden" class="product_id" t-att-value="product_variant.id"/>
                <td class='td-img'>
                    <img class="product_detail_img" t-if="product_variant" t-att-src="'/web/image/product.product/%s/image_128' % product_variant.id" alt="Product Image"/>
                    <img class="product_detail_img" t-else="" t-att-src="'/web/image/product.template/%s/image_128' % product.id" alt="Product Image"/>
                </td>
                <td class='td-product_name'>
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
                        <t t-if="product.product_tmpl_id and not combination">
                            <t t-set="combination" t-value="product.product_tmpl_id._get_first_possible_combination()"/>
                        </t>
                        <t t-if="combination and not already_configured" t-call="sale.variants">
                            <t t-set="ul_class" t-valuef="flex-column" />
                            <t t-set="product" t-value="product.product_tmpl_id"/>
                        </t>
                        <t t-else="">
                            <ul class="d-none js_add_cart_variants mb-0" t-att-data-attribute_exclusions="{'exclusions: []'}"/>
                            <div class="d-none oe_unchanged_value_ids" t-att-data-unchanged_value_ids="variant_values" ></div>
                            <!-- Keep the information to use it later (when leaving the modal window) -->
                            <div class="d-none no-attribute-info" t-att-data-attribute-value='json.dumps(no_attribute)'></div>
                            <div class="d-none custom-attribute-info" t-att-data-attribute-value='json.dumps(custom_attribute)'></div>
                        </t>
                    </div>
                </td>
                <td class="text-center td-qty">
                    <t t-call='sale_product_configurator.product_quantity_config' />
                </td>
                <td class="text-center td-price" name="price">
                    <div t-attf-class="text-danger oe_default_price oe_striked_price {{'' if combination_info['has_discounted_price'] else 'd-none'}}"
                        t-out="combination_info['list_price']"
                        t-options='{
                            "widget": "monetary",
                            "display_currency": (pricelist or product).currency_id
                        }'
                    />
                    <span class="oe_price product_id" style="white-space: nowrap;"
                        t-att-data-product-id="product.id"
                        t-out="combination_info['price']"
                        t-options='{
                            "widget": "monetary",
                            "display_currency": (pricelist or product).currency_id
                        }'/>
                    <span class="js_raw_price d-none" t-esc="combination_info['price']"/>
                    <p class="css_not_available_msg alert alert-warning">Option not available</p>
                </td>
            </tr>
            <tr class="o_total_row">
                <td colspan="4" class="text-end">
                    <strong>Total:</strong>
                    <span class="js_price_total fw-bold" style="white-space: nowrap;"
                        t-att-data-product-id="product.id"
                        t-out="combination_info['price'] * (add_qty or 1)"
                        t-options='{
                            "widget": "monetary",
                            "display_currency": (pricelist or product).currency_id
                        }'/>
                </td>
            </tr>
            <t t-if="product.optional_product_ids and mode != 'edit'">
                <tr class="o_select_options"><td colspan="4"><h4>Available Options:</h4></td></tr>
                <t t-call="sale_product_configurator.optional_product_items">
                    <t t-set="parent_combination" t-value="combination"/>
                </t>
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
                        <div class="mb-3">
                            <strong class="product-name product_display_name" t-out="combination_info['display_name']"/>
                            <div class="text-muted small" t-field="product.description_sale"/>
                        </div>
                        <t t-call="sale.variants">
                            <t t-set="combination" t-value="product._get_first_possible_combination(parent_combination)"/>
                        </t>
                    </td>
                    <td class="text-center td-qty d-none">
                        <t t-call='sale_product_configurator.product_quantity_config' />
                    </td>
                    <td class="text-center td-price">
                        <div t-attf-class="text-danger oe_default_price oe_optional oe_striked_price {{'' if combination_info['has_discounted_price'] else 'd-none'}}"
                            t-out="combination_info['list_price']"
                            t-options='{
                                "widget": "monetary",
                                "display_currency": (pricelist or product).currency_id
                            }'/>
                        <div class="oe_price" style="white-space: nowrap;"
                            t-out="combination_info['price']"
                            t-options='{
                                "widget": "monetary",
                                "display_currency": (pricelist or product).currency_id
                            }'/>
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

