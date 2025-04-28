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
    'version': '1.2',
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
        'views/event_ticket_views.xml',
        'views/event_registration_views.xml',
        'views/event_views.xml',
        'views/product_views.xml',
        'views/sale_order_views.xml',
        'data/event_sale_data.xml',
        'data/mail_data.xml',
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

    <record id="event.event_0_ticket_0" model="event.event.ticket">
        <field name="product_id" ref="product_product_event"/>
        <field name="price">0</field>
    </record>
    <record id="event.event_0_ticket_1" model="event.event.ticket">
        <field name="product_id" ref="product_product_event"/>
        <field name="price">1000.0</field>
    </record>
    <record id="event.event_0_ticket_2" model="event.event.ticket">
        <field name="product_id" ref="product_product_event"/>
        <field name="price">1500.0</field>
    </record>

    <record id="event.event_2_ticket_1" model="event.event.ticket">
        <field name="product_id" ref="product_product_event"/>
        <field name="price">1000.0</field>
    </record>
    <record id="event.event_2_ticket_2" model="event.event.ticket">
        <field name="product_id" ref="product_product_event"/>
        <field name="price">1500.0</field>
    </record>

    <record id="event.event_4_ticket_0" model="event.event.ticket">
        <field name="product_id" ref="product_product_event"/>
        <field name="price">99.0</field>
    </record>

    <record id="event.event_7_ticket_1" model="event.event.ticket">
        <field name="product_id" ref="product_product_event"/>
        <field name="price">0.0</field>
    </record>
    <record id="event.event_7_ticket_2" model="event.event.ticket">
        <field name="product_id" ref="product_product_event"/>
        <field name="price">1000.0</field>
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
            <field name="description_sale" eval="False"/>
            <field name="categ_id" ref="event_sale.product_category_events"/>
            <field name="type">service</field>
        </record>
    </data>
</odoo>




```

## File: data\mail_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="event_ticket_id_change_exception" name="Message: Alert on event ticket id change">
    <div>
        <p>
            <span>Registration modification for attendee:</span>
            <a href="#" data-oe-model="event.registration" t-att-data-oe-id="registration.id"><t t-esc="registration.name"/></a>.
            <span>Manual actions may be needed.</span>
        </p>
        <div class="mt16">
            <p>Exception:</p>
            <ul>
                <li>
                    <a href="#" data-oe-model="event.registration" t-att-data-oe-id="registration.id"><t t-esc="registration.name"/></a>:
                    <span>Ticket changed from <strong><t t-esc="old_ticket_name"/></strong> to <strong><t t-esc="new_ticket_name"/></strong></span>
                </li>
            </ul>
        </div>
    </div>
</template>

</odoo>

```

## File: models\account_move.py

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
        self.mapped('line_ids.sale_line_ids')._update_registrations(confirm=True, mark_as_paid=True)
        return res

