# Odoo Module: delivery

Category: Sales/Delivery

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models
from . import wizard

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.


{
    'name': 'Delivery Costs',
    'version': '1.0',
    'category': 'Sales/Delivery',
    'description': """
Allows you to add delivery methods in sale orders.
==================================================
You can define your own carrier for prices.
The system is able to add and compute the shipping line.
""",
    'depends': ['sale'],
    'data': [
        'data/delivery_data.xml',
        'security/ir.model.access.csv',
        'security/ir_rules.xml',

        'report/ir_actions_report_templates.xml',

        'views/delivery_carrier_views.xml',
        'views/delivery_price_rule_views.xml',
        'views/delivery_zip_prefix_views.xml',
        'views/res_partner_views.xml',
        'views/sale_order_views.xml',

        'wizard/res_config_settings_views.xml',
        'wizard/choose_delivery_carrier_views.xml',
    ],
    'demo': ['data/delivery_demo.xml'],
    'installable': True,
    'license': 'LGPL-3',
}

```

## File: controllers\location_selector.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.http import Controller, request, route


class LocationSelectorController(Controller):

    @route('/delivery/set_pickup_location', type='json', auth='user')
    def delivery_set_pickup_location(self, order_id, pickup_location_data):
        """ Fetch the order and set the pickup location on the current order.

        :param int order_id: The sales order, as a `sale.order` id.
        :param str pickup_location_data: The JSON-formatted pickup location address.
        :return: None
        """
        order = request.env['sale.order'].browse(order_id)
        order._set_pickup_location(pickup_location_data)

    @route('/delivery/get_pickup_locations', type='json', auth='user')
    def delivery_get_pickup_locations(self, order_id, zip_code=None):
        """ Fetch the order and return the pickup locations close to a given zip code.

        Determine the country based on GeoIP or fallback on the order's delivery address' country.

        :param int order_id: The sales order, as a `sale.order` id.
        :param int zip_code: The zip code to look up to.
        :return: The close pickup locations data.
        :rtype: dict
        """
        order = request.env['sale.order'].browse(order_id)
        if request.geoip.country_code:
            country = request.env['res.country'].search(
                [('code', '=', request.geoip.country_code)], limit=1,
            )
        else:
            country = order.partner_shipping_id.country_id
        return order._get_pickup_locations(zip_code, country)

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import location_selector

```

## File: data\delivery_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="product_category_deliveries" model="product.category">
        <field name="parent_id" ref="product.product_category_all"/>
        <field name="name">Deliveries</field>
    </record>
    <record id="product_product_delivery" model="product.product">
        <field name="name">Standard delivery</field>
        <field name="default_code">Delivery_007</field>
        <field name="type">service</field>
        <field name="categ_id" ref="delivery.product_category_deliveries"/>
        <field name="sale_ok" eval="False"/>
        <field name="purchase_ok" eval="False"/>
        <field name="list_price">0.0</field>
        <field name="invoice_policy">order</field>
    </record>
    <record id="free_delivery_carrier" model="delivery.carrier">
        <field name="name">Standard delivery</field>
        <field name="fixed_price">0.0</field>
        <field name="sequence">1</field>
        <field name="delivery_type">fixed</field>
        <field name="product_id" ref="delivery.product_product_delivery"/>
    </record>

</odoo>

