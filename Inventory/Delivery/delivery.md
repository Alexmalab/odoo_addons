# Odoo Module: delivery

Category: Inventory/Delivery

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


{
    'name': 'Delivery Costs',
    'version': '1.0',
    'category': 'Inventory/Delivery',
    'description': """
Allows you to add delivery methods in sale orders and picking.
==============================================================

You can define your own carrier for prices. When creating
invoices from picking, the system is able to add and compute the shipping line.
""",
    'depends': ['sale_stock', 'sale_management'],
    'data': [
        'security/ir.model.access.csv',
        'security/delivery_carrier_security.xml',
        'views/product_template_view.xml',
        'views/delivery_view.xml',
        'views/partner_view.xml',
        'views/delivery_portal_template.xml',
        'data/delivery_data.xml',
        'views/report_shipping.xml',
        'views/report_deliveryslip.xml',
        'views/report_package_barcode.xml',
        'views/res_config_settings_views.xml',
        'wizard/choose_delivery_package_views.xml',
        'wizard/choose_delivery_carrier_views.xml',
        'views/stock_package_type_views.xml',
        'views/stock_picking_type_views.xml',
        'views/stock_rule_views.xml',
        'views/stock_move_line_views.xml',
        'report/ir_actions_report_templates.xml',
    ],
    'demo': ['data/delivery_demo.xml'],
    'installable': True,
    'license': 'LGPL-3',
}

```

## File: data\delivery_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="product_category_deliveries" model="product.category">
            <field name="parent_id" ref="product.product_category_all"/>
            <field name="name">Deliveries</field>
        </record>
        <record id="product_product_delivery" model="product.product">
            <field name="name">Free delivery charges</field>
            <field name="default_code">Delivery_007</field>
            <field name="type">service</field>
            <field name="categ_id" ref="delivery.product_category_deliveries"/>
            <field name="sale_ok" eval="False"/>
            <field name="purchase_ok" eval="False"/>
            <field name="list_price">0.0</field>
            <field name="invoice_policy">order</field>
        </record>
        <record id="free_delivery_carrier" model="delivery.carrier">
            <field name="name">Free delivery charges</field>
            <field name="fixed_price">0.0</field>
            <field name="free_over" eval="True"/>
            <field name="amount">1000</field>
            <field name="sequence">1</field>
            <field name="delivery_type">fixed</field>
            <field name="product_id" ref="delivery.product_product_delivery"/>
        </record>
    </data>
</odoo>

```

## File: data\delivery_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <!-- Carrier -->

        <record id="product_product_delivery_poste" model="product.product">
            <field name="name">The Poste</field>
            <field name="default_code">Delivery_009</field>
            <field name="type">service</field>
            <field name="categ_id" ref="delivery.product_category_deliveries"/>
            <field name="sale_ok" eval="False"/>
            <field name="purchase_ok" eval="False"/>
            <field name="list_price">20.0</field>
            <field name="invoice_policy">order</field>
        </record>

        <record id="delivery_carrier" model="delivery.carrier">
            <field name="name">The Poste</field>
            <field name="fixed_price">20.0</field>
            <field name="sequence">2</field>
            <field name="delivery_type">base_on_rule</field>
            <field name="product_id" ref="delivery.product_product_delivery_poste"/>
        </record>

        <!-- Local Delivery -->
        <record id="product_product_local_delivery" model="product.product">
            <field name="name">Local Delivery</field>
            <field name="default_code">Delivery_010</field>
            <field name="type">service</field>
            <field name="categ_id" ref="delivery.product_category_deliveries"/>
            <field name="sale_ok" eval="False"/>
            <field name="purchase_ok" eval="False"/>
            <field name="list_price">10.0</field>
            <field name="invoice_policy">order</field>
        </record>
        <record id="delivery_local_delivery" model="delivery.carrier">
            <field name="name">Local Delivery</field>
            <field name="fixed_price">5.0</field>
            <field name="free_over" eval="True"/>
            <field name="amount">50</field>
            <field name="sequence">4</field>
            <field name="delivery_type">fixed</field>
            <field name="product_id" ref="delivery.product_product_local_delivery"/>
        </record>

        <record id="delivery_price_rule1" model="delivery.price.rule">
            <field name="carrier_id" ref="delivery_carrier"/>
            <field eval="5" name="max_value"/>
            <field eval="20" name="list_base_price"/>
        </record>
        <!--  delivery charge of product if weight more than 5kg-->
        <record id="delivery_price_rule2" model="delivery.price.rule">
            <field name="carrier_id" ref="delivery_carrier"/>
            <field name="operator">&gt;=</field>
            <field eval="5" name="max_value"/>
            <field eval="50" name="list_base_price"/>
        </record>

        <!--  free delivery charge if price more than 300-->
        <record id="delivery_price_rule3" model="delivery.price.rule">
            <field name="carrier_id" ref="delivery_carrier"/>
            <field eval="300" name="max_value"/>
            <field name="operator">&gt;=</field>
            <field name="variable">price</field>
            <field eval="0" name="list_base_price"/>
        </record>

        <record forcecreate="True" id="property_delivery_carrier" model="ir.property">
            <field name="name">property_delivery_carrier_id</field>
            <field name="fields_id" search="[('model','=','res.partner'),('name','=','property_delivery_carrier_id')]"/>
            <field name="value" eval="'delivery.carrier,'+str(ref('delivery_local_delivery'))"/>
        </record>

        <!--Sample sale orders with carrier defined-->
        <record id="outgoing_shipment_with_carrier_confirmed_1" model="stock.picking">
            <field name="picking_type_id" ref="stock.picking_type_out"/>
            <field name="origin">outgoing shipment</field>
            <field name="user_id"></field>
            <field name="carrier_id" ref="delivery_carrier"/>
            <field name="partner_id" ref="base.res_partner_1"/>
            <field name="scheduled_date" eval="DateTime.today() - timedelta(days=3)"/>
            <field name="location_id" ref="stock.stock_location_stock"/>
            <field name="location_dest_id" ref="stock.stock_location_customers"/>
            <field name="move_ids" model="stock.move" eval="[(0, 0, {
                'name': obj().env.ref('product.product_product_6').name,
                'product_id': ref('product.product_product_6'),
                'product_uom': ref('uom.product_uom_unit'),
                'product_uom_qty': 1.0,
                'picking_type_id': ref('stock.picking_type_out'),
                'location_id': ref('stock.stock_location_stock'),
                'location_dest_id': ref('stock.stock_location_customers'),
            })]"/>
        </record>
        
        <record id="outgoing_shipment_with_carrier_assigned_1" model="stock.picking">
            <field name="picking_type_id" ref="stock.picking_type_out"/>
            <field name="origin">outgoing shipment</field>
            <field name="user_id"></field>
            <field name="carrier_id" ref="delivery_carrier"/>
            <field name="partner_id" ref="base.res_partner_1"/>
            <field name="scheduled_date" eval="DateTime.today() - timedelta(days=3)"/>
            <field name="location_id" ref="stock.stock_location_stock"/>
            <field name="location_dest_id" ref="stock.stock_location_customers"/>
            <field name="move_ids" model="stock.move" eval="[(0, 0, {
                'name': obj().env.ref('product.product_product_6').name,
                'product_id': ref('product.product_product_6'),
                'product_uom': ref('uom.product_uom_unit'),
                'product_uom_qty': 1.0,
                'picking_type_id': ref('stock.picking_type_out'),
                'location_id': ref('stock.stock_location_stock'),
                'location_dest_id': ref('stock.stock_location_customers'),
            })]"/>
        </record>
        
        <record id="outgoing_shipment_with_carrier_done_1" model="stock.picking">
            <field name="picking_type_id" ref="stock.picking_type_out"/>
            <field name="origin">outgoing shipment</field>
            <field name="user_id"></field>
            <field name="carrier_id" ref="delivery_carrier"/>
            <field name="partner_id" ref="base.res_partner_1"/>
            <field name="scheduled_date" eval="DateTime.today() - timedelta(days=8)"/>
            <field name="location_id" ref="stock.stock_location_stock"/>
            <field name="location_dest_id" ref="stock.stock_location_customers"/>
            <field name="move_ids" model="stock.move" eval="[(0, 0, {
                'name': obj().env.ref('product.product_product_6').name,
                'product_id': ref('product.product_product_6'),
                'product_uom': ref('uom.product_uom_unit'),
                'product_uom_qty': 1.0,
                'picking_type_id': ref('stock.picking_type_out'),
                'location_id': ref('stock.stock_location_stock'),
                'location_dest_id': ref('stock.stock_location_customers'),
            })]"/>
        </record>
        <function model="stock.picking" name="action_confirm">
            <value model="stock.picking" eval="[
                obj().env.ref('delivery.outgoing_shipment_with_carrier_done_1').id,
                obj().env.ref('delivery.outgoing_shipment_with_carrier_assigned_1').id,
                obj().env.ref('delivery.outgoing_shipment_with_carrier_confirmed_1').id,
            ]"/>
        </function>
        <function model="stock.picking" name="action_assign">
            <value model="stock.picking" eval="[
                obj().env.ref('delivery.outgoing_shipment_with_carrier_done_1').id,
                obj().env.ref('delivery.outgoing_shipment_with_carrier_assigned_1').id,
            ]"/>
        </function>
        <function model="stock.move.line" name="write">
            <value model="stock.move.line" search="[('picking_id', '=', ref('delivery.outgoing_shipment_with_carrier_done_1'))]"/>
            <value eval="{'qty_done': 1}"/>
        </function>
        <function model="stock.picking" name="_action_done">
            <value model="stock.picking" eval="[
                obj().env.ref('delivery.outgoing_shipment_with_carrier_done_1').id,
            ]"/>
        </function>
    </data>
</odoo>

```

## File: data\neutralize.sql

```sql
-- disable prod environment in all delivery carriers
UPDATE delivery_carrier
   SET prod_environment = false;
-- disable delivery carriers from external providers
UPDATE delivery_carrier
   SET active = false
   WHERE delivery_type NOT IN ('fixed', 'base_on_rule');

