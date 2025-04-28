# Odoo Module: event_sale

Category: Marketing/Events

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import report
from . import wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-

{
    'name': 'Events Sales',
    'version': '1.3',
    'category': 'Marketing/Events',
    'website': 'https://www.odoo.com/app/events',
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
        'views/event_ticket_views.xml',
        'views/event_registration_views.xml',
        'views/event_views.xml',
        'views/sale_order_views.xml',
        'data/event_sale_data.xml',
        'data/mail_templates.xml',
        'report/event_sale_report_views.xml',
        'security/ir.model.access.csv',
        'security/ir_rule.xml',
        'security/event_security.xml',
        'wizard/event_edit_registration.xml',
        'wizard/event_configurator_views.xml',
    ],
    'demo': [
        'data/event_sale_demo.xml',
        'data/event_demo.xml',  # needs event_sale_demo
        'data/event_registration_demo.xml',  # needs event_sale_demo
    ],
    'installable': True,
    'auto_install': True,
    'assets': {
        'web.assets_backend': [
            'event_sale/static/src/**/*',
        ],
        'web.assets_tests': [
            'event_sale/static/tests/tours/**/*',
        ],
    },
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
        <field name="product_id" ref="product_product_event_standard"/>
        <field name="price">1000.0</field>
    </record>
    <record id="event.event_0_ticket_2" model="event.event.ticket">
        <field name="product_id" ref="product_product_event_vip"/>
        <field name="price">1500.0</field>
    </record>

    <record id="event.event_2_ticket_1" model="event.event.ticket">
        <field name="product_id" ref="product_product_event_standard"/>
        <field name="price">1000.0</field>
    </record>
    <record id="event.event_2_ticket_2" model="event.event.ticket">
        <field name="product_id" ref="product_product_event_vip"/>
        <field name="price">1500.0</field>
    </record>

    <record id="event.event_4_ticket_0" model="event.event.ticket">
        <field name="product_id" ref="product_product_event_standard"/>
        <field name="price">99.0</field>
    </record>

    <record id="event.event_7_ticket_1" model="event.event.ticket">
        <field name="product_id" ref="product_product_event_standard"/>
        <field name="price">0.0</field>
    </record>
    <record id="event.event_7_ticket_2" model="event.event.ticket">
        <field name="product_id" ref="product_product_event_vip"/>
        <field name="price">0.0</field>
    </record>

</odoo>

```

## File: data\event_registration_demo.xml

```xml
<?xml version="1.0"?>
<odoo><data>
    <!-- Design fair -->
    <record id="event.event_registration_0_0" model="event.registration">
        <field name="sale_order_id" ref="event_sale.event_0_sale_order_0"/>
        <field name="sale_order_line_id" ref="event_sale.event_0_sale_order_0_line_0"/>
    </record>
    <record id="event.event_registration_0_1" model="event.registration">
        <field name="sale_order_id" ref="event_sale.event_0_sale_order_0"/>
        <field name="sale_order_line_id" ref="event_sale.event_0_sale_order_0_line_0"/>
    </record>
    <record id="event.event_registration_0_2" model="event.registration">
        <field name="sale_order_id" ref="event_sale.event_0_sale_order_0"/>
        <field name="sale_order_line_id" ref="event_sale.event_0_sale_order_0_line_1"/>
    </record>

    <!-- Conference for architects -->
    <record id="event.event_registration_2_0" model="event.registration">
        <field name="sale_order_id" ref="event_sale.event_2_sale_order_0"/>
        <field name="sale_order_line_id" ref="event_sale.event_2_sale_order_0_line_0"/>
    </record>
    <record id="event.event_registration_2_1" model="event.registration">
        <field name="sale_order_id" ref="event_sale.event_2_sale_order_0"/>
        <field name="sale_order_line_id" ref="event_sale.event_2_sale_order_0_line_0"/>
    </record>
    <record id="event.event_registration_2_2" model="event.registration">
        <field name="sale_order_id" ref="event_sale.event_2_sale_order_0"/>
        <field name="sale_order_line_id" ref="event_sale.event_2_sale_order_0_line_1"/>
    </record>
    <record id="event.event_registration_2_3" model="event.registration">
        <field name="sale_order_id" ref="event_sale.event_2_sale_order_1"/>
        <field name="sale_order_line_id" ref="event_sale.event_2_sale_order_1_line_0"/>
    </record>
    <record id="event.event_registration_2_4" model="event.registration">
        <field name="sale_order_id" ref="event_sale.event_2_sale_order_1"/>
        <field name="sale_order_line_id" ref="event_sale.event_2_sale_order_1_line_0"/>
    </record>

    <!-- Business Workshop -->
    <record id="event.event_registration_4_0" model="event.registration">
        <field name="sale_order_id" ref="event_sale.event_4_sale_order_0"/>
        <field name="sale_order_line_id" ref="event_sale.event_4_sale_order_0_line_0"/>
    </record>
    <record id="event.event_registration_4_1" model="event.registration">
        <field name="sale_order_id" ref="event_sale.event_4_sale_order_1"/>
        <field name="sale_order_line_id" ref="event_sale.event_4_sale_order_1_line_0"/>
    </record>
    <record id="event.event_registration_4_2" model="event.registration">
        <field name="sale_order_id" ref="event_sale.event_4_sale_order_2"/>
        <field name="sale_order_line_id" ref="event_sale.event_4_sale_order_2_line_0"/>
    </record>

    <!-- OpenWood Collection Online Reveal: Gemini (all) -->
    <record id="event.event_registration_7_0" model="event.registration">
        <field name="sale_order_id" ref="event_sale.event_7_sale_order_0"/>
        <field name="sale_order_line_id" ref="event_sale.event_7_sale_order_0_line_0"/>
    </record>
    <record id="event.event_registration_7_1" model="event.registration">
        <field name="sale_order_id" ref="event_sale.event_7_sale_order_1"/>
        <field name="sale_order_line_id" ref="event_sale.event_7_sale_order_1_line_0"/>
    </record>
    <record id="event.event_registration_7_2" model="event.registration">
        <field name="sale_order_id" ref="event_sale.event_7_sale_order_0"/>
        <field name="sale_order_line_id" ref="event_sale.event_7_sale_order_0_line_1"/>
    </record>
    <record id="event.event_registration_7_3" model="event.registration">
        <field name="sale_order_id" ref="event_sale.event_7_sale_order_1"/>
        <field name="sale_order_line_id" ref="event_sale.event_7_sale_order_1_line_1"/>
    </record>
</data></odoo>

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
            <field name="list_price">30.0</field>
            <field name="standard_price">10.0</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="name">Event Registration</field>
            <field name="description_sale" eval="False"/>
            <field name="invoice_policy">order</field>
            <field name="categ_id" ref="event_sale.product_category_events"/>
            <field name="detailed_type">event</field>
        </record>
    </data>
</odoo>

