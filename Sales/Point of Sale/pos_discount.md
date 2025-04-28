# Odoo Module: pos_discount

Category: Sales/Point of Sale

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
    'name': 'Point of Sale Discounts',
    'version': '1.0',
    'category': 'Sales/Point of Sale',
    'sequence': 6,
    'summary': 'Simple Discounts in the Point of Sale ',
    'description': """

This module allows the cashier to quickly give percentage-based
discount to a customer.

""",
    'depends': ['point_of_sale'],
    'data': [
        'views/res_config_settings_views.xml',
        'views/pos_config_views.xml',
        ],
    'installable': True,
    'assets': {
        'point_of_sale._assets_pos': [
            'pos_discount/static/src/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: models\pos_config.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError
from odoo.osv.expression import OR


class PosConfig(models.Model):
    _inherit = 'pos.config'

    iface_discount = fields.Boolean(string='Order Discounts', help='Allow the cashier to give discounts on the whole order.')
    discount_pc = fields.Float(string='Discount Percentage', help='The default discount percentage when clicking on the Discount button', default=10.0)
    discount_product_id = fields.Many2one('product.product', string='Discount Product',
        domain="[('sale_ok', '=', True)]", help='The product used to apply the discount on the ticket.')

    @api.model
    def _default_discount_value_on_module_install(self):
        configs = self.env['pos.config'].search([])
        open_configs = (
            self.env['pos.session']
            .search(['|', ('state', '!=', 'closed'), ('rescue', '=', True)])
            .mapped('config_id')
        )
        # Do not modify configs where an opened session exists.
        product = self.env.ref("point_of_sale.product_product_consumable", raise_if_not_found=False)
        for conf in (configs - open_configs):
            conf.discount_product_id = product if conf.module_pos_discount and product and (not product.company_id or product.company_id == conf.company_id) else False

    def open_ui(self):
        for config in self:
            if not self.current_session_id and config.module_pos_discount and not config.discount_product_id:
                raise UserError(_('A discount product is needed to use the Global Discount feature. Go to Point of Sale > Configuration > Settings to set it.'))
        return super().open_ui()

    def _get_special_products(self):
        res = super()._get_special_products()
        return res | self.env['pos.config'].search([]).mapped('discount_product_id')

    def _get_available_product_domain(self):
        domain = super()._get_available_product_domain()
        return OR([domain, [('id', '=', self.discount_product_id.id)]])

```

## File: models\pos_session.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class PosSession(models.Model):
    _inherit = 'pos.session'

    def _get_pos_ui_product_product(self, params):
        result = super()._get_pos_ui_product_product(params)
        discount_product_id = self.config_id.discount_product_id.id
        product_ids_set = {product['id'] for product in result}

        if self.config_id.module_pos_discount and discount_product_id not in product_ids_set:
            productModel = self.env['product.product'].with_context(**params['context'])
            product = productModel.search_read([('id', '=', discount_product_id)], fields=params['search_params']['fields'])
            self._process_pos_ui_product_product(product)
            result.extend(product)
        return result

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    # pos.config fields
    pos_discount_pc = fields.Float(related='pos_config_id.discount_pc', readonly=False)
    pos_discount_product_id = fields.Many2one('product.product', compute='_compute_pos_discount_product_id', store=True, readonly=False)

    @api.depends('company_id', 'pos_module_pos_discount', 'pos_config_id')
    def _compute_pos_discount_product_id(self):
        default_product = self.env.ref("point_of_sale.product_product_consumable", raise_if_not_found=False) or self.env['product.product']
        for res_config in self:
            discount_product = res_config.pos_config_id.discount_product_id or default_product
            if res_config.pos_module_pos_discount and (not discount_product.company_id or discount_product.company_id == res_config.company_id):
                res_config.pos_discount_product_id = discount_product
            else:
                res_config.pos_discount_product_id = False

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import pos_config
from . import res_config_settings
from . import pos_session

```

## File: static\src\overrides\components\discount_button\discount_button.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { ProductScreen } from "@point_of_sale/app/screens/product_screen/product_screen";
import { useService } from "@web/core/utils/hooks";
import { NumberPopup } from "@point_of_sale/app/utils/input_popups/number_popup";
import { ErrorPopup } from "@point_of_sale/app/errors/popups/error_popup";
import { Component } from "@odoo/owl";
import { usePos } from "@point_of_sale/app/store/pos_hook";
import { parseFloat } from "@web/views/fields/parsers";

export class DiscountButton extends Component {
    static template = "pos_discount.DiscountButton";

    setup() {
        this.pos = usePos();
        this.popup = useService("popup");
    }
    async click() {
        var self = this;
        const { confirmed, payload } = await this.popup.add(NumberPopup, {
            title: _t("Discount Percentage"),
            startingValue: this.pos.config.discount_pc,
            isInputSelected: true,
        });
        if (confirmed) {
            const val = Math.max(0, Math.min(100, parseFloat(payload)));
            await self.apply_discount(val);
        }
    }

    async apply_discount(pc) {
        const order = this.pos.get_order();
        const lines = order.get_orderlines();
        const product = this.pos.db.get_product_by_id(this.pos.config.discount_product_id[0]);
        if (product === undefined) {
            await this.popup.add(ErrorPopup, {
                title: _t("No discount product found"),
                body: _t(
                    "The discount product seems misconfigured. Make sure it is flagged as 'Can be Sold' and 'Available in Point of Sale'."
                ),
            });
            return;
        }

        // Remove existing discounts
        lines
            .filter((line) => line.get_product() === product)
            .forEach((line) => order._unlinkOrderline(line));

        // Add one discount line per tax group
        const linesByTax = order.get_orderlines_grouped_by_tax_ids();
        for (const [tax_ids, lines] of Object.entries(linesByTax)) {
            // Note that tax_ids_array is an Array of tax_ids that apply to these lines
            // That is, the use case of products with more than one tax is supported.
            const tax_ids_array = tax_ids
                .split(",")
                .filter((id) => id !== "")
                .map((id) => Number(id));

            const baseToDiscount = order.calculate_base_amount(
                tax_ids_array,
                lines.filter((ll) => ll.isGlobalDiscountApplicable())
            );

            // We add the price as manually set to avoid recomputation when changing customer.
            const discount = (-pc / 100.0) * baseToDiscount;
            if (discount < 0) {
                order.add_product(product, {
                    price: discount,
                    lst_price: discount,
                    tax_ids: tax_ids_array,
                    merge: false,
                    description:
                        `${pc}%, ` +
                        (tax_ids_array.length
                            ? _t(
                                  "Tax: %s",
                                  tax_ids_array
                                      .map((taxId) => this.pos.taxes_by_id[taxId].amount + "%")
                                      .join(", ")
                              )
                            : _t("No tax")),
                    extras: {
                        price_type: "automatic",
                    },
                });
            }
        }
    }
}

ProductScreen.addControlButton({
    component: DiscountButton,
    condition: function () {
        const { module_pos_discount, discount_product_id } = this.pos.config;
        return module_pos_discount && discount_product_id;
    },
});

```

## File: static\src\overrides\components\discount_button\discount_button.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="pos_discount.DiscountButton">
        <span class="control-button js_discount btn btn-light rounded-0 fw-bolder" t-on-click="() => this.click()">
            <i class="fa fa-tag me-1"></i>
            <span> </span>
            <span>Discount</span>
        </span>
    </t>

</templates>

```

## File: static\src\overrides\models\models.js

```javascript
/* @odoo-modules */

import { Orderline } from "@point_of_sale/app/store/models";
import { patch } from "@web/core/utils/patch";

patch(Orderline.prototype, {
    /**
     * Checks if the current line applies for a global discount from `pos_discount.DiscountButton`.
     * @returns Boolean
     */
    isGlobalDiscountApplicable() {
        return !(
            this.pos.config.tip_product_id && this.product.id === this.pos.config.tip_product_id[0]
        );
    },
});

```

## File: views\pos_config_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="pos.config" name="_default_discount_value_on_module_install"/>
    </data>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.pos_discount</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="point_of_sale.res_config_settings_view_form" />
        <field name="arch" type="xml">
            <div id="warning_text_pos_discount" position="replace">
                <div class="row">
                    <label string="Discount Product" for="pos_discount_product_id" class="col-lg-3 o_light_label"/>
                    <field name="pos_discount_product_id" required="pos_module_pos_discount"/>
                </div>
                <div class="row">
                    <label string="Discount %" for="pos_discount_pc" class="col-lg-3 o_light_label"/>
                    <field name="pos_discount_pc"/>
                </div>
            </div>
        </field>
    </record>
</odoo>

```