```

## File: models\delivery_carrier.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import psycopg2
import re

from odoo import api, fields, models, registry, SUPERUSER_ID, _
from odoo.tools.float_utils import float_round
from odoo.tools.misc import groupby
from odoo.exceptions import UserError

from .delivery_request_objects import DeliveryCommodity, DeliveryPackage


class DeliveryCarrier(models.Model):
    _name = 'delivery.carrier'
    _description = "Shipping Methods"
    _order = 'sequence, id'

    ''' A Shipping Provider

    In order to add your own external provider, follow these steps:

    1. Create your model MyProvider that _inherit 'delivery.carrier'
    2. Extend the selection of the field "delivery_type" with a pair
       ('<my_provider>', 'My Provider')
    3. Add your methods:
       <my_provider>_rate_shipment
       <my_provider>_send_shipping
       <my_provider>_get_tracking_link
       <my_provider>_cancel_shipment
       _<my_provider>_get_default_custom_package_code
       (they are documented hereunder)
    '''

    # -------------------------------- #
    # Internals for shipping providers #
    # -------------------------------- #

    name = fields.Char('Delivery Method', required=True, translate=True)
    active = fields.Boolean(default=True)
    sequence = fields.Integer(help="Determine the display order", default=10)
    # This field will be overwritten by internal shipping providers by adding their own type (ex: 'fedex')
    delivery_type = fields.Selection([('fixed', 'Fixed Price')], string='Provider', default='fixed', required=True)
    integration_level = fields.Selection([('rate', 'Get Rate'), ('rate_and_ship', 'Get Rate and Create Shipment')], string="Integration Level", default='rate_and_ship', help="Action while validating Delivery Orders")
    prod_environment = fields.Boolean("Environment", help="Set to True if your credentials are certified for production.")
    debug_logging = fields.Boolean('Debug logging', help="Log requests in order to ease debugging")
    company_id = fields.Many2one('res.company', string='Company', related='product_id.company_id', store=True, readonly=False)
    product_id = fields.Many2one('product.product', string='Delivery Product', required=True, ondelete='restrict')

    invoice_policy = fields.Selection([
        ('estimated', 'Estimated cost'),
        ('real', 'Real cost')
    ], string='Invoicing Policy', default='estimated', required=True,
    help="Estimated Cost: the customer will be invoiced the estimated cost of the shipping.\nReal Cost: the customer will be invoiced the real cost of the shipping, the cost of the shipping will be updated on the SO after the delivery.")

    country_ids = fields.Many2many('res.country', 'delivery_carrier_country_rel', 'carrier_id', 'country_id', 'Countries')
    state_ids = fields.Many2many('res.country.state', 'delivery_carrier_state_rel', 'carrier_id', 'state_id', 'States')
    zip_prefix_ids = fields.Many2many(
        'delivery.zip.prefix', 'delivery_zip_prefix_rel', 'carrier_id', 'zip_prefix_id', 'Zip Prefixes',
        help="Prefixes of zip codes that this carrier applies to. Note that regular expressions can be used to support countries with varying zip code lengths, i.e. '$' can be added to end of prefix to match the exact zip (e.g. '100$' will only match '100' and not '1000')")
    carrier_description = fields.Text(
        'Carrier Description', translate=True,
        help="A description of the delivery method that you want to communicate to your customers on the Sales Order and sales confirmation email."
             "E.g. instructions for customers to follow.")

    margin = fields.Float(help='This percentage will be added to the shipping price.')
    free_over = fields.Boolean('Free if order amount is above', help="If the order total amount (shipping excluded) is above or equal to this value, the customer benefits from a free shipping", default=False)
    amount = fields.Float(string='Amount', help="Amount of the order to benefit from a free shipping, expressed in the company currency")

    can_generate_return = fields.Boolean(compute="_compute_can_generate_return")
    return_label_on_delivery = fields.Boolean(string="Generate Return Label", help="The return label is automatically generated at the delivery.")
    get_return_label_from_portal = fields.Boolean(string="Return Label Accessible from Customer Portal", help="The return label can be downloaded by the customer from the customer portal.")

    supports_shipping_insurance = fields.Boolean(compute="_compute_supports_shipping_insurance")
    shipping_insurance = fields.Integer(
        "Insurance Percentage",
        help="Shipping insurance is a service which may reimburse senders whose parcels are lost, stolen, and/or damaged in transit.",
        default=0
    )

    _sql_constraints = [
        ('margin_not_under_100_percent', 'CHECK (margin >= -100)', 'Margin cannot be lower than -100%'),
        ('shipping_insurance_is_percentage', 'CHECK(shipping_insurance >= 0 AND shipping_insurance <= 100)', "The shipping insurance must be a percentage between 0 and 100."),
    ]

    @api.depends('delivery_type')
    def _compute_can_generate_return(self):
        for carrier in self:
            carrier.can_generate_return = False

    @api.depends('delivery_type')
    def _compute_supports_shipping_insurance(self):
        for carrier in self:
            carrier.supports_shipping_insurance = False

    def toggle_prod_environment(self):
        for c in self:
            c.prod_environment = not c.prod_environment

    def toggle_debug(self):
        for c in self:
            c.debug_logging = not c.debug_logging

    def install_more_provider(self):
        exclude_apps = ['delivery_barcode', 'delivery_stock_picking_batch', 'delivery_iot']
        return {
            'name': _('New Providers'),
            'view_mode': 'kanban,form',
            'res_model': 'ir.module.module',
            'domain': [['name', '=like', 'delivery_%'], ['name', 'not in', exclude_apps]],
            'type': 'ir.actions.act_window',
            'help': _('''<p class="o_view_nocontent">
                    Buy Odoo Enterprise now to get more providers.
                </p>'''),
        }

    def _is_available_for_order(self, order):
        self.ensure_one()
        order.ensure_one()
        if not self._match_address(order.partner_shipping_id):
            return False

        if self.delivery_type == 'base_on_rule':
            return self.rate_shipment(order).get('success')

        return True

    def available_carriers(self, partner):
        return self.filtered(lambda c: c._match_address(partner))

    def _match_address(self, partner):
        self.ensure_one()
        if self.country_ids and partner.country_id not in self.country_ids:
            return False
        if self.state_ids and partner.state_id not in self.state_ids:
            return False
        if self.zip_prefix_ids:
            regex = re.compile('|'.join(['^' + zip_prefix for zip_prefix in self.zip_prefix_ids.mapped('name')]))
            if not partner.zip or not re.match(regex, partner.zip.upper()):
                return False
        return True

    @api.onchange('integration_level')
    def _onchange_integration_level(self):
        if self.integration_level == 'rate':
            self.invoice_policy = 'estimated'

    @api.onchange('can_generate_return')
    def _onchange_can_generate_return(self):
        if not self.can_generate_return:
            self.return_label_on_delivery = False

    @api.onchange('return_label_on_delivery')
    def _onchange_return_label_on_delivery(self):
        if not self.return_label_on_delivery:
            self.get_return_label_from_portal = False

    @api.onchange('state_ids')
    def onchange_states(self):
        self.country_ids = [(6, 0, self.country_ids.ids + self.state_ids.mapped('country_id.id'))]

    @api.onchange('country_ids')
    def onchange_countries(self):
        self.state_ids = [(6, 0, self.state_ids.filtered(lambda state: state.id in self.country_ids.mapped('state_ids').ids).ids)]

    def _get_delivery_type(self):
        """Return the delivery type.

        This method needs to be overridden by a delivery carrier module if the delivery type is not
        stored on the field `delivery_type`.
        """
        self.ensure_one()
        return self.delivery_type

    # -------------------------- #
    # API for external providers #
    # -------------------------- #

    def rate_shipment(self, order):
        ''' Compute the price of the order shipment

        :param order: record of sale.order
        :return dict: {'success': boolean,
                       'price': a float,
                       'error_message': a string containing an error message,
                       'warning_message': a string containing a warning message}
                       # TODO maybe the currency code?
        '''
        self.ensure_one()
        if hasattr(self, '%s_rate_shipment' % self.delivery_type):
            res = getattr(self, '%s_rate_shipment' % self.delivery_type)(order)
            # apply fiscal position
            company = self.company_id or order.company_id or self.env.company
            res['price'] = self.product_id._get_tax_included_unit_price(
                company,
                company.currency_id,
                order.date_order,
                'sale',
                fiscal_position=order.fiscal_position_id,
                product_price_unit=res['price'],
                product_currency=company.currency_id
            )
            # apply margin on computed price
            res['price'] = float(res['price']) * (1.0 + (self.margin / 100.0))
            # save the real price in case a free_over rule overide it to 0
            res['carrier_price'] = res['price']
            # free when order is large enough
            amount_without_delivery = order._compute_amount_total_without_delivery()
            if res['success'] and self.free_over and self._compute_currency(order, amount_without_delivery, 'pricelist_to_company') >= self.amount:
                res['warning_message'] = _('The shipping is free since the order amount exceeds %.2f.') % (self.amount)
                res['price'] = 0.0
            return res

    def send_shipping(self, pickings):
        ''' Send the package to the service provider

        :param pickings: A recordset of pickings
        :return list: A list of dictionaries (one per picking) containing of the form::
                         { 'exact_price': price,
                           'tracking_number': number }
                           # TODO missing labels per package
                           # TODO missing currency
                           # TODO missing success, error, warnings
        '''
        self.ensure_one()
        if hasattr(self, '%s_send_shipping' % self.delivery_type):
            return getattr(self, '%s_send_shipping' % self.delivery_type)(pickings)

    def get_return_label(self,pickings, tracking_number=None, origin_date=None):
        self.ensure_one()
        if self.can_generate_return:
            res = getattr(self, '%s_get_return_label' % self.delivery_type)(pickings, tracking_number, origin_date)
            if self.get_return_label_from_portal:
                pickings.return_label_ids.generate_access_token()
            return res

    def get_return_label_prefix(self):
        return 'ReturnLabel-%s' % self.delivery_type

    def get_tracking_link(self, picking):
        ''' Ask the tracking link to the service provider

        :param picking: record of stock.picking
        :return str: an URL containing the tracking link or False
        '''
        self.ensure_one()
        if hasattr(self, '%s_get_tracking_link' % self.delivery_type):
            return getattr(self, '%s_get_tracking_link' % self.delivery_type)(picking)

    def cancel_shipment(self, pickings):
        ''' Cancel a shipment

        :param pickings: A recordset of pickings
        '''
        self.ensure_one()
        if hasattr(self, '%s_cancel_shipment' % self.delivery_type):
            return getattr(self, '%s_cancel_shipment' % self.delivery_type)(pickings)

    def log_xml(self, xml_string, func):
        self.ensure_one()

        if self.debug_logging:
            self.env.flush_all()
            db_name = self._cr.dbname

            # Use a new cursor to avoid rollback that could be caused by an upper method
            try:
                db_registry = registry(db_name)
                with db_registry.cursor() as cr:
                    env = api.Environment(cr, SUPERUSER_ID, {})
                    IrLogging = env['ir.logging']
                    IrLogging.sudo().create({'name': 'delivery.carrier',
                              'type': 'server',
                              'dbname': db_name,
                              'level': 'DEBUG',
                              'message': xml_string,
                              'path': self.delivery_type,
                              'func': func,
                              'line': 1})
            except psycopg2.Error:
                pass

    def _get_default_custom_package_code(self):
        """ Some delivery carriers require a prefix to be sent in order to use custom
        packages (ie not official ones). This optional method will return it as a string.
        """
        self.ensure_one()
        if hasattr(self, '_%s_get_default_custom_package_code' % self.delivery_type):
            return getattr(self, '_%s_get_default_custom_package_code' % self.delivery_type)()
        else:
            return False

    # ------------------------------------------------ #
    # Fixed price shipping, aka a very simple provider #
    # ------------------------------------------------ #

    fixed_price = fields.Float(compute='_compute_fixed_price', inverse='_set_product_fixed_price', store=True, string='Fixed Price')

    @api.depends('product_id.list_price', 'product_id.product_tmpl_id.list_price')
    def _compute_fixed_price(self):
        for carrier in self:
            carrier.fixed_price = carrier.product_id.list_price

    def _set_product_fixed_price(self):
        for carrier in self:
            carrier.product_id.list_price = carrier.fixed_price

    def fixed_rate_shipment(self, order):
        carrier = self._match_address(order.partner_shipping_id)
        if not carrier:
            return {'success': False,
                    'price': 0.0,
                    'error_message': _('Error: this delivery method is not available for this address.'),
                    'warning_message': False}
        price = order.pricelist_id._get_product_price(self.product_id, 1.0)
        return {'success': True,
                'price': price,
                'error_message': False,
                'warning_message': False}

    def fixed_send_shipping(self, pickings):
        res = []
        for p in pickings:
            res = res + [{'exact_price': p.carrier_id.fixed_price,
                          'tracking_number': False}]
        return res

    def fixed_get_tracking_link(self, picking):
        return False

    def fixed_cancel_shipment(self, pickings):
        raise NotImplementedError()

    # -------------------------------- #
    # get default packages/commodities #
    # -------------------------------- #

    def _get_packages_from_order(self, order, default_package_type):
        packages = []

        total_cost = 0
        for line in order.order_line.filtered(lambda line: not line.is_delivery and not line.display_type):
            total_cost += self._product_price_to_company_currency(line.product_qty, line.product_id, order.company_id)

        total_weight = order._get_estimated_weight() + default_package_type.base_weight
        if total_weight == 0.0:
            weight_uom_name = self.env['product.template']._get_weight_uom_name_from_ir_config_parameter()
            raise UserError(_("The package cannot be created because the total weight of the products in the picking is 0.0 %s") % (weight_uom_name))
        # If max weight == 0 => division by 0. If this happens, we want to have
        # more in the max weight than in the total weight, so that it only
        # creates ONE package with everything.
        max_weight = default_package_type.max_weight or total_weight + 1
        total_full_packages = int(total_weight / max_weight)
        last_package_weight = total_weight % max_weight

        package_weights = [max_weight] * total_full_packages + ([last_package_weight] if last_package_weight else [])
        partial_cost = total_cost / len(package_weights)  # separate the cost uniformly
        order_commodities = self._get_commodities_from_order(order)

        # Split the commodities value uniformly as well
        for commodity in order_commodities:
            commodity.monetary_value /= len(package_weights)
            commodity.qty = max(1, commodity.qty // len(package_weights))

        for weight in package_weights:
            packages.append(DeliveryPackage(order_commodities, weight, default_package_type, total_cost=partial_cost, currency=order.company_id.currency_id, order=order))
        return packages

    def _get_packages_from_picking(self, picking, default_package_type):
        packages = []

        if picking.is_return_picking:
            commodities = self._get_commodities_from_stock_move_lines(picking.move_line_ids)
            weight = picking._get_estimated_weight() + default_package_type.base_weight
            packages.append(DeliveryPackage(commodities, weight, default_package_type, currency=picking.company_id.currency_id, picking=picking))
            return packages

        # Create all packages.
        for package in picking.package_ids:
            move_lines = picking.move_line_ids.filtered(lambda ml: ml.result_package_id == package)
            commodities = self._get_commodities_from_stock_move_lines(move_lines)
            package_total_cost = 0.0
            for quant in package.quant_ids:
                package_total_cost += self._product_price_to_company_currency(quant.quantity, quant.product_id, picking.company_id)
            packages.append(DeliveryPackage(commodities, package.shipping_weight or package.weight, package.package_type_id, name=package.name, total_cost=package_total_cost, currency=picking.company_id.currency_id, picking=picking))

        # Create one package: either everything is in pack or nothing is.
        if picking.weight_bulk:
            commodities = self._get_commodities_from_stock_move_lines(picking.move_line_ids)
            package_total_cost = 0.0
            for move_line in picking.move_line_ids:
                package_total_cost += self._product_price_to_company_currency(move_line.qty_done, move_line.product_id, picking.company_id)
            packages.append(DeliveryPackage(commodities, picking.weight_bulk, default_package_type, name='Bulk Content', total_cost=package_total_cost, currency=picking.company_id.currency_id, picking=picking))
        elif not packages:
            raise UserError(_("The package cannot be created because the total weight of the products in the picking is 0.0 %s") % (picking.weight_uom_name))

        return packages

    def _get_commodities_from_order(self, order):
        commodities = []

        for line in order.order_line.filtered(lambda line: not line.is_delivery and not line.display_type and line.product_id.type in ['product', 'consu']):
            unit_quantity = line.product_uom._compute_quantity(line.product_uom_qty, line.product_id.uom_id)
            rounded_qty = max(1, float_round(unit_quantity, precision_digits=0))
            country_of_origin = line.product_id.country_of_origin.code or order.warehouse_id.partner_id.country_id.code
            commodities.append(DeliveryCommodity(line.product_id, amount=rounded_qty, monetary_value=line.price_reduce_taxinc, country_of_origin=country_of_origin))

        return commodities

    def _get_commodities_from_stock_move_lines(self, move_lines):
        commodities = []

        product_lines = move_lines.filtered(lambda line: line.product_id.type in ['product', 'consu'])
        for product, lines in groupby(product_lines, lambda x: x.product_id):
            unit_quantity = sum(
                line.product_uom_id._compute_quantity(
                    line.qty_done if line.state == 'done' else line.reserved_uom_qty,
                    product.uom_id)
                for line in lines)
            rounded_qty = max(1, float_round(unit_quantity, precision_digits=0))
            country_of_origin = product.country_of_origin.code or lines[0].picking_id.picking_type_id.warehouse_id.partner_id.country_id.code
            unit_price = sum(line.sale_price for line in lines) / rounded_qty
            commodities.append(DeliveryCommodity(product, amount=rounded_qty, monetary_value=unit_price, country_of_origin=country_of_origin))

        return commodities

    def _product_price_to_company_currency(self, quantity, product, company):
        return company.currency_id._convert(quantity * product.standard_price, product.currency_id, company, fields.Date.today())

```

