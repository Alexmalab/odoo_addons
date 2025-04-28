# Odoo Module: website_sale_coupon_delivery

Category: Website/Website

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': "Free Delivery with Coupon on eCommerce",
    'summary': """Allows to offer free shippings in coupon reward on eCommerce""",
    'description': """Allows to offer free shippings in coupon reward on eCommerce""",
    'category': 'Website/Website',
    'version': '1.0',
    'depends': ['website_sale_delivery', 'sale_coupon_delivery'],
    'data': [
        'views/website_sale_coupon_delivery_templates.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
from odoo import http
from odoo.addons.website_sale_delivery.controllers.main import WebsiteSaleDelivery
from odoo.http import request


class WebsiteSaleCouponDelivery(WebsiteSaleDelivery):

    @http.route()
    def update_eshop_carrier(self, **post):
        Monetary = request.env['ir.qweb.field.monetary']
        result = super(WebsiteSaleCouponDelivery, self).update_eshop_carrier(**post)
        order = request.website.sale_get_order()
        free_shipping_lines = None

        if order:
            order.recompute_coupon_lines()
            order.validate_taxes_on_sales_order()
            free_shipping_lines = order._get_free_shipping_lines()

        if free_shipping_lines:
            currency = order.currency_id
            amount_free_shipping = sum(free_shipping_lines.mapped('price_subtotal'))
            result.update({
                'new_amount_delivery': Monetary.value_to_html(0.0, {'display_currency': currency}),
                'new_amount_untaxed': Monetary.value_to_html(order.amount_untaxed, {'display_currency': currency}),
                'new_amount_tax': Monetary.value_to_html(order.amount_tax, {'display_currency': currency}),
                'new_amount_total': Monetary.value_to_html(order.amount_total, {'display_currency': currency}),
                'new_amount_order_discounted': Monetary.value_to_html(order.reward_amount - amount_free_shipping, {'display_currency': currency}),
            })
        return result

    @http.route()
    def cart_carrier_rate_shipment(self, carrier_id, **kw):
        Monetary = request.env['ir.qweb.field.monetary']
        order = request.website.sale_get_order(force_create=True)
        free_shipping_lines = order._get_free_shipping_lines()
        # Avoid computing carrier price delivery is free (coupon). It means if
        # the carrier has error (eg 'delivery only for Belgium') it will show
        # Free until the user clicks on it.
        if free_shipping_lines:
            return {
                'carrier_id': carrier_id,
                'status': True,
                'is_free_delivery': True,
                'new_amount_delivery': Monetary.value_to_html(0.0, {'display_currency': order.currency_id}),
                'error_message': None,
            }
        return super(WebsiteSaleCouponDelivery, self).cart_carrier_rate_shipment(carrier_id, **kw)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: static\src\js\website_sale_coupon_delivery.js

```javascript
odoo.define('website_sale_coupon_delivery.checkout', function (require) {
'use strict';

var core = require('web.core');
var publicWidget = require('web.public.widget');
require('website_sale_delivery.checkout');

var _t = core._t;

publicWidget.registry.websiteSaleDelivery.include({

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    _handleCarrierUpdateResult: function (result) {
        this._super.apply(this, arguments);
        if (result.new_amount_order_discounted) {
            // Update discount of the order
            $('#order_discounted').html(result.new_amount_order_discounted);
        }
    },
    /**
     * @override
     */
    _handleCarrierUpdateResultBadge: function (result) {
        this._super.apply(this, arguments);
        if (result.new_amount_order_discounted) {
            // We are in freeshipping, so every carrier is Free but we don't
            // want to replace error message by 'Free'
            $('#delivery_carrier .badge:not(.o_wsale_delivery_carrier_error)').text(_t('Free'));
        }
    },
});
});

```

## File: views\website_sale_coupon_delivery_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="assets_frontend" inherit_id="website.assets_frontend">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/website_sale_coupon_delivery/static/src/js/website_sale_coupon_delivery.js"></script>
        </xpath>
    </template>

</odoo>

```

