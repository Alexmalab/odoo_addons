# Odoo Module: stock_delivery

Category: Inventory/Delivery

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import wizard


def _auto_install_sale_app(env):
    """Make sure you have at least one "Sales" app to use the advanced delivery logic.

    Either you have e-commerce (website_sale) or Sales (sale_management)
    """
    if env['ir.module.module']._get('website_sale').state != 'uninstalled':
        return
    module_sale_management = env['ir.module.module']._get('sale_management')
    if module_sale_management.state == 'uninstalled':
        module_sale_management.button_install()

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.


{
    'name': 'Delivery - Stock',
    'version': '1.0',
    'category': 'Inventory/Delivery',
    'description': """
Allows you to add delivery methods in pickings.
===============================================

When creating invoices from picking, the system is able to add and compute the shipping line.
""",
    'depends': ['sale_stock', 'delivery'],
    'data': [
        'security/ir.model.access.csv',
        'views/product_template_view.xml',
        'views/delivery_view.xml',
        'views/delivery_portal_template.xml',
        'views/report_shipping.xml',
        'views/report_deliveryslip.xml',
        'views/report_package_barcode.xml',
        'wizard/choose_delivery_carrier_views.xml',
        'wizard/choose_delivery_package_views.xml',
        'views/stock_package_type_views.xml',
        'views/stock_picking_type_views.xml',
        'views/stock_rule_views.xml',
        'views/stock_move_line_views.xml',
    ],
    'demo': ['data/delivery_demo.xml'],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
    'post_init_hook': '_auto_install_sale_app',
}

```

## File: data\delivery_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <!--Sample sale orders with carrier defined-->
        <record id="outgoing_shipment_with_carrier_confirmed" model="stock.picking">
            <field name="picking_type_id" ref="stock.picking_type_out"/>
            <field name="origin">outgoing shipment</field>
            <field name="user_id"></field>
            <field name="carrier_id" ref="delivery.delivery_carrier"/>
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

        <record id="outgoing_shipment_with_carrier_assigned" model="stock.picking">
            <field name="picking_type_id" ref="stock.picking_type_out"/>
            <field name="origin">outgoing shipment</field>
            <field name="user_id"></field>
            <field name="carrier_id" ref="delivery.delivery_carrier"/>
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

        <record id="outgoing_shipment_with_carrier_done" model="stock.picking">
            <field name="picking_type_id" ref="stock.picking_type_out"/>
            <field name="origin">outgoing shipment</field>
            <field name="user_id"></field>
            <field name="carrier_id" ref="delivery.delivery_carrier"/>
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
                ref('outgoing_shipment_with_carrier_done'),
                ref('outgoing_shipment_with_carrier_assigned'),
                ref('outgoing_shipment_with_carrier_confirmed'),
            ]"/>
        </function>
        <function model="stock.picking" name="action_assign">
            <value model="stock.picking" eval="[
                ref('outgoing_shipment_with_carrier_done'),
                ref('outgoing_shipment_with_carrier_assigned'),
            ]"/>
        </function>
        <function model="stock.move.line" name="write">
            <value model="stock.move.line" search="[('picking_id', '=', ref('outgoing_shipment_with_carrier_done'))]"/>
            <value eval="{'quantity': 1}"/>
        </function>
        <function model="stock.picking" name="_action_done">
            <value model="stock.picking" eval="[
                ref('outgoing_shipment_with_carrier_done'),
            ]"/>
        </function>
    </data>
</odoo>

```

## File: models\delivery_carrier.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, fields, models

from odoo.exceptions import UserError, ValidationError
from odoo.tools.float_utils import float_round
from odoo.tools.misc import groupby

from .delivery_request_objects import DeliveryCommodity, DeliveryPackage


