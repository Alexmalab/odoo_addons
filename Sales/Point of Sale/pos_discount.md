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
        'views/pos_discount_views.xml',
        'views/pos_discount_templates.xml'
    ],
    'qweb': [
        'static/src/xml/DiscountButton.xml',
    ],
    'installable': True,
    'license': 'LGPL-3',
}

```

## File: models\pos_config.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class PosConfig(models.Model):
    _inherit = 'pos.config'

    iface_discount = fields.Boolean(string='Order Discounts', help='Allow the cashier to give discounts on the whole order.')
    discount_pc = fields.Float(string='Discount Percentage', help='The default discount percentage', default=10.0)
    discount_product_id = fields.Many2one('product.product', string='Discount Product',
        domain="[('sale_ok', '=', True)]", help='The product used to model the discount.')

    @api.onchange('company_id','module_pos_discount')
    def _default_discount_product_id(self):
        product = self.env.ref("point_of_sale.product_product_consumable", raise_if_not_found=False)
        self.discount_product_id = product if self.module_pos_discount and product and (not product.company_id or product.company_id == self.company_id) else False

    @api.model
    def _default_discount_value_on_module_install(self):
        configs = self.env['pos.config'].search([])
        open_configs = (
            self.env['pos.session']
            .search(['|', ('state', '!=', 'closed'), ('rescue', '=', True)])
            .mapped('config_id')
        )
        # Do not modify configs where an opened session exists.
        for conf in (configs - open_configs):
            conf._default_discount_product_id()

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import pos_config

```

## File: static\src\js\DiscountButton.js

```javascript
odoo.define('pos_discount.DiscountButton', function(require) {
    'use strict';

    const PosComponent = require('point_of_sale.PosComponent');
    const ProductScreen = require('point_of_sale.ProductScreen');
    const { useListener } = require('web.custom_hooks');
    const Registries = require('point_of_sale.Registries');

    class DiscountButton extends PosComponent {
        constructor() {
            super(...arguments);
            useListener('click', this.onClick);
        }
        async onClick() {
            var self = this;
            const { confirmed, payload } = await this.showPopup('NumberPopup',{
                title: this.env._t('Discount Percentage'),
                startingValue: this.env.pos.config.discount_pc,
            });
            if (confirmed) {
                const val = Math.max(0,Math.min(100,parseFloat(payload)));
                await self.apply_discount(val);
            }
        }

        async apply_discount(pc) {
            var order    = this.env.pos.get_order();
            var lines    = order.get_orderlines();
            var product  = this.env.pos.db.get_product_by_id(this.env.pos.config.discount_product_id[0]);
            if (product === undefined) {
                await this.showPopup('ErrorPopup', {
                    title : this.env._t("No discount product found"),
                    body  : this.env._t("The discount product seems misconfigured. Make sure it is flagged as 'Can be Sold' and 'Available in Point of Sale'."),
                });
                return;
            }

            // Remove existing discounts
            var i = 0;
            while ( i < lines.length ) {
                if (lines[i].get_product() === product) {
                    order.remove_orderline(lines[i]);
                } else {
                    i++;
                }
            }

            // Add discount
            // We add the price as manually set to avoid recomputation when changing customer.
            var base_to_discount = order.get_total_without_tax();
            if (product.taxes_id.length){
                var first_tax = this.env.pos.taxes_by_id[product.taxes_id[0]];
                if (first_tax.price_include) {
                    base_to_discount = order.get_total_with_tax();
                }
            }
            var discount = - pc / 100.0 * base_to_discount;

            if( discount < 0 ){
                order.add_product(product, {
                    price: discount,
                    lst_price: discount,
                    extras: {
                        price_manually_set: true,
                    },
                });
            }
        }
    }
    DiscountButton.template = 'DiscountButton';

    ProductScreen.addControlButton({
        component: DiscountButton,
        condition: function() {
            return this.env.pos.config.module_pos_discount && this.env.pos.config.discount_product_id;
        },
    });

    Registries.Component.add(DiscountButton);

    return DiscountButton;
});

```

## File: static\src\js\models.js

```javascript
odoo.define('pos_discount.models', function (require) {
  "use strict";

  var models = require('point_of_sale.models');

  var existing_models = models.PosModel.prototype.models;
  var product_index = _.findIndex(existing_models, function (model) {
      return model.model === "product.product";
  });
  var product_model = existing_models[product_index];

  models.load_models([{
    model:  product_model.model,
    fields: product_model.fields,
    order:  product_model.order,
    domain: function(self) {return [['id', '=', self.config.discount_product_id[0]]];},
    context: product_model.context,
    loaded: product_model.loaded,
  }]);

});

```

## File: static\src\xml\DiscountButton.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="DiscountButton" owl="1">
        <span class="control-button js_discount">
            <i class="fa fa-tag"></i>
            <span> </span>
            <span>Discount</span>
        </span>
    </t>

</templates>

```

## File: views\pos_discount_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <template id="assets" inherit_id="point_of_sale.assets">
          <xpath expr="." position="inside">
              <script type="text/javascript" src="/pos_discount/static/src/js/DiscountButton.js"></script>
              <script type="text/javascript" src="/pos_discount/static/src/js/models.js"></script>
          </xpath>
        </template>

</odoo>

```

## File: views\pos_discount_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="pos_config_view_form_inherit_pos_discount" model="ir.ui.view">
            <field name="name">pos.config.form.inherit.pos.discount</field>
            <field name="model">pos.config</field>
            <field name="inherit_id" ref="point_of_sale.pos_config_view_form" />
            <field name="arch" type="xml">
                <div id="btn_use_pos_discount" position="replace">
                    <div class="row mt16">
                        <label string="Discount Product" for="discount_product_id" class="col-lg-3 o_light_label"/>
                        <field name="discount_product_id" attrs="{'required':[('module_pos_discount','=',True)]}"/>
                    </div>
                    <div class="row">
                        <label string="Discount %" for="discount_pc" class="col-lg-3 o_light_label"/>
                        <field name="discount_pc"/>
                    </div>
                </div>
            </field>
        </record>

        <data noupdate="1">
            <function model="pos.config" name="_default_discount_value_on_module_install"/>
        </data>
</odoo>

```