```

## File: data\event_sale_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>
    <!-- ****** Products ****** -->
    <record id="product_product_event_standard" model="product.product">
        <field name="list_price">30.0</field>
        <field name="standard_price">10.0</field>
        <field name="uom_id" ref="uom.product_uom_unit"/>
        <field name="uom_po_id" ref="uom.product_uom_unit"/>
        <field name="name">Event Registration - Standard</field>
        <field name="description_sale" eval="False"/>
        <field name="invoice_policy">order</field>
        <field name="categ_id" ref="event_sale.product_category_events"/>
        <field name="detailed_type">event</field>
    </record>

    <record id="product_product_event_vip" model="product.product">
        <field name="list_price">100.0</field>
        <field name="standard_price">50.0</field>
        <field name="uom_id" ref="uom.product_uom_unit"/>
        <field name="uom_po_id" ref="uom.product_uom_unit"/>
        <field name="name">Event Registration - VIP</field>
        <field name="description_sale" eval="False"/>
        <field name="invoice_policy">order</field>
        <field name="categ_id" ref="event_sale.product_category_events"/>
        <field name="detailed_type">event</field>
    </record>

    <!-- ****** Registrations ****** -->
    <!-- Design fair -->
    <record id="event_0_sale_order_0" model="sale.order">
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_id" ref="base.res_partner_address_1"/>
        <field name="date_order" eval="DateTime.now() - relativedelta(days=2)"/>
        <field name="state">sale</field>
    </record>
    <record id="event_0_sale_order_0_line_0" model="sale.order.line">
        <field name="order_id" ref="event_0_sale_order_0"/>
        <field name="name">Event Registration - Standard</field>
        <field name="product_id" ref="event_sale.product_product_event_standard"/>
        <field name="price_unit">1000</field>
        <field name="product_uom_qty">2</field>
        <field name="event_id" ref="event.event_0"/>
        <field name="event_ticket_id" ref="event.event_0_ticket_1"/>
    </record>
    <record id="event_0_sale_order_0_line_1" model="sale.order.line">
        <field name="order_id" ref="event_0_sale_order_0"/>
        <field name="name">Event Registration</field>
        <field name="product_id" ref="event_sale.product_product_event"/>
        <field name="price_unit">0</field>
        <field name="product_uom_qty">1</field>
        <field name="event_id" ref="event.event_0"/>
        <field name="event_ticket_id" ref="event.event_0_ticket_0"/>
    </record>

    <!-- Conference for architects -->
    <record id="event_2_sale_order_0" model="sale.order">
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_id" ref="base.res_partner_address_2"/>
        <field name="date_order" eval="DateTime.now() - relativedelta(days=0.5)"/>
        <field name="state">sale</field>
    </record>
    <record id="event_2_sale_order_0_line_0" model="sale.order.line">
        <field name="order_id" ref="event_2_sale_order_0"/>
        <field name="name">Event Registration - Standard</field>
        <field name="product_id" ref="event_sale.product_product_event_standard"/>
        <field name="price_unit">1000</field>
        <field name="product_uom_qty">2</field>
        <field name="event_id" ref="event.event_2"/>
        <field name="event_ticket_id" ref="event.event_2_ticket_1"/>
    </record>
    <record id="event_2_sale_order_0_line_1" model="sale.order.line">
        <field name="order_id" ref="event_2_sale_order_0"/>
        <field name="name">Event Registration - VIP</field>
        <field name="product_id" ref="event_sale.product_product_event_vip"/>
        <field name="price_unit">1500</field>
        <field name="product_uom_qty">1</field>
        <field name="event_id" ref="event.event_2"/>
        <field name="event_ticket_id" ref="event.event_2_ticket_2"/>
    </record>

    <record id="event_2_sale_order_1" model="sale.order">
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_id" ref="base.res_partner_address_3"/>
        <field name="date_order" eval="DateTime.now() - relativedelta(days=1)"/>
        <field name="state">sale</field>
    </record>
    <record id="event_2_sale_order_1_line_0" model="sale.order.line">
        <field name="order_id" ref="event_2_sale_order_1"/>
        <field name="name">Event Registration - Standard</field>
        <field name="product_id" ref="event_sale.product_product_event_standard"/>
        <field name="price_unit">1000</field>
        <field name="product_uom_qty">2</field>
        <field name="event_id" ref="event.event_2"/>
        <field name="event_ticket_id" ref="event.event_2_ticket_1"/>
    </record>

    <!-- Business Workshop -->
    <record id="event_4_sale_order_0" model="sale.order">
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_id" ref="base.res_partner_address_7"/>
        <field name="date_order" eval="DateTime.now() - relativedelta(days=8)"/>
        <field name="state">sale</field>
    </record>
    <record id="event_4_sale_order_0_line_0" model="sale.order.line">
        <field name="order_id" ref="event_4_sale_order_0"/>
        <field name="name">Event Registration - Standard</field>
        <field name="product_id" ref="event_sale.product_product_event_standard"/>
        <field name="price_unit">499</field>
        <field name="product_uom_qty">1</field>
        <field name="event_id" ref="event.event_4"/>
        <field name="event_ticket_id" ref="event.event_4_ticket_0"/>
    </record>

    <record id="event_4_sale_order_1" model="sale.order">
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_id" ref="base.res_partner_address_13"/>
        <field name="date_order" eval="DateTime.now() - relativedelta(days=7)"/>
        <field name="state">sale</field>
    </record>
    <record id="event_4_sale_order_1_line_0" model="sale.order.line">
        <field name="order_id" ref="event_4_sale_order_1"/>
        <field name="name">Event Registration - Standard</field>
        <field name="product_id" ref="event_sale.product_product_event_standard"/>
        <field name="price_unit">499</field>
        <field name="product_uom_qty">1</field>
        <field name="event_id" ref="event.event_4"/>
        <field name="event_ticket_id" ref="event.event_4_ticket_0"/>
    </record>

    <record id="event_4_sale_order_2" model="sale.order">
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_id" ref="base.res_partner_address_14"/>
        <field name="date_order" eval="DateTime.now() - relativedelta(days=7)"/>
        <field name="state">sale</field>
    </record>
    <record id="event_4_sale_order_2_line_0" model="sale.order.line">
        <field name="order_id" ref="event_4_sale_order_2"/>
        <field name="name">Event Registration - Standard</field>
        <field name="product_id" ref="event_sale.product_product_event_standard"/>
        <field name="price_unit">499</field>
        <field name="product_uom_qty">1</field>
        <field name="event_id" ref="event.event_4"/>
        <field name="event_ticket_id" ref="event.event_4_ticket_0"/>
    </record>

    <!-- OpenWood Collection Online Reveal: Gemini (all) -->
    <record id="event_7_sale_order_0" model="sale.order">
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_id" ref="base.res_partner_address_5"/>
        <field name="state">sale</field>
    </record>
    <record id="event_7_sale_order_0_line_0" model="sale.order.line">
        <field name="order_id" ref="event_7_sale_order_0"/>
        <field name="name">Event Registration - Standard</field>
        <field name="product_id" ref="event_sale.product_product_event_standard"/>
        <field name="price_unit">0</field>
        <field name="product_uom_qty">1</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="event_ticket_id" ref="event.event_7_ticket_1"/>
    </record>
    <record id="event_7_sale_order_0_line_1" model="sale.order.line">
        <field name="order_id" ref="event_7_sale_order_0"/>
        <field name="name">Event Registration - VIP</field>
        <field name="product_id" ref="event_sale.product_product_event_vip"/>
        <field name="price_unit">0</field>
        <field name="product_uom_qty">1</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="event_ticket_id" ref="event.event_7_ticket_2"/>
    </record>

    <record id="event_7_sale_order_1" model="sale.order">
        <field name="user_id" ref="base.user_admin"/>
        <field name="partner_id" ref="base.res_partner_address_25"/>
        <field name="state">sale</field>
    </record>
    <record id="event_7_sale_order_1_line_0" model="sale.order.line">
        <field name="order_id" ref="event_7_sale_order_1"/>
        <field name="name">Event Registration - Standard</field>
        <field name="product_id" ref="event_sale.product_product_event_standard"/>
        <field name="price_unit">0</field>
        <field name="product_uom_qty">1</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="event_ticket_id" ref="event.event_7_ticket_1"/>
    </record>
    <record id="event_7_sale_order_1_line_1" model="sale.order.line">
        <field name="order_id" ref="event_7_sale_order_1"/>
        <field name="name">Event Registration - VIP</field>
        <field name="product_id" ref="event_sale.product_product_event_vip"/>
        <field name="price_unit">0</field>
        <field name="product_uom_qty">1</field>
        <field name="event_id" ref="event.event_7"/>
        <field name="event_ticket_id" ref="event.event_7_ticket_2"/>
    </record>
</data></odoo>

```