class DeliveryCarrier(models.Model):
    _inherit = 'delivery.carrier'

    # -------------------------------- #
    # Internals for shipping providers #
    # -------------------------------- #

    invoice_policy = fields.Selection(
        selection_add=[('real', 'Real cost')],
        ondelete={'real': 'set default'},
        help="Estimated Cost: the customer will be invoiced the estimated cost of the shipping.\n"
        "Real Cost: the customer will be invoiced the real cost of the shipping, the cost of the"
        "shipping will be updated on the SO after the delivery."
    )

    route_ids = fields.Many2many(
        'stock.route', 'stock_route_shipping', 'shipping_id', 'route_id', 'Routes',
        domain=[('shipping_selectable', '=', True)])

    # -------------------------- #
    # API for external providers #
    # -------------------------- #

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

    def get_return_label(self, pickings, tracking_number=None, origin_date=None):
        self.ensure_one()
        if self.can_generate_return:
            res = getattr(self, '%s_get_return_label' % self.delivery_type)(
                pickings, tracking_number, origin_date
            )
            if self.get_return_label_from_portal:
                pickings.return_label_ids.generate_access_token()
            return res

    def get_return_label_prefix(self):
        return 'LabelReturn-%s' % self.delivery_type

    def _get_delivery_label_prefix(self):
        return 'LabelShipping-%s' % self.delivery_type

    def _get_delivery_doc_prefix(self):
        return 'ShippingDoc-%s' % self.delivery_type

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

    def _get_default_custom_package_code(self):
        """ Some delivery carriers require a prefix to be sent in order to use custom
        packages (ie not official ones). This optional method will return it as a string.
        """
        self.ensure_one()
        if hasattr(self, '_%s_get_default_custom_package_code' % self.delivery_type):
            return getattr(self, '_%s_get_default_custom_package_code' % self.delivery_type)()
        else:
            return False

    # -------------------------------- #
    # get default packages/commodities #
    # -------------------------------- #

    def _get_packages_from_order(self, order, default_package_type):
        packages = []

        total_cost = 0
        for line in order.order_line.filtered(lambda line: not line.is_delivery and not line.display_type):
            total_cost += self._product_price_to_company_currency(line.product_qty, line.product_id, order.company_id)

        total_weight = order._get_estimated_weight() + default_package_type.base_weight
        order_weight = self.env.context.get('order_weight', False)
        total_weight = order_weight or total_weight
        if total_weight == 0.0:
            weight_uom_name = self.env['product.template']._get_weight_uom_name_from_ir_config_parameter()
            raise UserError(_("The package cannot be created because the total weight of the products in the picking is 0.0 %s", weight_uom_name))
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
            packages.append(DeliveryPackage(
                order_commodities,
                weight,
                default_package_type,
                total_cost=partial_cost,
                currency=order.company_id.currency_id,
                order=order,
            ))
        return packages

    def _get_packages_from_picking(self, picking, default_package_type):
        packages = []

        if picking.is_return_picking:
            commodities = self._get_commodities_from_stock_move_lines(picking.move_line_ids)
            weight = picking._get_estimated_weight() + default_package_type.base_weight
            packages.append(DeliveryPackage(
                commodities,
                weight,
                default_package_type,
                currency=picking.company_id.currency_id,
                picking=picking,
            ))
            return packages

        # Create all packages.
        for package in picking.package_ids:
            move_lines = picking.move_line_ids.filtered(lambda ml: ml.result_package_id == package)
            commodities = self._get_commodities_from_stock_move_lines(move_lines)
            package_total_cost = 0.0
            for quant in package.quant_ids:
                package_total_cost += self._product_price_to_company_currency(
                    quant.quantity, quant.product_id, picking.company_id
                )
            packages.append(DeliveryPackage(
                commodities,
                package.shipping_weight or package.weight,
                package.package_type_id,
                name=package.name,
                total_cost=package_total_cost,
                currency=picking.company_id.currency_id,
                picking=picking,
            ))

        # Create one package: either everything is in pack or nothing is.
        if picking.weight_bulk:
            commodities = self._get_commodities_from_stock_move_lines(picking.move_line_ids)
            package_total_cost = 0.0
            for move_line in picking.move_line_ids:
                package_total_cost += self._product_price_to_company_currency(
                    move_line.quantity, move_line.product_id, picking.company_id
                )
            packages.append(DeliveryPackage(
                commodities,
                picking.weight_bulk,
                default_package_type,
                name='Bulk Content',
                total_cost=package_total_cost,
                currency=picking.company_id.currency_id,
                picking=picking,
            ))
        elif not packages:
            raise UserError(_(
                "The package cannot be created because the total weight of the "
                "products in the picking is 0.0 %s",
                picking.weight_uom_name
            ))
        return packages

    def _get_commodities_from_order(self, order):
        commodities = []

        for line in order.order_line.filtered(lambda line: not line.is_delivery and not line.display_type and line.product_id.type in ['product', 'consu']):
            unit_quantity = line.product_uom._compute_quantity(line.product_uom_qty, line.product_id.uom_id)
            rounded_qty = max(1, float_round(unit_quantity, precision_digits=0))
            country_of_origin = line.product_id.country_of_origin.code or order.warehouse_id.partner_id.country_id.code
            commodities.append(DeliveryCommodity(
                line.product_id,
                amount=rounded_qty,
                monetary_value=line.price_reduce_taxinc,
                country_of_origin=country_of_origin,
            ))

        return commodities

    def _get_commodities_from_stock_move_lines(self, move_lines):
        commodities = []

        product_lines = move_lines.filtered(lambda line: line.product_id.type in ['product', 'consu'])
        for product, lines in groupby(product_lines, lambda x: x.product_id):
            unit_quantity = sum(
                line.product_uom_id._compute_quantity(
                    line.quantity,
                    product.uom_id)
                for line in lines)
            rounded_qty = max(1, float_round(unit_quantity, precision_digits=0))
            country_of_origin = product.country_of_origin.code or lines[0].picking_id.picking_type_id.warehouse_id.partner_id.country_id.code
            unit_price = sum(line.sale_price for line in lines) / rounded_qty
            commodities.append(DeliveryCommodity(product, amount=rounded_qty, monetary_value=unit_price, country_of_origin=country_of_origin))

        return commodities

    def _product_price_to_company_currency(self, quantity, product, company):
        return company.currency_id._convert(quantity * product.standard_price, product.currency_id, company, fields.Date.today())

    # ------------------------------------------------ #
    # Fixed price shipping, aka a very simple provider #
    # ------------------------------------------------ #

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

    # ----------------------------------- #
    # Based on rule delivery type methods #
    # ----------------------------------- #

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

