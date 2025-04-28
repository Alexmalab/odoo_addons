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
        'views/product_packaging_view.xml',
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
            <field name="invoice_policy">order</field>
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

        <record id="product_product_delivery_normal" model="product.product">
            <field name="name">Normal Delivery Charges</field>
            <field name="default_code">Delivery_008</field>
            <field name="type">service</field>
            <field name="categ_id" ref="delivery.product_category_deliveries"/>
            <field name="sale_ok" eval="False"/>
            <field name="purchase_ok" eval="False"/>
            <field name="invoice_policy">order</field>
            <field name="list_price">10.0</field>
        </record>

        <record id="normal_delivery_carrier" model="delivery.carrier">
            <field name="name">Normal Delivery Charges</field>
            <field name="fixed_price">10.0</field>
            <field name="sequence">3</field>
            <field name="delivery_type">fixed</field>
            <field name="product_id" ref="delivery.product_product_delivery_normal"/>
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
            <field name="value" eval="'delivery.carrier,'+str(ref('normal_delivery_carrier'))"/>
        </record>
    </data>
</odoo>

```

## File: models\delivery_carrier.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import psycopg2

from odoo import api, fields, models, registry, SUPERUSER_ID, _

_logger = logging.getLogger(__name__)


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
    zip_from = fields.Char('Zip From')
    zip_to = fields.Char('Zip To')

    margin = fields.Float(help='This percentage will be added to the shipping price.')
    free_over = fields.Boolean('Free if order amount is above', help="If the order total amount (shipping excluded) is above or equal to this value, the customer benefits from a free shipping", default=False)
    amount = fields.Float(string='Amount', help="Amount of the order to benefit from a free shipping, expressed in the company currency")

    can_generate_return = fields.Boolean(compute="_compute_can_generate_return")
    return_label_on_delivery = fields.Boolean(string="Generate Return Label", help="The return label is automatically generated at the delivery.")
    get_return_label_from_portal = fields.Boolean(string="Return Label Accessible from Customer Portal", help="The return label can be downloaded by the customer from the customer portal.")

    _sql_constraints = [
        ('margin_not_under_100_percent', 'CHECK (margin >= -100)', 'Margin cannot be lower than -100%'),
    ]

    @api.depends('delivery_type')
    def _compute_can_generate_return(self):
        for carrier in self:
            carrier.can_generate_return = False

    def toggle_prod_environment(self):
        for c in self:
            c.prod_environment = not c.prod_environment

    def toggle_debug(self):
        for c in self:
            c.debug_logging = not c.debug_logging

    def install_more_provider(self):
        return {
            'name': 'New Providers',
            'view_mode': 'kanban,form',
            'res_model': 'ir.module.module',
            'domain': [['name', '=like', 'delivery_%'], ['name', '!=', 'delivery_barcode']],
            'type': 'ir.actions.act_window',
            'help': _('''<p class="o_view_nocontent">
                    Buy Odoo Enterprise now to get more providers.
                </p>'''),
        }

    def available_carriers(self, partner):
        return self.filtered(lambda c: c._match_address(partner))

    def _match_address(self, partner):
        self.ensure_one()
        if self.country_ids and partner.country_id not in self.country_ids:
            return False
        if self.state_ids and partner.state_id not in self.state_ids:
            return False
        if self.zip_from and (partner.zip or '').upper() < self.zip_from.upper():
            return False
        if self.zip_to and (partner.zip or '').upper() > self.zip_to.upper():
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
            if res['success'] and self.free_over and order._compute_amount_total_without_delivery() >= self.amount:
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
            self.flush()
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
        price = order.pricelist_id.get_product_price(self.product_id, 1.0, order.partner_id)
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

```

## File: models\delivery_grid.py

