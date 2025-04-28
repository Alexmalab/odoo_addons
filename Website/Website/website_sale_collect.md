# Odoo Module: website_sale_collect

Category: Website/Website

This file contains the source code of the Odoo module.

## File: const.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# The codes of the payment method to activate when Pay on site is activated.
DEFAULT_PAYMENT_METHOD_CODES = {
    'pay_on_site',
}

```

## File: utils.py

```python
import math


def format_product_stock_values(product, wh_id=None, free_qty=None):
    """ Format product stock values for the location selector.

    :param product.product|product.template product: The product whose stock values to format.
    :param int wh_id: The warehouse whose stock to check for the given product.
    :param bool free_qty: The free quantity of the product. If not given, calculated from the
                          warehouse.
    :return: The formatted product stock values.
    :rtype: dict
    """
    if product.is_product_variant:  # Only available for `product.product` records.
        if free_qty is None:
            free_qty = product.with_context(warehouse_id=wh_id).free_qty
        return {
            'in_stock': free_qty > 0,
            'show_quantity': product.show_availability and product.available_threshold >= free_qty,
            'quantity': free_qty,
        }
    else:
        return {}


def calculate_partner_distance(partner1, partner2):
    """ Calculate the Haversine distance between two partners.

    See https://en.wikipedia.org/wiki/Haversine_formula.

    :param res.partner partner1: The partner to calculate distance from.
    :param res.partner partner2: The partner to calculate distance to.
    :return: The distance between the two partners (in kilometers).
    :rtype: float
    """
    R = 6371  # The radius of Earth.
    lat1, long1 = partner1.partner_latitude, partner1.partner_longitude
    lat2, long2 = partner2.partner_latitude, partner2.partner_longitude
    dlat = math.radians(lat2 - lat1)
    dlong = math.radians(long2 - long1)
    arcsin = (
        math.sin(dlat / 2) * math.sin(dlat / 2)
        + math.cos(math.radians(lat1)) * math.cos(math.radians(lat2))
        * (math.sin(dlong / 2) * math.sin(dlong / 2))
    )
    d = 2 * R * math.atan2(math.sqrt(arcsin), math.sqrt(1 - arcsin))

    return d

```

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import controllers
from . import utils

from odoo.addons.payment import reset_payment_provider


def uninstall_hook(env):
    reset_payment_provider(env, 'custom', custom_mode='on_site')

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': "Click & Collect",
    'version': '1.0',
    'category': 'Website/Website',
    'description': """
Allows customers to check in-store stock, pay on site, and pick up their orders at the shop.
""",
    'depends': ['base_geolocalize', 'payment_custom', 'website_sale_stock'],
    'data': [
        'data/payment_method_data.xml',
        'data/payment_provider_data.xml',  # Depends on `payment_method_pay_on_site`.
        'data/product_product_data.xml',
        'data/delivery_carrier_data.xml',  # Depends on `product_pick_up_in_store`.

        'views/delivery_carrier_views.xml',
        'views/delivery_form_templates.xml',
        'views/res_config_settings_views.xml',
        'views/stock_picking_views.xml',
        'views/stock_warehouse_views.xml',
        'views/templates.xml',
    ],
    'demo': [
        'data/demo.xml',
    ],
    'assets': {
        'web.assets_frontend': [
            'website_sale_collect/static/src/**/*',
        ],
    },
    'uninstall_hook': 'uninstall_hook',
    'license': 'LGPL-3',
}

```

## File: controllers\delivery.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.http import request, route

from odoo.addons.website_sale.controllers.delivery import Delivery


class InStoreDelivery(Delivery):

    @route()
    def website_sale_get_pickup_locations(self, zip_code=None, **kwargs):
        """ Override of `website_sale` to set the pickup in store delivery method on the order in
        order to retrieve pickup locations when called from the the product page.
        """
        if kwargs.get('product_id'):  # Called from the product page.
            order_sudo = request.website.sale_get_order(force_create=True)
            in_store_dm = request.website.sudo().in_store_dm_id
            if order_sudo.carrier_id.delivery_type != 'in_store':
                order_sudo.set_delivery_line(in_store_dm, in_store_dm.product_id.list_price)
        return super().website_sale_get_pickup_locations(zip_code, **kwargs)

    @route('/shop/set_click_and_collect_location', type='json', auth='public', website=True)
    def shop_set_click_and_collect_location(self, pickup_location_data):
        """ Set the pickup location and the in-store delivery method on the current order.

        This route is distinct from /website_sale/get_pickup_locations as the latter is only called
        from the checkout page after the delivery method is selected.

        :param str pickup_location_data: The JSON-formatted pickup location data.
        :return: None
        """
        order_sudo = request.website.sale_get_order()
        if order_sudo.carrier_id.delivery_type != 'in_store':
            in_store_dm = request.website.sudo().in_store_dm_id
            order_sudo.set_delivery_line(in_store_dm, in_store_dm.product_id.list_price)
        order_sudo._set_pickup_location(pickup_location_data)