## File: models\delivery_grid.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields, api, _
from odoo.tools.safe_eval import safe_eval
from odoo.exceptions import UserError, ValidationError


class PriceRule(models.Model):
    _name = "delivery.price.rule"
    _description = "Delivery Price Rules"
    _order = 'sequence, list_price, id'

    @api.depends('variable', 'operator', 'max_value', 'list_base_price', 'list_price', 'variable_factor')
    def _compute_name(self):
        for rule in self:
            name = 'if %s %s %.02f then' % (rule.variable, rule.operator, rule.max_value)
            if rule.list_base_price and not rule.list_price:
                name = '%s fixed price %.02f' % (name, rule.list_base_price)
            elif rule.list_price and not rule.list_base_price:
                name = '%s %.02f times %s' % (name, rule.list_price, rule.variable_factor)
            else:
                name = '%s fixed price %.02f plus %.02f times %s' % (name, rule.list_base_price, rule.list_price, rule.variable_factor)
            rule.name = name

    name = fields.Char(compute='_compute_name')
    sequence = fields.Integer(required=True, default=10)
    carrier_id = fields.Many2one('delivery.carrier', 'Carrier', required=True, ondelete='cascade')

    variable = fields.Selection([('weight', 'Weight'), ('volume', 'Volume'), ('wv', 'Weight * Volume'), ('price', 'Price'), ('quantity', 'Quantity')], required=True, default='weight')
    operator = fields.Selection([('==', '='), ('<=', '<='), ('<', '<'), ('>=', '>='), ('>', '>')], required=True, default='<=')
    max_value = fields.Float('Maximum Value', required=True)
    list_base_price = fields.Float(string='Sale Base Price', digits='Product Price', required=True, default=0.0)
    list_price = fields.Float('Sale Price', digits='Product Price', required=True, default=0.0)
    variable_factor = fields.Selection([('weight', 'Weight'), ('volume', 'Volume'), ('wv', 'Weight * Volume'), ('price', 'Price'), ('quantity', 'Quantity')], 'Variable Factor', required=True, default='weight')


class ProviderGrid(models.Model):
    _inherit = 'delivery.carrier'

    delivery_type = fields.Selection(selection_add=[
        ('base_on_rule', 'Based on Rules'),
        ], ondelete={'base_on_rule': lambda recs: recs.write({
            'delivery_type': 'fixed', 'fixed_price': 0,
        })})
    price_rule_ids = fields.One2many('delivery.price.rule', 'carrier_id', 'Pricing Rules', copy=True)

    def base_on_rule_rate_shipment(self, order):
        carrier = self._match_address(order.partner_shipping_id)
        if not carrier:
            return {'success': False,
                    'price': 0.0,
                    'error_message': _('Error: this delivery method is not available for this address.'),
                    'warning_message': False}

        try:
            price_unit = self._get_price_available(order)
        except UserError as e:
            return {'success': False,
                    'price': 0.0,
                    'error_message': e.args[0],
                    'warning_message': False}

        price_unit = self._compute_currency(order, price_unit, 'company_to_pricelist')

        return {'success': True,
                'price': price_unit,
                'error_message': False,
                'warning_message': False}

    def _get_conversion_currencies(self, order, conversion):
        company_currency = (self.company_id or self.env['res.company']._get_main_company()).currency_id
        pricelist_currency = order.currency_id

        if conversion == 'company_to_pricelist':
            return company_currency, pricelist_currency
        elif conversion == 'pricelist_to_company':
            return pricelist_currency, company_currency

    def _compute_currency(self, order, price, conversion):
        from_currency, to_currency = self._get_conversion_currencies(order, conversion)
        if from_currency.id == to_currency.id:
            return price
        return from_currency._convert(price, to_currency, order.company_id, order.date_order or fields.Date.today())

    def _get_price_available(self, order):
        self.ensure_one()
        self = self.sudo()
        order = order.sudo()
        total = weight = volume = quantity = wv = 0
        total_delivery = 0.0
        for line in order.order_line:
            if line.state == 'cancel':
                continue
            if line.is_delivery:
                total_delivery += line.price_total
            if not line.product_id or line.is_delivery:
                continue
            if line.product_id.type == "service":
                continue
            qty = line.product_uom._compute_quantity(line.product_uom_qty, line.product_id.uom_id)
            weight += (line.product_id.weight or 0.0) * qty
            volume += (line.product_id.volume or 0.0) * qty
            wv += (line.product_id.weight or 0.0) * (line.product_id.volume or 0.0) * qty
            quantity += qty
        total = (order.amount_total or 0.0) - total_delivery

        total = self._compute_currency(order, total, 'pricelist_to_company')

        return self.with_context(wv=wv)._get_price_from_picking(total, weight, volume, quantity)

    def _get_price_dict(self, total, weight, volume, quantity):
        '''Hook allowing to retrieve dict to be used in _get_price_from_picking() function.
        Hook to be overridden when we need to add some field to product and use it in variable factor from price rules. '''
        return {
            'price': total,
            'volume': volume,
            'weight': weight,
            'wv': self.env.context.get('wv') or volume * weight,
            'quantity': quantity
        }

    def _get_price_from_picking(self, total, weight, volume, quantity):
        price = 0.0
        criteria_found = False
        price_dict = self._get_price_dict(total, weight, volume, quantity)
        if self.free_over and total >= self.amount:
            return 0
        for line in self.price_rule_ids:
            test = safe_eval(line.variable + line.operator + str(line.max_value), price_dict)
            if test:
                price = line.list_base_price + line.list_price * price_dict[line.variable_factor]
                criteria_found = True
                break
        if not criteria_found:
            raise UserError(_("No price rule matching this order; delivery cost cannot be computed."))

        return price

    def base_on_rule_send_shipping(self, pickings):
        res = []
        for p in pickings:
            carrier = self._match_address(p.partner_id)
            if not carrier:
                raise ValidationError(_('There is no matching delivery rule.'))
            res = res + [{'exact_price': p.carrier_id._get_price_available(p.sale_id) if p.sale_id else 0.0,  # TODO cleanme
                          'tracking_number': False}]
        return res

    def base_on_rule_get_tracking_link(self, picking):
        return False

    def base_on_rule_cancel_shipment(self, pickings):
        raise NotImplementedError()

```

## File: models\delivery_request_objects.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

class DeliveryPackage:
    """ Each provider need similar information about its packages. """
    def __init__(self, commodities, weight, package_type, name=None, total_cost=0, currency=None, picking=False, order=False):
        """ The UOMs are based on the config parameters, which is very convenient:
        we do not need to keep those stored."""
        self.picking_id = picking
        self.order_id = order
        self.company_id = order and order.company_id or picking and picking.company_id
        self.commodities = commodities or []  # list of DeliveryCommodity objects
        self.weight = weight
        self.dimension = {
            'length': package_type.packaging_length,
            'width': package_type.width,
            'height': package_type.height
        }
        self.packaging_type = package_type.shipper_package_code or False
        self.name = name
        self.total_cost = total_cost
        self.currency_id = currency


class DeliveryCommodity:
    """ Commodities information are needed for Commercial invoices with each provider. """
    def __init__(self, product, amount, monetary_value, country_of_origin):
        self.product_id = product
        self.qty = amount
        self.monetary_value = monetary_value  # based on company currency
        self.country_of_origin = country_of_origin

```

## File: models\delivery_zip_prefix.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class DeliveryZipPrefix(models.Model):
    """ Zip prefix that a delivery.carrier will deliver to. """
    _name = 'delivery.zip.prefix'
    _description = 'Delivery Zip Prefix'
    _order = 'name, id'

    name = fields.Char('Prefix', required=True)

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            # we cannot easily convert a list of prefix names into upper to compare with partner zips
            # later on, so let's ensure they are always upper
            vals['name'] = vals['name'].upper()
        return super(DeliveryZipPrefix, self).create(vals_list)

    def write(self, vals):
        vals['name'] = vals['name'].upper()
        return super().write(vals)

    _sql_constraints = [
        ('name_uniq', 'unique (name)', "Prefix already exists!"),
    ]

```

## File: models\partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResPartner(models.Model):
    _inherit = 'res.partner'

    property_delivery_carrier_id = fields.Many2one('delivery.carrier', company_dependent=True, string="Delivery Method", help="Default delivery method used in sales orders.")

```

## File: models\product_category.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, _
from odoo.exceptions import UserError


class ProductCategory(models.Model):
    _inherit = "product.category"

    @api.ondelete(at_uninstall=False)
    def _unlink_except_delivery_category(self):
        delivery_category = self.env.ref('delivery.product_category_deliveries', raise_if_not_found=False)
        if delivery_category and delivery_category in self:
            raise UserError(_("You cannot delete the deliveries product category as it is used on the delivery carriers products."))

```

## File: models\product_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields


class ProductTemplate(models.Model):
    _inherit = 'product.template'

    hs_code = fields.Char(
        string="HS Code",
        help="Standardized code for international shipping and goods declaration. At the moment, only used for FedEx and USPS shipping providers.",
    )
    country_of_origin = fields.Many2one(
        'res.country',
        'Origin of Goods',
        help="Rules of origin determine where goods originate, i.e. not where they have been shipped from, but where they have been produced or manufactured.\n"
             "As such, the ‘origin’ is the 'economic nationality' of goods traded in commerce.",
    )

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields, api, _
from odoo.exceptions import UserError


class SaleOrder(models.Model):
    _inherit = 'sale.order'

    carrier_id = fields.Many2one('delivery.carrier', string="Delivery Method", domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]", help="Fill this field if you plan to invoice the shipping based on picking.")
    delivery_message = fields.Char(readonly=True, copy=False)
    delivery_rating_success = fields.Boolean(copy=False)
    delivery_set = fields.Boolean(compute='_compute_delivery_state')
    recompute_delivery_price = fields.Boolean('Delivery cost should be recomputed')
    is_all_service = fields.Boolean("Service Product", compute="_compute_is_service_products")

    @api.depends('order_line')
    def _compute_is_service_products(self):
        for so in self:
            so.is_all_service = all(line.product_id.type == 'service' for line in so.order_line.filtered(lambda x: not x.display_type))

    def _compute_amount_total_without_delivery(self):
        self.ensure_one()
        delivery_cost = sum([l.price_total for l in self.order_line if l.is_delivery])
        return self.amount_total - delivery_cost

    @api.depends('order_line')
    def _compute_delivery_state(self):
        for order in self:
            order.delivery_set = any(line.is_delivery for line in order.order_line)

    @api.onchange('order_line', 'partner_id', 'partner_shipping_id')
    def onchange_order_line(self):
        self.ensure_one()
        delivery_line = self.order_line.filtered('is_delivery')
        if delivery_line:
            self.recompute_delivery_price = True

    def _get_update_prices_lines(self):
        """ Exclude delivery lines from price list recomputation based on product instead of carrier """
        lines = super()._get_update_prices_lines()
        return lines.filtered(lambda line: not line.is_delivery)

    def _remove_delivery_line(self):
        """Remove delivery products from the sales orders"""
        delivery_lines = self.order_line.filtered("is_delivery")
        if not delivery_lines:
            return
        to_delete = delivery_lines.filtered(lambda x: x.qty_invoiced == 0)
        if not to_delete:
            raise UserError(
                _('You can not update the shipping costs on an order where it was already invoiced!\n\nThe following delivery lines (product, invoiced quantity and price) have already been processed:\n\n')
                + '\n'.join(['- %s: %s x %s' % (line.product_id.with_context(display_default_code=False).display_name, line.qty_invoiced, line.price_unit) for line in delivery_lines])
            )
        to_delete.unlink()
        self.carrier_id = self.env['delivery.carrier']  # reset carrier

    def set_delivery_line(self, carrier, amount):
        self._remove_delivery_line()
        for order in self:
            order.carrier_id = carrier.id
            if order.state in ('sale', 'done'):
                pending_deliveries = order.picking_ids.filtered(
                    lambda p: p.state not in ('done', 'cancel') and not any(m.origin_returned_move_id for m in p.move_ids))
                pending_deliveries.carrier_id = carrier.id
            order._create_delivery_line(carrier, amount)
        return True

    def action_open_delivery_wizard(self):
        view_id = self.env.ref('delivery.choose_delivery_carrier_view_form').id
        if self.env.context.get('carrier_recompute'):
            name = _('Update shipping cost')
            carrier = self.carrier_id
        else:
            name = _('Add a shipping method')
            carrier = (
                self.with_company(self.company_id).partner_shipping_id.property_delivery_carrier_id
                or self.with_company(self.company_id).partner_shipping_id.commercial_partner_id.property_delivery_carrier_id
            )
        return {
            'name': name,
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'choose.delivery.carrier',
            'view_id': view_id,
            'views': [(view_id, 'form')],
            'target': 'new',
            'context': {
                'default_order_id': self.id,
                'default_carrier_id': carrier.id,
            }
        }

    def _prepare_delivery_line_vals(self, carrier, price_unit):
        context = {}
        if self.partner_id:
            # set delivery detail in the customer language
            context['lang'] = self.partner_id.lang
            carrier = carrier.with_context(lang=self.partner_id.lang)

        # Apply fiscal position
        taxes = carrier.product_id.taxes_id.filtered(lambda t: t.company_id.id == self.company_id.id)
        taxes_ids = taxes.ids
        if self.partner_id and self.fiscal_position_id:
            taxes_ids = self.fiscal_position_id.map_tax(taxes).ids

        # Create the sales order line

        if carrier.product_id.description_sale:
            so_description = '%s: %s' % (carrier.name,
                                        carrier.product_id.description_sale)
        else:
            so_description = carrier.name
        values = {
            'order_id': self.id,
            'name': so_description,
            'product_uom_qty': 1,
            'product_uom': carrier.product_id.uom_id.id,
            'product_id': carrier.product_id.id,
            'tax_id': [(6, 0, taxes_ids)],
            'is_delivery': True,
        }
        if carrier.invoice_policy == 'real':
            values['price_unit'] = 0
            values['name'] += _(' (Estimated Cost: %s )', self.currency_id.format(price_unit))
        else:
            values['price_unit'] = price_unit
        if carrier.free_over and self.currency_id.is_zero(price_unit) :
            values['name'] += '\n' + _('Free Shipping')
        if self.order_line:
            values['sequence'] = self.order_line[-1].sequence + 1
        del context
        return values

    def _create_delivery_line(self, carrier, price_unit):
        values = self._prepare_delivery_line_vals(carrier, price_unit)
        return self.env['sale.order.line'].sudo().create(values)

    # to remove in master
    def _format_currency_amount(self, amount):
        pre = post = u''
        if self.currency_id.position == 'before':
            pre = u'{symbol}\N{NO-BREAK SPACE}'.format(symbol=self.currency_id.symbol or '')
        else:
            post = u'\N{NO-BREAK SPACE}{symbol}'.format(symbol=self.currency_id.symbol or '')
        return u' {pre}{0}{post}'.format(amount, pre=pre, post=post)

    @api.depends('order_line.is_delivery', 'order_line.is_downpayment')
    def _compute_invoice_status(self):
        super()._compute_invoice_status()
        for order in self:
            if order.invoice_status in ['no', 'invoiced']:
                continue
            order_lines = order._get_lines_impacting_invoice_status()
            if all(line.product_id.invoice_policy == 'delivery' and line.invoice_status == 'no' for line in order_lines):
                order.invoice_status = 'no'

    def _get_lines_impacting_invoice_status(self):
        return self.order_line.filtered(
            lambda line:
                not line.is_delivery
                and not line.is_downpayment
                and not line.display_type
                and line.invoice_status != 'invoiced'
        )

    def _get_estimated_weight(self):
        self.ensure_one()
        weight = 0.0
        for order_line in self.order_line.filtered(lambda l: l.product_id.type in ['product', 'consu'] and not l.is_delivery and not l.display_type and l.product_uom_qty > 0):
            weight += order_line.product_qty * order_line.product_id.weight
        return weight


class SaleOrderLine(models.Model):
    _inherit = 'sale.order.line'

    is_delivery = fields.Boolean(string="Is a Delivery", default=False)
    product_qty = fields.Float(compute='_compute_product_qty', string='Product Qty', digits='Product Unit of Measure')
    recompute_delivery_price = fields.Boolean(related='order_id.recompute_delivery_price')

    def _is_not_sellable_line(self):
        return self.is_delivery or super(SaleOrderLine, self)._is_not_sellable_line()

    @api.depends('product_id', 'product_uom', 'product_uom_qty')
    def _compute_product_qty(self):
        for line in self:
            if not line.product_id or not line.product_uom or not line.product_uom_qty:
                line.product_qty = 0.0
                continue
            line.product_qty = line.product_uom._compute_quantity(line.product_uom_qty, line.product_id.uom_id)

    def unlink(self):
        self.filtered('is_delivery').order_id.filtered('carrier_id').carrier_id = False
        return super(SaleOrderLine, self).unlink()

    def _is_delivery(self):
        self.ensure_one()
        return self.is_delivery

    # override to allow deletion of delivery line in a confirmed order
    def _check_line_unlink(self):
        """
        Extend the allowed deletion policy of SO lines.

        Lines that are delivery lines can be deleted from a confirmed order.

        :rtype: recordset sale.order.line
        :returns: set of lines that cannot be deleted
        """

        undeletable_lines = super()._check_line_unlink()
        return undeletable_lines.filtered(lambda line: not line.is_delivery)

    def _compute_pricelist_item_id(self):
        delivery_lines = self.filtered('is_delivery')
        super(SaleOrderLine, self - delivery_lines)._compute_pricelist_item_id()
        delivery_lines.pricelist_item_id = False

```

