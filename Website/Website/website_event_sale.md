# Odoo Module: website_event_sale

Category: Website/Website

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models
from . import report

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': "Online Event Ticketing",
    'category': 'Website/Website',
    'summary': "Sell event tickets online",
    'description': """
Sell event tickets through eCommerce app.
    """,
    'depends': ['website_event', 'event_sale', 'website_sale'],
    'data': [
        'data/event_data.xml',
        'report/event_sale_report_views.xml',
        'views/event_event_views.xml',
        'views/website_event_templates.xml',
        'views/website_sale_templates.xml',
        'security/website_event_sale_security.xml',
    ],
    'auto_install': True,
    'assets': {
        'web.assets_tests': [
            'website_event_sale/static/tests/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict
from odoo import _
from odoo.http import request, route

from odoo.addons.website_event.controllers.main import WebsiteEventController


class WebsiteEventSaleController(WebsiteEventController):

    @route()
    def event_register(self, event, **post):
        event = event.with_context(pricelist=request.website.id)
        if not request.context.get('pricelist'):
            pricelist = request.website.pricelist_id
            if pricelist:
                event = event.with_context(pricelist=pricelist.id)
        return super().event_register(event, **post)

    def _process_tickets_form(self, event, form_details):
        """ Add price information on ticket order """
        res = super()._process_tickets_form(event, form_details)
        for item in res:
            item['price'] = item['ticket']['price'] if item['ticket'] else 0
        return res

    def _create_attendees_from_registration_post(self, event, registration_data):
        # we have at least one registration linked to a ticket -> sale mode activate
        if not any(info.get('event_ticket_id') for info in registration_data):
            return super()._create_attendees_from_registration_post(event, registration_data)

        order_sudo = request.website.sale_get_order(force_create=True)
        if order_sudo.state != 'draft':
            request.website.sale_reset()
            order_sudo = request.website.sale_get_order(force_create=True)

        tickets_data = defaultdict(int)
        for data in registration_data:
            event_ticket_id = data.get('event_ticket_id')
            if event_ticket_id:
                tickets_data[event_ticket_id] += 1

        cart_data = {}
        for ticket_id, count in tickets_data.items():
            ticket_sudo = request.env['event.event.ticket'].sudo().browse(ticket_id)
            cart_values = order_sudo._cart_update(
                product_id=ticket_sudo.product_id.id,
                add_qty=count,
                event_ticket_id=ticket_id,
            )
            cart_data[ticket_id] = cart_values['line_id']

        for data in registration_data:
            event_ticket_id = data.get('event_ticket_id')
            if event_ticket_id:
                data['sale_order_id'] = order_sudo.id
                data['sale_order_line_id'] = cart_data[event_ticket_id]

        request.session['website_sale_cart_quantity'] = order_sudo.cart_quantity

        return super()._create_attendees_from_registration_post(event, registration_data)

    @route()
    def registration_confirm(self, event, **post):
        res = super().registration_confirm(event, **post)

        registrations = self._process_attendees_form(event, post)

        # we have at least one registration linked to a ticket -> sale mode activate
        if any(info['event_ticket_id'] for info in registrations):
            order_sudo = request.website.sale_get_order()
            if order_sudo.amount_total:
                return request.redirect("/shop/checkout")
            # free tickets -> order with amount = 0: auto-confirm, no checkout
            elif order_sudo:
                order_sudo.action_confirm()  # tde notsure: email sending ?
                request.website.sale_reset()

        return res

```

## File: controllers\sale.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.addons.website_sale.controllers.main import WebsiteSale


class WebsiteEventSale(WebsiteSale):

    def _prepare_shop_payment_confirmation_values(self, order):
        values = super(WebsiteEventSale,
                       self)._prepare_shop_payment_confirmation_values(order)
        values['events'] = order.order_line.event_id
        return values

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-

from . import main
from . import sale

```

## File: data\event_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- Update the event type if exists -->
        <function model="event.type" name="write">
            <value eval="[ref('event.event_type_data_ticket', False)]"/>
            <value eval="{'name': 'Sell Online', 'website_menu': True}"/>
        </function>
    </data>
</odoo>

```

## File: models\product.py

```python
# -*- coding: utf-8 -*-

from odoo import api, fields, models


# defined for access rules
class Product(models.Model):
    _inherit = 'product.product'

    event_ticket_ids = fields.One2many('event.event.ticket', 'product_id', string='Event Tickets')

    def _is_add_to_cart_allowed(self):
        # Allow adding event tickets to the cart regardless of product's rules
        self.ensure_one()
        res = super()._is_add_to_cart_allowed()
        return res or any(event.website_published for event in self.event_ticket_ids.event_id)

class ProductTemplate(models.Model):
    _inherit = 'product.template'

    @api.model
    def _get_product_types_allow_zero_price(self):
        return super()._get_product_types_allow_zero_price() + ["event"]

```

## File: models\product_pricelist.py

```python
# -*- coding: utf-8 -*-

from odoo import _, api, models

class PricelistItem(models.Model):
    _inherit = "product.pricelist.item"

    @api.onchange('applied_on', 'product_id', 'product_tmpl_id', 'min_quantity')
    def _onchange_event_sale_warning(self):
        if self.min_quantity > 0:
            msg = ''
            if self.applied_on == '3_global' or self.applied_on == '2_product_category':
                msg = _("A pricelist item with a positive min. quantity will not be applied to the event tickets products.")
            elif ((self.applied_on == '1_product' and self.product_tmpl_id.detailed_type == 'event') or
                    (self.applied_on == '0_product_variant' and self.product_id.detailed_type == 'event')):
                msg = _("A pricelist item with a positive min. quantity cannot be applied to this event tickets product.")
            if msg:
                return {'warning':
                    {
                        'title': _("Warning"),
                        'message': msg
                    }
                }

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-

from odoo import api, models, _
from odoo.exceptions import UserError


class SaleOrder(models.Model):
    _inherit = "sale.order"

    def _cart_find_product_line(self, product_id=None, line_id=None, event_ticket_id=False, **kwargs):
        lines = super()._cart_find_product_line(product_id, line_id, **kwargs)
        if line_id or not event_ticket_id:
            return lines

        return lines.filtered(
            lambda line: line.event_ticket_id.id == event_ticket_id
        )

    def _verify_updated_quantity(self, order_line, product_id, new_qty, event_ticket_id=False, **kwargs):
        """Restrict quantity updates for event tickets according to available seats."""
        new_qty, warning = super()._verify_updated_quantity(order_line, product_id, new_qty, **kwargs)

        if not event_ticket_id:
            if not order_line.event_ticket_id or new_qty < order_line.product_uom_qty:
                return new_qty, warning
            else:
                return order_line.product_uom_qty, _("You cannot raise manually the event ticket quantity in your cart")

        # Adding new ticket to the cart (might be automatically linked to an existing line)
        ticket = self.env['event.event.ticket'].browse(event_ticket_id).exists()
        if not ticket:
            raise UserError(_("The provided ticket doesn't exist"))

        # TODO TDE consider full cart qty and not only added qty
        # if event seats are not auto confirmed.
        # Since created registrations are automatically reserved
        # We should only consider new added qty and not full quantity
        # when checking for seat availability
        existing_qty = order_line.product_uom_qty if order_line else 0
        qty_added = new_qty - existing_qty
        warning = ''
        if ticket.seats_limited and ticket.seats_available <= 0:
            # Remove existing line if exists and do not add a new one
            # if no ticket is available anymore
            new_qty = existing_qty
            warning = _(
                'Sorry, The %(ticket)s tickets for the %(event)s event are sold out.',
                ticket=ticket.name,
                event=ticket.event_id.name,
            )
        elif ticket.seats_limited and qty_added > ticket.seats_available:
            new_qty = existing_qty + ticket.seats_available
            warning = _(
                'Sorry, only %(remaining_seats)d seats are still available for the %(ticket)s ticket for the %(event)s event.',
                remaining_seats=ticket.seats_available,
                ticket=ticket.name,
                event=ticket.event_id.name,
            )

        return new_qty, warning

    def _prepare_order_line_values(self, product_id, quantity, event_ticket_id=False, **kwargs):
        """Add corresponding event to the SOline creation values (if ticket is provided)."""
        values = super()._prepare_order_line_values(product_id, quantity, **kwargs)

        if not event_ticket_id:
            return values

        ticket = self.env['event.event.ticket'].browse(event_ticket_id)

        if ticket.product_id.id != product_id:
            raise UserError(_("The ticket doesn't match with this product."))

        values['event_id'] = ticket.event_id.id
        values['event_ticket_id'] = ticket.id

        return values

    def _update_cart_line_values(self, order_line, update_values):
        """Remove event registrations on quantity decrease."""
        old_qty = order_line.product_uom_qty

        super()._update_cart_line_values(order_line, update_values)
        if not order_line.event_ticket_id:
            return

        new_qty = order_line.product_uom_qty
        if new_qty < old_qty:
            attendees = self.env['event.registration'].search([
                ('state', '!=', 'cancel'),
                ('sale_order_id', '=', self.id),
                ('event_ticket_id', '=', order_line.event_ticket_id.id),
            ], offset=new_qty, limit=(old_qty - new_qty), order='create_date asc')
            attendees.action_cancel()


class SaleOrderLine(models.Model):
    _inherit = "sale.order.line"

    @api.depends('product_id.display_name', 'event_ticket_id.display_name')
    def _compute_name_short(self):
        """ If the sale order line concerns a ticket, we don't want the product name, but the ticket name instead.
        """
        super(SaleOrderLine, self)._compute_name_short()

        for record in self:
            if record.event_ticket_id:
                record.name_short = record.event_ticket_id.display_name

```

## File: models\website.py

```python
# -*- coding: utf-8 -*-

from odoo import models


class Website(models.Model):
    _inherit = 'website'

    def sale_product_domain(self):
        # remove product event from the website content grid and list view (not removed in detail view)
        return ['&'] + super(Website, self).sale_product_domain() + [('detailed_type', '!=', 'event')]

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import product
from . import product_pricelist
from . import sale_order
from . import website

```

## File: report\event_sale_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class EventSaleReport(models.Model):
    _inherit = 'event.sale.report'

    is_published = fields.Boolean('Published Events', readonly=True)

    def _select_clause(self, *select):
        return super()._select_clause('event_event.is_published as is_published', *select)

```

## File: report\event_sale_report_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="event_sale_report_view_search" model="ir.ui.view">
        <field name="name">event.sale.report.view.search.inherit.website</field>
        <field name="model">event.sale.report</field>
        <field name="inherit_id" ref="event_sale.event_sale_report_view_search"/>
        <field name="arch" type="xml">
            <xpath expr="//search" position="inside">
                <filter string="Published Events" name="is_published" domain="[('is_published', '=', True)]"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: report\__init__.py

```python
from . import event_sale_report

```

## File: security\website_event_sale_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <record id="event_product_template_public" model="ir.rule">
        <field name="name">Product template linked to event: Public</field>
        <field name="model_id" ref="product.model_product_template"/>
        <field name="domain_force">[('product_variant_ids.event_ticket_ids.event_id.website_published', '=', True)]</field>
        <field name="groups" eval="[(4, ref('base.group_public')), (4, ref('base.group_portal'))]"/>
        <field name="perm_read" eval="True"/>
        <field name="perm_write" eval="False"/>
        <field name="perm_create" eval="False"/>
        <field name="perm_unlink" eval="False"/>
    </record>
</odoo>

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70">
    <defs>
        <path id="icon-a" d="M4,5.35309892e-14 C36.4160122,9.87060235e-15 58.0836068,-3.97961823e-14 65,5.07020818e-14 C69,6.733808e-14 70,1 70,5 C70,43.0488877 70,62.4235458 70,65 C70,69 69,70 65,70 C61,70 9,70 4,70 C1,70 7.10542736e-15,69 7.10542736e-15,65 C7.25721566e-15,62.4676575 3.83358709e-14,41.8005206 3.60818146e-14,5 C-1.13686838e-13,1 1,5.75716207e-14 4,5.35309892e-14 Z"/>
        <linearGradient id="icon-c" x1="100%" x2="0%" y1="0%" y2="100%">
            <stop offset="0%" stop-color="#DA956B"/>
            <stop offset="100%" stop-color="#CC7039"/>
        </linearGradient>
        <path id="icon-d" d="M30.4841117,22.0178818 L43,33 C39,37.3544294 37.3333333,42.3544294 38,48 L21.0744625,32.4683559 L30.4841117,22.0178818 Z M42.5223849,56.838607 C41.2078243,58.1148367 39.1294169,58.18659 37.7708523,56.9633329 L12.0893951,33.839645 C10.6710296,32.562543 10.574444,30.3575851 11.8736617,28.9146577 L16.5784862,23.6894206 C17.9968517,24.9665226 20.1998265,24.8320983 21.4990443,23.3891709 C22.798262,21.9462434 22.7016763,19.7412856 21.2833108,18.4641836 L25.9881354,13.2389465 C27.2873531,11.7960191 29.4903279,11.6615948 30.9086934,12.9386968 L49.9350371,30.0700936 C47.9735137,30.1405297 46.1915216,30.6862713 44.6377627,31.6080597 L31.6243623,19.8907413 C30.9151795,19.2521903 29.8136921,19.3194025 29.1640833,20.0408662 L18.9702967,31.3622132 C18.3206878,32.0836769 18.3689807,33.1861558 19.0781634,33.8247068 L37.0551835,50.0112884 C37.5260595,50.4352671 38.1698855,50.5481058 38.7395561,50.3648826 C39.4901538,52.938764 40.7880675,55.1720216 42.5223849,56.838607 Z M47.7280684,41.09729 C47.7280684,44.4791455 58.0000556,43.620791 58,50.0493457 C58,53.1285938 55.7508537,55.7588721 52.0971585,56.3075293 L52.0971585,58.8554688 C52.0971585,59.2114111 51.7984472,59.5 51.43002,59.5 L49.2062249,59.5 C48.8377977,59.5 48.5390864,59.2114111 48.5390864,58.8554688 L48.5390864,56.2656885 C46.3429776,55.9048584 44.386038,54.9254395 43.0490368,53.6890137 C42.805698,53.4639648 42.7750097,53.0985693 42.9766523,52.8378564 L44.666125,50.6535938 C44.8927853,50.3606006 45.3248687,50.3096289 45.6200774,50.5386523 C47.0052794,51.6132471 48.7943781,52.4661768 50.5039761,52.4661768 C52.4949954,52.4661768 53.4018034,51.3202539 53.4018034,50.255542 C53.4018034,47.1078662 43.1298162,47.7912305 43.1298162,41.1561572 C43.1298162,38.3244092 45.2797812,36.034873 48.539142,35.3210547 L48.539142,32.6445313 C48.539142,32.2885889 48.8378533,32 49.2062805,32 L51.4300755,32 C51.7985028,32 52.0972141,32.2885889 52.0972141,32.6445313 L52.0972141,35.1817822 C53.8848117,35.3813721 55.8225155,36.0588281 57.1724147,37.2123779 C57.4043009,37.4105176 57.4628423,37.7377783 57.3155715,38.0008545 L56.006201,40.3399121 C55.8146766,40.6821045 55.3560189,40.7828125 55.029121,40.5556689 C53.7868535,39.6925342 52.2641099,39.033877 50.7782812,39.033877 C48.9253596,39.033877 47.7280684,39.8438379 47.7280684,41.09729 Z"/>
        <path id="icon-e" d="M30.4841117,20.0178818 L43,31 C39,35.3544294 37.3333333,40.3544294 38,46 L21.0744625,30.4683559 L30.4841117,20.0178818 Z M42.5223849,54.838607 C41.2078243,56.1148367 39.1294169,56.18659 37.7708523,54.9633329 L12.0893951,31.839645 C10.6710296,30.562543 10.574444,28.3575851 11.8736617,26.9146577 L16.5784862,21.6894206 C17.9968517,22.9665226 20.1998265,22.8320983 21.4990443,21.3891709 C22.798262,19.9462434 22.7016763,17.7412856 21.2833108,16.4641836 L25.9881354,11.2389465 C27.2873531,9.79601907 29.4903279,9.66159477 30.9086934,10.9386968 L49.9350371,28.0700936 C47.9735137,28.1405297 46.1915216,28.6862713 44.6377627,29.6080597 L31.6243623,17.8907413 C30.9151795,17.2521903 29.8136921,17.3194025 29.1640833,18.0408662 L18.9702967,29.3622132 C18.3206878,30.0836769 18.3689807,31.1861558 19.0781634,31.8247068 L37.0551835,48.0112884 C37.5260595,48.4352671 38.1698855,48.5481058 38.7395561,48.3648826 C39.4901538,50.938764 40.7880675,53.1720216 42.5223849,54.838607 Z M47.7280684,39.09729 C47.7280684,42.4791455 58.0000556,41.620791 58,48.0493457 C58,51.1285938 55.7508537,53.7588721 52.0971585,54.3075293 L52.0971585,56.8554688 C52.0971585,57.2114111 51.7984472,57.5 51.43002,57.5 L49.2062249,57.5 C48.8377977,57.5 48.5390864,57.2114111 48.5390864,56.8554688 L48.5390864,54.2656885 C46.3429776,53.9048584 44.386038,52.9254395 43.0490368,51.6890137 C42.805698,51.4639648 42.7750097,51.0985693 42.9766523,50.8378564 L44.666125,48.6535938 C44.8927853,48.3606006 45.3248687,48.3096289 45.6200774,48.5386523 C47.0052794,49.6132471 48.7943781,50.4661768 50.5039761,50.4661768 C52.4949954,50.4661768 53.4018034,49.3202539 53.4018034,48.255542 C53.4018034,45.1078662 43.1298162,45.7912305 43.1298162,39.1561572 C43.1298162,36.3244092 45.2797812,34.034873 48.539142,33.3210547 L48.539142,30.6445313 C48.539142,30.2885889 48.8378533,30 49.2062805,30 L51.4300755,30 C51.7985028,30 52.0972141,30.2885889 52.0972141,30.6445313 L52.0972141,33.1817822 C53.8848117,33.3813721 55.8225155,34.0588281 57.1724147,35.2123779 C57.4043009,35.4105176 57.4628423,35.7377783 57.3155715,36.0008545 L56.006201,38.3399121 C55.8146766,38.6821045 55.3560189,38.7828125 55.029121,38.5556689 C53.7868535,37.6925342 52.2641099,37.033877 50.7782812,37.033877 C48.9253596,37.033877 47.7280684,37.8438379 47.7280684,39.09729 Z"/>
    </defs>
    <g fill="none" fill-rule="evenodd">
        <mask id="icon-b" fill="#fff">
            <use xlink:href="#icon-a"/>
        </mask>
        <g mask="url(#icon-b)">
            <rect width="70" height="70" fill="url(#icon-c)"/>
            <path fill="#FFF" fill-opacity=".383" d="M4,1.8 L65,1.8 C67.6666667,1.8 69.3333333,1.13333333 70,-0.2 C70,2.46666667 70,3.46666667 70,2.8 L1.10547097e-14,2.8 C-1.65952376e-14,3.46666667 -2.9161925e-14,2.46666667 -2.66453526e-14,-0.2 C0.666666667,1.13333333 2,1.8 4,1.8 Z" transform="matrix(1 0 0 -1 0 2.8)"/>
            <path fill="#393939" d="M40.940835,55 L4,55 C2,55 -7.10542736e-15,54.851752 0,50.8490566 L2.0990647e-16,27.6230718 L12,13.490566 L24,0 L37,3.11320755 L46,11.4150943 L52,16.6037736 L52,19.7169811 L57,21.7924528 L51,29.0566038 L57,35.9806542 L52,38.8120355 L52,42.1170824 L40.940835,55 Z" opacity=".324" transform="translate(0 15)"/>
            <path fill="#000" fill-opacity=".383" d="M4,4 L65,4 C67.6666667,4 69.3333333,3 70,1 C70,3.66666667 70,5 70,5 L1.77635684e-15,5 C1.77635684e-15,5 1.77635684e-15,3.66666667 1.77635684e-15,1 C0.666666667,3 2,4 4,4 Z" transform="translate(0 65)"/>
            <use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#icon-d"/>
            <use fill="#FFF" fill-rule="nonzero" xlink:href="#icon-e"/>
        </g>
    </g>
</svg>

```

## File: views\assets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="assets_tests" inherit_id="web.assets_tests" name="Website Event Sale Assets Tests">
    <xpath expr="." position="inside">
        <script type="text/javascript" src="/website_event_sale/static/tests/tours/website_event_sale.js"></script>
        <script type="text/javascript" src="/website_event_sale/static/tests/tours/website_event_sale_last_ticket.js"></script>
    </xpath>
</template>

</odoo>

```

## File: views\event_event_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="event_form_mandatory_company" model="ir.ui.view">
        <field name="name">event.event.view.form.inherit.company.mandatory</field>
        <field name="model">event.event</field>
        <field name="inherit_id" ref="event.view_event_form"/>
        <field name="arch" type="xml">
            <xpath expr="//group//field[@name='company_id']" position="attributes">
                <attribute name="required">1</attribute>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\website_event_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="registration_template" inherit_id="website_event.registration_template">
    <!-- Add price information on tickets (multi tickets, aka in collapse) -->
    <xpath expr="//div[hasclass('o_wevent_registration_multi_select')]" position="inside">
        <t t-if="ticket.price">
            <t t-if="(ticket.price - ticket.price_reduce) &gt; 1 and website.pricelist_id.discount_policy == 'without_discount'">
                <del class="text-danger me-1" t-field="ticket.price" t-options="{'widget': 'monetary', 'from_currency': event.company_id.sudo().currency_id, 'display_currency': website.pricelist_id.currency_id}"/>
            </t>
            <span t-field="ticket.price_reduce"
                  t-options="{'widget': 'monetary', 'from_currency': event.company_id.sudo().currency_id, 'display_currency': website.pricelist_id.currency_id}"
                  groups="account.group_show_line_subtotals_tax_excluded"/>
            <span t-field="ticket.price_reduce_taxinc"
                  t-options="{'widget': 'monetary', 'from_currency': event.company_id.sudo().currency_id, 'display_currency': website.pricelist_id.currency_id}"
                  groups="account.group_show_line_subtotals_tax_included"/>
            <span itemprop="price" class="d-none" t-out="ticket.price"/>
            <span itemprop="priceCurrency" class="d-none" t-out="website.pricelist_id.currency_id.name"/>
        </t>
        <span t-else="" class="fw-bold text-uppercase">Free</span>
    </xpath>
    <xpath expr="//div[hasclass('o_wevent_price_range')]" position="inside">
        <t t-if="event.event_ticket_ids[-1].price_reduce > 0">
            <span class="text-dark">
                From
                <span t-out="event.event_ticket_ids[0].price_reduce" t-options="{'widget': 'monetary', 'from_currency': event.company_id.sudo().currency_id, 'display_currency': website.pricelist_id.currency_id}"/>
                <t t-if="event.event_ticket_ids[-1].price_reduce != event.event_ticket_ids[0].price_reduce">
                    to
                    <span t-out="event.event_ticket_ids[-1].price_reduce" t-options="{'widget': 'monetary', 'from_currency': event.company_id.sudo().currency_id, 'display_currency': website.pricelist_id.currency_id}"/>
                </t>
            </span>
        </t>
    </xpath>
    <!-- Add price information on tickets (mono ticket, aka not in collapse) -->
    <xpath expr="//div[hasclass('o_wevent_registration_single')]//h6" position="after">
        <div class="px-2 text-dark d-flex align-items-center align-self-stretch">
            <t t-if="tickets.price">
                <t t-if="(tickets.price - tickets.price_reduce) &gt;1 and website.get_current_pricelist().discount_policy == 'without_discount'">
                    <del class="text-danger me-1" t-field="tickets.price" t-options="{'widget': 'monetary', 'from_currency': event.company_id.sudo().currency_id, 'display_currency': website.pricelist_id.currency_id}"/>
                </t>
                <span t-field="tickets.price_reduce"
                      t-options="{'widget': 'monetary', 'from_currency': event.company_id.sudo().currency_id, 'display_currency': website.pricelist_id.currency_id}"
                      groups="account.group_show_line_subtotals_tax_excluded"/>
                <span t-field="tickets.price_reduce_taxinc"
                      t-options="{'widget': 'monetary', 'from_currency': event.company_id.sudo().currency_id, 'display_currency': website.pricelist_id.currency_id}"
                      groups="account.group_show_line_subtotals_tax_included"/>
                <span itemprop="price" class="d-none" t-out="tickets.price"/>
                <span itemprop="priceCurrency" class="d-none" t-out="website.pricelist_id.currency_id.name"/>
            </t>
            <span t-else="" class="fw-bold text-uppercase">Free</span>
        </div>
    </xpath>
</template>

</odoo>

```

## File: views\website_sale_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- If the sale order line concerns an event, we want the "product" link to point to the event itself and not to the product on the ecommerce -->
    <template id="cart_line_product_link_inherit_website_event_sale" inherit_id="website_sale.cart_line_product_link" name="Event Shopping Cart Line Product Link">
        <xpath expr="//a" position="attributes">
            <attribute name="t-attf-href"/>
            <attribute name="t-att-href">
                line.event_id and ('/event/%s/register' % slug(line.event_id)) or line.product_id.website_url
            </attribute>
        </xpath>
    </template>

    <!-- If the sale order line concerns an event, we want to show an additional line with the event name even on small screens -->
    <template id="cart_lines_inherit_website_event_sale" inherit_id="website_sale.cart_lines" name="Event Shopping Cart Lines">
        <xpath expr="//t[@t-call='website_sale.cart_line_description_following_lines']/t[@t-set='div_class']" position="after">
            <t t-if="line.event_id">
                <t t-set="div_class" t-value="''"/>
            </t>
        </xpath>
        <xpath expr="//del" position="attributes">
            <attribute name="t-attf-class" separator=" " add="#{line.event_id and 'd-none' or ''}"/>
        </xpath>
    </template>

    <!-- If the sale order line concerns an event, we want to show an additional line with the event name -->
    <template id="cart_popover_inherit_website_event_sale" inherit_id="website_sale.cart_popover" name="Event Cart Popover">
        <xpath expr="//t[@t-call='website_sale.cart_line_product_link']" position="after">
            <t t-if="line.event_id" t-call="website_sale.cart_line_description_following_lines"/>
        </xpath>
    </template>

    <!-- If the sale order line concerns an event, we want to show an additional line with the event name -->
    <template id="cart_summary_inherit_website_event_sale" inherit_id="website_sale.cart_summary" name="Event Cart right column">
        <xpath expr="//td[hasclass('td-product_name')]/div/strong" position="after">
            <t t-if="line.event_id" t-call="website_sale.cart_line_description_following_lines"/>
        </xpath>
    </template>

    <template id="event_confirmation" inherit_id="website_sale.confirmation">
        <xpath expr="//div[@id='oe_structure_website_sale_confirmation_2']" position="inside">
            <t t-if="events">
                <section class="s_title pt40" data-snippet="s_title" data-name="Title">
                    <div class="s_allow_columns container">
                        <h4>
                            We are looking forward to meeting you at the following <t t-if="len(events) == 1">event</t><t t-else="">events</t>:
                        </h4>
                    </div>
                </section>
                <section class="pb32 o_cc o_cc2 o_colored_level bg-transparent">
                    <div class="s_nb_column_fixed s_col_no_bgcolor o_wevent_index" t-foreach="events" t-as="event">
                        <div class="col-lg-12 card mt-3 mx-auto item pt16 pb16">
                            <div class="row s_col_no_bgcolor g-0 align-items-center o_cc1">
                                <div class="col-lg-4 align-self-stretch d-block o_wevent_events_list">
                                    <t t-call="website.record_cover">
                                        <t t-set="_record" t-value="event" />
                                        <div class="o_wevent_event_date position-absolute bg-white shadow-sm text-dark">
                                            <span t-field="event.with_context(tz=event.date_tz).date_begin" t-options="{'format': 'LLL'}" class="o_wevent_event_month" />
                                            <span t-field="event.with_context(tz=event.date_tz).date_begin" t-options="{'format': 'dd'}" class="o_wevent_event_day oe_hide_on_date_edit" />
                                        </div>
                                        <small class="o_wevent_participating text-bg-success">
                                            <i class="fa fa-check me-2" />
                                            Registered
                                        </small>
                                        <small t-if="not event.website_published" class="o_wevent_unpublished text-bg-danger">
                                            <i class="fa fa-ban me-2" />
                                            Unpublished
                                        </small>
                                    </t>
                                </div>
                                <div class="col-lg-8 p-4">
                                    <h3 t-esc="event.name" />
                                    <a class="btn btn-primary mb-2" t-attf-href="/event/#{ slug(event) }">Go to Event</a>
                                </div>
                            </div>
                        </div>
                    </div>
                </section>
            </t>
        </xpath>
    </template>

</odoo>

```