```

## File: controllers\main.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _
from odoo.http import request

from odoo.addons.website_sale.controllers.main import WebsiteSale


class WebsiteSaleCollect(WebsiteSale):

    def _prepare_product_values(self, product, category, search, **kwargs):
        """ Override of `website_sale` to include the selected pickup location and zip code. """
        res = super()._prepare_product_values(product, category, search, **kwargs)
        if request.website.sudo().in_store_dm_id:
            order_sudo = request.website.sale_get_order()
            if (
                order_sudo.carrier_id.delivery_type == 'in_store'
                and order_sudo.pickup_location_data
            ):
                res['selected_wh_location'] = order_sudo.pickup_location_data
            res['zip_code'] = (  # Define the zip code.
                order_sudo.partner_shipping_id.zip
                or res.get('selected_wh_location', {}).get('zip_code')
                or request.geoip.postal.code
            )
        return res

    def _prepare_checkout_page_values(self, order_sudo, **query_params):
        """ Override of `website_sale` to include the unavailable products for the selected pickup
        location. """
        res = super()._prepare_checkout_page_values(order_sudo, **query_params)
        if order_sudo.carrier_id.delivery_type == 'in_store' and order_sudo.pickup_location_data:
            res['unavailable_order_lines'] = order_sudo._get_unavailable_order_lines(
                order_sudo.pickup_location_data.get('id')
            )
        return res

    def _get_shop_payment_errors(self, order):
        """ Override of `website_sale` to includes errors if no pickup location is selected or some
        products are unavailable. """
        errors = super()._get_shop_payment_errors(order)
        if (
            order.state != 'sale'
            and order._has_deliverable_products()
            and order.carrier_id.delivery_type == 'in_store'
        ):
            if not order.pickup_location_data:
                errors.append((
                    _("Sorry, we are unable to ship your order."),
                    _("Please choose a store to collect your order."),
                ))
            else:
                selected_wh_id = order.pickup_location_data['id']
                if not order._is_in_stock(selected_wh_id):
                    errors.append((
                        _("Sorry, we are unable to ship your order."),
                        _("Some products are not available in the selected store."),
                    ))
        return errors

```

## File: controllers\payment.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _
from odoo.exceptions import ValidationError

from odoo.addons.website_sale.controllers.payment import PaymentPortal


class OnSitePaymentPortal(PaymentPortal):

    def _validate_transaction_for_order(self, transaction, sale_order):
        """ Override of `website_sale` to ensure the on-site payment provider is not used without
        the in-store pickup delivery method.

        This also sets the warehouse of the selected pickup location on the sales order.

        :param payment.transaction transaction: The transaction used to make the payment.
        :param sale.order sale_order: The sales order to pay.
        :return: None
        :raises ValidationError: If the user tries to pay on site without the in-store pickup
                                 delivery method.
        """
        super()._validate_transaction_for_order(transaction, sale_order)

        # This should never be triggered unless the user intentionally forges a request.
        provider = transaction.provider_id
        if (
            sale_order.carrier_id.delivery_type != 'in_store'
            and provider.code == 'custom'
            and provider.custom_mode == 'on_site'
        ):
            raise ValidationError(
                _("You can only pay on site when selecting the pick up in store delivery method.")
            )

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import delivery
from . import main
from . import payment

```

## File: data\delivery_carrier_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="carrier_pick_up_in_store" model="delivery.carrier">
        <field name="name">Pick up in store</field>
        <field name="delivery_type">in_store</field>
        <field name="product_id" ref="website_sale_collect.product_pick_up_in_store"/>
        <field name="website_id" ref="website.default_website"/>
    </record>

</odoo>

```

## File: data\demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="website_sale_collect.payment_provider_on_site" model="payment.provider">
        <field name="state">disabled</field>
    </record>

</odoo>

```

## File: data\payment_method_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_method_pay_on_site" model="payment.method">
        <field name="name">Pay on site</field>
        <field name="code">pay_on_site</field>
        <field name="sequence">1000</field>
        <field name="image" type="base64" file="website_sale_collect/static/img/pay_on_site.png"/>
        <field name="support_tokenization">False</field>
        <field name="support_express_checkout">False</field>
        <field name="support_refund">none</field>
    </record>

</odoo>

```

## File: data\payment_provider_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="payment_provider_on_site" model="payment.provider">
        <field name="name">Pay on Site</field>
        <field name="module_id" ref="base.module_website_sale_collect"/>
        <field name="code">custom</field>
        <field name="state">enabled</field>
        <field name="custom_mode">on_site</field>
        <field name="payment_method_ids"
               eval="[(6, 0, [
                         ref('website_sale_collect.payment_method_pay_on_site'),
                     ])]"
        />
        <field
            name="image_128"
            type="base64"
            file="website_sale_collect/static/description/icon.png"
        />
        <field name="redirect_form_view_id" ref="payment_custom.redirect_form"/>
        <field name="pending_msg" type="html">
            <p>
                <i>Your order has been confirmed.</i><br/>Please come to the store to pay for your products.
            </p>
        </field>
    </record>

</odoo>

```

## File: data\product_product_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="product_pick_up_in_store" model="product.product">
        <field name="name">Pick up in store</field>
        <field name="type">service</field>
        <field name="list_price">0</field>
        <field name="purchase_ok">false</field>
        <field name="sale_ok">false</field>
    </record>

</odoo>

```

## File: models\delivery_carrier.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError
from odoo.http import request
from odoo.tools.misc import format_duration

from odoo.addons.website_sale_collect import utils