```

## File: data\delivery_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

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
        <record id="product.product_product_local_delivery" model="product.product">
            <field name="categ_id" ref="delivery.product_category_deliveries"/>
        </record>

        <record id="delivery_local_delivery" model="delivery.carrier">
            <field name="name">Local Delivery</field>
            <field name="fixed_price">5.0</field>
            <field name="free_over" eval="True"/>
            <field name="amount">50</field>
            <field name="sequence">4</field>
            <field name="delivery_type">fixed</field>
            <field name="product_id" ref="product.product_product_local_delivery"/>
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

        <function model="ir.default" name="set" eval="('res.partner', 'property_delivery_carrier_id', obj().env.ref('delivery.delivery_local_delivery').id)"/>

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
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import psycopg2
import re

from odoo import _, api, fields, models, Command, SUPERUSER_ID
from odoo.exceptions import UserError
from odoo.modules.registry import Registry
from odoo.tools.safe_eval import safe_eval


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
    delivery_type = fields.Selection(
        [('base_on_rule', 'Based on Rules'), ('fixed', 'Fixed Price')],
        string='Provider',
        default='fixed',
        required=True,
    )
    integration_level = fields.Selection([('rate', 'Get Rate'), ('rate_and_ship', 'Get Rate and Create Shipment')], string="Integration Level", default='rate_and_ship', help="Action while validating Delivery Orders")
    prod_environment = fields.Boolean("Environment", help="Set to True if your credentials are certified for production.")
    debug_logging = fields.Boolean('Debug logging', help="Log requests in order to ease debugging")
    company_id = fields.Many2one('res.company', string='Company', related='product_id.company_id', store=True, readonly=False)
    product_id = fields.Many2one('product.product', string='Delivery Product', required=True, ondelete='restrict')
    tracking_url = fields.Char(string='Tracking Link', help="This option adds a link for the customer in the portal to track their package easily. Use <shipmenttrackingnumber> as a placeholder in your URL.")
    currency_id = fields.Many2one(related='product_id.currency_id')

    invoice_policy = fields.Selection(
        selection=[('estimated', "Estimated cost")],
        string="Invoicing Policy",
        default='estimated',
        required=True,
        help="Estimated Cost: the customer will be invoiced the estimated cost of the shipping.",
    )

    country_ids = fields.Many2many('res.country', 'delivery_carrier_country_rel', 'carrier_id', 'country_id', 'Countries')
    state_ids = fields.Many2many('res.country.state', 'delivery_carrier_state_rel', 'carrier_id', 'state_id', 'States')
    zip_prefix_ids = fields.Many2many(
        'delivery.zip.prefix', 'delivery_zip_prefix_rel', 'carrier_id', 'zip_prefix_id', 'Zip Prefixes',
        help="Prefixes of zip codes that this carrier applies to. Note that regular expressions can be used to support countries with varying zip code lengths, i.e. '$' can be added to end of prefix to match the exact zip (e.g. '100$' will only match '100' and not '1000')")

    max_weight = fields.Float('Max Weight', help="If the total weight of the order is over this weight, the method won't be available.")
    weight_uom_name = fields.Char(string='Weight unit of measure label', compute='_compute_weight_uom_name')
    max_volume = fields.Float('Max Volume', help="If the total volume of the order is over this volume, the method won't be available.")
    volume_uom_name = fields.Char(string='Volume unit of measure label', compute='_compute_volume_uom_name')
    must_have_tag_ids = fields.Many2many(string='Must Have Tags', comodel_name='product.tag', relation='product_tag_delivery_carrier_must_have_rel',
                                         help="The method is available only if at least one product of the order has one of these tags.")
    excluded_tag_ids = fields.Many2many(string='Excluded Tags', comodel_name='product.tag', relation='product_tag_delivery_carrier_excluded_rel',
                                        help="The method is NOT available if at least one product of the order has one of these tags.")

    carrier_description = fields.Text(
        'Carrier Description', translate=True,
        help="A description of the delivery method that you want to communicate to your customers on the Sales Order and sales confirmation email."
             "E.g. instructions for customers to follow.")

    margin = fields.Float(help='This percentage will be added to the shipping price.')
    fixed_margin = fields.Float(help='This fixed amount will be added to the shipping price.')
    free_over = fields.Boolean('Free if order amount is above', help="If the order total amount (shipping excluded) is above or equal to this value, the customer benefits from a free shipping", default=False)
    amount = fields.Float(
        string="Amount",
        default=1000,
        help="Amount of the order to benefit from a free shipping, expressed in the company currency",
    )

    can_generate_return = fields.Boolean(compute="_compute_can_generate_return")
    return_label_on_delivery = fields.Boolean(string="Generate Return Label", help="The return label is automatically generated at the delivery.")
    get_return_label_from_portal = fields.Boolean(string="Return Label Accessible from Customer Portal", help="The return label can be downloaded by the customer from the customer portal.")

    supports_shipping_insurance = fields.Boolean(compute="_compute_supports_shipping_insurance")
    shipping_insurance = fields.Integer(
        "Insurance Percentage",
        help="Shipping insurance is a service which may reimburse senders whose parcels are lost, stolen, and/or damaged in transit.",
        default=0
    )

    price_rule_ids = fields.One2many(
        'delivery.price.rule', 'carrier_id', 'Pricing Rules', copy=True
    )

    _sql_constraints = [
        ('margin_not_under_100_percent', 'CHECK (margin >= -1)', 'Margin cannot be lower than -100%'),
        ('shipping_insurance_is_percentage', 'CHECK(shipping_insurance >= 0 AND shipping_insurance <= 100)', "The shipping insurance must be a percentage between 0 and 100."),
    ]

    @api.constrains('must_have_tag_ids', 'excluded_tag_ids')
    def _check_tags(self):
        for carrier in self:
            if carrier.must_have_tag_ids & carrier.excluded_tag_ids:
                raise UserError(_("Carrier %s cannot have the same tag in both Must Have Tags and Excluded Tags.") % carrier.name)

    def _compute_weight_uom_name(self):
        self.weight_uom_name = self.env['product.template']._get_weight_uom_name_from_ir_config_parameter()

    def _compute_volume_uom_name(self):
        self.volume_uom_name = self.env['product.template']._get_volume_uom_name_from_ir_config_parameter()

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
        if not self._match(order.partner_shipping_id, order):
            return False

        if self.delivery_type == 'base_on_rule':
            return self.rate_shipment(order).get('success')

        return True

    def available_carriers(self, partner, order):
        return self.filtered(lambda c: c._match(partner, order))

    def _match(self, partner, order):
        self.ensure_one()
        return self._match_address(partner) and self._match_must_have_tags(order) and self._match_excluded_tags(order) and self._match_weight(order) and self._match_volume(order)

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

    def _match_must_have_tags(self, order):
        self.ensure_one()
        return all(tag in order.order_line.product_id.all_product_tag_ids for tag in self.must_have_tag_ids)

    def _match_excluded_tags(self, order):
        self.ensure_one()
        return not any(tag in order.order_line.product_id.all_product_tag_ids for tag in self.excluded_tag_ids)

    def _match_weight(self, order):
        self.ensure_one()
        return not self.max_weight or sum(order_line.product_id.weight * order_line.product_qty for order_line in order.order_line) <= self.max_weight

    def _match_volume(self, order):
        self.ensure_one()
        return not self.max_volume or sum(order_line.product_id.volume * order_line.product_qty for order_line in order.order_line) <= self.max_volume

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

    @api.onchange('country_ids')
    def _onchange_country_ids(self):
        self.state_ids -= self.state_ids.filtered(
            lambda state: state._origin.id not in self.country_ids.state_ids.ids
        )
        if not self.country_ids:
            self.zip_prefix_ids = [Command.clear()]

    def copy_data(self, default=None):
        vals_list = super().copy_data(default=default)
        return [dict(vals, name=self.env._("%s (copy)", carrier.name)) for carrier, vals in zip(self, vals_list)]

    def _get_delivery_type(self):
        """Return the delivery type.

        This method needs to be overridden by a delivery carrier module if the delivery type is not
        stored on the field `delivery_type`.
        """
        self.ensure_one()
        return self.delivery_type

    def _apply_margins(self, price):
        self.ensure_one()
        if self.delivery_type == 'fixed':
            return float(price)
        order = self.env.context.get('order', self.env['sale.order'])
        fixed_margin_in_sale_currency = self._compute_currency(order, self.fixed_margin, 'company_to_pricelist') if order else self.fixed_margin
        return float(price) * (1.0 + self.margin) + fixed_margin_in_sale_currency

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
            res['price'] = self.with_context(order=order)._apply_margins(res['price'])
            # save the real price in case a free_over rule overide it to 0
            res['carrier_price'] = res['price']
            # free when order is large enough
            amount_without_delivery = order._compute_amount_total_without_delivery()
            if (
                res['success']
                and self.free_over
                and self.delivery_type != 'base_on_rule'
                and self._compute_currency(order, amount_without_delivery, 'pricelist_to_company') >= self.amount
            ):
                res['warning_message'] = _('The shipping is free since the order amount exceeds %.2f.', self.amount)
                res['price'] = 0.0
            return res
        else:
            return {
                'success': False,
                'price': 0.0,
                'error_message': _('Error: this delivery method is not available.'),
                'warning_message': False,
            }

    def log_xml(self, xml_string, func):
        self.ensure_one()

        if self.debug_logging:
            self.env.flush_all()
            db_name = self._cr.dbname

            # Use a new cursor to avoid rollback that could be caused by an upper method
            try:
                db_registry = Registry(db_name)
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

    # ----------------------------------- #
    # Based on rule delivery type methods #
    # ----------------------------------- #

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
        # weight is either,
        # 1- weight chosen by user in choose.delivery.carrier wizard passed by context
        # 2- saved weight to use on sale order
        # 3- total order line weight as fallback
        weight = self.env.context.get('order_weight') or order.shipping_weight or weight
        return self._get_price_from_picking(total, weight, volume, quantity, wv=wv)

    def _get_price_dict(self, total, weight, volume, quantity, wv=0.):
        '''Hook allowing to retrieve dict to be used in _get_price_from_picking() function.
        Hook to be overridden when we need to add some field to product and use it in variable factor from price rules. '''
        return {
            'price': total,
            'volume': volume,
            'weight': weight,
            'wv': wv or volume * weight,
            'quantity': quantity
        }

    def _get_price_from_picking(self, total, weight, volume, quantity, wv=0.):
        price = 0.0
        criteria_found = False
        price_dict = self._get_price_dict(total, weight, volume, quantity, wv=wv)
        for line in self.price_rule_ids:
            test = safe_eval(line.variable + line.operator + str(line.max_value), price_dict)
            if test:
                price = line.list_base_price + line.list_price * price_dict[line.variable_factor]
                criteria_found = True
                break
        if not criteria_found:
            raise UserError(_("Not available for current order"))

        return price

```

## File: models\delivery_price_rule.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models

from odoo.tools import format_amount

VARIABLE_SELECTION = [
    ('weight', "Weight"),
    ('volume', "Volume"),
    ('wv', "Weight * Volume"),
    ('price', "Price"),
    ('quantity', "Quantity"),
]


class PriceRule(models.Model):
    _name = "delivery.price.rule"
    _description = "Delivery Price Rules"
    _order = 'sequence, list_price, id'

    @api.depends('variable', 'operator', 'max_value', 'list_base_price', 'list_price', 'variable_factor', 'currency_id')
    def _compute_name(self):
        for rule in self:
            name = 'if %s %s %.02f then' % (rule.variable, rule.operator, rule.max_value)
            if rule.currency_id:
                base_price = format_amount(self.env, rule.list_base_price, rule.currency_id)
                price = format_amount(self.env, rule.list_price, rule.currency_id)
            else:
                base_price = "%.2f" % rule.list_base_price
                price = "%.2f" % rule.list_price
            if rule.list_base_price and not rule.list_price:
                name = '%s fixed price %s' % (name, base_price)
            elif rule.list_price and not rule.list_base_price:
                name = '%s %s times %s' % (name, price, rule.variable_factor)
            else:
                name = '%s fixed price %s plus %s times %s' % (
                    name, base_price, price, rule.variable_factor
                )
            rule.name = name

    name = fields.Char(compute='_compute_name')
    sequence = fields.Integer(required=True, default=10)
    carrier_id = fields.Many2one('delivery.carrier', 'Carrier', required=True, ondelete='cascade')
    currency_id = fields.Many2one(related='carrier_id.currency_id')

    variable = fields.Selection(selection=VARIABLE_SELECTION, required=True, default='quantity')
    operator = fields.Selection([('==', '='), ('<=', '<='), ('<', '<'), ('>=', '>='), ('>', '>')], required=True, default='<=')
    max_value = fields.Float('Maximum Value', required=True)
    list_base_price = fields.Float(string='Sale Base Price', digits='Product Price', required=True, default=0.0)
    list_price = fields.Float('Sale Price', digits='Product Price', required=True, default=0.0)
    variable_factor = fields.Selection(
        selection=VARIABLE_SELECTION, string="Variable Factor", required=True, default='weight'
    )

```

## File: models\delivery_zip_prefix.py

```python
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
        return super().create(vals_list)

    def write(self, vals):
        if 'name' in vals:
            vals['name'] = vals['name'].upper()
        return super().write(vals)

    _sql_constraints = [
        ('name_uniq', 'unique (name)', "Prefix already exists!"),
    ]

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

## File: models\res_partner.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResPartner(models.Model):
    _inherit = 'res.partner'

    property_delivery_carrier_id = fields.Many2one('delivery.carrier', company_dependent=True, string="Delivery Method", help="Default delivery method used in sales orders.")

