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
    'name': 'POS - Sale Product Configurator',
    'version': '1.0',
    'category': 'Hidden',
    'summary': 'Link module between point_of_sale and sale_product_configurator',
    'description': """
This module adds features depending on both modules.
""",
    'depends': ['point_of_sale', 'sale_product_configurator'],
    'installable': True,
    'auto_install': True,
    'assets': {
        'point_of_sale._assets_pos': [
            'pos_sale_product_configurator/static/src/**/*',
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
            *self.env['product.product']._check_company_domain(self.env.company),
            ['sale_ok', '=', True],
            ['available_in_pos', '=', True],
        ]

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import product
from . import pos_session

```

## File: static\src\overrides\components\product_info_popup\product_info_popup.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_sale_product_configurator.ProductInfoPopup" t-inherit="point_of_sale.ProductInfoPopup" t-inherit-mode="extension">
        <xpath expr="//div[hasclass('extra')]" position="after">
            <div class="section-optional-product mt-3 mb-4 pb-4 border-bottom text-start" t-if="productInfo.optional_products.length > 0">
                <h3 class="section-title">Optional Products:</h3>
                <div class="section-optional-product-body" t-foreach="productInfo.optional_products" t-as="optional" t-key="optional.name">
                    <span>
                        <a class="text-primary text-decoration-underline" t-esc="optional.name" t-on-click="() => this.searchProduct(optional.name)"/>
 from
                        <t t-esc="env.utils.formatCurrency(optional.price)"/>
                    </span>
                </div>
            </div>
        </xpath>
    </t>
</templates>

```