```

## File: models\event_event.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class Event(models.Model):
    _inherit = 'event.event'

    sale_order_lines_ids = fields.One2many(
        'sale.order.line', 'event_id',
        groups='sales_team.group_sale_salesman',
        string='All sale order lines pointing to this event')
    sale_price_subtotal = fields.Monetary(
        string='Sales (Tax Excluded)', compute='_compute_sale_price_subtotal',
        groups='sales_team.group_sale_salesman')
    currency_id = fields.Many2one(
        'res.currency', string='Currency',
        related='company_id.currency_id', readonly=True)

    @api.depends('company_id.currency_id',
                 'sale_order_lines_ids.price_subtotal', 'sale_order_lines_ids.currency_id',
                 'sale_order_lines_ids.company_id', 'sale_order_lines_ids.order_id.date_order')
    def _compute_sale_price_subtotal(self):
        """ Takes all the sale.order.lines related to this event and converts amounts
        from the currency of the sale order to the currency of the event company.

        To avoid extra overhead, we use conversion rates as of 'today'.
        Meaning we have a number that can change over time, but using the conversion rates
        at the time of the related sale.order would mean thousands of extra requests as we would
        have to do one conversion per sale.order (and a sale.order is created every time
        we sell a single event ticket). """
        date_now = fields.Datetime.now()
        sale_price_by_event = {}
        if self.ids:
            event_subtotals = self.env['sale.order.line'].read_group(
                [('event_id', 'in', self.ids),
                 ('price_subtotal', '!=', 0)],
                ['event_id', 'currency_id', 'price_subtotal:sum'],
                ['event_id', 'currency_id'],
                lazy=False
            )

            company_by_event = {
                event._origin.id or event.id: event.company_id
                for event in self
            }

            currency_by_event = {
                event._origin.id or event.id: event.currency_id
                for event in self
            }

            currency_by_id = {
                currency.id: currency
                for currency in self.env['res.currency'].browse(
                    [event_subtotal['currency_id'][0] for event_subtotal in event_subtotals]
                )
            }

            for event_subtotal in event_subtotals:
                price_subtotal = event_subtotal['price_subtotal']
                event_id = event_subtotal['event_id'][0]
                currency_id = event_subtotal['currency_id'][0]
                sale_price = currency_by_event[event_id]._convert(
                    price_subtotal,
                    currency_by_id[currency_id],
                    company_by_event[event_id],
                    date_now)
                if event_id in sale_price_by_event:
                    sale_price_by_event[event_id] += sale_price
                else:
                    sale_price_by_event[event_id] = sale_price

        for event in self:
            event.sale_price_subtotal = sale_price_by_event.get(event._origin.id or event.id, 0)

    def action_view_linked_orders(self):
        """ Redirects to the orders linked to the current events """
        sale_order_action = self.env["ir.actions.actions"]._for_xml_id("sale.action_orders")
        sale_order_action.update({
            'domain': [('state', '!=', 'cancel'), ('order_line.event_id', 'in', self.ids)],
            'context': {'create': 0},
        })
        return sale_order_action

```

## File: models\event_registration.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.tools import float_is_zero


class EventRegistration(models.Model):
    _inherit = 'event.registration'

    is_paid = fields.Boolean('Is Paid')
    # TDE FIXME: maybe add an onchange on sale_order_id
    sale_order_id = fields.Many2one('sale.order', string='Sales Order', ondelete='cascade', copy=False)
    sale_order_line_id = fields.Many2one('sale.order.line', string='Sales Order Line', ondelete='cascade', copy=False)
    payment_status = fields.Selection(string="Payment Status", selection=[
            ('to_pay', 'Not Paid'),
            ('paid', 'Paid'),
            ('free', 'Free'),
        ], compute="_compute_payment_status", compute_sudo=True)
    utm_campaign_id = fields.Many2one(compute='_compute_utm_campaign_id', readonly=False, store=True)
    utm_source_id = fields.Many2one(compute='_compute_utm_source_id', readonly=False, store=True)
    utm_medium_id = fields.Many2one(compute='_compute_utm_medium_id', readonly=False, store=True)

    @api.depends('is_paid', 'sale_order_id.currency_id', 'sale_order_line_id.price_total')
    def _compute_payment_status(self):
        for record in self:
            so = record.sale_order_id
            so_line = record.sale_order_line_id
            if not so or float_is_zero(so_line.price_total, precision_digits=so.currency_id.rounding):
                record.payment_status = 'free'
            elif record.is_paid:
                record.payment_status = 'paid'
            else:
                record.payment_status = 'to_pay'

    @api.depends('sale_order_id')
    def _compute_utm_campaign_id(self):
        for registration in self:
            if registration.sale_order_id.campaign_id:
                registration.utm_campaign_id = registration.sale_order_id.campaign_id
            elif not registration.utm_campaign_id:
                registration.utm_campaign_id = False

    @api.depends('sale_order_id')
    def _compute_utm_source_id(self):
        for registration in self:
            if registration.sale_order_id.source_id:
                registration.utm_source_id = registration.sale_order_id.source_id
            elif not registration.utm_source_id:
                registration.utm_source_id = False

    @api.depends('sale_order_id')
    def _compute_utm_medium_id(self):
        for registration in self:
            if registration.sale_order_id.medium_id:
                registration.utm_medium_id = registration.sale_order_id.medium_id
            elif not registration.utm_medium_id:
                registration.utm_medium_id = False

    def action_view_sale_order(self):
        action = self.env["ir.actions.actions"]._for_xml_id("sale.action_orders")
        action['views'] = [(False, 'form')]
        action['res_id'] = self.sale_order_id.id
        return action

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            if vals.get('sale_order_line_id'):
                so_line_vals = self._synchronize_so_line_values(
                    self.env['sale.order.line'].browse(vals['sale_order_line_id'])
                )
                vals.update(so_line_vals)
        registrations = super(EventRegistration, self).create(vals_list)
        for registration in registrations:
            if registration.sale_order_id:
                registration.message_post_with_view(
                    'mail.message_origin_link',
                    values={'self': registration, 'origin': registration.sale_order_id},
                    subtype_id=self.env.ref('mail.mt_note').id)
        return registrations

    def write(self, vals):
        if vals.get('sale_order_line_id'):
            so_line_vals = self._synchronize_so_line_values(
                self.env['sale.order.line'].browse(vals['sale_order_line_id'])
            )
            vals.update(so_line_vals)

        if vals.get('event_ticket_id'):
            self.filtered(
                lambda registration: registration.event_ticket_id and registration.event_ticket_id.id != vals['event_ticket_id']
            )._sale_order_ticket_type_change_notify(self.env['event.event.ticket'].browse(vals['event_ticket_id']))

        return super(EventRegistration, self).write(vals)

    def _synchronize_so_line_values(self, so_line):
        if so_line:
            return {
                'partner_id': so_line.order_id.partner_id.id,
                'event_id': so_line.event_id.id,
                'event_ticket_id': so_line.event_ticket_id.id,
                'sale_order_id': so_line.order_id.id,
                'sale_order_line_id': so_line.id,
            }
        return {}

    def _sale_order_ticket_type_change_notify(self, new_event_ticket):
        fallback_user_id = self.env.user.id if not self.env.user._is_public() else self.env.ref("base.user_admin").id
        for registration in self:
            render_context = {
                'registration': registration,
                'old_ticket_name': registration.event_ticket_id.name,
                'new_ticket_name': new_event_ticket.name
            }
            user_id = registration.event_id.user_id.id or registration.sale_order_id.user_id.id or fallback_user_id
            registration.sale_order_id._activity_schedule_with_view(
                'mail.mail_activity_data_warning',
                user_id=user_id,
                views_or_xmlid='event_sale.event_ticket_id_change_exception',
                render_context=render_context)

    def _action_set_paid(self):
        self.write({'is_paid': True})

    def _get_registration_summary(self):
        res = super(EventRegistration, self)._get_registration_summary()
        res.update({
            'payment_status': self.payment_status,
            'payment_status_value': dict(self._fields['payment_status']._description_selection(self.env))[self.payment_status],
            'has_to_pay': self.payment_status == 'to_pay',
        })
        return res