## File: models\stock_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo.tools.sql import column_exists, create_column


class StockMove(models.Model):
    _inherit = 'stock.move'

    def _auto_init(self):
        if not column_exists(self.env.cr, "stock_move", "weight"):
            # In case of a big database with a lot of stock moves, the RAM gets exhausted
            # To prevent a process from being killed We create the column 'weight' manually
            # Then we do the computation in a query by multiplying product weight with qty
            create_column(self.env.cr, "stock_move", "weight", "numeric")
            self.env.cr.execute("""
                UPDATE stock_move move
                SET weight = move.product_qty * product.weight
                FROM product_product product
                WHERE move.product_id = product.id
                AND move.state != 'cancel'
                """)
        return super()._auto_init()

    weight = fields.Float(compute='_cal_move_weight', digits='Stock Weight', store=True, compute_sudo=True)

    @api.depends('product_id', 'product_uom_qty', 'product_uom')
    def _cal_move_weight(self):
        moves_with_weight = self.filtered(lambda moves: moves.product_id.weight > 0.00)
        for move in moves_with_weight:
            move.weight = (move.product_qty * move.product_id.weight)
        (self - moves_with_weight).weight = 0

    def _get_new_picking_values(self):
        vals = super(StockMove, self)._get_new_picking_values()
        carrier_id = self.group_id.sale_id.carrier_id.id
        vals['carrier_id'] = any(rule.propagate_carrier for rule in self.rule_id) and carrier_id
        return vals

    def _key_assign_picking(self):
        keys = super(StockMove, self)._key_assign_picking()
        return keys + (self.sale_line_id.order_id.carrier_id,)

class StockMoveLine(models.Model):
    _inherit = 'stock.move.line'

    sale_price = fields.Float(compute='_compute_sale_price')
    destination_country_code = fields.Char(related='picking_id.destination_country_code')
    carrier_id = fields.Many2one(related='picking_id.carrier_id')
    carrier_name = fields.Char(related='picking_id.carrier_id.name', readonly=True, store=True, string="Carrier Name")

    @api.depends('qty_done', 'product_uom_id', 'product_id', 'move_id.sale_line_id', 'move_id.sale_line_id.price_reduce_taxinc', 'move_id.sale_line_id.product_uom')
    def _compute_sale_price(self):
        for move_line in self:
            if move_line.move_id.sale_line_id:
                unit_price = move_line.move_id.sale_line_id.price_reduce_taxinc
                qty = move_line.product_uom_id._compute_quantity(move_line.qty_done, move_line.move_id.sale_line_id.product_uom)
            else:
                unit_price = move_line.product_id.list_price
                qty = move_line.product_uom_id._compute_quantity(move_line.qty_done, move_line.product_id.uom_id)
            move_line.sale_price = unit_price * qty
        super(StockMoveLine, self)._compute_sale_price()

    def _get_aggregated_product_quantities(self, **kwargs):
        """Returns dictionary of products and corresponding values of interest + hs_code

        Unfortunately because we are working with aggregated data, we have to loop through the
        aggregation to add more values to each datum. This extension adds on the hs_code value.

        returns: dictionary {same_key_as_super: {same_values_as_super, hs_code}, ...}
        """
        aggregated_move_lines = super()._get_aggregated_product_quantities(**kwargs)
        for aggregated_move_line in aggregated_move_lines:
            hs_code = aggregated_move_lines[aggregated_move_line]['product'].product_tmpl_id.hs_code
            aggregated_move_lines[aggregated_move_line]['hs_code'] = hs_code
        return aggregated_move_lines

```

## File: models\stock_package_type.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields, api


class PackageType(models.Model):
    _inherit = 'stock.package.type'

    shipper_package_code = fields.Char('Carrier Code')
    package_carrier_type = fields.Selection([('none', 'No carrier integration')], string='Carrier', default='none')

    @api.onchange('package_carrier_type')
    def _onchange_carrier_type(self):
        carrier_id = self.env['delivery.carrier'].search([('delivery_type', '=', self.package_carrier_type)], limit=1)
        if carrier_id:
            self.shipper_package_code = carrier_id._get_default_custom_package_code()
        else:
            self.shipper_package_code = False

```

## File: models\stock_picking.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json
from collections import defaultdict
from datetime import date
from markupsafe import Markup

from odoo import models, fields, api, _, SUPERUSER_ID
from odoo.exceptions import UserError
from odoo.tools.sql import column_exists, create_column


class StockQuantPackage(models.Model):
    _inherit = "stock.quant.package"

    @api.depends('quant_ids', 'package_type_id')
    def _compute_weight(self):
        if self.env.context.get('picking_id'):
            package_weights = defaultdict(float)
            # Ordering by qty_done prevents the default ordering by groupby fields that can inject multiple Left Joins in the resulting query.
            res_groups = self.env['stock.move.line'].read_group(
                [('result_package_id', 'in', self.ids), ('product_id', '!=', False), ('picking_id', '=', self.env.context['picking_id'])],
                ['id:count'],
                ['result_package_id', 'product_id', 'product_uom_id', 'qty_done'],
                lazy=False, orderby='qty_done asc'
            )
            for res_group in res_groups:
                product_id = self.env['product.product'].browse(res_group['product_id'][0])
                product_uom_id = self.env['uom.uom'].browse(res_group['product_uom_id'][0])
                package_weights[res_group['result_package_id'][0]] += (
                    res_group['__count']
                    * product_uom_id._compute_quantity(res_group['qty_done'], product_id.uom_id)
                    * product_id.weight
                )
        for package in self:
            weight = package.package_type_id.base_weight or 0.0
            if self.env.context.get('picking_id'):
                package.weight = weight + package_weights[package.id]
            else:
                for quant in package.quant_ids:
                    weight += quant.quantity * quant.product_id.weight
                package.weight = weight

    def _get_default_weight_uom(self):
        return self.env['product.template']._get_weight_uom_name_from_ir_config_parameter()

    def _compute_weight_uom_name(self):
        for package in self:
            package.weight_uom_name = self.env['product.template']._get_weight_uom_name_from_ir_config_parameter()

    def _compute_weight_is_kg(self):
        self.weight_is_kg = False
        uom_id = self.env['product.template']._get_weight_uom_id_from_ir_config_parameter()
        if uom_id == self.env.ref('uom.product_uom_kgm'):
            self.weight_is_kg = True
        self.weight_uom_rounding = uom_id.rounding

    weight = fields.Float(compute='_compute_weight', digits='Stock Weight', help="Total weight of all the products contained in the package.")
    weight_uom_name = fields.Char(string='Weight unit of measure label', compute='_compute_weight_uom_name', readonly=True, default=_get_default_weight_uom)
    weight_is_kg = fields.Boolean("Technical field indicating whether weight uom is kg or not (i.e. lb)", compute="_compute_weight_is_kg")
    weight_uom_rounding = fields.Float("Technical field indicating weight's number of decimal places", compute="_compute_weight_is_kg")
    shipping_weight = fields.Float(string='Shipping Weight', help="Total weight of the package.")


