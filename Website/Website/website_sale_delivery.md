# Odoo Module: website_sale_delivery

Category: Website/Website

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models
from . import controllers

```

## File: __manifest__.py

```python
{
    'name': 'eCommerce Delivery',
    'category': 'Website/Website',
    'summary': 'Add delivery costs to online sales',
    'version': '1.0',
    'description': """
Add a selection of delivery methods to your eCommerce store.
Configure your own methods with a pricing grid or integrate with carriers for a fully automated shipping process.
    """,
    'depends': ['website_sale', 'delivery', 'website_sale_stock'],
    'data': [
        'views/website_sale_delivery_templates.xml',
        'views/website_sale_delivery_views.xml',
        'views/res_config_settings_views.xml',
        'data/website_sale_delivery_data.xml'
    ],
    'demo': [
        'data/website_sale_delivery_demo.xml'
    ],
    'qweb': [],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http, _
from odoo.http import request
from odoo.addons.website_sale.controllers.main import WebsiteSale
from odoo.exceptions import UserError


class WebsiteSaleDelivery(WebsiteSale):

    @http.route(['/shop/payment'], type='http', auth="public", website=True)
    def payment(self, **post):
        order = request.website.sale_get_order()
        carrier_id = post.get('carrier_id')
        if carrier_id:
            carrier_id = int(carrier_id)
        if order:
            order._check_carrier_quotation(force_carrier_id=carrier_id)
            if carrier_id:
                return request.redirect("/shop/payment")

        return super(WebsiteSaleDelivery, self).payment(**post)

    @http.route(['/shop/update_carrier'], type='json', auth='public', methods=['POST'], website=True, csrf=False)
    def update_eshop_carrier(self, **post):
        order = request.website.sale_get_order()
        carrier_id = int(post['carrier_id'])
        if order:
            order._check_carrier_quotation(force_carrier_id=carrier_id)
        return self._update_website_sale_delivery_return(order, **post)

    @http.route(['/shop/carrier_rate_shipment'], type='json', auth='public', methods=['POST'], website=True)
    def cart_carrier_rate_shipment(self, carrier_id, **kw):
        order = request.website.sale_get_order(force_create=True)

        if not int(carrier_id) in order._get_delivery_methods().ids:
            raise UserError(_('It seems that a delivery method is not compatible with your address. Please refresh the page and try again.'))

        Monetary = request.env['ir.qweb.field.monetary']

        res = {'carrier_id': carrier_id}
        carrier = request.env['delivery.carrier'].sudo().browse(int(carrier_id))
        rate = carrier.rate_shipment(order)
        if rate.get('success'):
            tax_ids = carrier.product_id.taxes_id.filtered(lambda t: t.company_id == order.company_id)
            if tax_ids:
                fpos = order.fiscal_position_id
                tax_ids = fpos.map_tax(tax_ids, carrier.product_id, order.partner_shipping_id)
                taxes = tax_ids.compute_all(
                    rate['price'],
                    currency=order.currency_id,
                    quantity=1.0,
                    product=carrier.product_id,
                    partner=order.partner_shipping_id,
                )
                if request.env.user.has_group('account.group_show_line_subtotals_tax_excluded'):
                    rate['price'] = taxes['total_excluded']
                else:
                    rate['price'] = taxes['total_included']

            res['status'] = True
            res['new_amount_delivery'] = Monetary.value_to_html(rate['price'], {'display_currency': order.currency_id})
            res['is_free_delivery'] = not bool(rate['price'])
            res['error_message'] = rate['warning_message']
        else:
            res['status'] = False
            res['new_amount_delivery'] = Monetary.value_to_html(0.0, {'display_currency': order.currency_id})
            res['error_message'] = rate['error_message']
        return res

    def order_lines_2_google_api(self, order_lines):
        """ Transforms a list of order lines into a dict for google analytics """
        order_lines_not_delivery = order_lines.filtered(lambda line: not line.is_delivery)
        return super(WebsiteSaleDelivery, self).order_lines_2_google_api(order_lines_not_delivery)

    def order_2_return_dict(self, order):
        """ Returns the tracking_cart dict of the order for Google analytics """
        ret = super(WebsiteSaleDelivery, self).order_2_return_dict(order)
        for line in order.order_line:
            if line.is_delivery:
                ret['transaction']['shipping'] = line.price_unit
        return ret

    def _get_shop_payment_values(self, order, **kwargs):
        values = super(WebsiteSaleDelivery, self)._get_shop_payment_values(order, **kwargs)
        has_storable_products = any(line.product_id.type in ['consu', 'product'] for line in order.order_line)

        if not order._get_delivery_methods() and has_storable_products:
            values['errors'].append(
                (_('Sorry, we are unable to ship your order'),
                 _('No shipping method is available for your current order and shipping address. '
                   'Please contact us for more information.')))

        if has_storable_products:
            if order.carrier_id and not order.delivery_rating_success:
                order._remove_delivery_line()

            delivery_carriers = order._get_delivery_methods()
            values['deliveries'] = delivery_carriers.sudo()

        values['delivery_has_storable'] = has_storable_products
        values['delivery_action_id'] = request.env.ref('delivery.action_delivery_carrier_form').id
        return values

    def _update_website_sale_delivery_return(self, order, **post):
        Monetary = request.env['ir.qweb.field.monetary']
        carrier_id = int(post['carrier_id'])
        currency = order.currency_id
        if order:
            return {
                'status': order.delivery_rating_success,
                'error_message': order.delivery_message,
                'carrier_id': carrier_id,
                'is_free_delivery': not bool(order.amount_delivery),
                'new_amount_delivery': Monetary.value_to_html(order.amount_delivery, {'display_currency': currency}),
                'new_amount_untaxed': Monetary.value_to_html(order.amount_untaxed, {'display_currency': currency}),
                'new_amount_tax': Monetary.value_to_html(order.amount_tax, {'display_currency': currency}),
                'new_amount_total': Monetary.value_to_html(order.amount_total, {'display_currency': currency}),
            }
        return {}

```

## File: controllers\__init__.py

```python
from . import main

```

## File: data\website_sale_delivery_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="delivery.free_delivery_carrier" model="delivery.carrier">
        <field name="is_published" eval="True"/>
    </record>
</odoo>

```

## File: data\website_sale_delivery_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <record id="delivery.delivery_carrier" model="delivery.carrier">
            <field name="is_published" eval="False" />
        </record>

        <record id="delivery.normal_delivery_carrier" model="delivery.carrier">
            <field name="is_published" eval="False" />
        </record>

    </data>
</odoo>

```

## File: models\delivery.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class DeliveryCarrier(models.Model):
    _name = 'delivery.carrier'
    _inherit = ['delivery.carrier', 'website.published.multi.mixin']

    website_description = fields.Text(related='product_id.description_sale', string='Description for Online Quotations', readonly=False)

```

## File: models\res_country.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResCountry(models.Model):
    _inherit = 'res.country'

    def get_website_sale_countries(self, mode='billing'):
        res = super(ResCountry, self).get_website_sale_countries(mode=mode)
        if mode == 'shipping':
            countries = self.env['res.country']

            delivery_carriers = self.env['delivery.carrier'].sudo().search([('website_published', '=', True)])
            for carrier in delivery_carriers:
                if not carrier.country_ids and not carrier.state_ids:
                    countries = res
                    break
                countries |= carrier.country_ids

            res = res & countries
        return res

    def get_website_sale_states(self, mode='billing'):
        res = super(ResCountry, self).get_website_sale_states(mode=mode)

        states = self.env['res.country.state']
        if mode == 'shipping':
            dom = ['|', ('country_ids', 'in', self.id), ('country_ids', '=', False), ('website_published', '=', True)]
            delivery_carriers = self.env['delivery.carrier'].sudo().search(dom)

            for carrier in delivery_carriers:
                if not carrier.country_ids or not carrier.state_ids:
                    states = res
                    break
                states |= carrier.state_ids
            if not states:
                states = states.search([('country_id', '=', self.id)])
            res = res & states
        return res

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import logging

from odoo import api, fields, models

_logger = logging.getLogger(__name__)


class SaleOrder(models.Model):
    _inherit = 'sale.order'

    amount_delivery = fields.Monetary(
        compute='_compute_amount_delivery',
        string='Delivery Amount',
        help="The amount without tax.", store=True, tracking=True)

    def _compute_website_order_line(self):
        super(SaleOrder, self)._compute_website_order_line()
        for order in self:
            order.website_order_line = order.website_order_line.filtered(lambda l: not l.is_delivery)

    @api.depends('order_line.price_unit', 'order_line.tax_id', 'order_line.discount', 'order_line.product_uom_qty')
    def _compute_amount_delivery(self):
        for order in self:
            if self.env.user.has_group('account.group_show_line_subtotals_tax_excluded'):
                order.amount_delivery = sum(order.order_line.filtered('is_delivery').mapped('price_subtotal'))
            else:
                order.amount_delivery = sum(order.order_line.filtered('is_delivery').mapped('price_total'))

    def _check_carrier_quotation(self, force_carrier_id=None):
        self.ensure_one()
        DeliveryCarrier = self.env['delivery.carrier']

        if self.only_services:
            self.write({'carrier_id': None})
            self._remove_delivery_line()
            return True
        else:
            # attempt to use partner's preferred carrier
            if not force_carrier_id and self.partner_shipping_id.property_delivery_carrier_id:
                force_carrier_id = self.partner_shipping_id.property_delivery_carrier_id.id

            carrier = force_carrier_id and DeliveryCarrier.browse(force_carrier_id) or self.carrier_id
            available_carriers = self._get_delivery_methods()
            if carrier:
                if carrier not in available_carriers:
                    carrier = DeliveryCarrier
                else:
                    # set the forced carrier at the beginning of the list to be verfied first below
                    available_carriers -= carrier
                    available_carriers = carrier + available_carriers
            if force_carrier_id or not carrier or carrier not in available_carriers:
                for delivery in available_carriers:
                    verified_carrier = delivery._match_address(self.partner_shipping_id)
                    if verified_carrier:
                        carrier = delivery
                        break
                self.write({'carrier_id': carrier.id})
            self._remove_delivery_line()
            if carrier:
                res = carrier.rate_shipment(self)
                if res.get('success'):
                    self.set_delivery_line(carrier, res['price'])
                    self.delivery_rating_success = True
                    self.delivery_message = res['warning_message']
                else:
                    self.set_delivery_line(carrier, 0.0)
                    self.delivery_rating_success = False
                    self.delivery_message = res['error_message']

        return bool(carrier)

    def _get_delivery_methods(self):
        address = self.partner_shipping_id
        # searching on website_published will also search for available website (_search method on computed field)
        return self.env['delivery.carrier'].sudo().search([('website_published', '=', True)]).available_carriers(address)

    def _cart_update(self, product_id=None, line_id=None, add_qty=0, set_qty=0, **kwargs):
        """ Override to update carrier quotation if quantity changed """

        self._remove_delivery_line()

        # When you update a cart, it is not enouf to remove the "delivery cost" line
        # The carrier might also be invalid, eg: if you bought things that are too heavy
        # -> this may cause a bug if you go to the checkout screen, choose a carrier,
        #    then update your cart (the cart becomes uneditable)
        self.write({'carrier_id': False})

        values = super(SaleOrder, self)._cart_update(product_id, line_id, add_qty, set_qty, **kwargs)

        return values

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_country
from . import delivery
from . import sale_order

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#7CC098"/><stop offset="100%" stop-color="#5F8A71"/></linearGradient></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M44.5 69H4c-2 0-4-1-4-4V36.525L21 16h9l4-4h3l4 5h9l4 11 1 26-10.5 15z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><path fill="#000" d="M16 51.482V29.488L20.875 18h9.147a5.5 5.5 0 0 1 10.956 0h9.147L55 29.488V56H16v-4.023l15.79-5.185.216 1.967 1.37-.61.642-2.245 5.024-2.236.628 7.555 1.826-.813 1.656-8.572 5.023-2.237c.758-.337 1.126-1.157.824-1.836-.302-.678-1.158-.953-1.916-.616l-5.023 2.237-7.478-4.506-1.827.813 5.195 5.522-5.023 2.237-2.098-1.025-1.37.61 1.466 1.645L16 51.482zM30.207 20h-8.951L18 29h35l-4.07-9h-8.137a5.502 5.502 0 0 1-10.586 0zm3.934-.08H33a2.49 2.49 0 0 0 .165.99c.116.287.281.528.495.725.213.197.47.348.77.455.3.107.63.163.99.17v.78h.43v-.78c.333-.013.643-.068.93-.165a2.23 2.23 0 0 0 .75-.42 1.93 1.93 0 0 0 .505-.69c.123-.277.185-.602.185-.975 0-.36-.067-.663-.2-.91a1.904 1.904 0 0 0-.53-.62 2.957 2.957 0 0 0-.76-.41c-.287-.107-.58-.2-.88-.28v-2.11c.36 0 .621.09.785.27.163.18.251.44.265.78h1.14a1.99 1.99 0 0 0-.175-.86 1.721 1.721 0 0 0-.47-.61 1.996 1.996 0 0 0-.695-.36c-.267-.08-.55-.12-.85-.12V14h-.43v.78c-.3 0-.587.043-.86.13a2.302 2.302 0 0 0-.725.375c-.21.163-.377.367-.5.61a1.845 1.845 0 0 0-.185.845c0 .367.066.673.2.92.133.247.308.452.525.615.216.163.46.297.73.4.27.103.541.192.815.265v2.42c-.46-.013-.79-.147-.99-.4-.2-.253-.297-.6-.29-1.04zM35.13 16v2a3.704 3.704 0 0 1-.415-.125 1.33 1.33 0 0 1-.36-.195 1.013 1.013 0 0 1-.255-.29.795.795 0 0 1-.1-.41c0-.353.098-.605.295-.755.197-.15.475-.225.835-.225zm.87 5.33V19c.153.04.303.087.45.14.147.053.278.123.395.21.117.087.21.193.28.32a.96.96 0 0 1 .105.47c0 .4-.115.692-.345.875-.23.183-.525.288-.885.315z" opacity=".3"/><path fill="#FFF" d="M16 49.482V27.488L20.875 16h9.147a5.5 5.5 0 0 1 10.956 0h9.147L55 27.488V54H16v-4.023l15.79-5.185.216 1.967 1.37-.61.642-2.245 5.024-2.236.628 7.555 1.826-.813 1.656-8.572 5.023-2.237c.758-.337 1.126-1.157.824-1.836-.302-.678-1.158-.953-1.916-.616l-5.023 2.237-7.478-4.506-1.827.813 5.195 5.522-5.023 2.237-2.098-1.025-1.37.61 1.466 1.645L16 49.482zM30.207 18h-8.951L18 27h35l-4.07-9h-8.137a5.502 5.502 0 0 1-10.586 0zm3.934-.08H33a2.49 2.49 0 0 0 .165.99c.116.287.281.528.495.725.213.197.47.348.77.455.3.107.63.163.99.17v.78h.43v-.78c.333-.013.643-.068.93-.165a2.23 2.23 0 0 0 .75-.42 1.93 1.93 0 0 0 .505-.69c.123-.277.185-.602.185-.975 0-.36-.067-.663-.2-.91a1.904 1.904 0 0 0-.53-.62 2.957 2.957 0 0 0-.76-.41c-.287-.107-.58-.2-.88-.28v-2.11c.36 0 .621.09.785.27.163.18.251.44.265.78h1.14a1.99 1.99 0 0 0-.175-.86 1.721 1.721 0 0 0-.47-.61 1.996 1.996 0 0 0-.695-.36c-.267-.08-.55-.12-.85-.12V12h-.43v.78c-.3 0-.587.043-.86.13a2.302 2.302 0 0 0-.725.375c-.21.163-.377.367-.5.61a1.845 1.845 0 0 0-.185.845c0 .367.066.673.2.92.133.247.308.452.525.615.216.163.46.297.73.4.27.103.541.192.815.265v2.42c-.46-.013-.79-.147-.99-.4-.2-.253-.297-.6-.29-1.04zM35.13 14v2a3.704 3.704 0 0 1-.415-.125 1.33 1.33 0 0 1-.36-.195 1.013 1.013 0 0 1-.255-.29.795.795 0 0 1-.1-.41c0-.353.098-.605.295-.755.197-.15.475-.225.835-.225zm.87 5.33V17c.153.04.303.087.45.14.147.053.278.123.395.21.117.087.21.193.28.32a.96.96 0 0 1 .105.47c0 .4-.115.692-.345.875-.23.183-.525.288-.885.315z"/></g></g></svg>
```

## File: static\src\js\website_sale_delivery.js

```javascript
odoo.define('website_sale_delivery.checkout', function (require) {
'use strict';

var core = require('web.core');
var publicWidget = require('web.public.widget');

var _t = core._t;
var concurrency = require('web.concurrency');
var dp = new concurrency.DropPrevious();

publicWidget.registry.websiteSaleDelivery = publicWidget.Widget.extend({
    selector: '.oe_website_sale',
    events: {
        'change select[name="shipping_id"]': '_onSetAddress',
        'click #delivery_carrier .o_delivery_carrier_select': '_onCarrierClick',
    },

    /**
     * @override
     */
    start: function () {
        var self = this;
        var $carriers = $('#delivery_carrier input[name="delivery_type"]');
        var $payButton = $('#o_payment_form_pay');
        // Workaround to:
        // - update the amount/error on the label at first rendering
        // - prevent clicking on 'Pay Now' if the shipper rating fails
        if ($carriers.length > 0) {
            if ($carriers.filter(':checked').length === 0) {
                $payButton.prop('disabled', true);
                var disabledReasons = $payButton.data('disabled_reasons') || {};
                disabledReasons.carrier_selection = true;
                $payButton.data('disabled_reasons', disabledReasons);
            }
            $carriers.filter(':checked').click();
        }

        // Asynchronously retrieve every carrier price
        _.each($carriers, function (carrierInput, k) {
            self._showLoading($(carrierInput));
            self._rpc({
                route: '/shop/carrier_rate_shipment',
                params: {
                    'carrier_id': carrierInput.value,
                },
            }).then(self._handleCarrierUpdateResultBadge.bind(self));
        });

        return this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {jQuery} $carrierInput
     */
    _showLoading: function ($carrierInput) {
        $carrierInput.siblings('.o_wsale_delivery_badge_price').html('<span class="fa fa-spinner fa-spin"/>');
    },
    /**
     * @private
     * @param {Object} result
     */
    _handleCarrierUpdateResult: function (result) {
        this._handleCarrierUpdateResultBadge(result);
        var $payButton = $('#o_payment_form_pay');
        var $amountDelivery = $('#order_delivery .monetary_field');
        var $amountUntaxed = $('#order_total_untaxed .monetary_field');
        var $amountTax = $('#order_total_taxes .monetary_field');
        var $amountTotal = $('#order_total .monetary_field, #amount_total_summary.monetary_field');

        if (result.status === true) {
            $amountDelivery.html(result.new_amount_delivery);
            $amountUntaxed.html(result.new_amount_untaxed);
            $amountTax.html(result.new_amount_tax);
            $amountTotal.html(result.new_amount_total);
            var disabledReasons = $payButton.data('disabled_reasons') || {};
            disabledReasons.carrier_selection = false;
            $payButton.data('disabled_reasons', disabledReasons);
            $payButton.prop('disabled', _.contains($payButton.data('disabled_reasons'), true));
        } else {
            $amountDelivery.html(result.new_amount_delivery);
            $amountUntaxed.html(result.new_amount_untaxed);
            $amountTax.html(result.new_amount_tax);
            $amountTotal.html(result.new_amount_total);
        }
    },
    /**
     * @private
     * @param {Object} result
     */
    _handleCarrierUpdateResultBadge: function (result) {
        var $carrierBadge = $('#delivery_carrier input[name="delivery_type"][value=' + result.carrier_id + '] ~ .o_wsale_delivery_badge_price');

        if (result.status === true) {
             // if free delivery (`free_over` field), show 'Free', not '$0'
             if (result.is_free_delivery) {
                 $carrierBadge.text(_t('Free'));
             } else {
                 $carrierBadge.html(result.new_amount_delivery);
             }
             $carrierBadge.removeClass('o_wsale_delivery_carrier_error');
        } else {
            $carrierBadge.addClass('o_wsale_delivery_carrier_error');
            $carrierBadge.text(result.error_message);
        }
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {Event} ev
     */
    _onCarrierClick: function (ev) {
        var $radio = $(ev.currentTarget).find('input[type="radio"]');
        this._showLoading($radio);
        $radio.prop("checked", true);
        var $payButton = $('#o_payment_form_pay');
        $payButton.prop('disabled', true);
        var disabledReasons = $payButton.data('disabled_reasons') || {};
        disabledReasons.carrier_selection = true;
        $payButton.data('disabled_reasons', disabledReasons);
        dp.add(this._rpc({
            route: '/shop/update_carrier',
            params: {
                carrier_id: $radio.val(),
            },
        })).then(this._handleCarrierUpdateResult.bind(this));
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onSetAddress: function (ev) {
        var value = $(ev.currentTarget).val();
        var $providerFree = $('select[name="country_id"]:not(.o_provider_restricted), select[name="state_id"]:not(.o_provider_restricted)');
        var $providerRestricted = $('select[name="country_id"].o_provider_restricted, select[name="state_id"].o_provider_restricted');
        if (value === 0) {
            // Ship to the same address : only show shipping countries available for billing
            $providerFree.hide().attr('disabled', true);
            $providerRestricted.show().attr('disabled', false).change();
        } else {
            // Create a new address : show all countries available for billing
            $providerFree.show().attr('disabled', false).change();
            $providerRestricted.hide().attr('disabled', true);
        }
    },
});
});

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.website.sale.delivery</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="website_sale.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <div id="msg_delivery_method_setting" position="after">
                <div class="content-group" attrs="{'invisible': [('module_website_sale_delivery', '=', False)]}">
                    <div class="mt16">
                        <button type="action" name="%(delivery.action_delivery_carrier_form)d" string="Shipping Methods" class="btn-link" icon="fa-arrow-right"/>
                    </div>
                </div>
            </div>
            <div id="website_delivery_dhl" position="after">
                <div class="content-group">
                    <div class="mt8" attrs="{'invisible': [('module_delivery_dhl', '=', False)]}">
                        <button name="%(delivery.action_delivery_carrier_form)d" icon="fa-arrow-right" type="action" string="DHL Shipping Methods" class="btn-link" context="{'search_default_delivery_type': 'dhl'}"/>
                    </div>
                </div>
            </div>
            <div id="website_delivery_fedex" position="after">
                <div class="content-group">
                    <div class="mt8" attrs="{'invisible': [('module_delivery_fedex', '=', False)]}">
                        <button name="%(delivery.action_delivery_carrier_form)d" icon="fa-arrow-right" type="action" string="FedEx Shipping Methods" class="btn-link" context="{'search_default_delivery_type': 'fedex'}"/>
                    </div>
                </div>
            </div>
            <div id="website_delivery_usps" position="after">
                <div class="content-group">
                    <div class="mt8" attrs="{'invisible': [('module_delivery_usps', '=', False)]}">
                        <button name="%(delivery.action_delivery_carrier_form)d" icon="fa-arrow-right" type="action" string="USPS Shipping Methods" class="btn-link" context="{'search_default_delivery_type': 'usps'}"/>
                    </div>
                </div>
            </div>
            <div id="website_delivery_bpost" position="after">
                <div class="content-group">
                    <div class="mt8" attrs="{'invisible': [('module_delivery_bpost', '=', False)]}">
                        <button name="%(delivery.action_delivery_carrier_form)d" icon="fa-arrow-right" type="action" string="bpost Shipping Methods" class="btn-link" context="{'search_default_delivery_type': 'bpost'}"/>
                    </div>
                </div>
            </div>
            <div id="website_delivery_easypost" position="after">
                <div class="content-group">
                    <div class="mt8" attrs="{'invisible': [('module_delivery_easypost', '=', False)]}">
                        <button name="%(delivery.action_delivery_carrier_form)d" icon="fa-arrow-right" type="action" string="Easypost Shipping Methods" class="btn-link" context="{'search_default_delivery_type': 'easypost'}"/>
                    </div>
                </div>
            </div>
        </field>
    </record>
</odoo>

```

## File: views\website_sale_delivery_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="cart_delivery" name="Delivery Costs" inherit_id="website_sale.total">
        <xpath expr="//tr[@id='order_total_untaxed']" position="before">
            <tr id="order_delivery" t-if="website_sale_order and website_sale_order.carrier_id">
              <td class="text-right border-0 text-muted"  title="Delivery will be updated after choosing a new delivery method">Delivery:</td>
              <td class="text-xl-right border-0 text-muted" >
                   <span t-field="website_sale_order.amount_delivery" class="monetary_field" style="white-space: nowrap;" t-options='{
                      "widget": "monetary",
                      "display_currency": website_sale_order.currency_id,
                  }'/>
              </td>
            </tr>
        </xpath>
    </template>

    <template id="assets_frontend" inherit_id="website.assets_frontend" name="Shop">
      <xpath expr="." position="inside">
        <script type="text/javascript" src="/website_sale_delivery/static/src/js/website_sale_delivery.js"></script>
        <link rel="stylesheet" type="text/scss" href="/website_sale_delivery/static/src/scss/website_sale_delivery.scss"/>
      </xpath>
    </template>

    <template id="assets_tests" name="Website Sale Delivery Assets Tests" inherit_id="web.assets_tests">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/website_sale_delivery/static/tests/tours/website_free_delivery.js"></script>
        </xpath>
    </template>

    <template id="payment_delivery_methods">
        <input t-att-value="delivery.id" t-att-id="'delivery_%i' % delivery.id" type="radio" name="delivery_type" t-att-checked="order.carrier_id and order.carrier_id.id == delivery.id and 'checked' or False" t-att-class="'d-none' if delivery_nb == 1 else ''"/>
        <label class="label-optional" t-field="delivery.name"/>
        <t t-set='badge_class' t-value="(delivery_nb != 1 and 'float-right ' or '') + 'badge badge-secondary'" />
        <span t-attf-class="#{badge_class} o_wsale_delivery_badge_price">Select to compute delivery rate</span>
        <t t-if="delivery.website_description">
            <div t-field="delivery.website_description" class="text-muted mt8"/>
        </t>
    </template>

    <template id="payment_delivery" name="Delivery Costs" inherit_id="website_sale.payment">
        <xpath expr="//div[@id='shipping_and_billing']" position="inside">
            <t t-set="delivery_nb" t-value="deliveries and len(deliveries)"/>
            <div t-if="delivery_nb == 1" id="delivery_carrier" class="mt4">
                <b>Shipping Method: </b>
                <t t-foreach="deliveries" t-as="delivery">
                    <t t-call="website_sale_delivery.payment_delivery_methods"/>
                </t>
            </div>
        </xpath>
        <xpath expr="//div[@id='payment_method']" position="before">
            <div t-if="deliveries" id="delivery_carrier">
                <t t-set="delivery_nb" t-value="len(deliveries)"/>
                <h3 t-if="delivery_nb &gt; 1" class="mb24">Choose a delivery method</h3>
                <div t-if="delivery_nb &gt; 1" class="card border-0" id="delivery_method">
                    <ul class="list-group">
                    <t t-foreach="deliveries" t-as="delivery">
                        <li class="list-group-item o_delivery_carrier_select">
                            <t t-call="website_sale_delivery.payment_delivery_methods"/>
                        </li>
                    </t>
                    </ul>
                </div>
            </div>
        </xpath>
        <!-- we shouldn't be able to pay if there is no way to deliver -->
        <xpath expr="//div[@id='payment_method']" position="attributes">
                <attribute name="t-att-style">'display: none!important' if not deliveries and delivery_has_storable else ''</attribute>
        </xpath>
    </template>

</odoo>

```

## File: views\website_sale_delivery_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_delivery_carrier_form_website_delivery" model="ir.ui.view">
        <field name="name">delivery.carrier.website.form</field>
        <field name="model">delivery.carrier</field>
        <field name="inherit_id" ref="delivery.view_delivery_carrier_form"/>
        <field name="arch" type="xml">
            <field name="company_id" position='after'>
                <field name="website_id" groups="website.group_multi_website" options="{'no_open': True, 'no_create_edit': True}"/>
            </field>
            <xpath expr="//page[@name='destination']" position='after'>
                <page string="Description">
                    <field name="website_description"  placeholder="Description displayed on the eCommerce and on online quotations."/>
                </page>
            </xpath>
            <xpath expr="//button[@name='toggle_prod_environment']" position='before'>
                <button name="website_publish_button" type="object" class="oe_stat_button" icon="fa-globe">
                    <field name="is_published" widget="website_publish_button"/>
                </button>
            </xpath>
        </field>
    </record>

    <record id="view_delivery_carrier_tree_inherit_website_sale_delivery" model="ir.ui.view">
        <field name="name">delivery.carrier.tree.inherit</field>
        <field name="model">delivery.carrier</field>
        <field name="inherit_id" ref="delivery.view_delivery_carrier_tree"/>
        <field name="arch" type="xml">
            <field name="delivery_type" position="after">
                <field name="is_published"/>
                <field name="website_id" groups="website.group_multi_website"/>
            </field>
        </field>
    </record>

    <record id="view_delivery_carrier_search_inherit_website_sale_delivery" model="ir.ui.view">
        <field name="name">delivery.carrier.search.inherit</field>
        <field name="model">delivery.carrier</field>
        <field name="inherit_id" ref="delivery.view_delivery_carrier_search"/>
        <field name="arch" type="xml">
            <filter name="inactive" position="after">
                <filter string="Published" name="is_published" domain="[('is_published','=',True)]"/>
            </filter>
        </field>
    </record>

    <menuitem
        id="menu_ecommerce_delivery"
        parent="website_sale.menu_ecommerce_settings"
        name="Shipping Methods"
        action="delivery.action_delivery_carrier_form"/>

</odoo>

```