```

## File: models\event_ticket.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from odoo import api, fields, models

_logger = logging.getLogger(__name__)


class EventTemplateTicket(models.Model):
    _inherit = 'event.type.ticket'

    def _default_product_id(self):
        return self.env.ref('event_sale.product_product_event', raise_if_not_found=False)

    description = fields.Text(compute='_compute_description', readonly=False, store=True)
    # product
    product_id = fields.Many2one(
        'product.product', string='Product', required=True,
        domain=[("event_ok", "=", True)], default=_default_product_id)
    price = fields.Float(
        string='Price', compute='_compute_price',
        digits='Product Price', readonly=False, store=True)
    price_reduce = fields.Float(
        string="Price Reduce", compute="_compute_price_reduce",
        compute_sudo=True, digits='Product Price')

    @api.depends('product_id')
    def _compute_price(self):
        for ticket in self:
            if ticket.product_id and ticket.product_id.lst_price:
                ticket.price = ticket.product_id.lst_price or 0
            elif not ticket.price:
                ticket.price = 0

    @api.depends('product_id')
    def _compute_description(self):
        for ticket in self:
            if ticket.product_id and ticket.product_id.description_sale:
                ticket.description = ticket.product_id.description_sale
            # initialize, i.e for embedded tree views
            if not ticket.description:
                ticket.description = False

    @api.depends('product_id', 'price')
    def _compute_price_reduce(self):
        for ticket in self:
            product = ticket.product_id
            discount = (product.lst_price - product.price) / product.lst_price if product.lst_price else 0.0
            ticket.price_reduce = (1.0 - discount) * ticket.price

    def _init_column(self, column_name):
        if column_name != "product_id":
            return super(EventTemplateTicket, self)._init_column(column_name)

        # fetch void columns
        self.env.cr.execute("SELECT id FROM %s WHERE product_id IS NULL" % self._table)
        ticket_type_ids = self.env.cr.fetchall()
        if not ticket_type_ids:
            return

        # update existing columns
        _logger.debug("Table '%s': setting default value of new column %s to unique values for each row",
                      self._table, column_name)
        default_event_product = self.env.ref('event_sale.product_product_event', raise_if_not_found=False)
        if default_event_product:
            product_id = default_event_product.id
        else:
            product_id = self.env['product.product'].create({
                'name': 'Generic Registration Product',
                'list_price': 0,
                'standard_price': 0,
                'type': 'service',
            }).id
            self.env['ir.model.data'].create({
                'name': 'product_product_event',
                'module': 'event_sale',
                'model': 'product.product',
                'res_id': product_id,
            })
        self.env.cr._obj.execute(
            f'UPDATE {self._table} SET product_id = %s WHERE id IN %s;',
            (product_id, tuple(ticket_type_ids))
        )

    @api.model
    def _get_event_ticket_fields_whitelist(self):
        """ Add sale specific fields to copy from template to ticket """
        return super(EventTemplateTicket, self)._get_event_ticket_fields_whitelist() + ['product_id', 'price']


class EventTicket(models.Model):
    _inherit = 'event.event.ticket'
    _order = "event_id, price"

    # product
    price_reduce_taxinc = fields.Float(
        string='Price Reduce Tax inc', compute='_compute_price_reduce_taxinc',
        compute_sudo=True)

    def _compute_price_reduce_taxinc(self):
        for event in self:
            # sudo necessary here since the field is most probably accessed through the website
            tax_ids = event.product_id.taxes_id.filtered(lambda r: r.company_id == event.event_id.company_id)
            taxes = tax_ids.compute_all(event.price_reduce, event.event_id.company_id.currency_id, 1.0, product=event.product_id)
            event.price_reduce_taxinc = taxes['total_included']

    @api.depends('product_id.active')
    def _compute_sale_available(self):
        inactive_product_tickets = self.filtered(lambda ticket: not ticket.product_id.active)
        for ticket in inactive_product_tickets:
            ticket.sale_available = False
        super(EventTicket, self - inactive_product_tickets)._compute_sale_available()

    def _get_ticket_multiline_description(self):
        """ If people set a description on their product it has more priority
        than the ticket name itself for the SO description. """
        self.ensure_one()
        if self.product_id.description_sale:
            return '%s\n%s' % (self.product_id.description_sale, self.event_id.display_name)
        return super(EventTicket, self)._get_ticket_multiline_description()

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

from odoo import api, fields, models, _


class SaleOrder(models.Model):
    _inherit = "sale.order"

    attendee_count = fields.Integer('Attendee Count', compute='_compute_attendee_count')

    def write(self, vals):
        """ Synchronize partner from SO to registrations. This is done notably
        in website_sale controller shop/address that updates customer, but not
        only. """
        result = super(SaleOrder, self).write(vals)
        if vals.get('partner_id'):
            registrations_toupdate = self.env['event.registration'].search([('sale_order_id', 'in', self.ids)])
            registrations_toupdate.write({'partner_id': vals['partner_id']})
        return result

    def action_confirm(self):
        res = super(SaleOrder, self).action_confirm()
        for so in self:
            # confirm registration if it was free (otherwise it will be confirmed once invoice fully paid)
            so.order_line._update_registrations(confirm=so.amount_total == 0, cancel_to_draft=False)
            if any(line.event_id for line in so.order_line):
                return self.env['ir.actions.act_window'] \
                    .with_context(default_sale_order_id=so.id) \
                    ._for_xml_id('event_sale.action_sale_order_event_registration')
        return res

    def _action_cancel(self):
        self.order_line._cancel_associated_registrations()
        return super()._action_cancel()

    def action_view_attendee_list(self):
        action = self.env["ir.actions.actions"]._for_xml_id("event.event_registration_action_tree")
        action['domain'] = [('sale_order_id', 'in', self.ids)]
        return action

    def _compute_attendee_count(self):
        sale_orders_data = self.env['event.registration'].read_group(
            [('sale_order_id', 'in', self.ids),
             ('state', '!=', 'cancel')],
            ['sale_order_id'], ['sale_order_id']
        )
        attendee_count_data = {
            sale_order_data['sale_order_id'][0]:
            sale_order_data['sale_order_id_count']
            for sale_order_data in sale_orders_data
        }
        for sale_order in self:
            sale_order.attendee_count = attendee_count_data.get(sale_order.id, 0)

    def unlink(self):
        self.mapped('order_line')._unlink_associated_registrations()
        return super(SaleOrder, self).unlink()


class SaleOrderLine(models.Model):

    _inherit = 'sale.order.line'

    event_id = fields.Many2one(
        'event.event', string='Event',
        help="Choose an event and it will automatically create a registration for this event.")
    event_ticket_id = fields.Many2one(
        'event.event.ticket', string='Event Ticket',
        help="Choose an event ticket and it will automatically create a registration for this event ticket.")
    event_ok = fields.Boolean(related='product_id.event_ok', readonly=True)

    @api.depends('state', 'event_id')
    def _compute_product_uom_readonly(self):
        event_lines = self.filtered(lambda line: line.event_id)
        event_lines.update({'product_uom_readonly': True})
        super(SaleOrderLine, self - event_lines)._compute_product_uom_readonly()

    def _update_registrations(self, confirm=True, cancel_to_draft=False, registration_data=None, mark_as_paid=False):
        """ Create or update registrations linked to a sales order line. A sale
        order line has a product_uom_qty attribute that will be the number of
        registrations linked to this line. This method update existing registrations
        and create new one for missing one. """
        RegistrationSudo = self.env['event.registration'].sudo()
        registrations = RegistrationSudo.search([('sale_order_line_id', 'in', self.ids)])
        registrations_vals = []
        for so_line in self.filtered('event_id'):
            existing_registrations = registrations.filtered(lambda self: self.sale_order_line_id.id == so_line.id)
            if confirm:
                existing_registrations.filtered(lambda self: self.state not in ['open', 'cancel']).action_confirm()
            if mark_as_paid:
                existing_registrations.filtered(lambda self: not self.is_paid)._action_set_paid()
            if cancel_to_draft:
                existing_registrations.filtered(lambda self: self.state == 'cancel').action_set_draft()

            for count in range(int(so_line.product_uom_qty) - len(existing_registrations)):
                values = {
                    'sale_order_line_id': so_line.id,
                    'sale_order_id': so_line.order_id.id
                }
                # TDE CHECK: auto confirmation
                if registration_data:
                    values.update(registration_data.pop())
                registrations_vals.append(values)

        if registrations_vals:
            RegistrationSudo.create(registrations_vals)
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
        return super(SaleOrderLine, self).unlink()

    def _cancel_associated_registrations(self):
        self.env['event.registration'].search([('sale_order_line_id', 'in', self.ids)]).action_cancel()

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

            return ticket._get_ticket_multiline_description() + self._get_sale_order_line_multiline_description_variants()
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

from . import account_move
from . import event_event
from . import event_registration
from . import event_ticket
from . import sale_order
from . import product

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
access_product_template_event_manager,product.template.event.manager,product.model_product_template,event.group_event_manager,1,1,1,1
access_product_product_event_manager,product.product.event.manager,product.model_product_product,event.group_event_manager,1,1,1,1
access_registration_editor,access.registration.editor,model_registration_editor,sales_team.group_sale_salesman,1,1,1,0
access_registration_editor_line,access.registration.editor.line,model_registration_editor_line,sales_team.group_sale_salesman,1,1,1,1
access_event_event_configurator,access.event.event.configurator,model_event_event_configurator,sales_team.group_sale_salesman,1,1,1,0

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
          if (stopPropagation || productId === undefined) {
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
            if (Array.isArray(result) && result.length && result[0].event_ok) {
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
<template id="qunit_suite" inherit_id="web.qunit_suite_tests" name="event_sale_tests">
    <xpath expr="." position="inside">
        <script type="text/javascript" src="/event_sale/static/tests/event_configurator.test.js"></script>
    </xpath>
</template>
</odoo>

```