## File: data\mail_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="event_ticket_id_change_exception" name="Message: Alert on event ticket id change">
    <div>
        <p>
            <span>Registration modification for attendee:</span>
            <a href="#" data-oe-model="event.registration" t-att-data-oe-id="registration.id"><t t-out="registration.name"/></a>.
            <span>Manual actions may be needed.</span>
        </p>
        <div class="mt16">
            <p>Exception:</p>
            <ul>
                <li>
                    <a href="#" data-oe-model="event.registration" t-att-data-oe-id="registration.id"><t t-out="registration.name"/></a>:
                    <span>Ticket changed from <strong><t t-out="old_ticket_name"/></strong> to <strong><t t-out="new_ticket_name"/></strong></span>
                </li>
            </ul>
        </div>
    </div>
</template>

</odoo>

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
        event_subtotals = self.env['sale.order.line']._read_group(
            [('event_id', 'in', self.ids), ('price_subtotal', '!=', 0), ('state', '!=', 'cancel')],
            ['event_id', 'currency_id'],
            ['price_subtotal:sum'],
        )
        event_subtotals_mapping = dict.fromkeys(self._origin, 0)
        for event, currency, sum_price_subtotal in event_subtotals:
            event_subtotals_mapping[event] += event.currency_id._convert(
                sum_price_subtotal,
                currency,
                event.company_id or self.env.company,
                date_now,
            )

        for event in self:
            event.sale_price_subtotal = event_subtotals_mapping.get(event._origin, 0)

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

from odoo import api, fields, models
from odoo.tools import float_is_zero


class EventRegistration(models.Model):
    _inherit = 'event.registration'

    # TDE FIXME: maybe add an onchange on sale_order_id
    sale_order_id = fields.Many2one('sale.order', string='Sales Order', ondelete='cascade', copy=False)
    sale_order_line_id = fields.Many2one('sale.order.line', string='Sales Order Line', ondelete='cascade', copy=False)
    sale_status = fields.Selection(string="Sale Status", selection=[
            ('to_pay', 'Not Sold'),
            ('sold', 'Sold'),
            ('free', 'Free'),
        ], compute="_compute_registration_status", compute_sudo=True, store=True, precompute=True)
    state = fields.Selection(default=None, compute="_compute_registration_status", store=True, readonly=False, precompute=True)
    utm_campaign_id = fields.Many2one(compute='_compute_utm_campaign_id', readonly=False,
        store=True, ondelete="set null")
    utm_source_id = fields.Many2one(compute='_compute_utm_source_id', readonly=False,
        store=True, ondelete="set null")
    utm_medium_id = fields.Many2one(compute='_compute_utm_medium_id', readonly=False,
        store=True, ondelete="set null")

    @api.depends('sale_order_id.state', 'sale_order_id.currency_id', 'sale_order_line_id.price_total')
    def _compute_registration_status(self):
        for so_line, registrations in self.filtered('sale_order_line_id').grouped('sale_order_line_id').items():
            cancelled_so_registrations = registrations.filtered(lambda reg: reg.sale_order_id.state == 'cancel')
            cancelled_so_registrations.state = 'cancel'
            cancelled_registrations = cancelled_so_registrations | registrations.filtered(lambda reg: reg.state == 'cancel')
            if float_is_zero(so_line.price_total, precision_rounding=so_line.currency_id.rounding):
                registrations.sale_status = 'free'
                registrations.filtered(lambda reg: not reg.state or reg.state == 'draft').state = "open"
            else:
                sold_registrations = registrations.filtered(lambda reg: reg.sale_order_id.state == 'sale') - cancelled_registrations
                sold_registrations.sale_status = 'sold'
                (registrations - sold_registrations).sale_status = 'to_pay'
                sold_registrations.filtered(lambda reg: not reg.state or reg.state in {'draft', 'cancel'}).state = "open"
                (registrations - sold_registrations - cancelled_registrations).state = 'draft'

        # set default value to free and open if none was set yet
        for registration in self:
            if not registration.sale_status:
                registration.sale_status = 'free'
            if not registration.state:
                registration.state = 'open'

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
                registration.message_post_with_source(
                    'mail.message_origin_link',
                    render_values={'self': registration, 'origin': registration.sale_order_id},
                    subtype_xmlid='mail.mt_note',
                )
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
                # Avoid registering public users but respect the portal workflows
                'partner_id': False if self.env.user._is_public() and self.env.user.partner_id == so_line.order_id.partner_id else so_line.order_id.partner_id.id,
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

    def _get_registration_summary(self):
        res = super(EventRegistration, self)._get_registration_summary()
        res.update({
            'sale_status': self.sale_status,
            'sale_status_value': self.sale_status and dict(self._fields['sale_status']._description_selection(self.env))[self.sale_status],
            'has_to_pay': self.sale_status == 'to_pay',
        })
        return res

