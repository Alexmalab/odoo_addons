# Odoo Module: website_sale_comparison_wishlist

Category: Website/Website

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Product Availability Notifications',
    'category': 'Website/Website',
    'summary': 'Bridge module for Website sale comparison and wishlist',
    'description': """
It allows for comparing products from the wishlist
    """,
    'depends': [
        'website_sale_comparison',
        'website_sale_wishlist',
    ],
    'data': [
        'views/templates.xml',
    ],
    'assets': {
        'web.assets_frontend': [
            'website_sale_comparison_wishlist/static/src/**/*',
        ],
    },
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: static\src\js\wishlist.js

```javascript
/** @odoo-module **/

import publicWidget from '@web/legacy/js/public/public_widget';
import '@website_sale_comparison/js/website_sale_comparison';

publicWidget.registry.ProductComparison.include({
    events: Object.assign({}, publicWidget.registry.ProductComparison.prototype.events, {
        'click .wishlist-section .o_add_to_compare': '_onClickCompare',
    }),

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {Event} ev
     */
    _onClickCompare: function (ev) {
        const productID = parseInt(ev.currentTarget.dataset.productId, 10);
        this.productComparison._addNewProducts(productID);
    },
});

```

## File: views\templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="product_wishlist" inherit_id="website_sale_wishlist.product_wishlist">
        <xpath expr="//button[hasclass('o_wish_rm')]" position="after">
            <t t-set="categories" t-value="wish.product_id.product_tmpl_id.valid_product_template_attribute_line_ids._prepare_categories_for_display()"/>
            <t t-set="product_variant_id" t-value="wish.product_id.product_tmpl_id._get_first_possible_variant_id()"/>
            <button
                t-if="is_view_active('website_sale_comparison.add_to_compare') and product_variant_id and categories"
                type="button"
                class="btn btn-link o_add_to_compare no-decoration"
                t-att-data-product-id='wish.product_id.id'>
                <small><i t-attf-class="fa fa-exchange"></i> Add to compare</small>
            </button>
        </xpath>
    </template>
</odoo>

```