class StockPicking(models.Model):
    _inherit = 'stock.picking'

    def _auto_init(self):
        if not column_exists(self.env.cr, "stock_picking", "weight"):
            # In order to speed up module installation when dealing with hefty data
            # We create the column weight manually, but the computation will be skipped
            # Therefore we do the computation in a query by getting weight sum from stock moves
            create_column(self.env.cr, "stock_picking", "weight", "numeric")
            self.env.cr.execute("""
                WITH computed_weight AS (
                    SELECT SUM(weight) AS weight_sum, picking_id
                    FROM stock_move
                    WHERE picking_id IS NOT NULL
                    GROUP BY picking_id
                )
                UPDATE stock_picking
                SET weight = weight_sum
                FROM computed_weight
                WHERE stock_picking.id = computed_weight.picking_id;
            """)
        return super()._auto_init()

    @api.depends('move_line_ids', 'move_line_ids.result_package_id')
    def _compute_packages(self):
        packages = {
            res["picking_id"][0]: set(res["result_package_id"])
            for res in self.env["stock.move.line"].read_group(
                [("picking_id", "in", self.ids), ("result_package_id", "!=", False)],
                ["result_package_id:array_agg"],
                ["picking_id"],
                lazy=False, orderby="picking_id asc",
            )
        }
        for picking in self:
            picking.package_ids = list(packages.get(picking.id, []))

    @api.depends('move_line_ids', 'move_line_ids.result_package_id', 'move_line_ids.product_uom_id', 'move_line_ids.qty_done')
    def _compute_bulk_weight(self):
        picking_weights = defaultdict(float)
        # Ordering by qty_done prevents the default ordering by groupby fields that can inject multiple Left Joins in the resulting query.
        res_groups = self.env['stock.move.line'].read_group(
            [('picking_id', 'in', self.ids), ('product_id', '!=', False), ('result_package_id', '=', False)],
            ['id:count'],
            ['picking_id', 'product_id', 'product_uom_id', 'qty_done'],
            lazy=False, orderby='qty_done asc'
        )
        products_by_id = {
            product_res['id']: (product_res['uom_id'][0], product_res['weight'])
            for product_res in
            self.env['product.product'].with_context(active_test=False).search_read(
                [('id', 'in', list(set(grp["product_id"][0] for grp in res_groups)))], ['uom_id', 'weight'])
        }
        for res_group in res_groups:
            uom_id, weight = products_by_id[res_group['product_id'][0]]
            uom = self.env['uom.uom'].browse(uom_id)
            product_uom_id = self.env['uom.uom'].browse(res_group['product_uom_id'][0])
            picking_weights[res_group['picking_id'][0]] += (
                res_group['__count']
                * product_uom_id._compute_quantity(res_group['qty_done'], uom)
                * weight
            )
        for picking in self:
            picking.weight_bulk = picking_weights[picking.id]

    @api.depends('move_line_ids.result_package_id', 'move_line_ids.result_package_id.shipping_weight', 'weight_bulk')
    def _compute_shipping_weight(self):
        for picking in self:
            # if shipping weight is not assigned => default to calculated product weight
            picking.shipping_weight = (
                picking.weight_bulk +
                sum(pack.shipping_weight or pack.weight for pack in picking.package_ids.sudo())
            )

    def _get_default_weight_uom(self):
        return self.env['product.template']._get_weight_uom_name_from_ir_config_parameter()

    def _compute_weight_uom_name(self):
        for package in self:
            package.weight_uom_name = self.env['product.template']._get_weight_uom_name_from_ir_config_parameter()

    carrier_price = fields.Float(string="Shipping Cost")
    delivery_type = fields.Selection(related='carrier_id.delivery_type', readonly=True)
    carrier_id = fields.Many2one("delivery.carrier", string="Carrier", check_company=True)
    weight = fields.Float(compute='_cal_weight', digits='Stock Weight', store=True, help="Total weight of the products in the picking.", compute_sudo=True)
    carrier_tracking_ref = fields.Char(string='Tracking Reference', copy=False)
    carrier_tracking_url = fields.Char(string='Tracking URL', compute='_compute_carrier_tracking_url')
    weight_uom_name = fields.Char(string='Weight unit of measure label', compute='_compute_weight_uom_name', readonly=True, default=_get_default_weight_uom)
    package_ids = fields.Many2many('stock.quant.package', compute='_compute_packages', string='Packages')
    weight_bulk = fields.Float('Bulk Weight', compute='_compute_bulk_weight', help="Total weight of products which are not in a package.")
    shipping_weight = fields.Float("Weight for Shipping", compute='_compute_shipping_weight',
        help="Total weight of packages and products not in a package. Packages with no shipping weight specified will default to their products' total weight. This is the weight used to compute the cost of the shipping.")
    is_return_picking = fields.Boolean(compute='_compute_return_picking')
    return_label_ids = fields.One2many('ir.attachment', compute='_compute_return_label')
    destination_country_code = fields.Char(related='partner_id.country_id.code', string="Destination Country")

    @api.depends('carrier_id', 'carrier_tracking_ref')
    def _compute_carrier_tracking_url(self):
        for picking in self:
            picking.carrier_tracking_url = picking.carrier_id.get_tracking_link(picking) if picking.carrier_id and picking.carrier_tracking_ref else False

    @api.depends('carrier_id', 'move_ids_without_package')
    def _compute_return_picking(self):
        for picking in self:
            if picking.carrier_id and picking.carrier_id.can_generate_return:
                picking.is_return_picking = any(m.origin_returned_move_id for m in picking.move_ids_without_package)
            else:
                picking.is_return_picking = False

    def _compute_return_label(self):
        for picking in self:
            if picking.carrier_id:
                picking.return_label_ids = self.env['ir.attachment'].search([('res_model', '=', 'stock.picking'), ('res_id', '=', picking.id), ('name', 'like', '%s%%' % picking.carrier_id.get_return_label_prefix())])
            else:
                picking.return_label_ids = False

    def get_multiple_carrier_tracking(self):
        self.ensure_one()
        try:
            return json.loads(self.carrier_tracking_url)
        except (ValueError, TypeError):
            return False

    @api.depends('move_ids.weight')
    def _cal_weight(self):
        for picking in self:
            picking.weight = sum(move.weight for move in picking.move_ids if move.state != 'cancel')

    def _carrier_exception_note(self, exception):
        self.ensure_one()
        line_1 = _("Exception occurred with respect to carrier on the transfer")
        line_2 = _("Manual actions might be needed.")
        line_3 = _("Exception:")
        return Markup('<div> {line_1} <a href="#" data-oe-model="stock.picking" data-oe-id="{picking_id}"> {picking_name}</a>. {line_2}<div class="mt16"><p>{line_3} {exception}</p></div></div>').format(line_1=line_1, line_2=line_2, line_3=line_3, picking_id=self.id, picking_name=self.name, exception=exception)

    def _send_confirmation_email(self):
        # The carrier's API processes validity checks and parcels generation one picking at a time.
        # However, since a UserError of any of the picking will cause a rollback of the entire batch
        # on Odoo's side and since pickings that were already processed on the carrier's side must
        # stay validated, UserErrors might need to be replaced by activity warnings.

        processed_carrier_picking = False

        for pick in self:
            try:
                if pick.carrier_id and pick.carrier_id.integration_level == 'rate_and_ship' and pick.picking_type_code != 'incoming' and not pick.carrier_tracking_ref and pick.picking_type_id.print_label:
                    pick.sudo().send_to_shipper()
                pick._check_carrier_details_compliance()
                if pick.carrier_id:
                    processed_carrier_picking = True
            except (UserError) as e:
                if processed_carrier_picking:
                    # We can not raise a UserError at this point
                    exception_message = str(e)
                    pick.message_post(body=exception_message, message_type='notification')
                    pick.sudo().activity_schedule(
                        'mail.mail_activity_data_warning',
                        date.today(),
                        note=pick._carrier_exception_note(exception_message),
                        user_id=pick.user_id.id or self.env.user.id or SUPERUSER_ID,
                        )
                else:
                    raise e

        return super(StockPicking, self)._send_confirmation_email()

    def _pre_put_in_pack_hook(self, move_line_ids):
        res = super(StockPicking, self)._pre_put_in_pack_hook(move_line_ids)
        if not res:
            if move_line_ids.carrier_id:
                if len(move_line_ids.carrier_id) > 1 or any(not ml.carrier_id for ml in move_line_ids):
                    # avoid (duplicate) costs for products
                    raise UserError(_("You cannot pack products into the same package when they have different carriers (i.e. check that all of their transfers have a carrier assigned and are using the same carrier)."))
                return self._set_delivery_package_type(batch_pack=len(move_line_ids.picking_id) > 1)
        else:
            return res

    def _set_delivery_package_type(self, batch_pack=False):
        """ This method returns an action allowing to set the package type and the shipping weight
        on the stock.quant.package.
        """
        self.ensure_one()
        view_id = self.env.ref('delivery.choose_delivery_package_view_form').id
        context = dict(
            self.env.context,
            current_package_carrier_type=self.carrier_id.delivery_type,
            default_picking_id=self.id,
            batch_pack=batch_pack,
        )
        # As we pass the `delivery_type` ('fixed' or 'base_on_rule' by default) in a key who
        # correspond to the `package_carrier_type` ('none' to default), we make a conversion.
        # No need conversion for other carriers as the `delivery_type` and
        #`package_carrier_type` will be the same in these cases.
        if context['current_package_carrier_type'] in ['fixed', 'base_on_rule']:
            context['current_package_carrier_type'] = 'none'
        return {
            'name': _('Package Details'),
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'choose.delivery.package',
            'view_id': view_id,
            'views': [(view_id, 'form')],
            'target': 'new',
            'context': context,
        }

    def send_to_shipper(self):
        self.ensure_one()
        res = self.carrier_id.send_shipping(self)[0]
        if self.carrier_id.free_over and self.sale_id:
            amount_without_delivery = self.sale_id._compute_amount_total_without_delivery()
            if self.carrier_id._compute_currency(self.sale_id, amount_without_delivery, 'pricelist_to_company') >= self.carrier_id.amount:
                res['exact_price'] = 0.0
        self.carrier_price = res['exact_price'] * (1.0 + (self.carrier_id.margin / 100.0))
        if res['tracking_number']:
            related_pickings = self.env['stock.picking'] if self.carrier_tracking_ref and res['tracking_number'] in self.carrier_tracking_ref else self
            accessed_moves = previous_moves = self.move_ids.move_orig_ids
            while previous_moves:
                related_pickings |= previous_moves.picking_id
                previous_moves = previous_moves.move_orig_ids - accessed_moves
                accessed_moves |= previous_moves
            accessed_moves = next_moves = self.move_ids.move_dest_ids
            while next_moves:
                related_pickings |= next_moves.picking_id
                next_moves = next_moves.move_dest_ids - accessed_moves
                accessed_moves |= next_moves
            without_tracking = related_pickings.filtered(lambda p: not p.carrier_tracking_ref)
            without_tracking.carrier_tracking_ref = res['tracking_number']
            for p in related_pickings - without_tracking:
                p.carrier_tracking_ref += "," + res['tracking_number']
        order_currency = self.sale_id.currency_id or self.company_id.currency_id
        msg = _(
            "Shipment sent to carrier %(carrier_name)s for shipping with tracking number %(ref)s<br/>Cost: %(price).2f %(currency)s",
            carrier_name=self.carrier_id.name,
            ref=self.carrier_tracking_ref,
            price=self.carrier_price,
            currency=order_currency.name
        )
        self.message_post(body=msg)
        self._add_delivery_cost_to_so()

    def _check_carrier_details_compliance(self):
        """Hook to check if a delivery is compliant in regard of the carrier.
        """
        return

    def print_return_label(self):
        self.ensure_one()
        self.carrier_id.get_return_label(self)

    def _get_matching_delivery_lines(self):
        return self.sale_id.order_line.filtered(
            lambda l: l.is_delivery
            and l.currency_id.is_zero(l.price_unit)
            and l.product_id == self.carrier_id.product_id
        )

    def _prepare_sale_delivery_line_vals(self):
        return {
            'price_unit': self.carrier_price,
            # remove the estimated price from the description
            'name': self.carrier_id.with_context(lang=self.partner_id.lang).name,
        }

    def _add_delivery_cost_to_so(self):
        self.ensure_one()
        sale_order = self.sale_id
        if sale_order and self.carrier_id.invoice_policy == 'real' and self.carrier_price:
            delivery_lines = self._get_matching_delivery_lines()
            if not delivery_lines:
                delivery_lines = sale_order._create_delivery_line(self.carrier_id, self.carrier_price)
            vals = self._prepare_sale_delivery_line_vals()
            delivery_lines[0].write(vals)

    def open_website_url(self):
        self.ensure_one()
        if not self.carrier_tracking_url:
            raise UserError(_("Your delivery method has no redirect on courier provider's website to track this order."))

        carrier_trackers = []
        try:
            carrier_trackers = json.loads(self.carrier_tracking_url)
        except ValueError:
            carrier_trackers = self.carrier_tracking_url
        else:
            msg = "Tracking links for shipment: <br/>"
            for tracker in carrier_trackers:
                msg += '<a href=' + tracker[1] + '>' + tracker[0] + '</a><br/>'
            self.message_post(body=msg)
            return self.env["ir.actions.actions"]._for_xml_id("delivery.act_delivery_trackers_url")

        client_action = {
            'type': 'ir.actions.act_url',
            'name': "Shipment Tracking Page",
            'target': 'new',
            'url': self.carrier_tracking_url,
        }
        return client_action

    def cancel_shipment(self):
        for picking in self:
            picking.carrier_id.cancel_shipment(self)
            msg = "Shipment %s cancelled" % picking.carrier_tracking_ref
            picking.message_post(body=msg)
            picking.carrier_tracking_ref = False

    def _get_estimated_weight(self):
        self.ensure_one()
        weight = 0.0
        for move in self.move_ids:
            weight += move.product_qty * move.product_id.weight
        return weight

    def _should_generate_commercial_invoice(self):
        self.ensure_one()
        return self.picking_type_id.warehouse_id.partner_id.country_id != self.partner_id.country_id


class StockReturnPicking(models.TransientModel):
    _inherit = 'stock.return.picking'

    def _create_returns(self):
        # Prevent copy of the carrier and carrier price when generating return picking
        # (we have no integration of returns for now)
        new_picking, pick_type_id = super(StockReturnPicking, self)._create_returns()
        picking = self.env['stock.picking'].browse(new_picking)
        picking.write({'carrier_id': False,
                       'carrier_price': 0.0})
        return new_picking, pick_type_id

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import delivery_carrier
from . import delivery_grid
from . import delivery_zip_prefix
from . import product_template
from . import sale_order
from . import partner
from . import product_category
from . import stock_move
from . import stock_picking
from . import stock_package_type

```

## File: report\ir_actions_report_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<template id="delivery_report_saleorder_document" inherit_id="sale.report_saleorder_document">
    <xpath expr="//p[@name='order_note']" position="before">
        <p t-if="doc.carrier_id.carrier_description" id="carrier_description">
            <strong>Shipping Description:</strong>
            <span t-out="doc.carrier_id.carrier_description"/>
        </p>
    </xpath>
</template>
</odoo>

```

## File: security\delivery_carrier_security.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<odoo noupdate="1">

    <record model="ir.rule" id="delivery_carrier_comp_rule">
      <field name="name">Delivery Carrier multi-company</field>
      <field name="model_id" ref="model_delivery_carrier"/>
      <field name="domain_force"> [('company_id', 'in', company_ids + [False])]</field>
    </record>

  </odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_delivery_carrier,delivery.carrier,model_delivery_carrier,sales_team.group_sale_salesman,1,0,0,0
access_delivery_carrier_system,delivery.carrier,model_delivery_carrier,base.group_system,1,0,0,0
access_delivery_zip_prefix,delivery.zip.prefix,model_delivery_zip_prefix,sales_team.group_sale_salesman,1,0,0,0
access_delivery_price_rule,delivery.price.rule,model_delivery_price_rule,sales_team.group_sale_salesman,1,0,0,0
access_delivery_carrier_manager,delivery.carrier,model_delivery_carrier,sales_team.group_sale_manager,1,1,1,1
access_delivery_price_rule_manager,delivery.price.rule,model_delivery_price_rule,sales_team.group_sale_manager,1,1,1,1
access_delivery_carrier_partner_manager,delivery.carrier partner_manager,model_delivery_carrier,base.group_partner_manager,1,0,0,0
access_delivery_carrier_stock_user,delivery.carrier stock_user,model_delivery_carrier,stock.group_stock_user,1,0,0,0
access_delivery_carrier_stock_manager,delivery.carrier,model_delivery_carrier,stock.group_stock_manager,1,1,1,1
access_delivery_zip_prefix_stock_manager,delivery.zip.prefix,model_delivery_zip_prefix,stock.group_stock_manager,1,1,1,1
access_delivery_price_rule_stock_manager,delivery.price.rule,model_delivery_price_rule,stock.group_stock_manager,1,1,1,1
access_choose_delivery_package,access.choose.delivery.package,model_choose_delivery_package,stock.group_stock_user,1,1,1,0
access_choose_delivery_carrier,access.choose.delivery.carrier,model_choose_delivery_carrier,stock.group_stock_user,1,1,1,0
access_choose_delivery_carrier_salesman,access.choose.delivery.carrier salesman,model_choose_delivery_carrier,sales_team.group_sale_salesman,1,1,1,0

