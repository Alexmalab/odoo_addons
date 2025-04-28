# Odoo Module: sale_gelato

Category: Sales/Sales

This file contains the source code of the Odoo module.

## File: const.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

COUNTRIES_WITHOUT_ZIPCODE = [
    'AO', 'AG', 'AW', 'BS', 'BZ', 'BJ', 'BW', 'BF', 'BI', 'CM', 'CF', 'KM',
    'CG', 'CD', 'CK', 'CI', 'DJ', 'DM', 'GQ', 'ER', 'FJ', 'TF', 'GM', 'GH',
    'GD', 'GN', 'GY', 'HK', 'IE', 'JM', 'KE', 'KI', 'MO', 'MW', 'ML', 'MR',
    'MU', 'MS', 'NR', 'AN', 'NU', 'KP', 'PA', 'QA', 'RW', 'KN', 'LC', 'ST',
    'SC', 'SL', 'SB', 'SO', 'ZA', 'SR', 'SY', 'TZ', 'TL', 'TK', 'TO', 'TT',
    'TV', 'UG', 'AE', 'VU', 'YE', 'ZW'
]

```

## File: utils.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

import requests

from odoo import _
from odoo.exceptions import UserError


_logger = logging.getLogger(__name__)


def make_request(api_key, subdomain, version, endpoint, payload=None, method='POST'):
    """ Make a request to the Gelato API and return the JSON-formatted content of the response.

    :param str api_key: The Gelato API key used for signing requests.
    :param str subdomain: The subdomain of the Gelato API.
    :param str version: The version of the Gelato API.
    :param str endpoint: The API endpoint to call.
    :param dict payload: The payload of the request.
    :param str method: The HTTP method of the request.
    :return: The JSON-formatted content of the response.
    :rtype: dict
    """
    url = f'https://{subdomain}.gelatoapis.com/{version}/{endpoint}'
    headers = {
        'X-API-KEY': api_key or None
    }
    try:
        if method == 'GET':
            response = requests.get(url=url, params=payload, headers=headers, timeout=10)
        else:
            response = requests.post(url=url, json=payload, headers=headers, timeout=10)
        response_content = response.json()
        try:
            response.raise_for_status()
        except requests.exceptions.HTTPError:
            _logger.exception("Invalid API request at %s with data %s", url, payload)
            raise UserError(response_content.get('message', ''))
    except (requests.exceptions.ConnectionError, requests.exceptions.Timeout):
        _logger.exception("Unable to reach endpoint at %s", url)
        raise UserError(_("Could not establish the connection to the Gelato API."))
    return response.json()

```

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controlers
from . import models
from . import wizards

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': "Gelato",
    'summary': "Place orders through Gelato's print-on-demand service",
    'category': 'Sales/Sales',
    'depends': ['sale_management', 'delivery'],
    'data': [
        'data/product_data.xml',
        'data/delivery_carrier_data.xml',  # Depends on product_data.xml
        'data/mail_template_data.xml',

        'views/delivery_carrier_views.xml',
        'views/product_document_views.xml',
        'views/product_product_views.xml',
        'views/product_template_views.xml',
        'wizards/res_config_settings_views.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: controlers\main.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import hmac
import logging
import pprint

from werkzeug.exceptions import Forbidden

from odoo import SUPERUSER_ID, _
from odoo.http import Controller, request, route


_logger = logging.getLogger(__name__)


class GelatoController(Controller):
    _webhook_url = '/gelato/webhook'

    @route(_webhook_url, type='http', methods=['POST'], auth='public', csrf=False)
    def gelato_webhook(self):
        """ Process the notification data sent by Gelato to the webhook.

        See https://dashboard.gelato.com/docs/orders/order_details/#order-statuses for the event
        codes.

        :return: An empty response to acknowledge the notification.
        :rtype: odoo.http.Response
        """
        event_data = request.get_json_data()
        _logger.info("Webhook notification received from Gelato:\n%s", pprint.pformat(event_data))

        if event_data['event'] == 'order_status_updated':
            # Check the signature of the webhook notification.
            order_id = int(event_data['orderReferenceId'])
            order_sudo = request.env['sale.order'].sudo().browse(order_id).exists()
            received_signature = request.httprequest.headers.get('signature', '')
            self._verify_notification_signature(received_signature, order_sudo)

            # Process the event.
            fulfillment_status = event_data.get('fulfillmentStatus')
            if fulfillment_status == 'failed':
                # Log a message on the order.
                log_message = _(
                    "Gelato could not proceed with the fulfillment of order %(order_reference)s:"
                    " %(gelato_message)s",
                    order_reference=order_sudo.display_name,
                    gelato_message=event_data['comment'],
                )
                order_sudo.message_post(
                    body=log_message, author_id=request.env.ref('base.partner_root').id
                )
            elif fulfillment_status == 'canceled':
                # Cancel the order.
                order_sudo.with_user(SUPERUSER_ID)._action_cancel()

                # Manually cache the currency while in a sudoed environment to prevent an
                # AccessError. The state of the sales order is a dependency of
                # `untaxed_amount_to_invoice`, which is a monetary field. They require the currency
                # to ensure the values are saved in the correct format. However, the currency cannot
                # be read directly during the flush due to access rights, necessitating manual
                # caching.
                order_sudo.order_line.currency_id

                # Send the generic order cancellation email.
                order_sudo.message_post_with_source(
                    source_ref=request.env.ref('sale.mail_template_sale_cancellation'),
                    author_id=request.env.ref('base.partner_root').id,
                )
            elif fulfillment_status == 'in_transit':
                # Send the Gelato order status update email.
                tracking_data = self._extract_tracking_data(item_data=event_data['items'])
                order_sudo.with_context({'tracking_data': tracking_data}).message_post_with_source(
                    source_ref=request.env.ref('sale_gelato.order_status_update'),
                    author_id=request.env.ref('base.partner_root').id,
                )
            elif fulfillment_status == 'delivered':
                # Send the Gelato order status update email.
                order_sudo.with_context({'order_delivered': True}).message_post_with_source(
                    source_ref=request.env.ref('sale_gelato.order_status_update'),
                    author_id=request.env.ref('base.partner_root').id,
                )
            elif fulfillment_status == 'returned':
                # Log a message on the order.
                log_message = _(
                    "Gelato has returned order %(reference)s.", reference=order_sudo.display_name
                )
                order_sudo.message_post(
                    body=log_message, author_id=request.env.ref('base.partner_root').id
                )
        return request.make_json_response('')

    @staticmethod
    def _verify_notification_signature(received_signature, order_sudo):
        """ Check if the received signature matches the expected one.

        :param str received_signature: The received signature.
        :param sale.order order_sudo: The sales order for which the webhook notification was sent.
        :return: None
        :raise Forbidden: If the signatures don't match.
        """
        company_sudo = order_sudo.company_id.sudo()  # In sudo mode to read on the company.
        expected_signature = company_sudo.gelato_webhook_secret
        if not hmac.compare_digest(received_signature, expected_signature):
            _logger.warning("Received notification with invalid signature.")
            raise Forbidden()

    @staticmethod
    def _extract_tracking_data(item_data):
        """ Extract the tracking URL and code from the item data.

        :param dict item_data: The item data.
        :return: The extracted tracking data.
        :rtype: dict
        """
        tracking_data = {}
        for i in item_data:
            for fulfilment_data in i['fulfillments']:
                tracking_data.setdefault(
                    fulfilment_data['trackingUrl'], fulfilment_data['trackingCode']
                )  # Different items can have the same tracking URL.
        return tracking_data

```

## File: controlers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\delivery_carrier_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="standard_delivery" model="delivery.carrier">
        <field name="name">Standard Delivery</field>
        <field name="delivery_type">gelato</field>
        <field name="integration_level">rate</field>
        <field name="gelato_shipping_service_type">normal</field>
        <field name="product_id" ref="sale_gelato.standard_delivery_product"/>
        <field name="prod_environment" eval="True"/>
    </record>

    <record id="express_delivery" model="delivery.carrier">
        <field name="name">Express Delivery</field>
        <field name="delivery_type">gelato</field>
        <field name="integration_level">rate</field>
        <field name="gelato_shipping_service_type">express</field>
        <field name="product_id" ref="sale_gelato.express_delivery_product"/>
        <field name="prod_environment" eval="True"/>
    </record>

</odoo>

```

## File: data\mail_template_data.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<odoo noupdate="1">

    <record id="order_status_update" model="mail.template">
        <field name="name">Gelato: Order status update</field>
        <field name="model_id" ref="model_sale_order"/>
        <field name="subject">{{ object.reference }}</field>
        <field name="partner_to">{{ object.partner_id.email and object.partner_id.id or object.partner_id.parent_id.id }}</field>
        <field name="description">Sent to the customer when Gelato updates the status of an order</field>
        <field name="body_html" type="html">
            <div style="margin: 0px; padding: 0px;">
                <p style="margin: 0px; padding: 0px; font-size: 13px;">
                    Hello <t t-out="object.partner_id.name or ''">Brandon Freeman</t>,<br/><br/>
                    <!-- Order in transit body -->
                    <t t-if="ctx.get('tracking_data')">
                        We are glad to inform you that your order is in transit.
                        <t t-if="len(ctx['tracking_data']) == 1">
                            <t t-set="tracking_url" t-value="list(ctx['tracking_data'].keys())[0]"/>
                            Your tracking number is <a t-attf-href="tracking_url" t-out="ctx['tracking_data'][tracking_url]"/>.
                            <br/><br/>
                        </t>
                        <t t-else="">
                            Your tracking numbers are:
                            <ul>
                                <li t-foreach="ctx['tracking_data']" t-as="tracking_url">
                                    <a t-attf-href="{{tracking_url}}" t-out="ctx['tracking_data'][tracking_url]"/>
                                </li>
                            </ul>
                        </t>
                    </t>
                    <!-- Order delivered body -->
                    <t t-if="ctx.get('order_delivered')">
                        We are glad to inform you that your order has been delivered.
                        <br/><br/>
                    </t>
                    Thank you,
                    <t t-if="object.user_id.name">
                        <br />
                        <t t-out="object.user_id.name or ''">--<br/>Mitchell Admin</t>
                    </t>
                </p>
            </div>
        </field>
        <field name="lang">{{ object.partner_id.lang }}</field>
        <field name="auto_delete" eval="True"/>
    </record>

</odoo>

```

## File: data\neutralize.sql

```sql
-- disable Gelato
UPDATE res_company
   SET gelato_api_key = NULL,
       gelato_webhook_secret = NULL;

```

## File: data\product_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="standard_delivery_product" model="product.product">
        <field name="name">Standard Delivery (Gelato)</field>
        <field name="default_code">normal</field>
        <field name="type">service</field>
        <field name="categ_id" ref="delivery.product_category_deliveries"/>
        <field name="sale_ok" eval="False"/>
        <field name="purchase_ok" eval="False"/>
        <field name="list_price">0.0</field>
    </record>

    <record id="express_delivery_product" model="product.product">
        <field name="name">Express Delivery (Gelato)</field>
        <field name="default_code">express</field>
        <field name="type">service</field>
        <field name="categ_id" ref="delivery.product_category_deliveries"/>
        <field name="sale_ok" eval="False"/>
        <field name="purchase_ok" eval="False"/>
        <field name="list_price">0.0</field>
    </record>

</odoo>

```

## File: models\delivery_carrier.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.exceptions import UserError

from odoo.addons.sale_gelato import const, utils


class ProviderGelato(models.Model):
    _inherit = 'delivery.carrier'

    delivery_type = fields.Selection(
        selection_add=[('gelato', "Gelato")], ondelete={'gelato': 'cascade'}
    )
    gelato_shipping_service_type = fields.Selection(
        string="Gelato Shipping Service Type",
        selection=[('normal', "Standard Delivery"), ('express', "Express Delivery")],
        required=True,
        default='normal',
    )

    # === BUSINESS METHODS === #

    def _is_available_for_order(self, order):
        """ Override of `delivery` to exclude regular delivery methods from Gelato orders and Gelato
        delivery methods from non-Gelato orders.

        :param sale.order order: The current order.
        :return: Whether the delivery method is available for the order.
        :rtype: bool
        """
        is_gelato_order = any(order.order_line.product_id.mapped('gelato_product_uid'))
        is_gelato_delivery = self.delivery_type == 'gelato'
        if is_gelato_order and not is_gelato_delivery or not is_gelato_order and is_gelato_delivery:
            return False
        return super()._is_available_for_order(order)

    def available_carriers(self, partner, order):
        """ Override of `delivery` to filter out regular delivery methods from Gelato orders and
        Gelato delivery methods from non-Gelato orders.

        :param res.partner partner: The partner to check.
        :param sale.order order: The current order.
        :return: The available delivery methods.
        :rtype: delivery.carrier
        """
        available_delivery_methods = super().available_carriers(partner, order)
        is_gelato_order = any(order.order_line.product_id.mapped('gelato_product_uid'))
        if is_gelato_order:
            return available_delivery_methods.filtered(lambda m: m.delivery_type == 'gelato')
        else:
            return available_delivery_methods.filtered(lambda m: m.delivery_type != 'gelato')

    def gelato_rate_shipment(self, order):
        """ Fetch the Gelato delivery price based on products, quantity and address.

        This method is called by `delivery`'s `rate_shipment` method.

        Note: `self._ensure_one()` from `rate_shipment`

        :param sale.order order: The order for which to fetch the delivery price.
        :return: The shipment rate request results.
        :rtype: dict
        """
        if error_message := self._ensure_partner_address_is_complete(order.partner_id):
            return {
                'success': False,
                'price': 0,
                'error_message': error_message,
            }

        # Fetch the delivery price from Gelato.
        payload = {
            'orderReferenceId': order.id,
            'customerReferenceId': f'Odoo Partner #{order.partner_id.id}',
            'currency': order.currency_id.name,
            'allowMultipleQuotes': 'true',
            'products': order._gelato_prepare_items_payload(),
            'recipient': order.partner_shipping_id._gelato_prepare_address_payload(),
        }
        try:
            api_key = order.company_id.sudo().gelato_api_key  # In sudo mode to read on the company.
            order_data = utils.make_request(api_key, 'order', 'v4', 'orders:quote', payload=payload)
        except UserError as e:
            return {
                'success': False,
                'price': 0,
                'error_message': str(e),
            }

        # Find the total delivery price by summing all products' matching methods' minimum price.
        total_delivery_price = 0
        for quote_data in order_data['quotes']:
            matching_shipment_method_prices = [
                shipment_method_data['price']
                for shipment_method_data in quote_data['shipmentMethods']
                if shipment_method_data['type'] == self.gelato_shipping_service_type
            ]
            if not matching_shipment_method_prices:
                return {
                    'success': False,
                    'price': 0,
                    'error_message': _("The delivery method is not available for this order."),
                }
            else:
                total_delivery_price += min(matching_shipment_method_prices)

        return {
            'success': True,
            'price': total_delivery_price,
        }

    @api.model
    def _ensure_partner_address_is_complete(self, partner):
        """ Ensure that all partner address fields required by Gelato are set.

        :param res.partner partner: The partner address to check.
        :return: An error message if the address is incomplete, None otherwise.
        :rtype: str | None
        """
        required_address_fields = ['city', 'country_id', 'street']
        if partner.country_id.code not in const.COUNTRIES_WITHOUT_ZIPCODE:
            required_address_fields.append('zip')
        missing_fields = [
            partner._fields[field_name]
            for field_name in required_address_fields if not partner[field_name]
        ]
        if missing_fields:
            translated_field_names = [f._description_string(self.env) for f in missing_fields]
            return _(
                "The following required address fields are missing: %s",
                ", ".join(translated_field_names),
            )

```

## File: models\product_document.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, fields, models
from odoo.exceptions import UserError


class ProductDocument(models.Model):

    _inherit = 'product.document'

    # Technical field to tell apart Gelato print images from other product documents.
    is_gelato = fields.Boolean(readonly=True)

    def _gelato_prepare_file_payload(self):
        """ Create the payload for a single file of an 'orders' request.

        :return: The file payload.
        :rtype: dict
        """
        if not self.datas:
            raise UserError(_("Print images must be set on products before they can be ordered."))

        query_string = f'access_token={self.ir_attachment_id.generate_access_token()[0]}'
        url = f'{self.get_base_url()}{self.ir_attachment_id.image_src}?{query_string}'
        return {
            'type': self.name.lower(),  # Gelato requires lowercase types.
            'url': url,
        }

```

## File: models\product_product.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ProductProduct(models.Model):
    _inherit = 'product.product'

    gelato_product_uid = fields.Char(name="Gelato Product UID", readonly=True)

```

## File: models\product_template.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import Command, _, api, fields, models
from odoo.exceptions import UserError
from odoo.osv import expression

from odoo.addons.sale_gelato import utils


class ProductTemplate(models.Model):
    _inherit = 'product.template'

    gelato_template_ref = fields.Char(
        string="Gelato Template Reference", help="Synchronize to fetch variants from Gelato",
    )
    gelato_product_uid = fields.Char(
        string="Gelato Product UID",
        compute='_compute_gelato_product_uid',
        inverse='_inverse_gelato_product_uid',
        readonly=True,
    )
    gelato_image_ids = fields.One2many(
        string="Gelato Print Images",
        comodel_name='product.document',
        inverse_name='res_id',
        domain=[('is_gelato', '=', True)],
        readonly=True,
    )
    gelato_missing_images = fields.Boolean(
        string="Missing Print Images", compute='_compute_gelato_missing_images',
    )

    # === COMPUTE METHODS === #

    @api.depends('product_variant_ids.gelato_product_uid')
    def _compute_gelato_product_uid(self):
        self._compute_template_field_from_variant_field('gelato_product_uid')

    def _inverse_gelato_product_uid(self):
        self._set_product_variant_field('gelato_product_uid')

    @api.depends('gelato_image_ids')
    def _compute_gelato_missing_images(self):
        for product in self:
            product.gelato_missing_images = any(
                not image.datas for image in product.gelato_image_ids
            )

    # === ACTION METHODS === #

    def action_sync_gelato_template_info(self):
        """ Fetch the template information from Gelato and update the product template accordingly.

        :return: The action to display a toast notification to the user.
        :rtype: dict
        """
        # Fetch the template info from Gelato.
        try:
            endpoint = f'templates/{self.gelato_template_ref}'
            template_info = utils.make_request(
                self.env.company.sudo().gelato_api_key, 'ecommerce', 'v1', endpoint, method='GET'
            )  # In sudo mode to read the API key from the company.
        except UserError as e:
            return {
                'type': 'ir.actions.client',
                'tag': 'display_notification',
                'params': {
                    'type': 'danger',
                    'title': _("Could not synchronize with Gelato"),
                    'message': str(e),
                    'sticky': True,
                }
            }

        # Apply the necessary changes on the product template.
        self._create_attributes_from_gelato_info(template_info)
        self._create_print_images_from_gelato_info(template_info)

        # Display a toaster notification to the user if all went well.
        return {
            'type': 'ir.actions.client',
            'tag': 'display_notification',
            'params': {
                'type': 'success',
                'title': _("Successfully synchronized with Gelato"),
                'message': _("Missing product variants and images have been successfully created."),
                'sticky': False,
                'next': {
                    'type': 'ir.actions.client',
                    'tag': 'soft_reload'
                }
            }
        }

    # === BUSINESS METHODS === #

    def _create_attributes_from_gelato_info(self, template_info):
        """ Create attributes for the current product template.
        
        :param dict template_info: The template information fetched from Gelato.
        :return: None
        """
        if len(template_info['variants']) == 1:  # The template has no attribute.
            self.gelato_product_uid = template_info['variants'][0]['productUid']
        else:  # The template has multiple attributes.
            # Iterate over the variants to find and create the possible attributes.
            for variant_data in template_info['variants']:
                current_variant_pavs = self.env['product.attribute.value']
                for attribute_data in variant_data['variantOptions']:  # Attribute name and value.
                    # Search for the existing attribute with the proper variant creation policy and
                    # create it if not found.
                    attribute = self.env['product.attribute'].search(
                        [('name', '=', attribute_data['name']), ('create_variant', '=', 'always')],
                        limit=1,
                    )
                    if not attribute:
                        attribute = self.env['product.attribute'].create({
                            'name': attribute_data['name']
                        })

                    # Search for the existing attribute value and create it if not found.
                    attribute_value = self.env['product.attribute.value'].search([
                        ('name', '=', attribute_data['value']),
                        ('attribute_id', '=', attribute.id),
                    ], limit=1)
                    if not attribute_value:
                        attribute_value = self.env['product.attribute.value'].create({
                            'name': attribute_data['value'],
                            'attribute_id': attribute.id
                        })
                    current_variant_pavs += attribute_value

                    # Search for the existing PTAL and create it if not found.
                    ptal = self.env['product.template.attribute.line'].search(
                        [('product_tmpl_id', '=', self.id), ('attribute_id', '=', attribute.id)],
                        limit=1,
                    )
                    if not ptal:
                        self.env['product.template.attribute.line'].create({
                            'product_tmpl_id': self.id,
                            'attribute_id': attribute.id,
                            'value_ids': [Command.link(attribute_value.id)]
                        })
                    else:  # The PTAL already exists.
                        ptal.value_ids = [Command.link(attribute_value.id)]  # Link the value.

                # Find the variant that was automatically created and set the Gelato UID.
                for variant in self.product_variant_ids:
                    corresponding_ptavs = variant.product_template_attribute_value_ids
                    corresponding_pavs = corresponding_ptavs.product_attribute_value_id
                    if corresponding_pavs == current_variant_pavs:
                        variant.gelato_product_uid = variant_data['productUid']
                        break

            # Delete the incompatible variants that were created but not allowed by Gelato.
            variants_without_gelato = self.env['product.product'].search([
                ('product_tmpl_id', '=', self.id),
                ('gelato_product_uid', '=', False)
            ])
            variants_without_gelato.unlink()

    def _create_print_images_from_gelato_info(self, template_info):
        """ Create print image for the current product template.

        :param dict template_info: The template information fetched from Gelato.
        :return: None
        """
        # Iterate over the print image data listed in the info of the first variant, as we don't
        # support varying image placements between variants.
        for print_image_data in template_info['variants'][0]['imagePlaceholders']:
            # Gelato might send image placements that are named '1' or 'front' that are not accepted
            # by their API when placing order.
            if print_image_data['printArea'].lower() in ('1', 'front'):
                print_image_data['printArea'] = 'default'  # Use 'default' which is accepted.

            # Gelato might send several print images for the same placement if several layers were
            # defined, but we keep only one because their API only accepts one image per placement.
            print_image_found = bool(self.env['product.document'].search_count([
                ('name', 'ilike', print_image_data['printArea']),
                ('res_id', '=', self.id),
                ('res_model', '=', 'product.template'),
                ('is_gelato', '=', True),  # Avoid finding regular documents with the same name.
            ]))
            if not print_image_found:
                self.gelato_image_ids = [Command.create({
                    'name': print_image_data['printArea'].lower(),
                    'res_id': self.id,
                    'res_model': 'product.template',
                    'is_gelato': True,
                })]

    # === GETTER METHODS === #

    def _get_related_fields_variant_template(self):
        """ Override of `product` to add `gelato_product_uid` as a related field. """
        return super()._get_related_fields_variant_template() + ['gelato_product_uid']

    def _get_product_document_domain(self):
        """ Override of `product` to filter out gelato print images. """
        domain = super()._get_product_document_domain()
        return expression.AND([domain, [('is_gelato', '=', False)]])

```

## File: models\res_company.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResCompany(models.Model):
    _inherit = 'res.company'

    gelato_api_key = fields.Char(string="Gelato API Key")
    gelato_webhook_secret = fields.Char(string="Gelato Webhook Secret")

```

## File: models\res_partner.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models

from odoo.addons.payment import utils as payment_utils


class ResPartner(models.Model):
    _inherit = 'res.partner'

    def _gelato_prepare_address_payload(self):
        first_name, last_name = payment_utils.split_partner_name(self.name)
        return {
            'companyName': self.commercial_company_name or '',
            'firstName': first_name or last_name,  # Gelato require a first name.
            'lastName': last_name,
            'addressLine1': self.street,
            'addressLine2': self.street2 or '',
            'state': self.state_id.code,
            'city': self.city,
            'postCode': self.zip,
            'country': self.country_id.code,
            'email': self.email,
            'phone': self.phone or ''
        }

```

## File: models\sale_order.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import pprint

from odoo import _, models
from odoo.exceptions import UserError, ValidationError

from odoo.addons.sale_gelato import utils


_logger = logging.getLogger(__name__)


class SaleOrder(models.Model):
    _inherit = 'sale.order'

    # === CRUD METHODS === #

    def _prevent_mixing_gelato_and_non_gelato_products(self):
        """ Ensure that the order lines don't mix Gelato and non-Gelato products.

        This method is not a constraint and is called from the `create` and `write` methods of
        `sale.order.line` to cover the cases where adding/writing on order lines would not trigger a
        constraint check (e.g., adding products through the Catalog).

        :return: None
        :raise ValidationError: If Gelato and non-Gelato products are mixed.
        """
        for order in self:
            gelato_lines = order.order_line.filtered(lambda l: l.product_id.gelato_product_uid)
            non_gelato_lines = (order.order_line - gelato_lines).filtered(
                lambda l: l.product_id.sale_ok and l.product_id.type != 'service'
            )  # Filter out non-saleable (sections, etc.) and non-deliverable products.
            if gelato_lines and non_gelato_lines:
                raise ValidationError(
                    _("You cannot mix Gelato products with non-Gelato products in the same order."))

    # === ACTION METHODS === #

    def action_open_delivery_wizard(self):
        """ Override of `delivery` to set a Gelato delivery method by default in the wizard. """
        res = super().action_open_delivery_wizard()

        if (
            not self.env.context.get('carrier_recompute')
            and any(line.product_id.gelato_product_uid for line in self.order_line)
        ):
            gelato_delivery_method = self.env['delivery.carrier'].search(
                [('delivery_type', '=', 'gelato')], limit=1
            )
            res['context']['default_carrier_id'] = gelato_delivery_method.id
        return res

    def action_confirm(self):
        """ Override of `sale` to send the order to Gelato on confirmation. """
        res = super().action_confirm()
        for order in self.filtered(
            lambda o: any(o.order_line.product_id.mapped('gelato_product_uid'))
        ):
            order._create_order_on_gelato()
        return res

    # === BUSINESS METHODS === #

    def _create_order_on_gelato(self):
        """ Send the order creation request to Gelato and log the request result on the chatter.

        :return: None
        """
        delivery_line = self.order_line.filtered(
            lambda l: l.is_delivery and l.product_id.default_code in ('normal', 'express')
        )
        payload = {
            'orderType': 'order',
            'orderReferenceId': self.id,
            'customerReferenceId': f'Odoo Partner #{self.partner_id.id}',
            'currency': self.currency_id.name,
            'items': self._gelato_prepare_items_payload(),
            'shipmentMethodUid': delivery_line.product_id.default_code or 'cheapest',
            'shippingAddress': self.partner_shipping_id._gelato_prepare_address_payload(),
        }
        try:
            api_key = self.company_id.sudo().gelato_api_key  # In sudo mode to read on the company.
            data = utils.make_request(api_key, 'order', 'v4', 'orders', payload=payload)
        except UserError as e:
            raise UserError(_(
                "The order with reference %(order_reference)s was not sent to Gelato.\n"
                "Reason: %(error_message)s",
                order_reference=self.display_name,
                error_message=str(e),
            ))

        _logger.info("Notification received from Gelato with data:\n%s", pprint.pformat(data))
        self.message_post(
            body=_("The order has been successfully passed on Gelato."),
            author_id=self.env.ref('base.partner_root').id,
        )

    def _gelato_prepare_items_payload(self):
        """ Create the payload for the 'items' key of an 'orders' request.

        :return: The items payload.
        :rtype: dict
        """
        items_payload = []
        for gelato_line in self.order_line.filtered(lambda l: l.product_id.gelato_product_uid):
            item_data = {
                'itemReferenceId': gelato_line.product_id.id,
                'productUid': gelato_line.product_id.gelato_product_uid,
                'files': [
                    image._gelato_prepare_file_payload()
                    for image in gelato_line.product_id.product_tmpl_id.gelato_image_ids
                ],
                'quantity': int(gelato_line.product_uom_qty),
            }
            items_payload.append(item_data)
        return items_payload

```

## File: models\sale_order_line.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class SaleOrderLine(models.Model):
    _inherit = 'sale.order.line'

    # === CRUD METHODS === #

    @api.model_create_multi
    def create(self, vals_list):
        order_lines = super().create(vals_list)
        order_lines.order_id._prevent_mixing_gelato_and_non_gelato_products()
        return order_lines

    def write(self, vals):
        res = super().write(vals)
        self.order_id._prevent_mixing_gelato_and_non_gelato_products()
        return res

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import delivery_carrier
from . import product_document
from . import product_product
from . import product_template
from . import res_company
from . import res_partner
from . import sale_order
from . import sale_order_line

```

## File: static\description\icon.svg

```svg
<?xml version="1.0" standalone="no"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 20010904//EN"
 "http://www.w3.org/TR/2001/REC-SVG-20010904/DTD/svg10.dtd">
<svg version="1.0" xmlns="http://www.w3.org/2000/svg"
 width="100.000000pt" height="100.000000pt" viewBox="0 0 100.000000 100.000000"
 preserveAspectRatio="xMidYMid meet">

<g transform="translate(0.000000,100.000000) scale(0.100000,-0.100000)"
fill="#000000" stroke="none">
<path d="M323 857 c-190 -190 -191 -164 17 -372 l160 -160 118 118 118 117 29
-30 29 -31 -147 -147 -147 -147 -188 188 c-177 176 -190 187 -226 187 -48 0
-86 -36 -86 -82 0 -29 24 -56 233 -265 214 -214 235 -233 267 -233 32 0 53 19
267 233 280 279 271 260 161 373 l-71 74 71 74 c82 85 91 114 46 160 -46 45
-75 36 -160 -46 l-74 -72 -104 102 c-142 141 -130 142 -313 -41z m234 -119
c29 -29 53 -57 53 -63 0 -6 -25 -35 -55 -65 l-55 -54 -55 54 c-30 30 -55 59
-55 64 0 10 100 116 109 116 3 0 29 -23 58 -52z"/>
</g>
</svg>

```

## File: views\delivery_carrier_views.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<odoo>

    <record id="delivery_carrier_form" model="ir.ui.view">
        <field name="name">Delivery Carrier Form</field>
        <field name="model">delivery.carrier</field>
        <field name="inherit_id" ref="delivery.view_delivery_carrier_form"/>
        <field name="arch" type="xml">
            <button name="toggle_prod_environment" position="attributes">
                <attribute name="invisible" separator=" or " add="delivery_type == 'gelato'"/>
            </button>
            <button name="toggle_debug" position="attributes">
                <attribute name="invisible" separator=" or " add="delivery_type == 'gelato'"/>
            </button>
            <field name="integration_level" position="attributes">
                <attribute name="invisible" separator=" or " add="delivery_type == 'gelato'"/>
            </field>
            <group name="content" position="attributes">
                <attribute name="invisible" separator=" or " add="delivery_type == 'gelato'"/>
            </group>
        </field>
    </record>

</odoo>

```

## File: views\product_document_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="product_document_form" model="ir.ui.view">
        <field name="name">Product Document Form</field>
        <field name="model">product.document</field>
        <field name="priority" eval="1000"/>
        <field name="arch" type="xml">
            <form>
                <sheet>
                    <field name="datas" widget="image" nolabel="True"/>
                </sheet>
            </form>
        </field>
    </record>

</odoo>

```

## File: views\product_product_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="product_product_normal_form" model="ir.ui.view">
        <field name="name">Product Product Normal Form</field>
        <field name="model">product.product</field>
        <field name="inherit_id" ref="product.product_normal_form_view"></field>
        <field name="arch" type="xml">
            <group name="upsell" position="after">
                <group
                    string="Gelato" invisible="not gelato_product_uid" groups="base.group_no_one"
                >
                    <field string="Product UID" name="gelato_product_uid"/>
                </group>
            </group>
        </field>
    </record>

    <record id="product_product_easy_form" model="ir.ui.view">
        <field name="name">Product Product Easy Form</field>
        <field name="model">product.product</field>
        <field name="inherit_id" ref="product.product_variant_easy_edit_view"></field>
        <field name="arch" type="xml">
            <group name="packaging" position="after">
                <group
                    string="Gelato" invisible="not gelato_product_uid" groups="base.group_no_one"
                >
                    <field string="Product UID" name="gelato_product_uid"/>
                </group>
            </group>
        </field>
    </record>

</odoo>

```

## File: views\product_template_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="product_template_form" model="ir.ui.view">
        <field name="name">Product Template Form</field>
        <field name="model">product.template</field>
        <field name="inherit_id" ref="product.product_template_only_form_view"/>
        <field name="arch" type="xml">
            <xpath
                expr="//page[@name='variants']/field[@name='attribute_line_ids']"
                position="attributes"
            >
                <attribute name="readonly" separator=" or " add="gelato_template_ref"/>
            </xpath>
            <group name="upsell" position="after">
                <group string="Gelato" invisible="type not in ['product', 'consu']">
                    <field
                        string="Template Reference"
                        name="gelato_template_ref"
                        readonly="gelato_product_uid"
                    />
                    <field
                        string="Product UID"
                        name="gelato_product_uid"
                        invisible="product_variant_count &gt; 1"
                        groups="base.group_no_one"
                    />
                    <label
                        for="gelato_image_ids" invisible="not gelato_image_ids" class="opacity-100"
                    />
                    <div invisible="not gelato_image_ids" class="o_row align-items-center">
                        <field
                            string="Print Images"
                            name="gelato_image_ids"
                            widget="many2many_tags"
                            options="{
                                'no_create': True,
                                'no_quick_create': True,
                                'no_open': True,
                                'edit_tags': True,
                            }"
                            context="{'form_view_ref': 'sale_gelato.product_document_form'}"
                            force_save="True"
                        />
                        <i
                            title="All print images are set"
                            invisible="gelato_missing_images"
                            class="fa fa-check-circle text-success fs-3 me-3"
                        />
                        <i
                            title="Some print images are missing"
                            invisible="not gelato_missing_images"
                            class="fa fa-question-circle text-danger fs-3 me-3"
                        />
                    </div>
                    <button
                        string="Synchronize"
                        type="object"
                        name="action_sync_gelato_template_info"
                        invisible="is_product_variant or not gelato_template_ref"
                        colspan="2"
                        class="btn btn-primary"
                    />
                </group>
            </group>
        </field>
    </record>

</odoo>

```

## File: wizards\res_config_settings.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    module_sale_gelato = fields.Boolean("Gelato")

    gelato_api_key = fields.Char(related='company_id.gelato_api_key', readonly=False)
    gelato_webhook_secret = fields.Char(related='company_id.gelato_webhook_secret', readonly=False)

```

## File: wizards\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="res_config_settings_form" model="ir.ui.view">
        <field name="name">Res Config Settings Form</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="sale.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <setting id="amazon_connector" position="after">
                <setting
                    id="gelato"
                    documentation="/applications/sales/sales/gelato.html"
                    help="Place orders through Gelato's print-on-demand service"
                >
                    <field name="module_sale_gelato" widget="upgrade_boolean"/>
                    <div
                        name="gelato_credentials"
                        class="content-group"
                        invisible="not module_sale_gelato"
                    />
                    <div>
                        <label for="gelato_api_key" string="API Key" class="me-3"/>
                        <field name="gelato_api_key" password="True"/>
                    </div>
                    <div>
                        <label for="gelato_webhook_secret" string="Webhook Secret" class="me-3"/>
                        <field name="gelato_webhook_secret" password="True"/>
                    </div>
                    <div class="mt-2">
                        <button
                            string="Manage Delivery Methods"
                            type="action"
                            name="%(delivery.action_delivery_carrier_form)d"
                            icon="oi-arrow-right"
                            class="btn-link"
                            context="{'search_default_delivery_type': 'gelato'}"
                        />
                    </div>
                </setting>
            </setting>
        </field>
    </record>
</odoo>

```

## File: wizards\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_config_settings

```