```python
# -*- coding: utf-8 -*-
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
        if conversion == 'company_to_pricelist':
            from_currency, to_currency = order.company_id.currency_id, order.pricelist_id.currency_id
        elif conversion == 'pricelist_to_company':
            from_currency, to_currency = order.currency_id, order.company_id.currency_id

        return from_currency, to_currency

    def _compute_currency(self, order, price, conversion):
        from_currency, to_currency = self._get_conversion_currencies(order, conversion)
        if from_currency.id == to_currency.id:
            return price
        return from_currency._convert(price, to_currency, order.company_id, order.date_order or fields.Date.today())

    def _get_price_available(self, order):
        self.ensure_one()
        self = self.sudo()
        order = order.sudo()
        total = weight = volume = quantity = 0
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
            quantity += qty
        total = (order.amount_total or 0.0) - total_delivery

        total = self._compute_currency(order, total, 'pricelist_to_company')

        return self._get_price_from_picking(total, weight, volume, quantity)

    def _get_price_dict(self, total, weight, volume, quantity):
        '''Hook allowing to retrieve dict to be used in _get_price_from_picking() function.
        Hook to be overridden when we need to add some field to product and use it in variable factor from price rules. '''
        return {
            'price': total,
            'volume': volume,
            'weight': weight,
            'wv': volume * weight,
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

## File: models\partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResPartner(models.Model):
    _inherit = 'res.partner'

    property_delivery_carrier_id = fields.Many2one('delivery.carrier', company_dependent=True, string="Delivery Method", help="Default delivery method used in sales orders.")

```

## File: models\product_packaging.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields, api