```

## File: models\sale_order.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json

from odoo import _, api, fields, models
from odoo.exceptions import UserError


class SaleOrder(models.Model):
    _inherit = 'sale.order'

    pickup_location_data = fields.Json()
    carrier_id = fields.Many2one('delivery.carrier', string="Delivery Method", check_company=True, help="Fill this field if you plan to invoice the shipping based on picking.")
    delivery_message = fields.Char(readonly=True, copy=False)
    delivery_set = fields.Boolean(compute='_compute_delivery_state')
    recompute_delivery_price = fields.Boolean('Delivery cost should be recomputed')
    is_all_service = fields.Boolean("Service Product", compute="_compute_is_service_products")
    shipping_weight = fields.Float("Shipping Weight", compute="_compute_shipping_weight", store=True, readonly=False)

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

    def set_delivery_line(self, carrier, amount):
        self._remove_delivery_line()
        for order in self:
            order.carrier_id = carrier.id
            order._create_delivery_line(carrier, amount)
        return True

    def _set_pickup_location(self, pickup_location_data):
        """ Set the pickup location on the current order.

        Note: self.ensure_one()

        :param str pickup_location_data: The JSON-formatted pickup location address.
        :return: None
        """
        self.ensure_one()
        use_locations_fname = f'{self.carrier_id.delivery_type}_use_locations'
        if hasattr(self.carrier_id, use_locations_fname):
            use_location = getattr(self.carrier_id, use_locations_fname)
            if use_location and pickup_location_data:
                pickup_location = json.loads(pickup_location_data)
            else:
                pickup_location = None
            self.pickup_location_data = pickup_location

    def _get_pickup_locations(self, zip_code=None, country=None, **kwargs):
        """ Return the pickup locations of the delivery method close to a given zip code.

        Use provided `zip_code` and `country` or the order's delivery address to determine the zip
        code and the country to use.

        Note: self.ensure_one()

        :param int zip_code: The zip code to look up to, optional.
        :param res.country country: The country to look up to, required if `zip_code` is provided.
        :return: The close pickup locations data.
        :rtype: dict
        """
        self.ensure_one()
        if zip_code:
            assert country  # country is required if zip_code is provided.
            partner_address = self.env['res.partner'].new({
                'active': False,
                'country_id': country.id,
                'zip': zip_code,
            })
        else:
            partner_address = self.partner_shipping_id
        try:
            error = {'error': _("No pick-up points are available for this delivery address.")}
            function_name = f'_{self.carrier_id.delivery_type}_get_close_locations'
            if not hasattr(self.carrier_id, function_name):
                return error
            pickup_locations = getattr(self.carrier_id, function_name)(partner_address, **kwargs)
            if not pickup_locations:
                return error
            return {'pickup_locations': pickup_locations}
        except UserError as e:
            return {'error': str(e)}

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
                'default_total_weight': self._get_estimated_weight()
            }
        }

    def _action_confirm(self):
        for order in self:
            order_location = order.pickup_location_data

            if not order_location:
                continue

            # Retrieve all the data : name, street, city, state, zip, country.
            name = order_location.get('name') or order.partner_shipping_id.name
            street = order_location['street']
            city = order_location['city']
            zip_code = order_location['zip_code']
            country_code = order_location['country_code']
            country = order.env['res.country'].search([('code', '=', country_code)]).id
            state = order.env['res.country.state'].search([
                ('code', '=', order_location['state']),
                ('country_id', '=', country),
            ]).id if (order_location.get('state') and country) else None
            parent_id = order.partner_shipping_id.id
            email = order.partner_shipping_id.email
            phone = order.partner_shipping_id.phone

            # Check if the current partner has a partner of type 'delivery' with the same address.
            existing_partner = order.env['res.partner'].search([
                ('street', '=', street),
                ('city', '=', city),
                ('state_id', '=', state),
                ('country_id', '=', country),
                ('parent_id', '=', parent_id),
                ('type', '=', 'delivery'),
            ], limit=1)

            shipping_partner = existing_partner or order.env['res.partner'].create({
                'parent_id': parent_id,
                'type': 'delivery',
                'name': name,
                'street': street,
                'city': city,
                'state_id': state,
                'zip': zip_code,
                'country_id': country,
                'email': email,
                'phone': phone,
            })
            order.with_context(update_delivery_shipping_partner=True).write({'partner_shipping_id': shipping_partner})
        return super()._action_confirm()

    def _prepare_delivery_line_vals(self, carrier, price_unit):
        context = {}
        if self.partner_id:
            # set delivery detail in the customer language
            context['lang'] = self.partner_id.lang
            carrier = carrier.with_context(lang=self.partner_id.lang)

        # Apply fiscal position
        taxes = carrier.product_id.taxes_id._filter_taxes_by_company(self.company_id)
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
            'price_unit': price_unit,
            'product_uom_qty': 1,
            'product_uom': carrier.product_id.uom_id.id,
            'product_id': carrier.product_id.id,
            'tax_id': [(6, 0, taxes_ids)],
            'is_delivery': True,
        }
        if carrier.free_over and self.currency_id.is_zero(price_unit) :
            values['name'] = _('%s\nFree Shipping', values['name'])
        if self.order_line:
            values['sequence'] = self.order_line[-1].sequence + 1
        del context
        return values

    def _create_delivery_line(self, carrier, price_unit):
        values = self._prepare_delivery_line_vals(carrier, price_unit)
        return self.env['sale.order.line'].sudo().create(values)

    @api.depends('order_line.product_uom_qty', 'order_line.product_uom')
    def _compute_shipping_weight(self):
        for order in self:
            order.shipping_weight = order._get_estimated_weight()

    def _get_estimated_weight(self):
        self.ensure_one()
        weight = 0.0
        for order_line in self.order_line.filtered(lambda l: l.product_id.type == 'consu' and not l.is_delivery and not l.display_type and l.product_uom_qty > 0):
            weight += order_line.product_qty * order_line.product_id.weight
        return weight

    def _update_order_line_info(self, product_id, quantity, **kwargs):
        """ Override of `sale` to recompute the delivery prices.

        :param int product_id: The product, as a `product.product` id.
        :return: The unit price price of the product, based on the pricelist of the sale order and
                 the quantity selected.
        :rtype: float
        """
        price_unit = super()._update_order_line_info(product_id, quantity, **kwargs)
        if self:
            self.onchange_order_line()
        return price_unit

```

## File: models\sale_order_line.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class SaleOrderLine(models.Model):
    _inherit = 'sale.order.line'

    is_delivery = fields.Boolean(string="Is a Delivery", default=False)
    product_qty = fields.Float(
        string='Product Qty', compute='_compute_product_qty', digits='Product Unit of Measure'
    )
    recompute_delivery_price = fields.Boolean(related='order_id.recompute_delivery_price')

    def _is_not_sellable_line(self):
        return self.is_delivery or super()._is_not_sellable_line()

    def _can_be_invoiced_alone(self):
        return super()._can_be_invoiced_alone() and not self.is_delivery

    @api.depends('product_id', 'product_uom', 'product_uom_qty')
    def _compute_product_qty(self):
        for line in self:
            if not line.product_id or not line.product_uom or not line.product_uom_qty:
                line.product_qty = 0.0
                continue
            line.product_qty = line.product_uom._compute_quantity(
                line.product_uom_qty, line.product_id.uom_id
            )

    def unlink(self):
        self.filtered('is_delivery').order_id.filtered('carrier_id').carrier_id = False
        return super().unlink()

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

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import delivery_carrier
from . import delivery_price_rule
from . import delivery_zip_prefix
from . import product_category
from . import res_partner
from . import sale_order
from . import sale_order_line

```

## File: report\ir_actions_report_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="delivery_report_saleorder_document" inherit_id="sale.report_saleorder_document">
        <span name="order_note" position="before">
            <p t-if="doc.carrier_id.carrier_description" id="carrier_description">
                <strong>Shipping Description</strong>
                <div t-out="doc.carrier_id.carrier_description"/>
            </p>
        </span>
    </template>

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
access_delivery_zip_prefix_sale_manager,delivery.zip.prefix,model_delivery_zip_prefix,base.group_partner_manager,1,1,1,1
access_delivery_price_rule_sale_manager,delivery.price.rule,model_delivery_price_rule,sales_team.group_sale_manager,1,1,1,1
access_choose_delivery_carrier,delivery.choose.delivery.carrier,model_choose_delivery_carrier,sales_team.group_sale_salesman,1,1,1,0

```

## File: security\ir_rules.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<odoo noupdate="1">

    <record model="ir.rule" id="delivery_carrier_comp_rule">
        <field name="name">Delivery Carrier multi-company</field>
        <field name="model_id" ref="model_delivery_carrier"/>
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
    </record>

</odoo>

```

## File: static\src\js\location_selector\location\location.js

```javascript
/** @odoo-module **/

import {
    LocationSchedule
} from '@delivery/js/location_selector/location_schedule/location_schedule';
import { Component } from '@odoo/owl';
import { _t } from '@web/core/l10n/translation';