## File: models\product_template.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


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
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, models


class SaleOrder(models.Model):
    _inherit = 'sale.order'

    def set_delivery_line(self, carrier, amount):
        res = super().set_delivery_line(carrier, amount)
        for order in self:
            if order.state != 'sale':
                continue
            pending_deliveries = order.picking_ids.filtered(
                lambda p: p.state not in ('done', 'cancel')
                          and not any(m.origin_returned_move_id for m in p.move_ids)
            )
            pending_deliveries.carrier_id = carrier.id
        return res

    def _create_delivery_line(self, carrier, price_unit):
        sol = super()._create_delivery_line(carrier, price_unit)
        context = {}
        if self.partner_id:
            # set delivery detail in the customer language
            context['lang'] = self.partner_id.lang
        if carrier.invoice_policy == 'real':
            sol.update({
                'price_unit': 0,
                'name': sol['name']+ _(
                    ' (Estimated Cost: %s )',
                    self.currency_id.format(price_unit)
                ),
            })
        del context
        return sol

    # to remove in master
    def _format_currency_amount(self, amount):
        pre = post = u''
        if self.currency_id.position == 'before':
            pre = u'{symbol}\N{NO-BREAK SPACE}'.format(symbol=self.currency_id.symbol or '')
        else:
            post = u'\N{NO-BREAK SPACE}{symbol}'.format(symbol=self.currency_id.symbol or '')
        return u' {pre}{0}{post}'.format(amount, pre=pre, post=post)