class DeliveryCarrier(models.Model):
    _inherit = 'delivery.carrier'

    delivery_type = fields.Selection(
        selection_add=[('in_store', "Pick up in store")], ondelete={'in_store': 'set default'}
    )
    warehouse_ids = fields.Many2many(string="Stores", comodel_name='stock.warehouse')

    @api.constrains('delivery_type', 'is_published', 'warehouse_ids')
    def _check_in_store_dm_has_warehouses_when_published(self):
        if any(self.filtered(
            lambda dm: dm.delivery_type == 'in_store'
            and dm.is_published
            and not dm.warehouse_ids
        )):
            raise ValidationError(
                _("The delivery method must have at least one warehouse to be published.")
            )

    @api.constrains('delivery_type', 'company_id', 'warehouse_ids')
    def _check_warehouses_have_same_company(self):
        for dm in self:
            if dm.delivery_type == 'in_store' and dm.company_id and any(
                wh.company_id and dm.company_id != wh.company_id for wh in dm.warehouse_ids
            ):
                raise ValidationError(
                    _("The delivery method and a warehouse must share the same company")
                )

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            if vals.get('delivery_type') == 'in_store':
                vals['integration_level'] = 'rate'
        return super().create(vals_list)

    def write(self, vals):
        if vals.get('delivery_type') == 'in_store':
            vals['integration_level'] = 'rate'
        return super().write(vals)

    # === BUSINESS METHODS ===#

    def _in_store_get_close_locations(self, partner_address, product_id=None):
        """ Get the formatted close pickup locations sorted by distance to the partner address.

        :param res.partner partner_address: The address to use to sort the pickup locations.
        :param str product_id: The product whose product page was used to open the location
                               selector, if any, as a `product.product` id.
        :return: The sorted and formatted close pickup locations.
        :rtype: list[dict]
        """
        try:
            product_id = product_id and int(product_id)
        except ValueError:
            product = self.env['product.product']
        else:
            product = self.env['product.product'].browse(product_id)

        partner_address.geo_localize()  # Calculate coordinates.

        pickup_locations = []
        order_sudo = request.website.sale_get_order()
        for wh in self.warehouse_ids:
            # Prepare the stock data based on either the product or the order.
            if product:  # Called from the product page.
                in_store_stock_data = utils.format_product_stock_values(product, wh.id)
            else:  # Called from the checkout page.
                in_store_stock_data = {'in_stock': order_sudo._is_in_stock(wh.id)}

            # Prepare the warehouse location.
            wh_location = wh.partner_id
            if not wh_location.partner_latitude or not wh_location.partner_longitude:
                wh_location.geo_localize()  # Find the longitude and latitude of the warehouse.

            # Format the pickup location values of the warehouse.
            try:
                pickup_location_values = {
                    'id': wh.id,
                    'name': wh_location['name'].title(),
                    'street': wh_location['street'].title(),
                    'city': wh_location.city.title(),
                    'zip_code': wh_location.zip,
                    'country_code': wh_location.country_code,
                    'state': wh_location.state_id.code,
                    'latitude': wh_location.partner_latitude,
                    'longitude': wh_location.partner_longitude,
                    'additional_data': {'in_store_stock': in_store_stock_data},
                }
            except AttributeError:
                continue  # Ignore warehouses with badly configured address.

            # Prepare the opening hours data.
            if wh.opening_hours:
                opening_hours_dict = {str(i): [] for i in range(7)}
                for att in wh.opening_hours.attendance_ids:
                    if att.day_period in ('morning', 'afternoon'):
                        opening_hours_dict[att.dayofweek].append(
                            f'{format_duration(att.hour_from)} - {format_duration(att.hour_to)}'
                        )
                pickup_location_values['opening_hours'] = opening_hours_dict
            else:
                pickup_location_values['opening_hours'] = {}

            # Calculate the distance between the partner address and the warehouse location.
            pickup_location_values['distance'] = utils.calculate_partner_distance(
                partner_address, wh_location
            )
            pickup_locations.append(pickup_location_values)

        return sorted(pickup_locations, key=lambda k: k['distance'])

    def in_store_rate_shipment(self, *_args):
        return {
            'success': True,
            'price': self.product_id.list_price,
            'error_message': False,
            'warning_message': False,
        }

```

## File: models\payment_provider.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models

from odoo.addons.payment import utils as payment_utils
from odoo.addons.website_sale_collect import const


class PaymentProvider(models.Model):
    _inherit = 'payment.provider'

    custom_mode = fields.Selection(selection_add=[('on_site', "Pay on site")])

    @api.model
    def _get_compatible_providers(
        self, company_id, *args, sale_order_id=None, website_id=None, report=None, **kwargs
    ):
        """ Override of payment to exclude on-site payment providers if the delivery method is not
        pick up in store.

        :param int company_id: The company to which providers must belong, as a `res.company` id
        :param int sale_order_id: The sale order to be paid, if any, as a `sale.order` id
        :param int website_id: The provided website, as a `website` id
        :param dict report: The availability report.
        :return: The compatible providers
        :rtype: recordset of `payment.provider`
        """
        compatible_providers = super()._get_compatible_providers(
            company_id,
            *args,
            sale_order_id=sale_order_id,
            website_id=website_id,
            report=report,
            **kwargs,
        )
        order = self.env['sale.order'].browse(sale_order_id).exists()

        # Show on-site payment providers only if in-store delivery methods exist and the order
        # contains physical products.
        if order.carrier_id.delivery_type != 'in_store' or not any(
            product.type == 'consu' for product in order.order_line.product_id
        ):
            unfiltered_providers = compatible_providers
            compatible_providers = compatible_providers.filtered(
                lambda p: p.code != 'custom' or p.custom_mode != 'on_site'
            )
            payment_utils.add_to_report(
                report,
                unfiltered_providers - compatible_providers,
                available=False,
                reason=_("no in-store delivery methods available"),
            )

        return compatible_providers

    def _get_default_payment_method_codes(self):
        """ Override of `payment` to return the default payment method codes. """
        default_codes = super()._get_default_payment_method_codes()
        if self.custom_mode != 'on_site':
            return default_codes
        return const.DEFAULT_PAYMENT_METHOD_CODES

```