class ProductPackaging(models.Model):
    _inherit = 'product.packaging'


    def _get_default_length_uom(self):
        # TODO master delete
        return self.env['product.template']._get_length_uom_name_from_ir_config_parameter()

    def _get_default_weight_uom(self):
        return self.env['product.template']._get_weight_uom_name_from_ir_config_parameter()

    height = fields.Integer('Height')
    width = fields.Integer('Width')
    packaging_length = fields.Integer('Length')
    max_weight = fields.Float('Max Weight', help='Maximum weight shippable in this packaging')
    shipper_package_code = fields.Char('Package Code')
    package_carrier_type = fields.Selection([('none', 'No carrier integration')], string='Carrier', default='none')
    weight_uom_name = fields.Char(string='Weight unit of measure label', compute='_compute_weight_uom_name', default=_get_default_weight_uom)
    length_uom_name = fields.Char(string='Length unit of measure label', compute='_compute_length_uom_name')

    _sql_constraints = [
        ('positive_height', 'CHECK(height>=0)', 'Height must be positive'),
        ('positive_width', 'CHECK(width>=0)', 'Width must be positive'),
        ('positive_length', 'CHECK(packaging_length>=0)', 'Length must be positive'),
        ('positive_max_weight', 'CHECK(max_weight>=0.0)', 'Max Weight must be positive'),
    ]

    @api.onchange('package_carrier_type')
    def _onchange_carrier_type(self):
        carrier_id = self.env['delivery.carrier'].search([('delivery_type', '=', self.package_carrier_type)], limit=1)
        if carrier_id:
            self.shipper_package_code = carrier_id._get_default_custom_package_code()
        else:
            self.shipper_package_code = False


    def _compute_length_uom_name(self):
        # FIXME This variable does not impact any logic, it is only used for the packaging display on the form view.
        #  However, it generates some confusion for the users since this UoM will be ignored when sending the requests
        #  to the carrier server: the dimensions will be expressed with another UoM and there won't be any conversion.
        #  For instance, with Fedex, the UoM used with the package dimensions will depend on the UoM of
        #  `fedex_weight_unit`. With UPS, we will use the UoM defined on `ups_package_dimension_unit`
        self.length_uom_name = ""

    def _compute_weight_uom_name(self):
        for packaging in self:
            packaging.weight_uom_name = self.env['product.template']._get_weight_uom_name_from_ir_config_parameter()

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
        help="Standardized code for international shipping and goods declaration. At the moment, only used for the FedEx shipping provider.",
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
        return self.env['delivery.carrier']._compute_currency(self, self.amount_total - delivery_cost, 'pricelist_to_company')

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
        delivery_lines = self.env['sale.order.line'].search([('order_id', 'in', self.ids), ('is_delivery', '=', True)])
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

        # Remove delivery products from the sales order
        self._remove_delivery_line()

        for order in self:
            order.carrier_id = carrier.id
            if order.state in ('sale', 'done'):
                pending_deliveries = order.picking_ids.filtered(
                    lambda p: p.state not in ('done', 'cancel') and not any(m.origin_returned_move_id for m in p.move_lines))
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

    def _create_delivery_line(self, carrier, price_unit):
        SaleOrderLine = self.env['sale.order.line']
        context = {}
        if self.partner_id:
            # set delivery detail in the customer language
            # used in local scope translation process
            context['lang'] = self.partner_id.lang
            carrier = carrier.with_context(lang=self.partner_id.lang)

        # Apply fiscal position
        taxes = carrier.product_id.taxes_id.filtered(lambda t: t.company_id.id == self.company_id.id)
        taxes_ids = taxes.ids
        if self.partner_id and self.fiscal_position_id:
            taxes_ids = self.fiscal_position_id.map_tax(taxes, carrier.product_id, self.partner_id).ids

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
            values['name'] += _(' (Estimated Cost: %s )', self._format_currency_amount(price_unit))
        else:
            values['price_unit'] = price_unit
        if carrier.free_over and self.currency_id.is_zero(price_unit) :
            values['name'] += '\n' + _('Free Shipping')
        if self.order_line:
            values['sequence'] = self.order_line[-1].sequence + 1
        sol = SaleOrderLine.sudo().create(values)
        del context
        return sol

    def _format_currency_amount(self, amount):
        pre = post = u''
        if self.currency_id.position == 'before':
            pre = u'{symbol}\N{NO-BREAK SPACE}'.format(symbol=self.currency_id.symbol or '')
        else:
            post = u'\N{NO-BREAK SPACE}{symbol}'.format(symbol=self.currency_id.symbol or '')
        return u' {pre}{0}{post}'.format(amount, pre=pre, post=post)

    @api.depends('order_line.is_delivery', 'order_line.is_downpayment')
    def _get_invoice_status(self):
        super()._get_invoice_status()
        for order in self:
            if order.invoice_status in ['no', 'invoiced']:
                continue
            order_lines = order.order_line.filtered(lambda x: not x.is_delivery and not x.is_downpayment and not x.display_type)
            if all(line.product_id.invoice_policy == 'delivery' and line.invoice_status == 'no' for line in order_lines):
                order.invoice_status = 'no'

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

    @api.depends('product_id', 'product_uom', 'product_uom_qty')
    def _compute_product_qty(self):
        for line in self:
            if not line.product_id or not line.product_uom or not line.product_uom_qty:
                line.product_qty = 0.0
                continue
            line.product_qty = line.product_uom._compute_quantity(line.product_uom_qty, line.product_id.uom_id)

    def unlink(self):
        for line in self:
            if line.is_delivery:
                line.order_id.carrier_id = False
        super(SaleOrderLine, self).unlink()

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
        vals['carrier_id'] = self.mapped('sale_line_id.order_id.carrier_id').id
        return vals

    def _key_assign_picking(self):
        keys = super(StockMove, self)._key_assign_picking()
        return keys + (self.sale_line_id.order_id.carrier_id,)

class StockMoveLine(models.Model):
    _inherit = 'stock.move.line'

    sale_price = fields.Float(compute='_compute_sale_price')

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

## File: models\stock_picking.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json
from collections import defaultdict

from odoo import models, fields, api, _
from odoo.exceptions import UserError
from odoo.tools.sql import column_exists, create_column


