# Odoo Module: delivery

Category: Sales/Delivery

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

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

from odoo import _, api, fields, models, registry, Command, SUPERUSER_ID
from odoo.exceptions import UserError
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

    @api.onchange('country_ids')
    def _onchange_country_ids(self):
        self.state_ids -= self.state_ids.filtered(
            lambda state: state._origin.id not in self.country_ids.state_ids.ids
        )
        if not self.country_ids:
            self.zip_prefix_ids = [Command.clear()]

    def copy(self, default=None):
        default = dict(default or {})
        default.setdefault('name', _("%(old_name)s (copy)", old_name=self.name))
        return super().copy(default=default)

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

from odoo import _, api, fields, models
from odoo.exceptions import UserError


class SaleOrder(models.Model):
    _inherit = 'sale.order'

    carrier_id = fields.Many2one('delivery.carrier', string="Delivery Method", check_company=True, help="Fill this field if you plan to invoice the shipping based on picking.")
    delivery_message = fields.Char(readonly=True, copy=False)
    delivery_rating_success = fields.Boolean(copy=False)
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
            values['name'] += '\n' + _('Free Shipping')
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
        for order_line in self.order_line.filtered(lambda l: l.product_id.type in ['product', 'consu'] and not l.is_delivery and not l.display_type and l.product_uom_qty > 0):
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
                <strong>Shipping Description:</strong>
                <span t-out="doc.carrier_id.carrier_description"/>
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

## File: views\delivery_carrier_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_delivery_carrier_tree" model="ir.ui.view">
        <field name="name">delivery.carrier.tree</field>
        <field name="model">delivery.carrier</field>
        <field name="arch" type="xml">
            <tree string="Carrier">
                <field name="sequence" widget="handle"/>
                <field name="name"/>
                <field name="delivery_type"/>
                <field name="company_id" groups="base.group_multi_company"/>
                <field name="country_ids" widget="many2many_tags" optional="hide"/>
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
                            <field name="product_id" context="{'default_detailed_type': 'service', 'default_sale_ok': False, 'default_purchase_ok': False, 'default_invoice_policy': 'order'}" />
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
                                <field name="price_rule_ids" nolabel="1" colspan="2"/>
                            </group>
                        </page>
                        <page string="Destination Availability" name="destination">
                            <group col="1">
                                <p>
                                    Filling this form allows you to filter delivery carriers according to the delivery address of your customer.
                                </p>
                                <group name="country_details" class="oe_inline">
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
        <field name="name">Shipping Methods</field>
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
        <field name="name">delivery.price.rule.tree</field>
        <field name="model">delivery.price.rule</field>
        <field name="arch" type="xml">
            <tree string="Price Rules">
                <field name="sequence" widget="handle" />
                <field name="name"/>
            </tree>
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
            <xpath expr="//field[@name='order_line']/tree/field[@name='price_unit']" position="before">
                <field name="recompute_delivery_price" column_invisible="True"/>
                <field name="is_delivery" column_invisible="True"/>
            </xpath>
            <xpath expr="//field[@name='order_line']/tree" position="attributes">
                <attribute name="decoration-warning">recompute_delivery_price and is_delivery</attribute>
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
            rec.available_carrier_ids = carriers.available_carriers(rec.order_id.partner_shipping_id) if rec.partner_id else carriers

    def _get_shipment_rate(self):
        vals = self.carrier_id.with_context(order_weight=self.total_weight).rate_shipment(self.order_id)
        if vals.get('success'):
            self.delivery_message = vals.get('warning_message', False)
            self.delivery_price = vals['price']
            self.display_price = vals['carrier_price']
            return {'no_rate': vals.get('no_rate', False)}
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

