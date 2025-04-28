# Odoo Module: website_sale_loyalty_delivery

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
    'name': "Free Delivery with Coupon & Loyalty on eCommerce",
    'summary': """Allows to offer free shippings in loyalty program rewards on eCommerce""",
    'description': """Allows to offer free shippings in loyalty program rewards on eCommerce""",
    'category': 'Website/Website',
    'version': '1.0',
    'depends': ['website_sale_delivery', 'website_sale_loyalty', 'sale_loyalty_delivery'],
    'auto_install': True,
    'assets': {
        'web.assets_frontend': [
            'website_sale_loyalty_delivery/static/src/**/*',
        ],
        'web.assets_tests': [
            'website_sale_loyalty_delivery/static/tests/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
from odoo import http
from odoo.addons.payment import utils as payment_utils
from odoo.addons.website_sale_delivery.controllers.main import WebsiteSaleDelivery
from odoo.http import request


class WebsiteSaleLoyaltyDelivery(WebsiteSaleDelivery):

    @http.route()
    def update_eshop_carrier(self, **post):
        Monetary = request.env['ir.qweb.field.monetary']
        result = super().update_eshop_carrier(**post)
        order = request.website.sale_get_order()
        free_shipping_lines = None

        if order:
            order._update_programs_and_rewards()
            order.validate_taxes_on_sales_order()
            free_shipping_lines = order._get_free_shipping_lines()

        if free_shipping_lines:
            currency = order.currency_id
            if request.env.user.has_group('account.group_show_line_subtotals_tax_excluded'):
                amount_free_shipping = sum(free_shipping_lines.mapped('price_subtotal'))
            else:
                amount_free_shipping = sum(free_shipping_lines.mapped('price_total'))
            result.update({
                'new_amount_delivery_discounted': Monetary.value_to_html(order.amount_delivery + amount_free_shipping, {'display_currency': currency}),
                'new_amount_delivery_discount': Monetary.value_to_html(amount_free_shipping, {'display_currency': currency}),
                'new_amount_untaxed': Monetary.value_to_html(order.amount_untaxed, {'display_currency': currency}),
                'new_amount_tax': Monetary.value_to_html(order.amount_tax, {'display_currency': currency}),
                'new_amount_total': Monetary.value_to_html(order.amount_total, {'display_currency': currency}),
                'new_amount_order_discounted': Monetary.value_to_html(order.reward_amount - amount_free_shipping, {'display_currency': currency}),
                'new_amount_total_raw': order.amount_total,
                'delivery_discount_minor_amount': payment_utils.to_minor_currency_units(
                    amount_free_shipping, currency
                ),
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
        return super().cart_carrier_rate_shipment(carrier_id, **kw)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: static\src\js\website_sale_loyalty_delivery.js

```javascript
/** @odoo-module */

import publicWidget from 'web.public.widget';

publicWidget.registry.websiteSaleDelivery.include({
    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    _handleCarrierUpdateResult: async function (result) {
        await this._super.apply(this, arguments);
        if (result.new_amount_delivery_discount) {
            // Update discount of the order
            const cart_summary_discount_line = document.querySelector('[data-reward-type="shipping"]')
            if (cart_summary_discount_line) {
                cart_summary_discount_line.innerHTML = result.new_amount_delivery_discount;
            }
        }
        if (result.new_amount_delivery_discounted) {
            // Update discount of the order
            $('#order_delivery .monetary_field').html(result.new_amount_delivery_discounted);
        }
    },
});

```

