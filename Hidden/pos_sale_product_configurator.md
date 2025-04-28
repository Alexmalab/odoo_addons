# Odoo Module: pos_sale_product_configurator

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


{
    'name': 'pos_sale_product_configurator',
    'version': '1.0',
    'category': 'Hidden',
    'summary': 'Link module between point_of_sale and sale_product_configurator',
    'description': """
This module adds features depending on both modules.
""",
    'depends': ['point_of_sale', 'sale_product_configurator'],
    'installable': True,
    'auto_install': True,
    'data': [],
    'assets': {
        'point_of_sale.assets': [
            'pos_sale_product_configurator/static/src/js/models.js',
            'pos_sale_product_configurator/static/src/css/popups/product_info_popup.css',
            'pos_sale_product_configurator/static/src/xml/**/*'
        ]
    },
    'license': 'LGPL-3',
}

```

## File: models\pos_session.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class PosSession(models.Model):
    _inherit = 'pos.session'

    def _loader_params_product_product(self):
        result = super()._loader_params_product_product()
        result['search_params']['fields'].append('optional_product_ids')
        return result

```

## File: models\product.py

```python
from odoo import models


class ProductProduct(models.Model):
    _inherit = 'product.product'

    def get_product_info_pos(self, price, quantity, pos_config_id):
        res = super().get_product_info_pos(price, quantity, pos_config_id)

        # Optional products
        res['optional_products'] = [
            {'name': p.name, 'price': min(p.product_variant_ids.mapped('lst_price'))}
            for p in self.optional_product_ids.filtered_domain(self._optional_product_pos_domain())
        ]

        return res

    def has_optional_product_in_pos(self):
        self.ensure_one()
        return bool(self.optional_product_ids.filtered_domain(self._optional_product_pos_domain()))

    def _optional_product_pos_domain(self):
        return [
            '&', '&', ['sale_ok', '=', True], ['available_in_pos', '=', True],
            '|', ['company_id', '=', self.env.company], ['company_id', '=', False]
        ]

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import product
from . import pos_session

```

## File: static\src\js\models.js

```javascript
odoo.define('pos_sale_product_configurator.models', function (require) {
    "use strict";

    const { Gui } = require('point_of_sale.Gui');
    var { Order } = require('point_of_sale.models');
    const Registries = require('point_of_sale.Registries');
    const { isConnectionError } = require('point_of_sale.utils');


    const PosSaleProductConfiguratorOrder = (Order) => class PosSaleProductConfiguratorOrder extends Order {
        async add_product(product, options) {
            super.add_product(...arguments);
            if (product.optional_product_ids.length) {
                // The `optional_product_ids` only contains ids of the product templates and not the product itself
                // We don't load all the product template in the pos, so it'll be hard to know if the id comes from
                // a product available in POS. We send a quick cal to the back end to verify.
                const isProductLoaded = await this.pos.env.services.rpc(
                    {
                        model: 'product.product',
                        method: 'has_optional_product_in_pos',
                        args: [[product.id]]
                    }
                );
                if (isProductLoaded) {
                    try {
                        const quantity = this.get_selected_orderline().get_quantity();
                        const info = await this.pos.getProductInfo(product, quantity);
                        Gui.showPopup('ProductInfoPopup', { info: info , product: product });
                    } catch (e) {
                        if (isConnectionError(e)) {
                            Gui.showPopup('OfflineErrorPopup', {
                                title: this.env._t('Network Error'),
                                body: this.env._t('Cannot access product information screen if offline.'),
                            });
                        } else {
                            Gui.showPopup('ErrorPopup', {
                                title: this.env._t('Unknown error'),
                                body: this.env._t('An unknown error prevents us from loading product information.'),
                            });
                        }
                    }
                }
            }
        }
    }
    Registries.Model.extend(Order, PosSaleProductConfiguratorOrder);
})

```

## File: static\src\xml\Popups\ProductInfoPopup.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="ProductInfoPopup" t-inherit="point_of_sale.ProductInfoPopup" t-inherit-mode="extension" owl="1">
        <xpath expr="//div[hasclass('extra')]" position="inside">
            <div class="section-optional-product" t-if="productInfo.optional_products.length > 0">
                <div class="section-title">
                    <span>Optional Products</span>
                    <div class="section-title-line"/>
                </div>
                <div class="section-optional-product-body">
                    <table>
                        <t t-foreach="productInfo.optional_products" t-as="optional" t-key="optional.name">
                            <tr>
                                <td><span class="searchable" t-esc="optional.name" t-on-click="() => this.searchProduct(optional.name)"/></td>
                                <td class="table-value">
                                    from <t t-esc="env.pos.format_currency(optional.price)"/>
                                </td>
                            </tr>
                        </t>
                    </table>
                </div>
            </div>
        </xpath>
    </t>
</templates>

```