export class Location extends Component {
    static components = { LocationSchedule };
    static template = 'delivery.locationSelector.location';
    static props = {
        id: String,
        number: Number,
        name: String,
        street: String,
        city: String,
        zipCode: String,
        openingHours: {
            type: Object,
            values: {
                type: Array,
                element: String,
                optional: true,
            },
        },
        additionalData: { type: Object, optional: true },
        isSelected: Boolean,
        setSelectedLocation: Function,
    };

    /**
     * Get the city and the zip code.
     *
     * @return {Object} The city and the zip code.
     */
    getCityAndZipCode() {
        return `${this.props.zipCode} ${this.props.city}`;
    }

    get openingHoursLabel() {
        return _t("Opening hours");
    }
}

```

## File: static\src\js\location_selector\location\location.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">
    <t t-name="delivery.locationSelector.location">
        <button
            t-attf-id="location-{{this.props.id}}"
            class="list-group-item list-group-item-action collapsed d-flex gap-2 gap-lg-3"
            t-att-class="{'text-bg-light': props.isSelected}"
            t-on-click="() => props.setSelectedLocation(props.id)"
            type="button"
            t-attf-data-bs-target="#collapseHours_{{props.id}}"
            aria-expanded="false"
            t-attf-aria-controls="collapseHours_{{props.id}}"
            data-bs-toggle="collapse"
        >
            <span
                class="position-absolute top-0 start-0 h-100 ps-1 bg-primary transition-base"
                t-attf-class="#{props.isSelected ? 'ms-0' : 'ms-n1' }"
            />
            <strong t-out="props.number" class="o_location_selector_number fw-bold text-center"/>
            <span class="d-flex flex-column gap-1 gap-md-0 w-100">
                <span class="d-flex flex-column">
                    <strong t-out="props.name" class="fw-bold"/>
                    <small class="opacity-75" t-out="props.street"/>
                    <small class="opacity-75" t-out="this.getCityAndZipCode()"/>
                </span>
                <small name="location_opening_hours" class="d-flex d-md-none align-items-center gap-1 fw-bold">
                    <i class="fa fa-clock-o" role="img"/>
                    <t t-out="openingHoursLabel"/>
                    <i class="o_location_selector_hours_caret fa fa-caret-up ms-auto transition-base"/>
                </small>

                <!-- Schedule -->
                <span
                    class="collapse d-md-none"
                    t-attf-id="collapseHours_{{props.id}}"
                    data-bs-parent="#o_location_selector_list_view"
                  >
                    <LocationSchedule openingHours="props.openingHours"/>
                </span>
            </span>
        </button>
    </t>
</templates>

```

## File: static\src\js\location_selector\location_list\location_list.js

```javascript
/** @odoo-module **/

import { Location } from '@delivery/js/location_selector/location/location';
import { Component, onMounted, useEffect } from '@odoo/owl';

export class LocationList extends Component {
    static components = { Location };
    static template = 'delivery.locationSelector.locationList';
    static props = {
        locations: {
            type: Array,
            element: {
                type: Object,
                values: {
                    id: String,
                    name: String,
                    openingHours: {
                        type: Object,
                        values: {
                            type: Array,
                            element: String,
                            optional: true,
                        },
                    },
                    street: String,
                    city: String,
                    zip_code: String,
                    state: { type: String, optional: true},
                    country_code: String,
                    additional_data: { type: Object, optional: true},
                    latitude: String,
                    longitude: String,
                }
            },
        },
        selectedLocationId: [String, {value: false}],
        setSelectedLocation: Function,
        validateSelection: Function,
    };

    setup() {
        onMounted(() => {
            document.getElementById(`location-${this.props.selectedLocationId}`).focus();
        });

        // Focus on the location on the list when clicking on the map marker.
        useEffect(
            (locations, selectedLocationId) => {
                const selectedLocation = locations.find(
                    l => String(l.id) === selectedLocationId
                );
                if (selectedLocation) {
                    document.getElementById(`location-${selectedLocation.id}`).focus();
                }
            },
            () => [this.props.locations, this.props.selectedLocationId]
        );
    }
}

```

## File: static\src\js\location_selector\location_list\location_list.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">
    <t t-name="delivery.locationSelector.locationList">
        <div class="d-flex flex-grow-1 h-100 h-md-auto overflow-x-auto">
            <div id="o_location_selector_list_view" class="list-group list-group-flush flex-grow-1">
                <t
                    t-foreach="this.props.locations"
                    t-as="location"
                    t-key="location.id"
                >
                    <Location
                        id="location.id.toString()"
                        number="location_index + 1"
                        name="location.name"
                        street="location.street"
                        city="location.city"
                        zipCode="location.zip_code"
                        openingHours="location.opening_hours"
                        additionalData="location.additional_data"
                        isSelected="this.props.selectedLocationId === location.id.toString()"
                        setSelectedLocation="this.props.setSelectedLocation"
                    />
                </t>
            </div>
        </div>
    </t>
</templates>

```

## File: static\src\js\location_selector\location_schedule\location_schedule.js

```javascript
/** @odoo-module **/

import { Component } from '@odoo/owl';
import { _t } from '@web/core/l10n/translation';

export class LocationSchedule extends Component {
    static template = 'delivery.locationSelector.schedule';
    static props = {
        openingHours: {
            type: Object,
            values: {
                type: Array,
                element: String,
                optional: true,
            },
        },
        wrapClass: { type: String, optional: true },
    };

    /**
     * Return the localized day's name given his index in the week.
     *
     * @param {Number} weekday - The number of the day of the week. 0 for Monday, 6 for Sunday.
     * @return {Object} the localized name of the day (long version).
     */
    getWeekDay(weekday) {
        return luxon.Info.weekdays()[weekday]
    }

    get closedLabel() {
        return _t("Closed");
    }
}

```

## File: static\src\js\location_selector\location_schedule\location_schedule.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">
    <t t-name="delivery.locationSelector.schedule">
        <span t-attf-class="o_location_selector_schedule d-grid gap-1 #{props.wrapClass}">
            <t t-foreach="this.props.openingHours" t-as="openingHour" t-key="openingHour_index">
                <t t-call="delivery.locationSelector.dailySchedule">
                    <t t-set="weekday" t-value="openingHour_index"/>
                    <t t-set="openingPeriods" t-value="openingHour_value"/>
                </t>
            </t>
        </span>
    </t>

    <t t-name="delivery.locationSelector.dailySchedule">
        <span class="d-contents">
            <small class="me-4 text-muted" t-out="getWeekDay(weekday)"/>
            <span class="o_location_selector_schedule_hours d-flex flex-wrap">
                <t t-if="openingPeriods.length">
                    <t t-foreach="openingPeriods" t-as="openingPeriod" t-key="openingPeriod_index">
                        <small t-out="openingPeriod" class="text-nowrap"/>
                    </t>
                </t>
                <t t-else="">
                    <small class="text-danger" t-out="closedLabel"/>
                </t>
            </span>
        </span>
    </t>
</templates>

```

## File: static\src\js\location_selector\location_selector_dialog\location_selector_dialog.js