## File: views\event_registration_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>

    <record id="view_event_registration_ticket_tree" model="ir.ui.view">
        <field name="name">event.registration.tree.inherit</field>
        <field name="model">event.registration</field>
        <field name="inherit_id" ref="event.view_event_registration_tree" />
        <field name="arch" type="xml">
            <field name="event_id" position="after">
                <field name="sale_order_id" optional="hide"/>
            </field>
        </field>
    </record>

    <record id="event_registration_view_kanban" model="ir.ui.view">
        <field name="name">event.registration.kanban.inherit.event.sale</field>
        <field name="model">event.registration</field>
        <field name="inherit_id" ref="event.event_registration_view_kanban"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='event_attendees_kanban_icons_desktop']" position="inside">
                <div class="mt-auto pt-2">
                    <field name="payment_status"/>
                </div>
            </xpath>
            <xpath expr="//div[@id='event_ticket_id']" position="before">
                <div class="d-md-none" >
                    <field name="payment_status"/>
                </div>
            </xpath>
        </field>
    </record>

    <record id="event_registration_ticket_view_form" model="ir.ui.view">
        <field name="name">event.registration.form.inherit</field>
        <field name="model">event.registration</field>
        <field name="inherit_id" ref="event.view_event_registration_form" />
        <field name="arch" type="xml">
            <xpath expr="//div[@name='button_box']" position="inside">
                <button name="action_view_sale_order" type="object"
                        class="oe_stat_button" icon="fa-usd" string="Sale Order">
                </button>
            </xpath>
            <xpath expr="//group" position="before">
                <field name="is_paid" invisible="1"/>
                <widget name="web_ribbon" title="Paid" bg_color="bg-success" attrs="{'invisible': [('is_paid', '=', False)]}"/>
            </xpath>
            <group name="utm_link" position="before">
                <group string="Transaction" groups="base.group_no_one">
                    <field name="sale_order_id"/>
                    <field name="sale_order_line_id" readonly="1" attrs="{'invisible': [('sale_order_id', '=', False)]}"/>
                </group>
            </group>
        </field>
    </record>

