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
        'web.assets_frontend': [
            'website_event_sale/static/src/scss/*.scss',
            'website_event_sale/static/src/js/*.js',
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
from odoo.http import request, route

from odoo.addons.website_event.controllers.main import WebsiteEventController


class WebsiteEventSaleController(WebsiteEventController):

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
                request.session['sale_last_order_id'] = order_sudo.id
                return request.redirect("/shop/checkout")
            # free tickets -> order with amount = 0: auto-confirm, no checkout
            elif order_sudo:
                order_sudo.action_confirm()  # tde notsure: email sending ?
                request.website.sale_reset()

        return res

```

## File: controllers\payment.py

```python
from odoo.http import request
from odoo.addons.website_sale.controllers.main import PaymentPortal


class PaymentPortalOnsite(PaymentPortal):

    def _validate_transaction_for_order(self, transaction, sale_order_id):
        """
        Throws a ValidationError if the user tries to pay for a ticket which isn't available
        """
        super()._validate_transaction_for_order(transaction, sale_order_id)

        count_per_ticket = request.env['event.registration'].sudo()._read_group(
            [('sale_order_id', '=', sale_order_id), ('state', '!=', 'cancel'), ('event_ticket_id', '!=', False)],
            ['event_ticket_id'], ['__count']
        )
        for ticket, count in count_per_ticket:
            ticket._check_seats_availability(minimal_availability=count)

        count_per_event = request.env['event.registration'].sudo()._read_group(
            [('sale_order_id', '=', sale_order_id), ('state', '!=', 'cancel'), ('event_ticket_id', '!=', False)],
            ['event_id'], ['__count']
        )
        for event, count in count_per_event:
            event._check_seats_availability(minimal_availability=count)

```

## File: controllers\sale.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.addons.website_sale.controllers.main import WebsiteSale
from odoo.http import request


class WebsiteEventSale(WebsiteSale):

    def _prepare_shop_payment_confirmation_values(self, order):
        values = super(WebsiteEventSale,
                       self)._prepare_shop_payment_confirmation_values(order)
        values['events'] = order.order_line.event_id
        attendee_per_event_read_group = request.env['event.registration'].sudo()._read_group(
            [('sale_order_id', '=', order.id), ('state', 'in', ['open', 'done'])],
            groupby=['event_id'],
            aggregates=['id:array_agg'],
        )
        values['attendee_ids_per_event'] = dict(attendee_per_event_read_group)
        return values

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-

from . import main
from . import payment
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
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M13.238 11.483a1.927 1.927 0 0 1 .654-2.55c1.403-.912 2.927-1.233 4.181-.613.2.1.376.24.54.39L49.26 37.346c.472.426.741 1.025.741 1.653H28L13.238 11.483Z" fill="#2EBCFA"/><path d="M50 39c0 1.657-4.925 3-11 3s-11-1.343-11-3 4.925-3 11-3 11 1.343 11 3Z" fill="#088BF5"/><path d="M36.762 11.483a1.927 1.927 0 0 0-.654-2.55c-1.403-.912-2.927-1.233-4.181-.613-.2.1-.376.24-.54.39L.74 37.346A2.226 2.226 0 0 0 0 39h22l14.762-27.517Z" fill="#985184"/><path d="M31.693 20.93 25 14.677l-6.693 6.255L25 33.407l6.693-12.476Z" fill="#144496"/><path d="M0 39c0 1.657 4.925 3 11 3s11-1.343 11-3-4.925-3-11-3-11 1.343-11 3Z" fill="#712258"/></svg>

```

## File: static\src\js\website_event_sale_ticket_details.js

```javascript
/** @odoo-module **/

import publicWidget from '@web/legacy/js/public/public_widget';

publicWidget.registry.ticketDetailsWidget.include({
    /**
     * Overriding the method to toggle the tickets registration
     * pricelist dropdown visibility on ticket details click
     */
    _onTicketDetailsClick: function(ev) {
        this._super(...arguments);
        if (this.foldedByDefault){
            $(ev.currentTarget).siblings('#o_wevent_tickets_pricelist').toggleClass('collapse');
        }
    }
});

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

<template id="modal_ticket_registration" inherit_id="website_event.modal_ticket_registration">
    <!-- Change pricelist -->
    <xpath expr="//div[hasclass('o_wevent_price_range')]" position="after">
        <div id="o_wevent_tickets_pricelist" class="collapse show">
            <t t-set="website_sale_pricelists" t-value="website.get_pricelist_available(show_visible=True)" />
            <t t-set="hasPricelistDropdown" t-value="website_sale_pricelists and len(website_sale_pricelists)&gt;1"/>
            <t t-call="website_sale.pricelist_list">
                <t t-set="_classes" t-valuef="d-inline p-0 ms-2 my-1"/>
            </t>
        </div>
    </xpath>
    <!-- Add price information on tickets (multi tickets, aka in collapse) -->
    <xpath expr="//div[hasclass('o_wevent_registration_multi_select')]" position="inside">
        <t t-if="ticket.price">
            <t t-if="(ticket.price - ticket.price_reduce) &gt; 0 and website.pricelist_id.discount_policy == 'without_discount'">
                <del t-if="website.show_line_subtotals_tax_selection == 'tax_excluded'"
                     class="text-danger me-1"
                     t-field="ticket.price"
                     t-options="{'widget': 'monetary', 'from_currency': event.company_id.sudo().currency_id, 'display_currency': website.currency_id}"/>
                <del t-else=""
                     class="text-danger me-1"
                     t-field="ticket.price_incl"
                     t-options="{'widget': 'monetary', 'from_currency': event.company_id.sudo().currency_id, 'display_currency': website.currency_id}"/>
            </t>
            <span t-if="website.show_line_subtotals_tax_selection == 'tax_excluded'"
                  class="fs-6"
                  t-field="ticket.price_reduce"
                  t-options="{'widget': 'monetary', 'from_currency': event.company_id.sudo().currency_id, 'display_currency': website.currency_id}"/>
            <span t-else=""
                  t-field="ticket.price_reduce_taxinc"
                  class="fs-6"
                  t-options="{'widget': 'monetary', 'from_currency': event.company_id.sudo().currency_id, 'display_currency': website.currency_id}"/>
            <span itemprop="price" class="d-none" t-out="ticket.price"/>
            <span itemprop="priceCurrency" class="d-none" t-out="website.currency_id.name"/>
        </t>
        <span t-else="" class="badge text-bg-success rounded-pill p-2 px-3">Free</span>
    </xpath>
    <xpath expr="//div[hasclass('o_wevent_price_range')]" position="inside">
        <t t-if="website.show_line_subtotals_tax_selection == 'tax_excluded'" t-set="all_prices" t-value="event.event_ticket_ids.mapped('price_reduce')"/>
        <t t-else="" t-set="all_prices" t-value="event.event_ticket_ids.mapped('price_reduce_taxinc')"/>
        <t t-set="lowest_price" t-value="min(all_prices)"/>
        <t t-set="highest_price" t-value="max(all_prices)"/>
        <t t-if="highest_price > 0">
            <small class="text-muted">
                From
                <span t-out="lowest_price" t-options="{'widget': 'monetary', 'from_currency': event.company_id.sudo().currency_id, 'display_currency': website.currency_id}"/>
                <t t-if="lowest_price != highest_price">
                    to
                    <span t-out="highest_price" t-options="{'widget': 'monetary', 'from_currency': event.company_id.sudo().currency_id, 'display_currency': website.currency_id}"/>
                </t>
            </small>
        </t>
    </xpath>
    <!-- Add price information on tickets (mono ticket, aka not in collapse) -->
    <xpath expr="//div[hasclass('o_wevent_registration_single_select')]" position="before">
        <div class="flex-md-grow-1 me-2 text-end">
            <t t-if="tickets.price">
                <t t-if="(tickets.price - tickets.price_reduce) &gt;0 and website.pricelist_id.discount_policy == 'without_discount'">
                    <del t-if="website.show_line_subtotals_tax_selection == 'tax_excluded'"
                         class="text-danger me-1"
                         t-field="tickets.price"
                         t-options="{'widget': 'monetary', 'from_currency': event.company_id.sudo().currency_id, 'display_currency': website.currency_id}"/>
                    <del t-else=""
                         class="text-danger me-1"
                         t-field="tickets.price_incl"
                         t-options="{'widget': 'monetary', 'from_currency': event.company_id.sudo().currency_id, 'display_currency': website.currency_id}"/>
                </t>
                <span t-if="website.show_line_subtotals_tax_selection == 'tax_excluded'"
                      class="badge text-bg-secondary fs-6"
                      t-field="tickets.price_reduce"
                      t-options="{'widget': 'monetary', 'from_currency': event.company_id.sudo().currency_id, 'display_currency': website.currency_id}"/>
                <span t-else=""
                      class="badge text-bg-secondary fs-6"
                      t-field="tickets.price_reduce_taxinc"
                      t-options="{'widget': 'monetary', 'from_currency': event.company_id.sudo().currency_id, 'display_currency': website.currency_id}"/>
                <span itemprop="price" class="d-none" t-out="tickets.price"/>
                <span itemprop="priceCurrency" class="d-none" t-out="website.currency_id.name"/>
            </t>
            <span t-else="" class="badge text-bg-secondary fs-6 text-uppercase">Free</span>
        </div>
    </xpath>
</template>

<template id="registration_attendee_details" inherit_id="website_event.registration_attendee_details">
    <!-- Change 'continue' button to use the website login settings -->
    <xpath expr="//div[hasclass('modal-footer')]//button[hasclass('btn-primary')]" position="replace">
        <t t-if="website.account_on_checkout == 'mandatory' and website.is_public_user()">
            <a class="btn btn-primary" t-attf-href="/web/login?redirect=/event/#{slug(event)}">
                <span>Sign In</span>
                <span class="fa fa-sign-in"/>
            </a>
        </t>
        <t t-else="">
            <t t-set="has_paying_ticket" t-value="any(ticket.get('price', 0) > 0 for ticket in tickets)"/>
            <button t-if="availability_check" type="submit" class="btn btn-primary">
                <span t-if="has_paying_ticket">Go to Payment</span>
                <span t-else="">Confirm Registration</span>
            </button>
        </t>
    </xpath>
</template>

</odoo>

```

## File: views\website_sale_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="index_topbar" inherit_id="website_event.index_topbar">
        <xpath expr="//div[hasclass('o_wevent_index_topbar_filters')]" position="inside">
            <t t-set="website_sale_pricelists" t-value="website.get_pricelist_available(show_visible=True)" />
            <t t-set="hasPricelistDropdown" t-value="website_sale_pricelists and len(website_sale_pricelists)&gt;1"/>
            <t t-call="website_sale.pricelist_list">
                <t t-set="_classes" t-valuef="d-none d-lg-inline me-2 my-1"/>
            </t>
        </xpath>
    </template>

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
    <template id="cart_summary_inherit_website_event_sale" inherit_id="website_sale.checkout_layout" name="Event Cart right column">
        <xpath expr="//td[@name='website_sale_cart_summary_product_name']/h6" position="after">
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
                                        <small t-if="event.is_participating" class="o_wevent_participating text-bg-success">
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
                                    <t t-set="attendee_ids" t-value="attendee_ids_per_event.get(event, [])"/>
                                    <a t-if="order.state == 'sale' and attendee_ids" class="btn btn-primary text-white mb-2 me-2" target="_blank"
                                       t-attf-href="/event/{{ event.id }}/my_tickets?registration_ids={{ attendee_ids }}&amp;tickets_hash={{ event._get_tickets_access_hash(attendee_ids) }}">
                                        Download Tickets <i class="ms-1 fa fa-download"/>
                                    </a>
                                    <a class="mb-2" t-attf-href="/event/#{ slug(event) }">Go to Event</a>
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