```

## File: views\delivery_portal_template.xml

```xml
<odoo>
    <template id="sale_order_portal_content_inherit_sale_stock_inherit_website_sale_delivery"
        name="Shipping tracking on orders followup"
        inherit_id="sale_stock.sale_order_portal_content_inherit_sale_stock">
        <xpath expr="//div[@name='delivery_details']" position="after">
            <div t-if="picking.carrier_tracking_ref" class="small d-lg-inline-block">
                Tracking:
                <t t-set="multiple_carrier_tracking" t-value="picking.get_multiple_carrier_tracking()"/>
                <t t-if="multiple_carrier_tracking">
                    <t t-foreach="multiple_carrier_tracking" t-as="line">
                        <a t-att-href="line[1]" target="_blank">
                            <span t-esc="line[0]"/>
                        </a>
                        <span t-if="not line_last"> + </span>
                    </t>
                </t>
                <t t-elif="picking.carrier_tracking_url">
                    <a t-att-href="picking.carrier_tracking_url" target="_blank">
                        <span t-field="picking.carrier_tracking_ref"/>
                    </a>
                </t>
                <t t-else="">
                    <span t-field="picking.carrier_id.name"/> <span t-field="picking.carrier_tracking_ref"/>
                </t>
            </div>
            <div t-if="picking.carrier_id.get_return_label_from_portal and picking.return_label_ids">
                <a class="ms-3" t-attf-href="/web/content/#{picking.return_label_ids[:1].id}?access_token=#{picking.return_label_ids[:1].access_token}" target="_blank">
                    Print Return Label
                </a>
            </div>
        </xpath>
    </template>
</odoo>

```

## File: views\delivery_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <!-- Delivery Carriers -->
        <record id="view_delivery_carrier_tree" model="ir.ui.view">
            <field name="name">delivery.carrier.tree</field>
            <field name="model">delivery.carrier</field>
            <field name="arch" type="xml">
                <tree string="Carrier">
                    <field name="sequence" widget="handle"/>
                    <field name="name"/>
                    <field name="delivery_type"/>
                    <field name="company_id" groups="base.group_multi_company"/>
                </tree>
            </field>
        </record>

        <record id="view_delivery_carrier_search" model="ir.ui.view">
            <field name="name">delivery.carrier.search</field>
            <field name="model">delivery.carrier</field>
            <field name="arch" type="xml">
                <search string="Delivery Carrier">
                    <field name="name" string="Carrier" />
                    <field name="delivery_type"/>
                    <separator/>
                    <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
                    <group expand="1" string="Group By">
                        <filter string="Provider" name="provider" context="{'group_by':'delivery_type', 'residual_visible':True}"/>
                    </group>
                </search>
            </field>
        </record>

        <record id="view_delivery_carrier_form" model="ir.ui.view">
            <field name="name">delivery.carrier.form</field>
            <field name="model">delivery.carrier</field>
            <field name="arch" type="xml">
                <form string="Carrier">
                    <sheet>
                        <div class="oe_button_box" name="button_box">
                            <button name="toggle_prod_environment"
                                    attrs="{'invisible': ['|', '|', ('prod_environment', '=', False), ('delivery_type', '=', 'fixed'), ('delivery_type', '=', 'base_on_rule')]}"
                                    class="oe_stat_button"
                                    type="object" icon="fa-play">
                                <div class="o_stat_info o_field_widget">
                                    <span class="text-success">Production</span>
                                    <span class="o_stat_text">Environment</span>
                                </div>
                            </button>
                            <!-- transfer referenced here due to view inheritance issue in current master (post-saas-16) -->
                            <button name="toggle_prod_environment"
                                    attrs="{'invisible': ['|', '|', ('prod_environment', '=', True), ('delivery_type', '=', 'fixed'), ('delivery_type', '=', 'base_on_rule')]}"
                                    class="oe_stat_button"
                                    type="object" icon="fa-stop">
                                <div class="o_stat_info o_field_widget">
                                    <span class="o_warning_text">Test</span>
                                    <span class="o_stat_text">Environment</span>
                                </div>
                            </button>
                            <button name="toggle_debug"
                                    attrs="{'invisible': ['|', '|', ('delivery_type', '=', 'fixed'), ('delivery_type', '=', 'base_on_rule'), ('debug_logging', '=', True)]}"
                                    class="oe_stat_button"
                                    type="object" icon="fa-code">
                                <div class="o_stat_info o_field_widget">
                                    <span class="text-danger">No debug</span>
                                </div>
                            </button>
                            <button name="toggle_debug"
                                    attrs="{'invisible': ['|', '|', ('delivery_type', '=', 'fixed'), ('delivery_type', '=', 'base_on_rule'), ('debug_logging', '=', False)]}"
                                    class="oe_stat_button"
                                    type="object" icon="fa-code">
                                <div class="o_stat_info o_field_widget">
                                    <span class="text-success">Debug requests</span>
                                </div>
                            </button>
                        </div>
                        <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                        <div class="oe_title" name="title">
                            <label for="name" string="Shipping Method"/>
                            <h1>
                                <field name="name" placeholder="e.g. UPS Express"/>
                            </h1>
                        </div>
                        <group>
                            <group name="provider_details">
                                <field name="active" invisible="1"/>
                                <field name="company_id" invisible="1"/>
                                <field name="prod_environment" invisible="1"/>
                                <field name="debug_logging" invisible="1"/>
                                <label for="delivery_type"/>
                                <div>
                                    <field name="delivery_type" />
                                    <button string="Install more Providers" name="install_more_provider" type="object" class="oe_link oe_edit_only"/>
                                </div>
                                <field name="integration_level" widget="radio" attrs="{'invisible': ['|', ('delivery_type', '=', 'fixed'), ('delivery_type', '=', 'base_on_rule')]}"/>
                                <field name="company_id" groups="base.group_multi_company"/>
                            </group>
                            <group name="delivery_details">
                                <field name="product_id" context="{'default_detailed_type': 'service', 'default_sale_ok': False, 'default_purchase_ok': False, 'default_invoice_policy': 'order'}" />
                                <field name="invoice_policy" widget="radio" attrs="{'invisible': ['|', ('delivery_type', 'in', ('fixed', 'base_on_rule')), ('integration_level', '=', 'rate')]}"/>
                                <label for="margin" string="Margin on Rate"/>
                                <div>
                                    <field name="margin" class="oe_inline"/>%
                                </div>
                                <field name="free_over"/>
                                <field name="amount" attrs="{'required':[('free_over','!=', False)], 'invisible':[('free_over','=', False)]}"/>
                                <field name="supports_shipping_insurance" invisible="1"/>
                                <label for="shipping_insurance" String="Shipping Insurance" attrs="{'invisible': [('supports_shipping_insurance', '=', False)]}"/>
                                <div attrs="{'invisible': [('supports_shipping_insurance', '=', False)]}">
                                    <field name="shipping_insurance" class="oe_inline"/>%
                                </div>
                            </group>
                        </group>
                        <notebook>
                            <page name="pricing" string="Pricing" attrs="{'invisible': [('delivery_type', 'not in', ['fixed', 'base_on_rule'])]}">
                                <group attrs="{'invisible':[('delivery_type', '!=', 'fixed')]}">
                                    <group name="fixed_price">
                                        <field name="fixed_price"/>
                                    </group>
                                </group>
                                <group name="general" attrs="{'invisible':[('delivery_type', '!=', 'base_on_rule')]}">
                                    <field name="price_rule_ids" nolabel="1" colspan="2"/>
                                </group>
                            </page>
                            <page string="Destination Availability" name="destination">
                                <group>
                                    <p colspan="2">
                                        Filling this form allows you to filter delivery carriers according to the delivery address of your customer.
                                    </p>
                                </group>
                                <group>
                                    <group name="country_details">
                                        <field name="country_ids" widget="many2many_tags" options="{'no_open': True, 'no_create': True}"/>
                                        <field name="state_ids" widget="many2many_tags"/>
                                        <field name="zip_prefix_ids" widget="many2many_tags"/>
                                    </group>
                                </group>
                            </page>
                            <page string="Description" name="description">
                                <field name="carrier_description" placeholder="Shipping method details to be included at bottom sales orders and their confirmation emails. E.g. Instructions for customers to follow."/>
                            </page>
                        </notebook>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="action_delivery_carrier_form" model="ir.actions.act_window">
            <field name="name">Shipping Methods</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">delivery.carrier</field>
            <field name="view_mode">tree,form</field>
            <field name="context">{'search_default_group_by_provider': True}</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Define a new delivery method
              </p><p>
                Each carrier (e.g. UPS) can have several delivery methods (e.g.
                UPS Express, UPS Standard) with a set of pricing rules attached
                to each method.
              </p><p>
                These methods allow to automatically compute the delivery price
                according to your settings; on the sales order (based on the
                quotation) or the invoice (based on the delivery orders).
              </p>
            </field>
        </record>

        <record id="action_delivery_zip_prefix_list" model="ir.actions.act_window">
            <field name="name">Zip Prefix</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">delivery.zip.prefix</field>
            <field name="view_mode">tree,form</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Manage delivery zip prefixes
              </p><p>
                Delivery zip prefixes are assigned to delivery carriers to restrict
                which zips it is available to.
              </p>
            </field>
        </record>

        <menuitem action="action_delivery_carrier_form" id="menu_action_delivery_carrier_form" parent="stock.menu_delivery" sequence="1"/>
        <menuitem action="action_delivery_carrier_form" id="sale_menu_action_delivery_carrier_form" parent="sale.menu_sales_config" sequence="4"/>
        <menuitem action="action_delivery_zip_prefix_list" id="menu_delivery_zip_prefix" parent="stock.menu_delivery" groups="base.group_no_one" sequence="100"/>

        <record id="view_delivery_price_rule_form" model="ir.ui.view">
            <field name="name">delivery.price.rule.form</field>
            <field name="model">delivery.price.rule</field>
            <field name="arch" type="xml">
                <form string="Price Rules">
                    <group>
                        <field name="name" invisible="1"/>
                    </group>
                    <group>
                        <label for="variable" string="Condition"/>
                        <div class="o_row">
                            <field name="variable"/>
                            <field name="operator"/>
                            <field name="max_value"/>
                        </div>
                        <label for="list_base_price" string="Delivery Cost"/>
                        <div>
                            <field name="list_base_price" class="oe_inline"/>
                            +
                            <field name="list_price" class="oe_inline"/>
                            *
                            <field name="variable_factor" class="oe_inline"/>
                        </div>
                    </group>
                </form>
            </field>
        </record>
        <record id="view_delivery_price_rule_tree" model="ir.ui.view">
            <field name="name">delivery.price.rule.tree</field>
            <field name="model">delivery.price.rule</field>
            <field name="arch" type="xml">
                <tree string="Price Rules">
                    <field name="sequence" widget="handle" />
                    <field name="name"/>
                </tree>
            </field>
        </record>

        <record id="view_picking_withcarrier_out_form" model="ir.ui.view">
            <field name="name">delivery.stock.picking_withcarrier.form.view</field>
            <field name="model">stock.picking</field>
            <field name="inherit_id" ref="stock.view_picking_form"/>
            <field name="arch" type="xml">
              <data>
                <xpath expr="//group[@name='other_infos']" position="before">
                    <group name='carrier_data' string="Shipping Information">
                        <field name="is_return_picking" invisible="1"/>
                        <field name="carrier_id" attrs="{'readonly': [('state', 'in', ('done', 'cancel'))]}" options="{'no_create': True, 'no_open': True}"/>
                        <field name="delivery_type" attrs="{'invisible':True}"/>
                        <label for="carrier_tracking_ref"/>
                        <div name="tracking">
                            <field name="carrier_tracking_ref" class="oe_inline text-break" attrs="{'readonly': [('state', 'in', ('done', 'cancel'))]}"/>
                            <button type='object' class="fa fa-arrow-right oe_link" confirm="Cancelling a delivery may not be undoable. Are you sure you want to continue?" name="cancel_shipment" string="Cancel" attrs="{'invisible':['|','|','|',('carrier_tracking_ref','=',False),('delivery_type','in', ['fixed', 'base_on_rule']),('delivery_type','=',False),('state','not in',('done'))]}"/>
                        </div>
                        <label for="weight" string="Weight"/>
                        <div>
                            <field name="weight" class="oe_inline"/>
                            <field name="weight_uom_name" nolabel="1" class="oe_inline" style="margin-left:5px"/>
                        </div>
                        <label for="shipping_weight" string="Weight for shipping"/>
                        <div>
                            <field name="shipping_weight" class="oe_inline"/>
                            <field name="weight_uom_name" nolabel="1" class="oe_inline" style="margin-left:5px"/>
                        </div>
                    </group>
                </xpath>
                <div name="button_box" position="inside">
                    <button type="object" name="open_website_url" class="oe_stat_button" icon='fa-truck' string="Tracking"
                         attrs="{'invisible': ['|','|',('carrier_tracking_ref','=',False),('carrier_id', '=', False),('delivery_type','=','grid')]}" />
                </div>
                <xpath expr="/form/header/button[last()]" position="after">
                    <button name="send_to_shipper" string="Send to Shipper" type="object" attrs="{'invisible':['|','|','|','|',('carrier_tracking_ref','!=',False),('delivery_type','in', ['fixed', 'base_on_rule']),('delivery_type','=',False),('state','not in',('done')),('picking_type_code', '=', 'incoming')]}" data-hotkey="shift+v"/>
                </xpath>
                <xpath expr="/form/header/button[last()]" position="after">
                    <button name="print_return_label" string="Print Return Label" type="object" attrs="{'invisible':['|', '|', ('is_return_picking', '=', False),('state', '=', 'done'),('picking_type_code', '!=', 'incoming')]}" data-hotkey="shift+o"/>
                </xpath>
              </data>
              <xpath expr="//field[@name='partner_id']" position="attributes">
                  <attribute name="attrs">{'required': ['&amp;', ('delivery_type', '!=', False), ('delivery_type', 'not in', ['fixed', 'base_on_rule'])]}</attribute>
              </xpath>
            </field>
        </record>

        <record id="view_picking_withweight_internal_move_form" model="ir.ui.view">
            <field name="name">stock.picking_withweight.internal.move.form.view</field>
            <field name="model">stock.move</field>
            <field name="inherit_id" ref="stock.view_move_form"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='location_dest_id']" position="after">
                    <field name="weight"/>
                </xpath>
            </field>
        </record>

        <record id="view_move_line_tree_detailed_delivery" model="ir.ui.view">
            <field name="name">stock.move.line.tree.detailed</field>
            <field name="model">stock.move.line</field>
            <field name="inherit_id" ref="stock.view_move_line_tree_detailed"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='picking_partner_id']" position="after">
                    <field name="destination_country_code" optional="hide"/>
                    <field name="carrier_id" optional="hide"/>
                </xpath>
            </field>
        </record>

        <record id="view_quant_package_weight_form" model="ir.ui.view">
            <field name="name">stock.quant.package.weight.form</field>
            <field name="model">stock.quant.package</field>
            <field name="inherit_id" ref="stock.view_quant_package_form"/>
            <field name="arch" type="xml">
                <field name="company_id" position="before">
                    <label for="shipping_weight"/>
                    <div class="o_row" name="Shipping Weight">
                        <field name="shipping_weight" class="oe_inline"/>
                        <span><field name="weight_uom_name"/></span>
                        <span class="text-muted">(computed: <field name="weight" class="oe_inline" nolabel="1"/></span><span class="text-muted"><field name="weight_uom_name" nolabel="1" class="oe_inline"/>)</span>
                    </div>
                </field>
            </field>
        </record>

        <record id="view_order_form_with_carrier" model="ir.ui.view">
            <field name="name">delivery.sale.order.form.view.with_carrier</field>
            <field name="model">sale.order</field>
            <field name="inherit_id" ref="sale.view_order_form"/>
            <field name="arch" type="xml">
                <data>
                    <xpath expr="//field[@name='partner_id']" position='after'>
                        <field name="delivery_set" invisible="1"/>
                        <field name="is_all_service" invisible="1"/>
                        <field name="recompute_delivery_price" invisible="1"/>
                    </xpath>
                    <xpath expr="//group[@name='note_group']" position="before">
                        <div class="oe_right">
                            <button
                                string="Add shipping"
                                name="action_open_delivery_wizard"
                                type="object"
                                attrs="{'invisible': ['|', '|',('is_all_service', '=', True), ('order_line', '=', []), ('delivery_set', '=', True)]}"
                            />
                            <button
                                string="Update shipping cost"
                                name="action_open_delivery_wizard"
                                context="{'carrier_recompute':True}"
                                type="object"
                                class="text-warning btn-secondary"
                                attrs="{'invisible': ['|', '|',('is_all_service', '=', True), ('recompute_delivery_price', '=', False), ('delivery_set', '=', False)]}"
                            />
                            <button
                                string="Update shipping cost"
                                name="action_open_delivery_wizard"
                                context="{'carrier_recompute':True}"
                                type="object"
                                attrs="{'invisible': ['|', '|',('is_all_service', '=', True), ('recompute_delivery_price', '=', True), ('delivery_set', '=', False)]}"
                            />
                        </div>
                    </xpath>
                    <xpath expr="//field[@name='order_line']/form/group/group/field[@name='price_unit']" position="before">
                        <field name="recompute_delivery_price" invisible="1"/>
                        <field name="is_delivery" invisible="1"/>
                    </xpath>
                    <xpath expr="//field[@name='order_line']/tree/field[@name='price_unit']" position="before">
                        <field name="recompute_delivery_price" invisible="1"/>
                        <field name="is_delivery" invisible="1"/>
                    </xpath>
                    <xpath expr="//field[@name='order_line']/tree" position="attributes">
                        <attribute name="decoration-warning">recompute_delivery_price and is_delivery</attribute>
                    </xpath>
                </data>
            </field>
        </record>

        <record id="delivery_tracking_url_warning_form" model="ir.ui.view">
            <field name="name">delivery.carrier.warning.url.form</field>
            <field name="model">stock.picking</field>
            <field name="arch" type="xml">
                <form string="Trackers URL">
                    <group>
                        <div class="alert alert-info" role="status">
                            <p>You have multiple tracker links, they are available in the chatter.</p>
                        </div>
                    </group>
                    <footer>
                        <button string="OK" special="cancel" data-hotkey="z" class="oe_highlight"/>
                    </footer>
                </form>
            </field>
        </record>

        <record id="act_delivery_trackers_url" model="ir.actions.act_window">
            <field name="name">Display tracking links</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">stock.picking</field>
            <field name="view_id" ref="delivery.delivery_tracking_url_warning_form"/>
            <field name="view_mode">form</field>
            <field name="target">new</field>
        </record>

        <record id="vpicktree_view_tree" model="ir.ui.view">
            <field name="name">stock.picking.delivery.tree.inherit.delivery</field>
            <field name="model">stock.picking</field>
            <field name="inherit_id" ref="stock.vpicktree"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='backorder_id']" position="after">
                    <field name="carrier_tracking_ref" optional="hide"/>
                    <field name="carrier_id" optional="hide"/>
                    <field name="destination_country_code" optional="hide" string="Destination"/>
                    <field name="weight" optional="hide"/>
                    <field name="shipping_weight" optional="hide"/>
                </xpath>
            </field>
        </record>

</odoo>

```