## File: models\payment_transaction.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class PaymentTransaction(models.Model):
    _inherit = 'payment.transaction'

    def _post_process(self):
        """ Override of `payment` to confirm orders with the on_site payment method and trigger
        a picking creation. """
        on_site_pending_txs = self.filtered(
            lambda tx: tx.provider_id.custom_mode == 'on_site' and tx.state == 'pending'
        )
        on_site_pending_txs.sale_order_ids.filtered(
            lambda so: so.state == 'draft'
        ).with_context(send_email=True).action_confirm()
        super()._post_process()

```

## File: models\product_template.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models

from odoo.addons.website_sale_collect import utils


class ProductTemplate(models.Model):
    _inherit = 'product.template'

    def _get_additionnal_combination_info(self, product_or_template, quantity, date, website):
        """ Override of `website_sale` to add information on whether Click & Collect is enabled and
        on the in-store stock of the product. """
        res = super()._get_additionnal_combination_info(
            product_or_template, quantity, date, website
        )
        if (
            bool(website.sudo().in_store_dm_id)  # Click & Collect is enabled.
            and product_or_template.is_product_variant
            and product_or_template.is_storable
        ):
            res['show_click_and_collect_availability'] = True
            order_sudo = website.sale_get_order()
            if (
                order_sudo
                and order_sudo.carrier_id.delivery_type == 'in_store'
                and order_sudo.pickup_location_data
            ):  # Get stock values for the product variant in the selected store.
                res['in_store_stock'] = utils.format_product_stock_values(
                    product_or_template.sudo(), wh_id=order_sudo.pickup_location_data['id']
                )
            else:
                res['in_store_stock'] = utils.format_product_stock_values(
                    product_or_template.sudo(),
                    free_qty=website.sudo()._get_max_in_store_product_available_qty(
                        product_or_template.sudo()
                    )
                )
        return res

```

## File: models\res_config_settings.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    def action_view_in_store_delivery_methods(self):
        """ Return an action to browse pickup delivery methods in list view, or in form view if
        there is only one. """
        in_store_dms = self.env['delivery.carrier'].search([('delivery_type', '=', 'in_store')])
        if len(in_store_dms) == 1:
            return {
                'type': 'ir.actions.act_window',
                'res_model': 'delivery.carrier',
                'view_mode': 'form',
                'res_id': in_store_dms.id,
            }
        return {
            'type': 'ir.actions.act_window',
            'name': _("Delivery Methods"),
            'res_model': 'delivery.carrier',
            'view_mode': 'list,form',
            'context': '{"search_default_delivery_type": "in_store"}',
        }

```

## File: models\sale_order.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json

from odoo import _, models
from odoo.exceptions import ValidationError
from odoo.http import request


class SaleOrder(models.Model):
    _inherit = 'sale.order'

    def set_delivery_line(self, carrier, amount):
        """ Override of `website_sale` to recompute warehouse and fiscal position when a new
        delivery method is not in-store anymore. """
        in_store_orders = self.filtered(
            lambda so: (
                so.carrier_id.delivery_type == 'in_store' and carrier.delivery_type != 'in_store'
            )
        )
        in_store_orders._compute_warehouse_id()
        in_store_orders._compute_fiscal_position_id()
        return super().set_delivery_line(carrier, amount)

    def _set_pickup_location(self, pickup_location_data):
        """ Override `website_sale` to set the pickup location for in-store delivery methods.
        Set account fiscal position depending on selected pickup location to correctly calculate
        taxes.
        """
        res = super()._set_pickup_location(pickup_location_data)
        if self.carrier_id.delivery_type != 'in_store':
            return res

        self.pickup_location_data = json.loads(pickup_location_data)
        if self.pickup_location_data:
            self.warehouse_id = self.pickup_location_data['id']
            AccountFiscalPosition = self.env['account.fiscal.position'].sudo()
            self.fiscal_position_id = AccountFiscalPosition._get_fiscal_position(
                self.partner_id, delivery=self.warehouse_id.partner_id
            )
        else:
            self._compute_warehouse_id()

    def _get_pickup_locations(self, zip_code=None, country=None, **kwargs):
        """ Override of `website_sale` to ensure that a country is provided when there is a zip
        code.

        If the country cannot be found (e.g., the GeoIP request fails), the zip code is cleared to
        prevent the parent method's assertion to fail.
        """
        if zip_code and not country:
            country_code = None
            if self.pickup_location_data:
                country_code = self.pickup_location_data['country_code']
            elif request.geoip.country_code:
                country_code = request.geoip.country_code
            country = self.env['res.country'].search([('code', '=', country_code)], limit=1)
            if not country:
                zip_code = None  # Reset the zip code to skip the `assert` in the `super` call.
        return super()._get_pickup_locations(zip_code=zip_code, country=country, **kwargs)

    def _get_cart_and_free_qty(self, product, line=None):
        """ Override of `website_sale_stock` to get free_qty of the product from the warehouse that
        was chosen rather than website's one.

        :param product.product product: The product
        :param sale.order.line line: The optional line
        """
        cart_qty, free_qty = super()._get_cart_and_free_qty(product, line=line)
        if self.carrier_id.delivery_type == 'in_store':
            free_qty = (product or line.product_id).with_context(
                warehouse_id=self.warehouse_id.id
            ).free_qty
        return cart_qty, free_qty

    def _check_cart_is_ready_to_be_paid(self):
        """ Override of `website_sale` to check if all products are in stock in the selected
        warehouse. """
        if (
            self._has_deliverable_products()
            and self.carrier_id.delivery_type == 'in_store'
            and not self._is_in_stock(self.warehouse_id.id)
        ):
            raise ValidationError(_("Some products are not available in the selected store."))
        return super()._check_cart_is_ready_to_be_paid()

    # === TOOLING ===#

    def _is_in_stock(self, wh_id):
        """ Check whether all storable products of the cart are in stock in the given warehouse.

        :param int wh_id: The warehouse in which to check the stock, as a `stock.warehouse` id.
        :return: Whether all storable products are in stock.
        :rtype: bool
        """
        return not self._get_unavailable_order_lines(wh_id)

    def _get_unavailable_order_lines(self, wh_id):
        """ Return the order lines with unavailable products for the given warehouse.

        :param int wh_id: The warehouse in which to check the stock, as a `stock.warehouse` id.
        :return: The order lines with unavailable products.
        :rtype: sale.order.line
        """
        unavailable_order_lines = self.env['sale.order.line']
        for ol in self.order_line:
            if ol.is_storable:
                product_free_qty = ol.product_id.with_context(warehouse_id=wh_id).free_qty
                if ol.product_uom_qty > product_free_qty:
                    ol.shop_warning = _(
                        'Only %(new_qty)s available', new_qty=int(max(product_free_qty, 0))
                    )
                    unavailable_order_lines |= ol
        return unavailable_order_lines

    def _verify_updated_quantity(self, order_line, product_id, new_qty, **kwargs):
        """ Override of `website_sale_stock` to skip the verification when click and collect
        is activated. The quantity is verified later. """
        self.ensure_one()
        product = self.env['product.product'].browse(product_id)
        if (
            product.is_storable
            and not product.allow_out_of_stock_order
            and self.website_id.in_store_dm_id
        ):
            return new_qty, ''
        return super()._verify_updated_quantity(order_line, product_id, new_qty, **kwargs)

```

