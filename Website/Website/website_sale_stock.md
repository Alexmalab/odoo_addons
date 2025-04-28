# Odoo Module: website_sale_stock

Category: Website/Website

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Product Availability',
    'category': 'Website/Website',
    'summary': 'Manage product inventory & availability',
    'description': """
Manage the inventory of your products and display their availability status in your eCommerce store.
In case of stockout, you can decide to block further sales or to keep selling.
A default behavior can be selected in the Website settings.
Then it can be made specific at the product level.
    """,
    'depends': [
        'website_sale',
        'sale_stock',
    ],
    'data': [
        'views/product_template_views.xml',
        'views/res_config_settings_views.xml',
        'views/website_sale_stock_templates.xml',
        'views/stock_picking_views.xml'
    ],
    'demo': [
        'data/website_sale_stock_demo.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo.addons.website_sale.controllers.main import WebsiteSale

from odoo import http,_
from odoo.http import request
from odoo.exceptions import ValidationError


class WebsiteSaleStock(WebsiteSale):
    @http.route()
    def payment_transaction(self, *args, **kwargs):
        """ Payment transaction override to double check cart quantities before
        placing the order
        """
        order = request.website.sale_get_order()
        values = []
        for line in order.order_line:
            if line.product_id.type == 'product' and line.product_id.inventory_availability in ['always', 'threshold']:
                cart_qty = sum(order.order_line.filtered(lambda p: p.product_id.id == line.product_id.id).mapped('product_uom_qty'))
                avl_qty = line.product_id.with_context(warehouse=order.warehouse_id.id).virtual_available
                if cart_qty > avl_qty:
                    values.append(_('You ask for %s products but only %s is available') % (cart_qty, avl_qty if avl_qty > 0 else 0))
        if values:
            raise ValidationError('. '.join(values) + '.')
        return super(WebsiteSaleStock, self).payment_transaction(*args, **kwargs)

```

## File: controllers\variant.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http
from odoo.addons.website_sale.controllers.variant import WebsiteSaleVariantController


class WebsiteSaleStockVariantController(WebsiteSaleVariantController):
    @http.route()
    def get_combination_info_website(self, product_template_id, product_id, combination, add_qty, **kw):
        kw['context'] = kw.get('context', {})
        kw['context'].update(website_sale_stock_get_quantity=True)
        return super(WebsiteSaleStockVariantController, self).get_combination_info_website(product_template_id, product_id, combination, add_qty, **kw)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import variant
from . import main

```

## File: data\website_sale_stock_demo.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="stock.product_cable_management_box" model="product.product">
        <field name="is_published" eval="True"/>
    </record>
    <record id="stock.product_cable_management_box_product_template" model="product.template">
        <field name="public_categ_ids" eval="[(6,0,[ref('website_sale.public_category_boxes')])]"/>
    </record>
</odoo>

```

## File: models\product_product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models
from odoo.addons.website.models import ir_http


class ProductProduct(models.Model):
    _inherit = 'product.product'

    cart_qty = fields.Integer(compute='_compute_cart_qty')

    def _compute_cart_qty(self):
        website = ir_http.get_request_website()
        if not website:
            self.cart_qty = 0
            return
        cart = website.sale_get_order()
        for product in self:
            product.cart_qty = sum(cart.order_line.filtered(lambda p: p.product_id.id == product.id).mapped('product_uom_qty')) if cart else 0

```

## File: models\product_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api


class ProductTemplate(models.Model):
    _inherit = 'product.template'

    inventory_availability = fields.Selection([
        ('never', 'Sell regardless of inventory'),
        ('always', 'Show inventory on website and prevent sales if not enough stock'),
        ('threshold', 'Show inventory below a threshold and prevent sales if not enough stock'),
        ('custom', 'Show product-specific notifications'),
    ], string='Inventory Availability', help='Adds an inventory availability status on the web product page.', default='never')
    available_threshold = fields.Float(string='Availability Threshold', default=5.0)
    custom_message = fields.Text(string='Custom Message', default='', translate=True)

    def _get_combination_info(self, combination=False, product_id=False, add_qty=1, pricelist=False, parent_combination=False, only_template=False):
        combination_info = super(ProductTemplate, self)._get_combination_info(
            combination=combination, product_id=product_id, add_qty=add_qty, pricelist=pricelist,
            parent_combination=parent_combination, only_template=only_template)

        if not self.env.context.get('website_sale_stock_get_quantity'):
            return combination_info

        if combination_info['product_id']:
            product = self.env['product.product'].sudo().browse(combination_info['product_id'])
            website = self.env['website'].get_current_website()
            virtual_available = product.with_context(warehouse=website._get_warehouse_available()).virtual_available
            combination_info.update({
                'virtual_available': virtual_available,
                'virtual_available_formatted': self.env['ir.qweb.field.float'].value_to_html(virtual_available, {'decimal_precision': 'Product Unit of Measure'}),
                'product_type': product.type,
                'inventory_availability': product.inventory_availability,
                'available_threshold': product.available_threshold,
                'custom_message': product.custom_message,
                'product_template': product.product_tmpl_id.id,
                'cart_qty': product.cart_qty,
                'uom_name': product.uom_id.name,
            })
        else:
            product_template = self.sudo()
            combination_info.update({
                'virtual_available': 0,
                'product_type': product_template.type,
                'inventory_availability': product_template.inventory_availability,
                'available_threshold': product_template.available_threshold,
                'custom_message': product_template.custom_message,
                'product_template': product_template.id,
                'cart_qty': 0
            })

        return combination_info

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    inventory_availability = fields.Selection([
        ('never', 'Sell regardless of inventory'),
        ('always', 'Show inventory on website and prevent sales if not enough stock'),
        ('threshold', 'Show inventory when below the threshold and prevent sales if not enough stock'),
        ('custom', 'Show product-specific notifications'),
    ], string='Inventory Availability', default='never')
    available_threshold = fields.Float(string='Availability Threshold')
    website_warehouse_id = fields.Many2one('stock.warehouse', related='website_id.warehouse_id', domain="[('company_id', '=', website_company_id)]", readonly=False)

    def set_values(self):
        super(ResConfigSettings, self).set_values()
        IrDefault = self.env['ir.default'].sudo()
        IrDefault.set('product.template', 'inventory_availability', self.inventory_availability)
        IrDefault.set('product.template', 'available_threshold', self.available_threshold if self.inventory_availability == 'threshold' else None)

    @api.model
    def get_values(self):
        res = super(ResConfigSettings, self).get_values()
        IrDefault = self.env['ir.default'].sudo()
        res.update(inventory_availability=IrDefault.get('product.template', 'inventory_availability') or 'never',
                   available_threshold=IrDefault.get('product.template', 'available_threshold') or 5.0)
        return res

    @api.onchange('website_company_id')
    def _onchange_website_company_id(self):
        if self.website_warehouse_id.company_id != self.website_company_id:
            return {'value': {'website_warehouse_id': False}}

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, api, fields
from odoo.tools.translate import _


class SaleOrder(models.Model):
    _inherit = 'sale.order'

    warning_stock = fields.Char('Warning')

    def _cart_update(self, product_id=None, line_id=None, add_qty=0, set_qty=0, **kwargs):
        values = super(SaleOrder, self)._cart_update(product_id, line_id, add_qty, set_qty, **kwargs)
        values = self._cart_lines_stock_update(values, **kwargs)
        return values

    def _cart_lines_stock_update(self, values, **kwargs):
        line_id = values.get('line_id')
        for line in self.order_line:
            if line.product_id.type == 'product' and line.product_id.inventory_availability in ['always', 'threshold']:
                cart_qty = sum(self.order_line.filtered(lambda p: p.product_id.id == line.product_id.id).mapped('product_uom_qty'))
                if (line_id == line.id) and cart_qty > line.product_id.with_context(warehouse=self.warehouse_id.id).virtual_available:
                    qty = line.product_id.with_context(warehouse=self.warehouse_id.id).virtual_available - cart_qty
                    new_val = super(SaleOrder, self)._cart_update(line.product_id.id, line.id, qty, 0, **kwargs)
                    values.update(new_val)

                    # Make sure line still exists, it may have been deleted in super()_cartupdate because qty can be <= 0
                    if line.exists() and new_val['quantity']:
                        line.warning_stock = _('You ask for %s products but only %s is available') % (cart_qty, new_val['quantity'])
                        values['warning'] = line.warning_stock
                    else:
                        self.warning_stock = _("Some products became unavailable and your cart has been updated. We're sorry for the inconvenience.")
                        values['warning'] = self.warning_stock
        return values

    def _website_product_id_change(self, order_id, product_id, qty=0):
        res = super(SaleOrder, self)._website_product_id_change(order_id, product_id, qty=qty)
        product = self.env['product.product'].browse(product_id)
        res['customer_lead'] = product.sale_delay
        return res

    def _get_stock_warning(self, clear=True):
        self.ensure_one()
        warn = self.warning_stock
        if clear:
            self.warning_stock = ''
        return warn


class SaleOrderLine(models.Model):
    _inherit = 'sale.order.line'

    warning_stock = fields.Char('Warning')

    def _get_stock_warning(self, clear=True):
        self.ensure_one()
        warn = self.warning_stock
        if clear:
            self.warning_stock = ''
        return warn

```

## File: models\stock_picking.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, api, fields
from odoo.tools.translate import _


class StockPicking(models.Model):
    _inherit = 'stock.picking'

    website_id = fields.Many2one('website', related='sale_id.website_id', string='Website',
                                 help='Website this picking belongs to.',
                                 store=True, readonly=True)


```

## File: models\website.py

```python
# -*- coding: utf-8 -*-
from odoo import api, fields, models


class Website(models.Model):
    _inherit = 'website'

    warehouse_id = fields.Many2one('stock.warehouse', string='Warehouse')

    def _prepare_sale_order_values(self, partner, pricelist):
        self.ensure_one()
        values = super(Website, self)._prepare_sale_order_values(partner, pricelist)
        if values['company_id']:
            warehouse_id = self._get_warehouse_available()
            if warehouse_id:
                values['warehouse_id'] = warehouse_id
        return values

    def _get_warehouse_available(self):
        return (
            self.warehouse_id and self.warehouse_id.id or
            self.env['ir.default'].get('sale.order', 'warehouse_id', company_id=self.company_id.id) or
            self.env['ir.default'].get('sale.order', 'warehouse_id') or
            self.env['stock.warehouse'].sudo().search([('company_id', '=', self.company_id.id)], limit=1).id
        )

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import website
from . import product_product
from . import product_template
from . import res_config_settings
from . import sale_order
from . import stock_picking

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#B06161"/><stop offset="45.785%" stop-color="#984E4E"/><stop offset="100%" stop-color="#7C3838"/></linearGradient><path id="d" d="M24.592 27.281l3.283 14.7L31 41.796V39h19v2.042h1.908c1.441 0 1.441 1.834.088 1.998L28.36 44l.942 3h21.801c.942 0 .942 2 0 2H27.417l-5.65-24h-1.884v1c0 .667-.313 1-.941 1-.628 0-.942-.333-.942-1v-2c.062-.667.376-1 .942-1h3.767c.486 0 .8.333.941 1l.942 3.281zM49.423 55a2.497 2.497 0 0 1-2.493-2.5c0-1.38 1.116-2.5 2.493-2.5a2.497 2.497 0 0 1 2.494 2.5c0 1.38-1.117 2.5-2.494 2.5zm-19.95 0a2.497 2.497 0 0 1-2.494-2.5c0-1.38 1.116-2.5 2.494-2.5a2.497 2.497 0 0 1 2.493 2.5c0 1.38-1.116 2.5-2.493 2.5zM31 35h19v3H31v-3zm0-5h19v4H31v-4zm0-5h19v4H31v-4zm1 1v2h17v-2H32z"/><path id="e" d="M24.592 25.281l3.283 14.7L31 39.796V37h19v2.042h1.908c1.441 0 1.441 1.834.088 1.998L28.36 42l.942 3h21.801c.942 0 .942 2 0 2H27.417l-5.65-24h-1.884v1c0 .667-.313 1-.941 1-.628 0-.942-.333-.942-1v-2c.062-.667.376-1 .942-1h3.767c.486 0 .8.333.941 1l.942 3.281zM49.423 53a2.497 2.497 0 0 1-2.493-2.5c0-1.38 1.116-2.5 2.493-2.5a2.497 2.497 0 0 1 2.494 2.5c0 1.38-1.117 2.5-2.494 2.5zm-19.95 0a2.497 2.497 0 0 1-2.494-2.5c0-1.38 1.116-2.5 2.494-2.5a2.497 2.497 0 0 1 2.493 2.5c0 1.38-1.116 2.5-2.493 2.5zM31 33h19v3H31v-3zm0-5h19v4H31v-4zm0-5h19v4H31v-4zm1 1v2h17v-2H32z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M35.869 69H4c-2 0-4-1-4-4V38.32l18.528-17.185h4.459l2.385 7.467L31 23h19v16l2.5 1.5-9.773 4.8H51V47l-.637 1.385 1.485 2.626-.388.927L35.869 69z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: static\src\js\variant_mixin.js

```javascript
odoo.define('website_sale_stock.VariantMixin', function (require) {
'use strict';

var VariantMixin = require('sale.VariantMixin');
var publicWidget = require('web.public.widget');
var ajax = require('web.ajax');
var core = require('web.core');
var QWeb = core.qweb;
var xml_load = ajax.loadXML(
    '/website_sale_stock/static/src/xml/website_sale_stock_product_availability.xml',
    QWeb
);

/**
 * Addition to the variant_mixin._onChangeCombination
 *
 * This will prevent the user from selecting a quantity that is not available in the
 * stock for that product.
 *
 * It will also display various info/warning messages regarding the select product's stock.
 *
 * This behavior is only applied for the web shop (and not on the SO form)
 * and only for the main product.
 *
 * @param {MouseEvent} ev
 * @param {$.Element} $parent
 * @param {Array} combination
 */
VariantMixin._onChangeCombinationStock = function (ev, $parent, combination) {
    var product_id = 0;
    // needed for list view of variants
    if ($parent.find('input.product_id:checked').length) {
        product_id = $parent.find('input.product_id:checked').val();
    } else {
        product_id = $parent.find('.product_id').val();
    }
    var isMainProduct = combination.product_id &&
        ($parent.is('.js_main_product') || $parent.is('.main_product')) &&
        combination.product_id === parseInt(product_id);

    if (!this.isWebsite || !isMainProduct){
        return;
    }

    var qty = $parent.find('input[name="add_qty"]').val();

    $parent.find('#add_to_cart').removeClass('out_of_stock');
    $parent.find('#buy_now').removeClass('out_of_stock');
    if (combination.product_type === 'product' && _.contains(['always', 'threshold'], combination.inventory_availability)) {
        combination.virtual_available -= parseInt(combination.cart_qty);
        if (combination.virtual_available < 0) {
            combination.virtual_available = 0;
        }
        // Handle case when manually write in input
        if (qty > combination.virtual_available) {
            var $input_add_qty = $parent.find('input[name="add_qty"]');
            qty = combination.virtual_available || 1;
            $input_add_qty.val(qty);
        }
        if (qty > combination.virtual_available
            || combination.virtual_available < 1 || qty < 1) {
            $parent.find('#add_to_cart').addClass('disabled out_of_stock');
            $parent.find('#buy_now').addClass('disabled out_of_stock');
        }
    }

    xml_load.then(function () {
        $('.oe_website_sale')
            .find('.availability_message_' + combination.product_template)
            .remove();

        var $message = $(QWeb.render(
            'website_sale_stock.product_availability',
            combination
        ));
        $('div.availability_messages').html($message);
    });
};

publicWidget.registry.WebsiteSale.include({
    /**
     * Adds the stock checking to the regular _onChangeCombination method
     * @override
     */
    _onChangeCombination: function (){
        this._super.apply(this, arguments);
        VariantMixin._onChangeCombinationStock.apply(this, arguments);
    }
});

return VariantMixin;

});

```

## File: static\src\xml\website_sale_stock_product_availability.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates>

    <t t-name="website_sale_stock.product_availability">
        <t t-if="product_type == 'product' and _.contains(['always', 'threshold'], inventory_availability)">
            <t t-if="virtual_available gt 0">
                <div t-if="inventory_availability == 'always'" t-attf-class="availability_message_#{product_template} text-success mt16">
                    <t t-esc="virtual_available_formatted" /> <t t-esc="uom_name" /> available
                </div>
                <t t-if="inventory_availability == 'threshold'">
                    <div t-if="virtual_available lte available_threshold" t-attf-class="availability_message_#{product_template} text-warning mt16">
                        <i class="fa fa-exclamation-triangle" title="Warning" role="img" aria-label="Warning"/>
                        <t t-esc="virtual_available_formatted" /> <t t-esc="uom_name" /> available
                    </div>
                    <div t-if="virtual_available gt available_threshold" t-attf-class="availability_message_#{product_template} text-success mt16">In stock</div>
                </t>
            </t>
            <div t-if="cart_qty" t-attf-class="availability_message_#{product_template} text-warning mt8">
                You already added <t t-if="!virtual_available">all</t> <t t-esc="cart_qty" /> <t t-esc="uom_name" /> in your cart.
            </div>
            <div t-if="!cart_qty and virtual_available lte 0" t-attf-class="availability_message_#{product_template} text-danger mt16"><i class="fa fa-exclamation-triangle" role="img" aria-label="Warning" title="Warning"/> Temporarily out of stock</div>
        </t>
        <div t-if="product_type == 'product' and inventory_availability == 'custom'" t-attf-class="availability_message_#{product_template} text-success mt16">
            <t t-esc="custom_message" />
        </div>
    </t>

</templates>

```

## File: views\product_template_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="product_template_form_view_inherit_website_sale_stock" model="ir.ui.view">
        <field name="name">product.template.form.inherit.website.sale.stock</field>
        <field name="model">product.template</field>
        <field name="inherit_id" ref="website_sale.product_template_form_view" />
        <field name="arch" type="xml">
            <xpath expr="//field[@name='public_categ_ids']" position="after">
                <field name="inventory_availability" string="Availability" widget="selection" attrs="{'invisible': [('type', 'in', ['service', 'consu'])]}"/>
                <field name="available_threshold" attrs="{'invisible': ['|', ('type', 'in', ['service', 'consu']), ('inventory_availability', '!=', 'threshold')], 'required': [('inventory_availability', '=', 'threshold'), ('type', '=', 'product')]}"/>
                <field name="custom_message" attrs="{'invisible': ['|', ('type', 'in', ['service', 'consu']), ('inventory_availability', '!=', 'custom')]}"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.website.sale.stock</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="website_sale.res_config_settings_view_form" />
        <field name="arch" type="xml">
            <xpath expr="//div[@id='product_availability_setting']" position="attributes">
                <attribute name="title">Manage the inventory of your products and display their availability status on the website.</attribute>
            </xpath>

            <xpath expr="//div[@name='stock_inventory_availability']" position="inside">
                <div class="content-group">
                    <div class="row mt16" attrs="{'invisible': [('group_stock_multi_warehouses', '=', False)]}">
                        <label for="website_warehouse_id" string="Warehouse" class="col-lg-3 o_light_label" />
                        <field name="website_warehouse_id"/>
                    </div>
                    <div class="row mt16" title="Default availability mode set on newly created storable products. This can be changed at the product level.">
                        <label for="inventory_availability" string="Mode" class="col-lg-3 o_light_label" />
                        <field name="inventory_availability" string="Inventory"/>
                    </div><br/>
                    <div class="row" attrs="{'invisible': [('inventory_availability', '!=', 'threshold')]}">
                        <label for="available_threshold" string="Threshold" class="col-lg-3 o_light_label" />
                        <field name="available_threshold" class="oe_inline" attrs="{'required': [('inventory_availability', '=', 'threshold')]}"/>
                    </div>
                </div>
            </xpath>
        </field>
    </record>

</odoo>


```

## File: views\stock_picking_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_picking_form_inherit_website_sale_stock" model="ir.ui.view">
        <field name="name">stock.picking.form.inherit.website.sale.stock</field>
        <field name="model">stock.picking</field>
        <field name="inherit_id" ref="stock.view_picking_form"/>
        <field name="arch" type="xml">
            <xpath expr="//page[@name='extra']/group/group/field[@name='company_id']" position="before">
                <field name="website_id" groups="website.group_multi_website" invisible="1"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\website_sale_stock_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="assets_frontend" inherit_id="website.assets_frontend" name="Website Sale Stock">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/website_sale_stock/static/src/js/variant_mixin.js" />
        </xpath>
    </template>

    <!-- Shopping Cart Lines -->
    <template id="website_sale_stock_cart_lines" inherit_id="website_sale.cart_lines" name="Shopping Cart Lines">
        <xpath expr="//input[@type='text'][hasclass('quantity')]" position="attributes">
          <attribute name='t-att-data-max'>(line.product_uom_qty + (line.product_id.virtual_available - line.product_id.cart_qty)) if line.product_id.type == 'product' and line.product_id.inventory_availability in ['always', 'threshold'] else None</attribute>
        </xpath>
        <xpath expr="//div[hasclass('css_quantity')]//i[hasclass('fa-plus')]/.." position="replace">
          <t t-if="line._get_stock_warning(clear=False)">
            <div class="input-group-append">
                <a t-attf-href="#" class="btn btn-link">
                  <i class='fa fa-warning text-warning' t-att-title="line._get_stock_warning()" role="img" aria-label="Warning"/>
                </a>
            </div>
          </t>
          <t t-else="1">
            <t>$0</t>
          </t>
        </xpath>
        <xpath expr="//div[hasclass('css_quantity')]" position="after">
            <div class='availability_messages'/>
        </xpath>
        <xpath expr="//div[hasclass('js_cart_lines')]" position="after">
          <t t-if='website_sale_order'>
            <div t-if='website_sale_order._get_stock_warning(clear=False)' class="alert alert-warning" role="alert">
              <strong>Warning!</strong> <t t-esc='website_sale_order._get_stock_warning()'/>
            </div>
          </t>
        </xpath>
    </template>

  <template id="website_sale_stock_product" inherit_id="website_sale.product" priority="4">
    <xpath expr="//a[@id='add_to_cart']" position="after">
      <div class="availability_messages o_not_editable"/>
    </xpath>
  </template>

  <template id="website_sale_stock_payment" inherit_id="website_sale.cart_summary">
     <xpath expr="//table[@id='cart_products']//td[hasclass('td-qty')]" position="inside">
      <t t-if='line._get_stock_warning(clear=False)'>
        <i class='fa fa-warning text-warning' t-att-title="line._get_stock_warning()" role="img" aria-label="Warning"/>
      </t>
    </xpath>
    <xpath expr="//table[@id='cart_products']" position="after">
        <t t-if='website_sale_order'>
          <t t-set='warning' t-value='website_sale_order._get_stock_warning(clear=False)' />
          <div t-if='warning' class="alert alert-warning" role="alert">
            <strong>Warning!</strong> <t t-esc='website_sale_order._get_stock_warning()'/>
          </div>
        </t>
    </xpath>
  </template>

</odoo>

```