class SaleOrderLine(models.Model):
    _inherit = 'sale.order.line'

    def _prepare_procurement_values(self, group_id):
        values = super(SaleOrderLine, self)._prepare_procurement_values(group_id)
        if not values.get("route_ids") and self.order_id.carrier_id.route_ids:
            values['route_ids'] = self.order_id.carrier_id.route_ids
        return values

```

## File: models\stock_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo.tools.sql import column_exists, create_column


class StockRoute(models.Model):
    _inherit = "stock.route"

    shipping_selectable = fields.Boolean("Applicable on Shipping Methods")


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
    carrier_id = fields.Many2one(related='picking_id.carrier_id', store=True)  # need to be stored for the groupby in `stock_move_line_view_search_delivery`

    @api.depends('quantity', 'product_uom_id', 'product_id', 'move_id.sale_line_id', 'move_id.sale_line_id.price_reduce_taxinc', 'move_id.sale_line_id.product_uom')
    def _compute_sale_price(self):
        for move_line in self:
            sale_line_id = move_line.move_id.sale_line_id
            if sale_line_id and sale_line_id.product_id == move_line.product_id:
                unit_price = sale_line_id.price_reduce_taxinc
                qty = move_line.product_uom_id._compute_quantity(move_line.quantity, sale_line_id.product_uom)
            else:
                # For kits, use the regular unit price
                unit_price = move_line.product_id.list_price
                qty = move_line.product_uom_id._compute_quantity(move_line.quantity, move_line.product_id.uom_id)
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
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


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

    @api.depends('package_carrier_type')
    def _compute_length_uom_name(self):
        package_without_carrier = self.env['stock.package.type']
        for package in self:
            if package.package_carrier_type and package.package_carrier_type != 'none':
                # FIXME This variable does not impact any logic, it is only used for the packaging display on the form view.
                #  However, it generates some confusion for the users since this UoM will be ignored when sending the requests
                #  to the carrier server: the dimensions will be expressed with another UoM and there won't be any conversion.
                #  For instance, with Fedex, the UoM used with the package dimensions will depend on the UoM of
                #  `fedex_weight_unit`. With UPS, we will use the UoM defined on `ups_package_dimension_unit`
                package.length_uom_name = ""
            else:
                package_without_carrier |= package
        super(PackageType, package_without_carrier)._compute_length_uom_name()

```

## File: models\stock_picking.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict
from datetime import date
from markupsafe import Markup
import json