## File: views\partner_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

        <record id="view_partner_property_form" model="ir.ui.view">
            <field name="name">res.partner.carrier.property.form.inherit</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="base.view_partner_form" />
            <field name="arch" type="xml">
                <group name="sale" position="inside">
                    <field name="property_delivery_carrier_id"/>
                </group>
            </field>
        </record>

</odoo>

```

## File: views\product_template_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<record model="ir.ui.view" id="product_template_hs_code">
    <field name="name">product.template.form.hs_code</field>
    <field name="model">product.template</field>
    <field name="inherit_id" ref="product.product_template_form_view"/>
    <field name="arch" type="xml">
	       <xpath expr="//group[@name='group_lots_and_weight']" position="inside">
                <field name="hs_code" string="HS Code"/>
                <field name="country_of_origin"/>
           </xpath>
    </field>
</record>

</odoo>

```

## File: views\report_deliveryslip.xml

```xml
<odoo>
    <template id="report_delivery_document2" inherit_id="stock.report_delivery_document">
        <xpath expr="//div[@name='div_sched_date']" position="after">
            <div t-if="o.picking_type_id.code == 'outgoing' and o.carrier_id" class="col-auto col-3 mw-100 mb-2">
                <strong>Carrier:</strong>
                <p t-field="o.carrier_id" class="m-0"/>
            </div>
            <div t-if="o.shipping_weight" class="col-auto col-3 mw-100 mb-2">
                <strong>Total Weight:</strong>
                <br/>
                <span t-field="o.shipping_weight"/>
                <span t-field="o.weight_uom_name"/>
            </div>
            <div t-if="o.carrier_tracking_ref" class="col-auto col-3 mw-100 mb-2">
                <strong>Tracking Number:</strong>
                <p t-field="o.carrier_tracking_ref" class="m-0"/>
            </div>
            <t t-set="has_hs_code" t-value="o.move_ids.filtered(lambda l: l.product_id.hs_code)"/>
        </xpath>

        <xpath expr="//table[@name='stock_move_line_table']/thead/tr" position="inside">
            <th t-if="has_hs_code"><strong>HS Code</strong></th>
        </xpath>
        <xpath expr="//table[@name='stock_move_table']/thead/tr" position="inside">
            <th t-if="has_hs_code"><strong>HS Code</strong></th>
        </xpath>
        <xpath expr="//table[@name='stock_move_table']/tbody/tr" position="inside">
            <td t-if="has_hs_code">
                <span t-field="move.product_id.hs_code"/>
            </td>
        </xpath>
    </template>

    <!--  HS Code to table rows-->
    <template id="stock_report_delivery_has_serial_move_line_inherit_delivery" inherit_id="stock.stock_report_delivery_has_serial_move_line">
        <xpath expr="//td[@name='move_line_lot_qty_done']" position="after">
            <td t-if="has_hs_code"><span t-field="move_line.product_id.hs_code"/></td>
        </xpath>
    </template>
    <template id="stock_report_delivery_aggregated_move_lines_inherit_delivery" inherit_id="stock.stock_report_delivery_aggregated_move_lines">
        <xpath expr="//td[@name='move_line_aggregated_qty_done']" position="after">
            <td t-if="has_hs_code"><span t-esc="aggregated_lines[line]['hs_code']"/></td>
        </xpath>
    </template>

    <!-- package related "section lines" -->
    <template id="stock_report_delivery_package_section_line_inherit_delivery" inherit_id="stock.stock_report_delivery_package_section_line">
        <!--  Add additional Package section line info -->
        <xpath expr="//td[@name='package_info']" position="inside">
            <t t-if="package.shipping_weight or package.weight">
                <!-- assume manually typed in value = priority -->
                <t t-if="package.shipping_weight">
                    <span> - Weight: </span>
                    <span t-field="package.shipping_weight"/>
                    <span t-field="package.weight_uom_name"/>
                </t>
                <!-- otherwise default to calculated value -->
                <t t-else="">
                    <span> - Weight (estimated): </span>
                    <span t-field="package.weight"/>
                    <span t-field="package.weight_uom_name"/>
                </t>
            </t>
        </xpath>
    </template>
    <template id="delivery_stock_report_delivery_no_package_section_line" inherit_id="stock.stock_report_delivery_no_package_section_line">
         <!-- Add additional No Package section line info -->
        <xpath expr="//td[@name='no_package_info']" position="inside">
            <t t-if="o.weight_bulk">
                <span> - Weight: </span>
                <span t-field="o.weight_bulk"/>
                <span t-field="o.weight_uom_name"/>
            </t>
        </xpath>
    </template>
</odoo>

```

## File: views\report_package_barcode.xml

```xml
<odoo>
    <template id="label_package_template_view_delivery" inherit_id="stock.label_package_template_view">
        <xpath expr="//t[@name='datamatrix_pack_date']" position="after">
            <t t-if="package.shipping_weight or package.weight">
                <t t-if="package.shipping_weight" t-set="weight_str" t-value="str(int(package.shipping_weight/package.weight_uom_rounding))"/>
                <t t-else="" t-set="weight_str" t-value="str(int(package.weight/package.weight_uom_rounding))"/>
                <t t-if="len(weight_str) &lt;= 6">
                    <t t-if="package.weight_is_kg" t-set="barcode" t-value="barcode + '310'"/>
                    <t t-else="" t-set="barcode" t-value="barcode + '320'"/>
                    <t t-set="barcode" t-value="barcode + str(len(str(package.weight_uom_rounding).split('.')[1]))"/>
                    <t t-set="barcode" t-value="barcode + '0' * (6 - len(str(weight_str))) + weight_str"/>
                </t>
^FO310,200
                <t t-if="package.shipping_weight">
^A0N,44,33^FDShipping Weight: <t t-out="package.shipping_weight"/> <t t-out="package.weight_uom_name"/>^FS
                </t>
                <t t-else="">
^A0N,44,33^FDWeight: <t t-out="package.weight"/> <t t-out="package.weight_uom_name"/>^FS
                </t>
            </t>
        </xpath>
    </template>

    <template id="report_package_barcode_delivery" inherit_id="stock.report_package_barcode">
        <xpath expr="//div[@name='datamatrix_barcode']" position="before">
            <t t-if="o.shipping_weight or o.weight">
                <t t-if="o.shipping_weight" t-set="weight_str" t-value="str(int(o.shipping_weight/o.weight_uom_rounding))"/>
                <t t-else="" t-set="weight_str" t-value="str(int(o.weight/o.weight_uom_rounding))"/>
                <t t-if="len(weight_str) &lt;= 6">
                    <t t-if="o.weight_is_kg" t-set="barcode" t-value="barcode + '310'"/>
                    <t t-else="" t-set="barcode" t-value="barcode + '320'"/>
                    <t t-set="barcode" t-value="barcode + str(len(str(o.weight_uom_rounding).split('.')[1]))"/>
                    <t t-set="barcode" t-value="barcode + '0' * (6 - len(str(weight_str))) + weight_str"/>
                </t>
            </t>
        </xpath>
        <xpath expr="//div[hasclass('o_packaging_type')]" position="after">
            <t t-if="o.valid_sscc">
                <!-- SSCC uses weight if necessary/available, standard barcode will not -->
                <div t-if="o.shipping_weight" class="col-auto"><strong>Shipping Weight: </strong><span t-field="o.shipping_weight"/> <t t-out="o.weight_uom_name"/></div>
                <div t-elif="o.weight" class="col-auto"><strong>Weight: </strong><span t-field="o.weight"/> <t t-out="o.weight_uom_name"/></div>
            </t>
            <t t-else="">
                <div t-if="o.shipping_weight" class="col-auto">
                    <strong>Shipping Weight:</strong>
                    <br/>
                    <span t-field="o.shipping_weight"/>
                    <span t-out="env['product.template']._get_weight_uom_id_from_ir_config_parameter().display_name"/>
                </div>
            </t>
        </xpath>
    </template>

    <template id="report_package_barcode_small_delivery" inherit_id="stock.report_package_barcode_small">
        <xpath expr="//div[@name='datamatrix_barcode']" position="before">
            <t t-if="o.shipping_weight or o.weight">
                <t t-if="o.shipping_weight" t-set="weight_str" t-value="str(int(o.shipping_weight/o.weight_uom_rounding))"/>
                <t t-else="" t-set="weight_str" t-value="str(int(o.weight/o.weight_uom_rounding))"/>
                <t t-if="len(weight_str) &lt;= 6">
                    <t t-if="o.weight_is_kg" t-set="barcode" t-value="barcode + '310'"/>
                    <t t-else="" t-set="barcode" t-value="barcode + '320'"/>
                    <t t-set="barcode" t-value="barcode + str(len(str(o.weight_uom_rounding).split('.')[1]))"/>
                    <t t-set="barcode" t-value="barcode + '0' * (6 - len(str(weight_str))) + weight_str"/>
                </t>
            </t>
        </xpath>
        <xpath expr="//div[@name='datamatrix_pack_type']" position="after">
            <div t-if="o.shipping_weight" class="row">Shipping Weight: <span t-field="o.shipping_weight"/> <t t-out="o.weight_uom_name"/></div>
            <div t-elif="o.weight" class="row">Weight: <span t-field="o.weight"/> <t t-out="o.weight_uom_name"/></div>
        </xpath>
        <xpath expr="//div[hasclass('o_packaging_type')]" position="after">
            <div class="row o_package_shipping_weight" t-if="o.shipping_weight">
                <div class="col-12 text-center" style="font-size:24px; font-weight:bold;"><span>Shipping Weight: </span><span t-field="o.shipping_weight"/> <t t-out="o.weight_uom_name"/></div>
            </div>
        </xpath>
    </template>
</odoo>

```

