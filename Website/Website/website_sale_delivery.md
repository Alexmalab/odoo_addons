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
    'installable': True,
    'auto_install': True,
    'assets': {
        'web.assets_frontend': [
            'website_sale_delivery/static/src/**/*',
        ],
        'web.assets_tests': [
            'website_sale_delivery/static/tests/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http, _
from odoo.http import request
from odoo.addons.payment import utils as payment_utils
from odoo.addons.website_sale.controllers.main import WebsiteSale, PaymentPortal
from odoo.exceptions import UserError, ValidationError


class WebsiteSaleDelivery(WebsiteSale):
    _express_checkout_shipping_route = '/shop/express/shipping_address_change'

    @http.route()
    def shop_payment(self, **post):
        order = request.website.sale_get_order()
        if order and not order.only_services:
            # Update order's carrier_id (will be the one of the partner if not defined)
            # If a carrier_id is (re)defined, redirect to "/shop/payment" (GET method to avoid infinite loop)
            carrier_id = post.get('carrier_id')
            keep_carrier = False
            if carrier_id:
                carrier_id = int(carrier_id)
            elif order.carrier_id:  # If a carrier is selected.
                keep_carrier = True  # Check availability of selected carrier and recompute rate.
            order._check_carrier_quotation(force_carrier_id=carrier_id, keep_carrier=keep_carrier)
            if carrier_id:
                return request.redirect("/shop/payment")

        return super(WebsiteSaleDelivery, self).shop_payment(**post)

    @http.route(['/shop/update_carrier'], type='json', auth='public', methods=['POST'], website=True, csrf=False)
    def update_eshop_carrier(self, **post):
        order = request.website.sale_get_order()
        carrier_id = int(post['carrier_id'])
        if order and carrier_id != order.carrier_id.id:
            if any(tx.state not in ("cancel", "error", "draft") for tx in order.transaction_ids):
                raise UserError(_('It seems that there is already a transaction for your order, you can not change the delivery method anymore'))
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
        rate = WebsiteSaleDelivery._get_rate(carrier, order)
        if rate.get('success'):
            res['status'] = True
            res['new_amount_delivery'] = Monetary.value_to_html(rate['price'], {'display_currency': order.currency_id})
            res['is_free_delivery'] = not bool(rate['price'])
            res['error_message'] = rate['warning_message']
        else:
            res['status'] = False
            res['new_amount_delivery'] = Monetary.value_to_html(0.0, {'display_currency': order.currency_id})
            res['error_message'] = rate['error_message']
        return res

    @http.route()
    def cart(self, **post):
        order = request.website.sale_get_order()
        if order and order.state != 'draft':
            request.session['sale_order_id'] = None
            order = request.website.sale_get_order()
        if order and order.carrier_id:
            # Express checkout is based on the amout of the sale order. If there is already a
            # delivery line, Express Checkout form will display and compute the price of the
            # delivery two times (One already computed in the total amount of the SO and one added
            # in the form while selecting the delivery carrier)
            order._remove_delivery_line()

        return super().cart(**post)

    @http.route()
    def process_express_checkout(
            self, billing_address, shipping_address=None, shipping_option=None, **kwargs
        ):
        """ Override of `website_sale` to records the shipping information on the order when using
        express checkout flow.

        Depending on whether the partner is registered and logged in, either creates a new partner
        or uses an existing one that matches all received data.

        :param dict billing_address: Billing information sent by the express payment form.
        :param dict shipping_address: Shipping information sent by the express payment form.
        :param dict shipping_option: Carrier information sent by the express payment form.
        :param dict kwargs: Optional data. This parameter is not used here.
        :return int: The order's partner id.
        """
        if not (shipping_address and shipping_option):
            return super().process_express_checkout(billing_address, **kwargs)

        order_sudo = request.website.sale_get_order()

        # Update the partner with all the information
        self._include_country_and_state_in_address(shipping_address)

        # At this point, if the user is a public user, the order will have a partner created by
        # `process_express_checkout_delivery_choice`. No need to check if he is connected or not.

        if order_sudo.partner_shipping_id.name.endswith(order_sudo.name):
            # The existing partner was created by `process_express_checkout_delivery_choice`, it
            # means that the partner is missing information, so we update it.
            order_sudo.partner_shipping_id = self._create_or_edit_partner(
                shipping_address,
                edit=True,
                type='delivery',
                partner_id=order_sudo.partner_shipping_id.id,
            )
        elif any(
            shipping_address[k] != order_sudo.partner_shipping_id[k] for k in shipping_address
        ):
            # The sale order's shipping partner's address is different from the one received. If all
            # the sale order's child partners' address differs from the one received, we create a
            # new partner. The phone isn't always checked because it isn't sent in shipping
            # information with Google Pay.
            child_partner_id = self._find_child_partner(
                order_sudo.partner_id.commercial_partner_id.id, shipping_address
            )
            if child_partner_id:
                order_sudo.partner_shipping_id = child_partner_id
            else:
                order_sudo.partner_shipping_id = self._create_or_edit_partner(
                    shipping_address,
                    type='delivery',
                    parent_id=order_sudo.partner_id.id,
                )

        # Process the delivery carrier
        order_sudo._check_carrier_quotation(force_carrier_id=int(shipping_option['id']))

        return super().process_express_checkout(billing_address, **kwargs)

    @http.route(
        _express_checkout_shipping_route, type='json', auth='public', methods=['POST'],
        website=True, sitemap=False
    )
    def express_checkout_process_shipping_address(self, partial_shipping_address):
        """ Processes shipping address and returns available carriers.

        Depending on whether the partner is registered and logged in or not, creates a new partner
        or uses an existing partner that matches the partial shipping address received.

        :param dict shipping_address: a dictionary containing part of shipping information sent by
                                      the express payment provider.
        :return dict: all available carriers for `shipping_address` sorted by lowest price.
        """
        order_sudo = request.website.sale_get_order()
        public_partner = request.website.partner_id

        self._include_country_and_state_in_address(partial_shipping_address)
        if order_sudo.partner_id == public_partner:
            # The partner_shipping_id and partner_invoice_id will be automatically computed when
            # changing the partner_id of the SO. This allow website_sale to avoid create duplicates.
            order_sudo.partner_id = self._create_or_edit_partner(
                partial_shipping_address,
                type='delivery',
                name=_('Anonymous express checkout partner for order %s', order_sudo.name),
            )
            # Pricelist are recomputed every time the partner is changed. We don't want to recompute
            # the price with another pricelist at this state since the customer has already accepted
            # the amount and validated the payment.
            order_sudo.env.remove_to_compute(
                order_sudo._fields['pricelist_id'], order_sudo
            )
        elif order_sudo.partner_shipping_id.name.endswith(order_sudo.name):
            self._create_or_edit_partner(
                partial_shipping_address,
                edit=True,
                type='delivery',
                partner_id=order_sudo.partner_shipping_id.id,
            )
        elif any(
            partial_shipping_address[k] != order_sudo.partner_shipping_id[k]
            for k in partial_shipping_address
        ):
            # Check if a child partner doesn't already exist with the same informations. The
            # phone isn't always checked because it isn't sent in shipping information with
            # Google Pay.
            child_partner_id = self._find_child_partner(
                order_sudo.partner_id.commercial_partner_id.id, partial_shipping_address
            )
            if child_partner_id:
                order_sudo.partner_shipping_id = child_partner_id
            else:
                order_sudo.partner_shipping_id = self._create_or_edit_partner(
                    partial_shipping_address,
                    type='delivery',
                    parent_id=order_sudo.partner_id.id,
                    name=_('Anonymous express checkout partner for order %s', order_sudo.name),
                )

        # Returns the list of delivery carrier available for the sale order.
        return sorted([{
            'id': carrier.id,
            'name': carrier.name,
            'description': carrier.website_description,
            'minorAmount': payment_utils.to_minor_currency_units(price, order_sudo.currency_id),
        } for carrier, price in WebsiteSaleDelivery._get_carriers_express_checkout(order_sudo).items()
        ], key=lambda carrier: carrier['minorAmount'])

    @staticmethod
    def _get_carriers_express_checkout(order_sudo):
        """ Return available carriers and their prices for the given order.

        :param sale.order order_sudo: The sudoed sales order.
        :rtype: dict
        :return: A dict with a `delivery.carrier` recordset as key, and a rate shipment price as
                 value.
        """
        res = {}
        for carrier in order_sudo._get_delivery_methods():
            rate = WebsiteSaleDelivery._get_rate(carrier, order_sudo, is_express_checkout_flow=True)
            if rate['success']:
                fname = f'{carrier.delivery_type}_use_locations'
                if hasattr(carrier, fname) and getattr(carrier, fname):
                    continue  # Express checkout doesn't allow selecting locations.
                res[carrier] = rate['price']
        return res

    @staticmethod
    def _get_rate(carrier, order, is_express_checkout_flow=False):
        """ Compute the price of the order shipment and apply the taxes if relevant

        :param recordset carrier: the carrier for which the rate is to be recovered
        :param recordset order: the order for which the rate is to be recovered
        :param boolean is_express_checkout_flow: Whether the flow is express checkout or not
        :return dict: the rate, as returned in `rate_shipment()`
        """
        # Some delivery carriers check if all the required fields are available before computing the
        # rate, even if those fields aren't required for computing the rate (although they are for
        # delivering the goods). If we only have partial information about the delivery address but
        # still want to compute the rate, this context key will ensure that we only check the
        # required fields for a partial delivery address (city, zip, country_code, state_code).
        rate = carrier.rate_shipment(order.with_context(
            express_checkout_partial_delivery_address=is_express_checkout_flow
        ))
        if rate.get('success'):
            tax_ids = carrier.product_id.taxes_id.filtered(
                lambda t: t.company_id == order.company_id
            )
            if tax_ids:
                fpos = order.fiscal_position_id
                tax_ids = fpos.map_tax(tax_ids)
                taxes = tax_ids.compute_all(
                    rate['price'],
                    currency=order.currency_id,
                    quantity=1.0,
                    product=carrier.product_id,
                    partner=order.partner_shipping_id,
                )
                if not is_express_checkout_flow and request.env.user.has_group(
                    'account.group_show_line_subtotals_tax_excluded'
                ):
                    rate['price'] = taxes['total_excluded']
                else:
                    rate['price'] = taxes['total_included']
        return rate

    def order_lines_2_google_api(self, order_lines):
        """ Transforms a list of order lines into a dict for google analytics """
        order_lines_not_delivery = order_lines.filtered(lambda line: not line.is_delivery)
        return super(WebsiteSaleDelivery, self).order_lines_2_google_api(order_lines_not_delivery)

    def order_2_return_dict(self, order):
        """ Returns the tracking_cart dict of the order for Google analytics """
        ret = super(WebsiteSaleDelivery, self).order_2_return_dict(order)
        delivery_line = order.order_line.filtered('is_delivery')
        if delivery_line:
            ret['shipping'] = delivery_line.price_unit
        return ret

    def _get_express_shop_payment_values(self, order, **kwargs):
        values = super(WebsiteSaleDelivery, self)._get_express_shop_payment_values(order, **kwargs)
        values['shipping_info_required'] = not order.only_services
        values['shipping_address_update_route'] = self._express_checkout_shipping_route
        return values

    def _get_shop_payment_errors(self, order):
        errors = super()._get_shop_payment_errors(order)

        if not order.only_services and not order._get_delivery_methods():
            errors.append((
                _('Sorry, we are unable to ship your order'),
                _('No shipping method is available for your current order and shipping address. '
                   'Please contact us for more information.'),
            ))
        return errors

    def _get_shop_payment_values(self, order, **kwargs):
        values = super(WebsiteSaleDelivery, self)._get_shop_payment_values(order, **kwargs)
        has_storable_products = any(line.product_id.type in ['consu', 'product'] for line in order.order_line)

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
                'new_amount_total_raw': order.amount_total,
            }
        return {}


class PaymentPortalDelivery(PaymentPortal):

    @http.route()
    def shop_payment_transaction(self, *args, **kwargs):
        order = request.website.sale_get_order()
        if not order.only_services and not order.carrier_id:
            raise ValidationError(_("No shipping method is selected."))
        return super().shop_payment_transaction(*args, **kwargs)

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
            res = res & states
        return res

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class SaleOrder(models.Model):
    _inherit = 'sale.order'

    amount_delivery = fields.Monetary(
        compute='_compute_amount_delivery',
        string='Delivery Amount',
        help="The amount without tax.", store=True, tracking=True)

    @api.depends('order_line.price_unit', 'order_line.tax_id', 'order_line.discount', 'order_line.product_uom_qty')
    def _compute_amount_delivery(self):
        for order in self:
            if self.env.user.has_group('account.group_show_line_subtotals_tax_excluded'):
                order.amount_delivery = sum(order.order_line.filtered('is_delivery').mapped('price_subtotal'))
            else:
                order.amount_delivery = sum(order.order_line.filtered('is_delivery').mapped('price_total'))

    def _check_carrier_quotation(self, force_carrier_id=None, keep_carrier=False):
        self.ensure_one()
        DeliveryCarrier = self.env['delivery.carrier']

        if self.only_services:
            self._remove_delivery_line()
            return True
        else:
            self = self.with_company(self.company_id)
            # attempt to use partner's preferred carrier
            if not force_carrier_id and self.partner_shipping_id.property_delivery_carrier_id and not keep_carrier:
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
        # searching on website_published will also search for available website (_search method on
        # computed field)
        return self.env['delivery.carrier'].sudo().search([
            ('website_published', '=', True),
        ]).filtered(lambda carrier: carrier._is_available_for_order(self))

    def _cart_update(self, *args, **kwargs):
        """ Override to update carrier quotation if quantity changed """
        self._remove_delivery_line()
        return super()._cart_update(*args, **kwargs)

```

## File: models\sale_order_line.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models

class SaleOrderLine(models.Model):
    _inherit = 'sale.order.line'

    def _show_in_cart(self):
        # Exclude delivery line from showing up in the cart
        return not self.is_delivery and super()._show_in_cart()

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import delivery
from . import res_country
from . import sale_order_line
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
        var $payButton = $('button[name="o_payment_submit_button"]');
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
        $carrierInput.siblings('.o_wsale_delivery_badge_price').empty();
        $carrierInput.siblings('.o_wsale_delivery_badge_price').append('<span class="fa fa-circle-o-notch fa-spin"/>');
    },
    /**
     * Update the total cost according to the selected shipping method
     * 
     * @private
     * @param {float} amount : The new total amount of to be paid
     */
    _updateShippingCost: function(amount){
        core.bus.trigger('update_shipping_cost', amount);
    },
    /**
     * @private
     * @param {Object} result
     */
    _handleCarrierUpdateResult: function (result) {
        this._handleCarrierUpdateResultBadge(result);
        var $payButton = $('button[name="o_payment_submit_button"]');
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
        if (result.new_amount_total_raw !== undefined) {
            this._updateShippingCost(result.new_amount_total_raw);
            // reload page only when amount_total switches between zero and not zero
            const hasPaymentMethod = document.querySelector(
                "div[name='o_website_sale_free_cart']"
            ) === null;
            const shouldDisplayPaymentMethod = result.new_amount_total_raw !== 0;
            if (hasPaymentMethod !==  shouldDisplayPaymentMethod) {
                location.reload(false);
            }
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
        var $payButton = $('button[name="o_payment_submit_button"]');
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
              <td class="text-end border-0 text-muted"  title="Delivery will be updated after choosing a new delivery method">Delivery:</td>
              <td class="text-xl-end border-0 text-muted" >
                   <span t-field="website_sale_order.amount_delivery" class="monetary_field" style="white-space: nowrap;" t-options='{
                      "widget": "monetary",
                      "display_currency": website_sale_order.currency_id,
                  }'/>
              </td>
            </tr>
        </xpath>
    </template>

    <template id="payment_delivery_methods">
        <input t-att-value="delivery.id" t-att-id="'delivery_%i' % delivery.id" type="radio" name="delivery_type" t-att-checked="order.carrier_id and order.carrier_id.id == delivery.id and 'checked' or False" t-att-class="'d-none' if delivery_nb == 1 else ''"/>
        <label class="label-optional" t-field="delivery.name"/>
        <t t-set='badge_class' t-value="(delivery_nb != 1 and 'float-end ' or '') + 'badge text-bg-secondary'" />
        <span t-attf-class="#{badge_class} o_wsale_delivery_badge_price">Select to compute delivery rate</span>
        <t t-if="delivery.website_description">
            <div t-field="delivery.website_description" class="text-muted mt8"/>
        </t>
    </template>

    <template id="payment_delivery_shipping_method" name="Delivery Shipping method" inherit_id="website_sale.address_on_payment">
        <xpath expr="//div[@id='shipping_and_billing']" position="inside">
            <t t-set="delivery_nb" t-value="deliveries and len(deliveries) or 0"/>
            <div t-if="delivery_nb == 1" id="delivery_carrier" class="mt4">
                <b>Shipping Method: </b>
                <t t-foreach="deliveries" t-as="delivery">
                    <div class="o_delivery_carrier_select d-inline">
                        <t t-call="website_sale_delivery.payment_delivery_methods"/>
                    </div>
                </t>
            </div>
        </xpath>
    </template>

    <template id="payment_delivery" name="Delivery Costs" inherit_id="website_sale.payment">
        <!-- //t[@t-if='website_sale_order.amount_total'] should be removed in master -->
        <xpath expr="//t[@name='website_sale_non_free_cart'] | //t[@t-if='website_sale_order.amount_total']" position="before">
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
            <field name="carrier_description" position='before'>
                <field name="website_description"  placeholder="Description displayed on the eCommerce and on online quotations."/>
            </field>
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