```

## File: models\event_ticket.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from odoo import api, fields, models
from odoo.addons.product.models.product_template import PRICE_CONTEXT_KEYS

_logger = logging.getLogger(__name__)


class EventTemplateTicket(models.Model):
    _inherit = 'event.type.ticket'
    _order = "sequence, price, name, id"

    def _default_product_id(self):
        return self.env.ref('event_sale.product_product_event', raise_if_not_found=False)

    description = fields.Text(compute='_compute_description', readonly=False, store=True)
    # product
    product_id = fields.Many2one(
        'product.product', string='Product', required=True,
        domain=[("detailed_type", "=", "event")], default=_default_product_id)
    currency_id = fields.Many2one(related="product_id.currency_id", string="Currency")
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

    # TODO clean this feature in master
    # Feature broken by design, depending on the hacky `_get_contextual_price` field on products
    # context_dependent, core part of the pricelist mess
    # This field usage should be restricted to the UX, and any use in effective
    # price computation should be replaced by clear calls to the pricelist API
    @api.depends_context(*PRICE_CONTEXT_KEYS)
    @api.depends('product_id', 'price')
    def _compute_price_reduce(self):
        for ticket in self:
            contextual_discount = ticket.product_id._get_contextual_discount()
            ticket.price_reduce = (1.0 - contextual_discount) * ticket.price

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
    _order = "event_id, sequence, price, name, id"

    # product
    price_reduce_taxinc = fields.Float(
        string='Price Reduce Tax inc', compute='_compute_price_reduce_taxinc',
        compute_sudo=True)
    price_incl = fields.Float(
        string='Price include', compute='_compute_price_incl',
        digits='Product Price', readonly=False, compute_sudo=True)

    def _compute_price_reduce_taxinc(self):
        for event in self:
            # sudo necessary here since the field is most probably accessed through the website
            tax_ids = event.product_id.taxes_id.filtered(lambda r: r.company_id == event.event_id.company_id)
            taxes = tax_ids.compute_all(event.price_reduce, event.event_id.company_id.currency_id, 1.0, product=event.product_id)
            event.price_reduce_taxinc = taxes['total_included']

    @api.depends('product_id', 'product_id.taxes_id', 'price')
    def _compute_price_incl(self):
        for event in self:
            if event.product_id and event.price:
                tax_ids = event.product_id.taxes_id.filtered(lambda r: r.company_id == event.event_id.company_id)
                taxes = tax_ids.compute_all(event.price, event.currency_id, 1.0, product=event.product_id)
                event.price_incl = taxes['total_included']
            else:
                event.price_incl = 0

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

    detailed_type = fields.Selection(selection_add=[
        ('event', 'Event Ticket'),
    ], ondelete={'event': 'set service'})

    @api.onchange('detailed_type')
    def _onchange_type_event(self):
        if self.detailed_type == 'event':
            self.invoice_policy = 'order'

    def _detailed_type_mapping(self):
        type_mapping = super()._detailed_type_mapping()
        type_mapping['event'] = 'service'
        return type_mapping


class Product(models.Model):
    _inherit = 'product.product'

    event_ticket_ids = fields.One2many('event.event.ticket', 'product_id', string='Event Tickets')

```

## File: models\sale_order.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _
from odoo.exceptions import ValidationError
from odoo.osv import expression


class SaleOrder(models.Model):
    _inherit = "sale.order"

    attendee_count = fields.Integer('Attendee Count', compute='_compute_attendee_count')

    def write(self, vals):
        """ Synchronize partner from SO to registrations. This is done notably
        in website_sale controller shop/address that updates customer, but not
        only. """
        result = super(SaleOrder, self).write(vals)
        if any(line.product_type == 'event' for line in self.order_line) and vals.get('partner_id'):
            registrations_toupdate = self.env['event.registration'].sudo().search([('sale_order_id', 'in', self.ids)])
            registrations_toupdate.write({'partner_id': vals['partner_id']})
        return result

    def action_confirm(self):
        unconfirmed_registrations = self.order_line.registration_ids.filtered(
            lambda reg: reg.state in ["draft", "cancel"]
        )
        res = super(SaleOrder, self).action_confirm()
        unconfirmed_registrations._update_mail_schedulers()

        for so in self:
            if not any(line.product_type == 'event' for line in so.order_line):
                continue
            so_lines_missing_events = so.order_line.filtered(lambda line: line.product_type == 'event' and not line.event_id)
            if so_lines_missing_events:
                so_lines_descriptions = "".join(f"\n- {so_line_description.name}" for so_line_description in so_lines_missing_events)
                raise ValidationError(_("Please make sure all your event related lines are configured before confirming this order:%s", so_lines_descriptions))
            # Initialize registrations
            so.order_line._init_registrations()
            if len(self) == 1:
                return self.env['ir.actions.act_window'].with_context(
                    default_sale_order_id=so.id
                )._for_xml_id('event_sale.action_sale_order_event_registration')
        return res

    def action_view_attendee_list(self):
        action = self.env["ir.actions.actions"]._for_xml_id("event.event_registration_action_tree")
        action['domain'] = [('sale_order_id', 'in', self.ids)]
        return action

    def _compute_attendee_count(self):
        sale_orders_data = self.env['event.registration']._read_group(
            [('sale_order_id', 'in', self.ids),
             ('state', '!=', 'cancel')],
            ['sale_order_id'], ['__count'],
        )
        attendee_count_data = {
            sale_order.id: count for sale_order, count in sale_orders_data
        }
        for sale_order in self:
            sale_order.attendee_count = attendee_count_data.get(sale_order.id, 0)

    def _get_product_catalog_domain(self):
        """Override of `_get_product_catalog_domain` to extend the domain.

        :returns: A list of tuples that represents a domain.
        :rtype: list
        """
        domain = super()._get_product_catalog_domain()
        return expression.AND([domain, [('detailed_type', '!=', 'event')]])

```

## File: models\sale_order_line.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class SaleOrderLine(models.Model):
    _inherit = 'sale.order.line'

    event_id = fields.Many2one(
        'event.event', string='Event',
        compute="_compute_event_id", store=True, readonly=False, precompute=True,
        help="Choose an event and it will automatically create a registration for this event.")
    event_ticket_id = fields.Many2one(
        'event.event.ticket', string='Event Ticket',
        compute="_compute_event_ticket_id", store=True, readonly=False, precompute=True,
        help="Choose an event ticket and it will automatically create a registration for this event ticket.")
    registration_ids = fields.One2many('event.registration', 'sale_order_line_id', string="Registrations")

    @api.depends('state', 'event_id')
    def _compute_product_uom_readonly(self):
        event_lines = self.filtered(lambda line: line.event_id)
        event_lines.update({'product_uom_readonly': True})
        super(SaleOrderLine, self - event_lines)._compute_product_uom_readonly()

    def _init_registrations(self):
        """ Create registrations linked to a sales order line. A sale
        order line has a product_uom_qty attribute that will be the number of
        registrations linked to this line. """
        registrations_vals = []
        for so_line in self:
            if not so_line.product_type == 'event':
                continue

            for _count in range(int(so_line.product_uom_qty) - len(so_line.registration_ids)):
                values = {
                    'sale_order_line_id': so_line.id,
                    'sale_order_id': so_line.order_id.id,
                }
                registrations_vals.append(values)

        if registrations_vals:
            self.env['event.registration'].sudo().create(registrations_vals)
        return True

    @api.depends('product_id')
    def _compute_event_id(self):
        event_lines = self.filtered(lambda line: line.product_id and line.product_id.detailed_type == 'event')
        (self - event_lines).event_id = False
        for line in event_lines:
            if line.product_id not in line.event_id.event_ticket_ids.product_id:
                line.event_id = False

    @api.depends('event_id')
    def _compute_event_ticket_id(self):
        event_lines = self.filtered('event_id')
        (self - event_lines).event_ticket_id = False
        for line in event_lines:
            if line.event_id != line.event_ticket_id.event_id:
                line.event_ticket_id = False

    @api.depends('event_ticket_id')
    def _compute_price_unit(self):
        super()._compute_price_unit()

    @api.depends('event_ticket_id')
    def _compute_name(self):
        """Override to add the compute dependency.

        The custom name logic can be found below in _get_sale_order_line_multiline_description_sale.
        """
        super()._compute_name()

    def _get_sale_order_line_multiline_description_sale(self):
        """ We override this method because we decided that:
                The default description of a sales order line containing a ticket must be different than the default description when no ticket is present.
                So in that case we use the description computed from the ticket, instead of the description computed from the product.
                We need this override to be defined here in sales order line (and not in product) because here is the only place where the event_ticket_id is referenced.
        """
        if self.event_ticket_id:
            return self.event_ticket_id._get_ticket_multiline_description() + self._get_sale_order_line_multiline_description_variants()
        else:
            return super()._get_sale_order_line_multiline_description_sale()

    def _use_template_name(self):
        """ We do not want configured description to get rewritten by template default"""
        if self.event_ticket_id:
            return False
        return super()._use_template_name()

    def _get_display_price(self):
        if self.event_ticket_id and self.event_id:
            event_ticket = self.event_ticket_id
            company = event_ticket.company_id or self.env.company
            pricelist = self.order_id.pricelist_id
            if pricelist.discount_policy == "with_discount":
                price = event_ticket.with_context(**self._get_pricelist_price_context()).price_reduce
            else:
                price = event_ticket.price
            return self._convert_to_sol_currency(price, company.currency_id)
        return super()._get_display_price()

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import event_event
from . import event_registration
from . import event_ticket
from . import product
from . import sale_order
from . import sale_order_line

```