```javascript
/** @odoo-module **/

import { LocationList } from '@delivery/js/location_selector/location_list/location_list';
import { MapContainer } from '@delivery/js/location_selector/map_container/map_container';
import { Component, onMounted, onWillUnmount, useEffect, useState } from '@odoo/owl';
import { browser } from '@web/core/browser/browser';
import { Dialog } from '@web/core/dialog/dialog';
import { _t } from '@web/core/l10n/translation';
import { rpc } from '@web/core/network/rpc';
import { useDebounced } from '@web/core/utils/timing';

export class LocationSelectorDialog extends Component {
    static components = { Dialog, LocationList, MapContainer };
    static template = 'delivery.locationSelector.dialog';
    static props = {
        orderId: Number,
        zipCode: String,
        selectedLocationId: { type: String, optional: true},
        save: Function,
        close: Function, // This is the close from the env of the Dialog Component
    };
    static defaultProps = {
        selectedLocationId: false,
    };

    setup() {
        this.state = useState({
            locations: [],
            error: false,
            viewMode: 'list',
            zipCode: this.props.zipCode,
            // Some APIs like FedEx use strings to identify locations.
            selectedLocationId: String(this.props.selectedLocationId),
            isSmall: this.env.isSmall,
        });

        this.getLocationUrl = '/delivery/get_pickup_locations';

        this.debouncedOnResize = useDebounced(this.updateSize, 300);
        this.debouncedSearchButton = useDebounced((zipCode) => {
            this.state.locations = [];
            this._updateLocations(zipCode);
        }, 300);

        onMounted(() => {
            browser.addEventListener('resize', this.debouncedOnResize);
            this.updateSize();
        });
        onWillUnmount(() => browser.removeEventListener('resize', this.debouncedOnResize));

        // Fetch new locations when the zip code is updated.
        useEffect(
            (zipCode) => {
                this._updateLocations(zipCode)
                return () => {
                    this.state.locations = []
                };
            },
            () => [this.state.zipCode]
        );
    }

    //--------------------------------------------------------------------------
    // Data Exchanges
    //--------------------------------------------------------------------------

    /**
     * Fetch the closest pickup locations based on the zip code.
     *
     * @private
     * @param {String} zip - The zip code used to look for close locations.
     * @return {Object} The result values.
     */
    async _getLocations(zip) {
        return rpc(this.getLocationUrl, {order_id: this.props.orderId, zip_code: zip});
    }

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Get the locations based on the zip code.
     *
     * Select the first location available if no location is currently selected or if the currently
     * selected location is not on the list anymore.
     *
     * @private
     * @param {String} zip - The zip code used to look for close locations.
     * @return {void}
     */
    async _updateLocations(zip) {
        this.state.error = false;
        const { pickup_locations, error } = await this._getLocations(zip);
        if (error) {
            this.state.error = error;
            console.error(error);
        } else {
            this.state.locations = pickup_locations;
            if (!this.state.locations.find(l => String(l.id) === this.state.selectedLocationId)) {
                this.state.selectedLocationId = this.state.locations[0]
                                                ? String(this.state.locations[0].id)
                                                : false;
            }
        }
    }

    /**
     * Find the selected location based on its id.
     *
     * @return {Object} The selected location.
     */
    get selectedLocation() {
        return this.state.locations.find(l => String(l.id) === this.state.selectedLocationId);
    }

    /**
     * Set the selectedLocationId in the state.
     *
     * @param {String} locationId
     * @return {void}
     */
    setSelectedLocation(locationId) {
        this.state.selectedLocationId = String(locationId);
    }

    /**
     * Confirm the current selected location.
     *
     * @return {void}
     */
    async validateSelection() {
        if (!this.state.selectedLocationId) return;
        const selectedLocation = this.state.locations.find(
            l => String(l.id) === this.state.selectedLocationId
        );
        await this.props.save(selectedLocation);
        this.props.close();
    }

    //--------------------------------------------------------------------------
    // User Interface
    //--------------------------------------------------------------------------

    /**
     * Determines the component to show in mobile view based on the current state.
     *
     * Returns the MapContainer component if `viewMode` is strictly equal to `map`, else return the
     * List component.
     *
     * @return {Component} The component to show in mobile view.
     */
    get mobileComponent() {
        if (this.state.viewMode === 'map') return MapContainer;
        return LocationList;
    }

    get title() {
        return _t("Choose a pick-up point");
    }

    get validationButtonLabel() {
        return _t("Choose this location");
    }

    get postalCodePlaceholder() {
        return _t("Your postal code");
    }

    get listViewButtonLabel() {
        return _t("List view");
    }

    get mapViewButtonLabel() {
        return _t("Map view");
    }

    get errorMessage() {
        return _t("No result");
    }

    get loadingMessage() {
        return _t("Loading...");
    }

    /**
     *
     * @return {void}
     */
    updateSize() {
        this.state.isSmall = this.env.isSmall;
    }
}

```

## File: static\src\js\location_selector\location_selector_dialog\location_selector_dialog.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">
    <t t-name="delivery.locationSelector.dialog">
        <Dialog
            title="title"
            bodyClass="'d-flex flex-column border-top p-0 px-md-3'"
            contentClass="'o_location_selector h-100 overflow-hidden'"
        >
            <!-- Mobile view -->
            <t t-if="this.state.isSmall">
                <!-- Search bar -->
                <t t-call="delivery.locationSelector.searchbar"/>

                <!-- "List view / Map view" navigation -->
                <div class="nav nav-tabs">
                    <button
                        class="o_location_selector_mobile_tab btn flex-grow-1 border-0 border-bottom rounded-0 py-3 bg-transparent"
                        t-att-class="{'active': this.state.viewMode === 'list'}"
                        t-on-click="() => this.state.viewMode = 'list'"
                        t-out="listViewButtonLabel"
                    />
                    <button
                        class="o_location_selector_mobile_tab btn flex-grow-1 border-0 border-bottom rounded-0 py-3 bg-transparent"
                        t-att-class="{'active' : this.state.viewMode === 'map'}"
                        t-on-click="() => this.state.viewMode = 'map'"
                        t-out="mapViewButtonLabel"
                    />
                </div>

                <!-- Component -->
                <div class="flex-grow-1 overflow-x-auto">
                    <t
                        t-if="this.state.locations.length"
                        t-component="mobileComponent"
                        locations="this.state.locations"
                        selectedLocationId="this.state.selectedLocationId.toString()"
                        setSelectedLocation.bind="setSelectedLocation"
                        validateSelection.bind="validateSelection"
                    />
                    <t t-else="">
                        <p t-if="this.state.error" class="p-3 fw-bold" t-out="errorMessage"/>
                        <div t-else="" class="position-absolute start-50 top-50 translate-middle">
                            <div class="spinner-border" role="status">
                                <span class="visually-hidden" t-out="loadingMessage"/>
                            </div>
                        </div>
                    </t>
                </div>
            </t>

            <!-- Desktop view -->
            <div t-else="" class="d-flex h-100 mx-0 mx-md-n3 overflow-hidden">

                <!-- List view -->
                <section class="o_location_selector_view col-md-4 d-flex flex-grow-1 flex-column">

                    <!-- Search bar -->
                    <t t-call="delivery.locationSelector.searchbar">
                        <t t-set="wrapClass" t-value="'d-flex'"/>
                    </t>

                    <!-- List group -->
                    <LocationList
                        t-if="this.state.locations.length"
                        locations="this.state.locations"
                        selectedLocationId="this.state.selectedLocationId.toString()"
                        setSelectedLocation.bind="setSelectedLocation"
                        validateSelection.bind="validateSelection"
                    />
                    <t t-else="">
                        <p t-if="this.state.error" class="p-3 fw-bold" t-out="errorMessage"/>
                        <div t-else="" class="position-absolute start-50 top-50 translate-middle">
                            <div class="spinner-border" role="status">
                                <span class="visually-hidden" t-out="loadingMessage"/>
                            </div>
                        </div>
                    </t>
                </section>

                <!-- Map view -->
                <section class="o_location_selector_view col-md-8 d-flex flex-grow-1 border-start pe-2">
                    <MapContainer
                        locations="this.state.locations"
                        selectedLocationId="this.state.selectedLocationId.toString()"
                        setSelectedLocation.bind="setSelectedLocation"
                        validateSelection.bind="validateSelection"
                    />
                </section>
            </div>

            <!-- Validation button in mobile view -->
            <t t-if="this.state.isSmall" t-set-slot="footer">
                <div class="w-100 m-0 border-top p-3">
                    <button
                        type="button"
                        id="submit_location_small"
                        class="btn btn-primary w-100"
                        t-att-disabled="!this.state.selectedLocationId"
                        t-on-click="validateSelection"
                        t-out="validationButtonLabel"
                    />
                </div>
            </t>
        </Dialog>
    </t>

    <t t-name="delivery.locationSelector.searchbar">
        <div role="search" class="input-group p-3 border-bottom" t-att-class="wrapClass">
            <input
                class="search-query form-control oe_search_box border-0 text-bg-light"
                t-model.lazy="this.state.zipCode"
                t-att-placeholder="postalCodePlaceholder"
            />
            <button
                t-on-click="() => this.debouncedSearchButton(this.state.zipCode)"
                aria-label="Search"
                title="Search"
                class="btn btn-light"
            >
                <i class="oi oi-search"/>
            </button>
        </div>
    </t>
</templates>

```

## File: static\src\js\location_selector\map\map.js

```javascript
/** @odoo-module **/
/*global L*/

import { Component, useEffect, useRef } from '@odoo/owl';
import { renderToString } from '@web/core/utils/render';

export class Map extends Component {
    static template = 'delivery.locationSelector.map';
    static props = {
        locations: {
            type: Array,
            element: {
                type: Object,
                values: {
                    id: String,
                    name: String,
                    openingHours: {
                        type: Object,
                        values: {
                            type: Array,
                            element: String,
                            optional: true,
                        },
                    },
                    street: String,
                    city: String,
                    zip_code: String,
                    state: { type: String, optional: true},
                    country_code: String,
                    additional_data: { type: Object, optional: true},
                    latitude: String,
                    longitude: String,
                }
            },
        },
        selectedLocationId: [String, {value: false}],
        setSelectedLocation: Function,
    };