class StockQuantPackage(models.Model):
    _inherit = "stock.quant.package"

    @api.depends('quant_ids')
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
            if self.env.context.get('picking_id'):
                package.weight = package_weights[package.id]
            else:
                weight = 0.0
                for quant in package.quant_ids:
                    weight += quant.quantity * quant.product_id.weight
                package.weight = weight

    def _get_default_weight_uom(self):
        return self.env['product.template']._get_weight_uom_name_from_ir_config_parameter()

    def _compute_weight_uom_name(self):
        for package in self:
            package.weight_uom_name = self.env['product.template']._get_weight_uom_name_from_ir_config_parameter()

    weight = fields.Float(compute='_compute_weight', help="Total weight of all the products contained in the package.")
    weight_uom_name = fields.Char(string='Weight unit of measure label', compute='_compute_weight_uom_name', readonly=True, default=_get_default_weight_uom)
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
            picking.shipping_weight = picking.weight_bulk + sum([pack.shipping_weight or pack.weight for pack in picking.package_ids])

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

    @api.depends('move_lines')
    def _cal_weight(self):
        for picking in self:
            picking.weight = sum(move.weight for move in picking.move_lines if move.state != 'cancel')

    def _send_confirmation_email(self):
        for pick in self:
            if pick.carrier_id:
                if pick.carrier_id.integration_level == 'rate_and_ship' and pick.picking_type_code != 'incoming':
                    pick.sudo().send_to_shipper()
            pick._check_carrier_details_compliance()
        return super(StockPicking, self)._send_confirmation_email()

    def _pre_put_in_pack_hook(self, move_line_ids):
        res = super(StockPicking, self)._pre_put_in_pack_hook(move_line_ids)
        if not res:
            if self.carrier_id:
                return self._set_delivery_packaging()
        else:
            return res

    def _set_delivery_packaging(self):
        """ This method returns an action allowing to set the product packaging and the shipping weight
         on the stock.quant.package.
        """
        self.ensure_one()
        view_id = self.env.ref('delivery.choose_delivery_package_view_form').id
        context = dict(
            self.env.context,
            current_package_carrier_type=self.carrier_id.delivery_type,
            default_picking_id=self.id
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
        if self.carrier_id.free_over and self.sale_id and self.sale_id._compute_amount_total_without_delivery() >= self.carrier_id.amount:
            res['exact_price'] = 0.0
        self.carrier_price = res['exact_price'] * (1.0 + (self.carrier_id.margin / 100.0))
        if res['tracking_number']:
            self.carrier_tracking_ref = res['tracking_number']
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
        pass

    def print_return_label(self):
        self.ensure_one()
        self.carrier_id.get_return_label(self)

    def _add_delivery_cost_to_so(self):
        self.ensure_one()
        sale_order = self.sale_id
        if sale_order and self.carrier_id.invoice_policy == 'real' and self.carrier_price:
            delivery_lines = sale_order.order_line.filtered(lambda l: l.is_delivery and l.currency_id.is_zero(l.price_unit) and l.product_id == self.carrier_id.product_id)
            if not delivery_lines:
                delivery_lines = [sale_order._create_delivery_line(self.carrier_id, self.carrier_price)]
            delivery_line = delivery_lines[0]
            delivery_line[0].write({
                'price_unit': self.carrier_price,
                # remove the estimated price from the description
                'name': self.carrier_id.with_context(lang=self.partner_id.lang).name,
            })

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
        for move in self.move_lines:
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
from . import product_packaging
from . import product_template
from . import sale_order
from . import partner
from . import stock_move
from . import stock_picking

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
access_delivery_price_rule,delivery.price.rule,model_delivery_price_rule,sales_team.group_sale_salesman,1,0,0,0
access_delivery_carrier_manager,delivery.carrier,model_delivery_carrier,sales_team.group_sale_manager,1,1,1,1
access_delivery_price_rule_manager,delivery.price.rule,model_delivery_price_rule,sales_team.group_sale_manager,1,1,1,1
access_delivery_carrier_partner_manager,delivery.carrier partner_manager,model_delivery_carrier,base.group_partner_manager,1,0,0,0
access_delivery_carrier_stock_user,delivery.carrier stock_user,model_delivery_carrier,stock.group_stock_user,1,0,0,0
access_delivery_carrier_stock_manager,delivery.carrier,model_delivery_carrier,stock.group_stock_manager,1,1,1,1
access_delivery_price_rule_stock_manager,delivery.price.rule,model_delivery_price_rule,stock.group_stock_manager,1,1,1,1
access_choose_delivery_package,access.choose.delivery.package,model_choose_delivery_package,stock.group_stock_user,1,1,1,0
access_choose_delivery_carrier,access.choose.delivery.carrier,model_choose_delivery_carrier,stock.group_stock_user,1,1,1,0

```

## File: views\delivery_portal_template.xml

```xml
<odoo>
    <template id="sale_order_portal_content_inherit_sale_stock_inherit_website_sale_delivery" name="Shipping tracking on orders followup" inherit_id="sale_stock.sale_order_portal_content_inherit_sale_stock">
        <xpath expr="//div[hasclass('o_sale_stock_picking')]/div" position="after">
            <div t-if="i.carrier_tracking_ref" class="small d-lg-inline-block">
                Tracking:
                <t t-set="multiple_carrier_tracking" t-value="i.get_multiple_carrier_tracking()"/>
                <t t-if="multiple_carrier_tracking">
                     <t t-foreach="multiple_carrier_tracking" t-as="line">
                         <a t-att-href="line[1]" target="_blank"><span t-esc="line[0]"/></a>
                         <span t-if="not line_last"> + </span>
                     </t>
                </t>
                <t t-else="">
                    <t t-if="i.carrier_tracking_url">
                        <a t-att-href="i.carrier_tracking_url" target="_blank"><span t-field="i.carrier_tracking_ref"/></a>
                    </t>
                    <t t-else="">
                        <span t-field="i.carrier_id.name"/> <span t-field="i.carrier_tracking_ref"/>
                    </t>
                </t>
            </div>
            <t t-if="i.carrier_id.get_return_label_from_portal and i.return_label_ids">
                <div>
                    <a class="ml-3"  t-attf-href="/web/content/#{i.return_label_ids[:1].id}?access_token=#{i.return_label_ids[:1].access_token}" target="_blank">Print Return Label</a>
                </div>
            </t>
        </xpath>
    </template>
</odoo>

```

## File: views\delivery_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <!-- Delivery Carriers -->
        <menuitem id="menu_delivery" name="Delivery" parent="stock.menu_stock_config_settings" groups="stock.group_stock_manager" sequence="50"/>

        <record id="action_delivery_packaging_view" model="ir.actions.act_window">
            <field name="name">Delivery Packages</field>
            <field name="res_model">product.packaging</field>
            <field name="domain">[('product_id', '=', False)]</field>
            <field name="view_ids" eval="[(5, 0, 0),
                (0, 0, {'view_mode': 'tree', 'view_id': ref('delivery.product_packaging_delivery_tree')}),
                (0, 0, {'view_mode': 'form', 'view_id': ref('delivery.product_packaging_delivery_form')})]"/>
        </record>

        <menuitem id="menu_delivery_packagings" name="Delivery Packages" parent="menu_delivery" action="action_delivery_packaging_view" groups="stock.group_tracking_lot"/>

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
                        <widget name="web_ribbon" text="Archived" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                        <div class="oe_title" name="title">
                            <label for="name" string="Name" class="oe_edit_only"/>
                            <h1>
                                <field name="name" placeholder="e.g. UPS Express"/>
                            </h1>
                        </div>
                        <group>
                            <group name="provider_details">
                                <field name="active" invisible="1"/>
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
                                <field name="product_id" context="{'default_type': 'service', 'default_sale_ok': False, 'default_purchase_ok': False, 'default_invoice_policy': 'order'}" />
                                <field name="invoice_policy" widget="radio" attrs="{'invisible': ['|', ('delivery_type', 'in', ('fixed', 'base_on_rule')), ('integration_level', '=', 'rate')]}"/>
                                <label for="margin" string="Margin on Rate"/>
                                <div>
                                    <field name="margin" class="oe_inline"/>%
                                </div>
                                <field name="free_over"/>
                                <field name="amount" attrs="{'required':[('free_over','!=', False)], 'invisible':[('free_over','=', False)]}"/>
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
                                    <field name="price_rule_ids" nolabel="1"/>
                                </group>
                            </page>
                            <page string="Destination Availability" name="destination">
                                <group>
                                    <p>
                                        Filling this form allows you to filter delivery carriers according to the delivery address of your customer.
                                    </p>
                                </group>
                                <group>
                                    <group name="country_details">
                                        <field name="country_ids" widget="many2many_tags"/>
                                        <field name="state_ids" widget="many2many_tags"/>
                                    </group>
                                    <group></group>
                                    <group name="zip_from">
                                        <field name="zip_from"/>
                                    </group>
                                    <group name="zip_to">
                                        <field name="zip_to"/>
                                    </group>
                                </group>
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

        <menuitem action="action_delivery_carrier_form" id="menu_action_delivery_carrier_form" parent="menu_delivery" sequence="1"/>
        <menuitem action="action_delivery_carrier_form" id="sale_menu_action_delivery_carrier_form" parent="sale.menu_sales_config" sequence="4"/>

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
                            <field name="carrier_tracking_ref" class="oe_inline" attrs="{'readonly': [('state', 'in', ('done', 'cancel'))]}"/>
                            <button type='object' class="fa fa-arrow-right oe_link" name="cancel_shipment" string="Cancel" attrs="{'invisible':['|','|','|',('carrier_tracking_ref','=',False),('delivery_type','in', ['fixed', 'base_on_rule']),('delivery_type','=',False),('state','not in',('done'))]}"/>
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
                    <button name="send_to_shipper" string="Send to Shipper" type="object" attrs="{'invisible':['|','|','|','|',('carrier_tracking_ref','!=',False),('delivery_type','in', ['fixed', 'base_on_rule']),('delivery_type','=',False),('state','not in',('done')),('picking_type_code', '=', 'incoming')]}"/>
                </xpath>
                <xpath expr="/form/header/button[last()]" position="after">
                    <button name="print_return_label" string="Print Return Label" type="object" attrs="{'invisible':['|', '|', ('is_return_picking', '=', False),('state', '=', 'done'),('picking_type_code', '!=', 'incoming')]}"/>
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

        <record id="view_quant_package_weight_form" model="ir.ui.view">
            <field name="name">stock.quant.package.weight.form</field>
            <field name="model">stock.quant.package</field>
            <field name="inherit_id" ref="stock.view_quant_package_form"/>
            <field name="arch" type="xml">
                <field name="company_id" position="before">
                    <field name="packaging_id" domain="[('product_id', '=', False)]"/>
                    <label for="shipping_weight"/>
                    <div class="o_row" name="Shipping Weight">
                        <field name="shipping_weight"/>
                        <span><field name="weight_uom_name"/></span>
                        <span class="text-muted">(computed: <field name="weight" nolabel="1"/></span><span class="text-muted"><field name="weight_uom_name" nolabel="1"/>)</span>
                    </div>
                </field>
            </field>
        </record>

        <record id="view_quant_package_packaging_tree" model="ir.ui.view">
            <field name="name">stock.quant.package.packaging.tree</field>
            <field name="model">stock.quant.package</field>
            <field name="inherit_id" ref="stock.view_quant_package_tree"/>
            <field name="arch" type="xml">
                <field name="display_name" position="after">
                    <field name="packaging_id"/>
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
                        <button string="OK" special="cancel" class="oe_highlight"/>
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
                <xpath expr="//page[@name='sales_purchases']//field[@name='user_id']" position="after">
                    <field name="property_delivery_carrier_id"/>
                </xpath>
            </field>
        </record>

</odoo>

```

## File: views\product_packaging_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<record model="ir.ui.view" id="product_packaging_delivery_form">
    <field name="name">product.packaging.form.delivery</field>
    <field name="model">product.packaging</field>
    <field name="inherit_id" eval="False"/>
    <field name="arch" type="xml">
        <form string="Delivery Packaging">
            <sheet>
                <label for="name"/>
                <h1>
                    <field name="name"/>
                </h1>
                <group name="delivery">
                  <group>
                    <field name="package_carrier_type"/>
                    <label for="height"/>
                    <div class="o_row" name="height">
                      <field name="height"/>
                      <span><field name="length_uom_name"/></span>
                    </div>
                    <label for="width"/>
                    <div class="o_row" name="width">
                      <field name="width"/>
                      <span><field name="length_uom_name"/></span>
                    </div>
                    <label for="packaging_length"/>
                    <div class="o_row" name="packaging_length">
                      <field name="packaging_length"/>
                      <span><field name="length_uom_name"/></span>
                    </div>
                  </group>
                  <group>
                    <label for="max_weight"/>
                    <div class="o_row" name="max_weight">
                      <field name="max_weight"/>
                      <span><field name="weight_uom_name"/></span>
                    </div>
                    <field name="barcode"/>
                    <field name="shipper_package_code"/>
                  </group>
                </group>
            </sheet>
        </form>
    </field>
</record>

<record model="ir.ui.view" id="product_packaging_delivery_tree">
    <field name="name">product.packaging.tree.delivery</field>
    <field name="model">product.packaging</field>
    <field name="inherit_id" eval="False"/>
    <field name="arch" type="xml">
        <tree string="Delivery Packages">
            <field name="sequence" widget="handle"/>
            <field name="package_carrier_type"/>
            <field name="name"/>
            <field name="height"/>
            <field name="width"/>
            <field name="packaging_length"/>
            <field name="max_weight"/>
            <field name="shipper_package_code"/>
            <field name="barcode" invisible="1"/>
        </tree>
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
            <field name="hs_code"/>
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
            <div t-if="o.picking_type_id.code == 'outgoing' and o.carrier_id" class="col-auto">
                <strong>Carrier:</strong>
                <p t-field="o.carrier_id"/>
            </div>
            <div t-if="o.shipping_weight" class="col-auto">
                <strong>Total Weight:</strong>
                <br/>
                <span t-field="o.shipping_weight"/>
                <span t-field="o.weight_uom_name"/>
            </div>
            <div t-if="o.carrier_tracking_ref" class="col-auto">
                <strong>Tracking Number:</strong>
                <p t-field="o.carrier_tracking_ref"/>
            </div>
            <t t-set="has_hs_code" t-value="o.move_lines.filtered(lambda l: l.product_id.hs_code)"/>
        </xpath>

        <xpath expr="//table[@name='stock_move_table']/thead/tr" position="inside">
            <th t-if="has_hs_code"><strong>HS Code</strong></th>
        </xpath>

        <xpath expr="//table[@name='stock_move_line_table']/thead/tr" position="inside">
            <th t-if="has_hs_code"><strong>HS Code</strong></th>
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
    <template id="report_package_barcode_delivery" inherit_id="stock.report_package_barcode">
        <xpath expr="//div[hasclass('o_packaging_type')]" position="after">
            <div t-if="o.shipping_weight" class="col-auto">
                <strong>Shipping Weight:</strong>
                <br/>
                <span t-field="o.shipping_weight"/>
                <span t-esc="env['product.template']._get_weight_uom_id_from_ir_config_parameter().display_name"/>
            </div>
        </xpath>
    </template>

    <template id="report_package_barcode_small_delivery" inherit_id="stock.report_package_barcode_small">
        <xpath expr="//div[hasclass('o_packaging_type')]" position="after">
            <div class="row o_package_shipping_weight" t-if="o.shipping_weight">
                <div class="col-12 text-center" style="font-size:24px; font-weight:bold;"><span>Shipping Weight: </span><span t-field="o.shipping_weight"/> <t t-esc="env['product.template']._get_weight_uom_id_from_ir_config_parameter().display_name"/></div>
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
        help="Choose the method to deliver your goods",
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
                                <i class="fa fa-arrow-right mr-1"/>Get rate
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
                    <button name="button_confirm" invisible="not context.get('carrier_recompute')" type="object" string="Update" class="btn-primary"/>
                    <button name="button_confirm" invisible="context.get('carrier_recompute')" type="object" string="Add" class="btn-primary"/>
                    <button string="Discard" special="cancel" class="btn-secondary"/>
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

    @api.model
    def default_get(self, fields_list):
        defaults = super().default_get(fields_list)
        if 'shipping_weight' in fields_list:
            picking = self.env['stock.picking'].browse(defaults.get('picking_id'))
            move_line_ids = picking.move_line_ids.filtered(lambda m:
                float_compare(m.qty_done, 0.0, precision_rounding=m.product_uom_id.rounding) > 0
                and not m.result_package_id
            )
            total_weight = 0.0
            for ml in move_line_ids:
                qty = ml.product_uom_id._compute_quantity(ml.qty_done, ml.product_id.uom_id)
                total_weight += qty * ml.product_id.weight
            defaults['shipping_weight'] = total_weight
        return defaults

    picking_id = fields.Many2one('stock.picking', 'Picking')
    delivery_packaging_id = fields.Many2one('product.packaging', 'Delivery Packaging', check_company=True)
    shipping_weight = fields.Float('Shipping Weight')
    weight_uom_name = fields.Char(string='Weight unit of measure label', compute='_compute_weight_uom_name')
    company_id = fields.Many2one(related='picking_id.company_id')

    @api.depends('delivery_packaging_id')
    def _compute_weight_uom_name(self):
        weight_uom_id = self.env['product.template']._get_weight_uom_id_from_ir_config_parameter()
        for package in self:
            package.weight_uom_name = weight_uom_id.name

    @api.onchange('delivery_packaging_id', 'shipping_weight')
    def _onchange_packaging_weight(self):
        if self.delivery_packaging_id.max_weight and self.shipping_weight > self.delivery_packaging_id.max_weight:
            warning_mess = {
                'title': _('Package too heavy!'),
                'message': _('The weight of your package is higher than the maximum weight authorized for this package type. Please choose another package type.')
            }
            return {'warning': warning_mess}

    def action_put_in_pack(self):
        picking_move_lines = self.picking_id.move_line_ids
        if not self.picking_id.picking_type_id.show_reserved and not self.env.context.get('barcode_view'):
            picking_move_lines = self.picking_id.move_line_nosuggest_ids

        move_line_ids = picking_move_lines.filtered(lambda ml:
            float_compare(ml.qty_done, 0.0, precision_rounding=ml.product_uom_id.rounding) > 0
            and not ml.result_package_id
        )
        if not move_line_ids:
            move_line_ids = picking_move_lines.filtered(lambda ml: float_compare(ml.product_uom_qty, 0.0,
                                 precision_rounding=ml.product_uom_id.rounding) > 0 and float_compare(ml.qty_done, 0.0,
                                 precision_rounding=ml.product_uom_id.rounding) == 0)

        delivery_package = self.picking_id._put_in_pack(move_line_ids)
        # write shipping weight and product_packaging on 'stock_quant_package' if needed
        if self.delivery_packaging_id:
            delivery_package.packaging_id = self.delivery_packaging_id
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
                    <field name="delivery_packaging_id"  domain="[('package_carrier_type', '=', context.get('current_package_carrier_type', 'none'))]"
                      context="{'form_view_ref':'delivery.product_packaging_delivery_form'}"/>
                    <label for="shipping_weight" attrs="{'invisible': [('delivery_packaging_id', '=', False)]}"/>
                    <div class="o_row" attrs="{'invisible': [('delivery_packaging_id', '=', False)]}" name="package_weight">
                        <field name="shipping_weight"/>
                        <field name="weight_uom_name"/>
                    </div>
                </group>
                <footer>
                    <button name="action_put_in_pack" type="object" string="Save" class="btn-primary"/>
                    <button string="Discard" special="cancel" class="btn-secondary"/>
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

