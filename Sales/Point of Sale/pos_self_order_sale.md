# Odoo Module: pos_self_order_sale

Category: Sales/Point Of Sale

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
{
    "name": "POS Self Order Sale",
    "category": "Sales/Point Of Sale",
    "depends": ["pos_sale", "pos_self_order"],
    "auto_install": True,
    "data": [
        "views/res_config_settings_views.xml",
        "data/kiosk_sale_team.xml",
    ],
    "assets": {
        # Assets
        "pos_self_order.assets": [
            "pos_self_order_sale/static/src/app/**/**",
        ],
    },
    "license": "LGPL-3",
}

```

## File: data\kiosk_sale_team.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="pos_sales_team" model="crm.team">
            <field name="name">Kiosk Sale Team</field>
        </record>
    </data>
</odoo>

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-

from odoo import models, api


class ResConfigSettings(models.TransientModel):
    _inherit = "res.config.settings"

    @api.onchange("pos_self_ordering_mode")
    def _onchange_pos_self_order_kiosk(self):
        super()._onchange_pos_self_order_kiosk()

        for record in self:
            if record.pos_config_id.self_ordering_mode == 'kiosk':
                if not record.pos_crm_team_id:
                    record.pos_crm_team_id = self.env.ref('pos_self_order_sale.pos_sales_team', raise_if_not_found=False)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import res_config_settings

```

## File: static\src\app\pages\cart_page\cart_page.js

```javascript
import { patch } from "@web/core/utils/patch";
import { CartPage } from "@pos_self_order/app/pages/cart_page/cart_page";
import { ProductCard } from "@pos_self_order/app/components/product_card/product_card";
import { onWillStart } from "@odoo/owl";

patch(CartPage.prototype, {
    setup() {
        super.setup(...arguments);
        onWillStart(() => {
            this.selfOrder.computeAvailableCategories();
        });
    },
    get optionalProducts() {
        const optionalProductIds = this.selfOrder.currentOrder.lines.flatMap(
            (line) => line.product_id.raw.optional_product_ids
        );

        const products = this.selfOrder.models["product.product"].filter(
            (p) => optionalProductIds.includes(p.raw.product_tmpl_id) && p.available_in_pos
        );

        return products;
    },
});
CartPage.components = { ...CartPage.components, ProductCard };

```

## File: static\src\app\pages\cart_page\cart_page.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_self_order_sale.CartPage" t-inherit="pos_self_order.CartPage" t-inherit-mode="extension">
        <xpath expr="//div[hasclass('order-content')]" position="inside">
            <div t-if="optionalProducts.length" class="upsale-content border-top bg-view mt-2 px-3 py-4 py-md-5">
                <h2 class="mb-5">Want to add something ?</h2>
                <div class="upsale-product row justify-content-between justify-content-md-start row-cols-sm-2 row-cols-md-4 row-cols-lg-4 row-cols-xl-5 row-cols-xxl-6 mb-1">
                    <t t-foreach="optionalProducts" t-as="product" t-key="product.id">
                        <ProductCard product="product" isAvailable="product.pos_categ_ids.some(categ => selfOrder.isCategoryAvailable(categ.id))"/>
                    </t>
                </div>
            </div>
        </xpath>

        <xpath expr="//div[hasclass('order-content')]" position="attributes">
            <attribute name="class" remove="pb-4" separator=" "/>
        </xpath>
    </t>
</templates>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="res_config_settings_view_form_menu" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.pos_self_order.view</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="pos_sale.res_config_settings_view_form"/>
        <field name="arch" type="xml">
           <setting id="down_payment_product" position="attributes">
                <attribute name="invisible">is_kiosk_mode</attribute>
            </setting>
        </field>
    </record>
</odoo>

```