    setup() {
        this.leafletMap = null;
        this.markers = [];
        this.mapRef = useRef('map');

        // Create the map.
        useEffect(
            () => {
                this.leafletMap = L.map(this.mapRef.el, {
                    zoom: 13,
                });
                this.leafletMap.attributionControl.setPrefix(
                    '<a href="https://leafletjs.com" title="A JavaScript library for interactive maps">Leaflet</a>'
                );
                L.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
                    maxZoom: 19,
                    attribution: "&copy; <a href='http://www.openstreetmap.org/copyright'>OpenStreetMap</a>"
                }).addTo(this.leafletMap);
                return () => {
                    this.leafletMap.remove();
                }
            },
            () => []
        );

        // Update the size of the map.
        useEffect(
            (locations) => {
                this.leafletMap.invalidateSize();
            },
            () => [this.props.locations]
        );

        // Update the markers and center the map on the selected location.
        useEffect(
            (locations, selectedLocationId) => {
                this.addMarkers(locations);
                const selectedLocation = locations.find(
                    l => String(l.id) === selectedLocationId
                );
                if (selectedLocation) {
                    // Center the Map.
                    this.leafletMap.panTo(
                        [selectedLocation.latitude, selectedLocation.longitude],
                        { animate: true }
                    );
                }
                return () => {
                    this.removeMarkers();
                };
            },
            () => [this.props.locations, this.props.selectedLocationId]
        );
    }

    /**
     * Add the markers of the closest locations on the map.
     * Binds events to the created markers.
     *
     * @param {Array} locations - The list of locations to display on the map.
     * @return {void}
     */
    addMarkers(locations) {
        for (const loc of locations) {
            const isSelected = String(loc.id) === this.props.selectedLocationId
            // Icon creation
            const iconInfo = {
                className: isSelected ? 'o_location_selector_marker_icon_selected'
                                      : 'o_location_selector_marker_icon',
                html: renderToString(
                    'delivery.locationSelector.map.marker',
                    { number: locations.indexOf(loc) + 1 },
                ),
                iconSize: [30, 40],
                iconAnchor: [15, 40],
            };

            const marker = L.marker(
                [ loc.latitude, loc.longitude ],
                {
                    icon: L.divIcon(iconInfo),
                    title: locations.indexOf(loc) + 1,
                },
            );

            // By default, the marker's zIndex is based on its latitude. This ensures the selected
            // marker is always displayed on top of all others.
            if (isSelected) marker.setZIndexOffset(100);

            marker.addTo(this.leafletMap);
            marker.addEventListener('click', () => {
                this.props.setSelectedLocation(loc.id);
            });

            this.markers.push(marker);
        }
    }

    /**
     * Remove the markers from the map and empty the markers array.
     *
     * @return {void}
     */
    removeMarkers() {
        for (const marker of this.markers) {
            marker.removeEventListener();
            this.leafletMap.removeLayer(marker);
        }
        this.markers = [];
    }

    /**
     * Find the selected location based on its id.
     *
     * @return {Object} The selected location.
     */
    get selectedLocation() {
        return this.props.locations.find(l => String(l.id) === this.props.selectedLocationId)
    }
}

```

## File: static\src\js\location_selector\map\map.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">
    <t t-name="delivery.locationSelector.map">
        <div class="o_location_selector_map w-100 flex-grow-1" t-ref="map"/>
    </t>

    <t t-name="delivery.locationSelector.map.marker">
        <svg width="30" height="40" viewBox="0 0 30 40" xmlns="http://www.w3.org/2000/svg">
            <ellipse cx="15" cy="38" rx="12" ry="2" fill="#000" fill-opacity="0.25"/>
            <path
                d="M15 39.2507C14.9216 39.1623 14.8316 39.0606 14.731 38.9461C14.323 38.482 13.7395 37.8091 13.0391 36.9752C11.6379 35.3068 9.77066 32.9967 7.90454 30.4276C6.03711 27.8567 4.17807 25.0364 2.78794 22.3465C1.38981 19.6411 0.5 17.1309 0.5 15.1621C0.5 7.06077 6.9955 0.5 14.9995 0.5C23.0004 0.5 29.5 7.06089 29.5 15.1621C29.5 17.1309 28.6102 19.6411 27.2121 22.3465C25.8219 25.0364 23.9629 27.8567 22.0955 30.4276C20.2293 32.9967 18.3621 35.3068 16.9609 36.9752C16.2605 37.8091 15.677 38.482 15.269 38.9461C15.1684 39.0606 15.0784 39.1623 15 39.2507Z"
                fill="var(--LocationSelectorMarker-background)"
                stroke="var(--LocationSelectorMarker-border-color)"
            />
            <text
                x="50%"
                y="50%"
                text-anchor="middle"
                t-out="number"
                fill="var(--LocationSelectorMarker-color)"
            />
        </svg>
    </t>
</templates>

```

## File: static\src\js\location_selector\map_container\map_container.js

```javascript
/** @odoo-module **/

import {
    LocationSchedule
} from '@delivery/js/location_selector/location_schedule/location_schedule';
import { Map } from '@delivery/js/location_selector/map/map';
import { Component, onWillStart, useState } from '@odoo/owl';
import { AssetsLoadingError, loadCSS, loadJS } from '@web/core/assets';
import { _t } from '@web/core/l10n/translation';

export class MapContainer extends Component {
    static components = { LocationSchedule, Map };
    static template = 'delivery.locationSelector.mapContainer';
    static props = {
        locations: {
            type: Array,
            element: {
                type: Object,
                values: {
                    id: String,
                    name: String,
                    openingHours: {
                        type: Object,
                        values: {
                            type: Array,
                            element: String,
                            optional: true,
                        },
                    },
                    street: String,
                    city: String,
                    zip_code: String,
                    state: { type: String, optional: true},
                    country_code: String,
                    additional_data: { type: Object, optional: true},
                    latitude: String,
                    longitude: String,
                }
            },
        },
        selectedLocationId: [String, {value: false}],
        setSelectedLocation: Function,
        validateSelection: Function,
    };

    setup() {
        this.state = useState({
            shouldLoadMap: false,
        });

        onWillStart(async () => {
            /**
             * We load the script for the map before rendering the owl component to avoid a
             * UserError if the script can't be loaded (e.g. if the customer loses the connection
             * between the rendering of the page and when he opens the location selector, or if the
             * CDN’s doesn't host the library anymore).
             */
            try {
                await Promise.all([
                    loadJS('https://unpkg.com/leaflet@1.9.4/dist/leaflet.js'),
                    loadCSS('https://unpkg.com/leaflet@1.9.4/dist/leaflet.css'),
                ])
                this.state.shouldLoadMap = true;
            } catch (error) {
                if (!(error instanceof AssetsLoadingError)) {
                    throw error;
                }
            }
        });
    }

    /**
     * Get the city and the zip code.
     *
     * @param {Number} selectedLocation - The location form which the city and the zip code
     *                                    should be taken.
     * @return {Object} The city and the zip code.
     */
    getCityAndZipCode(selectedLocation) {
        return `${selectedLocation.zip_code} ${selectedLocation.city}`;
    }

    /**
     * Find the selected location based on its id.
     *
     * @return {Object} The selected location.
     */
    get selectedLocation() {
        return this.props.locations.find(l => String(l.id) === this.props.selectedLocationId);
    }

    get errorMessage() {
        return _t("There was an error loading the map");
    }

    get chooseLocationButtonLabel() {
        return _t("Choose this location");
    }

    get openingHoursLabel() {
        return _t("Opening hours");
    }
}

```