## File: views\report_shipping.xml

```xml
<odoo>
    <template id="report_shipping2" inherit_id="stock.report_picking">
        <xpath expr="//div[@name='div_sched_date']" position="after">
            <div t-if="o.picking_type_id.code == 'outgoing' and o.carrier_id" class="col-auto">
                <strong>Carrier:</strong>
                <p t-field="o.carrier_id"/>
            </div>
            <div t-if="o.weight" class="col-auto">
                <strong>Weight:</strong>
                <br/>
                <span t-field="o.weight"/>
                <span t-field="o.weight_uom_name"/>
            </div>
        </xpath>
    </template>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.delivery</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="sale.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='delivery_carrier']" position="after">
                <div class="content-group">
                    <div class="mt8" attrs="{'invisible': [('module_delivery','=',False)]}">
                        <button name="%(delivery.action_delivery_carrier_form)d" icon="fa-arrow-right" type="action" string="Shipping Methods" class="btn-link"/>
                    </div>
                 </div>
             </xpath>
        </field>
    </record>

</odoo>

```

## File: views\stock_move_line_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record model="ir.ui.view" id="stock_move_line_view_search_delivery">
        <field name="name">stock.move.line.search.delivery</field>
        <field name="model">stock.move.line</field>
        <field name="inherit_id" ref="stock.stock_move_line_view_search"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='owner_id']" position="after">
                <field name="carrier_name" invisible="1" string="Carrier name"/>
            </xpath>
            <xpath expr="//group[@name='groupby']" position="inside">
                <filter string="Carrier" name="by_carrier" context="{'group_by': 'carrier_name'}"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\stock_package_type_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record model="ir.ui.view" id="stock_package_type_form_delivery">
        <field name="name">stock.package.type.form.delivery</field>
        <field name="model">stock.package.type</field>
        <field name="inherit_id" ref="stock.stock_package_type_form"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='delivery']" position="inside">
                <group>
                    <field name="package_carrier_type"/>
                    <field name="shipper_package_code"/>
                </group>
            </xpath>
        </field>
    </record>

    <record model="ir.ui.view" id="stock_package_type_tree_delivery">
        <field name="name">stock.package.type.tree.delivery</field>
        <field name="model">stock.package.type</field>
        <field name="inherit_id" ref="stock.stock_package_type_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='sequence']" position="after">
                <field name="package_carrier_type"/>
            </xpath>
            <xpath expr="//field[@name='max_weight']" position="after">
                <field name="shipper_package_code"/>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\stock_picking_type_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record model="ir.ui.view" id="view_picking_type_form_delivery">
        <field name="name">stock.picking.type.tree.delivery</field>
        <field name="model">stock.picking.type</field>
        <field name="inherit_id" ref="stock.view_picking_type_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='sequence_code']" position="after">
                <field name="print_label" attrs="{'invisible': [('code', 'not in', ['internal', 'outgoing'])]}"/>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\stock_rule_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_stock_rule_form_delivery" model="ir.ui.view">
        <field name="name">stock.rule.tree.delivery</field>
        <field name="model">stock.rule</field>
        <field name="inherit_id" ref="stock.view_stock_rule_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='propagate_cancel']" position="after">
                <field name="propagate_carrier" groups="base.group_no_one"/>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: wizard\choose_delivery_carrier.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api, _
from odoo.exceptions import UserError


class ChooseDeliveryCarrier(models.TransientModel):
    _name = 'choose.delivery.carrier'
    _description = 'Delivery Carrier Selection Wizard'

    order_id = fields.Many2one('sale.order', required=True, ondelete="cascade")
    partner_id = fields.Many2one('res.partner', related='order_id.partner_id', required=True)
    carrier_id = fields.Many2one(
        'delivery.carrier',
        string="Shipping Method",
        required=True,
    )
    delivery_type = fields.Selection(related='carrier_id.delivery_type')
    delivery_price = fields.Float()
    display_price = fields.Float(string='Cost', readonly=True)
    currency_id = fields.Many2one('res.currency', related='order_id.currency_id')
    company_id = fields.Many2one('res.company', related='order_id.company_id')
    available_carrier_ids = fields.Many2many("delivery.carrier", compute='_compute_available_carrier', string="Available Carriers")
    invoicing_message = fields.Text(compute='_compute_invoicing_message')
    delivery_message = fields.Text(readonly=True)

    @api.onchange('carrier_id')
    def _onchange_carrier_id(self):
        self.delivery_message = False
        if self.delivery_type in ('fixed', 'base_on_rule'):
            vals = self._get_shipment_rate()
            if vals.get('error_message'):
                return {'error': vals['error_message']}
        else:
            self.display_price = 0
            self.delivery_price = 0

    @api.onchange('order_id')
    def _onchange_order_id(self):
        # fixed and base_on_rule delivery price will computed on each carrier change so no need to recompute here
        if self.carrier_id and self.order_id.delivery_set and self.delivery_type not in ('fixed', 'base_on_rule'):
            vals = self._get_shipment_rate()
            if vals.get('error_message'):
                warning = {
                    'title': '%s Error' % self.carrier_id.name,
                    'message': vals.get('error_message'),
                    'type': 'notification',
                }
                return {'warning': warning}

    @api.depends('carrier_id')
    def _compute_invoicing_message(self):
        self.ensure_one()
        if self.carrier_id.invoice_policy == 'real':
            self.invoicing_message = _('The shipping price will be set once the delivery is done.')
        else:
            self.invoicing_message = ""

    @api.depends('partner_id')
    def _compute_available_carrier(self):
        for rec in self:
            carriers = self.env['delivery.carrier'].search(['|', ('company_id', '=', False), ('company_id', '=', rec.order_id.company_id.id)])
            rec.available_carrier_ids = carriers.available_carriers(rec.order_id.partner_shipping_id) if rec.partner_id else carriers

    def _get_shipment_rate(self):
        vals = self.carrier_id.rate_shipment(self.order_id)
        if vals.get('success'):
            self.delivery_message = vals.get('warning_message', False)
            self.delivery_price = vals['price']
            self.display_price = vals['carrier_price']
            return {}
        return {'error_message': vals['error_message']}

    def update_price(self):
        vals = self._get_shipment_rate()
        if vals.get('error_message'):
            raise UserError(vals.get('error_message'))
        return {
            'name': _('Add a shipping method'),
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'choose.delivery.carrier',
            'res_id': self.id,
            'target': 'new',
        }

    def button_confirm(self):
        self.order_id.set_delivery_line(self.carrier_id, self.delivery_price)
        self.order_id.write({
            'recompute_delivery_price': False,
            'delivery_message': self.delivery_message,
        })

```

## File: wizard\choose_delivery_carrier_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="choose_delivery_carrier_view_form" model="ir.ui.view">
        <field name="name">choose.delivery.carrier.form</field>
        <field name="model">choose.delivery.carrier</field>
        <field name="arch" type="xml">
            <form>
                <field name='available_carrier_ids' invisible="1"/>
                <group>
                    <group>
                        <field name="carrier_id" domain="[('id', 'in', available_carrier_ids)]"/>
                        <field name="delivery_type" invisible="1"/>
                        <field name="currency_id" invisible="1"/>
                        <field name="order_id" invisible="1"/>
                        <field name='delivery_price' invisible="1"/>
                        <label for="display_price"/>
                        <div class="o_row">
                            <field name='display_price' widget="monetary" options="{'currency_field': 'currency_id'}" attrs="{'invisible': [('carrier_id','=', False)]}"/>
                            <button name="update_price" type="object" attrs="{'invisible': [('delivery_type','in', ('fixed', 'base_on_rule'))]}">
                                <i class="fa fa-arrow-right me-1"/>Get rate
                            </button>
                        </div>
                    </group>
                </group>
                <div role="alert" class="alert alert-warning" attrs="{'invisible': [('invoicing_message', '=', '')]}">
                    <field name="invoicing_message" nolabel="1"/>
                </div>
                <div role="alert" class="alert alert-info" attrs="{'invisible': [('delivery_message', '=', False)]}">
                    <field name="delivery_message" nolabel="1"/>
                </div>
                <footer>
                    <button name="button_confirm" invisible="not context.get('carrier_recompute')" type="object" string="Update" class="btn-primary" data-hotkey="q"/>
                    <button name="button_confirm" invisible="context.get('carrier_recompute')" type="object" string="Add" class="btn-primary" data-hotkey="q"/>
                    <button string="Discard" special="cancel" data-hotkey="z" class="btn-secondary"/>
                </footer>
            </form>
        </field>
    </record>
</odoo>

```

## File: wizard\choose_delivery_package.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.tools.float_utils import float_compare


class ChooseDeliveryPackage(models.TransientModel):
    _name = 'choose.delivery.package'
    _description = 'Delivery Package Selection Wizard'

    picking_id = fields.Many2one('stock.picking', 'Picking')
    delivery_package_type_id = fields.Many2one('stock.package.type', 'Delivery Package Type', check_company=True)
    shipping_weight = fields.Float('Shipping Weight', compute='_compute_shipping_weight', store=True, readonly=False)
    weight_uom_name = fields.Char(string='Weight unit of measure label', compute='_compute_weight_uom_name')
    company_id = fields.Many2one(related='picking_id.company_id')

    @api.depends('delivery_package_type_id')
    def _compute_weight_uom_name(self):
        weight_uom_id = self.env['product.template']._get_weight_uom_id_from_ir_config_parameter()
        for package in self:
            package.weight_uom_name = weight_uom_id.name

    @api.depends('delivery_package_type_id')
    def _compute_shipping_weight(self):
        for rec in self:
            move_line_ids = rec.picking_id.move_line_ids.filtered(lambda m:
                float_compare(m.qty_done, 0.0, precision_rounding=m.product_uom_id.rounding) > 0
                and not m.result_package_id
            )
            # Add package weights to shipping weight, package base weight is defined in package.type
            total_weight = rec.delivery_package_type_id.base_weight or 0.0
            for ml in move_line_ids:
                qty = ml.product_uom_id._compute_quantity(ml.qty_done, ml.product_id.uom_id)
                total_weight += qty * ml.product_id.weight
            rec.shipping_weight = total_weight

    @api.onchange('delivery_package_type_id', 'shipping_weight')
    def _onchange_package_type_weight(self):
        if self.delivery_package_type_id.max_weight and self.shipping_weight > self.delivery_package_type_id.max_weight:
            warning_mess = {
                'title': _('Package too heavy!'),
                'message': _('The weight of your package is higher than the maximum weight authorized for this package type. Please choose another package type.')
            }
            return {'warning': warning_mess}

    def action_put_in_pack(self):
        move_line_ids = self.picking_id._package_move_lines(batch_pack=self.env.context.get("batch_pack"))
        delivery_package = self.picking_id._put_in_pack(move_line_ids)
        # write shipping weight and package type on 'stock_quant_package' if needed
        if self.delivery_package_type_id:
            delivery_package.package_type_id = self.delivery_package_type_id
        if self.shipping_weight:
            delivery_package.shipping_weight = self.shipping_weight

```

## File: wizard\choose_delivery_package_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="choose_delivery_package_view_form" model="ir.ui.view">
        <field name="name">choose.delivery.package.form</field>
        <field name="model">choose.delivery.package</field>
        <field name="arch" type="xml">
            <form string="Package">
                <field name="picking_id" invisible="1"/>
                <group>
                    <field name="delivery_package_type_id"  domain="[('package_carrier_type', '=', context.get('current_package_carrier_type', 'none'))]"
                      context="{'form_view_ref':'stock.stock_package_type_form'}"/>
                    <label for="shipping_weight" attrs="{'invisible': [('delivery_package_type_id', '=', False)]}"/>
                    <div class="o_row" attrs="{'invisible': [('delivery_package_type_id', '=', False)]}" name="package_weight">
                        <field name="shipping_weight"/>
                        <field name="weight_uom_name"/>
                    </div>
                </group>
                <footer>
                    <button name="action_put_in_pack" type="object" string="Save" class="btn-primary" data-hotkey="q"/>
                    <button string="Discard" special="cancel" data-hotkey="z" class="btn-secondary"/>
                </footer>
            </form>
        </field>
    </record>

</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import choose_delivery_package
from . import choose_delivery_carrier

```