</data></odoo>

```

## File: views\event_ticket_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>

    <!-- EVENT.TYPE.TICKET -->
    <record id="event_type_ticket_view_tree_from_type" model="ir.ui.view">
        <field name="name">event.type.ticket.view.tree.inherit.sale</field>
        <field name="model">event.type.ticket</field>
        <field name="inherit_id" ref="event.event_type_ticket_view_tree_from_type"/>
        <field name="arch" type="xml">
            <field name="name" position="after">
                <field name="product_id" context="{'default_event_ok': 1, 'default_type': 'service'}"/>
            </field>
            <field name="description" position="after">
                <field name="price"/>
            </field>
        </field>
    </record>

    <record id="event_type_ticket_view_form_from_type" model="ir.ui.view">
        <field name="name">event.type.ticket.view.form.inherit.sale</field>
        <field name="model">event.type.ticket</field>
        <field name="inherit_id" ref="event.event_type_ticket_view_form_from_type"/>
        <field name="arch" type="xml">
            <field name="name" position="after">
                <field name="product_id" context="{'default_event_ok': 1, 'default_type': 'service'}"/>
            </field>
            <field name="description" position="after">
                <field name="price"/>
            </field>
        </field>
    </record>

    <!-- EVENT.TICKET -->
    <record id="event_event_ticket_view_tree_from_event" model="ir.ui.view">
        <field name="name">event.event.ticket.view.tree.from.event.inherit.sale</field>
        <field name="model">event.event.ticket</field>
        <field name="inherit_id" ref="event.event_event_ticket_view_tree_from_event"/>
        <field name="arch" type="xml">
            <field name="start_sale_date" position="attributes">
                <attribute name="string">Sales Start</attribute>
            </field>
            <field name="end_sale_date" position="attributes">
                <attribute name="string">Sales End</attribute>
            </field>
            <field name="name" position="after">
                <field name="product_id" context="{'default_event_ok': 1, 'default_type': 'service'}"/>
            </field>
            <field name="description" position="after">
                <field name="price"/>
            </field>
        </field>
    </record>

    <record id="event_event_ticket_view_form_from_event" model="ir.ui.view">
        <field name="name">event.event.ticket.view.form.from.event.inherit.sale</field>
        <field name="model">event.event.ticket</field>
        <field name="inherit_id" ref="event.event_event_ticket_view_form_from_event"/>
        <field name="arch" type="xml">
            <field name="name" position="after">
                <field name="product_id" context="{'default_event_ok':1, 'default_type': 'service'}"/>
            </field>
            <field name="description" position="after">
                <field name="price"/>
            </field>
        </field>
    </record>

    <record id="event_event_ticket_view_kanban_from_event" model="ir.ui.view">
        <field name="name">event.event.ticket.view.kanban.from.event</field>
        <field name="model">event.event.ticket</field>
        <field name="inherit_id" ref="event.event_event_ticket_view_kanban_from_event"/>
        <field name="arch" type="xml">
            <field name="name" position="after">
                <field name="product_id"/>
                <field name="price"/>
            </field>
            <xpath expr="//div[hasclass('col-8')]" position="after">
                <div class="col-4 text-right"><strong> <t t-esc="record.price.value"/></strong></div>
            </xpath>
            <xpath expr="//div[hasclass('row')]" position="after">
                <div t-esc="record.product_id.value"/>
            </xpath>
        </field>
    </record>

    <record id="event_event_ticket_form_view" model="ir.ui.view">
        <field name="name">event.event.ticket.view.form.inherit.sale</field>
        <field name="model">event.event.ticket</field>
        <field name="inherit_id" ref="event.event_event_ticket_form_view"/>
        <field name="arch" type="xml">
            <field name="end_sale_date" position="after">
                <field name="price"/>
                <field name="price_reduce" groups="base.group_no_one"/>
            </field>
            <field name="seats_used" position="after">
                <field name="product_id"/>
            </field>
        </field>
    </record>
</data></odoo>


```