## File: static\src\js\location_selector\map_container\map_container.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">
    <t t-name="delivery.locationSelector.mapContainer">
        <div class="d-flex flex-column h-100 w-100">
            <Map
                t-if="this.state.shouldLoadMap"
                locations="this.props.locations"
                selectedLocationId="this.props.selectedLocationId"
                setSelectedLocation="this.props.setSelectedLocation"
            />
            <div
                t-else=""
                class="d-flex justify-content-center align-items-center flex-grow-1 w-100 bg-200"
            >
                <span t-out="errorMessage"/>
            </div>

            <!-- Desktop infos -->
            <div t-if="selectedLocation" class="d-none d-md-flex justify-content-between flex-column flex-lg-row gap-3 gap-lg-0 p-4">
                <div class="col-lg-5 d-flex flex-column justify-content-between">
                    <div class="d-flex gap-2">
                        <strong
                            class="o_location_selector_number flex-shrink-0 h5 fw-bold text-center"
                            t-out="this.props.locations.indexOf(selectedLocation) + 1"
                        />
                        <div class="d-flex flex-column flex-grow-1">
                            <strong class="h5 fw-bold" t-out="selectedLocation.name"/>
                            <small t-out="selectedLocation.street"/>
                            <small t-out="this.getCityAndZipCode(selectedLocation)"/>
                        </div>
                    </div>
                    <!-- large screen and + -->
                    <button
                        type="button"
                        id="submit_location_large"
                        class="btn btn-primary d-none d-lg-block mt-3"
                        t-att-disabled="!this.props.selectedLocationId"
                        t-on-click="this.props.validateSelection"
                        t-out="chooseLocationButtonLabel"/>
                </div>

                <!-- Schedule -->
                <LocationSchedule
                    openingHours="selectedLocation.opening_hours"
                    wrapClass="'col-lg-7 flex-grow-1 flex-lg-grow-0 ps-lg-4'"
                />

                <!-- medium size screen like Tablets, etc. -->
                <button
                    type="button"
                    id="submit_location_medium"
                    class="btn btn-primary d-block d-lg-none align-self-stretch ms-lg-4"
                    t-att-disabled="!this.props.selectedLocationId"
                    t-on-click="this.props.validateSelection"
                    t-out="chooseLocationButtonLabel"/>
            </div>

            <!-- Mobile infos -->
            <button
                t-if="selectedLocation"
                class="btn collapsed d-flex d-md-none gap-2 gap-lg-3 w-100 border-0 p-3 bg-transparent text-start"
                type="button"
                data-bs-target="#map_collapseHours"
                aria-expanded="false"
                aria-controls="map_collapseHours"
                data-bs-toggle="collapse"
            >
                <strong
                    t-out="this.props.locations.indexOf(selectedLocation) + 1"
                    class="o_location_selector_number fw-bold text-center"
                />
                <span class="d-flex flex-column gap-1 w-100">
                    <span class="d-flex flex-column">
                        <strong class="fw-bold" t-out="selectedLocation.name"/>
                        <small class="text-muted">
                            <span t-out="selectedLocation.street"/>
                            <span class="d-block" t-out="this.getCityAndZipCode(selectedLocation)"/>
                        </small>
                    </span>
                    <span class="d-flex align-items-center gap-1 small fw-bold">
                        <i class="fa fa-clock-o" role="img"/>
                        <t t-out="openingHoursLabel"/>
                        <i class="o_location_selector_hours_caret fa fa-caret-up ms-auto transition-base"/>
                    </span>

                    <!-- Schedule -->
                    <span class="collapse" id="map_collapseHours">
                        <LocationSchedule openingHours="selectedLocation.opening_hours"/>
                    </span>
                </span>
            </button>
        </div>
    </t>
</templates>

```

## File: views\delivery_carrier_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_delivery_carrier_tree" model="ir.ui.view">
        <field name="name">delivery.carrier.list</field>
        <field name="model">delivery.carrier</field>
        <field name="arch" type="xml">
            <list string="Carrier">
                <field name="sequence" widget="handle"/>
                <field name="name"/>
                <field name="delivery_type"/>
                <field name="company_id" groups="base.group_multi_company"/>
                <field name="country_ids" widget="many2many_tags" optional="hide"/>
                <field name="max_weight" optional="show"/>
                <field name="max_volume" optional="hide"/>
                <field name="must_have_tag_ids" widget="many2many_tags" optional="hide"/>
                <field name="excluded_tag_ids" widget="many2many_tags" optional="hide"/>
            </list>
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
                                invisible="not prod_environment or delivery_type == 'fixed' or delivery_type == 'base_on_rule'"
                                class="oe_stat_button"
                                type="object" icon="fa-play">
                            <div class="o_stat_info o_field_widget">
                                <span class="text-success">Production</span>
                                <span class="o_stat_text">Environment</span>
                            </div>
                        </button>
                        <!-- transfer referenced here due to view inheritance issue in current master (post-saas-16) -->
                        <button name="toggle_prod_environment"
                                invisible="prod_environment or delivery_type == 'fixed' or delivery_type == 'base_on_rule'"
                                class="oe_stat_button"
                                type="object" icon="fa-stop">
                            <div class="o_stat_info o_field_widget">
                                <span class="o_stat_text o_warning_text fw-bold">Test</span>
                                <span class="o_stat_text">Environment</span>
                            </div>
                        </button>
                        <button name="toggle_debug"
                                invisible="delivery_type == 'fixed' or delivery_type == 'base_on_rule' or debug_logging"
                                class="oe_stat_button"
                                type="object" icon="fa-code">
                            <div class="o_stat_info o_field_widget">
                                <span class="o_stat_text text-danger">No debug</span>
                            </div>
                        </button>
                        <button name="toggle_debug"
                                invisible="delivery_type == 'fixed' or delivery_type == 'base_on_rule' or not debug_logging"
                                class="oe_stat_button"
                                type="object" icon="fa-code">
                            <div class="o_stat_info o_field_widget">
                                <span class="text-success">Debug requests</span>
                            </div>
                        </button>
                    </div>
                    <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                    <div class="oe_title" name="title">
                        <label for="name" string="Delivery Method"/>
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
                            <field name="integration_level" widget="radio" invisible="delivery_type == 'fixed' or delivery_type == 'base_on_rule'"/>
                            <field name="company_id" groups="base.group_multi_company"/>
                        </group>
                        <group name="delivery_details">
                            <field name="currency_id" invisible="1"/>
                            <field name="fixed_price"
                                    widget="monetary"
                                    class="oe_inline"
                                    invisible="delivery_type != 'fixed'"/>
                            <field name="margin"
                                    string="Margin on Rate"
                                    widget="percentage"
                                    class="oe_inline"
                                    invisible="delivery_type == 'fixed'"/>
                            <field name="fixed_margin"
                                    string="Additional margin"
                                    widget="monetary"
                                    invisible="delivery_type == 'fixed'"/>
                            <label for="free_over" invisible="delivery_type == 'base_on_rule'"/>
                            <div name="free_over_amount" invisible="delivery_type == 'base_on_rule'">
                                <field name="free_over"/>
                                <field name="amount"
                                        widget="monetary"
                                        class="oe_inline"
                                        invisible="not free_over"
                                        required="free_over"/>
                            </div>
                            <field name="product_id" context="{
                                'default_type': 'service',
                                'default_sale_ok': False,
                                'default_purchase_ok': False,
                                'default_invoice_policy': 'order',
                            }"/>
                            <field name="tracking_url" placeholder="i.e. https://ekartlogistics.com/shipmenttrack/&lt;shipmenttrackingnumber&gt;" invisible="delivery_type not in ('fixed', 'base_on_rule')"/>
                            <field name="invoice_policy" widget="radio" invisible="delivery_type in ('fixed', 'base_on_rule') or integration_level == 'rate'"/>
                            <field name="supports_shipping_insurance" invisible="1"/>
                            <label for="shipping_insurance" String="Shipping Insurance" invisible="not supports_shipping_insurance"/>
                            <div invisible="not supports_shipping_insurance">
                                <field name="shipping_insurance" class="oe_inline"/>%
                            </div>
                        </group>
                    </group>
                    <notebook>
                        <page name="pricing" string="Pricing" invisible="delivery_type != 'base_on_rule'">
                            <group name="general">
                                <field name="price_rule_ids" nolabel="1"/>
                            </group>
                        </page>
                        <page string="Availability" name="destination">
                            <group col="1">
                                <p>
                                    Filling this form allows you to make the shipping method available according to the content of the order or its destination.
                                </p>
                                <group>
                                    <group name="country_details" string="Destination">
                                        <field name="country_ids"
                                                widget="many2many_tags"
                                                options="{'no_open': True, 'no_create': True}"/>
                                        <field name="state_ids"
                                                widget="many2many_tags"
                                                domain="[('country_id', 'in', country_ids)]"
                                                readonly="not country_ids"
                                                force_save="1"
                                                options="{'no_create': True}"/>
                                        <field name="zip_prefix_ids"
                                                widget="many2many_tags"
                                                readonly="not country_ids"
                                                force_save="1"
                                                options="{'no_create_edit': True}"/>
                                    </group>
                                    <group name="content" string="Content">
                                        <label for="max_weight"/>
                                        <div class="o_row">
                                            <field name="max_weight" class="oe_inline"/>
                                            <field name="weight_uom_name"/>
                                        </div>
                                        <label for="max_volume"/>
                                        <div class="o_row">
                                            <field name="max_volume" class="oe_inline"/>
                                            <field name="volume_uom_name"/>
                                        </div>
                                        <field name="must_have_tag_ids"
                                                widget="many2many_tags"
                                                options="{'no_open': True, 'no_create': True}"/>
                                        <field name="excluded_tag_ids"
                                                widget="many2many_tags"
                                                options="{'no_open': True, 'no_create': True}"/>
                                    </group>
                                </group>
                                <p class="fst-italic" invisible="country_ids">
                                    Please select a country before choosing a state or a zip prefix.
                                </p>
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
        <field name="name">Delivery Methods</field>
        <field name="res_model">delivery.carrier</field>
        <field name="view_mode">list,form</field>
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

    <menuitem id="sale_menu_action_delivery_carrier_form"
        action="action_delivery_carrier_form"
        parent="sale.menu_sales_config"
        sequence="4"/>

</odoo>

```