from odoo import _, api, fields, models, SUPERUSER_ID
from odoo.exceptions import UserError
from odoo.tools.sql import column_exists, create_column


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
        counts = dict(self.env['stock.move.line']._read_group(
           domain=[
              ('picking_id', 'in', self.ids),
              ('result_package_id', '!=', False)],
              groupby=['picking_id'],
              aggregates=['__count'],
        ))
        self.fetch(['move_line_ids'])
        self.move_line_ids.fetch(['result_package_id'])
        for picking in self:
            packs = set()
            if counts.get(picking, 0):
                for move_line in picking.move_line_ids:
                    if move_line.result_package_id:
                        packs.add(move_line.result_package_id.id)
            picking.package_ids = list(packs)

    @api.depends('move_line_ids', 'move_line_ids.result_package_id', 'move_line_ids.product_uom_id', 'move_line_ids.quantity')
    def _compute_bulk_weight(self):
        picking_weights = defaultdict(float)
        res_groups = self.env['stock.move.line']._read_group(
            [('picking_id', 'in', self.ids), ('product_id', '!=', False), ('result_package_id', '=', False)],
            ['picking_id', 'product_id', 'product_uom_id', 'quantity'],
            ['__count'],
        )
        for picking, product, product_uom, quantity, count in res_groups:
            picking_weights[picking.id] += (
                count
                * product_uom._compute_quantity(quantity, product.uom_id)
                * product.weight
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
                picking.return_label_ids = self.env['ir.attachment'].search([('res_model', '=', 'stock.picking'), ('res_id', '=', picking.id), ('name', '=like', '%s%%' % picking.carrier_id.get_return_label_prefix())])
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
        view_id = self.env.ref('stock_delivery.choose_delivery_package_view_form').id
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
        # Update the context 'default_package_type_id' passed from JS
        # to populate the scanned package type in the package wizard opened from the barcode.
        if self.env.context.get('default_package_type_id'):
            context['default_delivery_package_type_id'] = self.env.context.get('default_package_type_id')
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
        self.carrier_price = self.carrier_id.with_context(order=self.sale_id)._apply_margins(res['exact_price'])
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
        msg = _("Shipment sent to carrier %(carrier_name)s for shipping with tracking number %(ref)s",
                carrier_name=self.carrier_id.name,
                ref=self.carrier_tracking_ref) + \
              Markup("<br/>") + \
              _("Cost: %(price).2f %(currency)s",
                price=self.carrier_price,
                currency=order_currency.name)
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
            msg = _("Tracking links for shipment:") + Markup("<br/>")
            for tracker in carrier_trackers:
                msg += Markup('<a href="%s">%s</a><br/>') % (tracker[1], tracker[0])
            self.message_post(body=msg)
            return self.env["ir.actions.actions"]._for_xml_id("stock_delivery.act_delivery_trackers_url")

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

```

## File: models\stock_quant_package.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict

from odoo import api, fields, models


class StockQuantPackage(models.Model):
    _inherit = "stock.quant.package"

    @api.depends('quant_ids', 'package_type_id')
    def _compute_weight(self):
        if self.env.context.get('picking_id'):
            package_weights = defaultdict(float)
            res_groups = self.env['stock.move.line']._read_group(
                [('result_package_id', 'in', self.ids), ('product_id', '!=', False), ('picking_id', '=', self.env.context['picking_id'])],
                ['result_package_id', 'product_id', 'product_uom_id', 'quantity'],
                ['__count'],
            )
            for result_package, product, product_uom, quantity, count in res_groups:
                package_weights[result_package.id] += (
                    count
                    * product_uom._compute_quantity(quantity, product.uom_id)
                    * product.weight
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

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import delivery_carrier
from . import product_template
from . import sale_order
from . import stock_move
from . import stock_package_type
from . import stock_picking
from . import stock_quant_package

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_delivery_carrier_stock_user,delivery.carrier stock_user,model_delivery_carrier,stock.group_stock_user,1,0,0,0
access_delivery_carrier_stock_manager,delivery.carrier stock_manager,model_delivery_carrier,stock.group_stock_manager,1,1,1,1
access_choose_delivery_package,access.choose.delivery.package stock_user,model_choose_delivery_package,stock.group_stock_user,1,1,1,0
access_choose_delivery_carrier_stock_user,access.choose.delivery.carrier stock_user,model_choose_delivery_carrier,stock.group_stock_user,1,1,1,0
access_delivery_zip_prefix_stock_manager,delivery.zip.prefix stock_manager,delivery.model_delivery_zip_prefix,stock.group_stock_manager,1,1,1,1
access_delivery_price_rule_stock_manager,delivery.price.rule stock_manager,delivery.model_delivery_price_rule,stock.group_stock_manager,1,1,1,1

```

## File: views\delivery_portal_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="sale_order_portal_content_inherit_sale_stock_inherit_website_sale"
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
      <record id="view_delivery_carrier_form_inherit_stock_delivery" model="ir.ui.view">
            <field name="name">delivery.carrier.form</field>
            <field name="model">delivery.carrier</field>
            <field name="inherit_id" ref="delivery.view_delivery_carrier_form"/>
            <field name="arch" type="xml">
                <xpath expr="//group[@name='provider_details']" position="inside">
                    <field name="route_ids" string="Routes" options="{'no_create': True}" groups="stock.group_adv_location" widget="many2many_tags"/>
                </xpath>
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
                        <field name="carrier_id" readonly="state in ('done', 'cancel')" options="{'no_create': True, 'no_open': True}"/>
                        <field name="delivery_type" invisible="True"/>
                        <label for="carrier_tracking_ref"/>
                        <div name="tracking">
                            <field name="carrier_tracking_ref" class="oe_inline text-break" readonly="state in ('done', 'cancel')"/>
                            <button type='object' class="oi oi-arrow-right oe_link" confirm="Cancelling a delivery may not be undoable. Are you sure you want to continue?" name="cancel_shipment" string="Cancel" invisible="not carrier_tracking_ref or delivery_type in ['fixed', 'base_on_rule'] or not delivery_type or state != 'done'"/>
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
                         invisible="not carrier_tracking_ref or not carrier_id or delivery_type == 'grid'" />
                </div>
                <xpath expr="/form/header/button[last()]" position="after">
                    <button name="send_to_shipper" string="Send to Shipper" type="object" invisible="carrier_tracking_ref or delivery_type in ['fixed', 'base_on_rule'] or not delivery_type or state != 'done' or picking_type_code == 'incoming'" data-hotkey="shift+v"/>
                </xpath>
                <xpath expr="/form/header/button[last()]" position="after">
                    <button name="print_return_label" string="Print Return Label" type="object" invisible="not is_return_picking or state == 'done' or picking_type_code != 'incoming'" data-hotkey="shift+o"/>
                </xpath>
                <xpath expr="//field[@name='partner_id']" position="attributes">
                    <attribute name="required">carrier_id and carrier_id.integration_level == 'rate_and_ship'</attribute>
                </xpath>
              </data>
            </field>
        </record>

        <menuitem action="delivery.action_delivery_carrier_form" id="menu_action_delivery_carrier_form" parent="stock.menu_delivery" sequence="1"/>
        <menuitem action="delivery.action_delivery_zip_prefix_list" id="menu_delivery_zip_prefix" parent="stock.menu_delivery" groups="base.group_no_one" sequence="100"/>
        
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

        <record id="delivery_tracking_url_warning_form" model="ir.ui.view">
            <field name="name">delivery.carrier.warning.url.form</field>
            <field name="model">stock.picking</field>
            <field name="arch" type="xml">
                <form string="Trackers URL">
                    <div class="alert alert-info" role="status">
                        <p>You have multiple tracker links, they are available in the chatter.</p>
                    </div>
                    <footer>
                        <button string="OK" special="cancel" data-hotkey="x" class="oe_highlight"/>
                    </footer>
                </form>
            </field>
        </record>

        <record id="act_delivery_trackers_url" model="ir.actions.act_window">
            <field name="name">Display tracking links</field>
            <field name="res_model">stock.picking</field>
            <field name="view_id" ref="stock_delivery.delivery_tracking_url_warning_form"/>
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
<?xml version="1.0" encoding="utf-8"?>
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
        <xpath expr="//td[@name='move_line_lot_quantity']" position="after">
            <td t-if="has_hs_code"><span t-field="move_line.product_id.hs_code"/></td>
        </xpath>
    </template>
    <template id="stock_report_delivery_aggregated_move_lines_inherit_delivery" inherit_id="stock.stock_report_delivery_aggregated_move_lines">
        <xpath expr="//td[@name='move_line_aggregated_quantity']" position="after">
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
<?xml version="1.0" encoding="utf-8"?>
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
<?xml version="1.0" encoding="utf-8"?>
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

## File: views\stock_move_line_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="stock_location_route_view_form_inherit_stock_delivery" model="ir.ui.view">
        <field name="name">stock.route.form</field>
        <field name="model">stock.route</field>
        <field name="inherit_id" ref="stock.stock_location_route_form_view"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='route_selector']/group" position="inside">
                <field name="shipping_selectable" string="Shipping Methods"/>
            </xpath>
        </field>
    </record>

    <record model="ir.ui.view" id="stock_move_line_view_search_delivery">
        <field name="name">stock.move.line.search.delivery</field>
        <field name="model">stock.move.line</field>
        <field name="inherit_id" ref="stock.stock_move_line_view_search"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='owner_id']" position="after">
                <field name="carrier_id" invisible="1" string="Carrier name"/>
            </xpath>
            <xpath expr="//group[@name='groupby']" position="inside">
                <filter string="Carrier" name="by_carrier" context="{'group_by': 'carrier_id'}"/>
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
                <field name="print_label" invisible="code not in ['internal', 'outgoing']"/>
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
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, models


