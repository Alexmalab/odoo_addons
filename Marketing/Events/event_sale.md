# Odoo Module: event_sale

Category: Marketing/Events

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-

from . import models
from . import wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-

{
    'name': 'Events Sales',
    'version': '1.1',
    'category': 'Marketing/Events',
    'website': 'https://www.odoo.com/page/events',
    'description': """
Creating registration with sales orders.
========================================

This module allows you to automate and connect your registration creation with
your main sale flow and therefore, to enable the invoicing feature of registrations.

It defines a new kind of service products that offers you the possibility to
choose an event category associated with it. When you encode a sales order for
that product, you will be able to choose an existing event of that category and
when you confirm your sales order it will automatically create a registration for
this event.
""",
    'depends': ['event', 'sale_management'],
    'data': [
        'views/assets.xml',
        'views/event_views.xml',
        'views/product_views.xml',
        'views/sale_order_views.xml',
        'data/event_sale_data.xml',
        'report/event_event_templates.xml',
        'security/ir.model.access.csv',
        'security/event_security.xml',
        'wizard/event_edit_registration.xml',
        'wizard/event_configurator_views.xml',
    ],
    'demo': ['data/event_demo.xml'],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\event_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="event_0_ticket_1" model="event.event.ticket">
        <field name="name">Standard</field>
        <field name="event_id" ref="event.event_0"/>
        <field name="product_id" ref="product_product_event"/>
        <field name="deadline" eval="(DateTime.today() + timedelta(90)).strftime('%Y-%m-%d')"/>
        <field name="price">1000.0</field>
        <field name="seats_max">50</field>
    </record>

    <record id="event_0_ticket_2" model="event.event.ticket">
        <field name="name">VIP</field>
        <field name="event_id" ref="event.event_0"/>
        <field name="product_id" ref="product_product_event"/>
        <field name="deadline" eval="(DateTime.today() + timedelta(60)).strftime('%Y-%m-%d')"/>
        <field name="price">1500.0</field>
        <field name="seats_max">10</field>
    </record>

    <record id="event_4_ticket_1" model="event.event.ticket">
        <field name="name">General Admission</field>
        <field name="event_id" ref="event.event_4"/>
        <field name="product_id" ref="product_product_event"/>
        <field name="deadline" eval="(DateTime.today() + timedelta(30)).strftime('%Y-%m-%d')"/>
        <field name="price">99.0</field>
        <field name="seats_max">100</field>
    </record>

    <record id="event_2_ticket_1" model="event.event.ticket">
        <field name="name">Standard</field>
        <field name="event_id" ref="event.event_2"/>
        <field name="product_id" ref="product_product_event"/>
        <field name="deadline" eval="(DateTime.today() + timedelta(90)).strftime('%Y-%m-%d')"/>
        <field name="price">1000.0</field>
        <field name="seats_max">50</field>
    </record>

    <record id="event_2_ticket_2" model="event.event.ticket">
        <field name="name">VIP</field>
        <field name="event_id" ref="event.event_2"/>
        <field name="product_id" ref="product_product_event"/>
        <field name="deadline" eval="(DateTime.today() + timedelta(60)).strftime('%Y-%m-%d')"/>
        <field name="price">1500.0</field>
        <field name="seats_max">5</field>
    </record>
</odoo>

```

## File: data\event_sale_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="product_category_events" model="product.category">
            <field name="parent_id" ref="product.product_category_1"/>
            <field name="name">Events</field>
        </record>

        <record id="product_product_event" model="product.product">
            <field name="list_price">10.0</field>
            <field name="event_ok" eval="True"/>
            <field name="standard_price">30.0</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="name">Event Registration</field>
            <field name="default_code">EVENT_REG</field>
            <field name="categ_id" ref="event_sale.product_category_events"/>
            <field name="type">service</field>
        </record>

        <record id="event_type_data_sale" model="event.type">
            <field name="name">Ticketing</field>
            <field name="auto_confirm" eval="False"/>
            <field name="use_mail_schedule" eval="True"/>
            <field name="use_ticketing" eval="True"/>
        </record>
    </data>
</odoo>




```

## File: models\account_invoice.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class AccountMove(models.Model):
    _inherit = 'account.move'

    def action_invoice_paid(self):
        """ When an invoice linked to a sales order selling registrations is
        paid confirm attendees. Attendees should indeed not be confirmed before
        full payment. """
        res = super(AccountMove, self).action_invoice_paid()
        self.mapped('line_ids.sale_line_ids')._update_registrations(confirm=True)
        return res

```

## File: models\event.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError, UserError

from odoo.tools import float_is_zero


class EventType(models.Model):
    _inherit = 'event.type'

    @api.model
    def _get_default_event_ticket_ids(self):
        product = self.env.ref('event_sale.product_product_event', raise_if_not_found=False)
        if not product:
            return False
        return [(0, 0, {
            'name': _('Registration'),
            'product_id': product.id,
            'price': 0,
        })]

    use_ticketing = fields.Boolean('Ticketing')
    event_ticket_ids = fields.One2many(
        'event.event.ticket', 'event_type_id',
        string='Tickets', default=_get_default_event_ticket_ids)

    @api.onchange('name')
    def _onchange_name(self):
        if self.name:
            self.event_ticket_ids.filtered(lambda ticket: ticket.name == _('Registration')).update({
                'name': _('Registration for %s') % self.name
            })


class Event(models.Model):
    _inherit = 'event.event'

    event_ticket_ids = fields.One2many(
        'event.event.ticket', 'event_id', string='Event Ticket',
        copy=True)

    @api.onchange('event_type_id')
    def _onchange_type(self):
        super(Event, self)._onchange_type()
        if self.event_type_id.use_ticketing:
            self.event_ticket_ids = [(5, 0, 0)] + [
                (0, 0, {
                    'name': self.name and _('Registration for %s') % self.name or ticket.name,
                    'product_id': ticket.product_id.id,
                    'price': ticket.price,
                })
                for ticket in self.event_type_id.event_ticket_ids]

    def _is_event_registrable(self):
        res = super(Event, self)._is_event_registrable()
        if res and self.event_ticket_ids:
            return any(
                ticket.product_id.active and not ticket.is_expired and (not ticket.seats_max or ticket.seats_available)
                for ticket in self.event_ticket_ids.with_context(active_test=False))
        return res

class EventTicket(models.Model):
    _name = 'event.event.ticket'
    _description = 'Event Ticket'

    def _default_product_id(self):
        return self.env.ref('event_sale.product_product_event', raise_if_not_found=False)

    name = fields.Char(string='Name', required=True, translate=True)
    event_type_id = fields.Many2one('event.type', string='Event Category', ondelete='cascade')
    event_id = fields.Many2one('event.event', string="Event", ondelete='cascade')
    company_id = fields.Many2one('res.company', related='event_id.company_id')
    product_id = fields.Many2one('product.product', string='Product',
        required=True, domain=[("event_ok", "=", True)],
        default=_default_product_id)
    registration_ids = fields.One2many('event.registration', 'event_ticket_id', string='Registrations')
    price = fields.Float(string='Price', digits='Product Price')
    deadline = fields.Date(string="Sales End")
    is_expired = fields.Boolean(string='Is Expired', compute='_compute_is_expired')

    price_reduce = fields.Float(string="Price Reduce", compute="_compute_price_reduce", digits='Product Price')
    price_reduce_taxinc = fields.Float(compute='_get_price_reduce_tax', string='Price Reduce Tax inc')
    # seats fields
    seats_availability = fields.Selection([('limited', 'Limited'), ('unlimited', 'Unlimited')],
        string='Available Seat', required=True, store=True, compute='_compute_seats', default="limited")
    seats_max = fields.Integer(string='Maximum Available Seats',
       help="Define the number of available tickets. If you have too much registrations you will "
            "not be able to sell tickets anymore. Set 0 to ignore this rule set as unlimited.")
    seats_reserved = fields.Integer(string='Reserved Seats', compute='_compute_seats', store=True)
    seats_available = fields.Integer(string='Available Seats', compute='_compute_seats', store=True)
    seats_unconfirmed = fields.Integer(string='Unconfirmed Seat Reservations', compute='_compute_seats', store=True)
    seats_used = fields.Integer(compute='_compute_seats', store=True)

    def _compute_is_expired(self):
        for record in self:
            if record.deadline:
                current_date = fields.Date.context_today(record.with_context(tz=record.event_id.date_tz))
                record.is_expired = record.deadline < current_date
            else:
                record.is_expired = False

    def _compute_price_reduce(self):
        for record in self:
            product = record.product_id
            discount = product.lst_price and (product.lst_price - product.price) / product.lst_price or 0.0
            record.price_reduce = (1.0 - discount) * record.price

    def _get_price_reduce_tax(self):
        for record in self:
            # sudo necessary here since the field is most probably accessed through the website
            tax_ids = record.sudo().product_id.taxes_id.filtered(lambda r: r.company_id == record.event_id.company_id)
            taxes = tax_ids.compute_all(record.price_reduce, record.event_id.company_id.currency_id, 1.0, product=record.product_id)
            record.price_reduce_taxinc = taxes['total_included']

    @api.depends('seats_max', 'registration_ids.state')
    def _compute_seats(self):
        """ Determine reserved, available, reserved but unconfirmed and used seats. """
        # initialize fields to 0 + compute seats availability
        for ticket in self:
            ticket.seats_availability = 'unlimited' if ticket.seats_max == 0 else 'limited'
            ticket.seats_unconfirmed = ticket.seats_reserved = ticket.seats_used = ticket.seats_available = 0
        # aggregate registrations by ticket and by state
        if self.ids:
            state_field = {
                'draft': 'seats_unconfirmed',
                'open': 'seats_reserved',
                'done': 'seats_used',
            }
            query = """ SELECT event_ticket_id, state, count(event_id)
                        FROM event_registration
                        WHERE event_ticket_id IN %s AND state IN ('draft', 'open', 'done')
                        GROUP BY event_ticket_id, state
                    """
            self.env['event.registration'].flush(['event_id', 'event_ticket_id', 'state'])
            self.env.cr.execute(query, (tuple(self.ids),))
            for event_ticket_id, state, num in self.env.cr.fetchall():
                ticket = self.browse(event_ticket_id)
                ticket[state_field[state]] += num
        # compute seats_available
        for ticket in self:
            if ticket.seats_max > 0:
                ticket.seats_available = ticket.seats_max - (ticket.seats_reserved + ticket.seats_used)

    @api.constrains('registration_ids', 'seats_max')
    def _check_seats_limit(self):
        for record in self:
            if record.seats_max and record.seats_available < 0:
                raise ValidationError(_('No more available seats for this ticket type.'))

    @api.constrains('event_type_id', 'event_id')
    def _constrains_event(self):
        if any(ticket.event_type_id and ticket.event_id for ticket in self):
            raise UserError(_('Ticket cannot belong to both the event category and the event itself.'))

    @api.onchange('product_id')
    def _onchange_product_id(self):
        self.price = self.product_id.list_price or 0

    def get_ticket_multiline_description_sale(self):
        """ Compute a multiline description of this ticket, in the context of sales.
            It will often be used as the default description of a sales order line referencing this ticket.

        1. the first line is the ticket name
        2. the second line is the event name (if it exists, which should be the case with a normal workflow) or the product name (if it exists)

        We decided to ignore entirely the product name and the product description_sale because they are considered to be replaced by the ticket name and event name.
            -> the workflow of creating a new event also does not lead to filling them correctly, as the product is created through the event interface
        """

        name = self.display_name

        if self.event_id:
            name += '\n' + self.event_id.display_name
        elif self.product_id:
            name += '\n' + self.product_id.display_name

        return name


class EventRegistration(models.Model):
    _inherit = 'event.registration'

    event_ticket_id = fields.Many2one('event.event.ticket', string='Event Ticket', readonly=True, states={'draft': [('readonly', False)]})
    # in addition to origin generic fields, add real relational fields to correctly
    # handle attendees linked to sales orders and their lines
    # TDE FIXME: maybe add an onchange on sale_order_id + origin
    sale_order_id = fields.Many2one('sale.order', string='Source Sales Order', ondelete='cascade')
    sale_order_line_id = fields.Many2one('sale.order.line', string='Sales Order Line', ondelete='cascade')
    campaign_id = fields.Many2one('utm.campaign', 'Campaign', related="sale_order_id.campaign_id", store=True)
    source_id = fields.Many2one('utm.source', 'Source', related="sale_order_id.source_id", store=True)
    medium_id = fields.Many2one('utm.medium', 'Medium', related="sale_order_id.medium_id", store=True)

    @api.onchange('event_id')
    def _onchange_event_id(self):
        # We reset the ticket when keeping it would lead to an inconstitent state.
        if self.event_ticket_id and (not self.event_id or self.event_id != self.event_ticket_id.event_id):
            self.event_ticket_id = None

    @api.constrains('event_ticket_id', 'state')
    def _check_ticket_seats_limit(self):
        for record in self:
            if record.event_ticket_id.seats_max and record.event_ticket_id.seats_available < 0:
                raise ValidationError(_('No more available seats for this ticket'))

    def _check_auto_confirmation(self):
        res = super(EventRegistration, self)._check_auto_confirmation()
        if res:
            orders = self.env['sale.order'].search([('state', '=', 'draft'), ('id', 'in', self.mapped('sale_order_id').ids)], limit=1)
            if orders:
                res = False
        return res

    @api.model
    def create(self, vals):
        res = super(EventRegistration, self).create(vals)
        if res.origin or res.sale_order_id:
            res.message_post_with_view('mail.message_origin_link',
                values={'self': res, 'origin': res.sale_order_id},
                subtype_id=self.env.ref('mail.mt_note').id)
        return res

    @api.model
    def _prepare_attendee_values(self, registration):
        """ Override to add sale related stuff """
        line_id = registration.get('sale_order_line_id')
        if line_id:
            registration.setdefault('partner_id', line_id.order_id.partner_id)
        att_data = super(EventRegistration, self)._prepare_attendee_values(registration)
        if line_id:
            att_data.update({
                'event_id': line_id.event_id.id,
                'event_id': line_id.event_id.id,
                'event_ticket_id': line_id.event_ticket_id.id,
                'origin': line_id.order_id.name,
                'sale_order_id': line_id.order_id.id,
                'sale_order_line_id': line_id.id,
            })
        return att_data

    def summary(self):
        res = super(EventRegistration, self).summary()
        if self.event_ticket_id.product_id.image_128:
            res['image'] = '/web/image/product.product/%s/image_128' % self.event_ticket_id.product_id.id
        information = res.setdefault('information', {})
        information.append((_('Name'), self.name))
        information.append((_('Ticket'), self.event_ticket_id.name or _('None')))
        order = self.sale_order_id.sudo()
        order_line = self.sale_order_line_id.sudo()
        if not order or float_is_zero(order_line.price_total, precision_digits=order.currency_id.rounding):
            payment_status = _('Free')
        elif not order.invoice_ids or any(invoice.invoice_payment_state != 'paid' for invoice in order.invoice_ids):
            payment_status = _('To pay')
            res['alert'] = _('The registration must be paid')
        else:
            payment_status = _('Paid')
        information.append((_('Payment'), payment_status))
        return res

```

## File: models\product.py

```python
# -*- coding: utf-8 -*-

from odoo import api, fields, models


class ProductTemplate(models.Model):
    _inherit = 'product.template'

    event_ok = fields.Boolean(string='Is an Event Ticket', help="If checked this product automatically "
      "creates an event registration at the sales order confirmation.")

    @api.onchange('event_ok')
    def _onchange_event_ok(self):
        if self.event_ok:
            self.type = 'service'


class Product(models.Model):
    _inherit = 'product.product'

    event_ticket_ids = fields.One2many('event.event.ticket', 'product_id', string='Event Tickets')

    @api.onchange('event_ok')
    def _onchange_event_ok(self):
        """ Redirection, inheritance mechanism hides the method on the model """
        if self.event_ok:
            self.type = 'service'

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-

from odoo import api, fields, models


class SaleOrder(models.Model):
    _inherit = "sale.order"

    def _action_confirm(self):
        res = super(SaleOrder, self)._action_confirm()
        for so in self:
            # confirm registration if it was free (otherwise it will be confirmed once invoice fully paid)
            so.order_line._update_registrations(confirm=so.amount_total == 0, cancel_to_draft=False)
        return res

    def action_confirm(self):
        res = super(SaleOrder, self).action_confirm()
        for so in self:
            if any(so.order_line.filtered(lambda line: line.event_id)):
                return self.env['ir.actions.act_window'] \
                    .with_context(default_sale_order_id=so.id) \
                    .for_xml_id('event_sale', 'action_sale_order_event_registration')
        return res
    
    def action_cancel(self):
        self.mapped('order_line')._cancel_associated_registrations()
        return super(SaleOrder, self).action_cancel()

    def unlink(self):
        self.mapped('order_line')._unlink_associated_registrations()
        return super(SaleOrder, self).unlink()


class SaleOrderLine(models.Model):

    _inherit = 'sale.order.line'

    event_id = fields.Many2one('event.event', string='Event',
       help="Choose an event and it will automatically create a registration for this event.")
    event_ticket_id = fields.Many2one('event.event.ticket', string='Event Ticket', help="Choose "
        "an event ticket and it will automatically create a registration for this event ticket.")
    event_ok = fields.Boolean(related='product_id.event_ok', readonly=True)

    def _update_registrations(self, confirm=True, cancel_to_draft=False, registration_data=None):
        """ Create or update registrations linked to a sales order line. A sale
        order line has a product_uom_qty attribute that will be the number of
        registrations linked to this line. This method update existing registrations
        and create new one for missing one. """
        Registration = self.env['event.registration'].sudo()
        registrations = Registration.search([('sale_order_line_id', 'in', self.ids)])
        for so_line in self.filtered('event_id'):
            existing_registrations = registrations.filtered(lambda self: self.sale_order_line_id.id == so_line.id)
            if confirm:
                existing_registrations.filtered(lambda self: self.state not in ['open', 'cancel']).confirm_registration()
            if cancel_to_draft:
                existing_registrations.filtered(lambda self: self.state == 'cancel').do_draft()

            for count in range(int(so_line.product_uom_qty) - len(existing_registrations)):
                registration = {}
                if registration_data:
                    registration = registration_data.pop()
                # TDE CHECK: auto confirmation
                registration['sale_order_line_id'] = so_line
                Registration.with_context(registration_force_draft=True).create(
                    Registration._prepare_attendee_values(registration))
        return True

    @api.onchange('product_id')
    def _onchange_product_id(self):
        # We reset the event when keeping it would lead to an inconstitent state.
        # We need to do it this way because the only relation between the product and the event is through the corresponding tickets.
        if self.event_id and (not self.product_id or self.product_id.id not in self.event_id.mapped('event_ticket_ids.product_id.id')):
            self.event_id = None

    @api.onchange('event_id')
    def _onchange_event_id(self):
        # We reset the ticket when keeping it would lead to an inconstitent state.
        if self.event_ticket_id and (not self.event_id or self.event_id != self.event_ticket_id.event_id):
            self.event_ticket_id = None

    @api.onchange('product_uom', 'product_uom_qty')
    def product_uom_change(self):
        if not self.event_ticket_id:
            super(SaleOrderLine, self).product_uom_change()

    @api.onchange('event_ticket_id')
    def _onchange_event_ticket_id(self):
        # we call this to force update the default name
        self.product_id_change()

    def unlink(self):
        self._unlink_associated_registrations()
        super(SaleOrderLine, self).unlink()

    def _cancel_associated_registrations(self):
        self.env['event.registration'].search([('sale_order_line_id', 'in', self.ids)]).button_reg_cancel()

    def _unlink_associated_registrations(self):
        self.env['event.registration'].search([('sale_order_line_id', 'in', self.ids)]).unlink()

    def get_sale_order_line_multiline_description_sale(self, product):
        """ We override this method because we decided that:
                The default description of a sales order line containing a ticket must be different than the default description when no ticket is present.
                So in that case we use the description computed from the ticket, instead of the description computed from the product.
                We need this override to be defined here in sales order line (and not in product) because here is the only place where the event_ticket_id is referenced.
        """
        if self.event_ticket_id:
            ticket = self.event_ticket_id.with_context(
                lang=self.order_id.partner_id.lang,
            )

            return ticket.get_ticket_multiline_description_sale() + self._get_sale_order_line_multiline_description_variants()
        else:
            return super(SaleOrderLine, self).get_sale_order_line_multiline_description_sale(product)

    def _get_display_price(self, product):
        if self.event_ticket_id and self.event_id:
            return self.event_ticket_id.with_context(pricelist=self.order_id.pricelist_id.id, uom=self.product_uom.id).price_reduce
        else:
            return super()._get_display_price(product)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import account_invoice
from . import sale_order
from . import product
from . import event

```

## File: report\event_event_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="event_registration_report_template_badge" inherit_id="event.event_registration_report_template_badge">
        <xpath expr="//div[@id='o_event_name']" position="inside">
            <div t-if="o.event_ticket_id" class="col-12 text-center" style="padding:0px;">
                <div style="background: lightgrey; height: 65px;" class="mt16 text-center">
                    <h3><span t-field="o.event_ticket_id"/></h3>
                </div>
            </div>
        </xpath>
    </template>

    <template id="event_event_report_template_badge" inherit_id="event.event_event_report_template_badge">
        <xpath expr="//div[@id='o_event_attendee_name']" position="inside">
            <div t-if="bool(len(event.event_ticket_ids))" class="col-12" style="padding: 0px;" t-ignore="true">
                <div style="background: lightgrey; height: 65px;" class="mt16 text-center">
                    <h3>Ticket Type</h3>
                </div>
            </div>
        </xpath>
    </template>

</odoo>

```

## File: security\event_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="0">
        <record id="sales_team.group_sale_salesman" model="res.groups">
            <field name="implied_ids" eval="[(4, ref('event.group_event_user'))]"/>
        </record>
    </data>
</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_event_event_ticket_user,event.event.ticket.user,event_sale.model_event_event_ticket,event.group_event_user,1,0,0,0
access_event_event_ticket_admin,event.event.ticket.admin,event_sale.model_event_event_ticket,event.group_event_manager,1,1,1,1
access_product_template_event_manager,product.template.event.manager,product.model_product_template,event.group_event_manager,1,1,1,1
access_product_product_event_manager,product.product.event.manager,product.model_product_product,event.group_event_manager,1,1,1,1

```

## File: static\src\js\event_configurator_controller.js

```javascript
odoo.define('event.EventConfiguratorFormController', function (require) {
"use strict";

var FormController = require('web.FormController');

/**
 * This controller is overridden to allow configuring sale_order_lines through a popup
 * window when a product with 'event_ok' is selected.
 *
 * This allows keeping an editable list view for sales order and remove the noise of
 * those 2 fields ('event_id' + 'event_ticket_id')
 */
var EventConfiguratorFormController = FormController.extend({
    /**
     * We let the regular process take place to allow the validation of the required fields
     * to happen.
     *
     * Then we can manually close the window, providing event information to the caller.
     *
     * @override
     */
    saveRecord: function () {
        var self = this;
        return this._super.apply(this, arguments).then(function () {
            var state = self.renderer.state.data;
            self.do_action({type: 'ir.actions.act_window_close', infos: {
                eventConfiguration: {
                    event_id: {id: state.event_id.data.id},
                    event_ticket_id: {id: state.event_ticket_id.data.id}
                }
            }});
        });
    }
});

return EventConfiguratorFormController;

});

```

## File: static\src\js\event_configurator_view.js

```javascript
odoo.define('event.EventConfiguratorFormView', function (require) {
"use strict";

var EventConfiguratorFormController = require('event.EventConfiguratorFormController');
var FormView = require('web.FormView');
var viewRegistry = require('web.view_registry');

/**
 * @see EventConfiguratorFormController for more information
 */
var EventConfiguratorFormView = FormView.extend({
    config: _.extend({}, FormView.prototype.config, {
        Controller: EventConfiguratorFormController
    }),
});

viewRegistry.add('event_configurator_form', EventConfiguratorFormView);

return EventConfiguratorFormView;

});

```

## File: static\src\js\event_configurator_widget.js

```javascript
odoo.define('event_sale.product_configurator', function (require) {
var ProductConfiguratorWidget = require('sale.product_configurator');

/**
 * Extension of the ProductConfiguratorWidget to support event product configuration.
 * It opens when an event product_product is set.
 *
 * The event information include:
 * - event_id
 * - event_ticket_id
 *
 */
ProductConfiguratorWidget.include({
    /**
     * @returns {boolean}
     *
     * @override
     * @private
     */
    _isConfigurableLine: function () {
        return this.recordData.event_ok || this._super.apply(this, arguments);
    },

    /**
     * @param {integer} productId
     * @param {String} dataPointID
     * @returns {Promise<Boolean>} stopPropagation true if a suitable configurator has been found.
     *
     * @override
     * @private
     */
    _onProductChange: function (productId, dataPointId) {
      var self = this;
      return this._super.apply(this, arguments).then(function (stopPropagation) {
          if (stopPropagation) {
              return Promise.resolve(true);
          } else {
              return self._checkForEvent(productId, dataPointId);
          }
      });
    },

    /**
     * This method will check if the productId needs configuration or not:
     *
     * @param {integer} productId
     * @param {string} dataPointID
     * @returns {Promise<Boolean>} stopPropagation true if the product is an event ticket.
     *
     * @private
     */
    _checkForEvent: function (productId, dataPointId) {
        var self = this;
        return this._rpc({
            model: 'product.product',
            method: 'read',
            args: [productId, ['event_ok']],
        }).then(function (result) {
            if (result && result[0].event_ok) {
                self._openEventConfigurator({
                        default_product_id: productId
                    },
                    dataPointId
                );
                return Promise.resolve(true);
            }
            return Promise.resolve(false);
        });
    },

    /**
     * Opens the event configurator in 'edit' mode.
     *
     * @override
     * @private
     */
    _onEditLineConfiguration: function () {
        if (this.recordData.event_ok) {
            var defaultValues = {
                default_product_id: this.recordData.product_id.data.id
            };

            if (this.recordData.event_id) {
                defaultValues.default_event_id = this.recordData.event_id.data.id;
            }

            if (this.recordData.event_ticket_id) {
                defaultValues.default_event_ticket_id = this.recordData.event_ticket_id.data.id;
            }

            this._openEventConfigurator(defaultValues, this.dataPointID);
        } else {
            this._super.apply(this, arguments);
        }
    },

    /**
     * Opens the event configurator to allow configuring the SO line with events information.
     *
     * When the window is closed, configured values are used to trigger a 'field_changed'
     * event to modify the current SO line.
     *
     * If the window is closed without providing the required values 'event_id' and
     * 'event_ticket_id', the product_id field is cleaned.
     *
     * @param {Object} data various "default_" values
     * @param {string} dataPointId
     *
     * @private
     */
    _openEventConfigurator: function (data, dataPointId) {
        var self = this;
        this.do_action('event_sale.event_configurator_action', {
            additional_context: data,
            on_close: function (result) {
                if (result && !result.special) {
                    self.trigger_up('field_changed', {
                        dataPointID: dataPointId,
                        changes: result.eventConfiguration,
                        onSuccess: function () {
                            // Call post-init function.
                            self._onLineConfigured();
                        }
                    });
                } else {
                    if (!self.recordData.event_id || !self.recordData.event_ticket_id) {
                        self.trigger_up('field_changed', {
                            dataPointID: dataPointId,
                            changes: {
                                product_id: false,
                                name: ''
                            },
                        });
                    }
                }
            }
        });
    }
});


return ProductConfiguratorWidget;

});

```

## File: views\assets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<template id="assets_backend" inherit_id="web.assets_backend" name="event_sale assets backend">
    <xpath expr="script[last()]" position="after">
        <script type="text/javascript" src="/event_sale/static/src/js/event_configurator_controller.js"></script>
        <script type="text/javascript" src="/event_sale/static/src/js/event_configurator_view.js"></script>
        <script type="text/javascript" src="/event_sale/static/src/js/event_configurator_widget.js"></script>
    </xpath>
</template>
<template id="assets_tests" name="Event Sale Assets Tests" inherit_id="web.assets_tests">
    <xpath expr="." position="inside">
        <script type="text/javascript" src="/event_sale/static/tests/tours/event_configurator_ui.js"></script>
    </xpath>
</template>
<template id="qunit_suite" inherit_id="web.qunit_suite" name="event_sale_tests">
    <xpath expr="//t[@t-set='head']" position="inside">
        <script type="text/javascript" src="/event_sale/static/tests/event_configurator.test.js"></script>
    </xpath>
</template>
</odoo>

```

## File: views\event_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="view_event_registration_ticket_search" model="ir.ui.view">
            <field name="name">event.registration.search.inherit</field>
            <field name="model">event.registration</field>
            <field name="inherit_id" ref="event.view_registration_search" />
            <field name="arch" type="xml">
                <filter name="group_event" position="after">
                    <filter string="Ticket Type" name ="tickettype" domain="['|', ('company_id', '=', False), ('company_id', '=', parent.company_id)]" context="{'group_by':'event_ticket_id'}"/>
                </filter>
            </field>
        </record>

        <record id="view_event_registration_ticket_tree" model="ir.ui.view">
            <field name="name">event.registration.tree.inherit</field>
            <field name="model">event.registration</field>
            <field name="inherit_id" ref="event.view_event_registration_tree" />
            <field name="arch" type="xml">
                <field name="event_id" position="after">
                    <field name="event_ticket_id"/>
                    <field name="origin"/>
                </field>
            </field>
        </record>

        <record id="view_event_registration_ticket_form" model="ir.ui.view">
            <field name="name">event.registration.form.inherit</field>
            <field name="model">event.registration</field>
            <field name="inherit_id" ref="event.view_event_registration_form" />
            <field name="arch" type="xml">
                <field name="event_id" position="after">
                    <field
                        name="event_ticket_id"
                        domain="[
                            ('event_id', '=', event_id),
                            '|', ('seats_availability', '=', 'unlimited'), ('seats_available', '>', 0)
                        ]"
                        attrs="{'invisible': [('event_id', '=', False)]}"
                        options="{'no_open': True, 'no_create': True}"
                    />
                </field>
                <group name="event" position="after">
                    <group string="Origin">
                        <field name="sale_order_id"/>
                        <field name="origin" attrs="{'invisible': [('sale_order_id', '!=', False)]}"/>
                        <field name="sale_order_line_id" readonly="1" attrs="{'invisible': [('sale_order_id', '=', False)]}"/>
                    </group>
                    <group string="Marketing" name="utm_link" groups="base.group_no_one">
                        <field name="campaign_id"/>
                        <field name="medium_id"/>
                        <field name="source_id"/>
                    </group>
                </group>
            </field>
        </record>

        <record id="event_type_view_form_inherit_sale" model="ir.ui.view">
            <field name="name">event.type.view.form.inherit.sale</field>
            <field name="model">event.type</field>
            <field name="inherit_id" ref="event.view_event_type_form"/>
            <field name="arch" type="xml">
                <div name="event_type_attendees_auto_confirm" position="after">
                    <div class="col-12 col-lg-12 o_setting_box">
                        <div class="o_setting_left_pane">
                            <field name="use_ticketing"/>
                        </div>
                        <div class="o_setting_right_pane">
                            <label for="use_ticketing"/>
                            <div class="row mt16" attrs="{'invisible': [('use_ticketing', '=', False)]}">
                                <div class="col-lg-9">
                                    <field name="event_ticket_ids">
                                        <tree string="Tickets" editable="bottom">
                                            <field name="name"/>
                                            <field name="product_id" context="{'default_type': 'service'}"/>
                                            <field name="price"/>
                                        </tree>
                                    </field>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </field>
        </record>

        <record id="view_event_form_inherit_ticket" model="ir.ui.view">
            <field name="name">event.form.inherit</field>
            <field name="model">event.event</field>
            <field name="inherit_id" ref="event.view_event_form"/>
            <field name="arch" type="xml">
                <page name="event_communication" position="before">
                    <page string="Tickets">
                        <field name="event_ticket_ids" context="{'default_name': name}" mode="tree,kanban">
                            <tree string="Tickets" editable="bottom">
                                <field name="name"/>
                                <field name="product_id" context="{'default_event_ok':1, 'default_type': 'service'}"/>
                                <field name="deadline"/>
                                <field name="price"/>
                                <field name="seats_max"/>
                                <field name="seats_reserved" readonly="1"/>
                                <field name="seats_unconfirmed" readonly="1"/>
                            </tree>
                            <kanban class="o_kanban_mobile">
                                <field name="product_id"/>
                                <field name="name"/>
                                <field name="price"/>
                                <field name="seats_max"/>
                                <field name="seats_reserved"/>
                                <field name="seats_unconfirmed"/>
                                <templates>
                                    <t t-name="kanban-box">
                                        <div t-attf-class="oe_kanban_card oe_kanban_global_click">
                                            <div class="row">
                                                <div class="col-8">
                                                    <strong><t t-esc="record.name.value"/></strong>
                                                </div>
                                                <div class="col-4 text-right"><strong> <t t-esc="record.price.value"/></strong></div>
                                            </div>
                                            <div t-esc="record.product_id.value"/>
                                            <div><i>
                                            <t t-esc="record.seats_reserved.value"/> reserved + <t t-esc="record.seats_reserved.value"/> unconfirmed
                                            </i></div>
                                        </div>
                                    </t>
                                </templates>
                            </kanban>
                            <form string="Tickets">
                                <group>
                                    <group>
                                        <field name="name"/>
                                        <field name="product_id" context="{'default_event_ok':1}"/>
                                        <field name="deadline"/>
                                        <field name="price"/>
                                    </group><group>
                                        <field name="seats_max"/>
                                        <field name="seats_reserved" readonly="1"/>
                                        <field name="seats_unconfirmed" readonly="1"/>
                                    </group>
                                </group>
                            </form>
                        </field>
                    </page>
                </page>
            </field>
        </record>

        <record id="event_ticket_form_view" model="ir.ui.view">
            <field name="name">event.event.ticket.form</field>
            <field name="model">event.event.ticket</field>
            <field name="arch" type="xml">
                <form string="Event's Ticket">
                    <div class="oe_title">
                        <label for="name" class="oe_edit_only"/>
                        <h1><field name="name" placeholder="Event Name"/></h1>
                    </div>
                    <group>
                        <group>
                            <field name="event_id"/>
                            <field name="seats_availability"/>
                            <field name="seats_available"/>
                            <field name="deadline"/>
                            <field name="price"/>
                            <field name="price_reduce" groups="base.group_no_one"/>
                        </group>
                        <group>
                            <field name="seats_max"/>
                            <field name="seats_reserved"/>
                            <field name="seats_unconfirmed"/>
                            <field name="seats_used"/>
                            <field name="product_id"/>
                            <field name="is_expired"/>
                        </group>
                    </group>
                </form>
            </field>
        </record>

</odoo>

```

## File: views\product_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="event_sale_product_template_form" model="ir.ui.view">
             <field name="name">product.template.event.form.inherit</field>
             <field name="model">product.template</field>
             <field name="inherit_id" ref="product.product_template_form_view" />
             <field name="arch" type="xml">
                <group name="email_template_and_project" position="after">
                    <group string="Events">
                        <field name="event_ok" />
                    </group>
                </group>
             </field>
        </record>
</odoo>

```

## File: views\sale_order_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="sale_order_view_form" model="ir.ui.view">
        <field name="name">sale.order.form.inherit.event.sale</field>
        <field name="model">sale.order</field>
        <field name="inherit_id" ref="sale.view_order_form" />
        <field name="arch" type="xml">
            <xpath expr="//field[@name='order_line']//form//field[@name='product_id']" position="after">
                <field
                    name="event_id"
                    domain="[
                        ('event_ticket_ids.product_id','=', product_id),
                        ('date_end','&gt;=',time.strftime('%Y-%m-%d 00:00:00')),
                        '|', ('company_id', '=', False), ('company_id', '=', parent.company_id)
                    ]"
                    attrs="{'invisible': [('event_ok', '=', False)], 'required': [('event_ok', '!=', False)]}"
                    options="{'no_open': True, 'no_create': True}"
                />
                <field
                    name="event_ticket_id"
                    domain="[
                        ('event_id', '=', event_id),
                        ('product_id','=',product_id),
                        '|', ('seats_availability', '=', 'unlimited'), ('seats_available', '>', 0), '|', ('company_id', '=', False), ('company_id', '=', parent.company_id)
                    ]"
                    attrs="{
                        'invisible': ['|', ('event_ok', '=', False), ('event_id', '=', False)],
                        'required': [('event_ok', '!=', False), ('event_id', '!=', False)],
                    }"
                    options="{'no_open': True, 'no_create': True}"
                />
                <field name="event_ok" invisible="1"/>
            </xpath>
            <xpath expr="//field[@name='order_line']//tree//field[@name='product_template_id']" position="after">
                <field name="event_ok" invisible="1" />
                <field name="event_id" optional="hide" domain="['|', ('company_id', '=', False), ('company_id', '=', parent.company_id)]"/>
                <field name="event_ticket_id" optional="hide" domain="['|', ('company_id', '=', False), ('company_id', '=', parent.company_id)]"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: wizard\event_configurator.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields


class EventConfigurator(models.TransientModel):
    _name = 'event.event.configurator'
    _description = 'Event Configurator'

    product_id = fields.Many2one('product.product', string="Product", readonly=True)
    event_id = fields.Many2one('event.event', string="Event")
    event_ticket_id = fields.Many2one('event.event.ticket', string="Event Ticket")

```

## File: wizard\event_configurator_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="event_configurator_view_form" model="ir.ui.view">
        <field name="name">event.configurator.view.form</field>
        <field name="model">event.event.configurator</field>
        <field name="arch" type="xml">
            <form js_class="event_configurator_form">
                <group>
                    <field
                        name="event_id"
                        domain="[
                            ('event_ticket_ids.product_id','=', product_id),
                            ('date_end','&gt;=',time.strftime('%Y-%m-%d 00:00:00'))
                        ]"
                        required="1"
                        options="{'no_open': True, 'no_create': True}"
                    />
                    <field
                        name="event_ticket_id"
                        domain="[
                            ('event_id', '=', event_id),
                            ('product_id','=',product_id),
                            '|', ('seats_availability', '=', 'unlimited'), ('seats_available', '>', 0)
                        ]"
                        attrs="{
                            'invisible': [('event_id', '=', False)],
                            'required': [('event_id', '!=', False)],
                        }"
                        options="{'no_open': True, 'no_create': True}"
                    />
                    <field name="product_id" invisible="1"/>
                </group>
                <footer>
                    <button string="Ok" class="btn-primary o_event_sale_js_event_configurator_ok" special="save"/>
                    <button string="Cancel" class="btn-secondary" special="cancel"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="event_configurator_action" model="ir.actions.act_window">
        <field name="name">Configure an event</field>
        <field name="res_model">event.event.configurator</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
        <field name="view_id" ref="event_configurator_view_form"/>
    </record>