## File: views\delivery_price_rule_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

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
                    <field name="currency_id" invisible="1"/>
                    <label for="list_base_price" string="Delivery Cost"/>
                    <div>
                        <field name="list_base_price" widget="monetary" class="oe_inline"/>
                        +
                        <field name="list_price" widget="monetary" class="oe_inline"/>
                        *
                        <field name="variable_factor" class="oe_inline"/>
                    </div>
                </group>
            </form>
        </field>
    </record>
    <record id="view_delivery_price_rule_tree" model="ir.ui.view">
        <field name="name">delivery.price.rule.list</field>
        <field name="model">delivery.price.rule</field>
        <field name="arch" type="xml">
            <list string="Price Rules">
                <field name="sequence" widget="handle" />
                <field name="name"/>
            </list>
        </field>
    </record>


</odoo>

```

## File: views\delivery_zip_prefix_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="action_delivery_zip_prefix_list" model="ir.actions.act_window">
        <field name="name">Zip Prefix</field>
        <field name="res_model">delivery.zip.prefix</field>
        <field name="view_mode">list,form</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
            Manage delivery zip prefixes
            </p><p>
            Delivery zip prefixes are assigned to delivery carriers to restrict
            which zips it is available to.
            </p>
        </field>
    </record>

</odoo>

```

## File: views\res_partner_views.xml

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

## File: views\sale_order_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_order_form_with_carrier" model="ir.ui.view">
        <field name="name">delivery.sale.order.form.view.with_carrier</field>
        <field name="model">sale.order</field>
        <field name="inherit_id" ref="sale.view_order_form"/>
        <field name="arch" type="xml">
            <field name="partner_id" position='after'>
                <field name="delivery_set" invisible="1"/>
                <field name="is_all_service" invisible="1"/>
                <field name="recompute_delivery_price" invisible="1"/>
            </field>
            <div name="so_button_below_order_lines" position="inside">
                <button
                    string="Add shipping"
                    name="action_open_delivery_wizard"
                    type="object"
                    invisible="is_all_service or not order_line or delivery_set"/>
                <button
                    string="Update shipping cost"
                    name="action_open_delivery_wizard"
                    context="{'carrier_recompute':True}"
                    type="object"
                    class="text-warning btn-secondary"
                    invisible="is_all_service or not recompute_delivery_price or not delivery_set"/>
                <button
                    string="Update shipping cost"
                    name="action_open_delivery_wizard"
                    context="{'carrier_recompute':True}"
                    type="object"
                    invisible="is_all_service or recompute_delivery_price or not delivery_set"/>
            </div>
            <xpath expr="//field[@name='order_line']/form/group/group/field[@name='price_unit']" position="before">
                <field name="recompute_delivery_price" invisible="1"/>
                <field name="is_delivery" invisible="1"/>
            </xpath>
            <xpath expr="//field[@name='order_line']/list/field[@name='price_unit']" position="before">
                <field name="recompute_delivery_price" column_invisible="True"/>
                <field name="is_delivery" column_invisible="True"/>
            </xpath>
            <xpath expr="//field[@name='order_line']/list" position="attributes">
                <attribute name="decoration-warning" add="(recompute_delivery_price and is_delivery)" separator="or"/>
            </xpath>
            <label for="commitment_date" position="before">
                <field name="shipping_weight" readonly="True"/>
            </label>
        </field>
    </record>

</odoo>

```

## File: wizard\choose_delivery_carrier.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.exceptions import UserError


class ChooseDeliveryCarrier(models.TransientModel):
    _name = 'choose.delivery.carrier'
    _description = 'Delivery Carrier Selection Wizard'

    def _get_default_weight_uom(self):
        return self.env['product.template']._get_weight_uom_name_from_ir_config_parameter()

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
    total_weight = fields.Float(string='Total Order Weight', related='order_id.shipping_weight', readonly=False)
    weight_uom_name = fields.Char(readonly=True, default=_get_default_weight_uom)

    @api.onchange('carrier_id', 'total_weight')
    def _onchange_carrier_id(self):
        self.delivery_message = False
        if self.delivery_type in ('fixed', 'base_on_rule'):
            vals = self._get_delivery_rate()
            if vals.get('error_message'):
                return {'error': vals['error_message']}
        else:
            self.display_price = 0
            self.delivery_price = 0

    @api.onchange('order_id')
    def _onchange_order_id(self):
        # fixed and base_on_rule delivery price will computed on each carrier change so no need to recompute here
        if self.carrier_id and self.order_id.delivery_set and self.delivery_type not in ('fixed', 'base_on_rule'):
            vals = self._get_delivery_rate()
            if vals.get('error_message'):
                warning = {
                    'title': _("%(carrier)s Error", carrier=self.carrier_id.name),
                    'message': vals['error_message'],
                    'type': 'notification',
                }
                return {'warning': warning}

    @api.depends('carrier_id')
    def _compute_invoicing_message(self):
        self.ensure_one()
        self.invoicing_message = ""

    @api.depends('partner_id')
    def _compute_available_carrier(self):
        for rec in self:
            carriers = self.env['delivery.carrier'].search(self.env['delivery.carrier']._check_company_domain(rec.order_id.company_id))
            rec.available_carrier_ids = carriers.available_carriers(rec.order_id.partner_shipping_id, rec.order_id) if rec.partner_id else carriers

    def _get_delivery_rate(self):
        vals = self.carrier_id.with_context(order_weight=self.total_weight).rate_shipment(self.order_id)
        if vals.get('success'):
            self.delivery_message = vals.get('warning_message', False)
            self.delivery_price = vals['price']
            self.display_price = vals['carrier_price']
            return {'no_rate': vals.get('no_rate', False)}
        return {'error_message': vals['error_message']}

    def update_price(self):
        vals = self._get_delivery_rate()
        if vals.get('error_message'):
            raise UserError(vals.get('error_message'))
        return {
            'name': _('Add a shipping method'),
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'choose.delivery.carrier',
            'res_id': self.id,
            'target': 'new',
            'context': vals,
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
                        <label name="carried_weight_label" for="total_weight" groups="product.group_stock_packaging"/>
                        <div name="carried_weight" class="o_row" groups="product.group_stock_packaging">
                            <field name="total_weight"/>
                            <field name="weight_uom_name"/>
                        </div>
                        <field name="delivery_type" invisible="1"/>
                        <field name="currency_id" invisible="1"/>
                        <field name="order_id" invisible="1"/>
                        <field name='delivery_price' invisible="1"/>
                        <label for="display_price"/>
                        <div class="o_row">
                            <field name='display_price' widget="monetary" options="{'currency_field': 'currency_id'}" invisible="not carrier_id"/>
                            <button name="update_price" type="object" invisible="delivery_type in ('fixed', 'base_on_rule')">
                                <i class="oi oi-arrow-right me-1"/>Get rate
                            </button>
                        </div>
                    </group>
                </group>
                <div role="alert" class="alert alert-warning" invisible="invoicing_message == ''">
                    <field name="invoicing_message" nolabel="1"/>
                </div>
                <div role="alert" class="alert alert-info" invisible="context.get('no_rate') or not delivery_message">
                    <field name="delivery_message" nolabel="1"/>
                </div>
                <div role="alert" class="alert alert-danger" invisible="not context.get('no_rate') or not delivery_message">
                    <field name="delivery_message" nolabel="1"/>
                </div>
                <footer>
                    <button name="button_confirm" invisible="not context.get('carrier_recompute')" type="object" string="Update" class="btn-primary" data-hotkey="q"/>
                    <button name="button_confirm" invisible="context.get('carrier_recompute')" type="object" string="Add" class="btn-primary" data-hotkey="q"/>
                    <button string="Discard" special="cancel" data-hotkey="x" class="btn-secondary"/>
                </footer>
            </form>
        </field>
    </record>
</odoo>

```

## File: wizard\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.delivery</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="sale.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <setting id="delivery" position="inside">
                <div class="content-group">
                    <div class="mt8" invisible="not module_delivery">
                        <button name="%(delivery.action_delivery_carrier_form)d"
                                type="action"
                                class="btn-link"
                                icon="oi-arrow-right"
                                string="Shipping Methods"/>
                    </div>
                 </div>
            </setting>
        </field>
    </record>

</odoo>

```

## File: wizard\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import choose_delivery_carrier

```