## File: models\stock_warehouse.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class StockWarehouse(models.Model):
    _inherit = 'stock.warehouse'

    opening_hours = fields.Many2one(
        string="Opening Hours", comodel_name='resource.calendar', check_company=True
    )

```

## File: models\website.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class Website(models.Model):
    _inherit = 'website'

    in_store_dm_id = fields.Many2one(
        string="In-store Delivery Method",
        comodel_name='delivery.carrier',
        compute='_compute_in_store_dm_id',
    )

    def _compute_in_store_dm_id(self):
        in_store_delivery_methods = self.env['delivery.carrier'].search(
            [('delivery_type', '=', 'in_store'), ('is_published', '=', True)]
        )
        for website in self:
            website.in_store_dm_id = in_store_delivery_methods.filtered_domain([
               '|', ('website_id', '=', False), ('website_id', '=', website.id),
               '|', ('company_id', '=', False), ('company_id', '=', website.company_id.id),
            ])[:1]

    def _get_product_available_qty(self, product, **kwargs):
        """ Override of `website_sale_stock` to include free quantities of the product in warehouses
         of in-store delivery method and return maximum possible for one order. Needed only if a
         warehouse is set on website, otherwise free quantity is already calculated from all
         warehouses."""
        free_qty = super()._get_product_available_qty(product, **kwargs)
        if self.warehouse_id and self.sudo().in_store_dm_id:  # If warehouse is set on website.
            # Check free quantities in the in-store warehouses.
            free_qty = max(free_qty, self._get_max_in_store_product_available_qty(product))
        return free_qty

    def _get_max_in_store_product_available_qty(self, product):
        """ Return maximum amount of product available to deliver with in store delivery method. """
        return max([
            product.with_context(warehouse_id=wh.id).free_qty
            for wh in self.sudo().in_store_dm_id.warehouse_ids
        ], default=0)

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import delivery_carrier
from . import payment_provider
from . import payment_transaction
from . import product_template
from . import res_config_settings
from . import sale_order
from . import stock_warehouse
from . import website

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M9.5 8 6 36h6.019a6.5 6.5 0 0 0 12.962 0H25a6 6 0 0 0 12 0h.019a6.5 6.5 0 0 0 12.962 0H50v-1l-.045-.273a6.231 6.231 0 0 0-.058-.386l-.023-.118-.02-.1-3.797-22.78A4 4 0 0 0 42.112 8H9.5Z" fill="#1AD3BB"/><path d="m14 8-2 28a6 6 0 0 1-12 0v-1l3.494-23.586A4 4 0 0 1 7.451 8H14Zm21 0 2 28a6 6 0 0 1-12 0V8h10Z" fill="#985184"/><path d="M12 36a6 6 0 1 1-12 0h12Zm25 0a6 6 0 0 1-12 0h12Z" fill="#712258"/><path d="M12.02 36a6.5 6.5 0 0 0 12.961 0H12.02Zm25 0a6.5 6.5 0 0 0 12.961 0H37.02Z" fill="#03AF89"/></svg>

```

## File: static\src\js\checkout.js