</odoo>

```

## File: wizard\event_edit_registration.py

```python
# -*- coding: utf-8 -*-

from odoo import models, fields, api


class RegistrationEditor(models.TransientModel):
    _name = "registration.editor"
    _description = 'Edit Attendee Details on Sales Confirmation'

    sale_order_id = fields.Many2one('sale.order', 'Sales Order', required=True, ondelete='cascade')
    event_registration_ids = fields.One2many('registration.editor.line', 'editor_id', string='Registrations to Edit')

    @api.model
    def default_get(self, fields):
        res = super(RegistrationEditor, self).default_get(fields)
        if not res.get('sale_order_id'):
            sale_order_id = res.get('sale_order_id', self._context.get('active_id'))
            res['sale_order_id'] = sale_order_id
        sale_order = self.env['sale.order'].browse(res.get('sale_order_id'))
        registrations = self.env['event.registration'].search([
            ('sale_order_id', '=', sale_order.id),
            ('event_ticket_id', 'in', sale_order.mapped('order_line.event_ticket_id').ids),
            ('state', '!=', 'cancel')])

        attendee_list = []
        for so_line in [l for l in sale_order.order_line if l.event_ticket_id]:
            existing_registrations = [r for r in registrations if r.event_ticket_id == so_line.event_ticket_id]
            for reg in existing_registrations:
                attendee_list.append([0, 0, {
                    'event_id': reg.event_id.id,
                    'event_ticket_id': reg.event_ticket_id.id,
                    'registration_id': reg.id,
                    'name': reg.name,
                    'email': reg.email,
                    'phone': reg.phone,
                    'mobile': reg.mobile,
                    'sale_order_line_id': so_line.id,
                }])
            for count in range(int(so_line.product_uom_qty) - len(existing_registrations)):
                attendee_list.append([0, 0, {
                    'event_id': so_line.event_id.id,
                    'event_ticket_id': so_line.event_ticket_id.id,
                    'sale_order_line_id': so_line.id,
                }])
        res['event_registration_ids'] = attendee_list
        res = self._convert_to_write(res)
        return res

    def action_make_registration(self):
        self.ensure_one()
        for registration_line in self.event_registration_ids:
            values = registration_line.get_registration_data()
            if registration_line.registration_id:
                registration_line.registration_id.write(values)
            else:
                self.env['event.registration'].create(values)
        if self.env.context.get('active_model') == 'sale.order':
            for order in self.env['sale.order'].browse(self.env.context.get('active_ids', [])):
                order.order_line._update_registrations(confirm=False)
        return {'type': 'ir.actions.act_window_close'}