## File: views\event_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_event_form_inherit_ticket" model="ir.ui.view">
        <field name="name">event.form.inherit</field>
        <field name="model">event.event</field>
        <field name="inherit_id" ref="event.view_event_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[hasclass('oe_button_box')]" position="inside">
                <field name="currency_id" invisible="1"/>
                <button name="action_view_linked_orders"
                        type="object" class="oe_stat_button" icon="fa-dollar"
                        groups="sales_team.group_sale_salesman"
                        help="Total sales for this event"
                        attrs="{'invisible': ['|', ('sale_price_subtotal', '=', 0), ('sale_price_subtotal', '=', False)]}">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_value">
                            <field string="Sales" name="sale_price_subtotal"
                                widget="monetary" options="{'currency_field': 'currency_id'}"/>
                            </span>
                        <span class="o_stat_text">Sales</span>
                    </div>
                </button>
            </xpath>
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
                <group name="sale" position="inside">
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
            <xpath expr="//button[@name='preview_sale_order']" position="before">
                <button name="action_view_attendee_list" type="object"
                        class="oe_stat_button" icon="fa-users" attrs="{'invisible': [('attendee_count', '=', 0)]}">
                    <field name="attendee_count" widget="statinfo" string="Attendees"/>
                </button>
            </xpath>
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
                        '|', ('seats_limited', '=', False), ('seats_available', '>', 0), '|', ('company_id', '=', False), ('company_id', '=', parent.company_id)
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
                            '|', ('seats_limited', '=', False), ('seats_available', '>', 0)
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
                    'name': so_line.order_partner_id.name,
                    'email': so_line.order_partner_id.email,
                    'phone': so_line.order_partner_id.phone,
                    'mobile': so_line.order_partner_id.mobile,
                }])
        res['event_registration_ids'] = attendee_list
        res = self._convert_to_write(res)
        return res

    def action_make_registration(self):
        self.ensure_one()
        registrations_to_create = []
        for registration_line in self.event_registration_ids:
            values = registration_line.get_registration_data()
            if registration_line.registration_id:
                registration_line.registration_id.write(values)
            else:
                registrations_to_create.append(values)

        self.env['event.registration'].create(registrations_to_create)
        self.sale_order_id.order_line._update_registrations(confirm=self.sale_order_id.amount_total == 0)

        return {'type': 'ir.actions.act_window_close'}


class RegistrationEditorLine(models.TransientModel):
    """Event Registration"""
    _name = "registration.editor.line"
    _description = 'Edit Attendee Line on Sales Confirmation'
    _order = "id desc"

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
                            <field name="sale_order_line_id" invisible="1"/>
                        </tree>
                    </field>
                    <footer>
                        <button string="Confirm" name="action_make_registration" type="object" class="btn-primary"/>
                        <button string="Cancel" class="btn-secondary" special="cancel"/>
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