## File: report\event_sale_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, tools
from odoo.addons.sale.models.sale_order import SALE_ORDER_STATE


class EventSaleReport(models.Model):
    """Event Registrations-based sales report, allowing to analyze sales and number of seats
    by event (type), ticket, etc. Each opened record will also give access to all this information."""
    _name = 'event.sale.report'
    _description = 'Event Sales Report'
    _auto = False
    _rec_name = 'sale_order_line_id'

    event_type_id = fields.Many2one('event.type', string='Event Type', readonly=True)
    event_id = fields.Many2one('event.event', string='Event', readonly=True)
    event_date_begin = fields.Date(string='Event Start Date', readonly=True)
    event_date_end = fields.Date(string='Event End Date', readonly=True)
    event_ticket_id = fields.Many2one('event.event.ticket', string='Event Ticket', readonly=True)
    event_ticket_price = fields.Float(string='Ticket price', readonly=True)
    event_registration_create_date = fields.Date(string='Registration Date', readonly=True)
    event_registration_state = fields.Selection([
        ('draft', 'Unconfirmed'), ('cancel', 'Cancelled'),
        ('open', 'Confirmed'), ('done', 'Attended')],
        string='Registration Status', readonly=True)
    active = fields.Boolean('Is registration active (not archived)?')
    event_registration_id = fields.Many2one('event.registration', readonly=True)
    event_registration_name = fields.Char('Attendee Name', readonly=True)

    product_id = fields.Many2one('product.product', string='Product', readonly=True)
    sale_order_id = fields.Many2one('sale.order', readonly=True)
    sale_order_date = fields.Datetime('Order Date', readonly=True)
    sale_order_partner_id = fields.Many2one('res.partner', string='Customer', readonly=True)
    sale_order_state = fields.Selection(
        selection=SALE_ORDER_STATE, string='Sale Order Status', readonly=True)
    sale_order_user_id = fields.Many2one('res.users', string='Salesperson', readonly=True)
    sale_order_line_id = fields.Many2one('sale.order.line', readonly=True)
    sale_price = fields.Float('Revenues', readonly=True)
    sale_price_untaxed = fields.Float('Untaxed Revenues', readonly=True)
    invoice_partner_id = fields.Many2one('res.partner', string='Invoice Address', readonly=True)
    sale_status = fields.Selection(string="Payment Status", selection=[
            ('to_pay', 'Not Sold'),
            ('sold', 'Sold'),
            ('free', 'Free'),
        ])
    company_id = fields.Many2one('res.company', string='Company', readonly=True)

    def init(self):
        tools.drop_view_if_exists(self._cr, self._table)
        self._cr.execute('CREATE OR REPLACE VIEW %s AS (%s);' % (self._table, self._query()))

    def _query(self, with_=None, select=None, join=None, group_by=None):
        return "\n".join([
            self._with_clause(*(with_ or [])),
            self._select_clause(*(select or [])),
            self._from_clause(*(join or [])),
            self._group_by_clause(*(group_by or []))
        ])

    def _with_clause(self, *with_):
        # Extra clauses formatted as `cte1 AS (SELECT ...)`, `cte2 AS (SELECT ...)`...
        return """
WITH
    """ + ',\n    '.join(with_) if with_ else ''

    def _select_clause(self, *select):
        # Extra clauses formatted as `cte1.column1 AS new_column1`, `table1.column2 AS new_column2`...
        return """
SELECT
    ROW_NUMBER() OVER (ORDER BY event_registration.id) AS id,

    event_registration.id AS event_registration_id,
    event_registration.company_id AS company_id,
    event_registration.event_id AS event_id,
    event_registration.event_ticket_id AS event_ticket_id,
    event_registration.create_date AS event_registration_create_date,
    event_registration.name AS event_registration_name,
    event_registration.state AS event_registration_state,
    event_registration.active AS active,
    event_registration.sale_order_id AS sale_order_id,
    event_registration.sale_order_line_id AS sale_order_line_id,
    event_registration.sale_status AS sale_status,

    event_event.event_type_id AS event_type_id,
    event_event.date_begin AS event_date_begin,
    event_event.date_end AS event_date_end,

    event_event_ticket.price AS event_ticket_price,

    sale_order.date_order AS sale_order_date,
    sale_order.partner_invoice_id AS invoice_partner_id,
    sale_order.partner_id AS sale_order_partner_id,
    sale_order.state AS sale_order_state,
    sale_order.user_id AS sale_order_user_id,

    sale_order_line.product_id AS product_id,
    CASE
        WHEN sale_order_line.product_uom_qty = 0 THEN 0
        ELSE
        sale_order_line.price_total
            / CASE COALESCE(sale_order.currency_rate, 0) WHEN 0 THEN 1.0 ELSE sale_order.currency_rate END
            / sale_order_line.product_uom_qty
    END AS sale_price,
    CASE
        WHEN sale_order_line.product_uom_qty = 0 THEN 0
        ELSE
        sale_order_line.price_subtotal
            / CASE COALESCE(sale_order.currency_rate, 0) WHEN 0 THEN 1.0 ELSE sale_order.currency_rate END
            / sale_order_line.product_uom_qty
    END AS sale_price_untaxed""" + (',\n    ' + ',\n    '.join(select) if select else '')

    def _from_clause(self, *join_):
        # Extra clauses formatted as `column1`, `column2`...
        return """
FROM event_registration
LEFT JOIN event_event ON event_event.id = event_registration.event_id
LEFT JOIN event_event_ticket ON event_event_ticket.id = event_registration.event_ticket_id
LEFT JOIN sale_order ON sale_order.id = event_registration.sale_order_id
LEFT JOIN sale_order_line ON sale_order_line.id = event_registration.sale_order_line_id
""" + ('\n'.join(join_) + '\n' if join_ else '')

    def _group_by_clause(self, *group_by):
        # Extra clauses formatted like `column1`, `column2`...
        return """
GROUP BY
    """ + ',\n    '.join(group_by) if group_by else ''