class ChooseDeliveryCarrier(models.TransientModel):
    _inherit = 'choose.delivery.carrier'

    @api.depends('carrier_id')
    def _compute_invoicing_message(self):
        super()._compute_invoicing_message()
        if self.carrier_id.invoice_policy == 'real':
            self.invoicing_message = _('The shipping price will be set once the delivery is done.')

```

## File: wizard\choose_delivery_carrier_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="choose_delivery_carrier_view_form" model="ir.ui.view">
        <field name="name">choose.delivery.carrier.form</field>
        <field name="model">choose.delivery.carrier</field>
        <field name="inherit_id" ref="delivery.choose_delivery_carrier_view_form"/>
        <field name="arch" type="xml">
            <!-- Remove the groups to show the weight when stock is installed -->
            <xpath expr="//div[@name='carried_weight']" position="attributes">
                <attribute name="groups"></attribute>
            </xpath>
            <xpath expr="//label[@name='carried_weight_label']" position="attributes">
                <attribute name="groups"></attribute>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: wizard\choose_delivery_package.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _


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
            move_line_ids = rec.picking_id._package_move_lines(
                batch_pack=self.env.context.get('batch_pack')
            )
            # Add package weights to shipping weight, package base weight is defined in package.type
            total_weight = rec.delivery_package_type_id.base_weight or 0.0
            for ml in move_line_ids:
                qty = ml.product_uom_id._compute_quantity(ml.quantity, ml.product_id.uom_id)
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
        return self.picking_id._post_put_in_pack_hook(delivery_package)

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
                    <label for="shipping_weight" invisible="not delivery_package_type_id"/>
                    <div class="o_row" invisible="not delivery_package_type_id" name="package_weight">
                        <field name="shipping_weight"/>
                        <field name="weight_uom_name"/>
                    </div>
                </group>
                <footer>
                    <button name="action_put_in_pack" type="object" string="Save" class="btn-primary" data-hotkey="q"/>
                    <button string="Discard" special="cancel" data-hotkey="x" class="btn-secondary"/>
                </footer>
            </form>
        </field>
    </record>

</odoo>

```

## File: wizard\stock_return_picking.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class StockReturnPicking(models.TransientModel):
    _inherit = 'stock.return.picking'

    def _create_returns(self):
        # Prevent copy of the carrier and carrier price when generating return picking
        # (we have no integration of returns for now)
        new_picking, pick_type_id = super()._create_returns()
        self._reset_carrier_id(new_picking)
        return new_picking, pick_type_id

    def _reset_carrier_id(self, new_picking):
        """ Prevent copy of the carrier and carrier price when generating return picking
        (we have no integration of returns for now).
        """
        picking = self.env['stock.picking'].browse(new_picking)
        picking.write({
            'carrier_id': False,
            'carrier_price': 0.0,
        })

```

## File: wizard\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import choose_delivery_package
from . import choose_delivery_carrier
from . import stock_return_picking

```

