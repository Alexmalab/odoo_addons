# Odoo Module: event_product

Category: Marketing/Events

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
{
    'name': 'Events Product',
    'version': '1.0',
    'category': 'Marketing/Events',
    'depends': ['event', 'product', 'account'],
    'data': [
        'views/event_ticket_views.xml',
        'data/event_product_data.xml',
    ],
    'demo': [
        'data/event_product_demo.xml',
        'data/event_demo.xml',
    ],
    'installable': True,
    'auto_install': True,
    'assets': {},
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
        <field name="product_id" ref="event_product.product_product_event_standard"/>
        <field name="price">1000.0</field>
    </record>
    <record id="event.event_0_ticket_2" model="event.event.ticket">
        <field name="product_id" ref="event_product.product_product_event_vip"/>
        <field name="price">1500.0</field>
    </record>

    <record id="event.event_2_ticket_1" model="event.event.ticket">
        <field name="product_id" ref="event_product.product_product_event_standard"/>
        <field name="price">1000.0</field>
    </record>
    <record id="event.event_2_ticket_2" model="event.event.ticket">
        <field name="product_id" ref="event_product.product_product_event_vip"/>
        <field name="price">1500.0</field>
    </record>

    <record id="event.event_4_ticket_0" model="event.event.ticket">
        <field name="product_id" ref="event_product.product_product_event_standard"/>
        <field name="price">99.0</field>
    </record>

    <record id="event.event_7_ticket_1" model="event.event.ticket">
        <field name="product_id" ref="event_product.product_product_event_standard"/>
        <field name="price">0.0</field>
    </record>
    <record id="event.event_7_ticket_2" model="event.event.ticket">
        <field name="product_id" ref="event_product.product_product_event_vip"/>
        <field name="price">0.0</field>
    </record>
</odoo>

```

## File: data\event_product_data.xml

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
            <field name="categ_id" ref="event_product.product_category_events"/>
            <field name="type">service</field>
            <field name="service_tracking">event</field>
        </record>
    </data>
</odoo>

```

## File: data\event_product_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="product_product_event_standard" model="product.product">
            <field name="list_price">30.0</field>
            <field name="standard_price">10.0</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="name">Event Registration - Standard</field>
            <field name="description_sale" eval="False"/>
            <field name="categ_id" ref="event_product.product_category_events"/>
            <field name="type">service</field>
            <field name="service_tracking">event</field>
        </record>
        <record id="product_product_event_vip" model="product.product">
            <field name="list_price">100.0</field>
            <field name="standard_price">50.0</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="name">Event Registration - VIP</field>
            <field name="description_sale" eval="False"/>
            <field name="categ_id" ref="event_product.product_category_events"/>
            <field name="type">service</field>
            <field name="service_tracking">event</field>
        </record>
    </data>
</odoo>

```

## File: models\event_event.py

```python
from odoo import fields, models


class Event(models.Model):
    _inherit = 'event.event'

    currency_id = fields.Many2one(
        'res.currency', string='Currency',
        related='company_id.currency_id', readonly=True)

```

## File: models\event_event_ticket.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, models, fields


class EventTicket(models.Model):
    _inherit = 'event.event.ticket'
    _order = "event_id, sequence, price, name, id"

    price_reduce_taxinc = fields.Float(
        string='Price Reduce Tax inc', compute='_compute_price_reduce_taxinc',
        compute_sudo=True)
    price_incl = fields.Float(
        string='Price include', compute='_compute_price_incl',
        digits='Product Price', readonly=False, compute_sudo=True)

    @api.depends('product_id.active')
    def _compute_sale_available(self):
        inactive_product_tickets = self.filtered(lambda ticket: not ticket.product_id.active)
        for ticket in inactive_product_tickets:
            ticket.sale_available = False
        super(EventTicket, self - inactive_product_tickets)._compute_sale_available()

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

```

## File: models\event_type_ticket.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from odoo import api, fields, models
from odoo.addons.product.models.product_template import PRICE_CONTEXT_KEYS

_logger = logging.getLogger(__name__)


class EventTemplateTicket(models.Model):
    _inherit = 'event.type.ticket'
    _order = "sequence, price, name, id"

    def _default_product_id(self):
        return self.env.ref('event_product.product_product_event', raise_if_not_found=False)

    description = fields.Text(compute='_compute_description', readonly=False, store=True)
    # product
    product_id = fields.Many2one(
        'product.product', string='Product', required=True,
        domain=[("service_tracking", "=", "event")], default=_default_product_id)
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
            return super()._init_column(column_name)

        # fetch void columns
        self.env.cr.execute("SELECT id FROM %s WHERE product_id IS NULL" % self._table)
        ticket_type_ids = self.env.cr.fetchall()
        if not ticket_type_ids:
            return

        # update existing columns
        _logger.debug("Table '%s': setting default value of new column %s to unique values for each row",
                      self._table, column_name)
        default_event_product = self.env.ref('event_product.product_product_event', raise_if_not_found=False)
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
                'module': 'event_product',
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
        return super()._get_event_ticket_fields_whitelist() + ['product_id', 'price']

```

## File: models\product_product.py

```python
from odoo import fields, models


class Product(models.Model):
    _inherit = 'product.product'

    event_ticket_ids = fields.One2many('event.event.ticket', 'product_id', string='Event Tickets')

```

## File: models\product_template.py

```python
from odoo import fields, models


class ProductTemplate(models.Model):
    _inherit = 'product.template'

    service_tracking = fields.Selection(selection_add=[
        ('event', 'Event Registration'),
    ], ondelete={'event': 'set default'})

    def _service_tracking_blacklist(self):
        return super()._service_tracking_blacklist() + ['event']

```

## File: models\__init__.py

```python
from . import event_event
from . import event_event_ticket
from . import event_type_ticket
from . import product_product
from . import product_template

```

## File: views\event_ticket_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>

    <!-- EVENT.TYPE.TICKET -->
    <record id="event_type_ticket_view_tree_from_type" model="ir.ui.view">
        <field name="name">event.type.ticket.view.list.inherit.event.product</field>
        <field name="model">event.type.ticket</field>
        <field name="inherit_id" ref="event.event_type_ticket_view_tree_from_type"/>
        <field name="arch" type="xml">
            <field name="name" position="before">
                <field name="product_id"
                    context="{
                        'default_type': 'service',
                        'default_service_tracking': 'event',
                    }"
                />
            </field>
            <field name="description" position="after">
                <field name="price"/>
            </field>
        </field>
    </record>

    <record id="event_type_ticket_view_form_from_type" model="ir.ui.view">
        <field name="name">event.type.ticket.view.form.inherit.event.product</field>
        <field name="model">event.type.ticket</field>
        <field name="inherit_id" ref="event.event_type_ticket_view_form_from_type"/>
        <field name="arch" type="xml">
            <field name="name" position="before">
                <field name="product_id"
                    context="{
                        'default_type': 'service',
                        'default_service_tracking': 'event',
                    }"
                />
            </field>
            <field name="description" position="after">
                <field name="price"/>
            </field>
        </field>
    </record>

    <!-- EVENT.TICKET -->
    <record id="event_event_ticket_view_tree_from_event" model="ir.ui.view">
        <field name="name">event.event.ticket.view.list.from.event.inherit.event.product</field>
        <field name="model">event.event.ticket</field>
        <field name="inherit_id" ref="event.event_event_ticket_view_tree_from_event"/>
        <field name="arch" type="xml">
            <field name="start_sale_datetime" position="attributes">
                <attribute name="string">Sales Start</attribute>
            </field>
            <field name="end_sale_datetime" position="attributes">
                <attribute name="string">Sales End</attribute>
            </field>
            <field name="name" position="before">
                <field name="product_id"
                    context="{
                        'default_type': 'service',
                        'default_service_tracking': 'event',
                    }"
                />
            </field>
            <field name="description" position="after">
                <field name="price" widget="monetary" options="{'currency_field': 'currency_id'}"/>
            </field>
        </field>
    </record>

    <record id="event_event_ticket_view_form_from_event" model="ir.ui.view">
        <field name="name">event.event.ticket.view.form.from.event.inherit.event.product</field>
        <field name="model">event.event.ticket</field>
        <field name="inherit_id" ref="event.event_event_ticket_view_form_from_event"/>
        <field name="arch" type="xml">
            <field name="name" position="before">
                <field name="product_id"
                    context="{
                        'default_type': 'service',
                        'default_service_tracking': 'event',
                    }"
                />
            </field>
            <field name="description" position="after">
                <field name="price"/>
            </field>
        </field>
    </record>

    <record id="event_event_ticket_view_kanban_from_event" model="ir.ui.view">
        <field name="name">event.event.ticket.view.kanban.from.event.product</field>
        <field name="model">event.event.ticket</field>
        <field name="inherit_id" ref="event.event_event_ticket_view_kanban_from_event"/>
        <field name="arch" type="xml">
            <field name="name" position="after">
                <field class="fw-bold ms-auto" name="price"/>
            </field>
            <xpath expr="//div[hasclass('d-flex')]" position="after">
                <field name="product_id"/>
            </xpath>
        </field>
    </record>

    <record id="event_event_ticket_form_view" model="ir.ui.view">
        <field name="name">event.event.ticket.view.form.inherit.event.product</field>
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