```

## File: report\event_sale_report_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="event_sale_report_view_graph" model="ir.ui.view">
        <field name="name">event.sale.report.view.graph</field>
        <field name="model">event.sale.report</field>
        <field name="arch" type="xml">
            <graph string="Revenues" sample="1" type="line">
                <field name="sale_price" type="measure"/>
                <field name="event_registration_create_date" interval="day"/>
                <field name="event_ticket_price" type="measure" invisible="True"/>
            </graph>
        </field>
    </record>

    <record id="event_sale_report_view_form" model="ir.ui.view">
        <field name="name">event.sale.report.view.form</field>
        <field name="model">event.sale.report</field>
        <field name="arch" type="xml">
            <form string="Registration revenues" edit="false" create="false">
                <sheet>
                    <group col="2">
                        <group string="Event">
                            <field name="event_type_id"/>
                            <field name="event_id"/>
                            <field name="event_date_begin"/>
                        </group>
                        <group string="Registration">
                            <field name="event_registration_id"/>
                            <field name="event_registration_name"/>
                            <field name="event_registration_create_date"/>
                            <field name="event_ticket_id"/>
                            <field name="event_registration_state"/>
                        </group>
                    </group>
                    <group col="2">
                        <group string="Sale Order">
                            <field name="sale_order_partner_id"/>
                            <field name="sale_order_id"/>
                            <field name="product_id"/>
                        </group>
                        <group string="Revenues">
                            <field name="event_ticket_price"/>
                            <field name="sale_price_untaxed"/>
                            <field name="sale_price"/>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="event_sale_report_view_pivot" model="ir.ui.view">
        <field name="name">event.sale.report.view.pivot</field>
        <field name="model">event.sale.report</field>
        <field name="arch" type="xml">
            <pivot string="Revenues" sample="1">
                <field name="sale_price_untaxed" type="measure"/>
                <field name="sale_price" type="measure"/>
                <field name="event_id" type="row"/>
                <field name="product_id" type="row"/>
                <field name="event_ticket_price" invisible="True"/>
            </pivot>
        </field>
    </record>

    <record id="event_sale_report_view_tree" model="ir.ui.view">
        <field name="name">event.sale.report.view.tree</field>
        <field name="model">event.sale.report</field>
        <field name="arch" type="xml">
            <tree string="Revenues" edit="false" create="false">
                <field name="event_id"/>
                <field name="event_ticket_id"/>
                <field name="product_id" optional="hide"/>
                <field name="event_ticket_price"/>
                <field name="sale_price_untaxed" optional="hide"/>
                <field name="sale_price" optional="hide"/>
                <field name="event_registration_state" optional="hide"/>
                <field name="sale_order_partner_id"/>
                <field name="invoice_partner_id" optional="hide"/>
                <field name="event_registration_name" optional="hide"/>
                <field name="sale_order_state" widget="badge"
                       decoration-success="sale_order_state == 'sale'"
                       decoration-info="sale_order_state == 'draft' or sale_order_state == 'sent'"/>
            </tree>
        </field>
    </record>

    <record id="event_sale_report_view_search" model="ir.ui.view">
        <field name="name">event.sale.report.view.search</field>
        <field name="model">event.sale.report</field>
        <field name="arch" type="xml">
            <search string="Event Sales Analysis">
                <field name="event_id"/>
                <field name="event_registration_name" string="Participant"/>
                <field name="sale_order_partner_id" string="Booked by"/>
                <field name="company_id"/>
                <filter string="Non-free tickets" name="priced_tickets" domain="[('event_ticket_price', '!=', 0)]"/>
                <separator/>
                <filter string="Free" name="free" domain="[('sale_status', '=', 'free')]"/>
                <filter string="Pending payment" name="payment_pending" domain="[('sale_status', '=', 'to_pay')]"/>
                <filter string="Sold" name="is_sold" domain="[('sale_status', '=', 'sold')]"/>
                <separator/>
                <filter string="Registration Date" name="event_registration_create_date" date="event_registration_create_date" default_period="this_year"/>
                <separator/>
                <filter string="Upcoming/Running" name="upcoming" help="Upcoming events from today"
                        domain="[('event_date_end', '&gt;=', datetime.datetime.combine(context_today(), datetime.time(0,0,0)))]"/>
                <filter string="Past Events" name="past" help="Events that have ended"
                        domain="[('event_date_end', '&lt;', datetime.datetime.combine(context_today(), datetime.time(0,0,0)))]"/>
                <filter string="Event Start Date" name="event_date_start" date="event_date_begin" default_period="this_year"/>
                <filter string="Event End Date" name="event_date_end" date="event_date_end"/>
                <group expand="0" string="Group By">
                    <filter string="Event Type" name="group_by_event_type_id" context="{'group_by': 'event_type_id' }"/>
                    <filter string="Event" name="group_by_event_id" context="{'group_by': 'event_id' }"/>
                    <separator/>
                    <filter string="Product" name="group_by_product_id" context="{'group_by': 'product_id'}"/>
                    <filter string="Ticket" name="group_by_ticket_id" context="{'group_by': 'event_ticket_id'}"/>
                    <separator/>
                    <filter string="Registration Status" name="group_by_registration_state"
                            context="{'group_by': 'event_registration_state'}"/>
                    <filter string="Sale Order Status" name="group_by_sale_order_state"
                            context="{'group_by': 'sale_order_state'}"/>
                    <filter string="Customer" name="group_by_customer" context="{'group_by': 'sale_order_partner_id'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="event_sale_report_action" model="ir.actions.act_window">
        <field name="name">Revenues</field>
        <field name="res_model">event.sale.report</field>
        <field name="view_mode">graph,pivot</field>
        <field name="context">{
            'search_default_priced_tickets': 1,
            'search_default_event_date_start': 1,
            'pivot_measures': ['__count__', 'sale_price_untaxed', 'sale_price'],
        }</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                No Event Revenues yet!
            </p><p>
                Come back once tickets have been sold to overview your sales income.
            </p>
        </field>
    </record>

    <menuitem name="Revenues"
        id="menu_action_show_revenues"
        action="event_sale_report_action"
        sequence="5"
        parent="event.menu_reporting_events"
        groups="event.group_event_user"/>
</odoo>

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import event_sale_report

```

## File: security\event_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="0">
        <record id="sales_team.group_sale_salesman" model="res.groups">
            <field name="implied_ids" eval="[(4, ref('event.group_event_registration_desk'))]"/>
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
access_event_sale_report_manager,access.event.sale.report.manager,model_event_sale_report,event.group_event_manager,1,0,0,0