class RegistrationEditorLine(models.TransientModel):
    """Event Registration"""
    _name = "registration.editor.line"
    _description = 'Edit Attendee Line on Sales Confirmation'

    editor_id = fields.Many2one('registration.editor')
    sale_order_line_id = fields.Many2one('sale.order.line', string='Sales Order Line')
    event_id = fields.Many2one('event.event', string='Event', required=True)
    registration_id = fields.Many2one('event.registration', 'Original Registration')
    event_ticket_id = fields.Many2one('event.event.ticket', string='Event Ticket')
    email = fields.Char(string='Email')
    phone = fields.Char(string='Phone')
    mobile = fields.Char(string='Mobile')
    name = fields.Char(string='Name', index=True)

    def get_registration_data(self):
        self.ensure_one()
        return {
            'event_id': self.event_id.id,
            'event_ticket_id': self.event_ticket_id.id,
            'partner_id': self.editor_id.sale_order_id.partner_id.id,
            'name': self.name or self.editor_id.sale_order_id.partner_id.name,
            'phone': self.phone or self.editor_id.sale_order_id.partner_id.phone,
            'mobile': self.mobile or self.editor_id.sale_order_id.partner_id.mobile,
            'email': self.email or self.editor_id.sale_order_id.partner_id.email,
            'origin': self.editor_id.sale_order_id.name,
            'sale_order_id': self.editor_id.sale_order_id.id,
            'sale_order_line_id': self.sale_order_line_id.id,
        }