```javascript
import { rpc } from '@web/core/network/rpc';
import publicWidget from '@web/legacy/js/public/public_widget';

publicWidget.registry.WebsiteSaleCheckout.include({
    events: Object.assign({}, publicWidget.registry.WebsiteSaleCheckout.prototype.events, {
        'click .js_delete_product': '_onClickDeleteProduct',
    }),

    // #=== EVENT HANDLERS ===#

    /**
     * Remove a product from the cart.
     *
     * @private
     * @param {Event} ev
     */
    async _onClickDeleteProduct(ev) {
        await rpc('/shop/cart/update_json', {
            product_id: parseInt(ev.target.dataset.productId, 10),
            set_qty: 0,
            display: false,  // No need to return the rendered templates.
        });
        window.location.reload();  // Reload all cart values.
    },

    // #=== DOM MANIPULATION ===#

    /**
     * Remove a warning if available pickup location is selected.
     *
     * @override method from `@website_sale/js/checkout`
     */
    _updatePickupLocation(button, location, jsonLocation) {
        this._super.apply(this, arguments);
        const dmContainer = this._getDeliveryMethodContainer(button);
        const radio = dmContainer.querySelector('[name="o_delivery_radio"]');
        if (radio.dataset.deliveryType === 'in_store') {
            dmContainer.querySelector('[name="unavailable_products_warning"]')?.remove();
        }
    },

    /**
     * Return false if there is a warning message, otherwise return the result of the parent method
     * call.
     *
     * @override method from `@website_sale/js/checkout`
     */
    _canEnableMainButton() {
        if (this.dmRadios.length === 0) {  // If there are no delivery methods.
            return this._super.apply(this, arguments);  // Skip override.
        }
        // TODO: move logic below to `_isDeliveryMethodReady` override on master
        const checkedRadio = this.el.querySelector('input[name="o_delivery_radio"]:checked');
        let hasWarning = false;
        if (checkedRadio) {
            const deliveryContainer = this._getDeliveryMethodContainer(checkedRadio);
            hasWarning = (
                checkedRadio.dataset.deliveryType === 'in_store'
                && deliveryContainer.querySelector('[name="unavailable_products_warning"]')
            );
        }
        return this._super.apply(this, arguments) && !hasWarning;
    },

    /**
     * Also hide the warning message, if any.
     *
     * @override method from `@website_sale/js/checkout`
     */
    _hidePickupLocation() {
        this._super.apply(this, arguments);
        const warning = this.el.querySelector('[name="unavailable_products_warning"]');
        if (warning) {
            warning.classList.add('d-none');
        }
    },

    // #=== DELIVERY FLOW ===#

    /**
     * Display a warning if any when selecting an in_store delivery method.
     *
     * @override method from `@website_sale/js/checkout`
     */
    async _showPickupLocation(radio) {
        this._super.apply(this, arguments);
        if (radio.dataset.deliveryType === 'in_store') {
            const dmContainer = this._getDeliveryMethodContainer(radio);
            const warning = dmContainer.querySelector('[name="unavailable_products_warning"]');
            if (warning) {
                warning.classList.remove('d-none');
            }
        }
    },

});

```

## File: static\src\js\website_sale.js

```javascript
import { Component } from '@odoo/owl';
import publicWidget from '@web/legacy/js/public/public_widget';

publicWidget.registry.WebsiteSale.include({
    /**
     * Trigger a state update of the ClickAndCollectAvailability component when the combination info
     * is updated.
     *
     * @override
     */
    _onChangeCombination(ev, $parent, combination) {
        const res = this._super.apply(this, arguments);
        Component.env.bus.trigger('updateCombinationInfo', combination);
        return res;
    },

});

```

## File: static\src\js\click_and_collect_availability\click_and_collect_availability.js

```javascript
import { Component, useState, onWillDestroy } from '@odoo/owl';
import { rpc } from '@web/core/network/rpc';
import { registry } from '@web/core/registry';
import { useService } from '@web/core/utils/hooks';

import {
    LocationSelectorDialog
} from '@delivery/js/location_selector/location_selector_dialog/location_selector_dialog';

export class ClickAndCollectAvailability extends Component {
    static template = 'website_sale_collect.ClickAndCollectAvailability';
    static props = {
        productId: Number,
        active: {type: Boolean, optional: true},
        zipCode: { type: String, optional: true },
        selectedWhLocation: { type: Object, optional: true },
        inStoreStock: { type: Object, optional: true },
    }
    static defaultProps = {
        active: true,
    }
    setup() {
        super.setup();
        this.dialog = useService('dialog');
        this.state = useState({
            productId: this.props.productId,
            selectedWhLocation: this.props.selectedWhLocation,
            inStoreStock: this.props.inStoreStock,
            active: this.props.active,
        });
        const updateState = this._updateStateWithCombinationInfo.bind(this);
        this.env.bus.addEventListener('updateCombinationInfo', res => updateState(res.detail));
        onWillDestroy(() => this.env.bus.removeEventListener('updateCombinationInfo', updateState));
    }

    /**
     * Update the state with the product combination info.
     *
     * @private
     * @param {Object} combinationInfo - The information on the current product variant.
     * @return {void}
     */
    _updateStateWithCombinationInfo (combinationInfo) {
        this.state.productId = combinationInfo.product_id;
        this.state.inStoreStock = combinationInfo.in_store_stock;
        this.state.active = combinationInfo.is_combination_possible;
    }

    /**
     * Configure and open the location selector.
     *
     * @return {void}
     */
    async openLocationSelector() {
        const { zip_code, id } = this.state.selectedWhLocation;
        this.dialog.add(LocationSelectorDialog, {
            isProductPage: true,
            isFrontend: true,
            productId: this.state.productId,
            zipCode: zip_code || this.props.zipCode,
            selectedLocationId: String(id),
            save: async location => {
                this.state.selectedWhLocation = location;
                this.state.inStoreStock = location.additional_data.in_store_stock;
                const jsonLocation = JSON.stringify(location);
                // Set the in-store delivery method and the selected pickup location on the order.
                await rpc(
                    '/shop/set_click_and_collect_location', { pickup_location_data: jsonLocation }
                );
            },
        });
    }

}

registry.category('public_components').add(
    'website_sale_collect.ClickAndCollectAvailability', ClickAndCollectAvailability
);

```