```

## File: security\ir_rule.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <!-- Multi - Company Rules -->
    <record id="event_sale_report_comp_rule" model="ir.rule">
        <field name="name">Event Sales Report multi-company</field>
        <field name="model_id" ref="model_event_sale_report"/>
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
    </record>

</odoo>

```

## File: static\src\js\event_configurator_controller.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { useService } from "@web/core/utils/hooks";
import { formView } from "@web/views/form/form_view";

/**
 * This controller is overridden to allow configuring sale_order_lines through a popup
 * window when a product with 'detailed_type' == 'event' is selected.
 *
 * This allows keeping an editable list view for sales order and remove the noise of
 * those 2 fields ('event_id' + 'event_ticket_id')
 */

export class EventConfiguratorController extends formView.Controller {
    setup() {
        super.setup();
        this.action = useService("action");
    }

    /**
     * We let the regular process take place to allow the validation of the required fields
     * to happen.
     *
     * Then we can manually close the window, providing event information to the caller.
     *
     * @override
     */
    async onRecordSaved(record) {
        await super.onRecordSaved(...arguments);
        const { event_id, event_ticket_id } = record.data;
        return this.action.doAction({
            type: "ir.actions.act_window_close",
            infos: {
                eventConfiguration: {
                    event_id,
                    event_ticket_id,
                },
            },
        });
    }
}

registry.category("views").add("event_configurator_form", {
    ...formView,
    Controller: EventConfiguratorController,
});

```

## File: static\src\js\sale_product_field.js

```javascript
/** @odoo-module **/

import { patch } from "@web/core/utils/patch";
import { SaleOrderLineProductField } from '@sale/js/sale_product_field';


patch(SaleOrderLineProductField.prototype, {

    async _onProductUpdate() {
        super._onProductUpdate(...arguments);
        if (this.props.record.data.product_type === 'event') {
            this._openEventConfigurator();
        }
    },

    _editLineConfiguration() {
        super._editLineConfiguration(...arguments);
        if (this.props.record.data.product_type === 'event') {
            this._openEventConfigurator();
        }
    },

    get isConfigurableLine() {
        return super.isConfigurableLine || this.props.record.data.product_type === 'event';
    },

    async _openEventConfigurator() {
        let actionContext = {
            'default_product_id': this.props.record.data.product_id[0],
        };
        if (this.props.record.data.event_id) {
            actionContext.default_event_id = this.props.record.data.event_id[0];
        }
        if (this.props.record.data.event_ticket_id) {
            actionContext.default_event_ticket_id = this.props.record.data.event_ticket_id[0];
        }
        this.action.doAction(
            'event_sale.event_configurator_action',
            {
                additionalContext: actionContext,
                onClose: async (closeInfo) => {
                    if (!closeInfo || closeInfo.special) {
                        // wizard popup closed or 'Cancel' button triggered
                        if (!this.props.record.data.event_ticket_id) {
                            // remove product if event configuration was cancelled.
                            this.props.record.update({
                                [this.props.name]: undefined,
                            });
                        }
                    } else {
                        const eventConfiguration = closeInfo.eventConfiguration;
                        this.props.record.update(eventConfiguration);
                    }
                }
            }
        );
    },
});

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
            <field name="state" position="after">
                <field name="sale_status" optional="show" widget="badge"
                    decoration-success="sale_status == 'sold'"
                    decoration-danger="sale_status == 'to_pay'"/>
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
                    <field name="sale_status"/>
                </div>
            </xpath>
            <xpath expr="//div[@id='event_ticket_id']" position="before">
                <div class="d-md-none" >
                    <field name="sale_status"/>
                </div>
            </xpath>
        </field>
    </record>

    <record id="event_registration_view_graph" model="ir.ui.view">
        <field name="name">event.registration.graph.inherit.event.sale</field>
        <field name="model">event.registration</field>
        <field name="inherit_id" ref="event.view_event_registration_graph"/>
        <field name="arch" type="xml">
            <field name="event_id" position="after">
                <field name="sale_status"/>
            </field>
        </field>
    </record>

    <record id="event_registration_ticket_view_form" model="ir.ui.view">
        <field name="name">event.registration.form.inherit</field>
        <field name="model">event.registration</field>
        <field name="inherit_id" ref="event.view_event_registration_form" />
        <field name="arch" type="xml">
            <xpath expr="//div[@name='button_box']" position="inside">
                <button name="action_view_sale_order" type="object"
                        class="oe_stat_button" icon="fa-usd"
                        groups="sales_team.group_sale_salesman"
                        invisible="not sale_order_id">
                        <div class="o_stat_info">
                            <span class="o_stat_text">Sale Order</span>
                        </div>
                </button>
                <field name="sale_order_id" invisible="1"/>
            </xpath>
            <xpath expr="//group" position="before">
                <field name="sale_status" invisible="1"/>
                <widget name="web_ribbon" title="Sold" bg_color="text-bg-success"
                    invisible="sale_status != 'sold'"/>
                <widget name="web_ribbon" title="Not Sold" bg_color="text-bg-danger"
                    invisible="sale_status in ('sold', 'free') or not id"/>
            </xpath>
            <group name="utm_link" position="before">
                <group string="Transaction" groups="base.group_no_one">
                    <field name="sale_order_id"/>
                    <field name="sale_order_line_id" readonly="1" invisible="not sale_order_id"/>
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
                <field name="product_id" context="{'default_detailed_type': 'event'}"/>
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
                <field name="product_id" context="{'default_detailed_type': 'event'}"/>
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
            <field name="start_sale_datetime" position="attributes">
                <attribute name="string">Sales Start</attribute>
            </field>
            <field name="end_sale_datetime" position="attributes">
                <attribute name="string">Sales End</attribute>
            </field>
            <field name="name" position="after">
                <field name="product_id" context="{'default_detailed_type': 'event'}"/>
            </field>
            <field name="description" position="after">
                <field name="currency_id" column_invisible="True"/>
                <field name="price" widget="monetary" options="{'currency_field': 'currency_id'}"/>
            </field>
        </field>
    </record>

    <record id="event_event_ticket_view_form_from_event" model="ir.ui.view">
        <field name="name">event.event.ticket.view.form.from.event.inherit.sale</field>
        <field name="model">event.event.ticket</field>
        <field name="inherit_id" ref="event.event_event_ticket_view_form_from_event"/>
        <field name="arch" type="xml">
            <field name="name" position="after">
                <field name="product_id" context="{'default_detailed_type': 'event'}"/>
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
                <div class="col-4 text-end"><strong> <t t-out="record.price.value"/></strong></div>
            </xpath>
            <xpath expr="//div[hasclass('row')]" position="after">
                <div t-out="record.product_id.value"/>
            </xpath>
        </field>
    </record>

    <record id="event_event_ticket_form_view" model="ir.ui.view">
        <field name="name">event.event.ticket.view.form.inherit.sale</field>
        <field name="model">event.event.ticket</field>
        <field name="inherit_id" ref="event.event_event_ticket_form_view"/>
        <field name="arch" type="xml">
            <field name="end_sale_datetime" position="after">
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
        <field name="priority" eval="20"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@name='%(event.act_event_registration_from_event)d']" position="after">
                <field name="currency_id" invisible="1"/>
                <button name="action_view_linked_orders"
                        type="object" class="oe_stat_button" icon="fa-dollar"
                        groups="sales_team.group_sale_salesman"
                        help="Total sales for this event"
                        invisible="sale_price_subtotal == 0 or not sale_price_subtotal">
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