```

## File: wizard\event_edit_registration.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

        <record id="view_event_registration_editor_form" model="ir.ui.view">
            <field name="name">registration.editor.form</field>
            <field name="model">registration.editor</field>
            <field name="arch" type="xml">
                <form string="Registration">
                    <p>Before confirming <field name="sale_order_id" readonly="1" class="oe_inline"/>
                    please give details about the registrations</p>
                    <field name="event_registration_ids">
                        <tree string="Registration" editable="top" create="false" delete="false">
                            <field name="event_id" readonly='1' force_save="1"/>
                            <field name="registration_id" readonly='1' force_save="1"/>
                            <field name="event_ticket_id" domain="[('event_id', '=', event_id)]" readonly='1' force_save="1"/>
                            <field name="name"/>
                            <field name="email"/>
                            <field name="mobile" class="o_force_ltr"/>
                            <field name="phone" class="o_force_ltr"/>
                        </tree>
                    </field>
                    <footer>
                        <button string="Apply" name="action_make_registration" type="object" class="btn-primary"/>
                        <button string="Skip" class="btn-secondary" special="cancel"/>
                    </footer>
                </form>
            </field>
        </record>

        <record id="action_sale_order_event_registration" model="ir.actions.act_window">
            <field name="name">Event Registrations</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">registration.editor</field>
            <field name="view_mode">form</field>
            <field name="view_id" ref="view_event_registration_editor_form"/>
            <field name="target">new</field>
            <field name="context">{}</field>
        </record>
</odoo>

```

## File: wizard\__init__.py

```python
from . import event_edit_registration
from . import event_configurator

```