## File: static\src\js\click_and_collect_availability\click_and_collect_availability.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">

    <t t-name="website_sale_collect.ClickAndCollectAvailability">
        <div
            t-on-click="openLocationSelector"
            class="o_click_and_collect_availability btn d-flex align-items-center gap-3 border border-1 mb-3 text-start"
            t-att-class="{'disabled': !this.state.active}"
        >
            <div>
                <strong>
                    <i class="fa fa-map-marker me-2 text-muted"/>
                    <t
                        t-if="this.state.selectedWhLocation.id"
                        t-out="this.state.selectedWhLocation.name"
                    />
                    <t t-else="">Pick up in store</t>
                </strong>
                <div t-if="!this.state.selectedWhLocation.id" class="text-muted">
                    Check availability
                </div>
                <t t-elif="!!this.state.inStoreStock?.in_stock">
                    <div t-if="!!this.state.inStoreStock.show_quantity" class="text-warning">
                        <i class="fa fa-circle"/>
                        Only <t t-out="this.state.inStoreStock.quantity"/> available
                    </div>
                    <div t-else="" class="text-success">
                        <i class="fa fa-circle"/> Available
                    </div>
                </t>
                <div t-else="" class="text-danger">
                    <i class="fa fa-times"/> Not available
                </div>
            </div>
            <i class="oi oi-chevron-right"/>
        </div>
    </t>

</templates>

```

## File: static\src\js\location_selector\location\location.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">

    <t t-inherit="delivery.locationSelector.location" t-inherit-mode="extension">
        <small name="location_opening_hours" position="before">
            <t t-if="props.additionalData and !!props.additionalData.in_store_stock">
                <t t-if="!!props.additionalData.in_store_stock.in_stock">
                    <t t-if="!!props.additionalData.in_store_stock.show_quantity">
                        <span class="text-warning">
                            <i class="fa fa-circle"/>
                            Only <t t-out="props.additionalData.in_store_stock.quantity"/> available
                        </span>
                    </t>
                    <t t-else="">
                        <span class="text-success">
                            <i class="fa fa-circle"/> Available
                        </span>
                    </t>
                </t>
                <t t-else="">
                    <span class="text-danger">
                        <i class="fa fa-times"/> Out of stock
                    </span>
                </t>
            </t>
        </small>
    </t>

</templates>

```

## File: static\src\js\location_selector\location_selector_dialog\location_selector_dialog.js

```javascript
import { rpc } from '@web/core/network/rpc';
import { patch } from '@web/core/utils/patch';

import {
    LocationSelectorDialog
} from '@delivery/js/location_selector/location_selector_dialog/location_selector_dialog';

patch(LocationSelectorDialog, {
    props: {
        ...LocationSelectorDialog.props,
        productId: { type: Number, optional: true },
        isProductPage: { type: Boolean, optional: true },
    },
});

patch(LocationSelectorDialog.prototype, {
    async _getLocations(zip) {
         if (this.props.isProductPage) {
             return rpc(this.getLocationUrl, { zip_code: zip, product_id: this.props.productId });
         }
        else {
            return super._getLocations(...arguments);
         }
    },
});

```

## File: static\src\js\location_selector\location_selector_dialog\location_selector_dialog.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">

    <t t-inherit="delivery.locationSelector.dialog" t-inherit-mode="extension">
        <button id="submit_location_small" position="attributes">
            <attribute
                name="t-att-disabled"
                add="!(selectedLocation?.additional_data?.in_store_stock?.in_stock ?? true)"
                separator=" || "
            />
        </button>
    </t>

</templates>

```

## File: static\src\js\location_selector\map_container\map_container.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">

    <t t-inherit="delivery.locationSelector.mapContainer" t-inherit-mode="extension">
        <button id="submit_location_large" position="attributes">
            <attribute
                name="t-att-disabled"
                add="!(selectedLocation?.additional_data?.in_store_stock?.in_stock ?? true)"
                separator=" || "
            />
        </button>
        <button id="submit_location_medium" position="attributes">
            <attribute
                name="t-att-disabled"
                add="!(selectedLocation?.additional_data?.in_store_stock?.in_stock ?? true)"
                separator=" || "
            />
        </button>
    </t>

</templates>

```

## File: static\src\xml\product_availability.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates>

    <t t-inherit="website_sale_stock.product_availability" t-inherit-mode="extension">
        <div id="out_of_stock_message" position="replace">
            <t t-if="!in_store_stock">$0</t>
        </div>
        <div id="threshold_message" position="attributes">
            <attribute name="t-elif" add="!in_store_stock" separator=" and "/>
        </div>
    </t>

</templates>