## File: views\sale_order_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="sale_order_view_form" model="ir.ui.view">
        <field name="name">sale.order.form.inherit.event.sale</field>
        <field name="model">sale.order</field>
        <field name="inherit_id" ref="sale.view_order_form" />
        <field name="arch" type="xml">
            <xpath expr="//sheet/div[hasclass('oe_button_box')]" position="inside">
                <button name="action_view_attendee_list" type="object"
                    class="oe_stat_button" icon="fa-users" invisible="attendee_count == 0">
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
                    invisible="product_type != 'event'"
                    required="product_type == 'event'"
                    options="{'no_open': True, 'no_create': True}"/>
                <field
                    name="event_ticket_id"
                    domain="[
                        ('event_id', '=', event_id), ('product_id','=',product_id),
                         '|', ('company_id', '=', False), ('company_id', '=', parent.company_id)
                    ]"
                    invisible="product_type != 'event' or not event_id"
                    required="product_type == 'event' and event_id"
                    options="{'no_open': True, 'no_create': True}"/>
            </xpath>
            <xpath expr="//field[@name='order_line']//tree//field[@name='product_template_id']" position="after">
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

from odoo import _, api, models, fields
from odoo.exceptions import ValidationError


class EventConfigurator(models.TransientModel):
    _name = 'event.event.configurator'
    _description = 'Event Configurator'

    product_id = fields.Many2one('product.product', string="Product", readonly=True)
    event_id = fields.Many2one('event.event', string="Event")
    event_ticket_id = fields.Many2one('event.event.ticket', string="Event Ticket")

    @api.constrains('event_id', 'event_ticket_id')
    def check_event_id(self):
        error_messages = []
        for record in self:
            if record.event_id.id != record.event_ticket_id.event_id.id:
                error_messages.append(
                    _('Invalid ticket choice "%(ticket_name)s" for event "%(event_name)s".'))
        if error_messages:
            raise ValidationError('\n'.join(error_messages))

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
                        context="{'name_with_seats_availability': True}"
                        options="{'no_open': True, 'no_create': True}"
                    />
                    <field
                        name="event_ticket_id"
                        domain="[('event_id', '=', event_id), ('product_id', '=', product_id)]"
                        invisible="not event_id"
                        required="event_id"
                        context="{'name_with_seats_availability': True}"
                        options="{'no_open': True, 'no_create': True}"/>
                    <field name="product_id" invisible="1"/>
                </group>
                <footer>
                    <button string="Ok" class="btn-primary o_event_sale_js_event_configurator_ok" special="save" data-hotkey="q"/>
                    <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="x"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="event_configurator_action" model="ir.actions.act_window">
        <field name="name">Configure an event</field>
        <field name="res_model">event.event.configurator</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
        <field name="context">{'name_with_seats_availability': True}</field>
        <field name="view_id" ref="event_configurator_view_form"/>
    </record>
</odoo>

```

## File: wizard\event_edit_registration.py

```python
# -*- coding: utf-8 -*-

from collections import Counter, defaultdict

from odoo import models, fields, api
from odoo.exceptions import ValidationError


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

        so_lines = sale_order.order_line.filtered('event_ticket_id')
        so_line_to_reg = registrations.grouped('sale_order_line_id')
        attendee_list = []
        for so_line in so_lines:
            registrations = so_line_to_reg.get(so_line, self.env['event.registration'])
            # Add existing registrations
            attendee_list += [[0, 0, {
                'event_id': reg.event_id.id,
                'event_ticket_id': reg.event_ticket_id.id,
                'registration_id': reg.id,
                'name': reg.name,
                'email': reg.email,
                'phone': reg.phone,
                'sale_order_line_id': so_line.id,
            }] for reg in registrations]
            # Add new registrations
            attendee_list += [[0, 0, {
                'event_id': so_line.event_id.id,
                'event_ticket_id': so_line.event_ticket_id.id,
                'sale_order_line_id': so_line.id,
                'name': so_line.order_partner_id.name,
                'email': so_line.order_partner_id.email,
                'phone': so_line.order_partner_id.phone,
            }] for _count in range(int(so_line.product_uom_qty) - len(registrations))]
        res['event_registration_ids'] = attendee_list
        res = self._convert_to_write(res)
        return res

    def action_make_registration(self):
        self.ensure_one()
        registrations_to_create = []
        for registration_line in self.event_registration_ids:
            if registration_line.registration_id:
                registration_line.registration_id.write(registration_line._prepare_registration_data())
            else:
                registrations_to_create.append(registration_line._prepare_registration_data(include_event_values=True))

        self.env['event.registration'].create(registrations_to_create)

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
    name = fields.Char(string='Name')

    def _prepare_registration_data(self, include_event_values=False):
        self.ensure_one()
        registration_data = {
            'partner_id': self.editor_id.sale_order_id.partner_id.id,
            'name': self.name or self.editor_id.sale_order_id.partner_id.name,
            'phone': self.phone or self.editor_id.sale_order_id.partner_id.phone or self.editor_id.sale_order_id.partner_id.mobile,
            'email': self.email or self.editor_id.sale_order_id.partner_id.email,
        }
        if include_event_values:
            registration_data.update({
                'event_id': self.event_id.id,
                'event_ticket_id': self.event_ticket_id.id,
                'sale_order_id': self.editor_id.sale_order_id.id,
                'sale_order_line_id': self.sale_order_line_id.id,
            })
        return registration_data

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
                    <sheet>
                        <p>Before updating the linked registrations of <field name="sale_order_id" readonly="1" class="oe_inline"/>
                        please give attendee details.</p>
                        <field name="event_registration_ids">
                            <tree string="Registration" editable="top" create="false" delete="false">
                                <field name="event_id" readonly='1' force_save="1"/>
                                <field name="registration_id" readonly='1' force_save="1"/>
                                <field name="event_ticket_id" domain="[('event_id', '=', event_id)]" readonly='1' force_save="1"/>
                                <field name="name"/>
                                <field name="email"/>
                                <field name="phone" class="o_force_ltr"/>
                                <field name="sale_order_line_id" column_invisible="True"/>
                            </tree>
                        </field>
                    </sheet>
                    <footer>
                        <button string="Create/Update registrations" name="action_make_registration" type="object" class="btn-primary" data-hotkey="q"/>
                        <button string="Skip" class="btn-secondary" special="cancel" data-hotkey="x"/>
                    </footer>
                </form>
            </field>
        </record>

        <record id="action_sale_order_event_registration" model="ir.actions.act_window">
            <field name="name">Event Registrations</field>
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