```

## File: views\delivery_carrier_views.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<odoo>
    
    <record id="delivery_carrier_form" model="ir.ui.view">
        <field name="name">In-store Delivery Carrier Form</field>
        <field name="inherit_id" ref="delivery.view_delivery_carrier_form"/>
        <field name="model">delivery.carrier</field>
        <field name="arch" type="xml">
            <field name="integration_level" position="attributes">
                <attribute name="invisible" add="delivery_type == 'in_store'" separator=" or "/>
            </field>
            <field name="invoice_policy" position="attributes">
                <attribute name="invisible" add="delivery_type == 'in_store'" separator=" or "/>
            </field>
            <page name="pricing" position="before">
                <page string="Stores" name="warehouses" invisible="delivery_type != 'in_store'">
                    <field name="warehouse_ids">
                        <list create="False">
                            <field name="name"/>
                            <field name="opening_hours"/>
                            <field name="lot_stock_id" groups="stock.group_stock_multi_locations"/>
                            <field name="partner_id"/>
                            <field name="company_id" groups="base.group_multi_company"/>
                        </list>
                    </field>
                </page>
            </page>
        </field>
    </record>

</odoo>

```

## File: views\delivery_form_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="unavailable_products_warning">
        <div name="unavailable_products_warning" class="alert alert-warning mt-2">
            Some of the products are not available at <strong><t t-out="wh_name"/></strong>.
            <div
                t-foreach="unavailable_order_lines"
                t-as="order_line"
                t-attf-class="d-flex m-2 position-relative"
            >
                <div class="d-flex align-items-center gap-2">
                    <a t-att-href="order_line.product_id.website_url">
                        <span
                            t-field="order_line.product_id.image_128"
                            t-options="{'widget': 'image', 'qweb_img_responsive': False, 'class': 'o_image_64_max rounded'}"
                        />
                    </a>
                    <t t-out="order_line.product_id.name"/>
                    <a
                        href='#'
                        class="js_delete_product small"
                        aria-label="Remove from cart"
                        title="Remove from cart"
                    >
                        <i t-att-data-product-id="order_line.product_id.id" class="fa fa-trash"/>
                    </a>
                     ( <t t-out="order_line.shop_warning"/> )
                </div>
             </div>
        </div>
    </template>

    <template id="in_store_delivery_method" inherit_id="website_sale.delivery_method">
        <xpath expr="//t[@t-set='is_pickup_needed']" position="attributes">
            <attribute name="t-value" add="dm.delivery_type=='in_store'" separator=" or "/>
        </xpath>
        <div name="o_pickup_location" position="after">
            <t t-if="dm.delivery_type=='in_store' and order.carrier_id.id==dm.id">
                <t
                    t-if="unavailable_order_lines"
                    t-call="website_sale_collect.unavailable_products_warning"
                >
                    <t t-set="wh_name" t-value="order.pickup_location_data.get('name')"/>
                </t>
            </t>
        </div>
    </template>

</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="res_config_settings_form" model="ir.ui.view">
        <field name="name">Click &amp; Collect Settings Form</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="website_sale.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <setting id="click_and_collect_setting" position="inside">
                <div class="mt8">
                    <button
                        string="Configure Pickup Locations"
                        name="action_view_in_store_delivery_methods"
                        type="object"
                        class="btn-link"
                        icon="oi-arrow-right"
                    />
                </div>
            </setting>
        </field>
    </record>

</odoo>

```

## File: views\stock_picking_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="stock_picking_form" model="ir.ui.view">
        <field name="name">stock.picking.form</field>
        <field name="model">stock.picking</field>
        <field name="inherit_id" ref="stock_delivery.view_picking_withcarrier_out_form"/>
        <field name="arch" type="xml">
            <button name="send_to_shipper" position="attributes">
                <attribute name="invisible" add="delivery_type == 'in_store'" separator=" or "/>
            </button>
        </field>
    </record>
</odoo>

```

## File: views\stock_warehouse_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="stock_warehouse_form" model="ir.ui.view">
        <field name="name">Click &amp; Collect Stock Warehouse Form</field>
        <field name="model">stock.warehouse</field>
        <field name="inherit_id" ref="stock.view_warehouse"/>
        <field name="arch" type="xml">
            <field name="partner_id" position="after">
                <field name="opening_hours"/>
            </field>
        </field>
    </record>

</odoo>

```

## File: views\templates.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<odoo>

    <template
        id="payment_confirmation_status"
        inherit_id="website_sale.payment_confirmation_status"
    >
        <xpath expr="(//div[hasclass('card-body')])[1]" position="replace">
            <t t-if="tx_sudo.provider_id.custom_mode == 'on_site'">
                <div class="card-body">
                    <div class="o_header_carrier_message">
                        <b t-out="order.carrier_id.name"/>
                        <span class="text-muted"> (In-store pickup)</span>
                    </div>
                    <div class="o_body_carrier_message">
                        <t t-out="order.carrier_id.website_description"/>
                    </div>
                </div>
            </t>
            <t t-else="">
                <t>$0</t> <!-- Replaced by old content. -->
            </t>
        </xpath>
    </template>

    <template id="product_page_click_and_collect" inherit_id="website_sale.product">
        <xpath expr="//div[@id='o_wsale_cta_wrapper']" position="before">
            <owl-component
                t-if="combination_info.get('show_click_and_collect_availability')"
                name="website_sale_collect.ClickAndCollectAvailability"
                t-att-props="json.dumps({
                    'productId': product_variant.id,
                    'zipCode': zip_code,
                    'selectedWhLocation': selected_wh_location or {},
                    'inStoreStock': combination_info['in_store_stock'],
                })"
                class="d-flex o_not_editable"
            />
        </xpath>
    </template>

</odoo>

```

